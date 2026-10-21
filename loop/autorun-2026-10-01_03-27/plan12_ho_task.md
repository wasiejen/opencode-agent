# Task spec — v2 model-reader port, unit 1 (session_v2 model + limits read)

Assigned worker: `worker_FlashNext` (same model as the launching planner —
`strata/qwen3.8-flash-next-coder-iq1_m`; the cache rule).

## Goal
In the v2 clone repo `C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch
`migration`), make the shared session-DB reader resolve the session MODEL +
LIMITS from the v2 tables, so the context family (gauge/`ctx_gauge`/
`context_trim`) and the intercept plugin's `model` log field work on v2
sessions instead of returning nothing / `unknown`.

## Definition of done (measurable)
1. `.opencode/plugin/scripts/gauge.mjs` (clone): a v2-shaped read returns
   `id` + `model` + `limit` for a session whose row lives in `session_v2`
   (fixture DB in the existing smoke harness). The v1-shaped read stays green
   on a `session`-shaped fixture (the DB carries BOTH tables — coexistence
   measured 2026-10-21, see the knowledge file).
2. Clone smoke pins: one pin per read form (v1 table / v2 table / an id
   present in neither → the existing not-found outcome). Names carry the
   `v2` marker. The suite count goes up by the pin count, recorded in the
   handover.
3. `.opencode/plugins/intercept_observer.ts` (clone): the `model` field of
   the 8-field log line resolves through the new reader when resolvable;
   otherwise the literal `unknown` (byte-compat of the line shape is
   PRESERVED — one field, pipe-separated, unchanged order; the 2026-10-21
   ruling: byte-compat kept until this port lands, then the real id).
4. Clone gate green: probe + all smokes measured FRESH at spec-relative
   baseline (the worker records the pre-change counts FIRST).
5. Handover `agent/handover/handover_task_to_planner.md` (in the clone's
   tree if that is where its harness lives — the worker follows the clone's
   existing handover convention; state the path used).

## Verified facts (measured at spec-authoring time, 2026-10-21)
- v2 content layout: `session_message(id,session_id,type,seq,data)` +
  `part(message_id,…)`; assistant content INLINE in `message.data`; a v1-style
  `part.session_id` filter returns ZERO rows for v2 sessions (trap).
- `session_v2` columns = `session` PLUS fork_session_id, fork_boundary,
  time_suspended, resume_attempts, time_idle, time_viewed, idle_outcome
  (host-map §3, `agent/knowledge/opencode-plugins/host-map.md` L194) —
  so the `model` column name is unchanged; the TABLE name differs.
- The clone's gauge reader today still queries v1 tables:
  `.opencode/plugin/scripts/gauge.mjs` lines 519 / 525 / 534-536 / 543 /
  557 (`SELECT id, model FROM session …`).
- The clone's plugin header states the port is OUT of the Unit A range
  (clone `.opencode/plugins/intercept_observer.ts` lines ~60-62) and that
  the model slot carries `unknown` per the unavailable→literal house rule.
- v2 CLI runs need an explicit `--model` (the config agent model is NOT
  applied) — knowledge file
  `agent/knowledge/plugin_tools/2026-10-16_v2-plugin-harness-and-config.md`.
- The token-mass leg is NOT in this unit (ordered as unit 2; the reader
  signature must not pre-shape it).

## Suggested scope (non-exhaustive)
- clone `.opencode/plugin/scripts/gauge.mjs` (the read + the table choice)
- clone `.opencode/plugin/tests/` (or `plugin/tests/`, wherever the gauge
  smokes live — locate by the existing gauge smoke file)
- clone `.opencode/plugins/intercept_observer.ts` (the model-field call site)
- the clone's `agent/knowledge/` copy if the clone keeps one; otherwise
  no main-repo docs in this unit.

## DO NOT touch
- the main repo (`C:\Users\Wasiejen\Repos\opencode-agent`) code, gate files,
  `TODO.md`, `agent/prompts/**`, `agent/handover/handover_planner.md`
- the live GLOBAL DB (`~/.local/share/opencode/opencode.db`) — fixture DBs
  only, NEVER the live DB
- the clone root `opencode.jsonc` (maintainer domain) and anything under
  `maintainer/**` in either tree
- the token-mass / context-mass SQL legs (unit 2), the v2 tools estate (later
  unit), pair/numword channels, the log line shape (byte-compat)

## Commit + checkpoints
- One green checkpoint commit per verified unit step in the clone on
  `migration` (a named-path commit; NEVER `git add -A`); the clone's own
  handover/discrepancy notes ride the FINAL commit.
- Status wording: `LANDED` — the hash lands in the planner's follow-up
  bookkeeping commit in the MAIN repo (never in the same commit).

## Live acceptance (pending, name it in the handover)
- The clone `agent/scripts/v2_intercept_acceptance.sh` re-run is the
  MAINTAINER's (agents never run the clone host against the backend slot).
  Expect: intercept lines whose model slot carries the real model id
  (`qwen3.8-flash-next-coder-iq1_m`) instead of `unknown`.
