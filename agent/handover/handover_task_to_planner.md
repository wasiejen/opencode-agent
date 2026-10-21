# Worker handover — Spec A unit A: block_transfer sandbox roots become CONFIG-DRIVEN (TODO #140 unit A)

Status: LANDED (task complete; two items deferred to planner-13 by maintainer instruction — named below).

## Task and scope
Unit A of the spec in `agent/handover/handover_task.md`: `block_transfer` must accept file paths under any root the host config lists as an allowed external directory (`opencode.jsonc` `permission.external_directory` entries valued `allow`), instead of only the launching CWD repo + Windows temp. Main repo first, clone twin `C:/Users/Wasiejen/Repos/opencode-agent-v2` as a separate commit.

## Files changed
Main repo — commit `5ce85b3` (verified unit):
- `.opencode/tools/block_transfer.ts` — config block added: string-state `stripJsoncComments` + `stripTrailingCommas` + `loadConfigRoots()` (allow keys, trailing `/**` stripped, plain-dir twin dedupe by resolved-lowercase, fail-open to EMPTY set on unreadable/malformed) + `CONFIG_ROOTS` read ONCE at tool load from the tool's OWN repo root (`import.meta.url` -> dirname + `../..`). `sandboxCheck` roots = `[cwd, ...CONFIG_ROOTS, TEMP ?? TMP, os.tmpdir()]` exact-string-deduped; message form unchanged. Description SANDBOX paragraph now names the config roots (no backticks — inside a template literal).
- `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` — byte-exact REPLACE-error pin re-pinned for the new roots list (test-side mirror loader); 12 new pins: fixture-config module copies under the scratchpad sandbox (never the live config), TEMP/TMP override isolation, per-mode ALLOW pins (COPY/MOVE/PASTE/REPLACE/WRITE), item-4 twin-dedupe + deny-exclusion pins, item-3 malformed-config fail-open pins, live-root isolation pin. 64 -> 76 pins.
- `.opencode/plugin/probes/handover_probe.mjs` — check 113's expected roots re-pinned in place (self-contained mirror loader incl. trailing-comma pass); verdict unchanged; no new probe checks (planner-13 added the 2 S15 additions itself later).

Clone tree — port complete in the WORKING TREE, deliberately NOT committed (maintainer instruction arrived mid-session: planner-13's clone commit `c45b31c` re-ports ahead of mine; do not force-push over it):
- `.opencode/tools/block_transfer.ts` — same change transplanted onto the clone's PRE-HARDENING tool (roots keep the clone's temp form `[cwd, ...CONFIG_ROOTS, process.env.TEMP ?? process.env.TMP]` — no `os.tmpdir()` addition; the clone predates hardening item 10).
- `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` — updated version copied (clone's was byte-identical to main pre-change).
- `.opencode/plugin/probes/handover_probe.mjs` — check-113 mirror transplanted (roots mirror matches the clone tool's list form).
- (git-ignored, not in any commit:) `.opencode/node_modules` copied from main — required for the clone gates to import `@opencode-ai/plugin`.

## Behavior changed
- COPY/MOVE/PASTE/REPLACE/WRITE on paths under an allow-listed root SUCCEED cross-repo (measured live: a path under the clone root yields the not-found error, not a sandbox refusal).
- Paths outside every allowed root still yield the byte-identical teaching error (all existing REJECT verdicts held).
- Config unreadable/malformed -> fail-open to the smaller set ([cwd + temp] only), never fail-closed to "everything allowed" (fixture pins).
- `X` + `X/**` config keys dedupe to ONE root; `deny`-valued keys excluded.

## Verification (exact commands, measured)
Main repo (at commit 5ce85b3, clean tree):
- `node .opencode/plugin/tests/block_transfer.smoke.mjs` -> `BLOCK_TRANSFER_SMOKE: ALL PASS (150/150)` PASS
- `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` -> `BT-SANDBOX-SMOKE: ALL PASS (76/76)` PASS (baseline 64/64)
- `node .opencode/plugin/probes/handover_probe.mjs` -> `PROBE handover: 360/360 PASS` PASS (baseline 360/360)

Clone tree — gate numbers as a DIFF vs its known red baseline (NEVER "green"):
- bt smoke: baseline 131/131 (the clone's PRE-HARDENING 131-pin variant; main's is 150) -> after port 131/131 — NO REGRESSION.
- sandbox smoke: baseline 64/64 -> after port 73/76 with env-less bash (FAILURES: the 3 byte-exact roots-list pins; cause: my bash process lacked `TEMP`/`TMP` env, so the clone tool's list = `[cwd, ...cfg]` while the mirror expected the temp entry; with env exported 76/76 passes; planner-13 later recorded 76/76 PASS). All 10 fixture pins PASS in both conditions. Re-pin requirement -> `todo_inbox.md` (2026-10-22_01-34).
- probe: `node .opencode/plugin/probes/handover_probe.mjs` -> output byte-identical to baseline (`Cannot find module ...ctx_watchdog.ts`, 0 checks run) — the TODO #141 red baseline UNCHANGED, not claimed green.
- planner-13's post-restart re-verification (maintainer, not me): clone sandbox smoke 76/76 + clone probe 360/360 PASS with their 360-pin probe + 2 new S15 checks (342/360 was my pre-sync 64-pin run — expected supersession).

## Working-tree status
- Main repo at the 5ce85b3 commit point: CLEAN. Final bookkeeping commit (this handover + `TODO.md` #140 status line + `loop/<run>/loop_log.md`) follows this file; hash not in this handover (self-reference rule) — reported by the planner's next `git log`.
- Clone tree: DIRTY BY DESIGN — 3 modified files uncommitted (planner-13's c45b31c supersedes; re-fork ahead of it, do not force-push over).
- Live acceptance (3-tool cross-repo probe after host restart): PENDING — the maintainer's to run; agents never run the clone host or backend.

## Deliberately omitted / deferred
- Clone commit (superseded per maintainer instruction above).
- TODO.md #140 status line + this handover: ride the FINAL main-repo commit (per spec).
- Spec B (path-field truncation) — not this unit.
- TODO #141 (clone ctx_watchdog.ts missing) — untouched, not unit A.

## Friction / lessons submitted
- `todo_inbox.md`: env-robust re-pin of the 3 byte-exact roots-list pins (acceptance: green under `env -u TEMP -u TMP node …`).
- `knowledge_inbox.md`: import.meta config resolution + JSONC trailing-comma requirement + os.tmpdir() env-follow facts.
- `agent/agent_feedback.md`: clone preflight near-misses (wrong-tree HEAD~1, cmp process-substitution, undeclared node_modules provisioning).
- `agent/memory/worker/memory_inbox.md`: cross-tree comparison pitfalls.

## Reusable lesson
Ports between trees at different commit epochs: diff named refs dumped to real files first (`git -C <other> show <ref>:path`), never HEAD~1 from the wrong tree nor cmp with process substitution; transplant deltas, not file copies, when the trees diverge (clone tool = pre-hardening shape).
