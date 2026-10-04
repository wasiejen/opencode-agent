# Spec — the `state_snapshot` planner init tool (approved `proposals/approved/2026-09-18_p3-state-snapshot-tool.md`)

Worker: `worker_Q3S_slow`. Repo: this one (opencode-agent), stay on the current checkout.
One unit, one commit checkpoint per verified piece (code, then smoke, then probe —
TODO + handover ride the FINAL commit; no commit carries its own hash).

## Goal
New host tool `.opencode/tools/state_snapshot.ts` — the planner's 6–8-call init
sequence as ONE capped digest (no interpretation, deterministic). The tool is
auto-detected by the host (`.opencode/tools/*.ts`, plan39 precedent — NO
registration anywhere; the maintainer confirmed per-role config is not needed).

## Design (the approved proposal's sections, verbatim semantics)
Returns ONE Markdown digest, hard-capped, deterministic:
- Repo root = `context.directory` (the custom-tool context carries it — verified
  live this session), falling back to `process.cwd()`.
- Soft role gate: if `context.agent` is a non-empty string NOT starting with
  `planner` → return exactly `state_snapshot is the planner init digest — not for <agent>` (one line, no digest).
- Sections, IN THIS ORDER, each `## <name>` header:
  1. `## git` — `git log --oneline -20` (execFileSync, args array; on failure the line `(git log failed: <first error line, 80 chars>)`).
  2. `## nap` — the FIRST 40 lines of `agent/handover/handover_planner.md`; if the file has > 40 lines, append the steer line `… (read agent/handover/handover_planner.md for the rest)`.
  3. `## todo` — entry headers = lines matching `^## (?:#\d+|\d+)\.` in `TODO.md`; the count of ALL of them on the first line (`<N> open-entry headers`), then the LAST 15 headers, one line each: `#<id> — <header remainder truncated to 60 chars>` (file order).
  4. `## maintainer` — for each `.md` file in `maintainer/inbox_planner/` (excluding README.md): `<filename>: <first line truncated to 80 chars>` (max 5 files, then `… (+<k> more)`); then `priority.md`: its first 10 lines.
  5. `## proposals` — the `.md` FILENAMES at `proposals/` root (files only, no subfolders), one per line; none → `(none)`.
  6. `## markers` — the ready-made marker sweep (run via bash, EXACTLY the command in the planner prompt §maintainer calls/decisions — the grep with `-e "--main\|--now\|--info\|--todo\|--defer\|--wip\|--comment"` over `.opencode/ agent/ maintainer/ loop/ proposals/ TODO.md README.md WIKI.md` + the `grep -v "_past_priorities\|/done/\|agent_feedback\|nap_direct\|archive/"` filter), capped at 20 lines (then `… (+<k> more)`).
- Empty section content = `(none)` (the empty check is part of the job).
- Total digest hard cap ~60 lines: if the assembled digest exceeds 60 lines, truncate the LAST section's lines first, preserving all six headers + their `… (+<k> more)` steers.
- Tool form: the ctx_gauge.ts sibling shape (default `tool({ description, args, execute })` from `@opencode-ai/plugin`; NO `name` field — the host names by filename). `args: {}` (zero args). The description states: what each section is, the caps, the planner-only steer, and "fire at session start; follow up with full reads only where a section needs a decision".

## Verified facts (measured at spec time, 2026-10-04)
- The custom-tool `context` object carries `directory`, `worktree`, `agent`,
  `sessionID`, `messageID`, `callID` (verified via dev_get_tool_context_contents
  live this session).
- Tool file shape: `.opencode/tools/ctx_gauge.ts` (36 lines — the default-export
  tool() form, no registration).
- Smoke harness: `.opencode/plugin/tests/_smoke_base.mjs` exports
  `freshSandbox`, `loadRepo`, `makeChecker` (see
  `.opencode/plugin/tests/context_trim.smoke.mjs` header for the usage).
