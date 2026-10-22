# Task spec — TODO #137 unit 4c: the pre-compaction dump twin for v2 (clone `migration`) — launch AFTER 4b verifies

## Goal
Give the v2 clone its pre-compaction corpus dump: a v2-table-shaped dump script
+ the dump leg in the clone's v2 `compact_memory` plugin (replacing the 4a
`dump-skipped` placeholder), so the corpus stays complete for compacted sessions
as in v1 (TODO #152 semantics).

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (HEAD is `b5ccc0e` — the post-4b state this spec was written against; stay on
  the current checkout).
- Assigned worker: `worker_FlashNext`.
- TODO entry: `TODO.md` #137 (main repo). Design of record (READ-ONLY): main
  `.opencode/plugin/compact_memory.ts` lines ~100-170 (the dump hook: naming,
  log lines, retry) + main `agent/scripts/db/dump_session.cjs` (CLI + output).

## Definition of done (measurable)
1. NEW `agent/scripts/db/dump_session_v2.mjs` in the CLONE (extension open —
   `.mjs` if it reuses ESM helpers) with the SAME CLI as v1:
   `<sessionID>` (lossless md) / `--lite` / `--json` / `--out <relpath>` (exit 2
   on unsafe relpath) / `--all [--slim|--full]`; output under the CLONE's
   `archive/sessions/` (the script's own `__dirname`-relative OUT_DIR); DB
   `OPENCODE_DB` env override; opened READ-ONLY, NEVER written; reads the v2
   tables via the SAME backend-chain pattern as the clone `context_trim_v2.ts`
   (`bun:sqlite` → `node:sqlite` → `spawn-sqlite3` last resort, the clone
   `.opencode/plugin/tools/sqlite3.exe`, with a `setBackends`-style test hook).
   Message content comes from the v2 `message` + `session_message` rows (JSON
   `data` with `tokens{input,output,reasoning,cache.read/write}` and
   `time.completed`) — NO v1 `part`-table reads (they do not exist in v2).
2. The dump leg in the clone `.opencode/plugins/compact_memory.ts`: replaces
   the 4a `dump-skipped` placeholder — fires BEFORE the dispatch, porting the
   v1 hook contract byte-compatible: `preCompactionDumpName`
   (`compaction_dumps/<sid>_c<count>.<md|json>`, stamped fallback
   `_<YYYYMMDDTHHmmss>` on exact-exists), `DUMP-OK <sid> <relFile> ms=<ms>` /
   `DUMP-FAIL` / one `DUMP-RETRY=1` (#78) to `.opencode/temp/ctx.log`,
   best-effort (never throws, never blocks), spawn of the NEW script
   `<sid> --out <corpus-rel>`; `dump-skipped` removed from the response text.
3. SMOKE — extend the clone `plugins/tests/compact_memory.smoke.mjs` (41 pins)
   with dump pins: `preCompactionDumpName` byte-exact (normal + stamped + json
   forms), the stub-spawn success lane (a sandbox stub script like the MAIN
   probe S13 preamble / the smoke's FAKE_DUMP), the DUMP-FAIL non-blocking lane,
   the RETRY-once lane. Final line stays `SMOKE: ALL PASS` (count grows).
4. GATE — clone probe `360/360` UNCHANGED unless pins legitimately change
   (state any repin in the summary); ALL v2 plugin smokes ALL PASS; ALL node
   smokes ALL PASS (the teardown-EBUSY flake on gauge_core/context_trim/auto_resume
   is PRE-EXISTING — zero-FAIL teardown crash only, do not chase in-unit);
   tsgo zero errors; clone tree clean; MAIN repo untouched except the summary
   write.
5. Checkpoint commits in the CLONE (code only; one per verified unit); worker
   summary to the MAIN working tree `agent/handover/handover_task_to_planner.md`.

## Verified facts (measured at spec-authoring, 2026-10-22, clone @ b5ccc0e)
- The v1 dump hook lives in MAIN `.opencode/plugin/compact_memory.ts` ~lines
  100-170: naming + `dumpStamp()` (caller-computed) + `DUMP_ARCHIVE_REL =
  "archive/sessions"` + `preCompactionDumpName` exported for byte-exact probe
  pins + `DUMP-OK`/`DUMP-RETRY=1`/DUMP-FAIL line shapes (#107 full repo-relative
  path in the line; #78 the ONE retry + `ms=` prefix). The v1 plugin fires it
  before ANY dispatch (TODO #152, approved 2026-09-15).
- MAIN probe pins the v1 dump lane (grep `preCompactionDumpName|compaction_dumps`
  in `.opencode/plugin/probes/handover_probe.mjs`: ~lines 458-479 header + checks
  101/102/345; the S13 preamble carries a stub `dump_session.cjs`). The CLONE
  probe has the same lane (29 grep hits) and is expected UNCHANGED (the v1
  tool-shape pins read the inert v1 dirs, not the clone plugin).
- The clone v2 plugin's 4a deviation: comment lines ~69-70 + `buildLine("dump"…
  "dump-skipped")` at ~420 + response text at ~478 — the three sites to replace.
- Backend chain: the clone `context_trim_v2.ts` carries `TRIM_BACKENDS`-style
  chain + test hook (~lines 115-166 of MAIN `.opencode/tools/context_trim.ts`
  is the v1 original); clone `.opencode/plugin/tools/sqlite3.exe` EXISTS.
  The live host has NO in-process sqlite (bun:/node:sqlite both fail there) —
  the spawn last resort is load-bearing (#118/#126).
- v2 DB path = `DEFAULT_DB_PATH` = `~/.local/share/opencode/opencode.db`
  (clone gauge core line 157); `OPENCODE_DB` env override is the script
  convention (MAIN `agent/scripts/db/dump_session.cjs` line ~45).
- v2 tables (the clone gauge core's paired SQL, lines ~545-560): `session`
  (id, model, time_updated), `message` (JOIN on session_id), `session_message`
  (`data` JSON: `tokens{…}`, `time.completed`); compaction rows carry REQUEST
  usage, NOT the context size (gauge core comment ~542). NO `part` table.

## Suggested scope (non-exhaustive)
- NEW: clone `agent/scripts/db/dump_session_v2.mjs` (+ a smoke for it is
  optional IF the plugin-smoke stub lane pins the spawn contract; a direct
  fixture-DB smoke is ALSO fine — pick one, pin coverage is the DoD).
- EDIT: clone `plugins/compact_memory.ts` (the three dump-skipped sites),
  clone `plugins/tests/compact_memory.smoke.mjs`, clone tests README section.
- REFERENCE READ-ONLY: MAIN `.opencode/plugin/compact_memory.ts` 100-170,
  MAIN `agent/scripts/db/dump_session.cjs` (header 1-45 + the v1 md/json row
  shapes via grep, NOT the whole file), clone `plugins/context_trim_v2.ts`
  (chain + hook patterns).

## Approval boundary
Pre-approved (agent-facing corpus bookkeeping). DO NOT touch: MAIN repo (except
the summary); `maintainer/**`; clone `.opencode/plugin-unported-v1/**`,
`plugins/intercept_observer.ts`, `plugins/context_family.ts`,
`plugins/context_trim_v2.ts` (4b's file), `opencode.jsonc`; the live DB (READ-
ONLY opens only); NEVER any request at the backend inference server.

## Procedure (guidance)
1. Read the v1 hook block (MAIN plugin lines 100-170) + the 4a clone plugin's
   three dump-skipped sites.
2. Write the dump twin (v2-table reads, chain, CLI parity), exercise it
   against a FIXTURE DB (scratchpad), not the live DB.
3. Wire the plugin leg + smoke pins; full clone gate; checkpoint commits;
   summary to MAIN.

## Deliberately out of scope
- Switching the corpus-refresh cadence (Standing's `dump_session.cjs --all --slim`
  for the MAIN host) to the twin — the MAIN corpus stays on the v1 script.
- The MAIN-side probe v1 dump port; live v2 acceptance (maintainer domain).
