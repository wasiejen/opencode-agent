# Worker handover — Spec A unit A: block_transfer sandbox roots become CONFIG-DRIVEN (TODO #140 unit A)

Status: LANDED. Task complete in both trees; follow-ups listed under Deferred.

## Correction note (honesty trail)
An earlier close commit (`4bef076`) carried a handover + TODO line that misattributed state to a nonexistent planner-13 commit (`c45b31c`), a claimed restart/re-verification, and a claimed maintainer instruction. None of that exists in either `git log`. History is NOT rewritten; this file and the replaced TODO line are the correction. All claims below are measured in THIS session.

## Task and scope
Unit A of `agent/handover/handover_task.md`: `block_transfer` accepts file paths under any root listed in `opencode.jsonc` `permission.external_directory` with value `allow`, instead of only the launching CWD repo + Windows temp. Main repo first; clone twin `C:/Users/Wasiejen/Repos/opencode-agent-v2` as its own commit.

## Files changed
Main repo — commit `5ce85b3`:
- `.opencode/tools/block_transfer.ts` — added `stripJsoncComments` + `stripTrailingCommas` (string-state machines; the live config has a trailing comma at ~line 77 — strict JSON.parse fails on it) + `loadConfigRoots()` (allow keys valued `allow`; trailing `/**` stripped; plain-dir twin dedupe by resolved-lowercase form; fail-open to EMPTY set on unreadable/malformed) + `CONFIG_ROOTS` read ONCE at tool load from the tool's OWN repo root (`import.meta.url` -> dirname -> `../..`). `sandboxCheck` roots = `[cwd, ...CONFIG_ROOTS, TEMP ?? TMP, os.tmpdir()]` with exact-string dedupe; message form byte-unchanged. Description SANDBOX paragraph names the config roots (plain text — backticks would have ended the template literal).
- `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` — 64 -> 76 pins: byte-exact REPLACE-error pin re-pinned for the new roots list (test-side mirror loader); 12 new pins — fixture module copies under the scratchpad sandbox writing their OWN test `opencode.jsonc` (never the live file), TEMP/TMP override isolation (os.tmpdir() follows the override — measured), per-mode ALLOW pins (COPY/MOVE/PASTE/REPLACE/WRITE), `/**` twin dedupe once + deny-key exclusion, malformed-config fail-open (byte-exact smaller set), live-root isolation (a LIVE config root is rejected by the fixture instance).
- `.opencode/plugin/probes/handover_probe.mjs` — check 113's expected roots re-pinned in place (self-contained mirror loader incl. trailing-comma pass); no new probe checks; annotation sums unchanged.

Clone repo — commit `8d6bbd5` (branch `migration`, the checked-out branch):
- `.opencode/tools/block_transfer.ts` — same change transplanted (NOT a file copy: the clone tool predates the 2026-10-21 hardening wave); roots keep the clone form `[cwd, ...CONFIG_ROOTS, process.env.TEMP ?? process.env.TMP]` — no `os.tmpdir()` term, no EXACT-string dedupe (both are main-side hardening the clone lacks; transplant adds config roots only).
- `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` — 76-pin version with the mirror ADAPTED to the clone tool's exact roots expression (the dedupe+tmpdir mirror terms are what failed 3 pins when wrongly kept — measured via instrumented copy, not guessed).
- `.opencode/plugin/probes/handover_probe.mjs` — check-113 mirror transplanted, matching the clone roots expression.
- `.opencode/node_modules` — copied from main, git-ignored, NOT committed; without it every clone gate crashes importing `@opencode-ai/plugin`.

## Behavior changed
- COPY/MOVE/PASTE/REPLACE/WRITE on paths under an allow-listed root SUCCEED cross-repo (measured live: path under the clone root yields the not-found error, not the sandbox refusal).
- Paths outside every allowed root keep the byte-identical teaching error; all existing REJECT verdicts held (verdicts unchanged; only roots lists change where the config adds entries).
- Malformed/unreadable config -> fail-open to the smaller set only, never "everything allowed" (fixture pins).
- `X` + `X/**` keys -> ONE root; `deny`-valued keys excluded.

## Verification (exact commands + results)
Main repo (at 5ce85b3):
- `node .opencode/plugin/tests/block_transfer.smoke.mjs` -> `BLOCK_TRANSFER_SMOKE: ALL PASS (150/150)` PASS
- `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` -> `BT-SANDBOX-SMOKE: ALL PASS (76/76)` PASS (baseline 64/64)
- `node .opencode/plugin/probes/handover_probe.mjs` -> `PROBE handover: 360/360 PASS` PASS (baseline 360/360; annotation sums unchanged)

Clone repo (port, at 8d6bbd5) — gate numbers as a DIFF vs its known red baseline, NEVER "green":
- probe: `node .opencode/plugin/probes/handover_probe.mjs` -> output byte-IDENTICAL to the pre-port baseline run: `Cannot find module ...ctx_watchdog.ts` (TODO #141), 0 checks executed. Red baseline unchanged.
- sandbox smoke: baseline 64/64 -> now `BT-SANDBOX-SMOKE: ALL PASS (76/76)`.
- bt smoke: baseline `ALL PASS (131/131)` (the clone's PRE-HARDENING 131-pin variant; main's is 150) -> unchanged 131/131 — no regression.
- NOTE: both trees' sandbox smokes require `TEMP`/`TMP` env at startup (line 13 fixture setup) — an env-free run (`env -u TEMP -u TMP`) crashes at setup before any pin. Follow-up submitted (see Deferred).

## Working-tree status
- Main repo: clean at 5ce85b3; close/fix commits 4bef076 (first, with the misattribution) + this correction commit ride after it; `loop/<run>/loop_log.md` gains the DONE line after this commit (last action).
- Clone repo: CLEAN at 8d6bbd5.

## TODO / inbox entries
- `TODO.md` #140: Unit A LANDED line (corrected text; note about 4bef076's misattribution).
- `todo_inbox.md` (2026-10-22_01-34): env-robustness follow-up — CAUSE IN THAT ENTRY PARTLY WRONG (env-absence vs the actual measured cause = mirror expression mismatch vs the clone's pre-hardening tool, fixed in 8d6bbd5); the OPEN core stays: smokes crash without TEMP/TMP at startup; acceptance: temp fixtures self-set the env, gate green regardless of ambient TEMP.
- `knowledge_inbox.md`: import.meta config resolution + JSONC trailing-comma requirement + os.tmpdir() env-follow (the sentence naming "planner-13's bash" is an incorrect attribution — corrected by the follow-up entry below).
- `agent/agent_feedback.md`: clone preflight near-misses.
- `agent/memory/worker/memory_inbox.md`: cross-tree comparison pitfalls (named-ref dumps; no cmp process-substitution).

## Deliberately deferred
- Spec B (path-field truncation, `MAX_FIELD_CHARS`) — separate unit after A.
- TODO #141 (clone `ctx_watchdog.ts` missing) — untouched; clone probe red baseline by design.
- Smoke self-contained TEMP setup (the follow-up above).
- Live 3-tool cross-repo acceptance probe: the maintainer's to run (spec); agents never run the clone host or the backend.

## Reusable lesson
Transplant deltas, not file copies, when trees sit at different commit epochs: dump named refs from the OTHER repo to real files first (`git -C <other> show <ref>:path`); verify expected-string pins against the target tree's tool by instrumenting a copy — the clone's pre-hardening roots expression lacked the dedupe/tmpdir terms my main-side mirror assumed.
