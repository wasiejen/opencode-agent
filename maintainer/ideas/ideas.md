--wip--defer: can be ignored if it prevents work on autorun: scanning ans proposals/research/feedbacl always allowed
// --comment
// e.g. proposals are always allowed to write, test implementations as scripts in tmp also
// - in general all things you find that might lead to some general improvement or built up of actionable knowledge or helpful tools, etc can be done
//   - e.g. research: you (the planner) can go trough feedback, maintainer folder files, archive log and identify problems/opportunities/things-to-optimise and research them
// this is no priority sorting ... ideas are loosely grouped in topics but might contain crossrelevant snippets

2026-09-29_20-31: second git identidy for the agents? planner + worker`? or just agent?

2026-09-29_19-47:
- should i git crush the whole opencode_test branch of FST?
- remove branches fst_work, fst_work2
- merge into opencode branch or async branch?

(stached into ideas to not let it be the first and only item in priority - so the agent can work more freely)
# 3 3 destillation of knowledge
2026-09-29_19-25
- find a good place to stash all the session and filter the ones out that are mostly empty
  - with the folder movement of root, the existing connection of opencode to this repo is broken and thus (for me at least) i can no longer see the old sessions. might be a non issue for you but just a headsup in case it is a problem.

- use the same worker as yourself to keep switching cost low by preventing cache invalidation on model switch

- you are free to test out different destillation goals and experiment with different goals/tasks more, report formats. (make a plan of what would be useful information and what to test)
  - you are free to experiment and document what you find
    - e.g. let multiple workers with different settings run over the same session and compare their output
  - create different tools/scripts to work with the dumps. how to best extract what you and the agents might need and what the workers might need.

- i am also inclined to reset the opencode db after we dump all the sessions - it is already 2.6GB big.

# knowledge and memory
## prompt additions/edits/rewrites:
## safe knowledge when you gained it! 
- motivation: when you e.g. researched how an object is resolved and it is needed for solving a problem, this needs to be documented somewhere -> knowledge base
- addition in the agents.md, that general knowledge - actionable items,code,facts that helped to solve a problem should be send via submit directly after confirming it and it is needed for a problem solution
  - when looking for solutions one of the first things should be to grep the knowledge in the folder for relevant hits (remember to limit outputted lines for first grep call or similar)
    - knowledge folder may need to get keywords? or would a tool with e.g. increasing resolution and window of needle search be useful?

- do we need dedicated knowledge search tools? like a needle search with keywords that automatically give back a specified range of found knowledge in the knowledge base or on lean mode to just return number of hits with some examples?
  - knowledge base might need to be built up completely different for this. keyboard unique identifiers to be able to use tools the retrieve them

## codify knowledge gain and how to save it
  e.g. keywords to be easily searchable because the knowledge files could get very big
    specific instruction for retrieval
    - storage would be done in a knowledge inbox with recommendation for keywords
    - dedicated agent with skill knowledge_curator would then go over this and collect it from other places to built up the knowledge files - updates them and dedubles - optimises based on collected feedback of retrieval attempts from agents
    - feedback needs to be again a mandatory step - needs to be easy and frictionless?
      - feedback_tool that just appends to the inbox???

# other stuff

2026-09-27_20-08:
- adapt the current version of opencode to respect the keepToken setting
  - create a fork
- what even is the most up to date branch currently? there are nearly 2000 branches
  - should we migrate to v2 instead of adapting the now very little updated v1?

2026-09-26_07-50:
- explorer prompt needs a general makeover as a researcher, information finder, map creator for e.g. apis
  - should generally only have a very limited write access, not shell/bash?

2026-09-25_14-33:
- explorer prompt needs to be updated ? mh do i even need the explorer? same model to prevent
  - compaction protokol, file references, worker

2026-09-24_21-13:
- include infos about auto-resume in the compaction handout and system prompt
  - e.g. what unit 2 and 4 actually do, how the restart of the planner after compaction works, how a new planner is started
  - move it to knowledge folder together
  - and include changes of handout into agents.md

2026-09-23_05-19:
try to be more creative and experiment a bit with different appoaches to problems - to use context this was is never a waste - when you learn something from it save it in memories.

2026-09-23_04-20:
- marker sweep of a planner took 17k token as result
  - $ grep -rn -- "--main\|--now\|--info\|--todo\|--defer\|--wip\|--comment" --include="*.md" .opencode/ TODO.md README.md WIKI.md 2>/dev/null | grep -v "_past_priorities\|/done/\|agent_feedback\|nap_direct\|archive/"
  - The output of the marker sweep got polluted with plugin.log noise 

-pathfinder mentality as planner prompt part - when you are in an area (files/folders) and you see something is bad or not current or is easily fixed - leave it in a better state then before.
- but might distract from task. small edits yes, bigger ones todo_inbox?

- need a dedicated research agent
  - prompt and instruction set/skillset inclusive?
    - analyse how the word with fuzzy_numword was done
      - check the sessions and curate a research guideline. like the tasc_spec in function only describing how best to research a topic and write a comprehensive proposal
      - good starting point the planner created in a dedicated research session was:
        - .opencode\agent\research\fuzzy-numword\drafting\2026-09-16_fuzzy-and-numword-tool-reliability.md
      - then discussion phases and addendum while preserving history
      - and then creating a compresehensive summary with reasoning and draft of Phases
        - Phases based on task_spec in size and strucute
  - ideal working flow. i destribe an intented function, write some thoughts down and an agent checks viability, seaches in the internet (context might be too tight - might need to upgrade gemma to 256K (found a way to do so and gemma is fast)) and creating a comprehensive overview with potential problems, usages, benefits -> an analysis if this is workable, how much work it would need and what we could expect as return in worth (e.g. smoother interaction, less friction with tools,)

- using event hook messages.updated instead we might calculate the real token fill in real time!
  - this needs to be checked
    - apparently its the way the opencode TUI does this

# prompt and tool description engineering (research) DONE
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
  https://www.anthropic.com/engineering/building-effective-agents
  https://www.anthropic.com/engineering/multi-agent-research-system
  https://www.anthropic.com/engineering/writing-tools-for-agents
  - create a comprehensive guide on how to optimise system prompts for out agents, how to structure our added prompts via read and how to optimise tool descriptions for better usage
  - this is the knowledge basis for a prompt engineer skillset
    - hw should get all the knowledge to be able to optimize all the prompts, tool, plugin descriptions - essentially everything that
  - e.g. from writign-tools-for-agents
    - "When writing tool descriptions and specs, think of how you would describe your tool to a new hire on your team. Consider the context that you might implicitly bring—specialized query formats, definitions of niche terminology, relationships between underlying resources—and make it explicit. Avoid ambiguity by clearly describing (and enforcing with strict data models) expected inputs and outputs. In particular, input parameters should be unambiguously named: instead of a parameter named user, try a parameter named user_id"
