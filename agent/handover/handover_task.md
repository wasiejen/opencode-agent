# Task Spec — Unit A: the intercept observer ported to v2 (TODO #137)

## Goal
A v2-native intercept observer in the V2 test workspace, implementing the
surviving feature set. This is the V2 side of the dual-support strategy (one
plugin serving both hosts); the v1-side reduction + dual merge is a LATER unit
on the main repo — the main repo is READ-ONLY here.

## Working repo
`C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch `migration`) — commit there.
This spec lives in the MAIN repo (`C:\Users\Wasiejen\Repos\opencode-agent`,
`agent/handover/handover_task.md`) — read it from there. Your worker summary
goes to the CLONE's `agent/handover/handover_task_to_planner.md`.

## Design sources (read first)
- `agent/research/2026-10-15_opencode-v2-impact.md` (main repo) §0 (the hook API,
  the `path` field names, the discovery paths) and §4 (the intercept row).
- The v1 source of truth: `intercept_observer.ts` header L1-80 (the channel map)
  + the R8/redirect/sandbox/journal sections (grep `REDIRECT_PATH_FIELDS`,
  `HardStopError`, `journal_`, `underRoot` in the main repo's
  `.opencode/plugin/intercept_observer.ts` + `intercept_observer_core.ts` —
  bounded reads, the file is 1555 lines).

## Surviving feature set (maintainer ruling 2026-10-15 — PAIR/NUMWORD DROPPED)
- Observation log to `.opencode/temp/intercept.log`: surviving anomaly classes —
  path-anomaly (the C doubled-segment detector with the JSON-escaped `\\`
  collapse), out-of-sandbox note (`underRoot`), the `intercept-error` line; the
  write/edit journals survive. The PAIR (`[left:right]`) and NUMWORD classes are
  removed — no logic, no log lines, no verdicts for them.
- READ-scoped fuzzy resolution: v2 `read` tool, the `path` arg, d≤2 + gap rule,
  the strict existence gate (mutate only when the canonical EXISTS and the typed
  path does NOT), fail-closed, `fuzzy-resolved` / `fuzzy-rejected` lines.
- WRITE-scoped fuzzy resolution: v2 `edit` tool `path` arg, d≤1, same strict
  gate, `scope=write` flag. The v2 `write` tool is EXCLUDED (M1 — new-file
  intent). `block_transfer` does not exist in v2 yet — extend when it ports.
- B' hard-stop: the EFFECTIVE (post-redirect) write/edit `path` outside every
  allowed root AND containing directory missing → the hook throws (fails the
  tool call); log ONE `intercept-error` line, then re-throw (the hook's catch
  must distinguish the deliberate throw — v1 semantics, finding B in TODO #134).
- R8 typed-path redirect: port the v1 semantics (the `REDIRECT_PATH_FIELDS`
  list, fail-closed pass-through on unmappable forms, the redirect suppresses
  the duplicate out-of-sandbox note) with v2 field names.

## Verified facts (planner-verified 2026-10-15 — do not re-derive)
- v2 hook: `ctx.tool.hook("execute.before" | "execute.after", (event) => …)` —
  `event.input` is mutable and the core executes the post-hook value (fork
  `packages/core/src/tool.ts:271-280`; shipped example `tool-input-repair.ts`).
  The exact import/shape (promise vs effect surface) is pinned in the research
  doc §0 — use what it says.
- v2 tool arg field is `path` (not `filePath`) for read/edit/write (doc §0).
- v2 discovers local plugins from BOTH `.opencode/plugin/` and
  `.opencode/plugins/` (migrate-v1 guide) — the unported v1 plugin files in the
  clone (6 of them) must not break the v2 boot.
- Allowed-root set (v1 basis, TODO #134): workspace + scratchpad + the
  `external_directory` allows. The v2 source of roots/permissions (the ordered
  permissions array / Permission service) is pinned in doc §0/§6.
- Fork `C:\Users\Wasiejen\Repos\opencode` on `production` = read-only reference;
  its installed workspace provides `@opencode/plugin` for the smoke import.
- The standalone v2 binary sits in the clone root (`opencode.exe` v2.0.29) —
  you NEVER run it.

## DoD (measured end states)
- New v2 plugin file in the clone under `.opencode/plugins/` (the v2-preferred
  dir) + the clone `opencode.jsonc` gains the minimal v2 `plugins` entry
  pointing at it (your edit — it is the clone's test config).
- The unported v1 plugin files do not break the v2 boot — your mechanism
  (e.g. rename them out of discovery on the `migration` branch), documented.
- v2 smoke under bun: mock-`ctx` unit tests of the surviving channels — read
  fuzzy resolved/rejected + gate, edit write-scope d≤1, write-tool exclusion,
  the B' throw + its one log line, the out-of-sandbox note, the C doubled-segment
  with JSON-escape collapse, and ZERO pair/numword lines. Import
  `@opencode/plugin` from the fork's installed workspace (mechanics your call).
  If standalone mocking proves infeasible: a typecheck of the plugin against the
  `@opencode/plugin` types + the live-acceptance path below, stated in the
  handover.
- Live acceptance (MAINTAINER, after you): the handover names the exact test
  script — one scripted v2 session in the clone (a read with a near-miss path, a
  write outside the sandbox to a missing directory) → the `intercept.log`
  lines verified by him.
- Checkpoint commits per verified unit IN THE CLONE; TODO + handover in the
  final commit (clone). The main repo's standard gate is untouched (state that
  in the handover).

## DO-NOT-touch
- The MAIN repo (read-only), the fork (read-only), the live v1 `opencode.db`,
  the installed v1 exe, the live v1 host (no restarts), the clone's
  `opencode.exe` (never run), NO requests at the llama-swap backend except via
  opencode sessions, NO pushes.
- TODO entries: expectation only, never a promise.

## Worker
worker_Q3S_slow (effort M — one plugin port, two surfaces (hook + log), smoke +
config; the design is settled in the research doc).
