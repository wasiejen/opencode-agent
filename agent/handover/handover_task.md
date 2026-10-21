# Task spec — block_transfer hardening (merged fix unit, maintainer GO 2026-10-21)

## Goal
Fix the verified defect cluster in `.opencode/tools/block_transfer.ts` (the two
`--comment` review drafts in `maintainer/draft/block_transfer_feedback2.md`
are the inspiration only — several of their claims do NOT apply to the
deployed file; this spec carries the verified set). Behavior stays
deterministic and teaching; the tool description stays the single agent-facing
surface.

## Definition of done
- All 9 items below implemented in the tool file.
- Green gate, exact commands:
  - `node .opencode/plugin/tests/block_transfer.smoke.mjs` → ALL PASS (baseline 120 chk; re-pins + new pins allowed — state the new count)
  - `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` → ALL PASS (baseline 46)
  - `node .opencode/plugin/probes/handover_probe.mjs` → PROBE handover: 360/360 PASS baseline (re-pins + new pins allowed — state the new count)
- One green commit; write `agent/handover/handover_task_to_planner.md` (change summary, measured gate results with exact counts, working-tree status, commit hash, omitted work).
- Any legitimately surfaced discrepancy → append via `submit` (todo channel), never resolve alone.

## Items (WHAT; HOW is worker's call)
1. MOVE atomicity: resolve dst sandbox + `dstFile` presence + `targetMarker`
   ALL before touching src. Current violation: src written at ~L682, target
   resolved at ~L706-710 — a bad target leaves the source cut. ALSO stash the
   extracted block in buffer `last_move_source` before the cut (recovery
   buffer for destructive MOVE; document in the description BUFFERS note).
2. Same-file MOVE (`srcFile` and `dstFile` resolve to the same file): implement
   move-with-shift — pre-call line numbers for source span + target, adjust
   insert for the removed block, reject a target INSIDE the source span with a
   teaching error. (Review Option A; rejection-only is NOT acceptable.)
3. EOF insertion: use the existing `countLines()` (L169) for the EOF index in
   PASTE + MOVE + new-file paths — no spurious blank line appending to a
   trailing-newline file (measured: `"a\n"` → `"a\n\nb"`), no leading blank
   line on file creation.
4. EOL preservation: detect destination/file EOL once; write with it; re-attach
   `\r` to buffer lines pasted into a CRLF file. No whole-file CRLF→LF churn.
5. PEEK digit-string coercion: `from`/`count` accept the same string-union +
   `/^\d+$/` treatment as refs (plan25: the live channel stringifies schema
   integers). Current violation: `Number.isInteger` guards at ~L377/380 reject
   `"5"`. Widen the two schema fields like the ref fields' union.
6. Validation consistency: `refPresent()` at every form-detection site (replace
   the `!== undefined` check at ~L219); reject EMPTY `regions` / `refs` lists
   with a teaching error; COPY bare-input error names all THREE forms (mirror
   the APPEND message at ~L250).
7. Return lines: MOVE/PASTE name the insertion point (e.g. `(inserted after line
   12 in dst)` / `(at EOF)`); REPLACE/WRITE report old→new spans and the file's
   new line count (e.g. `old 12..15 -> new 12..19, file now 143 lines`) —
   line-number refs go stale after a length-changing op; the feedback must say
   so. Keep one line per op; update the RETURN LINES description paragraph to
   match byte-exactly.
8. MAP line-width cap: echo each line capped at ~200 chars + `…` (line COUNT
   already bounded by design); update the MODES description sentence
   (currently "VERBATIM ... (no cap)") + the knowledge doc
   `agent/knowledge/plugin_tools/2026-09-26_block_transfer-v2-s3-write-peek-
   feedback.md` MAP/PEEK section if it repeats the no-cap wording.
9. Doc drift (NO behavior change here): DELETE the "the file is created if
   absent" clause from the WRITE description — the pinned behavior is
   absent-file WRITE = teaching not-found/out-of-range errors, NO creation
   (behavior pins: block_transfer.smoke.mjs ~L213/L215 — do NOT touch these
   two checks). Remove the matching self-contradictory half-sentence in
   `agent/knowledge/plugin_tools/2026-09-26_block_transfer-v2-s3-write-peek-
   feedback.md` ("...and a successful write creates the file...").
10. sandboxCheck temp root: use `os.tmpdir()` (keep `TEMP`/`TMP` as an
    override ahead of it). Windows-impact nil; POSIX correctness.

## Verified facts (planner-measured, do not re-derive)
- Deployed file is 723 lines; guards for dst-sandbox + missing dst ALREADY
  EXIST (~L636-646) — do not re-add.
- The reviews' claims: syntax error, unknown-mode fallthrough ("Operation
  completed." lie), and the WRITE-creation CAPABILITY claim do NOT apply
  (schema enum at L326 blocks unknown modes; creation is doc drift, see item 9).
- Pin sets that WILL change and must be repurposed in-scope: block_transfer.smoke.mjs
  feedback-line byte-exact pins (Part F shapes), sandbox smoke, probe S15
  (~L657 area), S30 (checks 305-315, feedback shapes), S31 drift-guard
  equivalence section (anchor logic must stay equivalent with
  intercept_observer_core.ts `matchAnchorPrefixLines` — do NOT touch the
  plugin core).
- `countLines()` exists and models `"a\nb\n"` = 2; `textToLines()` exists (~L197).
- WRITE summary line at ~L598 is balanced and correct (the review's P0 #1 was
  paste-only).

## DO-NOT-touch
- `opencode.jsonc`, anything under `maintainer/`, `AGENTS.md`, `agent/prompts/`
- `.opencode/plugin/intercept_observer_core.ts` (drift-guard equivalence must
  hold WITHOUT editing it)
- `agent/handover/handover_planner.md`, `TODO.md`
- `agent/knowledge/plugin_tools/2026-10-16_v2-plugin-harness-and-config.md`
- Declared OUT of scope (maintainer ruled these away or deferred): create-file
  sentinel for WRITE, removing the `args.mode || "MOVE"` default beyond
  harmless, session-keyed buffers, LIST mode, undo buffers beyond
  `last_move_source`, description shortening (2-tier proposal pending), anchor
  trailing-space trim, `line:42` grammar, atomic-write helpers, PEEK/MAP merge.

## Assigned worker
worker_FlashNext (same model as the launching planner — cache rule).
