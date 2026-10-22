# Worker summary — TODO #137 unit 3a: the v2 context-family plugin core (clone `migration`)

Status: COMPLETE (the unit as specced; the live v2-host acceptance of the ctx
line stays PENDING maintainer restart, per the spec).

## Task and scope

Port the context-family core of the v1 `.opencode/plugin/ctx_watchdog.ts` into
ONE new v2 plugin in the clone `C:\Users\Wasiejen\Repos\opencode-agent-v2`
(branch `migration`, baseline `4054a0a` clean): the gauge readout through the
shared v2-paired gauge core + the per-request `ctx:` line injection at the
v2 `session.context` hook with cache discipline + evidence log lines. The
nudge ladder and the handover pre-flight ride spec 3b (same file, next
launch) — deliberately absent.

## Files changed (IN THE CLONE ONLY)

- NEW `.opencode/plugins/context_family.ts` — default export `{ id:
  "context_family", setup }` ONLY (the v2 loader contract); registers
  `ctx.session.hook("context", …)` (fires per PRIMARY request —
  `model-request.ts:422-431`); `readGauge(undefined, requestOwnSessionID)` →
  `formatGauge` + `loopIterationSuffix`; APPEND-ONLY tail entry
  `{ role: "user", content: [{ type: "text", text: "ctx: …" }] }`;
  match-only own-session gate (mismatch → no post, no line); evidence
  lines to `.opencode/temp/ctx_family.log` (v1 vocabulary: kinds `ctxmsg`
  / `gauge`, reasons `db-error` + `messages-not-array`; the v1
  `invalid-messageID` branch has no v2 counterpart — the draft carries no
  messageID — recorded, unemitted).
- NEW `.opencode/plugins/tests/context_family.smoke.mjs` — 24 checks, bun
  + mock ctx, fixtures in the scratchpad
  `C:/Users/Wasiejen/AppData/Local/Temp/opencode/ctx-family-smoke-v2`
  (fixture `session_v2` + `session_message` DB, budget + loop-root files;
  the live DB never read; the gauge core steered via `setDbPath` /
  `setBudgetFileForTest` / `setLoopRootForTest`).
- MOD `.opencode/plugins/tests/tsconfig.json` — `files` now
  `["typecheck.ts", "../context_family.ts"]`; `paths` gained
  `"@opencode/ai": ["C:/Users/Wasiejen/Repos/opencode/packages/ai/src/index.ts"]`
  (needed so the plugin's TYPE-ONLY `Message` import resolves for tsgo —
  same file, machine-local test infra).
- MOD `.opencode/plugins/tests/typecheck.ts` — pins BOTH plugin defaults
  against `Plugin.Plugin`.
- MOD `.opencode/plugins/tests/README.md` — a short new section for the
  new smoke (docs).

## Behavior changed

New v2 plugin module + its smoke + the typecheck config. No existing file's
behavior changed; no live host effect yet (plugin registration/restart =
maintainer domain; `opencode.jsonc` untouched).

## The bounded shape question (answered, measured)

The injectable entry is `@opencode/ai`'s `LLM.Message` —
`packages/ai/src/schema/messages.ts:237-244` (a `Schema.Class` over Struct
fields; instance type = plain structural fields: `role` required,
`content: Array<ContentPart>` required, `id` optional). The append form
`{ role: "user", content: [{ type: "text", text }] }` typechecks against the
REAL type (a TYPE-ONLY import of `Message` — erased at runtime, the clone
has no node_modules). NEGATIVE CONTROL MEASURED: a deliberately bogus
push (`role: "bogus-role", content: 5`) turned tsgo RED with
`Type '"bogus-role"' is not assignable to type '"assistant" | "system"
| "tool" | "user"'` + the content-part union error → the pin is real
(not an `any` widening). An earlier GENERIC attempt was the failed
approach (push into `Array<unknown>` = no check); replaced by the typed
import before commit.

## Measured verification (2026-10-22, run in the clone)

- New smoke: `bun .opencode/plugins/tests/context_family.smoke.mjs` →
  24 `PASS` lines, 0 `FAIL`, final `SMOKE: ALL PASS` → PASS.
- Typecheck: `C:/Users/Wasiejen/Repos/opencode/node_modules/.bin/tsgo.exe
  -p C:/Users/Wasiejen/Repos/opencode-agent-v2/.opencode/plugins/tests/tsconfig.json`
  → 0 bytes of output = ZERO errors → PASS.
