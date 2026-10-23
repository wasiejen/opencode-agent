  `ctx_family.log` carries the evidence lines).
# Task spec — TODO #137 unit 4a: compact_memory + U3 marker compaction as a v2 plugin (clone `migration`)

## Goal
Add the v2-native compaction plugin to the clone: the `compact_memory` agent
tool (budget-gated manual compaction) + the U3 marker mode, both built on the
v2 native surfaces (`ctx.session.compact`, `ctx.session.hook("compaction")`
`result` override, `delivery` queue) instead of the v1 SDK/DB plumbing.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (checked out; clean at `9e069cc`). Stay on the current checkout.
- Assigned worker: `worker_FlashNext` (same model — the cache rule).
- TODO entry: `TODO.md` #137 (main repo; the planner records status + hashes).
- Design of record (READ-ONLY): main `.opencode/plugin/compact_memory.ts`
  (header 1-90 = the behavior contract) + `.opencode/plugin/compaction_core.ts`
  (budget/config/cap/line layer).

## Definition of done (measurable)
1. NEW plugin `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\compact_memory.ts`
   — v2 shape: DEFAULT export `Plugin.define` form (`{id?, setup(ctx)}`; TYPE-ONLY
   `@opencode/plugin` import, value imports node: builtins + plain `.mjs` only —
   the context_family.ts header lines 9-13 state the loader contract). Auto-discovered
   from `.opencode/plugins/` — NO `opencode.jsonc` change (3a precedent).
2. TOOL REGISTRATION — inside `setup`, `ctx.tool.transform` with
   `editor.add({...})`: name `compact_memory`, description mirrors v1, input = a
   plain JSON schema `{sessionID?: string, emergency?: boolean, message?: string,
   mode?: "auto"|"marker", summary?: string, boundary?: string}`. `execute(input,
   toolContext)` is a plain Promise (promise-surface contract).
3. SELF/CROSS — SELF (no `sessionID`) = the CALLING session, resolved from
   `toolContext.sessionID` (the ToolContext carries it — no SDK self-trick); CROSS
   (explicit `sessionID`) = fire-and-forget, NEVER awaited (same-model dispatch rule).
