# Worker summary — v2 model-reader port, UNIT 1 (session MODEL read on `session_v2`)

Status: LANDED (unit 1 complete; unit 2 = the token-mass legs, not started).
Session: ses_e9fa32e59ffemX2wEPTSI2WMYJ (`worker_FlashNext`,
`strata/qwen3.8-flash-next-coder-iq1_m`), looprun `autorun-2026-10-01_03-27`.
Repo: clone `C:/Users/Wasiejen/Repos/opencode-agent-v2`, branch `migration`
(verified `git branch -v` at start: `* migration 28e4fa5 [ahead 3]`).
Handover path used: `agent/handover/handover_task_to_planner.md` IN THE CLONE
(the clone carries its own `agent/handover/` harness — the clone's convention
per spec DoD 5). The main repo was NOT edited (read-only per spec).

## Task and scope
Per the main repo's spec `agent/handover/handover_task.md`: make the shared
session-DB reader resolve `id` + `model` + the resolved LIMIT for a session
row living in `session_v2`, pin one read form per shape in the clone's
smokes, and wire the clone plugin's `model` log field through that reader
(`unknown` otherwise, 8-field pipe shape byte-compat).
NOT touched: the token-mass / context-mass SQL legs (unit 2), the v2 tools
estate, the clone root `opencode.jsonc`, `maintainer/**`, the live global DB.

## Files changed (clone, 3 + this handover)
1. `.opencode/plugin/scripts/gauge.mjs` — the SESSION-ROW leg is now
   TABLE-AWARE: `sessionRow`/`sessionTablesIn` read `session_v2` FIRST, then
   `session` (sqlite_master probe); an id present in BOTH → `session_v2`
   wins; the newest-session form across both tables takes the greater
   `time_updated`; a DB with NEITHER session table still throws (the v1
   db-error outcome preserved). The v1 SQL goes out BYTE-IDENTICAL on a
   v1-only DB (`SQL_NEWEST_SESSION`/`SQL_SESSION_BY_ID` are now the
   `session` instantiation of the table templates). The `no-total` result
   shape gains `window` = `resolveWindow(modelId)` — the LIMIT half (config-
   first, name-marker fallback); `modelId` was already the `{"id":…}`
   unwrap via `parseModelId`. The TOKEN-mass legs (`SQL_FINISHED_STEP*`,
   the spawn MARKER SQL) are UNCHANGED; that leg degrades to null (no-total)
   only when the v1 `message`/`session` tables are absent.
