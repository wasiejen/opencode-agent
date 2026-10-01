# Worker handover — TODO #128: compact_memory caller-scoped compaction override

Session: ses_f0abc032dffenb1K8yrfXKOg5V (worker_Q3S, Qwen3.8-27B-Q3S-170K).
Branch: `opencode_test` (unchanged). Spec: `agent/handover/handover_task.md`.
One verified unit — gate + Part 3 + worker-part docs + pins — landed in a
single commit (the code and its pins are inseparable: the old cross-shape
gate pins go red without the re-pins).

## What changed

### `.opencode/plugin/compact_memory.ts`
- After the `isSelf` line: `const isCross = explicitID != null && !isSelf;`
  (an explicit id EQUAL to the caller = self, per the spec).
- Gate: `effCap = isCross ? cap + 1 : cap`; condition `count >= effCap`; the
  emergency branch is UNCHANGED (`count === cap && emergencyArg &&
  cfg.emergency_budget >= 1`) — it now fires only for SELF at count == cap
  (cross at count == cap passes `count < effCap`; cross at count == cap+1
  falls through to the refusal — no second slot; the cross spendable total
  = cap + emergency = THE SAME AS SELF, the ruling).
- Cross refusal (count == cap+1) names the override consumed:
  "The caller-scoped cross override slot is consumed — the budget is fully
  exhausted." The SELF refusal wording is UNCHANGED (the pins depend on it).
- Denial keeps ZERO side effects (no increment, no COMPACT line) — the
  invariant stays.
- Part 3 (zero behavior): `isOverride = isCross && count === cap` (the gate
  passed, so pre-dispatch count == cap — the override slot consumed) →
  passed as the new 8th arg to `recordVerifiedSuccess` on BOTH dispatch
  paths (v2 `session.compact` + v1 `session.summarize`).
- Tool description: the budget paragraph gained the ruling wording (cross
  gets an effective cap of cap + 1 — the spendable TOTAL stays the same as
  self; cross at count == cap + 1 stays REFUSED).
- The file-header gate bullet documents the override + the `ovr` token.

### `.opencode/plugin/compaction_core.ts`
- `appendCompactLine(..., emergency = false, override = false)`: the ` ovr`
  token appends after the optional ` emergency` suffix (mutually exclusive
  by construction — override is cross-only, emergency is self-only). Line
  shape otherwise unchanged.
- `recordVerifiedSuccess(..., isEmergency = false, isOverride = false)`
  pass-through.
- Budget header comment: the cross-override paragraph (store schema
  UNCHANGED — the single `count` tracks both slots).
- The COMPACT-line header comment documents the ` ovr` suffix.

