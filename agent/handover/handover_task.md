# Task Spec — OpenCode v2 impact research (TODO #136)

## Goal
One research document that maps what OpenCode 2.0 changed and what it means for this
repo's stack: the shared-service architecture, the v1→v2 DB, the new compaction/
assembly, the new plugin API, a per-feature overlap audit of our 6 plugins + 8 tools,
the config carry-over for our actual files, and the upgrade path — to drive (a) the
plugin porting order, (b) the U2 per-message filter design ON v2, (c) the live-host
upgrade decision.

## Sources (all read-only)
- Fork `C:\Users\Wasiejen\Repos\opencode` — checked out on `production`
  (654b79b27e, the v2.0.29 generation; `bun install` done, `exact=true` mode).
  Work against THIS checkout; no fetches needed (`dev` = 2.0 line slightly ahead,
  local, useful for a diff if a symbol moved).
- Official v2 docs — `https://opencode.ai/v2/docs/` (migrate-v1, the dedicated
  plugin-migration guide, compaction, permissions). Webfetch what you need.
- The migration guide copy: `maintainer/done/2026-10-15_migration_from_v1.md`.
- Our repo: `.opencode/plugin/*.ts`, `.opencode/tools/*.ts`, `opencode.jsonc`,
  `agent/research/fuzzy-numword/` (decision-record.md — the intercept design),
  `agent/research/2026-10-01_magic-context-plugin.md`, TODO #135.
- The v2 clone `C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch `migration`,
  standalone `opencode.exe` v2.0.29 in its root) — READ-ONLY reference only.

## Sections (the document's outline)
0. **Intercept-plugin port analysis (PRIORITY — the maintainer's first port target).**
   The intercept observer (`.opencode/plugin/intercept_observer.ts`, 1555 lines)
   survives on: fuzzy path resolution (read-scoped d≤2; write-scoped d≤1 for
   edit/block_transfer, `write` excluded per M1) + the observation log
   (`intercept.log`). SCOPE CUT (maintainer ruling 2026-10-15): the `[left:right]`
   PAIR-RESOLUTION channels are DROPPED entirely, incl. the numword anomaly class
   and the pair-based git-refs-in-bash channel (3c) — flagged here as an
   assumption: confirm against the header, correct the doc if the git-refs
   channel was meant to stay. Answer: (a) does the v2 plugin API expose a
   `tool.execute.before`-equivalent that can MUTATE tool args before execution
   (read `packages/plugin/src` — `define()`, effect/promise surfaces, hook
   registration)? (b) the v2 tool pipeline — `packages/core/src/tool/plugin/*.ts`
   (edit.ts verified: exact match, Location-relative resolution, external-path
   approval, typographic match normalization) — where does fuzzy resolution plug
   in: plugin hook vs fork change in the core tool pipeline? Recommend one.
1. **Architecture:** the shared background-service model; package layout
   (core/server/plugin/tui/cli/session-ui/llm/codemode); what runs where; the
   process model — how v2 calls providers (the llama-swap single-slot setup must
   survive; flag any new subprocess that would hit the backend).
2. **DB v1→v2:** v1 format (session/message/part rows, `data` blobs,
   `tail_start_id`) vs the v2 schema (`packages/core/src/database/schema.gen.ts`,
   drizzle) — what `v1-migration.bun.ts` maps; do message-level `data` blobs
   survive (the U2 filter-store candidate); the DB file path
   (`database/path.ts`) — same location as v1?; is the migration one-way/destructive;
   what happens when v2 opens our live v1 `opencode.db`.
3. **Compaction & assembly:** `SessionCompaction` (checkpoint-based compaction),
   the request runner / `SessionModelRequest`, the `compaction.transform` hook
   (verified settings-level: auto/buffer/keep). Where are model-call messages
   assembled, and what are the candidate insertion points for the U2
   per-message filter ON v2 (plugin vs fork)?
4. **Overlap audit + port map:** for each of our features, find the v2 native
   equivalent (file pointer) and verdict it native / partial / absent, with a
   port effort S/M/L and order: auto-resume, ctx gauge + injected ctx line,
   nudge ladder (our `compact_budget.json` nudge1-4), compact_memory (budgeted
   self/cross compaction + queue relay), context_trim (report/tail/keepMessages),
   U3 marker compaction, intercept observer (see §0), ctx watchdog, loop_log,
   block_transfer, state_snapshot, submit, session_info,
   dev_get_tool_context_contents.
5. **Config carry-over:** field-by-field delta for OUR actual config files
   (root `opencode.jsonc` + `~/.config/opencode/`): permissions array
   (bash→shell, task→subagent), agents, providers (the llama-swap provider),
   plugin→plugins, compaction keep/buffer, `autoupdate`→`update`, and anything
   under `experimental/` that our plugins depend on.
6. **Upgrade path + risks:** the current side-by-side (npm v1.19.0 global +
   standalone v2 exe in the clone), `opencode upgrade --major`, no auto-updates
   in v2, the llama-swap serial-session discipline across two workspaces, live-DB
   migration timing, AGENTS.md/agents carry-over, the ordered-permissions model.
7. **Recommendations:** port order (intercept first), the U2-on-v2 design choice
   (plugin vs fork, insertion point), the upgrade timing recommendation, and what
   we retire from v1.

## DoD (measured end states)
- One document: `agent/research/2026-10-15_opencode-v2-impact.md`, committed on
  the current checkout of THIS repo (verified at spec authoring: `main` — the
  maintainer rebranched 2026-10-15 and dropped the old local branches; verify
  with `git branch -v` before committing, never assume the name).
- Every non-obvious claim carries a file pointer (path + line) into the fork or a
  doc URL; measured facts are labelled as such.
- §0 answers the arg-mutation hook question explicitly (yes/no + the API shape).
- §4 is a table: feature → v2 equivalent pointer → verdict → effort → order.
- Worker summary to `agent/handover/handover_task_to_planner.md` (change summary,
  key findings per section, the one recommendation block, working-tree status,
  commit hash).

## DO-NOT-touch
- The fork repo, the v2 clone, the live `opencode.db`, the installed exes,
  `maintainer/`, `opencode.jsonc` — all READ-ONLY.
- NO live v2 run (never start `opencode.exe` a session), NO requests at the
  llama-swap backend (agent rule — sessions only), NO installs, NO pushes.
- TODO entries: expectation only, never a promise.

## Assigned worker
explorer_Q3S_slow (effort L — large tree, doc-heavy; 245k window for lookups).
