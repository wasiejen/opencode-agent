# Spec wave — context budget package (2026-10-01)

Design source for ALL specs: `proposals/approved/2026-10-01_context-budget-package.md`
(approved by the maintainer, commit 4b4c34e).

Queue order (each spec leaves the repo green; launch from these committed
files — copy the active spec to `agent/handover/handover_task.md` + the
loop folder before each launch):

| # | spec | worker | status |
|---|------|--------|--------|
| U0 | `2026-10-01_context-package-U0-keep-N-surface.md` | worker_Q3S | COMMITTED 2026-10-01 |
| U1 | `2026-10-01_context-package-U1-recommendation-leg.md` | worker_Q3S | COMMITTED 2026-10-01 |
| U2 | `2026-10-01_context-package-U2-fork-filter.md` | worker_Q3S_slow | COMMITTED 2026-10-01 (effort L — the long pole; LIVE gate = the maintainer's `build_fork.bat` run) |
| U3 | — pending (write after U2 lands; design = the approved proposal §U3: marker compaction, cap backstop, worker_Q3S) |
| U4 | — pending (write after U1; design = the approved proposal §U4: nudge1-4 keys + recommendation one-liner, worker_Q3S) — NOTE: the prompt-ladder reduction part of U4 is the PLANNER'S to edit directly (planner HAS agent/prompts/** access; only .git/.github denied) |

Notes: U1 must land before U4 (the triage line carries the U1 recommendation
one-liner). U2's fork work lives in `C:\Users\Wasiejen\Repos\opencode`;
agent-repo changes are the plugin surface + `build_fork.bat` only.
Adaptive-keep is PARKED (proposal) — no spec.
