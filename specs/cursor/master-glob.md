# Ship Glob as a recursive, case-sensitive path-order lister that pages with `offset`, never answers a bare zero, and hides ignored paths through one rule that `Access` will own

## Summary

I recommend that Runvil ship `Glob` as a thin Pack over `Access::glob`: a recursive, case-sensitive, path-ordered lister that pages with `offset`, never answers a bare zero, and hides ignored paths through one rule that `Access` will own.

First, sort every result by full-path bytes. Cursor's modification-time sort degrades past a 2000 ms budget, so only a total order replays and pages. Second, after the match, hide `.git` and every path a `.gitignore` names, through one Pack module that Grep shares, and count them. A positive glob overrides `.gitignore` in Cursor and rig, so the rule runs after the match. A directory the model names is still searched. Third, cap a reply at 15,000 bytes of paths, shared with Grep, add `offset`, and print a notice only when paths remain, because six harnesses announce a cut at an exact fit. Fourth, prepend `**/` to relative patterns, pass a leading `/` through as a full VFS path, and match case-sensitively. Fifth, explain every zero, since `Access::glob` answers a missing directory with an empty list and reads `{` literally.

The largest risk, at roughly even chance, is that the shared ignore module misreads a `.gitignore` rule and hides a wanted file or lists a generated tree. Confidence is high for order, recursion and the zero reply, and medium for ignore handling, paging and case. The next step is `glob-real`, plus the VFS todo `vfs-glob-gaps` for the filter, partial results and `{a,b}`.

## Contents

