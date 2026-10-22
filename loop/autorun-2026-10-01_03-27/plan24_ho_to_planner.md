# Worker summary — state_snapshot digest: LIVE-class marker hits sort first

Status: LANDED (both units verified + committed; the clone-side commit hash
is recorded below after the clone commit landed).
Session: `ses_e9b8ed378ffewWRy0cDWYNTkQA` (`worker_FlashNext`), looprun
`autorun-2026-10-01_03-27`. Spec: `agent/handover/specs/2026-10-22_state-
snapshot-marker-order.md` (= `agent/handover/handover_task.md`).

## Task and scope

`markersSection` in the `state_snapshot` planner init digest partitions the
marker-sweep hits into LIVE-CLASS (`maintainer/` prefix, `.opencode/` prefix,
or exactly `TODO.md` / `README.md` / `WIKI.md`) and DOC-CLASS (everything
else), sorts each group by the line string, and emits LIVE first, then DOC.
The 20-line cap + `… (+<k> more)` steer, the sweep command, the tool
description and args are untouched. No new output line shape, no new header
or steer text.

## Files changed

MAIN (commit `97714b5`):
- `.opencode/tools/state_snapshot.ts` — new `isLiveHit` helper + the
  partition/sort in `markersSection` + the two header comments (206 → 234 lines).
- `.opencode/plugin/tests/state_snapshot.smoke.mjs` — the FX1 marker fixture is
  now `WIKI.md` (`--main`, LIVE-CLASS) alongside `loop/notes.md` (`--now`,
  DOC-CLASS); new pin 21 asserts the LIVE hit prints BEFORE the `loop/notes.md`
  DOC hit. Pins 20 → 21. FX3 + its fixture marker-line count untouched.
- `agent/readme/repo_commands.md` §Marker sweep — the note: the digest re-
  orders the sweep OUTPUT only; the ready-made command unchanged.

Clone `C:/Users/Wasiejen/Repos/opencode-agent-v2` (branch `migration`):
- `.opencode/tools/state_snapshot.ts` — copied from MAIN (byte-identical, 234
  lines, `cmp -s` PASS after the change).
- `.opencode/plugins/state_snapshot.ts` (v2) — the twin edit in the file's
  semicolon style: `isLiveHit` + the partition/sort + the markers header
  comment + a header note in the CARRIED block (265 → 293 lines).
- `.opencode/plugins/tests/state_snapshot.smoke.mjs` — FX1 `WIKI.md` fixture
  + the ordering pin + pin (12) FX1 digest line count 46 → 47 (the fixture
  adds one sweep hit) + the header-comment line for the pinned contract.
- `agent/readme/repo_commands.md` — the twin §Marker sweep note (identical text).

## Behavior changed

The `## markers` window is LIVE-first, so live maintainer/TODO markers can
no longer be crowded out by the DOC-heavy `agent/` hits. MAIN sweep measured
143 hits = 18 LIVE + 125 DOC; the old 20-line window carried ZERO live hits.

## Deviation from the suggested scope (measured — planner to curate)

The spec suggested the FX1 fixture `maintainer/live_note.md`. That fixture
cannot pin the ordering: `grep -rn` emits hits in its path ARGUMENT order, so
a `maintainer/` hit already precedes the `loop/notes.md` DOC line pre-change.
Measured: the pin written with that fixture passed `ALL PASS (21/21)` against
the UNCHANGED tool (vacuous pin). The worker used a `WIKI.md` fixture instead
(LIVE-CLASS per the spec's own list, prints AFTER `loop/` raw), which yields
the FAIL→PASS sequence that pins the behavior. Logged to `todo_inbox.md`
(2026-10-22_18-41); the spec text still names the weaker fixture.

Clone-side pin placement: label `(23)` sits inside the FX1 block (after pin
(10)) so the existing `(11)`..`(22)` labels — incl. the FX3 pins the spec
forbade touching — stay intact; the label is append-only, not positional.

## Measured verification

MAIN:
- Baseline (pre-edit): smoke `ALL PASS (20/20)`; probe `PROBE handover:
  360/360 PASS`, exit 0.
- Fixture+pin before the tool edit: exit 1 —
  `FAIL  markers: the LIVE-class WIKI.md hit prints BEFORE the DOC-class
  loop/notes.md hit  [{"live":1,"doc":0}]` (the pin discriminates).
- `node .opencode/plugin/tests/state_snapshot.smoke.mjs` after the tool edit
  → `STATE_SNAPSHOT_SMOKE: ALL PASS (21/21)`, exit 0 — PASS.
- `node .opencode/plugin/tests/*.smoke.mjs` (12 files) → 12/12 `ALL PASS`,
  the other suites' counts unchanged (auto_resume 170/170, block_transfer
  151/151, sandbox 76/76, compact_memory 110/110, context_recovery 17/17,
  context_trim 40/40, ctx_gauge 3/3, gauge_core ALL PASS,
  intercept_observer 96/96, loop_log 69/69, submit 32/32) — PASS.
