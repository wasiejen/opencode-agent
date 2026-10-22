# Task spec — TODO #137 unit 2: gauge TOKEN-mass legs v2-aware (clone `migration`)

## Goal
Make the gauge's token-mass read work against the v2 DB shape, so a v2-hosted
session gets a real `CTX=` readout instead of `CTX=notAvailable`. The session-row
leg is already table-aware (unit 1); the mass leg still reads v1 `message`/`session`
only.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch
  `migration` (checked out; stay on it). Main repo = NOT touched.
- Assigned worker: `worker_FlashNext` (same model as the launching planner —
  the cache rule; 262k window).
- TODO entry: `TODO.md` #137 (main repo — status line rides the bookkeeping
  commit by the planner; the worker writes its own handover only).

## Definition of done (measurable)
1. A v2-shaped fixture DB (tables `session_v2` + `session_message`, shaped
   like real migrated rows — see Verified facts) yields an OK gauge read:
   `total`/`output` resolve from the v2 mass form; the readout line shows
   `CTX=<n> (<p>%) REM=<m>` (with the compactions suffix as today).
2. A v1-only DB keeps ALL existing gauge_core pins byte-identical
   (all 32 PASS lines green, unchanged).
3. A coexisting DB: the newest-session mass comes from the SAME table that
   won the session-row pick (`session_v2` FIRST / tie rule — unit 1 semantics
   preserved; no cross-table mass pairing).
4. The per-session form (non-empty sessionID targeting a v2 session) works
   (scoped `session_message` read; the spawn backend's marker SQL variant
   covers the v2 tables too).
5. All backends in the chain handle the v2-aware read — the in-process
   backends AND the spawn-sqlite3 marker SQL (the production backend live).
6. Gate green in the clone: `node .opencode/plugin/probes/handover_probe.mjs`
   prints `PROBE handover: 360/360 PASS`; `node
   .opencode/plugin/tests/gauge_core.smoke.mjs` ALL PASS (32 pins + the new
   pins per form); the other 11 smokes ALL PASS unchanged.
