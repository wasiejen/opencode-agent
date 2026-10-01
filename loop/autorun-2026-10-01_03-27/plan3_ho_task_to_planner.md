# Worker handover — TODO #129: FST indicator context-menu crash

Session: ses_f0a8d978effem4PJrbIaas9Zfb (worker_Q3S, Qwen3.8-27B-Q3S-170K).
FST repo: `C:/Users/Wasiejen/Repos/Free-Snap-Tap`, stayed on `fst_work3`
(verified clean at spec time, e77c01f). Agent repo: `opencode_test`
(unchanged). Spec: `agent/handover/handover_task.md`. One small verified unit
(the one-line fix + its pin) — landed in a single FST code commit.

## What changed (FST repo)

### `fst_overlay.py` (`StatusOverlay.contextMenuEvent`, ~L594-597)
- Replaced the broken accessor `self.context_menu.exec_(event.globalPosition().toPoint())`
  with `self.context_menu.exec_(self.mapToGlobal(event.pos()))`.
  `QContextMenuEvent` has no `globalPosition()` (unlike `QMouseEvent`); its
  `pos()` is the widget-local QPoint, which `mapToGlobal` converts to the
  global QPoint `QMenu.exec_(point)` expects. One line + a 2-line comment.
- L541 / L555 (`QMouseEvent.globalPosition()`) UNTOUCHED (correct mouse-event
  usages, per the spec's DO-NOT-TOUCH).

### `tests/test_status_overlay.py`
- New pin `test_context_menu_event_executes_at_global_point`: spies on
  `overlay.context_menu.exec_` (instance-level monkeypatch capturing the
  QPoint — guarantees no modal block), dispatches a REAL
  `QContextMenuEvent(QContextMenuEvent.Reason.Mouse, local,
  overlay.mapToGlobal(local))` through the real `overlay.contextMenuEvent(event)`,
  asserts (a) no AttributeError and (b) `exec_` received
  `overlay.mapToGlobal(QPoint(3, 4))`.
- Imports: added `QContextMenuEvent` to the `PySide6.QtGui` import.
- Docstring L4-5: the "blocking contextMenuEvent/exec_ path is NOT exercised"
  line updated — the path is now exercised with a captured `exec_`.

## Measured verification (FST gate, run from the FST root)
- `./.venv/Scripts/python.exe -m pytest -q` → **465 passed, 1 warning**
  (baseline 464 + 1 new pin; the 1 warning is the pre-existing
  `fst_keyboard.py:817` coroutine-never-awaited RuntimeWarning, unchanged).
- `./.venv/Scripts/ruff.exe check --select F .` → **All checks passed** (F=0).

## FST code commit(s)
- `c61bce5` — `#129: fix indicator context-menu crash — mapToGlobal(event.pos())`
  (fst_overlay.py + tests/test_status_overlay.py). This is the ONLY FST commit
  for the task. (Its hash is recorded here / in the planner's follow-up
  bookkeeping commit — a commit never carries its own hash.)

## Agent-repo changes (this commit)
- `projects/Free-Snap-Tap/TODO.md` — #129 status open → LANDED (with the fix
  summary + gate numbers; hash noted as recorded in the planner's follow-up).
- This handover file (overwritten from the prior #128 handover).
- `loop/autorun-2026-10-01_03-27/loop_log.md` — START line (this session).

## TODO entries
- #129 → LANDED (no new TODO entry needed; nothing out of scope surfaced).

## What was deliberately NOT done
- `fst_overlay.py` L541 / L555 (correct mouse-event `globalPosition()` calls) — untouched.
- `fst_keyboard.py` tap-group section — untouched (2026-09-15 maintainer ruling).
- Maintainer live files (`opencode.jsonc`, `maintainer/**`) — untouched/never staged.
- #130 (first-toast lag) — NOT chased (separate entry, per the spec). No
  root-cause-relevant observation surfaced during this fix (nothing to report).
- No branch switches in FST; no pushes.

## Discrepancies found
- None. (One implementation note for the record, not a bug: the
  2-arg `QContextMenuEvent(reason, pos)` constructor is deprecated in
  PySide6 6.11.2 and emits a DeprecationWarning — the pin uses the
  non-deprecated 3-arg form `(reason, pos, globalPos)` to keep the gate at
  exactly 1 warning.)

Lessons: (none beyond the deprecation note above).
