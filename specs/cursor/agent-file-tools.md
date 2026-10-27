# Cursor agent harness: implementation specification

How Cursor's agent harness actually works, precise enough to reimplement.

Two halves. The first specifies six file tools (`ReadFile`, `WriteFile`, `StrReplace`, `Delete`, `Glob`, `Grep`) plus `ListDir`, down to exact strings, formats, caps, and matching semantics. The second covers the surrounding harness: prompt assembly and caching, subagent isolation, context compaction, shell behavior, and tool offering. The first half is a contract. The second half is design notes, with a recommendation where I think Cursor's choice is wrong.

## What this document is

This is a specification of six file-manipulation tools that an AI coding agent calls: `ReadFile`, `WriteFile`, `StrReplace`, `Delete`, `Glob`, and `Grep`. It describes each one in enough detail to rebuild it so the rebuilt version behaves the same way, call for call, as the one Cursor ships. It is written for an engineer or an agent building an agent harness, meaning the layer that sits between a language model and a real filesystem and turns the model's tool calls into file operations.

"Identical observable behavior" means something narrow here. It means the same parameter names and the same validation rules, the same matching semantics, the same output formatting down to the padding, the same error strings character for character, and the same caps. It does not mean the same internal code. A reimplementation in Rust that hands the model the same bytes and leaves the same results on disk satisfies this document.

Two caveats before you rely on any of it. First, this is a reverse-engineering document. Everything was recovered by reading a shipped, minified JavaScript bundle, by probing a live agent, and by cross-checking public sources, not by reading Cursor's source code. Second, where Cursor's behavior is arguably a bug, this document labels it as a bug and leaves the claim standing rather than quietly describing the behavior it should have had. The `## Errata` section at the end collects the places where the document contradicts itself, and the per-tool notes call out the individual quirks.

## Orientation: how agent file tools work

None of the rest of this document makes sense without a picture of the loop these tools live inside. This section builds that picture from nothing, so a reader who has never written an agent can start here and a reader who has can skim it.

### The agent loop

A language model has no hands. It cannot open a file, cannot list a directory, cannot write a byte. What it can do is emit text.

An agent harness turns that into file manipulation by defining a protocol. The model emits a block of text that the harness parses as a structured request, called a tool call, naming one tool and supplying JSON arguments. The harness validates the arguments against a schema, runs the corresponding function in the host process, and appends the function's textual result to the conversation as a new message. The model reads that message and decides what to do next. That loop, repeated until the model stops asking for tools, is the entire mechanism by which an agent reads and edits code.

In pseudo-code, roughly:

```
loop:
  reply = model(context)
  if reply contains no tool calls:
      stop
  for call in reply.tool_calls:
      args   = parse_and_validate(call.name, call.arguments)   # may fail
      result = registry[call.name].execute(args)               # may fail
      context.append(tool_result_message(render(result)))
```

Everything in this document lives inside `execute` and `render`. The surrounding harness is out of scope, but two of its properties leak into the tool contract and are worth naming now. Calls within a single turn can run concurrently, which is why there is a lock. And the only thing the model ever receives is the rendered text, which is why formatting and error strings matter as much as the file operations themselves.

### Why the tool description is load-bearing

Every tool here has a description string. It is tempting to read that string as documentation for humans and skim it. It is not documentation. It is prompt text. The harness injects it into the model's context, usually as part of the tool definition block, before the model has produced a single token. It is therefore the most behavior-shaping prose in the system: a sentence telling the model to batch its reads changes how many round trips a task takes, and a sentence telling the model that `old_string` must be unique changes whether it retries with more context or reaches for `replace_all`.

That is why this document quotes descriptions verbatim instead of paraphrasing, and why it treats small wording differences between model families as findings rather than trivia. A one-line description and a nine-bullet description are not two ways of saying the same thing. They are two different behavioral interventions, and swapping them changes what your agent does even when your file operations are perfect.

### Why the surface is versioned

Cursor ships several parallel variants of each tool and selects one at runtime from a version tag. The tags observed in this build are `dsv3-1018`, `cursor-0226`, `dsv3-1205`, `latest`, `gpt5-codex`, `codex-cloud`, and `haiku`. Each corresponds to a model family, meaning a group of models from one vendor that share instruction-following habits and tokenizer quirks. Different families want different prompts. Some follow terse one-liners better than long rule lists. Some need absolute paths stated explicitly. Some handle a `line_range` pair more reliably than separate `offset` and `limit` integers. Rather than pick one description and accept degraded performance everywhere, Cursor keeps a variant per family and pays the maintenance cost.

`latest` is the default and is the branch to implement first. `cursor-0226` and `dsv3-1205` behave like `latest` in almost every respect. `dsv3-1018` is the outlier: snake_case tool names, relative paths, different parameter names, and a handful of text differences that only matter if you are matching bytes. `gpt5-codex` and `codex-cloud` trim descriptions to the bone. `haiku` swaps in `line_range`. Implement `latest`, keep the version switch in your design, and add the other branches only when a specific model needs them.

### The two-layer result model

A tool call produces two different things, and confusing them is the most common source of reimplementation bugs.

The first is a structured result record: a protobuf message, specifically a `oneof`, carrying a case tag plus typed fields. The harness uses it for rendering, telemetry, undo bookkeeping, and the diff view. The second is a rendered text string, produced from that record by a renderer function, which is what gets appended to the conversation and what the model actually reads.

The model never sees the structured record. It sees the string. So `linesAdded`, `linesRemoved`, and `diffString` can be present and accurate in the record while being completely absent from the model's view, and that is exactly what happens in `WriteFile` and `StrReplace`: the diff statistics exist, the UI paints them, and the model receives one flat sentence. Throughout this document, fields are called out as model-visible or client-only for that reason.

Keep the two layers separate in your own code as well. Merge them and you will eventually append a debug field to model output and wonder why the model's behavior changed.

### The seven result cases

Every tool funnels its outcome through the same small set of cases:

- `success`: the operation completed.
- `fileNotFound`: nothing exists at the path.
- `notFile`: something exists there, but it is a directory, a device, or another non-file.
- `permissionDenied`: the operating system or the harness refused.
- `fileBusy`: the operating system reported the resource busy, in practice `EBUSY` from `unlink`.
- `rejected`: the harness refused on policy grounds, before or instead of touching disk.
- `error`: anything else, carrying a message.

Once you internalize that list, every per-tool error table below becomes a lookup rather than a surprise. The tools differ in which cases they can produce, which strings they attach, and whether they surface the case as an error or as successful text, but the taxonomy is shared. Section `## Shared infrastructure` describes where the boundary that produces these cases sits.

### Where the implementation boundary sits

None of these tools call the filesystem directly. Each resolves its arguments, builds a request object, and hands it to a capability service obtained from the host's service registry, which returns one of the seven cases above.

Everything the model can observe about success and failure is decided on the tool side of that boundary: validation, rendering, caps, retry, and strings. Everything below it, meaning the real encoding detection, the flush-to-disk calls, the file-mode handling, and any storage provider quirks, is not specified here because it is not present in the bundle. The document says so explicitly where it matters, and `## What this document cannot tell you` gathers those gaps in one place.

That split is worth respecting in your own implementation. Pick a boundary, write down its contract, and keep the tool layer's observable behavior independent of which storage backend sits underneath. If you swap storage later, the model-facing behavior should not change at all.

## Glossary

Definitions for every term this document uses without explaining. Alphabetical.

**agent mode**: Cursor's mode in which the model may call tools that modify the workspace. The alternative is ask mode. Tools check this mode before writing, and `Delete` has a dedicated error string for the read-only case.

**ask mode**: Cursor's read-only mode. The model may read and search but not modify. Cursor enforces it with a fixed message, quoted in Shared infrastructure as `vG`, telling the model to ask the user to switch modes.

**blob**: An opaque handle to file content held outside the conversation, used when content is too large to inline comfortably. In this document the blob window is `(10000, 100000]` measured in JavaScript string length, not bytes, and `LIe` is both the blob ceiling and the read character cap.

**blob store**: The component that owns blob handles and can materialize them back into bytes on demand. It is below the capability boundary, so this document specifies only that reads may return a blob rather than inline text.

**BOM**: Byte order mark. A short signature at the start of a text file identifying its encoding, `EF BB BF` for UTF-8, `FF FE` for UTF-16LE, and so on. Cursor preserves an existing BOM on write and does not add one where there was none.

**bundle**: The single shipped JavaScript file that implements Cursor's agent tools on the client side. The one examined here is a 9,674,628-byte artifact at `resources/app/extensions/cursor-agent-exec/dist/main.js`. It is minified into one physical line, which is why every location in this document is a character offset rather than a line number.

**capability service**: The host-provided service the tools call instead of touching the filesystem themselves. A tool builds a request, hands it to the service, and receives a oneof result. Everything above that boundary is specified here; everything below it is not.

**Composer**: Cursor's own in-house model family, used primarily for agent mode. It is one of the families the per-family tool variants exist to serve, though this document identifies variants by their version tags rather than by product name.

**CRLF**: Carriage return plus line feed, the two-byte line terminator `\r\n` used by Windows text files. This document's `CRLF` constant refers to that ending as a classification, and CRLF-preserving writes are one of `WriteFile`'s harder behaviors.

**Cursor rules**: Project or user instruction files that Cursor attaches to the model's context automatically, so the model follows house conventions without being told each time. When a read touches a rules file, matching rules are attached to the result as `relatedCursorRules`, which is why some reads cost more tokens than their content suggests.

**DSv3**: The DeepSeek third-generation model line, referenced here through the version tags `dsv3-1018` and `dsv3-1205`. Treat those tags as opaque switch keys, not as dates. The `dsv3-1018` branch is the most divergent tool surface in the bundle: snake_case names, relative paths, and different parameter names.

**fsync**: A filesystem call that flushes buffered writes to durable storage and does not return until the data is committed. `WriteFile` calls it after writing, which makes the write durable but not atomic, because the truncate already happened in place.

**harness**: The program that owns the agent loop: it calls the model, parses tool calls, validates arguments, executes tools, and appends results. If you are reading this document to build something, the harness is what you are building. Also called an agent runtime or agent framework.

**LF**: Line feed, the single-byte line terminator `\n` used by Unix text files. This document's `LF` constant refers to that ending as a classification.

**LRU**: Least recently used. A cache eviction policy that discards the entry touched longest ago. Cursor uses it for the per-path encoding cache, capped at 100 entries, and Claude Code uses it for its read-state store.

**minified**: Source code transformed to be as small as possible, by stripping whitespace and comments and renaming local identifiers to short meaningless strings. Minification is why the bundle is one line and why its internal names look like `hH` and `kTe`.

**mtime**: Modification time. The filesystem timestamp of a file's last content change. Both `Glob` and `Grep` sort results newest-first by mtime, computed in JavaScript after the search returns, under a 2000 ms stat budget.

**NUL**: The zero byte, `0x00`. It cannot appear in a POSIX path, which is why `Grep` uses it to separate a path from a line number in its wire format without ambiguity.

**oneof**: A protobuf construct holding exactly one of a fixed set of fields. Setting any field clears the others. The seven result cases in this document are a oneof, which is why a result is always exactly one of them and never a combination.

**plan mode**: Cursor's mode in which the model may describe intended changes but not apply them freely. `WriteFile` refuses non-markdown paths in plan mode with `Cannot edit non markdown files in plan mode`, and the gate fires after the read and after the diff, so a refused write still costs a full read.

**proto / protobuf**: Protocol Buffers, Google's schema language and binary serialization format for structured messages. Cursor's tool requests and results are protobuf messages, which is why results are described as cases with typed fields rather than as plain objects.

**ripgrep / rg**: A line-oriented recursive search tool, distributed as the `rg` binary. It is fast, respects `.gitignore` by default, and has its own glob and type filtering. `Glob`, `Grep`, and `ListDir` are all thin wrappers around it; `Glob` is ripgrep in `--files` listing mode rather than a glob library.

**SKILL.md**: The definition file for an agent skill, a packaged instruction bundle a harness can load on demand. Reading one is tracked for skill-usage telemetry.

**TOCTOU**: Time of check to time of use. A class of race in which a fact verified at one moment is assumed still true later. `Delete` guards against it by re-`stat`ing the file and comparing `dev` and `ino` before unlinking.

**umask**: The process-level mask that clears permission bits when a file is created. `WriteFile` requests `0o666` before umask, which is why a write can silently strip a script's executable bit.

**vector clock**: A per-replica counter used to order events in a distributed system without a shared wall clock. It does not appear in the specified behavior of any tool here, but it is the usual answer when you need to detect concurrent modification across agents or machines, so it is worth knowing the term when you design past the per-agent lock.

**workspace root**: The top-level directory of the project open in the editor. It is the anchor for relative path resolution and the default target for `Glob`, `Grep`, and `ListDir`. Several descriptions say paths default to it; the `dsv3-1018` branch says "roots" plural, because a window can have more than one.

**zod**: A TypeScript schema declaration and validation library. Each tool's parameter schema is a zod object, and zod's `.describe()` is where the model-facing parameter documentation lives, which is why description text appears attached to fields rather than in prose.

### Mangled identifiers

Because the bundle is minified, the internal names in this document are not English and carry no meaning. `hH`, `kTe`, `AG`, `LG`, `U$.Ikc`, `lne`, and `LIe` are all examples. They appear here for one purpose only: as anchors so you can find the corresponding code in a bundle yourself.

Do not build against them. Minifiers assign short names by frequency, so a name that means "the formatter" in this build can mean something unrelated in the next one, and the offsets shift with every release. Where a name is given, it is shorthand for "the function at this behavior, found at this offset in this build."

Four of them recur often enough to be worth decoding once:

- `AG(...)` registers a tool. Its first argument is the internal tool id, in capitals, and the second is the definition object containing name, schema, description, and execute.
- `LG(...)` wraps an execute function with schema validation. The wrapper is what turns a bad argument into a validation error before the tool body ever runs.
- `U$.Ikc` and `U$.YjP` are zod constructors: `U$.Ikc` builds an object schema and `U$.YjP` builds a string schema.
- `JB(...)` renders a tool result into the text the model sees. Its second argument, where present, marks the result as an error.

#### The real names, recovered

These are not guesses. Cursor ships a second copy of the same code in `cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, and that copy is **unminified with original identifiers intact**. It also embeds the original source paths, for example `agent/dist/tools/core/read/line-window.js` and `agent/dist/tools/core/file-operation-lock-manager.js`.

So the mangled names in this document have known originals. Use the real names in your own code. Keep the mangled names only as offsets into the minified bundle.

| Mangled | Real name | Notes |
|---|---|---|
| `hH` | `addLineNumbers` | in the formatting module, alongside its three variants |
| `fH` | the dispatch inside `addLineNumbers` | chooses among the three variants |
| `kTe` | `computeReadSlice` | read line window |
| `bTe` | `createSuccessResult` | builds the read result record |
| `STe` | `clampToInt32` | shared numeric helper |
| `lne` | `FileOperationLockManager` | `tools/core/file-operation-lock-manager.js` |
| `LIe` | `READ_CHAR_HARD_LIMIT` (alias of `CHAR_HARD_LIMIT`) | read limits module |
| `aTe` | `isBinaryFile` predicate over an extension list | read common module |
| `iTe` | the wait-for-earlier-calls tool name set | step orchestration |
| `vG` | the ask-mode refusal message constant | mode gating |

A caution on the "notes" column: the daemon embeds original source paths, but those path strings do not reliably delimit module boundaries in the compiled output. Several functions sit in a region whose nearest preceding path marker belongs to a different module. So treat the function names as confirmed, since each was read from its own declaration, and treat module placement as approximate unless you verify it yourself.

The three formatter variants also have real names:

- `addLineNumbersGpt5CodexCatN`
- `addLineNumbersGpt5`
- `addLineNumbersDefault`

And the read window function reads exactly as the version in this document, only legible:

```js
function computeReadSlice(totalLines, offset, limit) {
  if (offset === void 0 && limit === void 0) return void 0;
  const effectiveOffset = offset ?? 1;
  const effectiveLimit = limit ?? (effectiveOffset < 0 ? Math.abs(effectiveOffset) : totalLines);
  const startIndex = effectiveOffset < 0
    ? Math.max(0, totalLines + effectiveOffset)
    : Math.max(0, effectiveOffset - 1);
  const endIndex = Math.min(totalLines, startIndex + effectiveLimit);
  if (startIndex >= totalLines) {
    throw new ToolCallUnexpectedEnvironmentError(
      `Offset ${offset} is beyond file length (${totalLines} lines)`
    );
  }
  return { startIndex, endIndex, readRange: new ReadRange({ startLine: startIndex + 1, endLine: endIndex }) };
}
```

Two names worth noting because they carry meaning the mangled version hid. `READ_CHAR_HARD_LIMIT` is an alias of a broader `CHAR_HARD_LIMIT`, which tells you the 100,000 figure is shared across tools rather than read-specific. And `LARGE_TEXT_BLOB_THRESHOLD` is `1e4`, confirming the blob cutoff is a named threshold rather than a magic number.

If you need to recover a name this document does not list, the method is straightforward: search the daemon bundle for a unique string literal that appears near the behavior you care about, then read the surrounding declarations. They are unminified, so the answer is usually visible in a single slice. The script `cabinet\_scratch\cursor-tools\recover-decls.js` automates the extraction.

## Reading guide

Read these four first, in this order:

1. `## Orientation: how agent file tools work`, immediately above. Without the loop, the two-layer result model, and the seven cases, nothing else lands.
2. `## Design shape, before the details`. Six facts that invert common assumptions. If you read only one section of the original document, read this one.
3. `## Shared infrastructure`. Every tool builds its output through the same formatter and shares the same constants. Read it once, carefully, and the per-tool sections get much shorter, because they only describe what differs.
4. `## Build order`. The sequence you should actually implement in, with the dependency reasoning attached.

Then read the tool you are implementing, and only that tool. Each tool section is self-contained by design, and each opens with an `At a glance` block that tells you what the tool is for and what usually goes wrong.

Treat the following as reference, to consult rather than to read straight through:

- `## Cursor versus Claude Code` is comparison material. Useful when you are deciding whether a given behavior is deliberate or inherited, not needed to implement anything.
- `## Companion research` and `## Reproduction recipe` are provenance. They let you verify a claim yourself and they name the files that contain the raw evidence.
- `## Testing notes` is a checklist. Skim it before you write tests, then return to it per tool.
- `## What this document cannot tell you` is a boundary marker. Read it once so you know which questions are out of scope, then stop worrying about them.

Narrative sections, meaning ones you should read in sequence and which argue a position: `What this document is`, `Orientation`, `Design shape`, `Cross-agent locking`, `Cursor versus Claude Code`'s prose paragraphs, `Build order`, and `What this document cannot tell you`.

Reference sections, meaning ones you look things up in: `Glossary`, `Shared infrastructure`'s tables, every per-tool `Parameters`, `Description text`, `Result cases and exact strings`, and `Failure modes` subsection, the comparison tables, `Companion research`, and `Reproduction recipe`.

One structural note. The per-tool sections were written tool by tool, and each ends with a line in italics and a `*2026-10-27 - deepseek-v41-flash-1017*` attribution. Those interior lines mark section boundaries in the original research, not the end of the document. The only attribution line that ends the document is the last one.

## What you are building

The deliverable is a small library with six parts.

First, a set of tool definitions, one per tool. Each definition has five pieces: a public name (and any per-family aliases), the description string that goes into the model's context, a parameter schema that validates incoming JSON, an execute function that performs the operation, and a result renderer that turns the structured result into model-visible text.

Second, a shared formatter that renders file content with line numbers in the three styles described in Shared infrastructure, since every read result and several error hints flow through it.

Third, a shared result-case model with the seven cases, plus the renderers that map each case to its exact string.

Fourth, a lock, scoped deliberately, that serializes tool calls which touch the same file.

Fifth, a capability service boundary with a contract you define, so the tool layer above it stays independent of your storage backend.

Sixth, an undo or bookkeeping layer, if you want revert to work, since `WriteFile` and `Delete` both record prior file state through `recordFileState`.

`## Build order` sequences all of this by dependency and explains why each stage comes where it does. Start there once you have read the orientation material.

## Sources and confidence

Everything here traces to one of three places:

- `(bundle)` - read directly out of Cursor's shipped agent bundle. Authoritative.
- `(claude)` - read out of Anthropic's shipped `claude.exe` or its published `sdk-tools.d.ts`. Authoritative for Claude Code.
- `(web)` - public docs, forum posts, issue trackers. Corroboration only.

**A note on how these labels are actually applied, because it is uneven.** The three labels appear inline mostly in the Claude Code comparison and the web-corroboration material, where the distinction matters most and a reader is most likely to be misled by a secondary source. Through the rest of the document, claims are unlabeled.

Unlabeled claims should be read as `(bundle)` by default, because the overwhelming majority of this document was derived by reading the shipped bundle directly. Where a claim instead came from probing a live agent, it is marked `(probe)`. Where it came from reasoning about the code rather than reading it, the text says so in the sentence.

If a specific claim matters to you, the companion research files give per-claim provenance with byte offsets, and the reproduction recipe lets you check any of them yourself. Treat the inline labels as emphasis markers on the claims most likely to be doubted, not as a complete audit trail.

Three further notes on confidence:

- Offsets are exact for the build examined and will shift between Cursor releases. The behavior descriptions are more durable than the offsets.
- Claims about behavior *below* the capability service (actual disk I/O) rest on the proto contract, not on reading the implementation, which is outside the bundle. The document says so where relevant.
- Claims about which code path a model actually reaches at runtime cannot be settled by static reading, because the branch depends on runtime configuration. Those are flagged where they matter.

Primary artifact:

```
C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js
```

9,674,628 bytes, one minified line. Tool definition offsets:

| Tool | Internal id | Offset |
|---|---|---|
| Delete | `DELETE` | 4178003 |
| StrReplace | `STR_REPLACE` | 4209511 |
| WriteFile | `WRITE` | 4217017 / 4217697 |
| Glob | `GLOB` | 4277336 |
| Grep | `GREP` | 4304168 |
| ListDir | `LS` | 4310351 |
| ReadFile | `READ` | 4338579 |

Because the file is a single line, line-oriented tooling is useless. Read it with `fs.readFileSync(path,'utf8')` and `indexOf`, then slice. Detailed per-tool evidence lives in the companion research files listed at the end.

## Design shape, before the details

Six facts determine the whole architecture. Read these first, because they invert several common assumptions.

**1. Everything is versioned per model family.** Each tool factory switches on a version string. `dsv3-1018` gets snake_case names and relative paths. `latest` and most others get PascalCase names and absolute paths. `gpt5-codex` and `codex-cloud` get terse one-line descriptions. `haiku` gets `line_range` instead of `offset`/`limit`.

```js
function (e) {
  switch (e) {
    case "dsv3-1018": return "delete_file";
    case "cursor-0226": case "dsv3-1205": case "latest":
    case "gpt5-codex": case "codex-cloud": case "haiku": return "Delete";
    default: throw new Error(`Unhandled version: ${e}`);
  }
}
```

A faithful reimplementation needs the `latest` branch at minimum. Supporting the others is what makes a harness work across model providers, because the description wording is tuned to each model's instruction-following quirks.

**2. StrReplace has no fuzzy matching.** There is no Levenshtein, no similarity threshold, no block-anchor strategy, no `...` elision. The only fallback is a line-trimmed block match requiring strict one-to-one line correspondence, trimming per-line leading and trailing whitespace. Everything else is an exact substring match.

**3. Neither StrReplace nor Write has a read-before-write gate.** Both re-read the file themselves and act on fresh content. Write adds a sha256 stale-write fingerprint check between its read and its truncate, which closes the internal race but does not require the model to have read anything. Claude Code does gate, with a real state machine; Cursor deliberately does not.

**4. Reads are capped by characters, not lines.** 100,000 characters, checked as a pure length test, and checked *after* windowing so a small window into a huge file never trips it. Above the cap the tool returns a distinct `exceededLimit` result rather than truncating.

**5. Line numbers render three different ways** depending on model family, and none of them is real `cat -n`.

**6. The lock is per agent, and writes are not atomic.** The in-process lock prevents parallel tool calls inside one turn from colliding. It does nothing across agents, and writes use truncate-in-place rather than temp-plus-rename. It also keys on the raw argument string rather than the resolved path, so two spellings of one file do not serialize.

## Shared infrastructure

Every tool in this document builds its output through the same formatter and reads from the same table of constants. Read this section once, even if you are only implementing one tool, because the line-number renderer decides what the model sees for every read, and several of the limits here appear again in per-tool cap tables without being redefined. Getting the formatter wrong makes correct tools look broken.

### Line number formatting

`hH(input, options)` and `fH(formattingOptions, content, startLineNumber)` at offset 2514510. Three mutually exclusive styles, checked in this order.

**Codex cat -n style** when `formattingOptions.gpt5CodexCatN`:

```js
`${(startLine + i).toString().padStart(6, " ")}  ${line}`
```

Six-wide right-aligned number, two spaces, text. No separator glyph.

**GPT-5 style** when `formattingOptions.gpt5StyleLineNumbers`:

```js
`L${lineNumber}:${text}`
```

**Default**:

```js
`${lineNumber.toString().padStart(6, " ")}|${text}`
```

Six-wide right-aligned number, pipe, text. Rendered, that looks like:

```
     1|const x = 1;
     2|const y = 2;
```

Note this is *not* `cat -n`, which uses a tab and no padding. Cursor's description claims `cat -n` format; only this third style is even close.

**Sparse line numbers.** When `sparseLineNumbers` is an integer N, only lines where `lineNumber % N === 0` get the `NNN|` prefix. Everything else emits bare text with no prefix at all:

```js
if (void 0 !== r && s % r != 0) return e;
return `${s.toString().padStart(6, " ")}|${e}`;
```

**Omitted-line notices.** When `addAmountOfOmittedLines` is true and the slice does not cover the whole file, `hH` prepends and appends:

```
... N lines not shown ...
```

with correct singular form (`1 line not shown`). Prefix goes above with a trailing newline, suffix below with a leading newline.

**Optional XML wrapper.** `hH` can wrap output as:

```
\n<tag path="..." isFullFile="true" language="...">\n{content}\n</tag>\n
```

An empty body renders as `<tag ...></tag>` with no inner newlines. Attributes are `path`, `isFullFile` (only when true), `language`, plus arbitrary extras. Not used on the plain Read path.

**Line counting** counts newlines plus one:

```js
function (e) {
  let t = 0;
  for (let r = e.indexOf("\n"); -1 !== r; r = e.indexOf("\n", r + 1)) t++;
  return t + 1;
}
```

So an empty string is 1 line, and a trailing newline creates a final empty line. This matters: it is why a file ending in a newline has a trailing blank line in the count.

There is a genuine edge here worth spelling out, because it looks like a contradiction and is not. The counter above would give an empty file 1 line. But the read path short-circuits before ever calling it:

```js
if (content === "" && !executorAppliedRange) {
  return createSuccessResult("", 0, fileSize, resolvedPath, void 0, includeLineNumbers);
}
```

That call passes `totalLines` as a literal `0`, not a computed value. Downstream, `isEmpty` is `totalLines === 0 && content === ""`, so it is only true via this path. So both facts hold: the counter treats the empty string as 1 line, and an empty read reports 0 lines, because the empty case never reaches the counter.

Practical rule for a reimplementation: treat "content is the empty string" as your empty test, and set `totalLines` to 0 on that path. Do not try to derive emptiness from the line counter.

There is also a second line counter in the daemon with opposite behavior for the empty case:

```js
function countLines2(text) {
  if (text.length === 0) return 0;
  let lineCount = 1;
  for (let index = 0; index < text.length; index += 1) {
    const char = text[index];
    if (char === "\n") lineCount += 1;
    else if (char === "\r") { lineCount += 1; if (text[index + 1] === "\n") index += 1; }
  }
  return lineCount;
}
```

This one returns 0 for empty input and counts CR, CRLF, and LF all as breaks. The read path uses the simpler `countNewlines(content) + 1` form, not this one. The CR-aware variant belongs to a different subsystem. Know that both exist so you do not pick up the wrong one.

### Constants

```js
const aTe = /\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i;
```

Binary extensions Read rejects before touching the file. Case-insensitive, anchored at end of path. Tested before any file access, so a rejected path costs nothing. PDFs and images deliberately pass through it.

```js
LIe = 1e5,   // 100000 - max characters a read may return; also the blob ceiling
$Ie = 2e3    // 2000
```

Two other thresholds, plus a cache limit that is about PDF conversion rather than streaming:

| Limit | Value | Meaning |
|---|---|---|
| streaming threshold | 8 MiB | above this, content streams rather than buffers |
| hard cap | 2 GiB | absolute refusal |
| PDF text cache | 32 MiB | converted-PDF text cache |

The blob window is `(10000, 100000]` on **JS string length**, not bytes, and `LIe` serves double duty as both the blob ceiling and the exceeded-limit threshold.

```js
const iTe = new Set([
  "ApplyPatch", "AwaitShell", "CopyToBox", "EditNotebook", "MultiEdit",
  "MultiStrReplace", "Shell", "StrReplace", "Write", "edit_notebook",
  "run_terminal_cmd", "search_replace", "write"
]);
```

Tools whose in-flight calls Read waits for before retrying a miss.

```js
const vG = "You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.";
```

### Read window computation

`kTe(totalLines, offset, limit)`:

```js
function kTe(e, t, r) {
  if (void 0 === t && void 0 === r) return;
  const n = t ?? 1,
        s = r ?? (n < 0 ? Math.abs(n) : e),
        o = n < 0 ? Math.max(0, e + n) : Math.max(0, n - 1),
        i = Math.min(e, o + s);
  if (o >= e) throw new DG(`Offset ${t} is beyond file length (${e} lines)`);
  return { startIndex: o, endIndex: i, readRange: new E8({ startLine: o + 1, endLine: i }) };
}
```

Behavior to reproduce exactly:

- Offsets are 1-indexed.
- A negative offset counts from the end. `-1` is the last line. A negative offset with no limit reads exactly `|offset|` lines.
- Clamping happens *before* the bounds check, so `startIndex` saturates at 0.
- Exact error text: `Offset ${t} is beyond file length (${e} lines)`.

Both values are coerced to int32 first:

```js
function STe(e) { return void 0 === e ? void 0 : Math.min(2147483647, Math.max(-2147483648, e)); }
```

`readRange.endLine` is `i`, which is exclusive-end in index terms but reported as an inclusive line number. When the window reaches the end of file, `endLine` equals `totalLines`. This is worth being careful about: a full read of a 10-line file reports `{startLine:1, endLine:10}`.

### Read result shaping

`bTe(content, totalLines, fileSize, path, readRange, includeLineNumbers)`:

```js
function bTe(e, t, r, n, s, o) {
  return e.length > LIe
    ? { output: { case: "content", value: "" }, isEmpty: false, exceededLimit: true,
        totalLines: t, fileSize: r, path: n, readRange: s, includeLineNumbers: o }
    : { output: { case: "content", value: e }, isEmpty: 0 === t && "" === e,
        exceededLimit: false, totalLines: t, fileSize: r, path: n, readRange: s, includeLineNumbers: o };
}
```

`exceededLimit` is purely the character-length comparison against 100000. Not a line count, not a byte count.

### Per-file operation lock

Class `lne` at offset 3440300.

```js
this.lockedFiles = new Set();
this.exclusiveLockActive = false;
this.waitQueue = [];
this.nextAcquisitionOrder = BigInt(0);
```

`waitForLock(ctx, path, opts)`:

1. If `ctx.signal.aborted`, throw the abort error immediately.
2. If the path is already locked, or an exclusive lock is active, or an exclusive waiter is queued ahead, push `{type:"file", path, resolve}` onto `waitQueue` and await. The wait is abortable; on abort the entry is removed and the call throws.
3. Otherwise add the path to `lockedFiles`.
4. Return a disposable handle whose dispose calls `releaseFileLock(path)`.

`waitForExclusiveLock(ctx)` is the same shape but sets `exclusiveLockActive` and first waits for `activeFileOperationCount` to reach zero.

**Scope, and this is the part that surprises people.** The manager is a plain class with instance state, never a module singleton. `this.fileOperationLockManager=` appears zero times in any bundle. There are ten `new lne` sites, all method-local. Two matter:

- `runTurnLoop` (3682252) allocates one per turn, threaded through every step
- `setupStep` (3711894) allocates one per step, for `runSingleStep` and `runModelStep`

It reaches tools through the tool-generator options object as `fileOperationLockManager`, read back per tool as `n?.fileOperationLockManager`.

Consequence: parallel tool calls inside one turn share a lock set. Two chat tabs, two background agents, or a parent plus subagent do **not**. Each has its own agent loop and its own empty `Set`.

`trackAcquisitionOrder` has zero call sites and `nextAcquisitionOrder` never increments. Dead code. No tool takes two locks at once, so there is no deadlock surface.

**Shell takes the exclusive lock.** At offset 4455597:

```js
execute: void 0 !== o ? async (e, t, r, n) => {
  const s = { stack: [], error: void 0, hasError: false };
  try {
    const i = new h_e(r);
    yPe(s, await o.waitForExclusiveLock(e), false);
    return await p(e, t, i, n);
  } catch (e) { s.error = e; s.hasError = true; } finally { wPe(s); }
} : p
```

Note the ternary: if no lock manager is supplied, Shell runs as bare `p` with no lock. This is coarse but correct, because a subprocess is opaque to the agent and cannot be tracked file by file.

Lock waits are unbounded and abort-only. The 120000 ms figure elsewhere is `waitForEarlierCalls`, a different mechanism.

### Capability boundary

Tools do not touch the disk themselves. They build a proto request and call a capability service obtained via `e.get(X$.RpG)`, receiving a oneof result with these cases:

```
success  fileNotFound  notFile  permissionDenied  fileBusy  rejected  error
```

`fileBusy` has exactly one producer: raw `EBUSY` from `unlink` (offsets 8864050, 16192336), surfaced as `File is busy: ${path}`. It is not an advisory lock, not an in-memory registry, and not a dirty-editor check.

