# Worker handover — TODO #132: the silent default-cap-1 readout marker ` (unlisted)`

Worker: worker_Q3S_slow (ses_f09e89374ffeFc47RHh5PKNCot), branch `opencode_test`
(no branch work, no push). Spec: `agent/handover/handover_task.md` (single source
of truth). Status: **LANDED** — the code is on disk, the standard gate is GREEN at
the measured numbers below.

## What changed (3 code commits)

1. **`748f7cd` — `.opencode/plugin/scripts/gauge.mjs` + its smoke (gauge_core +2)**
   - `compactionsLeftSuffix` now tracks WHICH branch set the cap: a new
     `unlisted` flag, set ONLY on the silent built-in cap-1 fallback — the map
     present-but-model-unlisted/empty + no finite `model_budget.default` key
     (the `else { cap = 1 }` fall-through, L438-440) and the map key entirely
     ABSENT / not-an-object (`else { cap = 1 }`, L441-442). The CPU invariant
     (`/^cpu/i` → cap 0) and the explicit model / explicit `default` branches
     NEVER set it. The suffix line appends the byte-exact marker ` (unlisted)`
     after `left` iff the flag is set — ` | 1 compaction left (unlisted)`.
     No other byte of the line changed; the `remaining` computation is
     untouched.
   - `gauge_core.smoke.mjs` item-3 block: +2 pins — (a) unlisted modelId + map
     present with other entries but NO `default` key → ` | 1 compaction left
     (unlisted)`; (b) the `model_budget` key entirely ABSENT → ` | 1 compaction
     left (unlisted)`. Both fixture writes are inline (not via `wb`, which bakes
     the listed MB3 model), placed between state-2b and the fail-open pin; the
     existing pins are byte-identical.
2. **`bbd8705` — `.opencode/plugin/probes/handover_probe.mjs` (354 → 355)**
   - New check **28.8** in S6b (no-total read, modeled on 28.7): the budget
     fixture carries `model_budget` with other entries (`probe-model-120K_MTP:
     3`, `probe-model-256K_MTP: 2`) and NO `default` key; the session entry for
     `ses_fx_nomodel` carries the unlisted model `probe-model-unlisted` → the
     entry-model fallback resolves it → silent cap 1 → the line ends
     `SESSION=ses_fx_nomodel CTX=notAvailable | 1 compaction left (unlisted)`.
     Asserts `r.ok === false && r.kind === "no-total" && r.sid === "ses_fx_nomodel"
     && r.modelId === ""` (the read model is still the DB row model — the entry
     fallback is suffix-side) + the exact line. Placed after 28.7, BEFORE the
     section's restore line (L1674-1675, untouched) — the section ordering and
     the restore are intact. **28.7 is the discriminating no-marker pin and
     stays byte-identical (PASS, unchanged).**
   - Tally: S6b header comment `(7)` → `(8)`; the header annotation (L185-195)
     `(7)` → `(8)` + the marker case appended; the section-sum line (L1027)
     `S6b=7 … 354/354` → `S6b=8 … 355/355` (the final expected probe string
     repeated the total — updated).
3. **`e3ae1c1` — the 3 doc one-liners (comment/description only)**
   - `ctx_gauge.ts` description (L22): the suffix sentence gains the marker +
     its trigger condition (silent built-in fallback = model unlisted/empty +
     no finite `model_budget.default` key).
   - `compaction_core.ts` header (model_budget doc block, L52-54): the
     "else the default 1" clause gains the readout-marker line (comment only —
     the CODE is untouched, per DO-NOT-TOUCH).
   - `ctx_watchdog.md` L16: the suffix sentence gains the marker + trigger.

## Measured verification (standard gate, all green)

- **Probe: `PROBE handover: 355/355 PASS`** (baseline 354 + 1 = 28.8), exit 0.
  All 8 S6b checks PASS (28.1-28.7 byte-identical + 28.8 new).
- **All 11 smokes green** (plain `node`, exit 0 each), measured this session:
  auto_resume **155/155**, block_transfer.sandbox **64/64**, block_transfer
  **131/131**, compact_memory **89/89**, context_recovery **17/17**,
  context_trim **29/29**, ctx_gauge **3/3**, gauge_core **ALL PASS** (now incl.
  the 2 new pins — the 13 pre-existing pins unchanged), intercept_observer
  **78/78**, loop_log **69/69**, submit **31/31**.
- **ruff F=0**: FST venv `ruff check --select F agent/scripts` → `All checks
  passed!`, exit 0.
- **Marker byte-exactness:** ` (unlisted)` (one leading space, lowercase) —
  asserted verbatim in both new gauge_core pins, probe 28.8, and the smoke
  output lines above. Present ONLY on the silent fallback: 28.2/28.7 (explicit
  `default` key → no marker), 28.1/28.6 (listed model → no marker), the CPU
  branch (cap 0, untested-by-flag by construction — the flag is only set on the
  two silent branches).
- **DO-NOT-TOUCH honored:** `maintainer/**`, `AGENTS.md`, `agent/prompts/**`
  untouched; the live `.opencode/temp/compact_budget.json` never written (all
  steering via `setBudgetFileForTest`); no budget-file schema change (the
  marker is readout-only); the CPU branch untouched; `compaction_core.ts`
  CODE untouched (header comment only); `auto_resume.ts` / `compact_memory.ts`
  / `context_recovery.ts` / the FST repo untouched.

## TODO entries

- `TODO.md` #132: status → **LANDED** (2026-10-01, plan6, worker_Q3S_slow) —
  the marker, the branch coverage, the pin deltas, the doc one-liners, and the
  measured gate. The code commit hash(es) ride the PLANNER's follow-up
  bookkeeping commit (a worker commit can never carry its own hash). No new
  todo_inbox.md entries — no doc/code discrepancy surfaced (see the one
  measurement note below, which is not a discrepancy).

## Discrepancies / notes for the planner

1. **None against the spec.** The verified facts in the spec (the single cap
   mirror in `compactionsLeftSuffix`, the two silent entry points, the existing
   pin surfaces, the baseline 354/354) all held as written; nothing was
   re-derived and nothing had to be changed beyond the spec's scope. The only
   judgment call taken (allowed by the spec — "the fixture shape is your call"):
   28.8 uses the **no-total** read shape (the `readGauge` override sid
   `ses_fx_nomodel` + a budget entry carrying the unlisted model via the
   entry-model fallback), matching the section's existing 28.6/28.7 shape; the
   ok-shape would have needed a new DB fixture row and was the heavier path.

## Deliberately NOT done

- No `logged line` variant of the marker (the spec's scope is the readout
  suffix only — a log line would be a separate, unapproved behavior).
- No `default` key added to the live `compact_budget.json` (DO-NOT-TOUCH; the
  documented convention already exists in the header comment).
- No live acceptance test (the maintainer's own next gauge read of an unlisted
  model will show ` | N compaction(s) left (unlisted)` — natural occurrence).

## Commits (this task)

- `748f7cd` — gauge.mjs marker + gauge_core +2 pins
- `bbd8705` — probe 28.8 + tally 355/355
- `e3ae1c1` — the 3 doc one-liners

(Final bookkeeping commit carries these hashes + the TODO #132 status + this
handover.)
