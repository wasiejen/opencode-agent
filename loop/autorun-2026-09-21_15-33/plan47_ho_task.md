# Task spec — plan47: TODO #86 deferred audit (read-only, explorer)

## Goal
Read-only audit of ALL agent plugin / tool / script code: stale hardcoded
IDs, duplicated config, magic numbers, dead code, stale references.
Deliverable = a prioritized findings list. **Zero code changes.**

Worker: explorer_Q3S (170K window).
Branch: stay on the current checkout (opencode_test). Never switch branches.

## Verified facts (measured at spec time — do NOT re-derive)
- Live agent roster (opencode.jsonc, active only): planner_Q3S, worker_Q3S,
  explorer_Q3S, planner_Q3S_slow, worker_Q3S_slow, explorer_Q3S_slow,
  agent_gemma_Q4. All other agent blocks are commented out (looprunner_Q3S,
  agent_Q3S, worker_Q3S_HQKV, planner_Q3XS, worker_Q3XS, worker_gemma_Q4) —
  an UNCOMMENTED reference to any of them in code = stale.
- Live model ids (llama-swap provider): Qwen3.8-27B-Q3S-160K-MTP-Thireus,
  -140K-HQKV, -170K, -245K-slow, -210K-slow-HQKV. Note: NO live agent name
  carries a numeric window suffix — so any agent-id string matching
  `Q3S_[0-9]+K` (e.g. "worker_Q3S_170K", "planner_Q3S_160K") in non-test
  code is stale against the roster.
- Scope files (measured sizes):
  - .opencode/plugin/: auto_resume.ts (93K), intercept_observer.ts (76K),
    intercept_observer_core.ts (63K), ctx_watchdog.ts (38K),
    compact_memory.ts (33K), compaction_core.ts (31K),
    context_recovery.ts (13K)
  - .opencode/tools/: block_transfer.ts (48K), context_trim.ts (25K),
    loop_log.ts (13K), submit.ts (12K), ctx_gauge.ts (3K),
    dev_get_tool_context_contents.ts (2K), session_info.ts (0.3K)
  - agent/scripts/: db/ (6 .cjs), log/ (2 .cjs + tests/), binary/ (3 .cjs),
    numword/ (numword.cjs, w2n.py, numwords.json)
  - .opencode/plugin/tests/*.smoke.mjs + .opencode/plugin/probes/
    handover_probe.mjs: audit for stale hardcoded VALUES in assertions /
    comments only (stale fixture ses_* ids are NOT findings — fixture ids
    are intentional). Do NOT audit test logic.
- EXCLUSION per #86: the `PLANNER_AGENT_ID` case in auto_resume.ts was
  already handled by #85 part 2 — do not report it.

## Audit dimensions (one finding = severity + file + line + one-line fix)
1. **Stale hardcoded agent / model ids** not matching the live roster
   (the verified-facts list above). Severity high if it feeds a live
   runtime call (spawn/inject/registration), low if doc/comment only.
2. **Stale hardcoded session ids** (ses_*) in NON-TEST code.
3. **Duplicated config**: the same value defined in >1 file where a single
   source of truth already exists (compact_budget.json keys, context
   window sizes, thresholds). Report each duplicate pair.
4. **Magic numbers** without a named constant or config reference
   (thresholds, timeouts, budgets, sizes) — report location + what it
   encodes + whether a config source already covers it.
5. **Dead code**: exported-but-unused symbols, unreachable branches,
   constants referenced only by their own definition (verify with a
   bounded grep for each candidate before reporting).
6. **Stale doc / path references** in code comments (e.g. pre-#118
   `agent_readme_*` filenames — the plan46 stale-ref cluster hit exactly
   this class).

## Method (output discipline — context is the scarce resource)
- Grep first, per dimension, across the scope with caps, e.g.
  `grep -nE "Q3S_[0-9]+K|160K|170K" .opencode/plugin/*.ts .opencode/tools/*.ts | head -30`,
  then narrow per file.
- Read whole files only when < 400 lines (most tools/ + scripts/ qualify).
  For the big plugin files: grep by dimension + read ±15-line windows
  around each hit only.
- No gate runs (nothing changes). Do not edit ANY repo file except the two
  deliverables.

## Definition of done
1. Findings appended to `todo_inbox.md` via the `submit` tool (`todo`
   channel) — ONE entry per finding cluster (stale-ids / duplicated-config /
   magic-numbers / dead-code / stale-refs), each entry self-contained
   (file + line + suggested fix per finding; clusters may be multi-line).
2. `agent/handover/handover_task_to_planner.md`: executive summary — count
   per dimension, the top ~10 findings inline, the full list referenced to
   the todo_inbox entries, plus what was deliberately not done.
3. `git status`: ONLY the handover file (+ todo_inbox.md) modified; zero
   code changes.
4. ONE final commit carrying the handover + todo_inbox (no code units to
   checkpoint).

## DO-NOT-TOUCH
Anything under `maintainer/`; `opencode.jsonc` (read-only reference);
`.opencode/archive/`; `projects/`; any `--wip` file.
