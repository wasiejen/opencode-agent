# Task spec — plan39: the `context_trim` tool (Unit 1 of the approved context-trim build)

Goal: build `.opencode/tools/context_trim.ts` — an agent-facing tool with two
modes over the live opencode DB: `report` (read-only) + `tail` (a validated
`tail_start_id` rewrite). NO `turns`/row-deletion mode (ruled out by the
maintainer, 2026-09-29).

Design authority (read FIRST, bounded — do not re-research):
- `agent/research/2026-09-28_context-erase-tail-trim.md` §3 + §5 (window
  semantics, the guard, the marker quartet, the lever).
- `proposals/approved/2026-09-28_context-trim-tool.md` — §"Build scope"
  (Unit 1) + the round-2 feedback §8 (the report's file/line info).

Verified facts (planner-measured at spec time — claims, not assignments):
- DB: `~/.local/share/opencode/opencode.db` (the gauge's DEFAULT_DB_PATH,
  `.opencode/plugin/scripts/gauge.mjs:157`).
- Context is re-derived from the DB at every step (prompt.ts:1092 → fresh
  select message-v2.ts:433-446 → toModelMessagesEffect) — no in-memory
  history; a DB write changes the next prefill with zero restart.
- A COMPLETED compaction = a user row carrying a `compaction` part; the
  summary = an assistant child with `info.summary` + finish + no error,
  `parentID` = the compaction user row's id (message-v2.ts:545-566). The
  compaction part carries `tail_start_id` (:538-540) = the id of the first
  message of the RETAINED tail.
- Model window = `[compaction-user, summary, slice(tailIndex, compactionIndex),
  slice(summaryIndex+1)]` (message-v2.ts:568-573); the guard =
  `tailIndex >= 0 && tailIndex < compactionIndex && summaryIndex >
  compactionIndex` (:568). Guard fail → the FULL un-reordered history is
  sent → overflow (no host auto-compact on this host; the context_recovery
  plugin backstop fires on overflow — #93 live-verified).
- Message ordering = (time_created, id) (the cursor predicate,
  message-v2.ts:96).
- The host itself rewrites `tail_start_id` in place on the part row
  (compaction.ts:461-466) — a single-field JSON edit on ONE part row is the
  lever; it never touches the quartet's existence.
- keepMessages floor = 6 (the maintainer's round-2 ruling) — a retained-tail
  count below 6 is rejected; state the floor in the tool description.

Surface (the `tool()` form — model the shape on `.opencode/tools/submit.ts`,
incl. its header/doc conventions):
- `report <session> [target]` — READ-ONLY: (1) the window-state header — the
  last completed compaction (marker present? compaction user row id, summary
  id, current tail_start_id, retained-tail count, post-summary count; no
  marker → say so); (2) per-message rows for the messages INSIDE the model
  window (retained tail slice + post-summary slice): id, time, role, and for
  tool parts: tool name + target (`read`: input filePath + offset/limit;
  `bash`: command; `webfetch`: url), token mass (info.tokens
  input+output+cache.read when present, else bytes/4 of the part text);
  (3) with `target`: a dry-run of what a tail rewrite to it would
  remove/keep (counts + token mass) — no write.
- `tail <session> <messageID>` — WRITE: rewrite the last completed
  compaction part's `tail_start_id` to the target id. Validation,
  fail-closed (mirror the guard exactly): (1) session exists; (2) the last
  completed compaction exists (marker + summary child); (3) the target id
  exists in the session; (4) the target sits strictly BEFORE the compaction
  user row in (time_created, id) order; (5) retained-tail count (target →
  compaction row, exclusive) >= 6 (the floor). Single transaction; WAL — the
  host reads on its own connection (concurrent read safe); SQLITE_BUSY → one
  short-backoff retry → fail-closed. Return: `tail= <old> -> <new>
  keep=<count>` or the rejection reason.
- Core exported for the smoke (`reportWindow(dbPath, sessionID, target?)`,
  `tailSet(dbPath, sessionID, targetID)`); the `tool()` wrapper uses the
  gauge's default DB path.
- NO registration by the worker: the handover carries the registration
  snippet (name + description text) for the maintainer's opencode.jsonc.

Smoke — `.opencode/plugin/tests/context_trim.smoke.mjs` (FIXTURE DB, never
the live one; the fixture's table shapes come from a bounded READ-ONLY
inspection of the live DB — `.schema` + `LIMIT 5` selects, inspection only):
- Fixture session: ~12 messages before the marker (incl. ≥2 `read` parts
  carrying filePath/offset/limit, one `bash`, one `webfetch`), the full
  compaction quartet, ≥4 messages after the summary; FK cascade on; JSON
  `data` columns.
- Checks: report window fields exact; rows carry the tool targets; dry-run
  counts exact; the tail rewrite lands (part JSON tail_start_id = target,
  return line byte-exact); rejections — no marker / unknown id / target at
  or after the compaction row / floor violation (<6) — each with its reason.

Probe: a NEW section in `.opencode/plugin/probes/handover_probe.mjs` (next
number per the header): the tool's schema + report/tail contract pins on the
fixture (model the pinned-only-channel precedent S31); the header total is
machine-updated (baseline 346 → 346+N).

DoD:
- The context_trim smoke ALL PASS; the probe 346+N green (the 11 known #113
  env-fails 136-146 excluded); the other smokes UNCHANGED (no regression);
  ruff F=0; pytest status per #113 (report the measured result, do not
  block).
- Checkpoint commits per verified unit (code only); `TODO.md` entry #120
  status → LANDED + the code commits' hashes (NEVER your own final hash) +
  the handover in the FINAL commit.
- Handover: the registration snippet + the measured gate numbers + the
  live-acceptance note (report mode is read-only — the planner live-
  accepts it after the maintainer's restart + registration; the tail mode =
  a throwaway session, the maintainer's call).
- No new TODO entry is expected — but a discovered doc/code discrepancy gets
  filed (never promised clean).

DO-NOT-TOUCH: `opencode.jsonc` (registration = the maintainer's domain),
`AGENTS.md`, `agent/prompts/**`, `maintainer/**` (incl. its uncommitted
work), the live DB (read-only inspection only — NEVER a write), the FST
files, existing probe/smoke sections (add new only — never re-pin others).

First action: the loop_log START line (role worker-39, the task oneliner) —
the loop folder is `loop/autorun-2026-09-21_15-33/` (the spec is there as
`plan39_ho_task.md`). Stay on the current checkout.