4. BUDGET LAYER — ported from `compaction_core.ts` into a new paired module
   `.opencode/plugin/scripts/compaction_core_v2.mjs` (plain JS, node:fs/path only;
   the v2 plugin VALUE-imports it — the gauge.mjs precedent). Same file
   `.opencode/temp/compact_budget.json`, same keys, same semantics: cap resolver
   (bare model id → cap; unlisted/typo → `model_budget.default`, else 1; CPU models
   cap 0 — the SAFETY INVARIANT, tested first), denial → zero side effects (no
   increment, no dispatch, no line), at `count == cap` the `emergency` arg consumes
   the configured emergency 1 (count → cap+1; count > cap = exhausted), CROSS gets
   effective cap+1 (caller-scoped, #128 ruling), INCREMENT-ON-SUCCESS ONLY, the
   `ovr` token on the COMPACT line for cross. The v1 core module stays
   UNIMPORTED by v2 code (inert-dir contract).
5. DISPATCH — `ctx.session.compact({sessionID})` (POST `/api/session/:id/compact`;
   body `{id?, delivery?}` — NO keep fields exist on v2; see Verified fact 3).
   The response is NEVER a success claim. The budget increment + the COMPACT line
   land ONLY on a verified compaction: subscribe `ctx.event` for the session's
   compaction lifecycle (`session.compaction.started` / `.failed` / `.ended`
   types) and increment on the success signal, `.failed` → one `verify=failed`
   evidence line, no burn. If the event leg proves unreliable at worker time,
   fall back to the gauge-paired DB read of the durable `compaction` row —
   record which leg landed in the header + pin it in the smoke either way.
6. COMPACT LINE — byte-identical v1 format to the shared `.opencode/temp/ctx.log`
   (`COMPACT <sessionID> ...` + ` ovr` when cross) — the cross-host log stays
   one format; plugin evidence lines (dispatch/denied/marker/verify) go to
   `.opencode/temp/compact_v2.log` (capped, never-throws appends, the
   context_family.log pattern).
7. U3 MARKER MODE — `ctx.session.hook("compaction", cb)` registered in `setup`:
   a per-session PENDING-MARKER store (in-memory Map + the queued marker args
   written by the tool call with `mode:"marker"`, `summary` REQUIRED) → when the
   hook fires for that session, set `event.result = {summary, metadata:{marker:true}}`
   → the core uses it and SKIPS the summarizer model request. Deviation (header
   REQUIRED): v2 gives the plugin NO boundary control — `recent` is core-computed
   from config `compaction.keep.tokens`; the `boundary` arg is accepted, ignored,
   and logs ONE `boundary-ignored` evidence line; the durable `compaction` message
   is written by the core (no direct DB writes — the v1 `markerSet` quartet-write
   core has NO v2 counterpart).
8. QUEUE LEG — the `message` arg: v1 stored a temp file + the auto_resume unit-4
   relay (NOT PORTED in v2 — port map order 2 skipped). v2 native replacement:
   deliver via `delivery:"queue"` on the prompt/inbox channel AFTER the dispatch
   (fire-and-forget; capture+evidence on rejection). Header records: relay
   semantics changed from resume-relay to native queue; NO temp-file queue store
   needed.
9. KEEP KEYS — the budget file's `keepMessages`/`keepTokens` DO NOT apply to a
   v2 dispatch (the compact body has no keep fields). Behavior: read for
   auditability, one `keep-not-applied` evidence line when present; retention =
   config `compaction.keep.tokens` (the ONLY v2 control). Do NOT write config
   values (maintainer domain).
10. SMOKE — NEW `.opencode/plugins/tests/compact_memory.smoke.mjs` (bun, mock
    ctx, scratchpad fixture DBs, live DB NEVER read; mirror the context_family.smoke
    harness): denial zero side effects; cap resolver forms (CPU guard, unlisted
    default, listed); emergency at count==cap + exhausted above; SELF via toolContext
    sessionID; CROSS fire-and-forget + effective cap+1 (` ovr` token); increment
    ONLY on the verified-success signal; `.failed` → no burn + evidence;
    marker hook → `result` override captured + `boundary-ignored` line; queue
    delivery captured (`delivery:"queue"`) + rejection evidence; `keep-not-applied`
    line; compact body carries NO keep fields (pin the captured call). Final line
    `SMOKE: ALL PASS`.
11. GATE — clone: probe `PROBE handover: 360/360 PASS` UNCHANGED; all 12 node
    smokes ALL PASS; ALL v2 plugin smokes ALL PASS (incl. the new one);
    typecheck `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p
    .opencode/plugins/tests/tsconfig.json` zero errors (add the plugin to `files`).
    Clone tree clean; MAIN repo untouched except the handover summary write.
12. CHECKPOINT COMMITS in the clone (code only; core module BEFORE the plugin
    import, then plugin+smoke together if a split would be red); worker summary
    to the MAIN working tree `agent/handover/handover_task_to_planner.md`.

## Suggested scope (non-exhaustive)
- NEW: clone `.opencode/plugins/compact_memory.ts`,
  `.opencode/plugin/scripts/compaction_core_v2.mjs`,
  `.opencode/plugins/tests/compact_memory.smoke.mjs` (+ tests tsconfig `files`).
- Reference (READ-ONLY): main `.opencode/plugin/compact_memory.ts` (args +
  dispatch + line semantics), `.opencode/plugin/compaction_core.ts` (budget layer
  lines ~33-520), clone `.opencode/plugins/context_family.ts` (v2 plugin shape +
  evidence-file pattern), clone `.opencode/plugins/tests/context_family.smoke.mjs`
  (harness), clone `.opencode/plugin/scripts/gauge.mjs` (paired-DB reads for the
  optional verify fallback).
- Fork READ-ONLY: `packages/plugin/src/promise/tool.ts:34-59` (ToolEditor
  add/list/update), `:59-100` (ToolDomain transform),
  `packages/schema/src/tool.ts:14-19` (ToolContext.sessionID), `:92-99`
  (Tool.Info name/input/description/execute), `:86-90` (Result),
  `packages/plugin/src/promise/session.ts:43-52` (SessionCompactionResult +
  SessionCompaction `result?`), `:139-163` (SessionDomain Pick incl. `compact`),
  `packages/client/src/promise/generated/types.ts:4654-4663` (SessionCompactInput
  {sessionID,id?,delivery?} + Output), `:1343-1357` + `:2040` (compaction event
  types), `packages/core/src/session/compaction.ts:402` + `:601-612` (the result
  override consumption + fromHook), `packages/core/src/config/plugin/compaction.ts:15-24`
  (config keep → compaction.transform).

## Verified facts (measured at spec-authoring, 2026-10-22, fork @ current checkout)
- `ctx.session.compact` EXISTS in the plugin SessionDomain Pick; the generated
  client maps it to POST `/api/session/:sessionID/compact` with body
  `{id, delivery}` only — NO keep/summary fields on the request
  (types.ts:4654-4663, client.ts:799-810). Output = an inbox item (manual compaction
  is an inbox item per compaction.ts trigger reasons).
- The compaction draft hook: `SessionCompaction extends SessionContext` with
  `result?: SessionCompactionResult {summary; providerState?; metadata?; tokens?}`
  — doc: "Set to use this compaction and skip the model request"
  (promise/session.ts:43-52). Core consumption: compaction.ts:402 `if
  (prepared.event.result) return yield* fromHook(...)`; fromHook (compaction.ts:
  601-612) wraps `supplied.summary` + the CORE-computed `recent` into the durable
  result → NO plugin-side boundary control. The durable message renders as
  `<conversation-checkpoint>` `<summary>`/`<recent-context>`
  (runner/to-llm-message.ts:306-325).
- Kind hook firing: `SessionModelRequest.compaction = prepare("compaction", …,
  agentHook("compaction"))` — model-request.ts:428; primary fires `context`
  only, so the marker hook handler never runs on primary requests.
- Events: `session.compaction.started` (data {sessionID, reason auto|manual,
  recent, inputID?}) and `session.compaction.failed` exist (types.ts:1343/1357);
  an `ended` type exists at :2040 — verify the exact success signal at worker
  time before wiring the increment.
- `ToolContext` (schema/src/tool.ts:14-19) carries `sessionID` → SELF resolution
  is local; `Tool.Info` = {name, input, description, execute, output?, options?}.
- v1 U3 writes the marker quartet THROUGH `context_trim.ts`'s `markerSet`
  (compact_memory.ts:80 import) — in v2 that whole DB-write lane is REPLACED by
  the hook `result` override + the core's durable message (biggest deviation;
  header REQUIRED).
