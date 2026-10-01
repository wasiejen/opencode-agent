# Worker handover — TODO #120 Unit 2: the plugin-side post-compaction tail-set (zero fork)

Worker: worker-Q3S-slow (ses_f0a1d6a5affeFgN9YsdlKV7LNr), branch `opencode_test` (no
branch changes, no push). Spec: `agent/handover/handover_task.md` (read as the
single source of truth). Status: **LANDED (Unit 2)** — the code is on disk, the
standard gate is GREEN at the measured numbers below.

## What changed (3 code commits)

1. **`ad7db12` — `.opencode/tools/context_trim.ts` + its smoke (26 → 29)**
   - ONE new export, `tailSetKeep(dbPath, sessionID, keep)`: computes the
     boundary target = the `keep`-th message STRICTLY before the compaction
     user row (`targetIndex = compactionIndex - keep`), then delegates to the
     EXISTING `tailSet` — the same 5 fail-closed validations re-run on
     tailSet's own fresh read + the same json_set single-transaction write
     (reused, not duplicated; the `tail= ` return prefix is remapped to
     `tail-set= `). New rejection tokens: `keep-invalid` (keep <= 0 /
     non-integer) and `keep-exceeds-history` (targetIndex < 0). `tailSet` /
     `reportWindow` behavior unchanged (no existing pin touched).
   - context_trim.smoke.mjs: new section 9 on a dedicated `ses_ct_k` fixture
     (spawn chain FORCED via `setBackends(["spawn-sqlite3"])`): valid case
     (keep=8 → `tail-set= msg_k05 -> msg_k03 keep=8` + part row byte-exact +
     retained=8 re-report) + the 5 rejections (floor 5 / keep-exceeds-history
     50 / no-completed-compaction / session-not-found / keep-invalid 0).
