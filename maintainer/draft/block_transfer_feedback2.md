--comment: model Next-coder

## Overall verdict

The concept is strong and, for an agentic editing tool, unusually good:

- It avoids the fragile exact-`oldString` / dense-number matching failure modes.
- Anchors are deterministic and teachable.
- Errors explain what went wrong and often give candidates.
- PEEK/MAP are smart for a persistent clipboard.
- “No partial writes on rejection” is the right instinct.

However, I would not call it production-hardened yet. There are a few real correctness/safety bugs, especially around destructive `MOVE` behavior, EOF insertion/newlines, sandbox path resolution, and mode validation. For agentic use, those should be fixed before polishing the description or adding features.

A good priority order would be:

1. **Fix destructive operation atomicity and same-file `MOVE`.**
2. **Fix EOF insertion / trailing-newline behavior.**
3. **Harden sandbox resolution: symlinks, OS case handling, temp root.**
4. **Make mode strict: no default `MOVE`, reject unknown modes, reject empty `regions`.**
5. **Simplify the reference schema for better model behavior.**
6. **Preserve line endings and use atomic writes.**
7. **Improve agentic feedback and buffer state discoverability.**

---

# 1. High-priority correctness and safety issues

## 1.1 `MOVE` is not atomic when destination/target resolution fails

Your comments say:

> no partial writes on rejection

But for `MOVE`, the current flow is:

1. Read `srcFile`.
2. Resolve `startMarker`/`endMarker`.
3. Cut and **write `srcFile`**.
4. Then validate/write `dstFile`.
5. Only then resolve `targetMarker` in `dstFile`.

That means if `targetMarker` is invalid, ambiguous, not found, out of range, or if the destination write fails, the source file has already been cut.

This is the most serious issue. The agent can lose the block.

Example failure path:

```json
{
  "mode": "MOVE",
  "srcFile": "a.md",
  "dstFile": "b.md",
  "startMarker": "## Section",
  "endMarker": "## Next",
  "targetMarker": "# BadAnchor"
}
```

If `targetMarker` fails, `a.md` is already modified.

### Recommended fix

Make `MOVE` a two-phase transaction.

#### For different files

Pseudo-code:

```ts
// 1. Read both files BEFORE writing anything
const srcText = read(srcPath);
const dstText = exists(dstPath) ? read(dstPath) : null;

// 2. Resolve all source spans and destination target BEFORE writing
const start = resolveRef(args.srcFile, "Start marker", srcText, args.startMarker);
const end = resolveRef(args.srcFile, "End marker", srcText, args.endMarker);

const target = refPresent(args.targetMarker)
  ? resolveRef(args.dstFile, "Target marker", dstText ?? "", args.targetMarker)
  : null;

// 3. If any error, return now. No file has been written.
if (error) return error;

// 4. Compute new source and destination in memory
const newSrc = removeSpan(srcText, start.line, end.line);
const newDst = insertBlock(dstText, extracted, target);

// 5. Write both files, preferably atomically
writeAll({
  [srcPath]: newSrc,
  [dstPath]: newDst
});
```

For the best behavior, wrap the final writes in a helper that can roll back:

```ts
async function writeAll(changes: Record<string, string>) {
  const backups: Record<string, string | null> = {};

  // Read backups first
  for (const file of Object.keys(changes)) {
    backups[file] = fs.existsSync(file) ? fs.readFileSync(file, "utf-8") : null;
  }

  try {
    for (const [file, content] of Object.entries(changes)) {
      writeFileAtomic(file, content);
    }
  } catch (e) {
    for (const [file, content] of Object.entries(backups)) {
      if (content === null) {
        // If the backup was “nonexistent”, delete the newly created file if safe.
        if (fs.existsSync(file)) fs.unlinkSync(file);
      } else {
        writeFileAtomic(file, content);
      }
    }
    throw e;
  }
}
```

This is not crash-proof across process death, but it greatly reduces the “wrong anchor leaves source cut” problem.

For a coding agent, this is much more important than perfect atomicity.

---

## 1.2 `MOVE` between the same file is likely broken

If `srcFile` and `dstFile` resolve to the same file, the current code does this:

1. Read the original file.
2. Cut the block and write the file.
3. Read the already-cut file.
4. Insert the block into that already-cut file.

That can produce surprising results, especially if:

- `targetMarker` refers to a line inside the moved block.
- `targetMarker` is a line number from the pre-call file state.
- the destination EOF differs from the source EOF.
- the moved block and target marker overlap.

For a normal file tool, same-file move is a common agent request:

```text
Move this function after this other function.
```

You should either:

### Option A: implement same-file move properly

This is possible and worthwhile.

Use pre-call line numbers for source span and target line, then adjust for removal.

Pseudo-code:

