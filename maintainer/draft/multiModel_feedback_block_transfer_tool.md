based on the our block_transfer.ts file I simply requested feedback. This is the result.
I want you too look over it and make a recommendation if such feedback is useful and then i can add this model as a reviewer agent with an adapted model. (some you are running on but very high thinking enabled - you run on medium):
- my idea is to have it check specific tools/plugins and give feedback. but it might need to be bounded because the thinking uses a lot of tokens. 150-170k context window, but fast as compensation.

---
# feedback xhigh thinking:
---
Overall: this is a strong, well-thought-out agentic tool. The unified anchor rule, named buffers, teaching errors, and bounded inspection modes are all good choices. But there are a few reliability/security issues that I would fix before trusting it for autonomous file surgery.

I’d classify the tool as: **good design, but not yet safe enough for unsupervised use** due to line-ending/EOF handling, `MOVE` ordering, sandbox path checks, and unbounded output/memory paths.

---

## 1. Critical: file line representation is fragile

The biggest issue is that the tool uses raw `text.split(/\r?\n/)`, which preserves a trailing empty element when the file ends with a newline. That representation is okay for some slice operations, but it breaks append-style behavior and empty files.

### A. Appending to an existing file ending in a newline adds an extra blank line

Example:

```ts
// dstFile contains:
// a
// b
// <newline>

const dstLines = dstText.split(/\r?\n/); // ["a", "b", ""]
const insertIdx = dstLines.length;        // 3
dstLines.splice(insertIdx, 0, ...buffer);
// result: ["a", "b", "", ...buffer]
```

So `PASTE` without `targetMarker`, or `MOVE` without `targetMarker`, will often insert a blank line before the new block and may also drop the file’s final trailing newline.

Example:

```text
before:
a
b

after PASTE ["x"] with no target:
a
b

x
```

That is not “append at EOF”. It is “insert after a synthetic final blank element”.

The same affects empty existing files:

```ts
"".split(/\r?\n/) // [""]
```

So pasting into an existing zero-byte file creates a leading blank line.

### B. CRLF files are normalized to LF

The tool is CRLF-tolerant for matching, but writes always use:

```ts
dstLines.join("\n")
```

So a Windows/CRLF file will become LF after any successful `MOVE`, `PASTE`, `REPLACE`, or `WRITE`. That can create noisy diffs and is probably undesirable on Windows.

### Suggested fix

Introduce a single parse/serialize representation and use it everywhere:

```ts
type ParsedFile = {
  lines: string[];
  trailingNewline: boolean;
  eol: "\n" | "\r\n";
};

function parseFile(text: string): ParsedFile {
  if (text.startsWith("\uFEFF")) {
    text = text.slice(1);
  }

  const eol: "\n" | "\r\n" = text.includes("\r\n") ? "\r\n" : "\n";
  const trailingNewline = /\r?\n$/.test(text);

  const rawLines = text.split(/\r?\n/);
  const lines = trailingNewline ? rawLines.slice(0, -1) : rawLines;

  return { lines, trailingNewline, eol };
}

function serializeFile(parsed: ParsedFile, lines: string[]): string {
  if (lines.length === 0) {
    return parsed.trailingNewline ? parsed.eol : "";
  }

  return lines.join(parsed.eol) + (parsed.trailingNewline ? parsed.eol : "");
}
```

Then use conceptual `lines.length` for insertions, and use `serializeFile` for writes.

That fixes:

- empty files;
- trailing-newline append behavior;
- multiple trailing blank lines;
- CRLF preservation;
- mismatch between `countLines()` and actual array indices;
- some `MOVE`/`PASTE` edge cases.

Minimum hotfix, if you do not refactor fully:

```ts
let dstLines = dstText === "" ? [] : dstText.split(/\r?\n/);
let insertIdx = dstText.endsWith("\n") ? dstLines.length - 1 : dstLines.length;
```

But I would do the full parse/serialize refactor.

---

## 2. Critical: `MOVE` can partially mutate the source

Your docs and comments emphasize “no partial writes on rejection”, but `MOVE` currently violates that in several cases.

Current flow for `MOVE`:

1. validate source;
2. read source;
3. resolve `startMarker` / `endMarker`;
4. extract block;
5. **write source** — remove the block;
6. read destination;
7. resolve `targetMarker`;
8. write destination.

That means:

- If `targetMarker` is invalid in the destination, the source has already been cut.
- If destination read fails after source write, the source has already been cut.
- If destination write fails after source write, the source has already been cut.
- If `srcFile === dstFile`, `targetMarker` is resolved against the post-cut destination state, which is not the pre-call state and can be wrong.

Example:

```text
srcFile:
1. line A
2. line B
3. line C
4. line D

MOVE:
  startMarker: "line B"
  endMarker: "line C"
  targetMarker: "MISSING"
```

Current behavior: lines B and C are removed from source, then the tool returns a target error. The block is gone from the source and not saved in a buffer.

### Suggested fix

For `MOVE`, validate everything against the pre-call state before writing anything:

- resolve source start/end;
- read destination;
- resolve destination `targetMarker` before writing source;
- validate destination file type and parent directory;
- reject same-file moves, or implement same-file moves using the pre-call file state.

For cross-file `MOVE`, a safer ordering is:

1. read source;
2. read destination;
3. resolve all refs against pre-call state;
4. write destination first, inserting the extracted block;
5. write source, removing the block.

If destination write succeeds but source removal fails, you have duplication, which is safer than data loss.

I would also consider rejecting:

```ts
path.resolve(srcPath) === path.resolve(dstPath)
```

or canonical same-file cases via `realpath`. Same-file moves are subtle enough that I would not expose them unless explicitly tested.

---

## 3. Critical: `WRITE` cannot actually create an absent file

Your description says:

> WRITE ... creates the file if absent.

But the implementation reads an absent file as:

```ts
const dstText = "";
```

Then it requires `startMarker` and `endMarker` or `regions`. In an empty file, no marker can be found and no line number can be valid. So a successful anchored `WRITE` cannot happen for an absent file.

So either:

1. change the docs to say `WRITE` requires an existing file, or  
2. add a real create/append behavior.

If you want `WRITE` to create files, I’d suggest making no-span create explicit, for example:

```json
{
  "mode": "WRITE",
  "dstFile": "new.txt",
  "text": "hello\n"
}
```

where omitted span fields on a nonexistent file mean “create new file from `text`”. But that adds semantic surface, so maybe the simpler fix is to remove the “creates file if absent” claim and let `PASTE` create files.

---

## 4. High priority: sandbox path logic can be too loose or misleading

Current sandbox:

```ts
const resolved = path.resolve(cwd, givenPath).toLowerCase();
const r = path.resolve(root).toLowerCase();
return resolved === r || resolved.startsWith(r + path.sep);
```

There are a few issues.

### A. Lowercasing all paths on all OSes is dangerous on Linux/macOS

Example on a case-sensitive POSIX system:

```ts
cwd = "/home/User"
path = "/home/user/secret"
```

Both become `/home/user...`, so the second can pass. On case-insensitive filesystems that may not matter, but on Linux it can allow access to a sibling directory with the same name but different case.

Use OS-specific case handling, or avoid lowercasing entirely. `path.relative` is usually cleaner.

### B. `root + path.sep` can misbehave for root `/`

On POSIX:

```ts
"/" + path.sep => "//"
```

So paths under `/` may fail the prefix check unless handled specially.

### C. Symbolic links are not resolved

A path can resolve lexically inside the sandbox but point outside:

```text
cwd/link -> /
cwd/link/etc/passwd
```

Lexically that is inside `cwd`, but access escapes.

### D. Windows temp assumption

The docs say “Windows temp directory”, but the code uses `process.env.TEMP ?? process.env.TMP`. On non-Windows systems that can be undefined or behave inconsistently. Use `os.tmpdir()` if you want a cross-platform temp root.

### Suggested replacement shape

```ts
import os from "os";

function allowedRoots(cwd: string): string[] {
  return [cwd, os.tmpdir()].filter(Boolean).map((p) => path.resolve(p));
}

function isInside(root: string, target: string): boolean {
  const rel = path.relative(root, target);
  return rel === "" || (!rel.startsWith("..") && !path.isAbsolute(rel));
}

function lexicalCheck(cwd: string, givenPath: string): string | null {
  const roots = allowedRoots(cwd);
  const resolved = path.resolve(cwd, givenPath);

  if (!roots.some((root) => isInside(root, resolved))) {
    return `Error: '${givenPath}' is outside the sandbox (allowed: ${roots.join(", ")})`;
  }

  return null;
}

function canonicalCheck(cwd: string, filePath: string): string | null {
  const roots = allowedRoots(cwd);

  try {
    const realFile = fs.realpathSync.native(filePath);
    if (!roots.some((root) => isInside(fs.realpathSync.native(root), realFile))) {
      return `Error: '${filePath}' resolves outside the sandbox.`;
    }
  } catch {
    // File may not exist yet. Check parent directory if possible.
  }

  return null;
}
```

For write destinations that do not exist yet, canonicalize the deepest existing ancestor or the parent directory. Also add file/directory checks with `statSync` before reads/writes.

---

## 5. High priority: some output paths are unbounded

This matters a lot for agentic use because tool output becomes context.

### A. `COPY`/`APPEND` with a large `refs` list

