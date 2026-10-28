# Cursor `Read`: an implementation brief

Cursor's `Read` tool, internal id `READ`, as shipped in Cursor 3.27.12 on Windows. Everything here was read out of the installed artifacts on this machine, or executed against code lifted out of them. Where a claim comes only from the parent spec and could not be re-derived, it says so.

The build under study:

- `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js`, 9,814,287 bytes, one line.
- `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, unminified, original identifiers.

The parent spec was written against a 9,674,628-byte bundle. Every offset in it is stale by roughly 15,300 bytes. Both offsets are given below where the claim depends on one.

## The short version

- **The default line-number format is six wide, right aligned, then a bare pipe, then the text.** `     1|const x = 1;` Five leading spaces, a digit, a pipe, and the first character of the line. There is no space after the pipe and there is no tab. The string `cat -n` does not appear anywhere in the shipped bundle, and Cursor's `latest` description does not claim it - the parent spec says it does, and that is wrong. But the Runvil snapshot in `plugin-std\schemas\read.json` still carries `Results are returned using cat -n format, with line numbers starting at 1` plus a per-line truncation promise that Cursor does not implement. Fix both before anything else, because a model told `cat -n` and shown a pipe will mis-transcribe the prefix into edit anchors.
- **The 100,000-character cap is a pure string-length test on a JavaScript string, applied after windowing, and it errors rather than truncating.** `exceededLimit` is a *success* record with empty content, rendered with `isError: true`. So a two-line window into a 2 GB file succeeds, and a full read of a 100,001-character file fails with nothing. A Rust port that uses `str::len()` counts bytes and fires the cap at a third of the file size for CJK text.
- **A read has two independent truncation mechanisms and one silent one.** The cap errors. The omitted-line notices explain. The service's 8 MiB `substring` cut is silent and its `truncated` flag never reaches the model, because the tool's result proto does not carry it.
- **The window arithmetic has a deliberate asymmetry.** An offset past the last line throws `Offset 500 is beyond file length (10 lines)`, but `offset: 10` on a 10-line file is fine and returns line 10, because the test is `startIndex >= totalLines` after the one-based conversion. A negative offset larger than the file clamps to zero and reads forward instead of throwing, because the clamp runs before the bounds check. `offset: -1` is the last line, and a negative offset with no `limit` reads exactly `|offset|` lines.
- **`offset: 0` is accepted and silently becomes 1, while `limit: 0` is rejected.** The refine explicitly permits zero and a `.transform` rewrites it. The refine message `Offset must be >= 1.` therefore overstates the rule it enforces.
- **The line counter is newlines plus one, so `""` is 1 line and a trailing newline adds a numbered blank line.** But an empty file never reaches the counter: the tool short-circuits and passes `totalLines` as a literal `0`, so emptiness is only ever `totalLines === 0 && content === ""`. And the empty check is `content === ""`, character for character, so a one-byte file holding a single newline is *not* empty: it is 2 lines and renders as two numbered blank lines.
- **Line numbers are optional at the protocol level and off by default when the argument exists.** The model can pass `include_line_numbers`, and the field only exists when the `enableReadFileLineNumbersArg` flag is on. With the flag on, omitting the argument means no line numbers at all.
- **CRLF normalization is decided by a 5% vote.** A file is treated as CRLF only when at least 5% of its newline pairs are CRLF. Below that, `\r` survives into the model's text, so a mostly-LF file with a few Windows lines shows literal carriage returns. A consistent CRLF file is shown as LF.
- **Binary rejection is an extension list checked before any I/O, and the list is deliberately incomplete.** `.zip .tar .gz .exe .dll .so .dylib .bin .mp4 .webm .mov .avi .mkv .wmv .flv .m4v`. `.pdf`, `.png`, `.jpg`, `.gif`, `.webp`, `.ipynb` are absent because they take other branches. Content-based binary detection is separate and lives in the service: a NUL byte in the first 8192 bytes, or more than 5% control bytes other than tab, LF, and CR.
- **The path is normalized by exactly one regex, and it repairs a typo.** `lke` rewrites `~/.cursor/worktries/...` to `~/.cursor/worktrees/...`. That is the whole of it. No `~` expansion, no `resolve`, no case folding. The real resolution happens in the capability service.
- **The per-file lock is keyed on the raw argument string, before normalization.** `./foo.ts` and `C:\work\foo.ts` do not serialize against each other even though they are one file. The same string-keyed mistake shows up in `readPaths`, which stores the service-returned path, so read-before-write bookkeeping is spelling sensitive too. Runvil already does this right: `Access::gate` canonicalizes before any claim is registered, so the claim key is the canonical path.
- **The retry-on-miss releases the lock before waiting and re-acquires it after.** Holding it while waiting for a writer that needs the same key would deadlock, so the order is release, wait up to 120,000 ms for earlier mutating calls, re-acquire, retry exactly once. Getting this order wrong hangs instead of erroring.
- **The file is read whole before windowing, and Runvil is one method away from not doing that.** There is no seek in Cursor. `VfsAccess::read_range` exists on Runvil's backend trait with a real `seek` override in `RealBackend` and explicit delegation through all three routing layers, but `Access` does not expose it, so a Pack can only `read` whole files. Adding `Access::read_range` is the highest-value gap in the crate for this tool, because it also makes the 8192-byte binary sniff and the 2 GiB pre-check cheap.
- **`pi_read` is a second, simpler surface in the same bundle and it is a better shape for a fresh implementation.** Three parameters, byte-based caps (2000 lines and 50 KiB), no line numbers, and explicit continuation trailers telling the model the next `offset` to use. It is not the right *semantics* for Runvil because line-numbered output is what the edit tools anchor against, but its parameter set and its trailers are worth copying.

## Decisions you have to make

### 1. Do reads carry line numbers by default?

**Options.**

- **Always on, no argument.** Every successful text read is prefixed. Costs about 8 bytes per line of context, and any line the model quotes back has to be de-prefixed before it can be used as an `old_string`.
- **Always on, with an `include_line_numbers` escape.** Cursor's shape when the `enableReadFileLineNumbersArg` flag is on. Costs a fifth parameter that most models will ignore, and creates a silent failure mode where an omitted argument means no numbers.
- **Always off.** The `pi_read` shape. Cheapest, and the trailers do the navigation work. But `StrReplace` in Cursor's design is *defined* against line-numbered read output: the whole reason for a six-wide prefix is that the model can cite `line 41` and the edit path can check it.

**Cost of getting it wrong.** If reads carry no numbers and edits are anchored by text only, then a model that needs to say "line 41" has to count. If reads carry numbers and nothing consumes them, you have paid the tokens for no benefit.

**Recommendation: always on, no argument, default style.** Confidence high, because the prefix is the only reason the format exists and Cursor's own default branch emits it with no model cooperation; the optional argument is a feature-flag experiment, not the design.

### 2. Which width and separator?

**Options.**

- **Six wide, right aligned, pipe, no space.** Cursor's default. Matches what a large population of models has seen in Cursor transcripts.
- **Six wide, tab.** Zed's choice, and real `cat -n`. Parses unambiguously because a tab cannot appear in a line number. Costs alignment when the content is tab-indented.
- **Unpadded, tab.** Claude Code's choice. Cheapest per line and matches Claude's own trained format, but the prefix width moves with the line number so columns do not line up.
- **`L123:`.** Cursor's GPT-5 style. Only worth having if you actually route GPT-5.

**Cost of getting it wrong.** A model that has been told `cat -n` and shown a pipe will produce anchors with tabs in them, and those anchors fail to match. This is a silent, expensive failure because it looks like a matching bug in the edit tool.

**Recommendation: six wide, pipe, no space, and fix the description to say so.** Confidence high, because it is the most-seen format and the description fix removes the mismatch that causes the failure.

### 3. Cap on characters, or on bytes?

**Options.**

- **Bytes.** Natural in Rust, one comparison, no allocation. But it diverges from Cursor on exactly the files where the cap matters most: 100,000 CJK characters is 300,000 bytes, so a file Cursor reads fine is refused by a byte-counting port.
- **Unicode scalar values.** `chars().count()`. Honest about "characters" in the human sense. Diverges from Cursor on astral-plane text: 100,000 emoji is 100,000 scalars but 200,000 UTF-16 code units, so Cursor refuses and a scalar-counting port accepts.
- **UTF-16 code units.** Exactly Cursor. In Rust that is `s.encode_utf16().count()`, which allocates or iterates twice.
- **Lines, plus a byte cap.** The Copilot and pi shape. Predictable for the model, does not depend on encoding at all.

**Cost of getting it wrong.** Whatever you pick, a model that has learned "Read gives up at 100k" from Cursor will be surprised once. The question is whether it is surprised on ASCII only (cheap) or on non-ASCII too (expensive, because the model cannot see why).

**Recommendation: cap on bytes, and put the number high enough that it never fires in normal use.** Confidence medium. The exact-Cursor answer is UTF-16 code units, but that unit is an artifact of JavaScript and you would be carrying it forever for a compatibility you do not need. If you want the divergence to be invisible, publish the cap in the description as bytes.

### 4. Does the cap error, or truncate?

**Options.**

- **Error with empty content, as Cursor does.** The model gets a two-line message naming the size in bytes and telling it to use `offset` and `limit`. Costs a full round trip. Buys an unambiguous signal: there is no way to mistake a truncated read for a complete one.
- **Truncate and mark, as Copilot and `pi_read` do.** One round trip instead of two, and the trailer tells the model exactly how to continue. Costs the guarantee: a model that ignores the trailer believes it saw the whole file.
- **Truncate silently.** Cheapest and worst. Never do this.

**Cost of getting it wrong.** The failure mode of erroring is wasted turns on large files. The failure mode of silent truncation is an edit made against a partial view, which corrupts a file.

**Recommendation: truncate with an explicit, actionable trailer, never silently.** Confidence medium-high, because the trailer costs a few tokens and removes the entire class of "the model thought it saw everything" bugs, and because `pi_read`'s trailer wording is already good enough to copy.

### 5. Do you reproduce Cursor's negative offset and its asymmetry?

**Options.**

- **Yes, both.** `-1` is the last line, a negative offset with no limit reads `abs(offset)` lines, and an over-large negative offset clamps to zero instead of throwing. Matches Cursor exactly, including the asymmetry with the positive case.
- **Yes to negative offsets, but throw when the magnitude exceeds the file.** Symmetric and easier to explain. Diverges from Cursor in the one case where a model asks for the last 100 lines of a 10-line file, which is a reasonable thing to ask and where Cursor's answer (the whole file) is arguably better than an error.
- **No negative offsets.** Simpler, and `offset: -1` becomes a validation error the model can learn from. Loses the "tail of the log" pattern entirely, which is a real use.

**Cost of getting it wrong.** The asymmetry is the kind of thing that gets "fixed" by a later contributor who reads it as a bug, and then a model's working habit breaks.

**Recommendation: keep negative offsets, drop the asymmetry, document it.** Confidence medium, because the asymmetry has no defensible rationale and a symmetric rule is one sentence in the description.

### 6. Does the lock key on the argument or the resolved path?

**Options.**

- **Raw argument string, as Cursor does.** Zero cost, and it works for the case the lock exists for: two calls in one turn that name the same file the same way. Fails whenever a model spells a path two ways, which it does constantly.
- **Resolved canonical path.** Closes the spelling gap. Costs a resolution pass before the lock, which means the lock protects a path that the read might not use if resolution is not stable, and costs an extra syscall on a path that may not exist yet.
- **Canonical path when it resolves, raw string when it does not.** Best of both, at the cost of two keys per call and a rule about which one the release uses.

**Cost of getting it wrong.** Raw-string keying leaves a real race open between a Read and a Write of the same file spelled differently. Canonical keying costs one resolution per call and can be wrong across symlink changes.

**Recommendation: canonical where it resolves, raw otherwise.** Confidence high, because the spelling race is the one that actually bites and the fallback keeps not-yet-existing paths lockable.

**Note for Runvil specifically: you already have the better answer.** `Access::gate` canonicalizes at receipt and only then checks policy, and `claim_key` case-folds below a case-folding mount, so `read` and `write` on the same file share a claim key regardless of spelling. Do not reimplement Cursor's raw-string keying on top of it. The only thing worth adding is a test that proves two spellings collide, because that property is now load-bearing and is invisible from the call site.

### 7. Do you reproduce the 5% CRLF vote?

**Options.**

- **Yes.** Matches Cursor, including the odd case where 4% CRLF means the model sees carriage returns.
- **Normalize unconditionally.** Simpler, and the model never sees `\r`. Diverges on mixed files, which are common in repositories that have been through a Windows editor once.
- **Normalize per line, stripping a trailing `\r` from every line.** This is what Claude Code's renderer does and what Cursor's streaming path does when the vote passes. It is unconditional at the line level, so it is simpler than the vote and produces the same result for the 96%-CRLF file.

**Cost of getting it wrong.** If you normalize unconditionally and an edit tool round-trips the content, you rewrite a mixed-ending file into an LF file. Cursor avoids that because its Write does its own read and its own merge.

**Recommendation: strip a trailing `\r` per line at format time, unconditionally, and never claim the content is byte-faithful.** Confidence high, because it removes a whole class of confusing output and costs nothing that the tool was preserving anyway.

### 8. Do you implement `pi_read` alongside `Read`, or one tool?

**Options.**

- **One tool, `Read`, with line numbers and continuation trailers.** Simplest surface for the model.
- **Two tools.** Cursor ships both, which suggests the pi family exists for a different client (a Pi-compatible harness) rather than for the same models. Two tools with overlapping semantics is a good way to get the model to pick the wrong one.
- **One tool, but take `pi_read`'s parameter shape and trailers.** Three parameters, byte caps, no per-model branching, plus Cursor's line numbers.

**Cost of getting it wrong.** Shipping both means the description has to explain the difference, and every description sentence you spend on that is a sentence not spent on the tool.

**Recommendation: one tool, `Read`, with `pi_read`'s trailers.** Confidence high. The pi family's value to you is as evidence of what a second, simpler design chose, not as a second tool.

### 9. Do you paginate by line or by byte, and does the tool read the whole file first?

**Options.**

- **Lines only, whole-file read.** Exactly Cursor. The tool reads everything, decodes, splits, and slices. Simple and honest, and it means a windowed read of a 1 GB file costs a 1 GB read.
- **Lines, with a byte-range read underneath.** Add `Access::read_range` and use it for the sniff, the size pre-check, and - carefully - for the window. Costs one method on the crate and a byte-to-line index, which the tool does not have because lines are defined after decoding. So the honest version still reads the whole file for a *text* window and uses the range only for the prefix checks.
- **Bytes, `pi_read` and Copilot style.** `offset` and `limit` as byte offsets, then decode and split. Breaks line-oriented paging entirely; a caller cannot ask for "line 41".

**Cost of getting it wrong.** Reading the whole file for a two-line window is the difference between a 40 ms tool call and a 4 second one on a large file, and a hostile repository can force it. But a byte-range window over text means you have to find line boundaries by scanning bytes, and the first and last line of a range need care because a range can start and end mid-line and mid-character.

**Recommendation: line pagination, and add `Access::read_range` for the prefix work only.** Confidence high on the first half, medium on the second. Getting `read_range` exposed buys the cheap binary sniff and the cheap size check immediately, and leaves the harder byte-to-line question open for later without blocking anything.

### 10. Do you keep the retry-on-miss at all?

**Options.**

- **Keep it, with the lock dance.** Matches Cursor, closes the race where a model reads a file a sibling call is about to create.
- **Keep the retry, drop the lock dance.** Take a fresh `Access::read` in the retry, since the claim is per call anyway. Same behavior, none of the release-and-re-acquire complexity.
- **Drop it.** A read of a file that does not exist yet is a model error, and the model can retry. Costs a turn in the case where a `Write` and a `Read` of the same new file land in one response.

**Cost of getting it wrong.** Dropping it produces a class of transcript where a model writes a file, reads it, gets `File not found`, and then either retries (fine, one wasted turn) or concludes the write failed (bad, cascading confusion).

**Recommendation: keep the retry, drop the lock dance.** Confidence high. The retry is cheap insurance against a real model behavior, and Runvil's per-call claim makes the deadlock trap disappear rather than requiring careful ordering to avoid.

## The contract

### Tool name, per branch

The name is chosen by a switch on the prompt version string. Internal id is always `READ`.

| Version tag | Model-visible name | Path field |
|---|---|---|
| `dsv3-1018` | `read_file` | `target_file` |
| `cursor-0226` | `Read` | `path` |
| `dsv3-1205` | `Read` | `path` |
| `latest` | `Read` | `path` |
| `gpt5-codex` | `ReadFile` | `path` |
| `codex-cloud` | `ReadFile` | `path` |
| `haiku` | `Read` | `path` |
| minimal harness | `ViewImage` | `path` (image only) |
| anything else | throws `Unhandled version: <v>` | - |

Verbatim, from `main.js` at 4349, in the `g` binding beside the registration:

```js
const g=a?"ViewImage":function(e){switch(e){case"dsv3-1018":return"read_file";case"gpt5-codex":case"codex-cloud":return"ReadFile";case"cursor-0226":case"dsv3-1205":case"latest":case"haiku":return"Read";default:throw new Error(`Unhandled version: ${e}`)}}(r)
```

Note the spec's own table at line 638 says `read_file` for `dsv3-1018`, `ReadFile` for `cursor-0226`, and `Read` for the rest. The build says `ReadFile` is `gpt5-codex` and `codex-cloud`, and `cursor-0226` is `Read`. The build wins.

### Parameter schema, verbatim

The default branch, shared by `dsv3-1205`, `latest`, `gpt5-codex`, and `codex-cloud`, built by `TTe` (daemon: `createParametersSchemaLatest`):

```js
path:   z.string().describe("The absolute path of the file to read.")
offset: optional int  .describe("The line number to start reading from. Only provide if the file is too large to read at once.")
limit:  optional int  .describe("The number of lines to read. Only provide if the file is too large to read at once.")
```

When the `enableReadToolNegativeOffset` flag is on, the `offset` description becomes:

```
The line number to start reading from. Positive values are 1-indexed from the start of the file. Negative values count backwards from the end (e.g. -1 is the last line). Only provide if the file is too large to read at once.
```

Both are gated through `kve` (daemon: `lenientNumber`), which coerces a stringified number to a number before the refine runs, so `"12"` becomes `12` and `"abc"` stays a string and fails the integer check.

`dsv3-1018` differs in two ways. Its path field is `target_file`, and its description is the relative-path one:

```
The path of the file to read. You can use either a relative path in the workspace or an absolute path. If an absolute path is provided, it will be preserved as is.
```

`haiku` replaces `offset` and `limit` with one field:

```js
line_range: z.array(z.number().int()).min(2).max(2)
  .refine(([start, end]) => start >= 1 && end >= start,
          "line_range must be [start_line, end_line] with 1 <= start_line <= end_line.")
  .optional()
  .describe("Optional. A two-element array [start_line, end_line] specifying the range of lines to read (1-indexed, inclusive). Example: [100, 150] reads lines 100 through 150.")
