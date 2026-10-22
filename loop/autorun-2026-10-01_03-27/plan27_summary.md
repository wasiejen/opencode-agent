# plan27 summary — planner-27, ITER-27, 2026-10-22 (ses_e9b2e455dffe197VdpiExjySXS)

Idle-lane maintenance unit; nothing in flight at entry.

## What was verified
- MAIN gate baseline re-verified: `node .opencode/plugin/probes/handover_probe.mjs`
  → `PROBE handover: 360/360 PASS` at HEAD `1e9ce94` (391-line output).
- The plan24/25 `state_snapshot` LIVE acceptance is STILL PENDING, measured again
  from my own live digest: the 60-line hard cap applied, `## markers` came back
  unfiltered (`… (+126 more)`), and the `## todo` section printed priority.md lines
  — all signatures of the pre-`60a86cf` tool. The injected tool description also
  still reads "hard-capped at 60 lines". The committed tool (cap 150 + the 12
  doc-noise prefixes, smoke 23/23) is intact — verification rides the maintainer's
  NEXT host restart.
- Marker sweep (the ready-made command) → no new acted-on markers;
  `maintainer/inbox_planner/` empty (README only); `--wip` lines in
  `maintainer/ideas/ideas.md` and `maintainer/my_todos.md` left untouched.
- Maintainer live files untouched and unstaged: `opencode.jsonc` (the llama-swap-v2
  provider swap, dirty), `maintainer/SCRATCH_PAD.md` (the 2026-10-22_15-55 serving-shape
  note — backend domain, NAP-only).

## What was done
- TODO.md `## #146` FILED (OPEN): `agent/readme/repo_commands.md` lines 30-32 claim
  git-bash `$TMP` = the user Windows temp dir, but the live tool shell measures
  `TMP=/tmp TEMP=/tmp TMPDIR=[]` (in-session echo) and worker ses_e9b4198… measured
  `$OLDPWD` unset → `> "$OLDPWD/s.txt"` → `/s.txt` → Permission denied
  (agent_feedback 2026-10-22_20-08). Doc file is maintainer-owned (`repo_overview.md`
  lines 11-13) → flag + decision flag, recommendation: name the literal
  `C:/Users/Wasiejen/AppData/Local/Temp/opencode` path.
- NAP: planner-26 compressed into the archive, planner-27 archive line written,
  ENTRY POINT moved to planner-28 with the LIVE-acceptance evidence + the ITER-30
  maintenance trigger.
- loop_log START + DONE lines; counter.json (n=27) committed with the bookkeeping.

## Queue state (unchanged, OPEN/HELD)
auto_resume v2 port (order 2, HELD — v2-host loop mechanics), LIVE v2 acceptance
(backend slot), #143 twin refresh-vs-document, the 3 Magic Context proposals +
the 2-tier tool-description proposal, the memory/knowledge-usage research item,
#134 B' natural occurrence, FST logging live confirmation, #135 U4 `nudge=` leg,
NEW #146; HELD: #120 U2 (fork-side).

action: restart
