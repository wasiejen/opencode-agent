# Task spec — TODO #132: the silent default-cap-1 fallback marker in the compaction budget readout

Worker: `worker_Q3S_slow`. Repo: opencode-agent — stay on the current checkout
(`opencode_test`); no branch work.
Scope class: pre-approved (agent-usage friction removal — the only observable
change is the readout line itself).

## Goal
The compaction-budget readout suffix (` | N compactions left` on the injected
`ctx:` line / the `ctx_gauge` tool) must make the SILENT built-in cap fallback
visible. Today a model row unlisted in `model_budget` (with no `default` key)
resolves to cap 1 with no signal — the maintainer measured ~4 tool calls to
diagnose (agent_feedback 2026-09-30_16-57: injected suffix `1 compactions
left` vs self-gauge 5). When the cap resolves via that silent built-in path,
the suffix carries the marker ` (unlisted)`.

## Verified facts (measured 2026-10-01 by the planner — do not re-derive)
- The cap mirror lives in ONE place: `.opencode/plugin/scripts/gauge.mjs`
  `compactionsLeftSuffix(sid, modelId)` (L411-448). Branches: `/^cpu/i` → cap 0
  (CPU invariant, L436); `mb[model]` a finite number → explicit (L438);
  `mb["default"]` a finite number → the explicit `default` key (L439); else
  **the silent built-in cap 1** — reached from TWO entry points: `mb` absent /
  not-an-object (L441-442) and `mb` present but model unlisted/empty + no
  finite `default` key (L438-440 fall-through).
- Suffix format (L447): ` | ${remaining} compaction${remaining===1 ? "" :
  "s"} left`; `remaining = max(0, cap-count) + 1` iff count === cap &&
  effective emergency_budget >= 1 (key absent → fail-open 1).
- Display surfaces of that suffix: the injected `ctx:` line (ctx_watchdog) +
  the `ctx_gauge` tool — both via `formatGauge` → `compactionsLeftSuffix`.
  `auto_resume.ts` reads the same budget file but only for a boolean nudge
  gate (budgetExhausted) — it renders NO suffix. `resolveCap`
  (`.opencode/plugin/compaction_core.ts` L214-221) is the tool/hook GATE
  resolver (returns a label) — its refusal text does not render the suffix;
  untouched.
- Existing pins — ALL verified, NONE exercises the silent branch, so ALL stay
  byte-identical:
  - `.opencode/plugin/tests/gauge_core.smoke.mjs` item-3 block (L97-149):
    fixture `MB3 = { "probe-model-256K_MTP": 3, default: 1 }`, the reads carry
    the LISTED model — cap always the explicit 3.
  - `.opencode/plugin/probes/handover_probe.mjs` S6b (L1539-1672, 7 checks):
    28.1-28.5 listed model (cap 3); 28.6 row model listed (cap 3);
    **28.7 (L1654-1672): model "" (model nowhere) + fixture WITH `default: 1`
    → the explicit-default branch → ` | 1 compaction left`, no marker — it
    becomes the discriminating no-marker pin; unchanged.**
  - `.opencode/plugin/tests/auto_resume.smoke.mjs` L2307 carries a ctx.log
    INPUT fixture string with ` | 5 compactions left` — input, not a suffix
    assertion; unchanged.
- The `default` key convention is ALREADY documented:
  `.opencode/plugin/compaction_core.ts` header L52-54 ("the cap for an
  unlisted / typo'd model id is model_budget.default (else the default 1)").
- Baseline (2026-10-01, plan5): probe 354/354 (header tally L1027: `S6b=7
  ... 354/354`), gauge_core ALL PASS, ctx_gauge 3/3, all 11 smokes green,
  ruff F=0.

## The change (the WHAT — the HOW is yours)
1. `gauge.mjs` `compactionsLeftSuffix`: track which branch set the cap; when
   the silent built-in fallback applied (model unlisted OR empty AND no
   finite `model_budget.default` key; the CPU branch EXCLUDED), append the
   marker after `left` — the line becomes ` | <N> compaction(s) left
   (unlisted)` — byte-exact marker text: ` (unlisted)`.
2. Pins:
   - `gauge_core.smoke.mjs` item-3 block: +2 pins — (a) an unlisted modelId +
     `model_budget` present with other entries but NO `default` key →
     ` | 1 compaction left (unlisted)`; (b) the `model_budget` key entirely
     ABSENT → ` | 1 compaction left (unlisted)`.
   - `handover_probe.mjs` S6b: +1 check (28.8) — unlisted modelId +
     `model_budget` present (other entries, no `default` key) →
     ` | 1 compaction left (unlisted)` (ok-shape or no-total-shape fixture,
     your call — keep the section's restore line L1674 and the section's
     ordering intact). Update: the S6b header annotation (L185-195) gains the
     marker case; the S6b tally 7 → 8; the header total line L1027 →
     `S6b=8 ... 355/355` (+ the final expected probe string if it repeats the
     total).
3. Doc one-liners (ONE line each, no prose expansion):
   - `.opencode/tools/ctx_gauge.ts` description — the suffix sentence (the
     `Returns the ...` paragraph, L22) gains the marker + its trigger
     condition.
   - `.opencode/plugin/compaction_core.ts` header — the model_budget doc
     block (L52-54) gains the readout-marker line (comment only).
   - `.opencode/plugin/ctx_watchdog.md` L16 — the suffix sentence gains the
     marker.

## Definition of done
- The marker is byte-exact ` (unlisted)`, present ONLY for the silent
  built-in fallback (never for the CPU branch, an explicit model entry, or an
  explicit `default` key).
- Every pre-existing pin unchanged and green (the verified-facts list names
  every pin surface this change touches).
- New pins: gauge_core +2, probe +1 (28.8); probe total 355/355 (self-
  annotated header tally).
- Standard gate green: probe + all 11 smokes + ruff F=0.
- TODO.md #132 status → LANDED (the code commit hash rides the PLANNER's
  follow-up bookkeeping commit — a worker commit can never carry its own
  hash); the worker handover in `agent/handover/handover_task_to_planner.md`
  (measured gate numbers + any discrepancies).
- Checkpoint commits per verified unit; TODO + handover ride the FINAL commit.
- No new TODO entry expected; if a doc/code discrepancy surfaces, file it via
  todo_inbox (the planner curates).

## DO-NOT-TOUCH
- `maintainer/**`, `AGENTS.md`, `agent/prompts/**` (no edit access to prompts
  anyway).
- The live `.opencode/temp/compact_budget.json` (smokes steer via
  `setBudgetFileForTest` only); NO budget-file schema change (no new keys —
  the marker is readout-only).
- The CPU branch (no marker); `compaction_core.ts` CODE (its header comment
  only); `auto_resume.ts`; `compact_memory.ts`; `context_recovery.ts`; the FST
  repo.
