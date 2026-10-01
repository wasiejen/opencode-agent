# Research — Magic Context plugin (cortexkit/magic-context, tree `master`)

Date: 2026-10-01. Researcher: explorer_Q3S_slow. Read-only web research; no code
touched in our repo. Subject answers the maintainer's question (ideas.md
2026-10-01): *how they work with memory and how they make the best usage of
the context window (how do they trim it? when do they trim? decided by agent
or by the plugin?)*

Sources cited are paths in THEIR repo (`github.com/cortexkit/magic-context`,
branch `master`); code paths relative to `packages/plugin/` as they state it.
Key sources: `README.md`, `ARCHITECTURE.md`, `CONFIGURATION.md`,
`docs/architecture/historian.md`, `docs/architecture/reclaim.md`,
`docs/architecture/memory-and-search.md`, `docs/architecture/nudges.md`,
`packages/plugin/src/tools/ctx-reduce/tools.ts`.

Big picture: it is a **whole-session context manager that replaces the host's
built-in compaction entirely** (setup writes `opencode.jsonc`
`"compaction": { "auto": false, "prune": false }` — README "Manual setup").
It rewrites the outgoing message array + system prompt on every model call
(`ARCHITECTURE.md`, "How a session is managed": "Magic Context rewrites the
message array and system prompt the host is about to send, on every model
call"), and runs summarization/maintenance in hidden child sessions
(historian, dreamer). The design goal is **prompt-cache stability**: nothing
it does ever busts the cached prefix by its own action — every mutation
"rides a bust that happens anyway" (`ARCHITECTURE.md`, "Pass taxonomy":
"Deferred work rides the next bust cycle; it never forces its own").

---

## Q1 — The memory mechanism

### What they persist, where, in what format
- **Project memories**: short project-scoped facts in five categories —
  `PROJECT_RULES`, `ARCHITECTURE`, `CONSTRAINTS`, `CONFIG_VALUES`, `NAMING`
  (`docs/architecture/memory-and-search.md`, "Memories": "New writes use
  five categories…"). Rows carry a normalised-content hash (exact dedup),
  an importance/scope classification, retrieval counts, and verification
  state maintained by the dreamer (same source).
- **Storage**: one local SQLite database shared by all hosts/hosts,
  `~/.local/share/cortexkit/magic-context/context.db` (README "Storage";
  `ARCHITECTURE.md` "Storage"). Session-scoped tables carry a `harness`
  column; project-scoped tables (memories, git commits) are shared across
  hosts (`ARCHITECTURE.md` "Storage"). Memories are keyed to a stable
  project identity: `git:<root commit sha>` for git repos, `dir:<hash>`
  otherwise (`docs/architecture/memory-and-search.md`, "Memories",
  `project-identity.ts`).
- **Everything else durable lives in the same DB**: tags, raw source
  contents, compartments, notes, primers, events, embeddings (SQLite
  blobs), the FTS5 index of raw messages, the git-commit index
  (`docs/architecture/memory-and-search.md` "Where the code is" +
  `README.md` "Storage"). Nothing is lost on trim: "The original content is
  kept in `source_contents`, which is what `ctx_expand` and every replay
  read" (`docs/architecture/reclaim.md`, "Tags and tag identity") — "which
  is why no drop or compression on the reclaim page ever loses information"
  (`memory-and-search.md`, "Notes and expand").

### How it is created / updated
Three writers (`memory-and-search.md`, "Memories"): "Memories come from
three writers: the historian's fact promotion (when `memory.auto_promote`
is on), the agent through `ctx_memory`, and the dreamer."
- **Historian auto-promotion**: while the historian compresses old history
  into compartments, the same pass "lifts out the knowledge worth keeping
  forever… and promotes it into project memory" (README "Capture"). Each
  compartment emits `<facts>` in the five memory categories (
  `docs/architecture/historian.md`, "Producer"); on publish, "Facts are
  promoted to project memories when `memory.enabled` and `memory.auto_promote`
  are on, with exact deduplication" (`historian.md`, "Publish").
- **Agent writes**: `ctx_memory` offers `write`, `update`, `archive`,
  `merge`, `get` to primary agents (`memory-and-search.md`, "Memories").
- **Dreamer maintenance** (overnight cron child sessions): verify memories
  against code, curate duplicates, classify importance, retrospective
  lessons, user-profile review (`README.md` "Consolidate";
  `CONFIGURATION.md` dreamer task table with cron schedules, e.g.
  `verify: "0 3 * * *"`, `curate: "0 4 * * 0"`).

### How it is loaded back into context
- **Always-on injection**: baseline memory block lives in `m[0]` (frozen
  synthetic first message); new in-session memories surface through `m[1]`
  (volatile delta) via a `maxMemoryId` watermark; updates/archive/merge
  render as a `<memory-updates>` delta; both fold into `m[0]` on the next
  natural "HARD fold" (`memory-and-search.md`, "How memories reach the
  prompt without a bust").
- **Budgeted**: `memory.injection_budget_tokens` — "Token budget for memory
  injection into `<session-history>`", default **4000**, range 500–20000
  (`CONFIGURATION.md`, `memory` table).
- **On demand**: `ctx_search` (one query across memories, raw conversation
  history, git commits, compartment summaries, notes — semantic embeddings
  with FTS5 fallback, `memory-and-search.md` "Unified search"); **auto
  search** runs a background search each turn and appends only a compact
  "vague recall" hint to the triggering user message (never full content)
  when top score > `auto_search.score_threshold` (default 0.6) (
  `memory-and-search.md` "Auto-search"; `CONFIGURATION.md`
  `memory.auto_search`).
- **Recall is lossless**: `ctx_expand` pulls any compressed/dropped range
  back to the original `U:`/`A:` transcript (README "Recall";
  `memory-and-search.md` "Notes and expand").

---

## Q2 — Context-window management: what, when, how

### WHAT is trimmed
Three distinct lanes, all reversible (nothing destroyed — everything
re-derivable from `source_contents` / raw history):
1. **Compartmentalization (summarize/externalize)** — old raw history is
   replaced by **compartments**: chronological summaries, each with four
   paraphrase tiers `p1` (full detail) … `p4` (anchor only), an importance
   score 1–100, episode type, `<facts>`, `<events>`
   (`docs/architecture/historian.md`, "Producer"). The summarized raw
   messages then get queued drops (see below), so the live window holds
   summary + live tail only.
2. **Drops (deletion with placeholder)** — individual tagged units
   (`§N§` tags on every message part / tool output) are removed and
   replaced by the single placeholder `[dropped §N§]` — "a pure function
   of the tag id" (`reclaim.md`, "Agent drops"). Lanes: agent-requested
   (`ctx_reduce`), **age reclaim** (automatic drop of old tool outputs ≥
   250 tokens, two-step watermark), **heuristic cleanup** (dedup repeated
   read-only tool results, strip injected system content), **smart drops**
   (opt-in: superseded edits compressed to a 40-char region marker),
   **emergency drops** (force-band: tool outputs removed "by need instead
   of by position", oldest-first by tool tier, newest 20% of tiers 1–2
   held back) (`reclaim.md` — sections named).
3. **Strips (provider-needless content)** — old reasoning
   (`clear_reasoning_age` default 50 tags), structural noise, stale
   placeholders, processed images, stale `ctx_reduce` calls — applied
   once and replayed byte-identical thereafter (`reclaim.md`, "Strip and
   replay"). Optional **caveman compression** (off by default):
   deterministic age-tiered prose compression, oldest 20% → `ultra`,
   next 20% → `full`, next 20% → `lite`, newest 40% untouched
   (`reclaim.md` "Caveman compression"; `CONFIGURATION.md`
   `caveman_text_compression`).

### WHEN (triggers — all thresholds are plugin-side, not agent-side)
- **Execute threshold**: `execute_threshold_percentage` default **65**
  (20–90), per-model `execute_threshold_tokens` variant available —
  "Context usage that forces queued ops to execute. Capped at 90% of the
  output-reserved safe window" (`CONFIGURATION.md`, "Core"). Below it, the
  scheduler **defers** (SOFT+ pass: byte-identical replay, nothing applies);
  at/above it a pass **executes** (flushes queued ops into that pass).
- **Historian trigger** (`historian.md`, "Trigger"; budget = 5% of the
  usable window = context limit × execute threshold, clamped 5,000–50,000
  tokens), reasons in order:
  1. **Force band**: usage ≥ `max(85%, threshold + 2%)` (≈92% at the
     90% threshold cap) — fire, "unless queued and automatic drops are
     already projected to bring usage down to 75% of the threshold";
  2. **Commit clusters**: ≥3 distinct work phases ending in commits
     (`commit_cluster_trigger`, default on, `min_clusters: 3`) — trim at
     work-unit boundaries even at low pressure;
  3. **Tail size**: eligible (condensed-chunk) tail reaches **3×
     triggerBudget**;
  4. **Projected headroom**: usage ≥ proactive floor (threshold − 2%) with
     drops not projected sufficient.
- **Pressure backstop**: at the force band, queued work is applied and
  emergency reclaim runs; "at a provider-proven 95% with nothing folded,
  the turn is **refused** rather than sent over the limit"
  (`ARCHITECTURE.md` "Pressure"; `reclaim.md` "Heuristic cleanup and
  emergency drops" — at 95% even the protected window is given up, keeping
  only open arcs and the three newest `ctx_reduce` calls).
- **Explicit commands**: `/ctx-wrapup [messages_to_keep]` (on-demand
  compaction of older history, default keep 20), `/ctx-recomp`
  (rebuild compartments from raw history), `/ctx-flush` (force queued ops
  now, bypassing cache TTL) (`ARCHITECTURE.md` "Pressure";
  `historian.md` "Recomp and wrapup"; README "Commands").

### HOW — the cache-aware mechanics
- **Defer/execute scheduler** decides per pass; defer passes must replay
  byte-identical; "Deferred work rides the next bust cycle; it never forces
  its own" (`ARCHITECTURE.md`, "Pass taxonomy" + invariants).
- **m[0]/m[1] head layout**: `m[0]` = frozen cumulative baseline
  (project docs, user profile, decay-rendered compartments); `m[1]` =
  volatile delta (new memories, newest compartments at full tier). HARD
  fold = m[0] re-materializes (cache already dead — model change, system
  prompt hash change, idle > TTL, genuine content change, or m[1]/m[0]
  size-ratio backstop "OR an absolute m[1] token cap (~20% of history
  budget)") (`ARCHITECTURE.md`, "m[0]/m[1] cache layout").
- **Decay rendering** (deterministic, no LLM): one tier per compartment
  from age, importance and budget pressure:
  `H = H50 · 2^((importance − 50) / D) / max(p, 0.10)`, H50 = 24, D = 25;
  `z = (age in compartments) / H`; tier P1/P2/P3/P4 by z-cutoffs
  0.201/0.729/1.322/2.587, else archived. `p` = budget pressure from
  `history_budget_percentage` (default **15%** of usable context).
  "The renderer makes no model calls and adapts automatically when the
  context window or budget changes. Re-tiering only happens on a HARD
  fold" (`historian.md`, "Decay rendering").
- **Protected tail**: the live suffix the historian never summarizes is
  sized in raw tokens: `N = clamp(round(usable × 0.3 × (1 − usage)),
  floor, ceiling)` — floor 8% of usable (bounded 2,000–12,000), ceiling
  min(96,000, 40% of usable, usable − headroom reserve) (`historian.md`,
  "Protected-tail boundary"). Per-run eligible prefix capped ≈2/3/4× N at
  normal / 80%+ / 95%+ pressure.
- **Protected window for drops**: automatic reclaim never touches the
  newest content: `protected_tokens` default
  `clamp(round(0.05 × usableSoft), min(16,000, round(0.08 × usableSoft)),
  64,000)`; the three newest `ctx_reduce` calls are kept in every lane
  (`reclaim.md`, "The protected window").

---

## Q3 — Control surface: agent vs plugin

### What the AGENT can direct
Tools (all "queue/ask", none execute immediately on their own):
- **`ctx_reduce`** — the only direct context lever. Argument: one string,
  tag ids/ranges (`"3-5"`, `"1,2,9"`) —
  `packages/plugin/src/tools/ctx-reduce/tools.ts`: `drop: tool.schema
  .string() … describe('Tag IDs to drop: "3-5", "1,2,9", "1-5,8,12-15".')`.
  It **only queues**: "each id becomes a `drop` row in `pending_ops`…
  Queued drops are applied on the next pass that is busting anyway, never
  on their own" (`reclaim.md`, "Agent drops"). Ids in the protected window
  are accepted but reported "Held". The agent sees `§N§` tags on every
  droppable unit, and **nudges** tell it when to trim (below).
- **`ctx_memory`** — write/update/archive/merge/get project memories
  (`memory-and-search.md`).
- **`ctx_search`**, **`ctx_expand`**, **`ctx_note`** — recall tools
  (README "Agent tools at a glance").
- Agent opt-out: per-agent tool allow-list — "remove or deny `ctx_reduce`
  in that agent's tool allow-list; Magic Context then omits `§N§`
  prefixes, nudges, and reduce guidance for that session"
  (`CONFIGURATION.md`, `caveman_text_compression` section).

**Nudges** (plugin → agent direction, `docs/architecture/nudges.md`):
measurement `{U, T}` — T = eligible tail mass, U = the droppable part
outside the protected window; fixed bands on ratio U/T: quiet
(U/T < 0.20), gentle ≥ 0.20, firm ≥ 0.40, urgent ≥ 0.60, Channel 2 at
U/T ≥ 0.75 and U ≥ 50k. Channel 1 = `<system-reminder>` appended to the
next tool output ("states how many droppable tool outputs there are and
how much they hold, and lists the oldest reclaimable tags as hints");
re-fire only after U grew by `max(25k, 0.08 × T)` or 5+ user turns.
Channel 2 = synthetic steering message at event boundaries, once per tail
cycle. So the *decision of what to keep is the agent's*; the *timing
pressure and hints are the plugin's*.

### What the PLUGIN decides autonomously
- The entire **scheduler**: defer vs execute per pass; cache TTL
  (`cache_ttl`, default "5m", per-model map, "never" sentinel) converts
  idle defer passes into execute passes (`CONFIGURATION.md`, "Cache
  Awareness").
- **All thresholds**: execute threshold (65%), force band (85%), 95%
  refusal, trigger budget (5%), protected tail (30% formula), protected
  window (derived), history budget (15%), m[1] cap (~20% of history
  budget), tier-cutoff constants.
- **Historian + decay rendering** (which history becomes which
  summary tier, when to fire, boundary, validation, publish — all
  plugin-side; historian runs on a **separately configured, cheaper
  model**: `historian.opencode.model` required — README "Manual setup").
- **Dreamer cron tasks** (15-minute timer, per-task cron schedules, leases,
  child sessions — `CONFIGURATION.md` "How scheduling works").
- **Emergency drop** policy and 95% turn refusal — no agent input
  (`reclaim.md`).
- **Fail-closed**: if storage can't be opened, the transform "refuses
  loudly on every pass instead of letting the prompt grow unmanaged"
  (`ARCHITECTURE.md`, "Failure handling").
- **Conflict fail-safe**: it disables itself if another context manager is
  active (built-in compaction, DCP, OMO hooks) — README "Compatibility".

### What the USER (config) can direct
`magic-context.jsonc` (project `.cortexkit/magic-context.jsonc` + user
`~/.config/cortexkit/magic-context.jsonc`): historian/dreamer models +
fallbacks, `enabled`, `fail_closed_blocking`, `cache_ttl`,
`execute_threshold_percentage`/`execute_threshold_tokens`,
`protected_tokens`, `history_budget_percentage`, `output_reserve`,
`clear_reasoning_age`, `commit_cluster_trigger`, `memory.*` (enabled,
injection_budget_tokens, auto_promote, auto_search, git_commit_indexing),
`temporal_awareness`, `caveman_text_compression`, `smart_drops`,
`system_prompt_injection` (skip-signatures to exempt agents),
`prompt_surface` (full/light guidance presets), dreamer schedules
(`CONFIGURATION.md` — full reference).

**Direct answer to the maintainer's question**: trimming is **agent-informed,
plugin-executed**. The agent can only *mark* content droppable
(`ctx_reduce` queues, never applies) and the plugin owns every trigger,
threshold, boundary, and the actual application — which is deliberately
deferred to cache-safe moments. The agent never initiates a trim that
busts the cache; the plugin never trims on the agent's behalf outside its
own schedules — except that agent-reduction hints (nudges) are computed
from the agent's own droppable mass.

---

## Q4 — Compaction / retention relevance

- **Retention policy = tiered decay, not cutoff**: old history is never
  deleted from the DB — it's summarized into compartments and *rendered*
  at a fidelity tier per compartment (importance 1–100 = decay half-life;
  75 doubles H50, 100 quadruples it; pressure accelerates decay).
  "History degrades gracefully as it ages" (README "Context management").
- **Summary form**: compartments with 4 paraphrase tiers + importance +
  facts + events, produced by a bounded-prompt hidden-model call (seed
  examples, last 6 compartments, memory block — "never sends a full state
  dump"; `producer-window-guard.ts` verifies the prompt fits the model
  window with 3% margin) (`historian.md` "Producer"). **Discard-last**:
  the last compartment of a run is dropped and re-read next run with real
  lookahead (boundary quality) (`historian.md` "Validation").
- **Token budgets / keep semantics**:
  - `history_budget_percentage` default 15% of usable context — "Triggers
    compression when exceeded" (`CONFIGURATION.md` "Core").
  - Protected (raw-kept) tail: 30%-of-usable formula above (keeps the live
    conversation raw; never splits a tool call from its result).
  - Protected window for drops: ≈5% of usable soft window (derived, 64k
    ceiling).
  - `/ctx-wrapup [messages_to_keep]` — the only explicit **keep-N
    messages** lever (default 20): "compacts older raw history into
    compartments on demand while keeping the newest messages raw"
    (`historian.md` "Recomp and wrapup").
  - `memory.injection_budget_tokens` 4000 default for the memory block.
  - m[1] delta cap ≈20% of history budget before a backstop refold.
- **vs our model**: they keep the **tail raw** and **age the head**
  (compartments decay from the front), whereas our compaction drops the
  head with a summary and keeps a recent tail (keepMessages). Their
  "keep" is expressed in *protected raw-tail tokens* (usage-adaptive:
  `usable × 0.3 × (1 − usage)`) rather than message count or tokens — the
  closest analogue to a per-call keep knob is their `/ctx-wrapup
  [messages_to_keep]` (message count) and `protected_tokens` (token
  floor). Their summary is *incremental and tiered* (per-chunk
  compartments, re-rendered per tier) rather than one whole-session
  summary — i.e. their "summary" is a persistent data structure with a
  deterministic renderer, not a prose block regenerated on demand.
- **Cost/behavior deltas**: they run two hidden-model pipelines (cheap
  historian model; dreamer on idle) — our single-model host (one
  llama-swap slot, no background model) cannot run that shape today;
  their entire scheme is optimized around **provider prompt caching**,
  which is largely absent/unknown on this host (no cache pricing, single
  local slot) — the defer/execute/flush machinery buys little here.

---

## Adoption implications (our retention/compaction track)

Context: our levers are head-dropping compaction with a whole-session
summary (keepMessages / `compaction.keep.tokens` — #99/#120), the
`context_trim` tool (validated `tail_start_id` rewrite, floor 6), and the
plugin-side post-compaction tail-set (#120 U2). Our open areas: per-call
keep control (#99 close note), the real-time gauge (#116).

Candidates, each with effort + approval class:

1. **Usage-adaptive protected tail (their 30% formula)** — make the
   retained raw tail size a function of usage: `keep ≈ usable × 0.3 ×
   (1 − usage)` (bounded), so near-full sessions keep less raw history
   and leave headroom for in-turn growth. Directly attacks our open area
   "per-call keep control" (#99) — today keep is message/token-count,
   static per call. Effort: M (one formula + clamp in the keep
   computation of our compact path; needs a probe gate for
   re-derivation stability). Approval: **maintainer call** (changes
   observable context behavior for all sessions).
2. **Explicit keep-N command surface (their `/ctx-wrapup [messages_to_keep]`
   / `ctx-wrapup` analogue)** — a host command / tool parameter
   `keep: N` that compacts older history while keeping the newest N
   messages raw. We already have the mechanics (`context_trim` rewrites
   `tail_start_id` to a validated boundary, floor 6); this is mostly
   surfacing it as a first-class, agent-requestable per-call keep.
   Effort: S–M (reuse the validated tail rewrite; add a command +
   doc). Approval: **maintainer call** (new agent-facing control surface;
   though it only re-uses existing validated mechanics).
3. **Importance-scored tiered summary (their compartments)** — replace /
   enrich the single prose summary with per-chunk summaries carrying an
   importance score, rendered at a fidelity tier under a history budget
   (15% analogue). Their decay math (H50=24, D=25, tier cutoffs) is
   deterministic and borrowable as-is. This is the biggest structural
   idea — but it assumes their head-render model (m[0]/m[1] two-slot
   synthetic messages, cache-aware replay), which our host does not
   support; adopting only the *per-chunk + importance scoring* (summary
   structure, not the cache machinery) is a meaningful step. Effort: L
   (new summary schema + renderer + storage; multi-commit track).
   Approval: **maintainer call** (new subsystem; also interacts with our
   compaction-summary model choice #99 close note).
4. **Nudge bands for our gauge (#116)** — their `{U, T}` ratio bands
   (0.20/0.40/0.60/0.75) are a clean, deterministic scheme for a
   real-time "your tail has droppable mass" nudge, mappable onto our
   gauge (#116, open). Effort: S (band arithmetic on existing gauge
   readout + one injected reminder line). Approval: **pre-approved class
   candidate** (agent-side plugin/tool friction remover only — but it
   injects text into sessions, so it is *borderline*; treat as
   maintainer call if the injection is prompt-visible).
5. **Nothing-to-borrow list** (stated with reasons):
   - **Defer/execute cache-aware scheduling** — our host has no
     meaningful provider prompt cache on the single local slot; the
     whole flush-into-a-bust machinery has no cost target to optimize.
   - **Dreamer (overnight cron child sessions, memory verification
     against code)** — requires a second/background model and idle-time
     sessions; not runnable on this host's single model slot.
   - **Historian as separate cheap model** — same: no second model slot;
     our compaction model pair is fixed per compaction.
   - **SQLite shared cross-session memory DB + embeddings** — a large
     standalone system; our memory track is file-based by design
     (`agent/memory/`); not worth porting for our scale (1–2 roles, one
     repo).
   - **Fail-safe plugin conflict detection** — we run one context
     manager (ours); no coexistence problem.

Net: the borrowable ideas are (1) usage-adaptive keep, (2) explicit
keep-N surface, (3) importance-scored per-chunk summaries, (4) nudge
bands. All four are recommendations only — no build in this task.
Proposal trigger per task spec: (2) is the one where a concrete build most
clearly emerges from existing machinery — flagged for planner triage as a
maintainer call if the maintainer wants it.
