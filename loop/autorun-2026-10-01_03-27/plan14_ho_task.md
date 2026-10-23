# Spec — TODO #142 + #138: scope the observation path channels to the tool's PATH FIELDS

Assigned worker: `worker_FlashNext` (root `opencode.jsonc` agents block, line 364).
Branch state: stay on the current checkout (main repo `main`; clone `C:/Users/Wasiejen/Repos/opencode-agent-v2` branch `migration`).

## Goal

The observation log's path channels (`out-of-sandbox` note, `path-anomaly`) classify ONLY the tool's path-FIELD values, and the allow-list roots join the note check.

- #142: an `out-of-sandbox` note must NOT fire for a path under a config-`allow`ed root (the live cross-repo probe logged a false note on 2026-10-22).
- #138: path-looking substrings inside FREE TEXT (bash `command`, edit `newString`, `write` `content`) must produce NO `out-of-sandbox` / `path-anomaly` lines.

One unit, both findings — same two call sites.

## Definition of done

1. Main plugin: the observation-channel note + anomaly scan run over the joined string values of the tool's path fields (the `REDIRECT_PATH_FIELDS` table) instead of the full args string, and pass the allow-list roots; the redirect recompute site uses the same field text + allow-list.
2. Clone twin: the same change in the clone's v2 plugin.
3. Pin surfaces repurposed/added (see Pin audit) — new pin per false form + per genuine form.
4. Gates green in BOTH trees (counts below); checkpoint commit per verified unit (main, then clone); `TODO.md` status lines for #142 and #138 + `agent/handover/handover_task_to_planner.md` in the FINAL commit. Status `LANDED` — the hash lands in the planner's bookkeeping commit.
5. A new discrepancy may be filed; do not promise there is none.

## Verified facts (measured at spec-authoring, 2026-10-22)

