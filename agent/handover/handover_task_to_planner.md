# handover_task_to_planner.md — plan39: context_trim tool (WORKER-39, IN PROGRESS)

Session ses_f10a06089ffeVKusSGiE20Hjw9 (worker_Q3S_slow). Branch `opencode_test`,
never pushed. Compacted at the stop line mid-run — this IN-PROGRESS version is
the durable state; the final version replaces it before close-down.

## What changed so far
- **Unit 1 LANDED `2d4480e`** — `.opencode/tools/context_trim.ts` (new):
  - core exports `reportWindow(dbPath, sessionID, target?)` (read-only window
    map + dry-run) + `tailSet(dbPath, sessionID, targetID)` (validated
    tail_start_id rewrite, floor 6, single transaction, one busy retry,
    fail-closed); the `tool()` wrapper (mode/session/target args) uses the
    gauge's DEFAULT_DB_PATH; NO registration (the handover carries the
    snippet).
  - Backend (new for this repo — first DB-writable tool): `bun:sqlite` then
    `node:sqlite`, writable open + `busy_timeout 2500`; report performs no
    writes. No spawn fallback for writes (fail-closed).
  - Window math mirrors message-v2.ts:525-576 exactly (ascending
    (time_created, id); host-loose summary detection for the guard window,
    the strict summary+finish+no-error detection for the completed-compaction
    validation).
  - `.opencode/plugin/tests/context_trim.smoke.mjs` (new): fixture DB built by
    the smoke (live-DB inspected shapes, FK cascade ON), 20/20 ALL PASS —
    byte-exact report/dry-run/tail pins, the 5 rejection reasons, no-marker
    report, db-error shapes, FK cascade verified.
- **Live-DB inspection (read-only, bounded)**: `.schema session/message/part`,
  compaction part shape `{"type":"compaction","auto":false,"tail_start_id":…}`,
  summary row shape, read/bash/webfetch part shapes — all from
  `~/.local/share/opencode/opencode.db` via the maintainer's sqlite3.exe
  `-readonly` (the `?mode=ro` URI form FAILS on that build — `-readonly` works).

## Remaining work (on resume)
1. **Probe section S33** in `.opencode/plugin/probes/handover_probe.mjs`:
   - new section AFTER the S32 block (which ends ~line 8012, before the
     `// ------------------------------------------------------------------ summary` line ~8014); checks 346-351
     (String(n33) pattern, `n33` starts at 346):
     - 346 tool surface (direct type-stripped import, the S12/S15 load pattern:
       description + args mode/session/target schemas + async execute + NO name
       field + named exports reportWindow/tailSet functions);
     - 347 report (no target) full text BYTE-EXACT on the probe's own fixture
       (build it in SANDBOX with DatabaseSync — the smoke's fixture layout is
       the reference: 18 messages, quartet at msg_13/14, tail_start msg_07,
       tool parts INSIDE the retained tail on msg_08 read / msg_10 read+bash /
       msg_12 webfetch; the exact expected bytes are in the smoke's
       EXPECT_REPORT);
     - 348 dry-run line exact (target msg_06: `remove=0 (0 tokens) keep=7
       (38703 tokens) verdict=ok` — recompute if the probe fixture differs);
     - 349 tail rewrite lands (return byte-exact `tail= msg_07 -> msg_06
       keep=7` + part JSON byte-exact on re-read);
     - 350 rejections (no-completed-compaction / target-not-found /
       target-not-before-compaction / retained-tail-below-floor, each exact);
     - 351 no-marker report header line exact (`marker=none …
       window=full-history`).
   - HEADER updates (3 places, all machine-updated):
     - the top EXTENDED-history comment: add the plan39/S33 clause (insert
       between the #92 2026-09-27 clause ending `…the NEW check 345 (the json
       naming byte-exact)):` (~line 84) and the `// the pre-rebuild probe` line);
     - the section-index block: add the `S33 … (6)` entry after the S32 block
       (which ends ~line 968);
     - the EXPECTED OUTPUT tally (~line 971): add `S33=6` and 346 → 352
       (`"PROBE handover: 352/352 PASS"`).
2. **Gate**: `node .opencode/plugin/probes/handover_probe.mjs` → expect 352
   checks = 341 PASS + the 11 known #113 env-fails 136-146 (baseline measured
   this session: 335/346 + 11 env); all other smokes unchanged (run the full
   suite; baselines: auto_resume 146, block_transfer 131, bt-sandbox 64,
   compact_memory 78, intercept_observer 78, loop_log 69, context_recovery 17,
   ctx_gauge 3, gauge_core ALL, submit 23, context_trim 20); FST gate measured
   this session: ruff F=0 ("All checks passed!"), pytest 459 passed 1 warning
   (the #113 venv is FIXED now — report that; the probe's 11 fails remain: the
   agent repo has no `.venv` of its own — VENV_PY = REPO_ROOT/.venv/…).
3. **Final commit**: TODO #120 status → LANDED + code-commit hashes (2d4480e +
   the probe commit's hash — never my own final hash) + this file's FINAL
   version (the registration snippet + measured gate numbers + the
   live-acceptance note: report read-only → planner live-accepts after
   maintainer's restart + registration; tail = throwaway session, maintainer's
   call) + friction submit + loop_log DONE line (the close-down rides a commit).

## Registration snippet (for the maintainer's opencode.jsonc — in the FINAL
handover verbatim)
name `context_trim` → file `.opencode/tools/context_trim.ts`, description = the
tool's own description text (see the file).

## Deliberately not done / flags
- `opencode.jsonc` / AGENTS.md / agent/prompts / maintainer/** / live-DB
  writes: untouched (DO-NOT-TOUCH respected).
- No new TODO entry expected; doc/code discrepancies: none found so far
  (the research doc's line refs matched the 1.18.32 dev tree exactly).