```ts
function moveWithinSameFile(
  originalLines: string[],
  startLine: number,
  endLine: number,
  targetLine: number | null // null means append at EOF
) {
  const block = originalLines.slice(startLine - 1, endLine);
  const blockLen = block.length;

  if (targetLine !== null && targetLine >= startLine && targetLine <= endLine) {
    return {
      error: `Error: target marker is inside the source block in ${fileRef}.`
    };
  }

  const withoutBlock = [
    ...originalLines.slice(0, startLine - 1),
    ...originalLines.slice(endLine)
  ];

  let insertAt: number;

  if (targetLine === null) {
    insertAt = withoutBlock.length;
  } else if (targetLine < startLine) {
    // Target line stays before the removed block.
    insertAt = targetLine;
  } else {
    // targetLine > endLine
    insertAt = targetLine - blockLen;
  }

  withoutBlock.splice(insertAt, 0, ...block);

  return { lines: withoutBlock };
}
```

The return line could then include the destination insertion point:

```text
Moved 5 lines from 'notes.md' to 'notes.md' after line 12 (source lines 3..7, first: '## TODO')
```

or better:

```text
Moved 5 lines in 'notes.md' (source lines 3..7 -> inserted at lines 12..16, first: '## TODO')
```

### Option B: reject same-file MOVE explicitly

If you do not want to implement it yet:

```ts
if (realPath(srcPath) === realPath(dstPath)) {
  return `Error: MOVE to the same file is not supported. Use COPY + REPLACE/WRITE, or use line-number based move-within-file if implemented.`;
}
```

But for agentic use, I would prefer implementing it.

---

## 1.3 EOF insertion can add a spurious blank line

This is subtle but important.

Consider an existing destination file:

```text
a
```

with a trailing newline, i.e. actual file contents:

```text
a\n
```

Current code does:

```ts
dstText = fs.readFileSync(...);
dstLines = dstText.split(/\r?\n/);
```

For `"a\n"`, that gives:

```ts
["a", ""]
```

Then:

```ts
insertIdx = dstLines.length; // 2
dstLines.splice(insertIdx, 0, ...buffer);
```

Resulting array:

```ts
["a", "", "b"]
```

Written with `\n`:

```text
a

b
```

The intended EOF append was probably:

```text
a
b
```

Same issue for an existing empty file `""`:

```ts
dstLines = [""],
insertIdx = 1
```

can create a leading blank line.

### Recommended fix

Use normalized line arrays and line counts, not raw `split(/\r?\n/).length`, for EOF insertion.

Example:

```ts
function textToLines(text: string): string[] {
  if (text === "") return [];
  const lines = text.split(/\r?\n/);
  return text.endsWith("\n") ? lines.slice(0, -1) : lines;
}

function countTextLines(text: string): number {
  return textToLines(text).length;
}
```

For `PASTE`:

```ts
const dstText = fs.existsSync(dstPath) ? fs.readFileSync(dstPath, "utf-8") : "";
const dstLines = textToLines(dstText);

let insertIdx = dstLines.length; // not dstText.split().length
if (refPresent(args.targetMarker)) {
  const target = resolveRef(args.dstFile, "Target marker", dstText, args.targetMarker);
  if ("error" in target) return target.error;
  insertIdx = target.line;
}

dstLines.splice(insertIdx, 0, ...buffer);
```

Then write with preserved trailing-newline policy if desired:

```ts
const endsWithNewline = dstText === "" ? false : dstText.endsWith("\n") || dstText.endsWith("\r\n");
const output = dstLines.join("\n") + (endsWithNewline ? "\n" : "");
```

If you also preserve CRLF, see section 5.

---

## 1.4 Unknown or missing mode can behave surprisingly

Current code:

```ts
const mode = (args.mode || "MOVE").toUpperCase();
```

Issues:

- If `mode` is missing, it silently becomes `MOVE`, which is destructive.
- If `mode` is unknown, it may fall through to generic extraction validation and eventually return `"Operation completed."`.

That is bad for agentic use. A typo should not produce a successful-looking result.

### Recommended fix

Remove the default. Make mode required and validate explicitly.

```ts
const allowedModes = [
  "MOVE", "COPY", "APPEND", "CUT", "PASTE",
  "REPLACE", "WRITE", "PEEK", "MAP", "DELETE", "CLEAR"
];

const mode = typeof args.mode === "string"
  ? args.mode.toUpperCase()
  : "";

if (!allowedModes.includes(mode)) {
  return `Error: 'mode' is required and must be one of: ${allowedModes.join(", ")}.`;
}
```

Do not default `MOVE`.

If you want a safe default, consider:

```ts
const mode = args.mode || "PEEK";
```

But for a tool with destructive modes, I prefer no default.

---

## 1.5 Empty `regions` and possibly empty `refs` lists should be rejected

In `WRITE`, if:

```json
{
  "mode": "WRITE",
  "dstFile": "file.md",
  "text": "x",
  "regions": []
}
```

`hasRegions` is true, `spans` is empty, and the function eventually returns:

```ts
return feedback.join("\n");
```

which is an empty string.

That is confusing:

- No feedback.
- The file may be rewritten with normalized line endings.
- `last_write` may be updated.

### Recommended fix

```ts
if (hasRegions) {
  if (!Array.isArray(args.regions) || args.regions.length === 0) {
    return "Error: 'regions' must be a non-empty array.";
  }
}
```

