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
- TODO entry: `TODO.md` #137 + the staleness entry #143. Design of record (READ-ONLY):
  the MAIN repo `.opencode/tools/block_transfer.ts` (1034 lines MEASURED 2026-10-22 —
  the CURRENT v1 behavior, incl. the #140/#142 field rules, the WRITE-regions
  N>1+one-text REJECT and the item-7 feedback formats). The CLONE twin
  `.opencode/tools/block_transfer.ts` (845 lines) is STALE (189-line delta — TODO #143)
  and is NOT the baseline. Read the MAIN file in SECTIONS, never all at once:
  header 1-140, the mode/anchor resolver ~200-560, the execute/return assembly
  ~560-1034 (ranges indicative; grep the section markers first).

## Definition of done (measurable)
1. NEW plugin `.opencode/plugins/block_transfer.ts` (id `block_transfer`)
   registering the tool via `await ctx.tool.transform((editor) =>
   editor.add(blockTransferTool))` (5a/4a pattern; the input schema mirrors the
   v1 args: `mode`, `srcFile`/`dstFile`, `startMarker`/`endMarker`/`targetMarker`,
   `refs[]`, `text`, `regions[{start,end}]`, `from`/`count`, `bufferName`).
2. Behavior carried BYTE-COMPATIBLE against the MAIN twin (the current v1
   behavior): the MODES set
   (MOVE/COPY/APPEND/CUT/PASTE/REPLACE/WRITE/PEEK/MAP/DELETE/CLEAR), the anchor
   semantics (line-number refs vs prefix markers, the ALL-DIGIT rule), the buffer
   semantics (`last_write` auto-store, `last_move_source`, CLEAR), the RETURN-LINES
   feedback formats = the MAIN TWIN'S formats (incl. the WRITE
   `old A..B -> new C..D, file now N lines` line and the multi-region REJECT —
   the MAIN twin carries hardening the clone twin LACKS, see Verified facts),
   the teaching errors (non-unique
   anchor with match count + first 3 matches, out-of-range ref, empty buffer,
   overlapping regions list, the WRITE multi-region + one-text REJECT), the
   same-file MOVE-with-shift rule, the extraction-order guarantees (dst checks
   before the source cut), and the WRITE auto-stores-`last_write` leg.
3. Sandbox carried (the MAIN twin's CURRENT form, incl. the hardening items
   the clone twin lacks): roots = repo CWD root + config allow roots + the
   TEMP/TMP env OVERRIDE AHEAD of `os.tmpdir()` (hardening item 10, 2026-10-21 —
   POSIX-safe; Windows keeps the byte-exact 2-root error form since TEMP ==
   os.tmpdir()), dedupe by EXACT string equality on the resolved lower-cased
   native form, trailing `/**` stripped, config unreadable → EMPTY set (fail-open
   to the CWD+temp roots). Config root source = the plugin's OWN repo root
   `opencode.jsonc`: read the ON-DISK shape (measured today: the v1-style
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
- Sandbox facts measured in the MAIN twin (1034 lines @ 2026-10-22): `loadConfigRoots()`
  at module top (`:113` `const CONFIG_ROOTS = loadConfigRoots()` — ONCE at load); roots list
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
- Twin staleness (measured 2026-10-22, `cmp`/`wc -l`/`diff` between the trees):
  MAIN `.opencode/tools/block_transfer.ts` = 1034 lines, clone twin = 845 —
  333 `diff` lines; the clone twin LACKS the MAIN hardening items (item 10
  POSIX temp root at MAIN `:121-133` etc.). `submit.ts`: MAIN 189 vs clone 167
  (the 2026-10-21 newline-guard leg) — same staleness class → TODO #143. The
  5b trio is BYTE-IDENTICAL between the trees (`cmp` clean).
- Baselines: clone probe 360/360; clone v1 bt smokes 132/132 + sandbox 76/76
  (they pin the STALE twin — untouched, DO NOT repurpose); MAIN bt 151/151 +
  sandbox 76/76 (MAIN gate untouched by this unit); v2 smokes to date: intercept
  52, context_family 46, context_trim_v2 65, compact_memory 63, loop_log 29 +
  submit 29 (5a), (post-5b) the read-tool suites.

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
1. Read the MAIN twin in the 3 named sections (grep its section markers + error
   strings first); diff the twins (`diff .opencode/tools/block_transfer.ts
   ../opencode-agent/.opencode/tools/block_transfer.ts | head -60` from the clone)
   to see exactly what the clone twin lacks.
2. Port in checkpoint units: (i) the core ops + anchors, (ii) the buffer/PEEK/MAP/
   WRITE legs + feedback lines, (iii) the sandbox roots (both config shapes).
   Keep the clone's feedback-line formats byte-exact.
3. Smoke (mirror the v1 pin SETS where they encode behavior, incl. the
   teaching-error line formats), tsconfig, README, full clone gate; summary to
   MAIN `agent/handover/handover_task_to_planner.md`.

## Deliberately out of scope
- Refreshing the clone's STALE v1 twin (+ repinning its v1 smokes) — TODO #143,
  planner decision; this unit ports MAIN behavior and leaves the twin untouched.
- Live v2 acceptance (maintainer); `session_info` (native); auto_resume v2
  (order 2, HELD — see the NAP queue note).
