# Worker summary — TODO #147 unit 1: build + verify the port-sync helper (no real apply)

Worker: `worker_FlashNext` (session `ses_e9acefcfbffewfyOO0y7VCJsfP`), MAIN tree `C:/Users/Wasiejen/Repos/opencode-agent`, branch `main` (unchanged checkout).
Status: DONE (unit 1).

## Scope
Built `agent/scripts/git/port_sync.cjs` (MAIN `main` → clone `migration` sync by MERGE, dry-run default), its self-test, and the README rows. No `--apply` against the real trees; the scratch pair under `C:/Users/Wasiejen/AppData/Local/Temp/opencode/port_sync_selftest/` was the only apply surface.

## Files changed
| file | change |
|---|---|
| `agent/scripts/git/port_sync.cjs` | NEW — dry run (conflict set + per-path resolution plan + exact commands + `zero-mutation: PROVEN` for BOTH trees) / `--apply` (`merge --no-commit FETCH_HEAD` → `--theirs` resolutions → `opencode.jsonc` kept branch-local → discovery guard → clone gate → `gate: FAIL` + `git merge --abort`, or `gate: PASS` + `merge staged, not committed`) / `--main` `--clone` path overrides / `--no-gate`. Refuses on branch mismatch or a dirty clone. |
| `agent/scripts/git/port_sync.selftest.cjs` | NEW — throwaway git repo pair (base commit → clone BEFORE the MAIN-side commit → diverge both sides), 84 checks. |
| `agent/scripts/git/README.md` | NEW — purpose / what it does / excluded / env overrides / usage (11 lines, the <=20 cap) |
| `agent/scripts/README.md` | categories row for `git/`. |

## Behavior added
- Boundary as ruled: take MAIN (`--theirs`) for `.opencode/tools/`, `.opencode/plugin/probes/`, `.opencode/plugin/tests/`, `agent/handover/`, `loop/`, `TODO.md` (exact), `agent/knowledge/`, `agent/memory/`, `agent/readme/`, `agent/scripts/`; everything else git's own 3-way; `opencode.jsonc` never written (resolved `checkout --ours` + `add`, so its bytes stay byte-identical to the clone's own and MAIN's side never lands — conflict markers stripped).
- Discovery guard fires only on `.ts` NEWLY present under `.opencode/plugin/` in the merge result (computed from the `merge-tree --write-tree` tree minus the clone branch tree) → moved to `.opencode/plugin-unported-v1/` (substructure preserved) + WARNING line. Against the real trees the guard preview is empty — the clone's `R100` renames already carry MAIN's edits, matching the verified "a merge re-adds nothing at `.opencode/plugin/*.ts`".
- Gate (skippable by `--no-gate`): probe (PASS needs exit 0 + `PROBE handover: n/n PASS`), v1 smokes by directory enumeration (`node`, exit 0), v2 smokes (`bun run`, PASS needs `SMOKE: ALL PASS`), tsgo typecheck (exit 0, no output). Missing suites/`tsconfig`/`tsgo`/`bun` print SKIP, never fake PASS.

## Measured verification (exact commands)
| command | result |
|---|---|
| `node agent/scripts/git/port_sync.selftest.cjs` | PASS — exit 0, `checks: 84 pass=84 fail=0`, `SELFTEST: ALL PASS` |
| `node agent/scripts/git/port_sync.cjs` (real-tree dry run) | PASS — exit 0, `merge-tree: 7 conflict(s)`, 6 `take MAIN (--theirs)` plan lines, `config: kept branch-local (host-owned)`, `zero-mutation: PROVEN (both trees: HEAD + status --porcelain identical)` |
| `node .opencode/plugin/probes/handover_probe.mjs` (MAIN, after the change) | PASS — `PROBE handover: 360/360 PASS` (matches this tree's annotation; do not restate as the clone's) |
| `git -C C:/Users/Wasiejen/Repos/opencode-agent-v2 rev-parse --short HEAD` | `90d6d6c` before and after the dry run (clone untouched, status clean) |

Self-test coverage of the DoD-mandated points: both resolution glob sets (10 positive + 5 negative rule checks, incl. `TODO.md.bak`, `agent/handovers/`, `.opencode/plugin/x.ts` NOT take-main); `opencode.jsonc` never touched (byte-identical, no conflict markers, MAIN side absent); guard moves `.opencode/plugin/x.ts` → `.opencode/plugin-unported-v1/x.ts` with the WARNING line, leaves pre-existing plugin `.ts` and the `.opencode/plugins/**` estate alone; gate-FAIL path prints `gate: FAIL` + echoes `cmd: git -C <clone> merge --abort` and leaves the clone clean at HEAD with all merged content reverted (exit 1).

## Commits
- `dfebc3a` — `agent/scripts/git/port_sync.cjs` + `agent/scripts/git/README.md` + the `agent/scripts/README.md` row.
- `db91bb3` — `agent/scripts/git/port_sync.selftest.cjs` (green self-test).
- Final commit (this file + the TODO #147 status line + the loop-log DONE line + the `submit` inbox lanes) — its hash lives in `git log`, not in this file.
Committed together: `TODO.md`, `agent/handover/handover_task_to_planner.md`, `loop/autorun-2026-10-01_03-27/loop_log.md`, and the inbox lanes `agent/agent_feedback.md`, `agent/knowledge/knowledge_inbox.md`, `agent/memory/worker/memory_inbox.md` (my `submit` entries; those two lanes also carried other sessions' uncommitted appends — preserved, not authored by me).
Working tree after: `dirty` — `agent/agent_ideas.md` (pre-existing, not mine), `maintainer/SCRATCH_PAD.md` + `opencode.jsonc` (maintainer-owned, never staged). No code file left dirty.

## TODO / inbox
- `TODO.md` #147: appended one status line inside the existing entry (unit 1 BUILT + GREEN, the two deltas below). No new entry.
- Deltas recorded (not silently widened): #147's acceptance line says `--ours` for `agent/handover/**`+`loop/**`, and a green gate "commit the merge"; the ruled boundary + this spec say `--theirs` and stage-only. The script follows the ruled spec. The printed config line is `config: kept branch-local (host-owned)` per DoD 4, not `config: skipped (host-local)`.
- Friction + lesson submitted through the `submit` tool (see below).

## Omitted deliberately
- The first real `--apply` and the clone-gate run against the real v2 estate (spec DO-NOT-touch: the maintainer's act).
- The `agent/readme/repo_commands.md` pointer to the new script — maintainer-owned, flagged instead: that file's §Run/test section still describes only the MAIN-tree gate; the clone gate command set lives in the task spec.
- Merge-commit / push paths: the script has none.

## Blockers
None.

## Lesson
Node `spawnSync` cannot exec this host's PATH shims: `bun` (extensionless) → ENOENT, `bun.cmd` → EINVAL. The script resolves the real exe `C:/Users/Wasiejen/AppData/Roaming/npm/node_modules/bun/bin/bun.exe` (env `BUN_EXE` override) and falls back to PATH `bun`/`bun.exe`, printing SKIP rather than a fake PASS.
