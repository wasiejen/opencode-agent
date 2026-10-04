# state_snapshot handover — the planner init digest tool

Worker: worker_Q3S_slow (session ses_ef7344940ffe0UNjdAHseO8WUZ).
Task spec: `agent/handover/handover_task.md` (identical to
`agent/handover/specs/2026-10-04_p3-state-snapshot-tool.md`).

## What landed (3 surfaces + this handover)

1. **`.opencode/tools/state_snapshot.ts`** (new) — the planner's 6–8-call
   init sequence as ONE capped digest, per the approved proposal
   (design verbatim semantics): zero args, host auto-detect (NO
   registration, no `name` field — the ctx_gauge.ts sibling shape), soft
   role gate (non-empty `context.agent` not starting with `planner` → the
   exact one-line refusal), repo root = `context.directory` falling back
   to `process.cwd()`. The six sections IN ORDER: `## git` (git log
   --oneline -20, execFileSync args array, fail-closed line on failure),
   `## nap` (first 40 NAP lines + the read-more steer; fail-closed line
   when missing), `## todo` (the ALL-headers count line + the LAST 15
   headers, remainder truncated to 60 chars), `## maintainer` (inbox
   .md first lines, max 5 + `… (+<k> more)`; priority.md first 10 lines;
   `(none)` / fail-closed lines), `## proposals` (root .md FILENAMES
   only), `## markers` (the ready-made marker sweep VERBATIM from the
   planner prompt §maintainer calls/decisions, run via bash, capped 20 +
   `… (+<k> more)`). Empty sections → `(none)`. Total hard cap 60 lines:
   the LAST section's lines truncate first (end-backwards, content lines
   only), all six headers + every `… (…)` steer preserved. One design
   decision the spec left open (the fail-closed line TEXTS for nap /
   priority / marker-sweep failures): `(nap missing: …)`,
   `(priority.md missing)`, `(marker sweep failed: …)` — all
   deterministic, all smoke-pinned.
2. **`.opencode/plugin/tests/state_snapshot.smoke.mjs`** (new) — **N = 20
   checks**, three fixture repos in freshSandbox (NEVER the live repo):
   FX1 full (git init + 2 commits; 3 TODO headers; 45-line NAP; 2 inbox
   .md + README; 1-line priority; 2 proposals + 1 subfolder; a `--now`
   line in loop/ + a filtered `maintainer/done/` fixture) sized so the
   digest lands EXACTLY at 60 lines with no truncation; FX2 minimal
   (every fail-closed / `(none)` line); FX3 oversized (the 60-line cap:
   86 assembled → 60, the markers steer survives, proposals keep 24 of
   30, the todo section intact). `node <smoke>` exit 0 ALL PASS (20/20).
3. **`.opencode/plugin/probes/handover_probe.mjs`** — new S35 section,
   checks **354-356** (**N = 3**): (354) file exists + tool() default
   export (description non-empty string + execute function, the S12/S15
   direct type-stripped import); (355) registration-shape pin — the args
   key list is EMPTY (the compact_memory check-86 / context_trim
   check-346 pattern) + NO `name` field; (356) the description carries
   the planner-only steer (the phrase `planner`). Header tally
   machine-updated 355 → **358** (section-sum line + the EXTENDED
   history line).

## Measured verification (standard gate — worker-measured, node v24.19.0)

| gate member | baseline | now |
|---|---|---|
| probe handover | 355/355 | **358/358 PASS** (+3 S35, exit 0) |
| auto_resume | 165/165 | 165/165 (unchanged) |
| block_transfer (+sandbox) | 131/131 + 64/64 | 131/131 + 64/64 (unchanged) |
| compact_memory | 110/110 | 110/110 (unchanged) |
| context_recovery | 17/17 | 17/17 (unchanged) |
| context_trim | 40/40 | 40/40 (unchanged) |
| ctx_gauge | 3/3 | 3/3 (unchanged) |
| gauge_core | ALL PASS | ALL PASS (unchanged) |
| intercept_observer | 85/85 | 85/85 (unchanged) |
| loop_log | 69/69 | 69/69 (unchanged) |
| submit | 31/31 | 31/31 (unchanged) |
| **state_snapshot (NEW)** | — | **20/20 ALL PASS** (exit 0) |

No ruff/pytest in this repo (gate = probe + smokes). **No existing pin
broke** — DoD item 5 holds byte-green. The tool was also run LIVE
against this repo (planner stub): a 60-line digest with the cap
truncating the last sections (nap tail 10 lines), the refusal line
exact for a worker stub.

## Commits (checkpoint hashes)

- Code checkpoint: `8d332d7`
- Smoke checkpoint: `6130a70` (+ the tool fix it exposed, `025e9c5` —
  a no-match grep (exit 1, empty stdout) is an EMPTY sweep, not a
  failure; the fail-closed marker line stays for real bash/grep errors,
  and the first error line comes from the child's stderr when it
  carries text)
- Probe + handover checkpoint: **LANDED** (this final commit — its hash
  recorded by the planner's bookkeeping; no commit carries its own hash)

## TODO.md / discrepancies

- **No TODO.md entries, no TODO.md edits** (the spec: the planner owns
  the status/close note). No discrepancies found against the spec's
  verified facts.

## What was deliberately NOT done (DO-NOT-TOUCH honored)

- NO `agent/prompts/**` edit — the planner-prompt §Initialization
  (state_snapshot-at-session-start) edit is the PLANNER's separate
  follow-up commit.
- NO registration in `opencode.jsonc` (host auto-detect; confirmed in
  the spec's verified facts).
- NO writes under `maintainer/**`, NO `AGENTS.md` edits, NO changes to
  any existing tool/plugin/probe (only the two NEW test/probe surfaces).

## Environment / state flags

- node v24.19.0 throughout; bun never invoked (broken on this host).
- Host observation (cosmetic, not a TODO entry): a FAILED execFileSync
  child also ECHOES its stderr to the parent's stderr on this node —
  the smoke's intentional FX2 failure cases print `fatal: not a git
  repository…` lines to stderr; exit codes and the stdout PASS/FAIL
  stream are unaffected.
- The smoke's FX1 digest-size arithmetic (exactly 60 lines pre-cap) is
  a maintained invariant of the fixture: growing any fixture section
  changes the cap-interaction pins (check 6 pins the cap trimming the
  todo tail to #3..#12).

Lessons: a grep-in-pipeline via execFileSync exits 1 on NO MATCHES —
wrap it (status==1 + empty stdout → empty result), or the empty case
looks like a failure; and when pinning a capped digest in a fixture,
compute the assembled size line-by-line FIRST — the 60-line cap
interacts with the per-section caps silently.