- [The recommendation in brief](#the-recommendation-in-brief)
- [Criteria](#criteria)
  - [Confidence rates the evidence and likelihood rates the event](#confidence-rates-the-evidence-and-likelihood-rates-the-event)
- [Options and the do-nothing case](#options-and-the-do-nothing-case)
  - [Call `Access::glob` and let the Pack own the words, because a subprocess cannot see a virtual filesystem](#call-accessglob-and-let-the-pack-own-the-words-because-a-subprocess-cannot-see-a-virtual-filesystem)
  - [Sort by path bytes and nothing else, because only a total order with no clock replays and pages](#sort-by-path-bytes-and-nothing-else-because-only-a-total-order-with-no-clock-replays-and-pages)
  - [Filter in `Access` after matching and announce the count, because an override layer lets a pattern resurrect what a user ignored](#filter-in-access-after-matching-and-announce-the-count-because-an-override-layer-lets-a-pattern-resurrect-what-a-user-ignored)
  - [Cap the list by bytes and page with `offset`, because one unit and one notice cannot disagree](#cap-the-list-by-bytes-and-page-with-offset-because-one-unit-and-one-notice-cannot-disagree)
  - [Prefix every relative pattern with `**/`, because three harnesses agree and the anchored forms give false zeros](#prefix-every-relative-pattern-with--because-three-harnesses-agree-and-the-anchored-forms-give-false-zeros)
  - [Match case-sensitively and explain the zero, because the matcher and the grammar already decide both](#match-case-sensitively-and-explain-the-zero-because-the-matcher-and-the-grammar-already-decide-both)
- [Behaviors to implement](#behaviors-to-implement)
  - [Sort every result by full-path byte order, with no clock and no budget](#sort-every-result-by-full-path-byte-order-with-no-clock-and-no-budget)
  - [Put ignore and privacy rules in `Access`, apply them after matching, and count what they hide](#put-ignore-and-privacy-rules-in-access-apply-them-after-matching-and-count-what-they-hide)
  - [Never answer a bare zero: say what matched nothing and the likely reasons](#never-answer-a-bare-zero-say-what-matched-nothing-and-the-likely-reasons)
  - [Page with `offset` under one byte budget, and print the exact total](#page-with-offset-under-one-byte-budget-and-print-the-exact-total)
  - [Print a notice only when paths remain: an exact fit is silent](#print-a-notice-only-when-paths-remain-an-exact-fit-is-silent)
  - [Make every pattern recursive from the search directory](#make-every-pattern-recursive-from-the-search-directory)
  - [Pass a leading `/` through as a full VFS path, and refuse it beside `target_directory`](#pass-a-leading--through-as-a-full-vfs-path-and-refuse-it-beside-target_directory)
  - [Match case-sensitively and say so](#match-case-sensitively-and-say-so)
  - [Accept the grammar `Access::glob` implements, and explain a zero that holds braces, brackets or question marks](#accept-the-grammar-accessglob-implements-and-explain-a-zero-that-holds-braces-brackets-or-question-marks)
  - [Refuse an empty pattern and check the target directory before the walk](#refuse-an-empty-pattern-and-check-the-target-directory-before-the-walk)
  - [List files only, leave links unlisted, and read `X/` as the files under X](#list-files-only-leave-links-unlisted-and-read-x-as-the-files-under-x)
  - [Print absolute VFS paths, bare, one per line, and leave out any name with a control character](#print-absolute-vfs-paths-bare-one-per-line-and-leave-out-any-name-with-a-control-character)
  - [Render every `VfsError` in one function that names the rule, the path and the next step](#render-every-vfserror-in-one-function-that-names-the-rule-the-path-and-the-next-step)
  - [Write the description from the behavior](#write-the-description-from-the-behavior)
- [Where Runvil should differ from Cursor](#where-runvil-should-differ-from-cursor)
- [Mapping to Runvil](#mapping-to-runvil)
  - [What `Access::glob` does today and what it lacks](#what-accessglob-does-today-and-what-it-lacks)
  - [Tests to add to the crate](#tests-to-add-to-the-crate)
  - [Schema and description fixes to make](#schema-and-description-fixes-to-make)
- [Risks and open limits](#risks-and-open-limits)
  - [What the evidence does not settle](#what-the-evidence-does-not-settle)
- [Sources](#sources)

## The recommendation in brief

Table A gives one row per behavior, in the order of Behaviors to implement. The confidence word rates the evidence, and the reason is under each behavior.

| Behavior | Recommendation | Confidence |
|---|---|---|
| Order | Sort by full-path bytes with no stat, clock or locale, and re-sort in the Pack | High |
| Ignore rules | Hide `.git` and the paths a `.gitignore` names, after the match, through one shared Pack module, and count them. A per-match filter in `Access` replaces the module later | Medium |
| Zero reply | Say what matched nothing, with hints for case, syntax, an absolute pattern and hidden paths | High |
| Paging | Add `offset`, fill a page to 15,000 bytes of path lines, and print the exact total | Medium |
| Exact fit | Print a notice only when paths remain beyond the page | High |
| Recursion | Prepend `**/` unless the pattern starts with `**/` or is `**` | High |
| Absolute patterns | Treat a leading `/` as a full VFS path, and refuse it beside `target_directory` | High |
| Case | Match case-sensitively and say so | Medium |
| Grammar | Keep four tokens, explain a zero that holds `{`, `[` or `?`, and add `{a,b}` in the VFS later | Medium |
| Input checks | Refuse an empty pattern, `stat` the target directory before the walk, and refuse an undeclared parameter | High for the first two, medium for the parameter refusal |
| Kinds and links | List files only, leave links unlisted, and read `X/` as the files under X | Medium |
| Path text | Print bare absolute VFS paths with `\n` ends, and leave out names with control characters | High |
| Errors | Render every `VfsError` in one function that names the rule, the path and the next step | Medium |
| Description | Rewrite `glob.json`, add `offset` and correct `EDITS.md` | High |

*Table A. The recommendation for each behavior, with its confidence. Sources: the behaviors below and the options in Tables 1 to 6.*

## Criteria

Six criteria judge every option and every behavior below. Each keeps one name through the report.

- **Loud result.** The model learns the outcome of a call from the call. A zero, a cut or an omission says why, and no outcome is a silent wrong result. A bare "no files" over a tree that holds the file fails this criterion.
- **Replay.** The same call on the same tree returns the same bytes on every backend and host, so a recorded run replays. Anything that depends on a clock, a stat budget, a locale or a walker's arrival order fails. "Same tree" includes the files that hold ignore rules.
- **File safety.** Glob writes nothing, so survival of the user's files is not at stake. What remains is leakage and reach: a listing must not show a name that the mode policy would hide, and must not claim a rule that Read does not apply, and the walk must stay inside the run's root.
- **Bounded cost.** The reply has a size bound that the model is told, and the walk has a cost the report names.
- **Few calls.** The model reaches the files it wants in as few calls as it can. A missed match costs a call, and so does a page the model did not need.
- **VFS fit.** The behavior works on `Access` as it stands, the gap is named when it does not, and each policy is decided once in the layer that owns it: matching and order in the VFS, ignore rules in one shared Pack module until `Access` owns them, wording and paging in the Pack.

When two criteria conflict, the order is Loud result, then Replay, then File safety, then Bounded cost, then Few calls. VFS fit is a constraint rather than a preference. Loud result ranks first because the harnesses in the set fail a model silently far more often than loudly: an arbitrary order after a timeout, a zero over a file that exists, a stop notice at an exact fit. Replay ranks second because a run that cannot replay cannot be tested or cached.

### Confidence rates the evidence and likelihood rates the event

Every recommendation carries a confidence word: high, medium, or low. Confidence rates the evidence behind the recommendation. Likelihood rates an event and uses a fixed ladder: very unlikely, unlikely, roughly even chance, likely, very likely, almost certain. The two never share a sentence.

Evidence carries tier labels. `(bundle)` and `(daemon)` mean read out of Cursor 3.27.12's shipped agent bundle and its unminified daemon twin. `(source)` means read from a pinned commit of another harness or from Runvil's working tree. `(probe)` means measured by running something. `(web)` means a public document. A claim that I read from code and did not run says so in words. Each claim says whether it was observed, assumed or judged, and a judgment is marked as mine. The evidence report is [tool-glob.md](tool-glob.md), cited by section name.

## Options and the do-nothing case

Six decisions have credible alternatives. Each gets a table with the do-nothing option named, the recommended option marked, and a verdict. The remaining choices (absolute path text, the error renderer, the description) have no serious rival and appear only under Behaviors to implement. Confidence for each decision is in the brief table and under its behavior.

### Call `Access::glob` and let the Pack own the words, because a subprocess cannot see a virtual filesystem

Cursor's Glob is `rg --files` with an empty pattern, plus a JavaScript layer that sorts, filters and renders ([tool-glob.md](tool-glob.md), The short version). Claude Code, OpenCode and pi (through `fd`) also shell out, Copilot asks the editor's index, and Zed, rig and zeroclaw match in process. Table 1 lists the choices for Runvil, where `Access::glob` already validates, gates, claims, fires an event and sorts.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **`Access::glob` plus Pack wording (recommended)** | The VFS matches and sorts. The Pack builds the pattern, pages and words the reply | Gate, claim, event and mode policy with no Pack code. Both backends already sort. A linear-time matcher with a 1,024-byte pattern limit | A four-token grammar. No ignore filter, no limit, no metadata. The whole match list is allocated | Recommended |
| ripgrep or `fd` subprocess (Cursor, Claude Code, OpenCode, pi) | A child process over the host disk | The full globset grammar and ignore engine | Cannot see a memory backend or a mount. Semantics set by a pinned third-party binary. Newline splitting, a `.\` prefix on Windows, a timeout | Set aside |
| Pack-side walk with `Access::list` and the Pack's own matcher | The Pack | Room for ignore rules, a limit and an early stop | One claim and one event per directory. A second matcher that must agree with `glob.rs`, and Grep needs it too | Set aside |
| Richer VFS call now: limit, per-match filter, modification time, skip counts | The VFS | Closes most gaps in one place | A VFS design with no plan item, ahead of `glob-real` | The destination, not the first step |
| Do nothing: keep the stub | Nobody | No new surface | A model that knows a name and not a directory cannot find the file at all. The plan defers `ListDir` and the shell, so no other route to a file by name exists | Set aside |

*Table 1. Five ways to build Glob on Runvil. Sources: [tool-glob.md](tool-glob.md), Decision 1 and the Claude Code, Copilot, Zed, pi, rig and zeroclaw sections; Runvil `operations.rs` and `glob.rs`; the standing plan's non-goals.*

The subprocess route fails on Runvil's structure. The VFS is in process and virtual: `MemoryBackend` has no disk, and a mount can sit anywhere in the namespace, so a child process sees none of it `(source)`. The routes that do shell out pay for it in the evidence. Cursor reports a missing ripgrep binary as `Path does not exist: <cwd>`, a filename with a newline splits into two entries (read from the parser, not measured), and Windows output keeps a `.\` prefix `(daemon)` `(probe)`. The evidence report reaches the same verdict in Decision 1.

A Pack-side walk over `Access::list` gives the Pack room to add ignore rules and an early stop. It also takes a claim and an event per directory, and it needs a second matcher beside `glob.rs`. Grep needs the same matching, so any drift would show in two tools. Zed, rig, zeroclaw and Gemini each define `*` differently, so one pattern means different things in each (Decision 1, What the other harnesses add).

A richer VFS call is where most gaps close. Table 9 lists them. Each is real work in the VFS crate and none has a plan item today, so the recommendation does not wait for them. The plan edits under Mapping to Runvil propose one todo, `vfs-glob-gaps`, for them. Doing nothing leaves `Glob` answering "not implemented" on every call. Confidence in calling `Access::glob` is high, because the subprocess cannot reach the VFS and `Access::glob` already supplies the gate, the claim, the event and a sorted list.

### Sort by path bytes and nothing else, because only a total order with no clock replays and pages

Cursor sorts newest first in JavaScript under a 2000 ms stat budget, and past the budget it returns the walk order with no marker `(daemon)`. Claude Code asks ripgrep for `--sort=modified`, which runs oldest first, so under its cap of 100 the files kept are the oldest `(source)` `(probe)`. Gemini lists files modified in the last 24 hours first, so one tree orders differently an hour later `(source)`. Zed and Goose's `tree` sort by path and give the same output on every run `(source)`. Table 2 lists the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **Byte order of the full path (recommended)** | Both backends sort today. The Pack re-sorts as a safeguard | The same bytes on every backend and host. No stat call. `offset` pages stay stable | No help for "the file I just edited". Capitals sort before lowercase. `-` and `.` sort below `/`, so `/a-b/x` precedes `/a.rs` precedes `/a/x` | Recommended |
| Modification time, newest first, under a stat budget (Cursor) | The Pack, one `Access::stat` per match | Matches Cursor's description. Recency first | A gate, a claim and an event per match. A silent fallback past 2000 ms. Order changes under `git checkout` | Set aside |
| Modification time, oldest first, in the engine (Claude Code) | The engine | Unconditional | Under a cap of 100 it keeps the oldest 100 | Set aside |
| Modification time where the backend supplies it, else path (the evidence report's option D) | The Pack | Recency where it exists | A memory backend returns path order and a real backend returns time order, so a run moved between them replays differently | Set aside. A narrowing of the evidence report |
| Path-component order (Zed, Goose) | The Pack re-sorts every result | A directory's files stay together | A re-sort of every result. It differs from what both backends return today. Grep keeps full-path byte order too, by keying each directory's entries with a trailing `/` for directories, so its streaming walk is not a reason to change this | Set aside |
| Do nothing: trust the trait's word "sorted" | The backend | No Pack code | The trait names no comparison, so a custom backend may sort by locale | Set aside |

*Table 2. Six ways to order a Glob result. Sources: [tool-glob.md](tool-glob.md), Decision 3, Q11 and Ordering across the harnesses; Runvil `memory.rs`, `real.rs`, `stat.rs` and `traits/backend.rs`.*

The time-based options lose on cost and on replay. The Pack can read a modification time only through `Access::stat`, which runs a gate, a claim and an event per call, so 2,000 matches mean 2,000 of each, in sequence. `Stat.modified` is an `Option`, and a backend leaves it empty when it does not track the time `(source)`. A time-ordered list also reorders under a checkout. The default `glob_kind` already stats every match inside the backend, without claims, and `Access::glob` throws the result away (I read this in the source and did not run it). A later VFS call that returned those stats would cost nothing extra.

I narrow the evidence report's option D here. Decision 3 chose time order where the backend supplies a time and path order otherwise, and it says that the memory backend and a real backend "will not agree". A run recorded on a real mount then replays differently on a memory mount, which fails Replay on the first move. The report's own Q1 says a path-sorted list pages stably, and its Runvil section says "Return path order". I recommend the path order alone.

Component order is Zed's and Goose's. It keeps `/a/x` ahead of `/a-b/x`, and a depth-first walk over sorted names yields it. Both backends return whole-string order today, so that order costs no Pack code. Grep's streaming walk is no reason to revisit it: Grep keys each directory's entries by name, with a trailing `/` on a directory, so its depth-first walk yields the same byte order of the full path string (`master-grep.md`, Behavior 4, the ordering behavior). Doing nothing trusts a trait that names no comparison, and Gemini's `localeCompare` and Cline's numeric collation show what a custom backend could choose. A re-sort by bytes is cheap on input that is already sorted.

No recency order ships. The evidence report's Decision 7 proposes a session signal from the Pack's own writes at low confidence, and I leave it out for the same reason: no source says how often a model asks for the file it just created. Confidence in the decision is high, because Zed, Goose and Runvil's own backends already produce a total order, and paging needs one.

### Filter in `Access` after matching and announce the count, because an override layer lets a pattern resurrect what a user ignored

Runvil's VFS has no ignore concept, and the plan defers it `(source)`. Cursor's Glob has two precedence tiers: a `.cursorignore` rule beats a positive glob, and a positive glob beats a `.gitignore` rule. So `glob_pattern: "*.log"` returns an ignored `debug.log`, and `*` returns the whole subtree of an ignored directory by matching the directory node. Outside a git repository `.gitignore` does nothing, because `buildArgs` never passes `--no-require-git` `(probe)`. Rig reaches the same split through the `ignore` crate's override layer, which ripgrep's `--glob` also uses. ripgrep's help says the flag "always overrides any other ignore logic", the crate's source says "Overrides have the highest precedence", and rig's tool description still says it skips ignored files `(probe)` `(source)`. Zed's `find_path` applies none of the privacy filters that its read tools apply, so `**/.env*` lists names that `read_file` refuses (I read this in the source and did not run it). Table 3 sets out the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **A filter in `Access`, after the match, with one precedence and a count in the reply (recommended as the destination)** | The VFS, per match, beside `Policy::check` | Glob and Grep agree. No pattern can resurrect an ignored path. No git requirement. The Pack can say how many paths were hidden | A VFS design with no plan item. Today's policy hook sees only the pattern for `Op::Glob` | Recommended as the destination. The shared module below stands in until it lands |
| **One shared Pack module for version one: `.git` and the paths a `.gitignore` names, applied after the match, with a count (recommended for now)** | The Pack, once. Glob calls it after the match and Grep calls it during the walk | The same answer from both tools with no VFS change. Generated trees stay out of the listing. A directory the model names is still searched | A `.gitignore` matcher in `plugin-std`: the `ignore` crate or a small matcher of its own, with costs under the open limits. The VFS filter deletes the module later | Recommended for version one |
| Hide `.git` only, through one shared function, and read no `.gitignore` | The Pack, once | One answer from both tools and no new dependency | Grep then walks `target/` and `node_modules/`, where its work budget runs out. Glob pages through them | Set aside |
| Do nothing: no ignore rule | Nobody | Total determinism and no hidden state | `target/` and `node_modules/` flood a real mount | Set aside |
| A filter in each tool's Pack code, one copy per tool | The Pack, once per tool | No VFS change | Glob and Grep drift. Zed shows the miss: its listing skips filters its read tool applies | Set aside |
| Ignore rules as an override layer (Cursor, rig) | The walker | Familiar to a model that learned ripgrep | `*.log` returns the ignored file. `*` returns an ignored directory's subtree. `.gitignore` is inert outside a git repository | Set aside |
| Tracked files only (Aider) | git's index | Secrets and build output never appear | A new untracked file never matches. Needs git | Set aside |

*Table 3. Seven ways to treat ignore rules. Sources: [tool-glob.md](tool-glob.md), Decision 2, Trap 1, Trap 2, Trap 20 and the Aider and Zed sections; Runvil `policy.rs` and the plan's Decision Record.*

Ignore rules hide noise: `target/`, `node_modules/`, `.git/`. They hide entries from Glob and Grep listings only. Read, Write, StrReplace and Delete act on any path the call names. No privacy rule exists in version one: Read opens any path the call names, so `.env` appears in a listing and opens in Read. A privacy rule is future work and would apply in `Access` to every tool. The ignore rule belongs in `Access` in the end, so Glob and Grep give one answer. Today's hook is `Policy::check(op, path)`, and for `Op::Glob` the path is the pattern. A policy can refuse a whole glob but cannot remove one match from a result `(source)`. A per-match filter beside it is the missing piece.

The rule should win over the pattern. A filter that runs after matching cannot be undone by any glob. That reverses Cursor's `.gitignore` behavior and keeps its `.cursorignore` behavior. The one exception is a directory the model names as `target_directory`, which is searched because the model asked for it. The rule should not need a git repository: Goose passes `require_git(false)` and pi adds `--no-require-git` when no `.git` exists, where Cursor and rig leave `.gitignore` inert outside one `(source)`. If a real-disk backend later uses `ignore::WalkBuilder`, it should take ignore rules from the builder's standard filters and match the pattern with the VFS matcher, and never hand the pattern to the builder as an override.

The count matters. A model that knows a file exists and gets zero concludes the file is gone. The evidence report says Cursor's silence is a documented source of confusion, but the field report it cites is about binary and empty files that `rg -l` dropped `(web)`. That is the same kind of silent omission and is not ignore silence itself, so I rate this decision medium. Gemini counts the files its ignore filter removed `(source)`, which shows the shape: one line, with a count.

Until `Access` owns the rule, one Pack module does the work. `glob-real` writes it and `grep-real` calls it. After the match it leaves out any path that has a segment named `.git`, file or directory, and any path that a `.gitignore` on the way to it names, honoring `!` rules and needing no `.git` directory, and the reply says how many it left out. Cursor excludes `.git` by a literal glob, Gemini by a fixed list and `pi_find` by name `(daemon)` `(source)`. Moving the rule into `Access` deletes the module. Version one hides `.git` and the paths a `.gitignore` names, and nothing else. Grep's cost argument decides it over the `.git`-only row: a generated tree spends Grep's work budget and Glob's page budget alike, and one module serves both tools. Confidence is medium, because two probes and a source read support the destination while the VFS design is unwritten.

### Cap the list by bytes and page with `offset`, because one unit and one notice cannot disagree

Cursor has three caps that interact: 2000 entries returned, 10000 stdout lines parsed and 5000 rendered characters. The character cap binds first, at about 160 files, so a model that finds 400 matches sees about 160 `(daemon)`. Nothing pages. Claude Code caps at 100 results and has an `offset` that it always sends as 0. Zed pages 50 at a time and puts `offset` in the schema. pi caps at 1000 results and 51,200 bytes, rig at 1000 names, OpenCode at 100 paths and Cline's legacy listing at 200, and Gemini has no count cap and a 64 KiB cut `(source)`. Table 4 lists the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **One byte budget on the path lines, `offset` in entries, one notice (recommended)** | The Pack | The budget bounds tokens directly. `offset` makes a cut resumable. Pages are stable because the order is total | A parameter beyond Cursor's two. Page size varies with path length. The VFS still builds the whole list first | Recommended |
| An entry cap only (Zed's 50, Claude Code's 100) | The Pack | Easy to state and to test | Cost follows path length: 200 paths of 150 bytes cost five times 200 paths of 30 bytes | Set aside |
| Two caps, as Cursor has | The Pack | Matches Cursor's cut point | Two notices, and a header count that disagrees with the lines: `(total 5000 files)` over about 160 | Set aside |
| A cap with no `offset` (Claude Code's reachable surface) | The Pack | Keeps Cursor's two-parameter schema | A cut stays cut. The only recovery is a narrower pattern | Set aside. The fallback if the schema must stay Cursor's |
| Do nothing: no cap | Nobody | No cut to explain | A tree of 100,000 files returns 100,000 lines | Set aside |

*Table 4. Five ways to bound a Glob reply. Sources: [tool-glob.md](tool-glob.md), Decision 4, Q1, Q2, Q12 and the Zed, Claude Code, pi and rig sections.*

A path list costs what its bytes cost, and an entry cap hides that. I recommend a budget of 15,000 UTF-8 bytes on the path lines, each line counted as its path plus one newline. The header and the notice lines are not counted. That is about 480 paths of 30 bytes, against Cursor's 5,000 characters and about 160 files. The constant is shared with Grep, whose body cap is Cursor's own 15,000 figure, so a model that finds files with either tool gets pages of the same size for the same names. Nothing in the evidence says what page size a model finds adequate, so I rate the number low and the design medium. Cursor counts UTF-16 code units because it runs in JavaScript. A byte count differs from it on emoji and CJK names, and nothing in Runvil needs the match.

`offset` makes a cut resumable. Cursor's only recovery is a narrower pattern, which fails a caller that wants every match (Q1). Zed's `offset` works because its order is total, and the order here is too. Zed also shows three faults. Its range line prints on the last page, where nothing more exists. Its 1-based range sits beside a 0-based parameter. An offset past the end answers `No matches found`, which is false `(source)`. Behavior 4 fixes the third, and prints the next offset so the model never converts between the two bases. Behavior 5 prints a range line on a last page only when the offset is above 0, because Glob has no line numbers that show where a list ends. Adding `offset` departs from Cursor's two parameters, and an optional parameter does not break a model that never sends it.

Two caps and no cap both lose. Cursor's two caps produce two notices and a header that disagrees with the list (Q12). With no cap, a tree of 100,000 files returns 100,000 lines. A cap with no `offset` is the fallback if the schema must stay Cursor's, and Claude Code ships it. Confidence is medium, because paging adds a parameter and an obligation to keep a total order that nothing else in this report needs.

### Prefix every relative pattern with `**/`, because three harnesses agree and the anchored forms give false zeros

Cursor prepends `**/` to any pattern that does not start with it. Copilot does the same unless the query starts with `**/` or `/` or contains `:`. pi's own `find` reaches the same effect by another rule. A slash-less pattern matches basenames at any depth through `fd --glob`, and a pattern with a `/` gets `--full-path` and a `**/` prefix unless it starts with `/` or `**/` or is `**` `(source)`. So the three agree that `*.ts` and `src/**/*.spec.ts` both match at any depth. Cursor's `pi_find` port breaks pi's rule: it prefixes only a pattern that contains a `/`, so a bare `*.ts` matches only the root, and the most common call returns almost nothing ([tool-glob.md](tool-glob.md), the pi section). Rig and OpenCode anchor a pattern that has a slash, and Zed matches the pattern as given against `root_name/path` `(source)` `(probe)`. Table 5 lists the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **Prefix `**/` unless the pattern starts with `**/` or is `**`. A leading `/` is absolute. A trailing `/` becomes `/**` (recommended)** | The Pack | `*.rs` and `src/**/*.rs` find files at any depth. One sentence in the description says so. Matches Cursor, Copilot and pi's own behavior | `src/*.rs` also matches `lib/src/x.rs`: a superset that the model can see. Root-level-only needs an absolute pattern | Recommended |
| Anchor a pattern that contains `/` (rig, OpenCode) | The Pack | `src/*.rs` means the root's `src` | `src/**/*.rs` returns zero where `src` sits under `packages/app/`, and the model reads zero as absent | Set aside |
| Split off the pattern's literal base directory (Claude Code) | The Pack | Absolute patterns work | More code, and the recursion rule varies per call | Set aside |
| Do nothing: pass the pattern to `Access::glob` as given | Nobody | No Pack code | `*.rs` finds only the root level. A model trained on Cursor writes `*.js` and gets a partial list | Set aside |

*Table 5. Four ways to read a relative pattern. Sources: [tool-glob.md](tool-glob.md), Decision 6, Trap 4, Trap 15 and the Copilot, pi and rig sections; pi `find.ts` lines 199 to 212; Copilot `findFilesTool.tsx` lines 137 to 143.*

A superset beats a false zero. Under the prefix, `src/*.rs` also matches `lib/src/x.rs`, and the model sees both paths. Under an anchored rule, `src/**/*.rs` returns zero in a repository whose `src` sits under `packages/app/`, and the model concludes the files are absent. That is the Loud result criterion. A model that wants only the root level writes an absolute pattern, which behavior 7 passes through.

A trailing slash means "under". Copilot rewrites `src/` to `src/**`. Cursor strips the slash and searches for `**/src`. `Access::glob` itself reads a trailing `/` as directories only `(source)`. In Runvil a stripped `**/src` returns files only, so `src/` would become a silent zero, and rig's probe shows the same silent zero for `src` and `src/`. I take Copilot's reading, because models write `src/` to mean the files under it. That is a judgment of the evidence report (Decision 6) and no source measures it. The directory-only mode stays in the VFS for a future `ListDir`. Confidence is high for the prefix, because three harnesses agree and the alternatives give false zeros, and medium for the trailing slash.

### Match case-sensitively and explain the zero, because the matcher and the grammar already decide both

Cursor and Copilot match case-insensitively on purpose, and Copilot's test pins it `(source)`. Gemini does by default. Claude Code, Zed, rig, zeroclaw and OpenCode match case-sensitively, and rig does so even on NTFS `(source)` `(probe)`. Runvil's `matches_tokens` compares bytes, so it is case-sensitive on every backend `(source)`. The grammar has four tokens, so `{`, `[`, `?` and `!` are literal bytes. A model's `**/*.{ts,tsx}` then matches only a file literally named `x.{ts,tsx}`, and returns a silent zero. The evidence report points both ways on case: Trap 3 leans insensitive at medium-high and its Runvil section leans sensitive at medium. I settle it here. Table 6 lists the options.

| Option | Who does the work | Provides | Costs | Verdict |
|---|---|---|---|---|
| **Case-sensitive. Grammar unchanged for now. A zero reply says so when the pattern holds a letter, and names `{`, `[` and `?` when it holds one (recommended)** | The Pack words the reply | No VFS change. Literal `[id].tsx` paths still match. Each silent zero becomes an explained zero | A model that writes `*.JS` or `*.{ts,tsx}` spends one more call | Recommended |
| Case-insensitive through the Pack | The Pack, with a second matcher or a full-tree filter | Matches Cursor and Copilot | The grammar has no classes to rewrite into, and Grep needs the same matcher | Set aside |
| Case-insensitive through a VFS flag | The VFS | One matcher, shared with Grep | A VFS change with no plan item, and a locale question beyond ASCII | The later option if zero-reply transcripts justify it |
| Brace expansion in the Pack | The Pack, one `Access::glob` call per alternative | One call from the model | A walk, a claim and an event per alternative, and the code is deleted when the VFS gains alternation | Set aside |
| Alternation in the VFS grammar now | The VFS | One walk, one claim, one matcher shared with Grep | A grammar change with no plan item | The next step, for `{a,b}` only |
| Do nothing: leave the silent zero | Nobody | Nothing | The model reads zero as absent | Set aside |

*Table 6. Six ways to handle case and unsupported syntax. Sources: [tool-glob.md](tool-glob.md), Decision 1 (the grammar sub-decision), Trap 3 and the Copilot, Gemini, Zed, rig and OpenCode sections; Runvil `glob.rs`.*

Sensitive wins because the Pack cannot fold case. The grammar has no classes to rewrite `a` into `[aA]`, and a Pack-side matcher would walk and filter the whole tree itself. A VFS flag would serve Glob and Grep with one matcher and is the later option if transcripts show wrong-case zeros. Until then the zero reply says the match is case-sensitive, which turns a silent zero into one more call. Of the eight harnesses with a file-finding tool, five match case-sensitively (Claude Code, Zed, rig, zeroclaw, OpenCode), two case-insensitively by default (Copilot, Gemini), and pi's `fd` uses smart case.

Brace alternation is the one syntax worth adding. Copilot's own description teaches `**/*.{js,ts}` `(source)`, so a model that has seen that form will send it, and nothing measures how often real calls use braces ([tool-glob.md](tool-glob.md), Decision 1). Expanding braces in the Pack costs an `Access::glob` call per alternative. Alternation in the VFS grammar is the better home: one walk, one claim, one matcher that Grep shares. Character classes and `?` should wait, because the grammar has no escape and a class would break literal `[id].tsx` paths in file-based routers. Until alternation lands, a zero reply that holds `{`, `[` or `?` says those characters match themselves. Confidence is medium, because no source measures how often models send either form.

## Behaviors to implement

Fourteen behaviors, ordered by how much a wrong choice costs the model. Each opens with the recommendation, then the reasoning, the evidence, the concrete failure without it, a test, and a confidence word. Examples use the constant `LIST_CAP_BYTES = 15000`, a crate constant that Grep's output cap shares, and the notice texts below, which a test imports by name.

### Sort every result by full-path byte order, with no clock and no budget

**Recommendation.** Return paths in ascending byte order of the full path string. Re-sort in the Pack, although both backends already sort, and say "sorted by path" in the description. Read no modification time, no clock and no locale.

**Why.** Replay needs one order on every backend and host, and paging needs a total order over unique keys, which paths are. The Pack's re-sort costs little on sorted input and protects against a custom backend that sorts differently, since the trait promises only "sorted". Stating the order in the description turns it from an accident into a promise (Loud result). No stat call means no per-match gate, claim or event (Bounded cost). Grep lists files in the same order (`master-grep.md`, Behavior 4, the ordering behavior), so a model that runs both tools sees one sequence of names.

**Evidence.** Cursor's sort stats every match under a 2000 ms budget and, past it, returns walk order with no marker `(daemon)`. Claude Code's `--sort=modified` runs oldest first, so under its cap of 100 it keeps the oldest 100 (three files dated three, two and one day back came out oldest, middle, newest) `(probe)`. Gemini's 24-hour tier moves with the wall clock `(source)`. OpenCode's documentation promises a modification-time sort that its code lacks, and three runs over 400 files gave three orders `(probe)`. zeroclaw cuts at 1000 before it sorts `(probe)`. Zed and Goose's `tree` give byte-stable order that replays `(source)`. Runvil's `memory.rs` and `real.rs` both end `glob` with `sort_unstable()` over the path strings `(source)`. In byte order `-` and `.` sort below `/`, so `/a-b/x`, `/a.rs` and `/a/x` come in that sequence, as zeroclaw's probe shows for `a-b/y.txt`, `a.txt` and `a/z.txt`.

**Without it.** A model that repeats a call gets a different first page and cannot page. A run recorded on a real mount replays with a different list on a memory mount. A cap keeps the wrong files.

**Test.** Seed `/b.txt`, `/B.txt`, `/a.rs`, `/a-b/x.rs` and `/a/x.rs`. `Glob *` returns `/B.txt`, `/a-b/x.rs`, `/a.rs`, `/a/x.rs`, `/b.txt`, byte for byte, twice, on the memory backend and on the real backend. Add a test backend whose `glob` returns the reverse order, and assert the reply is still sorted. Set modification times in reverse order on the real backend and assert they change nothing.

**Confidence.** High, because Zed, Goose and Runvil's two backends already produce one kind of total order and paging needs one.

### Put ignore and privacy rules in `Access`, apply them after matching, and count what they hide

**Recommendation.** Build the filter in `Access` in the end: a per-match check beside `Policy::check`, with one precedence. A privacy rule, once one exists, wins, then an ignore rule, then the pattern, so no pattern resurrects a hidden path, and no rule needs a git repository. Until it exists, apply one Pack module that Grep shares: `glob-real` writes it and `grep-real` calls it. After the match it leaves out any path that has a segment named `.git`, file or directory, and any path that a `.gitignore` on the way to it names, honoring `!` rules and needing no `.git` directory. End the reply with `N paths were left out by .git or .gitignore rules.` (`1 path was left out by .git or .gitignore rules.` for one). A `target_directory` that is itself ignored is searched, because the model named it. A pattern never brings an ignored path back. Other hidden files stay visible. Whether the module uses the `ignore` crate or a small matcher of its own is an open item for Vinnie, and the open limits say what each costs. Once `Access` owns the rule, the module is deleted and the line reads `N paths were left out by ignore or privacy rules.` (`1 path was left out by ignore or privacy rules.` for one).

**Why.** Loud result: a model that knows a file exists and gets zero concludes the file is gone. Bounded cost: a generated tree such as `target/` or `node_modules/` fills Glob's pages and spends Grep's work budget (`master-grep.md`, Behavior 10, the ignore behavior). File safety: a listing must not claim a rule that Read does not apply, so the policy is stated once. Ignore rules and `.git` hide entries from Glob and Grep listings only, and Read, Write, StrReplace and Delete act on any path the call names. VFS fit: Glob and Grep must give one answer, so the rule belongs in the layer both call, and until `Access` owns it, in one module both call. Replay: the ignore inputs are part of the tree, and the reply says when a rule changed the list.

**Evidence.** In Cursor a positive glob beats a file-level `.gitignore` rule (`*.log` returns the ignored `debug.log`), `**/*` returns an ignored directory's whole subtree, and `.gitignore` does nothing outside a git repository `(probe)`. A `.cursorignore` rule beats the glob `(probe)`. Rig's probe gives the same split: `*`, `**`, `**/*` and `*.txt` bring back ignored paths, while `build/**` returns nothing from an ignored directory, and the `ignore` crate documents "Overrides have the highest precedence" `(probe)` `(source)`. Zed's `find_path` skips the privacy filters its sibling tools apply and force-includes `.env*` by default (I read this in the source and did not run it). Goose passes `require_git(false)` and pi adds `--no-require-git` outside a repository `(source)`. Gemini counts the files its ignore filter removed `(source)`. Runvil's `Policy::check` sees only the pattern for `Op::Glob`, and the plan records that the VFS has no ignore concept `(source)`. `ModePolicy` refuses by operation and by a `.md` suffix only, so no VFS rule refuses a path by name and Read refuses no `.env` `(source)`.

**Without it.** `target/` and `node_modules/` fill every page of `**/*.js`. With an override layer instead, `*.log` returns the file the user listed in `.gitignore`. Without the stated policy, a model that expects Glob to hide `.env` finds it listed, and then finds that Read opens it.

**Test.** Interim, on the real backend: with `.git/config`, a `.gitignore` holding `target/`, `src/a.rs` and `target/x.rs`, `Glob *` lists `/.gitignore` and `/src/a.rs` and ends with `2 paths were left out by .git or .gitignore rules.`. `Glob *` with `target_directory: "/target"` lists `/target/x.rs`. With no `.git` directory, a `.gitignore` holding `*.log` and `!keep.log` and the files `/a.log` and `/keep.log`: `Glob *.log` lists `/keep.log` and ends with `1 path was left out by .git or .gitignore rules.`. `Glob .git/config` answers a zero that includes the hidden count, so a pattern that names the path does not bring it back. Once `Access` has the rule: delete the module and repeat the cases, with the line `N paths were left out by ignore or privacy rules.` and the same lists, with and without a `.git` directory present.

**Confidence.** Medium, because two probes and a source read support the destination, no VFS design exists, the choice between the `ignore` crate and a small matcher is open, and no source shows models misled by ignore silence in particular.

### Never answer a bare zero: say what matched nothing and the likely reasons

**Recommendation.** When nothing matches, answer `No files match "<pattern>" in <dir>.`, with `<dir>` the search directory as the model wrote it (omitted for an absolute pattern). Then add only the hints that apply, one per line, in this order. The hidden count, from the shared ignore module or from `Access`. The absolute hint, for a pattern that starts with `/`, saying that it starts at the root of the run's filesystem. The case hint, `Matching is case-sensitive.`, when the pattern holds an ASCII letter. The brace hint from behavior 9. A zero is a success, not an error.

**Why.** Loud result. Every zero in the evidence set that hides its cause misleads the same way: the model concludes the code is absent. The hints add a line or two to a reply that is otherwise one line, so Bounded cost is untouched, and each hint saves the model a call it would spend guessing.

**Evidence.** Cursor answers `0 files found` for a whitespace-only pattern and for an absolute pattern, with no cause `(probe)`. `pi_find` answers `No files found matching pattern` for a bad glob and for a walk budget spent before a match `(probe)`. Rig's `No matches.` follows an unreadable file, a symlink, a walk error, an inert ignore rule and a directory-name glob `(probe)`. Claude Code, Copilot and OpenCode answer a bare `No files found` `(source)`. Of the eight harnesses with a file-finding tool, only Gemini says how many files its ignore filter removed from an empty result `(source)`.

**Without it.** A model sends `**/*.{ts,tsx}`, reads `No files match`, and reports that the project has no TypeScript.

**Test.** Table-driven, with exact text. `*.JS` over a tree holding `a.js` includes the case hint. `**/*.{ts,tsx}` includes the brace hint. `/home/dev/*.rs` includes the absolute hint. `.git/config` includes the hidden count. `**/*` over an empty directory is only the first line, because the pattern holds no letter and no `{`, `[` or `?`. No zero reply is an error result.

**Confidence.** High, because Cursor, `pi_find`, rig and OpenCode each return a bare zero over a cause the model cannot see, the evidence names each cause, and the hints cost a line.

### Page with `offset` under one byte budget, and print the exact total

**Recommendation.** Add an optional integer `offset`, default 0, the number of paths to skip. Fill a page with paths in order until the next one would take the path lines past `LIST_CAP_BYTES` (15000 UTF-8 bytes, each path counted with its newline), and always include at least one path. The cap counts the path lines. The header and the notice are not counted. Print the exact total in the first line, which `Access::glob` allows because it returns the whole list. A cut page ends with a notice and looks like this:

```text
Found 1432 files matching "*.rs" in /:
/a.rs
/b.rs
...
[Showing files 1-483 of 1432. Use offset=483 to see more, or narrow the pattern.]
```

The header names the search directory as the model wrote it, `/` when none was given, and leaves it out for an absolute pattern. The last page of a paged result, reached with an offset above 0, ends `[Showing files 967-1432 of 1432.]`. An offset past the end answers `No more results at offset=2000. 1432 files match "*.rs" in /.`, as a success. One file reads `Found 1 file matching`, and a count is plain digits with no separator. The singular and plural forms come from the shared `plural(n, noun)` helper in `plugin-std`.

**Why.** Bounded cost: a path list costs its bytes, and a byte budget bounds that cost whatever the path length. Few calls: the exact total tells the model whether to page or to narrow, and the next offset is printed, so the model does no arithmetic. Glob's `offset` counts paths to skip, as Grep's does, and Read's `offset` is a line number. Each parameter's text names its unit, and each notice prints the number to send, so the model copies it and never converts. Loud result: the reply says how much exists. The order from behavior 1 keeps pages stable, since a page is a window into a fixed list. The notice is Grep's grammar (`master-grep.md`, Behavior 7, the paging behavior), so a model reads the two tools' notices as one family.

**Evidence.** Cursor's three caps (2000 entries, 10000 stdout lines, 5000 UTF-16 code units) bind at about 160 files, and its header can read `(total 5000 files)` over about 160 lines `(daemon)`. Nothing in Cursor pages. Claude Code caps at 100 and sends `offset` as 0 `(source)`. Zed pages 50 at a time. Its range line also prints on the last page, it mixes a 1-based range with a 0-based `offset`, and it answers `No matches found` for an offset past the end `(source)`. pi's `Use limit=2N for more` cannot raise a byte cap: with 1500 files of 60-character names, 853 of 1000 survived the 51,200-byte cap and the notice advised `limit=2000` `(probe)`. Runvil's `Access::glob` returns the whole list, so the total is exact and no "at least" form is needed `(source)`.

**Without it.** A search with 400 matches shows about 160 and offers no route to the rest except a narrower pattern. An entry cap lets 200 paths of 150 bytes cost five times as much as 200 paths of 30 bytes.

**Test.** With `LIST_CAP_BYTES = 15000` and 1,000 paths of 30 bytes (31 with the newline), page 1 holds 483 paths and ends `[Showing files 1-483 of 1000. Use offset=483 to see more, or narrow the pattern.]`. `offset: 483` holds 483 paths and ends `[Showing files 484-966 of 1000. Use offset=966 to see more, or narrow the pattern.]`. `offset: 966` holds 34 paths and ends `[Showing files 967-1000 of 1000.]`. Joining the pages gives the sorted list with no repeat and no gap. `offset: 2000` answers `No more results at offset=2000. 1000 files match "*" in /.`. A single match reads `Found 1 file matching`. A test backend that returns one 16,000-byte path and two short ones gives a first page of that path alone, with next offset 1. Twenty paths made of one 300-character CJK name each (902 bytes with the slash and the newline) fill a page with 16 paths, where a count of characters would admit all twenty (16 times 902 is 14,432 and 17 times is 15,334).

**Confidence.** Medium, because paging adds a parameter beyond Cursor's schema and nothing measures the page size a model finds adequate.

### Print a notice only when paths remain: an exact fit is silent

**Recommendation.** Decide that a result was cut from the list and never from the cap: there is a cut if and only if at least one matching path lies beyond the page. Print no notice when offset is 0 and every path was shown, whether or not the page is full to the byte. Print `[Showing files {first}-{last} of {total}.]` as the last line only when the offset is above 0 and no path remains.

**Why.** Loud result. A notice at an exact fit claims a cut that did not happen, so the model spends a call to follow it, and it learns to distrust the next notice. The Pack holds the whole list, so it knows what remains with no extra walk: it is the total minus the offset and the page.

**Evidence.** Six sources fire the notice when the count reaches the cap and not when it exceeds it. OpenCode sets `truncated = files.length === limit` at 100. zeroclaw's flag fires at 1000: 999 files kept no flag, and 1000 and 1001 both kept 1000 with it `(probe)`. Rig's `MAX_FILES` check runs after the push, so exactly 1000 names get the stop notice `(probe)`. pi tests `relativized.length >= limit`. Cline's legacy listing fires at 200. Cursor's `ripgrepTruncated` fires at 10000 lines `(source)` `(daemon)`. OpenCode's `dev` branch asks the walker for `limit + 1` and slices, which makes the flag exact `(source)`.

**Without it.** A tree that holds 483 paths, as many as one page holds, prints a notice that invites `offset=483`, and that call answers with a reply that begins `No more results at offset=483.`

**Test.** Seed as many paths as one page holds (483 at 31 bytes each): the reply has no notice. Add one path: the notice appears and reads `[Showing files 1-483 of 484. Use offset=483 to see more, or narrow the pattern.]`. At `offset: 483` the reply ends `[Showing files 484-484 of 484.]`. Repeat at totals of 0, 1 and 482.

**Confidence.** High, because six harnesses show the fault and the fix is one comparison.

### Make every pattern recursive from the search directory

**Recommendation.** Trim the pattern. If it does not start with `**/` and is not `**`, prepend `**/`. Join the result onto the search directory and call `Access::glob`. The search directory is `/` when `target_directory` is absent. A `target_directory` without a leading `/` is read from the root, so `src` means `/src`, and its trailing slashes are dropped, so the join never doubles a slash. Read `target_directory` as a path: turn each backslash into `/` before the join, as `canonicalize` does for every other path. Only `glob_pattern` refuses a backslash (behavior 9). `*.rs` under `/src` becomes `/src/**/*.rs`. A pattern that starts with `/` is behavior 7, and a trailing `/` is behavior 11.

**Why.** Loud result: a rule that anchors `src/**/*.rs` to the root returns a false zero where `src` sits under `packages/app/`. Few calls: one sentence in the description tells the model it never has to write `**/`. VFS fit: `Access::glob` joins a relative pattern onto the root with no recursion, so the Pack has to add it, and the `**/` token matches zero segments, so root-level files still match.

**Evidence.** Cursor's normalizer prepends `**/` unless present `(daemon)`. Copilot prepends it unless the query starts with `**/` or `/` or holds `:` (`findFilesTool.tsx` lines 137 to 143) `(source)`. pi's `find` gives a slash-less pattern basename matching at any depth and gives a pattern with `/` a `--full-path` match with a `**/` prefix unless it starts with `/` or `**/` or is `**` (`find.ts` lines 199 to 212) `(source)`. Cursor's `pi_find` port prefixes only a pattern with a `/`, so a bare `*.ts` matches the root only, and `*.TS` and a bad glob answer `No files found matching pattern` `(probe)`. Rig's `src`, `src/` and `test*` match nothing `(probe)`. Runvil's matcher test shows `src/**/*.rs` matches `src/a.rs` `(source)`.

**Without it.** `*.js` returns the root-level files only, and the model reads a partial list as complete.

**Test.** Seed `/a.rs`, `/src/b.rs`, `/lib/src/c.rs` and `/src/deep/d.rs`. `*.rs` returns all four. `src/*.rs` returns `/lib/src/c.rs` and `/src/b.rs`. `**/*.rs` returns the same as `*.rs`. With `target_directory: "/src"`, `"src"` or `"/src/"`, `*.rs` returns `/src/b.rs` and `/src/deep/d.rs`, as absolute paths each time. With `target_directory: "src\deep"`, `*.rs` returns `/src/deep/d.rs`, the same as `"/src/deep"`. A logging backend shows the pattern that reached it was `/**/*.rs`, never `//**/*.rs`.

**Confidence.** High, because Cursor, Copilot and pi's own `find` agree and the anchored forms give false zeros.

### Pass a leading `/` through as a full VFS path, and refuse it beside `target_directory`

**Recommendation.** A pattern that starts with `/` after the trim is a full path pattern from the root of the run's filesystem. Send it to `Access::glob` as given, with no prefix. When `target_directory` is also set, refuse the call: ``std_Glob: `glob_pattern` starts with "/", so it already names a full path. Remove `target_directory`, or drop the leading "/" to search under it.`` A pattern made only of slashes counts as empty (behavior 10).

**Why.** In Runvil a model's paths are VFS paths: `Read` takes `/notes.md`, and `Access::glob` defines a leading `/` as absolute. So `/src/*.rs` is the natural spelling and means what it says. Cursor's rewrite turns an absolute pattern into a dead suffix match with no error, and that is the failure to avoid (Loud result). The refusal beside `target_directory` stops two directories from competing for one pattern.

**Evidence.** Cursor strips one leading slash and prepends `**/`, so `/home/dev/proj/src/*.ts` becomes `**/home/dev/proj/src/*.ts` and `C:\work\proj\*.ts` becomes `**/C:/work/proj/*.ts`, and both return zero `(probe)`. Claude Code splits an absolute pattern into a search directory and a relative pattern, Copilot converts it into a pattern scoped to the matching folder, and pi's `fd` matches it against the absolute path `(source)`. Zed never matches one, because the glob is tested against `root_name/relative`, and zeroclaw admits one and then drops every hit `(source)`. Three harnesses handle an absolute pattern and three break it without a message ([tool-glob.md](tool-glob.md), Comparison table). Runvil's `Access::glob` documents "A pattern that starts with `/` is absolute" `(source)`.

**Without it.** A model that pastes a path pattern gets a silent zero, or `/src/*.rs` also matches `/lib/src/x.rs` because a rewrite turned it into `**/src/*.rs`.

**Test.** Seed `/src/a.rs` and `/lib/src/b.rs`. `/src/*.rs` returns `/src/a.rs` only. `src/*.rs` returns both. `/src/*.rs` with `target_directory: "/lib"` answers the refusal text above. `/home/dev/*.rs` answers a zero with the absolute hint. `/` and `//` answer the empty-pattern refusal.

**Confidence.** High, because `Access::glob` already defines the form and the three harnesses that handle absolute patterns agree.

### Match case-sensitively and say so

**Recommendation.** Add no case folding. State in the description that matching is case-sensitive, and put `Matching is case-sensitive.` in a zero reply whose pattern holds an ASCII letter.

**Why.** VFS fit: the Pack cannot fold case, because the grammar has no classes to rewrite into and a second matcher would have to walk the whole tree. Loud result: the hint turns the one silent zero that case sensitivity can cause into an explained one. Replay: a byte match gives the same list on every backend and host.

**Evidence.** Cursor matches the pattern with `--iglob`, so `*.JS` returns `a.js` on both platforms `(probe)`. Copilot passes `{ caseInsensitive: true }` and pins it in `findFiles.spec.tsx` `(source)`. Gemini defaults to `nocase` `(source)`. Claude Code, Zed, rig, zeroclaw and OpenCode match case-sensitively, and rig does so on NTFS (`*.RS` finds nothing) `(source)` `(probe)`. pi's `fd` uses smart case, and its `pi_find` port is case-sensitive `(probe)`. Runvil's `matches_tokens` compares bytes, so `*.js` does not match `B.JS` on any backend `(source)`.

**Without it.** A model that writes `readme*` for `README.md` reads a bare zero as absence.

**Test.** Seed `/README.md` and `/a.js`. `readme*` answers a zero with the case hint. `README*` returns the file. `*.JS` answers a zero with the case hint. On a Windows temporary directory, `readme*` still returns zero although the operating system would open `readme.md`, which pins the VFS choice on a case-insensitive file system.

**Confidence.** Medium, because five of eight harnesses agree but Cursor and Copilot, the surfaces models know best, choose the other way, and no transcript data exists.

### Accept the grammar `Access::glob` implements, and explain a zero that holds braces, brackets or question marks

**Recommendation.** Keep the four tokens. Let `Access::glob` refuse a backslash, a run of three or more `*`, a `**` that shares a segment, a control character and a pattern over 1,024 bytes, and render each refusal with the rule and a fix. Treat `{`, `[`, `?` and `!` as literal bytes. When a zero reply's pattern holds `{`, `[` or `?`, add the brace hint `{, [ and ? match themselves in this glob; there is no alternation or character class. Call Glob once per alternative.` Alternation `{a,b}` is the next VFS step, and Glob and Grep's `glob` filter gain it together.

**Why.** Loud result: without the hint, `**/*.{ts,tsx}` returns a silent zero. VFS fit: the grammar is the VFS's, and the Pack does not grow a second grammar. Grep's `glob` filter uses this grammar and these rules (`*` and `**` only, case-sensitive, a relative pattern searched at any depth, a leading `/` a full path), so one parameter name has one meaning (`master-grep.md`, Behavior 10, the ignore, links, `glob` and `type` behavior). The matcher is one pure function exposed by the VFS crate in the first commit of `glob-real`, which runs before `grep-real`. If the VFS refuses to expose it, `plugin-std` holds a copy of the four-token grammar with one test file that runs the same cases against both. `globset` is not added. Few calls: an error that names the rule costs one retry, and a silent zero costs several turns or a wrong conclusion.

**Evidence.** The grammar has `Literal`, `Star`, `DoubleStar` and `DoubleStarSlash`, a 1,024-byte limit and no escape, and its tests pin `a/***/b` and `a/**b` as `PathReason::Wildcard` `(source)`. Cursor accepts `**b` and reports a bad class as `rg: error parsing glob '**/[a-': unclosed character class; missing ']'` `(probe)`. Zed returns `Invalid glob: <err>` and rig `bad glob: ...`, while `pi_find` and OpenCode answer a bad glob with a silent zero `(source)` `(probe)`. Copilot's description teaches `**/*.{js,ts}` `(source)`. How often real calls send braces is not measured ([tool-glob.md](tool-glob.md), Decision 1).

**Without it.** A model sends `**/*.{ts,tsx}` and concludes the project has no TypeScript.

**Test.** `**/*.{ts,tsx}` over a tree holding `/a.ts` answers a zero that holds the brace hint. Over a tree holding a file literally named `/x.{ts,tsx}` it returns that file. `a/**b` answers a refusal that names `**` and says it must fill a whole segment. `a/***/b` answers a refusal. `a\b` answers a refusal that names the backslash and says to use `/`. A 1,100-byte pattern answers a refusal that names the 1,024-byte limit. The same table of cases runs against the matcher that Grep's `glob` filter calls, and the two must agree.

**Confidence.** Medium, because no source measures how often models send braces or classes.

### Refuse an empty pattern and check the target directory before the walk

**Recommendation.** Trim `glob_pattern`. A pattern that is empty after the trim, or made only of slashes, is refused with ``std_Glob: `glob_pattern` is empty. Pass "*" to match every file.``. Treat an empty or whitespace-only `target_directory` as absent. For any other `target_directory`, call `Access::stat` first. An absent path answers `Path does not exist: {path}` and a file answers `Path is a file, not a directory: {path}`. The path in every reply and message is the argument as the model wrote it. Result lines are not an echo: they are the absolute paths of behavior 12. Refuse any parameter other than `glob_pattern`, `target_directory` and `offset`, naming it, with the text the shared argument helper builds, here for a stray `path`: ``std_Glob: `path` is not a parameter of this tool. The parameters are `glob_pattern`, `target_directory` and `offset`.`` Do the checks before `Access::glob`.

**Why.** Loud result. `Access::glob` answers a missing search directory with an empty list, so a model that mistypes `/scr` reads zero as "no files", and a stray space in the pattern would do the same. The extra `stat` is one claim and one event, which is cheap next to a walk. An undeclared optional parameter is the same silent case: a model that sends `path`, the name Grep uses, in place of `target_directory` would search the root and read the reply as if it had searched where it said.

**Evidence.** The real backend's `glob` returns `Vec::new()` when the walk root is not a directory, and the memory backend matches nothing `(source)`. The shared argument check visits only the keys in the schema's `properties`, so a key the schema does not declare passes it, and only a required key is protected by the required-key refusal (`check.rs`) `(source)`. Cursor rejects only the literal empty case, so `" "` becomes `**/ ` and renders `0 files found` `(probe)`. Cursor pre-checks the target and answers `Path does not exist: <absolute path>`, and it accepts a file as the target `(probe)`. Gemini rejects an empty pattern and answers `Search path does not exist` and `Search path is not a directory`, and Claude Code throws `Directory does not exist` with a "Did you mean" `(source)`. Zed turns an empty glob into `*`, and OpenCode lists up to 100 arbitrary files for an empty one `(source)` `(probe)`.

**Without it.** `Glob {glob_pattern: "*.rs", target_directory: "/scr"}` answers a clean zero, and `Glob {glob_pattern: "*.rs", path: "/src"}` lists the whole tree.

**Test.** `""`, `"   "`, `"/"` and `"//"` each answer the empty-pattern refusal as an error result. `target_directory: "/missing"` answers `Path does not exist: /missing` and `target_directory: "/a.rs"` answers `Path is a file, not a directory: /a.rs`. `target_directory: "missing"` answers `Path does not exist: missing`, the argument as written. `target_directory: ""` and `"  "` behave as absent. `{"glob_pattern": "*.rs", "path": "/src"}` answers the unknown-parameter refusal above, which names `path`. No message names another tool. A logging backend shows one `Stat` before the `Glob` when a directory is given, and none when it is absent.

**Confidence.** High for the empty-pattern and target checks, because the empty-list answer is in the backend source and every harness that refuses these inputs does it for the same reason. Medium for the undeclared-parameter refusal, because the evidence shows the silent case only for Glob and the helper is small.

### List files only, leave links unlisted, and read `X/` as the files under X

**Recommendation.** Return files only, which `Access::glob` does when the pattern has no trailing slash. Rewrite a trailing `/` after the prefix rule into `/**`, so `src/` becomes `**/src/**`. Never call `Access::glob` in directories-only mode in version one. State in the description that a walk does not enter linked directories and lists no links, that a `target_directory` or a literal pattern prefix that names an in-root directory link is walked through it, and that a trailing `/` means the files under that directory.

**Why.** Loud result: `src/` rewritten the way Cursor does becomes `**/src`, which in Runvil returns files named `src` and almost always zero. File safety: a walk that does not follow links stays inside the root and cannot loop. Reading `X/` as "under" is the reading Copilot chose, and the directory-only mode stays in the VFS for a future `ListDir`, which the plan defers.

**Evidence.** Copilot turns a trailing `/` into `**` (`findFilesTool.tsx` lines 141 to 143) `(source)`. Cursor strips the slash, and rig's probe shows `src` and `src/` return nothing `(probe)`. In Runvil, `walk` collects a linked directory without following it, and the default `glob_kind` keeps only entries whose `stat` is `File` (or `Directory`). `Access::stat` of a link returns `FileType::Symlink` and neither built-in backend overrides `glob_kind`, so a link leaf appears in neither mode (I read this in the source and did not run it) `(source)`. Cursor follows every link with `--follow` `(daemon)`. Rig never lists links, `fd` does not follow them, and Gemini passes `follow: false` `(source)`. That a model means "under" by `src/` is a judgment of the evidence report, Decision 6, and no source measures it.

**Without it.** A model looks for `latest.md`, a link to `v3.md`, never finds it, and concludes it is missing. Or `src/` returns zero.

**Test.** On a Unix real backend, make `/target.txt` and a link `/link` to it. `*` returns `/target.txt` and not `/link`, which pins the behavior so a later change is visible. Make a directory link `/dirlink` to `/real` (a junction on Windows, as the VFS link tests do): `*.txt` lists no path under `/dirlink/`. `target_directory: "/dirlink"` with `*.txt` lists `/dirlink/a.txt`, which the VFS test `a_glob_through_an_in_root_directory_link_keeps_the_links_spelling` pins for the same walk. Seed `/src/a.rs`, `/lib/src/b.rs` and a directory `/x.rs`: `src/` returns both files and no directory, and `*.rs` does not list `/x.rs`.

**Confidence.** Medium, because the link behavior is read from source and not run, and no source measures how often models look for links.

### Print absolute VFS paths, bare, one per line, and leave out any name with a control character

**Recommendation.** Print each path as the VFS returns it for an absolute pattern: a leading `/`, `/` separators, whatever the host and whatever `target_directory` says. One path per line, no bullet, with `\n` line ends. Leave out any path that holds a character below 0x20 or 0x7f, and count it in the last line: `1 path with a control character in its name was left out.` (`N paths with a control character in their names were left out.` for more than one). Return any reply that lists a path as untrusted output, because the model did not write the names. A zero reply that holds only the model's own pattern and the hints may be trusted.

**Why.** Few calls: a path copied from Glob into Read must work, and Read's `path` is an absolute path on an access rooted at `/`. The description says so, because a model that does not know the printed form is reusable may rewrite it. Replay and Loud result: a separator or prefix that varies by host, or by whether `target_directory` was set, makes one tree print two ways. A newline in a name would split a line into two false paths, so the problem a `Vec` avoids returns when the Pack renders text.

**Evidence.** Cursor prints paths relative to the target, so one file reads `src/a.ts` with no target and `a.ts` with `target_directory: "src"`, and on Windows the `.\` prefix survives because the strip test is `startsWith("./")` `(probe)`. Cursor's bullets add 2 bytes to every line, about 6% of a 31-byte line `(bundle)`. Claude Code, Copilot, pi and rig print bare paths `(source)`. Zed prints absolute host paths, and the evidence report rejects that because every later call carries a long prefix, an objection that does not apply to a VFS path, which is short `(source)`. A POSIX name with a newline splits an entry in Cursor, Zed and pi, by reading their parsers and not by running them `(source)`. Runvil's `walk` builds `/`-joined virtual paths, and `utf8_name` checks only UTF-8 `(source)`. Rust writes `\n` on every platform, so the test pins `\n`. A path name comes from the tree and an attacker can choose it, and the standing plan says `Glob` marks its output untrusted: an untrusted `ToolOutput` is nonce-wrapped before it reaches the model (`types/src/tools/output.rs`) `(source)`.

**Without it.** A model on Windows sees `.\src\a.ts`. A model that searched under `/src` feeds `a.ts` to Read and opens `/a.ts`.

**Test.** `*.rs` with `target_directory: "/src"` returns lines that all begin `/src/`, and every line passed to `Read` succeeds. On Windows no reply byte is `\`, and no reply holds `\r`. On Unix, a file named `a` plus a newline plus `b.rs` is absent and the last line is the control-character count, and two such files give the plural line. A reply that lists a path is `OutputTrust::Untrusted`.

**Confidence.** High, because Cursor's own relative form fails the copy test and absolute VFS paths are short and the same on every platform.

### Render every `VfsError` in one function that names the rule, the path and the next step

**Recommendation.** The shared `refusal` function in `plugin-std` (proposed todo `refusal-text`) turns each `VfsError` into the reply text, and Glob calls it with its own name and the argument as the model wrote it. `InvalidPath` names the reason and the fix. `PermissionDenied` and `ApprovalRequired` carry the policy's own reason, which holds the next step once the policy text says it (proposed todo `vfs-mode-text`, a VFS edit). `Conflict` says ``A path that "{pattern}" matches is held by another call in this round, so this search cannot run in this round. Call Glob again in your next message.`` with `{pattern}` the `glob_pattern` as written. `NotUtf8` and `Io` name the directory the walk could not read and say that no list was produced. The last arm uses the error's `Display`, so a variant added later still reads as text. No message names another tool. Every error is an error result. A zero is a success (behavior 3).

**Why.** Loud result: a raw `conflicting claim: ...` or `not UTF-8: /d, valid up to byte 3` gives the model no next step. VFS fit: policy wording belongs to the policy, and the Pack's job is to pass it through, as the plan says for `approval`. One shared function gives one sentence per variant in all six tools. Until the VFS can return a partial list with a count of skipped entries, a walk that cannot continue should fail loudly and name where, and should not return a short list that reads as complete. Once `Access::glob_with_skips` exists, a bad name is a counted skip, as in Grep.

**Evidence.** `VfsError` has the variants `InvalidPath`, `PermissionDenied`, `ApprovalRequired`, `Conflict`, `NotUtf8`, `Io` and `Backend`, and `Verdict::Deny`'s text "goes back to the model as the tool error" `(source)`. A glob claim conflicts with a sibling's write in both orders, which settles the question the evidence report leaves open in Q9 `(source)`. Real-backend `walk` fails the whole call on one unreadable directory or one non-UTF-8 name `(source)`. Cursor whitelists unreadable subdirectories and symlink loops and returns the rest with no notice, which the evidence report calls right for a search and wrong for a report `(daemon)`. `pi_find` fails the whole call on one unreadable subdirectory, because its walk has no per-directory `try` `(daemon)`.

**Without it.** The model retries a conflicted call with the same arguments, or abandons a whole tree because one file name was odd, with no hint of which.

**Test.** A table over `InvalidPath` for a control character, a backslash, a long pattern and `***`, then `PermissionDenied` under a deny policy, `ApprovalRequired` under a test policy that answers `Verdict::Ask` for `Op::Glob` (because `ModePolicy` allows a glob in every mode), `Conflict`, `NotUtf8` and `Io`. Each text names the rule or the path and a next step, none is a `Debug` form, and none names another tool. For `Conflict`, spawn a sibling scope that writes `/research/5.md`, then run `Glob /research/*` from another scope and assert ``A path that "/research/*" matches is held by another call in this round, so this search cannot run in this round. Call Glob again in your next message.`` byte for byte. For `NotUtf8`, use a Unix real backend with a name holding the byte 0xFF.

**Confidence.** Medium, because the variants are read from source, and the wording for `Conflict` and `NotUtf8` is mine and comes from no harness.

### Write the description from the behavior

**Recommendation.** Replace the description and parameter text in `glob.json`. Say that results are sorted by path and that patterns are searched at any depth. Say that a leading `/` is a full path, that a trailing `/` means the files under it, and that matching is case-sensitive. Say that only `*` and `**` are special, that a walk does not enter linked directories and lists no links, that `.git` and the paths a `.gitignore` names are left out while other hidden files are listed, that Read opens any path the call names, and that a long result is cut at 15000 bytes and continues with `offset`. Say that the paths in a reply are absolute and work unchanged as the path argument of the other file tools. Add `offset`. Drop "prefer it over listing directories one at a time", because no listing tool exists. Restore Cursor's three examples, adjusted to this grammar. Correct `EDITS.md`. The proposed text is in Mapping to Runvil.

**Why.** Loud result starts before the call, and the description is the one place the model learns the rules. The shipped description says nothing about depth, order, case, links or paging, and the parameter text drops the examples that teach a model to write `**/` patterns. Every rule above is one the model cannot discover from a result.

**Evidence.** Cursor's description says "sorted by modification time" and its `glob_pattern` text carries the prepend sentence and the examples `"*.js"`, `"**/node_modules/**"` and `"**/test/**/test_*.ts"` `(bundle)`. The shipped `glob.json` has none of those, and its `target_directory` text differs from both Cursor variants (`REe` and `MEe`), yet `EDITS.md` lists one edit for it `(source)`. The spec tests refuse a description that names the Shell tool, PDFs or the Cursor workspace root, so a promise this build cannot keep already fails a test `(source)`. The plan's decision to copy Cursor's surface applies to names and parameters, and its own edit rule allows changing text that describes behavior the Pack does not have `(source)`.

**Without it.** A model reads "prefer it over listing directories" and looks for a list tool, sends `*.{ts,tsx}`, or expects newest-first order.

**Test.** A spec test over `glob.json`: the description contains `sorted by path`, `case-sensitive`, `offset`, `15000` and `.gitignore`, and contains neither `modification time`, `listing directories` nor `8,000`. `glob_pattern`'s text contains `**/`. The `properties` are `glob_pattern`, `target_directory` and `offset`, `required` is `["glob_pattern"]`, and `offset` is an integer. A second test runs each example in the description through the Pack and checks the paths it promises.

**Confidence.** High, because each gap in the current text is observed and the fix costs wording.

## Where Runvil should differ from Cursor

Table 7 lists where the Runvil tool should depart from Cursor's `Glob`. Rows 13 to 17 are machinery that exists in Cursor only because of its ripgrep subprocess, its JavaScript wrapper or its proto layer. They are dropped and not reproduced.

| # | What Cursor does | What Runvil should do | Why |
|---|---|---|---|
| 1 | Sorts newest modification time first in JavaScript under a 2000 ms stat budget, and falls back to walk order with no marker | Sort by full-path bytes and stat nothing | Replay. A time sort costs a gate, a claim and an event per match |
| 2 | Applies three caps (2000 entries, 10000 lines, 5000 UTF-16 code units) that bind at about 160 files, and has no paging | Apply one 15,000-byte budget on the path lines, the constant Grep shares, and add `offset` | Bounded cost and Few calls |
| 3 | Prints `(total N files)`, drops the count when ripgrep truncated, and can print 5000 over about 160 lines | Print the exact total in the first line, always | `Access::glob` returns the whole list, so the total is never unknown |
| 4 | Ends a cut list with a mid-list notice or a banner, and flags a cut at exactly 10000 lines | End with one notice, only when paths remain | An exact fit is not a cut |
| 5 | Rewrites `\` to `/`, strips one leading and one trailing `/`, and prepends `**/` | Prepend `**/` only, keep a leading `/` as absolute, refuse a backslash in the pattern, read one in `target_directory` as `/`, and read a trailing `/` as `/**` | The VFS defines absolute patterns and refuses backslashes on purpose |
| 6 | Turns an absolute pattern into a dead suffix match and answers zero | Pass a full VFS path pattern through | A silent zero on a well-formed input is the worst answer |
| 7 | Matches the pattern case-insensitively with `--iglob` | Match case-sensitively and say so in a zero reply | The grammar cannot fold case, and five of eight harnesses agree |
| 8 | Passes a whitespace-only pattern through as a literal and renders `0 files found`, and accepts a file as `target_directory` | Trim and refuse an empty pattern, and refuse a file or a missing directory as the target | Loud result |
| 9 | Lets a positive glob beat `.gitignore`, leaves `.gitignore` inert outside a git repository, and lets `.cursorignore` beat the glob, through three mechanisms | Hide `.git` and the paths a `.gitignore` names after the match, through one shared Pack module with no git requirement and a count, and move the rule into `Access` later | One module and one precedence, so Glob and Grep agree. A directory the model names is still searched |
| 10 | Includes hidden files with `--hidden` and excludes `.git` with a literal `--iglob` pair | Keep hidden files visible, except `.git` and the paths a `.gitignore` names, which the shared module leaves out and announces with a count | Hidden files are matched by `*` already, and the exclusion has to be visible |
| 11 | Follows every link with `--follow` | Enter no linked directory in a walk, list no links, and say so. A `target_directory` that names an in-root directory link is walked through it | The walk stays inside the root and cannot loop |
| 12 | Prints a header and `- path` bullets, relative to the target, with `.\` on Windows | Print bare absolute VFS paths | A path copied into Read must work, on every host |
| 13 | Retries with `pattern: "."` when the executor rejects an empty pattern, which flips ripgrep into content mode and drops empty and binary files | Drop it. `Access::glob` matches names and reads no content | There is no executor to reject an empty pattern |
| 14 | Renders `glob_file_search didn't return the result` on all seven branches when no case is present | Drop it. The Pack always returns text | A proto-layer artifact that names a tool most branches do not have |
| 15 | Reports a missing ripgrep binary as `Path does not exist: <cwd>`, and bounds a call by a 25 s deadline, an 8 MiB stdout cap and two argv builders | Drop all of it | There is no subprocess |
| 16 | Ships `glob_file_search` on one branch and `Glob` on six, and throws `Unhandled version` on an unknown tag | Ship one name, `Glob` | Version machinery for per-model surfaces |
| 17 | Searches each workspace root in turn, sorts per root and sums the totals | Search one namespace and sort once | A VFS pattern is served by one mount. See Table 9 for the cost |
| 18 | Swallows unreadable subdirectories and symlink loops and returns the rest with no notice | Fail the call loudly and name the directory, until the VFS reports skipped entries | Loud result |
| 19 | Takes drive letters and UNC paths in `target_directory`, and prints a non-UTF-8 name as replacement characters that the model cannot open | Drop drive letters, since VFS paths use `/`. A non-UTF-8 name fails the call, as the VFS decides | The model never sees a host path |

*Table 7. Nineteen departures from Cursor's `Glob`. Sources: [tool-glob.md](tool-glob.md), sections The short version, Decisions 1 to 7, Algorithm and ordering, Failure matrix and Traps; Runvil `operations.rs`, `glob.rs`, `real.rs` and `router-access.rs`.*

Confidence for each row is the confidence of the behavior it names.

Binary detection does not apply to Glob, in Cursor or here: the tool lists names and reads no content (Q5). The one place a content read could creep in is the empty-pattern retry in row 13, and it is dropped.

## Mapping to Runvil

Table 8 maps each behavior to the `Access` method that covers it, names the owner, and gives the next step as a todo id from the standing plan in `runvil/vibe/2026-10-27-2-plugin-std-file-tools.md`. Rows 2, 4 and 9, and the failure row of Table 9, need a VFS item that the plan does not list. The plan edits at the end of this section propose one todo for them, `vfs-glob-gaps`.

| # | Behavior | `Access` method | Status | What the gap needs | Owner | Next step |
|---|---|---|---|---|---|---|
| 1 | Byte-order sort | `glob`, which both backends sort | Covered | The Pack re-sorts by bytes in case a backend sorts otherwise | Vinnie | `glob-real` |
| 2 | Ignore rules in `Access`, and one shared Pack module until then | none. `Policy::check` sees only the pattern | Missing | A per-match filter beside `Policy::check`. Meanwhile one shared Pack module for `.git` and `.gitignore`, built on the `ignore` crate or a small matcher | Vinnie | `glob-real` writes the module, `grep-real` calls it, and `vfs-glob-gaps` (new) owns the filter |
| 3 | No bare zero | `glob` returns an empty `Vec` | Covered | Pack wording only | Vinnie | `glob-real` |
| 4 | Paging and exact total | `glob` returns the whole list | Partial | No limit or early stop. The real backend collects every entry under the walk root before it matches, and the Pack holds the whole list | Vinnie | `glob-real`, and a VFS limit later in `vfs-glob-gaps` |
| 5 | Silent exact fit | none needed | Covered | One comparison in the Pack | Vinnie | `glob-real` |
| 6 | Recursive prefix | `glob`, which joins a relative pattern onto the root, with `**/` matching zero segments | Covered | The Pack builds the pattern | Vinnie | `glob-real` |
| 7 | Absolute patterns | `glob`, where a leading `/` is absolute | Covered | The refusal beside `target_directory` is Pack code | Vinnie | `glob-real` |
| 8 | Case-sensitive match | `glob`, a byte match | Covered | A case-insensitive option would need a VFS flag | Vinnie | `glob-real` |
| 9 | Grammar and the brace hint | `glob`, with four tokens | Partial | No alternation. Classes and `?` need an escape the grammar lacks. Grep's `glob` filter needs the matcher as one pure function | Vinnie | `glob-real` for refusals and, in its first commit, the exposed matcher that Grep's `glob` filter shares, and a VFS grammar item (`vfs-glob-gaps`) for `{a,b}` |
| 10 | Empty pattern and target check | `stat`, and `PathReason::Empty` | Partial | `glob` answers a missing walk root with an empty list, so the Pack stats first and trims the pattern. An undeclared parameter passes the schema check, so the shared argument helper in `check.rs` refuses it | Vinnie | `glob-real`, with the helper that `read-real` adds |
| 11 | Files only, links unlisted, `X/` as under | `glob` without a trailing slash | Partial | A link leaf appears in neither mode, and nothing reports links. Directories-only mode waits for `ListDir` | Vinnie | `glob-real`, and `listdir` |
| 12 | Absolute bare path text | `glob` with an absolute pattern | Covered | The Pack leaves out names with a control character and counts them | Vinnie | `glob-real` |
| 13 | One error renderer | the `VfsError` variants | Partial | The shared `refusal` function in `plugin-std`. `ApprovalRequired` text now, real approval later | Vinnie | `refusal-text`, `glob-real` |
| 14 | Description | `glob.json` | Partial | Rewrite the text, add `offset`, correct `EDITS.md` | Vinnie | `glob-real` |
| 15 | Mark output untrusted | `ToolOutput::untrusted` | Covered | Return `untrusted` for any reply that lists a path. A zero reply that holds only the model's own pattern and the hints may be trusted | Vinnie | `glob-real` |

*Table 8. Mapping from behavior to Runvil. Sources: Runvil `operations.rs`, `glob.rs`, `memory.rs`, `real.rs`, `real/files.rs`, `traits/backend.rs`, `policy.rs` and `error.rs` in the VFS crate; the `plugin-std` crate and the standing plan.*

### What `Access::glob` does today and what it lacks

Table 9 lists what `Access::glob` provides and the gaps that shape the tool. Every row is read from the source. Rows marked "read, not run" are my reading of the code and no test of mine ran them.

| Topic | Today | Gap and its effect |
|---|---|---|
| Validation | Refuses an empty pattern, over 1,024 bytes, a control character, a backslash and a bad wildcard, each as `InvalidPath` with a `PathReason` | None. The Pack renders each reason |
| Gate and claim | Checks the policy on the pattern and claims the pattern. A glob and a sibling's write conflict in both orders (tested) | The policy cannot hide one match inside a result |
| Kind | Files, or directories with a trailing `/` | A link leaf appears in neither mode (read, not run) |
| Order | Sorted by byte order of the full path on both backends | The trait names no comparison |
| Hidden files | `*` matches a leading dot, so `.hidden` is listed (tested) | None |
| Case | A byte match, so case-sensitive on every backend | No case-insensitive option |
| Grammar | `*`, `**` and `**/`, which matches zero segments | No braces, classes, `?` or negation, and no escape |
| Root and mounts | A relative pattern joins onto the root, and `..` past the root is `InvalidPath` (tested). One pattern is served by one mount | A pattern from `/` does not walk a mount at `/mnt` (read, not run) |
| Result | A `Vec<String>` of every match | No limit, no offset, no metadata and no count of skipped entries |
| Cost | The real backend walks the whole subtree under the pattern's literal prefix, collects every entry, then matches, and the default `glob_kind` stats each match | No deadline, and memory grows with the tree |
| Failure | One unreadable directory or one non-UTF-8 name fails the whole call, and so does a file removed between the walk and the stat (read, not run) | No partial result |
| Missing directory | An empty list from both backends | Cannot tell "absent" from "no match" |

*Table 9. What `Access::glob` does and lacks. Sources: Runvil `operations.rs` lines 137 to 204, `glob.rs`, `memory.rs`, `real.rs`, `real/files.rs`, `traits/backend.rs`, `router-access.rs`, and the tests `regions.rs`, `memory-tests.rs` and `router-tests.rs`.*

### Tests to add to the crate

Each test maps to a behavior's Test line. Unit tests sit in a `glob-tests.rs` sibling and integration tests in `tests/it/`, following the crate's layout.

- A byte-exact reply table for the found, singular, zero, cut, last-page and past-the-end cases, run twice for replay and importing `LIST_CAP_BYTES` and the notice texts by name. It includes `Found 1 file matching` for the singular and the control-character line for one and for two names.
- An order test on both backends, with a reverse-sorting test backend and reversed modification times.
- A pattern table for the prefix, `**`, an absolute pattern, a trailing slash, `target_directory`, and a logging backend that shows the pattern reaching `Access::glob`.
- Paging tests: 1,000 paths of 30 bytes, the exact-fit totals 0, 1, 482, 483 and 484, one 16,000-byte path, and the CJK names.
- A zero-hint table over each hint and over a pattern that earns none.
- Input-check tests for empty, spaces, slashes, a missing target, a file target, an empty target, a backslash in `target_directory` and an undeclared parameter, with the logging backend showing one `Stat` and each message showing the argument as written.
- Link tests: a file link on Unix, a directory link or junction on both, and a `target_directory` that names a directory link.
- Ignore-module tests: `.git` as a file and as a directory, a `.gitignore` with `!` rules and no `.git` directory, an ignored `target_directory` that is searched, a pattern that cannot bring an ignored path back, and the singular and plural count lines. Once `Access` has the rule, the same cases run against the filter.
- Case tests, including the Windows temporary directory that pins a byte match.
- Grammar tests: refusals for `**b`, `***`, a backslash and a 1,100-byte pattern, and the brace hint, run against the matcher that Grep's `glob` filter calls as well.
- An error table over the variants, including a real `Conflict` from a sibling scope and a real `NotUtf8` on Unix, and an assertion that no message names another tool.
- Path-text tests: only `\n` line ends, no `\r`, no `\` on Windows, and the control-character name on Unix.
- A trust test: a reply that lists a path is `OutputTrust::Untrusted`.
- A test that pins today's mount behavior: a pattern from `/` lists no file under a mount at `/mnt`, so the day it changes the change is visible.
- Spec tests over `glob.json` as described in behavior 14.

### Schema and description fixes to make

These are listed and not made here. The crate and its schemas were not changed. `schemas/EDITS.md` says one edit was made to `glob.json`, but the shipped text differs from both Cursor variants in the description and in both parameter texts, so the file needs a correction whatever else changes.

```json
{
  "wire_name": "Glob",
  "description": "Finds files by name pattern and returns their paths, sorted by path. A pattern that does not start with \"**/\" or \"/\" is searched at any depth, so \"*.rs\" finds every Rust file and \"src/**/*.ts\" finds TypeScript files under any src directory. A pattern that starts with \"/\" is a full path from the root of the run's filesystem. A trailing \"/\" means the files under that directory. Matching is case-sensitive. Only * and ** are special: {, [ and ? match themselves, so call once per extension. A walk does not enter linked directories, and links are not listed. Anything under .git and every path a .gitignore names are left out, and other hidden files are listed. Read opens any path you name, including the paths this tool leaves out. Paths in the reply are absolute from the root of the run's filesystem, and work unchanged as the path argument of the other file tools. A reply is cut at 15000 bytes of paths: pass offset to read the next page.",
  "parameters": {
    "type": "object",
    "properties": {
      "glob_pattern": {
        "type": "string",
        "description": "The glob pattern to match files against. Examples: \"*.js\" (every .js file at any depth), \"src/**/*.ts\", \"**/test/**/test_*.ts\", \"/docs/*.md\" (only the top level of /docs)."
      },
      "target_directory": {
        "type": "string",
        "description": "The absolute path of the directory to search in. If not provided, the run's filesystem root is used. Do not combine it with a glob_pattern that starts with \"/\"."
      },
      "offset": {
        "type": "integer",
        "description": "The number of paths to skip, taken from the notice at the end of the previous reply. Default 0."
      }
    },
    "required": ["glob_pattern"]
  }
}
```

Six plan edits follow. They are proposals: the standing plan stays as it is, and `file-tools-index.md` collects them.

1. Rewrite the `glob-real` todo to say what the confirmation found. Replace "after confirming recursion, case sensitivity, and dotfile behavior against Cursor's ripgrep walk" with "with the `**/` prefix, case-sensitive matching and `*` matching dotfiles, as `master-glob.md` confirms". Add that `glob-real` writes the shared ignore module that `grep-real` calls.
2. Add one todo, `vfs-glob-gaps`, for the VFS items: a per-match ignore filter that deletes the module, partial results through `Access::glob_with_skips`, which returns the matches and a `Skipped` list of path and reason, shared with `Access::list_with_skips`, `{a,b}` alternation, and a limit with an early stop. The glob matcher is exposed as one pure function in the first commit of `glob-real`, not here, because Grep's `glob` filter needs it before `vfs-glob-gaps` runs.
3. Amend step 6, which says to adjust in the Pack rather than change shared VFS behavior. Append "The exceptions are the ignore filter, partial results and `{a,b}`, which cannot live in the Pack (`vfs-glob-gaps`)."
4. Record in the Decision Record that `glob.json` is no longer Cursor's text, that the Pack hides `.git` and the paths a `.gitignore` names until `Access` owns the rule, and the new todo ids `refusal-text`, `vfs-glob-gaps` and `vfs-mode-text`.
5. Put the todos in this order: `grep-schema-fix`, `refusal-text`, `read-real`, `write-real`, `replace-real`, `delete-real`, `glob-real`, `grep-real`, `vfs-glob-gaps`, `approval`. The `approval` todo then only wires real approval. The plan's Non-goals and Out of scope lines also allow the `Access` additions that `vfs-glob-gaps` makes.
6. If the shared module takes the `ignore` crate, add it to the workspace dependencies, in the plan's dependency sentence. `globset` is not added directly, although the `ignore` crate depends on it.

Confidence in these plan edits is high, because they only make the plan text match the behaviors above. The new todo ids are proposals, confidence medium.

## Risks and open limits

Table 10 lists the risks. Likelihood uses the fixed ladder and rates the event. The confidence column rates the evidence behind that likelihood, and it is a separate judgment.

| # | Risk | Likelihood | Confidence in the evidence | What would change the recommendation |
|---|---|---|---|---|
| 1 | Models trained on Cursor ask for "the file I just edited" and get path order | Unlikely | Low. No source measures how often models ask | Transcripts that show repeated paging to find recent files. Then add a VFS call that returns the stats the backend already takes, with an optional recency order |
| 2 | Models send brace patterns, and each costs an extra call | Likely | Low. Copilot's description teaches the form, and no frequency is measured | Zero replies that carry the brace hint above a few percent of calls. Then move alternation to the front of the VFS work |
| 3 | A wrong-case pattern returns zero and costs a call | Unlikely | Low. Cursor and Copilot choose the other way, with no transcript data | Zero replies that carry the case hint and then succeed on the next call. Then add the VFS flag |
| 4 | The 15,000-byte page is too small or too large | Roughly even chance | Low. The number is Grep's cap, shared, and no source measures an adequate page | Transcripts. Tune `LIST_CAP_BYTES`, which moves Grep's cap with it. The design does not change |
| 5 | The shared ignore module hides a path the model wanted | Unlikely | Low. Cursor, Gemini and `pi_find` exclude `.git` by name, and no source measures how often a model wants an ignored directory | A task that works inside `.git` or an ignored directory without naming it as `target_directory`. Then add a hint to the zero reply, or a parameter |
| 6 | The shared ignore module misreads a `.gitignore` rule, so a wanted file disappears or a generated tree fills the listing | Roughly even chance | Low. No source here ran a matcher over real `.gitignore` files, and the matcher is an open choice | The first run on a real project that misses a file or pages through generated files. Then take the `ignore` crate if the module has a matcher of its own, and move the rule into `Access` |
| 7 | A symlinked file never appears in a listing | Unlikely | Medium. Read from source and not run | A test that lists a link, or a user report. Then report links in the VFS result |
| 8 | One unreadable directory or non-UTF-8 name fails the whole call on a real mount | Roughly even chance | Medium. Read in `walk`, not run | A real mount where it happens. Then the skip count moves to the front of the VFS work |
| 9 | A recursive pattern from `/` omits files under other mounts | Roughly even chance | Low. No source says how Workshop mounts backends | Workshop mounting more than one backend. Then a pattern must walk every mount under its base |
| 10 | A glob conflicts with a write in the same turn and the call fails | Unlikely | Medium. Tested for sibling scopes, and the `ToolContext` doc says calls in one round fork their identities together, so they conflict whatever their timing. No round has been run | A Harness that forks no identity per call, so a batch never conflicts, ends the risk. A rate of conflicts that wastes rounds means the Harness should order the batch |
| 11 | A pattern with a precomposed character misses a macOS name stored decomposed | Very unlikely | Low. No harness normalizes and none measured it | A non-English repository on macOS. Then normalize both sides in the VFS |
| 12 | The real backend's whole-tree `Vec` exhausts memory on a very large tree | Unlikely | Medium. Read in `walk`, not run | A tree with millions of entries. Then add a limit with an early stop |

*Table 10. Risks, with likelihood and confidence kept apart. Sources: Runvil `real.rs`, `real/files.rs`, `router-access.rs` and `handle/tests/regions.rs`; [tool-glob.md](tool-glob.md), the Copilot, Zed, Gemini and Q3 material.*

### What the evidence does not settle

Five limits remain, and each could move a recommendation. First, no source measures how models behave with these tools: the page size they find adequate, how often they send braces, how often they want the newest file, how often they write the wrong case, and how often they look for a link. Every number in this report that is a choice is marked low for that reason. Second, three behaviors are read from source and not run: the omission of link leaves, the single-mount routing of a pattern, and the failure on one bad name or a vanished file. The tests listed under Mapping to Runvil settle each. Third, the plan does not say how Workshop builds a run's mounts, so I rate row 9 of Table 10 low. Fourth, the per-match filter in `Access` is a contract here (a precedence, a count, no git requirement) and not a design, so its API and its rule sources are open. Fifth, Cursor's own order may come from an index in some builds, because the evidence report could not tell whether the indexed provider serves a Glob call ([tool-glob.md](tool-glob.md), What remains unknowable), so the comparison is to the documented path.

One choice inside the shared ignore module is open, and it belongs to Vinnie: the `ignore` crate or a small matcher of its own. The `ignore` crate, version 0.4.33, matched gitignore lines and directory pruning against virtual paths with no file system in Grep's probe (`master-grep.md`, Behavior 10, the ignore behavior). The workspace has no `ignore` crate today, so it is a new dependency, and its manifest lists `globset` and `walkdir` among its own. A matcher of its own adds no dependency. It costs code, which `master-grep.md`, in its Risks and open limits, sizes at about the glob grammar already in the VFS for a `.gitignore` matcher and a file-type table together, plus tests for `!` rules, directory-only rules, anchoring and nested `.gitignore` files. Either way the `.git` rule and the count line stay the same, and the VFS filter later deletes the module.

A `Conflict` that the Pack maps to a `ToolError` reaches the model as failure text and does not end the run (`scheduler/tool_call.rs`, read, not run).

This report departs from the evidence report in three places. It narrows Decision 3's option D to path order alone, for the replay reason under Table 2. It rates the announcement of ignored paths medium and not high, because the field report cited for Decision 2 concerns binary and empty files and not ignore silence. And it settles case sensitivity, which the evidence report leaves pointing both ways.

Three things would change the main call. If transcripts show that finding recent files is a common task, order moves to a stat-bearing VFS call with a newest-first option, and path order stays the default for replay. If Workshop runs with several mounts, the single-mount gap becomes the first VFS item, because a recursive pattern from the root would omit files with no notice. If the VFS filter is designed so that a pattern that names an ignored path can bring it back, behavior 2 changes and so does the zero-reply hint. The recommended default stands on today's evidence: call `Access::glob`, sort by path, page with `offset`, and never let a zero or a cut go unexplained.

## Sources

Evidence reports, by file name, both in this folder:

- [tool-glob.md](tool-glob.md), the primary ground: Cursor's `Glob` read from the 3.27.12 bundle and daemon, enriched with twelve harnesses and edited for prose, final on 2026-10-28. Cited by section name.
- [file-tools-index.md](file-tools-index.md), for the shared facts and the parent spec's corrections, including the two ripgrep builders and the ignore-precedence correction.

Sources behind those reports and used here, each with its tier. Cursor offsets are those the evidence report gives. Where I re-read a file for this report, the lines are named.

| Source | Tier | Used for |
|---|---|---|
| Cursor 3.27.12, `cursor-agent-exec/dist/main.js`, 9,814,287 bytes: `GLOB` registration at 4292676, the Glob region 4291000 to 4295600, description at 4292721, schemas at 4289658 and 4290208, render budget at 4290699 | `(bundle)` | Normalizer, caps, templates, description, schemas |
| Cursor 3.27.12, `cursor-agent-host/dist/agent-host-daemon/dist/bin/daemon.cjs`, 20,213,626 bytes: `normalizeGlob` at 12712190, `sortNewestFirst` region 16641900 to 16644100, `buildArgs` at 16780893, `walk2` at 16844366 | `(daemon)` | Sort and stat budget, argv, `pi_find` walk |
| ripgrep 15.1.0-cursor5 (rev c3e3c2f7ec) at the Cursor install: nine probe scripts `rgprobe.js` to `rgprobe10.js`; rig probes against `ignore` 0.4.25 and `globset` 0.4.18; `pi_find` and zeroclaw probes; one live Glob call in this session | `(probe)` | Ignore precedence, case, sort direction, caps, exact-fit counts, output format |
| Claude Code 2.1.311 win32-x64, `glob.js` extracted from `claude.exe`: I re-read lines 46, 84, 218 and 334 | `(source)` | `--sort=modified`, the slice, the notice, the 100 cap |
| VS Code repository at `b208bc25` (2026-10-25): `extensions/copilot/src/extension/tools/node/findFilesTool.tsx`; I re-read lines 68, 76, 137 to 150 and 172 to 181 | `(source)` | Case option, pattern rewrite, caps, output |
| Zed `37faef81f7` (2026-10-27): `crates/agent/src/tools/find_path_tool.rs`; I re-read lines 18, 21, 39, 59 to 72, 89, 143 to 144 and 184 to 186 | `(source)` | Order, paging, range line, empty glob |
| pi `a92283056` (2026-10-27): `packages/coding-agent/src/core/tools/find.ts`; I re-read lines 199 to 212 | `(source)` | The slash rule and `--max-results` |
| rig `a83b774d0` (2026-10-25): `crates/rig-tools/src/search.rs`, `workspace.rs`; zeroclaw `2d85371f3` (2026-09-18): `glob_search.rs` | `(source)` `(probe)` | Override precedence, exact-fit counts, cut before sort |
| Gemini CLI `v0.66.0`, `970413207732`; OpenCode `v1.19.5`, `810c0eba1a73` (dev `fa1260d6ca`); Cline `v4.1.26`, `a459f1d`, and legacy `v4.0.12`, `f267bf4`; Aider `5dc9490bb35f`; Goose `ee8500d5e060` (tag `v1.56.0`); Codex CLI `bea47c06dfac` | `(source)` `(probe)` | Case default, caps, truncation, tracked-files rule, `require_git(false)` |
| [Glob does not work the way agents expect it to work](https://forum.cursor.com/t/glob-does-not-work-the-way-agents-expect-it-to-work/168477), Cursor forum, October 2026 | `(web)` | The silent drop of binary and empty files |
| Runvil VFS crate, `crates/runvil-internal/vfs/src/` at `08801150fc` (2026-10-27): `lib.rs` (`ModePolicy`), `handle/operations.rs`, `glob.rs`, `memory.rs`, `real.rs`, `real/files.rs`, `traits/backend.rs`, `traits/policy.rs`, `stat.rs`, `error.rs`, `error-reason.rs`, `router.rs`, `router-access.rs`, `handle/claims_tree.rs`, `handle/tests/regions.rs`, `real/tests/links.rs`, `router-tests.rs`, `memory-tests.rs` | `(source)` | `Access::glob`, grammar, order, backends, claims, error variants, links, mode policy |
| Runvil `crates/plugin-std/`: `schemas/glob.json`, `schemas/read.json`, `schemas/EDITS.md`, `src/spec-tests.rs`, `src/pack.rs`, `src/check.rs` | `(source)` | The shipped text, the edit log, the spec tests, the stub, the argument check |
| Runvil `crates/runvil-plugin/src/context.rs`, `crates/runvil-internal/engine/src/execute/scheduler/tool_call.rs`, `crates/runvil-internal/types/src/tools/output.rs` | `(source)` | Calls in one round fork their identities together, a model-issued call resumes with content, untrusted output |
| Runvil workspace `Cargo.toml`, and the `ignore` 0.4.33 manifest in the local cargo registry | `(source)` | The workspace has no `ignore` or `globset`, and `ignore` depends on `globset` and `walkdir` |
| Runvil `vibe/2026-10-27-2-plugin-std-file-tools.md` | `(source)` | Todo ids, non-goals, the Decision Record, step 6 |

All Runvil files were read from the working tree, which was clean for these files at `08801150fc`. No Runvil code or schema was changed.

*2026-10-27 - Claude Sonnet 5.5*
