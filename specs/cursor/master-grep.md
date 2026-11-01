# Build Grep as a sliced, deterministic search in the Pack over `Access`, and add a VFS search only when a measured cost demands it

## Summary

I recommend building `Grep` as a search inside the `std` Pack that walks the run's filesystem with `Access::list` and `Access::read`, matches with the Rust `regex` crate, and returns results in path order. I recommend adding no search capability to the VFS yet. This overturns the evidence report's preference for a VFS search. The claim it would reuse is a write claim, so two parallel searches would conflict. The VFS crate cannot depend on `regex`. And a call that blocks cannot be stopped. The cost is measured: 4.0 s for 20,000 files on the real backend against 0.55 s for ripgrep, almost all of it containment paid on every `Access` call.

Four decisions matter most. Publish Cursor's twelve parameters with the bundle's defaults, since the shipped schema drifted in five ways, and add `literal`. Never fail on one unreadable file; count what was skipped and say why. Order by path and stop by files and bytes, never by a clock. Page by re-running with `offset` and keep no cache.

The largest risk is cost on large Windows trees, where 200 us per file puts 100,000 files near 20 s. Build the VFS search when a deployed run shows searches over 2 s. Confidence is medium overall, and high for the schema and the pattern dialect. The next step is the `grep-schema-fix` todo, then `grep-real`.

## Contents

