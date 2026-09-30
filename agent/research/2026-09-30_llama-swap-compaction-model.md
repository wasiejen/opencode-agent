# 2026-09-30 — llama-swap: the "compaction model" via filters / no-reload routing

Session_ID: ses_f0fef54baffeQ1VEvi6Z0RuTtN
Agent: planner (plan43, iter 43, autorun)

## Problem framing

His ideas.md entry 2026-09-29_23-19 (read-only, uncommitted):

> "compaction model as filters option in llama-swap? no reload needed and thus
> no cache invaliadation. can i set checkpoints to 0 ro presevere the planner
> and worker session in the limited ram cache?"

Context: with `agent.compaction` currently COMMENTED OUT in `opencode.jsonc`
(lines 198-202), compaction runs on the session's OWN model. On the single
llama-swap slot, compacting another session whose model is not the loaded one
triggers a SEQUENTIAL swap (unload session model → boot named model → back),
which invalidates the prompt cache and costs a flush delegation
(`agent/knowledge/knowledge_tools.md` §"llama-swap single slot"). The
commented-out config already sketches the intended shape:
`agent.compaction.model = "llama-swap/Qwen3.8-27B-Compaction"` + `temperature 0.1`.
The question: can llama-swap provide that "compaction model" WITHOUT a reload
(no swap → no cache invalidation), and can the planner/worker models be kept
resident in RAM?

Evidence source: the current main branch of `mostlygeek/llama-swap` —
`README.md` + `docs/config.example.yaml` (fetched 2026-09-30). The live server
(192.168.178.20:8033) was NOT queried (MEM-0108: no direct requests at the
backend); the maintainer can confirm the running build/version.

## Findings

1. **`filters` cannot route the `model` field — it only rewrites parameters.**
   Per-model and per-peer `filters` expose exactly three capabilities:
   `stripParams`, `setParams`, `setParamsByID` ("rewrite parts of requests
   before sending to the upstream server"). Both `setParams` and
   `setParamsByID` state: "protected params like `model` cannot be
   overridden." So a filter alone cannot send compaction traffic to a
   different model. (config.example.yaml, `models.*.filters` + `peers.*.filters`.)

2. **The no-reload "compaction model" IS achievable — as an alias/variant of
   the already-loaded model, or via `selectors`/`profiles`:**
   - `setParamsByID` "combine with aliases to create variant behaviour
     WITHOUT RELOADING THE MODEL — a model alias will be automatically created
     for each key" (e.g. `"${MODEL_ID}:compaction": { temperature: 0.1 }`
     auto-creates the alias `<session-model-id>:compaction`). An explicit
     `aliases:` list does the same manually.
   - opencode-side wiring: set `agent.compaction.model =
     "llama-swap/<that alias>"`. The compaction request names the alias;
     llama-swap resolves it to the SAME model configuration that is already
     running (solver step 2: "If X is already running, forward the request.
     Done.") → zero swap, zero reload, zero cache invalidation.
   - Caveat: an alias of the same configuration means the SAME weights — the
     only difference is opencode-level params (the compaction agent's own
     `temperature: 0.1` already sends that). If the goal is a DIFFERENT
     (lighter) model for compaction, an alias cannot do it — see (3).
   - `profiles` pins ("applied before aliases, filters, and routing"; targets
     "may be a local model, fully qualified peer model, alias,
     setParamsByID alias, or selector") and `selectors` ("virtual model IDs
     resolved to concrete targets per request") are the two routing levers.
     `selectors` with `strategy: warm` = "chooses the first READY target in
     order, then an already-starting target, and cold-starts the first target
     when none are running" — i.e. a compaction model ID like
     `selectors: "compaction-model": { strategy: warm, targets: [
       <session-model>, <light-model> ] }` would prefer the already-loaded
     session model (no swap) and fall back to a lighter model only when
     nothing is running (cold start = a swap anyway).

3. **Keeping planner + worker models resident in RAM: the real mechanisms.**
   The routing section offers two engines (`group` default, `matrix` newer):
   - `groups` with `persistent: true` — "a persistent group that other groups
     CAN NEVER UNLOAD" (members stay loaded; other groups load alongside when
     `exclusive: false` + `swap: false`). This is the documented way to keep
     the planner/worker models resident.
   - `matrix` engine: "When a model is requested, the solver makes room for
     it by evicting as few running models as possible, PREFERRING TO KEEP THE
     COSTLIEST ONES LOADED" (`evict_costs` per model) — concurrency of
     multiple models is expressible, but eviction decisions are cost-based,
     not RAM-based.
   - `globalTTL: 0` / per-model `ttl: 0` ("a value of 0 disables automatic
     unloading") + `hooks.on_startup.preload` (preload models at startup;
     "when preloading multiple models at once, define a group").

4. **There is no `checkpoints` key in the current llama-swap config** (full
   `docs/config.example.yaml` inspected 2026-09-30; keys: health/log/store,
   performance, startPort, apiKeys, security, tailcat, upstream, profiles,
   selectors, models, hooks, routing, scheduler, peers, macros, captureBuffer).
   The closest real mechanisms to "set checkpoints to 0 to preserve the
   sessions in the RAM cache" are (a) `ttl: 0` / `globalTTL: 0` (never auto-
   unload), (b) `persistent: true` groups (never unload by other groups),
   (c) `hooks.on_startup.preload`. If "checkpoints" refers to something in his
   specific build (ik_llama fork / a llama-swap version difference), the
   maintainer should name the exact key — the assumption recorded here is that
   it maps to TTL/persistence.

5. **RAM is the hard constraint, and llama-swap does not manage it.**
   llama-swap tracks which model runs and evicts per the swap engine, but has
   no RAM-capacity check — co-resident models must physically fit in the
   machine's RAM/VRAM. Whether planner model + worker model (+ a compaction
   model) fit simultaneously is a machine fact only the maintainer knows.
   Note the planner and worker model IDs on this host differ (e.g.
   `...Q3S-210K-slow-HQKV` vs `...Q3S-245K-slow`): if they are distinct
   llama-swap model entries (even on the same weights), they count as two
   resident models.

## Recommendations (ranked)

1. **Cheapest win, no backend risk:** point `agent.compaction.model` at an
   auto-created alias of the session model (`setParamsByID: "<id>:compaction"`)
   — compaction always runs on the already-loaded model, no swap ever, and
   the opencode-side temperature already differs. This removes the flush-
   budget rule (knowledge_tools.md item (2)) for the common case where the
   target session's model is the loaded one. (Maintainer's call: he edits
   `opencode.jsonc` + the llama-swap yaml.)
2. **If a lighter compaction model is wanted:** a `selectors` entry with
   `strategy: warm` targeting `[session model, light model]` — no-reload when
   the session model is up (always, in-session), cold-start of the light
   model only otherwise.
3. **For keeping planner+worker resident:** a `persistent: true` group +
   `ttl: 0` + optional `hooks.on_startup.preload` (with the group defined so
   both preload together) — ONLY if the RAM budget fits both models at once.
4. Clarify the "checkpoints" key with the maintainer before configuring
   anything (finding 4) — no matching key exists in current main.

## Status

Research only — nothing built, no config touched. Decision: maintainer call
(he owns `opencode.jsonc` + the llama-swap yaml + the backend). The live
acceptance (does the running llama-swap build match current main's config
surface) can be checked by him at his next backend touch.
