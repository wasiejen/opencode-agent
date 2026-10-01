# Task spec — context_trim NEW-HIRE test (plan8, iter 8)

Role: worker_Q3S_slow. Type: READ-ONLY test task — zero repo code changes.
Branch: stay on the current checkout (`opencode_test` — do not switch).

## Goal

A "new-hire" test of the `context_trim` tool: run a battery of operations
using ONLY the tool's description — never read the tool source, its smoke
tests, or any knowledge/research entry about it (deliberately file-blind).
Record every friction point where the description failed to predict the
behavior: missing semantics, ambiguous wording, or surprising return/error
forms. The battery finds DESCRIPTION friction, not bugs — do not fix
anything.

## Battery (record each call's args + the verbatim return)

0. **Canary:** `context_trim` `report` on YOUR OWN session (the id from your
   injected `SESSION=` ctx line). If the tool is NOT in your toolset: STOP,
   report the block in the handover — no workaround.
1. `report` on your own session (no `target`) — record the readout shape.
2. `report` on your own session + a `target` (any message id visible in the
   step-1 rows) — record the dry-run verdict shape.
3. `report` on a nonexistent session id (`ses_deadbeef000000000000000000`) —
   expect a fail-closed rejection naming the cause.
4. `tail` on your own session with an arbitrary target id — your session has
   no completed compaction → expect the no-completed-compaction rejection
   (no DB write).
5. `tail` on the nonexistent session id — expect session-not-found.

Write the raw transcript (args + returns) to the scratchpad:
`C:\Users\Wasiejen\AppData\Local\Temp\opencode\plan8_newhire_ctxtrim\`.

## Definition of done

- All 6 battery steps executed; transcript on the scratchpad.
- Findings file committed: `loop/autorun-2026-10-01_03-27/
  plan8_newhire_context_trim.md` — one section per friction (what you
  expected from the description / what you got / severity: missing,
  ambiguous, or misleading) + a PASS list of the battery + your self-
  declaration that no context_trim source/test/knowledge file was read.
- Handover to `agent/handover/handover_task_to_planner.md` (executive
  summary + friction count + the findings path).
- One green checkpoint commit (findings file); TODO note (if any) + the
  handover ride the same final commit.
- Zero writes to the live opencode.db: the battery is report-only +
  fail-closed rejection attempts (rejections return before any write).

## DO-NOT-touch

- `.opencode/tools/context_trim.ts` + its tests (NOT even read — that is the
  point of the test).
- Any `agent/knowledge/**` entry about context_trim / compaction /
  context-recovery (NOT read).
- Any `tail` call that could SUCCEED (only the rejection-form attempts above
  — your own session has no completed compaction; if one unexpectedly does,
  STOP and report it in the handover before attempting a tail).
- `maintainer/**`, `AGENTS.md`, `agent/prompts/**`, `.git/**`, `.github/**`.

## Verified facts (planner, at authoring time)

- `context_trim` is live on this host (auto-detected; `worker_Q3S_slow` in
  `opencode.jsonc` has no `tools` restriction → default toolset; the #126
  close verified report + tail + floor refusal LIVE post-restart).
- Rejection set per the tool description: floor (retained tail < 6),
  keep-exceeds-history, no-completed-compaction, session-not-found,
  keep-invalid; all fail-closed before any DB write.
- Standard gate: NOT applicable (no code changes).

## Expected TODO entry

None expected — but a legitimate doc/code discrepancy found via the
description is filed via the submit.todo channel (do not self-assign an ID).
