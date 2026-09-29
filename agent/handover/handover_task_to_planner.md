# handover_task_to_planner.md — #117 post-split reorg (WORKER, IN PROGRESS)

Session: ses_f12b5172dffeq1r2iwK4QLlWxj (worker_Q3S_245K_slow). Branch `opencode_test`,
never pushed. Spec: `agent/handover/handover_task.md` (post-move). U1–U3 DONE + committed;
U4–U5 REMAIN. After compaction: re-read spec + this file, continue at U4.

## Done (each = green checkpoint commit)
- **U1** `aafaab4` — five folders moved `.opencode/{agent,archive,loop,maintainer,proposals}` →
  root as git R-renames (713 R + 1 RM). `git mv` itself is broken on this host
  (ENOENT/EPERM despite existing sources) — plain `mv` + `git add` gives identical
  staged R-renames; use that if more moves are needed. `.opencode/` now = the 7 native
  entries only.
- **U2** `666e4d4` — code re-pointed: `tools/{submit,loop_log,ctx_gauge}.ts`,
  `plugin/{auto_resume,ctx_watchdog,intercept_observer,compact_memory,compaction_core}.ts`,
  `plugin/deactivated/*` (4), 5 smokes, probe. Key findings:
  - `intercept_observer.ts` MAP_PATH needed a SECOND `..` (`join(THIS_DIR, "..", "..",
    "agent", ...)`): one `..` from `.opencode/plugin` lands in `.opencode/`, not root —
    without it the numword pair gate silently went off (28 probe hook checks + io smoke
    failed). Fixed.
  - auto_resume.smoke needed its 3 `".opencode", "loop"` join-form fixture refs updated
    (the slash-form grep missed join forms).
  - probe sandbox needed an explicit `mkdirSync(SANDBOX/.opencode)` (the old handover
    mkdir created it as a side-effect).
  - Gate after U2: **probe 335/346** (the 11 fails = 136–146 python w2n = NO `.venv` in
    this repo, the known #113 env break, maintainer call — IDENTICAL fail set to the
    pre-U1 baseline worktree at 8a97e86, PASS sets byte-identical) + **10/10 smokes
    green** (auto_resume 146, bt 131, bt-sandbox 64, compact_memory 78, io 78, loop_log
    69, context_recovery 17, ctx_gauge 3, gauge_core ALL, submit 23).