For `COPY`/`APPEND`, you may want to reject empty `refs` too:

```ts
if (Array.isArray(args.refs) && args.refs.length === 0) {
  return "Error: 'refs' must contain at least one ref. Use CLEAR to empty a buffer.";
}
```

If you intentionally allow `refs: []` as a way to clear a buffer, document it. But I would make `CLEAR` the only way to clear a buffer.

---

# 2. Sandbox and path safety issues

## 2.1 Symlinks can defeat the sandbox

Current `sandboxCheck` uses:

```ts
path.resolve(cwd, givenPath)
```

but it does not use `fs.realpathSync`.

That means a symlink inside the working directory can point outside the sandbox.

Example:

```text
cwd/allowed-link -> /etc
```

Then:

```ts
cwd/allowed-link/passwd
```

may pass `startsWith(cwd)`.

For an agentic coding tool, this matters.

### Recommended fix

Use a safe realpath helper that resolves existing ancestors and appends missing path components.

```ts
function safeRealpath(p: string): string {
  const abs = path.resolve(p);
  const missing: string[] = [];
  let cur = abs;

  while (true) {
    try {
      const real = fs.realpathSync(cur);
      return path.join(real, ...missing);
    } catch {
      const parent = path.dirname(cur);
      if (parent === cur) return abs;
      missing.unshift(path.basename(cur));
      cur = parent;
    }
  }
}
```

Then check:

```ts
const cwdReal = fs.realpathSync(cwd);
const real = safeRealpath(abs);
```

Use real paths for containment checks.

Also validate `dstFile` before creating directories, because directory creation can follow symlinks.

---

## 2.2 Case folding is unsafe on non-Windows systems

Current code:

```ts
const resolved = path.resolve(cwd, givenPath).toLowerCase();
```

On Windows, case-insensitivity is often correct.

On Linux/macOS, filesystem paths are usually case-sensitive. Lowercasing both roots and paths can accidentally allow a path that is not actually inside the root.

Example on Linux:

```text
root: /a
given: /A
```

Lowercased:

```text
/a and /a
```

but they are different paths.

### Recommended fix

Only lowercase on Windows:

```ts
function normalizePath(p: string): string {
  if (process.platform === "win32") return p.toLowerCase();
  return p;
}
```

And use a path containment helper:

```ts
function isInside(root: string, target: string): boolean {
  const r = process.platform === "win32" ? root.toLowerCase() : root;
  const t = process.platform === "win32" ? target.toLowerCase() : target;

  if (r === t) return true;

  const rel = path.relative(r, t);
  return rel !== "" &&
    !rel.startsWith("..") &&
    !path.isAbsolute(rel);
}
```

Then:

```ts
const realCwd = fs.realpathSync(cwd);
const tempRoot = process.env.TEMP ?? process.env.TMP ?? require("os").tmpdir();
const rootsReal = [realCwd, tempRoot].filter(Boolean).map((r) => fs.realpathSync(r));

const real = safeRealpath(path.resolve(cwd, givenPath));

const allowed = rootsReal.some((root) => isInside(root, real));
```

Also allow configurable roots if you use this tool beyond one project:

```ts
const extraAllowed = context.allowedBlockTransferRoots ?? [];
```

For cross-platform use, using `os.tmpdir()` is usually better than `process.env.TEMP` only.

---

# 3. Reference schema and model usability feedback

## 3.1 String/number unions are risky for LLM tool schemas

Current:

```ts
tool.schema.union([tool.schema.string(), tool.schema.number().int()])
```

You are already dealing with a practical issue: the host channel may stringify integers. That means the type branch is fragile.

For agentic use, I would strongly consider using **string-only refs**.

Example:

```ts
const ref = tool.schema.string().optional().describe(
  "Either a 1-based line number as a digit string, e.g. '42' or 'line:42', or a marker string."
);
```

Then parse:

```ts
function parseRef(ref: string): RefResolution {
  if (typeof ref !== "string") return { error: `Error: ref must be a string.` };

  if (ref === "") return { error: `Error: ref is empty.` };

  const lineMatch = /^(?:line:)?(\d+)$/.exec(ref);
  if (lineMatch) {
    return { line: Number(lineMatch[1]) };
  }

  return { marker: ref };
}
```

This avoids:

- Provider schema weirdness.
- Model accidentally sending numbers as JSON numbers.
- Complex union descriptions.
- Type-branch dead code.

If you keep string-only digit parsing, you must clearly document that a marker cannot be all digits unless escaped or extended.

You can support an escape form:

```text
marker:42
```

or:

```text
line:42
```

or:

```text
@42
```

A clean grammar might be:

```text
42             -> line 42
line:42        -> line 42
marker:42      -> literal marker "42"
TODO           -> marker "TODO"
```

This gives agents an unambiguous way to express numeric-looking markers.

Current behavior cannot use `"42"` as a marker. That is usually okay, but you should mention it or support an escape.

---

## 3.2 Line-number refs need explicit positive bounds

The schema says:

```ts
tool.schema.number().int()
```

