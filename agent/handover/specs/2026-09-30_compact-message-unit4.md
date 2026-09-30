# Task spec — compact-message-delivery item 4: restart-branch inheritance (TODO #127)

Worker: worker_Q3S (170K). Pre-approved class (agent-usage). Queue order:
after the current `handover_task.md` (#126). Branch truth: stay on the
current checkout.

## Goal

The auto_resume unit-4 restart branch appends the CLOSING session's queued
`compact_message_<sid>` to the successor's restartText (closing-intent
inheritance), marks the file consumed (the `.consumed` tombstone), and stays
fail-open.

## Ruling + design context (settled — do not re-litigate)

- Maintainer ruling (2026-09-30, comment in
  `proposals/approved/2026-09-30_compact-message-delivery.md`): IN FAVOR of
  item 4 ("this appending would be more truthful").
- Planner's interference analysis (his question, answered in the direct
  session): NO blocking problem with the planner instructions — the
  appended text is a labeled INTENT HINT; the successor's protocol already
  rebuilds reality from committed state first (a stale intent is corrected
  by the file check); the size is bounded to one queued message.
- Items 1-3 already LANDED (`f96a39c` STEP-0 protocol / `b025159` zombie
  guard / `0a89e7c` age sweep) — the relay path and those behaviors are
  UNCHANGED by this task.

## Verified facts (measured at spec time, 2026-09-30)

- The restart branch: `.opencode/plugin/auto_resume.ts` —
  `restartText(sid, exhausted)` L898-915 (pure string construction), call
  site L1409 `spawnPlanner(restartText(sid, exhausted), msgs)`.
- The queue file: `.opencode/temp/compact_message_<sid>` — the SAME `logDir`
  the plugin already uses for `COMPACT_BUDGET_FILE` (L240). The `.consumed`
  tombstone convention = item 3 (`0a89e7c`, the age sweep).
- The existing unit-4 restart-branch pins live in
  `.opencode/plugin/tests/auto_resume.smoke.mjs` (current total 147/147).
  AUDIT the restartText/spawn pins BEFORE adding yours — any pin asserting
  the exact base text is behaviour-pinning (the absent-file case must stay
  byte-identical).

## Design (suggestion — the HOW is yours within the DoD)

- At the L1409 call site or inside `restartText` (your choice): if
  `compact_message_<sid>` exists, read it; append a labeled section (e.g.
  a line naming it as the closing session's queued message for `<sid>`,
  consumed by the restart branch, to be treated as an INTENT HINT — verify
  state from committed state before acting — then the content); then rename
  the file to `compact_message_<sid>.consumed`.
- Fail-open: file absent / unreadable → the base text unchanged, no throw,
  no tombstone.

## DO-NOT-touch

- The relay delivery path, items 1-3 behavior, the budget file, the
  maintainer's live files, anything under `maintainer/`, `agent/prompts/**`.

## Definition of done

1. Code: the append + rename, fail-open.
2. Smoke: 3 new pins — (a) file present → the spawn text contains the
   labeled section AND the file is renamed `.consumed`; (b) file absent →
   the restartText/spawn text byte-identical to the base (existing no-file
   pins unchanged); (c) unreadable file → the base text. The existing
   147/147 stay green (150 total).
3. Standard gate green: probe 352/352 + all 11 smokes.
4. Checkpoint commits per verified unit; the TODO #127 status + the handover
   summary ride the FINAL commit.
5. Live acceptance = the next restart-branch spawn with a queued file
   present (natural occurrence) — name it in the handover.

Context discipline: bounded reads only (the named regions + the smoke's
restart-branch pin section); no broad research.
