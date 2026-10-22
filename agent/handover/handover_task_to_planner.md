# Worker summary — TODO #137 unit 4b: context_trim ported to v2 (clone `migration`)

Status: COMPLETE. Both DoD units landed as clone checkpoints; the gate is green.

## Task and scope

Ported the v1 `.opencode/tools/context_trim.ts` (report + tail) to v2 as a plugin
in the CLONE `C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch `migration`,
started at 358cba7 = the post-4a state). Stayed on the current checkout; no
branch switch. MAIN repo: only this summary file written (plus two inbox appends,
listed below).

## Files changed

CLONE (branch `migration`):
- NEW `.opencode/plugins/context_trim_v2.ts` (804 lines) — the v2 plugin: the
  `context_trim` tool via `ctx.tool.transform` + `editor.add`, and ONE
  `ctx.session.hook("context")` handler (the trim's effect).
- NEW `.opencode/plugins/tests/context_trim_v2.smoke.mjs` (65 checks).
- EDIT `.opencode/plugins/tests/tsconfig.json` — the plugin added to `files`.
- EDIT `.opencode/plugins/tests/README.md` — the new smoke's section.

MAIN (working tree only, NOT committed):
- `agent/handover/handover_task_to_planner.md` (this file).
- `todo_inbox.md` + `agent/agent_feedback.md` + `agent/memory/worker/memory_inbox.md`
  appended via the `submit` tool (3 entries — content summarized below; the MAIN
  tree stays UNCOMMITTED, the planner curates).

Untouched as required: `maintainer/**`, clone `.opencode/plugin-unported-v1/**`,
clone `plugins/intercept_observer.ts` / `context_family.ts` / `compact_memory.ts`,
`opencode.jsonc`, the live DB, all MAIN code/docs.

## Behavior changed (v2 semantics)

1. WINDOW = the v2 model window: the `session_message` rows from the LAST
   COMPLETED `compaction` row onward — the head row IS included (fork
   `packages/core/src/session/history.ts` `latestCompaction` + `seq >=` load;
   `compaction.ts:690-694 previousCompaction`). No `tail_start_id` exists in v2,
   so the DB-write lane is GONE.
2. REPORT reads the window rows through the v2 service (`sessionApi.context`
   = GET `/api/session/:id/context` — the spec's "ready report source") and
   reuses the clone gauge core (`readGauge`/`formatGauge`) for the ctx figure —
   the gauge core stays the ONLY DB read; NO new SQL, NO backend chain duplicated.
   Row masses: `tokens=` = the fork's TokenUsage.total() sum
   (input+output+reasoning+cache.read+cache.write) gated on the completed-
   assistant discriminator (`$.time.completed` + `$.tokens` — the unit-2 finding);
   everything else `bytes4=` (utf8 bytes/4 over the row's text fields). Forms
   mirror v1: `header: head=… boundary=… boundary-state=… retained=…`,
   `row: N <type> <kind>=<mass> [tool=… target=…]`, `last: N`, the option
   block (v2 = the trim RANGE: low = floor-6 retention, high = earliest legal
   boundary), `biggest:` top-5, and the `target` dry-run.
3. TAIL = the persistent per-session BOUNDARY store
   `.opencode/temp/context_trim_v2.json` (`{sessions:{sid:{boundary,keep,
   set_at}}}`, fail-open read, additive write — the #119 discipline) + the
   `context`-hook filter: per PRIMARY request it drops the draft
   `event.messages` entries whose `id` matches a window row STRICTLY before
   the boundary — assembly-time only, persisted history UNTOUCHED. Entries
   without an `id` (the system/effort/tool-result projections) SURVIVE.
4. Fail-closed validation (a rejection writes NOTHING): `session-not-found`,
   `target-not-found`, `target-at-window-head` (the v2 rename of v1's
   target-not-before-compaction — the head row is never a boundary),
   `retained-tail-below-floor <n>` (the floor 6 carries), and for the keep form
   `keep-invalid <keep>` / `keep-exceeds-history <keep>` / `keep-at-window-
   head <keep>`; keep=0 refused. Success line keeps v1's shape:
   `tail-set= <old> -> <new> keep=<count>`; clear → `tail-set= <old> -> none`.
   Read failures print `context-read-failed: <preview>`.
5. STALE boundary (a new completed compaction row buries it): the hook logs
   ONE `boundary-stale` line (deduped per session+boundary id) and keeps the
   WHOLE post-compaction window (fail-open); the report prints
   `boundary-state=stale retained=<window rows>`.
6. `clear` is a NEW arg (v2-only addition) — v1 un-set by re-pointing its own
   compaction part field; a JSON store needs an explicit un-set lane.

## Measured verification (all commands run from the clone)

- `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360 PASS`
  → PASS (UNCHANGED vs the 358cba7 baseline; also re-measured at baseline in a
  scratchpad worktree: 360/360).
- `bun run context_trim_v2.smoke.mjs` (`.opencode/plugins/tests`) → exit 0,
  65 PASS / 0 FAIL, final line `SMOKE: ALL PASS` → PASS.
- `bun run smoke.mjs` / `context_family.smoke.mjs` / `compact_memory.smoke.mjs`
  (same dir) → all exit 0, 0 FAIL, `SMOKE: ALL PASS` → PASS (unchanged).
- `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p tsconfig.json`
  (in `.opencode/plugins/tests`) → exit 0, ZERO output lines → PASS.
- Clone node-dir smokes (`.opencode/plugin/tests`): 10/13 exit 0 with `ALL
  PASS`; `gauge_core.smoke.mjs`, `context_trim.smoke.mjs`, `auto_resume.smoke.mjs`
  exit 1 with ZERO `FAIL` lines — the crash is bun's teardown
  `fs.rmSync(...)`: `EBUSY: resource busy or locked, rm '…<scratch dir>'`. MEASURED
  IDENTICAL at the 358cba7 baseline (scratchpad worktree @ 358cba7: same three
  exit 1, 0 FAIL) and `context_trim` + `auto_resume` fail the same way in the
  untouched MAIN repo → PRE-EXISTING Windows teardown flake, NOT this unit
  (this unit changed no file those smokes read — `git diff --stat 358cba7..HEAD`
  = the plugin, the smoke, the tsconfig, the tests README).

Working tree: clone `clean` (0 dirty entries); MAIN dirty = the summary file +
the 3 inbox appends only.

## Clone commits

- `471dce7` — "Add the v2 context_trim plugin: report + boundary store +
  context-hook window filter (TODO #137 unit 4b)" (plugin + tsconfig entry).
- `b5ccc0e` — "Add the context_trim_v2 smoke: 65 pins for the v2 window-map +
  boundary-store contract (TODO #137 unit 4b)" (smoke + README section + the
  dry-run `verdict=` fix — the printed line had a hardcoded `ok`).
Not pushed (agents never push).

## Inbox entries submitted (MAIN, via `submit`; no TODO IDs assigned)

1. todo: the three clone node-dir smokes exit 1 on Windows teardown EBUSY
   (`fs.rmSync` of the scratch fixture dir) with zero check failures — baseline-
   reproducible, needs a teardown retry/close-the-handle fix or a scratch-dir
   change.
2. feedback: the block_transfer tool's REPLACE mode silently substituted a stale
   `default` buffer for the supplied `text` (twice corrupted the new files);
   WRITE needs BOTH start AND end markers.
3. memory (worker_FlashNext): byte-pin a report's masses by DERIVING them from
   the documented rule, not by copying measured output; a hardcoded verdict
   string in a report template was caught only by an expectation mismatch.

## Deliberately omitted / deviations (all recorded in the plugin header)

- The v1 FULL report form (id+time per row) and `recommendationOneLiner` (U4)
  are NOT ported — the CUT-DOWN form is the only form; the auto_resume nudge
  channel it served is skipped in v2.
- v1's own SQL/backend chain (node/bun:sqlite → spawn-sqlite3, the marker SQL)
  is NOT re-implemented: the window read rides the v2 service; the gauge core
  stays the only DB reader. The v1 `part`-table shapes have no v2 counterpart.
- The `no-completed-compaction` rejection is GONE (the store lever has no
  compaction dependency). The recommendation block lists the trim RANGE rather
  than v1's low=current-tail-head/high=index-0.
- The v1 `marker` mode lives in 4a's compact_memory plugin — not duplicated.
- No loop-log line (MAIN's `loop/` write would break the "MAIN untouched"
  boundary).
- 4c (dump twin) and the small tools (order 5-8) — separate specs.
- Live v2 acceptance = maintainer domain.

## Blockers

None. Two notes for the planner: (a) the three teardown-EBUSY smokes should get
their own TODO entry (already submitted to `todo_inbox.md`); (b) a dropped
assistant row with UNEXECUTED tool items can leave an orphan `tool`-result
entry (no `id`, so the filter keeps it) — flagged here; the floor-6 + human-
in-loop discipline covers it; a stricter lane would need id propagation in
`to-llm-message.ts` (fork code, out of scope).

action: resume

## Context state at close (gauge readout, verbatim)

    SESSION=ses_e9de1b381ffeM4308gaIvhBgTk CTX=234727 | 6 compactions left