but negative numbers are still accepted by the schema. Your runtime rejects them, which is good.

If your tool schema supports it, add:

```ts
tool.schema.number().int().min(1)
```

For `PEEK.count`, if you cap at 25 silently, either:

- enforce schema `max: 25`, or
- return an explicit capped message:

```text
Peeked buffer 'default': lines 10..34 of 100 — ... (capped at 25 lines)
```

For agentic use, silent caps can confuse. Prefer explicit feedback.

---

## 3.3 Consider separate fields for line numbers and markers

If you want maximum clarity, split the ref types:

```ts
startLine: number | optional
startMarker: string | optional
endLine: number | optional
endMarker: string | optional
```

But that increases schema complexity. The unified marker-or-number ref is elegant.

My recommendation: keep the unified concept, but make it string-only and grammar-based:

```text
42
line:42
#Heading
TODO
```

---

# 4. Line endings, encoding, and atomic writes

## 4.1 You are normalizing CRLF to LF

Current writes use:

```ts
fs.writeFileSync(dstPath, dstLines.join("\n"), "utf-8");
```

If the source file uses CRLF:

```text
line1\r\nline2\r\n
```

then after `PASTE`, `REPLACE`, `WRITE`, or `MOVE`, it may become LF:

```text
line1\nline2\n
```

For coding agents, this is a common source of noisy diffs.

### Recommended fix

Detect destination line ending and preserve it.

```ts
function detectEol(text: string): "\n" | "\r\n" {
  if (text.includes("\r\n")) return "\r\n";
  if (text.includes("\n")) return "\n";
  return "\n";
}
```

Normalize internal buffers to LF, then serialize using the file’s detected EOL.

```ts
function writePreservingEol(file: string, text: string, normalizedLines: string[]) {
  const eol = detectEol(text);
  const endsWithNewline = text === "" ? false : text.endsWith("\n") || text.endsWith("\r\n");
  const out = normalizedLines.join(eol) + (endsWithNewline ? eol : "");
  writeFileAtomic(file, out, "utf-8");
}
```

For new files, choose LF unless you have a project convention. You could inspect a repo `.gitattributes` later, but for this tool, simple LF creation is okay.

---

## 4.2 Use atomic writes

Replace direct writes with a helper:

```ts
function writeFileAtomic(file: string, content: string, encoding: "utf-8") {
  fs.mkdirSync(path.dirname(file), { recursive: true });

  const tmp = path.join(
    path.dirname(file),
    `${path.basename(file)}.block_transfer-${process.pid}-${Date.now()}.tmp`
  );

  fs.writeFileSync(tmp, content, encoding);
  fs.renameSync(tmp, file);
}
```

Caveats:

- Rename is best across the same filesystem.
- On Windows, cross-device rename behavior can differ.
- For maximum robustness, handle `EXDEV` and other errors.

But even a simple temp+rename is much better than direct overwrite.

---

## 4.3 BOM handling

If a file starts with a UTF-8 BOM:

```text
﻿## Heading
```

your anchor matching may not work because the first line includes BOM.

You can strip BOM for matching but preserve it on write:

```ts
function stripBom(text: string): string {
  return text.charCodeAt(0) === 0xfeff ? text.slice(1) : text;
}
```

On write, if the original file had BOM, re-add it.

This is uncommon in agent environments, but Windows files sometimes have BOM.

---

## 4.4 Permissions and metadata

`writeFileSync` can lose file permissions or executable bits.

If this tool ever touches scripts or executable files, consider preserving mode:

```ts
const st = fs.statSync(file);
fs.writeFileSync(...);
fs.chmodSync(file, st.mode);
```

For ordinary source files this is usually fine, but for coding agents, preserving permissions is a nice robustness improvement.

---

# 5. Agentic UX and feedback improvements

Your feedback lines are already excellent. I would improve them further.

## 5.1 Include insertion point for `MOVE` and `PASTE`

Current:

```text
Pasted N lines from buffer 'x' into 'dst'
```

The agent may not know where the block landed.

Better:

```text
Pasted 5 lines from buffer 'x' into 'dst' after line 8 (lines 1..5, first: '## TODO') - buffer: 5 lines.
```

or:

```text
Pasted 5 lines from buffer 'x' into 'dst' at lines 9..13 (first: '## TODO') - buffer: 5 lines.
```

Same for `MOVE`:

```text
Moved 5 lines from 'src' to 'dst' after line 12 (source lines 3..7, first: '## TODO')
```

or:

```text
Moved 5 lines from 'src' to 'dst' inserted at lines 13..17 (source lines 3..7, first: '## TODO')
```

This is extremely helpful because line numbers are otherwise stale.

For `MOVE` within the same file:

```text
Moved 5 lines in 'src' from lines 3..7 to lines 13..17 (first: '## TODO')
```

That is much better than saying only the source span.

---

## 5.2 Add a `LIST` mode for buffers

Buffers are persistent. Agents can forget names.

Add:

```ts
LIST
```

Example output:

```text
Buffers: default: 12 lines, last_write: 5 lines
```

