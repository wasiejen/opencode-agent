# Task spec — TODO #137 unit 5c: block_transfer ported to v2 (clone `migration`) — QUEUED, launch AFTER 5b verifies

## Goal
Port the `block_transfer` tool (the file-manipulation workhorse) to v2 as a
plugin-registered tool: same modes, anchors, buffers, feedback lines, sandbox
rules — re-registered via `ctx.tool.transform` because v2 never loads
`.opencode/tools/`.

## Assignment
- Repo: the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
  (HEAD = post-5b). Stay on the current checkout.
- Assigned worker: `worker_FlashNext`.
- TODO entry: `TODO.md` #137. Design of record (READ-ONLY): the CLONE v1 twin
  `.opencode/tools/block_transfer.ts` (845 lines — the clone's OWN feedback-line
  format is the baseline; read in SECTIONS, never all at once: header 1-140,
  the mode resolver ~200-540, the execute/return assembly ~540-845).

## Definition of done (measurable)
1. NEW plugin `.opencode/plugins/block_transfer.ts` (id `block_transfer`)
   registering the tool via `await ctx.tool.transform((editor) =>
   editor.add(blockTransferTool))` (5a/4a pattern; the input schema mirrors the
   v1 args: `mode`, `srcFile`/`dstFile`, `startMarker`/`endMarker`/`targetMarker`,
   `refs[]`, `text`, `regions[{start,end}]`, `from`/`count`, `bufferName`).
2. Behavior carried BYTE-COMPATIBLE against the CLONE v1 twin: the MODES set
   (MOVE/COPY/APPEND/CUT/PASTE/REPLACE/WRITE/PEEK/MAP/DELETE/CLEAR), the anchor
   semantics (line-number refs vs prefix markers, the ALL-DIGIT rule), the buffer
   semantics (`last_write` auto-store, `last_move_source`, CLEAR), the RETURN-LINES
   feedback formats IN THE CLONE'S OWN FORMAT, the teaching errors (non-unique
   anchor with match count + first 3 matches, out-of-range ref, empty buffer,
   overlapping regions list, the WRITE multi-region + one-text REJECT), the
   same-file MOVE-with-shift rule, the extraction-order guarantees (dst checks
   before the source cut), and the WRITE auto-stores-`last_write` leg.
3. Sandbox carried: roots = repo CWD root + config allow roots + the TEMP/TMP
   env override + `os.tmpdir()`, dedupe by resolved lower-cased native form,
   trailing `/**` stripped, config unreadable → EMPTY set (fail-open to today's
   set). Config root source = the clone's `opencode.jsonc` at the plugin's OWN
   repo root: read the ON-DISK shape (measured today: the v1-style
   `"permission": { "external_directory": {...allow...} }` object) AND support the
   v2 native array form `permissions: [{action:"external_directory", resource,
   effect}]` if the maintainer converts the file — support BOTH, never edit the
   config file.
4. SMOKE — new `.opencode/plugins/tests/block_transfer.smoke.mjs` +
   `block_transfer.sandbox.smoke.mjs` (bun or node per the plugins/tests home;
   scratchpad fixture trees; the sandbox twin self-sets its fixture TEMP env,
   mirroring the v1 sandbox smoke's mechanism). Pins: one per teaching error +
   the mode happy paths + the buffer legs + the multi-region REJECT + the
   same-file move-with-shift + the sandbox allow/deny lines (the deny line's
   allowed-roots list format carried). `SMOKE: ALL PASS`.
5. GATE — clone probe `360/360` UNCHANGED; the 12 v1 node smokes ALL PASS incl.
   `plugin/tests/block_transfer.smoke.mjs` 132/132 + `block_transfer.sandbox.smoke.mjs`
   76/76 (they pin the v1 file — DO NOT repurpose); ALL v2 plugin smokes ALL
   PASS; tsgo exit 0 (tsconfig `files` += the plugin); tests README section; clone
   tree clean.

## Verified facts (measured at spec-authoring, 2026-10-22)
- v1 sandbox facts (clone twin): `loadConfigRoots()` at module top (`:113`
  `const CONFIG_ROOTS = loadConfigRoots()` — ONCE at load); roots list
  `[cwd, ...CONFIG_ROOTS, tempOverride, os.tmpdir()]` at `:130-131`
  (`tempOverride = process.env.TEMP ?? process.env.TMP`); the out-of-sandbox
  error line `Error: '<givenPath>' is outside the sandbox (allowed: <roots>)`
  at `:140`; the note that out-of-sandbox is handled upstream by the intercept
  plugin + this stays the backstop at `:202-203`; execute root =
  `context.directory || process.cwd()` at `:523` → in v2: the setup-captured
  `ctx.location.directory` (context_family.ts:497 precedent).
- v2 config permission shape (if/when the file is converted): flat ordered
  `permissions: [{action, resource, effect}]`, last match wins — fork
  `schema/src/permission.ts:71-81` + `core/src/tool.ts:292-295` (`findLast`),
  research doc `agent/research/2026-10-15_opencode-v2-impact.md` §5 table;
  the clone `opencode.jsonc` on disk TODAY still carries the v1 object form
  (measured this session).
- Tool contract + registration + ToolContext facts: as pinned in the 5a spec
  (fork `packages/schema/src/tool.ts:99-106` / `:84-88`,
  `packages/plugin/src/promise/tool.ts:11-24`; clone
  `plugins/compact_memory.ts:693-694`).
- Baselines: clone probe 360/360; v1 bt smokes 132/132 + sandbox 76/76 (clone
  format — MAIN's formats differ, do NOT mix); v2 smokes to date: intercept 52,
  context_family 46, context_trim_v2 65, compact_memory 63, (post-5a/5b) the
  small-tool suites.

## Suggested scope (non-exhaustive)
- NEW: clone `.opencode/plugins/block_transfer.ts`, the two smokes, the tsconfig
  entry, the tests README section.
- Reference READ-ONLY: the clone v1 twin (section reads), 5a/5b's plugin shape,
  MAIN repo (bookkeeping only).

## Approval boundary
Pre-approved class (agent-facing tool port; live v2 acceptance = maintainer
domain). DO NOT touch: MAIN repo (except summary + planner bookkeeping);
`maintainer/**`; clone `opencode.jsonc`, `.opencode/tools/**`, `.opencode/plugin/**`,
`.opencode/plugin-unported-v1/**`, the landed plugins
(`intercept_observer.ts`, `context_family.ts`, `context_trim_v2.ts`,
`compact_memory.ts`, `loop_log.ts`, `submit.ts` + 5b's three), the live DB.

## Procedure (guidance)
1. Read the v1 twin in the 3 named sections; list its mode handlers + error
   strings via grep first.
2. Port in checkpoint units: (i) the core ops + anchors, (ii) the buffer/PEEK/MAP/
   WRITE legs + feedback lines, (iii) the sandbox roots (both config shapes).
   Keep the clone's feedback-line formats byte-exact.
3. Smoke (mirror the v1 pin SETS where they encode behavior, incl. the
   teaching-error line formats), tsconfig, README, full clone gate; summary to
   MAIN `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- The MAIN-side bt hardening items (MAIN feedback format) — the v1-side
  reduction/dual-merge unit (later, MAIN repo).
- Live v2 acceptance (maintainer); `session_info` (native); auto_resume v2
  (order 2, HELD — see the NAP queue note).
