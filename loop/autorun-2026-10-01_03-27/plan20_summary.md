# plan20 summary — autorun planner-20 (2026-10-22, ses_e9d1b5700ffeFZZb50qex5NWBx, ITER=20)

## Did
1. Launched + verified #137 unit 5b (the small read tools → v2 clone plugins).
2. Ran the ITER-20 MAINTENANCE pass (curation + TODO retire + baselines).

## 5b — LANDED + planner-verified
- Launch channel: spec `agent/handover/specs/2026-10-22_137u5b-small-read-tools.md`
  copied byte-exact (MAIN `23e5e14`) + loop copy `plan20_ho_task_u5b.md`.
- Worker `ses_e9d19c62fffeWnNuPGQpxwVMOa` (worker_FlashNext), MAIN summary commit
  `ae4eee2`; CLONE (`opencode-agent-v2`, branch `migration`, on `a5da17b`):
  - `8e630f1` — `.opencode/plugins/ctx_gauge.ts`: the SHARED gauge core wrapped
    via `"../plugin/scripts/gauge.mjs"`; `formatGauge(readGauge(undefined,sid))`;
    `kind:"db-error"` → ` — <error>` append; suffix legs untouched; deviation
    logged: v1 bare-string return → the v2 `{output}` result.
  - `ef3ebbb` — `.opencode/plugins/state_snapshot.ts`: the 6-section capped digest
    byte-shape carried (verbatim MARKER_SWEEP exec, `(none)` empties, the 60-line
    cap with the LAST section trimming first, the EXACT non-planner refusal,
    READ-ONLY pinned); deviation: repo root = setup-captured
    `ctx.location.directory`.
  - `45bb52e` — `.opencode/plugins/dev_get_tool_context_contents.ts`: ONE plain
    string; dump fields = the v2 `ToolContext` keys that EXIST; NO invented legs
    for the v1-only ones; keys-only fallback, never throws.
  - 3 new bun smokes: ctx_gauge 11 + state_snapshot 24 + dev helper 13 pins,
    scratchpad fixture DB pair (live DB never read); tsconfig `files` +3; tests
    README sections.
- My re-runs (clone): probe `360/360` UNCHANGED; the 3 new smokes `SMOKE: ALL
  PASS`; `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0; `git show
  --stat` matched all 3 commits; clone tree clean; the 12 v1 node smokes green
  (the v1 twins untouched). The first probe "crash" I saw was MY wrong relative
  workdir (ERR_MODULE_NOT_FOUND banner), not a gate fault — rerun from the clone
  root green.
- TODO #137 status line LANDED (MAIN `f91e2f6`).
- Worker docs finding FIXED inline (pre-approved): `agent/readme/repo_commands.md`
  gains the v1/v2 smoke-suite + tsgo commands — MAIN `c1062ad` + clone `5434a93`.

## ITER-20 MAINTENANCE pass
- Planner memory: 9 inbox entries → MEM-0121..0128 (worker-narrative fabrication;
  WRITE/REPLACE feedback ≠ landing proof; buffer hygiene; pin-audit token
  enumeration; typecheck negative control; Task launch-rejection signature;
  pre-reuse cross-compact; twin-freshness check) + destill lines; NO-OP: the
  clone-red-gate entry (SUPERSEDED by #141). Inbox EMPTIED.
- Worker memory lane (first curation ever): NEW `agent/memory/worker/memory.md`
  WM-01..WM-10 + `destilled_mem.md`; inbox EMPTIED.
- Knowledge inbox: 3 entries (v2 hook/LLM.Message facts; the v2 DB table pair +
  bun:sqlite/execFileSync facts; the stale clone v1 twins) cured into
  `agent/knowledge/plugin_tools/2026-10-16_v2-plugin-harness-and-config.md`;
  EMPTIED.
- todo_inbox: the EBUSY-teardown finding CURATED → NEW TODO #144 (pre-approved
  test fix, queued behind #137); the twin-staleness entry annotated → #143.
- TODO retire (shrink lane): #121 one-liner + #123 body → `todo_records.md`
  (TODO.md 1311→1286 lines; headers stamped with the records pointer).
  SELF-CAUGHT: my first retire MOVE used id-pattern-grep anchors and dragged 38
  lines (2 section headers + entries 53/56) into records — restored verbatim,
  redone narrowly. Lesson submitted to memory: an entry span ends at the NEXT
  ANY `^## ` header, not the next id-matching one.

## Baselines (current)
- CLONE probe 360/360; v2 smokes ctx_gauge 11 / state_snapshot 24 / dev helper
  13 / loop_log 29 / submit 29 / compact_memory 63 / context_trim_v2 65 /
  context_family 46 / intercept 52 / gauge_core 42; v1 node smokes 12 ALL PASS
  (ctx_gauge 3/3, state_snapshot 20/20 untouched); tsgo exit 0; MAIN gate
  untouched (plan15 baselines carry); EBUSY trio = TODO #144.

## Next (planner-21)
- Launch 5c from `agent/handover/specs/2026-10-22_137u5c-block-transfer-v2.md`
  (MAIN-twin baseline, #143). The §4 wave then ends (order 2 auto_resume v2 =
  HELD maintainer domain; `session_info` native). Then #144.
- Next maintenance trigger: ITER-25. Queue unchanged otherwise; LIVE v2
  acceptance = maintainer (backend slot).

## Gauge at close
CTX=149432 (~57%), 6 compactions left, no live nudge; no mid-session compaction
needed.
