# TODO — maintainer's open items

Numbering: every entry ID is UNIQUE and NEVER REUSED — used so far up to #124, new
entries start at #125 (closed IDs stay reserved in the `todo_records.md` files —
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

## #124. (open, 2026-09-30, plan45 — curated from todo_inbox worker-45 new-hire test; pre-approved class — agent-usage friction removal) block_transfer return-line / description clarity: the 3 new-hire friction findings
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
- **Status:** OPEN — pre-approved (agent-usage friction removal); a small
  build unit (worker-delegable).

## #123. (closed 2026-09-30, plan45 maintenance pass — scripted sweep + batch re-point + one-line annotations) stale pre-carve commit hashes in TODO.md / NAP status lines — batch re-point or annotate (maintenance pass)
- **Problem / evidence:** the 2026-09-29 carve (filter-repo rewrite)
  invalidated every pre-carve commit hash; status lines still cited them.
  The 4 measured DEAD revisions with known post-carve equivalents:
  `cbbebf8` (#106) -> `7a9e291`, `0c90abe` (#115) -> `60f031b`,
  `12e3262` (#114) -> `fea1cbb`, `44c50a2` (R3 import fix) ->
  `9e91878`. Full sweep over TODO.md + todo_records.md + NAP: 340 unique
  7-hex citations, 307 dead (the carve rewrote the whole history, not just
  the 4 measured), 33 alive.
- **Desired outcome:** every cited hash either re-pointed to the
  post-carve equivalent or annotated pre-carve — a future `git cat-file -e`
  never dead-ends.
- **Acceptance criteria:** a scripted sweep (`git cat-file -e` over every
  cited hash) reports zero dead revisions, or each dead one carries a
  one-line annotation; TODO.md + todo_records.md + NAP archive covered.
- **Suggested scope:** TODO.md, todo_records.md,
  `agent/handover/handover_planner.md` (compressed archive), loop-folder
  summaries (read-only annotation if stale).
- **Status:** CLOSED (2026-09-30, plan45) — the 4 verified pairs re-pointed
  inline (33 occurrences; each target script-verified on the current
  branch); the one-line pre-carve annotation landed in the headers of
  TODO.md, todo_records.md, and the NAP compressed archive, covering the
  other 303 dead historical citations; the loop-folder summaries carry
  179 dead citations (historical — not re-pointed per the read-only scope,
  covered by this entry + the TODO.md header note).

## #121. (closed 2026-09-30, plan40 planner-direct — inline, doc-only) two stale refs in the repo_commands.md parts post-#117/#113 — fixed: (a) `agent/readme/repo_commands.md`'s FST-commands pointer now points at `projects/Free-Snap-Tap/repo/repo_commands.md` (was a self-reference); (b) the FST part's "(#113: the venv's python.exe is currently broken …)" note updated to the 3.12.9 close

## 64. (closed 2026-09-16, plan9 planner-direct; finding 2026-09-16, worker-13 inbox) — stale "84/84" probe baseline in `.opencode/plugin/README.md` → replaced by the curate-don't-duplicate pointer to the probe's self-annotated header total (the #58 convention)

## 40. Explorer run #1 output unreliable — no entries on disk, no commit, fabricated gauge, endpoint 128k ≠ 256K (2026-09-10) (closed 2026-09-10 by maintainer ruling, see todo_records.md)

## 50. (closed 2026-09-11, see todo_records.md) — repo_map.md refresh — two stale bullets from the split build (2026-09-10, worker findings, iter-3 curation)

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
  preliminary scanning. The `--defer` stays.**
- an option is also; fuzzy name resolution search on read or when searching in files. or num_to_word autoreplace as intercept plugin on hook.execute.before to combine both and make tools calls more reliable even with bitshifts in numbers. worthwhile thing to research. but dont save research in your nap. make e.g. a agent/research folder if you want. see ideas #5 #6 #7

## 39. (closed 2026-09-10, see todo_records.md) — Looprunner prompt v2 proposal — applied + smoke test clean (2026-09-10)

## 49. `handover_task.md` worktree/HEAD conflict (2026-09-10) (closed 2026-09-11, see todo_records.md)

## Plugin & gauge (open)

## 74. (closed 2026-09-27, direct session ses_f20d1b39… — backend no longer on ik_llama, the offending PR was reverted — non-issue; full text in todo_records.md) — **Write tool fails on long content payloads on this host**

## 17. (closed 2026-09-11, see todo_records.md) — v1.3 log-growth CONFIRMATION — one-shot read, deferred by the no-`plugin.log` constraint (2026-09-08)

## 30. (closed 2026-09-11, see todo_records.md) — De-peek: replace the peek.py shell-out with an in-plugin `node:sqlite` read (2026-09-09)

## 37. (closed 2026-09-10, see todo_records.md) — Production plugin host lacks `node:sqlite` — the ctx nudge never lands in production (2026-09-10)

## 33. (closed 2026-09-12, see todo_records.md) — v2.5 auto-nudge ladder — LANDED (T2: per-session read, rungs 50/70/80/90/5k, dedup per rung, `promptAsync` synthetic-part delivery, `kind:"nudge"` evidence only; probe 52/52) + production evidence complete (50/70/80 % rungs fired in live planner sessions; the per-session read mechanic reached EVERY acting session per the maintainer's target-scope ruling); the v1.3 log-profile tail resolved via the executed one-shot read (#17 CLOSED). The stale 09-10 "NOT landed" note referred to the pre-wiring state; both T1 (de-peek, #35) and T2 (ladder) are landed.

## 38. (closed 2026-09-10, see todo_records.md) — (TEST) explorer smoke test — jill gemmaQ4-256K first launch

## 35. (closed 2026-09-11, see todo_records.md) — T1 de-peek build — LANDED (continuation 2); tail closed: v1.3 log-profile re-baseline executed (one-shot read → #17) + #34 residual doc refs (closed)

## 51. (closed 2026-09-16, plan8; 2026-09-11, T3 worker flag) — Stale probe header vs `.opencode/package.json` "type" field

## 52. (closed 2026-09-13, see todo_records.md) — `compact_memory` fails in the current host build — connection error on both paths (2026-09-12) — LANDED (2026-09-12, worker-2, per the approved v2 proposal) + live acceptance DONE (2026-09-13, iteration 1: compaction part + directive + budget 1/3 + COMPACT line verified in the DB); the resume-overflow finding → `proposals/2026-09-13_compact_memory-findings.md` (Item 1 superseded by the 2026-09-15 protocol; Item 2 ruling bundled in 2026-09-15_backlog-decisions.md, Decision 3).

## 65. (closed 2026-09-17, maintainer-ruled — NOT a tool bug, see todo_records.md for the full entry if needed) — loop_log tool folder-detection bug: spurious folders on the maintainer-renamed loop folder (2026-09-16, plan2)

## 66. (closed 2026-09-23, planner-13 bookkeeping; full text in todo_records.md) — 5.3+5.4 restart acceptance was stale: the read-scope mutation channel was proven LIVE by #68's one-shot live-acceptance (2026-09-17); the §5.4 sentinel torn down; intercept.log accumulated to 3076 lines / 139 sessions by 2026-09-23.

## 67. (closed 2026-09-28, plan30 maintenance pass — R3 build LANDED 2026-09-26 (3ec1c5c/9e91878/20d5a48, gate 337/337 + io 77/77) + live acceptance COMPLETE 2026-09-26 (plan23 re-test: grep/glob pair + bash quoted-form + bt anchor-marker pair live-accepted; section-anchor pinned-only, schema-shadowed — dormant by design on this host; see #95 closed status)) — Fuzzy scope extension: glob / grep / section-anchor resolvers (2026-09-16, plan2 queue)

## 68. (closed 2026-09-16, full text in todo_records.md) - Write-scope fuzzy (R2): approved + build landed green (35f8143) + one-shot live-accepted 2026-09-17 (benign mistype corrected; the #72 hazard live-measured; residual hazard -> #72 M1)

## 69. (closed 2026-09-16, full text in todo_records.md) - Redundancy form codification (the [left:right] pair convention, supersedes the Q2 angle-pipe form): AGENTS.md paste landed (bf18f14) + R1 green (96bb173) + role-prompt pointer lines (3e0406c); acceptance fully met

## 70. (closed 2026-09-28, plan30 maintenance pass — all units LANDED + live-accepted: unit A 6864bc0 (2026-09-21) + unit B d4ef76e (2026-09-22); live acceptance 2026-09-21 (DUMP-OK PASS + config resolution; the cross model-read bug found+fixed 280b8d0 + re-accepted); the requested research-spec follow-on superseded by the #99/#101/#105 research tracks) — compact_memory rework: config-resolved summarizer + queued message + dump diagnostics (2026-09-16, new priority.md item; re-scoped 2026-09-21 by his priority.md #1)

## 71. (closed 2026-09-17, planner-direct, full text in todo_records.md) - Stale probe totals in repo_commands.md: section now carries the curate-don't-duplicate pointer (per #58/#64; maintainer ruled the planner is allowed to update the file)

## 72. Write-scope residual hazard: new-file near-miss (maintainer decision; 2026-09-17)
- **Problem / evidence:** the write-fuzzy channel (and the pair gate) cannot
  distinguish "a mistyped path to an EXISTING file" from "a deliberately
  NEW filename that happens to sit within d<=1 of an existing sibling" —
  a legitimate new-file write (e.g. creating `file-5.txt` next to
  `file-4.txt`) can be mutated onto the sibling and overwrite it. Read
  scope has no such hazard (non-destructive). Pinned as behavior: S20
  checks 200/201 + smoke 8f (worker R2, commit 35f8143).
- **Outcome (decision needed):** accept as designed (audit lines carry
  the ORIGINAL arg — re-targeting verifiable after the fact), OR mitigate
  later with an intent signal the interceptor does not currently have
  (e.g. agent confirms the log line before the write lands — R6-era
  surface).
- **Acceptance:** your ruling recorded; if mitigate: spec'd as a follow-on
  stage (R6-adjacent), not built before approval.
- **Status:** RULING 2026-09-17 (direct session): **M1 approved** — restrict
  the implicit write-fuzzy to `edit`/`block_transfer` (no new-file intent is
  legal there → redirect is unambiguous); `write` loses the implicit channel
  (new-file IS a legal intent → every degraded outcome becomes a visible
  stray file, never a silent overwrite); the pair channel stays unchanged
  (strict existence, fail-closed). **M1 LANDED (commit 9ec4c0b, 2026-09-17,
  worker):** dispatch guard in intercept_observer.ts (write excluded from the
  fuzzy channel); S20 re-pinned (196/198 no fuzzy line, 200/201 NOT mutated +
  zero lines) + new edit counter-pins 208/209; probe 208/208, smoke 36/36,
   pytest 459+1w, ruff F=0. **LIVE ACCEPTED 2026-09-17 (post-restart one-
   shot, scratchpad fixture, torn down):** the d=1 new-file write landed
   LITERAL (zero fuzzy lines — the guard live; the d=1 sibling untouched —
   the hazard is dead) AND the `edit` d=1 typo was still corrected
   (`fuzzy scope=write … d=1 gap=inf`, no stray file). Follow-on: R7
   (segment resolver) + R8 (root re-anchoring) STAGED + design agreed
   (substitution bar approved 09-17, decision-record §5); R9 documented-
   optional. See NAP 2026-09-17 direct session.

## #73. (closed 2026-09-17, full text in todo_records.md) - R7 realistic doubled case: the collapse-adjacent-dup existence-gated pre-check (dce82ad) resolves the doubled-folder case (kind=dedup evidence); live-accepted 2026-09-17 (read/edit proven live; doubled-write pinned 21/218-220)

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
  - Unit 4 (planner liveness watchdog): LANDED + smoke-verified; LIVE ACCEPTANCE PENDING the next host restart (the four acceptance cases in the proposal lines 138-142).
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
- **Status:** DEFERRED — picked up only when nothing else is open (his
  2026-09-23_01-08 inbox entry; runs after #85 part 3 lands — now clear
  of that constraint).

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

## #102. (LANDED 2026-09-26, plan20, worker-20 `worker_Q3S_245K_slow`, commit be07ce6 — live acceptance pending the maintainer's process restart; maintainer live report 2026-09-26) R8 redirect extension: map the POSIX `/tmp` (and `/var/tmp`) root into the sandbox — unmappable forms STOP the loop
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

## #106. (LANDED 2026-09-28, plan31 unit 2, worker-31 — live acceptance pending the maintainer's restart; pre-approved class — agent-usage friction removal) edit-fuzzy hint line should state APPLIED vs REJECTED
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

## #109. (LANDED 2026-09-28, plan33 unit 1, worker-33 — live acceptance pending the maintainer's restart; was: 2026-09-28, plan28 ideas.md scan; research → design) silent context-limit stops carry NO signal — a worker dying at the wall without self-compaction returns an empty result with no log line
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
  of the standard gate is BLOCKED by #113 (exit 103, reported in the
  handover, not fixed). LIVE acceptance PENDING the maintainer's host
  restart (one worker dying at the wall → the `-WARNING` line lands in
  `loop_log.md` within ~65 s — doc §5 DoD). (Commit hashes recorded in
  the planner's follow-up bookkeeping commit — no self-reference.)

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

## #119. (LANDED 2026-09-29, direct session ses_f11b625d3ffeio02fDzjzypbN2, planner-direct — live acceptance pending the maintainer's restart; found mid-#117 worker run) compact_memory's budget increment rewrites `.opencode/temp/compact_budget.json` as a v2 sessions-only file, destroying all other keys
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
  in the handover (opencode.jsonc = the maintainer's domain). Live acceptance
  pending: `report` is read-only → the planner live-accepts it after the
  maintainer's restart + registration; `tail` = a throwaway session, the
  maintainer's call. (The final hash rides the planner's follow-up
  bookkeeping commit.)
