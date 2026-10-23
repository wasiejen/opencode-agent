# plan25 summary — planner-25 (ITER=25, ses_e9b75d138ffepge05jIQDjcRnU)

Two units landed this session.

## Unit 1 — ITER-25 maintenance pass (commit c039561)

- `knowledge_inbox.md` EMPTIED: 4 entries cured → `knowledge_tools.md`, `plugin_tools/2026-10-16_v2-plugin-harness-and-config.md`, `opencode-plugins/host-map.md` (new §8 host provider/model snapshot).
- Planner memory inbox EMPTIED: MEM-0129 (block_transfer line-number spans), MEM-0130 (planner-direct close discipline), MEM-0131 (discriminating spec fixtures) + destill lines.
- Worker memory lane EMPTIED: WM-11 (byte-exact pinning), WM-12 (discriminating pins) + destill lines.
- `todo_inbox.md` re-verified: no uncured entries.
- `TODO.md` #144 header fixed (stale OPEN → CLOSED); NEW #145 filed (intercept.log per-event filter helper, idea 00-36).
- Marker triage: 148 hits, 18 live-class — NO new maintainer markers to act on.

## Unit 2 — marker-sweep doc-noise filter + digest cap raise (idea 2026-10-22_04-38)

Root cause measured: at the old 60-line digest cap the `## markers` section was always starved (git ~21 + nap ~42 lines alone exceed 60) — the plan24 session's own digest carried 0 marker lines.

Fix, both trees:
- `MARKER_SWEEP` grep -v now drops the DOC trees that only QUOTE the markers table, PATH-ANCHORED (`^agent/prompts/ … ^maintainer/feedback/` — 12 prefixes). A marker word inside another file's text never hides a hit (pinned by the inline-quote fixture).
- Digest hard cap 60 → 150 (constant + tool description + comments).
- MAIN: commit 60a86cf (`tools/state_snapshot.ts` + smoke 21→23 pins + `repo_commands.md` sweep text).
- CLONE: commits 9fa7d88 + (twin) — v2 plugin twin + inert v1 twin byte-identical + smoke 24→26 pins + README line.

Verification (all green, measured):
- MAIN: state_snapshot smoke 23/23; probe 360/360 (pre-change run, MAIN tool comment-only edits after — the probe does not pin the comments).
- CLONE: smoke ALL PASS (26 pins incl. (23)/(24)/(25)), tsgo clean, probe 360/360, 10/10 clone smokes green, twin `cmp -s` identical.
- Post-change sweep: 17 hits (was 148), live-class content first.

Pending: live acceptance of the updated tool rides the NEXT host start (the tool loads at host start; this session's own digest ran the pre-change live tool).

## Friction

The block_transfer line-number-span slip (MEM-0129) re-fired twice this session (36/44 for 236/244; 43 for 243): one clobbered a smoke region (recovered via `git checkout --`), one left a 1-line orphan comment (deleted). Both caught by reading back immediately after the write.

## Queue (carried)

His decisions: auto_resume v2 port (HELD), LIVE v2 acceptance, #143 twin refresh, 3 Magic Context proposals + 2-tier tool descriptions, memory/knowledge usage research, #134 B', FST logging live confirmation, #135 U4 leg. NEW: #145 (PENDING-CURATION). HELD: #120 U2.
