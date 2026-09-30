# plan39 NAP excess (2026-09-29/30 direct, ses_f110d8065ffeCVoCfNeTcba0wU)
Compressed from the NAP current-session section at plan40 close (the plan39_summary.md +
git 2d4480e..1ea1f23 carry the code/gate side; this file carries the design-discussion
excess).

## context-trim ROUND-2 feedback (planner-direct, point-by-point on his 8 comments)
1. The backstop correction CONCEDED — context_recovery = the live backstop per #93;
   the "no backstop" claim was host-native-only; the backstop covers only the overflow
   direction, the silent-too-little case is why the fail-closed validation stays primary.
2. keepMessages floor 6 AGREED (code constant + description line).
3. TUI cost (a) = nothing to build (the TUI already shows the full DB history).
4. The self-prune use-case = the strongest argument to PROMOTE the state-list fork
   from "only if needed" to PLANNED FOLLOW-UP (1+2 alone give only the coarse tail
   boundary + the picker — fine per-message dropping needs the fork).
5. Model toggling = an idea, three parts — (a) per-session choice possible today,
   (b) mid-session per-step switching plausibly feasible (#80 live finding: the host
   runs a turn as the agent+model in the message's `info`), (c) no clean "slowing"
   trigger signal today → research pass queued (one question: where the host reads
   the per-step model).
6. Shallow dev-fork = factually right, nothing to change.
7. Item 4 (the 11-line wire) DEAD unless item 2 fails.
8. The report file/line-info addition = NOT planned, ADDED to the spec (the DB part
   rows carry the tool inputs: read path+offset/limit, bash command, webfetch URL).
`--comment:` → `comment:` per the acted-on convention; `--wip` LEFT (his marker —
treated as stale: the file is his-committed at fe12552 with no uncommitted changes;
flagged to him in the close).

## His ruling (2eba484 + the `approved/` move)
1. "go :-)" — units 1+2 approved for the build.
2. Unit 3 = agreed as the follow-up learning fork ("let see that the 'easy' part works
   and we have an immediate gain").
3. Model toggle = CLOSED, no research — his clarification: he meant a STATIC switch
   to a faster model (select model + send message; the model/agent apply on the next
   idle after a tool call → a complete reprefill → "not very interesting atm"; agent
   and model are separately setable per message; per-step mid-session toggling is not
   planned and would break the same-model cache planner-worker workflow).

## Inbox handled
`compact_message_worker.md` (unmarked observation — "the worker's compact message is
never delivered and thus never consumed" + 2 what-if bullets: auto-delete consumed
compact messages; the queued message as a deliberate-self-compact-vs-context-limit
signal) — triaged per the observation rule: NAP record + research queue (idle lane),
moved to `maintainer/done/` content-untouched.

## Observations (NAP-only, no action)
- New unmarked ideas.md observation (his file): 2026-09-29_23-19 "compaction model
  as filters option in llama-swap? no reload needed and thus no cache invalidation.
  can i set checkpoints to 0 to preserve the planner and worker session in the
  limited ram cache?" — a backend-config thought (his domain; the inference-server
  config is never touched by agents).
- His worktree state (NOT staged by me): the proposal move `commented/` → `bin/`
  (622cade) → `approved/` (his uncommitted move — the file there carries my worktree
  edit of the acted-on `--comment:` → `comment`, riding his move commit) +
  `maintainer/ideas/ideas.md` live edit (uncommitted — read, not staged).

## Watchdog ruling
Unit-4 watchdog fired on my earlier close (open questions, no `action:` line):
RULING LANDED — no Autorun/Direct marker → the default scope IS the Autorun one; the
firing was EXPECTED behavior, not a gap (my friction entry's fix suggestions are void
— corrected in the feedback file); that session is thus the autorun-scoped plan39
(the last loop-log planner-N + 1, per the counter rule).
