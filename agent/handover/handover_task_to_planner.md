# Worker summary — #136 OpenCode v2 impact research (explorer, ses_ebf38ec2cffeXAREzUwnlEjXpo, 2026-10-15)

Status: DONE — research doc complete, all 8 sections, §0 priority answered.

## Deliverable
`agent/research/2026-10-15_opencode-v2-impact.md` (committed on `main`, this repo).
Read-only everywhere else: fork `C:\Users\Wasiejen\Repos\opencode` (`production`,
654b79b27e), v2 clone, live DB, `maintainer/`, `opencode.jsonc` untouched. No live v2
run, no backend requests, no installs, no pushes.

## Key findings per section
- **§0 (priority) — arg-mutation hook: YES.** v2 plugin API
  `ctx.tool.hook("execute.before", cb)` receives a MUTABLE `event.input`
  (promise API `packages/plugin/src/promise/tool.ts:38-46`; the core executes with the
  post-hook `event.input`, `core/src/tool.ts:271-280`; shipped proof plugin
  `core/src/plugin/tool-input-repair.ts:21-36` reassigns `event.input`). The hook can
  also throw `Tool.Error` to reject the call (the (4b) hard-stop surface) and can
  rename/redirect via mutable `event.tool`. Fuzzy resolution plugs in as a **pure
  plugin, no fork change** — `execute.before` (recommended) or `ctx.tool.transform`
  `update` (wrap the executor). v2 read/write/edit take `path`, not v1 `filePath`.
  Scope cut confirmed against the plugin header: (3c) git-refs-in-bash IS a
  pair-based channel (header lines 60-71), so dropping it with the pair channels is
  consistent — the spec's assumption holds.
- **§1 — architecture.** Shared background-service model: CLI modes
  default/service/stdio (`cli/src/server-process.ts:19-23`), incumbent detection
  (86-91), service owns DB + sessions + provider traffic; TUI is a thin client.
  Providers called in-process via `packages/ai` protocol drivers (openai-compatible =
  our llama-swap path); no provider subprocesses. `packages/llm` is empty in this
  checkout (measured). New single-slot risks: auxiliary model calls (title,
  compaction summarize, generate) + opt-in `warming` keep-alive requests
  (`core/src/plugin/warming.ts:6-57`) — keep `warming` off, sessions serial.
- **§2 — DB.** v2 schema: `session_v2`, `session_message` (JSON `data` blob —
  message-level blobs SURVIVE, re-encoded from v1 message+part), `kv`, `event`,
  `session_inbox`, `session_pending`; no `part` table, no `tail_start_id` in v2.
  Migration (`core/src/database/v1-migration.bun.ts`) reads v1 `session`/`message`/
  `part` and writes v2 tables **in the SAME FILE**; auto-runs at v2 service startup
  (`server/src/routes.ts:189`); resumable via `kv` key `migration.v1-v2`; one-way;
  clears the v2 `event` table first; imports a previous `opencode-next.db` if present.
  DB path: v2 opens `opencode.db` in `~/.local/share/opencode` — MEASURED: exactly
  our live v1 DB location. First v2 service start = in-place rewrite of the live DB.
- **§3 — compaction & assembly.** `SessionCompaction` settings `auto/buffer/keep.tokens`
  (defaults auto=true, keep=15k; `config/plugin/compaction.ts:15-24`); compaction
  result = durable `compaction` checkpoint message rendered as
  `<conversation-checkpoint>` (`runner/to-llm-message.ts:306-325`). Assembly:
  history → `toLLMMessages` → `baseTranscript` → `prepare` fires
  `hooks.trigger("session", "context"|...)` with a MUTABLE draft
  (`session/model-request.ts:422-431`) before `LLM.request` (279-302); wire-level
  `model.request`/`http.request`/`http.response` hooks downstream. **U2 insertion
  point ON v2: plugin hook `ctx.session.hook("context", ...)` mutating
  `event.messages` — no fork needed** (official v2 destination of v1
  `experimental.chat.messages.transform`). U3 maps to the checkpoint message +
  `session.hook("compaction")` `result` override (skip the model call).