This helps agents avoid calling `PEEK` on wrong buffer names repeatedly.

Useful output shape:

```text
Buffer list: default: 14 lines; last_write: 5 lines; sectionA: 3 lines
```

---

## 5.3 Make non-unique anchor errors include snippets

Current non-unique error:

```text
Error: Start marker 'TODO' is not unique in 'file.md' (12 matches: lines 3, 10, 22 …).
```

This is good, but agents often widen anchors more effectively if they see the lines:

```text
Error: Start marker 'TODO' is not unique in 'file.md' (12 matches:
  3: '## TODO: fix login'
  10: '## TODO: fix logout'
  22: '## TODO: remove old code'
 …)
```

The anchor side already has a “closest lines” hint for not-found. For non-unique, include at least 3 candidate line snippets.

This does not violate the unique-anchor rule; it just helps the agent choose a unique longer prefix.

---

## 5.4 Include candidate hints for case-insensitive mistakes too

Your matching is case-sensitive. That is fine and often correct for code.

But headings and prose are frequently case-insensitive in human terms.

Example:

```text
Anchor: ## todo
Line:   ## TODO
```

Your not-found closest-prefix hint may be weak because common prefix is short.

You can leave matching case-sensitive, but improve the hint:

```text
Closest lines: 4: '## TODO' (case-sensitive prefix does not match; consider using the exact case)
```

or just include case-insensitive candidate lines in the hint.

---

## 5.5 Make `PEEK` cap behavior visible

Current behavior:

```ts
const window = Math.min(count, 25);
```

If the agent asks for `count: 100`, it silently receives 25 lines.

Better:

```text
Peeked buffer 'x': lines 10..34 of 100 — ... (requested 100, returned 25 capped)
```

or enforce schema `max: 25` and return an error:

```text
Error: 'count' must be between 1 and 25 for PEEK.
```

For agentic use, explicit is better.

---

## 5.6 Consider auto-saving destructive operations for recovery

You already auto-save `WRITE` text to `last_write`.

For destructive ops, consider similar:

- `DELETE` → store deleted block in `last_delete`
- `CUT` → already stores into named buffer
- `MOVE` source block → if destination fails, you can pre-store the block in a recovery buffer

Possible recovery buffers:

```text
last_write
last_delete
last_move_source
```

Tradeoff: if files contain sensitive data, buffers can persist in memory. For local agent use this is probably acceptable, but it should be documented.

---

# 6. Per-mode argument validation

Currently, many irrelevant args are silently ignored.

Example:

```json
{
  "mode": "PASTE",
  "dstFile": "out.md",
  "srcFile": "ignored.md",
  "startMarker": "also ignored"
}
```

This may hide model mistakes.

You do not need to be perfectly strict, but it would help to validate mode-specific forbidden args.

Example:

```ts
const forbidden: Record<string, string[]> = {
  PASTE: ["srcFile", "startMarker", "endMarker", "refs", "text"],
  COPY: ["dstFile", "targetMarker"],
  MOVE: ["refs", "text", "regions", "from", "count"],
  REPLACE: ["targetMarker", "text", "regions", "refs"],
  WRITE: ["bufferName", "targetMarker", "srcFile"],
  CLEAR: ["srcFile", "dstFile", "startMarker", "endMarker", "refs", "text"]
};
```

At minimum, return errors for obviously contradictory inputs:

```text
Error: 'targetMarker' is not valid for REPLACE mode. REPLACE uses startMarker/endMarker in dstFile.
```

This helps agents recover.

---

# 7. Session state and concurrency issues

## 7.1 Buffers are module-global

Current:

```ts
const clipboardBuffers: Record<string, string[]> = {};
```

If the tool process is reused across multiple opencode sessions, buffers can leak across sessions.

For a single agent process this is fine. But for concurrency or long-lived plugin processes, you may want session isolation.

If `context` has a session id, use it:

```ts
const sessionKey = context.sessionId ?? process.pid ?? "default";
const buffers = (sessionBuffers[sessionKey] ||= {});
```

If not, at least document that buffers are process-global.

---

## 7.2 Concurrent tool calls can race

If the agent issues two parallel tool calls touching the same file, there is no lock.

For opencode, tool calls may be sequential in practice, but if parallelism is possible, consider:

- disabling this tool for parallel calls,
- using a simple file lock,
- or documenting that file operations are not safe under concurrent writes.

A simple lock can be hard to do well in Node without dependencies, but for agentic coding tools it is often acceptable to assume the agent’s tool loop is sequential.

---

# 8. Description and documentation feedback

## 8.1 The description is useful but long

Your description is very rich and teaches the agent a lot. That is valuable.

But it may cost tokens in every prompt. For opencode, if this description is loaded on every turn, it may be too large.

I would split:

### Tool description: short and operational

Example:

```text
block_transfer performs line-anchored block edits and uses named clipboard buffers.
Refs are line numbers or marker strings. A marker must match exactly one line after trimming leading indentation.
Use MOVE for one-shot move, COPY/APPEND + PASTE/REPLACE for multi-buffer workflows, WRITE for direct text region writes, PEEK/MAP for buffer inspection.
All refs are resolved against the current file state before any write.
```