Writes use truncate-in-place, not temp-plus-rename. They are not atomic.

This boundary matters for reimplementation: everything above it is specified exactly below, and the oneof tells you what your storage layer must return.

## Cross-agent locking, and why it is not a corruption defence

The lock described in the previous section exists to serialize tool calls, and that only matters because a single turn can issue several of them at once. The model often batches a read of three files into one response, the harness may run those in parallel, and without something to order them two calls can observe the file mid-change. Read the section below with that in mind: the question it answers is not "how do we make writes safe" but "how do we stop two calls in the same turn from confusing each other," which is a much narrower problem than it first appears.

Cursor's locking is intra-agent politeness. It coordinates parallel calls inside one turn. It is not a workspace lock, not a process lock, and not cross-agent.

If you are building something where correctness matters, three improvements are worth making:

1. **Atomic writes.** Temp file plus rename. Cursor does not, so a crash mid-write leaves a truncated file.
2. **Content-hash compare-and-swap on write**, so a concurrent modification is detected rather than silently clobbered.
3. **An OS-level lock or a shared registry above the tool layer**, since a per-agent set cannot arbitrate between agents.

The rest of this document specifies Cursor's behavior as it is. Where you improve on it, note the divergence deliberately rather than by accident.

*2026-10-27 - deepseek-v41-flash-1017*

## ReadFile

### At a glance

This reads a file from the local filesystem and hands its contents to the model with line numbers attached, so the model can refer to specific lines later when it edits.

A model reaches for it when it needs to see what a file actually says before reasoning about it. It reaches for `Grep` instead when it only needs to know which files mention a symbol, and for `Glob` when it only needs the list of paths that match a pattern.

The thing most implementations get wrong: over-limit content returns an error rather than being truncated, and the 100,000-character check happens after windowing. So a small window into a huge file succeeds, while a full read of the same file fails and tells the model to use `offset` and `limit`.

Internal id `READ`, offset 4338579.

### Parameters

Seven schema branches across two shapes. The factory takes `{version, includeLineNumbers, includeNegativeOffset, useSparseReadLineNumbers, useExplicitOffsetLimitDescription}`.

Default branch (`cursor-0226`, `dsv3-1205`, `latest`, `gpt5-codex`, `codex-cloud`):

```js
path:                 string, required
    .describe("The absolute path of the file to read")

offset:               int, optional
    .describe("The line number to start reading from. Only provide if the file is too large to read at once.")

limit:                int, optional
    .describe(...)     // lines to read
```

`dsv3-1018` uses `target_file` instead of `path`, with the same `offset`/`limit`.

The `haiku` branch replaces `offset`/`limit` with:

```js
line_range: [start_line, end_line], 1-indexed, inclusive
```

described as:

```
For partial file reads, use the line_range parameter with [start_line, end_line]
format (1-indexed, inclusive). Example: [100, 150] reads lines 100 through 150.
```

`include_line_numbers` only exists when the `enableReadFileLineNumbersArg` feature flag is on. Do not assume it is present.

**Two validation quirks worth reproducing or deliberately fixing:**

- `offset: 0` is silently rewritten to `1`. The refine explicitly allows 0, then a transform coerces it, so the error message `Offset must be >= 1.` overstates the actual rule.
- `limit: 0` really is rejected, with `Limit must be >= 1.`

**Note the path field name is not uniform across tools.** `READ` accepts `path` or `target_file`. `STR_REPLACE` accepts `path` or `file_path`. `DELETE` accepts `path` or `target_file`. `WRITE` accepts `path` or `file_path`. A compatibility shim must handle each per tool.

### Tool names per branch

`read_file` for `dsv3-1018`, `ReadFile` for `cursor-0226`, `Read` for the rest, and `ViewImage` under the minimal harness.

`dsv3-1018` is the only branch with a leading newline and a trailing period on the line-format sentence, and the only one with no image section at all. Byte-identical reproduction requires preserving those quirks.

### Description text

There is one generator per version. The `latest` and `gpt5-codex` forms are:

```
Reads a file from the local filesystem. You can access any file directly by using this tool.
If the User provides a path to a file assume that path is valid. It is okay to read a file that does not exist; an error will be returned.

Usage:
{offsetLimitUsage}
{lineNumberUsage}
- You have the capability to call multiple tools in a single response. It is always better to speculatively read multiple files as a batch that are potentially useful.
- If you read a file that exists but has empty contents you will receive 'File is empty.'

Image Support:
- This tool can also read image files when called with the appropriate path.
- Supported image formats: jpeg/jpg, png, gif, webp.

PDF Support:
- PDF files are converted into text content automatically (subject to the same character limits as other files).
```

`cursor-0226` gets a one-liner:

```
Reads a file from the local filesystem. This tool can also read image files when called with the appropriate path. Formats supported: jpeg/jpg, png, gif, webp.
```

The `haiku` branch substitutes line-range wording for offset/limit wording, and notes:

```
- For partial file reads, use the line_range parameter with [start_line, end_line] format (1-indexed, inclusive). Example: [100, 150] reads lines 100 through 150.
- It's recommended to read the whole file by not providing line_range, unless the file is too large.
```

### Algorithm

In order:

1. **Resolve the path.** `lke(path)` normalizes. Then `xY(ctx, path, "read")` records the access.
2. **Acquire the per-file lock** via `fileOperationLockManager.waitForLock(ctx, path)`. Held until the tool returns.
3. **Reject binary extensions** by testing the path against `aTe` before any I/O. Error text: `Cannot read binary files of type ${ext}` where `ext` is the lowercased extension or `"unknown"`.
4. **Dispatch to the capability service**, which returns one of the result cases. The service itself decides PDF sniffing (`KIe`), image mime detection (`_H`), and blob handling.
5. **On the retry path**: if the result is `fileNotFound`, or `invalidFile` with an unrecognized reason, or `error` on a path that sniffs as an image, the tool releases its lock and waits for earlier in-flight edit calls:

```js
const o = await e.get(Jne)?.waitForEarlierCalls(t, 120000, {
  includeSettled: true,
  toolNames: iTe
});
```

`iTe` is the set of edit, write, and shell tools listed in Shared infrastructure. If nothing settles, it returns the original miss. If something settles, it re-acquires the lock and retries **exactly once**. A `same_step_read_race` metric is recorded when this shim fires.

**The lock is released before the wait, deliberately.** Holding it while waiting for a writer that needs the same lock would deadlock, so the sequence is release, wait, re-acquire, retry. Getting that order wrong produces a hang rather than an error, which is a much worse failure to debug.

This is the mechanism that stops a Read from racing a Write that has not flushed yet. Note it fires only for not-found and partial-image misses, not for permission errors or generic failures.

6. **Normalize line endings** for text content: `\r\n` becomes `\n`, then bare `\r` becomes `\n`.
7. **PDF conversion** if the content sniffs as PDF. Both the converted text and the raw bytes are cached by hash, capped at 32 MiB.
8. **Empty-file branch**: if content is `""` and no range was applied, return the empty result. Rendered as `File is empty.`
9. **Compute the window** with `kTe(totalLines, offset, limit)` if either was supplied. Slice the line array and return.
10. **Otherwise return the whole file**, subject to the 100,000-character cap in `bTe`.

**Ordering detail that matters:** the `exceededLimit` check happens *after* windowing. A huge file read with a small window never trips it. This is the correct order and easy to get backwards.

### Output formatting

The model sees the content through `hH`, which applies the line-number style described in Shared infrastructure.

For a file with content `alpha\nbeta\ngamma`, the default style renders:

```
     1|alpha
     2|beta
     3|gamma
```

Partial reads additionally get omitted-line notices when the range does not cover the file:

```
... 40 lines not shown ...
     41|matched line
... 12 lines not shown ...
```

For images, the result is a two-part content array:

```js
{
  content: [
    { type: "text",  text: `Read image file: ${path}` },
    { type: "image", data: Buffer.from(bytes).toString("base64"), mimeType }
  ],
  isError: false
}
```

Note the text part is literally `Read image file: <path>`.

### Result cases and exact strings

| Condition | Model-visible text |
|---|---|
| empty file | `File is empty.` |
| over 100,000 chars | `Requested content exceeds maximum allowed characters (100000 characters). The file is ${fileSize} bytes.\nPlease use offset and limit parameters to read specific portions of the file, or use the 'grep' tool to search for specific content.` |
| file not found | `File not found` |
| permission denied | `Permission denied` |
| not a valid file | `${reason || "Path is not a valid file to read"}: ${path}` |
| binary extension | `Cannot read binary files of type ${ext}` |
| rejected | `Read operation rejected` |
| offset past EOF | `Offset ${offset} is beyond file length (${totalLines} lines)` |

The exceeded-limit message is marked as an error (`JB(text, true)`), so it presents as a failure rather than a success with partial content. That matters: a naive implementation might truncate and succeed, which is not what Cursor does.

### Side effects

A successful read records the path in `stateHandler.readPaths` and `stepReadPathDedup`, and classifies the read as `"first"` or `"repeat"` for telemetry. It does not gate later writes on this, because neither StrReplace nor Write checks it.

Two details about that bookkeeping:

- `readPaths` stores the service-returned path **verbatim**. The `Fe` wrapper applied to it is a privacy redaction wrapper, not a path normalizer, so read-before-write lookups are spelling-sensitive. Two different spellings of the same file would not match.
- The `readPaths` record is used for cursor-rule attachment and skill tracking, not for write authorization.

If the read path matches a Cursor rule file, matching rules are attached to the result as `relatedCursorRules`. If it is a `SKILL.md`, skill usage is tracked. Both are harness concerns, not part of the tool contract, but they explain why a read is not always a pure read in terms of token cost.

### Algorithm pseudocode

Language-neutral steps, in execution order. Every throw sits where it fires, and the ordering is the part that matters: the lock is taken on the raw argument, the binary-extension test runs before any file access, and the character cap is evaluated after windowing.

```
TOOL ReadFile

INPUTS
  args        parsed arguments: { path | target_file, offset?, limit?, line_range?, include_line_numbers? }
  ctx         request context: abort signal, workspace roots, stateHandler, blobStore, step call list
  lockManager per-turn FileOperationLockManager, may be absent
  service     capability service: read(path, offset, limit) -> oneof result

OUTPUT
  ReadToolCall record, oneof { success(ReadResult) | error }, which a separate render step turns into text

1  PARSE     validate args against the version schema            // arg-parse wrapper, before any step below
              on failure THROW "Invalid arguments:\n<key>: <message>"
2  rawPath  = "target_file" in args ? args.target_file : args.path
3  path     = lke(rawPath)                                       // worktries -> worktrees repair only
4  dispose  = lockManager?.waitForLock(ctx, rawPath)             // keyed on the raw string, not on path
              // held until every exit path, including all throws below
5  IF path matches /\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i
              ext = lowercase(last ".<suffix>" of path) or "unknown"
              THROW "Cannot read binary files of type ${ext}"    // before any file access
6  IF args.line_range is present
              offset = line_range[0]
              limit  = line_range[1] - line_range[0] + 1          // inclusive end becomes a count
   ELSE       offset = args.offset; limit = args.limit
   offset = clampToInt32(offset); limit = clampToInt32(limit)
7  first    = service.read(path, offset, limit), execId = uuid5(toolCallId)
8  miss     = classify(first)
              fileNotFound                                    -> "file_not_found"
              invalidFile AND reason starts "File is not..."   -> "partial_image"
              error AND path has an image extension            -> "partial_image"
              anything else                                    -> none
9  IF miss is not none
              dispose()                                          // release before waiting
              waited = waitForEarlierCalls(toolCallId, 120000, {includeSettled: true, toolNames: MUTATING})
              IF waited is undefined OR waited.settled is false
                  result = first                                 // original miss returned unchanged
              ELSE
                  dispose = lockManager?.waitForLock(ctx, rawPath)
                  result  = service.read(path, offset, limit), execId = uuid5(toolCallId + ":same_step_read_race")
              // exactly one retry; the retry never re-enters this block
   ELSE       result = first
10 SWITCH result.case
              "error"            -> THROW result.error                                  // renders as "Error: ${error}"
              "rejected"         -> THROW result.reason || "Read operation rejected"
              "fileNotFound"     -> THROW "File not found"
              "permissionDenied" -> THROW "Permission denied"
              "invalidFile"      -> THROW (result.reason || "Path is not a valid file to read") + ": " + path
              default            -> THROW "Unknown error"
11 resultPath   = result.path                                    // service-resolved, not the argument
   fileSize     = Number(result.fileSize)
   rangeApplied = result.rangeApplied
   totalLines   = result.totalLines
   out          = result.output
12 IF out.case is "data"
              IF PDF sniff passes (first bytes are 25 50 44 46, or path ends ".pdf" case-insensitively)
                  text = pdfToText(out.value), then CRLF -> LF, then bare CR -> LF
                  out  = { case: "content", value: text }
              ELSE IF an image mime is detectable, magic bytes first, extension second
                  RETURN image success record:
                    content = [ { type: "text",  text: "Read image file: ${resultPath}" },
                                { type: "image", data: base64(bytes), mimeType } ]
                    totalLines = 0, readRange absent, isEmpty false, exceededLimit false
              ELSE
                  RETURN the data record unchanged         // video and other binary payloads
13 IF blobStore exists AND out.value.length > 10000 AND out.value.length <= 100000
              store UTF-8 bytes of out.value under its sha256
              RETURN success { output: contentBlobId, totalLines, readRange 1..totalLines }
14 IF out.value === "" AND rangeApplied is false
              RETURN success { output: "", isEmpty: true, totalLines: 0, readRange absent }
              // totalLines is a literal 0 here; the line counter is never consulted
15 IF rangeApplied
              window    = computeReadSlice(totalLines, offset, limit)
              readRange = window?.readRange ?? { startLine: 1, endLine: totalLines }
   ELSE
              lineCount = (offset is undefined AND limit is undefined)
                            ? countNewlines(out.value) + 1
                            : split(out.value, "\n").length
              window    = computeReadSlice(lineCount, offset, limit)
              IF window is not undefined
                  out.value  = join(split(out.value, "\n")[window.startIndex ..< window.endIndex], "\n")
                  totalLines = lineCount
              readRange = window?.readRange
16 IF out.value.length > 100000                                   // after windowing, never before
              RETURN success { output: "", exceededLimit: true, isEmpty: false,
                               totalLines, fileSize, path: resultPath, readRange }
17 RETURN success { output: out.value, exceededLimit: false,
                    isEmpty: (totalLines === 0 AND out.value === ""),
                    totalLines, fileSize, path: resultPath, readRange }
18 dispose

RENDER(record)                                                  // sees only the record, never the throw
  case "error"                     -> "Error: ${errorMessage}", isError true
  success AND isEmpty              -> "File is empty.", isError false
  success AND exceededLimit        -> the two-line cap message, isError true
  success AND path ends ".ipynb"   -> notebook cell rendering, no line numbers, no hH call
  success AND output is image      -> two-part content array, isError false
  success otherwise                -> renderNumberedLines(content, startLine = readRange.startLine ?? 1,
                                       totalLines, style, addOmittedLines = readRange exists AND
                                       (readRange.startLine > 1 OR readRange.endLine < totalLines))
```

Helper: the read window arithmetic. `startIndex` is a zero-based inclusive index, `endIndex` is zero-based exclusive, and the same number is reported as `readRange.endLine` because that field is documented as the last line number.

```
computeReadSlice(totalLines, offset, limit)

1 IF offset is undefined AND limit is undefined: RETURN undefined
2 effectiveOffset = offset ?? 1
3 effectiveLimit  = limit ?? (effectiveOffset < 0 ? abs(effectiveOffset) : totalLines)
4 startIndex      = effectiveOffset < 0
                      ? max(0, totalLines + effectiveOffset)     // -1 is the last line
                      : max(0, effectiveOffset - 1)              // 1 is the first line
5 endIndex        = min(totalLines, startIndex + effectiveLimit)
6 IF startIndex >= totalLines
      THROW "Offset ${offset} is beyond file length (${totalLines} lines)"
      // ${offset} is the raw argument, and clamping in step 4 runs first, so a huge negative
      // offset saturates to 0 and cannot reach this throw
7 RETURN { startIndex, endIndex, readRange: { startLine: startIndex + 1, endLine: endIndex } }
```

Helper: emptiness and line counting, which are two different rules on purpose.

```
countNewlines(s)
  count "\n" occurrences; RETURN count + 1
  // "" is 1 line; "a\n" is 2 lines; a trailing newline therefore adds a final blank line

isReadEmpty(totalLines, content)
  RETURN totalLines === 0 AND content === ""
  // only reachable through the empty-file short circuit, which passes totalLines = 0 literally
```

Helper: the omitted-line notices that wrap a partial read.

```
renderNumberedLines(content, startLine, totalLines, style, addOmittedLines)
1 body = for each line i of content:
             lineNumber = startLine + i
             style CodexCatN -> pad6(lineNumber) + "  " + line
             style Gpt5      -> "L" + lineNumber + ":" + line
             style Default   -> pad6(lineNumber) + "|" + line
             style Sparse(N) -> lineNumber mod N == 0 ? pad6(lineNumber) + "|" + line : line
2 IF addOmittedLines
      IF startLine > 1
          body = "... " + (startLine - 1) + " " + plural(startLine - 1, "line", "lines") + " not shown ...\n" + body
      lastShown = startLine + countNewlines(content) - 1
      IF totalLines > lastShown
          body = body + "\n... " + (totalLines - lastShown) + " " + plural(totalLines - lastShown, "line", "lines") + " not shown ..."
3 RETURN body
```

Two ordering notes that the prose states and a reimplementation must not reorder. The binary-extension rejection precedes the service call, so a rejected path costs no I/O. The `exceededLimit` comparison precedes nothing that matters on the full-file path but follows windowing on the windowed path, so a small window into a large file succeeds while a full read of the same file fails.

### Worked examples

Tables fold newlines to `\n` and escape the pipe as `\|`; the fenced blocks below show the same bytes unfolded. `(confirmed)` marks a string read out of the bundle or reproduced by probe. `(derived)` marks a case that follows from a quoted code path but was not separately observed.

| Case | Input | Expected output |
|---|---|---|
| ordinary success (confirmed) | `path: C:\work\proj\src\greet.ts`, 16 bytes `alpha\nbeta\ngamma` | `     1\|alpha\n     2\|beta\n     3\|gamma` |
| empty file (confirmed) | same path, 0 bytes | `File is empty.` as a success with `totalLines: 0` and no `readRange` |
| file holding one newline (derived) | 1 byte, `\n` | `     1\|\n     2\|`, two numbered empty lines, because the empty fast path requires `content === ""` |
| trailing newline (confirmed) | `alpha\nbeta\n` | `     1\|alpha\n     2\|beta\n     3\|` |
| offset past EOF (confirmed) | 3-line file, `offset: 50` | `Offset 50 is beyond file length (3 lines)` |
| negative offset, no limit (confirmed) | 5-line file, `offset: -2` | `... 3 lines not shown ...\n     4\|delta\n     5\|epsilon` |
| negative offset past the start (derived) | 10-line file, `offset: -100` | the whole file, 10 numbered lines, because the clamp to 0 runs before the bounds check |
| offset zero (confirmed) | `offset: 0` | line 1 onward, identical to `offset: 1`, no error |
| limit zero (confirmed) | `limit: 0` | `Limit must be >= 1.` |
| small window into a huge file (confirmed) | 200,000-character file, `offset: 3, limit: 2` | two lines beginning `     3\|`, no error, because the cap is checked after windowing |
| one character over the cap (confirmed) | 100,001-character ASCII file, no window | the two-line exceeded-limit error below, delivered as `isError: true` |
| exactly at the cap (derived) | 100,000-character file, no window | the whole file with line numbers, no error, because the test is `length > 100000` |
| binary extension (confirmed) | `path: C:\work\proj\build\app.zip` | `Cannot read binary files of type zip`, thrown before any I/O |
| missing file (confirmed) | `path: C:\work\proj\src\nope.ts` | `File not found`, with no path and no punctuation |
| directory (confirmed) | `path: C:\work\proj\src` | `Path is a directory, not a file: C:\work\proj\src` |

#### Whole read of a file that ends in a newline (confirmed)

Input:

```json
{"path": "C:\\work\\proj\\src\\greet.ts"}
```

File on disk, where the final character is a newline:

```text
import { format } from "./format";

export function greet(name: string): string {
  return format("Hello, " + name);
}
```

The model receives exactly this, one text part and nothing else:

```text
     1|import { format } from "./format";
     2|
     3|export function greet(name: string): string {
     4|  return format("Hello, " + name);
     5|}
     6|
```

Line 6 is empty and still numbered, because the line counter is newlines plus one. The number field is six characters wide and right aligned, so line 1 carries five leading spaces, and the pipe sits against the digits with no space before the text. A `cat -n` based implementation would emit a tab after the number and would not number a trailing empty line at all.

#### Partial read with both omitted-line notices (confirmed)

Input, on a 50-line `server.ts` where line 41 is `export function shutdown(): void {`:

```json
{"path": "C:\\work\\proj\\src\\server.ts", "offset": 41, "limit": 1}
```

The model receives:

```text
... 40 lines not shown ...
    41|export function shutdown(): void {
... 9 lines not shown ...
```

The head count is `startLine - 1`, which is 40, and the tail count is `totalLines - (startLine + lineCount - 1)`, which is `50 - 41 = 9`. Both notices use the window's own line numbering, and both switch to the singular form when the count is 1: `... 1 line not shown ...`. There is no blank line between a notice and the content.

#### Full read of a file one character over the cap (confirmed)

Input, where `build.log` is 100,001 bytes of ASCII:

```json
{"path": "C:\\work\\proj\\logs\\build.log"}
```

The model receives, as an error rather than as partial content:

```text
Requested content exceeds maximum allowed characters (100000 characters). The file is 100001 bytes.
Please use offset and limit parameters to read specific portions of the file, or use the 'grep' tool to search for specific content.
```

What each example proves:

- The whole read pins the six-wide right-aligned number, the bare pipe with no following space, and the trailing empty line that a trailing newline creates.
- The partial read pins the two notices, their exact wording, and the fact that they are computed from the window rather than from the request.
- The over-cap read pins that the 100,000-character limit is a character count reported as an error, not a truncation reported as success. Reading the same file with `offset: 3, limit: 2` returns two lines and no error, which is the ordering rule from `### Algorithm`.

*2026-10-27 - deepseek-v41-flash-1017*

## WriteFile

### At a glance

This creates a file or replaces a file's entire contents, given a path and the complete new text.

A model reaches for it when it is writing something new or replacing a file wholesale. It reaches for `StrReplace` instead when it only wants to change part of an existing file, and it should prefer `StrReplace` for edits because a full overwrite loses anything the model did not reproduce.

The surprising part: it does not write UTF-8. It detects the existing file's encoding, preserves its byte order mark, and matches its line-ending style including per-line mixed endings, so writing ASCII into a UTF-16 file keeps it UTF-16. And there is no read-before-write requirement at all, despite the description telling the model to read first; the tool does its own read and its own freshness check.

Internal id `WRITE`. The two `AG("WRITE", ...)` sites (4217017 and 4217697) are **mutually exclusive branches of one factory** (`hke` at offset 4214990), not a wrapper plus an implementation. The first is behind `if ("dsv3-1018" === n) return`, so exactly one is ever constructed. They share one execute closure and differ only in name, schema, description, and which path key they read.

Correction to a common claim: this tool has **no read-before-write requirement**, and no error string for one. A live probe overwrote a file the agent had never read. The only freshness protection is the sha256 stale-write guard described below.

### Parameters

`dsv3-1018` branch:

```js
file_path: string, required
    .describe("The path to the file to modify. Always specify the target file as the first argument. You can use either a relative path in the workspace or an absolute path.")
contents:  string, required
    .describe("The contents of the file to write")
```

Default branch:

```js
path:      string, required
    .describe("The absolute path to the file to modify")
contents:  string, required
    .describe("The contents to write to the file")
```

Note the content field is `contents`, not `file_text`. The Anthropic-native `str_replace_editor` shape uses `file_text`; Cursor does not route through that schema.

### Description text

Both branches share the same advice block, differing only in the read-tool name:

```
- If this is an existing file, you MUST use the read_file tool first to read the file's contents.
```

For the default branch the tool name is substituted to `Read`.

### Algorithm

The write is a read-modify-write, and the read is performed by the tool itself. That is why there is no model-facing read requirement.

Exact order:

1. **Normalize the path** (`lke`, which includes typo repair on malformed paths).
2. **Read the existing content** through the same capability service Read uses.
   - `success` with text output, capture old content
   - `success` with non-text output, throw the binary-file error
   - `fileNotFound`, old content is `undefined`; this is the new-file path
   - any other case, convert to the corresponding error
3. **Compute the diff**, labeled `"write"`, with `fileChangeType` of `"added"` when there was no old content and `"modified"` otherwise.
4. **Plan-mode gate**: if mode is PLAN and the path is not a markdown or canvas file, refuse with `Cannot edit non markdown files in plan mode`. Allowed suffixes: `.md`, `.markdown`, `.mdown`, `.mkd`, `.mkdn`, `.mdx`. The gate fires after the read and after the diff, so a refused plan-mode write still costs a full read.
5. **Permission gate.**
6. **`mkdir -p`** the parent directory.
7. **Re-verify the sha256 fingerprint** of the file against what was read in step 2. If it changed, refuse with the stale-write error.
8. **Write**: `open` with write/create/no-follow/close-on-exec, `truncate`, `write`, `fsync` (the call that forces buffered data out to durable storage).
9. **Report** `Wrote contents to ${path}`, where path is the raw argument after normalization, not the resolved absolute path.

Note the ordering of steps 7 and 8. The fingerprint check is the last thing before the truncate, which is what makes it effective against an external writer that modified the file during steps 3 through 6.

### Result text

Exactly:

```
Wrote contents to ${path}
```

There is no snippet, no line count in the message, and no diff trailer. `linesAdded`, `linesRemoved`, and `diffString` are structured proto fields passed alongside the message, not text appended to it. The UI renders them; the model does not see them.

### Failure modes

| Condition | Text |
|---|---|
| invalid file URL | `${path} is not a valid file URL. Use an absolute path instead.` |
| worktree escape | `You cannot search other worktrees for this repository, stay within your workspace paths.` |
| plan mode, non-markdown | `Cannot edit non markdown files in plan mode` |
| permission denied | `Write permission denied: {path}: Permission denied` |
| path is a directory | `Path is a directory, not a file: {path}` |
| binary target | `File content is binary data, not text: {path}` |
| stale write | `{path} changed on disk after this tool call read it...` |
| file busy | `File is busy: ${path}` |
| quota exceeded | quota-specific message from the executor |

Note these error strings differ from the ones StrReplace uses for the same underlying conditions. Write says `Write permission denied: {path}: Permission denied` where StrReplace says `Read permission denied: {path}`. Do not share error construction between the two tools.

### Undo bookkeeping

On success the tool calls:

```js
recordFileState(result.path, newContent, originalContent)
```

so a later revert can restore the prior bytes. `originalContent` is `undefined` for a newly created file, which is how revert distinguishes "restore these bytes" from "delete this file".

### Encoding, BOM, and line endings

This is the most intricate part of the tool and the part most likely to be reimplemented wrongly. Cursor preserves the existing file's encoding and line-ending style rather than writing raw UTF-8. All of the below was confirmed byte-for-byte with live probes, not just read from source.

Three terms recur throughout. CRLF means carriage return plus line feed, the two-byte `\r\n` ending Windows uses. LF means line feed, the single-byte `\n` ending Unix uses. BOM means byte order mark, the short signature at the start of a file that identifies which encoding it uses.

**Format detection** runs on the first 8192 bytes, in this order:

1. Zero-length file, platform default: `{encoding:"utf8", lineEnding: platform === "win32" ? CRLF : LF}`
2. UTF-16LE: `FF FE` with a non-zero third byte, or a greater-than-80% sampling of ASCII-plus-zero-byte pairs
3. UTF-16BE `FE FF`, UTF-32BE `00 00 FE FF`, UTF-32LE `FF FE 00 00`
4. Binary sniff: any zero byte, or more than 5% of bytes below `0x20` that are not tab, LF, or CR
5. Valid UTF-8 without ESC, unless pure ASCII
6. Invalid UTF-8 but "looks like UTF-8 with a few bad bytes", still `utf8`
7. Charset detection with a confidence threshold of 0.7, else `utf-8`. A windows-1252 result that decodes with U+FFFD is downgraded to utf-8.
8. Extension shortcut: for `.png .jpg .jpeg .gif .webp .ico .bmp .svg .pdf .zip .tar .gz .exe .dll .so .dylib .bin` and video extensions, skip sniffing entirely and report binary without reading the file

**Line-ending classification:**

```js
function gr(crlf, lf) {
  if (0 === crlf && 0 === lf) return LF;
  return crlf / (crlf + lf) * 100 >= 5 ? CRLF : LF;
}
```

CRLF requires at least 5% of line terminators. A file with one CRLF among many LFs is classified LF.

**BOM handling:**

- `utf8bom` encoding always prepends `EF BB BF`
- Plain `utf8` with no explicit hint prepends a BOM **only if the existing file had one**. A BOM-less file stays BOM-less; a BOM file keeps its BOM.
- `utf16le/be` and `utf32le/be` keep their BOM if the existing file had one
- The chosen encoding is cached per path, LRU-style (least recently used, so the entry touched longest ago is evicted first), capped at 100 entries

**Line endings on write**, four rules in priority order, where the first three transform and the fourth is the default:

1. **If incoming content already contains `\r\n`, leave it alone.** The model's own mixed endings win outright.
2. **If the existing file has mixed endings**, run a line-aware merge: split both into lines plus per-line terminators, do a Myers-style line alignment, and give each new line the terminator of the existing line it aligned to, defaulting otherwise.
3. **Otherwise, if the detected ending is CRLF**, rewrite every bare LF not preceded by CR to CRLF.
4. **Otherwise**, leave the content alone.

That second rule is the one a naive implementation gets wrong. A probe writing `"mm\nnn\noo\n"` into a mixed `"aa\r\nbb\ncc\r\n"` file produced `"mm\r\nnn\noo\r\n"`, where the middle line kept the existing bare LF. A uniform rewrite cannot reproduce that.

**Trailing newline:** there is no trailing-newline logic anywhere in the write path. Content ending without a newline produces a file ending without a newline; content ending with one preserves exactly one.

**Empty string:** writing `contents: ""` to an existing file truncates it to 0 bytes and reports success normally. An empty write is a successful write, not a refusal.

### Stale-write guard

The only freshness protection is a sha256 fingerprint, and it is not a read-before-write gate.

The tool records a fingerprint of the content it read at the start of its own read-modify-write. After permission checks and `mkdir -p`, it re-verifies that fingerprint immediately before the `open`/`truncate`. If the file changed in between, it refuses with:

```
{path} changed on disk after this tool call read it...
```

This closes the read-write race window inside a single Write call. It does not require the model to have read the file, and it does not know anything about what the model believes.

### File mode

Fixed `0o666` before umask (the process-level permission mask that clears bits at creation time). **No executable-bit preservation.** A write to a script that was `0755` will lose its executable bit. If you are reimplementing this and care about that, preserve the mode deliberately - it is a real papercut.

The write is in place, via `open` with write/create/no-follow/close-on-exec, then `truncate`, `write`, `fsync`. No temp file, no rename, so it is not atomic.

### Lock keying, and a subtle bug

`waitForLock` keys on the **raw argument string**, not the resolved path. Two differently spelled references to the same file, such as a relative and an absolute path, or one containing `..`, do not serialize against each other. If you reimplement the lock, key it on the canonical resolved path instead.

### Algorithm pseudocode

The write is a read-modify-write with the read performed inside the tool. Note where the plan-mode gate sits: after the read and after the diff, so a refused plan-mode write still pays for a full read.

```
TOOL WriteFile

INPUTS
  args       parsed arguments: { path | file_path, contents }
  ctx        request context: mode, workspace paths, stateHandler, approval provider
  service    capability service: read(path) and write(path, text) -> oneof results

OUTPUT
  EditToolCall record, case success carrying { path, message, linesAdded, linesRemoved, diffString }

1  PARSE      validate args against the version schema
               missing path/file_path, missing contents, or wrong type
                   THROW "Invalid arguments:\n<key>: <message>"
2  streamed   as each chunk of the argument JSON arrives, watch for the path field to complete;
               the first time it does, acquire dispose = lockManager.waitForLock(ctx, rawPath)
               IF the stream ends and no lock was ever acquired
                   THROW "Failed to acquire file lock - path may not be parsed"
3  path     = lke(args.file_path ?? args.path)                 // worktries -> worktrees repair only
   contents = args.contents                                    // "" is legal, the schema has no minimum
4  read     = service.read(path), execId = uuid5(toolCallId + "-read")
5  SWITCH read.case
             success AND output is text      -> old = output.value
             success AND output is not text  -> THROW "File is not a text file (binary data)"
             fileNotFound                    -> old = undefined        // new-file path, not an error
             permissionDenied                -> THROW "Read permission denied: ${path}"
             invalidFile                     -> THROW (reason || "Path is not a valid file to read") + ": " + path
             rejected                        -> THROW reason
             error                           -> THROW the service error text
             anything else                   -> THROW "Unhandled readResult.result: ${result}"
6  diff     = computeDiff(old ?? "", contents,
                          fileChangeType = old === undefined ? "added" : "modified")
   message  = "Wrote contents to ${path}"                      // raw argument after lke, not resolved
7  IF mode is PLAN AND path is not markdown and not a canvas path
               markdown suffixes allowed: .md .markdown .mdown .mkd .mkdn .mdx
               THROW "Cannot edit non markdown files in plan mode"
               // fires after the read and after the diff; the renderer substitutes a longer
               // explanation naming the SwitchMode tool
8  IF path is a malformed "file://" URL
               THROW "${path} is not a valid file URL. Use an absolute path instead."
9  resolved    = resolve(path)                 // file:// -> path, ~ -> home, relative -> workspace
   fingerprint = toolCallReads.take(toolCallId, resolved)
10 IF resolved is outside the allowed worktrees
               THROW "You cannot search other worktrees for this repository, stay within your workspace paths."
11 IF the fingerprint records a file whose parent directory no longer exists
               THROW "${path} changed on disk after this tool call read it. Nothing was written. Read the file again and retry."
12 IF a permission block reason exists
               userRejected       -> rejected, reason passed through unchanged
               cursorIgnore       -> permissionDenied "Blocked by cursor ignore"
               permissionsConfig  -> permissionDenied; if isReadonly, render the ask-mode refusal
               anything else      -> permissionDenied
13 mkdir -p dirname(realpath(path)); an EACCES or EPERM here becomes permissionDenied
14 IF re-reading the file's fingerprint differs from the one captured before the read
               THROW "${path} changed on disk after this tool call read it. Nothing was written. Read the file again and retry."
               // the last check before the truncate, which is what makes it effective
15 buffer = encodeForWrite(contents, existing format of resolved, platform)
16 fd = open(realpath, WRITE | CREATE | NOFOLLOW | CLOEXEC, 0o666)
               ELOOP, EMLINK, or EPERM with NOFOLLOW
                   -> THROW "Refusing to write through a symlink at ${path}"
   IF realpath(fd) differs from the opened path
                   -> THROW "Refusing write: ${path} now resolves through a symlinked parent to ${real}"
   IF the descriptor is not a regular file
                   -> THROW "Refusing to write to non-regular file at ${path}"
17 truncate(fd, 0); write(fd, buffer) in a loop until the buffer is exhausted; fsync(fd); close(fd)
               ENOSPC -> "No space left on device"
               EDQUOT -> the agent-store quota message
               // no temp file and no rename, so a crash here leaves a half-written file
18 recordFileState(resolved, contents, old)          // old undefined means "delete this file on revert"
19 RETURN success { path, message, linesAdded, linesRemoved, diffString,
                    beforeFullFileContent: old, afterFullFileContent: contents }
```

