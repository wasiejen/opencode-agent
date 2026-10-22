# Worker summary — TODO #137 unit 4c: the pre-compaction dump twin for v2 (clone `migration`)

Status: COMPLETE. Four clone checkpoints landed; the clone gate is green at HEAD.

## Task and scope

Goal: give the v2 clone its pre-compaction corpus dump — a v2-table-shaped dump
script + the dump leg in the clone's v2 `compact_memory` plugin (replacing the
4a `dump-skipped` placeholder), with the v1 hook contract ported byte-compatible.

Repo: the CLONE `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`.
Started at `b5ccc0e` (post-4b), stayed on the current checkout, no branch switch.
MAIN repo: ONLY `agent/handover/handover_task_to_planner.md` (this file) written
in the working tree — NOT committed, no other MAIN file touched, no `submit`
writes (the spec's MAIN-untouched boundary; the friction + knowledge notes below
are carried HERE for the planner to route).

## Files changed (CLONE, branch `migration`)

- NEW `agent/scripts/db/dump_session_v2.mjs` (498 lines) — the v2 dump twin
  (CLI parity + the read-only backend chain).
- EDIT `.opencode/plugins/compact_memory.ts` — the dump hook section (naming,
  ctx.log lines, `resolveNodeExe`, spawn + one retry) + the dump call in the
  execute + the deviation (d) header rewrite + the tool description tail
  (+212/−… lines).
- EDIT `.opencode/plugins/tests/compact_memory.smoke.mjs` — 41 → 63 pins
  (22 new dump + twin-CLI pins; 4 dump-skipped-era pins repinned).
- EDIT `.opencode/plugins/tests/README.md` — the compact_memory section
  documents units 4a + 4c.

Untouched as required: `maintainer/**`, clone `.opencode/plugin-unported-v1/**`,
clone `plugins/intercept_observer.ts`, `plugins/context_family.ts`,
`plugins/context_trim_v2.ts`, `opencode.jsonc`, the LIVE DB (READ-ONLY opens
only), all MAIN code/docs, clone `archive/**` (the fixture runs wrote dumps into
a sandbox copy, never into the CLONE corpus).

## Behavior changed (v2 semantics)

1. NEW dump script reads the v2 pair `session_v2` (the `model` column is a JSON
   string — unwrapped) + `session_message` (`id, session_id, type, seq,
   time_created, time_updated, data`), ordered `seq ASC, time_created ASC,
   rowid ASC`. NO `part`-table read exists in the file — v2 content lives INLINE
   in the row's `data`: assistant `content[]` items
   `{type:"text"|"reasoning", text}` and `{type:"tool", id, name, executed,
   state{status,input,content|error}, time}`; `user` rows carry `text`+`files`;
   `compaction` rows carry status/reason/summary/recent. `parts=`/the
   `# messages: N parts:` line therefore count CONTENT ITEMS, and the md gains
   ONE extra header line `# source: v2 tables=session_v2+session_message
   backend=<name>`.
