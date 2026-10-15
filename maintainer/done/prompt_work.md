feedback on the AGENTS.md, agent_planner.md and agent_worker.md from Perplexity AI:

discuss these with me the maintainer and adapt the prompts accordingly.
further to discuss:
- a potential inbox_maintainer folder in which you can create questions, instruction or general things that I need to be made aware of. would need to be added in the AGENTS.md or planner prompt.

feedback:
"
## Overall assessment

Your workflow is unusually mature for a local, serial agent system. It has the right core properties: explicit roles, committed state, bounded handovers, approval boundaries, context management, append-only findings, and recovery after compaction. The planner-worker split is especially appropriate for a 27B local model because it reduces the amount of simultaneous responsibility each session must track.

The main weakness is not missing rules. It is **protocol surface area**. Three large prompt layers—the shared `AGENTS.md`, role prompt, and distilled memory—can create precedence conflicts, duplicate instructions, and retrieval competition. Research on agent workflows also identifies context overflow, retry loops, unclear escalation, and weak observability as common operational failures. [jetbrains](https://www.jetbrains.com/pages/ai-agents/architecture/agentic-workflows/)

## Strong design choices

### Committed state

Using files and `git log` as the resume contract is stronger than relying on conversation memory. Your NAP, task spec, worker summary, and commit history form a recoverable state machine.

### Role separation

The planner owns goals and plan state.
The worker owns implementation.
The worker never edits the NAP.
This reduces accidental cross-role edits and keeps the serial chain understandable.

### Explicit handovers

The worker summary contains changed files, verification, commit hash, TODO entries, and omitted work. That is the correct information boundary for a fresh planner session.

### Context-aware operation

Your workflow treats compaction as routine maintenance instead of a failure. The early-handover rule is particularly valuable because it creates a recovery point before the context limit becomes urgent.

### Access boundaries

Your no-circumvention rule is excellent. An edit restriction must be treated as an authorization boundary, not as a technical obstacle.

## Main weaknesses

### 1. Multiple sources of authority

You currently have at least three instruction layers:

1. `AGENTS.md`
2. the role prompt
3. the distilled role memory

The risk is not only contradiction. It is **stale duplication**. A rule may be changed in `AGENTS.md` but remain active in memory or in a role prompt.

Define a strict precedence order near the start of every role prompt:

```text
Authority order:
1. Current maintainer instruction.
2. Repository code and enforced tool permissions.
3. AGENTS.md.
4. This role prompt.
5. Distilled role memory.
6. General model knowledge.

When two sources conflict, follow the higher source.
Record unresolved conflicts in the handover or NAP.
```

I would place repository code and tool permissions above `AGENTS.md`, as you already do in practice.

### 2. Too many historical rulings

Dates, issue numbers, maintainer names, and historical reasons help explain why a rule exists, but they consume context and can make old incidents look like current commands.

Move historical rationale into a separate decision log.
Keep only the current operational rule in the active prompts.

For example, retain:

```text
At approximately 80%, checkpoint the handover and commit before continuing.
```

Move the date, incident number, and historical explanation elsewhere.

### 3. Repeated compaction mechanics

The shared file contains general compaction rules.
The planner and worker prompts then restate many of them.

Keep the shared file responsible for:

- what compaction is
- general budgets
- `keepMessages`
- commit-before-compaction
- recovery semantics

Keep each role prompt responsible only for:

- when that role should checkpoint
- what file it must update
- how it resumes
- what it must not do after compaction

This avoids three slightly different interpretations of the same threshold.

### 4. Hard-coded thresholds

Values such as 70%, 80%, 90%, 95%, six calls, and ten calls are useful operationally, but they can become stale when the model, context window, output speed, or host changes.

Prefer named policy variables or one canonical configuration file:

```text
Use the live context nudge when available.
Otherwise follow the thresholds in AGENTS.md §Context Budget.
```

If the exact thresholds are operationally important, define them once in `AGENTS.md` and reference them elsewhere.

### 5. Verification is still partly trust-based

The planner checks `git log`, files, and reported test results, but the worker still supplies much of the evidence.

Strengthen the handover with machine-verifiable fields:

```text
Commit: <hash>
Working tree: clean | dirty
Verification command: <exact command>
Verification result: PASS | FAIL
Verification evidence: <bounded output path or summary>
Changed files: <exact paths>
```

The planner should independently verify the commit and changed-file list.
It should rerun only targeted checks unless a full gate is required.

### 6. No explicit idempotency rule

A resumed planner or worker may repeat an action that already happened.
Your committed-state model helps, but the prompts should state the behavior directly:

```text
Before repeating an action, check committed files, git status, git log, and the handover.
Do not repeat a commit, migration, submission, or external action that already succeeded.
```

This is especially important after ambiguous tool failures.

### 7. Ambiguous failure classes

You distinguish context-limit failures from failed starts.
Add a compact failure classification:

```text
Classify failures as:
- transient tool failure: retry once with bounded output
- context failure: compact or resume the same session
- permission failure: stop and report the boundary
- verification failure: diagnose and repair
- maintainer decision: pause and ask or create a proposal
- unknown failure: preserve state and escalate
```

This prevents every failure from becoming another implementation attempt.

### 8. Missing retry and escalation ceilings

Your workflow has strong context limits, but it needs a general attempt limit.

Add:

```text
Do not retry the same failing action more than twice without new evidence.
After two repeated failures, preserve the failure, update the handover or TODO inbox, and escalate.
```

A bounded retry rule is a standard protection against loops in agentic workflows. [taskade](https://www.taskade.com/blog/agentic-workflows-explained)

### 9. Marker scanning may be noisy

The marker system is powerful, but broad text searches can detect examples, historical documents, and documentation about markers.

Your filtering is already careful.
The stronger design is to define marker scope structurally:

- active source directories
- excluded archives
- excluded generated files
- excluded feedback and history
- exact marker syntax

If practical, use a small script that emits normalized marker records instead of relying on a large shell pipeline.

### 10. NAP and loop log can diverge

You use the NAP, loop log, handover files, summaries, and git history.
That is robust, but it creates several state representations.

Define the authority for each fact:

| Fact | Authoritative source |
|---|---|
| Current planner phase | NAP |
| Worker task status | Task handover and git |
| Worker result | Committed worker summary |
| Session activity | Loop log |
| Historical work | Git and archived summaries |
| Open maintainer decision | `TODO.md` or proposal |
| Current priority | `maintainer/priority.md` |

Add a rule that agents must not “repair” one state source from another without recording the discrepancy.

## Recommended additions

I would add these short rules to `AGENTS.md`:

```md
## Authority and Recovery

Use this precedence order:

1. Current maintainer instruction.
2. Repository code and enforced tool permissions.
3. `AGENTS.md`.
4. The role prompt.
5. Distilled role memory.
6. General model knowledge.

When sources conflict, follow the higher source.
Record unresolved conflicts in the NAP or handover.

Before repeating an action, inspect committed files, `git status`, `git log`,
and the relevant handover. Do not repeat an action that already succeeded.

Do not retry the same failing action more than twice without new evidence.
After two repeated failures, preserve the evidence and escalate.

Classify failures as transient, context, permission, verification,
maintainer-decision, or unknown. Apply the corresponding recovery path.
```

I would also extend the handover contract:

```md
Required verification fields:

- commit hash
- working-tree status
- exact verification command
- PASS or FAIL result
- bounded evidence location
- exact changed-file list
```

## What I would not change

I would not add parallel workers yet. Your serial design deliberately avoids concurrent file ownership, merge conflicts, and coordination overhead. Parallel systems need shared task state, dependency tracking, locking, and stronger observability; otherwise coordination becomes the new bottleneck. [addyosmani](https://addyosmani.com/blog/code-agent-orchestra/)

I would not remove the NAP, handover files, or committed-state rule to reduce prompt size. Those are the structural elements that make recovery reliable.

I would not place every historical lesson into the active prompts. Keep active prompts operational and move rationale, dates, and incident narratives into a versioned decision or lessons file.

## Priority order

Address the weaknesses in this order:

1. Define authority precedence across `AGENTS.md`, prompts, and memory.
2. Add idempotency and retry ceilings.
3. Standardize machine-verifiable handover fields.
4. Centralize compaction thresholds.
5. Define authoritative sources for NAP, loop log, handovers, and git.
6. Move historical explanations out of active prompts.

Your architecture is already sound. The next improvement is not another large rule set. It is making the existing protocol **more authoritative, less duplicated, and easier to verify mechanically**.

"
