# Worker summary — block_transfer hardening (merged fix unit, maintainer GO 2026-10-21)

Status: COMPLETE
Assigned: worker_FlashNext · session ses_ea0120bd5ffeM6ggDl5IPr5o49
Commit: the green commit containing this file (hash reported in the closing message; `git log -1` shows it).

## Task and scope
All 10 items of `agent/handover/handover_task.md` implemented in `.opencode/tools/block_transfer.ts`,
plus the two named knowledge-doc wording fixes. DO-NOT-touch list respected; no plugin-core edits
(`intercept_observer_core.ts` untouched — the S31 drift guard still passes unchanged).

## Files changed
- `.opencode/tools/block_transfer.ts` — the fix set (723 → 913 lines).
- `.opencode/plugin/tests/block_transfer.smoke.mjs` — 9 feedback pins re-pinned; 16 new pins (items 1-8).
- `.opencode/plugin/probes/handover_probe.mjs` — 8 byte-exact pins re-pinned IN PLACE in S15 (110/111/117/262/263-
  region) and S30 (305/307/313); NO probe check added or removed (section counts + the 360 annotation unchanged).
- `agent/knowledge/plugin_tools/2026-09-26_block_transfer-v2-s3-write-peek-feedback.md` — item 9 doc-drift fix
  (the WRITE-creation half-sentence removed; the WRITE Keys line now says "no file creation").
- `agent/handover/handover_task_to_planner.md` — this summary.
- `loop/<run>/loop_log.md` — the DONE line (committed with the unit).

## Behavior changed (measured, item by item)
1. MOVE atomicity: the targetMarker now resolves BEFORE the source is cut; a bad target leaves BOTH files
   byte-identical (smoke pin added). The block is stashed in buffer `last_move_source` before the cut
   (PEEK-pinned; documented in the BUFFERS description note).
2. Same-file MOVE (`srcFile`/`dstFile` resolve equal, case-insensitive): move-with-shift — span + target
   resolve against the pre-call state, the insert index adjusts for the removed block (target below ⇒
   `target.line - spanLen`), a target INSIDE the span returns the teaching shift error; the return names the
   FINAL insertion line (3 smoke pins added: down-shift, up-move, inside-target).
3. EOF insertion via `countLines()` (`eofIndex()`): `"a\n"` + block ⇒ `"a\nBLK one\nBLK two\n"` (no spurious
   blank line); a created file gets no leading blank line (2 pins).
4. EOL preservation: `detectEol()` once per file; `joinLines()` re-attaches `\r` to every line except the LAST
   element (so both `"a\r\nb"` and `"a\r\n"` round-trip — the first attempt double-CR'd, fixed and pinned).
   Source writes (CUT/DELETE/MOVE cut) keep the SOURCE's own EOL.
5. PEEK `from`/`count`: schema widened to the ref-style string|int union; digit-strings coerce
   (`from: "5", count: "3"` = the numeric call); non-numeric → the teaching error quoting the value as given.
6. Validation: `refPresent()` at every form-detection site (the `!== undefined` check replaced); EMPTY `refs`
   and EMPTY `regions` lists rejected with teaching errors; a bare COPY teaches the THREE input forms
   (the APPEND string stays byte-identical); an empty-string marker counts as absent.
7. Return lines: MOVE/PASTE append ` (inserted after line K in '<dstFile>')` / ` (at EOF)`; REPLACE/WRITE
   replace the `(lines R` group with `old A..B -> new C..D, file now K line(s)` (WRITE's NEW span in the
   post-op numbering — lower regions shift higher ones; EMPTY text ⇒ `new (empty)`). The RETURN LINES
   description paragraph updated to match. The WRITE multi-region summary line is UNCHANGED.
8. MAP: each echoed line capped at 200 chars + `…` (byte-exact pin); the MODES description sentence
   ("VERBATIM ... (no cap)") replaced. The knowledge doc has NO MAP/no-cap wording → nothing to fix there.
9. Doc drift: the "the file is created if absent" clause removed from the WRITE description, the mode-enum
   describe, and the dstFile describe; the knowledge doc's contradictory half-sentence removed. The two
   pinned absent-file checks (smoke ~L213/L215) were NOT touched.
10. `sandboxCheck` temp root: `os.tmpdir()` added, with `TEMP`/`TMP` as the override ahead of it and the root
    list de-duplicated by exact string equality — so the byte-exact 2-root error form is UNCHANGED on Windows
    (verified: sandbox smoke 64/64 with zero edits to that file).

## Measured verification (exact commands, this checkout, post-change)
- `node .opencode/plugin/tests/block_transfer.smoke.mjs` → `BLOCK_TRANSFER_SMOKE: ALL PASS (147/147)` PASS
  (pre-change baseline measured 131/131 → +16 new pins, 0 removed).
- `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` → `BT-SANDBOX-SMOKE: ALL PASS (64/64)` PASS
  (unchanged; that file needed no re-pin).
- `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360 PASS` PASS (unchanged total).
- `node .opencode/tools/_tscheck.ts` → zero `error TS` lines for block_transfer.ts.
- `grep -c "^FAIL"` over the three captured outputs → 0 / 0 / 0.
- Baseline note: the spec quoted 120 / 46 / 360; the pre-change measured counts were 131 / 64 / 360
  (submitted to `todo_inbox.md` — the spec's 120/46 are stale, never resolved alone).

## Working tree
Clean for the committed paths. NOT committed (pre-existing, not mine): `opencode.jsonc`, `agent/memory/planner/memory_inbox.md`,
untracked `maintainer/draft/block_transfer_feedback2.md` — all on the DO-NOT-touch list; left untouched.

## Deliberately omitted / deviations
- Declared-out-of-scope list respected (no create-file sentinel, no session-keyed buffers, no LIST mode,
  no description shortening, no anchor trailing-space trim, no `line:42` grammar change, no atomic-write helper,
  no PEEK/MAP merge; the `args.mode || "MOVE"` default left as-is).
- Deviation from the spec's expectation: the sandbox smoke pin set did NOT need repurposing (measured).
- Deviation from "one commit" mechanics: the tool file was edited in verified sub-steps, but the EOL fix
  needed one extra round-trip (item 4's first write joined with `eol` after re-attaching `\r` → double CR);
  caught by the new pin, fixed, gates re-run green before committing.
- The knowledge doc's historical Part F feedback bullet (`Wrote 2 lines ... (lines 2..3, ...)`) is left as the
  dated record; the new shapes went to `knowledge_inbox.md` instead.

## Inbox / TODO entries submitted
- todo: the stale gate baselines (120/46 vs measured 131/64) + the stale "smokes 112/112 + 61/61" line.
- knowledge: the hardened block_transfer surface (return shapes, `last_move_source`, same-file shift, EOF/EOL rules).
- memory (worker): the ~2 KB tool-payload JSON-parse limit hit twice this session; batch payloads ≤ ~1.8 KB
  and re-pin byte-exact pins with a fail-loud replacement script.

## Lesson
Byte-exact feedback pins are the real spec: a behavior change is only landed when the pins are re-pinned
IN PLACE and the whole suite goes green — a plausible-looking implementation passed every other check while
the CRLF write silently doubled the CR.
