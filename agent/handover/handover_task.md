# Task spec — TODO #137 unit 3b: nudge ladder + handover pre-flight in the v2 context-family plugin (clone `migration`)

## Goal
Extend the clone plugin `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\context_family.ts`
(landed by spec `2026-10-22_137u3a-context-family-core.md`) with the two
remaining watchdog legs, mirroring the v1 `.opencode/plugin/ctx_watchdog.ts` v2.6
semantics: (A) the 5-rung nudge ladder fired per completed tool call, delivered
through the v2 synthetic channel; (B) the handover-delegation pre-flight warn.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (checked out — stay on it; the tree is clean after unit 3a).
- Assigned worker: `worker_FlashNext` (same model — the cache rule).
- TODO entry: `TODO.md` #137 (main repo; the planner records status + hashes).

## Definition of done (measurable)
1. NUDGE LADDER — registered on the v2 tool AFTER-hook
   (`ctx.tool.hook("execute.after", …)`):
   - per fire: ONE gauge read for the event's OWN `sessionID` (per-session read —
     the v1 per-session ruling stays: a newest-session read would blind-spot
     concurrent sessions);
   - rungs VERBATIM from v1: 1 `pct>=50` | 2 `pct>=70 or REM<=30k` | 3 `pct>=80 or REM<=20k`
     | 4 `pct>=90 or REM<=10k` | 5 `REM<5k`; condition = pct OR REM whichever hits first;
     the HIGHEST rung met fires; at most ONE nudge PER RUNG PER SESSION (in-memory
     map keyed by session id);
   - unknown window / `no-total` / `db-error` reads NEVER fire (silent);
   - the rung texts mirror v1 (rung 5 carries the VERBATIM gauge readout +
     "stop line reached" phrasing; rung 3 wind-down, rung 4 commit+NAP-now).
2. DELIVERY — `ctx.session.synthetic({ sessionID, text, delivery: "queue" })`,
   FIRE-AND-FORGET: NEVER awaited in the hook; sync throws + rejections caught and
   evidence-logged (`kind:"nudge"`, reason `delivery-*`); a rejection NEVER fails
   the tool call.
3. EVIDENCE — `kind:"nudge"` lines ONLY when a nudge fires: `{session, rung, readout}`
   shape (capped JSON, the v1 volume discipline — NO line for a non-fire).
4. HANDOVER PRE-FLIGHT — on `ctx.tool.hook("execute.before", …)`: when
   `event.tool === "subagent"` and `event.input` is an object whose `prompt`
   string contains `agent/handover/handover_task.md`: the file MISSING or EMPTY
   (in the repo the delegation targets — resolve relative to the plugin's own
   repo root, `.opencode/plugins/` parent) → ONE warn line (kind
   `handover-preflight`). OBSERVATION ONLY: never blocks, never mutates
   `event.input`.
5. SMOKE — extend `.opencode/plugins/tests/context_family.smoke.mjs` with pins:
   each rung boundary (50/70+REM30k/80+20k/90+10k/REM<5k — fixture DBs), the
   highest-rung-met rule, the per-rung dedup (second fire same rung → no line),
   unknown-window/db-error silence, the synthetic delivery call captured on the
   mock ctx (`delivery:"queue"`, text carries the readout), a mock rejection →
   evidence line + call not failed, the pre-flight missing/empty warn + the
   non-handover silence. Final line `SMOKE: ALL PASS`.
6. Clone gate green UNCHANGED: probe `PROBE handover: 360/360 PASS`; 12 node
   smokes ALL PASS; the v2 intercept smoke unchanged; BOTH v2 plugin smokes
   ALL PASS; typecheck (`tsgo -p .opencode/plugins/tests/tsconfig.json`) zero
   errors.
7. Checkpoint commits per verified unit IN THE CLONE (code only); clone tree
   clean; worker summary to the MAIN repo working tree
   `C:\Users\Wasiejen\Repos\opencode-agent\agent\handover\handover_task_to_planner.md`
   (planner commits it).

