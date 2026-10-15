# Worker handover — Unit A: intercept observer ported to v2 (TODO #137)

Status: DONE
Baseline: clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`, branch `migration`
(created by the worker from `main` at 3297f1b — the branch did not exist yet).

## Task and scope

Port the surviving v1 intercept-observer feature set to a v2-native plugin in
the clone (task spec: main repo `agent/handover/handover_task.md`):

- observation log to `.opencode/temp/intercept.log` (path-anomaly incl. the C
  doubled-segment detector with the JSON-escaped `\\` collapse, the
  out-of-sandbox note, the `intercept-error` line, write/edit journals)
- read-scoped fuzzy (v2 `read` `path`, d<=2 + gap rule, R7 segment matcher +
  R7.1 dedup collapse, strict existence gate)
- write-scope fuzzy (v2 `edit` `path`, d<=1, same gate; the v2 `write` tool is
  EXCLUDED — M1)
- B' hard-stop (effective write/edit path outside every allowed root AND
  containing directory missing -> ONE `intercept-error` line, then throw)
- R8 typed-path redirect (v2 field names, fail-closed pass-through, the
  redirect suppresses the duplicate out-of-sandbox note)

DROPPED per maintainer ruling: the `[left:right]` PAIR channels (read 2a,
write 3a, git-refs 3c) and the numword class — no logic, no log lines, no
verdicts. Also outside the surviving set: the (5) edit-oldString fuzzy
channel, the (6) after-hook enrichment, the (4) R6 payload journal's
block_transfer part, the #102 bash `command`-string span redirect (see
"Deliberately omitted work").

## Files changed (all in the clone)

- `.opencode/plugins/intercept_observer.ts` — NEW v2 plugin (single file,
  default export `{ id, setup }`, type-only `import type { Plugin } from
  "@opencode/plugin"` — erased at runtime; the clone has no node_modules).
  Registers the `execute.before` hook; mutates `event.input` (v2 core
  executes the post-hook value — fork `packages/core/src/tool.ts:271-280`).
  The deliberate B' throw is a `HardStopError`; the adapter wraps the
  rejection into `Tool.Error` (fork `packages/plugin/src/promise/adapter.ts:632-641`).
- `opencode.jsonc` — the minimal v2 `plugins` entry (`./plugins/intercept_observer.ts`)
  with a comment: v2 file-path config entries are dropped with a warning
  (fork `packages/core/src/config/plugin/source.ts:151-153`); the plugin
  loads via DIRECTORY auto-discovery of `.opencode/plugins/`. Documented,
  not load-bearing.
- `.opencode/plugins/tests/` — `smoke.mjs` (46 checks, mock ctx),
  `tsconfig.json` + `typecheck.ts` (tsgo harness mapping `@opencode/plugin`
  to the fork's source), `README.md` (run instructions).
- `agent/scripts/v2_intercept_acceptance.sh` + `agent/scripts/v2_intercept/acceptance/file-4.txt`
  — the maintainer's live-acceptance fixture + script (see below).
- `agent/handover/handover_task_to_planner.md` — this file.

## Unit 1 (committed earlier: 5516a83)

The unported v1 plugin files are inert — all 7 top-level `.ts` modules +
5 doc twins moved out of `.opencode/plugin/` into
`.opencode/plugin-unported-v1/` (mechanism + the 6-vs-7 count documented in
that folder's README). No v1 file remains in the v2 discovery dirs
(`.opencode/plugin/`, `.opencode/plugins/`); the remaining subdirs there
carry no `server.ts`/`index.ts` entrypoint and are dropped at scan time
(fork `packages/core/src/config/plugin/source.ts:164-167`).

## Behavior changed

None observable for the maintainer until the clone is booted with v2 — and
it was not booted (never run the clone's `opencode.exe`; task boundary).
Once booted under v2: the surviving intercept channels observe/correct
read/edit/write tool calls as above; the v1 plugin no longer loads (inert),
so there is no double observation.

## Measured verification

1. Bun smoke (committed copy, run from the clone):
   command: `cd .opencode/plugins/tests && bun run smoke.mjs`
   result: PASS — `SMOKE: ALL PASS` (46/46 checks, 19 log lines). Covers all
   surviving channels (read fuzzy d<=2 + gap + dedup + existence gate;
   edit fuzzy d<=1; `write` exclusion; R8 redirect + fail-closed + note
   suppression; C doubled-segment with the JSON-escaped `\\` collapse;
   out-of-sandbox note; B' hard-stop throw + exactly one intercept-error
   line; in-root new-dir pass; existing-external pass; journals) and the
   whole-log invariants (8-field shape, surviving verdicts only,
   ZERO pair/numword lines, model=unknown everywhere, one journal line per
   write/edit call).
2. Typecheck with the fork's checker (committed copy):
   command: `cd .opencode/plugins/tests && C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p tsconfig.json`
   result: PASS — exit 0, zero errors. The default export is checked as
   `Plugin.Plugin` against the real v2 types via the `paths` mapping to
   fork `packages/plugin/src/promise/index.ts`.
3. `git status` after the final commit: clean (verified before committing).

## Live acceptance (MAINTAINER-RUN — the worker never ran the clone's opencode)

Exact script: `agent/scripts/v2_intercept_acceptance.sh` (run from the
clone root: `bash agent/scripts/v2_intercept_acceptance.sh`). One scripted
v2 session (`opencode run --auto`): a read with the near-miss path
`agent/scripts/v2_intercept/acceptance/file-f4.txt` (expected: one
`fuzzy-resolved` line, d=1; the read succeeds against `file-4.txt`) + a
write to `C:/Users/Wasiejen/Repos/v2-accept-missing-dir/x.txt` (outside
every allowed root, parent missing — expected: one `out-of-sandbox` note
line + one `intercept-error` line; the write fails). The script prints the
new `.opencode/temp/intercept.log` lines next to the expected list. Caveat
in the script header: the first v2 run against the default data dir
migrates the live `opencode.db` (research doc §6).

## Working tree

clean after the final commit (the commit containing this file — its hash
is reported in the worker's closing message, not here).

## TODO / inbox entries

One entry appended to the MAIN repo `todo_inbox.md` via the submit tool:
the v2 `plugins` config file-path entry is dropped with a warning, so the
spec'd "minimal v2 plugins directive" is documentary only — the plugin
loads by directory auto-discovery (decide keep-or-drop).

## Deliberately omitted work

- The v1 session-DB model reader — the model field is the literal
  `unknown` (v2 `session_v2` schema is the context-family unit; porting
  the reader is out of scope for Unit A).
- The (5) edit-oldString fuzzy channel, (6) after-hook enrichment,
  block_transfer journaling, #102 bash `command`-string span redirect —
  not in the surviving set.
- No live v2 boot of the clone (boundary); acceptance is the maintainer's.

## Blocker

None.

## Lessons

- v2 hook order (execute.before fires before the permission gate) makes
  the B' hard-stop permission-independent: a blocked write fails before
  any prompt, so `--auto` acceptance runs are deterministic.
