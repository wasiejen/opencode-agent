# plan12 summary — 2026-10-22 (autorun iteration 12, `ses_e9fb4e611ffeQmrH3PkDK6r4Ct`)

Closed the maintainer's priority item 1 (TODO #140, cross-repo access) in two worker units,
main repo + v2 clone, after my self-compaction at the 80% triage rung.

## Landed (all planner-verified from `git show` + gate re-runs)
- Unit A — `block_transfer` sandbox roots become CONFIG-DRIVEN (`permission.external_directory`
  `allow` set): main `5ce85b3`, clone twin `8d6bbd5` (branch `migration`).
  Pins: sandbox smoke 64→76 (per-mode ALLOW, `/**`-dedupe, deny-exclusion, malformed-config fail-open,
  fixture config — the live `opencode.jsonc` never read/written); probe check 113 re-pinned in place.
- Unit B — the intercept log NEVER cuts inside a single-path field: main `a46ca9b`
  (+ close `3889c42`), clone twin `bdbf663`. Pins: `intercept_observer.smoke.mjs` 85→93
  (whole-path forms, mixed prose+path cut-moves-back forms, the short-summary fallback);
  clone plugins smoke 46→49 (C11 re-pinned, C-B1/C-B2/C-B3 added).
- Gate as measured by me: main `93/93`, `76/76`, `150/150`, `PROBE handover: 360/360 PASS`;
  clone plugins smoke 49 PASS lines, bt `131/131`, clone probe RED on `ctx_watchdog.ts` (#141, by design).
- Trees: clone CLEAN at `bdbf663`; main dirty only in `loop/<run>/loop_log.md` + 4 pre-existing untracked compaction dumps.
- Worker-prompt guard (`NEVER self-compact at task close`) landed earlier in this session in `agent/prompts/agent_worker.md`.
- NAP refreshed (`b6c64fa`, plus the earlier post-compact line `50b9b12`); the wave specs sit in `agent/handover/specs/`
  (`2026-10-22_cross-repo-roots-A.md`, `2026-10-22_path-field-truncation-B.md`), the handover copies in this folder.

## Live acceptance — PENDING the maintainer's host restart
`.opencode/tools/*.ts` and the plugin core load at host start, so both fixes are un-accepted until his next start.
He runs: `write` + `edit` + `block_transfer` COPY into `C:/Users/Wasiejen/Repos/opencode-agent-v2/agent/…` + a read-back,
then grep `.opencode/temp/intercept.log` for any path echo cut mid-word. Expect no sandbox refusal, no `C:\Users\*` ask, full paths.

## Friction / lessons submitted (committed with this summary)
- Worker close narratives can FABRICATE references (`4bef076` cited a nonexistent `c45b31c`, a restart,
  a maintainer instruction; the worker self-corrected in `28a6685`) → memory entry: verify every hash/claim against `git log`/`git show` first.
- `agent/readme/readme_post_compaction.md` STEP 1 names three nonexistent files (`repo_map.md`, `repo_testgate.md`, `repo_gotchas.md`) → feedback entry.
- The sandbox smokes crash without `TEMP`/`TMP` at startup → `todo_inbox.md` follow-up (worker's, cause-corrected).

## Next session (priority order)
1. TODO #141 — the clone's probe gate red (`ctx_watchdog.ts` import); unblocks all clone verification.
2. TODO #137 unit 2 — the token-mass legs in the v2 gauge family, then the v2 tools estate (priority item 2).
3. TODO #138 — the R8 path resolver false-fires on path-looking substrings in command/payload TEXT (unit B deliberately left the `out-of-sandbox` evidence form alone).
4. TODO #139 — intercept.log flush-time sid/timestamp mislabeling.
5. Live-acceptance confirmations: the 3-tool cross-repo probe (his run), #135 U4 `nudge=` leg, #134 B', the FST logging confirmation.
6. Queue PENDING his decision: the 3 Magic Context proposals + the 2-tier tool-description proposal.

Iteration note: the counter file reads `n":11`, the loop log's max token is `planner-12`; my `plan12_*` files stay
(no clobber), the successor takes 13 by the max() rule. Compactions left after my self-compact: 2.
