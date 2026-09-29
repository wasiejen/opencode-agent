# repo_commands.md — Free-Snap-Tap: run/test commands (moved from `agent/prompts/repo/` on 2026-09-29, #117)

The agent-repo half of the former file (shell environment, the node-probe gate,
handover paths, the gauge) stays at `agent/prompts/repo/repo_commands.md`.

## Python / venv
- **Python**: bare `python` on PATH = 3.14, NO repo deps (fake-starts, then
  import-fails). Always `./.venv/Scripts/python.exe` (same for
  `./.venv/Scripts/ruff.exe`) — both verified working under bash.
  (#113: the venv's python.exe is currently broken — the base interpreter it
  points at is gone; environment break, maintainer call.)
- **Env var**: `FST` = repo root **with trailing `\`** — `$FSTtests`
  composes (the trailing backslash joins into the next path segment).

## Run / test
- venv with all deps: `.venv` (do NOT reinstall from scratch; `requirements.txt`
  is runtime, `requirements-dev.txt` adds test tooling, `requirements-build.txt`
  is executable-packaging only (Nuitka/PyInstaller) — CI installs runtime+dev only).
- Run tests: `./.venv/Scripts/python.exe -m pytest -q`
- Coverage: `./.venv/Scripts/python.exe -m pytest -q --cov=fst_data_types --cov=fst_manager --cov=fst_save_file_handler --cov=fst_keyboard --cov=fst_tasks --cov=vk_codes --cov=fst_overlay`
- Lint: `./.venv/Scripts/ruff.exe check --select F .` Current expected finding
  count is a moving baseline — see `agent/handover/handover_planner.md` and the
  relevant FST `TODO.md` entries (this folder).

## Gate
- The FST standard gate = pytest + ruff. The agent-repo gate (node probe +
  smoke suite) lives in `agent/prompts/repo/repo_commands.md`.