Current feedback for a `refs` list uses:

```ts
rangeText: resolved.join(", ")
```

If the model passes 10,000 refs, the feedback line becomes huge.

Suggest capping it:

```ts
function boundedLineList(nums: number[]): string {
  if (nums.length <= 10) return nums.join(", ");
  return `${nums.slice(0, 8).join(", ")}, ..., ${nums.slice(-2).join(", ")} (total ${nums.length})`;
}
```

### B. `MAP` lines are not length-capped

Your `PEEK` mode caps echoed lines at 40 chars, which is good. But `MAP` says:

> lines echoed VERBATIM WITH line numbers (no cap, blank lines kept)

A single huge line can make `MAP` enormous. That is risky in a loop.

Suggest:

- cap each `MAP` line to e.g. 80–120 chars, with `...`;
- or cap total `MAP` output to e.g. 2000 chars;
- keep line numbers exact.

If you want “verbatim structure”, maybe show line numbers and truncated content, not full content.

### C. Errors quote anchors as given

If the model supplies a huge anchor string, the error can echo that whole string. I would cap echoed anchors in errors too.

### D. Buffers have no size limits

Currently:

```ts
const clipboardBuffers: Record<string, string[]> = {};
```

There is no limit on:

- number of buffers;
- number of lines per buffer;
- total bytes;
- line length;
- text size accepted by `COPY text` / `APPEND text` / `WRITE text`.

For long agentic sessions, this is a memory and context risk.

I would add something like:

```ts
const MAX_BUFFER_BYTES = 1_000_000;
const MAX_TOTAL_BUFFER_BYTES = 5_000_000;
const MAX_BUFFER_COUNT = 50;
```

and reject or trim with a clear error.

---

## 6. High priority: schema validation is incomplete in `execute`

Several places assume field shape even when the schema should prevent it.

### A. `mode` should probably be required

Current:

```ts
const mode = (args.mode || "MOVE").toUpperCase();
```

If the model omits `mode` but provides source/span fields, the tool implicitly behaves like `MOVE`. I’d rather return:

```ts
"Error: 'mode' is required."
```

That avoids surprising implicit behavior.

### B. `regions` items should require `start` and `end`

Current schema:

```ts
regions: tool.schema.array(tool.schema.object({
  start: ...,
  end: ...
}))
```

But `execute` does:

```ts
resolveRef(..., region.start)
resolveRef(..., region.end)
```

If `start` or `end` is missing, you can get a runtime type error instead of a teaching error.

Make them required, and also validate in `execute`:

```ts
if (region == null || !refPresent(region.start) || !refPresent(region.end)) {
  return "Error: each WRITE 'regions' item must have both 'start' and 'end'.";
}
```

### C. `refs` items should be validated defensively

If `refs` contains `null`, boolean, object, etc., current code can throw inside `resolveRef`. Add:

```ts
if (typeof ref !== "string" && typeof ref !== "number") {
  return { error: `Error: invalid ref ${String(ref)}. Use a marker string or a line number.` };
}
```

### D. Reject irrelevant fields per mode

Currently extra fields are silently ignored. For example:

- `text` on `MOVE`
- `regions` on `PASTE`
- `from` / `count` on `COPY`
- `bufferName` on `DELETE`

Silent ignoring is usually fine, but for an agent tool it can hide mistakes. I’d consider stricter mode-specific validation, at least for mutually exclusive fields.

---

## 7. High priority: feedback shape could be better for agent recovery

Your feedback is already much better than typical. A few improvements would help.

### A. Include the actual insertion line for `MOVE` and `PASTE`

Your docs explicitly say the return line names the destination file but not the insertion line. That is a real gap.

Current:

```text
Moved 3 lines from 'a.md' to 'b.md' (lines 4..6, first: '## TODO').
```

More useful:

```text
Moved 3 lines from 'a.md' to 'b.md' (lines 4..6, first: '## TODO') - inserted after line 2.
```

or:

```text
Pasted 5 lines from buffer 'default' into 'b.md' (lines 1..5, first: '## Section') - inserted at EOF.
```

This lets the model continue planning without reading the file back immediately.

### B. Make `REPLACE` old/new counts explicit

Current `REPLACE` says:

```text
Replaced N lines in 'file' with buffer ...
```

where `N` is the old span size. That is documented, but easily misread.

Better:

```text
Replaced 3 old lines with 5 new lines in 'file' from buffer 'default' (old lines 7..9, first: '## New Section') - buffer: 5 lines.
```

Or at least:

```text
Replaced 3 lines with buffer 'default' (5 lines) in 'file' (old lines 7..9, first: '## New Section') - buffer: 5 lines.
```

### C. Improve empty-buffer errors

Current:

```text
Error: Clipboard buffer 'default' is empty. Perform a COPY or CUT first.
```

But buffers can also come from:

- `APPEND`
- `CUT`
- `COPY`
- `WRITE` through `last_write`

I’d use:

```text
Error: Clipboard buffer 'default' is empty. Use COPY, APPEND, CUT, or WRITE to populate it first.
```

### D. Add recovery hints to errors

For example:

```text
Error: Start marker 'foo' not found in 'file.md'. Use a longer unique line prefix or a line number.
```

```text
Error: line 12 is out of range in 'file.md' (the file has 8 lines). Use a line number between 1 and 8.
```

The current errors are already good; adding explicit “what to do next” will improve agent loop success.

### E. Non-unique anchor hints could include last match, not only first three

Current:

```text
12 matches: lines 3, 7, 12 …
```

This is bounded, which is good. But if the first matches are not the relevant one, the model may not know. I would maybe show:

```text
12 matches: first lines 3, 7, 12; last line 140
```

or, for small counts, show all.

---

## 8. Medium priority: `MOVE` and destination file type checks

You should explicitly guard destination type.

For example, if `dstFile` exists and is a directory, `readFileSync` or `writeFileSync` can throw. In `MOVE`, that can happen after the source has already been cut.

Add:

```ts
if (fs.existsSync(dstPath) && fs.statSync(dstPath).isDirectory()) {
  return `Error: dstFile '${args.dstFile}' is a directory.`;
}
```

Same for source:

```ts
if (fs.existsSync(srcPath) && fs.statSync(srcPath).isDirectory()) {
  return `Error: srcFile '${args.srcFile}' is a directory.`;
}
```

Also check that the parent directory exists or is creatable before modifying source in `MOVE`.

---

## 9. Medium priority: empty `WRITE text` is probably dangerous

Currently:

```ts
const newLines = textToLines(args.text);
```

For `text: ""`, `newLines` is `[]`. Then `WRITE` replaces a span with zero lines. That is effectively deletion, but it is not labeled as deletion.

I would either:

1. reject empty `text` for `WRITE`, or  
2. require an explicit `allowEmpty: true`, or  
3. rename the behavior clearly in feedback:

```text
Wrote 0 lines to 'file.md' (old lines 3..5 removed).
```

Also note that `WRITE` with empty text overwrites `last_write` with an empty buffer, which can be surprising.

---

## 10. Medium priority: BOM handling

If a file starts with a UTF-8 BOM:

```text
\uFEFF# Title
```

then an anchor like:

```ts
"# Title"
```

will not match the first line because the actual first line begins with the BOM character.

Strip BOM for parsing/matching, and ideally preserve it on write if the original had one.

---

## 11. Medium priority: buffer storage should be scoped per session/cwd

You use a module-level object:

```ts
const clipboardBuffers: Record<string, string[]> = {};
```

If the host process handles multiple sessions or working directories, buffers can leak across them.

I’d use:

```ts
const buffers = new Map<string, string[]>();
```

and key buffers by:

```ts
`${sessionId ?? "global"}::${cwd}::${bufferName}`
```

If OpenCode provides a session identifier in `context`, use it. If not, at least namespace by `cwd`.

Also, using `Map` avoids weird prototype-key behavior if someone uses buffer names like `__proto__`.

---

## 12. Medium priority: atomic single-file writes

You use:

```ts
fs.writeFileSync(dstPath, text, "utf-8");
```

For robustness, especially in long agentic sessions, consider:

```ts
const tmp = dstPath + `.tmp-${process.pid}`;
fs.writeFileSync(tmp, text, "utf-8");
fs.renameSync(tmp, dstPath);
```

That reduces the chance of leaving a partially written file if the process crashes mid-write.

For two-file `MOVE`, a full transaction is hard, but you can still reduce risk with careful ordering and recovery state.

---

## 13. Lower priority: description is very long

The description is thorough, but it is also large. For an agent tool, that matters because it contributes to prompt/context cost.

I would keep the core contract but shorten it. Suggested structure:

1. One-paragraph purpose.
2. Mode list in one compact line each.
3. `REFS` rule in one paragraph.
4. `ANCHORS` rule in one paragraph.
5. `BUFFER` rules.
6. `ERRORS` / recovery.
7. One or two examples.

Remove internal labels like:

- `plan25`
- `S1/S2/S3`
- `approved proposal`
- `probe-pinned`
- internal version notes

Those are useful for your changelog, but not useful for the agent.

Also, the phrase:

> ASCII-safe — no non-ASCII / dense-numeral oldString problems

is a little off. The tool is not “ASCII-safe”; it is UTF-8 line-anchored. The real advantage is that it avoids exact `oldString` matching. I’d say:

