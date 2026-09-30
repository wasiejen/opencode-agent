# plan40 summary (iter 40, unit-4 restart branch)

Session ses_f105e5378ffelMLKGQ4lL6lB7q (planner-40, Qwen3.8-27B-Q3S-210K-slow-HQKV,
210k window).

## Unit 1 — #113 CLOSED (planner-direct, small pre-approved)
- The agent-repo probe's `VENV_PY` re-pointed: agent-repo venv preferred if
  present, else the FST workspace venv (`C:\Users\Wasiejen\Repos\Free-Snap-Tap\.venv\Scripts\python.exe`,
  verified working 3.12.9 — he rebuilt it 2026-09-29; the old location is gone).
- Probe re-run: **352/352 PASS** (was 341 + the 11 env-fails 136-146) — #113 closed.
- TODO curation: #113 → one-line close (FST TODO) + full text (FST todo_records);
  #121 filed + closed inline (the two stale repo_commands.md refs post-#117/#113).
- Commit `9444248` (probe + TODO + NAP + baselines + plan39 compression + dumps).

## Unit 2 — fst-rebind-repeat LANDED (worker-40, FST repo)
- Spec `4229da8` (agent repo `agent/handover/handover_task.md` + loop copy
  `plan40_ho_task.md`) — written against the preserved spec
  (`archive/loop/autorun-2026-09-15_13-11/plan2_ho_task.md`) with the current
  branch/layout facts (the repo-split moved bookkeeping out of the FST repo;
  the stale `44dbc36` bad commit is unreachable in both repos — described
  textually in the spec).
- Worker-40 ses_f10473509ffeTH0wA3XURsl3SG (`worker_Q3S_slow`, same-model
  cache rule): **`e77c01f` on new branch `fst_work3`** (off `opencode_test`
  57e7fdb; code+tests only, 2 files +110/-4; tree back on `opencode_test`,
  clean). The let-through helper `_rebind_repeat_let_through` + region gate
  (press-only; rebind triggers in no tap group whose replacement is in no
  tap group; the tap-group section UNTOUCHED per the 2026-09-15 ruling).
- Gate (worker-measured, planner spot-verified): baseline 459+1w / ruff F=0
  → final **464 passed + 1 warning, ruff F=0** (delta = the 5 new
  `TestRebindRepeat` tests — I re-ran them: 5/5 in 0.19s).
- Edge-case coverage: core repeat, SUPPRESS_CODE, rebind+macro dual trigger,
  press-state bookkeeping, the MUST-pin tap-group ruling case + the old
  behaviour-pinning test repurposed to macro-repeat suppression (the
  discrepancy: my spec's "greenfield" fact missed that one existing test
  pinned the observable old behaviour — friction entry filed by the worker).
- TODO: **#122** (FST TODO — LANDED; live FST testing = maintainer domain).
- Agent-repo close commit `7da4a36` (handover + TODO note + loop log + friction).

## Planner verification (from files, per protocol)
- git: FST `e77c01f` scope = `fst_keyboard.py` + `tests/test_filter_behavior.py`
  only (DO-NOT-touch regions untouched — diff read); agent repo `7da4a36`
  scope exact; FST tree clean on `opencode_test`.
- Targeted spot re-run: `pytest -q -k TestRebindRepeat` on `fst_work3` = 5 passed.

## Baselines (NAP Standing updated)
- Agent-repo gate: probe **352/352** + 11 smokes green.
- FST gate: pytest **464 passed + 1 warning** + ruff **F=0** (new, post-#122).

## Queue for the next session (ordered)
1. context_trim Unit 2 spec (plugin-side post-compaction tail-set) — HELD
   until the Unit 1 live acceptance (his restart — his "let see that the
   'easy' part works").
2. LIVE acceptances pending his restart: #106/#109/#115/#119 budget-key/
   submit `memory` arg/context_trim `report` (read-only — first)/#75 unit
   2+4 + #122 live FST testing (his domain).
3. Idle-lane research: the compact-message non-delivery observation
   (`maintainer/done/compact_message_worker.md`).
4. Knowledge line: the static model/agent switch mechanics (his 2eba484) →
   the opencode-plugins inbox.
