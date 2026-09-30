# compact-message delivery — research (2026-09-30, planner-41, ses_f102c4a2dffezocrIY7kzd44YA)

Question (maintainer observation, `maintainer/done/compact_message_worker.md`):
"compact message of worker is never delivered and thus never consumed" +
(1) could consumed compact messages be deleted automatically? (2) queued
messages as a source of info whether a compaction was deliberate (self-
compact) or caused by a context-limit violation.

## Measured on-disk evidence (2026-09-30, `.opencode/temp/`)

| file | session | fate |
|---|---|---|
| `compact_message_ses_f110d8065…U` | planner-39 (in-scope autorun) | **`.consumed`** — relayed twice (`relay=` 2026-09-29T22:32:45Z / 22:51:09Z) |
| `compact_message_ses_f12b5172…xj` | worker #117 (Task tool) | unconsumed — self-compact 14:05:51Z, resumed via `task_id` |
| `compact_message_ses_f130ae200…qE` | direct session (scope none) | unconsumed — self-compact 14:59:49Z, session ended (never resumed) |
| `compact_message_ses_f10a06089…w9` | worker-39 (Task tool) | unconsumed — self-compact 23:41:00Z, resumed via `task_id` |

The observation is confirmed: every worker (and direct-session) queued
message is stranded. Only the in-scope recovery path ever delivers.

## Mechanism (source-verified)

- **Queue write** — `compact_memory.ts` L303-310, L494/L523: the `message`
  arg is written to `.opencode/temp/compact_message_<sid>` **at dispatch
  time, unconditionally** — before the fire-and-forget compact call settles.
  A failed dispatch (terminal error line, no budget increment, no COMPACT
  line) leaves the file behind: **file presence ≠ a compaction happened**.
- **Delivery** — `auto_resume.ts` L1349-1357: the file is read ONLY in the
  unit-4 CONTINUE (recovery) branch — `action === "resume"` or no action
  line at idle. Stored message = first message of the resumed turn + one
  line addendum; `relay=` logged; rename to `.consumed` only on a successful
  send. Single call site (grep-verified).
- **Gates before it** — L1303-1307: `w.scope === "none"` → never acted on.
  Measured (#109): Task-tool workers carry `scope= none`; direct sessions
  are scope "none" too. The Task tool's `task_id` resume is host-side and
  bypasses the plugin entirely.

## Root causes of the non-delivery

- **(A) scope-none sessions** (all Task-tool workers + direct sessions):
  excluded by the L1303 gate — their queued messages can never be relayed.
- **(B) in-scope restart closes**: if a compacted in-scope session continues
  and closes with a recognized `action: line (restart/resume/stop), routing
  takes that branch — the restart-spawn branch builds the successor prompt
  from `restartText()` and **never reads the closing session's queued
  message** → the message is silently stranded on disk (code-verified, not
  yet live-observed — the only in-scope self-compact so far, planner-39,
  closed via a no-action-line recovery first).
- **(C) zombie files**: a queued message exists even when the compaction
  dispatch failed (write-before-dispatch). The relay does NOT check that a
  COMPACT line exists for the sid — it delivers the stored message on the
  next no-action-line idle whatever the reason. Measured: planner-39
  (ses_f110d8065) — file present + relayed, but **no COMPACT line in
  ctx.log, no budget increment** (the dispatch failed; terminal error line
  not preserved). The "queued continuation message" was injected into a
  turn with no actual compaction behind it.

## Answers to his two questions

1. **Auto-deletion of consumed messages** — the `.consumed` rename already
   acts as a tombstone; a small age/orphan sweep at plugin init would keep
   the temp dir clean: delete `compact_message_*.consumed` older than N
   days, and orphaned unconsumed files whose sid no longer exists (or is
   older than N days). Caution: immediate deletion on consume would destroy
   the evidence used by (2) below — keep `.consumed` + age-sweep beats
   delete-on-consume.
2. **Deliberate vs limit-driven compaction** — a three-way discriminator
   (signal, not proof — a deliberate self-compact WITHOUT a `message` arg
   leaves no file):
   - COMPACT line **+** queue file → deliberate self-compact with queued
     message (agent intent).
   - COMPACT line, **no** queue file → limit-driven / auto (nudge-driven
     self-compact without message, host auto-compaction, emergency backstop).
   - queue file, **no** COMPACT line → **failed dispatch (zombie)** — the
     compaction never happened.
   Sources: `grep "COMPACT" .opencode/temp/ctx.log` + `ls .opencode/temp/
   compact_message_*`.

## Fix options (ranked)

1. **Protocol-side delivery (zero code, covers A):** `readme_post_compaction.md`
   instructs EVERY session — on resume, whatever the scope — to read its
   own `.opencode/temp/compact_message_<sid>` first (if present) and rename
   it to `.consumed`. Uniformly covers workers + direct sessions (their
   resume channel bypasses the plugin; the protocol is their only channel).
   The relay stays as the in-scope recovery backstop.
2. **Restart-branch inheritance (code, covers B):** the unit-4 restart
   branch reads the CLOSING session's queued message and appends it to the
   successor's restartText (successor continues the closing session's
   queued intent).
3. **Zombie guard (code, covers C):** the relay delivers only if a COMPACT
   line for the sid exists in ctx.log since the file's mtime — OR
   compact_memory deletes the queue file on verified dispatch failure.
4. **Cleanup sweep (code, covers his Q1):** init-time age/orphan sweep as
   above.

Recommendation: unit = (1)+(3)+(4) (small, agent-usage class; (1) is a
prompt edit per the planner's prompt-edit ruling); (2) = follow-up
proposal (changes what a spawned successor receives — a design choice for
the maintainer). See `proposals/2026-09-30_compact-message-delivery.md`.

## Related findings (this session, from the same files)

- **#106 live-accepted**: `fuzzy-edit applied … d=1` + `hint rejected
  reason=d-too-high/no-anchor-line` lines live in intercept.log (4 events
  post-restart).
- **#119 live-accepted**: `compact_budget.json` — all pre-existing config
  keys survived the post-restore increment (14:59:49Z, ses_f130ae200); the
  pre-#119 code would have rewritten the file sessions-only.
- **#75 unit 2 live-accepted**: `nudge=` lines live (ratios 0.857/0.887/
  0.895 — the budget-file 0.85 threshold honored).
- **#75 unit 4 live-accepted**: `recovery=` fired + `route= restart spawn`
  with the correct `agent=planner_Q3S_slow` identity (planner-40 +
  planner-41 spawns, 00:05:45Z / 01:00:24Z).
- **Unmarked maintainer idea (NAP-only, no action)**: ideas.md 2026-09-
  29_23-19 — "compaction model as filters option in llama-swap? no reload
  needed and thus no cache invalidation. can i set checkpoints to 0 to
  preserve the planner and worker session in the limited ram cache?"
  (backend domain; the unmarked-remark triage applies.)
