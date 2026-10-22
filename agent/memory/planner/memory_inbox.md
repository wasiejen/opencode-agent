

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

### 2026-10-22_02-47 planner_FlashNext ses_e9f0c2d53ffeYo5to2gs6Gb1W1
Planner lesson (2026-10-22, ses_e9f0c2d5, loop_log + handover_planner.md incident): a block_transfer WRITE/REPLACE feedback line (count + first-line echo) is NOT proof the text landed — a long single-call text can arrive truncated and the write silently no-ops; recovery = re-read the file, then re-apply as a smaller `edit` insert. Type=observed, confidence=high, scope=any big WRITE/REPLACE of >~15 lines of prose, evidence=the planner-13 NAP insert (file stayed 253 lines after a reported 24-line write), review trigger=if the WRITE tool later validates payload integrity.

### 2026-10-22_03-26 planner_FlashNext ses_e9eeb0952ffeP6BY7SnI5j2Yzs
MEM-0111-class surface distrust extends to block_transfer feedback (observed 2026-10-22, planner-14): a WRITE with a `regions` LIST + single `text` applies the SAME text to EVERY region (no per-region pairing), and REPLACE consumes whatever buffer sits at that name — the auto `last_write` buffer is NOT the content you assume; MAP/PEEK the buffer (or verify the file back) BEFORE REPLACE/WRITE-multi-region, repair with a line-number `sed`/`edit` when a whole block duplicates, and never trust the '+N line(s)' feedback line as proof of landing. Confidence: high. Scope: any NAP/bookkeeping surgery. Review trigger: a future block_transfer contract fix (reject N-regions-with-1-text) landing.

### 2026-10-22_03-26 planner_FlashNext ses_e9eeb0952ffeP6BY7SnI5j2Yzs
Spec pin-audit inventories must be token-enumerated, not range-recalled (learned 2026-10-22, planner-14, spec `agent/handover/specs/2026-10-22_142-138-path-field-scan.md`): my Pin audit listed pin sites by line ranges and missed three flipping surfaces (smoke (B3), probe checks 303 + 304) — the worker repinned them in scope and filed the gap. Procedure: before narrowing/grepping a channel, run `rg -n "<verdict-token>"` across EVERY pin file (smokes AND the probe) and build the audit from that hit list; treat the first post-change gate run's FAIL list as the authoritative inventory. Confidence: high. Scope: any spec naming a pin audit. Review trigger: none expected; re-check if pin files are restructured.

### 2026-10-22_04-38 planner_FlashNext ses_e9ec78ee4ffe9Nu75F5iRfdtl6
For a direct-text edit of a line-anchored REGION, use block_transfer WRITE (bufferless); REPLACE/COPY/APPEND/PASTE work on NAMED BUFFERS (empty-buffer error does not name the mode fix) — on the empty-buffer error, re-check the mode list before retrying the same mode (observed 2026-10-22, planner-15 NAP curation). Type: observed; confidence: high; scope: block_transfer region edits; review trigger: block_transfer mode redesign or description rework.

### 2026-10-22_05-23 planner_FlashNext ses_e9e860260ffeKO6KiJbGss5pwy
Typecheck pins need a NEGATIVE CONTROL: a tsgo run printing zero errors is NOT proof a pin is real — a generic `E extends Array<unknown>`-style hook param silently widens and accepts bogus literals (observed 2026-10-22 unit 3a: the first attempt accepted `role:"bogus-role", content:5`); push a deliberate bad literal, confirm RED, then commit — type (observed), confidence high, scope = any fork-typed v2 plugin work, review trigger = a future typecheck-only pin.

### 2026-10-22_06-52 planner_FlashNext ses_e9e860260ffeKO6KiJbGss5pwy
A Task-tool launch rejection ("The user rejected permission to use this specific tool call") means the sub-agent NEVER STARTED (verified 2026-10-22: target repo tree + git log untouched) — classify as a launch failure, NOT a context/permission task failure; do NOT compact or resume; re-issue ONCE with a SHORT prompt that points at the committed spec file (the long spec-echoing prompt is the suspected trigger — hypothesis, medium confidence), and if it rejects twice, stop and report the boundary per AGENTS.md §Authority and Recovery.

### 2026-10-22_07-36 planner_FlashNext ses_e9e0be707ffeKYt7BzTQjda5e9
Planner lesson (observed, high confidence, scope = block_transfer chains): COPY/CUT/PASTE without bufferName share 'default' — a later CUT/COPY silently replaces an earlier staged paste buffer mid-chain and the later op then pastes the WRONG content. Procedure: name buffers per artifact (e.g. u4astat/nap17), verify every write/read-back landed by reading the file (MEM-0111-class) before committing; evidence: this session (ses_e9e0be707ffe…, plan17) — the CUT of handover_task.md's 3b spec destroyed my staged 4a copy in 'default' and forced a re-COPY; review trigger: block_transfer description or a future buffer-scoping fix lands.

### 2026-10-22_10-26 planner_FlashNext ses_e9de38db2ffeuql8EJDBx23MkK
before REUSING a worker session via task_id for a NEW unit, check its close gauge: at ~89% of window (a 4b close at CTX=234727/262k) the resume request is rejected outright (prompt 262784 > window 262144 — 'requests are never truncated'); cross-compact FIRST (keepMessages=10, message= the new-unit pointer), verify the COMPACT line in ctx.log, THEN resume — saves the rejected-call round-trip (observed 2026-10-22 plan18 #137 4c: the 4b worker reused fresh would have died on launch; the pre-resume cross-compact made the resume land; type=observed, confidence=high, scope=serial one-slot loop with same-model workers, review trigger: worker close gauges start landing well below 80% or the cap semantics change).

### 2026-10-22_11-14 planner_FlashNext ses_e9d47a84dffejiRLQ1gHKUjCvH
Type: learned, confidence high, scope: planner spec authoring for the #137 v2-clone port work. Lesson: before committing a spec that anchors a port on a cross-tree design-of-record file, run the twin-freshness check (`cmp -s` + `wc -l` + a bounded `diff | head -60`) between the MAIN file and the clone copy; a stale twin baseline silently pins stale behavior and the delta re-surfaces as worker-side "discrepancy" noise. Evidence: plan19 5c spec — the draft pinned the CLONE's bt formats (845-line twin) and was re-baselined on the MAIN twin (1034 lines, 333 diff lines) before the launch; TODO #143. Review trigger: any future #137 unit spec, or a clone-twin refresh landing (then re-check the deltas).

