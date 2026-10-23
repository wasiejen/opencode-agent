# Planner memory

Seeded 2026-09-18 from the memory-pilot seed instruction (direct session
ses_f4c039ae2ffeRqvdPqGu8IdB37). Seeds only: high-value lessons already VERIFIED from
actual planner work; nothing that is authoritative elsewhere (prompt /
knowledge / NAP) is re-stated.

### MEM-0101: Live acceptance is an OBSERVATION-side test — never verify against your own perceived args

- Type: `learned`
- Status: `active`
- Confidence: `high`
- Scope: planning / running / verifying any live acceptance of an
  observer-mediated mutation (numword escape, fuzzy resolution, any
  intercept_observer feature)
- Keywords: live-acceptance, intercept.log, observer, post-mutation,
  false-repeat, self-perception, escape
- Memory: an acceptance test for an observer-mediated mechanism has TWO
  sides, and the test must verify BOTH: (a) the MECHANISM side — the
  pre-mutation `orig=`/`value=` lines in `.opencode/temp/intercept.log` +
  the on-disk artifact; (b) the PERCEPTION side — the agent CANNOT see its
  own pre-mutation args (only the corrected form enters its context), so a
  corrected value in your own view does NOT prove you "wrote it twice" —
  that is a false-repeat. A unit is accepted only when BOTH sides pass; a
  "repeat" observation alone proves nothing either way — cross-reference
  the log first.
