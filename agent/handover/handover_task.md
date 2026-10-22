# Task spec — TODO #137 order 2: auto_resume ported to v2 (plugin-driven)

Assigned worker: `worker_FlashNext` (model `llama-swap/Qwen3.8-NEXT-Coder`, same as the launching planner). Work happens in the v2 clone: `C:/Users/Wasiejen/Repos/opencode-agent-v2` (branch `migration`). Stay on the current checkout.

## Goal
A v2 plugin `.opencode/plugins/auto_resume.ts` carrying the v1 plugin's loop legs onto the v2 API, so the autorun loop works on a v2 host without the CLI: successor spawn, SELF-compaction at the limit, action-line routing, the silent-limit-stop detector, and the counter/lineage + evidence logs.

## Design of record
`.opencode/plugin/auto_resume.ts` (MAIN, v1, also present inert at clone `plugin-unported-v1/auto_resume.ts`). Do NOT read the whole file: read its header block (~first 120 lines) plus targeted greps — `spawnPlanner`, `limitStopCheck`, `actionRe`, `LINEAGE_MAX_DEPTH`, `saturationThreshold`, `ctx.log`, `spawn=`. The v1 unit map: UNIT 1 event log + one-shot surface probe; UNIT 2 the PASSIVE ctx-line suffix (ALREADY ported — it lives in clone `.opencode/plugins/context_family.ts`; do not re-port); UNIT 3 the successor spawn (create + ONE queued prompt + the spawn marker/ident + `spawn=`/`spawn-fail=` lines); UNIT 4 the restart / recovery-continue branch + the `action:` line reading; the #109 silent-limit-stop detector (5-clause signature, `limitStopCheck` at ~L1536, idle threshold ~60_000, `0.99 * window`); the loop-iteration counter + lineage legs.

