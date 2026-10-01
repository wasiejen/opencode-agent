# U4 handover — nudge policy reframe (nudge1-4 config keys)

Worker: worker_Q3S (session ses_f079f0150ffeQxgGCU7jtOKgCM).
Task spec: `agent/handover/handover_task.md` (U4, context-budget package).

## What changed (one verified unit — 3 files)

1. **`.opencode/tools/context_trim.ts`** — new export
   `recommendationOneLiner(dbPath, sessionID): Promise<string>` (READ-ONLY,
   no write statement ever executed): the U1 recommendation block in
   compact form — the 3 retention options (low/medium/high, the block's
   anchor choice) as `keep from <N> ≈<K>k | ...` + the biggest-droppable
   single turn as `drop turn <row> (≈<K>k)`. Empty string on no-session /
   db-error / no parts (the caller then emits the bare triage line).
   Shape-reference deviations from the proposal's illustrative example
   (the U1 smoke pins are the shape reference, per the spec):
   - the keep-option anchors are numbered by the message's 1-based
     position in the SESSION's message list (the medium/high anchors sit
     BEFORE the window — a window row number cannot express them);
   - the drop part carries the turn's TOTAL mass (the U1 biggest list's
     `row=mass` shape), not the example's category word ("≈18k read").
2. **`.opencode/plugin/auto_resume.ts`** — the one-config-key-per-rung
   ladder:
   - `saturationConfig()` now also parses `nudge1_triage` / `nudge2_stop`
     / `nudge3_emergency` / `nudge4_maintainer` from the SAME
     `compact_budget.json` (same per-call READ-ONLY parse; each valid
     only as a finite number 0 < v < 1; absent/invalid → that rung OFF).
   - Gate: ALL four keys absent → the legacy `saturationThreshold`
     behavior EXACTLY (backward compat); ≥1 key present → the ladder
     TAKES OVER (lowest present rung = the gate; the HIGHEST present
     rung met fires — a higher rung beats a lower one).
   - Lines: rung 1 → `--triage: context at ratio=<3dec> — <U1 one-liner>`
     (bare line when the recommendation is empty — the one-liner is the
     only rung needing the DB: `recommendationOneLiner` against the same
     DB path the tail-set leg uses, i.e. the gauge's DEFAULT_DB_PATH /
     the factory's dbPath option); rung 2 → `--stop: context at
     ratio=<3dec> — stop starting new work; checkpoint + handover,
     compact to continue`; rungs 3/4 → the legacy self-compact /
     `--maintainer` lines byte-identical.
   - Unchanged: once-per-busy-cycle nudge= log line format (no rung
     field added), the per-step suffix re-append, the Direct/scope
     suppression, the fail-safety (never throws out of the hook).
   - Header comments updated (UNIT 2 ladder bullet + gate-order notes).
3. **`.opencode/plugin/tests/auto_resume.smoke.mjs`** — new U4 section
   (10 pins) + a recommendation fixture session `ses_u4_rec` ADDED to
   the existing TAILDB (the shared `ses_tail_fix` fixture untouched):
   10 msgs before the marker, marker quartet (tail_start_id = msg_r07),
   summary child, 2 after.

## Measured verification (standard gate — probe + all 11 smokes)

Runtime: **node v24.19.0** (`bun` is broken — the global install lost
bun.exe; everything, including the gate, ran under plain node).

| gate member | before | after |
|---|---|---|
| probe handover | 355/355 | 355/355 (UNCHANGED) |
| auto_resume smoke | 155/155 | **165/165** (+10 new pins) |
| context_trim smoke | 40/40 | 40/40 (unchanged — the export is additive) |
| other 9 smokes | green | green (counts unchanged) |

Pins added (auto_resume smoke, all byte-exact suffix checks):
1. rung 1 bare: ratio 0.833 in [0.80, 0.90) → `--triage: context at
   ratio=0.833` (session not in the fixture db → no recommendation).
2. rung 1 with recommendation (fixture-pinned, machine-recomputed from
   TAILDB in the #99 style): `--triage: context at ratio=0.833 —
   options: keep from 7 ≈9k | 4 ≈14k | 1 ≈17k; drop turn 2 (≈3k)`.
3. rung 2: 0.929 in [0.90, 0.95) → the `--stop` line (the triage rung is
   met too — the HIGHER rung beats it).
4. rung 3: 0.952 → the legacy `self-compact now (ratio=0.952)`
   byte-identical.
5. rung 4: 0.988 → the legacy `⚠⚠ --maintainer: ...` byte-identical.
6. once-per-busy-cycle: second tool result → suffix re-appends, ONE
   nudge= log line for the cycle.
7. Direct (scope none) → suppressed in ladder mode.
8. config key absent → that rung never fires: only `nudge4_maintainer`
   present → 0.952 does NOT fire (the legacy 0.95 default WOULD have —
   the ladder took over the gate).
9. the PRESENT rung fires (0.988 → the maintainer line).
10. backward compat: ALL nudge keys absent (`saturationThreshold: 0.85`
    — the live config's exact shape) → the pre-change legacy line
    exactly.

## Pin audit (per the U3 lesson note)

- NO probe re-pin needed: no tool/prompt surface change; the probe
  registration shapes are untouched; the probe count is unchanged
  (355/355).
- NO re-pins in any smoke: all existing pins stayed byte-exact as-is
  (the live sandbox runs the existing pins with no budget file — the
  fail-open default path — and the new section cleans up its budget
  file). The only count change is the auto_resume smoke 155 → 165
  (+10, all NEW pins).
- No TODO.md entries: no discrepancies, no blocked work, no
  expectation-only findings from this unit.

## Commit

`LANDED` — one final commit (code + this handover + the loop-log
close-down), hash recorded by the planner's bookkeeping commit (the
worker's commit cannot carry its own hash).

## What was deliberately NOT done

- The prompt-ladder reduction (the static 80/90/95 ladders in
  `agent/prompts/**`) — the planner's direct follow-up per the spec
  (DO-NOT-touch honored; no prompt edits).
- No live-config change: `.opencode/temp/compact_budget.json` is
  UNTOUCHED — it still carries only `saturationThreshold: 0.85` (+ the
  live keys), so live behavior is byte-identical to pre-change until a
  rung key is added (zero drift until enabled, per the design).
- No `nudge=` log-line format change (no rung field) — keeps the
  existing evidence pins exact.
- No work in the fork, `maintainer/`, other tools/plugins, the
  proposal files, or the live DB.

## Environment / state flags

- Runtime: node v24.19.0 (bun broken — see the environment note in the
  task spec).
- **Uncommitted live-host state (pre-existing, not from this unit —
  left untouched):** `.opencode/temp/compact_budget.json` carries a
  live budget-store entry for the proposal session
  `ses_f0999d65dffeFxiJylRzecRbhu` (dated today 15:14 — recorded by the
  host's own compact_memory bookkeeping, present in the working tree
  before this session's first commit) plus two untracked compaction-
  dump files for that session under `archive/sessions/compaction_dumps/`.
  Suggest the planner sweeps it into a bookkeeping commit.

Lessons: adding a fixture SESSION to a shared smoke DB needs its
`session` row first (FK); an open DatabaseSync handle at crash time
makes the smoke's final sandbox `rmSync` throw EPERM, which MASKS the
original error (cost one debug run — consider a try/catch with a
logged code in the final cleanup).
