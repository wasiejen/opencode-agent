# TODO records — Free-Snap-Tap closed entries
Moved out of the root `todo_records.md` on 2026-09-29 (#117 per-project split).
One-line records + full texts; resolution lives in the file / git log.
Numbering: IDs continue the global sequence — counter in the root `TODO.md`.
The agent-repo records stay in the root `todo_records.md`.

## 2. Fix the 6 ruff `F` findings — CLOSED (worker, `cdbbdcd`, 2026-09-10) — all six F sites removed (incl. the cascaded dead `cube_distance`); ruff F 6→0; pytest 434→434, 13 warnings same profile.

## 5. Lint baseline 6 → 8 at `ca61a26` — CLOSED (`0025a57`, 2026-09-08) — the three unused `SimpleNamespace` imports removed — the 6-finding baseline restored (later → 0 via #2).

## 10. deprecated `QMouseEvent.globalPos()` — CLOSED (inline fix `ea3d920`) — 3× `globalPos()` → `globalPosition().toPoint()`; pytest 434/434; warnings 13→1 (verified 2026-09-10 by grep: no `globalPos(` call left in the code).

## 47. Docs: §3 undocumented features - CLOSED (looprun 2, iteration 4, docs commit after c4ad33c) - all §3 docs landed (b40a1a7 + 73c0097) + the two residual WIKI lines (shared text/integer variable name-space; horizontal scroll press=right/release=left) applied from the settled planner-verified answers; gate 436 passed / ruff F=0.

## 47. Docs: §3 undocumented features (variable system, invocations, extra start args, numpad debug combos) (2026-09-10, from the #3 residual) (closed 2026-09-10, full text moved from TODO.md)

- **Problem / evidence:** the #3 rework closed the §2+§4 gap items; the §3 scope
  (SPEC_FEATURES.md §3) remains undocumented: the variable system, typing/toast/mouse/
  clipboard/file invocations, extra start args, numpad debug combos.
- **Outcome (goal):** README/WIKI document the §3 features at the same quality bar as
  the §2/§4 rework (every claim code-verified before rewording, per the #3 method).
- **Acceptance:** all §3 features documented in README/WIKI; docs-only diff; suite
  unaffected (434+ passing, ruff F=0).
- **Scope (non-exhaustive):** `README.md`, `WIKI.md`, `SPEC_FEATURES.md` (decisions
  source); `fst_manager.py` / `fst_keyboard.py` / `free_snap_tap.py` (read-only
  verification).
- **Status:** OPEN — APPROVED (maintainer 260910-1252); docs-only, not a maintainer
  call. Delegation-ready — scheduled for looprun 2 iteration 2 (the iteration-1
  approved-fix batch takes the delegation lane first).
- **Status tail:** PARTIAL (2026-09-10) — WIKI [Configuration] chunk landed (multi-focus
   names + multiline `:` continuation, both code-verified); all §3 features were
   code-verified before the session hit the context stop line — the remaining docs
   (invocations sections, extra start args, numpad combos, vk-0 key strings, README list)
   are NOT yet written. Full per-feature verification notes (ready to write from) are in
   `.opencode/handover_task_to_planner.md`; re-run the gate on the final chunk.
- **Status tail:** LANDED (2026-09-10, resume run) — all remaining §3 docs written from
   the verified notes: WIKI new invocation sections ([Function results in general],
   [Variable system], [Typing], [Toasts], [Mouse control], [Mouse keys], [Clipboard],
   [File operations], [Misc]), WIKI [Numpad debug combos] (ALT+NUM1..NUM8, NUM6
   unassigned), WIKI [None/empty '' key event] extended to the vk-0 strings
   `none`/`NONE`/`_`/`reset`/`delay`, WIKI [Start Arguments] extended (-delay,
   -exec_one_macro, -debug_numpad, -always_active, -tray_icon, -hide_cmd_window,
   -save_dir=, -backup_root_dir=, deprecated -focusapp= + argument order note), README
   feature list item 14 [Extra Start Arguments]. The four confirmed-behavior flags are
   documented as-is (headless toast crash, ignored `immediately` param, unassigned
   ALT+NUM6, `check()` non-int/non-list value passing). Gate: `pytest -q` = 436 passed,
    1 known warning; `ruff check --select F .` = 0 findings; docs-only diff.
 - **Planner-verified (looprun 2, iteration 2, `73c0097`):** gate re-run green (436 / ruff
    0); `git diff` scope = README/WIKI/TODO/summary only. Core §3 goal MET. RESIDUAL
    (trivial, both answers code-verified in the NAP — no re-check needed): add to the WIKI
    (a) that text vars (`set_var`/`get_var`) and integer vars share the SAME
     `Output_Manager.variables` dict (`fst_manager.py:71`), and (b) horizontal scroll sends
     press = one notch RIGHT / release = one notch LEFT (`fst_manager.py:705-706`).
  - **CLOSED (looprun 2, iteration 4):** both residuals applied to the WIKI from the
    settled answers (no re-check): shared text/integer name-space line under
    [Variable system] + the horizontal scroll direction line under [Mouse keys].
    Gate: `pytest -q` = 436 passed, 1 known warning; `ruff check --select F .` = 0.
    Entry closed — one-line record stays in `todo_records.md`.

## 45. Doc errors found adjacent to the #3 rework: WIKI invocation "evaluate to False" claim, WIKI `+a, +b` rebind notation, README "he first" (2026-09-10) (closed 2026-09-10, full text moved from TODO.md)

One-line record: fixed in the adjacent commit of the #3 docs rework — WIKI [Suffixes] function-invocation description corrected (invocations always evaluate to True and the suffixed key_event is still played — all invocations return True in `constraint_evaluation`, cf. WIKI's own "ALL INVOCATIONS will always result in True"); WIKI [Rebinds] example `+a, +b` → `+a : +b` (the 2nd of the two Key:Key rebinds, `fst_keyboard.py` Key-pair expansion); README config comment "before he first <focus>" → "before the first <focus>".

## 43. `kb_env` fixture + `build()`/`down()` helpers are copy-pasted (drifted) across 6 test files — no shared conftest location (2026-09-10, Audit 3a) (closed 2026-09-10, full text moved from TODO.md)

- **Problem / evidence:** the identical FST-Keyboard-env fixture concept
  (`FST_Keyboard()`, mocked pynput controllers, `TIME_DIFF/START_TIME` reset,
  `_listener` mocks, TIME restoration teardown) exists SIX TIMES in THREE
  drifted shape-classes: (A) yields `SimpleNamespace(kb, kb_mock, mouse_mock)`
  + pre-sets `WIN32_FILTER_PAUSED/ACT_DELAY/ACT_CROSSOVER` (NO
  `_mouse_listener` mock): `test_filter_behavior.py:27-41`,
  `test_extraction_filter_edges.py:26-40`; (B) yields the raw `keyboard`,
  pre-sets the arg flags AND `_mouse_listener`: `test_filter_simulated.py:23-38`;
  (C) yields the raw `keyboard`, `_mouse_listener` mocked, NO arg flags:
  `test_control_actions.py:17-29`, `test_macro_playback_kbd.py:18-30`,
  `test_facade_wiring.py:15-27`. Shape (C) fixtures silently differ from (A): a
  `WIN32_FILTER_PAUSED=True` filter would behave differently per file — a
  fixture bug fix has to be applied 6× and per-variant. The
  config-build helper `build(kb, rebinds=None, macros=None, taps=None, aliases=None)`
  (setting `config_manager._*_hr` + `initialize_groups_from_presorted_lines()`)
  is triplicated at `test_filter_behavior.py:44-50`, `test_extraction_filter_edges.py:43-50`,
  `test_filter_simulated.py:41-…`; `down()`/`up()` filter-event shorthands
  duplicated at `test_filter_behavior.py:59-64` and `test_extraction_filter_edges.py:52-54`;
  `hold_keys`/`mock_control_handlers` at `test_filter_behavior.py:290-297`. The
  variants already DRIFT (SimpleNamespace vs. raw-yield fixtures; `_mouse_listener`
  set in some copies, not others) — a behavioral test in the drifted variants is
  invisible from the others, and a fixture bug fix has to be applied 6×.
- **Outcome (goal):** ONE shared location for the `kb_env`-class fixtures and the
  `build`/`down`/`up`/`hold_keys`/`mock_control_handlers` helpers (e.g. a
  `tests/conftest.py` fixture + tiny helper importable or duplicated in ONE file);
  no test file defines its own copy of these any more.
- **Acceptance:** the six files import/use the shared fixture (no local
  `def kb_env` in any of them); `pytest -q` count unchanged at 434; behavior of
  each file's existing tests byte-identical to before the move (they pass as
  before).
- **Scope (non-exhaustive):** `tests/conftest.py`, the six files above.
- **Status:** CLOSED (2026-09-10) — maintainer general ruling 260910: "everything
   pertaining testing is your job and as long as it does not change program behavior
   no maintainer involvement needed" — the consolidation LANDED + verified (status
   tail); the documented-preference call resolved in favor of the consolidation.
   One-line record: shared fixtures in `tests/conftest.py` + helpers in
   `tests/kb_helpers.py` (`1fd669c`).
