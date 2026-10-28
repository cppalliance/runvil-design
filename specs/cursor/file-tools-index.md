# Cursor file tools: index to the six reports

Six reports describe the six file tools in Cursor's shipped agent, one report per tool, each written to be read alone by someone about to build a version of that tool. They are deliberately overlapping, because the tools share a formatter, a read path, a lock, an edit pipeline, and a parallel `pi_*` surface, and a reader of the Grep report still needs to know how Read numbers its lines. This file is the entry point: read it first, then go to the one or two reports you actually need. Where the six disagree with each other or with `agent-file-tools.md`, this file records the resolution, because the shipped bundle always wins over the parent spec.

The set is derived from Cursor 3.27.12 on Windows: the 9,814,287-byte minified bundle at `cursor-agent-exec/dist/main.js`, and the unminified 20 MB daemon twin at `cursor-agent-host/dist/agent-host-daemon/dist/bin/daemon.cjs`. Every report labels its claims `(bundle)`, `(daemon)`, `(probe)`, `(source)` for the three comparison harnesses on disk, or `(web)`, so you can tell a read from a measurement from an inference.

## The six reports

| Tool | Wire name | Report | Lines | The one thing to know |
|---|---|---|---|---|
| Read | `Read` (`read_file` on `dsv3-1018`, `ReadFile` on the two Codex branches, `ViewImage` in minimal harness mode) | `tool-read.md` | 1785 | The 100,000-character cap is a UTF-16 string-length test applied *after* windowing, so a two-line window into a huge file always succeeds and a full read of 100,001 characters always fails with empty content. |
| Write | `Write` (`write` on `dsv3-1018`) | `tool-write.md` | 1917 | It is truncate-in-place plus fsync, not temp-plus-rename, and its only freshness protection is a sha256 fingerprint that silently switches off above 8 MB. |
| StrReplace | `StrReplace` (`search_replace` on `dsv3-1018`) | `tool-str-replace.md` | 1833 | The tool is not offered to GPT-5-family models at all; they get `ApplyPatch` instead, so a harness that registers it unconditionally has already diverged. |
| Delete | `Delete` (`delete_file` on `dsv3-1018`) | `tool-delete.md` | 1202 | Every failure is thrown and re-serialized, the re-serialization drops the path for all but `permissionDenied`, so the model sees `Error deleting file : <message>` with an empty slot. |
| Glob | `Glob` (`glob_file_search` on `dsv3-1018`) | `tool-glob.md` | 1606 | A positive glob beats a file-level `.gitignore` rule, and `.gitignore` is inert outside a git repository because `buildArgs` never passes `--no-require-git`. |
| Grep | `Grep` (`grep` on `dsv3-1018`, `rg` on the two Codex branches) | `tool-grep.md` | 2836 | `head_limit` is a client-side counter over parsed matches, global across every file, and never becomes `rg --max-count`; omitting it means no limit at all. |

All six carry the same skeleton: The short version, Decisions you have to make, The contract, Algorithm and ordering, Engineering questions, Failure matrix, Traps, How three harnesses do this, What Runvil can support today, What remains unknowable, Conformance checklist, Sources. The two sections worth reading first in any of them are The short version, for the five or six facts that invert a common assumption, and Decisions you have to make, because that is where the report states its forks rather than describing Cursor for its own sake.

## Corrections to `agent-file-tools.md`

The parent spec is useful and wrong in at least a dozen places. Each row below is a claim it makes, what the shipped build actually does, and the report that establishes it. These are the rows a reimplementation must not inherit.

