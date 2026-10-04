# Task spec — FST structured-logging build (TODO #131 build; approved
# proposal `proposals/approved/2026-10-01_fst-logging-build.md`)

Date: 2026-10-05 (planner-11, autorun ses_ef7155f2dffedZ7mKtZW0EZqTh,
looprun autorun-2026-10-01_03-27). Worker: worker_Q3S_slow.

## Goal
One FST worker unit on `fst_work3`: replace the five `CONSTANTS.DEBUG*`
console-debug flags with stdlib structured logging (area loggers,
append-only files, `-log=LEVEL` / `-flow` start args, numpad/menu remap)
and absorb the four audit bugs the research found. The design is SETTLED —
`agent/research/2026-10-01_fst-logging-design.md` (agent repo) IS the
design source: its §2 tables are the per-site disposition list, §3 the
scheme, §4 the DEBUG4 successor, §5 the error fixes. Build on that doc;
do not re-derive the design.

## Verified facts (planner-measured 2026-10-05, checkout fst_work3 @ b513bf2, clean tree)
- FST gate baseline (planner re-run today): pytest **466 passed + 1 warning**
  (the known #10 coroutine warning), ruff `--select F` **F=0**
  (`./.venv/Scripts/python.exe -m pytest -q`; `./.venv/Scripts/ruff.exe check
  --select F .`). venv = 3.12.9; never the bare `python` (3.14, no deps).
- The research doc was written against THIS checkout (b513bf2) — its line
  numbers are current: flag defs `fst_manager.py:29-33` (all False); the
  SECOND flag channel = `Argument_Manager` snapshot `fst_manager.py:1287-1290`;
  `logging.basicConfig(filename='fst.log', filemode='w', …)`
  `free_snap_tap.py:21-26` (the per-run wipe, `filemode` at :23);
  `check_result` `fst_keyboard.py:908-914` (`pass`, the `logger.error`
  commented at :912); ~96 flag-guarded print sites per the doc's §2 tables.
- Existing live `logger.*`/`logging.*` calls (~16) that stay and get
  re-homed as needed: `free_snap_tap.py:151,164,178,212,216`;
  `fst_keyboard.py:95,102,107,817,921,931,946,967` (+ commented :912);
  `fst_manager.py:354,776`; `fst_tasks.py:29`.
- Behaviour-pinning tests the change alters (repurposing is IN scope —
  rewrite them to pin the new logger-level behaviour, or delete
  deliberately with a one-line note): `tests/test_argument_manager.py:187`
  (`am.DEBUG is True`), `:202` (`CONSTANTS.DEBUG_NUMPAD is True`);
  `tests/test_cli_menu.py:48-53` (menu option 0 → `CONSTANTS.DEBUG4`);
  `tests/test_control_actions.py:63-66` (control combo → `CONSTANTS.DEBUG`);
  `tests/test_filter_simulated.py:115` (`CONSTANTS.DEBUG2 = True`);
  `tests/conftest.py:83` (FakeFST attrs list includes the five flags).
- The FST repo has NO `TODO.md` (verified) — bookkeeping rides the agent
  repo: `projects/Free-Snap-Tap/TODO.md:22` (#131, planner's lane).
- `FSTconfig_test.txt` is the maintainer's live config;
  `tests/test_config_parse.py::test_real_config_parses` is its regression
  guard — it must stay green untouched.

## Scope (3 ordered units; checkpoint commit per verified unit; gate green after each)
1. **Logger setup** (doc §3 + §3.4): `fst.log` append-only (`filemode='a'`);
   the area-logger tree `fst / fst.flow / fst.eval / fst.state /
   fst.macro / fst.config`; `fst.flow` own `fst_flow.log` + the
   propagate handling per doc §3.2 (worker's call between the two
   documented variants); console at WARNING; `-log=LEVEL`
   (debug/info/warning/error, default info) + `-flow` start args in
   `apply_start_arguments` (replacing `-debug`; `-debug_numpad` unchanged;
   unknown arg → WARNING); the numpad remap (`alt+num1` root debug level,
   `num2/3` collapse onto it, `alt+num4` flow toggle) + menu option 0 →
   flow toggle + the CLI keep-open gate (`fst_manager.py:1856`: any area
   logger at DEBUG → don't `cls`).
2. **Per-flag dispositions** (doc §2 tables — convert every listed site):
   kill `DEBUG/DEBUG2/DEBUG3`; error-class sites → always-on WARNING
   (`fst_keyboard.py:706-708, 1090-1092`; `fst_manager.py:310-311, 348-349,
   363-364, 684-685`); D4 → `fst.flow` records keeping the arrow
   vocabulary + the IN/OUT latency pairs (doc §4); `DEBUG_NUMPAD` kept as
   the safety gate; the two parallel flag channels collapse to one (the
   `Argument_Manager` flag snapshot dies — reads go to loggers / the
   single gate).
3. **Error-documentation fixes** (doc §5): restore `check_result` logging
   (`fst_keyboard.py:908-914`); `fst_manager.py:116,658` →
   `logger.warning`/`logger.error`; the stale "suppressed" print
   (`fst_keyboard.py:869-873`) fixed or deleted; config-load `ERROR:`
   prints → `logger.error`; delete the dead nested
   `clear_all_variables` (`fst_manager.py:454-459`).

## DO-NOT-touch
- `FSTconfig_test.txt` (live config), `FSTconfig.txt`, `playground/`
  (maintainer's live probes), `requirements*.txt`, the packaging scripts.
- Agent repo: anything under `maintainer/` EXCEPT the one requested new
  file (DoD below); `TODO.md` + `projects/Free-Snap-Tap/TODO.md` + the
  handover files (planner's lane — you write ONLY the handover summary +
  the guideline file).
- No live listener runs, no real keyboard, no FST live testing (maintainer
  domain; the live console/file confirmation of the build is his too).

## Definition of done
- FST gate green: pytest all passing (the 466 baseline + the
  new/repurposed tests) + ruff F=0 — measured counts in the handover.
- The flag surface is gone per doc §6: `grep -rn "CONSTANTS.DEBUG" ` +
  the `self.DEBUG`/`self._arg_manager.DEBUG*` snapshot forms outside the
  kept `DEBUG_NUMPAD` gate = zero hits (report the greps in the handover).
- The start args are pinned by tests (`-log=LEVEL`, `-flow`, unknown arg
  → WARNING) and the repurposed pin tests pin the new logger behaviour.
- Handover at `agent/handover/handover_task_to_planner.md` (agent repo):
  executive summary, measured gate, the FST-repo commits, the per-flag
  disposition list (what died / what converted / what was promoted),
  what was deliberately not done.
- Usage guideline (the maintainer's approved ask — "put it in
  maintainer/feedback"): ONE new file
  `maintainer/feedback/2026-10-05_fst-logging-usage.md` in the AGENT repo —
  short and practical (which `-log` level for which purpose, when `-flow`,
  how to read `fst_flow.log` for lag, the runtime toggles), written from
  the implementation.
- Checkpoint commits per verified unit in the FST repo (code + tests;
  commit routine applies to each); no TODO edits by you (no FST TODO.md —
  the #131 status update is the planner's); the agent-repo files you write
  stay uncommitted (the planner commits them).
- No new TODO entry expected — if the build surfaces a doc/code
  discrepancy, file it in the agent repo's `todo_inbox.md` instead.

## Branch
Stay on `fst_work3` (verified checked out at b513bf2, clean tree, at spec
time) — re-verify `git branch -v` before your first edit.