- **Status tail:** LANDED (2026-09-10) — the three drifted shape-classes moved
  VERBATIM into `tests/conftest.py` as `kb_env_ns` (shape A, SimpleNamespace
  yield + arg flags pre-set) / `kb_env_mouse` (shape B, raw yield +
  `_mouse_listener` + arg flags) / `kb_env_plain` (shape C, raw yield, no arg
  flags); `build`/`down`/`up`/`hold_keys`/`mock_control_handlers` defined once
  in `tests/kb_helpers.py` (plain functions, imported by the test files via the
  prepend-mode `tests/` sys.path entry); the six files' local `def kb_env` +
  helper copies and the now-unused imports (`pytest`, `FST_Keyboard`,
  `MagicMock`, `SimpleNamespace` per file) deleted, signatures/bodies re-bound
  to the new fixture names. Verification: `pytest -q` = 434 passed, 1 warning
  (the known #10 coroutine warning, unchanged); `ruff check --select F .` = 0
  findings. Entry stays open for the maintainer's documented-preference call.

## 44. Stale/unknown focus name → uncaught KeyError in `apply_focus_groups` / `apply_start_args_by_focus_name` (the config is reloaded *before* the lookup) (2026-09-10, Audit 3b) (closed 2026-09-10, full text moved from TODO.md)

- **Problem / evidence:** `apply_focus_groups` (`fst_keyboard.py:386`) and
  `apply_start_args_by_focus_name` (`:1021`) index
  `self._focus_manager.multi_focus_dict[focus_name]` with NO membership guard, and
  `apply_start_args_by_focus_name` runs `self.update_focus_groups()` (`:1018` — full
  config reload via `load_config()` that replaces `_multi_focus_dict` wholesale,
  `fst_manager.py:1560-1563`) BEFORE the lookup — so a focus group removed or
  renamed in the config file between the last `Focus_Task` match and the lookup
  makes `multi_focus_dict[FOCUS_APP_NAME]` raise. `FOCUS_APP_NAME` is only ever
  cleared to `''` by `Focus_Task` (`fst_tasks.py:124`) or set at
  `Focus_Group_Manager` init (`fst_manager.py:1469`) — replacing the dict does
  NOT clear it. Uncaught
  propagation paths (verified — no try/except between the entry point and the
  lookup):
  (a) win32 hot path: `check_control_actions` (`fst_keyboard.py:925`) →
      `control_toggle_pause` (`:975-976`, passes `FOCUS_APP_NAME` twice — the same
      name goes stale twice) — the `_win32_event_filter` body (534-967) has no
      try/except (the filter's excepts are 644 rebind, 754 coroutine, 879 macro,
      901 macro logger, 1011 macro-name KeyError only); an exception in the
      callback would kill the pynput hook thread = silent listener death.
  (b) GUI: `fst_overlay.py:210-214` / `:590-591` (StatusOverlay/TrayIcon "Toggle
      Pause") call `control_toggle_pause` directly — unguarded.
  (c) CLI menu option "2. Reload everything from file" (`fst_manager.py:1875-1876`)
      — unguarded; a KeyError breaks the menu loop.
  Only the `Focus_Task` path catches it (`fst_tasks.py:106-109`
  `except Exception` → stays paused — and that catch is tested:
  `test_focus_task.py:216` pins `side_effect = RuntimeError('boom')`). No test
  anywhere covers `apply_focus_groups('…absent')` / `control_toggle_pause` with an
  out-of-dict `FOCUS_APP_NAME` (`test_cli_menu.py` and `test_facade_wiring.py`
  always pass names that exist in their dicts).
- **Outcome (goal):** a missing/renamed focus group degrades to the default
  groups (or a logged, user-visible error) instead of raising out of the hot
  path, a GUI slot, or the menu loop — chosen semantics per maintainer call.
- **Acceptance:** `control_toggle_pause`, the menu reload, and
  `update_args_and_groups(name)` no longer raise KeyError for names absent from
  the *reloaded* `multi_focus_dict`; a test for at least paths (a) and (c) pins
  the chosen behavior (freshly-deleted focus group → resume with defaults, or the
  decided alternative); suite green.
- **Scope (non-exhaustive):** `fst_keyboard.py` 383-404 / 916-931 / 972-986 /
  1016-1024; `fst_manager.py` 1875-1876 (CLI menu) + 1483-1497 (`Focus_Group_Manager`
  dict/keys handling — if the fix clears `FOCUS_APP_NAME` on dict replacement);
  `fst_overlay.py` 210-214/590-591 (guard site choice — a central fix in the
  two accessors makes these untouched); tests: `test_control_actions.py`,
  `test_focus_task.py`, `test_cli_menu.py`.
- **Status:** FIXED (P08 build, 2026-09-10) — design approved (now in
   `.opencode/proposals/approved/P08_configerror-design.md`, degrade-to-defaults
   included). `ConfigError` added to `fst_data_types.py`; raised at the stale focus
   name sites (`apply_focus_groups` / `apply_start_args_by_focus_name`) and at both
   `convert_to_vk_code` failure branches; caught at every user-facing boundary:
   win32 hot path (`check_for_combination` warn-once + `check_control_actions`
   degrade-to-defaults), CLI menu option 2 (print, loop continues), the 4 GUI overlay
   handlers (toast), and top-level startup (print + exit 1). Parse blocks preserve the
   type (except-before-generic). Acceptance met: paths (a)+(c) pinned
   (`tests/test_config_error.py`), suite green 448 passed / ruff `--select F` clean.
   Handoff to planner: condense/close this entry. **Overlap note for #1:** the
   constraint `p()`/`tr()` path now catches `ConfigError` fail-closed (print +
   `return False`) so it no longer crashes, but it still only PRINTS (console) — #1's
   "user-visible error at the constraint path" is NOT yet met, so #1 stays OPEN.

## 46. Flaky test: `test_crossover_not_taken_on_low_roll` — timing/order-dependent (2026-09-10) (closed 2026-09-10, full text moved from TODO.md)

- **Problem / evidence:** during the #3 docs rework the first full `pytest -q` run
  failed exactly ONE test, `tests/test_output_manager.py::TestCrossover::test_crossover_not_taken_on_low_roll`
  (433 passed, 1 failed); the immediate re-run of the FULL suite was green (434 passed)
  and the test passes in isolation. The test monkeypatches `randint` (probability roll
  `0` → no crossover; delay roll `5` ms), fires `send_keys_for_tap_group`, then asserts
  the exact release/press order after a fixed `await asyncio.sleep(0.02)` — i.e. it
  races the 5 ms `asyncio.sleep` inside the scheduled coroutine against a 20 ms wall-clock
  window, which can be lost on a loaded machine or when scheduling is delayed by
  preceding tests. Docs-only task, no code touched — pre-existing instability.
- **Outcome (goal):** the crossover tests are deterministic — no dependence on
  wall-clock timing or test order.
- **Acceptance:** the test passes under repeated full-suite runs (e.g. 10 consecutive
  `pytest -q`) without flaking; no behavior change in `send_keys_for_tap_group`.
- **Scope (non-exhaustive):** `tests/test_output_manager.py::TestCrossover` (replace the
  fixed `asyncio.sleep(0.02)` with an event-driven wait — e.g. poll the mock's
  `method_calls` until both calls are recorded, bounded by a timeout); `fst_manager.py`
  only if a deterministic seam is added for the scheduled task.
- **Status:** OPEN — pre-approved (test-only, no observable behavior change); found
   during #3 (2026-09-10). Delegated (looprun 2, iteration 1, approved-fix batch).
- **Status tail:** LANDED (2026-09-10, approved-fix batch) — the fixed
   `await asyncio.sleep(0.02)` replaced with the event-driven bounded wait
   `TestCrossover.wait_for_calls` (polls the mock's `method_calls` until exactly
   the expected calls are recorded; 100 × 10 ms bound; AssertionError on
   timeout), applied to BOTH async crossover tests (the named one + its sibling
   `test_crossover_presses_new_key_first`, same race; scope named the whole
   class). Production `send_keys_for_tap_group` untouched. Gate: 10 consecutive
   FULL `pytest -q` runs green (436 passed each), ruff F=0.

## 41. Production bug: `remove_all_toasts()` control function calls a nonexistent attribute (plural/singular mismatch) (2026-09-10) (closed 2026-09-10, full text moved from TODO.md)

- **Problem / evidence (planner-verified from the dead Q4 re-run's lead, session 3):**
  `fst_manager.py:578` (`remove_all_toasts`, the control-function family built in
  `constraint_evaluation`) calls `self._fst.remove_all_callbacks()` (PLURAL), but
  `FST_Keyboard` only has the SINGULAR `remove_all_callback` (`fst_keyboard.py:64`,
  assigned `bridge.trigger_remove_all` at `free_snap_tap.py:193`; the playground
  probe `overlay_probe.py:70` also uses the singular). No plural attribute exists
  anywhere on a production object → calling the `remove_all_toasts()` control
  function in production raises `AttributeError`. The test suite HIDES this:
  `tests/conftest.py:69` (FakeFST) sets `remove_all_callbacks = MagicMock()` (plural)
  and `tests/test_output_manager.py:534` asserts on the plural mock. Already flagged
  in `.opencode/archive/COVERAGE_TRIAGE.md` ≈275–284 ("Suggested fix (for the
  maintainer)") but never promoted to an open TODO.
- **Outcome (goal):** name parity — one attribute, called the same everywhere, tests
  matching the production name.
- **Acceptance:** `fst_manager.py:578` + the conftest FakeFST + the test assertion
  all use the SAME name as the production `FST_Keyboard` attribute; a test fails if
  the names ever drift (e.g. the FakeFST attribute is asserted against
  `FST_Keyboard.__init__`'s); suite green.
- **Scope (non-exhaustive):** `fst_manager.py` ≈578, `fst_keyboard.py` ≈64,
  `free_snap_tap.py` ≈193, `tests/conftest.py` ≈69, `tests/test_output_manager.py`
  ≈534.
- **Recommended fix (planner):** call the SINGULAR `remove_all_callback()` at
  `fst_manager.py:578` (the production name, used by `free_snap_tap.py` + the
  playground) and update the two test references — one-line fix + two test refs.
  Alternative (set a plural alias in `FST_Keyboard.__init__`) is worse: two names
  for one thing.
- **Status:** OPEN — maintainer APPROVED (260910 ruling: "sounds ok ... approved").
   Delegated (looprun 2, iteration 1, approved-fix batch).
- **Status tail:** LANDED (2026-09-10, approved-fix batch) — `remove_all_toasts`
   calls the SINGULAR `remove_all_callback()` (fst_manager.py:578); FakeFST
   attribute (tests/conftest.py) + test assertion (tests/test_output_manager.py)
   use the singular production name; new drift-guard
   `test_remove_all_toasts_drift_guard` drives `remove_all_toasts()` against a
   stand-in exposing ONLY the singular attribute — name drift raises
   AttributeError, which the eval path does not swallow (only NameError is
   caught, fst_manager.py:676). Gate: pytest -q = 436 passed, ruff F=0.

## 42. Multi-notch scroll-wheel events (delta ≠ ±120) are untested across all layers; the filter pins wheel phase on single-notch equality (2026-09-10, Audit 3a) (closed 2026-09-10, full text moved from TODO.md)

- **Problem / evidence:** `FST_Keyboard.mouse_win32_event_filter`'s inner `is_press()`
  (fst_keyboard.py ≈456-460) returns True/False ONLY when `data.mouseData` is
  EQUAL to the single-notch constants `4287102976` (down, delta −120) / `7864320`
  (up, delta +120) — for any other wheel delta (240, 360, … common fast/momentum
  scroll) it returns **implicit `None`** (same failure shape as #1's numeric-vk
  branch). `None` then propagates as the falsy `is_keydown` into
  `_win32_event_filter` (fst_keyboard.py ≈505-510); the vk code 6/7 itself is
  assigned unconditionally for scroll messages (≈475-478), so multi-notch wheel
  events flow through as release-phase-only events. The test suite pins exactly
  the two single-notch constants — `tests/test_filter_behavior.py:402-410`
  (`test_scroll_messages_map_to_vk_6_and_7`) uses `mouse_data=4287102976` /
  `7864320` verbatim (constants echoed from the production source); a repo-wide
  grep of `tests/` finds NO other wheel `mouseData` values (only 0 / 65536 /
  131072 x-button values). The output-side `scroll_up/down/right/left(n)`
  functions ARE tested with arbitrary magnitudes (`tests/test_output_manager.py`
  `test_scroll_constraints` ≈594-601) — that covers only `Output_Manager`
  constraint functions, never the `mouse_win32_event_filter` entry with a
  multi-notch payload. Consequence: a regression in wheel delta handling (or a
  deliberate semantics change) cannot be caught anywhere in the suite, and in
  production every non-±120 delta event behaves like a release phase (phase
  inversion vs. the ±120 path).
- **Outcome (goal):** the suite pins multi-notch wheel semantics explicitly:
  a wheel event with |delta| ≠ 120 (concrete example values computed for the
  test: 2-notch up `mouseData=15728640`, 2-notch down `4279238656`) produces the
  documented idealized output — either aggregated magnitude or the documented
  single-unit equivalent — decided by the maintainer (this changes observable
  behavior).
- **Acceptance:** new test(s) drive `mouse_win32_event_filter` with the two
  multi-notch constants and assert the exact `_win32_event_filter` /
  `mouse.scroll` outcome for the chosen semantics; the single-notch tests stay
  green; entry closes with a status note naming the decided semantics.
- **Scope (non-exhaustive):** `tests/test_filter_behavior.py` (`TestMouseWin32Filter`,
  ≈371-430); `fst_keyboard.py` ≈451-478 + ≈503-514 (read-only verification);
  `fst_manager.py` ≈703-709 (scroll sign mapping — context for acceptance only).
- **Status:** OPEN — maintainer APPROVED the mask/shift semantics (260910 ruling:
   the direction info sits on bits 16/17 of `mouseData`; "we have to apply a binary
   mask or shift it to recognise it when any other bit is 1" — approved). Multi-notch
   wheel event = SAME phase as single-notch (magnitude NOT aggregated). ADDITIONAL
   RULING: any other place comparing a single bit in a series of status bits
   (packed-word equality checks) = REPORT BACK, implicitly approved. Delegated
   (looprun 2, iteration 1, approved-fix batch).
- **Status tail:** LANDED (2026-09-10, approved-fix batch) — the wheel branch of
   `is_press()` replaced with the mask/shift sign test
   `bool((data.mouseData >> 16) & 0x8000)` (True = down/press, False = up/
   release), node-verified BEFORE the edit against BOTH single-notch constants
   (7864320 → False, 4287102976 → True) and BOTH 2-notch spec constants
   (15728640 → False, 4279238656 → True). Bit note: bits 16/17 are NOT set in any
   wheel constant — the delta word occupies bits 16-31, the distinguishing bit is
   bit 31 (sign of the delta word); the approved "mask or shift, direction
   regardless of other bits" semantics are exactly implemented (low word = key
   state, ignored). Multi-notch = SAME phase as single-notch, magnitude NOT
   aggregated. New test `test_multi_notch_scroll_keeps_single_notch_phase`
   (tests/test_filter_behavior.py); single-notch tests stay green. Gate: pytest
   -q = 436 passed, ruff F=0. REPORT BACK (ruling): packed-word equality sites
   found → new entry #48 (implicitly approved).

## 3. Rework README and WIKI to the current state of the code (2026-09-06) - CLOSED (work LANDED 2026-09-10; curated 2026-09-11, iter 5) - all 15 fix-list items reworded per the 2026-09-06 decisions, each verified against the code before rewording (the per-item evidence status tail lives in TODO.md's git history); the §3-undocumented-features residual went to #47 (closed 2026-09-10).

## 48. Packed-word equality checks in the mouse filter: X-button mouseData + LLKHF flags (2026-09-10, #42 report-back) (closed 2026-09-11, full text moved from TODO.md)

- **Problem / evidence:** the #42 audit (ruling: report back on ANY other
   equality comparison against a packed multi-bit status word / single-bit-in-a-
   series check) found two production sites of the same defect class in
   `FST_Keyboard.mouse_win32_event_filter`:
   (a) `fst_keyboard.py:471-473` — X-button vk mapping compares
   `data.mouseData == 65536` (x1) / `== 131072` (x2) on EXACT equality; for
   WM_XBUTTONDOWN/UP the high word is the XBUTTON identifier and the low word
   is the key state (ctrl/shift) — with a modifier held the low word is
   nonzero, the equality fails, `get_mouse_vk_code()` returns None and the
   event is suppressed via `self._mouse_listener.suppress_event()` (≈514)
   without any rebind/tap processing (silently dropped).
   (b) `fst_keyboard.py:49` — mouse `is_simulated_key_event` is
   `flags == 1` on the packed LLKHF flags word; an injected event carrying any
   other LLKHF bit (e.g. LLKHF_LOWER_IL_INJECTED 0x20) is misclassified as real
   input. Correct bit-test pattern already in the same file: keyboard
   `flags & 0x10` (fst_keyboard.py:520). Secondary (playground probe, not
   production): `playground/pynput_mouse_probe.py:101, 109-125` carries the
   same patterns. The post-#42 wheel sign test is the reference pattern.
- **Outcome (goal):** the X-button vk mapping and the mouse simulated-check use
   bit tests / masks instead of packed-word equality — status bits in the other
   half of the word must not change the outcome (per the #42 ruling).
- **Acceptance:** x1/x2 down/up with a nonzero low word (shift/ctrl state)
   still map to vk 4/5; a flags value `1 | 0x20` is still classified simulated;
   tests pin both; suite green.
- **Scope (non-exhaustive):** `fst_keyboard.py` ≈49, ≈471-474; tests in
  `tests/test_filter_behavior.py::TestMouseWin32Filter`.
- **Status:** CLOSED (2026-09-11, worker) — the mouse `is_simulated_key_event`
  now tests bit 0 (`bool(flags & 1)`) and the X-button vk mapping tests the
  high word (`(data.mouseData >> 16) == 1` → vk 4, `== 2` → vk 5, mirroring the
  #42 wheel idiom); status bits in the other half of the word do not change
  the outcome. 3 new tests in `tests/test_filter_behavior.py::TestMouseWin32Filter`:
  `test_x_buttons_ignores_key_state_low_word` (x1 down/up + x2 down with
  nonzero low words → vk 4/5), `test_x_button_other_identifier_suppresses`
  (x3 identifier `196608` → suppress path, regression guard),
  `test_simulated_flag_bit0_only` (flags 1 / 0x21 simulated; 0 / 0x20 real).
  The existing `test_x_buttons_use_mousedata_for_vk` +
  `test_simulated_flag_passthrough` unchanged and green. Gate: `pytest -q` =
  **451 passed + 1 known #10 warning** (baseline 448 before the new tests);
  `ruff check --select F .` = 0. Landing commit: the first commit after
   `00bc24f`, subject "Mouse filter: packed-word equality → bit tests (TODO
   #48)" (a commit cannot cite its own hash — self-referential SHA is
   infeasible; the repo convention is date + gate + subject).

## 2026-09-12 (iter-9, branch `fst_work`) — FST behavior batch closed (ruling 2026-09-12; unit A `4b93d37` + unit B `2891dab`; gate 459 passed + 1 known #10 warning, ruff F=0) — full text of closed entries #1, #7, #8, #9, #4, #6 (moved from TODO.md):

## 1. General vk resolution: unknown keys must surface to the user (tabled 2026-09-06)

- **Problem / evidence:** wherever a key string resolves to a vk_code (`convert_to_vk_code`
  + all its call sites) it should raise AND be communicated to the user — important
  feedback; today many paths fail silently or only print to console. State-shorthand
   constraints now fail-closed on unknown keys (2026-09-06, `fst_manager.py`
   `constraint_evaluation`) but still only print — fold into the general solution.
   Verified crash path (2026-09-10, session-3 probe): `convert_to_vk_code('300')` /
   `('256')` return an implicit `None` (the numeric branch swallows the KeyError when
   `key_int` is out of range — `fst_keyboard.py:146-150`), and the following
   `if vk_code <= 0:` in `extract_data_from_key` (`fst_keyboard.py:224`) then raises
   `TypeError: '<=' not supported between instances of 'NoneType' and 'int'` — an
   out-of-range numeric key string in the config crashes group init with an
   unhelpful TypeError instead of a surfaced error. `test_extraction_filter_edges.py`
   ≈61-64 pins the implicit-None behavior (archived triage "suspected bug #2").
    **Audit 3b addition (2026-09-10):** `check_for_combination` (`fst_keyboard.py:906-912`,
    called from the hot path at `:627` via `check_control_actions`) converts its string
    combo entries with `convert_to_vk_code` (`:910`); a non-resolving string returns
    implicit None and then SILENTLY poisons state instead of erroring —
    `get_real_key_press_state(None)` catches its own KeyError
    (`fst_manager.py:1604-1609`) and INSERTS a `None` key into BOTH
    `_real_key_press_states_dict` AND `_all_key_press_states_dict`
    (via `set_real_key_press_state`, 1612-1613 — this setter writes `_all` unguarded).
    The combos resolve today (alt/end/delete/page_down all in `vk_codes_dict`,
    verified 2026-09-10) — the defect is latent for any custom/unresolvable combo string.
 - **Outcome (goal):** ONE general solution covering every vk-resolution site (not
  per-call-site fixes) with the user-visible error; the constraint path reuses it.
- **Acceptance:** unknown key ⇒ user-visible error at every resolution site (never
  console-only); the constraint fail-closed path emits the same user-visible error; suite
  green.
- **Scope (non-exhaustive):** `convert_to_vk_code` + all its call sites (`fst_manager.py`,
  `fst_keyboard.py`, …); `constraint_evaluation`'s unknown-key branch.
- **Status:** RULING 2026-09-12 — Rec approved (build the P08-style user-visible error at
   every vk-resolution site; the constraint path reuses it). LANDED on branch `fst_work`
   (iter-9, unit A, commit 4b93d37): one `FST_Keyboard.surface_config_error` helper; both
   constraint fail-closed guards + `check_for_combination` (+ the hot-path resume catch)
   route through it (GUI P08 error toast / headless print, dedup + fail-closed preserved) —
   the console-only residual is closed by this build. Unknown constraint *names* stay silent
   no-ops by design (`SPEC_FEATURES.md` §4 #2) — out of scope.

## 7. Empty macro: comment/behavior mismatch at `fst_keyboard.py` 707 (2026-09-08)

- **Problem / evidence:** the comment says an empty key group does "not supress the
  triggerkey", but `alias_fired = True` (line 697) is set BEFORE the empty check — so the
  trigger IS suppressed (`_listener.suppress_event`, verified by
  `test_empty_macro_sequence_no_playback`). Comment and behavior cannot both be right.
- **Outcome (goal):** comment and behavior agree — decide which is right, reword or fix
  accordingly.
- **Acceptance:** the 707 comment matches the (possibly new) behavior; a test pins the
  chosen semantics; suite green.
- **Scope:** `fst_keyboard.py` empty-macro check (≈697/707); the pinning test in `tests/`.
- **Status:** RULING 2026-09-12 — Rec approved (behavior wins: keep the suppression,
  reword the comment, pin with a test). LANDED on branch `fst_work` (iter-9, unit B,
  commit 2891dab): the 737 comment reworded to match the kept suppression ("supress"
  spelling fixed); `test_empty_macro_sequence_no_playback` already pinned both
  no-playback AND trigger suppression — comment-only change, no new test needed.

## 8. `ap`/`ar` "all keys (incl simulated)" is not a union — last-write-wins shared dict (2026-09-08)

- **Problem / evidence:** `ap(...)` documents "press of all keys (incl simulated)"
  (`fst_manager.py` 253–256), but `set_real_key_press_state` (1611–1614) and
  `set_simulated_key_press_state` (1622–1625) both write the shared
  `_all_key_press_states_dict` last-write-wins — a release from either side clears `all`
  even while the other press is still active, so `ap` may not behave as "real OR
  simulated". Asymmetric: the real-setter lacks the `vk_code > 0` guard the other two
  setters have. Found in the interrupted Phase-5 run (it explains the 1630–1632 KeyError
  coverage gap).
- **Outcome (goal):** confirm the intended `ap`/`ar` semantics (or fix the dict handling +
  the missing guard) and pin the chosen behavior with a test.
- **Acceptance:** documented semantics; crossing press/release on either side behaves per
  the confirmed semantics (the other side's `all` state preserved under a union); the
  guard symmetric if the union is confirmed; suite green.
- **Scope:** `Input_State_Manager` state setters (≈1611–1632); `ap(...)` docs (253–256).
- **Status:** RULING 2026-09-12 — Rec approved (union semantics real OR simulated;
  symmetric `vk_code > 0` guard; pinning test). LANDED on branch `fst_work` (iter-9,
  unit B, commit 2891dab): both setters now write the `all` dict as a union
  (`is_press or <other side's state>`, missing side defaults False); the real setter
  gained the symmetric `vk_code > 0` guard; `ap`/`ar` doc comments state the union
  (the "relese" typo fixed as an adjacent comment fix); 3 new pinning tests
  (crossing release both directions + vk guard) in `test_input_state_manager.py`.

## 9. Repeat-constraint excepts too narrow for malformed `repeat_thread_dict` entries (2026-09-08)

- **Problem / evidence:** `toggle_repeat` (327), `is_repeat_active` (340), `reset_repeat`
  (350) catch `(KeyError, AttributeError)`; `stop_all_repeat` (365) only `AttributeError` —
  but unpacking an entry that is not a 2-tuple raises `ValueError`, uncaught, propagating
  out of `constraint_evaluation`. Low severity: entries are only written as 2-tuples
  `[task, handle]` by `start_repeat` (301).
- **Outcome (goal):** decide harden the excepts or leave as-is — record the decision.
- **Acceptance:** decision recorded here; if harden: `ValueError` covered in the four
  methods + a test; suite green.
- **Scope:** `fst_manager.py` ≈301–365 (`Input_State_Manager` repeat methods).
- **Status:** RULING 2026-09-12 — Rec approved (harden: add `ValueError` to the four
  repeat methods + a test). LANDED on branch `fst_work` (iter-9, unit B, commit
  2891dab): `ValueError` added to the excepts of `toggle_repeat` / `is_repeat_active`
  / `reset_repeat` / `stop_all_repeat` (+ `stop_repeat` as the fifth site — consistency
  addition beyond the ruling's letter); one new pinning test covers all five methods
  with malformed (1- and 3-element) entries.

## 4. Dead code: `fst_manager.py` 116–117 ("None result → pass") unreachable (2026-09-07/08)

- **Problem / evidence:** `check_constraint_fulfillment`'s "None → pass" branch (line 117)
  can never run — `constraint_evaluation` normalizes `None` → `True` (≈691) before
  returning, and no other branch returns `None`; line 117 stays uncovered in every run and
  the triage's 2098/2217 (94.6 %) ceiling is really 2097/2217 (same rounded number).
  Recorded at discovery (`2007698` commit msg). Absorbs the 260908-0951 dedup-block item 1.
- **Outcome (goal):** the dead branch is deleted — the A→C triage reclassification is the
  maintainer's call (`COVERAGE_TRIAGE.md` is agent-read-only).
- **Acceptance:** 116–117 removed (or the triage plan reclassified by the maintainer);
  suite green; coverage expectations updated.
- **Scope:** `fst_manager.py` ≈107–125; the triage plan (maintainer-side only).
- **Status:** RULING 2026-09-12 — Rec approved (delete the dead branch). LANDED on
  branch `fst_work` (iter-9, unit B, commit 2891dab): the unreachable
  `elif result is None: pass` branch is deleted (suite green, the `else:` print for
  other non-bool/int results stays); the A→C triage reclassification stays
  maintainer-side (`COVERAGE_TRIAGE.md` is agent-read-only — untouched).

## 6. Dead code: `fst_keyboard.py` 302–303 (mixed-Key rebind conversion) unreachable (2026-09-08)

- **Problem / evidence:** in `initialize_groups_from_presorted_lines`, `convert_key_string_group`
  only ever appends `Key_Event`s (bare keys expand to press/release events), so
  `new_trigger_group[0]` is never a `Key`; the 295-block is entered only via a `Key`
  replacement — which makes line 301 `False`, so 302–303 (`replacement_key = Key(...)`)
  can never execute. Proven at `fffea8b` (rebinds `w : e` and `w : +e` both leave 302–303
  uncovered). Triage classed them A — same misclassification as #4. Also: the triage's
  numeric example `"8" → 8` is wrong — `"8"` resolves via the dict to 56; the numeric
  branch needs a string absent from `vk_codes_dict` (e.g. `"255"`). Absorbs the 260908-0951
  dedup-block item 4 (first half).
- **Outcome (goal):** the dead block is deleted (or kept on an explicit maintainer
  decision — the triage class correction rides with #4).
- **Acceptance:** 302–303 removed; suite green; coverage expectations updated.
- **Scope:** `fst_keyboard.py` ≈295–305.
- **Status:** RULING 2026-09-12 — KEEP 302-303 (maintainer: "keep this until I can test a
  bit more") → CLOSED by ruling (no deletion; the triage class correction stays
  maintainer-side per #4).

## 11. General contradiction prevention disabled (XXX 241016-1101) — CLOSED (maintainer ruling D1-A, 2026-09-15) — kept OFF as an intentional decision; decision comment below the untouched pin; pinning tests unchanged.

## 113. The repo venv's python.exe was broken — the base interpreter it points at is gone (2026-09-28, plan29 planner gate run — ENVIRONMENT break, maintainer call) — CLOSED (2026-09-30, plan40 planner-40) — venv REBUILT on 3.12.9 by the maintainer (2026-09-29) + the agent-repo probe's VENV_PY re-pointed at the FST venv (the agent repo has no .venv of its own post-split; the agent-repo venv is still preferred if present) — probe 352/352 (the 11 env-fails 136-146 resolved) + FST gate 459 passed 1 warning + ruff F=0 re-established.

- **Problem / evidence:** the plan29 gate run (2026-09-28) measured:
  `./.venv/Scripts/python.exe -c "print(...)"` fails with
  `No Python at '"C:\Users\Wasiejen\Projects\OpenCodeProjects\Free-Snap-Tap\python312\python.exe"'`
  (the error text carries an embedded quote). `.venv/pyvenv.cfg` points
  `home`/`executable` at `C:\Users\Wasiejen\Projects\OpenCodeProjects\Free-Snap-Tap\python312`
  (ONE level above the repo root); that `python312` dir exists but holds
  only `Doc` — `python.exe` is ABSENT. Consequence measured in the same
  run: the agent-repo probe's 11 numword python checks (136-146, `runPyW2n`
  over VENV_PY) all FAIL with that message; the FST standard gate's pytest
  half is unrunnable (the venv is the only python with the repo deps — bare
  PATH `python` = 3.14 without deps per repo_commands.md). The 2026-09-27
  baselines (probe 345, pytest 459+1w) predate the break.
- **Resolution trail:** (1) the maintainer REPLACED the venv with a fresh
  3.14.3 build (2026-09-29), then REBUILT it on 3.12.9 (verified 02:00 —
  `pyvenv.cfg` → the proper `AppData\Local\Programs\Python\Python312`
  install, 459 tests collect). (2) plan39 (worker-39) measured the full FST
  standard gate in the FST workspace: ruff F=0, pytest 459 passed 1
  warning. (3) RESIDUAL (this close): the agent-repo probe's `VENV_PY`
  pointed at the agent repo's own `REPO_ROOT/.venv`, which no longer
  exists post-split — the probe's S17 python twin (numword `w2n` check,
  pure stdlib) now resolves agent-repo-venv-first with the FST workspace
  venv as fallback (`C:\Users\Wasiejen\Repos\Free-Snap-Tap\.venv\Scripts\
  python.exe`, 3.12.9 verified). Probe re-run: **352/352 PASS** (was 341
  + the 11 env-fails 136-146).
