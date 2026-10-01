# TODO — Free-Snap-Tap (per-project entries)
IDs continue the global sequence — counter in the root `TODO.md` (used so far up to #131, new entries start at #132).
Full texts of closed entries: `todo_records.md` (this folder). FST repo docs: `repo/` (this folder).
Moved out of the root `TODO.md` on 2026-09-29 (#117 per-project split).

## FST code & tests (open)

## #129. (open, 2026-10-01, maintainer inbox `FST_bugs.md`) — Indicator context-menu crash: right-click on the status indicator raises AttributeError (`QContextMenuEvent` has no `globalPosition`)
- **Problem / evidence:** right-clicking the status indicator (context menu) opens no menu and raises: `AttributeError: 'PySide6.QtGui.QContextMenuEvent' object has no attribute 'globalPosition'` at `fst_overlay.py:595` in `contextMenuEvent` → `self.context_menu.exec_(event.globalPosition().toPoint())`.
- **Desired outcome:** right-click opens the indicator context menu without the error.
- **Acceptance criteria:** the menu opens on right-click (maintainer live test); no AttributeError in the console; a pinning test where the repo's test harness can cover the overlay event path.
- **Suggested scope:** `fst_overlay.py` `contextMenuEvent` (~L595) — replace `globalPosition()` with the correct PySide6 position accessor for a context-menu event (local `position()` vs global coordinates — the worker verifies against the installed PySide6 version + the menu's exec_ semantics).
- **Status:** open — maintainer bug report (inbox 2026-10-01, FST_bugs.md).

## #130. (open, 2026-10-01, maintainer inbox `FST_bugs.md`) — First toast of a session lags input ~2 s (mouse inclusive); retriggers are instant
- **Problem / evidence:** the first trigger of a function that sends a toast (displayed under the status indicator) lags input extremely for ~2 s (mouse inclusive) until the toast is displayed; retriggering the same macro shows no such delay (toast appears immediately). Toast causation is the maintainer's current guess (the only observable change).
- **Desired outcome:** no perceptible input lag on first-toast display; root cause identified (a first-time initialization on the toast path — e.g. font/layout/overlay lazy init — or another first-call cost).
- **Acceptance criteria:** root cause identified + measured (timing evidence); first-toast trigger instant (maintainer live test); a pinning test where possible.
- **Suggested scope:** the toast display path in the FST code (overlay/toast module — the worker maps it) + first-call initialization suspects (fonts, geometry, Qt lazy resources).
- **Status:** open — maintainer bug report (inbox 2026-10-01, FST_bugs.md).

## #131. (open, 2026-10-01, maintainer inbox `FST_logging.md`) — DEBUG→logging research: switch `CONSTANTS.DEBUG*` console output to structured logging with clear separation + level/file control
- **Problem / evidence:** the maintainer used `CONSTANTS.DEBUG<X>` flags (4 distinct values) as live console debugging, accreted ad hoc over the years (some output is convenience-only or leftover from fixed bugs). He started integrating logging but is unsure whether/how it would be more helpful than the DEBUG constants. `CONSTANTS.DEBUG4` is the notable exception: the formatted key-flow print (pressed key → replaced key → macro keyactions, indented hierarchy, `<--`/`-->` control-flow markers).
- **Desired outcome:** a research doc: how to instrument the whole repo so logging helps find lag causes, documents errors, and associates them to code areas — without overwhelming; control via start-argument log level and/or separate log files per area; what the DEBUG4 flow print should become (kept as a formatted log); which DEBUG outputs die.
- **Acceptance criteria:** a research doc (`projects/Free-Snap-Tap/` or `agent/research/`) with a concrete logging design (levels, files, start-argument control) + a per-DEBUG-flag disposition list (keep/convert/kill) + a recommendation on whether logging beats the DEBUG constants for his use case.
- **Suggested scope:** read-only over the FST code (grep `CONSTANTS.DEBUG`), the research doc; the build (if approved later) = an FST worker on `fst_work3`.
- **Status:** open — maintainer research request (inbox 2026-10-01, FST_logging.md).

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

## #122. (LANDED 2026-09-30, plan40 worker-40, commit `e77c01f` on `fst_work3` — live FST testing = maintainer domain) — Held REBIND keys auto-repeat their target key (approved proposal 2026-09-12_fst-rebind-repeat + maintainer tap-group ruling 2026-09-15): the let-through is press-only in the repeat-detection region (new helper `_rebind_repeat_let_through`: the trigger is in no tap group AND its replacement is in no tap group — the tap-group section is untouched); 5 pinning tests added in `tests/test_filter_behavior.py` (`TestRebindRepeat`, incl. the MUST-pin tap-group ruling case) + the old `test_repeated_trigger_key_is_suppressed_without_refiring` repurposed to pin macro-repeat suppression (it pinned the old rebind behaviour, which the approved proposal changes); FST gate 464 passed + 1 warning, ruff F=0 (baseline 459+1w). FST branch `fst_work3` (off `opencode_test` 57e7fdb).

## FST docs (closed)

## 3. Rework README and WIKI to the current state of the code (2026-09-06) (closed 2026-09-10, see todo_records.md)

## 47. (closed 2026-09-10, see todo_records.md) — Docs: §3 undocumented features (variable system, invocations, extra start args, numpad debug combos) (2026-09-10, from the #3 residual)

## 45. (closed 2026-09-10, see todo_records.md) — Doc errors found adjacent to the #3 rework: WIKI invocation "evaluate to False" claim, WIKI `+a, +b` rebind notation, README "he first" (2026-09-10)

## FST environment (closed)

## #113. (closed 2026-09-30, plan40 planner-40 — full text in todo_records.md) — the repo venv's python.exe was broken (base interpreter gone) — venv REBUILT on 3.12.9 by the maintainer (2026-09-29) + the agent-repo probe's VENV_PY re-pointed at the FST venv (fallback, agent-repo venv preferred if present) — probe 352/352 green, FST gate 459+1w + ruff F=0 re-established
