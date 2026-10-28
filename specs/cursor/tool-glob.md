# Cursor's `Glob` tool: an implementation report

This report specifies one tool from Cursor's shipped agent bundle, in enough detail that you can build a different but compatible one. It is the Glob volume of a six-volume set; the other five cover Read, Write, StrReplace, Delete, and Grep. Everything here is derived from the installed Cursor 3.27.12 artifacts on this machine, from the unminified daemon twin, or from probes run against the shipped `rg.exe`. Where the parent spec at `runvil-design/specs/cursor/agent-file-tools.md` and the shipped build disagree, the shipped build wins and the disagreement is recorded.

Two offsets matter and both are given throughout. The parent spec was written against a 9,674,628 byte bundle; the installed bundle is 9,814,287 bytes. `GLOB` registration moved from `4277336` to `4292676`. The shared ripgrep executor moved further, from about `9017686` to `9158000`.

## The short version

- **Glob is not a glob library. It is ripgrep in `--files` mode with an empty pattern.** There is no `fast-glob`, no `picomatch`, no `minimatch` anywhere on this path. Matching is done by ripgrep's globset engine, ignoring is done by ripgrep's ignore machinery, and the walk is ripgrep's parallel directory traversal. Every surprising behavior below is a consequence of that one fact.
- **A positive glob defeats `.gitignore` at file level, which is the opposite of what the parent spec says.** `-g`/`--iglob` "always overrides any other ignore logic" per ripgrep's own `--help`. So `glob_pattern: "**/*.log"` returns a `*.log` file that `.gitignore` excludes. What saves the common case is directory pruning: an ignored *directory* is never descended into unless the glob matches the directory path itself, so `**/*.js` still returns nothing from a gitignored `vendor/`. The parent spec's claim that "`.gitignore` wins over a positive glob" is false for file-level rules and true only for directory-level ones.
- **`.cursorignore` does not have that hole.** Cursor added a custom ripgrep flag, `--cursor-ignore`, documented in the shipped `rg --help` as applying "with higher precedence than `-g`/`--glob` overrides. These rules can exclude files even when a glob would otherwise include them." Both ignore sources are therefore in force, but by two different mechanisms with different precedence.
- **The pattern is rewritten before ripgrep ever sees it.** Backslashes become forward slashes, one leading slash is stripped, one trailing slash is stripped, and `**/` is prepended if absent. Absolute patterns are therefore destroyed: `/home/dev/proj/*.ts` becomes `**/home/dev/proj/*.ts`, a suffix match that cannot succeed. `target_directory` is the only way to search an absolute location.
- **The sort is newest-mtime-first, and it is computed in JavaScript after ripgrep returns, under a 2000 ms stat budget.** If statting the whole result set takes longer than two seconds, the sort is abandoned silently, ripgrep's walk order is kept, and the rendered order becomes arbitrary. The description's promise of "sorted by modification time" is conditional.
- **The empty-pattern guard is `"**/" === normalized`, which catches `""` and nothing else.** A pattern of `" "` normalizes to `"**/ "` and is passed through as a literal, matching nothing. A pattern of `"/"` normalizes to `"**/"` and is rejected as if empty.
- **The cap that binds is the one nobody documents.** There are three caps: 2000 entries returned, 10000 stdout lines parsed, and 5000 rendered characters. On a normal repository the character budget binds first, at roughly 160 files, so a model that finds 400 matches sees about 160. The mid-list cut does announce itself with `... N more files ... (Do a more specific search if needed)`, so the truncation is visible. What is *not* visible is the total in the header, which can say `(total 5000 files)` while 2000 were retained and 160 shown.
- **The render fallback string is `glob_file_search didn't return the result` on every branch,** including the six where the tool is named `Glob`. `glob_file_search` is the `dsv3-1018` name only. This is reachable and model visible, and it is the kind of thing you fix deliberately or copy deliberately, never by accident.
- **Case-insensitive matching is unconditional and comes from `--iglob`.** The argv also contains `--case-sensitive`, which reads like a contradiction and is not: `--case-sensitive` governs the *content* regex, `--iglob` governs the *pattern*. So on a case-sensitive filesystem, `*.JS` and `*.js` both match, which diverges from every shell glob a model has seen.
- **`.git` is excluded by a literal `--iglob !**/.git` pair, not by an ignore rule.** Remove that pair and the entire `.git` directory contents come back, which the probe confirmed. There is no user-facing exclude parameter to add anything similar.
- **`--hidden` and `--follow` are unconditional.** Dotfiles are always included. Symlinks are always followed, so a link pointing outside the workspace is traversed unless a later permission filter catches it. There is a second ripgrep builder, `ripwalk`, which disagrees with this one on three defaults: it passes `--no-require-git`, it does not pass `--follow` unless asked, and its `caseSensitive` defaults to `false`. Glob never reaches it. Reading the wrong builder produces the opposite answer on symlinks, on `.gitignore` inside a non-git directory, and on case sensitivity.
- **Output is a header plus one `- path` line per file, as a single text block.** Never JSON. The header count is suppressed when ripgrep was truncated, because the count would then be a lie.
- **Paths are workspace-relative, and on Windows they keep a `.\` prefix.** The stripping test is `startsWith("./")`, which a backslash does not satisfy. So the platform most likely to be reimplemented keeps the prefix that the POSIX path drops.
- **`pi_find`, the parallel "pi-compatible" surface in the same bundle, is a completely different engine** - a JavaScript recursive `readdir` walk filtered by `minimatch`, with `pattern`/`path`/`limit` parameters and a 1000 default limit. It is a better structural model for a fresh implementation and a worse semantic one.
- **Nothing pages.** There is no offset, no cursor, no resume. A truncated result is truncated, and the only recovery is to make the pattern more specific.

## Decisions you have to make

### Decision 1: Do you keep ripgrep, or do you write a matcher?

This is the decision that determines the shape of everything else.

**Option A, keep ripgrep.** Spawn `rg --files` with the same argv. You inherit the globset matcher, the ignore machinery, the parallel walk, the `.git` handling, and the dotfile and symlink behavior. You also inherit the precedence bug, the `-l` versus `--files` distinction, the 8 MB stdout cap, and the fact that your tool's semantics are defined by a third-party binary whose version you now pin. Cost: a subprocess per call, a binary to ship on four platforms, and semantics you cannot modify without forking ripgrep. Buys: two days instead of two months, and a matcher that already handles brace expansion, character classes, and the full gitignore rule grammar.

**Option B, write a matcher over your VFS.** Vinnie's `Access::glob` already exists, already validates, gates, claims, fires an event, and delegates to a backend that owns its own matching. Cost: you must decide what the grammar is, and you will get it wrong in the corners that models actually use - `{a,b}` braces, `[a-z]` classes, and `**` in the middle of a segment. Buys: determinism, no subprocess, no binary, and a glob whose semantics you can specify in a sentence.

**Option C, hybrid.** Use the VFS walk for enumeration and a small in-process matcher for filtering, exactly the way `pi_find` does.

Recommendation: Option B for Runvil, because the VFS is in-process and virtual, which means a subprocess cannot see it at all - ripgrep only works against a real filesystem. Where you need to match Cursor's *visible behavior* rather than its engine, note that a correct globset-compatible matcher over `**` and `*` covers over ninety percent of real calls, and the remaining cases (`{}` and `[]`) are cheap to add. Confidence: high, because the subprocess cannot reach the VFS, so the constraint decides itself.

There is a real sub-decision hiding here that you should make separately: **what does your pattern grammar accept?** Runvil's `validate_glob_pattern` currently refuses backslashes, control characters, more than two consecutive `*`, a `**` that does not occupy a whole segment, and anything over 1024 bytes. Cursor accepts all of those except the length limit, because it hands them to globset, which has its own opinions. If you plan to serve Cursor-trained models, refusing `**b` while Cursor accepts it is a divergence a model will hit. Confidence: medium, because the frequency of `**b` in real calls is unknown.

### Decision 2: Do you honor ignore files at all?

Runvil's VFS has no ignore concept whatsoever. Cursor's Glob honors `.gitignore` and `.cursorignore`. That gap changes what the model can see.

**Option A, no ignore files.** Every file in the tree is visible to Glob. This is what the VFS does today. Cost: on a real repository the model drowns. A `node_modules` that is gitignored becomes tens of thousands of candidate matches, which will blow through every cap and still return noise. Buys: total determinism, no hidden state, and a tool whose output depends only on the tree and the pattern.

**Option B, honor a project ignore file.** Read `.gitignore` or a Runvil-specific equivalent and apply it during the walk. Cost: the tool's output now depends on files the model cannot see, which is the single biggest trap in Cursor's design. Buys: the same practical behavior Cursor-trained models expect, and a usable result set on any real repository.

**Option C, honor ignore files but report them.** Same as B, except the rendered result mentions that N paths were excluded by ignore rules, so the model is never silently lied to.

Recommendation: Option C, and make the ignore source a property of the *Pack*, not of the tool, so every tool agrees. The reason C is worth the extra line of output is that Cursor's silence on this point is a documented source of model confusion - a model that knows a file exists and gets zero matches concludes the file is gone. Confidence: high, because the failure mode of B is already known from Cursor's own field reports.

If you pick B or C, you must also decide **precedence**, and this is where copying Cursor's argv without understanding it will bite you. In ripgrep, `--iglob` beats `.gitignore` for files it names, and `--cursor-ignore` beats `--iglob`. So a positive glob resurrects a gitignored file. Decide deliberately whether your ignore rules win over your pattern or the other way around. I would make ignore rules always win, because a user who writes `.gitignore` means it, and report the exclusion rather than silently dropping. Confidence: medium, because the opposite choice matches ripgrep and therefore matches what models have seen.

### Decision 3: What do you sort by, and what happens when you cannot?

Cursor sorts newest-mtime-first, in JavaScript, after the search, under a 2000 ms budget, and falls back to walk order when the budget expires. That fallback is a determinism hole.

**Option A, copy Cursor exactly.** Sort by mtime descending with a 2000 ms budget, ties by path segments, missing mtime last, silent fallback. Cost: on a large result set or a slow disk, two identical runs produce different orders, which breaks replay. Buys: byte-identical output to Cursor in the common case.

**Option B, sort with no budget.** Stat every match, sort, done. Cost: unbounded time on a pathological tree. Buys: deterministic order every time.

**Option C, sort by path only.** Deterministic, cheap, needs no stat at all, and needs no mtime from your backend. Cost: diverges from Cursor's stated behavior and from what the description promises, so a model that says "find the file I just edited" no longer gets it first.

**Option D, sort by mtime when the backend supplies it, path otherwise, and say which.** Deterministic given a backend, honest about the fallback.

Recommendation: Option D. `Stat.modified` is `Option<SystemTime>`, and Runvil's own `stat.rs` says a backend fills an optional field only with a value it tracks, "so that a reported time stays deterministic and a sort by time, such as `ls -t`, stays meaningful." That is exactly the right contract. Sort by mtime when present, break ties by path, sort purely by path when the whole set lacks mtime, and never mix the two orders within one result. Confidence: high, because the alternative silently breaks the replay property you care about and the mtime data may simply not exist.

The consequence to accept: an in-memory backend will return path order, a real backend will return newest-first, and the two will not agree. That is fine as long as it is documented, and it is much better than a sort that sometimes happens. If you want cross-backend agreement, the only answer is C.

### Decision 4: What is your cap, and does it announce itself?

Cursor has three caps that interact: 2000 entries returned, 10000 stdout lines parsed, and 5000 rendered characters. The character cap binds first at roughly 160 files.

**Option A, one cap on entries.** Return at most N files, and say how many were dropped. Simple, predictable, and the units are unambiguous.

**Option B, one cap on rendered bytes.** Return as many files as fit in a byte budget, and say how many were dropped. This matches what actually matters for context cost, and it is what Cursor effectively does.

**Option C, both, like Cursor.** Cost: two caps means two distinct truncation messages and a class of bugs where the count in the header reflects one cap and the line count reflects another.

Recommendation: Option B, a single byte budget on the rendered list, plus a total count, plus an explicit "N more files" line. One cap, one message, one unit. The reason to prefer bytes over entries is that a path list's cost is its bytes, not its item count, and a directory of forty-character paths costs four times what a directory of ten-character paths costs. Confidence: medium-high, because a byte budget makes the truncation point depend on path length, which is less predictable for a caller but exactly right for a context window.

Whatever you choose, **announce it**. Cursor's mid-list cut does announce itself. Its `totalFiles` count does not always tell the truth, and that is the failure to avoid.

### Decision 5: What does the header look like when the count is not known?

Cursor suppresses the count when ripgrep was truncated, because the number would be a lie, and prints `(total N files)` otherwise. That means the header has two shapes and a caller cannot parse it uniformly.

**Option A, always print a count.** Simple to parse. Cost: the count is a lower bound under truncation, so the header lies.

**Option B, never print a count; print the returned count and a separate total line.** Uniform, honest.

**Option C, copy Cursor's suppression.** Two shapes, honest, slightly harder to parse.

Recommendation: Option B. `Returned 40 of at least 2000 matches` is one line, tells the truth in both cases, and needs no conditional. Confidence: high for a new implementation, because the only reason to prefer C is byte-compatibility with Cursor, and you have already decided not to copy exactly.

### Decision 6: Directory-only matching

Runvil's `Access::glob` treats a trailing `/` as "directories only." Cursor's Glob has no such concept and strips the trailing slash.

**Option A, adopt the trailing-slash convention in the tool.** `target_directory`-style calls can then ask for directories. Cost: a model trained on Cursor will never send one, and a model that does send `src/` will be surprised when it means "directories named src" rather than "files under src."

**Option B, strip it like Cursor and expose no directory mode.** `glob_pattern: "src/"` becomes `**/src`, which matches both files and directories named `src`.

Recommendation: Option B at the tool surface, Option A retained in the VFS. The VFS-level capability is worth having, and the `Access::glob` signature already supports it. But the tool surface should match what models do, and models send `src/` meaning "under src", not "directories named src". Confidence: medium, because the model's intent for a trailing slash is genuinely ambiguous and Cursor's choice to strip is a coin flip that happens to be the common reading.

### Decision 7: Do you keep the mtime sort at all, or replace it with a recency signal you own?

The mtime sort exists to answer "which of these did I just touch." That is a real need, and mtime is a bad signal for it: it changes under `git checkout`, it is not preserved across copies, and a build step touching a hundred files reorders the whole list.

**Option A, keep mtime.** Familiar, matches the description, needs no state.

**Option B, track recency in the harness.** The Runvil Pack knows which files its own Write and StrReplace calls touched this session. Sort by that first, then mtime, then path.

Recommendation: Option B as an addition, not a replacement, if you find models asking "the file I just created." Confidence: low, because the frequency of that pattern is unknown and it adds state to a tool that currently has none. Worth noting because it is the one place where a harness can beat Cursor, which has no cross-tool recency signal for Glob.

## The contract

### Names

| Version tag | Tool name shown to the model |
|---|---|
| `dsv3-1018` | `glob_file_search` |
| `cursor-0226` | `Glob` |
| `dsv3-1205` | `Glob` |
| `latest` | `Glob` |
| `gpt5-codex` | `Glob` |
| `codex-cloud` | `Glob` |
| `haiku` | `Glob` |

The switch has a `default` arm that throws `Unhandled version: ${e}`, so an unknown family is a hard failure rather than a fallback. The default parameter is `"latest"`.

```js
const name = version === "dsv3-1018" ? "glob_file_search" : "Glob";
```

Internal registration id is `GLOB`, at bundle offset `4292676` in this build and `4277336` in the parent spec's build.

### Parameter schema

Both variants have exactly two fields. `target_directory` optional, `glob_pattern` required. There is no `pattern`, no `path`, no `case_sensitive`, no `max_results`, no `no_ignore`, and no `include_ignored`. Searches for all of those return nothing in either schema.

`REe`, used by `latest`, `cursor-0226`, `dsv3-1205`, `gpt5-codex`, `codex-cloud`, and `haiku`, at bundle offset `4289658`:

```js
U$.Ikc({
  target_directory: U$.YjP().optional().describe("Absolute path to directory to search for files in. If not provided, defaults to Cursor workspace root."),
  glob_pattern: U$.YjP().describe('The glob pattern to match files against.\nPatterns not starting with "**/" are automatically prepended with "**/" to enable recursive searching.\n\nExamples:\n\t- "*.js" (becomes "**/*.js") - find all .js files\n\t- "**/node_modules/**" - find all node_modules directories\n\t- "**/test/**/test_*.ts" - find all test_*.ts files in any test directory')
})
```

`MEe`, used by `dsv3-1018` only, at bundle offset `4290208`:

```js
U$.Ikc({
  target_directory: U$.YjP().optional().describe("Path to directory to search for files in. If not provided, defaults to Cursor workspace roots."),
  glob_pattern: U$.YjP().describe('The glob pattern to match files against.\nPatterns not starting with "**/" are automatically prepended with "**/" to enable recursive searching.\n\nExamples:\n\t- "*.js" (becomes "**/*.js") - find all .js files\n\t- "**/node_modules/**" - find all node_modules directories\n\t- "**/test/**/test_*.ts" - find all test_*.ts files in any test directory\n')
})
```

Three differences, all cosmetic. `Absolute` is present in `REe`'s target description and absent from `MEe`'s. `root.` singular versus `roots.` plural. `MEe`'s `glob_pattern` describe text ends with a newline. The `glob_pattern` describe text is otherwise byte-identical, and it contains **literal tab characters** before each example bullet, not spaces. A reimplementation that normalizes those tabs produces a schema string that differs from Cursor's.

`U$.Ikc` is a plain `z.object`, so unknown keys are **stripped, not rejected**. The `Ikc:()=>ye` export resolves to `function ye(e,t){const r={type:"object",shape:e??{},...}` in the zod schema module, and none of the six tool schemas calls `.strict()`. The `Invalid arguments:` envelope therefore comes only from type and refine failures, never from an unrecognized key. Do not build a validator that rejects unknown keys if you want Cursor's behavior: a model that sends `pattern` instead of `glob_pattern` gets a missing-required-field error for `glob_pattern`, not an "unrecognized key" error naming the key it actually sent.

There is also a static JSON Schema projection at bundle offset `4673861`, used only by the Smart Mode classifier, not by the main agent. It carries the shorter description with the batch-calling bullet removed, and its `required: ["glob_pattern"]` confirms the optionality.

### Description text

One string, byte-identical across all seven branches, at bundle offset `4292721`. The generator ignores its argument.

```text

Tool to search for files matching a glob pattern

- Works fast with codebases of any size
- Returns matching file paths sorted by modification time
- Use this tool when you need to find files by name patterns
- You have the capability to call multiple tools in a single response. It is always better to speculatively perform multiple searches that are potentially useful as a batch.

```

Leading newline, trailing newline, both present.

Three claims in that text deserve separate judgment, because the implementation honors them unevenly.

| Claim | Honored? |
|---|---|
| "Works fast with codebases of any size" | Mostly. The walk is parallel and the caps bound the output, but the 2000 ms stat budget is precisely the point where a large codebase stops being fast, and the response is to give up on ordering rather than to give up on size. |
| "Returns matching file paths sorted by modification time" | Conditionally. Direction is newest-first, which the text does not say. Ties break by path. The sort is abandoned entirely on a stat timeout, and per-root ordering is not re-established when multiple workspace roots merge. |
| "Use this tool when you need to find files by name patterns" | True and useful. It is what distinguishes this from Grep. |

The text is derived from Claude Code's, which is worth knowing because it explains why the middle bullet appears verbatim in both: Cursor inherited the claim rather than deriving it. See `## How three harnesses do this`.

### Caps and limits

| Limit | Value | Unit | Layer that enforces it |
|---|---|---|---|
| `CLIENT_LIMIT_LINES` | 2000 | files returned to the renderer | `LocalGrepExecutor` static |
| `HARD_MAX_OUTPUT_LINES` | 10000 | stdout lines parsed | `LocalGrepExecutor` static, passed to `parseRipgrepFilesOutput` |
| stdout cap | 8388608 | bytes of ripgrep stdout | `executeRg`, local `const c=8388608` |
| wall clock | 25000 | ms per root | `MAIN_TIMEOUT_MS` |
| stat budget | 2000 | ms for the whole batch of mtime stats | `SORT_STAT_TIMEOUT_MS` in `newest-first.js` |
| render budget | 5000 | JavaScript string characters | `TOTAL_CHARACTER_BUDGET`, bundle offset `4290699` |
| pattern length | none | - | no limit in Cursor; Runvil caps at 1024 bytes |

The 5000 is `5e3` in the minified bundle and `TOTAL_CHARACTER_BUDGET` in the daemon. The comparison is `accumulated.length + file.length > budget`, where `accumulated` is the whole rendered buffer built so far, header included. That length is a JavaScript string length, so the unit is UTF-16 code units, not bytes.

**The "100 results" figure that circulates in public documentation is not Cursor's number.** It belongs to Claude Code, OpenCode, and Gemini CLI. Cursor is 2000 entries and 10000 parsed lines, with a 5000-character render budget that usually binds first.

### Result cases

The tool returns a `GlobToolResult` whose `result` oneof has three cases: `success`, `error`, and an absent case. The rendered text for each:

| Case | Model-visible text |
|---|---|
| success, zero files | `Result of search in '<path>': 0 files found\n` |
| success, files present | `Result of search in '<path>' (total N files):\n- <path>\n...` |
| success, ripgrep truncated | `Result of search in '<path>':\n- <path>\n...` (count suppressed) |
| success, mid-list cut | prior lines then `... N more files ... (Do a more specific search if needed)\n` |
| success, mid-list cut under ripgrep truncation | prior lines then `... at least N more files ... (Do a more specific search if needed)\n` |
| success, no mid-list cut but a cap hit | prior lines then `[Results truncated. N files in total. Narrow the search with a more specific target_directory or glob_pattern.]\n` |
| error | the error text, head-and-tail truncated at 5000 characters |
| neither success nor error | `glob_file_search didn't return the result` |

The `<path>` in the header is `result.path || "."`, which is the `target_directory` the caller supplied. When no target was supplied it renders as `.`.

Two exact templates, for byte-level work:

```text
Result of search in '${t}': 0 files found
```

```text
Result of search in '${t}' (total ${n} file${1===n?"":"s"}):
```

Note the pluralization helper: `file` when the count is one, `files` otherwise, **including zero**. A zero-count result never reaches the plural helper because the zero case is a separate branch.

### The `pi_find` surface

The same bundle registers a parallel family of `pi_*` tools. Glob's counterpart is `pi_find`, at registration offset `4562769`, described as:

```text
Search for files by glob pattern using pi-compatible find semantics.
```

This is a genuinely different tool and not a renamed Glob. Its schema, from the daemon at `13912670`:

```js
object({
  pattern: string2().describe("Glob pattern to match files, e.g. '*.ts', '**/*.json', or 'src/**/*.spec.ts'"),
  path: string2().optional().describe("Directory to search in (default: current directory)"),
  limit: number2().optional().describe("Maximum number of results (default: 1000)")
})
```

| Dimension | `Glob` | `pi_find` |
|---|---|---|
| Pattern parameter | `glob_pattern`, required | `pattern`, required |
| Directory parameter | `target_directory`, optional | `path`, optional |
| Result limit | fixed 2000 internal, not settable | `limit`, settable, default 1000 |
| Matching engine | ripgrep globset via subprocess | `minimatch` over relative paths, in process |
| Enumeration | ripgrep's parallel walk | recursive `readdir` with `withFileTypes`, depth first |
| Ignore files | `.gitignore` and `.cursorignore`, via ripgrep | none at all |
| Skipped entries | `.git` only, explicitly | `.git` and `node_modules`, hardcoded |
| Sort | mtime descending under a budget | none; walk order |
| Case sensitivity | insensitive, forced | minimatch default, case sensitive |
| Recursive prefix | `**/` always prepended if absent | `**/` prepended only if the pattern has a `/` and does not start with `**/` |
| Output | header plus `- path` lines | bare newline-joined paths, no header |
| Zero-match text | `Result of search in '<p>': 0 files found` | `No files found matching pattern` |
| Truncation | mid-list notice plus banner | bracketed notice appended |
| Dotfiles | included, `--hidden` unconditional | included, `{dot:true}` in minimatch |
| Symlinks | followed, `--follow` unconditional | not followed; `readdir` reports the link, not the target's contents |

The walk, from `../local-exec/dist/pi/find.js`:

```js
async function walk2(root, limit, shouldInclude) {
  const results = [];
  async function visit3(dir) {
    if (results.length >= limit) return;
    const entries2 = await readdir(dir, { withFileTypes: true });
    for (const entry of entries2) {
      if (results.length >= limit) return;
      if (entry.name === ".git" || entry.name === "node_modules") continue;
      const fullPath = join(dir, entry.name);
      if (!await shouldInclude(fullPath)) continue;
      results.push(fullPath + (entry.isDirectory() ? sep : ""));
      if (entry.isDirectory()) { await visit3(fullPath); }
    }
  }
  await visit3(root);
  return results;
}
```

Four things about that walk matter. It records **directories as well as files**, with a trailing separator, so `pi_find` matches directories where `Glob` does not. It counts the limit against the walk rather than against the matches, so `limit: 1000` means "inspect 1000 entries," and a directory-heavy tree can return zero matches after a full budget. It is recursive with no depth guard and no visited set, so a symlink loop is a stack overflow rather than a graceful error - which cannot happen only because `readdir` with `withFileTypes` and no `follow` does not descend into links. And it is strictly sequential, one directory at a time.

The `limit` notice is exactly:

```text
${limit} results limit reached. Use limit=${limit*2} for more, or refine pattern
```

joined into a bracketed block with the byte-truncation notice when that also fired:

```text
[${notices.join(". ")}]
```

The result is truncated with `truncateHead`, but the call is `truncateHead(output2, { maxLines: Number.MAX_SAFE_INTEGER })`, so the 2000-line default is overridden out of existence and only the `PI_DEFAULT_MAX_BYTES` of 51200 engages. The line cap is dead on this path. That is a small but real correction to the obvious reading of the constants: `PI_DEFAULT_MAX_LINES = 2e3` exists and is the default for `truncateHead` and `truncateTail`, and `pi_find` deliberately does not use it, because it already bounds the result with its own `limit`.

Two matching details. The recursive prefix rule differs from `Glob`'s: `pi_find` prepends `**/` only when the pattern contains a `/` and does not already start with `**/`, so a bare `*.ts` matches only at the root, where `Glob` would make it recursive. That is a real semantic difference and it will silently change results for the most common pattern shape. And matching is against the path relative to `path`, converted to POSIX separators.

**Is `pi_find` a better model for a fresh implementation?** Structurally yes, semantically no. It is better because it is in-process, uses the VFS's own enumeration, needs no subprocess or shipped binary, and has a settable limit with an honest notice. It is worse because it has no ignore support at all, no sort, a walk-order result, a limit that counts the wrong thing, and a matching engine (`minimatch`) whose semantics differ from ripgrep's in the corners. Take the structure and the honest notices; keep Cursor's matcher semantics and add the ignore layer.

## Algorithm and ordering

Steps in the order they run. Every step marked **load bearing** changes observable behavior if moved.

### Tool layer

1. **Parse and validate arguments** against the version-selected schema. Failure throws `Invalid arguments:\n<key>: <message>`.
2. Open a tracing span named `globExecute`. Cosmetic, but it is where `ctx` comes from.
3. **Normalize the pattern.** Load bearing, and it must come after validation and before everything else:
   ```
   normalizeGlob(pattern):
     1. replace every "\" with "/"
     2. if it starts with "/", strip exactly one leading "/"
     3. if it ends with "/", strip exactly one trailing "/"
     4. if it does not start with "**/", prepend "**/"
     5. return
   ```
   The daemon has this as `function normalizeGlob(glob)` at `12712199`, with identical semantics.
4. **Reject the empty case.** `if (normalized === "**/") throw 'glob_pattern is empty. Pass "*" to match every file.'` with classification `INVALID_ARGS`. Load bearing: it must be after step 3, because `""`, `"/"`, and `"//"` all reach `**/` only after normalization. Moving it before normalization misses all three.
5. Track MCP directory access if the target looks like an MCP path. Has no effect on a non-MCP run.
6. **Build the `GrepArgs` message** with `path: target_directory`, `glob: normalized`, `outputMode: "files_with_matches"`, and `pattern` left at its protobuf default. Load bearing: leaving `pattern` empty is what selects `--files` later. Passing `""` explicitly is the same thing; passing `"."` or any other value silently switches the executor to content mode.
7. Build a display-args record carrying the **raw** `target_directory` and `glob_pattern`, for the transcript. Note these are unnormalized, so the trace shows what the model sent, not what ran.
8. **Call the shared executor**, which is the same executor Grep uses.
9. **Retry once on the empty-pattern error.** If the executor returned `error` and the message contains `received an empty pattern`, re-run with `pattern: "."`, and if that succeeds adopt its result. Increments `glob.empty_pattern_fallback` with an `outcome` label either way. Load bearing: this retry changes semantics, because `pattern: "."` breaks the `--files` condition and falls through to `-l --regexp .`, which is content mode and therefore drops empty and binary files. It exists for executors that reject the empty pattern. The shipped executor does not, so it never fires in practice, but a reimplementation that adds an empty-pattern guard to the executor must keep the fallback or fail.
10. **Classify errors.** `error` case goes through `classifyRipgrepError`, which substring-matches in this order: `timed out` maps to `TIMEOUT`; `is not allowed in a regex` or the empty-pattern test maps to `INVALID_ARGS`; `regex parse error` maps to an environment error with a rewritten message; then a list of known-good user-facing substrings maps to an environment error; then the `os error` code set `{1,2,3,5,13,32,33}` maps to an environment error; anything left becomes a plain `Error`.
11. Any other case throws `Unexpected result case: ${case}`.
12. **Map the executor result to the tool result.** `mapGrepResultToGlobToolResult` walks `workspaceResults`, skips entries whose case is not `files`, and if **no** entry was `files` it returns an absent result, which renders as the `glob_file_search didn't return the result` string. If a `target_directory` was supplied, it rewrites every path through the `BEe` display transform. It ORs `clientTruncated` and `ripgrepTruncated` across roots and sums `totalFiles`.
13. **Render.** See below.

### Executor layer

14. **Resolve roots.** `workspacePath` is a string or an array; normalize to an array, defaulting to `[process.cwd()]`.
15. **Decide the search target.** If `target_directory` is absent, the target is `.` and every root is searched. Otherwise `searchTarget` resolves the path against the single workspace path, and if it is inside the root it becomes `path.relative(root, target)`; if it is outside, the absolute path is used. `isRootIndependentSearchPath` treats an absolute path, a `~` path, or a `file://` URL as root independent.
16. **Per root, four pre-checks before spawning.** Load bearing, and the order matters because each produces a different message:
    - root does not exist: `Path does not exist: ${l}`
    - target filtered by cursorignore: `Path '<p>' is filtered out by .cursorignore`
    - target is neither file nor directory: `Path '<p>' is neither a file nor a directory`
    - realpath failed: `Path '<p>' could not be verified while resolving symlinks. This can happen when access is denied or the path contains a symlink loop.`
17. **Build argv.** Load bearing order, because later globs beat earlier ones:
    ```
    --cursor-ignore <file> ...    (one pair per ignore file per root)
    --files                       (because pattern === "" and glob is set)
    --case-sensitive
    --iglob <normalized pattern>
    --iglob !**/.git
    --line-buffered               (streaming path only)
    --no-config
    --color=never
    --hidden
    --follow
    --encoding <enc>              (only when the target resolves to a file)
    --
    <relative target>
    ```
    Note what is **absent**: no `--no-ignore`, no `--regexp`, and no sort flag *on the streaming path*. The `!**/.git` pair is emitted only when the `--files` branch is active, so Grep with a content pattern never gets it.
18. **Spawn** with `cwd` at the root, a 25 s abort deadline, an 8 MB stdout cap, and a 10000 line parse cap. If the spawn fails with ENOENT, report `Path does not exist: ${cwd}` - the directory, not the binary. That is a real quirk: a missing `rg` presents as a missing directory.
19. **Check for failure.** `throwOnRipgrepFailure` throws only when stderr is non-empty *or* the exit code is 2, *and* stdout is blank, *and* the stderr is not entirely per-file access errors (`os error` 13, 5, 2, or 1) or a filesystem-loop message. So an unreadable subdirectory does not fail the whole search. Exit code 1 with blank stdout is **not** an error - it means no matches.
20. **Parse stdout.** `parseRipgrepFilesOutput(stdout, clientLimit, hardLimit)` trims, returns an empty result for empty input, counts every line to get `totalFiles`, retains the first `clientLimit` lines, and returns `clientTruncated = total > clientLimit` and `ripgrepTruncated = total >= hardLimit`. Line scanning strips a trailing `\r` before each `\n`, so CRLF output is handled. Note `ripgrepTruncated` uses `>=` while `clientTruncated` uses `>`, so exactly 10000 lines counts as truncated.
21. **Order newest first**, only on the streaming path. `orderNewestFirst` calls `sortNewestFirst`, which stats every path in parallel with `mtimeNs` bigint precision under a 2000 ms timeout, then sorts. On timeout it returns the input untouched with `statTimedOut: true`, and the caller logs `newest_first.stat_timeout_fallback` and increments a counter. Load bearing: the sort happens **before** the slice, so the 2000 kept entries are the 2000 newest, not the first 2000 found.
22. **Slice to 2000.** `keepNewestPage` does `orderNewestFirst(...).slice(0, CLIENT_LIMIT_LINES)`.
23. **Post-filter.** `filterBlockedFiles` resolves each path and drops anything the permissions service blocks, that `isCursorIgnored` rejects, that `isRepoBlocked` rejects, or that a read-boundary blocker refuses. The drop count is subtracted from `totalFiles`. Load bearing: this runs **after** the sort and the slice, so dropping 1900 of 2000 leaves only 100 rendered despite a `totalFiles` that still reflects the larger number.
24. **Flatten across roots.** Concatenate, sum `totalFiles`, OR the truncation flags. Ordering holds **per root**, not globally - no cross-root sort is re-applied.

### The second builder, and why the singular is wrong

Describing "Cursor's ripgrep invocation" in the singular is the single easiest way to get three defaults wrong. `buildArgs` is one of at least **two** builders in the daemon, and they disagree on three defaults that matter:

| Default | `LocalGrepExecutor.buildArgs` (the one Glob uses, daemon 16780893) | `ripwalk` (a different helper Glob never reaches) |
|---|---|---|
| `--no-require-git` | never passed, so `.gitignore` is inert outside a git repository | always passed, third in the fixed prefix |
| `--follow` | **always** passed, so symlinks are followed | `followSymlinks = false` by default, `--follow` only when passed |
| `--case-sensitive` | always passed when `caseInsensitive !== true` | `caseSensitive = false` by default, `--case-sensitive` only when passed |
| sort | `--sortr <mode>` on the non-streaming path, nothing on the streaming path | no sort flag at all |

The `ripwalk` signature is `function ripwalk(ctx, { root, searchPaths = [], includeGlobs = [], excludeGlobs = [], caseSensitive = false, cursorIgnoreFiles = [], sandboxPolicy = { type: "insecure_none" }, noIgnoreVcs = false, followSymlinks = false, source = "ripwalk" })`, and its fixed prefix is `["--files","--hidden","--no-require-git","--no-config","--color=never"]`. It also prepends a `--threads` count from `getRipwalkThreadCount()`, which `buildArgs` never emits, and it caches results under a key built from every option.

Glob does not use `ripwalk`. It constructs a search request with `outputMode: "files_with_matches"` and dispatches to the shared executor. So every statement in this report about `.gitignore`, symlinks, and case sensitivity is a statement about `buildArgs`, and `ripwalk` has the opposite default on all three. A report that cites `ripwalk` to argue "symlinks are not followed" has read the wrong builder; a report that cites it to argue ".gitignore applies outside a git repository" has read the wrong builder in the other direction. Both conclusions are wrong for the path Glob actually takes, which is why both are called out. The subsection is a note on step 17, not a new step, so the numbering continues at 25 in the render layer below.

### Render layer

25. **Error case.** Return `truncateOutput(error, 5000, true).output`, which keeps a 2500-character head and a 2500-character tail joined by `\n\n... (output truncated) ...\n\n`.
26. **Absent case.** Return the literal `glob_file_search didn't return the result`.
27. **Zero files.** Return the zero template and stop. No banner, no count, no plural.
28. **Non-zero.** Compute the display path as `searchRoot.replace(/\/$/, "")`, and if the result is exactly `.`, strip a leading `./` from each path. Otherwise leave the paths alone.
29. Emit the header. Suppress the count when `ripgrepTruncated`.
30. **Emit lines under the budget.** For each file, if `buffer.length + file.length > 5000`, emit the mid-list notice and set `listedAll = false`, else emit `- ${file}\n`. Load bearing: the comparison is `>`, not `>=`, and it compares against the accumulated buffer including the header, so the effective capacity depends on the header length.
31. **Emit the banner** only when `listedAll` is still true, at least one file was emitted, and a cap was hit.

The two truncation families never both fire for the same cut. The mid-list notice fires when the character budget cuts; the banner fires when a cap was hit but the character budget did not cut. Reaching the banner with `clientTruncated` set requires the post-filter to remove almost everything, which is rare but real.

### Where order is load bearing, summarized

| Step | Wrong order produces |
|---|---|
| normalize before empty-check | `""`, `"/"`, `"//"` all slip through and search the whole tree |
| empty pattern into `GrepArgs` | content mode instead of listing mode, so empty and binary files vanish |
| sort before slice | the 2000 kept are the first found, not the newest |
| post-filter before sort | the wrong 2000 survive, and `totalFiles` disagrees with the rendered count |
| `--iglob !**/.git` before the positive glob | `.git` contents come back; later globs win |
| error classification before the empty-pattern retry | a recoverable case presents as a hard failure |

## Engineering questions

### Q1. Pagination and chunking

**There is no pagination.** No offset, no limit parameter, no cursor, no resume token, no page size. A caller gets up to whatever fits and cannot ask for the next slice.

This is worth stating as a finding rather than an omission, because two of the three comparison harnesses do it differently. Claude Code's Glob has a real `offset` in its internal search signature (`{ limit: r, offset: s }`) and slices `Ie.slice(s, s + r)` for the simple case, but the tool's `call()` always passes `offset: 0`, so the capability exists and is unreachable from the model. Its truncation notice even says "Narrow the pattern or path to see the rest," which is the only recovery either harness offers. Zed's `find_path` does expose `offset` to the model, page by page at 50 results, and tells the model to pass `offset` for more.

The recovery mechanism Cursor offers instead is in the notice text itself:

```text
... N more files ... (Do a more specific search if needed)
```

and

```text
[Results truncated. N files in total. Narrow the search with a more specific target_directory or glob_pattern.]
```

Both point at the two parameters that do exist. That is the whole strategy: narrow the query. It works reasonably for a model, which can add a directory component on the next call, and it fails completely for a caller that wants an exhaustive list.

**What this means for a reimplementation.** If you add paging, you must decide what a page is *of*, and the answer is not obvious here because the result is sorted by a property the caller cannot sort on. A `limit` plus an `offset` over an mtime-sorted list is not stable across calls unless the mtimes are stable, and they are not. A path-sorted list pages stably, which is one more argument for path sorting in Decision 3.

There is no page-boundary-mid-line problem because there are no pages, and no mid-UTF-8 problem for the same reason. The character budget can cut between a header and its first file, but it cannot cut inside a path: the comparison is `buffer.length + file.length > budget`, so either the whole path line fits or it does not. A single path longer than 5000 characters would produce a mid-list notice with zero files emitted, which is a degenerate case the code does not special-case.

### Q2. Units

Three different units, at three different layers, and they are not interchangeable.

| Cap | Unit | Exact comparison |
|---|---|---|
| 2000 entries | files | `totalFiles > CLIENT_LIMIT_LINES` for `clientTruncated`; `files.length < CLIENT_LIMIT_LINES` bounds the parse loop |
| 10000 lines | stdout lines | `totalFiles >= HARD_MAX_OUTPUT_LINES` for `ripgrepTruncated`, note `>=` not `>` |
| 5000 render characters | JavaScript string length | `accumulated.length + file.length > budget` |
| 8388608 stdout bytes | bytes | enforced by the process reader, kills the child past it |
| 25000 wall clock | milliseconds | `setTimeout` on an `AbortController` |

The render budget counts UTF-16 code units, because it is `String.prototype.length` on a JavaScript string. A path containing an astral character, or any emoji in a filename, counts as two units per code point. A byte-based reimplementation of the same budget will produce a different cut point on such a tree. That is a byte-level divergence that only shows up on unusual filenames, and it is worth knowing about before you chase a diff.

The entry cap counts *entries*, and in file-listing mode an entry is always a file, since ripgrep's `--files` emits files. Directories never appear. This is the opposite of `pi_find`, whose walk pushes directories too, with a trailing separator, and whose limit therefore counts directories against the budget.

### Q3. Text encoding

The tool's own strings are ASCII. Filenames are not necessarily.

Ripgrep writes raw OS bytes for each path, one per line. The files-mode parser splits on newline and returns substrings. Unlike the content path, which calls `decodeLinesAsRead` and has a 2000 ms encoding budget, **files mode does not run any decoder over the paths**. So a filename containing bytes that are not valid UTF-8 arrives as whatever the string assembly produced from a UTF-8 decode with replacement, and the model sees a path it cannot open.

This is a real limitation and Cursor has no error for it. On a POSIX filesystem a byte like `0xFF` in a filename is legal and common enough in a repository that has been moved between systems. On Windows, where I tried to create such a file for the probe, NTFS refused the name outright, so the case is POSIX-only in practice.

Two consequences worth deciding on.

**Newlines in filenames.** Ripgrep emits `--files` output newline-delimited by default, and the parser splits on `\n`. A filename containing a literal newline therefore splits into two bogus entries. The `-0` / `--null` machinery exists for the content path (`push("-0")` in content mode) but **Glob's files path does not use `--null`**. I could not construct such a file on Windows to prove the split, but the parser is unambiguous: it scans to `\n` and takes everything before it as the path. If you reimplement against a POSIX backend, either pass `--null` and split on NUL, or emit paths through an API that does not need delimiters at all. The VFS already does the latter, which is one reason to prefer it.

**Unicode normalization.** Nothing normalizes anything. macOS stores filenames in decomposed form (NFD) and Linux typically in composed form (NFC), so a pattern containing a precomposed `é` will fail to match a macOS filename stored as `e` plus a combining accent. globset compares bytes. A model that copies a path from one tool's output into another tool's `glob_pattern` is therefore at risk of a mismatch that is invisible on screen. Nothing in Cursor's chain is at fault here; it is a filesystem property. But it becomes your problem the moment you let the model round-trip a path through a pattern.

**BOM.** Irrelevant. The tool never reads file contents, so there is no BOM to preserve or strip.

### Q4. Line endings

Ripgrep always emits `\n` between records on every platform, including Windows. The parser strips a trailing `\r` before each `\n` anyway, which matters only if something upstream re-emits CRLF. Both behaviors are correct and neither is visible to the model.

The one place line endings could leak is the rendered output. The templates use `\n` explicitly, so the model receives LF regardless of host platform. A reimplementation that uses Rust's `writeln!` on Windows would produce `\r\n` and diverge by one byte per line. Use explicit `\n`.

There is no CR-only handling, and no need for any: no record in this tool is ever long enough or shaped such that a bare `\r` would appear inside one, except through a filename containing one, which falls under the newline-in-filename problem above.

### Q5. Binary detection

**None, and that is the point.** In `--files` mode ripgrep never opens a file to inspect it, so it cannot classify anything as binary. A file full of NUL bytes is listed exactly like a text file.

This was not always true, and the history is the most useful thing in the public record about this tool. Before October 2026, Glob ran `rg -l --iglob PATTERN`, which is content mode: ripgrep reads each file to test the regex, and by default it skips files it classifies as binary and files it finds empty. The result was that a `*.py` glob over a directory containing `real.py`, `empty.py`, and a NUL-containing `binary.py` reported one file, while `dir` reported three. Cursor's staff confirmed it and shipped the fix.

I reproduced both sides against the shipped `rg.exe` (`ripgrep 15.1.0-cursor5`):

```text
$ rg --files --iglob "**/*.py"
.\binary.py
.\empty.py
.\real.py

$ rg -l --iglob "**/*.py" --regexp ".*"
.\real.py
```

All three survive `--files`; only `real.py` survives `-l`. The `--files` form is what ships now.

The distinction still matters because the **empty-pattern fallback** re-introduces the old behavior. If the executor rejects an empty pattern, Glob retries with `pattern: "."`, which breaks the `--files` condition and falls through to `-l --regexp .`. That retry has the pre-fix semantics and would drop the empty and binary files. It never fires against the shipped executor, but a reimplementation that adds its own empty-pattern guard inherits the bug.

So: a file's contents are never consulted, the model is never told a file was skipped for being binary, and there is no way to ask for "only text files" through this tool.

**A related fact worth stating, because it is easy to assume otherwise.** Grep's content mode pushes `--max-columns 1000 --max-columns-preview`, which abbreviates any line over 1000 bytes. Glob never sees that abbreviation, because `buildArgs` pushes those two flags only inside its `mode === "content"` branch and Glob always calls it with `files_with_matches`. So a path list from Glob is never length-limited, and the only thing that bounds it is the 5,000-character render budget in the tool's own renderer. See `tool-grep.md` for what the abbreviation does to a matching line.

### Q6. Empty input and empty result

Each gets its own answer because each takes a different path.

| Input | Behavior |
|---|---|
| `glob_pattern: ""` | Rejected. Normalizes to `**/`, caught by the guard, throws `glob_pattern is empty. Pass "*" to match every file.` with `INVALID_ARGS` |
| `glob_pattern: "/"` | Rejected, same message. Strips the leading slash to `""`, then the trailing-slash test does not fire because the string is empty, then `**/` is prepended, then the guard catches it |
| `glob_pattern: "//"` | Rejected, same message. One leading slash stripped, then one trailing slash stripped, leaving `""`, then prefixed to `**/` |
| `glob_pattern: "**"` | Accepted. Does not start with `**/`, so becomes `**/**`, which works and matches everything |
| `glob_pattern: " "` | **Not rejected.** Becomes `**/ ` and is passed to `--iglob` as a literal path segment containing one space, which matches nothing. Renders the zero case |
| `glob_pattern: "*"` | Accepted, becomes `**/*`, matches every file the ignore rules allow |
| `target_directory` absent | Searches every workspace root, target `.` |
| `target_directory: ""` | Treated as absent by the `if (!params.path)` test in `searchTarget` and by `targetDirectory` falsiness in the display transform, so it searches all roots |
| empty directory | Zero case: `Result of search in '<p>': 0 files found` |
| zero matches | Zero case, same string. Exit code 1 from ripgrep is explicitly not an error |
| empty file in the tree | Listed, because contents are never read |
| directory named in `target_directory` that is empty | Zero case |
| result set empty after the post-filter | Zero case, because the filter runs before rendering |

The `" "` case is the interesting one. The guard is an equality test against the fully normalized pattern, not a trim-then-test, so any whitespace-only pattern that is not literally empty survives normalization and matches nothing. That is a silent failure rather than an error, and a model that sends a stray space gets a confident zero instead of a diagnostic.

There is a deliberate inverse of that bug available to you: `glob_pattern.trim()` before the normalizer, plus empty-after-trim rejection, turns a silent zero into a clear error. That is a divergence from Cursor, and I would take it. Confidence: high, because the failure mode is a model concluding a file does not exist.

### Q7. Cost

Worst case first, because that is what a hostile repository can force.

**Time.** The walk is bounded by a 25 s wall clock per root, and it is ripgrep's parallel walker, so on a warm cache it covers a very large tree quickly. The bound is per root, so a multi-root workspace costs 25 s per root worst case, sequentially. Cursor does run roots sequentially in the `for (const [root, result] of Object.entries(workspaceResults))` flatten, and the executor's own loop is sequential over roots too.

**A second, independent time cost sits after the walk.** Sorting by mtime requires one `stat` per returned path, issued in parallel via `Promise.all`, under a 2000 ms budget. On 2000 files that is 2000 concurrent stats, which will exhaust a typical file-descriptor limit or thread pool; Node's `fs.promises.stat` queues through libuv's threadpool, default size 4, so 2000 stats at 4-way concurrency is roughly 500 serial round trips. On a warm local disk that is well under a second. On a cold disk, a network mount, or a container with an overlay filesystem, it is easily over two seconds, and then the sort is abandoned wholesale. That is the mechanism behind the determinism problem in Q11.

**Memory.** The stdout cap is 8 MB, so the raw output is bounded. The parse retains up to 2000 paths plus a `totalFiles` counter that keeps counting to 10000 and then stops, so memory is bounded by 8 MB plus a bounded vector. There is no content buffering at all, because nothing is read.

**Worst case a hostile repository can force.** A tree with a very large number of files at shallow depth, all matching the pattern, plus a `.gitignore` that ignores nothing. Ripgrep walks until the 8 MB stdout cap or the 25 s deadline, whichever comes first. The parser stops at 10000 lines. The result is 2000 sorted entries and a `ripgrepTruncated` flag, and the render emits about 160 lines. So the *model-visible* cost is bounded at roughly 5000 characters no matter how hostile the tree. The *machine* cost is one full walk of the tree, every time, because there is no index and no cache on this path.

Two mitigations Cursor has elsewhere but does not use here, worth knowing because they are available to you. There is a `ripwalk` client with a directory-listing cache (`CURSOR_RIPWALK_CACHE_TTL_MS`, default 20000 ms) used by other consumers - Glob's own path spawns a fresh process and does not consult it. And there is an indexed grep provider (`executeIndexedGrep`) that the executor tries first, which would avoid the walk entirely; whether it answers a `files_with_matches` call with an empty pattern is a runtime property of the extension host and not determinable from the bundle.

**For the VFS.** Your `Access::glob` delegates to the backend, so cost is the backend's problem. `RealBackend` walking from disk pays what ripgrep pays, minus the parallelism. `MemoryBackend` iterates a map, which is much cheaper but still linear in the number of stored paths unless the backend indexes. The honest statement for a report like this: a glob over a virtual filesystem should be linear in the number of nodes, and any backend that cannot promise that should say so.

### Q8. Paths

This is where Glob differs most from the other five tools, and the difference is entirely in the pattern normalizer.

**The pattern is normalized, not resolved.** Four rewrites, in order: every `\` becomes `/`; one leading `/` is stripped; one trailing `/` is stripped; `**/` is prepended if absent. Nothing resolves `..`, nothing resolves symlinks, nothing makes anything absolute. The normalizer is a string function, and it runs before the executor, before the root resolution, and before zod has done anything but type-check.

Tracing a Windows path through it, which is the case the prompt asked me to pin:

| Input | After `\`→`/` | After leading strip | After trailing strip | After `**/` prefix |
|---|---|---|---|---|
| `src\**\*.ts` | `src/**/*.ts` | same | same | `**/src/**/*.ts` |
| `C:\work\proj\*.ts` | `C:/work/proj/*.ts` | same | same | `**/C:/work/proj/*.ts` |
| `\src\*.ts` | `/src/*.ts` | `src/*.ts` | same | `**/src/*.ts` |
| `src\` | `src/` | same | `src` | `**/src` |
| `\` | `/` | `""` | `""` | `**/` → rejected |
| `C:\` | `C:/` | same | `C:` | `**/C:` |

Two things follow. A Windows-style pattern is accepted and works, which is the good half. A Windows absolute pattern is destroyed in exactly the same way a POSIX one is: `C:/work/proj/*.ts` becomes `**/C:/work/proj/*.ts`, and since globset treats `:` as an ordinary character and no path segment sequence contains `C:`, it matches nothing. There is no error. The model gets the zero case.

**Output paths.** Ripgrep runs with `cwd` at the workspace root and the target passed as a relative positional, so it emits paths relative to that. On Windows those paths look like `.\src\file.ts` - backslash separated, with a leading `.\`. The renderer then strips a leading `./` only:

```js
"success" === e.result.case && e.result.value.path || "."
```

and

```js
r.files.map((e) => e.startsWith("./") ? e.slice(2) : e)
```

The strip applies only when the display path, after removing a trailing `/`, is exactly `.`. `.\src\file.ts` does not start with `./`, so the prefix survives. My probe confirmed ripgrep's raw Windows output is `.\a.js` and that the strip test cannot match it:

```text
$ rg --files --case-sensitive --iglob "**/*.js" --iglob "!**/.git" --no-config --color=never --hidden --follow -- .
.\B.JS
.\a.js
```

So on Windows the model sees `- .\README.md`, and on POSIX it sees `- README.md`. That is a one-character difference in every line on the platform most people will reimplement against, and it is the single most likely path-format bug in a clone.

**A supplied `target_directory` changes the reference point.** When `target_directory` is present, the paths are rewritten by `BEe` to be relative to the search directory rather than to the workspace root. The rewrite has three cases: target equals the root, target is inside the root, target is outside. The helper `OEe` guards each rewrite, rejecting a relative path that contains a `..` segment, a zero-length segment, or (on Windows) a drive colon. My probe confirmed raw ripgrep behavior, which the transform then adapts:

```text
$ rg --files --iglob "**/*.md" -- . ./docs
./docs\guide.md
```

**`..` in the target.** `searchTarget` resolves the target against the single workspace path, then tests containment. A `..` that escapes the root produces the absolute path, which is passed as the positional. Ripgrep will happily walk it. So `target_directory: "../secrets"` searches outside the workspace, subject only to the later permission and cursorignore post-filter. That is a real escape surface and worth deciding on deliberately: your VFS `OutsideRoot` error already closes it, and I would keep it closed.

**Symlinks.** `--follow` is unconditional. A link inside the workspace pointing outside it is traversed and its target's files are returned, because ripgrep sees a directory. A link loop produces `File system loop found` on stderr, which `hasOnlyUnsearchableEntriesBelowTarget` explicitly whitelists so it does not fail the search. A dangling link produces an `os error` line that is also whitelisted. If realpath itself fails during the pre-checks, the message is the long one naming access-denied and symlink loops.

**Case sensitivity.** The pattern is matched case-insensitively, always, because the flag is `--iglob`. The filesystem's own case sensitivity is irrelevant to pattern matching; it is relevant only to what ripgrep finds, and on a case-insensitive filesystem there is only one file to find. So `*.JS` matches `a.js` on both platforms, which is a genuine divergence from shell globbing and from what most models expect.

**Drive letters and UNC.** `target_directory` accepts an absolute path, a `~` path, and a `file://` URL, per `isRootIndependentSearchPath`. A UNC path such as `\\server\share` becomes `//server/share` after the target-side normalization, and the `.startsWith("//")` case is not special-cased anywhere, so it likely resolves as a POSIX-style path on Windows. I did not test this, and it belongs in `## What remains unknowable`.

### Q9. Concurrency

**Glob takes no lock at all.** It is not in the set of tools that a Read waits for before retrying a miss:

```js
["ApplyPatch","AwaitShell","CopyToBox","EditNotebook","MultiEdit",
 "MultiStrReplace","Shell","StrReplace","Write","edit_notebook",
 "run_terminal_cmd","search_replace","write"]
```

Every entry in that set mutates. Glob does not, so it is absent, and there is no per-path lock either - the per-file lock is keyed on a path, and Glob touches an unbounded set of paths that it does not know until the walk returns.

What that means concretely: two Glob calls in the same turn run concurrently and neither blocks the other, which is fine and desirable. A Glob concurrent with a Write can list a file that is being truncated, and the mtime it stats may be from before or after the write. A Glob concurrent with a Delete can list a file that no longer exists by the time the model tries to read it. None of these are errors; the tool has no mechanism to notice.

**The races it leaves open, named:**

| Race | Window | Result |
|---|---|---|
| glob versus write | between the walk and the stat | mtime may reflect pre-write or post-write content, so sort position is arbitrary |
| glob versus delete | between the walk and the return | a listed path may already be gone when the model reads it |
| glob versus rename | during the walk | the same file may appear under both names, or neither |
| glob versus directory creation | during the walk | a directory created after the walker passes its parent is missed entirely |

The last one is inherent to any single-pass walk and is not worth fixing. The first is worth deciding on: if your backend supplies mtime, and your sort depends on it, then a Glob racing a Write produces a nondeterministic order. That is the same class of problem as the stat timeout in Q11, and the same answer applies - make the determinism property be about the tree, not about concurrent mutation.

**For the VFS.** Your `Access::glob` calls `admit(Claims::claim_glob, &claimed)`, so a glob does take a claim, keyed on the canonicalized pattern. That is strictly stronger than Cursor, and the interesting question is whether a pattern claim conflicts with a path write. If it does, Glob and Write serialize correctly, which is better than Cursor. If it does not, you have the same races as above but at least a claim record to reason about. Worth settling before you write the tool, because the answer determines whether Glob can safely be called in the same batch as a Write.

### Q10. Fidelity

Glob never writes, so nothing can be corrupted by it. That is the honest answer, and the section could end there. Two adjacent facts are worth recording anyway.

**Nothing read is ever written back.** There is no read-modify-write, no temp file, no truncate, and no mode change. The only side effect on the filesystem is the `atime` update that the walk itself causes, which on most modern mounts is `relatime` and therefore usually not even that.

**Nothing in the tool changes file ordering for the next call.** Unlike a read-state store, Glob keeps no per-session memory of what it has seen. Claude Code's counterpart is concurrency-safe and read-only too, but Claude Code's harness has a separate `readFileState` LRU that Glob does not touch. So consecutive Glob calls are independent.

The one fidelity-adjacent behavior: the sort reads `mtimeNs` through `stat(path, {bigint: true})`, which is nanoseconds. A reimplementation that uses millisecond mtime will order files that differ by sub-millisecond timestamps differently. That is invisible in practice but it will break a byte-exact fixture recorded from Cursor. Use nanosecond precision if your `Stat` carries it, and note that Runvil's carries a `SystemTime`, which on most platforms is nanoseconds internally - so the precision is there, and only the platform's own granularity limits it.

### Q11. Determinism

This is the most important section in the report, because a file listing that varies between identical runs makes an agent's behavior unreproducible, and every downstream property - replay, caching, regression testing, eval scoring - depends on it.

**The short answer: two identical Glob calls on the same unchanged tree do not reliably produce byte-identical output.** There are five independent sources of variation, and they compose.

**Source 1: the walk order is not defined, and it is parallel.** Ripgrep's `--files` output order is the order its parallel walker happens to emit. It is not alphabetical, it is not depth-first, and it is not stable across runs. With `--sort` or `--sortr` ripgrep would sort, but Glob's streaming path passes neither:

```js
const sortMode = sortAfterRipgrep ? "none" : params.sort ?? "modified";
```

and `sortsAfterRipgrep` returns true for Glob always, because `sort` is undefined and streaming is on:

```js
sortsAfterRipgrep(e, t, r) {
  return void 0 === t.sort && (r ? true !== t.sortAscending : isAbsolute(this.searchTarget(e, t)));
}
```

So ripgrep emits in walk order and the JS layer sorts afterward. When the JS sort succeeds, walk order is invisible. When it fails, walk order **is** the output.

My probe demonstrated the non-determinism directly. The same tree, the same argv, two invocations: one returned `.\.gitignore | .\a.js | .\.env | ...` and another returned a different permutation. Neither order is alphabetical. That is the fallback state.

**Source 2: the stat budget.** 2000 ms for the whole batch. On a cold cache, a network mount, a container overlay, or a tree with 2000 files and a 4-thread libuv pool, it expires. When it does, `sortNewestFirst` returns the input untouched:

```js
catch (error) {
  if (error instanceof TimeoutError) {
    return { items: [...items2], statTimedOut: true };
  }
  throw error;
}
```

and the caller logs `newest_first.stat_timeout_fallback` and increments a counter labeled "Grep and Glob results left in ripgrep's order because stat was too slow to sort them." The model is told nothing. So the same call can be sorted on a warm cache and unsorted on a cold one, with no signal.

**Source 3: mtime ties, and mtime granularity.** The comparator falls back to path segments when mtimes are equal, which is a total order and therefore deterministic:

```js
function comparePathSegments({ left, right }) {
  const shared = Math.min(left.length, right.length);
  for (let index = 0; index < shared; index++) {
    const leftSegment = left[index] ?? "";
    const rightSegment = right[index] ?? "";
    if (leftSegment !== rightSegment) return leftSegment < rightSegment ? -1 : 1;
  }
  return left.length - right.length;
}
```

So ties are fine. But the tie-break only fires on exact equality of `mtimeNs`. On a filesystem with one-second granularity, a `git checkout` of a thousand files gives them all the same timestamp, and then the path tie-break saves you. On a filesystem with nanosecond granularity, files written in a loop differ by microseconds and the order becomes the write order, which is deterministic but arbitrary. Neither is wrong; both are surprising when a fixture recorded on one filesystem is replayed on another.

**Source 4: the sort uses `path.sep`, and the paths being split are already platform-specific.** `orderByModifiedDesc(files, mtimes, sep12)` splits each path on `import_node_path74.sep`. On Windows that is `\`, so a path `.\src\file.ts` splits into `[".", "src", "file.ts"]`. On POSIX a path `src/file.ts` splits into `["src", "file.ts"]`. The segment counts differ by one at the root, so the same tree sorted on two platforms can order differently in the tie case. It is a subtle inconsistency and it means a cross-platform golden test on the tie-break will fail.

**Source 5: the ignore inputs can change between runs.** `.gitignore` and `.cursorignore` are read per call. A build step that writes a `.gitignore` entry, or a `.cursorignore` written by a previous tool call, changes the result set. That one is legitimate - the tree changed - but it means the "same tree" precondition for determinism has to include the ignore files.

There is a sixth, which is not Cursor's fault but will bite you: **the mtime of a file is part of the input.** Two runs on a tree that was `git checkout`-ed between them produce different orders. A golden fixture recorded from a real repository is not reproducible on another checkout. Record fixtures from a controlled tree with explicitly set mtimes, or sort by path in the fixture.

**Would two identical runs produce byte-identical output?** On a warm cache, a local SSD, a tree small enough that 2000 stats finish in time, and an unchanged tree: yes, because the JS sort is a total order. Outside those conditions: no. The determinism guarantee is conditional on the stat budget, and the budget is not something the caller can observe or control.

**What a reimplementation should do.** Three concrete choices, in order of importance.

1. **Make the sort total and unconditional, or make it absent.** Never "sort if there's time." If the backend cannot supply mtime for a path, that path sorts last under a stable key, not into an arbitrary position.
2. **Report which order was used.** A one-word marker is enough. The model does not need it; a test harness does. Cursor's silent fallback is the mistake worth not copying.
3. **Make the tie-break platform-independent.** Compare full path strings lexicographically by bytes, not by splitting on the platform separator. That gives one order on every platform and removes source 4.

Recommendation for Runvil: sort by mtime when every path in the result set carries `Some(mtime)`, tie-break by full path bytes, and when any path lacks mtime, sort the entire set by path. That is deterministic given the tree and the backend, which is the strongest property available. Confidence: high, because the alternative is a tool whose output cannot be replayed and whose tests cannot be golden.

### Q12. Truncation and omission

Three caps, two announcement mechanisms, and one place where the count is not what it appears to be.

**The mid-list cut announces itself.** When the render buffer would exceed 5000 characters, the loop breaks and appends one of:

```text
... N more files ... (Do a more specific search if needed)
```

```text
... at least N more files ... (Do a more specific search if needed)
```

The `at least` form appears when `ripgrepTruncated` is set, because the true total is then unknown. `N` is `Math.max(0, totalFiles - emitted)`, which is the number of files that exist but were not shown. Both forms say "more files" even when `N` is 1, which is a grammar bug rather than a lie.

**The banner announces a cap that did not cut the list.** When the loop completes without a cut but a cap was hit, one of four forms is appended:

```text
[Results truncated. 2000 files in total. Narrow the search with a more specific target_directory or glob_pattern.]
```

```text
[Results truncated. At least 2000 files in total. Narrow the search with a more specific target_directory or glob_pattern.]
```

```text
[Results truncated. 1 file in total. Narrow the search with a more specific target_directory or glob_pattern.]
```

```text
[Results truncated. Narrow the search with a more specific target_directory or glob_pattern.]
```

The last is the `totalFiles === 0` case, which is reachable only when the post-filter removed everything while both truncation flags stayed set. The `At least` form appears under `ripgrepTruncated`.

In practice the banner almost never appears, for arithmetic reasons: if `clientTruncated` is set then 2000 entries were retained, and 2000 lines always exceed 5000 characters, so the mid-list cut fires first and clears the flag the banner needs. The banner needs the post-filter to remove nearly all 2000. Real but rare.

**Where the tool is less than fully honest.**

The header prints `(total N files)` where `N` is the parsed total, and under `clientTruncated` that `N` can be 5000 while only 2000 were retained and only about 160 are shown. The number is a true total and a false impression of the list. The mid-list notice does say how many more there are, so a careful reader can reconcile them - but the header alone overstates what the list contains.

Under `ripgrepTruncated` the count is suppressed entirely, which is honest. So the header has two shapes and the model has to notice which it got.

**What is *not* announced, and this is the important list:**

| Omission | Announced? |
|---|---|
| files excluded by `.gitignore` | never |
| files excluded by `.cursorignore` | never |
| files excluded by the permission post-filter | never, though `totalFiles` is decremented |
| the `.git` directory | never, and the model has no way to ask for it |
| files skipped because the walk timed out at 25 s | partially, via `ripgrepTruncated` becoming set on timeout, which suppresses the count and adds `at least` |
| files dropped because stdout hit 8 MB | same as above, `stdoutByteCapped` folds into `ripgrepTruncated` |
| a sort that did not happen | never |
| the empty-pattern fallback having switched to content mode | never, only a metric increments |
| a directory that could not be read (permission denied) | never, the stderr line is whitelisted |

That table is the answer to Q12. The tool announces what it cut for size and says nothing about what it cut for policy. A model that gets zero results for `*.log` because of a gitignore rule, or 40 results where 400 exist because 360 were cursorignored, has no way to distinguish that from the tree genuinely containing what it asked for.

**A reimplementation should fix the policy half of that table.** One line, appended when a policy filter removed anything:

```text
[12 paths excluded by ignore rules; 3 excluded by permissions.]
```

That is more useful than another truncation banner, because it distinguishes "not there" from "hidden from you." Confidence: high, because this is exactly the confusion that produced the field report cited in the sources.

## Failure matrix

Every failure the tool can produce, with the case it arrives as, the exact model-visible text, and the layer that produces it. Note that several failures present as *success* with a zero result, and those rows are marked.

| Condition | Case in the result oneof | Exact model-visible text | Layer |
|---|---|---|---|
| pattern is `""`, `"/"`, or `"//"` | error (`INVALID_ARGS`) | `glob_pattern is empty. Pass "*" to match every file.` | tool, after normalization |
| pattern is whitespace only | **success**, zero files | `Result of search in '.': 0 files found` | tool, silent |
| pattern is an absolute path | **success**, zero files | `Result of search in '.': 0 files found` | normalizer, silent |
| pattern has invalid glob syntax, e.g. `[a-` | error (environment) | `rg: error parsing glob '**/[a-': unclosed character class; missing ']'` | backend |
| pattern has an unmatched brace | error (environment) | `rg: error parsing glob '<normalized>': unclosed alternate group; missing '}' (maybe escape '{' with '{')` | backend |
| zero matches | **success**, zero files | `Result of search in '<p>': 0 files found` | tool |
| `target_directory` does not exist | error (environment) | `Path does not exist: <absolute path>` | executor, pre-spawn |
| `target_directory` is a file, not a directory | success (a single file is allowed) | `Result of search in '<p>' (total 1 file):` and the file, if it matches | executor, silent |
| `target_directory` filtered by cursorignore | error (environment) | `Path '<p>' is filtered out by .cursorignore` | executor, pre-spawn |
| `target_directory` realpath fails | error (environment) | `Path '<p>' could not be verified while resolving symlinks. This can happen when access is denied or the path contains a symlink loop.` | executor, pre-spawn |
| `target_directory` is neither file nor directory | error (environment) | `Path '<p>' is neither a file nor a directory` | executor, pre-spawn |
| target spans workspace roots | error (environment) | `Path '<p>' points to a different folder from each workspace root; use an absolute path` | executor root resolution |
| ripgrep binary missing | error (environment) | `Path does not exist: <cwd>` - names the directory, not the binary | executor spawn |
| ripgrep path not configured | error (environment) | `ripgrep path not configured` | capability service |
| 25 s wall clock exceeded, no output | error (`TIMEOUT`) | `Timed out after 25s` | executor |
| 25 s wall clock exceeded, partial output | success, with `ripgrepTruncated` set | the rendered list, with the count suppressed and `at least` in any notice | executor, keeps partial |
| run cancelled by the user | error | `Grep was cancelled` | executor |
| stdout exceeds 8 MB | success, `ripgrepTruncated` set | the rendered list with the count suppressed | executor |
| more than 10000 output lines | success, `ripgrepTruncated` set | the rendered list with the count suppressed | parser |
| more than 2000 files | success, `clientTruncated` set | the rendered list plus `... N more files ...` | renderer |
| rendered text exceeds 5000 characters | success, mid-list notice | `... N more files ... (Do a more specific search if needed)` | renderer |
| stat budget expires | **success**, order undefined | the rendered list, in walk order, with no marker | renderer, silent |
| executor returned neither success nor error | absent case | `glob_file_search didn't return the result` | renderer |
| unexpected executor case | error | `Unexpected result case: <case>` | tool |
| a subdirectory is unreadable | **success**, partial | the results found, with the unreadable subtree silently absent | executor, whitelisted stderr |
| a symlink loop exists | **success**, partial | results, with the looped subtree absent | executor, whitelisted stderr |
| unknown argument key | error (validation) | `Invalid arguments:\n<key>: <message>` | validation |
| wrong argument type | error (validation) | `Invalid arguments:\n<key>: <message>` | validation |
| error text longer than 5000 characters | error | 2500-character head, `... (output truncated) ...`, 2500-character tail | renderer |
| model calls the tool on a different version branch | n/a | `Unhandled version: <tag>` | registration, before the tool runs |

Three rows deserve a note.

**"target_directory is a file" is not an error.** My probe confirmed ripgrep accepts a file as the positional and emits it if it matches the glob. So pointing Glob at a single file works, which is a mildly useful accident and something a reimplementation that insists on a directory would break. The pre-check `isDirectory` test does not exist on this path; the checks are existence, cursorignore, file-or-directory, and realpath.

**"subdirectory unreadable" is deliberately swallowed.** `hasOnlyUnsearchableEntriesBelowTarget` returns true when every stderr line matches either `rg: <path>: ... (os error 13|5|2|1)` or `rg: File system loop found: <path> points to an ancestor`, and the named path is not the target itself. In that case the failure is not thrown. So a repo with one unreadable directory returns results and says nothing, which is the right call for a search and the wrong call for a report to the model.

**"missing ripgrep binary" names the wrong thing.** `Fx(e)` returns `Path does not exist: ${e}` where `e` is the cwd. A model reading that concludes the directory is gone. The diagnostics gatherer records the real cause (`rgPath`, `rgPathExists`, parent listings, asar state) but only for telemetry. This is a bug worth not reproducing.

## Traps

### Trap 1: `.gitignore` does not beat a positive glob, and the parent spec says it does

This is the biggest correction in the report, so it goes first.

The parent spec states, under `### Ignore behavior`, that "`.gitignore` is respected by default," and in its worked examples that "gitignored files absent (confirmed)" with the note "because `.gitignore` wins over a positive glob." The shipped ripgrep says otherwise, in its own `--help`:

```text
-g GLOB, --glob=GLOB
    Include or exclude files and directories for searching that match the
    given glob. This always overrides any other ignore logic.
```

I confirmed it against the shipped binary. A repo with `.gitignore` containing `*.log`, and a single `debug.log` at the root:

```text
$ rg --files --case-sensitive --iglob "**/*.log" --iglob "!**/.git" --no-config --color=never --hidden --follow -- .
.\debug.log
```

The `*.log` file that `.gitignore` excludes comes back. And with a directory-level rule, the behavior differs again:

```text
# .gitignore = "ignored/"
$ rg --files --iglob "**/ignored/*.js" ...      -> exit 1, nothing
$ rg --files --iglob "**/*" ...                 -> .\ignored\skip.js   (returned!)
```

Three distinct behaviors, and the reason is directory pruning.

- **File-level ignore rule** (`*.log`, `ignored/*`): the file is a candidate, the glob names it, `--iglob` wins, the file appears.
- **Directory-level ignore rule** (`ignored/`): ripgrep never descends into the directory, so no glob can name anything inside it. An explicit `**/ignored/*.js` returns nothing.
- **Catch-all glob** (`**/*`): it matches the *directory* path too, which resurrects the whole ignored subtree, files and all.

That third case is the genuinely alarming one. `glob_pattern: "*"` - the very pattern the error message recommends for "match every file" - normalizes to `**/*`, matches the directory nodes, defeats the directory-level ignore, and drags in everything the user thought was excluded.

My probe, in a repo with `.gitignore` = `*.key`, `vendor/`, `node_modules/`:

```text
glob "**/*"       -> .\.env | .\.gitignore | .\node_modules\left-pad\index.js |
                     .\secret.key | .\src\app.ts | .\vendor\lib.js
glob "**/*.ts"    -> .\src\app.ts
glob "**/*.js"    -> (nothing)
```

And the same tree with no `.git/info` at all, i.e. not a git repo:

```text
glob "**/*.js"    -> .\node_modules\left-pad\index.js | .\vendor\lib.js
```

So ripgrep requires a git repository for `.gitignore` to apply at all, unless `--no-require-git` is passed. Cursor does not pass it. In a directory that is not a git repo, `.gitignore` is inert and the walk sees everything.

This is worth pinning hard, because it is the claim most likely to be dismissed as an artifact of a bad probe. A probe run inside a subdirectory of a git repository will honor `.gitignore` even though the directory has no `.git` of its own, because ripgrep walks up to find the repository that contains it. So the case has to be tested somewhere with no enclosing repository at all. Verified that way on this machine, same two files, same argv, only the presence of `.git` differing:

```text
# outside any git repository (git rev-parse exits 128)
$ rg --files --hidden -- .
.\debug.log
.\a.js
.\.gitignore

$ rg --files --case-sensitive --iglob "**/*.log" --iglob "!**/.git" ... -- .
.\debug.log

# inside a git repository, same files
$ rg --files --hidden -- .
.\a.js
.\.gitignore

$ rg --files --case-sensitive --iglob "**/*.log" --iglob "!**/.git" ... -- .
.\debug.log
```

Read the first and second blocks against each other. Outside a repo, `.gitignore` has no effect at all, not even on the plain `--files` call with no positive glob. Inside a repo, `.gitignore` suppresses `debug.log` on the plain call and the positive `--iglob` brings it straight back. That is the whole trap in four commands.

**Why this is easy to get wrong.** Three reasons stack. The docs say ignore files are respected, which is true of the default `rg` invocation and false of this one. The parent spec asserts the opposite of what ships, so a reader who trusts it builds the wrong precedence. And the common case - `node_modules` excluded, `**/*.ts` returns only source files - looks exactly like ignore files working, because the extension glob does not match the directory. The bug only surfaces for a file-level rule whose extension the glob names, or for a catch-all pattern.

**The test that catches it.** A repo with `.gitignore` containing `*.log` and one `debug.log`, then `glob_pattern: "*.log"`. Cursor returns the file. If your implementation returns zero, you have chosen different precedence - which is a defensible choice, but it must be a choice. Add a second case: `.gitignore` containing `vendor/` and a `vendor/x.js`, then `glob_pattern: "*"`. Cursor returns `vendor/x.js`. Decide whether you want that.

**What I recommend.** Make ignore rules always win over the pattern, and never let a pattern resurrect an ignored path. Then say so in the description, because a model that has learned the opposite from Cursor will be surprised. Confidence: medium, because the argument for matching Cursor is model familiarity and the argument against is that a user's `.gitignore` is a statement of intent.

### Trap 2: `.cursorignore` is a patched ripgrep flag, and it is not ordinary ignore precedence

It is tempting to assume `.cursorignore` is applied like `.gitignore` and to model both the same way. It is not. Cursor ships a forked ripgrep with a `--cursor-ignore` flag that does not exist upstream, documented in the shipped `--help`:

```text
--cursor-ignore=PATH
    Specifies a path to one or more gitignore-formatted rules files that
    are applied with higher precedence than -g/--glob overrides. These
    rules can exclude files even when a glob would otherwise include them.
    Multiple files may be specified by repeating this flag; later files
    have higher precedence.

    Whitelist rules (lines starting with !) override ignore rules from
    every other ignore source, including .gitignore, .ignore, .rgignore,
    --ignore-file, global gitignore, and .git/info/exclude.

    This behaves like --ignore-file except for its higher precedence.
```

So the precedence chain, from strongest to weakest, is: `--cursor-ignore` rules, then `-g`/`--iglob`, then `.gitignore` / `.ignore` / `.rgignore` / global gitignore / `.git/info/exclude`. That is precisely inverted from what most people assume, and it is the reason a `.cursorignore` exclusion survives a catch-all glob while a `.gitignore` exclusion does not.

My probe confirmed the asymmetry in one tree: `.gitignore` = `*.log`, `.cursorignore` = `*.js`.

```text
"**/*"    -> .\.cursorignore | .\.gitignore | .\a.log | .\src\c.log     (js excluded)
"**/*.js" -> exit 1, nothing                                            (js still excluded)
"**/*.log"-> .\a.log | .\src\c.log                                      (log files returned)
```

**Why this is easy to get wrong.** If you model both ignore files as one list of gitignore rules, you get `.gitignore` right in the common case and `.cursorignore` wrong in the catch-all case, and both wrong in the file-level case. The two sources are not two instances of one thing; they are two different precedence tiers.

**The test that catches it.** One tree, both ignore files, three calls: a catch-all, a glob naming the `.gitignore`d extension, and a glob naming the `.cursorignore`d extension. Expect the first to return the gitignored file and not the cursorignored one, and the third to return nothing.

### Trap 3: `--case-sensitive` and `--iglob` in the same argv

The argv contains both flags, which reads like a bug:

```text
--case-sensitive
--iglob <pattern>
```

It is not. `--case-sensitive` governs the *content* regex, which is irrelevant in `--files` mode because there is no regex. `--iglob` governs the *pattern* and is documented as "Globs used via this flag are matched case insensitively." So the pattern is always case-insensitive and the `--case-sensitive` flag is a no-op on this path.

My probe confirms both directions:

```text
$ rg --files --iglob "**/*.js" ...       -> .\B.JS  and  .\a.js
$ rg --files --iglob "**/*.JS" ...       -> also matches a.js
```

**Why this is easy to get wrong.** Two ways. An implementer reading the argv may conclude the pattern is case-sensitive and build a case-sensitive matcher, which is wrong. Or an implementer may "clean up" the redundant flag and accidentally make the pattern case-sensitive, which is also wrong. The flag looks like a contradiction and is actually a statement about a different input.

**The test that catches it.** A tree with `a.js` and `B.JS`, pattern `*.JS`. Both must appear. On a case-sensitive filesystem this is unambiguous; on a case-insensitive one there is only one file and the test proves nothing, so run it on Linux or on a case-sensitive APFS volume.

**For Runvil.** Your `validate_glob_pattern` does not care about case, and `Access::glob` delegates matching to the backend. So the decision lands on the backend, and you should specify it: case-insensitive matching to match Cursor, or case-sensitive to match the platform. I would go case-insensitive at the tool layer, because a model that writes `*.JS` means "javascript files", and silently returning nothing is the worse failure. Confidence: medium-high.

### Trap 4: normalization destroys absolute patterns, and the description invites them

The `glob_pattern` description ends with three examples, the last of which is `"**/test/**/test_*.ts"`. A model reading that reasonably infers that patterns are path-shaped and may supply an absolute one. The normalizer then does this:

```text
/home/dev/proj/src/*.ts
  -> (strip leading slash)  home/dev/proj/src/*.ts
  -> (prepend **/)          **/home/dev/proj/src/*.ts
```

That is a suffix match requiring a directory literally named `home`, then `dev`, then `proj`, then `src`, relative to the workspace root. On a workspace at `/home/dev/proj` the actual relative path is `src/foo.ts`, which does not contain those segments at the end. Zero matches, no error.

The same happens to `C:/work/proj/*.ts`, and to `C:\work\proj\*.ts` after the backslash rewrite.

**Why this is easy to get wrong.** Two ways. An implementer may "fix" the normalizer by not prepending `**/` when the pattern looks absolute, which changes behavior for a pattern like `src/*.ts` that happens to have a leading component matching a root name. Or an implementer may decide the empty result is correct and never notice that a whole class of plausible model inputs is silently dead.

**The test that catches it.** A workspace with one file at `src/a.ts`. Call with `glob_pattern: "/src/*.ts"` - this works, because stripping the slash gives `src/*.ts` then `**/src/*.ts`, which matches. Then call with the absolute path to that same file. The first returns the file; the second returns zero. That pair pins both halves.

**What I recommend.** Detect an absolute pattern and either reject it with a message naming `target_directory`, or route it into `target_directory` for the caller. Both are better than a silent zero. Confidence: high, because a silent zero on a well-formed-looking input is the worst possible response.

### Trap 5: the sort is conditional, and the condition is invisible

Covered at length in Q11; recorded here because it is a trap and not just a design note.

`sortNewestFirst` returns the input untouched on a 2000 ms timeout. The caller logs a metric and moves on. The renderer emits the list in whatever order arrived. Nothing in the model-visible text says the order is arbitrary.

**Why this is easy to get wrong.** A reimplementation that reads the description, sees "sorted by modification time", and implements an unconditional sort will be *more* correct than Cursor and will fail a byte-comparison test on a large tree. Conversely, a reimplementation that copies the budget and the silent fallback inherits a nondeterminism it probably does not want. The trap is that both are defensible and the choice is invisible in the output.

**The test that catches it.** Run the same Glob call twice on a tree with more than a few thousand matching files, with a cold page cache (`echo 3 > /proc/sys/vm/drop_caches` on Linux, or just a large tree on Windows). If the two outputs differ, the fallback fired. On a warm cache the test passes and proves nothing, which is why this one gets missed.

### Trap 6: the cap the model sees is not the cap in the constant table

Three caps are named in the source: 2000, 10000, and 8388608. The cap that actually binds on a normal repository is none of them; it is the 5000-character render budget, which is not a "limit" in the executor at all and lives in the render function.

Rough arithmetic: the header is about 30 characters, each line is `- ` plus the path plus `\n`, and a typical repo-relative path is 25 to 40 characters. So 5000 characters is about 150 to 190 files.

**Why this is easy to get wrong.** An implementer who reads `CLIENT_LIMIT_LINES = 2000` and builds a test with 300 matching files expects 300 lines and gets about 160 plus a notice. Then the test is "fixed" by raising the render budget, which changes the model-visible cost of every call. The caps are in three different layers and the tightest one is the least visible.

**The test that catches it.** 400 matching files with paths of about 30 characters. Expect roughly 160 `- ` lines, then `... 240 more files ... (Do a more specific search if needed)`. If you see 400 lines, you have no render budget. If you see exactly 2000, you have the entry cap but not the character cap.

Also worth pinning: the budget compares `>` not `>=`, and it includes the header in the accumulated length, so the cut point shifts by the header's length. A byte-exact fixture must account for that.

### Trap 7: `./` is stripped, `.\` is not

The strip test is `startsWith("./")`, applied only when the display root is exactly `.`:

```js
const s = "." === t.replace(/\/$/, "")
  ? r.files.map((e) => e.startsWith("./") ? e.slice(2) : e)
  : r.files;
```

On Windows, ripgrep emits `.\src\file.ts`. That does not start with `./`, so the prefix survives and the model sees `- .\src\file.ts`. On POSIX the same call renders `- src/file.ts`.

**Why this is easy to get wrong.** A reimplementation written on Linux, tested on Linux, and shipped on Windows produces paths a model will then pass to other tools, where `.\` may or may not be normalized. And a golden fixture recorded on one platform fails on the other for a reason that looks like a path-handling bug rather than a one-character prefix difference.

**The test that catches it.** One tree, one call, two platforms, compare bytes. Expect `.\a.ts` on Windows and `a.ts` on POSIX from the same pattern. If you want them to agree, normalize the separator in the renderer - which is a deliberate divergence and worth a line in your changelog.

There is a second half to this trap: when a `target_directory` is supplied, the strip does not apply at all, and paths are rewritten relative to the target by `BEe`. So the same file renders as `src/a.ts` with no target and as `a.ts` with `target_directory: "src"`. A test that only exercises one shape will miss the other.

### Trap 8: the empty-pattern fallback changes the engine, not just the pattern

If the executor rejects an empty pattern, the tool retries with `pattern: "."`. That does not merely fill in a default; it flips the executor out of `--files` mode:

```js
const o = (e, t) => "files_with_matches" === t && "" === e.pattern && Boolean(e.glob);
```

With `pattern: "."`, `o` is false, so `buildArgs` emits `-l` instead of `--files`, and `-l` means ripgrep reads every file to test the regex `"."`. Empty files do not match `"."`. Files ripgrep classifies as binary are skipped. So the retry silently reverts to the pre-October-2026 semantics.

My probe shows the difference directly on the same three files:

```text
$ rg --files --iglob "**/*.py"           -> binary.py, empty.py, real.py
$ rg -l --iglob "**/*.py" --regexp "."   -> real.py
```

**Why this is easy to get wrong.** The retry is a small function with a clear purpose ("handle executors that need a pattern"), and it looks like a compatibility shim with no semantic content. It has semantic content. An implementer who adds an empty-pattern guard to their own executor, then keeps the fallback for compatibility, gets the old bug back.

**The test that catches it.** A directory with `real.py`, `empty.py`, and a NUL-containing `binary.py`, and a stubbed executor that returns `received an empty pattern` for the first call. All three files must be listed after the retry, or the fallback is in content mode.

**For Runvil.** Do not have an empty-pattern guard in your matcher at all. Your `Access::glob` already rejects an empty pattern with `PathReason::Empty`, which is the right layer for that check - it means the tool must translate `glob_pattern: ""` into `PathReason::Empty` before calling, rather than calling and retrying. Confidence: high, because a retry that changes mode is a bug waiting to be reintroduced.

### Trap 9: the render fallback names the wrong tool

```js
if ("success" !== e.result.case) return "glob_file_search didn't return the result";
```

This fires on all seven branches, including the six where the tool is called `Glob`. `glob_file_search` exists only as the `dsv3-1018` name, so the string reads like dead code left behind by a rename. It is reachable: any executor result whose case is neither `success` nor `error`, and any flatten that finds no root with a `files` result, lands here.

**Why this is easy to get wrong.** It looks like dead code, so an implementer deletes it or replaces it with a `Glob`-flavored string, and then the branch that reaches it has no message at all. Or an implementer copies it verbatim into a tool that is never called `glob_file_search`, which is a small wrongness that a model reading the error will not understand.

**The test that catches it.** Stub the executor to return a result with `case: undefined` and confirm the model sees exactly `glob_file_search didn't return the result`. Then decide: reproduce it for byte compatibility, or fix it to name the actual tool. Either is fine; drifting into it by accident is not.

### Trap 10: `totalFiles` is decremented after the slice, so the header and the list disagree

Order in `runFilesMode`: parse, sort, slice to 2000, post-filter, and subtract the dropped count from `totalFiles`. So `totalFiles` is the post-filter count while the *slice* was taken pre-filter. If 1900 of the 2000 are cursorignored, the result is 100 files rendered under a header that says `(total 100 files)` - which is arithmetically consistent - but the *reason* only 100 survived is invisible, and the truncated-at-2000 fact is lost.

Worse, in the reverse direction: if the parse counted 5000 and the slice kept 2000 and the filter dropped 0, the header says `(total 5000 files)` and 2000 were candidates and about 160 are shown. Three different numbers, one of which appears in the output.

**Why this is easy to get wrong.** The decrement looks like bookkeeping hygiene. It actually changes the meaning of the field from "how many matched" to "how many survived policy," and the renderer treats it as the former.

**The test that catches it.** 2500 matching files, of which 2400 are cursorignored. The header and the `N more files` arithmetic must reconcile with the visible lines. In Cursor they do not, cleanly, and the mid-list notice will report a number larger than the list plus the dropped set.

### Trap 11: multi-root order is per root, not global

With no `target_directory`, every workspace root is searched and the results are concatenated in root order. Each root was sorted internally. No global sort is re-applied. So a file in root 2 that is newer than everything in root 1 still appears after all of root 1's files.

`totalFiles` sums across roots, and the truncation flags OR across roots. So a workspace with two roots where one is truncated and the other is not produces a single `ripgrepTruncated` flag that suppresses the header count for both.

**Why this is easy to get wrong.** The flatten function looks like a simple reduce and the per-root sort looks like it composed. It does not.

**The test that catches it.** Two roots, distinct mtimes, one file per root with the newer file in root 2. The output must show root 1's file first if the order is per-root; a global sort would show root 2's first.

### Trap 12: a filename with a newline splits into two entries

The files-mode parser scans to `\n` and takes everything before it as one path. It does not pass `--null`, and the content path's `-0` flag is not used here. So a POSIX filename containing `\n` becomes two lines, and therefore two entries, neither of which exists.

I could not create such a file on Windows (NTFS refuses the name), so this is derived from the parser rather than measured. The parser is unambiguous, and ripgrep's documented behavior is newline-delimited output unless `--null` is given.

**Why this is easy to get wrong.** Nobody tests it, because creating the file requires `touch $'a\nb'` and most test harnesses cannot express it.

**The test that catches it.** On Linux: `touch $'weird\nname.ts'`, then `glob_pattern: "*.ts"`. Cursor returns two bogus paths. Your implementation should return one path, either by splitting on NUL or by never putting paths in a delimiter-joined stream.

**For Runvil.** Your `Access::glob` returns `Vec<String>`, so there is no delimiter and the problem does not exist. This is a concrete advantage of the VFS design over the subprocess design and worth stating as one.

### Trap 13: `--hidden` means the walk sees things the model should not

`--hidden` is unconditional, so `.env`, `.ssh`, `.git` (except for the explicit negation), `.venv`, and every other dotfile are candidates. There is no user-facing way to turn it off.

In practice `.env` is gitignored in most repos and therefore excluded by the directory rules, and `.git` is excluded by the literal glob. But a `.env` that is *not* gitignored, or a tree that is not a git repo at all (where `.gitignore` is inert, per Trap 1), puts `.env` in the result set for `glob_pattern: "*"`.

**Why this is easy to get wrong.** A reimplementation that skips hidden files by default is doing the safe thing and diverging. A reimplementation that follows `--hidden` and has no ignore layer exposes secrets.

**The test that catches it.** A non-git directory with `.env` and `glob_pattern: "*"`. Cursor returns `.env`. If your implementation does too, make sure your ignore layer is what protects the user, because `--hidden` will not.

### Trap 14: the description promises speed, and the mechanism that fails is the sort

"Works fast with codebases of any size" is true of the walk, which is parallel and capped. It is not true of the sort, which is a serial-ish 2000-way stat burst under a 2 second deadline, and it is not true in the sense a user expects, because the walk happens on every call with no cache.

**Why this is easy to get wrong.** The sentence is about the walk and readers apply it to the whole tool. An implementer who benchmarks the walk and finds it fast concludes the tool is fast.

**The test that catches it.** Time a Glob call on a repo with 2000 matching files on a cold cache, and separately time the same call where stat is artificially slowed. The second should show the fallback firing and the total time pinned near the budget.

## How three harnesses do this

All three have a file-finding tool, which makes this the most directly comparable tool in the six. Each is `(source)` since the code is on disk.

### Claude Code

Source: `cabinet\_scratch\claude-code\tools\glob.js`, extracted from `claude.exe` at the offsets in its header. Name: `Glob`. The registration object is `q0`, the search core is `m6t`, the truncation notice is `hGo`.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `pattern` | string | yes | `The glob pattern to match files against` |
| `path` | string | no | `The directory to search in. If not specified, the current working directory will be used. IMPORTANT: Omit this field to use the default directory. DO NOT enter "undefined" or "null" - simply omit it for the default behavior. Must be a valid directory path if provided.` |

The `path` description is unusual and worth reading closely: it contains an explicit instruction not to send the literal string `"undefined"` or `"null"`, which means models were doing exactly that often enough to warrant a schema-level warning. That is a real finding about model behavior, and Cursor's schema has nothing equivalent.

The engine is ripgrep, but with a materially different argv:

```js
let ke = ["--files", "--null", "--glob", n, "--sort=modified",
  ...(q ? ["--no-ignore"] : []),
  ...(Q ? ["--hidden"] : [])];
for (let st of z) ke.push("--iglob", st);
```

Read that against Cursor's and the differences are all load-bearing.

| Dimension | Cursor | Claude Code |
|---|---|---|
| Pattern flag | `--iglob` (case-insensitive) | `--glob` (case-sensitive) |
| Ignore files | respected | **disabled by default**, `CLAUDE_CODE_GLOB_NO_IGNORE` defaults `"true"` |
| Hidden files | always on | on, `CLAUDE_CODE_GLOB_HIDDEN` defaults `"true"` |
| Sort | in JS after the search, conditional | `--sort=modified`, in ripgrep, unconditional |
| Record delimiter | newline | `--null`, split on NUL |
| Recursive prefix | always prepended | resolved from the pattern's own base directory |
| Cap | 2000 entries, 5000 chars | 100 results |
| Paging | none | `offset` exists internally, always 0 from `call()` |
| Output | header + `- path` lines | bare paths, one per line, no header |

The `--sort=modified` is the most interesting divergence. Claude Code asks ripgrep to sort during the walk, which is unconditional and cannot time out. Cursor sorts afterward in JavaScript, which is where the timeout lives. Claude Code's approach costs a slower walk, because ripgrep's sorted mode is single-threaded by necessity, and buys a guarantee Cursor does not have. That trade is the single most transferable idea in this harness.

`--null` is the second. Splitting on NUL removes the newline-in-filename problem entirely, at the cost of one flag.

The caps, from `limits.js`: `globLimits.maxResults ?? 100` in the `call()` body, and `QFe = 200` is the output line cap before the truncation notice. So the *search* limit is 100 and the *render* cap is 200, which means the render cap never binds - a small inconsistency in the constants that is harmless because the search limit is tighter.

The truncation notices, all three forms:

```text
(Results are truncated. Consider using a more specific path or pattern.)
(Showing ${n} of ${total} matching files; ${r} more are not listed. Narrow the pattern or path to see the rest.)
(Showing the first ${n} files; there are more than ${total} matches. Narrow the pattern or path to see the rest.)
```

The second and third distinguish a complete count from a lower bound, via `countIsComplete`. That is a better honesty design than Cursor's count suppression, because it always gives the model a number and separately says whether the number is exact.

Zero results render as exactly:

```text
No files found
```

**Idea worth taking:** `--sort=modified` in the engine rather than after it. It makes the ordering guarantee unconditional and removes the entire class of determinism problems in Q11. The cost is a serialized walk, which for a search tool is usually the right trade.

**Idea worth rejecting:** `--no-ignore` by default. Claude Code shows the model every file including `node_modules`, which on a large repo is a much worse outcome than a hidden file. Cursor's choice to respect ignore files is better, provided the precedence is sane.

**One more thing worth noting.** Claude Code's Glob is concurrency-safe and read-only by declaration (`isConcurrencySafe()` returns true, `isReadOnly()` returns true), and it declares `maxResultSizeChars: 1e5`. That is the same shape as Cursor's lock-free treatment, arrived at independently.

### VS Code Copilot

Source: `C:\Users\Vinnie\src\vscode\extensions\copilot\src\extension\tools\node\findFilesTool.tsx`, with the name from `common\toolNames.ts` line 25: `FindFiles = 'file_search'`. The internal telemetry name is `copilot_findFiles`.

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | string | yes | - |
| `maxResults` | number | no | 20, or 200 in `FullContext` mode |

An explicit `path` parameter is **rejected**:

```ts
if ((options.input as unknown as Record<string, string>).path) {
  throw new Error('The property "path" is not supported');
}
```

That is a hardcoded refusal with a TODO above it reading `// TODO strict input validation` and a comment that "Certain models just really want to pass incorrect input." Same finding as Claude Code's `"undefined"` warning, from a different vendor: models add a `path` parameter to a file-find tool whether or not the schema has one.

The engine is not a subprocess at all. It is the editor's own search service:

```ts
const results = await raceTimeoutAndCancellationError(
  (searchToken) => Promise.resolve(this.searchService.findFiles(globResult.patterns, { caseInsensitive: true }, searchToken)),
  token, 20_000, 'Timeout in searching files, try a more specific search pattern');
```

That is the deepest architectural difference in this comparison. Because it uses the workspace index rather than the filesystem, it sees unsaved buffers, respects VS Code's own `search.exclude` and `files.exclude`, and is fast on a large repo in a way no walk can match. The cost is that it only sees what the editor has indexed: a file created outside the editor in a non-watched directory may not appear, and a virtual filesystem has no equivalent.

Note `{ caseInsensitive: true }` is passed explicitly, and `findFiles.spec.tsx` pins it: `expect(searchService.lastOptions?.caseInsensitive).toBe(true)`. So Copilot made the same case-insensitivity call as Cursor, on purpose, with a test.

The pattern rewrite happens in `resolveInput`:

```ts
if (!query.startsWith('**/') && !query.startsWith('/') && !query.includes(':')) {
  query = `**/${query}`;
}
if (query.endsWith('/')) {
  query = `${query}**`;
}
```

Two things there are better than Cursor's normalizer. It does **not** prefix when the query already starts with `/` (an absolute path), and it does **not** prefix when the query contains a `:` (a Windows drive). So an absolute path is left alone and handled by `inputGlobToPattern`, which converts it into a `RelativePattern` scoped to the matching workspace folder. That is the behavior Cursor lacks and the trap in Trap 4. And `query.endsWith('/')` becomes `**`, i.e. a trailing slash means "everything under this directory," which is the reading a model intends.

The 20 s timeout message is a plain string the model sees:

```text
Timeout in searching files, try a more specific search pattern
```

Output shape, from `FindFilesResult`:

```text
N total results
<path>
<path>
...
```

with a literal `...` line when `totalResults > fileResults.length`. Zero results:

```text
No files found
```

So three harnesses, three shapes: Cursor's `Result of search in '<p>' (total N files):` header with `- ` bullets, Claude Code's bare newline list, Copilot's `N total results` header with bare paths and a bare `...`.

**The `test_search` specialization.** `findTestsFilesTool.tsx` registers `ToolName.FindTestFiles = 'test_search'`, and it is a genuinely different kind of tool: it takes `filePaths: string[]` and optional `ranges`, and for each path it finds the *related* test file or source file using `TestFileFinder`, checking `isTestFile(document)` to decide the direction. It does not take a pattern at all. This is worth a paragraph because it is the one place in the set where a vendor decided file discovery deserved a purpose-built tool rather than a better glob. The implication is that the general glob is bad enough at the "where is the test for this file" question that Microsoft built a separate path for it, with language-aware heuristics rather than a pattern. Whether that is worth copying depends on how often your agent asks that question; if it does often, a `related_files` tool is a better investment than a smarter Glob.

**Idea worth taking:** the absolute-path handling. Copilot detects an absolute query, does not mangle it with a recursive prefix, and converts it into a scoped pattern against the right workspace folder. That is the correct answer to Trap 4 and costs about ten lines.

**Idea worth rejecting:** delegating to a workspace index. It is faster and it sees unsaved buffers, but it means the tool's answer depends on indexing state, which is a worse determinism story than Cursor's and much worse for a virtual filesystem that has no index to consult. Take the pattern handling, not the backend.

### Zed

Source: `C:\Users\Vinnie\src\zed\crates\agent\src\tools\find_path_tool.rs`. Name: `find_path`, which is `find_PATH` and not `find_FILES` - and that name is accurate, because it returns directories as well as files.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `glob` | string | yes | `The glob to match against every path in the project.` |
| `offset` | usize | no | `Optional starting position for paginated results (0-based). When not provided, starts from the beginning.` |

`offset` has `#[serde(default)]`, so it is genuinely optional and defaults to 0.

The matcher is Zed's own `PathMatcher` over `globset`:

```rust
let path_matcher = match PathMatcher::new(
    [if glob.is_empty() { "*" } else { glob }],
    path_style,
) {
    Ok(matcher) => matcher,
    Err(err) => return Task::ready(Err(anyhow!("Invalid glob: {err}"))),
};
```

Three things follow. An **empty glob becomes `*`**, which is the opposite of Cursor's rejection and much friendlier: "match everything" is a reasonable thing to ask for and Cursor makes you spell it `*`. The matcher is `globset`, the same engine family ripgrep uses, so brace expansion and character classes behave the same way - and since `globset` is a Rust crate you can depend on directly, this is the most transferable choice in the whole comparison for a Rust harness. And an invalid glob returns `Invalid glob: <err>`, carrying the crate's own message, rather than a `rg:` prefixed line.

The enumeration is the project's in-memory snapshot, not the filesystem:

```rust
for snapshot in snapshots {
    for entry in snapshot.entries(false, 0) {
        if path_matcher.is_match(&snapshot.root_name().join(&entry.path)) {
            results.push(snapshot.absolutize(&entry.path));
        }
    }
}
```

`entries(false, 0)` means `include_ignored: false`, so ignore rules are honored from the snapshot's own state. And because `entries` iterates an ordered map keyed by path, **the output is in path order, not mtime order**, and the tool's own doc comment says so:

```text
- Returns matching file paths sorted alphabetically
```

That is an explicit, documented divergence from both Cursor and Claude Code, and it is the honest version of Decision 3: pick one order, say which, and be unconditional. For a Rust harness that cannot cheaply stat every match, alphabetical is the right pick.

Output shape, from the `From<FindPathToolOutput>` impl:

```text
Found {all_matches_len} total matches.
Showing results {offset+1}-{offset+n} (provide 'offset' parameter for more results):
<abs path>
<abs path>
```

with paths as **absolute** paths, not workspace-relative. When the page is empty:

```text
No matches found
```

And when the total fits in one page, the "Showing results" line is omitted entirely, leaving just the count and the paths.

Paging is real: `RESULTS_PER_PAGE = 50`, and the slice is `matches[cmp::min(input.offset, matches.len()) .. cmp::min(input.offset + 50, matches.len())]`. Both bounds clamp, so an out-of-range offset gives an empty page rather than a panic. The model is told how to continue, in the same line that tells it there is more. That is the cleanest paging story of the three and the one worth copying.

Note the paths are absolute, so a paged result set is stable across calls in a way a relative one is not if the workspace root changes. That matters for resume.

**Idea worth taking:** `offset` in the tool schema, clamping both slice bounds, and stating the page range in the output. Three lines of code and it turns a truncation into a resumable conversation.

**Idea worth rejecting:** returning absolute paths when everything else in the harness is workspace-relative. It makes every downstream tool call carry a long prefix, and it differs from Cursor, so a model switching between the two has to relearn. Relative paths plus a documented root is cheaper.

**A note on the neighbors.** Zed also has `list_directory_tool.rs` (43 KB) and no separate glob-vs-ls ambiguity, because `find_path` returns entries of any kind and `list_directory` shows one level. Cursor's split - `Glob` returning files only, `ListDir` returning one directory - leaves no way to ask for "all directories named `test`," which Zed gets for free. Runvil's `Access::glob` supports the trailing-slash directory form, so this is a capability you already have and Cursor does not. Worth exposing.

### Comparison table

Dimensions that change a design, across all four.

| Dimension | Cursor | Claude Code | Copilot | Zed |
|---|---|---|---|---|
| Tool name | `Glob`, or `glob_file_search` on one branch | `Glob` | `file_search` | `find_path` |
| Parameter shape | `glob_pattern` + `target_directory` | `pattern` + `path` | `query` + `maxResults`, `path` refused | `glob` + `offset` |
| Engine | ripgrep `--files` subprocess | ripgrep `--files` subprocess | editor workspace index | in-process `globset` over a project snapshot |
| Matching rule | case-insensitive, `--iglob` | **case-sensitive**, `--glob` | case-insensitive, explicit | `globset` default, case per path style |
| Recursive prefix | always `**/` if absent | resolved from pattern base | `**/` unless absolute or drive-colon | none; pattern is used as given |
| Absolute pattern | becomes dead suffix match | supported | converted to a scoped `RelativePattern` | matched against `root_name/rel` |
| Directory results | files only | files only | files only | **files and directories** |
| Ignore files | `.gitignore` + `.cursorignore`, with `--cursor-ignore` precedence | **disabled by default** | editor `search.exclude` and `files.exclude` | snapshot's own ignore state |
| Hidden files | always included | always included | index-dependent | included unless ignored |
| Sort | mtime desc, in JS, 2000 ms budget, silent fallback | mtime via `--sort=modified`, unconditional | index order | **alphabetical by path**, unconditional |
| Cap | 2000 entries / 10000 lines / 5000 chars | 100 results | `maxResults`, default 20 or 200 | 50 per page |
| Units of the cap | files, lines, characters | results | results | results per page |
| Paging | none | `offset` internal, always 0 | none | **`offset` in the schema** |
| Truncation notice | mid-list `... N more files ...` plus banner | three forms, with count-completeness flag | bare `...` line | `Showing results A-B (provide 'offset' ...)` |
| Encoding handling | none; raw bytes to string | `--null` delimiter | editor API | Rust `PathBuf`, OS-native |
| Gating | none; read-only, no lock | permission checker, declarative read-only | tool registry | `ToolPermissions` |
| Output format | header + `- path` bullets | bare paths | `N total results` + bare paths | `Found N total matches.` + absolute paths |
| Zero-result text | `Result of search in '<p>': 0 files found` | `No files found` | `No files found` | `No matches found` |

Four rows in that table are worth staring at, because they are the ones where all four disagree.

**Absolute patterns.** Cursor breaks them, Claude Code resolves them, Copilot converts them, Zed matches them against the joined path. Three out of four handle them; Cursor is the outlier, and it is the one whose surface models are trained on. Fix this.

**Sort.** Two sort by mtime (one conditionally), one by index, one by path. Only one documents its order in the description and honors it unconditionally. Copy Zed's approach to the guarantee and Cursor's choice of key if you want recency.

**Paging.** Only Zed exposes it. Claude Code has the machinery and hides it. That asymmetry suggests the capability is cheap and the surface change is expensive, which is a reason to build it into the schema from the start rather than retrofit it.

**Directories.** Only Zed returns them. Everyone else silently means "files." If your `Access::glob` can return both, deciding which the tool returns is a real choice; returning both is more useful for structural questions like "where are the test directories."

## What Runvil can support today

The gap against `runvil-internal/vfs` and the `plugin-std` Pack as it stands. Every row is a behavior this report specified, mapped to the `Access` method that would cover it.

| Behavior | `Access` method | Status | What the gap needs |
|---|---|---|---|
| Enumerate paths matching a pattern | `glob` | covered | Already validates, gates, claims, fires, delegates, and supports trailing-`/` for directories only |
| Restrict the search to a subdirectory | `glob` | covered | Prepend the directory to the pattern, or add a root argument |
| Files only, excluding directories | `glob` | covered | `glob` with no trailing slash returns files only |
| Directories only | `glob` | covered | Trailing `/`. Cursor has no equivalent; this is a Runvil advantage |
| Return paths relative to the root | `glob` | covered | `glob_pattern` strips the root for relative patterns |
| Return absolute paths | `glob` | covered | A pattern starting with `/` is absolute and results come back absolute |
| Sort newest first by mtime | `glob` + `stat` | partial | `glob` returns `Vec<String>` with no metadata and no order guarantee beyond "sorted." Getting mtime means a `stat` per match, then a sort in the Pack. `Stat.modified` is `Option<SystemTime>` and nothing guarantees the backend fills it |
| Sort by path when mtime is unavailable | `glob` | covered | The backend already returns sorted paths, so path order is the natural default |
| Total match count before capping | `glob` | partial | `glob` returns the whole `Vec`, so the count is `len()`. The gap is that a cap must be applied *after* collecting, so a huge tree allocates the whole vector first |
| Cap the returned entries | none | missing | The Pack must slice. There is no limit parameter on `glob` |
| Cap the rendered characters | none | missing | The Pack owns rendering, so this is Pack logic, not a VFS gap |
| Honor `.gitignore` | none | missing | No ignore concept anywhere in the VFS. Requires either a new capability or a Pack-level walk |
| Honor `.cursorignore` or a Runvil equivalent | none | missing | Same. And a place to store which files are ignore sources |
| Exclude `.git` unconditionally | none | missing | A walk with a filter, or a pattern that excludes it, which the current grammar cannot express since it has no negation |
| Include dotfiles | `glob` | covered | Presumably the backend does not filter hidden entries; this needs verifying per backend |
| Follow symlinks | `glob` | partial | `FileType::Symlink` exists and `read_link` is optional, so a backend *can* follow, but nothing specifies whether `glob` descends into a link |
| Skip symlinks pointing outside the root | `glob` | partial | `OutsideRoot` exists as an error, but a walk that follows links needs to apply it per entry rather than per call |
| Detect a pattern with no matches | `glob` | covered | Empty `Vec` |
| Reject an empty pattern | `glob` | covered | `PathReason::Empty`. Note this is the right layer, see Trap 8 |
| Reject a malformed pattern | `glob` | covered | `PathReason::Wildcard`, `Control`, `Backslash`, `TooLong` |
| Report the reason a pattern was refused | `glob` | covered | `VfsError::InvalidPath { reason: PathReason }` and the `Display` impl names the rule |
| Case-insensitive pattern matching | `glob` | partial | Depends entirely on the backend. `runvil-internal/vfs/src/glob.rs` compares bytes, so matching is case-**sensitive** today |
| Brace expansion `{a,b}` | `glob` | missing | The grammar in `glob.rs` has four tokens: `Literal`, `Star`, `DoubleStar`, `DoubleStarSlash`. No alternation |
| Character classes `[abc]` | `glob` | missing | Same grammar. `[` is a literal byte |
| Negation `!` in a pattern | `glob` | missing | No negation token. Cursor has none at the tool surface either, so this is not a gap against Cursor, only against ripgrep |
| Distinguish "absent" from "hidden by policy" | `glob` | partial | `PermissionDenied` exists for a whole-call refusal, but a filtered-out entry inside a result set has no channel. The Pack cannot tell the model that N paths were suppressed |
| Read a file to test binary-ness | not needed | covered | Not needed: no content is read |
| Time out a slow walk | none | missing | `Access::glob` is synchronous with no deadline. A hostile tree blocks the calling thread |
| Report the walk's partial results on timeout | none | missing | Same |
| Report an error without aborting the whole call | none | partial | `VfsError` has no "partial with errors" shape. A backend that hits one unreadable directory must either swallow it or fail the call |
| Wire name `Glob` | Pack | covered | `schemas/glob.json` has `"wire_name": "Glob"` |
| Description and schema | Pack | partial | `glob.json` carries Cursor's edited text. It says `the run's filesystem root` instead of `Cursor workspace root`, which is right, but it drops the three worked examples that train models to write `**/` patterns |
| Per-model-family name variants | Pack | missing | `glob.json` has one `wire_name`. No `glob_file_search` branch |
| Structured error text the model can read | Pack | partial | `ToolError` exists; the exact strings above are not implemented |
| Truncation notice | Pack | missing | Stub returns `NOT_IMPLEMENTED` |

Three gaps deserve a paragraph each, because they are not mechanical.

**The mtime gap is the one that decides the tool's shape.** `Access::glob` returns `Vec<String>`. To sort by mtime you need `stat` per match, which is N calls each taking a claim, each firing an event, each potentially failing. For 2000 matches that is 2000 gates, 2000 claim admissions, 2000 events, and 2000 backend round trips, in a synchronous single-threaded runtime. That is materially more expensive than Cursor's `Promise.all` over 2000 stats, because Cursor's are concurrent and yours would be serial and instrumented.

So the design question is not "can I sort by mtime" but "do I want to pay N instrumented stats for a sort." My answer: no, not by default. Return path order, and add a `stat_many` or a `glob_with_stat` capability only if recency sorting turns out to matter. Confidence: medium-high, because the cost is structural and the benefit is speculative.

**The ignore gap cannot be closed inside the tool.** Runvil's VFS has no ignore concept, and the Pack is not the right place to invent one, because Grep needs the same rules and the two would drift. If you want ignore behavior, it belongs either in a new `Access` capability (something like `glob_filtered(pattern, filters)`) or in a Pack-level walk that owns the rule parser. The VFS-level option is better because the claim and the policy gate then cover the whole walk, but it is a bigger change.

Worth noting the precedent: Cursor's ignore handling is three mechanisms in three places - `--cursor-ignore` in the executor's argv, `!**/.git` as a literal glob, and a post-filter in the tool. That is why its precedence is inconsistent. Do not repeat it. One place, one precedence, documented.

**The case-sensitivity gap is a specification bug, not a feature gap.** `glob.rs` matches bytes, so `*.js` does not match `B.JS`. Cursor matches case-insensitively. If you keep `glob.rs` as the matcher and keep Cursor's surface, the same call produces different results. Three ways out: make `glob.rs` case-fold (cheap for ASCII, expensive and locale-dependent for Unicode), have the Pack rewrite patterns into a case-tolerant form the grammar can express (impossible without character classes), or accept the divergence and document it. I would accept it and document it, because case-insensitive path matching is a surprising default and Cursor's choice there is arguably a mistake. Confidence: medium, because model familiarity is a real cost.

## What remains unknowable

Things the evidence cannot reach, stated rather than guessed.

**Whether the indexed path ever serves a Glob call.** `executeIndexedGrep` is tried before the ripgrep path and falls back when the provider returns undefined. Whether the provider answers a `files_with_matches` call with an empty pattern is a runtime property of the extension host, not visible in the bundle. If it does, in-workspace Glob results may come from an index rather than a walk, which would mean they respect index ignore rules, may include unsaved buffers, and may be stale by up to the index's refresh interval. Cursor's own staff statement that Instant Grep "was not changed by this update" implies the indexed path can serve and therefore may still have the pre-fix behavior. This is the single largest unresolved question in the report.

**What `useStreamingGrep` defaults to in a real session.** The flag is injected at the resource registration site, `streamingEnabled` returns `true` when the provider is undefined, but the actual configuration value is not in the bundle. It matters because the entire mtime sort, the `--line-buffered` flag, and the `sortsAfterRipgrep` decision all branch on it. If streaming is off, the non-streaming path passes `--sortr modified` to ripgrep and the sort is unconditional and done by ripgrep.

**How realpath behaves on Windows junctions versus symlinks.** `en()` is a realpath wrapper whose failure produces the long "could not be verified" message. Junction handling is not distinguishable from the bundle, and a Windows junction inside a repository is common enough (package managers create them) that the behavior matters.

**Whether `.cursorignore` files are discovered from nested directories or only the root.** `listCursorIgnoreFilesByRoot` returns a list per root, but where it searches and how it merges nested files is in the ignore service, which is not in this bundle. The precedence documentation says later files win, so ordering within that list matters and is not determinable here.

**The byte-level behavior of the ignore service's own glob parsing.** Whether it uses ripgrep's gitignore parser, a JavaScript one, or something else determines whether edge cases like trailing-whitespace escaping and `**` in the middle of a segment behave identically. Not visible.

**Whether a filename with a newline actually splits in the shipped parser.** Derived from the parser's shape, not measured, because NTFS refuses to create such a file and I did not have a POSIX filesystem available. The parser scans to `\n` and takes the prefix, so the split is near-certain, but it is derived.

**Whether ripgrep's walk order is stable for a given tree on a given machine.** My probe produced different orders across invocations, which suggests not, but ripgrep's parallel walker could be stable per-machine at a given thread count. Either way the order is not specified, which is enough to say it is not safe to depend on.

**What the Smart Mode classifier actually does with the static `Glob` schema.** The short description without the batch bullet is projected into a static array at bundle offset `4673861`, but how the classifier weights it is not in this file.

**How a model's training actually represents `Glob`.** The description says `glob_pattern` and `target_directory`, but models were also trained on Claude Code's `pattern` and `path`, and on the `dsv3-1018` `glob_file_search` variant. Which name a given model reaches for first is an empirical question no static read answers, and it is the one input to Decision 1 that cannot be resolved from the artifacts. If it matters, run the same task against several models and count the parameter errors.

## Conformance checklist

The parent spec's twenty items for Glob, from `## Conformance checklist` at line 4780, each marked confirmed or corrected against the shipped 3.27.12 build.

| # | Item | Verdict | Reason |
|---|---|---|---|
| 1 | Parameters are exactly `glob_pattern` and `target_directory`, with no `pattern`, `case_sensitive`, or `max_results` | confirmed | Both schema variants have exactly those two fields, at offsets `4289658` and `4290208` |
| 2 | `dsv3-1018` describes the target as `Cursor workspace roots` in the plural plus a trailing newline, while every other branch uses the singular | confirmed | The plural is in `MEe`'s target description; the trailing newline is in `MEe`'s `glob_pattern` description, not the target description. The singular/plural half is exact; the newline lands on the other field |
| 3 | The tool is named `glob_file_search` only on `dsv3-1018` and `Glob` everywhere else | confirmed | `"dsv3-1018"===t?"glob_file_search":"Glob"`, with a throwing `default` arm |
| 4 | The description is byte-identical across all seven branches | confirmed | One `descriptionGenerator` ignoring its argument; the only other copy is the shorter Smart Mode projection, which the main agent never sees |
| 5 | No glob library is used: the search is ripgrep in listing mode with an empty pattern | confirmed | `isGlobFileListing` requires mode `files_with_matches`, `pattern === ""`, and a truthy glob; that selects `--files`. No `fast-glob`, `picomatch`, or `minimatch` on this path. Note `pi_find` *does* use `minimatch` - a different tool |
| 6 | Backslashes in the pattern are rewritten to forward slashes, one leading and one trailing slash are removed | confirmed | `normalizeGlob`, four rewrites in that order, identical in bundle and daemon |
| 7 | The recursive prefix is added exactly once when absent, and a pattern normalizing to just that prefix is rejected | confirmed | `!result.startsWith("**/")` guards the prefix; the equality test `"**/"===normalized` rejects |
| 8 | A whitespace-only pattern is not rejected and matches nothing instead | confirmed | Probed: `**/ ` passed to `--iglob` exits 1 with no output. The guard is equality, not trim-then-test |
| 9 | An absolute pattern becomes a recursive-prefixed suffix match rather than an absolute match | confirmed | Probed: `**/C:/work/proj/*.ts` returns nothing. `/home/dev/proj/src/*.ts` becomes `**/home/dev/proj/src/*.ts` |
| 10 | The glob flag is the case-insensitive form, so pattern matching ignores case | confirmed | `--iglob`; probed `**/*.js` matching `B.JS`. `--case-sensitive` is also passed and governs only the content regex |
| 11 | The `.git` exclusion is passed as its own negated glob pair rather than as an ignore rule | confirmed | `o && s.push("--iglob","!**/.git")`, gated on the listing-mode flag. Probed: removing it returns the whole `.git` tree |
| 12 | Hidden files are included and symlinks are followed on every call | confirmed | `--hidden` and `--follow` pushed unconditionally, after `--no-config --color=never` |
| 13 | No sort flag is emitted and sorting happens in JavaScript afterward | **corrected** | True only on the streaming path. `sortsAfterRipgrep` returns true when `sort` is undefined and streaming is on, which yields `--line-buffered` and no sort flag. The non-streaming path emits `--sortr modified`. Which path serves a call depends on `useStreamingGrep`, whose default is not in the bundle |
| 14 | `.gitignore` is respected and the no-ignore flag is never emitted | **corrected** | `--no-ignore` is indeed never emitted, and `.gitignore` does apply - but a positive `--iglob` overrides file-level ignore rules, per ripgrep's documented "always overrides any other ignore logic," and `**/*` even resurrects directory-level ignored subtrees by matching the directory node. Also, `.gitignore` only applies inside a git repository, since `--no-require-git` is not passed |
| 15 | Cursor ignore rules are applied through dedicated pairs plus a post-filter, not through ripgrep's ignore machinery | **corrected** | The dedicated pairs are real, but they *are* ripgrep's ignore machinery: `--cursor-ignore` is a patched flag documented as behaving "like `--ignore-file` except for its higher precedence," and it beats `-g`/`--glob` where `.gitignore` loses. The post-filter exists and covers permissions and repo blocks, but the main ignore path is in the engine |
| 16 | Results sort newest first by mtime under a 2000 ms stat budget, ties break by path segments ascending, missing mtime sorts last | confirmed | `sortNewestFirst` / `orderByModifiedDesc`. `SORT_STAT_TIMEOUT_MS = 2e3` in `../local-exec/dist/newest-first.js`. The comparator returns 1 for `a.mtime === undefined` and -1 when only `b` is undefined. Note the tie-break splits on `path.sep`, which differs per platform |
| 17 | At most 2000 files are returned and at most 10000 stdout lines are parsed, with an 8 MB stdout cap and a 25 s wall clock | confirmed | `CLIENT_LIMIT_LINES = 2e3`, `HARD_MAX_OUTPUT_LINES = 1e4`, `const c=8388608` in `executeRg`, `MAIN_TIMEOUT_MS = 25e3` |
| 18 | The header omits the total count when ripgrep was truncated, and pluralizes as `file` for 1 and `files` otherwise including 0 | confirmed | `success2.ripgrepTruncated !== true` guards the count; `${totalFiles} file${totalFiles === 1 ? "" : "s"}`. The zero case is a separate branch that never reaches the plural |
| 19 | The leading dot-slash is stripped only when the display path is exactly a single dot, so a Windows dot-backslash survives | confirmed | `normalizedRoot === "."` gates the map; the strip test is `startsWith("./")`. Probed: raw Windows output is `.\a.js`, which the test cannot match |
| 20 | The mid-list cut uses `at least` only when ripgrep was truncated, and the trailing banner fires only when no cut happened | confirmed | Two branches in the loop, and the banner is gated on `listedAll && resultFiles.length > 0 && (ripgrepTruncated || clientTruncated)` |

Sixteen confirmed, four corrected. The four corrections are all in one cluster - ignore semantics and the sort - and three of the four are consequences of the same thing: the parent spec read the argv and the executor's branch structure but did not run the binary, so it inferred `-g`/`--iglob` precedence from the shape of the code rather than from ripgrep's documented and probed behavior.

The one correction outside that cluster, item 13, is a genuine conditional in the code that the spec flattened into an absolute: there are two argv shapes and which one runs is a configuration value not present in the bundle.

## Sources

### Bundle slices

All offsets are into `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js`, 9,814,287 bytes, read as latin1, from Cursor 3.27.12.

| Offset | What |
|---|---|
| `4291000..4295600` | The whole Glob registration: `normalizeGlob`, the empty guard, `GrepArgs` construction, the retry, `mapGrepResultToGlobToolResult`, and `formatFilesForGlob` |
| `4292676` | `AG("GLOB"` registration (was `4277336` in the parent spec's build) |
| `4292721` | `descriptionGenerator` string |
| `4293469` | `glob_pattern is empty. Pass "*" to match every file.` |
| `4294836` | `glob_file_search didn't return the result` |
| `4295032` and `4295086` | `Result of search in` |
| `4295145` | `(total ${` |
| `4289658` | `REe` schema, `target_directory` describe |
| `4290208` | `MEe` schema |
| `4289795`, `4290337` | `The glob pattern to match files against` in both variants |
| `4286928` | `received an empty pattern` |
| `4289378` | `glob.empty_pattern_fallback` |
| `4290699` | `const JEe=5e3`, the render budget (`TOTAL_CHARACTER_BUDGET` in the daemon) |
| `4673861` | Static Smart Mode JSON Schema projection |
| `4562769` | `pi_find` registration |
| `4562863` | `Search for files by glob pattern using pi-compatible find semantics.` (`bundle`) |

Executor, same file:

| Offset | What |
|---|---|
| `9157500..9159900` | `sortsAfterRipgrep`, `buildArgs`, `searchTarget`, `resolveSearchTargets`, `isRootIndependentSearchPath` |
| `9158354` | `async buildArgs(` |
| `9159299`, `9159327` | `--iglob` pushes |
| `9159337` | `!**/.git` |
| `9158239` | `sortsAfterRipgrep` definition |
| `9160700..9163600` | `orderNewestFirst`, `keepNewestPage`, `runFilesMode`, `runCountMode`, `runContentMode` start |
| `9160913` | `newest_first.stat_timeout_fallback` |
| `9161205` | `runFilesMode` |
| `9161510` | `HARD_MAX_OUTPUT_LINES` |
| `9161535` | `CLIENT_LIMIT_LINES` |
| `9166975` | `Timed out after` |
| `9167279` | `Grep was cancelled` |
| `9165500..9167600` | `sortsAfterRipgrep`, `streamingEnabled`, `executeIndexedGrepWithTimeout`, `ripgrepTimeoutError`, `executeRg` start |
| `9070300..9070900` | `oS` comparator and `iS` stat-with-timeout |

### Daemon slices

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,625 bytes, original identifiers and embedded source paths. Used for names, not for exact strings.

| Offset | What |
|---|---|
| `12712190` | `function normalizeGlob(glob)` |
| `12712757` | `function formatFilesForGlob(result, searchRoot)` |
| `12715697` | `function mapGrepResultToGlobToolResult` |
| `12712000..12713000` | `normalizeGlob`, `truncationNotice`, `isEmptyPatternExecutorError` region |
| `12719000..12726000` | `createGlobTool`, both schemas, `description2`, `EMPTY_PATTERN_FALLBACK_REGEX`, `truncationNotice` |
| `16641900..16644100` | `statMtimeNs`, `comparePathSegments`, `orderByModifiedDesc`, `sortNewestFirst`, `SORT_STAT_TIMEOUT_MS = 2e3`, source path `../local-exec/dist/newest-first.js` |
| `16770300..16772400` | `isGlobFileListing`, `hasOnlyUnsearchableEntriesBelowTarget`, `parseFilesOutput` |
| `16771992` | `parseFilesOutput`, the one-line wrapper `parseRipgrepFilesOutput(stdout, clientLimitLines, ripgrepHardCuttoffLines)` that `runFilesMode` actually calls. The definition it forwards to is in a shared module at `16210282` |
| `16778800..16779200` | `GREP_CLIENT_LIMIT_LINES = 2e3`, `LocalGrepExecutor` class start |
| `16780000..16783000` | `searchTarget`, `sortsAfterRipgrep`, `buildArgs`, `filterBlockedFiles`, `throwOnRipgrepFailure`, `keepNewestPage`, `runFilesMode` |
| `16783988` | `newest_first.stat_timeout_fallback` |
| `16802800..16803100` | `HARD_MAX_OUTPUT_LINES`, `CLIENT_LIMIT_LINES` statics |
| `16844366` | `async function walk2` (the `pi_find` walk) |
| `16845048` | `LocalPiFindExecutor`, source path `../local-exec/dist/pi/find.js` |
| `16825942` | `truncateHead`, `PI_DEFAULT_MAX_LINES = 2e3`, `PI_DEFAULT_MAX_BYTES = 50*1024` |
| `13912670` | `findSchema` (`pattern`, `path`, `limit`) |
| `13896012` | `function renderPiFind`, `renderPiResult`, `extractResultText` |
| `17018419` | `LocalPiFindExecutor` resource registration |

### Scratch corpus

`cabinet\_scratch\cursor-tools\`:

- `tools\GLOB.js` - the prettified slice, header names `slice=[start,end]`. Read in full.
- `shared\raw-glob-ls.txt` - the shared Glob/ListDir region.
- `ripwalk-repro.js`, `ripwalk_15490200.txt`, `ripwalk_15492500.txt` - the ripwalk argv and its cache, used for the contrast in Q7.
- `globtest.js`, `dotcheck.js`, `lencheck.js` - earlier probes.
- `BRIEFING.md` - the identifier-mangling map.

`cabinet\_scratch\six-tool-reports\glob\` holds this report's own probe scripts and extracts. They are scratch and disposable.

### Probes

`(probe)` - run against `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\node_modules\@vscode\ripgrep\bin\rg.exe`, which reports `ripgrep 15.1.0-cursor5 (rev c3e3c2f7ec)`. Nine scripts; each built a temp tree, ran the exact Cursor argv, and printed exit status and output.

| Script | What it settled |
|---|---|
| `rgprobe.js` | Case-insensitive matching; binary and empty files included by `--files` and dropped by `-l`; gitignore inert outside a git repo; invalid-glob stderr text; whitespace-only pattern; absolute pattern; dotfile match; `.git` contents without the negation; subdirectory path form |
| `rgprobe2.js` | Confirmed `--no-require-git` and `--no-ignore` position sensitivity (my own argv bug, instructive) |
| `rgprobe3.js` | `--sortr modified` and `--sort modified` ordering; `--no-ignore` with correct flag position; bare `--files` honoring ignores |
| `rgprobe4.js` | File-level versus directory-level ignore, first pass |
| `rgprobe5.js` | Precedence matrix across `**/*`, `**/*.*`, `**/*.log`, `**/vendor/**`, `**/index.js` |
| `rgprobe6.js` | `--null` versus newline delimiters; invalid-UTF-8 filename creation refused by NTFS |
| `rgprobe7.js` | Git repo versus plain directory, eight glob shapes - the table in Trap 1 |
| `rgprobe8.js` | The directory-pruning mechanism proved with two ignored dirs, one dotted and one not |
| `rgprobe9.js` | File-level versus directory-level ignore with a nested tree, and `--cursor-ignore` directory awareness |
| `rgprobe10.js` | `--cursor-ignore` versus `.gitignore` precedence in one tree; `rg --help` text for both flags |

### Harnesses

`(source)`:

- Claude Code: `cabinet\_scratch\claude-code\tools\glob.js` and `limits.js`. Tool name `Glob`; search core `m6t`; notice builder `hGo`; registration `q0`.
- VS Code Copilot: `src\vscode\extensions\copilot\src\extension\tools\node\findFilesTool.tsx`, `findTestsFilesTool.tsx`, `test\findFiles.spec.tsx`, `node\toolUtils.ts` (`inputGlobToPattern`), `common\toolNames.ts` lines 25 and 46.
- Zed: `src\zed\crates\agent\src\tools\find_path_tool.rs`, with `crates\util\src\paths.rs` for `PathMatcher` (line 735) and `crates\worktree\src\worktree.rs` for `entries` (line 2835).

### Runvil

- `runvil\crates\runvil-internal\vfs\src\handle\operations.rs` - `Access::glob`, `glob_pattern`, and the full capability surface.
- `runvil\crates\runvil-internal\vfs\src\glob.rs` - the four-token grammar, `MAX_GLOB_PATTERN_BYTES = 1024`, and `matches_tokens`.
- `runvil\crates\runvil-internal\vfs\src\stat.rs` - `Stat.modified: Option<SystemTime>` and the doc comment about determinism.
- `runvil\crates\runvil-internal\vfs\src\error.rs` - `VfsError` and `PathReason`.
- `runvil\crates\runvil-internal\vfs\src\traits\backend.rs` - `VfsAccess::glob` and `glob_kind`.
- `runvil\crates\plugin-std\src\lib.rs` - the Pack, the wire-name table, and `NOT_IMPLEMENTED`.
- `runvil\crates\plugin-std\schemas\glob.json` and `schemas\EDITS.md` - the current definition and the recorded edits.

### Web

`(web)` - secondary to the bundle, used only for history and for the field report.

- [Glob does not work the way agents expect it to work](https://forum.cursor.com/t/glob-does-not-work-the-way-agents-expect-it-to-work/168477) - the October 2026 report that Glob ran `rg -l --iglob PATTERN` and therefore dropped binary and empty files, Cursor's staff confirmation, and the statement that the fix made Glob match "files by name, not by content." This is the single most valuable web source in the set, because it independently corroborates the `--files` versus `-l` fork found in `buildArgs`.
- [Cursor Agent tools documentation](https://cursor.com/docs/agent/tools/search) - describes Instant Grep and does not document Glob's parameters, sort order, or limits at all. Its absence is itself a finding: there is no authoritative public specification to conflict with.
- [Cursor Agent overview](https://cursor.com/docs/agent/overview) - generic tool descriptions, no per-tool schema.
- [Leaked Cursor Agent prompt 2.0](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools/blob/main/Cursor%20Prompts/Agent%20Prompt%202.0.txt) - contains the `glob_file_search` block matching the `MEe` schema verbatim, confirming that variant was in production with the plural `roots`.
- [Cross-harness glob survey](https://github.com/avifenesh/tools/blob/main/agent-knowledge/glob-impl-and-prompts-in-major-tools.md) - the source of the widely repeated but incorrect "100 results" figure for Cursor.

Where web and bundle disagree, the bundle wins, and every such conflict is recorded in the sections above rather than repeated here.

### Where the shipped build disagrees with the parent spec

Consolidated, because these are the findings that matter most.

1. **Positive globs override `.gitignore` at file level.** The spec says gitignore wins over a positive glob. ripgrep's `--help` says `-g`/`--iglob` "always overrides any other ignore logic," and the probe confirms a gitignored `*.log` is returned by `glob_pattern: "*.log"`. Spec offset `2417`, probe `rgprobe3.js` through `rgprobe7.js`.
2. **`**/*` resurrects directory-level ignored subtrees** by matching the directory node itself. Not stated in the spec at all. The spec's own recommended "match every file" pattern is the one that does it.
3. **`.gitignore` is inert outside a git repository.** `--no-require-git` is not passed, so a plain directory's `.gitignore` does nothing. The spec does not mention the git-repository precondition.
4. **`.cursorignore` is not "dedicated pairs plus a post-filter, not ripgrep's ignore machinery."** It is a patched `--cursor-ignore` flag that behaves "like `--ignore-file` except for its higher precedence," and it beats `--iglob`. The spec's description at `2444` inverts the mechanism, and the inversion hides the fact that the two ignore sources have different precedence.
5. **The sort flag is conditional, not absent.** The spec says "No sort flag." On the streaming path that is true; on the non-streaming path the executor emits `--sortr modified`. Which runs depends on `useStreamingGrep`, whose default is not in the bundle.
6. **The tie-break splits on the platform separator**, so a tie resolves differently on Windows and POSIX. The spec says "path segments ascending" without noting the platform dependence.
7. **`--line-buffered` is passed on the streaming path.** The spec's argv listing at `2372` omits it.
8. **The `dsv3-1018` trailing-newline difference is on `glob_pattern`, not `target_directory`.** The spec's conformance item 2 puts it on the target description, which has the singular/plural difference but not the newline.
9. **Offsets.** `GLOB` is at `4292676`, not `4277336`. The executor region is around `9158000..9174000`, not `9017686..9034500`.
10. **`pi_find` is a different engine, not a renamed Glob.** It uses `minimatch` over a JavaScript `readdir` walk with hardcoded `.git` and `node_modules` skips, and it reports `No files found matching pattern` rather than a header. The spec does not cover it.

*2026-10-27 - deepseek-v41-flash-1017*