## Verified facts (measured at spec time, this planner session, ITER-28)
- v2 plugin session domain = `Pick<SessionApi, "create" | "get" | "remove" | "switchAgent" | "switchModel" | "prompt" | "generate" | "command" | "compact" | "synthetic" | "interrupt" | "update" | "move" | "wait" | "context">` + `hook` (fork `packages/plugin/src/promise/session.ts:137-176`).
- `SessionCreateInput` (fork `packages/client/src/promise/generated/types.ts:3064-3082`): `id?: { id?, parentID?, title?, agent?, model?: { id, providerID, variant? }, location?: { directory }, metadata?, permissions?: [{action,resource,effect:"allow"|"deny"|"ask"}] }` + a top-level `parentID?`. So the v1 spawn-with-agent + title-ident mechanism ports directly; `create` is OPTIONAL-input (`input?`).
- v2 hooks available for this shape: `session.hook("prompt" | "context" | "compaction" | "retry" | …)` (same file, `SessionHooks`); the status/idle event names live in fork `packages/schema/src/session-status-event.ts` — confirm the exact event key with ONE grep there; the v1 UNIT 1 event log lines in `.opencode/temp/auto_resume.log` show the v1 keys as the reference set.
- Plugin auto-discovery roots = `["plugin","plugins"]` (fork `packages/core/src/plugin/source-directory.ts:7`) → the new plugin LOADS at the next v2 host start with NO config edit. Therefore: every leg must FAIL-OPEN (a throw inside a hook must never reach the host), and no `opencode.jsonc` change is expected.
- Clone gate baselines at HEAD `90d6d6c`: probe `node .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360 PASS`; 12 v1 node smokes green (clone root); 8 v2 bun smokes `SMOKE: ALL PASS` (from `.opencode/plugins/tests`); tsgo `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p .opencode/plugins/tests/tsconfig.json` exit 0. Re-run fresh; the trio EBUSY flake is fixed (TODO #144), so red = real.
- The v2 ports so far (same tree, same registration shape): `plugins/compact_memory.ts` (dispatch via `ctx.session.compact` fire-and-forget + the `session.compaction.ended` evidence gate), `plugins/context_family.ts` (the `context` hook + `session.synthetic(delivery:"queue")`), `plugins/loop_log.ts`/`submit.ts` (`ctx.tool.transform` + `editor.add`). Root path rides the setup capture `ctx.location.directory` (the v1 `context.directory` has no v2 carrier) — the same deviation applies here.
- #147 (`agent/scripts/git/port_sync.cjs`, MAIN) exists; the real sync (`--apply`) is HELD by the maintainer until this unit lands. Do not run it.

## Definition of done
1. NEW clone file `.opencode/plugins/auto_resume.ts` (`{ id: "auto_resume", setup(ctx) }`, tool/event legs per the v1 unit map, minus UNIT 2 which is already in `context_family.ts`).
2. New/renamed evidence log shapes byte-carried where v1 has them (`auto_resume.log` event lines `<ISO time> event=<type> sid=<sessionID> <key fields>`, `spawn=` / `spawn-fail=`, the action-line decision lines, the #109 detector fire line → the loop_log `-WARNING` line), plus a v2-in-file comment block listing every deviation (v1-only legs with no v2 carrier, e.g. `extra.model.id`, `session?.id`).
3. `tests/auto_resume.smoke.mjs` under `.opencode/plugins/tests/` with pins per leg (spawn input shape incl. `title`/`agent`/`model` fields, the SELF-compact gate thresholds, the once-per-overflow claim guard, the detector's clauses, fail-open on a throwing hook, the counter/lineage suffix) → `SMOKE: ALL PASS`; `tsconfig.json` `files` += the new plugin; the tests README gains a section.
4. Gate green after the change, measured by the worker: probe `360/360`, the 9 v2 bun smokes `ALL PASS` (8 existing + the new one), the 12 v1 node smokes untouched-green, tsgo exit 0, clone tree clean.
5. Checkpoint commits per verified unit (code only); the `TODO.md` #137 status line (order 2 status + hashes of the code commits, never its own hash) and `agent/handover/handover_task_to_planner.md` ride the FINAL commit.

## Suggested scope (non-exhaustive)
`.opencode/plugins/auto_resume.ts`, `.opencode/plugins/tests/auto_resume.smoke.mjs`, `.opencode/plugins/tests/tsconfig.json`, `.opencode/plugins/tests/README.md`, `TODO.md`, `agent/handover/handover_task_to_planner.md`.

## DO-NOT touch
- `opencode.jsonc` in either tree (maintainer-owned; no registration needed — auto-discovery).
- `.opencode/plugins/context_family.ts`, `compact_memory.ts`, `context_trim_v2.ts`, `block_transfer.ts`, `loop_log.ts`, `submit.ts`, `ctx_gauge.ts`, `state_snapshot.ts`, `dev_get_tool_context_contents.ts`, `intercept_observer.ts` — read-only reference only.
- MAIN tree files (the planner's own tree) — nothing in it, including `agent/readme/**`, `agent/scripts/**`, `loop/**`. This unit's `TODO.md` status line and handover file are written in the CLONE tree (this task's tree); the MAIN-side bookkeeping is the planner's own commit.
- `maintainer/**`, `AGENTS.md`. Never `git push`; never `port_sync.cjs --apply`; never start the v2 host or any request to the inference backend; no deletion of the inert v1 estate.

## Tree and path discipline (read first)
The launching planner's cwd is the MAIN tree. Every git operation for this unit runs with `workdir` = `C:/Users/Wasiejen/Repos/opencode-agent-v2` (or `git -C <that path>`), and every file read/edit uses absolute clone paths (`C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/plugins/…`) — relative paths resolve against the session CWD and would hit the wrong tree.

## Expectation on discrepancies
No new TODO entry is expected; if a v1 leg turns out to have no v2 carrier at all, log it in-file AND append a numbered `TODO.md` entry rather than inventing a substitute.
