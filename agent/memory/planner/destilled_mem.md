# Planner memory — destill

Condensed planner memory, one line per entry; each line points at the full
entry (evidence + rationale + review conditions) in `memory.md` (same folder —
read it only when you need the full text). Write/review policy: `README.md`
(same folder). Never restate prompt/AGENTS.md protocol text here.

- MEM-0101: live acceptance of an observer-mediated mutation verifies BOTH sides — the mechanism side (intercept.log `orig=`/`value=` lines + on-disk artifact) and the perception side; you CANNOT see your own pre-mutation args, so a "repeat" in your own view is a false-repeat — cross-reference the log first, never self-perception; a feedback claim of "delivered AND applied" is likewise verified against intercept.log, never from the hint line (2026-09-30 addendum).
- MEM-0102: tool/plugin units close via the two-phase maintainer-domain handoff — your close-out NAMES every pending item verbatim (registration in the live opencode.jsonc, the AGENTS.md paste, live acceptance after restart); the REOPENING session verifies FROM FILES FIRST (grep the live config, read the pasted text, `git log`) and only then fires live acceptance — never mark a pending item done from a worker summary or memory.
- MEM-0103: maintainer files live in the shared tree and grow as thoughts mature (single-tree status-quo ruling 2026-09-18) — uncommitted content is NOT a task unless a marker/inbox/handover designates it; read freely unless `--wip` is placed (strictest handling: no edit, no stage, no read until cleared); named-path commits only, never `git add -A`.
- MEM-0105: in the serial (one-slot) setup a delegation is INSTANT to the planner (I block + resume; the worker experiences the runtime) — never reason about "how long the worker ran" from my perspective; judge from the committed state / the logs / a context-limit failure, not perceived time (MEM-0104).
- MEM-0108: NEVER launch ANY request at the backend inference server (the single llama-swap slot) — not for tests/probes, not even enumeration: a direct request evicts the live session's own model (measured 2026-09-24: my 80k prefill probe kicked my own model, returned empty; prefill was already covered by the maintainer's fork test). All model traffic via opencode sessions only.
- MEM-0111: the edit tool's hint line (`hint rejected … d=0 gap=inf`) is NOT the authority on apply-vs-reject — ALWAYS read the file back after any hint line before concluding rejection (2026-09-30 plan44/45 misattribution cluster).
- MEM-0112: a stale-reference sweep must diff EVERY index line against the actual `ls` of the target folder after a reorg — a pattern-only sweep misses files that do not exist in the target folder at all (plan46/47; the 2026-10-15 prompt rework shipped 3 stale paths of its own, caught by exactly this check).