> Line-anchored, so it avoids exact-string/dense-numeral edit failures. It cannot replace mid-line substrings.

---

## 14. Lower priority: `last_write` is convenient but potentially surprising

Every `WRITE` overwrites `last_write`. If a user or agent intentionally names a manual buffer `last_write`, it can be clobbered.

Options:

1. Document it strongly.
2. Reserve it and reject non-WRITE uses of `bufferName: "last_write"`.
3. Use a private reserved name like `__last_write`, but expose it only via special semantics.
4. Keep `last_write` but make overwrites explicit in feedback? You currently say it is silent, which is intentional poka-yoke, but maybe too magical.

I’d at least make it obvious in docs and error/feedback messages.

---

## 15. Lower priority: `PEEK` default overlap can confuse

For small buffers, `PEEK` head and tail can overlap:

```text
head: 1, 2, 3
tail: 1, 2, 3
```

That is documented, but it can be confusing in logs.

Alternative for small buffers:

```text
buffer 'x' (3 lines): 1: 'a', 2: 'b', 3: 'c'
```

or:

```text
head: 1, 2, 3
tail: (same as head)
```

Not a big issue, but easier for the model to interpret.

---

## 16. Lower priority: `MAP` heading detection is Markdown-only

Current:

```ts
/^#{1,6} /
```

This matches:

```text
# Title
## Section
```

but not:

```text
#Title
```

That is fine for Markdown, and the docs say Markdown H1-H6. I’d just make sure the docs do not imply general heading detection.

Also, for code buffers with `# comment`, the docs say the verbatim echo makes it self-evident. That is acceptable.

---

## 17. Recommended regression tests

I would add tests for at least these cases.

### Line representation / EOF

1. `PASTE` into nonexistent file.
2. `PASTE` into existing empty file.
3. `PASTE` into file with no trailing newline.
4. `PASTE` into file with one trailing newline.
5. `PASTE` into file with multiple trailing blank lines.
6. `MOVE` into file with trailing newline, no `targetMarker`.
7. `MOVE` into file with target at last line.
8. `REPLACE` last line in file with trailing newline.
9. `WRITE` replacing last line in file with trailing newline.
10. CRLF file preservation through all modes.

### `MOVE` safety

11. `MOVE` with invalid destination `targetMarker` must not modify source.
12. `MOVE` to nonexistent destination creates destination correctly.
13. `MOVE` destination path outside sandbox rejected before source write.
14. `MOVE` destination is directory rejected before source write.
15. `MOVE` same source and destination rejected or handled predictably.
16. `MOVE` source write succeeds but destination write fails — assert behavior is safe/recoverable.

### Refs / anchors

17. Marker that is all digits is treated as line number.
18. Line number `0` rejected with teaching error.
19. Line number above file length rejected with count.
20. Marker with leading indentation matches trimmed line.
21. All-whitespace marker matches nothing.
22. Non-unique anchor returns match count and first matches.
23. Not-found anchor returns bounded closest lines.
24. CRLF file marker resolution.
25. Marker that matches exactly once only.
26. Marker appearing inside trailing newline area.
27. Line number refs against pre-call state in `WRITE regions`.

### `WRITE`

28. `WRITE` to absent file cannot succeed with anchored span, or create mode works if implemented.
29. `WRITE` with overlapping regions rejected.
30. `WRITE` with multiple non-overlapping regions applied highest-first.
31. `WRITE` empty text behavior.
32. `WRITE` stores `last_write`.
33. `WRITE` with missing `region.start` or `region.end` returns teaching error.
34. `WRITE` regions using markers and line numbers mixed.

### Sandbox

35. Path inside cwd.
36. Path using `..` that escapes cwd.
37. Absolute path outside sandbox.
38. Path inside temp root.
39. Symlink inside cwd pointing outside.
40. Case-sensitive POSIX case mismatch.
41. Root `/` behavior, if temp root can be `/`.
42. Destination parent directory is a symlink outside sandbox.

### Buffers / memory

43. `COPY` replaces buffer, does not append.
44. `APPEND` creates buffer if absent.
45. `APPEND` to empty buffer.
46. Failed `COPY` leaves buffer unchanged/absent.
47. `PEEK` empty buffer.
48. `MAP` empty buffer.
49. `PEEK` from/count boundary.
50. `MAP` huge line length capped.
51. Large `refs` list feedback capped.
52. Multiple buffers coexist.
53. Buffer namespaced by cwd/session if applicable.

---

## 18. Suggested priority order

If you only have time to fix a few things first:

1. **Fix line parse/serialize/EOF/CRLF handling.**  
   This is the biggest correctness issue.

2. **Fix `MOVE` ordering and same-file behavior.**  
   This is the biggest safety issue.

3. **Fix sandbox path logic.**  
   Lowercase prefix check and symlink escape are real security concerns.

4. **Fix the `WRITE` creates-file claim.**  
   Either implement it or remove the doc claim.

5. **Bound output and buffer memory.**  
   This is important for long agentic sessions.

6. **Tighten schema validation and error messages.**  
   This improves agent recovery and prevents runtime type errors.

---

## 19. Things I would keep

The core design is good:

- one unified anchor rule;
- line-number vs marker disambiguation;
- teaching errors;
- named buffers;
- `PEEK` bounded;
- `WRITE` pre-call resolution for regions;
- explicit “no partial state on rejection” philosophy;
- separation of `REPLACE` and `PASTE`;
- `last_write` convenience.

The main problem is not the API concept. The main problem is that the low-level file-line handling and `MOVE` sequencing do not yet fully match the stated safety contract.

If those are fixed, this becomes a very useful tool for agentic multi-file block surgery.


---
# feedback of default Qwen3.8-3QS model without any system prompt and the same feedback ask as before
---

# Feedback on `block_transfer` Tool

## Overall Assessment

This is a well-architected agentic tool. The unified anchor rule, the teaching-error taxonomy, the "no partial state on rejection" invariant, and the consistent feedback format all point to someone who has iterated on this through real agent interactions. The design is clearly battle-tested. A few issues below, but nothing structural.

---

## Strengths

- **Single anchor rule, single matching function.** Routing every mode through `matchAnchorLines` → `resolveAnchor` → `resolveRef` is the right call. The comment block explaining *why* it's centralized (swapping behavior = one place) will save the next maintainer hours.
- **Teaching errors that self-correct.** The "Closest lines:" hint in `notFoundError` is exactly what an agent needs to fix its own input without a second read. The non-unique error carrying match count + first 3 line numbers is bounded and actionable. The overlap error naming both spans with actual line numbers is clear.
- **The "no partial state" invariant is consistently maintained.** Every mode resolves all refs, validates all paths, checks sandbox, checks existence — *before* any `fs.writeFileSync`. An agent that gets an error can safely retry without fear of a half-mutated file.
- **Feedback format is parseable and consistent.** One line per op, predictable shape, the `- buffer: N lines` tail is only on buffer ops. An agent (or a parser) can reliably extract what happened.
- **The `WHEN` / `WHEN-NOT` / `MODES` structure in the description** is well-suited for LLM tool selection. The "use `edit` for mid-line substring" routing note is the single most important sentence for preventing tool misuse.
- **Poke-yoke on `last_write`.** Auto-storing WRITE's text into a fixed buffer name means an agent that fails an anchor on a subsequent op can PEEK/MAP `last_write` without re-typing. The "no parameter to request it" constraint prevents accidental clobber of a named buffer.
- **The value-based digit-string disambiguation** in `resolveRef` correctly handles the host's stringification quirk. The plan25 comment explains *why* the type branch was dead — future readers won't "fix" it back.

---

## Issues (ranked by severity)

### 1. MOVE has an atomicity gap on I/O failure

The source write (`fs.writeFileSync(srcPath, remainingLines.join(...))`) happens **before** the destination write. If the destination write throws (disk full, permission denied, path becomes invalid between check and write), the source is already cut. The `try/catch` returns an error string, but the block is lost from the source and never landed in the destination.

This isn't a logical-rejection issue (those are all pre-checked), it's a filesystem I/O failure. For an agentic tool that might run for hours, this is the one scenario where data is silently destroyed.

**Suggested fix:** Either (a) write to a temp file in the sandbox, `rename` it over the destination, and *then* write the source — so a rename failure leaves both files intact; or (b) accept the risk explicitly in the doc and add a note that MOVE is not atomic on I/O errors. Option (a) is ~6 lines:

```ts
// In MOVE, after extracting the block and before writing src:
const dstTmp = path.join(path.dirname(dstPath), `.${path.basename(dstPath)}.bt_tmp`);
fs.mkdirSync(path.dirname(dstPath), { recursive: true });
fs.writeFileSync(dstTmp, dstLines.splice(insertIdx, 0, ...extractedBlock).join("\n"), "utf-8");
fs.renameSync(dstTmp, dstPath);
// NOW write the source (safe to do after dst is committed)
const remainingLines = [...srcLines.slice(0, startIdx), ...srcLines.slice(endIdx + 1)];
fs.writeFileSync(srcPath, remainingLines.join("\n"), "utf-8");
```

If `renameSync` fails, the temp file is garbage (cleanable) and the source is untouched.

### 2. PEEK `from`/`count` may receive stringified integers

