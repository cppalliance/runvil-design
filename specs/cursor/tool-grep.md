# The `Grep` file tool

Reverse engineered from the shipped Cursor 3.27.12 agent bundle, the unminified daemon twin, and probes against the shipped `rg.exe`. Bundle path:

```
C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js
```

9,814,287 bytes, one physical line, read as latin1. Every offset in this report is from that build. Where the parent spec's offsets differ, both are given. Claims marked `(bundle)` come from that file, `(daemon)` from `agent-host-daemon\dist\bin\daemon.cjs`, `(probe)` from a measurement I ran, `(source)` from the source of the other harnesses on disk (Claude Code, Copilot, Zed, pi, rig, zeroclaw, Codex CLI, Gemini CLI, OpenCode, Cline, Aider, Goose; each with its commit or version under `## Sources`), `(web)` from public sources.

Offset map for this build, so you can re-slice any of it yourself:

| What | Current offset | Parent spec offset |
|---|---|---|
| `AG("GREP", ...)` registration | 4319512 | 4304168 |
| Factory `_Ie` | 4315448 | 4300107 |
| Schema `wIe` (default branch) | 4311441 | 4296095 |
| Schema `vIe` (`dsv3-1018`) | 4313434 | 4298093 |
| Description switch | 4315866 | 4300513 |
| Broad-glob set `uIe` | 4310786 | not given |
| Invalid-args builder `dIe` | 4310816 | 4295476 |
| Boolean coercion `hIe` | 4311134 | 4295794 |
| Int clamp `fIe` | 4311239 | 4295899 |
| Cache class `VEe` | 4300579 | not given |
| Cache key `KEe` | 4300217 | not given |
| Char budget `YEe` | 4301037 | 4285697 |
| Footer `QEe` | 4302018 | 4286678 |
| No-result `XEe` | 4303234 | 4287894 |
| Mode renderer `ZEe` | 4303547 | 4288207 |
| Envelope `tIe` | 4306800 | 4290918 |
| Pagination entry `rIe` | 4307544 | 4292204 |
| Array slice `nIe` | 4308254 | 4292914 |
| Content slice `sIe` | 4308410 | 4293070 |
| Files slice `oIe` | 4308949 | not given |
| Count slice `iIe` | 4309184 | not given |
| Error classifier `IEe` | 4286956 | 4271616 |
| Errno set `xEe` | 4286847 | 4271507 |
| Executor class `Lx` | 9157073 | 9017630 |
| `buildArgs` | 9158354 | 9018911 |
| Content parser `Ax` | 9153344 | 9013903 |
| Files parser `Cx` | 9152226 | 9012785 |
| Count parser `Px` | 9152663 | 9013222 |
| `executeRg` | 9167020 | 9027573 |
| `execute` | 9169464 | 9030017 |
| Class constants | 9173743 | 9034296 |
| `pi_grep` registration | 4561631 | not in the spec |
| `LocalPiGrepExecutor` | 9202537 | not given |

The behavior did not change between the two builds. The entire delta is insertion above the region. I re-derived the argv, the parsers, the renderer, the pagination, and the classifier from the current build and got the same logic the spec and the research describe, with the exceptions listed in the closing section.

## The short version

