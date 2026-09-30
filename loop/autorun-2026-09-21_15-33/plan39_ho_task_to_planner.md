# handover_task_to_planner.md — plan39: context_trim tool (WORKER-39, FINAL)

Session ses_f10a06089ffeVKusSGiE20Hjw9 (worker_Q3S_slow). Branch `opencode_test`,
never pushed. Task DONE — all acceptance criteria met.

## What changed (the code commits)
- **`2d4480e`** — `.opencode/tools/context_trim.ts` (new) +
  `.opencode/plugin/tests/context_trim.smoke.mjs` (new):
  - core exports `reportWindow(dbPath, sessionID, target?)` (read-only
    window map + dry-run — no write statement is ever executed) +
    `tailSet(dbPath, sessionID, targetID)` (the validated `tail_start_id`
    rewrite: session exists → last COMPLETED compaction exists (marker +
    summary child, the strict summary+finish+no-error detection) → target
    exists → target strictly before the compaction row in (time_created, id)
    order → retained-tail count >= 6 (the floor) — each fail-closed with its
    exact reason; success = `tail= <old> -> <new> keep=<count>`);
  - single-field JSON edit on the ONE compaction part row (the host's own
    lever, compaction.ts:461-466), single transaction, WAL;
  - backend (new for this repo — first DB-writable tool): `bun:sqlite` then
    `node:sqlite` (in-process, writable, `busy_timeout 2500`, one busy
    retry → fail-closed db-error; no spawn fallback for writes);
  - the `tool()` wrapper (mode/session/target args, the gauge's
    DEFAULT_DB_PATH); NO registration (maintainer's domain);
  - window math mirrors message-v2.ts:525-576 exactly (ascending
    (time_created, id); host-loose summary detection for the guard window,
    the strict detection for the completed-compaction validation).
- **`48aab9f`** — `.opencode/plugin/probes/handover_probe.mjs`: new **S33**
  section, checks 346-351 (the header's 3 machine-updated places done:
  EXTENDED clause, section index, tally 346 → 352).

## Measured verification (this session, post-commit)
- **Probe**: 352 checks = **341 PASS + 11 known #113 env-fails (136-146**,
  python numword — the agent repo has no `.venv`; VENV_PY points at
  REPO_ROOT/.venv) — baseline preserved exactly (335/346 + 11 env before
  S33). Exit code 1 solely from those 11.
- **All 11 smokes green** (exit 0): auto_resume 146/146, block_transfer
  131/131, bt-sandbox 64/64, compact_memory 80/80, context_recovery 17/17,
  ctx_gauge 3/3, gauge_core ALL, intercept_observer 78/78, loop_log 69/69,
  submit 31/31, **context_trim 20/20** (the new one: byte-exact
  report/dry-run/tail pins, the 5 rejection reasons + session-not-found,
  no-marker report, db-error shapes, FK cascade verified).
- NOTE: the compact_memory (78→80) and submit (23→31) counts grew in the
  intervening commit d2cad69 (pre-dates this task) — the handover
  IN-PROGRESS version recorded the pre-d2cad69 numbers; current counts are
  the green baseline.
- FST repo (measured pre-compaction this session): the #113 venv is FIXED
  (rebuilt 3.12.9) — ruff `All checks passed!` (F=0), pytest 459 passed,
  1 warning.
- Live DB: NEVER written (smoke + probe use sandbox/fixture DBs only; the
  live-DB work was bounded read-only inspection — `.schema` + LIMIT-5
  selects via the maintainer's sqlite3.exe `-readonly`; the `?mode=ro` URI
  form FAILS on that build — `-readonly` works).

## TODO entry
- `TODO.md` #120 status → LANDED (this final commit) with the code commits'
  hashes (2d4480e, 48aab9f) — this commit's own hash rides the planner's
  follow-up bookkeeping commit.

## Registration snippet (maintainer's opencode.jsonc)
- **name:** `context_trim` (the host names the tool by FILENAME —
  `.opencode/tools/context_trim.ts`; no `name` field in the tool file)
- **description:** the tool's own description text (verbatim in the file's
  `tool({...})` call — starts "Reports and trims a session's live model
  context over the opencode DB…").
- Wiring note: no other custom tool has an explicit opencode.jsonc entry
  (grep: none; the permission map is `"*": true`), i.e. the host auto-
  detects `.opencode/tools/*.ts` — the snippet is for an explicit entry if
  the maintainer wants one. The tool only activates on host restart.

## Live-acceptance note (planner's next cycle)
- `report` mode is READ-ONLY → safe to live-accept on a real session after
  the maintainer's restart + registration (the first thing to try).
- `tail` mode writes the live DB → a dedicated throwaway session, the
  maintainer's call (never a working session blind).

## Deliberately not done
- `opencode.jsonc` / `AGENTS.md` / `agent/prompts/**` / `maintainer/**`:
  untouched (DO-NOT-TOUCH respected). Unit 2 (the plugin-side
  post-compaction tail-set) = the proposal's follow-up, NOT in this task.
- No new TODO entry — no doc/code discrepancy found (the research doc's
  line refs matched the 1.18.32 dev tree exactly).

## Lessons
- Bash `python -c "…"` with embedded quotes/`${…}` mangles under this host's
  bash — use the edit tool (or a written .py file) for multi-line string
  replacements in JS/TS files.
