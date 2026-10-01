# Task spec — #129 FST indicator context-menu crash (small FST worker unit)

Worker: worker_Q3S
Repo: FST `C:/Users/Wasiejen/Repos/Free-Snap-Tap` — stay on the current checkout
`fst_work3` (verified e77c01f, clean tree at spec time). Code commits land in the
FST repo; the TODO status + handover file land in the AGENT repo
(`C:/Users/Wasiejen/Repos/opencode-agent`) — two repos, two commit streams.
FST docs: `projects/Free-Snap-Tap/repo/` in the agent repo (repo_commands,
repo_gotchas, repo_map, repo_testgate — read repo_commands + repo_gotchas).

## Goal
Right-clicking the status indicator opens its context menu without raising
`AttributeError: 'QContextMenuEvent' object has no attribute 'globalPosition'`.

## Verified facts (planner, 2026-10-01, at spec time — not for re-derivation)
- The bug: `fst_overlay.py:595` — `self.context_menu.exec_(event.globalPosition().toPoint())`
  inside `StatusOverlay.contextMenuEvent` (L594-595). Installed PySide6 = 6.11.2
  (FST venv, 3.12.9): `QContextMenuEvent` inherits from `QEvent` — it has NO
  `globalPosition()`. Live evidence: the maintainer's bug report (FST_bugs.md,
  inbox 2026-10-01, curated as FST TODO #129).
- `event.globalPosition()` at `fst_overlay.py:541` and `:555` is CORRECT — those
  are mouse events (`QMouseEvent` has `globalPosition` in Qt 6). DO NOT TOUCH
  those lines.
- The menu: `self.context_menu = QMenu(self)` built at `fst_overlay.py:473-483`;
  `QMenu.exec_(point)` expects a GLOBAL QPoint.
- Expected fix shape (worker may pick the exact equivalent):
  `self.context_menu.exec_(self.mapToGlobal(event.pos()))` — `event.pos()` on a
  `QContextMenuEvent` is the widget-local QPoint.
- Test harness: offscreen pytest-qt (`tests/conftest.py` sets
  `QT_QPA_PLATFORM=offscreen`). `tests/test_status_overlay.py` already builds a
  `StatusOverlay` via a `SimpleNamespace` `fst` stand-in fixture + an `overlay`
  fixture — build on that shape, do not re-derive it.
- Existing behaviour pin to flag: `tests/test_status_overlay.py` docstring
  (L4-5) says the blocking `contextMenuEvent/exec_` path is NOT exercised; no
  test currently pins `contextMenuEvent`. The new pin must therefore NOT block:
  spy on the menu's `exec_` (instance-level monkeypatch of
  `overlay.context_menu.exec_` capturing the QPoint argument), dispatch a real
  `QContextMenuEvent(QPoint(local_x, local_y))` through the real
  `overlay.contextMenuEvent(event)`, assert (a) no AttributeError, (b)
  `exec_` received `overlay.mapToGlobal(QPoint(local_x, local_y))`. Update the
  docstring's "NOT exercised" line accordingly (the path is now exercised with
  a captured exec_).
- Baseline gate (2026-10-01, post-#122): pytest 464 passed + 1 warning, ruff F=0.
- Commands (repo_commands.md): tests `./.venv/Scripts/python.exe -m pytest -q`;
  lint `./.venv/Scripts/ruff.exe check --select F .` — run from the FST root.

## Scope
1. `fst_overlay.py:595` — replace the broken accessor with the correct global
   point (one line).
2. `tests/test_status_overlay.py` — add the one pin test above + update the
   docstring line.

## Definition of done
- FST gate: pytest 465 passed + 1 warning (464 + the new pin), ruff F=0.
- The new test exercises the REAL `contextMenuEvent` (a `QContextMenuEvent`
  object dispatched through the method) and asserts the global point passed to
  the menu's `exec_` equals `mapToGlobal(local)`; the captured `exec_`
  guarantees no modal block in the test.
- Checkpoint commits per verified unit in the FST repo; `projects/Free-Snap-Tap/
  TODO.md` (#129 status → LANDED, "hash recorded in the planner's follow-up
  bookkeeping commit") + `agent/handover/handover_task_to_planner.md` in the
  agent repo, in the agent-repo final commit (carrying the FST code commit
  hashes, never its own).

## DO-NOT-TOUCH
- `fst_overlay.py` L541 / L555 (the correct mouse-event usages).
- The tap-group section of `fst_keyboard.py` (2026-09-15 maintainer ruling).
- Maintainer live files: `opencode.jsonc`, `maintainer/**` (never edit/stage).
- No branch switches in FST; no pushes (maintainer pushes).

## Notes
- No new TODO entry is expected, but a fix may surface a doc/code discrepancy —
  if so, note it in the handover (the planner curates).
- #130 (first-toast lag) is a SEPARATE entry: do not chase or fix it; if you
  incidentally observe something root-cause-relevant, one line in the handover.