2. **`f625a65` — `.opencode/plugin/auto_resume.ts` + its smoke (152 → 155)**
   - Additive tick leg `tailSetLeg()`, awaited in `tick()` AFTER
     `tailCompactRearm()` and BEFORE `limitStopCheck()`, in its own
     try/catch (the timer never rejects). It drains `tailSetQueue` — populated
     by `tailCompactRearm` from EVERY NEW ctx.log COMPACT line carrying
     `keep=<N>m` (watched or NOT; a line without the field queues nothing, so
     the leg never fires for it; non-COMPACT lines never match) — calling
     `tailSetKeep(tailSetDbPath, sid, keep)` + logging `tail-set= sid=<sid>
     <result>` (rejections logged the same way — fail-closed, no write).
   - Imports `tailSetKeep` from `../tools/context_trim.ts` (the established
     relative-TS-import pattern, cf. intercept_observer.ts) + `DEFAULT_DB_PATH`
     from `./scripts/gauge.mjs`.
   - The stale COMPACT-line-format comment (former L1462-1464, said
     `messages=<n>`) fixed to the #99 form (`keep=<N>m tok=<tok> <source>
     [ emergency][ ovr][( preField)]` — the measured writer,
     compaction_core.ts appendCompactLine).
   - Factory `dbPath` option (test-only lever, the `tickMs` pattern): default
     = `DEFAULT_DB_PATH` (the live opencode DB — the live host never passes
     the option); the smoke passes its FIXTURE db, so the live DB is NEVER
     written by the smokes.
   - DO-NOT-touch honored: the restart/spawn path, the nudge ladder
     (`onToolAfterNudge`), `limitStopCheck`, the routing loop, and the
     existing `tailSet`/`reportWindow` write path are all untouched; no change
     to compaction_core.ts; the `keep=<N>` semantics = the agent-requested
     keep from the line (no new config).
   - auto_resume.smoke.mjs: fixture db (`ses_tail_fix` quartet) built before
     the first factory call + `dbPath` on that call; new section after #109:
     a NEW ctx.log batch (the keep=7m line for `ses_tail_fix`, a no-keep
     COMPACT line for `ses_tail_nok`, a non-COMPACT gauge line) → pin 1 the
     rewrite landed (`tail-set= msg_t05 -> msg_t04 keep=7` log line, exactly
     one) + pin 2 the part row byte-exact (fixture only) + pin 3 no-fire
     (no line for `ses_tail_nok`, exactly TWO `tail-set=` lines total — the
     second is the #109(2) `ses_ls_compact` keep=5m line failing closed as
     `session-not-found` against the fixture). New sids added to the
     live-log smokeSids guard.
3. **`9bdcfc9` — `.opencode/plugin/probes/handover_probe.mjs` (352 → 354)**
   - New section S34 (checks 352-353) on the probe's own fixture DB (a fresh
     `ses_ct_k2` session so S33's pins and their mutations are untouched):
     352 valid + landed (keep=8 → `tail-set= msg_k25 -> msg_k23 keep=8` +
     part row byte-exact — the exact tailSet write path reused) + 353 the 5
     fail-closed rejections. Header annotation refreshed (the extension
     paragraph + the S34 index entry + the section-sum line, the
     curate-don't-duplicate rule).

## Measured verification (standard gate, all green)

- **Probe: `PROBE handover: 354/354 PASS`** (baseline 352 + 2 S34 checks) —
  FULLY GREEN; the 11 #113 env-fails no longer exist (see discrepancy 1).
- **All 11 smokes green** (plain `node`, exit 0 each):
  auto_resume **155/155** (baseline 152 + 3), context_trim **29/29** (baseline
  26 + 3), compact_memory 89/89, block_transfer 131/131 + sandbox 64/64,
  intercept_observer 78/78, loop_log 69/69, submit 31/31, context_recovery
  17/17, ctx_gauge 3/3, gauge_core ALL PASS.
- **ruff F=0** (FST venv: `All checks passed!`); **pytest** — 466+1w
  (planner-verified 2026-10-01 baseline; this task touches NO FST code, not
  re-run).
- **Live opencode.db**: never written by any smoke — context_trim smoke =
  its own fixture; auto_resume smoke = the `dbPath` fixture lever (the
  `ses_ls_compact` leg attempt against it failed closed as
  session-not-found, no write).
- The live host process must postdate this build for the leg to run (the
  plugin reloads on the maintainer's restart) — registration is already
  live-side (the auto_resume.log live instance is confirmed by the smoke's
  live-log guard); the `context_trim` tool registration in opencode.jsonc
  remains the maintainer's domain (untouched).

## TODO entries

- `TODO.md` #120: new **Status (Unit 2, 2026-10-01)** block — LANDED, the 3
  commit hashes, the measured gate, and the live-acceptance note (NOT a DoD
  gate): the next REAL host compaction shows a `tail-set=` line in
  auto_resume.log (natural occurrence). Entry stays open (U2 live
  acceptance + the tool registration = maintainer domain). No new
  todo_inbox.md entries — nothing found out of scope.

## Discrepancies / notes for the planner

1. **Stale spec parentheticals:** the DoD says "probe 352 → 352+N (the 11
   #113 env-fails unchanged)" and "pytest per #113 (blocked)". #113 was
   CLOSED 2026-09-30 (plan40 VENV_PY re-point) — the MEASURED probe is fully
   green (354/354, no env-fails), and FST pytest is runnable (466+1w
   baseline, planner-verified 2026-10-01). Measured numbers reported
   above; nothing was changed to force either.
2. **`dbPath` factory option (additive, beyond the literal spec wording):**
   the spec's leg text says `tailSetKeep(DEFAULT_DB_PATH, sid, keep)`, but
   the DoD also requires the smoke to drive the leg end-to-end against a
   fixture DB without ever touching the live one — a test-only dbPath lever
   (default = DEFAULT_DB_PATH, live host never passes it) was the minimal
   way to satisfy both, following the file's existing test-lever pattern
   (`tickMs`, `maxLogBytes`). Live behavior = exactly DEFAULT_DB_PATH.
3. **Reuse via delegation:** `tailSetKeep` opens the DB for the boundary
   computation (session/marker/floor-ability guards), then delegates the
   validated write to the EXISTING `tailSet` (its validations re-run on a
   fresh read — fail-closed if a new compaction lands between the two
   reads; its floor check IS the kept count, retained == keep). The `tail= `
   prefix is remapped to `tail-set= ` (one replace — all tailSet returns
   start with `tail= `). No second write path, no SQL duplication.
4. The auto_resume smoke's existing #109(2) COMPACT line
   (`ses_ls_compact keep=5m`) now deterministically produces a fail-closed
   `tail-set= ... rejected: session-not-found` line against the fixture —
   folded into pin 3's exact count (TWO total `tail-set=` lines). Existing
   pins unchanged.

## Deliberately NOT done

- No `turns` mode (not part of this entry's approved scope).
- No opencode.jsonc / registration change (maintainer's domain).
- No change to the live opencode.db, compaction_core.ts, or the
  spawn/nudge/limit-stop/routing legs (DO-NOT-touch honored).
- No live acceptance test (a real host compaction = natural occurrence,
  noted in the TODO status; not a DoD gate).

## Commits (this task)

- `ad7db12` — tailSetKeep core export + context_trim smoke 26 → 29
- `f625a65` — the auto-resume tail-set leg + auto_resume smoke 152 → 155
- `9bdcfc9` — probe S34 (2 checks), 352 → 354

(Final bookkeeping commit carries these hashes + the TODO #120 status +
this handover.)
