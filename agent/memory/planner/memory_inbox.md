

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

### 2026-10-22_14-24 planner_FlashNext ses_e9d1b5700ffeFZZb50qex5NWBx
File-surgery span anchors: an entry/section span ends at the NEXT ANY `^## ` header, NOT the next id-pattern-matching one — re-grep the exact line spans immediately before EVERY multi-op block_transfer surgery (MOVE/DELETE/WRITE with line numbers). Observed 2026-10-22 (planner-20, TODO retire at the plan20 maintenance pass): anchoring a MOVE on an id-grep's 'next header' carried 38 unintended lines (2 section headers + entries 53/56) into todo_records.md; repaired by a verbatim MOVE-back + narrow redo (4 extra calls). Type: observed; confidence: high; scope: any TODO/records/NAP line-anchored surgery in this repo; review trigger: block_transfer re-validating spans before cutting (then anchors may stay cached). Extends MEM-0112 (sweep vs ls) to intra-file surgery.

### 2026-10-22_18-21 planner_FlashNext ses_e9c6da8fbffeOHy63B2mMFliv1
Planner-direct units (≤~50 diff lines, ≤4 files): close them inline in the planner session with per-file grep-back verification after every edit/bulk replaceAll — the `edit` tool's `hint rejected lines=` output is NOT a rejection signal (two 14/14-class replaceAlls landed while it fired); NEVER conclude rejection from the hint line, always read the file back (2026-10-22 #144, extends MEM-0111 to the replaceAll form).

### 2026-10-22_18-54 planner_FlashNext ses_e9b95111affe0iVT7mWWG732Rx
Spec-authoring lesson (planner, verified 2026-10-22 marker-order unit): when a spec suggests a TEST FIXTURE for an ORDERING/behavior pin, verify at authoring time that the fixture DISCRIMINATES — run/state the raw emit order and require the pin to FAIL on the UNCHANGED code (measured: my suggested `maintainer/live_note.md` fixture passed 21/21 pre-change because `grep -rn` emits hits in its path ARGUMENT order, `maintainer/` before `loop/`; the worker had to switch to a `WIKI.md` fixture). Also check the fixture's line-count math before adding to it (the state_snapshot FX1 digest is pinned at the EXACT 60-line cap — replace a fixture line, never add). Type: learned; confidence: high; scope: any planner spec naming a smoke fixture for a non-vacuous pin; review trigger: a new pin that passes against the pre-change code.

