# knowledge_plugins.md — opencode plugins (`.opencode/plugin/`)

Gained, verified knowledge for opencode plugins. Format per the README:
**Do** / **Why (evidence)** / **Ref** / **Keys**.

## Plugin context ≠ custom-tool context
- **Do:** a PLUGIN receives a context exposing session operations directly
  (e.g. `ctx.session.get({sessionID})`, `ctx.session.context({sessionID})`);
  a CUSTOM TOOL receives `Tool.Context` (`sessionID` / `messageID` / `agent`).
  Do not conflate the two shapes.
- **Why (evidence):** the current opencode docs describe plugin vs custom-tool
  context differently; using the wrong shape is the common trap.
- **Ref:** `maintainer/done/knowledge_opencode_tools_plugins.md`
  §1 + §4.
- **Keys:** plugin context, Tool.Context, ctx.session, get, context, v2.

## Compaction hooks (prefer over manual triggering for state preservation)
- **Do:** to preserve durable project state across compaction, use the
  `experimental.session.compacting` hook (inject context that survives
  compaction) and `experimental.compaction.autocontinue` for post-compaction
  continuation. Trigger compaction from a tool only when the agent must
  checkpoint at a specific point.
- **Why (evidence):** both hooks are present in the installed plugin package
  (grep-verified 2026-09-12); the docs recommend the hook for "information
  that should survive compaction."
- **Ref:** `.opencode/node_modules/@opencode-ai/plugin/dist/index.d.ts`
  (~L277-296); knowledge doc §4.
- **Keys:** experimental.session.compacting, autocontinue, hook, compaction,
  durable, prompt.

## The shared gauge core is ONE implementation
- **Do:** for context-usage reads, wrap `.opencode/plugin/scripts/gauge.mjs`
  (`readGauge` / `formatGauge`) — do NOT re-derive the backend chain / window
  rule / readout form. `peek.mjs` and the `ctx_gauge` tool both wrap the same
  core.
- **Why (evidence):** a single shared implementation keeps readouts
  byte-identical across surfaces (smoke-verified vs peek.mjs).
- **Ref:** `ctx_gauge.ts` (imports gauge.mjs); de-peek build (T1); loop-tool-
  batch part 2.
- **Keys:** gauge.mjs, readGauge, formatGauge, peek, byte-identical, readout.

## Production host lacks `node:sqlite`
- **Do:** for cross-process / persistent state in a plugin or tool, use an
  fs/JSON state store (e.g. the `compact_budget.json` pattern under
  `.opencode/temp/`), not `node:sqlite`.
- **Why (evidence):** the production plugin host does not expose
  `node:sqlite` (TODO #37, closed with this finding); a JSON file shared
  in-process is the portable mechanism.
- **Ref:** TODO #37; `compact_memory.ts` budget store.
- **Keys:** node:sqlite, unavailable, JSON store, compact_budget, cross-process,
  temp.

## Plugin scripts run under Node; the probe is the gate
- **Do:** run plugin scripts with Node; verify changes against
  `handover_probe.mjs` (the probe) — keep it green, APPEND-only checks.
- **Why (evidence):** the probe is the shared green gate for plugin/tool
  changes; renumbering is forbidden.
- **Ref:** `repo_commands.md`; `handover_probe.mjs`.
- **Keys:** node, probe, handover_probe.mjs, green gate, append-only.

## Plugins can register tools; client-needing tools belong HERE
- **Do:** a custom tool (`.opencode/tools/*.ts`) does NOT get `context.client`
  (by design — see `knowledge_tools.md`). If a tool needs the SDK client /
  RPC access (e.g. `session.compact`), register it FROM a plugin
  (`.opencode/plugin/*.ts`): the plugin function receives `ctx` which
  carries client/RPC access, and a plugin can register custom tools it
  exposes to agents. The registered tool's `execute` still receives
  sessionID / agent.
- **Why (evidence):** maintainer Q&A (2026-09-12,
  `maintainer/done/plugin_exposed_custom_tool.md`): the
  custom-tool context is intentionally limited (no client); a custom tool
  CANNOT "start"/obtain a plugin context, but a plugin CAN register tools —
  "Do not use a separate `.opencode/tools/...` tool if it requires
  `context.client`; register that tool from `.opencode/plugins/...`
  instead." A spawned Node process is NOT a plugin context (would need its
  own server connection) — avoid that workaround.