2. CLI parity with v1 (flags, mutual exclusions, exit codes):
   `<sid>` full md / `--lite` / `--json` / `--out <relpath>` (unsafe relpath
   → exit 2 with the v1 stderr line) / `--all [--slim|--full]`; no such
   session → exit 1; a DB WITHOUT the v2 pair → exit 2 with a clear
   `no v2 tables (session_v2 + session_message)` line (the v1 pair stays
   `dump_session.cjs`'s job); `db-error:` → exit 1 (never a blind write).
   OUT_DIR = the script's own dir-relative `archive/sessions`.
3. Backend chain (read-only, FIRST SUCCESS wins): `bun:sqlite` (`{readonly:
   true, timeout:2500}`, reads via `db.prepare(sql).all(...)` — bun 1.4.3
   exposes NO `db.all` sugar) → `node:sqlite` (`{readOnly:true}` + `PRAGMA
   busy_timeout`) → `spawn-sqlite3` LAST RESORT (`<clone root>/.opencode/plugin/
   tools/sqlite3.exe`, `execFileAsync` ARGS ARRAY, `-json` rows, a `file:
   …?mode=ro` URI, a hard 2500 ms KILL). Chain steer: `setBackends()` export
   + the CLI-side `DUMP_V2_BACKENDS="…"` env hook (production never sets it).
4. Plugin leg: EVERY allowed dispatch (SELF or CROSS, auto or marker) dumps
   BOTH artifacts BEFORE the `session.compact` call, named
   `compaction_dumps/<sid>_c<PRE-DISPATCH budget count>.{md,json}`; an
   exact-exists md base → the STAMPED fallback `_<YYYYMMDDTHHmmss>` shared by
   both artifacts. The v1 ctx.log contract is byte-shape:
   `<stamp> DUMP-OK <sid> archive/sessions/<name> ms=<ms>`, one
   `DUMP-RETRY=1 … ms=<ms> err=<one-line>`, `DUMP-FAIL … <one-lined detail>`
   (the child's multi-line error message is one-lined — one event, one line).
   Best-effort: a failure NEVER blocks the dispatch, the response carries the
   v1 `WARNING: pre-compaction dump failed for <sid> (<error>)` line, success
   leaves the response UNCHANGED. `dump-skipped` is gone (tool description
   updated; the response mentions the dump only on failure).
5. ONE dump leg covers SELF and CROSS (v1 had two call sites; the v2 plugin has
   ONE execute). A new `compact_v2.log` audit line
   `kind:"dump" state:"ok"|"failed"` (v2 addition — v1 had no such line) lands
   per dispatch; denials dump NOTHING (ZERO side effects preserved).

## Measured verification (all commands run in the CLONE)

- `node .opencode/plugin/probes/handover_probe.mjs` → final line
  `PROBE handover: 360/360 PASS` (exit 0) → PASS (UNCHANGED vs the b5ccc0e
  baseline; the probe's dump lane pins the v1 plugin path, so no repin was
  needed).
- `bun run compact_memory.smoke.mjs` (in `.opencode/plugins/tests`) → exit
  0, 63 `PASS` lines, 0 `FAIL`, final `SMOKE: ALL PASS` → PASS.
- `bun run smoke.mjs` / `context_family.smoke.mjs` / `context_trim_v2.smoke.mjs`
  (same dir) → exit 0, `SMOKE: ALL PASS` (52 / 46 / 65 PASS lines) → PASS.
- `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p tsconfig.json`
  (same dir) → exit 0, ZERO output lines → PASS.
- The 12 node-dir smokes (`.opencode/plugin/tests`): 9 exit 0 with
  `…ALL PASS`; `gauge_core.smoke.mjs`, `context_trim.smoke.mjs`,
  `auto_resume.smoke.mjs` exit 1 with `FAIL=0` and the teardown crash
  `EBUSY: resource busy or locked, rm '<scratch dir>'` → PRE-EXISTING Windows
  flake, already logged from 4b (measured identical at the 4b baseline); this
  unit touched none of the files those smokes read.
- Direct twin exercise (sandbox COPY of the script + a COPY of `sqlite3.exe`
  + FIXTURE DBs built with `node:sqlite`, never the live DB): full/`--lite`/
  `--json` `--out` lanes; `--all --slim` corpus line; unsafe relpath exit 2;
  v1-only fixture exit 2 (`no v2 tables`); `ses_nope` exit 1; the md byte-
  EQUAL across the `node:sqlite` and `spawn-sqlite3` lanes modulo the backend
  field; `bun:sqlite` picked FIRST under the bun host. The same lanes are
  now PINNED inside `compact_memory.smoke.mjs` (S-T4c).
- Working tree: clone `clean` (0 dirty entries); MAIN dirty = this summary file
  only.

## Clone commits (branch `migration`, not pushed — agents never push)

- `167a7c2` — "Add the v2 dump twin script: v2-table reads + backend chain + v1
  CLI parity (TODO #137 unit 4c)".
- `136426b` — "Wire the pre-compaction dump leg into the v2
  compact_memory plugin (TODO #137 unit 4c)" (plugin + the 4 smoke repins).
- `a8a61ac` — "Add the 4c dump pins to the compact_memory smoke and document
  the twin (TODO #137 unit 4c)" (smoke + README).
- `cfbb426` — "One-line the ctx.log DUMP-FAIL detail so one dump event writes
  one log line (TODO #137 unit 4c)".
Checkpoint note (honest): the DUMP-FAIL one-line fix was needed only AFTER the
new pins ran, so `a8a61ac` + `cfbb426` together form one verified unit —
`cfbb426` is the first commit whose tree holds the full verified state (the
gate above was measured on that tree; the tree has been unchanged since).

## Discrepancies found (for the planner to route into `TODO.md` / inboxes)

1. Spec text said the CLONE `context_trim_v2.ts` carries a `TRIM_BACKENDS`-style
   chain — it does NOT (4b rides the gauge core's `readGauge` and holds no
   chain). The twin's chain follows MAIN's v1 `.opencode/tools/context_trim.ts`
   (~115-166) + the gauge core's spawn discipline.
2. Spec text listed the v2 tables as `session` (id, model, time_updated) +
   `message` + `session_message`. The actual v2 pairing (the gauge core's own
   SQL) is `session_v2` ↔ `session_message`; the twin rejects a DB without that
   pair rather than reading the v1 pair.
3. `execFileSync` failure shapes (measured on BOTH hosts): the child's stderr
   arrives as a Buffer on `err.stderr`, and the error MESSAGE already embeds
   the stderr text — so the v1 `| stderr: …` detail needs an explicit one-line
   of the whole detail (v1's appender did it; the port initially dropped it).

## Deliberately omitted / deviations

- No dedicated smoke file for the script — the plugin smoke's S-T4c section
  pins it (DoD 1 allows either; one gate command covers both).
- The MAIN corpus-refresh cadence (`dump_session.cjs --all --slim`) is NOT
  switched to the twin (spec out-of-scope); the MAIN corpus stays on v1's script.
- Live v2-host acceptance (a real compaction writing a real dump) =
  maintainer domain; the marker-mode dispatch path shares the ONE dump leg
  (pinned indirectly by the marker legs' dump audit lines, not a dedicated pin).
- No loop-log line and no `submit` writes — MAIN's `loop/` and inbox files sit
  behind the spec's "MAIN untouched" boundary; the notes are in this file.
- The twin prints `backend=<name>` in its summary line (a v2 diagnostic
  addition to v1's line) — recorded rather than hidden.
- The v2 dump has no `orphan_parts` lane in `--json` (no part table) — the
  doc shape is `{session, messages[…]}`.

## Blockers

None.

## Lessons (one, reusable)

A v1-parity port must re-check the HOST's API shape, not just the v1 code: bun
1.4.3's `bun:sqlite` Database exposes `prepare`/`query` (NO `db.all` sugar), and
`execFileSync` failures carry `stderr` as a Buffer whose message already embeds
it — so keep `String(buffer)` coercion and one-LINE the whole detail to hold
the ctx.log one-line-per-event contract.

## Context state at close (gauge readout, verbatim)

    SESSION=ses_e9de1b381ffeM4308gaIvhBgTk CTX=204282 | 5 compactions left
