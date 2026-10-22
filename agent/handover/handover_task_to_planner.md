# Worker summary — planner-15 units: sandbox-smoke env-robustness + block_transfer WRITE regions (BOTH trees)

Status: COMPLETE
Assigned: `worker_FlashNext`, session `ses_e9e9f2891ffe2dP7gkx74SGSxH`.
Work order: MAIN repo (`opencode-agent`, branch `main`) units 1+2, then the clone (`opencode-agent-v2`, branch `migration`) twins. Stayed on each branch; absolute paths used for every clone file edit (relative paths resolve against the session CWD = MAIN).

## Files changed

MAIN (`C:\Users\Wasiejen\Repos\opencode-agent`) — commit `05df70f`:
- `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` — the smoke SELF-SETS `process.env.TEMP`/`TMP` to `<Temp>/bt_sandbox_env_temp` (a fixture dir under `SCRATCHPAD/..`, created at startup, deleted in the exit finally, ambient values restored). `tempRoot` no longer reads the ambient shell env. The 3 byte-exact roots-list pins (REPLACE reject, fixture REJECT, item-3 single-root) now hold because the test-side mirror and the tool's `sandboxCheck` read the SAME env at call time. Tool isolation design untouched.

MAIN — commit `b713585`:
- `.opencode/tools/block_transfer.ts` — WRITE gains the regions/text contract guard; the `description` MODES WRITE sentence and the `regions` arg describe gain the N-regions contract line.
- `.opencode/tools/block_transfer.md` — one new bullet: the WRITE `regions` contract.
- `.opencode/plugin/tests/block_transfer.smoke.mjs` — the old 2-region fan-out pin replaced by a reject pin; a paired 1-region pin added.
- `.opencode/plugin/probes/handover_probe.mjs` — check 307 REPINNED (named repin).

CLONE (`C:\Users\Wasiejen\Repos\opencode-agent-v2`) — commit `2a3759f`:
- `.opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` — same self-set fixture env block (clone twin), same finally restore.

CLONE — commit `4054a0a`:
- `.opencode/tools/block_transfer.ts`, `.opencode/tools/block_transfer.md`, `.opencode/plugin/tests/block_transfer.smoke.mjs`, `.opencode/plugin/probes/handover_probe.mjs` (check 307 REPINNED) — same contract guard and message, byte-identical reject text; the pins use the CLONE's own older feedback format (`(lines 2..3, first: 'REPL')`, not MAIN's hardening-item-7 form).

## Behavior changed

- WRITE with a `regions` LIST of more than one region and the single `text` arg now returns, byte-exact:
  `Error: the 'regions' list has 2 regions but WRITE takes ONE 'text' (it would fan the same text into every region) — use exactly one region in the list, or one WRITE call per region.`
  (the count tracks the region count). The guard runs AFTER the overlap check — the overlap teaching stays reachable — and BEFORE any write, so no partial state and no `last_write` store on rejection. The single-span form and the ONE-region list form are UNCHANGED (verified byte-exact).
- Gate hygiene only: the sandbox smoke no longer depends on the ambient `TEMP`/`TMP`; it self-sets them and restores the ambient values.
- Deviation from the suggested procedure: none in substance. Probe repins exceeded the spec's single check-113 assumption — the regions change forces check 307 in both trees (its byte-exact fan-out pin asserted the old behavior); check 113 did NOT move (no sandbox message shape change).

## Measured verification (exact commands, PASS/FAIL)

MAIN, after unit 1 (commit `05df70f`):
- `env -u TEMP -u TMP node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` → PASS `BT-SANDBOX-SMOKE: ALL PASS (76/76)` (was a startup `TypeError` before: `path.join(undefined)`).
- `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` → PASS 76/76.
- `node .opencode/plugin/tests/block_transfer.smoke.mjs` → PASS 150/150 (baseline intact).
- `node .opencode/plugin/probes/handover_probe.mjs` → PASS 360/360.

MAIN, after unit 2 (commit `b713585`):
- bt smoke → PASS `BLOCK_TRANSFER_SMOKE: ALL PASS (151/151)` (150 baseline + 1 net new pin).
- sandbox smoke both forms → PASS 76/76 each.
- probe → PASS 360/360. Pre-repin run showed exactly one FAIL: `[307] WRITE 'regions' LIST (2 regions...` → the named repin.

CLONE, after unit 1 (commit `2a3759f`):
- sandbox smoke env-less → PASS 76/76; with env → PASS 76/76; bt smoke → PASS 131/131; probe → PASS 360/360.

CLONE, after unit 2 (commit `4054a0a`):
- bt smoke → PASS `BLOCK_TRANSFER_SMOKE: ALL PASS (132/132)` (131 baseline + 1).
- sandbox smoke env-less and with env → PASS 76/76 each.
- probe → PASS 360/360 (check 307 repinned to the paired + reject forms).

Working tree: MAIN `clean` after the final commit (branch `main`); CLONE `clean` (branch `migration`). Fixture dir `bt_sandbox_env_temp` removed by the smoke's finally (verified by `ls` of the Temp dir).

## Commits

- MAIN `05df70f` — sandbox smoke self-set fixture TEMP env.
- MAIN `b713585` — WRITE regions reject + desc/.md lines + smoke + probe 307 repin.
- CLONE `2a3759f` — clone twin of the smoke env fix.
- CLONE `4054a0a` — clone twin of the regions contract (smoke + probe 307 repin).
- MAIN final commit (with this file) — handover + loop-log DONE + inbox writes.

Probe repins named: check 307 in BOTH trees (MAIN `b713585`, CLONE `4054a0a`). Check 113 untouched.

## Inbox entries submitted (todo channel + others)

- `todo_inbox.md`: MAIN `.opencode/tools/block_transfer.md` staleness (line 7 "creates the file if absent" vs MAIN's "never creates a file"; line 13 omits the config allow roots from TODO #140 unit A) — found while editing the file, not fixed (out of scope, would need a 5th commit).
- `agent/agent_feedback.md`: the spec's pin audit named only check 113; a contract change forces one named repin per tree in the probe.
- `agent/knowledge/knowledge_inbox.md`: the smoke self-set fixture env pattern (gate runs must not read ambient TEMP/TMP).
- `agent/memory/worker/memory_inbox.md`: pin-inventory-before-contract-change + twin message-parity/feedback-format rules.

## Deliberately omitted

- The paired per-region-text alternative end state (the inbox's other option) NOT built, per spec.
- The clone's older WRITE hardening gaps (no empty-`regions` rejection, no hardening-item-7 feedback form) NOT ported — out of task scope.
- MAIN `.md` staleness lines NOT fixed (see the inbox entry).
- The multi-region summary line in the WRITE feedback (`Wrote N regions into ...`) left in place — unreachable while the guard rejects, but it becomes live again if the paired-text alternative is ever chosen; removing it would touch more pin surface than the guard needs.

## Reusable lesson

Pin inventory before a contract change: grep the byte-exact message/feedback strings in EVERY pin file (both smokes and the probe) in BOTH trees; the probe pins what the smokes pin, so each tree needs one named repin. Keep the new error text byte-identical across trees while copying each tree's own feedback-line format.
