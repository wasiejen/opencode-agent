# handover_task_to_planner.md — plan40: FST rebind-repeat (WORKER-40, FINAL)

Session ses_f10473509ffeTH0wA3XURsl3SG (worker_Q3S_slow). Task DONE — all
DoD items met. Work in the SEPARATE FST repo `C:\Users\Wasiejen\Repos\Free-Snap-Tap`.

## What changed (the FST commit)
- **`e77c01f`** on new branch **`fst_work3`** (off `opencode_test` @ 57e7fdb;
  created per spec; stale `fst_work`/`fst_work2`/`opencode` branches untouched) —
  code + tests only, 2 files, +110/-4:
  - `fst_keyboard.py` (37 lines): new helper `_rebind_repeat_let_through(current_ke)`
    + the repeat-detection region (the `_all_trigger_events` loop) now gates
    `to_be_suppressed` on it instead of setting it unconditionally. The
    let-through (press-only — detection is press-only, release events
    untouched): a repeated real press is let through ONLY if it is a rebind
    trigger (base-only match: vk_code + is_press, constraints decided by the
    existing rebind loop exactly as for the first press) AND the trigger vk
    is in no tap group AND the replacement vk is in no tap group. Everything
    else (macro/toggle triggers, tap-group keys, tap-consumed replacements,
    missing rebind dict entry) keeps the current suppression.
  - `tests/test_filter_behavior.py` (77 lines): new class `TestRebindRepeat`
    (5 tests) + the existing `test_repeated_trigger_key_is_suppressed_without_refiring`
    repurposed (see Discrepancy below).
  - DO-NOT-touch respected: `###XXX 241016-1101` block + decision comment,
    `###XXX 241022-1341` block, the tap-group section (773-region), and all
    other areas untouched (verified in diff: only the helper + the region).

## Measured gate numbers (this session)
- **Baseline (re-measured on `fst_work3` BEFORE touching code):** pytest
  **459 passed, 1 warning** (the known #10 coroutine warning), ruff
  **F=0** ("All checks passed!"). Matches the spec's expected numbers.
- **Final (post-commit, same commands):** pytest **464 passed, 1 warning**,
  ruff **F=0**. Delta = exactly the 5 new tests.

## Edge-case → test mapping (all in `TestRebindRepeat` unless noted)
| proposal/spec edge case | test |
|---|---|
| core new behaviour: held rebind key auto-repeats its target | `test_repeated_rebind_trigger_repeats_target_key` (target pressed twice, source suppressed on both events) |
| rebind → `SUPPRESS_CODE` stays suppressed on repeats | `test_rebind_to_suppress_stays_suppressed_on_repeat` (no presses, 2 suppressions) |
| key BOTH rebind + macro trigger: first press = rebind, repeats = target only, macro never fires | `test_rebind_and_macro_trigger_repeats_target_only_macro_never_fires` (macro spy: 0 firings first press AND repeat) |
| press-state bookkeeping consistent across repeats | `test_rebind_press_state_bookkeeping_stays_consistent_on_repeat` (pressed-set tracks B not A, idempotent remove+re-add; real press state of held A stays True) |
| **MAINTAINER RULING (MUST pin): rebind trigger in a tap group does NOT repeat** | `test_rebind_trigger_in_tap_group_does_not_repeat` (replacement c sent exactly ONCE — no auto-repeat; both source events suppressed; tap group then owns the held key itself, exactly as current behaviour) |
| macro/toggle repeat suppression unchanged (repurposed existing test) | `test_repeated_trigger_key_is_suppressed_without_refiring` (now built as a MACRO trigger: fires once, repeat suppressed) |

## How the tap-group handling differs from the failed first attempt
- First attempt's change #1 let repeated rebind triggers through
  UNCONDITIONALLY (no tap-group exclusion) and change #2 touched the
  tap-group section (setting `to_be_suppressed` when `key_replaced`).
  That combination is the observed tap-group test failure.
- My implementation: the tap-group section is UNTOUCHED (the ruling makes
  the second change unnecessary; also spec DO-NOT-touch). The let-through
  decision lives entirely in the press-only repeat-detection region:
  rebind triggers that are in a tap group fall through to the old
  `to_be_suppressed = True` — their repeats stay suppressed, their
  replacement is never auto-repeated, and the tap section handles the held
  key exactly as today (pinned by the ruling test).
- Extra conservative exclusion (not in the proposal, not in the first
  attempt): a rebind whose REPLACEMENT vk is in a tap group also stays
  suppressed. Traced: otherwise the let-through would leave `key_replaced`
  reset by the tap section and `to_be_suppressed` False → the source key's
  repeats would LEAK through to the OS while the replacement sits in the
  tap state. Not covered by the proposal → conservative keep-current per
  the spec's approval-boundary clause.

## Discrepancy found (flagged, handled per the approval boundary)
- The spec's verified fact "no existing test references
  `real_input_repeated` / `to_be_suppressed` — your pinning tests are
  greenfield" is true for the flag names, but `test_repeated_trigger_key_is_suppressed_without_refiring`
  pinned the OBSERVABLE old rebind behaviour (repeat of `a→b` rebind sends
  the target once). The approved proposal changes exactly that behaviour,
  so the test could not stay green unchanged. I repurposed it (same name)
  to pin the unchanged part — macro-trigger repeat suppression — and put
  the new rebind behaviour in the new class. If the planner prefers a
  differently-worded replacement, the diff is self-explanatory.

## TODO entry
- NEW entry appended to `projects/Free-Snap-Tap/TODO.md` (agent repo, under
  "FST code & tests"): status **LANDED**, per the spec the commit hash is
  recorded by the planner in the follow-up bookkeeping commit (not by me).
  I deliberately did NOT assign a global ID (counter lives in the root
  `TODO.md`; planner assigns at curation) — entry references plan40/worker-40.

## FST repo state
- `fst_work3` @ `e77c01f` (the only commit on it); tree back on
  `opencode_test`, CLEAN. Nothing pushed (push policy).

## Deliberately not done
- No live-listener verification (test-gate: mocked pattern only).
- No behaviour change outside "rebind key auto-repeats its target" with the
  ruling's tap-group exclusion + the conservative tap-consumed-replacement
  exclusion above.
- The agent-repo commit for this close contains exactly: this handover,
  the FST TODO note, my `loop_log.md` START+DONE lines (close-down rule),
  and the `agent/agent_feedback.md` friction entry (submit tool). The
  maintainer's uncommitted `maintainer/ideas/ideas.md` edit was left
  untouched.