2. `.opencode/plugin/tests/gauge_core.smoke.mjs` — +5 pins (27 → 32), one
   per read form, names carry the `v2unit1` marker: (1) v1 `session` table
   read (sid + model id + mass + window), (2) `session_v2` read (sid +
   JSON-unwrapped model + `window:120000`, lands as no-total), (3) an id
   in NEITHER table (the existing not-found outcome), (4) the coexisting
   DB newest form (the v2 row's greater `time_updated` wins), (5) a DB with
   no session table (db-error preserved). Fixture DBs live under the
   scratchpad (`freshSandbox("gauge_v2_read")`) — NEVER the live DB.
3. `.opencode/plugins/intercept_observer.ts` — the model field now
   resolves: value import `readGauge` from `../plugin/scripts/gauge.mjs`,
   the v1 pattern restored verbatim (`getModel` + TTL-bounded
   `modelCache`, `MODEL_CACHE_TTL_MS = 60_000`), called AFTER the "nothing
   to log" gate and BEFORE the (4b) hard-stop throw, so intercept-error
   lines carry the real id too. Empty sid → `unknown` (no cross-session
   fallback); any lookup failure → `unknown`, never a throw. Header note
   (the old "reader port is OUT of this unit") replaced.
4. `.opencode/plugins/tests/smoke.mjs` (bun) + `README.md` — 45 → 46
   checks: the gauge core is STEERED at a scratchpad FIXTURE DB
   (`fx_gauge_v2.db`) carrying a `session_v2` row for the mock sid (module
   identity: the smoke imports the same `gauge.mjs` URL the plugin imports,
   then `setDbPath`); C1 and the whole-log INV now pin the model field =
   `smoke-v2-model-120K`; a new pin fires the hook with an id present in
   NEITHER table → the field stays `unknown`. Byte-compat of the 8-field
   shape re-pinned unchanged.

## Behavior changed
- Gauge/`ctx_gauge`/`context_trim` session reads on v2 sessions resolve
  sid + model id + the resolved limit instead of `unknown`/empty. The
  readout line format is UNCHANGED (`formatGauge` ignores `window` when
  `!ok`); the "N compactions left" suffix now keys off the v2 row's model.
- Intercept log lines: the `model` slot carries the real model id when
  resolvable, `unknown` otherwise (the 2026-10-21 byte-compat ruling
  ends here for that field only).
- Regression note (intentional): a v1-shaped DB that LACKS the `message`
  table now returns `no-total` with the session row resolved, where v1
  returned `db-error`.

## Measured verification (exact commands, PASS/FAIL counts)
Baseline (measured FRESH at `28e4fa5`, before any edit):
- `node .opencode/plugin/probes/handover_probe.mjs` → exit 1, 0 checks:
  `ERR_MODULE_NOT_FOUND … .opencode/plugin/ctx_watchdog.ts` (pre-existing:
  commit `5516a83` moved the unported v1 plugin sources out of `.opencode/
  plugin/`; the probe still imports them) — RED at baseline.
- `node .opencode/plugin/tests/<11>.smoke.mjs` (auto_resume, block_transfer,
  block_transfer.sandbox, compact_memory, context_recovery, context_trim,
  ctx_gauge, intercept_observer, loop_log, state_snapshot, submit) → 0
  PASS / 0 FAIL each: same pre-existing `ERR_MODULE_NOT_FOUND` for the
  moved v1 sources — RED at baseline.
- `node .opencode/plugin/tests/gauge_core.smoke.mjs` → 27 PASS / 0 FAIL,
  `GAUGE_CORE_SMOKE: ALL PASS` — GREEN.
- `bun run smoke.mjs` (in `.opencode/plugins/tests/`) → 45 PASS / 0 FAIL,
  `SMOKE: ALL PASS` — GREEN.
- `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p tsconfig.json`
  → exit 0, 0 error lines — GREEN.

After (`1b7a202`): probe exit 1 (output BYTE-IDENTICAL to baseline,
`diff` clean); the 11 untouched smokes BYTE-IDENTICAL to baseline (`diff`
clean, 0 PASS/0 FAIL each); `gauge_core.smoke.mjs` 32 PASS / 0 FAIL →
PASS; `bun run smoke.mjs` 46 PASS / 0 FAIL → PASS; tsgo exit 0, 0 error
lines → PASS. No new red anywhere; both GREEN harnesses grew their pins
(27→32, 45→46).

## Commits (branch `migration`, named-path staging, no `git add -A`)
- `4c0e607` port gauge session-row read to session_v2 (table-aware…)
- `1b7a202` wire intercept observer model field through the gauge session
  reader (unknown fallback kept)
- final (this handover) — the hash lands in the planner's follow-up bookkeeping
  commit in the MAIN repo, never here.
- Deviation (recorded): the session-activity DONE line + the two `submit`
  inbox entries were written and committed in the MAIN repo
  (`loop/autorun-2026-10-01_03-27/loop_log.md`,
  `agent/agent_feedback.md`, `agent/knowledge/knowledge_inbox.md`) — the
  role-mandated activity/friction channels; NO main-repo code, gate file,
  `TODO.md`, prompt, or NAP was touched.

Working tree at writing time: clean for the tracked files above (this file
becomes the third commit).

## Inbox / findings
- Two `submit` entries fired (2026-10-22_00-08): FEEDBACK — `block_transfer`
  is sandboxed to the launching CWD repo + Windows temp, so it has NO
  access to a sibling clone repo (`edit`/`write` do reach it); a truncated
  path typo escalated to maintainer approval instead of a plain sandbox
  error. KNOWLEDGE — the same fact + the "verify path completeness before
  a write call" rule.
- No `TODO.md`/`todo_inbox.md` entries: no discrepancy inside unit scope.
  The probe/v1-smoke red-at-baseline is PRE-EXISTING and already tracked
  by the v2 migration's later units (the tools estate), not by this one.

## Deliberately omitted
- The TOKEN-MASS legs (`SQL_FINISHED_STEP`, `SQL_FINISHED_STEP_SESSION`,
  the spawn-sqlite3 MARKER SQL `GAUGE_SQL_MARKER`/`sqlMarkerForSession`) —
  unit 2 by spec; the reader signature was NOT pre-shaped for them.
  Consequence: on a v2 session the gauge still reads `CTX=notAvailable`
  (no-total) until unit 2 lands — the sid/model/limit halves are the fix.
- The spawn-sqlite3 session-row leg stays v1 SQL (the last-resort backend);
  the in-process backends cover v2 (both share `readApiDb`).
- Re-pinning the probe (it cannot run at HEAD); adding v1-table pins to the
  bun smoke (gauge_core covers that form).
- The clone `agent/scripts/v2_intercept_acceptance.sh` re-run — the
  MAINTAINER's (agents never run the clone host against the backend slot).

## Pending live acceptance (name it)
Expect intercept.log lines whose model slot carries the real model id
(`qwen3.8-flash-next-coder-iq1_m`) instead of `unknown` — the maintainer's
re-run of `agent/scripts/v2_intercept_acceptance.sh` decides it.

## Blockers
None for unit 1. Unit 2 (token mass) and the later tools-estate units stay
the planner's call.

## Lesson (reusable)
In this clone, the "standard gate" (probe + the 11 node smokes) is RED at
HEAD because the v1 plugin sources moved to `.opencode/plugin-unported-v1/`
— the verifiable v2 harness is `gauge_core.smoke.mjs` (node) + `bun run
smoke.mjs` + the tsgo typecheck; diff untouched smokes before/after to
prove no new red.
