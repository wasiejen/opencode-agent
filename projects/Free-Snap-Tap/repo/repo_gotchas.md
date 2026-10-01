# repo_gotchas.md — gotchas part of `repo_overview.md` (split 2026-09-11)

Sections moved verbatim from the former root file; the overview (`repo_overview.md`, this folder) is now a
thin index pointing at the parts.

## Gotchas
- `Config_Manager.load_config` opens `self._file_name` directly — point it at a
  `tmp_path` fixture or monkeypatch `_open_config_file`.
- `presort_lines` receives the **cleaned** lines (no spaces after commas inside
  key groups) — feed cleaned-form strings in tests.
- `Output_Manager.execute_key_event` is `async` and calls `asyncio.sleep` +
  pynput — mock both when testing.
- `CONSTANTS` is a real module-level class used as a global config holder; tests
  that mutate it should restore it.
- The repo's `FSTconfig_test.txt` is the maintainer's live config —
  `tests/test_config_parse.py::test_real_config_parses` uses it as a regression
  guard.
- PySide6 `destroyed` gotcha: a handler connected to `obj.destroyed` ALSO fires
  when OTHER objects are destroyed, and the signal argument is an untrusted
  placeholder (a bare QWidget) — capture the object in the closure and probe
  liveness (any C++ method call raises RuntimeError once the C++ side is
  deleted).
- pytest-qt quirks: `QTest.mouseMove(widget, pos)` takes a LOCAL position
  (global = widget.pos() + pos); double-click is `qtbot.mouseDClick` (NOT
  `mouseDoubleClick`); `QSystemTrayIcon` is a QObject, not a QWidget →
  `qtbot.addWidget` rejects it.
- Offscreen destruction ordering: `destroyed` fires while the dying widget's item
  is still in its parent layout — never assert layout-count-based visibility
  right after deletion; use the dict (synchronous source of truth).
- NEVER call a widget's `contextMenuEvent` in tests (its `exec_` blocks the
  loop); trigger the `QAction`s of `widget.context_menu` via `.trigger()`
  instead.
- Edit tool `replaceAll` semantics (probe-verified 2026-09-10, P05): SINGLE
  PASS over the ORIGINAL string — the replaced output is never re-scanned
  (rename `kb_env`→`kb_env_ns` does NOT cascade into `kb_env_ns_ns…`). It
  still matches SUBSTRINGS inside longer tokens (`kb_envx`→`kb_env_nsx`),
  and if the TARGET name already exists in the file, its source-name prefix
  is corrupted in the same pass (`kb_env_ns`→`kb_env_ns_ns`) — that is the
  only case needing the two-stage token dance or a scripted whole-file
  substitution; otherwise a direct rename is safe in one edit (verify with a
  grep afterward).
- Host-dependent capability checks (built-in modules, runtime flags, bundled
  binaries) must probe the REAL host (P06): inside the plugin/process or with
  the bundled runtime — never a same-named system CLI (the system-bun vs.
  opencode.exe-bun false confidence, #37).
- Qt/PySide6 offscreen + first-show facts (verified 2026-10-01, worker-4
  #130, measured offscreen): (1) `QWidget.windowHandle()` is None until the
  widget has been shown at least once (it does NOT create the window); a
  `show()` BEFORE the event loop creates the native handle synchronously and
  the handle persists after `hide()` — re-showing a hidden top-level window is
  cheap (the #130 pre-warm relies on this). (2) Offscreen: a shown zero-area
  (e.g. 500x0 childless) top-level window never receives a Paint event, and
  no Expose event is delivered offscreen at all — paint-based triggers need a
  non-zero area; QWindow has no exposed signal bound in PySide6. (3) An
  explicit `resize()` on a top-level widget WITH a layout PERMANENTLY disables
  auto-resize to the layout sizeHint (top-level auto-resize tracks sizeHint
  only until the first explicit resize) — children then get clipped. (4) The
  first build of a widget with a custom font/stylesheet pays the
  process-wide first font/stylesheet cost (18.6 ms cold vs 1.1 ms warm
  offscreen) — pre-payable by building one throwaway widget at startup.