## Suggested scope (non-exhaustive)
- Clone `.opencode/plugins/context_family.ts` (extend; 3a's regions stay intact)
  + `.opencode/plugins/tests/context_family.smoke.mjs` (+ fixture DB helpers).
- Reference (READ-ONLY): main `.opencode/plugin/ctx_watchdog.ts` header lines 96-135
  (v2.6 rung table + delivery + evidence) and lines 128-140 (v2.7 P02 removal —
  the summary-mirror must NOT be re-ported); fork
  `packages/plugin/src/promise/tool.ts:36-70` (hook shapes),
  `packages/client/src/promise/generated/types.ts:4592-4610`
  (`SessionSyntheticInput`), `packages/core/src/tool/plugin/subagent.ts:35`
  (`prompt: Schema.String`); the clone v1 twin
  `.opencode/plugin-unported-v1/ctx_watchdog.ts` (the rung TEXTS verbatim —
  grep the nudge strings there).

## Verified facts (measured at spec-authoring, 2026-10-22, clone @4054a0a + 3a landed)
- `execute.after` event (`promise/tool.ts:47-63`): `{tool, sessionID, agent,
  messageID, id, input}` + `status:"completed"|"error"` with `result`|`error` —
  the sessionID is present → per-session read is possible (v1 ruling carries).
- `execute.before` event: same shape, `input` MUTABLE; ONLY `execute.before`
  may reject a call (`core/src/plugin/hooks.ts:23`) — the pre-flight NEVER
  throws.
- v2 delegation tool = `subagent` (v1 `task` → v2 rename, port map §4); its
  input object has `prompt: string` (fork `tool/plugin/subagent.ts:35`).
- `ctx.session` domain INCLUDES `synthetic` (the Pick list
  `promise/session.ts:139-163`); `SessionSyntheticInput` =
  `{sessionID, id?, text, description?, metadata?, delivery?: "steer"|"queue",
  resume?}` (generated client types :4592) — `delivery:"queue"` queues as the
  next turn at idle (the v1 synthetic-part semantics); POST path
  `/api/session/:id/synthetic`.
- v1 v2.6 rungs + fire-and-forget + dedup + silence rules: main ctx_watchdog.ts
  header lines 101-128 (verbatim above in DoD 1).
- v2.7 lesson (lines 128-140): the `tool.execute.after` SUMMARY MIRROR was
  REMOVED (P02, 7 collisions) — do NOT port it.
- 3a landed the plugin file + its smoke; baseline = re-run the gate
  (probe 360/360, 12 smokes, both v2 smokes) — the worker measures the fresh
  counts; expect NO probe changes (the probe pins the v1 module estate +
  hygiene; it has no `.opencode/plugins/` inventory check — grep-verified).

## Approval boundary
Pre-approved class (agent-facing context machinery; no maintainer-visible
behavior change; live firing waits for the maintainer's v2 host runs). DO NOT
touch: the MAIN repo except the handover summary write; `maintainer/**`; the
fork (read-only); clone `.opencode/plugin-unported-v1/**`; `.opencode/plugins/intercept_observer.ts`;
`opencode.jsonc`; the live DB except `mode=ro`; the worker-prompt/spec files.

## Procedure (guidance, not protocol)
1. Read this spec + 3a's plugin file + the v1 v2.6/v2.7 header block (96-140).
2. Port the rung table + texts verbatim; wire the after-hook + delivery.
3. Wire the pre-flight on the before-hook.
4. Extend the smoke; run the new smoke + typecheck + the full clone gate.
5. Checkpoint commits (clone, code only); worker summary to the MAIN working
   tree; final message points at `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- Port map §4 order 4+ (compact_memory/context_trim/U3 + the small tools).
- The budget store (`compact_budget.json`) v2 wiring — the gauge core already
  reads it; nothing new here.
- Live acceptance of nudges on the v2 host (backend slot = maintainer domain).
- The v1 event-observer plugin.log role (retired in v2 — the plugin's own
  `ctx_family.log` carries the evidence lines).
