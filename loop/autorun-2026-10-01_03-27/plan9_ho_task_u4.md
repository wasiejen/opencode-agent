# Task spec — U4: nudge policy reframe (nudge1-4 config keys)

Design source (read first): `proposals/approved/2026-10-01_context-budget-package.md` §U4.
DEPENDS ON U1 — the triage line's recommendation one-liner is the U1 output
in compact form; write/verify against the U1 implementation (landed
before this unit per the queue order).

## Goal
ONE CONFIG KEY PER RUNG in `compact_budget.json` (absent = that rung off —
zero drift until enabled): `nudge1_triage: 0.80`, `nudge2_stop: 0.90`,
`nudge3_emergency: 0.95`, `nudge4_maintainer: 0.98`. The triage rung's line
carries the recommendation one-liner (U1 compact form). The existing
`saturationThreshold` behavior is preserved whenever all nudge keys are
absent (backward compatibility). The prompt-ladder reduction is the
PLANNER'S direct follow-up — NOT in this worker's scope.

## Verified facts (measured 2026-10-01 by the planner)
- `.opencode/plugin/auto_resume.ts`: `saturationConfig` L592-610 (reads
  `.opencode/temp/compact_budget.json` per call — the new keys ride the
  SAME file + read path), `nudgeText` L616-620, `appendNudge` L628-633,
  `onToolAfterNudge` L652-688 (ratio computation L664-668, the
  scope/Direct-suppression gate L673-675, once-per-busy-cycle dedup
  L680-683).
- Live `.opencode/temp/compact_budget.json` today: single gate
  `saturationThreshold: 0.85` — the new keys are ADDITIVE; every existing
  auto_resume pin must stay green.
- The recommendation one-liner: produced from the U1 report logic in
  compact form (the worker reads the U1 implementation at run — the U1
  pins are the reference for the shape).

## DoD (measured end states)
- ONE PIN PER RUNG: rung met (ratio ≥ the rung's key value, no higher rung
  met) → its line once per busy cycle; a higher rung beats it; config key
  absent → that rung never fires; Direct session → suppressed (the
  existing scope gate).
- The triage line carries the recommendation one-liner (fixture-pinned).
- Backward-compat pin: ALL nudge keys absent → the pre-change exact
  `saturationThreshold` line + behavior (existing auto_resume pins stay
  green — re-verify the count at run).
- No tool/prompt surface change (the prompt edits are the planner's
  follow-up).
- Standard gate green: probe + all 11 smokes (auto_resume smoke at its
  current baseline — re-verify the count at run).
- Checkpoint: one code commit per verified unit; TODO + handover in the
  final commit (hashes recorded by the planner, not the worker).

## DO-NOT-touch
- `agent/prompts/**` (the planner's domain — do NOT edit); the fork;
  `maintainer/`; other tools/plugins; the proposal files; the live DB.
- New TODO entries: expectation only, never a promise.

## Worker
worker_Q3S (config-driven rung ladder + per-rung pins).