- v2 auto_resume is NOT ported (port map order 2 skipped; clone plugins/ =
  intercept_observer.ts + context_family.ts only) → the v1 resume-relay has no
  v2 counterpart; `delivery:"queue"` is the native fit (3b precedent: the nudge
  delivery already uses it).

## Approval boundary
Pre-approved class: agent-facing compaction machinery; no maintainer-visible
behavior change; live firing waits for the maintainer's v2 host runs (backend
slot). DO NOT touch: the MAIN repo (except the handover summary write);
`maintainer/**`; the fork (READ-ONLY); clone `.opencode/plugin-unported-v1/**`,
`.opencode/plugins/intercept_observer.ts`, `.opencode/plugins/context_family.ts`,
`opencode.jsonc`; the live DB (fixture DBs only, `mode=ro`); spec/prompt files.

## Procedure (guidance, not protocol)
1. Read this spec + the v1 headers (compact_memory.ts 1-90, compaction_core.ts
   budget section ~33-120 + cap resolver 209-260).
2. Port the budget layer to `compaction_core_v2.mjs`; commit it (green alone).
3. Write the plugin: tool registration + dispatch + verify-increment + marker
   hook + queue; wire the evidence files.
4. Smoke + typecheck + full clone gate; checkpoint commit (plugin+smoke if a
   split is red — record the reason like 3b did).
5. Worker summary to the MAIN working tree; final message points at
   `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- The pre-compaction DUMP leg (#152): `agent/scripts/db/dump_session.cjs` is v1-table
  shaped; a v2-table twin rides follow-up spec `2026-10-22_137u4c-dump-twin.md`
  (queued, next planner). The v2 plugin logs `dump-skipped` ONCE per dispatch.
- Unit 4b (context_trim v2 report) — same wave, separate spec, launch AFTER 4a verifies.
- auto_resume v2 (order 2), the small tools (order 5-8), live v2 acceptance