- **Ref:** `maintainer/done/plugin_exposed_custom_tool.md`;
  `get_context_keys` key dump (clientKeys empty); TODO #52.
- **Keys:** plugin, register tool, context.client, RPC, SDK access,
  .opencode/tools, .opencode/plugin, compact, ctx.

## The plugin ctx client on THIS host (probe-verified) + detection gotchas
- **Do:** reach the SDK client via the CAPTURED plugin ctx
  (`ctx.client.session.*`). Detect methods with `typeof` — `Object.keys`
  MISSES prototype methods (`Object.keys(client.session)` returns only
  `["_client"]`). On this build `summarize` is a function and `compact` is
  UNDEFINED (v1-generation client). A tool-exposing plugin registers ONLY
  via the live `opencode.jsonc` `plugins` array — a dropped-in file without
  the config entry does not register.
- **Why (evidence):** `dev_probe_ctx.ts` probe (2026-09-12, output in its
  header comment): pluginCtxKeys = client, project, worktree, directory,
  experimental_workspace, serverUrl, $; client exposes 21 keys (session,
  app, event, config, ...); summarizeType=function, compactType=undefined;
  the tool's execute context inside the plugin has NO client (same 11 keys
  as the custom-tool context).
- **Ref:** `.opencode/plugin/dev_probe_ctx.ts`; TODO #52;
  `proposals/2026-09-12_compact_memory_plugin.md`.
- **Keys:** plugin ctx, client, summarize, compact, v1, v2, prototype,
  typeof, plugins array, registration, dev_probe_ctx.

## A plugin can REPLACE the compaction prompt (shape the resume context)
- **Do:** to control what a compaction produces, use the
  `experimental.session.compacting` hook and set `output.prompt` (replaces
  the default compaction prompt ENTIRELY). This is a prompt-level lever —
  it does NOT trigger compaction; the host does that. Useful to bias the
  resume prompt toward durable project state (task status, files touched,
  blockers, next steps) for the multi-agent swarm.
- **Why (evidence):** the installed plugin types expose
  `experimental.session.compacting` (dist L277-296) whose doc says
  `prompt` "replaces the default compaction prompt entirely". A maintainer
  WIP (`custom_compaction.ts`, 2026-09-12) uses it to inject a swarm-oriented
  resume prompt.
- **Ref:** `.opencode/node_modules/@opencode-ai/plugin/dist/index.d.ts`
  (~L277-296); maintainer WIP `custom_compaction.ts` (uncommitted);
  `maintainer/done/plugin_exposed_custom_tool.md`; TODO #52.
- **Keys:** experimental.session.compacting, output.prompt, compaction,
  resume prompt, swarm, durable state.

## compact_memory verified working again + Gemma is the DEFAULT compaction model
- **Do:** compact_memory is back to a working state — use the cross-session
  contract as documented (SELF sync / CROSS fire-and-forget; success = the
  COMPACT line in `.opencode/temp/ctx.log` + the budget increment). The
  DEFAULT compaction model is now Gemma (`opencode.jsonc`
  `agent.compaction.model` = `llama-swap/Gemma4-12B-Q4KXL-MTP-128K`, set
  2026-09-15) → the default cross-compact runs on a DIFFERENT model → no
  flush budget. The summarize body STILL requires `providerID` + `modelID`
  (server payload schema) — pass the explicit pair for cross compaction
  rather than relying on the auto-resolve (the plugin REFUSES to send when
  the pair cannot be resolved).
- **Why (evidence):** maintainer short test 2026-09-15 — cross-session
  compact of `ses_f5dedec39ffeYhwWJNGbjHa4U0` "triggered as expected":
  COMPACT line in `.opencode/temp/ctx.log` 00:38 (tokens=30000 messages=12),
  the target's next read shows 0%/119K (post-compact reset).
- **Ref:** `opencode.jsonc` L12-20 (top-level compaction: auto=false) +
  L118-120 (agent.compaction.model); `compact_memory.ts` L37 / L281 /
  L517-524; `.opencode/temp/ctx.log` 2026-09-15; planner verification
  (2026-09-15, direct session).
- **Keys:** compact_memory, gemma default, agent.compaction.model,
  providerID, modelID, summarize, COMPACT line, verified working.
