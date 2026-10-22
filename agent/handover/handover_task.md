# Task spec — TODO #137 unit 5b: ctx_gauge + state_snapshot + dev helper ported to v2 (clone `migration`) — QUEUED, launch AFTER 5a verifies

## Goal
Port the three read-only/dev tools to v2 plugin-registered tools: the gauge
readout tool, the planner init digest, and the tool-context dev helper. Behavior
byte-compatible where the v1 surface carries pins; the v1 context-carrier legs with
no v2 counterpart logged as deviations.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (HEAD = post-5a). Stay on the current checkout.
- Assigned worker: `worker_FlashNext`.
- TODO entry: `TODO.md` #137. Design of record (READ-ONLY): clone
  `.opencode/tools/ctx_gauge.ts` (36 lines, whole file OK), clone
  `.opencode/tools/state_snapshot.ts` (206 lines, whole file OK), clone
  `.opencode/tools/dev_get_tool_context_contents.ts` (59 lines, whole file OK),
  clone `.opencode/plugins/loop_log.ts` + `submit.ts` (5a's plugin shape — read
  the plugin-module section only). Registration pattern: 4a's compact_memory.ts.

## Definition of done (measurable)
1. NEW plugin `.opencode/plugins/ctx_gauge.ts` (id `ctx_gauge`) — the tool wraps
   the SHARED gauge core (clone `.opencode/plugin/scripts/gauge.mjs` — ALREADY
   v2-paired by units 1-2; import via the context_family.ts relative-spec pattern
   `"../plugin/scripts/gauge.mjs"`). Same in-band mirror as v1:
   `formatGauge(readGauge(undefined, args?.sessionID))`; on `kind:"db-error"`
   APPEND ` — <error>` (never a stack). The budget suffix + `(unlisted)` marker
   ride the core's `compactionsLeftSuffix` — NOTHING re-derived here.
2. NEW plugin `.opencode/plugins/state_snapshot.ts` (id `state_snapshot`) —
   the 6-section capped digest, byte-shape carried: `## git` (execFileSync
   `git log --oneline -20`), `## nap` (first 40 lines of
   `agent/handover/handover_planner.md` + read-more steer), `## todo` (open-header
   count + LAST 15 headers), `## maintainer` (inbox_planner `.md` first lines, max
   5 + `priority.md` first 10 lines), `## proposals` (root `.md` names), `##
   markers` (the READY-MADE sweep via `execFileSync("bash", ["-c", MARKER_SWEEP])`
   — verbatim from the v1 file), `(none)` empties, hard 60-line cap (LAST
   section truncates first). The soft role gate carries: a non-empty
   `context.agent` that is not a planner → the exact one-line refusal.
   READ-ONLY — NEVER writes.
3. NEW plugin `.opencode/plugins/dev_get_tool_context_contents.ts` (id
   `dev_get_tool_context_contents`) — returns the v2 tool-context dump as ONE
   plain string (the v1 2026-09-27 string-return discipline kept): the v2 `ToolContext`
   keys/fields that EXIST (`sessionID`, `agent`, `messageID`, `id`, `progress`,
   `signal` — see Verified facts) with the same `<function>` markers; the v1-only
   legs (`callID`, `extra`, `messages`, `metadata`, `ask`, `directory`, `worktree`,
   `client`) → NO keys invented: report them as absent per the actual runtime
   keys; non-serializable → the v1 keys-only fallback line, NEVER throw.
4. SMOKES — new `.opencode/plugins/tests/ctx_gauge.smoke.mjs`,
   `state_snapshot.smoke.mjs`, `dev_get_tool_context_contents.smoke.mjs` (bun,
   mock ctx, scratchpad fixture dirs, gauge steered at the scratchpad FIXTURE DB
   pair — NEVER the live DB; state_snapshot fixture repo tree in the scratchpad
   incl. one `.md` with a marker line). Pins: gauge readout forms + the db-error
   append form; digest section order + the 60-line cap + the refusal line +
   `(none)` empties; dev-helper string return + absent-leg handling. Each final
   line `SMOKE: ALL PASS`.
