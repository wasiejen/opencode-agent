# TASK — FST: let repeated keys through for REBINDS (approved proposal, FST repo)

Worker: `worker_Q3S_slow`. Planner: plan40 (autorun-2026-09-21_15-33, iteration 40).

## Worktree (IMPORTANT — this is a SEPARATE git repo)
- Work in `C:\Users\Wasiejen\Repos\Free-Snap-Tap` (its own repo; the agent
  repo `C:\Users\Wasiejen\Repos\opencode-agent` is only for your handover +
  TODO note + reading the proposal/specs below).
- Current checkout: branch `opencode_test` @ 57e7fdb, clean tree (verified
  at spec time). Create branch **`fst_work3`** from it: `git checkout -b
  fst_work3`. The branches `fst_work`, `fst_work2`, `opencode` are STALE
  pre-split branches — do NOT build on or touch them.
- On completion: `git checkout opencode_test` in the FST repo, leave the
  tree clean on it.

## Goal
A held REBIND key auto-repeats its target key; macro/toggle repeat-
suppression stays unchanged. Design of record (read FIRST — full design +
edge-case list): `proposals/approved/2026-09-12_fst-rebind-repeat.md`
(agent repo). The preserved earlier spec (superseded on branch/layout
facts only — read for the failed-first-attempt context): `archive/loop/
autorun-2026-09-15_13-11/plan2_ho_task.md` (agent repo).

## MAINTAINER RULING on the tap-group edge case (2026-09-15, `tap_groups_
behaviour.md` — SUPERSEDES the proposal's "tap-group behavior unchanged"
recommendation)
The repeat-let-through path applies ONLY to rebind triggers that are NOT
in any tap group. A rebind trigger that is ALSO in a tap group keeps the
current (suppressed) repeat behaviour — it must NOT auto-repeat its
replacement. This is the exact area the failed first attempt broke — pin it.

## Failed first attempt (READ BEFORE CODING)
A previous worker implemented a first version, emergency-committed as
`44dbc36` on the old `fst_work` — that commit is UNREACHABLE in both
repos now (filter-repo split); the diff is described here, not inspectable:
1. repeat-detection: skipped `to_be_suppressed` when the repeated trigger
   matches a rebind trigger (`self._rebind_triggers`, via `tg.get_trigger()`).
2. tap-group section: when `key_replaced is True` (a rebind replacement key
   consumed by a tap group), it set `to_be_suppressed = True` on the CURRENT
   event — the current event there IS the replacement key, so suppressing
   it broke rebinds inside tap groups (the observed tap-group test
   failures).
Your implementation must not reproduce that regression: the let-through is
press-only in the repeat-detection region, and the tap-group section is
untouched (the ruling above makes the second change unnecessary).

## Verified facts (planner-verified 2026-09-30 at `opencode_test` 57e7fdb —
do NOT re-derive)
- `fst_keyboard.py` markers (identify regions by these, not raw numbers):
  `_all_trigger_events` repeat-detection loop @631; `if not
  real_input_repeated:` @648; `'STOP REPEATED KEYS FROM HERE'` @699; `if not
  real_input_repeated and not to_be_suppressed:` @703; `# to allow repeated
  keys from hold` @773 (the tap-group repeat allowance); `###XXX
  241022-1341` @317 (RULING-KEEP); `###XXX 241016-1101` @822 (decision
  comment below it — do not edit either).
- Baseline (measured 2026-09-30, plan39, this workspace): pytest **459
  passed + 1 warning** (the known #10 coroutine warning), ruff **F=0**.
  RE-MEASURE it on `fst_work3` BEFORE touching code (`./.venv/Scripts/
  python.exe -m pytest -q` + `./.venv/Scripts/ruff.exe check --select F .`);
  if the baseline is RED: STOP and report — do not fix out-of-scope red.
- The venv works (3.12.9, `./.venv/Scripts/python.exe` — #113 closed).
- NO existing test references `real_input_repeated` / `to_be_suppressed`
  (zero grep hits in `tests/`) — your pinning tests are greenfield.

## Definition of done
1. Baseline (re-measured) + NEW pinning tests all green on `fst_work3`;
   ruff F count UNCHANGED vs baseline (F=0).
2. 3–5 new pinning tests in a keyboard-level test file in `tests/`
   (your choice — follow the existing mocked pattern, `conftest.py` /
   `kb_helpers.py` fixtures, no live listeners) covering the proposal's
   edge cases: rebind → `SUPPRESS_CODE` stays suppressed; key that is BOTH
   rebind and macro trigger (first press = rebind, repeats = target-key
   only, macro never fires); press-state bookkeeping consistent across
   repeats; and — the MAINTAINER RULING case, which MUST pin: a rebind
   trigger in a tap group does NOT repeat.
3. Checkpoint commits per verified unit in the FST repo (code + tests only).
   The bookkeeping rides the AGENT repo (never the FST repo): a NEW entry
   in `projects/Free-Snap-Tap/TODO.md` (agent repo — status LANDED, the
   commit hash is recorded by the planner in the follow-up bookkeeping
   commit, never by you) + your handover
   (`agent/handover/handover_task_to_planner.md`, agent repo).
4. Handover: executive summary, measured baseline + final gate numbers,
   commit hash(es), the edge-case → test mapping, how your tap-group
   handling differs from the first attempt (why the regression is avoided),
   anything deliberately not done.
5. FST tree back on `opencode_test`, clean.

## DO-NOT-touch
- `fst_keyboard.py`: the `###XXX 241016-1101` block + its decision comment,
  the `###XXX 241022-1341` block (RULING-KEEP).
- The tap-group section (773-region) — the ruling makes it untouched.
- Anything under the agent repo's `maintainer/` or `agent/prompts/**`;
  `opencode.jsonc` (both repos). No behavior change outside "rebind key
  auto-repeats its target" (tap-group rebinds EXCLUDED per the ruling).
- The agent repo working tree: only your handover file + the FST
  `projects/Free-Snap-Tap/TODO.md` note.

## Approval boundary
The proposal is in `approved/` = the observable behavior change is
MAINTAINER-APPROVED (un-deferred 2026-09-29 by the maintainer). If you hit
a case the proposal does not cover, prefer the conservative choice (keep
current behavior) and note it in your handover — do not invent new
behavior.

## Context discipline (mandatory)
Name areas by the markers above; first greps bounded (`| head -30`); read
only the relevant sections of `fst_keyboard.py` (the 620–790 region + your
test file). FST-repo command output: redirect big runs to a temp file,
check size, report only the summary line.
