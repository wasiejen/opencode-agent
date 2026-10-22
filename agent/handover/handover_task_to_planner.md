# Worker summary — TODO #137 unit 5c: block_transfer ported to v2 (clone `migration`)

Status: COMPLETE

## Task and scope
Ported `block_transfer` to v2 as a plugin-registered tool in the CLONE
`C:/Users/Wasiejen/Repos/opencode-agent-v2` (branch `migration`). The design of
record = the MAIN repo `.opencode/tools/block_transfer.ts` (1034 lines, the
current v1 behavior incl. the #140/#142 hardening). The CLONE's stale v1 twin
(845 lines, TODO #143) was NOT touched and NOT the baseline. The MAIN repo
itself was untouched except this summary.

## Files changed (CLONE, commit `1b0ebdb`)
- NEW `.opencode/plugins/block_transfer.ts` (1276 lines) — id `block_transfer`,
  registered via `await ctx.tool.transform((editor) => editor.add(blockTransferTool))`
  (the compact_memory shape); teardown disposes.
- NEW `.opencode/plugins/tests/block_transfer.smoke.mjs` (90 checks).
- NEW `.opencode/plugins/tests/block_transfer.sandbox.smoke.mjs` (23 checks).
- `.opencode/plugins/tests/tsconfig.json` — `files` += `../block_transfer.ts`.
- `.opencode/plugins/tests/README.md` — the unit-5c section.

## Behavior carried (byte-compatible vs the MAIN twin)
The helper region, the description template literal, and the execute body were
sliced VERBATIM from the MAIN twin by a generation script (the reader truncates
the 9298-char description line; transcription-by-hand would have broken byte-
identity) — same MODES set + no-default-mode guard, unified anchor rule + Closest-
lines hint, ALL-DIGIT line-number refs, buffer semantics (`last_write` auto-store,
`last_move_source`, COPY-replaces/APPEND-appends/PASTE-REPLACE-preserve, no-partial-
state-on-rejection), all feedback-line formats (opLine/assemblyFeedback, the WRITE
`old A..B -> new C..D, file now N line(s)` group, insertion suffixes), the
multi-region + one-text REJECT, the overlap REJECT, same-file MOVE-with-shift
(above/below/INSIDE legs), dst-checks-before-cut ordering, and the EOL/EOF
hardening (items 3/4/5/6/7/8/10). v2 surface deviations only: JSON-schema input
(required `[mode]`, refs as anyOf string|integer, every describe() text VERBATIM
— 12/12 verified consumed by the generator), `{output: string}` return,
setup-captured `ctx.location.directory` root (fallback `process.cwd()`), and
`loadConfigRoots()` extended to support BOTH config shapes: the on-disk v1
object form UNCHANGED + the v2 native array form `permissions:[{action,resource,
effect}]` with LAST MATCH WINS (fork findLast), /** stripped, cross-shape dedupe
— the config file itself NEVER edited.

## One deliberate code delta (flagged)
`let insertAfter: number = 0;` in the MOVE block — v1 left it UNINITIALIZED;
tsgo (which never checks the v1 tools files) demands definite assignment.
Typecheck-only, behavior identical (every sameFile path assigns before use).
Noted in the file comment + commit body.

## Measured verification (all at the CLONE, post-commit state)
- `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360
  PASS` UNCHANGED — PASS.
- 12 v1 node smokes (`node .opencode/plugin/tests/*.smoke.mjs`) ALL PASS, incl.
  `block_transfer.smoke.mjs` 132/132 + `block_transfer.sandbox.smoke.mjs`
  76/76 pinning the STALE twin UNCHANGED — PASS.
- ALL 11 v2 plugin smokes (`bun run <f>.smoke.mjs` from `.opencode/plugins/tests`)
  final line `SMOKE: ALL PASS` — incl. the new 90-check + 23-check twins — PASS.
- `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p .opencode/
  plugins/tests/tsconfig.json` → exit 0 — PASS.
- Clone `git status` clean at `1b0ebdb`; HEAD chain: 5434a93 → 1b0ebdb.
- Exact verify commands + results in README section; one-shot re-run of every
  suite against the committed tree (no EBUSY flakes seen this run — TODO #144
  untouched).

## Inbox/TODO
No new TODO entries. Findings submitted at close: 1 friction entry (block_transfer
relative paths resolve against the SESSION CWD — cross-repo needs absolute paths
— `agent/agent_feedback.md`), 1 knowledge entry (deny-matrix fixtures must sit
OUTSIDE the scratchpad config root; fixture-repo plugin copies read their OWN
config — `agent/knowledge/knowledge_inbox.md`), 1 memory entry (byte-exact pin
discipline — `agent/memory/worker/memory_inbox.md`). Loop-log DONE line written.

## Deliberately omitted (per spec)
- Refreshing the clone's stale v1 twin + repinning its 132+76 v1 smokes — TODO
  #143 (planner decision).
- Live v2 acceptance (maintainer domain); `session_info` (native); auto_resume
  v2 (HELD queue item).

## Blockers
None.

## Lesson
Verbatim-slice generation (script assembling the new file from the design-of-
record file) beats hand-transcription when the source has lines the reader
truncates — and the extractor's count asserts (12 describes, anchor lines) caught
every drift attempt.
