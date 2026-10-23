# OpenCode v2 impact research (2026-10-15)

Research for TODO #136. Sources: fork `C:\Users\Wasiejen\Repos\opencode` on `production`
(654b79b27e, v2.0.29 generation; all fork pointers below are path:line into that
checkout), official v2 docs at `https://opencode.ai/v2/docs/` (migrate-v1 page:
`maintainer/done/2026-10-15_migration_from_v1.md` local copy; plugin-migration guide:
fork `services/www/src/docs/content/build/plugins/migrate-v1.mdx`), and this repo's
`.opencode/plugin/`, `.opencode/tools/`, `opencode.jsonc`.

Facts marked "measured" were observed on disk at research time, not read from docs.

## 0. Intercept-plugin port analysis (PRIORITY)

**Question (a): does the v2 plugin API expose a `tool.execute.before`-equivalent that
can MUTATE tool args before execution?**

**YES.** API shape (promise surface, the port target for our plugins):

```ts
import { Plugin } from "@opencode/plugin"   // packages/plugin/package.json exports "."

export default Plugin.define({
  id: "intercept",
  async setup(ctx) {
    await ctx.tool.hook("execute.before", (event) => {
      // event: { tool, sessionID, agent, messageID, id, input } — `input` is MUTABLE
      const input = event.input as { path?: string }
      input.path = resolveFuzzy(input.path)
    })
  },
})
```

Evidence:
- Hook event carries a mutable `input` (no `readonly`, unlike `execute.after` which is
  `readonly input`): fork `packages/plugin/src/promise/tool.ts:38-46` (effect twin:
  `packages/plugin/src/effect/tool.ts:20-28`).
- The core fires the hook and executes with the POST-hook `event.input`:
  `packages/core/src/tool.ts:103-111` (`beforeExecute`), `tool.ts:271-280`
  (`const event = yield* beforeExecute(...)` then `executeTool(tool, name, event.input, context)`;
  the CodeMode `execute` path does the same at `tool.ts:240-243`).
- The hook trigger runs callbacks on the same event object and returns it:
  `packages/core/src/plugin/hooks.ts:88-95`.
- **Built-in proof that arg mutation is a first-class supported pattern:** the shipped
  `opencode.tool.input.repair` plugin reassigns `event.input` inside `execute.before`
  (`packages/core/src/plugin/tool-input-repair.ts:21-36`).
- Bonus mutation surfaces: `event.tool` is also mutable — the snapshot resolves the tool
  by `event.tool` AFTER the hook (`tool.ts:272`), so a hook can redirect to another tool.
  And `execute.before` is the ONLY hook with a failure channel: throwing a `Tool.Error`
  rejects the call before it runs (`packages/plugin/src/effect/tool.ts:48-52`,
  `packages/core/src/plugin/hooks.ts:23`) — the exact surface our (4b) hard-stop used
  (a throwing v1 before hook fails the call).
- Official mapping: v1 `tool.execute.before(input, output)` →
  v2 `ctx.tool.hook("execute.before", ...)` (plugin-migration guide, hook table).

**Question (b): where does fuzzy path resolution plug in on v2 — plugin hook or fork
change in the core tool pipeline?**

The v2 tool pipeline is a per-tool plugin registry: tools are Effect plugins registered
via `ctx.tool.transform` (`packages/core/src/tool/plugin/*.ts` — `edit.ts`, `read.ts`,
`write.ts`, `glob.ts`, `grep.ts`, `shell.ts`, ...). The edit tool (`edit.ts`) does exact
match, Location-relative resolution (`access.resolve({ path: input.path })`,
`edit.ts:244`), external-path approval via a separate `external_directory` permission
(`edit.ts:245`; `packages/core/src/file-access.ts:26,117,136`), and typographic
match-string normalization (`edit.ts:49-55,168-188`). Field rename vs v1: v2
read/write/edit take **`path`**, not v1's `filePath` (measured:
`read.ts:17-26`, `write.ts:23-24`, `edit.ts:24-34`; `shell.ts:48-52` has
`command` + `workdir`); v2 tool names: `bash`→`shell`, `task`→`subagent`,
`write`/`patch`→`edit` (migrate-v1 docs "Permissions and tools").

