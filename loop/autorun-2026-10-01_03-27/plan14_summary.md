# plan14 summary — autorun planner-14 (2026-10-22)

Session: ses_e9eeb0952ffeP6BY7SnI5j2Yzs, planner_FlashNext, qwen3.8-flash-next-coder-iq1_m, 262k, iter 14.

## Unit — TODO #142 + #138 (one spec, both findings)

Goal: the observation log's path channels (`out-of-sandbox` note, `path-anomaly`)
classify ONLY the tool's PATH-FIELD values, and the config allow-list roots join
the note check. #142 = no note for a path under an `allow`-ed root; #138 = no
path verdicts for path-looking substrings inside free text.

Spec: `agent/handover/specs/2026-10-22_142-138-path-field-scan.md` (committed
`314ae76`, copied to `agent/handover/handover_task.md`).

Worker: `worker_FlashNext` ses_e9ee33291ffeTzq5RYmLfksD0x.

## Landed

- MAIN `main` `dd5b63b` — `intercept_observer_core.ts` `observeArg` gains
  `extraRoots: string[] = []` + `pathScanText: string | null = null` (the (d)
  anomaly + (e) note scan the override; defaults byte-identical for direct
  calls); `intercept_observer.ts` gains `pathFieldText(tool, args)` (joins the
  `REDIRECT_PATH_FIELDS` string values), the hook passes `allowedRoots` +
  `pathText` at the observation site (was 1433) and the EFFECTIVE field text
  at the redirect recompute (was 1474); smoke pins repurposed (2b)/(12n)/(B3)
  + 3 new pins; probe 303/304 bash-note halves repinned.
- CLONE `migration` `622e75e` — the same change with the v2 `path` field names
  (`plugins/intercept_observer.ts` ~814/~912/~923/~929) + 3 new smoke pins
  inserted after the whole-log invariants.
- MAIN bookkeeping `70736fb` — the TODO status lines + worker summary + loop
  lines + the worker's inbox entries.

## Verification (planner, from files + re-runs)

- `git show --stat` matches both trees' claims; both trees clean apart from
  untracked compaction dumps (pre-existing from planner-12/13 self-compactions).
- My re-runs: main `intercept_observer.smoke.mjs` 96/96; main
  `probes/handover_probe.mjs` 360/360; clone `plugins/tests/smoke.mjs` 52 PASS
  lines + `SMOKE: ALL PASS`; clone probe 360/360.
- Design spot-check of the core diff: the pair/dense/numword channels still
  scan the full arg string; only the two path channels narrowed; the initial
  scan uses the ORIGINAL typed fields (the doubled-segment collapse pins still
  fire), the recompute re-reads the EFFECTIVE fields (a redirected target still
  suppresses the note). `runRedirectCommand` (the bash temp-root substitution)
  untouched.
- Accepted the worker's pin-audit correction (smoke (B3), probe 303, 304 —
  the first probe run failed exactly those two, nothing else).

## Curation

- TODO #138, #142 → closed headers + planner stamp lines with the hashes.
- TODO #140 → closed header (units A+B landed + live-accepted by planner-13,
  the residual note fixed by #142); the unit-A env-robustness follow-up stays
  in todo_inbox 2026-10-22_01-34.
- maintainer/priority.md item 1 (cross-repo block-transfer) FULLY HANDLED →
  removed, moved to `maintainer/_past_priorities.md` with the reply line.
- todo_inbox 2026-10-22_03-17 (the spec's incomplete Pin audit) → RESOLVED
  in place, no TODO entry (a spec-authoring gap, not a repo defect).
- Committed the two pre-existing compaction-dump pairs (recovery history).

## Queue for planner-15

1. TODO #137 unit 2 = the token-mass legs (clone; the gate is green).
2. TODO #139 — intercept.log sid/timestamp labeling (fix-vs-document decision).
3. ITER-15 maintenance pass at the next counter trigger (nudge: the worker's
   `a5f6446`/`70736fb` inbox entries + the two stale "live acceptance pending
   restart" batches ride the cure).
4. PENDING his decision (carry-overs): the 3 Magic Context proposals + the
   2-tier tool-description proposal; the memory/knowledge-usage research item;
   #134 B' natural occurrence; the FST logging live confirmation; #135 U4
   `nudge=` leg.
5. HELD-ON-BACKEND-SLOT: the clone's live v2 host runs stay the maintainer's.

## Baselines (post-unit)

MAIN: probe 360/360, intercept_observer smoke 96/96 (was 93), bt smoke 150/150,
sandbox 76/76, SUBMIT_SMOKE 32/32, gauge_core 32/32.
CLONE (`migration`): probe 360/360, `plugins/tests/smoke.mjs` 52/52 (was 49),
v1-twin intercept smoke 85/85 (frozen twin, untouched), other smokes as in
planner-13's list.

## Friction + lessons submitted

- Feedback (from the worker): `block_transfer` WRITE with TWO `regions` and ONE
  `text` writes the SAME text into EVERY region (needed a repair script).
- Planner memory: the spec pin-audit must enumerate the pin inventory by
  GREPING the verdict tokens in every pin file (the first gate run's FAIL list
  is authoritative) — three flipping pins were missed by a range-based audit.
- Planner memory: the loop-log `-RETURN-` line duplicated a hash (content
  echo) — cosmetic, keep the DONE line authoritative.

Gauge at close: CTX≈120k/262k, 6 compactions left.
