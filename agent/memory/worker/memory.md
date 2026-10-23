# Worker memory — full entries

Cured from `memory_inbox.md` by the planner's maintenance passes. Format per
`agent/memory/README.md`. Ids are this role's own namespace (WM-xx).

## WM-01 — In a buffered CLI write transaction, COMMIT is the flush TRIGGER, not a buffer member
- Type: observed; confidence: high; scope: spawn/CLI-backed DB write paths (the #126 spawn-sqlite3 adapter pattern).
- Memory: when buffering a write transaction into one CLI flush, explicitly
  append the terminating `COMMIT` to the flushed script — the CLI exits 0, an
  uncommitted batch rolls back at process exit, and the tool still prints its
  success line.
- Evidence: memory_inbox 2026-09-30_16-46; TODO #126 worker session; the smoke
  pin "the part row carries the new tail_start_id (spawn path)" failed until
  COMMIT was added.
- Review when: any future spawn/CLI-backed write path.

## WM-02 — `fs.rmSync` on a DIRECTORY needs `{recursive:true, force:true}`
- Type: learned; confidence: high; scope: test-fixture cleanup in
  `.opencode/plugin/tests/*.smoke.mjs` on this Windows/node host.
- Memory: `{force:true}` alone throws `ERR_FS_EISDIR` on a directory (even an
  empty one); add `recursive:true`.
- Evidence: memory_inbox 2026-10-01_04-09; auto_resume smoke cleanup line 1150
  (SystemError, then 152/152 PASS after the fix).
- Review when: never expected.

## WM-03 — pytest print→logger conversions: capsys CANNOT see the records
- Type: learned; confidence: high; scope: any FST logging refactor.
- Memory: during pytest, the root LogCaptureHandler intercepts records — use
  `caplog`/`caplog.at_level(logger=...)` pins instead of capsys. BEFORE
  starting the build, grep the suite for (a) `caplog.at_level` pins naming the
  old loggers and (b) capsys assertions on the exact texts being converted —
  both classes break silently.
- Evidence: memory_inbox 2026-10-05_02-34; the #131 build (test_extraction
  filter_edges.py:176 + test_output_manager.py:607 missing from the spec's
  list); FST commits ea2aac3/cac6704/cd0f3a2.
- Review when: next print→logging conversion task.

## WM-04 — Tool-call JSON payloads fail above ~2 KB on this host — batch payloads under ~1.8 KB
- Type: observed; confidence: high; scope: every multi-file/multi-edit content
  payload on this host.
- Memory: calls over ~2 KB fail with "JSON Parse error: Unterminated string"
  (a ~4 KB write + a ~3 KB heredoc); every call under ~1.8 KB succeeded.
  Split big content into batches (heredoc appends + one splice script).
  Re-pin byte-exact feedback pins with a fail-loud replacement script (one
  `MISS(n)` report per pair), not many sequential edits.
- Evidence: memory_inbox 2026-10-21_21-59; repo_commands #74 advises the same.
- Review when: short payloads start failing again (re-test the length claim).

## WM-05 — Cross-tree diff pitfalls: `git -C` for other repos; process-substitution cmp is unreliable on Windows
- Type: observed; confidence: high; scope: any clone/second-tree port task.
- Memory: (1) `git show HEAD~1:path` runs against the CURRENT tree's HEAD —
  for another repo use `git -C <other-repo> show <ref>:path` into a temp file;
  (2) `cmp -s file <(cmd)` process substitution in git-bash on Windows yields
  false DIFFERS — write both sides to real files first.
- Evidence: memory_inbox 2026-10-22_01-34; both nearly corrupted a port.
- Review when: any clone/second-tree port task.

## WM-06 — File tools resolve RELATIVE paths against the SESSION CWD, never the task's target clone
- Type: observed; confidence: high; scope: cross-repo/clone work.
- Memory: pass ABSOLUTE paths for clone files; a wrong-path write lands in the
  main (forbidden) repo and must be caught via `git status` before the
  checkpoint.
- Evidence: memory_inbox 2026-10-22_04-07; gauge unit 2, recovered via
  `git checkout`.
- Review when: any task whose file tools target a second tree.

## WM-07 — Before a tool-contract change, grep the byte-exact message strings across EVERY pin file in BOTH trees
- Type: observed; confidence: high; scope: opencode-agent + opencode-agent-v2
  tool-contract work.
