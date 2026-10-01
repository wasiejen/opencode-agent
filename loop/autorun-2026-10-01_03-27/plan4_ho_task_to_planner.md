# Worker handover — #130 FST first-toast lag (diagnose + fix)

Worker: worker-4 (Qwen3.8-27B-Q3S-170K), looprun 2026-10-01_03-27.

## FST code commit (carried here, never self-referenced)
- `b513bf2` on `fst_work3` — pre-warm fix + pin test (2 files, +21 lines, additions only).
- FST gate at that commit: **pytest 466 passed + 1 warning** (baseline 465 + the new pin), **ruff F=0**.

## Root cause (evidence in handover per DoD)
Diagnosis = offscreen segment-timing instrumentation (scratch script, scratchpad — nothing
left in the repo) + code analysis of the first-call inits.

Measured segments (offscreen, ms; fresh-process cold values first):
- queued emit → `add_toast` delivery (cross-thread): ~0.7 (5.71 incl. a deliberate 5 ms emit delay)
- `ToastWidget` build: **18.61 cold / 1.06 warm** (the cold one includes the process-wide
  first font/stylesheet/font-directory cost)
- `addWidget`: 0.33–0.76 · `update_position`: 0.06–1.62
- first `show()` of the toast window [COLD]: 0.44 (windowHandle null before, set after)
- first paint delivered: 5.92 · second `show()` [WARM, after hide]: 0.16
- full first `add_toast` cold: 0.55 · full retrigger warm: 1.17 (same order — no delta)

Conclusion: every in-process segment is millisecond-scale offscreen — the ~2 s is the
Windows-native portion, which offscreen cannot measure (spec-acknowledged limit). By
elimination, though, the first-toast path's first-call-only costs are exactly two, both
confirmed:
1. **The toast window's native side is created only on its first `show()`** —
   `add_toast` (fst_overlay.py:794) is the only show call site; `check_empty` hides the
   window when empty, so retriggers re-show the same hidden native window (handle
   persistence probed offscreen: set after first show, still set after hide). This is the
   spec's strong lead — CONFIRMED as the only first-call window segment.
2. **The first `ToastWidget` build pays the process-wide first font/stylesheet cost**
   (measured 18.61 ms cold vs 1.06 ms warm offscreen; the live Consolas bold load can only
   be larger). A second first-call candidate — also pre-warmed.

The INCONCLUSIVE escape was NOT taken: instrumentation + analysis identified the dominant
first-call segment (first `show()` = native window creation), so a targeted pre-warm (not a
speculative fix) was landed per the spec's scope item 3.

## The fix (FST `b513bf2`)
`GUI_Manager.__init__`, right after `ToastManager` construction:
- `self.toast_manager.show()` + `self.toast_manager.hide()` before the event loop:
  `show()` creates the native window handle synchronously (probed), `hide()` keeps it —
  the first trigger then only re-shows the existing window, structurally identical to the
  verified-instant retrigger path. The window is invisible (frameless + translucent +
  empty 500×0 + WindowTransparentForInput + Tool).
- `ToastWidget("prewarm", 0.1, 12)` — throwaway widget paying the first font/stylesheet
  cost; its own master timer destroys it after the first tick (no parent, no layout,
  never shown).

Pin test: `tests/test_gui_manager.py::test_toast_window_prewarmed_at_startup` — after
`GUI_Manager` construction the toast manager's `windowHandle()` is not None and the
manager is hidden (matches the steady/retrigger state). Without the fix the handle is
None (never shown before the first toast) → the pin would fail.

## Deliberately NOT done
- **Live confirmation** (first toast instant on the real display) = maintainer domain —
  no live FST run was made (spec DO-NOT-TOUCH).
- **The first expose/paint (DWM surface) of the toast window itself still happens on the
  first toast** — it cannot be pre-paid without a visible toast at startup: a zero-area
  window never receives Paint events (probed), and an explicit pre-resize is not an
  alternative — it permanently disables top-level auto-resize to the layout sizeHint
  (probed) and would clip multi-toast stacking. If the maintainer's live test still shows
  lag, the remaining suspects are that expose/surface cost or the first cross-thread
  queued delivery — planner decides the next step.
- No changes to `fst_keyboard.py` tap-group, maintainer files, branch (`fst_work3` kept),
  no pushes.

## TODO
- `projects/Free-Snap-Tap/TODO.md` #130 status → LANDED (this commit), with the FST commit
  hash and the live-confirmation-is-maintainer-domain note. No new TODO entries.

## Knowledge
- One verified Qt/PySide fact set submitted to the knowledge inbox (windowHandle()
  semantics, offscreen Expose/Paint behavior, auto-resize disable on explicit resize).

## Lessons
- None beyond the knowledge submission.