**Recommendation: pure plugin, no fork change.** Two viable plugin-only insertion
points, in order of fit:

1. `ctx.tool.hook("execute.before")` — rewrite `event.input.path` before the tool's
   `access.resolve`/`authorizeExternal`/`permission.assert` run. Same order as our v1
   before hook (mutate first, then the host resolves/approves the mutated path), so the
   permission ask lands on the RESOLVED path — identical semantics to v1. This is the
   1:1 port of `intercept_observer.ts`.
2. `ctx.tool.transform((editor) => editor.update("edit", (tool) => { tool.execute = ... }))`
   — wrap/replace a tool's executor (`packages/plugin/src/promise/tool.ts:33-34`).
   Deeper (sees the definition, can replace behavior wholesale) but re-implements tool
   logic; use only if the hook surface proves insufficient.

Fork changes in the core pipeline are NOT required for the surviving port set. One
fork-level consideration (optional): the built-in input-repair plugin shows the vendor
expects arg repair at the hook level — we align with vendor direction.

**Scope cut confirmation (maintainer ruling 2026-10-15).** Surviving set = fuzzy path
resolution (read-scoped d≤2; write-scoped d≤1 for edit/block_transfer, `write` excluded)
+ the observation log to `intercept.log`. The dropped `[left:right]` PAIR channels are
confirmed consistent with the plugin header (`.opencode/plugin/intercept_observer.ts:17-71`):
(2a) read-scope pair, (3a) write-scope pair, and **(3c) git-refs-in-bash IS a
pair-based channel** ("GIT REFS in the bash `command` string: pair → digit (right-wins),
gated on ref existence", header lines 60-71) — so the spec's drop of 3c with the pair
channels is confirmed, not an open assumption. Also outside the surviving set (per spec):
(4) R6 payload journal (observation-only; trivially portable if wanted later), (4b)
hard-stop (the v2 hook CAN throw — surface exists, decision to drop stands), (5)
edit-oldString fuzzy, (6) after-hook enrichment. Note for the port: our v1 log channel
reads model id from the session DB (`readGauge` in `scripts/gauge.mjs`) — on v2 the
session table is `session_v2` (see §2), so that reader needs a schema update.

**Recommendation block (§0):** port the intercept as a v2 plugin using
`ctx.tool.hook("execute.before")` (point 1); map v1 `filePath` → v2 `path` for
read/edit/write; keep the observation log + numword map read; drop pair channels
incl. 3c (confirmed pair-based); no fork work.

## 1. Architecture

Package layout (measured, `packages/`): `core` (Effect app: sessions, tools, DB,
config), `server` (HTTP API + process), `plugin` (plugin API: `promise` + `effect`
surfaces), `cli` (the `opencode` binary: commands, updater, server process bootstrap),
`tui` (terminal client), `client` (generated API client), `schema` (Effect Schema types),
`protocol` (HTTP API endpoint definitions), `ai` (LLM client + provider protocol drivers),
`codemode` (in-process "execute" interpreter), `session-ui`, `app`, `desktop`, `web`,
`util`, `theme`, `sdk`, `console`, ... **`llm` is empty in this checkout** (only
`node_modules`, no tracked files — measured via `git ls-files packages/llm`).

Process model — a shared background service:
- The CLI runs in mode `default` | `service` | `stdio`
  (`packages/cli/src/server-process.ts:19-23`).
- `service` mode: a persistent background process with incumbent detection — if a
  service already listens on the port, the CLI connects instead of starting another
  (`server-process.ts:86-91`); it keeps the install replaceable via a retained image on
  Windows (`server-process.ts:93-95`).
- The service process owns the DB (`server-process.ts:107-109` →
  `packages/server/src/routes.ts:117`), the session runners, plugins, and all provider
  traffic.
- The TUI is a thin client of the server API (`packages/tui`).

How v2 calls providers: in-process HTTP via the `@opencode/ai` protocol drivers
(`packages/ai/src/protocols/` — `openai-compatible.ts` is the one our llama-swap
provider uses). No child-process providers (measured: no `spawn`/`child_process` in
`packages/ai/src/llm.ts`, `provider.ts`, or `packages/codemode/src`). Subagents run as
sessions in the same process (`packages/core/src/tool/plugin/subagent.ts:271`);
persistent terminals (PTY) are the only spawned children and never touch the backend.

**llama-swap single-slot risks (new in v2):**
- v2 issues AUXILIARY model calls that v1 did not: session titles
  (`packages/core/src/session/title.ts`), compaction summaries
  (`packages/core/src/session/compaction.ts:232-233` — `compactNatively` or
  `summarize`), `generate` (transient text), and — opt-in — **`warming`**: a built-in
  plugin that fires keep-alive model requests to recently active sessions every 4 min
  for up to 30 min (`packages/core/src/plugin/warming.ts:6-57`, config field
  `warming`, default false — `packages/schema/src/config.ts:109-111`). Any of these
  evicts the live session's model on our single-slot backend. Keep `warming` OFF and
  keep sessions serial; consider a dedicated compaction model (`compaction` settings
  exist on provider/model — `packages/schema/src/config/provider.ts` `Provider.Compaction`).
- The auto-compaction trigger is window-based: `due(context, calculateCeiling(model.limit,
  buffer))` (`packages/core/src/session/compaction.ts:211,911-916`).

**Recommendation block (§1):** the llama-swap setup survives unchanged (same in-process
HTTP, one model at a time if sessions stay serial). New subprocesses that would hit the
backend: none. New in-process backend traffic: title/compaction/generate/warming calls —
disable `warming`, budget compaction explicitly, keep serial-session discipline.

## 2. DB v1→v2

**v2 schema** (drizzle, `packages/core/src/database/schema.gen.ts`): `session_v2`
(lines 184-223: `revert`, `permission`, `agent`, `model`, `time_compacting`,
`resume_attempts`, token counters), `session_message` (160-170: `id, session_id, type,
seq, time_created, time_updated, data` — `data` is a JSON text blob), `event` +
`event_sequence` (61-70), `kv` (71-78), `session_inbox` (148-158), `session_pending`
(172-182), `permission` (80-89), `project`, `worktree`, `instruction_*`, `workspace`.
**There is no `part` table in v2** — v1 parts are folded into the message record during
migration.

**What `v1-migration` maps** (`packages/core/src/database/v1-migration.bun.ts`):
- Reads v1 rows from the SAME file: `SELECT ... FROM session` (line 589, 610-624),
  `FROM message WHERE session_id = ?` (631-633), `FROM part WHERE session_id = ?`
  (634-636).
- `transformSession` (222-491) decodes v1 `SessionV1.Info`/`SessionV1.Part` blobs:
  user/assistant text parts → v2 `user`/`assistant` messages; file parts →
  attachments; v1 **compaction part with `tail_start_id`** → v2 `compaction`
  message preserving the tail (295-315); assistant snapshots/patches/finish →
  v2 fields (395-401, 457-491).
- Writes into `session_v2` (610-624) and `session_message` with `data` =
  `JSON.stringify(message.data)` (642-655); updates the v1 `session` row with the
  transformed cost/token fields (656-660); records an `event_sequence` watermark
  (661-668); marks completion in `kv` under `migration.v1-v2` (683-695).
- **Data blobs survive**: v2 `session_message.data` IS the message-level JSON blob
  (v1 message+part data re-encoded into it). Candidate U2 filter-store surfaces on
  v2: the `kv` table (generic key/value) and per-message `data.metadata`.
- Resumable mid-run via the KV cursor (`534-575`); imports a PREVIOUS v2 database
  `opencode-next.db` from the data dir if present (`702-748`); **clears the `event`
  table first** (543-565).
- `tail_start_id` exists only in the v1 schema
  (`packages/schema/src/v1/session.ts:200`) and the migration source — v2 has no
  equivalent (measured: no `tail_start` in v2 session code).

**DB file path — same location as v1.**
- v2 opens `databasePath(global.data)` (`packages/cli/src/database-path.ts:4-12`):
  `opencode.db` in the data dir for channels latest/dev/beta/next/prod (or
  `OPENCODE_DISABLE_CHANNEL_DB=1`), else `opencode-<channel>.db`.
- `global.data` = `$XDG_DATA_HOME/opencode` or `~/.local/share/opencode`
  (`packages/util/src/global-roots.ts:5,13`).
- Measured: our live v1 DB is `C:\Users\Wasiejen\.local\share\opencode\opencode.db` —
  exactly where v2 (prod channel) points.

**What happens when v2 opens our live v1 `opencode.db`: in-place migration.**
1. `DatabaseMigration.apply` bootstraps the v2 schema INTO THE SAME FILE (v2 tables
   created alongside v1 tables) — `packages/core/src/database/migration.ts:30-48`,
   called from `database.ts:49`.
2. `V1Migration.layer` is merged into the server API layer and **auto-runs `run()` at
   service startup** (`packages/server/src/routes.ts:189`,
   `v1-migration.bun.ts:505-522`).
3. `run()` checks `hasLegacySessions` (v1 `session` table exists,
   `v1-migration.bun.ts:1130-1135`) and migrates in place as above.
- **One-way**: there is no v2→v1 path (measured: only `v1-migration` exists). The v1
  tables are not dropped — v2 simply reads `session_v2`/`session_message`. Status
  endpoint `GET /api/experimental/migration/v1` reports required/running/completed
  (`packages/server/src/handlers/migration.ts:6-12`,
  `packages/protocol/src/groups/migration.ts:20-27`).

**Recommendation block (§2):** treat first v2 service start as a live-DB rewrite event.
Before it: copy `opencode.db` to a scratch file (e.g. `opencode-dryrun.db` +
`OPENCODE_DB=opencode-dryrun.db` — the env var overrides the filename,
`database-path.ts:6`) and dry-run the migration there; verify sessions/messages
before pointing v2 at the real file. Our DB-reading tools (gauge, context_trim) must
move to the `session_v2`/`session_message` schema (no `part` table; no `tail_start_id`).

## 3. Compaction & assembly

**Compaction** (`packages/core/src/session/compaction.ts`):
- Settings: `{ auto: boolean; buffer?: number; keep: number }` (tokens of recent
  conversation kept verbatim beside the summary; defaults `auto: true, keep: 15_000`,
  lines 41-47, 197). Config-level hook verified: the built-in
  `opencode.config.compaction` plugin maps config `auto`/`buffer`/`keep.tokens` via
  `compaction.transform` (`packages/core/src/config/plugin/compaction.ts:15-24`).
  Our config's `keep.system`/`keep.messages` have NO v2 fields
  (`packages/schema/src/config/compaction.ts` — `Keep` = `tokens` only).
- Trigger reasons: `auto` | `overflow` | `manual` (53-57); manual is an **inbox item**
  (`/compact`) whose `inputID` message shows the outcome (56-57).
- Two engines: native provider compaction (`compactNatively`) or a summarizer model
  (`summarize`) (232-233); split = `splitConversation(context.messages, keep)`
  (705-729) — oldest summarized, newest `keep` tokens kept verbatim.
- The result is a **`compaction` message** (a checkpoint) with `summary`, `recent`,
  optional provider checkpoint; rendered back into context as a
  `<conversation-checkpoint>` block with `<summary>` + `<recent-context>`
  (`packages/core/src/session/runner/to-llm-message.ts:306-325`). This is the v2
  equivalent of the U3 marker: the marker IS a durable message in history.
- `session_pending` holds one in-flight compaction per session
  (`schema.gen.ts:267-269`).

**Message assembly** (candidate U2 insertion points):
1. DB `session_message` rows → `SessionMessage.Info[]` (projected history).
2. `toLLMMessages(messages, model.ref, ...)` — per-type translation
   (`runner/to-llm-message.ts:253-334`; types: user, synthetic, skill, system, shell,
   assistant, compaction, model-switched, location-switched, agent-switched→[],
   idle→[]).
3. `baseTranscript` — system parts (agent system + initial) + messages
   (`packages/core/src/session/model-request.ts:100-120`).
4. `SessionModelRequest.prepare(kind, input, agentHook)` (225-335):
   - **`hooks.trigger("session", "context"|"compaction"|"generate"|"title", draft)`**
     — the draft `{ sessionID, model, system, messages, options, tools, agent }` is
     MUTABLE (`SessionContext` — `packages/plugin/src/effect/session.ts:33-36`;
     fired per kind at `model-request.ts:422-431`).
   - Then `boundImages(unsupportedParts(shaped.messages, ...))` (297).
   - `LLM.request(...)` (279-302); **`session` hook `model.request`** can rewrite
     `baseURL`/`headers` (305-320); **`http.request`/`http.response`** hooks rewrite
     the wire request/response (337-364); experimental WS frame hooks (368-392).

**U2 per-message filter ON v2 — candidate insertion points:**
- **Plugin (recommended): `ctx.session.hook("context", cb)`** — fires for every
  primary model request, `event.messages` is a mutable `Array<Message>`; filter/
  replace entries before `LLM.request`. This is the official v2 destination for v1's
  `experimental.chat.messages.transform` (plugin-migration guide, hook table) and
  `chat.params`. Per-request kind, register `context`/`compaction`/`generate`/
  `title` separately (guide, lines 165-167).
- Wire-level: `http.request` (brute force, parses the provider body) — avoid.
- Fork: `toLLMMessages`/`baseTranscript` — structural; would change persistence
  semantics (filter at assembly ≠ persisted history). Not needed given the hook.

**Recommendation block (§3):** build the U2 filter as a v2 plugin hook on
`session.context` (insertion point `model-request.ts:422-431` firing before
`LLM.request` at 279-302); store filter state in the `kv` table or message
`metadata`; U3 marker compaction maps to the durable `compaction` message +
`ctx.session.hook("compaction", ...)` with a `result` override to skip the model
call (`effect/session.ts:45-48`).

## 4. Overlap audit + port map

Verdicts: native (v2 has it built in) / partial (surface exists, logic is ours) /
absent (custom logic, port as a v2 tool/plugin). Effort S/M/L; order = suggested
port sequence (intercept first per maintainer).

| Feature (our file) | v2 equivalent (pointer) | Verdict | Effort | Order |
|---|---|---|---|---|
| intercept observer (`.opencode/plugin/intercept_observer.ts`) | `ctx.tool.hook("execute.before")` mutable-arg hook (§0; `core/src/plugin/tool-input-repair.ts:21-36`) | native surface, custom logic | M (scope-cut shrinks it) | 1 |
| auto-resume (`.opencode/plugin/auto_resume.ts`) | CLI `--session`/`--continue` (`cli/src/commands/handlers/run.ts:18-19`); `ctx.event.subscribe()` on `session.idle` + `ctx.session.prompt` (plugin guide hook table, event example) | partial | M | 2 |
| ctx gauge + injected ctx line (`.opencode/tools/ctx_gauge.ts` + nudge injection) | `GET /api/session/:sessionID/context` (messages after last compaction, `protocol/src/groups/session.ts:569-579`); no native injected line | partial | M | 3 |
| nudge ladder (`compact_budget.json` nudge1-4) | none (v2 auto-compaction `buffer` is a different concept, `compaction.ts:43-44`) | absent | M | 3 |
| compact_memory (`.opencode/plugin/compact_memory.ts`) | manual compaction = inbox item + `ctx.session.compact` (`session.ts:163` domain pick; `compaction.ts:56-57`); queue-relay ≈ `session_inbox` user items (`schema.gen.ts:148-158`, `session/inbox.ts:1-33`); budgeted self/cross logic: ours | partial | L | 4 |
| context_trim (`.opencode/tools/context_trim.ts`) | revert stage/commit (`session/revert.ts:21-84`; `session.revert.commit` endpoint, `protocol/groups/session.ts:560-566`); context endpoint; no `tail_start_id` in v2 | partial | M | 4 |
| U3 marker compaction (compact_memory marker mode) | durable `compaction` checkpoint message (`to-llm-message.ts:306-325`) + `session.hook("compaction")` `result` override (`effect/session.ts:45-48`) | partial | M | 5 |
| ctx watchdog (`.opencode/plugin/ctx_watchdog.ts`) | none (no native per-step gauge/nudge) | absent | M | 3 |
| loop_log (`.opencode/tools/loop_log.ts`) | none (custom tool → `ctx.tool.transform` `editor.add`) | absent | S | 6 |
| block_transfer (`.opencode/tools/block_transfer.ts`) | none (custom tool; our own field names unchanged) | absent | S | 6 |
| state_snapshot (`.opencode/tools/state_snapshot.ts`) | none (custom tool) | absent | S | 7 |
| submit (`.opencode/tools/submit.ts`) | none (custom tool) | absent | S | 7 |
| session_info (`.opencode/tools/session_info.ts`) | native session API (`ctx.session` domain get/list) | native | S | 8 |
| dev_get_tool_context_contents (`.opencode/tools/dev_get_tool_context_contents.ts`) | none (dev helper) | absent | S | 8 |

Structural notes:
- **All 8 `.opencode/tools/*.ts` must move into plugins**: v2 discovers plugins from
  `.opencode/plugin/` and `.opencode/plugins/` only
  (`packages/core/src/plugin/source-directory.ts:5,12-17`); v1 `tool()` helper files
  under `.opencode/tools/` do not load (plugin-migration guide: V1 implementations do
  not run in V2; custom tools go through `ctx.tool.transform`, guide "Migrate custom
  tools").
- v1 plugin shape `(input, options) => ({hooks})` → v2
  `Plugin.define({ id, setup(ctx) })` (`packages/plugin/src/promise/plugin.ts:59-66`);
  the loader accepts effect (`"effect" in value`) or promise plugins
  (`packages/core/src/plugin/module.ts:134`).
- Tool-name renames hit our permission maps and intercept field names:
  `bash`→`shell`, `task`→`subagent`, `write`/`patch`→`edit` (migrate-v1 docs);
  `filePath`→`path` (§0).

**Recommendation block (§4):** port order = 1 intercept (§0), 2 auto-resume (the
autorun loop depends on it), 3 the context family as one plugin (gauge read from the
native context endpoint + injected ctx line + nudge ladder + watchdog), 4
compact_memory + context_trim + U3 (they share the compaction surfaces), 5 the small
tools in one multi-tool plugin.

## 5. Config carry-over (our actual files)

V2 reads the same locations (migrate-v1 docs "Use your existing configuration";
`packages/core/src/config/discovery.ts:12,61-88` — `opencode.json`/`opencode.jsonc`
at system → global `~/.config/opencode` → project root/`.opencode/`). V1 fields are
normalized in memory; native V2 shape is optional.

Field-by-field delta for root `opencode.jsonc`:

| v1 field (ours) | v2 native | Notes |
|---|---|---|
| `autoupdate: false` | `update: "disable"` | mapping `false→disable, "notify"→notify, true→auto` (migrate-v1 docs; `cli/src/services/updater.ts:130-136`; `schema/src/config.ts:39-41`) |
| `formatter: true` | same | "keeps the same shape" (docs); `formatter` field `schema/src/config.ts:70-72` |
| `scroll_speed`, `scroll_acceleration`, `cursor`, `mouse`, `attention`, `diff_style` | → global `~/.config/opencode/cli.json` | TUI config moves from layered tui.json to one global cli.json, auto-migrated at first V2 terminal start (docs "Terminal client configuration"); `cli/src/config/migrate.ts:196-215` (scroll), `220-235` (attention/diffs) |
| `permission.external_directory` roots | `permissions: [{ action: "external_directory", resource, effect: "allow" }]` | ordered ruleset, last match wins (`schema/src/permission.ts:71-81`; `core/src/tool.ts:292-295` `findLast`); action name unchanged (`core/src/file-access.ts:26,117,136`) |
| `references` | same shape | already plural; `ConfigReference` Local `{path, description}` (`schema/src/config/reference.ts:14-19`, `config.ts:97-99`) |
| `watcher.ignore` | `watcher` | keeps same shape (docs) |
| `snapshots: false` | same | `schema/src/config.ts:64-66` |
| `compaction: { auto:false, keep:{system:true, tokens:40000}, buffer:2000 }` | `compaction: { auto:false, keep:{tokens:40000}, buffer:2000 }` | `keep.system`/`keep.messages` have no v2 equivalent (`schema/src/config/compaction.ts` Keep = `tokens` only) — matches our 2026-09-25 config comment |
| `shell: "C:/Program Files/Git/bin/bash.exe"` | same | `schema/src/config.ts:30-32` |
| `provider` (singular) | `providers` | `npm`→`package` with `aisdk:` prefix; `options.baseURL`→`settings.baseURL` (docs "Providers"; `schema/src/config/provider.ts` `Info`: `package`, `settings`, `models`) |
| `subagent_depth: 2` | `experimental.subagent_depth: 2` | top-level is explicitly unsupported (`core/src/config/normalize.ts:45,451-458`; `schema/src/config/experimental.ts:9-12`) |
| `default_agent` | same | `schema/src/config.ts:36-38` |
| `agent` (singular) | `agents` | `prompt`→`system`; `temperature`→`request.body`; `permission` (object)→`permissions` (ordered array); `mode: "all"` and `color` unchanged (`schema/src/config/agent.ts:11-24`; docs "Agents and modes") |
| (none — we rely on directory discovery) | `plugin`→`plugins` | config key only matters for package/enable directives; local discovery is directory-based (§4 note) |

Agent prompt templates: our `system`/`prompt` values use `{file:./agent/prompts/...}`
interpolation. The docs rename `prompt`→`system` but do NOT confirm `{file:}` template
support in the native `system` field — **open item: verify during porting** (expected
to work since V1 syntax stays accepted, but test).

AGENTS.md: unchanged — v2 discovers the ambient `AGENTS.md` (global
`~/.config/opencode/AGENTS.md` + up-to-home from cwd; no CLAUDE.md fallback) (docs
"Instruction files").

**Recommendation block (§5):** keep V1 syntax as-is for now (it loads), convert in
place when porting: `provider`→`providers` (aisdk package + settings),
`subagent_depth`→`experimental`, `agent`→`agents` with `permissions` arrays,
`compaction.keep` trimmed to `tokens`, `autoupdate`→`update: "disable"`; TUI fields
auto-migrate to `cli.json`. Verify the `{file:}` agent prompt templates after the
`prompt`→`system` rename.

## 6. Upgrade path + risks

- **Side-by-side today (measured):** npm global v1 (`opencode-ai`, v1.19.0 generation)
  + standalone v2 `opencode.exe` v2.0.29 in `C:\Users\Wasiejen\Repos\opencode-agent-v2`.
  The docs state V1 and V2 "are no longer installed side by side by default. Remove a
  package-managed V1 installation before installing V2" (migrate-v1 docs "Install V2").
  The v2 clone keeps working as-is until we replace the npm install.
- **`opencode upgrade` exists in v2** (`cli/src/commands/handlers/upgrade.ts`) with
  installation-method detection; the updater checks at CLI startup and acts per the
  `update` policy (`cli/src/services/updater.ts`, `index.ts:118` `Updater.layer`).
  **Nuance vs the spec's "no auto-updates in v2":** the policy IS in the code —
  `"disable" | "notify" | "auto"` (`schema/src/config.ts:39-41`,
  `cli/src/services/updater-action.ts:1-11`) and the default when unset is `"notify"`
  (`updater.ts:171`). So: no auto-INSTALL by default (only notify); auto-install
  happens only if we set `update: "auto"`. Our `autoupdate: false` → `"disable"`
  keeps everything manual.
- **Live-DB migration timing:** first v2 service start on the default data dir opens
  our live `opencode.db`, bootstraps v2 tables in it, and auto-migrates (§2). The
  rewrite is one-way and clears the v2 `event` table. Mitigation: dry-run on a copy
  first (`OPENCODE_DB=<copy>.db`), back up the real file, then start v2 deliberately.
- **llama-swap serial-session discipline across two workspaces:** v2 adds auxiliary
  model traffic (title/compaction/generate/warming — §1). With v1 and v2 workspaces
  coexisting during the transition, keep at most one session live per workspace and
  never run a v2 session while a v1 session is mid-step on the shared backend.
- **AGENTS.md/agents carry-over:** ambient AGENTS.md discovery continues (§5); our
  agent definitions live in `opencode.jsonc` (not `.opencode/agents/` files), so the
  carry-over is the `agent`→`agents` field mapping (§5).
- **Ordered-permissions model:** v1 per-action objects → v2 flat ordered
  `permissions` array with last-match-wins (`schema/src/permission.ts:71-81`;
  `core/src/tool.ts:292-295`). Our deny-then-allow lists must be RE-ORDERED as
  arrays (order now encodes precedence).
- **Plugin loading errors:** our two known v1 load errors (repo_overview.md safety
  limits) become moot — all 6 plugins must be rewritten for the v2 API anyway.

**Recommendation block (§6):** upgrade path = (1) dry-run migration on a DB copy,
(2) port plugins (§4 order), (3) convert config (§5), (4) remove the npm v1 install
and install/replace with v2 (standalone or npm, per docs), (5) keep `update:
"disable"`, `warming` off, sessions serial. Do NOT start the v2 service against the
live data dir until the copy dry-run passes.

## 7. Recommendations

1. **Port order.** Intercept first (the maintainer's first port target; §0 proves it is
   plugin-only via `ctx.tool.hook("execute.before")`, scope-cut confirmed against the
   header incl. 3c). Then auto-resume (the loop depends on it), the context family
   (gauge + injected line + nudge + watchdog as one plugin), compact_memory +
   context_trim + U3 (shared compaction surfaces), then the six small tools in one
   multi-tool plugin (v2 does not load `.opencode/tools/` — §4).
2. **U2-on-v2 design.** Plugin, not fork: `ctx.session.hook("context", ...)` mutating
   `event.messages` per primary model request (insertion point
   `core/src/session/model-request.ts:422-431` before `LLM.request` at 279-302).
   Filter state in the `kv` table / message `metadata`. The fork is only needed if we
   later want filtered-vs-persisted divergence — the hook already gives
   request-time filtering with history intact.
3. **Upgrade timing.** Validate v2 against a COPIED data dir first (dry-run the v1→v2
   in-place migration, §2), port the intercept + auto-resume, run a full autorun
   cycle under v2, then retire v1 (remove the npm install per the docs). Keep v1 as
   the fallback until that cycle is green.
4. **Retire from v1.** npm v1 install, v1 plugin implementations (archive under
   `.opencode/deactivated/` after porting), TUI tui.json fields (auto-migrated to
   `cli.json`), `subagent_depth`/`keep.system`/`keep.messages` (no v2 behavior), and
   the `tail_start_id`-based tooling (rebuild on `session_v2`/`session_message` +
   revert).
