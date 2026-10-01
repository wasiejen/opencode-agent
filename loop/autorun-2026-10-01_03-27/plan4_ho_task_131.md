# Task spec — #131 FST DEBUG→logging research (read-only, explorer unit)

Explorer: explorer_Q3S (read-only — NO code edits, no FST repo commits; the
research doc lands in the AGENT repo only).
FST repo: `C:/Users/Wasiejen/Repos/Free-Snap-Tap` — stay on the current checkout
`fst_work3` (verified b513bf2, clean tree at spec time); read-only access.
Agent repo: `C:/Users/Wasiejen/Repos/opencode-agent` — your research doc + handover
land here (one commit).

## Goal
A research doc: how to instrument the FST repo so logging helps find lag causes,
documents errors, and associates them to code areas — without overwhelming;
control via start-argument log level and/or separate log files per area; what the
DEBUG4 flow print should become (kept as a formatted log); which DEBUG outputs
die. Plus a recommendation on whether structured logging beats the DEBUG
constants for the maintainer's use case.

## Verified facts (planner, 2026-10-01, at spec time — not for re-derivation)
- `CONSTANTS` is a module-level config-holder class in `fst_manager.py:26-44`:
  `DEBUG = DEBUG2 = DEBUG3 = DEBUG4 = DEBUG_NUMPAD = False`.
- The maintainer toggles them by hand at `free_snap_tap.py:32-41` (each flag has
  a commented `= True` variant right below) — that is his live-debugging
  workflow today.
- Runtime numpad combos toggle `DEBUG2`/`DEBUG3`/`DEBUG4` at
  `fst_keyboard.py:1016-1020`.
- DEBUG-flag usages outside tests (per flag): DEBUG 24, DEBUG2 20, DEBUG3 22,
  DEBUG4 24, DEBUG_NUMPAD 6 — concentrated in `fst_manager.py` (34 total),
  `fst_keyboard.py` (18), `free_snap_tap.py` (6).
- `DEBUG4` is the notable keep: the formatted key-flow print (pressed key →
  replaced key → macro keyactions, indented hierarchy, `<--`/`-->` control-flow
  markers) — the maintainer wants it KEPT as a formatted log, not killed.
- Logging today is embryonic: `logging.basicConfig(` at `free_snap_tap.py:21`
  (the only config site) + 4 `logging.{info,warning,error,debug}` calls outside
  tests; `fst_manager.py:24` already has a module logger.
- Baseline: FST gate green (pytest 466 + 1 warning, ruff F=0) — read-only task,
  no gate run needed by you.

## Scope (research only — no FST code changes)
1. Map the DEBUG-flag usage sites (grep `CONSTANTS.DEBUG` in the FST repo,
   excluding tests/.venv) and classify each: what it prints, when, and what it
   would become (keep-as-log / convert / kill / keep-as-debug-level).
2. Design the logging scheme: levels, per-area loggers, file vs console
   separation, start-argument control (his existing start-args pattern in
   `free_snap_tap.py` — read that file's arg handling), where errors get
   documented, and how lag diagnosis benefits (timed segments / first-call
   paths).
3. Design the DEBUG4 successor: the formatted key-flow print as a structured
   log record (shape, level, destination).
4. Recommendation: logging vs the DEBUG constants for his workflow (he asked
   whether/how logging would be more helpful than the DEBUG constants).

## Definition of done
- Research doc at `agent/research/2026-10-01_fst-logging-design.md` in the
  agent repo:
  - a concrete logging design (levels, loggers per area, files,
    start-argument control);
  - a per-DEBUG-flag disposition list (every one of the 5 flags: keep /
    convert / kill, with the reasoning);
  - the DEBUG4 successor design;
  - the logging-vs-DEBUG-constants recommendation (one short section,
    opinionated).
- NO FST repo edits/commits (read-only); NO maintainer live files
  (`opencode.jsonc`, `maintainer/**` — never read for editing intent; never
  edit/stage).
- Your handover at `agent/handover/handover_task_to_planner.md` (the
  agent repo) + ONE agent-repo commit carrying the doc + handover.
- First action after reading this spec: a loop_log START line (role=explorer-4,
  your model id, session id); on completion a DONE<--- line with your final
  gauge readout.

## Notes
- No new TODO entry is expected, but the research may surface doc/code
  discrepancies in the FST bookkeeping — note them in the handover (the
  planner curates).
- The build (if the maintainer approves the design later) would be a separate
  FST worker unit — do NOT design its task list in detail here.
