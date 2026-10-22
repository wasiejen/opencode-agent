# Task spec — TODO #137 unit 5a: loop_log + submit ported to v2 (clone `migration`) — LAUNCH FIRST of the orders 5-8 wave

## Goal
Port the two append-with-stamping tools (`loop_log`, `submit`) to v2 as plugin-
registered tools, behavior byte-compatible with v1 (same line formats, same
rejection strings, same channel map), adapted to the v2 tool-context carrier.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (HEAD `cfbb426` after unit 4c). Stay on the current checkout.
- Assigned worker: `worker_FlashNext`.
- TODO entry: `TODO.md` #137. Design of record (READ-ONLY): clone
  `.opencode/tools/loop_log.ts` (235 lines — header lines 1-70 carry the Parts A-D
  contract; read header + grep the format strings, not the whole file) + clone
  `.opencode/tools/submit.ts` (167 lines — header lines 17-40 carry the channel
  map + stamp rule). Registration pattern: clone
  `.opencode/plugins/compact_memory.ts` (landed by unit 4a).

## Definition of done (measurable)
1. NEW plugin `.opencode/plugins/loop_log.ts` — v2 DEFAULT export
   `{ id: "loop_log", async setup(ctx) {...} }`; registers the tool via
   `await ctx.tool.transform((editor) => editor.add(loopLogTool))`; the setup
   returns a teardown closure disposing the registration (compact_memory.ts
   pattern). Args: `status` (free-form string), `content` (required string),
   optional `role` / `model` / `session`.
2. loop_log behavior = the v1 contract, carried:
   - status keyword table → the 8-char tokens (`-->START`/`DONE<---`/`-RETURN-`/
     `-WARNING`/`--INFO--`/`CORRECT-`); unrecognized status → the v1 error
     string, NOTHING written;
   - line = `<stamp> <status> <role> <session> <model> <content>`, append-only;
   - folder: newest `autorun-*` under `loop/`, else create
     `autorun-<YYYY-MM-DD_HH-MM>`; the multi-folder anomaly note;
   - readback verify: `folder:` / `verified: readback-match` (or
     `readback-MISMATCH: <actual last line>`) — byte-compare the last line;
   - auto-fill chains on the v2 context: session = args.session →
     `context.sessionID`; role = args.role → `context.agent`; model = args.model →
     `context.agent` (the identifier preference) → else the literal `unknown`;
     NEVER throws.
