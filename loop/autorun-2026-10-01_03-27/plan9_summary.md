# plan9 summary — 2026-10-01 (planner-9, ses_f0999d65dffeFxiJylRzecRbhu, direct→autorun)

## What landed
- **#134 B' + C** (out-of-sandbox hard-stop + doubled-path detector fix):
  LANDED `19f7e3a` (worker, early this session) — probe 355/355 + all 11
  smokes; live acceptance pending natural occurrence (post-restart host).
- **loop-iteration-counter proposal** FILED `1972eb2` (awaiting his ruling);
  inbox item moved to done/.
- **Context budget package** (#135): design exchange (3 commented proposals
  collapsed) → approved by the maintainer (`4b4c34e`) → spec wave
  (`1a9d376`/`2c1b053` — 5 specs + queue README) → BUILD:
  - **U0** keep-N surface — worker_Q3S `0538aa4` (context_trim 34/34) —
    planner-verified (node spot re-run).
  - **U1** recommendation leg — worker_Q3S `9f1f668` (context_trim 40/40) —
    planner-verified; decision: the surface report returns the cut-down
    form BY DEFAULT, the full form stays byte-identical via the core export;
    a `form`/`top` surface flag would require re-pinning probe 346
    (todo_inbox flag — maintainer call, non-blocking).
  - **U3** marker compaction — worker_Q3S `5c1e286`/`40e008f`
    (compact_memory 110/110) — planner-verified; the worker died at its
    context wall (170240/170000, finish=length) AFTER the final commit: no
    handover (reconstructed from files by the planner) + probe check 86 red
    (the compact_memory args-list pin) — the planner re-pinned 86 in place;
    spec-discipline bullet added (probe registration-shape pins are named
    pin surfaces for tool surface-arg changes).
  - **U4** nudge policy — worker_Q3S `4e2b901` (auto_resume 165/165) —
    planner-verified; the prompt-ladder reduction done by the planner
    (planner + worker prompts: live-nudge-primary, static thresholds as the
    Direct/absent-keys fallback; AGENTS_pending note for his paste). The
    `nudge1-4` keys are NOT enabled in the live config (absent = off —
    enabling is the maintainer's call).
  - **U2** fork per-message filter — QUEUED BEHIND the bun repair: its fork
    test/build gate needs bun, and bun.exe vanished from the global install
    ~17:45 (todo_inbox flag raised; the node fallback was verified for the
    whole agent-side gate in the meantime).

## Environment
- bun broken since ~17:45 (shim → missing exe); ALL gate runs under node
  v24.19.0 (friction feedback filed — a stable bun install or a documented
  node fallback in repo_commands.md would help).
- `.opencode/temp/compact_budget.json` live drift uncommitted (host-managed
  runtime state — left alone, per precedent).

## Inbox
- `try_to_prefer_the_slow_worker.md` handled → `maintainer/done/`: prefer
  worker_Q3S_slow from the next launch (worker_Q3S's model switch reprefills
  the planner session above ~150k planner context; the 245k window suits
  lookup-heavy units; the U3 limit-death is the data point).

## Gate at close (measured under node)
probe 355/355 + all 11 smokes: auto_resume 165, block_transfer 131,
block_transfer.sandbox 64, compact_memory 110, context_recovery 17,
context_trim 40, ctx_gauge 3, gauge_core ALL PASS, intercept_observer 85,
loop_log 69, submit 31.

## Next (ordered)
1. U2 (once bun is repaired — the spec carries the probe-346 re-pin note;
   then the maintainer's `build_fork.bat` run = the live gate).
2. #134 live acceptance (natural occurrence).
3. The research inbox item (`2026-10-01_research_memory_knowledge_usage.md`)
   — needs a design exchange + a delegation ruling before an explorer launch.
4. The loop-iteration-counter proposal — awaiting his ruling.
