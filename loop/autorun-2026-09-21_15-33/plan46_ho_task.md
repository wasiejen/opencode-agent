# TASK — plan46: TODO #124 block_transfer return-line / description clarity

Worker: `worker_Q3S` (170K). Planner: plan46 (autorun-2026-09-21_15-33,
iteration 46). Class: PRE-APPROVED (agent-usage friction removal —
description-only is the default path; a PASTE return-line addition is
allowed per the DoD).

## Goal
Make the `block_transfer` description complete and unambiguous so a new
hire can predict EVERY return line and error from the description alone
(TODO #124). Three findings from the plan45 new-hire test (worker-45,
battery 10/10 PASS; scratch evidence `.opencode/temp/bt_newhire_plan45/`,
todo_inbox entries 2026-09-30_05-25) are the work items.

## Work items (the DoD list)
1. **Return-shape documentation** — the PASTE/REPLACE/WRITE/MOVE return
   lines report a source span / buffer range + a `first:` field, but never
   the destination insertion line, and `first:` mixes the old-span range
   with the new content's first line (for REPLACE). Document the exact
   per-mode return shape in the description: what the range means per mode
   (PASTE = the buffer's own lines 1..N; REPLACE = the REPLACED old span
   start..end; MOVE = the source span), what `first:` echoes (the NEW
   content's first line for PASTE/REPLACE/WRITE — capped ~40 chars — NOT
   the old span), and where the block landed in the destination (after the
   `targetMarker` line, or appended at EOF when omitted).
2. **Non-unique anchor error** — the description says "the match count +
   the first match line numbers" (ambiguous). State the ACTUAL bounded
   behavior: the match count + the FIRST 3 match line numbers (all of them
   when 3 or fewer), ` …` appended when more (the line stays bounded).
3. **Buffer-on-failure** — document that a failed COPY/CUT/APPEND (anchor
   resolution error) leaves NO buffer behind (no partial state on
   rejection); a later PEEK/MAP of that buffer name returns the "buffer is
   empty" error.
4. **Optional, same run (only if the unit stays comfortably in scope):**
   the PEEK head/tail overlap on small buffers (a 3-line buffer has head ==
   tail) + the residual-blank-line note after MOVE/DELETE (the block's
   own lines are removed; adjacent lines — including blank ones — are
   untouched, so two blank lines can end up adjacent).

**Build call (per TODO #124 DoD):** item 1's destination-landing gap may be
closed in the DESCRIPTION (preferred, zero behavior change) OR by adding
the destination insertion line to the PASTE return line (behavior change —
then re-pin the smoke pins). Your call; a changed return line is in-scope
only with its re-pins.

## Verified facts (planner-measured at spec time — do not re-derive)
- Baseline GREEN (measured 2026-09-30, plan46): bt smoke `BLOCK_TRANSFER_
  SMOKE: ALL PASS (131/131)`, sandbox `BT-SANDBOX-SMOKE: ALL PASS (64/64)`,
  probe header total `PROBE handover: 352/352 PASS`
  (`.opencode/plugin/probes/handover_probe.mjs` line 1002 annotation).
- Return lines are built by `opLine` (block_transfer.ts:292-297) and
  `assemblyFeedback` (278-284); the PASTE return is at 464, the REPLACE
  return at 506 (range = old span, echo = buffer's first line); read the
  MOVE/WRITE branches in the 430-560 area to document them exactly.
- The non-unique error is `nonUniqueError` (124-128): first 3 matches +
  ` …` when more.
- "no partial state on rejection" is already implemented (comment at 215)
  — item 3 is documentation only.
- Standard gate in THIS repo = the probe + the smoke suite (no pytest —
  FST-only, #117). Run: `node .opencode/plugin/probes/handover_probe.mjs`
  + `node .opencode/plugin/tests/block_transfer.smoke.mjs` +
  `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` (+ the
  other smokes in that folder, unchanged — re-run at least the two bt
  ones and the probe; the full smoke list = every `*.smoke.mjs` in
  `.opencode/plugin/tests/`).
- Existing behavior-pinning tests: the bt smoke + sandbox smoke pin the
  return-line shapes byte-exactly (they are the re-pin targets if any
  return line changes); the probe pins the error strings (check the S-
  sections that quote non-unique / empty-buffer returns before changing
  anything — re-pinning is in scope).

## Definition of done
1. The description (block_transfer.ts `description` template, lines
   301-322) carries items 1-3 (and 4 if done) — a per-mode return-shape
   note + the bounded non-unique error form + buffer-on-failure.
2. If any return line changed: smoke pins re-pinned (baseline 131/131 +
   64/64) and the run still ALL PASS.
3. Gate green: probe `352/352` (or the re-pinned total, header annotation
   updated in the SAME commit) + both bt smokes ALL PASS + the remaining
   smokes ALL PASS.
4. `TODO.md` #124 status → `LANDED` (the commit hash is recorded in the
   planner's follow-up bookkeeping commit — never your own).
5. Checkpoint commits per verified unit (code only); `TODO.md` + the
   handover file ride the FINAL commit.

## DO-NOT-touch
`maintainer/**`, `opencode.jsonc`, `AGENTS.md`, `agent/prompts/**`, the
NAP (`agent/handover/handover_planner.md`), `todo_inbox.md` (already
curated — #124 covers the entries), the other tools/plugins, and
`projects/Free-Snap-Tap/**`. The probe file may be touched ONLY to
re-pin a changed pinned line (header annotation updated in the same
commit).

## Handover
`agent/handover/handover_task_to_planner.md`: executive summary, which
build call you made (description-only vs return-line addition + why),
the per-item changes, measured gate numbers, commit hashes (of the code
commits, not your own), TODO entries, what you deliberately did not do.
Loop log: one `-->START` (your first action) + one `DONE<---` (final gauge
readout) — role `worker-46`, your model id verbatim, session from your
injected `ctx:` line.
