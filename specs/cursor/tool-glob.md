# Cursor's `Glob` tool: an implementation report

This report specifies one tool from Cursor's shipped agent bundle, in enough detail that you can build a different but compatible one. It is the Glob volume of a six-volume set; the other five cover Read, Write, StrReplace, Delete, and Grep. Everything here is derived from the installed Cursor 3.27.12 artifacts on this machine, from the unminified daemon twin, or from probes run against the shipped `rg.exe`. Where the parent spec at `runvil-design/specs/cursor/agent-file-tools.md` and the shipped build disagree, the shipped build wins and the disagreement is recorded.

Two offsets matter and both are given throughout. The parent spec was written against a 9,674,628 byte bundle; the installed bundle is 9,814,287 bytes. `GLOB` registration moved from `4277336` to `4292676`. The shared ripgrep executor moved further, from about `9017686` to `9158000`.

## The short version

- **Glob is not a glob library. It is ripgrep in `--files` mode with an empty pattern.** There is no `fast-glob`, no `picomatch`, no `minimatch` anywhere on this path (the separate `pi_find` tool in the same bundle does use `minimatch`, and pi's own `find` wraps `fd`; see `pi`). Matching is done by ripgrep's globset engine, ignoring is done by ripgrep's ignore machinery, and the walk is ripgrep's parallel directory traversal. Every surprising behavior below is a consequence of that one fact.
- **A positive glob defeats `.gitignore` at file level, which is the opposite of what the parent spec says.** `-g`/`--iglob` "always overrides any other ignore logic" per ripgrep's own `--help`. So `glob_pattern: "**/*.log"` returns a `*.log` file that `.gitignore` excludes. What saves the common case is directory pruning: an ignored *directory* is never descended into unless the glob matches the directory path itself, so `**/*.js` still returns nothing from a gitignored `vendor/`. The parent spec's claim that "`.gitignore` wins over a positive glob" is false for file-level rules and true only for directory-level ones. Rig reaches the same finding independently, through the `ignore` crate in process and without ripgrep: any glob that matches an ignored file, or an ignored directory's own node, brings it back (`*`, `**`, `**/*` and `*.txt` all did; probe).
- **`.cursorignore` does not have that hole.** Cursor added a custom ripgrep flag, `--cursor-ignore`, documented in the shipped `rg --help` as applying "with higher precedence than `-g`/`--glob` overrides. These rules can exclude files even when a glob would otherwise include them." Both ignore sources are therefore in force, but by two different mechanisms with different precedence.
- **The pattern is rewritten before ripgrep ever sees it.** Backslashes become forward slashes, one leading slash is stripped, one trailing slash is stripped, and `**/` is prepended if absent. Absolute patterns are therefore destroyed: `/home/dev/proj/*.ts` becomes `**/home/dev/proj/*.ts`, a suffix match that cannot succeed. `target_directory` is the only way to search an absolute location. Zed never matches an absolute pattern either, and zeroclaw drops every hit from one; only Claude Code, Copilot and pi's `fd` handle them.
- **Order is the replay problem, and no two harnesses do it the same way.** Cursor sorts newest-mtime-first in JavaScript after ripgrep returns, under a 2000 ms stat budget; past it the sort is abandoned silently and the order becomes arbitrary, so the description's "sorted by modification time" is conditional. Gemini sorts files modified in the last 24 hours first and the rest by `localeCompare`, so its order moves with the clock. OpenCode's docs promise an mtime sort and its code has none. Rig's order is the file system's. Zeroclaw cuts at 1000 in walk order before it sorts. pi's `fd` output is unordered. Claude Code's `--sort=modified` runs oldest first. Only Zed and Goose give a byte-stable total order, and that is the only kind that replays as is (see `### Ordering across the harnesses`).
- **The empty-pattern guard is `"**/" === normalized`, which catches `""` and nothing else.** A pattern of `" "` normalizes to `"**/ "` and is passed through as a literal, matching nothing. A pattern of `"/"` normalizes to `"**/"` and is rejected as if empty.
- **The cap that binds is the one nobody documents.** There are three caps: 2000 entries returned, 10000 stdout lines parsed, and 5000 rendered characters. On a normal repository the character budget binds first, at roughly 160 files, so a model that finds 400 matches sees about 160. The mid-list cut does announce itself with `... N more files ... (Do a more specific search if needed)`, so the truncation is visible. What is *not* visible is the total in the header, which can say `(total 5000 files)` while 2000 were retained and 160 shown.
- **The render fallback string is `glob_file_search didn't return the result` on every branch,** including the six where the tool is named `Glob`. `glob_file_search` is the `dsv3-1018` name only. This is reachable and model visible, and so fix it deliberately or copy it deliberately.
- **Case-insensitive matching is unconditional and comes from `--iglob`.** The argv also contains `--case-sensitive`, which reads like a contradiction and is not: `--case-sensitive` governs the *content* regex, `--iglob` governs the *pattern*. So on a case-sensitive filesystem, `*.JS` and `*.js` both match, which diverges from every shell glob a model has seen.
- **`.git` is excluded by a literal `--iglob !**/.git` pair.** Remove that pair and the entire `.git` directory contents come back, which the probe confirmed. There is no user-facing exclude parameter to add anything similar.
- **`--hidden` and `--follow` are unconditional.** Dotfiles are always included. Symlinks are always followed, so a link pointing outside the workspace is traversed unless a later permission filter catches it. There is a second ripgrep builder, `ripwalk`, which disagrees with this one on three defaults: it passes `--no-require-git`, it does not pass `--follow` unless asked, and its `caseSensitive` defaults to `false`. Glob never reaches it. Reading the wrong builder produces the opposite answer on symlinks, on `.gitignore` inside a non-git directory, and on case sensitivity.
- **Output is a header plus one `- path` line per file, as a single text block.** Never JSON. The header count is suppressed when ripgrep was truncated, because the count would then be a lie.
- **Paths are workspace-relative, and on Windows they keep a `.\` prefix.** The stripping test is `startsWith("./")`, which a backslash does not satisfy. So the platform most likely to be reimplemented keeps the prefix that the POSIX path drops.
- **`pi_find`, the parallel "pi-compatible" surface in the same bundle, is a completely different engine, and it is not pi's.** Upstream pi's `find` wraps the external `fd` binary (source). Cursor's port keeps pi's three parameters, the 1000 default and the notice strings, but swaps the engine for a JavaScript recursive `readdir` walk filtered by `minimatch` (bundle). The swap changes behavior against pi as well as against `Glob`: `limit` counts walked entries, not matches; a bare `*.ts` matches only the root; matching is case sensitive; and a bad glob returns `No files found matching pattern` (probe). It is a better structural model for a fresh implementation and a worse semantic one.
- **Nothing pages.** There is no offset, no cursor, no resume. A truncated result stays truncated, and the only recovery is to make the pattern more specific. Of the twelve harnesses compared, only Zed's `find_path` puts `offset` in the schema; Claude Code has it internally and always sends 0.

## Decisions you have to make

### Decision 1: Do you keep ripgrep, or do you write a matcher?

**Option A, keep ripgrep.** Spawn `rg --files` with the same argv. You inherit the globset matcher, the ignore machinery, the parallel walk, the `.git` handling, and the dotfile and symlink behavior. You also inherit the precedence bug, the `-l` versus `--files` distinction, the 8 MB stdout cap, and the fact that your tool's semantics are defined by a third-party binary whose version you now pin. Cost: a subprocess per call, a binary to ship on four platforms, and semantics you cannot modify without forking ripgrep. Gives: two days instead of two months, and a matcher that already handles brace expansion, character classes, and the full gitignore rule grammar.

**Option B, write a matcher over your VFS.** Vinnie's `Access::glob` already exists, already validates, gates, claims, fires an event, and delegates to a backend that owns its own matching. Cost: you must decide what the grammar is, and you will get it wrong in the corners that models actually use - `{a,b}` braces, `[a-z]` classes, and `**` in the middle of a segment. Gives: determinism, no subprocess, no binary, and a glob whose semantics you can specify in a sentence.

**Option C, hybrid.** Use the VFS walk for enumeration and a small in-process matcher for filtering, the way Cursor's `pi_find` port does. Upstream pi has the same split as a seam: `FindOperations.glob(pattern, cwd, { ignore, limit })` lets a host replace `fd` with its own walk, and the host owns matching and counting (source: pi `find.ts:52-57`). Rig's `Workspace::files(root, glob, listing)` is a third version of the split (source: rig `workspace.rs:147`).

Recommendation: Option B for Runvil, because the VFS is in-process and virtual, which means a subprocess cannot see it at all - ripgrep only works against a real filesystem. Where you need to match Cursor's *visible behavior* rather than its engine, note that a correct globset-compatible matcher over `**` and `*` covers over ninety percent of real calls, and the remaining cases (`{}` and `[]`) are cheap to add. Confidence: high, because the subprocess cannot reach the VFS, so the constraint decides itself.

**What the other harnesses add.** The other harnesses that match in process show that the grammar is the decision. Zed builds `globset` globs without `literal_separator`, so `*` crosses `/`; rig puts the `ignore` crate's override layer over `globset`, where `*` stops at `/` and a bare directory name matches nothing; zeroclaw's `glob` crate has no braces; Gemini uses the npm `glob` package. Option B stands, and what moved it is that the same pattern means different things in each. Pick the options deliberately and write them in the tool description. Confidence stays high.

There is a sub-decision here that you should make separately: **what does your pattern grammar accept?** Runvil's `validate_glob_pattern` currently refuses backslashes, control characters, more than two consecutive `*`, a `**` that does not occupy a whole segment, and anything over 1024 bytes. Cursor accepts all of those except the length limit, because it hands them to globset, which applies its own rules. If you plan to serve Cursor-trained models, refusing `**b` while Cursor accepts it is a divergence a model will hit. Confidence: medium, because the frequency of `**b` in real calls is unknown.

### Decision 2: Do you honor ignore files at all?

Runvil's VFS has no ignore concept. Cursor's Glob honors `.gitignore` and `.cursorignore`. That gap changes what the model can see.

**Option A, no ignore files.** Every file in the tree is visible to Glob. This is what the VFS does today. Cost: on a large repository most matches are noise. A `node_modules` that is gitignored becomes tens of thousands of candidate matches, which exceed every cap and still return noise. Gives: total determinism, no hidden state, and a tool whose output depends only on the tree and the pattern.

**Option B, honor a project ignore file.** Read `.gitignore` or a Runvil-specific equivalent and apply it during the walk. Cost: the tool's output now depends on files the model cannot see, which is the biggest trap in Cursor's design. Gives: the same practical behavior Cursor-trained models expect, and a usable result set on any real repository.

**Option C, honor ignore files but report them.** Same as B, except the rendered result mentions that N paths were excluded by ignore rules, so the model is never silently lied to.

Recommendation: Option C, and make the ignore source a property of the *Pack*, so every tool agrees. The reason C is worth the extra line of output is that Cursor's silence on this point is a documented source of model confusion - a model that knows a file exists and gets zero matches concludes the file is gone. Confidence: high, because the failure mode of B is already known from Cursor's own field reports.

If you pick B or C, you must also decide **precedence**, and this is where copying Cursor's argv without understanding it will bite you. In ripgrep, `--iglob` beats `.gitignore` for files it names, and `--cursor-ignore` beats `--iglob`. So a positive glob resurrects a gitignored file. Decide deliberately whether your ignore rules win over your pattern or the other way around. I would make ignore rules always win, because a user who writes `.gitignore` means it, and report the exclusion rather than silently dropping. Confidence: medium, because the opposite choice matches ripgrep and therefore matches what models have seen.

**What the other harnesses add.** Aider builds every listing from git's tracked-and-not-ignored set instead of the disk, which keeps secrets and build output out without a parameter, at the cost of hiding a new untracked file. Rig makes the choice a parameter of the backend, `Listing::{Visible, Hidden, All}`, and its description says ignored files are skipped while a glob can bring them back, which is the belief this decision warns about. Zed filters ignored entries in the snapshot but force-includes `.env*` and skips the privacy filters its sibling tools apply. Claude Code turns ignore off by default. Option C and the recommendation stand, with one change: Zed shows that the ignore rules and the privacy rules must live in `Access`, not in each tool, so that Glob and Read cannot disagree. Rig's independent probe moves the precedence fact from "found in one patched binary" to "how the engine works", which is why I now state it flatly.

### Decision 3: What do you sort by, and what happens when you cannot?

Cursor sorts newest-mtime-first, in JavaScript, after the search, under a 2000 ms budget, and falls back to walk order when the budget expires. That fallback makes the order non-deterministic.

**Option A, copy Cursor.** Sort by mtime descending with a 2000 ms budget, ties by path segments, missing mtime last, silent fallback. Cost: on a large result set or a slow disk, two identical runs produce different orders, which breaks replay. Gives: byte-identical output to Cursor in the common case.

**Option B, sort with no budget.** Stat every match, sort, done. Cost: unbounded time on a pathological tree. Gives: deterministic order every time.

**Option C, sort by path only.** Deterministic, cheap, needs no stat at all, and needs no mtime from your backend. Cost: diverges from Cursor's stated behavior and from what the description promises, so a model that says "find the file I just edited" no longer gets it first.

**Option D, sort by mtime when the backend supplies it, path otherwise, and say which.** Deterministic given a backend, honest about the fallback.

Recommendation: Option D. `Stat.modified` is `Option<SystemTime>`, and Runvil's own `stat.rs` says a backend fills an optional field only with a value it tracks, "so that a reported time stays deterministic and a sort by time, such as `ls -t`, stays meaningful." That is the right contract. Sort by mtime when present, break ties by path, sort purely by path when the whole set lacks mtime, and never mix the two orders within one result. Confidence: high, because the alternative silently breaks the replay property you care about and the mtime data may simply not exist.

The consequence to accept: an in-memory backend will return path order, a real backend will return newest-first, and the two will not agree. That is fine as long as it is documented, and it is much better than a sort that sometimes happens. If you want cross-backend agreement, the only answer is C.

**What the other harnesses add.** Three options, none of them yours yet. A total byte-wise path order, as Zed and Goose's `tree` do, is unconditional and is the only order a test pins. An engine-side mtime sort, as Claude Code does with `--sort=modified`, is unconditional but ascending, so under a cap it keeps the oldest files. Gemini's recency tier, files under 24 hours first and the rest by `localeCompare`, depends on the wall clock and the locale. Option D stands, with confidence still high. What moved it: Claude Code and Gemini show that even a sort by time has direction and clock choices that break replay, so state the direction and keep the clock out.

### Decision 4: What is your cap, and does it announce itself?

Cursor has three caps that interact: 2000 entries returned, 10000 stdout lines parsed, and 5000 rendered characters. The character cap binds first at roughly 160 files.

**Option A, one cap on entries.** Return at most N files, and say how many were dropped. Simple, predictable, and the units are unambiguous.

**Option B, one cap on rendered bytes.** Return as many files as fit in a byte budget, and say how many were dropped. This matches what matters for context cost, and it is what Cursor effectively does.

**Option C, both, like Cursor.** Cost: two caps means two distinct truncation messages and a class of bugs where the count in the header reflects one cap and the line count reflects another.

Recommendation: Option B, a single byte budget on the rendered list, plus a total count, plus an explicit "N more files" line. One cap, one message, one unit. The reason to prefer bytes over entries is that a path list's cost is its bytes, not its item count, and a directory of forty-character paths costs four times what a directory of ten-character paths costs. Confidence: medium-high, because a byte budget makes the truncation point depend on path length, which is less predictable for a caller but the right unit for a context window.

Whatever you choose, **announce it**. Cursor's mid-list cut does announce itself. Its `totalFiles` count does not always tell the truth, and that is the failure to avoid.

**What the other harnesses add.** A fourth option, **D, a cap plus `offset`**, as Zed does: 50 per page, a true total, and a range line. It turns a cut into a resumable call, and it is stable only over a total order, so it depends on Decision 3's path order. Two failures to avoid come from the others. Zeroclaw cuts at the cap in walk order before it sorts and prints a `Total` that is the shown count. Four harnesses (OpenCode, zeroclaw, rig, pi) fire the stop notice when the count reaches the cap rather than exceeds it, so an exact fit looks cut; ask the walker for one more than the cap and slice. I now recommend B with D added once the order is a total path order, and I put confidence at medium, because paging adds a parameter and a stable-order obligation that nothing else here needs.

### Decision 5: What does the header look like when the count is not known?

Cursor suppresses the count when ripgrep was truncated, because the number would be a lie, and prints `(total N files)` otherwise. That means the header has two shapes and a caller cannot parse it uniformly.

**Option A, always print a count.** Simple to parse. Cost: the count is a lower bound under truncation, so the header lies.

**Option B, never print a count; print the returned count and a separate total line.** Uniform, honest.

**Option C, copy Cursor's suppression.** Two shapes, honest, slightly harder to parse.

Recommendation: Option B. `Returned 40 of at least 2000 matches` is one line, tells the truth in both cases, and needs no conditional. Confidence: high for a new implementation, because the only reason to prefer C is byte-compatibility with Cursor, and you have already decided not to copy Cursor. Claude Code's `countIsComplete` flag and Zed's exact total show two honest shapes, and zeroclaw's `Total: N files`, which is the shown count, shows the dishonest one. That confirms B and changes nothing.

### Decision 6: Directory-only matching

Runvil's `Access::glob` treats a trailing `/` as "directories only." Cursor's Glob has no such concept and strips the trailing slash.

**Option A, adopt the trailing-slash convention in the tool.** Calls can then ask for directories. Cost: a model trained on Cursor will never send one, and a model that does send `src/` will be surprised when it means "directories named src" rather than "files under src."

**Option B, strip it like Cursor and expose no directory mode.** `glob_pattern: "src/"` becomes `**/src`, which matches both files and directories named `src`.

Recommendation: Option B at the tool surface, Option A retained in the VFS. The VFS-level capability is worth having, and the `Access::glob` signature already supports it. But the tool surface should match what models do, and models send `src/` meaning "under src", not "directories named src". Confidence: medium, because the model's intent for a trailing slash is ambiguous and Cursor's choice to strip is a coin flip that happens to be the common reading. Rig is a warning for any gitignore-like grammar: its glob goes to the `ignore` crate's override layer, where `src`, `src/` and `test*` return nothing at all because the glob matches the directory node and the files inside are tested on their own (probe). A grammar like that needs an explicit rewrite, as Cursor's `**/src` is. On the other side, Zed's `find_path` and pi's `fd` return directories from a glob, and Cline's legacy `list_files`, Codex's removed `list_dir` and Goose's `tree` list them, so models do ask for them. That supports exposing the VFS's trailing-slash mode as a parameter; the recommendation and the confidence stay as they are.

### Decision 7: Do you keep the mtime sort at all, or replace it with a recency signal you own?

The mtime sort exists to answer "which of these did I just touch." That is a legitimate need, and mtime is a poor signal for it: it changes under `git checkout`, it is not preserved across copies, and a build step touching a hundred files reorders the whole list.

**Option A, keep mtime.** Familiar, matches the description, needs no state.

**Option B, track recency in the harness.** The Runvil Pack knows which files its own Write and StrReplace calls touched this session. Sort by that first, then mtime, then path.

Recommendation: Option B as an addition, not a replacement, if you find models asking "the file I just created." Confidence: low, because the frequency of that pattern is unknown and it adds state to a tool that currently has none. It is the one place where a harness can beat Cursor, which has no cross-tool recency signal for Glob. Gemini's 24 hour tier is the nearest thing in the set, and it is a clock signal, not a session signal, so it breaks replay and does not answer "the file I just created". The recommendation and the confidence stand.

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

Three differences, all cosmetic. `Absolute` is present in `REe`'s target description and absent from `MEe`'s. `root.` singular versus `roots.` plural. `MEe`'s `glob_pattern` describe text ends with a newline. The `glob_pattern` describe text is otherwise byte-identical, and it contains **literal tab characters** before each example bullet. A reimplementation that normalizes those tabs produces a schema string that differs from Cursor's.

`U$.Ikc` is a plain `z.object`, so unknown keys are **stripped, not rejected**. The `Ikc:()=>ye` export resolves to `function ye(e,t){const r={type:"object",shape:e??{},...}` in the zod schema module, and none of the six tool schemas calls `.strict()`. The `Invalid arguments:` envelope therefore comes only from type and refine failures, never from an unrecognized key. Do not build a validator that rejects unknown keys if you want Cursor's behavior: a model that sends `pattern` instead of `glob_pattern` gets a missing-required-field error for `glob_pattern`, not an "unrecognized key" error naming the key it actually sent.

There is also a static JSON Schema projection at bundle offset `4673861`, used only by the Smart Mode classifier. It carries the shorter description with the batch-calling bullet removed, and its `required: ["glob_pattern"]` confirms the optionality.

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

The implementation honors the three claims in that text unevenly.

| Claim | Honored? |
|---|---|
| "Works fast with codebases of any size" | Mostly. The walk is parallel and the caps bound the output, but the 2000 ms stat budget is the point where a large codebase stops being fast, and the response is to give up on ordering rather than to give up on size. |
| "Returns matching file paths sorted by modification time" | Conditionally. Direction is newest-first, which the text does not say. Ties break by path. The sort is abandoned entirely on a stat timeout, and per-root ordering is not re-established when multiple workspace roots merge. |
| "Use this tool when you need to find files by name patterns" | True and useful. It is what distinguishes this from Grep. |

The text is derived from Claude Code's, which explains why the middle bullet appears verbatim in both: Cursor inherited the claim rather than deriving it. See `## How the harnesses do this`.

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

**The "100 results" figure that circulates in public documentation is not Cursor's number, and it is not one number.** In Claude Code it caps results (`globLimits.maxResults ?? 100`). In OpenCode, `const limit = 100` counts file paths for `glob` and matching lines for `grep` (source: `glob.ts:49` and `grep.ts:67` at v1.19.5), and because `truncated` is `files.length === limit`, exactly 100 results print a truncation notice even when the list is complete. In Gemini CLI the 100 is `DEFAULT_TOTAL_MAX_MATCHES` and belongs to `grep_search`, where it counts matches (source: `constants.ts:6`, `ripGrep.ts:530-535`). Gemini's `glob` has no result cap at all: `glob.ts` has no slice, limit or count, and the only cut is the scheduler's 64 KiB output rule (source: `tool-executor.ts:340-351`). The parent spec's attribution of the 100 to Gemini's glob (`agent-file-tools.md` line 2429) is wrong for that reason, and this report's earlier text repeated it. Cursor is 2000 entries and 10000 parsed lines, with a 5000-character render budget that usually binds first.

The public OpenCode docs describe a different tool than the code does. They say `glob` returns paths "sorted by modification time", but the code has no sort step: it passes `--glob=<pattern>` to `rg --files` with no `--sort` and no `-j1`, and three runs over 400 files gave three orders, so the first 100 of a larger set change between calls (source: `ripgrep.ts:160-168`; probe). And dot-directories are invisible to `glob` (its argv adds `--hidden` only when a caller asks, and the tool never does) but visible to `grep`, which does pass `--hidden`, so `*.yml` cannot find `.github/workflows/ci.yml` through `glob` while `grep` finds the same file (probe).

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

### The `pi_find` counterpart

The same bundle registers a parallel family of `pi_*` tools, and Glob's counterpart is `pi_find`. It is a reduced port of pi's own `find` tool, with a different engine from both pi's `find` and Cursor's `Glob`, so it is not a renamed Glob. Its full account, with pi's original beside it, is under `### pi, and Cursor's pi_find port` in `## How the harnesses do this`. In short: three parameters (`pattern`, `path`, `limit`), a 1000 default that counts walked entries and not matches, a recursive `readdir` walk filtered by `minimatch`, hardcoded `.git` and `node_modules` skips, no sort, bare relative paths, and a bracketed notice. (bundle, daemon, source)

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
7. Build a display-args record carrying the **raw** `target_directory` and `glob_pattern`, for the transcript. Note these are unnormalized, so the trace shows what the model sent.
8. **Call the shared executor**, which is the same executor Grep uses.
9. **Retry once on the empty-pattern error.** If the executor returned `error` and the message contains `received an empty pattern`, re-run with `pattern: "."`, and if that succeeds adopt its result. Increments `glob.empty_pattern_fallback` with an `outcome` label either way. Load bearing: this retry changes semantics, because `pattern: "."` breaks the `--files` condition and falls through to `-l --regexp .`, which is content mode and therefore drops empty and binary files. It exists for executors that reject the empty pattern. The shipped executor does not, so it never fires in practice, but a reimplementation that adds an empty-pattern guard to the executor must keep the fallback or fail.
10. **Classify errors.** `error` case goes through `classifyRipgrepError`, which substring-matches in this order. `timed out` maps to `TIMEOUT`. `is not allowed in a regex` or the empty-pattern test maps to `INVALID_ARGS`. `regex parse error` maps to an environment error with a rewritten message. Next, a list of known-good user-facing substrings maps to an environment error, and then the `os error` code set `{1,2,3,5,13,32,33}` maps to an environment error. Anything left becomes a plain `Error`.
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
18. **Spawn** with `cwd` at the root, a 25 s abort deadline, an 8 MB stdout cap, and a 10000 line parse cap. If the spawn fails with ENOENT, report `Path does not exist: ${cwd}` - the directory. That is a quirk: a missing `rg` presents as a missing directory.
19. **Check for failure.** `throwOnRipgrepFailure` throws only when stderr is non-empty *or* the exit code is 2, *and* stdout is blank, *and* the stderr is not entirely per-file access errors (`os error` 13, 5, 2, or 1) or a filesystem-loop message. So an unreadable subdirectory does not fail the whole search. Exit code 1 with blank stdout is **not** an error - it means no matches.
20. **Parse stdout.** `parseRipgrepFilesOutput(stdout, clientLimit, hardLimit)` trims, returns an empty result for empty input, counts every line to get `totalFiles`, retains the first `clientLimit` lines, and returns `clientTruncated = total > clientLimit` and `ripgrepTruncated = total >= hardLimit`. Line scanning strips a trailing `\r` before each `\n`, so CRLF output is handled. Note `ripgrepTruncated` uses `>=` while `clientTruncated` uses `>`, so exactly 10000 lines counts as truncated.
21. **Order newest first**, only on the streaming path. `orderNewestFirst` calls `sortNewestFirst`, which stats every path in parallel with `mtimeNs` bigint precision under a 2000 ms timeout, then sorts. On timeout it returns the input untouched with `statTimedOut: true`, and the caller logs `newest_first.stat_timeout_fallback` and increments a counter. Load bearing: the sort happens **before** the slice, so the 2000 kept entries are the 2000 newest.
22. **Slice to 2000.** `keepNewestPage` does `orderNewestFirst(...).slice(0, CLIENT_LIMIT_LINES)`.
23. **Post-filter.** `filterBlockedFiles` resolves each path and drops anything the permissions service blocks, that `isCursorIgnored` rejects, that `isRepoBlocked` rejects, or that a read-boundary blocker refuses. The drop count is subtracted from `totalFiles`. Load bearing: this runs **after** the sort and the slice, so dropping 1900 of 2000 leaves only 100 rendered despite a `totalFiles` that still reflects the larger number.
24. **Flatten across roots.** Concatenate, sum `totalFiles`, OR the truncation flags. Ordering holds **per root**, and no cross-root sort is re-applied.

### The second builder, and why the singular is wrong

Describing "Cursor's ripgrep invocation" in the singular is the easiest way to get three defaults wrong. `buildArgs` is one of at least **two** builders in the daemon, and they disagree on three defaults that matter:

| Default | `LocalGrepExecutor.buildArgs` (the one Glob uses, daemon 16780893) | `ripwalk` (a different helper Glob never reaches) |
|---|---|---|
| `--no-require-git` | never passed, so `.gitignore` is inert outside a git repository | always passed, third in the fixed prefix |
| `--follow` | **always** passed, so symlinks are followed | `followSymlinks = false` by default, `--follow` only when passed |
| `--case-sensitive` | always passed when `caseInsensitive !== true` | `caseSensitive = false` by default, `--case-sensitive` only when passed |
| sort | `--sortr <mode>` on the non-streaming path, nothing on the streaming path | no sort flag at all |

The `ripwalk` signature is `function ripwalk(ctx, { root, searchPaths = [], includeGlobs = [], excludeGlobs = [], caseSensitive = false, cursorIgnoreFiles = [], sandboxPolicy = { type: "insecure_none" }, noIgnoreVcs = false, followSymlinks = false, source = "ripwalk" })`, and its fixed prefix is `["--files","--hidden","--no-require-git","--no-config","--color=never"]`. It also prepends a `--threads` count from `getRipwalkThreadCount()`, which `buildArgs` never emits, and it caches results under a key built from every option.

Glob does not use `ripwalk`. It constructs a search request with `outputMode: "files_with_matches"` and dispatches to the shared executor. So every statement in this report about `.gitignore`, symlinks, and case sensitivity is a statement about `buildArgs`, and `ripwalk` has the opposite default on all three. A report that cites `ripwalk` to argue "symlinks are not followed" has read the wrong builder; a report that cites it to argue ".gitignore applies outside a git repository" has read the wrong builder in the other direction. Both conclusions are wrong for the path Glob takes. This subsection annotates step 17 and adds no step, so the numbering continues at 25 in the render layer below.

### Render layer

25. **Error case.** Return `truncateOutput(error, 5000, true).output`, which keeps a 2500-character head and a 2500-character tail joined by `\n\n... (output truncated) ...\n\n`.
26. **Absent case.** Return the literal `glob_file_search didn't return the result`.
27. **Zero files.** Return the zero template and stop. No banner, no count, no plural.
28. **Non-zero.** Compute the display path as `searchRoot.replace(/\/$/, "")`, and if the result is `.`, strip a leading `./` from each path. Otherwise leave the paths alone.
29. Emit the header. Suppress the count when `ripgrepTruncated`.
30. **Emit lines under the budget.** For each file, if `buffer.length + file.length > 5000`, emit the mid-list notice and set `listedAll = false`, else emit `- ${file}\n`. Load bearing: the comparison is `>`, not `>=`, and it compares against the accumulated buffer including the header, so the effective capacity depends on the header length.
31. **Emit the banner** only when `listedAll` is still true, at least one file was emitted, and a cap was hit.

The two truncation families never both fire for the same cut. The mid-list notice fires when the character budget cuts; the banner fires when a cap was hit but the character budget did not cut. Reaching the banner with `clientTruncated` set requires the post-filter to remove almost everything, which is rare but can happen.

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

Two of the first three comparison harnesses do it differently. Claude Code's Glob has an `offset` in its internal search signature (`{ limit: r, offset: s }`) and slices `Ie.slice(s, s + r)` for the simple case, but the tool's `call()` always passes `offset: 0`, so the capability exists and is unreachable from the model. Its truncation notice even says "Narrow the pattern or path to see the rest," which is the only recovery either harness offers. Zed's `find_path` does expose `offset` to the model, page by page at 50 results, and tells the model to pass `offset` for more.

The recovery mechanism Cursor offers instead is in the notice text itself:

```text
... N more files ... (Do a more specific search if needed)
```

and

```text
[Results truncated. N files in total. Narrow the search with a more specific target_directory or glob_pattern.]
```

Both point at the two parameters that do exist, and the whole strategy is to narrow the query. It works reasonably for a model, which can add a directory component on the next call, and it fails completely for a caller that wants an exhaustive list.

**What this means for a reimplementation.** If you add paging, you must decide what a page is *of*, and the answer is not obvious here because the result is sorted by a property the caller cannot sort on. A `limit` plus an `offset` over an mtime-sorted list is not stable across calls unless the mtimes are stable, and they are not. A path-sorted list pages stably, which is one more argument for path sorting in Decision 3.

There is no page-boundary-mid-line problem because there are no pages, and no mid-UTF-8 problem for the same reason. The character budget can cut between a header and its first file, but it cannot cut inside a path: the comparison is `buffer.length + file.length > budget`, so either the whole path line fits or it does not. A single path longer than 5000 characters would produce a mid-list notice with zero files emitted, which is a degenerate case the code does not special-case.

**Other harnesses.** Only Zed pages through the schema: `offset`, 50 per page, a range line that also prints on the last page. Claude Code has the machinery and sends 0. pi's trailer says `Use limit=2N for more`, which re-runs the search with a bigger cap and, because `fd`'s order is unspecified, may not return the same first N. Codex's removed `list_dir` paged by `offset` and `limit` over a sorted flat list. Gemini, OpenCode, rig, zeroclaw, Cline and Aider have none, and Cline's legacy listing is breadth first, so a cut still shows the top-level directories. (source)

### Q2. Units

Three different units, at three different layers, and they are not interchangeable.

| Cap | Unit | Exact comparison |
|---|---|---|
| 2000 entries | files | `totalFiles > CLIENT_LIMIT_LINES` for `clientTruncated`; `files.length < CLIENT_LIMIT_LINES` bounds the parse loop |
| 10000 lines | stdout lines | `totalFiles >= HARD_MAX_OUTPUT_LINES` for `ripgrepTruncated`, note `>=` not `>` |
| 5000 render characters | JavaScript string length | `accumulated.length + file.length > budget` |
| 8388608 stdout bytes | bytes | enforced by the process reader, kills the child past it |
| 25000 wall clock | milliseconds | `setTimeout` on an `AbortController` |

The render budget counts UTF-16 code units, because it is `String.prototype.length` on a JavaScript string. A path containing an astral character, or any emoji in a filename, counts as two units per code point. A byte-based reimplementation of the same budget will produce a different cut point on such a tree. That byte-level divergence shows up only on unusual filenames, so check for it before you chase a diff.

The entry cap counts *entries*, and in file-listing mode an entry is always a file, since ripgrep's `--files` emits files. Directories never appear. This is the opposite of `pi_find`, whose walk pushes directories too, with a trailing separator that `path.relative` drops before the model sees them, and whose limit therefore counts directories against the budget.

**Other harnesses.** Every cap counts something else. Claude Code counts results (100), Copilot results (20, or at least 200), Zed matches per page (50). pi counts `fd` results and UTF-8 bytes (1000 and 51,200), where its Cursor port counts walked entries. Rig counts file names (1000) and, for `list`, entries and `String::len` bytes. Zeroclaw counts accepted entries (1000) under a 50,000-byte loop cap. Gemini has no count cap and a 64 KiB scheduler cut in UTF-8 bytes. OpenCode counts file paths (100), Cline's legacy listing entries (200), Codex tokens at 4 bytes each. Cursor's 5000 render budget is UTF-16 code units, which no other harness uses. (source)

### Q3. Text encoding

The tool's own strings are ASCII. Filenames are not necessarily.

Ripgrep writes raw OS bytes for each path, one per line. The files-mode parser splits on newline and returns substrings. Unlike the content path, which calls `decodeLinesAsRead` and has a 2000 ms encoding budget, **files mode does not run any decoder over the paths**. So a filename containing bytes that are not valid UTF-8 arrives as whatever the string assembly produced from a UTF-8 decode with replacement, and the model sees a path it cannot open.

This is a limitation, and Cursor has no error for it. On a POSIX filesystem a byte like `0xFF` in a filename is legal and common enough in a repository that has been moved between systems. On Windows, where I tried to create such a file for the probe, NTFS refused the name outright, so the case is POSIX-only in practice.

**Newlines in filenames.** Ripgrep emits `--files` output newline-delimited by default, and the parser splits on `\n`. A filename containing a literal newline therefore splits into two bogus entries. The `-0` / `--null` machinery exists for the content path (`push("-0")` in content mode) but **Glob's files path does not use `--null`**. I could not construct such a file on Windows to prove the split, but the parser is unambiguous: it scans to `\n` and takes everything before it as the path. If you reimplement against a POSIX backend, either pass `--null` and split on NUL, or emit paths through an API that does not need delimiters at all. The VFS already does the latter, which is one reason to prefer it.

**Unicode normalization.** Nothing normalizes anything. macOS stores filenames in decomposed form (NFD) and Linux typically in composed form (NFC), so a pattern containing a precomposed `é` will fail to match a macOS filename stored as `e` plus a combining accent. globset compares bytes. A model that copies a path from one tool's output into another tool's `glob_pattern` is therefore at risk of a mismatch that is invisible on screen. Nothing in Cursor's chain is at fault here; it is a filesystem property. But it becomes your problem the moment you let the model round-trip a path through a pattern.

**BOM.** Irrelevant. The tool never reads file contents, so there is no BOM to preserve or strip.

**Other harnesses.** Zed's snapshot holds `RelPath`, valid Unicode by type, and the scanner skips a non-UTF-8 name without a trace; zeroclaw's `glob` crate skips them too. pi's `fd` output goes through `readline`, so such bytes become U+FFFD, and rig shows paths through `display()`, which also replaces bytes. OpenCode decodes stdout as UTF-8 and turns every backslash into `/`, so a POSIX name containing `\` is misreported. No harness normalizes Unicode, so the macOS NFD against NFC mismatch is open everywhere. (source, probe)

### Q4. Line endings

Ripgrep always emits `\n` between records on every platform, including Windows. The parser strips a trailing `\r` before each `\n` anyway, which matters only if something upstream re-emits CRLF. Both behaviors are correct and neither is visible to the model.

The one place line endings could leak is the rendered output. The templates use `\n` explicitly, so the model receives LF regardless of host platform. A reimplementation that uses Rust's `writeln!` on Windows would produce `\r\n` and diverge by one byte per line. Use explicit `\n`.

There is no CR-only handling, and no need for any: no record in this tool can contain a bare `\r`, except through a filename containing one, which falls under the newline-in-filename problem above.

**Other harnesses.** A newline in a name splits the entry in Cursor, Zed, pi (through `readline`) and Gemini's `\n`-joined list; only Claude Code, with `--null`, avoids it. pi strips a trailing `\r` and then trims each line, which also removes leading and trailing spaces that belong to a name. Zeroclaw prints the platform separator, so `\` on Windows. (source)

### Q5. Binary detection

**None.** In `--files` mode ripgrep never opens a file to inspect it, so it cannot classify anything as binary. A file full of NUL bytes is listed like a text file.

This was not always true. Before October 2026, Glob ran `rg -l --iglob PATTERN`, which is content mode: ripgrep reads each file to test the regex, and by default it skips files it classifies as binary and files it finds empty. The result was that a `*.py` glob over a directory containing `real.py`, `empty.py`, and a NUL-containing `binary.py` reported one file, while `dir` reported three. Cursor's staff confirmed it and shipped the fix.

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

**Glob has no line-length abbreviation.** Grep's content mode pushes `--max-columns 1000 --max-columns-preview`, which abbreviates any line over 1000 bytes. Glob never sees that abbreviation, because `buildArgs` pushes those two flags only inside its `mode === "content"` branch and Glob always calls it with `files_with_matches`. So a path list from Glob is never length-limited, and the only thing that bounds it is the 5,000-character render budget in the tool's own renderer. See `tool-grep.md` for what the abbreviation does to a matching line.

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

**Other harnesses.** An empty pattern is rejected by Cursor, becomes `*` in Zed, is an ordinary pattern in rig (with `files: true` and an empty pattern it lists every file), is rejected by Gemini with `The 'pattern' parameter cannot be empty.`, lists every file in OpenCode (up to 100 arbitrary ones), and matches nothing in pi's port. A whitespace-only pattern passes through as a literal in Cursor, Zed and rig, and Gemini rejects it. Gemini alone says how many files its ignore filter removed from an empty result. (source, probe)

### Q7. Cost

Worst case first, because that is what a hostile repository can force.

**Time.** The walk is bounded by a 25 s wall clock per root, and it is ripgrep's parallel walker, so on a warm cache it covers a very large tree quickly. The bound is per root, so a multi-root workspace costs 25 s per root worst case, sequentially. Cursor does run roots sequentially in the `for (const [root, result] of Object.entries(workspaceResults))` flatten, and the executor's own loop is sequential over roots too.

**A second, independent time cost sits after the walk.** Sorting by mtime requires one `stat` per returned path, issued in parallel via `Promise.all`, under a 2000 ms budget. On 2000 files that is 2000 concurrent stats, which will exhaust a typical file-descriptor limit or thread pool; Node's `fs.promises.stat` queues through libuv's threadpool, default size 4, so 2000 stats at 4-way concurrency is roughly 500 serial round trips. On a warm local disk that is well under a second. On a cold disk, a network mount, or a container with an overlay filesystem, it is easily over two seconds, and then the sort is abandoned wholesale. That is the mechanism behind the determinism problem in Q11.

**Memory.** The stdout cap is 8 MB, so the raw output is bounded. The parse retains up to 2000 paths plus a `totalFiles` counter that keeps counting to 10000 and then stops, so memory is bounded by 8 MB plus a bounded vector. There is no content buffering at all, because nothing is read.

**Worst case a hostile repository can force.** A tree with a very large number of files at shallow depth, all matching the pattern, plus a `.gitignore` that ignores nothing. Ripgrep walks until the 8 MB stdout cap or the 25 s deadline, whichever comes first. The parser stops at 10000 lines. The result is 2000 sorted entries and a `ripgrepTruncated` flag, and the render emits about 160 lines. So the *model-visible* cost is bounded at roughly 5000 characters no matter how hostile the tree. The *machine* cost is one full walk of the tree, every time, because there is no index and no cache on this path.

Cursor has two mitigations elsewhere that Glob does not use. There is a `ripwalk` client with a directory-listing cache (`CURSOR_RIPWALK_CACHE_TTL_MS`, default 20000 ms) used by other consumers - Glob's own path spawns a fresh process and does not consult it. And there is an indexed grep provider (`executeIndexedGrep`) that the executor tries first, which would avoid the walk entirely; whether it answers a `files_with_matches` call with an empty pattern is a runtime property of the extension host and not determinable from the bundle.

**For the VFS.** Your `Access::glob` delegates to the backend, so cost is the backend's problem. `RealBackend` walking from disk costs what ripgrep costs, without the parallelism. `MemoryBackend` iterates a map, which is much cheaper but still linear in the number of stored paths unless the backend indexes. A glob over a virtual filesystem should be linear in the number of nodes, and any backend that cannot promise that should say so.

**Other harnesses.** Cost depends on where the walk lives. Zed re-enumerates every worktree on every page and holds all matches in a `Vec` before slicing. Rig builds the whole path `Vec` before it caps, so a files-mode search that stops at 1000 still walked the whole tree. Zeroclaw canonicalizes every hit and walks the whole subtree when a pattern matches few files, with blocking `std::fs` inside an async function and no timeout. Gemini stats and `realpath`s every match before the 64 KiB cut. pi's `fd` stops at `--max-results`, but pi buffers all stdout lines until `fd` exits, and the port's per-entry `shouldBlockRead` runs sequentially. (source)

### Q8. Paths

This is where Glob differs most from the other five tools, and the difference is entirely in the pattern normalizer.

**The pattern is normalized.** Four rewrites, in order: every `\` becomes `/`; one leading `/` is stripped; one trailing `/` is stripped; `**/` is prepended if absent. Nothing resolves `..`, nothing resolves symlinks, nothing makes anything absolute. The normalizer is a string function, and it runs before the executor, before the root resolution, and before zod has done anything but type-check.

Tracing a Windows path through it:

| Input | After `\`→`/` | After leading strip | After trailing strip | After `**/` prefix |
|---|---|---|---|---|
| `src\**\*.ts` | `src/**/*.ts` | same | same | `**/src/**/*.ts` |
| `C:\work\proj\*.ts` | `C:/work/proj/*.ts` | same | same | `**/C:/work/proj/*.ts` |
| `\src\*.ts` | `/src/*.ts` | `src/*.ts` | same | `**/src/*.ts` |
| `src\` | `src/` | same | `src` | `**/src` |
| `\` | `/` | `""` | `""` | `**/` → rejected |
| `C:\` | `C:/` | same | `C:` | `**/C:` |

Two things follow. A relative pattern with backslashes is accepted and works, which is the good half. A Windows absolute pattern is destroyed the same way a POSIX one is: `C:/work/proj/*.ts` becomes `**/C:/work/proj/*.ts`, and since globset treats `:` as an ordinary character and no path segment sequence contains `C:`, it matches nothing. There is no error. The model gets the zero case.

**Output paths.** Ripgrep runs with `cwd` at the workspace root and the target passed as a relative positional, so it emits paths relative to that. On Windows those paths look like `.\src\file.ts` - backslash separated, with a leading `.\`. The renderer then strips a leading `./` only:

```js
"success" === e.result.case && e.result.value.path || "."
```

and

```js
r.files.map((e) => e.startsWith("./") ? e.slice(2) : e)
```

The strip applies only when the display path, after removing a trailing `/`, is `.`. `.\src\file.ts` does not start with `./`, so the prefix survives. My probe confirmed ripgrep's raw Windows output is `.\a.js` and that the strip test cannot match it:

```text
$ rg --files --case-sensitive --iglob "**/*.js" --iglob "!**/.git" --no-config --color=never --hidden --follow -- .
.\B.JS
.\a.js
```

So on Windows the model sees `- .\README.md`, and on POSIX it sees `- README.md`. That is a one-character difference in every line on the platform most people will reimplement against, and it is the most likely path-format bug in a clone.

**A supplied `target_directory` changes the reference point.** When `target_directory` is present, the paths are rewritten by `BEe` to be relative to the search directory rather than to the workspace root. The rewrite has three cases: target equals the root, target is inside the root, target is outside. The helper `OEe` guards each rewrite, rejecting a relative path that contains a `..` segment, a zero-length segment, or (on Windows) a drive colon. My probe confirmed raw ripgrep behavior, which the transform then adapts:

```text
$ rg --files --iglob "**/*.md" -- . ./docs
./docs\guide.md
```

**`..` in the target.** `searchTarget` resolves the target against the single workspace path, then tests containment. A `..` that escapes the root produces the absolute path, which is passed as the positional. Ripgrep walks it. So `target_directory: "../secrets"` searches outside the workspace, subject only to the later permission and cursorignore post-filter. That is an escape surface you should decide on deliberately: your VFS `OutsideRoot` error already closes it, and I would keep it closed.

**Symlinks.** `--follow` is unconditional. A link inside the workspace pointing outside it is traversed and its target's files are returned, because ripgrep sees a directory. A link loop produces `File system loop found` on stderr, which `hasOnlyUnsearchableEntriesBelowTarget` explicitly whitelists so it does not fail the search. A dangling link produces an `os error` line that is also whitelisted. If realpath itself fails during the pre-checks, the message is the long one naming access-denied and symlink loops.

**Case sensitivity.** The pattern is matched case-insensitively, always, because the flag is `--iglob`. The filesystem's own case sensitivity is irrelevant to pattern matching; it is relevant only to what ripgrep finds, and on a case-insensitive filesystem there is only one file to find. So `*.JS` matches `a.js` on both platforms, which diverges from shell globbing and from what most models expect.

**Drive letters and UNC.** `target_directory` accepts an absolute path, a `~` path, and a `file://` URL, per `isRootIndependentSearchPath`. A UNC path such as `\\server\share` becomes `//server/share` after the target-side normalization, and the `.startsWith("//")` case is not special-cased anywhere, so it likely resolves as a POSIX path on Windows. I did not test this, and it belongs in `## What remains unknowable`.

**Other harnesses.** Output paths are absolute in Zed, Gemini and OpenCode and relative in pi, rig and zeroclaw. Zeroclaw resolves symlinks and prints the target, so two links to one file print one line twice. Rig never lists symlinks and drops them without a trace, `fd` does not follow them, and the ripgrep command Cursor runs follows all of them. Case is insensitive on purpose in Cursor and Copilot, by default in Gemini, and sensitive in Zed, rig, zeroclaw and OpenCode, even on NTFS. Confinement: rig refuses a root outside its base only when the agent has a `WorkDir`; pi applies none; Zed searches only the project's worktrees, invisible ones included; Gemini validates only the search root and not the results. (source, probe)

### Q9. Concurrency

**Glob takes no lock at all.** It is not in the set of tools that a Read waits for before retrying a miss:

```js
["ApplyPatch","AwaitShell","CopyToBox","EditNotebook","MultiEdit",
 "MultiStrReplace","Shell","StrReplace","Write","edit_notebook",
 "run_terminal_cmd","search_replace","write"]
```

Every entry in that set mutates. Glob does not, so it is absent, and there is no per-path lock either - the per-file lock is keyed on a path, and Glob touches an unbounded set of paths that it does not know until the walk returns.

Two Glob calls in the same turn run concurrently and neither blocks the other, which is fine and desirable. A Glob concurrent with a Write can list a file that is being truncated, and the mtime it stats may be from before or after the write. A Glob concurrent with a Delete can list a file that no longer exists by the time the model tries to read it. None of these are errors; the tool has no mechanism to notice.

**The races it leaves open, named:**

| Race | Window | Result |
|---|---|---|
| glob versus write | between the walk and the stat | mtime may reflect pre-write or post-write content, so sort position is arbitrary |
| glob versus delete | between the walk and the return | a listed path may already be gone when the model reads it |
| glob versus rename | during the walk | the same file may appear under both names, or neither |
| glob versus directory creation | during the walk | a directory created after the walker passes its parent is missed entirely |

The last one is inherent to any single-pass walk and is not worth fixing. The first matters: if your backend supplies mtime, and your sort depends on it, then a Glob racing a Write produces a nondeterministic order. That is the same class of problem as the stat timeout in Q11, and the same answer applies - make the determinism property be about the tree.

**For the VFS.** Your `Access::glob` calls `admit(Claims::claim_glob, &claimed)`, so a glob does take a claim, keyed on the canonicalized pattern. That is strictly stronger than Cursor, and the interesting question is whether a pattern claim conflicts with a path write. If it does, Glob and Write serialize correctly, which is better than Cursor. If it does not, you have the same races as above but at least a claim record to reason about. Settle this before you write the tool, because the answer determines whether Glob can safely be called in the same batch as a Write.

**Other harnesses.** Zed takes one immutable snapshot per worktree in a single foreground update, so a call sees one point in time per worktree. Rig declares a footprint, `Reads(Paths([root]))`, which the kernel orders against overlapping writes in the same reply; its walk is not a snapshot, since content is read in batches after the walk. Zeroclaw and Gemini lock nothing and skip an entry that vanishes. Claude Code declares itself concurrency safe and read-only, as Cursor's lock-free treatment implies. (source)

### Q10. Fidelity

Glob never writes, so nothing can be corrupted by it.

**Nothing read is ever written back.** There is no read-modify-write, no temp file, no truncate, and no mode change. The only side effect on the filesystem is the `atime` update that the walk itself causes, which on most modern mounts is `relatime` and therefore usually not even that.

**Nothing in the tool changes file ordering for the next call.** Unlike a read-state store, Glob keeps no per-session memory of what it has seen. Claude Code's counterpart is concurrency-safe and read-only too, but Claude Code's harness has a separate `readFileState` LRU that Glob does not touch. So consecutive Glob calls are independent.

The one fidelity-adjacent behavior: the sort reads `mtimeNs` through `stat(path, {bigint: true})`, which is nanoseconds. A reimplementation that uses millisecond mtime will order files that differ by sub-millisecond timestamps differently. That is invisible in practice but it will break a byte-exact fixture recorded from Cursor. Use nanosecond precision if your `Stat` carries it, and note that Runvil's carries a `SystemTime`, which on most platforms is nanoseconds internally - so the precision is there, and only the platform's own granularity limits it.

### Q11. Determinism

A file listing that varies between identical runs makes an agent's behavior unreproducible, and every downstream property - replay, caching, regression testing, eval scoring - depends on it.

**The short answer: two identical Glob calls on the same unchanged tree do not reliably produce byte-identical output.** There are five independent sources of variation, and they compose.

**Source 1: the walk order is not defined, and it is parallel.** Ripgrep's `--files` output order is the order its parallel walker happens to emit. It is not alphabetical, it is not depth-first, and it is not stable across runs. That is a property of ripgrep's parallel walker and not of the `ignore` crate as such: rig drives the same crate through its single-threaded `Walk` (built with `build()`, not `build_parallel()`), gets the file system's directory order, and got the same list on repeated calls (probe on NTFS), though it never sorts. With `--sort` or `--sortr` ripgrep would sort, but Glob's streaming path passes neither:

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

**Byte-identical output across identical runs** holds on a warm cache, a local SSD, a tree small enough that 2000 stats finish in time, and an unchanged tree, because the JS sort is a total order. Outside those conditions it does not hold. The determinism guarantee is conditional on the stat budget, and the budget is not something the caller can observe or control.

**What a reimplementation should do.** Three concrete choices, in order of importance.

1. **Make the sort total and unconditional, or make it absent.** Never "sort if there's time." If the backend cannot supply mtime for a path, that path sorts last under a stable key.
2. **Report which order was used.** A one-word marker is enough. The model does not need it; a test harness does. Cursor's silent fallback is the mistake not to copy.
3. **Make the tie-break platform-independent.** Compare full path strings lexicographically by bytes, not by splitting on the platform separator. That gives one order on every platform and removes source 4.

Recommendation for Runvil: sort by mtime when every path in the result set carries `Some(mtime)`, tie-break by full path bytes, and when any path lacks mtime, sort the entire set by path. That is deterministic given the tree and the backend, which is the strongest property available. Confidence: high, because the alternative is a tool whose output cannot be replayed and whose tests cannot be golden.

**Other harnesses.** Ten orders, set out in `### Ordering across the harnesses`. Only a total order over unique keys with no clock, as in Zed and Goose's `tree`, replays without conditions. (source, probe)

### Q12. Truncation and omission

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

In practice the banner almost never appears, for arithmetic reasons: if `clientTruncated` is set then 2000 entries were retained, and 2000 lines always exceed 5000 characters, so the mid-list cut fires first and clears the flag the banner needs. The banner needs the post-filter to remove nearly all 2000, so it is rare.

**Where the tool is less than fully honest.**

The header prints `(total N files)` where `N` is the parsed total, and under `clientTruncated` that `N` can be 5000 while only 2000 were retained and only about 160 are shown. The number is a true total and a false impression of the list. The mid-list notice does say how many more there are, so a careful reader can reconcile them - but the header alone overstates what the list contains.

Under `ripgrepTruncated` the count is suppressed entirely, which is honest. So the header has two shapes and the model has to notice which it got.

**What is *not* announced:**

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

The tool announces what it cut for size and says nothing about what it cut for policy. A model that gets zero results for `*.log` because of a gitignore rule, or 40 results where 400 exist because 360 were cursorignored, has no way to distinguish that from the tree containing what it asked for.

**A reimplementation should fix the policy half of that table.** One line, appended when a policy filter removed anything:

```text
[12 paths excluded by ignore rules; 3 excluded by permissions.]
```

That is more useful than another truncation banner, because it distinguishes "not there" from "hidden from you." Confidence: high, because this is the confusion that produced the field report cited in the sources.

**Other harnesses.** Zed's total is exact and its range line says where the page ended, but the hint prints on the last page and an out-of-range `offset` answers `No matches found`. Claude Code is the only harness that tells the model whether a total is exact (`countIsComplete`). Gemini has no tool-level notice at all. Zeroclaw's `Total` is the shown count, and rig, pi, OpenCode and Cline's legacy tool fire their notice at an exact fit. On policy omissions the harnesses stay about as silent as Cursor: Gemini counts the files its ignore filter removed, and rig says nothing about skipped files. (source, probe)

## Failure matrix

Every failure the tool can produce, with the case it arrives as, the exact model-visible text, and the layer that produces it. Several failures present as *success* with a zero result, and those rows are marked.

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
| `pi_find`: search root blocked | error | `Find blocked: <reason>`, with reasons such as `Path is blocked by cursorignore` or `Blocked by permissions configuration` | permission service, `shouldBlockRead` |
| `pi_find`: search root is a file | error | `Not a directory: <abs path>` | tool |
| `pi_find`: search root missing | error | the raw Node text `ENOENT: no such file or directory, stat '<abs path>'`, because nothing checks existence first | Node, passed through |
| `pi_find`: pattern over 65,536 characters | error | `pattern is too long` | `minimatch`, caught by the tool |
| `pi_find`: invalid glob such as `[` | **success**, zero files | `No files found matching pattern` | tool, `minimatch` tolerates it |
| `pi_find`: walk budget spent before a match | **success**, zero files | `No files found matching pattern`, with no limit notice | tool, silent |
| `pi_find`: unreadable subdirectory or any throw | error | the Node error message, or `Unknown find error`; one failure fails the whole call | tool catch |
| `pi_find`: user cancels | not handled | the executor takes no abort signal, so the walk runs to its end | tool |

**"target_directory is a file" is not an error.** My probe confirmed ripgrep accepts a file as the positional and emits it if it matches the glob. So pointing Glob at a single file works, which is a mildly useful accident and something a reimplementation that insists on a directory would break. The pre-check `isDirectory` test does not exist on this path; the checks are existence, cursorignore, file-or-directory, and realpath.

**"subdirectory unreadable" is deliberately swallowed.** `hasOnlyUnsearchableEntriesBelowTarget` returns true when every stderr line matches either `rg: <path>: ... (os error 13|5|2|1)` or `rg: File system loop found: <path> points to an ancestor`, and the named path is not the target itself. In that case the failure is not thrown. So a repo with one unreadable directory returns results and says nothing, which is the right call for a search and the wrong call for a report to the model.

**"missing ripgrep binary" names the wrong thing.** `Fx(e)` returns `Path does not exist: ${e}` where `e` is the cwd. A model reading that concludes the directory is gone. The diagnostics gatherer records the real cause (`rgPath`, `rgPathExists`, parent listings, asar state) but only for telemetry. Do not reproduce this bug.

## Traps

### Trap 1: `.gitignore` does not beat a positive glob, and the parent spec says it does

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

That third case is the alarming one. `glob_pattern: "*"` - the very pattern the error message recommends for "match every file" - normalizes to `**/*`, matches the directory nodes, defeats the directory-level ignore, and drags in everything the user thought was excluded.

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

This is the claim most likely to be dismissed as an artifact of a bad probe. A probe run inside a subdirectory of a git repository will honor `.gitignore` even though the directory has no `.git` of its own, because ripgrep walks up to find the repository that contains it. So the case has to be tested somewhere with no enclosing repository at all. Verified that way on this machine, same two files, same argv, only the presence of `.git` differing:

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

Read the first and second blocks against each other. Outside a repo, `.gitignore` has no effect at all, not even on the plain `--files` call with no positive glob. Inside a repo, `.gitignore` suppresses `debug.log` on the plain call and the positive `--iglob` brings it straight back.

**A second harness found the same thing without ripgrep.** Rig runs the `ignore` crate (0.4.25) in process, and the override layer that a glob goes into has the highest precedence there (source: `ignore` `dir.rs:396-404`, "Overrides have the highest precedence"). In a git work tree, rig's walk returned files that `.gitignore` and `.ignore` had excluded for a glob such as `*.txt`. The globs `*`, `**` and `**/*` also brought back an ignored directory's whole subtree, because they match the directory node, while `build`, `build/`, `build/*`, `build/**` and `**/*.yml` returned nothing from that directory (probe, rig `rig-tools` at `a83b774d0`, `Workspace::files` in `workspace.rs:486-510`). That is the same three-way split as the probes above, from a different codebase and a different build of the engine. So the finding that a positive glob overrides `.gitignore` no longer rests on Cursor's patched `rg.exe` alone, and it contradicts the parent spec's opposite claim. Rig's tool description still says "Skips files ignored by git", which is the same mistaken belief the parent spec holds.

**Why this is easy to get wrong.** Three reasons stack. The docs say ignore files are respected, which is true of the default `rg` invocation and false of this one. The parent spec asserts the opposite of what ships, so a reader who trusts it builds the wrong precedence. And the common case - `node_modules` excluded, `**/*.ts` returns only source files - looks like ignore files working, because the extension glob does not match the directory. The bug only surfaces for a file-level rule whose extension the glob names, or for a catch-all pattern.

**The test that catches it.** A repo with `.gitignore` containing `*.log` and one `debug.log`, then `glob_pattern: "*.log"`. Cursor returns the file. If your implementation returns zero, you have chosen a different precedence, which is defensible if it is deliberate. Add a second case: `.gitignore` containing `vendor/` and a `vendor/x.js`, then `glob_pattern: "*"`. Cursor returns `vendor/x.js`. Decide whether you want that.

**What I recommend.** Make ignore rules always win over the pattern, and never let a pattern resurrect an ignored path. Then say so in the description, because a model that has learned the opposite from Cursor will be surprised. Confidence: medium, because the argument for matching Cursor is model familiarity and the argument against is that a user's `.gitignore` is a statement of intent.

### Trap 2: `.cursorignore` is a patched ripgrep flag, and it is not ordinary ignore precedence

It is tempting to model `.cursorignore` the same way as `.gitignore`, but it works differently. Cursor ships a forked ripgrep with a `--cursor-ignore` flag that does not exist upstream, documented in the shipped `--help`:

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

So the precedence chain, from strongest to weakest, is: `--cursor-ignore` rules, then `-g`/`--iglob`, then `.gitignore` / `.ignore` / `.rgignore` / global gitignore / `.git/info/exclude`. That is the reverse of what most people assume, and it is the reason a `.cursorignore` exclusion survives a catch-all glob while a `.gitignore` exclusion does not.

My probe confirmed the asymmetry in one tree: `.gitignore` = `*.log`, `.cursorignore` = `*.js`.

```text
"**/*"    -> .\.cursorignore | .\.gitignore | .\a.log | .\src\c.log     (js excluded)
"**/*.js" -> exit 1, nothing                                            (js still excluded)
"**/*.log"-> .\a.log | .\src\c.log                                      (log files returned)
```

**Why this is easy to get wrong.** If you model both ignore files as one list of gitignore rules, you get `.gitignore` right in the common case and `.cursorignore` wrong in the catch-all case, and both wrong in the file-level case. The two sources sit in different precedence tiers.

**The test that catches it.** One tree, both ignore files, three calls: a catch-all, a glob naming the `.gitignore`d extension, and a glob naming the `.cursorignore`d extension. Expect the first to return the gitignored file and not the cursorignored one, and the third to return nothing.

### Trap 3: `--case-sensitive` and `--iglob` in the same argv

The argv contains both flags, which reads like a bug but is not one:

```text
--case-sensitive
--iglob <pattern>
```

`--case-sensitive` governs the *content* regex, which is irrelevant in `--files` mode because there is no regex. `--iglob` governs the *pattern* and is documented as "Globs used via this flag are matched case insensitively." So the pattern is always case-insensitive and the `--case-sensitive` flag is a no-op on this path.

My probe confirms both directions:

```text
$ rg --files --iglob "**/*.js" ...       -> .\B.JS  and  .\a.js
$ rg --files --iglob "**/*.JS" ...       -> also matches a.js
```

**Why this is easy to get wrong.** Two ways. An implementer reading the argv may conclude the pattern is case-sensitive and build a case-sensitive matcher, which is wrong. Or an implementer may "clean up" the redundant flag and accidentally make the pattern case-sensitive, which is also wrong. The flag looks like a contradiction but is a statement about a different input.

**The test that catches it.** A tree with `a.js` and `B.JS`, pattern `*.JS`. Both must appear. On a case-sensitive filesystem this is unambiguous; on a case-insensitive one there is only one file and the test proves nothing, so run it on Linux or on a case-sensitive APFS volume.

**For Runvil.** Your `validate_glob_pattern` does not care about case, and `Access::glob` delegates matching to the backend. So the decision lands on the backend, and you should specify it: case-insensitive matching to match Cursor, or case-sensitive to match the platform. I would go case-insensitive at the tool layer, because a model that writes `*.JS` means "javascript files", and silently returning nothing is the worse failure. Confidence: medium-high.

### Trap 4: normalization destroys absolute patterns, and the description invites them

The `glob_pattern` description ends with three examples, the last of which is `"**/test/**/test_*.ts"`. A model reading that reasonably infers that patterns match full paths and may supply an absolute one. The normalizer then does this:

```text
/home/dev/proj/src/*.ts
  -> (strip leading slash)  home/dev/proj/src/*.ts
  -> (prepend **/)          **/home/dev/proj/src/*.ts
```

That is a suffix match requiring a directory literally named `home`, then `dev`, then `proj`, then `src`, relative to the workspace root. On a workspace at `/home/dev/proj` the relative path is `src/foo.ts`, which does not contain those segments at the end. Zero matches, no error.

The same happens to `C:/work/proj/*.ts`, and to `C:\work\proj\*.ts` after the backslash rewrite.

**Why this is easy to get wrong.** Two ways. An implementer may "fix" the normalizer by not prepending `**/` when the pattern looks absolute, which changes behavior for a pattern like `src/*.ts` that happens to have a leading component matching a root name. Or an implementer may decide the empty result is correct and never notice that a whole class of plausible model inputs is silently dead.

**The test that catches it.** A workspace with one file at `src/a.ts`. Call with `glob_pattern: "/src/*.ts"` - this works, because stripping the slash gives `src/*.ts` then `**/src/*.ts`, which matches. Then call with the absolute path to that same file. The first returns the file; the second returns zero. That pair pins both halves.

**What I recommend.** Detect an absolute pattern and either reject it with a message naming `target_directory`, or route it into `target_directory` for the caller. Both are better than a silent zero. Confidence: high, because a silent zero on a well-formed-looking input is the worst possible response.

### Trap 5: the sort is conditional, and the condition is invisible

Covered at length in Q11 and recorded here as a trap.

`sortNewestFirst` returns the input untouched on a 2000 ms timeout. The caller logs a metric and moves on. The renderer emits the list in whatever order arrived. Nothing in the model-visible text says the order is arbitrary.

**Why this is easy to get wrong.** A reimplementation that reads the description, sees "sorted by modification time", and implements an unconditional sort will be *more* correct than Cursor and will fail a byte-comparison test on a large tree. Conversely, a reimplementation that copies the budget and the silent fallback inherits a nondeterminism it probably does not want. The trap is that both are defensible and the choice is invisible in the output.

**The test that catches it.** Run the same Glob call twice on a tree with more than a few thousand matching files, with a cold page cache (`echo 3 > /proc/sys/vm/drop_caches` on Linux, or just a large tree on Windows). If the two outputs differ, the fallback fired. On a warm cache the test passes and proves nothing, which is why this one gets missed.

### Trap 6: the cap the model sees is not the cap in the constant table

Three caps are named in the source: 2000, 10000, and 8388608. The cap that binds on a normal repository is none of them; it is the 5000-character render budget, which is not a "limit" in the executor at all and sits in the render function.

Rough arithmetic: the header is about 30 characters, each line is `- ` plus the path plus `\n`, and a typical repo-relative path is 25 to 40 characters. So 5000 characters is about 150 to 190 files.

**Why this is easy to get wrong.** An implementer who reads `CLIENT_LIMIT_LINES = 2000` and builds a test with 300 matching files expects 300 lines and gets about 160 plus a notice. Then the test is "fixed" by raising the render budget, which changes the model-visible cost of every call. The caps are in three different layers and the tightest one is the least visible.

**The test that catches it.** 400 matching files with paths of about 30 characters. Expect roughly 160 `- ` lines, then `... 240 more files ... (Do a more specific search if needed)`. If you see 400 lines, you have no render budget. If you see 2000, you have the entry cap but not the character cap.

The budget compares `>` not `>=`, and it includes the header in the accumulated length, so the cut point shifts by the header's length. A byte-exact fixture must account for that.

### Trap 7: `./` is stripped, `.\` is not

The strip test is `startsWith("./")`, applied only when the display root is `.`:

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

**Why this is easy to get wrong.** The retry is a small function with a clear purpose ("handle executors that need a pattern"), and it looks like a compatibility shim with no semantic content, but it changes the output. An implementer who adds an empty-pattern guard to their own executor, then keeps the fallback for compatibility, gets the old bug back.

**The test that catches it.** A directory with `real.py`, `empty.py`, and a NUL-containing `binary.py`, and a stubbed executor that returns `received an empty pattern` for the first call. All three files must be listed after the retry, or the fallback is in content mode.

**For Runvil.** Do not have an empty-pattern guard in your matcher at all. Your `Access::glob` already rejects an empty pattern with `PathReason::Empty`, which is the right layer for that check - it means the tool must translate `glob_pattern: ""` into `PathReason::Empty` before calling, rather than calling and retrying. Confidence: high, because a retry that changes mode is a bug waiting to be reintroduced.

### Trap 9: the render fallback names the wrong tool

```js
if ("success" !== e.result.case) return "glob_file_search didn't return the result";
```

This fires on all seven branches, including the six where the tool is called `Glob`. `glob_file_search` exists only as the `dsv3-1018` name, so the string reads like dead code left behind by a rename. It is reachable: any executor result whose case is neither `success` nor `error`, and any flatten that finds no root with a `files` result, lands here.

**Why this is easy to get wrong.** It looks like dead code, so an implementer deletes it or replaces it with a string that names `Glob`, and then the branch that reaches it has no message at all. Or an implementer copies it verbatim into a tool that is never called `glob_file_search`, and a model reading the error will not understand it.

**The test that catches it.** Stub the executor to return a result with `case: undefined` and confirm the model sees `glob_file_search didn't return the result`. Then decide: reproduce it for byte compatibility, or fix it to name the actual tool. Either is fine; drifting into it by accident is not.

### Trap 10: `totalFiles` is decremented after the slice, so the header and the list disagree

Order in `runFilesMode`: parse, sort, slice to 2000, post-filter, and subtract the dropped count from `totalFiles`. So `totalFiles` is the post-filter count while the *slice* was taken pre-filter. If 1900 of the 2000 are cursorignored, the result is 100 files rendered under a header that says `(total 100 files)` - which is arithmetically consistent - but the *reason* only 100 survived is invisible, and the truncated-at-2000 fact is lost.

Worse, in the reverse direction: if the parse counted 5000 and the slice kept 2000 and the filter dropped 0, the header says `(total 5000 files)` and 2000 were candidates and about 160 are shown. Three different numbers, one of which appears in the output.

**Why this is easy to get wrong.** The decrement looks like bookkeeping hygiene. It changes the meaning of the field from "how many matched" to "how many survived policy," and the renderer treats it as the former.

**The test that catches it.** 2500 matching files, of which 2400 are cursorignored. The header and the `N more files` arithmetic must reconcile with the visible lines. In Cursor they do not, cleanly, and the mid-list notice will report a number larger than the list plus the dropped set.

### Trap 11: multi-root order is per root, not global

With no `target_directory`, every workspace root is searched and the results are concatenated in root order. Each root was sorted internally. No global sort is re-applied. So a file in root 2 that is newer than everything in root 1 still appears after all of root 1's files.

`totalFiles` sums across roots, and the truncation flags OR across roots. So a workspace with two roots where one is truncated and the other is not produces a single `ripgrepTruncated` flag that suppresses the header count for both.

**Why this is easy to get wrong.** The flatten function looks like a simple reduce and the per-root sort looks like it composed, but it does not.

**The test that catches it.** Two roots, distinct mtimes, one file per root with the newer file in root 2. The output must show root 1's file first if the order is per-root; a global sort would show root 2's first.

### Trap 12: a filename with a newline splits into two entries

The files-mode parser scans to `\n` and takes everything before it as one path. It does not pass `--null`, and the content path's `-0` flag is not used here. So a POSIX filename containing `\n` becomes two lines, and therefore two entries, neither of which exists.

I could not create such a file on Windows (NTFS refuses the name), so this is derived from the parser rather than measured. The parser is unambiguous, and ripgrep's documented behavior is newline-delimited output unless `--null` is given.

**Why this is easy to get wrong.** Nobody tests it, because creating the file requires `touch $'a\nb'` and most test harnesses cannot express it.

**The test that catches it.** On Linux: `touch $'weird\nname.ts'`, then `glob_pattern: "*.ts"`. Cursor returns two bogus paths. Your implementation should return one path, either by splitting on NUL or by never putting paths in a delimiter-joined stream.

**For Runvil.** Your `Access::glob` returns `Vec<String>`, so there is no delimiter and the problem does not exist. This is an advantage of the VFS design over the subprocess design.

### Trap 13: `--hidden` means the walk sees things the model should not

`--hidden` is unconditional, so `.env`, `.ssh`, `.git` (except for the explicit negation), `.venv`, and every other dotfile are candidates. There is no user-facing way to turn it off.

In practice `.env` is gitignored in most repos and therefore excluded by the directory rules, and `.git` is excluded by the literal glob. But a `.env` that is *not* gitignored, or a tree that is not a git repo at all (where `.gitignore` is inert, per Trap 1), puts `.env` in the result set for `glob_pattern: "*"`.

**Why this is easy to get wrong.** A reimplementation that skips hidden files by default is doing the safe thing and diverging. A reimplementation that follows `--hidden` and has no ignore layer exposes secrets.

**The test that catches it.** A non-git directory with `.env` and `glob_pattern: "*"`. Cursor returns `.env`. If your implementation does too, make sure your ignore layer is what protects the user, because `--hidden` will not.

### Trap 14: the description promises speed, and the mechanism that fails is the sort

"Works fast with codebases of any size" is true of the walk, which is parallel and capped. It is not true of the sort, which is a partly serial 2000-way stat burst under a 2 second deadline, and it is not true in the sense a user expects, because the walk happens on every call with no cache.

**Why this is easy to get wrong.** The sentence is about the walk and readers apply it to the whole tool. An implementer who benchmarks the walk and finds it fast concludes the tool is fast.

**The test that catches it.** Time a Glob call on a repo with 2000 matching files on a cold cache, and separately time the same call where stat is artificially slowed. The second should show the fallback firing and the total time pinned near the budget.

### Trap 15: a glob that names a directory can match nothing

Rig hands the glob to the `ignore` crate's override layer, where a pattern that matches a directory node does not select the files inside it: `src`, `src/` and `test*` return `No matches.`, while `src/**` returns the files (probe). A model that writes `src/` meaning "under src" gets a confident zero. **Why it is easy to get wrong:** the grammar looks like gitignore and models know gitignore, but the override semantics differ, and Decision 6's trailing-slash rule interacts with it. **The test that catches it:** a tree with `src/a.rs`, then `src`, `src/` and `test*` as patterns. Expect the files, or an error that names the rewrite; never a bare zero.

### Trap 16: cap, sort and flag must happen in that order

Zeroclaw breaks out of the walk at 1000 accepted entries, sorts only the survivors, sets the truncation flag when the count merely reaches 1000, and prints `Total` as the shown count (probe). OpenCode, rig and pi also fire their notice at an exact fit. **Why it is easy to get wrong:** each step looks right alone, and `results.len() >= MAX` reads like a bounds check. **The test that catches it:** 1001 files where the alphabetically first one is walked last, then 1000 files, then 999. Expect the first file in the 1001 case, a notice only for 1001, and a total of 1001.

### Trap 17: a limit that counts the walk turns a hit into a silent zero

Cursor's `pi_find` stops the walk at `limit` entries before the glob filter and returns `No files found matching pattern` before it builds the limit notice, so a file beyond the budget is reported absent (probe: 30 directories, `limit: 10`, target at the end). **Why it is easy to get wrong:** a limit on the walk is the cheapest way to bound cost, and the empty-result branch is written first. **The test that catches it:** a tree larger than the limit with the only match last. Expect the match, or a notice, but never the bare zero.

### Trap 18: an ascending mtime sort under a cap keeps the oldest files

Claude Code asks ripgrep for `--sort=modified`, which is ascending, and slices the first 100 (probe: oldest first with the shipped `rg.exe`; the slice is in `m6t`). **Why it is easy to get wrong:** "sort by modification time" does not say a direction, and ripgrep's flag name does not either. **The test that catches it:** three files dated one, two and three days back and a cap of 2. Expect the two newest.

### Trap 19: splicing the root into the pattern lets the root's own characters act as glob syntax

Zeroclaw joins the workspace path onto the pattern as a string, so a workspace directory named `ws[1]` is read as a character class and `*.txt` finds nothing (probe: `glob('<tmp>/ws[1]/*.txt')` gave 0 matches). **Why it is easy to get wrong:** `format!("{root}/{pattern}")` is the obvious way to build a rooted pattern. **The test that catches it:** a root directory containing `[`, `*` or `{`, with one file in it. Expect the file. In Runvil, hand the matcher the root and the pattern separately, which `Access::glob` already does.

### Trap 20: a glob tool that skips the privacy filter lists what Read refuses

Zed's `find_path` is in `EXCLUDED_TOOLS` and does not apply the `private_files` or `file_scan_exclusions` filters that `read_file`, `list_directory` and `grep` do, and its default `file_scan_inclusions` force-includes `.env*` even when gitignored, so `**/.env*` lists names Read will not open (derived; no test). **Why it is easy to get wrong:** a name listing looks harmless next to a read. **The test that catches it:** a path that Read refuses, then `*` and a pattern that names it. Expect the same outcome from Glob that Read gives, hidden or refused, and not a listing.

### Trap 21: a clock in the sort breaks replay without changing the tree

Gemini puts files modified within 24 hours of the wall clock first, so the same tree orders differently an hour later, once a file crosses the boundary (source: `glob.ts:47-70`, `:262`). **Why it is easy to get wrong:** recency is a good idea and the constant is easy to pick. **The test that catches it:** inject a clock, run the same call at 23 hours and at 25 hours after the newest write, and compare bytes. Expect them to be identical, or the tier named in the description.

## How the harnesses do this

Twelve harnesses are compared. Eight have a file-finding tool: Claude Code, Copilot, Zed, pi, rig, zeroclaw, Gemini CLI and OpenCode. Codex CLI, Cline, Aider and Goose have none, and each of those sections says what the harness does instead and what that implies. Cursor's own `Glob` is the baseline and is not repeated here. Each claim carries a tier label: `(source)` for code read on disk or at a pinned commit, `(bundle)` and `(daemon)` for the installed Cursor artifacts, `(web)` for documentation or code fetched from a URL, and `(probe)` for something run.

### Claude Code

Source: Claude Code 2.1.311 (win32-x64), the `glob.js` region extracted from `claude.exe` at the offsets in its header (bytes 256099961 to 256108320). Name: `Glob`. The registration object is `q0`, the search core is `m6t`, the truncation notice is `hGo`.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `pattern` | string | yes | `The glob pattern to match files against` |
| `path` | string | no | `The directory to search in. If not specified, the current working directory will be used. IMPORTANT: Omit this field to use the default directory. DO NOT enter "undefined" or "null" - simply omit it for the default behavior. Must be a valid directory path if provided.` |

The `path` description is unusual: it contains an explicit instruction not to send the literal string `"undefined"` or `"null"`, which means models were sending those strings often enough to warrant a schema-level warning. Cursor's schema has nothing equivalent. OpenCode's `path` description is the same text word for word, and Copilot's `path` refusal below answers the same habit.

The engine is ripgrep, but with a materially different argv:

```js
let ke = ["--files", "--null", "--glob", n, "--sort=modified",
  ...(q ? ["--no-ignore"] : []),
  ...(Q ? ["--hidden"] : [])];
for (let st of z) ke.push("--iglob", st);
```

Against Cursor's argv the differences are all load-bearing.

| Dimension | Cursor | Claude Code |
|---|---|---|
| Pattern flag | `--iglob` (case-insensitive) | `--glob` (case-sensitive) |
| Ignore files | respected | **disabled by default**, `CLAUDE_CODE_GLOB_NO_IGNORE` defaults `"true"` |
| Hidden files | always on | on, `CLAUDE_CODE_GLOB_HIDDEN` defaults `"true"` |
| Sort | in JS after the search, conditional | `--sort=modified`, in ripgrep, unconditional, and ascending, so oldest first (probe) |
| Record delimiter | newline | `--null`, split on NUL |
| Recursive prefix | always prepended | resolved from the pattern's own base directory |
| Cap | 2000 entries, 5000 chars | 100 results |
| Paging | none | `offset` exists internally, always 0 from `call()` |
| Output | header + `- path` lines | bare paths, one per line, no header |

The `--sort=modified` is the most interesting divergence. Claude Code asks ripgrep to sort during the walk, which is unconditional and cannot time out. Cursor sorts afterward in JavaScript, which is where the timeout lives. Claude Code's approach costs a slower walk, because ripgrep's sorted mode is single-threaded by necessity, and provides a guarantee Cursor does not have. That trade is the most transferable idea in this harness.

`--null` is the second. Splitting on NUL removes the newline-in-filename problem entirely, at the cost of one flag.

**The sort runs oldest first.** ripgrep's `--sort=modified` is ascending. With the shipped `rg.exe` (`ripgrep 15.1.0-cursor5`), three files dated three, two and one day back came out oldest, middle, newest, and `--sortr=modified` reversed them (probe). Between the engine and the slice nothing reverses the list: `Uzr`, which post-processes the rows, only splits the NUL-delimited records (source: `claude.exe`, `function Uzr(` at byte 252365564), and `m6t` takes `slice(offset, offset + limit)`. So when more than 100 files match, the 100 kept are the oldest, which is the opposite of what a model asking for recent files expects, and the description's "sorted by modification time" does not name the direction. This is derived from the argv, the slice and the probe, and was not observed end to end. Cursor's own sort is newest first.

**What the call does around the engine** (source: `m6t`, `q2e`, `eGo`, `yGo`, `Z1o`). An absolute pattern is split by `q2e` into a search directory and a relative pattern, which is why Claude Code supports absolute patterns, and `eGo` drops leading pattern segments that repeat the tail of the search directory. A `path` is validated first: a missing directory throws `Directory does not exist: <path>.` with the working directory and, when one is found, `Did you mean <path>?`, and a file throws `Path is not a directory: <path>`. Read-deny permission rules are passed to ripgrep as more `--iglob` arguments and the result is checked again after the search. The tool result carries `filenames`, `numFiles`, `durationMs`, `truncated`, `totalMatches` and `countIsComplete`, and the model sees only the paths and the notice.

The caps, from `limits.js`: `globLimits.maxResults ?? 100` in the `call()` body, and `QFe = 200` is the output line cap before the truncation notice. So the *search* limit is 100 and the *render* cap is 200, which means the render cap never binds - a small inconsistency in the constants that is harmless because the search limit is tighter.

The truncation notices, all three forms:

```text
(Results are truncated. Consider using a more specific path or pattern.)
(Showing ${n} of ${total} matching files; ${r} more are not listed. Narrow the pattern or path to see the rest.)
(Showing the first ${n} files; there are more than ${total} matches. Narrow the pattern or path to see the rest.)
```

The second and third distinguish a complete count from a lower bound, via `countIsComplete`. That is a better honesty design than Cursor's count suppression, because it always gives the model a number and separately says whether the number is exact.

Zero results render as:

```text
No files found
```

**Idea worth taking:** a sort in the engine rather than after it. It makes the ordering guarantee unconditional and removes the entire class of determinism problems in Q11. The cost is a serialized walk, which for a search tool is usually the right trade. Take it as `--sortr=modified`, or as a newest-first sort in your own walk, because Claude Code's ascending sort keeps the oldest 100 files.

The model-visible output is the paths and, when truncated, one notice on the last line. Composed from `mapToolResultToToolResultBlockParam` and `hGo` (source; the path form comes from `Z6t`, which was not extracted):

```text
<path 1>
...
<path 100>
(Showing 100 of 340 matching files; 240 more are not listed. Narrow the pattern or path to see the rest.)
```

**Idea worth rejecting:** `--no-ignore` by default. Claude Code shows the model every file including `node_modules`, which on a large repo is a much worse outcome than a hidden file. Cursor's choice to respect ignore files is better, provided the precedence is sane.

**Concurrency.** Claude Code's Glob is concurrency-safe and read-only by declaration (`isConcurrencySafe()` returns true, `isReadOnly()` returns true), and it declares `maxResultSizeChars: 1e5`. That is the same shape as Cursor's lock-free treatment, arrived at independently.

### Copilot

Source: VS Code Copilot, `C:\Users\Vinnie\src\vscode\extensions\copilot\src\extension\tools\node\findFilesTool.tsx` at vscode commit `b208bc25` (2026-10-25), with the name from `common\toolNames.ts` line 25: `FindFiles = 'file_search'`. The internal telemetry name is `copilot_findFiles`. (source)

The model-visible description and schema are in the extension's `package.json`, not in the tool file:

```text
Search for files in the workspace by glob pattern. This only returns the paths of matching files. Use this tool when you know the exact filename pattern of the files you're searching for. Glob patterns match from the root of the workspace folder. Examples:
- **/*.{js,ts} to match all js/ts files in the workspace.
- src/** to match all files under the top-level src folder.
- **/foo/**/*.js to match all js files under any foo folder in the workspace.

In a multi-root workspace, you can scope the search to a specific workspace folder by using the absolute path to the folder as the query, e.g. /path/to/folder/**/*.ts.
```

The `query` description adds "Can also be an absolute path to a workspace folder to scope the search in a multi-root workspace." The `maxResults` description tells the model "Do not use this unless necessary, it can slow things down." (source)

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | string | yes | - |
| `maxResults` | number | no | 20; in `FullContext` mode `max(maxResults, 200)` (`resolveInput`) |

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

The call passes `{ caseInsensitive: true }` explicitly, and `findFiles.spec.tsx` pins it: `expect(searchService.lastOptions?.caseInsensitive).toBe(true)`. So Copilot made the same case-insensitivity call as Cursor, on purpose, with a test.

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

Output shape, from `FindFilesResult` (the header reads `1 total result` for one, `N total results` otherwise):

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

**The `test_search` specialization.** `findTestsFilesTool.tsx` registers `ToolName.FindTestFiles = 'test_search'`, and it is a different kind of tool: it takes `filePaths: string[]` and optional `ranges`, and for each path it finds the *related* test file or source file using `TestFileFinder`, checking `isTestFile(document)` to decide the direction. It does not take a pattern at all. It is the one place in the set where a vendor decided file discovery deserved a purpose-built tool rather than a better glob. The implication is that the general glob is bad enough at the "where is the test for this file" question that Microsoft built a separate path for it, with language-aware heuristics rather than a pattern. Whether that is worth copying depends on how often your agent asks that question; if it does often, a `related_files` tool is a better choice than a smarter Glob.

**Idea worth taking:** the absolute-path handling. Copilot detects an absolute query, does not mangle it with a recursive prefix, and converts it into a scoped pattern against the right workspace folder. That is the correct answer to Trap 4 and costs about ten lines.

**Idea worth rejecting:** delegating to a workspace index. It is faster and it sees unsaved buffers, but it means the tool's answer depends on indexing state, which is a worse determinism story than Cursor's and much worse for a virtual filesystem that has no index to consult. Take the pattern handling.

### Zed

Source: `crates\agent\src\tools\find_path_tool.rs` (275 lines) in the Zed checkout `C:\Users\Vinnie\cursor\zed` at commit `37faef81f7` (2026-10-27, Zed 1.27.0), the canonical one. The report was first written against `C:\Users\Vinnie\src\zed` at `d9ad6aff67`; the differences are under `### Zed checkout drift`. (source) Name: `find_path`, which is `find_PATH` and not `find_FILES` - and that name is accurate, because it returns directories as well as files.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `glob` | string | yes | `The glob to match against every path in the project.`, then in the schema a blank line and an `<example>` block that uses `*thing*.txt` against three root directories (`find_path_tool.rs:24-34`) |
| `offset` | usize | no | `Optional starting position for paginated results (0-based). When not provided, starts from the beginning.` |

`offset` has `#[serde(default)]`, so it is optional and defaults to 0. The tool description is five bullets, not one line: the `**/*.js` and `src/**/*.ts` examples, the sort claim, a bullet telling the model to prefer `grep` when it is searching for symbols, the find-by-name bullet, and the paging bullet (source: `find_path_tool.rs:15-21`).

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

Three things follow. An **empty glob becomes `*`**, which is the opposite of Cursor's rejection and much friendlier: "match everything" is a reasonable thing to ask for and Cursor makes you spell it `*`. The matcher is `globset`, the crate ripgrep builds on, so brace and class syntax is the same, but the semantics are not ripgrep's. Zed builds each glob with only `backslash_escape` set, so `literal_separator` is off and `*` crosses `/` (`*thing*.txt` matches `directory1/a/something.txt`), where the `ignore` crate behind ripgrep sets `literal_separator(true)` (web: globset 0.4.17 `lib.rs` lines 72-73; ignore 0.4.23 `gitignore.rs:504-507`). `PathMatcher::is_match` also adds a prefix and suffix shortcut that is not globset behavior (source: `paths.rs:858-875`), so a bare name such as `root` matches the whole tree. And the layer above the crate matters: rig puts the `ignore` crate's override layer over globset, and there `src`, `src/` and `test*` match nothing (probe, see `rig`). `globset` is still a Rust crate you can depend on directly, so it remains the most transferable choice in the whole comparison for a Rust harness, provided you set its options on purpose. And an invalid glob returns `Invalid glob: <err>`, carrying the crate's own message, rather than a `rg:` prefixed line.

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

That is an explicit, documented divergence from both Cursor and Claude Code, and it is the honest version of Decision 3: pick one order, say which, and be unconditional. For a Rust harness that cannot cheaply stat every match, a path order is the right pick. Two limits qualify the word alphabetical. First, the order is the byte-wise comparison of path components, so uppercase sorts before lowercase and a directory's contents follow it directly (`apple/banana/carrot` comes before `apple/bandana/carbonara`, although `carbonara` sorts first by file name, and the tool's own test pins that). Second, it holds per worktree only, because several worktrees are concatenated in the worktree store's insertion order with no global sort (source: `rel_path.rs:305-314`, `find_path_tool.rs:248-254`, `worktree_store.rs:425-429`). `entries(false, 0)` also yields the worktree root itself first, and keeps gitignored entries out except those marked always-included, which by default means `.env*` (source: `worktree.rs:4221-4226`, `default.json:1801-1805`).

Output shape, from the `From<FindPathToolOutput>` impl:

```text
Found {all_matches_len} total matches.
Showing results {offset+1}-{offset+n} (provide 'offset' parameter for more results):
<abs path>
<abs path>
```

with paths as **absolute** paths. When the page is empty:

```text
No matches found
```

And when the total fits in one page, the "Showing results" line is omitted entirely, leaving just the count and the paths. An example from the tool's own test fixture (`/root/apple/banana/carrot`, `/root/apple/bandana/carbonara`, `/root/apple/endive`) and the glob `**/car*`; no test asserts the formatted string, so this is the formatter at `find_path_tool.rs:55-87` applied by hand to the vector the test pins at `:243-264` (derived, not run):

```text
Found 2 total matches.
/root/apple/banana/carrot
/root/apple/bandana/carbonara
```

Paging exists: `RESULTS_PER_PAGE = 50`, and the slice is `matches[cmp::min(input.offset, matches.len()) .. cmp::min(input.offset + 50, matches.len())]`. Both bounds clamp, so an ordinary out-of-range offset gives an empty page, but the end bound is `input.offset + RESULTS_PER_PAGE`, an unchecked add evaluated before the clamp. An offset within 49 of `usize::MAX` therefore overflows: a debug build panics, and a release build wraps (the release profile sets no `overflow-checks`) and then slices with start past end (derived from Rust semantics, not run). The model is told how to continue but not when to stop: the `Showing results A-B (provide 'offset' parameter for more results):` line prints whenever the total exceeds 50, including on the last page, where nothing more exists (source: `find_path_tool.rs:67`). A 1-based range beside a 0-based parameter also invites an off-by-one, since after `Showing results 1-50` the next call is `offset: 50`. And an offset past the end returns `No matches found`, which is false for the result set. It is still the cleanest paging story of the harnesses that page at all, and the one worth copying with those fixes.

Paging is stateless: every call takes fresh worktree snapshots and re-enumerates the whole tree (source: `find_path_tool.rs:135-144`), so each page costs a full scan, and a tree that changes between pages shifts the window. A page is stable only because the order is total over unique paths and the tree is unchanged. Absolute paths do not make it stable.

**What the snapshot leaves out, none of it announced** (source: `worktree.rs:5489-5494`, `:5528-5540`, `:6299-6316`; `default.json:1788-1820`). Gitignored entries; entries under `file_scan_exclusions` (`.git`, `.svn`, `.hg`, `.DS_Store` and others), which never enter the snapshot; files whose names are not valid UTF-8; the contents of symlinked directories unless the directory was expanded or `scan_symlinks` is `always`; directories five or more components deep when the project is not a git repository (`file_scan_depth`); and anything the scanner has not reached yet, since `find_path` does not wait for the scan. The result does include the worktree root itself, first, and directories as well as files, although the tool description says "file paths" (derived from the code and the `test_traversal` assertions).

**Idea worth taking:** `offset` in the tool schema, clamping both slice bounds with `saturating_add`, printing the next offset, and stating the page range only when more remain. A few lines of code turn a truncation into a resumable conversation.

**Idea worth rejecting:** returning absolute paths when everything else in the harness is workspace-relative. It makes every downstream tool call include a long prefix, and it differs from Cursor, so a model switching between the two has to relearn. Relative paths plus a documented root is cheaper. The system prompt tells the model to start paths with a project root name and `read_file` says paths "should never be absolute", so the instruction and the output disagree, although `find_project_path` still accepts absolute paths, so the round trip works (source: `system_prompt.hbs:57`, `read_file_tool.rs:162-164`, `project.rs:5546-5558`).

**A second idea worth rejecting: a search tool that skips the visibility rules its siblings apply.** `read_file`, `list_directory` and `grep` refuse or hide `private_files` and `file_scan_exclusions` paths, and `list_directory` gates symlink escapes. `find_path` applies none of that, and it is not in `tool_permissions.rs`: it sits in `EXCLUDED_TOOLS` as a "read-only / low-risk" tool, so the `agent.tool_permissions` settings cannot restrict it and only the agent profile switch can (source: `tool_permissions_setup.rs:1404-1411`; `read_file_tool.rs:304-330`, `grep_tool.rs:246-247`, `list_directory_tool.rs:139-151` and `:258-327`; `list_directory` also shows ignored entries). With the default `file_scan_inclusions` of `[".env*"]`, a glob of `**/.env*` lists `.env` files that `read_file` and `list_directory` refuse to touch (derived; no test covers it). Only names leak. For Runvil, route Glob through the same `Access` gate and the same hidden-path filter as Read; `Access::glob` already gates and claims, so the work is to make the filter shared. Confidence: high, because the inconsistency is visible in three files and a read gate that a name listing can walk around is not a gate.

**A note on the neighbors.** Zed also has `list_directory_tool.rs` (41,569 bytes in the canonical checkout, not 43 KB) and no separate glob-vs-ls ambiguity, because `find_path` returns entries of any kind and `list_directory` shows one level. Cursor's split - `Glob` returning files only, `ListDir` returning one directory - leaves no way to ask for "all directories named `test`," which Zed gets for free. Runvil's `Access::glob` supports the trailing-slash directory form, so you already have a capability that Cursor does not, and it is worth exposing. The two tools also differ in the visibility rules they apply, as the second rejected idea above says.

### pi, and Cursor's `pi_find` port

**Source and name.** pi's tool is `find`, in `packages/coding-agent/src/core/tools/find.ts` (318 lines) with `truncate.ts`, `path-utils.ts` and `src/utils/tools-manager.ts` beside it, in the pi repository (package `@earendil-works/pi-coding-agent` 1.4.0) at commit `a92283056` (2026-10-27). It is opt-in: the default tool set is `read`, `bash`, `edit` and `write`, and `find`, `grep` and `ls` belong to the read-only set (source: `settings-manager.ts:85`, `tools/index.ts:173-180`). **pi's own `find` is not a JavaScript walk. It wraps the external `fd` binary** (source: `find.ts:172-216`), and downloads a release from GitHub when none is on the machine (source: `tools-manager.ts:337-395`). The package also has a host seam, `FindOperations { exists, glob }`, so a host such as an SSH or VM backend can replace the engine. Its gondolin example implements `glob` as a JavaScript walk that visits files only and counts matches (source: `find.ts:52-57`, `examples/extensions/gondolin/index.ts:137-201`). Cursor's port is `pi_find`, with `LocalPiFindExecutor` (`16846956`) and `walk2` (`16844366`) in the daemon, compiled from `../local-exec/dist/pi/find.js`, behind the feature flag `enablePiTools`. Which pi release Cursor ported is not in the sources; the trailer strings match pi since 2025-12-07 and the 1000 default matches pi's first version. (source, bundle, daemon)

The same bundle registers a parallel family of `pi_*` tools. Glob's counterpart is `pi_find`, at registration offset `4562769`, described as:

```text
Search for files by glob pattern using pi-compatible find semantics.
```

This tool is not a renamed Glob, and it is not pi's engine either. Its schema, from the daemon at `13912670`, is verbatim pi's: the same three names and the same three description strings, in Zod instead of TypeBox:

```js
object({
  pattern: string2().describe("Glob pattern to match files, e.g. '*.ts', '**/*.json', or 'src/**/*.spec.ts'"),
  path: string2().optional().describe("Directory to search in (default: current directory)"),
  limit: number2().optional().describe("Maximum number of results (default: 1000)")
})
```

pi's own description, which the port cut to one sentence, names three facts the port does not: results are relative to the search directory, `.gitignore` is respected, and the output cap is 1000 results or 50KB (source: `find.ts:78`, with the caps interpolated).

```text
Search for files by glob pattern. Returns matching file paths relative to the search directory. Respects .gitignore. Output is truncated to 1000 results or 50KB (whichever is hit first).
```

Dropping "Respects .gitignore" from the port's description is accurate, because the port does not respect it.

**Cursor's port against pi's original, and against Cursor's own `Glob`.** The port keeps pi's schema, default limit, trailer strings and byte cap, and replaces the engine; so it agrees with pi on the surface and disagrees on matching, ignoring, counting and ordering. (source, bundle, probe)

| Dimension | Cursor `Glob` | Cursor `pi_find` (port) | pi `find` (upstream) |
|---|---|---|---|
| Engine | ripgrep `--files` subprocess, globset | `minimatch` over relative paths, in process | external `fd` subprocess with `--glob --hidden --max-results` |
| Pattern parameter | `glob_pattern`, required | `pattern`, required | `pattern`, required, same description text |
| Directory parameter | `target_directory`, optional | `path`, optional | `path`, optional |
| Result limit | fixed 2000 internal, not settable | `limit`, settable, default 1000, clamped to at least 1 by `Math.max(1, limit ?? 1000)`, so 0 and negatives become 1 and 2.5 stays 2.5; counts **walked entries** (bundle `@16847668`; probe) | `limit`, default 1000, no clamp (source: `find.ts:112`); counts **fd results** through `--max-results` |
| Enumeration | ripgrep's parallel walk | recursive `readdir` with `withFileTypes`, depth first, sequential | fd's parallel walk |
| Ignore files | `.gitignore` and `.cursorignore`, via ripgrep | no `.gitignore`, `.ignore` or global ignore, but `.cursorignore` and permission denies apply per entry through `shouldBlockRead`, and a blocked directory is not descended (bundle `@16844366`, `@16676746`) | `.gitignore` honored by fd, with `--no-require-git` added when no `.git` is found in the search path or above it |
| Skipped entries | `.git` only, explicitly | `.git` and `node_modules`, hardcoded by name at every depth | none in the default branch, so `.git` contents are likely listed (not probed); the custom-operations branch passes `**/node_modules/**` and `**/.git/**` |
| Sort | mtime descending under a budget | none; `readdir` order | none; fd's arrival order, which is unspecified |
| Case sensitivity | insensitive, forced | `minimatch` default, case sensitive (probe: `*.TS` did not match `a.ts`) | smart case, because fd is (web: fd man page) |
| Recursive prefix | `**/` always prepended if absent | `**/` prepended only if the pattern has a `/` and does not start with `**/`, so a bare `*.ts` matches only the root | a slash-less pattern matches basenames at any depth; a pattern with `/` gets `--full-path` and `**/` (source: `find.ts:201-213`) |
| Output | header plus `- path` lines | bare newline-joined relative paths, no header, no trailing `/` on directories | bare relative paths, `/` separators, a trailing `/` on directories |
| Zero-match text | `Result of search in '<p>': 0 files found` | `No files found matching pattern`, also when the walk budget ran out before a match | `No files found matching pattern` |
| Truncation | mid-list notice plus banner | bracketed notice appended; byte notice `50KB limit reached` | bracketed notice appended; byte notice `50.0KB limit reached` |
| Bad glob such as `[` | error text from ripgrep | `No files found matching pattern`, no error (probe) | fd's error text relayed (source: `tools.test.ts:1005-1013`) |
| Dotfiles | included, `--hidden` unconditional | included, `{dot:true}` in minimatch | included, `--hidden` |
| Symlinks | followed, `--follow` unconditional | not descended: `readdir` reports the link, not the target's contents, and it is listed like a file with no trailing slash (probe, Windows junction) | not followed; fd does not descend into symlinked directories without `-L` (web) |

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

The walk records **directories as well as files**, with a trailing separator, so `pi_find` matches directories where `Glob` does not. The separator does not reach the model: the executor maps each walked path through `path.relative` before matching and printing, which drops it, so the model sees `src` for a directory and cannot tell it from a file named `src` (probe). It counts the limit against the walk rather than against the matches, so `limit: 1000` means "inspect 1000 entries," and a directory-heavy tree can return zero matches after a full budget. That zero is silent: the zero-match return comes before the limit notice is built, so a file that exists beyond the budget is reported as `No files found matching pattern` with no warning, and the notice can fire with fewer matches than the limit (probe: 10 entries walked, 5 matched, and the notice says `10 results limit reached`). One unreadable subdirectory fails the whole call, because there is no per-directory `try`, and the executor takes no abort signal. It is recursive with no depth guard and no visited set, so a symlink loop would be a stack overflow rather than a graceful error, but `readdir` with `withFileTypes` and no `follow` does not descend into links, so loops cannot occur. And it is strictly sequential, one directory at a time.

The `limit` notice is:

```text
${limit} results limit reached. Use limit=${limit*2} for more, or refine pattern
```

joined into a bracketed block with the byte-truncation notice when that also fired:

```text
[${notices.join(". ")}]
```

Two details of that notice block matter. The byte-cap notice is `${formatSize(51200)} limit reached`, and Cursor's own `formatSize` rounds to whole kilobytes, so it prints `50KB limit reached` where pi's prints `50.0KB limit reached` (bundle `@16825492`; source: pi `truncate.ts:61-68`). And the block is joined to the list by exactly two newlines, `\n\n[`, outside the byte budget, so the text can run past 51200 bytes by the length of the notice (probe: 1500 files with 60-character names at the default limit gave 51,273 characters, with 853 of the 1000 matches surviving the byte cap and the trailer advising `limit=2000`, which cannot raise a byte cap).

The result is truncated with `truncateHead`, but the call is `truncateHead(output2, { maxLines: Number.MAX_SAFE_INTEGER })`, so the 2000-line default is overridden out of existence and only the `PI_DEFAULT_MAX_BYTES` of 51200 engages. The line cap is dead on this path. That override is pi's own, not Cursor's: pi's `find.ts`, `ls.ts` and `grep.ts` all pass `{ maxLines: Number.MAX_SAFE_INTEGER }` (source: `find.ts:147`, `find.ts:277`, `ls.ts:141`, `grep.ts:284`), with a comment in `ls.ts` that the entry count is already capped, and the port inherited it. `PI_DEFAULT_MAX_LINES = 2e3` still exists as the default for `truncateHead` and `truncateTail`. The reasoning holds upstream, where `limit` counts matches. In the port `limit` counts walked entries, so the stated bound is weaker.

The recursive prefix rule differs from `Glob`'s: `pi_find` prepends `**/` only when the pattern contains a `/` and does not already start with `**/`, so a bare `*.ts` matches only at the root, where `Glob` would make it recursive. That is a semantic difference, and it silently changes results for the most common pattern. It differs from upstream pi too. Upstream's `fd --glob` matches a slash-less pattern against the basename at any depth, so `*.ts` finds every TypeScript file, and pi pins that with a regression test (source: `find.ts:201-213`; `3302-find-path-glob.test.ts:50-53`). Upstream is also smart-case where the port is case sensitive (probe: `*.TS` did not match `a.ts`), and upstream relays `fd`'s error for a bad glob such as `[` (source: `tools.test.ts:1005-1013`) where the port answers `No files found matching pattern` (probe). So the port departs from pi, not only from Cursor's `Glob`, which is what the label "pi-compatible" does not say. And matching is against the path relative to `path`, converted to POSIX separators.

**`pi_find` as a model for a fresh implementation.** It is a better model structurally and a worse one semantically. It is better because it is in-process, uses the VFS's own enumeration, needs no subprocess or shipped binary, and has a settable limit with an honest notice. It is worse because it has no `.gitignore` support (only `.cursorignore` and permission denies, per entry), no sort, a walk-order result, a limit that counts the wrong thing, and a matching engine (`minimatch`) whose semantics differ from ripgrep's in the corners. Take the structure and the honest notices; keep Cursor's matcher semantics and add the ignore layer.

**What pi's own `find` does, in order** (source: `find.ts:82-310`; `fd` itself was not run, so fd behavior is cited to its man page, web). It resolves `path` against the working directory (accepting `@`, `~`, `file://` and Git Bash drive paths), takes `limit ?? 1000`, and runs `fd --glob --color=never --hidden`, adding `--no-require-git` when no `.git` exists in the search path or any ancestor, so `.gitignore` files still apply outside a repository. It adds `--max-results <limit>`. If the pattern contains `/` it adds `--full-path` and, unless the pattern starts with `/` or `**/` or is `**`, prepends `**/`; on Windows each `/` becomes `[/\\]`. A pattern with no `/` goes through untouched. It runs `fd <args> -- <pattern> <searchPath>`, and collects stdout lines in arrival order with no sort. Each line then has a trailing `\r` stripped and is trimmed, which also strips legitimate leading and trailing spaces in a name, and is relativized to the search root. The count `relativized.length >= limit` is taken before truncation, `truncateHead` cuts the list at 51,200 bytes with the line cap disabled, and the bracketed notices are appended. A non-zero fd exit with some output is returned as a partial success; with no output it returns fd's stderr as the error. The host-supplied `FindOperations.glob` branch prints a shorter notice, `<limit> results limit reached`, with no `Use limit=` hint, so two branches of one tool disagree on trailer text, and Cursor copied the fd branch's.

| Cap | pi `find` | Cursor `pi_find` |
|---|---|---|
| Result limit | 1000 default, any number accepted, fd results | 1000 default, at least 1, walked entries |
| Byte cap | 51,200 UTF-8 bytes of the newline-joined list; notices not counted | the same |
| Line cap | `DEFAULT_MAX_LINES` 2000, disabled with `Number.MAX_SAFE_INTEGER` | the same |
| One path over the byte cap | empty content plus the byte notice | the same |
| Time, depth, stdout size | none; fd output is buffered until fd exits, bounded only by `limit`, which the model sets with no ceiling | the walk stops at `limit` entries; each entry awaits `shouldBlockRead`, sequentially; `readdir` reads a whole directory before the limit check |

Output from the tests in pi's repository and from a probe of the port (the port was transliterated from the daemon with `shouldBlockRead` allowing everything, on Windows, with `minimatch` 9.0.9 standing in for the bundled 10.2.5):

```text
visible.txt
.secret/hidden.txt
```

That is pi's own `**/*.txt` test result (source: `tools.test.ts:972-990`). A limit of 10 on a tree of 30 directories each holding `x.txt`, pattern `**/x.txt`, through the port: ten entries walked, five matched, and the notice still says the limit was reached (probe):

```text
d00/x.txt
d01/x.txt
d02/x.txt
d03/x.txt
d04/x.txt

[10 results limit reached. Use limit=20 for more, or refine pattern]
```

The same tree with `zz/target.md` added at the end and `limit: 10`, pattern `**/target.md`: the file exists, and the model sees only `No files found matching pattern` (probe).

**The one idea worth taking: pi's pattern-shape rule and its `FindOperations.glob(pattern, cwd, { ignore, limit })` contract.** A pattern with no `/` matches the basename at any depth, so `*.ts` finds every TypeScript file, and a pattern with a `/` is anchored with `**/` and matched against the path, so `src/**/*.spec.ts` works. pi pins both with regression tests, and issue 3302 says the path form originally returned nothing. The port lost the most common model call, the bare extension glob. The contract is the structure to copy: it hands the host the ignore list and the limit, and pi's own example counts matches. Confidence: high, because pi pins the pattern rule with tests and the port's departure is reproducible. (source, probe)

**The one idea worth rejecting: counting the walk instead of the matches, and answering "no files" before the limit notice.** In the port a file that exists is reported as absent with no warning, and a trailer can claim `limit reached` with half the limit in matches. Do not copy pi's reliance on an external `fd` whose arrival order is unspecified either: the fd man page says `-l` is how to get a deterministic order, and `--max-results` picks an unspecified subset, which defeats replay (web). For Runvil, count matches, print the trailer even when the list is empty so that "no files" can never hide a cut-short search, and sort by path at minimum. Confidence: high for the first, medium for the second, because fd's ordering was read from the man page and not probed. (source, probe, web)

After reading upstream I would flip part of the verdict above on structure. The structure to take is pi's `FindOperations.glob` contract, with a match-counting walk behind it. The port's structure, where `limit` counts the walk, is the part not to copy.

### rig

**Source and name.** `crates/rig-tools/src/search.rs` (385 lines) and `list.rs` (226 lines) in the rig repository at commit `a83b774d0` (2026-10-25), with `rt/workspace.rs` and `rt/fs.rs` beneath. **Rig has no glob tool.** One tool, `search`, does both file-name and content search, and a second, `list`, is a recursive, sorted, files-only listing. The shipped agent offers neither, since its coding tools are `read`, `edit`, `write` and `shell` "as pi has"; a plugin registers them. A glob is asked for as `files: true, pattern: "", glob: "*.rs"`. (source)

| Name | Type | Required | Default | Description (doc comment) |
|---|---|---|---|---|
| `pattern` | string | yes | none | The regular expression; with `files`, empty lists every file. |
| `path` | string or null | no | the current directory | The directory or file to search; default the current directory. |
| `glob` | string or null | no | no filter | Only files matching this glob, such as `*.rs`. |
| `hidden` | boolean | no | false | Also hidden files and directories (named with a leading `.`), such as `.github/`; files git ignores stay out. |
| `files` | boolean | no | false | Only the paths of the files with a match, one per line. |

`list` takes `path`, `depth` (1 lists the directory's own entries; default every level), `glob` and `hidden`. `search` refuses unknown keys (`deny_unknown_fields`). (source)

| Cap | Value | Unit |
|---|---|---|
| `MAX_FILES` | 1000 | file names in files mode, counted after a push, so a result of exactly 1000 gets the stop notice too (probe) |
| `MAX_MATCHES` | 200 | hits in content mode |
| `list` | 2000 entries or 51,200 bytes | entries and `String::len`; the notice is added after the check, so a cut listing can pass 50 KiB (probe: 51,255 bytes) |
| Time, depth, cancellation | none | the call runs on its own thread, which a dropped future cannot stop |

**What the glob means.** The `glob` argument goes to the `ignore` crate's `OverrideBuilder`, which puts gitignore-syntax rules on top of globset, and the walk is the crate's single-threaded `Walk`. On a small tree, probed against the locked crates (`ignore` 0.4.25, `globset` 0.4.18), `*.rs` matches at any depth. `src/*.rs` is anchored and does not cross `/`. `src/**/*.rs` works. `!*.md` returns every file except the `.md` ones, and braces work. Matching is case sensitive even on NTFS (`*.RS` finds nothing). A glob that is only a directory name, `src`, `src/` or `test*`, **returns nothing**, because it matches the directory node and the files inside are tested on their own. A malformed glob is `bad glob: error parsing glob '[': unclosed character class; missing ']'`. When `path` is a file, the file is searched whatever the glob says. (probe)

**What it skips.** `.gitignore`, `.ignore`, `.git/info/exclude` and parents' ignore files, but only inside a git work tree: with no `.git`, `.gitignore` is inert and `.ignore` is not (probe). Hidden files and any path part named `target` are dropped by name, which also drops a regular file called `target`. Symlinks are never listed, and walk errors are dropped silently. A glob has the highest precedence, so a glob that matches an ignored file, or an ignored directory's own node, brings it back (see the independent confirmation under Trap 1). The description still says "Skips files ignored by git". (source, probe)

Output is one path per line, relative to the base when `path` is relative, then, at the cap, one notice line; nothing found is `No matches.` Rig's tests expect `src/a.rs`, and the same call on Windows printed `src\a.rs` (probe). A cut result, from a probe with 1,001 files in files mode, is the 1000 paths followed by `[stopped at 1000 files; narrow the search with path or glob]`.

**Ordering.** The local walk is unsorted and single-threaded, so it follows the file system's directory order; on NTFS the same call twice gave equal lists (probe). The remote backend sorts, and `list` sorts by the platform-spelled string, so the three disagree. The authors know it: the files-mode test sorts before asserting. (source, probe)

**The one idea worth taking: search is two trait calls and a pure function that returns data.** `find(&dyn Workspace, &SearchArgs) -> Result<Found, _>` walks with `Workspace::files(root, glob, listing)` and reads with `read_files`, and `render` is a separate function. The same code runs over `Local`, `Overlay` and `Hosts` with no search method in the storage trait, and `Listing` (`Visible`, `Hidden`, `All`) is a parameter, so what counts as ignored is the backend's decision. For Runvil, `Access` has `glob` and `read`, so a Pack-level search can take the same shape. Rust mechanisms Runvil could borrow are `ignore::WalkBuilder` with `OverrideBuilder` for a real-disk backend, and the `Listing` enum for the VFS. Confidence: medium-high, because tests and four consumers pin the shape. (source)

**The one idea worth rejecting: silent omission behind a reassuring zero.** `No matches.` follows an unreadable file, a symlink, a walk error, a non-UTF-8 file in content mode, an ignore rule that was inert for lack of a `.git`, and a directory-name glob, all under a description that says ignored files are skipped, and the stop notice fires on an exact fit. A model that gets `No matches.` concludes the code is absent. Count what was skipped by reason and say so. Confidence: high, because the behaviors are probed with the real crates; whether models are misled was not measured. (probe)

### zeroclaw

**Source and name.** `crates/zeroclaw-tools/src/glob_search.rs` in the zeroclaw repository at commit `2d85371f3` (2026-09-18), workspace version 0.8.5. The tool is `glob_search`, wrapped as `RateLimitedTool(PathGuardedTool(GlobSearchTool))`, auto-approved by default, and built on the `glob` crate 0.3.4. It lists files only; directories are dropped. (source)

Its description, verbatim (`glob_search.rs:27-29`):

```text
Search for files matching a glob pattern within the workspace. Returns a sorted list of matching file paths relative to the workspace root. Examples: '**/*.rs' (all Rust files), 'src/**/mod.rs' (all mod.rs in src).
```

| Name | Type | Required | Default | Description, verbatim |
|---|---|---|---|---|
| `pattern` | string | yes | none | `Glob pattern to match files, e.g. '**/*.rs', 'src/**/mod.rs'` |

There is no base-directory parameter, no ignore control, no sort option and no limit parameter.

| Cap | Value | Unit |
|---|---|---|
| `MAX_RESULTS` | 1000 | accepted file entries |
| result cap in the agent loop | 50,000 | bytes, cut in the middle |
| action budget | 20 per 3600 s | successful calls, reads included |
| walk time | none | blocking `std::fs` calls inside an async function |

**What it does** (source: `glob_search.rs:45-170`; `glob` 0.3.4 `lib.rs`). The pattern passes a lexical path guard first, then a leading `/` or `\` is refused unless it is under an allowed root, and `..` is refused. `resolve_tool_path` joins the pattern onto the workspace path and the result goes to `glob::glob`, so the workspace path is spliced into the pattern unescaped. A workspace directory named `ws[1]` is therefore read as a character class and finds nothing (probe: `glob('<tmp>/ws[1]/*.txt')` gave 0 matches). The crate is case sensitive, `*` does not cross `/`, dotfiles match, braces are not supported (`*.{txt,rs}` matched 0), and non-UTF-8 names are skipped (probe). Each hit is canonicalized, re-checked against the read policy, dropped if it is a directory, and pushed as a workspace-relative path.

**The cap cuts before the sort.** After each push, `if results.len() >= MAX_RESULTS { truncated = true; break; }` runs inside the walk, and `results.sort()` runs afterwards, so which 1000 survive is decided by the crate's walk order and the sort only orders the survivors. The truncation flag fires at 1000 matches, where nothing was cut (probe: 999 kept and no flag, 1000 and 1001 both kept 1000 with the flag). And `Total` reports the shown count, not the match count: the line is `Total: {results.len()} files` and it is printed after the cut. Output is the sorted paths, an optional `[Results truncated: showing first 1000 of more matches]` line, then `Total: N files`. A sorted result, from the probe's five files:

```text
.hidden.txt
B.txt
a-b/y.txt
a.txt
a/z.txt

Total: 5 files
```

The sort is a byte-wise string sort of the relative path, so `a-b/y.txt` precedes `a.txt` precedes `a/z.txt`, capitals precede lowercase, and on Windows `\` sorts differently from `/` (probe). No match is the success text `No files matching pattern '<p>' found in workspace.` (source, probe)

**Dead path.** The tool admits an absolute pattern under an allowed root, but then drops every hit outside the workspace with `strip_prefix(&workspace_canon)`, so that call returns the no-match text. This is from reading the code; no test covers it and it was not run. (source)

**The one idea worth taking: re-check every hit by its canonical path after the walk.** A symlink inside the workspace cannot surface a file outside it, and a hit under a write-only root is dropped. Unix-only tests pin both. For a VFS with gating, listing is the place where names escape. Confidence: high. (source)

**The one idea worth rejecting: the cap logic.** It cuts in walk order before it sorts, sets the flag when the count merely reaches the cap, and prints a `Total` that is the shown count. A model reads `Total: 1000 files` as the whole answer. Count the true total, cut after sorting, and flag only when something was cut. Confidence: high, because the three behaviors are probed. (probe)

### Codex CLI

**Source and name.** `github.com/openai/codex`, `main` at commit `bea47c06dfac` (committed 2026-10-28T05:46Z; the workspace version is `0.0.0`, so no release number can be derived). **There is no glob tool.** The model lists and finds files with `exec_command`: `rg --files`, `fd`, `find`, `ls` or `tree`. Every shipped instruction template for the 12 models in `models.json` says "When you search for text or files, you reach first for `rg` or `rg --files`; they are much faster than alternatives like `grep`." A dedicated `list_dir` tool existed and was removed on 2026-05-05 by PR 21170; it was behind `experimental_supported_tools`, which none of the 12 models lists. Its last version is at the PR's parent `9d579813bb` (`core/src/tools/handlers/list_dir.rs`). The shell classifier labels `rg --files`, `ls` and `tree` as `ListFiles` for the UI only. (source, web)

What the model uses today, `exec_command`:

| Parameter | Type | Required | Default |
|---|---|---|---|
| `cmd` | string | yes | none |
| `workdir` | string | no | the turn's working directory |
| `yield_time_ms` | number | no | 10000, effective range 250 to 30000 |
| `max_output_tokens` | number | no | 10000 |

The removed `list_dir`:

| Parameter | Type | Required | Default | Description, verbatim |
|---|---|---|---|---|
| `dir_path` | string | yes | none | `Absolute path to the directory to list.` |
| `offset` | number | no | 1 | `The entry number to start listing from. Must be 1 or greater.` |
| `limit` | number | no | 25 | `The maximum number of entries to return.` |
| `depth` | number | no | 2 | `The maximum directory depth to traverse. Must be 1 or greater.` |

Caps: shell output is middle-truncated at `max_output_tokens`, 4 bytes per token, half kept at each end, with `…N tokens truncated…` and a header `Warning: truncated output (original token count: N)`. `list_dir` capped an entry name at 500 bytes and counted `limit` in entries and `depth` in levels. (source)

`list_dir` walked breadth first to `depth`, sorted the full list by relative path in byte order, and then sliced by `offset` and `limit`, so pages were stable on an unchanged tree but every call walked everything. Symlinks were labelled and never entered, and denied entries and subtrees were hidden. A test's output, `offset` 1, `limit` 20, `depth` 3 (source: `list_dir_tests.rs:20-76`):

```text
entry.txt
link@
nested/
  child.txt
  deeper/
    grandchild.txt
```

When entries remain after the page, the last line is `More than {n} entries found`, where `n` is the page size returned, not the total. The description promised "1-indexed entry numbers", but the output has none.

**What this implies.** Codex trusts the model to use a shell for finding and trusts the OS sandbox plus an approval policy as the gate: there is no read allowlist in the exec policy, and no per-tool check. The order is whatever `rg --files` prints, the harness never sorts, and a middle cut can drop the middle of a path list with only a token count to say so. Nothing is announced about ignore files, because `rg` applies its own. Whether the models were trained on this surface is not in the sources. (source)

**The one idea worth taking: apply the read-deny policy to every entry of a listing.** `list_dir` hid denied names and pruned denied subtrees (tested), so a listing could never leak a name that a read would be refused. For a VFS with gating, listing is where names escape. Confidence: high. (source)

**The one idea worth rejecting: a description that promises "1-indexed entry numbers" while the output prints none, with a trailer that gives the page size and not the next `offset`.** The model has to count lines to compute the next call. Confidence: high. (source)

### Gemini CLI

**Source and name.** `github.com/google-gemini/gemini-cli`, tag `v0.66.0`, commit `970413207732` (2026-10-27), `packages/core/src/tools/glob.ts`, with the npm `glob` package 12.0.0. The tool is `glob` (display name FindFiles). There are two schema sets, `default-legacy` and `gemini-3`, chosen by model family; the parameter names are identical and the glob description is identical in both. `read_file`, `glob`, `grep_search` and `list_directory` are `allow` at priority 50, so there is no prompt. (source)

| Name | Type | Required | Default | Description, verbatim |
|---|---|---|---|---|
| `pattern` | string | yes | none | `The glob pattern to match against (e.g., '**/*.py', 'docs/*.md').` |
| `dir_path` | string | no | every workspace directory | `Optional: The absolute path to the directory to search within. If omitted, searches the root directory.` |
| `case_sensitive` | boolean | no | false | `Optional: Whether the search should be case-sensitive. Defaults to false.` |
| `respect_git_ignore` | boolean | no | true | `Optional: Whether to respect .gitignore patterns when finding files. Only available in git repositories. Defaults to true.` |
| `respect_gemini_ignore` | boolean | no | true | `Optional: Whether to respect .geminiignore patterns when finding files. Defaults to true.` |

```text
Efficiently finds files matching specific glob patterns (e.g., `src/**/*.ts`, `**/*.md`), returning absolute paths sorted by modification time (newest first). Ideal for quickly locating files based on their name or path structure, especially in large codebases.
```

| Cap | Value | Unit |
|---|---|---|
| Result count | **none**: `glob.ts` has no slice, limit or count cap | entries |
| Output size | 64 KiB, cut by the scheduler for every tool, with a suffix pointing at a temp file | UTF-8 bytes |
| Recency window | 24 hours | milliseconds |
| Fixed excludes | `**/node_modules/**`, `**/.git/**`, `**/bower_components/**`, `**/.svn/**`, `**/.hg/**` | glob patterns; no parameter turns them off |

**What it does** (source: `glob.ts:143-296`). Roots are `dir_path`, or every workspace directory, and the pattern runs once per root. If `path.join(root, pattern)` exists on disk, the pattern is escaped and matched literally. The options are `nodir`, `stat: true`, `nocase: !case_sensitive`, `dot: true`, `follow: false`. So the default is case insensitive and hidden files are included. Each entry is mapped through a synchronous `realpath` and made relative, then the `.gitignore` and `.geminiignore` filters run over the relative paths. Results are not re-checked against the workspace; only the search root is validated. Paths in the output are absolute, although the repository's docs show relative ones.

**The sort.** `sortFileEntries` puts entries modified less than 24 hours ago first, newest first, and sorts the rest by `fullpath().localeCompare`. The 24-hour split uses the wall clock at call time, so the same tree can order differently an hour later, and the second tier depends on the runtime's locale data (probe: `["B.txt","a.txt","_x.txt","Z.txt","é.txt"]` sorts to `_x a B é Z` under the default ICU locale). (source, probe)

Output, from the template at `glob.ts:278-287`:

```text
Found 2 file(s) matching "*.ts" within /repo/src, sorted by modification time (newest first):
/repo/src/b.ts
/repo/src/a.ts
```

With several roots the header says `across <n> workspace directories`, and with ignored files `(<n> additional files were ignored)` sits before the comma. Zero matches is `No files found matching pattern "<p>" within <dir>`, plus ` (<n> files were ignored)` when the ignore filter removed some. Errors: `The 'pattern' parameter cannot be empty.`, `Search path does not exist <dir>`, `Search path is not a directory: <dir>`, `Path not in workspace: ...` and `Error during glob search operation: <msg>`. (source)

**Where the 100 comes from.** Gemini's `glob` has no cap, so the "100 results" that this report's earlier text and the parent spec attributed to it belongs to `grep_search`, where `DEFAULT_TOTAL_MAX_MATCHES = 100` counts matches (see `### Caps and limits`). The `glob` cost is the unbounded part: `stat: true` stats every match, a synchronous `realpath` runs per match, and the whole set is sorted in memory before the 64 KiB cut. (source)

**The one idea worth taking: the recency tier.** Files touched in the last day come first, newest first, then the rest alphabetically (pinned by tests). It puts what the user is working on at the top without a flag. Confidence: medium, because the 24-hour constant is a guess and the time dependence breaks reproducibility. (source)

**The one idea worth rejecting: an unbounded result set with a synchronous `realpath` per match.** Cost grows with the match count and the only brake is a late byte cut. Confidence: high. (source)

### OpenCode

**Source and name.** `github.com/anomalyco/opencode` (formerly `sst/opencode`), tag `v1.19.5`, commit `810c0eba1a73` (2026-10-27), `packages/opencode/src/tool/glob.ts` (77 lines) with the engine in `packages/core/src/ripgrep.ts`. Three implementations coexist at the pin: this legacy set, which the session loop uses; a V2 rewrite in `packages/core/src/tool/glob.ts`, which takes a `limit` with no default; and the dev branch (`fa1260d6ca`), whose `plugin/glob.ts` adds `hidden` and `limit` (default 100) and a 30 second timeout. The tool is `glob`. (source, web)

| Name | Type | Required | Default | Description, verbatim |
|---|---|---|---|---|
| `pattern` | string | yes | none | `The glob pattern to match files against` |
| `path` | string | no | the instance directory | `The directory to search in. If not specified, the current working directory will be used. IMPORTANT: Omit this field to use the default directory. DO NOT enter "undefined" or "null" - simply omit it for the default behavior. Must be a valid directory path if provided.` |

The `path` description is Claude Code's, word for word, including the warning not to send `"undefined"` or `"null"`. The description is six bullets (`glob.txt`) that share the fast-matching, find-by-name and batch-calling bullets with Cursor's description, add a bullet sending open-ended search to the Task tool, and say only "Returns matching file paths". The public docs add "Returns matching file paths sorted by modification time", which the code does not do. (source, web)

| Cap | Value | Unit |
|---|---|---|
| Results | 100 (`const limit = 100`, `glob.ts:49`) | file paths; the adapter takes `limit + 1` lines from ripgrep |
| Wall clock | none in the legacy set; the dev branch adds 30,000 ms | ms |
| Reply | the generic 2000-line or 51200-byte wrapper is bypassed, because the tool sets `metadata.truncated` | not applicable |

**What it does** (source: `glob.ts:19-72`, `ripgrep.ts:154-185`). The call asks permission `glob` on the pattern text before it checks the path, resolves `path`, throws `glob path must be a directory: <path>` for a file, and runs `rg --no-config --files --glob=<pattern> --glob=!**/.git/** .` with the search directory as the working directory. `--hidden` and `--follow` are added only when the caller asks, and the tool never does. It strips a leading `./` from each line, turns every backslash into `/`, keeps the first 101 lines, resolves each to an absolute path, and sets `truncated = files.length === limit`. The pattern goes to `--glob`, so it follows gitignore glob rules, and ripgrep applies `.gitignore` and `.ignore`.

With the same argv (probe, ripgrep 15.1.0-cursor5), `--glob=*.ts` lists files at any depth. `--glob=*.yml` finds nothing under `.github/workflows` because dot-directories are skipped, while `grep` over the same tree, which passes `--hidden`, finds the file. Three runs over 400 files in 20 directories gave three orders. An invalid glob such as `[` exits 2 with nothing on stdout, and the tool answers `No files found`, because the glob call passes no `pattern` and so skips the invalid-pattern branch. An empty pattern lists every file, up to 100 arbitrary ones.

Output, `glob.ts:53-62`. The last two lines appear only when the count equals the limit:

```text
/abs/project/src/a.ts
/abs/project/src/sub/b.ts

(Results are truncated: showing first 100 results. Consider using a more specific path or pattern.)
```

Zero results print `No files found`. Because `truncated` is `files.length === limit`, a result of 100 prints the notice although the list is complete, and there is no total. (source)

**The one idea worth taking: ask the walker for `limit + 1` and slice, so `truncated` is exact.** The dev branch's `plugin/glob.ts` does it; the legacy set takes `limit + 1` inside the adapter and then loses the information at `glob.ts:51`. It costs one extra line and removes a false notice. (source)

**The one idea worth rejecting: unsorted output from a parallel walker presented as fast file matching.** The first 100 of a larger set differ from run to run, the docs promise a modification-time sort the code never does, and dot-directories are invisible. A model cannot reason about a list whose membership changes between calls. Confidence: high, because the three-orders probe reproduces it. (probe)

### Cline

**Source and name.** `github.com/cline/cline`, tag `v4.1.26` (`a459f1d`, 2026-10-25), and the legacy tag `v4.0.12` (`f267bf4`, 2026-07-28). **There is no glob tool in v4.1.26.** The SDK's tools are `read_files`, `search_codebase`, `run_commands`, `editor` and `apply_patch`, and the model lists files through `run_commands`, whose description says "Use for listing files, checking git status, running builds, executing tests, etc." The SDK maps the classic name `list_files` to `run_commands`. An internal `getFileIndex(cwd)` runs `rg --files --hidden -g '!.git'`, caches for 15,000 ms, and falls back to a directory walk that skips `.git node_modules vendor dist build .next coverage .turbo .cache target out`, but it serves only `search_codebase`'s fallback and @-mentions, never the model directly, and it returns an empty set when the working directory is a home directory or a filesystem root. (source)

The legacy line had `list_files`:

| Name | Type | Required | Default | Description, verbatim |
|---|---|---|---|---|
| `path` | string | yes | none | `The path of the directory to list contents for (relative to the current working directory {{CWD}}){{MULTI_ROOT_HINT}}` |
| `recursive` | boolean | no | false | `Whether to list files recursively. Use true for recursive listing, false or omit for top-level only.` |

There is no pattern parameter, so a model that wants `*.ts` must use `search_files`, whose `file_pattern` takes a glob, or the shell. (source)

| Cap | Value | Unit |
|---|---|---|
| Entries | 200 | files and directories alike |
| Walk time | 10,000 | ms, returning the partial list with only a log line |

**What the legacy tool did** (source: `ListFilesToolHandler.ts:77-104`, `list-files.ts`, `responses.ts:160-220`). It checked `.clineignore`, then walked breadth first with one `globby` call per level, so a truncated list still shows the top-level directories. Default ignores were `node_modules __pycache__ env venv target/dependency build/dependencies dist out bundle vendor tmp temp deps Pods` plus every dot-name unless the target itself was hidden; the root `.gitignore` was read, nested ones as the walk entered each directory, and negation lines were skipped. The root directory and the home directory were refused by returning an empty list, which rendered as the plain string `No files found.`, so the model was told the directory was empty when the tool declined to look. Output was one relative POSIX path per line, directories with a trailing `/`, sorted with `localeCompare(..., {numeric:true, sensitivity:"base"})`, so the order depends on the runtime's locale, and a truncated list ended with `(File list truncated. Use list_files on specific subdirectories if you need to explore further.)`. The sentence also fires when the list holds 200 entries. No output was captured; the legacy tree was not built. (source)

**The one idea worth taking: a breadth-first listing under a hard cap, so truncation degrades into "here are the top-level directories, ask for one".** For a Runvil `Access::glob` that returns entries up to a cap, the order of the cut matters as much as the cap. Confidence: medium, because the source comment states the intent and the legacy walk was not run. (source)

**The one idea worth rejecting: having no listing tool at all, and answering `No files found.` for a refused path.** The first pushes every listing through a shell approval class and a shell dialect that differs by model; the second is a lie about completeness. Confidence: medium, because the SDK-line absence is confirmed in the source but the product reason may be deliberate minimalism. (source)

### Aider

**Source and name.** `github.com/Aider-AI/aider` at commit `5dc9490bb35f` (2026-05-22, `__version__` `0.86.3.dev`). **Aider has none of the six tools, and the model cannot list files at all.** The model never calls a function; it writes edit text in its reply and Aider does the file work. Globbing exists only as input from the human to the slash commands `/add`, `/drop` and `/read-only`, plus the `/ls` listing and the `--read` launch flag. The model finds files only through the repo map and by naming them in prose. Aider's design bet is that a human chooses the files and the model only edits them, which removes the need for a search tool, and moves every discovery problem onto the human and onto prompt text. (source)

| Command | What the user sees | Pattern handling |
|---|---|---|
| `/add` | Add files to the chat so aider can edit them or review them in detail | a named existing file is added directly; otherwise `Path(root).glob(pattern)`, and a directory adds every file under it |
| `/drop` | Remove files from the chat session to free up context space | glob characters `*?[]` use the same glob; otherwise a substring match on chat paths |
| `/read-only` | Add files to the chat that are for reference only, or turn added files to read-only | the literal path first, then `glob.glob` for absolute and `Path(root).glob` for relative patterns |
| `/ls` | List all known files and indicate which are included in the chat session | no pattern; prints everything |

There are no model-visible parameters and no caps on matches, depth or entries. Ignore inputs are `.aiderignore`, `--subtree-only`, and gitignore through `--add-gitignore-files`, which is off by default. (source: `commands.py:765-826, 912, 1064, 1328`, `args.py:416-438`)

**What a pattern does** (source: `commands.py:765-797`). A blank pattern returns nothing, an absolute pattern is taken as one literal path and not expanded, and a relative one runs `list(Path(self.coder.root).glob(pattern))`, with `ValueError` printed as `Error matching {pattern}: {err}`. Every match goes through `expand_subdir`, which turns a directory into every file below it with `rglob("*")`. Matches outside the root are dropped silently. **Last:** in a git repository the result is intersected with the tracked files, the HEAD tree plus staged paths minus ignored ones, so an untracked file never matches a glob and `/add *.py` skips a brand-new file, although naming it explicitly still works. Order is the filesystem's until a later sort: `cmd_add` sorts, and the `/ls` listing of read-only files iterates a set. A failed match is a message such as `No match, and cannot create file with wildcard characters: {fname}`; the output of `/ls` is a complete list. No output was captured. (source)

**The one idea worth taking: build every file listing from the repository's tracked-and-not-ignored set instead of from the disk.** It keeps build output, vendored trees and secrets out of the model's view without a per-call ignore parameter. Confidence: medium, because the same rule hides a brand-new untracked file from globs. (source)

**The one idea worth rejecting: silent drops.** Outside-root and untracked matches vanish from a glob with no count and no message, and `/read-only dir` ignores every ignore rule. A listing should say how many entries it dropped and why. Confidence: high. (source)

### Goose

**Source and name.** `github.com/aaif-goose/goose` (formerly `block/goose`) at commit `ee8500d5e060` (2026-10-28), with tag `v1.56.0` (`9949cf9`, 2026-10-22) for the files that match. The developer extension is compiled into the `goose` crate and exposes exactly `write`, `edit`, `shell`, `tree` and `read_image`, pinned by a test. **There is no glob tool.** The system prompt tells the model to "prefer rg which correctly respects gitignored content", so finding files is `shell` with `rg --files`; the nearest tool is `tree`, a gitignore-aware directory tree with line counts that takes no pattern. (source)

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `path` | string | yes | none | none; the schema has no per-field descriptions |
| `depth` | `u32` | no | 2; `0` means unlimited | none |

```text
List a directory tree with line counts. Traversal respects .gitignore rules.
```

`tree` has the read-only hint, so the `SmartApprove` permission mode lets it through. There is no cap on entries, output, or the whole-file read used for counting; only the agent loop's 200,000-character response cap applies. For `shell`, output is limited to 2000 lines or 50,000 bytes per stream, with the tail kept and the full text saved to a temp file. (source)

**What `tree` does** (source: `dev/tree.rs:33-221`; the walk was not run, the renderer was compiled and run from a verbatim copy). It walks with `ignore::WalkBuilder` with `git_ignore`, `git_exclude`, `git_global`, `ignore` on, `hidden` skipped, `require_git(false)` so ignore files apply even outside a git repository, and `max_depth(depth + 1)`. The tests pin that **`depth` counts one level more than its name says**. A file's count is `fs::read_to_string` then `lines().count()`, and `0` on any error, so a non-UTF-8 file shows `[0]` with no marker; every visited file is read whole, with no size cap, synchronously. Symlinks are omitted without notice. A missing path is `Path does not exist: {resolved}` and a file is `Path is not a directory: {resolved}`. Output sorts directories first, then files, each in byte order, so capitals precede lowercase, and two runs on an unchanged tree are byte identical. A directory cut by `depth` prints `name/  [N]` with no children and no marker, and its `[N]` is an undercount. Sample output, from the compiled renderer with entries inserted by hand (probe):

```text
empty_dir/  [0]
src/  [2K]
  big.rs  [2K]
  lib.rs  [1]
  main.rs  [1]
tests/  [2]
  test.rs  [2]
Zeta.txt  [1]
alpha.txt  [1]
blob.bin  [0]
```

A count of 999 prints `[999]`, 1000 and 1999 print `[1K]`, and 2500 prints `[2K]`. When `rg --files` finds nothing through `shell`, the result is `(no output)` then `Command exited with code 1` as an error. (source, probe)

**The one idea worth taking: list and size in one call.** Per-file and per-directory line counts, a deterministic sort and gitignore awareness let a model budget its reads before opening anything. Confidence: medium; the idea is good and the cost model is not. (source)

**The one idea worth rejecting: the cost model and the vocabulary.** Counting lines by reading every file with no cap, swallowing errors as `[0]`, and a `depth` that is off by one. Confidence: high, because all three are visible in the source and the first two in the probe. (source, probe)

### Comparison table

Each cell is sourced in that harness's section above.

| Dimension | Cursor `Glob` | Claude Code | Copilot | Zed | pi `find` | rig `search` |
|---|---|---|---|---|---|---|
| Tool name | `Glob`, or `glob_file_search` on one branch | `Glob` | `file_search` | `find_path` | `find`; Cursor's port is `pi_find` | `search` with `files: true`; no glob tool |
| Parameter shape | `glob_pattern` + `target_directory` | `pattern` + `path` | `query` + `maxResults`, `path` refused | `glob` + `offset` | `pattern` + `path` + `limit` | `pattern` + `path` + `glob` + `hidden` + `files` |
| Engine | ripgrep `--files` subprocess | ripgrep `--files` subprocess | editor workspace index | in-process `globset` over a project snapshot | external `fd`; the port is a `minimatch` walk | in-process `ignore` crate walk with override globs |
| Matching rule | case-insensitive, `--iglob` | **case-sensitive**, `--glob` | case-insensitive, explicit | `globset` default: always case sensitive, and `*` crosses `/` | `fd` smart case; the port is case sensitive | gitignore-syntax overrides, case sensitive; `src` and `src/` match nothing |
| Recursive prefix | always `**/` if absent | resolved from pattern base | `**/` unless absolute or drive-colon | none; pattern is used as given | slash-less matches at any depth, with `/` gets `**/`; the port loses the recursion | none added; slash-less matches at any depth, a slash anchors to the root |
| Absolute pattern | becomes dead suffix match | supported | converted to a scoped `RelativePattern` | never matches: tested against `root_name/rel`, with no leading slash | kept as is, matched against the absolute path | not applicable: a glob is root-relative |
| Directory results | files only | files only | files only | **files and directories** | files and directories; the port drops the trailing `/` | files only |
| Ignore files | `.gitignore` + `.cursorignore`, with `--cursor-ignore` precedence | **disabled by default** | editor `search.exclude` and `files.exclude` | snapshot's own ignore state; `.env*` force-included | `.gitignore` via `fd`; the port: `.cursorignore` and denies only | `.gitignore`, `.ignore`, excludes, only inside a git work tree; a glob that names an ignored path brings it back |
| Hidden files | always included | always included | index-dependent | included unless ignored | included, `--hidden` | excluded unless `hidden` or the glob names a dot part |
| Sort | mtime desc, in JS, 2000 ms budget, silent fallback | `--sort=modified` in ripgrep, unconditional, **ascending: oldest first** | index order | **byte-wise path order per worktree**, unconditional | `fd` arrival order, unspecified; the port: `readdir` order | walk order, single-threaded, unsorted |
| Cap | 2000 entries / 10000 lines / 5000 chars | 100 results | `maxResults`, default 20 (at least 200 in `FullContext`) | 50 per page | 1000 `limit` and 51,200 bytes | 1000 files |
| Units of the cap | files, lines, characters | results | results | results per page | `fd` results (the port: walked entries), bytes | file names |
| Paging | none | `offset` internal, always 0 | none | **`offset` in the schema** | none; retry with `limit=2N` | none |
| Truncation notice | mid-list `... N more files ...` plus banner | three forms, with count-completeness flag | bare `...` line | `Showing results A-B (provide 'offset' ...)`, also on the last page | `[N results limit reached. Use limit=2N ...]` plus a byte notice | `[stopped at 1000 files; ...]`, also at 1000 |
| Encoding handling | none; raw bytes to string | `--null` delimiter | editor API | `RelPath`, valid Unicode only; the scanner skips non-UTF-8 names | `readline`; non-UTF-8 becomes U+FFFD | `display()`, lossy; no normalization |
| Line-ending handling | explicit `\n`; a filename with a newline splits | `--null`, so no split | not applicable | joined with `\n`; a newline in a name splits | `\r` stripped, then trimmed; a newline in a name splits | not in the sources |
| Stale-read protection | none; Glob keeps no read state | none; does not touch `readFileState` | none | none | none | none; `find` never touches `Seen` |
| Gating | none; read-only, no lock | permission checker, declarative read-only | tool registry | none: `EXCLUDED_TOOLS`, skips the `private_files` filter | none in `find`; the port: `shouldBlockRead` per entry | `place` confinement with a `WorkDir`; one `Reads` footprint |
| Output format | header + `- path` bullets | bare paths | `N total results` + bare paths | `Found N total matches.` + absolute paths | bare relative paths | bare paths, relative to the base |
| Zero-result text | `Result of search in '<p>': 0 files found` | `No files found` | `No files found` | `No matches found` | `No files found matching pattern` | `No matches.` |

Caption: Cursor beside Claude Code, Copilot, Zed, pi and rig.

| Dimension | Cursor `Glob` | zeroclaw `glob_search` | Gemini CLI `glob` | OpenCode `glob` | Cline |
|---|---|---|---|---|---|
| Tool name | `Glob`, or `glob_file_search` on one branch | `glob_search` | `glob` | `glob` | none in v4.1.26; legacy `list_files` |
| Parameter shape | `glob_pattern` + `target_directory` | `pattern` | `pattern` + `dir_path` + `case_sensitive` + two ignore flags | `pattern` + `path` | legacy `path` + `recursive`; or the shell |
| Engine | ripgrep `--files` subprocess | `glob` crate 0.3.4 | npm `glob` 12.0.0 | ripgrep `--files --glob` | legacy `globby`, breadth first |
| Matching rule | case-insensitive, `--iglob` | case sensitive, `*` stops at `/`, no braces | case-insensitive by default; literal if the path exists | gitignore-syntax `--glob`, case sensitive | none; a listing |
| Recursive prefix | always `**/` if absent | none | none | none; slash-less matches at any depth | not applicable |
| Absolute pattern | becomes dead suffix match | admitted under an allowed root, then every hit is dropped | not in the sources | not probed | not applicable |
| Directory results | files only | files only | files only (`nodir`) | files only | files and directories |
| Ignore files | `.gitignore` + `.cursorignore` | none: no `.gitignore`, no `.git` skip | `.gitignore` and `.geminiignore`, plus five fixed excludes | ripgrep's `.gitignore` and `.ignore` | `.clineignore`, a default list, `.gitignore` |
| Hidden files | always included | included | included, `dot: true` | **excluded**, no `--hidden` | dot-names excluded by default |
| Sort | mtime desc, in JS, 2000 ms budget | cut at 1000 in walk order, then a byte-wise sort | modified under 24 hours first, newest first; the rest by `localeCompare` | none; the docs claim mtime | `localeCompare`, numeric |
| Cap | 2000 entries / 10000 lines / 5000 chars | 1000 entries; 50,000 bytes in the loop | none; 64 KiB scheduler cut | 100 file paths | 200 entries; 10 s |
| Units of the cap | files, lines, characters | accepted file entries | bytes | file paths | entries |
| Paging | none | none | none | none | none; breadth first shows top levels |
| Truncation notice | mid-list notice plus banner | `[Results truncated: ...]` plus `Total: N files`, which is the shown count | none at tool level | `(Results are truncated: ...)`, also at 100 | `(File list truncated. ...)`, also at 200 |
| Encoding handling | none; raw bytes to string | non-UTF-8 names skipped | no normalization | UTF-8 decode; backslash becomes `/` | `toPosix`, no normalization |
| Line-ending handling | explicit `\n`; a newline in a name splits | platform separator, `\` on Windows | joined with `\n`; a newline in a name is ambiguous | split on newline | not applicable |
| Stale-read protection | none | none | none | none | none for listing |
| Gating | none; read-only, no lock | 20 actions an hour; path guard; per-hit read check | policy `allow`; workspace check on the root | permission `glob` on the pattern text | `.clineignore` first (legacy) |
| Output format | header + `- path` bullets | sorted relative paths + `Total: N files` | `Found N file(s) ...:` + absolute paths | absolute paths | relative POSIX paths, directories end in `/` |
| Zero-result text | `Result of search in '<p>': 0 files found` | `No files matching pattern '<p>' found in workspace.` | `No files found matching pattern "<p>" within <dir>` | `No files found` | `No files found.` |

Caption: Cursor beside zeroclaw, Gemini CLI, OpenCode and Cline, whose tool is a legacy listing.

| Dimension | Cursor `Glob` | Codex CLI | Aider | Goose |
|---|---|---|---|---|
| Tool name | `Glob`, or `glob_file_search` on one branch | none; `exec_command` with `rg --files`; removed `list_dir` | none; `/add`, `/drop`, `/read-only`, `/ls`, for the human | none; `tree` and `shell` |
| Parameter shape | `glob_pattern` + `target_directory` | `cmd`; removed `dir_path` + `offset` + `limit` + `depth` | none model-visible | `tree`: `path` + `depth` |
| Engine | ripgrep `--files` subprocess | whatever the shell runs; the removed tool walked in Rust | `Path.glob` and `rglob`, then git's tracked set | `ignore` crate walk |
| Matching rule | case-insensitive, `--iglob` | the command's own | `Path.glob` rules; untracked files never match | none; no pattern |
| Recursive prefix | always `**/` if absent | the command's own | none | not applicable |
| Absolute pattern | becomes dead suffix match | the command's own | taken as one literal path | not applicable |
| Directory results | files only | the command's own; the removed tool marked `/` | a directory expands to its files | directories first, with counts |
| Ignore files | `.gitignore` + `.cursorignore` | `rg` applies its own | tracked and not ignored; `.aiderignore` | `.gitignore`, `.ignore`, excludes, even outside a git repository |
| Hidden files | always included | the command's own | not applicable | skipped |
| Sort | mtime desc, in JS, 2000 ms budget | `rg` order; the removed tool byte order | `cmd_add` sorts | `BTreeMap` byte order, directories first |
| Cap | 2000 entries / 10000 lines / 5000 chars | 10,000 tokens of output; removed tool 25 entries | none | none; 200,000 characters in the loop |
| Units of the cap | files, lines, characters | tokens, 4 bytes each; entries | not applicable | not applicable |
| Paging | none | none; removed tool `offset` | none | none |
| Truncation notice | mid-list notice plus banner | `…N tokens truncated…`; removed `More than N entries found` | none | none; a depth cut is silent |
| Encoding handling | none; raw bytes to string | removed tool: lossy names | `str` paths, no normalization | lossy names; non-UTF-8 content counts as `[0]` |
| Line-ending handling | explicit `\n`; a newline in a name splits | not applicable | not applicable | `lines()` counts CRLF and LF alike |
| Stale-read protection | none | none | none; files are re-sent each request | none |
| Gating | none; read-only, no lock | OS sandbox plus approval policy | the human picks the files | permission mode by tool name |
| Output format | header + `- path` bullets | `Chunk ID`, `Wall time`, exit line, `Output:` | printed to the terminal | indented tree with `[N]` counts |
| Zero-result text | `Result of search in '<p>': 0 files found` | the command's own, with an exit line | `No matches found for: <pattern>` for `/read-only` | `(empty directory)` |

Caption: Cursor beside the three harnesses that have no glob tool, and the shell, human or listing mechanism each uses instead.

**Absolute patterns.** Cursor breaks them, Claude Code resolves them, Copilot converts them, pi's `fd` matches them against the absolute path, Zed never matches them (the glob is tested against `root_name/relative`, with no leading slash, and `RelPath::new` rejects an absolute source; source: `find_path_tool.rs:204`, `rel_path.rs:75-77`, derived, no test pins it), and zeroclaw admits one under an allowed root and then drops every hit. Of the harnesses that accept an absolute pattern as input, three handle it and three break it without a message. Cursor is also the harness whose surface models are trained on, so this is the one to fix.

**Sort.** The harnesses order results in at least eight different ways, set out in `### Ordering across the harnesses` below. Only Zed and Goose's `tree` document an order and honor it unconditionally and byte for byte; copy that guarantee and Cursor's choice of key if you want recency.

**Paging.** Only Zed exposes it in the tool schema. Claude Code has the machinery and hides it, and Codex's removed `list_dir` paged by `offset` and `limit`. Gemini, OpenCode, rig, zeroclaw, pi, Cline and Aider have none. That asymmetry suggests the capability is cheap and the surface change is expensive, which is a reason to build it into the schema from the start rather than retrofit it.

**Directories.** Zed and pi (through `fd`) return them from a glob. Cline's legacy `list_files`, Codex's removed `list_dir` and Goose's `tree` list them too, but those are listing tools. The glob tools of Claude Code, Copilot, rig, zeroclaw, Gemini and OpenCode mean "files". If your `Access::glob` can return both, deciding which the tool returns is a design choice; returning both is more useful for structural questions like "where are the test directories."

**An exact fit announces a cut that did not happen.** The stop notice is triggered by reaching the cap, not by exceeding it, in OpenCode (at 100 paths), zeroclaw (at 1000), rig (at 200 hits or 1000 names), pi (`>=` the limit, so 1000 triggers it), Cline's legacy listing (at 200) and Cursor's own `ripgrepTruncated` (at 10000 lines), while Zed's paging hint prints on the last page. Ask the walker for one more than the cap and slice, as OpenCode's dev branch does, and the notice becomes exact.

### Ordering across the harnesses

Order decides whether a result can be replayed. Cursor sorts by modification time under a 2000 ms budget that silently degrades; Gemini sorts recent-first within 24 hours; OpenCode claims to sort and does not; rig's order is the file system's; zeroclaw cuts before it sorts; and pi's `fd` output is unordered. What each means for replay:

| Harness | Order | What it means for replay |
|---|---|---|
| Cursor `Glob` | mtime newest first, in JavaScript, after the search; abandoned silently if the 2000 ms stat budget expires; ties by path segments split on the platform separator | identical runs match only on a warm, small tree; a cold cache or slow disk gives walk order with no marker, and a Windows and a POSIX golden file differ on ties |
| Claude Code | `--sort=modified` in ripgrep, unconditional, ascending, so oldest first (probe) | stable given the mtimes, and under the 100 cap it keeps the oldest 100, which is not what a recency question wants |
| Copilot | the editor search service's order | depends on index state; no guarantee |
| Zed | byte-wise path order per worktree, unconditional; several worktrees in insertion order | byte-identical on an unchanged tree; the only order a test pins |
| pi `find` | `fd` arrival order, with a parallel walker; `--max-results` picks an unspecified subset | two runs can differ, and which N come back is unspecified; pi's own tests sort before comparing |
| Cursor `pi_find` | `readdir` order, depth first, sequential | stable on one file system and different on another; NTFS gave alphabetical order |
| rig | `ignore::Walk` single-threaded directory order; the remote backend and `list` sort | stable on one machine across two calls (probe); local and remote disagree, and a 200-hit cap makes the surviving hits depend on the order |
| zeroclaw | the first 1000 in walk order, then a byte-wise sort | same tree, same output, but which 1000 survive was decided before the sort, and `\` sorts differently on Windows |
| Gemini CLI | files modified under 24 hours ago first, newest first, then `localeCompare` on the full path | depends on the wall clock, since the boundary moves, and on the runtime's locale data, so not replayable across an hour |
| OpenCode | none, although the docs promise mtime order; parallel ripgrep walker | three runs gave three orders; the first 100 of a larger set change between calls |
| Codex CLI | whatever `rg --files` prints; the removed `list_dir` sorted by byte order | the harness never sorts; a middle cut can drop the middle of the list |
| Cline | legacy: `localeCompare` with numeric collation | locale dependent |
| Goose `tree` | `BTreeMap` byte order, directories first | byte-identical on an unchanged tree |
| Aider | none at the model level; `/add` sorts | not applicable |

Caption: the order each harness gives a glob result, and what that means for a replay or a golden fixture.

Only a total order over unique keys with no clock and no budget, as in Zed and Goose, is replayable as is. Everything keyed on mtime is replayable only if the fixture sets the mtimes, and everything that depends on a walker's arrival order or a locale is not replayable at all. Decision 3 and the Q11 recommendation rest on this.

### Zed checkout drift

This report's Zed material was first written against `C:\Users\Vinnie\src\zed` at commit `d9ad6aff67` (2026-08-22, Zed 1.18.0). It is now reconciled with the canonical checkout `C:\Users\Vinnie\cursor\zed` at commit `37faef81f7` (2026-10-27, Zed 1.27.0), by `git diff --no-index --ignore-cr-at-eol` and by reading the code at the new commit. `find_path` itself did not change behavior. Several claims were wrong in both checkouts and are corrected in the Zed section and the tables above; they are not drift. (source, probe)

| What the report said | What is true at `37faef81f7` |
|---|---|
| source `C:\Users\Vinnie\src\zed\...\find_path_tool.rs` | the canonical path `C:\Users\Vinnie\cursor\zed\crates\agent\src\tools\find_path_tool.rs`, 275 lines, 9,103 bytes. The older copy has CRLF endings, which is 275 of its extra 304 bytes (9,407). The remaining diff is 12 lines: `acp` schema `v1` to `v2`, closure and `.to_string()` changes. The matcher, empty-glob rule, paging, strings and test are identical |
| `PathMatcher` at `paths.rs` line 735, `entries` at `worktree.rs` line 2835 | `pub struct PathMatcher` at line 755, `impl PathMatcher` at 778, `is_match` at 858, `entries` at 2842. `PathMatcher::new_lenient` and a private `from_globs` were added and `new` now calls `from_globs`; `PathMatcher::new`, which `find_path` uses, behaves the same |
| `list_directory_tool.rs` is 43 KB | 41,569 bytes (42,757 in the older copy, again mostly CR bytes); the only content change is the `acp` version and one test assertion |
| the tool schema is built per model (not in the report) | one schema through `normalize_tool_schema` for every model (`thread.rs:4341`, `tool_schema.rs:15-32`), which strips `$schema`, `title` and the root `description`; the old checkout called `tool.input_schema(model.tool_input_format())` |
| `RelPath::new` rejects absolute paths (the check behind Zed's absolute-pattern behavior) | still true; the test is now `path_style.has_root(..)` at `rel_path.rs:75-77`, where the older copy used `is_absolute(..)`. Drive-relative Windows paths were not tested |
| `worktree.rs` scanner and `entries` | scanner race fixes, root-moved detection and an external-symlink watch mapping were added; nothing in `entries`, `Traversal` or `absolutize` changed, and the line numbers moved |
| gating by `ToolPermissions`, and matching "case per path style" | not drift, because both were wrong in both checkouts: `find_path` has no permission gate, and matching is always case sensitive |

## What Runvil can support today

Each row maps a behavior this report specified to the `Access` method that would cover it, measured against `runvil-internal/vfs` and the `plugin-std` Pack as they stand.

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

**The mtime gap is the one that decides the tool's shape.** `Access::glob` returns `Vec<String>`. To sort by mtime you need `stat` per match, which is N calls each taking a claim, each firing an event, each potentially failing. For 2000 matches that is 2000 gates, 2000 claim admissions, 2000 events, and 2000 backend round trips, in a synchronous single-threaded runtime. That is materially more expensive than Cursor's `Promise.all` over 2000 stats, because Cursor's are concurrent and yours would be serial and instrumented.

So the design question is not "can I sort by mtime" but "do I want to accept N instrumented stats for a sort." My answer: not by default. Return path order, and add a `stat_many` or a `glob_with_stat` capability only if recency sorting turns out to matter. Confidence: medium-high, because the cost is structural and the benefit is speculative.

**The ignore gap cannot be closed inside the tool.** Runvil's VFS has no ignore concept, and the Pack is not the right place to invent one, because Grep needs the same rules and the two would drift. If you want ignore behavior, it belongs either in a new `Access` capability (something like `glob_filtered(pattern, filters)`) or in a Pack-level walk that owns the rule parser. The VFS-level option is better because the claim and the policy gate then cover the whole walk, but it is a bigger change.

Cursor's ignore handling is three mechanisms in three places - `--cursor-ignore` in the executor's argv, `!**/.git` as a literal glob, and a post-filter in the tool. That is why its precedence is inconsistent. Do not repeat it: keep one place and one documented precedence.

**The case-sensitivity gap is a specification bug.** `glob.rs` matches bytes, so `*.js` does not match `B.JS`. Cursor matches case-insensitively. If you keep `glob.rs` as the matcher and keep Cursor's surface, the same call produces different results. Three ways out: make `glob.rs` case-fold (cheap for ASCII, expensive and locale-dependent for Unicode), have the Pack rewrite patterns into a case-tolerant form the grammar can express (impossible without character classes), or accept the divergence and document it. I would accept it and document it, because case-insensitive path matching is a surprising default and Cursor's choice there is arguably a mistake. Confidence: medium, because model familiarity is a cost.

**Rust mechanisms the other harnesses show.** Rig, Zed and zeroclaw hand you named types to borrow, and each comes with a condition. `globset::GlobBuilder`, with `literal_separator(true)` and `case_insensitive` set on purpose, is what Zed's `PathMatcher` wraps (Zed leaves both off), and `globset::GlobSet` matches many patterns in one pass. `ignore::WalkBuilder` with `ignore::overrides::OverrideBuilder` is rig's walk and glob layer and would serve a real-disk backend, but its override semantics are the ones behind Trap 1 and Trap 15, so a Runvil that wants ignore rules to beat the pattern must apply them as a filter after matching and not as overrides. Rig's `Listing` enum (`Visible`, `Hidden`, `All`) is a shape for the VFS's ignore decision, and `Workspace::files(root, glob, listing)` is the trait method a Pack-level search would call. Zed's `RelPath`, valid Unicode with no absolute form, is a better result type than `String`. Zeroclaw's `glob` crate is the counter-example: no braces, and a root spliced into the pattern. (source)

## What remains unknowable

Things the evidence cannot reach, stated rather than guessed.

**Whether the indexed path ever serves a Glob call.** `executeIndexedGrep` is tried before the ripgrep path and falls back when the provider returns undefined. Whether the provider answers a `files_with_matches` call with an empty pattern is a runtime property of the extension host and does not appear in the bundle. If it does, in-workspace Glob results may come from an index rather than a walk, which would mean they respect index ignore rules, may include unsaved buffers, and may be stale by up to the index's refresh interval. Cursor's own staff statement that Instant Grep "was not changed by this update" implies the indexed path can serve and therefore may still have the pre-fix behavior. This is the largest unresolved question in the report.

**What `useStreamingGrep` defaults to in a real session.** The flag is injected at the resource registration site, `streamingEnabled` returns `true` when the provider is undefined, but the configuration value is not in the bundle. It matters because the entire mtime sort, the `--line-buffered` flag, and the `sortsAfterRipgrep` decision all branch on it. If streaming is off, the non-streaming path passes `--sortr modified` to ripgrep and the sort is unconditional and done by ripgrep.

**How realpath behaves on Windows junctions versus symlinks.** `en()` is a realpath wrapper whose failure produces the long "could not be verified" message. Junction handling is not distinguishable from the bundle, and a Windows junction inside a repository is common enough (package managers create them) that the behavior matters.

**Whether `.cursorignore` files are discovered from nested directories or only the root.** `listCursorIgnoreFilesByRoot` returns a list per root, but where it searches and how it merges nested files is in the ignore service, which is not in this bundle. The precedence documentation says later files win, so ordering within that list matters and is not determinable here.

**The byte-level behavior of the ignore service's own glob parsing.** Whether it uses ripgrep's gitignore parser, a JavaScript one, or something else determines whether edge cases like trailing-whitespace escaping and `**` in the middle of a segment behave identically. Not visible.

**Whether a filename with a newline splits in the shipped parser.** Derived from the parser's shape, not measured, because NTFS refuses to create such a file and I did not have a POSIX filesystem available. The parser scans to `\n` and takes the prefix, so the split is near-certain, but it is derived.

**Whether ripgrep's walk order is stable for a given tree on a given machine.** My probe produced different orders across invocations, which suggests not, but ripgrep's parallel walker could be stable per-machine at a given thread count. Either way the order is not specified, which is enough to say it is not safe to depend on. Rig's single-threaded walk over the same crate was stable across two calls (probe, NTFS), which fits the reading that the parallel walker is the source of the variation. It was a different tree and a different build, so it does not settle ripgrep's own behavior.

**What the Smart Mode classifier does with the static `Glob` schema.** The short description without the batch bullet is projected into a static array at bundle offset `4673861`, but how the classifier weights it is not in this file.

**How a model's training represents `Glob`.** The description says `glob_pattern` and `target_directory`, but models were also trained on Claude Code's `pattern` and `path`, and on the `dsv3-1018` `glob_file_search` variant. Which name a given model reaches for first is an empirical question no static read answers, and it is the one input to Decision 1 that cannot be resolved from the artifacts. If it matters, run the same task against several models and count the parameter errors.

**What the other harnesses leave unverified.** Each item is a claim above that rests on reading the code or documentation.

| Harness | Not verified |
|---|---|
| pi `find` | `fd` was not run, so smart case in `--glob` mode, the trailing slash on directories, an empty glob, `--max-results 0`, whether `.git` contents appear under `--hidden`, and fd's stderr text come from its man page or are `not in the sources` (web) |
| Cursor `pi_find` | which pi release was ported, what sets `enablePiTools`, and `minimatch` 10.2.5, the bundled version; the probes used 9.0.9 |
| Claude Code | the oldest-first order is derived from the argv, the slice and a probe, not seen end to end; `Z6t`, which fixes the path form, was not extracted |
| Zed | no live run and no captured schema, so every formatted string and failure text is derived; panic behavior near `usize::MAX`; a remote Windows project opened from a Unix host; UNC globs; whether `**/.env*` lists `.env` at runtime; how soon a just-written file appears in a snapshot |
| rig | the remote path was not run; file and directory symlinks could not be created (error 1314); order on ext4 or APFS; the 155 ms walk time on NTFS is unexplained against the 5.6 ms in rig's own commit message |
| zeroclaw | the real tool was not built or run; the dead absolute-pattern path and the Unix symlink tests are source reading only |
| Gemini CLI | whether an absolute or `../` pattern lists files outside the workspace; whether the context-management path changes the 64 KiB rule |
| OpenCode | ripgrep's `.gitignore` handling outside a git repository; whether the TUI drives the V2 core tools |
| Codex CLI | ripgrep's own semantics (sort, exit code 2 with partial output) and the raw exec output cap before truncation |
| Cline, Aider, Goose | Cline's legacy listing was not run; no real Aider output was captured; Goose's walk was not run, only its renderer |

Caption: what each harness's section above could not settle.

## Conformance checklist

The parent spec's twenty items for Glob, from `## Conformance checklist` at line 4780, each marked confirmed or corrected against the shipped 3.27.12 build.

| # | Item | Verdict | Reason |
|---|---|---|---|
| 1 | Parameters are exactly `glob_pattern` and `target_directory`, with no `pattern`, `case_sensitive`, or `max_results` | confirmed | Both schema variants have exactly those two fields, at offsets `4289658` and `4290208` |
| 2 | `dsv3-1018` describes the target as `Cursor workspace roots` in the plural plus a trailing newline, while every other branch uses the singular | **corrected** | The plural is in `MEe`'s target description; the trailing newline is in `MEe`'s `glob_pattern` description, not the target description. The singular/plural half is exact; the newline lands on the other field |
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

Sixteen confirmed, four corrected. Items 14 and 15 are the same mistake: the parent spec read the argv and the executor's branch structure but did not run the binary, so it inferred `-g`/`--iglob` precedence from the shape of the code rather than from ripgrep's documented and probed behavior.

Item 13 is a conditional in the code that the spec flattened into an absolute: there are two argv shapes, and which one runs is a configuration value not present in the bundle. Item 2 puts a trailing newline on the wrong field.

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

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, original identifiers and embedded source paths. Used for names, not for exact strings.

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
| `16825933` | `function truncateHead(`; `formatSize` is at `16825492` and prints whole kilobytes (`50KB`) |
| `16828361`, `16828386` | `PI_DEFAULT_MAX_LINES = 2e3` and `PI_DEFAULT_MAX_BYTES = 50*1024` |
| `13912670` | `findSchema` (`pattern`, `path`, `limit`) |
| `13896012` | `function renderPiFind`, `renderPiResult`, `extractResultText` |
| `17018419` | `LocalPiFindExecutor` resource registration |

### Extraction artifacts

The working extracts behind the bundle slices above. They are disposable and are named here by what they are.

- `GLOB.js` - the prettified slice of the bundle, whose header names `slice=[start,end]`. Read in full.
- `raw-glob-ls.txt` - the shared Glob and ListDir region.
- `ripwalk-repro.js`, `ripwalk_15490200.txt`, `ripwalk_15492500.txt` - the ripwalk argv and its cache, used for the contrast in Q7.
- `globtest.js`, `dotcheck.js`, `lencheck.js` - earlier probes.
- `BRIEFING.md` - the identifier-mangling map.

This report's own probe scripts and extracts (`rgprobe.js` through `rgprobe10.js`, listed under Probes) were scratch files and are not reproduced.

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

- Claude Code 2.1.311 (win32-x64): the `glob.js` and `limits.js` regions extracted from `claude.exe` in the npm package `@anthropic-ai/claude-code-win32-x64` 2.1.311, with `glob.js` covering bytes 256099961 to 256108320. Tool name `Glob`; search core `m6t`; notice builder `hGo`; registration `q0`.
- VS Code Copilot, vscode commit `b208bc25` (2026-10-25): `src\vscode\extensions\copilot\src\extension\tools\node\findFilesTool.tsx`, `package.json` for the model description and schema, `findTestsFilesTool.tsx`, `test\findFiles.spec.tsx`, `node\toolUtils.ts` (`inputGlobToPattern`), `common\toolNames.ts` lines 25 and 46.
- Zed, canonical checkout `C:\Users\Vinnie\cursor\zed` at commit `37faef81f7` (2026-10-27, Zed 1.27.0): `crates\agent\src\tools\find_path_tool.rs`, with `crates\util\src\paths.rs` for `PathMatcher` (struct at line 755, `is_match` at line 858) and `crates\worktree\src\worktree.rs` for `entries` (line 2842). An earlier version of this report cited the checkout `C:\Users\Vinnie\src\zed` at `d9ad6aff67` (2026-08-22, Zed 1.18.0), where `PathMatcher` sat at line 735 and `entries` at line 2835.

`(source)`, `(web)` and `(probe)`, the harnesses added to the comparison:

- pi, `C:\Users\Vinnie\cursor\pi` at commit `a92283056` (2026-10-27): `packages/coding-agent/src/core/tools/find.ts`, `ls.ts`, `truncate.ts`, `path-utils.ts`, `src/utils/tools-manager.ts`, and the tests `test/tools.test.ts`, `3302-find-path-glob.test.ts` and `6104-find-root-relativization.test.ts`. The fd man page, https://raw.githubusercontent.com/sharkdp/fd/master/doc/fd.1, `(web)`.
- rig, `C:\Users\Vinnie\cursor\rig` at commit `a83b774d0` (2026-10-25): `crates/rig-tools/src/search.rs`, `list.rs`, `workspace.rs`, `fs.rs` and their tests, with the locked crates `ignore` 0.4.25, `globset` 0.4.18 and `regex` 1.12.3 read from the cargo cache. `(probe)`: seven standalone programs built from verbatim copies against those crates, on Windows.
- zeroclaw, `C:\Users\Vinnie\src\zeroclaw` at commit `2d85371f3` (2026-09-18): `crates/zeroclaw-tools/src/glob_search.rs` and `wrappers.rs`, with the `glob` crate 0.3.4. `(probe)` for the cap, sort and `ws[1]` cases.
- Codex CLI, https://github.com/openai/codex at commit `bea47c06dfac` (2026-10-28), and the removed `list_dir` at its parent `9d579813bb`, `(web)`.
- Gemini CLI, https://github.com/google-gemini/gemini-cli tag `v0.66.0`, commit `970413207732f5d4d8c50540c7fc359c44fdaaf2`: `packages/core/src/tools/glob.ts` and `constants.ts`; docs https://geminicli.com/docs/tools/file-system/, `(web)`.
- OpenCode, https://github.com/anomalyco/opencode tag `v1.19.5`, commit `810c0eba1a7348036f8bdf61e6632d0c36c6876c`, with dev `fa1260d6ca`: `packages/opencode/src/tool/glob.ts` and `packages/core/src/ripgrep.ts`; docs https://opencode.ai/docs/tools/, `(web)`.
- Cline, https://github.com/cline/cline tag `v4.1.26` (`a459f1d`) and legacy `v4.0.12` (`f267bf4`); docs https://docs.cline.bot/tools-reference/all-cline-tools, `(web)`.
- Aider, https://github.com/Aider-AI/aider at commit `5dc9490bb35f9729ef2c95d00a19ccd30c26339c` (2026-05-22), `(web)`.
- Goose, https://github.com/aaif-goose/goose at commit `ee8500d5e060eea6f2bcee92e8a5e351f12c3a70` (2026-10-28) and tag `v1.56.0` (`9949cf9`), `(web)`, with `(probe)` for the compiled renderer.
- ripgrep sort direction, `(probe)`: `--sort=modified` and `--sortr=modified` against `rg.exe` 15.1.0-cursor5. The post-processor `Uzr` in `claude.exe` at byte 252365564, `(source)`.

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

- [Glob does not work the way agents expect it to work](https://forum.cursor.com/t/glob-does-not-work-the-way-agents-expect-it-to-work/168477) - the October 2026 report that Glob ran `rg -l --iglob PATTERN` and therefore dropped binary and empty files. Cursor's staff confirmed it and said the fix made Glob match "files by name, not by content." It is the most valuable web source in the set, because it independently corroborates the `--files` versus `-l` fork found in `buildArgs`.
- [Cursor Agent tools documentation](https://cursor.com/docs/agent/tools/search) - describes Instant Grep and does not document Glob's parameters, sort order, or limits at all. Its absence means there is no authoritative public specification to conflict with.
- [Cursor Agent overview](https://cursor.com/docs/agent/overview) - generic tool descriptions, no per-tool schema.
- [Leaked Cursor Agent prompt 2.0](https://github.com/x1xhlol/system-prompts-and-models-of-ai-tools/blob/main/Cursor%20Prompts/Agent%20Prompt%202.0.txt) - contains the `glob_file_search` block matching the `MEe` schema verbatim, confirming that variant was in production with the plural `roots`.
- [Cross-harness glob survey](https://github.com/avifenesh/tools/blob/main/agent-knowledge/glob-impl-and-prompts-in-major-tools.md) - the source of the widely repeated but incorrect "100 results" figure for Cursor.

Where web and bundle disagree, the bundle wins, and every such conflict is recorded in the sections above rather than repeated here.

### Where the shipped build disagrees with the parent spec

1. **Positive globs override `.gitignore` at file level.** The spec says gitignore wins over a positive glob. ripgrep's `--help` says `-g`/`--iglob` "always overrides any other ignore logic," and the probe confirms a gitignored `*.log` is returned by `glob_pattern: "*.log"`. Spec offset `2417`, probe `rgprobe3.js` through `rgprobe7.js`.
2. **`**/*` resurrects directory-level ignored subtrees** by matching the directory node itself. Not stated in the spec at all. The spec's own recommended "match every file" pattern is the one that does it.
3. **`.gitignore` is inert outside a git repository.** `--no-require-git` is not passed, so a plain directory's `.gitignore` does nothing. The spec does not mention the git-repository precondition.
4. **`.cursorignore` is not "dedicated pairs plus a post-filter, not ripgrep's ignore machinery."** It is a patched `--cursor-ignore` flag that behaves "like `--ignore-file` except for its higher precedence," and it beats `--iglob`. The spec's description at `2444` inverts the mechanism, and the inversion hides the fact that the two ignore sources have different precedence.
5. **The sort flag is conditional, not absent.** The spec says "No sort flag." On the streaming path that is true; on the non-streaming path the executor emits `--sortr modified`. Which runs depends on `useStreamingGrep`, whose default is not in the bundle.
6. **The tie-break splits on the platform separator**, so a tie resolves differently on Windows and POSIX. The spec says "path segments ascending" without noting the platform dependence.
7. **`--line-buffered` is passed on the streaming path.** The spec's argv listing at `2372` omits it.
8. **The `dsv3-1018` trailing-newline difference is on `glob_pattern`, not `target_directory`.** The spec's conformance item 2 puts it on the target description, which has the singular/plural difference but not the newline.
9. **Offsets.** `GLOB` is at `4292676`, not `4277336`. The executor region is around `9158000..9174000`, not `9017686..9034500`.
10. **`pi_find` is a different engine from Glob, and it is not pi's engine either.** Upstream pi's own `find` wraps the external `fd` binary (source: pi `find.ts:172-216`). Cursor's port uses `minimatch` over a JavaScript `readdir` walk with hardcoded `.git` and `node_modules` skips, counts walked entries against `limit`, and reports `No files found matching pattern` rather than a header. The spec does not cover it.

*2026-10-27 - deepseek-v41-flash-1017; enriched with the other harnesses 2026-10-28 - claude-sonnet-5-5*
