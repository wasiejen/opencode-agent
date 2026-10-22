# specs/ — pre-written task spec queue

## 2026-10-05 wave (plan11, autorun ses_ef7155f2…)
- `2026-10-05_loop-iteration-counter.md` (approved proposal
  `proposals/approved/2026-10-01_loop-iteration-counter.md`) — the
  machine-side loop counter (Part 1 counter.json at spawn + Part 2 the
  ctx-line `ITER` suffix + the n=11 seed). LANDED plan11 (worker_Q3S_slow
  ses_ef70261b…; probe 360/360, auto_resume 170/170).
- `2026-10-05_fst-logging-build.md` (approved proposal
  `proposals/approved/2026-10-01_fst-logging-build.md`, TODO #131 build —
  FST `projects/Free-Snap-Tap/TODO.md:22`) — one FST worker unit on
  fst_work3: the DEBUG flags → structured logging (3 ordered units) +
  the 4 audit bugs + the maintainer-requested usage guideline in
  `maintainer/feedback/`. LANDED plan11 unit 2 (worker-11
  ses_ef6b793f…; FST ea2aac3/cac6704/cd0f3a2; gate 474+1w, ruff F=0).

## 2026-10-04 wave (plan10)
- `2026-10-04_p3-state-snapshot-tool.md` (approved proposal
  `proposals/approved/2026-09-18_p3-state-snapshot-tool`) — the planner
  init digest tool. LANDED plan10 (worker_Q3S_slow; probe 358/358 +
  state_snapshot 20/20); proposal → implemented/ with verdict; live
  acceptance = the next planner init after the host restart.

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
## Wave 2026-10-22 — the cross-repo access unit (TODO #140; maintainer GO 2026-10-22, config-driven roots + the path-field truncation fix)
- `2026-10-22_cross-repo-roots-A.md` — `block_transfer` sandbox roots become CONFIG-DRIVEN (main repo + clone twin). Launch channel: copied into `agent/handover/handover_task.md` (2026-10-22).
- `2026-10-22_path-field-truncation-B.md` — the intercept log NEVER cuts inside a single-path field (main repo + clone twin).
- Order: A then B (independent changes; both fix TODO #140; #141 = the clone gate red baseline the clone numbers must be diffed against).

## Wave 2026-10-22 — the v2 clone gate (TODO #141; maintainer priority item 2, first step)
- `2026-10-22_v2-clone-gate-141.md` — repoint the clone probe + smoke v1-module paths to `.opencode/plugin-unported-v1/` (or v2 twins) until the clone gate prints real green counts. Launch channel: copied into `agent/handover/handover_task.md`.

## Wave 2026-10-22 (planner-15, iter 15)
- `2026-10-22_137u2-token-mass-legs.md` — TODO #137 unit 2: the clone gauge TOKEN-mass legs v2-aware (clone `migration` only; gate green; live v2 CTX acceptance = maintainer). Launch channel: copied to `agent/handover/handover_task.md`.
- `2026-10-22_15-smoke-env-bt-regions.md` — two small pre-approved fixes: sandbox smoke ENV robustness (todo_inbox 01-34) + bt WRITE N-regions-with-1-text REJECT (todo_inbox 03-26; REJECT recommended end state), BOTH trees.
- Order: unit 2 (clone gauge) first, then the small-fixes spec.

## Wave 2026-10-22 (planner-16, iter 16) — TODO #137 unit 3, the v2 CONTEXT FAMILY (clone `migration`)
- `2026-10-22_137u3a-context-family-core.md` — new clone plugin `.opencode/plugins/context_family.ts`: gauge readout via the shared v2-paired gauge core + the `ctx:` line injection at the v2 `session.context` hook (append-at-tail, cache discipline) + evidence lines + its own bun smoke + tsgo coverage. Launch channel: copy to `agent/handover/handover_task.md`.
- `2026-10-22_137u3b-nudge-ladder.md` — extends the SAME plugin: 5-rung nudge ladder at `execute.after` (per-session read, verbatim v1 rungs, dedup, silence rules) delivered via `ctx.session.synthetic(delivery:"queue")` fire-and-forget + the `subagent` handover pre-flight warn. Launch AFTER 3a verifies (same file, additive).
- Order: 3a then 3b; both clone-only, live v2 acceptance = maintainer (backend slot).

## What does NOT go here
NAP, non-compaction specs.