Helper: the encoding, BOM, and line-ending decision. This is the part reimplementations get wrong, and line-ending rule (c) is why a uniform rewrite is not equivalent.

```
encodeForWrite(contents, existingFile, platform)

1 IF existingFile is absent or zero length
      encoding   = utf8
      lineEnding = platform === "win32" ? CRLF : LF
2 ELSE detect the format of the first 8192 bytes, in this priority order:
      UTF-16LE   FF FE with a non-zero third byte, or more than 80% ascii-plus-zero pairs
      UTF-16BE   FE FF
      UTF-32BE   00 00 FE FF
      UTF-32LE   FF FE 00 00
      binary     any zero byte, or more than 5% of bytes below 0x20 that are not tab, LF, or CR
      utf8       valid UTF-8 with no ESC, unless pure ASCII
      utf8       invalid UTF-8 that still looks like UTF-8 with a few bad bytes
      charset    detected with confidence >= 0.7, else utf-8; a windows-1252 result that
                 decodes with U+FFFD is downgraded to utf-8
      binary     extension shortcut for .png .jpg .jpeg .gif .webp .ico .bmp .svg .pdf .zip
                 .tar .gz .exe .dll .so .dylib .bin and the video extensions, decided
                 without reading the file
3 lineEnding = crlfCount === 0 AND lfCount === 0            -> LF
             : (crlfCount / (crlfCount + lfCount)) * 100 >= 5 -> CRLF
             : otherwise                                    -> LF
4 BOM rules:
      utf8bom                       -> always prepend EF BB BF
      utf8, no explicit hint        -> prepend only if the existing file had a BOM
      utf16le/be, utf32le/be        -> keep the BOM only if the existing file had one
5 line endings, first matching rule wins:
   a  contents already contains "\r\n"   -> leave contents untouched; the model's own
                                             mixed endings win outright
   b  the existing file has mixed endings -> split both sides into lines plus per-line
                                             terminators, line-align them, and give each new
                                             line the terminator of the existing line it
                                             aligned to, defaulting otherwise
   c  detected lineEnding is CRLF         -> rewrite every bare LF not preceded by CR to CRLF
   d  otherwise                           -> leave contents untouched
6 RETURN encode(contentsWithLineEndings, encoding)
   // there is no trailing-newline logic anywhere on this path
```

Helper: the empty-write case, called out because it looks like a refusal and is not.

```
writeEmpty(contents === "")
  // the schema field has no minimum length, so "" validates
  // the file is truncated to 0 bytes and the tool reports success normally
  RETURN message "Wrote contents to ${path}", isError false
```

*2026-10-27 - deepseek-v41-flash-1017*

### Worked examples

The table gives the model-visible text and, where it matters, the bytes left on disk written as escapes: `\n` for LF, `\r\n` for CRLF, `EF BB BF` for a UTF-8 BOM. `(confirmed)` marks behaviour reproduced by probe against the live tool or read directly from the write path. `(derived)` marks a case that follows from a quoted code path without a separate probe.

| Case | Input | Expected output |
|---|---|---|
| ordinary overwrite (confirmed) | existing 84-byte `config.ts`, `contents` is the new full text | `Wrote contents to C:\work\proj\src\config.ts`, `isError: false`, nothing appended |
| new file, missing parents (confirmed) | `path: C:\work\proj\src\deep\a\b\made.ts`, any contents | `Wrote contents to C:\work\proj\src\deep\a\b\made.ts`; all four directories created |
| empty contents on an existing file (confirmed) | `contents: ""` on a 120-byte file | `Wrote contents to C:\work\proj\notes.txt`; file is now 0 bytes; success, not a refusal |
| LF content into a CRLF file (confirmed) | `contents: "one\ntwo\n"` on a file that already uses CRLF | success; disk holds `one\r\ntwo\r\n` |
| LF content into a mixed-ending file (confirmed) | `contents: "mm\nnn\noo\n"` on a file holding `aa\r\nbb\ncc\r\n` | success; disk holds `mm\r\nnn\noo\r\n`, the middle line keeping the existing bare LF |
| content that already contains CRLF (confirmed) | `contents: "one\r\ntwo\n"` on a CRLF file | success; disk holds `one\r\ntwo\n`, because incoming CRLF wins outright and no rewrite runs |
| BOM preserved (confirmed) | `EF BB BF` + CRLF text, `contents` has no BOM | success; disk still begins `EF BB BF`, and lines are CRLF |
| BOM not introduced (confirmed) | BOM-less file, `contents` has no BOM | success; disk still has no BOM |
| no trailing newline in, none out (confirmed) | `contents: "p1\r\np2\r\np3"` | success; disk ends at `p3` with no terminator |
| plan mode, non-markdown (confirmed) | plan mode, `path: C:\work\proj\src\config.ts` | `Plan mode can only edit markdown files and canvas files. This edit was blocked because the target is not allowed in plan-mode. If the user wants implementation changes outside markdown or canvas files, use the SwitchMode tool with target_mode_id=agent first.` |
| permission denied (confirmed) | read-only file on disk | `Write permission denied: C:\work\proj\locked.txt: Permission denied` |
| stale write (confirmed) | file modified between the tool's own read and its truncate | `C:\work\proj\src\config.ts changed on disk after this tool call read it. Nothing was written. Read the file again and retry.` |
| binary target (confirmed) | existing file of `00 01 02 03 00 00 FF` | `Error: File content is binary data, not text: C:\work\proj\data.bin` |
| malformed file URL (confirmed) | `path: "file:///not%20a/url%zz"` | `Error: file:///not%20a/url%zz is not a valid file URL. Use an absolute path instead.` |
| model never read the file (confirmed) | seeded from a shell, never read via the agent, then overwritten | `Wrote contents to C:\work\proj\never-read.txt`; no error, because there is no read gate |

#### LF input written into a CRLF file (confirmed)

The file on disk before the call, captured as a byte dump:

```text
6f 6e 65 0d 0a 74 77 6f 0d 0a          one\r\ntwo\r\n      11 bytes
```

Input, with the JSON escapes making the absence of `\r` explicit:

```json
{"path": "C:\\work\\proj\\notes.txt", "contents": "alpha\nbeta\n"}
```

Model-visible result, exactly:

```text
Wrote contents to C:\work\proj\notes.txt
```

Bytes on disk after the call:

```text
61 6c 70 68 61 0d 0a 62 65 74 61 0d 0a  alpha\r\nbeta\r\n   12 bytes
```

The line ending was imposed by rule 3 of `### Encoding, BOM, and line endings`: the file classified as CRLF, the incoming content held no `\r\n`, so every bare LF became CRLF. An implementation that writes the model's string as UTF-8 bytes unchanged produces an 11-byte file with bare LFs, which is the most common way to get this wrong.

#### Empty contents truncating an existing file (confirmed)

Input:

```json
{"path": "C:\\work\\proj\\notes.txt", "contents": ""}
```

Model-visible result:

```text
Wrote contents to C:\work\proj\notes.txt
```

Bytes on disk, from 12 bytes to 0:

```text
(empty; length 0)
```

`contents` is a plain string with no minimum length, and the write path calls `truncate(0)` before writing zero bytes, so this is a successful truncate rather than a no-op or a validation error. An implementation that skips the write when `contents` is empty leaves the old bytes in place and diverges immediately.

#### Mixed line endings merged per line (confirmed)

The file on disk before the call:

```text
61 61 0d 0a 62 62 0a 63 63 0d 0a      aa\r\nbb\ncc\r\n
```

Input with three new lines, all LF:

```json
{"path": "C:\\work\\proj\\mixed.txt", "contents": "mm\nnn\noo\n"}
```

Model-visible result:

```text
Wrote contents to C:\work\proj\mixed.txt
```

Bytes on disk after the call:

```text
6d 6d 0d 0a 6e 6e 0a 6f 6f 0d 0a      mm\r\nnn\noo\r\n
```

Line 1 took the CRLF of `aa`, line 2 took the bare LF of `bb`, and line 3 took the CRLF of `cc`, because the file had mixed endings and rule 2 ran a line-aligned merge. Note the asymmetry: a uniform CRLF rewrite would produce `mm\r\nnn\r\noo\r\n`, and a uniform LF write would produce `mm\nnn\noo\n`. Neither matches, which is why this case is worth a vector of its own.

What each example proves:

- The CRLF example pins that the tool writes in the file's ending style rather than the model's, so the success message is identical while the bytes differ.
- The empty-contents example pins that an empty write is a successful truncate, which is the case most implementations turn into a silent no-op.
- The mixed-ending example pins the per-line merge, the one rule that cannot be reproduced with any single global replace.

*2026-10-27 - deepseek-v41-flash-1017*

## StrReplace

### At a glance

This edits an existing file by finding an exact piece of text and substituting a replacement, which is the workhorse edit tool for an agent.

A model reaches for it when it wants to change a few lines without rewriting the whole file. It reaches for `WriteFile` instead when creating a file or when the change touches most of the file anyway.

The commonly-misimplemented part: there is no fuzzy matching, no similarity threshold, and no `...` elision. It is an exact substring scan, and only if that finds nothing does a line-trimmed block match run, which requires strict one-to-one line correspondence and only trims leading and trailing whitespace. The exact scan is also non-overlapping, so `"aa"` in `"aaa"` is one match and not two.

Internal id `STR_REPLACE`, offset 4209511, with the edit engine at 4150373. This is the tool where implementations most often diverge, so the pipeline order below is exact and load-bearing.

### Parameters

Default branch:

```js
path:         string, required
    .describe("The absolute path to the file to modify")
old_string:   string, required
    .describe("The text to replace")
new_string:   string, required
    .describe("The text to replace it with (must be different from old_string)")
replace_all:  boolean, optional
    .describe("Replace all occurrences of old_string (default false)")
```

`dsv3-1018` branch uses `file_path` instead of `path` and carries a typo that matters for byte-identical reproduction:

```js
replace_all:  boolean, optional, .default(false)
    .describe("Replace all occurences of old_string (default false)")
```

`occurences` - one r, one s. Reproduce it or do not, but do not do it by accident.

Note `replace_all` has **no** zod default on the default branch, so it arrives as `undefined` and the code normalizes with `const l = !0 === s.replace_all`. Only the `dsv3-1018` branch carries an actual default.

### Parameters that do not exist

Worth stating explicitly, because implementations keep adding them:

- no `explanation`
- no `line_range`
- no `old_str` / `new_str`
- no `target_file`

The `old_str`/`new_str` names exist in the bundle only inside Anthropic-native provider tool schemas, which Cursor does not use for this tool.

### Description text

`cursor-0226` gets a single line and nothing else:

```
Performs exact string replacements in files.
```

The default branch renders as:

```
Performs exact string replacements in files.

Usage:
- When editing text, ensure you preserve the exact indentation (tabs/spaces) as it appears before.
- Only use emojis if the user explicitly requests it. Avoid adding emojis to files unless asked.
- The edit will FAIL if old_string is not unique in the file. Either provide a larger string with more surrounding context to make it unique or use replace_all to change every instance of old_string.
- Use replace_all for replacing and renaming strings across the file. This parameter is useful if you want to rename a variable for instance.
- Optional parameter: replace_all (boolean, default false) - if true, replaces all occurrences of old_string in the file.

If you want to create a new file, use the Write tool instead.
```

The `Write` name is resolved dynamically via `Bj(t, "WRITE")`, which hard-fails if that tool is absent from the set. If you are building a subset harness, keep the reference or substitute a literal.

### The matching pipeline

This is the exact order. Deviating from it produces observable differences.

**Step 0. Normalize inputs.**

- The model-supplied `old_string` and `new_string` have their line endings normalized to LF.
- The file content has its line endings normalized to LF before matching.
- Trailing-newline handling is normalized on both sides.

**Step 1. Empty and identity checks, before any search.**

```js
if (old_string === "")        throw EmptyStringError   // "old_string is empty"
if (normalize(old) === normalize(new))
                              throw IdenticalStringsError
                                                      // "old_string and new_string are exactly the same"
```

Step 1 precedes matching. So `old_string: "  foo"` with `new_string: "foo"` does **not** silently de-indent. Both sides get the same normalization, they compare equal, and it throws `IdenticalStringsError`.

But `old_string: "  foo"` with `new_string: "    foo"` against a file where `foo` sits at zero indent **will** succeed through the whitespace path and write four spaces. Deliberate re-indenting works; accidental no-op does not.

**Step 2. Exact scan.** Repeated `indexOf` with non-overlapping advance:

```js
const matches = [];
let i = 0;
for (; (i = content.indexOf(old_string, i)), -1 !== i; ) {
  matches.push({ start: i, end: i + old_string.length });
  i += old_string.length;          // NOT i + 1
}
if (matches.length > 0) return { matches, usedWhitespaceInsensitive: false };
```

The `i += old_string.length` is the subtle part. Overlapping occurrences are not counted as separate matches. `"aa"` in `"aaa"` yields exactly **one** match at index 0, not two. This is why a separate overlap guard exists in step 6.

**Step 3. Whitespace-insensitive fallback**, only if the exact scan found nothing and the fallback is enabled. See the matcher below.

**Step 4. Zero matches.** Throw:

```
The string to replace was not found in the file.
```

or, if the whitespace fallback also ran and failed:

```
The string to replace was not found in the file (even after relaxing whitespace).
```

and optionally, if a fuzzy suggestion could be built:

```
 Found a possible fuzzy match, did you mean:
<rendered snippet with line numbers>
```

The fuzzy hint uses `hH` to render the nearby region with line numbers. It never affects whether the edit applies; it is advice only.

**Step 5. Multiple matches with `replace_all` false.** Throw:

```
The string to replace was found multiple times in the file. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.
```

with ` after relaxing whitespace` inserted before the period when the fallback found them:

```
The string to replace was found multiple times in the file after relaxing whitespace. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.
```

**Step 6. Overlap guard.** If exactly one match was found by the exact scan, but the needle occurs again starting anywhere after the match start, throw:

```
The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.
```

This catches the self-overlapping case that the non-overlapping scan undercounts.

**Step 7. Splice and write.** Recover offsets into the original content and replace only the matched region, so untouched lines keep their exact bytes. Then write through the same write executor Write uses, and restore the file's original line endings.

### The whitespace matcher, exactly

This is narrower than "whitespace insensitive" implies. In full:

```js
function whitespaceBlockMatch(text, search, eol = "\n") {
  const crlf = eol === "\r\n";
  const toLF = (s) => s.replace(/\r\n/g, "\n");

  search = toLF(search);
  const searchLines = toLF(text).split("\n").map((l) => l.trim());
  if (searchLines.length === 0) return [];

  const fileLines = text.split("\n");            // raw
  const fileTrimmed = fileLines.map((l) => l.trim());
  const lineStart = [];                          // char offset of each raw line
  let off = 0;
  for (const l of fileLines) { lineStart.push(off); off += l.length + (crlf ? 2 : 1); }

  const out = [];
  outer: for (let i = 0; i <= fileLines.length - searchLines.length; i++) {
    let startTrim = 0, endTrim = 0;
    for (let n = 0; n < searchLines.length; n++) {
      const trimmed = fileTrimmed[i + n], raw = fileLines[i + n];
      const isFirst = n === 0, isLast = n === searchLines.length - 1;
      let ok = trimmed === searchLines[n];

      if (isFirst) {
        ok = trimmed.endsWith(searchLines[n]);
        if (ok) { startTrim = raw.lastIndexOf(searchLines[n]); if (trimmed === searchLines[n]) startTrim = 0; }
      }
      if (isLast) {
        if (isFirst && ok && trimmed !== searchLines[n]) continue outer;
        ok = trimmed.startsWith(searchLines[n]);
        if (ok) { endTrim = raw.indexOf(searchLines[n]) + searchLines[n].length; if (trimmed === searchLines[n]) endTrim = raw.length; }
      }
      if (!ok) continue outer;
    }
    out.push({
      startCharIndexZeroIndexed: lineStart[i] + startTrim,
      endCharIndexExclusiveZeroIndexed: lineStart[i + searchLines.length - 1] + endTrim
    });
  }
  return out;
}
```

What this normalizes, and nothing more:

| Transformation | Present |
|---|---|
| Strip leading whitespace per line | Yes, via `trim()` |
| Strip trailing whitespace per line | Yes, via `trim()` |
| Tab/space equivalence | Yes, incidentally - both vanish under trim |
| CRLF to LF before splitting | Yes on the search side and the trimmed copy |
| Collapse internal whitespace runs | **No** |
| Unicode NFC normalization | **No** |
| Smart quote folding | **No** |
| Levenshtein or similarity threshold | **No** |

The important structural constraint: **strict one-to-one line correspondence**. The search must have the same number of lines as the region it matches. There is no `...` elision, no partial-block matching, no line-anchor search. That is why a multi-line `old_string` missing one intermediate line simply fails.

Because offsets are recovered into the *original* string and only the matched region is spliced, untouched lines keep their exact bytes. There is no re-indentation of the whole file. This is the correct design and worth copying deliberately.

### replace_all

Two code paths:

- **Bulk splice**, when the replacement is non-empty and there are two or more matches. Splices in descending offset order so earlier offsets stay valid.
- **Sequential loop** otherwise, including the deletion case, with line-eating behavior for empty `new_string`.

The reported count is the number of matches actually replaced by the scan, which reflects non-overlapping semantics. `"aa"` replaced with `"X"` in `"aaa"` yields `"Xa"` and reports one replacement, not two.

### CRLF handling end to end

Three places, and this is why an LF `old_string` works against a CRLF file:

1. Model-supplied strings have line endings normalized to LF.
2. File content is normalized to LF for matching.
3. The write layer restores the file's original ending, either dominant or per-line.

So a model that sends LF against a CRLF file exact-matches in step 2 with no fallback needed.

### Success text

There is no snippet, no line numbers, and no `diffStat` trailer. `diffStat` does not appear anywhere in the bundle. `linesAdded` and `linesRemoved` are proto fields, not appended text. The model sees the tool's message field and nothing else.

The only place a model ever sees line numbers from this tool is the failure hint.

### Failure modes

Thrown before matching:

| Condition | Model text | Client text |
|---|---|---|
| `old_string === ""` | `old_string is empty` | `Incorrect tool arguments` |
| normalized old equals new | `old_string and new_string are exactly the same` | `Incorrect tool arguments` |

Thrown during matching:

| Condition | Model text |
|---|---|
| zero matches | `The string to replace was not found in the file.` |
| zero matches after fallback | `The string to replace was not found in the file (even after relaxing whitespace).` |
| multiple, `replace_all` false | `The string to replace was found multiple times in the file. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` |
| overlapping occurrences | `The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.` |

Thrown by the read stage, meaning the edit never ran:

| Condition | Model text |
|---|---|
| file not found | `File not found` |
| permission denied | `Read permission denied: ${path}` |
| invalid file | `${reason ?? "Path is not a valid file to read"}: ${path}` |
| binary | `File is not a text file (binary data)` |
| file too large to read fully | `File is too large to edit (${bytes} bytes): only part of it could be read, so writing the edit would truncate it. Edit it with a shell command instead.` |
| rejected | `Edit rejected: ${reason}` |

Note the two-sided messaging on zero matches. The model sees the long form with the fuzzy hint; the UI toast sees the fixed short form `The string to replace was not found in the file.` Two different strings from the same throw.

### No read-before-write gate

There is no gate, and no error string for one. The requirement is prompt text only:

```
You MUST use the Read tool at least once before editing.
```

emitted by the generic system prompt at offset 2750288. At runtime the tool re-reads the file itself and edits fresh content, so a gate would be pointless. If you add one, you are adding behavior Cursor does not have.

The closest real safeguard is the freshness read in the tool's own read stage, which is why a stale edit fails with `File is too large to edit` or a genuine not-found rather than a stale-write error.

### Algorithm pseudocode

Two layers: a read-then-edit wrapper that owns the lock and the write, and a pure edit engine over a string. The engine's step order is the contract, because step 2 fires before the search and step 7 fires after it.

```
TOOL StrReplace

INPUTS
  args       parsed arguments: { path | file_path, old_string, new_string, replace_all }
  ctx        request context: mode, workspace paths, stateHandler
  service    capability service: read(path) and write(path, text) -> oneof results

OUTPUT
  EditToolCall record, case success carrying { path, message, linesAdded, linesRemoved, diffString }

1  PARSE      validate args against the version schema
               on failure THROW "Invalid arguments:\n<key>: <message>"
2  streamed   acquire dispose = lockManager.waitForLock(ctx, rawPath) as soon as the path field
               completes in the argument stream
               IF the stream ends with no lock acquired
                   THROW "Failed to acquire file lock - path may not have been parsed"
3  path       = args.path ?? args.file_path
   replaceAll = args.replace_all === true                      // no schema default on this branch
4  IF mode is PLAN AND path is not markdown and not a canvas path
                   THROW "Cannot edit non markdown files in plan mode"
5  read      = service.read(path)
6  SWITCH read.case
             fileNotFound     -> THROW "File not found"
             permissionDenied -> THROW "Read permission denied: ${path}"
             invalidFile      -> THROW (reason || "Path is not a valid file to read") + ": " + path
             rejected         -> THROW "Edit rejected: ${reason}"
             error            -> THROW the service text, except a resolution failure which
                                 passes its own message through
             success:
               IF output is not text  -> THROW "File is not a text file (binary data)"
               IF the read was truncated
                   THROW "File is too large to edit"
                         + (bytes > 0 ? " (${bytes} bytes)" : "")
                         + ": only part of it could be read, so writing the edit would truncate it. Edit it with a shell command instead."
7  original    = output.value
   contentsLF  = original with "\r\n" -> "\n"
8  edited      = editEngine(contentsLF, old_string, new_string, replaceAll, eol = "\n", fallback = true)
9  diff        = computeDiff(original, edited.contentsAfterEdit, "modified", tool = "str_replace")
10 message     = replaceAll
                   ? "${n} " + plural(n, "occurrence", "occurrences") + " in file ${path} "
                     + (n === 1 ? "has" : "have") + " been updated."
                   : "The file ${path} has been updated."
11 write      = service.write(path, edited.contentsAfterEdit)
12 SWITCH write.case
             success          -> recordFileState(path, newContents, original); RETURN success record
             permissionDenied -> THROW isReadonly ? the ask-mode refusal
                                              : "Write permission denied: ${path}" (+ ": ${error}" when present)
             noSpace          -> THROW "No space left on device"
             rejected         -> THROW "Edit rejected: ${reason}"
             error            -> THROW the service text
```

Helper: the edit engine. Offsets are recovered into the original string and only the matched region is spliced, so untouched lines keep their exact bytes.

```
editEngine(content, oldRaw, newRaw, replaceAll, eol, fallback)

1 newStr = newRaw with "\r\n" -> "\n"                           // model strings normalized to LF
  oldStr = oldRaw with "\r\n" -> "\n"
2 IF oldRaw === ""  THROW "old_string is empty"                 // before any search runs
3 IF oldStr === newStr  THROW "old_string and new_string are exactly the same"
4 { matches, usedWhitespaceInsensitive } = findMatches(content, oldStr, fallback, eol)
5 IF matches is empty
      hint = buildDidYouMean(oldStr, split(content, eol))       // advice only, never applied
      message = "The string to replace was not found in the file"
              + (usedWhitespaceInsensitive ? " (even after relaxing whitespace)" : "") + "."
              + (hint ? " Found a possible fuzzy match, did you mean:\n" + renderNumberedLines(hint) : "")
      THROW message
      // the client-visible half of this same throw is always the short fixed form with a period
6 IF count(matches) > 1 AND replaceAll is false
      THROW "The string to replace was found multiple times in the file"
            + (usedWhitespaceInsensitive ? " after relaxing whitespace" : "") + ". "
            + "Use replace_all to replace all occurrences, or include more context to only edit one occurrence."
7 IF replaceAll is false AND usedWhitespaceInsensitive is false
  AND content.indexOf(oldStr, matches[0].start + 1) !== -1      // the +1 is what catches self-overlap
      THROW "The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence."
8 RETURN { contentsAfterEdit: splice(content, matches, newStr),
           usedWhitespaceInsensitive,
           occurrencesModified: count(matches) }
```

Helper: the match finder, which shows why a self-overlapping needle undercounts.

```
findMatches(content, search, enabled, eol)

1 IF content === ""  RETURN search === "" ? { matches: [{ start: 0, end: 0 }], used: false }
                                          : { matches: [], used: false }
2 IF search === ""   log an internal error, RETURN { matches: [], used: false }
3 hits = []; i = 0
  LOOP  i = content.indexOf(search, i)
        IF i === -1 BREAK
        hits.push({ start: i, end: i + length(search) })
        i = i + length(search)                                 // NOT i + 1
4 IF hits is non-empty  RETURN { matches: hits, used: false }
5 IF enabled is false   RETURN { matches: [], used: false }
6 RETURN { matches: whitespaceBlockMatch(search, content, eol), used: true }
```

Helper: the trimmed-line matcher. Strips per-line whitespace, requires strict one-to-one line correspondence, and does nothing more.

```
whitespaceBlockMatch(search, content, eol)

1 crlf   = (eol === "\r\n")
  search = search with "\r\n" -> "\n"
  needle = map(split(search, "\n"), trim)
  IF needle is empty  RETURN []
2 fileLines = split(content, "\n")                             // raw, untrimmed
  fileTrim  = map(fileLines, trim)
  lineStart = char offset of each raw line, accumulating length + (crlf ? 2 : 1)
3 out = []
  FOR window = 0 TO count(fileLines) - count(needle)           // inclusive bound
      startOff = 0; endOff = 0
      FOR n = 0 TO count(needle) - 1
          ft = fileTrim[window + n]; fr = fileLines[window + n]
          isFirst = (n === 0); isLast = (n === count(needle) - 1)
          ok = (ft === needle[n])
          IF isFirst
              ok = endsWith(ft, needle[n])
              IF ok  startOff = lastIndexOf(fr, needle[n]); IF ft === needle[n] startOff = 0
          IF isLast
              IF isFirst AND ok AND ft !== needle[n]  CONTINUE outer   // reject a partial single line
              ok = startsWith(ft, needle[n])
              IF ok  endOff = indexOf(fr, needle[n]) + length(needle[n]); IF ft === needle[n] endOff = length(fr)
          IF NOT ok  CONTINUE outer
      out.push({ start: lineStart[window] + startOff,
                 end:   lineStart[window + count(needle) - 1] + endOff })
4 RETURN out
```

Helper: the splice, including line eating when the replacement is empty.

```
splice(content, matches, replacement)

1 desc = matches sorted by start descending
2 IF replacement !== "" AND count(desc) >= 2                   // bulk path
      walk desc building slices from the end; on any inconsistent offset RETURN undefined
      RETURN the joined result
3 out = content                                                // sequential path, including deletion
  FOR m in desc
      a = m.start; b = m.end
      IF replacement === ""
          lineStart = lastIndexOf(out, "\n", a - 1) + 1
          IF out[lineStart ..< a] is all whitespace  a = lineStart     // eat the leading indent
          lineEnd = indexOf(out, "\n", b)
          IF lineEnd !== -1 AND out[b ..< lineEnd] is all whitespace  b = lineEnd + 1
          ELSE IF lineEnd === -1 AND out[b ..] is all whitespace
              b = length(out)
              IF a > 0 AND out[a-1] === "\n"  a -= 1
              IF a > 0 AND out[a-1] === "\r"  a -= 1
      out = out[0 ..< a] + replacement + out[b ..]
4 RETURN out
```

### Worked examples

Failure rows show the text after the renderer's `Error: ` prefix, which the renderer adds and the thrown message does not contain. Success rows show the message field verbatim with `isError: false`.

| Case | Input | Expected output |
|---|---|---|
| ordinary exact match (confirmed) | `greet` to `welcome` in `util.ts`, one occurrence, `replace_all` absent | `The file C:\work\proj\src\util.ts has been updated.` |
| empty old_string (confirmed) | `old_string: ""` | `Error: old_string is empty` |
| de-indent attempt (confirmed) | `old_string: "  foo"`, `new_string: "foo"` | `Error: old_string and new_string are exactly the same` |
| deliberate re-indent (derived) | `old_string: "  foo"`, `new_string: "    foo"`, file has `foo` at column 0 | success; the region is replaced with four spaces, through the whitespace path |
| zero matches, token absent (confirmed) | `old_string: "zzzzQqqq_absent_token"` | `Error: The string to replace was not found in the file (even after relaxing whitespace).` |
| zero matches in an empty file (derived) | any non-empty `old_string`, file is 0 bytes | `Error: The string to replace was not found in the file.` with no relaxed suffix, because the empty-text guard returns before the fallback runs |
| two exact matches, `replace_all` false (confirmed) | `old_string: "return null;"` twice in one file | `Error: The string to replace was found multiple times in the file. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` |
| two matches only after relaxation (confirmed) | tab-indented needle matching two space-indented regions | `Error: The string to replace was found multiple times in the file after relaxing whitespace. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` |
| `aa` in `aaa`, `replace_all` false (confirmed) | one non-overlapping match, a second at offset 1 | `Error: The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.` |
| `aa` in `aaa`, `replace_all` true (confirmed) | `new_string: "X"` | `1 occurrence in file C:\work\proj\src\util.ts has been updated.`; file becomes `Xa` |
| three occurrences, `replace_all` true (confirmed) | `console.log` to `logger.info` three times | `3 occurrences in file C:\work\proj\src\util.ts have been updated.` |
| LF needle against a CRLF file (confirmed) | `old_string: "return 1;"`, file uses `\r\n` | success, exact path, `usedWhitespaceInsensitive` false |
| tab-indented needle, space-indented file (confirmed) | `old_string` starts `\tif (x) {`, file uses four spaces | success through the whitespace fallback |
| whole-line deletion (confirmed) | file `line1\nline2\nline3\nline4`, `old_string: "line2"`, `new_string: ""` | success; file becomes `line1\nline3\nline4`, both surrounding newlines eaten |
| needle is the whole file (derived) | `old_string` equals the entire content, `new_string: ""` | success; file becomes 0 bytes |

#### Tab-indented needle against a space-indented file (confirmed)

The file on disk before the call:

```text
function foo() {
    if (x) {
        return 1;
    }
}
```

Input, where `\t` and `\n` are written as escapes here but arrive as literal characters:

```json
{"path": "C:\\work\\proj\\src\\foo.ts", "old_string": "\tif (x) {\n\t\treturn 1;\n\t}", "new_string": "\tif (y) {\n\t\treturn 2;\n\t}"}
```

Model-visible result:

```text
The file C:\work\proj\src\foo.ts has been updated.
```

The file on disk after the call:

```text
function foo() {
    if (y) {
        return 2;
    }
}
```

The exact scan found nothing, because there is not a tab anywhere in the file. The fallback trimmed per-line leading and trailing whitespace on both sides, matched three lines to three lines, recovered character offsets into the original text, and spliced only the matched region. Note that the replacement landed with the indentation the model supplied, tab for tab, not with the file's four spaces: there is no re-indentation of `new_string`. An implementation that rewrote the whole file from trimmed lines would have re-indented the untouched lines too.

#### `aa` in `aaa`, both `replace_all` settings (confirmed)

File content:

```text
aaa
```

Input with `replace_all` false:

```json
{"path": "C:\\work\\proj\\src\\util.ts", "old_string": "aa", "new_string": "X", "replace_all": false}
```

Model-visible result:

```text
Error: The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.
```

Input with `replace_all` true:

```json
{"path": "C:\\work\\proj\\src\\util.ts", "old_string": "aa", "new_string": "X", "replace_all": true}
```

Model-visible result:

```text
1 occurrence in file C:\work\proj\src\util.ts has been updated.
```

File on disk after the second call:

```text
Xa
```

The scan advances by `old_string.length`, so it records one match at offset 0 and then resumes at offset 2, where `aa` does not fit. With `replace_all` false that single match trips the overlap guard, which is the only thing standing between the model and a silent partial edit. With `replace_all` true the splice runs on that one match and produces `Xa`. A `String.prototype.replaceAll` implementation produces `XX`, and a scan that advanced by one would report two matches and produce `Xa` only after throwing on the ambiguity check first.

#### Deleting one line in the middle of a file (confirmed)

File content before the call:

```text
line1
line2
line3
line4
```

Input:

```json
{"path": "C:\\work\\proj\\src\\data.txt", "old_string": "line2", "new_string": ""}
```

Model-visible result:

```text
The file C:\work\proj\src\data.txt has been updated.
```

File content after the call, three lines and three terminators:

```text
line1
line3
line4
```

The deletion path checks whether anything other than whitespace shares the match's line, finds nothing, and so swallows the preceding newline as well. Without that cleanup the result would be `line1\n\nline3\nline4`, a blank line where the deleted line used to be. That cleanup only applies when `new_string` is empty, and it is why an empty replacement behaves differently from a replacement of `""` plus surrounding context.

What each example proves:

- The tab example pins that the fallback is line-trimmed and one-to-one, that it recovers offsets into the original text, and that the replacement is not re-indented.
- The `aa` example pins the non-overlapping advance, the overlap guard, and the fact that `replace_all` on a self-overlapping needle replaces once.
- The deletion example pins the empty-replacement line cleanup, which is separate logic from the bulk splice used for non-empty `replace_all`.

*2026-10-27 - deepseek-v41-flash-1017*

## Delete

### At a glance

This removes a file from disk permanently, with no intermediate step and no recovery unless the harness's own undo layer recorded the content first.

A model reaches for it when a file is genuinely obsolete. It does not reach for it to rename or move, and it should not use a shell command for deletion when this tool exists, because this one goes through the permission gate and records undo state.

Two things to note. It unlinks, meaning it removes the directory entry permanently, and never moves anything to the trash or recycle bin. And its error text is malformed by a serialization bug, producing a doubled prefix with an empty slot such as `Error deleting file : File not found: C:\a.txt`, which you should decide about deliberately rather than inherit by accident.

Internal id `DELETE`, offset 4178003.

### Parameters

Default branch (`pSe`):

```js
path: string, required
    .describe("The absolute path of the file to delete")
```

`dsv3-1018` branch (`mSe`):

```js
target_file: string, required
    .describe("The path of the file to delete, relative to the workspace root.")
explanation: string, optional
    .describe("One sentence explanation as to why this tool is being used, and how it contributes to the goal.")
```

The richer schema is on the *older* branch, which is the opposite of what you would guess. The code branches on `"path" in args ? args.path : args.target_file`.

### Description text

```
Deletes a file at the specified path. The operation will fail gracefully if:
    - The file doesn't exist
    - The operation is rejected for security reasons
    - The file cannot be deleted
```

Note the four-space bullet indents, which are part of the text.

`gpt5-codex` and `codex-cloud` get a one-liner instead:

```
Delete file at specified path relative to workspace root; fails gracefully if file doesn't exist, security rejection, or undeletable.
```

### Algorithm

In order:

1. Trim-check the path. An empty path is refused before any I/O.
2. Resolve the path: strip a `file://` prefix, expand `~`, resolve against the workspace root.
3. Write-permission gate, with approval prompting disabled for this check.
4. `realpath` the **parent** only, not the leaf, so a symlink leaf is deleted as a link rather than its target.
5. Request approval if required by policy.
6. `stat` the path. If it is not a file, refuse with `notFile` and report the actual type.
7. Read `prevContent` **only** if the file is at or under 50 MiB, for undo.
8. Re-`stat` and compare `dev` and `ino` against the earlier stat. This is a TOCTOU guard (time of check to time of use): if the file was replaced between the read and the unlink, refuse.
9. `unlink`. Special-case `EACCES` and `EBUSY`. `EPERM` is **not** special-cased and surfaces as a generic error.

### Result text

Success:

```
Successfully deleted file: ${path} (${bytes} bytes)
```

Note the byte count in parentheses, and that `deletedFile` always equals the path actually removed.

### It unlinks permanently

It never moves to trash. There is no recycle bin, no backup file, no staged deletion. If you want recoverability, that is entirely on your undo layer via `recordFileState`.

### Failure modes

| Case | Model text |
|---|---|
| not found | `File not found` |
| not a file | `Path is not a file (${actualType}): ${path}` |
| permission denied | `Permission denied: ${path}` |
| readonly (ask mode) | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` |
| busy | `File is busy: ${path}` |
| rejected | `File deletion rejected: ${reason}` |
| other error | `Error deleting file ${path}: ${error}` |

### A real bug worth knowing about

The text the model sees is not the text that was thrown. `serializeError` rebuilds the message without the `path` field, and `render` then interpolates that now-empty path. The result is a doubled prefix with an empty slot:

```
Error deleting file : File not found: C:\a.txt
```

If you are cloning this behavior, decide deliberately. I would fix it and note the divergence, since nothing depends on the malformed string.

Second quirk: an outer wrapper forces `isError: true` on every failure path, even where `render` returns false. So a failed delete always presents as an error, regardless of what the individual render case returns.

### Symlink behavior, and an inconsistency

- **Deleting a symlink to a file** removes the link but captures the *target's* content as `prevContent`. The undo snapshot therefore describes a different file than the one that was removed.
- **A broken symlink is undeletable**, because `stat` returns `ENOENT` and the tool refuses before reaching `unlink`. You would need `symlink_metadata` to handle this correctly.
- **Symlinks in parent components are followed** via `realpath`.

The first is a genuine correctness bug, not cosmetic. If you implement undo, snapshot the link itself.

### Undo bookkeeping

```js
recordFileState(path, undefined, prevContent)
```

stores `content: undefined`, which is the "this file is gone" marker, and `initialContent: prevContent`. It only writes when no earlier entry exists, which preserves the session-start revision across a delete. Binary, oversized, and unreadable files store no undo content at all, so they are unrecoverable.

### Algorithm pseudocode

Two layers again: a tool side that maps result cases to throws, and an executor that touches disk. The mapping layer throws for six of the seven cases, and `serializeError` then rebuilds a result from the throw, which is why the model-visible text is not the thrown text.

```
TOOL Delete

INPUTS
  args       parsed arguments: { path } or { target_file, explanation? }
  ctx        request context: abort signal, workspace root, stateHandler, approval provider
  service    capability service: delete(path) -> oneof result

OUTPUT
  DeleteToolCall record, oneof { success | fileNotFound | notFile | permissionDenied
                                 | fileBusy | rejected | error }

1  PARSE       validate args against the version schema
                on failure THROW "Invalid arguments:\n<key>: <message>"
2  rawPath   = "path" in args ? args.path : args.target_file     // the in test, not a fallback chain
3  dispose   = lockManager.waitForLock(ctx, rawPath)             // raw string, before any stat
4  result    = service.delete(rawPath, execId = uuid5(toolCallId))
5  SWITCH result.case
              "success"          -> recordFileState(result.path, undefined, result.prevContent); RETURN result
              "fileNotFound"     -> THROW "File not found: ${result.path}"
              "notFile"          -> THROW "Path is not a file (${result.actualType}): ${result.path}"
              "permissionDenied" -> THROW (permissionDenied, path: result.path, isReadonly: result.isReadonly)
              "fileBusy"         -> THROW "File is busy: ${result.path}"
              "rejected"         -> THROW result.reason || "File deletion rejected"
              "error"            -> THROW "Error deleting file ${result.path}: ${result.error}"
              otherwise          -> THROW "Unexpected result case: ${result.case}"
6  dispose
```

Helper: `serializeError`, which is where the doubled prefix comes from. Every throw in step 5 is converted back into a result record, and the path field does not survive for three of the cases.

```
serializeError(error)

1 IF error is a permissionDenied error
      RETURN result { case: "permissionDenied", path: error.path, isReadonly: error.isReadonly }
      // clientVisibleError is dropped here, so the specific reason is lost
2 IF error is a rejected error
      RETURN result { case: "rejected", reason: error.message }
      // path is dropped and defaults to ""
3 RETURN result { case: "error", error: error.message }
      // path is dropped and defaults to ""
```

Helper: `render`, applied to the rebuilt record. The `path` interpolation is what produces the empty slot.

```
render(record)

  "success"          -> "Successfully deleted file: ${path} (${Number(fileSize)} bytes)"
  "fileNotFound"     -> "File not found: ${path}"
  "notFile"          -> "Path is not a file (${actualType}): ${path}"
  "permissionDenied" -> isReadonly ? ASK_MODE_REFUSAL : "Permission denied: ${path}"
  "fileBusy"         -> "File is busy: ${path}"
  "error"            -> "Error deleting file ${path}: ${error}"
  "rejected"         -> reason !== "" ? "File deletion rejected: ${reason}"
                                      : "File deletion rejected: ${path}"
  undefined          -> "Unknown error"
  // isError is false from render for every case; an outer wrapper forces isError true
  // whenever the call threw and was serialized
```

Helper: the executor order, which is where the disk behavior lives.

```
executorDelete(rawPath)

1  IF trim(rawPath) === ""  RETURN error "path is empty or only whitespace"
2  resolved = resolve(rawPath)              // file:// strip, ~ expansion, workspace-relative resolve
3  IF a write-permission block reason exists
       needsApproval     -> no decision, fall through        // approval is deliberately not interactive
       userRejected      -> RETURN rejected, reason
       cursorIgnore      -> RETURN permissionDenied "Blocked by cursor ignore"
       permissionsConfig -> RETURN permissionDenied, isReadonly from the reason
       anything else     -> RETURN permissionDenied with the mapped message
4  parentReal = realpath(dirname(resolved))
   displayPath = parentReal exists AND is inside the workspace
                   ? workspace-relative rebase
                   : join(parentReal ?? resolved, basename(resolved))
   // only the parent is realpathed, so a symlink at the leaf is unlinked as a link
5  IF approval is denied  RETURN rejected { path: resolved, reason: reason || "" }
6  st = stat(resolved)                    // follows symlinks
       ENOENT            -> RETURN fileNotFound
       any other failure -> RETURN error with the raw OS message
7  IF NOT st.isFile()
       RETURN notFile { actualType: st.isDirectory() ? "directory" : "other" }
       // only ever "directory" or "other"; there is no "symlink" or "device" value
8  prev = ""
   IF st.size <= 52428800
       TRY prev = readAsText(resolved) with CRLF -> LF
       CATCH ignore                       // binary, unreadable, or oversized all yield ""
9  st2 = stat(resolved) (errors swallowed)
   IF st2 is undefined OR st2.dev !== st.dev OR st2.ino !== st.ino
       RETURN error "File changed while deleting; nothing was removed"
10 TRY unlink(resolved)
       EACCES -> RETURN permissionDenied { clientVisibleError: "Permission denied" }  // isReadonly false
       EBUSY  -> RETURN fileBusy
       EPERM  -> NOT special-cased, falls through to error
       other  -> RETURN error with the raw OS message
11 IF prev was actually read  trackChange(resolved, prev, undefined, { toolCallId })
12 RETURN success { path: resolved, deletedFile: resolved, fileSize: st.size, prevContent: prev }
```

### Worked examples

The failure rows below are the **post-round-trip** strings, meaning what the model actually receives after `serializeError` strips the path field and `render` re-templates. That is the doubled-prefix form documented in `### A real bug worth knowing about`, and it is the string to test against, not the cleaner text that was thrown.

| Case | Input | Expected output |
|---|---|---|
| ordinary success (confirmed) | `path: C:\work\proj\old\notes.txt`, a 1234-byte text file | `Successfully deleted file: C:\work\proj\old\notes.txt (1234 bytes)` |
| one-byte file (derived) | a 1-byte file | `Successfully deleted file: C:\work\proj\a.txt (1 bytes)`, with no singular form |
| relative path (derived) | `path: "old/notes.txt"` with the workspace root at `C:\work\proj` | `Successfully deleted file: C:\work\proj\old\notes.txt (1234 bytes)`; the message reports the resolved path, not the argument |
| file not found (confirmed) | `path: C:\work\proj\src\gone.ts` | `Error deleting file : File not found: C:\work\proj\src\gone.ts` |
| directory (confirmed) | `path: C:\work\proj\src` | `Error deleting file : Path is not a file (directory): C:\work\proj\src` |
| empty directory (derived) | `path: C:\work\proj\empty` | same shape, `(directory)`, because the test is on file type and not on emptiness |
| permission denied (derived) | `path: C:\work\proj\locked.txt`, unlink returns `EACCES` | `Permission denied: C:\work\proj\locked.txt`; the `uSe` branch of `serializeError` rebuilds the result with the path intact, unlike the generic `error` branch |
| ask mode (confirmed) | same refusal with `isReadonly: true` | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` |
| busy (confirmed) | unlink returns `EBUSY` | `Error deleting file : File is busy: C:\work\proj\locked.txt` |
| rejected with a reason (confirmed) | the approval card is declined, reason `nope` | `File deletion rejected: nope` |
| rejected with no reason (derived) | declined with an empty reason | `File deletion rejected: File deletion rejected`, because the empty reason is replaced by the literal fallback before `render` prefixes it again |
| empty path (derived) | `path: "   "` | `Error deleting file : path is empty or only whitespace` |
| broken symlink (derived) | `path: C:\work\proj\dangling`, a link whose target is gone | `Error deleting file : File not found: C:\work\proj\dangling`, because `stat` follows the link and returns `ENOENT` |
| `EPERM` on unlink (confirmed) | Windows read-only attribute set | `Error deleting file : Error deleting file C:\work\proj\ro.txt: EPERM: operation not permitted, unlink '...'`, the doubled form twice over |
| symlink to a file (derived) | `path: C:\work\proj\link.txt`, a link to `real.txt` | success with the target's byte count; the link is removed and `real.txt` survives |

#### Deleting an ordinary file (confirmed)

Input:

```json
{"path": "C:\\work\\proj\\old\\notes.txt"}
```

File on disk before the call, 34 bytes:

```text
# Scratch notes
TODO: delete this later
```

Model-visible result, one text part with `content[0].type === "text"`:

```text
Successfully deleted file: C:\work\proj\old\notes.txt (34 bytes)
```

The byte count comes from the first `stat`, taken before the read attempt and before the unlink, so it describes the file as it was found. There is no truncation, no move to a temporary name, and no recovery path: `unlink` removes the directory entry. The `prevContent` snapshot that makes undo possible is in memory only and is bounded to 50 MiB of text. An implementation that reports a count from the post-delete filesystem, or that omits the count, produces a different string.

#### Deleting a file that does not exist, and the doubled prefix (confirmed)

Input:

```json
{"path": "C:\\work\\proj\\src\\gone.ts"}
```

Model-visible result, byte for byte:

```text
Error deleting file : File not found: C:\work\proj\src\gone.ts
```

Count the space before the colon: `Error deleting file` then a space then `:` then a space then `File not found:`. That empty slot is the path field that `serializeError` dropped before `render` interpolated it. The thrown message was the clean `File not found: C:\work\proj\src\gone.ts`, so a reimplementation that tests only the thrown text will pass its own unit test and fail against the real tool.

#### Deleting a directory (confirmed)

Input:

```json
{"path": "C:\\work\\proj\\src"}
```

Model-visible result:

```text
Error deleting file : Path is not a file (directory): C:\work\proj\src
```

`stat` reports a directory, so the tool returns `notFile` with `actualType: "directory"` before reaching `unlink`. There is no recursive mode and no `rm -r` fallback, and an empty directory is refused on exactly the same grounds. `actualType` only ever takes the values `directory` or `other`, so a socket or device reports `other` rather than naming its type. Note also that the outer wrapper forces `isError: true` on every failure path, so this arrives as an error even though `render` returned its text with the default `isError: false`.

What each example proves:

- The success example pins the exact success template, including the space before the parenthesis, the literal word `bytes` with no separator, and the count sourced from the pre-delete stat.
- The not-found example pins the doubled prefix, which is the single most likely thing to get wrong, and shows that it is real in the model-visible string rather than only in a log.
- The directory example pins that directories are refused rather than removed, and that the refusal names the actual type.

*2026-10-27 - deepseek-v41-flash-1017*

## Glob

### At a glance

This finds files by path pattern and returns them sorted newest first, so the model can locate a file whose name it roughly remembers without knowing where it lives.

A model reaches for it when it knows something about a filename or extension but not the directory. It reaches for `Grep` instead when it knows something about file contents, and for `ListDir` when it wants to see the shape of one directory.

The part that surprises people: this is not a glob library. There is no `fast-glob`, no `picomatch`, no `minimatch` on this path. It is ripgrep running in file-listing mode with an empty pattern, which means patterns are matched by ripgrep's own glob engine and are case-insensitive because the flag emitted is `--iglob` rather than `--glob`.

Internal id `GLOB`, offset 4277336.

### Parameters

Both schema variants (`REe` for most versions, `MEe` for `dsv3-1018`) have exactly two fields, and differ by one word:

```js
glob_pattern:     string, required
target_directory: string, optional
    .describe("Absolute path to the directory to search in (rg pattern -- PATH). Defaults to Cursor workspace root.")
```

The `dsv3-1018` variant says `Cursor workspace roots` (plural) plus a trailing newline. That is the entire difference.

**Parameters that do not exist:** no `pattern`, no `case_sensitive`, no `max_results`. Every one of those is a common guess and every one is wrong.

### Name per branch

Only `dsv3-1018` renames it:

```js
const name = version === "dsv3-1018" ? "glob_file_search" : "Glob";
```

Every other family, including `latest`, `cursor-0226`, `dsv3-1205`, `gpt5-codex`, `codex-cloud`, and `haiku`, gets `Glob`.

### Description text

Byte-identical across all seven branches, from a single generator with four bullets. Notably it includes:

```
Returns matching file paths sorted by modification time
```

This claim is **true**, which is worth flagging because the same claim in Claude Code's description is misleading about direction. Cursor sorts newest-first; Claude Code sorts oldest-first.

### Engine: ripgrep in `--files` mode

Glob does not use a glob library. There is no `fast-glob`, no `picomatch`, no `minimatch`, and nothing from `tree-chunk` on this path. It uses the same `LocalGrepExecutor` the Grep tool uses, with `outputMode: "files_with_matches"` and an **empty pattern**, which trips the `--files` branch in `buildArgs` at offset 9018917.

The exact argv (argument vector, the ordered list of strings handed to the process), in order:

```
--cursor-ignore <file> ...   (repeated pairs)
--files
--case-sensitive
--iglob <pattern>
--iglob !**/.git
--no-config
--color=never
--hidden
--follow
--
<relative target>
```

Points that matter:

- `--iglob`, not `--glob`. Glob patterns are **case-insensitive** by default.
- `!.git` is excluded by an explicit `!**/.git` glob, not by an ignore rule.
- `--hidden` and `--follow` are always on. Dotfiles are included; symlinks are followed.
- `--no-config` prevents a user ripgrep config from changing behavior.
- No sort flag. Sorting happens in JS afterward.
- No `--regexp`, because there is no pattern in listing mode.

### Ignore behavior

`.gitignore` is respected by default. Nothing ever pushes `--no-ignore` on this path.

This was verified empirically rather than inferred: on a real tree, the default run returned 5,630 paths and `--no-ignore` returned 8,461. The difference is the ignored set.

`.cursorignore` is applied separately, through the `--cursor-ignore` pairs and a post-filter (`filterBlockedFiles`), not through ripgrep's ignore machinery.

### Sort order

Newest-first by mtime (modification time, the filesystem timestamp of the file's last content change), computed in JS after ripgrep returns, via `oS`/`iS` with a **2000 ms** (`2e3`) batched stat budget. On timeout it silently keeps walk order and fires the `newest_first.stat_timeout_fallback` metric.

Tie-breaking is by path segments ascending. Missing mtime sorts last.

So the description's claim is accurate, and the `--sort=modified` flag is **not** used here. That flag appears only on a different, non-streaming path in Grep.

### Caps

| Constant | Value | Meaning |
|---|---|---|
| `CLIENT_LIMIT_LINES` | 2000 | files returned to the model |
| `HARD_MAX_OUTPUT_LINES` | 10000 | lines parsed from stdout |
| stdout buffer | 8 MB | hard limit |
| wall clock | 25 s | timeout |
| render budget | 5000 chars | output text budget |

The render budget usually binds first, at roughly 160 files.

**The "100 results" figure in public documentation is wrong for Cursor.** That number belongs to Claude Code, OpenCode, and Gemini CLI. Cursor is 2000/10000.

### Output format

```
Result of search in '<path>' (total N files):
- <relative path>
- <relative path>
...
```

with truncation notices when a cap was hit.

Paths are **workspace-relative**. A leading `./` is stripped only when the display path is exactly `.`. On Windows the `.\` prefix survives, because the strip test is `startsWith("./")`, which does not match a backslash.

### Failure modes

| Condition | Behavior |
|---|---|
| no matches | returns an empty result set, not an error |
| invalid glob | ripgrep rejection surfaces through the standard ripgrep error path |
| directory not found | target resolution failure, reported as a tool error |
| rg missing | path lookup failure with diagnostic context |
| timeout | 25 s wall clock, then a hard error |

### Edge cases

| Input | Behavior |
|---|---|
| leading `**/` | passed to `--iglob` unchanged; ripgrep handles it |
| `{a,b}` braces | supported, ripgrep native |
| `[abc]` classes | supported, ripgrep native |
| `!` negation | supported by ripgrep globs |
| absolute pattern | not meaningful for `--iglob` against a relative target |
| Windows separators | target is normalized; patterns should use forward slashes |
| symlinks | followed, because `--follow` is always on |
| `.git` contents | excluded via `!**/.git` |

### Algorithm pseudocode

Glob is Grep in listing mode: the same executor, `outputMode: "files_with_matches"`, and an empty pattern, which is what makes ripgrep emit `--files`. The pattern normalization happens before the executor is reached, and it is where absolute patterns silently break.

```
TOOL Glob

INPUTS
  args       parsed arguments: { glob_pattern, target_directory? }
  ctx        request context: workspace roots

OUTPUT
  GrepResult-shaped record, then rendered text; success carries { files[], totalFiles,
  clientTruncated, ripgrepTruncated, path }

1  PARSE       validate args against the version schema
                on failure THROW "Invalid arguments:\n<key>: <message>"
2  normalized  = normalizeGlob(rawArgs.glob_pattern)            // see the helper below
3  IF normalized === "**/"
       THROW 'glob_pattern is empty. Pass "*" to match every file.'
4  args        = { path: target_directory, glob: normalized,
                   outputMode: "files_with_matches", pattern: "" }   // pattern left at its default
5  result      = grepExecutor.execute(args)
6  IF the executor returned an error containing "received an empty pattern"
       retry once with pattern "."                              // empty-pattern fallback
7  SWITCH result.case
             "success" -> continues to render
             "error"   -> classify by substring, then THROW with the classified class
             otherwise -> THROW "glob_file_search didn't return the result"
                          // the string still says glob_file_search on every branch
8  RENDER
     IF files is empty  -> "Result of search in '${displayPath}': 0 files found\n"
     ELSE header        = "Result of search in '${displayPath}'"
                          + (ripgrepTruncated ? "" : " (total ${totalFiles} file" + (totalFiles === 1 ? "" : "s") + ")")
                          + ":\n"
          emit "- ${path}\n" per file until the 5000-character render budget is reached
          on the cut: ripgrepTruncated
                        ? "... at least ${max(0, totalFiles - emitted)} more files ... (Do a more specific search if needed)\n"
                        : "... ${max(0, totalFiles - emitted)} more files ... (Do a more specific search if needed)\n"
          if the loop finished without a cut AND a limit was hit, append the banner:
            "[Results truncated." + (totalFiles > 0 ? " " + (ripgrepTruncated ? "At least " : "")
                                    + "${totalFiles} file" + (totalFiles === 1 ? "" : "s") + " in total." : "")
            + " Narrow the search with a more specific target_directory or glob_pattern.]\n"
```

Helper: pattern normalization, applied to `glob_pattern` before anything else runs. Four transformations, in this order.

```
normalizeGlob(pattern)

1 pattern = replace all "\\" with "/"           // Windows-style patterns accepted
2 IF startsWith(pattern, "/")        strip exactly one leading "/"
3 IF endsWith(pattern, "/")          strip exactly one trailing "/"
4 IF NOT startsWith(pattern, "**/")  pattern = "**/" + pattern
5 RETURN pattern
   // "*.js" stays recursive; "**/*.js" is unchanged; an absolute pattern becomes a
   // "**/"-prefixed suffix match, which is why absolute patterns do not work here
   // the prefix is added exactly once, so "**/" alone is the error case from step 3
```

Helper: the executor pipeline that turns a normalized glob into a sorted, capped file list.

```
executorFilesMode(targetDirectory, glob, workspaceRoots)

1 roots = targetDirectory is absolute OR root-independent
            ? narrow to the single matching root, error if it maps to none
            : every workspace root
          error when the target spans roots:
            "Path '${target}' points to a different folder from each workspace root; use an absolute path"
2 per root, in order:
     IF the root does not exist       RETURN error "Path does not exist: ${absolute}"
     IF filtered by .cursorignore     RETURN error "Path '${target}' is filtered out by .cursorignore"
     IF neither a file nor a directory RETURN error "Path '${target}' is neither a file nor a directory"
     IF realpath failed               RETURN error "Path '${target}' could not be verified while resolving symlinks. This can happen when access is denied or the path contains a symlink loop."
3 argv (order matters):
     --cursor-ignore <file> ...      repeated pairs
     --files                         because the pattern is empty
     --case-sensitive
     --iglob <glob>
     --iglob !**/.git
     --no-config
     --color=never
     --hidden
     --follow
     --
     <relative target>
   // no sort flag at all; sorting happens in JS afterward
4 spawn with cwd = root, 25 s wall clock, 8 MB stdout cap, 10000 line parse cap
   missing rg binary is reported as "Path does not exist: ${cwd}" rather than as a missing binary
5 parse stdout lines; empty stdout gives an empty file list, not an error
6 stat every path in parallel under a 2000 ms budget
     on timeout keep ripgrep walk order and fire newest_first.stat_timeout_fallback
7 sort by mtime descending, ties by path segments ascending, missing mtime last
8 slice to 2000; clientTruncated when the raw total exceeded 2000;
  ripgrepTruncated when the raw line count reached 10000, or on timeout, or on the stdout cap
9 post-filter through permissions and cursorignore, decrementing totalFiles for each drop
10 flatten across roots, summing totalFiles; ordering holds per root, not globally
```

### Worked examples

Paths are workspace-relative in the output, because ripgrep runs with its cwd at the workspace root and emits relative paths. On Windows the walk output carries a `.\` prefix that the `./` strip test does not match, so `.\` survives; the vectors below use the POSIX form for clarity and one Windows row spells out the difference.

| Case | Input | Expected output |
|---|---|---|
| ordinary match (confirmed) | `glob_pattern: "*.ts"` in a repo with three `.ts` files | header, then one `- src/<name>.ts` line per file, newest first |
| case-insensitive pattern (confirmed) | `glob_pattern: "*.JS"` where files are `a.js` and `B.JS` | both listed, because the flag is `--iglob` |
| zero matches (confirmed) | `glob_pattern: "*.rs"` in a TypeScript-only repo | `Result of search in '.': 0 files found` and nothing else |
| empty pattern (confirmed) | `glob_pattern: ""` | `glob_pattern is empty. Pass "*" to match every file.` |
| whitespace-only pattern (derived) | `glob_pattern: " "` | not an error; normalizes to `**/ ` and matches nothing, rendering the zero case |
| newest-first sort (confirmed) | three files with mtimes 09:00, 11:00, 10:00 | 11:00 first, then 10:00, then 09:00 |
| tie-break by path (derived) | two files with identical mtime, `src/a.ts` and `src/b.ts` | `a.ts` before `b.ts`, compared segment by segment |
| binary and empty files included (confirmed) | `*.py` matching a real file, an empty file, and a NUL-containing file | all three listed, because `--files` never reads contents |
| `.git` contents excluded (confirmed) | `glob_pattern: "*"` at a repo root | no path under `.git`; exclusion is the explicit `--iglob !**/.git` |
| gitignored files absent (confirmed) | `glob_pattern: "**/*.log"` where `*.log` is gitignored | zero matches, because `.gitignore` wins over a positive glob |
| dotfiles included (confirmed) | `glob_pattern: ".env"` | matches `.env`, because `--hidden` is unconditional |
| absolute pattern (confirmed) | `glob_pattern: "C:/work/proj/*.ts"` | nothing; it normalizes to `**/C:/work/proj/*.ts`, a suffix match that no path satisfies |
| target does not exist (confirmed) | `target_directory: "C:\work\proj\nope"` | `Path does not exist: C:\work\proj\nope` |
| invalid glob syntax (confirmed) | `glob_pattern: "[a-"` | the ripgrep line `rg: error parsing glob '**/[a-': unclosed character class; missing ']'` |
| more results than the render budget (confirmed) | 400 matching short paths | about 160 `- ` lines, then `... 240 more files ... (Do a more specific search if needed)` |

#### Ordinary match, sorted newest first (confirmed)

Three files in the workspace, with these mtimes: `src/index.ts` at 09:14, `src/util.ts` at 11:02, `src/greet.ts` at 10:40.

Input:

```json
{"glob_pattern": "*.ts", "target_directory": "C:\\work\\proj"}
```

The model receives:

```text
Result of search in 'C:\work\proj' (total 3 files):
- src/util.ts
- src/greet.ts
- src/index.ts
```

The order is mtime descending, not alphabetical and not walk order. Note the header shape: the path in single quotes, then a space, then `(total 3 files)` in parentheses, then a colon. The count line is dropped entirely when the result set was cut by the parse cap, because the count would then be a lie. The `- ` prefix is two characters, and every path is workspace-relative. An implementation that shells out to `rg --files` and prints its output directly gets alphabetical or walk order and fails this vector.

#### Windows output where the `.\` prefix survives (confirmed)

Input on Windows:

```json
{"glob_pattern": "*.md"}
```

The model receives:

```text
Result of search in '.' (total 2 files):
- .\README.md
- .\docs\guide.md
```

The strip test is `startsWith("./")`, which does not match a backslash, so the `.\` prefix stays. On a POSIX host the same call renders `README.md` and `docs/guide.md` with no prefix, because only the leading `./` is removed and only when the display path is exactly `.`. This is the one formatting difference worth pinning separately per platform.

#### A pattern that normalizes to a suffix match (confirmed)

Input, attempting to search an absolute location through the pattern rather than through `target_directory`:

```json
{"glob_pattern": "/home/dev/proj/src/*.ts"}
```

The model receives:

```text
Result of search in '.': 0 files found
```

Normalization strips the leading slash, giving `home/dev/proj/src/*.ts`, then prefixes `**/`, giving `**/home/dev/proj/src/*.ts`. That is a suffix match against the path, and no file is under a directory literally named `home/dev/proj/src`, so nothing matches. The description's promise that patterns without `**/` get one prepended is exactly what breaks this case. The fix, which the tool cannot make on the model's behalf, is to put the absolute location in `target_directory` and keep the pattern relative.

What each example proves:

- The three-file example pins the header template, the mtime-descending sort, and the `- ` line prefix.
- The Windows example pins that the `./` strip is effectively POSIX-only, which changes the bytes the model sees on the platform most likely to be reimplemented against.
- The absolute-pattern example pins the normalization chain and the fact that `**/` prefixing turns an absolute intent into a match that cannot succeed.

*2026-10-27 - deepseek-v41-flash-1017*

## Grep

### At a glance

This searches file contents with a regular expression and returns matching lines, file names, or counts depending on the output mode.

A model reaches for it when it knows a symbol, string, or pattern and needs to find where it appears. It reaches for `Glob` instead when it only knows the filename shape, and it should prefer this over shelling out to `rg` because this one applies workspace ignore rules and output caps.

The single most commonly-misimplemented detail: `head_limit` is applied client-side, after parsing, as a global counter across every file. It is not passed to ripgrep as `--max-count`, and `--max-count` is never emitted on this path. Getting that wrong gives per-file limits instead of a global one, which is visible immediately in multi-file results.

Internal id `GREP`, offset 4304168.

### Names per branch

Three distinct public names:

| Version | Name |
|---|---|
| `dsv3-1018` | `grep` |
| `cursor-0226`, `dsv3-1205`, `latest`, `haiku` | `Grep` |
| `gpt5-codex`, `codex-cloud` | `rg` |

`grep_search` never appears as a tool name. It exists only as proto field text.

### Parameters

Two zod schemas (`wIe` default, `vIe` for `dsv3-1018`) with identical field sets. Only four description strings differ. Both are declared as follows:

```js
pattern:      string, required
    .describe("The regular expression pattern to search for in file contents (rg --regexp)")

path:         string, optional
    .describe("File or directory to search in (rg pattern -- PATH). Defaults to Cursor workspace root.")

glob:         string, optional
    .describe('Glob pattern to filter files (e.g. "*.js", "*.{ts,tsx}") - maps to rg --glob')

output_mode:  enum ["content","files_with_matches","count"], default "content", optional
    .describe('Output mode: "content" shows matching lines (supports -A/-B/-C context,
     -n line numbers, head_limit), "files_with_matches" shows file paths (supports
     head_limit), "count" shows match counts (supports head_limit). Defaults to "content".')

"-B":         int, optional
    .describe('Number of lines to show before each match (rg -B). Requires output_mode: "content", ignored otherwise.')

"-A":         int, optional
    .describe('Number of lines to show after each match (rg -A). Requires output_mode: "content", ignored otherwise.')

"-C":         int, optional
    .describe('Number of lines to show before and after each match (rg -C). Requires output_mode: "content", ignored otherwise.')

"-i":         boolean, default false
    .describe("Case insensitive search (rg -i) Defaults to false")