- **Only Cursor, Claude Code and Zed page at all, and the three do it differently.** Cursor and Claude Code take a global `offset` and `head_limit` over a search that is re-run on every call (Claude Code) or cached for five entries (Cursor). Zed pages with `offset` and a fixed 20, re-runs the search on every page, and counts two units: `offset` skips raw regex ranges while the page of 20 counts merged context blocks, so after a merge page 2 repeats page 1 (hand trace, not run). The other nine harnesses with a search path (Copilot, pi, rig, zeroclaw, Gemini CLI, OpenCode, Cline, Codex's removed `grep_files`, and Goose through the shell) have a cap and no resume, and pi's trailer says `Use limit=2N`, which restarts the search; Aider has no search tool. The implication: a page that neither repeats nor skips needs a stable order, one counting unit, and a snapshot or a deterministic search. Cursor has the order only when its mtime sort finishes; Zed has the order and not the unit. See `### How each harness pages`. (bundle, source)
- **`head_limit` is a client-side counter over parsed matches, global across every file.** It is not `--max-count`, and `--max-count` appears zero times in the entire 9.8 MB bundle. If you emit `--max-count` you get a per-file limit, which looks right on a single-file search and is wrong everywhere else.
- **`head_limit: 0` means unlimited**, via `fIe(r.head_limit, Math.ceil) || void 0`. And there is **no default** `head_limit` at all. Claude Code defaults to 250; Cursor does not. Omitting it means the only ceiling is 2000 parsed entries.
- **The schema his crate ships today is wrong in five ways.** It has `-n` and `context`, it describes `-C` as an alias for `context`, it says `output_mode` defaults to `files_with_matches`, and it says `head_limit` defaults to 250. The bundle has none of those. See `### The schema his crate ships today is wrong`.
- **`-A` and `-B` silently defeat `-C`.** `-C` is forwarded as `--before-context N` and `--after-context N`, but only when neither `-A` nor `-B` is present. `-C` alone works correctly. `-C: 3, -A: 1` gives you one line of after-context and no before-context, with no warning.
- **The wire format separates the path from the line number with a NUL byte, not a colon.** `-0` is passed in content mode, so a line arrives as `<path>\0<line>:<content>\n`. A parser that splits on `:` truncates every path, and an absolute Windows path like `C:\src\a.ts` becomes `C`. This is the most fragile point in the tool and the reason `-0` exists.
- **Two output-shape facts that byte-exact tests catch.** The renderer never emits the trailing blank line the spec's worked examples show: the body is `.trim()`ed before the envelope is assembled, so the last file's group ends flush against `</workspace_result>`, with a blank line *between* files and none at the end. And `path` is rewritten to a workspace-relative string before it reaches ripgrep, so on Windows the model sees `.\src\a.ts` and not `src\a.ts`; Grep does not strip the `.\`, unlike Glob which strips `./`.
- **The truncation trailer's `at least` is the only signal for a `head_limit` cut.** `[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]` is the text, and there is **no** `Results truncated. ` prefix on that case, because the page was full. The prefix appears only when a cap hid content before the page end. The parent spec's example omits `at least` entirely, so a byte-exact test would fail on it.
- **`-i` emits `--ignore-case`; it does not omit `--case-sensitive`.** The code is `!0===t.caseInsensitive ? s.push("--ignore-case") : s.push("--case-sensitive")`, so exactly one of the two is always on the wire. The spec's claim that `-i` works "by absence" is wrong, and its parenthetical about ripgrep's default is wrong twice over: ripgrep's default is case-sensitive, and smart-case is opt-in.
- **`--max-columns 1000` decides in bytes, and the preview it shows is 1000 characters.** A 999-byte line passes untouched; a 1000-byte line is cut. A 334-character CJK line is 1002 bytes and gets cut. The preview then keeps 1000 *characters*, which for CJK is 3000 bytes. Two different units inside one feature, and both differ from the 15000-character render budget, which counts UTF-16 code units.
- **A binary file is invisible in content mode and visible in files and count mode.** ripgrep prints `bin.dat: binary file matches (found "\0" byte around offset 30)` with no NUL separator, so the content parser skips the line. `-l` and `-c` print the path normally, so the same file appears there. Nothing in the tool decides this; it falls out of the parser.
- **`--follow` is always on, so a symlink can walk the tool off the workspace.** A symlink to `/` makes ripgrep traverse the whole filesystem. `filterBlockedFiles` removes the *results* after the fact, but the read already happened. There is no parameter to disable either `--follow` or `--hidden`.
- **The output is only deterministic when the mtime sort completes.** Files sort newest-first by `mtimeNs` with a 2000 ms stat budget. On timeout the order is ripgrep's parallel walk order, which varies run to run. A large search can therefore produce different bytes on two identical runs. Cursor is in company: with a hit cap and no sort flag, pi gave 8 distinct outputs in 8 runs on 20,000 files, OpenCode 4 in 4, zeroclaw's `rg` backend 12 in 12, and Cline 6 in 6, so which hits survive the cap is arbitrary. Only Zed (worktree id, path, match offset) and zeroclaw's internal backend (sorted walk) are deterministic by design. (probe, source)
- **There is a 5-entry result cache keyed on everything except the tool call id and the pagination.** Two calls with the same search but different `head_limit` share one execution, so the second call sees the first call's snapshot even if files changed in between.
- **A lone-CR file is one line to this tool.** The line iterator splits on `\n` only and strips a trailing `\r`. Classic Mac line endings produce a single enormous match with line number 1.

### The schema his crate ships today is wrong

`C:\Users\Vinnie\cursor\runvil\crates\plugin-std\schemas\grep.json` has drifted from what the bundle offers. Five divergences, each with the corrected value. This is the file to edit.

#### 1. It declares `-n`, which does not exist

Shipped file:

```json
"-n": {
  "type": "boolean",
  "description": "Show line numbers in output. Requires output_mode: \"content\", ignored otherwise. Defaults to true."
}
```

The bundle has no `-n` field in either schema. Line numbers are unconditional in content mode because `buildArgs` always pushes `--line-number`. A model that sets `"-n": false` on his build expects no line numbers and gets them anyway.

**Fix: delete the field.**

The field came from Claude Code. Claude Code's `Grep` does have `-n` (`"-n": vE(M().optional()).describe('Show line numbers in output (rg -n). ...')`), and Cursor's `output_mode` description still says "supports -A/-B/-C context, -n line numbers, head_limit" as a leftover. The parameter itself is gone. Cursor inherited the description text from Claude Code and removed the field.

#### 2. It declares `context`, which does not exist

Shipped file:

```json
"context": {
  "type": "number",
  "description": "Number of lines to show before and after each match. Requires output_mode: \"content\", ignored otherwise."
}
```

The bundle has no `context` field. `-C` is the symmetric-context parameter, and its description is:

```
Number of lines to show before and after each match (rg -C). Requires output_mode: "content", ignored otherwise.
```

**Fix: delete `context`, and give `-C` the bundle's description.**

Note there *is* a `context` field in the internal `GrepArgs` proto message (`7 context 5?`), which is why the executor's `buildArgs` has a `t.context` branch at all. But no model-facing schema field populates it, so the branch is dead on the `GREP` path. Glob does not reach it either, since Glob passes an empty pattern and no context. The only producer would be an internal caller.

#### 3. It describes `-C` as an alias for `context`

Shipped file:

```json
"-C": {
  "type": "number",
  "description": "Alias for context."
}
```

**Fix: replace with the bundle's text.**

```
Number of lines to show before and after each match (rg -C). Requires output_mode: "content", ignored otherwise.
```

This matters beyond wording. If `-C` were an alias for a separate `context` field, then `-C` and `context` would be two spellings of one parameter and setting both would be ambiguous. In the bundle they are one parameter, and `-C` is the only spelling.

#### 4. It says `output_mode` defaults to `files_with_matches`

Shipped file:

```json
"output_mode": {
  "type": "string",
  "enum": ["content", "files_with_matches", "count"],
  "description": "... Defaults to \"files_with_matches\"."
}
```

The bundle's default is `"content"`, in both the `.default("content")` call and the sentence at the end of the description. And the executor independently defaults:

```js
const i=t.outputMode??"content",...
```

so the default survives even if the schema default were removed.

**Fix: change the sentence to `Defaults to "content".` and add `"default": "content"` to the JSON Schema.**

This is the most consequential of the five, because it changes what a model gets when it omits `output_mode`. `files_with_matches` returns paths only; `content` returns lines. A model that omits the field expecting paths gets lines.

This default also came from Claude Code. Claude Code's `Grep` defaults to `files_with_matches`, and his file appears to have been written against the Claude Code surface.

#### 5. It says `head_limit` defaults to 250

Shipped file:

```json
"head_limit": {
  "type": "number",
  "description": "Limit output to first N lines/entries. Works across all output modes: content (limits output lines), files_with_matches (limits file paths), count (limits count entries). Defaults to 250 when unspecified. Pass 0 for unlimited (use sparingly - large result sets waste context)."
}
```

There is **no** default `head_limit`. The bundle's description is:

```
Limit output size. For "content" mode: limits total matches shown. For "files_with_matches" and "count" modes: limits number of files.
```

And the code is `fIe(r.head_limit, Math.ceil) || void 0`, so an absent `head_limit` stays `undefined` and the client-side slice is skipped entirely.

**Fix: replace the description with the bundle's text.** Do not add a default.

The "Pass 0 for unlimited" half happens to be right, but for the wrong reason. Cursor's `0` means unlimited because of the `|| void 0` falsy coercion, not because of an explicit `0` check. The phrase "use sparingly - large result sets waste context" is Claude Code's, which writes an em dash there where the shipped file has a hyphen.

#### The corrected file, in full

```json
{
  "wire_name": "Grep",
  "parameters": {
    "type": "object",
    "properties": {
      "pattern": {
        "type": "string",
        "description": "The regular expression pattern to search for in file contents"
      },
      "path": {
        "type": "string",
        "description": "File or directory to search in (rg pattern -- PATH). Defaults to the run's filesystem root."
      },
      "glob": {
        "type": "string",
        "description": "Glob pattern to filter files (e.g. \"*.js\", \"*.{ts,tsx}\") - maps to rg --glob"
      },
      "output_mode": {
        "type": "string",
        "enum": ["content", "files_with_matches", "count"],
        "default": "content",
        "description": "Output mode: \"content\" shows matching lines (supports -A/-B/-C context, -n line numbers, head_limit), \"files_with_matches\" shows file paths (supports head_limit), \"count\" shows match counts (supports head_limit). Defaults to \"content\"."
      },
      "-B": {
        "type": "number",
        "description": "Number of lines to show before each match (rg -B). Requires output_mode: \"content\", ignored otherwise."
      },
      "-A": {
        "type": "number",
        "description": "Number of lines to show after each match (rg -A). Requires output_mode: \"content\", ignored otherwise."
      },
      "-C": {
        "type": "number",
        "description": "Number of lines to show before and after each match (rg -C). Requires output_mode: \"content\", ignored otherwise."
      },
      "-i": {
        "type": "boolean",
        "default": false,
        "description": "Case insensitive search (rg -i) Defaults to false"
      },
      "type": {
        "type": "string",
        "description": "File type to search (rg --type). Common types: js, py, rust, go, java, etc. More efficient than glob for standard file types."
      },
      "head_limit": {
        "type": "number",
        "minimum": 0,
        "description": "Limit output size. For \"content\" mode: limits total matches shown. For \"files_with_matches\" and \"count\" modes: limits number of files."
      },
      "offset": {
        "type": "number",
        "minimum": 0,
        "description": "Skip first N entries. For \"content\" mode: skips first N matches. For \"files_with_matches\" and \"count\" modes: skips first N files. Use with head_limit for pagination."
      },
      "multiline": {
        "type": "boolean",
        "default": false,
        "description": "Enable multiline mode where . matches newlines and patterns can span lines (rg -U --multiline-dotall). Default: false."
      }
    },
    "required": ["pattern"]
  }
}
```

I have kept the `path` description's substitution to "the run's filesystem root", since that is the edit `EDITS.md` already records and it is correct for his crate. Everything else is the bundle's own text. `EDITS.md` should gain a line noting that the `-n`, `context`, and 250-default fields were removed because they never existed in the Cursor schema, and that the `output_mode` default was corrected from `files_with_matches` to `content`.

One thing the corrected file does not fix, and cannot: the `output_mode` description still says it "supports -n line numbers". That is Cursor's own text and it is stale. If you want a description that does not name a parameter that does not exist, strike `, -n line numbers` from it. I would leave it, because byte-fidelity to Cursor's text is the premise of the crate, and the sentence sits inside the description the model reads for what `content` shows; it does not claim that a `-n` field exists.

## Decisions you have to make

Eleven forks. The first is the one the whole six-tool set turns on, so it gets the most space. The rest are ordered by how much they constrain the implementation, and the last, D11, comes from what the other harnesses do with patterns.

### D1. Does the VFS gain a search capability, or does the Pack walk?

This is the blocking question in your plan and it is not a detail. `Access` has read, read_string, write, append, str_replace, remove, exists, glob, list, stat, mkdir, rename, copy. Nothing searches content. So either you add a capability, or the Pack composes a search out of `glob` plus `read_string`.

The two options are not close in cost, and the difference lies in the claim granularity and the memory profile rather than the CPU.

**Option A: a `search` capability on the VFS.**

The capability takes a pattern, a set of file filters, and a budget, walks the backend's own storage, and streams matches out. Signature roughly `fn search(&self, request: &SearchRequest) -> Result<SearchMatches, VfsError>`.

What it provides:

- One claim, taken once, covering the search as a whole. This is the only way to get a *correct* claim. A Pack-level walk takes N claims, one per file, and N is unbounded, so the claim table grows with the tree. On a 200,000 file tree a `glob`-based search either claims 200,000 paths, which is a lot of table, or claims something coarse like "the whole subtree", which is what `Claims::claim_subtree` already does for recursive delete. If the right answer is "claim the subtree", you have already got the primitive and you do not need per-file claims.
- The backend can skip the read. A memory backend holds bytes it can search in place. A real backend can hand the request to a `grep`-equivalent crate that memory-maps and never materializes the file. The Pack cannot do either, because `read_string` must return an owned `String`.
- One place to enforce the budget. The capability counts entries as it finds them and stops. The Pack counts after the fact, which means it has already done the work for everything it is about to throw away.
- The event is one event. `fire(Op::Search, &path)` once, which is what an observer wants. A Pack walk fires 200,000 `Read` events, which will swamp whatever consumes them.

What it costs:

- A new backend trait method, so every backend including the in-memory one and any test doubles must implement it. You have a small number of backends so this is bounded, but it is a breaking change to `traits::backend.rs`.
- A regex engine dependency inside the VFS crate, or a matcher trait the VFS passes through. `regex` is already a workspace dependency for work modeled on Zed, so this is cheap, but it does pull the crate into the VFS's dependency graph. If you would rather keep the VFS regex-free, the request carries a `&dyn Matcher` and the Pack supplies it, which keeps the dependency out but adds a lifetime to the API and forecloses a backend that wants to push the pattern into its own query language.
- The claim has to be defined carefully. "The search claims every path it visits" is not a claim you can take up front. See the claim discussion below.

**Option B: the Pack walks with `glob` and reads with `read_string`.**

What it provides:

- Zero new infrastructure. It works today.
- The policy, the event, and the claims all ride on operations that already exist and are already tested.
- It composes: `glob("**/*.rs")` then `read_string` each is what a model would do by hand, and it inherits the policy for free.

What it costs, measured on this machine on a 20,000 file tree of 3.3 KB files, 66.8 MB total:

| Approach | Wall clock | Peak RSS delta |
|---|---|---|
| ripgrep with the Grep argv | 554 ms | n/a (child process) |
| `rg --files` listing only | 53 ms | n/a |
| Walk with `readdirSync`, no reads | 94 ms | negligible |
| Walk + `stat` every file (the mtime sort pass) | 763 ms | negligible |
| Walk + read every file + regex match, retaining nothing | 1152 ms | 4.8 MB |
| Walk + read every file + regex, retaining all matches | 1315 ms | **154.6 MB** |
| Same walk, retaining at most 2000 entries | 1153 ms | 4.1 MB |

1. **The walk is not the expensive part.** It took 94 ms for 20,000 files, while the reads took 1152 ms, which is 20x the walk. The `glob`-then-read approach is slow because it reads every file; the walk itself is cheap.
2. **Retention is where it goes wrong.** Holding all 1.2 million matches cost 154.6 MB. Holding 2000 cost 4.1 MB, and the time was identical. This argues for a streaming API with a hard entry cap, and the argument applies equally to both options. A VFS `search` that returns `Vec<Match>` has the same problem as a Pack that collects them.
3. **The stat pass is 66% of the walk cost** (763 ms of the 1152 ms total is stat alone). The mtime sort is expensive. If you want newest-first ordering you incur the cost twice: once to `stat`, once to sort. Grep bounds it with a 2000 ms budget and gives up when the tree is big.
4. **ripgrep is 2x faster than the JS/Node walk, and it does the full content search as well.** Some of that is a compiled binary versus an interpreter, so a Rust Pack closing over `Access` would be closer. But not all of it: ripgrep parallelizes across cores and does not allocate a `String` per file.
5. **The 4.8 MB in the table comes from this probe's read design; the work itself does not guarantee it.** The probe read one file at a time and dropped it. Rig's `search` reads 100 whole files per `read_files` batch before it scans any of them, and on 100 files of 16 MiB its peak working set was 1,626 MiB (probe, rig `crates/rig-tools/src/search.rs:305-308` at commit `a83b774d0`). Point 2 is about retained matches; this point is about retained files, and a Pack that batches its reads needs a byte budget per batch, not a file count.

For a Rust implementation the estimate is that a Pack-level walk costs roughly 2x to 4x a native search on the same tree, because you give up parallelism and you materialize each file as an owned `String`. On a 200,000 file tree, scaling the 20,000 numbers linearly, that is 11 seconds versus 5, and 668 MB of reads.

**The claim question.**

A search touches every file. So what does it claim? Four candidates:

1. **Claim each visited path as a read.** Correct, precise, and unusable. 200,000 entries in the claim table, and the claim is taken incrementally as the walk proceeds, so a concurrent write can land halfway through the search and you get a torn view with no way to say so.
2. **Claim the subtree.** `Claims::claim_subtree` already exists and is what recursive delete uses. It conflicts with any write anywhere under the root. This is coarse but accurate, it is one table entry, and it is the same semantic you already ship for `remove(recursive: true)`. A search that conflicts with a concurrent write to the searched tree is arguably right.
3. **Claim nothing, and document that a search is a best-effort snapshot.** Cheapest, and a search is a read so this is not obviously wrong. The risk is that a caller reads a match, then acts on it, and the line is gone. But that risk exists in Cursor too, and Cursor takes no claim at all on the grep path. Grep does not touch the per-file lock manager.
4. **Claim the pattern.** This is what `claim_glob` does: "the pattern itself is the claim, not each match." A search that mirrors that would claim the (pattern, filters) pair and conflict with writes that would change its answer, which is not computable in general.

**Option C, from rig: a pure `find` over the existing traits, with a batch read hook.** Rig has neither a search method on its storage trait nor one read per file. `find(&dyn Workspace, &SearchArgs) -> Result<Found, ToolExecutionError>` is a plain function: it lists with `Workspace::files(root, glob, listing)`, reads with `Workspace::read_files(paths, limit)`, and returns data. `render(&Found)` is a separate pure function, and the tool publishes the typed `Found` and the `FilesRead` hashes beside the text (source, `search.rs:283-308`, `:236-241`). The `read_files` default does one read per path; a remote backend overrides it to read 100 files per script, so a search over a remote host costs two transport calls, which a test pins (source, `rt/remote/tests.rs:215-218`). The same function ran over `Local`, `Overlay` and `Hosts` with no search method in the trait. Zeroclaw's internal backend is the same idea with a deadline: a sorted walk, a deadline check per entry, `read_to_string` per file, and NUL files and non-UTF-8 files skipped (source, `content_search.rs:444-738`, commit `2d85371f3`).

What it costs, all read or measured: the walk collects the whole path list before anything is read, so a files-mode search that stops at 1000 names still walked the tree. The batch is 100 whole files, which peaked at 1,626 MiB (probe). `blocking` starts a thread that cannot be cancelled (source, `rt/blocking.rs:14-22`). Rig declares one footprint, `Reads(Paths([root]))`, for the whole call above the storage trait (source, `search.rs:206-211`), where Runvil's claims live inside `Access`. For Runvil, option C would be a function over `Access` that returns a struct, a separate renderer, and a batch read added to `Access` only when a backend needs it, with batches sized in bytes and the path list produced lazily.

My recommendation: **option A with candidate 2, claim the searched subtree, with a streaming result.** Confidence medium-high, down from high. It is one table entry, it reuses a claim kind you already have, it is the only shape that scales to 200,000 files, and the alternative is a claim table that grows with the tree. What moved it: rig runs option C over three backends with tests that pin its cost, so a walk with a batch hook is a working design and not a hypothetical, and the cases that still favor A are the claim and the memory profile, which are the two things rig leaves open. Take option C if the backend trait must not grow, and keep the subtree claim either way. If the granularity turns out to be too coarse in practice, the escape hatch is to add a `claim_search` that takes the root plus the filter and conflicts only with writes under paths the filter admits, which is a refinement you can land later without changing the call sites.

If you take option B, take it knowingly: cap `glob` results *before* reading, because `glob` returning 200,000 paths is itself a 200,000 element `Vec<String>`, and a 200,000 path list at roughly 80 bytes per path is 16 MB before you have read a single byte. Rig does not do this: `Workspace::files` returns the whole `Vec<PathBuf>` and `find` caps afterwards, so a search that stops at 1000 files still walked the whole tree (source, `workspace.rs:147`, `search.rs:288-304`). That shows the cost is accepted in practice. It does not show that it is free.

### D2. Do you copy the hyphenated parameter names?

`"-A"`, `"-B"`, `"-C"`, `"-i"` are literal JSON property names in Cursor's schema; they are not renamed to `context_before` or `case_insensitive`, and they are not normalized on the way in.

Options:

- **Copy the hyphens.** The model was trained on these. A model that has seen `Grep` will emit `"-A": 2` because that is what the schema it saw said. In a JSON Schema, `"-A"` is a legal property name. Cost: it looks bizarre in Rust, your `serde` derives need `#[serde(rename = "-A")]`, and any hand-written struct field called `a` or `minus_a` needs a doc comment explaining why. It also breaks the Rust naming convention lint if you are strict.
- **Normalize to snake_case and accept both.** `#[serde(alias = "-A")]` plus a field named `context_after`. Cost: the schema you publish to the model no longer matches the one the model was trained on, and `-i` in particular would become `case_insensitive`, which is a proto field name the model has almost certainly never seen as a JSON key.
- **Accept both, publish the hyphens.** The schema in `schemas/grep.json` keeps `"-A"`; the Rust struct has `context_after` with `#[serde(rename = "-A", alias = "context_after")]`. Best of both, at the cost of two names per field in the code.

My recommendation: **publish the hyphens, alias the snake_case forms.** Confidence high, because the whole premise of `plugin-std` is that models are trained on Cursor's surface, and the hyphens are part of that surface. Aliasing costs one attribute per field and gives you the ability to write ergonomic tests.

### D3. Do you keep the loose coercion, or go typed?

Today `head_limit: "50"` works, `"-A": "2"` works, `"-i": "yes"` works, `"-i": "no"` works, `"-i": 0` works. The machinery is three functions:

```js
const mIe=new Map([["true","true"],["yes","true"],["1","true"],[1,"true"],
                   ["false","false"],["no","false"],["0","false"],[0,"false"]]);
function hIe(e){if(null===e)return!1;return Che(mIe.get("string"==typeof e?e.trim().toLowerCase():e)??e)}
```

and a `stringified_number` pipe that turns `" 50 "` into `50` but leaves `"fifty"` alone for zod to reject. Then `fIe` clamps to `[0, 2147483647]`.

Options:

- **Reproduce it.** Cost: a `Value`-typed deserializer in front of your typed struct, plus a table of spellings. Provides: a model that says `"-i": "true"` gets a search instead of an argument error, which is one fewer wasted turn.
- **Typed only.** `bool` and `u32` via serde. Cost: every coercion case becomes an `Invalid arguments:` error. Provides: a much simpler schema layer and errors that name the problem instead of silently coercing.
- **Typed, with a `serde` deserializer that accepts the string spellings.** A `deserialize_with` for the bool and the numbers, no `Value` plumbing. Cost: about 30 lines. Provides: everything the loose path provides, with a typed struct at the end.

My recommendation: **typed with a lenient `deserialize_with`.** Confidence high, because you get Cursor's tolerance without the plumbing for a raw `Value`, and the tolerance matters: the coercion table exists because models do emit these strings.

**Other harnesses.** Pi validates arguments with typebox `Value.Convert` before the tool runs, so `"5"` becomes `5` and `"true"` becomes `true`, and `null` on an optional field is dropped (source, pi `validation.ts:317-351`). Rig goes the other way: `deny_unknown_fields` makes a misspelled key an error, and `-1`, `1.5` and `"10"` fail to decode for its `usize` fields (source, `search.rs:57-58`). Copilot refuses a stray `pattern` with a message that names the right key, `query` (source). Typed with a lenient deserializer sits between pi and rig, which is where the recommendation already is.

One asymmetry to decide deliberately: `"-i": null` returns `false` in Cursor (`if(null===e)return!1`), while `"-i": "maybe"` fails the lookup and falls through to `Che("maybe")`, which returns `"maybe"`, which zod's boolean rejects with an `Invalid arguments` error. So null is tolerated and garbage is not.

### D4. Do you clamp negative context or reject it?

Cursor does both, in a specific order that makes only one of them visible. The zod layer's `fIe` clamps to `>= 0`. But the execute body checks the *raw* value first:

```js
for(const e of["-A","-B","-C"]){const t=r[e];if(void 0!==t&&t<0)throw dIe(`${e} is ${t}. Pass 0 or more context lines.`)}
```

So `-A: -1` produces `-A is -1. Pass 0 or more context lines.` and never reaches the clamp.

Options:

- **Reject, like Cursor.** Cost: one extra error branch. Provides: the model learns the constraint instead of getting a silently different search.
- **Clamp silently.** Cost: a `-A: -5` becomes `-A: 0`, and the model thinks it has context when it does not.
- **Reject at schema validation.** Cost: the error text becomes `Invalid arguments:\n-A: Too small: expected number to be >=0`, which is a worse message.

My recommendation: **reject with Cursor's exact string**, before clamping, at the same point in the order. Confidence high, because a silently different result is the worst of the three.

### D5. Do you fix the `-C` drop, or reproduce it?

`-C` alone works. `-C` with `-A` or `-B` is dropped entirely, with no warning:

```js
void 0!==t.context&&void 0===t.contextBefore&&void 0===t.contextAfter
  &&s.push("--before-context",String(t.context),"--after-context",String(t.context));
```

So `-C: 3, -A: 1` gives one line of after-context and zero before-context. The model asked for symmetric context and got asymmetric, silently.

Options:

- **Reproduce the drop.** Cost: a model gets a subtly wrong result and no way to know. Provides: byte-identical behavior, which matters if you ever diff your output against Cursor's.
- **Make `-C` override `-A`/`-B`.** Cost: a divergence. Provides: the parameter name `-C` means what the description says.
- **Make `-A`/`-B` override `-C` per side.** `-C: 3, -A: 1` gives before 3, after 1. Cost: a different divergence. Provides: no parameter is ever silently ignored, which is the principled version.
- **Reject the combination.** Cost: an error the model did not expect. Provides: nothing is silently wrong.

My recommendation: **`-A`/`-B` override `-C` per side, and never drop a supplied parameter silently.** Confidence medium, because it is a divergence, but the drop is a bug that costs the model a turn every time it happens, and a per-side override is what every other tool in the space does. If you would rather be bug-compatible, reproduce the drop and add a line to the description saying `-C` is ignored when `-A` or `-B` is present, so the model can at least know.

### D6. Do you keep the broad-glob guard?

There is a hard refusal behind a feature flag:

```js
uIe=new Set(["**/*","**/**"])
...
if(s&&(void 0!==(l=r.glob)&&uIe.has(l.trim())))
  throw function(e){return dIe(`Glob pattern "${e}" matches every file and is not allowed. Use a more specific glob or no glob.`)}(r.glob.trim())
```

`s` is `true` only when `featureFlags.enableGrepBroadGlobGuard` is set. The set is exactly two strings, tested after `.trim()`, so `" **/* "` is refused and `"**/**/*"` is not.

Why it exists: `--iglob` overrides `.gitignore`. A `glob` of `**/*` makes ripgrep search `node_modules`, `target`, `.venv`, and everything else, which is both slow and a waste of the model's context. I confirmed this on a fixture: with no glob, ripgrep returned 3 of 5 files (gitignore respected); with `--iglob '**/*'` it returned all 5.

Options:

- **Keep the guard, always on.** Cost: a legitimate `glob: "**/*"` that the model meant literally is refused. Provides: the model cannot accidentally search an ignored tree, which is the most common way to get a useless 2000-line result.
- **Keep it behind a flag, defaulting on.** Cost: a flag to document. Provides: an escape hatch.
- **Drop it, and make the description's advice a warning only.** Cost: the failure mode the guard prevents comes back. Provides: no surprising refusals.

My recommendation: **keep the guard, on by default, and also widen the set to include `*` and `**`.** Confidence medium-high. The reasoning: the shipped description already tells the model to avoid broad globs "as they bypass .gitignore rules and may be slow", which means Cursor knows the hazard and has decided to advise rather than refuse. But advising did not work, which is why the hard guard was added, and it is behind a flag only because it was rolled out gradually. Widening the set to `*` and `**` closes the `--iglob *` case, which I confirmed has the same ignore-bypassing effect as `**/*`.

**Other harnesses.** Rig's glob grammar is the `ignore` crate's override layer, and it confirms the hazard from another engine: in a git work tree `*`, `**`, `**/*` and `*.txt` return ignored files and ignored directories, while `build`, `build/`, `build/*` and `build/**` do not (probe, rig `OverrideBuilder`, `ignore` 0.4.25). That supports widening the set to `*` and `**`, and it shows that no fixed list closes the gap, since `*.txt` also returned `ignored.txt` and `skip.txt`. Its tool description says "Skips files ignored by git" and is wrong on those inputs, which is the D7 problem again. (probe)

If you keep the guard, fix the description contradiction in D7.

### D7. Do you say what the tool actually does in the description?

The `latest` description says:

```
- Avoid overly broad glob patterns (e.g., '--glob *') as they bypass .gitignore rules and may be slow
```

That is advice, phrased as a speed warning. The tool *refuses* two specific globs with an error, when the flag is on. So the description and the behavior disagree, and the description's example `--glob *` is not even one of the refused values.

Three more description defects, each needing a decision:

- `Filter files with glob parameter (e.g., ".js", "**/.tsx")` is wrong twice. `.js` is missing its leading `*`, and `**/.tsx` should be `**/*.tsx`. A model that copies either example gets zero results.
- The description says results are "capped to several thousand output lines". The limits are 2000 entries parsed, 15000 characters rendered, and no default `head_limit`. "Several thousand" is defensible for the parse cap and wrong for the render budget.
- The `gpt5-codex` text contains a literal U+2014 em dash character inside the phrase `lines` + U+2014 + `it`. That character is in the shipped string. It is reproduced correctly in the fenced block above; do not normalize it when reimplementing.

Options: copy the text verbatim including the typos, or fix them.

My recommendation: **fix the two glob examples and the broad-glob sentence, keep everything else verbatim.** Confidence high on the examples, because a wrong example in a tool description is a guaranteed wasted call. Confidence medium on the broad-glob sentence, because if you drop the guard you should keep the advice and if you keep the guard you should say it is refused.

### D8. Do you keep the mtime sort?

Files come back newest-first by `mtimeNs`, with ties broken by path segments ascending and missing mtimes last, under a 2000 ms budget after which the order is ripgrep's walk order. The stat pass costs 763 ms on 20,000 files, which is most of the walk time.

Options:

- **Keep newest-first with the budget.** Cost: 66% of your walk time, and nondeterminism when the budget expires. Provides: the tool matches the description, and recent files are usually the relevant ones.
- **Drop the sort, return walk order.** Cost: the description's implicit promise, and results feel arbitrary. Provides: a saving of two-thirds of the wall clock.
- **Sort by path.** Cost: no recency signal. Provides: full determinism, low overhead, and stable results across runs, which matters if you want byte-exact tests.
- **Sort by path within a file, mtime across files, but only when the result is small enough to stat cheaply.** A hybrid: if the match count is under, say, 500, stat and sort; otherwise return path order and say so.

My recommendation: **sort by path.** Confidence medium-high, up from medium. The newest-first order is a Cursor inheritance from Claude Code, not a considered choice, and it brings a stat pass and a nondeterminism you have to document. Path order is deterministic, free, and easy to test. If you want recency, add a `sort` parameter and let the model ask. What moved it: four other harnesses ship no sort and their results differ from run to run (pi 8 outputs in 8 runs, zeroclaw 12 in 12, OpenCode 4 in 4, Cline 6 in 6), while Zed and zeroclaw's internal backend order by path and are stable (probe, source).

**Other harnesses.** Codex's removed `grep_files` asked ripgrep for `--sortr=modified`, the same newest-first idea without a client-side stat pass, and listed only paths (source, `grep_files.rs`, parent of PR 15775). Zeroclaw measured the fix inside ripgrep: with the tool's flags 12 runs gave 12 orders, and `--sort path` gave one (probe). Rig's own test sorts before it asserts because its walk is unordered (source, `search/tests.rs:140-141`).

Whatever you choose, **decide what happens on a stat timeout**. Cursor silently falls back to walk order and increments a metric. A silent order change is worse than a slow search, so if you keep mtime sorting, sort by path on timeout instead of leaving walk order.

### D9. Binary files: skip, report, or search?

Today: content mode drops them silently (the parser cannot parse the line), files and count mode include them. Nothing tells the model a binary file was skipped.

Options:

- **Reproduce it.** Cost: the model cannot tell whether a file was searched. Provides: identical output.
- **Skip them and count them.** Cost: one trailer field. Provides: the model knows 3 binary files were skipped, which is often the answer to "why didn't my search find it".
- **Search them anyway** (`-a` / `--text`). Cost: a binary file can produce a 2400-byte line of noise, and the `--max-columns` cut makes it a useless preview. Provides: little.

My recommendation: **skip, and report the count in the trailer.** Confidence high, because silent omission is the failure mode that wastes the most turns: the model concludes the symbol does not exist.

**Other harnesses.** None of the harnesses whose notes cover a directory search tells the model that a binary file was skipped. Zed skips on a 1,024-byte classifier with a debug log. Pi, OpenCode and Cline lose the file silently because ripgrep reports nothing for a NUL file in a directory search. Rig has no check: it searches any valid UTF-8, NULs included, and skips a non-UTF-8 file whole (source, probe). Zeroclaw's `rg` backend prints `binary file matches` for a file named on the command line, and its formatter counts that line as a match. Gemini's directory behavior was not probed; on an explicit file ripgrep returned the matches with `binary_offset` in the end event. The recommendation stands, and it is the one place every harness agrees on the failure and none has fixed it. (source, probe)

### D10. Do you keep the 5-entry result cache?

The cache is keyed on every field except `toolCallId`, `sandboxPolicy`, `headLimit`, and `offset`:

```js
function KEe(e,t){return e.pattern===t.pattern&&e.path===t.path&&e.glob===t.glob
  &&(e.outputMode??"content")===(t.outputMode??"content")&&e.type===t.type
  &&e.caseInsensitive===t.caseInsensitive&&e.multiline===t.multiline
  &&e.contextBefore===t.contextBefore&&e.contextAfter===t.contextAfter&&e.context===t.context
  &&e.sort===t.sort&&e.sortAscending===t.sortAscending}
```

Options:

- **Keep it.** Cost: a second call with a different `head_limit` sees the first call's snapshot. Provides: a model that pages with `offset` does not re-run the search, which is why pagination and the cache interact the way they do.
- **Drop it.** Cost: every page re-runs ripgrep. Provides: every call sees current disk state, which is what a read tool should do.
- **Keep it with a short TTL**, so a page within a second reuses the snapshot and a later call does not.

My recommendation: **keep it but scope it to the pagination chain.** Confidence medium-high. The cache exists so `offset` paging is coherent: page 2 must see the same snapshot as page 1 or the pages do not line up. But an unbounded-lifetime 5-entry cache keyed on the search means a search after an edit can return stale results, and the model has no way to know. A generation counter that the Pack bumps on any write would give you both.

**Other harnesses.** Of the harnesses whose notes cover this, none keeps a search cache beyond Cursor's. Zed re-runs the whole search on every page, which is coherent only because its order is deterministic; pi re-runs it for `limit=2N` and its order is not, so the larger result need not extend the smaller one (probe). Rig re-walks on every call. A cache is what makes Cursor's `offset` coherent under an unstable order, and it is also what makes a search after an edit stale, so the generation counter is the part nobody else has needed to build. (source, probe)

### D11. Which pattern dialect does the model see, and do you add a literal flag?

Cursor's pattern is whatever ripgrep's Rust `regex` accepts, with no literal mode, and the dialect decides what a model can write. The other harnesses show three answers.

- **The Rust `regex` crate, linear time.** Rig runs it in process (`regex` 1.12.3). It refuses look-around and back-references with a message that names the feature, `error: look-around, including look-ahead and look-behind, is not supported`. It also refuses a pattern whose compiled program passes 10,485,760 bytes, with `Compiled regex exceeds size limit of 10485760 bytes.` On `^(a|aa)+$` it took 246 us, 966 us and 3.85 ms for 100,000, 400,000 and 1,600,000 characters, so growth is linear (probe).
- **`fancy_regex`, which backtracks.** Zed runs it, so look-around, back-references and `\c` and `\C` case overrides work, a backtracking limit of 1,000,000 bounds each pattern, and a file that hits the limit is dropped silently. Its parameter text tells the model the pattern is parsed by "the Rust `regex` crate", which is not the engine (source, `grep_tool.rs:38`, `search.rs:4`).
- **Validate in one dialect, run in another.** Gemini checks the pattern with JavaScript `new RegExp` and runs it with ripgrep: `(?i)foo` is rejected before ripgrep sees it, and `(?=x)y` passes validation, fails inside ripgrep with exit 2, and comes back as a string that looks like a success (probe, source, `ripGrep.ts:662-668`, `:348-355`).

A literal mode is a separate question. Pi has `literal`, Gemini has `fixed_strings`, Copilot has `isRegexp: false` and re-runs a search that found nothing in the other mode, and rig has none, so `foo(` is a pattern error there (source, probe). Cursor has none either, and a model that forgets to escape `a.b` gets a wrong answer without an error.

My recommendation: **the Rust `regex` dialect, a `literal` boolean, and a parse error that names the unsupported feature.** Confidence high, because it matches what ripgrep already does for the Cursor surface, it is linear time so no pattern can hang the tool, and the one in-process Rust harness that ships it also ships the error text that makes the dialect learnable. State the dialect in the description ("Rust regex syntax", as rig's does) and do not name a crate you do not run. Keep Cursor's case-sensitive default: Zed, Gemini, Cline and Copilot default to insensitive, so a model trained on them may omit `-i` and expect it.

## The contract

### Names on every branch

`AG("GREP", ...)` takes no `toolIdentifier` override, so the internal id is the literal `GREP` on every branch. The public name switches on the model family (offset 4315866):

| Version tag | Public name |
|---|---|
| `dsv3-1018` | `grep` |
| `cursor-0226` | `Grep` |
| `dsv3-1205` | `Grep` |
| `latest` | `Grep` |
| `haiku` | `Grep` |
| `gpt5-codex` | `rg` |
| `codex-cloud` | `rg` |

Default is `latest`. An unknown tag throws `Unhandled version: ${e}` rather than falling through.

`grep_search` is **never** a Grep tool name. The four occurrences in the bundle are protobuf field descriptors, `RipgrepRawSearch` and `ripgrep_raw_search_result`. Do not use it.

### The schema, verbatim

Two schemas, `wIe` at 4311435 and `vIe` at 4313434. Selected by `u="dsv3-1018"===t?vIe:wIe`, so `dsv3-1018` gets `vIe` and everything else gets `wIe`. The field sets are **identical**; exactly three description strings differ. My field-by-field diff across both objects found 12 fields in each, in the same order, with only indexes 0, 1, and 2 differing.

Below is `wIe`, the default. Descriptions are given as the model receives them, after unescaping.

```js
U$.Ikc({
  pattern: U$.YjP()
    .describe("The regular expression pattern to search for in file contents"),
  path: U$.YjP().optional()
    .describe("File or directory to search in (rg pattern -- PATH). Defaults to Cursor workspace root."),
  glob: U$.YjP().optional()
    .describe('Glob pattern to filter files (e.g. "*.js", "*.{ts,tsx}") - maps to rg --glob'),
  output_mode: U$.k5n(["content","files_with_matches","count"])
    .describe('Output mode: "content" shows matching lines (supports -A/-B/-C context, -n line numbers, head_limit), "files_with_matches" shows file paths (supports head_limit), "count" shows match counts (supports head_limit). Defaults to "content".')
    .default("content").optional(),
  "-B": yIe(U$.aig()).optional()
    .describe('Number of lines to show before each match (rg -B). Requires output_mode: "content", ignored otherwise.'),
  "-A": yIe(U$.aig()).optional()
    .describe('Number of lines to show after each match (rg -A). Requires output_mode: "content", ignored otherwise.'),
  "-C": yIe(U$.aig()).optional()
    .describe('Number of lines to show before and after each match (rg -C). Requires output_mode: "content", ignored otherwise.'),
  "-i": U$.vkY(hIe, U$.zMY().optional()
    .describe("Case insensitive search (rg -i) Defaults to false").default(!1)),
  type: U$.YjP().optional()
    .describe("File type to search (rg --type). Common types: js, py, rust, go, java, etc. More efficient than glob for standard file types."),
  head_limit: yIe(U$.aig().min(0)).optional()
    .describe('Limit output size. For "content" mode: limits total matches shown. For "files_with_matches" and "count" modes: limits number of files.'),
  offset: yIe(U$.aig().min(0)).optional()
    .describe('Skip first N entries. For "content" mode: skips first N matches. For "files_with_matches" and "count" modes: skips first N files. Use with head_limit for pagination.'),
  multiline: U$.vkY(hIe, U$.zMY().optional()
    .describe("Enable multiline mode where . matches newlines and patterns can span lines (rg -U --multiline-dotall). Default: false.").default(!1))
})
```

`U$.Ikc` is the strict object builder, so unknown keys are rejected before the executor sees them. There is no `.passthrough()` and no `.loose()`.

Flattened:

| Field | Type | Required | Default | Notes |
|---|---|---|---|---|
| `pattern` | string | **yes** | none | The only required field |
| `path` | string | no | workspace root | Relative, absolute, `~`, or `file://` |
| `glob` | string | no | none | Becomes `--iglob` |
| `output_mode` | enum | no | `"content"` | `content`, `files_with_matches`, `count` |
| `-B` | number | no | none | Before-context lines |
| `-A` | number | no | none | After-context lines |
| `-C` | number | no | none | Symmetric context, dropped if `-A` or `-B` present |
| `-i` | boolean | no | `false` | Becomes `--ignore-case` |
| `type` | string | no | none | Becomes `--type` |
| `head_limit` | number, min 0 | no | **none** | Global across files; `0` means unlimited |
| `offset` | number, min 0 | no | `0` | Global across files |
| `multiline` | boolean | no | `false` | `-U --multiline-dotall` |

**Fields that do not exist.** No `target_directory` (that is Glob and ListDir, and the field here is `path`), no `-n` (line numbers are unconditional in content mode), no `case_insensitive` (the key is `-i`; `case_insensitive` is only the proto field name in `GrepArgs`), no `--no-ignore` and no `no_ignore` parameter, no `-o` / `only_matching`, no `sort` or `sort_ascending` (both exist in the proto message but no schema field reaches them), no `max_results`.

### The three description differences on `dsv3-1018`

| Field | `wIe` | `vIe` |
|---|---|---|
| `pattern` | `The regular expression pattern to search for in file contents` | `The regular expression pattern to search for in file contents (rg --regexp)` |
| `path` | `... Defaults to Cursor workspace root.` | `... Defaults to Cursor workspace roots.` |
| `glob` | `Glob pattern to filter files (e.g. "*.js", "*.{ts,tsx}") - maps to rg --glob` | `Glob pattern (rg --glob GLOB -- PATH) to filter files (e.g. "*.js", "*.{ts,tsx}").` |

The plural `roots` reflects that the `dsv3-1018` era searched every workspace root at once. The executor still iterates all roots on both branches.

### The tool description, canonical branch

`dsv3-1205`, `latest`, and `haiku` share one string, at 4315866. Reproduced byte for byte, newlines included:

```text
A powerful search tool built on ripgrep
Usage:
- Prefer using Grep for search tasks when you know the exact symbols or strings to search for. Whenever possible, use this tool instead of invoking grep or rg as a terminal command. The Grep tool has been optimized for speed and file restrictions inside Cursor.
- Supports full regex syntax (e.g., "log.*Error", "function\s+\w+")
- Filter files with glob parameter (e.g., ".js", "**/.tsx") or type parameter (e.g., "js", "py", "rust")
- Output modes: "content" shows matching lines (default), "files_with_matches" shows only file paths, "count" shows match counts
- Pattern syntax: Uses ripgrep (not grep) - literal braces need escaping (use interface\{\} to find interface{} in Go code)
- Multiline matching: By default patterns match within single lines only. For cross-line patterns like struct \{[\s\S]*?field, use multiline: true
- Results are capped to several thousand output lines for responsiveness; when truncation occurs, the results report "at least" counts, but are otherwise accurate.
- Content output formatting closely follows ripgrep output format: '-' for context lines, ':' for match lines, and all context/match lines below each file group.
```

Note there is **no blank line** after `A powerful search tool built on ripgrep` in this branch. The `dsv3-1018` branch has one.

That text has three defects:

1. `(e.g., ".js", "**/.tsx")` should be `"*.js"` and `"**/*.tsx"`. Both examples as written match nothing useful: `.js` is a literal filename, and `**/.tsx` is a file literally named `.tsx` in any directory.
2. The broad-glob bullet is in the `dsv3-1018` text only, and it references `'--glob *'`, a CLI flag the tool does not expose. Its meaning is "a glob of `*`", and the tool's hard refusal covers `**/*` and `**/**` but not `*`.
3. `several thousand output lines` understates the parse cap (2000 entries) and overstates the render budget (15000 characters, typically fewer than 300 lines). The phrase "report 'at least' counts" is accurate.

### The other description branches

**`cursor-0226`**, one sentence:

```text
A search tool built on ripgrep. Results are capped to several thousand output lines for responsiveness; when truncation occurs, the results report "at least" counts, but are otherwise accurate.
```

**`gpt5-codex` and `codex-cloud`**, seven bullets:

```text
Search the workspace with ripgrep.

- Use this tool instead of shell rg; respects .gitignore and .cursorignore
- Default scope is the workspace root; set path (absolute path) to narrow it
- Supply a regex pattern; escape metacharacters, e.g. "functionCall\(", "\{", "\}"
- Prefer type over broad glob; wildcard globs like * bypass ignore rules and slow searches
- Enable multiline only when a match spans lines—it can degrade performance
- Context flags (-A, -B, -C) only affect content output
- If results show "at least …", the output was truncated; tighten the query or raise head_limit
```

The em dash between `lines` and `it` is U+2014, in the shipped string. The ellipsis in `"at least …"` is U+2026. Both matter for byte-exact reproduction.

**`dsv3-1018`**, the longest, ten bullets, with the blank line after the first line and the `(unsaved)` claim:

```text
A powerful search tool built on ripgrep

Usage:
- Prefer grep for exact symbol/string searches. Whenever possible, use this instead of terminal grep/rg. This tool is faster and respects .gitignore/.cursorignore.
- Supports full regex syntax, e.g. "log.*Error", "function\s+\w+". Ensure you escape special chars to get exact matches, e.g. "functionCall\("
- Avoid overly broad glob patterns (e.g., '--glob *') as they bypass .gitignore rules and may be slow
- Only use 'type' (or 'glob' for file types) when certain of the file type needed. Note: import paths may not match source file types (.js vs .ts)
- Output modes: "content" shows matching lines (default), "files_with_matches" shows only file paths, "count" shows match counts per file
- Pattern syntax: Uses ripgrep (not grep) - literal braces need escaping (e.g. use interface\{\} to find interface{} in Go code)
- Multiline matching: By default patterns match within single lines only. For cross-line patterns like struct \{[\s\S]*?field, use multiline: true
- Results are capped for responsiveness; truncated results show "at least" counts.
- Content output follows ripgrep format: '-' for context lines, ':' for match lines, and all lines grouped by file.
- Unsaved or out of workspace active editors are also searched and show "(unsaved)" or "(out of workspace)". Use absolute paths to read/edit these files.
```

The `(unsaved)` bullet is **aspirational**. I found no code path in the Grep executor that searches an editor buffer. The `activeEditorResult` field exists in the result message and the paginator rotates it (`activeEditorResult:n.activeEditorResult?s(n.activeEditorResult):void 0`), so some producer populates it, but that producer is not in this bundle. Treat the claim as unverified.

### Caps and limits

| Limit | Value | Unit | Layer that enforces it |
|---|---|---|---|
| content entries kept | 2000 | entries (matches plus context lines) | parser `Ax`, `linesProcessed >= clientLimitLines` |
| content ripgrep line budget | `2000 * (context ? 2 : 1)` = 2000 or 4000 | stdout lines | `runContentMode`, passed to `executeRg` as `lineBudget` |
| files kept | 2000 | file paths | `runFilesMode`, `totalFiles > CLIENT_LIMIT_LINES`; `keepNewestPage` slices to 2000 |
| count rows kept | 2000 | file-count rows | `runCountMode`, same comparison |
| files and count parse window | 10000 | stdout lines | `Cx` / `Px` third argument, `HARD_MAX_OUTPUT_LINES` |
| ripgrep stdout | 8388608 | **bytes** | `executeRg`, `y += e.length` on the Buffer, child killed on exceed |
| ripgrep wall clock | 25000 | ms, per root | `MAIN_TIMEOUT_MS`, `AbortController` |
| indexed-search wall clock | 25000 (+100 outer) | ms | `INDEXED_GREP_TIMEOUT_MS`, then silent fallback to ripgrep |
| mtime stat batch | 2000 | ms | `sortNewestFirst`, `withTimeout(..., 2e3, "stat timed out")` |
| per-file encoding re-decode | 2000 | ms total | `LINE_ENCODING_TIMEOUT_MS` |
| rendered output per workspace | 15000 | **UTF-16 code units** (JS string length) | `GREP_CHAR_BUDGET`, `renderMultiWorkspace` |
| rendered output, multi-root | ~15000 total | same unit | binary-searched per-workspace budget |
| error text | 15000 | characters, head-and-tail split | `EG(error, 15e3, true)` |
| long line in content output | 1000 | **bytes**, decided by ripgrep | `--max-columns 1000` |
| long line preview length | 1000 | **characters** | ripgrep's `--max-columns-preview` |
| `head_limit` ceiling | 2147483647 | entries | `fIe` clamp |
| `offset` ceiling | 2147483647 | entries | `fIe` clamp |
| result cache | 5 | search results | `VEe` constructor default |

The three unit families in that table come from three subsystems: ripgrep decides the long-line cut in bytes, ripgrep's preview counts characters, and the JS renderer counts UTF-16 code units. A line of 400 emoji is 1600 bytes, so ripgrep cuts it; the preview keeps all 400 because 400 < 1000 characters; and the rendered line then costs 800 UTF-16 units against the 15000 budget.

### Result cases

The Grep result oneof has three success shapes and one error:

```
success -> { pattern, path, outputMode, workspaceResults: map<root, GrepUnionResult>, activeEditorResult? }
           GrepUnionResult.case in { content, files, count }
error   -> { error: string }
```

Both success shapes are wrapped in `<workspace_result workspace_path="...">`. The observable model-visible text for each case follows.

### Content mode output

Structure:

```
<workspace_result workspace_path="/abs/root">
<path line, bare, no separator>
  <lineNumber>:<content>          for a match line
  <lineNumber>-<content>          for a context line
  --                              gap separator, two spaces then two hyphens
<blank line>
<next path line>
...
</workspace_result>
```

Byte-level facts:

- Match line prefix is two ASCII spaces, then the line number in decimal with **no padding**, then `:`.
- Context line prefix is two spaces, the number, then `-`.
- The gap separator is the literal `  --`. It is emitted only when the result contains at least one context line anywhere (`hasContext = matches.some(fm => fm.matches.some(m => m.isContextLine))`), and only when the current line number is more than 1 greater than the previous one in the same file.
- Between files there is one blank line, from `rawLines.push("")`.
- The whole assembled body is trimmed. In the single-workspace path the template is:

```js
`<workspace_result workspace_path="${n}">\n${body}\n</workspace_result>\n`
```

and then the whole thing is `.trim()`ed. So a body ending in a blank line loses it, and the closing tag sits directly under the last match line. There is no trailing newline at the end of the model-visible text.

Worked example, produced by a faithful port of the renderer rather than by hand. Two files, nine-line and one-line, matches on 3 and 9 and 1, `-A: 1`:

```json
{"pattern": "const", "-A": 1}
```

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

The blank line before `src/b.ts` is the per-file trailing blank. There is no blank line before `</workspace_result>`. Both facts came out of the port running the algorithm, and both contradict the parent spec's example, which shows a blank line before the closing tag.

Without any context flag the gap separator disappears entirely, even when the line numbers are far apart:

```text
src/a.ts
  3:const alpha = 1
  9:const gamma = 3
```

That is `hasContext` false, so the `  --` branch never runs. A model reading this sees two matches with no indication that 5 lines were skipped, which is correct ripgrep behavior but is not what a `--` separator would suggest.

### files_with_matches output

One bare path per line. `formatFilesResult` passes `fo.files` straight to `truncateLinesToCharBudget` without interleaving blank lines, so there is no blank line between files:

```text
<workspace_result workspace_path="/home/dev/proj">
src/a.ts
src/b.ts
</workspace_result>
```

Paths are workspace-relative. On Windows they carry a leading `.\`.

### count output

`path:count` per line, no spaces around the colon:

```text
<workspace_result workspace_path="/home/dev/proj">
src/a.ts:3
src/b.ts:1
</workspace_result>
```

The count parser splits on the **last** colon of the line, which is what makes `C:\src\a.ts:3` parse correctly on Windows.

### The truncation trailer

Appended to the body, with a leading newline and no trailing newline of its own:

```
\n[<Results truncated. >Showing <shown> of <total> <unit>. <next step>]
```

Component rules, from `describePosition`, `describeShown`, `describeTotal`, and `describeNextStep`:

| Part | Rule |
|---|---|
| `Results truncated. ` | present iff `charTruncated OR cutBeforeEnd`, where `cutBeforeEnd = ripgrepTruncated OR (reachedEnd AND (clientTruncated OR end < total))` and `reachedEnd = headLimitApplied === undefined OR returned < headLimitApplied` |
| `shown` | `none` if 0; `${shown}` if offset is 0; `${offset+1}-${offset+shown}` otherwise |
| `total` | `max(total, offset + returned)` |
| `at least ` | present iff `ripgrepTruncated OR (clientTruncated AND (!reachedEnd OR total <= end))` |
| unit | `matching line` / `matching lines` for content; `file` / `files` for files and count |
| pluralization | `s` iff the displayed number is not exactly 1 |
| next step, none | absent, or `Narrow the search to see the rest.` when something was cut |
| next step, single workspace | `Use offset=${offset+shown} to see more, or narrow the search.` |
| next step, cut and reached end | `Use offset=${offset+shown} to see more of the first ${end}, or narrow the search to see the rest.` |
| next step, multi workspace | `Set path to one workspace to page through it, or narrow the search.` |

I ported the footer builder line for line and ran it against ten cases:

| Case | Rendered footer |
|---|---|
| `head_limit: 2`, 5 total, offset 0 | `[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]` |
| `head_limit: 2`, `offset: 2`, 5 total | `[Showing 3-4 of at least 5 matching lines. Use offset=4 to see more, or narrow the search.]` |
| `head_limit: 2`, `offset: 2`, 4 total | `[Showing 3-4 of at least 4 matching lines. Use offset=4 to see more, or narrow the search.]` |
| `head_limit: 2`, `offset: 3`, 5 total | `[Showing 4-5 of at least 5 matching lines. Use offset=5 to see more, or narrow the search.]` |
| 2000-entry parse cap hit, 4211 total | `[Results truncated. Showing 2000 of 4211 matching lines. Narrow the search to see the rest.]` |
| 15000-character cut, no `head_limit` | `[Results truncated. Showing 120 of 300 matching lines. Use offset=120 to see more, or narrow the search.]` |
| `head_limit: 3`, 5 files | `[Showing 3 of at least 5 files. Use offset=3 to see more, or narrow the search.]` |
| `head_limit: 3`, 5 files, multi-root | `[Showing 3 of at least 5 files. Set path to one workspace to page through it, or narrow the search.]` |
| everything fits | (no footer) |
| single match, nothing cut | (no footer) |

Two things this settles that the parent spec gets wrong or leaves open:

1. **`at least` is present for a plain `head_limit` cut.** The spec's `Global head_limit across two files` example shows `[Showing 2 of 5 matching lines. ...]` with no `at least`, and marks the line `(derived)`. The derivation was wrong. `clientTruncated` is true after `sIe` sets its flag, and `reachedEnd` is `returned < headLimitApplied`, which is `2 < 2`, false, so `isLowerBound` is `clientTruncated && (!reachedEnd || ...)`, true. The rendered text has `at least`.
2. **A full page with nothing left gets no footer at all.** My first reading of the code was that a `head_limit: 1` result with one match would still hedge with `at least 1`. It does not. `describeNextStep` computes `fullPageMayHaveMore = !reachedEnd && (end < total || clientTruncated || ripgrepTruncated)`, and with `returned === headLimitApplied` the first conjunct is false and `clientTruncated` is false (no further match was found), so `canContinue` is false and the function returns `undefined` unless something was cut. The result is no footer. The same holds for `head_limit: 2` with two matches. I verified both cases by running the port, and the ported output was the empty string for each.

The case I cannot fully pin from the bundle: whether `Results truncated. ` appears for a plain `head_limit` cut. By the code it does not, because `cutBeforeEnd = ripgrepTruncated || (reachedEnd && (clientTruncated || end < total))` and `reachedEnd` is false while the page is full. So `head_limit: 2` of 5 renders `[Showing 2 of at least 5 matching lines. ...]` with **no** `Results truncated. ` prefix, and the only signal that something was cut is the word `at least`. That is a much weaker warning than the parent spec implies, and it should be confirmed live before the test is frozen.

### Empty-result strings

From `XEe`, `describeEmptyPage`:

| Situation | Exact text |
|---|---|
| content or count, offset 0, nothing truncated | `No matches found` |
| files, offset 0, nothing truncated | `No files with matches found` |
| any mode, offset 0, ripgrep truncated | `<noun> before the search stopped. Narrow the search to see the rest.` |
| any mode, offset > 0, more available | `No more results at offset=<offset>. Narrow the search to see the rest.` |
| any mode, offset > 0, exhausted | `No more results at offset=<offset>.` |

The third row uses the noun from the first, so files mode gives `No files with matches found before the search stopped. Narrow the search to see the rest.`

The offset in that string is `offsetApplied`, which is only set when `offset > 0`. The footer can therefore never print `offset=0`.

### Error and empty envelopes

```
error case              -> "Error running tool: " + headTailTruncate(error, 15000)
non-success, non-error  -> "No results found."
success, no workspaces  -> "No results found."
```

`EG(text, 15000, true)` keeps 7500 characters from the head and 7500 from the tail, joined by `\n\n... (output truncated) ...\n\n`.

### The pi surface

`pi_grep`, Cursor's port of pi's `grep`, is covered in one place, under the heading "pi, and Cursor's `pi_grep` port" in the harness section: the comparison of pi's original with the port, the schema, the argv, the output, the trailer strings and the recommendation. Two facts belong to the contract. The tool is registered at 4561631 under the internal id `PI_GREP`, always named `pi_grep`, with schema `ZRe` at 4554895, and it is offered only when `featureFlags.enablePiTools === true`, which registers all seven pi tools in one call. `(bundle)`

### A second schema projection

There is a static tool array at 4671940 that feeds the Smart Mode classifier rather than the main agent. Its `Grep` entry uses the `gpt5-codex` description, the `wIe` field descriptions, and a **different parameter set**: it omits `offset` entirely. So a classifier model sees a Grep with a `head_limit` and no `offset` to pair with it. That is a Cursor inconsistency, not behavior to reproduce, but it explains why a leaked prompt might show a schema without `offset`.

## Algorithm and ordering

The tool body and the executor are two separate stages, and the order inside each matters.

### The tool body

```
GREP_EXECUTE(args)

 1  PARSE        validate `args` against `wIe` or `vIe`
                 on failure THROW INVALID_ARGS "Invalid arguments:\n<key>: <message>"

 2  SPAN         begin tracing span "grepExecute"

 3  IF args.pattern === ""
        THROW INVALID_ARGS 'pattern is empty. Pass "." to match every line.'

 4  FOR flag IN ["-A", "-B", "-C"]
        v = args[flag]
        IF v !== undefined AND v < 0
            THROW INVALID_ARGS `${flag} is ${v}. Pass 0 or more context lines.`

 5  IF broadGlobGuardEnabled AND args.glob !== undefined
        IF DISALLOWED.has(args.glob.trim())
            THROW INVALID_ARGS `Glob pattern "${args.glob.trim()}" matches every file and is not allowed. Use a more specific glob or no glob.`

 6  IF args.path !== undefined AND isAgentTranscriptsPath(args.path)
        log "Model accessed agent transcript path"     // observational only

 7  offset     = clampInt32(floor(args.offset))   ?? 0
    headLimit  = clampInt32(ceil(args.head_limit)) || undefined

 8  bare       = GrepArgs{ pattern, path, glob, type, outputMode, multiline,
                            caseInsensitive: args["-i"],
                            contextBefore:  clampInt32(ceil(args["-B"])),
                            contextAfter:   clampInt32(ceil(args["-A"])),
                            context:        clampInt32(ceil(args["-C"])),
                            toolCallId }
    full       = GrepArgs{ ...bare, headLimit, offset }

 9  cached = cache.get(bare)                       // key EXCLUDES headLimit and offset
    IF cached !== undefined
        SKIP to step 11 with the cached result

10  result = executeToolCall( LocalGrepExecutor.execute(ctx, bare, {execId}) )
        on result.case === "error"   THROW classify(error, {pattern})
        on result.case === "success" use it
        else                         THROW `Unexpected result case: ${case}`

11  result = paginate(result, offset, headLimit)
    cache.set(bare, result)                          // only when step 9 missed

12  IF mcpPathTelemetryEnabled AND result defined
        histogram response byte length

13  RENDER
        error case        -> "Error running tool: " + headTail(error, 15000)
        not success       -> "No results found."
        zero workspaces   -> "No results found."
        success           -> envelope per workspace, mode-specific body

14  IF boostSemanticSearch AND not yet shown AND result is success
        IF count of "|" alternatives in pattern >= semanticSearchReminderMinPatterns (default 5)
            append the semantic-search system reminder, once per session
```

Five ordering facts that are load bearing:

**Step 3 runs before step 5.** An empty pattern with a broad glob produces the empty-pattern error, and the broad-glob check never runs. If you reorder them you get a different message for `{"pattern": "", "glob": "**/*"}`, and the model will be told about the glob when the problem is the pattern.

**Step 4 runs on the raw value.** The `fIe` clamp happens at steps 7 and 8. So `-A: -1` is caught by the negativity check, and by the time it reaches the clamp it can no longer be negative. If you clamp first, the check is unreachable and the error string never fires. The parent spec's conformance item "A negative context value is rejected on the raw pre-clamp value rather than silently clamped" is about this ordering.

**Step 9 keys on `bare`, which excludes pagination.** So `{pattern: "x", head_limit: 2}` and `{pattern: "x", head_limit: 50, offset: 10}` are one cache entry. Whichever runs first decides the underlying snapshot for the other. This is deliberate: it is what makes `offset` paging coherent. It is also why a stale search after a write can be returned.

**Step 11 runs after the executor, unconditionally on success**, including on a cache hit. So pagination is never cached, only the unpaginated result. That is what lets the same cached search serve two different `head_limit` values.

**Step 14 is once per session**, guarded by a `let a = !1` closed over the factory call. So the reminder appears at most once even if every subsequent search has five alternatives.

### Inside the executor

```
LocalGrepExecutor.execute(ctx, args, {execId, hookContextCollector})

 1  outputMode = args.outputMode ?? "content"
 2  roots      = Array.isArray(workspacePath) ? workspacePath : [workspacePath ?? cwd]
 3  IF worktreeGuard({targetPath: args.path, workspacePaths: roots, mainWorktreePath})
        RETURN error "You cannot search other worktrees for this repository, stay within your workspace paths."

 4  outside = findOutsideTargets(args.path, roots)
       // undefined if the path is empty, or if it resolves inside ANY root
       // otherwise the deduplicated absolute targets

 5  IF args.path !== undefined AND outside !== undefined
        chk = checkOutsideTarget(args.path, outside)
        IF "error" in chk  RETURN error chk.error
        roots = roots.slice(0, 1)
        args  = args.clone(); args.path = outside[0]

    ELSE IF args.path !== undefined
        targets = []
        ignoredFlag = false
        FOR each {root, resolvedRoot, target} of resolveSearchTargets(args.path, roots)
            IF NOT isPathInsideWorkspaceRoot(target, resolvedRoot)  CONTINUE
            real = realpath(target)
            IF real === null                          CONTINUE
            IF isCursorIgnored(real)                  ignoredFlag = true; CONTINUE
            IF isNeitherFileNorDirectory(target)      RETURN error "Path '<path>' is neither a file nor a directory"
            targets.push(root)
        IF targets.length === 0
            RETURN error (ignoredFlag
                          ? "Path '<path>' is filtered out by .cursorignore"
                          : "Path '<path>' could not be verified while resolving symlinks. ...")
        roots = targets

 6  IF outside === undefined
        indexed = executeIndexedGrepWithTimeout(ctx, roots, args, telemetry)
        IF indexed !== undefined
            per root, filterBlockedFiles on matches/files/counts
            SKIP to step 9 with indexed results
        // a 25 s timeout here falls back silently to ripgrep

 7  streaming = streamingEnabled(ctx, args)          // true unless useStreamingGrep says otherwise
    deadline  = streaming ? startDeadline(25000) : undefined
    stoppedCount = 0; allFailed = false
    FOR each root of roots
        IF deadline?.signal.aborted
            results[root] = emptyResultWith(ripgrepTruncated: true)
            stoppedCount++; allFailed = true; CONTINUE
        IF NOT pathExists(root)
            RETURN error `Path does not exist: ${root}`
        perRoot = deadline ?? startDeadline(25000)
        TRY  results[root] = runMode(ctx, outputMode, root, args, streaming, perRoot.signal, sandbox, telemetry)
             IF timedOut  stoppedCount++
        CATCH TimeoutError
             results[root] = emptyResultWith(ripgrepTruncated: true)
             stoppedCount++; allFailed = true
             IF NOT streaming  BREAK
        FINALLY perRoot.clear()
    deadline?.clear()
    IF stoppedCount > 0
        failed = allFailed && (!streaming || NO root produced non-empty output)
        metric("deadline_reached", {outcome: failed ? "failed" : "partial"})
        IF failed  THROW "Timed out after 25s"

 8  dedupeAcrossRoots(results, roots)     // when roots.length > 1

 9  RETURN success { pattern, path, outputMode, workspaceResults }
```

Three ordering facts here:

**Step 5's outside-target branch narrows to one root and overwrites `args.path`.** A path outside the workspace therefore searches one root, and the display path in the result is absolute rather than workspace-relative.

**Step 6's indexed path has no deadline of its own beyond the 25000 ms the provider gets.** On timeout it returns `undefined` and ripgrep runs, so the worst case for one call is 50 seconds. The spec's "25 s timeout" applies to each engine separately.

**Step 7's `deadline` is created once for all roots when streaming**, so the 25 s budget is shared across roots on the streaming path. But `runMode` also gets `perRoot = deadline ?? startDeadline(25000)`, so on the non-streaming path each root gets its own fresh 25 s. That asymmetry means a two-root non-streaming search can take 50 seconds.

### runMode, and the three modes

```
runMode(ctx, mode, root, args, streaming, signal, sandbox, telemetry)
  sortAfterRipgrep = sortsAfterRipgrep(root, args, streaming)
  SWITCH mode
    "content"            -> runContentMode(...)
    "files_with_matches" -> runFilesMode(...)
    "count"              -> runCountMode(...)
    default              -> THROW `Unknown output mode: ${mode}`
```

```
runContentMode
 1  argv      = buildArgs(root, args, "content", sortAfterRipgrep)
 2  hasCtx    = args.context !== undefined || args.contextBefore !== undefined
                            || args.contextAfter !== undefined
 3  lineBudget = 2000 * (hasCtx ? 2 : 1)                 // 2000 or 4000
 4  out       = executeRg(ctx, root, argv, lineBudget, signal, sandbox, telemetry)
 5  throwOnRipgrepFailure(out, args, root)
 6  text      = decodeLinesAsRead(ctx, out, args, telemetry)
 7  parsed    = parseContentOutput(text, 2000, lineBudget, ctx)
 8  files     = Array.from(parsed.byFile.values())
 9  allowed   = filterBlockedFiles(files, root)
10  dropped   = files.filter(f => !allowed.includes(f))
11  lineDelta   = sum of dropped[i].matches.length
    matchDelta  = sum of dropped[i].matches.filter(m => !m.isContextLine).length
12  ordered   = sortAfterRipgrep ? orderNewestFirst(ctx, allowed, root, f => f.file, telemetry) : allowed
13  RETURN contentResult{ matches: ordered, totalLines: totalLines - lineDelta,
                          totalMatchedLines: totalMatchedLines - matchDelta,
                          clientTruncated: parsed.clientTruncated,
                          ripgrepTruncated: parsed.ripgrepTruncated || out.timedOut || out.stdoutByteCapped }
```

```
runFilesMode
 1  argv     = buildArgs(root, args, "files_with_matches", sortAfterRipgrep)
 2  out      = executeRg(ctx, root, argv, undefined, ...)   // undefined -> 10000
 3  throwOnRipgrepFailure(out, args, root)
 4  parsed   = parseFilesOutput(out.stdout, sortAfterRipgrep ? 10000 : 2000, 10000)
 5  overCap  = parsed.totalFiles > 2000
 6  files    = parsed.files
 7  IF sortAfterRipgrep
        files = keepNewestPage(ctx, files, root, identity, telemetry)   // sorts, then slices to 2000
        IF overCap  files = files.join("\0").split("\0")                // re-split after the sort
 8  allowed  = filterBlockedFiles(files.map(file => ({file})), root).map(x => x.file)
 9  dropped  = files.length - allowed.length
10  RETURN filesResult{ files: allowed, totalFiles: totalFiles - dropped,
                        clientTruncated: overCap,
                        ripgrepTruncated: parsed.ripgrepTruncated || out.timedOut || out.stdoutByteCapped }
```

```
runCountMode
 1  argv     = buildArgs(root, args, "count", sortAfterRipgrep)
 2  out      = executeRg(ctx, root, argv, undefined, ...)
 3  throwOnRipgrepFailure(out, args, root)
 4  parsed   = parseCountOutput(out.stdout, sortAfterRipgrep ? 10000 : 2000, 10000)
 5  counts   = sortAfterRipgrep ? keepNewestPage(ctx, parsed.counts, root, c => c.file, telemetry)
                                : parsed.counts
 6  allowed  = filterBlockedFiles(counts, root)
 7  keptSum  = sum of allowed[i].count
    allSum   = sum of counts[i].count
 8  RETURN countResult{ counts: allowed, totalFiles: totalFiles - droppedFiles,
                        totalMatches: totalMatches - (allSum - keptSum),
                        clientTruncated: parsed.totalFiles > 2000,
                        ripgrepTruncated: parsed.ripgrepTruncated || out.timedOut || out.stdoutByteCapped }
```

The trap in `runFilesMode` step 7: `keepNewestPage` slices, and then the code does `p && (m = m.join("\0").split("\0"))`. That re-split is a no-op on an array of strings with no embedded NUL. It looks like a bug but is not: the intent was to restore the array shape after `join`, and `split` does that. An implementation that removes it gets the same result.

The trap in `runCountMode` step 5: `keepNewestPage` is called with `(e) => e.file` as the path extractor and the **whole count record** as the item, so the sort is over records and the slice is over records. Fine. But `runFilesMode` passes `(e) => e` as the extractor because its items are already bare strings. Two different shapes for the same helper.

### buildArgs, which decides everything

```
buildArgs(root, args, mode, sortAfterRipgrep)

  argv = []

  // 1. ignore files, one pair each
  FOR f of listCursorIgnoreFilesByRoot(root)
      argv.push("--cursor-ignore", f)

  // 2. mode
  listFiles = (mode === "files_with_matches" && args.pattern === "" && Boolean(args.glob))
  SWITCH mode
    "content"
        argv.push("--line-number", "--with-filename", "--no-heading", "-0")
        IF args.contextBefore !== undefined  argv.push("--before-context", str(args.contextBefore))
        IF args.contextAfter  !== undefined  argv.push("--after-context",  str(args.contextAfter))
        argv.push("--max-columns", "1000", "--max-columns-preview")
        IF args.context !== undefined
           AND args.contextBefore === undefined
           AND args.contextAfter === undefined
            argv.push("--before-context", str(args.context), "--after-context", str(args.context))
    "files_with_matches"
        argv.push(listFiles ? "--files" : "-l")
    "count"
        argv.push("-c", "--with-filename")
    default
        THROW `Unknown output mode: ${mode}`

  // 3. case
  IF args.caseInsensitive === true  argv.push("--ignore-case")
  ELSE                              argv.push("--case-sensitive")

  // 4. filters
  IF args.type                          argv.push("--type", args.type)
  IF args.glob  argv.push("--iglob", args.glob)
                IF listFiles  argv.push("--iglob", "!**/.git")
  IF args.multiline  argv.push("--multiline", "--multiline-dotall")

  // 5. sort
  sortMode = sortAfterRipgrep ? "none" : (args.sort ?? "modified")
  IF sortAfterRipgrep  argv.push("--line-buffered")
  IF sortMode !== "none"  argv.push(args.sortAscending === true ? "--sort" : "--sortr", sortMode)

  // 6. fixed flags
  argv.push("--no-config", "--color=never")
  argv.push("--hidden")
  argv.push("--follow")

  // 7. encoding, single-file target only
  target = searchTarget(root, args)
  IF args.path !== undefined
      enc = encodingForFile(resolve(target, root))     // undefined unless a regular file
      IF enc !== undefined AND isUtf16Family(enc)  argv.push("--encoding", enc.toLowerCase())

  // 8. pattern and target
  IF NOT listFiles  argv.push("--regexp", args.pattern)
  argv.push("--")
  argv.push(target)
```

The concrete argv for a default content search in a single-root workspace, which is the case you will test first:

```
rg
--line-number --with-filename --no-heading -0
--max-columns 1000 --max-columns-preview
--case-sensitive
--line-buffered
--no-config --color=never
--hidden
--follow
--regexp <pattern>
--
.
```

I ran that against the shipped binary and got `.\\a.txt\u00003:line3 const alpha = 1\n...`. Note the `.\` prefix: ripgrep is given `cwd` and the positional `.`, so its output paths are `.\name` on Windows and `./name` on POSIX.

Order facts that matter for reproducibility:

**`--cursor-ignore` pairs come first, before the mode.** Order within the argv does not change ripgrep's behavior for these, but a byte-comparison test of the argv would.

**`--before-context` is pushed before `--after-context`.** Irrelevant to ripgrep, matters for an argv test.

**The `-C` expansion happens after `--max-columns`.** So the argv order is `--max-columns 1000 --max-columns-preview --before-context N --after-context N`, which separates the expansion from the other context flags. Cosmetically odd, behaviorally identical.

**`--case-sensitive` or `--ignore-case`, never neither.** One of the two is always pushed. This is the correction to the spec's claim that `-i` works by omitting `--case-sensitive`.

**`--hidden` and `--follow` are unconditional and appear after the sort flags.** So an ignored dotfile is searched, and a symlink is followed, on every call, with no parameter to change either.

**`--max-count` never appears.** I grepped for both `--max-count` and `max-count` across all 9,814,287 bytes and got zero hits in the entire bundle.

**`--no-ignore` appears once**, at offset 8619613, and it is in a different function: the ripwalk listing client, `["--files","--hidden","--no-require-git","--no-config","--color=never"]` with an optional `--no-ignore-vcs`. That is the ignore *scanner*. Grep never emits it.

There are two ripgrep builders in this bundle, and they disagree on three defaults. `buildArgs` is the one Grep and Glob both use. `ripwalk` is a separate file-walking client that neither tool reaches. `buildArgs` never passes `--no-require-git`, always passes `--follow`, and pushes `--case-sensitive` unless the caller set `caseInsensitive` to exactly true. `ripwalk` always passes `--no-require-git`, follows symlinks only when asked, and treats `caseSensitive` as false by default. So a conclusion drawn from the wrong builder comes out inverted on `.gitignore` outside a repository, on symlink traversal, and on case default. `tool-glob.md` carries the full table.

### The NUL wire format, and why splitting on `:` breaks

In content mode `-0` is passed, which is ripgrep's `--null`. Its effect, from `rg --help` on the shipped binary: "Whenever a file path is printed, follow it with a NUL byte. This includes printing file paths before matches."

So a match line on the wire is:

```
<path> 0x00 <lineNumber> : <content> \n
```

and a context line is the same with `-` instead of `:`. Nothing sits between the path and the NUL.

I confirmed byte for byte on a file named `weird:name.txt`:

```
weird:name.txt\u00001:NEEDLE colon\n
```

and on an ordinary recursive search:

```
.\\a.txt\u00003:line3 const alpha = 1\n.\\a.txt\u00009:line9 const gamma = 3\n
```

The parser, `Ax`, works like this:

```
parseContentOutput(stdout, clientLimitLines, ripgrepHardCutoffLines)

 byFile = Map(), allFiles = Set()
 totalLines = 0, totalMatchedLines = 0, linesProcessed = 0

 IF stdout.trim() === ""  RETURN empty

 FOR each line [lineStart, lineEnd) of stdout
     totalLines++
     IF the line is exactly "--"  CONTINUE                  // ripgrep's own separator

     nul = first index in [lineStart, lineEnd) with charCode 0
     IF nul === -1  CONTINUE                                // no NUL: not a match line
     p = nul + 1
     IF p >= lineEnd  CONTINUE

     lineNumber = 0; digits = 0
     WHILE p < lineEnd AND charCode(p) is 0x30..0x39
         lineNumber = lineNumber*10 + (charCode(p) - 0x30); digits++; p++
     IF digits === 0 OR p >= lineEnd  CONTINUE

     sep = charCode(p)
     IF sep !== 0x3A (':') AND sep !== 0x2D ('-')  CONTINUE

     isContextLine = (sep === 0x2D)
     IF NOT isContextLine  totalMatchedLines++

     filename = stdout[lineStart .. nul)
     allFiles.add(filename)

     IF linesProcessed >= clientLimitLines  CONTINUE         // NOTE: continue, not break
     content = stdout[p+1 .. lineEnd)
     byFile[filename].matches.push({lineNumber, content, isContextLine})
     linesProcessed++

 RETURN { byFile, totalLines, totalMatchedLines, totalFiles: allFiles.size,
          clientTruncated: linesProcessed >= clientLimitLines,
          ripgrepTruncated: totalLines >= ripgrepHardCutoffLines }
```

Four things about that parser that a reimplementation gets wrong:

**It counts every line, even ones it skips.** `totalLines` increments before any filtering, and `ripgrepTruncated` is `totalLines >= hardCutoff`. So the ripgrep-truncation signal reflects the *raw* line count rather than matches.

**It continues rather than breaking at the client cap.** Once `linesProcessed` hits 2000, every remaining line is still scanned for `totalLines`, `totalMatchedLines`, and `allFiles`. That is how the trailer can say `at least 4211` while showing 2000: the totals are complete even though the kept entries are not. An implementation that `break`s early loses the correct total.

**`clientTruncated` is `>=`, not `>`.** So 2000 entries sets `clientTruncated: true` even if there are no more. That is why a search with 2000 matches still gets a `Results truncated.` footer. It is an off-by-one in Cursor's favor of caution, and you should decide deliberately whether to keep it.

Note the interaction with the `head_limit` case: a 2000-entry result that hit the parse cap has `reachedEnd` true (because `headLimitApplied` is undefined), so `cutBeforeEnd` is true and the prefix appears. A `head_limit: 2000` result that happened to fit has `reachedEnd` false, so the prefix does not. Same entry count, different footer, and the difference is entirely in whether a user limit was supplied.

**The `--` check comes before the NUL scan.** A line of exactly two hyphens is skipped even if it somehow contained a NUL, which it cannot, so the check is a redundant safeguard, as the research notes.

If you split each line on `:` and take the first field as the path, the parser fails in these ways:

- `src/a.ts\0 3:const x` splits on the first `:` into `src/a.ts\0 3` and `const x`. You get a path containing a NUL and a line number, so the path is wrong and the line number is unparseable.
- `C:\src\a.ts\0 3:const x` splits on the first `:` into `C` and `\src\a.ts\0 3:const x`. You get the file `C` and lose the path entirely. On Windows every result is wrong.
- `weird:name.txt\0 1:NEEDLE` splits on the first `:` into `weird` and `name.txt\0 1:NEEDLE`.

The NUL exists so that no path character can be confused with the separator, because NUL cannot appear in a path on any supported filesystem. The `Ax` parser finds the *first* NUL and takes everything before it as the path, with no ambiguity. If `-0` were omitted, Cursor would have to parse from the right, which is what `count` mode does (`lastIndexOf(":")`) because it does not pass `-0`.

The `count` parser splits on the last colon, and I confirmed why: `weird:name.txt:1` parses to file `weird:name.txt` and count `1`, and `C:\src\a.ts:3` parses to `C:\src\a.ts` and `3`.

### The 8 MiB stdout cap and the 10,000 line budget

**8 MiB, in `executeRg`.** It accumulates chunks and kills the child on exceed:

```js
h.stdout?.on("data",(e=>{
  if(!b&&!_)                                     // not byte-capped, not line-capped
    if(y+=e.length, y>c) { b=!0; try{h.kill()}catch{} }   // c = 8388608
    else if(void 0!==n) {
      let t=-1,r=0;
      for(let s=0;s<e.length;s++) if(10===e[s] && (r++, v+r>=n)) { t=s+1; _=!0; try{h.kill()}catch{}; break }
      v+=r;
      const s=t>=0?e.subarray(0,t):e;
      f.push(s)
    } else f.push(e)
}))
```

The byte cap protects the **agent process's memory**. Without it, `rg` on a huge tree can emit gigabytes and Node buffers all of it. When it trips, `stdoutByteCapped` becomes true, which ORs into `ripgrepTruncated` and therefore into the trailer.

The line budget protects **work**. `n` is `2000` for content or `4000` with context, and `10000` for files and count. The loop counts newlines in each chunk and kills the child on reaching the budget, keeping only whole lines up to that point. That is how a 4211-match search finishes in milliseconds instead of scanning the whole tree.

Note the two are on the same `if` chain with `if(!b&&!_)`, so once either trips, no further chunks are read. And the byte cap is checked first, so a single chunk that blows the byte budget is not also line-scanned.

| Cap | Protects | Model-visible effect on trip |
|---|---|---|
| 8 MiB stdout | the agent process heap | `Results truncated. ` footer, `at least` in the total |
| 2000 / 4000 line budget | wall clock and CPU, per root | same footer, same `at least` |
| 10000 parse window (files/count) | the parser's memory on a huge listing | `ripgrepTruncated` true, count suppressed in some renderers |
| 2000 client slice | context window | `clientTruncated`, `at least`, `Use offset=...` |
| 15000 rendered chars | context window, the real limiter | `Results truncated. ` plus `shown < returned` |

The ordering of those five, from cheapest to most visible: the client slice and the render budget both bind in ordinary use, and the byte cap and line budget only bind on pathological trees.

## Engineering questions

### Q1. Pagination and chunking

**Yes, there is paging, and it is `offset` plus `head_limit` applied client-side.** There is no cursor, no token, no page number. The caller resumes by re-issuing the search with `offset` advanced by the number of entries previously shown, which the trailer tells them (`Use offset=12 to see more`).

The mechanics, from `sIe` for content and `nIe` for files and count:

```
paginateContent(content, offset, headLimit)
  limit    = headLimit ?? Infinity
  skipped  = 0        // matched entries skipped so far, GLOBAL across files
  collected= 0        // matched entries emitted so far, GLOBAL across files
  hasMore  = false
  out      = []
  FOR each fileMatch of content.matches
      IF hasMore  BREAK
      lines = []; pending = []
      FOR each line of fileMatch.matches
          IF line.isContextLine
              IF lines is not empty  lines.push(line)  ELSE pending.push(line)
              CONTINUE
          IF skipped < offset
              skipped++; pending = []                  // drop the leading context too
              CONTINUE
          IF collected >= limit
              hasMore = true; BREAK
          lines.pushAll(pending); pending = []
          lines.push(line); collected++
      IF lines is not empty  out.push({file: fileMatch.file, matches: lines})
  RETURN contentResult{ matches: out,
                        clientTruncated: content.clientTruncated || hasMore,
                        headLimitApplied: headLimit,
                        offsetApplied: offset > 0 ? offset : undefined, ... }
```

Four behaviors:

**Both counters are global across files.** A `head_limit` of 2 with two matches in the first file and three in the second returns two matches, both from the first file, and the second file never appears. This is the first item in the traps list.

**Context lines are buffered and flushed only when the match they precede is emitted.** A leading context block whose match is skipped never appears. That is why an `offset` landing mid-file does not produce orphaned context above the first shown match.

**A file with every match skipped produces no entry at all**, from the `lines.length > 0` guard before the push. So the rendered output never has a bare path line with nothing under it.

**`offsetApplied` is only set when `offset > 0`.** The footer and the empty-result strings both read it, which is why they can never print `offset=0`.

There is an early return that skips all of this:

```js
function rIe(e,t,r){ if("success"!==e.result.case) return e; if(0===t && void 0===r) return e; ... }
```

So `offset: 0` with no `head_limit` leaves the result untouched, including leaving `headLimitApplied` and `offsetApplied` unset. That matters for the trailer: with no pagination applied at all, `reachedEnd` is `headLimitApplied === undefined` which is true, and `cutBeforeEnd` falls back to `clientTruncated || end < total`. So the 2000-parse-cap case still gets a correct footer without any pagination having run.

**A page boundary never falls mid-line or mid-UTF-8.** The reason is structural rather than careful. Pagination operates on the parsed `matches` array, where each element is one whole line already decoded. There is no byte or character slicing anywhere in the pagination path, so a boundary cannot fall inside a line or inside a code point. The only slicing that can fall mid-line is the 15000-character render budget, which does `content.substring(0, lastNewlineIndex)` so it cuts at a line boundary by construction, and even then it can fall inside a surrogate pair if a line contains astral characters near the cut, which would leave a lone high surrogate at the end of the model-visible string. That is a narrow defect.

```
paginateArray(items, offset, headLimit)
  start = min(offset, items.length)
  end   = headLimit !== undefined ? min(start + headLimit, items.length) : items.length
  RETURN { items: items.slice(start, end),
           truncated: headLimit !== undefined && start + headLimit < items.length }
```

Note `truncated` here is `start + headLimit < length`, a strict `<`, whereas the content parser uses `>=`. So files and count mode can say "not truncated" where content mode would say "truncated" for the same boundary. A one-entry search with `head_limit: 1` in files mode reports no truncation; the same in content mode reports truncation.

**The trailer's advice is mostly accurate, with two exceptions.**

Accurate: `Use offset=N to see more` where N is `offsetApplied + shown`. Since pagination skips `offset` matched entries and emits `shown` more, re-issuing with `offset=N` is correct.

Inaccurate, case one: when the cut came from the **15000-character render budget**, the advice is still `Use offset=N to see more`, and it works, because `N` is the count of *matched* lines rendered, so the next page starts at the right match.

Inaccurate, case two: when `ripgrep` was truncated, the trailer says `Narrow the search to see the rest.` and does not offer an offset. That is correct, because there is nothing more to page to. But the total is a lower bound and the string `at least` signals it, so the model is told the truth.

The inaccuracy that matters: **`head_limit` counts matched entries, not rendered lines.** A `head_limit: 2` with `-A: 5` renders 2 matches plus up to 10 context lines, so `Showing 2 of N matching lines` understates what the model received by a factor of six. The unit word is `matching lines`, which names what is counted, but a model reading `Showing 2` after seeing 12 lines has no way to reconcile the two.

**Other harnesses.** Besides this tool only Claude Code and Zed page; the full table is `### How each harness pages`. Claude Code uses the same global `offset` plus `head_limit`, with a default of 250 (source, `K2e`). Zed pages with `offset` and a fixed 20 and re-runs the search on every page. Its `offset` counts raw regex ranges while its page counts merged blocks, so after a merge page 2 repeats page 1. With 25 blocks made of 50 raw ranges, page 2 at `offset: 20` shows blocks 11 to 25 under `Found 15 matches:` (source, hand trace of `grep_tool.rs:299-305` and `:382` at `37faef81f7`; not run, and no test sets `offset` above 0). The other nine have a cap and no resume, and pi's trailer says `Use limit=2N`, which restarts the search; with an unstable order the larger run need not extend the smaller one (probe). So a page that neither repeats nor skips needs a stable order and one unit, and only Zed has the order.

### Q2. Units

Five different units in four different layers, and a wrong unit produces a plausible-looking wrong answer.

| Quantity | Unit | Where compared | Exact comparison |
|---|---|---|---|
| content entry cap | entries (match + context lines) | parser `Ax` | `linesProcessed >= clientLimitLines`, then `continue` |
| ripgrep line budget | stdout lines | `executeRg` | `v + r >= n` where `r` counts `\n` in the chunk |
| files kept | file paths | `runFilesMode` | `d.totalFiles > Lx.CLIENT_LIMIT_LINES` |
| count rows kept | file-count rows | `runCountMode` | `d.totalFiles > Lx.CLIENT_LIMIT_LINES` |
| files/count parse window | stdout lines | `Cx` / `Px` | `o >= r` sets `ripgrepTruncated` |
| stdout cap | **bytes** | `executeRg` | `y += e.length` (`e` is a Buffer), `y > 8388608` |
| rendered budget | **UTF-16 code units** | `renderMultiWorkspace` | `section.length`, JS string length |
| mtime budget | milliseconds | `sortNewestFirst` | `withTimeout(Promise.all(...), 2000)` |
| ripgrep deadline | milliseconds | `startDeadline` | `setTimeout(() => controller.abort(), 25000)` |
| long-line cut | **bytes** | ripgrep itself | `--max-columns 1000` |
| long-line preview | **characters** | ripgrep itself | `--max-columns-preview` |
| error text | characters | `EG` | `e.length > 15000` |
| `head_limit` / `offset` | entries | `sIe` / `nIe` | `collected >= limit`, `skipped < offset` |

The three that surprise people:

**The stdout cap is bytes and everything else about output is characters.** `y` accumulates `e.length` where `e` is a `Buffer`, so it is byte length. The 15000 ceiling is `string.length`, which is UTF-16 code units. A file of CJK text therefore hits the byte cap at roughly a third of the character count that a Latin file would, but hits the render budget at the same character count, so two limits with different units act on the same stream.

**`--max-columns` decides in bytes and previews in characters.** I bisected this against the shipped binary:

| Line | Bytes | Chars | Cut? |
|---|---|---|---|
| 999 ASCII | 999 | 999 | no |
| 1000 ASCII | 1000 | 1000 | **yes** |
| 998 of 2-byte chars | 998 | 499 | no |
| 1000 bytes of 2-byte chars | 1000 | 500 | **yes** |
| 996 bytes of 3-byte chars | 996 | 332 | no |
| 1002 bytes of 3-byte chars | 1002 | 334 | **yes** |
| 249 emoji | 996 | 249 | no |
| 250 emoji | 1000 | 250 | **yes** |

So the rule is "cut when the line is 1000 bytes or more", confirmed at four different character widths. Then the preview keeps 1000 *characters*: a 3600-byte CJK line previews 1000 characters (3000 bytes), and a 2200-byte accented line previews 1000 characters (2000 bytes). Both measured by decoding the output and counting.

**The `head_limit` counter does not see context lines.** `sIe` increments `collected` only in the non-context branch. So `head_limit: 10` with `-A: 3` is 10 matches plus up to 30 context lines, which is 40 entries against a parser cap of 4000. The numbers are in different units, and the description's phrase `limits total matches shown` is precise about that.

**What a reimplementation should do.** Use bytes for anything derived from a byte stream (stdout cap, line cut) and Unicode scalar values for anything derived from text (render budget, error truncation, line preview). Do not mix. If you want one unit, make it bytes everywhere for caps and scalars everywhere for text, and never convert between them implicitly. The current design's mixed units are a consequence of three subsystems each choosing locally, and they produce the odd behavior where a CJK-heavy result hits the byte cap early and the character budget late.

**Other harnesses.** The caps count different things in each. Zed's page counts merged blocks and its `offset` counts raw ranges. Pi's `limit` counts ripgrep `match` events (matching lines, context not counted) and its 500-unit line cut counts UTF-16 units. Rig's 200 counts hit lines across files and its 300-character clip counts Unicode scalar values (`char_indices`), so 300 `é` is 600 bytes. Gemini CLI's 100 counts non-context matches while `--max-count` counts per file, and its 2000-unit line cut counts code points. OpenCode's 100 counts matching lines, its 2000-unit preview counts UTF-16 units and its 64 KiB record limit counts bytes. Zeroclaw's 1000 counts lines, files or per-file lines by mode. Cline's 100 counts parsed match objects. (source, probe)

### Q3. Text encoding

**Invalid UTF-8 is handled in two passes, and the second pass only runs when you got lucky with a U+FFFD.**

Pass one is ripgrep, which does not validate and emits bytes. When Cursor decodes the child's stdout as UTF-8, invalid sequences become U+FFFD:

```js
function Rx(e,t){let r=Buffer.concat(e);t&&(r=r.subarray(0,r.lastIndexOf(10)+1));const n=r.toString("utf8");
  return n.includes("\uFFFD")?{stdout:n,stdoutBytes:r}:{stdout:n}}
```

The `t` flag (complete lines only) trims the buffer to the last newline before decoding, which matters because a partial final line could be a truncated multi-byte sequence that is valid once the rest arrives.

So if the decoded string contains a replacement character, `Rx` keeps the **raw bytes** alongside the string. Then `decodeLinesAsRead` gets a chance to fix individual files:

```
decodeLinesAsRead(ctx, out, args, telemetry)
  IF out.stdoutBytes === undefined  RETURN out.stdout            // nothing looked invalid

  // walk the decoded text line by line, tracking each line's byte range
  // for every line containing U+FFFD, record { file, contentStart, contentEnd,
  //                                             contentBytesStart, contentBytesEnd }

  files = distinct file names from those lines
  IF files.isEmpty  RETURN out.stdout

  deadline = now + 2000
  FOR each file, in order
      IF now > deadline  BREAK                                   // silently give up
      encoding = detectEncoding(realpath(file))                  // iconv-lite style
      IF encoding undefined OR utf8  CONTINUE
      FOR each recorded line of that file
          decoded = iconv.decode(bytes[contentBytesStart..contentBytesEnd], encoding, {stripBOM: false})
          splice decoded into the output text

  RETURN partially repaired text
```

Six things this does and does not do:

**It only runs when a U+FFFD appeared.** A file in a non-UTF-8 encoding whose bytes happen to decode into valid but wrong UTF-8 produces no replacement character, so no repair happens, and the model sees mojibake. This is the common case for Latin-1 text: `café` in Latin-1 is `caf\u00E9` in bytes, and `0xE9` alone is not valid UTF-8, so you do get U+FFFD. But a two-byte sequence like `0xC3 0xA9` in a Latin-1 file is valid UTF-8 for `é`, so a Latin-1 file that happens to contain those two bytes decodes as UTF-8 with no warning and the wrong characters. There is no way to detect that without an encoding hint.

**I reproduced the U+FFFD path.** A file containing `aaa caf` then the single byte `0xE9` then ` NEEDLE zzz` produced, from raw ripgrep:

```
badutf8.txt\u00001:aaa caf\uFFFD NEEDLE zzz\n
```

So the model, absent a successful repair pass, sees a replacement character in the middle of the line. Whether the repair pass fixes it depends on `detectEncoding`, which is outside this bundle.

**BOM handling.** ripgrep's default is to treat a UTF-8 BOM as a byte-order marker and not part of the first line. I verified: a file beginning `EF BB BF` then `NEEDLE first` returns `bom.txt\u00001:NEEDLE first`, with no BOM in the content. And a pattern matching the BOM itself (`^\uFEFF`) found nothing, which confirms ripgrep consumed it. `decodeLinesAsRead` passes `{stripBOM: false}` to iconv, so on the repair path the BOM would be preserved in the spliced text, so one tool has two BOM policies, depending on whether the repair pass ran.

**UTF-16.** `buildArgs` detects the encoding of a single-file target and passes `--encoding utf-16le` if the detected encoding is UTF-16 or UCS-2 family (`/^(utf-?16|ucs-?2)/i`). For a directory search no `--encoding` is passed, and ripgrep sniffs. I verified both paths return identical output for a UTF-16LE file with a BOM: `utf16.txt\u00001:NEEDLE wide\r\n`. So the explicit flag is redundant for the single-file case, and ripgrep's own BOM detection covers the rest.

**Lone surrogates.** A JS string cannot hold a lone surrogate through `toString("utf8")` without becoming U+FFFD, because Node's UTF-8 decoder replaces them. So a search matching a lone surrogate in a file is not possible: the byte sequence for a surrogate does not exist in UTF-8. In UTF-16 input, ripgrep decodes and would produce U+FFFD. Not a case you can meaningfully match.

**Unicode normalization.** **None is applied anywhere.** No NFC, no NFD, no case folding beyond `--ignore-case`, no width folding. So a file containing `é` as U+00E9 (precomposed) will not match a pattern written as `e` + U+0301 (combining). Both are visually identical. This is correct and expected for a byte-oriented search, but a model writing a pattern from memory may use either form. Same for CJK: a full-width `Ａ` and an ASCII `A` are different characters with no folding.

The `-i` flag is `--ignore-case`, which is ripgrep's Unicode-aware case-insensitive mode by default. It folds `İ` to `i` in a locale-sensitive way and does simple case folding rather than full folding, so `ß` does not match `ss`.

**Other harnesses.** Three answers for text that is not UTF-8. Zed decodes before matching: BOM, then a UTF-16 guess, then `chardetng`, so GBK, EUC-KR and UTF-16 files are found and the BOM stays out of the match offsets (source, engine test `project_tests.rs:13213-13272`). Rig skips a non-UTF-8 file whole with no trace, so the ASCII lines of a Latin-1 file are lost, and keeps a BOM in line 1 (probe). Pi, OpenCode and Cline ask ripgrep for JSON, which sends a non-UTF-8 line as base64 `bytes` with no `text`: pi re-reads the file and substitutes U+FFFD, Gemini CLI drops the line, Cline prints an empty text, and OpenCode fails the whole call with `Invalid ripgrep match output` (probe, derived from its schema). Pi's context mode also decodes a UTF-16 file as UTF-8 and prints `��n\u0000e\u0000...` even on the match line. Cursor's U+FFFD plus a 2 s repair pass is the only repair of its kind. Unicode normalization: none in any harness whose notes cover it, so NFC and NFD forms of one word do not match each other (probe for rig; for pi it follows from byte matching). Separately, Gemini CLI checks the pattern as a JavaScript `RegExp` and runs it as Rust, and Zed's parameter text names the `regex` crate while running `fancy_regex` (D11).

### Q4. Line endings

**The tool detects nothing and normalizes nothing. It splits on `\n` and strips one trailing `\r`.** Everything downstream sees whatever ripgrep emitted.

The line iterator, at 9006050:

```js
function xv(e,t,r){const n=e.length; if(t>=n)return!1;
  const s=e.indexOf("\n",t);
  if(-1!==s){const n=s>t&&e.charCodeAt(s-1)===wv; r.lineEnd=n?s-1:s; r.nextStart=s+1; return!0}
  r.lineEnd=n; r.nextStart=n; return!0}
```

with `wv = 13`. So:

| Input | Behavior |
|---|---|
| LF only | one line per `\n`, no `\r` stripping needed |
| CRLF | `\r` before `\n` is stripped from `lineEnd`, so the content has no `\r` |
| CR only | **no split at all.** The whole file is one line |
| mixed LF and CRLF | each CRLF line loses its `\r` individually; a bare LF line does not |
| mixed LF and CR | the CRs stay in the content, since only a CR immediately before `\n` is stripped |
| final line without a newline | emitted as a line |

I confirmed the CR-only case against the shipped binary. A file containing `first\rNEEDLE here\rthird\r` returned:

```
cr.txt\u00001:first\rNEEDLE here\rthird\r\n
```

One match, line number 1, with both internal CRs preserved as literal characters in the content. So a classic-Mac file is one enormous line to this tool, and `--max-columns 1000` will cut it if it is over 1000 bytes. There is no way for the model to get line numbers from such a file through Grep.

And I confirmed CRLF is stripped, because ripgrep normalizes it before Cursor sees it:

```
crlf.txt\u00002:NEEDLE here\n
```

Note the `\n` at the end with no `\r`. ripgrep's `--null` output still uses `\n` as the record separator and it strips the CR itself. So Cursor's `xv` `\r` strip is redundant on this path but harmless.

**What this means for a reimplementation.** Split on `\n` and strip one trailing `\r`, as Cursor does. Do not split on `\r` as well, because that changes the line numbering for CRLF files (it would be fine) and for CR-only files (it would be better, but it would diverge from Cursor). If you want to improve on it, split on `\r\n`, `\n`, or `\r` like Rust's `str::lines` does, and accept that your line numbers will differ from Cursor's on CR-only files, which is a very small population.

The one place a line ending could leak into output: the **last** line of a file with no trailing newline. ripgrep emits it without one, and `xv` sets `lineEnd = n`, so the content is the full remainder with no `\r` to strip.

**Other harnesses.** Line endings split three ways. Zed normalizes `\r\n` and a lone `\r` to `\n` in both the prefilter and the buffers, so a CR-only file matches with correct line numbers, where this tool sees one line (source, `search.rs:422`). Rig uses `str::lines`: `\n` and `\r\n` end a line, and a lone `\r` stays inside it (probe). Pi deletes every `\r` from its output, and fixed on 2026-10-27 a bug where a lone CR shifted every later context line (commit `fecc19e4e`); Cursor's `pi_grep` still turns a lone CR into a line break, so the match for `needle` prints as `progress`, and its fast path keeps a trailing `\r` (probe). OpenCode and Cline pass `\r\n` through inside the text, Gemini CLI's `trimEnd()` hides `\r`, and zeroclaw's internal backend uses `str::lines`.

### Q5. Binary detection

**The tool never decides. ripgrep decides, and the answer differs by output mode.**

Grep passes none of `--binary`, `-a`/`--text`, or `-I`/`--binary-files=without-match`. So ripgrep's default applies, which is "binary detection on, quit mode": on the first NUL byte in a file, ripgrep stops searching that file and reports it as a binary match.

What that produces, and what Cursor does with it, depends entirely on the mode.

**Content mode.** ripgrep prints one line per binary file:

```
binmatch.txt: binary file matches (found "\0" byte around offset 30)
```

I captured that output verbatim. Note there is **no NUL byte between the path and the message**, because `-0` only applies when a line number is printed, and there is no line number here. So `parseContentOutput` finds no NUL, hits `if(nul === -1) continue`, and the line is dropped. Even if a NUL were present, the character after it is a space, not a digit, so the line-number parse would reject it too.

**Result: a binary file is completely invisible in content mode.** No match, no error, no notice. The model cannot tell it was skipped.

**files_with_matches mode.** ripgrep's `-l` prints the path:

```
binmatch.txt
```

No NUL, no message. `Cx` splits on newlines and takes each line as a path, so the binary file **appears**.

**count mode.** ripgrep prints:

```
binmatch.txt:1
```

and `Px` parses the last colon, so the binary file **appears with a count of 1**.

So the same file is absent from content output and present in files and count output, for the same pattern. That is not a design decision; it is an accident of which parser handles which shape. The parent spec's worked-example table states this correctly, and you should reproduce it or fix it deliberately.

A second wrinkle: the count of `1` is misleading. ripgrep stops at the NUL, so it reports one match for a file that may contain a thousand. The `pi_grep` surface does not change this. In `--json` mode ripgrep reports `binary_offset` in the end event for a file named explicitly, but `pi_grep` reads only `match` events (`"match"!==t.type` returns early, in `main.js` at 9202537 and in the daemon), so it never sees that field. In a directory search ripgrep emits no match events at all for a file that holds a NUL (`bytes_searched: 0`), so there is nothing to act on: the file vanishes and the call says `No matches found`. `(bundle, daemon, probe)`

**A NUL after the matches changes nothing.** I tested a file with a match on line 1 and a NUL at the end. Same output: `binmatch.txt: binary file matches (found "\0" byte around offset 30)`. So ripgrep reports the binary verdict and does not print the matches it found, even though it found them.

**Permission errors.** `throwOnRipgrepFailure` suppresses errors that are purely about unreadable paths:

```js
throwOnRipgrepFailure(out, args, root){
  IF (out.stderr !== "" || out.exitCode === 2)
     AND out.stdout.trim() === ""
     AND NOT everyStderrLineIsTolerable(out.stderr, searchTarget(root, args))
  THEN THROW Error(out.stderr || `ripgrep exited with code ${out.exitCode}`)
}

everyStderrLineIsTolerable(stderr, target) =
  stderr.trim().split(/\r?\n/).every(line => {
     p = /^rg: (.+?): .*\(os error (?:13|5|2|1)\)$/.exec(line)?.[1]
      ?? /^rg: File system loop found: (.+) points to an ancestor /.exec(line)?.[1]
     return p !== undefined && p !== target
  })
```

Read it as: throw only when there is stderr or exit 2, stdout is empty, and *not* every stderr line is a permission-ish error about some path other than the search target itself.

Consequences:

- An unreadable **subdirectory** inside the searched tree produces `rg: <dir>: Permission denied (os error 13)` on stderr and results for everything else. Stdout is non-empty, so no throw. **The search succeeds and the model never learns a directory was skipped.**
- An unreadable **search target** (the `path` argument itself) produces the same line with `p === target`, so the tolerance test fails, and the tool throws with ripgrep's stderr verbatim.
- A symlink loop produces `rg: File system loop found: ...` and is tolerated if the loop is not at the target.
- **Empty stdout plus a permission error about the target** throws. Empty stdout plus a permission error about a subdirectory, on a tree where nothing else matched, does **not** throw, because `p !== target` is satisfied. So a search that found nothing and could not read three of its directories reports `No matches found` with no hint that anything was skipped.

The errno set for the *classifier* differs from the errno set in this regex. The regex matches `13|5|2|1` (EACCES, EIO, ENOENT, EPERM on Linux). The classifier's set is `xEe = new Set([1,2,3,5,13,32,33])`, which adds 3 (ESRCH), 32 (EPIPE), and 33 (EDOM). So an `(os error 32)` line is tolerated by the classifier but would fail the tolerance test in `throwOnRipgrepFailure`, which means it throws as a generic error and then gets classified as a user-facing error from the errno set. That inconsistency is harmless in practice.

**Other harnesses.** None of the harnesses whose notes cover a directory search tells the model that a binary file was skipped. Zed classifies the first 1,024 bytes and skips with a debug log (source, `project_search.rs:846-849`). Pi, OpenCode and Cline lose a NUL file silently because ripgrep reports nothing for it (probe). Rig has no check at all: a NUL inside valid UTF-8 is searched and shown raw (`nul.txt:1: before\0after secret`), and a non-UTF-8 file is skipped whole (probe). Zeroclaw's `rg` backend prints `binary file matches` for a file named on the command line and its formatter counts that line as a match (probe). Gemini CLI passes no binary flag; on an explicit file ripgrep returned the matches and put `binary_offset` in the end event, and its directory behavior was not probed. And on permission errors the harnesses split: Cursor tolerates an unreadable subdirectory when stdout is non-empty, while pi and zeroclaw let one unreadable file make ripgrep exit 2 and discard every match, and OpenCode keeps the rows and drops the `partial` flag (probe, source).

### Q6. Empty input and empty result

| Input | Behavior |
|---|---|
| **empty pattern `""`** | Thrown before any search: `pattern is empty. Pass "." to match every line.` |
| **a single space `" "`** | Passes validation and reaches ripgrep as a literal space. It matches every line that contains a space, and nothing in a file that has none. That is why the empty-pattern error message says `Pass "." to match every line` rather than mentioning a space. |
| **`.` as the pattern** | Matches every line. Confirmed: 12 of 12 lines on `a.txt`. |
| **empty file** | Contributes nothing in content mode. In files mode with `-l`, ripgrep does not match a file with no lines, so it does not appear. |
| **empty directory** | `No matches found` in content and count mode, `No files with matches found` in files mode. |
| **zero matches, offset 0** | `No matches found` (content, count) or `No files with matches found` (files). |
| **zero matches, offset > 0** | `No more results at offset=N.` |
| **offset past the end** | Same string as above, with the offset you asked for, not the last valid index. |
| **empty `glob` `""`** | Becomes `--iglob ""`, which ripgrep reads as a pattern matching nothing, so zero results. Not caught by the broad-glob guard because `"".trim()` is not in the set. |
| **empty `path` `""`** | `searchTarget` returns `.` because `if(!t.path)return"."` treats `""` as falsy. So an empty path means the workspace root. It does not error. |
| **empty `type` `""`** | `t.type &&` is falsy, so `--type` is not pushed. No error, no filtering. |
| **`head_limit: 0`** | Coerced to `undefined`, so unlimited. Confirmed in code; the behavior is all matches plus whatever caps apply. |
| **`offset: 0`** | No offset applied. `rIe` early-returns when offset is 0 and headLimit is undefined. |
| **`head_limit: 1`, one match** | **No footer at all.** `clientTruncated` stays false because no further match was found after the limit, and `describeNextStep` computes `fullPageMayHaveMore` as false on both terms, so it returns `undefined`. The `at least` hedge fires only when something is left. |
| **`multiline: true`, single-line pattern** | Works normally, just slower. `--multiline-dotall` makes `.` match newlines, which changes a pattern like `a.b` to match across lines. |
| **whitespace-only `pattern` with `multiline`** | No special case. |

The `Showing 1 of at least 1 matching lines` case reads wrong. The code cannot distinguish "there is exactly one match" from "there might be two, I stopped at one" without doing extra work, so it hedges. If you want to fix it, the fix is to note whether the scan reached the end before the limit was hit, which requires the parser to keep scanning. Cursor does keep scanning for `totalLines` and `totalMatchedLines`, so the information is there: `totalMatchedLines` would be 1, and the footer could compare. It does not, because `describeTotal` uses `total >= 1` and `clientTruncated` rather than comparing to `totalMatchedLines`.

**Other harnesses.** An empty pattern is an error in Cursor (`pattern is empty. Pass "." to match every line.`), OpenCode (`pattern is required`) and zeroclaw (`Empty pattern is not allowed.`), matches every line up to the cap in pi, rig, Gemini CLI and Cline, and matches nothing in Zed (`No matches found`) (source, probe). Zero matches reads `No matches.` in rig, `No files found` in OpenCode (a file-search wording for a content search), `No matches found for pattern "<p>" in path "<dir>"` in Gemini CLI, `No matches found.` in zeroclaw and Codex's removed tool, and `(no output)` plus `Command exited with code 1` as an error result in Goose. Zed's `offset` past the end prints the same `No matches found`, so the model cannot tell the end of results from an empty search; Cursor's `No more results at offset=N.` can. (source, probe)

### Q7. Cost

I measured these costs rather than estimating them, because they bear on the VFS question.

**The fixture.** 20,000 files across 200 directories, 60 lines each, 3.3 KB per file, 66.8 MB total, on a local SSD under Windows. Paths are around 82 characters.

| Measurement | Wall clock | Note |
|---|---|---|
| `rg` content search, full Grep argv | **554 ms** | 20,000 matching lines, 1.54 MB of stdout |
| `rg --files` listing | **53 ms** | 20,000 paths, 360 KB |
| `readdirSync` walk, no reads | **94 ms** | 1.66 MB of path strings |
| `stat` every file | **763 ms** | the mtime sort pass |
| walk + `readFileSync(utf8)` + regex | **1234 ms** | 66.8 MB read |
| walk + `readFileSync` buffer + `includes` | **1296 ms** | same bytes, byte search |
| walk + read, retaining all 1.2M matches | 1315 ms, **+154.6 MB RSS** | the unbounded case |
| walk + read, retaining 2000 entries | 1153 ms, +4.1 MB RSS | the bounded case |

**Reading the numbers.**

ripgrep is 2.2x faster than the Node walk that does the same work (554 vs 1234), and it is a compiled binary with a memory map and thread pool, so a Rust walk closing over a VFS would be closer. Say Rust gets within 1.5x: **about 800 ms for 20,000 files versus ripgrep's 550.**

The walk is cheap and the reads are not: 94 ms of 1234 ms is walking. That inverts the usual intuition and means the Pack's `glob` step is not the problem. **The reads are the problem**, and they are the same reads either way.

The stat pass is the most expensive optional step in the tool: 763 ms, 62% of the walk-plus-read time on its own, for a sort that produces an order most models cannot use. If you drop the mtime sort you save more than you would save by dropping the read.

Retention dominates memory. All matches cost 154.6 MB for a search that, in the end, renders 2000. That is the argument for streaming plus a cap, and it applies to both options equally.

**On a 200,000 file tree.** Scaling linearly, which is roughly right for a walk with no cache effects:

| | ripgrep | Rust walk over VFS |
|---|---|---|
| walk | ~1 s | ~1 s |
| read 668 MB | ~2.5 s | ~5 s |
| match | included above | ~3 s |
| stat for sort | ~7.6 s (times out at 2 s and gives up) | same |
| **total** | **~5 s** | **~11 s** |

The stat budget is the interesting number: 200,000 stats at the measured rate is 7.6 seconds, which blows the 2000 ms budget six times over, so Cursor's mtime sort **does not complete** on a tree that size and the order silently degrades. The newest-first guarantee therefore does not hold on large trees.

**The worst case a hostile repository can force.**

Five separate attacks, and the current design stops four of them.

1. **A huge file.** No per-file size limit in the Grep path. A 10 GB file with a rare pattern makes ripgrep read all 10 GB. Mitigated by the 8 MiB stdout cap only if it produces output; a 10 GB file with no matches produces no output and is read in full. **Mitigated by the 25 s deadline, and only on the streaming path.** On the non-streaming path there is a deadline too (each root gets a fresh 25 s), so this one is covered. But `--follow` plus a symlink to `/` can make the tree unbounded, and 25 s of a spinning disk is a substantial cost.

2. **A catastrophic regex.** Rust's `regex` crate is linear-time by construction and cannot backtrack, so `(a+)+b` is not a denial of service. **ripgrep uses Rust `regex`, so Cursor is safe here by inheritance.** But `--multiline` with `-U` does not change that. The one pattern class that can still hurt is `\p{...}` with a huge Unicode class on a huge input, which is linear but with a large constant.

3. **A deep tree.** `--follow` plus a symlink loop produces `File system loop found` and ripgrep stops that branch. A deep tree (100,000 nested directories) is bounded by the walk. No stack risk in ripgrep.

4. **A tree with 200,000 files.** Covered above: 5 to 11 seconds and the mtime sort gives up. The 2000-entry cap means the *render* is fine.

5. **A repository that makes the tool read something it should not.** `--follow` follows a symlink out of the workspace and reads the target. `filterBlockedFiles` removes the *results*, but the read already happened. So a symlink to `~/.ssh` gets read, its matches are dropped before rendering, and the read is visible to anyone watching file access times. **This one is not stopped.** The permission check is post-hoc.

**Content is streamed at the child-process level and buffered at the parent.** `executeRg` accumulates chunks in an array and only concatenates at the end (`Buffer.concat(f)`), so the whole output is in memory before parsing starts. Truncation happens during accumulation (the line budget and byte cap kill the child), so what is buffered is bounded by 8 MiB. Then the parser builds a second copy in `byFile`, capped at 2000 entries.

So the memory profile is bounded and small: up to 8 MiB of raw output, plus a `Map` of up to 2000 entries, plus the renderer's array of lines, plus the 15000-character result. Cursor is fine here; the unbounded case is what a naive reimplementation does.

**Time profile for a small tree.** A single-file search is dominated by process spawn. My probes measured roughly 30 to 60 ms per invocation on Windows, mostly CreateProcess. On a 10-file tree the search is faster than the spawn. If your VFS search is in-process, you win by an order of magnitude on small trees, which is the case a model hits most often.

**Other harnesses.** Rig collects the whole path list before it reads anything and then reads 100 whole files per batch: 100 files of 16 MiB reached 1,626 MiB resident in 1.47 s, and a no-match search over 1,000 files of 200 lines took 155.5 and 170.5 ms per call against ripgrep's 48.8 ms on the same warm Windows machine (probe). Rig's in-process `regex` 1.12.3 is linear: `^(a|aa)+$` took 246 us, 966 us and 3.85 ms on 100,000, 400,000 and 1,600,000 characters, and a pattern whose program passes 10,485,760 bytes is refused at compile time (probe); that confirms the linear-time claim above from a second engine. Zed reads each candidate file whole into a `String` in the prefilter, again as a rope, and again for the scan, with no output cap and no timeout (source). Pi runs ripgrep with no `--max-columns`, so one 5 MiB line becomes 5,243,675 characters of stdout and 18 to 20 MB of extra resident memory (probe). Cline's JavaScript fallback runs a backtracking `RegExp` on the main thread under a timeout that cannot cancel it, and zeroclaw's `rg` child keeps running after its 30 s timeout (source). Zed's `fancy_regex` limit of 1,000,000 backtracking attempts bounds the cost and drops the file's later matches silently.

### Q8. Paths

**`path` is accepted relative, absolute, `~`-prefixed, or as a `file://` URL.** `Te.o1` expands all four. Then:

```
searchTarget(root, args)
  IF NOT args.path  RETURN "."
  target = resolvePath(args.path, singleWorkspacePath ?? root)
  IF isPathInsideWorkspaceRoot(target, resolve(root))
      RETURN computeRelativeTarget(args.path, root)
  RETURN target

computeRelativeTarget(p, root)
  resolved = resolvePath(p, singleWorkspacePath ?? root)
  RETURN resolved === root ? "." : relative(root, resolved)
```

So a path inside the workspace becomes **relative to the root** and a path outside stays absolute. That relative string is what reaches ripgrep as the positional argument, with `cwd` set to the root.

**Consequence: the paths in the output are relative to the workspace root.** On Windows, ripgrep emits `.\src\a.ts` and Grep does not strip the `.\`. Glob strips a leading `./` when the display path is `.`, but Grep has no such step. So on Windows the model sees `.\src\a.ts`, and on POSIX it sees `./src/a.ts`. A model that then calls `Read` with `./src/a.ts` is fine; one that treats the string as a clean relative path may not be.

I confirmed the shape directly: raw ripgrep with `cwd` and a positional `.` gives `.\\a.txt\u00003:...`, so the `.\` comes from ripgrep and Cursor does not add it.

**Multi-root.** With no `path`, every root is searched, and results are keyed per root in `workspaceResults`. Each root's result has paths relative to that root. Then `deduplicateCrossWorkspaceRoots` removes a file from the root that does not own it:

```
dedupe(results, roots)
  IF roots.length <= 1  RETURN results
  resolvedRoots = distinct resolve(root)
  FOR each (root, union) of results
      belongsToCurrent(file) = deepestCoveringRoot(file, resolvedRoots) in {undefined, resolve(root)}
      // content: keep entries whose file belongs here, subtract the rest from totals
      // files:  keep paths that belong here, subtract the count
      // count:  keep rows that belong here, subtract both the file count and the match sum
```

So an overlapping nested-root workspace does not double-count. The counts are decremented by what was removed, which keeps the trailer accurate.

**`..` handling.** `resolvePath` normalizes, and `isPathInsideWorkspaceRoot` tests the result. So `path: "../../etc"` resolves to something outside every root, which routes to the outside-target branch, which is allowed (Grep will search outside the workspace, as long as the path survives `checkOutsideTarget`). This is different from the write and delete tools, which refuse to leave the workspace.

There is a worktree guard before all of it:

```
IF worktreeGuard({targetPath, workspacePaths, mainWorktreePath})
   RETURN error "You cannot search other worktrees for this repository, stay within your workspace paths."
```

which fires only when every workspace path contains `.cursor/worktrees`, the target is outside them all, and the target is inside a sibling worktree or the main worktree, a narrow condition.

**Symlinks.** `--follow` is always on, so ripgrep follows them. `realpath` runs on each root and the outside-target path. A symlink loop produces the tolerated `File system loop found` line.

**Windows specifics.**

- Drive letters: the `count` parser's `lastIndexOf(":")` handles `C:\...:3` correctly. The content parser never needs to, because of the NUL.
- UNC paths: `\\server\share\...`. `resolve` handles them, `relative` works, and ripgrep emits them with backslashes. The NUL keeps the content parse safe. No special casing in the Grep code, so behavior depends on Node's path module and ripgrep, both of which handle UNC.
- **Case sensitivity of the path itself.** The search is case-sensitive for pattern matching (`--case-sensitive` by default) but the *filesystem* may be case-insensitive, which ripgrep inherits. There is no path-comparison case folding in Grep. So `path: "SRC"` and `path: "src"` are the same directory on Windows and different on Linux, and the tool does nothing about that.
- `isRootIndependentSearchPath` treats `/abs/path`, `~/path`, and `file://...` as root-independent. A Windows drive-letter path like `C:\src` is absolute, so `path.isAbsolute` catches it.
- **A pattern beginning with `-`.** `--` is always emitted before the target, so the positional is safe. But the *pattern* goes through `--regexp <pattern>`, so a pattern of `--foo` is `--regexp --foo`, which ripgrep reads as the pattern, so it is safe by construction.

**The cache key is built from the raw path.** The cache is keyed on the **raw** `pattern`, `path`, `glob`, `type`, and the rest, with no resolved form. So `{path: "src"}` and `{path: "./src"}` and `{path: "/abs/root/src"}` are three different cache entries even though they search the same files. That is a small cache-efficiency loss and a correctness gain (no accidental sharing between different targets, though there are none here).

The per-file lock manager plays no part in Grep.

**Other harnesses.** Zed has no path parameter: scope is every visible worktree, and `include_pattern` is matched against the full path starting with the root name, so in a multi-root project the root name is part of the glob; `..` cannot escape the project (source, tests `grep_tool.rs:1196-1215`, `:1355-1373`). Zeroclaw refuses `..` by substring and relativizes output by stripping the workspace string with no component check, so `/wsx/a.rs:1:z` under `/ws` prints `x/a.rs:1:z` (probe). Rig keeps `..` in the shown prefix, since the walk root is the typed path and not the canonical one (probe). OpenCode searches a file's parent directory when `path` is a file, because the legacy tool does not pass `file` (source). Pi prints `/` always and the port prints OS separators, and rig on Windows prints `src\a.rs:1:` where its tests expect `/` (probe).

### Q9. Concurrency

**Grep takes no lock.** It never calls `waitForLock`, never touches `lockedFiles`, and has no read-modify-write window because it writes nothing. The only shared mutable state is the 5-entry result cache, which lives in the tool instance.

**What the cache races on.** The cache is a per-factory `new VEe` closed over the `_Ie` call, so it lives as long as the tool object, which is per-session. Two concurrent Grep calls with the same search key both miss, both run, and both `set`, so the second one overwrites the first. Since they ran the same search they should have the same result, so the race is benign in the common case and mildly wasteful.

The non-benign case: call A runs at T0 and sees the old tree. Call B runs at T1, after a write, and sees the new tree. If A finishes last, the cache holds A's stale result, and every subsequent call with that key sees the pre-write tree until the entry is evicted. **That is a stale-read window with no bound**, and it is the strongest argument for the generation-counter fix in D10.

**A search concurrent with a write is unprotected.** Grep does not participate in the file lock at all, so:

- A parallel `Write` and `Grep` in the same turn can interleave arbitrarily. The `Write` takes the per-file lock; `Grep` does not take it, so the lock does not serialize them.
- A file being truncated while ripgrep reads it can produce a partial read. ripgrep will search whatever bytes it got. No error, just fewer results, or a match on a half-written line.
- Cursor's `Shell` takes an exclusive lock. Since `Grep` takes no lock, a Shell command and a Grep call can run simultaneously, and the Shell can modify the tree mid-search.

So the answer to "what is locked, for how long, keyed on what" is **nothing, for zero time, on nothing**. The race it leaves open is a torn read against a concurrent write, and Cursor accepts that. For Runvil, this is a decision: if your VFS search takes a subtree claim (D1 option 2), then a concurrent write under that subtree **conflicts** rather than tearing, which is strictly better and is another argument for the capability.

**Per-root parallelism.** On the streaming path, `execute` iterates roots **sequentially** in a `for` loop. So a two-root search is two ripgrep processes, one after the other, sharing one 25 s deadline. There is no parallelism across roots in the tool. ripgrep itself parallelizes within a root (`--threads` defaults to the core count), so the per-root work is parallel and the cross-root work is not.

**Within the 2000 ms stat budget, the stats are parallel.** `Promise.all` over every file's `stat`, with one shared timeout. So 200,000 files means 200,000 concurrent `stat` calls racing a 2 s clock, which is a lot of open file handles. On Windows this is where I would expect the timeout to fire first.

**There are two TOCTOU checks**, both post-hoc and both about results rather than reads:

1. `filterBlockedFiles` re-realpaths each result path and re-checks the ignore list, catching a file that was cursorignored between the search and the render. It does not catch a file that was deleted.
2. The deadline check `if(deadline.signal.aborted)` between roots, so a root is skipped entirely if the budget expired during the previous root.

Neither closes the read-then-render window. A file deleted after ripgrep read it still renders.

**Other harnesses.** Rig declares one footprint, `Reads(Paths([root]))`, for the whole call, so its kernel holds a search behind an earlier write of an overlapping path in the same reply and runs it beside other reads (source, `search.rs:206-211`, `footprint.rs:62-71`); that is the one-claim model of D1 candidate 2, in a different layer. Zed takes no lock and works on immutable buffer snapshots, then takes a second snapshot after waiting for the parse, so an edit between the two shows in the snippet. Pi and Cursor's `pi_grep` read a matched file again after ripgrep exits, with no check that it is unchanged. (source)

### Q10. Fidelity

**Grep writes nothing, so most of this is trivially yes.** The one thing it changes is the working set: it opens files, and on Windows an open file cannot be deleted. That is not a fidelity question but it is the only side effect.

The fidelity question that does apply is whether the text the model sees equals the bytes in the file. It does not, in eight ways, and the CR-only row loses line structure rather than bytes:

| Transformation | Where | What is lost |
|---|---|---|
| CRLF → LF | ripgrep | the `\r` before each `\n` |
| CR preserved as a literal | `xv` | nothing, but a CR-only file becomes one line |
| UTF-8 BOM consumed | ripgrep | the leading `EF BB BF` |
| invalid bytes → U+FFFD | `Buffer.toString("utf8")` | the original byte values |
| lines over 1000 bytes cut | ripgrep `--max-columns` | everything after the 1000th byte, replaced by ` [... omitted end of long line]` |
| NUL bytes → binary verdict | ripgrep | the whole file's matches |
| matches beyond 2000 entries | parser | everything past the cap |
| output beyond 15000 UTF-16 units | renderer | whole lines past the budget |

**BOM.** Consumed by ripgrep, confirmed: a file with a UTF-8 BOM returns content without it, and a pattern matching the BOM finds nothing. So a model that greps for a BOM-anchored pattern gets nothing.

**Trailing newline.** The renderer's `.trim()` removes the trailing blank line from the assembled body, so the model never sees a trailing newline after the final match. The file's own trailing newline is inside the last line's content only if ripgrep included it, which it does not (it strips the record terminator).

**Encoding round-trip.** If a caller reads a file through `Read` and greps it, the two tools see different text for a non-UTF-8 file. `Read` may repair the encoding; `Grep` may not. There is no guarantee the two agree, and no way for the model to know which encoding each settled on.

**File mode, ownership, timestamps.** Untouched. Grep does not write.

The practical rule for a reimplementation: **Grep is a lossy view, and the loss is silent.** Everything the model learns about a file from a Grep result should be re-verified with `Read` before it is used to construct an edit, which is what the tool description tells the model to do.

**Other harnesses.** The stable orderings are few. Zed's is worktree id, then path component order, then match offset, through ordered one-shot receivers that hide thread timing (source, `project_search.rs:532-554`, `:636-662`). Zeroclaw's internal backend sorts by path, while its `rg` backend gave 12 different orders in 12 runs and `--sort path` gave one (probe). Without a sort flag, pi gave 8 outputs in 8 runs on 20,000 files in both its original and the port, OpenCode 4 in 4 on 400 files, and Cline 6 in 6 on 60 files (probe); Gemini CLI passes no `--sort` with 4 threads and was not measured. Rig's order is the file system's: stable on NTFS across two calls, sorted on `Remote`, so `Local` and `Remote` disagree, and its own test sorts before it asserts (source, `search/tests.rs:140-141`). Codex's removed tool asked ripgrep for `--sortr=modified`. With a hit cap and no order, which hits survive is arbitrary.

### Q11. Determinism

**Output order is defined but not guaranteed.** Two mechanisms, and the choice between them decides whether two identical runs produce identical bytes.

```
sortsAfterRipgrep(root, args, streaming)
  IF args.sort !== undefined  RETURN false
  RETURN streaming ? args.sortAscending !== true : isAbsolute(searchTarget(root, args))
```

With no `sort` field reachable from any schema, `args.sort` is always undefined, so:

| Condition | ripgrep sort flag | Client resort | Deterministic? |
|---|---|---|---|
| streaming on (default) | none, plus `--line-buffered` | yes, by `mtimeNs` | **only if the stat pass completes** |
| streaming off, relative target | `--sortr modified` | no | yes, ripgrep sorts |
| streaming off, absolute target | none, plus `--line-buffered` | yes, by `mtimeNs` | same caveat |

`streamingEnabled` returns `true` when `useStreamingGrep` is unset, so **the default path is the client-sorted one.**

The sort itself, from the daemon's `newest-first.js`:

```
orderByModifiedDesc(files, mtimes, sep)
  keyed = files.map((f, i) => ({ f, mtime: mtimes[i], segments: f.path.split(sep) }))
  keyed.sort((a, b) => {
     if (a.mtime !== b.mtime) {
        if (a.mtime === undefined) return 1        // unknown mtime last
        if (b.mtime === undefined) return -1
        return a.mtime > b.mtime ? -1 : 1          // newest first
     }
     return comparePathSegments(a.segments, b.segments)
  })
  return keyed.map(x => x.f)

comparePathSegments(left, right)
  FOR i IN 0..min(len)
     IF left[i] !== right[i]  RETURN left[i] < right[i] ? -1 : 1
  RETURN left.length - right.length
```

Four properties:

1. **Descending by `mtimeNs`.** Nanosecond resolution, from `stat(path, {bigint: true}).mtimeNs`.
2. **Undefined mtime sorts last.** A file whose stat failed goes to the end.
3. **Ties break on path segments, ascending**, compared lexicographically segment by segment with shorter-first on a prefix. So `src/a.ts` sorts before `src/b.ts`, and both sort before `src/sub/a.ts`. Take `src/a.ts` against `src/sub/a.ts`: segment 0 is `src` in both, equal; segment 1 is `a.ts` against `sub`, and `a.ts` < `sub`, so `src/a.ts` goes first. The comparison is lexicographic on segments rather than on the joined string, which is why `a/b` sorts before `a-b` in a way string comparison would not.
4. **A missing mtime has no tiebreak partner**, so two undefined-mtime files fall through to `comparePathSegments`, which is deterministic.

**Within a file, lines are in ripgrep's emission order**, which is ascending line number because ripgrep emits in scan order, and never interleaved across files. So the intra-file order is deterministic even when the inter-file order is not.

**Where determinism breaks.** The stat pass is:

```
sortNewestFirst(items, absolutePath, statMtime)
  keyed = items.map(item => ({ item, path: absolutePath(item) }))
  TRY  mtimes = await withTimeout(Promise.all(keyed.map(k => statMtime(k.path))), 2000, "stat timed out")
  CATCH TimeoutError  RETURN { items: [...items], statTimedOut: true }     // ORIGINAL ORDER
  RETURN { items: orderByModifiedDesc(keyed, mtimes, sep).map(x => x.item), statTimedOut: false }
```

On timeout the **original ripgrep order** is returned untouched. And ripgrep's order with no sort flag is its parallel walk order, which depends on thread scheduling and the filesystem's readdir order. So:

- Two identical runs on a small tree: identical bytes.
- Two identical runs on a tree big enough to blow the 2000 ms stat budget: **possibly different bytes**, because the fallback order is scheduling-dependent.

I measured the stat cost at 763 ms for 20,000 files, so a tree of roughly 50,000 files blows the budget. Above that size, Grep's output is not reproducible.

**There is a second, subtler nondeterminism.** `Promise.all` over 200,000 stats opens a huge number of handles; the ones that fail (`statMtime` returns `undefined` on any error via `catch{return undefined}`) all get `undefined`, which sorts last in their original relative order. So an over-subscribed tree gets a partially sorted result: known-mtime files sorted by mtime, unknown-mtime files at the end in walk order. That is deterministic only if walk order is.

**For a reimplementation**, the recommendation from D8 applies: sort by path and get full determinism for free. If you keep mtime sorting, make the timeout behavior *sort by path* rather than "leave in walk order", so the failure mode is still deterministic and still documented.

### Q12. Truncation and omission

This tool has the most elaborate truncation machinery in the set: seven independent cuts, three of which can fire in one call, and one trailer that has to describe all of them coherently.

**The seven cuts, in the order they fire.**

| # | Cut | Trigger | Signal |
|---|---|---|---|
| 1 | ripgrep line budget | child emits 2000 (4000 with context) lines | `ripgrepTruncated` via `totalLines >= hardCutoff` |
| 2 | stdout byte cap | child emits 8 MiB | `stdoutByteCapped`, ORed into `ripgrepTruncated` |
| 3 | deadline | 25 s elapsed | `timedOut`, ORed into `ripgrepTruncated` |
| 4 | client entry cap | 2000 entries parsed | `clientTruncated` |
| 5 | user `head_limit` | N matched entries emitted | `headLimitApplied` plus `clientTruncated` when more remain |
| 6 | user `offset` | N matched entries skipped | `offsetApplied` |
| 7 | render char budget | 15000 UTF-16 units | `charTruncated` and `keptLines < lines.length` |

Cuts 1, 2, and 3 all collapse into one boolean, `ripgrepTruncated`, and the trailer cannot tell you which fired. That is an information loss: a model told `at least 4211` cannot tell whether the tree is bigger than the budget or the search was too slow.

**The long-line cut, which is separate from all of them.** `--max-columns 1000` makes ripgrep replace a long line's tail with ` [... omitted end of long line]`, with a leading space. The trailer does not report this truncation, and there is no count. A model that gets a preview ending in that marker knows one line was cut but not how much was lost. There is also no way to ask for the full line: re-running with a narrower pattern does not help, because the cut depends on line length rather than on the match.

This is arguably the worst truncation in the tool, because `--max-columns-preview` keeps the first 1000 characters, and if the match is at character 1500 the preview does not contain the match at all. I verified this: a 2000-character line with `NEEDLE` at position 1001 previews 1000 characters and the preview does **not** contain `NEEDLE`. The model gets a line with no match in it and no indication that the match is further along.

**Marker, count, and hint.** Cuts 4 through 7 have the trailer. Cuts 1 through 3 have `Results truncated. ` plus `at least`. The long-line cut has only the inline marker. The binary drop has **nothing**.

**The trailer, worked.**

A `head_limit` cut, 5 matches total, showing 2:

```
[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]
```

A page past the first:

```
[Showing 3-4 of at least 5 matching lines. Use offset=4 to see more, or narrow the search.]
```

A parse-cap cut with no user limit:

```
[Results truncated. Showing 2000 of 4211 matching lines. Narrow the search to see the rest.]
```

A render-budget cut:

```
[Results truncated. Showing 120 of 300 matching lines. Use offset=120 to see more, or narrow the search.]
```

Files mode:

```
[Showing 3 of at least 5 files. Use offset=3 to see more, or narrow the search.]
```

Multi-root files mode:

```
[Showing 3 of at least 5 files. Set path to one workspace to page through it, or narrow the search.]
```

**The tool misreports completeness in four places.**

1. **`at least` where the total is exact.** The parse-cap case with 4211 matches renders `4211 matching lines`, because the parser counts every match even past the cut. But the exact-2000 case renders `at least 2000`, because `total <= end` makes the lower-bound test fire on a count the parser knows to be complete. The rule is `clientTruncated && (!reachedEnd || total <= end)`, so it hedges when the count is complete. That is mildly misleading, and it makes the total useless for deciding whether to narrow.
2. **`clientTruncated` is `>=`.** A result of 2000 entries sets it true, so a search with 2000 matches claims truncation when nothing was cut. The rendered footer, from my port, is:

   ```text
   [Results truncated. Showing 2000 of at least 2000 matching lines. Narrow the search to see the rest.]
   ```

   Three of those words are false at once: nothing was truncated, `at least` is wrong for an exact count, and there is no rest to narrow to. **That false report is one the model can act on**: it will narrow a search that was already complete, which is a wasted turn plus a worse result.
3. **The count-mode totals.** `totalMatches` is the sum of parsed counts, and the counts are per-file match counts from `-c`, which for a binary file is 1 regardless of how many matches there were. So `totalMatches` undercounts binary files. Nobody sees it, because the trailer's unit in count mode is `file`.
4. **The binary drop.** Content mode silently omits binary files entirely. A model searching for a symbol in a binary-adjacent file gets `No matches found` with no hint that a file was skipped.

**What a reimplementation should do.** Keep the trailer shape, because a model trained on it will read it correctly. Fix the `>=`/`>` asymmetry to a consistent `>`. Add a binary-skip count. And make the three `ripgrepTruncated` causes distinguishable, since the model's next action differs: on a line budget it should narrow the search, and on a deadline it should narrow the *path*.

**Other harnesses.** Reporting a cut at an exact fit is common. Pi says `3 matches limit reached` at `limit: 3` with 3 matches; rig prints `[stopped at 200 matches]` at 199 plus one more or at 200; zeroclaw flags `max_results` equal to the match count; Gemini CLI's header test is `matchCount >= totalMaxMatches`; OpenCode prints `Found 100 matches (more matches available)` at 100 (probe, source). OpenCode's `dev` branch fixes it by fetching `limit + 1` and slicing (source, `D:plugin/grep.ts:110-145`). Several cuts have no marker at all: Zed's 5,001-file cut, rig's 300-character line clip, and Codex's removed `limit`. Cline's `Found 100 results` is a floor, and a later provider-side cut to 8,000 characters leaves a marker that names no way back. Gemini CLI returns a timeout, a missing binary and a ripgrep exit 2 as success text. (source, probe)

## Failure matrix

Every failure the tool can produce. `Result case` is the structured record case; `Layer` says which stage produced it. The model-visible text is what arrives after `serializeError` and `render`.

| Condition | Result case | Exact model-visible text | Layer |
|---|---|---|---|
| arguments do not match the schema | `error` | `Invalid arguments:\n${key}: ${message}` | validation, in `LG` before the body |
| arguments are not valid JSON | `error` | `Tool call arguments were not valid JSON (${parseError}). Re-issue the call with arguments as a single well-formed JSON object.` | validation |
| JSON contained literal `<arg_key>`/`<arg_value>` tags on a `zai` model | `error` | the same string plus ` Your arguments contain literal <arg_key>/<arg_value> tags; write nested objects as plain JSON, e.g. {"query": "x"}.` | validation |
| `pattern` is exactly `""` | `rejected` (`INVALID_ARGS`) | `pattern is empty. Pass "." to match every line.` | tool, step 3 |
| `-A`, `-B`, or `-C` is negative | `rejected` (`INVALID_ARGS`) | `-A is -1. Pass 0 or more context lines.` | tool, step 4 |
| `glob` trims to `**/*` or `**/**`, guard on | `rejected` (`INVALID_ARGS`) | `Glob pattern "**/*" matches every file and is not allowed. Use a more specific glob or no glob.` | tool, step 5 |
| tool execution is disabled for the run | `error` | `tool execution disabled for this run` | harness `executeToolCall` |
| target path is inside another worktree | `error` | `You cannot search other worktrees for this repository, stay within your workspace paths.` | executor, step 3 |
| `path` resolves to different folders across roots | `error` | `Path '${path}' points to a different folder from each workspace root; use an absolute path` | executor, step 5 |
| `realpath` failed on the target | `error` | `Path '${path}' could not be verified while resolving symlinks. This can happen when access is denied or the path contains a symlink loop.` | executor, step 5 |
| target is filtered by `.cursorignore` | `rejected` (`DG`) | `Path '${path}' is filtered out by .cursorignore` | executor, step 5 |
| target exists but is neither file nor directory | `notFile`-shaped (`DG`) | `Path '${path}' is neither a file nor a directory` | executor, step 5 |
| no root survived resolution | `error` | the `filtered out` or `could not be verified` text, depending on why | executor, step 5 |
| root does not exist | `error` | `Path does not exist: ${root}` | executor, step 7 |
| `rg` binary missing | `error` | `Path does not exist: ${cwd}` | `executeRg` spawn ENOENT, via `Fx(cwd)` |
| ripgrep path never configured | `error` | `Ripgrep path not configured. Call configureRipgrepPath() at startup.` | `xe.Ko()` |
| invalid regex | `error` (`DG`) | `rg: regex parse error: invalid regex "${pattern}": ${reason}.` | classifier `IEe`, stderr rewrite |
| invalid regex, reason matches `unclosed` / `unopened` / `repetition operator missing expression` | `error` | `... ${reason}. Escape special characters such as ( [ { with a backslash to match them literally.` | same |
| regex uses an unsupported feature | `error` | stderr verbatim, because `backreferences are not supported` matches none of the three hint reasons | classifier, `DG` passthrough |
| unrecognized `type` | `error` (`DG`) | `rg: unrecognized file type: ${type}` | classifier, passthrough |
| invalid `glob` syntax | `error` (`DG`) | `rg: error parsing glob '${glob}': unclosed character class; missing ']'` | classifier, passthrough |
| permission error on the target itself, empty stdout | `error` | stderr verbatim, e.g. `rg: /root: Permission denied (os error 13)` | `throwOnRipgrepFailure` |
| every root timed out and none produced stdout | `error` (`TIMEOUT`) | `Timed out after 25s` | executor, step 7 |
| indexed grep timed out | (never model-visible) | silent fallback to ripgrep | `executeIndexedGrepWithTimeout` |
| the call was cancelled | `error` | `Grep was cancelled` | `executeRg` abort listener |
| ripgrep exited non-zero, stderr empty | `error` | `ripgrep exited with code ${code}` | `throwOnRipgrepFailure` |
| executor returned an unexpected oneof case | `error` | `Unexpected result case: ${case}` | tool, step 10 |
| zero matches, offset 0 | `success` | `No matches found` | renderer, content and count |
| zero matches, offset 0, files mode | `success` | `No files with matches found` | renderer |
| zero matches, offset 0, ripgrep truncated | `success` | `No matches found before the search stopped. Narrow the search to see the rest.` | renderer |
| zero matches, offset 0, files mode, ripgrep truncated | `success` | `No files with matches found before the search stopped. Narrow the search to see the rest.` | renderer |
| offset past the end, more available | `success` | `No more results at offset=${offset}. Narrow the search to see the rest.` | renderer |
| offset past the end, exhausted | `success` | `No more results at offset=${offset}.` | renderer |
| success with no workspace entries | `success` | `No results found.` | renderer |
| binary file matched, content mode | `success` | nothing; the file is absent from the output | parser, silently |
| unreadable subdirectory, some results | `success` | results, with no notice that a directory was skipped | `throwOnRipgrepFailure`, tolerated |
| unreadable subdirectory, zero results | `success` | `No matches found` with no notice | same |
| file deleted between the search and the render | `success` | the stale result renders normally | no check exists |
| `--follow` reached a symlink out of the workspace | `success` | matching results removed by `filterBlockedFiles`, but the read already happened | post-filter, too late |

Four rows that look like successes and are not what the model thinks:

1. **`Path does not exist: ${cwd}` when the problem is a missing `rg` binary.** `Fx` is called on the cwd rather than on the binary. The diagnostic telemetry records `rgPath`, `rgPathExists`, `rgIsSymlink`, and the asar state, so Cursor knows the cause and the model never does. Reproducing this is bug-compatibility; fixing it costs one string.

2. **An unreadable directory reported as a complete search.** The tolerance regex exists so that scanning a tree with one unreadable folder does not fail the whole search, which is right. The cost is that the model cannot distinguish "no matches" from "could not look". A counter in the trailer would fix it.

3. **A stale result out of the cache.** No timeout, no generation check, no signal. The model reads a tree that no longer exists.

4. **A `clientTruncated` that says `Results truncated.` on a complete 2000-entry result**, because the comparison is `>=` not `>`. The model is told to narrow a search that already finished.

## Traps

The places a faithful-looking implementation goes wrong. Each one is a specific defect with a specific test that catches it.

### Trap 1: `head_limit` implemented as `--max-count`

**What goes wrong.** You map `head_limit` to `--max-count N` because that is what a "limit" sounds like in ripgrep. Now the limit is per file. Two files with three matches each and `head_limit: 5` returns six lines instead of five, and the counts in the trailer are wrong.

**Why it is easy to get wrong.** `--max-count` is the obvious flag. It is documented, it is fast, it stops the scan early, and on a single-file search it produces the right answer. Every test with one file passes.

**The test that catches it.** Two files, `a.ts` with two matches on lines 12 and 40 and `b.ts` with three. `head_limit: 2`. The correct output is:

```text
src/a.ts
  12:  return null;
  40:  return null.
[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]
```

`b.ts` must not appear. A `--max-count 2` implementation shows all five matches, because two is under the per-file limit for both files.

**Second test for the same trap.** `head_limit: 2` with three matches in `a.ts` and none anywhere else, where all three are on the same line via a multiline pattern. `--max-count` counts matching *lines*, and the tool counts matched *entries*, so the two diverge.

### Trap 2: `head_limit: 0` treated as zero

**What goes wrong.** `head_limit: 0` returns nothing. Cursor returns everything, because `fIe(r.head_limit, Math.ceil) || void 0` turns `0` into `undefined`, which skips the slice entirely.

**Why it is easy to get wrong.** Every other tool in the set treats `0` as zero. The `||` idiom is doing double duty as a "was it set" test, and it does not distinguish "not set" from "set to a falsy value".

**The test that catches it.** `head_limit: 0` with 40 matches. All 40 render, and the trailer has no `Showing` at all because no limit was applied.

### Trap 3: pagination counters scoped per file

**What goes wrong.** You apply `offset` inside the per-file loop, so `offset: 5` skips five matches *in each file*. A five-file search returns the 6th match onward from every file instead of the 6th match onward overall.

**Why it is easy to get wrong.** The natural place to write the skip is next to the entries you are skipping, which is inside the file loop. Hoisting the counter out is a one-line change that is easy to omit.

**The test that catches it.** Three files, three matches each, `offset: 4`, `head_limit: 2`. Correct: two matches from the second file (entries 5 and 6 counting from 1). Per-file: two matches from each file, six lines.

### Trap 4: splitting the wire format on `:`

**What goes wrong.** Every Windows path is truncated to its drive letter. `C:\src\a.ts\0 3:const x` becomes file `C`, line number unparseable, so the line is skipped. Every result on Windows disappears, and the tool reports `No matches found`.

**Why it is easy to get wrong.** It works on a Linux fixture with relative paths and no colons in filenames. The bug is invisible until a Windows path or a filename with a colon.

**The test that catches it.** A file named `weird:name.txt`, and any search on Windows. Correct output includes `weird:name.txt` with line 1. A colon-splitting parser gives file `weird`.

**Second test.** A file whose *content* contains a colon before the line number, which cannot happen on the wire, but a file whose path contains a newline can, and that is the case `-0` cannot save you from. This is not reachable in practice, and it marks the boundary of what `-0` protects.

### Trap 5: `-C` silently dropped when `-A` or `-B` is present

**What goes wrong.** `-C: 3, -A: 1` produces one line of after-context and nothing before, because the `-C` expansion is guarded by `contextBefore === undefined && contextAfter === undefined`. The model asked for symmetric context and got asymmetric, with no indication.

**Why it is easy to get wrong.** In the other direction: if you write the obvious "if `-C` is set, expand it", you get *both* `-C` and `-A` in the argv, and ripgrep's last-flag-wins behavior gives you whichever came later in the string, a third behavior that differs from both Cursor and the fix.

**The test that catches it.** `-C: 3, -A: 1` on a file with matches on lines 5 and 20 of a 30-line file. Cursor's output has line 6 after each match and no line 4 or 19 before it. A `-C`-wins implementation has three lines on each side. A last-flag-wins implementation depends on argv order.

### Trap 6: the gap separator emitted when there is no context

**What goes wrong.** You emit `  --` between non-adjacent matches always. Cursor emits it only when the result contains at least one context line anywhere.

**Why it is easy to get wrong.** `--` between distant line numbers is what ripgrep does and what `less` does, so it feels correct. The condition `hasContext` is computed over the *entire result*, not per file, which is the surprising part: a match in file A with context turns on the separator for file B even if file B has none.

**The test that catches it.** Two matches on lines 3 and 9 with no context flag: no separator. Then the same file with `-A: 1` and matches on lines 1 and 2: `hasContext` becomes true, but the separator check is `lineNumber - previousLineNumber > 1` and every gap is 1, so there is still no separator anywhere. I confirmed the underlying ripgrep output on a dense fixture: `m[13]` with `--after-context 3` on a five-line file emitted lines 1 through 5 with no `--` line at all, because ripgrep merges overlapping context groups.

`hasContext` is a property of the **whole result**, computed before any file is rendered, so one file with context turns the separator logic on for every file in the result, including files whose own matches are adjacent.

### Trap 7: `clientTruncated` as `>` instead of `>=`

**What goes wrong.** A result of 2000 entries sets `clientTruncated: true` in Cursor, because the parser's test is `linesProcessed >= clientLimitLines`. If you use `>`, a search with 2000 matches reports no truncation and no trailer, and a byte-exact test against Cursor fails.

**Why it is easy to get wrong.** `>` is the correct comparison for "did I drop anything". `>=` is the correct comparison for "did I hit the cap". Cursor chose the second, which is conservative and slightly misleading.

**The test that catches it.** A search with 2000 matches. Cursor renders `[Results truncated. Showing 2000 of at least 2000 matching lines. Narrow the search to see the rest.]`. If you find that output absurd and want to fix it, change to `>` and accept the divergence, which is a deliberate choice.

**Other harnesses.** Six harnesses fire their cut notice on an exact fit: Cursor, pi, rig, zeroclaw, Gemini CLI and OpenCode (Q12). The test that catches it in a port is the same exact-fit case. OpenCode's `dev` branch avoids the false notice by fetching `limit + 1` and slicing, which is the mechanical fix if you decide to use `>` (source, `D:plugin/grep.ts:110-145`).

### Trap 8: `at least` omitted from the trailer

**What goes wrong.** The parent spec's worked example omits `at least`, so a faithful copy of that example produces `[Showing 2 of 5 matching lines. ...]`. Cursor's string is `[Showing 2 of at least 5 matching lines. ...]`.

**Why it is easy to get wrong.** The spec marks the line `(derived)`, so a reader assumes the derivation is right. The condition is subtle: `isLowerBound = ripgrepTruncated || (clientTruncated && (!reachedEnd || total <= end))`, and for a `head_limit` cut `clientTruncated` is true and `reachedEnd` is false, so the second clause fires.

**The test that catches it.** `head_limit: 2` with 5 total matches. Compare the exact footer bytes.

The mirror-image mistake needs a guard too: `at least` must **not** appear when the page was full and nothing was left. `head_limit: 2` with 2 total matches has no footer at all, and an implementation that always hedges prints one where Cursor prints nothing. Test both the cut and the exact-fit case, or you will fix one and break the other.

### Trap 8b: `Results truncated. ` expected on a `head_limit` cut

**What goes wrong.** You assume a truncated result says `Results truncated. `. For a plain `head_limit` cut it does not, because `reachedEnd` is false while the page is full, so `cutBeforeEnd` is false, so the prefix is absent. The only signal that anything was cut is the words `at least`.

**Why it is easy to get wrong.** `Results truncated. ` is the obvious marker and it does appear for the parse-cap and render-budget cuts, so a test that only covers those two cases passes while the `head_limit` case is wrong.

**The test that catches it.** `head_limit: 2` of 5 must render exactly `[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]` with no prefix, and the 2000-parse-cap case must render with one.

### Trap 9: the trailing blank line

**What goes wrong.** Your renderer emits a blank line after the last file's group, so the output ends `...const first = 0\n\n</workspace_result>`. Cursor's ends `...const first = 0\n</workspace_result>`, because the assembled body is `.trim()`ed.

**Why it is easy to get wrong.** The per-file blank line is pushed unconditionally in the loop, so the last file gets one too, and it is only the final `trim()` that removes it. If you build the string differently, for example by joining file blocks with `\n\n`, you get the right-looking inter-file spacing and the wrong ending.

**The test that catches it.** Compare the last 30 bytes of a two-file content result. `b.ts` content, then a single `\n`, then `</workspace_result>`.

### Trap 10: `--max-columns` assumed to be characters

**What goes wrong.** You cut at 1000 characters instead of 1000 bytes. A CJK line of 334 characters is 1002 bytes and should be cut; a character-based implementation leaves it whole. A 600-character emoji line is 2400 bytes and should be cut hard; a character-based implementation previews 600 characters.

**Why it is easy to get wrong.** `--max-columns` sounds like columns, which sounds like characters. The `rg --help` text says "omit lines longer than this limit in bytes", which is the only place the unit is stated.

**The test that catches it.** Four fixtures, one per character width, at the byte boundary. My measurements:

| Fixture | Bytes | Chars | Cursor |
|---|---|---|---|
| 999 `x` | 999 | 999 | whole |
| 1000 `x` | 1000 | 1000 | cut |
| 499 `é` | 998 | 499 | whole |
| 500 `é` | 1000 | 500 | cut |
| 332 `一` | 996 | 332 | whole |
| 334 `一` | 1002 | 334 | cut |
| 249 emoji | 996 | 249 | whole |
| 250 emoji | 1000 | 250 | cut |

If your implementation cuts the 499-`é` line, you are counting characters.

### Trap 11: the preview assumed to contain the match

**What goes wrong.** You assume `--max-columns-preview` shows the match. It shows the first 1000 characters of the line. A match at character 1500 is not in the preview at all.

**Why it is easy to get wrong.** On ordinary code the match is near the start of the line, so the preview contains it. On minified JavaScript, a generated lockfile, or a long data line, it is not.

**The test that catches it.** A 2000-character line with `NEEDLE` at position 1001. The output line is 1000 characters of padding plus ` [... omitted end of long line]`, with no `NEEDLE`. I verified this: `ascii2000` previews 1002 characters, and `previewEndsWithNEEDLE` was `false`.

The model gets a result line for a pattern it cannot see in the result. If you want to fix this, the fix is to compute the match offset yourself and window the preview around it, which is what Copilot does and what Cursor does not.

### Trap 12: `-i` implemented by omitting `--case-sensitive`

**What goes wrong.** You follow the parent spec's claim that `-i` works "by absence of `--case-sensitive`, because ripgrep defaults to case-insensitive when the pattern has no uppercase". Then a `-i` search for `alpha` behaves the same as a case-sensitive search, because ripgrep's default is case-sensitive, not smart-case.

**Why it is easy to get wrong.** The spec states it as fact and adds a plausible-sounding parenthetical. Both are wrong. ripgrep's default is `--case-sensitive`; smart-case is opt-in via `S`/`--smart-case` or `--ignore-case` only when the pattern is all-lowercase, and it is not the default.

**The test that catches it.** A file with `const Alpha = 1`, `const alpha = 2`, `ALPHA = 3`. A search for `alpha` with `-i` must return all three. A search for `alpha` without `-i` must return only line 2. I ran both against the shipped binary and confirmed.

The correct implementation is `--ignore-case` when `-i` is true and `--case-sensitive` otherwise, with one of the two always present.

### Trap 13: assuming `--no-ignore` is involved

**What goes wrong.** You add a `no_ignore` parameter or emit `--no-ignore` for a broad glob, because the description warns that broad globs "bypass .gitignore rules". The bypass is real, but it comes from `--iglob` overriding ignore rules, not from any ignore flag.

**Why it is easy to get wrong.** The description's phrasing makes it sound like a flag. And there *is* a `--no-ignore-vcs` in the bundle, but it belongs to the ripwalk listing client.

**The test that catches it.** A tree with a `.gitignore` containing `ignored/` and `*.log`, plus files in both. A search with no glob returns 3 of 5 files. A search with `--iglob '**/*'` returns all 5. A search with `--no-ignore` returns all 5 too, but for a different reason, and adding `--no-ignore` to the Grep argv is a divergence.

### Trap 14: the mtime sort assumed to be ripgrep's job

**What goes wrong.** You pass `--sortr modified` and assume you are done. On the default (streaming) path Cursor does not pass any sort flag, adds `--line-buffered`, and sorts client-side. Both produce newest-first, so the *order* looks right, but the failure modes differ: the client-side path has a 2000 ms budget that silently degrades to walk order.

**Why it is easy to get wrong.** `--sortr modified` is the obvious implementation and it is what Cursor does on the non-streaming relative-target path. Copying it looks like copying Cursor.

**The test that catches it.** A tree big enough that the stat pass exceeds 2000 ms (roughly 50,000 files at my measured 763 ms per 20,000). Cursor's output order becomes nondeterministic. A `--sortr modified` implementation stays deterministic. If you compare bytes against Cursor on a big tree, you will see a difference that is Cursor's fault.

### Trap 15: the 8 MiB cap counted as characters

**What goes wrong.** You count `string.length` instead of `Buffer.length`. A result full of CJK matches uses three bytes per character, so a character-based cap lets 24 MiB of stdout through, and the process dies or the agent runs out of memory.

**Why it is easy to get wrong.** Everything else about output is character-counted, so bytes feels wrong. But `y += e.length` where `e` is a Buffer is bytes, and the comparison is against `8388608`.

**The test that catches it.** A file of CJK matches whose stdout exceeds 8 MiB. Cursor kills the child and sets `ripgrepTruncated`. A character-counted implementation keeps going.

### Trap 16: the line number parsed as padded

**What goes wrong.** You pad the line number to six characters or add a pipe, because that is what `Read` does. Grep's format is two spaces, then the bare number, then `:` or `-`.

**Why it is easy to get wrong.** The six-wide pipe format is the dominant line-number format in this toolset and it is what the shared formatter produces. Grep does not use the shared formatter at all; it builds its own strings in `formatContentResult`.

**The test that catches it.** Any content result. `  3:const alpha = 1`, not `     3|const alpha = 1` and not `     3:const alpha = 1`.

### Trap 17: assuming a claim or a lock exists

**What goes wrong.** You assume Grep participates in the per-file lock manager because every other file tool does. It does not. There is no lock call anywhere in the Grep path, so a Grep and a Write can interleave.

**Why it is easy to get wrong.** The lock is described in the shared infrastructure section and every sibling tool uses it. Grep is the exception, and the reason is that it writes nothing.

**The test that catches it.** Run a `Write` and a `Grep` on the same file in one turn and check whether the Grep sees the pre-write or post-write bytes. Both outcomes are legal in Cursor. If you want a defined answer, you must decide on one yourself.

### Trap 18: the pattern validated in one dialect and run in another

**What goes wrong.** Gemini CLI checks the pattern with JavaScript `new RegExp` and runs it with ripgrep, which is Rust. `(?i)foo` is refused before ripgrep sees it, and `(?=x)y` passes the check, fails inside ripgrep with exit 2, and comes back as success text. Zed's parameter text says the pattern is parsed by the Rust `regex` crate while the engine is `fancy_regex`, so a model told "Rust regex" that sends a look-behind gets it accepted. `(source, probe)`

**Why it is easy to get wrong.** An early validation step feels careful, and the regex engine nearest to hand is the host language's.

**The test that catches it.** Send `(?=x)y`, `(?i)foo`, `(a)\1` and `foo(`. Each must give an error that names the unsupported feature or the parse failure, as an error and not as success text, and the description must name the dialect the engine runs (D11).

### Trap 19: one bad file, line or record fails or erases the call

**What goes wrong.** Pi and zeroclaw let one unreadable file make ripgrep exit 2 and return only the error text, though a readable file matched. OpenCode fails the whole call when one matching line is over 64 KiB or is not valid UTF-8. Rig skips a whole file for one invalid byte, so the ASCII lines of a Latin-1 file vanish. Cursor avoids the first only while stdout is non-empty: a search that found nothing and could not read three directories says `No matches found`. `(probe, source)`

**Why it is easy to get wrong.** The simple rule is "a non-zero exit is a failure" or "a record that does not parse is an error", and a hostile or merely old tree makes it fire.

**The test that catches it.** A tree with one unreadable file, one Latin-1 file whose ASCII line matches, one 70,000-character matching line (a minified file), and one clean match. The result must hold the clean match, the ASCII line and the long line clipped, and it must say how many files were skipped and why.

### Trap 20: paging that skips one unit and counts another

**What goes wrong.** Zed skips `offset` raw regex ranges and counts a page in merged context blocks, so page 2 starts inside page 1. With 25 blocks made of 50 raw ranges, page 2 at `offset: 20` shows blocks 11 to 25 and calls them `Found 15 matches`. `(source, hand trace)`

**Why it is easy to get wrong.** Skipping is easiest in the loop that sees raw ranges, and merging happens later in the same loop. Cursor avoids it because its skip and its count both use matched entries, and context lines are attached afterward.

**The test that catches it.** Matches on row pairs `(10k, 10k+1)` for 25 values of `k`, `context_lines: 2`. Page 1 and page 2 joined must equal one unpaged result, with no block shown twice and the second header's range matching what it shows.

### Trap 21: context built from a second read of the file

**What goes wrong.** Pi re-reads each matched file with a second decoder and a hand-written line splitter. A lone CR shifts every later line number (pi fixed it on 2026-10-27; Cursor's `pi_grep` still has it), a UTF-16 file prints as `��n\u0000e\u0000...` including the match line, a UTF-8 BOM shows on line 1, and overlapping blocks print the same line twice. `(probe)`

**Why it is easy to get wrong.** The search returns a line number and no neighbors, and reading the file again looks cheaper than asking the search for context.

**The test that catches it.** With `context: 1`: a file with a lone CR, a UTF-16 file with a BOM, a UTF-8 BOM file, and a file with matches on lines 2, 4, 6 and 7. Every context line's number must equal the line's number in the file, the UTF-16 match must read cleanly, no BOM may appear, and no line may print twice.

### Trap 22: the engine chosen by what is installed

**What goes wrong.** Zeroclaw picks `rg`, `grep` or an internal walker at startup, and Cline runs `rg` and falls back to a JavaScript scan when it returns nothing. The same call then differs in dialect (`-E` against Rust `regex`), ignore rules, hidden files, hits per file (one against all), order and path separators. `(source, probe)`

**Why it is easy to get wrong.** A fallback is added so that the tool works on a machine without the preferred program, and nobody compares the two outputs.

**The test that catches it.** Run one fixture with and without the preferred engine on `PATH`; the outputs must be byte-identical. In Runvil there is one engine, so the test becomes: no test double or second backend may differ in case rule, ordering, line endings or skipped files.

### Trap 23: ambient state that changes what a call means

**What goes wrong.** A user's `RIPGREP_CONFIG_PATH` containing `--fixed-strings` silently turns regex searches into literal ones, and a pattern shaped like `--pre=/x/payload.sh` can be read as a flag. Cursor's `Grep` passes `--no-config` and `--regexp <pattern>`; its `pi_grep` has the `--` guard and lacks `--no-config`, and pi itself added both after two bug reports (commits `8c4e87389`, `3d43d2e17`). `(bundle, probe, source)`

**The test that catches it.** Set `RIPGREP_CONFIG_PATH` to a file holding `--fixed-strings` and search for `a.b`: the result must not change. Search for the pattern `--pre=/x/payload.sh`: it must be matched as text.

## How the harnesses do this

Twelve harnesses, one section each, in this order: Claude Code, Copilot, Zed, pi, rig, zeroclaw, Codex CLI, Gemini CLI, OpenCode, Cline, Aider and Goose. Three of them ship no grep tool at all (Codex CLI, Aider, Goose), and rig ships one that its own agent leaves unregistered. Evidence labels: `(source)` is code read at the pinned commit, `(probe)` is something run against that code or against ripgrep 15.1.0-cursor5, `(web)` is a documentation page or an API response, `(bundle)` and `(daemon)` are Cursor's own build.

### Claude Code

**Source:** the `Grep` tool extracted from `claude.exe` 2.1.311 (win32-x64, Bun-compiled) as `tools/grep.js`, 28 KB, byte range 256108403 to 256126241, decoded as latin1. Schema builder `_6t` at 245424014, registration `Y0` at 256112432. Constants from `limits.js`. `(source)`

**Tool name:** `Grep`. The `userFacingName()` returns `Search`.

**Parameter set:**

| Parameter | Type | Required | Default |
|---|---|---|---|
| `pattern` | string | **yes** | |
| `path` | string | no | `current working directory` |
| `glob` | string | no | |
| `output_mode` | enum | no | **`files_with_matches`** |
| `-B` | number | no | |
| `-A` | number | no | |
| `-C` | number | no | |
| `context` | number | no | |
| `-n` | boolean | no | `true` |
| `-i` | boolean | no | |
| `-o` | boolean | no | `false` |
| `type` | string | no | |
| `head_limit` | number | no | **250** |
| `offset` | number | no | `0` |
| `multiline` | boolean | no | `false` |

There is also a spread `...U3()` at the end, which adds the shared tool-safety fields this extract does not include.

Three parameters Cursor's `Grep` does not have: `context`, `-n` and `-o`. Cursor has no parameter that Claude Code lacks (15 against 12). The two schemas are nearly identical otherwise, down to the description strings, so Cursor's `Grep` was probably adapted from Claude Code's.

**Caps, with units.** `maxResultSizeChars: 20000` on the tool object, so 20,000 characters of result, which is a larger budget than Cursor's 15,000. `TGo = 250` is the default `head_limit`. `-M 500` is passed to ripgrep (`--max-columns 500`), half of Cursor's 1000, and there is no `--max-columns-preview`, so a long line is replaced entirely rather than previewed. The `head_limit` is applied by `K2e`:

```js
function K2e(e, n, r = 0) {
  if (n === 0) return { items: e.slice(r), appliedLimit: void 0 };   // 0 means unlimited
  let s = n ?? TGo,                                                  // 250 when omitted
      h = e.slice(r, r + s),
      y = e.length - r > s;
  return { items: h, appliedLimit: y ? s : void 0 };
}
```

So `head_limit: 0` means unlimited in both harnesses, reached by an explicit check here and by a falsy coercion in Cursor. But an **omitted** `head_limit` is 250 here and unlimited in Cursor. That is the biggest behavioral difference between the two tools.

**Output shape.** For content mode (`mapToolResultToToolResultBlockParam`):

```js
let Se = Y2e(L, z),                                                // "limit: N, offset: M"
    we = s || (z && (w ?? 0) > 0 ? "No entries at this offset" : "No matches found"),
    ve = Se ? `${we}\n\n[Showing results with pagination = ${Se}]` : we;
return { tool_use_id: q, type: "tool_result", content: ve };
```

So the content body is the joined match lines, and the trailer is `[Showing results with pagination = limit: 250, offset: 0]`, which is a very different shape from Cursor's `[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]`. An example:

```text
src/a.ts:12:  return null;
src/a.ts:40:  return null.
src/b.ts:3:   return null;
```

Claude Code uses ripgrep's default `path:line:content` with no heading and no blank line between files, and it passes `--json` in content mode (`je.push(h === "content" ? "--json" : "--null")`) so it parses ripgrep's JSON stream rather than a NUL format. Its parser (`DGo`) then re-renders each match as `<path>\0<line>:<content>` internally, which is the same intermediate Cursor gets from `-0` directly.

Files mode:

```text
Found 3 files
src/a.ts
src/b.ts
src/c.ts
```

Count mode adds a summary:

```text
src/a.ts:3
src/b.ts:1

Found 4 total occurrences across 2 files. with pagination = limit: 250
```

Note the stray period before `with`: `... across 2 files.${Se ? ` with pagination = ${Se}` : ""}` produces `files. with pagination`, with a space where there should be none. That is a defect in the shipped string.

**The one design idea worth taking.** `-o` / `only_matching`. Cursor has no way to ask for just the matched text, so a model hunting for a value inside a long line has to parse the line itself. Claude Code's implementation is careful about it too: it uses `submatches` from the JSON stream and computes each submatch's line by counting newlines before the offset (`LGo`), so a multiline match produces one output line per matched line. That is a useful capability.

**The one design idea worth rejecting.** The **250 default `head_limit`**. It silently truncates, the trailer is a `limit: N` string rather than a `Showing X of Y` count, and a model reading `[Showing results with pagination = limit: 250]` has no idea whether it got 1 result or 250. Cursor's "no default, but a 2000 hard cap with a trailer that reports the cut" is better on both counts.

### VS Code Copilot

**Source:** `extensions/copilot/src/extension/tools/node/findTextInFilesTool.tsx` in the VS Code repository (`microsoft/vscode`, local checkout at commit `b208bc25`, 2026-10-25), 33,928 bytes. Tests: `node/test/findTextInFilesTool.spec.tsx` (14.9 KB) and `node/test/findTextInFilesResult.spec.tsx` (11.6 KB). Name from `common/toolNames.ts`: `FindTextInFiles = 'grep_search'`. `(source)`

**Tool name:** `grep_search`. There is a second enum entry `FindTextInFiles = 'copilot_findTextInFiles'` in an adjacent enum, so both spellings exist.

**Parameter set:**

| Parameter | Type | Required | Default |
|---|---|---|---|
| `query` | string | **yes** | |
| `isRegexp` | boolean | no | `true` |
| `includePattern` | string | no | |
| `maxResults` | number | no | |
| `defaultMaxResults` | number | no | injected by `resolveInput` |
| `includeIgnoredFiles` | boolean | no | `false` |

Six parameters. Note `query`, not `pattern`, and the tool **throws** if you send `pattern`:

```typescript
if ((options.input as unknown as Record<string, string>).pattern) {
    throw new Error('The property "pattern" is not supported, please use "query"');
}
```

That is a deliberate accommodation for models trained on other tools: rather than silently ignoring the wrong field, it tells the model what to do, which is worth copying.

**Caps, with units.** Three numbers, all in *results* (matches, not files, not lines):

- `getDefaultMaxResults()` returns the experiment config or **100**.
- `getMaxResultsCap()` returns the experiment config or **200**.
- `maxResults = Math.min(input.maxResults ?? input.defaultMaxResults ?? defaultMaxResults, maxResultsCap)`, so the caller can never exceed 200.
- A `maxResults` above the cap sets `askedForTooManyResults`, which renders ` (maxResults capped at 200)` in the tag output.
- The search itself asks for `maxResults * 5` in grep style, "to account for the fact that some files will be cut off".

Plus a **20,000 ms timeout**, via `raceTimeoutAndCancellationError`, with an embedded hint: `Timeout in searching text in files with regex search, try a more specific search pattern or change regex/literal mode`.

**Output shape.** Two formats, chosen by an experiment flag. The grep style, from `FindTextInFilesGrepResult`:

```text
Found 2 matches in 1 file for "const"

/src/a.ts
5:const a = 1;
9:const b = 2;
```

The header is `Found ${matchText} in ${filesText} for "${query}"${elided}` where `elided` is ` (showing ${shownMatchText} in ${shownFilesText})` when anything was cut. Between files: one blank line, rendered as `${'\n'}` before each file's block.

The tag style, from `FindTextInFilesResult`, wraps each match in XML:

```text
1 match
<match path="/file.txt" line=6>
Line before
This is a test
Line after
</match>
```

**Copilot does not shell out.** It calls the editor's own search service:

```typescript
const searchResult = this.searchService.findTextInFiles2(
    { pattern: query, isRegExp },
    { include: patterns, maxResults: maxResults + 1,
      useExcludeSettings: includeIgnoredFiles ? ExcludeSettingOptions.None : ExcludeSettingOptions.SearchAndFilesExclude,
      useIgnoreFiles: includeIgnoredFiles ? { local: false, parent: false, global: false } : undefined,
      caseInsensitive: true },
    token);
```

That single choice changes six things:

1. **It searches the workspace index, not the filesystem.** So it sees whatever the editor's search provider sees, which in VS Code includes the ripgrep-based provider by default but can include a remote index, a search extension, or a language server's index.
2. **It can see unsaved buffers.** A file with unsaved edits is searched as the editor holds it, not as it sits on disk. Cursor's `Grep` cannot do this (the `dsv3-1018` description claims it can, but no code path implements it), and Claude Code cannot do it at all.
3. **It obeys the user's search settings.** `useExcludeSettings: SearchAndFilesExclude` means `search.exclude` and `files.exclude` from the *editor configuration* apply, in addition to ignore files. So a user's personal exclusion applies to the model's search, which is a policy decision Cursor does not make.
4. **`caseInsensitive: true` is hardcoded.** Every Copilot grep is case-insensitive. There is no parameter to change it. That is the opposite of Cursor's default and a surprising choice for a code search tool.
5. **Results are `Uri` plus `Range` objects.** So clickable links and editor navigation come free, and the text rendering is a separate presentation step. That is the same separation Zed makes and Cursor does not.
6. **There is no offset.** `maxResults` is the only dial, and the tool has no way to resume. A model that needs match 201 has to narrow the query.

Two behaviors from the tests. **It retries with plain text when a regex search finds nothing**: `if (!results.length && queryIsValidRegex)` re-runs with `isRegExp: !isRegExp`. So a model that accidentally writes a regex metacharacter gets results anyway. And **it produces a hint on a zero-result search** that names the exclusion mechanism:

```text
Your search pattern might be excluded completely by either the search.exclude settings or .*ignore files.
If you believe that it should have results, you can check into the .*ignore files and the exclude setting (here are some excluded patterns for reference:[...]).
Then if you want to include those files you can call the tool again by setting "includeIgnoredFiles" to true.
```

That is the answer to the silent-omission problem Cursor has.

**The one design idea worth taking.** The **zero-result hint that names the exclusion mechanism and the parameter that overrides it.** Cursor's `No matches found` is a dead end; Copilot's tells the model what to try next. Combined with `includeIgnoredFiles` as a parameter, this turns the most common failure (searching an ignored tree) into a recoverable one.

**The one design idea worth rejecting.** **Hardcoded `caseInsensitive: true`.** A code search that cannot be made case-sensitive will find `const` in `constant`, and a model that wants an exact identifier has no recourse. Cursor's `-i` defaulting to false is correct.

### Zed

**Source:** `crates/agent/src/tools/grep_tool.rs` in the Zed repository at commit `37faef81f7` (2026-10-27), 57,868 bytes and 1,557 lines; the tool code ends at line 408 and the test module runs from line 410. The search itself is `crates/project/src/search.rs` (query and regex), `project_search.rs` (candidate pipeline and caps) and `project.rs:4947-4987` (`Project::search`), and the file classifier is `crates/file_content/src/file_content.rs`. The first version of this section read commit `d9ad6aff67` (2026-08-22); every change is listed in `### Zed checkout drift`. `(source)`

**Tool name:** `grep`. `const NAME: &'static str = "grep"` (`grep_tool.rs:102`), `kind() = acp::ToolKind::Search` (`:104`). `ToolKind` is a display category: it is handed to `send_tool_call` and gates nothing (`thread.rs:1698`, `:3774`). **Grep has no permission gate.** `grep_tool.rs` never calls `decide_permission_from_settings`, `authorize` or anything in either `tool_permissions.rs`, and the settings UI lists `grep` among the tools that do not call it (`tool_permissions_setup.rs:1405`, `:1415`). It is on in the `write` and `ask` profiles, absent from `minimal`, and still available in restricted mode. The only filters are `file_scan_exclusions` and `private_files` (default `**/.env*`, `**/*.pem`, `**/*.key`, `**/*.cert`, `**/*.crt`, `**/secrets.yml`), applied twice, with no notice to the model. `(source)`

**Parameter set:**

| Parameter | Type | Required | Default |
|---|---|---|---|
| `regex` | string | **yes** | |
| `include_pattern` | string | no | all files |
| `offset` | u32 | no | `0` |
| `case_sensitive` | bool | no | `false` |
| `max_ancestor_lines` | u32 | no | `10` |
| `context_lines` | u32 | no | `2` |

Six parameters, deriving from `JsonSchema`, so the model sees doc comments as descriptions. The last two are new since `d9ad6aff67`, where the same two numbers were constants. `max_ancestor_lines` caps the syntax-ancestor prefix (its first line plus this many more; `0` permits only the first line), and `context_lines` is the fallback, used only when no ancestor exists or its capped prefix does not hold every matched line. The `regex` description tells the model the pattern is "parsed by the Rust `regex` crate" (`grep_tool.rs:38`), and the engine is `fancy_regex`. Inside the pattern, `\c` and `\C` override `case_sensitive` (`search.rs:234-239`), and the regex is built `.multi_line(true).crlf(true)`, so `.` does not match `\n` without `(?s)` (`:257-261`). `include_pattern` is a globset glob matched case-sensitively against the full path including the root name; a bad glob returns `invalid include glob pattern: {error}`. `(source)`

**Caps, with units.**

| Cap | Value | Unit | Where |
|---|---|---|---|
| page size `RESULTS_PER_PAGE` | 20 | merged context blocks, printed as "matches" | `grep_tool.rs:86`, check at `:305` |
| `offset` | any `u32` | raw regex ranges, skipped one at a time before merging | `:299-302` |
| `max_ancestor_lines`, `context_lines` | 10, 2 | rows | `:273`, `:285`, `:288` |
| files let through the search | 5,001 | `MAX_SEARCH_RESULT_FILES + 1`; the cut gives no marker | `project.rs:4975`, `project_search.rs:166`, `:649` |
| `MAX_SEARCH_RESULT_RANGES` | 10,000 | raw ranges, checked between buffers | `project_search.rs:167`, `:650` |
| `MAX_CONCURRENT_BUFFER_OPENS` | 64 | buffers in flight | `project_search.rs:174` |
| classifier window, prefilter header | 1,024 bytes, 8 KiB | bytes | `file_content.rs:12`, `project_search.rs:843` |
| load cap | 6 GiB | bytes, `>=`, at buffer load only | `worktree.rs:1721-1727` |
| `fancy_regex` backtrack limit | 1,000,000 | attempts, the library default, not set by Zed | fancy-regex `lib.rs` `(web)` |
| output size, timeout | none | no byte, line, snippet or line-length cap, and no deadline; the tool waits for the worktree scan and the parse with no timeout | no `MAX_`, `truncate` or `Duration` in `grep_tool.rs` outside the rows above |

**It does not shell out, it does not use a grep crate, and it does not search only buffers.** From `crates\project\src\search.rs`:

```rust
use aho_corasick::{AhoCorasick, AhoCorasickBuilder};
use fancy_regex::{Captures, Regex, RegexBuilder};
```

`search.rs` imports both crates. The grep tool always builds `SearchQuery::regex` (`grep_tool.rs:172`), which runs on `fancy_regex`; the `AhoCorasick` searcher belongs to the plain-text variant, is built from one pattern, and grep never builds that variant. `(source)`

`Project::search` mixes memory and disk. Open buffers are sampled first, as clones of their rope, so unsaved edits are visible (an engine test edits a file without saving it and gets the new matches, `project_tests.rs:12221-12253`). Every other file is read from disk by a prefilter: it reads 8 KiB, classifies the first 1,024 bytes, skips a binary file silently with a debug log, decodes a BOM, a UTF-16 guess and `chardetng` input, and runs the pattern over the decoded text. A hit opens the file as a buffer, and a worker scans a fresh snapshot (`project_search.rs:832-905`, `:559-633`). So the earlier line that Zed searches "the live document model, not the filesystem" was wrong: unopened files come off the disk. The engine hands back `Anchor` ranges over live snapshots, and the tool turns them into row labels. Results are re-sequenced into submission order, so the output order is worktree id, then path component order, then match offset, whatever the thread timing (`:532-554`, `:636-662`). `(source)`

**What `fancy_regex` costs and provides.** It hands ordinary patterns to the `regex` crate and runs its own backtracker only for fancy features, look-around and back-references. The backtracker has a limit of 1,000,000 attempts that Zed leaves at the default; hitting it returns an error, and Zed then drops that file's later matches without a word (`search.rs:554`, `project_search.rs:870`, `:886`). So the earlier line that Zed "can hang on a pathological pattern" is overstated: a pathological pattern is bounded, and the cost is silent loss of results in that file. Zed accepts look-around and back-references, which ripgrep (Cursor, pi, Gemini CLI, zeroclaw) and rig's `regex` crate refuse, and Cursor inherits the linear-time guarantee from ripgrep. The lockfile pins fancy-regex 0.19.0 and I read `main` for the limit, so the number could differ at the tag. `(source, web)`

**Output shape.** Markdown, with a syntax-aware twist no other harness has:

````text
Found 1 matches:

## Matches in root/test_syntax.rs

### fn top_level_function › L1-3
```
fn top_level_function() {
    println!("This is at the top level");
}
```
````

The `###` line is the **enclosing syntax node path**, joined with ` › `: `mod feature_module › pub mod nested_module › pub fn nested_function › L10-14`. That comes from `snapshot.syntax_ancestor(full_lines)` and `snapshot.symbols_containing(matched.start, None)`. So the model is told which function the match is inside, which is information Cursor's `Grep` makes the model get with a second `Read`.

The snippet is a fenced code block with a **fixed three-backtick fence** in the string the model receives (`grep_tool.rs:341-343`). The longer fence, chosen to exceed any backtick run inside the snippet (`MarkdownCodeBlock`), is used only for the UI content block (`:364-372`, test `:706-759`), so a snippet that itself contains a fence can break the model-facing markdown. `(source)`

Pagination:

```text
Showing matches 1-20 (there were more matches found; use offset: 20 to see next page):
```

and the complete case is `Found {n} matches:`, and zero is `No matches found`.

Note `Found 1 matches` in the test expectations: **there is no pluralization**. A single match says `1 matches`. That is a defect, confirmed by the test's own `expected` string.

**The pagination drifts.** `offset` skips raw regex ranges one at a time before any merging (`:299-302`), the page counts merged blocks (`:305`, `:382`), and the next-offset hint assumes blocks (`:398-401`). Take 25 blocks made of 50 raw ranges. Page 1 shows blocks 1 to 20 and says `use offset: 20`. Page 2 skips raw ranges 0 to 19, which is only blocks 1 to 10, so it shows blocks 11 to 25 under `Found 15 matches:`, and blocks 11 to 20 appear twice (hand trace, not run; no test in the file sets `offset` above 0). Nothing is lost, because the restart point never passes what page 1 displayed. An `offset` past the last match prints `No matches found`, indistinguishable from no matches. Every page re-runs the search from the first file and stops early once the page is full, because dropping the search handle cancels it (`:190-193`), so page N costs more than page 1. The 10,000-range cut prints the same "more matches" header and a next offset, and paging past it ends in `No matches found`; the 5,001-file cut prints nothing. `(source)`

**Two more things Zed does that the others do not.**

First, **context is syntax-aware.** From the range mapping (this excerpt is from `d9ad6aff67`):

```rust
if let Some(ancestor_node) = snapshot.syntax_ancestor(full_lines.clone()) {
    let full_ancestor_range = ancestor_node.byte_range().to_point(&snapshot);
    let end_row = full_ancestor_range.end.row.min(full_ancestor_range.start.row + MAX_ANCESTOR_LINES);
    ...
    if capped_ancestor_range.contains_inclusive(&full_lines) {
        return (capped_ancestor_range, Some(full_ancestor_range), symbols)
    }
}
```

At `37faef81f7` both numbers are parameters: the cap is `start.row.saturating_add(input.max_ancestor_lines)` (`grep_tool.rs:273`) and the fallback is `context_lines` whole rows above and below (`:283-291`). So instead of `-A 2`/`-B 2`, it shows the whole enclosing syntax node, capped at `max_ancestor_lines` (default 10) lines past the start, and appends `\nN lines remaining in ancestor node. Read the file to see all.` when the node was capped. That is a much better answer to "show me enough context" than a fixed line count, and it is only possible because Zed has a parser. A test pins the notice with `max_ancestor_lines: 0` and `context_lines: 1` over `fn sample() {`, `needle();`, `}` (`test_grep_context_options`, `:1051-1110`):

````text
Found 1 matches:

## Matches in root/main.rs

### fn sample › L1
```
fn sample() {
```

2 lines remaining in ancestor node. Read the file to see all.
````

Second, **it streams clickable links separately from the model-visible text.** `event_stream.update_fields(... .content(content).locations(locations))` sends ACP `ResourceLink` blocks per match while the string returned to the model stays link-free. The test asserts this explicitly (`assert!(!output.contains("file://"))`). Cursor puts everything in one text blob.

**What Zed reads and what it never shows the model.** Searched text is always decoded UTF-8: a BOM gives the encoding, a UTF-16-looking NUL pattern gives UTF-16, invalid UTF-8 falls back to `chardetng`, and line endings are normalized to `\n` (a test finds GBK, EUC-KR and UTF-16 files with the BOM outside the match offsets, `project_tests.rs:13213-13272`). Skipped with no notice: gitignored files (`.gitignore`, `.git/info/exclude`, the global gitignore), `file_scan_exclusions`, `private_files`, binary files, files of 6 GiB or more, and files where the backtracker gave up. `.ignore` and `.rgignore` are not read. Directory symlinks that point outside a worktree are not scanned, and `grep_tool.rs` has no symlink check for a symlinked file, where `read_file` has 21 lines of them (medium confidence; no test pins it). `(source)`

**The one design idea worth taking.** **Syntax-aware context with a remaining-lines notice.** A model reading `### fn parse_header › L40-58` plus a ten-line snippet knows where it is in the file and what the enclosing function is. That removes a `Read` follow-up in most cases, which saves tokens.

**The one design idea worth rejecting.** **`offset` and the page size count different things, and no total is ever given.** `Showing matches 1-20 (there were more matches found; use offset: 20 to see next page)` never says how many there are, so the model cannot tell whether it needs one more page or twenty; and after any merge the next page repeats blocks from the last one. Page on one unit, report a total or a lower bound, and make every cut say so. Cursor's `at least N` is worse than an exact count but better than nothing. `(source)`

### pi, and Cursor's `pi_grep` port

**Source and name.** pi's `grep` is `packages/coding-agent/src/core/tools/grep.ts` (325 lines) in the pi repository (package `@earendil-works/pi-coding-agent` 1.4.0) at commit `a92283056` (2026-10-27), with `truncate.ts` (314 lines) and `path-utils.ts` beside it. Its tests are the six at `test/tools.test.ts:877-968` plus one `ctx.cwd` test. The tool is `grep` and it is opt-in: pi's default tools are `read`, `bash`, `edit` and `write` (`settings-manager.ts:85`), and `grep`, `find` and `ls` ship together as the read-only set. Cursor's port is `pi_grep`, registered at 4561631 under the internal id `PI_GREP`, always named `pi_grep`, with one description on every branch. Its schema `ZRe` is at 4554895 and its executor `LocalPiGrepExecutor` is at 9202537 in `main.js` and 16851630 in `daemon.cjs`, the two copies holding the same logic. It is offered only when `featureFlags.enablePiTools === true`, which registers all seven pi tools in one call; where that flag is set is not in the sources. `(source, bundle, daemon)`

**The port is a rewrite of an older pi snapshot.** From `git log -S` on pi (an inference, since nothing states it): the port has pi's `--` guard (commit `3d43d2e17`, 2026-04-30) and the `context === 0` fast path (`e9ba9e2eb`, 2026-04-16). It lacks `--no-config` (`8c4e87389`, 2026-10-21), the lone-CR fix (`fecc19e4e`, 2026-10-27), pi's per-call file cache and its `(unable to read file)` fallback. Which pi release Cursor took is not in the sources. Both ran against the same ripgrep, 15.1.0-cursor5, so the differences below are tool code and not the binary. `(source, probe)`

| Dimension | pi original | Cursor `pi_grep` | Wrong side |
|---|---|---|---|
| description | a paragraph naming the caps and `.gitignore` | `Search file contents for a pattern using pi-compatible grep semantics.` | Cursor: the model is not told 100, 50 KB or 500 |
| `--no-config` | yes | no; the argv is the four flags in the fence below | Cursor: with `RIPGREP_CONFIG_PATH` naming a file that holds `--fixed-strings`, pi returned 2 lines for `a.b` and the port returned 1 |
| cancellation | the abort signal kills the child | `execute` takes no signal | Cursor, as far as visible |
| context, lone CR | `split("\n")` after `\r\n` becomes `\n`, so numbers match ripgrep | `.replace(/\r/g,"\n")` before the split, the bug pi fixed | Cursor: the match for `needle` prints as `progress` |
| fast path (`context` 0) | strips `\r\n` and every `\r` | strips one trailing `\n` | Cursor: `crlf.txt:2: needle one\r` |
| context file reads | cached per call | `readFile` inside the per-match loop | Cursor |
| a context read fails | that match prints `(unable to read file)` | the exception text becomes the whole result | Cursor |
| permission layer | none | `shouldBlockRead` on the root, then per match after the limit has been applied | Cursor: blocked files use up slots; with one file blocked and `limit: 1`, 1 of 14 runs said `No matches found` |
| output separator | `/` always | OS separator (`deep\x.js` on Windows) | Cursor, on Windows |
| bad JSON line | try/catch, skipped | bare `JSON.parse` | Cursor, in theory |
| missing path | `Path not found: <abs>` | raw `ENOENT: no such file or directory, stat '<abs>'` | pi is friendlier |
| byte notice | `50.0KB limit reached` | `50KB limit reached` | cosmetic |
| bugs both have | exit code 2 discards matches; the limit notice on an exact fit; `limit=2N`; fractional `context`; UTF-16 and BOM files in context mode; overlapping blocks repeated; a glob with a slash anchored to the process cwd | the same | both |

Caption: pi's `grep.ts` beside Cursor's `pi_grep`, from differential runs over identical inputs. `(probe)`

**Parameters.** Seven, with the same names and description strings in both, verified byte for byte. Cursor's zod schema, verbatim:

```js
U$.Ikc({
  pattern: U$.YjP().describe("Search pattern (regex or literal string)"),
  path: U$.YjP().optional().describe("Directory or file to search (default: current directory)"),
  glob: U$.YjP().optional().describe("Filter files by glob pattern, e.g. '*.ts' or '**/*.spec.ts'"),
  ignoreCase: U$.zMY().optional().describe("Case-insensitive search (default: false)"),
  literal: U$.zMY().optional().describe("Treat pattern as literal string instead of regex (default: false)"),
  context: U$.aig().optional().describe("Number of lines to show before and after each match (default: 0)"),
  limit: U$.aig().optional().describe("Maximum number of matches to return (default: 100)")
})
```

Comparison against `Grep`:

| Dimension | `Grep` | `pi_grep` |
|---|---|---|
| case flag | `-i` (hyphenated) | `ignoreCase` (camelCase) |
| output mode | three modes | content only |
| line numbers | unconditional | always on, `--line-number` |
| context | `-A`, `-B`, `-C` separately | `context` only, symmetric |
| literal mode | none (regex only) | `literal: true`, `--fixed-strings` |
| limit | `head_limit`, no default | `limit`, default 100 |
| offset | `offset` | none |
| type filter | `type` | none |
| multiline | `multiline` | none |
| glob flag | `--iglob` | `--glob` |
| output format | `<workspace_result>` envelope | plain `path:line: text` |
| engine | `LocalGrepExecutor` | `LocalPiGrepExecutor` |

pi coerces arguments with typebox `Value.Convert` before the tool runs (`"5"` becomes `5`); whether Cursor's zod layer coerces is not in the sources. `context` and `limit` are `Type.Number` with no minimum, so `limit: 1.5` returns 2 matches and says `1.5 matches limit reached`, and `context: 0.5` prints `plain.txt-1.5- ` lines with empty text. `limit` of 0 or less becomes 1. `(source, probe)`

**Caps, with units.**

| Cap | Value | Unit |
|---|---|---|
| match limit | 100, floor 1 | ripgrep `match` events, one per matching line; context lines are not counted |
| output bytes | 51,200 | UTF-8 bytes of the joined output, measured before the trailer, so the final text runs over (51,213 in pi, 51,211 in the port) |
| output lines | none | `maxLines: Number.MAX_SAFE_INTEGER`, set by pi itself |
| one line | 500 | UTF-16 code units, then `... [truncated]`; the cut can leave a lone high surrogate |
| context, time | none | lines each side; only an abort signal stops a long search, and the port has none |

The port's argv, from the bundle:

```js
const r=["--json","--line-number","--color=never","--hidden"];
t.ignoreCase&&r.push("--ignore-case"),
t.literal&&r.push("--fixed-strings"),
void 0!==t.glob&&r.push("--glob",t.glob),
r.push("--",t.pattern,n);
```

pi's argv starts with `--no-config`, so `RIPGREP_CONFIG_PATH` cannot change the tool; the port's does not. `--json` means it parses ripgrep's event stream rather than the NUL wire format. `--glob`, not `--iglob`, so filtering is case-sensitive, and a glob that contains a slash resolves against ripgrep's working directory, which neither tool sets: `deep/*.js` with `path: "sub"` matched nothing (probe). There is no `--max-columns`, so ripgrep emits each matching line whole (a 5 MiB line produced 5,243,675 characters of stdout and 18 to 20 MB of extra resident memory per such match), but the model never sees that text, because each line is cut to 500 UTF-16 units. The port's `formatMatches` re-reads a matched file (`readFile(i.filePath,"utf8")`) only when `context` is above 0 or when ripgrep gave no `lines.text` because the line is not valid UTF-8; the read happens once per match with no cache, and an exception in it fails the whole call. One unreadable file makes ripgrep exit 2, and both tools then return only the first stderr line and no matches, even though a readable file matched. Result order is undefined: 8 outputs in 8 runs on 20,000 files, in both. `(bundle, daemon, probe)`

**Output.** A match prints `path:N: text` and a context line `path-N- text`, relative to the search root (the basename when a file is searched). Blocks are not merged, so overlapping context repeats lines and there is no `--` separator. Captured output for the case pi's own test asserts and the port reproduces:

```
context.txt-1- before
context.txt:2: match one
context.txt-3- after

[1 matches limit reached. Use limit=2 for more, or refine pattern]
```

The trailer lists the notices in a fixed order, limit, bytes, line cut, joined with `. ` inside one pair of brackets and set off by a blank line. The three strings as the port builds them (pi spells the middle one `50.0KB limit reached`; I did not trigger all three at once, so a combined trailer is assembled from the code):

```
[100 matches limit reached. Use limit=200 for more, or refine pattern]
[50KB limit reached]
[Some lines truncated to 500 chars. Use read tool to see full lines]
```

The limit notice appears when the limit is reached, not when more exist: 3 matches at `limit: 3` still says `3 matches limit reached`. The byte notice gives no count of what was dropped. `(probe)`

**Binary files.** In a directory search ripgrep skips a file that holds a NUL and emits no match events for it (`bytes_searched: 0`), so the file is gone and a search that only it satisfies says `No matches found`. For a file named as `path`, the matches arrive with the raw NUL in the text and the end event carries `binary_offset: 13`; `pi_grep` reads only `match` events (`"match"!==t.type` returns early), so it never sees that field. The model gets no hint either way. `(bundle, probe)`

**`pi_grep` is simpler than `Grep` in one dimension and richer in another; neither is a superset.**

- Simpler: no output modes, no offset, no `-A`/`-B`/`-C` split, no type filter, no multiline.
- Richer: `literal` (fixed-string mode), a default `limit` of 100, and three distinct truncation reasons instead of one combined trailer.

**`Grep` is the better model for a fresh implementation, for four reasons, two of them corrected.** First, `pi_grep` has no output modes, so a model that wants paths only must parse content output, which is worse for "which files mention X". Second, the earlier claim that its `--json` parse is "more code and more failure surface than the NUL format, for no gain" is contestable. The JSON stream is what lets `context: 0` use ripgrep's own decoded text with no re-read, UTF-16 transcoding included (a UTF-16 file prints clean at `context: 0` and as garbage at `context: 1`). The failure surface is elsewhere, in the exit-code-2 rule, the CR handling and the context re-read. Third, its context implementation re-reads each matched file with a second decoder, which races with the search and has its own bugs. Fourth, it has no offset, so paging is impossible: the trailer tells the model to run again with `limit=2N`, which restarts the search, and since the order is not stable the larger result need not extend the earlier one. `(probe)`

**The one idea worth taking: keep the tool's contract independent of ambient state.** pi runs ripgrep with `--no-config` and puts `--` before the pattern, and pins both with tests (`grep.ts:162-168`; `tools.test.ts:929-951`, `:953-968`). The history says why: a user's `RIPGREP_CONFIG_PATH` containing `--fixed-strings` silently turned regex searches into literal ones and the agent got incomplete results with no error (pi commit `8c4e87389`), and a pattern shaped like `--pre=...` could be read as a flag (`3d43d2e17`). The port has the first bug today. For Runvil the rule is that every behavior of `grep` comes from its arguments and nothing else, and a pattern is data, never a flag. Confidence high, because the incident, the fix, the test and the probe on the unfixed port agree. From the earlier version of this report, a second idea to take: **the `literal` flag.** A fixed-string mode costs almost nothing and Cursor's `Grep` lacks it, so a model searching for `a.b` must remember `a\.b` (D11). `(source, probe)`

**The one idea worth rejecting: building context by re-reading the file with a second decoder and a hand-written line splitter.** ripgrep has already read the file and numbered its lines. The re-read produced the lone-CR bug and still produces a garbage UTF-16 file (`��n\u0000e\u0000...`, match line included), a BOM on line 1, repeated lines for overlapping blocks, and in the port one full read per match. Take context from the same bytes in one pass, merge overlapping ranges, and mark the gaps. Confidence high, because three of the four failures are reproduced and the fourth, a file that changes between the two reads, is visible in the code. `(probe)`

### rig

**Source and name.** `crates/rig-tools/src/search.rs` (385 lines) in the rig repository at commit `a83b774d0` (2026-10-25), with five tests in `search/tests.rs`, the walk in `workspace.rs`, and a sibling `list.rs`. **Rig has no `glob` tool and no `grep` tool.** One tool, `search`, does both jobs: `files: true` with an empty pattern is a file-name search. The shipped agent registers neither `search` nor `list`: `plugins/rig-basics/src/coding_tools.rs:1-6` says it ships `read`, `edit`, `write` and `shell` "as pi has", and that the library's other tools "are the shell's job here". A plugin registers `search` with `add_tool_per_call`. `(source)`

| Name | Type | Required | Default and description |
|---|---|---|---|
| `pattern` | string | yes | `The regular expression; with files, empty lists every file.` |
| `path` | string or null | no | the current directory |
| `glob` | string or null | no | no filter; `Only files matching this glob, such as *.rs.` |
| `hidden` | boolean | no | `false`; also hidden files and directories, but files git ignores stay out |
| `files` | boolean | no | `false`; only the paths of files with a match, one per line |

`SearchArgs` has `deny_unknown_fields`, so a misspelled key is an error. There is no case flag (the model writes `(?i)`), no context, multiline, count, type filter, offset, per-file limit or fixed-string mode: `foo(` is a pattern error. `(source)`

| Cap | Value | Unit |
|---|---|---|
| `MAX_MATCHES` | 200 | hits, one per matching line, counted across files |
| `MAX_FILES` | 1000 | names, in `files` mode |
| `MAX_LINE` | 300 | Unicode scalar values (`char_indices().nth(300)`, so 300 `é` is 600 bytes); no marker on a clipped line |
| `MAX_FILE_BYTES` | 16 MiB | bytes; a file of 16,777,216 bytes is searched and one byte more is skipped |
| read batch | 100 | files per `read_files` call, each read whole into memory |
| compiled regex | 10,485,760 | bytes of compiled program, the `regex` default |
| output bytes, time | none | no deadline; `blocking` starts a thread that nothing can stop |

Output is plain `path:line: text`, with no grouping and no blank lines. Under a base, paths are relative (test `search/tests.rs:99-100`):

```
src/a.rs:1: todo one
```

A cut ends with a notice as the last line, and the notice fires on an exact fit: 199 hits gave none, 200 and 201 gave it, and so did 1000 and 1001 files in `files` mode (probe):

```
[stopped at 200 matches]
```

Nothing found is `No matches.`; a bad pattern is `bad pattern: ` plus the regex crate's text, for `(` `bad pattern: regex parse error:\n    (\n    ^\nerror: unclosed group`. On Windows the same call printed `src\a.rs:1: todo one`, while rig's tests expect `/` and its CI runs on ubuntu and macOS only. `(source, probe)`

**The order it runs in.** The pattern compiles before the walk, so a bad pattern beats a bad glob beats a missing path. The whole path list is then built before anything is read. Files are read 100 at a time; a file that is not valid UTF-8 is skipped whole with no trace (a Latin-1 file's ASCII lines are lost, probe); a BOM stays in line 1; NULs in valid UTF-8 are searched and shown. A prefilter runs the pattern once over the whole text (`multi_line(true).crlf(true)`) and skips the line loop when nothing can match; it switches itself off for `\A`, `\z`, a `-` flag or a lone `\r`, and a fuzz of 400,000 random patterns found no case where it changed the hits. Matching is per line of `str::lines`: `\n` and `\r\n` end a line, a lone `\r` stays in it, and a pattern that crosses lines never matches. `(source, probe)`

**What the walk skips.** The `ignore` crate's single-threaded `WalkBuilder`, with no sorter, so the order is the file system's. `.gitignore` is inert without a `.git` (before `git init`, `ignored.txt` and `build/` were searched; probe). A path part named `target` is dropped at any depth, and so is any dot-name unless `hidden` is set or the glob names one. `glob` is the `ignore` override grammar and not a plain glob: `*.rs` matches at any depth, `src/*.rs` anchors at the root, `src`, `src/` and `test*` match nothing, and a glob beats the ignore rules, so `*`, `**` and `**/*` return ignored trees. Symlinks are never listed and walk errors are dropped. Every one of these omissions is silent, and so are files over 16 MiB and, over a remote host, a read call that failed (the whole chunk of 100 becomes `No matches.`). `(source, probe)`

**Cost and engine.** `regex` 1.12.3 runs in process in linear time: `^(a|aa)+$` took 246 us, 966 us and 3.85 ms for 100,000, 400,000 and 1,600,000 characters. Look-around and back-references are refused with a message. 100 files of 16 MiB reached a peak working set of 1,626 MiB in 1.47 s. A no-match search over 1,000 files of 200 lines took 155.5 and 170.5 ms per call against ripgrep's 48.8 ms (probe, one warm Windows machine). `(probe)`

**The one idea worth taking: search as a pure function that returns data.** `find(&dyn Workspace, &SearchArgs) -> Result<Found, ToolExecutionError>` walks with `Workspace::files` and reads with `Workspace::read_files`, whose default is one read per path and which a remote backend overrides to read 100 files in one script. `render` is a separate pure function, and the tool publishes the typed `Found` and the `FilesRead` hashes beside the text, so `protected_paths` filters hits, `glob_rules` counts them as touched paths and the TUI renders counts without parsing text. That is rig's answer to D1. Confidence medium-high: tests and four consumers pin the shape, the memory cost is measured, and the two fixes (batch by bytes, return the list lazily) are untested. `(source, probe)`

**The one idea worth rejecting: silent omission behind a reassuring zero-result text.** A model that gets `No matches.` concludes the code is absent, and the cause may be a non-UTF-8 file, a symlink, a `.gitignore` the walk did not read, or a glob that matched a directory and nothing under it, all under a description that says "Skips files ignored by git" while `*` brings ignored directories back. Count skipped files by reason and say so in the text. Smaller rejects: the stop notice on an exact fit, and an unsorted local walk behind a 200-hit cap with no offset. Confidence high, because every behavior is probed against the crates; whether models are misled is not measured. `(probe)`

### zeroclaw

**Source and name.** `crates/zeroclaw-tools/src/content_search.rs` in the zeroclaw repository at commit `2d85371f3` (2026-09-18, workspace 0.8.5); the checkout is about five weeks behind upstream. The tool is `content_search`, wrapped as `RateLimitedTool(PathGuardedTool(...))` and auto-approved. **It has three backends, picked once at construction:** `rg` if `which::which("rg")` succeeds, else `grep`, else an internal Rust walker (`content_search.rs:42-50`), and the same call has different semantics on each. `(source)`

| Param | Type | Required | Default | Description |
|---|---|---|---|---|
| `pattern` | string | yes | | `Regular expression pattern to search for` |
| `path` | string | no | `.` | `Directory to search in, relative to workspace root. Defaults to '.'` |
| `output_mode` | enum | no | `content` | `content`, `files_with_matches`, `count` |
| `include` | string | no | none | `File glob filter, e.g. '*.rs', '*.{ts,tsx}'` |
| `case_sensitive` | boolean | no | true | |
| `context_before`, `context_after` | integer | no | 0 | content mode only |
| `multiline` | boolean | no | false | ripgrep only; an error on the fallbacks |
| `max_results` | integer | no | 1000 | clamped to 1000; `0` is kept |

| Cap | Value | Unit |
|---|---|---|
| `MAX_RESULTS` | 1000 | matching lines (content), files (`files_with_matches`), per-file lines (`count`), applied after the search ends |
| `MAX_OUTPUT_BYTES` | 1,048,576 | bytes, cut on a character boundary, then `[Output truncated: exceeded 1 MB limit]` |
| `TIMEOUT_SECS` | 30 | seconds; the ripgrep child is not killed on timeout |
| result cap in the agent loop | 50,000 | bytes, cut in the middle |
| action budget | 20 per 3600 s | successful calls, reads included |

Output is `path:line:text`, then a cut marker and a total (3 matches with `max_results` 3, through the verbatim formatter; the marker fires when the count merely reaches the cap):

```
a.rs:1:hit 1
a.rs:2:hit 2
a.rs:3:hit 3

[Results truncated: showing first 3 results]

Total: 3 matching lines in 1 files
```

Count mode prints `a.rs:3` and `Total: 5 matches in 2 files`; no hits is `No matches found.`. `(probe)`

Behavior that bites. Exit code 2 or more from `rg` or `grep` discards all stdout and returns `Search error: {stderr}`: one unreadable directory in a tree with 301 matches turned the whole search into an error. Stdout is fully buffered before the cap is applied. With no `path`, the wrapper checks the regex `pattern` as if it were a path, so a pattern like `../x` is refused as `Path blocked by security policy`. The formatter relativizes by stripping the workspace string prefix with no component check: `/wsx/a.rs:1:z` under workspace `/ws` becomes `x/a.rs:1:z`. A context line whose text contains `:3:` is classified as a match and counted. The `grep` backend uses POSIX `-E`, and `include` braces work in `rg --glob` but not in `glob::Pattern` or `grep --include`. The internal backend uses `regex::RegexBuilder` and `glob::Pattern`, walks in sorted path order, skips a file that is not UTF-8 or holds a NUL, re-checks each canonical path, and checks a deadline per entry. `(source, probe)`

**The one idea worth taking: an in-process fallback that is deterministic and cancellable by design.** A sorted walk, a deadline check per entry, a NUL and non-UTF-8 skip, the canonical path rechecked per entry, and the same formatter as the subprocess path (`content_search.rs:444-738`, `:863-992`). For Runvil, whose VFS has no subprocess, this is the shape to copy. Confidence high. **The one idea worth rejecting: choosing the engine by `which` at construction and hiding it behind one schema.** The three backends differ on regex dialect, ignore files, hidden files, ordering, `include` braces and what an unreadable file does to the result, and 12 runs of the `rg` backend gave 12 orders where `--sort path` gave one. Reject the exit-code-2 rule that throws away partial matches too. Confidence high. `(source, probe)`

### Codex CLI

**Source and name.** `github.com/openai/codex`, `main` at commit `bea47c06dfac` (committed 2026-10-28T05:46Z; the workspace version is `0.0.0`, so no release number can be derived). **There is no grep tool.** `core/src/tools/spec_plan.rs` registers three file-touching tools, `exec_command`, `apply_patch` and `view_image`, and the instruction templates of all 12 models in `models.json` tell the model to "reach first for `rg` or `rg --files`" in the shell. A dedicated `grep_files` existed and was removed on 2026-03-25 by PR 15775 (commit `178c3b15b4`; the last version is at its parent `32c4993c8a`, `core/src/tools/handlers/grep_files.rs`, 176 lines). It sat behind `experimental_supported_tools`, which none of the 12 models lists. `(source, web)`

What the model uses instead, and what the removed tool took:

| Tool | Parameters | Role in searching |
|---|---|---|
| `exec_command` | `cmd`, `workdir`, `tty`, `yield_time_ms`, `max_output_tokens` (default 10000) | `rg`, `grep`, `rg --files`; the model adds `-m` or `head` itself |
| removed `grep_files` | `pattern`, `include`, `path`, `limit` (default 100) | files whose contents match, newest first |

| Cap | Value | Unit |
|---|---|---|
| `DEFAULT_LIMIT`, `MAX_LIMIT` (removed tool) | 100, 2000 | file paths |
| `COMMAND_TIMEOUT` (removed tool) | 30 | seconds |
| shell output | 10,000 | tokens, at four bytes per token, cut in the middle with `…N tokens truncated…` |

The removed tool spawned `rg --files-with-matches --sortr=modified --regexp <pattern> --no-messages`, with `--glob <include>` when given, and returned one path per line with no line numbers and no match text. It ran `rg` through `tokio::process::Command` directly and bypassed the sandboxed exec path. `command.output()` holds all of stdout before `limit` is applied, exit 1 means no matches, and any other exit code returned `rg failed: {stderr}` and dropped the stdout. The `limit` cut gave no count and no marker, and no match is the text below with `success: false` (test `tests/suite/grep_files.rs:122`):

```text
No matches found.
```

The shell route, which is what a Codex model uses today, comes back wrapped (source `core/src/tools/context.rs:543-567`):

```text
Chunk ID: <id>
Wall time: 0.0000 seconds
Process exited with code 0
Original token count: <n>
Output:
<command output>
```

The gate is the OS sandbox plus an approval policy, not a per-tool check; there is no read allowlist, and the TUI classifies `rg` and `grep` as `Search` for display only. Cursor's `rg` tool name on its `gpt5-codex` and `codex-cloud` branches mirrors the shell command these models use in Codex CLI; that last link is my inference and is not in the sources. `(source)`

**The one idea worth taking: a file list ordered by recency as the first triage step.** `--files-with-matches --sortr=modified` with a 100-path default and a 2000-path ceiling gives the model the likely files first, and it reads them next. Confidence medium: it hides all line content, so it is a first step and not a full grep. **The one idea worth rejecting: applying `limit` after the search finished, with no count and no marker, and throwing away stdout on any exit code other than 0 or 1.** The model cannot tell "exactly 100 matches" from "100 of 4000". Confidence high (`grep_files.rs:143-172`). `(source)`

### Gemini CLI

**Source and name.** `github.com/google-gemini/gemini-cli` tag `v0.66.0`, commit `970413207732` (2026-10-27). The tool is `grep_search` (display name SearchText, legacy alias `search_file_content`). The default implementation is `RipGrepTool` in `packages/core/src/tools/ripGrep.ts` with a vendored `rg` binary; when ripgrep is off or missing, the legacy `GrepTool` in `grep.ts` registers under the same name with a smaller schema and runs `git grep -E --ignore-case --untracked`, then `grep -r -n -H -E -I -i`, then a JavaScript `RegExp(pattern, 'i')` walk. Both format through `grep-utils.ts`. Two schema sets exist, one for Gemini 3 and one for every other model, with the same parameter names. `grep_search` is `allow` by policy. Everything below is the ripgrep tool. `(source)`

| Name | Type | Required | Default and notes |
|---|---|---|---|
| `pattern` | string | yes | "Rust-flavored regular expression"; validated as a JavaScript `RegExp` unless `fixed_strings` |
| `dir_path` | string | no | `.`; a file or a directory |
| `include_pattern` | string | no | all files; becomes `--glob` |
| `exclude_pattern` | string | no | a regex applied client-side to each matching line |
| `names_only` | boolean | no | false |
| `case_sensitive` | boolean | no | **false** (ignore case) |
| `fixed_strings` | boolean | no | false |
| `context`, `after`, `before` | integer, min 0 | no | 0 in the schema, but see auto-context |
| `no_ignore` | boolean | no | false |
| `max_matches_per_file` | integer, min 1 | no | none; becomes `--max-count` |
| `total_max_matches` | integer, min 1 | no | 100; no maximum |

| Cap | Value | Unit |
|---|---|---|
| total matches | 100 | non-context matches |
| output line | 2000 | code points, then `... [truncated]` |
| timeout | 30000 | ms |
| auto-context | 50 lines for 1 match, 15 for 2 or 3 | lines each side |
| stderr kept | 20 KB | bytes |
| tool output | 64 KiB | UTF-8 bytes, cut by the scheduler with a pointer to a temp file |

Output, from `formatGrepResults` run on hand-built matches (the file had five lines and a trailing newline, so `L6-` is a phantom line that comes from the JS reader of the legacy tool; the ripgrep tool sets `context` first and skips that reader). `(probe)`

```
Found 1 match for pattern "needle" in path ".":
---
File: a.ts
L1- one
L2- two
L3: needle here
L4- four
L5- five
L6- 
---
```

At 100 matches the header reads `Found 100 matches for pattern "needle" in path "." (filter: "*.ts") (results limited to 100 matches for performance):`, and the test is `>=`, so it fires when nothing was cut. There is no paging: the model narrows with `include_pattern` or `dir_path`, or raises `total_max_matches`.

Three behaviors. **Auto-context:** with 1 to 3 matches and no `names_only` and no context parameter, ripgrep runs a second time over just those files with `--context 50` or `--context 15`, so the model skips the follow-up read; a source comment credits about 10 percent fewer turns on SWE-bench, which I did not verify. **Dialect mismatch:** the pattern is checked with JavaScript `new RegExp` and run as Rust, so `(?i)foo` is rejected before ripgrep sees it, and `(?=x)y` passes the check, fails inside ripgrep with exit 2 and `error: look-around, including look-ahead and look-behind, is not supported`, and comes back as a success. Timeouts, a missing binary and exit 2 are all returned as success text. **Silent drops:** a line that is not valid UTF-8 is dropped, results are filtered through `.gitignore` and `.geminiignore` after the cap so ignored matches can eat the budget, and `trimEnd()` removes `\r` and trailing spaces. No `--sort` is passed with 4 threads, so which matches survive the cap depends on thread scheduling (not measured). `(source, probe)`

**The one idea worth taking: auto-context for small result sets.** One to three hits come back with 50 or 15 surrounding lines in the same call, so the model skips a `Read`. Confidence medium: the 10 percent figure is a source comment, and the description still says context defaults to 0. **The one idea worth rejecting: validating the pattern in one dialect and running it in another, and returning ripgrep failures as success text.** A valid-looking lookahead fails inside ripgrep and arrives as normal output, and a valid Rust inline flag is refused before ripgrep sees it. Confidence high, because both mismatches are probed. `(source, probe)`

### OpenCode

**Source and name.** `github.com/anomalyco/opencode` (formerly `sst/opencode`) tag `v1.19.5`, commit `810c0eba1a73` (2026-10-28), with the `dev` branch at `fa1260d6ca` as a second reading. The tool is `grep`, in `packages/opencode/src/tool/grep.ts` with its description in `grep.txt`, running `Ripgrep.grep`, which is `rg --json`. Three tool sets coexist at the tag: the legacy set above is the one the session loop uses; a V2 rewrite in `packages/core/src/tool/` passes `file` for a file path; and `dev` adds `literal`, `caseSensitive` and `limit` (default 100), a 30-second timeout, and the no-match text `No matches found`. `(source)`

| Name | Type | Required | Default and description |
|---|---|---|---|
| `pattern` | string | yes | `The regex pattern to search for in file contents` |
| `path` | string | no | the instance directory; `The directory to search in. Defaults to the current working directory.` |
| `include` | string | no | `File pattern to include in the search (e.g. "*.js", "*.{ts,tsx}")` |

There is no output mode, context, case, multiline or count parameter; the description sends counting to `bash` with `rg`.

| Cap | Value | Unit |
|---|---|---|
| matches | 100, across all files | matching lines |
| line preview | 2000, then `...` | UTF-16 code units; a trailing high surrogate is dropped first |
| one ripgrep JSON record | 65,536 | bytes of the raw JSON line; over it the whole call fails |
| wall clock | none in the legacy tool; 30,000 in `dev` | ms |

The argv is `rg --no-config --json --hidden --no-messages [--glob=<include>] --glob=!**/.git/** -- <pattern> <file or .>`. Output groups hits by file; the blank lines come from ripgrep's `lines.text`, which keeps its `\n`, and from the loop joining with another (probe, two files):

```
Found 2 matches
<root>\x.txt:
  Line 2: line2 needle


<root>\sub\y.ts:
  Line 1: export const needle = 1

```

At 100 hits the header becomes `Found 100 matches (more matches available)` and a notice follows, including when there are 100 and nothing more. The header counts matching lines rather than files. Zero matches is `No files found`, a file-search wording for a content search. A CRLF file arrives with `\r\n` inside the text. A file path as `path` searches its parent directory, because the legacy tool does not pass `file`. A ripgrep exit 2 with some output returns the rows with `partial: true`, which `grep.ts` never reads, so the model gets partial results with no warning. A matching line over about 64 KiB (a minified file) makes the whole search fail, and a line that is not valid UTF-8 comes from ripgrep as base64 `bytes`, the adapter requires `lines.text`, and the call fails with `Invalid ripgrep match output`. Four runs over 400 matching files gave four orders. `(source, probe)`

**The one idea worth taking: fetch `limit + 1`, slice to `limit`, and set `truncated` from the extra row, with a wall-clock timeout whose message says what to do.** `dev` does both (`D:plugin/grep.ts:110-145`): it fixes the false notice at 100 and bounds a hostile tree in a few lines. **The one idea worth rejecting: letting one record fail the whole call.** A single minified line over 64 KiB, or a single non-UTF-8 line, turns a search with 99 good matches into an error; a search tool should drop or mark the bad record, keep the rest, and say that something was skipped. `(source, probe)`

### Cline

**Source and name.** `github.com/cline/cline` tag `v4.1.26` (commit `a459f1d`, 2026-10-25). The tool is `search_codebase`: executor `sdk/packages/core/src/extensions/tools/executors/search.ts` (521 lines), schema `schemas.ts:106-125`, wrapper `definitions.ts:360-418`. The legacy line (tag `v4.0.12`, commit `f267bf4`) had `search_files(path, regex, file_pattern)`. There is no glob tool. `(source)`

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `queries` | array of string | yes | none | `Array of regex search queries to execute` |

The runtime also accepts a bare string, a string array or `{queries: string}`. There is no path, glob, case, context or output-mode parameter: scope is always the session cwd. Each query runs as its own search inside one `Promise.all`.

| Cap | Value | Unit |
|---|---|---|
| results per query | 100 | parsed match objects |
| context | 2 | lines each side, fixed |
| ripgrep time | 5,000 | ms; the tool allows 30,000 per query and 60,000 per call |
| ripgrep stdout buffer | 10 MiB | UTF-16 units; the excess is dropped and the trailing partial event cut |
| tool output | 48,000 | characters, cut in the middle |
| provider-side cut | 8,000 | characters per string, applied later by the message builder, which silently undoes the 48,000 |

Output, from the JavaScript fallback (query `(?=TARGET)TARGET`, which ripgrep refuses; probe):

```
Found 5 results for pattern: (?=TARGET)TARGET
Searched 3 files.

b/two.ts:4:1
  2: second
  3: third
> 4: TARGET two
  5: fourth
  6: fifth
```

**Two engines behind one name.** The first path is `rg --json --context=2 --max-count=1 -i <query>`: always case-insensitive, one hit per file, no `-e` and no `--`, and the matched line itself is never printed; before-context is attached to the previous match, and every context line is followed by a blank line because `lines.text` keeps its newline. Zero matches, a pattern ripgrep refuses, non-JSON output or the 5 s timeout all return `null`, which hands the query to a JavaScript scan with `new RegExp(query, "gim")` over files whose extension is on a 44-entry allowlist; that path reports every hit and marks the hit line with `>`. So the same query returns different files, different hit counts, different line display and different path separators (`b\two.ts` against `b/two.ts`) depending on whether ripgrep found anything. Six runs of one 60-file query gave six outputs, an empty query lists every file, and a query that starts with `-` makes ripgrep print its version and falls through. `Found 100 results` is a floor, and `Searched N files` counts only the JavaScript scan. `(source, probe)`

**The one idea worth taking: a bounded `rg --json` capture that drops the trailing partial event.** The executor stops buffering past 10 MiB and cuts at the last newline before parsing; one match in a giant single-line file produces a multi-hundred-megabyte event, and the code comment records that it crashed the process with a `RangeError` before. Any Runvil content search that streams must bound the buffer and treat the last record as possibly partial (test `search.test.ts:102-125`). Confidence high. **The one idea worth rejecting: two engines behind one tool name.** Five divergences were reproduced; in Runvil the engine is one Rust `regex` implementation behind `Access`, and a second path whose results differ by exit code should not exist. Confidence high. `(source, probe)`

### Aider

**Source and name.** `github.com/Aider-AI/aider` at commit `5dc9490bb35f` (2026-05-22, `__version__` `0.86.3.dev`). **There is no grep tool**, and no tool schema for any file work: the model writes edit text in its reply and Aider does the file work. Four mechanisms stand in for search: the repo map (a ranked symbol outline), filename and identifier matching against the chat text, the `--watch-files` regex for `AI` comments, and the "Did you mean" line matcher inside edit failures. A user can run `/run grep ...` and choose to add the output to the chat. `(source)`

| Question | What Aider does |
|---|---|
| parameters, paging | none; the map is one block per request, refreshed per `--map-refresh` |
| caps | `--map-tokens` 1024 tokens (`0` disables), a map up to 15 percent over budget is accepted, map lines cut at 100 characters, the watcher skips files over 1 MiB |
| ranking | PageRank over a definitions-and-references graph, personalized to chat files, mentioned files and identifiers in the request; tags cached per file in SQLite keyed on mtime |
| encoding | strict UTF-8; an undecodable file contributes nothing |
| binary | by extension, tree-sitter language lookup and decode failure; no NUL scan |
| truncation | the map elides with `⋮...` lines and drops whole files to fit, and nothing says which |

The map's framing text, verbatim (`coders/base_prompts.py:45-48`), turns the map into a request mechanism:

```
Here are summaries of some files present in my git repository.
Do not propose changes to these files, treat them as *read-only*.
If you need to edit any of these files, ask me to *add them to the chat* first.
```

The watcher's matching rule, compiled with `re.IGNORECASE` (`watch.py:69-71`):

```
(?:#|//|--|;+) *(ai\b.*|ai\b.*|.*\bai[?!]?) *$
```

**The one idea worth taking: the repo map as a search substitute.** A fixed token budget, ranked by the chat files, the files named in the request and the identifiers in it, gives the model a map without any tool call. Confidence medium, because ranking is a heuristic and needs a tree-sitter grammar per language. **The one idea worth rejecting: silent caps.** The 1 MiB watcher skip and the map's dropped files leave no marker, so neither the model nor the user can tell what was left out (`watch.py:107-109`, `repomap.py:666-706`). Confidence high. `(source)`

### Goose

**Source and name.** `github.com/aaif-goose/goose` (formerly `block/goose`) at commit `ee8500d5e060` (2026-10-28), latest release `v1.56.0`. The developer extension ships exactly `write`, `edit`, `shell`, `tree` and `read_image`, pinned by a test. **There is no grep tool.** The system prompt says to prefer `rg` through `shell` ("prefer findstr or Select-String" on Windows), and the only code-aware search is `analyze`, which finds a symbol's definition and callers and not text. `(source)`

| `analyze` parameter | Type | Required | Default |
|---|---|---|---|
| `path` | string | yes | none |
| `focus` | string | no | none; a symbol name that triggers call-graph mode |
| `max_depth` | `u32` | no | 3; `0` is unlimited |
| `follow_depth` | `u32` | no | 2 |
| `force` | bool | no | false; allows large output |

| Cap | Value | Unit |
|---|---|---|
| shell stream | 2000 lines or 50,000 bytes | the tail of 50 lines (at most 10,000 bytes) is kept and the full text saved to a temp file |
| `analyze` output | 50,000 | bytes compared with `output.len()`, though the error says "chars" |
| any tool response | 200,000 | characters |

A search is a `shell` call, and Goose adds a 300 s timeout, a null stdin, a lossy decode, full buffering and the tail truncation. **A zero-match search is an error result:** `is_error` is `timed_out || exit_code.unwrap_or(1) != 0`, and `rg` exits 1 when nothing matches, so the model reads (`dev/shell.rs:339-349`; public issue 11835, where the maintainers fixed only the desktop rendering):

```
(no output)

Command exited with code 1
```

A match list over 2000 lines shows only its last 50 lines, so the model sees the end of whatever order the program printed. `analyze` refuses instead of cutting, with a message that names the remedies: ``Output too large ({n} chars, limit 50000). Use `force: true` to override, or narrow scope with max_depth/focus.`` `(source, web)`

**The one idea worth taking: a guard that turns an oversized answer into an instruction.** `analyze` says how big the output was, what the limit is and the two remedies, so the model recovers in one step instead of reading a truncation. Confidence medium: the message is right, the unit label is wrong. **The one idea worth rejecting: grep as an untyped shell call with exit-code-as-error and tail-first truncation.** A normal zero-match search is flagged as a failure, and a long hit list shows only its end. Confidence high. `(source, web)`

### How each harness pages

One row per harness, because whether the model can get result 101 varies widely between harnesses. The order column decides whether a resume is safe.

| Harness | Resume | Unit | Default cap | Order stable? | The model's move |
|---|---|---|---|---|---|
| Cursor | global `offset` + `head_limit`; 5-entry cache | matched entries, context not counted | none; 2000 parsed, 15000 UTF-16 units rendered | only when the mtime sort finishes | read `Use offset=N` from the trailer |
| Claude Code | `offset` + `head_limit`, re-run each call | entries or files | 250 | mtime newest-first, no budget | same; `head_limit: 0` for all |
| Zed | `offset` (raw ranges) + fixed page, re-run each call | merged blocks | 20 | yes: worktree id, path, offset | follow `use offset: 20`; page 2 may repeat page 1 |
| Copilot | none | matches | 100, cap 200 | by path | narrow the query |
| pi | none; the trailer says `Use limit=2N` | matching lines | 100 | no: 8 outputs in 8 runs | re-run bigger; the prefix may change |
| rig | none | hits, names | 200 hits, 1000 names | file-system order; sorted on `Remote` | narrow `path`, `glob` or `pattern` |
| zeroclaw | none; `max_results` | lines, files | 1000 | `rg` no; internal backend sorted | raise `max_results` or narrow |
| Codex CLI | none; removed `grep_files` had `limit` only | file paths | 100, ceiling 2000 | `--sortr=modified` | add `-m` or `head` in the shell |
| Gemini CLI | none | non-context matches | 100, no maximum | no `--sort`, 4 threads | raise `total_max_matches` or narrow |
| OpenCode | none | matching lines | 100 | no: 4 outputs in 4 runs | narrow |
| Cline | none | parsed match objects | 100 | no: 6 outputs in 6 runs | refine the query |
| Aider | no search tool | n/a | n/a | n/a | the repo map |
| Goose | none; the model pages a saved file with `head`, `tail`, `sed` | lines, bytes | 2000 lines, 50,000 bytes, tail kept | ripgrep's | read the temp file |

Caption: how each harness lets the model see past its first page of matches. `(source, probe)`

What this implies. First, only three of thirteen can resume, and each shows a different way to fail: Cursor's order is stable only when its stat pass finishes, which is why it needs a cache; Claude Code re-runs the search and orders by mtime, so a resume is only as stable as the files; Zed has the stable order and loses the unit, so page 2 repeats page 1. Second, a harness that cannot resume leaves the model one move, to narrow, so the trailer's wording carries the whole recovery: rig names `path` or `glob`, Gemini names `include_pattern` and `dir_path`, OpenCode says "a more specific path or pattern", and pi says to raise the limit, which cannot help when the byte cap was the cause. Third, the cut notice is unreliable on an exact fit in six harnesses: Cursor (`>=` at 2000), pi, rig, zeroclaw, Gemini CLI and OpenCode print it when the count merely reaches the cap, so the model cannot tell exactly N from more than N. My reading for Runvil: offer `offset` only with path order and one counting unit (raw matches, not blocks), or do not offer it and make the cut notice say how to narrow. Confidence medium-high, because three harnesses show the three ways a resume fails and the nine that do not page leave the model no way to see result 101.

### Across all the harnesses

| Dimension | Cursor `Grep` | Claude Code `Grep` | Copilot `grep_search` | Zed `grep` | pi `grep` | rig `search` |
|---|---|---|---|---|---|---|
| engine | ripgrep subprocess | ripgrep subprocess | editor search service | in-process `fancy_regex` over open buffers and disk | ripgrep subprocess, `--json` | in-process `regex` over `Workspace` files |
| searches unsaved buffers | no (claimed, not implemented) | no | **yes** | **yes**, for files with an open buffer | no | an `Overlay` search sees previewed edits |
| parameters, pattern field | 12; `pattern` | 15; `pattern` | 6; `query` | 6; `regex` | 7; `pattern` | 5; `pattern` |
| output modes, default | 3; `content` | 3; `files_with_matches` | 1 (grep or tag style); content | 1; content | 1; content | 2 (hits, or `files`); hits |
| default `head_limit`, `0` | **none**; `0` is unlimited | **250**; `0` is unlimited | n/a | n/a | `limit` 100, `0` becomes 1 | n/a |
| paging | `offset` + `head_limit` | `offset` + `head_limit` | none | `offset` only | none | none |
| page size, cap unit | 2000 entries or files | 250 (default) entries or files | 100 (default), 200 cap, matches | **20 merged blocks**, matches | 100 matching lines | 200 hits, 1000 names |
| second cap | 15000 UTF-16 units rendered | 20000 chars (`maxResultSizeChars`) | 200 tokens-ish via prompt budget | none; no output cap of any kind | 50 KiB UTF-8 bytes; 500 UTF-16 per line | 300 scalar values per line, no marker |
| hard engine cap | 8 MiB stdout, 2000 lines | `--max-columns 500` | 20000 ms timeout | 10000 ranges, 5001 files, backtrack limit 1,000,000 | none; only an abort signal | 16 MiB per file, 100 files per batch |
| matching rule | Rust `regex`, linear | Rust `regex`, linear | JS `RegExp` (backtracking) | `fancy_regex` (backtracking, bounded) | Rust `regex`, linear | Rust `regex` 1.12.3, linear; look-around refused |
| literal mode | no | no | `isRegexp: false` | no | `literal` | no |
| case default, parameter | case-sensitive; `-i` | case-sensitive; `-i` | **insensitive, hardcoded**; none | insensitive; `case_sensitive`, `\c`, `\C` | sensitive; `ignoreCase` | sensitive; `(?i)` inline |
| context | `-A`, `-B`, `-C` lines | `-A`, `-B`, `-C`, `context` lines | none | `context_lines` 2, or the enclosing syntax node up to `max_ancestor_lines` 10 | `context` lines, re-read from the file | none |
| line numbers, grouping | always `  N:`; bare path line, blank line between | `path:N:`; no grouping | tag style only; header, blank line between | range labels `L10-14`; `## Matches in <path>` | `path:N: text`, `path-N- text` | `path:N: text`, no grouping |
| gap separator | `  --`, conditional | ripgrep's `--` | none | none | none; overlapping blocks repeat lines | none |
| truncation signal | `[Showing X of at least Y ...]` | `[Showing results with pagination = limit: N]` | `(showing X matches in Y files)` | `(there were more matches found...)`; none at the 5,001-file cut | `[100 matches limit reached ...]`, also on an exact fit | `[stopped at 200 matches]`, also on an exact fit |
| binary handling | invisible in content, visible in files | ripgrep default | n/a (index) | 1,024-byte classifier, silent skip | ripgrep skips; the port reads only `match` events | no check; non-UTF-8 skipped whole |
| symlinks, ignore rules | `--follow` **always**; gitignore on, `--hidden` | no `--follow` | editor settings | outside-worktree directory links not scanned; gitignore yes, `.ignore` no | no `-L`; gitignore only in a git repo | never followed; `.gitignore` inert without `.git` |
| encoding | invalid bytes to U+FFFD, then a 2 s repair pass | no repair | editor decoded | BOM, UTF-16 guess, `chardetng` | non-UTF-8 line re-read lossy; UTF-16 garbled in context mode | non-UTF-8 file skipped whole; BOM kept |
| line endings | split on `\n`, strip one `\r`; lone CR is one line | not visible | editor-normalized | all to LF, lone CR too | every `\r` deleted; the port keeps `\r` on the fast path | `str::lines`; a lone `\r` stays in the line |
| order | mtime newest-first, 2 s stat budget, else walk order | mtime newest-first, no budget | by path | worktree id, path, offset | undefined | file-system order; sorted on `Remote` |
| error rewrite | yes, invalid regex | no | no | `invalid include glob pattern: {error}`; regex errors unchanged | none; ripgrep's stderr | `bad pattern: ` plus the engine text |
| permission gate | none (post-hoc filter) | `checkPermissions` + stash | editor settings | **none**; `file_scan_exclusions`, `private_files` | none; the port adds `shouldBlockRead` after the limit | one footprint, `Reads(Paths([root]))`; confinement only with a `WorkDir` |

Caption: Cursor beside Claude Code, Copilot, Zed, pi and rig; each cell is sourced in that harness's section above.

| Dimension | Cursor `Grep` | zeroclaw `content_search` | Gemini CLI `grep_search` | OpenCode `grep` | Cline `search_codebase` |
|---|---|---|---|---|---|
| engine | ripgrep subprocess | `rg`, `grep` or an internal walker, chosen once at startup | ripgrep; fallback `git grep`, `grep`, a JS walk | ripgrep `--json` | ripgrep, else a JS scan, per query |
| parameters, pattern field | 12; `pattern` | 9; `pattern` | 12; `pattern` | 3; `pattern` | 1; `queries[]` |
| output modes, default | 3; `content` | 3; `content` | 2 (`names_only`); matches | 1 | 1 |
| limit parameter | `head_limit`, no default | `max_results` 1000, `0` kept | `total_max_matches` 100, no maximum; `max_matches_per_file` | fixed 100 (`dev`: `limit`) | fixed 100 |
| paging | `offset` + `head_limit` | none | none | none | none |
| cap unit | entries or files | lines, files, or per-file lines by mode | non-context matches | matching lines | parsed match objects |
| second cap | 15000 UTF-16 units | 1 MiB output; 50,000 bytes in the loop | 64 KiB scheduler cut; 2000 code points per line | 2000 UTF-16 per line; 64 KiB per JSON record fails the call | 48,000 chars, then 8,000 chars per string |
| hard cap | 8 MiB stdout, 2000 lines | 30 s; the child is not killed | 30,000 ms; 4 threads | none (`dev`: 30 s) | 5 s for ripgrep; 30 s per query |
| matching rule | Rust `regex`, linear | Rust `regex` (rg, internal) or POSIX ERE (grep) | Rust `regex`; validated as JavaScript | Rust `regex` | rg `-i`, one hit per file, else JS `gim` |
| literal mode | no | no | `fixed_strings` | none (`dev`: `literal`) | no |
| case default, parameter | case-sensitive; `-i` | sensitive; `case_sensitive` | **insensitive**; `case_sensitive` | sensitive; none | insensitive always |
| context | `-A`, `-B`, `-C` | `context_before`, `context_after` | `context`, `after`, `before`; auto 50 or 15 lines for 1 to 3 matches | none | 2 lines, fixed |
| format | `<workspace_result>`, `  N:` | `path:line:text`, `Total:` footer | `File:` blocks, `L3:` and `L1-` | `Line N:` under `path:` | `file:line:col`, context lines |
| truncation signal | `[Showing X of at least Y ...]` | `[Results truncated: showing first N results]`, also on an exact fit | `(results limited to N matches for performance)`, also on an exact fit | `(more matches available)`, also on an exact fit | `(Showing first 100 results...)`; then a cut with no way back |
| binary handling | invisible in content, visible in files | rg skips; names a file as `binary file matches`; internal skips NUL | no flag; explicit file returned matches with `binary_offset` | silent skip | rg skips; JS has no NUL check |
| symlinks, ignore rules | `--follow` always; `--hidden` | rg honors ignores; internal walks `.git` and `node_modules` | gitignore and `.geminiignore`, applied after the cap | `.git` excluded, hidden searched, gitignore on | rg honors; JS uses an allowlist |
| encoding | U+FFFD, then a repair pass | lossy; internal skips non-UTF-8 files | non-UTF-8 lines dropped | one non-UTF-8 line fails the call | rg: empty text; JS: U+FFFD |
| line endings | strip one `\r`; lone CR is one line | internal `str::lines`; the others rg defaults | `trimEnd()` hides `\r` | `\r\n` passed through | rg raw; JS splits `\n` |
| order | mtime, then walk order | rg: 12 outputs in 12 runs; internal sorted | no `--sort`, 4 threads | 4 outputs in 4 runs | 6 outputs in 6 runs |
| error handling | rewrites invalid regex | `Search error: {stderr}`; exit 2 drops all matches | failures returned as success text | stderr text; partial results unreported | `Search failed: ...` |
| permission gate | none (post-hoc filter) | path guard, readable roots, 20 calls an hour | policy `allow`; `.git`, `.env` segments refused | permission `grep`, default allow; external directory asks | approval policy; `.clineignore` not enforced |

Caption: Cursor beside zeroclaw, Gemini CLI, OpenCode and Cline.

| Dimension | Cursor `Grep` | Codex CLI | Aider | Goose |
|---|---|---|---|---|
| tool | `Grep` | none; `exec_command` with `rg`; removed `grep_files` | none; repo map, `/run grep` | none; `shell` with `rg`; `analyze` for symbols |
| parameters | 12 | removed tool: `pattern`, `include`, `path`, `limit` | none | the shell's |
| paging | `offset` + `head_limit` | none; `limit` only | none | the model pages a temp file |
| caps and units | 2000 entries; 15000 UTF-16 units | removed tool: 100 paths, ceiling 2000; shell: 10,000 tokens | map 1024 tokens, 100 characters per line | 2000 lines or 50,000 bytes per stream; 200,000 characters per response |
| truncation signal | `[Showing X of at least Y ...]` | removed tool: none; shell: `…N tokens truncated…` | `⋮...`, dropped files unnamed | tail kept with a temp-file notice |
| matching rule | Rust `regex` | ripgrep's, through the shell | tree-sitter tags, PageRank | ripgrep's, through the shell |
| order | mtime | removed tool: `--sortr=modified` | ranked | arrival order |
| gating | none (post-hoc filter) | OS sandbox plus approval policy | the human picks files | permission mode by tool name |
| zero-match result | `No matches found` | removed tool: `No matches found.` with `success: false` | not applicable | `(no output)` and `Command exited with code 1`, an error result |

Caption: Cursor beside the three harnesses that have no grep tool, and the shell or human mechanism each uses instead.

Three rows carry the main differences. **Copilot's and Zed's "searches unsaved buffers"** is the only way a model can see an edit it has not saved. **Zed's "in-process over buffers and disk"** is the only way to get syntax-aware context. **Cursor's "no default limit but a trailer that reports the cut"** is the only combination that never silently cuts a result without saying so, and even it says `at least` where the count is exact. Two rows show where the field agrees on a failure and nobody has fixed it: binary files are skipped without a notice everywhere a directory search was probed, and nine of twelve harnesses leave the model no way to see past the cap.

### Zed checkout drift

The Zed section was first written against a working copy at commit `d9ad6aff67` (2026-08-22). It is now reconciled with the canonical checkout at commit `37faef81f7` (2026-10-27), read directly in `grep_tool.rs`, `search.rs`, `project_search.rs` and `file_content.rs`. Some claims were wrong in both checkouts and are corrected in the Zed section and the tables above; they are listed here as well so that the old text can be found. The first block is what changed between the checkouts, from an LF-normalized `git diff --no-index` (the old working copy has CRLF line endings, which added about 1.4 KB to the old file). `(source, probe)`

| What the report said | What is true at `37faef81f7` |
|---|---|
| `grep_tool.rs` is 53 KB; the permission gate is `tool_permissions.rs`, 55 KB | the file is 57,868 bytes (the old file is 53,068 bytes raw and 51,641 bytes with LF endings), and it never calls either `tool_permissions.rs`: grep has no permission gate. A search of the file for `decide_permission`, `authorize` and `tool_permissions` finds nothing, and `ToolKind::Search` is only a display category |
| four parameters (`regex`, `include_pattern`, `offset`, `case_sensitive`) | six: `max_ancestor_lines` (default 10) and `context_lines` (default 2) are new (`grep_tool.rs:20-25`, `:66-77`), pinned by `test_grep_context_options` (`:1051-1110`) |
| `CONTEXT_LINES = 2` and `MAX_ANCESTOR_LINES = 10` are constants | they are the defaults of those two parameters; the ancestor cap is `start.row.saturating_add(input.max_ancestor_lines)` (`:273`) and the fallback uses `context_lines` whole rows (`:283-291`); the rust excerpt in the Zed section is from the old checkout |
| `MAX_SEARCH_RESULT_FILES` and `MAX_SEARCH_RESULT_RANGES` at `project_search.rs` lines 161 and 162; the test module from line 393 | lines 166 and 167; the test module starts at line 410. The search is started with 5,001 files (`project.rs:4975`), the checks use `>`, and the file cut gives no marker |
| Zed "searches the live document model, not the filesystem"; `aho_corasick` does "literal multi-pattern search" | open buffers come from memory, every other file is read from disk by a prefilter; grep always builds `SearchQuery::regex`, never the plain-text variant that holds the `AhoCorasick` searcher |
| `fancy_regex` "can hang on a pathological pattern" | bounded by a 1,000,000 backtrack limit; hitting it returns an error and Zed silently drops that file's later matches. Also missing from the old text: the `regex` parameter description names the Rust `regex` crate, which is not the engine |
| the fence length "chosen by `MarkdownCodeBlock`" in the snippet | only the UI content block uses it; the string the model receives has a fixed three-backtick fence (`grep_tool.rs:341-343`) |
| binary handling, encoding repair "n/a (buffers)" | unopened files go through a 1,024-byte classifier and are skipped silently; BOM, UTF-16 and `chardetng` decoding apply (`project_search.rs:832-905`, `file_content.rs:239-310`) |
| sort "buffer order" | worktree id, then path component order, then match offset |
| the rejected idea was only "twenty matches per page with no total" | it also misses the offset drift: `offset` skips raw ranges and the page counts merged blocks |

| Changed between `d9ad6aff67` and `37faef81f7` | What it means for the model |
|---|---|
| the engine's prefilter streams 64 KiB blocks for single-line plain-text queries; regex queries still use `read_to_string` | nothing for grep, which always builds the regex variant, so each candidate file is still read whole into a `String` |
| disk files now decode through the new `file_content` crate (`DecodingReader`) instead of `decode_file_text` into a `Cursor` | none visible |
| the prefilter's line or offset hint is no longer passed to the full scan (`MatchPositionHint::default()`, `project_search.rs:880`) | the scan starts at byte 0, which removes a case where look-behind or an anchor saw a truncated start |
| open buffers now run through `detect` on their rope (`:933-941`); entry-less file buffers merge in path order; worktrees sort by id | a rare change in result order |
| ACP `schema::v1` to `schema::v2`; the tool-call location line is now `start.row + 1` | UI only |
| `saturating_add` for the ancestor and context rows | `u32::MAX` is safe |

Caption: what changed in Zed's grep between the two checkouts, and what each change does to the text the model reads.

## What Runvil can support today

### Operation-level gap

| Behavior the report specifies | `Access` method | Status | What the gap needs |
|---|---|---|---|
| read a file's bytes for matching | `read` | **covered** | returns `Vec<u8>`, so a byte matcher works and a UTF-8 lossy decode is the caller's choice |
| read a file as text for matching | `read_string` | **covered** | returns `String`, errors `VfsError::NotUtf8 { path, valid_up_to }` on invalid UTF-8 |
| enumerate every file under a root | `glob` | **covered** | `glob("**/*")` after canonicalization; results are sorted and relative |
| enumerate a directory's children | `list` | **covered** | returns `Vec<Entry>` with `name`, `stat`, `description` |
| get a file's mtime for the newest-first sort | `stat` | **covered** | `Stat.modified: Option<SystemTime>`, explicitly optional so a backend that does not track it can say so |
| get a file's size for a budget | `stat` | **covered** | `Stat.size: u64` |
| search file contents | **none** | **missing** | this is D1 |
| stream matches without buffering them all | **none** | **missing** | `Access` returns `Result<T, VfsError>` by value; there is no streaming shape in the trait |
| take one claim covering a search | `Claims::claim_subtree` via `remove(recursive: true)` | **partial** | `claim_subtree` exists but is only reachable through `remove`; a search needs it as its own operation |
| honor an ignore file | **none** | **missing** | no concept of `.gitignore` or `.cursorignore` anywhere in `Access`. Rig shows what has to be decided: its `ignore`-crate walk applies `.gitignore` only inside a git work tree, reads `.ignore` and parents' ignore files, drops `target` and dot-names by name, and lets a glob that matches an ignored file or directory bring it back (probe, `workspace.rs:486-510`) |
| report a binary file was skipped | **none** | **missing** | no binary detection in the VFS |
| rewrite an invalid regex into a model-facing message | **none** | **missing** | no regex engine in the VFS, so no parse errors to translate |
| cap stdout from a child process | **none** | **n/a** | there is no child process; this whole class of limit disappears |

**Five of the tool's caps exist only because ripgrep is a child process**: the 8 MiB stdout cap, the line budget in `executeRg`, the `--max-columns` cut, the `--encoding` flag, and the `--sortr` fallback. A VFS search has none of them, because there is no pipe to fill and no argv to build. So the native version of this tool is substantially *smaller* than Cursor's. The complexity is concentrated in argv, which you do not need.

### Behavioral gap

| Behavior | Covered by | Status | What the gap needs |
|---|---|---|---|
| global `head_limit` across files | nothing | **pack work** | a counter in the Pack's own loop; no VFS change |
| `offset` skipping matched entries globally | nothing | **pack work** | same |
| context lines attached to their match | nothing | **pack work** | the Pack reads N lines before and after a match; the VFS gives no line-addressed read |
| the `-A`/`-B`/`-C` semantics | nothing | **pack work** | pure Pack logic |
| per-file line numbers | `read_string` | **covered** | split on `\n`, count; the Pack does this |
| the gap separator and the blank line between files | nothing | **pack work** | rendering only |
| the `<workspace_result>` envelope | nothing | **pack work** | rendering only |
| the truncation trailer | nothing | **pack work** | rendering only, but needs `total` and `shown`, which means the Pack must know the true total after the cap |
| newest-first mtime sort | `stat` | **covered** | `Stat.modified` is `Option<SystemTime>`, so a memory backend can report `None` and the Pack can sort those last, which matches Cursor's undefined-last rule |
| the 2000 ms stat budget | nothing | **n/a or pack work** | a VFS `stat` has no I/O latency, so the budget is meaningless; a memory backend's stat is a hash lookup |
| path rewriting to workspace-relative | `glob` already returns relative | **covered** | `glob_pattern` strips the root with `strip_root` |
| path resolution for `..`, `~`, `file://` | `glob` validates and canonicalizes | **partial** | `Access::glob` refuses a backslash and a control character in the raw pattern (`validate_glob_pattern`), which `Grep`'s `path` argument should not do |
| refusing a path outside the workspace | policy layer | **partial** | `gate(Op::Read, path)` can refuse, so this is a policy question, not a capability one |
| searching a file the caller cannot read | policy layer | **covered** | `gate` runs before the backend, and `VfsError::PermissionDenied { path, reason }` is the shape |
| reporting an unreadable file as skipped | **none** | **missing** | `read_string` returns an error the Pack must catch and count; there is no partial-success shape |
| a result cache keyed on the search | nothing | **pack work** | pure Pack state |
| deterministic output order | `glob` returns sorted paths | **covered** | sorting by path is free if you use `glob`'s order and do not re-sort by mtime |
| a claim covering the search | `Claims::claim_subtree` | **partial** | see D1 |
| a search event to observers | `fire(Op::...)` | **missing** | there is no `Op::Search`; the closest is `Op::Read`, which would fire per file |
| binary detection | nothing | **pack work** | a NUL scan over the bytes from `read`, done in the Pack |
| invalid-UTF-8 handling | `read_string` | **partial** | the VFS *errors* where Cursor *lossy-decodes*; the Pack can use `read` and `String::from_utf8_lossy` instead, which matches Cursor's behavior more closely. Rig takes a third path: `String::from_utf8(bytes).ok()` and skip the whole file with no trace, so a Latin-1 file's ASCII lines are lost (source, `search.rs:310-312`; probe). Zed decodes a BOM, UTF-16 and `chardetng` input before matching (source) |
| a line-addressed read for context | nothing | **missing** | `read_string` gives the whole file; the Pack splits and slices, which is fine in memory and expensive over a real backend |

### What his crate already has that helps

Three things in the existing VFS map onto Grep better than I expected.

**`Access::glob` returns sorted paths.** From the doc comment: "Returns the sorted paths of the files, or of the directories, that match `pattern`." So a Pack search that uses `glob` gets determinism for free, which is the D8 recommendation. No extra work.

**`Stat.modified` is `Option<SystemTime>`**, with an explicit note: "A backend fills an optional field only with a value it tracks, and leaves it `None` otherwise, so that a reported time stays deterministic and a sort by time, such as `ls -t`, stays meaningful." That is the shape the undefined-last mtime rule needs, and the comment shows the design intent already anticipated this use.

**`claim_glob` claims the pattern.** From `claims_tree.rs`: "The pattern itself is the claim, not each match." That is the precedent for a `claim_search`, and it suggests the design pattern: a search claims the thing that describes it, not the things it finds.

The tension is that `claim_glob` conflicts with writes to *matching paths*, which it can enumerate from the table. A search cannot enumerate what it will match without doing the search. So a `claim_search` needs either the subtree (D1 option 2) or a different conflict rule.

### Rust mechanisms the other harnesses show

Three of the harnesses are Rust and run their search in process, which is the case Runvil is in.

| Need | Where it is done | Crate or type | Fit for `Access` |
|---|---|---|---|
| a linear-time regex with a size limit and clear errors | rig `find`, zeroclaw's internal backend | `regex::Regex`, `RegexBuilder` (`regex` 1.12.3) | direct; the pattern compiles before any walk, a program over 10,485,760 bytes is refused, and look-around and back-references fail with named messages (D11) |
| an ignore-aware walk with glob overrides | rig `Local::files`, `workspace.rs:486-510` | `ignore::WalkBuilder`, `OverrideBuilder`; `globset` | a real backend only; the VFS needs its own rule for `.gitignore` outside a git tree and for override precedence |
| a search as a pure function over two traits | rig `find` and `render` | `Workspace::files`, `Workspace::read_files` (a batch with a per-file default), `Found`, `FileRead` | the same shape as `Access::glob` plus `read`, with a batch read sized in bytes (D1 option C) |
| typed facts beside the text | rig `SearchHits`, `FilesRead` | a struct published next to the string | a `runvil_plugin::ToolOutput` question, not a VFS one |
| a prefilter over the whole file before the line loop | rig `whole_text`, `multi_line(true).crlf(true)` | `Regex` | an optional speedup; off for `\A`, `\z`, a `-` flag or a lone `\r`, and a fuzz of 400,000 patterns found no case where it changed the hits |
| a sorted walk with a deadline per entry, NUL and non-UTF-8 skipped | zeroclaw internal backend, `content_search.rs:444-738` | `glob::Pattern`, `regex::RegexBuilder`, `read_to_string` | the shape to copy for an in-process search, with the skip count added |
| a binary classifier and a streaming decode | Zed `crates/file_content` | `DecodingReader`, `detect_encoding`, `chardetng`, `encoding_rs` | would settle D9 and the UTF-16 gap; needs crates the VFS does not have |
| ordered results from parallel workers | Zed `maintain_sorted_search_results` | per-item one-shot receivers consumed in submission order | needed only if the search is parallel |
| look-around and back-references | Zed `search.rs` | `fancy_regex::RegexBuilder` | not recommended (D11) |

Caption: borrowable mechanisms, with the crate or type each rests on and how it fits `Access`. `(source, probe)`

### What a minimal native search would look like

If you take D1 option A, the smallest shape that covers Grep and stays consistent with the existing code:

```rust
/// One match line.
pub struct SearchMatch {
    /// The path the match is in, in the same form `glob` returns.
    pub path: String,
    /// The 1-based line number.
    pub line: u64,
    /// The line's text, without its terminator.
    pub text: String,
    /// Whether this is context rather than a match.
    pub is_context: bool,
}

/// A search request.
pub struct SearchRequest {
    /// The pattern, already compiled by the caller.
    pub pattern: ...,
    /// The root to search under, canonical.
    pub root: String,
    /// Lines of leading context to include.
    pub before: u32,
    /// Lines of trailing context to include.
    pub after: u32,
    /// Matches to skip before collecting.
    pub offset: u64,
    /// Matches to collect, or `None` for no client limit.
    pub limit: Option<u64>,
    /// Hard ceiling on entries examined, so a hostile tree cannot run forever.
    pub budget: u64,
}
```

and the `Access` method would follow the existing shape, which is gate, admit a claim, fire, delegate:

```rust
pub fn search(&self, request: &SearchRequest) -> Result<SearchMatches, VfsError> {
    let root = self.gate(Op::Search, &request.root)?;
    self.admit(Claims::claim_subtree, &root)?;
    self.fire(Op::Search, &root);
    self.inner().search(&root, request)
}
```

That is four lines of plumbing and one new backend method. The `SearchMatches` return type carries the collected entries plus `total: u64` and `truncated: bool`, so the Pack can build the trailer without re-scanning.

The one thing this shape does not give you is streaming, and if you want the bounded-memory property from my measurements you need either a callback or an iterator. A callback is easier to fit into the existing `Result<T, VfsError>` shape:

```rust
pub fn search_each<F>(&self, request: &SearchRequest, on_match: F) -> Result<SearchSummary, VfsError>
where F: FnMut(&SearchMatch) -> ControlFlow<()>;
```

That gives the backend a way to stop early and the Pack a way to cap memory at 2000 entries, with `SearchSummary { total, truncated, skipped }` as the return.

## What remains unknowable

Six things the artifacts cannot settle. Each is a decision a new implementation has to make for itself.

**1. Whether the indexed path ever serves a live Grep call.** `executeIndexedGrepWithTimeout` runs before ripgrep whenever `this.grepProvider.executeIndexedGrep` exists, with a 25 s timeout and a silent fallback. Whether that provider is present, what it returns, and whether its ignore rules match ripgrep's are all runtime properties of the Cursor extension host. The provider's `executeIndexedGrep` maps `glob` to `includes: [glob]` and ignores `head_limit` and `offset` entirely, so if it answers, the *result set* could differ from ripgrep's on the same tree. Static reading cannot tell you whether it answers in a normal session. A live probe with two identical searches, one with the index warm and one cold, and a diff of the results, would settle it.

**2. What `decodeLinesAsRead`'s encoding detection decides.** The function calls a `detectEncoding` helper once per affected file inside a 2000 ms budget, then re-slices the original bytes through iconv. The helper lives outside this bundle. So the repair path's behavior for a Latin-1 file, a Shift-JIS file, and a UTF-16 file without a BOM is not determinable from what I read. What is determinable: there is a 2000 ms budget across all affected files, an undetectable file is left with its U+FFFD, and the splice replaces only the content bytes and leaves the path and the line number alone.

**3. Whether `activeEditorResult` is ever populated.** The field exists in `GrepSuccess`, and `rIe` paginates it (`activeEditorResult:n.activeEditorResult?s(n.activeEditorResult):void 0`), and the renderer's `Object.entries(t.workspaceResults)` ignores it. So it is paginated and never rendered, which means the `(unsaved)` claim in the `dsv3-1018` description cannot be satisfied through this renderer even if a producer fills the field. Either the producer and renderer are elsewhere, or the description is aspirational. I lean toward aspirational, but the pagination code exists, so something intends to use it.

**4. What `streamingEnabled` returns in a live session.** `const r=this.useStreamingGrep; if(undefined===r) return true;` so the default is on, but the option is injected by whoever constructs `LocalGrepExecutor`, which is outside the bundle. If `useStreamingGrep` returns false, the sort path changes: ripgrep gets `--sortr modified` for a relative target and there is no 2000 ms budget. So the determinism answer in Q11 depends on a flag I cannot read. The default being `true` makes the client-sort path the likely one, and my measurements assume it.

**5. What the `2000 ms` stat budget produces on a 200,000-file tree.** I extrapolated from 20,000 files at 763 ms. The extrapolation assumes linear scaling, which holds for a cold cache and fails for a warm one, and it assumes the `Promise.all` over 200,000 stats does not hit a handle limit first. So "the mtime sort gives up above roughly 50,000 files" is a model, not a measurement. The direction is certain (the budget is fixed and the work scales) and the exact threshold is not.

**6. Whether the `Results truncated. ` prefix appears for a plain `head_limit` cut in a live call.** By the code it does **not**, because `cutBeforeEnd` requires `reachedEnd`, and a full `head_limit` page has `returned === headLimitApplied`, so `reachedEnd` is false and `cutBeforeEnd` is false. The only signal of the cut is `at least` in the total. The parent spec implies the opposite in one place and marks its example `(derived)`. I trust the code and I have run the derivation through a port, and the port produces no prefix. A live call should still be made before a byte-exact test is frozen, because this is the one trailer case where a plausible reading of the code and the spec's example disagree.

A seventh, narrower unknown: **what a `path` argument pointing at a single file does to the output shape.** `searchTarget` returns the file path, ripgrep searches one file, and the result is a content result with one `file` key. The `--encoding` branch fires only for a single file. But whether Cursor suppresses the path line in that case, or renders it as it does for a directory search, I could not determine without a live call, because the renderer does not branch on file-versus-directory. I believe it renders the path line the same way, which is a slightly odd output for a single-file search, but I did not confirm it.

**From the other harnesses, still unverified.**

- Zed's pagination drift is a hand trace of `grep_tool.rs:299-305` and `:382`, not a run; no test in the file sets `offset` above 0. Whether a hit that opens a buffer attaches language servers or other listeners, what `include_pattern: ""` does, and how `encoding_rs` repairs malformed UTF-16 are not in the sources. I read `fancy-regex` `main` for the backtrack limit and not the 0.19.0 tag that the lockfile pins, and I ran no Zed test.
- Pi: where `featureFlags.enablePiTools` is set and which model profiles get the pi tools, whether Cursor's zod layer coerces `"5"`, which ripgrep binary the daemon uses in production and its working directory are not in the bundle or the daemon. The combined three-notice trailer is assembled from the code and was not triggered.
- Rig: output order on ext4 or APFS; symlink behavior on the probe machine (creating them failed with error 1314); the remote script and `Hosts` with a live transport (only the `git ls-files` command and `Override::matched` were probed); SHA-256 cost per matching file. Rig was not built or run: every rig probe is a verbatim copy of its functions compiled against the locked crate versions.
- Zeroclaw: the GNU `grep` backend and the internal backend were not run, and ripgrep on a BOM file, an empty `include` and `\r\n` were not tested; upstream commits after `2d85371f3` were not fetched.
- Gemini CLI: ripgrep's directory-walk behavior with a NUL file under Gemini's flags, and ordering variance with 4 threads, were not measured; the 10 percent turn saving credited to auto-context is a source comment.
- OpenCode: how an Effect defect is worded to the model, and permission-rejection text, are not in the sources; the non-UTF-8 failure is derived from the adapter's schema and was not run.
- Codex CLI: whether ripgrep's exit 2 keeps partial matches and whether a dropped `output()` future kills the child (the removed handler has no `kill_on_drop`) are not shown; no release number exists for the pinned commit.
- Cline: the column byte offset on non-ASCII lines on the ripgrep path is source inference and was not probed.

## Sources

**Tier: bundle.** `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js`, 9,814,287 bytes, read as latin1.

| Claim area | Offsets |
|---|---|
| registration, name switch, description switch, parameter selection | 4315448 to 4319512 |
| schema `wIe` | 4311435 |
| schema `vIe` | 4313434 |
| broad-glob set `uIe` and invalid-args builder `dIe` | 4310786, 4310816 |
| numeric and boolean coercion `fIe`, `gIe`, `yIe`, `hIe`, `mIe` | 4311134 to 4311393 |
| semantic-search reminder wiring | 4322020 |
| result cache `VEe` and key `KEe` | 4300579, 4300217 |
| char budget `YEe` | 4301037 |
| footer `QEe` and its four helpers | 4302018 |
| no-result `XEe` | 4303234 |
| mode renderer `ZEe` | 4303547 |
| envelope and multi-workspace budget `tIe` | 4306800 |
| pagination `rIe`, `nIe`, `sIe`, `oIe`, `iIe` | 4307544 to 4309184 |
| error classifier `IEe`, errno set `xEe` | 4286956, 4286847 |
| executor class `Lx` and static constants | 9157073, 9173743 |
| `buildArgs` | 9158354 |
| `searchTarget`, `resolveSearchTargets`, `findOutsideTargets`, `isRootIndependentSearchPath`, `computeRelativeTarget` | 9158071 to 9158334 |
| `sortsAfterRipgrep` | 9158239 |
| `filterBlockedFiles`, `isPostRipgrepBlocked`, `throwOnRipgrepFailure` | 9159943, 9160358 |
| `orderNewestFirst`, `keepNewestPage` | 9160807, 9161095 |
| `runFilesMode` | 9161199 |
| `runCountMode` | 9162088 |
| `runContentMode` | 9163038 |
| `decodeLinesAsRead` | 9163435 |
| `streamingEnabled` | 9165908 |
| `executeIndexedGrepWithTimeout` | 9166399 |
| `executeRg`, the 8388608 byte cap | 9167020, 9167113 |
| `execute` | 9169464 |
| content parser `Ax`, files `Cx`, count `Px` | 9153344, 9152226, 9152663 |
| `decodeRipgrepOutput` `Rx`, `isUtf16Encoding` `Mx` | 9154521, 9154690 |
| char-code constants, line iterator `xv`, byte-splice `Ev` | 9005941 to 9006600 |
| static classifier tool array | 4671940 |
| `pi_grep` registration, schema `ZRe` | 4561631, 4554895 |
| `LocalPiGrepExecutor` | 9202537 |
| ripwalk ignore-scanner argv (the only `--no-ignore-vcs`) | 8619100 |

**Tier: daemon.** `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, unminified with original identifiers and source paths.

| Real name | Purpose | Offset |
|---|---|---|
| `LocalGrepExecutor` | the shared executor class | 16776285 module start, class at 16780500 region |
| `buildArgs` | argv construction, with real names | 16780899 |
| `searchTarget`, `computeRelativeTarget`, `isRootIndependentSearchPath`, `resolveSearchTargets`, `findOutsideTargets`, `sortsAfterRipgrep` | path handling | 16780450 to 16780866 |
| `parseContentOutput` / `parseRipgrepContentOutput` | the NUL parser | 16772229, 16207202 |
| `parseRipgrepFilesOutput`, `parseRipgrepCountOutput` | the line parsers | 16210273, 16209015 |
| `locateContent`, `decodeReplacementLines` | the byte-splice path | 16205000 region |
| `isDisallowedBroadGrepGlob`, `createBroadGrepGlobError`, `DISALLOWED_BROAD_GREP_GLOBS` | the guard, with the set literal | 12744472, 12746183, 12748384 |
| `describePosition`, `buildResultFooter`, `describeShown`, `describeTotal`, `describeNextStep` | the footer, unminified | 12732000 region |
| `formatContentResult`, `formatFilesResult`, `formatCountResult`, `describeEmptyPage` | the three mode renderers | 12734000 to 12739000 |
| `renderMultiWorkspace`, `GREP_CHAR_BUDGET` | the envelope and the 15000 budget | 12738232, 12739142 |
| `applyGrepPagination`, `paginateContent`, `paginateFiles`, `paginateCounts`, `paginateArray` | pagination | 12741685 to 12743790 |
| `isGlobFileListing` | the `--files` fork predicate | 16770900 region |
| `hasOnlyUnsearchableEntriesBelowTarget` | the permission tolerance test | 16770900 region |
| `startDeadline`, `hasEntries`, `stoppedFolderResult` | deadline handling | 16771000 region |
| `deduplicateCrossWorkspaceResults`, `fileBelongsToCurrentWorkspace`, `findOwningWorkspaceRoot` | the multi-root dedupe | 16768300 region |
| `RipgrepTimeoutError`, `GREP_CLIENT_LIMIT_LINES` | constants | 16779000 region |
| `statMtimeNs`, `comparePathSegments`, `orderByModifiedDesc`, `sortNewestFirst`, `SORT_STAT_TIMEOUT_MS` | the mtime sort | 16643414 module |
| `truncateLinesToCharBudget`, `cutLine` | the char budget | 2161810, 2161200 |
| `decodeRipgrepOutput`, `ripgrepEncodingForFile`, `isUtf16Encoding` | encoding | 16774000 region |

**Module paths recovered from the daemon**, which name the source files these came from:

```
../local-exec/dist/grep.js
../local-exec/dist/grep-output-parse.js
../local-exec/dist/grep-line-encoding.js
../local-exec/dist/newest-first.js
../local-exec/dist/int32.js
../agent/dist/tools/core/grep/grep.js
../agent/dist/tools/core/grep/formatters.js
../agent/dist/tools/core/grep/pagination.js
../agent/dist/tools/core/grep/parser.js       (inferred from the pagination/formatters naming)
../utils/dist/result-formatting.js
```

**Tier: probe.** Measurements I ran on this machine.

| What | How |
|---|---|
| `rg` version | `rg.exe --version` → `ripgrep 15.1.0-cursor5 (rev c3e3c2f7ec)` |
| NUL wire format | raw stdout capture, byte-inspected, on paths including `weird:name.txt` |
| `-C` alone vs with `-A` | argv replay of `buildArgs` output |
| binary file behavior per mode | fixtures with a NUL, three modes |
| `--max-columns` byte boundary | bisected at 1-byte and 8-character widths, 4 character widths |
| preview character count | decoded output, counted codepoints before the marker |
| long match not in preview | 2000-char line with the needle at 1001 |
| invalid regex stderr shape | `a(b` and `(a)\1` |
| case sensitivity | `--case-sensitive` vs `--ignore-case` vs no flag vs `--smart-case` |
| ignore bypass by `--iglob` | fixture with `.gitignore`, five files, three glob variants |
| BOM consumption | UTF-8 BOM file, plus a pattern matching the BOM |
| UTF-16 | BOM'd UTF-16LE file, with and without `--encoding` |
| lone-CR line endings | `\r`-only fixture |
| context group merging | `m[13]` with `--after-context 3` on five dense lines |
| search cost | 20,000 files, 200 directories, 66.8 MB, six strategies, three repeats each |
| retention cost | 1.2M matches retained vs 2000 retained, RSS delta |
| trailer text | a line-for-line port of `describePosition`/`buildResultFooter` run over 10 cases |
| rendered body | a line-for-line port of `formatContentResult`/`formatFilesResult` plus `truncateLinesToCharBudget` |

The ports are my own reimplementations of the daemon's functions in JavaScript, which means they are only as good as my reading. Where the report quotes a footer or a body, it came from the port and was never produced by a live Cursor call. The port was written from the unminified daemon rather than the minified bundle, to reduce the chance of a misread.

**Tier: probe, from the harness comparison.** Each ran against the code named, with ripgrep 15.1.0-cursor5 unless stated.

| What | How |
|---|---|
| pi against Cursor's `pi_grep` | pi's `grep.ts`, `truncate.ts`, `path-utils.ts` and `paths.ts` copied with only the imports rewritten, and the `LocalPiGrepExecutor` class sliced from the daemon (16851630) with daemon internals stubbed; cases covering CRLF, lone CR, BOM, UTF-16, Latin-1, binary, limits, globs, `RIPGREP_CONFIG_PATH` and a 20,000-file tree run 8 times |
| rig `search` | seven single-file Rust programs that `include!` verbatim copies of `find`, `whole_text`, `render`, `clip` and `Local::files`, compiled with `rustc` 1.97.1 against hand-built rlibs of the locked `regex` 1.12.3, `ignore` 0.4.25, `globset` 0.4.18 and their dependencies; a 400,000-pattern fuzz of the prefilter; timing on 1,000 files of 200 lines |
| zeroclaw `content_search` | a Rust program including the verbatim formatter and the `glob` 0.3.4 crate, run against the bundled ripgrep with the tool's flags on a 300-file tree, 12 times |
| Gemini CLI `grep_search` | `formatGrepResults` run on hand-built matches, and the bundled ripgrep for the lookahead and binary cases |
| OpenCode and Cline | ripgrep `--json` fed through verbatim copies of the output loops; four and six repeated runs for order |

**Tier: bundle, extracted slices.** The `GREP` registration prettified from the Cursor 3.27.12 `main.js` at the byte range its header names; context dumps and searches over that same file; the shared raw listing of the `LocalGrepExecutor` that Glob also uses; and the briefing that maps the mangled identifiers to the daemon's unminified names. Every offset above is from `main.js` itself.

**Tier: source.** The harnesses, each at the commit or version it was read.

- **Claude Code:** `claude.exe` 2.1.311 (win32-x64, Bun-compiled), `tools/grep.js` 28 KB, `limits.js`, `input-schemas.md` lines 55 to 74.
- **Copilot:** `extensions/copilot/src/extension/tools/node/findTextInFilesTool.tsx` (33,928 bytes) in `microsoft/vscode` at `b208bc25` (2026-10-25); `node/test/findTextInFilesTool.spec.tsx`; `node/test/findTextInFilesResult.spec.tsx`; `common/toolNames.ts` line 25.
- **Zed:** the Zed repository at `37faef81f7` (2026-10-27): `crates/agent/src/tools/grep_tool.rs` (57,868 bytes, tests from line 410), `crates/agent/src/thread.rs`, `crates/project/src/search.rs`, `project_search.rs` (limits at lines 166 and 167), `project.rs:4947-4987`, `crates/file_content/src/file_content.rs`, `crates/worktree/src/worktree.rs`. The settings UI is cited at `crates/settings_ui/src/pages/tool_permissions_setup.rs:1405` and `:1415`, and the engine tests are in the `project` crate. The lockfile pins `fancy-regex` 0.19.0. The first reading was `d9ad6aff67` (2026-08-22).
- **pi:** the pi repository at `a92283056` (2026-10-27), package `@earendil-works/pi-coding-agent` 1.4.0: `packages/coding-agent/src/core/tools/grep.ts` (325 lines), `truncate.ts`, `renderers/grep.ts`, `path-utils.ts`, `src/utils/paths.ts`, `src/utils/tools-manager.ts`, `test/tools.test.ts:877-968`, `packages/agent/src/agent-loop.ts`, `packages/ai/src/utils/validation.ts`; commits `3d43d2e17`, `e9ba9e2eb`, `8c4e87389`, `fecc19e4e`.
- **rig:** the rig repository at `a83b774d0` (2026-10-25): `crates/rig-tools/src/search.rs` (385 lines), `search/tests.rs`, `list.rs`, `workspace.rs`, `lib.rs`, `remote.rs`, `overlay.rs`, `plugins/rig-basics/src/coding_tools.rs`; locked `regex` 1.12.3 and `ignore` 0.4.25 from crates.io.
- **zeroclaw:** the zeroclaw repository at `2d85371f3` (2026-09-18): `crates/zeroclaw-tools/src/content_search.rs`, `wrappers.rs`, `crates/zeroclaw-runtime/src/agent/history.rs`; `glob` 0.3.4 and `tokio` 1.53.1 from the cargo registry.
- **Codex CLI:** `openai/codex` at `bea47c06dfac` (main, 2026-10-28), `core/src/tools/spec_plan.rs`, `core/src/tools/context.rs`, `models-manager/models.json`; the removed `grep_files.rs` and its tests at `32c4993c8a` (removed by `178c3b15b4`, PR 15775).
- **Gemini CLI:** `google-gemini/gemini-cli` tag `v0.66.0`, commit `970413207732`: `packages/core/src/tools/ripGrep.ts`, `grep.ts`, `grep-utils.ts`, `constants.ts`, `packages/core/src/scheduler/tool-executor.ts`.
- **OpenCode:** `anomalyco/opencode` tag `v1.19.5`, commit `810c0eba1a73`, and `dev` at `fa1260d6ca`: `packages/opencode/src/tool/grep.ts` and `grep.txt`, `packages/core/src/ripgrep.ts`, `packages/core/src/tool/grep.ts`, `packages/core/src/tool/plugin/grep.ts`.
- **Cline:** `cline/cline` tag `v4.1.26`, commit `a459f1d`: `sdk/packages/core/src/extensions/tools/executors/search.ts`, `schemas.ts`, `definitions.ts`, `message-builder.ts`; the legacy `search_files` at tag `v4.0.12`, commit `f267bf4`.
- **Aider:** `Aider-AI/aider` at `5dc9490bb35f` (2026-05-22): `aider/repomap.py`, `coders/base_coder.py`, `coders/base_prompts.py`, `watch.py`.
- **Goose:** `aaif-goose/goose` at `ee8500d5e060` (2026-10-28), release `v1.56.0` (`9949cf9`): `platform_extensions/developer/shell.rs` and `mod.rs`, `platform_extensions/analyze/`.

**Tier: web.** The parent spec's companion note of 2026-10-27 on public corroboration of Cursor's tools records the public sources and their conflicts, and its findings are consistent with mine on every point I checked: no public documentation of Grep's schema, the "Instant Grep" marketing claim, and the widely repeated 100-result figure that belongs to other harnesses. I did not find a public source that documents the NUL wire format, the broad-glob guard, the `-C` drop, or the `head_limit: 0` coercion, so all four are bundle-only findings. For the harness comparison: the `fancy-regex` `main` branch `lib.rs` (`backtrack_limit` and the iterator's error behavior), read 2026-10-28; `https://api.github.com/repos/openai/codex/commits/main` and the commit message of PR 15775; `https://github.com/aaif-goose/goose/issues/11835` and PR 13144; `https://geminicli.com/docs/tools/file-system/`; `https://opencode.ai/docs/tools/`; `https://docs.cline.bot/tools-reference/all-cline-tools`.

## Conformance checklist

The parent spec's twenty items for Grep, each confirmed or corrected with a reason.

| # | Item | Verdict | Reason |
|---|---|---|---|
| 1 | The hyphenated keys `-A`, `-B`, `-C`, and `-i` are literal JSON property names and are not normalized | **confirmed** | Both schemas use quoted hyphenated keys; the executor reads them as `r["-A"]`, `r["-i"]` with no normalization step |
| 2 | No `target_directory`, `-n`, `case_insensitive`, or `--no-ignore` parameter exists | **confirmed** | The field sets in `wIe` and `vIe` are exactly the 12 in this report. `case_insensitive` is only the proto field name |
| 3 | `output_mode` defaults to content mode, and the case and multiline flags default to false | **confirmed** | `.default("content")`, `.default(!1)` twice, plus `t.outputMode??"content"` in the executor |
| 4 | The tool is named `grep` on `dsv3-1018`, `rg` on the two Codex branches, and `Grep` otherwise, and `grep_search` is never a name | **confirmed** | The switch is exhaustive with a throwing default; the four `grep_search` occurrences are proto descriptors |
| 5 | Only the exact empty pattern is rejected, so a single space passes through | **confirmed** | `if(""===r.pattern)` is a strict equality; `" "` reaches ripgrep and matched every line of a file containing spaces in my probe |
| 6 | A negative context value is rejected on the raw pre-clamp value rather than silently clamped | **confirmed** | The check runs at step 4 on `r[e]` before `fIe` is applied at step 8 |
| 7 | The broad-glob guard tests a trimmed value against exactly the two broad patterns and is gated behind a feature flag | **confirmed** | `uIe=new Set(["**/*","**/**"])` at 4310786, `.trim()` before the test, `s` from `featureFlags.enableGrepBroadGlobGuard` |
| 8 | `head_limit` is ceiled and `offset` is floored | **confirmed** | `fIe(r.head_limit, Math.ceil)` and `fIe(r.offset, Math.floor)` |
| 9 | A `head_limit` of 0 means unlimited, implemented as a falsy coercion to undefined | **confirmed** | `\|\| void 0` on the result of `fIe` |
| 10 | There is no default `head_limit` | **confirmed** | No `.default()` on the field; `null` from `fIe` when absent |
| 11 | `head_limit` is applied client-side after parsing and never becomes a max-count flag | **confirmed** | Zero occurrences of `max-count` in the entire 9.8 MB bundle; `sIe` slices after `Ax` |
| 12 | `offset` and `head_limit` count globally across all files, not per file | **confirmed** | Both counters are declared outside the file loop in `sIe`; `nIe` slices the flattened array |
| 13 | The glob filter maps to the case-insensitive glob flag rather than the case-sensitive one | **confirmed** | `s.push("--iglob", t.glob)`; `--glob` never appears in this builder |
| 14 | Symmetric context is expanded into both before and after only when neither one-sided flag is present, and dropped entirely when one is | **confirmed** | The guard is `void 0!==t.context && void 0===t.contextBefore && void 0===t.contextAfter` |
| 15 | The case-insensitive mode is implemented by omitting the case-sensitivity flag rather than adding an ignore-case flag | **corrected** | The code is `!0===t.caseInsensitive ? s.push("--ignore-case") : s.push("--case-sensitive")`, so exactly one is always emitted. The spec's parenthetical, that ripgrep defaults to case-insensitive when the pattern has no uppercase, is also wrong: ripgrep's default is case-sensitive and smart-case is opt-in. I confirmed all four combinations against the shipped binary |
| 16 | Context lines are buffered and flushed only when the match they precede is emitted, and a file with every match skipped produces no entry | **confirmed** | The `pendingContext` buffer plus the `lines2.length > 0` push guard, in both `sIe` and its daemon twin `paginateContent` |
| 17 | Content mode caps at 2000 entries or 4000 with context, while files and count modes cap at 2000 files | **corrected** | The 2000/4000 figure is the **ripgrep line budget** passed to `executeRg`, not the entry cap. The parser's entry cap is 2000 in both cases (`Ax(m, CLIENT_LIMIT_LINES, d, ...)`), and `d` is the line budget. So content mode keeps at most 2000 entries whether or not context is present, and the 4000 only widens the input window. Files and count keep 2000, confirmed |
| 18 | Files sort newest first with lines inside a file in ascending order, and lines from different files are never interleaved | **confirmed, with a caveat** | The comparator is mtime descending, ties by path segments, undefined last. Intra-file order is ripgrep's scan order, which is ascending. The caveat: on a stat timeout the whole file list keeps ripgrep's walk order, so newest-first is conditional, not guaranteed |
| 19 | A match line is two spaces, the line number, a colon, then content, while a context line uses a hyphen instead of the colon | **confirmed** | `s.push(`  ${r.lineNumber}${t}${r.content}`)` with `t = r.isContextLine ? "-" : ":"` |
| 20 | Invalid regexes are rewritten into one line with the pattern interpolated raw, and the escape hint appears only for three specific reasons | **confirmed** | `IEe` extracts `/^error: (.+)$/m`, rebuilds the one-liner, and gates the hint on `/unclosed\|unopened\|repetition operator missing expression/`. I confirmed both the raw stderr shape and the short form for `backreferences are not supported` |

So: **17 confirmed, 3 corrected.** The three corrections are items 15, 17, and 18's caveat, and the first two are behavioral differences rather than wording.

Item 17 is the most likely to mislead an implementer. The spec's cap table lists `content: 2000 entries, or 4000 when context is present`. That reads as "the result can contain 4000 entries when context is present", and it cannot. `Ax` is called with `CLIENT_LIMIT_LINES` as its second argument in both cases, so it stops pushing entries at 2000. The 4000 is the third argument in `runContentMode`'s call to `executeRg`, and it is a *line budget for the child process*: ripgrep is allowed to emit up to 4000 stdout lines because with context each match can consume up to three lines (one before, the match, one after), so 2000 matches need roughly 4000 lines of output to be delivered. The extra 2000 lines are consumed by the parser but not all kept, because the parser's own cap stops at 2000 entries.

The practical consequence: **a search with context returns fewer matches than one without**, for the same tree, because context lines eat the entry budget. That is a behavior a reimplementation should decide about deliberately. If you want "2000 matches plus context", the parser has to count matches and context separately, which Cursor does not.

*2026-10-27 - deepseek-v41-flash-1017; enriched with the other harnesses 2026-10-28 - claude-sonnet-5-5*