- Clone gate UNCHANGED (15 units, `green=15 red=0`): `node
  .opencode/plugin/probes/handover_probe.mjs` → `PROBE handover: 360/360
  PASS`; all 12 node smokes (`auto_resume`,
  `block_transfer.sandbox`, `block_transfer`, `compact_memory`,
  `context_recovery`, `context_trim`, `ctx_gauge`, `gauge_core`,
  `intercept_observer`, `loop_log`, `state_snapshot`, `submit`) → `ALL
  PASS`; the existing v2 intercept smoke `bun .opencode/plugins/tests/smoke.mjs`
  unchanged → `SMOKE: ALL PASS` (47 checks + the unit pins); the new smoke
  → `SMOKE: ALL PASS`.
- Baseline re-measured BEFORE unit A's commit (green=14 red=0 pre-smoke) and
  AFTER every commit; identical.
- Working tree: CLEAN in the clone after each commit (`git status --short`
  empty). MAIN repo: only the UNCOMMITTED handover file (this file) + no
  other change; MAIN repo untouched otherwise.

## Commits (IN THE CLONE, branch `migration`)

- `1174d98` — Add v2 context_family plugin: session.context ctx-line injection
  (plugin + tsconfig `files`/`paths` + typecheck pins).
- `f149745` — Add the context_family v2 smoke: 24 pins (+ tests README
  section). Code-only per the #141 precedent (no handover file in the clone).

## Deviations from the suggested procedure (recorded)

1. Evidence per-fire ordering: v1 logged its `chatmsg` line BEFORE the db-error
   check (two lines); the spec DoD 5 demands ONE line on `db-error` → the
   per-fire line is suppressed on gauge-failure branches. Documented in the
   plugin header.
2. DoD 5 vocabulary kept: `db-error` + `messages-not-array` (the
   `parts-not-array` analogue on the draft array). No new reason invented.
3. The `messages` param is typed against `@opencode/ai` `Message` (TYPE-ONLY
   import + tsconfig `paths` addition) instead of the spec's guessed
   generic-inference route — measured: the generic route silently accepted a
   bogus literal (no pin). Fail-closed is preserved by tsgo.
4. `typecheck.ts` pins both plugin defaults (the spec only required the tsconfig
   `files` extension) — kept the existing file's purpose, extended it.

## TODO / inbox entries

- None written into `TODO.md` (worker ownership: the planner records the #137
  status line with its bookkeeping commit).
- Two findings submitted through `submit` (feedback + knowledge) — see the
  friction lines below; both are tooling facts, not repo defects.

## Deliberately omitted (per spec)

The nudge ladder + watchdog handover pre-flight (spec 3b, same file, next
launch); compact_memory / context_trim / U3 ports (port map §4 order 4);
the small tools (order 5-8); LIVE v2-host acceptance of the ctx line
(backend slot = maintainer domain) — STATE: PENDING; the v1-side event
observer log (no v2 role — retired-in-v2 for this family).

## Blockers

None.

## Lessons

- On win32 Git-Bash, node cannot spawn the `bun` PATH shim (ENOENT) and
  `bun.cmd` needs `shell: true` (EINVAL otherwise); run bun units through
  bash, or spawn `bun.cmd` with a shell (measured 2026-10-22).
- A generic `E extends Array<unknown>` hook param silently widens to
  `Array<unknown>` — for a real type pin, name the fork's type
  (`@opencode/ai` `Message`) with a TYPE-ONLY import and verify with a
  negative control.

## Friction (recorded here, NOT submitted)

The spec restricts MAIN-repo writes to this summary file, so the
`submit` channels (which write into the MAIN repo's inbox files) and the
loop-log `DONE` line were deliberately WITHHEPT — the planner owns that
bookkeeping. Friction found: (1) no aggregate clone-gate runner exists, so
the 14-unit gate was hand-rolled (a scratchpad bash runner
`$TMP/opencode/run_gate.sh` did the job; a ready-made aggregate runner would
save calls); (2) the win32 bun-spawn surprise cost ~6 calls of runner
debugging (fact above); (3) the illusory typecheck pin (zero errors while a
bogus literal was accepted) cost one extra iteration before the negative
control confirmed the REAL pin.