7. Checkpoint commits per verified unit IN THE CLONE (code only — the
   precedent: #141 clone commits carried no handover); write the worker
   summary to the MAIN repo working tree at
   `C:\Users\Wasiejen\Repos\opencode-agent\agent\handover\handover_task_to_planner.md`
   (uncommitted — the planner commits it with its bookkeeping; the #141
   precedent). Clone tree clean after.

## Suggested scope (non-exhaustive)
- Clone `.opencode/plugin/scripts/gauge.mjs` (841 lines) — the mass-leg SQL
  constants + the per-session forms + the backend read paths.
- Clone `.opencode/plugin/tests/gauge_core.smoke.mjs` — new pins per form
  (v2-only OK read; coexisting pick pairing; per-session v2; v1-only
  unchanged stays pinned).
- Clone `.opencode/tools/ctx_gauge.ts` twin + `.opencode/plugin/scripts/peek.mjs`
  ONLY if the result shape needs adapting (they call `readGauge`; likely no
  change).
- Reference (READ-ONLY): `agent/research/2026-10-15_opencode-v2-impact.md`
  lines 156-219 (§2 DB v1→v2, §2 recommendation block); the fork
  `C:\Users\Wasiejen\Repos\opencode` branch `production` (v2 line),
  `packages/core/src/database/schema.gen.ts` (session_v2 columns ~202-208:
  `cost`, `tokens_input/output/reasoning/cache_read/cache_write` —
  session-CUMULATIVE counters, NOT the mass) + `packages/core/src/session/**`
  (where the message `data` blob is written — find the per-message usage shape).

## Verified facts (measured at spec-authoring, 2026-10-22)
- Clone gauge twin: `.opencode/plugin/scripts/gauge.mjs` — session-row leg
  table-aware via `sessionTablesIn`/`sessionRow` (:588-629, `session_v2` FIRST,
  tie keeps `session_v2`); the MASS leg constants are v1-only: `SQL_FINISHED_STEP`
  (:531-537), `GAUGE_SQL_MARKER` (:542-547), `SQL_FINISHED_STEP_SESSION`
  (:555-561), `sqlMarkerForSession` (:562-570) — all `FROM message m JOIN
  session s ... json_extract(m.data,'$.tokens.total'/'$.tokens.output') ...
  LIKE '%"finish"%'`. Header comment :526 says "The TOKEN-mass leg ...
  UNTOUCHED — unit 2."
- `gaugeFromRaw` (:781-805): NaN total → `{ok:false,kind:"no-total",sid,
  modelId,window}` (unit 1 shape); OK → `ctx = total` (#103: in+out+cr at
  the start of the NEXT turn; `output` kept as evidence, NOT subtracted).
- Fork `production` @654b79b27e: `grep -c session_v2 packages/core/src/
  database/schema.gen.ts` = 11; `session_message` = `id, session_id, type,
  seq, time_created, time_updated, data` (research §2:160-165); NO `part`
  table; data blob = message-level JSON (research §2:180-182).
- Clone baselines measured now: `gauge_core.smoke.mjs` → `GAUGE_CORE_SMOKE:
  ALL PASS` (32 PASS lines); `ctx_gauge.smoke.mjs` → 3/3; probe 360/360
  (TODO #141 clone gate landed GREEN); unit-1 smoke forms 4/5 exist (coexisting
  newest + NEITHER-table db-error form preserved).
- The v2 per-message usage storage path in `session_message.data` is NOT yet
  verified — the ONE research question of this task: grep the fork's
  `packages/core/src/session/**` (or `history.ts`/`info.ts`/execution write
  path) for the usage/tokens shape written into the message `data`; optionally
  ONE `mode=ro` sqlite3 SELECT on the live migrated DB (same pattern as the
  spawn backend) to see a real v2 row. Bounded — one question.

## Approval boundary
Pre-approved class (agent-usage tool fix, no maintainer-visible behavior
change). DO NOT touch: the MAIN repo (any file), `maintainer/**`, the live
DB file except mode=ro reads, the fork repo, `.opencode/plugin-unported-v1/**`
in the clone (frozen v1 twins — the gauge.mjs shim re-exports the clone core;
that is intended, do not edit the shim unless a check requires it).

## Procedure (guidance, not protocol)
1. Read this spec + the unit-1 area of clone gauge.mjs (:505-640, :780-841)
   + the gauge_core smoke's v2unit1 pins.
2. Answer the research question (fork grep; one optional mode=ro query).
3. Implement: mass leg table-aware (mirror the session-row pattern; keep the
   v1 SQL constants byte-identical for v1-only DBs; spawn variant per form).
   Keep the `#103` semantics: ctx = total = in+out+cr of the latest FINISHED
   step; map the v2 usage shape to that total (compute in SQL json_extract
   arithmetic or JS post-map — worker's call; a v2 `finish`/in-flight
   discriminator: use what v2 actually stores, verify, do not guess).
4. Pin smoke forms per DoD 1-4 (fixture DB builder — extend the existing
   fixture helpers); run: gauge_core smoke, ctx_gauge smoke, the full clone
   gate (12 smokes + probe).
5. Checkpoint commits per verified unit (clone, code only); write the worker
   summary into the MAIN repo `agent/handover/handover_task_to_planner.md`
   working tree (planner commits it); clone tree clean.
6. Final message points at `agent/handover/handover_task_to_planner.md`
   (in the clone).

## Deliberately out of scope
- The v2 tools estate (port map §4 order 3+), ctx line injection, nudge
  ladder, the MAIN repo v1-side dual-support, live v2-host acceptance
  (maintainer domain — after this lands, his next v2 session shows `CTX=n`;
  state that as PENDING in the handover).