## Spec authoring + host-fact triggers
- MEM-0113: a spec naming a SMOKE block as the pin site for a gate change needs an explicit PROBE-section pin audit too — name probe check ids in the spec (#128, plan2).
- MEM-0114: verify installed-library API signatures with one inspect call at spec-authoring time before citing them (plan3 #129 PySide6 ctor).
- MEM-0115: count Python logging call sites with BOTH greps — `logging.` AND `logger.` (plan4 #131: 4 vs ~14).
- MEM-0116: a `.opencode/tools/*.ts` landed mid-session is NOT live until the next host start — verify from files + spot smoke; live acceptance rides the NEXT session (plan10; today's bt mode micro-fix followed the path).
- MEM-0117: after a model/config swap the injected ctx budget suffix can transiently read cap 1 — trust the ctx_gauge self-read for budget decisions (compaction_core resolveCap L205-211).
- MEM-0118: the live host has NO in-process sqlite (bun:/node:sqlite both fail live) — every DB tool must ship the #126 spawn-sqlite3 last-resort adapter; fixture smokes cannot detect the gap.
- MEM-0119: an explicit maintainer worker-OK overrides the MEM-0110 no-delegation guideline; still verify --comment review drafts against current line ranges before believing them.
- MEM-0120: a model self-review of PASTED code mixes real catches with confident P0 hallucinations — diff paste-vs-deployed + check load/smoke proof before believing any P0 (2026-10-21 bt review).
- MEM-0124: spec pin audits = verdict-token grep (`out-of-sandbox|path-anomaly`-class) across EVERY pin file (smokes AND probe) — the first post-change gate FAIL list is the authoritative inventory, never range-recall (142/138 spec missed (B3)+probe 303/304).
- MEM-0125: typecheck pins need a NEGATIVE CONTROL — push a deliberate bad literal, confirm RED, then commit (a widened generic hook param silently accepts bogus literals; unit 3a).
- MEM-0128: cross-tree port specs — twin-freshness check (`cmp -s` + `wc -l` + bounded `diff | head -60`) BEFORE committing a design-of-record anchor; a stale clone twin silently pins stale behavior — anchor on the MAIN twin when the clone lags (#143, 5c re-baseline).

## Planner behavioral guidelines (direct sessions — CANONICAL HOME per the maintainer's 2026-09-28 move instruction; the NAP's scattered lines are superseded)
- MEM-0110: direct sessions = NO delegation without prior explicit maintainer OK (ruling 2026-09-27) — interact/answer first (verify shortly if needed), stop guessing and ask, push back without sugarcoating, question his assumptions; trigger = his post-compaction-reliability complaint about the fork planner that implemented the keepTokens metric rework before his review (the #99 "updated prematurely" remark).
- TODO refs in dialogs carry a SHORT TITLE (ruling 2026-09-30, direct session): e.g. "#126 context_trim spawn fallback" — he has no overview of the bare numbers; the title makes dialog self-contained without a TODO lookup.

## Worker failure-mode recognition (baseline — fires on EVERY worker return)
Recognition patterns, not situational memories: apply them the moment a
worker result arrives (empty result, a Work State summary, or an error) —
before deciding resume vs. relaunch vs. verify. Full evidence + review
conditions: memory.md.

- MEM-0104: a Task-tool "context_length_exceeded" failure ALWAYS means the sub-agent RAN and hit its context limit — the work may be COMPLETE in the uncommitted tree (the death can land just after the work, before the final commit/feedback); rebuild from files first (git status + working-tree handover + gate), then land the last steps yourself or resume via task_id (without compaction when the maintainer orders it); distinguish a genuine launch failure (e.g. the "prefill incompatible with enable_thinking" signature) by file state, never by the message alone.
- MEM-0106: a sub-agent that self-compacts mid-task returns the COMPACTION SUMMARY as the Task result (generated by the compaction model, NOT the agent) — recognize it by the `## Objective` Work State form (a normal worker closing never uses it) + the guaranteed ctx.log `COMPACT <sessionID>` line, then resume via task_id using `## Next Move` — no worker convention needed.
- MEM-0107: worker limit-death forensics = the dump's last step-finish meta (`reason=length` + big `output=N` = per-turn OUTPUT cap — the truncated write's file change never lands; `tokens.total` at window = the wall) + per-message byte sizes (giant planning-prose outputs can burn more than the reads); the in-flight write is salvageable verbatim (last `## msg_` header → EOF); hand the successor the on-disk diff + salvage + spec with chunked-write (≤ ~8KB/call) + section-only-read instructions — compaction+resume does NOT save re-emission and costs a same-model slot flush.
- MEM-0109: Task-tool `request (N tokens) exceeds the available context size (M tokens)` = the sub-agent RAN and hit the wall when a tool output was inserted into its context (hard rejection of the in-flight request — context + the new tool output > window): a normal limit death whose final push was a tool result, not the thinking part — the work is usually already staged in the uncommitted tree; rebuild from files first (git status + handover + a targeted spot gate), then land the commit yourself or resume via task_id (extends MEM-0104).
- MEM-0121: a worker's close NARRATIVE can FABRICATE references (nonexistent hash, restart claim, maintainer-instruction claim) even in a session that did real work — before recording any hash/restart/instruction, `git log --oneline -N` + `git show --stat <hash>` in that tree (extends MEM-0120 to narratives).
- MEM-0126: Task-tool "user rejected permission to use this specific tool call" = the sub-agent NEVER STARTED (target tree + git log untouched) — a LAUNCH failure, not a context failure: never compact/resume, re-issue ONCE with a SHORT spec-pointer prompt (long spec-echoing prompt suspected, medium confidence); twice → report the boundary.
- MEM-0127: before REUSING a worker session via task_id, check its close gauge — at ~89% of window the resume is REJECTED OUTRIGHT; cross-compact FIRST (keepMessages=10 + the new-unit pointer), verify the `COMPACT` line in `.opencode/temp/ctx.log`, THEN resume (plan18 4b→4c).

## v2 port era — file-surgery hygiene (2026-10-22 cluster)
- MEM-0122: block_transfer WRITE/REPLACE feedback (`+N line(s)` + first-line echo) is NOT landing proof — a long single-call prose payload can arrive TRUNCATED and silently no-op; read the file back after >~15-line writes, repair with a smaller `edit` (MEM-0111-class).
- MEM-0123: block_transfer buffers: name bufferName per artifact (an unnamed CUT silently destroys a staged 'default' buffer mid-chain); MAP/PEEK before REPLACE (`last_write` ≠ assumed content); on an empty-buffer error re-check the MODE list (WRITE = bufferless direct text); the N-regions+1-text half RESOLVED at plan15 (REJECT landed).
