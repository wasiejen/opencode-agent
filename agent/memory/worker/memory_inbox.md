# worker memory_inbox.md — append-only (the `submit(memory=...)` lane)

### 2026-10-22_14-03 planner_FlashNext (ses_e9d1b570…, ITER-20 maintenance pass) curation
First curation of this lane — all 10 entries cured into the NEW
`agent/memory/worker/memory.md` (WM-01…WM-10) + `destilled_mem.md` (same folder):
2026-09-30_16-46 → WM-01; 2026-10-01_04-09 → WM-02; 2026-10-05_02-34 → WM-03;
2026-10-21_21-59 → WM-04; 2026-10-22_01-34 → WM-05; 04-07 → WM-06; 04-28 → WM-07;
08-18 → WM-08; 11-01 → WM-09; 14-03 → WM-10. Inbox EMPTIED.
### 2026-10-22_14-55 worker_FlashNext ses_e9c6c3cbcffeSLiyxqFZ9f5pl7
Byte-exact smoke pinning (observed, high confidence, port work): derive expected feedback strings from the code's format functions first, and design fixtures to satisfy the tool's own anchor-uniqueness rule (prefix anchors match shorter twins, e.g. 'X' matches X1..X5); new-file writes have NO trailing newline while existing files keep their own EOL — mispinned both legs 4 times in unit 5c before re-running. Evidence: clone block_transfer.smoke.mjs fix iterations 2026-10-22. Review trigger: writing any new byte-exact smoke.

### 2026-10-22_18-41 worker_FlashNext ses_e9b8ed378ffewWRy0cDWYNTkQA
Ordering/behavior pins must be proven DISCRIMINATING before the implementation lands: run the new pin against the UNCHANGED tool first and require it to FAIL (observed 2026-10-22 state_snapshot marker-order unit — the spec-suggested `maintainer/live_note.md` fixture passed 21/21 pre-change, pin was vacuous; switching the fixture to `WIKI.md` gave the measured FAIL-then-PASS sequence). Type: learned; confidence: high; scope: any worker adding a pin for a reordering/behavior change; review trigger: a new pin that passes on the pre-change code.