- `.opencode/plugin/intercept_observer_core.ts:648` `observeArg(arg, map, workspaceRoot, skipPairs)`; line 659 `...observeSandbox(s, workspaceRoot)` — no `extraRoots`; `observePathAnomaly(s)` same full string.
- `.opencode/plugin/intercept_observer_core.ts:551` `observeSandbox(arg, workspaceRoot, extraRoots = [])` — the 3rd param EXISTS and is documented additive (line 545).
- `.opencode/plugin/intercept_observer.ts:1433` — the ONLY `observeArg` call site, `argStr` = the full args string.
- `.opencode/plugin/intercept_observer.ts:1474` — redirect recompute site: `observeSandbox(argsToString(output?.args), dir || null, allowedRoots)` — full string again.
- `.opencode/plugin/intercept_observer.ts:587` `REDIRECT_PATH_FIELDS`: read/write/edit `filePath` + block_transfer `srcFile`/`dstFile`; `runRedirectCommand` (bash `command` POSIX spans, #102) is a SEPARATE pass — do NOT change it.
- B' hard-stop site `.opencode/plugin/intercept_observer.ts` ~1503-1513 (write/edit `filePath`, `allowedRoots.some(...)`) — already field-scoped; do NOT change.
- Clone: `.opencode/plugins/intercept_observer.ts` line 923 observation build (`observePathAnomaly(argStr)` + `observeSandbox(argStr, dir || null)` — no roots), line 928 recompute (`allowedRoots` already), table `REDIRECT_PATH_FIELDS` at line 810 (read/write/edit `path` — the v2 field name).
- The clone's `.opencode/plugin-unported-v1/**` files are FROZEN v1 twins (precedent: TODO #140 unit B did not touch them) — leave untouched.
- Baselines: MAIN probe 360/360, intercept_observer smoke 93/93, bt 150/150, sandbox 76/76, SUBMIT_SMOKE 32/32, gauge_core 32/32. CLONE (`migration`) probe 360/360, `plugins/tests/smoke.mjs` 49/49.

## Suggested scope (non-exhaustive)

- `.opencode/plugin/intercept_observer_core.ts` — `observeArg` gains optional trailing params (allow-list roots + a scan-text override). DEFAULTS keep the direct-call behavior byte-identical (the probe's direct `observeArg`/`observeSandbox` checks stay green).
- `.opencode/plugin/intercept_observer.ts` — a small helper joining `REDIRECT_PATH_FIELDS[tool]` values; pass it + `allowedRoots` at 1433 and the same text at 1474.
- `.opencode/plugin/tests/intercept_observer.smoke.mjs` — pins per Pin audit.
- CLONE `.opencode/plugins/intercept_observer.ts` + `.opencode/plugins/tests/smoke.mjs` — the same change (the v2 field names).

## Pin audit (behaviour-pinning tests that flip — repurposing is IN SCOPE)

MAIN `.opencode/plugin/tests/intercept_observer.smoke.mjs`:
- (2b) lines 140-147: a BASH string arg firing mismatch + path-anomaly + out-of-sandbox under the 3-line cap. Free-text path classes DISAPPEAR → retarget the fixture to a TYPED call (e.g. `write` with an out-of-sandbox doubled `filePath`) so the cap pin still yields mismatch → path-anomaly → out-of-sandbox.
- (12n) lines 906-916: bash `cat /etc/hostname` — "the out-of-sandbox NOTE fires (as today)" is now FALSE: bash free text fires nothing. Keep the mutation-identity + no-kind=redirect half; flip the note expectation.
- Lines 619-835 (`R8`/`R8 config` hook pins) and the (7) read pin at 185-190: TYPED fields — expected UNCHANGED; verify.
- NEW pins: (i) #142 — a `read`/`write` with a path NESTED under a config-`allow`ed root (reuse the `proj2` fixture config block at 754-802, e.g. `rA + "/nested/deep/x.txt"`) → NO `out-of-sandbox` line and no kind=redirect; (ii) #138 — a `write`/`edit` with an in-root path whose `content`/`newString` carries path-looking text (`/tmp-redirect`, `/^## #120/`, `c:\users\users\x`) → ZERO path verdicts; (iii) a bash command with quoted doubled example paths → ZERO path verdicts.
- MAIN probe `.opencode/plugin/probes/handover_probe.mjs`: direct-call checks (4729-4798 observeArg/observeSandbox unit forms, S19 5276-5302, S28 6913-7006 hook e2e) are expected UNCHANGED by the defaults; the probe run is the verification — a hook-level pin that flips IS in scope. Baseline 360/360.

CLONE `.opencode/plugins/tests/smoke.mjs`:
- C9 (239-247), C10 (252-259), C11 (270-293), C12 (297-…), C13 (311-323), C14 (327-344), C-B1/C-B2 (380-400): typed `path` fields — expected UNCHANGED; verify.
- NEW pins: (i) a `write` with `path` NESTED under an allowed root from the fixture config written at line 60 → NO out-of-sandbox line; (ii) an `edit`/`write` with an in-root `path` + path-looking text in `content`/`newString`/`oldString` → ZERO path verdicts.

## DO NOT touch

- `maintainer/**`, root `opencode.jsonc`, `AGENTS.md`, `agent/prompts/**`.
- `runRedirectCommand` (the bash `command` redirect pass, #102) and `resolveRedirect` semantics.
- The B' hard-stop site, the 8-field log line shape, `MAX_FIELD_CHARS` / `MAX_PATH_FIELD_CHARS`, the pair/numword channel grammars.
- `.opencode/tools/**`, `.opencode/plugin-unported-v1/**` (clone), the FST repo, the main repo's `.opencode/plugin/intercept_observer.ts` channels other than the two call sites + the helper.

## Procedure suggestion (guidance, not protocol)

1. Read `TODO.md` entries #142 and #138, then the two call sites and the tables named above (line ranges given — do not read the whole probe).
2. Main edit + smoke pins; run: `node .opencode/plugin/tests/intercept_observer.smoke.mjs` then `node .opencode/plugin/probes/handover_probe.mjs`; checkpoint commit.
3. Clone twin + clone smoke pins; run in the clone: `node .opencode/plugins/tests/smoke.mjs` and `node .opencode/plugin/probes/handover_probe.mjs` (bash tool access to the clone path is allow-listed); checkpoint commit.
4. `TODO.md` status lines + `agent/handover/handover_task_to_planner.md`; final commit.
