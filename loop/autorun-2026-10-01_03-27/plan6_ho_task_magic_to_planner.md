# Handover — Magic Context plugin research (explorer_Q3S_slow)

2026-10-01. Research-only task; repo unchanged except the deliverables.

**Deliverable:** `agent/research/2026-10-01_magic-context-plugin.md`
(committed `6862348`, checkpointed before this handover). All findings
source-cited (their repo paths + quotes).

The 3 direct answers (≤10 lines):
1. **Memory** — project-scoped facts in 5 categories stored in one shared
   local SQLite DB (`~/.local/share/cortexkit/magic-context/context.db`);
   written by (a) the historian's auto fact-promotion while it summarizes
   history, (b) the agent via `ctx_memory`, (c) an overnight dreamer cron
   (verify/curate/classify). Loaded back always-on: baseline block in
   frozen `m[0]` + in-session delta in `m[1]`, budgeted to 4000 tokens;
   on demand via `ctx_search` (+ per-turn "vague recall" hint); lossless
   recovery via `ctx_expand`.
2. **How/when they trim** — agent marks droppable units with `ctx_reduce`
   (queues only, `[dropped §N§]` placeholder, applied on the next
   cache-busting pass); the plugin fires a hidden-model **historian** at
   its own triggers (execute threshold 65%, force band 85%, commit
   clusters, 3× trigger-budget tail) and replaces old raw history with
   **tiered compartments** (4 detail tiers + importance 1–100, decayed by
   age/importance/budget via a deterministic no-LLM renderer; history
   budget 15%); 95% ⇒ turn refused. Everything recoverable from raw
   `source_contents`.
3. **Agent vs plugin** — **agent-informed, plugin-executed**: the agent
   can only queue drops and write memories; every trigger, threshold,
   boundary, protected tail (≈30% formula), TTL and the 95% refusal is
   plugin-side; the user tunes thresholds via `magic-context.jsonc`.

Adoption section: 4 borrowable candidates (usage-adaptive keep formula,
explicit keep-N surface via our validated `context_trim` mechanics,
importance-scored per-chunk summaries, gauge nudge bands) each with effort
+ approval class, plus a not-to-borrow list (cache-aware scheduling,
dreamer, separate historian model — all need a second model slot /
provider cache this host lacks). (2) is the clearest concrete build →
planner triage as maintainer call if wanted. No build done (task scope).

Commit: `6862348` (doc). No TODO entries; no discrepancies found.
Note: this file superseded the prior #132 worker handover (already consumed —
planner verified it in `87927fa`; archived copy in the loop folder; full
prior content in git history at `87927fa^`).
Lessons: GitHub HTML pages are navigation-heavy — `raw.githubusercontent.com`
fetches kept all research output clean; one large page (CONFIGURATION.md)
truncated to a saved file and was grepped in place (no context bloat).

Gauge (verbatim):
SESSION=ses_f09d877e4ffe7sI5ZnOHKWKBQ6 CTX=89546 (38%) REM=145454 | 5 compactions left