| Claim in the parent spec | What is actually true | Established by |
|---|---|---|
| Cursor's Read description advertises `cat -n` format (lines 4155, 4184) | Cursor's description says `using following format: LINE_NUMBER\|LINE_CONTENT`, and the string `cat -n` occurs zero times in the whole 9,814,287-byte bundle. The only line-number wording is that bullet, twice, at 4349631 and 4669907, and it describes no padding. | `tool-read.md` |
| Read emits an omitted-lines notice of the form `... N lines not shown ...` | True, and this is the claim most easily mis-verified: a substring search for `lines not shown` returns -1 because the phrase is built from a ternary inside a template literal. The head fires when `startLineNumber > 1`, the tail when `totalLineNumbersInFile > startLineNumber + countLines(content) - 1`, and a full-file read emits neither. The spec's checklist item 19 stands as written. | `tool-read.md` |
| `old_string: "  foo"` with `new_string: "foo"` throws identical-strings rather than de-indenting | False. The identity check compares line-ending-normalized strings only, with no trim, so those two differ, the fallback matches, and the edit de-indents successfully. | `tool-str-replace.md` |
| `-C` handling | The spec's detailed comment is right and its one-line summary is imprecise. The real rule is a fallback, not a deletion: `-C` is expanded into `--before-context` and `--after-context` only when both `contextBefore` and `contextAfter` are undefined, so `-C` alone works, `-C` with `-A` or `-B` is silently ignored, and the spec's phrase "dropped entirely" invites an implementer to delete it unconditionally. The spec also places the expansion adjacent to `-A`/`-B` in its argv listing, when the shipped code pushes it after the `--max-columns` pair. | `tool-grep.md` |
| Content-mode search passes only the documented context flags | Every content-mode search also passes `--max-columns 1000 --max-columns-preview`, which appears nowhere in the spec, the parameter list, or the tool description. The cut is decided in bytes and the preview keeps 1000 characters, so a 334-character CJK line is cut and a 999-byte line is not. A byte-exact test written against a naive implementation fails on any long line. | `tool-grep.md` |
| `-i` works by omitting `--case-sensitive`, "because ripgrep is case-insensitive by default when the pattern has no uppercase" | Wrong in both halves. The pseudocode's `IF -i is NOT set -> --case-sensitive` is right, but there is no counterpart branch for `-i` set, and the shipped code pushes `--ignore-case` explicitly: `!0===t.caseInsensitive ? s.push("--ignore-case") : s.push("--case-sensitive")`. Exactly one of the two is always present, so there is no default to inherit. The comment also describes smart-case, which ripgrep only does under `--smart-case`, not by default. | `tool-grep.md` |
| `StrReplace` is always present in the tool list | Chosen by `modelInfo.vendor`. anthropic, google, xai, zai, and unknown get `StrReplace` plus `Write`; openai and cursor with `isGpt5Family` get `ApplyPatch` *instead of* both; openai and cursor without it get `StrReplace` plus `Write`. | `tool-str-replace.md`, `tool-write.md` |
| Delete's `permissionDenied` loses the path | It keeps it. `serializeError` builds `{case: permissionDenied, value: {path: e.path, isReadonly: e.isReadonly}}`, so the render interpolates the real path. What it loses is the gate's specific reason, because `isReadonly` is a boolean. | `tool-delete.md` |
| `.gitignore` is respected by default, and gitignored files are absent because `.gitignore` wins over a positive glob | Backwards for file-level rules. `--iglob` "always overrides any other ignore logic" per ripgrep's own help, so a positive glob resurrects an ignored file. Directory-level rules still prune, because ripgrep never descends. And `.gitignore` is inert outside a git repository, because `buildArgs` never passes `--no-require-git`. | `tool-glob.md` |
| `MULTI_STR_REPLACE` is a live tool in the mutating-wait set | Two occurrences in the whole bundle, both lookup tables (`edit_tool_call` and `agent-quality`), and no `AG("MULTI_STR_REPLACE", ...)` registration. The read-retry whitelist does name `MultiStrReplace`, so anyone implementing from that set is waiting for a tool that cannot be called. | `tool-str-replace.md` |
| Line numbers render three different ways, one of them Codex `cat -n` style | Three styles exist in `fH`, but `gpt5CodexCatN` is `false` at all eight assignment sites and propagation is `!l && t.gpt5CodexCatN`, so a false cannot become true. Two styles are reachable: the default six-wide pipe, and the GPT-5 `L<n>:` form for `isGpt5Family`. | `tool-read.md` |
| The per-file lock is "held across the retry" (checklist item 6) | It is released before the retry wait and re-acquired inside the retry. Holding it across the wait deadlocks, because the wait is for mutating calls that need that same key. Item 7 of the same checklist says the correct thing, so the two items contradict each other and item 6 is the one to discard. | `tool-read.md` |
| `StrReplace` has no fuzzy matching, and the spec says so as a design fact | True of matching, misleading about the tool. A fuzzy finder does exist: when the match fails it locates the longest trimmed line of `old_string` in the file, and the model gets `Found a possible fuzzy match, did you mean:` followed by a numbered excerpt rendered through `fH`. It never changes whether the edit applies, so the spec's conclusion holds while its phrasing hides machinery a reimplementation has to decide whether to copy. | `tool-str-replace.md` |
| Glob's file-list cap is `5e3` entries | The parent spec is right here: it lists the render budget as 5,000 characters and says it binds first at roughly 160 files. The correction is only to an earlier brief that read `JEe=5e3` as an entry cap. Worth recording because the number appears in the constant table next to two genuine entry caps (2,000 returned, 10,000 stdout lines parsed), so the mistake is easy to repeat. | `tool-glob.md` |
| Read truncation has one mechanism | Three, and only one is visible. The 100,000-character cap errors with a message; the executor's 8 MiB budget silently cuts and sets a `truncated` flag nothing renders; `readWholeTextFile` refuses outright above 8 MiB and returns `undefined`. The parent spec lists the 8 MiB figure only as a streaming threshold, so it names the number without naming the silent cut or the refusal. | `tool-read.md` |
| The write path's read cannot see a truncated view | The parent spec does not address it. `StrReplace` checks `truncated` and refuses with `File is too large to edit`; `Write` contains no reference to that flag at all, so the tool that overwrites the whole file is the one that will write from a partial view and lose everything past the cut. | `tool-write.md`, `tool-str-replace.md` |

