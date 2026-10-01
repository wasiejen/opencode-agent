# plan41 summary (iter 41, unit-4 restart branch)

Session ses_f102c4a2dffezocrIY7kzd44YA (planner-41,
Qwen3.8-27B-Q3S-210K-slow-HQKV, 210k window).

## Unit 1 — compact-message non-delivery (queue item 3, idle-lane research)
- The maintainer's observation (`maintainer/done/compact_message_worker.md`)
  confirmed + root-caused from source + files:
  - Delivery = the auto_resume unit-4 recovery path ONLY
    (auto_resume.ts L1349-1357, single call site).
  - (A) scope-none sessions (all Task-tool workers + direct sessions, the
    L1303 gate) are NEVER delivered; their `task_id` resume bypasses the
    plugin entirely.
  - (B) an in-scope session that closes with an `action:` line after
    compaction strands the file (the restart branch never reads it —
    code-verified, not yet live-observed).
  - (C) write-before-dispatch (compact_memory.ts L494/L523): the queue file
    exists even when the dispatch failed → zombie file; the relay delivers
    without checking that a compaction happened (measured: planner-39's
    file — relayed, NO COMPACT line in ctx.log, NO budget increment).
  - Measured on disk: 3 of 4 queued files unconsumed (both workers + the
    direct session); only planner-39's (recovery path) consumed.
- Deliverables: research doc `agent/research/2026-09-30_compact-message-
  delivery.md` (incl. the answers to his two questions — the
  deliberate-vs-limit discriminator table + the consumed-file cleanup
  design) + proposal `proposals/2026-09-30_compact-message-delivery.md`
  (rec: (1) protocol-side delivery in readme_post_compaction.md +
  (2) zombie guard + (3) init-time age/orphan sweep — pre-approved
  class; (4) restart-branch inheritance = his call).
- Knowledge entry appended to `agent/knowledge/knowledge_plugins.md`.

## Unit 2 — live-acceptance triage (queue item 2, from files — no live probes)
- **#106 PASS** — `fuzzy-edit applied … d=1` + `hint rejected
  reason=d-too-high/no-anchor-line` lines live in intercept.log (4 events
  post-restart).
- **#119 PASS** — `compact_budget.json`: all pre-existing config keys
  survived the post-restore increment (14:59:49Z, ses_f130ae200); the
  pre-#119 code would have rewritten the file sessions-only.
- **#75 unit 2 PASS** — `nudge=` lines live (ratios 0.857/0.887/0.895 —
  the budget-file 0.85 threshold honored).
- **#75 unit 4 PASS** — `recovery=` fired + `route= restart spawn` with
  the correct `agent=planner_Q3S_slow` identity (planner-40 00:05:45Z,
  planner-41 01:00:24Z).
- Still pending: #109 (no limit-stop incident since the build),
  context_trim `report` (registration = his domain — Unit 2 HELD),
  submit `memory` arg (no live fire yet), #122 (his domain), #115 (not
  discriminating — config 210000 == the name marker; readouts normal).

## Unit 3 — knowledge curation (queue item 4)
- The static model/agent switch entry (his 2eba484, appended to the
  knowledge inbox 2026-09-30 by planner-39) curated into
  `agent/knowledge/opencode-plugins/host-map.md` §7 (per-message
  agent/model semantics, complete reprefill on switch, STATIC whole-
  session switch = the intended use; the context-trim "model toggling"
  item closed on that basis).

## Records
- Unmarked maintainer idea (NAP-only, no action): ideas.md 2026-09-
  29_23-19 (llama-swap compaction-model-as-filter + checkpoints=0).
- His `maintainer/ideas/ideas.md` live edit remains uncommitted (NOT
  staged).

## Baselines
- Agent-repo gate: probe 352/352 + 11 smokes green (unchanged — no code
  touched this session).
- FST gate: pytest 464 passed + 1 warning + ruff F=0 (unchanged).

## Queue for the next session (ordered)
1. context_trim Unit 2 spec — still HELD until the Unit 1 `report` live
   acceptance (his restart + registration).
2. The compact-message-delivery proposal — awaiting his ruling (then the
   build, pre-approved class).
3. The live-acceptance residue (#109 / submit `memory` / #122) — natural
   occurrence / his domain.
4. MAINTENANCE PASS at iter 45 (counter trigger).
