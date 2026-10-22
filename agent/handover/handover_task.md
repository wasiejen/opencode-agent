# Spec — TODO #141: make the v2 clone's standard gate green (clone gate unit)

## Goal
The v2 clone `C:/Users/Wasiejen/Repos/opencode-agent-v2` (branch `migration`) gains a green
standard gate: `node .opencode/plugin/probes/handover_probe.mjs` prints
`PROBE handover: <t>/<t> PASS` with the header annotation agreeing with the section sums,
and every smoke under `.opencode/plugin/tests/` reports its real check count (no 0-PASS suite).

## Verified facts (measured at spec authoring, 2026-10-22, planner-13)
- Clone baseline: `node .opencode/plugin/probes/handover_probe.mjs` → exit 1, `ERR_MODULE_NOT_FOUND`
  on `file:///…/.opencode/plugin/ctx_watchdog.ts` (0 checks run). The 11 node smokes in
  `.opencode/plugin/tests/` each report 0 PASS at baseline (same class: they import v1 plugin
  modules from `.opencode/plugin/<name>.ts`, which does NOT exist in the clone).
- Clone layout (measured): the v1 plugin modules are INTACT under `.opencode/plugin-unported-v1/`
  (`auto_resume.ts`, `compact_memory.ts`, `compaction_core.ts`, `context_recovery.ts`,
  `ctx_watchdog.ts`, `intercept_observer.ts`, `intercept_observer_core.ts` + `.md` docs);
  `.opencode/plugins/` = the v2 port (`intercept_observer.ts` + `tests/`, smoke 49/49 green);
  `.opencode/tools/` = present with `block_transfer.ts`, `context_trim.ts`, `ctx_gauge.ts`,
  `loop_log.ts`, `session_info.ts`, `state_snapshot.ts`, `submit.ts`, `dev_get_tool_context_contents.ts`,
  `deactivated/`; `.opencode/plugin/scripts/gauge.mjs` + `peek.mjs` present.
- Probe module-path constants (grep anchors in `.opencode/plugin/probes/handover_probe.mjs`):
  `PLUGIN_TS = join(REPO_ROOT, ".opencode", "plugin", "ctx_watchdog.ts")`;
  `TOOL_TS = …"plugin","deactivated","compact_memory_v1.ts"`; `RC_TS = …"plugin","context_recovery.ts"`;
  `QC_PLUGIN_TS = …"plugin","compact_memory.ts"`; `OBS_TS` / `OBS_CORE_TS = …"plugin","intercept_observer(.core).ts"`;
  the `.opencode/tools/*` constants (`CG_TOOL_TS`, `BT_TOOL_TS`, `LL_TOOL_TS`, …) already match the clone.
- The smokes embed the same pattern inline, e.g. `.opencode/plugin/tests/auto_resume.smoke.mjs` lines
  2185/2283: `'const mod = await import(pathToFileURL(path.join(REPO, ".opencode/plugin/auto_resume.ts")).href);'`
  (grep `".opencode/plugin/` per smoke file to find every site; string forms vary — some use
  `path.join(REPO, ".opencode", "plugin", …)`, some a single joined string).
- The clone's v2 intercept smoke (`plugins/tests/smoke.mjs`, 49/49) is OUT of this unit — untouched.

## Definition of done
1. Probe: every v1-plugin module constant resolves to the clone's actual location
   (`.opencode/plugin-unported-v1/<file>.ts` for the inert v1 set; v2 twin under `.opencode/plugins/`
   where one exists and the section's subject is the v2 port; `.opencode/deactivated` paths kept if the file exists).
   Any import that has NO clone subject is removed/excluded WITH a one-line comment naming the reason.
2. `node .opencode/plugin/probes/handover_probe.mjs` in the clone prints `PROBE handover: <t>/<t> PASS` (t > 0),
   header annotation = section sums, zero FAIL.
3. Each of the 12 smokes under `.opencode/plugin/tests/` runs green with real counts (run each; record the
   readout line per suite in the handover).
4. Checkpoint commits per verified unit (probe green / smokes green may be separate commits);
   the FINAL commit carries `TODO.md` status (main repo) + `agent/handover/handover_task_to_planner.md` (main repo).

## Approval boundary
Pre-approved: test/gate infrastructure in the clone (the maintainer allowed clone edits on `migration`;
the clone gate is agent-facing). No product/plugin/tool SOURCE changes: the v1 modules under
`.opencode/plugin-unported-v1/` and `.opencode/tools/*.ts` and `.opencode/plugins/**` stay byte-untouched
(exceptions only if a check itself embeds a stale path constant that must move — keep semantics identical).
DO NOT touch: main-repo `.opencode/**`, `maintainer/**`, `opencode.jsonc` (either repo), the clone
`plugins/` port + its tests, git history (no rebase/amend), never `git add -A` (named paths only).

## Suggested scope (non-exhaustive)
- clone `.opencode/plugin/probes/handover_probe.mjs` (module constants + the header section-sum annotation)
- clone `.opencode/plugin/tests/*.smoke.mjs` (11 files + `_smoke_base.mjs` if it holds the path shape)
- main `TODO.md` (#141 status line, LANDED form: hash recorded by the planner later)
- main `agent/handover/handover_task_to_planner.md`

## Known traps
- A probe section whose fixture writes/reads `.opencode/temp/` paths is fine (the clone has its own temp).
- Section sums: if a check is excluded (no subject), the header tally line must move with it — never leave a
  stale annotation (the probe self-checks agreement).
- The clone repo's `git status` must stay clean apart from this unit's files; the unit runs on branch `migration`.
- Do NOT run the clone's opencode host (backend-slot rule); node-only verification.
- Do NOT self-compact at task close (worker §Context budget guard: only on a live nudge / 80% / stop line).
