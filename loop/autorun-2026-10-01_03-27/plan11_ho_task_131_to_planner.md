# Worker summary — #131 FST structured-logging build (worker-11, ses_ef6b793f2ffek8H7AISRzFV4gX, 2026-10-05)

Status: DONE — all 3 units built, gate green after each, FST repo branch `fst_work3`.
Commits (FST repo, in order): `ea2aac3` (U1 logger setup), `cac6704` (U2 per-flag
dispositions), `cd0f3a2` (U3 error-documentation fixes). Baseline at start: b513bf2,
466 passed + 1 warning / ruff F=0 (re-measured, matched the spec).

## Measured gate (final, U3)
- `./.venv/Scripts/python.exe -m pytest -q` → **474 passed, 1 warning** (the known
  #10 coroutine warning at `fst_keyboard.py` mouse-rebind dispatch, unchanged)
- `./.venv/Scripts/ruff.exe check --select F .` → **F=0** (All checks passed)
- Per-unit gates: U1 471 passed, U2 471 passed, U3 474 passed (each + ruff F=0)
- `test_real_config_parses` (the live-config regression guard) stayed green untouched
  through all three units; `FSTconfig_test.txt` / `FSTconfig.txt` / `playground/` /
  requirements / packaging untouched. No live listener runs, no real keyboard.

## What changed per unit
**U1 — logger setup (doc §3/§3.4):**
- Area-logger tree `fst / fst.flow / fst.eval / fst.state / fst.macro / fst.config`
  (loggers defined in `fst_manager.py`, imported where used). `fst.log` now
  `filemode='a'` (per-run wipe fixed), console handler WARNING, formats per doc §3.3/§4.
  `setup_fst_logging()` called from `free_snap_tap.py` main (idempotent).
- `fst.flow` variant chosen: own `fst_flow.log` (`filemode='a'`) + **propagate=False
  while enabled** (the doc's clean variant); disabled = WARNING + propagate on.
- Start args: `-log=LEVEL` (debug/info/warning/error; invalid → WARNING), `-flow`
  (idempotent re-apply on focus changes), `-debug` removed (now an unknown-arg
  WARNING), `-debug_numpad` unchanged. Start-arg echo + `-file=` echo → `fst.config`.
- Numpad remap: `alt+num1/2/3` → shared root-debug toggle (DEBUG↔INFO over
  `fst` + the four non-flow areas), `alt+num4` → flow toggle, `alt+num5` unchanged
  (console repr dump), `alt+num7/8` → `fst.state` DEBUG records. Menu option 0 →
  flow toggle. CLI keep-open gate (`clear_cli`) = any area logger at DEBUG → no `cls`.
- free_snap_tap: source-edit flag block deleted, startup group dump → INFO,
  existing `logging.info` calls re-homed to the `fst` logger.

**U2 — per-flag dispositions (doc §2 tables):**
- `CONSTANTS.DEBUG/DEBUG2/DEBUG3/DEBUG4` deleted; `DEBUG_NUMPAD` kept (safety gate).
  The `Argument_Manager` snapshot channel fully dies (class attrs + snapshot lines
  + commented reset lines); single channel = the loggers.
- ~80 flag-guarded print sites converted: per-key/pipeline → `fst.flow` DEBUG
  (incl. the D4 successor with the arrow vocabulary: `--> IN/OUT (t_rel_ms)
  real|sim <ke>`, `-- rebind/toggle/toggle-suppress`, `-- XX SUPPRESSED`, `-- |`
  sim marker kept); state/timing → `fst.state` DEBUG (incl. first-call KeyErrors);
  eval lines → `fst.eval` DEBUG; macro/repeat → `fst.macro` DEBUG; config-load
  constraint parse + group counts → `fst.config` DEBUG; backup/restore → INFO.
- 6 error-class sites → **always-on WARNING**: rebind-not-found
  (`fst_keyboard.py`, dict dump demoted to DEBUG), no-macro-sequence reset (merged
  with the always-on print), repeat-alias not found ×3, constraint NameError ignored.
- Kills: commented mouse-event debug, commented old `release_all_*` block
  (the `###XXX 241014-1926 changed` marker kept), commented `inc()` block, the stale
  lying D2 "suppressed" print (suppression is commented out — pin comment kept),
  `makro` typo fixed in the log message.

**U3 — error-documentation fixes (doc §5):**
- `check_result` restored: `fut.result()` in try/except — crash →
  `fst.macro` ERROR; `asyncio.CancelledError` → DEBUG (intentional cancels stay
  silent; it's a BaseException, so it must be caught separately or it leaks to GC
  "Exception ignored").
- Invalid constraint result → `logger_eval.warning`; unexpected eval exception →
  `logger_eval.error` (the "stays console-only" comment corrected); config-load
  `ERROR:` prints (all 4 extraction pairs) → `fst.config` ERROR.
- Nested `clear_all_variables` **KEPT, not deleted** — see "deliberately not done".

## DoD greps (flag surface, measured at close)
- `grep -rn "CONSTANTS.DEBUG" --include="*.py"` (code): only
  `fst_keyboard.py:676` (the kept `DEBUG_NUMPAD` gate read) and
  `fst_manager.py:1408` (`-debug_numpad` sets it) + the test pin
  `tests/test_argument_manager.py:242`. Zero other hits.
- `grep -rn "self.DEBUG\|_arg_manager.DEBUG\|Argument_Manager.DEBUG"`: **zero hits**.

## Tests added/repurposed (8 new, 6 repurposed)
New: `-log=` level set (default-not-debug, debug/info/warning/error, invalid →
WARNING, `-debug` now unknown, `-flow` opens the flow file exactly once) in
`test_argument_manager.py`; `alt+num1` toggle-on/off in `test_control_actions.py`;
`check_result` crash-logged / cancel-silent in `test_macro_playback_kbd.py`;
config-load error → `fst.config` ERROR in `test_config_error.py`.
Repurposed: menu option 0 → flow toggle (+ pins the new keep-open gate: second
loop pass does NOT clear the console); numpad actions → logger-level toggles;
D2 contradiction diagnostics → flow DEBUG records (caplog); the mouse-rebind
dispatch-error pin re-pointed to logger `fst.flow`; the invalid-constraint pin →
`fst.eval` WARNING record. `tests/conftest.py`: `restore_fst_loggers` autouse
fixture (levels/propagate/handlers per test); `restore_constants` attrs list
shrunk to the surviving attributes.

## Deliberately not done
1. **Nested `clear_all_variables` kept** — the research doc (§5.5) calls it dead/
   self-recursive, but it is the eval-scope shim for bare `clear_all_variables()`
   config calls: used live in `FSTconfig_test.txt:850,1596` and pinned by
   `test_clear_all_variables_constraint_empties_store`. Deleting it would silently
   no-op the maintainer's live config. Kept with an explanatory comment; the dead
   commented lines inside it were removed. **Filed in `todo_inbox.md`** for planner
   curation (doc discrepancy; the doc's §5.5 line should be corrected).
2. **Out-of-range start-arg notices kept as `print`** (invalid delay, delay/probability
   out of range, invalid crosshair, no valid tap/alias/macro delay). Doc §3.1's
   WARNING row says "unknown start argument / out-of-range values (`:1461` etc.)"
   but the §2 per-site list only itemizes the unknown-arg site, and the DoD pins
   only "unknown arg → WARNING". Those prints are console UX and not flag-guarded;
   converting them (and their ~8 capsys tests) is a small follow-up if the maintainer
   wants it.
3. **`-logdir=` start arg** — explicitly deferred by the doc §3.3 ("cheap future
   extension, not needed for the first build").
4. No live-console/file confirmation (maintainer's domain per spec); the flow
   toggle's runtime behavior is pinned by tests, not by a live run.

## Friction / lessons
Friction entries filed via submit (see `agent/agent_feedback.md`): the edit-tool
"hint rejected" advisory that actually corrupted a line mid-build (`fst_manager.py:201`,
caught by the gate) — worth a look; and the spec's affected-tests list missing
`test_extraction_filter_edges.py:176` + `test_output_manager.py:607` (both
logger-name/print pins this change alters).

Lessons: when converting `print`→`logger` in a suite with pytest's root
LogCaptureHandler, capsys tests break (records go to the root handler, not
stdout/stderr) — plan caplog/caplog.at_level pins for every converted site, and
grep the suite for `logger='` pins before starting (caught one via failure,
the rest via the gate).