- **§4 — overlap audit.** Table in the doc (feature → pointer → verdict → effort →
  order). Native: session_info. Partial: intercept (native hook surface), auto-resume
  (CLI `--session`/`--continue` + `session.idle` event), ctx gauge (native context
  endpoint `GET /api/session/:id/context`), compact_memory (manual-compaction inbox +
  `ctx.session.compact`), context_trim (revert stage/commit + context endpoint), U3.
  Absent: nudge ladder, ctx watchdog, loop_log, block_transfer, state_snapshot,
  submit, dev_get_tool_context_contents (port as v2 tools via
  `ctx.tool.transform`). Structural: `.opencode/tools/*.ts` is NOT a v2 discovery
  path (v2 loads `.opencode/plugin(s)/` only) — all 8 tools move into plugins.
- **§5 — config carry-over.** Field-by-field delta table in the doc. Main deltas:
  `autoupdate: false`→`update: "disable"`; TUI fields (scroll/cursor/mouse/attention/
  diff_style) → global `~/.config/opencode/cli.json` (auto-migrated);
  `permission` object → ordered `permissions` array (last match wins);
  `agent`→`agents` (`prompt`→`system`, `temperature`→`request.body`);
  `provider`→`providers` (`npm`→`package: "aisdk:@ai-sdk/openai-compatible"`,
  `options.baseURL`→`settings.baseURL`); `subagent_depth`→`experimental.subagent_depth`;
  `compaction.keep` → tokens-only (`keep.system`/`keep.messages` gone);
  `references`/`watcher`/`snapshots`/`shell`/`default_agent` unchanged. Open item:
  verify `{file:}` agent prompt templates under the `prompt`→`system` rename.
- **§6 — upgrade path.** Side-by-side today (npm v1 + standalone v2 exe in the
  clone); docs say remove package-managed v1 before installing v2. `opencode upgrade`
  exists in v2; nuance: `update` policy IS in code (`disable|notify|auto`, default
  `notify`) — no auto-INSTALL by default; our `"disable"` keeps it manual.
  Migration timing = first v2 service start on the live data dir; mitigations: dry-run
  on a DB copy (`OPENCODE_DB=<copy>.db`), back up, deliberate first start. Serial-
  session discipline must extend to v2's auxiliary model calls.
- **§7 — recommendations.** Port order: intercept → auto-resume → context family →
  compact_memory/context_trim/U3 → small tools. U2 on v2 = plugin `context` hook
  (no fork). Upgrade timing: validate on a copied data dir + full autorun cycle, then
  retire v1. Retire: npm v1 install, v1 plugin implementations, tui.json fields,
  `subagent_depth`/`keep.system`/`keep.messages`, `tail_start_id` tooling.

## Recommendation block (the one)
1. Port the intercept FIRST as a v2 plugin on `ctx.tool.hook("execute.before")`
   (arg mutation confirmed yes; scope cut incl. 3c confirmed against the header).
2. Design the U2 per-message filter as a v2 plugin hook on `session.context`
   (`model-request.ts:422-431`, before `LLM.request`) — no fork change.
3. Upgrade timing: dry-run the v1→v2 in-place migration on a copied `opencode.db`,
   port intercept + auto-resume, run a green autorun cycle on v2, then remove the
   npm v1 install; keep `update: "disable"`, `warming` off, sessions serial.

## Verification
- No build/test gate applies (research task, no code changes).
- Working-tree status at commit: `dirty` (only the two new/changed files staged;
  pre-existing untracked `maintainer/inbox_planner/migration_hints_from_perplexity_ai.md`
  left uncommitted, not mine).
- Commit: see hash below (this file was committed with the research doc).

## TODO / inbox entries
- None appended (no discrepancies requiring curation; the spec's "no auto-updates in
  v2" claim is nuanced in §6 but is a maintainer-stated fact, not a repo discrepancy).

## Deliberately omitted work
- No live v2 run (per spec) — v2 runtime behavior (e.g. whether `warming`/title
  calls actually fire in a TUI session, exact `cli.json` migration contents) is
  code-read only.
- `dev` branch diff not performed (symbols found where expected on `production`).
- `{file:}` prompt-template support on v2 `system` not verified (no live run allowed) —
  flagged as open item in §5.

## Lessons
- v2's hook system is uniformly "one mutable event object in, same object out"
  (`core/src/plugin/hooks.ts:88-95`) — every v1 `(input, output)` hook ports to a
  single-event v2 hook; the built-in `tool-input-repair` plugin is the reference
  example for arg-mutation plugins.