- Why it matters: in the 2026-09-18 escape acceptance the planner's own view
  showed it repeating `a7b c861d` literal three times (a perceived
  generation loop); the log proved all four writes carried the sentinels and
  all were `pair-resolved` — the feature worked the whole time. The
  perception mechanism (measured 2026-09-17, #73) + the maintainer's ruling
  (2026-09-18) make self-observation NON-ADMISSIBLE evidence for
  mutation-behavior claims. Future planners: put this cross-reference into
  any worker spec that verifies observer behavior, and keep it in your own
  acceptance protocol — otherwise a working unit gets re-tested, or a real
  defect gets waved through on "it looked like I typed the literal."
- Evidence: ses_f4c039ae2ffeRqvdPqGu8IdB37, direct session 2026-09-18
  (escape live acceptance); `.opencode/temp/intercept.log` 2026-09-18_12-19
  — 4x `kind=escape scope=content ... pair-resolved` lines with `orig=` the
  sentinel form and `value=` the field-2 digits, `hits=2`; corrected
  file-on-disk readback + unmarked control line byte-identical; maintainer
  ruling on the perception mechanism (same session). Related mechanism fact
  (the post-mutation `state.input` storage) already lives in
  `knowledge/knowledge_plugins.md` — this memory carries the TEST-DESIGN
  ruling, not the mechanism.
- Verified: 2026-09-18 (the acceptance run itself)
- Related: MEM-0102 (close-out protocol — acceptance is its final phase)
- Review when: the observer ever logs pre-mutation evidence INTO the
  session DB (perception gap closed), or acceptance moves to a harness
  (the P6 loop_stats draft).
- 2026-09-30 addendum (plan39/40/44 cluster; cures the 2026-09-30_04-49
  inbox entry): a feedback claim that "a hint-rejected line was delivered
  AND the edit was applied" must be verified against intercept.log before
  being treated as a tool bug — in both 2026-09-29/30 cases (intercept.log
  lines 265/421) the rejected edit call had actually FAILED; the applied
  state came from (a) a later successful edit call (plan39) or (b) an
  external unlogged edit (plan40 — commit 9444248 carries both
  repo_commands.md changes with zero logged edit calls; the maintainer's
  own editor is the likeliest source).
- Review addendum: if the cluster recurs AFTER the #106 build (7a9e291) is
  confirmed live, re-examine the note-delivery path (stale-note hypothesis)
  instead of closing as misattribution again.

### MEM-0102: A tool/plugin unit closes with a MAINTAINER-DOMAIN HANDOFF — the planner re-verifies from files at the next session

- Type: `learned`
- Status: `active`
- Confidence: `high`
- Scope: closing any unit that lands a tool/plugin/config change the
  maintainer must register or paste (submit, #0 escape, R1-R7.5, and any
  future tool build)
- Keywords: handoff, pending, maintainer-domain, registration, paste,
  restart, close-out, live-acceptance
- Memory: the close-out for such units is a TWO-PHASE protocol with a
  file-level boundary. Phase 1 (planner's, closable): build + gate green +
  bookkeeping + NAP updated + close-summary pending list NAMING every
  maintainer-domain item verbatim ("registration in the live
  opencode.jsonc + per-agent grant; LIVE ACCEPTANCE after restart;
  AGENTS.md paste <which line, where the draft lives>"). Phase 2 (maintainer
  runs it — registration/paste at his restart cycle): the REOPENING planner
  session verifies FROM FILES FIRST (grep the live opencode.jsonc for the
  tool grant; read the pasted text in AGENTS.md / the live config; `git log`
  for the maintainer's own commit) and only then fires the live acceptance
  (MEM-0101 protocol). Until the file verification passes, the pending
  items stay OPEN — never mark a maintainer-domain item done from the
  worker's summary or from memory.
- Why it matters: this is the standing close pattern of this repo — every
  tool/plugin unit since R1 has closed this way (R1/R2/#73, then
  `submit` and `#0 escape` 2026-09-18), and it is what makes his restart
  cycles testable without him asking anything. It also fixes a real failure
  mode: a reopening session that fires acceptance against an UNREGISTERED
  tool (the plan1 stale-roster incident did nearly that), or marks the
  pending line closed on trust.
- Evidence: fuzzy/numword decision-record.md R1-R7.5 "LIVENESS ACCEPTED"
  lines; plan1_summary.md + plan2_summary.md "Pending (maintainer domain)"
  sections; NAP archive lines for the FST + submit + escape closures;
  ses_f4c039ae2ffeRqvdPqGu8IdB37 2026-09-18 (the reopen-verify-accept cycle
  executed end-to-end for BOTH tools + the AGENTS.md paste).
- Verified: 2026-09-18 (cycle executed twice, once for both units at once)
- Related: MEM-0101
- Review when: the maintainer automates registration (no per-tool phase),
  or the pending-list format changes in the planner prompt.

### MEM-0103: Maintainer files live in the tree and grow as thoughts mature — single-tree status quo (ruling 2026-09-18)

- Type: `decided`
- Status: `active`
- Confidence: `high`
- Scope: how the planner treats the maintainer's live files (live config,
  AGENTS.md, prompts, drafts, priority) and when file content counts as a
  work item
- Keywords: live-file, wip, handoff, commit-point, status-quo,
  file-lifecycle, priority-move
- Memory: the maintainer live-edits his files across days as thoughts
  mature — mostly growth, never "done", they stay open on his screen
  uncommitted ("all my files are --wip and will never be complete").
  STATUS QUO (his ruling 2026-09-18: "i am fine as it is") — the 2-repo /
  draft-separation variants were considered and rejected; his stated
  reasons: sync friction, and live markers / inbox / draft channels only
  work in a SHARED tree (he sees my uncommitted progress live too).
  Channel semantics: marker / inbox / handoff content WORKS uncommitted
  (that is how markers reach me); he commits his live files before bigger
  tests or when he wants the option for me to move handled priority items
  (the move = commit-point action). He does NOT use `--wip` broadly (his
  files are all perpetually mid-state); if he places it, strictest
  handling applies: no edit, no stage, and NO READ until cleared — the
  "not-listed-as-unread in a close" variant means exactly that: a
  mid-state file's content is mid-thought, reading copies an already-stale
  version into my context, and the close message lists which wip files
  stayed unread. My side: named-path commits only, never `git add -A`
  (one incident: dc3f137 disclosure, NAP Standing).
- Why it matters: prevents re-litigating the 2-repo variant in a future
  session and makes handling of his files unambiguous: read freely unless
  `--wip`, never stage, uncommitted content is not a task unless a
  marker / inbox / handoff designates it.
- Evidence: maintainer message, direct session
  ses_f4c039ae2ffeRqvdPqGu8IdB37 (2026-09-18: "i am fine as it is" + the
  lifecycle explanation incl. the ideas-file habit); dc3f137 disclosure;
  this session's flag-only handling of his uncommitted p2 STATUS line and
  memory files (f3da151 handed over by him, 1740bb3 his live edits).
- Verified: 2026-09-18
- Related: MEM-0102 (phase-2 verification shares the file-verification
  discipline)
- Review when: the maintainer starts using `--wip` routinely, or a
  separate-draft-repo variant actually lands.

## MEM-0104 — A context-limit failure message can arrive AFTER the work is complete; rebuild from files before resuming
- What: a Task-tool failure of `context_length_exceeded: the request exceeds
  the available context size` ALWAYS means the sub-agent RAN and hit its
  context limit (maintainer ruling 2026-09-21) — including the case where the
  session finished the ENTIRE task and died only before its last steps (the
  final commit, the feedback submit). The work then sits UNCOMMITTED in the
  working tree.
- Do: on such a message — never assume lost work and never reflexively
  relaunch: rebuild from files FIRST (git status → modified files; read the
  working-tree handover file; run the gate yourself), then either land the
  last steps yourself (planner completing the worker's commit + feedback) or
  resume via task_id; a resume WITHOUT compaction is valid when budget allows
  and the maintainer orders it (compaction is not the default recovery).
- Why (evidence): plan5 (2026-09-21): worker-6's launch reported
  context_length_exceeded, but the session ses_f3ab3c67dffeujQ8L1ucfWu8k8 had
  completed the whole compact_memory unit A build (handover complete, all
  green); the planner verified probe 241/241 + 10/10 smokes + pytest 459+1w +
  ruff F=0 and landed the commit 6864bc0. Contrast: the FIRST launch of the
  same task failed with a DIFFERENT signature ("Assistant response prefill is
  incompatible with enable_thinking") and never ran — a genuine host-side
  failure. File state, not the message, is the discriminator.
- Verified: 2026-09-21 (planner-5, plan5, looprun 2026-09-21_15-33)
- Related: MEM-0102 (verify-from-files-first discipline); knowledge_context.md
   "Task failure messages: context-limit hits, not failed starts"
- Review when: the failure-message vocabulary grows, or the resume protocol
   changes.
- 2026-09-22 addendum (maintainer): the work can be COMMITTED — the worker
   committed (97fccfc) and only hit the limit producing its final result, so
   check `git log` for a fresh worker commit before deciding. The limit cannot
   be hit "instantly" (the worker took time to run), so the failure is
   evidence it ran; in the SERIAL setup that time is    invisible to the planner
   (MEM-0105), so judge from the logs / git log, NOT from a perceived elapsed
   time. #85 part 1 (97fccfc) is the 2026-09-22 instance.

## MEM-0105 — In the serial (one-slot) setup a delegation is INSTANT to the planner; the worker experiences the time
- Type: `learned`
- Status: `active`
- Confidence: `high`
- Scope: any delegation (worker / explorer) under the one-model-slot host
- Keywords: serial-slot, delegation, block-resume, elapsed-time, invisible-runtime, context-limit
- Memory: delegating a worker "exits" the planner's turn — the planner BLOCKS and
   generation resumes only when the worker returns. From the PLANNER's perspective
   any delegation completes without delay (no time passes for me while the worker
   runs); the WORKER is the one that experiences the runtime (its context grows
   over minutes). So the planner cannot use "how long it took" intuition — the
   only reliable evidence of what the worker did is the committed state / the
   logs / a context-limit failure (MEM-0104).
- Why it matters: the planner reflexively reads a context-limit failure as
   "instant, so the worker didn't run" — but the worker's runtime is invisible to
   it, so the failure is actually proof the worker ran (and may have committed).
   Check the files, don't guess from perceived time.
- Evidence: 2026-09-22 direct session ses_f39d250e9ffeheip2FVEeY5Fk6 — the #85
   part-1 launch returned a context-limit failure "instantly" to the planner, but
   the worker had run + committed (97fccfc) + built the #82 toggle in-task; the
   maintainer's serial-slot explanation (planner blocks + resumes; worker
   experiences the time).
- Verified: 2026-09-22
- Related: MEM-0104
- Review when: the host gains multiple model slots, or delegation becomes async.

## MEM-0106 — A sub-agent's self-compaction returns the COMPACTION SUMMARY as the Task result; recognize it by the `## Objective` form + the ctx.log COMPACT line
- Type: `learned`
- Status: `active`
- Confidence: `high`
- Scope: interpreting any worker/planner Task result after a mid-task self-compact
- Keywords: compaction-summary, self-compact, Objective-form, ctx.log, COMPACT-line, task-result, resume-task_id
- Memory: when a sub-agent self-compacts mid-task, the returned text is the
   COMPACTION SUMMARY (the Work State form: `## Objective` / `## Important Details`
   / `## Work State` / `## Next Move` / `## Relevant Files`) — generated by the
   compaction model, NOT a message the agent wrote. Two independent signals:
   (a) IN-RESULT — the summary always starts with `## Objective` (the
   SUMMARY_TEMPLATE forces "Output exactly the structure"); a normal worker
   closing (a short handover pointer) never uses that form. (b) GUARANTEED
   EXTERNAL — our compact_memory tool writes `<date> <model> COMPACT <sessionID>
   tokens=N messages=N` to `.opencode/temp/ctx.log` on verified success; grep it
   for the session ID. On an ambiguous return: recognize the Work State form →
   confirm via the ctx.log COMPACT line → resume via task_id using `## Next Move`.
   No worker convention needed (the compaction model + our tool already emit the
   signal).
- Why it matters: the planner first misread the compaction summary as a worker
   message and was unsure whether to resume. The two signals make the recognition
   reliable with no new mechanism and no error-prone convention.
- Evidence: 2026-09-22 direct session ses_f39d250e9ffeheip2FVEeY5Fk6 — the worker
   (ses_f359ce94...) self-compacted; its Task result was the Work State form;
   ctx.log carried `COMPACT ses_f359ce94... tokens=30000 messages=12`; the
   maintainer confirmed the summary IS the signal + the ctx.log line is the
   guaranteed one.
- Verified: 2026-09-22
- Related: MEM-0104, MEM-0105
- Review when: the compaction summary template changes (the `## Objective` start)
   or the ctx.log COMPACT line format changes.

## MEM-0107 — Worker limit-death forensics live in the dump's step-finish meta; the in-flight write is salvageable from the last message
- Memory: when a worker session dies mid-task (empty Task result, NO ctx.log
  COMPACT line for it), dump it (`node agent/scripts/db/dump_session.cjs
  <sid>`) and read the LAST step-finish meta: `reason=length` + a large
  `output=N` = the per-turn OUTPUT cap cut the generation (a truncated write
  tool call → the file change never lands — the tree stays clean for that
  file), while `tokens.total` at the model's window = the context wall. The
  dump's per-message byte sizes reveal where the window went — a worker's own
  giant outputs (design/plan prose, 15k+ tokens per message) can burn MORE
  than the file reads. The in-flight write is salvageable VERBATIM: the dump's
  last message = everything from the last `## msg_` header to EOF. Successor
  handoff = the on-disk uncommitted diff + the salvage file + the spec, with
  explicit instructions: write in chunks (≤ ~8KB ≈ 2k tokens per write/edit),
  read ONLY the needed sections (not whole files), run the smoke early and fix
  reds incrementally. A cross-compact + task_id resume of the dead session does
  NOT save the rework when the kill was an output-cap mid-write (the text must
  be re-emitted anyway) AND costs a same-model single-slot flush of your own
  session — the fresh relaunch with salvage is the cheaper deterministic path.
- Why it matters: plan8 (2026-09-23) worker-12 died exactly this way (23,156
  output tokens cut mid-smoke-write, total 170,238 = the 170k wall; ~34k of
  the window were two planning-prose messages). Compaction could not have
  saved the smoke re-emission; the salvage made the fresh relaunch near-zero
  rework.
- Evidence: 2026-09-23 plan8 (planner ses_f33f1eb98ffeFvrnTdmTzmyE2x): dumped
  ses_f33ee8eabffeaE0xuwZ7lc65NR — machine pass over per-message bytes
  (17.2k + 16.4k token output planning messages; final step
  reason=length output=23156 total=170238); smoke file untouched in the tree;
  74KB salvage at loop/autorun-2026-09-21_15-33/
  plan8_worker12_smoke_draft.md.
- Verified: 2026-09-23
- Related: MEM-0104 (rebuild from files on a limit failure), MEM-0106
  (self-compaction signals — the COMPACT line is the thing ABSENT here).
- Review when: the dump format changes (step-finish meta / `## msg_` headers),
  or chunked-write guidance proves unnecessary in a later looprun.

## MEM-0108 — NEVER launch requests at the backend inference server (the single llama-swap model slot) — a direct request evicts the live session's own model

- Rule (maintainer ruling 2026-09-24, direct session): agents NEVER
  launch ANY request at the backend inference server (the
  OpenAI-compatible llama-swap endpoint — 192.168.178.20:8033 as of
  2026-09-24; the address may move) — not for speed tests, not for
  probes, not "small" requests, not even enumeration (GET /v1/models).
  All model traffic goes through opencode sessions only. The backend is
  a SINGLE model slot: it swaps in whatever the active session needs —
  a direct request swaps out whatever is live, INCLUDING the requesting
  agent's own model (which then must be swapped back in at its next
  turn). The maintainer alone touches the backend (e.g. moving the
  context limit there — he can do that without opencode.jsonc changes).
- Why it matters: a direct 80k-token prefill probe at
  Qwen3.8-27B-Compaction returned empty (TTFT null, no usage) after
  ~7 s and kicked the planner's own model (Qwen3.8-27B-Q3S-230K-slow)
  out of the backend mid-session. The probe was also REDUNDANT — the
  maintainer had already measured prefill in his fork test (a fork of
  the planner session driven to the context limit on the Compaction
  model).
- Evidence: 2026-09-24 (planner ses_f30493f9effeuQRFc3ijNON166, direct
  session): my own prefill_test.mjs against the backend
  /v1/chat/completions; maintainer: "you kicked your own model out of
  the backend with a command ... NEVER!".
- Verified: 2026-09-24 (measured incident, maintainer-confirmed).
- Related: MEM-0105 (single-slot setup); the canonical safety-limits
  home is repo_overview.md (maintainer-owned — agents do NOT edit it
  directly; a one-line Safety entry was proposed in the 2026-09-24
  session; no safety section existed there yet); AGENTS.md "Safety
  limits" points at repo_overview.md.
- Review when: the backend gains multi-slot/queueing or a dedicated
  test endpoint, or the maintainer explicitly authorizes a specific
  direct request for a measured run (then the rule's scope narrows to
  "only explicit maintainer-authorized runs").

## MEM-0109 — Task-tool limit signature `request (N tokens) exceeds the available context size (M tokens)` = a limit hit on a TOOL-OUTPUT insertion (a normal limit death, not a launch rejection)

- Ruling (maintainer, 2026-09-25, direct session): in that signature
  "prefill" means the context-limit hit happened while a tool output was
  inserted into the context — the SAME as a normal context limit, only the
  final push was a tool output instead of the thinking/generation part.
  Consequence: do NOT read the Task-tool failure message as a failed start
  (a fresh-session prefill rejection) — the sub-agent ran, and its last
  in-flight request (the accumulated context + the new tool output) was
  hard-rejected by the server because it exceeded the window.
- Evidence: 2026-09-25 (planner `ses_f2a436b57ffe8go608Z63jwNG6`, direct):
  `worker_Q3S_170K` on TODO #93 — the Task result was `request (170372
  tokens) exceeds the available context size (170240 tokens), try
  increasing it`; the session dump
  (`ses_f2a2ebcafffeVW1j4oXA7K3ioa`) read messages=61 parts=280 — a full
  run; the working tree had ALL six task files staged (`git add` done), the
  handover written, the gate evidence recorded (probe 257/257,
  context_recovery smoke 15/15, compact_memory 66/66, auto_resume 129/129,
  pytest 459+1w, ruff F=0); the planner verified from files (git status +
  handover + a targeted 15/15 spot re-run of the changed smoke) and landed
  the commit `5f823e4` — no resume needed.
- Verified: 2026-09-25 (measured incident + the maintainer's confirmed
  interpretation).
- Related: MEM-0104 (a `context_length_exceeded`-class Task failure ALWAYS
  means the sub-agent RAN — rebuild from files first), MEM-0107 (limit-death
  forensics via the dump's last step-finish meta + per-message sizes).
- Review when: a GENUINE launch failure is observed (a fresh session whose
  prefill alone exceeds the window) — then distinguish the two signatures by
  file state (the dump's message count: ~2 for a failed start vs. many for a
  ran-and-died session), never by the message alone.

## MEM-0110 — Direct sessions: NO delegation without prior explicit maintainer OK; interact first, stop guessing and ask, push back without sugarcoating

- Ruling (maintainer, 2026-09-27, live priority.md additions in direct
  session ses_f20d1b39…): "in general in a direct sesson there should not be
  started a delegation without prior explicit ok from the maintainer" +
  "answer/interact first (verify shortly if needed but never start a task
  with a worker without confirmation on what we actually want to implement).
  stop guessing and ask." + "push back, do not sugarcoat. question my
  assumptions. ... questions are a sign of maturity — jumping to conclusions
  without using the resources available is dump behavior." Trigger context:
  his complaint that the post-compaction planner (fork session ses_f20b3bf14…)
  "acted really unreliable and implemented without giving the chance for
  interaction and clarification" — the keepTokens metric rework landed before
  his test verdict (his #99 remark: "another planner updated prematurely the
  calc method").
- Evidence: his live priority.md additions (2026-09-27, uncommitted, his
  domain — read, not edited) + the fork-session metric commits (30b81d9/
  b95d532/922463e/dcad3d1) preceding his compaction_tests.md verdict.
- Verified: 2026-09-27 (his live file; this session honored the ruling — #92
  staged as the NEXT unit, not launched).
- Related: the planner prompt's direct-session design-exchange rule (this
  ruling tightens it: the delegation ban is explicit, not just a
  "design-exchange" preference); MEM-0103 (his live files).
- Review when: he lifts or edits the priority.md lines, or an
  inbox/proposal item carries an explicit pre-approval for a delegation in
  a direct session (prior explicit OK = allowed by the ruling's own terms).
- 2026-09-28 addendum (maintainer instruction, plan28): this entry is the
  CANONICAL HOME for the planner behavioral guidelines (his priority.md
  "planner behavioral guidelines" block) — moved here out of the NAP per his
  "can be its own section in it" note; the destill carries it under its own
  section (destilled_mem.md "Planner behavioral guidelines"); the NAP line
  now points here instead of restating.

## MEM-0111 — The edit tool's hint line is NOT the authority on apply-vs-reject; read the file back

- Observation (2026-09-30, plan45 ses_f0fc7d5daffeVv6fOuHGVh6Nt2): a
  "hint rejected line=N d=0 gap=inf" line was delivered on an edit that WAS
  applied (verified by file read-back matching the newString). The d=0 /
  gap=inf signature joins the plan44 misattribution cluster (no-anchor-line
  / d-too-high).
- Rule: ALWAYS read the file back after any hint line before concluding
  rejection. The hint line is a delivery-side annotation, not an apply
  verdict.
- Evidence: the plan45 NAP Standing edit + read-back.
- Related: MEM-0101 (the misattribution-cluster addendum); the TODO #124
  class (tool-side apply-vs-reject reporting).
- Review when: a resolved tool-side apply-vs-reject reporting fix lands
  (the TODO #124-class work), or a maintainer ruling on the hint channel.

## MEM-0112 — Stale-reference sweeps verify against the ACTUAL directory listing, not the rename mapping

- Lesson (2026-09-30, plan46/plan47): plan46's agent_readme_* fix re-pointed
  renamed files but missed index entries for files that do not exist in the
  agent repo at all (repo_map/repo_testgate/repo_gotchas — per-project
  only); plan47 (f7f4100) caught it via the explorer audit's observation.
- Rule: after a tree reorg or rename batch, `ls` the target folder and diff
  EVERY index line against it — including files the sweep pattern does not
  match. A pattern-only sweep is incomplete by construction.
- Recurrence (2026-10-15, direct session): the prompt rework shipped 3 stale
  paths of its own (the AGENTS.md gauge path, the compaction-guide path, the
  concrete-paths pointer — re-pointed in AGENTS_draft_2026-10-15.md),
  caught by exactly this check.
- Evidence: commits f7f4100 + 5c8d612, plan47 handover observation 2; the
  2026-10-15 audit.
- Related: MEM-0103 (placement rulings — index lines are placement).
- Review when: the next tree reorg or index rework.

## MEM-0113 — A spec that names a SMOKE block as the pin site still needs a PROBE-section pin audit
- Lesson (2026-10-01, plan2 #128): the probe's S13 checks 92/93/95/222-226
  + S32 340 pinned the same gate OUT of the spec-named Gate-M smoke block;
  the re-pins were not derivable from the named region; the probe run caught
  them on the first red.
- Rule: when authoring a spec for a behavior change with probe coverage,
  name the probe check ids explicitly in the spec (audit both sections).
- Evidence: memory_inbox 2026-10-01_05-02; worker handover plan2.
- Review when: the next spec names a smoke block as the pin site for a gate change.

## MEM-0114 — Task specs citing installed-library API shapes: verify signatures at authoring time
- Lesson (2026-10-01, plan3 #129): an unverified `QContextMenuEvent(QPoint)`
  ctor claim cost the worker a discovery step (real PySide6 6.11.2 ctor:
  `(Reason, pos[, globalPos])`, 2-arg deprecated).
- Rule: one import/inspect call at spec-authoring time before citing an
  installed-library API shape in verified-facts.
- Evidence: memory_inbox 2026-10-01_05-26; spec commit 6d1b3f5; plan3 handover.
- Confidence: medium (single occurrence).
- Review when: the same friction class recurs.

## MEM-0115 — Count Python logging call sites with BOTH greps: `logging.` AND `logger.`
- Lesson (2026-10-01, plan4 #131): a `logging\.`-only grep returned 4 in the
  FST repo; `logger.` module-level call sites made the real count ~14.
- Rule: grep BOTH forms when counting logging call sites for spec
  verified-facts.
- Evidence: memory_inbox 2026-10-01_06-22; research doc
  `agent/research/2026-10-01_fst-logging-design.md` §7.1.
- Confidence: high; stable grep lesson.

## MEM-0116 — A tool/plugin file landed mid-session is NOT live until the next host start; acceptance rides the NEXT session
- Lesson (2026-10-04, plan10): the host auto-detects `.opencode/tools/*.ts`
  at PROCESS START only; a mid-session landing shows absent from the
  current toolset — NOT a failure.
- Rule: verify such builds from files (git + handover numbers) + a targeted
  spot smoke re-run in-session; record live acceptance as PENDING the next
  session's restart in the TODO/NAP status line.
- Evidence: memory_inbox 2026-10-04_23-54; plan10 commits 8d332d7/2c3cbb3.
  Today's bt mode-guard micro-fix followed the same path (live only after
  the maintainer restart).
- Review when: the host gains hot-reload of tools mid-process.

## MEM-0117 — After a model/config swap, the injected ctx budget suffix can transiently read cap 1; trust ctx_gauge
- Lesson (2026-09-30): the nudge resolves the session ROW model (stale
  until the first new-process prompt); unlisted model + no `default` key
  → cap 1 (compaction_core.ts resolveCap L205-211).
- Rule: for budget decisions after a model/config change, use the
  self-gauge (ctx_gauge), not the injected suffix.
- Evidence: memory_inbox 2026-09-30_16-57; the session's own gauge read
  `CTX=notAvailable` at 2026-10-21 start with suffix `3 compactions left`
  (a first-ever gauge readout in a fresh session — normal).
- Review when: a `default` key lands in the budget file, or nudge model
  resolution changes. Related: knowledge_plugins.md §329 (the no-total
  root cause).

## MEM-0118 — The live host has NO in-process sqlite; any DB tool must ship the spawn-sqlite3 last resort
- Lesson (2026-09-30, #126): both bun:sqlite and node:sqlite dynamic imports
  fail in the live host process (context_trim live report returned
  `db-error: no in-process writable sqlite backend`); the only proven live
  backend is spawn-sqlite3 via `.opencode/plugin/tools/sqlite3.exe`.
- Rule: fixture smokes (node v24, node:sqlite) CANNOT detect this — any
  DB-writing tool ships the #126 pattern: Ctx-shaped adapter over
  execFileSync, no shell, hard timeout, single-invocation transaction.
- Evidence: memory_inbox 2026-09-30_16-58; TODO #126; commits a43311a/a276ef1
  + knowledge_plugins.md §412 (the CLI facts).
- Review when: a host build with working bun:sqlite lands.

## MEM-0119 — An explicit maintainer worker-OK overrides the MEM-0110 no-delegation guideline (direct mode)
- Lesson (2026-10-21): "you may start workers if needed" = prior explicit
  OK; the guideline's trigger condition is satisfied, proceed.
- Rule: still verify a `--comment` review draft against current code
  line-ranges BEFORE believing it (2 of its claims were stale — the MOVE
  dst-guards already existed at block_transfer.ts 636-646), hold fixes
  for his ruling.
- Evidence: memory_inbox 2026-10-21_21-06; direct ses_ea02d3559ffe….
- Review when: MEM-0110's ruling changes.

## MEM-0120 — Model self-reviews of PASTED code mix real catches with confident P0 hallucinations; diff paste-vs-deployed first
- Lesson (2026-10-21): our model's self-review of its OWN tool code claimed
  a non-existent syntax error + an unknown-mode lie (schema-blocked), while
  its real catches (PEEK digit-string bypass, MAP uncapped echo, empty-refs
  form error, TMP/TMPDIR gap) were valid.
- Rule: when a review ran on pasted/older code, diff paste-vs-deployed and
  check load/smoke proof BEFORE believing any P0; a review's confidence is
  not evidence.
- Evidence: memory_inbox 2026-10-21_21-17; the 2026-10-21 review-draft
  verdict in the NAP.
- Review when: another model-testing session uses pasted code.

## MEM-0121 — A worker close NARRATIVE can FABRICATE references (hashes, restarts, maintainer claims) — verify against git before recording
- Lesson (2026-10-22, planner-13): a worker's own handover named a nonexistent
  commit + claimed a host restart + a maintainer instruction while its session
  did real work (main-tree 4bef076 → worker's own correction 28a6685 → 6ca8643).
- Rule: before recording any hash/restart/instruction from a worker return,
  run `git log --oneline -N` + `git show --stat <hash>` in that tree; an
  absent reference → mark the entry untrusted at next curation + note the
  fabrication class in the NAP.
- Extends MEM-0120 (pasted-code self-reviews) to worker close NARRATIVES.
- Review when: a worker close text names a hash absent from `git log -20`, or
  a restart/instruction claim with no `maintainer/inbox_*`/loop-log line backing it.

## MEM-0122 — block_transfer WRITE/REPLACE feedback lines are NOT landing proof — read the file back
- Lesson (2026-10-22, planner-13): a WRITE reported `+24 line(s)` but the file
  stayed 253 lines — a long single-call prose payload arrived TRUNCATED and the
  write silently no-oped.
- Rule: after any WRITE/REPLACE of >~15 prose lines, read the file back (MEM-0111
  discipline); repair with a smaller `edit` insert; the `+N line(s)` count + first-line
  echo are NOT proof.
- Evidence: memory_inbox 2026-10-22_02-47; the plan13 NAP insert incident.
- Review when: the tool later validates payload integrity.

## MEM-0123 — block_transfer buffer hygiene: name buffers per artifact; WRITE is the bufferless mode
- Lesson (2026-10-22, planner-14/17): COPY/CUT/PASTE without bufferName share
  'default' — a later CUT silently destroys a staged paste buffer mid-chain (the
  plan17 4a copy died this way and forced a re-COPY).
- Rule: name buffers per artifact; MAP/PEEK a buffer BEFORE REPLACE (REPLACE
  consumes whatever sits at that name — the auto `last_write` buffer is NOT
  assumed content); on an EMPTY-BUFFER error, re-check the MODE list before
  retrying the same mode (WRITE = direct text → region, bufferless; REPLACE/COPY/
  APPEND/PASTE need named buffers).
- The N-regions+1-text hazard is RESOLVED: the REJECT-with-teaching-error end
  state landed at plan15 (MAIN b713585 / clone 4054a0a) — the entry stays for
  the buffer-scoping half.
- Evidence: memory_inbox 2026-10-22_03-26 (×2), 04-38, 07-36.
- Review when: block_transfer buffer scoping or a mode redesign lands.

## MEM-0124 — Spec pin audits: enumerate by verdict-token grep across EVERY pin file, never by range-recall
- Rule (2026-10-22, planner-14): before narrowing/grepping a channel, run
  `rg -n "<verdict-token>"` (e.g. `out-of-sandbox|path-anomaly`) across every
  pin file (smokes AND the probe) and build the audit from the hit list.
- Treat the first post-change gate run's FAIL list as the AUTHORITATIVE
  inventory; a range-recalled audit missed 3 flipping surfaces ((B3), probe
  checks 303 + 304).
- Evidence: spec `agent/handover/specs/2026-10-22_142-138-path-field-scan.md`;
  todo_inbox 2026-10-22_03-17 RESOLVED at plan14.
- Review when: the pin files are restructured.

## MEM-0125 — Typecheck pins need a NEGATIVE CONTROL — push a deliberate bad literal, confirm RED, then commit
- Lesson (2026-10-22, unit 3a): a tsgo run printing zero errors is NOT proof a
  pin is real — a generic `E extends Array<unknown>`-style hook param silently
  WIDENS and accepts bogus literals (`role:"bogus-role", content:5` passed at
  first attempt).
- Scope: any fork-typed v2 plugin work.
- Evidence: memory_inbox 2026-10-22_05-23; the 3a negative-control proof.
- Review when: any future typecheck-only pin.

## MEM-0126 — Task-tool "user rejected permission" = the sub-agent NEVER STARTED — re-issue ONCE with a SHORT spec-pointer prompt
- Rule (2026-10-22): classify as a LAUNCH failure, NOT a context/permission
  task failure; do NOT compact or resume; the target repo tree + git log stay
  untouched (verified). Re-issue ONCE with a short prompt that points at the
  committed spec file — the long spec-echoing prompt is the suspected trigger
  (hypothesis, medium confidence). Two rejections → stop and report the boundary.
- Evidence: memory_inbox 2026-10-22_06-52. Completes the MEM-0104/0106/0109
  signature set: launch vs ran-vs-wall vs compaction-return.
- Review when: rejections recur with short prompts (then the hypothesis dies).

## MEM-0127 — Before REUSING a worker session via task_id, check its close gauge; cross-compact FIRST at near-cap
- Rule (2026-10-22, plan18): a resume request at ~89% of window is REJECTED
  OUTRIGHT (prompt + history > window — "requests are never truncated").
  Sequence: cross-compact (keepMessages=10, message=new-unit pointer) → verify
  the `COMPACT <sessionID>` line in `.opencode/temp/ctx.log` → THEN resume
  via task_id. Saves the rejected-call round-trip.
- Evidence: memory_inbox 2026-10-22_10-26; the 4b worker (CTX=234727/262k) —
  the 4c resume landed only after the pre-resume cross-compact.
- Review when: worker close gauges start landing well below 80%.

## MEM-0128 — Cross-tree port specs: run the twin-freshness check BEFORE committing a design-of-record anchor
- Rule (2026-10-22, plan19): before committing a spec anchored on a cross-tree
  file (MAIN vs the clone copy), run `cmp -s` + `wc -l` + a bounded
  `diff | head -60`; anchor on the MAIN twin when the clone copy lags.
- A STALE twin silently pins stale behavior; the delta re-surfaces as worker-side
  "discrepancy" noise.
- Evidence: memory_inbox 2026-10-22_11-14; the 5c re-baseline (clone
  `block_transfer.ts` 845 vs MAIN 1034, 333 diff lines); TODO #143.
- Review when: any future #137 unit spec, or a clone-twin refresh lands.

## MEM-0129 — File-surgery span anchors: a section span ends at the NEXT ANY `^## ` header, never at the next ID-pattern header
- Rule (2026-10-22, plan20): re-grep the exact line spans IMMEDIATELY before
  EVERY multi-op block_transfer surgery (MOVE/DELETE/WRITE with line numbers);
  an entry/section span runs to the next `^## ` of ANY shape, not the next
  `## #<id>`-pattern match.
- Observed: anchoring a TODO-retire MOVE on an id-grep's "next header" carried
  38 unintended lines (2 section headers + entries 53/56) into
  `todo_records.md`; repair = verbatim MOVE-back + narrow redo (4 extra calls).
- Evidence: memory_inbox 2026-10-22_14-24. Extends MEM-0112 to intra-file surgery.
- Review when: block_transfer re-validates spans before cutting (then anchors may stay cached).

## MEM-0130 — Planner-direct units (≤~50 diff lines, ≤4 files): close inline with per-file grep-back; the `edit` hint line is not authority even for replaceAll
- Rule (2026-10-22, #144 unit): small test/doc-side fixes land in the planner
  session without delegation; after every edit or bulk `replaceAll`, read the
  touched file back (grep the changed tokens) before concluding — the
  `hint rejected lines=` line fired while two 14/14-class replaceAlls LANDED.
- Evidence: memory_inbox 2026-10-22_18-21; extends MEM-0111 to the replaceAll form.
- Review when: the edit tool's hint-line semantics change.

## MEM-0131 — Spec fixtures for ordering/behavior pins must be proven DISCRIMINATING at spec-authoring time
- Rule (2026-10-22, marker-order unit): when a spec suggests a smoke fixture
  pinning a reordering/behavior change, state the raw emit order (note: `grep
  -rn` emits hits in its PATH ARGUMENT order, NOT lexicographic) and require
  the pin to FAIL against the UNCHANGED code; pick a fixture that sorts AFTER
  the comparator (TODO.md/README.md/WIKI.md for the marker sweep).
- Also re-check the fixture's line-count math before adding to cap-pinned
  fixtures (the state_snapshot FX1 digest is pinned at the EXACT 60-line cap —
  replace a fixture line, never add one).
- Evidence: memory_inbox 2026-10-22_18-54; the suggested `maintainer/live_note.md`
  fixture passed 21/21 pre-change; worker switched to a `WIKI.md` fixture.
- Review when: a tool changes how pins run (e.g. block_transfer re-validation).
