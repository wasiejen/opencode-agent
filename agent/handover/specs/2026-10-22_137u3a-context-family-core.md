# Task spec — TODO #137 unit 3a: the v2 context-family plugin core (clone `migration`)

## Goal
Port the context-family core of the v1 `.opencode/plugin/ctx_watchdog.ts` into ONE
new v2 plugin in the clone: the gauge readout through the shared (already v2-paired)
gauge core + the per-request `ctx:` line injection at the v2 `session.context` hook
with cache discipline + evidence log lines. The nudge ladder and the handover
pre-flight ride spec 3b (same file, next launch).

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (checked out — stay on it). Main repo = only the handover summary write (below).
- Assigned worker: `worker_FlashNext` (same model — the cache rule; 262k window).
- TODO entry: `TODO.md` #137 (main repo). Status line recorded by the PLANNER's
  bookkeeping commit; the worker writes its own handover only.

## Definition of done (measurable)
1. New file `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\context_family.ts`
   exports the DEFAULT plugin object ONLY (`{ id: "context_family", setup }` — the
   v2 loader contract; see Verified facts).
2. Every PRIMARY model request gets exactly ONE injected text piece whose text starts
   `ctx: ` and equals the gauge readout line (same shape the v1 ctx line and the
   `ctx_gauge` tool print: `SESSION=… CTX=n (p%) REM=m | N compactions left`,
   or `CTX=notAvailable`).
3. Cache discipline: the injection is APPEND-ONLY AT THE TAIL of
   `event.messages` — the bytes of every entry BEFORE the injection point are
   UNCHANGED (a smoke pin byte-compares the untouched prefix).
4. Own-session gate: the gauge read runs with the request's own `sessionID`;
   the core returns a different `sid` → SILENT skip (no post, no line) — v1 v2.5
   semantics. An honest own-session readout (incl. `CTX=notAvailable`) IS posted.
5. `db-error` → no injection + ONE evidence line (reason `db-error`, capped
   preview) — v1 vocabulary; other readout kinds post normally.
6. New smoke `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\tests\context_family.smoke.mjs`
   (bun, mock ctx — no opencode process; fixtures in the scratchpad
   `C:/Users/Wasiejen/AppData/Local/Temp/opencode/ctx-family-smoke-v2`, NEVER the
   live DB; gauge steered at a fixture DB like the existing v2 smoke does) pins
   DoD 2-5 + the prefix-unchanged pin; final line `SMOKE: ALL PASS`.
7. Typecheck green: extend `.opencode/plugins/tests/tsconfig.json` `files` to
   include `../context_family.ts`; `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/plugins/tests/tsconfig.json` → zero errors.
8. Clone gate green UNCHANGED: `node .opencode/plugin/probes/handover_probe.mjs` →
   `PROBE handover: 360/360 PASS`; the 12 node smokes ALL PASS; the existing
   v2 intercept smoke `bun .opencode/plugins/tests/smoke.mjs` unchanged
   (`SMOKE: ALL PASS`); the new smoke ALL PASS.
9. Checkpoint commits per verified unit IN THE CLONE (code only — the #141
   precedent); clone tree clean after. Write the worker summary to the MAIN
   repo working tree at `C:\Users\Wasiejen\Repos\opencode-agent\agent\handover\handover_task_to_planner.md`
   (uncommitted; the planner commits it with its bookkeeping).

## Suggested scope (non-exhaustive)
- NEW: clone `.opencode/plugins/context_family.ts` + `.opencode/plugins/tests/context_family.smoke.mjs` (+ tsconfig `files` line).
- Reference (READ-ONLY): clone `.opencode/plugins/intercept_observer.ts` (the
  v2 plugin shape + the fixture-DB gauge steering pattern in
  `.opencode/plugins/tests/smoke.mjs`); main
  `C:\Users\Wasiejen\Repos\opencode-agent\.opencode\plugin\ctx_watchdog.ts`
  header lines 96-140 (v2.4 / v2.4.1 / v2.5 injection + gate + evidence vocabulary);
  clone `.opencode/plugin/scripts/gauge.mjs` (the `readGauge` export + fixture
  setters); fork `C:\Users\Wasiejen\Repos\opencode` `packages/plugin/src/promise/session.ts`,
  `packages/core/src/session/model-request.ts:422-431`; research
  `C:\Users\Wasiejen\Repos\opencode-agent\agent\research\2026-10-15_opencode-v2-impact.md`
  §3 lines 247-275 (the hook + "filter at assembly ≠ persisted history").