- **Correction (2026-09-24, prompt wave task d):** `agent.compaction.model`
  is now COMMENTED OUT in `opencode.jsonc` → the default compaction
  summarizer is the SAME model as the session's model (the "Gemma is the
  DEFAULT compaction model" claim above is stale; a default cross-compact
  now runs on the target's own model). Factual source: `maintainer/draft/
  compaction_guide/full_guide.md` §12 Corrections.

## Spawning a node script from a plugin: execPath is NOT a node runtime on the live host
- **Do:** when a plugin `execFileSync`s a node script, resolve the executable
  with `resolveNodeExe()` (exported from `compact_memory.ts`): basename
  (lower-cased) starts with `node` → pass through; otherwise the literal
  `"node"` (PATH resolution — on Windows PATHEXT finds node.exe).
- **Why (evidence):** on the live opencode host `process.execPath` is the CLI
  binary (`opencode.exe`), not a node runtime — spawning the dump script with
  it ran `opencode.exe dump_session.cjs ...` (the CLI printed its help, the
  dump failed, a WARNING rode every dispatch); the 2026-09-15 live #55 test
  shows exactly this (DUMP-FAIL line + CLI help text); plain-node smokes/probe
  could not catch it (their execPath IS a node runtime).
- **Ref:** `compact_memory.ts` `resolveNodeExe` + the `preCompactionDump`
  spawn site (commit 9fd7557, TODO #55 note 2026-09-16); plan7 spec
  `#57-live`.
- **Keys:** execPath, resolveNodeExe, execFileSync, opencode.exe, dump hook,
  spawn, PATH, PATHEXT.

## Stored tool-call args are POST-mutation — you only see the corrected args
- **Do:** NEVER trust the tool-call args you see in your own context / the
  session dump to diagnose a bitdrift or fuzzy-resolve — the DB stores the
  POST-mutation `state.input` (the corrected path), not what you EMITTED. To
  know what you actually emitted, read `.opencode/temp/intercept.log` (the
  `orig=` field of the `fuzzy-resolved` line + the `path-anomaly`
  `doubled=...` line) — that is the ONLY pre-mutation record. A live mutation
  is indistinguishable from producer drift inside your own session. EXTENDED
  to the escape/numword content channel: the SAME applies to
  `kind=escape scope=content` lines (`orig=[...] value=...`) — and the
  corrected form re-ENTERS your own context on the next pass, so a mutation
  looks like "I typed the corrected value again" (a false repetition loop).
  For any acceptance/diagnosis of escape or fuzzy behavior, check
  intercept.log FIRST; your own perception of your args is not evidence.
- **Why (evidence):** 2026-09-17 (planner ses_f4f539d7c, #73 live acceptance):
  the EMITTED doubled path `.../OpenCodeProjects/OpenCodeProjects/...` was
  fuzzy-resolved (intercept.log `orig=` doubled → corrected, `kind=dedup`),
  but the stored part `state.input` for that SAME read shows the SINGLE
  (corrected) path — queried directly from `opencode.db`. The planner, reading
  back its own "sent" args from context, concluded it had dropped the doubling
  (perceived drift); the log proved the read had arrived correctly doubled —
  while 5 write calls in the same session genuinely arrived single (the log
  shows `orig=` single, no anomaly line) — a REAL emission collapse. Both
  diagnoses were possible ONLY via the log; the stored args could not
  distinguish the two cases. Same class as the R2 display finding (post-
  mutation result shown) — extended to the CALL side. SECOND INSTANCE (the
  escape channel): 2026-09-18 (direct, ses_f4c039ae2ffeRqvdPqGu8IdB37) — the
  planner's live-acceptance writes EMITTED sentinel forms (one short `esc`,
  one `escape` numword form); the file on disk shows the corrected digits,
  and the planner PERCEIVED its own 3 "repeated" writes as literal digit
  strings (a false repetition loop); intercept.log 2026-09-18_12-19 lines
  (4 `kind=escape` verdicts, `hits=2`) prove every write carried the
  sentinels and each was `pair-resolved` (`orig=` = the sentinel form,
  `value=` = the field-2 digits).
  Ruled by the maintainer the same session: the agent CANNOT perceive the
  pre-corrected parts of a tool call — only the corrected form enters the
  context.
- **Ref:** `opencode.db` `part.state.input` for the #73 read;
  `.opencode/temp/intercept.log` 2026-09-17_21-03 lines; planner verification
  (2026-09-17 direct, ses_f4f539d7c); intercept.log 2026-09-18_12-19 lines
  (escape acceptance, ses_f4c039ae2ffeRqvdPqGu8IdB37).
