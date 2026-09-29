# TODO — Free-Snap-Tap (per-project entries)
IDs continue the global sequence — counter in the root `TODO.md` (used so far up to #118, new entries start at #119).
Full texts of closed entries: `todo_records.md` (this folder). FST repo docs: `repo/` (this folder).
Moved out of the root `TODO.md` on 2026-09-29 (#117 per-project split).

## FST behavior decisions (closed)

## 1. (closed 2026-09-12, iter-9 unit A `4b93d37` on `fst_work`, full text in todo_records.md) — General vk resolution: unknown keys surface as ONE user-visible error at every vk-resolution site (P08 error toast / headless print) via the shared `FST_Keyboard.surface_config_error` helper; both constraint fail-closed guards + `check_for_combination` (+ hot-path resume catch) route through it, dedup + fail-closed preserved; unknown constraint *names* stay silent no-ops by design.

## 7. (closed 2026-09-12, iter-9 unit B `2891dab` on `fst_work`, full text in todo_records.md) — Empty macro: BEHAVIOR WINS — the trigger key IS suppressed for an empty key group (kept); the stale comment reworded to match ("supress" fixed); pinned by the pre-existing `test_empty_macro_sequence_no_playback` (no-playback AND suppression).

## 8. (closed 2026-09-12, iter-9 unit B `2891dab` on `fst_work`, full text in todo_records.md) — `ap`/`ar` "all keys (incl simulated)" = UNION (real OR simulated): both setters write the `all` dict as `is_press or <other side's state>` + the symmetric `vk_code > 0` guard on the real setter; doc comments state the union; 3 new pinning tests (crossing release both directions + vk guard).

## 9. (closed 2026-09-12, iter-9 unit B `2891dab` on `fst_work`, full text in todo_records.md) — Repeat-constraint excepts hardened: `ValueError` added to `toggle_repeat`/`is_repeat_active`/`reset_repeat`/`stop_all_repeat` (+ `stop_repeat` as the fifth consistency site); one pinning test covers all five methods with malformed (1-/3-element) entries.

## 4. (closed 2026-09-12, iter-9 unit B `2891dab` on `fst_work`, full text in todo_records.md) — Dead `elif result is None: pass` branch deleted from `check_constraint_fulfillment` (unreachable — `constraint_evaluation` normalizes None→True); the A→C triage reclassification stays maintainer-side (`COVERAGE_TRIAGE.md` is agent-read-only).

## 6. (closed 2026-09-12 by maintainer ruling — KEEP, full text in todo_records.md) — `fst_keyboard.py` 302-303 (mixed-Key rebind conversion) RULING-KEEP ("keep this until I can test a bit more"): the `###XXX 241022-1341` block stays and is DO-NOT-TOUCH; the triage class correction stays maintainer-side per #4.

## 11. (closed 2026-09-15, see todo_records.md) — General contradiction prevention — maintainer ruling D1-A (2026-09-15_backlog-decisions.md): kept OFF as an intentional decision (no re-enable); decision comment added below the untouched XXX 241016-1101 pin in fst_keyboard.py; the pinning tests stay the semantic pin.

## 48. (closed 2026-09-11, first commit after `00bc24f`, see todo_records.md) — Packed-word equality checks in the mouse filter: X-button mouseData + LLKHF flags (2026-09-10, #42 report-back)

## FST code & tests (closed)

## 46. (closed 2026-09-10, see todo_records.md) — Flaky test: `test_crossover_not_taken_on_low_roll` — timing/order-dependent (2026-09-10)

## 41. (closed 2026-09-10, see todo_records.md) — Production bug: `remove_all_toasts()` control function calls a nonexistent attribute (plural/singular mismatch) (2026-09-10)

## 42. (closed 2026-09-10, see todo_records.md) — Multi-notch scroll-wheel events (delta ≠ ±120) are untested across all layers; the filter pins wheel phase on single-notch equality (2026-09-10, Audit 3a)

## 43. (closed 2026-09-10, see todo_records.md) — `kb_env` fixture + `build()`/`down()` helpers are copy-pasted (drifted) across 6 test files — no shared conftest location (2026-09-10, Audit 3a)

## 44. (closed 2026-09-10, see todo_records.md) — Stale/unknown focus name → uncaught KeyError in `apply_focus_groups` / `apply_start_args_by_focus_name` (the config is reloaded *before* the lookup) (2026-09-10, Audit 3b)

## FST docs (closed)

## 3. Rework README and WIKI to the current state of the code (2026-09-06) (closed 2026-09-10, see todo_records.md)

## 47. (closed 2026-09-10, see todo_records.md) — Docs: §3 undocumented features (variable system, invocations, extra start args, numpad debug combos) (2026-09-10, from the #3 residual)

## 45. (closed 2026-09-10, see todo_records.md) — Doc errors found adjacent to the #3 rework: WIKI invocation "evaluate to False" claim, WIKI `+a, +b` rebind notation, README "he first" (2026-09-10)

## FST environment (open)

## #113. (open, 2026-09-28, plan29 planner gate run — ENVIRONMENT break, maintainer call) the repo venv's python.exe is broken — the base interpreter it points at is gone, so the standard gate's pytest + the probe's numword python checks are unrunnable
- **Problem / evidence:** the plan29 gate run (2026-09-28) measured:
  `./.venv/Scripts/python.exe -c "print(...)"` fails with
  `No Python at '"C:\Users\Wasiejen\Projects\OpenCodeProjects\Free-Snap-Tap\python312\python.exe"'`
  (the error text carries an embedded quote). `.venv/pyvenv.cfg` points
  `home`/`executable` at `C:\Users\Wasiejen\Projects\OpenCodeProjects\Free-Snap-Tap\python312`
  (ONE level above the repo root); that `python312` dir exists but holds
  only `Doc` — `python.exe` is ABSENT. Consequence measured in the same
  run: the probe's 11 numword python checks (136-146, `runPyW2n` over
  VENV_PY) all FAIL with that message; the standard gate's pytest half is
  unrunnable (the venv is the only python with the repo deps — bare PATH
  `python` = 3.14 without deps per repo_commands.md). The 2026-09-27
  baselines (probe 345, pytest 459+1w) predate the break.
- **Desired outcome:** the venv's python works again (re-pointed or
  rebuilt by the maintainer — repo_commands.md forbids agents from
  reinstalling the venv from scratch); then one full standard-gate re-run
  to re-establish the baselines (probe total + pytest count).
- **Acceptance:** `./.venv/Scripts/python.exe -c "print(...)"` works;
  standard gate green (probe self-annotated total + pytest count) with the
  fresh counts recorded in the NAP Standing baselines.
- **Suggested scope:** maintainer domain (the venv + the base python
  install); the planner re-baselines on the next gate run.
- **Status:** OPEN — ENVIRONMENT FIXED (2026-09-29, round 2): the venv
  is REBUILT on 3.12.9 (verified 02:00 — `pyvenv.cfg` → the proper
  `AppData\Local\Programs\Python\Python312` install, 459 tests collect)
  after an interim 3.14.3 replacement (the original venv had been
  REPLACED, not repaired — the `..\python312` base dir was gone from
  disk; the 3.14 slip = the bare-`python`/`py` default, `py -0` marks
  3.14 as `*`). Remaining: ONE full standard-gate re-run (worker or
  next autorun) to re-baseline (probe total incl. the 11 numword checks
  136-146 + pytest count) and close this.