- Memory: probe checks pin the same behavior the smokes do — a contract change
  forces ONE named repin per tree. Twin rule: keep the reject MESSAGE byte-
  identical across trees but copy each tree's OWN feedback format (clone
  WRITE line `(lines 2..3, first: 'X')`; MAIN hardening item 7 `(old 2..3 ->
  new 2..2, file now N lines, first: 'X')`).
- Evidence: memory_inbox 2026-10-22_04-28; commits 05df70f/b713585 (MAIN),
  2a3759f/4054a0a (clone); probe FAIL at 307 before repin.
- Review when: any tool message or span-format change.

## WM-08 — Derive pinned numbers from the DOCUMENTED rule, never copy measured output into a smoke
- Type: observed; confidence: high; scope: any ported report/readout with
  byte-exact expectation pins.
- Memory: enumerate the fixture masses, then apply the rule to derive the
  slice/expectation — a copied measurement PINS a bug in place (the unit-4b
  dry-run hardcoded `verdict=ok` pinned the real bug: a floor-rejected target
  printed `verdict=ok`).
- Evidence: memory_inbox 2026-10-22_08-18; the keep-mass sum mismatch found
  only by hand re-derivation.
- Review when: a smoke expectation is written from stdout.

## WM-09 — v2 plugin smokes: EVERY fixture workspace needs its OWN `plugin.setup(makeCtx(dir))`
- Type: observed; confidence: high; scope: v2 plugin smokes in the
  opencode-agent-v2 clone.
- Memory: the plugin's repo-root carrier is captured at SETUP (`dir =
  ctx.location.directory`) — the imported module is SHARED, so dir retargets
  per setup call; each teardown disposes only its own registration — count
  dispose deltas PER teardown call, not globally.
- Evidence: memory_inbox 2026-10-22_11-01; loop_log/submit smokes (be5a278/a5da17b).
- Review when: any future v2 smoke (5b/5c-era ports).

## WM-10 — A v2 smoke can steer the SHARED gauge core directly — relative `.mjs` import and `pathToFileURL` resolve to ONE module instance
- Type: observed; confidence: high; scope: clone v2 plugin smokes.
- Memory: call `setDbPath`/`setBudgetFileForTest` on the gauge module from the
  smoke — no plugin-side test hook needed (the plugin's relative `.mjs`
  import and the smoke's absolute `pathToFileURL` import hit the SAME instance).
- Evidence: memory_inbox 2026-10-22_14-03; ctx_gauge.smoke.mjs 11 pins +
  context_family.smoke.mjs:114-117 precedent.
- Review when: the gauge core gains a per-plugin path override.

## WM-11 — Byte-exact smoke pinning: derive expected strings from the code's format functions; fixtures must satisfy the tool's own anchor-uniqueness rule
- Type: observed; confidence: high; scope: writing any new byte-exact smoke.
- Memory: read the tool's format functions FIRST and compose the expected
  feedback strings from them (never hand-copy a guess); design fixture lines
  to satisfy the tool's anchor-uniqueness rule (a short prefix anchor like
  `X` matches the longer twins `X1..X5`); remember new-file writes have NO
  trailing newline while existing files keep their own EOL.
- Evidence: memory_inbox 2026-10-22_14-55; unit 5c — both legs mispinned 4
  times before re-running (clone `.opencode/plugins/tests/block_transfer.smoke.mjs`).
- Review when: a smoke harness change lands (e.g. the EBUSY teardown fix).

## WM-12 — An ordering/behavior pin must be proven DISCRIMINATING before the implementation lands
- Type: learned; confidence: high; scope: any worker adding a pin for a reordering/behavior change.
- Memory: run the new pin against the UNCHANGED tool FIRST and require it to
  FAIL; a fixture whose raw emit order already matches the post-change order
  gives a VACUOUS pin that passes pre-change.
- Evidence: memory_inbox 2026-10-22_18-41; the spec-suggested
  `maintainer/live_note.md` state_snapshot fixture passed 21/21 pre-change
  (grep emits in path-argument order); switching to a `WIKI.md` fixture gave
  the measured FAIL-then-PASS sequence.
- Review when: pin-run mechanics change (see planner MEM-0131 for the spec side).

