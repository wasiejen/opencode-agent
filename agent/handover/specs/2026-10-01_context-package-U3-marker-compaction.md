# Task spec — U3: marker compaction, the cap backstop (compact_memory mode)

Design source (read first): `proposals/approved/2026-10-01_context-budget-package.md` §U3.

## Goal
A new `compact_memory` mode: the agent supplies the summary text (compaction
style, current model) + the boundary; the tool writes the **marker quartet**
into the target session's DB (sqlite — the same access `context_trim tail`
uses) and runs the FULL compaction protocol (budget increment + ctx.log
COMPACT line + pre-compaction dump + queued-message relay). It consumes
ONE budget compaction. The window re-derives at the NEXT STEP (zero
restart).

## Verified facts (measured 2026-10-01 by the planner)
- The tool is PLUGIN-registered (not in `.opencode/tools/`):
  `.opencode/plugin/compact_memory.ts` (dispatch + the pre-compaction dump
  hook L93-105; the dump script is `agent/scripts/db/dump_session.cjs
  <sid> --out <relpath>`) + `.opencode/plugin/compaction_core.ts` (the
  budget / ctx.log writes — the verified-success-only COMPACT line).
- Marker quartet schema: a user row carrying the compaction part
  `{type:"compaction", tail_start_id:<target id>}` + the first assistant
  row strictly AFTER it with `info.summary` set + the tail target id.
  The window guard: `tailIndex < compactionIndex && summaryIndex >
  compactionIndex` (fork message-v2.ts:568, mirrored in
  `.opencode/tools/context_trim.ts`).
- The quartet FIXTURE CONSTRUCTION already exists in
  `.opencode/plugin/tests/context_trim.smoke.mjs` — read that fixture
  builder (bounded) and REUSE its exact row shapes; never hand-derive a
  row shape.
- `tailSet` (messageID form) at `.opencode/tools/context_trim.ts` L586
  (U0 edited this file after this spec — re-verify by grep at run) —
  the single-sqlite-transaction precedent (fail-closed, WAL-safe).
- The relay = the auto_resume unit-4 CONTINUE relay (compact_memory.ts
  L34 — the existing queued-message store).
- The new mode = a SELF dispatch where the summary text is agent-provided
  (the host summarize call is skipped); the budget increment +
  verified-success-only COMPACT line + dump reuse the existing dispatch
  machinery (do not re-derive them).

## DoD (measured end states)
- Fixture-DB pin THROUGH THE TOOL SURFACE: the quartet is written —
  `context_trim report` afterwards shows the new marker + tail (the
  window re-derives); the budget increment lands in the fixture budget
  file; the COMPACT-line + dump mechanics fire (fixture equivalents).
- Budget: ONE budget compaction consumed (the normal cap applies — pin
  the increment; no new budget line).
- One live throwaway-session test (probe-style spawn — the probe already
  spawns sessions; fixture pins first).
- Tool DESCRIPTION updated (the new mode + when to use it: cap/full-window
  case, when the filter can't free enough mass, one budget consumed).
- Standard gate green: probe + all 11 smokes (compact_memory smoke at its
  current baseline — re-verify the count at run).
- Checkpoint: one code commit per verified unit; TODO + handover in the
  final commit (hashes recorded by the planner, not the worker).

## DO-NOT-touch
- the live opencode.db of REAL sessions (fixture DBs + throwaway sessions
  only); the fork; `maintainer/`; other tools/plugins; the proposal
  files; `agent/prompts/**`.
- New TODO entries: expectation only, never a promise.

## Worker
worker_Q3S (one DB-write mode + protocol reuse + fixture pins).