```

`include_line_numbers` exists only when the harness option is on:

```js
include_line_numbers: z.boolean().optional().describe(`Whether to include line numbers in the output.${dense} Prefer using this only when needed, e.g. for citing codeblocks to the user. Defaults to false.`)
```

where `${dense}` is the empty string in sparse mode and otherwise:

```
 Lines are numbered starting at 1, using the format LINE_NUMBER|LINE_CONTENT.
```

A `machineId: string` field is appended when the turn has registered machines, described `Registered machine identifier.` Its value is a plain optional string under one variant and an enum of the registered ids otherwise.

### Validation quirks, exactly

| Input | Result |
|---|---|
| `offset: 0` | accepted, rewritten to `1` by a `.transform` |
| `offset: -1` | accepted when negative offsets are enabled; `0` and `-1` are both allowed by the refine |
| `offset: -0` | `-0 === 0`, so accepted and rewritten to `1` |
| `offset: 1e20` | accepted by zod as an integer, then clamped to `2147483647` before use |
| `limit: 0` | rejected: `Limit must be >= 1.` |
| `limit: -5` | rejected with the same message |
| `offset: "12"` | coerced to `12` |
| `offset: "abc"` | fails the integer check |
| `line_range: [5, 2]` | rejected: `line_range must be [start_line, end_line] with 1 <= start_line <= end_line.` |

### Description text, verbatim, `latest`

```
Reads a file from the local filesystem. You can access any file directly by using this tool.
If the User provides a path to a file assume that path is valid. It is okay to read a file that does not exist; an error will be returned.

Usage:
- You can optionally specify a line offset and limit (especially handy for long files), but it's recommended to read the whole file by not providing these parameters.
- Lines in the output are numbered starting at 1, using following format: LINE_NUMBER|LINE_CONTENT
- You have the capability to call multiple tools in a single response. It is always better to speculatively read multiple files as a batch that are potentially useful.
- If you read a file that exists but has empty contents you will receive 'File is empty.'

Image Support:
- This tool can also read image files when called with the appropriate path.
- Supported image formats: jpeg/jpg, png, gif, webp.

PDF Support:
- PDF files are converted into text content automatically (subject to the same character limits as other files).
```

Two things to preserve if you copy it. The line-number sentence says `using following format`, with no article. That is not a typo introduced here; it is what the bundle emits, and a leaked prompt from November 2025 carries the same missing article, which is good evidence the leak is verbatim. And when the `enableReadFileExplicitOffsetLimitDescription` flag is on, the first usage bullet is replaced by two:

```
- Omit offset and limit to read the entire file.
- Provide offset and limit to read a specific line range (especially for long files).
```

How the other branches differ:

| Branch | Difference |
|---|---|
| `dsv3-1018` | leading `\n`, trailing period after `LINE_NUMBER\|LINE_CONTENT.`, no image section, no PDF section |
| `dsv3-1205` | no leading newline, no trailing period, image section, no PDF section |
| `cursor-0226` | one line, no usage bullets, no line-number sentence: `Reads a file from the local filesystem. This tool can also read image files when called with the appropriate path. Formats supported: jpeg/jpg, png, gif, webp.` |
| `haiku` | `line_range` bullets instead of offset/limit, then the standard tail |
| minimal harness | `View an image on the local filesystem. This tool can only view IMAGE files. For normal file reads use the Shell tool.` |

The `haiku` bullets, verbatim:

```
- For partial file reads, use the line_range parameter with [start_line, end_line] format (1-indexed, inclusive). Example: [100, 150] reads lines 100 through 150.
- It's recommended to read the whole file by not providing line_range, unless the file is too large.
```

### Caps and limits

| Limit | Value | Unit | Layer that enforces it |
|---|---|---|---|
| binary extension list | 16 extensions | path suffix, case insensitive | tool, before any I/O |
| read character cap | 100000 | JavaScript string length, UTF-16 code units | tool, after windowing |
| blob window | `(10000, 100000]` | JavaScript string length | tool, after windowing |
| whole-file buffer | 8388608 (8 MiB) | JavaScript string length | capability service |
| whole-file refusal | 8388608 (8 MiB) | bytes on disk, from `stat` | capability service, `readWholeTextFile` |
| streaming threshold | 8388608 | bytes on disk | capability service |
| hard refusal | 2147483647 (2 GiB) | bytes on disk | capability service, terminal-file path only |
| read chunk size | 65536 | bytes | capability service |
| PDF text cache | 33554432 (32 MiB) | string length | tool, module global |
| notebook cell output | 2000 | characters per cell | renderer |
| retry wait | 120000 | milliseconds | tool step controller |
| binary sniff window | 8192 | bytes from the start of the file | capability service |
| control-byte binary threshold | 5 | percent of the sniff window | capability service |
| CRLF detection threshold | 5 | percent of newline pairs | capability service |
| encoding confidence floor | 0.7 | jschardet confidence | capability service |
| line number width | 6 | characters, space padded | formatter |
| sparse line-number interval | 10 | lines, hardcoded in `Read` | formatter |

The 100,000 figure appears twice, once as `LIe=1e5` in the constant declaration and once hardcoded in the message text. They agree today. The blob window uses the same constant as its upper bound, so raising the cap for the message without raising `LIe` produces a message that lies about the threshold.

### Result cases, and the text the model sees

A read produces a record and a render. The record is a oneof; the model only sees the render. The cases:

`success`, `fileNotFound`, `notFile`, `permissionDenied`, `fileBusy`, `rejected`, `error`. `Read` never produces `fileBusy` and never produces a literal `notFile`: directory and binary-content failures arrive as `invalidFile` from the service and are thrown as text, which `serializeError` turns back into an `error`-case record. So the *record* case for a directory is `error` even though the *model* is told `Path is a directory, not a file`.

| Condition | Exact model-visible text | isError |
|---|---|---|
| ordinary text | the formatted content, nothing else | false |
| empty file, no range applied | `File is empty.` | false |
| empty range applied | one line, `     1\|` | false |
| over the cap | two lines, below | **true** |
| `.ipynb` | notebook cells, no line numbers | false |
| image | text part `Read image file: <resolved path>` plus a base64 image part | false |
| file not found | `File not found` | true |
| permission denied | `Permission denied` | true |
| directory | `Path is a directory, not a file: <path>` | true |
| device, socket, fifo | `Path is neither a file nor a directory: <path>` | true |
| binary content | `File content is binary data, not text: <path>` | true |
| undecodable image | `File is not a valid image: its data could not be decoded (<reason>). A truncated or hand-copied file does this; write it again from the original bytes with code.: <path>` | true |
| binary extension | `Cannot read binary files of type <ext>` | true |
| rejected | `Read operation rejected` | true |
| offset past EOF | `Offset <offset> is beyond file length (<n> lines)` | true |
| service error | `Error: <message>` | true |
| unknown result case | `Unknown error` | true |

The cap message, exactly two lines with one `\n` between them:

```text
Requested content exceeds maximum allowed characters (100000 characters). The file is 1234567 bytes.
Please use offset and limit parameters to read specific portions of the file, or use the 'grep' tool to search for specific content.
```

The `<n>` in the offset message is the *file's* line total, not the offset. The `<offset>` is the raw argument after int32 clamping, so a `line_range` read that overruns prints the start line, and `1e20` prints as `2147483647`.

The two failures that carry no path and no punctuation, `File not found` and `Permission denied`, are the exception rather than the rule. Every other failure names the path, and the path it names is the `lke`-normalized *argument*, not the service-resolved absolute path, so a relative argument is echoed back relative.

### The three line-number styles

`fH(formattingOptions, content, startLineNumber)`, in `main.js` at 2529940 and daemon `addLineNumbers`. The offset 2530076, which the parent spec gives, lands inside the function's first branch rather than at its start. Mutually exclusive, tested in this order.

**Codex style, when `gpt5CodexCatN` is true.** Six wide, right aligned, then two spaces, then the text. No separator glyph.

```js
`${(startLine + i).toString().padStart(6, " ")}  ${line}`
```

```
     1  alpha
     2  beta
```

**GPT-5 style, when `gpt5StyleLineNumbers` is true.** The letter `L`, the unpadded number, a colon, the text.

```js
`L${lineNumber}:${text}`
```

```
L1:alpha
L2:beta
```

**Default.** Six wide, right aligned, pipe, text, no space after the pipe.

```js
`${lineNumber.toString().padStart(6, " ")}|${text}`
```

```
     1|alpha
     2|beta
```

**Sparse, layered on the default.** When `sparseLineNumbers` is an integer N, only lines whose number is a multiple of N get a prefix; every other line is emitted raw with no padding. `Read` forces N to 10 and forces both other styles off when sparse mode is on:

```js
gpt5StyleLineNumbers:!l&&t.gpt5StyleLineNumbers,gpt5CodexCatN:!l&&t.gpt5CodexCatN,enableLineNumbers:u,sparseLineNumbers:l?10:t.sparseLineNumbers
```

Measured, N=10, lines 1 through 12:

```
line 1
line 2
line 3
line 4
line 5
line 6
line 7
line 8
line 9
    10|line 10
