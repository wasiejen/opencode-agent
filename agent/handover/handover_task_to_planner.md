# Worker handover — Spec B unit B: the intercept log NEVER cuts inside a single-path FIELD (TODO #140 unit B)

Status: LANDED. Task complete in both trees; live acceptance stays the maintainer's (spec).
Previous Unit A handover content (Spec A) is preserved in git history (commits 28a6685/e07f9b9); this file is superseded per the channel contract.

## Task and scope
`agent/handover/handover_task.md` (Spec B, uncommitted planner edit now committed with the final commit): field-content rule in `.opencode/plugin/intercept_observer_core.ts` — a single-path field prints whole (raised cap); a mixed prose+path field's cut moves back so it never lands inside a path substring; 8-field shape, field order, ASCII `...` marker byte-compatible. Clone twin `C:/Users/Wasiejen/Repos/opencode-agent-v2` as its own commit on branch `migration`.

## Files changed
Main repo — commit `a46ca9b`:
- `.opencode/plugin/intercept_observer_core.ts` — `MAX_PATH_FIELD_CHARS = 1024` (worker's call, documented in the CAPS block); helpers `isPathShapedValue` (one whitespace-free token containing a path separator — the JSON-stringified single-field arg qualifies) + `safeCutLen` (largest p, cap-3 >= p >= 1, keeping every path-shaped whitespace token whole); `flattenField` = whole-print under the raised cap, else moved-back cut, else the existing short summary form `slice(0, cap-3) + "..."` (degenerate: leading over-cap path token). Docs: line-shape block + CAPS block updated.
- `.opencode/plugin/tests/intercept_observer.smoke.mjs` — 85 -> 93 pins (DoD item 1: >-cap single path prints whole + beyond-raised-cap fallback; item 2 per form: leading-path fallback, prose+trailing, two paths, prose-only byte-compat with the old cut; item 3: hook end-to-end byte-shape pin — 8 fields, field 5 = whole JSON arg, no `...` in the line); header block + new pins. No existing smoke pin re-pinned (none asserted a literal truncated path field; the flattenField-computed expected pins self-track).
- `.opencode/plugin/probes/handover_probe.mjs` — NOT edited. Intercept sections audited IN PLACE: every intercept expected value flows through `ioCore.flattenField` (self-consistent); 360/360 PASS, annotation == section sums. No probe check ids re-pinned.

Clone repo — commit `bdbf663` (branch `migration`, the checked-out branch):
- `.opencode/plugins/intercept_observer.ts` — same rule transplanted into the single-file twin (not a file copy: the clone predates the core split; constants non-exported, clone comment style `->`).
- `.opencode/plugins/tests/smoke.mjs` — 46 -> 49 pins: C11 re-pinned (measured: the field is now `kind=redirect tool=write arg=path orig=<sibling> ...` — the cut moved back to the `value=` token start, so the old `value=` 20-char-prefix assertion was replaced with orig-whole + value-dropped-whole exact equality) + C-B1/C-B2/C-B3 e2e pins (long doubled path line: 8 fields + whole JSON arg + no marker; long out-of-sandbox note: whole arg field + evidence short-summary fallback, byte-exact expected from the known `path=... root=...` evidence form; the note line stays 8 fields >160).

## Behavior changed
- A field whose flattened value is one path-shaped token (incl. the JSON-stringified single-field arg) prints WHOLE up to 1024 chars — no `...` mid-path.
- A mixed prose+path field over 160: the cut moves BACK to the last position that keeps every path-shaped token whole (paths print whole or are dropped whole; prose may still cut mid-word as before).
- Degenerate leading-path-over-cap fields keep the existing short summary form (documented fallback, pinned both trees).
- Unchanged: verdict semantics, B' hook, fuzzy resolvers, R8 root list, the 8-field shape, field order, `...` marker, all live-line channel/mutation behavior.

## Verification (exact commands + results)
Main repo (gates re-run at the final committed state, which contains everything below):
- `node .opencode/plugin/tests/intercept_observer.smoke.mjs` -> `INTERCEPT_OBSERVER_SMOKE: ALL PASS (93/93)` PASS (baseline 85/85)
- `node .opencode/plugin/probes/handover_probe.mjs` -> `PROBE handover: 360/360 PASS` PASS (baseline 360/360; unchanged)
Clone repo (HEAD bdbf663) — gate as DIFF vs its known red baseline, NEVER "green":
- `node .opencode/plugin/probes/handover_probe.mjs` -> output byte-IDENTICAL to the pre-port baseline (`Cannot find module ...ctx_watchdog.ts`, TODO #141; 0 checks executed) — no new red.
- `node .opencode/plugins/tests/smoke.mjs` -> `SMOKE: ALL PASS` with 49 PASS lines (baseline 46 -> 49) PASS.

## Working-tree status
- Main repo: clean after the final commit EXCEPT: `loop/autorun-2026-10-01_03-27/loop_log.md` gains the DONE line AFTER the final commit (last action, same as unit A's pattern — planner commits it), and 4 PRE-EXISTING untracked `archive/sessions/compaction_dumps/*` files (another session's dumps — not touched).
- Clone repo: clean at bdbf663.

## Commits
- Main: `a46ca9b` (core + smoke, named paths) — final main commit carries TODO.md #140 status line + this handover + the spec `agent/handover/handover_task.md` (committed with it; the planner's spec-B working edit becomes canonical; report that hash from the `git log` line after committing).
- Clone: `bdbf663` (plugins/intercept_observer.ts + tests/smoke.mjs, branch migration).

## TODO / inbox entries
- `TODO.md` #140: Unit B LANDED line added after the Unit A line (entry stays OPEN — live acceptance pending, maintainer).
- `agent/agent_feedback.md` (via submit): block_transfer sandbox refused the clone tree (its roots load once at host start, before unit A landed) — the `edit` tool was used for the clone per the task boundary; noted as tool-availability friction.
- No new todo_inbox findings beyond that.

## Deliberately omitted
- Live 3-tool/path-echo acceptance (spec: maintainer's next session / grep of the live log) — agents do not run the clone host or the backend.
- The out-of-sandbox note evidence form itself (`path=<span> root=<ws>`) — a leading over-cap path token still summary-forms (documented fallback); re-forming the evidence string would change existing byte-exact pins (out of scope).
- TODO #141 (clone ctx_watchdog.ts missing) — untouched by design; clone probe red baseline preserved.
- Unit A files, `block_transfer`, `opencode.jsonc`, `maintainer/**`, prompts, knowledge, tools — untouched per boundaries.

## Reusable lesson
When a suite computes its expected field value through the SAME function under change, pins self-track and the pin audit is a grep for LITERAL truncated strings (`value=${X.slice(0,20)}`-style prefix asserts) — that was the only breakage class (clone C11). Use `cmp` (not `fc -L`) to prove probe-output identity vs a red baseline.
