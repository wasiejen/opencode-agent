# Task spec — TODO #137 unit 4b: context_trim ported to v2 (clone `migration`) — QUEUED, launch AFTER 4a verifies

## Goal
Port the `context_trim` tool (report + tail) to v2: the report reads the v2
session tables; the tail rewrite becomes a plugin-side NON-destructive window
filter (the `session.context` hook + a persistent boundary store) — the v2 analog
of the host's own `tail_start_id` rewrite, which does NOT exist in v2.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (after unit 4a lands). Stay on the current checkout.
- Assigned worker: `worker_FlashNext`.
- TODO entry: `TODO.md` #137. Design of record (READ-ONLY): main
  `.opencode/tools/context_trim.ts` (1168 lines — read the HEADER contract + the
  report/tail shapes via grep, NOT the whole file) + the compact_memory v2 twin
  landed by spec `2026-10-22_137u4a-compact-memory-v2.md` (evidence-file +
  registration patterns).

## Definition of done (measurable)
1. NEW plugin `.opencode/plugins/context_trim_v2.ts` (name open — worker may pick
   `context_trim.ts`; the v1 name in the unported dir is precedent for a same-name
   twin) exposing the tool via `ctx.tool.transform` `editor.add` (same surface as
   4a DoD 2): args mirror v1 — `mode:"report"` (`session?`, `target?`) /
   `mode:"tail"` (`session`, `target?` message id OR `keepMessages=N`).
2. REPORT — reads the v2 tables (the clone gauge core is ALREADY v2-paired —
   reuse `readGauge`/its helpers rather than new SQL where possible): the model
   window = the `session_message` rows after the LAST completed `compaction` row
   (the v2 window model — the context endpoint's own description states it);
   per-message token mass = `session_message.data.tokens{input,output,
   reasoning,cache.read/write}` + the `$.time.completed` discriminator (unit-2
   finding); the readout forms mirror v1 (numbered rows, `last:`, recommendation
   block, biggest-droppable) adapted: anchors = v2 message ids; no `part` table,
   no `tail_start_id`.
3. TAIL — v2 has NO `tail_start_id` field (measured: the window =
   after-last-compaction; there is NO host lever to rewrite it). The v2 lane:
   a persistent per-session BOUNDARY store (`.opencode/temp/context_trim_v2.json`
   — sessionID → boundary messageID; fail-open) + a `ctx.session.hook("context")`
   handler in the SAME plugin that, per primary request, DROPS `event.messages`
   entries strictly before the boundary (assembly-time filter, persisted history
   UNTOUCHED = the v1 non-destructive + reversible semantics). Validation
   mirrors v1 fail-closed: boundary exists in the session's rows, sits STRICTLY
   before the last compaction row? — re-check: in v2 the boundary must sit inside
   the current window (after the last compaction row) and leave a retained tail of
   >= 6 messages (the floor carries); rejections = the plain
   `tail-set= rejected: <cause>` lines. `keep=0` refused.
4. The tail store's effect must survive compaction: when a new durable `compaction`
   row lands, the boundary is stale if before it → the hook logs ONE
   `boundary-stale` line and keeps the whole post-compaction window (fail-open),
   same discipline as v1 (the DB keeps everything; nothing is lost).
5. SMOKE — new `.opencode/plugins/tests/context_trim_v2.smoke.mjs` (bun, mock
   ctx + the hook trigger, scratchpad fixture v2-shaped DBs — reuse the gauge
   fixture pair): report row forms + mass sums over a v2-shaped fixture; tail
   set/clear; floor-6 refusal; unknown-session refusal; the hook filter drops
   exactly the pre-boundary entries and appends NOTHING; stale-boundary fail-open
   + the ONE line. Final line `SMOKE: ALL PASS`.
6. GATE — clone probe `360/360` UNCHANGED; all node smokes ALL PASS; all v2
   plugin smokes ALL PASS; tsgo zero errors (add the plugin to the tests tsconfig
   `files`); clone tree clean; MAIN repo untouched except the summary write.
7. Checkpoint commits in the clone (code only); worker summary to the MAIN
   working tree `agent/handover/handover_task_to_planner.md`.

## Verified facts (measured at spec-authoring, 2026-10-22, fork @ current checkout)
- NO `tail_start_id` anywhere in v2: the context endpoint = "all messages after
  the last compaction" (`packages/protocol/src/groups/session.ts:569-579`
  annotation + handler `server/src/handlers/session.ts:512`); window selection =
  the durable `compaction` row (compaction.ts `previousCompaction` ~690-694:
  the FIRST row of the window, if type compaction + status completed).
- `session.context` domain method = GET `/api/session/:id/context` →
  `Array<SessionMessageInfo>` (client.ts:858-868; types SessionContextInput/
  Output ~4685) — a ready report source for the CURRENT window.
- The `session.context` HOOK (`ctx.session.hook("context", …)`) fires per PRIMARY
  request with MUTABLE `messages` (model-request.ts:427-428 +
  `:422-431`; the 3a context_family plugin already uses this hook — the trim
  plugin registers its OWN handler; handler order independent, the filter mutates
  the same draft the injection appends to — APPEND-only discipline of 3a stays
  intact by context_family; the trim handler runs on its own draft copy semantics
  = verify at worker time that both plugins' mutations compose (both mutate
  `event.messages` on the SAME draft — pin the composition in the smoke).
- v2 revert lane (`session.revert.stage/clear/commit`, protocol
  `groups/session.ts:560-566`) is a DIFFERENT feature (rewind work), NOT a trim —
  do NOT map tail to revert.
- v2 message rows carry `data.tokens` per message (unit 2:
  `session_message.data.tokens{input,output,reasoning,cache.read/write}` +
  `$.time.completed`).

## Suggested scope (non-exhaustive)
- NEW: clone `.opencode/plugins/context_trim_v2.ts` + its smoke + tsconfig entry.
- Reference READ-ONLY: main `.opencode/tools/context_trim.ts` (header +
  readBudget/floor/validation greps, ~lines 1-120 + the rejection strings), the
  4a plugin + `compaction_core_v2.mjs` (patterns), clone
  `.opencode/plugins/context_family.ts` (the `context`-hook mutation pattern),
  clone `.opencode/plugin/scripts/gauge.mjs` (paired table reads).

## Approval boundary
Pre-approved class (agent-facing; live acceptance = maintainer's v2 hosts).
DO NOT touch: MAIN repo (except summary); `maintainer/**`; clone
`.opencode/plugin-unported-v1/**`, `plugins/intercept_observer.ts`,
`plugins/context_family.ts`, `plugins/compact_memory.ts` (4a's file — its
composition pins live in the NEW smoke only); `opencode.jsonc`; live DB.

## Procedure (guidance)
1. Grep the v1 tool for the arg surface + rejection strings (do NOT read all
   1168 lines); read 4a's plugin header.
2. Implement report (fixture-backed), then the tail store + hook handler.
3. Smoke (incl. the composition pin with a simulated context_family append),
   typecheck, full clone gate; checkpoint commits; summary to MAIN.

## Deliberately out of scope
- The destructive-lane question (row deletion) — v1 never had it, v2 stays out.
- 4c (dump twin) and the small tools (order 5-8) — separate specs.
- Live v2 acceptance (maintainer domain).