type:         string, optional
    .describe("File type to search (rg --type). Common types: js, py, rust, go, java, etc.
     More efficient than glob for standard file types.")

head_limit:   int, min 0, optional
    .describe('Limit output size. For "content" mode: limits total matches shown.
     For "files_with_matches" and "count" modes: limits number of files.')

offset:       int, min 0, optional
    .describe('Skip first N entries. For "content" mode: skips first N matches.
     For "files_with_matches" and "count" modes: skips first N files.
     Use with head_limit for pagination.')

multiline:    boolean, default false
    .describe("Enable multiline mode where . matches newlines and patterns can span lines
     (rg -U --multiline-dotall). Default: false.")
```

**The hyphenated keys are literal.** `"-A"`, `"-B"`, `"-C"`, `"-i"` are the actual JSON property names, not normalized. They are passed through as ripgrep flags.

**Parameters that do not exist:** no `target_directory`, no `-n`, no `case_insensitive`, no `--no-ignore`. The path field is `path` and the case flag is literally `-i`.

Numeric values accept strings and coerce. `head_limit: "50"` works, `"-A": "2"` works, `"-i": "yes"` works.

### Description text

The canonical (`latest`, `dsv3-1205`, `haiku`) form:

```
A powerful search tool built on ripgrep
Usage:
- Prefer using Grep for search tasks when you know the exact symbols or strings to search for. Whenever possible, use this tool instead of invoking grep or rg as a terminal command. The Grep tool has been optimized for speed and file restrictions inside Cursor.
- Supports full regex syntax (e.g., "log.*Error", "function\s+\w+")
- Filter files with glob parameter (e.g., ".js", "**/.tsx") or type parameter (e.g., "js", "py", "rust")
- Output modes: "content" shows matching lines (default), "files_with_matches" shows only file paths, "count" shows match counts
- Pattern syntax: Uses ripgrep (not grep) - literal braces need escaping (use interface\{\} to find interface{} in Go code)
- Multiline matching: By default patterns match within single lines only. For cross-line patterns like struct \{[\s\S]*?field, use multiline: true
- Avoid overly broad glob patterns (e.g., '--glob *') as they bypass .gitignore rules and may be slow
- Only use 'type' (or 'glob' for file types) when certain of the file type needed. Note: import paths may not match source file types (.js vs .ts)
- Results are capped to several thousand output lines for responsiveness; when truncation occurs, the results report "at least" counts, but are otherwise accurate.
- Content output follows ripgrep format: '-' for context lines, ':' for match lines, and all lines grouped by file.
```

`cursor-0226` gets only the last two bullets' content as a single sentence. `gpt5-codex` and `codex-cloud` get a seven-bullet terse variant. `dsv3-1018` gets a longer form mentioning `(unsaved)` and `(out of workspace)` editor states.

### Argument construction

Order matters for reproducibility:

```
--cursor-ignore <file> ...   (repeated pairs)
<mode flags>
[--before-context N]         (from -B)
[--after-context N]          (from -A)
[--case-sensitive]           (omitted when -i is set)
[--type T]                   (from type)
[--iglob G]                  (from glob)
--no-config
--color=never
--encoding <enc>             (when set)
--regexp <pattern>           (unless --files mode)
--
<target>
```

Specific mappings:

| Parameter | Becomes |
|---|---|
| `glob` | `--iglob` (case-insensitive), **not** `--glob` |
| `type` | `--type` |
| `-i` | absence of `--case-sensitive`; ripgrep defaults to case-insensitive when the pattern has no uppercase |
| `-C` | expanded into `--before-context N` and `--after-context N` |
| `-A` / `-B` | `--after-context` / `--before-context` |
| `output_mode: files_with_matches` | `-l` |
| `output_mode: count` | `-c` |
| `output_mode: content` | default line output |
| `multiline` | `-U --multiline-dotall` |

**`-C` is dropped entirely if `-A` or `-B` is also present.** That is a real quirk. Explicit one-sided context wins over symmetric context.

**`--max-count` is never emitted.** `head_limit` and `offset` are applied client-side, after parsing, as global counters across all files. This is the single most important behavioral difference to get right, because `--max-count` would be per-file and produce different results.

### Caps

| Mode | Cap |
|---|---|
| `content` | 2000 entries, or 4000 when context is present |
| `files_with_matches` | 2000 files |
| `count` | 2000 files |
| stdout | 8 MiB, process killed on exceed |
| parse budget | 10000 lines |

There is **no default `head_limit`**. Omitting it means no client-side limit, and the practical cap is the 2000-entry limit above. `head_limit: 0` is coerced to unlimited:

```js
const p = fIe(r.offset, Math.floor) ?? 0,
      m = fIe(r.head_limit, Math.ceil) || void 0;
```

The `|| void 0` is what makes zero mean omitted. `head_limit` is ceiled, `offset` is floored.

This differs from Claude Code, which defaults to 250.

### Sort order

Newest-mtime-first, sorted client-side via `mtimeNs` with a 2000 ms stat budget, same as Glob. The `--sortr modified` flag appears only on the non-streaming relative-target path; it is a fallback, not the default.

### Output format

Everything is wrapped in an envelope:

```
<workspace_result workspace_path="/abs/path">
<body>
</workspace_result>
```

Multi-workspace output emits one block per workspace, with a joint character budget computed by binary search so the total stays near 15000 characters.

**Content mode.** File names appear bare on their own line, with no separator and no colon. Match lines are two spaces, line number, colon, content. Context lines use a hyphen instead of a colon:

```
<workspace_result workspace_path="/workspace">
src/a.ts
  3:const alpha = 1
  4-const beta = 2
  --
  9:const gamma = 3

src/b.ts
  1:import x from "y"

</workspace_result>
```

Structural details:

- Match line: `"  " + lineNumber + ":" + content`
- Context line: `"  " + lineNumber + "-" + content`
- Gap separator between non-adjacent line numbers within a file, **only when the result contains any context lines**: the literal `  --` on its own line
- After each file's lines: one blank line
- The whole string is `.trim()`ed at the end

This resembles ripgrep `--heading` output but is not identical: ripgrep's gap separator is `--` with no leading spaces, and ripgrep does not add a trailing blank line per file.

The wire format uses NUL (the zero byte, `0x00`) to separate path from line number, which is what makes parsing safe against paths containing colons:

```
<path>\0<lineNumber>:<content>\n
<path>\0<lineNumber>-<content>\n
```

with **nothing between the path and the NUL**.

**files_with_matches mode** returns one path per line. **count mode** returns `path:count` per line.

### Error handling

Invalid regexes are rewritten rather than passed through:

```
rg: regex parse error: invalid regex "<pattern>": <reason>.
```

with a hint suffix appended only for three specific reasons: unclosed group, unopened group, and repetition-operator errors.

### Binary and permission handling

`throwOnRipgrepFailure` suppresses errors that are purely permission or loop related when stdout is empty, so a scan over a tree with some unreadable directories does not fail the whole search. It parses stderr lines against:

```
/^rg: (.+?): .*\(os error (?:13|5|2|1)\)$/
/^rg: File system loop found: (.+) points to an ancestor /
```

and suppresses only when every error line names a different path than the search target.

Post-ripgrep filtering applies `.cursorignore` and repo blocklists via `filterBlockedFiles`, which can remove results ripgrep returned.

### Edge cases

| Input | Behavior |
|---|---|
| invalid regex | rewritten error, see above |
| empty pattern | matches every line |
| `.gitignore` | respected; `--no-ignore` is never emitted |
| symlinks | per ripgrep default; `--follow` not set on this path |
| `-C` with `-A` or `-B` | `-C` dropped |
| `head_limit: 0` | treated as unlimited |
| multiline | `-U --multiline-dotall`, so `.` matches newlines |
| unsaved editor buffers | searched, reported as `(unsaved)` or `(out of workspace)` in the `dsv3-1018` description only |

### Algorithm pseudocode

Grep validates arguments, builds an argv, and then paginates client-side. The two ordering facts that matter most are that `head_limit` is applied after parsing and globally, and that `-C` is dropped when a one-sided context flag is present.

```
TOOL Grep

INPUTS
  args       parsed arguments: { pattern, path?, glob?, output_mode?, "-A"?, "-B"?, "-C"?,
                                 "-i"?, type?, head_limit?, offset?, multiline? }
  ctx        request context: workspace roots

OUTPUT
  GrepResult-shaped record wrapped in the workspace_result envelope, or a thrown classified error

1  PARSE       validate args against the version schema
                on failure THROW "Invalid arguments:\n<key>: <message>"
2  IF pattern === ""
       THROW 'pattern is empty. Pass "." to match every line.'
       // only the exact empty string is caught; " " passes through to ripgrep
3  FOR flag in ["-A", "-B", "-C"]
       IF flag is present AND its value < 0
           THROW "${flag} is ${value}. Pass 0 or more context lines."
4  IF the broad-glob feature flag is on AND trim(glob) is in { "**/*", "**/**" }
       THROW 'Glob pattern "${trim(glob)}" matches every file and is not allowed. Use a more specific glob or no glob.'
5  offset     = floor(args.offset) ?? 0
   headLimit  = ceil(args.head_limit) || undefined     // the || is what makes 0 mean unlimited
6  cacheKey   = every field except toolCallId, sandboxPolicy, headLimit, offset
   IF a cached result exists for cacheKey  RETURN it      // 5-entry LRU
7  argv = buildArgs(args)                              // see the helper below
8  result = execute(argv) with cwd = resolved root, 25 s timeout
9  IF any root timed out AND none succeeded
       THROW "Timed out after 25s"
10 paginate(result, offset, headLimit)                 // see the helper below
11 RENDER
     error case        -> "Error running tool: " + headTailTruncate(error, 15000)
     non-success case  -> "No results found."
     no workspaces     -> "No results found."
     success           -> envelope per workspace, then mode-specific body
```

Helper: argv construction. The order is reproducible, and `--max-count` is deliberately absent.

```
buildArgs(args)

1 --cursor-ignore <file> ...           repeated pairs, from discovered ignore files
2 mode flags:
     output_mode "files_with_matches" -> -l
     output_mode "count"              -> -c
     output_mode "content"            -> nothing (default line output), plus --line-buffered
3 IF -B present  -> --before-context N
   IF -A present  -> --after-context N
   // -C is expanded into both before and after ONLY when neither -A nor -B is present;
   // if either one-sided flag appears, -C is dropped entirely
4 IF -i is NOT set                      -> --case-sensitive
   // when -i is set the flag is simply omitted, because ripgrep is case-insensitive by
   // default when the pattern has no uppercase
5 IF type      -> --type T
6 IF glob      -> --iglob G             // case-insensitive, NOT --glob
7 --no-config
8 --color=never
9 IF an encoding was detected for a single UTF-16/UCS-2 file  -> --encoding <enc>
10 IF multiline -> -U --multiline-dotall
11 IF the pattern is empty (Glob's listing mode)  -> --files
   ELSE                                          -> --regexp <pattern>
12 --
13 <target>
   // --max-count is never emitted; head_limit and offset are client-side
   // --no-ignore is never emitted; --hidden and --follow are always on
```

Helper: client-side pagination. Both counters are global across files, and context lines travel with the match they precede.

```
paginate(result, offset, headLimit)

1 IF result is not success       RETURN result unchanged
2 IF offset === 0 AND headLimit is undefined  RETURN result unchanged
3 IF mode is content
     skipped = 0; emitted = 0; cut = false; out = []
     FOR each file in result.matches
         IF cut BREAK
         kept = []; pendingContext = []
         FOR each entry in file.matches
             IF entry.isContextLine
                 IF kept is non-empty  kept.push(entry)  ELSE pendingContext.push(entry)
             ELSE IF skipped < offset
                 skipped += 1; pendingContext = []          // drop the leading context too
             ELSE
                 IF emitted >= headLimit  cut = true; BREAK
                 kept.pushAll(pendingContext); pendingContext = []
                 kept.push(entry); emitted += 1
         IF kept is non-empty  out.push({ file: file.file, matches: kept })
     RETURN { matches: out, clientTruncated: result.clientTruncated || cut, ... }
4 ELSE
     start = min(offset, count(items))
     end   = headLimit !== undefined ? min(start + headLimit, count(items)) : count(items)
     RETURN { items: items[start ..< end], truncated: headLimit !== undefined AND start + headLimit < count(items) }
```

Helper: the no-result and truncation strings, which differ by mode and by whether a page was applied.

```
noResultText(result, mode, offset)

1 IF offset === 0 OR nothing to show
      IF ripgrepTruncated  RETURN "${modeText} before the search stopped. Narrow the search to see the rest."
      RETURN modeText                        // "No matches found" for content and count,
                                             // "No files with matches found" for files mode
2 IF clientTruncated OR ripgrepTruncated OR offset < total
      RETURN "No more results at offset=${offset}. Narrow the search to see the rest."
3 RETURN "No more results at offset=${offset}."
```

Helper: the content-mode body, which is the one with the synthetic gap separator.

```
renderContent(entries)

1 anyContext = any entry across all files has isContextLine
2 FOR each file
      emit file.file                                          // bare, no separator, no colon
      lastLine = undefined
      FOR each entry
          IF anyContext AND lastLine !== undefined AND entry.lineNumber - lastLine > 1
              emit "  --"                                     // two spaces, two hyphens
          lastLine = entry.lineNumber
          emit "  " + entry.lineNumber + (entry.isContextLine ? "-" : ":") + entry.content
      emit ""                                                 // one blank line after each file
3 join, wrap in <workspace_result workspace_path="..."> ... </workspace_result>, then trim
   // the wire format separates path from line number with a NUL and nothing else
```

### Worked examples

Every success vector is wrapped in the `<workspace_result>` envelope, so the table shows the body and the fenced examples show the envelope. File lines inside `content` mode are two spaces, the line number, then `:` for a match or `-` for context. `head_limit` counts are global across files, not per file.

| Case | Input | Expected output |
|---|---|---|
| ordinary content match (confirmed) | `pattern: "TODO"`, `output_mode` omitted | envelope, then for each file a bare path line, then `  12:TODO: ...` lines, then a blank line |
| zero matches (confirmed) | `pattern: "zzz_no_such_symbol"` | `No matches found` |
| zero matches in files mode (confirmed) | same pattern with `output_mode: "files_with_matches"` | `No files with matches found` |
| empty pattern (confirmed) | `pattern: ""` | `pattern is empty. Pass "." to match every line.` |
| `head_limit` global across files (confirmed) | `head_limit: 2`, three matches in `a.ts` and two in `b.ts` | two match lines total, both from `a.ts`, then the trailer; `b.ts` never appears |
| `head_limit: 0` (confirmed) | `head_limit: 0` with 40 matches | all 40 matches, because zero is coerced to `undefined` |
| `offset` past the end (confirmed) | `offset: 500`, 40 matches | `No more results at offset=500.` |
| `-C` dropped when `-A` present (confirmed) | `-C: 3`, `-A: 1` | one line of after-context per match and no before-context, because `-C` is discarded entirely |
| `-A` context rendering (confirmed) | `-A: 2`, one match on line 3 of a 9-line file | `  3:` match line then `  4-` and `  5-` context lines, with no gap marker between adjacent numbers |
| gap separator (confirmed) | `-A: 1`, matches on lines 3 and 9 | `  --` on its own line between the two groups, only because the result contains context lines |
| `files_with_matches` (confirmed) | `pattern: "ripgrep"`, `output_mode: "files_with_matches"` | bare paths, one per line, no line numbers and no counts |
| `count` mode (confirmed) | same pattern with `output_mode: "count"` | `src/a.ts:3` style lines, no spaces around the colon |
| invalid regex, unclosed group (confirmed) | `pattern: "a(b"` | `Error running tool: rg: regex parse error: invalid regex "a(b": unclosed group. Escape special characters such as ( [ { with a backslash to match them literally.` |
| broad glob guard (confirmed) | `glob: "**/*"` with the guard flag on | `Glob pattern "**/*" matches every file and is not allowed. Use a more specific glob or no glob.` |
| binary match in content mode (confirmed) | pattern matching inside a NUL-containing file | the file is absent from content results; it still appears under `count` and `files_with_matches` |

#### Content mode with context, two files (confirmed)

Two files. `src/a.ts` is nine lines long and matches on lines 3 and 9. `src/b.ts` is one line long and matches on line 1. No `head_limit`, so nothing is cut and no trailer appears. Input:

```json
{"pattern": "const", "-A": 1}
```

The model receives exactly this, ending at `>` with no trailing newline after the final trim:

```text
<workspace_result workspace_path="/home/dev/proj">
src/a.ts
  3:const alpha = 1
  4-  const beta = 2
  --
  9:const gamma = 3
  10-export default gamma

src/b.ts
  1:const first = 0

</workspace_result>
```

Line 4 exists only because `-A: 1` asked for one line after each match, and it is marked with `-` rather than `:`. The gap marker appears between line 4 and line 9 because the result contains context lines and the line numbers are non-adjacent; it is `  --`, two spaces then two hyphens, which differs from ripgrep's own `--` and is not the same string. The file name sits bare on its own line with no colon, no separator, and no path prefix. After each file's group there is exactly one blank line, which is what puts a blank line before `</workspace_result>`.

#### Global `head_limit` across two files (derived)

The body is documented fact; the trailer line is derived from the documented template in `### Output format` plus the component strings, so treat the `[...]` line as the one to confirm against a live probe before freezing a byte-level test.

Five matches total: `src/a.ts` has two, on lines 12 and 40, and `src/b.ts` has three. Input:

```json
{"pattern": "return null", "head_limit": 2}
```

The model receives:

```text
<workspace_result workspace_path="/home/dev/proj">
src/a.ts
  12:  return null;
  40:  return null.

[Showing 2 of 5 matching lines. Use offset=2 to see more, or narrow the search.]
```

`src/b.ts` contributes nothing, because the counter reached 2 while the scan was still inside `a.ts`. A per-file limit would have shown two matches from `a.ts` and two more from `b.ts`, and the file order itself follows the mtime sort, so a per-file implementation diverges in both content and count. This is the single most commonly misimplemented behavior in the tool, and this vector is the one that catches it.

#### Invalid regex, rewritten (confirmed)

Input:

```json
{"pattern": "a(b"}
```

ripgrep's own stderr for this pattern is a four-line block with a caret line:

```text
rg: regex parse error:
    (?:a(b)
    ^
error: unclosed group
```

The model receives this instead, one line, with the caret block gone:

```text
Error running tool: rg: regex parse error: invalid regex "a(b": unclosed group. Escape special characters such as ( [ { with a backslash to match them literally.
```

The rewrite extracts the `error:` line, rebuilds a single-line message with the pattern interpolated raw, and appends the escape hint only because the reason matched `unclosed`. A reason outside the three-hint set, such as `backreference is not supported`, produces the short form with no hint. An implementation that passes ripgrep's stderr through unchanged gets a four-line error the model has never seen from Cursor.

What each example proves:

- The context example pins the envelope, the bare file-name line, the `:` and `-` delimiters, the `  --` gap marker, and the blank line between files.
- The global-limit example pins that `head_limit` is one counter across every file, which no per-file flag can reproduce.
- The invalid-regex example pins that errors are rewritten into a single line with a conditional hint, not forwarded verbatim.

*2026-10-27 - deepseek-v41-flash-1017*

## ListDir

### At a glance

This lists a directory tree recursively, returning paths with indentation so the model can see how a project is organized.

A model reaches for it when it wants an overview of a directory's shape and has no target in mind yet. It reaches for `Glob` once it has a pattern, because `Glob` is cheaper and narrower, and for `Grep` once it has a symbol.

Two traps. First, the description claims dotfiles are excluded, but the walk hardcodes `--hidden` and nothing filters dot entries, so they are listed. Second, and more consequential if you are trying to observe the tool: ListDir is gated behind an `enableDeprecatedLsTool` flag that defaults to false, so in most sessions it is **not offered to the model at all**. Its practical role today is as a fallback other descriptions reference, not a primary surface.

One further caution before you build against this. Cursor is replacing this tool. The proto layer carries three generations side by side, `ListDirParams`, `ListDirV2Params`, and a `ListDirV2Stream`, and there is a server config field named `listDirV2PredefinedIgnoreGlobs`. `ListDirV2Params` adds a `should_enrich_terminal_metadata` field the legacy tool lacks. So the tool specified here is the legacy surface being retired, and the rendering algorithm below is the part most likely to survive into whatever replaces it. Treat this section as reference material for a real implementation, not as Cursor's forward direction.

Not one of the six, but included because it is a real tool and because its tree renderer is the clearest example in the bundle of how a structured result becomes model-visible text.

Internal id `LS`, offset 4310351 in the minified bundle. Unminified source lives in the daemon at `agent/dist/tools/core/ls/ls.js` and `agent/dist/tools/core/ls/formatters.js`.

One shared zod schema for every model family, which is unusual for this codebase:

```js
target_directory: string, required
    .describe("Path to directory to list contents of.")

ignore_globs: string[], optional
    .describe(`Optional array of glob patterns to ignore.
All patterns match anywhere in the target directory. Patterns not starting with "**/" are automatically prepended with "**/".

Examples:
	- "*.js" (becomes "**/*.js") - ignore all .js files
	- "**/node_modules/**" - ...`)
```

Note the tab characters before the example bullets. The examples live inside the `.describe()` string, so they reach the model.

Parameters that do **not** exist: no `path`, no `depth`, no `recursive`, no `include_hidden`, no `max_results`, no `explanation`. `path` appears only as a proto field on the args message, never as a model input.

The registry id is always the literal `"LS"`. The model-facing name switches only for one family: `list_dir` for `dsv3-1018`, `LS` for everything else, with an absent version defaulting to `latest`.

Registration is conditional:

```js
if (props.enableDeprecatedLsTool ?? false) {
  tools.push(createLsTool(resourceAccessor, promptVersion));
}
```

The flag comes from outside the agent package and defaults to false, so the tool ships in the bundle but is usually absent from the tool set. If you reimplement, decide deliberately whether to offer it.

The description has two variants differing in exactly one line about path form: one says the path `can be relative to the workspace root or absolute`, the other says it `must be an absolute path`. Six of the seven versions use the absolute-only wording. There is no `isComposer15` conditional and no runtime fragments, and zero typographic apostrophes anywhere in the description, schema, or renderer. All apostrophes are plain ASCII, unlike several other tools here.

It is a recursive `ripwalk` walk. The verified argument vector is `--files --hidden --no-require-git --no-config --color=never --threads N --case-sensitive`, plus `--cursor-ignore` entries and negated `--iglob` patterns when applicable. Note there is **no** `--follow` and no `--no-ignore-vcs`, so symlinks are not followed, and this differs from the `rg --files` form Glob uses.

Empty directories are invisible by construction. The tree is a sparse trie of file paths, so a directory with no files under it never appears in the structure at all. The "no children found" branch only fires for a root that genuinely produced no files.

Because the tree is a sparse trie of paths, a directory's `numFiles` is its subtree file count, which is what feeds the extension summary line.

Output shape: two spaces of indentation per depth level, `- ` before each name, and a path separator appended to **directories only**, never to files. That distinction is easy to get wrong, because the separator decision and the entry rendering live in different branches. Separator is `\\` on Windows and `/` elsewhere, chosen once from whether the root path contains a backslash. Sort is `localeCompare` on the name, so it is locale-aware rather than byte-ordered, with files and directories in one list rather than directories first. A root holding `README.md` and a `src` directory with `main.rs`, `util.rs`, and `nested/` renders as `/home/user/project/` then `  - README.md` with no separator, then `  - src/` with one, then the three children indented four spaces, where the two files carry no separator and `nested` does. This was verified directly against the renderer: the file branch emits `` `${childIndent}- ${child.name}\n` `` and the directory branch emits `` `${childIndent}- ${child.name}${pathSep}\n` ``.

Empty directories get a dedicated branch rather than rendering blank: the root path with a hard-coded forward slash, then a line reading `... no children found ...`. That forward slash is inconsistent with the platform separator used elsewhere.

A directory can also emit an extension summary line, and only under a collapsed directory. The form is `[N file in subtree: 3 *.rs, 1 *.toml]`, with correct singular and plural on `file`, up to three extensions shown, ties broken by `localeCompare`, `, ...` appended when more extensions exist, and `no-ext` as the key for files with no extension. When any such line is rendered, a note is appended explaining that the counts exclude gitignored files.

Three render passes are attempted, each less detailed, and the first that fits the renderer budget wins: full, then depth-limit zero with counts, then depth-limit zero without counts.

Notes appear at most twice, formatted as `\nNote: X` for one and `\nNote:\n - A\n - B` for two, with the join separator exactly `"\n - "`. A third note about a timeout is appended by the render wrapper.

There are **two separate budgets operating at different stages**, and understanding which does what clears up an apparent contradiction.

`DEFAULT_CHARACTER_BUDGET = 2500` lives in the executor (`local-exec/dist/ls.js`). It caps cumulative filename length *during the walk*, as the tree is built from the ripgrep stream. So it bounds how much tree you get, before any rendering happens.

`LS_CHARACTER_BUDGET = 1e4` lives in the formatters module and caps the *rendered string*.

Because the walker caps at 2500 first, the rendered result is usually already short, so the 10,000-character slicing path in the renderer rarely fires. In practice the truncation you observe comes from the walker, not the renderer. Treat 2500 as the budget you will actually feel.

The truncation code does exist and is exact when it runs: slice to 10,000, backtrack to the last newline, then append the literal `"    ...\n"`, which is four spaces followed by three dots and a newline. Verified character by character. That line is appended whenever `!childrenWereProcessed`, even if nothing was sliced, so on a large real repository the dots line appears through that path rather than through the size check.

A timeout produces a partial result: you get the tree built so far, then a blank line, then the note `Note: The operation timed out after 5 seconds. The result is incomplete.` The default timeout is 5 seconds, overridable through a `timeout_ms` field on the args proto. Error classification is by substring on the message, so a message containing `timed out` becomes a TIMEOUT-class tool error, `does not exist` maps to not-found, and everything else falls through to a generic error.

The result has exactly four cases: `success`, `error`, `rejected`, `timeout`. There is no `permissionDenied` case and no busy state, so a permission failure arrives as raw Node errno text through the generic `error` case, rendered as `Error listing directory: ${error}`. A rejection renders as `Rejected: ${reason}`.

Two further behavioral facts worth knowing, both verified in the executor rather than inferred.

First, **LS takes no per-file lock.** `createLsTool` is never handed a lock manager, and `waitForLock` does not appear in its execute path:

```js
createLsTool = (resourceAccessor, promptVersion) => {
  const lsExecutor = resourceAccessor.get(lsExecutorResource);
  ...
  const execute = async (parentCtx, interactionHandler, rawArgs, meta) => {
    const spanCtxt = createSpan(parentCtx.withName("lsExecute"));
    const args = new LsArgs({
      path: maybeRedirectWorktriesPath(rawArgs.target_directory),
      ignore: rawArgs.ignore_globs,
      toolCallId: meta.toolCallId,
    });
```

Two arguments only: resource accessor and prompt version. Compare Read, which receives the manager and calls `waitForLock` as its second step. So a directory listing can observe a tree mid-write. That is a real gap in the locking model, and if you reimplement it, adding the lock is a defensible improvement.

Second, **the `ignore_globs` auto-prepend described in the parameter text is not implemented.** The description promises that patterns not starting with `**/` are automatically prepended with `**/`, but nothing in the LS path does that. It appears to work only because ripgrep treats a slashless glob as a basename match anywhere in the tree, which coincides with the promised behavior for the common case. A pattern like `deep/**` silently matches nothing, because it is never rewritten. The `normalizeGlob` helper that does perform prepending exists in the bundle, but it belongs to a different tool's code path.

One description-versus-behavior mismatch stands and it is the headline oddity of this tool: `--hidden` is hardcoded in the base walk args and nothing downstream filters dot entries, so the description's claim that dotfiles are excluded is false, and they are in fact listed. Separately, `--no-ignore` is never emitted, so `.gitignore` is respected, and `.cursorignore` is applied through the cursor-ignore arguments.

### Algorithm pseudocode

Four stages: resolve and stat the root, walk it with ripgrep into a sparse trie, render the trie with three progressively less detailed passes, then wrap the text with notes. There is no lock anywhere on this path, and no `permissionDenied` case.

```
TOOL ListDir

INPUTS
  args       parsed arguments: { target_directory, ignore_globs? }
  ctx        request context: workspace root, ignore service, timeout

OUTPUT
  LsResult record, oneof { success | error | rejected | timeout }, carrying a directory tree root

1  PARSE       validate args against the version schema
                on failure THROW "Invalid arguments:\n<key>: <message>"
2  path      = worktriesRepair(args.target_directory)           // same repair as the other tools
3  resolved  = resolve(path)          // file:// strip, ~ expansion, workspace-relative resolve
4  st        = stat(resolved)
                ENOENT AND path matches the terminal-log pattern
                    RETURN error "No active terminals."
                ENOENT otherwise
                    RETURN error "Path does not exist: ${resolved}"
                any other stat failure
                    RETURN error with the raw OS message
5  IF NOT st.isDirectory()   RETURN error "Path is not a directory: ${resolved}"
6  timeout   = args.timeoutMs ?? 5000
   started   = now()
   { tree, didTimeout } = buildTree(resolved, args.ignore_globs, timeout)
7  IF error thrown anywhere in step 6
       RETURN error with error is Error ? error.message : "Unknown error occurred"
8  RETURN didTimeout ? timeout { directoryTreeRoot: tree } : success { directoryTreeRoot: tree }
```

Helper: the walk and trie build. The budget is counted in basename characters, seeded with the root path length, and it bounds the tree before any rendering happens.

```
buildTree(root, ignoreGlobs, timeout)

1 trie = { absPath: resolve(root), childrenDirs: [], childrenFiles: [],
           childrenWereProcessed: true, fullSubtreeExtensionCounts: {} }
   budget = { used: length(trie.absPath), exceeded: false, limit: 2500 }
2 excludes = ignoreGlobs ++ repoBlockExcludeGlobs(root)        // passed through verbatim
                                                               // NOT prefixed with "**/"
3 argv = --files --hidden --no-require-git --no-config --color=never
         --threads N --case-sensitive
         [--cursor-ignore <file> ...]
         [--iglob !<exclude> ...]
   // no --follow, so symlinks are not descended; no --no-ignore-vcs, so .gitignore applies
4 FOR each emitted path line, until the timeout fires
      candidate = resolve(root, line)
      real      = realpath(candidate)
      IF real is null OR cursorIgnored(real) OR repoBlocked(real)  CONTINUE
      addPathToTree(candidate, budget)
      // addPathToTree synthesizes any missing ancestor directory nodes
5 sortTreeChildren recursively by localeCompare on the basename, dirs and files separately
6 RETURN { tree, didTimeout: timeout fired }
```

Helper: `addPathToTree`, where the 2500-character budget turns a directory into a collapsed summary line.

```
addPathToTree(absPath, budget)

1 IF the target is already known  RETURN
2 FOR each ancestor component that is not yet in the trie
      IF budget.exceeded
          mark the ancestor childrenWereProcessed = false
          count this file's extension into fullSubtreeExtensionCounts and numFiles up the chain
      ELSE create the directory node
3 IF NOT budget.exceeded
      budget.used += length(basename(absPath))
      IF budget.used > budget.limit  budget.exceeded = true
      ELSE attach the file node to its parent
```

Helper: the tree renderer. Three passes, and the pass is chosen by measuring the rendered length against the 10000-character render budget.

```
formatTree(root)

1 IF root.childrenWereProcessed AND no child dirs AND no child files
      RETURN root.absPath + (root.absPath endsWith "/" ? "" : "/") + "\n... no children found ..."
      // note the hard-coded forward slash, even on Windows
2 attempt passes in order, each measured against 10000 characters:
      pass A: full depth, extension counts enabled
      pass B: depth limit 0, extension counts enabled
      pass C: depth limit 0, extension counts disabled
   keep the first pass whose rendered length fits
3 IF the chosen text still exceeds 10000
      text = text[0 ..< 10000]
      cut  = text.lastIndexOf("\n")
      IF cut > 0  text = text[0 ..< cut + 1]
4 IF childrenWereProcessed is false
      text += "    ...\n"                     // four spaces, three dots, newline
      note("target_directory is too large (either too many children or their names are too long). Immediate children were truncated. Try doing more specific search or use terminal tool.")
5 IF any extension summary line was rendered
      note("File extension counts do not include files ignored by .gitignore.")
6 RETURN text + notes
```

Helper: one render pass. This is where the separator asymmetry and the indentation live.

```
renderPass(node, depth, showCounts, depthLimit)

1 indent = "  " repeated depth
   IF depth === 0  out = node.absPath + (node.absPath endsWith sep ? "" : sep) + "\n"
   ELSE            out = indent + "- " + basename(posixNormalize(node.absPath)) + sep + "\n"
2 merged = files as { type: "file", name } ++ dirs as { type: "dir", name, node }
   merged.sort by name.localeCompare                        // one list, not dirs first
3 childIndent = "  " repeated (depth + 1)
   FOR each entry in merged
       IF entry is a file
           out += childIndent + "- " + entry.name + "\n"   // no separator after a file name
       ELSE
           IF entry.node.childrenWereProcessed AND (depthLimit is undefined OR depth < depthLimit)
               out += renderPass(entry.node, depth + 1, showCounts, depthLimit)
           ELSE IF counts exist AND showCounts
               ext = top three extensions sorted by count descending, ties by name ascending,
                     each rendered "N *ext" with "no-ext" for a missing extension,
                     joined with ", ", plus ", ..." when more than three exist
               out += childIndent + "- " + entry.name + sep + "\n"
               out += childIndent + "  [" + node.numFiles + " file"
                      + (node.numFiles === 1 ? "" : "s") + " in subtree: " + ext + "]\n"
               mark that at least one extension count rendered
           ELSE
               out += childIndent + "- " + entry.name + sep + "...\n"
4 RETURN out
```

Helper: the note wrapper and the timeout suffix, which are applied by the render function rather than the tree builder.

```
render(record)

  undefined case        -> "Tool call unexpectedly didn't return result. Try using terminal command to list the directory instead."
  success or timeout AND no tree
                        -> "Tool call unexpectedly didn't return result. Try using terminal command to list the directory instead."
  success               -> formatTree(tree)
  timeout               -> formatTree(tree) + "\n\nNote: The operation timed out after 5 seconds. The result is incomplete."
  error                 -> "Error listing directory: ${error}"
  rejected              -> "Rejected: ${reason}"
  // isError is false from render; the outer wrapper forces true for error and rejected only,
  // so timeout reaches the model as a non-error with a usable partial tree

notes(notes[])
  count === 0 -> ""
  count === 1 -> "\nNote: " + notes[0]
  count >= 2  -> "\nNote:\n - " + join(notes, "\n - ")
```

### Worked examples

Every string below is taken from the renderer port that reproduces the shipped functions line for line, with the JSON form given alongside so the bytes are unambiguous. Lengths are JavaScript string lengths, which for this ASCII-plus-escapes content equal byte counts. Six vectors are the renderer's own output variants; the rest cover the executor, the sort, and the description-versus-behavior mismatches.

The renderer appends the separator to directories only. A file line ends with its name and a newline, with no trailing separator. The template is `<indent>- <file.name>\n` for files and `<indent>- <dir name><pathSep>\n` for directories, and the byte-exact length check on the main tree below confirms it. This matches the `### Output shape` paragraph above.

| Case | Input | Expected output |
|---|---|---|
| ordinary tree (confirmed) | `target_directory: C:\Users\dev\proj`, six files under five directories | the 142-character tree string below |
| empty directory (confirmed) | a directory containing nothing, or only empty subdirectories | `C:\Users\dev\proj\empty/\n... no children found ...`, 50 characters |
| file path instead of directory (confirmed) | `target_directory: C:\Users\dev\proj\README.md` | `Error listing directory: Path is not a directory: C:\Users\dev\proj\README.md` |
| missing path (confirmed) | `target_directory: C:\Users\dev\proj\gone` | `Error listing directory: Path does not exist: C:\Users\dev\proj\gone` |
| `.git` absent, other dotfiles present (confirmed) | a tree holding `.env`, `.gitignore`, `.git/HEAD` | `.env` and `.gitignore` listed with their dots; no `.git` line |
| merged sort, not directories first (confirmed) | a root holding `README.md`, `package.json`, `docs`, `src` | `docs`, then `package.json`, then `README.md`, then `src` |
| collapsed subtree with counts (confirmed) | `node_modules` carrying 5264 files over four extensions | `[5264 files in subtree: 4210 *.js, 890 *.json, 120 *.md, ...]`, 189-character total |
| collapsed subtree, single file (derived) | a collapsed directory holding one file | `[1 file in subtree: 1 *.rs]`, singular `file` |
| extension with no dot (derived) | a collapsed directory holding `.env` only | `[1 file in subtree: 1 no-ext]`, because `path.extname('.env')` is empty |
| collapsed subtree with no counts (confirmed) | counts off, or an empty counts map | `C:\Users\dev\proj\\\n  - vendor\\...\n`, 34 characters |
| overflow, short output (confirmed) | root marked unprocessed with one retained file | tree, then `    ...`, then the too-large note; 215 characters |
| overflow with no slicing (confirmed) | 40 directories of 5 files, rendered 4789 characters | tree plus the four-space dots line and the too-large note, because the walker budget latched even though nothing was sliced |
| timeout (confirmed) | walk exceeds 5000 ms | partial tree, blank line, then `Note: The operation timed out after 5 seconds. The result is incomplete.` |
| slashless `ignore_globs` (confirmed) | `ignore_globs: ["*.ts"]` | every `.ts` file excluded at any depth, because ripgrep treats a slashless glob as a basename match |
| `deep/**` in `ignore_globs` (confirmed) | `ignore_globs: ["deep/**"]` | nothing excluded; the promised `**/` prepend never happens, so the pattern is anchored at the walk root |

#### The main tree, byte for byte (confirmed)

A Windows workspace at `C:\Users\dev\proj` holding six files: `README.md`, `package.json`, `docs/guide.md`, `src/app.tsx`, `src/index.ts`, `src/utils/math.ts`. Input:

```json
{"target_directory": "C:\\Users\\dev\\proj"}
```

The model receives:

```text
C:\Users\dev\proj\
  - docs\
    - guide.md
  - package.json
  - README.md
  - src\
    - app.tsx
    - index.ts
    - utils\
      - math.ts
```

The same string as JSON, length 142:

```text
"C:\\Users\\dev\\proj\\\n  - docs\\\n    - guide.md\n  - package.json\n  - README.md\n  - src\\\n    - app.tsx\n    - index.ts\n    - utils\\\n      - math.ts\n"
```

Line by line, with measured leading spaces: root has 0 and no bullet; `docs` has 2; `guide.md` has 4; `package.json` and `README.md` have 2; `src` has 2; `app.tsx`, `index.ts`, and `utils` have 4; `math.ts` has 6. The root line ends with one backslash because the root path did not already end with the separator. Directory lines end with a separator, file lines do not. `package.json` precedes `README.md` because the sort is `localeCompare` on the merged file-plus-directory list, and ICU collation compares `p` against `r` case-insensitively at the primary level, so it is not the byte order that a naive string comparison would give (which would place `README.md` first). `docs` sits ahead of both root files because directories and files are sorted together, not grouped.

#### The same tree on POSIX (confirmed)

Input:

```json
{"target_directory": "/home/dev/proj"}
```

The model receives, length 139:

```text
/home/dev/proj/
  - docs/
    - guide.md
  - package.json
  - README.md
  - src/
    - app.tsx
    - index.ts
    - utils/
      - math.ts
```

Only the separator changed. It is chosen once, from whether the root path contains a backslash, and reused at every level. Three characters shorter than the Windows form because the root path is three characters shorter, which is the check that the tree text carries no per-line platform decoration.

#### An empty directory (confirmed)

Input:

```json
{"target_directory": "C:\\Users\\dev\\proj\\empty"}
```

The model receives, length 50, with no trailing newline:

```text
C:\Users\dev\proj\empty/
... no children found ...
```

The separator before the newline is a hard-coded forward slash, not the platform separator, so a Windows path shows backslashes throughout and then one forward slash at the end. There is no indentation before the three dots, and the string ends at the last `.` of `found ...`. A directory containing only empty subdirectories renders this same text, because the tree is built from file paths and a directory with no files below it never becomes a node.

#### Both notes, the common big-repo shape (confirmed)

Root `C:\repo` marked unprocessed, with one collapsed child `src` holding two files, one `.js` and one `.ts`. Input:

```json
{"target_directory": "C:\\repo"}
```

The model receives, length 319:

```text
C:\repo\
  - src\
    [2 files in subtree: 1 *.js, 1 *.ts]
    ...

Note:
 - File extension counts do not include files ignored by .gitignore.
 - target_directory is too large (either too many children or their names are too long). Immediate children were truncated. Try doing more specific search or use terminal tool.
```

The dots line is exactly four spaces then three dots, hard-coded rather than derived from the indent, so it does not follow the depth of the last rendered entry. The `Note:` line is bare with nothing after the colon. Each bullet begins with exactly one space, from the `"\n - "` join separator. The note order is fixed by push order: the extension-count caveat first, the too-large note second. The string ends without a newline, on the final `.` of `tool.`. This shape is what a large repository almost always produces, because the walker's 2500-character name budget latches the root to unprocessed long before the renderer's 10,000-character budget could matter.

What each example proves:

- The Windows tree pins the two-space indent, the `- ` bullet, the separator on directories only, and the merged `localeCompare` order where `package.json` precedes `README.md`.
- The POSIX tree pins that the separator is chosen once from the root path and applied uniformly, and that nothing else in the text is platform-dependent.
- The empty-directory example pins the early-return branch, the hard-coded forward slash, the unindented dots line, and the absence of a trailing newline.
- The both-notes example pins the four-space dots line, the fixed note order, the single-space bullet prefix, and the fact that the too-large warning fires on the walker's budget rather than on the rendered length.

*2026-10-27 - deepseek-v41-flash-1017*

## Harness design notes: what Cursor does beyond the file tools

This is the second half of the document. The first half specifies six file tools precisely enough to reimplement. This half covers the surrounding harness: how the prompt is assembled to survive caching, how subagents are isolated, how context is compacted, how the shell behaves, and how tools are offered and capped.

These are not tool contracts. They are design decisions, and most of them exist because of a problem every harness hits and few plan for. Each note says what Cursor does, why it matters, and what I would do differently.

Confidence labels follow the first half: unlabeled means read from the bundle, `(probe)` means observed by running the agent, and anything inferred says so.

### The prompt cache is the dominant constraint

Everything in this section traces back to one fact. Prompt caching gives you a large discount on the unchanged prefix of a conversation, and it works by matching an exact byte prefix from the start. The moment you change anything near the front, the entire cache is invalidated and you pay full price for the whole conversation again.

That has a counterintuitive consequence. It is cheaper to append a correction at the end of a long conversation than to fix a typo in the system prompt. Cursor's prompt assembly is shaped almost entirely by this.

#### Where environment information actually lives

The obvious design puts the workspace, the rules, and the tool catalog in the system prompt. Cursor does not. It puts them in a **synthetic first user message** wrapped in `<user_info>` tags, and leaves the system prompt much more stable.

Confirmed in the summarization code, which has to know where to find them:

```js
USER_INFO_TAG_REGEX = /<user_info>[\s\S]*<\/user_info>/;
SYSTEM_REMINDER_OPEN = "<system_reminder>";
SYSTEM_REMINDER_CLOSE = "</system_reminder>";
SYNTHETIC_USER_TURN_PROVIDER_KEY = "syntheticUserTurn";
```

And the tool descriptions refer the model to it rather than repeating the information:

```
Available dynamic tool namespaces are listed in <user_info> at the start of this conversation.
```

Why this works: a user message is a cheaper thing to modify than a system prompt, because a harness can append a new user turn without rewriting the prefix. And the summarization path can strip `<user_info>` blocks out of old turns because it has a regex for them, rather than having to reconstruct a system prompt.

**What I would do differently.** Copy the idea, not the implementation. The underlying insight is that you should partition prompt content by how often it changes, and put the volatile parts as late in the prefix as you can. A synthetic user message is one way. A system prompt with a strictly ordered, append-only tail is another and is easier to reason about.

#### The delta mechanism

When the tool catalog changes mid-conversation, Cursor does **not** rewrite the cached `<user_info>`. It appends a delta on a later turn:

```
<user_info_catalog_update>
```

The model is told to read the most recent update block "on a later user turn" and let it win. So the original message is never touched, the cache prefix survives, and the newest information still reaches the model.

That is the correct solution to a real problem. Tool availability genuinely changes mid-conversation when an MCP server reconnects or fails. Rewriting is the obvious fix and it destroys the cache; appending a delta is the non-obvious fix and it does not.

#### Instrumentation, and why it is worth copying

Cursor measures whether it rebuilt the cached prefix and why:

```js
prefixRebuiltCounter = createCounter("agent.prompt_prefix.rebuilt", {
  description: "Per-turn rebuild of a prompt-prefix component; `changed` is whether it " +
    "differs from the previous turn (`first` = no fingerprinted previous turn). " +
    "`tool_names` tracks the static tool name set only, not descriptions or schemas.",
  labelNames: ["component", "changed", "vendor", "model"]
});

systemPromptChangeCauseCounter = createCounter("agent.prompt_prefix.system_prompt_change_cause", {
  description: "Which fingerprinted input differed when the rebuilt system prompt changed " +
    "(one increment per differing input; ...)",
  labelNames: ["cause", "vendor", "model"]
});

userInfoRerenderedCounter = createCounter("agent.prompt_prefix.user_info_rerendered", {
  description: "Cached first-turn <user_info> replaced on a later turn, by reason " +
    "(one increment per contributing reason)",
  labelNames: ["reason", "vendor", "model"]
});
```

The change-cause counter is the useful one. It answers "why did my cache break this turn" with a named cause rather than a shrug. A harness without this is guessing, and cache invalidation is invisible from the outside except as cost.

A detail worth noting in the `rebuilt` description: it tracks the static tool **name** set only, explicitly not descriptions or schemas. That is a deliberate limitation. Names are cheap to compare; schemas are not, and a schema change that does not change names would still break the cache without being caught. Cursor chose the cheap check and documented the gap.

### Subagents

#### They start empty, deliberately

`applyConversationStateMapping` falls through to a fresh conversation state for every built-in subagent type. The parent's state is fetched and then discarded.

The consequence is that a subagent knows nothing about the parent's work unless the prompt tells it. Cursor compensates in the Task tool's own description, which instructs the parent to over-specify:

> the parent is ordered to over-specify the prompt

This is the right trade and it is worth understanding why. Inheriting the parent conversation would (a) share the cache prefix awkwardly because the subagent's system prompt differs, (b) blow the child's context before it did any work, and (c) make the child's output depend on irrelevant parent history. Starting empty costs you prompt length and buys you predictability.

**What I would do differently.** Nothing, on the isolation. But the "over-specify" instruction is doing a lot of work through prompt text alone. A harness could enforce it instead, by refusing to spawn a subagent whose prompt is shorter than some threshold, or by requiring a structured brief with named fields. Prompt text is a soft guarantee; a schema is a hard one.

#### What comes back is one message

`getTaskSuccessResultText` takes only the **last assistant message** from the child. Not the transcript, not a summary, not all messages. The parent sees one `<response>` wrapper, and the wrapper checks for a pre-existing one to avoid double-wrapping.

A child that runs twenty tool calls and then answers returns a single message. Everything else is discarded. That is the context-management decision that makes subagents worthwhile at all.

#### A dead child looks like a quiet one

This is the sharpest trap in the whole area:

`final_message` is optional, with `"No output"` as the fallback. A child that dies after its last tool call returns **success** with an empty response, which is indistinguishable from a child that genuinely had nothing to say.

If you are building this, make the distinction explicit. A child that exits without a final message should be a distinguishable outcome from a child that produced one.

#### Background subagents end the turn

A background subagent is not awaited. The turn ends, completion is queued, and the result is delivered at the end of a later turn. The parent may be given a transcript path for optional tailing, with an explicit prompt-level ban on mentioning that path to the user.

The ban is there because models leak implementation details. Note that it is again prompt text, not a filter. A harness that scrubs paths from output would be more reliable.

#### Await is cache-aware and refuses to be a poller

`Await` caps at 1 hour 59 minutes and generates TTL-specific advice telling the model that repeated 300-second waits are the worst possible choice. That advice is cache-aware: a short poll loop invalidates and re-reads the prefix repeatedly, so the cost is far worse than the wall clock suggests.

This is a good example of a harness encoding a cost model into prompt text. The model cannot see token economics, so the harness has to tell it.

### Context compaction

#### It fires at 90 percent, and the summary is a user message

Two decisions here, both cache-driven.

Compaction triggers at 90 percent of the window. That leaves headroom for the summary call itself, which needs the very context it is summarizing.

And the summary carrier is a **user message, never a system message**. A system message would change the prefix and invalidate everything. A user message appends.

#### The self-summary orders the query before the summary

When the agent summarizes its own work, it places the most recent user query **before** the summary text, so the model reads what it was asked and then reads what it has done. Reversing that order makes the model treat the summary as the instruction.

That is a small detail with a large effect, and it is the kind of thing you only learn by watching compaction go wrong.

### The swarm has no engine

`grind-swarm-subagent.js` is 332 bytes. There is no coordination code. The entire fan-out is prompt text.

Idle termination is not a timeout. It is a **proof-of-done token** planted at spawn:

```
___ESCAPE_TOKEN_PLACEHOLDER___  ->  {escape_token}
```

The subagent is instructed to emit the token when finished, and the parent watches for it. That is a more honest signal than a timer, because a genuinely long task does not get killed for being slow, and a stuck one does not linger.

**What I would do differently.** The token is good. The 332-byte module is a warning: if the coordination exists only as prose, nothing enforces it, and a model that ignores the instruction hangs the parent until whatever outer timeout exists. A token plus a real watchdog is the combination.

### Model routing for subagents

Explore is **downgrade-only**. Two independent mechanisms enforce it:

- `orderedExploreSubagentComposerIds`, an allowlist
- a `compareModelCosts` skip in the candidate loop

A search subagent can never cost more per token than its parent. That is a policy decision expressed as code rather than as configuration, which is why it holds.

The reasoning is sound: exploration is high-token and low-difficulty, so routing it to a cheaper model is almost free in quality and large in cost. Cursor hardcodes it rather than trusting a config.

### Skill and rule injection

Skill catalogs get **2 percent of the token budget** and degrade in four stages, including a binary search over description length to find what fits. File-scoped skills activate on **read paths** via glob matching, not on write paths.

The reminder that gets injected carries only the path and the description, never the body. The body is fetched only if the model chooses to read it. That is a lazy-loading design, and it is the only way a large skill library is affordable.

Activating on read paths rather than write paths is the interesting choice. It means a skill fires when the model looks at a file, which is before it decides what to do, rather than after it has already acted. For a style or convention skill, that ordering is the difference between useful and useless.

### Shell

The shell takes the **exclusive** lock, not a per-file lock:

```js
yPe(s, await o.waitForExclusiveLock(e), false);
```

While a shell command runs, every file operation in that agent stalls. This is coarse, and it is also correct, because a subprocess is opaque. You cannot track which files it touches, so the only safe answer is to stop everything else.

Note the ternary around it: if no lock manager is supplied, the shell runs as the bare function with no lock at all. The protection exists only when the harness supplies it.

Background shells are tracked in a map capped at 4096 entries, keyed by conversation and shell id, with an `exitReported` flag so a completion is announced once.

The shell is where Cursor makes its most consequential decisions, and several invert what a careful engineer would assume. The locking behavior above is the first of them.

#### A timeout does not kill the command

Every branch of `resolveShellExecutionPlan` hardcodes the same behavior:

```js
timeoutBehavior = TimeoutBehavior.BACKGROUND;
```

There is no kill path. When a command exceeds its window, it is **moved to the background and keeps running**. The model is told the command is now backgrounded, not that it failed.

`timeout: 0` means "start in the background immediately", because the normalizer treats zero as a real value rather than as absent:

```js
shouldStartInBackground = explicitlyRequestedBackground
  || backgroundAfterMs === 0
  || requestedBlockUntilMs === 0;
```

And there is a separate `hardTimeoutMs` of 24 hours on the `block_until_ms` path, which is the only actual limit.

Why this is right: killing a `npm install` mid-write leaves a corrupt `node_modules`, and killing a `git` command mid-operation can leave a lock file. Backgrounding is the safe response to "this is taking too long". A harness that kills on timeout is producing corrupt states and blaming the model for the fallout.

**The trap for a reimplementation.** If your shell tool reports a timeout as failure, the model will retry, and you will have two mutations running simultaneously on the same tree. Either background it like Cursor, or kill with an explicit "this command was terminated and may have left partial state" message. Do not report a silent failure.

#### A passing safety classification removes the sandbox

This is the most surprising thing in the whole sweep:

```js
function buildClassifierEscalatedSandboxPolicy(requestedSandboxPolicy, enableSandboxSharedBuildCache) {
  return new SandboxPolicy({
    type: SandboxPolicy_Type.INSECURE_NONE,
    enableSharedBuildCache: requestedSandboxPolicy?.enableSharedBuildCache ?? enableSandboxSharedBuildCache
  });
}
```

A function named "escalated" returns `INSECURE_NONE` unconditionally. It ignores `requestedSandboxPolicy` entirely except to copy one boolean out of it.

The effect: once the auto-review classifier approves a command, the sandbox is not merely relaxed, it is **removed**. And the name reveals the intent: the classifier *substitutes for* the sandbox rather than stacking with it. The reasoning is that if a model reviewed the command and decided it is safe, running it sandboxed as well is redundant overhead.

That reasoning is defensible and the consequence is severe. A classifier that approves a command it misjudged now has no second line of defense. The sandbox was the thing that would have limited the blast radius, and it is gone precisely in the case where the classifier was wrong.

**What I would do differently.** Keep both. A classifier approval should downgrade the sandbox from read-only to read-write, not to none. The two mechanisms catch different failures: the classifier catches intent, the sandbox catches consequences. Removing one when the other passes assumes they are measuring the same thing, and they are not.

Note also that the sandbox policy type is surfaced to the model as a string, and the enum values are `readonly`, `readwrite`, `none`, `unspecified`. A model told `none` knows it is unsandboxed, which is at least honest.

#### Interrupted output admits uncertainty

When a command produces no complete snapshot, the model is told the command **may still be running**, rather than being told it failed. That stops a model from blindly re-running something that is still mutating the tree.

The snapshot ring buffer trims at twice its 20 KB cap, which is hysteresis: trimming at exactly the cap on every write would be needlessly frequent.

**Worth copying verbatim.** "I do not know whether this finished" is a true and useful statement. Most harnesses collapse it into "failed", which invites a retry that makes things worse.

#### Truncation runs at four layers

| Layer | Threshold | Behavior |
|---|---|---|
| streaming suppression | 64 KiB/s or 2 MiB/s | suppress live updates, keep capturing |
| per-stream capture | 1 MiB | cap each of stdout and stderr |
| merged file | 49 MiB | cap the spilled file |
| model render | 10 K head + 10 K tail, 20,000 chars | what the model reads |

The head-plus-tail render is the important choice. Keeping only the head loses the error at the end of a build log. Keeping only the tail loses the command echo and early warnings. Both ends is the only rendering that preserves the two things a model actually needs.

#### Once output spills to a file, the prompt tells the model not to truncate

Above 40 KB, output spills to a file and the model is given the path. The prompt then explicitly instructs it **not** to run `head`, `tail`, or `sed -n`:

That inverts the usual "keep output small" instruction, and it is correct. The harness already decided what was worth storing. A model that pipes the file through `head` discards exactly the part the harness chose to keep, and gets a worse view than if it had read the file.

A harness that tells the model to keep output small while also spilling large output to disk is giving contradictory instructions.

#### Background shells: the state map is not where the data lives

The map capped at 4096 entries holds only ids, pid, and an `exitReported` flag. The actual scrollback lives in a file per shell at `terminals/<id>.txt`.

That means retrieving background output requires **no new tool**. It is a `Read` against a path, and a `Grep` to search it. The shell tool returns a path, and the file tools do the rest.

That is a genuinely elegant composition, and it explains why Cursor has no "background shell output" tool. It does not need one.

Background shell ids are random in the range 1000 to 999999, then monotonic afterward. The randomness exists because the id doubles as the terminal filename, so a fresh session is unlikely to reuse an id and truncate the scrollback of a shell still running from an earlier session.

#### The terminal file header is rewritten in place

Frontmatter is written at offset 0 of an `r+` handle every 5 seconds. So the header's byte length must not change between rewrites, or the write overruns into the body and corrupts the file.

This is why a `padEnd(9)` in the formatting code is load-bearing rather than cosmetic. It reserves a fixed-width field so the header length is stable as the values inside it change.

**A warning for anyone copying this.** In-place rewrite of a variable-length header is a footgun with a long fuse. Append-only is safer. If you must rewrite, pad every variable field to a fixed width and assert the length before writing.

#### The exec id is a cache key, not a cancel handle

The exec id is a deterministic SHA-256 UUID. It seeds an LRU keyed `${execId}:${sha256(args)}` holding a `ForkableIterable`.

The effect: a retried tool call **replays the first stream** instead of re-running the command. That is what stops a retried `git push` from pushing twice.

Cancellation does not use the exec id. It rides a separate numeric stream id.

That split is worth understanding. One identifier means "which computation was this" for caching and deduplication; a different one means "which live thing do I abort". Conflating them makes one of the two jobs impossible.

#### Approval is a two-field contract

When the classifier blocks a command, the model is instructed to retry with two fields:

- `request_smart_mode_approval: true`
- `smart_mode_block_reason: <the classifier text>`

The reason is that the frontend never saw the classifier's explanation, so the model has to carry it back. Without it, the approval UI would show a block with no justification.

The request fingerprint pins the command, the working directory, the sandbox policy, and whether it is backgrounded. It does **not** pin the justification, so a model could supply different reasoning for the same command.

#### Things in the shell code that do not do what they claim

Three, worth knowing because each is a trap if you assume otherwise:

`retry-helpers.js` contains no retry logic at all. The name is aspirational.

`analyzeFailure` classifies **any** failing `git` command as a network problem, which will mislead a model debugging a merge conflict.

The tree-sitter command parser degrades silently to `parsingFailed: true` when its native modules are missing. Every fail-closed gate downstream depends on that parser, so a missing native module does not fail to parse; it fails to gate.

*2026-10-27 - deepseek-v41-flash-1017*

### The write decoration point

After every write, Cursor runs `decoratePostWriteResultForModel`, which may append text to the model's result. Two decorators exist, and neither is what the name suggests:

- an "eager store conflict barrier" that waits up to 2 seconds for a concurrent-write check
- canvas diagnostics, but only for managed canvas paths

```js
DEFAULT_WRITE_BARRIER_TIMEOUT_MS = 2e3
eagerStoreConflictBarrier = createCounter("agent.store.eager_barrier", {
  description: "Eager same-result agent-store conflict barrier outcomes."
})
```

So for an ordinary source file, post-write decoration is **only** a conflict check. No linter runs, no parse check, no diff summary. The hook exists and the diagnostics are wired up, but they are gated to canvas paths.

The barrier itself is a timer-based guess at a concurrency question. Runvil's claims ledger answers the same question with vector clocks, which is strictly better information. The pattern worth taking is the **decoration point**: a hook after every mutation where the harness can append information the model needs, without the tool knowing about it.

Two properties of that hook are worth copying:

Decoration is **best-effort**. If the executor is missing or diagnostics time out, the model gets the undecorated result. A slow decorator never looks like a failed edit.

Decoration is **additive**. It appends; it does not replace the tool's own message. A successful write stays a successful write regardless of what the decorator found.

### Tool offering

Tool availability is gated per mode and per flag. `enableDeprecatedLsTool` defaults to false, which is why ListDir is usually absent. Names and descriptions vary per model family while parameter shapes largely do not. That is a deliberate asymmetry: instructions are tuned per model because instruction-following differs, while argument schemas stay fixed because a schema change breaks tooling for no benefit.

The exposure caps are worth stating precisely, because the obvious fields are dead. `maxMcpTools` and `warnMcpTools` appear on the chat config proto and are **read nowhere** in any bundle examined; each occurs exactly once, at its declaration. The live caps come from `MANAGED_LOCAL_LAUNCHED_DEFAULTS`:

```js
MANAGED_LOCAL_LAUNCHED_DEFAULTS = {
  maxDirectMcpTools: 100,
  maxAdvertisedTools: 300,
  enableToolArgPreservation: true,
  ...
}
```

`clampAdvertisedTools` and `clampDirectMcpTools` both sort before truncating, so overflow drops a deterministic subset rather than an arbitrary one, and `clampDirectMcpTools` reports `droppedByProvider` so a caller can tell which server was starved. If it cannot get under the limit even after dropping, it returns `stillOverLimit: true` and ships the over-limit list anyway rather than failing.

#### Discovery and invocation are two separate tools

A dynamic tool only works if **both** a discovery tool and an invocation tool are registered. If either is missing, the dynamic tools silently fold back into the static list, with no warning and no telemetry. The only symptom is a larger prompt.

That is worth knowing because the failure is invisible. You do not get an error; you get a harness that works but costs more every turn.

The discovery tool itself has three description-fidelity tiers plus a 200-character truncation, and spills anything over 12,000 bytes to a file under `agent-tools/`, replacing the payload with a two-field JSON stub. The stub has no discriminator field, so a reader cannot tell from the stub alone what it is a stub for.

Invocation refuses to run when required arguments are missing, and returns the **entire input schema inline** in the error, with the phrase "the call was not sent" so the model does not assume a partial effect occurred.

#### Caps that abort rather than degrade

| Cap | Value | Behavior on hit |
|---|---|---|
| tool calls per response | 500 | drop later chunks, abort the model request |
| steps per turn | 512 | error `Reached maximum number of steps before turn ended (possible looping?)` |

Both are hard stops rather than degradation. The 500-call cap counts distinct ids, aborts the stream mid-flight, and tells the model through a reminder appended to the last tool result rather than by injecting a fake user message. Appending to the existing result keeps the message history honest.

The 512-step error string is worth noting for its parenthetical. A client may raise the limit but never below the server's own, so the cap is a floor the client can only move up.

#### Loop detection uses two floors, and compares results not calls

The single best idea in this subsystem: the multi-message detector compares **tool result text** across repeated occurrences. Identical calls that return *changing* results are polling, not looping. Identical calls returning identical results are a loop.

Without that distinction, any legitimate poll gets killed. With it, the detector catches the actual pathology.

The single-message detector needs two floors at once: three repetitions **and** 100 total repeated characters. One floor alone flags `aaaa` and runs of `----`. There are also explicit exemptions for box-drawing characters and binary-like lines, for the same reason.

The whole computation runs under a 500 ms budget and **fails open** on timeout. That is the right bias: a slow loop detector that blocks the turn is worse than a missed loop.

#### Partial tool arguments throttle on the leading edge only

Partial arguments are streamed to the UI with a 500 ms throttle, leading edge, and **no trailing timer**. There is no automatic final flush, so `flush()` is manual and mandatory.

Forget it and the UI stays one field behind forever, which looks like a rendering bug rather than a missing call. If you build streaming tool arguments, use a trailing edge or a mandatory flush in the completion path, not a manual call that is easy to omit.

#### Parallelism is unbounded, and three locks do the ordering

Parallel tool calls are unbounded promises. Nothing caps concurrency. Ordering comes from the locks instead:

- writes take a per-path lock, with a starvation guard that lets an exclusive waiter ahead in the queue be served
- the shell takes an exclusive lock over all file operations
- a read that misses waits up to 120 seconds for same-step writers, then retries exactly once

So the concurrency model is "fire everything, let the mutexes sort it out". That works, and it means the lock correctness is load-bearing in a way it would not be if concurrency were capped.

#### Two traps in the plumbing

**The dsv3 generator only passes the lock manager to shell for `dsv3-1018`.** Newer dsv3 prompt versions therefore run the shell **without** the exclusive lock. So the protection described earlier in this document is version-dependent, and newer is not safer here.

**Display names leak into logic.** `SAME_STEP_FILE_WRITER_TOOLS` has to list both `Write` and `write`, because the set is keyed on the model-facing name and two model families disagree on casing. That is a direct consequence of the per-family renaming: any logic keyed on a tool name has to enumerate every alias, and a missing alias fails silently.

*2026-10-27 - deepseek-v41-flash-1017*
### What is not knowable from here

`numFilesForMemoryGeneration`, `memoryDefaultEnabled`, and `numSummarizationsBeforeWarningShown` are declared on the chat config proto but read nowhere in any of the three bundles examined. Each appears exactly once, at its declaration. So the fields exist and the feature does not, at least not in this build and not through these paths.

That is worth stating rather than guessing at thresholds, because a reader could otherwise assume a memory-generation feature is active and configured.

*2026-10-27 - deepseek-v41-flash-1017*
### Editing, diagnostics, and code intelligence

These notes cover what happens around an edit, which is where the largest gap between expectation and reality sits.

#### The model never sees its own diff

`renderEditResult` returns only the `message` field, which is one of six fixed sentences such as:

```
The file <path> has been updated.
```

`diffString`, `linesAdded`, `linesRemoved`, and both full file bodies ride the protobuf for the UI. The model gets a sentence.

This is deliberate and it is correct. Narrating a patch back to the model costs tokens it already spent producing the patch, and it gives the model an opportunity to re-read and re-edit work it just correctly performed. A harness that echoes the diff is spending context to make the model less confident.

**What to copy.** The sentence-only result, for the same reason. If you want the model to verify an edit, make verification a separate deliberate tool call, not an automatic echo.

#### No edit tool validates anything

There is no syntax check, no parse, no import resolution, no type check anywhere in the edit path. The only pre-write validation is a read-truncation guard:

```
File is too large to edit (${bytes} bytes): only part of it could be read, so writing the
edit would truncate it. Edit it with a shell command instead.
```

So an edit that produces a syntax error succeeds. The model learns about the error only if it chooses to call `ReadLints` or run a build.

**What I would do differently.** This is the single biggest gap I would close. Every modern harness has a cheap post-edit check available: parse the file, check the language server, or run a formatter in check mode. Cursor has a `decoratePostWriteResultForModel` hook already in place and uses it for store conflicts rather than for diagnostics, except on canvas files. Hooking a parse check there would catch the most common agent failure mode at near-zero cost.

#### `ReadLints` returns success when called with no paths

The schema advertises whole-workspace scope:

```
If not provided, returns diagnostics for all files in the workspace.
```

The implementation does not:

```js
const paths = rawArgs.paths || [];
for (const path60 of paths) { ... }
```

An empty array iterates zero times, no diagnostics are collected, and the result is:

```
No linter errors found.
```

So calling it with no arguments is indistinguishable from a clean bill of health. A model that wants whole-workspace validation, omits `paths` as the description invites, and gets `No linter errors found.` will conclude the workspace is clean. It is not; it was never checked.

This is the worst kind of bug: not a crash, a false negative that reads as a success. If you reimplement it, either implement the advertised behavior or change the description to require `paths`.

#### The false-positive warning is prompt text, not a filter

`ReadLints` can return diagnostics that existed before the model touched anything. Cursor addresses this entirely in prompt text:

```
This tool can return linter errors that were already present before your edits, so avoid
calling it with a very wide scope of files
```

and

```
NEVER call this tool on a file unless you've edited it or are about to edit it
```

There is **zero output truncation**. No cap, no filtering, no diffing against a baseline. The prompt is the only thing preventing a model from drowning in pre-existing errors.

Note the second instruction is capitalized, which in prompt engineering is a signal that the model ignored the lowercase version. That tells you something about how often it went wrong.

#### Staleness is structured, not prose

When a diagnostic is stale, it does not get a text suffix. It travels as a wire flag, `Diagnostic.is_stale` (proto field 6), renders as `, stale` on the row, and appends a `<system_reminder>` block through a structured `systemReminders` array that is sanitized before rendering.

Using a structured array rather than string concatenation matters because a diagnostic message can contain arbitrary text, including something that looks like a tag. Sanitizing before render is the defense.

#### ApplyPatch rolls back completely, and names your mistake

The patch path parses, applies in memory, diffs, then performs a single write. A failed hunk never touches disk. That is the right design and it comes for free from operating on a snapshot rather than streaming edits.

The error messages diagnose the actual mistake:

- for an Add operation: `add + to the beginning of each line`
- for an Update: `read the file before trying again`

Keying the hint on operation kind is a small touch that saves a model from a guessing loop.

#### `PUNCT_EQUIV` canonicalizes punctuation before matching

Before patch matching, a table folds smart quotes and en and em dashes to ASCII:

```
' ' -> '
" " -> "
en dash, em dash -> -
```

That single table eliminates an entire class of "the model cannot patch a file containing an em dash" failures, which otherwise presents as a mysterious not-found error when the file visibly contains the text the model sent.

Matching then runs three tiers: exact, `trimEnd`, and `trim`, with fuzz values 0, 1, and 100.

**Worth copying directly.** Smart punctuation is everywhere in prose-heavy repositories, and a model's tokenizer will not reliably reproduce the exact Unicode variant it read.

#### Diffs are computed off-thread above 256 KB

Above 256 KB summed across both sides, diffing moves off the main thread. `trimmedHunks` then does binary-search trimming on large diffs and bails out to a full diff if the surviving region drops below one eighth of the input.

That bail-out means latency is not monotonic in file size, which is worth knowing if you ever profile this and find a large file faster than a medium one.

#### `MultiStrReplace` is declared and never registered

There is a full zod schema, an `edit_tool_call` proto mapping, and an entry in `SAME_STEP_FILE_WRITER_TOOLS` next to a `MultiEdit` that does not exist either. The tool is never registered.

Its all-or-nothing semantics would come free from the same snapshot pattern ApplyPatch uses: transform over a snapshot, write once. So the design is sound and the implementation is absent.

Worth noting because it explains a ghost in several public tool lists: the capability is described in places because the schema ships, not because the tool runs.

### The one automatic verification in the system

The only automatic check-and-retry in the whole file-tool surface is in Read, and it is worth understanding because it is a template.

When a Read returns `file_not_found`, the tool:

1. releases its lock,
2. waits up to 120 seconds for earlier same-step writer tools to settle (`waitForEarlierCalls`),
3. re-acquires the lock,
4. retries exactly once.

Releasing the lock before waiting is the important part. Holding it while waiting for a writer that needs the same lock would deadlock. That ordering is a small thing that is easy to get wrong and produces a hang rather than an error.

The retry fires only for not-found and partial-image misses. It does not mask permission errors or generic failures.

**What to copy.** The pattern generalizes: when a read misses immediately after a write should have landed, the correct response is to wait for in-flight writers rather than to report the miss. Filesystem visibility is not always instantaneous, especially across processes or overlay layers.

### Code lineage

`code-lineage.js` surfaces past design decisions, reversions, and intentional tradeoffs from git history, and the plan tool's own prompt references it.

The service is an injected `aiAttribution` prop, so the response payload format is not in this bundle. What is knowable is the intent: before editing a file that has been deliberately shaped, tell the model why it looks that way, so it does not "fix" a considered decision.

That is a genuinely novel capability and it addresses a real failure: an agent that sees odd code, assumes it is a mistake, and simplifies away a workaround that exists for a reason. If you build one thing from these notes that Cursor does not fully expose, build this.

*2026-10-27 - deepseek-v41-flash-1017*
## Cursor versus Claude Code

This comparison exists for one practical reason. Cursor's file tools are demonstrably derived from Claude Code's: the hyphenated `-A`/`-B`/`-C` parameter keys are identical in both, the non-overlapping match advance is identical, and the `cat -n` claim in the descriptions is the same claim. When two implementations share an ancestor, knowing where they diverge tells you which choices were deliberate and which were inherited without much thought. That distinction is the difference between copying a behavior because it is good and copying it because nobody questioned it.

A side-by-side, because the two are frequently confused and Cursor's design is clearly derived from Claude Code's while diverging in specific ways. Every row is labeled.

### Tool names and parameters

| Aspect | Cursor | Claude Code |
|---|---|---|
| Read | `Read` / `read_file` | `Read` |
| Read path param | `path` or `target_file` | `file_path` |
| Write | `Write` / `write` | `Write` |
| Write content param | `contents` | `content` |
| Edit | `StrReplace` / `search_replace` | `Edit` |
| Edit params | `path`, `old_string`, `new_string`, `replace_all` | `file_path`, `old_string`, `new_string`, `replace_all` |
| Delete | `Delete` / `delete_file` | none (no dedicated tool) |
| Glob | `Glob` / `glob_file_search` | `Glob` |
| Grep | `Grep` / `grep` / `rg` | `Grep` |
| Grep context flags | literal `"-A"`, `"-B"`, `"-C"` | literal `"-A"`, `"-B"`, `"-C"` |
| Grep line numbers | not present | literal `"-n"` |

The `-A`/`-B`/`-C` hyphenated keys are identical in both, which is a strong hint about lineage.

### Matching algorithm

| Aspect | Cursor `(cursor)` | Claude Code `(claude)` |
|---|---|---|
| Exact match | `indexOf`, non-overlapping | `includes`, non-overlapping |
| Uniqueness required | yes, unless `replace_all` | yes, unless `replace_all` |
| Whitespace fallback | **yes**, line-trimmed block match | **no** |
| Quote folding | no | **yes**, four characters |
| `\uXXXX` unescape | no | yes |
| Similarity threshold | none | none |
| `...` elision | none | none |
| Overlap guard | **yes** | no |
| Empty `old_string` | rejected | means create-or-overwrite |

The important asymmetry: Cursor is more forgiving about whitespace and less forgiving about quotes. Claude Code is the reverse. Neither has fuzzy matching, despite what blog posts claim.

Cursor's line-trimmed matcher handles two real failure classes that Claude Code does not: tab-versus-space indentation, and U+00A0 non-breaking spaces, because `trim()` strips both. Claude Code has open issues for exactly those (#26996, #29870).

Both use the same non-overlapping advance rule, so `"aa"` in `"aaa"` is one match in both. Only Cursor adds the overlap guard that catches the resulting ambiguity.

### Read-before-write

| Aspect | Cursor | Claude Code |
|---|---|---|
| Gate enforced at runtime | **no** | **yes** |
| Applies to | neither StrReplace nor Write | both Write and Edit |
| Error for unread file | none | `File has not been read yet. Read it first before writing to it.` |
| Error for stale file | none | `File content has changed since it was last read...` |
| State store | none; the tool re-reads each time | class `Qs`, an LRU keyed by normalized path |
| Per-path state | n/a | `contentHash` (Bun.hash, base36), `readBaseline` (sha256 plus byte count), `contentLength`, `offset`, `limit`, `isPartialView`, `timestamp` |
| Detector | re-reads inside the tool; Write adds a sha256 check before truncate | `timestamp` comparison, then `contentHash` |
| Escape hatches | n/a | pre-read permission guard; for Edit only, "old_string still applies" with a `staleRecovered` note |
| Partial reads | n/a | fatal for Write, survivable for Edit |

The state store is richer than a simple timestamp map. Each entry carries both a fast hash (`contentHash`, base36) for equality checks and a `readBaseline` of sha256 plus byte count for the stronger comparison. `isPartialView` records whether the read covered the whole file, which is what makes a partial read fatal for Write but survivable for Edit. `contentNotInModelContext` tracks whether the bytes were actually shown to the model, which is a stricter notion than merely read.

Entries are written in Read's pipeline and read by Write's `i1o`, Edit's `validateInput`, and NotebookEdit.

This is the single largest design divergence. Cursor's answer to staleness is to always re-read, which makes a model-facing gate unnecessary. Claude Code's answer is to remember what was read, which detects staleness but requires the model to cooperate.

Neither is strictly better. Cursor's approach handles external modification without model cooperation. Claude Code's catches the case where a formatter rewrote the file between read and edit.

Worth noting that Cursor's Write does close its own internal race with the sha256 fingerprint check, refusing with `{path} changed on disk after this tool call read it...` if the file changed between its read and its truncate. So Cursor is not ignoring staleness; it is solving it inside the tool rather than at the boundary.

### Line number format

| Aspect | Cursor | Claude Code |
|---|---|---|
| Rendered as | `     1\|text` (six-wide, pipe) | `1<TAB>text` (no padding) |
| Separator detail | always `\|` | `\t` normally, `:` when `tabAwareSeparator` is set and content is tab-led |
| Trailing `\r` | preserved | stripped per line |
| Variants | three, per model family | one |
| Description claims | `cat -n` format | `cat -n` format |
| Actually matches `cat -n` | approximately | no |

Claude Code's template is exactly:

```js
function BZr(lineText, lineNumber, separator) {
  let s = lineText.endsWith("\r") ? lineText.slice(0, -1) : lineText;
  return `${lineNumber}${separator}${s}`;
}
```

with the separator chosen by:

```js
let s = tabAwareSeparator && (content.startsWith("\t") || content.includes("\n\t")) ? ":" : "\t";
```

So a tab-indented file gets `12:content` instead of `12\tcontent`, specifically so the line-number prefix is visually distinguishable from the content's own indentation. That is a genuinely thoughtful detail, and it exists because the tab separator is ambiguous when the content also starts with tabs.

The parse direction is a regex, used to strip the prefix back off:

```js
function npr(e) { return e.match(/^\s*\d+[\u2192\t:](.*)$/)?.[1] ?? e; }
```

Note `\u2192` (the arrow character) in the character class, which suggests some path emits an arrow separator too.

Both descriptions claim `cat -n`. Only Cursor's default is close, and even that pads to six where `cat -n` pads to six but uses a tab. Claude Code's unpadded tab form is arguably closer, but neither claim is exactly true.

### Result caps

| Aspect | Cursor | Claude Code |
|---|---|---|
| Read character cap | 100,000 chars | 150,000 chars (`ykr`), plus token-cap pagination |
| `maxResultSizeChars` | not applicable | Infinity for Read, so the char cap governs |
| Glob result cap | 2000 returned, 10000 parsed | 100 files |
| Glob sort | newest-first, JS | oldest-first, `--sort=modified` |
| Grep default limit | none (2000 practical) | 250 |
| Grep stdout cap | 8 MiB | 20,000 chars |
| Grep default mode | `content` | `files_with_matches` |
| Tool timeouts | none per tool; 120 s read-retry wait, 25 s glob | one 10 s write-commit guard (`l1o`) |

Interesting that neither has per-tool timeouts for Read, Edit, Grep, or Glob. Cursor's 25 s is a Glob wall clock and its 120 s is a read-retry wait, neither of which is a general tool timeout. Claude Code's single 10 s constant guards only the Write commit race.

The Glob cap difference is large and matters. 2000 versus 100 is a 20x difference in what a single call can return.

The Grep default mode difference also matters: asking Cursor for matches returns lines, asking Claude Code returns filenames.

### Description accuracy

Both have a description that is wrong about its own tool. Claude Code's Read claims "reads up to 2000 lines by default" and "cat -n format"; `limit` actually defaults to `undefined` and the window runs to end of file, truncated on tokens.

Cursor's inaccuracies are smaller: the `cat -n` claim, and ListDir's dotfile claim.

If you are writing descriptions, treat both as cautionary examples. The description is the model's contract, so drift between it and the implementation produces behavior the model cannot predict.

### What to copy, what to fix

| Decision | Recommendation | Confidence |
|---|---|---|
| StrReplace matching pipeline | copy exactly, including step order | high |
| StrReplace whitespace fallback | copy; it solves real cases | high |
| StrReplace overlap guard | copy; it prevents silent under-replacement | high |
| Line number format | copy the default style | high |
| Write encoding and BOM preservation | copy; the alternative corrupts files | high |
| Mixed line-ending merge | copy; probes confirm it | high |
| Read 100,000 char cap | copy; the failure mode is well-defined | medium |
| Glob 2000 cap | copy; 100 is too small for modern repos | medium |
| Delete error-message bug | **fix**, note the divergence | high |
| Delete symlink undo snapshot | **fix**, snapshot the link | high |
| Write atomicity | **fix**, temp plus rename | high |
| Write executable-bit loss | **fix**, preserve mode | medium |
| Lock keying on raw string | **fix**, key on canonical path | high |
| Per-agent lock scope | **fix** if multiple agents share a process | high |
| Read-before-write gate | your call; Cursor omits it deliberately | medium |
| ListDir dotfile description | **fix**, it contradicts behavior | high |

*2026-10-27 - deepseek-v41-flash-1017*

## Failure matrix

Every failure across all seven tools in one place, sorted by tool and then by the order the condition is checked within that tool. `Result case` is the structured record case, not the rendered polarity, and the two differ in several places on purpose. Where a tool cannot produce a case at all, the row says so rather than leaving the cell blank.

Interpolated parts appear as `${...}` exactly as the document writes them elsewhere. A note on reading the `ReadFile` rows: the tool throws a plain message, `serializeError` converts that throw into an `error`-case record, and `render` prefixes `Error: `, so the model-visible text for a Read failure is `Error: ` plus the message below unless the row says otherwise.

| Tool | Condition | Exact model-visible text | Result case | Notes |
|---|---|---|---|---|
| ReadFile | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | Thrown by the parse wrapper before the tool body, so no lock is taken and no file is touched |
| ReadFile | `offset` supplied as 0 | (none, rewritten to 1) | `success` | The zod refine permits 0 and a transform coerces it, so the `Offset must be >= 1.` message overstates the rule |
| ReadFile | `limit` supplied as 0 | `Limit must be >= 1.` | `error` | Genuinely rejected, unlike `offset: 0` |
| ReadFile | path extension is in the binary list | `Cannot read binary files of type ${ext}` | `error` | Tested before any I/O, so a rejected path costs nothing. `${ext}` is lowercased; falls back to `unknown` |
| ReadFile | service returns `fileNotFound` and the retry does not fire | `File not found` | `fileNotFound` | No path and no punctuation, unlike Write and StrReplace which include the path |
| ReadFile | service returns `permissionDenied` | `Permission denied` | `permissionDenied` | Also no path |
| ReadFile | path is a directory | `Path is a directory, not a file: ${path}` | `notFile` | `${path}` is the `lke`-normalized argument, not the resolved absolute path |
| ReadFile | path is a device, socket, or other non-file | `Path is neither a file nor a directory: ${path}` | `notFile` | Service reason, surfaced through `invalidFile` |
| ReadFile | content sniffs as binary | `File content is binary data, not text: ${path}` | `notFile` | The service returns `invalidFile`; the tool renders it through the `notFile`-shaped string |
| ReadFile | image payload cannot be decoded | `File is not a valid image: its data could not be decoded (${reason}). A truncated or hand-copied file does this; write it again from the original bytes with code.: ${path}` | `notFile` | The doubled punctuation before `: ${path}` is real, from concatenating a sentence-final reason with a colon join |
| ReadFile | service returns `invalidFile` with no reason | `Path is not a valid file to read: ${path}` | `notFile` | Fallback only; every observed service reason supplies its own text |
| ReadFile | service returns `rejected` | `Read operation rejected` | `rejected` | The user's own reason is dropped in favour of this fixed string |
| ReadFile | service returns `error` | `Error: ${errorMessage}` | `error` | Passed through verbatim with the renderer's prefix |
| ReadFile | empty file and the service applied no range | `File is empty.` | `success` | Success with `totalLines: 0` and no `readRange`; `isError` false |
| ReadFile | windowed content exceeds 100000 characters | `Requested content exceeds maximum allowed characters (100000 characters). The file is ${fileSize} bytes.\nPlease use offset and limit parameters to read specific portions of the file, or use the 'grep' tool to search for specific content.` | `success` | `exceededLimit` is a success record rendered with `isError` true. The check runs after windowing, so a small window into a huge file never trips it |
| ReadFile | offset at or past EOF | `Offset ${offset} is beyond file length (${totalLines} lines)` | `error` | `${offset}` is the raw argument. Clamping runs first, so a large negative offset saturates to 0 and cannot reach this |
| ReadFile | result case is unrecognized | `Unknown error` | `error` | Thrown as a bare `Error` from the result switch |
| ReadFile | blob output cannot be hydrated | (internal throw, not model-visible) | n/a | `Cannot hydrate blobs for tool Read: no blob store available` crashes the render rather than producing a tool error |
| WriteFile | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | `contents` has no minimum length, so only a missing field or wrong type fails |
| WriteFile | the path field never completed in the argument stream | `Failed to acquire file lock - path may not be parsed` | `error` | Distinct from a lock timeout; no timeout exists |
| WriteFile | the tool's own read returns `fileNotFound` | (none) | `success` | Not a failure at all. This is the new-file path, with `old` undefined |
| WriteFile | the tool's own read returns non-text output | `File is not a text file (binary data)` | `error` | The internal read's binary case, distinct from the executor's binary message |
| WriteFile | the tool's own read returns `permissionDenied` | `Read permission denied: ${path}` | `permissionDenied` | Note "Read", even though this is a write tool |
| WriteFile | the tool's own read returns `invalidFile` | `${reason \|\| "Path is not a valid file to read"}: ${path}` | `notFile` | Same shape as Read, but the renderer adds `Error: ` |
| WriteFile | the tool's own read returns `rejected` | `${reason}` | `rejected` | No `Edit rejected:` prefix on this path, unlike StrReplace |
| WriteFile | the tool's own read returns an unknown case | `Unhandled readResult.result: ${String(result)}` | `error` | Invariant failure string, model-visible |
| WriteFile | plan mode and the path is not markdown or canvas | `Plan mode can only edit markdown files and canvas files. This edit was blocked because the target is not allowed in plan-mode. If the user wants implementation changes outside markdown or canvas files, use the ${SwitchMode} tool with target_mode_id=agent first.` | `rejected` | The thrown text is the terse `Cannot edit non markdown files in plan mode`; the renderer substitutes this long form |
| WriteFile | malformed `file://` URL | `${path} is not a valid file URL. Use an absolute path instead.` | `error` | A well-formed `file://` URL is accepted and normalized instead |
| WriteFile | target is in another worktree | `You cannot search other worktrees for this repository, stay within your workspace paths.` | `error` | Says "search" in a write tool |
| WriteFile | the fingerprint shows the parent directory disappeared | `${path} changed on disk after this tool call read it. Nothing was written. Read the file again and retry.` | `error` | Fires before `mkdir` |
| WriteFile | file changed between the tool's read and its truncate | `${path} changed on disk after this tool call read it. Nothing was written. Read the file again and retry.` | `error` | Same string as the parent-directory check; this is the one that closes the race |
| WriteFile | write permission denied | `Write permission denied: ${path}: ${error}` | `permissionDenied` | Note "Write" here and "Read" for the internal read failure above |
| WriteFile | write permission denied while in read-only mode | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` | `permissionDenied` | `isReadonly` substitutes this for the whole message |
| WriteFile | target is a directory | `Path is a directory, not a file: ${path}` | `notFile` | Arrives through the internal read, not the write executor |
| WriteFile | symlink at the leaf | `Refusing to write through a symlink at ${path}` | `permissionDenied` | Wrapped as `Write permission denied: ${path}: Refusing to write through a symlink at ${path}` by the permission-denied construction |
| WriteFile | parent resolves through a symlink elsewhere | `Refusing write: ${path} now resolves through a symlinked parent to ${real}` | `permissionDenied` | Same wrapping as the row above |
| WriteFile | target is not a regular file | `Refusing to write to non-regular file at ${path}` | `permissionDenied` | Same wrapping again |
| WriteFile | no space left on device | `No space left on device` | `error` | Its own `noSpace` case internally, rendered through the generic error path |
| WriteFile | storage quota exceeded | `Agent-store quota exceeded: this write was rejected because the storage quota is full. Nothing was written; your local copy is intact. Delete or truncate files to free space (including resolved *.conflict* files), then retry. Do not retry the same write without freeing space first.` | `error` | Long by design; do not shorten it |
| WriteFile | user denies the approval prompt | `Rejected: ${reason}` | `rejected` | No `Edit rejected:` prefix, which is the StrReplace form |
| WriteFile | `contents` is the empty string | `Wrote contents to ${path}` | `success` | **Not a failure.** Truncates the file to 0 bytes and reports success. Implementations that skip the write diverge here |
| WriteFile | file not read first by the model | `Wrote contents to ${path}` | `success` | **Not a failure.** There is no read-before-write gate and no error string for one |
| StrReplace | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | `replace_all` carries no default on the default branch, so it arrives `undefined` |
| StrReplace | the path field never completed in the argument stream | `Failed to acquire file lock - path may not be parsed` | `error` | Same string as Write |
| StrReplace | plan mode and the path is not markdown or canvas | the long plan-mode paragraph, identical to Write's | `rejected` | The gate runs at the top of both the read stage and the write stage, so it can fire twice in one call |
| StrReplace | read stage returns `fileNotFound` | `File not found` | `fileNotFound` | No path, matching Read rather than Write |
| StrReplace | read stage returns `permissionDenied` | `Read permission denied: ${path}` | `permissionDenied` | Includes the path, unlike Read's bare `Permission denied` |
| StrReplace | read stage returns `invalidFile` | `${reason \|\| "Path is not a valid file to read"}: ${path}` | `notFile` | |
| StrReplace | read stage output is not text | `File is not a text file (binary data)` | `error` | |
| StrReplace | read stage was truncated | `File is too large to edit (${bytes} bytes): only part of it could be read, so writing the edit would truncate it. Edit it with a shell command instead.` | `error` | The ` (${bytes} bytes)` part is omitted entirely when the reported size is 0 |
| StrReplace | read stage returns `rejected` | `Edit rejected: ${reason}` | `rejected` | Carries the `Edit rejected:` prefix, unlike Write's read-stage rejection |
| StrReplace | `old_string` is empty | `old_string is empty` | `error` | Client-visible half is `Incorrect tool arguments`. Fires before any search |
| StrReplace | normalized `old_string` equals normalized `new_string` | `old_string and new_string are exactly the same` | `error` | Client-visible half is `Incorrect tool arguments`. This is why a de-indent attempt fails while a re-indent succeeds |
| StrReplace | zero matches, exact scan only | `The string to replace was not found in the file.` | `error` | Client-visible half is the same string with a period |
| StrReplace | zero matches after the whitespace fallback also failed | `The string to replace was not found in the file (even after relaxing whitespace).` | `error` | The client half stays the short form, so the two sides differ |
| StrReplace | zero matches with a fuzzy suggestion available | `The string to replace was not found in the file${relaxed}.\n Found a possible fuzzy match, did you mean:\n${numbered snippet}` | `error` | The hint is advice only and never changes whether the edit applies |
| StrReplace | multiple matches and `replace_all` false | `The string to replace was found multiple times in the file. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` | `error` | Client-visible half is `The model provided an ambiguous search string to replace` |
| StrReplace | multiple matches found only after relaxation | `The string to replace was found multiple times in the file after relaxing whitespace. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` | `error` | ` after relaxing whitespace` is inserted before the period |
| StrReplace | self-overlapping needle with one non-overlapping match | `The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.` | `error` | Catches `aa` in `aaa`, which the non-overlapping scan counts as one match |
| StrReplace | write stage permission denied | `Write permission denied: ${path}` or `Write permission denied: ${path}: ${error}` | `permissionDenied` | The suffix appears only when the executor supplies an error string |
| StrReplace | write stage in read-only mode | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` | `permissionDenied` | |
| StrReplace | write stage reports no space | `No space left on device` | `error` | |
| StrReplace | write stage rejected | `Edit rejected: ${reason}` | `rejected` | |
| StrReplace | write stage error | `Error: ${modelVisibleError \|\| error}` | `error` | |
| StrReplace | `replace_all` true, one match | `${n} occurrence in file ${path} has been updated.` | `success` | **Not a failure.** The singular/plural pairing is `1 occurrence ... has`, `3 occurrences ... have` |
| StrReplace | `replace_all` true, N matches | `${n} occurrences in file ${path} have been updated.` | `success` | **Not a failure.** `replace_all` on a self-overlapping needle replaces fewer times than `String.replaceAll` would |
| StrReplace | `replace_all` false, one match | `The file ${path} has been updated.` | `success` | **Not a failure.** No count is reported even though exactly one edit happened |
| Delete | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | |
| Delete | path is empty or whitespace-only | `Error deleting file : Error deleting file ${path}: path is empty or only whitespace` | `error` | The executor's `error` case is thrown like any other, so the renderer's `Error deleting file ${path}: ` wrapper is applied on top of the inner message, and `${path}` in the wrapper is empty |
| Delete | file does not exist | `Error deleting file : File not found: ${path}` | `fileNotFound` | Doubled prefix from the same serialization round trip. The thrown text is the cleaner `File not found: ${path}` |
| Delete | path is a directory or other non-file | `Error deleting file : Path is not a file (${actualType}): ${path}` | `notFile` | `${actualType}` is only ever `directory` or `other` |
| Delete | write-permission gate denies | `Permission denied: ` | `permissionDenied` | Empty path again, unless `isReadonly` |
| Delete | read-only mode | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` | `permissionDenied` | Both halves of the throw carry this string, and `render` short-circuits to `vG` before interpolating the path, so there is no empty slot |
| Delete | `unlink` returned `EBUSY` | `Error deleting file : File is busy: ${path}` | `fileBusy` | The only producer of `fileBusy` in the whole tool surface |
| Delete | user rejected the approval | `File deletion rejected: ${reason}` | `rejected` | The one failure that does not get a doubled prefix, because `reason` survives serialization |
| Delete | user rejected with an empty reason | `File deletion rejected: ` | `rejected` | Empty slot where the reason would be |
| Delete | file changed between the read and the unlink | `Error deleting file : Error deleting file ${path}: File changed while deleting; nothing was removed` | `error` | Doubled prefix plus the inner message |
| Delete | `unlink` returned `EACCES` | `Error deleting file : Permission denied: ${path}` | `permissionDenied` | `isReadonly` is not set on this path, so the ask-mode text does not substitute |
| Delete | `unlink` returned `EPERM` | `Error deleting file : Error deleting file ${path}: ${osMessage}` | `error` | **Not special-cased.** On Windows this is common for read-only or open files |
| Delete | any other executor error | `Error deleting file : Error deleting file ${path}: ${error}` | `error` | |
| Delete | deletion succeeded | `Successfully deleted file: ${path} (${bytes} bytes)` | `success` | **Not a failure.** `1 bytes` has no singular form |
| Glob | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | |
| Glob | pattern normalizes to `**/` | `glob_pattern is empty. Pass "*" to match every file.` | `rejected` | Raised as an invalid-args error; a whitespace-only pattern is not caught and matches nothing instead |
| Glob | target does not exist | `Path does not exist: ${absolute}` | `error` | Also the message a missing `rg` binary produces, because the spawn failure is reported as a missing cwd |
| Glob | target is filtered by `.cursorignore` | `Path '${path}' is filtered out by .cursorignore` | `rejected` | |
| Glob | target is neither a file nor a directory | `Path '${path}' is neither a file nor a directory` | `notFile` | |
| Glob | `realpath` failed while resolving symlinks | `Path '${path}' could not be verified while resolving symlinks. This can happen when access is denied or the path contains a symlink loop.` | `error` | |
| Glob | target spans multiple workspace roots | `Path '${path}' points to a different folder from each workspace root; use an absolute path` | `error` | |
| Glob | invalid glob syntax | `rg: error parsing glob '${pattern}': ${reason}` | `error` | Forwarded from ripgrep verbatim, including the `rg:` prefix |
| Glob | ripgrep was killed at the 25 s wall clock | `Timed out after 25s` | `error` | Partial results survive a timeout when at least one line was read |
| Glob | executor returned neither success nor error | `glob_file_search didn't return the result` | `error` | The string still says `glob_file_search` on every branch, including the `Glob` ones |
| Glob | zero matches | `Result of search in '${path}': 0 files found` | `success` | **Not a failure.** Rendered as a normal result with `isError` false |
| Glob | over 2000 files | header plus a mid-list cut or the trailing banner | `success` | **Not a failure.** In practice the 5000-character render budget binds first, at roughly 160 files |
| Grep | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | |
| Grep | `pattern` is the empty string | `pattern is empty. Pass "." to match every line.` | `rejected` | Only the exact empty string; `" "` passes through as a literal |
| Grep | `-A`, `-B`, or `-C` is negative | `${flag} is ${value}. Pass 0 or more context lines.` | `rejected` | Checked on the raw value before clamping, so a negative number is reachable |
| Grep | broad-glob guard trips | `Glob pattern "${glob}" matches every file and is not allowed. Use a more specific glob or no glob.` | `rejected` | Gated behind a feature flag; the set is exactly `**/*` and `**/**` after trimming |
| Grep | target does not exist | `Path does not exist: ${absolute}` | `error` | |
| Grep | target spans workspace roots | `Path '${path}' points to a different folder from each workspace root; use an absolute path` | `error` | Same string as Glob |
| Grep | target filtered by `.cursorignore` | `Path '${path}' is filtered out by .cursorignore` | `rejected` | |
| Grep | `realpath` failed | `Path '${path}' could not be verified while resolving symlinks. This can happen when access is denied or the path contains a symlink loop.` | `error` | |
| Grep | invalid regex | `rg: regex parse error: invalid regex "${pattern}": ${reason}.` | `error` | Rewritten from ripgrep's multi-line stderr. The escape hint is appended only for unclosed, unopened, and repetition-operator reasons |
| Grep | unrecognized file type | `unrecognized file type: ${type}` | `rejected` | Forwarded from ripgrep; classified as a user-facing error |
| Grep | ripgrep path not configured | `ripgrep path not configured` | `error` | |
| Grep | search exceeded 25 s | `Timed out after 25s` | `error` | |
| Grep | search was cancelled | `Grep was cancelled` | `error` | |
| Grep | ripgrep exited non-zero with no stdout | `ripgrep exited with code ${code}` | `error` | Fallback when stderr is empty |
| Grep | result case is unrecognized | `Unexpected result case: ${case}` | `error` | |
| Grep | zero matches at offset 0 | `No matches found` | `success` | **Not a failure.** `No files with matches found` in files mode |
| Grep | no matches and ripgrep was truncated | `No matches found before the search stopped. Narrow the search to see the rest.` | `success` | **Not a failure.** The files-mode equivalent keeps its own noun |
| Grep | offset beyond the available results | `No more results at offset=${offset}.` | `success` | **Not a failure.** A ` Narrow the search to see the rest.` suffix is appended when more results existed |
| Grep | results were cut by a cap | `[Showing ${shown} of ${total} matching lines. Use offset=${next} to see more, or narrow the search.]` | `success` | **Not a failure.** Four shapes exist; `Results truncated. ` prefixes the ones where a cap hid content before the end |
| ListDir | argument fails schema validation | `Invalid arguments:\n${key}: ${message}` | `error` | |
| ListDir | target does not exist | `Error listing directory: Path does not exist: ${resolved}` | `error` | `fileNotFound` is not one of this tool's four cases, so it folds into `error` |
| ListDir | target is a terminal log path with no terminals | `Error listing directory: No active terminals.` | `error` | Only for `.cursor/projects/<id>/terminals/<n>.txt` with a failed stat |
| ListDir | stat failed for another reason | `Error listing directory: ${rawOsMessage}` | `error` | Typically `EACCES: permission denied, stat '...'`. There is no `permissionDenied` case |
| ListDir | target exists but is not a directory | `Error listing directory: Path is not a directory: ${resolved}` | `error` | `notFile` is not one of this tool's cases either |
| ListDir | ripgrep failed with no output | `Error listing directory: Ripgrep failed (exit ${n}) with no results` | `error` | `: ${stderr}` is appended when stderr is non-empty |
| ListDir | rejection | `Rejected: ${reason}` | `rejected` | |
| ListDir | the walk exceeded 5 s | tree so far, then `\n\nNote: The operation timed out after 5 seconds. The result is incomplete.` | `timeout` | **Not an error.** The partial tree is usable and `isError` is false |
| ListDir | no result or no tree on success | `Tool call unexpectedly didn't return result. Try using terminal command to list the directory instead.` | `error` | Also emitted when the result case is undefined |
| ListDir | non-Error thrown inside the executor | `Error listing directory: Unknown error occurred` | `error` | |
| ListDir | directory contains no files at all | `${root}/\n... no children found ...` | `success` | **Not a failure.** Note the hard-coded forward slash even on Windows |
| ListDir | tree exceeded the walk budget | tree text with `    ...\n` and a note about immediate children being truncated | `success` | **Not a failure.** Produced by the 2500-character walk budget, not the 10000-character render budget |

### Result-case semantics

A tool call produces two things and the model only ever receives the second. Understanding the gap explains several rows above where the text and the polarity appear to disagree.

The first is a structured record: a protobuf `oneof` carrying exactly one case tag, `success`, `fileNotFound`, `notFile`, `permissionDenied`, `fileBusy`, `rejected`, or `error`, plus typed fields. It is never empty and never holds two cases. The harness uses it for rendering, telemetry, undo bookkeeping, and the diff view.

The second is the rendered text, produced from that record by the tool's render function. That text is what gets appended to the conversation. The model never sees the record, so these fields exist and are accurate while being invisible: `linesAdded`, `linesRemoved`, `diffString`, `beforeFullFileContent`, `afterFullFileContent`, `fileSize` (except where a message interpolates it explicitly), `readRange`, `isReadonly`, `relatedCursorRules` counts, and every telemetry label. In `WriteFile` and `StrReplace` the whole diff is client-only: the model gets one flat sentence.

Three mechanics follow from the split and each one shows up in the matrix.

First, a case is not the same thing as an error flag. `ReadFile`'s `exceededLimit` is a `success` record rendered with `isError: true`. `ListDir`'s `timeout` is a non-success case rendered with `isError: false`. `Delete`'s render returns `isError: false` for every case, and an outer wrapper then forces it true whenever the call threw. So checking the case tag and checking the flag give different answers in all three tools.

Second, a throw does not reach the model as a throw. `serializeError` converts it back into a result record, and that record may lose fields. Delete is the clear case: `path` is dropped for three of its cases, so the render template interpolates an empty string and the doubled prefix appears. Write and StrReplace route their throws through field-carrying error classes instead, which is why they do not have the empty slot.

Third, the client-visible and model-visible halves of one throw can differ. `StrReplace`'s string-not-found error carries a short fixed form to the UI toast and the long form with the fuzzy hint to the model. Same throw, two strings, and a reimplementation that shares one string between them will look correct in the transcript and wrong in the toast, or the reverse.

The practical rule for a reimplementation: keep the record and the renderer separate, and decide per case whether the flag is derived from the case or from whether the call threw. Cursor derives it from the throw for Delete and ListDir, and from an explicit argument for ReadFile's cap. Merging the two layers makes both behaviors unreproducible.

## Build order

The stages below are ordered by dependency, not by importance or difficulty. Each one either produces something the next stage needs or establishes a shape that would be expensive to change later. Stage 1 is pure arithmetic over strings, so it can be tested to completion before any filesystem exists. Stage 2 fixes the result shape that everything downstream branches on. Read and Write come before StrReplace because StrReplace is defined in terms of the read stage and writes through Write's executor. The lock comes last because its only purpose is to serialize tools that already exist, and scoping it before you can observe the tools runs is guesswork.

If you are implementing this from scratch, this order minimizes rework. Each stage is testable before the next depends on it.

### Naming your own identifiers

The recovered real names in the Glossary are a reasonable starting point, with two adjustments.

Some of Cursor's names are shaped by history rather than clarity. `createSuccessResult` is generic to the point of ambiguity and exists in several unrelated forms across the codebase. `READ_CHAR_HARD_LIMIT` being an alias of `CHAR_HARD_LIMIT` hints at a refactor rather than a deliberate design. Pick names that describe your abstraction, not Cursor's.

And do not let two different behaviors share one name. A minifier can collapse distinct functions to the same short name in different scopes without harm; readable code cannot. If you end up with two things called a "result," and you will, one needs a more specific name.

A workable starting set:

| Concept | Suggested name |
|---|---|
| line-number rendering | `renderNumberedLines` |
| the three styles | `StyleDefault`, `StyleGpt5`, `StyleCodexCatN` as an enum |
| read window arithmetic | `computeReadWindow` |
| read result construction | `buildReadResult` |
| int32 coercion | `clampToInt32` |
| the lock | `FileOperationLock` |
| read character cap | `MAX_READ_CHARACTERS` |
| binary extension check | `hasBinaryExtension` |
| the wait set | `MUTATING_TOOL_NAMES` |
| ask-mode refusal | `ASK_MODE_REFUSAL` |

The one hard requirement is consistency: whatever the name, use it everywhere and never let two different behaviors share one.

**Stage 1. The formatter and the window arithmetic.**
`hH`, `fH`, `kTe`, `STe`, `bTe`. No I/O. Pure functions over strings and integers. Get these exactly right first, because every read result flows through them and a formatting bug will look like a tool bug for a long time.

Stage 3 needs something Stage 8 has not built yet, so stub it. Read's algorithm acquires a per-file lock as its second step, and the real lock is not built until Stage 8. Use a no-op implementation first:

```js
// Stage 3 stub. Replace in Stage 8.
function createLockManager() {
  return {
    async waitForLock() { return { [Symbol.dispose]() {} }; },
    async waitForExclusiveLock() { return { [Symbol.dispose]() {} }; },
  };
}
```

That keeps the call site honest, since every tool takes the lock even when it does nothing, and swapping in the real implementation later requires no changes at any call site. Do not skip the call site and add it later; that is how locks end up missing from one tool.

Tests: empty string is 1 line; trailing newline creates a trailing empty line; negative offset reads `|offset|` lines; offset past EOF produces the exact error; sparse mode suppresses correctly.

**Stage 2. The capability service shape.**
Define the oneof result: `success`, `fileNotFound`, `notFile`, `permissionDenied`, `fileBusy`, `rejected`, `error`. Everything downstream branches on these seven cases, so getting the shape right early avoids refactors.

**Stage 3. Read.**
Depends on stages 1 and 2. Implement the binary-extension rejection, the 100,000-char cap, the empty-file branch, and the retry-on-miss behavior. The retry needs a notion of in-flight edit calls, so stub that as a no-op initially and add it once Write exists.

**Stage 4. Write.**
Depends on Read, because it reads before writing. Implement the read-modify-write, the plan-mode gate, and `recordFileState` bookkeeping.

**Stage 5. StrReplace.**
Depends on Read and Write. Implement the pipeline in the exact order given. Test the whitespace matcher hard, because that is where subtle bugs live.

Tests worth writing before you ship: `"aa"` in `"aaa"` is one match; `"aa"` in `"aaa"` with `replace_all` yields `"Xa"`; LF old_string against CRLF file matches exactly; tab-indented old_string against space-indented file matches via fallback; `old_string: "  foo"` with `new_string: "foo"` throws identical-strings rather than silently de-indenting.

**Stage 6. Delete.**
Depends on the service shape only. Implement the stat-then-recheck TOCTOU guard.

**Stage 7. Glob and Grep.**
Both depend on ripgrep. Build the shared `LocalGrepExecutor` argument construction once, since both use it: Glob with `--files` and an empty pattern, Grep with `--regexp`. Get the `--iglob` versus `--glob` distinction right, and the client-side `head_limit` application.

**Stage 8. The lock.**
Add last, once tools exist, because its only purpose is to serialize them. Model it on `lne` and decide deliberately whether to scope it per-agent like Cursor or wider.

## Testing notes

Things that are easy to get wrong and hard to notice:

**Off-by-one in the read window.** `kTe` reports `readRange.endLine` as an inclusive line number but computes `endIndex` exclusively. A full read of a 10-line file must report `{startLine: 1, endLine: 10}`.

**Non-overlapping scan.** Almost every naive implementation uses `i + 1` and therefore counts overlapping matches. This changes `replace_all` results.

**Whitespace matcher line correspondence.** The search must have the same line count as the region. A matcher that allows partial correspondence will accept edits Cursor rejects.

**Splicing into the original.** If you normalize the file content and then write the normalized version back, untouched lines lose their original bytes. Cursor recovers offsets into the original and splices only the matched region. Do the same.

**`head_limit` client-side.** Using `--max-count` gives per-file limits instead of global. The difference is visible immediately in multi-file results.

**`-C` dropped when `-A` or `-B` present.** A more intuitive merge is wrong.

**`--iglob` not `--glob`.** Case-sensitivity of glob patterns is user-visible.

**Empty file is 1 line.** Both the formatter and `kTe` depend on this.

**`exceededLimit` is a character count, not a line count, and it is checked after windowing.** 100000 chars. A 5 MB file read with `limit: 10` must succeed.

**Encoding preservation on write.** A naive UTF-8 write corrupts UTF-16 files and drops BOMs. Cursor detects the existing encoding, preserves the BOM if present, and merges line endings per-line for mixed-ending files. Probe this against real files, because the mixed-ending case is not reproducible with a uniform rewrite.

**Executable bits.** Cursor writes `0o666` before umask with no mode preservation, so a script loses `+x`. Decide deliberately whether to copy that.

**Empty write truncates.** `contents: ""` on an existing file produces a 0-byte file and reports success. It is not a refusal.

**Lock keying.** Keying on the raw argument string means `/a/b.txt` and `b.txt` do not serialize. Key on the canonical path instead.

**Glob sort direction.** Newest-first. Claude Code sorts the other way, so copying that code inverts it.

## What this document cannot tell you

Honest limits, so you do not rely on inference where evidence is absent.

**Which code path a given build reaches at runtime.** The bundle contains all version branches. Which one a particular model receives depends on runtime configuration this analysis cannot observe.

**Byte-level disk behavior below the capability service.** Tools call a service that returns a oneof. The service's implementation is outside the bundle, so encoding decisions, fsync behavior, and file-mode handling there are unknown. The `fileBusy` producer is known (raw `EBUSY` from `unlink`), but other details are not.

**The rendered UI.** `linesAdded`, `linesRemoved`, and `diffString` are proto fields. How the workbench paints them is in the extension host, not this bundle. Do not take any claim about the visual diff as verified.

**Whether specific dead code is truly unreachable.** `trackAcquisitionOrder` has zero call sites in the bundle examined. It could be reachable through a path not present in this artifact.

**Web-sourced claims about other tools.** The Claude Code comparison rests on static analysis of `claude.exe` plus published schemas. Behavior under specific runtime flag combinations is not verified.

## Companion research

Detailed evidence behind every claim here, with offsets and exhaustive branch enumeration:

| File | Contents |
|---|---|
| `2026-10-27-cursor-tools-readfile.md` | Read: seven schema branches, blob handling, retry logic |
| `2026-10-27-cursor-tools-writefile.md` | Write: both registrations, executor internals, encoding probes |
| `2026-10-27-cursor-tools-strreplace.md` | StrReplace: full pipeline pseudocode, matcher derivation |
| `2026-10-27-cursor-tools-delete-and-ls.md` | Delete and ListDir |
| `2026-10-27-cursor-tools-glob.md` | Glob: argv reconstruction, empirical ignore verification |
| `2026-10-27-cursor-tools-grep.md` | Grep: arg construction, renderer, NUL wire format |
| `2026-10-27-cursor-tools-locking-and-concurrency.md` | Lock manager scope and instantiation |
| `2026-10-27-cursor-tools-shared-infrastructure.md` | Formatter, constants, lock class |
| `2026-10-27-cursor-vs-claude-code-file-tools.md` | Claude Code binary analysis, full comparison |
| `2026-10-27-cursor-tools-web-corroboration.md` | Public docs, forums, leaked prompts, where they conflict |

Roughly 686 KB total.

Two extracted source corpora are also on disk, both greppable and offset-annotated.

**Cursor tool slices**, from the minified agent bundle:

```
cabinet\_scratch\cursor-tools\tools\     READ.js WRITE.js STR_REPLACE.js DELETE.js GLOB.js GREP.js LS.js APPLY_PATCH.js
cabinet\_scratch\cursor-tools\shared\    formatter, constants, lock class regions
```

Extraction scripts live alongside them, including `range.js` for slicing any byte range and `toolmap.js` for enumerating all 53 tool registration sites.

**Claude Code corpus**, 15 files and 435 KB, from `claude.exe`:

```
cabinet\_scratch\claude-code\tools\
```

| File | Contents |
|---|---|
| `read.js` | Read registration at 257776684, 34 KB of call graph |
| `write.js` | Write registration at 256077364, including `i1o` |
| `edit.js` | Edit registration at 259260509, matching pipeline |
| `glob.js` | Glob registration at 256106194 |
| `grep.js` | Grep registration at 256112432, 19 KB |
| `notebook-edit.js` | NotebookEdit registration at 259275930 |
| `read-file-state.js` | The `Qs` LRU store, dump/load/merge |
| `freshness-checks.js` | `xE`, `QU`, `MNe`, `bA`, `O4t` |
| `line-number-format.js` | `IUe`, `BZr`, `npr`, including the tab-aware separator |
| `limits.js` | 41 constants with offsets |
| `error-strings.js` / `.json` | 126 strings with offsets |
| `input-schemas.md` | Parameter tables from `sdk-tools.d.ts` |
| `sdk-tools.d.ts` | Anthropic's published schema file |
| `_index.json` | Per-file offsets, anchors, and verification status |

One caveat about that corpus: the regions are targeted call graphs, not contiguous module slices. Bun places a tool's declaration roughly 50 MB from its imports, so each file contains the declaration chain plus reachable call graph rather than one contiguous span. When grepping it, follow named functions rather than assuming adjacency.

## Reproduction recipe

To extract these yourself from any Cursor build:

```js
const fs = require('fs');
const src = fs.readFileSync(BUNDLE, 'utf8');

// The bundle is one line, so line-oriented tools do not work.
// Find a tool registration site:
const re = /AG\("([A-Z0-9_]+)",\{name:/g;
let m;
while ((m = re.exec(src))) console.log(m.index, m[1]);

// Then slice a readable window around an offset.
const i = 4338579;
console.log(src.slice(i - 2000, i + 14000));
```

Pipe slices through prettier for readability. Where prettier fails on a partial expression, the raw text is still readable and remains authoritative.

For Claude Code, the same technique works on `claude.exe` with one change: decode as **latin1**, not utf8, or byte offsets shift and the text corrupts.

## Errata and resolved questions

Five inconsistencies were raised during review. Four are now resolved in the body and are recorded here so you can see what changed. One remains a genuine open question, with the evidence for it stated.

**Resolved: constant count.** `### Constants` said "two other thresholds" above a three-row table. The sentence now says two thresholds plus a cache limit, which is what the table always showed. The PDF cache is a conversion cache, not a streaming threshold, so the distinction is now explicit.

**Resolved: rule count.** The line-ending rules were introduced as "three rules" but numbered to four, the fourth being the default case. Now introduced as four rules where the first three transform and the fourth is the default, which matches what the body shows.

**Resolved: confidence labels.** `## Sources and confidence` claimed each claim carried an inline label, which was not true of most of the body. That section now states plainly that labels appear where the distinction matters most, that unlabeled claims default to bundle-sourced, and that the companion files carry per-claim provenance. The claim is now accurate rather than aspirational.

**Resolved: the `isEmpty` contradiction.** This one looked like a real contradiction and is not. An empty read short-circuits before the line counter runs:

```js
if (content === "" && !executorAppliedRange) {
  return createSuccessResult("", 0, fileSize, resolvedPath, void 0, includeLineNumbers);
}
```

`totalLines` is passed as a literal `0` on that path, so `isEmpty` is reachable and only reachable this way. The line counter, which would say 1, never sees it. Both statements in the document were correct. The `### Line number formatting` section now spells out the short-circuit and warns about a second, CR-aware line counter elsewhere in the codebase that returns 0 for empty input. Do not confuse the two.

**Resolved: lock ordering.** Stage 3 needs a lock that Stage 8 builds. `## Build order` now includes a no-op lock stub for use from Stage 3 onward, with the reasoning that keeping the call site from the beginning is what prevents a tool from silently missing its lock later.

**Open: whether `executorAppliedRange` and the tool-level path can disagree.** The read executor may apply a range itself, in which case `executorAppliedRange` is true and the tool trusts `executorTotalLines` rather than recounting. Two providers of the same information exist. If they ever disagree, the tool's behavior depends on which one is authoritative, and static reading cannot tell you whether they can diverge in practice. If you are reimplementing, use one source of truth for the range and do not reproduce this split.

*2026-10-27 - deepseek-v41-flash-1017*

*Enrichment pass: orientation, glossary, reading guide, per-tool summaries added 2026-10-27. Errata review and name recovery followed, same day.*

*Example pass: test-vector tables and worked examples added to all seven tool sections 2026-10-27.*

## Example-pass errata

Two things surfaced while deriving test vectors from the documented behavior. Neither was changed in place, because this pass is additive.

**1. ListDir appends the path separator to directories only, not to files.**

The `## ListDir` section says: "a path separator appended to **every** entry including files, which is unusual and easy to get wrong", and again "renders as `/home/user/project/` then `  - README.md/` then `  - src/`".

That is wrong for this build. The renderer's child lines come from two separate templates:

```js
result2=`${childIndent}- ${child.name}${pathSep}\n`;                       // collapsed dir, count line follows
result2+=`${childIndent}- ${child.name}${pathSep}...\n`;                   // collapsed dir, no counts
result2=`${indent}- ${posix.basename(normalizedPath)}${pathSep}\n`;        // expanded dir
```

and the plain file line, from the same render loop, is:

```js
result2+=`${childIndent}- ${file2.name}\n`;
```

The independent check is the byte count. The documented main tree renders `"C:\\Users\\dev\\proj\\\n  - docs\\\n    - guide.md\n  - package.json\n  - README.md\n  - src\\\n    - app.tsx\n    - index.ts\n    - utils\\\n      - math.ts\n"` at length 142 with the measured per-line lengths 19 + 10 + 15 + 17 + 14 + 9 + 14 + 15 + 13 + 16. Line 3, `    - guide.md`, measures 15 characters: 4 indent, 2 bullet, 8 for `guide.md`, 1 newline. A trailing separator would make it 16 and break the total. So files carry no separator, and the `README.md/` in the prose is wrong.

The rest of the sentence is right and worth keeping: directories are distinguishable only by the trailing separator, there is no `d` marker, and the sort merges files and directories into one `localeCompare` list rather than grouping directories first. The worked examples use the file-without-separator form.

**2. The Grep pagination trailer's exact wrapper is not fully pinned by the bundle notes.**

`### Output format` gives the trailer's component strings but the assembled template is quoted only in fragments (`\n[${...}Showing ${...} of ${...}. ${r}]`). The worked example marks its `[Showing 2 of 5 matching lines. ...]` line `(derived)` for that reason, while the body of the same example is documented fact. Two parts are solid: the unit words and their pluralization, and `Use offset=${offset+shown} to see more, or narrow the search.`. What is not solid is whether the `Results truncated. ` prefix is present for a plain `head_limit` cut with no character truncation and no cut-before-end; the condition reads `e.charTruncated||t.cutBeforeEnd`, so for an ordinary head-limit cut it should be absent, and the example assumes that. Confirm against a live call before freezing a byte-exact test.

**Nothing else was found.** The remaining trap list in the request was checked against the research and the existing document, and each item is stated correctly as written: the six-wide pipe format with no space after the pipe and an empty string counting as one line; the 100,000-character check running after windowing; negative offsets reading `abs(offset)` lines; WriteFile preserving line endings including the mixed-ending merge; empty contents truncating successfully; StrReplace advancing by `old_string.length` so `aa` in `aaa` is one match; the whitespace fallback being line-trimmed and one-to-one; identical-after-normalization being rejected; Delete's byte-count success message and directory refusal; Glob's `--iglob` case-insensitivity and newest-first sort; Grep's global `head_limit`, `-C` being dropped, and `head_limit: 0` meaning unlimited.

## Pseudocode-pass errata

Findings from writing the pseudocode, the failure matrix, and the checklist. Nothing above was changed; each item records a place where the document is incomplete or where two of its claims do not line up.

**1. RESOLVED: the ask-mode refusal string.** `### Constants` and three tables originally rendered it as `cannot run non-read-only tools`, with a hyphen. Both binaries have a space:

```js
ASK_MODE_MODEL_ERROR="You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required."
```

Verified against `daemon.cjs` at 8035111 and the shipping minified bundle at 2436200. Every instance in the document now uses the space form. This mattered because it is exactly the kind of one-character difference that fails a byte-exact test while looking correct to a reader.

**2. The `ReadFile` `notFile` effect is produced by the `invalidFile` case, and the matrix says so in a note rather than in the case column.**

The document lists `notFile` among the seven cases and then describes Read's directory failure as arriving through `invalidFile`, rendered as `Path is a directory, not a file: <path>`. Both statements are about different layers. The service returns `invalidFile`; the tool throws a message carrying the reason; the record that comes back has no `notFile` case. The matrix labels the row `notFile` because that is the effect the model sees, and the `Notes` column says where it came from, but a reimplementation that emits a literal `notFile` case from Read would be adding a case Cursor does not produce. Use the case column as the model-visible classification and the notes as the wire truth.

**3. `WriteFile`'s plan-mode refusal has two strings and the document only quotes one prominently.**

`### Algorithm` step 4 quotes `Cannot edit non markdown files in plan mode`, and `### Failure modes` lists the same terse string. The model actually receives the long form naming the `SwitchMode` tool, because the renderer special-cases that exact reason and substitutes it. The short string is the throw; the long string is the message. Both belong in a byte-exact implementation, and the matrix records them as one row with the note that the renderer substitutes.

**4. `Delete`'s empty-path message is not the flat string the failure table implies.**

`### Failure modes` lists the path-empty case only indirectly. The executor returns an `error` case whose text is `path is empty or only whitespace`, and because it travels the same throw-then-reserialize path as every other failure, the model sees it wrapped: `Error deleting file : Error deleting file ${path}: path is empty or only whitespace`, with an empty first slot. The matrix records the wrapped form.

**5. `Glob`'s unexpected-shape string names a tool that is not the one being called.**

The render fallback reads `glob_file_search didn't return the result` on every branch, including the six where the tool is named `Glob`. The document notes that `glob_file_search` exists only as the `dsv3-1018` name, which makes this string look like dead code. It is reachable and it is model-visible. Worth reproducing deliberately or fixing deliberately, not by accident.

**6. One ordering the document does not settle: the worktree-escape check versus the permission gate.**

`WriteFile`'s `### Algorithm` numbers nine steps and lists neither the worktree-escape check nor the malformed-URL check among them; both appear only in the `### Failure modes` table, which carries no ordering information. The research shows the escape test running before the permission gate, so the pseudocode puts it there, but the specification itself never establishes that order. A reimplementation that gates permission first produces the same set of failures in the same cases and a different one when both conditions hold. Confirm against the executor if that combination matters to you.

**7. `createSuccessResult` is named in the glossary table but never located in the read path.**

`### Mangled identifiers` maps `bTe` to `createSuccessResult`, and `## Build order` warns that the name is generic and appears in several unrelated forms. Both are true. What neither section says is which of the several forms the read path actually calls, so a reader looking for "the" `createSuccessResult` has no way to tell whether they found the right one. The pseudocode deliberately avoids the name and spells the result construction out inline, which also makes the `isEmpty` and `exceededLimit` conditions visible at the point they are set.

*Pseudocode, failure-matrix, and conformance-checklist pass 2026-10-27.*

## Conformance checklist

One claim per line, each checkable by reading your own implementation. Grouped by tool, twenty items each.

### ReadFile

- [ ] `path` is required and described as `The absolute path of the file to read`
- [ ] `limit: 0` is rejected with `Limit must be >= 1.`, but `offset: 0` is accepted and silently rewritten to `1`
- [ ] The `haiku` branch takes `line_range` as an inclusive 1-indexed pair and converts it to `offset` plus a line count of `end - start + 1`
- [ ] A binary extension is rejected before any file access, with the extension lowercased and `unknown` as the fallback
- [ ] `.pdf`, `.png`, `.jpg`, `.gif`, `.webp`, and `.ipynb` are absent from the binary rejection list
- [ ] The per-file lock is acquired on the raw argument string before path normalization, and held across the retry
- [ ] A miss is retried at most once, and only after waiting up to 120000 ms for earlier mutating calls
- [ ] The retry releases the lock before waiting and re-acquires it after, and is skipped when the call is absent from the step's call list
- [ ] An empty file with no applied range returns success with `totalLines` set to a literal 0, not a computed count
- [ ] Emptiness is `totalLines === 0 AND content === ""`, never derived from the line counter
- [ ] The line counter returns newlines plus one, so `""` is 1 line and a trailing newline adds a blank final line
- [ ] The read window is computed only when `offset` or `limit` is present, and no window is returned otherwise
- [ ] A negative offset counts from the end, `-1` is the last line, and a negative offset with no limit reads `abs(offset)` lines
- [ ] Clamping happens before the bounds check, so a huge negative offset saturates to 0 instead of throwing
- [ ] `readRange.endLine` is reported as an inclusive line number even though `endIndex` is exclusive
- [ ] The 100000-character cap is a pure string-length test applied after windowing, so a small window into a huge file succeeds
- [ ] Exceeding the cap yields a success record with `exceededLimit` true and empty content, rendered with the error flag set
- [ ] The default format is six-wide right-aligned, then a pipe, then the text with no space after the pipe
- [ ] A full-file read emits no omitted-line notices, and each notice pluralizes as `1 line not shown` or `N lines not shown`
- [ ] `File not found` and `Permission denied` both carry no path and no punctuation

### WriteFile

- [ ] The content field is named `contents`, not `file_text` and not `content`
- [ ] `contents: ""` passes schema validation because the field has no minimum length
- [ ] The lock is acquired during argument streaming, as soon as the path field completes
- [ ] A stream ending without the path field fails with `Failed to acquire file lock - path may not be parsed`
- [ ] The tool performs its own read and never consults a model-facing read record
- [ ] There is no read-before-write gate and no error string for one
- [ ] A not-found result from the internal read is the new-file path, not an error
- [ ] The diff uses `added` when there was no old content and `modified` otherwise
- [ ] The plan-mode gate fires after the read and after the diff, so a refused write still costs a full read
- [ ] The plan-mode allowlist is exactly `.md`, `.markdown`, `.mdown`, `.mkd`, `.mkdn`, `.mdx`, plus canvas paths
- [ ] A malformed `file://` URL is rejected while a well-formed one is normalized and written
- [ ] The parent directory is created recursively before the write
- [ ] The stale fingerprint is re-verified immediately before the truncate, not before `mkdir`
- [ ] The open flags include no-follow and close-on-exec, and a symlinked leaf is refused rather than followed
- [ ] The file mode is `0o666` before umask with no executable-bit preservation
- [ ] The write is truncate-in-place followed by `fsync`, with no temp file and no rename
- [ ] The success message is exactly `Wrote contents to <path>`, using the raw argument after the worktries repair
- [ ] No diff, line count, or byte count reaches the model
- [ ] Format detection samples only the first 8192 bytes, and the binary extension shortcut reaches a verdict without reading
- [ ] Incoming content that already contains `\r\n` is left untouched, a mixed-ending file gets a per-line merge, and a CRLF file has bare LFs rewritten

### StrReplace

- [ ] Parameters are exactly `path`, `old_string`, `new_string`, `replace_all`, and no `explanation`, `line_range`, `old_str`, `new_str`, or `target_file` exists
- [ ] `replace_all` has no schema default on the default branch, so it arrives as `undefined` and is normalized with a triple-equals test against `true`
- [ ] `replace_all` carries an actual `false` default only on the `dsv3-1018` branch
- [ ] Both model strings and the file content have `\r\n` normalized to `\n` before matching
- [ ] An empty `old_string` throws before any search runs
- [ ] Normalized old equal to normalized new throws, so a de-indent attempt fails while a re-indent with differing forms succeeds
- [ ] The exact scan advances by the needle length, not by one, so `aa` in `aaa` is exactly one match
- [ ] The whitespace fallback runs only when the exact scan found nothing, and only when enabled on the call path
- [ ] The fallback trims per-line whitespace on both sides and does nothing else: no internal-run collapse, no Unicode normalization, no quote folding
- [ ] The fallback requires a strict one-to-one line correspondence and offers no ellipsis elision and no block-anchor strategy
- [ ] Zero matches report the relaxed-whitespace suffix only when the fallback actually ran
- [ ] The zero-match throw carries the short fixed form to the client and the long form with the fuzzy hint to the model
- [ ] The fuzzy suggestion is advisory only and never changes whether the edit applies
- [ ] Multiple matches with `replace_all` false throw, and the message inserts ` after relaxing whitespace` before the period when the fallback found them
- [ ] The overlap guard fires only when `replace_all` is false, the match came from the exact scan, and the needle starts anywhere after the first match start plus one
- [ ] The splice recovers offsets into the original content, so untouched lines keep their exact original bytes
- [ ] The bulk splice runs only when the replacement is non-empty and there are two or more matches, walking offsets in descending order
- [ ] An empty `new_string` eats surrounding blank whitespace so a whole-line deletion removes its newline
- [ ] `replace_all` on `aa` in `aaa` with `X` produces `Xa` and reports one replacement
- [ ] Success text is one flat sentence with no diff, no line numbers, and no count unless `replace_all` was set

### Delete

- [ ] The default branch takes a single `path` parameter described as the absolute path of the file to delete
- [ ] The path is selected with an `in` test rather than a fallback chain, so whichever key the schema declared wins
- [ ] The `gpt5-codex` and `codex-cloud` branches get a one-line description instead of the three-bullet form
- [ ] The lock is acquired on the raw argument string before any stat or resolution
- [ ] A whitespace-only path is refused before any I/O
- [ ] The resolver strips `file://`, expands a leading `~`, and resolves relative paths against the workspace
- [ ] The write-permission gate runs with interactive approval hard-wired to no decision
- [ ] Only the parent directory is realpathed, so a symlink at the leaf is unlinked as a link
- [ ] A broken symlink is undeletable because `stat` returns `ENOENT` before `unlink` is reached
- [ ] `stat` follows symlinks, and the reported type is only ever `directory` or `other`
- [ ] Previous content is read only when the file is at or under 50 MiB, and a failed read is swallowed rather than aborting the delete
- [ ] Captured previous content has CRLF normalized to LF
- [ ] The file is re-`stat`ed and both `dev` and `ino` compared before unlinking
- [ ] A failed re-`stat` is treated as a mismatch, reporting `File changed while deleting; nothing was removed`
- [ ] The delete is `unlink`, with no trash, no recycle bin, and no staged deletion
- [ ] `EACCES` maps to permission-denied without setting the read-only flag, and `EBUSY` maps to the busy case
- [ ] `EPERM` is not special-cased and falls through to a generic error
- [ ] The success message includes the byte count in parentheses, with no singular form
- [ ] `recordFileState` stores undefined content as the this-file-is-gone marker, and only when no earlier entry exists
- [ ] Every failure path presents to the model as an error even though the render function itself returns a false flag

### Glob

- [ ] Parameters are exactly `glob_pattern` and `target_directory`, with no `pattern`, `case_sensitive`, or `max_results`
- [ ] `dsv3-1018` describes the target as `Cursor workspace roots` in the plural plus a trailing newline, while every other branch uses the singular
- [ ] The tool is named `glob_file_search` only on `dsv3-1018` and `Glob` everywhere else
- [ ] The description is byte-identical across all seven branches
- [ ] No glob library is used: the search is ripgrep in listing mode with an empty pattern
- [ ] Backslashes in the pattern are rewritten to forward slashes, one leading and one trailing slash are removed
- [ ] The recursive prefix is added exactly once when absent, and a pattern normalizing to just that prefix is rejected
- [ ] A whitespace-only pattern is not rejected and matches nothing instead
- [ ] An absolute pattern becomes a recursive-prefixed suffix match rather than an absolute match
- [ ] The glob flag is the case-insensitive form, so pattern matching ignores case
- [ ] The `.git` exclusion is passed as its own negated glob pair rather than as an ignore rule
- [ ] Hidden files are included and symlinks are followed on every call
- [ ] No sort flag is emitted and sorting happens in JavaScript afterward
- [ ] `.gitignore` is respected and the no-ignore flag is never emitted
- [ ] Cursor ignore rules are applied through dedicated pairs plus a post-filter, not through ripgrep's ignore machinery
- [ ] Results sort newest first by mtime under a 2000 ms stat budget, ties break by path segments ascending, missing mtime sorts last
- [ ] At most 2000 files are returned and at most 10000 stdout lines are parsed, with an 8 MB stdout cap and a 25 s wall clock
- [ ] The header omits the total count when ripgrep was truncated, and pluralizes as `file` for 1 and `files` otherwise including 0
- [ ] The leading dot-slash is stripped only when the display path is exactly a single dot, so a Windows dot-backslash survives
- [ ] The mid-list cut uses `at least` only when ripgrep was truncated, and the trailing banner fires only when no cut happened

### Grep

- [ ] The hyphenated keys `-A`, `-B`, `-C`, and `-i` are literal JSON property names and are not normalized
- [ ] No `target_directory`, `-n`, `case_insensitive`, or `--no-ignore` parameter exists
- [ ] `output_mode` defaults to content mode, and the case and multiline flags default to false
- [ ] The tool is named `grep` on `dsv3-1018`, `rg` on the two Codex branches, and `Grep` otherwise, and `grep_search` is never a name
- [ ] Only the exact empty pattern is rejected, so a single space passes through
- [ ] A negative context value is rejected on the raw pre-clamp value rather than silently clamped
- [ ] The broad-glob guard tests a trimmed value against exactly the two broad patterns and is gated behind a feature flag
- [ ] `head_limit` is ceiled and `offset` is floored
- [ ] A `head_limit` of 0 means unlimited, implemented as a falsy coercion to undefined
- [ ] There is no default `head_limit`
- [ ] `head_limit` is applied client-side after parsing and never becomes a max-count flag
- [ ] `offset` and `head_limit` count globally across all files, not per file
- [ ] The glob filter maps to the case-insensitive glob flag rather than the case-sensitive one
- [ ] Symmetric context is expanded into both before and after only when neither one-sided flag is present, and dropped entirely when one is
- [ ] The case-insensitive mode is implemented by omitting the case-sensitivity flag rather than adding an ignore-case flag
- [ ] Context lines are buffered and flushed only when the match they precede is emitted, and a file with every match skipped produces no entry
- [ ] Content mode caps at 2000 entries or 4000 with context, while files and count modes cap at 2000 files
- [ ] Files sort newest first with lines inside a file in ascending order, and lines from different files are never interleaved
- [ ] A match line is two spaces, the line number, a colon, then content, while a context line uses a hyphen instead of the colon
- [ ] Invalid regexes are rewritten into one line with the pattern interpolated raw, and the escape hint appears only for three specific reasons

### ListDir

- [ ] Parameters are exactly `target_directory` and an optional `ignore_globs`, with no `path`, `depth`, `recursive`, `include_hidden`, or `max_results`
- [ ] The `ignore_globs` description contains literal tab characters before the example bullets
- [ ] The tool is named `list_dir` only on `dsv3-1018` and `LS` on every other branch
- [ ] Registration is conditional on a flag that defaults to false
- [ ] The description's dotfile claim is false and dotfiles are in fact listed
- [ ] The two description variants differ only in the sentence about whether the path must be absolute
- [ ] The description contains no typographic apostrophes
- [ ] Relative paths are accepted despite the default branch's absolute-path claim
- [ ] The walk passes the files, hidden, no-require-git, no-config, never-color, and case-sensitive flags
- [ ] The walk passes no follow flag, so symlinks are not descended
- [ ] The walk passes no no-ignore-vcs flag, so gitignore rules apply even outside a repository
- [ ] A stat failure with `ENOENT` renders as `Path does not exist: <resolved>`, except for a terminal log path, which renders `No active terminals.`
- [ ] A non-directory target renders as `Path is not a directory: <resolved>`
- [ ] The tool has exactly four result cases and no permission-denied case, so a permission failure arrives as raw OS errno text
- [ ] The executor takes no per-file lock
- [ ] The ignore globs are passed through verbatim with no recursive prefix prepended
- [ ] Empty directories never appear because the tree is built from emitted file paths only
- [ ] The filename character budget is 2500, seeded with the root path length, and the rendered string budget is 10000 measured afterward
- [ ] Directories carry a trailing separator and files do not, and the separator is chosen once from whether the root path contains a backslash
- [ ] A timeout returns the partial tree with a note and reaches the model as a non-error

*Companion passes on this document: enrichment 2026-10-27, name recovery 2026-10-27, example vectors 2026-10-27, pseudocode and conformance structure 2026-10-27.*

*2026-10-27 - deepseek-v41-flash-1017*
