# v2 plugin harness + config discovery (Unit A, TODO #137, 2026-10-16)

Facts measured landing the v2 intercept-plugin port in the clone
`C:\Users\Wasiejen\Repos\opencode-agent-v2` (branch `migration`,
standalone opencode v2.0.29 in its root). Consult when building/typechecking
ANY plugin against the fork or the v2 line, or wiring v2 plugin config.

## Typechecking a file that imports `@opencode/plugin` against the fork
- **Do:** the harness tsconfig needs `typeRoots` pointing at the fork's
  installed `packages/plugin/node_modules/@types` + `types: ["node"]`,
  plus a `paths` mapping `@opencode/plugin` →
  `C:/Users/Wasiejen/Repos/opencode/packages/plugin/src/promise/index.ts`.
  The fork's checker is tsgo (`node_modules/.bin/tsgo.exe`); tsgo REJECTS
  the removed `baseUrl` option (TS5102) — use `paths`, not `baseUrl`.
  Committed reference harness: clone
  `C:\Users\Wasiejen\Repos\opencode-agent-v2\.opencode\plugins\tests\`.
- **Why (evidence):** Unit A (TODO #137) worker (ses_ebea1bd3c…, 2026-10-16_01-03).
  Without @types/node in the program, tsgo/tsc report TS7017 at fork
  `packages/ai/src/protocols/shared.ts:408` (`globalThis.process`) — a
  HARNESS artifact, not a fork bug; chasing it wastes a worker session.
- **Ref:** knowledge_inbox entry 2026-10-16_01-03 (cured here); clone
  `.opencode/plugins/tests/` (tsconfig + smokes committed on `migration`).
- **Keys:** @opencode/plugin, tsconfig, typeRoots, paths, tsgo, TS5102, TS7017, baseUrl.

## v2 `plugins` config entries pointing at a FILE are dropped
- **Do:** in v2, a `plugins` config entry whose path is a FILE is dropped
  with a warning ("configured plugin path must be directory" — fork
  `packages/core/src/config/plugin/source.ts:151-153`). v2 plugins load via
  DIRECTORY auto-discovery of `.opencode/plugins/`. A `./plugins/intercept_observer.ts`
  entry in a v2 `opencode.jsonc` is therefore DOCUMENTARY ONLY — decide per
  clone: keep the entry as documentation or drop it (the v2 clone's copy
  currently KEEPS it with a comment — maintainer question raised 2026-10-21, see
  todo_inbox 2026-10-16_01-02; the clone file is maintainer domain).
- **Why (evidence):** measured in Unit A (TODO #137): the v2 intercept plugin
  loaded in the clone via `.opencode/plugins/` auto-discovery while the
  file-path `plugins` entry logged the drop warning.
- **Ref:** worker ses_ebea1bd3c… 2026-10-16_01-02 (todo_inbox entry);
  clone `opencode.jsonc` comment (Unit A); fork source pointer above.
- **Keys:** v2, plugins config, must be directory, auto-discovery,
  opencode-agent-v2, intercept_observer.
