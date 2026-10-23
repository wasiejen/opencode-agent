# worker memory_inbox.md — append-only (the `submit(memory=...)` lane)

### 2026-10-22_14-03 planner_FlashNext (ses_e9d1b570…, ITER-20 maintenance pass) curation
First curation of this lane — all 10 entries cured into the NEW
`agent/memory/worker/memory.md` (WM-01…WM-10) + `destilled_mem.md` (same folder):
2026-09-30_16-46 → WM-01; 2026-10-01_04-09 → WM-02; 2026-10-05_02-34 → WM-03;
2026-10-21_21-59 → WM-04; 2026-10-22_01-34 → WM-05; 04-07 → WM-06; 04-28 → WM-07;
08-18 → WM-08; 11-01 → WM-09; 14-03 → WM-10. Inbox EMPTIED.
### 2026-10-22 (planner-25, ITER-25 maintenance pass) curation
Entries cured → `memory.md` + `destilled_mem.md`: 2026-10-22_14-55 → WM-11;
2026-10-22_18-41 → WM-12. Inbox EMPTIED.

### 2026-10-22_20-08 worker_FlashNext ses_e9b41981bffejgVkyxQDFUsp2C
block_transfer mode contract (observed, high confidence, scope: any block_transfer call): REPLACE takes a named BUFFER only; a 'text' argument in a REPLACE call is ignored and the mode fans the existing 'default' buffer into the anchored region (2026-10-22: my tests/intercept_sweep_fixture.log lines 9-10 were replaced with foreign WM-11/WM-12 memory text, then repaired via WRITE). WRITE takes direct 'text' (single span, or one text per region for a regions list); edit remains right for a small exact string swap. Review trigger: any REPLACE call I intend as "replace this span with this text I am typing now" - that is WRITE, not REPLACE.

### 2026-10-22_22-19 worker_FlashNext ses_e9acefcfbffewfyOO0y7VCJsfP
observed, high confidence, scope: any node script that builds a scratch git pair to test MERGE behaviour — the clone must be created BEFORE the source-side commit (`git clone` after the base commit, then diverge each side); cloning after the source tip makes the pair a fast-forward descendant and merge-tree reports 0 conflicts, silently voiding the test. Evidence: port_sync.selftest.cjs buildPair, selftest runs 2-5 (0-conflict) vs 6 (3 conflicts, 84/84 pass) 2026-10-22. Review trigger: any new git-shape selftest.

### 2026-10-23_01-58 worker_FlashNext ses_e9a023f54ffe1a5bvvPLmFBph9
Line-anchored splice between two large files: derive the added-block range from a unified-diff hunk header by hand and you drop lines (the `-A,B +C,D` counts cover the WHOLE hunk = context + added, not just the added block) — twice this session I spliced 11 lines where the block was 12 and 4 where it was 5, producing a mock whose mass-return line vanished and a missing pin. Lesson: after ANY splice, run `diff -u <target> <source>` and require the intended hunks to be GONE (empty grep) before running the gate; the assertion anchors alone do not catch a short block. Type: observed, confidence high, scope: any probe/smoke re-pin done by line-range splice (gauge.mjs 1015 lines, handover_probe.mjs 8677 — never whole-read). Review trigger: a splice feedback line whose `file now K line(s)` does not equal old-count + (cloneLen - mainLen) of the intended hunks.

