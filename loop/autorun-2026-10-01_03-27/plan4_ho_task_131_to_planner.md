# Explorer handover — #131 FST DEBUG→logging research (read-only)

Explorer: explorer-4 (Qwen3.8-27B-Q3S-170K), looprun 2026-10-01_03-27.

## Result
Research doc: `agent/research/2026-10-01_fst-logging-design.md` — complete per the
spec's DoD: concrete logging design (levels, per-area loggers, 2 files,
`-log=LEVEL`/`-flow` start-arg control), per-flag disposition for all 5 flags,
DEBUG4 successor design, opinionated recommendation (**structured logging wins;
retire the flags in one FST worker unit**).

Files touched: `agent/research/2026-10-01_fst-logging-design.md` (new) + this
handover. **No FST repo edits/commits** (read-only over `fst_work3` @ b513bf2,
clean at start, still clean at end). No maintainer live files touched.

## Key findings beyond the spec's verified facts (doc §7, 8 items)
1. **`fst.log` uses `filemode='w'`** — every launch wipes the log; error evidence
   from a crashed run is destroyed. Biggest argument for the redesign.
2. **Two parallel flag channels** (`CONSTANTS.*` vs `Argument_Manager` snapshot at
   `fst_manager.py:1287-1290`): runtime numpad toggles write only `CONSTANTS`, so
   `self.DEBUG` reads (`fst_manager.py:1360,1374`) and `self._arg_manager.DEBUG2`
   (`fst_keyboard.py:953`) don't follow the toggle.
3. **`check_result` (`fst_keyboard.py:908-914`) is `pass`** (the
   `logger.error(fut.result())` is commented out) — repeat-task crashes are
   swallowed, never documented.
4. Stale print `fst_keyboard.py:869-870` says "suppressed" where the suppression
   is commented out at `:873` — a lying DEBUG line.
5. Dead nested `clear_all_variables` (`fst_manager.py:454-459`, self-recursive;
   shadowed by the real method at `:781`) — safe to delete.
6. Spec bookkeeping: "4 logging calls outside tests" is actually ~14 live calls
   (embryonic still); flag-toggle lines are `free_snap_tap.py:30-41` (spec said
   32-41); `fst_manager.py` is 1943 lines (spec said ~1900).
7. `-debug`/`-debug_numpad` are the only debug start args — **no
   `-debug2/-debug3/-debug4` args exist** (those flags: source edit, numpad, or
   menu option 0 only).
8. The CLI-keep-open behavior at `fst_manager.py:1856` is **functional** (any debug
   flag → don't `cls`), not just a print — the design preserves it via
   "any area logger at DEBUG".

## Design in one breath
- Loggers: `fst` (root, main) + `fst.flow` (own file `fst_flow.log`, the DEBUG4
  successor), `fst.eval`, `fst.state`, `fst.macro`, `fst.config`.
- `fst.log` appended (never wiped), INFO default, WARNING+ always on;
  `fst_flow.log` DEBUG only when enabled; console WARNING (headless) + existing
  error toasts (GUI) unchanged.
- Control: `-log=LEVEL`, `-flow` (extends the existing `apply_start_arguments`
  pattern; re-applied on focus change like the other args), `-debug_numpad` kept
  as the safety gate; `alt+num1`/`num4` + menu option 0 become runtime logger
  level toggles (workflow parity with today).
- Dispositions: DEBUG/DEBUG2/DEBUG3 → kill flag, convert sites (error-class sites
  promoted to always-on WARNING — esp. the silently-ignored constraint NameError
  at `fst_manager.py:684`); DEBUG4 → keep as `fst.flow` with the same
  `-->/--/XX/-->` arrow vocabulary + IN/OUT relative-ms latency pairs;
  DEBUG_NUMPAD → keep (safety gate), targets remapped.
- Hot-path cost unchanged (one `isEnabledFor` check ≈ one boolean read).

## Recommendation
Opinionated, doc §8: **structured logging beats the DEBUG constants** for the
maintainer's use case (same mental model, one channel, errors that survive
across runs, per-key latency structure, no-overwhelm by construction). Build
would be one FST worker unit (task list deliberately NOT detailed here, per spec).

## TODO
- No new TODO entries expected/added; the 8 discrepancies above are noted for
  planner curation (none warrant a standalone TODO entry on their own — items 1–4
  are all inside the future #131 build scope; the rest are bookkeeping).

## Gauge
Final readout at end of session (see DONE loop-log line): gauge below.
