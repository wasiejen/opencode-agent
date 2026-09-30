# Task spec — context_trim: the spawn-sqlite3 backend fallback (TODO #126)

Worker: worker_Q3S (170K). Pre-approved class (agent-usage tool fix).
Branch truth: stay on the current checkout.

## Goal

`context_trim` must work in the LIVE host process: add a third, last-resort
backend (`spawn-sqlite3` via `.opencode/plugin/tools/sqlite3.exe`) to
`openDb`, mirroring the gauge core's proven spawn cascade. Keep fail-closed:
when NO backend is available at all, the tool still returns a db-error
(never a blind write).

## Verified facts (measured at spec time, 2026-09-30 — do not re-derive)

- Live: the first live `report` call returned `db-error: no in-process
  writable sqlite backend (bun:sqlite / node:sqlite both unavailable)`. The
  live process IS the current build (`context_trim` is in the live toolset).
- `.opencode/tools/context_trim.ts` L98-129 `openDb`: tries `bun:sqlite`
  then `node:sqlite` only. The header L72-79 documents the deliberate
  "NO spawn fallback for writes" design — this task reverses that for the
  spawn case ONLY (the fail-closed end stays).
- `.opencode/plugin/scripts/gauge.mjs` = the reference implementation (READ
  the bounded regions only, do not copy blindly): `DEFAULT_BACKENDS` (L163,
  order node→bun→spawn), `DEFAULT_EXE_PATH` (L161 — the exe is on the tree,
  verified), the `setBackends` test hook (L187), the spawn-sqlite3 backend
  with its hard-timeout discipline (see the `spawn-sqlite3` + `hard-timeout`
  comment block L42-44), the backend-quirks notes (L12-32: NULL vs
  undefined, busy_timeout, readOnly flags).
- `context_trim.ts` already imports from gauge.mjs (`DEFAULT_DB_PATH,
  BUSY_TIMEOUT_MS`, L86) — extending that import is the established pattern.
- The smoke host = system node v24+ (`node:sqlite` available) → the
  in-process pins never exercise the spawn path unless the backend list is
  FORCED via a test hook.
- win32 gotcha (plan45 lesson): never `execSync` with `^`-bearing args
  (cmd-escape). Use `execFile`/`execFileSync` (no shell) — that is also the
  gauge core's discipline.

## Design (suggestion — the HOW is yours within the DoD)

- Backend 3 in `openDb`: `spawn-sqlite3` via `DEFAULT_EXE_PATH`. The
  returned `Ctx` keeps the `{db, backend}` shape — a thin adapter object
  over `execFile` (query → rows; the single tail transaction → ONE CLI
  invocation carrying `PRAGMA busy_timeout…; BEGIN; …; COMMIT;`) keeps the
  report/tail logic UNCHANGED.
- A backend-list test hook (mirror `gauge.mjs` `setBackends`) so the smoke
  can force `["spawn-sqlite3"]`-only.
- The spawn path stays fail-closed: exe missing / spawn error / timeout →
  the db-error path (no partial write).
- `report` = read-only queries (fine via the CLI); `tail` = the single
  transaction via ONE CLI invocation.

## DO-NOT-touch

- `gauge.mjs` internals (reference only — no edits to it).
- The context_trim query/validation logic (the adapter isolates the change).
- The smoke fixtures' live-DB discipline (the smoke NEVER points at the
  live DB).
- The maintainer's live files; anything under `maintainer/`; `agent/prompts/**`.

## Definition of done

1. Code: `openDb` gains the spawn-sqlite3 fallback + the backend test hook;
   report/tail logic unchanged.
2. Smoke (`context_trim.smoke.mjs`): the existing 20/20 pins stay green +
   2-3 new pins FORCING the spawn-only backend: (a) report on the fixture DB
   returns the same key fields as the in-process path, (b) a tail rewrite
   lands in the fixture DB (tail_start_id changed, readable back), (c) at
   least one validation rejection still fires via the spawn path.
3. Standard gate green: probe 352/352 + all 11 smokes (auto_resume 147/147,
   block_transfer 131/131 + sandbox 64/64, compact_memory 82/82,
   intercept_observer 78/78, loop_log 69/69, submit 31/31,
   context_recovery 17/17, context_trim (new total), ctx_gauge 3/3,
   gauge_core).
4. Checkpoint commits per verified unit (code only); the TODO #126 status +
   the handover summary ride the FINAL commit (per the commit routine).
5. Live re-acceptance = the maintainer's NEXT restart (the live process lags
   HEAD) — name it in the handover: the planner runs `report` on a real
   session (read-only, first); `tail`'s live test stays the maintainer's
   throwaway-session call.

Context discipline: bounded reads only (the named regions + the smoke file,
~350 lines); grep with `| head -30`; no broad research.
