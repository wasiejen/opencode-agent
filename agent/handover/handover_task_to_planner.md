# Worker summary — #147 gauge-core unification (MAIN adopts the clone's table-aware core)

Status: COMPLETE (green, committed). Task spec `agent/handover/handover_task.md` (spec commit `f9baff6`).
Work tree: `C:/Users/Wasiejen/Repos/opencode-agent`, branch `main`. No merge performed, no `port_sync --apply` run — those stay the planner's/maintainer's act.

## Files changed (MAIN)

| File | Change |
|---|---|
| `.opencode/plugin/scripts/gauge.mjs` | REPLACED verbatim with the clone's table-aware core (769 → 1015 lines). Blob `b4bb98eb8b064294d3d38445dd937236ebc23694` = byte-identical to the clone's `HEAD:.opencode/plugin/scripts/gauge.mjs`. |
| `.opencode/plugin/probes/handover_probe.mjs` | 3 gauge-pin hunks taken from the clone probe, nothing else: the bun:sqlite unit-mock `get()` (mass prepare keys on `tokens.total` OR `session_message`), check 32's 7-ordered-prepares assertion, `MockBunNoStep` `get()`. 8677 → 8691 lines. The other 20 diverging hunks (path repoints + clone lag) untouched. |
| `.opencode/plugin/tests/gauge_core.smoke.mjs` | := the clone's superset (v1 pins + the v2 table-aware read/mass/spawn pins, 370 lines) with MAIN's two `// #144 retry-on-EBUSY` annotations re-added at the `rmTear(tmpC…)` / `rmTear(tmpB…)` lines. |
| `agent/scripts/git/port_sync.cjs` | `TAKE_MAIN` += `.opencode/plugin/scripts/`. |
| `agent/scripts/git/port_sync.selftest.cjs` | +5 checks: the gauge-core take-main pin, `classify(...) === "take-main"`, a fixture `gauge.mjs` conflicting on BOTH sides (base → clone v2 / main unified) with the dry conflict + `plan:` pin, the apply assertion that MAIN's bytes land in the clone copy, and the abort-restore pin. 82 → 87 `ck(` sites. |
| `TODO.md` | two bullets appended to the `#147` entry (LANDED status + the measured spec discrepancies). |

Unchanged, verified still green: `.opencode/plugin/tests/ctx_gauge.smoke.mjs` (live-db pins — no schema pin to move) and the other 10 `.opencode/plugin/tests/*.smoke.mjs`.

## Behavior changed

One gauge core serves both trees. Against a v1-only DB (no `session_v2`) the unified core emits the v1 SQL byte-identically, so every pre-existing MAIN pin (S6 21-28, S7 29/30/31/33-39, S33/S34, the 12 smokes) passes unchanged. The unit-mock pins now script the coexisting catalog, so they assert the table-aware prepare order in BOTH trees (tree-independent pins). The clone's v2 smoke pins now live in MAIN, so `port_sync` (tests + scripts take MAIN) cannot drop them at merge.

## Verification (exact commands, run in MAIN, post-change)

| Command | Result |
|---|---|
| `node .opencode/plugin/probes/handover_probe.mjs` | `PROBE handover: 360/360 PASS`, exit 0 — PASS |
| `node .opencode/plugin/tests/gauge_core.smoke.mjs` | `GAUGE_CORE_SMOKE: ALL PASS`, exit 0 — PASS |
| `node .opencode/plugin/tests/ctx_gauge.smoke.mjs` | `CTX_GAUGE_SMOKE: ALL PASS (3/3)`, exit 0 — PASS |
| `for f in .opencode/plugin/tests/*.smoke.mjs; do node "$f"; done` | 12/12 exit 0 (`ok=12 fail=0`) — PASS |
| `node agent/scripts/git/port_sync.selftest.cjs` | `checks: 106 pass=106 fail=0` + `SELFTEST: ALL PASS`, exit 0 — PASS |
| `node agent/scripts/git/port_sync.cjs` (dry) | exit 0, `zero-mutation: PROVEN (both trees: HEAD + status --porcelain identical)`, `merge-tree: 9 conflict(s)`, 8 `take MAIN (--theirs)` plan lines + `config: kept branch-local (host-owned)` — PASS (see delta below) |
| `git -C C:/Users/Wasiejen/Repos/opencode-agent-v2 show HEAD:.opencode/plugin/scripts/gauge.mjs \| wc -l` | `1015` — PASS (core size per spec) |

Before the pin move, the same probe run failed exactly 2 checks — `[32] unit-mock bun:sqlite …` and `[38] fallback: …` — the spec's predicted pair; after the move 0 FAIL. No core defect: the unified core never returned `no-total`/`notAvailable`/a throw against the v1-only fixtures (all 360 green).

Delta vs the spec's DoD-2 wording: the real-tree dry run prints NO `plan:` line for `.opencode/plugin/scripts/gauge.mjs`, because after the swap both sides hold the SAME blob (verified: clone `git rev-parse HEAD:…` == `git hash-object` MAIN's file) so `merge-tree` auto-merges it — a `plan:` line prints only for conflicted paths. The rule is pinned instead in the selftest fixture (a conflicting `gauge.mjs` both sides). The dry run now lists 9 conflicts (the unit-1 record said 7): `TODO.md` and `loop/autorun-2026-10-01_03-27/loop_log.md` joined the set because both trees edited them; both take MAIN by rule.

## Commits

1. `ac7c468` (this worker) — the core swap + the 3 probe pins + the unified gauge_core smoke.
2. the port_sync glob + selftest pins + TODO.md + this handover = THIS commit (its own hash cannot appear here; the planner reads it from `git log`).

Working tree after the commits: clean for the task files; the only residue is host/runtime noise deliberately left unstaged — `opencode.jsonc`, `maintainer/SCRATCH_PAD.md`, `.opencode/temp/compact_budget.json`, `todo_inbox.md`, and two untracked `archive/sessions/compaction_dumps/ses_e9b246…_*` files (never staged per the access rules).

## Deliberately omitted / not done

- No `--apply`, no merge commit, no clone-tree write — the planner/maintainer owns that (spec approval boundary).
- MAIN's probe path-repoint hunks NOT taken from the clone (the #141 `plugin-unported-v1` repoint stays clone-side; `port_sync`'s Rule-A patch applies it post-merge).
- No second gauge module, no fork of the core (spec step 4); `.opencode/plugin-unported-v1/scripts/gauge.mjs` (clone 7-line shim) untouched.
- The clone tree's own files stay as they are until the merge lands them.

## Lesson

A gauge-core swap needs the MOCK catalog pinned before the SQL pins: the probe's unit-mock `get()` decides which catalog the core sees, so a pin that keys only on `tokens.total` silently turns a table-aware core into a `no-total` reader (that is what made check 38, a fallback-chain pin, fail for a schema reason).
