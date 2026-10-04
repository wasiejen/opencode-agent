# Agentic Compact Trim aka ACT 

sketch of the Unit 3 of compact trim projected trajectory.

add message numbering as append to each message 
-> reduces the need to have a very detailed message overview in report and thus reduced its size considerably
- question: is the agent already aware of the numbering? or the messageID? or any info that is actually sitting in the meta data json of the DB of each message?

3 part toolset via one plugin (naming wip)
1. act_info
2. act_set_filter
3. act_apply

functionality:
1. act_info: report functionality of compact_trim in general and show currently set filter for still active windows
2. act_set_filter: to set the filter on a per message basis without applying the changes. makes it possible to gather messages to trim over the session instead of deciding at the end when the memory might be fuzzy about the middle context (known issue - start and end the sharpest - middle gets fuzzy with filling context window)
- makes it possible to gather to trimmed messages over the context and thus make an informed trim even in contextlimit case
  - on close to context limit now the only necessary thing is to update the filter or to apply it ... no long deliberation needed anymore when context is tight already.
  - moves decicions what to keep and awareness what can be dropped closer to the actual content that is dropped and not to gather in the end of the context window
- might need a second json meta data entry in the session DB to make it interruption prove
3. act_apply: single function to apply the current selection and thus start the trim and reassembly of the context; deliberate seperation to prevent accidental firing
- feedback is a small overview of the disabled/inactivated messages and e.g. what file reads have been removed? and info which messages are still active?
  - then on reassmebly act_info and acat_set_filter messages can be removed also automaically because these are replaced by the feedback info of acat_apply (no need to keep stale info)
    - so act_info and act_set_filter are automatically marked as to be trimmed (always). also the act_apply for the next trim.
      - might be problematic in parallel tool calls? i do not know have these are handled in the DB. each parallel tool call is its own message??

- this leads to the change of the nudge messages
  - from reminding to compact to reminding to select messages to discard and to inform if no messages are selected yet to discard. also to remind to fire acat_apply when closing on context limit.
- if no messages are marked to be removed and automatic decision is done.
  - beginning might be similar to compaction - just keep system prompt, starting reads from readme folder, and last <N> messages
  - then to an informed approach based on the message content. earlier and bigger messages are preferred to drop than more current ones. reads and tool calls prefereed to thoughts and a min token amount to be dropped is to be reached by this. (wip)

- open question: cross trim?? do all 3 tools work with an optional session id provided?
- 

Planner replies (2026-10-04, plan10, ses_f0777c6d0fferrQ5Skto9dmzn0 —
feedback per the `--comment` request, grounded in the measured
context-package state U0-U4):

1. **ACT is the surface + policy layer for the package's U2 — not a new
   mechanism.** The approved package
   (`proposals/approved/2026-10-01_context-budget-package.md`) already
   carries the trim mechanism: U2 = the per-message context filter AS A
   FORK CHANGE, non-destructive, filter state = a JSON field on the
   message `data` blob (your ruling), the fork assembly skips filtered
   messages, kill switch `.opencode/temp/filter_off`, live gate = your
   `build_fork.bat` (bun currently broken — U2 is queued behind that).
   ACT's act_set_filter/act_apply are exactly the agent-facing surface
   for that mechanism: act_set_filter = write the `data`-blob field
   (the write path already exists — context_trim's core does the exact
   single-transaction DB write, fail-closed validation); act_apply =
   make the filter effective for the next assembly (the fork's job).
   Recommendation: build the act_* surface against the U2 design so
   there is ONE mechanism (the data-blob field), ONE surface (act_*),
   ONE policy (nudge + fallback) — instead of two parallel filter
   stores.

2. **Your numbering question — the measured answer: none of the three.**
   The agent in the raw window sees neither numbering, messageIDs, nor
   DB metadata — the context is plain text. That is exactly the gap the
   package's U1 report leg was built for (numbered rows 1..N, tool-
   target detail, the number→messageID map stays INTERNAL — the agent
   never sees or types the ids). Two complementary pieces for ACT:
   (a) the numbering leg is feasible WITHOUT the fork — the research
   doc for #105(d) (`agent/research/2026-09-28_compaction-summary-
   customization.md`) measured that `experimental.chat.messages.
   transform` gives plugin-side control of the assembled messages; a
   transform can append a stable number to each message. Use a STABLE
   number (e.g. short hash of the messageID or the session-absolute
   position), NOT a window-relative index — window positions shift on
   every trim/compaction, session-absolute positions never do.
   (b) the in-window number answers "which message", but only the
   report can answer "how heavy" (token mass / droppability) — keep the
   report for the mass question; the numbering makes the report's row
   numbers referable in-window.

3. **Your middle-fuzziness diagnosis is right, and it is the ACT
   surface's real value.** U1's biggest-droppable list makes the END-
   of-window decision cheaper, but by then the content knowledge is
   already degraded (you named it). act_set_filter's "gather marks over
   the session while the content is fresh" is a better decision channel
   — and it changes the U4 nudge semantics (currently live in your
   config: nudge1-4 = 0.80/0.90/0.925/0.95) from "compact now" to
   "mark droppable messages now / fire act_apply at the limit". That
   is a one-line policy change on the existing rung, not new plumbing.

4. **Your fallback auto-decision already exists — it is U3.** "Keep
   system prompt + starting reads + last N messages" is precisely the
   landed U3 marker compaction (agent-written summary + tail re-derive,
   one budget compaction, zero restart). The ACT fallback needs no new
   code: it is a routing decision in the nudge policy (no marks pending
   → route to the U3 marker; marks pending → route to act_apply).

5. **"Interruption-proof" — your open concern resolves for free.** The
   data-blob field (the U2 store, your own ruling) lives in the DB, not
   in plugin memory — it survives restarts by construction. No second
   JSON metadata entry is needed.

6. **act_apply's self-trim of the act_* messages: good, one dependency
   to verify first.** The parallel-tool-call granularity question you
   asked ("each parallel tool call is its own message??") decides the
   auto-trim rule: in the DB each tool call is its own assistant
   message (verify against host-map §3 before pinning the rule) — if
   parallel calls ever share one message, the "mark this turn" rule
   needs the part level, not the message level.

7. **Cross-trim: yes — consistent with the existing surfaces.**
   context_trim already takes an explicit session id (report/target
   forms work cross-session); compact_memory has the cross path.
   act_* should take the optional session id too — it is the same
   pattern, no new mechanism.

8. **Suggested sequencing (effort order, each leaving the repo green):**
   S1: U2's fork lands (your `build_fork.bat`, queued behind the bun
   repair — maintainer domain). S2: act_set_filter surface (write the
   data-blob field via the context_trim core write path; inert until S1
   — pre-marking while inert is already useful: the first post-fork
   run starts with a warm filter). S3: act_info (U1 report + current
   filter state of the active window). S4: act_apply (fork-gated; the
   pre-S1 fallback = the U3 marker routing per 4). S5: the nudge-policy
   reword + the numbering transform. S2-S4 can be built before S1
   against the agreed data-blob schema (smoke on fixture DBs, the
   established pattern) — the fork only gates LIVE effect, not the
   surface build.

- 

comment: feedback pls