- `node .opencode/plugin/probes/handover_probe.mjs` after the edit →
  `PROBE handover: 360/360 PASS`, exit 0 — count IDENTICAL to the pre-edit
  capture — PASS.

Clone (`C:/Users/Wasiejen/Repos/opencode-agent-v2`, branch `migration`):
- Pre-change plugin + the edited smoke (`bun run
  .opencode/plugins/tests/state_snapshot.smoke.mjs`) → exit 1 —
  `FAIL  (23) markers ordering ... [[2,0]]` (raw order: DOC at 0, LIVE at 2),
  26 PASS — the pin discriminates.
- Edited v2 plugin → `SMOKE: ALL PASS`, exit 0, 27 PASS lines (pins 26 → 27)
  — PASS.
- Every `*.mjs` in `.opencode/plugins/tests` (11 files) run with `bun run` →
  all `SMOKE: ALL PASS`, loop exit 0 — PASS.
- `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p
  .opencode/plugins/tests/tsconfig.json` from the clone root → exit 0,
  0 output lines — PASS.
- `cmp -s .opencode/tools/state_snapshot.ts
  ../opencode-agent/.opencode/tools/state_snapshot.ts` → identical (234 lines
  each) — PASS.
- `git status` of the clone before its commit: only the 4 task files dirty.

Working tree: MAIN clean except the maintainer's own
`maintainer/SCRATCH_PAD.md` + `opencode.jsonc` (never staged, never committed
— the SCRATCH_PAD is a maintainer file). Clone clean except its 4 task files
before the clone commit.

Commits: MAIN unit `97714b5` (branch `main`); clone unit `aabc705` (branch
`migration`); this summary + the loop-log DONE line ride the MAIN
bookkeeping commit that follows.

## TODO / inbox entries

- No `TODO.md` entry added (the unit is idea-driven, no TODO id; `TODO.md`
  unchanged in both trees).
- `todo_inbox.md` 2026-10-22_18-41 — the spec-fixture gap (above), for the
  planner to correct the spec text + the clone twin suggestion.
- `agent/knowledge/knowledge_inbox.md` 2026-10-22_18-41 — the sweep's raw
  path ARGUMENT order fact (measured both trees).
- `agent/memory/worker/memory_inbox.md` 2026-10-22_18-41 — prove a new pin
  FAILS on the unchanged tool before implementing.
- `agent/agent_feedback.md` 2026-10-22_18-48 — the vacuous-pin friction (the
  spec-suggested fixture) + the action (require the pin to FAIL pre-change).
- `agent/knowledge/knowledge_inbox.md` 2026-10-22_18-48 — the v2 smoke root-
  carrier trap (`dir` = the LAST setup's capture → an appended FX1 pin read
  the fx3 root, `[-1,-1]`).

## Deliberately omitted

- No `doc-quotes: N` counter line (the spec's deliberate non-change).
- No change to the tool description or args (the probe pins the SURFACE; the
  probe count stayed 360).
- FX3 fixtures and their marker-line counts untouched in both trees.
- No live acceptance: `.opencode/tools/*.ts` registers at the NEXT host start;
  the digest behavior is smoke-pinned (no host restart in this unit).
- No push (agents never push).

## Lesson (one line)

A new pin must be proven DISCRIMINATING — run it against the unchanged tool
and require the FAIL — before the implementation lands.