line 11
line 12
```

Which model family gets which, from the formatting-options builder:

- `gpt5StyleLineNumbers` is `modelInfo.isGpt5Family`, true for `gpt-5` and its point releases and any `gpt-5*codex` slug.
- `gpt5CodexCatN` is **hardcoded `false`** at every one of the eight construction sites in the four shipped bundles. There is no path in this build that sets it true. The Codex style is dead code in 3.27.12.
- Sparse mode is chosen by `Kze`, which enables it only when the prompt version is exactly `cursor-0226`, and then only inside a Composer check.

That last point changes a design decision. The three-style split is a compatibility layer for models Cursor no longer routes, and reproducing it buys nothing. Implement the default, and make the style a parameter so a future model family can get a different one.

### Omitted-line notices

They are prepended and appended by `hH` only when the caller asks, and `Read` asks only when the window is partial:

```js
{addAmountOfOmittedLines:void 0!==r.value.readRange&&(r.value.readRange.startLine>1||r.value.readRange.endLine<a)}
```

So a full-file read gets none, and neither does `offset: 1` with no limit, because its range starts at 1 and ends at `totalLines`.

The head notice fires when `startLineNumber > 1`:

```js
`... ${t} ${1===t?"line":"lines"} not shown ...\n${r}`
```

with `t = startLineNumber - 1`. The tail notice fires when `totalLineNumbersInFile > startLineNumber + count(content) - 1`, with the count taken from the *content* rather than from the requested range:

```js
`${r}\n... ${t} ${1===t?"line":"lines"} not shown ...`
```

Measured, a one-line window at line 41 of a 50-line file:

```text
... 40 lines not shown ...
    41|export function shutdown(): void {
... 9 lines not shown ...
```

There is no blank line between a notice and the content, and the wording is identical head and tail.

### The `pi_read` counterpart

Cursor ships a second family of tools in the same bundle, registered immediately after the `PR_MANAGEMENT` tools, described as "pi-compatible". `pi_read` is registered at `AG("PI_READ", ...)` with the name `pi_read` at 4556843, forwarding to its own proto message pair `Sg.tC` (args) and `Sg.ni` (result), and on the daemon side to `LocalPiReadExecutor` in `../local-exec/dist/pi/read.js`.

Its parameters, verbatim:

```js
path:   z.string().describe("Path to the file to read (relative or absolute)")
offset: z.number().optional().describe("Line number to start reading from (1-indexed)")
limit:  z.number().optional().describe("Maximum number of lines to read")
```

Its entire description:

```
Read the contents of a file using pi-compatible semantics.
```

What it does, in order:

1. `lke` the path (the same `worktries` repair).
2. `resolvePiReadPath`, which is `resolvePiPath` plus a macOS fallback. `resolvePiPath` replaces `\u00a0` with a space, strips one leading `@`, converts `file://` to a path, expands `~`, then `path.normalize` and, if relative, `path.resolve` against the workspace. The macOS fallback then tries the basename in NFD, with `'` replaced by `\u2019`, with a space replaced by `\u202f`, and both.
3. **No binary extension check at all.** A `.zip` is read and decoded.
4. `stat`, then `Path is a directory: <resolved>` if it is one.
5. `readFile(...).toString("utf8")`, so invalid UTF-8 becomes U+FFFD and nothing is reported. **No encoding detection, no BOM handling, no binary sniff.**
6. `split("\n")`, slice by `offset - 1` and `limit`, join.
7. `truncateHead` with 2000 lines and 50 KiB.
8. Trailers.

The trailers, verbatim:

```text
[Showing lines {start}-{end} of {total}. Use offset={end + 1} to continue.]
```

```text
[{remaining} more lines in file. Use offset={next} to continue.]
```

```text
[Line {n} is {size}, exceeds {maxSize} limit. Use bash: sed -n '{n}p' {path} | head -c {maxBytes}]
```

with `formatSize` producing `1234B` below 1024 and `12KB` above, rounded. The errors:

```text
Read blocked: {reason}
```

```text
Path is a directory: {resolved}
```

```text
Offset {offset} is beyond end of file ({n} lines total)
```

Is it a better model for a fresh implementation? **Partly, and the split matters.** The parameter shape is better: three fields, no per-model branching, no `target_file`, no `line_range` array, no optional boolean. The cap units are better: bytes and lines are things a Rust implementation counts natively, and the model can reason about them. The trailers are better: `[Showing lines 1-2000 of 5000. Use offset=2001 to continue.]` gives the model the exact next call, where Cursor's error message only says "use offset and limit".

What is worse: no line numbers, so nothing downstream can anchor on a line; no encoding handling, so a Latin-1 file reads as mojibake with no signal; no binary detection, so a model that asks for a `.zip` gets 50 KiB of garbage that it will try to interpret; and the fallback message tells the model to shell out to `sed`, which presumes a shell you may not have.

The recommendation, restated: take the parameter shape, the byte-and-line caps, and the trailers. Keep Cursor's line-numbered output and Cursor's encoding and binary handling. That combination does not exist in either product.

## Algorithm and ordering

Eighteen steps in execution order. The numbered ones are load bearing where marked.

```
TOOL Read

INPUTS
  args        parsed arguments: { path | target_file, offset?, limit?, line_range?, include_line_numbers? }
  ctx         request context: abort signal, workspace roots, stateHandler, blobStore, step call list
  lockManager per-turn FileOperationLockManager, may be absent entirely
  service     capability service: read(path, offset, limit) -> oneof result

OUTPUT
  a ReadToolCall record, oneof { success(ReadToolSuccess) | error }, which render turns into text

 1  PARSE      validate against the version schema
                 on failure THROW "Invalid arguments:\n<key>: <message>"
                 no lock taken, no file touched

 2  rawPath  = "target_file" in args ? args.target_file : args.path
 3  path     = lke(rawPath)                       // the worktries repair only, see below

 4  dispose  = lockManager?.waitForLock(ctx, rawPath)     // RAW string, not path
                 held until every exit, including all throws below

 5  IF path matches /\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i
                 ext = lowercase(last ".<suffix>") or "unknown"
                 THROW "Cannot read binary files of type <ext>"     // before any file access

 6  IF args.line_range is present
                 offset = line_range[0]
                 limit  = line_range[1] - line_range[0] + 1        // inclusive end becomes a count
    ELSE        offset = args.offset; limit = args.limit
    offset = clampToInt32(offset); limit = clampToInt32(limit)     // see ORDERING NOTE 1

 7  first    = service.read(path, offset, limit)
                 execId = seededUuid(toolCallId)                   // sha256 based, deterministic

 8  miss     = classify(first)
                 fileNotFound                                   -> "file_not_found"
                 invalidFile AND reason starts "File is not a valid image:"  -> "partial_image"
                 error AND the path has an image extension       -> "partial_image"
                 anything else                                  -> none

 9  IF miss is not none
                 dispose()                                         // RELEASE before waiting
                 waited = waitForEarlierCalls(toolCallId, 120000, {includeSettled:true, toolNames: MUTATING})
                 IF waited is undefined OR waited.settled is false
                     result = first                                // original miss, unchanged
                 ELSE
                     dispose = lockManager?.waitForLock(ctx, rawPath)   // RE-ACQUIRE
                     result  = service.read(path, offset, limit)
                     execId  = seededUuid(toolCallId + ":same_step_read_race")
                 // exactly one retry, and the retry never re-enters this block
    ELSE         result = first

10  SWITCH result.case
                 "error"            -> THROW result.error
                 "rejected"         -> THROW result.reason || "Read operation rejected"
                 "fileNotFound"     -> THROW "File not found"
                 "permissionDenied" -> THROW "Permission denied"
                 "invalidFile"      -> THROW (result.reason || "Path is not a valid file to read") + ": " + path
                 default            -> THROW "Unknown error"
                 // each of these is caught by serializeError and becomes an error-case record,
                 // which render then prefixes with "Error: "

11  resultPath   = result.path                    // service-resolved, NOT the argument
    fileSize     = Number(result.fileSize)        // arrives as bigint from protobuf
    rangeApplied = result.rangeApplied === true
    totalLines   = result.totalLines              // the service's full-file count
    out          = result.output                  // oneof: content | data | contentBlobId | dataBlobId

12  IF out.case is "data"
                 IF PDF sniff passes (%PDF magic, or path ends ".pdf" case-insensitively)
                     text = pdfToText(out.value)
                     text = text.replace(/\r\n/g, "\n").replace(/\r/g, "\n")
                     cache by sha256 hex, LRU 32 MiB
                     out = { case:"content", value: text }
                 ELSE IF an image mime is detectable (magic bytes first, extension second)
                     RETURN image success:
                       content = [ {type:"text",  text:"Read image file: " + resultPath},
                                   {type:"image", data: base64(bytes), mimeType} ]
                       totalLines = 0, readRange absent, isEmpty false, exceededLimit false
                 ELSE
                     RETURN the data record unchanged        // video and other binary payloads

13  IF blobStore exists AND out.value.length > 10000 AND out.value.length <= 100000
                 store UTF-8 bytes of out.value under sha256
                 RETURN success { output: contentBlobId, totalLines, readRange 1..totalLines }
                 // note: the range here covers the WHOLE file, not a window

14  IF out.value === "" AND rangeApplied is false
                 RETURN success { output:"", isEmpty:true, totalLines:0, readRange:absent }
                 // totalLines is a literal 0; the line counter is never consulted

15  IF rangeApplied
                 window    = computeReadSlice(totalLines, offset, limit)
                 readRange = window?.readRange ?? { startLine:1, endLine: totalLines }
                 RETURN success { output: out.value, totalLines, readRange }
                 // the service already sliced, so the tool slices nothing

16  ELSE
                 lines     = (offset !== undefined OR limit !== undefined)
                               ? out.value.split("\n")
                               : undefined
                 lineCount = lines === undefined ? countNewlines(out.value) + 1 : lines.length
                 window    = computeReadSlice(lineCount, offset, limit)
                 IF window !== undefined
                     out.value  = lines.slice(window.startIndex, window.endIndex).join("\n")
                     totalLines = lineCount
                 readRange = window?.readRange

17  IF out.value.length > 100000                       // AFTER windowing, never before
                 RETURN success { output:"", exceededLimit:true, isEmpty:false,
                                  totalLines, fileSize, path: resultPath, readRange }

18  RETURN success { output: out.value, exceededLimit:false,
                    isEmpty: (totalLines === 0 AND out.value === ""),
                    totalLines, fileSize, path: resultPath, readRange }

RENDER(record)                                          // sees only the record, never a throw
  case "error"                     -> "Error: " + errorMessage, isError true
  success AND isEmpty              -> "File is empty.", isError false
  success AND exceededLimit        -> the two-line cap message, isError TRUE
  success AND path ends ".ipynb"   -> notebook cell rendering, no line numbers, no formatter call
  success AND output is image      -> two-part content array, isError false
  success otherwise                -> renderNumberedLines(content,
                                        startLine = readRange.startLine ?? 1,
                                        totalLines,
                                        style,
                                        addOmittedLines = readRange exists AND
                                          (readRange.startLine > 1 OR readRange.endLine < totalLines))
```

### The window arithmetic, exactly

```
computeReadSlice(totalLines, offset, limit)

1  IF offset is undefined AND limit is undefined: RETURN undefined
2  effectiveOffset = offset ?? 1
3  effectiveLimit  = limit ?? (effectiveOffset < 0 ? abs(effectiveOffset) : totalLines)
4  startIndex      = effectiveOffset < 0
                       ? max(0, totalLines + effectiveOffset)     // -1 is the last line
                       : max(0, effectiveOffset - 1)              // 1 is the first line
5  endIndex        = min(totalLines, startIndex + effectiveLimit)
6  IF startIndex >= totalLines
       THROW "Offset <offset> is beyond file length (<totalLines> lines)"
       // <offset> is the raw argument, not effectiveOffset, so `undefined` can never appear
7  RETURN { startIndex, endIndex, readRange: { startLine: startIndex + 1, endLine: endIndex } }
```

Measured against the shipped function, with the numbers it actually produces:

| Call | Result |
|---|---|
| `(10, undefined, undefined)` | `undefined`, no window |
| `(10, 1, 2)` | start 0, end 2, range `{1, 2}` |
| `(10, 0, undefined)` | start 0, end 10, range `{1, 10}`; zero became one |
| `(10, 10, undefined)` | start 9, end 10, range `{10, 10}` |
| `(10, 11, undefined)` | throws `Offset 11 is beyond file length (10 lines)` |
| `(3, 50, undefined)` | throws `Offset 50 is beyond file length (3 lines)` |
| `(5, -2, undefined)` | start 3, end 5, range `{4, 5}`; `abs(-2)` lines |
| `(10, -100, undefined)` | start 0, end 10, range `{1, 10}`; clamped, no throw |
| `(10, -2, 5)` | start 8, end 10, range `{9, 10}`; limit does not extend past EOF |
| `(10, undefined, 3)` | start 0, end 3, range `{1, 3}`; limit alone means from line 1 |
| `(10, undefined, 0)` | start 0, end 0, range `{1, 0}`; unreachable through the schema |
| `(0, 0, undefined)` | throws `Offset 0 is beyond file length (0 lines)`; unreachable, see step 14 |

`readRange.endLine` is `endIndex`, which is a zero-based exclusive index, but it is *reported* as an inclusive line number. They coincide because `startIndex` is zero-based and `endIndex` counts lines: for a full read of 10 lines, `startLine` is `0 + 1 = 1` and `endLine` is `10`. Worth stating explicitly because a reimplementation that computes `endLine = endIndex + 1` reports an 11-line file for a 10-line read.

### The line counter

```
countNewlines(s)
  count every "\n"; RETURN count + 1
```

So `""` is 1, `"a"` is 1, `"a\n"` is 2, `"\n"` is 2. The empty-file path never reaches it, which is why an empty file reports `totalLines: 0` rather than 1.

There is a second counter in the daemon with the opposite convention for empty input and CR awareness:

```js
function countLines2(text){if(text.length===0)return 0;let lineCount=1;for(let i=0;i<text.length;i+=1){const c=text[i];if(c==="\n")lineCount+=1;else if(c==="\r"){lineCount+=1;if(text[i+1]==="\n")i+=1}}return lineCount}
```

The read path does not use it. It belongs to a different subsystem, and picking it up by mistake gives you a CR-aware count on a read path that has already normalized CRLF.

### Ordering notes

**1. The int32 clamp happens before the service call and before the window.** `clampToInt32` saturates to `[-2147483648, 2147483647]`. Because it runs at step 6, the value the window function sees and the value the error message interpolates are both the clamped one, so `offset: 1e20` produces `Offset 2147483647 is beyond file length (10 lines)` and never mentions `1e20`.

**2. The binary-extension rejection precedes the service call.** This is the only place in the read path where a bad path costs zero I/O. Swapping it after the read produces the same *error* but pays a full read first, and on a 2 GB `.bin` file that is the difference between a fast failure and a slow one.

**3. The lock is taken on `rawPath`, and the binary check uses `path`.** The two differ only when `lke` rewrites the argument, which is to say for `~/.cursor/worktries/...` paths. Reproducing that split faithfully means a `worktries` path is locked under its misspelled form and read under its correct one.

**4. The empty-file short circuit at step 14 is checked before the window at step 15.** Swapping them changes `offset: 500` on an empty file: with the current order you get `File is empty.` and with the swap you get an exception from `computeReadSlice(0, 500, undefined)`. Both are defensible; only one is Cursor.

**5. The cap at step 17 is checked after the slice at step 16.** This is the single most important ordering fact in the tool. A 200,000-character file read whole trips the cap; the same file read with `offset: 3, limit: 2` returns two lines and no error, because the slice happens first. Reversing these makes every large-file read fail and removes the only documented escape hatch.

**6. The blob path at step 13 returns before the cap check.** Because the blob window's upper bound is the same 100,000 as the cap, nothing that reaches the blob can exceed the cap, so the order is harmless. It is still worth knowing that a blob-backed result never carries `readRange` from a window: step 13 uses `{startLine: 1, endLine: totalLines}`, which means a blob read of a *windowed* result would report a full-file range. In practice a windowed read is always under 10,000 characters in the cases that matter, so this is latent rather than live.

**7. The retry releases the lock before waiting.** Holding it while waiting for a writer to finish, when the writer needs the same key, is a deadlock. The observable difference is a hang versus a slow failure, and a hang is much harder to diagnose. If you rewrite this, keep the release and put the re-acquire inside the retry thunk, exactly as Cursor does, rather than around it.

**8. The retry is bounded at one attempt and gated four ways.** It fires only when the miss classifier returns a label, only when the step controller is present, only when this tool call appears in the step's call list, and only when everything it waits for settled inside the timeout. Any of the four failing returns the original miss unchanged, so the model sees the ordinary error text.

## Engineering questions

### Q1 Pagination and chunking

Yes, but the paging is line-based and the caller resumes by recomputing an offset. There is no cursor and no server-side state.

`offset` is 1-indexed and is the first line returned. `limit` is a count of lines, not an end line. Omit both and you get the whole file. Provide `limit` alone and you get lines 1 through `limit`. Provide `offset` alone and you get from `offset` to end of file. The `haiku` branch offers `line_range: [start, end]` with an inclusive end, converted internally to `offset = start`, `limit = end - start + 1`.

A page boundary never falls mid-line, because the slice is over an array of lines produced by `split("\n")`. It can, however, fall mid-*sequence* in one specific way: the split happens on the JavaScript string, which is UTF-16, and a slice at a line boundary is always at a code-unit boundary that is also a scalar boundary, because a newline cannot appear inside a surrogate pair. So there is no way to split a character here. There is also no way to split a CRLF pair, because the `\r` is at the end of the previous line's text and the window boundary sits after the `\n`.

Two details that matter for a reimplementation. First, a windowed read does not renumber: the line numbers reflect the file's own numbering, so `offset: 41` produces lines beginning `    41|`. Second, the *tail notice* is computed from the content's line count rather than from the requested range, so if the content was truncated somewhere upstream the notice understates how much is missing.

**Resuming past the cap requires the caller to do arithmetic.** There is no "here is where to continue" in the cap message. The model is told to use `offset` and `limit` but not which values. `pi_read` does this better; see its trailers under `## The contract`.

### Q2 Units

This is the most consequential question in the report, so it gets the most detail.

**Four different units are in play and they are not interchangeable.**

| What is measured | Unit | Where |
|---|---|---|
| the 100,000 read cap | JavaScript string length, i.e. UTF-16 code units | `bTe`, the tool |
| the blob window `(10000, 100000]` | same | tool, step 13 |
| the 8 MiB whole-file cut | same, via `content.length` then `substring` | capability service |
| the 2 GiB refusal | bytes on disk, from `stat` | capability service |
| the streaming threshold | bytes on disk | capability service |
| `offset` and `limit` | lines | tool |
| `fileSize` in the cap message | bytes on disk | capability service, into the message |

The exact comparison, verbatim from `bTe` at 4343863 in the bundle, which the daemon calls `createSuccessResult3`:

```js
content.length > READ_CHAR_HARD_LIMIT
```

`content` is a JavaScript `String`, so `.length` is the number of UTF-16 code units. `READ_CHAR_HARD_LIMIT` is `CHAR_HARD_LIMIT`, which is `1e5`.

**What that means concretely, in the three encodings that matter.**

| File contents | JS length (UTF-16 units) | UTF-8 bytes | Rust `str::len()` | Rust `chars().count()` |
|---|---|---|---|---|
| 100,000 ASCII characters | 100,000 | 100,000 | 100,000 | 100,000 |
| 100,000 CJK characters (U+4E00 each) | 100,000 | 300,000 | 300,000 | 100,000 |
| 100,000 emoji (U+1F600 each) | 200,000 | 400,000 | 400,000 | 100,000 |
| 50,000 CJK plus 50,000 ASCII | 100,000 | 200,000 | 200,000 | 100,000 |

So a 100,000-character all-CJK file sits exactly *at* Cursor's cap and passes, while a byte-counting Rust port sees 300,000 and refuses it. The inverse error is worse: a 100,001-character file of CJK is 300,003 bytes, so a byte cap set anywhere near Cursor's number refuses files Cursor reads.

And the emoji case cuts the other way. 100,000 emoji are 200,000 UTF-16 code units, so Cursor **refuses** a file that a `chars()`-counting port would accept. Both ports diverge from Cursor, in opposite directions, on different inputs.

**A specific numeric example to test against.** Take a file of exactly 50,000 CJK characters with no newlines, so one line:

- Cursor: `length` is 50,000, under the cap, so the read succeeds and the model sees one line of 50,000 characters with a `     1|` prefix.
- A `str::len()` port with a 100,000 byte cap: 150,000 bytes, over the cap, so it refuses.
- A `chars().count()` port with a 100,000 scalar cap: 50,000, under, succeeds. Same answer as Cursor here, different answer on the emoji case.

**What I recommend.** Pick one unit and put it in the description in plain words. If you want byte-for-byte Cursor parity, use `s.encode_utf16().count()` and accept the double pass. If you want sanity, use `s.len()` in bytes and set the number high enough that it never fires for text people actually read, and say so. What you should not do is use `s.len()` while calling the limit "characters", because that produces a tool whose error message is false on exactly the files where it fires.

**One more unit note that is easy to miss.** `fileSize` in the cap message comes from the service's `stat`, so it is bytes even when the cap is measuring code units. A 100,001-character CJK file reports `The file is 300003 bytes` beside a cap of 100000 characters, which reads as a contradiction to a model that is not thinking about encodings.

### Q3 Text encoding

**Invalid UTF-8 on the streaming path** (which is the default when streaming is enabled, and forced above 8 MiB): the decoder is `new TextDecoder("utf-8", {fatal: true, ignoreBOM: true})`, and a `TypeError` becomes `InvalidUtf8Error` with message `Invalid UTF-8 while streaming read`. That is caught and the whole read is retried with a fallback decoder chosen by sampling the first 1 MiB:

```js
const lossy = new TextDecoder("utf-8", {fatal:false}).decode(sample, ...);
if (isMostlyValidUtf8(lossy)) return { kind:"lossyUtf8", cacheEncoding:"utf8" };
return { kind:"iconv", decoderEncoding:"latin1", cacheEncoding:"latin1" };
```

`isMostlyValidUtf8` counts U+FFFD against clean non-ASCII characters and returns `cleanNonAsciiCount > replacementCount`. So a file that is *mostly* valid UTF-8 with a few stray bytes is read lossily, with U+FFFD in place of the strays, and a file that is mostly not UTF-8 is decoded as Latin-1, which never fails and always produces one character per byte. There is no refusal. The model cannot tell which happened.

**Invalid UTF-8 on the whole-file path:** `readText` calls `getFormatForBuffer` on the raw bytes and throws `Binary file cannot be decoded as text` if the binary heuristic fires. Otherwise it decodes with the detected encoding via `iconv-lite`, which replaces undecodable bytes with U+FFFD and does not report how many.

**Lone surrogates.** A JavaScript string can hold an unpaired surrogate, and nothing in the read path guards against one. It cannot arise from a UTF-8 decode, because UTF-8 cannot encode a lone surrogate; a WTF-8 or CESU-8 byte sequence would decode as replacement characters. So in practice the read path produces well-formed strings, and the lone-surrogate problem is the *write* side's, not Read's.

**BOM.** Three different things happen depending on the path.

- A UTF-8 BOM is stripped from the text (`stripUtf8Bom`) and remembered as the encoding `utf8bom`, so a later write through the same subsystem restores it. The model never sees U+FEFF.
- The streaming decoder is constructed with `ignoreBOM: true`, and `normalizeChunk` additionally strips a leading `U+FEFF` from the first chunk, recording `hadLeadingBom`. So the streaming path strips it twice, harmlessly.
- A UTF-16 BOM selects the encoding; the decoded text carries no BOM.
- For the terminal-file and agent-tool-output paths, `materializeUtf8Document` records `bomStart` so the byte count excludes the BOM, and `contentEmpty` accounts for it.

**Normalization.** None. No NFC, no NFD, no case folding, no Unicode whitespace folding. The one exception is in `pi_read`, which is a different tool: `resolvePiPath` replaces `\u00a0` with a space and tries NFD variants of the basename on macOS. `Read` does none of that.

**What this means for a Rust port.** `Access::read_string` returns `VfsError::NotUtf8 { valid_up_to }` on invalid UTF-8, which is a *hard refusal*. Cursor's Read never refuses on encoding; it substitutes or falls back. So a straight port turns "mojibake the model can see and reason about" into "an error the model cannot act on". Three options:

1. Use `String::from_utf8_lossy` and accept U+FFFD, matching the lossy path.
2. Decode with `encoding_rs` using the same BOM and heuristic ladder, which is a fair amount of code for a case that mostly produces mojibake either way.
3. Read bytes, detect binary with the NUL-and-control-byte rule, and if it is not binary, use lossy decoding plus a one-line note in the output that non-UTF-8 bytes were replaced.

**Recommendation: option 3.** Confidence medium-high. It never refuses, it tells the model what happened, and it costs a handful of lines instead of an encoding-detection subsystem. The one thing not to do is return `NotUtf8` to the model, because there is no useful action for it to take.

### Q4 Line endings

**Detection.** `determineLineEndingsForBuffer` walks the first 8192 bytes and counts each `\n` as CRLF if the preceding byte is `\r`, else as LF-only. Then:

```js
function lineEndingFromCounts(crlfCount, lfOnlyCount) {
  if (crlfCount === 0 && lfOnlyCount === 0) return LineEnding.LF;
  const total = crlfCount + lfOnlyCount;
  return crlfCount / total * 100 >= 5 ? LineEnding.CRLF : LineEnding.LF;
}
```

**So the classification is a vote over the 8 KiB sniff window, with a 5% threshold.** A file that is 96% LF and 4% CRLF is classified LF. A file that is 5% CRLF is classified CRLF and every `\r\n` in it is replaced.

**Whole-file path.** `readText` ends with:

```js
if (format.lineEnding === LineEnding.CRLF) return str.replaceAll("\r\n", "\n");
return str;
```

Bare `\r` is untouched on this path, so a classic-Mac file with CR-only endings arrives as one line containing carriage returns.

**Streaming path.** `\r` is kept in each line's text while scanning, and stripped at join time when the vote passed:

```js
const shouldNormalizeCrlf = totalLineEndings > 0 && crlfCount / totalLineEndings * 100 >= 5;
```

recomputed over the *whole file* rather than the sniff window, so the streaming and whole-file paths can disagree on a file whose first 8 KiB and full contents sit on opposite sides of 5%. The strip also requires `hasNewline`, so a final line with a bare `\r` and no `\n` keeps it.

**PDF-converted text** is normalized unconditionally, both `\r\n` and bare `\r`, before caching:

```js
text.replace(/\r\n/g, "\n").replace(/\r/g, "\n")
```

**Windowing does not change this, and the asymmetry people expect is not there.** The tool-side window is `d.split("\n")`, slice, `join("\n")`, which is byte-lossless with respect to `\r`: a below-threshold file read with a window keeps exactly the same carriage returns a whole-file read of the same file would show, because the `\r` lives at the end of the previous line's text and the join puts it straight back. Verified by executing the shipped expression on a mixed file: `"a\r\nb\nc\r\nd"` windowed to its first three lines gives `"a\r\nb\nc\r"`, with the carriage return intact.

What *does* differ is a different pair of paths. `materializeTerminalFileText` for terminal logs and agent-tool output normalizes through `takeUtf8Window` or `takeNormalizedString`, both of which call `normalizeCrlf` **unconditionally**, replacing every `\r\n` with `\n` whether or not the 5% vote passed. The ordinary text path goes through `readText`, which applies the vote. So the same bytes read as a terminal log and read as a source file can come back with different line endings, and that is the real inconsistency in this build.

**Net effect the model sees.** A consistent CRLF file is shown as LF. A consistent LF file is shown as LF. A mixed file is shown with literal `\r` characters before the `\n` on any CRLF line, if it fell below the 5% threshold. A CR-only file is shown as one very long line with embedded carriage returns.

**A trap inside the trap.** The line *counter* runs on `\n` only, so a CR-only file counts as 1 line no matter how many logical lines it has. `offset: 2` on such a file throws `Offset 2 is beyond file length (1 lines)`. Note also the plural: the message says `1 lines`, because the template is `` `... (${e} lines)` `` with no singular form.

**Recommendation for Runvil.** Strip one trailing `\r` from every line at format time, unconditionally, and count lines on `\n`. That is what Claude Code's renderer does, it is what Cursor's streaming path does when the vote passes, and it never leaves the model looking at an invisible control character. Diverges from Cursor only on files between 0% and 5% CRLF, where Cursor's output is the worse of the two.

### Q5 Binary detection

Three separate mechanisms, at three layers, with three different outcomes.

**1. Extension list, in the tool, before any I/O.** `aTe`, offset 4326097 in the old bundle and 4341, in the new one, immediately before the `cTe` basename helper:

```js
/\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i
```

Sixteen extensions, case insensitive, anchored at the end of the path. A match throws `Cannot read binary files of type <ext>` where `<ext>` is the lowercased suffix after the last `.`, or `unknown` if the regex matched but no suffix could be extracted (unreachable in practice).

What is **not** in the list, and why it matters:

| Extension | Absent because |
|---|---|
| `.pdf` | goes to the PDF conversion branch |
| `.png` `.jpg` `.jpeg` `.gif` `.webp` `.ico` `.bmp` `.svg` `.tif` `.heic` `.avif` | go to the image branch |
| `.ipynb` | is JSON, read as text, then reformatted as notebook cells |
| `.jar` `.7z` `.bz2` `.xz` `.wasm` `.class` `.o` `.a` `.woff` `.woff2` `.ttf` | **not handled at all**, so they fall through to the content sniff |
| `.docx` `.xlsx` `.pptx` | also fall through; they are zip containers with a `.docx` name |

So an archive renamed to `.dat` is caught by the content sniff and a `.docx` is caught by the content sniff, but a `.jar` is caught only if the content sniff recognizes it, which for a zip container it will (the `PK\x03\x04` prefix is in the magic list).

**2. Content sniff, in the capability service, on the first 8192 bytes.**

```js
function isBufferText(buffer){
  const len = Math.min(8192, buffer.length);
  if (len === 0) return true;
  for (let i = 0; i < len; i++) if (buffer[i] === 0) return false;
  let nonPrintable = 0;
  for (let i = 0; i < len; i++) {
    const b = buffer[i];
    if (b < 32 && b !== 9 && b !== 10 && b !== 13) nonPrintable++;
  }
  return nonPrintable / len * 100 < 5;
}
```

Two rules: **any NUL byte in the first 8192 bytes is binary**, and **more than 5% control bytes other than tab, LF, and CR is binary**. Note that the NUL rule has no percentage at all: one NUL in 8192 bytes makes the file binary. Note also what this misses: a UTF-16LE text file has NULs on every other byte, so it is classified binary *by this function* and only rescued because the UTF-16 BOM check runs first and returns early.

A binary verdict produces `invalidFile` with reason `File content is binary data, not text`, which the tool renders as:

```
File content is binary data, not text: <path>
```

**3. A magic-prefix list, but only on the agent-tool-output path.** `KNOWN_BINARY_MAGIC_PREFIXES` holds 25 signatures, including `%PDF-`, `PK\x03\x04`, `\x7fELF`, Mach-O in four flavours, `MZ\x90`, `\0asm`, `\x89PNG\r\n\x1a\n`, JPEG `\xff\xd8\xff`, GIF87a, GIF89a, `SQLite format 3\0`, gzip `\x1f\x8b`, bzip2 `BZh`, xz, zstd, and two RAR variants. This list is consulted only in `readTerminalFile` when the path is an agent-tools output file and not a terminal log. On the ordinary read path it is not reached, because the 8192-byte control-byte and NUL heuristics fire first.

**What the model sees, summarized.**

| Case | Text |
|---|---|
| extension in the list | `Cannot read binary files of type zip` |
| content sniff says binary | `File content is binary data, not text: <path>` |
| `.pdf` | converted text, or a conversion error |
| image | `Read image file: <path>` plus an image part |
| `.ipynb` | cell rendering |
| video extension or content | the raw bytes as a `data` result, which the render treats as an image attempt |

That last row is a genuine oddity. A `.mp4` is refused by the extension list, but an `.mkv` renamed to `.dat` sniffs as binary text and is refused by the content rule; while a file the service decides is video returns raw bytes, and the tool's `data` branch tries the image mime sniff, fails, and returns the bytes unchanged, which the renderer then... renders as an image anyway, because `data` always goes through the two-part image builder. Whether that produces a broken image part or an error depends on the mime sniff, and `_H` will return `undefined` for bytes `file-type` does not recognize, so the image part gets `mimeType: undefined`. This is not a path worth reproducing.

### Q6 Empty input and empty result

One line each, because they differ.

- **Zero-byte file, no `offset` or `limit`.** Success with `totalLines: 0`, `readRange: undefined`, and the text `File is empty.` `isError` false. The path is still recorded in `readPaths`.
- **Zero-byte file with an `offset`.** `File is empty.` again, not an offset error. The service declines to apply a range when the content is empty (`resolveRequestedLineRange` returns `{applied:false}` for `contentEmpty`), so `rangeApplied` is false and the empty short circuit at step 14 fires before the window is ever computed. `offset: 500` on an empty file gives `File is empty.`
- **File containing exactly one newline, 1 byte.** Not empty. `totalLines` is 2, and the render produces two lines: `     1|` and `     2|`. Worth a test, because it is the boundary between the two mechanisms.
- **File containing only spaces, no newline.** Not empty by any definition here. 1 line, rendered `     1|   `.
- **A range that selects no lines.** Reachable when `limit` and `offset` combine so that `endIndex <= startIndex`, which the schema mostly prevents but a `limit` larger than the file with a `startIndex` at the last line makes the window a single line, not empty. A genuinely empty *window* requires `limit: 0`, which the schema rejects. If you relax that, note that `computeReadSlice(10, undefined, 0)` returns `{startIndex: 0, endIndex: 0, readRange: {startLine: 1, endLine: 0}}` and the content becomes `""` with `rangeApplied` semantics, which renders as one numbered blank line.
- **The service applies a range and the result is empty.** `tryApplyRequestedReadRange` returns `{content: "", rangeApplied: true}` when `start >= end`. Then step 14 is skipped (range applied), step 15 runs, `computeReadSlice` yields a window, and the content is `""`. The renderer formats `""` as `     1|` plus the head and tail notices for the skipped lines. So a valid-but-empty range is a numbered blank line, not `File is empty.`
- **`path: ""` or whitespace only.** The service returns `error` with `path is empty or only whitespace`, which the tool renders as `Error: path is empty or only whitespace`. Note that this is caught by the tool's `error` case at step 10, so the renderer's `Error: ` prefix applies.
- **Empty `offset` string.** Coerced by `lenientNumber`: `"".trim() === ""` returns the original string, which then fails the integer check, producing an `Invalid arguments:` failure naming `offset`.
- **Empty pattern, empty directory, zero matches.** Not applicable to Read. Those belong to Grep and Glob.

### Q7 Cost

**The tool never seeks.** The service reads the whole file into a `Buffer`, decodes it, and returns the string; the tool then splits and slices. So the memory cost of a read is roughly `fileSize` bytes for the buffer plus `2 × characterCount` bytes for the JavaScript string, plus the split array (one string object per line, each with its own header) while it exists.

Concretely, a 100 MB UTF-8 file of ASCII text costs about 100 MB for the buffer, 200 MB for the string, and then `split("\n")` allocates an array of short strings whose overhead is on the order of 20 to 40 bytes each. A 100 MB file of short lines is roughly 2 million lines, so the split alone is 40 to 80 MB of overhead, and the slice-then-join then allocates the result.

**The layering, in the order a large file meets it:**

1. The extension list. Free, no I/O.
2. `stat`. One syscall, no size gate.
3. The encoding and binary sniff, on 8192 bytes.
4. **On the terminal-file and agent-tool-output path only**, `stat.size > 2147483647` (2 GiB) throws `RangeError: File size (N) is greater than 2 GiB`, which lands in the generic `error` case and the model sees `Error: File size (N) is greater than 2 GiB`. This is `readFileUpTo`, called by `readTerminalFile` and by the UTF-8 fallback sampler. The ordinary text path does not go through it: `readTextStreamingWithDecoder` calls `createReadStream` with no size gate at all, and the whole-file `readText` has no size check either. So a 3 GB source file is not refused by Cursor; it is streamed and then fails on the string limit or the cap, depending on where it dies. Treat the 2 GiB figure as a bound on the terminal-file path, not on Read generally.
5. Streaming is forced above 8 MiB. Below it, streaming happens only if the `useStreamingRead` option says so, and that option defaults to true when absent.
6. The whole-file path truncates at 8 MiB: `content.substring(0, 8388608)`, with `truncated: true`.
7. The tool's 100,000-character cap.
8. The 32 MiB PDF text LRU.

**The worst case a hostile repository can force.** A 2 GiB file of one-byte-per-line text, read with no `offset` or `limit`:

- The 2 GiB check does **not** fire on this path, because the ordinary text read never calls `readFileUpTo`. So a 2 GiB file is not refused up front; it is streamed and then dies wherever the string limit or the cap catches it.
- A file just under 2 GiB is streamed with a 64 KiB chunk size and decoded into a JavaScript string of roughly 2 billion code units. That exceeds V8's maximum string length, so the decode throws rather than returning, and the throw lands in the generic error handler. The model sees an opaque error, not the cap message. The exact ceiling depends on the V8 build and is not something this bundle pins, so treat "roughly 2 GiB of UTF-8 text" as the danger zone rather than a specific number.
- Under the string limit, say a 400 MB file: the streaming path retains only the lines it needs, but the fallback buffer retains up to 8 MiB of characters, and `totalLines` is counted over the whole stream. So memory is bounded by the 8 MiB cut but time is proportional to the file size. A read takes as long as the decode takes.
- Many parallel reads in one turn: each holds its own buffer and string, and the per-turn lock only serializes same-path reads. A model that reads twenty 400 MB files in one response allocates twenty of them concurrently, because the lock keys differ and nothing else bounds concurrency.

**A specific asymmetry worth knowing.** The whole-file path's 8 MiB cut is measured in **characters**, and it cuts the *string*. So a 20 MiB UTF-8 file of 3-byte characters decodes to about 6.9 million characters and does not reach the 8 MiB character cut at all, meaning it is returned whole (and then refused by the 100,000-character cap). A 20 MiB file of ASCII reaches the 8 MiB cut at 8,388,608 characters and is silently truncated to 8 MiB, then refused by the cap. Either way the cap fires, so the truncation is invisible.

**What the truncation flag does.** Nothing. `truncated` is set by the service, carried on the `ReadSuccess` proto, and never read by the tool. It is not in the render, not in the message, not in the log. A silent 8 MiB truncation is therefore genuinely silent, which is the one place this tool can lie about completeness.

The read path has three separate mechanisms at this size, and they are easy to conflate. Two cut and one refuses, and only one of the three tells the model:

| Mechanism | Unit | Behaviour | Model learns? |
|---|---|---|---|
| 100,000-character cap | UTF-16 code units, after windowing | errors, empty content | yes, the two-line message |
| 8 MiB budget in the executor | UTF-16 code units | silently cuts the text, sets `truncated` | no, the flag is never rendered |
| `readWholeTextFile` refusal | bytes on disk, from `stat` | returns `undefined` rather than a partial read | depends on the caller's fallback |

The third one is the write path's reader, not the tool's, but it is on the same file and the same limit, so a report that describes read truncation has to name it or scope itself to the tool. `readWholeTextFile` is `stats.size > MAX_TEXT_SIZE` and it returns `void 0`; the caller in `getBufferForWrite` treats that as "no existing bytes", which is why `Write` can compute a diff against an empty original for a file that is merely large.

### Q8 Paths

**What the tool does.** Takes `path` or `target_file`, applies `lke`, and hands the result to the service.

```js
const cke=/(^\/(?:Users|home)\/[^/]+\/\.cursor\/)worktries(?=\/|$)/g;
function lke(e){return e.replace(cke,"$1worktrees")}
```

That regex is the entire tool-side normalization. It matches an absolute path under `/Users/<name>/.cursor/` or `/home/<name>/.cursor/` whose next segment is exactly `worktries`, and rewrites that segment to `worktrees`. Nothing else: no `~`, no relative resolution, no `..` collapse, no case folding, no UNC handling, no drive-letter handling.

**What the service does.** `resolvePath(filePath, workspacePath)`:

```js
function a(e){return e.replace(/^~(?=$|\/|\\)/,(0,s.homedir)())}
function l(e,t){
  const r=a(function(e){if(!e.startsWith("file://"))return e;try{return(0,i.fileURLToPath)(e)}catch{return e}}(e));
  return t&&!(0,o.isAbsolute)(r)?(0,o.resolve)(t,r):(0,o.resolve)(r)
}
```

So, in order: expand a leading `~`; convert a `file://` URL, falling back to the literal string if it is malformed; then, if a workspace path is known and the result is not absolute, resolve against the workspace; then `path.resolve` in all cases. `path.resolve` on Windows handles drive letters and normalizes `..`, and on all platforms collapses `.` segments. A malformed absolute path that resolves nowhere produces `ENOENT` and therefore `File not found`.

**`..` and paths outside the workspace.** Nothing in Read refuses either. `path.resolve` collapses `..`, the workspace-relative branch only runs for relative paths, and an absolute path that leaves the workspace is stat'd and read like any other. The permission service sees the resolved path and may block it, but the tool does not. That is a deliberate difference from the write path, which does have worktree and root-escape checks.

**Symlinks.** `stat`, not `lstat`, so a symlink is followed to its target. A symlink to a file reads the target and reports the **link path** as `path`, because the stat and the read both use the resolved-but-not-canonicalized path. A symlink to a directory yields `Path is a directory, not a file`. A broken symlink yields `ENOENT` and therefore `File not found`. A symlink loop yields `ELOOP`, which is not special-cased, so it lands in the generic error case and the model sees the raw operating-system message, e.g. `ELOOP: too many symbolic links encountered`.

Runvil diverges here in a way that is easy to miss and probably worth keeping. `RealBackend::read` resolves with `self.resolve(path)`, which follows a final-component link, but `RealBackend::stat` uses `symlink_metadata`, which does not. So on Runvil a `stat` of a symlink reports `FileType::Symlink` and a `read` of the same path returns the target's bytes. That is strictly more informative than Cursor, which cannot tell you a path is a link at all. The catch: a Read tool that pre-checks with `stat` and refuses anything that is not `FileType::File` would refuse every symlink, where Cursor would follow it. Decide deliberately and write it down, because the failure looks like a broken repository.

**Windows specifics.** `path.resolve` handles `C:\...` and `\\server\share\...` correctly. Case sensitivity is whatever the filesystem does, and nothing in Read folds case, so `C:\Work\Foo.ts` and `c:\work\foo.ts` are two lock keys and two `readPaths` entries pointing at one file.

**When the path is normalized, relative to the lock and the cache.** This is the part to get right.

| Use | The string used | Normalized? |
|---|---|---|
| the per-file lock key | the raw argument | no |
| the binary-extension test | the `lke` result | typo repair only |
| the argument echoed in an `invalidFile` message | the `lke` result | typo repair only |
| the path sent to the service | the `lke` result | typo repair only |
| the path the service resolves against the workspace | fully resolved | yes, in the service |
| the `path` field on the result, and the key in `readPaths` | the service-resolved path | yes |
| the path in the image text part | the service-resolved path | yes |

So the same file can be known by three different strings within one call. The lock uses one, the read uses another, and the bookkeeping uses a third.

### Q9 Concurrency

**What is locked.** One key per path, in a `Set` on a lock manager, plus an exclusive-lock flag and a FIFO wait queue.

**For how long.** From step 4 of the algorithm until the tool returns, across every throw, because the handle is registered with a disposal scope rather than released in a `finally` that a reimplementation might forget.

**Keyed on what.** The raw argument string, before `lke` and before resolution. So `./foo.ts`, `foo.ts`, and `C:\work\foo.ts` are three keys for one file.

**What race that closes.** Two calls in one turn that name the same file identically, one of which mutates. The canonical case is a model that emits a `Write` and a `Read` of the same path in one response: without the lock the read can land between the truncate and the write-back and observe a partial file.

**Which races it leaves open.** Three, in increasing order of how much they matter.

1. **Same file, different spelling.** The lock does not see them as the same key. A model that writes `src/foo.ts` and reads `./src/foo.ts` gets no serialization.
2. **Across agents.** The manager is a per-turn object, allocated in `runTurnLoop` and in `setupStep`. Two chat tabs, two background agents, or a parent plus a subagent each have their own, so nothing serializes between them. The spec records zero occurrences of `this.fileOperationLockManager=` in the bundle, confirming there is no singleton.
3. **Against anything outside the agent.** An editor save, a formatter, a git checkout, a build tool. The lock is invisible to all of them.

**Is there a read-modify-write window.** For Read, no: the tool reads once and returns. But the lock is held across a potentially very long operation (the read of a large file), so a Write queued behind a Read of a 400 MB file waits for the whole decode.

**Is there a TOCTOU check.** Not in Read. Read stat's then reads, and if the file changed in between it returns whatever it read. The write path has a sha256 check; read does not, and does not need one, because a read has no invariant to violate.

**The retry's locking, precisely.** On a miss the tool disposes the lock, calls `waitForEarlierCalls`, and on success re-acquires and retries. Between the dispose and the re-acquire there is a window in which another call can take the lock, which is fine because the retry then queues behind it. The 120,000 ms figure is the *wait for earlier calls*, not a lock timeout; lock acquisition itself is unbounded and abort-only.

**The tool-name wait set**, verbatim:

```js
new Set(["ApplyPatch","AwaitShell","CopyToBox","EditNotebook","MultiEdit","MultiStrReplace","Shell","StrReplace","Write","edit_notebook","run_terminal_cmd","search_replace","write"])
```

Twelve names, mixing PascalCase and snake_case, because the set has to match whatever wire name each tool is registered under on each branch. A reimplementation with one naming scheme needs a shorter set, and needs it to cover any tool that can create a file, including a shell.

**What to build instead.** In Runvil, `Access` already gates and claims: `read` takes `Claims::claim_read`, `write` takes `claim_write`, and two overlapping claims conflict when at least one is a write and they are concurrent. That is a strictly better primitive than Cursor's string-keyed set, and the spelling race is already closed, because `gate` does two things in a deliberate order:

```rust
let path = canonicalize(&self.root, path)?;
self.check_policy(op, &path)
```

The doc comment says why this matters: "Canonicalizes at receipt and consults the policy - in that order, so a denied operation never registers a claim and every claim key is the canonical path." `claim_key` then case-folds below a mount whose filesystem folds case. So on Runvil, `./foo.ts` and `foo.ts` are one claim key, which is exactly what Cursor gets wrong. The one thing to check is that the canonicalization is the same for `read` and `write` - it is, because both go through `gate` - and that a path which does not exist yet still canonicalizes to something two callers agree on, which is the case where a Read and a Write legitimately race over a file being created.

**A specific design consequence.** Because Runvil's claim is taken inside `Access::read` and released when the call returns, there is no way for the tool to hold a claim across a *sequence* of operations. Cursor needs that for its retry, which releases the lock, waits, and re-takes it. In Runvil the natural equivalent is to call `read`, and on a miss call `read` again after the wait, since each call takes its own claim. That is simpler and correct, and it means the "release before waiting" trap does not exist here. Do not add a manual hold to reproduce it.

### Q10 Fidelity

Read does not write anything, so the honest answer is about what it *loses on the way out*, not what it changes on disk. But there is one place where a read/write round trip matters, and it is worth being explicit because `StrReplace` is built on Read.

| Property | Preserved through Read? | Where it is lost |
|---|---|---|
| bytes | yes, in the buffer | - |
| encoding | detected, then decoded to a JavaScript string | no longer bytes after decoding |
| UTF-8 BOM | **stripped** from the text | `stripUtf8Bom`; remembered as `utf8bom` so a write can restore it |
| UTF-16 BOM | consumed by the decoder | not in the text |
| line endings | **conditionally rewritten** | `\r\n` to `\n` when the 5% vote passes |
| bare `\r` | preserved on the whole-file path, dropped in PDF text | - |
| trailing newline | preserved | - |
| file mode | not read, not exposed | - |
| ownership | not read, not exposed | - |
| mtime | read into `readFileState` on the Claude side; on Cursor's side, not surfaced | - |
| invalid UTF-8 bytes | **replaced by U+FFFD** on the lossy path, or transcoded on the latin1 path | - |

**So a caller that reads a file and writes it back unchanged does not preserve the bytes** in three cases: CRLF is rewritten to LF, a UTF-8 BOM is dropped (and would be restored only by the write path's own BOM logic, not by the text), and invalid UTF-8 bytes become U+FFFD, which is irreversible.

This is why Cursor's Write does its own read and its own line-ending merge rather than reusing what the model was shown. If Runvil's `StrReplace` operates on the model's transcript text, it inherits all three losses. If it operates on fresh bytes from `Access::read`, it inherits none. **It must be the latter**, and it must anchor on bytes the model never saw verbatim, which is a real design tension: the model's `old_string` came from a normalized view.

The practical resolution, and the one Cursor effectively uses, is that `\r\n` normalization is applied to *both* sides before matching. That is a StrReplace concern, but it originates here, and a Read that normalizes must be paired with an edit that normalizes identically.

### Q11 Determinism

**Is output order defined?** Yes, trivially: file content in file order, lines in line order. There is no sorting anywhere in the read path.

**Would two identical runs produce byte-identical output?** For one file, yes, with three exceptions.

1. **The image branch depends on a model capability.** `XK` / `boundInlineImageForModel` may re-encode or resize the bytes depending on what the model can accept, so the base64 payload differs by model even for the same file.
2. **PDF text depends on an external worker.** The conversion runs in a Piscina pool over a separate chunk, and the cache is keyed by sha256 of the source bytes. Same bytes, same text, but the extraction library is a dependency whose version you do not control from the tool.
3. **The trailers are not deterministic in content.** The cursor-rule and skill trailers depend on which rules and skills the host supplied, and they are appended to the tool text. Two runs with different rule sets produce different text for the same read.

Everything else is a pure function of the file bytes, the arguments, and the model family.

**One determinism trap for a reimplementation.** `countNewlines` and `split("\n")` must agree, or the same file reports a different `totalLines` depending on whether an `offset` was supplied. They do agree in Cursor, and the agreement is not accidental: both count `\n` and both treat the segment after the final `\n` as a line. A port that uses `lines()` from `str` and a separate newline count will drift on the trailing-newline case, because `"a\n".lines()` yields one item and the counter yields two.

### Q12 Truncation and omission

Four mechanisms. Two are visible, one is visible but wrong, one is invisible.

**1. The character cap: visible, and it errors.** Two lines, `isError: true`, empty content. The model is told the exact byte size and told to use `offset` and `limit`. It is *not* told where to resume, which is the weakness.

**2. The omitted-line notices: visible, and they explain.** `... 40 lines not shown ...` above and `... 9 lines not shown ...` below, with correct singular. They are computed from the window against `totalLines`, so they are accurate for a normal partial read.

**3. The tail notice can be wrong after a silent truncation.** The tail is `totalLineNumbersInFile - (startLineNumber + count(content) - 1)`. If the content was cut by the 8 MiB rule, `count(content)` is smaller than the range the caller asked for, so the notice overstates how many lines remain *after the shown content* while understating how many were dropped *inside* the range. This is reachable: an 8 MiB ASCII file read with `offset: 1` and a `limit` of 2,000,000 gives content that was cut to 8,388,608 characters, and the tail notice is computed from the cut content.

**4. The service's `truncated` flag: invisible.** Set, carried, never read. This is the one place the tool lies about completeness, and it lies by omission rather than by a false statement.

**Plus a fifth thing that is not truncation but is read as it.** The `exceededLimit` record has `isEmpty: false` and empty content. A renderer that checks `isEmpty` before `exceededLimit` would show `File is empty.` for a 200 KB file. Cursor checks `isEmpty` first and gets away with it only because the cap path sets `isEmpty: false` explicitly. Reversing the two checks in your render is a one-line change that produces a catastrophic lie, so put the cap check first and add a test.

**Does the tool lie anywhere else?** Two small places.

- The cap message says `100000 characters`. If the content is CJK, that is true of the code-unit count and false of the byte count, and the message also states the byte count beside it, which invites the model to compare them and conclude the tool is inconsistent.
- The `truncated` flag's existence means the *service* knows it cut content while the tool does not say so. Any read between 8 MiB and 100,000 characters after the cut reports a complete-looking result.

## Failure matrix

Every failure `Read` can produce, in the order the checks run. `Result case` is the record case, not the rendered text, and they differ: `exceededLimit` is a success record rendered as an error, and a directory is an `error` record whose text is shaped like the `notFile` case.

| # | Condition | Result case | Exact model-visible text | Layer |
|---|---|---|---|---|
| 1 | argument fails schema validation | `error` | `Invalid arguments:` then newline then `<key>: <message>` | validation, before the tool body |
| 2 | `limit: 0` | `error` | `Limit must be >= 1.` inside the envelope above | validation |
| 3 | `offset: -1` with negative offsets off | `error` | `Offset must be >= 1.` inside the envelope | validation |
| 4 | `line_range` with `end < start` | `error` | `line_range must be [start_line, end_line] with 1 <= start_line <= end_line.` inside the envelope | validation |
| 5 | path suffix in the binary list | `error` | `Cannot read binary files of type zip` | tool, before I/O |
| 6 | path is `""` or whitespace | `error` | `Error: path is empty or only whitespace` | capability service |
| 7 | path does not exist | `fileNotFound` | `File not found` | service, `ENOENT` |
| 8 | path is a directory | `error` (from `invalidFile`) | `Path is a directory, not a file: C:\work\proj\src` | service |
| 9 | path is a fifo, socket, or device | `error` (from `invalidFile`) | `Path is neither a file nor a directory: <path>` | service |
| 10 | read permission denied by policy, user declines | `rejected` | `Read operation rejected` | service, permission prompt |
| 11 | read permission denied by policy, denied outright | `permissionDenied` | `Permission denied` | service |
| 12 | `stat` or open returns `EACCES` or `EPERM` | `permissionDenied` | `Permission denied` | service |
| 13 | content sniffs as binary | `error` (from `invalidFile`) | `File content is binary data, not text: <path>` | service |
| 14 | image bytes fail to decode | `error` (from `invalidFile`) | `File is not a valid image: its data could not be decoded (<reason>). A truncated or hand-copied file does this; write it again from the original bytes with code.: <path>` | service |
| 15 | `invalidFile` with no reason at all | `error` | `Path is not a valid file to read: <path>` | tool |
| 16 | offset at or past EOF | `error` | `Offset 11 is beyond file length (10 lines)` | tool, in the window function |
| 17 | window content over 100000 characters | `success` | the two-line cap message; `isError` **true** | tool, after windowing |
| 18 | file over 2 GiB on the terminal-file path | `error` | `Error: File size (2147483648) is greater than 2 GiB` | service, `readFileUpTo` only |
| 19 | symlink loop | `error` | `Error: ELOOP: too many symbolic links encountered` | service, unhandled |
| 20 | PDF worker failed | `error` | `Error: <worker message>` | tool, PDF branch |
| 21 | output proto is `data` but not image and not PDF | `success` | the raw bytes as a two-part image content array with `mimeType: undefined` | tool, render |
| 22 | result case unrecognized | `error` | `Unknown error` | tool |
| 23 | `render` receives no result at all | (none) | `Unknown error` with `isError` true | render |
| 24 | success with no `output` field | (none) | `Unknown error: no output` with `isError` true | render |
| 25 | output case is `void 0` | (none) | `Unknown error: no output case` with `isError` true | render |
| 26 | output case unrecognized | (none) | throws `Unhandled output case: <case>`; crashes the render | render |
| 27 | blob output with no blob store | (none) | throws `Cannot hydrate blobs for tool Read: no blob store available`; crashes | render |
| 28 | blob present but missing from the store | (none) | throws `Failed to hydrate blob <hex> for tool Read`; crashes | render |

**The failures that look like successes.** This is the part a reimplementation gets wrong, so each one gets a line.

- **Over the cap.** `success` case, `exceededLimit: true`, empty content, rendered with `isError: true`. If you check the case tag you conclude success; if you check the flag you conclude error.
- **Empty file.** `success`, `isEmpty: true`, `isError: false`. Genuinely a success, and the path is recorded, so a read-before-write gate would consider it read.
- **A window that selects nothing.** `success` with content `""` and `rangeApplied`, rendering as one numbered blank line. Neither empty nor an error.
- **A silent 8 MiB cut.** `success`, content looks complete, `truncated` true and never surfaced.
- **A blob-backed result.** `success` with `contentBlobId` rather than content, and a `readRange` of `1..totalLines` regardless of what was requested. Renders identically to an inline result after hydration.

**Two rows that are not failures and should not be treated as such.** A read of a file that does not exist is deliberately allowed by the description (`It is okay to read a file that does not exist; an error will be returned`), so the error is the designed outcome, not a surprise. And a read of a directory is refused rather than turned into a listing, which means the model must know that `LS` exists; the description does not say so.

## Traps

### The pipe is not `cat -n`, and nothing in the shipped build says it is

**What goes wrong.** A reimplementer reads the Runvil snapshot in `plugin-std\schemas\read.json`, sees two sentences about line numbers, and builds to them: `Any lines longer than 2000 characters will be truncated` and `Results are returned using cat -n format, with line numbers starting at 1`. Neither is true of Cursor's Read. There is no per-line truncation at all, and the format is six-wide plus a pipe. So the implementation emits `1\tconst x = 1;` or truncates lines at 2000 characters, and every line-numbered read is wrong in a way that looks plausible.

**Why it is easy to get wrong.** The two sentences read like specifications, they are in the repository the implementation is being written in, and `cat -n` appears in Cursor's *older* published prompts and in several community reimplementations. It looks corroborated. It is not corroborated by anything shipped: the string `cat -n` does not occur anywhere in the 9,814,287-byte 3.27.12 bundle, and it does not occur in the extracted Claude Code `read.js` either.

That second negative is weaker than the first, so state it carefully. The parent spec asserts at `agent-file-tools.md` line 4155 and again at 4184 that both descriptions claim `cat -n`. For Cursor the shipped text refutes it outright: the `latest` description says `using following format: LINE_NUMBER|LINE_CONTENT`. For Claude Code the extracted prompt interpolates `${z2t}` for a line count and ends with `${nr}` or `${er}`, both identifiers whose text lives in modules outside the extracted region, so the tail of that prompt was not captured and the claim can be neither confirmed nor refuted from this corpus. The honest statement is that Cursor's claim is false and Claude Code's is unverified, which is not what the spec says about either.

**The test that catches it.** Read a three-line file and check three bytes and one absence.

```
assert_eq!(&out.as_bytes()[0..6], b"     1");
assert_eq!(out.as_bytes()[6], b'|');
assert!(!out.contains('\t'));
assert!(!out.contains("cat -n"));
```

### The cap fires after windowing, and this is the escape hatch

**What goes wrong.** A reimplementation checks total file size before slicing, so a 200 KB file cannot be read at all, and the model's only recourse - read a window - is broken by the very check that was supposed to protect it.

**Why it is easy to get wrong.** Checking length first is the obvious order, and it is the order every streaming implementation uses. Cursor's order is unusual and is the entire reason the tool is usable on large files.

**The test that catches it.** Two calls on the same 200,000-character file: one whole, which must fail with the cap message, and one with `offset: 3, limit: 2`, which must succeed and return exactly two numbered lines. If the second fails, the order is wrong.

### Emoji cost two and CJK costs one

**What goes wrong.** A Rust port counts bytes. A file of 100,000 CJK characters (300,000 bytes) is refused by the port and accepted by Cursor. A file of 100,000 emoji (400,000 bytes, 200,000 code units) is refused by both, but for different reasons and at different thresholds.

**Why it is easy to get wrong.** `s.len()` is the natural thing to write, and it is right for ASCII, which is what the test fixtures use.

**The test that catches it.** Three fixtures at three encodings, each just under and just over the cap: 100,000 ASCII characters, 100,000 CJK characters, and 50,000 emoji. Record which pass. Then decide whether your answer is Cursor's or your own, and write the decision down.

### `offset: 0` passes validation and `limit: 0` does not

**What goes wrong.** A hand-written validator rejects `offset: 0` with `Offset must be >= 1.` - which is what the message says - and then a model that counts from zero fails on every call. Or the validator accepts `limit: 0`, and `computeReadSlice` produces a zero-length window with `readRange: {startLine: 1, endLine: 0}`, which the notice logic then renders with a nonsensical tail count.

**Why it is easy to get wrong.** The refine message and the actual rule disagree, so reading the message gives the wrong answer. Only reading the refine does.

**The test that catches it.** `offset: 0` on a 3-line file must return lines 1 through 3 with no error. `limit: 0` must fail with `Limit must be >= 1.`

### A negative offset does not throw when it overshoots

**What goes wrong.** A reimplementation bounds-checks the raw offset before computing the start index, so `offset: -100` on a 10-line file throws. Cursor clamps to 0 and returns the whole file, because the clamp is inside the index computation and the bounds check runs on the *clamped* index.

**Why it is easy to get wrong.** It reads as a bug, so a careful implementer "fixes" it.

**The test that catches it.** `offset: -100` on a 10-line file returns 10 numbered lines and no error. `offset: 11` on the same file throws `Offset 11 is beyond file length (10 lines)`. Both in one test, so the asymmetry is visible.

### One newline is not an empty file

**What goes wrong.** A reimplementation treats whitespace-only content as empty, or trims before testing. A file containing one `\n` then renders as `File is empty.` instead of two numbered blank lines.

**Why it is easy to get wrong.** The natural emptiness test is `content.trim().is_empty()`, which is true for `"\n"`. Cursor's test is `content === ""` character for character.

**The test that catches it.** A one-byte file containing `\n` must render as exactly two lines, `     1|` and `     2|`, with `isError` false, and must not say `File is empty.`

### Emptiness is a flag on the record, not a property of the content

**What goes wrong.** A renderer that computes `content.is_empty()` instead of reading the `isEmpty` field reports `File is empty.` for the over-cap case, because that case has empty content by design.

**Why it is easy to get wrong.** The over-cap record really does carry `content: ""`, and the natural renderer checks the content.

**The test that catches it.** A 100,001-character file must produce the cap message with `isError` true, and must never produce `File is empty.`

### The lock key is not the path

**What goes wrong.** Two calls in one turn, one reading `src/foo.ts` and one writing `./src/foo.ts`, do not serialize, and the read can observe a half-written file.

**Why it is easy to get wrong.** Keying a lock on the path you are about to use is the obvious implementation, and it is right until a model spells a path two ways. Cursor does exactly this and is wrong; a reimplementation that copies the call site reproduces the bug.

**The test that catches it.** In one turn, issue a write and a read of the same file spelled differently, with the write large enough to take measurable time. The read must not see a partial file. With string keying it will. **On Runvil this test already passes** because `gate` canonicalizes before claiming; run it anyway, because it is the only observable proof that the canonicalization is doing its job.

### Releasing the lock after the wait instead of before deadlocks

**What goes wrong.** The retry path holds the lock while waiting for an earlier writer that needs the same lock, so neither can proceed. The symptom is a turn that never finishes, not an error.

**Why it is easy to get wrong.** The disposal is in a scope guard, so it is easy to reason "the lock is held for the whole tool, including the retry", which is exactly the bug.

**The test that catches it.** A write of a 10 MB file and a read of the same path, issued in one turn with the read second, must complete. If the read hangs indefinitely, the release is in the wrong place.

### CRLF is decided by a 5% vote, so `\r` can reach the model

**What goes wrong.** A reimplementation normalizes `\r\n` unconditionally, so a mostly-LF file with two Windows lines is shown as LF, and when the edit tool writes the model's `old_string` back it silently converts those two lines. Or it does not normalize at all, and a CRLF file shows a `\r` at the end of every line, which the model then includes in its `old_string` and which fails to match after the file is re-read as LF.

**Why it is easy to get wrong.** Both unconditional choices look correct on the fixture you happen to test with.

**The test that catches it.** One file with 95 LF lines and 5 CRLF lines, and one file with 90 LF and 10 CRLF. Both must be handled, and the *same* rule must apply in the read formatter and the edit matcher, or an edit that the model believes is exact will fail.

### The tail notice counts from the content, not from the request

**What goes wrong.** A reimplementation computes the tail as `totalLines - requestedEndLine`, which is right for a normal partial read and wrong after any upstream truncation, because the content is shorter than the request.

**Why it is easy to get wrong.** The two formulas agree in every case that is easy to construct.

**The test that catches it.** Force a content truncation (an 8 MiB cut, or a stub service that returns fewer lines than requested) and check that the head and tail notices still add up to the number of lines not shown, counting from the content's own length.

### `1 lines` in the offset error

**What goes wrong.** A reimplementation adds a singular form to the offset message, producing `Offset 2 is beyond file length (1 line)` where Cursor says `(1 lines)`.

**Why it is easy to get wrong.** The formatter next door pluralizes correctly (`1 line not shown`), so assuming the same care here is natural.

**The test that catches it.** `offset: 2` on a one-line file must produce `Offset 2 is beyond file length (1 lines)`.

### `include_line_numbers` off means no numbers, not default numbers

**What goes wrong.** A reimplementation treats the argument as an override that defaults to the model family's setting, so omitting it yields numbers. Cursor's expression is `result.includeLineNumbers ?? (!enableLineNumbersArg && familyDefault)`, so with the argument feature on and the argument omitted, the result has `undefined` and the fallback `!true && ...` is false: **no numbers**.

**Why it is easy to get wrong.** The `??` reads like a default, and it is a default only in the branch where the argument does not exist.

**The test that catches it.** With the argument enabled, a call with no `include_line_numbers` must return unprefixed text, and a call with `include_line_numbers: true` must return prefixed text.

### The sparse style silently drops the pipe

**What goes wrong.** A reimplementation applies sparse numbering by padding unnumbered lines to column 8 so the text aligns, which is not what Cursor does and which changes the bytes the model sees on most lines.

**Why it is easy to get wrong.** Alignment looks like an improvement, and the difference is invisible in a screenshot.

**The test that catches it.** Sparse mode with modulus 10 over 12 lines: line 9 must be exactly `line 9` with no leading spaces, and line 10 must be exactly `    10|line 10`.

### A directory is an error, not a listing

**What goes wrong.** A reasonable reimplementation lists the directory, because that is the helpful thing to do. Cursor refuses with `Path is a directory, not a file: <path>`, so a model that asks a Read tool for a directory gets nothing.

**Why it is easy to get wrong.** Listing is strictly more useful, so it looks like a fix.

**The test that catches it.** Reading a directory must fail with the exact message, and must not return any entries. If you decide to list instead, change the description, because the description says the tool reads files.

### `fileSize` is bytes while the cap counts code units

**What goes wrong.** The cap message reports the file's size in bytes beside a cap expressed in characters. For CJK text those numbers differ by a factor of three, and a model that reasons about them concludes the tool is inconsistent, or that it has room to retry.

**Why it is easy to get wrong.** Both numbers look like "size", so wiring them from the same source looks right.

**The test that catches it.** Read a 100,001-character all-CJK file and check the message: the cap says 100000 characters and the size says 300003 bytes. Both must be true statements about their own unit.

## How three harnesses do this

### Claude Code

**Source and name.** `C:\Users\Vinnie\cursor\cabinet\_scratch\claude-code\tools\read.js`, extracted from `claude.exe` 2.1.311, latin1 byte offsets preserved. The schema builder is `Akr` at 257768503, the registration is at 257776684, the execute body is `$kr` at 257782502. The tool is named `Read`. (source)

**Parameters.**

| Name | Type | Optional | Description, verbatim |
|---|---|---|---|
| `file_path` | string | no | `The absolute path to the file to read` |
| `offset` | integer, nonnegative | yes | `The line number to start reading from. Only provide if the file is too large to read at once` |
| `limit` | integer, positive | yes | `The number of lines to read. Only provide if the file is too large to read at once.` |
| `pages` | string | yes | `Page range for PDF files (e.g., "1-5", "3", "10-20"). Only applicable to PDF files. Maximum 20 pages per request.` |
| `allow_large` | boolean | yes | `Set to true to read a text file, or a line range of one, that is over the usual size limits, up to what still fits in your context. Only use this when you genuinely need all of it or the user asked for the whole file; otherwise read it in parts with offset and limit, or search it.` |

Note `offset` is `.nonnegative()`, so zero passes here too, and `limit` is `.positive()`, so zero fails - the same asymmetry as Cursor, reached by a different route.

**Caps.**

| Limit | Value | Unit |
|---|---|---|
| per-read character budget `ykr` | 150000 | characters |
| `maxResultSizeChars` on the registration | `1/0`, i.e. Infinity | characters; Read is uncapped at the registration level |
| default line count `z2t` | **unresolved** | the prompt interpolates `${z2t}` and the constant lives outside the extracted region |
| PDF pages per request `a5` | 20 | pages |
| PDF pages per turn `_kr` | 300 | pages |
| read chunk size `zkr` | 65536 | bytes |
| notebook size cap `X1` | 104857600 | bytes |
| readFileState LRU `pu` | 26214400 | bytes |
| readFileState retained-content threshold `gu` | 4096 | bytes |
| staleness timestamp window `cR` | 5000 | milliseconds |

The `z2t` row is the one gap in this section. The prompt says `By default, it reads up to ${z2t} lines starting from the beginning of the file`, and `z2t` is not defined anywhere in the 49 KB extracted region, so the number is not visible here. The parent spec's comparison table gives `MAX_LINES_TO_READ = 2000` for Claude Code; that identifier does not appear in this corpus either, so the 2000 figure is not independently confirmed by the artifacts on this machine. Treat it as the spec's claim, not as verified.

**Output shape.** Unpadded line number, a tab, then the line text with any trailing `\r` stripped:

```js
function BZr(e,n,r){let s=e.endsWith("\r")?e.slice(0,-1):e;return`${n}${r}${s}`}
```

with the separator chosen by:

```js
let s = tabAwareSeparator && (content.startsWith("\t") || content.includes("\n\t")) ? ":" : "\t";
```

So a tab-indented file gets `12:content` instead of `12\tcontent`, specifically so the line-number prefix is visually distinguishable from the content's own indentation. `tabAwareSeparator` is `rot()`, a feature flag whose runtime value a static read cannot see. A three-line file reads:

```
1	alpha
2	beta
3	gamma
```

**The one idea worth taking: the tab-aware separator.** It solves a real ambiguity at zero cost, and it is the kind of detail that only shows up after watching models mis-parse a prefix. If you ever consider a tab separator, take this with it.

The `nr` and `er` tails of those two prompt variants are bare identifiers whose text lives in a module outside the extracted region, so the last bullet of the Claude Code prompt is not visible in this corpus. That is why the `cat -n` claim attributed to this description by the parent spec cannot be confirmed or refuted here, only noted as absent from the part that was captured.

**The one idea worth rejecting: the read-state gate.** Claude Code requires the model to have read a file before writing it, and refuses otherwise with `File has not been read yet. Read it first before writing to it.` That gate lives in a separate LRU (`Qs`) keyed by normalized path, holding `contentHash`, `readBaseline`, `contentLength`, `offset`, `limit`, `isPartialView`, `timestamp`, and `contentNotInModelContext`, with a 5-second staleness window. It is a large amount of machinery, it depends on model cooperation, and it produces a whole class of "please read it again" round trips. Cursor skips it and re-reads inside each tool instead, which handles external modification without cooperation. (source)

### VS Code Copilot

**Source and name.** `C:\Users\Vinnie\src\vscode\extensions\copilot\src\extension\tools\node\readFileTool.tsx`, and the tool is `read_file`, from `common\toolNames.ts`. Tests are `node\test\readFile.spec.tsx`, 58 KB, which pins real behavior and is unusually informative. (source)

**The v1/v2 split.** Two parameter shapes are live at once, discriminated by whether `startLine` is present:

| Shape | Parameters | Status |
|---|---|---|
| v1 | `filePath`, `startLine`, `endLine` | legacy, 1-based, both required |
| v2 | `filePath`, `offset?`, `limit?` | current |

The v2 description, verbatim:

```
Read the contents of a file. Line numbers are 1-indexed. This tool will truncate its output at 2000 lines and may be called repeatedly with offset and limit parameters to read larger files in chunks. Binary files use offset/limit as byte offsets.
```

That last clause is a genuine design decision worth noting: **the same two parameters switch units depending on whether the file is binary.** A binary read treats `offset` and `limit` as byte offsets into a hexdump, not line numbers.

**Caps.**

| Limit | Value | Unit |
|---|---|---|
| `MAX_LINES_PER_READ` | 2000 | lines |
| `MAX_LINE_LENGTH` | 2000 | characters per line |
| effective limit | `clamp(limit, 1, 1999)` | lines |

The clamp to `MAX_LINES_PER_READ - 1` is a real off-by-one: a caller that asks for 2000 lines gets 1999, so the tool reserves one line for the truncation trailer.

**Output shape.** The content, optionally wrapped in a code fence with a header line, plus trailers:

```
File: `src/greeter.ts`. Lines 1 to 2000 (3000 lines total): 
```

Per-line truncation is surrogate-aware:

```ts
if (line.length > MAX_LINE_LENGTH) {
    let end = MAX_LINE_LENGTH;
    if (isHighSurrogate(line.charCodeAt(end - 1)) && isLowSurrogate(line.charCodeAt(end))) end--;
    return line.slice(0, end) + ' [truncated]';
}
```

and the tests pin it exactly: a line of 1999 `x` followed by U+1F6E1 followed by `tail` truncates to `'x'.repeat(1999) + ' [truncated]'`, not 2000 characters, so an emoji is never cut in half. The trailer is:

```
[One or more long lines were truncated at 2000 characters]
```

and then, when the line range was cut:

```
[File content truncated at line 2000. Use read_file with offset/limit parameters to view more.]
```

**Empty and whitespace files.** Distinct messages, both parenthesized:

```
(The file `/workspace/empty.ts` exists, but is empty)
```

```
(The file `/workspace/whitespace.ts` exists, but contains only whitespace)
```

An offset past the end throws rather than returning an empty result:

```
Invalid offset 535: file only has 5 lines. Line numbers are 1-indexed.
```

**Binary handling.** `hexdumpIfBinary` reads the raw bytes and checks for null bytes, and a binary file produces a hexdump rather than a refusal, with `offset`/`limit` reinterpreted as byte offsets. Text files with no null bytes are unaffected, which the tests pin.

**The one idea worth taking: the surrogate-safe per-line truncation plus its own trailer.** Two independent caps (lines and per-line length), each with its own message, so the model can tell which limit it hit and what to do about it. That is strictly better than one cap doing both jobs.

**The one idea worth rejecting: reinterpreting `offset`/`limit` as bytes for binary files.** One parameter, two units, chosen by inspecting the content. A model that does not know whether a file is binary cannot predict what `limit: 100` will return, and the error messages do not say which unit was in play. If you want byte paging for binary, give it its own parameters.

### Zed

**Source and name.** `C:\Users\Vinnie\src\zed\crates\agent\src\tools\read_file_tool.rs`, 77 KB. `ReadFileToolInput` at line 161, `const NAME: &'static str = "read_file"` at line 212. (source)

**Parameters.**

| Name | Type | Optional | Notes |
|---|---|---|---|
| `path` | `String` | no | relative to a project root, and the doc comment says it "should never be absolute" |
| `start_line` | `Option<u32>` | yes | 1-based, `#[serde(default)]` |
| `end_line` | `Option<u32>` | yes | 1-based, **inclusive**, `#[serde(default)]` |

Note `u32`: a negative value is a deserialization failure, so there is no negative-offset case at all.

**Caps.** There is no line cap and no character cap. Instead, the *shape* of the response changes with size: `outline::get_buffer_content_or_outline` returns either the content or a tree-sitter symbol outline, and the outline is a success.

**Output shape.** Right-aligned in a 6-character field, then a **tab**, then the line text:

```rust
let _ = write!(output, "{line_number:>6}\t");
```

So Zed is the closest of the three to real `cat -n`, and its own doc comment says so:

```rust
/// Prefixes each line of `text` with its line number in `cat -n` format:
/// the line number is right-aligned in a 6-character field, followed by a
/// single tab, followed by the line's original content (including its
/// trailing newline if present).
```

The output is then wrapped in a fenced code block tagged with the requested path so the UI can syntax-highlight it, and the outline response deliberately omits the tag so the highlighter does not run tree-sitter over pseudo-code.

**Path restriction.** The only path allowed outside the project is `~/.agents/skills` (line 159), and symlinks are handled explicitly: `resolve_project_path` returns a `symlink_canonical_target`, and `authorize_symlink_access` asks the user before following a link out of the project. Tests pin all three outcomes: escape plus approval succeeds, escape plus denial fails, and an escape into a private path needs no authorization at all.

**The outline fallback.** When the file is too large, the model gets:

```
SUCCESS: File outline retrieved. This file is too large to read all at once, so the outline below shows the file's structure with line numbers.

IMPORTANT: Do NOT retry this call without line numbers - you will get the same outline.
Instead, use the line numbers below to read specific sections by calling this tool again with start_line and end_line parameters.

{outline}

NEXT STEPS: To read a specific symbol's implementation, call read_file with the same path plus start_line and end_line from the outline above.
For example, to read a function shown as [L100-150], use start_line: 100 and end_line: 150.
```

**The one idea worth taking: the outline as a success.** It converts the worst case (a 50,000-line file the model cannot usefully read) into a useful structural summary, and it tells the model in advance not to retry. A truncation trailer says "there is more"; an outline says "here is the shape, and here is how to get the parts you want". For a code-reading agent that is a better answer than either a cap or a truncation.

**The one idea worth rejecting: making the outline unconditional on size.** Zed does not let the model ask for the whole file once the outline threshold is crossed - the description even says retrying without line numbers gives the same outline. There is no escape hatch, so a model that genuinely needs the whole file (to copy it, or to count something) cannot get it. Cursor's cap plus `offset`/`limit` at least offers a path, however tedious.

### Across all four

| Dimension | Cursor `Read` | Claude Code `Read` | Copilot `read_file` | Zed `read_file` |
|---|---|---|---|---|
| Path parameter | `path` or `target_file` | `file_path` | `filePath` | `path` |
| Range parameters | `offset` + `limit`, or `line_range` | `offset` + `limit`, or `pages` | `offset` + `limit` (v2), `startLine` + `endLine` (v1) | `start_line` + `end_line`, inclusive |
| Index base | 1, and 0 is silently coerced to 1 | 1, and 0 passes validation | 1, and 0 clamps to 1 | 1, and 0 clamps to 1 |
| Negative offsets | yes on some branches | no, `.nonnegative()` | no | no, `u32` |
| Paging | `offset`/`limit`, no cursor | `offset`/`limit`, no cursor | `offset`/`limit`, no cursor | `start_line`/`end_line` |
| Line cap | none | none | 2000 lines | none |
| Character cap | 100000, errors | 150000 | none | none |
| Per-line cap | none | none | 2000 characters, surrogate safe | none |
| Cap unit | UTF-16 code units | characters | lines, and characters per line | n/a |
| Cap behavior | `exceededLimit`, rendered as an error | auto-paginate with `truncatedByTokenCap` | truncate with a trailer | switch to an outline |
| Line number format | `     1\|text` | `1\ttext`, or `1:text` when tab-led | none in the text; a fence header gives the range | `     1\ttext` |
| Line numbers optional | yes, via `include_line_numbers` | no | no | no |
| Encoding detection | yes, BOM, UTF-16, UTF-32, jschardet with a 0.7 floor, Latin-1 rescue | not visible in the extracted region | no, the document is already decoded by the editor | no, the buffer is already decoded |
| Invalid UTF-8 | lossy fallback or Latin-1, never refuses | not visible | not applicable | not applicable |
| Line endings | 5% CRLF vote, then conditional rewrite | per line, trailing `\r` always stripped | editor-normalized | buffer-native |
| Binary detection | extension list, then NUL and 5% control bytes | not visible | null-byte scan, then hexdump | not applicable, buffers |
| Symlinks | followed, `stat` not `lstat`, link path reported | not visible | editor-resolved | followed only with authorization |
| Path escape | allowed, service may block | permission rules only | `isFileExternalAndNeedsConfirmation` | refused outside the project except `~/.agents/skills` |
| Gating | permission prompt, yields `rejected` | permission rules, `deny` gives errorCode 1 | content-exclusion check, external-file confirmation | user approval for symlink escape |
| Empty file | `File is empty.`, a success | `<system-reminder>Warning: the file exists but the contents are empty.</system-reminder>` | `(The file \`p\` exists, but is empty)` | an empty fenced code block, no message |
| Offset past EOF | throws a text error | `<system-reminder>Warning: the file exists but is shorter than the provided offset (N). The file has M lines.</system-reminder>` | throws `Invalid offset N: file only has M lines. Line numbers are 1-indexed.` | clamps silently |
| Read-before-write gate | none | yes, an LRU with hashes | none found in this tool | none in this tool |
| Output wrapper | none | none | optional fence plus a header line | fenced block tagged with the path |

Two rows in that table are worth staring at. **Offset past EOF** is handled five different ways across four harnesses: Cursor throws a plain message, Claude Code returns a system reminder, Copilot throws a structured error, Zed clamps. And **empty file** is a success in Cursor, a warning in Claude Code, and a parenthesized aside in Copilot. Neither is a compatibility question you can duck; whatever you pick, a model trained on one of these will be surprised by the others.

## What Runvil can support today

The crate that will implement this is `plugin-std`, and the file access is `runvil_internal::vfs::Access`. Current state: all six tools are declared with Cursor's wire name and a stored JSON schema, each call is checked against that schema, and each call answers `NOT_IMPLEMENTED`. No file logic exists.

### The gap table

| Behavior the report specifies | Access method | Status | What the gap needs |
|---|---|---|---|
| read a file's bytes | `read` | covered | - |
| read a file as text | `read_string` | partial | `read_string` returns `VfsError::NotUtf8` on invalid UTF-8, where Cursor substitutes U+FFFD or falls back to Latin-1. Needs a lossy or detected-decode path, or a decision to refuse |
| **read a byte range without materializing the file** | **none on `Access`** | **missing, and the backend already has it** | `VfsAccess::read_range(path, offset, len)` exists on the backend trait and `RealBackend` overrides it with a real `seek`, so a real file can serve a range without a whole read. `Access` in `handle/operations.rs` does not expose it. Adding `Access::read_range` follows the same four steps as `read` and is the single highest-value gap in this table |
| know a file's size before reading | `stat` | covered | `Stat.size` is `u64` bytes |
| know a path is a directory before reading | `stat` | covered | `Stat.file_type` distinguishes all seven POSIX kinds, more than Cursor distinguishes |
| reject an extension before any I/O | none | missing | a pure function in the Pack; no VFS call needed |
| detect binary content | `read_range` | partial | the NUL-and-5%-control-byte rule needs only the first 8192 bytes, which `read_range` would serve without reading the whole file. Today it forces a full `read` |
| **follow a symlink to read its target** | `read` | covered | `read` resolves with `self.resolve(path)`, which follows a final-component link under the containment check. So `read` follows a link and `stat` does not |
| **report what kind of node a path is, without following** | `stat` | covered | `RealBackend::stat` uses `symlink_metadata`, so a symlink reports `FileType::Symlink` rather than its target's kind. That is *stricter* than Cursor, which uses `stat` and cannot tell you a path is a link at all |
| window by line | none | missing | a pure function over the decoded string; no VFS call |
| count lines | none | missing | same |
| format with line numbers | none | missing | same |
| the 100,000 character cap | none | missing | same, but the unit question (Q2) has to be settled first |
| the empty-file short circuit | `stat` | covered | `Stat.size == 0` answers it before any read, which is cheaper and more exact than Cursor's `content === ""` |
| a per-file lock across the whole read | `Access` claims | **covered, and better than Cursor's** | `read` admits `Claims::claim_read`, and `gate` canonicalizes at receipt before any claim is registered, so the claim key is the canonical path. `claim_key` additionally case-folds below a mount whose filesystem folds case. That closes the spelling race Cursor leaves open |
| release the lock before waiting and re-acquire after | `Access` claims | not needed | the claim is taken and released per call. A retry is a second `read` call, which takes its own claim. Do not add a manual hold to mimic Cursor |
| wait for earlier mutating calls in the same step | none | missing | a harness concern, not a VFS one: it needs the step's call list and a settle signal. `ToolContext` carries `origin` and `services`, so a service could supply it |
| PDF to text | `read` | missing | needs a PDF extraction dependency and a cache. Nothing in `Access` helps; this is a Pack-level subsystem |
| image bytes to a model content part | `read` | partial | `read` gives bytes; the result type has to carry a non-text part, which is a `runvil_plugin::ToolOutput` question, not a VFS one |
| notebook cell rendering | `read_string` | missing | pure function over the JSON text, once JSON parsing is available in the Pack |
| reject `limit: 0` | none | missing | **the schema checker does not enforce it.** `check.rs` honors only `type`, `properties`, `required`, `enum`, and `items`; `read.json` has no `minimum` on `limit`, and even if it did, `minimum` is not in the honored list. So the tool body must reject zero itself |
| coerce `offset: 0` to 1 | none | missing | same: a body-level decision, since the schema cannot express it |
| reject `offset` past EOF with the exact message | `stat` plus a count | missing | `stat` gives bytes, not lines, so the line total needs a decode or a newline scan over the bytes |
| record the read for later tools | none | missing | `Access` fires an `Op::Read` event, which a harness could observe, but there is no read-path set. If Runvil wants a read-before-write gate it needs a store; if it does not, nothing is needed |

### The `read_range` finding, spelled out

`Access` exposes thirteen operations and none of them takes a byte offset. The backend trait does:

```rust
fn read_range(&self, path: &VfsPath, offset: u64, len: u64) -> Result<Vec<u8>, VfsError>
```

with a default that reads the whole file and slices, and an override in `RealBackend` that opens the file, seeks, and reads only the range. `RoutingAccess`, `StoreMountSession`, and `StoreScoped` all delegate it explicitly, and the router's comment says why: "Delegated, not defaulted, so backends that can seek never materialize the file."

So the capability exists and is plumbed through three layers, and the one layer a Pack can reach does not offer it. Three consequences for Read specifically:

1. **Binary sniffing costs a full read.** Cursor sniffs 8192 bytes. A Runvil Read that calls `read` pays for the whole file to look at 8 KiB of it.
2. **A windowed read of a 1 GB file pays for the whole file.** Cursor does too, so this is not a divergence from Cursor - but it is a place where the crate is one method away from being much better, and a Read tool built on `read` alone bakes the worse behavior in.
3. **The 2 GiB refusal cannot be enforced before reading.** `Stat.size` gives you the check, but the check has to be written in the tool and the tool has to trust that `stat` and `read` see the same file.

The shape of the fix, if you want it, is one method on `Access` following the same four steps as `read`:

```rust
pub fn read_range(&self, path: &str, offset: u64, len: u64) -> Result<Vec<u8>, VfsError> {
    let path = self.gate(Op::Read, path)?;
    self.admit(Claims::claim_read, &path)?;
    self.fire(Op::Read, &path);
    self.inner().read_range(&path, offset, len)
}
```

Note `Op::Read` and `claim_read`, not a new op: a range read is a read, and giving it its own op would let it slip past a policy that denies `Read`.

### Three facts that shape the work

**`Access` has no search operation.** Not needed for Read, but it means the cap message's advice ("use the 'grep' tool to search") points at a tool that has to be built as a walk in the Pack rather than a VFS capability. If you rewrite the message for Runvil, do not promise a capability the crate does not have.

**`Access` has no range read, though the backend does.** This is the one gap worth closing before the Read tool is written, because every alternative bakes in a whole-file read. It also decides two other rows in the table: binary sniffing and the 2 GiB pre-check both want to look at a prefix, and only a range read makes that cheap.

**`Access` never touches the disk directly.** Every operation gates the path, admits a claim, fires an event, then delegates. That shape is the template for a new capability: if line counting or binary sniffing turns out to be worth putting below the tool (for instance because a future backend can answer it without materializing bytes), it goes in as an `Access` method with the same four steps, not as a special case in the Pack.

**`read_string`'s error type is a design fork, not a detail.** `VfsError::NotUtf8 { path, valid_up_to }` is a precise, honest error, and it is the right default for a VFS. It is the wrong answer for a read tool, because the model cannot act on it. The clean resolution is to keep `NotUtf8` on `read_string` and have the Read tool call `read` and decode lossily itself, which keeps the VFS honest and puts the policy where the model can see it.

### What the stub gets wrong today

`plugin-std\schemas\read.json` is a snapshot of Cursor's description with some lines edited, and `schemas\EDITS.md` records those edits. Three lines in it are now wrong against the shipped 3.27.12 build, independent of Runvil's own changes:

| Line in the snapshot | Why it is wrong |
|---|---|
| `Any lines longer than 2000 characters will be truncated` | Cursor does not truncate long lines at all. That is Copilot's behavior, adopted into Cursor's description at some earlier point |
| `Results are returned using cat -n format, with line numbers starting at 1` | the shipped format is six-wide plus a pipe; `cat -n` appears nowhere in the 3.27.12 bundle, and the shipped description says `using following format: LINE_NUMBER\|LINE_CONTENT` instead |
| `Assume this tool is able to read all files on the machine.` | already edited to `every file the run can see`, which is right for Runvil and is not Cursor's sentence either |

The first two matter because they are promises to the model that the implementation will not keep. If Runvil's Read does not truncate long lines, drop the sentence. If it does not emit `cat -n`, drop that claim too, and say what it does emit.

## What remains unknowable

**What the capability service does below the `ReadSuccess` boundary.** The service lives in the daemon and in the extension host, and both are present on this machine, so the *logic* is readable - which is how the 5% CRLF vote, the 8192-byte sniff, and the 2 GiB check were recovered. What is not readable is which build a given user runs. The extension ships four bundles (`cursor-agent-exec`, `cursor-agent-host`, `cursor-local-agent-runtime`, and a daemon inside the second) that each contain their own copy of the read path, and they are not guaranteed to be in sync. Every claim here is from the `cursor-agent-exec` copy unless it says otherwise.

**Whether the permission prompt appears, and what it says.** The tool calls `permissionsService.shouldBlockFileRead` and then a resolve path with an approval callback. Whether a prompt is drawn, and its text, is in the extension host UI. Read only ever sees the resulting `rejected` or `permissionDenied`, so the two model-visible strings are all a reimplementation can be sure of.

**Which feature flags are on for a given user.** Four change observable behavior: `enableReadFileLineNumbersArg` adds a parameter, `enableReadToolNegativeOffset` changes both the schema and the semantics, `enableReadFileExplicitOffsetLimitDescription` changes the description, and `useMinimalHarness` replaces the whole tool with `ViewImage`. Their defaults are visible in one place (`Vze`, which sets `enableReadToolNegativeOffset: true` for one configuration) and their rollout state is server-side. So the question "does Cursor accept a negative offset" has no single answer; it depends on the flag.

**Which formatting options a given model gets.** `isGpt5Family` drives `gpt5StyleLineNumbers` and its implementation is visible, but `isComposer1` through `isComposerMatterhorn` decide sparse mode and those are internal model identities this analysis cannot map to public model names.

**Whether `gpt5CodexCatN` is reachable at all.** Every one of the eight construction sites in the four shipped bundles sets it to `false`, and there is no assignment of `true` anywhere. It may be reachable through a path not present in these artifacts, in which case the Codex style is live; or it may be dead. A reimplementation should treat it as documentation of a format that existed, not as a branch to build.

**The exact PDF extraction fidelity.** The heavy lifting is `pdf-worker.js`, a separate chunk loaded through a Piscina pool. This bundle only calls it and expects `{text}` back. Line breaks, page separators, hyphenation, and table layout are all the library's decisions, and reproducing them byte for byte means pinning the same library at the same version.

**What `XK` does to a given image.** It delegates to a resizing middleware keyed on model capabilities, so the base64 the model receives depends on the model. Not reproducible without the same middleware.

**The full encoding ladder.** The `chardet` branch, the Latin-1 rescue heuristic, and the script-ratio overrides are all visible as code, but `chardet`'s own behavior at a given confidence is external and version-dependent. The `0.7` floor is the only number in the chain that can be pinned.

**Whether `executorAppliedRange` and the tool-side window can disagree.** The service may apply a range itself, in which case the tool trusts `executorTotalLines` and does not recount. Two providers of the same information exist, and if they ever disagree the behavior depends on which one is authoritative. Static reading cannot tell whether they can diverge in practice. A reimplementation should use one source of truth and not reproduce the split.

**Whether the 120,000 ms retry wait is ever reached in practice.** It is a ceiling on a wait that usually resolves in milliseconds, and no artifact records the distribution. The value is known; its significance is not.

**What the model does with the output.** Everything above is about the bytes the tool produces. Whether a six-wide prefix helps or hurts a given model, whether the omitted-line notices are read, and whether the cap message actually causes a retry with `offset` are all empirical questions this evidence cannot answer.

## Conformance checklist

Every item from the parent spec's `### ReadFile` list, at `agent-file-tools.md` lines 4688 to 4710, checked against the shipped 3.27.12 build.

| # | Item, shortened | Verdict | Reason |
|---|---|---|---|
| 1 | `path` required, described `The absolute path of the file to read` | **corrected** | the build's describe text ends with a period: `The absolute path of the file to read.` The `dsv3-1018` branch uses `target_file` with a longer relative-path description instead |
| 2 | `limit: 0` rejected, `offset: 0` accepted and rewritten to 1 | **confirmed** | verified in the live bundle: the refine permits `val === 0` and the `.transform` maps it to 1 |
| 3 | `haiku` takes `line_range` inclusive and converts to `offset` + count | **confirmed** | `offset = start`, `limit = end - start + 1`, verbatim in both bundles |
| 4 | binary extension rejected before I/O, lowercased, `unknown` fallback | **confirmed** | the regex and the throw are both in the live bundle; the fallback is present though unreachable |
| 5 | `.pdf`, `.png`, `.jpg`, `.gif`, `.webp`, `.ipynb` absent from the binary list | **confirmed** | the list is exactly the sixteen extensions quoted above |
| 6 | lock acquired on the raw argument before normalization, held across the retry | **corrected** | the lock is taken on the raw argument, but it is *released* before the retry wait and re-acquired inside the retry. It is held across the retry's service call, not across the wait |
| 7 | a miss is retried at most once, after waiting up to 120000 ms | **confirmed** | one retry, `12e4` wait, gated on the miss classifier and on `settled` |
| 8 | retry releases the lock before waiting and re-acquires after; skipped when the call is absent from the step's list | **confirmed** | `earlierCalls` returns `undefined` when the tool call id is not found, and the retry helper returns the original miss |
| 9 | an empty file with no applied range returns success with `totalLines` a literal 0 | **confirmed** | the short circuit is `bTe("",0,...)` and the daemon spells it `createSuccessResult3("",0,...)` |
| 10 | emptiness is `totalLines === 0 AND content === ""`, never derived from the counter | **confirmed** | verbatim in `createSuccessResult3` |
| 11 | the counter is newlines plus one, so `""` is 1 line and a trailing newline adds a blank line | **confirmed** | `countNewlines` in `line-window.js` counts `\n` and the caller adds 1 |
| 12 | the window is computed only when `offset` or `limit` is present | **confirmed** | `computeReadSlice` returns `undefined` for both-absent, and the caller tests for it |
| 13 | negative offset counts from the end, `-1` is last, no limit reads `abs(offset)` | **confirmed** | verified by executing the shipped function: `(5, -2, undefined)` gives start 3, end 5 |
| 14 | clamping happens before the bounds check, so a huge negative offset saturates to 0 | **confirmed** | `(10, -100, undefined)` gives start 0, end 10, no throw |
| 15 | `readRange.endLine` is an inclusive line number though `endIndex` is exclusive | **confirmed** | a full read of 10 lines reports `{startLine: 1, endLine: 10}` |
| 16 | the cap is a pure string-length test applied after windowing | **confirmed** | `content.length > READ_CHAR_HARD_LIMIT` sits in the result builder, which the window path calls after slicing |
| 17 | exceeding the cap yields a success record with `exceededLimit` true and empty content, rendered with the error flag | **confirmed** | `JB(message, !0)` in the render, on a `success` case |
| 18 | the default format is six wide right aligned, pipe, no space after the pipe | **confirmed** | executed the shipped formatter: `"     1\|alpha"` |
| 19 | a full-file read emits no notices; each pluralizes `1 line not shown` or `N lines not shown` | **confirmed** | executed the shipped formatter for both forms and for the range-covers case |
| 20 | `File not found` and `Permission denied` carry no path and no punctuation | **confirmed** | both are bare `new DG("...")` with no interpolation |

Eighteen confirmed, two corrected. The two that matter are item 1, a one-character difference that fails a byte-exact test, and item 6, which if implemented as written produces a deadlock rather than a wrong answer.

A note on item 7 versus item 8. Those two read as a pair and they agree, but only item 8 describes the actual sequence. Item 7 says the retry happens after waiting; item 8 says the lock is released *before* the wait. Read together they specify the correct order. Read alone, item 7 invites holding the lock through the wait, which is the deadlock.

Three items the checklist does not ask about but that a reimplementation has to decide, discovered while writing this report:

- **Line numbers are conditional on a feature flag and an argument.** The checklist assumes they are always on. With the argument present they default to off.
- **`gpt5CodexCatN` is unreachable.** The checklist's implied third style is dead code in this build.
- **The service's `truncated` flag never reaches the model.** The checklist has no item for silent truncation because Cursor's own spec does not treat it as one.

And one correction to a claim outside the checklist, in the same tool's section: `agent-file-tools.md` lines 4155 and 4184 both state that Cursor's and Claude Code's descriptions claim `cat -n`. Cursor's does not - the shipped text says `using following format: LINE_NUMBER|LINE_CONTENT`, and the string `cat -n` is absent from the whole 9,814,287-byte bundle. Claude Code's cannot be checked, because the extracted prompt ends in an identifier whose text is in an unextracted module. This matters because the claim is used to argue that both harnesses inherited an inaccurate description from a common ancestor. On the evidence available, the inherited description inaccuracy is the "reads up to N lines by default" claim, not `cat -n`.

## Sources

**Bundle.** `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js`, 9,814,287 bytes, read as latin1 and sliced. `(bundle)`

| Claim | Offset in 3.27.12 | Offset in the spec's 9,674,628-byte build |
|---|---|---|
| `AG("READ",{` registration | 4353923 | 4338579 |
| `LIe=1e5` alongside `$Ie=2e3` | 4337313 | 4321969 - both are in a different scope, so the offsets are not comparable row for row |
| `iTe` retry tool-name set | 4341254 (`const iTe=new Set(`; 4341281 is the first string, `"ApplyPatch"`) | 4325904 |
| `aTe` binary extension regex | 4341441 | 4326097 |
| `cTe` basename helper | 4341524 | 4326180 |
| `bTe` result shaper | 4343863 | 4328552 |
| `STe` int32 clamp | 4344281 | 4328937 |
| `kTe` window computation | 4344367 | 4329023 |
| `xTe` offset field | 4344664 | 4329320 |
| `ETe` limit field | 4345283 | 4329939 |
| `ITe` include-line-numbers field | 4345510 | 4330166 |
| `TTe` shared schema | 4345804 | 4330460 |
| `CTe` schema dispatch | 4346111 | 4330767 |
| `PTe` tool builder | 4347632 | 4332288 |
| the `g` name switch | 4349000 | 4333644 |
| description generator | 4349 | 4333936 |
| `Cannot read binary files of type` throw | 4355491 | 4340147 |
| `Read image file: ` image text builder | 4362659 | 4347280 |
| `File is empty.` render | 4362791 | 4347447 |
| the cap message | 4362874 | 4347530 |
| `File is empty.` as a description string | 4350826, 4351376, 4352107, 4353192, 4362821 | - |
| `hH` formatter | 2528934 | 2514510 |
| `fH` line-number renderer | 2529940 | 2515516 |
| `lke` worktries repair | 4228597 | 4213265 |
| `Kze` line-number defaults per model | 5078455 | 5063009 |
| `eWe` formatting-options builder | 5079861 | 5064600 |
| `pi_read` registration | 4556823 | not in the spec |

Every offset in the middle column was re-derived for this report by searching the live bundle for the exact anchor, and the anchor is named beside each row so it can be checked.

The delta clusters tightly, which is itself evidence that the two builds differ by one insertion ahead of the `READ` block rather than by edits scattered through it:

| Delta | Rows |
|---|---|
| 15344 | `AG("READ")`, `LIe`, `aTe`, `cTe`, `STe`, `kTe`, `xTe`, `ETe`, `ITe`, `TTe`, `CTe`, `PTe`, the binary throw, the `File is empty.` render, the cap message |
| 15377 | `iTe` |
| 15356 | the `g` name switch |
| 15332 | `lke` |
| 15379 | the image text builder |
| 15311 | `bTe` |
| 15446 | `Kze` |
| 15261 | `eWe` |
| 14424 | `hH`, `fH` |

The fifteen rows at exactly 15,344 are the load-bearing ones: the registration, the whole schema and window chain, the binary throw, and both render strings. The outliers are rows where the spec's own figure points at a nearby-but-not-identical anchor, which is what writing offsets by eye produces; the *behavior* at each is unchanged.

The formatter rows move by 14,424 rather than 15,344 because `hH` and `fH` sit far earlier in the file than the insertion point, so they are affected by whatever changed between the formatter and the read block as well.

The behavior is unchanged between the two builds at every one of these anchors. Every rule in this report was re-derived from the live bundle.

**Daemon.** `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, unminified. `(daemon)`

| Symbol | Real name | Offset | Embedded source path |
|---|---|---|---|
| `kTe` | `computeReadSlice` | 13561741 | `agent/dist/tools/core/read/read.js` |
| `STe` | `clampToInt32` | 13561635 | same |
| `bTe` | `createSuccessResult3`, exported as `createSuccessResult` | 13561153 | same |
| `hH` | `formatCodeBlock` | 8155814 | `agent/dist/tools/core/formatting` |
| `fH` | `addLineNumbers` | 8157397 | same |
| `LIe` | `READ_CHAR_HARD_LIMIT`, aliased from `CHAR_HARD_LIMIT` | 12792688, 8035034 | `agent/dist/tools/core/read/common.js`, `tools/common` |
| `aTe` | `UNSUPPORTED_BINARY_EXTENSIONS` | 13566641 | `agent/dist/tools/core/read/read.js` |
| `iTe` | `SAME_STEP_FILE_WRITER_TOOLS` | 13566500 region | same |
| `aTe`'s twin for content | `isBinaryFile` / `isBufferText` | 15441812 | `local-exec/dist/read.js` |
| `_H` | `detectImageMimeType` | 2518404 region | `agent/dist/tools/core/read` |
| `KIe` | `isPdfBinary` | 13554500 region | `agent/dist/tools/core/read/pdf-utils.js` |
| countNewlines | `countNewlines` | 12795633 | `agent/dist/tools/core/read/line-window.js` |
| sparse interval | `READ_LINE_NUMBER_INTERVAL = 10` | 13566500 region | `agent/dist/tools/core/read/read.js` |
| `lne` | `FileOperationLockManager` | 11316578 onward | `local-exec` and lock modules |
| pi read executor | `LocalPiReadExecutor` | 16860697 | `local-exec/dist/pi/read.js` |
| pi truncation | `truncateHead`, with `PI_DEFAULT_MAX_LINES=2e3` and `PI_DEFAULT_MAX_BYTES=50*1024` | 16825942, 16826006 | `local-exec/dist/pi/truncate.js` |
| pi path resolution | `resolvePiPath`, `resolvePiReadPath`, `macReadVariants` | 16838486, 16839536 | `local-exec/dist/pi/path.js` |

**Scratch corpus.** `C:\Users\Vinnie\cursor\cabinet\_scratch\cursor-tools\`. `(scratch)`

- `tools\READ.js`, the prettified slice, whose prettier run failed and which is therefore raw text. Useful as an index, not as a source.
- `shared\formatter.js`, `shared\read-helpers.js`.
- `readwin.js` (a slicer, not a measurement), `probe-empty.js`, `newlines.js`, `scratch_lie.txt`.
- `BRIEFING.md`, for the mangled-identifier map.

**Probes run for this report.** Scripts written to `cabinet\_scratch\six-tool-reports\read\` and executed against the live bundle. `(probe)`

- `run-fh.js` extracted `fH` verbatim from `main.js` and executed it, producing the measured outputs for all three styles, sparse mode, a trailing newline, and a 7-digit line number.
- `run-hh.js` extracted `hH` and `fH` and executed them, producing the measured head-only, tail-only, singular, and range-covers outputs.
- `run-kte.js` extracted `kTe` and `STe` and executed them against the fifteen cases in the window table.
- `probe1.js` through `probe-catn2.js` for anchor discovery, including the case-insensitive search that established `cat -n` is absent from the bundle.
- `probe-daemon.js`, `slice-daemon.js`, `probe-pi*.js`, `slice-pi*.js` for identifier recovery and the `pi_read` surface.
- `scan-ext.js` across all four shipped bundles, which established that `gpt5CodexCatN` is set to `false` at all eight construction sites.

**Harness sources.** `(source)`

- Claude Code: `C:\Users\Vinnie\cursor\cabinet\_scratch\claude-code\tools\read.js` (registration 257776684, schema builder `Akr` 257768503, execute `$kr` 257782502), `line-number-format.js` (`IUe` 246185269, `BZr` 246185549), `read-file-state.js` (class `Qs` 251201929), `limits.js`, `input-schemas.md`, `readFile.spec.tsx`.
- Copilot: `C:\Users\Vinnie\src\vscode\extensions\copilot\src\extension\tools\node\readFileTool.tsx`, `node\test\readFile.spec.tsx`, and the name mapping in `common\toolNames.ts`.
- Zed: `C:\Users\Vinnie\src\zed\crates\agent\src\tools\read_file_tool.rs` (`format_with_line_numbers` at line 46, `ReadFileToolInput` at 161, `const NAME` at 212, the `~/.agents/skills` restriction at 159), plus `tool_permissions.rs` for `resolve_project_path`, `authorize_symlink_access`, and `resolve_global_skill_path`.

**Runvil crate.** `(source)`

- `C:\Users\Vinnie\cursor\runvil\crates\runvil-internal\vfs\src\handle\operations.rs` for the `Access` methods and their four-step shape.
- `...\vfs\src\error.rs` for `VfsError` and `NotUtf8`.
- `...\vfs\src\stat.rs` for `FileType` and `Stat`.
- `C:\Users\Vinnie\cursor\runvil\crates\runvil-plugin\src\context.rs` for `ToolContext`.
- `C:\Users\Vinnie\cursor\runvil\crates\plugin-std\src\` for the stubs, `check.rs`, `spec.rs`, and `schemas\read.json` with `schemas\EDITS.md`.

**Parent spec.** `C:\Users\Vinnie\cursor\runvil-design\specs\cursor\agent-file-tools.md`: `## Shared infrastructure` at 319 (formatter 323, constants 420, window 457, result shaping 488, lock 504, capability boundary 552), `## Design shape` at 290, `## ReadFile` at 582, `## Cursor versus Claude Code` at 4080, `## Failure matrix` at 4236, `## Build order` at 4387, `## What this document cannot tell you` at 4492, errata at 4584, 4614, and 4646, and the `### ReadFile` conformance list at 4688.

**Deep research.** `C:\Users\Vinnie\cursor\cabinet\_research\2026-10-27-cursor-tools-readfile.md`, read in full; `2026-10-27-cursor-tools-shared-infrastructure.md` and `2026-10-27-cursor-tools-locking-and-concurrency.md` for the formatter, constants, and lock; `2026-10-27-cursor-vs-claude-code-file-tools.md` for the tab-aware separator and the read-state machine.

**Web.** Not used as a primary source anywhere in this report. The one public artifact worth noting is the leaked Cursor prompt at `github.com/EliFuzz/awesome-system-prompts/blob/main/leaks/cursor/2025-11-01_prompt_agent.md`, which matches the `dsv3-1018` branch byte for byte, including its `target_file` field and the article-less `using following format`. That match is the strongest available evidence that a leaked prompt is a faithful copy of the wire text, which is a useful calibration for anyone reading leaked prompts generally. It also shows an image-support section that the shipped `dsv3-1018` branch does not have, so the leak and the build diverge in one place. `(web)`

*2026-10-27 - deepseek-v41-flash-1017*