Then keep the current long version as internal documentation or a knowledge note.

You already have a separate Markdown doc. That is good.

---

## 8.2 Remove proposal dates from the main description

Comments like:

```text
approved proposal 2026-09-25_block_transfer-v2, Part A
```

are useful for maintainers, but they are noise for the agent.

Keep implementation comments separate from user/model-facing description if possible.

---

## 8.3 Keep the WHEN-NOT section

This is good:

```text
WHEN-NOT — a small exact string replacement inside one file: use 'edit'
```

Agents need that guardrail. Keep it.

---

# 9. Anchor matching: good choices, with small UX tweaks

Your core rule is good:

- prefix match, not substring
- unique match required
- leading whitespace on anchor ignored
- CRLF tolerant
- all-whitespace anchor matches nothing
- non-unique anchors produce errors

This is a solid agentic design.

Do **not** re-introduce fuzzy matching. Fuzzy anchors make agent actions unpredictable.

But I would consider these small tweaks:

## 9.1 Consider trimming trailing whitespace from anchors

Current anchor trims leading spaces/tabs but not trailing spaces/tabs.

Agents sometimes copy:

```text
function foo 
```

with an invisible trailing space.

If the file line is:

```text
function foo(x)
```

anchor `function foo ` with trailing space may fail if the line does not contain a space before `(`.

For robustness, you could also trim trailing spaces/tabs from the anchor:

```ts
const trimmed = anchor.replace(/^[ \t]+/, "").replace(/[ \t]+$/, "");
```

Tradeoff: an anchor can no longer intentionally include meaningful trailing spaces. That is rarely important for block anchors.

I lean toward trimming both sides for agentic use.

---

## 9.2 Reject newline-containing refs

If an agent sends:

```ts
startMarker: "a\nb"
```

that is probably invalid.

Add:

```ts
if (/\r|\n/.test(marker)) {
  return "Error: marker refs must be single-line. Use line numbers or a one-line prefix.";
}
```

This prevents confusing matches.

---

## 9.3 Consider line number syntax `line:42`

As mentioned, string-only refs are better for models. A grammar like:

```text
42
line:42
# Section
TODO:
```

would be clean.

You can keep backwards compatibility:

- all digit string


--comment: model Next-coder2

I'll check whether the pasted source matches what's actually deployed before flagging the top item as a real defect.
## Verdict

Design is solid — the unified anchor rule, pre-call resolution, validate-before-write discipline, and teaching errors are the right shape for agentic use. But as pasted, **the file cannot load**, and several behaviors contradict the description the model is being told to trust. In agentic use, that second class is worse than a crash: a crash the agent sees, a wrong write it doesn't.

---

## P0 — blockers

**1. Syntax error in the WRITE summary line (whole tool fails to register).**
```ts
feedback.push`Wrote ${spans.length} regions ...)`);   // stray ")", missing "("
```
`feedback.push` + a template literal parses as a *tagged template*, but the unbalanced `)` is a SyntaxError → the plugin file never loads. Fix: `feedback.push(\`Wrote ...\`);`

Corollary worth acting on: `tests/block_transfer.smoke.mjs` is in the md, and it could not have missed a parse error — so either the tests don't import this module, or the file under test isn't this one. Add an import/compile smoke case before anything else.

