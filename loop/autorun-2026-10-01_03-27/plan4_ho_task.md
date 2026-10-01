# Task spec — #130 FST first-toast lag (diagnose + fix, FST worker unit)

Worker: worker_Q3S
Repo: FST `C:/Users/Wasiejen/Repos/Free-Snap-Tap` — stay on the current checkout
`fst_work3` (verified c61bce5, clean tree at spec time). Code commits land in the
FST repo; the TODO status + handover file land in the AGENT repo
(`C:/Users/Wasiejen/Repos/opencode-agent`) — two repos, two commit streams.
FST docs: `projects/Free-Snap-Tap/repo/` in the agent repo (read repo_commands +
repo_gotchas).

## Goal
Root-cause the ~2 s lag on the FIRST toast of a session (retriggers are instant),
then land the fix so the first toast no longer pays the first-call cost.

## Verified facts (planner, 2026-10-01, at spec time — not for re-derivation)
- Threading path: the keyboard logic runs in a SECOND thread (`free_snap_tap.py:181`).
  Toast trigger = `fst_keyboard.toast_callback = bridge.trigger_toast`
  (`free_snap_tap.py:195`). `ToastBridge` (fst_overlay.py:855) is created on the
  MAIN thread (`free_snap_tap.py:187`) before `gui_manager.start()` (:208). The
  cross-thread signal emit is a QUEUED connection — `add_toast` runs on the GUI
  thread (receiver `gui_manager.toast_manager.add_toast`, connected at
  `free_snap_tap.py:190`).
- `ToastManager` is built in `StatusOverlay.__init__` (fst_overlay.py:111).
  `add_toast` (779-794): builds `ToastWidget`, adds it to the layout,
  `update_position()`, then `self.show()` (794). After the last toast dies,
  `check_empty()` HIDES the window (849-853) — a re-trigger after a gap re-shows
  the SAME native window (cheap). The FIRST show pays native window creation
  (frameless + translucent + StaysOnTop tool window, flags 764-765) + first
  paint / font cost.
- Strong lead (UNVERIFIED — your job to confirm or falsify): the ~2 s first-call
  cost is the first `show()` of the ToastManager window, not the signal chain
  (retriggers instant → the event loop is not persistently busy).
- OFFSCREEN LIMIT: tests run `QT_QPA_PLATFORM=offscreen` (tests/conftest.py) —
  offscreen timing CANNOT measure the real Windows native-window cost. Diagnosis
  = segment-timing instrumentation (relative ordering) + code analysis of the
  first-call inits. Live "first toast instant" confirmation = maintainer domain.
- Existing test shape: `tests/test_gui_smoke.py` — `make_toast_env()` (L40) builds
  `ToastManager` + `ToastBridge` + the signal connection and calls
  `trigger_toast` (round-trip tests). Build on that shape, do not re-derive it.
- Baseline gate (2026-10-01, post-#129): pytest 465 passed + 1 warning, ruff F=0.
- Commands (run from the FST root): `./.venv/Scripts/python.exe -m pytest -q`;
  `./.venv/Scripts/ruff.exe check --select F .`

## Scope
1. Instrument: measure the first-toast path in segments (emit → queued delivery →
   `ToastWidget` build → `addWidget` → `show`/first paint), offscreen. Keep the
   instrumentation clean (timed prints you remove, or a small in-test measurement —
   your call); do not leave noisy permanent console output.
2. Identify the dominant first-call cost; record the evidence (measured numbers +
   analysis) in the handover.
3. Land the fix so the first `show()` no longer pays the first-call cost —
   pre-warm shape (e.g. show/hide the toast window once at startup, or pre-build a
   `ToastWidget`; your call on the exact mechanism; smallest change wins).
4. A pin test (offscreen-runnable) covering the fix (e.g. after GUI init the toast
   window's native side already exists, or the first `add_toast` no longer creates
   it — your call on the exact pin).

## Definition of done
- Root cause identified WITH evidence in the handover: which segment dominates,
  measured or analyzed.
- Fix landed: the first toast no longer creates/pays its first-call cost
  offscreen (the mechanism is pinned by the test); retrigger behavior unchanged.
- FST gate: pytest 466 passed + 1 warning (465 + the new pin), ruff F=0.
- Checkpoint commits per verified unit in the FST repo; `projects/Free-Snap-Tap/
  TODO.md` (#130 status → LANDED, "hash recorded in the planner's follow-up
  bookkeeping commit") + `agent/handover/handover_task_to_planner.md` in the
  agent repo's FINAL commit (carrying the FST code commit hashes, never its own).
- Live confirmation (first toast instant on the real display) = maintainer domain —
  say so in the TODO status.
- INCONCLUSIVE ESCAPE: if the offscreen instrumentation shows no dominant
  first-call segment (i.e. the 2 s must be live-only), STOP after the
  instrumentation + analysis, report the evidence in the handover, and do NOT land
  a speculative fix — the planner decides the next step.

## DO-NOT-TOUCH
- The tap-group section of `fst_keyboard.py` (2026-09-15 maintainer ruling).
- Maintainer live files: `opencode.jsonc`, `maintainer/**` (never edit/stage).
- No branch switches in FST; no pushes (maintainer pushes).
- NO live FST app run on the real display — offscreen only (the live test is the
  maintainer's domain).

## Notes
- No new TODO entry is expected, but the diagnosis may surface a doc/code
  discrepancy or a second root-cause candidate — if so, note it in the handover
  (the planner curates).
