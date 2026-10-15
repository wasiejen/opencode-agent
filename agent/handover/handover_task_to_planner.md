# Worker handover — Unit A: intercept observer ported to v2 (TODO #137)

Status: IN PROGRESS
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

## Completed

- [x] Unit 1 (committed): the unported v1 plugin files are inert — all 7 top
  level `.ts` modules + 5 doc twins moved out of `.opencode/plugin/` into
  `.opencode/plugin-unported-v1/` (mechanism + the 6-vs-7 count documented in
  that folder's README). No v1 file remains in the v2 discovery dirs
  (`.opencode/plugin/`, `.opencode/plugins/`); the remaining subdirs there
  carry no `server.ts`/`index.ts` entrypoint and are dropped at scan time
  (fork `packages/core/src/config/plugin/source.ts:164-167`).

## Remaining

- [ ] Unit 2: the v2 plugin file in `.opencode/plugins/` + the clone
  `opencode.jsonc` `plugins` entry.
- [ ] Unit 3: bun smoke (mock ctx) of the surviving channels + typecheck
  against the fork's `@opencode/plugin`.
- [ ] Unit 4: live-acceptance fixture + script for the maintainer, handover
  final, todo_inbox entries, final commit.

## Verification so far

- `git status` clean after Unit 1 (measured, after the commit).
- Boot-cleanliness of the moved files is verified by code inspection of the
  fork loader (paths above), NOT by running the clone's `opencode.exe`
  (never run — task boundary).

## Blocker

None.