- [Summary](#summary)
- [The recommendation in brief](#the-recommendation-in-brief)
- [Criteria](#criteria)
- [Options and the do-nothing case](#options-and-the-do-nothing-case)
  - [Walk in the Pack with `Access::list` and `Access::read`, and keep a VFS search as the measured upgrade](#walk-in-the-pack-with-accesslist-and-accessread-and-keep-a-vfs-search-as-the-measured-upgrade)
  - [Page by re-running the search with `offset`, and keep no cache](#page-by-re-running-the-search-with-offset-and-keep-no-cache)
  - [Order by full path, and cut by work done instead of by a clock](#order-by-full-path-and-cut-by-work-done-instead-of-by-a-clock)
  - [Compile the Rust `regex` dialect once, and add a `literal` flag](#compile-the-rust-regex-dialect-once-and-add-a-literal-flag)
  - [Let `-A` and `-B` override `-C` per side](#let--a-and--b-override--c-per-side)
  - [Skip what cannot be read, and report it](#skip-what-cannot-be-read-and-report-it)
  - [Cut a long line around its match](#cut-a-long-line-around-its-match)
- [Behaviors to implement](#behaviors-to-implement)
  - 1. [Search in the Pack with a pure `find` over `Access`, sliced and yielding, and add no VFS search yet](#search-in-the-pack-with-a-pure-find-over-access-sliced-and-yielding-and-add-no-vfs-search-yet)
  - 2. [Publish the corrected schema, add `literal`, and refuse any other key by name](#publish-the-corrected-schema-add-literal-and-refuse-any-other-key-by-name)
  - 3. [Skip what cannot be searched, count it by reason, and name the first five](#skip-what-cannot-be-searched-count-it-by-reason-and-name-the-first-five)
  - 4. [Order results by full path, and stop by work done instead of by a clock](#order-results-by-full-path-and-stop-by-work-done-instead-of-by-a-clock)
  - 5. [Compile the Rust `regex` dialect before the walk, and name the unsupported feature](#compile-the-rust-regex-dialect-before-the-walk-and-name-the-unsupported-feature)
  - 6. [Count `head_limit` and `offset` globally in entries, with context left out](#count-head_limit-and-offset-globally-in-entries-with-context-left-out)
  - 7. [Page by re-running with `offset`, keep no cache, and name the cause of every cut](#page-by-re-running-with-offset-keep-no-cache-and-name-the-cause-of-every-cut)
  - 8. [Bound one result with a single byte budget, and cut a long line around its match](#bound-one-result-with-a-single-byte-budget-and-cut-a-long-line-around-its-match)
  - 9. [Render files as grouped lines, let `-A` and `-B` override `-C`, and take context from the same bytes](#render-files-as-grouped-lines-let--a-and--b-override--c-and-take-context-from-the-same-bytes)
  - 10. [Prune ignored trees during the walk, never follow a link, and let an explicit `path` win](#prune-ignored-trees-during-the-walk-never-follow-a-link-and-let-an-explicit-path-win)
  - 11. [Decode text as UTF-8 with replacement, strip a BOM, and split lines the way Read does](#decode-text-as-utf-8-with-replacement-strip-a-bom-and-split-lines-the-way-read-does)
  - 12. [Refuse bad arguments with messages that name the parameter and the fix](#refuse-bad-arguments-with-messages-that-name-the-parameter-and-the-fix)
  - 13. [Say in the description what the tool does: dialect, limits, skips and ignore rules](#say-in-the-description-what-the-tool-does-dialect-limits-skips-and-ignore-rules)
- [Where Runvil should differ from Cursor](#where-runvil-should-differ-from-cursor)
- [Mapping to Runvil](#mapping-to-runvil)
- [Risks and open limits](#risks-and-open-limits)
- [Sources](#sources)

## The recommendation in brief

Build the search in the Pack over `Access::list` and `Access::read`, and add a VFS search only when a measured trigger fires. Table 1 lists the thirteen behaviors, numbered in the order of the Behaviors section.

| Behavior | Recommendation | Confidence |
|---|---|---|
| 1. Pack `find` over `Access`, sliced and yielding | Walk with `list`, read with `read`, match in process, yield every 32 files or 2 MiB, add no VFS method | Medium, because I measured one machine and the yield is untested |
| 2. Corrected schema, `literal`, undeclared keys | Cursor's twelve parameters with the bundle's defaults, plus `literal`. Refuse other keys by name, accept `-n: true` | High for the schema, medium for `literal` and the refusal |
| 3. Skips counted by reason | Never fail on one entry. Count skips, name the first five, and say how many files were searched on a zero-match result | High, because every harness shows the failure |
| 4. Path order and a work budget | Full-path byte order (Glob's). Stop at 100,000 files or 1 GiB. Skip files over 16 MiB. Read no clock | Medium, because the numbers are my judgment |
| 5. Rust `regex` dialect | Compile before the walk, name the unsupported feature, add `literal`, keep case-sensitive | High, because the crate is measured and rig ships the error text |
| 6. `head_limit` and `offset` | Global, counted in entries, `0` and omitted mean no limit, one entry past the page decides the notice | High, because Cursor's counters are read from the bundle |
| 7. Re-run paging and notices | No cache. Cursor's footer grammar, with an exact total on the last page and a named cause for each cut | Medium, because two live calls match the grammar |
| 8. Byte budget and long-line window | 15,000 bytes of output, one constant shared with Glob, and a 1,000-byte window around the first match, with every cut on a character boundary | Medium, because the numbers are Cursor's and my judgment |
| 9. Render, `-A` and `-B` over `-C` | Cursor's grouped body with no envelope. Per-side override. Gap marker follows the request | Medium, because the body is Cursor's and ported |
| 10. Ignore rules, links, filters | Prune `.gitignore` trees with the module Glob shares, never follow links, skip `.git`, explicit `path` wins, `glob` uses Glob's grammar and never re-includes | Medium, because the shared module is open between the `ignore` crate and a small matcher |
| 11. UTF-8 with replacement, BOM, lines | Read's decode ladder and binary sniff, Read's line rule, one shared module | Medium, because the choices have wide support and I ran none against a model |
| 12. Argument messages | Cursor's two strings under one `std_Grep:` prefix, three siblings, one order, empty strings read as absent | High, because the main strings are in the bundle |
| 13. Description | State dialect, order, limits, skips and ignore rule; fix the `glob` examples | High for removing false claims, medium for the wording |

*Table 1. The thirteen behaviors, what to do for each, and the confidence in it.*

## Criteria

Five criteria judge every option and every behavior below. Each keeps one name through the report.

- **Accuracy.** The model finds what it asked for in few calls, and no result is silently wrong or partial. A file the search could not read, a page that repeats, and a cut with no marker all fail this criterion. A wasted round matters because a run has a fixed number of them.
- **File safety.** Grep writes nothing, so the risk is indirect. A search must not leave the tree through a link and must not hand the model a stale page that a later edit builds on. It takes read and list claims only, which a sibling read, glob or list can share. A sibling write to a file Grep read, or to a directory it listed, conflicts with those claims until the run's scope ends, and the sibling write carries the refusal.
- **Replay.** The same tree and the same arguments give the same bytes. A clock, a thread schedule, a walk order that the operating system chooses, and a cache of an earlier run each break it.
- **Bounded cost.** Tokens, memory and blocking time each have a stated ceiling, and the description tells the model what the ceiling is. The Harness polls every effect of a run on one task, so blocking time counts as much as tokens (`Pack::call` invariants, source).
- **VFS fit.** The behavior works on `Access` as it stands, and the report names each gap. A behavior that needs a VFS change says so and says what the change is.

The order when two criteria conflict is Accuracy, File safety, Replay, Bounded cost, VFS fit. VFS fit is a constraint more than a preference, so a behavior that fails it needs a named next step before it can ship.

Five terms stay fixed. An *entry* is what `head_limit` counts: a matching line in content mode, a file in files mode, a file row in count mode. A context line is never an entry. A *skip* is a file or directory the search did not read, for a stated reason, and the output counts it. The *output budget* is the most bytes of text one call returns. The *work budget* is the most files and bytes one call reads. A *page* is the entries one call returns after `offset`.

Each claim states its basis. An observed claim names a tier: `(bundle)` and `(daemon)` mean read from Cursor 3.27.12's shipped agent, `(source)` means code read at a named commit, `(web)` means a fetched page, and `(probe)` means something run. A `(probe)` with no further mark comes from the evidence report, and one marked "I measured" or "live" comes from this task. A claim I assume begins "I assume", and a judgment begins "I judge". Confidence is high, medium or low and describes the evidence. Likelihood uses one ladder: very unlikely, unlikely, roughly even chance, likely, very likely, almost certain. The two never share a sentence. The evidence report is [tool-grep.md](tool-grep.md), cited by section name, and the cross-tool facts come from [file-tools-index.md](file-tools-index.md).

Every `(probe)` marked "I measured" comes from this task, on one Windows 10.0.26300 machine with a warm file cache. The fixture is 20,000 files in 200 directories, 78,793,400 bytes, with 60 lines per file and one line per file that contains the pattern. Four probe programs ran: one on the in-memory backend with a counting allocator, one on `RealBackend::rooted` beside the shipped ripgrep 15.1.0-cursor5, one on the `regex` 1.13.1 crate, and one on the `ignore` 0.4.33 and `globset` 0.4.20 crates. I also re-read a dozen offsets and strings in Cursor's bundle and found each as the evidence report states. I ran nothing on Linux or macOS, nothing against a tree over 20,000 files on a real disk, and nothing that polls a Pack through the Harness.

## Options and the do-nothing case

Seven decisions have credible alternatives. Each gets a table with the do-nothing option named and the recommended option marked, then a paragraph for each option set aside. The remaining choices have no serious rival and appear only under Behaviors to implement. The confidence for each decision is in the brief table and under its behavior.

### Walk in the Pack with `Access::list` and `Access::read`, and keep a VFS search as the measured upgrade

The evidence report calls this the decision the six-tool set turns on ([tool-grep.md](tool-grep.md), Decision 1). It recommends a VFS `search` capability with a streaming result and a claim on the searched subtree, at medium-high confidence, and it names rig's pure `find` over a file list as the fallback. I recommend the fallback, built on a directory listing instead of a file list (Table 2). The reasons are three Runvil facts that the evidence report did not weigh, and a cost I measured on Runvil's own backend.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **C. Pack `find` over `Access::list` and `Access::read` (recommended)** | The Pack walks directories in full-path order, reads each file, matches in process, and yields to the Harness every 32 files or 2 MiB | No VFS change. A failing directory or file is caught where it happens. File type and size arrive with the listing, so no extra call. A files-mode search stops at its limit. A stop can take effect between slices | 4.0 s per 20,000 files on the real backend, 7 times ripgrep. About 140 us of containment per `Access` call. One claim entry (116 bytes) and one event per file | Recommended |
| A. VFS `search` capability | The backend walks and matches; the Pack supplies a matcher and renders | Containment is paid once per search, which is about 2.8 of the 4.0 s. One event. A backend can push the search down | A matcher trait, because the VFS crate has no dependencies. A read-kind subtree claim that does not exist. A new `Op`. A resumable shape so the call can yield. A new backend method | Set aside until a trigger below fires |
| B. Pack `Access::glob`, then `Access::read_string` | The Pack, with the shortest code | Nothing new to learn | 6.5 s per 20,000 files. One unreadable directory or non-UTF-8 name fails the whole glob. The path list sits in memory. No early stop | Set aside |
| Do nothing: keep the stub, ship no Grep | Nobody. The model uses Glob, then Read on each candidate | No search code to get wrong | A search costs one Read round per candidate file. A model trained on Cursor calls Grep first and gets a stub | Set aside |

*Table 2. Four ways to search file contents on Runvil's VFS. Sources: [tool-grep.md](tool-grep.md), Decision 1 and Q7; `operations.rs`, `claims_tree.rs`, `real.rs` and `real/files.rs` in the VFS crate at commit `08801150fc67`; my probes.*

The cost is measured, and the numbers are in Table 3. Over the real backend the Pack route takes 3.95 to 4.09 s for the 20,000 files. Ripgrep takes a median 550 ms, and a raw single-thread `std::fs` walk with the same matcher takes 1.24 s. The gap between the last two figures and the first is 2.8 s, or about 140 us per call. It comes from `RealBackend`, which runs `exists` and `fs::canonicalize` before every read and stat so that a link cannot leave the mounted root (`real/resolve.rs`, source). `Access::read` costs 184 us and `Access::stat` costs 127 us per file on this machine (probe, I measured). I assume the same code costs far less on Linux, where `realpath` is cheaper, and I did not measure it.

| Search over 20,000 files, 78.8 MB | Wall clock | Per file |
|---|---|---|
| ripgrep with Cursor's Grep argv, five runs | 532 to 589 ms, median 550 ms | 27 us |
| raw `std::fs` walk, one thread, whole-text prefilter | 1.23 s (1.71 s on the first round) | 62 us |
| Pack over `Access`: `list` walk, `read`, regex | 3.95 to 4.09 s | 200 us |
| Pack over `Access`: `glob("/**")`, `read`, regex | 6.47 to 6.58 s | 325 us |
| `Access::glob("/**")` alone | 2.65 s | 132 us |
| `list` walk that stops at 100 matching files | 21 ms | n/a |

*Table 3. What a search costs on `RealBackend::rooted`, Windows, warm cache (probe, I measured). A tree of 20,000 files of 3,300 bytes on the in-memory backend takes 98 ms through `list` and `read`.*

Three facts about Runvil decide against option A as the evidence report specifies it. First, the claim it recommends is the wrong kind. `Claims::claim_subtree` records a write: `record_subtree` sets `region.write`, and `check_read`, `check_write`, `claim_glob` and another `check_subtree` all refuse against a covering subtree write (`claims.rs`, `claims_tree.rs`, source). The evidence report says the claim "conflicts with any write anywhere under the root". It also conflicts with every concurrent Read, Glob, List and Grep under the root. A model that asks for two searches, or a search and a Read, in one round would see one of them fail with `VfsError::Conflict`. Second, the VFS crate is std only, with a manifest test that forbids any dependency (`vfs/Cargo.toml`, source). A search inside it cannot use `regex`, so it needs a matcher trait that the Pack implements, which the evidence report names as a variant and does not cost. Third, `Pack::call` must not block while polled and must be cancellable, and it cannot move the `Access` to a thread (`pack.rs` and `context.rs`, source). One synchronous `search` that runs for 5 s breaks the first rule and cannot be dropped midway.

Option A is still the better end state, and it needs five things.

- A matcher trait object that the Pack passes in, because the VFS crate has no `regex`.
- A read-kind subtree claim: a new region in `ClaimsTables`, checked by `check_write` and `check_subtree`. `claim_glob` is the nearest model, because it claims the pattern as a whole (source).
- An `Op::Search` variant. `Op` is `#[non_exhaustive]`, and the policy contract says an unnamed operation is denied, so `ModePolicy`'s `is_mutation` match must name it.
- A resumable step, so the Pack can run a bounded slice, yield, and resume from a cursor.
- A backend method with a default built from `list` and `read`, the way `read_range`, `glob_kind` and `str_replace` already have defaults, so `MemoryBackend` and every test double stay unchanged and `RealBackend` overrides it with a walk that contains the root once.

I recommend building A when one of three things happens. A deployed run shows a Grep over a tree the model uses costing more than 2 s on the platform it runs on. A tree over 100,000 files becomes routine. Or a backend that is not local needs to run the search where the files are. Confidence in the trigger is low, because no source says what trees Runvil agents search. Until then the Pack's `find` takes one seam so that the swap changes nothing above it: a pure function from a request to a typed `Found` (hits, counts, skips, the reason it stopped), and a separate renderer, as rig does (source, `search.rs:283-308`).

Option B loses on three measured grounds. It is 60 percent slower than the `list` walk, because `Access::glob` on the real backend walks the whole tree and then stats every match through the default `glob_kind` (`real.rs`, `backend.rs`, source). It fails whole on one unreadable directory or one non-UTF-8 name, because `walk` returns the first error with `?` (`real/files.rs`, source). And it materializes the path list: 10.3 MB for 200,000 paths of 20 bytes in my run (probe), which I estimate at about 20 MB for paths of 64 bytes. Doing nothing loses on Accuracy, since the model needs one round per candidate file. The claim cost that the evidence report treats as decisive is small: 116 bytes per distinct file, 23.2 MB at 200,000 files, none for a path read twice, and the memory returns when the call's scope ends (probe, I measured).

The winner has trade-offs. It is the slower route on large real trees: I extrapolate 40 s at 200,000 files, assuming linear scaling, which is why the work budget in the order behavior exists. It emits one event per file, and I did not measure what a sink costs, since the VFS crate says only that "the sink must be cheap" (source). And it makes the memory backend's `list`, which scans every key for each directory, the slow part of an in-memory search: 2.06 s for 200,000 files in 202 directories against 0.52 s for `glob` plus `read` (probe, I measured). The in-memory store a prompt uses is small, so I rate that cost low.

### Page by re-running the search with `offset`, and keep no cache

Only Cursor, Claude Code and Zed page at all. Nine other harnesses cap the result and leave the model one move, which is to narrow ([tool-grep.md](tool-grep.md), "How each harness pages"). The three that page fail in three ways, and Table 4 sets Runvil's choice beside them.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Do nothing: cap, then tell the model to narrow (nine harnesses) | The model | No state. No drift | The model cannot see result 101. pi's trailer says `Use limit=2N`, which restarts an unordered search, so the larger run need not extend the smaller | Set aside |
| **Re-run with `offset`, keep no cache (recommended)** | The Pack skips `offset` entries, then stops one entry past the page | Pages join when the order is stable. Every call reads the tree as it is now. No state to invalidate or to record for replay | Page k walks to its own position. An edit between two pages shifts the later page | Recommended |
| Cache five full results and re-paginate them (Cursor) | The Pack | Page 2 matches page 1 even under an unstable order. No second walk | A search after an edit returns the old tree with no signal. The key omits `head_limit` and `offset`. A generation counter that bumps on every write would fix it, and no harness has built one | Set aside |

*Table 4. Three ways to let the model see past the first page of matches. Sources: [tool-grep.md](tool-grep.md), Decision 10, Q1 and Q9; Claude Code and Zed sections.*

Claude Code re-runs the search on every page and slices afterward, and Zed re-runs it and stops once the page is full (source). Zed's pages drift because its `offset` skips raw regex ranges while its page counts merged blocks, so page 2 repeats part of page 1 (`grep_tool.rs:299-305`, a hand trace that I did not run). Cursor's cache exists to make its `offset` coherent under an order that is only stable when a 2 s stat pass finishes. With a path order that never depends on a clock, the cache has no job left and only its hazard remains. The cost of a re-run is real: a page deep in a large result walks most of the tree again, 4 s here for 20,000 files. I assume models rarely ask for more than three pages, because the trailer also tells them to narrow, and I did not measure it. A search that stops at 100 matching files took 21 ms (probe, I measured), so early pages are cheap.

### Order by full path, and cut by work done instead of by a clock

Cursor sorts files newest first after the search, under a 2000 ms stat budget, and keeps ripgrep's thread-dependent order when the budget expires ([tool-grep.md](tool-grep.md), Decision 8 and Q11). Its search also stops at 25 s by clock. Two identical runs on a large tree can print different bytes, which fails Replay (Table 5).

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Do nothing: whatever order the walk yields (pi, OpenCode, Cline, zeroclaw's `rg`) | The file system and thread timing | Free | pi gave 8 outputs in 8 runs on 20,000 files, OpenCode 4 in 4, zeroclaw 12 in 12, Cline 6 in 6. Which hits survive a cap is arbitrary | Set aside |
| mtime newest first, then walk order past a stat budget (Cursor) | The Pack, after the whole walk | Recent files first | The first line cannot print until the walk ends, so no early stop and no cheap page 1. The memory backend has no mtime. A time budget makes the cut a clock decision | Set aside |
| mtime when the backend tracks it, else path | The Pack | Recency where it exists | The same call orders differently on the memory and real backends. Glob's evidence recommends it for Glob | Set aside for Grep |
| **Full-path byte order, with a work budget in files and bytes (recommended)** | The walk itself, from `list`'s name order and a one-line sibling key | No extra call. Streams. Stops early. Same bytes on every backend and run, and the same order as `Access::glob` | No recency signal | Recommended |

*Table 5. Four ways to order matches and to stop a long search. Sources: [tool-grep.md](tool-grep.md), Decision 8, Q11 and Q12; Glob evidence, Decision 3; `real.rs` and `memory.rs` in the VFS crate.*

The path order needs no sort of the whole result. `RealBackend::list` returns names in byte order, and the walk re-sorts each directory's entries by the key `name` for a file and `name/` for a directory, so a depth-first walk produces the byte order of the full path string. That is the order `Access::glob` returns and Glob prints: `/a-b/x`, then `/a.rs`, then `/a/x`, because `-` and `.` are bytes 0x2d and 0x2e and `/` is 0x2f. Sorting siblings by the bare name would put `/a/x` before `/a-b/x`, which is not Glob's order. I checked the key on 20,000 random trees (up to four entries per directory, names of up to three bytes over `a`, `b`, `B`, `-`, `.`, `_`, `!` and `~`, depth three, files and directories mixed): the keyed walk matched a plain sort of the full paths on all 20,000, and the bare-name walk differed on 334 (probe, I measured). Zed orders by worktree id, then path component order, then match offset (source). The work budget is a count of files and bytes, so a slow machine and a fast one stop at the same file. Claude Code's newest-first sort has no budget at all, and rig has no deadline, so its worker thread cannot be stopped (source).

### Compile the Rust `regex` dialect once, and add a `literal` flag

Cursor's pattern is whatever ripgrep's Rust `regex` accepts, with no literal mode ([tool-grep.md](tool-grep.md), Decision 11). Runvil runs the same crate in process, so the question is which dialect the model sees and how a failure reads (Table 6).

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Do nothing: Rust `regex`, no literal flag (Cursor) | The `regex` crate | Linear time. One dialect | A model that forgets to escape `a.b` matches `axb` and sees no error | Set aside |
| **Rust `regex`, a `literal` flag, errors that name the feature (recommended)** | The Pack compiles before the walk; `regex::escape` serves `literal` | Linear time. `foo(` and `(?=x)y` fail at once, naming the cause | One added parameter, so `grep.json` leaves Cursor's text | Recommended |
| `fancy_regex` (Zed) | The Pack | Look-around and back-references | Backtracking, bounded at 1,000,000 attempts, and a file that hits the bound loses later matches silently. Zed's description names the `regex` crate it does not run | Set aside |
| Validate in one dialect, run in another (Gemini CLI) | A JavaScript `RegExp`, then ripgrep | Nothing | `(?i)foo` is refused before ripgrep sees it. `(?=x)y` passes, fails inside ripgrep, and returns as success text | Set aside |

*Table 6. Four ways to define the pattern language. Sources: [tool-grep.md](tool-grep.md), Decision 11 and Trap 18; my `regex` 1.13.1 probe.*

I measured linear time instead of assuming it. On `^(a|aa)+$`, `regex` 1.13.1 took 304.5 us, 962.1 us and 3.84 ms for 100,000, 400,000 and 1,600,000 characters, and `(x+x+)+y` on 200,000 characters took 508.3 us (probe, I measured). Rig's run of the same crate gave 246 us, 966 us and 3.85 ms ([tool-grep.md](tool-grep.md), Q7). A pattern whose compiled program passes 10,485,760 bytes is refused with `Compiled regex exceeds size limit of 10485760 bytes.` (probe). So no pattern can hang the call, which removes a hazard that Copilot manages with a 20,000 ms timeout and Zed with a backtrack limit.

### Let `-A` and `-B` override `-C` per side

Cursor forwards `-C` only when neither `-A` nor `-B` is present, so `-C: 3, -A: 1` gives one line after and none before, with no warning ([tool-grep.md](tool-grep.md), Decision 5). Doing nothing means reproducing that (Table 7).

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Do nothing: reproduce the drop (Cursor) | The Pack | Byte parity with Cursor | A supplied parameter is silently ignored. The model asked for symmetric context and got a different shape | Set aside |
| `-C` overrides `-A` and `-B` | The Pack | `-C` means what its text says | The model's explicit `-A` is lost instead | Set aside |
| **`-A` and `-B` override `-C` on their own side (recommended)** | The Pack | No supplied value is ignored. `-C: 3, -A: 1` gives 3 before and 1 after | A small divergence from Cursor | Recommended |
| Refuse the combination | The Pack | Nothing is silently wrong | An error the model did not expect, one round lost | Set aside |

*Table 7. Four ways to resolve `-C` against `-A` and `-B`. Source: [tool-grep.md](tool-grep.md), Decision 5 and Trap 5.*

### Skip what cannot be read, and report it

Today the evidence shows three outcomes for one unreadable file. pi and zeroclaw let it make ripgrep exit 2 and discard every match. Cursor tolerates an unreadable subdirectory when stdout is non-empty and says nothing, so a search that found nothing and could not read three directories prints `No matches found`. Rig and Zed skip a non-UTF-8 or binary file with no notice to the model ([tool-grep.md](tool-grep.md), Q5, Q6 and Trap 19). Table 8 sets out the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Fail the call on the first error (pi, zeroclaw) | The walk | Nothing is hidden | One bad file erases every match. Runvil's own `glob` already does this on a bad directory | Set aside |
| Do nothing: skip silently (Cursor, Zed, rig) | The walk | Always a result | The model reads `No matches found` as "absent" and concludes the symbol does not exist | Set aside |
| **Skip, count by reason, name the first five (recommended)** | The Pack catches each error where it happens | A partial result with its limits stated. The model learns what to retry | A few lines of output, and a classification table to keep | Recommended |

*Table 8. Three ways to treat a file or directory the search cannot read. Sources: [tool-grep.md](tool-grep.md), Decision 9, Q5 and Trap 19; `error.rs` in the VFS crate.*

### Cut a long line around its match

Cursor passes `--max-columns 1000 --max-columns-preview`, so a longer line shows its first 1,000 characters. A match at character 1,500 is not in the preview, and the model gets a result line with no match in it ([tool-grep.md](tool-grep.md), Trap 11, probe). Table 9 sets out the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| Do nothing: keep the first 1,000 characters (Cursor, pi, rig, Gemini CLI, OpenCode) | The renderer | Trivial | The match can be absent from its own line. Minified code and data files hit this | Set aside |
| No per-line cap | Nobody | The whole line | One 5 MiB line becomes 5,243,675 characters of output (pi, probe) | Set aside |
| **Window of 1,000 bytes that contains the first match (recommended)** | The renderer, from the match offsets the regex returns | The match is always visible. The cost is bounded | A marker and some arithmetic. Copilot windows around the match too | Recommended |

*Table 9. Three ways to bound the length of one result line. Sources: [tool-grep.md](tool-grep.md), Trap 10, Trap 11 and the Copilot section.*

Reproducing the `-C` drop lost because it fails Accuracy for no gain. Cursor's description gives no hint of the drop, and the only reason to keep it is byte parity with a tool Runvil is not diffing against. Overriding both sides with `-C` lost because it throws away a value the model chose on purpose. Refusing the pair lost because it costs a round to protect a combination that has one sensible reading.

Failing the call on the first unreadable file lost because Runvil's own `glob` already shows the result: one bad directory ends the search, and the model gets an error where matches existed. Skipping silently lost because it leaves the model one wrong conclusion to draw, which is that the symbol is absent. The cost of the winner is a short notice and a table of reasons to keep.

Keeping the first 1,000 characters lost because the evidence shows it hiding the match, and dropping the cap lost because a single 5 MiB line then costs more output than the whole budget. The cost of the winner is a marker string and the arithmetic of a window, both of which a test pins.

## Behaviors to implement

Thirteen behaviors follow, ordered by what a mistake costs. The first two fix where the work happens and what the model can send. The next three decide what the model learns about files that were not searched, the order of results, and the pattern language. The rest set the counting, the paging, the output limits, the rendering, the ignore rules, the text rules, the argument messages and the description. Behaviors 1, 3, 4, 8 and 11 use numbers that I chose by judgment. Each says so.

### Search in the Pack with a pure `find` over `Access`, sliced and yielding, and add no VFS search yet

**Recommendation.** Write `find(access, request) -> Found` as a plain function that walks depth-first with `Access::list`, reads each candidate file with `Access::read`, matches in process, and returns data (hits, counts, skips, why it stopped). Write `render(&Found) -> String` as a separate function. Yield to the Harness after every 32 files or 2 MiB read, whichever comes first, with a hand-written future that wakes its own task and returns `Pending` once. Add no method to `Access`. Start with a `stat` of `path`: a directory is walked, and a file is searched alone (Behavior 12).

**Why.** The walk, the reads and the matching all run inside `Pack::call`, which "must not block while polled" and must be cancellable, so the work has to come in slices that end at an `await`. A single synchronous VFS `search` call cannot be sliced or dropped midway, and a resumable one costs what Table 2 lists (Bounded cost, VFS fit). The `list` walk also catches an error in the directory or file where it happens, so one bad entry never ends the search (Accuracy). Splitting `find` from `render` follows rig's design, where tests and four consumers read the typed result without parsing text (source). I judge 32 files because at 184 us per `Access::read` it is about 6 ms of blocking per slice. That figure is an assumption about the Harness: no source shows a Pack that yields, so the first test below is also an experiment.

**Evidence.** Over the real backend the walk costs 3.95 to 4.09 s per 20,000 files, against 0.55 s for ripgrep and 1.24 s for a raw single-thread walk, and the 2.8 s gap is per-call containment (Table 3, probe, I measured). The claim table grows by 116 bytes per file and by nothing for a path read twice (probe, I measured). `claim_subtree` is a write claim and would make every Grep conflict with a sibling Read or Grep (`claims.rs`, source). The VFS crate has no dependencies, so a VFS search needs a matcher trait ([tool-grep.md](tool-grep.md), Decision 1, Option C; Table 2). Rig runs the same pure-function shape over three backends with tests that pin its cost, and its batch of 100 whole files peaked at 1,626 MiB, so the batch here is bounded in bytes (source, probe).

**Without it.** A synchronous search over a 200,000-file tree stalls every other effect of the run for tens of seconds, and a stop cannot reach it. A `glob` plus `read_string` search fails whole on one unreadable directory, because `walk` returns the first error. A VFS search built on `claim_subtree` makes two parallel Grep calls conflict.

**Test.** Build the `ToolContext` with `ToolContext::new` over a `VfsRef` that has an `on_op` sink and 1,000 seeded files, since `TestCall` offers only an empty in-memory store. Poll the call future by hand with a no-op waker and count `Pending` results: expect at least 31 for 1,000 small files. Drop the future after the first `Pending` and assert that the sink saw at most 32 `Read` events. Assert that `find` returns `Found` without touching a renderer, and that two Grep calls polled together over one tree both succeed.

**Confidence.** Medium. I measured the cost and read the invariants, but I ran one machine and no Harness, and the yield pattern is untested. It would change if the Harness cannot yield or if a tree over 100,000 files is routine.

### Publish the corrected schema, add `literal`, and refuse any other key by name

**Recommendation.** Replace the property set in `schemas/grep.json` with Cursor's twelve parameters in the bundle's order, using the bundle's text: `pattern`, `path`, `glob`, `output_mode`, `-B`, `-A`, `-C`, `-i`, `type`, `head_limit`, `offset`, `multiline`. Delete `-n` and `context`. Give `-C` the text that now sits on `context`: "Number of lines to show before and after each match. Requires output_mode: \"content\", ignored otherwise." Set `output_mode` to default to `content`. Give `head_limit` no default and `minimum` 0. Add one parameter, `literal`, a boolean that defaults to false. Keep the hyphenated names and add no aliases. Refuse every other key with the shared unknown-parameter message, which names the key and lists the thirteen parameters: ``std_Grep: `context` is not a parameter of this tool. The parameters are `pattern`, `path`, `glob`, `output_mode`, `-B`, `-A`, `-C`, `-i`, `type`, `head_limit`, `offset`, `multiline` and `literal`.`` Accept `-n: true`, which asks for what the tool always does, and refuse `-n: false` with ``std_Grep: `-n` must be true or left out, because line numbers are always shown.``

**Why.** The shipped file drifted from the bundle in five ways, all copied from Claude Code, and two of them change what a call returns. A model that omits `output_mode` expects paths because the shipped text says `files_with_matches`, and gets lines. A model that sends `-n: false` expects no line numbers and gets them. Cursor's schema strips an undeclared key without a word, so there `context: 3` and `-n: false` do nothing and the model is not told. Runvil's `check` lets undeclared keys through "so the operation sees it and refuses it itself", so the Pack must choose, and silence gives the model a wrong result (Accuracy). One helper in `check.rs` builds the unknown-parameter message for every tool, so the six read alike. `-n: true` is the one exception, because Claude-trained models send it often and it changes nothing. The hyphens stay because the premise of the crate is that models know Cursor's surface. An alias would add a name that no published schema shows a model, and it would contradict the refusal. A refusal can name the fix: Copilot answers a stray `pattern` with a message that names `query` (source). `literal` is the only addition and it is additive: a model that never sends it sees no change (D11).

**Evidence.** The bundle's `wIe` schema has twelve keys in that order and `output_mode` defaults to `content` (bundle, probe, I re-read it). It has no `-n` and no `context`, and `head_limit` has no default and is coerced with `|| void 0` so that `0` means unlimited (bundle, probe, I re-read it). The shipped file has fourteen properties, says `Defaults to "files_with_matches"`, and says `head_limit` "Defaults to 250 when unspecified" ([tool-grep.md](tool-grep.md), "The schema his crate ships today is wrong"). The evidence report called Cursor's object strict, and my checks do not support that, so I corrected it there. `wIe` is built with `U$.Ikc`, and the export list of the bundle's zod module 77602 names `Ikc` as `ye`, which is `function ye(e,t){...return new ge(r)}` and sets no catchall. In that module `strict()` and `loose()` each set a catchall and `strip()` clears it, and `wIe` has no `.strict()`. A live Grep call with an invented key and `-n: false` returned `No matches found` with no error (bundle and probe, one call; I re-read the alias for this pass). Cursor's `Grep` rounds `head_limit` and the context values up and `offset` down, and clamps to 2147483647. Runvil's `check` has no `minimum`, accepts any JSON number, and refuses `"50"` for a number with a message that names the parameter and the value (`check.rs`, source). Cursor coerces `"50"` and `"yes"`, pi converts them, and rig refuses them (source).

**Without it.** The shipped schema sends a model to the wrong default and invites a `-n` that does nothing. A call with `context: 3` passes the check and the Pack either ignores it, which is a silent wrong result, or the model learns nothing. A silent `head_limit` default of 250 that Cursor does not have hides results with no signal.

**Test.** Extend `spec-tests.rs` to assert the parameter names in order, thirteen of them, and that `-n` and `context` are absent. Assert that `output_mode` has `"default": "content"` and that `head_limit` has no `default` and no description containing `250`. Assert the `-C` text. Call with `context: 3` and expect the unknown-parameter message above, which contains both `context` and `-C`. Call with `-n: true` and expect a normal result, and with `-n: false` expect the refusal above, which says line numbers are always shown. Call with `head_limit: 2.5` and expect a page of 3 entries, and with `offset: 2.9` expect an offset of 2.

**Confidence.** High for the twelve parameters and the five fixes, because I re-read the bundle and the evidence report confirms each. Medium for `literal` and for the refusal, because no transcript shows how models respond to either.

### Skip what cannot be searched, count it by reason, and name the first five

**Recommendation.** Never fail a search because of one file or one subdirectory. Record a skip with a reason and keep going. Report the skips after the results as one line: `[Skipped 5 entries: 3 binary files, 1 directory that could not be read, 1 file over 16777216 bytes. First: /a.bin, /b.bin, /c.bin, /d, /big.log]`. List the first five in walk order. Build every count with one `plural(n, noun)` helper in `plugin-std`, so one skip reads `[Skipped 1 entry: 1 binary file. First: /a.bin]`. A name with a character below 0x20 or 0x7f is never printed: it is a counted skip with the reason `name with a control character`. Fail the call only when the pattern is invalid, a parameter is wrong, or the search root itself is absent or refused, and let the shared `refusal` function word a root failure. That is the one function in `plugin-std` that builds every `VfsError` sentence for all six tools, from the `refusal-text` todo. A zero-match result states its coverage: `No matches found. Searched 412 files.` plus the skip line when there is one, and `Searched 1 file.` for one.

**Why.** A model that gets `No matches found` concludes the symbol is absent, and the cause may be a binary file, a permission, a size cap or an ignore rule. This is the one failure that every harness shares and none has fixed ([tool-grep.md](tool-grep.md), Decision 9). A skip count is the cheapest honest answer (Accuracy). The reasons map onto `VfsError` variants the walk already receives: `PermissionDenied`, `Io`, `NotUtf8` on a directory name, `Conflict` with a sibling call, `NotFound` for a file removed after the listing. A conflict is a skip with the reason `another call in this round`, and not a failure, because under Runvil's claims a sibling write to one file must not end a search of ten thousand. A name with a control character would split a path line in every mode, which [master-glob.md](master-glob.md), "Print absolute VFS paths, bare, one per line, and leave out any name with a control character", also removes.

**Evidence.** In content mode Cursor drops a binary file and reports nothing, while `-l` and `-c` list it, so one file is present in two modes and absent in a third ([tool-grep.md](tool-grep.md), Q5). A search that finds nothing and cannot read three directories prints `No matches found` (Q5). pi and zeroclaw let one unreadable file make ripgrep exit 2 and discard every match, and one unreadable directory turned a 301-match search into an error in zeroclaw (probe). OpenCode fails the whole call on one line over 64 KiB or one non-UTF-8 line (probe). I hit the same failure in Cursor itself: a Grep with no `path` in this workspace returned only `Error running tool: rg: .\.cursor\skills\runvil: The system cannot find the path specified. (os error 3)`, because `.cursor\skills\runvil` is a junction with no target. The shipped `rg.exe` run with the tool's flags reproduced it with exit code 2 (probe, two live calls and one direct run). `Access::list` reports a good and a dangling junction as `FileType::Symlink` on Windows (probe, I measured). The evidence report rejects rig's "silent omission behind a reassuring zero-result text", and Copilot's zero-result hint names the exclusion and the parameter that overrides it (source). `RealBackend::list` fails an entire directory on one non-UTF-8 name and `glob` fails the whole walk on one unreadable directory (`real.rs`, `real/files.rs`, source).

**Without it.** A model searching `/src` for a function in a tree with one unreadable directory sees `No matches found`, edits around a function that exists, and creates a duplicate.

**Test.** Seed a tree with a binary file, a 16 MiB plus one byte file, a directory the policy denies for `Op::List`, a file the policy denies for `Op::Read`, and one clean match. Run the search and assert the clean match, a skip line with the counts `1 binary file`, `1 file over 16777216 bytes`, one unreadable directory and one denied file, and the first paths in walk order. Search for text that occurs nowhere and assert `No matches found. Searched <n> files.` with the correct count and the same skip line. Assert that a missing `path` fails with `Path does not exist: /x`. Assert `Searched 1 file.` and `[Skipped 1 entry: 1 binary file. First: /a.bin]` for one of each. On Unix, seed a file whose name holds a newline and assert that it is a counted skip and that no path line is split.

**Confidence.** High. Every harness probed shows the failure, and the fix is a counter and a string.

### Order results by full path, and stop by work done instead of by a clock

**Recommendation.** Visit each directory's entries in the order of one key, compared byte by byte: the name for a file or link, and the name followed by `/` for a directory. A depth-first walk in that order yields the byte order of the full path string, which is the order `Access::glob` returns and Glob prints. Read no clock anywhere in the search. Stop at 100,000 files considered or 1 GiB read, whichever comes first, and say so: `[Search stopped after reading 100000 files. Narrow path, glob or type to search the rest.]`. Skip any single file over 16 MiB, as a counted skip. Keep the three limits in one value that a test can lower.

**Why.** Replay needs the same bytes from the same tree. A time budget cuts at a different file on a slower machine, and a walk order that the file system or a thread pool picks changes which matches survive a cap. Glob's order lets a model that runs Glob and then Grep over one tree see one sequence. The keyed walk costs one in-memory sort of each directory's entries, because `list` already returns names sorted, and it lets the search stop early without a stat pass (Replay, Bounded cost). A newest-first order would have to finish the tree before it printed a line. I chose the numbers by judgment. At 200 us per file, 100,000 files is about 20 s on this machine, which is near Cursor's 25 s deadline and so keeps the wait a model already knows. 1 GiB bounds a tree of large files. 16 MiB is rig's per-file ceiling (`MAX_FILE_BYTES`, source).

**Evidence.** Cursor sorts by `mtimeNs` under a 2000 ms stat budget and falls back to walk order, and its search stops at 25 s per root (bundle, probe, I re-read both constants). The stat pass cost 763 ms for 20,000 files in Node, and the evidence report extrapolates that roughly 50,000 files blow the budget, which is a model and not a measurement ([tool-grep.md](tool-grep.md), Q11). Without a sort flag pi gave 8 outputs in 8 runs on 20,000 files, OpenCode 4 in 4, zeroclaw's `rg` backend 12 in 12 and Cline 6 in 6, and ripgrep's `--sort path` gave one order in 12 runs (probe, source). Only Zed (worktree id, path component order, offset) and zeroclaw's internal walker (sorted, a deadline check per entry) are deterministic by design (source). On Runvil, `RealBackend::list` and `MemoryBackend::children` both return names in byte order, and `Access::glob` ends in a sort of the whole path strings (`real.rs`, `memory.rs`, source). The sibling key matched that sort on 20,000 random trees (probe, I measured; see the paragraph after Table 5).

**Without it.** A model that repeats a search to confirm a result sees a different first hit after a file system change in walk order, and a recorded run does not replay. A clock cut sends the same tree to different outputs on two machines.

**Test.** Seed `/B`, `/a/x`, `/a-b`, `/a.txt` and `/b`, each holding the pattern. Assert the files-mode order `/B`, `/a-b`, `/a.txt`, `/a/x`, `/b`, the same bytes on a second run, and that the same tree listed by Glob with `**/*` gives the same sequence. Lower the file limit to 3 and assert that the output is the first three files of the full run and ends with the stop notice. Seed a 16 MiB plus one byte file and assert one skip. Add a source check that the search module names neither `Instant` nor `SystemTime::now`.

**Confidence.** Medium. The order is well supported: four harnesses are unstable and the two deterministic ones use a path order. The numbers 100,000, 1 GiB and 16 MiB are my judgment, and no source measures which trees models search.

### Compile the Rust `regex` dialect before the walk, and name the unsupported feature

**Recommendation.** Compile the pattern with the `regex` crate before any filesystem call: `case_insensitive` from `-i`, `multi_line(true)` always, `dot_matches_new_line` and `crlf(true)` only when `multiline` is set. Match line by line outside multiline mode, and over the whole text in multiline mode. When `literal` is true, compile `regex::escape(pattern)`. Keep the case-sensitive default. On a compile failure, return ``std_Grep: Invalid regular expression "<pattern>": <reason>. Grep matches with Rust regex syntax, which has no look-around and no back-references. To match the text literally, set literal to true.``, with `<reason>` the text after `error: ` in the crate's message.

**Why.** The crate runs in linear time, so no pattern can hang the call, which removes a hazard that other harnesses manage with a timeout or a bound (Bounded cost). It names the unsupported feature, so the model fixes the call in one round (Accuracy). The error text is the thing to keep from rig: its message for a look-around reads `error: look-around, including look-ahead and look-behind, is not supported`, and the crate's own `regex` 1.13.1 prints the same text. A `literal` flag removes the case where `a.b` matches `axb` with no error. Keeping case-sensitive matches Cursor and Claude Code, while Zed, Gemini CLI, Cline and Copilot default to insensitive, so a model trained on those may omit `-i` and expect it. I judge that cost acceptable, because the shipped schema says what `-i` does.

**Evidence.** I ran `regex` 1.13.1: `(?=x)y` and `(?<=a)b` fail with the look-around text; `(a)\1` fails with `backreferences are not supported`; `foo(` with `unclosed group`; `[a-` with `unclosed character class`; and `(?i)foo` compiles (probe, I measured). `^(a|aa)+$` took 304.5 us, 962.1 us and 3.84 ms at 100,000, 400,000 and 1,600,000 characters, which is linear (probe, I measured; rig's run gave 246 us, 966 us and 3.85 ms). A program past 10,485,760 bytes fails with `Compiled regex exceeds size limit of 10485760 bytes.` (probe). With `crlf(true)`, `^abc$` matched `x\r\nabc\r\ny` and without it did not (probe, I measured). Gemini CLI checks the pattern as a JavaScript `RegExp` and runs it in Rust, so `(?=x)y` passes the check and fails later as success text ([tool-grep.md](tool-grep.md), Trap 18).

**Without it.** A model sends `(?<=a)b`, gets a bare parse error with no dialect, and retries three times. A model that forgets to escape `foo(` in a pattern it meant literally spends a round.

**Test.** Send `(?=x)y`, `(?<=a)b`, `(a)\1` and `foo(`, and assert a `std_Grep: Invalid regular expression` error for each that names the reason and the Rust dialect, as an error and not as success text. Send `a.b` with `literal: true` over a file holding `axb` and `a.b`, and assert one match. Send `^(a|aa)+$` over a 1,600,000-character line and assert that the call completes. Send `multiline` with `a.b` over `a\nb` and assert a match, and without `multiline` assert none. Send the same pattern with and without `-i` over `Alpha`, `alpha` and `ALPHA`, and assert one and three matches.

**Confidence.** High. The crate's behavior is measured twice, rig ships the same error text, and the choice follows what ripgrep already does for the Cursor surface.

### Count `head_limit` and `offset` globally in entries, with context left out

**Recommendation.** Count one entry per matching line in content mode, per file in files mode, and per file row in count mode. A context line is never an entry. Both counters are global across files. `head_limit: 0` means no limit, and an omitted `head_limit` means no limit. The search collects `offset + head_limit + 1` entries and uses the last one only to know whether a notice is due. Round `head_limit` and the context values up and `offset` down. In multiline mode every line a match touches is a match line and one entry.

**Why.** A per-file counter is the usual mistake, and it returns six lines for `head_limit: 5` over two files. Counting only matches keeps the number the description names, "total matches shown", and keeps `head_limit` from changing when the model adds context (Accuracy). The extra entry fixes a false notice: six harnesses print their cut notice when the count merely reaches the cap, so the model cannot tell exactly N from more than N (Accuracy). Treating `0` as unlimited is Cursor's behavior and Claude Code's, and an omitted limit is unlimited in Cursor, so a model trained on either meets no surprise. The output budget in Behavior 8 is the ceiling that remains.

**Evidence.** `head_limit` is a client-side counter and `--max-count` appears zero times in the bundle (bundle, probe, I re-read it). `0` becomes `undefined` through `|| void 0` (bundle, probe, I re-read it). Both counters sit outside the file loop, and `collected` increments only for non-context lines ([tool-grep.md](tool-grep.md), Q1 and Q2). Pi, rig, zeroclaw, Gemini CLI, OpenCode and Cursor fire the cut notice on an exact fit, and OpenCode's `dev` branch fixes it by fetching `limit + 1` and slicing (source, probe). Claude Code's default of 250 truncates silently, and its trailer `[Showing results with pagination = limit: 250, offset: 0]` does not say whether anything was cut (source).

**Without it.** A per-file counter prints six lines for `head_limit: 5` over two files of three matches each, and the model counts wrong. A search whose last page holds exactly the limit prints a notice, the model asks for the next page, and gets `No more results at offset=N.`

**Test.** Seed `/a.ts` with matches on lines 12 and 40 and `/b.ts` with three. With `head_limit: 2`, assert that only `/a.ts` appears, with both lines, and the notice says `at least 3`. Seed three files of three matches each and assert that `offset: 4, head_limit: 2` returns the 2nd and 3rd matches of the second file. With `head_limit: 0` and 40 matches assert that all 40 print. With `head_limit: 2` and exactly two matches assert that no notice is printed. With `-A: 5, head_limit: 2` assert two entries and their context lines.

**Confidence.** High. Cursor's counters are read from the bundle, and the exact-fit repair is a known fix.

### Page by re-running with `offset`, keep no cache, and name the cause of every cut

**Recommendation.** Keep no state between calls. Each call walks from the start, skips `offset` entries, collects one page plus one entry, and stops. Print no notice when `offset` is 0 and nothing was cut. Otherwise end the output with a blank line and one bracketed line, in Cursor's grammar:

```text
[Showing 1-50 of at least 51 matching lines. Use offset=50 to see more, or narrow the search.]
[Showing 1-120 of at least 121 matching lines. Output stopped at 15000 bytes. Use offset=120 to see more, or narrow the search.]
[Showing 51-60 of 60 matching lines.]
[Search stopped after reading 100000 files. Narrow path, glob or type to search the rest.]
```

The unit word is `matching lines` in content mode and `files` in the other two. Reaching the end after the first page prints the exact total, and a cut prints `at least`. An `offset` past the end prints `No more results at offset=N.` A work-budget stop offers no `offset`, because a re-run stops at the same file.

**Why.** Pages must join: page 1 followed by page 2 equals the unpaged result, which needs a stable order and one counting unit, and both come from the earlier behaviors (Replay). A cache would need an invalidation signal that no harness has built, and it would put state into a recorded run (File safety, Replay). Naming the cause helps the model act: `head_limit` cuts say `offset`, an output-budget cut says so, and a work-budget stop says to narrow, because paging cannot help (Accuracy). I keep Cursor's phrasing because a model trained on it reads `at least` and `Use offset=N` correctly, and I add the exact total on the last page because Cursor's `at least` fires even when the count is complete.

**Evidence.** Cursor's footer for a `head_limit` cut is `[Showing 2 of at least 5 matching lines. Use offset=2 to see more, or narrow the search.]`, with no `Results truncated. ` prefix, and a full page with nothing left prints no footer ([tool-grep.md](tool-grep.md), "The truncation trailer", ported and run). Its cache key omits `head_limit` and `offset`, so a second call after an edit returns the old tree (Decision 10 and Q9). Zed's `offset` counts raw ranges while its page counts merged blocks, so page 2 repeats page 1 (a hand trace that I did not run). Pi's trailer says `Use limit=2N`, which restarts an unordered search, so the larger run need not extend the smaller (probe). The evidence report asks for a live call before a byte-exact test is frozen. Two live Grep calls in this session, with `head_limit` 15 and 12, ended with `[Showing 15 of at least 56 matching lines. Use offset=15 to see more, or narrow the search.]` and `[Showing 12 of at least 108 matching lines. Use offset=12 to see more, or narrow the search.]`, each with no `Results truncated. ` prefix (probe, two live calls, build not recorded).

**Without it.** A cache serves a stale tree to a model that edited a file between pages. A page boundary in two different units repeats results, and a model that sees a result twice distrusts the next page. A notice that fires on an exact fit sends the model for a page that says `No more results`.

**Test.** Seed 25 matches. Run `head_limit: 10` at offsets 0, 10 and 20 and assert that the three pages joined equal the unpaged result and that the three notices are `[Showing 1-10 of at least 11 matching lines. ...]`, `[Showing 11-20 of at least 21 matching lines. ...]` and `[Showing 21-25 of 25 matching lines.]`. Assert `No more results at offset=30.` at offset 30. Write to a seeded file between two page calls and assert that page 2 reflects the new bytes. Stop at a lowered file limit and assert that the notice offers no `offset`.

**Confidence.** Medium. The grammar is Cursor's, ported from its code and matched by two live calls, and three harnesses show how paging fails. The new parts, the exact total on the last page and the cause sentence, are mine and untested against any model.

### Bound one result with a single byte budget, and cut a long line around its match

**Recommendation.** Cap the rendered body at 15,000 bytes of UTF-8 in every mode, cut at a line boundary, and say so in the notice. Count bytes with `str::len`. The same constant caps Glob's reply. The cap counts the body: the `[Showing ...]` line and the skip line come after it and are not counted. A line over 1,000 bytes shows a window of at most 1,000 bytes that contains the first match, starting up to 200 bytes before it, cut on character boundaries, with `[... N bytes omitted]` before and after what was cut. A context line over 1,000 bytes keeps the longest prefix of at most 1,000 bytes that ends on a character boundary, and a trailing marker. No cut in this tool splits a UTF-8 code point. The body cap cuts only at a line boundary. The window moves its start forward and its end back to the nearest character boundary, so the first match stays inside it, and the context-line clip steps back. Both call `clip_to_boundary` in the shared `text` module that `master-read.md` defines, so Read and Grep share one cut and one rule, and the omitted counts stay in bytes. Like Read, the cut protects code points and not grapheme clusters. Drop the 2000-entry cap, the 8 MiB stdout cap, the line budget and the 10,000-line parse window.

**Why.** Five of Cursor's caps exist because ripgrep is a child process, and they vanish when the search is in process ([tool-grep.md](tool-grep.md), "What Runvil can support today"). What remains is one cap on text the model reads. Cursor's remaining limits use three units: bytes for the long-line cut, characters for its preview, and UTF-16 code units for the 15,000 render budget. A model cannot reason about three. Bytes are exact, cheap in Rust and conservative for CJK text (Bounded cost). The window fixes a failure in Cursor: its preview shows the first 1,000 characters, and a match at character 1,500 is not in them, so the model gets a result line with no match in it (Accuracy). I judge 15,000 and 1,000 because they are Cursor's figures, and I assume the habits of its models formed on them.

**Evidence.** A 2,000-character line with the needle at position 1,001 previewed 1,000 characters with no needle in them (probe, [tool-grep.md](tool-grep.md), Trap 11). The cut is decided in bytes: a line of 999 ASCII bytes passes and one of 1,000 is cut, and 334 CJK characters (1,002 bytes) are cut (probe, ER). Copilot windows around the match; pi, rig, Gemini CLI and OpenCode keep the head; Claude Code drops a long line and keeps no preview (source). Pi, which has no `--max-columns`, turned one 5 MiB line into 5,243,675 characters of output (probe). Rig clips at 300 Unicode scalar values with no marker (source).

**Without it.** A minified bundle returns one line of 80,000 bytes and fills the budget with a result the model cannot use. A head-only cut hides the match.

**Test.** Seed a 2,000-character line with `NEEDLE` at position 1,001 and assert that `NEEDLE` appears, that both markers appear, and that the two omitted counts plus the window equal the line's byte length. Seed a line of 600 CJK characters and assert that the output is valid UTF-8 and the window starts and ends on a character boundary. Seed a line of 4-byte emoji with the match in the middle and assert that the window, the first match inside it, and the omitted counts are all whole characters, with the two counts plus the window equal to the line's byte length. Seed a context line of 600 CJK characters beside a match and assert that the clipped context line is valid UTF-8 and keeps at most 1,000 bytes. Sweep the window's start offset across every byte position inside one 3-byte character and assert that the output is always valid UTF-8. Seed 400 matching lines of 100 bytes and assert that the body is at most 15,000 bytes and the notice says `Output stopped at 15000 bytes.` Assert the same in files mode with 400 long paths.

**Confidence.** Medium. The window is a measured fix and Copilot ships the idea. The two numbers are Cursor's and my judgment, and no source measures what budget suits a model's context.

### Render files as grouped lines, let `-A` and `-B` override `-C`, and take context from the same bytes

**Recommendation.** Print each file's path once, as the virtual path with its leading `/`, then its lines as two spaces, the line number with no padding, `:` for a match or `-` for context, and the text. Separate files with one blank line and end the output with none. Print no envelope. Between two non-adjacent groups of one file print `  --`, whenever `-A`, `-B` or `-C` is above 0. Merge overlapping context, and never print a line twice. Let `-A` and `-B` set their own side after `-C` has set both. Ignore context in the other two modes. In files mode print one path per line, and in count mode print `path:count` for matching lines per file, where the model reads the count after the last colon.

```text
/src/a.ts
  3:const alpha = 1
  4-let beta = 2
  --
  9:const gamma = 3
  10-export default gamma

/src/b.ts
  1:const first = 0
```

**Why.** The path appears once per file, so the form costs fewer tokens than the `path:line:text` of Claude Code, pi and rig, and Cursor's models know it. The gap marker follows the request and not the result. Cursor's `hasContext` is computed over the whole result, so a context line in one file turns the marker on for every file (Accuracy, Replay). `-C` with `-A` is the case where Cursor's silent drop gives the model a shape it did not ask for. Context comes from the same text the search already holds. Pi re-reads each matched file with a second decoder, which shifted every later line after a lone CR, printed a UTF-16 file as garbage and repeated lines for overlapping blocks (probe).

**Evidence.** Cursor's body is `  N:text` for a match and `  N-text` for context, a blank line between files, and a `.trim()` on the whole result so the last file ends flush against the closing tag ([tool-grep.md](tool-grep.md), "Content mode output" and Trap 9). `-C` is forwarded only when neither `-A` nor `-B` is present (bundle, probe, I re-read the guard). The gap separator is emitted only when the result has a context line anywhere and the gap is over one (Trap 6). Count mode splits on the last colon so that `C:\src\a.ts:3` parses (Q8).

**Without it.** `-C: 3, -A: 1` gives one line after and none before. Pi's re-read repeats a line when two blocks overlap. A model parsing a count row with a colon in the path reads the wrong number.

**Test.** Seed `/src/a.ts` as a 10-line file in which only lines 3 and 9 contain `const`, and search for `const` with `-A: 1`. Assert the first file's block above, with lines 4 and 10 as context. With no context flag assert that lines 3 and 9 print with no `--`. With `-C: 3, -A: 1` on matches at lines 5 and 20 of a 30-line file assert lines 2 to 6 and lines 17 to 21, which is three before each match and one after. With `-C: 2` on matches at lines 5 and 7 assert lines 3 to 9 once, with no `--`. Assert the last 30 bytes of a two-file result end at `const first = 0` with no trailing newline. In count mode assert `/a:b.ts:3` for a file whose name holds a colon.

**Confidence.** Medium. The body is Cursor's, run through a port, and the three changes are small. The count-mode and path conventions are my judgment.

### Prune ignored trees during the walk, never follow a link, and let an explicit `path` win

**Recommendation.** Read the `.gitignore` in each directory the walk enters, and in each ancestor of the search root, and prune an ignored directory before listing it. Honor `!` rules. Never descend into or read through a symbolic link or junction met during the walk, and count each as a skip with the reason `symbolic link`. A `path` that names a link is searched through it, as Glob does for `target_directory`. Never enter a path with a segment named `.git`, whether that segment is a file or a directory. Search other hidden files. Apply no ignore rule to the `path` argument itself, so naming an ignored directory searches it, and apply the rules found beneath it. Apply `glob` and `type` after the ignore rules, so they never bring an ignored file back. Drop Cursor's broad-glob guard, which this order makes moot. Use `.gitignore` whether or not a `.git` directory exists. The ignore rules are the shared module that Glob also calls, not a second copy ([master-glob.md](master-glob.md), "Put ignore and privacy rules in `Access`, apply them after matching, and count what they hide"). `glob-real` writes it and `grep-real` calls it during the walk. This tool never lists a pruned directory, so it cannot count what it left out, and its zero-match line names the rules instead.

The `glob` filter uses the same grammar as Glob ([master-glob.md](master-glob.md), "Accept the grammar `Access::glob` implements, and explain a zero that holds braces, brackets or question marks"): `*` and `**` only, case-sensitive, a relative pattern searched at any depth, and a leading `/` a full path. `{`, `[` and `?` match themselves until the VFS gains `{a,b}`, and then both tools gain it together. The matcher is one pure function that the VFS crate exposes in the first commit of `glob-real`, which runs before `grep-real`. If the VFS crate refuses that, `plugin-std` holds a copy of the four-token grammar and one test file runs the same cases against both. `globset` is not added. The cost against Cursor is that its `glob` filter ignores case and accepts braces (Table 10). When nothing matches and `glob` holds `{`, `[` or `?`, the zero-match line adds `{, [ and ? in glob match themselves; there is no alternation or character class. Call Grep once per alternative, or filter with type.` The `type` filter uses a table of file types, and an unknown type is an error. On a zero-match result with ignore rules in play, say so: `No matches found. Searched 412 files. Ignore rules from .gitignore were applied. Name an ignored directory in path to search it.`

One item stays open for Vinnie: whether the shared module uses the `ignore` crate or a small matcher of its own. The crate matches gitignore lines, directory pruning and a table of 224 file types against virtual paths, and the table also serves the `type` filter (probe, I measured). It adds two crates to the lock file, `ignore` and `globset`, because every other crate it names is already there (workspace `Cargo.lock` and the crate's manifest, read, not built). The standing plan allows no dependency beyond `serde` and `serde_json`, so the plan needs an edit for it, as it does for `regex`. A small matcher adds no dependency. It costs the gitignore rules that Runvil would then own, which are anchoring, `**`, a trailing `/`, `!`, and a file staying ignored when its parent directory is ignored, and it needs its own table of file types. For scale, the crate's `gitignore.rs` has 885 lines and its default type table 378 (source, read). I leave the choice open.

**Why.** Pruning is what makes the walk usable on a real project. A project root holds `target` or `node_modules` with 10^5 to 10^6 files, and without pruning the first search spends its work budget there (Bounded cost). A search that never follows a link cannot leave the tree through one, and it behaves the same on every platform (File safety, Replay). Ignore rules that always win are what the Glob evidence recommends, and they remove the case where a positive glob brings back an ignored file. The rules hide entries from Glob and Grep listings only. Read, Write, StrReplace and Delete act on any path the call names, and no privacy rule exists in version one. A zero-match line that names the rules turns the commonest silent omission into a recoverable one, which is Copilot's idea (Accuracy). I judge that `.gitignore` should apply without a repository, because the file is visible to the model and a rule that depends on a hidden `.git` is the kind of ambient state that Cursor's design suffers from.

**Evidence.** Cursor passes `--hidden` and `--follow` unconditionally and has no flag to turn either off, so a link to `/` makes ripgrep walk the file system and `filterBlockedFiles` removes results after the read ([tool-grep.md](tool-grep.md), "The short version" and Q7, attack 5). Claude Code passes no `--follow`, pi passes no `-L`, rig never lists links, and Zed skips directory links that leave the worktree (source). A positive `--iglob` overrides `.gitignore`, which is why the broad-glob guard exists, and the guard is off unless the host sets `enableGrepBroadGlobGuard` (bundle, probe, I re-read the flag wiring). The plan records that "the VFS has no ignore-file concept". The `ignore` 0.4.33 crate matches gitignore lines, directory pruning and a table of 224 file types against virtual paths with no file system, and reports `unrecognized file type: foo` for an unknown type (probe, I measured). `globset` 0.4.20 matches `*.{ts,tsx}` case-insensitively and not at depth unless prefixed with `**/` (probe, I measured), and Runvil does not add it. `Access::list` reports a junction as `Symlink` on Windows (probe, I measured). A live Cursor Grep failed whole on a dangling junction, which is the cost of following links ([Skip what cannot be searched](#skip-what-cannot-be-searched-count-it-by-reason-and-name-the-first-five)).

**Without it.** A search of a Rust project walks `target`, hits the 100,000-file budget, and reports a stop, or prints build artifacts. A link to a parent directory loops or leaves the tree. A model that wants a vendored file cannot reach it.

**Test.** Seed `/.gitignore` with `target/`, `*.log` and `!keep.log`, and files `/target/x.rs`, `/a.log`, `/keep.log` and `/src/y.rs`, each holding the pattern. Assert that only `/keep.log` and `/src/y.rs` match. Search with `path: "/target"` and assert `/target/x.rs` matches. Search with `glob: "**/*"` and assert `/target/x.rs` is still absent. Seed `/.git/config` with the pattern and assert it is absent, and seed a file named `/sub/.git` with the pattern and assert it is absent too. On Unix, and with a junction on Windows, add a link to a directory and assert it is a counted skip and its target's files appear only once. Search with that link as `path` and assert its files match through it. Assert that `glob: "*.ts"` matches `/d/a.ts` at depth and not `/d/A.TS`, that `glob: "*.{ts,tsx}"` matches nothing and the zero-match line carries the brace hint, and that `type: "ts"` matches `.tsx`. Assert the zero-match sentence about ignore rules.

**Confidence.** Medium. Cursor, Glob's evidence and four harnesses agree on pruning and on not following links. Applying `.gitignore` without a repository is my judgment, and the choice between the `ignore` crate and a small matcher is open.

### Decode text as UTF-8 with replacement, strip a BOM, and split lines the way Read does

**Recommendation.** Read each file with `Access::read` and never with `read_string`. Decode it with the ladder `Read` uses ([master-read.md](master-read.md), "Decode lossily, hide the byte order mark, and say when bytes were replaced" and "Refuse binary content by sniffing the first 8,192 bytes"). Drop a leading UTF-8 byte-order mark. Decode a file that starts with `FF FE` or `FE FF` with `String::from_utf16_lossy`. Skip any other file with the reason `binary` when any of its first 8,192 bytes is NUL, or when at least 5% of those bytes are control bytes other than tab, LF and CR. Decode every other file with `String::from_utf8_lossy`, count each file that needed a replacement, and say so: `[Note: invalid UTF-8 in 2 files was shown as U+FFFD: /a.txt, /b.txt]` (`1 file` for one). Put `decode`, `sniff_binary`, `split_lines` and `clip_to_boundary` in one `plugin-std` module that Read and Grep both call, so the two cannot drift. Split on `\n` and strip one trailing `\r` from each line. A file whose only terminator is `\r` is one line. `a\nb\n` has two lines, and a zero-byte file has none. Apply no Unicode normalization.

**Why.** `read_string` fails a whole file on one bad byte, so a Latin-1 file's ASCII lines would be lost, as they are in rig. A lossy decode finds them and shows the damage. Using Read's line rule makes a line number from Grep the line number Read prints, so the model can read what it found (Accuracy). Grep is a lossy view whatever is done, so the description must say to read the file before editing it. A UTF-16 file with a mark is decoded as Read decodes it, so a Grep line number and a Read line number name the same line. A UTF-16 file without a mark is binary here, as it is in Read.

**Evidence.** Cursor decodes UTF-8, shows U+FFFD for a bad byte, and runs a 2 s repair pass that depends on a detector outside the bundle ([tool-grep.md](tool-grep.md), Q3). Zed decodes a BOM, then a UTF-16 guess, then `chardetng`. Rig skips a non-UTF-8 file whole and keeps a BOM in line 1. Pi, OpenCode and Cline lose, garble or fail on a non-UTF-8 line. No harness applies normalization, so `é` as U+00E9 and as `e` plus U+0301 do not match each other (probe, ER). Cursor splits on `\n`, strips one `\r`, and treats a lone-CR file as one line; Zed turns lone CR into a line break; rig's `str::lines` leaves it inside the line (Q4). `Access::read_string` returns `VfsError::NotUtf8` with the offset of the first bad byte (`operations.rs`, source).

**Without it.** With `read_string`, one Latin-1 file in a tree makes Grep skip it silently or fail. With a line rule that differs from Read, `Grep` says line 41 and `Read` shows a different line 41.

**Test.** Seed `caf\xE9 NEEDLE` and assert a match on line 1 whose text holds U+FFFD, plus the notice. Seed a file starting with a BOM and `NEEDLE first`, search `^NEEDLE`, and assert a match on line 1 with no BOM in the output. Seed `a\r\nNEEDLE\r\n` and assert line 2 with no `\r`. Seed a file `first\rNEEDLE\rthird` and assert one match on line 1. Seed UTF-16LE text with a BOM and `NEEDLE` on line 2 and assert one match on line 2 with no BOM in the output. Seed UTF-16LE text without a mark and assert one binary skip. Seed a file of 100 bytes with 5 control bytes and no NUL and assert one binary skip, and one with 4 control bytes and assert a match. Seed `é` as two code-point sequences and assert that each pattern matches only its own file.

**Confidence.** Medium. The lossy decode and the line rule have wide support, and the ladder is the one [master-read.md](master-read.md) settles. The shared module is not written, so no test has run the ladder over a search.

### Refuse bad arguments with messages that name the parameter and the fix

**Recommendation.** Build every argument error with one helper, in the shape of the shared check in `check.rs`: `std_Grep:`, the parameter in backticks, then the problem. Check in this order and stop at the first failure. An empty pattern: ``std_Grep: `pattern` is empty. Pass "." to match every line.`` A negative context value: ``std_Grep: `-A` is -1. Pass 0 or more context lines.``, with the key and the value filled in, for `-A`, `-B` and `-C`. A negative `head_limit` or `offset`: ``std_Grep: `head_limit` is -5. Pass 0 or more.`` An unknown `type`: ``std_Grep: Unrecognized file type "foo". Use a type such as js, ts, py, rust, go or java, or filter with glob.`` An invalid `glob`: ``std_Grep: `glob` is invalid: "[" does not parse: <reason>.`` Then compile the pattern, which fails with the message of Behavior 5, then `stat` the `path`. A directory is walked. A file is searched alone: no ignore rule applies to it, and `glob` and `type` do not filter it, because the model named it. An absent path answers `Path does not exist: {path}`, worded by the shared `refusal` function. A message that names `path` echoes it as the model wrote it, while result lines print the virtual path with its leading `/`. Treat an empty `path`, `glob` or `type` as absent. Return these as errors and never as success text.

**Why.** The first two strings are Cursor's own, under the prefix that every Runvil argument error carries. A model that sends `head_limit: "5"` and one that sends `head_limit: -5` then read one style, and a round with several failed calls shows which tool refused. The pattern check comes first so that a call with several faults reports the one that makes the rest moot. Cursor checks the raw context value before it clamps, so only the rejection is visible, and a clamp would turn `-A: -5` into a search with no context while the model thinks it has some. Runvil's `check` has no `minimum` and accepts any number, so the Pack must do these checks itself (Accuracy). Cursor's three empty-string rules disagree: an empty `path` means the root, an empty `type` means no filter, and an empty `glob` returns nothing silently. One rule is simpler and cannot produce a silent empty result.

**Evidence.** The strings and the order come from the bundle: `pattern is empty. Pass "." to match every line.` then the negative-context loop, with the `fIe` clamp applied later (bundle, probe, I re-read the empty-pattern message; [tool-grep.md](tool-grep.md), "The tool body"). The empty-string rules are from Q6 of the same report. Cursor rewrites an invalid regex as `rg: regex parse error: invalid regex "<pattern>": <reason>.`, and the same report's failure matrix lists the `type` and `glob` errors as ripgrep's stderr passed through. Runvil's error messages are meant to be "concise, factual, and naming the parameter and the value" (plan, source). The `std_` prefix is the shape that `check.rs` builds for every argument error it finds (source).

**Without it.** `-A: -5` clamps to 0 and the model believes it asked for context. `glob: ""` returns no matches and the model concludes the file is absent. A `type` typo returns nothing instead of a list of valid types.

**Test.** Assert each string above byte for byte, prefix included. Send `{"pattern": "", "glob": "**/*"}` and assert the pattern error. Send `glob: ""`, `type: ""` and `path: ""` and assert the result equals the call without them. Send `-A: -1` and assert the string. Send `head_limit: -1` and `offset: -1` and assert theirs. Send `type: "foo"` and assert the error names the type. Search a file named as `path` and assert its matches, and name an ignored file as `path` and assert it is searched. Send `path: "x"` for a missing file and assert the message shows `x`, not `/x`.

**Confidence.** High. The two main strings come from the shipped bundle and the rest are small changes in the same shape.

### Say in the description what the tool does: dialect, limits, skips and ignore rules

**Recommendation.** Replace the tool description with text that states the pattern dialect, the order, the two limits, what is skipped, and the ignore rule. Fix the `glob` text, set the `path` text to `File or directory to search in, as an absolute path. Defaults to the run's filesystem root.`, add a `literal` description, and add "0 means no limit" to `head_limit`. The `glob` text names Glob's grammar and no braces: `Glob pattern to filter files, for example "*.js" or "**/*.tsx". Only * and ** are special, and matching is case-sensitive. A pattern that does not start with "/" is matched at any depth. A pattern that starts with "/" is a full path.` A starting text:

```text
Searches file contents with a regular expression and reports the matches. Use it to find where a symbol, string, or pattern occurs, and read the matching file before changing it.

- Patterns use Rust regex syntax, which has no look-around and no back-references. Escape ( [ { . with a backslash, or set literal to true to match the text as written. Matching is case-sensitive unless -i is set.
- content shows each match as N:text and each context line as N-text under the file's path, where N is the line number Read shows, counting from 1. The N: and N- are not part of the line. files_with_matches lists paths. count gives matching lines per file. Results come in path order, the same order Glob uses.
- head_limit counts matching lines, files or count rows across all files, and 0 means no limit. Output stops at 15000 bytes whatever head_limit is, and the last line says how to continue with offset, or to narrow the search.
- Files named by .gitignore, the .git directory, symbolic links, binary files (the ones Read refuses) and files over 16777216 bytes are not searched, and the result says how many were skipped and why. Name an ignored directory in path to search it.
- A line over 1000 bytes is shown as a window around its first match.
- Paths are absolute from the root of the run's filesystem, and work unchanged as the path argument of the other file tools.
```

**Why.** A model acts on the description and nothing else. Cursor's text names `ripgrep`, offers `"*.js"` as `".js"` and `"**/*.tsx"` as `"**/.tsx"`, and says results are "capped to several thousand output lines", which understates the 2000-entry cap and overstates the 15,000-character budget ([tool-grep.md](tool-grep.md), Decision 7). The dialect, the caps and the skip report are what let the model recover in one round (Accuracy, Bounded cost). The shipped description is Runvil's own short text and already removed the `rg` and `Shell` sentences, so this is an extension and not a reversal.

**Evidence.** The three defects in Cursor's description are in the evidence report (Decision 7), as is the `(unsaved)` claim that no code path implements. The shipped `grep.json` description is two sentences and states no dialect, limit or skip (`schemas/grep.json`, source). Rig's description says "Skips files ignored by git" while `*` brings ignored directories back, which is the same defect from another side (probe, ER).

**Without it.** A model that is not told the dialect sends look-around. A model that is not told about skips reads `No matches found` as proof of absence.

**Test.** In `spec-tests.rs`, assert that the description contains `Rust regex`, `literal`, `15000`, `16777216`, `.gitignore` and `skipped`, and does not contain `ripgrep`, `rg `, `several thousand`, `--glob` or `Shell`. Assert that the `glob` text's examples are `"*.js"` and `"**/*.tsx"`, that it holds no `{`, and that the `path` text says `absolute`. The existing test that refuses a description naming PDFs, the Shell tool or the Cursor workspace root stays.

**Confidence.** High for removing false claims and stating the limits, because each limit is a number the code owns. Medium for the exact wording, which no model has been tested against.

## Where Runvil should differ from Cursor

Cursor's Grep is a ripgrep child process behind a zod schema and a proto result. Part of its design is machinery for that arrangement, and part is a choice that Runvil can make otherwise. Table 10 marks dropped machinery as "Drop" and changed behavior as "Change".

| What Cursor does | What Runvil should do | Why |
|---|---|---|
| Spawns `rg` with a built argv, reads NUL-separated output (`-0`) and parses it, so that a path with a colon survives | Drop. Search in process with `find` and return typed data | The child process is the only reason for the argv, the wire format and the parser (Trap 4 of the evidence report is the colon split) |
| Caps for the pipe: 8,388,608 bytes of stdout, a 2000 or 4000 line budget, a 10,000 line parse window, `--encoding`, a `--sortr` fallback | Drop all five | "Five of the tool's caps exist only because ripgrep is a child process" ([tool-grep.md](tool-grep.md), What Runvil can support today) |
| Three units for limits: bytes (stdout, long-line cut), characters (preview), UTF-16 units (15,000 render budget) | Change. One unit, UTF-8 bytes, for every cap | One number the model can reason about, exact in Rust |
| Cuts at 2000 parsed entries and prints a notice on exactly 2000, `Showing 2000 of at least 2000` | Change. Collect one entry past the page and print a notice only when something follows | The notice is false at an exact fit in six harnesses (Behavior 6) |
| Strips an undeclared key without a word (zod default `strip`) | Change. Refuse it by name through the shared unknown-parameter message, accept `-n: true` | `-n: false` and `context: 3` silently do nothing there |
| Coerces `"50"` to 50 and `"yes"` to true in three small functions | Drop in Grep. Leave coercion to the crate-wide argument check, where `check.rs` refuses strings today | One place owns the policy for all six tools (Risks) |
| Drops `-C` when `-A` or `-B` is present | Change. `-A` and `-B` override their own side | No supplied value is silently ignored (Table 7) |
| `--iglob` beats `.gitignore`, and a host flag can add a guard against `**/*` and `**/**` | Change. Ignore rules always win; no guard | The guard exists only to patch the override (Behavior 10) |
| The `glob` filter ignores case, accepts braces, and matches a pattern with no `/` at any depth | Change. Use Glob's grammar: `*` and `**` only, case-sensitive, a relative pattern at any depth, a leading `/` a full path | One parameter name has one meaning in both tools. The cost is that `*.JS` no longer finds `a.js` and `*.{ts,tsx}` no longer expands, until the VFS gains `{a,b}` and a case flag (Behavior 10) |
| `--follow` and `--hidden` always on, `.git` searched, links followed out of the tree | Change. Never follow, skip `.git`, search other hidden files | `--follow` read a link's target before the result filter ran (Q7) |
| `.gitignore` inert outside a git repository, plus `.cursorignore` | Change. `.gitignore` always applies. `.cursorignore` is Cursor's own file, so Runvil's ignore-file name is the Glob report's decision | Rules should come from files the model can see, not from a hidden `.git` |
| Newest first by `mtimeNs` after a 2000 ms stat pass, then walk order | Change. Full-path byte order, no stat | Replay (Table 5) |
| A 25 s clock per root, and an indexed-search provider that falls back silently | Change. A work budget of files and bytes, and no provider | A clock cut is different bytes on a slower machine |
| A five-entry cache keyed on every field except `head_limit` and `offset` | Drop. Re-run for every page | A cache serves a tree that no longer exists (Table 4) |
| Binary file invisible in content mode, visible in files and count mode | Change. Skip it in every mode and count it | One rule, and the model is told |
| An unreadable subdirectory is tolerated for four error numbers and not reported. Another number, such as 3 for a dangling junction, fails the whole call | Change. Skip any unreadable entry and report it | I hit the whole-call failure live (Behavior 3) |
| `--max-columns 1000` with a preview of the first 1,000 characters | Change. A window around the first match | The match can be missing from its own line (Table 9) |
| Invalid UTF-8 shown as U+FFFD, then a 2 s repair pass through a detector outside the bundle | Change. U+FFFD and a count, with no repair pass | The repair depends on code nobody can read, and it is a clock budget |
| A lone CR file is one line | Keep, by following Read's rule | Line numbers agree with Read |
| `<workspace_result workspace_path>` envelope, multi-root results, a worktree guard, the `(unsaved)` and `activeEditorResult` machinery, a transcript-path log, a semantic-search reminder after five alternatives | Drop all of it | Runvil has one root `/` and no editor buffers, and the `(unsaved)` claim has no code behind it ([tool-grep.md](tool-grep.md), "What remains unknowable") |
| Per-model name and description variants (`Grep`, `grep`, `rg`) and a proto result with three cases | Drop. One name, one description | The variants serve Cursor's model routing and its wire protocol |
| Paths such as `.\src\a.ts` when `path` is omitted on Windows, and relative strings otherwise | Change. Print virtual paths with their leading `/` | `Read` and `StrReplace` take the same form |
| Ripgrep's stderr rewritten into `rg: regex parse error: ...` | Change. Own text from the `regex` error, with the dialect and the `literal` hint | The message names the dialect Runvil actually runs |
| No literal mode | Add `literal` | `a.b` matches `axb` with no error today |

*Table 10. Where Runvil's Grep should differ from Cursor's. Sources: [tool-grep.md](tool-grep.md), sections as named; my probes.*

Confidence for each row is the confidence of the behavior it names.

## Mapping to Runvil

Every behavior has Vinnie as its owner, since he writes `plugin-std`. The next step names a todo id from the standing plan, [2026-10-27-2-plugin-std-file-tools.md](../../../runvil/vibe/2026-10-27-2-plugin-std-file-tools.md). Table 11 maps each behavior onto `Access`. Status is covered when `Access` already does the work, partial when it does part, and missing when it does none. Grep's todos run in this order: `grep-schema-fix`, `refusal-text`, `read-real` (it writes the shared `text` module), `glob-real` (it writes the shared ignore module), `grep-real`, then `vfs-glob-gaps`.

| Behavior | `Access` method | Status | What the gap needs | Owner | Next step |
|---|---|---|---|---|---|
| 1. Pack `find`, sliced and yielding | `list`, `read`, `stat` | Covered | A yield future (std only), a `Limits` struct, a `Found` type, a `render` function. No VFS change | Vinnie | `grep-real` |
| 2. Schema, `literal`, undeclared keys | none | n/a | Edit `schemas/grep.json`. The undeclared-key refusal is body code that calls the shared unknown-parameter helper beside `check`, so it ships with `grep-real` | Vinnie | `grep-schema-fix`, then `grep-real` |
| 3. Skips counted by reason | `list`, `read`; `VfsError::PermissionDenied`, `Io`, `OutsideRoot`, `Conflict`, `NotFound` | Partial | A non-UTF-8 name fails the whole `list` of its directory on the real backend, so one such name hides its siblings. The Pack can only count the directory. `Access::list_with_skips` from `vfs-glob-gaps` removes this limit | Vinnie | `grep-real` |
| 4. Full-path order, work budget | `list` (sorted by name on both backends), `Entry.stat.size` and the entry's file type | Covered | The walk's own recursion, a one-line sibling key, and a `Limits` value. No clock | Vinnie | `grep-real` |
| 5. `regex` dialect, `literal` | none | n/a | Add `regex.workspace = true` to `plugin-std`; the workspace already has `regex = "1"`. The VFS crate stays dependency-free | Vinnie | `grep-real` |
| 6. `head_limit` and `offset` | none | n/a | A counter in the Pack's collector | Vinnie | `grep-real` |
| 7. Re-run paging and notices | none | n/a | The notice strings of Behavior 7, a `Found` that records why it stopped | Vinnie | `grep-real` |
| 8. Byte budget, long-line window | none | n/a | Window arithmetic on the match span the `regex` crate returns, with the start and end snapped to character boundaries through `clip_to_boundary` in the shared `text` module | Vinnie | `grep-real` |
| 9. Render, context | none | n/a | A renderer that merges ranges from the text already held | Vinnie | `grep-real` |
| 10. Ignore, links, `glob`, `type` | `read` for `.gitignore`; `list` reports `Symlink` | Missing | The VFS has no ignore concept ([2026-10-27-2-plugin-std-file-tools.md](../../../runvil/vibe/2026-10-27-2-plugin-std-file-tools.md), Decision Record). One Pack module for `.git` and `.gitignore` that Glob and Grep both call, built on the `ignore` crate or a small matcher (open item in Behavior 10), and the VFS glob matcher as one pure function for the `glob` filter | Vinnie | `glob-real` writes the module and exposes the matcher, `grep-real` calls both |
| 11. UTF-8 with replacement, BOM, lines | `read` (bytes) | Covered | Use `read`, never `read_string`. Call the shared `text` module (decode, binary sniff, split lines) that `read-real` writes and Read also calls | Vinnie | `grep-real` |
| 12. Argument messages | none | n/a | `check.rs` has no `minimum`, so the Pack checks negatives itself. One helper builds each `std_Grep:` error | Vinnie | `grep-real` |
| 13. Description | none | n/a | Edit `schemas/grep.json` and `spec-tests.rs` | Vinnie | `grep-schema-fix` |
| Claims for the walk | `claim_list` per directory, `claim_read` per file, taken inside `list` and `read` | Covered | A `Conflict` becomes a skip. No new claim kind in version one | Vinnie | `grep-real` |
| One event for the search | `Op::List` and `Op::Read` per entry | Partial | There is no `Op::Search`, so the sink sees one event per directory and per file. I did not measure a sink | Vinnie | `grep-real`, revisit with the VFS search |
| A range read, to bound a large file | `VfsAccess::read_range` exists below `Access` | Missing on `Access` | Version one does not need it, because the listing gives `stat.size` before the read. Add it if a streaming read is wanted | Vinnie | none now |
| Output trust | `ToolOutput::untrusted` | Covered | Mark any text that holds a path or file text untrusted. A bare `No matches found` line may be trusted | Vinnie | `grep-real` |
| A policy that asks on reads | `VfsError::ApprovalRequired` | Partial | A refused root is an error that the shared `refusal` function words. A refused entry inside the walk is a skip with the reason `denied by policy` | Vinnie | `refusal-text` |
| The VFS search of option A | none | Missing | The five items in the Options section. Build it only when a trigger there fires | Vinnie | none now |

*Table 11. How each behavior maps onto `Access`, with the gap, the owner and the plan todo.*

The tests to add to the crate follow, one per file the code will have. The crate's rule is a `<stem>-tests.rs` sibling for each module and no Rust file over 500 lines, so I expect the code to split into `grep.rs`, `grep-find.rs`, `grep-render.rs`, `grep-ignore.rs` and `grep-text.rs`. The last two are thin calls into the shared ignore module that `glob-real` writes and the shared `text` module that `read-real` writes, so their tests check Grep's use of each.

- `spec-tests.rs`: the thirteen parameter names in order, the five fixes, the description strings, and the `glob` and `path` text (Behaviors 2 and 13).
- `grep-find-tests.rs`: full-path order on `/B`, `/a-b`, `/a.txt`, `/a/x`, `/b`, matched against Glob's order for the same tree; the lowered work limits; polled `Pending` counts and a drop after the first `Pending`; two Grep calls polled together; a source check that the module names no clock (Behaviors 1 and 4).
- `grep-find-tests.rs`, skips: a policy that denies one directory and one file; a binary file; a file over 16 MiB; a conflict from a sibling write; the skip line and the zero-match coverage line (Behavior 3).
- `grep-count-tests.rs`: `head_limit`, `offset`, `0`, exact fit, and page joins at 25 matches (Behaviors 6 and 7).
- `grep-render-tests.rs`: the worked example, `-C` with `-A`, merged context, the last 30 bytes, count mode with a colon in the name, the byte budget, and the window around a match at position 1,001 (Behaviors 8 and 9).
- `grep-ignore-tests.rs`: the `.gitignore` tree, an explicit `path`, `glob` and `type` cases, a link, and `.git` (Behavior 10).
- `grep-text-tests.rs`: Latin-1, BOM, CRLF, lone CR, UTF-16, and the two normalization forms (Behavior 11).
- `grep-args-tests.rs`: every string in Behavior 12, the order, the empty-string rule, and the undeclared-key refusals (Behaviors 2 and 12).
- `tests/it/tools.rs`: flip the Grep entry from the stub text to real behavior, and add a test over a real directory with `RealBackend::rooted`, because `TestCall` offers only an in-memory store and no sink.

The schema and description fixes, in order, each recorded in `schemas/EDITS.md`:

1. Delete `-n` and `context`, and give `-C` the text that was on `context`.
2. Set `output_mode` to default to `content`, in the text and in a `"default"` key.
3. Remove the `250` default from `head_limit`, add `"minimum": 0`, and add "0 means no limit".
4. Do the same for `offset`, with `"minimum": 0`.
5. Add `literal` with the text "Treat pattern as plain text, not a regular expression. Default: false."
6. Correct the `glob` text to the examples `"*.js"` and `"**/*.tsx"` and to Glob's grammar, with no braces and no case folding (Behavior 13).
7. Set the `path` text to `File or directory to search in, as an absolute path. Defaults to the run's filesystem root.`
8. Replace the tool description with the text in Behavior 13.
9. Update the `grep-real` todo. Replace "Decide first whether the VFS gains a search capability or the Pack walks with `Access::glob` and reads with `Access::read_string`, because `Access` has no grep" with "Walk in the Pack with `Access::list` and `Access::read`; no VFS search yet (`master-grep.md`)", and note that `glob-real` owns the shared ignore module and `read-real` the shared `text` module.
10. Take Grep's refusal text from the proposed `refusal-text` todo, which lands before `read-real`. The `approval` todo then only wires real approval.
11. Dependencies: `regex` comes from the workspace, `ignore` joins it only if the open item in Behavior 10 takes the crate, and `globset` is not added.

Confidence in these plan edits is high, because they only make the plan text match the behaviors above. The new todo ids are proposals, confidence medium.

## Risks and open limits

Each row of Table 12 gives the likelihood of the event and, separately, my confidence in the evidence behind it. The last column says what the event would change.

| Event | Likelihood | Confidence in the evidence | What it would change |
|---|---|---|---|
| On Windows, a Grep over a project tree of 10,000 files or more takes over 2 s on the real backend, because each `Access` call pays containment | Likely | Medium: one machine, one backend, and I assume linear scaling | Build the VFS search of option A. This is the trigger |
| The Harness does not honor a yield, or a stop does not reach the future at a yield | Unlikely | Low: no source shows a Pack that yields, and I ran no Harness | Option A with a resumable step, since a Pack cannot hand the `Access` to a thread |
| In a code repository, one non-UTF-8 name hides every sibling in its directory, because `list` fails whole | Unlikely | Medium: read in `real/files.rs`, not run | `Access::list_with_skips` from `vfs-glob-gaps`, which returns the entries and a `Skipped` list of path and reason, shared with `Access::glob_with_skips` |
| In a prompt's store, an in-memory search over 100,000 or more files takes seconds, because `MemoryBackend::list` scans every key | Very unlikely | High: I measured 2.06 s at 200,000 files | Index the memory backend's children, or use `glob` for a memory store |
| The `ignore` crate is refused or judged too heavy, because the plan allows no new dependency | Roughly even chance | Low: I did not read the dependency policy beyond the plan | Write a small `.gitignore` matcher and a type table, which the open item in Behavior 10 costs |
| Models trained on Claude Code send `context` or `-n` and meet a refusal | Likely | Low: no transcript counts it | Accept `context` as an alias for `-C` |
| Models never send `literal` | Likely | Low | Nothing: the parameter is additive |
| The chosen numbers (15,000, 1,000, 100,000 files, 1 GiB, 16 MiB, 32 files per slice, 8,192) are wrong by some factor for real use | Very likely | Low: they are my judgment | Change one `Limits` value and its test |
| In a round that mixes a Grep and a sibling Write, the Write makes the Grep skip a file, because a read or list claim and a write claim conflict in either order while the scope lives | Roughly even chance | Low: I read the claims and ran no round | A skip line names it; no change unless it is common |

*Table 12. What could go wrong with the recommendation, with likelihood and confidence kept apart.*

Four things the evidence does not settle. First, I ran nothing on Linux or macOS, so the 140 us of containment per call is a Windows figure, and `realpath` is cheaper elsewhere, which would favor the Pack route. Second, I ran no tree over 20,000 files and no Harness, so the 40 s at 200,000 files is an extrapolation and the yield is untested. Third, I read the claims code and did not run two concurrent searches; the statement that `claim_subtree` conflicts with a concurrent Read comes from reading `check_read` and `check_subtree`, and no failing test shows it. Fourth, one correction to the evidence report, now made there: it called Cursor's argument object strict, and the bundle, with `Ikc` resolved to zod's `object` factory, and one live call say it strips undeclared keys. The recommendation does not depend on which is right, but the Differ table row does.

Four events would change the main recommendation, in the order I expect them. First, a deployed run shows Greps over trees the model routinely uses costing more than 2 s. Second, the Harness cannot yield. Third, the VFS gains a read-kind subtree claim for other reasons, which removes one of the three costs of option A. Fourth, a backend that is not local needs the search pushed down to it. Any one of them moves the weight to option A, and the `find`/`render` seam means the move changes the walk and nothing above it.

A `Conflict` that the Pack maps to a `ToolError` reaches the model as failure text and does not end the run (`scheduler/tool_call.rs`, module doc, read, not run). Grep turns a conflict inside the walk into a skip, so only a conflict on the search root takes that path.

Two smaller open items. Cursor's `Grep` also takes a `sort` parameter in its proto that no schema reaches, and Runvil has no use for it. Zed's syntax-aware context and Gemini CLI's auto-context for one to three hits are real ideas that a later version could take. Neither blocks version one.

## Sources

**Evidence reports.**

- [tool-grep.md](tool-grep.md), the Grep evidence report, 3,450 lines, enriched with twelve harnesses. Cited by section name throughout.
- [file-tools-index.md](file-tools-index.md), the cross-tool facts and the corrections to the parent spec.
- [tool-glob.md](tool-glob.md), Decisions 1 to 4 and 6, for the ignore rule, the order and the glob grammar that Grep shares.

**(bundle) and (daemon).** Cursor 3.27.12, `cursor-agent-exec/dist/main.js`, 9,814,287 bytes, read as latin1, and the unminified twin `agent-host-daemon/dist/bin/daemon.cjs`. I re-read these offsets. The schema `wIe` is at 4311435 and the broad-glob set `uIe` at 4310786. The factory `_Ie` is at 4315448, with its `enableBroadGlobGuard` option at 4315514 and the host flag `featureFlags.enableGrepBroadGlobGuard` at 4592432. The `head_limit` and `offset` coercion is at 4320682, the empty-pattern message near 4319759, the `-C` guard at 9158869, the case flags at 9159198 and `--max-columns` at 9158817. The executor constants are at 9173743, the stat budget at 9070633, and the zod `object` factory that `Ikc` names, which is `function ye` at 1603217 inside module 77602 (its export list is at 1587505) and sets no catchall, so it strips unknown keys. Everything else from these two files comes through the evidence report.

**(source), Runvil.** The `runvil` repository at commit `08801150fc67` ("Add the std Pack with Cursor's six file tools", 2026-10-27):

- VFS crate, `crates/runvil-internal/vfs`: `Cargo.toml`, `src/lib.rs`, `src/error.rs`, `src/stat.rs`, `src/glob.rs`, `src/memory.rs`, `src/observe.rs`, `src/real.rs`, `src/real/files.rs`, `src/real/resolve.rs`, `src/router.rs`, `src/handle/operations.rs`, `src/handle/claims.rs`, `src/handle/claims_tree.rs`, `src/traits/backend.rs`, `src/traits/policy.rs`.
- Plugin contract and Harness: `crates/runvil-plugin/src/pack.rs`, `context.rs`, `testing.rs`; `crates/harness-internal/runner/src/performers.rs`.
- The crate under work: `crates/plugin-std/schemas/grep.json`, `schemas/EDITS.md`, `src/check.rs`, `src/pack.rs`, `Cargo.toml`; the workspace `Cargo.toml` (`regex = "1"`).
- The standing plan, `vibe/2026-10-27-2-plugin-std-file-tools.md`, for the todo ids and the Decision Record.
- For the dependency cost in Behavior 10: the workspace `Cargo.lock`, and the `ignore` 0.4.33 manifest and sources in the cargo registry, read and not built.
- For the claim wording and the Conflict sentence: `crates/runvil-internal/vfs/src/handle/claims_tree.rs` as read again for this pass, and the module doc of `scheduler/tool_call.rs` in the Engine.

**(source), other harnesses.** The evidence report read these at the commits it names, and I cite them through it. I did not re-read them for this report.

- Claude Code 2.1.311 `tools/grep.js`; Copilot `microsoft/vscode` at `b208bc25`; Zed at `37faef81f7`; pi at `a92283056`.
- rig at `a83b774d0`; zeroclaw at `2d85371f3`; Codex CLI at `bea47c06dfac`; Gemini CLI v0.66.0 at `970413207732`.
- OpenCode v1.19.5 at `810c0eba1a73`; Cline v4.1.26 at `a459f1d`; Aider at `5dc9490bb35f`; Goose at `ee8500d5e060`.

**(probe), mine.** All run for this report on Windows 10.0.26300, one machine, warm file cache, Rust 1.99.1:

- Claims and memory cost on `MemoryBackend`, 20,000 and 200,000 files of 3,300 bytes, with a counting allocator, over the `runvil-vfs` crate by path.
- Search cost on `RealBackend::rooted`: 20,000 files, 78,793,400 bytes, `list` walk, `glob` walk, `Access::stat` and `Access::read` timings, a raw `std::fs` walk, and an early-stop walk. The shipped `rg.exe` 15.1.0-cursor5 on the same tree with the Grep argv, five runs.
- `Access::list` and `Access::stat` over a good and a dangling directory junction.
- `regex` 1.13.1: error texts, escaping, the size limit, linear-time growth and CRLF anchors.
- `ignore` 0.4.33 and `globset` 0.4.20: gitignore matching on virtual paths, the 224-entry type table, the unknown-type error and brace globs.
- Cursor bundle string and offset checks, listed above, including the resolution of the minified alias `Ikc` to zod's `object` factory.
- A walk-order check on 20,000 random trees: a depth-first walk that sorts each directory by the key `name` for a file and `name/` for a directory, compared with a plain byte sort of the full paths. The keyed walk matched all 20,000 and the bare-name walk differed on 334.
- Live Grep calls in this session: the footer after a `head_limit` cut (two calls), an undeclared key with `-n: false` (one call), and a whole-call failure on a dangling junction (two calls and one direct `rg.exe` run). The Cursor build behind the live calls was not recorded.

I did not use a (web) source. The ripgrep and zod behavior I cite comes from the bundle and from runs.

*2026-10-27 - Claude Sonnet 5.5*