3. NEW plugin `.opencode/plugins/submit.ts` — same plugin shape, id `submit`.
   Args `feedback?`/`knowledge?`/`todo?`/`ideas?`/`memory?` (all optional strings,
   >= 1 required, an empty string counts as NOT provided). Channels + header
   levels byte-identical to v1: `agent/agent_feedback.md` (###),
   `agent/knowledge/knowledge_inbox.md` (##), `todo_inbox.md` (##),
   `agent/agent_ideas.md` (###), `agent/memory/<role>/memory_inbox.md` (###).
   Entry = `<header> <YYYY-MM-DD_HH-MM> <role> <session>` + the raw text;
   role/session auto-filled (role = `context.agent` else `agent`; session =
   `context.sessionID` else `unknown`); targets resolved against the plugin's
   repo root; the final-byte newline probe kept (2026-10-21 fix); append-only —
   NEVER reads/rewrites a target BODY.
4. SMOKES — new `.opencode/plugins/tests/loop_log.smoke.mjs` and
   `.../submit.smoke.mjs` (bun, mock ctx — the fixture loop/ and inbox trees live
   in a scratchpad subdir, e.g. `.../Temp/opencode/small-write-tools-smoke-v2`).
   Pins: the token table + one case per keyword; the unrecognized-status error
   (no write); the folder create/locate + anomaly lines; the readback lines; the
   auto-fill chains incl. the `unknown` fallback; submit's channel headers +
   stamp shape, the zero-arg error string, the empty-string rule, the memory
   role-scoped path, the newline handling. Each file's final line
   `SMOKE: ALL PASS`.
5. GATE — clone probe `PROBE handover: 360/360 PASS` UNCHANGED (the probe pins
   the v1 files — untouched); the 12 v1 node smokes ALL PASS (incl.
   `plugin/tests/loop_log.smoke.mjs` + `submit.smoke.mjs` — they pin the v1
   `.opencode/tools/` files, DO NOT repurpose); ALL v2 plugin smokes ALL PASS;
   `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0 (add BOTH new plugins
   to the tsconfig `files` list); clone tree clean.

## Verified facts (measured at spec-authoring, 2026-10-22)
- v2 tool contract (fork @ `654b79b27e`): `Tool.Info` requires `name`, `input`,
  `description`, `execute(input, context)`; `output`/`options` optional
  (`packages/schema/src/tool.ts:99-106`); `Result` = `{output?, content?,
  metadata?}` (`:84-88`).
- `ToolContext` (promise surface) = `{ sessionID, agent, messageID, id, signal,
  progress }` — `packages/schema/src/tool.ts:14-19` +
  `packages/plugin/src/promise/tool.ts:11-24`. NO `directory`, NO `extra`,
  NO `client`, NO `session.sessionId` variants (the v1 Part A chains' legs
  `context.sessionId`/`context.session?.id`/`context.extra.model.id` have NO
  v2 carrier — logged deviation; the `unknown` fallback carries).
- `input` accepts a plain JSON Schema object (compact_memory.ts:494 `TOOL_INPUT`
  = `{type:"object", additionalProperties:false, properties:{...}}`).
- Registration idiom proven in the clone: compact_memory.ts `:693-694`
  (`ctx.tool.transform` + `editor.add`) + teardown `dispose()`; the repo root
  carrier = setup-time capture `dir = ctx.location.directory`
  (context_family.ts:497 precedent — v1 used `context.directory`).
- Plugin discovery: `.opencode/plugins/*.ts` DEFAULT export must decode
  `{id, setup}` (clone `.opencode/plugin-unported-v1/README.md` mechanism
  section) — a v1 `tool()` default export does NOT decode; no config entry needed
  (directory discovery).
- Baselines: clone probe 360/360; v2 smokes — intercept 52, context_family 46,
  context_trim_v2 65, compact_memory 63; v1 node smokes incl. loop_log 69/69
  and submit 31/31.

## Suggested scope (non-exhaustive)
- NEW: clone `.opencode/plugins/loop_log.ts`, `.opencode/plugins/submit.ts`,
  their two smokes, the tsconfig `files` entries, the tests README sections
  (`.opencode/plugins/tests/README.md`).
- Reference READ-ONLY: clone `.opencode/tools/loop_log.ts` + `submit.ts` (the
  v1 twins), clone `.opencode/plugins/compact_memory.ts`, the MAIN repo
  (bookkeeping only).

## Approval boundary
Pre-approved class (agent-facing tool port; live v2 acceptance = maintainer
domain, backend slot). DO NOT touch: MAIN repo (except the worker summary
+ planner bookkeeping); `maintainer/**`; clone `opencode.jsonc`,
`.opencode/tools/**` (the v1 twins — inert design-of-record),
`.opencode/plugin-unported-v1/**`, `.opencode/plugin/**` (v1 smokes/probes),
`plugins/intercept_observer.ts`, `plugins/context_family.ts`,
`plugins/context_trim_v2.ts`, `plugins/compact_memory.ts`, the live DB.

## Procedure (guidance)
1. Read the two v1 headers (loop_log 1-70, submit 17-40); grep the format
   strings + rejection strings; read compact_memory.ts's plugin-module section
   only.
2. Implement both plugins (thin; share nothing between files).
3. Smoke each, add the tsconfig entries + README sections, run the full clone
   gate; checkpoint commit per verified tool; worker summary to the MAIN
   working tree `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- The other small tools (5b: ctx_gauge + state_snapshot + dev helper; 5c:
  block_transfer) — separate specs.
- `session_info` — native in v2, NO port (port map §4 order 8).
- auto_resume v2 (port map order 2 — never launched; see the NAP queue note).
- Live v2 acceptance (maintainer domain); the v1-side reduction/dual-merge unit.
