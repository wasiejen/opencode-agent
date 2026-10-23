OpenCode V2 is not a small feature release: it is a deliberate API/platform reset in three areas — **plugins**, **server/client API**, and **terminal-client configuration**. Most ordinary V1 configuration, agents, commands, skills, and MCP definitions still work, but **your V1 plugin code will not load at all** and must be ported. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

## Why V2 changed

The core reasoning is that V1 grew organically: hooks were string-keyed, plugin state was often ad hoc, the context mixed client access and extension points, and configuration had many singular/deprecated aliases (`plugin`, `agent`, `command`, `provider`, `mode`, etc.). V2 makes ownership explicit: each subsystem owns its hooks/transforms, plugins get stable identities and storage, and configuration becomes more declarative and consistent. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

## Plugin changes: the big one

This is almost certainly why your recent plugins are failing. V1’s default export was an async function returning `{ hooks, tools, ... }`; V2 requires a default-exported definition with a stable `id` plus `setup(ctx)` (or `effect()` for Effect-based plugins). [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)

```ts
// V1
export const MyPlugin: Plugin = async ({ client, project }) => {
  return {
    "tool.execute.before": async (input, output) => { /* ... */ },
  }
}

// V2
import { Plugin } from "@opencode/plugin"

export default Plugin.define({
  id: "my-plugin",
  async setup(ctx) {
    await ctx.tool.hook("execute.before", (event) => {
      // event is mutable
    })
  },
})
```

The package also changes from `@opencode-ai/plugin` to `@opencode/plugin`. [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)

### Conceptual shift

| V1 model | V2 model | Reason |
|---|---|---|
| Plugin function returns a bundle of hooks/tools | `Plugin.define({ id, setup(ctx) })` registers capabilities | Gives plugins identity, lifecycle, diagnostics, and scoped storage.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| `client` property for all server calls | Domain APIs: `ctx.session`, `ctx.tool`, `ctx.provider`, `ctx.permission`, etc. | Avoids one oversized client surface; operations belong to their subsystem.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| String hook keys, e.g. `"tool.execute.before"` | Hooks registered on the owning domain, e.g. `ctx.tool.hook(...)` | Makes extension points discoverable and scoped.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| Separate `input` and `output` callback args | One mutable `event` object | Simplifies mutation and makes the data flow clearer.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| Returned `dispose()` hook | Cleanup function returned from `setup()` | Aligns resource cleanup with plugin lifecycle.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| `event` hook | `ctx.event.subscribe()` | Uses the public event stream explicitly.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| Custom `tool()` map | `ctx.tool.transform()` editor; JSON Schema input; structured output | Standardizes tool definitions and makes transforms replayable.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |
| Bun `$` helper | Bring/use your own process API | Removes a runtime-specific dependency from the plugin contract.  [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1) |

### Hook mapping cheat sheet

Common V1 hooks map roughly like this, though they are **not exact renames** in every case: [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)

- `event` → `ctx.event.subscribe()`
- `dispose` → cleanup function returned by `setup()`
- `tool.execute.before/after` → `ctx.tool.hook("execute.before" | "execute.after", ...)`
- `shell.env` → `ctx.shell.hook("create.before", ...)`
- `permission.ask` → `ctx.permission.hook("evaluate", ...)`
- `chat.message` → `ctx.session.hook("prompt", ...)`
- `chat.params` → `ctx.session.hook("context", ...)`
- `chat.headers` → `ctx.session.hook("model.request", ...)` or `"http.request"`
- `experimental.chat.system.transform` / `messages.transform` → `ctx.session.hook("context", ...)`
- `experimental.session.compacting` → `ctx.session.hook("compaction", ...)`
- `config` / `provider` / `tool.definition` → domain-specific `transform()` calls

Some experimental V1 hooks have **no direct V2 equivalent**, including `experimental.compaction.autocontinue`, `experimental.provider.small_model`, and `experimental.text.complete`; those need redesign around the new session/provider/model/event APIs. [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)

## Configuration changes

Your existing `opencode.json(c)` and many `.opencode/` files should continue to work: V2 normalizes supported V1 fields in memory rather than forcing an immediate rewrite. Agents, commands, skills, permissions, MCP servers, and other file-based definitions are intended to remain compatible outside the three explicit breaking areas. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

