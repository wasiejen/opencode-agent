# Worker handover — TODO #137 order 2: the `auto_resume` v2 port

Status: COMPLETE (this unit) — one baseline discrepancy reported, deliberately
not fixed.

## Task and scope

Port the v1 `auto_resume` plugin's LOOP legs onto the v2 plugin surfaces, in the
v2 clone `C:/Users/Wasiejen/Repos/opencode-agent-v2` (branch `migration`), so the
autorun loop works on a v2 host without the CLI. Assigned: `worker_FlashNext`.
Scope: `.opencode/plugins/auto_resume.ts`, `.opencode/plugins/tests/auto_resume
.smoke.mjs`, `.opencode/plugins/tests/tsconfig.json`, `.opencode/plugins/tests
/README.md`, `TODO.md`, `agent/handover/handover_task_to_planner.md`.

## Files changed

1. `.opencode/plugins/auto_resume.ts` (NEW, 1084 lines) — `{ id: "auto_resume
   ", setup(ctx) }`, DEFAULT export only. Legs: UNIT 1 event log (`<ISO> event
   =<type> sid=<sid> <key fields>`, the delta classes never logged) + the
   `surface=` probe + the #96 log-size guard; UNIT 3 successor spawn (the
   trigger file, the FLAT `SessionCreateInput` with `title`/`agent`/`model`,
   ONE `delivery:"queue"` prompt, `spawn=`/`spawn-fail=`, consume-even-on-failure);
   UNIT 4 watchdog (the `context`-hook verdict carrier, the LINE-ANCHORED
   `action:` scan over `session.text.ended` accumulation, stop/ask routes, the
   recovery budget + dead-mark, the successor check, the LINEAGE-DEPTH cap,
   STICKY deactivation, the loop counter + `ident=`, `tailCompactRearm`, the
   #90 part-C log RESTORE); #109 detector (5 clauses, `limit-stop=` + the
   `-WARNING` line in `loop/<folder>/loop_log.md`, the per-episode latch).
   Deviations (a)-(g) in the header block.
2. `.opencode/plugins/tests/auto_resume.smoke.mjs` (NEW, 432 lines, 45 checks).
3. `.opencode/plugins/tests/tsconfig.json` — `files` += `"../auto_resume.ts"`.
4. `.opencode/plugins/tests/README.md` — the new smoke section.
5. `TODO.md` — the `#137` entry with the order-2 status line + the code hashes.
6. This handover.

UNIT 2 (the PASSIVE ctx-line nudge) was NOT re-ported — it already lives in
`plugins/context_family.ts`.

## Behavior changed (measured, per leg)

- Every hook/event handler swallows (fail-open); the tick is the only decision
  +send funnel; `setInterval` is `unref`'d and cleared by the setup cleanup.
- Test levers ride `ctx.options` (`tickMs`, `maxLogBytes`, `logTailBytes`,
  `limitStopIdleMs`); absent → the v1 defaults (5000 / 20 MiB / 2 MiB / 60_000)
  — live behavior unchanged. `runTick()` is the smoke-only single-tick export.
- No `opencode.jsonc` change (auto-discovery); no v2 host start; no backend
  request; the inert v1 estate untouched; `port_sync.cjs` NEVER run.

## Verification (exact commands, run in the CLONE tree)

- `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe -p .opencode/plug
  ins/tests/tsconfig.json` → **PASS** (exit 0, no output). Wiring pinned: a
  deliberate type error appended to `auto_resume.ts` produced
  `error TS2322 … auto_resume.ts(1076,7)`, then reverted → exit 0 again.
- `cd .opencode/plugins/tests && bun run auto_resume.smoke.mjs` → **PASS**
  (`SMOKE: ALL PASS`, `FAILURES=0`, 45 checks).
- `for f in .opencode/plugins/tests/*.smoke.mjs; do bun run "$f" | tail -1;
  done` → **PASS** (11 files, each `SMOKE: ALL PASS`).
- `node .opencode/plugin/probes/handover_probe.mjs` → **PASS**
  (`PROBE handover: 360/360 PASS`).
- v1 node smokes, `for f in .opencode/plugin/tests/*.smoke.mjs; do node "$f"
  | tail -1; done` → **PARTIAL: 11 of 12 PASS**. `state_snapshot` FAILS 4
  checks (`60-line digest` → 59; the fixture `--now` marker line → `(none)`;
  `proposals 24 of 30`). Reproduced with that smoke run ALONE on a clean tree;
  the smoke's own last-touch commit is `6130a70` and this unit's diff is 3
  files (`git diff --stat 90d6d6c..HEAD`) → PRE-EXISTING at baseline
  `90d6d6c`, not caused by this unit.

## Working tree / commits

- Working tree: **clean** after the final commit (`git status --porcelain`
  empty; branch `migration`).
- Checkpoint commits: `d2f7cff` (plugin + tsconfig line), `34b27f8` (smoke
  levers), `accd960` (smoke). FINAL (this bookkeeping): TODO.md status line +
  README section + handover — hash reported below by the planner, NOT here
  (a commit cannot contain its own hash).
- NOTHING pushed (agents never push).

## TODO.md entries

- `#137` entry appended in the CLONE with the order-2 status line + the three
  code hashes; the entry carries the `state_snapshot` baseline discrepancy (with
  evidence) and the two deferred no-carrier v1 legs. MAIN-side curation is the
  planner's own commit (this unit never writes the MAIN tree).
- No new TODO entry was invented for the deviations: the v1 legs with no v2
  carrier are recorded in-file (deviations (c)+(d)) and in the #137 entry.

## Deliberately omitted

- UNIT 2 (already ported); the #120 TAIL-SET write lane (the v2 twin
  `context_trim_v2.ts` owns that surface — a separate unit); the
  `compact_message_*` relay/intent/sweep legs (no v2 file store — the native
  inbox queue); abort escalation, subagent special-casing, busy-silence stall
  detection (v1 "DELIBERATELY ABSENT" list); the `state_snapshot` smoke repair
  (out of scope, own unit); NO `opencode.jsonc` edit; NO `switchAgent`/`switchM
  odel` (would be a behavior change → a maintainer call).

## Blockers

None for the code. LIVE acceptance is PENDING by design: it needs the maintainer
's v2 host runs (the backend slot); registration/config stays maintainer domain.

## Deviation from the suggested procedure (explain)

The spec listed `spawn=`/`actionRe`/`limitStopCheck` greps against the v1 file
plus its header block; I read those slices ONLY (never the whole 2191-line file)
and additionally confirmed each v2 surface in the fork (the status-event key,
`session.text.ended`, `session.step.ended` finish+tokens, `SessionCreateInput`,
`SessionPromptInput`, `ModelDomain`, `EventDomain.subscribe`, `App`) so no leg
rests on assumption. Two findings differ from the spec's stated facts and are
recorded, not patched over: (1) the v2 `session.status` event (NOT the deprecated
`session.idle`) is the arm/idle carrier — the plugin accepts both; (2) the
`state_snapshot` baseline RED above.

## Friction

- The spec's `SessionCreateInput` sketch (`id?: { … }`) reads as a NESTED object
  ; the generated type is FLAT (each field declared via the `{ <field>: <type> }
  ["<field>"]` idiom, types.ts:3064-3134). The smoke pins the FLAT body; the
  plugin header records it. Worth a knowledge-line from the planner/curation.

## Lessons

- A baseline gate claim must be re-measured per suite and per file: a RED that
  pre-dates the unit must be REPORTED with its own evidence (last-touch commit,
  alone re-run, diff stat), not absorbed into the unit's gate.
