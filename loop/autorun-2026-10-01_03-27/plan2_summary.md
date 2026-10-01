# plan2 summary — autorun-2026-10-01_03-27, iteration 2 (ses_f0ad5d279ffe5cso7Q0CL60RD2, planner-2)

Two queue units LANDED + verified, one maintainer question answered.

## #127 compact-message unit 4 (worker_Q3S ses_f0ad0bbfcffeCKb4FmU2gK7c29)
- Code 5ed36cf / close f78f556: `restartIntentSection(sid)` in
  `auto_resume.ts` — fail-open read of `.opencode/temp/compact_message_<sid>`,
  `.consumed` tombstone rename, `intent= sid=` log line; appended at the
  restart-branch spawn call site, `restartText` stays pure.
- Smoke 147 → 152 (5 pins; the 3 DoD scenarios, absent case byte-identical).
  Gate: probe 352/352 + all 11 smokes. Planner spot re-ran auto_resume
  152/152 green.
- Planner bookkeeping: the deferred item-4 doc line landed in
  `.opencode/plugin/auto_resume.md`; TODO #127 LANDED (worker commit);
  live acceptance = the next restart-branch spawn with a queued file
  present (the `intent=` log line is the detector).
- Handover: `plan2_ho_task.md` + `plan2_ho_task_to_planner.md` (this folder).

## #128 compact_memory cross-override (worker_Q3S ses_f0abc032dffenb1K8yrfXKOg5V)
- Code 5c1af35: gate `isCross`/`effCap` (the emergency branch now SELF-only;
  cross@cap dispatches via the override, cross@cap+1 stays refused — cross
  spendable total = self total, the ruling) + Part-3 `ovr` token on the
  verified-success COMPACT line + Part-2 worker-part docs.
- Smoke 82 → 89 (override pins a/b/c + Part-3 line pins + self regressions
  d/e/f + cap-0 corner ×2 + IQ4 cap-3; old cross-shape pins re-pinned —
  incl. the CPU plain-call pins → SELF shape); probe 352/352 (S13
  92/93/95/222-226 + S32 340 re-pinned, count unchanged). Planner spot
  re-ran compact_memory 89/89 green.
- Planner-side doc parts LANDED in this session's bookkeeping commit: the
  CROSS-dispatch bullet in `agent/prompts/agent_planner.md` + the AGENTS.md
  Budgets block staged at repo root `AGENTS_pending_2026-10-01.md` (his
  paste).
- Live acceptance = his next real cross-override dispatch (the ` ovr`
  COMPACT line + the budget count cap+1).

## Maintainer inbox `question.md`
- Question: the bash avoid-list instruction — why it exists, where it comes
  from (handover_task or general). Answer (evidence): it is the host's
  built-in bash tool description from the fork,
  `packages/opencode/src/tool/shell/prompt.ts` L100-106 (injected into every
  session's bash schema); not in AGENTS.md / prompts / handover. It steers
  file ops to the dedicated tools; `cp` is NOT on the list (no
  dedicated-tool equivalent for a plain copy) — the session's cp use was
  correct. Answered in the file + moved to `maintainer/done/question.md`.

## Baselines
- auto_resume smoke 147/147 → 152/152 (#127)
- compact_memory smoke 82/82 → 89/89 (#128)
- probe 352/352 (re-pinned under #128, check count unchanged)

## Commits (this session)
- 6784b63 plan2 bookkeeping: #127 live spec staged + queue advanced + NAP
- c959f37 plan2 #127 verified + bookkeeping (handover to loop folder, doc
  line, question answered, specs README, NAP)
- d1ab253 plan2: #128 live spec staged
- (worker units: 5ed36cf + f78f556 (#127), 5c1af35 (#128))
- this closing commit: #128 planner-side doc parts + TODO status + summary

## Open / next
- FST queue (#129/#130/#131), live-acceptance residue (incl. #127/#128),
  #120 Unit 2 UNHELD — see the NAP queue.
