# Task spec — unify the gauge core so the MAIN→clone merge lands green (TODO #147)

## Goal

Make ONE gauge core and ONE probe serve both trees, so `port_sync --apply` ends
`gate: PASS` and the merge can be committed on the clone (`migration`).

Durable route (chosen by the maintainer: "durable is always better than an interim fix"):
MAIN adopts the clone's table-aware gauge core, and MAIN's probe + gauge smokes are
re-pinned to the unified core. The clone keeps its own files only until the merge lands them.

## Facts measured (do not re-derive)

- MAIN core: `C:/Users/Wasiejen/Repos/opencode-agent/.opencode/plugin/scripts/gauge.mjs` = 769 lines, v1 schema only (`tokens.total`), 2 ordered prepares.
- Clone core: `C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/plugin/scripts/gauge.mjs` = 1015 lines, table-aware: catalog probes x2 → `session_v2`/`session` newest x2 → `time_updated` x2 → mass (`tokens.total` OR `session_message`), 7 ordered prepares.
- Probe loads it RELATIVE: `handover_probe.mjs:1198` → `await import(new URL("../scripts/gauge.mjs", import.meta.url).href)` (path-agnostic — no patch needed).
- Only 2 probe checks fail today in the merged tree: `[32] unit-mock bun:sqlite ...` and `[38] fallback: ...` (clone probe passes 360/360; MAIN probe fails those 2 against the clone core).
- Probe divergence between trees: 276 diff lines / 12 hunks (see `C:/Users/Wasiejen/AppData/Local/Temp/opencode/probe_diff.txt`). 3 hunks = the gauge pins, 3 = path repoints, the rest = clone lagging MAIN.
- `port_sync.cjs` Rule (A) already landed (MAIN `f651dd6`): the probe takes MAIN + a post-merge patch repoints its `".opencode", "plugin", "<rel>.ts"` refs to `plugin-unported-v1`; `.opencode/plugin/probes/` is in `TAKE_MAIN`.

## Definition of done

1. MAIN working tree green:
   - `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: N/N PASS` with N >= 360, exit 0.
   - `node .opencode/plugin/tests/gauge_core.smoke.mjs` and `node .opencode/plugin/tests/ctx_gauge.smoke.mjs` → `SMOKE: ALL PASS`, exit 0.
   - the other 10 `.opencode/plugin/tests/*.smoke.mjs` unchanged and still PASS.
2. `node agent/scripts/git/port_sync.cjs` (dry) prints a `plan:` line taking MAIN for
   `.opencode/plugin/scripts/gauge.mjs`; the selftest prints `SELFTEST: ALL PASS`.
3. One commit per logical change; the planner performs the sync and the merge commit.

## Approval boundary

Agent-side tooling and gate pins only. No host behavior change for the maintainer, no
new user-visible feature. The merge itself is the maintainer's approved act and is the
planner's job, not this worker's.

## Suggested scope (non-exhaustive)

- `.opencode/plugin/scripts/gauge.mjs` (MAIN) — replace with the clone's core.
- `.opencode/plugin/probes/handover_probe.mjs` (MAIN) — re-pin checks 31-40 to the unified core.
- `.opencode/plugin/tests/gauge_core.smoke.mjs`, `.opencode/plugin/tests/ctx_gauge.smoke.mjs` (MAIN) — re-pin.
- `agent/scripts/git/port_sync.cjs` + `agent/scripts/git/port_sync.selftest.cjs` — add the take-main glob.
- `TODO.md`, `agent/handover/handover_task_to_planner.md`.

## Procedure (guidance, not protocol)

1. Take the core from the clone verbatim, do not rewrite it:
   `git -C C:/Users/Wasiejen/Repos/opencode-agent-v2 show HEAD:.opencode/plugin/scripts/gauge.mjs > .opencode/plugin/scripts/gauge.mjs`
   (run from MAIN; then `wc -l` must read 1015).
2. Run `node .opencode/plugin/tests/gauge_core.smoke.mjs 2>&1 | tail -20` and
   `node .opencode/plugin/probes/handover_probe.mjs 2>&1 | grep -E "^(FAIL|PROBE)" | head -8`.
3. Re-pin only what the unified core legitimately changed: pin to the OBSERVED
   behavior when that behavior is correct for BOTH a v1-only DB (no `session_v2`
   table) and a v2 DB (`session_v2` + `session_message`). The probe's own MOCK
   decides which catalog the core sees — make the MOCK explicit per check so the
   pins are tree-independent. If the unified core returns `no-total`,
   `notAvailable`, or throws against a v1-only DB catalog, STOP and report: that is
   a core defect, not a pin to move.
4. Keep one implementation: never fork the core in MAIN, never add a second gauge
   module. `.opencode/plugin-unported-v1/scripts/gauge.mjs` (clone, 7-line shim)
   already re-exports `../../plugin/scripts/gauge.mjs` — leave it.
5. `port_sync.cjs`: add `.opencode/plugin/scripts/` to `TAKE_MAIN`; extend the
   selftest rule-layer pins (`isTakeMain(".opencode/plugin/scripts/gauge.mjs")`,
   `classify(...) === "take-main"`) and, in the fixture, add a conflicting
   `.opencode/plugin/scripts/gauge.mjs` on both sides so the apply case asserts the
   MAIN bytes land in the clone copy.
6. Commit: one commit for the core swap + its pins, one for the port_sync glob.
   Never stage `opencode.jsonc` or `maintainer/**`.

## Large-file discipline

The core is 1015 lines and the probe ~4600. Never read either whole. Use
`grep -n "<anchor>"` plus `sed -n "<range>p"`, and write with chunked
`edit`/`block_transfer` calls of at most ~8 KB. Re-grep exact line ranges
immediately before any line-anchored surgery.

## Worker

`worker_FlashNext`. Work happens in `C:/Users/Wasiejen/Repos/opencode-agent`
(branch `main`, stay on the current checkout). Write
`agent/handover/handover_task_to_planner.md` with the change summary, the exact
verification commands with PASS/FAIL, the working-tree status, and the commit hashes.
