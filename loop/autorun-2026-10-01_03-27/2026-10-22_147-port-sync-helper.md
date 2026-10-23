# Task spec — TODO #147: the port-sync helper (unit 1: build + verify; NO real apply)

Assigned worker: `worker_FlashNext` (model `llama-swap/Qwen3.8-NEXT-Coder`, same as the launching planner — cache rule).
Work happens in the MAIN tree: `C:/Users/Wasiejen/Repos/opencode-agent` (branch `main`). Stay on the current checkout.

## Goal
A node script that syncs MAIN's `main` branch into the v2 clone's `migration` branch by MERGE — previewable (dry run default), boundary-resolved by rule, gate-verified, and safe by default (it never auto-commits the merge in this unit).

## Definition of done
1. NEW `agent/scripts/git/port_sync.cjs` (plain node, no deps, POSIX/Git-Bash safe, Windows paths OK) with:
   - `--dry-run` = DEFAULT: fetch into the clone, print the merge-tree conflict set + the per-path resolution plan + the exact commands, and mutate NOTHING (see DoD 3).
   - `--apply`: fetch → `git merge --no-commit FETCH_HEAD` → apply the resolutions of DoD 4 → the discovery guard → run the clone gate → on any FAIL print `gate: FAIL` + `git merge --abort`; on green print `gate: PASS` + `merge staged, not committed` and STOP (no commit — the first real apply is the maintainer's act).
   - `--main <path>` / `--clone <path>` default to the two real trees; `--no-gate` skips the gate step (self-test uses it).
2. NEW `agent/scripts/git/port_sync.selftest.cjs` printing `SELFTEST: ALL PASS` (>= 25 checks), run against a THROWAWAY git repo pair under the scratchpad `C:/Users/Wasiejen/AppData/Local/Temp/opencode/port_sync_selftest/` (never `$TMP` — TODO #146: this host's Git-Bash has `$TMP=/tmp`), covering: the two resolution glob sets, `opencode.jsonc` never touched, the discovery guard MOVING a stray `.opencode/plugin/x.ts` into `.opencode/plugin-unported-v1/` with a WARNING line, and the FAIL path calling `git merge --abort`.
3. Dry-run ZERO-mutation proof printed: `git -C <tree> status --porcelain` + `git -C <tree> rev-parse HEAD` recorded before and after for BOTH trees, diffed to empty.
4. Resolution boundary, as ruled by the maintainer 2026-10-22: take MAIN (`--theirs`) for `.opencode/tools/**`, `.opencode/plugin/probes/**`, `.opencode/plugin/tests/**`, `agent/handover/**`, `loop/**`, `TODO.md`, `agent/knowledge/**`, `agent/memory/**`, `agent/readme/**`, `agent/scripts/**`; NEVER touch `opencode.jsonc` (on conflict print `config: kept branch-local (host-owned)`); everything else merges normally.
5. A `agent/scripts/README.md` row + a line in `agent/scripts/git/README.md` (purpose / allowed contents / excluded / usage, <= 20 lines).
6. Checkpoint commits per verified unit (code only); `TODO.md` status line + `agent/handover/handover_task_to_planner.md` ride the FINAL commit, recording the code commits' hashes (never the final commit's own hash).

## Verified facts (measured at spec time, this planner session, ITER-28)
- One repo, two trees: MAIN `C:/Users/Wasiejen/Repos/opencode-agent` branch `main`; clone `C:/Users/Wasiejen/Repos/opencode-agent-v2` branch `migration` (HEAD `90d6d6c`, tree clean at measurement). Same `origin` (`github.com/wasiejen/opencode-agent`), merge-base `3297f1b`, MAIN ahead 67 commits, the clone's `origin/main` at `3bdfbd6`.
- Dry-run shape measured read-only in the clone: `git fetch -q /c/Users/Wasiejen/Repos/opencode-agent main` then `git merge-tree --write-tree --name-only migration FETCH_HEAD` → 27 files touched, exactly 7 conflicts: `.opencode/plugin/probes/handover_probe.mjs`, `.opencode/plugin/tests/block_transfer.smoke.mjs`, `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs`, `.opencode/plugin/tests/gauge_core.smoke.mjs`, `.opencode/tools/block_transfer.ts`, `agent/handover/handover_task_to_planner.md`, `opencode.jsonc`.
- The v1 plugins were MOVED not deleted: clone commit `5516a83 move unported v1 plugins out of the v2 plugin discovery`, recorded as `R100` renames into `.opencode/plugin-unported-v1/` → MAIN-side edits follow into that path; a merge re-adds nothing at `.opencode/plugin/*.ts`.
- v2 plugin discovery roots = `["plugin","plugins"]` — fork `packages/core/src/plugin/source-directory.ts` line 7 (`export const names = ["plugin", "plugins"] as const`). `.opencode/tools/` is NOT a discovery root → inert in v2.
- Clone gate commands (from `agent/readme/repo_commands.md` §Run/test): probe `node .opencode/plugin/probes/handover_probe.mjs` (clone root) → expect `PROBE handover: 360/360 PASS`; v1 smokes `node .opencode/plugin/tests/<f>.smoke.mjs` (clone root, 12 files, baseline 12/12 green per plan22); v2 smokes from `.opencode/plugins/tests`: `bun run <f>.smoke.mjs` (8 files) → each `SMOKE: ALL PASS`; typecheck `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p .opencode/plugins/tests/tsconfig.json` (clone root) → exit 0, no output. Re-run the gate fresh; do not restate these counts as yours.
- The teardown-EBUSY trio flake is FIXED (TODO #144, MAIN + clone `4045331`), so a red smoke means a real regression.

## Suggested scope (non-exhaustive)
`agent/scripts/git/port_sync.cjs`, `agent/scripts/git/README.md`, `agent/scripts/git/port_sync.selftest.cjs`, `agent/scripts/README.md`, `TODO.md` (#147 status), `agent/handover/handover_task_to_planner.md`.

## DO-NOT touch
- `opencode.jsonc` in EITHER tree (maintainer-owned, live host config).
- `maintainer/**`, `AGENTS.md`, `agent/readme/**` (the last two are maintainer-owned docs; flag any discrepancy in the handover instead).
- MAIN's working tree/branch: the script fetches INTO the clone only; no merge, checkout, commit, or `git add` in MAIN.
- `.opencode/plugins/**` in the clone (the v2 estate), any `.opencode/plugin-unported-v1/**` content edits, any file deletion.
- Never `git push`, never `git commit` of a merge in this unit, never a live host start, never a request to the inference backend.
- Do NOT run `--apply` against the two real trees in this unit; the self-test scratch pair is the test surface.

## Expectation on discrepancies
No new TODO entry is expected; if the script's boundary rule turns out to conflict with a pin or a doc line, append a numbered `TODO.md` entry rather than silently widening scope.
