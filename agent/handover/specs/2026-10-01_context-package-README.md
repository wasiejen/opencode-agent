# Spec wave — context budget package (2026-10-01)

Design source for ALL specs: `proposals/approved/2026-10-01_context-budget-package.md`
(approved by the maintainer, commit 4b4c34e).

Queue order (each spec leaves the repo green; launch from these committed
files — copy the active spec to `agent/handover/handover_task.md` + the
loop folder before each launch):

| # | spec | worker | status |
|---|------|--------|--------|
| U0 | `2026-10-01_context-package-U0-keep-N-surface.md` | worker_Q3S | LANDED 2026-10-01 (0538aa4 — smoke 34/34, probe 355/355, all 11 smokes; planner-verified spot re-run under node) |
| U1 | `2026-10-01_context-package-U1-recommendation-leg.md` | worker_Q3S | LANDED 2026-10-01 (9f1f668 — smoke 40/40, probe 355/355, all 11 smokes under node; surface report = cut-down form by default, full form via core export — decision recorded) |
| U2 | `2026-10-01_context-package-U2-fork-filter.md` | worker_Q3S_slow | COMMITTED 2026-10-01 (effort L — the long pole; LIVE gate = the maintainer's `build_fork.bat` run) |
| U3 | `2026-10-01_context-package-U3-marker-compaction.md` | worker_Q3S | LANDED 2026-10-01 (5c1e286/40e008f — compact_memory 110/110, gate green under node; worker limit-death after final commit — no handover (planner reconstructed from files), probe 86 re-pinned by the planner) |
| U4 | `2026-10-01_context-package-U4-nudge-policy.md` | worker_Q3S | LANDED 2026-10-01 (4e2b901 — auto_resume 165/165, gate green under node) + prompt-ladder reduction done by the planner (planner + worker prompts + AGENTS_pending note); the `nudge1-4` keys are NOT enabled in the live config (absent = off — enabling is the maintainer's call) |

Notes: U1 must land before U4 (the triage line carries the U1 recommendation
one-liner). U2's fork work lives in `C:\Users\Wasiejen\Repos\opencode`;
agent-repo changes are the plugin surface + `build_fork.bat` only.
Adaptive-keep is PARKED (proposal) — no spec.
