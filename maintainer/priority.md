# priority — what I want next (my ordering, top = first)

## My list (active)

<!-- Add numbered lines here: reorder = reprioritize, delete = no longer
     wanted. Entries may name a TODO.md ID, a proposal file, or just a theme. -->

## How this works (for agents — read this, it is the whole convention)

1. The planner reads this file at session start and orders its planning by
   the list: the top item is the next work when it is clear and does not need
   the maintainer to clarify; the list is also the tie-breaker between clear
   `TODO.md` candidates. `TODO.md` stays the detail record; this file is only
   the ordering.
2. You may only EDIT this file for one thing: removing a line you have FULLY
   handled. Never reorder, reword, or add his lines.
3. When you move a handled line, APPEND it to `_past_priorities.md` (same
   folder) with a SHORT reply — one line, e.g.:
   `- attention-keywords proposal — done, implemented (commit 6ef2c4e)`
   `- #37 loop-log anomaly — see TODO.md #37, fixed in plan 3`
   For easy tasks `— done` is enough; for anything with a trail, point at the
   proposal file / commit / TODO ID. Append only — do not re-read or edit the
   old entries.
4. PARTIAL or BLOCKED items stay in the active list; note their status in the
   NAP (not here). The maintainer erases entries in `_past_priorities.md`
   whenever he wants — it is his log.

--maintainer: if no actionable item remains in an autorun:
- --wip/--deferred is lifted if it prevents work
    - e.g. proposals are always allowed to write, test implementations as scripts in tmp also
- in general all things you find that might lead to some general improvement or built up of actionable knowledge or helpful tools, etc can be done
  - e.g. research
    - you can go trough ideas feedback, maintainer files, archive log and identify problems/opportunities/things-to-optimise and research them
- use the sessions with nothing to do - do maintenance on your nap and todo
  - e.g. think about how to improve the set of prompts we have. are their ways to better organize them. to make them more succinct and clear. are there redundant instructions or badly worded (only examples)
- you are intelligent - you will find something to do
  - go through my ideas for new research on functions. create a folder in research for each if you find something worthwhile
