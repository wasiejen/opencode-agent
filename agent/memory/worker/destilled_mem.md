# Worker memory — destill

Condensed worker memory, one line per entry; full entries (evidence +
review conditions) in `memory.md` (same folder). Policy: `agent/memory/README.md`.

- WM-01: in a buffered CLI write transaction, explicitly APPEND COMMIT to the flushed script — an uncommitted batch rolls back silently at process exit while the tool prints success.
- WM-02: `fs.rmSync` on a directory needs `{recursive:true, force:true}` — `{force:true}` alone throws ERR_FS_EISDIR on this Windows/node host.
- WM-03: pytest print→logger work: capsys is BLIND to records — use caplog.at_level pins; BEFORE the build grep for caplog pins naming old loggers + capsys assertions on the converted texts.
- WM-04: tool-call JSON payloads FAIL over ~2 KB on this host (Unterminated string) — batch content payloads ≤ ~1.8 KB; re-pin byte-exact pins with one fail-loud script (MISS(n) per pair), not sequential edits.
- WM-05: cross-tree diffs — other repos need `git -C <other> show <ref>:path` into a temp file; `cmp` with process substitution FALSE-DIFFERS in git-bash Windows (write real files first).
- WM-06: file tools resolve RELATIVE paths against the SESSION CWD (main repo), never the target clone — ALWAYS absolute paths for clone files; catch wrong-path writes via `git status` before the checkpoint.
- WM-07: before a tool-contract change grep the byte-exact message strings across EVERY pin file (smokes AND probe) in BOTH trees — one named repin per tree; keep reject messages byte-identical, copy each tree's OWN feedback format.
- WM-08: derive pinned smoke numbers from the DOCUMENTED rule (enumerate fixtures, apply the rule) — NEVER copy measured output: a copied measurement pins a bug in place (the 4b `verdict=ok`).
- WM-09: v2 plugin smokes — per fixture workspace call `plugin.setup(makeCtx(dir))` (the repo-root carrier is setup-captured, the module SHARED); count dispose deltas per teardown call, not globally.
- WM-10: a v2 smoke steers the shared gauge core via setDbPath/setBudgetFileForTest — the plugin's relative `.mjs` import and the smoke's `pathToFileURL` import resolve to ONE module instance; no plugin-side test hook needed.
