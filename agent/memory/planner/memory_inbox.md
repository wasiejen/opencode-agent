

### 2026-10-15 curation (direct ses_ec048b57affeZzCMG1NTRZFluG, prompt-adaptation session)
Inbox entries cured this session: 2026-09-30_04-49 (observer-feedback
delivery+apply claims) → MEM-0101 addendum (extended); 2026-09-30_05-40
(hint line not the apply authority) → MEM-0111 (new); 2026-09-30_06-50
(stale-reference sweep vs actual listing) → MEM-0112 (new). Destill lines
updated in the same commit. The six remaining uncured entries stay for the
next maintenance pass.
### 2026-10-21 (autorun planner-12 maintenance pass) curation
Entries cured this session → `memory.md` (full entries) + destill lines in the
new "Spec authoring + host-fact triggers" section: 2026-09-30_16-57 → MEM-0117;
2026-09-30_16-58 → MEM-0118; 2026-10-01_05-02 → MEM-0113; 2026-10-01_05-26 →
MEM-0114; 2026-10-01_06-22 → MEM-0115; 2026-10-04_23-54 → MEM-0116; 2026-10-21_21-06
→ MEM-0119; 2026-10-21_21-17 → MEM-0120. Inbox EMPTIED.
### 2026-10-22_14-03 planner_FlashNext (ses_e9d1b570…, ITER-20 maintenance pass) curation
Entries cured → `memory.md` + destill lines: 2026-10-22_02-11 → MEM-0121; 02-47
→ MEM-0122; 03-26 (both) + 04-38 + 07-36 → MEM-0123 (merged, buffer half kept,
N-regions half noted RESOLVED at plan15); 03-26 pin-audit → MEM-0124; 05-23
→ MEM-0125; 06-52 → MEM-0126; 10-26 → MEM-0127; 11-14 → MEM-0128. NO-OPs (logged,
not cured): 00-36 (clone red-probe baseline) — SUPERSEDED by the #141 clone-gate
landing (clone DoDs now require a green probe). Inbox EMPTIED.
### 2026-10-22 (planner-25, ITER-25 maintenance pass) curation
Entries cured → `memory.md` + destill lines: 2026-10-22_14-24 → MEM-0129;
2026-10-22_18-21 → MEM-0130; 2026-10-22_18-54 → MEM-0131. Inbox EMPTIED.

### 2026-10-22_20-13 planner_FlashNext ses_e9b46b7a2ffeRuFWSaQcOKdMTQ
When verifying a worker DoD that bounds output WIDTH, measure it twice-safe: run the script over a LARGE sample (--lines=200) and compute max char length with a node one-liner (bytes vs chars and display `cut -c` both lie) — evidence: plan26 #145 re-verification, awk reported 73 over-125 lines, node reported max 120 chars, 0 over (2026-10-22). Confidence: high. Scope: any bounded-output sweep/DoD check. Review trigger: if a future DoD states a BYTE bound, re-read the DoD wording.

### 2026-10-22_22-24 planner_FlashNext ses_e9b246abdffesGhYjB50ymiwqy
When the maintainer rules a sync/ownership BOUNDARY (which paths are shared vs branch-local), apply it against the CURRENT writer set and say so: a rule like "plan state always taken from MAIN" is correct while only one tree writes plan state, but becomes destructive the moment the second tree runs its OWN loop (its NAP/loop writes get overwritten at the next sync). Record the tension + the resolution options IN the entry/NAP at ruling time (observed 2026-10-22, direct planner-28, TODO #147: `agent/handover/**` + `loop/**` ruled `--theirs` = MAIN; the v2 host's own loop would need the one-tree switch or a direction flip), with a review trigger = the day the second host starts a loop.

### 2026-10-22_23-19 planner_FlashNext ses_e9b246abdffesGhYjB50ymiwqy
Before recording any worker-claimed BASELINE FAILURE or commit hash: (1) run the claimed-red command yourself at the claimed baseline AND at the worker's HEAD; (2) validate every cited hash with `git log --oneline -3 -- <path>` in that tree. A worker can fabricate a plausible hash and a plausible failing baseline while its actual work is good (observed 2026-10-22, TODO #137 order 2: claimed `state_snapshot.smoke.mjs` red at `90d6d6c` with hash `6130a70`; planner re-runs ALL PASS at both trees, last-touch was `9fa7d88`). Record the discrepancy in the TODO status line, do NOT propagate the claim into a new TODO entry.