- Probe baseline: `handover_probe.mjs` 355/355 (self-annotated tally in its header).
- Existing smokes at baseline (plan9 close, node v24.19.0): auto_resume 165,
  context_trim 40, compact_memory 110, intercept_observer 85, block_transfer
  131 + sandbox 64, context_recovery 17, loop_log 69, submit 31, ctx_gauge 3,
  gauge_core, loop_stats n/a.
- bun is BROKEN (bun.exe missing) — the whole gate runs under plain node.
- `git` works from execFileSync (used live this session).
- The marker-sweep command lives in the planner prompt §maintainer
  calls/decisions — copy it verbatim into the tool (do NOT re-derive the
  pattern list).

## DoD (measurable end states)
1. `.opencode/tools/state_snapshot.ts` lands per the design (zero args, the
   six sections, the caps, the soft role gate, the fail-closed section lines).
2. NEW smoke `.opencode/plugin/tests/state_snapshot.smoke.mjs`: fixture repo
   built in freshSandbox (git init + 1-2 committed bookkeeping files: a TODO.md
   with 3 entry headers, an NAP file with 45 lines, a `maintainer/inbox_planner/`
   with 2 .md files + README.md, `maintainer/priority.md`, a `proposals/` root
   with 2 .md files + 1 subfolder, one `--now` marker line in a fixture .md)
   driven through the tool's execute() with a context stub
   (`{ directory: <fixture>, agent: "planner_Q3S" }`). Pins (>= 12 checks):
   the six section headers present IN ORDER; the git section carries the
   fixture commit subject; the nap section = first 40 lines + the steer line;
   the todo count line + the 15-cap behavior; the maintainer section (the 2
   inbox filenames + priority first lines); the proposals section (filenames
   only — the subfolder excluded); the markers section carries the fixture
   `--now` line and NOT the filtered allow-list paths; an EMPTY inbox fixture
   case → `(none)`; the non-planner agent stub → the exact refusal line; a
   file-missing case (no NAP) → its fail-closed line, no throw.
   Count N reported in the handover; `node <smoke>` exit 0 ALL PASS.
3. NEW probe section in `.opencode/plugin/probes/handover_probe.mjs`:
   (a) the tool file exists + default export has `description` (string,
   non-empty) + `execute` (function); (b) the registration-shape pin: the args
   key list is EMPTY (the exact args keys — same pattern as the
   compact_memory check-86 / context_trim check-346 pins); (c) the description
   carries the planner-only steer (the phrase `planner`). Header tally
   machine-updated 355 → 355+N (N reported).
4. Standard gate GREEN under node: probe 355+N + ALL smokes (the 11 existing at
   baseline + the new state_snapshot) + no ruff/pytest in this repo (the gate =
   probe + smokes).
5. NO behavior change to any existing tool/plugin/probe (the existing smokes
   stay byte-green; if ANY existing pin broke, stop and report — do not
   re-pin outside this spec's surfaces).
6. Handover `agent/handover/handover_task_to_planner.md`: what landed, the
   measured N counts (smoke + probe), the gate numbers (worker-measured), the
   commit hashes of the code/smoke/probe checkpoints, TODO.md + any
   discrepancies appended. TODO.md entry: the planner curates the close note
   (no TODO.md edits by the worker beyond the handover — the planner owns the
   status line).

## DO-NOT-TOUCH
- `maintainer/**` (the tool READS inbox/priority only — never writes),
  `agent/prompts/**` (the prompt §Initialization edit is the PLANNER's, a
  separate commit after this lands — NOT this task), `AGENTS.md`,
  `.opencode/plugin/**` (except the two NEW/extended test+probe files named
  above), any existing tool under `.opencode/tools/`, the FST repo,
  `opencode.jsonc` (no registration — the host auto-detects).

## Suggested scope (files)
- `.opencode/tools/state_snapshot.ts` (new)
- `.opencode/plugin/tests/state_snapshot.smoke.mjs` (new)
- `.opencode/plugin/probes/handover_probe.mjs` (new section + tally)
- `agent/handover/handover_task_to_planner.md` (final commit)