## Shared facts

Six reports restate the same infrastructure. These are the correct statements, so you can check any report against them.

| Fact | The correct statement |
|---|---|
| Line-number format | Six-wide right-aligned, then a bare pipe, then the text with no space after it: `     1|const x = 1;`. Five leading spaces, a digit, a pipe. No tab. The GPT-5 style is `L1:text`, reachable for `isGpt5Family`. Sparse mode, when `sparseLineNumbers` is an integer N, emits lines whose number is not a multiple of N with no prefix at all, and it forces both other styles off. The Codex style exists in `fH` but `gpt5CodexCatN` is unreachable in this build. |
| Read cap | 100,000, measured on JavaScript string length, which is UTF-16 code units, and applied *after* windowing. It errors rather than truncating: the record is `success` with `exceededLimit: true` and empty content, rendered with the error flag set. A byte-counting port diverges on CJK by up to 3x and on emoji by 2x, in both directions. |
| 8 MiB cut | `MAX_TEXT_SIZE = 8 * 1024 * 1024`, used as a *character* budget in the executor's slicing (`takeUtf8Window`, `appendBounded`) and as a *byte* threshold in `readWholeTextFile`'s `stats.size > MAX_TEXT_SIZE` refusal. The slice sets `truncated: true` and nothing renders it, so the cut is silent. These are two different comparisons on two different units at the same number, which is the easiest thing in the whole read path to get wrong. |
| Lock ordering | Keyed on the raw argument string, before any normalization, so two spellings of one path do not serialize. For Read's retry-on-miss the lock is released *before* the wait and re-acquired inside the retry, because the wait is for mutating calls that need the same key. Glob takes no lock at all and is absent from the mutating set. |
| Two ripgrep builders | `buildArgs` is the one Grep and Glob both use. `ripwalk` is a separate file-walking client neither tool reaches. They disagree on three defaults: `buildArgs` never passes `--no-require-git`, always passes `--follow`, and pushes `--case-sensitive` unless `caseInsensitive` is exactly true; `ripwalk` always passes `--no-require-git`, follows symlinks only when asked, and defaults `caseSensitive` to false. Do not describe "Cursor's ripgrep invocation" in the singular. |
| The pi family | A second complete surface in the same bundle: `pi_read`, `pi_bash`, `pi_edit`, `pi_write`, `pi_grep`, `pi_find`, `pi_ls`, registered from 4556823 to 4563905, each with its own proto message pair, and all seven rendering through one function that pulls a plain string out of `output`, `error`, or `reason` and returns `Pi tool returned no result.` when none is present. There is no `pi_delete`; deletion on that surface is `pi_bash` with `rm`. `pi_edit` takes an array of edits applied atomically against one snapshot, which is a better contract than `StrReplace`'s single pair, but its fuzzy path applies whole-file NFKC normalization and rewrites untouched lines, which you must not copy. |