- **Keys:** post-mutation, state.input, intercept.log, orig, display,
  self-diagnosis, bitdrift, producer-drift, indistinguishable, escape,
  numword, sentinel, false-repetition, pre-correction-perception.

## `session.create()` title param + the vendored SDK types (plan9 unit A)
- **Do:** to name a freshly-created session, pass
  `session.create({ body: { title } })` — `title` is a supported field of
  `SessionCreateData` (`body?: { parentID?: string; title?: string }`,
  url `/session`). For bounded SDK-surface lookups, use the VENDORED types
  in the repo at `.opencode/node_modules/@opencode-ai/sdk/dist/gen/{sdk,types}.gen.d.ts`
  (1.18.29, v1 generation) — the npm-global `opencode-ai@1.18.32` install at
  `C:\Users\Wasiejen\AppData\Roaming\npm` ships ONLY the CLI binary, NO
  `@opencode-ai/sdk` package (and `~/.config` is off-limits — maintainer
  ruling 2026-09-23).
- **Why (evidence):** bounded grep 2026-09-23 (worker-15, plan9 unit A,
  ses_f32120a60ffeoM8J0xg8Sb5Yym): `types.gen.d.ts` L1811-1821 carries
  `body.title`; the plugin passes it and the smoke pins the landing
  (104/104, UNIT 3 plan9 checks). The task spec's "the repo has NO
  node_modules" was stale — the vendored copy exists (flagged in the
  handover).
- **Ref:** `.opencode/node_modules/@opencode-ai/sdk/dist/gen/types.gen.d.ts`
  (L1811-1821); `.opencode/plugin/auto_resume.ts` (`spawnTitleFor` +
  `spawnPlanner`, 2026-09-23); `.opencode/plugin/tests/auto_resume.smoke.mjs`
  (UNIT 3 plan9 checks).
- **Keys:** session.create, title, SessionCreateData, vendored SDK,
  node_modules, plugin session naming, plan9, bounded-check.

## The auto_resume smoke: module-level `client`/`logDir` are overwritten by EVERY factory call
- **Do:** a smoke section added after the fail-safety block (which
  re-factories with the throwing v3Session client) must RE-FACTORY with a
  fresh spying session client itself — otherwise sends log `trigger=` but
  hit the throwing client (`send-fail=`, zero pushes to the spy array) and
  the case falsely fails. All watch state is shared module-level.
- **Why (evidence):** verified 2026-09-22 (worker_Q3S_160K
  ses_f3a03af20ffe1bRa56xVl143VG, the plan7/plan8 smoke work).
- **Ref:** `.opencode/plugin/tests/auto_resume.smoke.mjs`;
  `.opencode/plugin/auto_resume.ts` (module-level `client`, `logDir`,
  `watches`).
- **Keys:** auto_resume, smoke, factory, module-level, client, spy,
  re-factory, false-fail.

## R3 pair-channel live acceptance: the log is the authority, not self-perception
- **Do:** to live-verify an R3 pair/anchor channel, fire the REAL tool call
  then read `.opencode/temp/intercept.log` for the verdict line
  (`pair=[L:R] canon=N dist=0 gate=mutated line=M arg=...` /
  `kind=quoted ... pair-resolved`). NEVER judge "the model emitted the pair
  form" from the tool return or your own memory of the arg — the hook
  MUTATES the arg in place, so the return and your own view show only the
  POST-mutation canonical form. A "repeat" you perceive is a false-repeat.
- **Why (evidence):** 2026-09-26 (planner-23, ses_f219349ffffe1IL7z1xCByoX45,
  post-restart): a `block_transfer` COPY with `startMarker="Alpha [7:seven] T"`
  reached the hook VERBATIM (intercept.log `pair=[7:seven] canon=7 dist=0
  gate=mutated line=2 arg=startMarker ... pair-resolved`, 2/2) even though
  the tool return showed only the canonical block — the model DID emit the
  pair. This corrects the plan22 worker's "5/5 → canonical, model can't emit
  pairs" reading (a MEM-0101 false-repeat). The 14-26 blocked-anchor root
  cause (live process predating import fix 44c50a2) is gone: the anchor
  channels run clean, zero new `intercept-error` lines.
- **Ref:** `.opencode/temp/intercept.log` (2026-09-26_18-20 lines);
  `TODO.md` #95; planner destill MEM-0101.