### Pins (each pin's self/cross shape audited before re-pinning, per the spec)
- `compact_memory.smoke.mjs` 82 → 89 (ALL PASS):
  - Gate-M block (fixture cap 2 — all CROSS shape: ctx `ses_sm_self` ≠ the
    target): (a) cross@cap no-arg → dispatched (count → cap+1), the COMPACT
    line carries `ovr`; (b) cross@cap WITH the `emergency` arg → dispatched,
    SAME store state (one increment — the arg is moot), `ovr` and NO
    ` emergency`; (c) cross@cap+1 → refused naming the override consumed.
    The fresh-re-import pin stays (self shape, still refused). eb=0: a CROSS
    caller is STILL dispatched at cap (the override is UNCONDITIONAL — eb 0
    removes only the SELF emergency), then refused at cap+1. eb absent:
    re-pinned to SELF shape (the fail-open default 1 is only observable on
    the self path — the cross override dispatches regardless of eb), the
    ` emergency` line, NO `ovr`.
  - SELF regressions (d/e/f) via a `selfCtx` helper (ctx sessionID == the
    target, extra model Gate-M): self@cap no-arg → refused (wording
    unchanged, incl. the "`emergency` argument" note); self@cap+emergency →
    dispatched, ` emergency` line, NO `ovr`; self@cap+1 → refused (fully
    exhausted).
  - Cap-0 corner (CPU-Qwen3-0.6B, per the spec's corrected fact):
    self@0+emergency → dispatched then refused; cross@0 → dispatched
    (0 < effCap 1), the COMPACT line carries `ovr`, then refused at 1. NO
    special CPU denial — the uniform arithmetic.
  - IQ4 cap-3 block (cross shape, no-messages fallback → the caller's
    model): 4 dispatched (the 4th = the override at count == cap), the 5th
    refused ("cap 3", "4/3", the override named consumed) + a Part-3 line
    pin; the v2-schema block's count 3 → 4.
  - The CPU "always denied" pins (originally CROSS shape — under the ruling
    they now get the override slot) re-pinned to SELF shape: a plain self
    CPU call (no emergency arg) is refused, cap 0 named, zero side effects —
    the safety invariant survives as "a PLAIN cap-0 call is always denied".
- `handover_probe.mjs` 352/352 (re-pinned, check count unchanged):
  - S13 92 (cap-3 gate, cross): the 4th call = the override dispatched, the
    5th refused ("4/3", override named); 93 (CPU): the plain SELF call
    refused naming cap 0; 95 (v2 schema on disk): count 4; 222 (Gate-M
    cross): call 3 dispatched via the override + `ovr` line + the normal
    lines carry NO suffix; 223: call 4 with the `emergency` arg REFUSED at
    count == cap+1 (the override named consumed — no second slot, NO
    ` emergency` line ever written); 224 stays green as-is (count 3, fully
    exhausted, fresh re-import still refuses); 225 (eb 0): the CROSS
    override is UNCONDITIONAL — dispatched at cap (the `ovr` line), then
    DENIED (the override consumed + the emergency unavailable); 226 (eb
    ABSENT, SELF shape): the fail-open default 1 is consumed, the
    ` emergency` line, NO `ovr`.
  - S32 340 (tool/hook equivalence): the tool (CROSS) dispatches 2 normal +
    1 override (count 3), then REFUSES (the override named consumed); the
    hook side is UNCHANGED (2 normal + auto-emergency, count 3) — THE SAME
    TOTAL: cap + 1.
  - S10 (the deactivated v1 tool) and S11 (the hook) untouched/unaffected.
- All other smokes green (the standard gate): auto_resume 152/152,
  block_transfer 131/131, bt-sandbox 64/64, context_recovery 17/17,
  context_trim 26/26, ctx_gauge 3/3, gauge_core ALL PASS,
  intercept_observer 78/78, loop_log 69/69, submit 31/31.

## Measured verification
- `node .opencode/plugin/tests/compact_memory.smoke.mjs` →
  `COMPACT_MEMORY_SMOKE: ALL PASS (89/89)`.
- `node .opencode/plugin/probes/handover_probe.mjs` →
  `PROBE handover: 352/352 PASS` (exit 0).
- All 11 smokes green (the list above).

## TODO entries
- #128 status → LANDED (the landed line rides this commit).

## What was deliberately NOT done (pending planner-side — named)
1. **The AGENTS.md Compaction Guidelines "Budgets" block** — the STAGED copy
   for the maintainer's paste (the AGENTS.md live-file protocol).
2. **The planner-prompt budget line** — `agent/prompts/agent_planner.md`
   (the Compaction Guidelines "Budgets" paragraph — workers have no edit
   access to `agent/prompts/**`).
3. The hook (`context_recovery.ts`) — unchanged BY DESIGN: the hook has no
   caller concept (it fires on the target session's own limit) — the
   override is caller-scoped to the tool's cross path only.
4. Live acceptance = the next real cross-override dispatch (maintainer
   domain): the verified-success COMPACT line carrying ` ovr` + the budget
   count cap+1 in `compact_budget.json`.

## Deviations from the spec (noted per the worker protocol)
1. The spec named the Gate-M block as the pin site; the probe audit found
   ADDITIONAL cross-shape gate pins outside it (S13 92/93/95, 222-226, S32
   340). The spec's "re-pin the compact section IF it pings the refusal
   wording or the COMPACT line shape" covered them — all re-pinned, the
   probe total stays 352/352.
2. The old CPU "always denied" pins were CROSS shape — under the ruling a
   cross CPU call at cap 0 gets the override slot (the spec's corrected
   fact). I re-pinned them to SELF shape (a plain cap-0 call with no
   emergency arg stays refused, zero side effects) rather than deleting the
   safety-invariant pin — consistent with "no special CPU denial".

Lessons: the spec's pin-audit scope should name the probe's S13 gate checks
(92/93/95/222-226) explicitly — they were cross-shape and their re-pins
were not derivable from the spec's named Gate-M region alone; the probe
run caught them on the first red.
