# Spec — state_snapshot digest: live-class marker hits sort first

Task id: idea-driven (agent_ideas.md entry 2026-10-22_04-38), no TODO id.
Pre-approved class: agent-facing tool digest shape, no maintainer-visible behavior change.

## Goal

In the `## markers` section of the planner init digest, marker-sweep hits from
LIVE-CLASS paths print BEFORE DOC-CLASS hits, so live maintainer/TODO markers are
never crowded out by the 20-line cap. Nothing else changes.

## Definition of done

- `markersSection` partitions the sweep lines into LIVE-CLASS (path matches
  `maintainer/` prefix, `TODO.md`, `README.md`, `WIKI.md`, or `.opencode/` prefix)
  and DOC-CLASS (everything else), sorts each group by the line string, and emits
  LIVE first, then DOC, then the existing cap + `… (+<k> more)` steer unchanged.
- No new output line shape; no new header/steer text; the sweep command itself
  stays byte-verbatim.
- MAIN smoke + clone smoke each gain ONE pin: a LIVE-path fixture marker line
  appears in the markers section BEFORE the `loop/notes.md` DOC fixture line.
- Verification (both trees): the state_snapshot smoke ends `ALL PASS` with
  the pin count grown by exactly the pins added; MAIN v1 gate 12/12 suites
  PASS; the MAIN probe count captured BEFORE the edit stays identical after it
  (exit 0); clone v2 bun smokes ALL PASS + tsgo exit 0.
- Checkpoint commits per verified unit (MAIN, then clone); `TODO.md` +
  `agent/handover/handover_task_to_planner.md` ride the final commit;
  status `LANDED` (hash recorded by the planner, never in the worker commit).

## Verified facts (measured 2026-10-22, this session)

- Sweep total: 143 hits = 18 LIVE-CLASS + 125 DOC-CLASS (awk prefix test).
  The digest today shows ZERO live hits inside the 20-line window (`… (+123 more)`).
- MAIN `.opencode/tools/state_snapshot.ts` (206 lines): `MARKER_SWEEP` const
  27-30; `markersSection` 141-170 (cap line 158: `const body = lines.slice(0, 20)`);
  buildDigest entry 174.
- Clone twin `.opencode/tools/state_snapshot.ts` is BYTE-IDENTICAL to MAIN pre-change
  (`cmp -s` PASS, 206 lines) — it must stay byte-identical after the change.
- Clone v2 twin `.opencode/plugins/state_snapshot.ts` (265 lines): `MARKER_SWEEP`
  const at 47; `markersSection` 166-190 (cap line 184, semicolon style);
  buildDigest entry 200.
- Pin surfaces: MAIN `.opencode/plugin/tests/state_snapshot.smoke.mjs` — pins
  `10` at 211-217 and `11` at 219-225 read the markers section; pin `19` (line 284+)
  is the FX3 60-line cap arithmetic — DO NOT touch FX3 or its fixture marker-line count.
  Clone `.opencode/plugins/tests/state_snapshot.smoke.mjs` — pins `(9)` 173-175,
  `(10)` 176; FX3 pins `(21)` 240, `(22)` 242 — DO NOT touch FX3 there either;
  the FX1 fixture block writes `loop/notes.md` at 131.
- Probe `.opencode/plugin/probes/handover_probe.mjs` S35 block at 8578-8600 pins
  the SURFACE only (file exists + `tool()` default export + empty args key list
  + the description steer). Do not change the tool description or args → the
  probe count must stay unchanged; capture it before editing.
- `.opencode/tools/*.ts` is host-registered at the NEXT host start (no
  registration step, no live acceptance in this unit; the digest behavior is
  smoke-pinned).

## Suggested scope (non-exhaustive)

- `.opencode/tools/state_snapshot.ts` — `markersSection` + its header comment
  (the `grep -v` filter note near line 12).
- `.opencode/plugin/tests/state_snapshot.smoke.mjs` — add a FX1 fixture file
  `maintainer/live_note.md` carrying a `--main` line (safe: `maintainerSection`
  reads only `maintainer/inbox_planner/` and `maintainer/priority.md`), + 1 pin.
- `agent/readme/repo_commands.md` §Marker sweep — one note: the digest orders
  LIVE-CLASS paths first; the ready-made command itself is unchanged.
- Clone: `.opencode/plugins/state_snapshot.ts`, `.opencode/tools/state_snapshot.ts`
  (copy of MAIN), `.opencode/plugins/tests/state_snapshot.smoke.mjs`,
  `agent/readme/repo_commands.md`.

Deliberate non-change: NO `doc-quotes: N` counter line (keeps every existing
cap/steer pin green; doc hits stay visible in the tail of the window).

## Assigned worker

worker_FlashNext (model `llama-swap/Qwen3.8-NEXT-Coder` — same as the planner).
