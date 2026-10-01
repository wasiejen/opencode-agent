# Worker summary — U1: the recommendation leg on `context_trim` report

Task spec: `agent/handover/handover_task.md` (U1 of the context budget
package, design source `proposals/approved/2026-10-01_context-budget-package.md` §U1).
Commit: LANDED (single code unit; the hash is yours to record — a commit
can never carry its own hash). Files: `.opencode/tools/context_trim.ts`,
`.opencode/plugin/tests/context_trim.smoke.mjs`.

## What changed

1. **Cut-down report form** (the surface default — `report` mode now
   returns it): NUMBERED rows `1..N` from the window start (the number →
   messageID map stays internal — rows carry no id/time), role + mass +
   tool-target detail only (no content echoes), then the `last: N` flag.
   Row shape: `row: <n> <role> <kind>=<mass> [tool=<t> target=<...>]`.
2. **Recommendation block** (cut-down form only; emitted only for a
   proper completed-compaction window — hostGuard + strict summary):
   3 retention options **low / medium / high**, each anchored at a
   MESSAGE ID (stable across pre/post-compaction deliberation) + its
   `keep` count (the tail-mode keepMessages value) + the **read /
   thought / shell mass split** of retained vs dropped + the
   oldest-kept (anchor) message's one-line hint. Shape:
   `option low: msg=<id> keep=<K> retained=<m> (read=<r> thought=<t> shell=<s>) dropped=<m> (read=<r> thought=<t> shell=<s>) hint=<hint>`.
   - low = the current tail head (status quo); high = the earliest
     floor-valid anchor (index 0, omitted when the pre-compaction history
     leaves < FLOOR messages); medium = the midpoint index (omitted when
     it coincides with an end).
   - retained = anchor → just before the compaction row + the
     always-in-window post-summary slice; dropped = strictly before the
     anchor; the marker + summary pair is in neither.
   - Splits = part bytes per category floored /4 (read = `read`+`webfetch`
     tool outputs; thought = text parts; shell = `bash` outputs) — a
     composition estimate; the totals stay the spec's massOf rule
     (provider tokens when present, else bytes/4).
3. **Biggest-droppable list** (cut-down form only): `biggest: <row>=<mass>
   ...` — the top-N (default 5) heaviest finished turns (finished, no
   error — the call-result pairs) in the window, as window row number +
   mass, tie → lower row first (the U2 filter's target list).
4. **Full form unchanged**: the core `reportWindow(dbPath, session,
   target?, opts?)` without opts returns the pre-U1 shape BYTE-IDENTICAL
   (all existing report pins stay green); the surface passes
   `{ brief: true }`. Core opts `{ brief, top }` allow the cut-down form
   with a custom top (the smoke pins top=3).
5. **Tool DESCRIPTION updated** (header doc + tool() description + the
   `mode` arg description): the cut-down form, the recommendation block,
   the `last: N` flag, the biggest list, and the full-form availability
   note.

## Measured verification (runtime: NODE v24.19.0 — bun is broken, see below)

- Baseline re-verified at run (spec's two named re-checks): the
  `report` dispatch line confirmed by grep (L779 pre-U1 — shifted by
  U0's edits as the spec anticipated); the pre-change smoke was 34/34.
- `context_trim.smoke.mjs`: **40/40** (34 pre-existing + 6 new section-11
  pins; the new pins' mass figures are MACHINE-RECOMPUTED from the
  fixture DB in a DB-loading reference — #99 pin style, no hardcoded
  tokens). New fixture `ses_ct_u1`: 12 pre-marker messages with
  non-trivial tool outputs (read 1000/2000, webfetch 600, bash
  400/200/800/300, text 200/100), marker quartet tail_start_id=msg_u07
  (retained=6 = floor), 4 post-summary. Pins: brief byte-exact; full
  byte-identical + ≠ brief; surface default = brief (byte-exact);
  surface brief dry-run (recomputed keepMass=8285); no-marker brief
  (no option block + biggest single turn + last: 3); core top=3.
- Probe: **355/355** (unchanged — the S33 pins drive the core exports and
  the surface arg-key list stayed exactly `[mode, session, target,
  keepMessages]`).
- Standard gate: probe 355/355 + ALL 11 smokes green — auto_resume 155,
  block_transfer.sandbox 64, block_transfer 131, compact_memory 89,
  context_recovery 17, context_trim 40, ctx_gauge 3, gauge_core ALL PASS,
  intercept_observer 85, loop_log 69, submit 31.
- Fixture DBs only — the live opencode.db was never touched.

## TODO entries (appended to `todo_inbox.md`, loose — expectation only)

- The broken `bun` npm shim (the exe is missing; verified at run:
  `/c/Users/Wasiejen/AppData/Roaming/npm/node_modules/bun/bin/bun.exe:
  No such file or directory`) — the whole gate ran under node; the
  maintainer's call to repair/reinstall bun or drop the bun preference.
- The probe pin 346 constraint note (below — for the planner's future
  surface decisions, esp. U4).

## Deliberately NOT done (design decisions, for the record)

- **No new surface args.** Probe pin 346 asserts the exact tool args key
  list `[mode, session, target, keepMessages]` and the gate must stay
  355/355 — so `form`/`top` are CORE-ONLY (`reportWindow` opts). The
  surface `report` ALWAYS returns the cut-down form; the full form is
  available through the core export (smoke/probe shape). If the planner
  wants a `form`/`top` surface flag, that requires a probe-pin change
  (his call — the pin is the gate).
- The recommendation block + biggest list live ONLY in the cut-down form
  (the full form stays byte-identical — the DoD's "existing report pins
  stay green").
- No probe changes (355/355 fixed by the DoD); no TODO renumbering;
  `maintainer/`, the fork, `agent/prompts/**`, proposal files, other
  tools untouched; no live-DB access.

## Environment note (per the spec's instruction)

`bun` is unavailable: the npm shim exists but its `bun.exe` is missing
(vanished from the global install 2026-10-01 ~17:45 per the spec;
verified again at run — exit 127). All verification ran under **node
v24.19.0** (the smoke's documented host).
