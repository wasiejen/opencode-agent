# Task spec — Magic Context plugin research (maintainer ideas.md 2026-10-01)

Role: `explorer_Q3S_slow` — RESEARCH ONLY (read-only on our repo, no code
changes, no branch work).
Repo: opencode-agent.

## Goal
Answer the maintainer's question — verbatim from the `maintainer/ideas/
ideas.md` entry "2026-10-01: scope out Magic Context plugin" (READ-ONLY
there; the entry is `--wip--defer` but scanning/research is explicitly
allowed):
- "how they work with memory and how they make the best usage of the
  context window (how do they trim it? when do they trim? decided by agent
  or by the plugin?)"

Subject: the open-source plugin `magic-context` —
https://github.com/cortexkit/magic-context (tree `master`).

## Scope (research only)
Web research (bounded — fetch the README + key source files INDIVIDUALLY,
one fetch at a time; never bulk-pull a whole tree; large pages → read the
relevant sections, don't echo them):
1. **The memory mechanism** — what they persist (what, where, what format),
   how it is created/updated, and how it is loaded back into context.
2. **Context-window management** — the trim/summarize/evict mechanism: WHAT
   is trimmed, WHEN (trigger: token threshold? message count? explicit
   command?), and HOW (drop vs summary vs externalize to memory).
3. **The control surface** — what the AGENT can direct (tools/commands/
   config/params) vs what the PLUGIN decides autonomously (schedulers,
   thresholds, policies).
4. **Compaction/retention relevance** — their retention policy, summary
   form, any token budgets or keep semantics.

Our-repo context for the adoption section (given, do not re-derive):
- Our retention levers: head-dropping compaction with a summary (keep =
  messages/tokens — #99/#120), the `context_trim` tool (validated
  `tail_start_id` rewrite, floor 6), the plugin-side post-compaction tail-
  set (#120 U2 — every COMPACT line rewrites the fresh part's tail to the
  agent-requested keep boundary).
- Our open areas: per-call keep control (on this host only the config
  `compaction.keep.tokens` — #99 close note), the real-time gauge (#116).

Deliverable: ONE research doc at `agent/research/2026-10-01_magic-context-
plugin.md` —
- dated header; every finding carries its SOURCE (their repo file path,
  quoted code/line where it matters);
- each of the maintainer's sub-questions (memory / how+when they trim /
  agent-vs-plugin control) answered DIRECTLY with its findings;
- a short closing "Adoption implications" section for our workflow: what,
  if anything, is worth borrowing for our retention/compaction track —
  each candidate with an effort estimate + approval class (pre-approved /
  maintainer call). No build in this task — recommendations only.

Also (where applicable): verified actionable findings → `submit`
(knowledge / todo) per the shared protocol; a proposal ONLY if a concrete
build clearly emerges (the planner triages the build as a maintainer call
unless pre-approved class).

## Definition of done
- The research doc answers the 3 sub-questions with dated, source-cited
  findings (file paths in THEIR repo — not our guesswork).
- The adoption section present (may be "nothing to borrow" — that is a
  valid finding, stated with reasons).
- Our repo UNCHANGED except: the research doc + the handover + any
  knowledge/todo_inbox/proposal additions per above.
- Worker handover in `agent/handover/handover_task_to_planner.md`: pointer
  to the doc + the 3 direct answers in ≤ 10 lines.
- Checkpoint: the doc committed before the handover commit (the handover
  rides the final commit). No new TODO entry expected; discrepancies via
  todo_inbox.

## DO-NOT-TOUCH
- Every repo file except the deliverables above; `maintainer/**`
  READ-ONLY; `AGENTS.md`; `agent/prompts/**`; no code, no config, no
  branch work, no push.
