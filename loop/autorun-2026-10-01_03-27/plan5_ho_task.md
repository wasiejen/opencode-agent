# Task spec — TODO #120 Unit 2: the plugin-side post-compaction tail-set (zero fork)

## Goal
After any host compaction lands, the auto-resume plugin rewrites the fresh
compaction part's `tail_start_id` to the agent-requested keepMessages boundary
(exact count-based retention), replacing the host's default token-budget tail.
The proposal (`proposals/implemented/2026-09-28_context-trim-tool.md` §Build
scope, Unit 2) is approved ("1. + 2. are approved" + round-2 "go :-)").
Pre-approved class (agent-usage, no observable change to the maintainer's app).

## Verified facts (measured 2026-10-01)
- **COMPACT line format** (the writer `compaction_core.ts` L375-380, the #99
  form): `<stamp> [model] COMPACT <sid> keep=<N>m tok=<tok> <source>
  [ emergency][ ovr][( preField)]`. The keep count is **`keep=<N>m`** (always
  present). NOTE: `auto_resume.ts` L1462-1464 comment is STALE (it says
  `messages=<n>`); fix it in this PR.
- **The tick** (`auto_resume.ts` L1624-1652): `checkSpawnTrigger` →
  `tailCompactRearm` → `limitStopCheck` → routing loop, each in its own
  try/catch (the timer must never reject). `tailCompactRearm` (L1481-1523)
  reads the NEW ctx.log chunk from the `ctxLogOffset` cursor and iterates the
  NEW `COMPACT` lines (currently re-arms watched sids only). `log()` (L403)
  appends to `logPath`. `COMPACT_SID_RE` at L1465.
- **The tool already has the write path** (`context_trim.ts`): `FLOOR = 6`
  (L102), `computeWindow` (L383-431 — compactionIndex, strictSummaryIndex,
  retained), `tailSet` (L586-668 — the 5 fail-closed validations + the
  json_set single-transaction write at L630-642), `DEFAULT_DB_PATH` imported
  from `../plugin/scripts/gauge.mjs` (L100), backend chain bun:sqlite →
  node:sqlite → spawn-sqlite3 (async execFile, 16 MB, `\x01` separator).
- **The lever** is the host's own (compaction.ts:461-466); the context
  re-derives from the DB at the top of every loop step → zero restart.

## What to build
1. **`context_trim.ts`** — ADD one exported function, do NOT change the
   existing `tailSet`/`reportWindow` behavior (reuse, don't duplicate):
   - `tailSetKeep(dbPath, sessionID, keep): Promise<string>` — compute the
     boundary target = the `keep`-th message STRICTLY before the compaction
     user row (targetIndex = compactionIndex - keep), then reuse the exact
     `tailSet` validation (session exists; completed compaction exists —
     marker + strict summary child; target exists; target before compaction;
     retained = keep >= FLOOR) + the same json_set single-transaction write.
     Return `tail-set= <old> -> <new> keep=<N>` or a rejection reason
     (`tail-set= rejected: <reason>`). Reject `keep <= 0` and `targetIndex < 0`
     (keep exceeds the pre-compaction message count).
2. **`auto_resume.ts`** — ADD an additive tail-set leg (do NOT alter the
   spawn/nudge/limit-stop/routing legs):
   - Extend the NEW-COMPACT-line reading (the `tailCompactRearm` ctx.log
     cursor) so it also yields the parsed `keep=<N>m` for **every** NEW
     COMPACT line (not only watched sids).
   - Add `tailSetLeg(...)` — awaited in `tick()` right after
     `tailCompactRearm()`, before `limitStopCheck()`, in its own try/catch —
     that for each (sid, keep) calls `tailSetKeep(DEFAULT_DB_PATH, sid, keep)`
     and `log(`tail-set= sid=${sid} ${result}`)`.
   - Fix the stale line-format comment (L1462-1464).
3. **Smoke + probe** (below).

## Definition of done
- **`context_trim.smoke.mjs`** new section for `tailSetKeep` (fixture DB,
  force `spawn-sqlite3` via `setBackends(["spawn-sqlite3"])`): valid case
  (keep=N → `tail_start_id` rewritten to the N-th-pre-compaction message id,
  return `tail-set= old -> new keep=N`) + rejections (keep below floor 6;
  keep > available pre-compaction messages; no completed compaction; session
  not found). Baseline 26 → 26+N.
- **`auto_resume.smoke.mjs`** new section: write a NEW ctx.log COMPACT line
  (`keep=<N>m`) + a fixture DB carrying a completed compaction part; run the
  tick; assert `tail_start_id` was rewritten to the keep boundary AND a
  `tail-set=` log line landed. Assert the leg does NOT fire for a COMPACT line
  with no `keep=` field or for non-COMPACT lines. Baseline 152 → 152+N.
- **`handover_probe.mjs`** new section S34 pinning the tail-set (2-3 checks).
  Baseline 352 → 352+N.
- **Standard gate green:** probe 352 → 352+N (the 11 #113 env-fails
  unchanged); all 11 smokes green (context_trim 26 → 26+N, auto_resume 152 →
  152+N, compact_memory 89/89, the rest at baseline); ruff F=0; pytest per
  #113 (blocked). The live opencode.db is NEVER written by the smokes (fixture
  only).
- Checkpoint commits per verified unit; **TODO #120 status → LANDED (Unit 2)**
  + this handover ride the FINAL commit (the code commits' hashes, never
  theirs). Live acceptance = the next real compaction shows the `tail-set=`
  line in auto_resume.log (natural occurrence) — note it in the TODO status,
  it is not a DoD gate.

## DO-NOT-touch
- The restart/spawn path, the nudge ladder (`onToolAfterNudge`),
  `limitStopCheck`, the routing loop — the tail-set is a NEW additive leg.
- The existing `tailSet`/`reportWindow` write path in `context_trim.ts`
  (reuse it; only ADD `tailSetKeep`).
- `compaction_core.ts` (the COMPACT-line writer) — the plugin reads the line,
  it does not change it.
- The live opencode.db (smokes = fixture only); the `keep=<N>` semantics (the
  agent-requested keep from the line — not a new config).

## Worker
`worker_Q3S_slow` (same model as the planner — the cache rule; roster
`opencode.jsonc` L386-389). Branch: stay on the current checkout
(`opencode_test`).
