# Worker handover — TODO #127: compact-message-delivery item 4 (restart-branch inheritance)

Worker: worker_Q3S (llama-swap/Qwen3.8-27B-Q3S-170K), session ses_f0ad0bbfcffeCKb4FmU2gK7c29.
2026-10-01. Task spec: `agent/handover/handover_task.md` (the committed #127 spec).
State: DONE — code + smoke committed (unit commit 5ed36cf), standard gate green, live acceptance pending natural occurrence.

## What changed

`.opencode/plugin/auto_resume.ts` (the only code file touched):
- New helper `restartIntentSection(sid)` (placed after `restartText`): reads
  `.opencode/temp/compact_message_<sid>` (the SAME `queuedMessagePath` /
  `logDir` the item-2 relay uses). If present and non-empty: renames the
  file to `.consumed` (best-effort, item-3 tombstone convention), logs
  `intent= sid=`, and returns a labeled section — a line naming the closing
  session's queued message, consumed by the restart branch, to be treated
  as an INTENT HINT (verify state from committed state before acting) —
  then the message content.
- Fail-open: file absent / empty / unreadable → returns "" (the base text
  unchanged, no throw, no tombstone).
- Call site (the restart branch, was L1409):
  `spawnPlanner(restartText(sid, exhausted) + restartIntentSection(sid), msgs)`.
  `restartText` itself is UNCHANGED (stays pure string construction — the
  spec offered both options; this one keeps the pure function pinned as-is).

`.opencode/plugin/tests/auto_resume.smoke.mjs` — new section (5) right after
the item-11 pins, in the unit-4 test block:
- Pin (b): file ABSENT → the restart spawn text is BYTE-IDENTICAL to the
  base (pinned on the existing emg scenario; `BASE_RESTART` exact base
  transcribed in-test — machine-verified by the pin itself).
- Pin (a): file PRESENT → the spawn text starts with the base, carries the
  labeled "closing-intent hint" section (names the sid), ends with the
  queued message; the file is renamed `.consumed`; the `intent= sid=` line
  is logged (3 asserts for scenario (a)).
- Pin (c): UNREADABLE file (a DIRECTORY at the queue path — readFileSync
  throws) → base text unchanged, no tombstone (the directory survives).

## Measured verification

- auto_resume smoke: **152/152 ALL PASS** (baseline 147 + my 5 pins; the 3
  required DoD scenarios covered, scenario (a) split into section + tombstone
  + log-line asserts — deviation from the spec's "3 new pins / 150 total",
  strictly more coverage, all existing 147 unchanged).
- Standard gate: **probe 352/352 PASS** + all 11 smokes green (auto_resume
  152, block_transfer.sandbox 64, block_transfer 131, compact_memory 82,
  context_recovery 17, context_trim 26, ctx_gauge 3, gauge_core ALL PASS,
  intercept_observer 78, loop_log 69, submit 31).
- Existing-pins audit (spec requirement): NO pin asserts the exact full
  restart text — the #90 byte pin checks only line 1 (the toggle) + line-2
  prefix; the other restart pins are substring checks. The absent-file
  case is byte-identical ("" appended), confirmed by the new pin (b).

## Commits

- `5ed36cf` — code + smoke + loop-log START lines (the verified unit).
- THIS commit — TODO #127 status LANDED + this handover + the loop-log DONE
  line. (The final hash rides the planner's follow-up bookkeeping commit.)

## Live acceptance (DoD 5 — named)

The next RESTART-BRANCH spawn with a queued `compact_message_<sid>` present
(natural occurrence: a session that stores a compact_memory message at
queue time, then closes with `action: restart` after compaction). Detector:
the `intent= sid=…` line in `.opencode/temp/auto_resume.log` + the
`compact_message_<sid>.consumed` tombstone. No live occurrence yet (the
current queue files in `.opencode/temp/` predate the build — they ride the
item-2 relay path or the age sweep, not the restart branch).

## Deliberately NOT done

- `.opencode/plugin/auto_resume.md` (the plugin doc) was NOT updated with an
  item-4 line — outside the spec's DoD (docs are pre-approved but not
  required; left for the planner's bookkeeping judgment).
- No other file touched: relay path, items 1-3 behavior, the budget file,
  `maintainer/`, `agent/prompts/**` — all untouched.
- No push (policy: agents never push).

## Notes for the planner

- The rename happens at READ time (before the spawn attempt) per the spec's
  design — even if the spawn later fails, the tombstone is already in
  place (the queued message was consumed by the restart-branch text
  construction). Consistent with the spec wording ("append … then rename");
  noted here so the live-acceptance read doesn't surprise.
- The untracked `loop/autorun-2026-10-01_03-27/plan2_ho_task.md` (the planner's
  own spec copy) was left uncommitted — yours to ride your bookkeeping.