## Open decisions

The six Decision sections, deduplicated, grouped by the part of the design they settle. Each of these is a real fork with a cost on both sides, not a question with an obvious answer.

**Read shape**

- Do successful reads carry line numbers always, never, or behind an argument? Cursor's argument is a feature-flag experiment, and when the flag is on, omitting it means no numbers at all.
- Which width and separator? The recommendation is six-wide pipe, and to fix the description rather than the output.
- Cap on bytes, UTF-16 code units, or characters, and does it error or truncate? Exact parity means UTF-16. A byte cap with a high number is defensible if the description says bytes.
- Reproduce the negative-offset asymmetry, where `offset: 11` on a ten-line file throws but `offset: -100` clamps and reads forward?
- Keep the 5% CRLF vote, or strip a trailing `\r` per line unconditionally?
- Paginate by line over a whole-file read, or add an `Access::read_range` and use it for the prefix checks?

**Edit matching**

- Does the matching live in the Pack over `read_string` plus `write`, or in the VFS as a richer anchor operation? The Pack option makes `Access::str_replace` dead weight unless it is retired or redefined.
- Keep the line-trimmed fallback at all? It is the only path that can silently swallow a line's indentation.
- Copy `replace_all`'s non-overlapping semantics, where `aa` in `aaa` replaces once and yields `Xa`, or use a real global replace?
- Keep Cursor's success sentence, or return the changed region? The sentence currently discards the one piece of information a model needs to confirm the edit landed where it meant.
- Add a read-before-write gate? Cursor deliberately has none on either editing tool.

**Search backend**

- Does the VFS gain a search capability, or does the Pack walk with `glob` plus `read_string`? This is the largest single decision in the set, because five of Grep's caps exist only because ripgrep is a child process and disappear entirely from a native implementation.
- Copy the hyphenated `-A`/`-B`/`-C`/`-i` parameter names, which are a ripgrep flag passthrough leaking into the contract, or normalize them?
- Keep the loose coercion, where `"-i": "yes"` is true, or go typed and reject?
- Clamp a negative context value or reject it? Cursor rejects the raw value before its own clamp runs, so only the rejection is visible.
- Fix the `-C` drop or reproduce it? The recommendation is a per-side override so no supplied parameter is ever silently ignored.
- Keep the broad-glob guard, widen its set beyond `**/*` and `**/**`, or drop it? It is host-gated by `enableBroadGlobGuard`, so it is off unless the host turns it on.
- Keep the five-entry result cache, which is keyed on everything except `head_limit` and `offset` and therefore re-paginates a cached full result?

**Ignore handling**

- Honor ignore files at all, and if so, where? The VFS has no ignore concept, and putting the rule parser in each Pack means Grep and Glob will drift.
- Which precedence? Cursor's is inconsistent by accident: `--cursor-ignore` in the argv, `!**/.git` as a literal glob, and a post-filter in the tool, three mechanisms in three places. The recommendation is one place, one precedence, documented.
- Reproduce the positive-glob-beats-gitignore hole for familiarity, or make ignore rules always win? Either is defensible; the report asks only that it be a choice.

**Tool selection per model**

- Gate the editing tools by vendor as Cursor does, or offer one set to every model? Cursor's own split is the strongest argument for per-model selection in the whole evidence set, because on the models it most wants to serve well, `StrReplace` is not registered at all.
- Reproduce the per-family name variants (`read_file`, `ReadFile`, `search_replace`, `glob_file_search`, `rg`), which exist for the `dsv3-1018`, `gpt5-codex`, and `codex-cloud` branches, or ship one `latest` surface?
- Adopt the pi parameter design where the primary surface is worse, without adopting the pi matcher. That means `Grep` with `literal` and a real `limit` default, `StrReplace` accepting an array of edits, and `Write` keeping `contents` because that is the name the model expects.

*2026-10-27 - DeepSeek V4.1 Flash 1017*