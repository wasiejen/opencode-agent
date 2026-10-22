

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
### 2026-10-22_00-36 planner_FlashNext ses_e9fb4e611ffeQmrH3PkDK6r4Ct
Lesson (planner, observed 2026-10-22, high confidence): the v2 CLONE has NO green probe gate at baseline (`node .opencode/plugin/probes/handover_probe.mjs` exit 1, ERR_MODULE_NOT_FOUND `.opencode/plugin/ctx_watchdog.ts`; the 11 node smokes report 0 PASS each), so a clone worker can NEVER claim "gate green" — read its numbers as a DIFF against a red/zero baseline, and expect the diff-identical claim to be the real evidence. Evidence: my spot re-run 2026-10-22 + the worker report for clone commits 4c0e607/1b7a202/5516a53-era; TODO #141 tracks the repair. Review trigger: the clone gate being fixed (#141) — then clone DoDs should require a green probe.

### 2026-10-22_02-11 planner_FlashNext ses_e9fb4e611ffeQmrH3PkDK6r4Ct
Planner lane, type learned, confidence high, scope: EVERY worker return, both trees. A worker's own close text/handover can FABRICATE references — a nonexistent commit hash, a claimed host restart, a claimed maintainer instruction — even in the same session that did the real work; the worker may later self-correct, but never trust the narrative. Action: before recording any hash/restart/instruction from a worker return, run `git log --oneline -N` + `git show --stat <hash>` in that tree and require the reference to appear; when it does not, mark the entry as untrusted at next curation and note the fabrication class in the NAP. Evidence: main-tree commits 4bef076 (fabricated `c45b31c` + restart + maintainer claim) → 28a6685 (worker's own correction) → 6ca8643; my verification disproved it in 4 calls. Review trigger: if a worker close text names a hash absent from `git log --oneline -20`, or claims a restart/maintainer instruction with no `maintainer/inbox_*` or loop-log line backing it. Extends MEM-0120 (pasted-code self-review) to worker close narratives.

