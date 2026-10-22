# Spec — TODO #145: per-event filter helper for `.opencode/temp/intercept.log`

Planner-26 (ITER-26, 2026-10-22). Entry: `TODO.md` #145 (OPEN, pre-approved class — agent-usage friction removal).

## Goal
Add `agent/scripts/log/intercept_sweep.cjs`: a read-only PER-EVENT filter view of the intercept log — last-N events by default, filters by session id / verdict kind / needle, ONE BOUNDED line per event. The existing aggregate summary stays untouched.

## Definition of done
- The script runs on the live log, exit 0, for all three demo forms (record the exact commands + output in the handover):
  - (a) default = last 20 events;
  - (b) `--session=<ses_…>` with a real id taken from the log;
  - (c) `--verdict=<kind>` — e.g. `fuzzy-rejected`, `out-of-sandbox`, or `error`.
- Every output line ≤ ~120 chars; the long args-JSON field is truncated with a visible `…` marker.
- Read-only: writes stdout only.
- New smoke `agent/scripts/log/tests/intercept_sweep.smoke.cjs` green with plain `node` (exit 0), fixture-based. The existing `summarize_intercept.smoke.cjs` stays green and UNCHANGED.
- `agent/scripts/log/README.md` gains one table row + at most 2 usage lines.
- `TODO.md` #145 status line → `LANDED` (the hash rides the planner's follow-up commit, per the spec-discipline rule).
- `agent/handover/handover_task_to_planner.md` written per AGENTS.md §Worker summary.

## Verified facts (measured at spec authoring, 2026-10-22)
- Line shape: `date_time | session_id | model | tool | args-json | evidence | scope | verdict`, ` | `-separated, parsed RIGHT-ANCHORED (verdict = last field, scope = second-to-last; the args region is the long field between `tool` and `evidence`).
- `agent/scripts/log/summarize_intercept.cjs` (210 lines) already parses this shape and EXPORTS the helpers: `parseLine` (line 54) + `FROZEN_VERDICTS` (line 210 `module.exports = { summarize, FROZEN_VERDICTS, parseLine, pathPrefix, pairLeft }`). Reuse them via `require("./summarize_intercept.cjs")`; do NOT edit that script.
- Live log at authoring: 5674 lines / 85 sessions / 0 malformed (the summary header). The verdict vocabulary includes NEW kinds beyond the frozen list — measured: `edit-ambiguous`, `edit-hint`, `error`, `fuzzy-edit`. The filter must accept ANY trailing verdict token, never only the frozen list.
- No pin surfaces: the probe and the plugin smokes pin no `agent/scripts/log/` file. Expected gate after the change: `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360 PASS` unchanged; run it ONCE.

## Approval boundary
Pre-approved class. DO NOT touch: `maintainer/**`, `opencode.jsonc`, `AGENTS.md`, `agent/prompts/**`, anything under `.opencode/**`, `summarize_intercept.cjs` + its fixture + its smoke (all unchanged), any TODO entry other than #145's status line.

## Suggested scope (non-exhaustive)
- `agent/scripts/log/intercept_sweep.cjs` (new file — decision taken: NEW file, keeps the aggregate summary stable)
- `agent/scripts/log/tests/intercept_sweep.smoke.cjs` (new) + a fixture if the existing `tests/intercept_fixture.log` is not enough (its 3 first lines are R7-era; the new verdict kinds may need extra fixture lines — the worker's call)
- `agent/scripts/log/README.md`
- `TODO.md` #145 status line
- `agent/readme/repo_commands.md` — optional one-line pointer ONLY if a natural home exists (do not restructure that file)

Suggested flag shape (worker's call inside the DoD): `intercept_sweep.cjs [--session=ID] [--verdict=KIND] [--grep=NEEDLE] [--lines=N] [logfile]`, default last 20.

## Commit routine
One green checkpoint per verified unit; `TODO.md` + the handover file ride the FINAL commit (carrying the code commits' hashes, never their own).

## Assigned worker
worker_FlashNext