- **Keys:** R3, pair-form, anchor-marker, section-anchor, intercept.log,
  live-acceptance, false-repeat, gate=mutated, constrained-decoding,
  read.offset integer.

## The section-anchor channel (R3 1.3) is schema-shadowed on this host
- **Do:** don't expect the `read` section-anchor channel to fire live from a
  quantized model on this host — `read.offset` is INTEGER-typed, so
  constrained decoding delivers a number, never a string anchor, and the
  resolver (probe-pinned at S31 check 319) is dormant live. Treat pair-form
  live acceptance as available ONLY on string-typed args (`block_transfer`
  markers, `glob`/`grep` `path`, `bash` `command`) — not on `read.offset`.
- **Why (evidence):** 2026-09-26 (planner-23): the anchor resolver + numeric-
  string repair are probe-pinned (S31 checks 319, 327) but the live `read`
  offset field cannot carry a string anchor under constrained decoding
  (worker 7/7 integer-1 stands). Pure resolver stays green; live channel
  dormant.
- **Ref:** `.opencode/plugin/probes/handover_probe.mjs` (S31, L7153+);
  `TODO.md` #95.
- **Keys:** section-anchor, read.offset, integer schema, constrained
  decoding, dormant channel, S31-319, R3.

## Compaction is now ONE shared core: `compaction_core.ts` (A+B+C unified)
- **Do:** when touching compaction, the SHARED behavior lives in
  `.opencode/plugin/compaction_core.ts` (a PURE module — NO tool
  registration, the T5 pattern). `compact_memory.ts` = thin wrapper (tool
  registration + arg validation + SELF/CROSS routing + the pre-compaction
  dump + queued-message path). `context_recovery.ts` = the hook (limit
  trigger + per-fire budget gate → the core) and it COMPACTS ONLY — it never
  resumes (no `promptAsync` / no `COMPACTION_RELOAD_DIRECTIVE` anywhere).
  The summarizer-pair resolution CARRIES THE SESSION'S OWN providerID+
  modelID (the 14-26 default-agent-drift fix). Both entry points resolve the
  SAME summarize body + cap semantics — pinned by probe S32 (checks 338-340).
  Do NOT re-fork the config/budget/cap/keep/summarize/callSummarize logic
  back into either wrapper.
