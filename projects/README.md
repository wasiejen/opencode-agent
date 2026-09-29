# projects/ — per-project bookkeeping

Home of per-project records in this agent repo (layout ruled 2026-09-29, #117).
Each project folder holds:
- `TODO.md` — the project's TODO entries (IDs continue the global sequence —
  the counter stays in the root `TODO.md`).
- `todo_records.md` — the project's closed-entry records (one-line records +
  full texts).
- `repo/` — the project's repo-doc set (repo_map, repo_testgate,
  repo_gotchas, repo_commands — the project-specific parts of the repo map).
- `README.md` — the project's conventions (≤10 lines).
What does NOT live here: the live loop (`loop/`), handover files + the NAP
(`agent/handover/`), the shared agent prompts (`agent/prompts/`), the central
`todo_inbox.md` + root `TODO.md` counter — those stay central at root / `agent/`.
Currently: `Free-Snap-Tap/` — the snap-tapping product's bookkeeping (its code
lives in its own repo; this folder is bookkeeping only).