The schema is `tool.schema.number().int()` (not a union), so the host *should* pass a number. But the plan25 notes explicitly state that the live channel's constrained decoding stringifies integers. If the host applies the same decoding path to pure-number schema fields (not just unions), `args.from` arrives as `"3"` and `Number.isInteger("3")` is `false` → spurious error.

**Suggested fix (defensive, one line):**

```ts
const from = typeof args.from === "string" ? parseInt(args.from, 10) : args.from;
const count = typeof args.count === "string" ? parseInt(args.count, 10) : args.count;
```

This is cheap insurance and mirrors what `resolveRef` already does for the digit-string case.

### 3. `last_write` clobber can surprise an agent

If an agent does:

1. `WRITE` (text → region) → `last_write` = text A
2. `COPY` from file into buffer `"last_write"` (explicit `bufferName: "last_write"`)
3. `WRITE` (different text → region) → `last_write` = text B (text A is gone)

The agent's step 2 data is silently destroyed. The poka-yoke intent is "you can't lose the WRITE text," but it creates a collision with a user-chosen buffer name.

**Suggested fix:** Either (a) reserve the name `"last_write"` and reject `bufferName: "last_write"` on all other modes with a teaching error, or (b) prefix it internally (`_\x00last_write`) so it's unreachable from the API but still auto-stored. Option (a) is simpler and the error message teaches the constraint:

```ts
if (bufferKey === "last_write" && mode !== "WRITE") {
  return `Error: the buffer 'last_write' is reserved (auto-filled by WRITE). Use a different bufferName.`;
}
```

### 4. Line-splitting: `matchAnchorLines` uses `"\n"`, everything else uses `/\r?\n/`

This is functionally correct (the `\r` strip makes line indices align), but it's a subtle maintenance trap. If someone later changes the matching to something that *isn't* prefix-based (e.g., a regex that could match across the `\r`), the inconsistency bites.

**Suggested fix:** Split on `/\r?\n/` in `matchAnchorLines` too, and drop the per-line `endsWith("\r")` check. The code becomes:

```ts
const lines = fileText.split(/\r?\n/);
for (let i = 0; i < lines.length; i++) {
  if (lines[i].replace(/^[ \t]+/, "").startsWith(trimmed)) out.push(i + 1);
}
```

This makes the invariant "all line splitting in this file uses `/\r?\n/`" true and removes the mental step of "oh, but the `\r` is handled separately here."

### 5. `WRITE` with `regions`: same text for all regions, no per-region text

This is a design constraint that's implicit. An agent that wants to write different content to three regions must issue three separate `WRITE` calls (each overwriting `last_write`). The description says "one 'text' into a single span … or into a 'regions' LIST" but doesn't explicitly say "the same text goes into every region."

**Suggested fix:** Add one sentence to the `regions` schema description: *"All regions receive the SAME 'text' content. For per-region content, issue multiple WRITE calls."*

---

## Smaller / Cosmetic

| Item | Note |
|---|---|
| **"ASCII-safe" in the description** | Misleading. The tool handles any Unicode fine. The actual point is "no exact-string-match failure modes." Suggest: *"avoids exact-string-match failure modes (mid-line substrings, dense numerals)"* and drop "ASCII-safe." |
| **`countLines` / `resolveRef` / `matchAnchorLines` are exported** | Fine for testing, but the `export` on a module that's also a host plugin tool might be confusing if the host imports the file and sees exports. If the host only calls `execute`, the exports are harmless. Just noting. |
| **`sandboxCheck` lowercases on Linux** | `path.resolve(...).toLowerCase()` is correct for Windows. On Linux, two files `Foo.txt` and `foo.txt` in the same directory are distinct, but the sandbox check would treat them as the same path. Since the sandbox only gates *allow/deny* (not file identity), and the actual `fs` calls use the original case, this is harmless in practice. No change needed, but a comment noting "lowercase for Windows case-insensitivity; Linux `fs` calls preserve original case" would prevent a future "bug report." |
| **`return "Operation completed."` at the end** | Unreachable (all modes return before it). Harmless, but if a future mode is added and forgets to return, the agent gets a non-informative string. Consider replacing with `return "Error: unknown mode."` as a tripwire. |
| **`assemblyFeedback` is only called by COPY/APPEND** | The other modes use `opLine`. The two functions produce the same shape. If they ever diverge, the consistency breaks. Consider having `assemblyFeedback` call `opLine` internally (it already has all the args). |
| **Buffer memory in long sessions** | No size cap, no eviction. A session that does 500 COPY/APPEND ops into 20 named buffers holds all of them. For a single agent session this is fine (<100 MB of strings). No action needed, but a one-line comment noting the assumption would help. |

---

## Documentation (`block_transfer.md`)

The .md is well-scoped and appropriately terse. A few notes:

