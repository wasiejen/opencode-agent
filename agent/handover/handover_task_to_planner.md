# Worker summary — TODO #137 unit 4a: compact_memory + U3 marker compaction as a v2 plugin

Status: COMPLETE (all DoD items landed; the full clone gate green)
Assigned: worker_FlashNext · Repo: `C:/Users/Wasiejen/Repos/opencode-agent-v2` branch `migration`

## Change summary

Three new files in the clone, no existing plugin touched:

1. `.opencode/plugin/scripts/compaction_core_v2.mjs` — the budget layer ported
   from v1 `compaction_core.ts` (plain JS, node:fs/path only, the gauge.mjs
   sibling contract). Same `.opencode/temp/compact_budget.json`, same keys,
   same semantics: fail-open config reader, cap resolver (CPU guard cap 0 FIRST,
   bare model id, unlisted → `model_budget.default`, else 1), budget-store v2
   schema + the #119 additive-rewrite invariant, `budgetCount`, `recordSuccess`
   (INCREMENT-ON-SUCCESS), `localStamp`, `appendCompactLine` (byte-identical v1
   COMPACT line incl. ` emergency` / ` ovr` tokens), plus a NEW pure
   `budgetGate` (the v1 arithmetic incl. the emergency-1 and the caller-scoped
   CROSS cap+1 override, TODO #128) so the plugin stays thin and the smoke pins
   the forms directly. `hasKeepKeys` added (the keep-not-applied audit read).
2. `.opencode/plugins/compact_memory.ts` — the v2 plugin (`{id, setup}` DEFAULT
   export; TYPE-ONLY `@opencode/plugin`; VALUE import of the paired `.mjs`
   core). Registers `ctx.tool.transform` → `editor.add` (name
   `compact_memory`, plain JSON-schema input
   `{sessionID?, emergency?, message?, mode?, summary?, boundary?}`,
   `output:{type:"string"}`, promise `execute`), `ctx.session.hook("compaction",
   …)` (U3 marker `result` override), and consumes `ctx.event.subscribe()`.
   Dispatch = `ctx.session.compact({sessionID})` fire-and-forget; the
   increment + COMPACT line land ONLY on the `session.compaction.ended` event
   for a session with a PENDING dispatch record (`.failed` → one evidence line,
   NO burn; auto/maintainer compactions WITHOUT a pending record are silent,
   no burn). Evidence lines → `.opencode/temp/compact_v2.log` (the context
   family capped-log shape: kinds `dispatch/marker/verify/keep/dump/queue`).
3. `.opencode/plugins/tests/compact_memory.smoke.mjs` (+ the tests
   `tsconfig.json` `files` entry) — 41 pins, mock ctx, scratchpad fixtures.

## Deviations landed (all recorded in the plugin header)

- VERIFY leg = the EVENT leg (pinned in the smoke); the gauge-paired DB-read
  fallback was NOT needed.
- U3 marker: NO boundary control in v2 → `boundary` accepted, ignored, ONE
  `boundary-ignored` line; the plugin sets `event.result =
  {summary, metadata:{marker:true}}`; the durable compaction message is
  written by the CORE (the v1 `markerSet` quartet DB-write lane has NO v2
  counterpart).
- QUEUE leg = native `session.prompt({…, delivery:"queue"})`, delivered AFTER
  VERIFIED SUCCESS (keeps the v1 zombie-guard guarantee: queue present ⇒ the
  compaction happened); NO temp-file queue store (auto_resume is not ported).
- KEEP keys: not applied (the compact body has NO keep fields) — read for
  auditability + ONE `keep-not-applied` line; retention = config
  `compaction.keep.tokens`; no config written.
- DUMP leg (#152): skipped, ONE `dump-skipped` line per dispatch (the v2 dump
  twin rides spec 2026-10-22_137u4c-dump-twin).
- The v1 core module stays UNIMPORTED by v2 code (inert-dir contract).

## Measured verification (exact commands, all run from the clone root)

| command | result |
|---|---|
| `node .opencode/plugin/probes/handover_probe.mjs` | `PROBE handover: 360/360 PASS` — PASS (unchanged) |
| `bun run .opencode/plugins/tests/compact_memory.smoke.mjs` | 41 PASS lines, final `SMOKE: ALL PASS`, exit 0 — PASS |
| `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p .opencode/plugins/tests/tsconfig.json` | exit 0, zero error lines — PASS (the plugin added to `files`) |
| `bun run .opencode/plugins/tests/smoke.mjs` (v2 intercept) | 52 PASS, `SMOKE: ALL PASS` — PASS |
| `bun run .opencode/plugins/tests/context_family.smoke.mjs` | 46 PASS, `SMOKE: ALL PASS` — PASS |
| `node .opencode/plugin/tests/<each>.smoke.mjs` × 12 | ALL PASS (auto_resume 170/170, block_transfer.sandbox 76/76, block_transfer 132/132, compact_memory(v1 dir) 110/110, context_recovery 17/17, context_trim 40/40, ctx_gauge 3/3, gauge_core, intercept_observer 85/85, loop_log 69/69, state_snapshot 20/20, submit 31/31) |

Note: three node smokes (`auto_resume`, `context_trim`, `gauge_core`) print a
bun banner as their last line when run under `bun` (they need `node`); under
`node` they read ALL PASS. Recorded as an observation, NOT a failure — no file
was changed for it.

Live firing: NONE. No request was made to the backend inference server; the
live opencode DB was never opened (scratchpad fixture budget file + logs only).

## Clone commits (tree CLEAN)

- `afd6b26` Port the compaction budget layer to v2 (compaction_core_v2.mjs)
- `ae2f36f` Add the v2 compact_memory plugin: tool dispatch, verify-gated
  budget, U3 marker hook, native queue (TODO #137 unit 4a) — plugin+smoke
  bundled (a plugin-only commit would be unverified: the 41-pin smoke is its
  DoD evidence)
- `358cba7` Document the compact_memory v2 smoke in the plugins tests README

## Working-tree status

- Clone `opencode-agent-v2` @ `migration`: CLEAN.
- Main `opencode-agent` tree: DIRTY by design — this file + the loop-log
  DONE line are uncommitted (this task's instruction: no commits in the main
  repo; the planner lands the bookkeeping).

## TODO / inbox items

- No new `TODO.md` entries; #137 unit 4a is DONE (the planner records status
  + hashes).
- `submit` (feedback channel, 2 lines) sent: (a) `block_transfer` REPLACE/WRITE
  with `bufferName:"last_write"` silently pasted a STALE foreign buffer and
  corrupted my file (repaired by rewrite; the buffer is session-persistent,
  never request it by name unless you just wrote it); (b) a ~4.5 KB single
  bash heredoc append was TRUNCATED mid-line (append ≤ ~4 KB per call, or
  use the write/edit tools).
- Deferred by spec: #152 dump twin (spec 2026-10-22_137u4c-dump-twin), unit
  4b (context_trim v2 report), auto_resume v2 (port order 2), live v2
  acceptance.

## Deliberately omitted

- The pre-compaction DUMP leg (queued follow-up spec) — `dump-skipped` lines
  only.
- No config writes (maintainer domain); no `opencode.jsonc` change (auto
  discovery).
- No live host activation (waits for the maintainer's v2 runs).

## Lessons (reusable)

Pin the verification leg you actually wire: the v2 event leg
(`session.compaction.ended` / `.failed`) is deterministic under a mock
AsyncIterable, so the DB-read fallback was unnecessary and stays unported —
the smoke pins the leg that landed.
