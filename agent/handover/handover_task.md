# Task Spec — loop iteration counter, Part 1 + Part 2 + seed

Approved proposal: `proposals/approved/2026-10-01_loop-iteration-counter.md`
(read it first — the design + acceptance criteria live there). This spec covers
Part 1 (the counter file written at spawn), Part 2 (the ctx-line `ITER`
suffix), and the Part 3 seed file. The Part 3 DOC changes (readme_loop.md)
land planner-side — NOT your scope.

## Scope
- `.opencode/plugin/auto_resume.ts` — replace `spawnTitleFor()` with
  `spawnCounterFor()` (counter read + log cross-check + write); the restart
  branch passes the computed N into `restartText()` (new text below).
- `loop/autorun-2026-10-01_03-27/counter.json` — the bootstrap seed (below).
- `.opencode/plugin/scripts/gauge.mjs` — an exported helper
  `loopIterationSuffix()` + a test hook (mirror the `setBudgetFileForTest` /
  `getBudgetFile` pattern; default loop root resolved relative to THIS_DIR
  like `DEFAULT_BUDGET_FILE`).
- `.opencode/plugin/ctx_watchdog.ts` — in `onChatMessage`, append the suffix
  to the posted part text (`ctx: ${line}${suffix}`) — the injected
  chat.message ctx line ONLY. The peek.mjs / ctx_gauge tool readouts are
  unchanged.
- Pins: `.opencode/plugin/tests/auto_resume.smoke.mjs`,
  `.opencode/plugin/probes/handover_probe.mjs` (new S36 section).

## Behavior (Part 1)
`spawnCounterFor() → { folder, n } | null`:
1. folder = `currentLoopFolder()`; null → null (un-named spawn, unchanged).
2. Candidate: `loop/<folder>/counter.json` = `{"looprun": <folder>, "n": N}`
   — absent / unparseable / `looprun` ≠ folder → 1; else N + 1.
3. Log max: largest match of `/planner-(\d+)/g` in `loop/<folder>/loop_log.md`
   (missing / unreadable → 0 — the old code returned null there; that changes).
4. n_final = max(candidate, logMax).
5. Write the counter with n_final — best effort (a write failure must not
   break the spawn).
Title = `<folder> planner-<n_final>`; the `ident=` bit and the `spawn=` line
format are unchanged. The restart-text base line becomes:
"Your iteration number is <N> (the authoritative loop counter — the
loop-log grep stays the fallback verification only)." — the rest of
`restartText` (incl. the exhausted variant) is unchanged.

## Behavior (Part 2)
`loopIterationSuffix()` → `""` or `" ITER=<n> <folder>"`: resolve the current
loop folder (exactly one `autorun-*` dir → it; several → most-recently-modified;
none / unreadable → `""`), read the folder's `counter.json`, present +
parseable → the suffix. Never throws.

## Bootstrap seed (Part 3 file)
`loop/autorun-2026-10-01_03-27/counter.json` =
`{"looprun":"autorun-2026-10-01_03-27","n":11}` — the current iteration (see
verified facts). WITHOUT the seed, the first post-build spawn computes
max(1, 10) = 10 — a doubled naming with plan10; the seed makes it 12.

## Pin audit (every pin surface the change touches — repurposing in-scope)
- auto_resume.smoke.mjs, plan9 ident block (fixture folder `autorun-test_0-0`,
  log max planner-7, no counter → N = max(1,7) = 8: title/ident UNCHANGED) +
  NEW pin: the fixture `counter.json` written byte-exact with n=8.
- auto_resume.smoke.mjs, "no planner-<N> line → no identifier" block —
  REPURPOSED (behavior pin): no counter + no planner line → NOW N = 1 →
  title `<folder> planner-1` (old: no title at all).
- auto_resume.smoke.mjs, the restart-text byte-exact pin (search
  `Your iteration number`) — re-pin to the new explicit-N text.
- NEW pins, the proposal acceptance (a)–(d): (a) counter n=4 + log max 4 → 5;
  (b) counter n=4 + log max 6 → 7; (c) counter n=5, log max 4 (a dead session
  wrote no line) → 6, never 5; (d) counter of a DIFFERENT looprun → 1.
- handover_probe.mjs — NEW S36 section pinning `loopIterationSuffix()` via the
  test hook: (i) a loop root without a counter → `""`; (ii) fixture
  `autorun-X/counter.json` n=7 → ` ITER=7 autorun-X`. The EXISTING byte-exact
  ctx pins (CTX_OK / CTX_UNKNOWN / CTX_UNAVAILABLE) must stay byte-identical
  (no counter in the live loop folder at probe time). Update the header
  section-sum line + the total annotation (the annotation is the source).

## Verified facts (planner-verified at spec authoring, 2026-10-05)
- The live loop folder = `autorun-2026-10-01_03-27` (exactly one under
  `loop/`); NO counter.json yet; the log regex max is `planner-10`
  (`grep -oE "planner-[0-9]+" loop/autorun-2026-10-01_03-27/loop_log.md |
  sort -u -V | tail -1`); the current planner session = iteration 11, its
  successor = 12.
- auto_resume.ts: `currentLoopFolder()` and `spawnTitleFor()` are adjacent
  (search `function spawnTitleFor`); `restartText(sid, exhausted)` carries
  the old iteration line (search `Your iteration number`); the spawn call
  uses the title for the create + the `ident=` bit.
- The probe's current total = 358/358 (header self-annotated; the last
  section S35 = checks 354-356) → S36 starts at check 357.
- gauge.mjs convention: `join(THIS_DIR, "..", "..")` = the `.opencode/`
  dir (see `DEFAULT_BUDGET_FILE`) → the loop root =
  `join(THIS_DIR, "..", "..", "..", "loop")`.
- Baseline gate: probe 358/358 + all smokes green under node v24.19.0
  (bun is broken — run the gate under node).

## DoD
1. Counter scenarios (a)–(d) + the repurposed no-line case + the
   counter-write pin green in auto_resume.smoke.
2. The restart text carries the explicit N (re-pinned).
3. Probe S36 green; header self-annotation accurate; the old ctx pins
   byte-identical.
4. The seed file committed with n=11.
5. Standard gate green (probe + all smokes, node).
6. Checkpoint commits per verified unit (Part 1 code+pins; Part 2 code+pins;
   seed + TODO/handover in the final commit). No new TODO entry expected — a
   doc/code discrepancy may legitimately surface one (file it per the TODO
   contract then).

## DO-NOT-touch
`maintainer/**`, `opencode.jsonc`, `AGENTS.md`, `agent/prompts/**`,
`agent/readme/**` (the Part 3 docs land planner-side), `intercept_observer*`,
the FST repo, `archive/**`, other loop folders,
`.opencode/temp/compact_budget.json`.

## Close
Status: LANDED (the commit hash recorded in the planner's follow-up
bookkeeping commit — no self-reference); the handover to
`agent/handover/handover_task_to_planner.md`.
