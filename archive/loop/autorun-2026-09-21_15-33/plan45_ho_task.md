# TASK — plan45: block_transfer NEW-HIRE test (description-only, file-blind)

Worker: `worker_Q3S` (170K). Planner: plan45 (autorun-2026-09-21_15-33,
iteration 45). Class: PRE-APPROVED (agent-usage test — zero behavior
change, no code edits).

## Goal
A new-hire test of the `block_transfer` tool (the orientation.md test
doctrine): drive a bounded file-manipulation battery using ONLY the tool's
description — never reading the tool source — against fixture files you
have not seen before (file-blind), and report every description gap,
ambiguous return, or error-message shortcoming as a numbered friction item.
You are the "new hire": the description is your only documentation.

## Verified facts (planner-verified at spec time — do not re-derive)
- `block_transfer` is auto-detected from `.opencode/tools/block_transfer.ts`
  (host auto-detect — no registration needed, plan39) and IS in your
  toolset.
- The current surface is defined by the tool's OWN description (modes
  MOVE / COPY / APPEND / CUT / PASTE / REPLACE / WRITE / PEEK / MAP /
  DELETE / CLEAR; refs = a short unique line-prefix marker OR an all-digit
  line number; `bufferName` default `default`; a WRITE auto-stores its
  text in the `last_write` buffer; PEEK = bounded preview; MAP = line
  count + head/tail + heading skeleton).
- Sandbox: the repo working dir + `C:\Users\Wasiejen\AppData\Local\Temp\
  opencode`.
- Scratch dir for this run (create it): `.opencode/temp/bt_newhire_plan45/`
  — gitignored; files there need no commit.
- A plan22 precedent run (2026-09-26) did this doctrine on the PRE-v2 tool
  (11/11 PASS). The tool has since gained the v2 unified anchor semantics +
  line-number refs + buffers + PEEK/MAP — this is a FRESH pass on the
  current surface.

## Battery (all files in the scratch dir; you author the fixture content)
1. AUTHOR: write `a.md` — a ~40-line markdown-ish fixture you invent.
   Mix: `#`/`##` headers, a numbered list, 2-space-indented lines, blank
   lines, and at least one line prefix that recurs elsewhere (so one
   deliberate non-unique anchor is available for step 10).
2. MOVE: move one `##` section from `a.md` into `b.md` (created by the
   MOVE), anchored by header prefixes, with a `targetMarker` line.
3. COPY: extract a small block from `a.md` into buffer `n1` (single-ref
   pair form); PEEK it; note the returned line count vs the actual.
4. APPEND: append two more single lines (the `refs` LIST form) to
   buffer `n1`.
5. PASTE: paste `n1` into `c.md` (an absent file — does it get created?)
   and once more at an explicit `targetMarker` position.
6. REPLACE: replace a line-anchored span in `a.md` with a buffer's content
   (use `n1`); verify the span boundaries landed exactly.
7. WRITE: write direct text into a span of `a.md` (single-span form);
   then reuse the `last_write` buffer for a second span via PASTE.
8. DELETE: purge one block from `a.md`.
9. MAP: MAP buffer `n1`; record the output shape verbatim.
10. ERROR PATH: trigger ONE deliberate failure (your known non-unique
    anchor) and quote the exact error return verbatim — message quality
    is test material.
After EVERY step: verify the file state you expected (read/grep the
scratch files) and record PASS / FRICTION / FAIL for that step.

## Rules
- DO NOT read `.opencode/tools/block_transfer.ts` or any other tool/plugin
  source. If you are truly stuck on a step: record the friction, THEN you
  may read the source as an ESCAPE — flag it in the report (`ESCAPE step N:
  <what the description lacked>`). Every escape is data, not failure.
- File-blind: never assume you know fixture line numbers — discover them
  via PEEK/MAP/reads (that is the point of the test).
- Allowed file changes ONLY: the scratch dir, the handover file,
  `todo_inbox.md` (via `submit(todo)` for keep-worthy frictions), and the
  loop-log lines. Never touch `maintainer/**`, `opencode.jsonc`,
  AGENTS.md, or `agent/prompts/**`.
- If you find a tool bug: FILE it (todo_inbox via submit) — do NOT fix it.
- One FINAL commit: the handover file (frictions ride `todo_inbox.md`;
  the planner assigns TODO IDs at curation — do not edit TODO.md).

## Definition of done
- All 10 battery steps attempted; each reported: step, mode, PASS /
  FRICTION / FAIL / ESCAPE, one line of evidence (quote the tool's return
  line verbatim where it matters).
- A numbered friction list; for each: what the description said, what you
  needed, what was missing or ambiguous. Keep-worthy items appended to
  `todo_inbox.md` via `submit(todo)` (one line each).
- `git status` clean except the handover file (the scratch dir is
  gitignored).
- Loop log: one `-->START` line (your FIRST action — before planning or
  heavy tool calls) + one `DONE<---` line (the final gauge readout) — role
  `worker-45`, your model id verbatim, session = the `SESSION=` field of
  your injected `ctx:` line.
- Handover summary to `agent/handover/handover_task_to_planner.md`
  (executive summary, the battery table, the friction list, measured
  `git status` evidence, what you deliberately did not do).

## DO-NOT-touch
`maintainer/**`, `opencode.jsonc`, `AGENTS.md`, `agent/prompts/**`,
`.opencode/tools/**`, `.opencode/plugin/**` (read-only ESCAPE per the rule
above; never edit), `TODO.md`, the NAP (`agent/handover/
handover_planner.md`).