**2. Unknown mode returns `"Operation completed."` — a silent success lie.**
`const mode = (args.mode || "MOVE")` plus the fall-through `return "Operation completed."` means any mode string that slips past the enum (host-side coercion, a typo the schema didn't catch, future mode) reports success having done nothing. This is the single worst agentic failure mode: the agent updates its world-model and moves on.
```ts
default: return `Error: unknown mode '${mode}'. Expected one of MOVE|COPY|APPEND|CUT|PASTE|REPLACE|WRITE|PEEK|MAP|DELETE|CLEAR.`;
```

**3. MOVE resolves the destination *after* the source is already cut — the exact partial-write they claim to prevent.**
The code comments promise "guard the destination BEFORE any write," but only sandbox + `dstFile` presence are pre-checked. `resolveRef(dstFile, "Target marker", ...)` runs in step 4, *after* `fs.writeFileSync(srcPath, ...)`. So a non-unique / missing / out-of-range `targetMarker` in dst → **block deleted from source, nothing written to dst, and MOVE populates no buffer**, so the content is unrecoverable from tool state. Fix the ordering: resolve dst (sandbox + targetMarker) *before* touching src; then cut; then write dst. Better still, for MOVE stash the extracted block in a shadow buffer before the cut so an error is recoverable.

**4. EOF insertion is off by one, and it silently inserts a blank line.**
`dstLines = dstText.split(/\r?\n/)`, so a file ending in `\n` has a phantom trailing `""`, and `insertIdx = dstLines.length` lands *after* it:

- dst `"# Backlog\n"` → `["# Backlog",""]` → splice at 2 → `"# Backlog\n\n<block>"` — a stray blank line every PASTE/MOVE at EOF. Repeat pastes accumulate blank lines.
- dst **nonexistent** (`dstText = ""`, `dstLines = [""]`) → splice at 1 → `"\n<block>"` — the created file gets a **leading blank line**.

The tool's own `countLines()` already models this correctly (`"a\nb\n"` = 2 lines). Use it: `insertIdx = countLines(dstText)` for the EOF case, and `0` for a nonexistent file. Note the feedback line deliberately "names the destination file but not the insertion line," so this is invisible to the agent — see P2 below.

**5. Every write converts CRLF files to LF — file-wide.**
`split(/\r/\n/)` drops all `\r`; `join("\n")` restores none. A `CUT`/`DELETE`/`REPLACE` on a CRLF file rewrites **every line ending in the file**, producing a whole-file diff in git (and your sandbox explicitly carves out a Windows temp dir, so Windows workflows are in scope). Fix: detect once and preserve —
```ts
const eol = srcRaw.includes("\r\n") ? "\r\n" : "\n";
fs.writeFileSync(p, lines.join(eol) + (srcRaw.endsWith("\n") ? eol : ""), "utf-8");
```
and re-attach `\r` to buffer lines pasted into a CRLF destination.

**6. `PEEK`'s `from`/`count` still have the bug plan25 fixed for refs.**
You documented that the live channel's constrained decoding *stringifies schema integers*, and fixed it in `resolveRef` via the `/^\d+$/` branch. `PEEK` bypasses `resolveRef` entirely: `Number.isInteger(from)` is false for `"5"`, so a valid window request returns `Error: from 5 is not a 1-based line number` — and `tool.schema.number().int()` for `from`/`count` may reject it at parse time before your code even runs. Either accept the same string-union + digit-string coercion, or drop the runtime `Number.isInteger` guards on the theory that parsing already guarantees ints. Right now the code and its own plan25 comment are mutually contradictory.

**7. "WRITE creates the file if absent" is unreachable.**
For an absent file `dstText = ""`, so `countLines = 0`: a marker ref matches nothing (`notFound` + no `Closest lines` candidates), and line-number refs `≥ 1` are `ref-out-of-range`. So the `fs.mkdirSync`/create path only ever runs for an existing file, while the description and the md both promise creation. Either implement it (e.g. allow an empty span, or a `create: true` sentinel meaning "insert at line 1") or delete the claim from the description. An agent that trusts this description will try to create a file with WRITE and burn 2–3 retries on contradictory errors.

---

## P2 — agentic-usability feedback (highest value per line changed)

**8. Report where the block *landed*, not just where it came from.** Your format spec deliberately omits the insertion line. For `MOVE`/`PASTE` that is the one number the agent needs; without it, every move provokes a follow-up read to verify. Add `(after line N)` or `(at EOF, line N+1)`.

**9. For `REPLACE`/`WRITE`, report the *new* span and the file's new line count.** You report the pre-call old span and `N = old lines removed`. That's backwards for the agent's next action: line-number refs resolve against the pre-call state, so once a replace changes the line count, all of the agent's cached numbers are silently stale — the next line-number ref writes the wrong span with no error. Emit e.g. `old 12..15 -> new 12..19, file now 143 lines`. Same for WRITE regions.

**10. Decouple reporting order from application order.** WRITE `regions` feedback is emitted "HIGHEST LINE first" (correct for splicing) but reads backwards against the file, which is a confusion tax on the model. Apply descending, report ascending.

**11. Errors on list inputs must name the item.** `refs` errors say `List ref 'foo' not found…` with no position; WRITE `regions` errors say `End marker '…' not found after start marker` with no region index. For a 7-ref / 5-region list the agent can't tell which element to fix. Add `ref 4 of 7`, `region 2 of 5 ({ start: "…", end: "…" })`.

**12. The reversed-span message teaches the wrong action.** `Error: End marker 'X' not found after start marker in f.` is a *findability* claim about a marker the tool did find — the real cause is ordering. "Not found" reliably sends agents re-searching for the marker instead of swapping. Say: `End marker 'X' (line 4) resolves before start marker 'Y' (line 9) in f — swap them or pick another anchor.` Also: REPLACE uses different wording for the same condition (`Start marker … is after end marker`) — three wordings, one failure mode; your stated design is one taxonomy.

**13. `COPY` with no input form hides two of its three forms.** The bare `COPY {}` returns `'srcFile', 'startMarker', and 'endMarker' are required for mode COPY`, which pushes the agent to the single-ref form only. APPEND already has the good "needs one input form: refs / markers+srcFile / text" message — COPY needs the same one.

**14. `MAP` is not bounded where it matters.** Verbatim, uncapped line echo means a minified/one-line JSON or CSV buffer dumps the entire payload into context — the exact thing MAP's "inspect without pasting it out" promise exists for. Truncate long lines (`200 chars` + `…`) and cap total output; keep verbatim for short lines.

**15. Buffer state is module-global.** `clipboardBuffers` is shared across concurrent chats/sessions in the same host process, so two parallel agents using `'default'` clobber each other's clipboard with no error. Key it by session (`context.sessionId ?? context.id ?? "default"`), and consider a per-op `undo` buffer so `CUT`/`DELETE`/`MOVE` are reversible — cheap insurance for the destructive modes.

**16. Presence semantics differ between form-detection and validation.** `hasSingle` uses `args.startMarker !== undefined`, while the required check uses `refPresent` (which also treats `""` as absent). So `COPY { refs: [...], startMarker: "" }` errors with "the 'refs' list takes the input alone (no markers)" for a marker the tool elsewhere calls absent. Use `refPresent` in both.

**17. Sandbox roots don't match the description on POSIX.** `process.env.TEMP ?? process.env.TMP` — on macOS/Linux the real variable is `TMPDIR`, so the advertised temp root usually isn't in `roots`, and legit `/tmp` writes get a confusing "outside the sandbox." Use `os.tmpdir()`. Two related nits: `.toLowerCase()` comparison is *correct* for Windows but over-permissive on case-sensitive filesystems (a cwd containing uppercase can admit a genuinely-outside path), and `path.resolve` is lexical, so a symlink inside cwd escapes. If the intercept plugin is the real boundary, say so in the code comment (you do) and use `fs.realpathSync` here as the backstop it claims to be.

---

## P3 — schema & prompt surface (this is where "agentic use" cost lives)

**18. The description is ~3k characters and duplicates itself three ways** (description ↔ 12 `describe()` strings ↔ the md). Two problems: it's injected into every request in opencode, and multi-copy rules *drift* (e.g. the description promises REPLACE "never creates a file" and WRITE "creates if absent" — one of those pairs is already wrong). Pick a single source: keep the description to mode-families + the 4 rules that actually change behavior (digit-string = line number; anchors must match exactly one line; buffer is replaced vs appended; pre-call resolution), let each `describe()` carry only its own arg, and let the **error messages** carry the recovery detail — your `Closest lines:` machinery is already better teaching than any prose, and it's paid only on failure. Target: description < ~800 tokens.

**19. `\\n` in the ASSEMBLY sentence renders as a literal backslash-n** to the model ("joined by EXACTLY ONE `\\n`" inside a template literal). Use `one newline` in prose; escape noise is exactly what makes models mis-emit string literals.

**20. 11 modes × 12 optional args is a combinatorial surface JSON Schema can't constrain**, so every invalid combination costs a round-trip. You partly accept that tradeoff (runtime errors are good). If you want to cut it: `PEEK`/`MAP` collapse into one mode with a flag; `DELETE` = `CUT` into a discarded buffer; `MOVE` = `CUT`+`PASTE`. But I'd keep the conveniences — they're poka-yoke, and they *are* the modes agents reach for. If splitting, split by arg shape (`*_to_buffer`, `*_from_buffer`, `*_inspect`) rather than by verb.

**21. Add one line of "which ref form to use when."** The rules are stated, the guidance isn't. For agents this is the highest-leverage sentence you could add: *prefer markers in files you just read; use line numbers when the anchor would be non-unique (`}`, `const`, `##`) — but line numbers are only valid for the file state you last saw, so refresh them after any REPLACE/WRITE.* That converts a class of silent corruption into a stated precondition.

**22. Type-hygiene nits:** `args: any` / `context: any` + `context.directory || process.cwd()` — if the host's field isn't `directory`, the sandbox root silently becomes the plugin process cwd, which changes which paths are allowed *and* reported in errors. Worth a defensive read of both `context.directory` and `context.worktree`. Also `CLEAR` on a nonexistent buffer reports success (`- buffer: 0 lines`) — fine as idempotence, but say `was already empty` so the agent doesn't believe it had state.

**23. The catch-all swallows stack** (`block_transfer failed: ${err.message}`) — for a tool that returns failures as ordinary strings, include enough to localize (`mode`, the file, and `err.stack` on a debug flag) or you'll be bisecting by eye.

---

## What's good (keep it)

`matchAnchorLines` as the single point of matching change; resolve-everything-before-any-write for COPY/APPEND/REPLACE/WRITE-regions; the digit-string ref fix; deterministic capped feedback; the `Closest lines:` candidate hint with a deterministic score and a byte-identical fallback; `countLines` matching the `/\r?\n/` convention; `last_write` auto-buffer as poka-yoke for failed anchor resolution.

## If you only do five

1. Fix the `feedback.push` syntax error (tool is dead as pasted).
2. Move MOVE's dst/targetMarker resolution before the source cut (unrecoverable data loss).
3. Preserve destination line endings (silent whole-file CRLF→LF churn).
4. Fix EOF / nonexistent-file insertion index via `countLines` (stray leading and trailing blank lines).
5. Add `unknown mode` error + report insertion line and new spans/line counts (closes the stale-line-number hazard that your line-number refs create).

Then extend the smoke tests to cover: module import, unknown mode, EOF insert, CRLF round-trip, MOVE with a bad `targetMarker`, WRITE into a nonexistent file, `PEEK` with `from:"5"`.