## Verified facts (measured at spec-authoring, 2026-10-22, clone @4054a0a)
- v2 plugin surface (`promise/plugin.ts:26-55`): `ctx.session`, `ctx.tool`,
  `ctx.event` (`subscribe` only — `promise/event.ts:3`), `ctx.storage`, etc.
  `Plugin.define` = `{id, setup(context)}` (`promise/plugin.ts:59-66`); the
  loader reads the DEFAULT export only (the clone intercept plugin's header
  states the rule + the TYPE-ONLY `@opencode/plugin` import requirement).
- `SessionDomain` (`promise/session.ts:139-163`) = Pick<SessionApi, create|get|
  remove|switchAgent|switchModel|prompt|generate|command|compact|synthetic|
  interrupt|update|move|wait|context> & `{ hook: ModelHooks<SessionHooks> }`.
- `SessionHooks` keys (:157-170): `prompt, context, compaction, generate, title,
  "model.request", "http.request", "http.response", experimental.ws.*, retry`.
- `SessionContext` (:36-40) extends `SessionRequest` (:20-27):
  `{sessionID, model, system, messages, options}` + `agent, tools`;
  `system`/`messages`/`options` are mutable (`Types.DeepMutable` on the
  prompt entry; the arrays mutable) — hook fires per PRIMARY request at
  `core/src/session/model-request.ts:422-431`; kinds register separately.
- The clone gauge core is ALREADY v2-paired (unit 2: clone `922f74c`+`2c1fa23`,
  gauge_core smoke 42 pins ALL PASS): `readGauge(undefined, sessionID?)` →
  `{ok, kind, ctx, pct, rem, sid, window…}`; the budget suffix reads
  `.opencode/temp/compact_budget.json` relative to the clone.
- v1 injection vocabulary (main ctx_watchdog.ts): one text piece `ctx: <readout>`,
  append-only onto the NEWEST message (v2.3 REJECTED: anything BEFORE the
  newest message = prompt-cache invalidation — recorded ruling, keep it),
  session-gated match-only post (v2.5), evidence kind per fire with a `sess`
  field, failure reasons `db-error` / `parts-not-array` / `invalid-messageID`.
  v2 lands the injection at the request DRAFT instead of the stored message —
  transient, never persisted (research §3).
- Clone baselines NOW: probe 360/360; 12 smokes ALL PASS; `.opencode/plugins/tests/smoke.mjs`
  47 checks `SMOKE: ALL PASS`; `tsconfig.json` has `"files": ["typecheck.ts"]`;
  bun on PATH (`C:/Users/Wasiejen/AppData/Roaming/npm/bun`).
- The v2 `Message` entry shape inside `event.messages` comes from `@opencode/ai`
  (union per type) — the exact shape of the injectable entry is the worker's
  bounded question: tsgo + ONE grep in fork `packages/ai/src/**` for the
  message/part types; pick an append form tsgo accepts, fail-closed otherwise.

## Approval boundary
Pre-approved class (agent-facing tooling; no maintainer-visible behavior change;
the live v2 host activation stays maintainer restart domain). DO NOT touch: the
MAIN repo except the handover summary write; `maintainer/**`; the fork repo
(read-only); clone `.opencode/plugin-unported-v1/**` (frozen v1 twins);
clone `.opencode/plugins/intercept_observer.ts`; the live DB except `mode=ro`;
`opencode.jsonc` anywhere (registration/config = maintainer domain).

## Procedure (guidance, not protocol)
1. Read this spec + clone intercept plugin header + `.opencode/plugins/tests/README.md`
   + the main ctx_watchdog.ts header lines 96-140.
2. Answer the bounded shape question (tsgo + one fork grep).
3. Implement `context_family.ts` (hook registration + readout + append-tail
   injection + evidence lines to `.opencode/temp/ctx_family.log` — the
   v1 plugin.log role moves to this file's own log; keep lines capped JSON).
4. Write the smoke; run: new smoke → `SMOKE: ALL PASS`, typecheck, the full
   clone gate (probe + 12 smokes + the v2 intercept smoke).
5. Checkpoint commits per verified unit (clone); write the worker summary into the
   MAIN repo working tree; clone tree clean; final message points at
   `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- The nudge ladder + watchdog handover pre-flight (spec
  `agent/handover/specs/2026-10-22_137u3b-nudge-ladder.md`, same file, NEXT
  launch); compact_memory/context_trim/U3 (port map §4 order 4); the small
  tools (order 5-8); live v2-host acceptance of the ctx line (backend slot
  = maintainer domain; state PENDING in the handover); the v1-side event
  observer log (no v2 role — note it as retired-in-v2 if asked).
