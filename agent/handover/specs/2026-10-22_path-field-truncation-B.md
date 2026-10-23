# Spec B — the intercept log NEVER cuts inside a single-path FIELD (TODO #140, unit B)

Assigned worker: `worker_FlashNext`. Run AFTER spec A
(`agent/handover/specs/2026-10-22_cross-repo-roots-A.md`) lands.

## Goal
An intercept line's path evidence/context field prints a PATH in FULL (or not
at all), so an agent reading its own line never sees a truncated path like
`C:\Users\Wasi` and concludes it mistyped. Today the generic cap cuts
mid-path.

## Definition of done (measured)
1. A field whose value IS one path (no surrounding prose) longer than the
   generic cap prints UNTRUNCATED (a new cap for path-shaped single values),
   pinned by a smoke fixture with a >-cap-long path.
2. Mixed prose+path fields keep the existing generic cap + the ASCII `...`
   marker, but the cut NEVER lands INSIDE a path-looking substring: the
   truncation point moves BACK to the last position that keeps every path
   substring whole, and if no such position keeps the head of the field
   intact, the field falls back to the existing short summary form. Pinned
   per form (leading path + prose, prose + trailing path, two paths).
3. Line-shape byte-compat holds: 8 pipe-separated fields, order unchanged,
   `...` marker unchanged; only the field CONTENT rule changes. Pin: a
   byte-shape check on a long-path line.
4. Pin audit: `.opencode/plugin/tests/intercept_observer.smoke.mjs` (today
   85 pins) — new pins per items 1-3 + re-pin any existing pin that asserts a
   truncated path field; the probe's intercept sections audited IN PLACE
   (name the changed check ids in the handover).
5. Clone twin: the same change in
   `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\intercept_observer.ts`
   + its `.opencode/plugins/tests/smoke.mjs` (today 46 pins); the clone gate
   recorded as a DIFF against the red baseline (TODO #141) — never "green".
6. Main-repo gate green: probe `PROBE handover: <t>/<t> PASS` (annotation ==
   section sums), all smokes green; working tree clean.

## Verified facts (planner, 2026-10-22)
- `.opencode/plugin/intercept_observer_core.ts:137` — `export const MAX_FIELD_CHARS = 160`;
  the truncate helper at `:660`: `if (s.length > cap) s = s.slice(0, cap - 3) + "..."`.
- Measured artifact: `.opencode/temp/intercept.log` holds 4 truncated path echoes
  (`C:/Users/Wasi`, `C:/Users/Wasie` x2, `C:/Users/Wasiej`) vs 2099 complete
  — all in the log's own `path=`/`root=` fields, none in an actual CALL.
- The 8-field line shape + the surviving verdict list are documented in the
  clone's plugin header (`.opencode/plugins/intercept_observer.ts` lines ~55-70)
  and in `agent/knowledge/knowledge_plugins.md` — do NOT alter the shape.
- Unit A touched `block_transfer` only; B is independent — land it as its own
  unit.

## DO NOT touch
- the verdict semantics (note-only vs hard-stop), the B' hook, the fuzzy resolvers,
  the R8 root list, `opencode.jsonc`, `maintainer/**`, the live DB, the backend
  server, `TODO.md` except a status line in the FINAL commit.

## Commit shape
- One green checkpoint per verified unit step; main repo and clone in SEPARATE
  commits, named paths only. `LANDED` wording; the hash rides the planner's
  follow-up bookkeeping commit. TODO + handover in the FINAL commit.

## Live acceptance (pending)
- The maintainer's next session's own tool calls (or the 3-tool probe from
  spec A) show full paths in the intercept lines; grep the log for any path
  echo ending mid-word.
