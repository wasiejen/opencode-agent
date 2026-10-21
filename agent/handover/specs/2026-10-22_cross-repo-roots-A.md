# Spec A — block_transfer sandbox roots become CONFIG-DRIVEN (TODO #140, unit A)

Assigned worker: `worker_FlashNext` (same model as the planner — the cache rule).
Order: unit A first; spec B (`.opencode/temp` of the specs folder:
`agent/handover/specs/2026-10-22_path-field-truncation-B.md`) after A lands.

## Goal
`block_transfer` accepts file paths under any repository/root the host config
lists as an allowed external directory (`permission.external_directory` entries
valued `allow` in `opencode.jsonc`), instead of only the launching CWD repo
+ the Windows temp dir. Cross-repo work (the v2 clone, FST) stops being
refused while the guard still rejects everything genuinely outside.

## Definition of done (measurable)
1. A `block_transfer` COPY/MOVE/PASTE/REPLACE/WRITE on a path under an
   allow-listed root SUCCEEDS (measured by a smoke pin using the repo's own
   config — the scratchpad sandbox fixture must write a test config file, not
   read the live `opencode.jsonc`; one pin per tool mode that checks a root).
2. A path outside EVERY allowed root (config root + CWD repo + temp root) still
   returns the existing teaching error, byte-identical to today's message
   (pins stay green).
3. Config-unreadable/malformed → the CURRENT behavior (CWD repo + temp only)
   — fail-open to the smaller set, never fail-closed to "everything allowed".
4. `/**`-suffix keys and the plain-dir twin dedupe to one root (same rule the
   R8 resolver uses).
5. Pin audit green: `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs`
   (today 64 pins) re-pinned where the allowed set changes + new pins per item
   1/3/4; the probe's S15 `block_transfer tool (12)` section
   (`.opencode/plugin/probes/handover_probe.mjs` header annotation at line ~481)
   audited — re-pin in place any check whose expected verdict changes.
6. Clone twin: the same change in
   `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\tools\block_transfer.ts`
   + its smoke; the clone's gate numbers recorded as a DIFF (the clone probe is
   red at baseline — TODO #141; never claim "gate green" in the clone).
7. Working tree clean; the standard gate green in the MAIN repo: probe
   `PROBE handover: <t>/<t> PASS` with the annotation agreeing with the section
   sums; all smokes green (baseline: bt smoke 150/150, sandbox 64/64 → higher
   after the new pins, probe 360/360 → higher after any new checks).

## Verified facts (measured by the planner, 2026-10-22)
- `.opencode/tools/block_transfer.ts` — `sandboxCheck(cwd, givenPath)` at line
  12; the temp root = `os.tmpdir()` with `process.env.TEMP ?? process.env.TMP`
  as an OVERRIDE (lines 13-20, the hardening item 10); call sites at lines
  278 / 303 / 522 / 571 / 638 / 761 / 766.
- The allowed-external-roots shape ALREADY EXISTS: `resolveAllowedRoots()` at
  `.opencode/plugin/intercept_observer.ts:319-345` — reads `opencode.jsonc`
  (JSON.parse over a comment-stripping helper), collects
  `permission.external_directory` keys valued `allow`, strips a trailing
  `/**` (line 330), adds `references.*.path` values + the workspace dir, and
  falls back to [workspace, scratchpad] on a read failure. Build on that
  shape; do NOT re-research the config format.
- `block_transfer` runs as a TOOL file (`.opencode/tools/`), not a plugin:
  it must resolve the config path itself (the tool's own repo root, same as
  the sandbox uses today) — it CANNOT call the plugin's function.
- The live `opencode.jsonc` (`C:/Users/Wasiejen/Repos/opencode-agent/opencode.jsonc`)
  lines 25-40: `external_directory` allow entries incl. BOTH
  `…/opencode-agent-v2` and `…/opencode-agent-v2/**`, `…/Free-Snap-Tap`,
  `…/opencode`, the npm dirs, and the scratchpad.
- The 2026-10-21 hardening unit changed the return-line shapes + `mode` is
  strictly required (150/150 pins) — do not alter those shapes.

## DO NOT touch
- `opencode.jsonc` (the maintainer's live file) — read-only; the smokes use a
  fixture config under the scratchpad sandbox.
- `maintainer/**`, `agent/prompts/**`, `TODO.md` (except a status line in the
  FINAL commit), the live opencode DB, the backend inference server, any
  other plugin/tool file, the pair/numword channels, `intercept_observer*`.
- The clone root `opencode.jsonc`.

## Commit shape
- One green checkpoint commit per verified unit step (named paths only, NEVER
  `git add -A`); main repo + clone are SEPARATE commits in their own trees.
- Status wording `LANDED`; the hash is recorded by the planner's follow-up
  bookkeeping commit in the main repo (never in its own commit).
- `TODO.md` (#140 status) + the handover ride the FINAL commit.

## Live acceptance (pending, name it in the handover)
- The maintainer runs the 3-tool cross-repo probe after the host restarts: a
  `write`, an `edit`, and a `block_transfer` COPY into
  `C:/Users/Wasiejen/Repos/opencode-agent-v2/agent/…` + a read back, expecting
  NO sandbox refusal and NO permission ask naming `C:\Users\*`.
- Agents never run the clone host or any backend request.
