# specs/ — pre-written task spec queue

## 2026-09-30 wave (direct session ses_f0e129deeffeqmM5rc8mpnTY2Q)
- `2026-09-30_compact-message-unit4.md` (TODO #127) — restart-branch
  inheritance (compact-message-delivery item 4, maintainer "in favor").
- `2026-09-30_compact-memory-cross-override.md` (TODO #128) — the
  caller-scoped compaction override (ruling-adjusted: cross total = self
  total).
Queue order: #126 (context_trim spawn fallback — LANDED + live-accepted
2026-10-01, plan1) → #127 (LAUNCHING 2026-10-01, plan2, live in
`agent/handover/handover_task.md`) → #128.
Status (2026-10-01): all committed; #127 + #128 LANDED (plan2, worker_Q3S,
gates green); #128 live acceptance + the AGENTS.md Budgets-block paste
(maintainer domain) pending.

## 2026-09-24 wave (trusted compaction system) — COMPLETE

Purpose: task specs written by the planner (2026-09-24, direct session) from the
consolidated change list (`maintainer/inbox_planner/compaction_feedback_by_planner.md`)
— pre-compact spec wave per the maintainer's ruling. Each spec is committed before
launch; a new planner (or this one post-compaction) launches them in queue order.
- `spec_item01_keepTokens.md` — change-list item 1 (his --maintainer item): remove
  keepTokens from compact_memory (top of queue = also the current `handover_task.md`).
- `spec_item10_emergency1.md` — item 10: the emergency-1 budget (total 6, post-drain,
  either system, once). Depends on item 1 (same file).
- `spec_item2_11_auto_resume.md` — items 2 + 11: post-compaction message relay +
  forced-new-session directive (auto_resume.ts).
- `spec_item3_budget_left.md` — item 3: "N compactions left" in the ctx readout.
- `spec_items4-9_prompt_wave.md` — items 4-9 (+12 note): prompt/knowledge text
  updates, executed by a PLANNER agent in text-worker mode (workers have no edit
  access to `.opencode/agent/prompts/**`). Runs LAST, after the build specs.
Queue order: 01 → 10 → 2+11 → 3 → prompt wave.
Status (2026-09-24): 01 = LANDED (7f253ea; probe S10/S11 carve-out accepted,
rides #93); 10 = LANDED (04053e8; auto side design-only → #93 port must carry
it); 2+11 = LANDED (526e7e1; narrow exception used — queueMessage persists);
3 = LANDED (9d2e727; suffix via formatGauge — ctx line/peek/ctx_gauge; live
effect awaits the host restart). Wave: task d = LANDED (16e0bfb, worker_Q3S_230K_slow; item-4 corrected inline
by the planner); c/e/g/h = LANDED (14e9352, planner_Q3S_230K_slow text-
worker). REMAINING: the worker-prompt parts (task g primer bullet + task h
Context-budget section + 1 stale pre-spec-2+11 line) — deferred, awaiting
his prompt_agent_task.md commit; f2-cleanup = his 4 looprunner lines in
AGENTS.md (wave spec lists them).
What does NOT go here: launched specs' worker summaries (handover folder root),
NAP, non-compaction specs.
