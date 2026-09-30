# Worker handover — TODO #126: context_trim spawn-sqlite3 backend fallback

Worker: worker_Q3S (llama-swap/Qwen3.8-27B-Q3S-170K), session ses_f0d49cb6bffeNMhGpmF7O3DyjU.
2026-09-30. Task spec: `agent/handover/handover_task.md` (the committed #126 spec).
State: DONE — code + smoke committed, standard gate green, live re-acceptance pending.

## What changed

`.opencode/tools/context_trim.ts` (the only code file touched; gauge.mjs read-only,
untouched):
- `openDb` is now a CHAIN tried in order, first success wins: `bun:sqlite` →
  `node:sqlite` → **`spawn-sqlite3`** (the new last-resort backend). Every
  backend failure is collected; the all-fail end is the SAME fail-closed
  db-error as before (now naming all three: `no writable sqlite backend
  (bun:sqlite …; node:sqlite …; spawn-sqlite3 …)`).
- The spawn backend = a thin Ctx-shaped adapter (`openSpawnDb`) over
  `execFileSync` on the gauge's `DEFAULT_EXE_PATH` (import extended from
  gauge.mjs — the established pattern): ARGS ARRAY, no shell (the plan45
  win32 gotcha), hard 2500 ms KILL per call, 1 MB maxBuffer.
  - `report` reads: ONE CLI invocation each on a READ-ONLY URI
    (`file:<path>?mode=ro` — no journal write while the host writes).
  - `tail` write: the BEGIN..COMMIT batch is BUFFERED and flushed as ONE
    read-write CLI invocation carrying `PRAGMA busy_timeout = 2500; BEGIN;
    UPDATE; COMMIT;` (the single-transaction guarantee survives the process
    boundary; ROLLBACK discards the buffer — nothing was executed yet).
  - Column separator forced to TAB (`-separator '\t'`): the CLI default is
    `|`, and pipe can occur inside the JSON data columns (a raw tab never
    does — JSON.stringify escapes control bytes).
  - `?` bindings are inlined with single-quote escaping (the CLI takes ONE
    SQL text); rows parse back to named objects (the in-process shape).
- `setBackends(list?)` test hook (the gauge core's `setBackends` pattern) +
  exported `TRIM_BACKENDS` — the smoke forces `["spawn-sqlite3"]`; omit the
  arg to restore the default. The production host never calls it.
- The report/tail logic (loadSession / computeWindow / validation / the
  single-transaction write) is UNCHANGED — the adapter isolates the change.
- Header block L72-79 rewritten (the old "NO spawn fallback for writes"
  design note is reversed for the spawn case; the fail-closed end is
  documented as staying) + one sentence in the tool description naming the
  backend chain.

`.opencode/plugin/tests/context_trim.smoke.mjs`:
- The existing 20 pins stay in place and green (the fixture discipline
  untouched — the smoke never points at the live DB).
- NEW section 4b (5 pins) on a SECOND fixture session `ses_ct_spx` (its own
  quartet — the in-process pins' `ses_ct_fix` state is untouched), chain
  FORCED to `["spawn-sqlite3"]` via the hook: (a) spawn report byte-exact
  against BOTH the in-process report and the expected text; (b) spawn tail
  rewrite return line byte-exact + the part row reads back the new
  `tail_start_id` + the post-rewrite spawn report shows retained=6 with the
  new tail head row; (c) a validation rejection fires via the spawn path
  (target at the compaction row → `target-not-before-compaction`).
  Total now 25/25.

## Measured verification (standard gate, 2026-09-30, this checkout)

- probe: `PROBE handover: 352/352 PASS` (rc 0, 0 FAIL lines)
- auto_resume 147/147, block_transfer 131/131 + sandbox 64/64,
  compact_memory 82/82, context_recovery 17/17, context_trim 25/25,
  ctx_gauge 3/3, gauge_core ALL PASS, intercept_observer 78/78,
  loop_log 69/69, submit 31/31 — all rc 0.

All spec DoD numbers match (the context_trim new total = 25).

## Commits

- `a43311a` — the code + smoke unit (checkpoint commit, code only).
- THIS commit — TODO #126 status + this handover + the loop-log START/DONE
  + the friction/knowledge/memory inboxes (the close-down commit).

## TODO entries

- #126 status updated to LANDED (live re-acceptance still open — see below).
  No new TODO entries (nothing out-of-scope found).

## Live re-acceptance (DoD item 5 — the planner's next session)

The live process lags HEAD — the build lands with the maintainer's NEXT
restart. After it:
1. the planner runs `context_trim report` on a REAL session (read-only,
   first) — pre-restart it returned `db-error: no in-process writable
   sqlite backend`; with this build the chain must fall through to
   `spawn-sqlite3` (the gauge reads already prove that backend is live).
2. `tail`'s live test stays the maintainer's throwaway-session call.
3. context_trim Unit 2 stays HELD on this live re-acceptance (per the NAP).

## Deliberately not done

- No edits to gauge.mjs (reference only), the report/tail validation logic,
  the smoke's live-DB discipline, `maintainer/`, `agent/prompts/**`.
- No probe pins added (the spec scoped the new pins to the smoke; the
  probe's 6 S33 pins stay green as-is — 352/352).
- No live test (restart-gated, above).

Lessons: the sqlite3 CLI's default column separator is `|` (force `-separator
'\t'` for JSON data columns), `PRAGMA busy_timeout` echoes to stdout, and an
uncommitted BEGIN batch rolls back silently at process exit with exit 0 —
all three captured in the knowledge inbox; the one reusable gotcha
(a buffered CLI flush must include the COMMIT statement) in my memory inbox.

Final context gauge (verbatim):
`SESSION=ses_f0d49cb6bffeNMhGpmF7O3DyjU CTX=101449 (59%) REM=68551 | 5 compactions left`