- **U3** `86fb413` — 170+ old-path refs updated in active docs: `agent/prompts/**`,
  `agent/knowledge/**`, `agent/memory/**`, `agent/orientation.md`, `agent/scripts/`
  READMEs + `dump_session.cjs`/`summarize_intercept.cjs` + fixture smoke (ROOT 5→4
  `..`, green) + `expected_summary.txt`, `agent/handover/handover_task.md` (spec's own
  old-path lines reworded gate-clean), `.opencode/plugin/{intercept_observer,
  ctx_watchdog,auto_resume}.md` + `tests/README.md`, `.opencode/tools/{submit,loop_log}.md`,
  root `TODO.md` open entries (the planner marker-sweep command also gained the new
  root folders). **Deliberately untouched:** TODO.md L747 (closed #107 line).
- **cleanup** `a196e5b` — stray `__pycache__` out of the U3 commit + root `.gitignore`
  (pyc/pycache) created.

## U4 — PER-PROJECT (NEXT)
Build `projects/`:
1. `projects/README.md` (≤20 lines) + `projects/Free-Snap-Tap/README.md` (≤10 lines).
2. `projects/Free-Snap-Tap/TODO.md` ← the FST entries moved OUT of root `TODO.md`
   (1015 lines; entry header map already taken — `rg -n "^## " TODO.md`). FST entries
   (16 total, classified): FST behavior (section "FST behavior decisions" L23-43):
   #1, #7, #8, #9, #4, #6, #11, #48; FST docs (section "Docs & misc"): #3, #45, #46, #47;
   FST code/tests (section "Closed entries" L172+): #41, #42, #43, #44. The mixed
   sections also hold AGENT entries that STAY at root: #40, #50, #64 (Docs & misc),
   #34, #36, #39 (Closed entries), plus all Loop/Plugin/other sections. READ ONLY the
   ranges you move (never whole-file). Root TODO.md keeps: agent entries,
   the global counter (header — up to #118, next #119), + ONE pointer line
   "FST project entries: `projects/Free-Snap-Tap/TODO.md`". Project TODO header gets
   "IDs continue the global sequence — counter in root `TODO.md`".
3. `projects/Free-Snap-Tap/todo_records.md` ← the CLOSED FST records moved out of root
   `todo_records.md` (2064 lines) — map headers first (`rg -n "^## " todo_records.md |
   head -80`), read ranges only.
4. `projects/Free-Snap-Tap/repo/` ← `agent/prompts/repo/{repo_map,repo_testgate,
   repo_gotchas}.md` + the FST half of `repo_commands.md` (split the file: FST
   product code/venv/pytest lines vs agent-repo gate lines). Then REWRITE
   `agent/prompts/repo/repo_overview.md` for THIS repo (agent-repo map, parts index,
   pointer to `projects/Free-Snap-Tap/repo/`). `repo_commands.md`/`repo_custom_tools.md`
   stay in `agent/prompts/repo/` (agent-repo parts).
   ONE green commit for U4.

## U5 — STAGE + VERIFY (LAST)
1. Stage complete replacements at root (copy live file, edit the COPY):
   - `opencode_reorg_2026-09-29.jsonc` from `opencode.jsonc`: `{file:.` pointers
     `.opencode/agent/…` → `agent/…`; worker/explorer permission paths
     `.opencode/agent/prompts/**` → `agent/prompts/**`; `.opencode/agent/handover/
     handover_planner.md` → `agent/handover/handover_planner.md`; commented
     looprunner block + `destilled_mem` pointers likewise. Diff = path changes only.
   - `AGENTS_reorg_2026-09-29.md` from `AGENTS.md` (~7 path refs).
   NEVER edit the live `opencode.jsonc` / `AGENTS.md`.
2. Grep gate: `rg --hidden -l -e "\.opencode/(agent|loop|archive|maintainer|proposals)"`
   → zero hits in `.opencode/plugin/*.ts`, `.opencode/tools/*.ts`,
   `.opencode/plugin/*.md`, `.opencode/tools/*.md`, `agent/prompts/**`,
   `agent/knowledge/**`, `agent/memory/**`, `agent/orientation.md`,
   `agent/scripts/{README,INVENTORY}.md`, `agent/handover/handover_task*.md`,
   TODO.md open entries. Allow-list (old paths MAY survive): `todo_records.md`,
   `todo_inbox.md`, `proposals/**`, `archive/**`, `agent/handover/specs/**`,
   `agent/research/**`, `agent/handover/handover_planner.md`.
3. Re-run gate: probe (expect 335/346 + the 11 #113 env fails) + 10 smokes (expect all
   green) + `node agent/scripts/log/tests/summarize_intercept.smoke.cjs`.
4. `TODO.md` #117 status → LANDED (hash goes in the planner's bookkeeping commit) +
   this file's FINAL version → ONE final commit. loop_log DONE line (final gauge
   readout, verbatim) + friction `submit` before stopping.

## Flags / deliberate non-edits (report in final handover)
- Historical append-only records LEFT with old paths (not on the gate allow-list —
  planner call): `agent/agent_feedback.md`, `agent/agent_ideas.md`,
  `agent/handover/handover_draft_worker_*.md`, `playground/outline_rework_prompts.md`
  (DO-NOT-TOUCH zone). TODO.md closed #107 line (L747) left per spec.
- DEFERRED TODO entries (#56, #86) were updated (treated as open — not closed records).
- #113 env break: no `.venv` in this repo (NAP says broken venv; measured: absent) —
  the 11 python probe fails + pytest unrunnable; baseline==post-change verified.
- Live opencode process still runs pre-move plugin code — live-tool behavior may show
  old paths; verification is on-disk (done).
