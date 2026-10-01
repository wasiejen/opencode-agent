# Worker summary — U0: keep-N surface (`keepMessages`) for context_trim tail

Status: DONE — every DoD item achieved (measured below). One verified unit
→ one code commit.

## What changed
1. `.opencode/tools/context_trim.ts` — the existing `tailSetKeep` core
   export wired into the tool's `tail` mode as the keep-COUNT form:
   - New `keepMessages` arg (`number`, optional — the compact_memory
     keepMessages schema precedent) — mutually exclusive with `target`;
     the surface decides the form, the core's existing guards cover the
     rest (zero new validation logic, zero new rejections).
   - `tail <session> keepMessages=N` → pure delegation to `tailSetKeep`;
     the return line is the core's pinned `tail-set= <old> -> <new>
     keep=N` — identical in shape to the messageID form (the prefix is
     already remapped by the core per the source proposal's Part 1 — no
     surface remap; the auto-resume/probe/smoke pins on that prefix are
     untouched).
   - Surface guards: both forms given →
     `error: tail mode: target and keepMessages are mutually exclusive
     (use one or the other)`; neither → `error: tail mode requires target
     (the new tail_start_id message id) or keepMessages (the retained-tail
     message count)`.
   - The tool's db path now resolves at call time via the gauge's
     `getDbPath()` (the gauge core's own resolution — the live opencode
     DB on the live host; smoke/probe steer it via the existing
     `setDbPath` test hook, the ctx_gauge-smoke pattern). No live
     behavior change: `getDbPath()` returns the default path unless the
     test hook is called.
   - Tool DESCRIPTION updated (MODES line + tail-mode parameter notes):
     the keep form, mutual exclusion, count semantics = compact_memory
     keepMessages (a message COUNT, NOT a token budget), the
     `tail-set=` return line, the shrink-lever framing (growing stays
     available; keep=0 stays rejected by the floor 6). File header
     comments updated to match.
2. `.opencode/plugin/tests/context_trim.smoke.mjs` — new section 10:
   keep-form pins THROUGH THE TOOL SURFACE (`execute()` + the gauge's
   `setDbPath` steering at the fixture; a dedicated `ses_ct_sfc` session
   so the earlier pins' state is untouched): valid (keep=8 → `tail-set=
   msg_f05 -> msg_f03 keep=8` byte-exact, the part row landed byte-exact,
   the SURFACE report shows retained=8) + each of the 5 rejections
   (floor 5 / keep-exceeds-history 50 / no-completed-compaction /
   session-not-found / keep-invalid 0 — plain lines, NO DB write pinned
   by a part-row read-back) + the two surface guards. The existing 29
   pins are untouched (the diff is pure addition).
3. `.opencode/plugin/probes/handover_probe.mjs` — check 346 re-pinned IN
   PLACE (args now `[mode, session, target, keepMessages]` + the
   keepMessages schema safeParse pins: undefined/8 pass, "8" fails). No
   new probe checks — the total stays 355.
4. `agent/knowledge/knowledge_tools.md` — the keep-N surface knowledge
   one-liner (Part 2 of the approved source proposal).
5. `agent/knowledge/knowledge_plugins.md` — the probe-tally one-liner
   (check ids sparse/hand-assigned; total = check-call count).

## Measured verification (2026-10-01, branch `opencode_test`)
- Pre-change baseline: probe `PROBE handover: 355/355 PASS` (exit 0);
  context_trim smoke 29/29.
- The one named re-verification (spec Verified-facts): the current S33
  range vs the probe header tally — S33 = checks 346-351 (6 checks),
  S34 = 352-353 (2), total 355 — AGREES with the header annotation
  (verified by a run + a scripted id-tally before touching the probe).
- Post-change STANDARD GATE GREEN:
  - Probe: `PROBE handover: 355/355 PASS` (355 PASS lines, 0 FAIL, exit 0).
  - All 11 smokes ALL PASS: auto_resume 155/155, block_transfer.sandbox
    64/64, block_transfer 131/131, compact_memory 89/89,
    context_recovery 17/17, context_trim 34/34 (the existing 29 + 5 new
    surface pins), ctx_gauge 3/3, gauge_core ALL PASS,
    intercept_observer 85/85, loop_log 69/69, submit 31/31.
- The live opencode.db was NEVER written: the smoke uses fixture DBs
  only; the surface pins are steered at the fixture via `setDbPath` and
  the original path is restored in the smoke.

## Commit
- One code commit carries the unit (tool + smoke + probe + both
  knowledge one-liners + this handover + the loop-log close line),
  subject: `U0: context_trim tail gains the keepMessages form (the
  tailSetKeep surface)`. The hash is recorded by the planner in the
  follow-up bookkeeping commit (a commit can never carry its own hash).
- TODO entries: NONE — no genuine discrepancy found (the S33
  re-verification agreed; nothing out of scope surfaced).

## Deliberately NOT done
- No `tail-set=` → `tail=` prefix remap at the surface — the core's
  pinned prefix is returned as-is (Part 1: "the `tail=` return-line
  prefix is already remapped by the core"); the return-line SHAPE is
  identical to the messageID form (`<old> -> <new> keep=<count>`). If a
  literal `tail=` prefix is wanted for the keep form, it is a one-line
  surface remap — flagged for the planner's call.
- No new probe checks (DoD: probe 355/355) — check 346 re-pinned in
  place instead.
- Nothing in the DO-NOT-touch list was touched: live DB, the fork,
  `maintainer/`, other tools/plugins, proposal files, `agent/prompts/**`.
  (Pre-existing uncommitted changes — `.opencode/temp/compact_budget.json`,
  `agent/agent_feedback.md`, the untracked planner-session dumps — were
  left alone and are NOT in the commit.)
