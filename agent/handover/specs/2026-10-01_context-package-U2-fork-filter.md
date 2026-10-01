# Task spec — U2: the per-message context filter (FORK, non-destructive)

Design source (read first): `proposals/approved/2026-10-01_context-budget-package.md` §U2.
Supporting research: `agent/research/2026-09-28_context-erase-tail-trim.md` (§3 guard, §5 safe zone — read those sections only).

## Goal
A non-destructive per-message context filter: a JSON field on the message
row's `data` blob marks a message excluded from assembly; the FORK's
assembly skips flagged rows; a live kill switch disables the whole filter
without rebuild/exe-swap; the agent toggles rows via `context_trim filter`;
a build/swap bat for the maintainer.

## Verified facts (measured 2026-10-01 by the planner)
- Fork repo: `C:\Users\Wasiejen\Repos\opencode` (a separate repo — fork code
  changes live there; the agent repo gets the plugin surface + bat only).
- Fork assembly: `packages/opencode/src/message-v2.ts` — marker quartet
  detection ~L549-573 (findLastIndex of the compaction part; the guard at
  L568: `tailIndex >= 0 && tailIndex < compactionIndex && summaryIndex >
  compactionIndex`; the summary found by `parentID === compaction.info.id`
  ~L564). Read THAT section (bounded), not the whole file.
- Message/part metadata = JSON `data` blobs (`sql.ts:77,92`); the host's
  `updatePart` spreads (`{...part}`) → unknown extra fields survive
  rewrites (pin this).
- Fork build: `bun run build` in `packages/opencode` (`"build": "bun run
  script/build.ts"` — verified in its package.json). Build-artifact path +
  the installed-exe location: one bounded check each (script/build.ts
  output + the current install site) — do not re-derive the install layout.
- Kill switch: sentinel file `.opencode/temp/filter_off` — one `existsSync`
  in the fork assembly BEFORE the filter read; present → ignore all filter
  fields (mainline behavior); the plugin report shows the switch state.
- Plugin surface: `.opencode/tools/context_trim.ts` gains a `filter` mode:
  toggle on/off by the U1 row numbers, per finished turn or per call-result
  pair; FAIL-CLOSED validation mirroring the plan36 safe zone — never the
  current last user turn, never the marker quartet rows (rejections in the
  existing plain-line style). The fork DOUBLE-GUARDS (defense in depth): a
  protected row with the flag is never skipped (fail-open).
- Session dump: on every filter change, ONE current dump file per session
  (overwrite — no `_c<N>` chain; the DB is not destructively changed).
  Locate the existing dump machinery with one bounded grep before writing.
- Probe pin 346 asserts the tool's EXACT surface arg-key list
  (`[mode, session, target, keepMessages]` post-U0/U1) — the `filter`
  mode's new surface arg(s) REQUIRE re-pinning 346 (in scope — it is a
  behaviour-pinning test; re-pin it like U0 did, do NOT flag it as a
  constraint instead).

## DoD (measured end states)
- Fork tests (its `bun test` infra): flagged row excluded from assembly +
  guard still passes; toggle-on restores; kill switch present → flag
  ignored; protected rows (quartet/last turn) never skipped even when
  flagged; `updatePart` spread survives the field.
- Agent smoke (fixture DB only): filter on/off pins through the tool
  surface + every rejection fail-closed (no DB write on rejection) +
  report shows filter state + switch state.
- `build_fork.bat` at the agent repo root: bun build in the fork + replace
  the installed exe (NOT run by the worker — the maintainer runs it; the
  unit is not live until his run).
- Standard gate green BEFORE the bat run (probe 355/355 + all 11 smokes —
  the fork is not live yet).
- Live acceptance (planner/maintainer, AFTER the bat run): one watched
  session toggling a row off/on, the kill switch exercised once.
- Checkpoint: one code commit per verified unit (fork unit / plugin unit /
  bat); TODO + handover in the final commit.

## DO-NOT-touch
- the LIVE opencode.db (fixture DBs only); the INSTALLED opencode exe /
  node_modules (only the bat references the install path — never written
  by the worker); the mainline opencode repo (the fork repo is
  `C:\Users\Wasiejen\Repos\opencode`); `maintainer/`; the proposal files.
- New TODO entries: expectation only, never a promise.

## Worker
worker_Q3S_slow (effort L — fork change + build script + two test surfaces).
