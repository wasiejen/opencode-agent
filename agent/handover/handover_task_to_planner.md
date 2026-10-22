# Worker summary — TODO #137 unit 5b: ctx_gauge + state_snapshot + dev helper ported to v2 (clone `migration`)

Status: COMPLETE

## Task and scope
Ported the three read-only/dev tools to v2 plugin-registered tools: the gauge
readout (`ctx_gauge`), the planner init digest (`state_snapshot`), and the tool-
context dump (`dev_get_tool_context_contents`). Digest byte-shape and gauge
readout forms carried from the v1 twins (the clone `.opencode/tools/` files,
byte-identical to MAIN at baseline). All code changes landed in the CLONE
`C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch `migration`, baseline
`a5da17b` — stayed on the checkout). MAIN repo: this summary only; nothing
else touched. DO-NOT-touch list (maintainer/**, clone `opencode.jsonc`,
`.opencode/tools/**`, `.opencode/plugin/**`, `.opencode/plugin-unported-v1/**`,
`plugins/intercept_observer.ts`, `context_family.ts`, `context_trim_v2.ts`,
`compact_memory.ts`, 5a's `loop_log.ts`/`submit.ts`, the live DB) — UNBROKEN:
`git status` shows no edit to any of them; the v1 twins stayed inert design-of-
record (their 12 v1 smokes still pin them).

## Files changed (CLONE, 3 checkpoint commits)
- `8e630f1` — NEW `.opencode/plugins/ctx_gauge.ts` (id `ctx_gauge`) + NEW
  `.opencode/plugins/tests/ctx_gauge.smoke.mjs` (11 pins) + tsconfig `files`
  entry + README section.
  The tool wraps the SHARED gauge core via the context_family relative spec
  `"../plugin/scripts/gauge.mjs"`: `formatGauge(readGauge(undefined, sid))`;
  `kind:"db-error"` → ` — <error>` APPENDED (never a stack). The budget suffix +
  the `(unlisted)` marker ride the core's `compactionsLeftSuffix` — NOTHING
  re-derived. Deviation logged in-file: the v1 bare-string return becomes the
  v2 `{output}` result; NO repo-root capture needed (the core resolves its own
  paths).
- `ef3ebbb` — NEW `.opencode/plugins/state_snapshot.ts` (id `state_snapshot`) +
  NEW `state_snapshot.smoke.mjs` (24 pins) + tsconfig entry + README section.
  All six sections carried: `## git` (`execFileSync git log --oneline -20`),
  `## nap` (first 40 + `… (read … for the rest)` steer), `## todo` (count +
  LAST 15 `#<id> — <60-char trunc>` headers), `## maintainer` (inbox first-
  lines max 5 + `… (+k more)` + priority first 10), `## proposals` (root `.md`
  names), `## markers` (`execFileSync("bash", ["-c", MARKER_SWEEP])` — the sweep
  VERBATIM from v1), `(none)` empties, fail-closed lines, the 60-line hard cap
  (LAST section's content lines trim from the end backwards; headers + steers
  preserved), and the EXACT soft-gate refusal line for a non-planner agent.
  READ-ONLY (pinned: the fixture tree byte-identical around a run). Deviation
  logged: the root rides the setup-captured `dir = ctx.location.directory`
  (context_family.ts:497 precedent) — the v1 execute-time `context.directory`
  has NO v2 carrier.
- `45bb52e` — NEW `.opencode/plugins/dev_get_tool_context_contents.ts` (same
  id) + NEW `dev_get_tool_context_contents.smoke.mjs` (13 pins) + tsconfig
  entry + README section. ONE plain string (the v1 2026-09-27 discipline kept).
  Dump fields = the v2 `ToolContext` keys that EXIST (`contextKeys` +
  `sessionID`/`agent`/`messageID`/`id`/`progress`/`signal`, `progress` →
  `<function>`). The v1-only legs (`callID`,`extra`,`messages`,`metadata`,`ask`,
  `directory`,`worktree`,`client`) → NO keys invented; absence rides the
  runtime `contextKeys`. Non-serializable → the v1 keys-only fallback line,
  NEVER a throw.
- `8e630f1..45bb52e` — `.opencode/plugins/tests/tsconfig.json` (`files` +3) and
  `.opencode/plugins/tests/README.md` (+3 sections).

## Behavior changed
Three new tools become available to v2 plugin-discovery (`ctx.tool.transform` +
`editor.add`, teardown `dispose()`, no `opencode.jsonc` change — the 3a auto-
discovery precedent). Live v2 acceptance is the maintainer's domain (unchanged
here). `state_snapshot` now gates on the v2 `context.agent`; its repo root is
the setup capture, not an execute-time context field.

## Measured verification (workdir = the CLONE)
- Gate probe: `node .opencode/plugin/probes/handover_probe.mjs` →
  `PROBE handover: 360/360 PASS`, exit 0 — UNCHANGED baseline.
- All 12 v1 node smokes, each `node .opencode/plugin/tests/<f>.smoke.mjs`
  (auto_resume 170/170, block_transfer.sandbox 76/76, block_transfer 132/132,
  compact_memory 110/110, context_recovery 17/17, context_trim 40/40, ctx_gauge
  3/3, gauge_core, intercept_observer 85/85, loop_log 69/69, state_snapshot
  20/20, submit 31/31) → ALL PASS, exit 0 (the v1 twins' pins intact; NOT
  repurposed).
- All 7 v2 plugin smokes, `bun run <f>.smoke.mjs` from
  `.opencode/plugins/tests` (smoke 52-pin intercept, context_family, compact_memory,
  context_trim_v2, loop_log, submit, + the 3 new) → `SMOKE: ALL PASS`, exit 0.
- Exact typecheck command: `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/
  tsgo.exe -p tsconfig.json` (run in `.opencode/plugins/tests`) → no output,
  **PASS** (exit 0), with all three new plugins in `files`. Re-verified after
  the last commit: `... -p .opencode/plugins/tests/tsconfig.json` from the clone
  root → PASS (exit 0).
- New smoke pin counts: ctx_gauge 11, state_snapshot 24, dev helper 13.

## Working-tree status
`clean` in the CLONE at `45bb52e` (branch `migration`). MAIN repo: this summary
file written UNCOMMITTED (the planner owns MAIN bookkeeping/commits); MAIN also
shows a pre-existing `maintainer/SCRATCH_PAD.md` modification — not mine,
untouched.

## Commits (clone, on top of baseline a5da17b)
`8e630f1` ctx_gauge · `ef3ebbb` state_snapshot · `45bb52e` dev helper. No
push (agents never push).

## Findings for the planner (TODO entry work NOT mine — no clone TODO.md edit)
1. `agent/readme/repo_commands.md` (clone AND MAIN) names "this probe + the
   smoke suite" as the standard gate but documents NO smoke-suite commands
   (v1 = `node .opencode/plugin/tests/<f>.smoke.mjs` from the root; v2 = `bun
   run <f>.smoke.mjs` in `.opencode/plugins/tests`); I had to derive them from
   the two tests READMEs. Worth a docs entry.
2. The clone's v1 twin `.opencode/tools/dev_get_tool_context_contents.ts`
   carries the v1-only legs (`callID`/`extra`/`ask`/…); the v2 dump has NO
   carrier for them — expected deviation, already logged in the plugin header
   (mirrors the 5a `submit` twin note, TODO #143 twin staleness).
3. Verified nuance: the dev-helper keys-only fallback fires ONLY when a DUMP-
   CARRIED field is unserializable (a `contextKeys`-only string can never trip
   `JSON.stringify`); pinned that way (`signal` circular).

## Deliberately omitted / out of scope
- `block_transfer` v2 (unit 5c) and `session_info` (native — no port), per the
  spec's out-of-scope list.
- Live v2 acceptance (maintainer); the v1-side reduction/dual-merge unit; no
  `opencode.jsonc` registration.
- No MAIN `TODO.md` / `todo_inbox.md` writes (the planner curates); findings
  above go through the `submit` tool's todo channel instead — see Friction.

## Blockers
None.

## Lessons
- A v2 plugin smoke can steer the SHARED gauge core (`setDbPath` /
  `setBudgetFileForTest`) because the plugin's relative `.mjs` import and the
  smoke's absolute `pathToFileURL` import resolve to ONE module instance — the
  fixture-DB steering needs no plugin test hook (context_family precedent).
