# plan17 summary — autorun planner-17, 2026-10-22 (ses_e9e0be707ffeKYt7BzTQjda5e9, qwen3.8-flash-next-coder-iq1_m, 262k, ITER n=17)

## Goal
TODO #137 port map §4 order 4 — the compaction family in the v2 clone
(`C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`): spec wave
(4a + 4b) + the 4a worker unit (compact_memory v2 twin + U3 marker mode).

## Landed
- Spec wave commit `e2822ec`: `agent/handover/specs/2026-10-22_137u4a-compact-memory-v2.md`
  + `2026-10-22_137u4b-context-trim-v2.md` (queued) + specs/README wave entry (4c queued)
  + NAP planner-17 + loop START. v2 facts verified at authoring against the fork.
- Launch channel `2ed077e`: 4a spec → `agent/handover/handover_task.md` (+ loop copy
  `plan17_ho_task_u4a.md`).
- Clone unit (worker_FlashNext, ses_e9dfde7d5ffeQ4Ya0runeu8s04): `afd6b26` budget core
  `.mjs` twin → `ae2f36f` v2 plugin + 41-pin smoke + tsconfig → `358cba7` tests README.
  Clone tree clean; MAIN gate untouched.
- Bookkeeping (this commit): TODO #137 4a status line, worker summary committed
  (+ loop copy `plan17_ho_to_planner_u4a.md`), NAP entry point for planner-18, loop DONE.

## Verification (planner re-runs, clone root)
- `bun .opencode/plugins/tests/compact_memory.smoke.mjs` → `SMOKE: ALL PASS` (41 pins) — PASS
- `tsgo -p .opencode/plugins/tests/tsconfig.json` → exit 0, zero errors — PASS
- `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360 PASS` — PASS
- `git -C <clone> show --stat` for all 3 commits matches the claimed file lists — PASS
- `git -C <clone> status --short` → empty (clean) — PASS

## Deviations carried (in the plugin header + TODO status)
- VERIFY leg = the `session.compaction.ended`/`.failed` event leg (pinned in the
  smoke); the DB-read fallback stays unported.
- U3 marker: v2 gives NO plugin-side `boundary` (the core computes `recent` from
  config `compaction.keep.tokens`) → `boundary-ignored` line; the durable `compaction`
  message is core-written (no v1 `markerSet` quartet DB write).
- Queue leg = native `delivery:"queue"` at verified success (auto_resume v2 = order 2,
  skipped — the resume relay has no v2 counterpart yet).
- `keepMessages`/`keepTokens` NOT applied on v2 dispatches (`keep-not-applied` audit
  line); retention = config `compaction.keep.tokens` (maintainer domain, not written).
- DUMP leg (#152) skipped until 4c (`dump-skipped` per dispatch).

## Omitted / held
- Live v2 acceptance (compact firing, marker, `CTX=n`) = maintainer (backend slot).
- 4b launch (my context budget — 4b rides planner-18); 4c; orders 5-8.

## Baselines after this plan
CLONE: probe 360/360; 12 node smokes ALL PASS; v2 smokes: intercept 52, context_family
46, compact_memory 41; tsgo 0 bytes. MAIN unchanged (probe 360/360, bt 151/151, sandbox
76/76, intercept 96, SUBMIT 32, gauge_core 32).

## Friction submitted
- (feedback) block_transfer CUT clobbered a staged COPY in the same buffer name → use
  distinct named buffers when assembling multi-part content (worker filed the
  `last_write`-variant of the same trap).
- (memory) planner lesson: verify a worker return from files BEFORE trusting it;
  named block_transfer buffers per artifact; the gauge's missing pct on window-unknown
  reads → divide by the known 262k window manually.
