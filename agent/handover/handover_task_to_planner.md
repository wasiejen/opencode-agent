# Worker handover — TODO #141: the v2 clone's standard gate made green (clone gate unit)

Status: LANDED (the entry hash is stamped by the planner, per spec). Task complete in both trees; no live acceptance required (node-only gate).
Previous unit B handover content is preserved in git history; this file is superseded per the channel contract.

## Task and scope
Spec `agent/handover/handover_task.md` (#141): clone `C:/Users/Wasiejen/Repos/opencode-agent-v2` on branch `migration` — probe + 12 smokes under `.opencode/plugin/tests/` repointed from the missing `.opencode/plugin/<module>.ts` paths to the clone's actual subjects (`.opencode/plugin-unported-v1/` v1 set; existing paths kept where the file exists). No v1 module, tool, or v2-port source edits. Main repo: TODO.md #141 status line + this summary only.

## Files changed
Clone repo (`migration`), 2 checkpoint commits:
- `652ee6f` — `.opencode/plugin/probes/handover_probe.mjs`: 5 constants repointed (PLUGIN_TS ctx_watchdog, RC_TS context_recovery, QC_PLUGIN_TS compact_memory, OBS_TS+OBS_CORE_TS intercept pair, CORE_TS compaction_core → `plugin-unported-v1/`); TOOL_TS (deactivated `compact_memory_v1.ts`) KEPT — the file EXISTS at that path in the clone. Check 32 (bun:sqlite adapter) re-pinned to the clone's table-aware gauge twin: 7 ordered prepares (catalog x2, `session_v2`-first newest x2, time_updated compare x2, tokens.total step LAST), same construct-opts/PRAGMA/close pins — clone-twin precedent = unit A check-113 re-pin. New file `.opencode/plugin-unported-v1/scripts/gauge.mjs`: 1-line `export *` re-export shim resolving the v1 modules' byte-unchanged relative `./scripts/gauge.mjs` imports to the clone's gauge-core home (ONE implementation, same module instance for probe/tool/smoke loads).
- `47aa082` — 4 smoke files repointed: `auto_resume.smoke.mjs` (load site + the 2 child-script embedded import strings), `compact_memory.smoke.mjs` (load site + reimport join + source-read join; the `plugin/scripts/gauge.mjs` loads kept), `context_recovery.smoke.mjs` (load site), `intercept_observer.smoke.mjs` (plugin+core loads → v1 pair; the `.opencode/plugins/` v2 twin has NO named exports, so it is NOT this suite's subject).

Main repo — final commit: `TODO.md` (#141 LANDED status line) + this file.

## Behavior changed
None in product code — gate infrastructure only, clone side. The clone probe now imports the same v1 subjects from their new home; check 32 pins the clone gauge twin's prepare sequence (the clone core is the unit-1 table-aware port). No probe section excluded → header annotation stays `... hygiene=6 → "PROBE handover: 360/360 PASS"` (agreement self-verified by the totals).

## Measured verification (clone, run with plain node from the clone root)
- `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360 PASS` exit 0 (t=360 = annotation sums).
- 12 smokes, readouts (each exit 0):
  - AUTO_RESUME_SMOKE: ALL PASS (170/170)
  - BT-SANDBOX-SMOKE: ALL PASS (76/76)
  - BLOCK_TRANSFER_SMOKE: ALL PASS (131/131)
  - COMPACT_MEMORY_SMOKE: ALL PASS (110/110)
  - CONTEXT_RECOVERY_SMOKE: ALL PASS (17/17)
  - CONTEXT_TRIM_SMOKE: ALL PASS (40/40)
  - CTX_GAUGE_SMOKE: ALL PASS (3/3)
  - GAUGE_CORE_SMOKE: ALL PASS — custom verdict line prints no n/n; measured 32 PASS lines, 0 FAIL
  - INTERCEPT_OBSERVER_SMOKE: ALL PASS (85/85)
  - LOOP_LOG_SMOKE: ALL PASS (69/69)
  - STATE_SNAPSHOT_SMOKE: ALL PASS (20/20)
  - SUBMIT_SMOKE: ALL PASS (31/31)
- Main-repo gate unaffected: `node .opencode/plugin/probes/handover_probe.mjs` FROM THE MAIN REPO → `PROBE handover: 360/360 PASS` exit 0 (run after all commits; see below — final run PASS).
- Exact verification command: `node .opencode/plugin/probes/handover_probe.mjs` (per repo root) + `node .opencode/plugin/tests/<name>.smoke.mjs` ×12 — PASS.
- Working-tree status: clone `clean` (both commits landed, `git status` empty); main repo clean apart from pre-existing untracked `archive/sessions/compaction_dumps/*` (left untouched).

## Commits
- Clone `migration`: `652ee6f` (probe+shim), `47aa082` (smokes).
- Main repo: the final commit carries `TODO.md` + this handover (hash = this commit; recorded above the fold, planner stamps #141 with the clone hashes).

## Discrepancies / notes (recorded in the TODO.md status line, not a new entry)
- Spec baseline said "11 smokes 0-PASS"; measured: 8 were already green (their subjects exist in the clone); only the 4 plugin-module smokes were red.
- Clone `intercept_observer.smoke` 85/85 is the pre-unit-B pair (clone v1 smoke + pre-unit-B v1 core — consistent pair); the unit B rule lives in the clone's v2 twin (49/49, out of this unit's scope).
- TODO #141 entry left OPEN (planner curation + hash stamping).

## Deliberately omitted
- Clone v2 port `plugins/` + its tests (out of unit; byte-untouched).
- Any main-repo `.opencode/**` change; git history ops; `git add -A`.
- gauge_core smoke verdict-line n/n format improvement (its own harness; count measured externally instead).

## Blockers
None.

Lessons: a clone twin re-pin mirrors the clone module's OWN sequence (unit A precedent) — check the clone's module diff before excluding a section; and v1 folders moved away from their scripts home need a re-export shim, not module edits, to keep relative imports byte-unchanged.

Friction: submitted via the feedback/knowledge channels (spec baseline miscount; write-tool blocked until mkdir of the clone subdir).
