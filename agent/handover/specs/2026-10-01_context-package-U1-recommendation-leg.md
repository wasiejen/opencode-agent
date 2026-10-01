# Task spec — U1: the recommendation leg on context_trim report

Design source (read first): `proposals/approved/2026-10-01_context-budget-package.md` §U1.

## Goal
A token-efficient, cut-down report form + a recommendation block + a
biggest-droppable list, on `context_trim` report.

## Verified facts (measured 2026-10-01 by the planner)
- `.opencode/tools/context_trim.ts` — `reportWindow` dispatch at L752; the
  report already emits per-message rows with token mass (`info.tokens`
  in+out+cache.read, bytes/4 fallback — the existing logic, reuse it) and
  tool-target detail (read filePath + offset/limit, bash command, webfetch
  url).
- The window slice reported = retained-tail slice + post-summary slice
  (the context_trim report semantics — read the function before extending).
- Smoke: `.opencode/plugin/tests/context_trim.smoke.mjs` (post-U0 count —
  re-verify at run); fixture DBs only.

## DoD (measured end states)
- Cut-down form: NUMBERED rows (1..N from the window start; number →
  messageID mapped internally — the agent never sees/enters dense ids),
  tool-target detail only (no content echoes), `last: N` flag.
- Recommendation block: **3 retention options — low / medium / high —
  each anchored at a messageID** (stable), each with the read / thought /
  shell mass split of retained vs dropped + the oldest-kept message's
  one-line hint. The mass figures are MACHINE-RECOMPUTED in the smoke
  from the fixture DB (the #99 pin style — never hardcoded tokens).
- Biggest-droppable list: top-N (default 5) heavy finished turns /
  call-result pairs — turn number + mass (the U2 target list).
- The FULL report form stays available and byte-identical (existing
  report pins stay green).
- Tool DESCRIPTION updated (the cut-down form + the recommendation block +
  the flags).
- Standard gate green: probe 355/355 + all 11 smokes.
- Checkpoint: one code commit per verified unit; TODO + handover in the
  final commit.

## DO-NOT-touch
- the live opencode.db (fixture DBs only); the fork; `maintainer/`; other
  tools/plugins; the proposal files; `agent/prompts/**`.
- New TODO entries: expectation only, never a promise.

## Worker
worker_Q3S (report extension + machine-recomputed pins).