- **Why (evidence):** 2026-09-26 (worker-23, ses_f2176db5affeHupTUDVa5cZ0vR):
  Parts A/B/C LANDED (218a2c1 / d9d93f8 / 88f902f). Final gate: probe
  340/340, pytest 459+1w, ruff F=0, cm 74/74, rc 17/17. `emergencyRecovery`
  left unflipped (the re-enable is the maintainer's live acceptance).
- **Ref:** `.opencode/plugin/compaction_core.ts`; `.opencode/plugin/
  context_recovery.ts`; `.opencode/plugin/probes/handover_probe.mjs` (S32,
  L7689+); `proposals/implemented/2026-09-26_compaction-unification.md`.
- **Keys:** compaction_core, shared core, context_recovery, no-resume,
  summarizer-pair, session-own modelID, S32 equivalence, T5, drift-guard.

## auto_resume: the LINEAGE-DEPTH CAP (verified 2026-09-27, from knowledge_inbox 2026-09-27_16-15)
- **Do:** a session at depth ≥ N (generations of plugin spawns from a user
  session) does NOT auto-spawn a successor on `action: restart` — the plugin
  logs `skip= depth sid=… depth=N` and NOTHING else (no `route=`, no trigger
  file, no sticky state; a failed/interrupted spawn sets no state either).
  The cap is LIVE-CONFIGURABLE: `.opencode/temp/lineage_max_depth` (one
  integer; -1 = unbounded; missing/unparseable → default 10 — raised from 2
  by the 2026-09-27 maintainer ruling, the autorun loop should never stall).
  The depth is RESTORED from `auto_resume.log` on host restart
  (`restoreLineageFromLog` pairs `route=`/`spawn=` lines) — a host restart
  NEVER resets the chain.
- **Why (evidence):** auto_resume.log + auto_resume.ts, verified 2026-09-27
  (planner-27): the depth-2 `skip=` lines for the plan26/27 restarts (before
  the 2→10 raise); the raise re-pinned smoke #90 (iv) (depth-2 now spawns) +
  #96 (c) 10-pair trim-restore fixture (the depth-10 boundary).
- **Diagnostic:** `grep -E "spawn=|route=|skip=|deactivate="
  .opencode/temp/auto_resume.log | tail`.
- **Keys:** lineage, depth cap, skip= depth, lineage_max_depth, restore,
  restart, spawn chain.

## Gauge budget suffix: no-total reads resolved the model to "" → default cap 1 (verified 2026-09-28, planner-31 — #114 root cause)
- **Do:** a gauge read with no finished step yet (a fresh session at
  session start) must resolve its model from the SESSION ROW's own
  `model` column (fallback), not from the finished-step row alone —
  otherwise the ` | N compactions left` suffix resolves
  `model_budget.default` (the store carries `"default":1`) and reports
  the wrong remainder (a fresh 245K-slow session reads "1 compaction
  left" instead of "5").
- **Why (evidence):** measured 2026-09-28 (planner-31,
  ses_f1a4f121cffepXvNl2sCuSaM51): the session-start injected `ctx:` line
  = `CTX=notAvailable | 1 compaction left`; the ctx_gauge self-read of
  the SAME session after the first finished step = `CTX=79335 (32%) … |
  5 compactions left` — 3rd live data point after planner-29 +
  explorer-30 (the #114 discrepancy). Root cause: `gauge.mjs`
  `readApiDb` sources `model` ONLY from the finished-step row
  (`step?.model`, L416); `SQL_NEWEST_SESSION` / `SQL_SESSION_BY_ID`
  select `id` only; no finished step → `modelId ""` →
  `compactionsLeftSuffix` fallback (entry.model — a fresh session has
  no budget entry yet — → "") → not in `model_budget` → `default: 1`.
  The live DB session row DOES carry
  `model = {"id":"Qwen3.8-27B-Q3S-245K-slow","providerID":"llama-swap",…}`
  — `parseModelId` yields the exact model_budget key (cap 5). BOTH
  surfaces (the ctx_watchdog chat.message post + the ctx_gauge tool)
  share the SAME gauge core — the divergence is ONE code path read at
  two times (session start vs post-first-step), not two code paths.
  `auto_resume.ts` `budgetExhausted` is NOT affected (it reads
  entry.model — an entry always carries its model; a fresh session is
  never exhausted).
- **Ref:** `.opencode/plugin/scripts/gauge.mjs` (SQL_NEWEST_SESSION L350,
  SQL_SESSION_BY_ID L374, readApiDb L402-438, compactionsLeftSuffix
  L292-329, GAUGE_SQL_MARKER L362-367, sqlMarkerForSession L382-390,
  readSpawnSqlite3 S-row parse L500-513); the store
  `.opencode/temp/compact_budget.json` (model_budget.default=1,
  sessions[sid].count).
- **Keys:** gauge, compactionsLeftSuffix, no-total, session-row model,
  model_budget default, injected ctx line, session start, #114.

## Gauge window is CONFIG-FIRST: root opencode.jsonc `limit.context` beats the name marker (verified 2026-09-28, worker-34 — #115)
- **Do:** the gauge's context window resolves CONFIG-FIRST: the root
  `opencode.jsonc` `provider.<pid>.models.<mid>.limit.context` (a finite
  number > 0) wins; the name-marker `parseWindow` is the FALLBACK (no
  provider prefix / config missing / unparseable / no entry / non-finite /
  <= 0 → the name parse, never a throw). A model rename (no K/M marker) is
  free as long as the config entry keeps its `limit.context` — the readout
  no longer loses `(pct%)/REM`.
- **Why (evidence):** #115 smoke pins green (config hit beats the name
  marker, no-marker + config, not-in-config, missing config, a JSONC config
  with comments incl. a `//` inside a string literal, no-prefix fallback);
  standard gate green (probe 335 PASS + the 11 pre-existing #113
  environmental numword-python failures 136-146, all other smokes at
  baseline, ruff F=0; pytest blocked by #113 — reported, not fixed).
- **Ref:** `.opencode/plugin/scripts/gauge.mjs` (`resolveWindow` +
  `parseJsonc` + `setConfigFileForTest`); `.opencode/plugin/tests/
  gauge_core.smoke.mjs` (#115 section); TODO #115.
- **Keys:** resolveWindow, limit.context, opencode.jsonc, JSONC, parseJsonc,
  name-marker fallback, gauge window, model rename, #115.
## 2026-09-30 planner-41 ses_f102c4a2dffezocrIY7kzd44YA
Compact-message delivery mechanics (source-verified 2026-09-30, research doc
`agent/research/2026-09-30_compact-message-delivery.md`): (1) the
`compact_memory` `message` arg is written to
`.opencode/temp/compact_message_<sid>` at DISPATCH time, unconditionally
(compact_memory.ts L494/L523 — write-before-dispatch) — **file presence ≠ a
compaction happened** (a failed dispatch leaves a zombie file); (2) the file
is delivered ONLY by the auto_resume unit-4 recovery (no-action-line idle)
path (auto_resume.ts L1349-1357, single call site) — scope-none sessions
(all Task-tool workers + direct sessions, the L1303 gate) NEVER get their
queued message, and an in-scope session that closes with an `action:` line
after compaction strands it (the restart branch never reads it); (3)
discriminator for deliberate vs limit-driven compaction (signal, not proof —
a self-compact without a `message` arg leaves no file): COMPACT line in
ctx.log + queue file = deliberate self-compact; COMPACT line without file =
limit-driven/auto; file without COMPACT line = failed dispatch (zombie).
Measured: 3 of 4 queued messages unconsumed (both workers + the direct
session); only the in-scope recovery path (planner-39) ever delivered.
- **Ref:** `.opencode/plugin/compact_memory.ts` (L303-310, L494/L523),
  `.opencode/plugin/auto_resume.ts` (L1303-1307, L1349-1357),
  `agent/research/2026-09-30_compact-message-delivery.md`,
  `proposals/2026-09-30_compact-message-delivery.md`.
- **Keys:** compact_message, .consumed, relay=, queued message, zombie file,
  scope none, task_id resume, deliberate vs limit compaction, ctx.log
  COMPACT line.

## sqlite3 CLI facts (verified 2026-09-30, from knowledge_inbox 2026-09-30_16-46, #126 worker)
- **Do:** with the bundled `.opencode/plugin/tools/sqlite3.exe`: (1) force
  `-separator '\t'` (argv, no shell) when fetching JSON data columns — the
  CLI's DEFAULT separator is `|`, which can occur inside JSON values, while
  a raw tab never does (JSON.stringify escapes all control bytes); (2) never
  inline `PRAGMA busy_timeout = N;` in the call — it ECHOES the value to
  stdout (matters for read-marker parsing; harmless where stdout is
  ignored); (3) every write script must carry the COMMIT statement — an
  uncommitted BEGIN..COMMIT batch that never reaches COMMIT is SILENTLY
  rolled back at process exit with exit code 0 (a flush script that omits
  COMMIT "succeeds" without persisting — this bit the #126 adapter once).
- **Why (evidence):** #126 worker (ses_f0d49cb6), measured against the
  bundled CLI (3.53.4, JSON1 verified).
- **Ref:** `.opencode/tools/context_trim.ts` (runCli),
  `.opencode/plugin/scripts/gauge.mjs` (readSpawnSqlite3).
- **Keys:** sqlite3, separator, PRAGMA echo, silent rollback, COMMIT,
  spawn-sqlite3.

## Spawning CLIs on the Bun-compiled live host: three measured defects (verified 2026-10-01, from knowledge_inbox 2026-10-01_01-46 + feedback 2026-10-01_02-59 + the #126 live-debug)
- **Do:** every spawn-based tool on the live host must (1) NEVER pass
  whitespace-bearing argv (Bun's Windows command-line join collapses it →
  the CLI's arg parsing shifts → interactive mode → deterministic 2500 ms
  kill; use a non-whitespace control-char separator — `\x01` (SOH) never
  appears in JSON.stringify output); (2) NEVER inline double-quote-bearing
  strings (the SAME join family mangles double-quote argv — do in-string
  edits INSIDE sqlite, e.g. `json_set`); (3) use the ASYNC execFile
  (promisified, `res.stdout` — the gauge's `readSpawnSqlite3` pattern),
  NEVER execFileSync — on the live Bun host execFileSync succeeds only ONCE
  per process (compiled-Bun Windows stdio-handle issue; every later spawn
  2500 ms-kills, while bash/node parents and async execFile keep working);
  (4) set a spawn maxBuffer of ≥ 16 MB (a context_trim report pulls
  1–1.3 MB of window part-JSON through the CLI; the 1 MB default →
  ENOBUFS).
- **Why (evidence):** #126 forked live test 2026-09-30/10-01 (three 2500 ms
  kills + ENOBUFS; the identical calls 19–30 ms from bash); the gauge's ctx
  lines never failed (it already uses the async pattern).
- **Ref:** `.opencode/tools/context_trim.ts` (runCli — the async execFile +
  `\x01` + 16 MB pattern), `.opencode/plugin/scripts/gauge.mjs`.
- **Keys:** Bun, Windows, command-line join, argv, execFileSync once-per-
  process, async execFile, SOH separator, ENOBUFS, 2500 ms kill.
- **Review trigger:** the host leaves Bun, or Bun fixes its Windows argv
  quoting.

## Backend CUDA errors: suspect GPU clocks first; the backend is SELF-HEALING (maintainer-confirmed 2026-10-01, from knowledge_inbox 2026-10-01_03-12/03-15)
- **Do:** when CUDA-style backend errors recur (mid-step kills, empty
  assistant rows, "Cannot have 2 or more assistant messages" halts): suspect
  OVER-AGGRESSIVE GPU CLOCK settings BEFORE suspecting session/compaction
  infrastructure — gaming-stable-but-inference-unstable clocks caused the
  2026-09-30/10-01 error window; the backend was stable again after the
  maintainer's clock change. And do NOT manually interrupt a session that is
  interrupting/auto-retrying: the backend is SELF-HEALING (it restarts and
  takes the interrupted message back up exactly where it left — re-prefill,
  seamless; observed mid-sentence) — a manual interruption colliding with
  the auto-retry cycle is the suspected producer of the trailing assistant
  rows that trigger the provider rejection (SUSPECTED — confirm only by an
  un-interfered observation).
- **Why (evidence):** maintainer-confirmed 2026-10-01 (the fork test
  session ses_f0b92a96 window; DB tail + auto_resume.log + his backend
  observation).
- **Ref:** TODO #126 close note (todo_records.md), todo_inbox
  2026-10-01_01-03 (the fork retry path, SUSPECTED).
- **Keys:** CUDA, GPU clocks, backend restart, self-healing, manual
  interrupt, retry path.

## Magic Context (external context-manager plugin, cortexkit/magic-context) — evaluated 2026-10-01; full doc `agent/research/2026-10-01_magic-context-plugin.md`
- **Do:** do NOT re-research Magic Context. It is an external context
  manager that replaces host compaction (their trim = agent `ctx_reduce`
  + plugin-side historian on a hidden cheap model; tiered compartment
  summaries; 5-category shared-SQLite memory). Our adoption list (doc
  §Adoption implications, 2026-10-01): (2) explicit keep-N surface →
  proposed `proposals/2026-10-01_explicit-keep-N-surface.md` (plan8,
  pending); (1) usage-adaptive keep `usable × 0.3 × (1 − usage)` (floor 8 %
  of usable, clamped 2,000–12,000) + (4) usage-band nudges ({U,T} bands
  0.20/0.40/0.60/0.75) → proposed `proposals/2026-10-01_usage-adaptive-keep.md`
  + `proposals/2026-10-01_gauge-nudge-bands.md` (plan9, pending); (3)
  importance-scored tiered summaries → PARKED (effort L; assumes their
  head-render model m[0]/m[1], which our host lacks). NOT to borrow
  (reasons in the doc): cache-aware flush/defer scheduling (no meaningful
  provider prompt cache on the single local slot), the dreamer overnight
  cron + historian-as-cheap-model (no second model slot), the shared-SQLite
  cross-session memory DB + embeddings (our memory track is file-based by
  design at this scale), fail-safe plugin conflict detection (we run one
  context manager).
- **Why (evidence):** explorer_Q3S_slow research 2026-10-01
  (ses_f09d877e4ffe7sI5ZnOHKWKBQ6) — source reads of historian.md /
  reclaim.md / nudges.md / CONFIGURATION.md via raw.githubusercontent.
- **Ref:** `agent/research/2026-10-01_magic-context-plugin.md` (396 lines),
  the 3 pending proposals at the proposals/ root.
- **Keys:** Magic Context, cortexkit, protected tail, ctx_reduce,
  historian, compartments, usage-adaptive keep, nudge bands,
  not-to-borrow.
- **Review trigger:** a maintainer ruling on any of the 3 proposals, or a
  re-research of Magic Context.
