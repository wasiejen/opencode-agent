# loop iteration counter handover — Part 1 + Part 2 + bootstrap seed

Worker: worker_Q3S_slow (session ses_ef70261b3ffewfIAhOa8hu603o).
Task spec: `agent/handover/handover_task.md`; approved proposal
`proposals/approved/2026-10-01_loop-iteration-counter.md`.

## What landed (3 commits, all green)

**Commit 1 — `b97bf75` Part 1 code + pins**
- `.opencode/plugin/auto_resume.ts`:
  - `spawnTitleFor()` REPLACED by `spawnCounterFor() → {folder, n} | null`
    — reads `loop/<folder>/counter.json` (candidate = n+1; absent /
    unparseable / `looprun` ≠ folder → 1), cross-checks the log max
    (missing/unreadable → 0), `n_final = max(candidate, logMax + 1)`,
    writes the counter best-effort (a write failure never breaks the
    spawn). The old "no planner line → null (un-named)" behavior is
    GONE: a title now always resolves when a loop folder exists.
  - `spawnPlanner(startPrompt, sourceMsgs?, precomputed?)` — the counter
    is computed EXACTLY ONCE per spawn; the file-trigger path computes
    it inside, the restart branch passes its precomputed counter.
  - `restartText(sid, exhausted, n?)` — the iteration line becomes
    `Your iteration number is <N> (the authoritative loop counter — the
    loop-log grep stays the fallback verification only).`; `n == null`
    (no loop folder — degenerate sandbox / pre-loop case) → the OLD
    derivation text (kept as the fallback; the `loop_log.md` pin at the
    first UNIT-4 restart stays green on it). Exhausted variant unchanged.
  - The restart branch computes the counter once and passes the same N
    into the text and the title (no double write, no drift).
- `.opencode/plugin/tests/auto_resume.smoke.mjs` (165 → 170):
  - plan9 ident block: title/ident UNCHANGED (planner-7 → planner-8) +
    NEW pin: the fixture `counter.json` written byte-exact `{"looprun":"autorun-test_0-0","n":8}`.
  - "no planner-<N> line" block REPURPOSED (behavior pin): no counter +
    no planner line → NOW N = 1 → title `autorun-test_0-0 planner-1` +
    counter written n=1.
  - NEW block (10): acceptance scenarios (a) counter 4 + log 4 → 5;
    (b) counter 4 + log 6 → 7; (c) counter 5 + log 4 (dead session) → 6,
    never 5; (d) foreign-looprun counter + no planner line → 1 (the
    write normalizes the looprun) — each pinning title + written
    counter.json bytes.
  - ITEM 2/11/4 section: a deterministic loop fixture (counter n=4, log
    max planner-4) makes the restart N machine-computable; the
    byte-exact restart-text pin re-pinned to the explicit-N text via a
    parameterized `baseRestartText(n)` (emg 6 / i4 7 / unread 8 — the
    counter advances per restart); the fixture is removed after the
    item-4 section (the #109 section creates its own loop folder).

**Commit 2 — `e9447c2` Part 2 code + pins**
- `.opencode/plugin/scripts/gauge.mjs`: exported `loopIterationSuffix()`
  (→ `""` or ` ITER=<n> <folder>`; folder resolution mirrors
  auto_resume's currentLoopFolder; present + parseable counter → suffix;
  never throws) + test hook `setLoopRootForTest` / `getLoopRoot`
  (the setBudgetFileForTest pattern) + `DEFAULT_LOOP_ROOT =
  join(THIS_DIR, "..", "..", "..", "loop")` (= `<repo>/loop`).
- `.opencode/plugin/ctx_watchdog.ts`: the INJECTED chat.message ctx line
  only — `text: ctx: ${line}${iterSuffix}`. The peek.mjs / ctx_gauge
  readouts are UNCHANGED (they format via formatGauge, untouched).
- `.opencode/plugin/probes/handover_probe.mjs`:
  - `setLoopRootForTest(path.join(SANDBOX, "loop"))` at probe start
    (never-created sandbox root) — the EXISTING byte-exact ctx pins
    (CTX_OK / CTX_UNKNOWN / CTX_UNAVAILABLE) stay byte-identical
    (no loop folder/counter → `""` suffix).
  - NEW S36 section (checks 357-358): (i) steered loop root without a
    counter → `""`; (ii) fixture `autorun-X/counter.json` n=7 →
    ` ITER=7 autorun-X` byte-exact.
  - Header self-annotation: `S35=3 S36=2 hygiene=6 → "PROBE handover:
    360/360 PASS"` (was 358/358); the EXTENDED 2026-10-05 note added.

**Commit 3 — seed + close (this commit)**
- `loop/autorun-2026-10-01_03-27/counter.json` =
  `{"looprun":"autorun-2026-10-01_03-27","n":11}` (45 bytes, no trailing
  newline — byte-verified). The live log max is NOW planner-11 (the
  current session's START line landed after spec authoring) — the next
  spawn computes max(11+1, 11+1) = **12** as the spec intends.
- `todo_inbox.md` entry + `agent/agent_feedback.md` friction entry
  (the spec doc discrepancy — below).
- This handover + the loop_log.md DONE line.

## Measured gate evidence (all under node v24.19.0)

- Baseline (pre-change): probe 358/358 + 12/12 smokes green
  (auto_resume 165/165).
- Post-Part-1: auto_resume.smoke **170/170** green.
- Post-Part-2: probe **360/360 PASS** (checks 357/358 = the new S36; the
  S4 byte-exact ctx pins re-verified green in-run) + 12/12 smokes green
  (auto_resume 170/170, the other 11 byte-unchanged).
- One transient failure during development: the re-pinned restart-text
  checks first failed because `baseRestartText` put `${n}` in a
  double-quoted string (no interpolation) — fixed by concatenation;
  final run all green.

## Spec discrepancy found (filed to todo_inbox.md, NOT fixed — spec is planner-owned)

The spec's Behavior (Part 1) step 4 says `n_final = max(candidate,
logMax)`, but the spec's own acceptance (b) (counter n=4 + log max
planner-6 → **N=7**) is only satisfiable by `max(candidate, logMax + 1)`
(candidate already = counter.n + 1). The approved proposal's design text
carries the same loose phrasing. I implemented to the ACCEPTANCE
(the machine-verifiable contract) — the pins prove (a)→5, (b)→7, (c)→6,
(d)→1. The one-word fix (`logMax + 1`) belongs in the spec + proposal
wording before the Part 3 docs (readme_loop.md) are written planner-side,
so the docs state the correct formula.

## Deliberately NOT done (out of scope / per spec)

- **Part 3 docs (readme_loop.md §Self-identification / §Iteration
  semantics)** — planner-side per the spec's DO-NOT-touch list
  (`agent/readme/**` untouched).
- The live loop folder's `loop_log.md` START/DONE lines — loop tooling,
  committed with the close-down.
- `loop/autorun-2026-10-01_03-27/plan11_ho_task.md` (untracked, the
  planner's bookkeeping file) — not mine to commit.
- No `TODO.md` edit (worker role); the discrepancy went to
  `todo_inbox.md` via submit (the planner curates + assigns the ID).

## Branch

`opencode_test` (verified via `git branch -v` at session start; all
commits on it). Status: **LANDED**.