5. GATE — clone probe `360/360` UNCHANGED; the 12 v1 node smokes ALL PASS
   (incl. `plugin/tests/ctx_gauge.smoke.mjs` 3/3 + `state_snapshot.smoke.mjs`
   20/20 — they pin the v1 `.opencode/tools/` twins, DO NOT repurpose); ALL v2
   plugin smokes ALL PASS; `tsgo -p .opencode/plugins/tests/tsconfig.json` exit
   0 (add the 3 plugins to `files`); tests README gains the sections; clone tree
   clean.

## Verified facts (measured at spec-authoring, 2026-10-22)
- Twin freshness (measured 2026-10-22, `cmp`): the THREE 5b design-of-record files are
  BYTE-IDENTICAL between MAIN and clone (ctx_gauge 36/36, state_snapshot 206/206,
  dev helper 59/59) — the clone twin is a valid baseline here. (Contrast: the
  `submit`/`block_transfer` twins DIFFER — the clone copies are STALE, TODO #143.)
- `ToolContext` v2 (promise surface) = `{sessionID, agent, messageID, id, signal,
  progress}` — fork `packages/schema/src/tool.ts:14-19` (Context) +
  `packages/plugin/src/promise/tool.ts:11-13` (adds `signal`, re-types
  `progress`). The v1 dev-helper list (`sessionID, abort, messageID, callID,
  extra, agent, messages, metadata, ask, directory, worktree`) is the v1 shape —
  the v2 execute context has NO `directory`/`extra`/`client` carrier.
- The gauge core export set the v1 tool uses: `readGauge` / `formatGauge`
  (`packages/…` irrelevant — clone `.opencode/plugin/scripts/gauge.mjs:503
  formatGauge`, `:458 compactionsLeftSuffix`, `readGauge` in the same file); the
  v2-paired session-row + token-mass legs landed in units 1-2 (`4c0e607`-era +
  `922f74c`/`2c1fa23`).
- state_snapshot v1 exec facts: `execFileSync("git", ["log","--oneline","-20"],
  {cwd: root, encoding:"utf8"})` at `.opencode/tools/state_snapshot.ts:57-61`;
  the marker sweep `execFileSync("bash", ["-c", MARKER_SWEEP])` at `:146`; the
  soft role gate comment at `:195-196`. `root` = the repo root (v1 resolved via
  context; in v2 use the setup-captured `ctx.location.directory` —
  context_family.ts:497 precedent).
- Plugin discovery + decode contract + registration idiom: as in the 5a spec
  (compact_memory.ts:693-694 `ctx.tool.transform` + `editor.add`; teardown
  `dispose()`).
- Baselines: clone probe 360/360; v2 smokes intercept 52 / context_family 46 /
  context_trim_v2 65 / compact_memory 63 / (post-5a) loop_log + submit; v1 node
  smokes incl. ctx_gauge 3/3, state_snapshot 20/20.

## Suggested scope (non-exhaustive)
- NEW: the 3 plugin files + 3 smokes + tsconfig `files` entries + tests README
  sections (`.opencode/plugins/tests/README.md`).
- Reference READ-ONLY: the 3 v1 twins in clone `.opencode/tools/`, clone
  `.opencode/plugin/scripts/gauge.mjs`, 5a's plugins (shape), MAIN repo
  (bookkeeping only).

## Approval boundary
Pre-approved class (agent-facing; live v2 acceptance = maintainer domain). DO NOT
touch: MAIN repo (except summary + planner bookkeeping); `maintainer/**`; clone
`opencode.jsonc`, `.opencode/tools/**`, `.opencode/plugin/**`,
`.opencode/plugin-unported-v1/**`, `plugins/intercept_observer.ts`,
`plugins/context_family.ts`, `plugins/context_trim_v2.ts`, `plugins/compact_memory.ts`,
5a's `loop_log.ts`/`submit.ts` (5b must not edit them), the live DB.

## Procedure (guidance)
1. Read the 3 v1 files in full (all small); read 5a's plugin-module sections.
2. Implement ctx_gauge (thinnest), then state_snapshot, then the dev helper.
3. Smoke each; tsconfig + README; full clone gate; checkpoint commits per
   verified tool; worker summary to MAIN `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- block_transfer v2 (5c) — separate spec; `session_info` — native, no port.
- Live v2 acceptance (maintainer); the injected-ctx-line lane (3a/3b, landed);
  the v1-side reduction/dual-merge unit.
