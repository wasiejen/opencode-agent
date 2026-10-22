# TODO — maintainer's open items

Numbering: every entry ID is UNIQUE and NEVER REUSED — used so far up to #133, new
entries start at #134 (closed IDs stay reserved in the `todo_records.md` files —
root (agent entries) + `projects/Free-Snap-Tap/todo_records.md` (FST entries)).
Closed entries live in those `todo_records.md` files (one-line records — resolution in file/git log).
Entries follow the AGENTS.md contract (title / evidence / outcome / acceptance / scope / status).
Pre-carve hash note (2026-09-30, plan45, #123): the 2026-09-29 filter-repo carve rewrote the entire history — every 7-hex hash cited below from before 2026-09-29 is a dead revision in the post-carve repo (historical text, not a live pointer). Measured remaps: cbbebf8→7a9e291 (#106), 0c90abe→60f031b (#115), 12e3262→fea1cbb (#114), 44c50a2→9e91878 (R3 import fix). The 4 pairs are re-pointed inline; the remaining dead citations are annotated by this line.

## Maintainer calls (open, in order)

1. (none open as of 2026-09-15 — the #11 call was resolved by his ruling D1-A in the consolidated decision file)
2. (resolved 2026-09-27, direct session ses_f20d1b39…) #99 follow-up — his
   ruling: PURSUE the flexible per-dispatch keepToken (research + wire it if
   it exists in the installed host; the config knob stays the fallback; the
   fixed global 40k is "too much in most cases, too little in some" — not the
   answer).
3. Resolved calls (records): v1.3 log-growth confirmation → #17 CLOSED (one-shot read
   executed); v2.5 nudge target scope ruling → #33 (per-session read); Deferred FST
   behavior batch → all 5 Recs approved + LANDED on `fst_work` (unit A `4b93d37` +
   unit B `2891dab`, proposal in `implemented/`); build schedule → complete;
   `handover_task.md` worktree conflict → #49 CLOSED (`b6dc3e7`); Looprunner prompt
   v2 → #39 CLOSED.

FST project entries: `projects/Free-Snap-Tap/TODO.md` (moved 2026-09-29, #117 — FST behavior / code / docs / env entries live there)

## Docs & misc (open)

## #124. (LANDED 2026-09-30, plan46 worker-46; curated from todo_inbox worker-45 new-hire test; pre-approved class — agent-usage friction removal) block_transfer return-line / description clarity: the 3 new-hire friction findings
- **Problem / evidence:** the plan45 new-hire test (worker-45
  ses_f0fad3537ffe6BVZbeAUPPkoxi — battery 10/10 PASS, 0 escapes, the tool
  source never read) found 3 description/return gaps (full text in
  `todo_inbox.md` 2026-09-30_05-25 entries; scratch evidence
  `.opencode/temp/bt_newhire_plan45/`): (1) PASTE/REPLACE/WRITE returns
  report the source span / buffer range + a `first:` field but never the
  destination insertion line, and `first:` mixes old-span range with
  new-content first line; (2) the non-unique-anchor error lists ALL match
  lines (2 → both) but the description says "the first match line numbers"
  — ambiguous at scale; (3) a failed COPY/CUT leaves no buffer (PEEK →
  "empty") — the description is silent. Same run, same unit (optional):
  the PEEK head/tail overlap note on small buffers + the blank-line
  adjacency after MOVE/DELETE (residual blank line stays).
- **Desired outcome:** a new hire can predict every return line and error
  from the description alone; PASTE reports where it landed (description
  or return — the build's call inside the DoD).
- **Acceptance criteria:** description updated (per-mode return-shape
  block: range semantics + `first:` label + the PASTE destination line +
  buffer-on-failure + all-match-line error listing); if any return line
  changes, the bt smoke pins re-pinned (131/131 + 64/64 baseline) + the
  standard gate green.
- **Suggested scope:** `.opencode/tools/block_transfer.ts` (description +
  possibly the PASTE return line), `.opencode/plugin/tests/
  block_transfer.smoke.mjs` (re-pin if behavior changes).
- **Status:** LANDED 2026-09-30 (plan46, worker-46 — description-only
  build call, zero behavior change, no re-pins): new RETURN LINES
  paragraph in the `block_transfer` description (per-mode range semantics:
  PASTE = buffer's own 1..N; REPLACE = the replaced old span, N = old lines
  removed; MOVE/CUT/DELETE = the source span; WRITE = the pre-call span,
  N = new text lines; COPY/APPEND = resolved span / selected refs / text
  1..N; `first:` always the NEW/extracted content's first line, capped
  ~40 chars; `- buffer: M` = the count AFTER the op; destination landing =
  after the targetMarker line or appended at EOF — the return line names
  the file but not the insertion line) + the bounded non-unique error form
  (match count + FIRST 3 match line numbers — all of them when 3 or fewer,
  " …" appended when more) + buffer-on-failure note (a failed COPY/CUT/
  APPEND leaves NO buffer; a later PEEK/MAP returns the empty-buffer
  error) + the optional item 4 notes (PEEK head == tail overlap on small
  buffers; residual blank-line adjacency after MOVE/CUT/DELETE). Gate
  green: probe 352/352 + bt smoke 131/131 + sandbox 64/64 + all other
  smokes ALL PASS (the description-only change broke no pin — the
  sandbox-smoke description pins re-verified green in the same run).
  (Commit hash recorded in the planner's follow-up bookkeeping commit —
  no self-reference.)

## #123. (closed 2026-09-30, plan45 maintenance pass — scripted sweep + batch re-point + one-line annotations) stale pre-carve commit hashes in TODO.md / NAP status lines — batch re-point or annotate (maintenance pass); full text in todo_records.md (body retired at the plan20 maintenance pass)


## Loop & coordination (open)

## 53. (closed 2026-09-23, planner-13 bookkeeping; full text in todo_records.md) — Agent-feedback protocol: the mandatory close-down friction step (Part A, 5e29cb0, all 4 role prompts) + the `submit` write-tool (Part B, b83b34f + session/role autofill 86a977f); maintainer-domain tail closed (registration live in the role toolsets, AGENTS.md paste LANDED 2026-09-18, machine-stamped entries firing in live sessions since).

## 56. (DEFERRED 2026-09-15, maintainer `--defer` in priority.md) — Distillation worker runs over the session dumps (his # 3 3 mandate)

- **Problem / evidence:** the 137-session corpus (`archive/sessions/`,
  backfilled 2026-09-15) is the basis for distilling session history; the
  maintainer deferred the runs 2026-09-15 ("to much work right now") while
  focusing on the proposal/inbox backlog (his #0).
- **Desired outcome (his # 3 3, verbatim gist):** test different distillation
  workers on session dumps — gemma4 workers (`worker_gemma_Q4_128K` /
  `agent_gemma_Q4_128K`, much faster, parallelizable) with a WIDE range of
  scanning perspectives, compared against qwen runs (4x slower); roles/
  skillsets to keep track of them; document what is found; also usable to
  recover the last messages of a failed worker.
- **Acceptance criteria:** when the deferral lifts — ≥2 perspectives run on
  the same dump with ≥1 gemma and ≥1 qwen model; a written comparison (what
  each model found, quality, cost); findings worth keeping in the knowledge
  base; the skillset/roles documented for reuse.
- **Suggested scope:** `skill_session_scan.md` (ALREADY BUILT, plan1 —
  perspectives P1 FRICTION / P2 DECISIONS / P3 KNOWLEDGE + output format);
  `archive/sessions/` (read-only dumps); the `worker_gemma_Q4_128K`
  + `worker_Q4_120K` roster; a comparison notes file in the loop folder.
- **Status:** DEFERRED — picked up only when the maintainer lifts the
  `--defer` marker in `priority.md` (# 3 3) or re-prioritizes it.
  **Still DEFERRED 2026-09-27 (direct session ses_f20d1b39…) — REWORK
  PENDING before any runs: the idea is to be clarified first — per-agent-
  type distill (planner/worker/explorer/…) gathering friction points +
  undocumented knowledge/memory (behaviors, repeatedly re-derived
  methods/tricks) into per-agent memories (memories not yet seeded for all
  roles); the gemma speedup is nullified by cache invalidation on every
  model switch (the planner re-prefills per delegation) → gemma only for
  preliminary scanning. The `--defer` stays. Re-verified 2026-10-01 (plan1 maintenance pass): still DEFERRED (the deferral marker is unchanged; the rework is still pending before any runs).**

## Plugin & gauge (open)

## #125. (closed 2026-09-30, plan48, planner-48 inline — all 8 items LANDED, zero behavior change; gate probe 352/352 + all 11 smokes green) #86 audit LOW findings batch: 7 inline/stale code refs + 2 doc-only duplicates
- **Problem / evidence:** the #86 read-only audit (explorer-47 ses_f0f768f33,
  plan47 — handover in `agent/handover/handover_task_to_planner.md`, full
  per-finding detail in the `plan47 TODO #86 cluster A-D` todo_inbox entries,
  2026-09-30) found 11 findings; the 2 med ones are already fixed
  (61cd616: BUSY_TIMEOUT_MS dedup + ctx_watchdog nudge-ladder constants;
  f7f4100: the role-prompt index stale refs from the explorer's
  observation). Remaining (all low): (1) .opencode/tools/loop_log.ts:127 —
  description example `Qwen3.8-27B-IQ4KT-120K` is not a live model id → a
  live id (e.g. Qwen3.8-27B-Q3S-170K) or a neutral example; (2)
  .opencode/tools/submit.ts:34 + :141 — comment example
  `planner_Q3S_245K_slow` is a stale agent name → `planner_Q3S`; (3)
  .opencode/plugin/auto_resume.ts:1536 — inline `0.99 * window`
  (limitStopCheck #109 condition 2) → named constant
  (LIMIT_STOP_WINDOW_RATIO); (4) auto_resume.ts:1538 — inline `60_000`
  idle threshold (#109 condition 3) → named constant; (5)
  auto_resume.ts:1885/1887 — inline `20MiB`/`2MiB` log-size defaults →
  named constants; (6) auto_resume.ts:1901 — inline `5000` tickMs default →
  named constant; (7) auto_resume.ts:173-177 — the "OVERLAP-ERA CAVEAT"
  note describes the RETIRED looprunner_Q3S (commented out in the live
  opencode.jsonc) as if live → update/remove; (8) doc-only: the opencode.db
  path is defined 5x in the standalone agent/scripts/db/*.cjs CLIs +
  gauge.mjs DEFAULT_DB_PATH, and HOST_EXE 2x in agent/scripts/binary/
  {binwin,binhits}.cjs → document the single source of truth (no shared
  .cjs helper unless wanted).
- **Desired outcome:** every inline literal named/deduplicated, the stale
  examples and the retired-looprunner note current — zero behavior change.
- **Acceptance criteria:** grep-clean for the stale example strings and the
  five inline literals (named constants in place); the caveat note removed
  or reworded to the retired state; standard gate green (smokes + ruff F=0,
  the #113 env-fail exclusion applies); no smoke pin changes expected (the
  values are unchanged).
- **Suggested scope:** the files named above; worker-delegable as one small
  cleanup unit (spec names each literal + its replacement).
- **Status:** CLOSED 2026-09-30 (plan48, planner-48, inline): (1)
  loop_log.ts example → Qwen3.8-27B-Q3S-170K; (2) submit.ts comment
  example ×2 → planner_Q3S; (3)–(6) auto_resume.ts literals named
  (LIMIT_STOP_WINDOW_RATIO / LIMIT_STOP_IDLE_MS / MAX_LOG_BYTES_DEFAULT
  / LOG_TAIL_BYTES_DEFAULT / TICK_MS_DEFAULT); (7) the OVERLAP-ERA
  CAVEAT reworded to the retired-looprunner state; (8) the
  agent/scripts/README.md machine-path single-source-of-truth note.
  No smoke pin changes (values unchanged); ruff/pytest do not apply in
  this repo (post-#117 split — the standard gate = probe + smoke
  suite). The 2 med findings of the same audit were already LANDED
  (61cd616).

## 74. (closed 2026-09-27, direct session ses_f20d1b39… — backend no longer on ik_llama, the offending PR was reverted — non-issue; full text in todo_records.md) — **Write tool fails on long content payloads on this host**

## 65. (closed 2026-09-17, maintainer-ruled — NOT a tool bug, see todo_records.md for the full entry if needed) — loop_log tool folder-detection bug: spurious folders on the maintainer-renamed loop folder (2026-09-16, plan2)

## 66. (closed 2026-09-23, planner-13 bookkeeping; full text in todo_records.md) — 5.3+5.4 restart acceptance was stale: the read-scope mutation channel was proven LIVE by #68's one-shot live-acceptance (2026-09-17); the §5.4 sentinel torn down; intercept.log accumulated to 3076 lines / 139 sessions by 2026-09-23.

## 67. (closed 2026-09-28, plan30 maintenance pass — R3 build LANDED 2026-09-26 (3ec1c5c/9e91878/20d5a48, gate 337/337 + io 77/77) + live acceptance COMPLETE 2026-09-26 (plan23 re-test: grep/glob pair + bash quoted-form + bt anchor-marker pair live-accepted; section-anchor pinned-only, schema-shadowed — dormant by design on this host; see #95 closed status)) — Fuzzy scope extension: glob / grep / section-anchor resolvers (2026-09-16, plan2 queue)

## 70. (closed 2026-09-28, plan30 maintenance pass — all units LANDED + live-accepted: unit A 6864bc0 (2026-09-21) + unit B d4ef76e (2026-09-22); live acceptance 2026-09-21 (DUMP-OK PASS + config resolution; the cross model-read bug found+fixed 280b8d0 + re-accepted); the requested research-spec follow-on superseded by the #99/#101/#105 research tracks) — compact_memory rework: config-resolved summarizer + queued message + dump diagnostics (2026-09-16, new priority.md item; re-scoped 2026-09-21 by his priority.md #1)


## Closed entries

Moved to `todo_records.md` on 2026-09-10 — one-line records, IDs 2, 5, 10, 12, 13, 14, 15,
16, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 31, 32 (+ the 260908-0951 dedup block).
All those IDs stay reserved — see the numbering rule in the header.

## (closed 2026-09-15, the first commit after f4de326, worker-16) — Smoke-harness build: the tool/plugin smokes moved from the scratchpad into `.opencode/plugin/tests/` (7 smokes + shared `_smoke_base.mjs` + README, all green; gates 99/99 + 459 passed + ruff F=0) per `approved/2026-09-15_smoke-harness-home.md` — the cm_v2 + qc smokes folded into `compact_memory.smoke.mjs` (assertions adapted to the 2026-09-14 fire-and-forget build, no source change) and the `context_recovery` smoke pinned to the deactivated plugin file (attribution verified).

## 34. (closed 2026-09-10, see todo_records.md) — Stale `peek.py` documentation refs + worker prompt permission block

## 36. (closed 2026-09-09, see todo_records.md) — `agents_repo.md` `Environment & shell` — wrong/stale lines (lab-verified, fixed)

## 54. (closed 2026-09-16, plan8 planner-direct; maintainer call 2026-09-12) — Rule: never circumvent access restrictions; blocked-file protocol for agents

## 55. (closed 2026-09-17, live-accepted in direct session ses_f4f539d7c…; maintainer call 2026-09-15) — `compact_memory` needs a dump function of the current session

## 57. (closed 2026-09-16, worker-8, plan7/iter7; 2026-09-15, worker T1 block_transfer sandbox, curated plan3) — block_transfer MOVE silently deletes a block when `dstFile` is missing

## 58. (closed 2026-09-16, plan8; 2026-09-15, script-collection worker, curated plan3) — standard gate definition lacks the probe command

## 59. (closed 2026-09-16, plan6; 2026-09-15, script-collection worker, curated plan3) — session-corpus refresh cadence

## 60. (closed 2026-09-16, worker-9, plan7/iter7; 2026-09-15, planner plan5) — `block_transfer` + `loop_log` lack probe pinning

## 63. (closed 2026-09-16, plan7 worker-7; finding 2026-09-15 worker-5, planner plan5) — compact_memory smoke: 4 failures at HEAD (dump-hook sandbox gap)

## 62. (closed 2026-09-16, planner plan6) — LANDED (planner-direct): all three lines present in BOTH role prompts — (1) compaction is NOT a restart (recent messages INTACT, summary auto-created, re-read only the named head files), (2) above 90 % → EMERGENCY handover + commit + self-compact IF budget available, (3) above 95 % → commit + self-compact, DO NOT DELIBERATE while budget remains (`keepMessages` keeps the last N messages INTACT) — in `prompt_agent_planner.md` §Context-budget trigger and `prompt_agent_task.md` §Context-budget trigger + §compact_memory. Adjacent stale-ref fixes rode the same commit: stopline proposal name → `2026-09-15_agents-knowledge-stopline.md`, `dump_session.cjs` path → `scripts/db/`.

## 61. (closed 2026-09-15, planner plan5; title reworded plan6) — probe baseline corrected: post-S14 baseline is one-zero-six, not the plan3/plan4-era nine-four

## 76. (closed 2026-09-21, plan1, planner-direct; finding 2026-09-21 worker_Q3S_160K auto-resume UNIT 1 gate run, via todo_inbox.md) — stale classifier pin in `handover_probe.mjs`: check [87] expected `iq3`→1 but the `compact_memory.ts` classifier (the 2026-09-21 quant-class budget ruling) returns 3 → probe pin updated to 3; probe 235/235 green. Pre-existing (the probe references no UNIT 1 file; `compact_memory.ts` untouched).

## 77. (closed 2026-09-21, plan3, planner-direct; finding 2026-09-21 worker-3 UNIT 3 gate run, via todo_inbox.md) — stale description pin in `block_transfer.sandbox.smoke.mjs`: line 53 expected the "Housekeeping rule … instead of write/edit" sentence that the maintainer's commit `ff4c2fc` (2026-09-18) deliberately dropped from the `block_transfer` description → pin re-pointed to the new first sentence (the one-liner); smoke 52/52 ALL PASS. Pre-existing since ff4c2fc (worker-3 verified by stash; the Unit 3 commit touches neither file).

## 78. (closed 2026-09-28, plan30 maintenance pass — LANDED 2026-09-23 (6b33907, per his 2026-09-23 ruling "go for 1 + 2, plus 3+4 unified": --json raw mode + lossless full markdown + --lite preset + hook 120 s budget / pipe stderr / one retry; planner-verified: --json spot-check 70 msgs / 337 parts exact-match the live DB)) — Dump completeness: the session dumps filter out parts (thinking/writing) (2026-09-21, planner; his --info note in priority.md)

## 79. (closed 2026-09-23 - live-accepted, full text in todo_records.md) - auto_resume Unit 4 msgPairs never unwrapped the SDK { data } wrapper -> action lines were NEVER recognized (spurious recovery prompts / context drain); fixed eaef397 (dual-shape unwrap + ses_u4_wrap smoke) + LIVE ACCEPTED 2026-09-23 (route= restart spawn for a valid action: restart on both builds: v=24972ebd 12:52:39Z + v=d2b9d510 13:00:08Z, no recovery= lines for the sid)

## 75. (open, 2026-09-21, planner) — **Build our own auto-resume plugin** (opencode-auto-resume research, Phase 3 seed).
- **Problem / evidence:** the looprunner is a mechanical relay; the maintainer wants infinite direct planner sessions (his ideas.md item 2026-09-18). Three measured gaps: no auto-resume after compaction, no auto compaction trigger on context limit, no auto-restart on `action: restart`. A working reference exists and is vendored in-repo (opencode-auto-resume v1.1.16, v1-era API surface, verified compatible with our opencode-ai@1.18.31).
- **Desired outcome:** a plugin in `.opencode/plugin/` that keeps a direct planner session running through compaction and restart without the looprunner.
- **Acceptance criteria:** the four-unit acceptance list in `proposals/2026-09-21_opencode-auto-resume-plugin.md` (unit 1 = skeleton logging plugin/testbed; unit 2 = context-limit compaction trigger; unit 3 = auto-resume after compaction; unit 4 = restart detection + new planner; each unit leaves the repo green).
- **Suggested scope:** `.opencode/plugin/auto_resume.ts` (new), `knowledge/opencode-plugins/` (surface report append), the proposal file itself. Units are independently approvable, strict build order.
- **Status per unit** (full unit history in `todo_records.md` — appended by the 2026-09-23 curation):
  - Unit 1 (skeleton logging plugin/testbed): LANDED + planner-verified; LIVE ACCEPTANCE PASSED post-restart (init `surface=` line + live event lines; verdict in the unit-1 surface report).
  - Unit 2 (context-limit compaction trigger, `d90973b`): LANDED + planner-verified; the live `statusOf` shape bug found in live acceptance + fixed; LIVE re-acceptance PENDING the next host restart (`arm=` lines verified live; saturation/trigger pending a natural 85 % crossing).
  - Unit 3 (new-planner spawn helper): LANDED + smoke-verified; LIVE ACCEPTANCE PASSED (planner-run 2026-09-21: one-shot trigger file → `spawn=` line + `.consumed` rename + the spawned session wrote the acceptance file; verdict in the unit-1 surface report).
  - Unit 4 (planner liveness watchdog): LANDED + smoke-verified; PARTIALLY LIVE — case (b) recovered 2026-10-21T18:56-18:57Z (the `recovery= sid=ses_ea02d3559ffe… attempt=1` x3 lines in auto_resume.log, watchdog detected the stopped planner); case (c) `action: restart` spawn fires at THIS session's close (the successor verifies from its `spawn=` line); (a)/(d) unchanged from the 2026-09-23-era evidence (the four acceptance cases in the proposal lines 138-142).
- **NOTE:** unit numbering per the revised proposal — Unit 3 = new-planner spawn helper (shared building block), Unit 4 = planner liveness watchdog (auto-resume after compaction is its first branch); the "unit 3 = auto-resume / unit 4 = restart detection" wording is the pre-revision numbering.

## 80. (closed 2026-09-23 - fix LANDED 4098253 (worker-2) + plugin reactivated, live-accepted: (b) the injected turns keep the session agent (DB, 2026-09-22) + (c) the Direct session idle-untouched under the post-#90 build (scope=none 17:08:57Z, zero recovery/route/spawn); maintainer confirm 2026-09-23; full text in todo_records.md) - auto_resume inject calls lost the session agent → the turns ran as "build" (the prompt-cache invalidation + the planner lost its system prompt); fixed: the injected bodies carry the session's CURRENT agent+model (the last REAL assistant's info — #91: the compaction summary is skipped; else the JSONC configured-model fallback; else the host default — never a planner constant)

## 81. (closed 2026-09-23 - re-pin LANDED af38e2f (worker_Q3S_160K), gate green (probe 241/241, smoke 53/53, pytest 459+1w, ruff F=0), maintainer confirm 2026-09-23; full text in todo_records.md) - probe [97] + the compact_memory smoke message pin red since the maintainer's temp fix 0f192e5 (the queued promptAsync commented out); re-pinned to the temp-fix behavior (the dispatch line + the queued note byte-exact, NO queued promptAsync — NOT deactivated, promptAsync NOT restored)

## 82. (closed 2026-09-23, planner-13 bookkeeping; full text in todo_records.md) — scope toggle (last-toggle-wins, own-line anchor, bidirectional) LANDED as #85 part 1 (97fccfc) + the unit-2 suppression ruling (Direct suppresses Unit 2 = the already-landed #85 part 3 behavior, no code change); FULL live acceptance 2026-09-23 on the #90 spawn tail (own-line toggle judged scope=autorun, route= restart spawn, successor's first message carries the own-line marker, trigger deactivate=).

## 83. (closed 2026-09-27, direct session ses_f20d1b39… — his autoCompact question answered: nudge gate only, no autocompact; backstop live-verified via #93; the threshold knob = nudge position, stays in the budget file (0.85 confirmed by his 2026-09-27 ruling); full text in todo_records.md) — unit-2 backstop: catch the ACTUAL context-limit hit cleanly

## 84. (LANDED 2026-09-22, worker `worker_Q3S_170K`) — Compaction config consolidation: ALL compaction config in the shared `compact_budget.json` (replaces the `QUANT_CLASS_RULES` substring table + the opencode.jsonc emergency flag + the hardcoded recovery keeps)

## #85. (part 1 (scope) LANDED 2026-09-22 / part 2 (current-agent+modelID + dead-mark) LANDED 2026-09-23 / part 3 (Unit-2 ctx-line-suffix + Direct gates) LANDED 2026-09-23; live verification PENDING post-restart, HIGH priority — maintainer call) — auto_resume unbounded session-spawn loop
- **Problem + evidence:** on 2026-09-22 (~19:57–20:00Z) the LIVE auto_resume
  plugin created a NEW session every ~10s (every 2nd recovery attempt — 2
  attempts per session at 5s each, confirmed by the session naming) —
  auto_resume.log shows the `ses_f354e4cb… / ses_f354e259… / ses_f354dfe7… / …`
  series, each logged as `scope= planner` + `recovery= attempt 1` then
  `attempt 2`, each failing with `UnknownError` at `SessionPrompt.createUserMessage`.
  The maintainer saw `UnknownError` every 5s in the terminal and had to RESTART
  opencode to stop it. (Diagnosed from auto_resume.log this turn, 2026-09-22.)
- **Desired outcome:** auto_resume must never create sessions unboundedly. A
  spawn / continue / recovery that keeps failing (or keeps re-triggering on
  freshly-spawned sessions) must STOP: a GLOBAL cap (not just per-session) + a
  dead-mark (a session whose recovery/continue attempt fails is marked dead and
  skipped on later cycles), and freshly auto-spawned sessions must NOT be
  re-scoped as in-scope planners.
- **Acceptance criteria:** (a) under load a failing spawn/continue is globally
  capped and the offending session is dead-marked + skipped (no per-cycle
  retry); (b) auto-spawned sessions are not re-scoped as planners; (c) an
  `UnknownError` on a failed `createUserMessage` discards the session (no
  retry loop); (d) a test/repro showing the loop stops after the cap.
- **Suggested scope:** `.opencode/plugin/auto_resume.ts` — the spawn path
  (unit-4 "restart→spawn") + the recovery path; the global cap + dead-mark; the
  scope logic. TIES IN with the earlier design issue (unit-4 scope / the
  spurious worker resume / the action-line dilemma) — all resolved by one scope
  refinement (below).
- **GENERALIZED SCOPE (his ruling 2026-09-22):** unit-4 scope = actual PLANNER
  sessions (always) ∪ sessions started/toggled via `<|Autorun|>` (the #82
  own-line toggle). NO new marker needed — the #82 toggle already gates scope
  ON/OFF; unit 4 just follows it for ANY agent type (drop the planner-only
  gate). This lets him run other agents (prompt_builder, a future researcher,
  etc.) in a loop by toggling them with `<|Autorun|>`. Bonus: freshly-spawned
  sessions + unmarked worker sessions stay OUT of scope (not a planner, not
  `<|Autorun|>`-marked) → fixes the #85 loop AND the spurious worker-resume.
 - **Status:** part 1 (scope) LANDED 2026-09-22 (worker worker_Q3S_170K —
   the #82 generalized scope in auto_resume.ts + smoke: real-planner-via-
   agent-field ∪ last-own-line-toggle-ON for ANY agent type; the `spawned`
   self-mark is now an EXCLUSION, so a self-spawned successor is never
    re-scoped → the #85 loop condition is gone — smoke 97/97 + gate green:
    probe 241/241, all smokes, pytest 459+1w, ruff F=0; the commit hash is
    recorded by the planner in the follow-up bookkeeping commit). PART 2
    (current-agent+modelID in the injected bodies + dead-mark on a failed
    send) LANDED 2026-09-23 (worker worker_Q3S_170K — the hardcoded
    PLANNER_AGENT_ID is replaced by the session's CURRENT agent+modelID,
    resolved at fire-time: last-assistant info → opencode.jsonc agent-
    config fallback (cached) → host default (NEVER a planner constant); a
    failed CONTINUE dead-marks the idle cycle — remaining retries + the
     cap-exhaustion fallback spawn are skipped (no doomed successor),
     cleared on a fresh busy — auto_resume smoke 105/105; the commit hash
     is recorded by the planner in the follow-up bookkeeping commit).
     PART 3 (the live test exposed the Unit-2 tick design: re-fire on
     stale armed sessions + a self-loop) LANDED 2026-09-23 (worker
     worker_Q3S_170K — the Unit-2 tick leg is REMOVED: the nudge is a
     PASSIVE ctx-line SUFFIX on the session's OWN tool-call return (the
     gauge plugin's ctx: line channel — tool.execute.after), per busy
     session, ladder (>= 0.95 "self-compact now" / >= 0.98 --maintainer-
     flagged), scope "none" (Direct) suppresses it (c); NO promptAsync on
     the Unit-2 path (no resume, no loop, stale sessions unreachable by
     construction); scopeVerdict checks the LAST OWN-LINE TOGGLE FIRST —
     Direct deactivates Unit 4 for the planner (d); the Unit-4 scope is
      unchanged — auto_resume smoke 102/102 (baseline 105: the old
      promptAsync-era checks adapted to the passive mechanism) — ded7245
      (code + smoke + TODO) + 5992f38 (handover), recorded by planner-8 in
      the 2026-09-23 bookkeeping commit). The post-restart live
      verification stays a maintainer call.
     The orphan-session cleanup is DONE (the maintainer removed all the
  - **LIVE EVIDENCE 2026-09-23 (plan10, new build v=d2b9d510):** the Unit-4 restart branch fired correctly on planner-9 close (`route= restart spawn` 13:00:08Z -> the named spawn `ident=autorun-2026-09-21_15-33 planner-10`, session ses_f31a5dee5ffe1DIBxZzEDZF8aF); the trigger session was NOT re-routed afterwards (no recovery=/route= lines for its sid) and no unbounded-spawn loop recurred. His 2026-09-23_14-25 item now directs a change to the spawned-exclusion design (TODO #90).
     new sessions, 2026-09-22).

## #86. (DEFERRED — maintainer-proposed 2026-09-23, curated from todo_inbox 2026-09-23_01-08) — worker audit of ALL `.opencode/plugin/` + `agent/scripts/` tools/plugins
- **Problem:** stale hardcoded agent ID found in `auto_resume.ts`
  (`PLANNER_AGENT_ID planner_Q3S_160K` — already handled by #85 part 2);
  likely the same class elsewhere: hardcoded agent/model IDs not matching
  the live backend/roster, magic numbers, duplicated config, dead
  constants.
- **Desired outcome:** a prioritized findings list — ID + file + line +
  suggested fix — filed to `todo_inbox.md`. No behavior change (read-only
  audit).
- **Acceptance:** findings list covering `.opencode/plugin/*.ts` +
  `.opencode/plugin/tests/` + `agent/scripts/` (node/cjs); no
  edits; the `auto_resume.ts` PLANNER_AGENT_ID case excluded (handled by
  #85 part 2).
- **Suggested scope:** explorer or worker, read-only grep-driven scan.
- **Status:** CLOSED (2026-09-30, plan47, planner-47 — the audit RAN,
  explorer-47 read-only, zero code changes): 11 findings, all in the
  `plan47 TODO #86 cluster A-D` todo_inbox entries — 0 dead code (125
  exports verified used), 0 stale ses_* in non-test code, 2 low stale-ids,
  3 dup-config, 5 magic-numbers, 1 stale-ref. The 2 med findings + the
  prompt-index stale refs were fixed by planner-47 the same session
  (61cd616 / f7f4100); the remaining LOW findings are carried by
  TODO #125.

## #87. (closed 2026-09-23 - subsumed by #90; full text in todo_records.md) - Unit-4 cannot revive a dead self-spawned successor (the plugin-driven loop stalls): RESOLVED by #90 Part A — the self-spawned successor is now TRACKED (scope "autorun" via the restart prompt's own-line toggle) and RECOVERED after an idle without an action line (the #87 stall case, inverted); option (b)'s "committed progress" idea is replaced by the lineage-depth cap (N=2) on the spawn branch.

## #88. (LANDED 2026-09-23, worker-14 `worker_Q3S_170K`) — auto_resume smoke wall-time cut 91.4 %: 145.5 s → 12.5 s (his 2026-09-23_00-12 complaint measured at 145.5 s, not ~300 s) — the tick period is now a DEFAULT-PRESERVING factory option (`tickMs`, default 5000 ms — the live tick is unchanged; first factory call sets the module-level tick); the smoke instantiates with `tickMs: 300` and its 7× `sleep(5600)` became `tickWait()` (2 ticks + margin — same "at least one full tick period" pin semantics); no check removed (102/102 before AND after); gate re-verified: probe 241/241, all 10 smokes, pytest 459+1w, ruff F=0. Commit 532ddbc.

## #89. (closed 2026-09-23 - live-accepted plan10; full text in todo_records.md) - autorun-identifiable names for plugin-spawned sessions (title <loop-folder> planner-<N>): LIVE - the first named spawn verified 2026-09-23 13:00:08Z (the spawn= line carries ident=autorun-2026-09-21_15-33 planner-10 + the session title in the DB)

## #90. (closed 2026-09-23, planner-13 bookkeeping; full text in todo_records.md) — spawned successors inherit the trigger's Autorun state + the trigger deactivates (Parts A+B+C, commit c4b244d, worker_Q3S_170K) — FULL live acceptance 2026-09-23 (planner-13, build v=7d2e6207): `route= restart spawn` + `spawn= agent=planner_Q3S_170K ident=autorun-2026-09-21_15-33 planner-13` + `deactivate=` (19:20:55Z) + the restartText own-line `<|autonom|>` as the successor's first message + zero `skip=` depth-cap lines; subsumes #87 (closed).

## 93. (closed 2026-09-27, direct session ses_f20d1b39… — live-verified by his repeated overflow tests: recovery fires on overflow, compacts, no resume, session continues; full text in todo_records.md) — Emergency compact backstop: port context_recovery.ts to the `event` hook

## #91. (open, 2026-09-23, planner-12 direct; live incident 14:44–14:46Z; HIGH) compaction summary leaks into spawn identity + unit-4 routing (agent=compaction successor; mis-route on a quoted action line)
- **Problem / evidence:** live incident 2026-09-23 14:44–14:46Z (pre-#90 build d2b9d510): planner-11 (ses_f318f0d77…) self-compacted (14:44:19Z) → the compaction summary is stored as an ASSISTANT message with info.agent=compaction + info.model=<session model> (the same-model fallback — agent.compaction.model is commented out). The summary text QUOTES "Previous planner-10 … closed with `action: restart`" (6612 chars). The unit-4 tick (14:46:06Z) then: (1) `lastAssistantAction` scanned the summary (the last assistant message) → found the QUOTED `action: restart` → ROUTE= RESTART SPAWN (mis-route — the planner's actual last real turn, 14:44:12Z, was a 76-char no-action-line close; the loop log records "self-compact + unit-4 resume per maintainer inbox"); (2) `resolveInjectIdentity` → the summary's info → the spawn body agent=compaction + model=Qwen3.8-27B-Q3S-170K (the spawn= line 14:46:06.647Z). "compaction" is NOT an agent defined in the live opencode.jsonc → the successor (ses_f3144d9d6…, ident=…planner-12) ran with the WRONG agent + mode (maintainer: "i started the session prior with autorun but the wrong agent and mode. thus reverted and then started you with direct"). DB evidence: the ses_f318f0d77 message 14:44:19.866Z (assistant / agent=compaction) + the auto_resume.log lines 14:46:06.629–647Z.
- **STILL LATENT in the live build (ef8c6149 = #90):** `lastAssistantInfo` (L921-930) + `resolveInjectIdentity` (L1052-1060) + `lastAssistantAction` (L1065-1077) do NOT skip the compaction agent → recurs whenever an autorun session self-compacts immediately before the idle routing decision (the near-limit protocol makes this common): the successor is spawned with the wrong agent, AND the routing reads action lines QUOTED in the summary instead of the real close.
- **Desired outcome:** identity + routing resolve from the last REAL turn (the compaction summary is never used for either).
- **Fix design (recommended — maintainer call, observable behavior change):** skip assistant messages with info.agent === "compaction" in BOTH `lastAssistantInfo` (→ the spawn/inject identity falls back to the last real assistant message: the planner's agent+model) and `lastAssistantAction` (→ the routing reads the pre-compact close: a no-action-line Work State dump → recovery, as intended). One guard each; smoke fixture: a messages() shape whose last assistant message = a compaction summary (agent=compaction, text quoting "action: restart") → (a) the routing returns null (recovery), (b) the spawn body agent = the preceding real assistant's agent.
- **Acceptance criteria:** the smoke pins above + gate green; LIVE: the next autorun self-compact→idle cycle (a) routes correctly (no mis-spawn from a quoted line), (b) the spawn= line carries agent=planner_* (not compaction).
- **Suggested scope:** `.opencode/plugin/auto_resume.ts` (L921-930, L1052-1060, L1065-1077), `.opencode/plugin/tests/auto_resume.smoke.mjs`.
- **Status:** fix LANDED 2026-09-23 (planner-12 direct, planner-implemented after his "approved") — the one-guard-each fix per the design (skip agent="compaction" in `lastAssistantInfo` + `lastAssistantAction`, auto_resume.ts); smoke re-pinned 121/121 (+3 #91 pins e1/e2) + gate green (probe 241/241, all smokes, pytest 459+1w, ruff F=0). Commit **2fa4bb6**. REMAINING — live acceptance (needs his restart to the new build + the next autorun cycle): a self-compact→idle cycle must (a) route from the real close (no mis-spawn from a quoted line), (b) the spawn= line carries agent=planner_* (not compaction) — this also completes the #90 new-build spawn tail (DONE 2026-09-23). **Spawn-tail live acceptance (planner-13, build v=7d2e6207 live, process start 19:13:25Z):** the `spawn=` line carried `agent=planner_Q3S_170K` (the real agent, not compaction) and `route= restart spawn` came from the real close — no mis-route off the quoted 17:32Z compaction summary in this episode. NOT YET live-discriminated: the failure case needs the compaction summary to be the LAST evaluated turn (a self-compact then idle cycle in an autorun-scoped session) — the maintainer's re-engagement + own-line toggle + real close all followed the 17:32Z summary, so the guard was not the deciding factor here; the guard is smoke-covered (3 pins) and the discriminating cycle will occur naturally under the near-limit protocol.

## #92. (LANDED 2026-09-27, 2026-09-23, planner-12 direct; his question; SMALL) pre-compaction hook should save BOTH the lossless full markdown AND a raw `--json` snapshot
- **Problem / evidence:** after #78 the hook keeps only the markdown backup. His question (2026-09-23): with raw JSON as the on-demand `--json` mode (DB-sourced), how do you get the pre-compaction JSON later — "the db is overwritten by the compacted session"? MEASURED on this session (compacted 17:32:02Z): the DB is NOT overwritten — 36 pre-compaction messages + 170 parts are still in the DB, 76 reasoning parts whole-session (the compaction summary is an ADDED assistant message, agent=compaction; planner-11's session showed the same). So on-demand `--json` of a compacted session works TODAY. BUT the DB is a live host-managed store (session deletion / pruning / migration / corruption can lose the rows) — the hook's _c0 snapshot is the only PINNED "state at compaction time" record; markdown-only forfeits the lossless source at exactly that moment.
- **Desired outcome:** every pre-compaction dump writes BOTH: the lossless full markdown (current) + a raw `--json` snapshot (the lossless master — any later filtered view re-derivable from it).
- **Acceptance criteria:** two DUMP-OK lines per dump (md + json); both files land in the archive dir; the #78 diagnostics (ms= / DUMP-RETRY= / DUMP-FAIL + stderr) + the 120 s budget + one retry apply to BOTH dumps; compact_memory smoke re-pinned; gate green.
- **Suggested scope:** `.opencode/plugin/compact_memory.ts` (the preCompactionDump call site — the dump script already has `--json`), `.opencode/plugin/tests/compact_memory.smoke.mjs`.
- **Status:** LANDED 2026-09-27 (worker-26, plan26 — per the 2026-09-27 ruling "save both"): `preCompactionDump` now writes BOTH artifacts — the lossless md (first) + the raw `--json` snapshot (the lossless master) — each INDEPENDENTLY (the same attempt → DUMP-RETRY=1 → one retry → DUMP-FAIL per artifact, the #78 120 s budget + stderr capture on both); the DUMP-RETRY=/DUMP-FAIL lines gain the artifact relFile; the hook returns `{ ok, files, error }` (files = the landed paths, md first; error = `<md|json>: <detail>` per failed artifact, joined by " | "); the no-overwrite stamp stays MD-base-driven (both artifacts share the stamped base); the dispatch call site reads only ok/error (unchanged). The probe S14 checks 104/105/106/107 re-pinned + NEW check 345 (the json naming byte-exact) — 345/345 (header self-annotated); the compact_memory smoke +2 (78/78: the json artifact exists + the json DUMP-OK line byte-exact); gate green (all smokes, pytest 459 passed + 1 warning, ruff F=0). **Closed (planner-26 bookkeeping, 2026-09-27):** commits `11f4a12` (code) / `5a6e842` (probe) / `e2a1a52` (smoke) / `86bda25` (this status + handover) — planner-verified by own re-run (probe 345/345 + compact_memory smoke 78/78, exit 0). **LIVE-ACCEPTED (planner-27, 2026-09-27, verified from files):** the first real compaction under the #92 build was the planner-27 part-2 self-compact (ctx.log 2026-09-27_15-42/15-45, ses_f1d198b41): TWO DUMP-OK lines (`…_c0.md` ms=49 + `…_c0.json` ms=50) + the COMPACT line (keep=12m tok=29498 computed); both artifacts on disk in `archive/sessions/compaction_dumps/` — the md (616,987 B, correct session/title/agent header) and the json (1,216,263 B, valid JSON, top keys `session,messages,orphan_parts`).

## #95. (CLOSED 2026-09-28, plan28 maintenance pass; was open, 2026-09-25, planner direct; his rulings 2026-09-25) fuzzy edit-oldstring track — PARENT entry (replaces the stale #67 references — that ID never existed in the committed TODO.md)
- **Problem / evidence:** edit oldString exact-match failure is a very regular problem (his priority.md "fuzzy matching of edit oldstring" + ideas.md L143-158); the #94 worker's anchor-semantics drift finding is queued here (todo_inbox 2026-09-25). Track state: R1/R2 live, R4/R7 landed, R6 STAGED (gate cleared), R3 STAGED, R8 + the escape return-info not staged.
- **Sub-items (order):** (1) R6 as-is (pre-approved per its spec: payload journal + edit hints, observation-only); (2) the MUTATING edit-fuzzy (design pinned below — spec at launch); (3) R3 (gate CLEARED by his ruling 2026-09-25 — bitdrift retired, no correction data, R4 mining retired; absorbs the anchor-drift fix: existing block_transfer modes `includes` + no unique-check → startsWith+unique per #94); (4) R8 sandbox redirect + escape return-info.
- **Pinned design for (2) (agreed 2026-09-25 direct):** oldString miss → the R6 content-locator candidates (anchors = distinctive lines of the oldString) → **normalize BOTH sides before comparing** (`\r\n`→`\n` + strip per-line trailing whitespace) → d=0 normalized → mutate oldString to the file's exact bytes (the CRLF/LF + trailing-whitespace class is unbounded in length — a proportional bar like 1-in-100 REJECTED: a 40-line CRLF drift = 39 chars, far beyond any proportional cap, while normalization removes it exactly); d≤1 normalized → mutate ONLY on exactly-one candidate (typo tolerance, the #72/M1 strict bar); else fail-closed + the edit-hint line; mandatory log line kind=fuzzy-edit orig=/value= (his return-info ruling); no auto-retry beyond the mutation; the R6 journal stays the recovery fallback.
- **Acceptance:** per sub-item (each spec at launch); overall: a controlled CRLF-drift edit and a single-typo edit both resolve without agent action (log evidence), gate green at each landing.
- **Suggested scope:** `.opencode/plugin/intercept_observer.ts` (+ core), `.opencode/plugin/tests/`, `.opencode/plugin/probes/handover_probe.mjs`, the `research/fuzzy-numword/` specs.
- **Status:** LANDED 2026-09-26 (all four sub-items: (1) R6 acb6323;
  (2) edit-fuzzy 15761d8+78b68e7; (3) R3 3ec1c5c/9e91878/20d5a48;
  (4) R8 + return-info #97 07bdd56/0d9b8e6). Live acceptances
  2026-09-26 (plan22): the grep/glob pair + bash quoted-form R3 channels
  live-accepted (worker-22) + the #97 Windows-root redirect form
  (planner spot-check); the section-anchor + bt anchor-marker channels
  NOT live-verifiable — ROOT CAUSE: the live opencode process was
  restarted DURING the 14-26 incident, before the R3 import fix 9e91878
  landed, so both anchor channels throw the swallowed
  `LOCATOR_MAX_FILE_CHARS is not defined` ReferenceError live (planner
   spot-check: a pair-form startMarker reached the tool VERBATIM + the
   `intercept-error` log line); the section-anchor channel is additionally
   shadowed by the integer `offset` schema (constrained decoding — the
   worker's 7/7 integer-1 observation).
   2026-09-26 (plan23, planner-23, post-restart): the re-test — the
   import-fix 9e91878 is now LIVE (canary: anchor channels run clean, zero
   new `intercept-error` lines). (b) the bt anchor-marker PAIR channel is
   LIVE-ACCEPTED — the pair-form `startMarker` reached the hook VERBATIM +
   resolved (`pair=[7:seven] canon=7 dist=0 gate=mutated line=2 arg=
   startMarker ... pair-resolved`, 2/2; the log is the authority — the
   tool return shows only the post-mutation canonical form, correcting the
   plan22 "model can't emit pairs" reading as a MEM-0101 false-repeat).
   (d) the section-anchor channel stays schema-shadowed — `read.offset` is
   integer-typed, so constrained decoding never delivers a string anchor
   (the worker's 7/7 integer-1 stands); the pure resolver remains
   probe-pinned (S31 check 319) while the live channel is dormant on this
   host. R3's live acceptance is now COMPLETE for this model class (the
   four channels: grep/glob pair + bash quoted-form + bt anchor-marker
   pair live-accepted; section-anchor pinned-only).
   2026-09-25 (autorun, ses_f29afbb66ffeRM1EHgBIpTwTg5, planner-14): his
  GO for sub-item (2) — the normalize-then-compare design is CONFIRMED
  (his words: better than the proportional distance rule; go ahead) +
  two directives folded into the (2) spec: (a) resolutions AND failed
  resolutions are LOGGED — every attempt, the line carries the best-
  candidate d (a near-miss d<10 is visible; no d bar on logging),
  (b) the return feedback does NOT carry the full oldstring (context
  saving — a truncated identifier: first ~40 chars + length + d +
   target; the full payload stays in the R6 journal). R6 (sub-item 1)
   LANCHED (worker_Q3S_170K, spec plan14_ho_task.md).
   2026-09-25 (autorun, ses_f29afbb66ffeRM1EHgBIpTwTg5, planner-14,
   post-compaction): sub-item (1) R6 LANDED + planner-verified from file
   evidence (the worker's close-out died silently — the state was
   committed from the working tree): content-locator `locateContent`
   (VERDICTS 11) + the payload journal (journal_write.log /
   journal_edit.log) + the edit-hint channel (edit-hint /
   edit-ambiguous / no-candidate) + the after-hook enrichment (live
   acceptance restart-gated) + the DoD machine check. Gates: probe
   279/279 (baseline 259 + S26's 20), intercept_observer smoke 48/48,
   pytest 459+1w, ruff F=0. Docs: the plugin README recovery protocol +
   decision-record §8.1 addendum. Next: sub-item (2) the edit-fuzzy spec
   (normalize-then-compare).
   2026-09-25 (worker ses_f27bb616dffe8yue93zR3sEwHs, worker_Q3S_170K,
   resumed post-compaction): sub-item (2) the MUTATING edit-fuzzy
   oldString (normalize-then-compare) LANDED: `runEditFuzzy` supersedes
   the R6 hint path for the 0-raw-occurrence case — exactly-one
   candidate at d=0/d≤1 → oldString MUTATED to the file's exact unique
   bytes + the `fuzzy-edit` line (VERDICTS = 12; NO after-hook hint);
   else FAIL-CLOSED (the R6 verdict carrying the best-candidate d —
   directive a; the after-hook hint stored). Directive b: the feedback
   line truncated (first 40 + ...), the journal's edit `old` = the
   ORIGINAL pre-mutation oldString. Code commit 15761d8; the probe/docs
   commit hash is in the worker handover (the planner records it in the
   follow-up — no self-reference). Gates: probe 287/287 (baseline 279 +
   S27's 8; the S26 re-pins 271/275/276), intercept_observer smoke 55/55
   (from 48/48), pytest 459+1w, ruff F=0. Docs:
   spec_sub2_edit_fuzzy_oldstring.md + decision-record §8.2 addendum.
    Next: sub-item (3) R3 (gate cleared — absorbs the anchor-drift fix).
   **CLOSED (plan28 maintenance pass, 2026-09-28):** all four sub-items
   LANDED 2026-09-26 — (1) R6 `acb6323`, (2) edit-fuzzy `15761d8`+`78b68e7`,
   (3) R3 `3ec1c5c`/`9e91878`/`20d5a48`, (4) R8 + return-info `#97`
   `07bdd56`/`0d9b8e6` — + the R3 live-acceptance is COMPLETE for this
   model class (plan23 2026-09-26 re-test: grep/glob pair + bash quoted-
   form + bt anchor-marker pair live-accepted; section-anchor pinned-only,
   schema-shadowed — see the status text above). Entry complete; full text
   now in `todo_records.md`.

## #94. (LANDED 2026-09-25, planner direct; his approval 2026-09-25) block_transfer REPLACE mode — line-anchored span replacement from a buffer (edit-like, no exact oldString)
- **Problem / evidence:** edit oldString exact-match is a very regular failure (his priority.md "fuzzy matching of edit oldstring"; ideas.md L153-158: "what would be needed to make block_transfer as versatile as edit but less prone to oldstring mismatch?"); block_transfer PASTE is insert-only (append after targetMarker / EOF) — a slot/region replacement needs a MOVE+DELETE composition (two calls, intermediate state); the 2026-09-24 slot-clobber incident (agent_feedback) showed PASTE-as-slot-replacement is a trap.
- **Desired outcome:** one atomic call replaces the line-anchored span (short unique line-prefix anchors — no exact oldstring, no whitespace sensitivity) with the named buffer content; the return string reports what was replaced (perceptibility — the caller cannot see its own args after the call).
- **Acceptance criteria:** smokes 30/30 + 53/53 (from 22/52); probe S15 = 12 checks, total 259; standard gate green; the tool description documents REPLACE; a knowledge note appended.
- **Suggested scope:** `.opencode/tools/block_transfer.ts`, `.opencode/plugin/tests/block_transfer*.smoke.mjs`, `.opencode/plugin/probes/handover_probe.mjs` (S15), `agent/knowledge/plugin_tools/`.
- **Status:** LANDED 2026-09-25 (worker, `worker_Q3S_170K`): `REPLACE` mode in `.opencode/tools/block_transfer.ts` (enum + the dispatch branch next to PASTE + the description MODES/ANCHORS/BUFFERS/EDGE text) — the line-anchored span (short UNIQUE line prefixes, `startsWith`, non-unique → an error naming the cause, start..end INCLUSIVE, start ≤ end) of an EXISTING dstFile is replaced atomically by the named buffer (REPLACE never creates a file; the buffer is preserved, PASTE semantics; all checks before any fs write; the return reports the 1-based line span + counts); smokes re-pinned 30/30 (+8) + 53/53 (+1); probe S15 = 12 checks (+262/263), probe total 259/259 (header totals machine-updated); gate green (pytest 459 passed + 1 warning, ruff F=0); knowledge note appended at `agent/knowledge/plugin_tools/2026-09-25_block_transfer.replace_mode.md`. (Commit hash recorded in the planner's follow-up bookkeeping commit — no self-reference.) The broader fuzzy-oldstring track (anchor-based fuzzy oldString resolution + the edit/write journal dump per his ideas.md L146-150 + the priority.md fuzzy_numword items) is a SEPARATE track awaiting design ruling.

## #96. (CLOSED 2026-09-25, planner-17 live-verified post-restart; full text in todo_records.md) — auto_resume.log write-volume reduction: delta exclusion + init size guard LANDED (worker-Q3S-170K, 22c36e4/70399ea, smoke 133/133); LIVE-VERIFIED 2026-09-25 (planner-17, post-restart): `log-trim= old=293026007 new=2097152` (12:50:57Z — the 293MB file trimmed to 2MB at init) + zero `message.part.delta` lines appended after the trim (the last delta line predates the trim; the new build's lines are delta-free).

## 97. (closed 2026-09-28, plan30 maintenance pass — LANDED 2026-09-25 (worker-17: 07bdd56 Unit 1 redirect + 0d9b8e6 Unit 2 escape return-info + 6684991 handover; gate green) + LIVE-ACCEPTED 2026-09-26 (plan22 planner spot-check: the Windows-root form redirected 1:1 into the scratchpad, kind=redirect line delivered)) — R8 sandbox redirect: out-of-sandbox path args redirected INTO the sandbox (2026-09-25, his live priority.md edit labeled "TODO #97"; planner-14 filed)

## #98. (LANDED, 2026-09-25, worker-16; Parts A+B implemented + smoke re-pinned, Part C not-applicable per the planner ruling) unit-4 resume-after-compaction: line-anchor the action regex (A) + re-arm on COMPACT (B) + prompt note (C)
- **Problem / evidence:** `proposals/approved/2026-09-23_unit4-
  compaction-resume.md` (his `--comment` "approved A, B and C") —
  verified NOT implemented 2026-09-25: `ACTION_RE` (auto_resume.ts
  L267) is still the unanchored `/action:\s*(restart|resume|stop|
  ask_maintainer)/g` (a prose-quoted action line still drives routing —
  the live-verified 2026-09-23 20:14Z mis-route) and there is NO
  ctx.log `COMPACT` tail-read in the tick (a landing compaction is
  silent → the unit-4 recovery-continue promise is non-deterministic).
- **Desired outcome:** per the proposal — Part A: `action:` matches only
  at line start (`/(^|\n)\s*action:\s*(…)/g`, last match still wins);
  Part B: on the 5s tick, tail-read `.opencode/temp/ctx.log` for NEW
  `COMPACT <sid>` lines → `idlePending = true` + `recoveryCount = 0`
  (fresh budget — the context situation changed); Part C: the Work
  State dump form should not quote a literal `action: restart` in
  prose (prompt-text note — planner-as-text-worker, workers have no
  prompt edit access).
- **Acceptance:** per the proposal: smoke pin A (a last assistant
  message with a PROSE-quoted mid-line `action: restart` →
  `lastAssistantAction` null; a standalone line → "restart"), smoke
  pin B (a synthetic ctx.log `COMPACT` line for a watched sid → re-armed
  on the next tick), standard gate green; LIVE: the next self-compact
  → idle cycle routes from the real close (no mis-spawn from a quoted
  line) + a `recovery=`/`route=` line follows the `COMPACT` line
  WITHOUT a user message in between.
- **Suggested scope:** `.opencode/plugin/auto_resume.ts` (L267, the
  tick), `.opencode/plugin/tests/auto_resume.smoke.mjs`; Part C =
  prompt text (planner, not the worker).
- **Status:** LANDED (worker-16, 2026-09-25, commit 4f90218 Part A +
  cf7e6f5 Part B) — Part A: `ACTION_RE` line-anchored (NON-capturing
  `(?:^|\n)` — group 1 stays the action word); Part B: the 5s tick
  tail-reads `.opencode/temp/ctx.log` for NEW `COMPACT <sid>` lines
  (module-level `ctxLogOffset` cursor, partial-line hold-back,
  never-throw) → for a WATCHED sid: `idlePending=true` +
  `recoveryCount=0` BEFORE the routing loop (the silent-compaction gap
  closed). Smoke 139/139 (133 baseline + 6 new: A1/A2/B1/B2/B3 + the
  section re-factory chk; 17 scripted closing texts re-pinned to
  own-line action lines — the pins' routing assertions unchanged).
  Standard gate green: pytest 459 passed + 1 warning, ruff F=0, probe
  291/291 (no probe pin breakage — the existing probe pins use
  standalone `action:` lines). LIVE acceptance (the next self-compact
  routes from the real close; a `recovery=`/`route=` line follows the
  `COMPACT` line WITHOUT a user message in between) = PENDING live
  observation. Part C: not-applicable (the Work State dump form was
  removed in the 2026-09-24 rework — the planner records it in the
  handover).

## #99. (open, 2026-09-25, planner-15; his priority.md top item 2026-09-25) compaction keep: the plugin must pass keepTokens (computed primary, budget-file fallback) — keepMessages alone does not control the retention
- **Problem / evidence:** maintainer live measurements 2026-09-25
  (priority.md top item, his words): "keepToken was worken [worked]. with
  30000 keepToken the newly compacted session was around 55k token. with
  the keepToken 0 and keepMessages X it was always around 25K token after
  compaction. so the 30k keeptoken were a direct retention that added
  25k +30k to the observed 55k" — `keep.tokens` = direct retention on a
  CONSTANT ~25k base (system + summary + minimal tail); the
  `keep.messages` count does NOT control the retention (always ~25k
  regardless of X). Our compact_memory sends only `keep.messages` in the
  summarize body (keepTokens removed 2026-09-24 spec 01 as "never
  respected" — superseded: the respected knob is `keep.tokens`). The
  installed SDK's summarize body schema is `{ providerID, modelID }`
  ONLY (no documented `keep` field — the body field's effect is
  UNVERIFIED; the live retention tracked the CONFIG
  `compaction.keep` in opencode.jsonc). Dev-branch host source
  (maintainer link, `session/compaction.ts`): token-budget tail
  (`preserve_recent_tokens ?? clamp(0.25*usable, 2k..15k)` + optional
  `tail_turns`) — no message-count knob. His ask: "settings for keepToken
  as fallback from compact_budged.json" + "can we caluculate the actual
  keepToken based on the dump and then supply the correct keepToken to
  exactly keep these messages?" — the budget file already carries
  `keepTokens: 30000` + `keepMessages: 18` (his live edit).
- **Desired outcome:** at dispatch the plugin RESOLVES keepTokens and
  passes `keep.tokens` in the summarize body: (1) primary = COMPUTED
  (token size of the last `keepMessages` messages from the DB — user:
  `tokens.input`, assistant: `tokens.output + tokens.reasoning`;
  dual-shape unwrap per #79 — the DB is the clean source of "the dump"),
  (2) fallback = `keepTokens` from the budget file (30000) when the read
  fails / sum 0, (3) else omit (host config default). Body keeps
  `keep.messages` too (his config comment: non-zero tokens wins). The
  resolution source is logged in the COMPACT line. Tool schema unchanged
  (keepMessages stays the agent-facing knob — it drives the
  computation). context_recovery.ts gets the same resolution.
- **Acceptance:** computed / budget / none paths each smoke-pinned;
  COMPACT line carries the resolved tokens + source; standard gate
  green; LIVE (maintainer fork test post-restart): a self-compact with
  a computed keep.tokens (~27k) → post-compaction ≈ 25k base + 27k ≈
  52k (NOT 55k, NOT 25k) proves the body `keep.tokens` is honored; if
  ignored (result tracks the config) → his fallback ruling (config-level
  knob is his file; the plugin's value stays logged as advisory).
- **Suggested scope:** `.opencode/plugin/compact_memory.ts`,
  `.opencode/plugin/context_recovery.ts`,
  `.opencode/plugin/tests/compact_memory.smoke.mjs`,
  `.opencode/plugin/tests/context_recovery.smoke.mjs`,
  `.opencode/plugin/probes/handover_probe.mjs`,
  `knowledge/opencode-plugins/` (dated note).
- **Status:** IMPLEMENTED (worker-15, 2026-09-25, commit c5859c7 +
  ebf59b2) — plugins + smokes + probe all green (291/291, 74/74, 17/17,
  459+1w, F=0). The LIVE fork test (acceptance item: self-compact with
  computed keep.tokens ~27k → post-compaction ≈ 52k, proving the body
  `keep.tokens` is honored) is PENDING the maintainer's post-restart
  fork test — that part stays OPEN. Design call (planner, veto-able):
  computed PRIMARY, budget `keepTokens` the fallback (his question (b)
  is the fix; the budget 30k stays the safety value when the DB read
  fails). SIDE NOTE (his observation, recorded): the history is
  completely DROPPED and REPLACED by the summary → no bit-rot from a
  compaction chain, only from an N-summarized summary (the budget cap's
  rationale — his review).
- **Close note (2026-09-26, worker-24, commit b95d532):** metric fixed → raw part mass (bytes/4), usage-field fallback kept (all pins updated; gate 340/340 + 74/74 + 17/17 + 459+1w + F=0). The live fork-test acceptance stays OPEN.
- **Close note (2026-09-27, worker_Q3S_245K_slow, commit dcad3d1):** metric v2: S-diff provider-true primary (S = input+output+cache.read per assistant; window mass = S[last assistant in window] − S[last assistant before window]), bytes/4 part mass kept as the fallback (no assistant in window / diff ≤ 0) — all pins machine-recomputed + updated (smoke 76/76 + 17/17, probe 340/340 count unchanged, 459+1w, F=0). The live fork-test acceptance stays OPEN.
- **Status (2026-09-27, direct session ses_f20d1b39…): LIVE TESTS COMPLETE (maintainer, 2026-09-26/27, `maintainer/draft/compaction_guide/compaction_tests.md`): the body `keep.tokens` is NOT honored by the installed host — retention always tracks opencode.json `compaction.keep.tokens` (now 40000); `keep.messages` ignored; measured floor = 25.8-25.9k (system + summary). Metric v2 (S-diff, dcad3d1) stands as the computed advisory (the COMPACT line `tok=`). FOLLOW-UP RESEARCH APPROVED (his ruling 2026-09-27 direct: "if there is a way to set keeptoken flexibly, it should be used" — compact_memory must keep control over WHAT to keep, not just WHEN; a fixed global 40k is "too much in most cases, too little in some"): map the installed host's summarize path for a per-call keep input — the dev-branch source has `preserve_recent_tokens ?? clamp(0.25*usable, 2k..15k)` + `tail_turns`, but host-map §compaction does NOT trace where `preserve_recent_tokens` is read from (config vs per-call body — the gap); his fork-test matrix is the decider (undocumented body field variants vs the config value → measured post-compaction retention; the body `keep.tokens` we send is measured-IGNORED on this build — config wins twice: 44865 vs 30000/44878, 45005 vs 1 → floor). If a per-call field exists → wire the computed keepTokens into it; if none → the config `compaction.keep.tokens` (his file) is the only control, the computed value stays advisory (the COMPACT line `tok=`). The N=10 discriminator + the summarize-scope probe (fork session) ride this research.**
- **Close note (2026-09-27, plan27, planner-27, planner-direct): FOLLOW-UP RESEARCH DONE — installed host = 1.18.32 (measured: npm platform package `opencode-windows-x64`; the host-map "1.18.31" reading is stale) and the scratchpad source tree `opencode-dev` IS that build (provenance spot-verified vs the published v1.18.32 tag). NO per-call keep input exists: `SummarizePayload = {providerID, modelID, auto?}` (opencode-dev `groups/session.ts` L65-69), the handler passes only model + auto (`handlers/session.ts` L273-294), the budget reads CONFIG only (`session/compaction.ts` L115-120). The legacy mapping `compaction.keep.tokens` → `preserve_recent_tokens` (+ `buffer` → `reserved`) in `config/v2-compat.ts` normalizeCompaction (L163-184) explains the config-wins live series. The budget is consumed whole-turn under an estimated-token metric (`select` L223-269); the floor (no tail) = system + summary only — matches the measured 25.8k. **Fallback ruling confirmed: the config `compaction.keep.tokens` is the ONLY retention control; the computed keepTokens stays advisory (the COMPACT `tok=`).** No code change (the body `keep.*` fields are dead but harmless — server ignores extras, live-verified). RESIDUAL (maintainer, live, on 1.18.32): the N=10 discriminator (count-vs-budget retention semantics) + the summarize-scope probe — protocol in `knowledge/opencode-plugins/2026-09-27_summarize-keep-source-trace.md`; the 2026-09-26 fork observation (full 30-message tail retained despite a 30k budget) contradicts 1.18.32's `select()` — version drift 1.18.31→1.18.32 is the candidate explanation the N=10 test resolves.**

## #100. (LANDED 2026-09-26, plan18, worker-18 `worker_Q3S_245K_slow`, commit bc374b2; direct session 2026-09-25; maintainer GO 2026-09-25) remove the numword escape channel — bit-drift solved backend-side, the escape's use-case is gone
- **Problem / evidence:** the escape sentinel (`[<incident>:<safe-form>:esc]` in
  write/edit content → the digits, the `kind=escape` channel) existed to
  protect dense values from model bit-drift. Bit-drift no longer occurs
  (maintainer, direct session 2026-09-25: "the bitdrift problematic is
  solved as of now"). It fulfills no usage right now and makes fuzzy-channel
  development more cumbersome (every write/edit change must respect the
  escape path).
- **Desired outcome:** the escape channel is gone (code + pins) while the
  fuzzy track's positive parts stay fully intact: the pair channel
  (R1/R2 — right-side numwords → digits), the shared numword map
  (`agent/scripts/numword/numwords.json`), the dense/numword
  observation logging, the R6/R7/R8 channels. The Unit-2 (#97) noteCache /
  after-hook note delivery STAYS (the R8 redirect notes ride it).
- **Scope (agreed, direct session 2026-09-25):**
  - core: `ESCAPE_RE`, `EscapeHit`, `resolveEscapeSafe`, `resolveEscapes`
    (`intercept_observer_core.ts` ~L445-506)
  - plugin: `runEscapeContent` + the `onToolBefore` wiring + the
    `pre-escape=` journal extension (`appendJournal`'s escapeForms param)
    + the `kind=escape` log lines
  - tests: the escape pins (intercept smoke: ~24 escape grep hits incl.
    13a-13d; probe: ~45 grep hits) — removed; one 13-style note-delivery
    pin repurposed to the R8 redirect note (the note mechanism stays)
  - knowledge: the primer `agent/research/fuzzy-numword/primer.md`
    → new `agent/knowledge/fuzzy-numword/primer.md` (escape
    section annotated removed 2026-09-25) + the subfolder README (≤20
    lines) in the SAME commit; NOT into AGENTS.md (his ruling)
  - DO-NOT-touch: the pair channel, the map, R1/R2/R6/R7/R8,
    auto_resume / compact_memory / context_recovery / block_transfer
    plugins, FST code, `maintainer/**`
- **Acceptance criteria:** standard gate green; zero escape code paths
  (grep-clean for `ESCAPE_RE` / `runEscapeContent` / `kind=escape`); the
  pair/fuzzy pins unchanged; the note mechanism still delivers the R8
  redirect notes (smoke pin); the primer present in knowledge/ with its
  README; `numwords.json` untouched.
- **Interaction:** #95 sub-item (3) R3 is UNAFFECTED (still blocked on his
  GO — a separate unit).
- **Status:** LANDED (2026-09-26, plan18, worker-18
  `worker_Q3S_245K_slow`, commit bc374b2): escape channel removed
  (core + plugin + journal extension + pins); grep-clean; smoke 64/64,
  probe 297/297 (baseline 303 − the 6 S24 pins), pytest 459+1w, ruff F=0;
  the 13-style note pins repurposed to the R8 redirect note (mechanism
  stays covered); primer moved to `knowledge/fuzzy-numword/` (+ README,
  escape section annotated removed); numwords.json untouched.

## #102. (LIVE ACCEPTANCE COMPLETE 2026-10-21, planner-12 sweep — the 2026-09-26 maintainer live report + re-verified on the new model: intercept.log 2026-10-21_21-01 + 22-56 carry `kind=redirect tool=bash arg=command orig=/tmp/bt_probe.cjs value=C:/Users/Wasiejen/AppData/Local/Temp/opencode/bt_probe.cjs | path | pair-resolved`; LANDED 2026-09-26, plan20, worker-20 `worker_Q3S_245K_slow`, commit be07ce6) R8 redirect extension: map the POSIX `/tmp` (and `/var/tmp`) root into the sandbox — unmappable forms STOP the loop
- **Problem / evidence:** maintainer live report (2026-09-26, autorun session):
  "worker keep trying to access the temp/tmp folder directly and are
  stopping the loop repeatedly." Log evidence: `intercept.log` line 4668
  (2026-09-26_07-55, S2 worker `ses_f23d1afbaffeUSeabbt0aArOdj`) — the
  Windows-Root form (`C:\Users\Wasiejen\AppData\Local\Temp\bt_smoke_out.txt`)
  WAS redirected 1:1 (`kind=redirect ... value=C:/Users/Wasiejen/AppData/
  Local/Temp/opencode/...`) — that case WORKS since #97. The POSIX `/tmp/...`
  form (Git-Bash habit; S1 worker friction entry 2026-09-26: "/tmp under
  Git-Bash resolves outside the sandbox") has NO 1:1 mapping onto an allowed
  root -> R8 fails closed -> the permission gate STOPS the session until the
  maintainer intervenes. Worker prompt bullet added (scratchpad discipline,
  2026-09-26) as the immediate mitigation; this unit is the structural fix.
- **Desired outcome:** POSIX `/tmp/<rest>` (and `/var/tmp/<rest>`) map 1:1
  onto the scratchpad root `C:\Users\Wasiejen\AppData\Local\Temp\opencode/<rest>`
  — redirected + logged (`kind=redirect` line) BEFORE the call runs, for ALL
  tools incl. the bash `command` string; no new out-of-sandbox access is ever
  granted (redirect only, never an allow-widening); unmappable forms still
  fail closed.
- **Acceptance criteria:** a controlled `/tmp/x` write+read (bash + a typed
  tool arg) is redirected, the file lands in the sandbox, and the
  `kind=redirect` line carries orig=/value=; a no-mapping path still fails
  closed; probe pins per the established pattern; standard gate green.
- **Suggested scope:** `.opencode/plugin/intercept_observer_core.ts`
  (the redirect resolver / root mapping), `.opencode/plugin/
  intercept_observer.ts` (allowed-root resolution from opencode.jsonc),
  `.opencode/plugin/probes/handover_probe.mjs`, the intercept smoke.
- **Status:** LANDED (2026-09-26, worker-20 `worker_Q3S_245K_slow`, commit
  be07ce6): the POSIX temp-root prefix mapping (core `resolveRedirect` — a
  CODE constant, root-list-independent: `/tmp/<rest>` + `/var/tmp/<rest>` →
  scratchpad root, full remainder kept; bare root → the root itself) + the
  BASH `command`-string redirect (same resolver over the mapped POSIX spans;
  unmapped spans fail-closed, the note stays) + pins (smoke 64→68, probe
  297→305 — the new S29 section, checks 297-304). Measured: smoke 68/68,
  probe 305/305, pytest 459+1w, ruff F=0. **LIVE ACCEPTED 2026-09-26
  (plan22, worker-22 ses_f21bb91c):** both forms redirected 1:1 — bash
  `kind=redirect … orig=/tmp/plan22_acc.txt value=C:/…/Temp/opencode/
  plan22_acc.txt` + the write-tool form (`orig=/tmp/plan22_acc2.txt`); both
  files landed in the scratchpad with correct content (entry complete).

## #101. (LANDED 2026-09-26, explorer ses_f24bf71beffekwRYMtnlrWy5UG, commit 1081e52; GO 2026-09-25 direct) opencode host map — one-time explorer task: map the installed host internals so planner/worker LOOK UP instead of re-deriving
- **Problem / evidence:** every task re-pays the derivation cost of opencode host facts (SDK v1/v2 shape, plugin hook surface, session/message/part DB schema, permission system, the compaction/summarize path) — knowledge is gathered per task, not accumulated as a map (maintainer observation 2026-09-25: "we derive the same knowledge often again and again").
- **Desired outcome:** a durable, dated host map in the knowledge base of the relevant installed opencode internals (with file/line locators); upstream (web) research delegated to the explorer (cost in its context, finding → knowledge base); the planner reads installed types for VERIFICATION only.
- **Scope (suggested):** explorer task → new `agent/knowledge/opencode-plugins/host-map.md`: SDK surface (v1/v2 endpoints from the installed .d.ts), plugin hook registration + ordering, session/message/part DB schema, permission / external_directory mechanics, the compaction/summarize path, tool registration; every entry dated + locator.
- **Acceptance criteria:** the map covers the areas above with locators verified against the installed build; the knowledge folder/README rules are followed; planner spot-check: one host question answered from the map alone.
- **Status:** DONE (explorer 2026-09-26, ses_f24bf71beffekwRYMtnlrWy5UG) — the map is at `agent/knowledge/opencode-plugins/host-map.md` (all 6 areas, dated + located; 6 open/unverified items listed there).

## #103. (LANDED 2026-09-27, plan27 part 2; was: maintainer instruction in direct fork session ses_f20b3bf14ffedWHp2HGajHmGLN) fix the ctx gauge readout semantics — it lags the live context by one generated turn
- **Problem / evidence:** the ctx gauge reads `input + cache.read` of the LAST FINISHED assistant step — verified 2026-09-26 by exact matches (61,298 = 78 in + 61,220 cr of the last finished row; 32,507 = the summarizer row). Because it excludes `output`, the readout sits one generated turn below the live context → the documented "≈2 tool-call lag" (AGENTS.md §Context budget, role prompts, compaction guide) is STRUCTURAL, not measurement noise. Full field semantics: `agent/knowledge/knowledge_inbox.md` entry 2026-09-27 (S-continuity, fork-robustness, gauge semantics).
- **Desired outcome:** the gauge readout ≈ the live context within one tool call (e.g. read `input + output + cache.read` of the last finished step, or the in-flight step's recorded input when available), OR the lag note in all docs is replaced by the verified exact semantics if the maintainer rules the current behavior acceptable.
- **Acceptance criteria:** after one known small turn, gauge readout differs from the last finished step's S (input+output+cache.read, DB-verifiable) by < 1k tokens; the lag wording in AGENTS.md / role prompts / compaction guide updated to the measured semantics; gauge smoke re-pinned.
- **Scope (suggested):** FIRST locate the gauge source — it is NOT in the repo (`.opencode/**/ctx_gauge*` glob empty; check the live `opencode.jsonc` tool registration + the referenced file, maintainer domain if host-side); then the one-metric change + doc updates + smoke pin.
- **Status:** LANDED (2026-09-27, plan27 part 2, planner-27 planner-direct — GO per his in-session ruling; the approval-boundary ruling pre-approves agent-usage-facing improvements): the gauge core now reads **ctx = TOTAL (in+out+cr)** of the last FINISHED step (was total−output — the structural 2-turn lag is gone; the readout now lags the live context by ONE generated turn: the in-flight step's output, not yet in the DB) — `gauge.mjs` gaugeFromRaw + header; probe S6/S7/S12 re-pinned (FX_OK readout CTX=12345 (4%) REM=243655; ses_fx_old CTX=12345 (10%) REM=107655) + S8/S9 fixture rows rebuilt to the new semantics (total = the intended readout value); gauge_core smoke re-pinned (R_OK ctx 12345); probe 345/345 + gauge_core + ctx_gauge 3/3 + io 77/77; lag wording updated in the role prompts + repo_custom_tools + the ctx_gauge tool description (AGENTS.md + the compaction guide are his files — noted for his paste, NOT edited). Hash in the plan27 bookkeeping commit.

## #104. (LANDED 2026-09-27, plan27 part 2; was: worker-25 plan25 — doc/code discrepancy found while landing the bt anchor fix) S31 drift-guard equivalence note is stale in the anchor-trim dimension: the block_transfer tool's `matchAnchorLines` now trims leading whitespace from the ANCHOR (plan25 b), while the plugin core's `matchAnchorPrefixLines` still uses the anchor as typed
- **Problem / evidence:** plan25 fix (b) changed `.opencode/tools/block_transfer.ts` `matchAnchorLines` (leading spaces/tabs on the anchor stripped; an all-whitespace anchor matches nothing), but the S31 drift guard (probe check 318, handover_probe.mjs ~L7244-7262) asserts EQUIVALENCE with `.opencode/plugin/intercept_observer_core.ts` `matchAnchorPrefixLines`, and its comment claims both implement "the S1 unified rule, 0d85a8c" — the comment is now wrong in the anchor-trim dimension. The guard's fixtures use only no-leading-whitespace anchors ("Dup", "h1"), so the probe stays green (344/344) and would NOT catch the two matchers re-diverging in this dimension. The core's own pin (check 317: `m("  beta") === []`) still pins the core's as-typed behavior — the core was deliberately left unchanged (out of plan25 scope; an R3-gate behavior change is a maintainer call).
- **Desired outcome:** either (1) extend the S31 check-318 fixtures with a leading-whitespace anchor and pin the INTENDED relationship (equivalent after a matching core-side trim, or intentionally different with the comment corrected), or (2) correct the S31 comment to state the equivalence holds except in the anchor-trim dimension.
- **Acceptance criteria:** check 318's fixtures distinguish the anchor-trim behavior (a fixture where a leading-whitespace anchor resolves differently in the two matchers), or the comment states the exact intended relationship; probe green.
- **Scope (suggested):** `.opencode/plugin/probes/handover_probe.mjs` (S31, checks 317/318); `.opencode/plugin/intercept_observer_core.ts` ONLY if the maintainer rules the core should match the tool's new rule (observable behavior change — approval needed); `.opencode/tools/block_transfer.ts` (doc-only if the comment route is chosen).
- **Status:** LANDED (2026-09-27, plan27 part 2, planner-27 planner-direct — GO per his in-session ruling: "a removal of friction (like the mismatch in #104)"; the new approval-boundary ruling pre-approves agent-usage-facing improvements): option (a) taken — the core `matchAnchorPrefixLines` now MIRRORS the tool's plan25-b anchor-side trim EXACTLY (an anchor typed with the line's indentation matches; an all-whitespace anchor matches nothing) — the S31 equivalence is restored in the trim dimension; probe check 317 re-pinned (`"  beta"` → [2,3,5] + all-whitespace anchor → []) and check 318 gains the `"  Dup"` drift fixture (pre-fix it would have diverged: tool [2,3] vs core []); headers updated; probe 345/345 + intercept_observer smoke 77/77 (hash in the plan27 bookkeeping commit).

## #105. (open, 2026-09-27, maintainer direction in direct session ses_f1d198b41 — next maintenance pass) research bundle: scan the existing ideas.md + 3 research targets (keepTokens fork effort, v2 migration hardening, compaction-summary customization)
- **Problem / evidence:** maintainer direction 2026-09-27 (direct session ses_f1d198b41): "on the next maintenance pass the existing ideas.md can be scanned and ideas, research, feedback and if applicable proposals created on it." His `maintainer/ideas/ideas.md` carries ~15 dated ideas, several researchable (incl. the keepTokens fork + v2 questions of 2026-09-27_20-08 and the explorer-makeup of 2026-09-26_07-50). His new compaction understanding (same session): the not-kept history is DROPPED (not compressed) — the compaction model only creates the summary — so bit-rot is NOT the limiting factor; the risks are a stale/diluted summary (dragging points addressed long ago) and the optional post-compaction read being skipped (the behavior-changing case).
- **Desired outcome:** (a) the ideas.md scan is triaged — each entry → idea (`agent_ideas.md`), research task, feedback, or proposal where applicable; (b) the keepTokens fork effort is estimated (adapting the installed opencode to respect the keepTokens setting — via the repo copy in temp); (c) v2: the current "main" branch is identified (nearly 2000 branches) and a hardening plan for our plugins/app for a potential switch is researched (incl. whether v2's summarize already has keepTokens functionality not based solely on opencode.jsonc); (d) compaction-summary customization: using the exact summarize prompt (`maintainer/draft/compaction_guide/compaction_prompt.md` + the temp opencode:dev copy), find how the compaction model's summary behavior can be influenced — e.g. drop all content older than 4 compactions, mark new additions with the current compaction iteration (keeps the summary current + trackable); (e) **explorer/researcher makeover** (his GO 2026-09-27; ideas.md 2026-09-26_07-50 + 2026-09-23_05-19): rework the explorer prompt as a researcher / information finder / map creator (APIs, online research, idea mining) — the maintainer will ALSO tighten the explorer's write access in opencode.jsonc (his domain; precaution so online sources cannot influence destructive changes), reuse his existing researcher roles in `agent/prompts/addition/` (actionable_researcher, knowledge_researcher, prompt_engineer), and NOTE the explorer prompt is a worker-prompt + explorer-prompt COMBINATION — the basic worker guidelines are always present and must not be repeated in the rework.
- **Acceptance criteria:** a research doc per target (`agent/research/` or per-research folders); the ideas.md scan triage recorded in the summary/NAP with actionable items placed (agent_ideas.md / TODO / proposals); a recommendation (effort phase + estimate) per target; the stale-summary question (d) answers whether the summarize call accepts any instruction/prompt override (measured from the dev copy).
- **Scope (suggested):** `maintainer/ideas/ideas.md` (READ-ONLY), the temp opencode copies (the opencode:dev copy + `C:\Users\Wasiejen\AppData\Local\Temp\opencode\opencode-auto-resume-master` — paths in the host-map knowledge), `maintainer/draft/compaction_guide/` (READ-ONLY), `.opencode/plugin/compact_memory.ts` (the summarize call surface), and for (e) `agent/prompts/` (the explorer + worker prompt files + the `roles/` folder — read the current combination structure first).
- **Status:** open — parts (a)+(b)+(c)+(d) DONE (see per-part notes below); remaining: (e) explorer/researcher makeover.
  **Part (a) DONE (plan28, 2026-09-28, planner-direct):** the ideas.md scan
  is triaged — 4 ideas submitted to `agent_ideas.md` (ctx-gauge
  config-derived window + session/role attribution; the context-erase/
  tail-trim tool; a `submit` memory channel; the messages.updated
  real-time gauge), 4 new TODO entries filed (#109-#112), 2 items are
  maintainer-domain backend (NAP note only), the rest are already covered
  (full triage in the plan28 summary + the loop folder).
   **Part (b) DONE (plan28, 2026-09-28, explorer-28):** the keepTokens fork
    effort is estimated — research doc
    `agent/research/2026-09-28_keeptokens-fork-effort.md`: change set
    ~11 lines / 6 files (all line refs grep-verified vs the 1.18.32 dev tree),
    phased estimate ~3-5 h wall, plugin-side work = 0 (the body field is already
    sent), recommendation = fork (b) now, scoped Phase 1-3; wait-for-upstream is
    undecidable from the pinned tree.
    **Part (d) DONE (plan30, 2026-09-28, explorer-30):** research doc
    `agent/research/2026-09-28_compaction-summary-customization.md`
    the summarize call takes NO prompt override in the body
    or via env var; YES via config `agent.compaction.prompt`/`system` (the
    summarizer's system prompt) — and the plugin hooks
    `experimental.session.compacting` (context append / full prompt replace),
    `experimental.chat.messages.transform`, `experimental.chat.system.transform`,
    `experimental.text.complete` give plugin-side control of both maintainer
    ideas (drop >4-compaction-old content; tag additions with the iteration)
    without a fork — recommendation: phased plugin-side (soft context
    injection first, ~0.5-1 day; hard prompt-replace / text-complete
    stamping only if needed).
    **Part (c) DONE (plan30, 2026-09-28, explorer-30):** research doc
    `agent/research/2026-09-28_v2-migration-hardening.md`
    mainline = default branch `dev` (no `main` branch), latest
    tag v2.0.18 (2026-09-25, v2 line moving ~1 release/day; v1.18.32 the last
    v1 tag); npm `latest` is STILL 1.18.32 (v2 only on the npm `dev` tag); the
    branch count is not measurable from the fetched pages (active branches
    listed, incl. a `v2` branch) — the "nearly 2000" figure is unverified;
    v2 summarize has NO per-call keep (SummarizePayload unchanged on dev;
    ConfigV2.Compaction unchanged; the v1→v2 config shim still ships; the v2
    `/api/compact` payload is the one open gap — follow-up specified);
    hardening table per plugin/tool: hooks interface UNCHANGED on dev, all our
    client endpoints present, risks concentrated in event payload shapes,
    the gauge's DB schema, and the v1-format opencode.jsonc fields
    (permission/provider/TUI items) — recommendation: do NOT switch now
    (re-check each v2 minor; switch when v2 lands on npm latest or adds
    per-call keep). Remaining: (e) explorer/researcher makeover
    (his opencode.jsonc write-access tightening pending — his domain).

## #106. (LIVE ACCEPTANCE COMPLETE 2026-10-21, planner-12 sweep — hint lines verified live on BOTH models: `hint rejected reason=no-anchor-line` (2026-10-15_22-57 + 2026-10-21_21-55) + `hint rejected reason=d-too-high best-d=7 / best-d=798` in intercept.log; the APPLIED side observed in-session (this session's edit returned `fuzzy-edit applied orig=... len=123 d=1`); LANDED 2026-09-28, plan31 unit 2, worker-31; pre-approved class — agent-usage friction removal) edit-fuzzy hint line should state APPLIED vs REJECTED
- **Problem / evidence:** agent_feedback 2026-09-26_12-15 (planner): a NAP
  edit returned `hint reason=d-too-high best-d=24` yet the edit WAS applied
  (the file carried the new text) — the hint reads like a rejection, so an
  agent may uselessly retry/verify; the R6/edit-fuzzy hint channel does not
  state whether the fuzzy resolution was APPLIED (and at what d) or
  REJECTED.
- **Desired outcome:** the edit-hint / edit-fuzzy feedback line distinguishes
  APPLIED (carrying the resolved d) from REJECTED (carrying the best
  candidate d) — one unambiguous token per outcome.
- **Acceptance criteria:** the line format carries the outcome token;
  affected smoke/probe pins re-pinned (the hint lines are byte-pinned);
  standard gate green.
- **Suggested scope:** `.opencode/plugin/intercept_observer_core.ts` (the
  hint construction) + `intercept_observer.ts` (the edit-fuzzy feedback
  line), `.opencode/plugin/tests/`, `.opencode/plugin/probes/handover_probe.mjs`.
- **Status:** LANDED (worker-31, 2026-09-28, plan31 unit 2 — spec ea730f9):
  the mutating edit-fuzzy evidence now reads
  `fuzzy-edit applied orig=<t40> len=<n> d=<0|1> value=<t40>` and is
  STORED + DELIVERED on the (successful) tool result (the delivered
  edit-hint line is no longer failure-only); every fail-closed / ambiguous
  hint evidence now reads `hint rejected …` (existing fields unchanged);
  the outcome token makes every delivered edit-hint line self-describing;
  NO verdict-table change; `locateContent` / `resolveEditOldString`
  untouched. Pins re-pinned: intercept_observer smoke (10e/10f/10g/10h +
  11a-11f) + ONE new check (10h2 — the mutating edit-fuzzy delivers the
  `fuzzy-edit applied` line on the successful result) — smoke 77 → 78;
  probe S20-203 + S26 271-276 + S27 277-284 re-pinned (check count
  unchanged — the header tally stays 346). Measured: intercept_observer
  smoke 78/78; probe 346 checks = 335 PASS + the 11 pre-existing
  environmental numword-python failures 136-146 (#113, not regressions);
  all other smokes unchanged (auto_resume 140/140, compact_memory 78/78,
  context_recovery 17/17, block_transfer 131/131 + 64/64, gauge_core,
  ctx_gauge 3/3, loop_log 69/69, submit 23/23); ruff F=0; pytest
  UNRUNNABLE (#113 — not attempted). Hash recorded in the planner's
  follow-up bookkeeping commit (a worker commit cannot carry its own hash).

## 107. (closed 2026-09-28, plan35 maintenance pass — LANDED 2026-09-28 plan29 planner-direct; full text in todo_records.md) — the DUMP-OK / DUMP-RETRY / DUMP-FAIL ctx.log lines carry the FULL repo-relative dump path (`.opencode/archive/sessions/` + name, shared `DUMP_ARCHIVE_REL` constant — a live acceptance is a single `ls`); probe S14/S25 + the compact_memory smoke's two DUMP-OK pins re-pinned

## 108. (closed 2026-09-28, plan35 maintenance pass — LANDED 2026-09-28 plan29 planner-direct; full text in todo_records.md) — the #96 (c) trim-restore fixture's 10 pairs are now built programmatically (the `L(i)` charCode loop) + a PAIR-COUNT assert (a dropped pair fails instantly with the count); auto_resume smoke 140/140

## 109. (CLOSED 2026-10-21, planner-12 sweep — LIVE-ACCEPTED 2026-10-01: the two `loop/autorun-2026-10-01_03-27/loop_log.md` lines 40 + 46 (`-WARNING auto_resume ... silent limit-stop: finish=length ... no-COMPACT`, 13-38 + 18-43 local); the header annotation was stale — plan10 had already triaged it closed; LANDED 2026-09-28, plan33 unit 1, worker-33; was: 2026-09-28, plan28 ideas.md scan; research → design) silent context-limit stops carry NO signal — a worker dying at the wall without self-compaction returns an empty result with no log line
- **Problem / evidence:** ideas.md 2026-09-25_18-01 (maintainer) + the
  submit feedback in that entry: a worker session that stops at the context
  wall WITHOUT triggering a self-compaction returns only an EMPTY Task
  result (no error, no Work State, no loop-log line) — recovery required
  full file forensics (git log + status + in-progress handover). MEASURED
  twice: the 2026-09-25_18-01 episode + MEM-0104/0109 (limit deaths where
  the signal is a failure MESSAGE — the silent variant is worse: not even
  that).
- **Desired outcome:** a SILENT limit-stop becomes visible — either a
  plugin-side detection (the auto_resume tick or a hook checks the DB/gauge
  for a session whose last step is a limit stop — `reason=length` /
  `tokens.total` at the window — with NO ctx.log COMPACT line → a
  `-WARNING` loop-log line + optionally a compaction dispatch) or a
  result-channel stop reason. Design the cheapest reliable detector first.
- **Acceptance criteria:** a research note with the chosen detector design
  (where it hooks, what it reads, the false-positive risk) + a small build
  spec (or a maintainer call if the design touches observable behavior).
- **Suggested scope:** `.opencode/plugin/auto_resume.ts` (the tick) or
  `.opencode/plugin/compact_memory.ts` (the dispatch path),
  `agent/scripts/db/` (the step-meta read), the research doc in
  `agent/research/`.
- **Status:** RESEARCH DONE (plan32 unit 2, 2026-09-28, explorer-32 —
  doc `agent/research/2026-09-28_silent-limit-stop-detector.md`):
  the detector = a zero-IO `limitStopCheck()` leg in the auto_resume 5s
  tick — signature = last assistant finish `length` + `tokens.total` ≥
  0.99 window + 60 s silence + idle + no NEW COMPACT line since the death
  step + in-scope (role-agent prefix `planner/worker/explorer` OR
  scope≠none — KEY FINDING: Task-tool workers carry `scope= none`, so a
  scope-only gate would never fire); visibility = a `-WARNING` line
  appended directly to the looprun's `loop_log.md` (the loop_log tool is
  agent-facing; the plugin writes the file); build spec sketch in doc §5
  (6 smoke pins, ~90-130 lines, ~0.5-1 day; detector + -WARNING =
  pre-approved agent-usage class; a compaction DISPATCH on detection =
  maintainer call).
- **Status:** LANDED (plan33 unit 1, 2026-09-28, worker-33): the
  detector is built per the research doc — the zero-IO `limitStopCheck()`
  leg in the auto_resume tick (5 new `Watch` fields; the 5-clause
  signature verbatim from doc §2; leg order = AFTER `tailCompactRearm`,
  BEFORE the routing loop, own try/catch); fire = ONE `-WARNING` line per
  episode appended to the current looprun's `loop_log.md` (the plugin
  writes the file itself — `auto_resume` in the role slot, the shared
  `currentLoopFolder()` resolution — no folder → `auto_resume.log`
  only) + ONE `limit-stop= sid=… total=…` line in `auto_resume.log`; NO
  action on the session (no dispatch/send/resume — that part stays a
  maintainer call). Verification: auto_resume smoke 140 → 146 (6 pins;
  the 60 s silence gate tested with a global `Date.now` warp — no real
  60 s waits); every other smoke at baseline UNCHANGED (block_transfer
  131/131 + 64/64, intercept_observer 78/78 — the spec DoD's "77" is
  stale, the baseline was already 78 since #106 `7a9e291`, compact_memory
  78/78, context_recovery 17/17, submit 23/23); probe 346 = 335 pass +
  11 ENVIRONMENTAL #113 failures 136-146 (`No Python at python312` — the
  venv NOT fixed, per spec: MAINTAINER CALL); ruff F=0; the pytest half
  of   the standard gate is BLOCKED by #113 (exit 103, reported in the
  handover, not fixed). (Commit hashes recorded in
  the planner's follow-up bookkeeping commit — no self-reference.)
  **LIVE-ACCEPTED (plan10, 2026-10-04, verified from files):** two live
  firings in the post-build process — `limit-stop= sid=ses_f0999d65…
  total=230144` (auto_resume.log 2026-10-01T11:38:17Z) + `limit-stop=
  sid=ses_f07be53e2… total=170240` (16:43:52Z), each followed by its
  `-WARNING auto_resume … silent limit-stop: finish=length … no-COMPACT`
  line in `loop/autorun-2026-10-01_03-27/loop_log.md` (13:38 / 18:43 local)
  — the detector + the loop-log visibility channel both fired as
  designed. Entry complete.

## 110. (closed 2026-09-28, plan35 maintenance pass — LANDED 2026-09-28 plan32 unit 1, worker-32, 3a5de39; full text in todo_records.md) — per-plugin / per-tool README files for `.opencode/plugin/` + `.opencode/tools/` (9 READMEs, each ≤ ~60 lines, pointer-only, hook names spot-checked against the code) + the two folder-README index lines

## 111. (closed 2026-09-28, plan35 maintenance pass — LANDED 2026-09-28 plan29 planner-direct; full text in todo_records.md) — doc/prompt friction batch: the worker-prompt closing-commit worked example (the hash NEVER in the worker's commit) + the `node --check` probe-syntax line in repo_testgate.md + the intercept-smoke module-state-flip header note

## 112. (closed 2026-09-28, plan35 maintenance pass — LANDED 2026-09-28 plan32 unit 1, worker-32, f406a20; full text in todo_records.md) — the compact auto-resume units explainer `knowledge/opencode-plugins/2026-09-28_auto-resume-units-explainer.md` (units 1-4, the 14 log tokens, pointer-only; the compaction-handout inclusion = the maintainer's paste)

## #114. (closed 2026-09-29, first fresh session in the new workspace root ses_f130ae200ffeDRuhAMKmw0N1qE — the final no-total data point landed: a fresh session's first injected ctx line read `CTX=notAvailable | 5 compactions left`; root-cause history + full entry in todo_records.md)

## #115. (LANDED, 2026-09-28, plan34 idle-lane — agent_ideas.md triage, idea 1; pre-approved class — bookkeeping reduction; worker-34 landed — the code commit's hash is recorded in the planner's follow-up bookkeeping commit) the gauge window is config-first: root opencode.jsonc `limit.context` beats the name marker (parseWindow is the fallback)
- **Problem / evidence:** `gauge.mjs` `parseWindow` (L242-250) derives the
  context window ONLY from a trailing `<N>K`/`<N>M` marker in the model id —
  a model rename (or a backend limit move without a marker update) → window
  UNKNOWN → the readout loses `(pct%)/REM` on BOTH surfaces (the injected
  ctx: line via ctx_watchdog + the ctx_gauge tool — the shared
  `gaugeFromRaw`). The declarative source of truth already exists: the live
  `opencode.jsonc` declares `provider.<pid>.models.<mid>.limit.context` per
  model (e.g. "Qwen3.8-27B-Q3S-245K-slow" → 245000, L111-114). The auto_resume
  saturation path already resolves limits via `client.provider.list()`
  (`getModelLimits`, auto_resume.ts L478) — only the gauge surfaces use the
  name parse. Origin: agent_ideas.md 2026-09-28_01-22 idea (1) (from the
  maintainer's ideas.md 2026-09-23_04-53 + 2026-09-22_17-41 — "derive the
  context limit from opencode.jsonc instead of the model-ID name, so the
  model can be renamed freely without syncing name→limit").
- **Desired outcome:** a model rename no longer degrades the gauge readout —
  the window resolves config-first (the root opencode.jsonc `limit.context`),
  with the existing name-marker parse as fallback.
- **Acceptance criteria:** the config-first chain pinned in the gauge_core
  smoke (config hit beats the name marker; no-marker model + config →
  config value; not-in-config / missing config / no provider prefix → the
  parseWindow fallback; a JSONC (comments) config parses, incl. a `//`
  inside a string literal); the `formatGauge` readout format UNCHANGED (the
  existing fixture bytes identical); standard gate green (probe self-
  annotated total + the other smokes at baseline + ruff F=0; the pytest
  half is BLOCKED by #113 — reported, not fixed); the dated knowledge
  one-liner appended (knowledge_plugins.md).
- **Suggested scope:** `.opencode/plugin/scripts/gauge.mjs` (the new
  `resolveWindow` + the per-call config read + the `setConfigFileForTest`
  hook + the string-aware JSONC strip per auto_resume.ts L1098
  `parseJsonc`), `.opencode/plugin/tests/gauge_core.smoke.mjs`,
  `.opencode/plugin/tests/ctx_gauge.smoke.mjs` (re-pin only if affected),
  `.opencode/plugin/probes/handover_probe.mjs` (re-pin only if affected),
  `agent/knowledge/knowledge_plugins.md` (one-liner).
- **Status:** LANDED (plan34, 2026-09-28, worker-34
  `worker_Q3S_245K_slow`, code **60f031b** — gauge.mjs: replicated
  string-aware `parseJsonc` + exported `resolveWindow(modelId)` (config-
  first, first-`/` provider split, per-call root-config read, never-throw)
  + `setConfigFileForTest` hook; `gaugeFromRaw` switched; 7 new resolveWindow
  pins (a)-(f) in gauge_core.smoke.mjs, existing pins unchanged) —
  planner-verified: own gauge_core ALL PASS + ctx_gauge 3/3 re-runs + the
  live byte-identity readout (CTX=155274 (63%) REM=89726 — window 245000 =
  config value); probe 346 = 335 PASS + 11 #113 env fails (no re-pin needed);
  ruff F=0; pytest blocked #113. Live note: the running host process still
  predates this build — the gauge surfaces pick it up at the maintainer's
  next restart (no visible change until then: config 245000 == the current
   model's name marker).

## #116. (open, 2026-09-28, plan35 maintenance pass — agent_ideas.md triage, idea 4; research → build, pre-approved class — agent-usage) real-time gauge capture: does the `message.updated` event payload carry token content?
- **Problem / evidence:** the gauge readout lags the live context by ONE
  generated turn by construction (#103, LANDED — it reads the last
  FINISHED step's in+out+cr; the in-flight step's output is not yet in
  the DB). The maintainer's ideas.md 2026-09-22 carries the
  "real-time context gauge via the `messages.updated` event hook (the
  way the TUI does it)" idea. Known facts (host-map §2 + our own
  auto_resume firstAgent use, 2026-09-28): the installed host's `event`
  hook DOES deliver `message.updated` events live (host-map L226: the
  live event carries `properties.sessionID`). OPEN QUESTION: what does
  the `message.updated` payload carry about tokens — nothing (ids only),
  per-part deltas, or the finished step's usage — i.e. can a plugin
  derive the live context (in+out+cr of the in-flight step) WITHOUT a
  DB read?
- **Desired outcome:** a dated evidence entry answering the question —
  the captured `message.updated` payload shapes (which fields exist,
  when they populate) + a verdict: (a) a real-time readout is
  feasible plugin-side (a design note for the gauge surfaces: which
  field feeds the ctx line, the update cadence, the DB-read fallback
  when the event is silent), or (b) it is not (the one-turn lag stands,
  the docs keep the #103 wording).
- **Acceptance criteria:** ≥ 3 captured `message.updated` payloads
  around a busy tool-call step (raw, in the research/knowledge folder);
  the dated evidence entry with the verdict; if (a): a small build spec
  (or a maintainer call if it changes the injected ctx line format —
  that surface is maintainer-visible). NO behavior change in this
  research step.
- **Suggested scope:** a small capture plugin (`.opencode/plugin/` —
  event hook, append-only writes to a temp log, no mutation),
  `agent/knowledge/opencode-plugins/` (the dated evidence
  entry), the gauge docs if the verdict is (a). BLOCKED on the
  maintainer's host restart + plugin registration (his domain) — the
  capture plugin only lives in the live process after his restart.
- **Status:** open — filed from the plan28/34 ideas queue (idea 4);
  the host-map facts above are the starting point, the payload shapes
  are the unknown.
- **Status note (2026-09-28, plan38, source-side research, planner-inline):
  the payload shapes are now KNOWN from the installed-build source
  (opencode-dev = 1.18.32, provenance per the #99 close note):
  `message.updated` carries `properties.info` = the FULL AssistantMessage
  (tokens {input, output, reasoning, cache.read/write} + cost + finish),
  emitted by `Session.updateMessage` (session.ts:629-633) on the
  step-finish path (processor.ts:452-470 — tokens REPLACED per step, cost
  accumulated) + finalization (processor.ts:609-610); the CLI's own run
  consumer corroborates (session-data.ts:825-850 reads info.tokens/cost).
  Verdict leans (a): a real-time gauge readout is feasible plugin-side
  (last `message.updated` per session → in+out+cr, the #103 formula, no DB
  read). Dated evidence entry: knowledge/opencode-plugins/
  2026-09-28_message_updated_payload.md. The LIVE capture (3+ raw payloads
  around a busy step — the acceptance) still requires the maintainer's
  restart +   capture-plugin registration (his domain).

## #117. (LANDED 2026-09-29, worker ses_f12b5172dffeq1r2iwK4QLlWxj, final 7c5d623; direct session — ruling "per project folder is enough") reorganize the agent-repo bookkeeping into per-project folders (projects/<name>/)
- **Problem / evidence:** post-carve, the agent repo will hold
  per-project bookkeeping flat (root TODO.md / todo_inbox.md /
  todo_records.md, `agent/handover/`, repo docs,
  knowledge/research) — with multi-project use ahead ("other repos
  will be added and worked on in the future"), bookkeeping must be
  organized per project; additionally every .md under
  `agent/` auto-registers as an opencode agent type
  (measured ~90 phantom Task-tool agent entries in the live agent
  list).
- **Desired outcome:** a per-project bookkeeping layout in the agent
  repo (`projects/Free-Snap-Tap/`: TODO.md, todo_inbox.md,
  todo_records.md, handover/, the repo doc set; the shared agent
  prompts stay under `agent/prompts/`) and the Task-tool
  agent list reduced to the real role prompts.
- **Acceptance:** the layout is in place; ALL path references updated
  in one pass (role prompts, the AGENTS.md copy, `tools/submit.ts`,
  `tools/loop_log.ts`, repo docs, probe/smoke geometry where touched);
  gate green; the moved .md folders no longer appear as Task-tool
  agent types.
- **Suggested scope:** the agent repo (post-carve),
  `agent/{handover,knowledge,research,memory,scripts}`,
  `.opencode/tools/submit.ts`, `.opencode/tools/loop_log.ts`,
  AGENTS.md (copy protocol), the repo docs.
- **Status:** LANDED 2026-09-29 (worker session ses_f12b5172dffeq1r2iwK4QLlWxj — final commit 7c5d623): the five folders (agent/archive/loop/maintainer/proposals) at root with R-rename history kept, all active code/docs re-pointed in one pass, `projects/Free-Snap-Tap/` built (TODO.md, todo_records.md, repo/), staged replacements at root — his apply + the prompts reorg followed (b4adfd1/529bdb5/7b41b5d, he did #118 himself). Gate: probe 335/346 (11 = #113 env, no venv) + 10/10 smokes + summarize smoke green; grep gate clean (hits = allow-list + maintainer domain + live config only); planner re-ran the full gate after his post-landing reorg/link-fix (green) and re-pinned the 10 filename-form smoke pins his full-path replace missed (87a0487). Follow-up: #118 (internal prompts restructure, post-move) — CLOSED, done by the maintainer.

## #118. (closed 2026-09-29 — DONE by the maintainer himself: 529bdb5 prompts reorg (agent_readme_* → agent/readme/readme_*, repo docs → agent/readme/, roles/ → prompts/addition/, prompt_agent_* → agent_*, skill/ in place) + 7b41b5d repo-wide link fix + b4adfd1 staged-config apply; he kept the readme_ prefix — deviation from the requested prefix drop) internal restructure of the prompts/ tree post-reorg: move the agent_readme_* files into an `agent/readme/` folder and drop the prefix (e.g. `agent/readme/loop.md`); re-home `prompts/skill/` (dynamically loaded — not a prompt component like agent/role prompts); restructure the `prompts/agents/` layout; decide the placement of the repo content (repo docs)
- **Problem / evidence:** maintainer's message 2026-09-29 (direct,
  ses_f130ae200ffeDRuhAMKmw0N1qE): "I only contemplate to move the
  readme/readme_* files into an extra readme folder and remove the
  agent_readme_ prefix ... same with prompt/skill folder (dynamically
  loaded — not a part of a prompt like agent prompts or role prompts).
  generally will restructure the prompts/agents folder a bit after
  movement. unsure of the best placement for repo content though."
- **Desired outcome:** a tidier prompts/ tree with stable paths; all
  `{file:}` pointers in opencode.jsonc + prompt self-references +
  AGENTS.md updated in one pass (same staging protocol as #117).
- **Acceptance:** new tree in place; all references updated; agents load
  the correct prompts after a restart (live check); no phantom-agent
  growth (the .md files stay outside the `.opencode/` agent-registration
  namespace — unaffected post-reorg).
- **Suggested scope:** `agent/prompts/**` (readme/, skill/, agents/,
  repo/), opencode.jsonc (staged replacement), AGENTS.md (copy), active
  prompt self-references.
- **Status:** CLOSED 2026-09-29 — done by the maintainer (see title); the planner's layout-proposal step became unnecessary.

## #119. (LIVE ACCEPTANCE COMPLETE 2026-10-21, planner-12 sweep — `.opencode/temp/compact_budget.json` today still carries the non-session keys (`autoCompact`, `nudge1-4`, `saturationThreshold`, `keepTokens`...) alongside the session counters (key survival verified live on the new build); LANDED 2026-09-29, direct session ses_f11b625d3ffeio02fDzjzypbN2, planner-direct; found mid-#117 worker run) compact_memory's budget increment rewrites `.opencode/temp/compact_budget.json` as a v2 sessions-only file, destroying all other keys
- **Problem / evidence:** the #117 worker's self-compaction rewrote the file: the committed v1 config (autoCompact, saturationThreshold, outputReserve, keepTokens, keepMessages, emergencyRecovery, emergency_budget, model_budget map) was replaced by a v2 `{version:2, sessions:{…}}` file (measured via git diff; the rewrite landed 2026-09-29 14:05Z). Consequence: the worker's model cap silently fell to the DEFAULT 1 mid-run (its next compaction attempt was refused: "model_budget default (cap 1), used 1/1") and the planner session's injected budget line dropped 5 → 1.
- **Desired outcome:** the budget increment is read-modify-write — all pre-existing keys survive; only the `sessions` map changes (additively).
- **Acceptance:** a compaction event leaves a `compact_budget.json` diff with ONLY the sessions block changed; a compact_memory smoke check asserts non-session-key survival across an increment on a pre-seeded v1-style config (model_budget + emergency_budget present).
- **Suggested scope:** the budget-increment write path in `.opencode/plugin/compact_memory.ts` (+ `compaction_core.ts` if shared), `.opencode/plugin/tests/compact_memory.smoke.mjs` (survival pin).
- **Status:** LANDED 2026-09-29 (direct session ses_f11b625d3ffeio02fDzjzypbN2,
  planner-direct — pre-approved class): `readBudget` in
  `.opencode/plugin/compaction_core.ts` now returns the parsed object AS-IS
  (every pre-existing top-level key survives — the file is shared with the
  compaction config) and only normalizes the `sessions` map (absent / wrong-
  typed → fresh empty map); the `version` bump to 2 still lands on the first
  write (recordSuccess). Root cause confirmed: the committed config file had
  NO `sessions` block → the old lenient read fell through to the fresh v2
  object → the first increment rewrote the whole file. Pin: compact_memory
  smoke +2 (80/80) — the config-only file (autoCompact / saturationThreshold /
  outputReserve / keepTokens / keepMessages / emergencyRecovery /
  emergency_budget / model_budget, no sessions block) survives an increment
  with ONLY the sessions block changed + the version bump. Gate: probe = 335
  PASS + 11 #113 env, all 10 smokes green. Live note: the running host process
  carries the old code until the maintainer's next restart (the next real
  compaction under the new build shows the config keys surviving).

## #120. (open, 2026-09-29, planner ses_f110d8065ffeCVoCfNeTcba0wU; proposal `proposals/approved/2026-09-28_context-trim-tool.md` — his "1. + 2. are approved" + round-2 "go :-)") context_trim tool: `report` (read-only window map) + `tail` (validated tail_start_id rewrite) — NO turns mode
- **Problem / evidence:** a live session cannot drop middle context without
  compaction (the host only drops the head, summary-retaining); the research
  doc `agent/research/2026-09-28_context-erase-tail-trim.md` proved the tail
  lever (the host rewrites `tail_start_id` in place, compaction.ts:461-466;
  the window guard message-v2.ts:568; the DB is re-read per step).
- **Desired outcome:** an agent-facing tool with two modes — `report`
  (effective window state + per-message rows with tool targets + a trim
  dry-run) and `tail` (a fail-closed-validated `tail_start_id` rewrite,
  floor 6); Unit 2 follow-up = the plugin-side post-compaction tail-set
  (zero fork).
- **Acceptance:** fixture-DB smoke ALL PASS (valid cases + the 5 rejection
  cases); a new probe section (baseline 346 → 346+N, the 11 #113 env-fails
  excluded); the standard gate green (other smokes unchanged, ruff F=0,
  pytest per #113 status); the registration snippet in the handover
  (opencode.jsonc = the maintainer's domain); the live DB NEVER written
  (smokes = fixture only).
- **Suggested scope:** `.opencode/tools/context_trim.ts` (new),
  `.opencode/plugin/tests/context_trim.smoke.mjs` (new),
  `.opencode/plugin/probes/handover_probe.mjs` (new section), this entry.
- **Status:** LANDED (2026-09-30, worker-39 ses_f10a06089ffeVKusSGiE20Hjw9) —
  code commits `2d4480e` (tool + fixture smoke 20/20) + `48aab9f` (probe S33,
  checks 346-351). Gate: probe 352 = 341 PASS + the 11 known #113 env-fails
  (136-146); all 11 smokes green (context_trim 20/20). Registration snippet
  registration snippet in the handover (opencode.jsonc = the maintainer's
  domain). Live acceptance COMPLETE (2026-10-21, planner-12 sweep; per the
  2026-10-01 U1 line + #126 close): report + tail + the floor refusal live
  + the Unit 2 `tail-set=` lines live 2026-10-04T20:50-20:54Z (8 sessions).
  maintainer's restart + registration; `tail` = a throwaway session, the
  (The final hash rides the planner's follow-up bookkeeping commit.)
- **Status (Unit 2, 2026-10-01, worker-Q3S-slow ses_f0a1d6a5affeFgN9YsdlKV7LNr):** LANDED —
  code commits `ad7db12` (the `tailSetKeep` core export — the keep-count
  boundary form of `tailSet`, the exact write path reused — +
  context_trim smoke 26 -> 29) + `f625a65` (the auto-resume tail-set
  leg — `tailSetLeg` after `tailCompactRearm`, before `limitStopCheck`;
  every NEW ctx.log COMPACT line's `keep=<N>m` (watched or not) drives
  `tailSetKeep` + one `tail-set=` log line; the stale line-format
  comment fixed; the factory `dbPath` option (test-only lever) +
  auto_resume smoke 152 -> 155) + `9bdcfc9` (probe S34, checks 352-353).
  Gate: probe 354/354 (fully green — the 11 #113 env-fails are gone since
  the plan40 VENV_PY re-point); all 11 smokes green (context_trim 29/29,
  auto_resume 155/155, compact_memory 89/89, the rest at baseline); ruff
  F=0; pytest 466+1w (the planner-verified 2026-10-01 baseline — this
  task touches no FST code). The live opencode.db is NEVER written by
  the smokes (fixture only — the factory's dbPath option). Live
  acceptance (NOT a DoD gate): the next REAL host compaction shows a
  `tail-set=` line in auto_resume.log (the tail rewritten to the
  agent-requested keep boundary — natural occurrence; the host process
  must postdate the build for the leg to run — the maintainer's
  restart). **LIVE-ACCEPTED (plan10, 2026-10-04, verified from files):**
  the post-restart process (2026-10-04) processed the existing ctx.log
  COMPACT lines at init and fired the tail-set leg for 8 sessions —
  auto_resume.log 2026-10-04T20:50:22Z–20:54:07Z `tail-set= sid=… msg_…
  -> msg_… keep=<N>` lines (incl. keep=18 for ses_f0a4c9ad…, keep=12 for
  ses_f0999d65…; one session twice, keep=12 then keep=8 — the two
  COMPACT lines for it). The leg runs live; the entry is COMPLETE.


## #126. (closed 2026-10-01, plan1 maintenance pass — #120 U1 context_trim FULLY LIVE-ACCEPTED: the spawn-sqlite3 fallback + the json_set tail fix + the async-execFile fix (execFileSync once-per-process) all live-verified post-restart (report ×2 + floor-6 refusal in one process); the live-debug saga (TAB-arg collapse → \x01, 1 MB → 16 MB, double-quote argv mangling → json_set) in full text in todo_records.md) context_trim's in-process-only sqlite backend fails in the live host process

## #127. (open, 2026-09-30, direct session ses_f0e129deeffeqmM5rc8mpnTY2Q — maintainer ruling on `proposals/approved/2026-09-30_compact-message-delivery.md` item 4: "in favor"; pre-approved class — agent-usage) restart-branch inheritance: the closing session's queued compact-message appended to the successor's restartText
- **Problem / evidence:** a queued `compact_message_<sid>` stranded by an
  `action:` close after compaction (root cause B, research doc
  `agent/research/2026-09-30_compact-message-delivery.md`) is lost to the
  restart-spawned successor — the restart branch never reads it. Items 1-3
  LANDED (`f96a39c` STEP-0 protocol / `b025159` zombie guard / `0a89e7c` age
  sweep); item 4 was the open ruling.
- **Ruling + design:** the maintainer is in favor (his 2026-09-30 comment —
  "more truthful"). Planner's interference analysis (settled): NO blocking
  problem with the planner instructions — the appended text is a labeled
  INTENT HINT (the successor still rebuilds reality from committed state
  first — a stale intent gets corrected by the file check; the size is
  bounded to one queued message). Read `.opencode/temp/compact_message_<sid>`
  at the restart branch (`auto_resume.ts` L1409 call site / `restartText`
  L898-915); if present: append the labeled section + rename the file
  `.consumed` (the item-3 tombstone); fail-open (absent / unreadable → the
  base text unchanged, no throw).
- **Acceptance:** 3 new pins in `auto_resume.smoke.mjs` (file present → the
  labeled content in the spawn text + the file renamed `.consumed`; absent →
  the base text byte-identical; unreadable → the base text); the existing
  147/147 stay green; the standard gate (probe 352/352 + all 11 smokes); live
  acceptance = the next restart-branch spawn with a queued file present
  (natural occurrence).
- **Suggested scope:** `.opencode/plugin/auto_resume.ts` (the `restartText`
  region + the L1409 call site), `.opencode/plugin/tests/auto_resume.smoke.mjs`.
- **Status:** LANDED 2026-10-01 (worker_Q3S ses_f0ad0bbfcffeCKb4FmU2gK7c29 —
  spec `agent/handover/specs/2026-09-30_compact-message-unit4.md`): the
  restart branch appends the closing session's queued `compact_message_<sid>`
  as a labeled INTENT HINT section (`restartIntentSection` in
  `auto_resume.ts` — fail-open read, `.consumed` tombstone rename,
  `intent=` log line; appended at the spawn call site, `restartText` stays
  pure). Smoke 152/152 (5 new pins — the 3 required scenarios, scenario
  (a) split into section + tombstone + log-line asserts; the absent case
  byte-identical to the base). Standard gate green: probe 352/352 + all
  11 smokes (code unit 5ed36cf). RESIDUE: live acceptance = the next
  restart-branch spawn with a queued file present (natural occurrence —
  the `intent= sid=` log line in `auto_resume.log` is the detector).
  (The final hash rides the planner's follow-up bookkeeping commit.)

## #128. (LANDED, 2026-09-30, direct session ses_f0e129deeffeqmM5rc8mpnTY2Q — maintainer ruling on `proposals/approved/2026-09-30_compact-memory-cross-override.md`: option (b), cross spend ≤ model_budget + emergency_budget, "the same as when self-compacting"; pre-approved class — agent-usage) compact_memory caller-scoped compaction override (a cross caller gets effective cap+1)
- **Problem / evidence:** a worker dead at the wall at `count == cap` is
  resolvable only by a cross call that BURNS its one self-rescue (the
  emergency arg); at `count == cap+1` it is refused outright — no planner
  rescue path (`compact_memory.ts:422-440`).
- **Design (ruling-adjusted — the proposal's original Part 1 is RETRACTED):**
  `isCross` (explicit id ≠ the caller) → `effCap = cap + 1` (the SELF path
  unchanged); the gate allows `count < effCap`, else the existing emergency
  branch (`count == cap && emergency`). Cross at `count == cap+1` stays
  REFUSED — the cross spendable total = cap+1 = the same as self (no second
  slot). Part 3 (the `ovr` audit token) INCLUDED (his live comment
  2026-09-30: "part 3 can be implemented as well" — a successful cross
  dispatch at pre-dispatch `count == cap` gains the `ovr` token on the
  COMPACT line). Part 2 docs:
  the tool description + the `compaction_core.ts` budget header (worker
  part); the planner-prompt line + the AGENTS.md staged copy = the planner's
  bookkeeping (workers have no edit access to `agent/prompts/**`).
  VERIFIED-FACT CORRECTION (the proposal's "CPU cap 0 denied for all" line
  is inaccurate): at cap 0 the current gate ALLOWS one EMERGENCY self
  compact (`count 0 === cap 0 && emergency`, L425) — under "the same as
  self" a cap-0 cross gets exactly 1 as well; no special CPU denial is
  added (flagged to the maintainer in the direct session).
- **Acceptance:** the adjusted pin set in the `compact_memory.smoke.mjs`
  Gate-M block (~L422-500, fixture cap 2): cross@cap no-emergency →
  dispatched; cross@cap+1 → refused (names the override consumed);
  self@cap no-emergency → refused (regression, wording unchanged);
  self@cap+emergency → dispatched (regression); self@cap+1 → refused
  (regression); the cap-0 corner pinned; the old cross@cap-refusal pins
  re-pinned; 82/82 total; the standard gate (probe 352/352 + all 11 smokes);
  live acceptance = the next real cross-override dispatch (maintainer
  domain).
- **Suggested scope:** `.opencode/plugin/compact_memory.ts` (the gate
  L410-440 + the tool description), `.opencode/plugin/compaction_core.ts`
  (the budget header comment), `.opencode/plugin/tests/compact_memory.smoke.mjs`.
- **Status:** spec committed
  `agent/handover/specs/2026-09-30_compact-memory-cross-override.md`; queue
  order after #127. **LANDED 2026-10-01** (worker session
  ses_f0abc032dffenb1K8yrfXKOg5V): gate (`isCross`/`effCap` — the emergency
  branch fires only for SELF) + Part-3 `ovr` token + Part-2 worker-part docs
   (tool description + `compaction_core.ts` header); smoke 82 → 89 (net +7:
   the override pins (a)/(b)/(c) + Part-3 line pins, self regressions (d)/(e)/(f),
   cap-0 corner ×2, the IQ4 cap-3 override pin; the old cross-shape pins
   re-pinned — incl. the CPU plain-call pins → SELF shape); probe
  352/352 (S13 92/93/95/222/223/225/226 + S32 340 re-pinned). PENDING
  PLANNER-SIDE (done 2026-10-01, plan2 bookkeeping): the AGENTS.md
   Compaction Guidelines "Budgets" block (staged copy at repo root
   `AGENTS_pending_2026-10-01.md`) + the planner-prompt budget line
   (`agent/prompts/agent_planner.md` — the CROSS-dispatch bullet).
   **AGENTS.md Budgets paste LANDED by the maintainer (commit 3296339,
   2026-10-04 — the cross-override bullet now in the live AGENTS.md
   Budgets block).** Live acceptance = the next real cross-override
   dispatch (maintainer domain) — still PENDING.

## #133. (LANDED 2026-10-01, plan8 planner-direct — the 4 new-hire-test frictions of the context_trim description; pre-approved class — agent-usage friction) context_trim new-hire test: 4 description frictions found + fixed (the report rejection surface + 3 missing description lines)
- **Problem / evidence:** the plan8 new-hire test (worker_Q3S_slow
  ses_f09aca202ffe7Dn08BNLLCObnQ — battery 6/6 PASS from the tool's
  description alone, zero source reads; findings
  `loop/autorun-2026-10-01_03-27/plan8_newhire_context_trim.md`) found 4
  frictions: (F1) the report dry-run on a no-marker session degenerates to
  a bare rejection with no counts — the shared fail-closed validation was
  undocumented; (F2) the report rejection `error: session-not-found` made
  the host inject a spurious SYSTEM-Warning retry nudge (the tail mode
  returns the SAME cause as a plain `tail= rejected:` line — no warning);
  (F3) the report rows include the CURRENT in-flight message with
  `tokens=0` (uncommitted, not zero-mass) — undocumented; (F4) the row
  format is under-specified (epoch-ms `time`, the `bytes4=` fallback label,
  the conditional offset/limit display).
- **Status:** LANDED (plan8, planner-direct, inline): the report rejection
  now returns the plain two-line `session <id>\nreport= rejected:
  session-not-found` (no error surface — the genuine `db-error:` / `error:`
  lines stay for real backend failures) + the 4 description lines (the
  uniform rejection-surface note, the in-flight-message note, the
  row-field note, the dry-run-shared-validation note); the smoke
  exact-string check re-pinned. Gate green: probe 355/355 + all 11 smokes
  (context_trim 29/29). LIVE acceptance (natural occurrence, post-restart —
  the live host process predates the change): the next report call on a
  dead session shows the `report= rejected:` line WITHOUT the host
  SYSTEM-Warning retry nudge (the detector: the warning line's absence).

## #132. (LANDED 2026-10-01, plan6 worker_Q3S_slow — the byte-exact readout marker ` (unlisted)` on the silent built-in cap-1 fallback; gate probe 355/355 + all 11 smokes + ruff F=0; filed in the plan5 maintenance pass — agent_feedback 2026-09-30_16-57; pre-approved class — agent-usage friction) the silent default-cap-1 budget fallback is invisible: an unlisted model row falls back to cap 1 with no marker
- **Problem / evidence:** agent_feedback 2026-09-30_16-57 (planner): after the maintainer's model key swap, the row model was no longer listed in `model_budget` → the silent default-cap-1 fallback (compaction_core.ts ~L205-211 — the file has no `default` key) → the injected ctx suffix read `1 compactions left` while the self-gauge read 5; ~4 tool calls to diagnose.
- **Desired outcome:** the default-cap fallback is visible — a marker on the injected budget suffix (and/or a logged line) whenever the cap resolves to the silent default (not an explicit model_budget entry), and/or a documented `default` key convention in `compact_budget.json`.
- **Acceptance criteria:** a fixture with an unlisted model → the marker appears on the injected suffix / logged line; a compact_memory smoke pin for the marker; standard gate green.
- **Suggested scope:** `.opencode/plugin/compaction_core.ts` (the budget read), `.opencode/plugin/tests/compact_memory.smoke.mjs`, the budget-file header comment.
- **Status:** LANDED 2026-10-01 (plan6, worker_Q3S_slow — the byte-exact marker ` (unlisted)` appended after `left` ONLY on the silent built-in cap-1 fallback branch (model unlisted/empty + no finite `model_budget.default` key; the CPU branch excluded); gauge.mjs `compactionsLeftSuffix` tracks the branch; 2 new gauge_core pins + probe S6b 28.8 (S6b 8, total 355/355) + 3 doc one-liners (ctx_gauge.ts description, compaction_core.ts header comment, ctx_watchdog.md). Standard gate green: probe 355/355 + all 11 smokes + ruff F=0. Code commits `748f7cd` (gauge.mjs + gauge_core pins) / `bbd8705` (probe 28.8 + tally) / `e3ae1c1` (doc one-liners) / `85fbf7a` (close-down) — planner-verified from files (own re-run: probe 355/355 + gauge_core ALL PASS). Live acceptance = natural occurrence (the next gauge readout of an unlisted model shows the marker).)

## #135. (OPEN, 2026-10-01, planner-9 ses_f0999d65dffeFxiJylRzecRbhu — direct→autorun; approved design `proposals/approved/2026-10-01_context-budget-package.md`, his commit 4b4c34e) the context budget package build — U0 keep-N surface (`keepMessages` on context_trim tail) / U1 recommendation leg (cut-down report + 3 anchored options + biggest-droppable list) / U2 per-message context filter AS A FORK CHANGE (non-destructive `data`-blob field, fork assembly skip, kill switch `.opencode/temp/filter_off`, his `build_fork.bat` = live gate, effort L) / U3 marker compaction cap backstop (agent summary + quartet write, full protocol, one budget compaction) / U4 nudge policy (`nudge1-4` keys in compact_budget.json + recommendation one-liner; the prompt-ladder reduction is the planner's direct edit)
- **Desired outcome:** the package units LANDED in order U0→U1→U2→U3→U4, each leaving the repo green; U2 live after the maintainer's bat run.
- **Acceptance criteria:** per-unit DoD in the spec wave `agent/handover/specs/2026-10-01_context-package-README.md` (each spec names its measured end states + the standard gate).
- **Suggested scope:** per the specs — `.opencode/tools/context_trim.ts` (U0/U1/U2 surface), the fork `C:\Users\Wasiejen\Repos\opencode` (U2), `.opencode/tools/compact_memory.ts` (U3), `.opencode/plugin/auto_resume.ts` (U4), `agent/prompts/**` (U4 planner edit).
- **Status:** OPEN — spec wave COMPLETE (2c1b053); **U0 LANDED** (worker_Q3S 0538aa4 — smoke 34/34, gate green, planner-verified under node); **U1 LANDED** (worker_Q3S 9f1f668 — smoke 40/40, gate green under node; surface report = cut-down form by default, full form via core export; probe-346 arg-list constraint → todo_inbox flag); **U3 LANDED** (worker_Q3S 5c1e286/40e008f — compact_memory 110/110, gate green under node; worker limit-death after final commit — handover reconstructed from files by the planner, probe 86 re-pinned by the planner); **U4 LANDED** (worker_Q3S 4e2b901 — auto_resume 165/165, gate green under node) + the U4 prompt-ladder reduction done by the planner (planner/worker prompts + AGENTS_pending_2026-10-01.md note; the `nudge1-4` keys ENABLED in the live config by the maintainer (commit 3296339, 2026-10-04: nudge1_triage 0.80 / nudge2_stop 0.90 / nudge3_emergency 0.925 / nudge4_maintainer 0.95 — the U4 live acceptance is now reachable on this process); **U2 RESCOPIED 2026-10-15** (direct ses_ebfa5be88ffegbOOhAztizTIJD: bun REPAIRED — 1.4.3 via `npm i -g bun` + `allow-scripts=bun`; but OpenCode 2.0 is officially released — the upstream `dev` branch folded into the 2.0 line, the final v1 = 1.19.0 on branch `v1`, `production` = the official v2 branch, the installed live host = 1.18.34 = fork tag aec0b9a6d8; the old U2 target `packages/opencode/src/session/message-v2.ts` is GONE from the 2.0 line and V1 plugin implementations do not run in V2 — the maintainer's ruling: abandon U2-on-v1 as dead work; U2 now awaits the v2 RESEARCH — what changed, the plugin-porting impact (6 plugins + 8 tools), and the filter design on v2; research scope proposed, delegation OK pending); worker preference per the maintainer's inbox ruling (2026-10-01, done/): prefer worker_Q3S_slow (no model-switch reprefill above ~150k planner context + the 245k window for lookup-heavy tasks); U2 live gate = the maintainer's `build_fork.bat` run.

- **Status note (2026-10-21, planner-12 sweep, new-model host):** U4 partial live data on the restarted process — the `arm=` leg + the `scope= autorun|none|planner` verdict lines fire live (auto_resume.log 2026-10-21T18:55-21:16Z, incl. the Direct-session suppression `scope= none`); no `nudge=` line yet on this process (no watched autorun session has crossed 0.80 since the restart) — the nudge firing leg stays PENDING a natural crossing. U2 unchanged (v2 research lane; the fork `build_fork.bat` gate = maintainer domain).
## #134. (open, 2026-10-01, planner-9 ses_f0999d65dffeFxiJylRzecRbhu — direct maintainer engagement in the autorun session: "write access with users/users should have been blocked via intercept plugin"; maintainer call — a behavior change) the out-of-sandbox guard for host write/edit is note-only: an unmappable path passes through the intercept layer AND (observed) the host permission gate, and only the host filesystem stopped it by chance
- **Problem / evidence:** plan9 (this session): a typo'd write to `C:\Users\Users\Wasiejen\Repos\opencode-agent\loop\...` (doubled `Users`). Verified chain: (1) the intercept plugin's `tool.execute.before` hook FIRED and logged the call — `.opencode/temp/intercept.log` 09:48 line, verdict `out-of-sandbox`, `path=C:\Users\Users\... root=C:\Users\Wasiejen\...`; (2) the R8 typed-path redirect covers the host write/edit/read `filePath` fields (intercept_observer.ts `REDIRECT_PATH_FIELDS` L574-578) but is fail-closed PASS-THROUGH on unmappable forms (`runRedirect` L584-587: `resolveRedirect → null → continue` — the field is not mutated, no block); (3) the call then reached the host's `write` tool: fork `tool/write.ts` flow = `assertExternalDirectoryEffect` (L44) → `ctx.ask({permission: "edit"})` (L54) → `fs.writeWithDirs` (L64); `assertExternalDirectoryEffect` (external-directory.ts L35) asks the `external_directory` permission, which resolves to the default `action: "ask"` for an unlisted path (fork `permission/index.ts` L28-38 `findLast(...) ?? {action: "ask"}`) — the session should have STOPPED and asked; (4) OBSERVED: it did not stop — the call proceeded to `fs.writeWithDirs` and died with `Unknown: FileSystem.makeDirectory (C:\Users\Users\Repos\opencode-agent\loop\...)` — Windows refused to create `C:\Users\Users` (no admin rights on the C:\ root). Nothing was written to disk; the save was ACCIDENTAL (the typo pointed at a non-existent root).
- **The gap:** a host write/edit to an EXISTING unlisted external path is blocked by nothing agent-side (the intercept layer is note-only for host tools; the only hard rejection is block_transfer's own core `sandboxCheck`) — it would succeed with only an after-the-fact `out-of-sandbox` log line. The #102 design premise ("unmappable forms STOP the loop" = the host permission gate stops the session) did NOT hold for this plugin-spawned session.
- **Open question (host-side, maintainer domain):** why the `external_directory` ask did not stop the call in this plugin-spawned session — candidates: permission asks auto-answered in SDK/plugin-spawned (non-interactive) sessions, or the installed host build differs from the fork checkout. Needs a live test (a write to an existing unlisted external dir in a spawned vs a direct session) or a host-permission-log inspection.
- **Desired outcome:** an out-of-sandbox host write/edit is BLOCKED (or at least reliably stopped at the permission gate), not merely logged after the fact; or a verified ruling that the gate behaves correctly and the observed pass-through was a one-off.
- **Acceptance criteria:** a probe write to an existing unlisted external path returns a block/permission error with NO write (both session types, or a verified explanation per type); AND/OR the intercept before-hook gains a hard-stop for unmappable typed-path forms on the mutating tools (write/edit) — a new verdict + smoke pin + a prompt note — if the maintainer approves the behavior change.
- **Suggested scope:** `.opencode/plugin/intercept_observer.ts` (before-hook hard-stop candidate), fork `permission/index.ts` / `tool/write.ts` / `tool/external-directory.ts` / the SDK session permission-ask flow (root cause), `opencode.jsonc` `permission.external_directory` (his config domain — an explicit `"*": "deny"`-style default could close the gap host-side with zero code).
- **Resolution (2026-10-01, same session, direct):** the open question is CLOSED — the host permission gate DID stop the request; the maintainer allowed it interactively (to observe the outcome + keep the write payload — `journal_write.log` already records it). The `FileSystem.makeDirectory` error came AFTER his allow. The #102 gate premise holds in plugin-spawned sessions.
- **Follow-up finding A (the folder-doubling detector blind spot):** the `path-anomaly` doubled-segment detector (observation d, `DOUBLED_RE` intercept_observer_core.ts L482) runs on the JSON-SERIALIZED args (`argsToString` = `JSON.stringify`, intercept_observer.ts L272-281) — the JSON-escaped `\\` (double backslash) breaks the regex's segment+separator+segment adjacency, so doubled segments in typed Windows paths NEVER fire (probe: raw `C:\Users\Users\...` matches, JSON-escaped `C:\\Users\\Users\\...` does not). The 09:48 call logged `out-of-sandbox` only — no `path-anomaly` line. (The 53 historical `path-anomaly` hits are mostly `\n\n` newline artifacts in JSON content — `doubled=n` — not real path doublings.) The detector is NOTE-ONLY by design (addendum C6: one check, no stale duplicates) — it never stops a request.
- **Follow-up finding B (the before hook CAN block):** fork `plugin/index.ts` trigger = `yield* Effect.promise(fn(input, output))` — a THROWING before hook fails the tool call (the agent sees the error message); the hook's RETURN value cannot abort, only mutate `args` (the R8 mechanism).
- **Maintainer follow-up proposal (2026-10-01, direct):** add an existence check in the before hook — if the PATH (the containing directory, NOT the specific file) does not exist, block the request; only the path, to catch the invalid-path cases. Design options to rule: (A) literal — block on missing parent directory: catches in-root dir typos (e.g. `agent/handver/file.md` — today the write tool's `writeWithDirs` silently CREATES `handver/`) but kills legitimate new-subdirectory writes (the write tool creates intermediate dirs by design); (B) hard-stop upgrade of the EXISTING out-of-sandbox note on the mutating tools' typed path fields (write/edit): the check (`underRoot`) already exists (core note + block_transfer `sandboxCheck`); catches the doubled-root / escaped-root typo class (the full target path is not under any allowed root), preserves in-root new-dir creation, no new detection logic — implemented as a throwing before hook per finding B. (C) the detector blind-spot fix (run `DOUBLED_RE` on the unescaped field, or normalize `\`→`/` first) — note-only, complements A/B.
- **Maintainer ruling (2026-10-01, direct):** build B' + C, NOT A. B' semantics per his clarification: trigger = OUTSIDE THE SANDBOX (absorbs "(doubled AND outside) OR outside") — a folder doubling in itself is NEVER wrong (it may sit in an allowed root → stays a log signal via C); "path exists" = the CONTAINING DIRECTORY (never the file); "outside the sandbox" = not under any allowed root = the R8 set (workspace + scratchpad + the `external_directory` allows in opencode.jsonc).
- **LANDED (2026-10-01, same session, direct):** B' hard-stop in the before hook (`intercept_observer.ts`): for the EFFECTIVE (post-redirect) write/edit `filePath` — outside every allowed root AND containing directory missing → the hook throws a `HardStopError` (the hook's catch distinguishes the deliberate throw: logs ONE `intercept-error` line, then RE-THROWS — a throwing before hook fails the tool call, the agent sees the error); existing external paths pass to the host permission gate; in-root new-dir writes pass to writeWithDirs; read is out of scope (read-only). C fix (`intercept_observer_core.ts` `observePathAnomaly`): collapse the JSON-escaped `\\` pairs back to one separator before `DOUBLED_RE` — doubled segments in typed Windows paths now log `path-anomaly` (the 14-verdict vocabulary is unchanged — the block is a throw, not a verdict).
- **Status:** LANDED 2026-10-01 (planner-9 direct — hook block + C fix; standard gate green: probe 355/355 + all 11 smokes; intercept_observer.smoke 85/85 incl. 7 new (4b)/C pins; 7 probe pins updated — they encoded the pre-C detector blindness (2 new path-anomaly lines / 1 line where ZERO was expected). Live acceptance = natural occurrence after the host restart (the next out-of-sandbox write to a missing path shows the block error; the next doubled typed path logs `path-anomaly`). **C-part LIVE-ACCEPTED (plan10, 2026-10-04, verified from files):** the post-restart process logged `path-anomaly` for a live typed-path call carrying adjacent doubled segments (`doubled= maintainer, proposals`, intercept.log 2026-10-04_22-54 — the JSON-escaped `\\` collapse now lets DOUBLED_RE fire on typed Windows/POSIX paths). **B'-part** (the hard-stop throw) still PENDING a natural occurrence (no out-of-sandbox write to a missing path since the restart).)

## #136. (OPEN, 2026-10-15, direct ses_ebfa5be88ffegbOOhAztizTIJD — the maintainer's v2 pivot: OpenCode 2.0 released, `dev` folded into the 2.0 line, live host 1.19.0, the v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2` on branch `migration` with standalone opencode v2.0.29) the OpenCode v2 impact research — architecture (shared background service), the v1→v2 DB (schema + migration + data-blob survival), the new compaction/assembly, the v2 plugin API (arg-mutation before-tool hook?), the per-feature overlap audit of our 6 plugins + 8 tools (v2-native / partial / absent), config carry-over for our actual files, and the upgrade path — to drive the plugin porting order (the intercept plugin FIRST, with the pair-resolution/numword channels DROPPED per the maintainer's 2026-10-15 ruling — surviving set: fuzzy resolution read d≤2 / write d≤1 + the observation log; the pair-based git-refs-in-bash channel 3c drops with the pair grammar — flagged as an assumption in the spec), the U2 per-message filter design on v2, and the live-host upgrade decision
- **Problem / evidence:** V1 plugin implementations do not run in V2 (migration guide, `maintainer/done/2026-10-15_migration_from_v1.md` L447); the old U2 fork target is gone from the 2.0 line; measured first pass: no Levenshtein/fuzzy in v2 core/tui/cli/server src (the intercept plugin's core value is absent natively); the v2 edit tool = `packages/core/src/tool/plugin/edit.ts` (exact match + Location-relative resolution + external-path approval + typographic match normalization).
- **Desired outcome:** one research document (`agent/research/2026-10-15_opencode-v2-impact.md`, 7 sections per the spec) that lets the porting plan, the U2-on-v2 design, and the upgrade decision be made from the document + this entry alone.
- **Acceptance criteria / DoD:** per the spec `agent/handover/handover_task.md` — §0 answers the arg-mutation hook question explicitly; §4 is a feature→verdict→effort→order table; every non-obvious claim file-pointered (path+line) or doc-URL'd; document committed on `opencode_test`; worker summary to the handover channel.
- **Suggested scope:** the fork `C:\Users\Wasiejen\Repos\opencode` on `production` (READ-ONLY), the official v2 docs, our `.opencode/**` + `opencode.jsonc` (READ-ONLY), the v2 clone (READ-ONLY reference).
- **Status:** LANDED 2026-10-15 (direct ses_ebfa5be88ffegbOOhAztizTIJD — explorer_Q3S_slow ses_ebf38ec2cffeXAREzUwnlEjXpo, commit f59eda0 on `main`; the explorer's first run died mid-step at ~80k of 245k (backend stall, no compaction) and was RESUMED via task_id to completion; doc `agent/research/2026-10-15_opencode-v2-impact.md` 428 lines, planner-verified spot-checks: §0 arg-mutation hook confirmed against `core/src/tool.ts:271-280` + shipped `tool-input-repair.ts` plugin; §2 `hasLegacySessions` at `v1-migration.bun.ts:1130-1135` exact). Key results: (1) the intercept port is PLUGIN-ONLY — v2 `ctx.tool.hook("execute.before")` gives a mutable `event.input`, executed by the core (v2 tools use `path`, not `filePath`); (2) v2 opens the live v1 `opencode.db` in the same location and auto-migrates IN-PLACE at first service start (one-way, v1 tables not dropped — dry-run on a DB copy first, per doc §2); (3) the U2 filter maps to plugin hook `session.context` (mutable `event.messages` before `LLM.request`); (4) overlap audit: only `session_info` native, several partial, most absent; `.opencode/tools/` is NOT a v2 discovery path — all 8 tools become plugins; (5) config carry-over table for our actual files + `update` policy (no auto-install by default). Follow-up: the maintainer's dual-support hint (one plugin package exporting both V1 `server()` and V2 `setup()`) — `maintainer/done/2026-10-15_migration_hints_from_perplexity_ai.md` — adopted as the porting strategy; the intercept port's scope cut = pair/numword channels DROPPED (the maintainer confirmed sandboxing — R8 redirect, out-of-sandbox note, B' hard-stop, journals — STAYS); next unit = #137.

## #137. (OPEN, 2026-10-15, direct ses_ebfa5be88ffegbOOhAztizTIJD; design source `agent/research/2026-10-15_opencode-v2-impact.md` §0/§4) the intercept observer ported to v2 (Unit A of the dual-support strategy) — a v2-native plugin in the v2 clone (`C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`) implementing the SURVIVING feature set: observation log (path-anomaly incl. the C doubled-segment/JSON-escape-collapse, out-of-sandbox note, intercept-error, write/edit journals), read-scoped fuzzy resolution (d≤2, strict existence gate), write-scope fuzzy resolution (edit d≤1, `write` excluded per M1), the B' hard-stop (throw on write/edit outside every allowed root with missing containing dir), and the R8 typed-path redirect — with the PAIR (`[left:right]`) + NUMWORD channels DROPPED (maintainer ruling 2026-10-15). The v1-side reduction + the dual-export merge (one file serving both hosts) is the follow-up unit on the main repo
- **Problem / evidence:** the intercept plugin does the background reliability work v2 lacks (measured: no Levenshtein/fuzzy in v2 core/tui/cli/server src); the v2 hook API supports the port plugin-only (mutable `event.input`, verified `core/src/tool.ts:271-280` + shipped `tool-input-repair.ts`); v2 tools use `path`, not `filePath`.
- **Desired outcome:** the v2 clone boots cleanly (unported v1 files inert) with the ported interceptor live: fuzzy resolution + sandboxing work in v2 exactly as in v1 (minus pair/numword), proven by the bun smoke and the maintainer's live v2 acceptance.
- **Acceptance criteria:** per the spec `agent/handover/handover_task.md` — surviving channels with v2 field names; zero pair/numword logic or lines; the B' throw + its single log line; the C detector fires on JSON-escaped doubled paths; the clone `opencode.jsonc` gains the v2 `plugins` entry; v2 smoke green under bun (or the stated typecheck + live path); live-acceptance script named in the handover; main repo + its gate untouched.
- **Suggested scope:** the clone's `.opencode/plugins/intercept_observer.ts` (+ test), the clone `opencode.jsonc` (test config), the unported-v1-file inert mechanism on `migration`; reference: main repo `.opencode/plugin/intercept_observer.ts` + `intercept_observer_core.ts` (READ-ONLY), the fork (READ-ONLY).
- **Status:** Unit A LANDED 2026-10-15/16 (direct ses_ebfa5be88ffegbOOhAztizTIJD — worker_Q3S_slow ses_ebea1bd3cffe3ktwt4MRmehe6a in the clone, final commit 9868aec on `migration` + checkpoint 5516a83; the worker hit the context wall at 230k, the queued compaction's retained tail still exceeded the cap on resume (request 235239 > 230144) — resolved via `context_trim tail keepMessages=8` + resume; unit then completed). Planner-verified: commit + clean tree from files, smoke RE-RUN by the planner = `SMOKE: ALL PASS` (46/46 checks, 19 log lines, `.opencode/plugins/tests`), tsgo exit 0 (worker-measured, not re-run). Surviving set live in v2: observation log (path-anomaly C incl. JSON-escape collapse, out-of-sandbox, intercept-error, journals), read-scoped fuzzy d≤2, write-scope fuzzy d≤1 (edit only, write excluded), B' hard-stop, R8 redirect — pair/numword absent (zero logic/lines). Unported v1 files inert under `.opencode/plugin-unported-v1/` (v2 boot stays clean, loader code-verified). Live acceptance PENDING the maintainer: `bash agent/scripts/v2_intercept_acceptance.sh` from the clone root — CAVEAT (handover + research doc §2): v2 points at the SAME global `opencode.db` and auto-migrates it IN-PLACE (one-way) at first v2 start → back up/dry-run the DB copy first (`OPENCODE_DB=opencode-dryrun.db`). Follow-up unit: the v1-side reduction + dual-export merge on the main repo (one file serving both hosts).
- **Status (unit 1 — the session_v2 model-reader port, 2026-10-22, planner-12):** LANDED in the clone (commits `4c0e607` gauge session-row read table-aware (`session_v2` first, then `session`, coexisting newest by `time_updated`; the no-total result now carries `window = resolveWindow(modelId)`); `1b7a202` the intercept model field through that reader, `unknown` fallback, 8-field byte-compat kept; `59d71d8` handover). Planner-verified: `git show --stat` matches the file list; `node .opencode/plugin/tests/gauge_core.smoke.mjs` ALL PASS (27→32, the new `v2unit1` pins incl. the coexisting-DB and NEITHER-table forms); `node .opencode/plugin/probes/handover_probe.mjs` exit 1 = the PRE-EXISTING clone gate breakage (TODO #141), unchanged by these commits. LIVE-ACCEPTED for the model-field leg (the maintainer's 2026-10-22 re-run of `agent/scripts/v2_intercept_acceptance.sh`: the 3 new intercept lines carry `qwen3.8-flash-next-coder-iq1_m` in the model slot instead of `unknown`; 15→18 lines, 3 PASS verdicts). REMAINING: the token-mass legs = unit 2 (a v2 session still reads `CTX=notAvailable` until then), then the v2 tools estate (his priority.md item 2; the clone gate #141 first).
- **Status (unit 2 — the token-mass legs, 2026-10-22, planner-15):** LANDED in the clone — `922f74c` (gauge.mjs mass legs TABLE-PAIRED: the mass rides the session-row winner — `session_v2`↔`session_message` / `session`↔`message`; spawn marker gets a v2 form via a probe exec + winner-pinned marker; v1-only DBs BYTE-IDENTICAL; probe mock relabel) + `2c1fa23` (10 new gauge_core pins — 42 PASS lines incl. v2-shaped OK mass 44150, pairing both directions, spawn-forced forms, readout `SESSION=ses_v2s_fx CTX=44150 (36%) REM=75850`, v1-only byte-identical 43210). Research answer (→ knowledge inbox): v2 per-message usage EXISTS — `session_message.data.tokens{input,output,reasoning,cache.read/write}` + the `$.time.completed` discriminator; mass = the 5-field `TokenUsage.total()` sum. Planner verification: `git show --stat` matches both clone commits; my re-runs gauge_core ALL PASS (42) + ctx_gauge 3/3 + probe 360/360; clone tree clean, main gate untouched. LIVE acceptance PENDING (maintainer): his next v2 session shows `CTX=n` (no `notAvailable`). REMAINING: #137 v2 TOOLS ESTATE (port map §4 order 3+).
- **Status (unit 3a — the v2 CONTEXT FAMILY core, 2026-10-22, planner-16):** LANDED in the clone — `1174d98` (NEW `.opencode/plugins/context_family.ts` — DEFAULT-export v2 plugin `{id:"context_family",setup}`; `ctx.session.hook("context", …)` fires per PRIMARY request (`model-request.ts:422-431`); gauge read = the SHARED v2-paired gauge core `readGauge(undefined, ownSessionID)` + `formatGauge` + the `ITER` suffix; injection = APPEND-ONLY tail entry `{role:"user",content:[{type:"text",text:"ctx: …"}]}` — cache discipline: NOTHING before the tail is touched (the v1 v2.3 ruling carried); match-only own-session gate, mismatch SILENT; evidence lines to `.opencode/temp/ctx_family.log` (kinds `ctxmsg`/`gauge`, reasons `db-error`/`messages-not-array`; the v1 `invalid-messageID` branch has NO v2 counterpart — the request draft carries no messageID) + tests `tsconfig.json` (`files` += the plugin; `paths` += `@opencode/ai`) + `typecheck.ts` pins BOTH plugin defaults) + `f149745` (NEW `context_family.smoke.mjs` — 24 pins, bun + mock ctx, scratchpad fixture DBs, live DB never read). Planner re-runs: new smoke `SMOKE: ALL PASS` (24) + tsgo 0 bytes (zero errors) + clone probe `PROBE handover: 360/360 PASS`; clone tree clean; main gate untouched. Shape answer (→ knowledge inbox): the injectable entry = `@opencode/ai` `LLM.Message` (`packages/ai/src/schema/messages.ts:237-244`, TYPE-ONLY import; a generic `Array<unknown>` param silently widens = NO pin — the negative control proved the real pin). REMAINING: 3b (the nudge ladder at `execute.after` + `ctx.session.synthetic(delivery:"queue")` + the `subagent` handover pre-flight, same file — spec `agent/handover/specs/2026-10-22_137u3b-nudge-ladder.md`), then port map §4 order 4+; LIVE v2 ctx-line acceptance PENDING (maintainer restart + backend slot).
- **Status (unit 3b — the nudge ladder + handover pre-flight, 2026-10-22, planner-16):** LANDED in the clone — `9e069cc` (`.opencode/plugins/context_family.ts` extended: `execute.after` 5-rung ladder — per-fire gauge read for the event's OWN `sessionID`, rungs VERBATIM v1 v2.6 (`1 pct>=50 | 2 pct>=70|REM<=30k | 3 pct>=80|REM<=20k | 4 pct>=90|REM<=10k | 5 REM<5k`), highest-met fires, one nudge per rung per session (in-memory), unknown-window/no-total/db-error SILENT; delivery `ctx.session.synthetic({delivery:"queue"})` FIRE-AND-FORGET (never awaited; `delivery-threw`/`delivery-rejected` evidence reasons; the tool call never fails) + `execute.before` pre-flight (tool `subagent` + `input.prompt` contains `agent/handover/handover_task.md` → missing/whitespace-empty file = ONE `handover-preflight` warn; observation-only, `input` never mutated) + smoke 24→46 pins + README; v2.7 P02 summary mirror NOT re-ported; the v2.8 busy-check/`setImmediate` race block NOT ported (native `delivery:"queue"` replaces it — deviation in the plugin header).) — ONE combined clone commit (the plugin+smoke pair is the smallest green unit: the new `setup` registers tool hooks, so a split commit would be red — the never-commit-red rule overrides the split-checkpoint guidance here). Planner re-runs: smoke 46 PASS + `SMOKE: ALL PASS`, tsgo 0 bytes, clone probe `360/360 PASS`, clone tree clean. Nudge readout = the BARE `formatGauge` line (no `ITER=` suffix — that rides only the injected ctx line). REMAINING: port map §4 order 4 (compact_memory + context_trim + U3 marker via the durable `compaction` message + `session.hook("compaction")` `result` override), then 5-8 small tools; LIVE acceptance of ctx line + nudges = maintainer (backend slot).
- **Status (unit 4a — the compact_memory + U3 v2 clone twin, 2026-10-22, planner-17):** LANDED in the clone — `afd6b26` (`.opencode/plugin/scripts/compaction_core_v2.mjs` — the v1 budget layer ported: fail-open config reader, cap resolver (CPU cap-0 guard tested FIRST, unlisted/typo → `model_budget.default` else 1), the #119 additive-rewrite invariant, `recordSuccess` increment-on-success, byte-identical COMPACT line incl. the ` emergency`/` ovr` tokens, NEW pure `budgetGate` (the emergency-1 + the #128 caller-scoped CROSS cap+1), `hasKeepKeys` audit read; the v1 `compaction_core.ts` stays UNIMPORTED by v2 code — inert-dir contract) + `ae2f36f` (`.opencode/plugins/compact_memory.ts` — the tool via `ctx.tool.transform`/`editor.add`, dispatch `ctx.session.compact` fire-and-forget, the budget increment + COMPACT line land ONLY on the `session.compaction.ended` event with a PENDING dispatch record (`.failed` → one evidence line, no burn; auto/manual compactions with no pending record are silent), U3 marker mode = `ctx.session.hook("compaction")` `result` override `{summary, metadata:{marker:true}}` (the core writes the durable `compaction` message; the v1 `markerSet` DB-write lane has NO v2 counterpart), the queue leg = native `session.prompt({delivery:"queue"})` AT verified success (the temp-file store + auto_resume relay NOT ported — order 2 skipped), deviations logged: `boundary-ignored` (v2 computes `recent` from config `compaction.keep.tokens`), `keep-not-applied`, `dump-skipped` (4c); + 41-pin bun smoke + tests tsconfig `files`) + `358cba7` (clone tests README section). Planner re-runs (clone root): `compact_memory.smoke.mjs` `SMOKE: ALL PASS` (41), `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0, probe `360/360 PASS`; `git show --stat` matches all 3 commits; clone tree clean; MAIN gate untouched (the 12 node smokes + v2 intercept 52 + context_family 46 re-run green by the worker, my spot re-runs green). REMAINING: #137 unit 4b (context_trim v2 — spec committed, launch next planner), 4c (dump twin), port map §4 orders 5-8; LIVE v2 compact/marker acceptance = maintainer (backend slot).
- **Status (unit 4b — the context_trim v2 port, 2026-10-22, planner-18):** LANDED in the clone — `471dce7` (`.opencode/plugins/context_trim_v2.ts` — the tool via `ctx.tool.transform`/`editor.add`; report over the v2 window (the `session_message` rows from the LAST COMPLETED `compaction` row onward, head row INCLUDED — no `tail_start_id` in v2, the DB-write lane GONE; the window read rides the v2 service, the gauge core stays the ONLY DB reader; masses = fork `TokenUsage.total()` sum gated on `$.time.completed`, else `bytes4=`; v1 row/`last:`/option-block/biggest-droppable forms) + the persistent per-session BOUNDARY store `.opencode/temp/context_trim_v2.json` (fail-open read, additive write) + the ONE `ctx.session.hook("context")` filter (drops draft `event.messages` entries strictly before the boundary — assembly-time only, persisted history UNTOUCHED; id-less entries survive; stale boundary → ONE deduped `boundary-stale` line + whole-window fail-open; `boundary-state=stale` in the report header; NEW `clear` arg — a JSON store needs an explicit un-set lane; rejections = plain `tail-set= rejected: <cause>` lines, floor 6 carries, keep=0 refused; the v1 FULL report form + `recommendationOneLiner` NOT ported — the CUT-DOWN form is the only v2 form) + tests tsconfig `files`) + `b5ccc0e` (`tests/context_trim_v2.smoke.mjs` — 65 pins incl. the composition pins with the REAL context_family handler in BOTH orders + the tests README section + the dry-run `verdict=` fix). Planner re-runs (clone): probe `360/360 PASS`; `context_trim_v2.smoke.mjs` `SMOKE: ALL PASS` (65 pins); the 3 node-dir smokes the worker reported exit-1 on Windows teardown `fs.rmSync` EBUSY (zero FAIL) — `gauge_core` ALL PASS, `context_trim.smoke` 40/40, `auto_resume.smoke` 170/170 — ALL green in my env → transient flake, worker's baseline repro recorded in `todo_inbox.md`; `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0; `git show --stat` matches both commits; clone tree clean. REMAINING: #137 4c (dump twin — spec to be written), port map §4 orders 5-8 (the small tools via `ctx.tool.transform`; `session_info` = native); LIVE v2 acceptance = maintainer (backend slot).
- **Status (unit 4c — the pre-compaction dump twin for v2, 2026-10-22, planner-18):** LANDED in the clone — `167a7c2` (`agent/scripts/db/dump_session_v2.mjs` — v2-table reads (`session_v2` + `session_message` pair ONLY, rejects a DB without the pair exit 2; NO part-table), v1 CLI parity (md/`--lite`/`--json`/`--out` exit-2 relpath/`--all [--slim|--full]`), read-only backend chain `bun:sqlite`→`node:sqlite`→`spawn-sqlite3` (clone `plugin/tools/sqlite3.exe`, `file:…?mode=ro` URI, 2500 ms KILL) + `DUMP_V2_BACKENDS`/`setBackends()` test hooks, md gains `# source: v2 tables=… backend=` line) + `136426b` (plugin dump leg in `plugins/compact_memory.ts` replacing the 4a `dump-skipped` sites: EVERY allowed dispatch dumps BOTH artifacts BEFORE the compact call, `compaction_dumps/<sid>_c<PRE-DISPATCH count>.{md,json}` + stamped fallback shared, v1 ctx.log shapes `DUMP-OK … ms=` / one `DUMP-RETRY=1` / `DUMP-FAIL` byte-port, best-effort never blocks, v1 `WARNING:` line on failure, NEW `compact_v2.log` `kind:"dump"` audit line, denials dump NOTHING) + `a8a61ac` (smoke 41→63 pins: 22 dump + twin-CLI pins, 4 era repins; README) + `cfbb426` (DUMP-FAIL one-line detail fix — one event = one log line; `a8a61ac`+`cfbb426` = one verified unit, `cfbb426` first full-verified tree). Planner re-runs (clone): probe `360/360 PASS` UNCHANGED (dump lane pins the v1 path — no repin); `compact_memory.smoke.mjs` `SMOKE: ALL PASS` (63); tsgo exit 0; `git show --stat` matches all 4; clone tree clean; the teardown-EBUSY trio unchanged (0 FAIL). Worker-reported discrepancies (2) routed: the 4c spec's chain-in-4b claim was WRONG (4b rides the gauge core, no chain — twin followed MAIN v1 `tools/context_trim.ts` ~115-166 instead) → feedback submitted; the real v2 table pairing `session_v2`↔`session_message` (model column JSON string) → knowledge submitted. REMAINING (#137): port map §4 orders 5-8 (the small tools via `ctx.tool.transform`; `session_info` = native); LIVE v2 acceptance = maintainer (backend slot).
- **Status (unit 5a — the small write tools (loop_log + submit), 2026-10-22, planner-19):** LANDED in the clone — `be5a278` (`.opencode/plugins/loop_log.ts` — v2 plugin `{id:"loop_log",setup}` via `ctx.tool.transform` + `editor.add`, teardown disposes; v1 Parts A-D carried: the token table, the `<stamp> <status> <role> <session> <model> <content>` line, the folder create/locate + ANOMALY note, the readback byte-compare `verified:` lines, the exact unrecognized-status Error string; the v2 deviation logged: auto-fill rides `context.sessionID`/`context.agent` only — the v1 `sessionId`/`session?.id`/`extra.model.id` legs have NO v2 carrier, the `unknown` fallback carries; repo root = setup-captured `ctx.location.directory`) + `a5da17b` (`.opencode/plugins/submit.ts` — the 5-channel map + header levels byte-identical (feedback ### / knowledge ## / todo ## / ideas ### / memory ###), the `<header> <stamp> <role> <session>` rule, the zero-param + empty-string error byte-carried, the memory role-prefix split, the 2026-10-21 FINAL-BYTE NEWLINE GUARD carried from the MAIN twin — the clone's v1 twin lacks it, TODO #143) + `tests/loop_log.smoke.mjs` (29 pins) + `tests/submit.smoke.mjs` (29 pins) + tsconfig `files` += both plugins + tests README sections. Planner re-runs (clone root): probe `360/360 PASS` (the v1 loop_log 69/69 + submit 31/31 smokes UNTOUCHED/green), both new smokes `SMOKE: ALL PASS`, `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0, `git show --stat` matches both commits, clone tree clean; MAIN gate untouched. REMAINING (#137): 5b (ctx_gauge + state_snapshot + dev helper) and 5c (block_transfer — the MAIN twin is the baseline, the clone twin is stale per #143), auto_resume v2 (port-map order 2, HELD — the loop mechanics on a v2 host = maintainer domain), LIVE v2 acceptance = maintainer (backend slot).
- **Status (unit 5b — the small read tools (ctx_gauge + state_snapshot + dev helper), 2026-10-22, planner-20):** LANDED in the clone — `8e630f1` (`.opencode/plugins/ctx_gauge.ts` — wraps the SHARED gauge core via the context_family relative-spec import `"../plugin/scripts/gauge.mjs"`; `formatGauge(readGauge(undefined, sid))`; `kind:"db-error"` → ` — <error>` appended, never a stack; the budget suffix + `(unlisted)` ride the core's `compactionsLeftSuffix` — nothing re-derived; deviation logged: v1 bare-string return → the v2 `{output}` result, NO repo-root capture needed) + `ef3ebbb` (`.opencode/plugins/state_snapshot.ts` — the 6-section capped digest byte-shape carried: `## git` execFileSync `git log --oneline -20`, `## nap` first 40 + steer, `## todo` count + LAST 15 headers, `## maintainer` inbox first-lines max 5 + priority first 10, `## proposals` root `.md` names, `## markers` the verbatim `execFileSync("bash", ["-c", MARKER_SWEEP])`, `(none)` empties, the 60-line hard cap (LAST section's content trims first, headers+steers preserved), the EXACT non-planner refusal line; READ-ONLY pinned (fixture tree byte-identical around a run); deviation: root rides the setup-captured `ctx.location.directory` — the v1 execute-time `context.directory` has NO v2 carrier) + `45bb52e` (`.opencode/plugins/dev_get_tool_context_contents.ts` — ONE plain string (the v1 2026-09-27 discipline kept); dump fields = the v2 `ToolContext` keys that EXIST (`contextKeys` + `sessionID`/`agent`/`messageID`/`id`/`progress`/`signal`, `progress` → `<function>`); NO keys invented for the v1-only legs (`callID`/`extra`/`messages`/`metadata`/`ask`/`directory`/`worktree`/`client` — absence rides the runtime `contextKeys`); non-serializable → the v1 keys-only fallback line, never a throw) + 3 new bun smokes (ctx_gauge 11 + state_snapshot 24 + dev helper 13 pins, scratchpad fixture DB pair — live DB never read) + tsconfig `files` +=3 + tests README sections. Planner re-runs: clone probe `360/360 PASS` UNCHANGED; the 3 new smokes `SMOKE: ALL PASS`; `tsgo -p .opencode/plugins/tests/tsconfig.json` exit 0; `git show --stat` matches all 3 commits; clone tree clean; the 12 v1 node smokes green, the v1 twins untouched. Worker docs finding FIXED inline (pre-approved): repo_commands.md gains the smoke-suite commands — MAIN `c1062ad` + clone `5434a93`. REMAINING (#137): 5c (block_transfer — MAIN-twin baseline per #143), auto_resume v2 (port-map order 2, HELD — v2-host loop mechanics = maintainer domain), LIVE v2 acceptance = maintainer (backend slot).
- **Status (unit 5c — block_transfer, 2026-10-22, planner-21):** LANDED in the clone — `1b0ebdb` (`.opencode/plugins/block_transfer.ts` 1276 lines — id `block_transfer`, registered via `ctx.tool.transform` + `editor.add(blockTransferTool)` (the compact_memory shape); the MAIN twin (1034 lines) carried VERBATIM by a generation script (helper region + description template literal + execute body sliced from the design-of-record — reader truncates the 9298-char description line, hand-transcription would break byte-identity); all modes/anchors/buffers/feedback formats byte-carried incl. the WRITE `old A..B -> new C..D, file now N line(s)` group, the multi-region+one-text REJECT, same-file MOVE-with-shift (above/below/INSIDE), dst-checks-before-cut, EOL/EOF hardening items 3-8+10; sandbox BOTH config shapes: on-disk v1 object form UNCHANGED + v2 native array `permissions:[{action,resource,effect}]` LAST-MATCH-WINS (fork findLast), /** stripped, cross-shape dedupe, TEMP/TMP env override ahead of `os.tmpdir()`; config NEVER edited; v2 surface deviations: JSON-schema input (refs anyOf string|integer, describes VERBATIM 12/12), `{output}` return, root = setup-captured `ctx.location.directory`; ONE flagged delta: `insertAfter` definite-assignment initializer (typecheck-only, behavior identical) + `tests/block_transfer.smoke.mjs` 90 pins + `tests/block_transfer.sandbox.smoke.mjs` 23 pins (scratchpad OUTSIDE fixture trees — the scratchpad is a live allow root) + tsconfig `files` +=1 + tests README section. Planner verification: `git show --stat` matches `1b0ebdb` (5 files, 1999 ins), registration lines grepped in-file, clone tree clean, worker DONE line in loop_log, gate green at committed tree (probe 360/360, v1 12/12 incl. stale-twin bt 132/132 + sandbox 76/76 UNTOUCHED, v2 11/11 ALL PASS, tsgo 0 — worker one-shot re-run; full gate NOT re-rerun in planner (worker-verified)). Worker findings: 1 friction (relative paths resolve against SESSION CWD — cross-repo needs absolute) + 1 knowledge (deny-matrix fixtures must sit OUTSIDE the scratchpad config root) + 1 memory (byte-exact pin discipline) — all in inboxes. REMAINING (#137): auto_resume v2 (port-map order 2, HELD — v2-host loop mechanics = maintainer domain), LIVE v2 acceptance = maintainer (backend slot). The §4 small-tools wave (4a/4b/4c/5a/5b/5c) is now COMPLETE.

- **Status (v2 migration inventory sweep, 2026-10-22, DIRECT ses_e9b246abdffesGhYjB50ymiwqy — the maintainer's status question, files-verified):** the §4 port map is COMPLETE except **order 2 (auto_resume — HELD on his v2-host loop-mechanics call)**. Ported in the clone (branch `migration`, HEAD `90d6d6c`, tree clean): intercept observer (order 1 = Unit A, `intercept_observer.ts`, LIVE-ACCEPTED at plan14 direct), the CONTEXT FAMILY as one plugin (order 3 = units 2 + 3a + 3b: gauge mass legs TABLE-PAIRED, the injected ctx line, the nudge ladder + `session.synthetic` queue + the `subagent` pre-flight; the v1 `ctx_watchdog` rung ladder absorbed there, its v1 file inert-in-`plugin-unported-v1/`), the COMPACTION family (orders 4-5 = 4a `compaction_core_v2.mjs` + `compact_memory.ts` + the U3 marker hook, 4b `context_trim_v2.ts`, 4c `dump_session_v2.mjs` + the plugin dump leg), and the SMALL-TOOLS estate (orders 6-8 = 5a `loop_log` + `submit`, 5b `ctx_gauge` + `state_snapshot` + `dev_get_tool_context_contents`, 5c `block_transfer`; `session_info` = native, NOT ported). Gate re-verified THIS session: clone probe `360/360 PASS` + the 8 v2 bun smokes `SMOKE: ALL PASS` (context_family, ctx_gauge, state_snapshot, loop_log, submit, block_transfer, context_trim_v2, compact_memory); MAIN gate untouched (probe 360/360 at 1e9ce94). Post-his-restart v1 acceptance CLOSED: the plan24/25 `state_snapshot` unit (the live digest shows the 150 cap + the doc-noise filter + LIVE-first markers). GAP outside §4: `.opencode/plugin/context_recovery.ts` (245 lines, auto-loaded in MAIN v1, emergency overflow recovery on the shared `compaction_core`) has NO §4 map row and NO v2 twin — ruled TABLED 2026-10-22 (direct planner-28): a safeguard, not a hot path; a worker never needs it (the planner can fire a cross-compaction), so only a planner would benefit — the v2 twin waits until a clear need recurs. LIVE v2 acceptance of the ported estate still PENDING (needs the v2 host running + the backend slot).

## #138. (closed 2026-10-22, plan14 — #142+#138 field-scan unit LANDED + planner-verified: main dd5b63b + clone 622e75e, gates green both trees) the R8/C path resolver FALSE-FIRES on path-looking SUBSTRINGS inside command/arg TEXT — verdict noise today, false hard-stops once #134 B' lands
- **Problem / evidence:** intercept.log 2026-10-21_23-17 (my own bash `grep -n ... '/^## #120/'`): `path=/^## root=C:\Users\Wasiejen\Repos\opencode-agent | path | out-of-sandbox`; 2026-10-21_23-04 (an edit whose `filePath` was fully in-sandbox, the literal `/tmp-redirect` inside the newString): `path=/tmp-redirect ... out-of-sandbox`; 21-53/22-17 `doubled=n | path | path-anomaly` on bash heredocs with quoted example paths. The resolver scans the ARG STRING, not the effective path field.
- **Desired outcome:** the path/fuzzy channels classify only the TOOL'S PATH FIELDS (write/edit/read `filePath`, block_transfer `srcFile`/`dstFile`) — free-text content inside a bash command or an edit payload is not a path candidate.
- **Acceptance criteria:** the three observed calls log NO `out-of-sandbox`/`path-anomaly` verdict; the genuine cases (the `/tmp/bt_probe.cjs` redirect, the clone-root out-of-sandbox line) still fire; intercept_observer smoke gains a pin per false form; standard gate green.
- **Suggested scope (non-exhaustive):** `.opencode/plugin/intercept_observer.ts` (the path-scan site + `REDIRECT_PATH_FIELDS` already separates fields — reuse it), `intercept_observer.smoke.mjs`.
- **Status:** LANDED 2026-10-22 (worker_FlashNext, ses_e9ee3329) — the (d) path-anomaly + (e) out-of-sandbox channels scan the JOINED string values of the call's TYPED path fields (the `REDIRECT_PATH_FIELDS` table; clone v2 `path` fields), never the args string; a path-looking substring inside free text (bash `command`, `content`, `oldString`/`newString`) logs NOTHING. Pins per false form (smoke (16b): in-root write/edit payload with `c:\\users\\users\\x.txt` + `/tmp-redirect` + `/^## #120/`; (16e): bash command with quoted doubled example paths) + per genuine form (the `C:\Windows` / config-root-sibling controls still fire). Repurposed pins: (2b) cap retargeted to a typed write, (12n) + probe 303/304 bash-note halves flipped (`runRedirectCommand` itself untouched), smoke (B3) retargeted to a typed write inside the allowed root — the spec's Pin audit missed (B3), probe 303, probe 304 (filed to todo_inbox, closed at plan14 curation). Gates: main smoke 93→96/96, probe 360/360, bt 150/150, bt-sandbox 76/76; clone smoke 49→52/52, clone probe 360/360. Planner stamp: main `dd5b63b`, clone `622e75e`; my re-runs — main smoke 96/96 + probe 360/360, clone `plugins/tests/smoke.mjs` 52 PASS + clone probe 360/360.

## #139. (CLOSED 2026-10-22, plan15 — DOCUMENTED leg LANDED: the knowledge entry states the artifact with both dated examples + the interpretation rule; the fix option NOT taken (documented-leg acceptance met; no behavior change shipped) — intercept.log lines can carry a session id + TIMESTAMP that do not match the CALL (flush-time labeling across a process restart)
- **Status:** CLOSED 2026-10-22 (planner-15) — `agent/knowledge/knowledge_plugins.md` entry "intercept.log lines can carry a session id + TIMESTAMP that do NOT match the CALL" carries both dated examples (2026-10-21_23-04 sid-vs-payload mismatch; 2026-10-04 plan10 line 2089) + the rule (attribute by payload, not the sid field). Fix (sid at call time) remains an alternative — decision taken: document (low severity, pre-approved class).
- **Problem / evidence:** intercept.log 2026-10-21_23-04 lines (local, = ~21:04Z UTC, BEFORE this session's first tool call at 21:15:40Z) carry `ses_e9fb4e611ffe…` (my session) while the payload is the PREVIOUS direct session's NAP edit. Plan10 logged the same class 2026-10-04 (line 2089, plan9-era content + current sid).
- **Desired outcome:** an intercept line's sid + timestamp identify the CALLER reliably, or the knowledge base states the lag as a hard interpretation rule (the log is authoritative for the CALL, NOT for who/when the line landed).
- **Acceptance criteria:** either a fix (sid resolved at call time) + smoke pin, or a verified `knowledge_plugins.md` entry stating the artifact with the two dated examples; no wrong-agent attribution in the NEXT sweep that relies on the sid field.
- **Suggested scope:** `.opencode/plugin/intercept_observer.ts` (where the sid is read + the flush/queue path), `agent/knowledge/opencode-plugins/knowledge_plugins.md`.
- **Status:** OPEN — low severity, investigation trap; decision-needed (fix vs document).

## #140. (closed 2026-10-22, plan14 — unit A (config-driven sandbox roots, main 5ce85b3 / clone 8d6bbd5) + unit B (whole-path field rule, main a46ca9b / clone bdbf663) LANDED + LIVE-ACCEPTED by planner-13 (the 3-tool cross-repo probe, no `C:\Users\*` ask) + the residual `out-of-sandbox` note fixed by #142 (main dd5b63b / clone 622e75e, smoke (16a) + clone pin (i)); the unit-A env-robustness follow-up stays in todo_inbox 2026-10-22_01-34) cross-repo file access into the v2 clone: the tool refuses or the path arrives TRUNCATED, and the permission ask shows `C:\Users\*` instead of the real path
- **Problem / evidence:** his report: the clone paths are `allow`-ed in `opencode.jsonc` yet the asks surface `C:\Users\*`, and agents conclude they typo'd `C:\Users\Wasi`. Measured this session: `.opencode/temp/intercept.log` 2026-10-21_22-56 line 3201 — my own `cd C:/Users/Wasiejen/Repos/opencode-agent-v2` logged `path=C:/Users/Wasiejen/Repos/opencode-agent-v2 root=C:\Users\Wasiejen\Repos\opencode-agent | path | out-of-sandbox`; the worker's knowledge_inbox entry (commit 1bf76a3) — `edit`/`write` reach the clone, `block_transfer` hard-refuses outside the launching CWD + Windows temp (`sandboxCheck`), and a truncated `C:\Users\Wasi` went to maintainer approval. #138 shows the resolver scans ARG TEXT, not only path fields. His hypothesis (intercept-side mangling) is the leading candidate; the truncation happens BEFORE the host ask.
- **Desired outcome:** an `allow`-ed external root works end-to-end for every file tool (block_transfer included); any ask names the FULL path; no `C:\Users\*` glob ask.
- **Acceptance criteria:** a controlled probe from the main repo — write a fixture into the clone with `write`, `edit`, `block_transfer` (COPY + PASTE) and read it back; `intercept.log` shows the full path with NO mutation line, and the host ask text (if any) carries the full path; the fix lands in BOTH trees (main + clone `plugins/intercept_observer.ts` + `tools/block_transfer.ts`); intercept_observer + block_transfer smokes gain a pin per form; the standard gate green in both.
- **Suggested scope (non-exhaustive):** `.opencode/plugin/intercept_observer.ts` (the `REDIRECT_PATH_FIELDS` scan + `runRedirect` + the path verdict), `.opencode/tools/block_transfer.ts` (`sandboxCheck`, the sandbox roots + the external-root allow), `.opencode/plugin/tests/intercept_observer.smoke.mjs`, `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs`, the clone twins.
- **Status:** RULINGS LANDED 2026-10-22 (planner-12, the maintainer's answers to the 3 close-out questions) — build approved, no decision needed: unit A = `block_transfer` sandbox roots become CONFIG-DRIVEN (the same `permission.external_directory` `allow` set the R8 resolver already reads at `.opencode/plugin/intercept_observer.ts:319-345` — the `/**`-suffix strip + dedupe + fail-open fallback already exist there, reuse the shape), fix in BOTH trees; unit B = the log-field truncation must NEVER cut inside a single-path field (raise the path-field cap over `MAX_FIELD_CHARS = 160`, `.opencode/plugin/intercept_observer_core.ts:137` + the truncate helper `:660`), fix in BOTH trees. The `C:\Users\*` ask and the self-blame loop are the truncation artifact + the init-time root snapshot, both covered. The live 3-tool probe run stays the maintainer's.
- **Unit A status: LANDED 2026-10-22 (worker-12, ses_e9f4b2f)** — main repo commit 5ce85b3: tool (config block + `CONFIG_ROOTS` load + sandboxCheck roots + description) + sandbox smoke (64->76 pins, fixture config under the scratchpad, TEMP-override isolation) + probe check 113 re-pin — green: 150/150 bt, 76/76 sandbox, 360/360 probe. Clone commit 8d6bbd5 (branch `migration`): same change transplanted onto the clone's pre-hardening form (`[cwd, ...CONFIG_ROOTS, TEMP ?? TMP]`, no dedupe/tmpdir terms — the smoke/probe mirrors match it exactly); clone gate DIFF vs its red baseline: probe output IDENTICAL to baseline (module-not-found `ctx_watchdog.ts`, TODO #141), sandbox smoke 64->76 ALL PASS, bt smoke 131/131 unchanged. Follow-up OPEN (todo_inbox 2026-10-22_01-34, cause-corrected): the sandbox smokes require `TEMP`/`TMP` at startup (line 13) — an env-free gate run crashes before pins; make the smoke's temp fixtures self-set the env. Live 3-tool acceptance: pending (maintainer). NOTE: the earlier close commit 4bef076 misattributed clone/probe state to a nonexistent planner-13 commit and restart; this line replaces that text (no history rewritten).
- **Live acceptance (BOTH units) 2026-10-22 — COMPLETE (planner-13, this session, host restarted per the maintainer marker):** 3-tool cross-repo probe ran from the main repo — `write` + `edit` + `block_transfer` COPY→PASTE into `C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/temp/live_accept_2026-10-22.txt` / `live_accept_bt_2026-10-22.txt` + `read` back; all accepted, NO sandbox refusal, NO `C:\Users\*` ask, PASTE content byte-correct on read-back, clone tree stayed clean (fixtures deleted). Mechanism side, intercept.log lines 3634-3640 (this session's sid): every `path=`/`orig=` field prints WHOLE incl. the 74-char clone paths (unit B LIVE); prose/payload fields cut with the ASCII `...` and the cut moves BACK before path substrings; the write-scope fuzzy line shows `fuzzy-rejected` (fail-closed, no filename mutation); unit A live via the accepted PASTE (CONFIG_ROOTS loaded at tool load). Remaining: the `out-of-sandbox` NOTE still fires on the allowed clone paths — observation-level only, see #142; entry can CLOSE once #142 lands.
- **Unit B status: LANDED 2026-10-22 (worker-13, ses_e9f24eaf)** — main repo commit a46ca9b: `intercept_observer_core.ts` FIELD CONTENT rule in `flattenField` (new `MAX_PATH_FIELD_CHARS = 1024` + helpers `isPathShapedValue`/`safeCutLen`: a single path-shaped field prints WHOLE; a mixed prose+path field's cut moves BACK so it never lands inside a path substring; fallback = the existing short summary form; the 8-field shape + ASCII `...` marker byte-unchanged) + smoke 85->93 pins (DoD items 1-3 incl. the hook long-path byte-shape pin) — green: smoke 93/93, probe 360/360 UNCHANGED (the probe's intercept sections audited in place: no check re-pinned, all expected values flow through the SAME `flattenField`; annotation/section sums still agree). Clone commit bdbf663 (branch `migration`): same rule transplanted into `plugins/intercept_observer.ts` + smoke 46->49 (C11 re-pinned: orig whole + value dropped whole; 3 new e2e pins) — clone gate as DIFF vs its red baseline (TODO #141): probe output byte-IDENTICAL to baseline (0 checks, `ctx_watchdog.ts` module-not-found), smoke ALL PASS (49/49). Entry stays OPEN: live acceptance pending (maintainer's next session shows whole paths in its own intercept lines; grep the log for mid-word path echoes).

## #141. (CLOSED 2026-10-22, plan13 — clone gate LANDED GREEN (clone `652ee6f`+`47aa082`: probe 360/360 + 12 smokes ALL PASS); header stamped at plan15 curation — the CLOSED status line existed under the stale OPEN header, 2026-10-22, planner-12) the v2 clone's probe gate is RED at baseline — `ERR_MODULE_NOT_FOUND` on `.opencode/plugin/ctx_watchdog.ts`
- **Problem / evidence:** measured in the clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`: `node .opencode/plugin/probes/handover_probe.mjs` → 0 PASS, module-not-found on `file:///…/.opencode/plugin/ctx_watchdog.ts` (the worker's baseline run 2026-10-21 attributes the import to clone commit 5516a83). So the clone has NO green probe gate; its 11 node smokes report 0 PASS each at baseline.
- **Desired outcome:** the clone's standard gate (probe + smokes) runs green against the v2 line, so v2 units are verifiable the same way as the main repo.
- **Acceptance criteria:** `node .opencode/plugin/probes/handover_probe.mjs` in the clone prints `PROBE handover: <t>/<t> PASS` with the annotation agreeing with the section sums; each v1-only import is either ported, pointed at its v2 twin, or excluded with a comment; the 11 node smokes each report their real counts (no 0-PASS suite).
- **Suggested scope (non-exhaustive):** clone `.opencode/plugin/probes/handover_probe.mjs` (the import list + section sums), clone `.opencode/plugin/` (the missing v1 plugin file vs the ported set), the clone suite header counts.
- **Status (clone-gate unit, 2026-10-22, worker-14, ses_e9effdf2):** LANDED — clone `migration` branch, 2 checkpoint commits (hashes recorded by the planner; not in this line by design): (1) probe v1-module constants repointed to `.opencode/plugin-unported-v1/` (ctx_watchdog/context_recovery/compact_memory/intercept pair/compaction_core; deactivated `TOOL_TS` kept — the file EXISTS in the clone) + a `.opencode/plugin-unported-v1/scripts/gauge.mjs` re-export shim (ONE implementation — resolves the v1 modules' byte-unchanged relative `./scripts/gauge.mjs` imports to the clone's gauge-core home) + probe check 32 re-pinned to the clone's table-aware gauge twin (7 ordered prepares — catalog x2, newest x2 session_v2-first, time_updated x2, step/tokens.total LAST; clone-twin precedent = unit A check-113); (2) 4 smoke files repointed (auto_resume incl. the 2 child-script import strings, compact_memory incl. reimport + source-read joins, context_recovery, intercept_observer). Measured green in the clone: `PROBE handover: 360/360 PASS` (section sums UNCHANGED — no section excluded, every subject exists); all 12 smokes ALL PASS: auto_resume 170/170, block_transfer.sandbox 76/76, block_transfer 131/131, compact_memory 110/110, context_recovery 17/17, context_trim 40/40, ctx_gauge 3/3, gauge_core ALL PASS (32 PASS lines measured — the suite's custom verdict line prints no n/n), intercept_observer 85/85 (the clone v1 smoke+core pair predates unit B's field rule — the v2 twin carries it at 49/49, out of this unit), loop_log 69/69, state_snapshot 20/20, submit 31/31. Baseline correction to the Problem line: measured 11 red smokes — actually 8 were already green (tools/gauge subjects exist in the clone); only the 4 plugin-module smokes were red. Main-repo gate untouched (probe 360/360 there, re-run after the unit). Entry stays OPEN: planner hash stamping + curation into #137's tools-estate units; decision-needed: NONE.

- **Status:** CLOSED 2026-10-22 (planner-13 stamp): clone `migration` `652ee6f` (probe repoint + gauge.mjs re-export shim — a NEW 7-line file, no v1 bytes touched) + `47aa082` (4 smokes repointed); main bookkeeping `c9c94a3` + `a5f6446`. Planner re-ran: clone probe `PROBE handover: 360/360 PASS` + main probe `360/360 PASS`; `git show --stat` matches both clone commits; worker's baseline correction accepted (only the 4 plugin-module smokes were red). The clone standard gate is GREEN; the v2 tools-estate survivors ride #137 (unit 2 = the token-mass legs, then the tools estate).

## #142. (closed 2026-10-22, plan14 — the #142+#138 field-scan unit LANDED + planner-verified: main dd5b63b, clone 622e75e; the cross-repo note is gone, the controls still fire) the `out-of-sandbox` NOTE fires for paths under a config-`allow`ed root — misleading evidence for cross-repo calls
- **Problem / evidence:** live probe 2026-10-22_02-16/02-17 (this session, sid `ses_e9f0c2d5…`): `write`/`edit`/`block_transfer` PASTE/`read` on `C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/temp/live_accept*` all EXECUTED, yet each logged `... | path=C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/temp/live_accept_2026-10-22.txt root=C:\Users\Wasiejen\Repos\opencode-agent | path | out-of-sandbox` although the clone root is in `opencode.jsonc` `permission.external_directory` (lines 34-35, allow). Cause: `intercept_observer_core.ts` `observeArg` calls `observeSandbox(s, workspaceRoot)` WITHOUT `extraRoots` (659); the `extraRoots` join is documented as "used when an R8 redirect fires" (545-550) and the recomputed-notes site (observer 1474) only filters after a redirect. The clone path is nested 3+ levels inside the allowed root → `resolveRedirect` 1:1 cases (i)/(ii) do not match → no redirect → the note stands.
- **Desired outcome:** the observation-level `out-of-sandbox` note does not fire for spans under ANY allowed root (workspace + scratchpad + config roots + references), so an agent reading its own log sees no false escalation signal for an allow-listed cross-repo call.
- **Acceptance criteria:** the same cross-repo call pattern logs NO `out-of-sandbox` verdict; a genuinely un-allowed path STILL logs the note; the B' hard-stop site (`observer:1519`, already full-roots) unaffected; intercept_observer smoke gains a pin per form (allowed nested path → no note; un-allowed → note); standard gate green in both trees (clone twin `plugins/intercept_observer.ts`).
- **Suggested scope (non-exhaustive):** `.opencode/plugin/intercept_observer_core.ts` (`observeArg` + `observeSandbox` call site), `.opencode/plugin/intercept_observer.ts` (pass `allowedRoots` through), `.opencode/plugin/tests/intercept_observer.smoke.mjs`, the clone twins.
- **Status:** LANDED 2026-10-22 (worker_FlashNext, ses_e9ee3329) — the note check now receives the allow-list roots (`observeArg` gained `extraRoots`; the plugin passes `allowedRoots` at the observation site AND the redirect recompute) and scans the TYPED path-field text: a path nested under a config-`allow`ed root logs NO `out-of-sandbox` note (main smoke (16a), clone smoke pin (i)), an un-allowed path STILL logs it (controls pinned: (7), (12g), (16b control), probe 295). #140's remaining "the NOTE still fires on the allowed clone paths" item is covered by this unit — stamped below. Planner stamp: main `dd5b63b`, clone `622e75e`; my re-runs — main smoke 96/96 + probe 360/360, clone `plugins/tests/smoke.mjs` 52 PASS + clone probe 360/360.

## #143. (OPEN, 2026-10-22, planner-19 — found by the unit-5a worker (todo_inbox), measured by the planner) the v2 clone's inert v1 tool twins `submit.ts` + `block_transfer.ts` are STALE copies of MAIN — a port reading the clone twin silently drops MAIN-side hardening
- **Problem / evidence:** measured 2026-10-22 across the trees (`cmp` + `wc -l` + `diff`): MAIN `.opencode/tools/submit.ts` 189 lines vs clone 167 — the clone copy LACKS the 2026-10-21 final-byte NEWLINE GUARD; MAIN `.opencode/tools/block_transfer.ts` 1034 lines vs clone 845 — 333 `diff` lines, the clone copy LACKS the MAIN hardening items (e.g. the item-10 POSIX temp-root override block at MAIN `:121-133`); the clone's v1 smokes pin the STALE copies (clone bt 132/132 vs MAIN 151/151; clone submit 31/31 vs MAIN 32/32-era set). Clean set (BYTE-IDENTICAL both trees): `loop_log.ts` 235, `ctx_gauge.ts` 36, `state_snapshot.ts` 206, `dev_get_tool_context_contents.ts` 59, `context_trim.ts` 1168.
- **Desired outcome:** the clone inert twins match MAIN's CURRENT v1 behavior, OR the port map records the per-file deltas explicitly — so the orders 5-8 (and later) ports cannot lose MAIN hardening by anchoring on the clone twin.
- **Acceptance criteria:** decision recorded (refresh-and-repin vs document-per-file). If refreshed: every clone `.opencode/tools/*.ts` BYTE-EQUAL to MAIN + the clone's v1 smokes repinned to the refreshed behavior + the clone gate green (probe + the repinned smoke counts). If documented: the §4 port map (`agent/research/2026-10-15_opencode-v2-impact.md`) or the port-map section names EVERY stale file + its delta, and the affected specs cite the MAIN twin as the baseline (the 5c spec already re-baselined on MAIN by planner-19).
- **Suggested scope (non-exhaustive):** clone `.opencode/tools/submit.ts` + `block_transfer.ts`, the clone v1 smokes `.opencode/plugin/tests/submit.smoke.mjs` + `block_transfer.smoke.mjs` + `block_transfer.sandbox.smoke.mjs`, `agent/research/2026-10-15_opencode-v2-impact.md` §4, `agent/handover/specs/2026-10-22_137u5c-block-transfer-v2.md`.
- **Status:** OPEN — decision-needed: refresh-and-repin (clone churn, keeps one design of record) vs document-per-file (cheap, drift-prone). NOT blocking: 5c is already re-baselined on the MAIN twin; the clone's stale twin stays inert + its smokes green either way.

## #144. (CLOSED 2026-10-22, plan22 — CLOSED planner-direct (small test-side fix): retry-on-EBUSY `rmTear` teardown in `_smoke_base.mjs` + the three node-dir smokes; trio green ×3 + gates 12/12 BOTH trees, clone `4045331`; header was stale-OPEN until this plan25 curation) three node-dir smokes exit 1 on the Windows teardown `fs.rmSync` EBUSY flake — 0 FAIL, green on re-run, but every gate run pays the re-run tax
- **Problem / evidence:** bun's final `fs.rmSync(fixtureDir,{recursive:true})` in
  `.opencode/plugin/tests/gauge_core.smoke.mjs` (scratch `gauge_v2_mass`),
  `context_trim.smoke.mjs` and `auto_resume.smoke.mjs` throws
  `EBUSY: resource busy or locked` AFTER ZERO check failures — measured identical
  at the clone baseline `358cba7` (scratchpad worktree) and `context_trim` +
  `auto_resume` fail the same way in MAIN; the open sqlite handles are the
  suspected holder.
- **Desired outcome:** the three exit 0 (`ALL PASS`) on EVERY run — no per-session
  re-run cost (each planner session currently burns a re-run trio at the gate).
- **Acceptance criteria:** running all smokes in `.opencode/plugin/tests` →
  13/13 exit 0, in a clean-env AND an env shell, BOTH trees; check counts
  UNCHANGED (gauge_core 42, context_trim 40, auto_resume 170). Fix class: retry-
  on-EBUSY teardown OR close every sqlite handle before the rm — worker's call.
- **Suggested scope (non-exhaustive):** `.opencode/plugin/tests/gauge_core.smoke.mjs`
  + `context_trim.smoke.mjs` + `auto_resume.smoke.mjs` (MAIN + clone twins).
- **Status: CLOSED 2026-10-22 (planner-22 direct, no worker — small test-side fix).** Fix class chosen: RETRY-ON-BUSY teardown — `rmTear` exported from `.opencode/plugin/tests/_smoke_base.mjs` (bounded backoff: `Atomics.wait` sleeps 25–175ms over 8 attempts on `EBUSY|EPERM|ENOTEMPTY`, non-busy errors rethrow; `freshSandbox` pre-clean uses it too); converted all `fs.rmSync` teardown calls in the three smokes (gauge_core 2 MAIN/3 clone incl. the clone's extra `d` site, context_trim 1 — its node:fs `rmSync` import dropped, auto_resume 14 both trees). Check counts UNCHANGED (gauge_core ALL PASS, context_trim 40/40, auto_resume 170/170); trio ran green ×3 BOTH trees; full v1 gate MAIN 12/12 + clone 12/12 exit 0. MAIN commit with this status line; clone `4045331`. Note: the entry said "13/13"; the folder holds 12 `*.smoke.mjs` — 12/12 is the real full-suite count.

## #145. (LANDED 2026-10-22, planner-25 — curated from `agent_ideas.md` 2026-10-22_00-36; pre-approved class — agent-usage friction) per-event filter helper for `.opencode/temp/intercept.log` — every live-acceptance sweep re-reads the log through raw greps and the field widths blow the context budget
- **Problem / evidence:** idea 2026-10-22_00-36 (planner_FlashNext ses_e9fb4e611…); `agent/scripts/log/summarize_intercept.cjs` is an AGGREGATE summary, NOT a filter/last-N tool (gap re-confirmed planner-24 NAP entry); full-line greps of intercept.log return the 8+ field lines (path fields are long) and blow context.
- **Desired outcome:** a curated read-only helper that filters intercept.log by date/session-id/verdict-kind and prints BOUNDED fields, defaulting to the last N lines.
- **Acceptance criteria:** a script (new `.mjs`/`.cjs` under `agent/scripts/log/`, or new flags on `summarize_intercept.cjs` — implementer's call) with a folder-README line; demo on the live intercept.log prints: (a) last-20 default, (b) `--session=<ses_…>` filter, (c) verdict-kind filter (e.g. `redirect|reject|applied`) — each output ≤ ~120 chars/line, exit 0; read-only (never writes the log).
- **Suggested scope (non-exhaustive):** `agent/scripts/log/` (new file + README), `agent/readme/repo_commands.md` (script pointer if a doc line fits).
- **Status: LANDED 2026-10-22 (worker_FlashNext ses_e9b41981…, code commit c3f856e; docs+TODO+handover pair 5d04c0c).** Decision taken at spec time: NEW file `agent/scripts/log/intercept_sweep.cjs` (aggregate summary stable, `parseLine` reused via require). Verified by the worker AND planner-re-run at plan26: `node agent/scripts/log/tests/intercept_sweep.smoke.cjs` → all checks PASS exit 0; `summarize_intercept.smoke.cjs` green and byte-UNCHANGED; live-log views (default 20 / `--lines=4` / `--session=`) exit 0; `--lines=200` → max 120 CHARS, 0 lines over (measured with a node char-count — `awk length()` over-counts because the log carries multibyte `→`/`…`); gate probe `360/360 PASS` unchanged. Opportunistic planner fix: `fit()` no longer splits a surrogate pair at the cut (the `�` artifact in truncated args fields). No maintainer decision needed (pre-approved).

## #146. (OPEN, 2026-10-22, planner-27 — measured in-session; the doc file is maintainer-owned per `agent/readme/repo_overview.md` line 13) `repo_commands.md` §Environment & shell claims git-bash `$TMP` = the user Windows temp dir — the live tool shell does not resolve it there, so scratchpad redirects fail or land outside the approved path
- **Problem / evidence:** `agent/readme/repo_commands.md` lines 30-32: "git-bash `$TMP` = the user Windows temp dir (unix form) — the pre-approved scratchpad for work outside the repo is `$TMP/opencode` (= `C:/Users/Wasiejen/AppData/Local/Temp/opencode`)". Measured 2026-10-22 (planner-27, in-session): `echo "TMP=[$TMP] TEMP=[$TEMP] TMPDIR=[$TMPDIR]"` → `TMP=/tmp TEMP=/tmp TMPDIR=[]` — `$TMP/opencode` resolves to `/tmp/opencode`, NOT the Windows path. Worker ses_e9b4198… (agent_feedback 2026-10-22_20-08) measured `$TMP`/`$OLDPWD` UNSET → `> "$OLDPWD/s.txt"` → `/s.txt` → Permission denied.
- **Desired outcome:** agent docs name a scratch path that actually resolves in this host's Git-Bash tool shell, so redirects to the approved scratchpad never fail or silently land elsewhere.
- **Acceptance criteria:** the Temp line matches a live `echo "TMP=[$TMP]"` measurement, OR names the literal path (`C:/Users/Wasiejen/AppData/Local/Temp/opencode`, unix form `/c/Users/Wasiejen/AppData/Local/Temp/opencode`) — which the tools' sandbox and block_transfer accept and bash reaches; no other doc line keeps the `$TMP`-variable claim.
- **Suggested scope (non-exhaustive):** `agent/readme/repo_commands.md` lines 29-32; verification = the one `echo` above.
- **Status/decision needed:** OPEN — doc-vs-live discrepancy in a maintainer-owned file (agents flag, do not edit: `repo_overview.md` lines 11-13). Recommendation: name the literal path, drop the `$TMP`-variable phrasing.

## #147. (unit 1 LANDED 2026-10-22 (worker, `dfebc3a`+`db91bb3` — the script + an 84-check selftest, dry-run zero-mutation PROVEN); still OPEN for unit 2 = the first real `--apply`, HIS act — 2026-10-22, direct planner-28 — the maintainer's ordering ask, he approved the direction: copy-paste between branches "would lose history… anything else is better") a port-sync helper that lands MAIN's `main`-branch changes into the v2 `migration` branch by MERGE — dry-run preview first, a fixed conflict-resolution boundary, gate-verified

- **Problem / evidence:** the two estates are ONE repo (same origin `wasiejen/opencode-agent`, merge-base `3297f1b`, MAIN `main` ahead 67, the clone's `origin/main` stale at `3bdfbd6`), yet every v2 unit carried changes between trees by hand — plan25 `90d6d6c` hand-carried the doc-noise filter into the inert v1 twin, and TODO #143 exists BECAUSE a twin went stale (clone `submit.ts` 167 vs MAIN 189; `block_transfer.ts` 845 vs 1034 — a port anchored on a stale twin silently pins stale behavior; the 5c spec had to be re-baselined mid-wave). Measured 2026-10-22 (read-only): `git -C <clone> fetch <MAIN path> main` then `git merge-tree --write-tree --name-only migration FETCH_HEAD` → 27 files touched by the merge, EXACTLY 7 conflicts: `.opencode/plugin/probes/handover_probe.mjs`, `.opencode/plugin/tests/block_transfer.smoke.mjs`, `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs`, `.opencode/plugin/tests/gauge_core.smoke.mjs`, `.opencode/tools/block_transfer.ts`, `agent/handover/handover_task_to_planner.md`, `opencode.jsonc`; everything else auto-merges. The v1 plugin move was recorded as `R100` renames (clone `5516a83`) → MAIN-side edits to those plugins follow into `.opencode/plugin-unported-v1/…`; a merge re-adds NOTHING under `.opencode/plugin/*.ts`.
- **Desired outcome:** a repeatable, history-preserving sync with a preview step, so branch-local state is protected by a rule instead of by planner memory, and the inert v1 twins never drift again.
- **Acceptance criteria:** NEW `agent/scripts/git/port_sync.cjs` (plain node): DEFAULT = dry run printing the merge-tree conflict set + the per-path resolution plan, ZERO repo mutation (verified by `git status` before/after); `--apply` = fetch → `git merge --no-commit FETCH_HEAD` → resolve with `--theirs` for `.opencode/tools/**`, `.opencode/plugin/probes/**`, `.opencode/plugin/tests/**` (the refresh that closes #143) and `--ours` for `opencode.jsonc`, `agent/handover/**`, `loop/**` → the discovery guard: any `.ts` NEWLY present under `.opencode/plugin/` (v2 discovery roots are `["plugin","plugins"]`, fork `packages/core/src/plugin/source-directory.ts:7`) is moved to `.opencode/plugin-unported-v1/` with a WARNING line → run the clone gate (probe + the 12 v1 node smokes + the 8 v2 bun smokes + tsgo) printing PASS/FAIL per suite → on any FAIL, `git merge --abort` and report; on green, commit the merge. Prints the exact commands it ran.
- **Suggested scope (non-exhaustive):** `agent/scripts/git/port_sync.cjs` + `agent/scripts/git/README.md` (or the `agent/scripts/README.md` row); `agent/readme/repo_commands.md` pointer is maintainer-owned — flag, do not edit.
- **Status/decision needed:** OPEN pending build; direction AND boundary RULED by the maintainer this session (ITER-28 direct). Boundary as ruled: plan state is ONE with MAIN as the source of truth — `agent/handover/**` + `loop/**` take MAIN's side (`--theirs`; they auto-merge anyway, only `handover_task_to_planner.md` conflicts, MAIN wins), SHARE also `TODO.md`, `agent/knowledge/**`, `agent/memory/**`, `agent/readme/**`, `agent/scripts/**` (auto-merge), refresh the inert v1 twins from MAIN rather than dropping the v1 estate from the `migration` tree (the clone's own gate runs those v1 smokes + the probe repointed per #141). ONLY `opencode.jsonc` stays branch-owned while BOTH hosts run — it is NOT outdated, it configures a DIFFERENT host (v2 schema: `permissions:[{action,resource,effect}]` array entries + the v2 agent/model rows vs MAIN's v1 object shape; the 5c plugin carries both shapes because of this) → the script must NEVER touch it (print a `config: skipped (host-local)` line instead).
- **Endgame (his #3 ruling, 2026-10-22):** the live host switches to v2 AFTER auto_resume v2 lands, so all later debugging happens on the v2 side with no switch-back; after that switch, checking out `migration` in the MAIN tree collapses the two-copy problem (one config + one NAP + one loop folder). The script addresses both trees by PATH, so it keeps working before and after that switch.
- **Status (unit 1 BUILT + GREEN, 2026-10-22, worker_FlashNext):** code commits `dfebc3a` (`agent/scripts/git/port_sync.cjs` + `agent/scripts/git/README.md` + the `agent/scripts/README.md` categories row) and `db91bb3` (`agent/scripts/git/port_sync.selftest.cjs` — `node agent/scripts/git/port_sync.selftest.cjs` → `checks: 84 pass=84 fail=0` + `SELFTEST: ALL PASS`, exit 0, scratch pair under `C:/Users/Wasiejen/AppData/Local/Temp/opencode/port_sync_selftest/`); real-tree dry run `node agent/scripts/git/port_sync.cjs` → `merge-tree: 7 conflict(s)`, 6 plan lines `take MAIN (--theirs)`, `config: kept branch-local (host-owned)`, `zero-mutation: PROVEN (both trees: HEAD + status --porcelain identical)`, exit 0. Deferred per spec: the first real `--apply` (the maintainer's act — the script stages only, `merge staged, not committed`) and the clone-gate run. Two spec-vs-entry deltas recorded: the acceptance line's `--ours` for `agent/handover/**`+`loop/**` is superseded by the ruled boundary (`--theirs`, MAIN wins) and the printed line is `config: kept branch-local (host-owned)` (not `config: skipped (host-local)`); `opencode.jsonc` is resolved `checkout --ours` so its bytes stay byte-identical to the clone's own (never written).
