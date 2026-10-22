# Worker summary — TODO #142 + #138: the observation path channels are FIELD-SCOPED

Status: COMPLETE (both units landed; the FINAL commit carries this file + the TODO status lines).

## Task and scope

One spec, two findings, one unit pair. The observation log's path channels
(`out-of-sandbox` note, `path-anomaly`) classify ONLY the tool's PATH-FIELD
values, and the config allow-list roots join the note check.

- #142: no `out-of-sandbox` note for a path under a config-`allow`ed root.
- #138: NO path verdicts for path-looking substrings inside FREE TEXT
  (bash `command`, `write` `content`, `edit` `oldString`/`newString`).

Branch state: main repo `main`; clone `C:/Users/Wasiejen/Repos/opencode-agent-v2`
branch `migration`. The `plugin-unported-v1/**` frozen twins, `runRedirectCommand`,
the B' hard-stop site, the 8-field line shape, the caps and the channel grammars
stayed untouched.

## Files changed

MAIN (`main`, commit `dd5b63b`):
- `.opencode/plugin/intercept_observer_core.ts` — `observeArg` (was line 648)
  gains two trailing params: `extraRoots: string[] = []` and `pathScanText:
  string | null = null`; the (d) anomaly + (e) note calls scan `pt` (the
  override or the full string) and `observeSandbox(pt, workspaceRoot,
  extraRoots)`; pairs/dense/numword still scan the full arg string. Defaults
  keep every direct-call form byte-identical.
- `.opencode/plugin/intercept_observer.ts` — new helper `pathFieldText(toolName, args)`
  after `REDIRECT_PATH_FIELDS` (~line 592, joins the table's string field
  values with ONE space); `pathText` captured in the hook right after
  `argStr` (the ORIGINAL args, before any channel mutation); observation site
  (was 1433) passes `allowedRoots` + `pathText`; redirect recompute site
  (was 1474) now `observeSandbox(pathFieldText(tool, output?.args), dir ||
  null, allowedRoots)` (the EFFECTIVE field text); the R8 hook comment notes
  the field-scoped note.
- `.opencode/plugin/tests/intercept_observer.smoke.mjs` — pins (2b), (12n), (B3)
  repurposed; 3 new pins (16a/16b/16e).
- `.opencode/plugin/probes/handover_probe.mjs` — checks 303 + 304 note halves
  repinned; the S29 header comment corrected.

CLONE (`migration`, commit `622e75e`):
- `.opencode/plugins/intercept_observer.ts` — the same change (v2 `path` field
  names): helper after `REDIRECT_PATH_FIELDS` (~line 814), `pathText` capture
  after `argStr` (~line 912), observation build (~line 923) scans `pathText` +
  `allowedRoots`, recompute (~line 929) uses the EFFECTIVE field text.
- `.opencode/plugins/tests/smoke.mjs` — 3 new pins, inserted AFTER the whole-
  log invariants (like the v2unit1 pin, so the INV journal counts
  C8/C11/C15/C16 + C6/C7/C14 stay valid).

## Behavior changed

- A typed path under ANY allowed root (workspace + scratchpad + config allow
  + references) logs NO note; a genuinely un-allowed path STILL logs it.
- A bash `command`, a `content`, or an `oldString`/`newString` carrying path-
  looking text logs NO path verdict at all. Consequence: an UNmapped POSIX
  span inside a bash command no longer produces the fail-closed note (#102's
  substitution pass itself is unchanged, still one `kind=redirect` line).
- The initial scan uses the ORIGINAL typed fields (so the doubled-segment
  collapse pins still fire); the redirect recompute re-reads the EFFECTIVE
  fields (so a redirected target still suppresses the note).
- No gate-side behavior changed: the B' hard-stop, the permission gate, the
  8-field line shape and the caps are untouched.

## Measured verification (all runs after the edits)

Main repo:
- `node .opencode/plugin/tests/intercept_observer.smoke.mjs` → PASS,
  `INTERCEPT_OBSERVER_SMOKE: ALL PASS (96/96)` (baseline 93 → 3 new pins).
- `node .opencode/plugin/probes/handover_probe.mjs` → PASS,
  `PROBE handover: 360/360 PASS` (first run after the code change, before
  repinning 303/304: 358/360 — the two named bash checks failed, nothing else).
- `node .opencode/plugin/tests/block_transfer.smoke.mjs` → PASS, 150/150.
- `node .opencode/plugin/tests/block_transfer.sandbox.smoke.mjs` → PASS, 76/76.

Clone (`C:/Users/Wasiejen/Repos/opencode-agent-v2`):
- `node .opencode/plugins/tests/smoke.mjs` → PASS, `SMOKE: ALL PASS`, 52
  `PASS` lines (baseline 49 → 3 new pins).
- `node .opencode/plugin/probes/handover_probe.mjs` → PASS, `PROBE handover:
  360/360 PASS` (the probe's observer sections run on the frozen
  `plugin-unported-v1` pair — untouched, so the counts stayed 360).

Working tree: main `clean` except the untracked
`archive/sessions/compaction_dumps/*` (never committed by rule); clone `clean`.

## Commits

- main `dd5b63b` — "Scope observation path channels to typed path fields +
  allow-list roots (#142 #138)" (plugin + core + smoke + probe pins).
- clone `622e75e` — same subject, "(#142 #138 clone twin)".
- main FINAL (this file + the #142/#138 `LANDED` status lines) — hash reported
  after commit; NO hash inside the TODO status lines (planner stamps them).

## TODO / inbox

- `TODO.md` #138 → `LANDED 2026-10-22 (worker_FlashNext, ses_e9ee3329)` (line
  1243); #142 → same form (line 1276). No hashes in either line.
- `todo_inbox` (submitted): the spec's Pin audit missed three flipping
  surfaces — smoke `(B3)` (bash `command` long-doubled path pin), probe check
  303 and probe check 304 (the bash `/etc/` note halves). Repinning them was
  in scope; the audit line range list was incomplete.
- `agent_feedback` (submitted): `block_transfer WRITE` with TWO `regions` and
  ONE `text` writes the SAME text into EVERY region — the duplicated TODO.md
  pair it produced needed a repair script; the parameter contract should either
  reject N-regions-with-1-text or take per-region text.

## Deliberately omitted

- New probe pins for the #142/#138 forms: the spec puts the NEW pins in the
  two smoke files; the probe keeps 360 checks (repin only), so its section
  sums and annotations stay valid.
- #140's entry-level close ("the NOTE still fires on the allowed clone paths")
  — the planner owns that stamp.
- Any `.opencode/plugin-unported-v1/**` twin change (frozen v1 precedent, #140).

## Reusable lesson

`AGENTS.md` §Pin audit: a "expected UNCHANGED" list is only as good as its
line-range enumeration. Grep the pin files for the verdict tokens
(`grep -n "out-of-sandbox|path-anomaly" <smoke/probe>`) BEFORE editing the
channel, and treat the first gate run's FAIL list as the authoritative pin
inventory — it named exactly the three pins the audit missed.