- **"Host-named custom tool (no `name` field — the file IS the tool name…)"** — This is the single most important integration fact. Good that it's in the first line.
- **The "Gotchas / lessons" section** is the right level of detail. The "swapping the general matching behavior is a change in `matchAnchorLines` ONLY" line is the key invariant a future contributor needs.
- **The "Knowledge notes (pointer-only)" section** is a good pattern — it avoids duplicating decision context while keeping the pointer discoverable.
- **Missing:** A one-line note that MOVE is not atomic on I/O errors (issue #1 above), and a one-line note that `last_write` is reserved (issue #3 above). These are the two "gotchas" an agent (or a maintainer) will actually hit.
- **The "Tests" section** lists two smoke files. If the test suite covers the "Closest lines:" candidate hint, the non-unique format, and the WRITE regions overlap rejection, a parenthetical "(covers anchor errors, ref disambiguation, WRITE regions, sandbox)" would help a new reader know coverage without reading the test file.

---

## For the Description (the LLM-facing string)

The description is thorough, but at ~2,500 words it's dense. An LLM reading a tool schema sees this *every time it considers the tool*. Two concrete suggestions:

1. **Move the "RETURN LINES" section** into a comment above `execute` (or the .md) rather than the description. The agent doesn't need to *parse* the return format from the description to *call* the tool — it reads the return *after* the call. The description's job is to get the agent to (a) pick this tool over `edit`/`write`, and (b) fill in the right args. The return format is for *post-hoc* understanding.

2. **The "EDGE" paragraph** (non-unique, out-of-range, overlap, etc.) is valuable but could be tightened to: *"Errors are self-descriptive: non-unique anchors carry the match count + first 3 line numbers; missing files, out-of-range line numbers, overlapping WRITE regions, and out-of-sandbox paths each name the cause. Read the error, fix the input, re-issue."* The specific formats are in the code and the .md; the agent only needs to know *that* errors are actionable.

This would cut the description by ~30% while keeping all decision-relevant information.

---

## Summary

The tool is in good shape. The unified anchor rule, the teaching errors, and the no-partial-state invariant are the right design choices and they're well-implemented. The two items I'd actually change before the next agent hits them are **the MOVE atomicity gap** (#1) and **the `last_write` clobber** (#3). The PEEK stringification (#2) is cheap insurance. Everything else is polish.

---


---
# feedback by current Qwen3.8-Q3S-slow without any prompt
___

## Critical issues

**1. Invalid / omitted mode falls through silently.**
If `args.mode` is missing, it defaults to `"MOVE"` (the most destructive mode). If it's a string the enum rejects but somehow reaches `execute` (e.g. `"FLY"`), none of the mode branches match and it hits the final `return "Operation completed."` — a silent no-op with a lie. Guard at the top:

```ts
const VALID = new Set(["MOVE","COPY","APPEND","CUT","PASTE","REPLACE","WRITE","PEEK","MAP","DELETE","CLEAR"]);
if (!VALID.has(mode)) return `Error: unknown mode '${args.mode}'. Valid: ${[...VALID].join(", ")}.`;
```

And drop the `|| "MOVE"` default — a missing mode should be an error, not a silent cut.

**2. CRLF → LF normalization on every write.**
You read with `split(/\r?\n/)` and write with `join("\n")`. A CRLF file becomes LF after the first successful operation. This is a silent mutation the agent doesn't know about. Either preserve the original line-ending style (detect on read, use for write) or document it prominently in the description so the agent knows.

**3. `last_write` buffer can be clobbered by user intent.**
If an agent names its own buffer `last_write`, every subsequent WRITE overwrites it. There's no protection or warning. At minimum, add to the description: "Do not name a buffer `last_write` — it is reserved and overwritten by every WRITE." Or reject `bufferName: "last_write"` with a teaching error.

## Design concerns for agentic use

**4. All-digit markers are unexpressible.**
`/^\d+$/` → line number. A line that genuinely starts with digits (e.g. `192.168.1.1`, `100% confident`, `42 is the answer`) cannot be used as a marker. There's no escape hatch (`"@"` prefix, explicit `type: "marker"` flag, etc.). The agent will hit this and be stuck. Options:
- Document that such lines must be addressed by line number or a longer prefix that includes a non-digit.
- Add a syntax like `"marker:42"` to force marker interpretation.

**5. In-memory buffers are a black box.**
The agent has no way to verify buffer contents beyond PEEK (40-char cap, 3+3 lines) and MAP (head/tail + headings). After 5+ COPY/APPEND operations, the agent is reasoning about state it can't fully observe. If the session restarts, all buffers vanish. Consider:
- Allowing `PEEK` with `from`/`count` to show up to 25 full lines (you have this — make sure the description makes it prominent as the "escape hatch").
- A `DUMP` mode that writes a buffer to a file (or just returns the full content, accepting the context cost) for when the agent genuinely needs to verify.
- Documenting that buffers are session-scoped and lost on restart, so the agent should PASTE to a file if the content matters across sessions.

**6. Description length vs. effective attention.**
The description is ~900 words of dense specification. An LLM has to parse all of it on every tool call to choose a mode. The critical decision points (which mode, which ref form, which file is source vs. destination) get buried. Suggest restructuring the top of the description as a short decision table:

```
| Goal | Mode | Input |
| Move block file→file | MOVE | srcFile, dstFile, start..end [, target] |
| Extract to buffer | COPY | srcFile + (markers \| refs \| text) |
| Append to buffer | APPEND | same as COPY |
| Buffer→file (insert) | PASTE | dstFile [, target] |
| Swap span with buffer | REPLACE | dstFile, start..end |
| Swap span with text | WRITE | dstFile, text, start..end \| regions |
| Cut + delete | CUT / DELETE | srcFile, start..end |
| Inspect buffer | PEEK / MAP | [from, count] |
| Empty buffer | CLEAR | — |
```

Then the detailed mode descriptions follow. The agent reads the table first; the details are there when needed.

**7. Schema is all-optional; requirements live in prose.**
Every arg is `.optional()`. The agent has to read the description to know that `dstFile` is required for PASTE, `text` for WRITE, etc. If the SDK supports it, a conditional/dependent schema would help. If not, at least in the schema descriptions, lead with "REQUIRED for X, Y, Z" so the pattern is scannable.

## Correctness / edge cases

**8. `countLines` vs. `matchAnchorLines` split mismatch.**
`countLines` splits on `/\r?\n/`; `matchAnchorLines` splits on `"\n"` and strips `\r`. They agree on line count, but the index arithmetic in file operations uses `split(/\r?\n/)` while anchor matching uses `split("\n")`. For a file with `\r\n`, the line arrays have the same elements, so indices align. But it's fragile — if someone changes one split to the other, indices break. Use one canonical split everywhere (recommend `/\r?\n/` with no per-line `\r` stripping, since the split already removes it).

**9. Empty `text` in COPY/APPEND.**
`COPY` with `text: ""` produces `textToLines("")` → `[]`, setting the buffer to empty. This is equivalent to CLEAR but via a different path with a different feedback message ("Copied 0 lines"). Either reject empty text in COPY/APPEND with a teaching error, or let it through but make the feedback unambiguous.

**10. `path.dirname` for bare filenames.**
If `dstFile` is `"test.txt"`, `path.resolve(cwd, "test.txt")` gives an absolute path, and `path.dirname` gives `cwd` — fine. But if `cwd` itself doesn't exist (fresh clone, wrong directory), `fs.mkdirSync(path.dirname(dstPath), {recursive:true})` creates it. This is probably fine, but worth noting.

**11. Sandbox `toLowerCase()` is Windows-specific.**
On Linux/macOS, `/Working/Dir/` and `/working/dir/` are different. The `toLowerCase()` comparison would incorrectly allow access. If this is Windows-only (your doc says "Windows temp directory"), fine — but note it. If you want cross-platform, compare without `toLowerCase()` on POSIX.

**12. No file-size guard.**
`readFileSync` on a multi-GB file will OOM. For agentic use on code files this is unlikely, but a cheap guard (e.g. reject > 10 MB with a clear error) prevents a hard crash.

## What's well done

- **Unified anchor rule in one function.** Swapping matching behavior is a one-place change. Correct.
- **Teaching error messages.** The "closest lines" hint, match count + line numbers, out-of-range with file line count — these let the agent self-correct without guessing. This is the single most valuable design decision for agentic use.
- **All-validation-before-write.** No partial state on rejection. Critical for agent recovery.
- **Consistent one-line feedback per op.** Easy for the agent to parse and verify.
- **The "WHEN-NOT" boundary with `edit`.** Explicitly tells the agent when NOT to use this tool. Prevents the most common misuse.
- **`last_write` auto-buffer.** Clever poka-yoke: if an anchor fails on the next attempt, the text is already available without re-typing.
- **Bounded PEEK/MAP.** Protects the context window from a large buffer dump.
- **The exactly-one contract** for anchors. No ambiguity, no "first match" surprises.

## Summary of priority fixes

| Priority | Item |
|----------|------|
| P0 | Validate mode; reject unknown / missing mode |
| P0 | Decide on line-ending preservation vs. normalization and document |
| P1 | Protect `last_write` from user-named collision |
| P1 | Escape hatch for all-digit markers (or explicit documentation) |
| P2 | Restructure description with a decision table up front |
| P2 | Unify the line-split regex across all functions |
| P3 | File-size guard; empty-text handling; sandbox portability note |


---
# feedback of a "recovered" model version of Qwen3.8-Q3S (chat template sharp)
---

## Critical bug

**MOVE with `srcFile === dstFile` reads the destination *after* cutting the source.** The code writes `srcPath` (the cut) in section 3, then reads `dstPath` in section 4. If they're the same file, the target resolution runs against the post-cut file, so `targetMarker` line numbers are wrong and `insertIdx` may exceed the file's new length (silent mis-insertion or an unhandled `splice` at an invalid index). Either guard `srcFile !== dstFile` with an explicit error, or read `dstText` before the cut (resolve the target against the pre-call state, consistent with every other mode).

## Durability: MOVE cuts before it pastes

Section 3 writes the source (removing the block), then section 4 reads and writes the destination. A crash between the two writes **loses the block** — it's gone from disk and was only in `extractedBlock` in memory. Reversing the order (insert into dst first, then cut src) makes the failure mode *duplication* instead of *loss*, which is trivially recoverable by the agent (read both files, `DELETE` the duplicate from src).

## Agentic UX

- **Description length.** The `description` string is roughly 1,000 words / ~700 tokens. For a tool the agent sees on every call, that's a heavy fixed cost. The mode-by-mode detail (Part A/B/C/F references, plan25, S1/S2/S3) is for maintainers, not for the model. A leaner agent-facing description could be ~400 words: one-line "when to use / when not," the 11 modes as one-line each, the ref/anchor rules, and the error contract. Move the implementation sketch, gotchas, and test pointers to the `.md` only (which you already do — just trim the inline `description` to match that level of abstraction).

- **`last_write` is a hidden side-channel.** Every `WRITE` silently overwrites `clipboardBuffers["last_write"]`. If the agent `COPY`s into a buffer named `last_write` and later does a `WRITE`, the COPY'd content is gone with no warning. Two options: (a) reserve the name and reject `bufferName: "last_write"` on explicit COPY/APPEND/CUT with a teaching error, or (b) rename the auto-buffer to something less likely to collide (`__write_cache`, `_bt_last_write`).

- **No preview for destructive ops.** PEEK/MAP work on buffers only. There's no "show me what lines 3–7 of `foo.ts` look like" without committing to a CUT or MOVE. An agent that wants to verify a span before cutting it has to `COPY` (mutates a buffer), `PEEK`, then `PASTE`+`CUT` — four calls where one would do. A lightweight `READ_LINES(file, start, end)` mode (or extending PEEK to accept a file + span) would close this gap and is cheap to add.

- **Mode count.** 11 modes is a lot of decision surface. The strongest consolidation candidate: **DELETE ≡ WRITE with empty text** (or `text: ""` into a span). Removing DELETE reduces the enum and the description. MOVE is also a derived op (CUT + PASTE), but it's high-frequency enough to keep as a convenience.

## Error-message precision

The end-before-start error reads:

> `Error: End marker 'X' not found after start marker in file.ts.`

The marker *was* found — it's just at a line before the start. An agent seeing "not found" will try a *different* marker instead of swapping start/end. Suggest:

> `Error: End marker 'X' resolved to line 4, which is before start marker line 7 in file.ts. Swap them or widen the start.`

This matches the teaching taxonomy's spirit (show the resolved line numbers) without the misleading "not found" verb.

## Smaller items

- **`async` with no `await`.** The `execute` callback is `async` but every operation is synchronous. Drop the `async` to avoid implying there's an I/O boundary the agent needs to reason about (and to let the host skip the microtask hop).

- **"Windows temp directory" in the description.** The code handles it generically (`process.env.TEMP ?? process.env.TMP`), but on macOS/Linux neither is set, so the only allowed root is `cwd`. The description should say "the working directory and the system temp directory" so a non-Windows agent isn't confused.

- **`countLines` vs. `matchAnchorLines` split strategies.** `countLines` splits on `/\r?\n/`; `matchAnchorLines` splits on `"\n"` then strips a trailing `\r`. They produce the same line count for CRLF and LF files, so there's no bug, but the divergence is a maintenance trap. Route both through one `splitLines(fileText): string[]` helper.

- **Catch-all error format.** The `catch` block returns `block_transfer failed: ${err.message}`. If `fs.writeFileSync` hits EACCES or ENOSPC, the agent gets an opaque one-liner. Consider `return \`Error: write to '${dstFile}' failed: ${err.message}\`` so the agent can at least identify which path failed.

- **Adjacent-span overlap in WRITE `regions`.** `asc[i].start <= asc[i - 1].end` correctly rejects spans that share a line. But two spans that are *adjacent* (A ends at 5, B starts at 6) are allowed. After applying B first (highest line), A's splice shifts nothing (it's lower). This is correct — just confirm the description's "overlapping" wording matches the `<=` boundary so the agent knows "sharing a line" is the threshold.

- **Sandbox allows writing into `path.dirname(dstPath)` via `mkdirSync({recursive: true})`.** This is fine within the sandbox, but the `mkdirSync` happens *after* all validation. If the path is deep and a parent component is a symlink escaping the sandbox, the `mkdirSync` could create directories outside the sandbox before `writeFileSync` fails. Unlikely in practice, but a `fs.realpath` check on the resolved path before the `mkdirSync` would close it.

## What's working well

- The unified anchor rule through `resolveAnchor` / `matchAnchorLines` is the right architecture. One matching function, one disambiguation rule, one error taxonomy. The "swap the behavior in one place" comment is the exact right invariant.
- Teaching errors (closest-line hints, match counts with line numbers, out-of-range with total) are genuinely good for an agent — they encode the fix, not just the failure.
- "All checks before any buffer change / file write" is correctly implemented in every mode. The agent can trust that a rejected call leaves no partial state.
- The `WHEN-NOT` section (use `edit` for mid-line substrings, `write` for whole-file rewrites) is the single most valuable piece of the description for preventing the agent from reaching for the wrong tool.
- The `refs`-list form for assembling scattered lines into a buffer is a genuinely useful capability that's hard to do with the `edit` tool.

The two items I'd fix before the next agent session hits a real file are the **MOVE src===dst read-order bug** and the **cut-before-paste durability order**. Everything else is polish.

---
# feedback of a "recovered" model version of Qwen3.8-Q3S (chat template normal)
---
# Feedback: `block_transfer` Tool

## Overall Impression

This is a well-architected tool with a clear central design principle (the unified anchor rule) and a strong invariant (all checks before any write). The teaching error taxonomy, the ref disambiguation strategy, and the buffer lifecycle are all thoughtful. For an agentic tool that does line-level file surgery, this is a solid foundation. Below are issues and suggestions organized by severity.

---

## Correctness / Data-Integrity Issues

### 1. CRLF → LF silent normalization (High)

The split is `/\r?\n/` (which strips `\r`) and the join is always `"\n"`. A CRLF file read in and written back becomes LF. This is a silent, irreversible data modification to the file's line-ending convention. For a tool an agent uses repeatedly on a codebase, this will:

- Break `.gitattributes` expectations (`* text=auto` or `*.cs text eol=crlf`)
- Cause spurious diffs on Windows teams
- Corrupt files where line endings are semantically meaningful (e.g., some legacy formats, or files shared with non-git tools)

**Suggestion:** Detect the dominant line ending on read (or the per-file setting) and preserve it on write. A minimal approach:

```ts
// After reading:
const usesCRLF = /\r\n/.test(raw);
// On write:
fs.writeFileSync(dstPath, lines.join(usesCRLF ? "\r\n" : "\n"), "utf-8");
```

At minimum, document this behavior prominently in the description and the `.md` so the agent (or user) is not surprised. Right now it's invisible.

### 2. MOVE: source is mutated before destination is confirmed writable (Medium-High)

The execution order for MOVE is:

```
read src → resolve anchors → write src (cut) → read/write dst (paste)
```

If the `fs.writeFileSync(dstPath, ...)` throws (disk full, permission denied, path resolved to a read-only location that passed `sandboxCheck` but fails at the OS level), the source file has already been truncated. The `catch` block returns a generic error string, but the damage is done. The "no partial writes on rejection" invariant is broken by runtime exceptions (as opposed to validation rejections).

**Suggestion:** For MOVE specifically, read both files, perform both mutations in memory, then write dst first (or write both in a transactional-ish pattern). If dst write fails, src is untouched. Alternatively, write a temp file for dst and rename (atomic on most filesystems):

```ts
const tmp = dstPath + ".block_transfer_tmp";
fs.writeFileSync(tmp, dstLines.join("\n"), "utf-8");
fs.renameSync(tmp, dstPath); // atomic
fs.writeFileSync(srcPath, remainingLines.join("\n"), "utf-8");
```

### 3. `countLines` vs. the raw split arrays (Low, but worth noting)

`countLines("a\nb\n")` returns 2, but `srcRaw.split(/\r?\n/)` gives `["a", "b", ""]` (length 3). The range check in `resolveRef` uses `countLines`, so line number 3 is correctly rejected. But if any code path uses `srcLines.length` directly instead of `countLines`, it could allow access to the trailing empty string. I checked and it's consistent — `countLines` is used for all range checks. Just flagging that this is a fragile invariant; a single refactor that swaps to `srcLines.length` would introduce an off-by-one.

**Suggestion:** Add a comment at the split sites: `// NOTE: trailing \n produces an empty tail element; use countLines() for the conceptual line count, never .length`.

---

## Agentic UX (How the LLM Agent Experiences the Tool)

### 4. Description length and information density

The description is ~4,200 characters. For a tool with 11 modes and 3 input forms, this is defensible, but the agent must parse all of it in context on every tool-consideration cycle. Specifically:

- The **RETURN LINES** section (~700 chars) describes the *output format*. The agent doesn't need to predict the exact format to *use* the tool — it just needs to read the result. This section could be halved or moved to a "see the result" note.
- The **EDGE** section re-describes error behaviors that the error messages themselves convey. "A non-unique anchor: widen the prefix, do not guess" is the one genuinely useful instruction there; the rest is redundant with the actual error strings.
- The schema field descriptions repeat the "all-digit ref = line number" rule in **six** separate `.describe()` calls. This is belt-and-suspenders, but it also inflates the schema the model sees. One authoritative statement in the top-level description + a short "see REFS" pointer in each field would halve the token cost.

**Suggestion:** Trim the description by ~30–40% by:
- Compressing RETURN LINES to 2–3 lines of format skeleton
- Keeping only the "widen the prefix, do not guess" line from EDGE
- Replacing the repeated ref-disambiguation text in schema with `"See REFS above"` or a 5-word pointer

The model will still see the full schema field types (`union[string, number]`), and the top-level REFS paragraph handles the semantics.

### 5. The `mode` enum `.describe()` duplicates the top-level description

The `mode` field description re-lists all 11 modes with their semantics. This is the second place the model reads the same information. For a 11-value enum, a concise one-liner per value (or just the value + a 5-word verb phrase) in the schema, with the full semantics in the top-level description, would reduce redundancy.

### 6. "When in doubt, use `edit`" routing signal

The WHEN-NOT section correctly directs the agent to `edit` for mid-line substring replacement. This is important and well-placed. One addition I'd suggest: explicitly state that `block_transfer` **cannot** operate on a substring of a line. The current wording says "block_transfer cannot replace a mid-line substring (use 'edit' for that)" which is good, but it's buried in a paragraph. A one-line bold or caps callout at the top of the description would make this routing decision faster for the model:

> **Cannot target a substring within a line. For that, use `edit`.**

### 7. The `last_write` auto-buffer is a hidden side effect

Every WRITE silently overwrites `clipboardBuffers["last_write"]`. The agent has no way to know its previous `last_write` content is gone unless it PEEKed or MAPped it beforehand. The description documents this, but an agent doing a sequence of operations (WRITE file A → COPY from `last_write` → WRITE file B → try to PEEK `last_write`) will get the *second* write's content, not the first. This is by design (poka-yoke), but it's a footgun in multi-step agentic workflows.

**Suggestion:** Consider making the auto-buffer name include a discriminator, or at minimum: include the previous `last_write` line count in the WRITE feedback (e.g., `" - last_write overwritten (was N lines)"`). Or make it opt-out: if the agent explicitly passes `bufferName: "last_write"`, it gets a different message ("You are overwriting the auto last_write buffer"). This is a minor UX polish, not a correctness issue.

---

## Code Quality / Maintainability

### 8. The `execute` function is a 200-line sequential if-chain

Each mode is a block of logic in a single function. It's readable because of the section comments, but:

- It's hard to unit-test individual modes without mocking the entire execute
- Adding a new mode means inserting another block in the middle of the chain
- The shared preamble (mode parsing, buffer key, cwd) is fine, but the per-mode logic could be extracted:

```ts
const handlers: Record<string, (args, ctx) => Promise<string>> = {
  CLEAR: handleClear,
  PEEK: handlePeek,
  MAP: handleMap,
  PASTE: handlePaste,
  REPLACE: handleReplace,
  WRITE: handleWrite,
  COPY: handleCopy,
  APPEND: handleAppend,
  MOVE: handleMove,
  CUT: handleCut,
  DELETE: handleDelete,
};
```

This doesn't change behavior, but makes each mode independently testable and the main function a 10-line dispatch.

### 9. `assemblyFeedback` and `opLine` are near-duplicates

`assemblyFeedback` is called for COPY/APPEND. `opLine` is called for everything else. They produce nearly identical strings with slightly different parameter shapes. `assemblyFeedback` could be expressed as:

```ts
function assemblyFeedback(verb, prep, a, bufferKey, bufferAfter) {
  const srcLabel = a.fileRef === null ? "text" : `'${a.fileRef}'`;
  return opLine(verb, a.lines.length, `${prep} buffer '${bufferKey}' from ${srcLabel}`, a.rangeText, a.lines[0] ?? "", bufferAfter);
}
```

One fewer function to keep in sync.

### 10. The `try/catch` returns a bare string, losing the stack

```ts
catch (err: any) {
  return `block_transfer failed: ${err.message}`;
}
```

For the agent, this is fine (it can't do anything with a stack trace). But for *you* debugging, a silent `console.error` or a structured log line before the return would help. Consider:

```ts
catch (err: any) {
  console.error("[block_transfer]", err);
  return `block_transfer failed: ${err.message}`;
}
```

### 11. Repeated `path.resolve(cwd, args.srcFile)` + `sandboxCheck(cwd, args.srcFile)` pattern

This pair appears 5+ times. A small helper:

```ts
function resolveChecked(cwd: string, givenPath: string, label: string): string | { error: string } {
  const violation = sandboxCheck(cwd, givenPath);
  if (violation) return { error: violation };
  return path.resolve(cwd, givenPath);
}
```

Reduces the "did I forget the sandbox check before the resolve?" class of bugs.

---

## Schema / API Design

### 12. `refPresent` treats `0` as present (then errors)

```ts
function refPresent(ref: Ref | undefined): boolean {
  return ref !== undefined && ref !== null && ref !== "";
}
```

So `startMarker: 0` passes the presence check, then fails in `resolveRef` with "0 is not a 1-based line number." This is correct and well-documented. Just noting that an agent that passes `0` (e.g., from an off-by-one in its own reasoning) gets a teaching error rather than a silent skip, which is the right call.

### 13. The `regions` items use `start`/`end` but the top-level uses `startMarker`/`endMarker`

This naming inconsistency is minor but could confuse the agent. A `regions` item with `{ start: "## Section", end: 42 }` looks structurally different from the top-level `{ startMarker: "## Section", endMarker: 42 }`. Consider either:
- Renaming regions items to `{ startMarker, endMarker }` (consistent), or
- Keeping the shorter names but adding a schema description line: "Same semantics as startMarker/endMarker, shorter name for the nested form."

### 14. PEEK `from`/`count` are `number().int()` only — no string form

Every other ref in the tool accepts `string | number` (for the digit-string disambiguation). PEEK's `from` and `count` are `number().int()` only. This is a minor inconsistency. If the host's constrained decoding can stringify integers, the agent might pass `"5"` as a string and the schema would reject it. Consider making these `union([string, number])` and routing through the same digit-string check, or at minimum documenting that they must be actual numbers (not strings).

---

## Security / Sandbox

### 15. The sandbox uses `path.resolve` which resolves symlinks? No — it doesn't.

`path.resolve` does **not** resolve symlinks. It only normalizes the path string (handles `..`, `.`, absolute/relative). If the working directory contains a symlink pointing outside the sandbox, `sandboxCheck` will pass it (the *string* path is inside cwd), but `fs.readFileSync` will follow the symlink and read the external file.

**Suggestion:** If this is a concern (and it depends on the threat model — a symlink in your own working directory is not obviously adversarial), add an `fs.realpathSync` check:

```ts
const real = fs.realpathSync(resolved);
const allowed = roots.some(root => real === path.resolve(root) || real.startsWith(path.resolve(root) + path.sep));
```

This is defense-in-depth. If the intercept plugin handles this upstream (as the comment notes), it's acceptable to leave as-is, but a comment noting "symlink resolution is the intercept plugin's responsibility" would make the boundary explicit.

### 16. `fs.mkdirSync(path.dirname(dstPath), { recursive: true })` before write

In PASTE and WRITE, if the destination path's parent directory doesn't exist, the tool creates it recursively. This is correct for the "creates the file if absent" behavior. But it means `WRITE` with `dstFile: "a/b/c/d.txt"` will create the entire `a/b/c/` tree. This is probably intended, but worth a one-line note in the description: "Parent directories are created if absent."

---

## Documentation (block_transfer.md)

### 17. Missing from the doc:

- **Line-ending normalization** (CRLF → LF on write) — see issue #1
- **The `last_write` auto-buffer** — mentioned only in the tool's description, not in the doc
- **The "no partial state" invariant scope** — the doc says it applies to validation rejections, but the MOVE exception case (issue #2) is a runtime exception, not a rejection. Worth noting the boundary.
- **The `refs` list selects one line per ref** — the doc says "each ref selects ONE line" in the description, but the implementation sketch in the `.md` doesn't repeat this. An agent reading only the `.md` might assume ranges are possible in the list form.

### 18. The "Gotchas" section could use one more entry:

> - CRLF files are normalized to LF on write. If your project requires CRLF, set this via `.gitattributes` and be aware that block_transfer will not preserve the original line endings.

---

## Minor / Polish

| # | Item | Note |
|---|------|------|
| 19 | `mode` defaults to `"MOVE"` when absent (`args.mode \|\| "MOVE"`) | The schema marks `mode` as required (no `.optional()`). The fallback is dead code unless the host strips required fields. Fine to leave, but the comment should note it's a safety net. |
| 20 | `bufferKey` defaults to `"default"` | Same — the schema has no default, the code provides one. Fine. |
| 21 | The `opLine` echo uses `firstLine.length > 40` but the description says "capped ~40 chars" | The `~` is doing work here. It's exactly 40 + `"..."` = 43 chars. Trivial, but "capped at 40" is more precise. |
| 22 | PEEK default shows "head: ... tail: ..." with `...` as a literal separator | For a 4-line buffer, head = lines 1–3, tail = line 4. The output is `head: 'l1', 'l2', 'l3' ... tail: 'l4'`. The `...` suggests omitted content, but nothing is omitted. For very small buffers (≤6), the `...` is misleading. Consider omitting it when head + tail cover all non-blank lines. |
| 23 | The `notFoundError` "Closest lines" hint uses longest-common-prefix scoring | This is a good heuristic for "what did you probably mean?" For an anchor like `## TODO` where the file has `## To Do` and `## Todays`, the common prefix with `## To Do` is 5 chars (`## To`), which is reasonable. But for very short anchors (2–3 chars), many lines will "qualify" with `common >= 1` and the top-5 will be near-arbitrary. Consider raising the threshold to `common >= 2` for 2-char+ anchors, or weighting by ratio (`common / anchor.length`). |
| 24 | `textToLines("")` returns `[]` | So `COPY` with `text: ""` produces an empty buffer. Then `PASTE` of that buffer fails with "empty buffer." This is correct, but the COPY feedback would be "Copied 0 lines from text into buffer 'X' - buffer: 0 lines." An agent might not realize the empty text was the problem. Consider a specific error: "Error: 'text' is empty; nothing to copy." |

---

## Summary: What I'd Prioritize

1. **Fix CRLF normalization** (#1) — silent data modification, will bite in real use
2. **Fix MOVE write ordering** (#2) — the one case where "no partial writes" is violated
3. **Trim the description** (#4) — 30% shorter with no loss of routing information; the agent will make fewer mistakes if the signal-to-noise ratio improves
4. **Extract mode handlers** (#8) — makes the next iteration (adding modes, fixing edge cases) significantly easier
5. **Add symlink note or `realpathSync`** (#15) — cheap insurance
6. **Update the `.md` doc** (#17–18) — the doc is the agent's "manual"; gaps there mean the agent relies solely on the (long) description

The core design — unified anchor rule, pre-call resolution, teaching errors, named buffers, the ref-disambiguation-by-value strategy — is strong and well-reasoned. The feedback above is mostly about the edges, the agent's parsing cost, and a couple of data-integrity gaps.


---
# feedback of a "recovered" model version of Qwen3.8-Q3S (no chat template)
---
Overall: this is a strong, unusually deliberate agentic tool. The unified anchor rule, the teaching error taxonomy, the named buffers, and the bounded `PEEK`/`MAP` ideas are all good instincts. The main risks are not “it does the wrong edit”; they are **partial state / data-loss risk**, **line-ending / empty-file edge cases**, **sandbox weaknesses**, and a few **agent-ergonomic surprises** that could make an autonomous agent waste turns or damage files.

Below is structured feedback, ordered by priority.

---

## 1. High-priority correctness issues

### 1.1 `MOVE` can lose data if destination validation fails after the source is cut

This is the biggest issue.

Current sequence for `MOVE`:

1. Resolve source span.
2. Extract block from source.
3. **Write source file without the block.**
4. Then read destination.
5. Then resolve `targetMarker`.
6. Then write destination.

That means if `targetMarker` is invalid, non-unique, out of range, or if destination write fails, the source has already been modified and the extracted block exists only in memory. That is a data-loss path.

Also, for `MOVE`, the destination directory may be created before the destination target is validated.

Recommended behavior:

- Read and parse both source and destination.
- Resolve source span against pre-call source.
- Resolve destination insertion point against pre-call destination.
- Validate all refs, sandbox, file types, and target existence before any mutation.
- Reject or specially handle `srcFile === dstFile`.
- Write destination first, then source. Better: write both through atomic temp-file/rename.
- Only create destination directories after all validation.

Example desired order:

```ts
// MOVE
parse src
parse dst
resolve src start/end
resolve dst targetMarker if present
if srcRealPath === dstRealPath -> error
mkdir only after validation
write dst atomically
write src atomically
```

If you cannot make `MOVE` atomic across two files, at least make it fail closed: invalid destination target must not mutate the source.

For the same-file case, I would explicitly reject:

```text
Error: MOVE requires distinct source and destination files.
```

Same-file move is possible to implement, but it is much harder to reason about because insertion shifts lines after the cut. Rejecting is safer and easier to document.

---

### 1.2 `PASTE` creates directories before validating the insertion point

In `PASTE`, the code does:

```ts
if (fs.existsSync(dstPath)) {
  dstText = fs.readFileSync(dstPath, "utf-8");
  dstLines = dstText.split(/\r?\n/);
} else {
  fs.mkdirSync(path.dirname(dstPath), { recursive: true });
}

let insertIdx = dstLines.length;
if (refPresent(args.targetMarker)) {
  const target = resolveRef(...);
  if ("error" in target) return target.error;
}
```

If `targetMarker` is invalid, the destination parent directory may already have been created. That violates your “no partial state on rejection” goal.

Fix:

- Delay `fs.mkdirSync` until after all destination validation.
- Resolve `targetMarker` before creating any directories.
- Do the same for `MOVE`.

---

### 1.3 Line model is inconsistent: trailing newline and empty-file handling

This is subtle but important.

You compute conceptual line count with `countLines()`, which excludes the trailing empty element from `split("\n")`. But many operations still use raw `text.split(/\r?\n/)`, which includes that trailing empty element when the file ends with a newline.

Consequences:

#### Example A: appending to a file ending with a newline

File:

```text
a
b
```

In memory raw split:

```ts
["a", "b", ""]
```

If you `PASTE` without `targetMarker`, current code inserts at `dstLines.length`, which is after the trailing empty string:

```text
a
b

<pasted block>
```

There is an extra blank line.

Expected:

```text
a
b
<pasted block>
```

#### Example B: pasting into an existing empty file

If `dstFile` exists but is empty:

```ts
"".split(/\r?\n/) => [""]
```

Appending at `dstLines.length` inserts after the empty string, producing a leading blank line:

```text

<pasted block>
```

Expected:

```text
<pasted block>
```

Recommended fix: use one canonical parse/serialize helper for all file mutations.

Something like:

```ts
type ParsedFile = {
  lines: string[];
  trailing: boolean;
  lineEnding: "\n" | "\r\n";
};

function parseFile(text: string): ParsedFile {
  const lineEnding: "\r\n" | "\n" = text.includes("\r\n") ? "\r\n" : "\n";

  if (text === "") {
    return { lines: [], trailing: false, lineEnding };
  }

  const trailing = text.endsWith("\n");
  const lines = text.split(/\r?\n/);

  if (trailing && lines.length > 0 && lines[lines.length - 1] === "") {
    lines.pop();
  }

  return { lines, trailing, lineEnding };
}

function serializeFile(parsed: ParsedFile): string {
  if (parsed.lines.length === 0) {
    return "";
  }

  let out = parsed.lines.join(parsed.lineEnding);
  if (parsed.trailing) {
    out += parsed.lineEnding;
  }
  return out;
}
```

Then use `parsed.lines` for all operations:

- anchor matching
- line-number refs
- `PASTE`
- `MOVE`
- `CUT`
- `DELETE`
- `REPLACE`
- `WRITE`

This fixes:

- extra blank line before EOF,
- leading blank line on empty existing file,
- inconsistent line counts,
- easier reasoning about final line,
- CRLF preservation if you choose to keep it.

If you do not want to preserve CRLF, document that clearly: “This tool normalizes line endings to LF.” But preserving the dominant original line ending is nicer for real repo use.

---

### 1.4 CRLF is matched but probably not preserved

Your anchor matching is CRLF-tolerant because it splits on `\n` and strips a trailing `\r`. But all writes use:

```ts
dstLines.join("\n")
```

So a CRLF file becomes LF after editing.

That may be acceptable, but it is a behavior agents will notice.

Options:

1. Preserve original line endings using `lineEnding` from `parseFile()`.
2. Normalize to LF and document it explicitly.
3. Detect dominant line ending and use that.

For agentic editing, preserving CRLF is safer.

---

### 1.5 Sandbox has real bypass / false-allow issues

Your sandbox is a good backstop, but it has several weaknesses.

#### 1.5.1 Lowercasing paths is unsafe on case-sensitive filesystems

Current:

```ts
const resolved = path.resolve(cwd, givenPath).toLowerCase();
const r = path.resolve(root).toLowerCase();
```

On Linux, this can allow paths that are technically outside the working directory.

Example:

```text
cwd = /Foo
path = /foo/secret
```

After lowercasing:

```text
resolved = /foo/secret
root = /foo
```

That path is outside `/Foo`, but it passes.

Fix:

```ts
const normalize = (p: string) =>
  process.platform === "win32" ? p.toLowerCase() : p;
```

Only lowercase on Windows.

#### 1.5.2 `path.resolve()` does not resolve symlinks

A symlink inside the working directory can point outside:

```text
cwd/link -> /etc
cwd/link/passwd
```

`path.resolve(cwd, "link/passwd")` is inside `cwd`, but `fs` follows the symlink and reads/writes outside.

Fix: resolve the deepest existing ancestor with `fs.realpathSync()` and check that the real path is inside an allowed root.

Sketch:

```ts
function isInsideAnyRoot(realPath: string, roots: string[]): boolean {
  return roots.some((root) => {
    const r = path.resolve(root);
    const p = path.resolve(realPath);
    const prefix = r.endsWith(path.sep) ? r : r + path.sep;
    return p === r || p.startsWith(prefix);
  });
}

function assertSandbox(cwd: string, givenPath: string): string | null {
  const abs = path.resolve(cwd, givenPath);

  const roots = [cwd, process.env.TEMP ?? process.env.TMP]
    .filter((r): r is string => typeof r === "string" && r.length > 0)
    .map((r) => path.resolve(r));

  // Check final path if it exists.
  if (fs.existsSync(abs)) {
    const real = fs.realpathSync(abs);
    if (!isInsideAnyRoot(real, roots)) {
      return `Error: '${givenPath}' resolves outside the sandbox via a symlink.`;
    }
  }

  // Check deepest existing ancestor for paths that do not exist yet.
  let current = abs;
  while (true) {
    if (fs.existsSync(current)) {
      const real = fs.realpathSync(current);
      if (!isInsideAnyRoot(real, roots)) {
        return `Error: '${givenPath}' resolves outside the sandbox via a symlink.`;
      }
    }

    const parent = path.dirname(current);
    if (parent === current) break;
    current = parent;
  }

  return null;
}
```

Also fix the root prefix check. The current `r + path.sep` can misbehave for root paths like `/` or `C:\`. Use:

```ts
const prefix = root.endsWith(path.sep) ? root : root + path.sep;
```

#### 1.5.3 File versus directory checks

Currently a directory path can cause a raw `fs` exception:

- `srcFile` is a directory,
- `dstFile` is a directory.

Better to detect and return a teaching error:

```text
Error: 'foo' is a directory; expected a file.
```

For writes, if the destination exists and is a directory, fail before any mutation.

---

### 1.6 `WRITE` says it creates the file if absent, but it effectively cannot

Current `WRITE` requires a span:

- `startMarker` + `endMarker`, or
- `regions`.

If the destination file does not exist:

- marker refs are not found,
- line-number refs are out of range,
- regions fail.

So a successful `WRITE` on an absent file is basically impossible.

Example:

```json
{
  "mode": "WRITE",
  "dstFile": "new.md",
  "startMarker": "# Title",
  "endMarker": "# End",
  "text": "body"
}
```

If `new.md` does not exist, `# Title` is not found. The file is not created.

Same for an existing empty file: there is no resolvable span.

You have a few options:

1. Remove “creates the file if absent” from `WRITE` docs.
2. Add a separate `CREATE` / `OVERWRITE` mode.
3. Allow `WRITE` with no span to create/overwrite, but make that explicit and dangerous.
4. Keep `WRITE` for region replacement only and document: “To create a new file from direct text, use `COPY text -> PASTE`.”

For safety, I would not make bare `WRITE dstFile text` overwrite a file. Better to keep `WRITE` region-only and document that `PASTE` is the creation path.

Also decide what `text: ""` means.

Currently:

- `WRITE` with empty text replaces a span with zero lines.
- `COPY` with empty text replaces buffer with zero lines.
- `APPEND` with empty text is a no-op.

That may be useful, but it is also dangerous. I would either:

- reject empty `text` and tell the agent to use `DELETE` / `CLEAR`, or
- explicitly document that `text: ""` deletes/replaces with nothing.

For blank lines, use a real blank line, e.g. `"\n"`, not `""`.

---

### 1.7 `PEEK` `from` and `count` may not survive opencode’s constrained integer decoding

You already handle all-digit string refs for `startMarker`, `endMarker`, `targetMarker`, `refs`, and `regions`. But `PEEK` does not:

```ts
if (!Number.isInteger(from) || from < 1) {
  return `Error: from ${from} is not a 1-based line number...`;
}
```

If the host stringifies integers, `from: 5` may arrive as `"5"`, and `Number.isInteger("5")` is false.

Fix: parse digit strings for `from` and `count`.

```ts
function parsePositiveInt(value: unknown, label: string): number | null {
  let v = value;

  if (typeof v === "string" && /^\d+$/.test(v)) {
    v = parseInt(v, 10);
  }

  if (!Number.isInteger(v) || v < 1) {
    return null;
