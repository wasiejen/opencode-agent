# v2 plugin harness + config discovery (Unit A, TODO #137, 2026-10-16)

Facts measured landing the v2 intercept-plugin port in the clone
`C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch `migration`,
standalone opencode v2.0.29 in its root). Consult when building/typechecking
ANY plugin against the fork or the v2 line, or wiring v2 plugin config.

## Typechecking a file that imports `@opencode/plugin` against the fork
- **Do:** the harness tsconfig needs `typeRoots` pointing at the fork's
  installed `packages/plugin/node_modules/@types` + `types: ["node"]`,
  plus a `paths` mapping `@opencode/plugin` →
  `C:/Users/Wasiejen/Repos/opencode/packages/plugin/src/promise/index.ts`.
  The fork's checker is tsgo (`node_modules/.bin/tsgo.exe`); tsgo REJECTS
  the removed `baseUrl` option (TS5102) — use `paths`, not `baseUrl`.
  Committed reference harness: clone
  `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\tests\`.
- **Why (evidence):** Unit A (TODO #137) worker (ses_ebea1bd3c…, 2026-10-16_01-03).
  Without @types/node in the program, tsgo/tsc report TS7017 at fork
  `packages/ai/src/protocols/shared.ts:408` (`globalThis.process`) — a
  HARNESS artifact, not a fork bug; chasing it wastes a worker session.
- **Ref:** knowledge_inbox entry 2026-10-16_01-03 (cured here); clone
  `.opencode/plugins/tests/` (tsconfig + smokes committed on `migration`).
- **Keys:** @opencode/plugin, tsconfig, typeRoots, paths, tsgo, TS5102, TS7017, baseUrl.

## v2 `plugins` config entries pointing at a FILE are dropped
- **Do:** in v2, a `plugins` config entry whose path is a FILE is dropped
  with a warning ("configured plugin path must be directory" — fork
  `packages/core/src/config/plugin/source.ts:151-153`). v2 plugins load via
  DIRECTORY auto-discovery of `.opencode/plugins/`. A `./plugins/intercept_observer.ts`
  entry in a v2 `opencode.jsonc` is therefore DOCUMENTARY ONLY — decide per
  clone: keep the entry as documentation or drop it (the v2 clone's copy
  currently KEEPS it with a comment — maintainer question raised 2026-10-21, see
  todo_inbox 2026-10-16_01-02; the clone file is maintainer domain).
- **Why (evidence):** measured in Unit A (TODO #137): the v2 intercept plugin
  loaded in the clone via `.opencode/plugins/` auto-discovery while the
  file-path `plugins` entry logged the drop warning.
- **Ref:** worker ses_ebea1bd3c… 2026-10-16_01-02 (todo_inbox entry);
  clone `opencode.jsonc` comment (Unit A); fork source pointer above.
- **Keys:** v2, plugins config, must be directory, auto-discovery,
  opencode-agent-v2, intercept_observer.

## v1 + v2-migrated global DB coexist (measured 2026-10-21, knowledge_inbox 2026-10-21_22-35)
- **Do:** treat the global DB (`~/.local/share/opencode/opencode.db`) as
  runnable AFTER a v2 migration touched it: it carries BOTH the v1 tables
  (`session`/`message`/`part`) AND `session_v2`; the v1.19.0 host starts
  and runs fine on the mixed DB. A fresh dry-run copy is NOT required
  for v1 coexistence. For v2 CLI plugin-load evidence read
  `~/.local/share/opencode/log/opencode.log` ("loading plugin" lines).
  GOTCHA: v1-style `part.session_id` filters return ZERO rows for v2
  sessions — v2 content lives in `session_message(id,session_id,type,seq,data)`
  + `part(message_id,…)`, assistant content INLINE in `message.data`.
- **Why (evidence):** the one-way v2 migration landed 2026-10-16 (worker
  live v2 runs); today's v1.19.0 restart + tool calls ran green on the
  same mixed DB (direct ses_ea02d3559ffe…).
- **Ref:** knowledge_inbox 2026-10-21_22-35 (cured here); the §3 table
  map in `opencode-plugins/host-map.md`.
- **Keys:** DB coexistence, session_v2, session_message, part.message_id,
  mixed DB, opencode.log, loading plugin.

## v2 CLI run: model selection + OPENCODE_DB (measured 2026-10-21, knowledge_inbox 2026-10-21_22-43)
- **Do:** always pass `--model <id>` to `opencode run` in the v2 line —
  the config agent's `model` key is NOT applied (sessions ran the provider
  default `flint-40b-free`, which FAKES tool calls: prints the call JSON
  as TEXT, finish=stop, zero tool parts in `session_message`; fork
  `packages/cli/src/commands/commands.ts:366`). `OPENCODE_DB` = a
  FILENAME resolved against the data dir (fork
  `packages/cli/src/database-path.ts:6`) — `export OPENCODE_DB="x.db"`
  isolates a run DB; a path value is wrong. A healthy plugin can log
  NOTHING when the run model fakes calls — a scripted acceptance run is
  valid ONLY with an explicit capable `--model`.
- **Why (evidence):** the 2026-10-21 v2 intercept-acceptance root-cause
  (3 clone v2 sessions, zero tool parts, weak default model).
- **Ref:** knowledge_inbox 2026-10-21_22-43 (cured here); clone
  `agent/scripts/v2_intercept_acceptance.sh` (fixed d63d53a).
- **Keys:** opencode run, --model, flint-40b-free, fake tool calls,
  OPENCODE_DB, filename data dir, zero tool parts.

## Clone `migration` module layout + the gauge.mjs re-export shim (2026-10-22, TODO #141 clone-gate unit, worker-14)
- **Do:** navigate the clone as: frozen v1 plugin modules under
  `.opencode/plugin-unported-v1/` (relative imports intact —
  `./scripts/gauge.mjs` resolves via a 7-line RE-EXPORT SHIM at
  `.opencode/plugin-unported-v1/scripts/gauge.mjs` → the clone gauge core at
  `.opencode/plugin/scripts/gauge.mjs`; ONE implementation, edit the core
  only); `.opencode/plugin/` still holds probes/tests/scripts/deactivated
  (the `compact_memory_v1.ts` probe path EXISTS — left as-is);
  `.opencode/plugins/intercept_observer.ts` (the v2 plugin) exports the
  DEFAULT factory ONLY — named-core pins always load the
  plugin-unported-v1 pair. The clone standard gate (probe + 12 node smokes)
  is GREEN against this layout (TODO #141 CLOSED).
- **Why (evidence):** clone `migration` `652ee6f` + `47aa082` (2026-10-22,
  planner-13 verified: probe 360/360, 12 smokes ALL PASS — 170/76/131/110/17/
  40/3/32/85/69/20/31; baseline correction: only the 4 plugin-module smokes
  were red).
- **Keys:** plugin-unported-v1, gauge.mjs shim, re-export, named-core pins,
  default factory only, clone gate green, 12 smokes.

## v2 per-message token usage storage — the gauge mass + tools-estate fact (2026-10-22, TODO #137 unit 2 research, worker ses_e9ebd604…)
- **Do:** read v2 message usage as: `session_message` rows carry a `type`
  column; the assistant `data` JSON = `{type:"assistant", finish?, rawFinish?,
  cost?, tokens?: {input, output, reasoning, cache:{read, write}}, time:
  {created, streamed?, completed?}}` (`@opencode/schema/session-message.ts`
  `Assistant` + `token-usage.ts`); FINISHED-STEP filter = the fork's own:
  `type='assistant' AND json_extract(data,'$.time.completed') IS NOT NULL
  AND json_extract(data,'$.tokens') IS NOT NULL`; per-message total =
  `TokenUsage.total()` = input+output+reasoning+cache.read+cache.write
  (llama-swap keeps reasoning/cache.write at 0); compaction rows ALSO carry
  tokens (the REQUEST usage, not the context size) — exclude by `type`. A
  migrated live DB keeps ALL FOUR tables side by side
  (`session_v2`+`session`+`session_message`+`message`) and the v1.19.0 host
  runs fine on it.
- **Why (evidence):** fork production @654b79b27e schema + migration source +
  ONE `mode=ro` live-DB read (real row: input 72 / output 176 / cache.read
  14283); pinned by the clone gauge unit-2 forms (`922f74c`/`2c1fa23`,
  gauge_core 42 pins incl. v2-shaped OK mass 44150).
- **Keys:** session_message, tokens.input/reasoning/cache, time.completed,
  TokenUsage.total, compaction-rows-excluded, four-table coexist, mass leg.

## Same-model MULTI-HOST coexist at the single llama-swap slot (2026-10-22, measured direct ses_ea02d3559…, inbox 03-26)
- **Do:** treat a second opencode host pointed at the same slot with a
  MATCHING `--model` id as SAFE — no eviction, no warm-break (the running
  session kept generating normally through the clone v2 host run); the
  eviction risk is DIFFERENT-MODEL traffic (the measured 80k different-purpose
  prefill kicked the session's model out), not multi-host traffic per se.
- **Keys:** multi-host, same-model, --model, no eviction, warm-break,
  different-model risk.

## v2 request-draft hook + injectable message shape (2026-10-22, #137 unit 3a; inbox 05-23)
- **Do:** inject via `ctx.session.hook("context", …)` — fires per PRIMARY
  model request (`core/src/session/model-request.ts:422-431`); the `messages`
  mutation is TRANSIENT (never persisted); the injectable entry type is
  `@opencode/ai` `LLM.Message` (`packages/ai/src/schema/messages.ts:237-244` —
  `role` user|assistant|system|tool required, `content: Array<ContentPart>`
  required, `id` optional) — import TYPE-ONLY + a tsconfig `paths` entry
  for `@opencode/ai`; `ctx.session.synthetic({sessionID,text,delivery:"queue"})`
  queues as the next turn at idle (types.ts:4592-4610); `execute.after` events
  carry `sessionID` (per-session reads possible); ONLY `execute.before` may
  reject (`core/src/plugin/hooks.ts:23`); the v2 delegation tool `subagent`
  input has `prompt: string` (`core/src/tool/plugin/subagent.ts:35`).
- **Why (evidence):** clone `plugins/context_family.ts` + tsgo-verified 2026-10-22.
- **Keys:** session.hook context, model-request, transient messages,
  LLM.Message, synthetic queue, execute.after sessionID, subagent prompt.

## v2 live DB table pair + sqlite/CLI surface facts (2026-10-22, clone dump twin 4c; inbox 10-24)
- **Do:** read v2 rows from the `session_v2` ↔ `session_message` pair (the
  v1 `session`/`message` pair does NOT serve v2 reads — a twin lacking the
  pair exits 2); `session_v2.model` is a JSON string (unwrap);
  `session_message.data` carries content INLINE — assistant `content[]`
  items {type:text|reasoning|tool,…} (NO part table in v2), user rows
  text+files, compaction rows status/reason/summary/recent. bun 1.4.3
  `bun:sqlite` exposes prepare/query (NO `db.all` sugar). `execFileSync`
  failures carry stderr as a Buffer whose message already embeds it —
  keep `String(buffer)` coercion; ONE-LINE the whole detail for one-line-per-event
  ctx.log contracts.
- **Why (evidence):** clone `agent/scripts/db/dump_session_v2.mjs` fixture
  runs, 2026-10-22 (unit 4c).
- **Keys:** session_v2, session_message, no part table, model JSON string,
  bun:sqlite prepare/query, execFileSync stderr Buffer.

## The clone's inert v1 tool twins PREDATE MAIN hardening — anchor ports on the MAIN twin (2026-10-22, planner-19; inbox 11-14)
- **Do:** before anchoring a v2 port spec on a cross-tree design-of-record
  file, run the twin check (`cmp -s` + `wc -l` + bounded `diff | head -60`).
  STALE clone copies measured 2026-10-22: `.opencode/tools/submit.ts` 167
  lines vs MAIN 189 (the 2026-10-21 NEWLINE GUARD missing), `block_transfer.ts`
  845 vs MAIN 1034 (333 diff lines; MAIN carries the hardening items incl.
  the item-10 POSIX temp-root block at MAIN `:121-133`).
  BYTE-IDENTICAL both trees: `loop_log.ts` 235, `ctx_gauge.ts` 36,
  `state_snapshot.ts` 206, `dev_get_tool_context_contents.ts` 59,
  `context_trim.ts` 1168. The clone's v1 smokes (bt 132/132, submit 31/31)
  pin the STALE behavior.
- **Why (evidence):** the planner-19 twin sweep (`cmp`/`wc -l`/`diff`);
  TODO #143 tracks the refresh-vs-document decision.
- **Keys:** twin freshness, stale clone twin, MAIN-twin anchor, #143,
  newline guard, POSIX temp-root block.

## v2 plugin sandbox smokes: the scratchpad IS a live ALLOW ROOT — fixture repos sit OUTSIDE it (worker_FlashNext, knowledge_inbox 2026-10-22_14-55)
- **Do:** put deny-matrix fixture configs and plugin-copy fixture repos in the
  DIRECT CHILDREN of the parent Temp dir (outside `C:/Users/Wasiejen/AppData/Local/Temp/opencode`), never inside the scratchpad root itself.
- **Why (measured, unit 5c clone sandbox smoke):** the scratchpad is an allow
  root of the live `opencode.jsonc`, so a fixture inside it can never
  demonstrate DENY; the SELF-set TEMP/TMP fixture-env mechanism makes
  `os.tmpdir()==TEMP` dedupe to ONE temp root on Windows.
- **Keys:** allow root, scratchpad, deny-matrix, fixture repo, TEMP dedupe, 5c.

## v2 smoke root-carrier trap — the setup-captured repo root is MODULE-level and re-assigned (worker_FlashNext, knowledge_inbox 2026-10-22_18-48)
- **Do:** keep each fixture's pins INSIDE its own fixture block (before any
  later `plugin.setup(...)`), or re-setup that fixture before its `execute`
  (which then breaks the smoke's `DISPOSED` count pin).
- **Why (measured, clone `migration`):** `.opencode/plugins/state_snapshot.ts`
  captures the repo root in a MODULE-level `let dir = ctx.location.directory`
  reassigned at EVERY setup — after a later setup(ROOT2/ROOT3), an `execute`
  of the FIRST fixture's registered tool reads the LAST setup's root
  (an FX1 pin appended after FX2/FX3 saw fx3's markers `[-1,-1]`).
- **Keys:** root carrier, setup reassignment, fixture block order, DISPOSED pin.
