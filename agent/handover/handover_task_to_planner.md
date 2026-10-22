# Worker summary — TODO #137 unit 2: gauge TOKEN-mass legs v2-aware (clone `migration`)

Status: COMPLETE. Both checkpoint commits are in the CLONE; the clone tree is
clean; the main repo was NOT touched (its gauge.mjs misfire was reverted,
git-clean state verified). This file is left UNCOMMITTED in the main-repo
working tree (the #141 precedent — the planner commits it with its
bookkeeping).

## Task and scope

Goal: the gauge's TOKEN-mass read works against the v2 DB shape, so a
v2-hosted session gets a real `CTX=` readout. Unit-1 table awareness
(session-row leg) was already in the clone; this unit ported the mass leg
and the spawn-sqlite3 marker SQL (the production backend), preserving the
v1 SQL byte-identically.

## The research question — ANSWERED (the v2 usage shape EXISTS)

`session_message` rows carry a `type` column; the assistant `data` blob
stores `tokens: {input, output, reasoning, cache:{read, write}}` and
`time: {created, streamed?, completed?}` + optional `finish`
(`@opencode/schema/session-message.ts` `Assistant` + `token-usage.ts` at
fork production @654b79b27e; `database/v1-migration.bun.ts` writes the same
shape; ONE `mode=ro` live-DB read confirmed real rows, e.g. input 72 /
output 176 / cache.read 14283 / reasoning 0 / cache.write 0).
- Mass TOTAL mapped to the fork's OWN `TokenUsage.total()` =
  input+output+reasoning+cache.read+cache.write — the #103 in+out+cr
  semantics of v1's `tokens.total` mapped to what v2 actually stores
  (llama-swap keeps reasoning/cache.write at 0, so the sum lands where v1's
  total landed).
- FINISHED-STEP discriminator = the fork's own completed-assistant filter
  (projector.ts fork-copy): `type='assistant' AND $.time.completed IS NOT
  NULL AND $.tokens IS NOT NULL` — in-flight assistant rows lack both;
  the `type` match also keeps `compaction` rows (their REQUEST usage, not
  the context size) out.

## Files changed (CLONE only, branch `migration`)

1. `.opencode/plugin/scripts/gauge.mjs`
   - Mass-leg constants: `SQL_STEP_DISCRIMINATOR_V2`, `SQL_TOKEN_TOTAL_V2`
     (COALESCE'd five-field sum), `SQL_FINISHED_STEP_V2`,
     `SQL_FINISHED_STEP_SESSION_V2`, `GAUGE_SQL_MARKER_V2`, `SQL_TABLE_PROBE`,
     `sqlMarkerV2ForSession` (+ `sqlSidLiteral` extraction — v1 builder output
     unchanged).
   - `sessionRow` also REPORTS the winning TABLE; `readApiDb` pairs the
     mass SQL with it (session_v2↔session_message, session↔message).
   - `readSpawnSqlite3`: probe exec (sqlite_master names) → per-shape marker
     instantiation; v1-only DBs keep the OLD marker constants BYTE-IDENTICAL;
     coexisting session tables resolved by a `W`-candidate exec (greater
     time_updated, tie keeps session_v2) then the marker PINNED to the
     winner's sid (pairing by construction); target ids resolve by holding
     table (session_v2 wins). NO session table → throw (db-error shape kept).
   - v1 SQL constants untouched; header/catch comments updated for unit 2.
2. `.opencode/plugin/probes/handover_probe.mjs` — pin RELABEL only (check
   32/33 mock mass keys accept both v1 and v2 mass forms; check-32 step
   assertion keyed on `session_message`). Precedent: unit 1 relabeled these
   same pins. TOTAL stays 360/360.
3. `.opencode/plugin/tests/gauge_core.smoke.mjs` — 10 new pins + header
   note; the 32 pre-existing pins byte-unchanged (all still green).

NOT changed (verified no adaptation needed): `.opencode/tools/ctx_gauge.ts`,
`.opencode/plugin/scripts/peek.mjs` (shape-preserving; they call
`readGauge`/`formatGauge` only), the plugin-unported-v1 shim, the main
repo, the fork, the live DB (read-only queries only: 3 bounded sqlite3
`mode=ro` statements).

## Measured verification (clone, run 2026-10-22)

- `node .opencode/plugin/tests/gauge_core.smoke.mjs` → `GAUGE_CORE_SMOKE:
  ALL PASS`, 42 PASS lines (32 pre-existing + 10 new: v2-shaped OK mass 44150
  (in-flight + compaction rows excluded), newest form, readout line
  `SESSION=ses_v2s_fx CTX=44150 (36%) REM=75850`, pairing both directions
  (v2 winner 31000 / v1 winner 11111), spawn-forced v2 per-session, v2 newest,
  coexisting W-pick both directions, v1-only byte-identical 43210).
- `node .opencode/plugin/tests/ctx_gauge.smoke.mjs` → `CTX_GAUGE_SMOKE:
  ALL PASS (3/3)`.
- `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover:
  360/360 PASS`.
- All 12 smokes exit 0 (loop over `.opencode/plugin/tests/*.smoke.mjs`),
  probe 360/360 — gate green in the clone.
- Exact verify command (clone root):
  `node .opencode/plugin/tests/gauge_core.smoke.mjs && node
  .opencode/plugin/tests/ctx_gauge.smoke.mjs && node
  .opencode/plugin/probes/handover_probe.mjs` → PASS / PASS / PASS.

## Commits (CLONE, branch `migration`; code only per #141 precedent)

- `922f74c` port gauge TOKEN-mass read to v2 (table-paired + spawn marker
  v2 form + probe mock relabel) — gate green at this commit.
- `2c1fa23` pin gauge_core v2 unit-2 forms (smoke pins, 42 PASS lines).

Working tree: clone `clean`; main repo dirty ONLY in the host-written
loop files (`loop/autorun-2026-10-01_03-27/counter.json`,
`loop_log.md`) + this uncommitted summary — as the channel table intends.
Main repo gate: NOT touched (per spec).

## TODO / inbox entries

- submit: knowledge (v2 usage-shape facts, verified 2026-10-22 — the
  handover's research answer, for the v2-tools estate port map),
  feedback (file tools resolve relative paths against the SESSION CWD — the
  first clone edit misfired into the main repo and was reverted via
  `git checkout`; cross-repo tasks must always pass absolute paths),
  memory (worker: same lesson).
- TODO #137 status line: planner-side bookkeeping (not mine).

## Deliberately omitted / pending

- Live v2-host acceptance: PENDING (maintainer domain) — after this lands
  and the host restarts, his next v2 session shows a real `CTX=n` readout
  (no `notAvailable`) from the in-process backend; the spawn marker is
  pinned end-to-end with the REAL exe.
- v2 tools estate port (§4 order 3+), ctx line injection, nudge ladder,
  main-repo dual-support — out of spec (§Deliberately out of scope).
- Deviations from the suggested procedure: none material; the spawn design
  uses a probe exec + winner-pinned marker (instead of instantiating one
  marker blindly) to guarantee DoD 3 pairing — worker's call per spec §3.

## Lessons

One reused: gauge pin suites mock at the PREPARE level, so a SQL-shape
change must be mirrored in the probe mocks/pins in the SAME commit (unit-1
precedent made this the accepted pattern).
