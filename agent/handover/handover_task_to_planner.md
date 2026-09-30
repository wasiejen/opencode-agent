# handover_task_to_planner.md — plan47: TODO #86 deferred audit (EXPLORER, FINAL)

Session ses_f0f768f33ffe79T7V9zKQGH6ao (explorer_Q3S, llama-swap/Qwen3.8-27B-Q3S-170K).
Task DONE — read-only audit of ALL agent plugin/tool/script code complete.
**Zero code changes** (verified: `git status` shows only todo_inbox.md + this file modified).

## Findings per dimension (11 total, all in todo_inbox.md — 4 clusters via submit)

| dimension | findings | severity spread |
|---|---|---|
| 1. stale agent/model ids | 2 | both low (comment/description only) |
| 2. stale ses_* in non-test code | 0 | the 3 example ids in db/README.md verified to still exist |
| 3. duplicated config | 3 | 1 med, 2 low |
| 4. magic numbers | 5 | 1 med, 4 low |
| 5. dead code | 0 | all 125 exported symbols verified used (machine-checked) |
| 6. stale doc/path refs | 1 | low |

## Top findings inline
1. [med, dup-config] `BUSY_TIMEOUT_MS = 2500` defined twice — gauge.mjs:151 AND context_trim.ts:88 (context_trim already imports from gauge.mjs). Fix: export + import, drop the local.
2. [med, magic] ctx_watchdog.ts:394-398 — the nudge-rung ladder thresholds (5000/10k/20k/30k + 90/80/70/50) are inline, unnamed, no config source.
3. [low, stale-ids] loop_log.ts:127 — tool-description example `Qwen3.8-27B-IQ4KT-120K` is not a live model id.
4. [low, stale-ids] submit.ts:34/:141 — comment example `planner_Q3S_245K_slow` is a stale agent name.
5. [low, stale-ref] auto_resume.ts:173-177 — the looprunner "OVERLAP-ERA CAVEAT" describes a retired agent (looprunner_Q3S commented out) as if live.
6. [low, magic] auto_resume.ts:1536 `0.99 * window`, :1538 `60_000` idle, :1885/:1887 `20MiB`/`2MiB` defaults, :1901 `5000` tick — inline unnamed.
7. [low, dup] db-path string in 5 standalone .cjs scripts + gauge.mjs DEFAULT_DB_PATH; HOST_EXE in binwin/binhits.

Full per-finding detail (file + line + suggested fix): the four `plan47 TODO #86 cluster A-D`
entries appended to `todo_inbox.md` this session (stamps 2026-09-30_06-30/_06-31).

## Verified clean (negative results — do not re-derive)
- No `Q3S_[0-9]+K` agent ids and no commented-out agent names (looprunner_Q3S, agent_Q3S,
  worker_Q3S_HQKV, planner_Q3XS, worker_Q3XS, worker_gemma_Q4) anywhere in live
  plugin/tool/script code. The only such strings in scope are intentional test fixtures
  (auto_resume.smoke.mjs, submit.smoke.mjs, compact_memory.smoke.mjs) — not findings.
- The #85 part-2 `PLANNER_AGENT_ID` case (the #86 exclusion) was not re-reported.
- Dead code: all exported symbols in scope (125) are used — internally, cross-file, or by
  tests. No orphan constants.
- Stale refs: no pre-#118 `agent_readme_*` filename references remain in live code (the
  plan46 cluster is resolved); all checked doc/path references (readme_post_compaction.md,
  handover_task.md, ctx.log, compact_budget.json, dump_session.cjs, peek.mjs, gauge.mjs)
  resolve to existing files.

## Observations (NOT todo_inbox findings — out of scope, planner's call)
1. `.opencode/temp/compact_budget.json` model_budget lists model ids not in the live roster
   (Q3XS-160K-MTP, Q3S-110K-MTP, Q2S-128K-x2, Q2S-210K-MTP, Q3XS-262K). The file is a live
   temp config (gitignored, outside the scope file list) — stale entries there are harmless
   (they only grant caps to unlisted models), so not reported as a finding.
2. The explorer role prompt's instruction index references `agent/readme/repo_map.md`,
   `repo_testgate.md`, `repo_gotchas.md`, `agent_readme_todo.md` — none exist in
   `agent/readme/` (actual set: README, readme_loop, readme_post_compaction,
   readme_proposals, readme_task_spec, readme_todo, repo_commands, repo_custom_tools,
   repo_opencode, repo_overview). The prompt file (agent/prompts/agent_explorer.md) likely
   carries the same stale index — prompt files are outside this audit's scope.

## Deliberately not done
- No code changes at all (read-only task). No gate runs. Test logic not audited (per spec).
  `.opencode/plugin/deactivated/`, `.opencode/tools/deactivated/`, `.opencode/archive/`,
  `projects/`, `maintainer/`, `opencode.jsonc` untouched.
- No per-finding TODO.md entries (explorer deliverable = todo_inbox loose entries; the planner
  assigns IDs at curation).

## Files touched
- `todo_inbox.md` (4 cluster entries appended via submit)
- `agent/handover/handover_task_to_planner.md` (this file)
- `loop/autorun-2026-09-21_15-33/loop_log.md` (START + DONE lines via the loop_log tool)
- NOT committed / NOT touched: `maintainer/ideas/ideas.md` carries an uncommitted
  maintainer-side edit (2026-09-29_23-19 entry, not mine — left as-is, per the
  DO-NOT-TOUCH list).

Gauge (final, verbatim): see the DONE loop-log line / closing message.
