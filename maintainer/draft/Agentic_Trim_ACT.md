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

--comment: feedback pls
