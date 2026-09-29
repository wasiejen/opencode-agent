# handover_task_to_planner.md — #117 post-split reorg (WORKER, DONE)

Session: ses_f12b5172dffeq1r2iwK4QLlWxj (worker_Q3S_245K_slow). Branch `opencode_test`,
never pushed. All five units LANDED; one self-compaction mid-run (U4 resumed from this
file's IN-PROGRESS version, committed 35a50ef).

## What changed (commits in order)
- **U1** `aafaab4` — the five bookkeeping folders moved from their `.opencode/`
  prefix to the root (`agent/`, `archive/`, `loop/`, `maintainer/`, `proposals/`)
  as git R-renames (713 R + 1 RM). `git mv` itself is broken on this host
  (ENOENT/EPERM despite existing sources) — plain `mv` + `git add` gives identical
  staged R-renames. `.opencode/` now holds the seven native entries only.
- **U2** `666e4d4` — code re-pointed: `tools/{submit,loop_log,ctx_gauge}.ts`,
  `plugin/{auto_resume,ctx_watchdog,intercept_observer,compact_memory,compaction_core}.ts`,
  `plugin/deactivated/*` (4), the 5 affected smokes, the probe. Key findings:
  - `intercept_observer.ts` MAP_PATH needed a SECOND `..` (`join(THIS_DIR, "..", "..",
    "agent", ...)`): one `..` from `.opencode/plugin` lands in `.opencode/`, not the
    root — without it the numword pair gate silently went off (28 probe hook checks
    + the io smoke failed). Fixed.
  - auto_resume.smoke needed its 3 join-form loop-fixture refs updated (the
    slash-form grep missed join forms).
  - the probe sandbox needed an explicit `mkdirSync` of its `.opencode` dir (the
    old handover fixture created it as a side-effect).
- **U3** `86fb413` — 170+ old-path refs updated in active docs: `agent/prompts/**`,
  `agent/knowledge/**`, `agent/memory/**`, `agent/orientation.md`, the
  `agent/scripts/` READMEs + `dump_session.cjs`/`summarize_intercept.cjs` + fixture
  smoke (ROOT 5→4 `..`, green) + `expected_summary.txt`, the spec's own old-path
  lines (reworded gate-clean), the plugin/tool `.md` docs, root `TODO.md` open
  entries, + the planner marker-sweep command gained the new root folders.
  Deliberately untouched: root `TODO.md` closed #107 line.
- **cleanup** `a196e5b` — stray `__pycache__` out of the U3 commit + root
  `.gitignore` (pyc/pycache) created.
- **U4** `68b603a` — per-project split: 17 FST entries → `projects/Free-Snap-Tap/
  TODO.md` (behavior / code & tests / docs / environment); 21 closed records
  (incl. both #47 blocks + the iter-9 wrapper with its 6 full-text sub-entries
  + #11) → `projects/Free-Snap-Tap/todo_records.md`; `repo_map.md` /
  `repo_testgate.md` / `repo_gotchas.md` → `projects/Free-Snap-Tap/repo/`;
  `repo_commands.md` split (FST venv/pytest half vs agent-repo gate half);
  `repo_overview.md` rewritten for the agent repo; `projects/README.md` +
  project README built. Root `TODO.md` keeps the global counter + a pointer line;
  root `todo_records.md` keeps the agent records (+ its stale #59 counter note
  corrected).
- **U5** (this commit + the staged-files commit) — staged replacements at root:
  `opencode_reorg_2026-09-29.jsonc` (42 path refs, 38 lines changed, diff =
  path-only, verified) + `AGENTS_reorg_2026-09-29.md` (7 refs, 7 lines). Live
  `opencode.jsonc` / `AGENTS.md` NEVER touched — the maintainer applies the
  staged copies.

## Measured verification (final state)
- **Grep gate** (`rg --hidden -l` on the old-path pattern): 332 hit files (final
  count, after this file itself became gate-clean), ALL
  classified: the spec allow-list (root `todo_records.md`, `todo_inbox.md`,
  `proposals/**`, `archive/**`, `agent/handover/specs/**`, `agent/research/**`,
  `agent/handover/handover_planner.md`) + the planner-extended allow-list
  (`agent/agent_feedback.md`, `agent/agent_ideas.md`, `agent/handover/
  handover_draft_*.md`, `playground/**`) + maintainer-domain files
  (`maintainer/**` — never edited per DO-NOT-TOUCH) + the pre-U1 looprun docs in
  the live loop folder + the live `opencode.jsonc` / `AGENTS.md` (pending the
  maintainer's apply of the staged copies) + `projects/Free-Snap-Tap/
  todo_records.md` (historical records, same status as the root file) + the
  closed #107 line in root `TODO.md` (left per spec). Zero hits in every
  spec-enumerated active file.
- **Standard gate:** probe 335/346 (the 11 fails = checks 136–146 python w2n =
  no `.venv` in this repo, the known #113 env break — IDENTICAL fail set to the
  pre-U1 baseline worktree at 8a97e86, PASS sets byte-identical) + 10/10 smokes
  green (auto_resume 146, block_transfer 131, bt-sandbox 64, compact_memory 78,
  intercept_observer 78, loop_log 69, context_recovery 17, ctx_gauge 3,
  gauge_core ALL, submit 23) + the summarize_intercept cjs smoke green.

## TODO entries
- None filed. #117 → LANDED in this final commit (its hash is for the planner's
  follow-up bookkeeping commit).

## Deliberately not done / flags
- Live `opencode.jsonc` + `AGENTS.md` untouched (staged copies only — his apply +
  restart is his tail).
- Historical append-only records left with old paths per the extended allow-list
  (ruling 2026-09-29): feedback/ideas/draft/playground files; root `TODO.md`
  closed #107 line.
- The pre-U1 looprun docs in the live loop folder (plan14/plan15/plan19) still
  reference the old paths — looprun records, not agent-editable.
- `projects/Free-Snap-Tap/todo_records.md` carries old paths in its records —
  same historical-records status as the root file (the spec allow-list predates
  the split; noted here for the planner's call if strictness matters).
- DEFERRED root TODO entries (#56, #86) were path-updated in U3 (treated as
  open — not closed records).
- #113 env break stands: no `.venv` in this repo (the 11 python probe fails +
  pytest unrunnable) — baseline==post-change verified.
- The live opencode process still runs pre-move plugin code — live-tool
  behavior may show old paths until his restart; all verification is on-disk.
  Measured at close-down: the live `submit` + `loop_log` tools (pre-move code)
  wrote my friction entry + DONE line into RECREATED old-path folders
  (`.opencode/agent/agent_feedback.md`, `.opencode/loop/autorun-2026-09-29_16-48/`)
  — both entries relocated to the correct new files and the stray dirs removed,
  so `.opencode/` is back to the seven native entries. Until his restart, any
  live tool writes may land at the old paths again (his to re-verify).
