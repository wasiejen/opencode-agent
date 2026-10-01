# orientation.md — Free-Snap-Tap: where the project is going

Planner-authored distillation (2026-10-01, plan5 maintenance pass) of the
maintainer's goal sketch (his ideas.md 2026-09-30 "goal overview for each
Repo" entry) + the settled FST facts. The NAP owns the CURRENT state; this
file owns the DIRECTION. Explicit maintainer instructions override both.

## What FST is
The snap-tap / rebind / macro tool — the product code (stable, gates green).
Per-project bookkeeping + repo docs live in THIS folder (`TODO.md`,
`todo_records.md`, `repo/`).

## Direction (maintainer goal sketch, ideas.md 2026-09-30)
- **Updates always on `opencode_test`** — always checked out, directly
  testable by the maintainer.
- **Input lag is the standing goal:** always find and resolve cases that
  would lead to input lag — a failing part must NEVER impede general
  responsiveness (no blocking, no lag when it fails). The first start of any
  repeat may be a bit laggy (acceptable; first-call pre-warm candidates —
  see #130).
- **Keep the wiki + readme current** so new versions can be published —
  once a successful rework is tested, a new version (e.g. v1.3.0) is
  published on GitHub as a new release; the issue-tracker access may be
  granted to find problems and fix them directly.
- **Config support direction** (his ideas.md 2026-09-30 "FST config file
  check"): near-term = a config validation function set (syntactic +
  semantic checks, pipeable independent of the complete program — he
  flags the logic-doubling cost: it must be maintained alongside each code
  change) and/or a small GUI (tabs per focus group, a config-file picker +
  live check without loading, displays pointing at the wrong parts, and the
  currently-pressed key with its vk code from `vk_codes.py`). The grand
  goal = a drag-and-drop config designer (he rates it probably impractical —
  the real win is better diagnostics of config errors + hidden
  interconnections: forgotten suppressions, interacting variables, repeat-
  function internals). NOT a task yet — a future FST feature (maintainer
  call).
- **Wacom Touch Driver** (separate repo — NO agent access yet): add Pen
  support; improve responsiveness + configurability via the GUI (rotation of
  input options, multiple connected touch pads); the opentabletdrive repo
  as inspiration source; possibly a vision module to let the agent see the
  GUIs.

## Working conventions (settled facts)
- FST gate = pytest + ruff (commands in `repo/repo_commands.md`); the repo
  venv is 3.12.9 (#113 close).
- Work branches: `fst_work*` (current `fst_work3`, off `opencode_test`);
  branch truth is verified with `git branch -v` at task start, never from
  memory.
- Live testing / the real display = maintainer domain; agents verify via the
  test gate + offscreen probes (the Qt offscreen facts are in
  `repo/repo_gotchas.md`).
- FST TODO entries live in THIS folder's `TODO.md`; IDs continue the global
  sequence (the counter in the root `TODO.md` header).
