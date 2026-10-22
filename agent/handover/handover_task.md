# Task spec — two small pre-approved fixes: sandbox-smoke env-robustness + block_transfer WRITE regions (BOTH trees)

## Goal
1. `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` stops depending
   on the ambient shell `TEMP`/`TMP` env (an env-free gate run crashes before
   pins / fails byte-exact pins).
2. `block_transfer` WRITE with a `regions` LIST + ONE `text` REJECTS with a
   teaching error instead of writing the identical text into every region.

## Assignment
- Worker: `worker_FlashNext`. Work order: MAIN repo
  `C:\Users\Wasiejen\Repos\opencode-agent` (branch `main`, checked out — stay
  on it) = unit 1; then the clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`
  (branch `migration` — stay on it) = unit 2 twins. Both units in BOTH trees.
- Sources: `TODO.md`-adjacent findings — todo_inbox entries
  `## 2026-10-22_01-34` (env-robust) and `## 2026-10-22_03-26` (regions;
  `todo_inbox.md` lines 258-266; decision-needed line names the recommended
  end state: REJECT with a teaching error).

## Definition of done (measurable)
Unit 1 (env-robust, smoke file only):
- The 3 byte-exact sandbox-roots-list pins (REPLACE reject message, fixture
  REJECT message, item-3 single-root form) derive their expected roots from
  the tool's OWN live message shape or from a fixture env the smoke sets
  itself — never the ambient shell env.
- Acceptance: `env -u TEMP -u TMP node .opencode/plugin/tests/block_transfer.
  sandbox.smoke.mjs` → ALL PASS in MAIN and in the clone; WITH env set: still
  ALL PASS (both trees).

Unit 2 (regions reject, tool + smokes + doc):
- `WRITE` with a `regions` LIST and a single `text` (1 text, N>1 regions)
  returns a teaching error naming the mismatch (per-region `text` per
  region, or exactly one region per text — reject, do not silently fan one
  text across regions). The single-span form (markers + one text) is
  UNCHANGED.
- Smoke gains pins: (a) N-regions+1-text → rejection (error text shape),
  (b) a correctly paired form if supported, else a second rejection form,
  (c) single-span form still writes (existing pin stays green).
- Tool description (`tool()` desc in the .ts + `.opencode/tools/block_transfer.md`)
  gains one line: N regions require per-region `text`.

## Baselines (measured fresh, 2026-10-22)
- MAIN: bt smoke 150/150; sandbox smoke 76/76 (73/76 under env-less bash —
  3 byte-exact roots-list pins fail; worker-14 measured, planner re-run
  76/76 with env); probe 360/360.
- CLONE: bt smoke 131/131; sandbox smoke 76/76; probe 360/360 (GREEN —
  TODO #141 landed).

## Suggested scope (non-exhaustive)
- MAIN `.opencode/tools/block_transfer.ts` (WRITE regions handling + the
  `sandboxCheck` roots message + the tool description) ;
  `.opencode/plugin/tests/block_transfer.smoke.mjs` ;
  `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` ;
  `.opencode/tools/block_transfer.md` (WRITE section line).
- CLONE twins: `.opencode/tools/block_transfer.ts`,
  `.opencode/plugin/tests/block_transfer.smoke.mjs`,
  `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs`,
  `.opencode/tools/block_transfer.md`.

## Verified facts (measured at spec-authoring)
- MAIN `block_transfer` WRITE `regions` = LIST of `{start,end}`; passing TWO
  regions with ONE `text` writes the identical text into BOTH (measured 2026-
  10-22: the planner-14 NAP repair + the worker's TODO.md repair both hit it
  unnoticed at first — todo_inbox `2026-10-22_03-26`).
- The sandbox smoke reads `TEMP`/`TMP` at startup (line ~13) — env-free run
  crashes before pins (todo_inbox `2026-10-22_01-34`; measured 73/76 env-less
  in the clone after unit A).
- The tool already loads config allow-roots (`CONFIG_ROOTS`) — unit A of
  TODO #140 (MAIN 5ce85b3 / clone 8d6bbd5); sandbox smoke 64→76 pins there
  (fixture config under the scratchpad, TEMP-override isolation — the env
  dependency rides that design; do NOT change the isolation, make the smoke
  SELF-SET its fixture env or derive the expected strings from the tool's
  output).
- Probe check 113 (MAIN) / clone twin pin the config-roots sandbox form —
  probe sections NOT expected to move (tool message shape for the CONFIG
  roots line stays; if a message text changes, repin 113 + the clone twin
  IN SCOPE, name it in the handover).

## Approval boundary
Pre-approved class (tool fixes + test robustness, agent-facing only). DO
NOT touch: `maintainer/**`, probe code beyond a named repin (if any), the
WRITE single-span behavior, other tool modes, `.opencode/plugin-unported-v1/**`.

## Procedure (guidance)
1. MAIN unit 1: fix the sandbox smoke (self-set env or message-derived
   expectations); run both acceptance forms (env / env-less).
2. MAIN unit 2: regions reject + smoke pins + description lines; bt smoke
   150/150 → ALL PASS with new pins; probe stays 360/360.
3. Clone twins of both units; run the clone smokes + clone probe 360/360.
4. Checkpoint commits per unit per tree (4 commits max; code only); FINAL
   commit carries `agent/handover/handover_task_to_planner.md`; trees clean.

## Out of scope
- The reject-vs-paired-text decision alternative (REJECT is the recommended
  end state per the inbox line; if the worker finds paired-text already
  half-implemented, say so in the handover — do NOT build it).
- TODO status lines (planner records them in its bookkeeping commit).