The main optional renames are consistency-driven: plural nouns and clearer ownership. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

| Area | V1 | V2 |
|---|---|---|
| Plugins | `plugin` | `plugins`; local package/options tuple becomes `{ package, options }`  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| Agents | `agent`, `mode` | `agents`; `prompt` → `system`, `disable` → `disabled`, variant appended as `model#variant`  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| Commands | `command`, `subtask` | `commands`, `subagent`  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| Permissions | Per-tool permission map plus `tools` | One ordered `permissions` array; `bash` → `shell`, `task` → `subagent`, `write`/`patch` → `edit`  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| Providers | `provider`, `npm`, `api`, `options` | `providers`, `package` (`aisdk:` prefix for AI SDK packages), `settings.baseURL`, split `settings`/`headers`/`body`  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| MCP | `mcp.<name>` | `mcp.servers.<name>`; `enabled` → `disabled`; separate catalog/execution timeouts  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| Skills | Separate `paths` and `urls` | One ordered `skills` array  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |
| Compaction | `preserve_recent_tokens`, `reserved` | `keep.tokens`, `buffer`  [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/) |

The reason is not merely cosmetic: the ordered permission array makes precedence and exceptions explicit, provider settings are separated by role, and MCP timeouts distinguish catalog discovery from tool execution. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

## Server and client API

If your plugins or external tools call OpenCode’s HTTP/server API, those integrations must move to the V2 contracts and `@opencode/client`. This is a separate intentional break from the plugin API; V2 redesigned the API for a more consistent, ergonomic client surface. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

For example, one reported V1-to-V2 migration changed `ctx.client.session.messages()` to `ctx.session.context()`, illustrating the broader move from generic client access toward domain-owned operations. [github](https://github.com/code-yeongyu/oh-my-openagent/issues/8485)

## Terminal configuration

V2 replaces layered `tui.json(c)` files with one global `~/.config/opencode/cli.json`. The first V2 terminal startup migrates supported global TUI settings automatically; project-local client configuration is not migrated because V2 treats terminal-client configuration as global. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

## Practical migration strategy

1. **Do not rewrite all config first.** Start V2 with your current supported V1 configuration and verify models, credentials, agents, permissions, and MCP servers. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)
2. **Port plugins one at a time.** Rename `plugin` → `plugins` in config, but understand that this alone does nothing for the implementation: V1 plugin code does not execute in V2. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)
3. **Convert each plugin to `Plugin.define`.** Add an `id`, move initialization into `setup(ctx)`, replace `client.*` calls with domain APIs, and register hooks/transforms in `setup`. [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)
4. **Handle tools and events explicitly.** Convert custom tools to `ctx.tool.transform()` with JSON Schema, and convert event handling to `ctx.event.subscribe()` plus a cleanup function where needed. [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)
5. **Keep dual support temporarily if useful.** A package can export both a V2 `setup()` and a V1 `server()` implementation; V2 calls `setup()`, while newer V1 releases can call `server()`. The two implementations remain separate—there is no automatic hook translation. [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)
6. **Only then optionally modernize config.** V2 can normalize legacy fields, so converting to native V2 syntax is not required to run; it is mainly for clarity and future-proofing. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)

## Bottom line for your plugins

If your plugins mostly hook into tools, shell execution, permissions, prompts, model requests, providers, or events, expect a real but mechanical port: new import, `Plugin.define`, `id`, `setup(ctx)`, domain-scoped hook registration, mutable event objects, and possibly replacement of Bun `$` with your own process handling. If they rely on experimental V1 lifecycle hooks or call the V1 server API directly, expect more redesign than renaming. [dev.opencode](https://dev.opencode.ai/v2/docs/migrate-v1/)


especially useful is i think:
5. **Keep dual support temporarily if useful.** A package can export both a V2 `setup()` and a V1 `server()` implementation; V2 calls `setup()`, while newer V1 releases can call `server()`. The two implementations remain separate—there is no automatic hook translation. [dev.opencode](https://dev.opencode.ai/v2/docs/build/plugins/migrate-v1)
