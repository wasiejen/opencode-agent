# Task spec — compact_memory caller-scoped override (TODO #128)

Worker: worker_Q3S (170K). Pre-approved class (agent-usage). Queue: #127 is
LANDED (2026-10-01, plan2) — this is the live task spec. Branch truth: stay
on the current checkout (`opencode_test`).

## Goal

Implement the maintainer-approved caller-scoped compaction override: a CROSS
caller (target sessionID ≠ the calling session) gets an effective cap of
`cap + 1`; the cross spendable total stays ≤ `model_budget +
emergency_budget` — THE SAME AS SELF.

## Ruling (2026-09-30, comment in
`proposals/approved/2026-09-30_compact-memory-cross-override.md`)

"Option (b) … only up to model_budget + emergency_budget can be spend via
cross. the same as when self-compacting." → the proposal's original Part 1
(which also allowed a cross second slot at `count == cap+1` via the
emergency arg → cap+2) is RETRACTED: cross at `count == cap+1` stays
REFUSED. Part 3 (the `ovr` audit token) is INCLUDED (maintainer live
comment 2026-09-30: "part 3 can be implemented as well").

## Verified facts (measured at spec time, 2026-09-30 — do not re-derive)

- The gate: `.opencode/plugin/compact_memory.ts` L410-440 —
  `if (count >= cap) { if (count === cap && emergencyArg &&
  cfg.emergency_budget >= 1) isEmergency … else REFUSE }`.
- Ids: `explicitID` L384-387, `isSelf = sessionID === c?.sessionID` L391.
  New: `isCross = explicitID != null && !isSelf` (an explicit id equal to
  the caller = self).
- VERIFIED-FACT CORRECTION (the proposal's "CPU cap 0 denied for all" line
  is INACCURATE): at cap 0 the current gate ALLOWS one EMERGENCY self
  compact (`count 0 === cap 0 && emergencyArg`, L425). Under "the same as
  self" a cap-0 cross therefore gets exactly 1 as well. Do NOT add a
  special CPU denial — keep the uniform arithmetic (flagged to the
  maintainer).
- The Gate-M smoke block: `.opencode/plugin/tests/compact_memory.smoke.mjs`
  ~L422-500 (fixture `model_budget { "Gate-M": 2, default: 1 }`). Refusal
  pins at ~L444 / L460-461 / L487 assert wording (`cap 2`, `2/2`, `3/2`,
  `fully exhausted`, `unavailable or already consumed`). AUDIT each pin's
  self/cross shape (its `toolCtx`/args) before re-pinning — bounded read of
  the block.
- Part 2 docs (worker part): the tool description's budget paragraph (top
  of `compact_memory.ts`) + the `compaction_core.ts` budget header comment.
  The planner-prompt line + the AGENTS.md staged copy are PLANNER-SIDE
  (workers have no edit access to `agent/prompts/**`) — do NOT touch prompt
  files; name them in the handover as pending planner-side.

## Design (the gate change, exact)

- After L391: `const isCross = explicitID != null && !isSelf;` and
  `const effCap = isCross ? cap + 1 : cap;`
- The gate condition `if (count >= cap)` → `if (count >= effCap)`. Inside,
  the emergency branch is UNCHANGED (`count === cap && emergencyArg &&
  cfg.emergency_budget >= 1`) — it now only fires for SELF at
  `count == cap` (a cross at `count == cap` passes `count < effCap`);
  cross at `count == cap+1` falls through to the refusal.
- Refusal message: for cross at `count == cap+1`, name the override as
  consumed (the caller-scoped override slot is used — the budget is fully
  exhausted); the SELF refusal wording stays unchanged (pins depend on
  it). Cross at `count == cap` with the `emergency` arg = allowed via the
  plain pass (the arg is moot; the increment happens once on success — no
  double consumption).
- Denial keeps ZERO side effects (no increment, no COMPACT line) — the
  invariant stays.
- Part 3 (audit token, zero behavior): when a CROSS dispatch SUCCEEDS with
  pre-dispatch `count == cap` (the override slot is consumed — the target's
  normal cap was already spent), the verified-success COMPACT line
  (`compaction_core.ts:319-332`) gains the `ovr` token (next to the existing
  `emergency` suffix). Self dispatches and cross dispatches at
  `count < cap` get NO token.

## DO-NOT-touch

- The self-path behavior (all self pins stay green), the store schema, the
  maintainer's live files, `agent/prompts/**`, anything under `maintainer/`.

## Definition of done

1. Gate pins (Gate-M block, fixture cap 2) — new/adjusted:
   (a) cross@count==cap, no emergency → dispatched (count → cap+1);
   (b) cross@count==cap, `emergency: true` → dispatched, same store state;
   (c) cross@count==cap+1 → refused (names the override consumed);
   (d) self@count==cap, no emergency → refused (regression, wording
   unchanged); (e) self@count==cap + emergency → dispatched (regression);
   (f) self@count==cap+1 → refused (regression); (g) the cap-0 corner: a
   cap-0 model (find the existing CPU fixture name pattern — grep `cpu` in
   the smoke) self@0+emergency → dispatched then refused; cross@0 →
   dispatched (0 < effCap 1), then refused at 1.
2. The tool description + the `compaction_core.ts` header updated with the
   ruling wording (cross total = self total).
3. `compact_memory.smoke.mjs`: 82/82 + the new pins green — incl. the
   Part-3 pins: the verified-success COMPACT line of a cross@cap dispatch
   carries `ovr`; a cross dispatch at `count < cap` and the self emergency
   dispatch do NOT; probe 352/352 (re-pin the compact section IF it pings
   the refusal wording or the COMPACT line shape); all other smokes green
   (standard gate).
4. Checkpoint commits per verified unit; the TODO #128 status + the handover
   summary ride the FINAL commit (naming the pending planner-side doc
   parts).
5. Live acceptance = the next real cross-override dispatch (maintainer
   domain) — name it in the handover.

Context discipline: bounded reads only (the named regions + the Gate-M
block); no broad research.
