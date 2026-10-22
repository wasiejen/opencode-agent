# Worker summary — state_snapshot digest: LIVE-class marker hits sort first

Status: IN PROGRESS (MAIN unit LANDED + verified; clone v2 unit pending).
Session: `ses_e9b8ed378ffewWRy0cDWYNTkQA` (`worker_FlashNext`), looprun
`autorun-2026-10-01_03-27`. Spec: `agent/handover/specs/2026-10-22_state-
snapshot-marker-order.md` (= `agent/handover/handover_task.md`).

## Task and scope

`markersSection` in the `state_snapshot` planner init digest partitions the
marker-sweep hits into LIVE-CLASS (`maintainer/` prefix, `.opencode/` prefix,
or exactly `TODO.md` / `README.md` / `WIKI.md`) and DOC-CLASS (everything else),
sorts each group by the line string, and emits LIVE first, then DOC. The
20-line cap + `… (+<k> more)` steer, the sweep command, the tool description
and args stay untouched. No new output line shape.

## MAIN files changed (unit 1)

- `.opencode/tools/state_snapshot.ts` — new `isLiveHit` helper + the
  partition/sort in `markersSection` + the two header comments.
- `.opencode/plugin/tests/state_snapshot.smoke.mjs` — FX1 marker fixture is
  now `WIKI.md` (`--main`, LIVE-CLASS) + the existing `loop/notes.md`
  (`--now`, DOC-CLASS); new pin 21 asserts the LIVE hit prints BEFORE the
  `loop/notes.md` DOC hit. Pin count 20 → 21. FX3 and its fixture marker-line
  count untouched.
- `agent/readme/repo_commands.md` §Marker sweep — one note: the digest re-
  orders the sweep OUTPUT only (LIVE first, each group sorted); the ready-
  made command unchanged.

## Behavior changed

The `## markers` section window is now ordered LIVE-first, so live
maintainer/TODO markers can no longer be crowded out by the DOC-heavy `agent/`
hits (MAIN sweep measured 143 hits = 18 LIVE + 125 DOC; the old window carried
ZERO live hits).

## Deviation from the suggested scope (measured, planner to curate)

The spec suggested the FX1 fixture `maintainer/live_note.md`. That fixture
CANNOT pin the ordering: the sweep emits hits in its path ARGUMENT order, so a
`maintainer/` hit already precedes the `loop/notes.md` DOC line pre-change
(the pin written with it passed 21/21 against the UNCHANGED tool — measured).
The worker used a `WIKI.md` fixture (LIVE-CLASS per the spec's own list, prints
AFTER `loop/` raw) → the pin FAILS pre-change, PASSES after. Logged to
`todo_inbox.md` (2026-10-22_18-41) for the planner to correct the spec text.

## Measured verification (MAIN)

- Pre-change baseline: `node .opencode/plugin/tests/state_snapshot.smoke.mjs`
  → `ALL PASS (20/20)`; probe → `PROBE handover: 360/360 PASS` (exit 0).
- Fixture+pin before the tool edit → exit 1, `FAIL markers: the LIVE-class
  WIKI.md hit prints BEFORE the DOC-class loop/notes.md hit [{live:1,doc:0}]`
  (the pin discriminates).
- After the tool edit → `STATE_SNAPSHOT_SMOKE: ALL PASS (21/21)`, exit 0.
- Full v1 gate → 12/12 suites ALL PASS (state_snapshot 21/21, the other
  counts unchanged: auto_resume 170/170, block_transfer 151/151, sandbox
  76/76, compact_memory 110/110, context_recovery 17/17, context_trim 40/40,
  ctx_gauge 3/3, gauge_core ALL PASS, intercept_observer 96/96, loop_log
  69/69, submit 32/32).
- Probe AFTER the edit → `PROBE handover: 360/360 PASS`, exit 0 — count
  IDENTICAL to the pre-edit capture.
- `git status` before the MAIN commit: only the task files + the inbox files
  dirty; `maintainer/SCRATCH_PAD.md` + `opencode.jsonc` are the maintainer's
  own dirty files — NOT staged, NOT committed.

## TODO / inbox entries

- No `TODO.md` entry added (the unit is idea-driven, no TODO id).
- `todo_inbox.md` 2026-10-22_18-41 — the spec-fixture gap above.
- `agent/knowledge/knowledge_inbox.md` 2026-10-22_18-41 — the sweep's raw
  path ARGUMENT order fact.
- `agent/memory/worker/memory_inbox.md` 2026-10-22_18-41 — prove a new pin
  FAILS on the unchanged tool before implementing.

## Deliberately omitted

No `doc-quotes: N` counter line (spec's deliberate non-change); no tool
description/args change (the probe pins the surface); the live host is not
restarted (`.opencode/tools/*.ts` registers at the NEXT host start — the
behavior is smoke-pinned, no live acceptance in this unit).

## Remaining (unit 2 — clone `C:/Users/Wasiejen/Repos/opencode-agent-v2`, branch `migration`)

- `.opencode/tools/state_snapshot.ts` ← copy of MAIN (must stay byte-identical
  to MAIN — `cmp -s`).
- `.opencode/plugins/state_snapshot.ts` (v2) — the twin edit (semicolon style).
- `.opencode/plugins/tests/state_snapshot.smoke.mjs` — FX1 `WIKI.md` fixture
  (already added: 1 write) + the ordering pin + pin (12) line-count 46→47.
- `agent/readme/repo_commands.md` — the twin §Marker sweep note.
- Verify: `bun run state_snapshot.smoke.mjs` ALL PASS + the other v2 smokes,
  `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0, twin `cmp -s` PASS.

Baseline for the clone unit: HEAD `4045331`, clean at session start.
