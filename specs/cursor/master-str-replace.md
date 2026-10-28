# Build StrReplace as a pure matcher in the std Pack: exact scan, overlap guard, line-trimmed fallback, splice into the original bytes

## Summary

I recommend that Runvil's `StrReplace` own its matching in the `std` Pack: read with `Access::read_string`, run a pure `apply` over the text, and write once. Table 1 lists the fourteen behaviors.

Five decisions matter most. First, keep the matcher out of the VFS. Its default `str_replace` counts with `str::matches`, so it passes `aa` in `aaa` as unique, and Lua scripts still use it (high confidence). Second, scan for the exact text, advance by the needle's length, and ship Cursor's overlap guard with it (high). Third, splice into the original bytes so no untouched byte changes (high). Fourth, keep Cursor's one-to-one line-trimmed fallback and tell the model when it ran (high for the fallback, medium for the note). Fifth, accept an optional `edits` array applied in order, because sibling calls on one path conflict in Runvil and a Pack may not wait (low).

The largest risk is the array's order. Rig reversed to in-order application on 2026-10-19, and the reason is not in its repository. I read the commit and found a title, a one-line body and no benchmark. One test pins the order.

Confidence is high on matching and byte preservation, medium on line endings, stale diagnosis and the success note, and low on the array. Vinnie's next step, under `replace-real`, is `apply` over a slice of edits, then the two-parallel-calls test, which confirms that a conflict does not end the run.

## Contents

- [Summary](#summary)
- [The recommendation in brief](#the-recommendation-in-brief)
- [Criteria](#criteria)
- [Options and the do-nothing case](#options-and-the-do-nothing-case)
- [Behaviors to implement](#behaviors-to-implement)
  - [1. Own the matching in the Pack as a pure function over text](#1-own-the-matching-in-the-pack-as-a-pure-function-over-text)
  - [2. Scan for the exact text first, count by the needle's length, and ship the overlap guard with it](#2-scan-for-the-exact-text-first-count-by-the-needles-length-and-ship-the-overlap-guard-with-it)
  - [3. Splice by offset into the original text so no untouched byte changes](#3-splice-by-offset-into-the-original-text-so-no-untouched-byte-changes)
  - [4. Fall back to a one-to-one line-trimmed match only when the exact scan finds nothing](#4-fall-back-to-a-one-to-one-line-trimmed-match-only-when-the-exact-scan-finds-nothing)
  - [5. Refuse a file that is not valid UTF-8 instead of decoding it lossily](#5-refuse-a-file-that-is-not-valid-utf-8-instead-of-decoding-it-lossily)
  - [6. Match on an LF view with the BOM removed and map offsets back to the original](#6-match-on-an-lf-view-with-the-bom-removed-and-map-offsets-back-to-the-original)
  - [7. Refuse, do not wait, when a sibling call holds the path](#7-refuse-do-not-wait-when-a-sibling-call-holds-the-path)
  - [8. Accept an optional `edits` array, applied in order and all or nothing](#8-accept-an-optional-edits-array-applied-in-order-and-all-or-nothing)
  - [9. Diagnose a stale edit on a miss and never gate on a read](#9-diagnose-a-stale-edit-on-a-miss-and-never-gate-on-a-read)
  - [10. Name the cause and the next move in every refusal](#10-name-the-cause-and-the-next-move-in-every-refusal)
  - [11. Say what happened on success: Cursor's sentence, plus a note when the fallback ran](#11-say-what-happened-on-success-cursors-sentence-plus-a-note-when-the-fallback-ran)
  - [12. Reject an empty needle and an identical pair after line-ending normalization, and allow a de-indent](#12-reject-an-empty-needle-and-an-identical-pair-after-line-ending-normalization-and-allow-a-de-indent)
  - [13. Remove the whole line when an empty replacement leaves nothing on it](#13-remove-the-whole-line-when-an-empty-replacement-leaves-nothing-on-it)
  - [14. Refuse files over 8 MiB and bound the match bookkeeping](#14-refuse-files-over-8-mib-and-bound-the-match-bookkeeping)
- [Where Runvil should differ from Cursor](#where-runvil-should-differ-from-cursor)
- [Mapping to Runvil](#mapping-to-runvil)
- [Risks and open limits](#risks-and-open-limits)
- [Sources](#sources)

## The recommendation in brief

**Table 1.** One row per behavior, in the order of the behaviors section, with the recommendation and its confidence.

| Behavior | Recommendation | Confidence |
|---|---|---|
| 1. Own the matching in the Pack as a pure function over text | Write `apply(text, edits)` with no I/O; ask `Access::check_write` first, read with `read_string`, write with `write`; keep `Access::str_replace` for Lua only | High |
| 2. Scan for the exact text first, count by the needle's length, and ship the overlap guard with it | Advance by needle length, refuse a self-overlapping needle when `replace_all` is false, splice `new_string` as literal text | High |
| 3. Splice by offset into the original text so no untouched byte changes | Replace byte ranges of the original; never rebuild the file from trimmed lines | High |
| 4. Fall back to a one-to-one line-trimmed match only when the exact scan finds nothing | Cursor's fallback, byte for byte, plus one divergence for a whitespace-only last line | High |
| 5. Refuse a file that is not valid UTF-8 instead of decoding it lossily | Map `NotUtf8` to a sentence naming the path and the byte offset | High |
| 6. Match on an LF view with the BOM removed and map offsets back to the original | One LF view for both sides, splice through an offset map, inserted breaks take the starting line's ending | Medium |
| 7. Refuse, do not wait, when a sibling call holds the path | Map `VfsError::Conflict` to a sentence pointing at one call per file, the `edits` array or the next message | Medium |
| 8. Accept an optional `edits` array, applied in order and all or nothing | Optional second form beside Cursor's four parameters; order pinned by a test | Low |
| 9. Diagnose a stale edit on a miss and never gate on a read | Per-run hash record; one sentence on a miss, one clause on a stale success | Medium |
| 10. Name the cause and the next move in every refusal | Name the path in every refusal; count and 8 line numbers on ambiguity, a labelled excerpt on a miss, both numbered as Read numbers lines; tie-break by line order | High |
| 11. Say what happened on success: Cursor's sentence, plus a note when the fallback ran | Cursor's sentence as prefix; one added sentence when the fallback matched | Medium |
| 12. Reject an empty needle and an identical pair after line-ending normalization, and allow a de-indent | Compare only line-ending-normalized strings; never trim; build each argument error with the shared helper | High |
| 13. Remove the whole line when an empty replacement leaves nothing on it | Widen the match to its line and newline, computed against the original text | High |
| 14. Refuse files over 8 MiB and bound the match bookkeeping | `stat` before the read; keep 8 offsets, or splice while scanning | Medium |

## Criteria

Six standards judge every option below. The first two decide ties, because a wrong edit that reports success costs more than any amount of tokens or code.

- **Right edit.** The model succeeds in few calls and gets no silent wrong result. A refusal that tells the model what to change counts as success on the next call. A success that changed a region the model did not mean counts as failure.
- **Bytes survive.** Every byte outside the matched region stays identical, and no file is re-encoded, re-indented, or re-terminated by the edit.
- **Replay.** The same file bytes and the same arguments give the same output bytes and the same text, so a run can replay.
- **Bounded cost.** Tokens, memory and time stay bounded, and the description or the error text tells the model about each bound.
- **Fits the VFS.** The behavior can be built on `Access` as it stands today, and each gap is named with the call that would close it.
- **Familiar surface.** A model that knows Cursor's `StrReplace` needs no new instructions. The standing plan lists this as a product requirement, and the owner has said he will diverge where the evidence supports it, so it is the weakest of the six and gives way to the others.

The evidence for each judgment carries a tier. `(source)` is code read at a pinned commit, `(probe)` is code run, `(daemon)` and `(bundle)` are Cursor's shipped code, and `(web)` is documentation. Confidence is high, medium, or low and describes the evidence, never the likelihood of an event. Likelihood uses the ladder very unlikely, unlikely, roughly even chance, likely, very likely, almost certain, and appears only in the risks section.

The needle is the `old_string` text the tool searches for. The view is the LF-normalized copy of the file text that the matcher reads. The fallback is the line-trimmed block match that runs when the exact scan finds nothing. A sibling call is another tool call from the same model round. The record is the per-run read record of what the model last saw of a file: a hash, a shown range, a total and a whole-file flag (`master-read.md`, "Publish a read record to a per-run service, and keep the render pure"). The hint line is the file line that locates the excerpt shown on a miss.

## Options and the do-nothing case

Seven decisions have real alternatives. Each table names the do-nothing option and marks the recommended one. The evidence behind the cells is in [`tool-str-replace.md`](tool-str-replace.md), under its `Decisions you have to make` section, unless a cell cites Runvil source.

### Matching belongs to the Pack, because the VFS default has the overlap defect

Ownership is the largest choice, because it decides the file layout (Table 2). The evidence report recommends Option A at high confidence and names the open read-modify-write window as its cost. Runvil's source narrows that cost. A call's own read and write are ordered by its identity, so they never conflict with each other. A sibling call in the same round that touches the same path gets `VfsError::Conflict`. The window that stays open is a writer outside the claims ledger, such as an editor or another process, and the VFS cannot see one today (source: `handle/claims.rs`, `handle/scope.rs:156-159`, `runvil-plugin/src/context.rs`).

**Table 2.** Who owns the matching. The recommended row is marked. The do-nothing row keeps the stub that ships today.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: keep the stub | the Pack answers "not implemented" | no new code | the model cannot edit any file, so no Runvil coding agent works | Reject |
| Minimal: call `Access::str_replace` unchanged | the VFS | one call, one write claim, no Pack matching code | no `replace_all`, no fallback; `aa` in `aaa` passes as unique because the default counts with `str::matches`; `Anchor` carries a count but cannot report overlap or relaxation | Reject as the whole answer |
| **A. The Pack reads, matches, writes (recommended)** | the Pack: `read_string`, a pure `apply`, `write` | one module for all semantics; testable with no filesystem; reaches the overlap guard and the fallback; rig ships the same split | `Access::str_replace` goes unused by the tool; read and write claim separately, though one call identity orders them | **Recommended** (high: the VFS default has the overlap defect) |
| B. The Pack delegates the exact case | the Pack counts, then calls `Access::str_replace` | a backend could push the replacement down for the common case | two paths that must agree; the Pack already matched, so the delegation is redundant; two error texts for one condition | Reject |
| C. The VFS gains a match policy | the VFS backend | one claim covers the whole edit | the VFS learns what a line is; every backend implements or inherits it | Reject for now |

Three Runvil facts back Option A beyond the evidence report. First, the default backend `str_replace` counts with `text.matches(old).count()` and replaces with `replacen(old, new, 1)`, so it has the defect that the evidence report finds in zeroclaw. No in-tree backend overrides it. The VFS crate's eight `str_replace` tests use a unique needle, a missing one, `foo` twice, an empty one and a non-UTF-8 file, and none uses a self-overlapping needle (source: `traits/backend.rs:215-244`, `traits/tests.rs`, `handle/tests/operations.rs`). Second, `Access::str_replace` is also the primitive behind Lua's `store.str_replace`, so retiring it would break scripts (source: `lua/src/engine_globals.rs:338`). Third, `Anchor { path, anchor, count }` reports only a count, so no caller can learn that a match needed relaxation or that two matches overlapped.

Option B lost because the Pack has to match before it can choose a path, so the delegation duplicates work it already did and leaves two error texts for one condition. Option C lost because it asks the VFS to learn what a line is, which a layer that speaks paths and bytes does not need. It becomes worth revisiting only if a backend must edit in place, such as a database or an object store. The router already comments that `str_replace` is "Delegated so backends can push down" (source: `router-access.rs:207`), but no backend does today.

### Keep Cursor's line-trimmed fallback, because exact-only tools fail on tabs and non-breaking spaces

The fallback is the one tolerance the evidence supports, and the designs set aside differ in how many bytes they change (Table 3).

**Table 3.** What the Pack does when the exact scan finds nothing. The do-nothing row is Claude Code's exact-only design.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: exact only (Claude Code) | the exact scan alone | a match is the text the model wrote, byte for byte | a tab-indented needle against a space-indented file fails; a non-breaking space at a line edge fails; each failure costs a turn | Reject |
| **A. Cursor's trimmed-line block match (recommended)** | the Pack, after a zero-match exact scan | tab versus space and U+00A0 at line edges both vanish under `trim()`; one-to-one lines mean no line is skipped | it replaces the file's leading whitespace with the model's; two regions can be ambiguous only after trimming | **Recommended** (high: conservative, and the alternatives failed) |
| B. Collapse internal runs and blank lines | the Pack | fewer retries on badly copied context | a silent rewrite of strings inside literals; a match no longer means what the model thinks | Reject |
| C. Aider's uniform indent repair | the Pack | the file's indentation wins | still changes the model's bytes; stricter than `trim()`; one shared indent offset for every non-blank line | Set aside |
| D. A graded ladder (Codex, Gemini CLI, OpenCode dev) | the Pack | handles curly quotes, dashes, Unicode folds | first hit wins, so Codex edits the first of two identical lines (probe); a fold tier changes bytes unless it keeps offsets | Reject for v1 |

Claude Code's exact-only design has open issues for both whitespace classes it cannot match (web, via the evidence report: Claude Code issues 26996 and 29870), and each retry costs a turn. The three alternatives set aside all change bytes the model did not name. A thresholded tier is the clearest case. Copilot (0.95), Zed (0.8), Cline (0.66) and OpenCode (0.65) ship one with a stated threshold, and Gemini CLI's is a weighted distance of at most 0.1. Aider disabled its own after its maintainer wrote that fuzzy block matching "allows GPT to silently delete some important lines" (web, Aider issue 306, via the evidence report). Aider's uniform indent repair is the one tolerance worth revisiting, and only together with the changed-region echo of behavior 11.

### Match on an LF view and splice into the original, because it keeps mixed endings with no alignment algorithm

The four designs for line endings differ in which untouched bytes they can change (Table 4).

**Table 4.** How the Pack handles line endings and a byte-order mark. The do-nothing row is byte-exact matching.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: byte-exact (zeroclaw, Goose) | nobody | the simplest code | a multi-line LF needle never matches a CRLF file (probe), and the model sees every file as LF | Reject |
| A. An LF view with an offset map, spliced into the original (recommended) | the Pack | untouched bytes identical by construction, mixed files included; no alignment algorithm | an offset map to write and test; a rule for the ending of inserted lines | **Recommended** (medium: composed from three harnesses, none ships it) |
| B. Normalize, then restore by line alignment (Cursor's `At`) | the Pack | Cursor's tested behavior | an LCS alignment Runvil does not have; the lone-`\r` hole; a uniform-CRLF restore rewrites a deliberate bare LF | Reject |
| C. Homogenize to one ending (pi, Gemini CLI) | the Pack | one ending after the edit | untouched lines change: pi turns `a\nb\r\nc\r\n` into `a\nB\nc\n` on its exact path (probe) | Reject |

Option B of Table 4 is the most tested design, and it still loses. Cursor's alignment exists to undo a normalization that an offset map never performs. The evidence report also records that Cursor's write-side restore was read from the bundle and not re-run. Homogenizing (Option C) changes untouched lines, which fails the second criterion on the plain exact path. Byte-exact matching fails the first criterion on every CRLF file, because Read hides the `\r` of every CRLF line, so the model sees LF, and a multi-line needle then misses.

### Diagnose a stale edit on a miss and never gate on a read, because the VFS has no write_if

Two designs reached the same threshold independently. Zed and rig record what the model last saw and mention a change only when an edit fails to match (evidence report, Decision 4, citing Zed `edit_session.rs:971-976` and rig's probe S3). Runvil's `Access` has no compare-and-swap write and no hash parameter. What it has is a fresh `read_string` on every call, which already supplies the current bytes to compare against a recorded hash (Table 5).

**Table 5.** How the tool treats a file that changed since the model read it. The do-nothing row is Cursor's behavior.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: no check (Cursor) | nobody | no state, no false refusals after a restart or a compaction | a change outside the matched region is overwritten without a word when the needle still matches | Reject as the whole answer |
| B. A hard read-before-edit gate (Claude Code) | the Pack, with a per-run store | catches a formatter rewrite between read and edit | a store invalidated on every write; an error string to learn; refuses a legitimate edit of a file the prompt named | Reject |
| **C. Diagnose on a miss, note a stale success (recommended)** | the Pack, with a per-run hash record | one sentence on a failed match; no turn lost on a harmless stale read | one per-run service entry for each path, written by Read, replaced by Write and updated by a successful edit | **Recommended** (medium: two harnesses converge) |
| D. Compare-and-swap with re-apply (rig) | the VFS needs a `write_if` | closes the window between read and write | `Access` has no such method; rig's own probe shows the window narrowed, not closed (probes S7, S8) | Defer to `write-real` |

The hard gate loses because it refuses legitimate edits of files the prompt named, and it needs state that fails when a run restarts or a context compacts. Compare-and-swap loses for now only because the VFS cannot do it. Inside one call it adds little, because `apply` is pure CPU between the read and the write. The stale case that matters spans turns, and only a record of what the model saw can detect it.

### Take an array of edits as an optional second form, because sibling calls on one path conflict

The harnesses split four ways on how one call carries edits (Table 6). Runvil adds a reason that Cursor lacks. Sibling calls on one path conflict, so an array is the only way to edit one file several times in one round.

**Table 6.** How one call carries edits. The do-nothing row is Cursor's one-edit surface. Each non-recommended row names the failure it causes.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: one edit per call (Cursor, Claude Code, Gemini CLI, OpenCode, Cline `editor`, Goose, zeroclaw) | the model, one call per edit | the surface models know best | in Runvil, N edits to one file in one round hit `Conflict` on all but the first, so they cost N rounds | Keep as the base form |
| A. Array against the original, all or nothing (pi, Copilot, rig before 2026-10-19) | the Pack | edits are independent of each other | `{a to b}` then `{b to c}` fails; needs an overlap rule | Set aside |
| **B. Array in order on the running text, all or nothing (rig since 2026-10-19) (recommended, low confidence)** | the Pack | the same as N single edits in one claim and one write; no overlap rule | a later needle can be changed or duplicated by an earlier edit; rig names the cause in its refusal | **Recommended as an optional second form** (low: rig's reason is not recorded) |
| C. Array in order, not atomic (Zed) | the Pack | partial progress | a half-applied file; safe only because Zed's error shows the diff | Reject |

Array-against-the-original loses to in-order on the evidence report's argument, and the argument is thin. In-order is the composition of the single-edit tool a model already knows, so it needs no new matching rule and no overlap rule. Matching against the original suits independent edits and fails on `{a to b}` then `{b to c}`. Zed's non-atomic design loses because a half-applied file is safe only when the result shows the diff, as Zed's does. A patch grammar is the fifth shape (Codex, Cline's `apply_patch`, Aider's blocks). It needs a parser and a second matcher, so Runvil sets it aside; the Pack system can offer one later as its own Pack.

### Add one note to the success sentence, and only when the fallback ran

Cursor's success sentence says nothing about how the match was made (Table 7).

**Table 7.** What the success text contains. The do-nothing row is Cursor's one sentence.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: Cursor's sentence | nobody | byte-identical behavior | the model cannot tell that the fallback ran or that its replacement went in at its own indentation | Reject as the whole answer |
| **A. The sentence plus one note when the fallback ran (recommended)** | the Pack | the model learns about the one case where the match differed from what it wrote | a small departure from Cursor's text; the sentence stays as the prefix | **Recommended** (medium: no harness ships it) |
| B. The sentence plus a changed-region excerpt (rig) | the Pack | the model sees the result in file coordinates, up to 89 file lines in rig | tokens; no transcript in the sources shows a model skipping a Read because of it | Later |

The excerpt loses only for the first release. Rig's reply went through three rewrites in four days (unified diff, line numbers only, lines with context), which shows the size is a tunable, and the engine returns the changed range, so the excerpt can be added without touching the matcher.

### Offer one tool to every model, because the Pack system can reshape the set later

Cursor offers `ApplyPatch` instead of both `StrReplace` and `Write` to GPT-5-family models (bundle, daemon; `file-tools-index.md`). OpenCode and Cline make the same split. Offering one tool to every model costs nothing today and has one test surface. Reproducing the gate means building `ApplyPatch`, which is a second parser and matcher. The evidence report gives the gate's cause as model tuning rather than correctness, and no measurement in the sources shows a GPT-5 model doing worse with `StrReplace`. If one appears, `Pack::render` can drop `StrReplace` and `Write` together and add another Pack's tool for that model, as Cursor does, so the tool itself never checks a vendor (source: `runvil-plugin/src/pack.rs`). Dropping only one leaves a model that can rewrite whole files and cannot edit them, because `Write` is gated on a whole read (`master-write.md`, "Refuse to overwrite an existing file the run has not read whole and current"). Confidence in offering one tool to every model is medium, because no measurement favors `ApplyPatch` and none favors `StrReplace` either.

## Behaviors to implement

Behaviors 1 to 6 protect the right edit and the user's bytes. Behaviors 7 to 9 handle sibling calls, batches and stale files. Behaviors 10 to 14 cover refusal text, success text, input rules, whole-line deletion and cost bounds.

### 1. Own the matching in the Pack as a pure function over text

**Recommendation.** Write `apply(text: &str, edits: &[Edit]) -> Result<Applied, EditError>` in `plugin-std` with no `Access` and no I/O. The tool first calls `Access::check_write(path)`, so a Plan or Ask refusal comes before `stat`, before the read and before any match result. It then reads with `Access::read_string`, calls `apply`, writes with `Access::write`, and turns the error into text. Keep `Access::str_replace` for Lua scripts and stop calling it from this tool.

**Why.** One module then holds every rule, one test file proves them, and a dry run or a benchmark can call `apply` with no filesystem. That serves the right edit, because the overlap guard and the fallback have a place to live, and replay, because a pure function has no hidden state. It fits the VFS because it uses only `read_string` and `write`, which `Access` already gates, claims and fires, plus `check_write`, which the first commit of `write-real` adds. `check_write` asks the mode policy and takes no claim, fires no event and calls no backend (`master-write.md`, "Write through one `Access::write` and let the VFS own atomicity and parent creation"). Only `Access::write` meets the mode policy today, because `stat`, `read` and `read_string` gate `Op::Stat` and `Op::Read`, which the policy lets through in every mode. Without `check_write` a model that edits code in Plan mode gets "not found" or "ambiguous" for a file it can never edit. `Applied` should carry the new text, the matched ranges, whether the fallback ran, and the count, so the tool can write its success text without a second pass. Taking a slice of edits from the first commit makes behavior 8 a matter of argument parsing.

**Evidence.** Cursor splits `Sbe` (read, gate, write) from `Bbe` (the pure engine) (evidence report, Decision 1; daemon). Rig ships the same split: `rt::edit::apply(path, text, edits)` is public and pure, `edit()` is the read-apply-write wrapper, and a dry-run plugin and a benchmark call `apply` alone (source, rig `a83b774d0`, `rt/edit.rs:175-214,398-400`). In Runvil the default backend `str_replace` counts with `text.matches(old).count()` and replaces with `replacen(old, new, 1)`, no in-tree backend overrides it, and Lua's `store.str_replace` is a second caller (source: `traits/backend.rs:215-244`, `lua/src/engine_globals.rs:338`, `router-access.rs:207`).

**Without it.** A tool that delegates to `Access::str_replace` edits the first two bytes of `aaa` when the model asks to replace `aa` once, cannot offer `replace_all` or the fallback, and cannot report a count. A tool that splits the rules between the Pack and the VFS puts the next change in two places and answers one condition with two different error texts.

**Test.** Unit tests on `apply` alone, with no filesystem: the overlap, identity and bytes-outside vectors given under behaviors 2, 3 and 12. One integration test through `TestCall` over a backend that records its calls, asserting that a successful edit is one read and one write and never a `str_replace`. A second case under `Mode::Plan`: an edit of a `.rs` file whose needle is absent answers the Plan refusal, not the not-found sentence, and the backend records no read.

**Confidence.** High, because the evidence report reaches it, Cursor and rig both ship the split, and Runvil's own default shows the defect the split avoids.

### 2. Scan for the exact text first, count by the needle's length, and ship the overlap guard with it

**Recommendation.** Find matches with a left-to-right scan that advances by the needle's length in bytes. When `replace_all` is false and the scan found a match, search again from the next character boundary after the first match start. If that finds an occurrence, refuse with a sentence saying the matches overlap and asking for more context. When `replace_all` is true, replace every non-overlapping match with `new_string` as literal text and report the count.

**Why.** An advance-by-length scan undercounts a self-overlapping needle, and the guard is the control that compensates. Together they give one notion of a match, a count that equals the number of splices, and no silent edit of one of two overlapping sites, which serves the right edit. A splice by offset never expands `$` patterns in `new_string`. The two parts ship together or neither is correct.

**Evidence.** In Cursor the scan does `i += search.length`, so `aa` in `aaa` is one match, the guard refuses it with `replace_all` false, and `replace_all` true gives `Xa` and `1 occurrence in file <path> has been updated.` (evidence report, Layer 4 and its worked example; daemon, probe). Ten of the twelve other harnesses lack the guard, two of those (Aider, Codex) have no uniqueness rule, and only Zed and OpenCode refuse the case (source, probe). Zeroclaw counts with `str::matches`, passes `aa` in `aaa`, and edits the first two bytes, which is what Runvil's default backend does today (source: `traits/backend.rs:215-244`). OpenCode's `content.replaceAll(search, newString)` turns `X` into `$&$&` on `price: X and X` as `price: XX and XX`, while Cursor's engine gives `price: $&$& and $&$&` (probe). Rust's `str::replace` expands nothing, so the `$` trap opens only through a regex crate.

**Without it.** A needle `aba` in `ababa` edits offset 0 when the model meant offset 2, or both, and nothing tells the model a second site existed.

**Test.** On file `aaa`: `aa` to `X` with `replace_all` false gives the overlap refusal; with `replace_all` true gives `Xa` and `1 occurrence`. On `aaaa`: `aa` to `X` with `replace_all` true gives `XX` and `2 occurrences`. On `ababa`: `aba` to `X` with `replace_all` false gives the overlap refusal, and with `replace_all` true gives `Xba`. On `price: X and X`: `X` to `$&$&` with `replace_all` true gives `price: $&$& and $&$&`.

**Confidence.** High, because the guard is the compensating control for the scan, and ten of twelve harnesses omit it, so the mistake is easy to repeat.

### 3. Splice by offset into the original text so no untouched byte changes

**Recommendation.** Compute each match as a byte range in the original text and build the output as the original with those ranges replaced. Never build the file from trimmed or normalized lines. The final newline is a byte like any other: the tool never adds one and never drops one, and a needle that includes it must match it.

**Why.** This is the bytes-survive criterion in one rule. Read drops the empty element after a final newline (`master-read.md`, "Report the same line total whether or not the file ends in a newline"), so the model cannot see whether the file ends in one, and the splice must keep whatever the file has. A matcher that trims lines to compare and joins them to write re-indents every untouched line, so a one-line edit to a 3,000-line file rewrites 3,000 lines. Offsets into the original make that impossible by construction, and the test for it is one assertion.

**Evidence.** Cursor's `StrReplace` splices `slice(0, a) + replacement + slice(b)` over the original string (daemon). Its `pi_edit` writes back a whole NFKC-normalized copy on the fuzzy path, and a probe on a file with a trailing-space line, a U+00A0 and a curly quote changed three lines when one was intended (daemon, probe). Pi shipped the same defect until `128330e36` (2026-06-19, issue 5899, pi 0.79.9) and fixed it with regression tests. Pi at HEAD still normalizes the whole touched line, so `x = 'a'; y = 'b';` loses the curly quotes outside the match (source, probe). Bytes outside the match survive in zeroclaw, Goose, OpenCode, Cline `editor` and rig's exact path, and Aider and Codex rewrite the whole file and add a missing final newline (evidence report, Q10).

**Without it.** Every untouched line with trailing whitespace loses it, the success text reports one change, and a `git diff` shows the whole file.

**Test.** File `"\tconst a = 1;  \n    const b = 2;\t\nconst c = 3;\n"`, `old_string` `"\tconst b = 2;"`, `new_string` `"    const b = 9;\t"`. Assert the result is `"\tconst a = 1;  \n    const b = 9;\t\nconst c = 3;\n"`, and assert the byte length of the untouched prefix and suffix separately. The needle must need the fallback, because `b = 2` alone is an exact substring. Files `"a\nb\n"` and `"a\nb"`, edit `b` to `B`: results `"a\nB\n"` and `"a\nB"`. The same two files, edit `b` to `"b\nc"`: results `"a\nb\nc\n"` and `"a\nb\nc"`. The needle `"b\n"` against `"a\nb"` is refused as not found.

**Confidence.** High, because the evidence report calls the rebuild the worst failure available in this tool, and every harness that avoids it replaces only the matched span.

### 4. Fall back to a one-to-one line-trimmed match only when the exact scan finds nothing

**Recommendation.** When the exact scan found zero matches, trim every line of the needle and of the file, and look for a window with the same number of lines in which each line matches after trimming. The first needle line may match as a suffix of the trimmed file line and the last as a prefix. A one-line needle must equal the trimmed file line or be a prefix of it. Insert `new_string` verbatim over the matched region, which starts at column 0 of the first line when that line matched whole and ends at the end of the needle in the last line. Add no other tolerance: no collapsing of inner whitespace, no blank-line tolerance, no quote folding, no similarity score. One deliberate divergence: a needle that ends in a newline ends its match at column 0 of the next line and consumes nothing from it, even when that line holds only whitespace.

**Why.** The fallback fixes a tab-indented needle against a space-indented file and a U+00A0 at a line edge, because both vanish under `trim()`. That serves the right edit. Strict one-to-one lines mean the fallback can never skip or delete a line the model did not name, which serves the bytes-survive criterion. Each extra tolerance would change bytes the model did not name, so a match would stop meaning what the model thinks. The cost is that a whole-line match replaces the file's leading whitespace with the model's, so a replacement at the wrong depth lands at the wrong depth. Behavior 11 tells the model when that can have happened.

**Evidence.** The rules and the eight Trap 7 vectors come from running Cursor's own matcher (evidence report, Layer 5 and Trap 7; daemon, probe). Five of the twelve other harnesses ship a live thresholded tier (Copilot 0.95, Zed 0.8, Cline 0.66, OpenCode 0.65, Gemini CLI at a weighted distance of 0.1). Zed's accepts a window at 80 percent alignment and then deletes the skipped buffer line, which Zed's own test pins (source, probe). Cline's deletes hunk lines by position (probe). OpenCode's chain edits the first of two whitespace-different candidates without an error (probe). Aider's fuzzy tier sits behind a bare `return`. The divergence has a cause in Cursor's code. An empty last needle line equals a whitespace-only file line after trimming, so the matcher takes that whole line. As a result `"foo\nbar  \n   "` edited at `bar\n` to `BAR\n` returns `"foo\nBAR\n"` and loses the final `   ` (evidence report, Trap 12; daemon, probe).

**Without it.** A tab-indented needle against a space-indented file fails, and the model retries, spending a turn per attempt. The opposite error is a fallback that re-indents `new_string` to match the file. Zed, Aider and Gemini CLI do that on purpose, and it changes the model's bytes, which Cursor's does not.

**Test.** The tab-versus-space example: a file `function foo() {\n    if (x) {\n        return 1;\n    }\n}` with a needle of three tab-indented lines and a tab-indented `new_string`, which must change only those three lines. Then a two-line needle that omits a middle line must be refused. `"foo  bar"` against `"foo bar"` must be refused. `"  const a = 1; // note\n"` with the needle `"\tconst a = 1;"` and `new_string "X"` must succeed through the fallback as a prefix match and leave `X // note\n`. `"  foo bar\n"` with the needle `"\tbar"` must be refused, because a suffix match is not allowed. File `"foo\nbar  \n   "` edited at `"bar\n"` to `"BAR\n"` must give `"foo\nBAR\n   "`, which Cursor does not.

**Confidence.** High, because it is the conservative option that still fixes two failure classes, and the thresholded alternatives each failed in a way that reports success. The one divergence from Cursor is a corner case and is pinned by its own test.

### 5. Refuse a file that is not valid UTF-8 instead of decoding it lossily

**Recommendation.** Read with `Access::read_string`. On `VfsError::NotUtf8`, refuse with the sentence `File is not valid UTF-8 text: {path}. The first bad byte is at offset {valid_up_to}. This tool edits UTF-8 text only, even when a read showed the file decoded.` The shared `refusal` function builds it. Never replace bad bytes with U+FFFD and write them back.

**Why.** Cursor's read stage tries strict UTF-8, then a lossy decoder, then Latin-1, and the lossy path writes U+FFFD over the bad bytes. The evidence report calls that "a silent corruption path". Runvil's `read_string` already refuses, so this behavior costs one mapping and serves the bytes-survive criterion. Read does not refuse such a file for its encoding. It decodes UTF-16 by its mark and everything else lossily, and adds a note line (`master-read.md`, "Decode lossily, hide the byte order mark, and say when bytes were replaced"). So the model can read the text, a read record exists for it, and the stale check of behavior 9 can pass. This refusal is the only guard on the edit side, and its sentence is where the model learns why. The refusal also covers a UTF-16 file that has a byte-order mark, because those bytes are invalid UTF-8. Naming the offset lets the model decide whether to use another tool.

**Evidence.** Four harnesses refuse invalid UTF-8 (rig, zeroclaw, Goose, Codex), and five rewrite bad bytes (pi, Gemini CLI, OpenCode, Cline, Zed) (evidence report, Q3; source, probe). Cursor reports the lossy case as success: `The file <path> has been updated.` (evidence report, Failure matrix; daemon). `Access::read_string` returns `NotUtf8 { path, valid_up_to }` (source: `operations.rs:31-37`).

**Without it.** A mostly-UTF-8 file with a few bad bytes gets U+FFFD in their place, and the model is told the edit worked.

**Test.** File bytes `b"abc \xff def\n"` at path `/a.bin`, edit `abc` to `X`: the call must be refused with `File is not valid UTF-8 text: /a.bin. The first bad byte is at offset 4. This tool edits UTF-8 text only, even when a read showed the file decoded.` byte for byte, and the file bytes must be unchanged. A UTF-16 LE file with a byte-order mark must be refused the same way. Read each of those files first, so a read record exists, and the refusal must not change.

**Confidence.** High, because Runvil already refuses at the source, four harnesses do the same, and the lossy path is shown to corrupt.

### 6. Match on an LF view with the BOM removed and map offsets back to the original

**Recommendation.** Build an LF view of the text. CRLF and a lone CR each become one `\n`, and a leading U+FEFF is removed. Keep a map from view offsets to original offsets that holds one entry per CRLF, lone CR or BOM, and is empty for a plain LF file. Normalize `old_string` and `new_string` the same way. Match in the view, splice into the original through the map, and give each line break that `new_string` adds the ending of the line where the match starts (the nearest earlier break when that line has none, and LF when the file has no break). The BOM stays in the original bytes, because the splice never touches it.

**Why.** The model sees LF, so a multi-line LF needle must match a CRLF file, which serves the right edit. Read strips one `\r` from the end of every line, even in a file with mixed endings, and leaves a `\r` inside a line alone (`master-read.md`, "Strip one trailing carriage return from every line"). The model therefore cannot tell which lines are CRLF, and this tool matches on the view and picks the ending of an inserted break itself. Untouched bytes stay identical by construction, mixed-ending files included, with no alignment algorithm, which serves the bytes-survive criterion. The rule closes Cursor's lone-`\r` hole, where a classic-Mac file cannot be matched across a line boundary. Read shows such a file as one numbered line with the `\r` in its text. A needle copied from that page still matches, because the view reads each lone `\r` as a break, and a refusal's line numbers still match Read, because behavior 10 numbers lines with Read's splitter. The rule is deterministic, which serves replay. Runvil's `read_string` does not strip a BOM, so the Pack must remove it from the view itself. Read agrees: it drops a UTF-8 mark from what it shows (`master-read.md`, "Decode lossily, hide the byte order mark, and say when bytes were replaced"), so the model never sees the mark and never needs it in a needle.

**Evidence.** Cursor normalizes in three places: model strings with `\r\n?`, file content with `\r\n`, and a per-line restore on write by line alignment. A lone-`\r` file with `old_string "a\nb"` is refused as not found (evidence report, Q4 and Trap 10; bundle, daemon, probe; the write-side restore was read and never run). Pi homogenizes: `a\nb\r\nc\r\n` with an edit to `b` returns `a\nB\nc\n`, losing two carriage returns on untouched lines (source, probe). Rig converts only an all-CRLF file, and the evidence report concludes that the fix is per line: match on LF-normalized text and give each written line its own original ending (source, probe). Cline normalizes the model's text to the file's ending before counting and edits the matched span only, and Codex keeps each untouched line's ending (source). No harness ships an offset map, so the design is composed from these parts, which is why its confidence is medium.

**Without it.** A byte-exact matcher never matches a multi-line LF needle against a CRLF file (zeroclaw and Goose, by probe). A homogenizing matcher rewrites untouched line endings. A BOM at the start of the file makes the first line unmatchable or is lost on write.

**Test.** File `"a\r\nb\r\nc\r\n"`, edit `b` to `B`: result `"a\r\nB\r\nc\r\n"`. File `"a\r\nb\nc\r\nd\n"`, edit `b` to `B`: result `"a\r\nB\nc\r\nd\n"`. File `"a\r\n"`, edit `a` to `"a\nb"`: result `"a\r\nb\r\n"`, because the inserted break takes the ending of the line where the match starts. File `"a\rb\rc\r"`, `old_string "a\nb"`, `new_string "X"`: result `"X\rc\r"`, and pin it. A file `{BOM}abc\n` with `old_string "abc"` must succeed and keep the BOM.

**Confidence.** Medium, because the offset map is my composition of three harnesses' pieces and no harness ships it, while Cursor's alignment design is tested but needs machinery Runvil lacks.

### 7. Refuse, do not wait, when a sibling call holds the path

**Recommendation.** When `Access` answers `VfsError::Conflict` for the target path, the shared `refusal` function returns this sentence, with the path as the model wrote it: ``{path} is also used by another call in this round (a read, write, glob, grep, list or delete that covers it). Send one StrReplace per file per round, put the edits in one `edits` call, or send this edit in your next message.`` Any sibling that holds a claim over the file or its directory causes the conflict, in either order, and a finished sibling still conflicts. Do not wait for the sibling call.

**Why.** Cursor serializes same-path calls with a per-path mutex that it takes while the arguments stream, so the second call waits. Runvil cannot copy that. A Pack's `call` must not wait for another call of the same run, and calls a model asks for in one round are logically concurrent, so the VFS refuses the second one however the Harness orders them in time. The refusal text is the only thing between the model and a blind retry, which serves the right edit. The rule comes from the VFS, so it is the same on every replay.

**Evidence.** `ToolContext` says calls from one round "fork their identities from the chain together, so they are concurrent with each other: one call's write to a path conflicts with a sibling's read or write of it" (source: `runvil-plugin/src/context.rs`). A claim "holds for its scope's whole life", a call's own earlier claims never conflict with its later ones, and claims from another live scope always conflict (source: `handle/claims.rs`, `handle/scope.rs:156-159`). `Pack::call` "must not wait for another call of the same run to finish" (source: `runvil-plugin/src/pack.rs`). Cline lost an update when two parallel edits of one file both succeeded (probe), which is the outcome the refusal prevents. The `Conflict` text names both execution identities, so it means nothing to a model. A write claim is checked against the path's own reads and writes, its parent's children, every pattern that matches it and every subtree that covers it (source: `handle/claims.rs`), so a sibling Glob, Grep, list or recursive Delete conflicts with the edit as a sibling read or write does.

**Without it.** Parallel same-file edits return `conflicting claim: ...` with identifiers the model cannot act on. The doc comment on the conflict constructor says the executor maps a conflict to a fatal run error. The Engine does that only for a Lua store answer (`classify_vfs_failure` in `scheduler/apply.rs`). A model-issued tool call always resumes with content, and a tool's own failure becomes untrusted failure text (`scheduler/tool_call.rs`), so a conflict that the Pack maps to a `ToolError` does not end the run (read from source, not run).

**Test.** Send two parallel StrReplace calls on one path `/a.txt` in one round. The first must apply, the second must answer the conflict sentence above byte for byte, with `/a.txt` as the path, and the file must hold the first edit only. Then send the same two edits as one `edits` call and assert both apply. Run this test first, because it confirms that the conflict does not end the run.

**Confidence.** Medium, because the conflict rule and the Engine's treatment of a tool failure are read from source and not run, and the frequency of parallel same-file edits is not verified.

### 8. Accept an optional `edits` array, applied in order and all or nothing

**Recommendation.** Add an optional `edits` parameter beside Cursor's four: an array of `{old_string, new_string, replace_all}`. A call carries either the top-level pair or `edits`, and the tool refuses both together or neither, with an argument error built by the helper of behavior 12. Apply the entries in order, each against the text the earlier ones left. Write once under one claim, and write nothing if any entry fails. Name the failing entry's index. When an earlier entry removed the text a later one needed, say so.

**Why.** The single-edit form stays, so a model that knows Cursor's `StrReplace` needs no new instructions. The array lets the model make several edits to one file in one round, which behavior 7 otherwise forbids. In-order is the composition of N single-edit calls with one claim and one write, so it needs no overlap rule and no new matching rule. I hold the array shape at medium confidence and the application order at low, and the order is part of the behavior, so the behavior is low. The tool itself must check that exactly one form is present, because the argument checker honors only `type`, `properties`, `required`, `enum` and `items`.

**Evidence.** The harnesses split four ways (evidence report, Decision 10). Cursor's `StrReplace`, Claude Code, Gemini CLI, OpenCode, Cline `editor`, Goose and zeroclaw take one edit per call. Pi, Copilot's `multi_replace_string_in_file`, Cursor's `pi_edit` and rig before 2026-10-19 match an array against one snapshot of the original. Rig since 2026-10-19 applies in order. Zed applies in order and does not undo earlier edits when a later one fails.

The one harness that shipped both orders chose in-order, and I read its reversal directly. Rig commit `c2b6c392` (2026-10-19, PR 2857) has the subject "triobench round 4: edits apply in order, an edit of a file seen whole is one transport call, a stopped command says how it was stopped". Its body adds only "docs(rig-tools): reflow the edit and shell texts". The README line gains "applied in order", and the module comment changes from "all matched against the file as it was before the call" to "applied in order, each to the text the ones before it left". Neither gives a reason. The word `triobench` appears only in five `CHANGELOG.md` files, so the benchmark data is not in the repository. The one thing the repository does say is what the new order permits, in the doc comment of the test `edits_apply_in_order_even_where_they_overlap`: "two that touch the same lines, or a `replace_all` and an edit of one of its matches, both apply" (source, rig `a83b774d0`). Pi also repairs a single edit object and top-level `oldText` and `newText` sent to its array-only tool, which suggests that models send the single shape to an array tool (source: pi `edit.ts:103-134`).

**Without it.** N edits to one file cost N rounds, because behavior 7 refuses the rest of a parallel batch.

**Test.** File `alpha beta` with edits `alpha` to `gamma`, then `alpha beta` to `X`: the call must be refused naming `edits[0]` as the cause, and nothing may be written. File `A` with edits `A` to `B`, then `B` to `C`: the result is `C`. A call with `edits: []`, and a call with both forms, must each be refused. A call whose second entry fails must leave the file byte-identical. These pin the order, so a later switch to matching against the original changes one test and one code path.

**Confidence.** Low, because the reason for rig's reversal is not in its repository and no transcript or benchmark in the sources shows which order models need. A measured case of models sending dependent edits, or independent ones that an earlier edit breaks, would change it.

### 9. Diagnose a stale edit on a miss and never gate on a read

**Recommendation.** Keep a per-run record for each file the model last saw. The record lives in a run service whose key `plugin-std` declares, whose provider the Host supplies for each run, and which the Pack lists in `needs` (`master-read.md`, "Publish a read record to a per-run service, and keep the render pure"), so this tool never meets a missing service. Read writes it, keyed by the canonical path (from `Access::canonical_path`, `master-read.md`), and it holds a hash, a shown range, a total and a whole-file flag. This tool compares the hash only. The hash covers the raw file bytes, not the decoded text and not the LF view, so the tool hashes the bytes that `read_string` returned, BOM and `\r` included. A successful Write replaces the whole entry (`master-write.md`). A successful edit replaces only the hash and keeps the shown range, total and whole-file flag, as `master-read.md` specifies, so an edit never turns a partial view into a whole one. When no entry exists for the path, a successful edit creates none. When an edit fails to match or is ambiguous and the file's current hash differs from the record, add the sentence "The file has changed since you last read it." When an edit succeeds on a file whose hash differs, say in one clause that other changes landed after the last read. Never refuse an edit because no read happened.

**Why.** A stale file whose needle is gone looks like a model error, and the sentence names the real cause, which serves the right edit. A stale file whose needle is still unique is edited correctly, and refusing it would cost a turn, so the tool lets it through and says so. Each call already reads fresh bytes, so the record only needs to supply what the model last saw. The service is per run, so the record is keyed by path alone and two runs never share an entry. An edit that created an entry would let a later `Write` pass its read gate without any read, which is why an edit with no entry creates none. A compare-and-swap inside one call adds little, because `apply` is pure CPU between the read and the write. The stale case that matters spans turns.

**Evidence.** Zed records the disk modification time at each read and compares it only when an edit fails to match, and an edit with no prior read succeeds (source: Zed `edit_session.rs:971-976`, commit `37faef81f7`). Rig keeps a SHA-256 of what Read showed and says "changed since it was last read" only on a miss (source, probe S3). Claude Code lets an edit proceed when `old_string` still applies and appends a note that the file holds other changes (bundle, via the evidence report). A successful edit on a changed file is silent in rig, Codex and Cursor. Cursor has no check at all, and its `Write` re-verifies a sha256 fingerprint that `StrReplace` never computes (evidence report, Q9; daemon).

**Without it.** A formatter that rewrites a file between the read and the edit produces a bare "not found", and the model re-guesses its needle. When the needle still matches, the model never learns that the file held changes it had not seen.

**Test.** Read a file, append a line from outside the run, then edit with a unique needle: the edit must succeed and the answer must carry the changed-file clause. Delete the needle text from outside and edit again: the answer must be a not-found plus the changed-file sentence. Edit a file with no prior read: it must succeed with no clause. Edit a file with no prior read, then `Write` it, and assert that the gate refuses the `Write`. Two runs against one `Std` must not share records. Read a CRLF file that starts with a BOM, then edit it with no outside change: the answer must hold no changed-file clause, which pins that both sides hash the raw bytes.

**Confidence.** Medium, because two harnesses converge on the miss-only sentence, but no transcript shows the sentence changing a model's next move, and the record depends on Read, Write, this tool and the Host's provider being written to match.

### 10. Name the cause and the next move in every refusal

**Recommendation.** Keep Cursor's refusal sentences as the base, name the file in each one, and add what the tool already knows. The path in every message is the argument as the model wrote it. The sentences keep Cursor's tails:

- Not found: `The string to replace was not found in {path}.`, or `The string to replace was not found in {path} (even after relaxing whitespace).` when the fallback ran.
- Ambiguous: `The string to replace was found multiple times in {path}. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.`, or `The string to replace was found multiple times in {path} after relaxing whitespace. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` for a match that exists only after the fallback.
- Overlap: `The string to replace was found at more than one place in {path}, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.`
- Array call: prefix the sentence with `edits[{i}]: `.
- Missing file: `File not found: {path}`. Directory: `Path is a directory, not a file: {path}`. The shared `refusal` function builds these two, and every other `VfsError` sentence. For a mode refusal it passes the policy's reason through unchanged, and the policy states the next step (todo `vfs-mode-text`).
- Blank path: ``std_StrReplace: `path` is empty or only whitespace, and the call passed "{value}"``.
- Unknown parameter, with the shared helper: ``std_StrReplace: `old_str` is not a parameter of this tool. The parameters are `path`, `old_string`, `new_string`, `replace_all` and `edits`.``

On an ambiguous match, give the count and the line numbers of the first 8 matches, then `and N more`. On no match, add a numbered excerpt of the nearest region, found through Cursor's hint line (the longest non-empty trimmed line of the needle, then the first file line containing it), with ties broken by line order and the excerpt's line range labelled. Print each excerpt line in Read's format, a six-wide number and a pipe (`master-read.md`, "Number every line with a six-wide pipe prefix, and say so in the description"), and say in the label that the number and the pipe are not file text. Build the excerpt and the match line numbers from the original text with Read's splitter and formatter (split on `\n`, strip one `\r` per line, a lone `\r` is not a break), not from the LF view, so every number and every line is what Read printed. The hint line search runs over the same lines. When the same needle matches once after ignoring leading whitespace, say that the difference is indentation. No message names another tool. Excerpts appear only in refusals, which the Engine delivers as untrusted failure text, and the success text holds no file text.

**Why.** Every refused call costs a turn, and a refusal that says where and why lets the next call succeed, which serves the right edit. A batch of failed edits cannot be told apart without the file, which is the reason `master-read.md` gives for naming the path in every failure ("Name the path and the next step in every failure"). The `edits` array makes it worse, because a refusal then names the entry and still not the file. The excerpt window is bounded (2 lines before, the needle's line count plus 4 in all), and the line list is capped at 8, which serves bounded cost. Cursor breaks ties among equal-length lines by sort stability, so a different engine could pick a different hint line. Breaking ties by line order removes that, which serves replay. The excerpt has no clip marker in Cursor, so label its range.

**Evidence.** Cursor's engine sentences name no file, and its missing-file answer is the bare `File not found` (evidence report, Failure matrix; daemon). Six harnesses improve the failure text in this direction. Cursor shows the hint under ` Found a possible fuzzy match, did you mean:` (daemon). Zed lists line numbers, at most two for an exact match (source). Rig lists 8 line numbers then `and {N} more`, and shows the closest text (source, probe). Goose names two ambiguous sites with line numbers and shows a 20-line preview on a miss (source). Aider echoes the model's block with the nearest real lines (probe). Zeroclaw re-matches under a weaker rule and names the class: `The difference is indentation (width, or tabs vs spaces)` (probe on the shipped helper). Two hint designs fail in ways to avoid. Goose's matching test `first_line.contains(line.trim())` is true for every blank line, so its hint can point at an unrelated blank-line neighbourhood. A hint built with `lines()` hides a `\r`, and a hint for an NFC-against-NFD mismatch prints text that looks identical to the needle (rig, Goose, probe).

**Without it.** A bare "not found" sends the model to retry blindly. A hint built like Goose's sends it to the wrong region. An indentation difference in a CRLF file reads as a text error.

**Test.** File `line one\n\nline three\nline four\n` with needle `zzz not present`: the answer must show no excerpt, or one that shares a token with the needle. A CRLF file with a multi-line LF needle that misses by one word: the excerpt must not print text that is byte-identical to the needle. An ambiguous needle with 10 matches: the answer must list 8 line numbers and say `and 2 more`. Two equal-length candidate lines: the earlier one must be the hint line. On any miss with an excerpt, assert that each excerpt line starts with a number right-aligned in six columns and a pipe, with no tab as the separator, and that the label says the prefix is not file text. File `a\rNEEDLE\rb` with a needle that misses by one word: the excerpt is one line numbered 1, byte-identical to Read's output for that file. File `a\r\r\nb`: two lines, numbered 1 and 2, as Read shows them. Assert that every refusal contains the path argument verbatim, so a missing `notes/a.md` answers `File not found: notes/a.md`, and that no message names another tool. A call with `path: "   "` answers ``std_StrReplace: `path` is empty or only whitespace, and the call passed "   "``. A valid call with the extra key `old_str` answers the unknown-parameter sentence above, byte for byte.

**Confidence.** High, because six harnesses converge on naming the cause, and the two failing hint designs are reproduced by probe. The cap of 8 is rig's number and is a tunable.

### 11. Say what happened on success: Cursor's sentence, plus a note when the fallback ran

**Recommendation.** Return Cursor's sentence as the prefix: `The file <path> has been updated.`, or with `replace_all` true `1 occurrence in file <path> has been updated.` and `N occurrences in file <path> have been updated.` For an array call return `The file <path> has been updated with N edits.`, and `with 1 edit.` for one entry, through the shared `plural` helper. When the fallback matched, append one sentence. It says that the match ignored leading and trailing whitespace on each line, and that `new_string` went in as written, without re-indenting. When behavior 9 found the file changed since the last read, append its one clause.

**Why.** The fallback case is the one where the match differed from the model's text and the model's indentation went in verbatim. Cursor records that in a telemetry counter, `agent.tools.str_replace.whitespace_insensitive`, and tells the model nothing, so a model can write a tab-indented replacement into a space-indented file and be told it worked. One sentence uses information the tool already has, adds a bounded number of tokens, and is the same on every replay. It serves the right edit and bounded cost and keeps the familiar surface, because Cursor's sentence stays the prefix. Naming the path as the model gave it matches the plan's `Wrote contents to <path>` for Write.

**Evidence.** Cursor's success text is one flat sentence with the raw path (evidence report, Result cases and Q12; daemon). Gemini CLI labels only its fuzzy tier (`Applied fuzzy match at line(s) S-E.`), and its flexible and regex tiers look the same as an exact success (probe). OpenCode never says a match was fuzzy. Rig returns the changed lines with context (source, `rt/edit.rs:218-284`). No harness labels the trimmed-line tier on success, so the note is my judgment and not a copied behavior.

**Without it.** The model cannot learn that its replacement kept its own indentation, so a wrong depth stays in the file until a later Read.

**Test.** A tab needle against a space-indented file must produce the sentence plus the note. An exact match must produce Cursor's sentence byte for byte. `replace_all` true with three matches must produce `3 occurrences in file <path> have been updated.` `aa` in `aaa` with `replace_all` true must produce `1 occurrence in file <path> has been updated.`, which pins the count against a later change to two. An `edits` call of two entries must produce `The file <path> has been updated with 2 edits.`, and of one entry `The file <path> has been updated with 1 edit.`

**Confidence.** Medium, because no harness ships this note and no transcript shows its effect, but it is one sentence and uses facts the tool holds.

### 12. Reject an empty needle and an identical pair after line-ending normalization, and allow a de-indent

**Recommendation.** Refuse an empty `old_string` before any search, with ``std_StrReplace: `old_string` is empty. Pass the text to replace.`` Normalize both strings' line endings to LF (CRLF and a lone CR each become one `\n`) and refuse the call when they are identical, with ``std_StrReplace: `old_string` and `new_string` are exactly the same.`` Compare nothing else, and in particular do not trim. A call that passes both checks then runs the pipeline. One helper builds every argument error of this tool in the shape ``std_StrReplace: `{name}` {problem}``, and it is the helper of the other five tools too.

**Why.** The no-op refusal stops a model from probing whether a line exists by sending it back unchanged, which serves the right edit. The narrow comparison keeps a deliberate de-indent working. Comparing raw strings lets `a\r\nb` to `a\nb` through as a reported no-op, and comparing trimmed strings blocks a de-indent.

**Evidence.** The evidence report states the rule as: an `old_string` and `new_string` that are identical after line-ending normalization are rejected, and trimming is not part of normalization. It verified against the shipped matcher that `"  foo"` to `"foo"` on a file with `foo` at column 0 succeeds and leaves `foo\n`, that `"  foo"` to `"    foo"` gives `"    foo\n"`, and that `"foo"` to `"foo"` and `"a\r\nb"` to `"a\nb"` are refused (daemon, probe). Cursor tests emptiness on the raw string before normalizing. Runvil's `Access::str_replace` refuses an empty anchor before it gates (source: `operations.rs:74-91`). Zeroclaw and Goose have no identity check and rewrite the file on success.

**Without it.** A raw comparison reports a no-op as an edit. A trimmed comparison makes a model that wants to fix indentation fail with an error that says nothing about why.

**Test.** An empty `old_string` must answer ``std_StrReplace: `old_string` is empty. Pass the text to replace.`` byte for byte. `"foo"` to `"foo"` must be refused with ``std_StrReplace: `old_string` and `new_string` are exactly the same.`` `"a\r\nb"` to `"a\nb"` must be refused. `"  foo"` to `"foo"` on `foo\n` must succeed and leave `foo\n`. `"  foo"` to `"    foo"` must succeed and give `"    foo\n"`. An empty file with a non-empty needle must answer not found with no whitespace-relaxation wording. A needle equal to the whole file with an empty `new_string` must leave a 0-byte file.

**Confidence.** High, because the rule was verified against Cursor's own matcher, and the parent spec's contrary example was found and corrected.

### 13. Remove the whole line when an empty replacement leaves nothing on it

**Recommendation.** When `new_string` is empty, widen each match to its whole line if only whitespace shares the line with it, and consume the line's newline. When the match is on the last line and no newline follows, consume the line break before it, whether it is LF, CRLF or CR. A match that shares its line with other text removes only the match. Compute the widening in the LF view against the original text for every match, then splice all ranges in one pass.

**Why.** A deletion that leaves a blank line costs a second edit, because the model must send another call to remove it, which serves the right edit. Computing every widening against the original text gives one splice path for one match or many. The evidence report finds that a single unified path is equivalent to Cursor's two paths for the inputs Cursor can produce, since matches are processed in descending order and earlier splices cannot affect a lower match's lookahead.

**Evidence.** Cursor's splice has this three-case clause for an empty replacement. The evidence report verified these vectors (Layer 6 and Trap 6; daemon, probe). Deleting `line2` from `line1\nline2\nline3\nline4` gives `line1\nline3\nline4`, and deleting `line2\n` gives the same. Deleting `line3` from `line1\nline2\nline3` gives `line1\nline2`. Deleting both lines of `line1\nline2` gives an empty file. Deleting `a` with `replace_all` from `a\nb\na\nb\n` gives `b\nb\n`. Pi and rig, which were probed, leave a blank line unless `old_text` took the newline (evidence report, Q6).

**Without it.** Deleting a line leaves a hole, the model issues a second edit, and the turn costs twice as much.

**Test.** `line1\nline2\nline3\nline4` with `old_string "line2"` and `new_string ""` must give `line1\nline3\nline4`. The same file with `old_string "line4"` must give `line1\nline2\nline3`, with no trailing newline and no blank line. `abc\n` with `old_string "b"` and `new_string ""` must give `ac\n`. `a\nb\na\nb\n` with `old_string "a"`, `replace_all` true and an empty `new_string` must give `b\nb\n`.

**Confidence.** High, because the vectors are verified against Cursor's engine, and no harness in the sources shows harm from the rule.

### 14. Refuse files over 8 MiB and bound the match bookkeeping

**Recommendation.** Call `Access::stat` before the read, and refuse a file larger than 8 MiB (8,388,608 bytes) with `File is too large to edit: {path} is {size} bytes, over the limit of 8388608 bytes.` Count every match, but keep only the first 8 offsets when `replace_all` is false. When `replace_all` is true, splice while scanning or store byte offsets only, and do not build an object per match.

**Why.** The tool holds the original, the LF view, a map with one entry per CRLF, lone CR or BOM, and the output, so memory is a small multiple of the file size, and a cap bounds it, which serves the bounded-cost criterion. Cursor refuses a file whose read was truncated, because writing the edit would write back a prefix. The evidence report calls that the best safety idea in Cursor's edit path. Runvil's `read_string` reads the whole file and never truncates, so the size check replaces it. A full count of a pattern in 8 MiB is linear and fast, and it gives the ambiguity message its count. The slow cases in other harnesses come from per-match objects and per-match line lookups. The fallback costs lines times needle lines, and it runs only after a zero-match exact scan, so the cap also bounds it.

**Evidence.** Cursor's cut is `MAX_TEXT_SIZE = 8 * 1024 * 1024` as a JavaScript string length (UTF-16 code units), and its refusal text begins `File is too large to edit` (bundle, daemon). Rig's cap is 16 MiB in bytes, Claude Code's is 1 GiB, Zed's load ceiling is 6 GiB, and Goose and zeroclaw have none. Cursor built 1,048,576 match objects for a 1 MiB file of `e` in 139 ms. Pi failed on 8 MiB of `e` after 126 ms. Rig's ambiguous path took 110 s on 512 KiB of `a` lines because it looks up the line of every match (probe). `Stat` carries `size` in bytes (source: `stat.rs`). Because Rust counts bytes and Cursor counts UTF-16 code units, the same 8 MiB figure cuts a non-ASCII file at a different place.

**Without it.** A hostile 8 MiB file of one repeated character with a one-character needle allocates a record per character, and a large file held several times over exhausts memory in a run that has no other limit.

**Test.** A file of 8,388,609 bytes at `/big.txt` must be refused with `File is too large to edit: /big.txt is 8388609 bytes, over the limit of 8388608 bytes.` byte for byte. A 1 MiB file of `e` with `old_string "e"`, `new_string "x"` and `replace_all` true must complete and give 1 MiB of `x`. The same file with `replace_all` false must be refused as ambiguous, with 8 line numbers and `and N more`, in bounded time.

**Confidence.** Medium, because the 8 MiB figure is Cursor's in a different unit, and the cost figures come from other harnesses' probes. No Runvil measurement exists yet.

## Where Runvil should differ from Cursor

Runvil keeps Cursor's parameter names, sentences and matching order, and departs where Runvil's VFS, its Pack contract or the evidence calls for it (Table 8). The last two rows list machinery that exists for JavaScript, the protobuf layer or the Cursor workbench and has no counterpart to build.

**Table 8.** Where Runvil's StrReplace departs from Cursor's, with the reason for each departure.

| What Cursor does | What Runvil should do | Why |
|---|---|---|
| Keeps matching in a pure engine (`Bbe`) under a read-gate-write wrapper (`Sbe`) | Put `apply` in the Pack over `read_string` and `write`, and stop calling `Access::str_replace` | Same split as Cursor and rig; the VFS default has the overlap defect (behavior 1) |
| Offers `ApplyPatch` instead of both `StrReplace` and `Write` to GPT-5-family models | Offer one `StrReplace` to every model, and let `Pack::render` reshape the set, dropping `StrReplace` and `Write` together, if a measurement later asks for it | `ApplyPatch` is a second parser and matcher, and no measurement shows `StrReplace` doing worse |
| Ships a `search_replace` and `file_path` branch for `dsv3-1018`, with the `occurences` typo and an `isComposer15` line | Ship one surface and no branch | Only parameter names differ, a per-model table can be added later, and no typo needs preserving |
| Takes a per-path mutex while the arguments stream, keyed on the raw path string, and makes the second call wait | Refuse on `VfsError::Conflict` with a sentence, and never wait; the VFS canonicalizes the path | A Pack's `call` must not wait on a sibling call (behavior 7) |
| Decodes strict UTF-8, then lossily, then as Latin-1, and writes U+FFFD back | Refuse a file that is not UTF-8, naming the byte offset | The lossy path is a silent corruption path (behavior 5) |
| Collapses `\r\n` in the file and `\r\n?` in the model's strings, so a lone-`\r` file cannot match across a line | Use one LF view for both sides, so a lone CR is a line break | One rule closes the hole (behavior 6) |
| Restores line endings on write by line alignment, or with a global `(?<!\r)\n` rewrite | Splice into the original through an offset map | Untouched bytes stay identical without an alignment algorithm (behavior 6) |
| Strips a BOM on read and restores it on write | Remove it from the view, and leave it in the original bytes | The splice never touches it; Read hides it too |
| Refuses a read that was truncated with `File is too large to edit`, which a disk-full spill can also trigger | Check `Access::stat` size against 8 MiB and name the path and the size | The Cursor message misdiagnoses a disk problem and names no file, and Runvil never truncates (behavior 14) |
| Says only that the string was found multiple times, names no file, picks the hint line by sort stability, and labels no range | Name the path, give the count and 8 line numbers numbered as Read numbers them, tie-break by line order, and label the excerpt's lines | Replay and a faster retry (behavior 10) |
| Returns one flat sentence on success | Add a note when the fallback ran and a clause when the file changed | The model learns the one fact it lacks (behaviors 9 and 11) |
| Takes one edit per call, ships `MULTI_STR_REPLACE` as an id with no tool, and carries an array tool whose fuzzy path rewrites the whole file | Add an optional `edits` array, in order, all or nothing, with no folding tier | Sibling calls on one path conflict in Runvil (behavior 8) |
| Keeps no stale check on StrReplace while `Write` re-verifies a sha256 fingerprint | Keep a per-run hash record and diagnose on a miss | Cheap, and two harnesses converge on it (behavior 9) |
| Checks plan mode twice and builds the ask-mode sentence in the renderer | Call `Access::check_write` first so the refusal precedes any match result, let `Access::write` gate once more, and pass the policy's reason through the shared `refusal` function | The mode policy lives in the VFS, so a tool cannot forget it, and the model gets the refusal before a not-found for a file it cannot edit (behavior 1) |
| Repairs the path segment `worktries` to `worktrees`, streams live `new_string` deltas to the UI, caps the diff record at 524,288 characters, and counts telemetry | Drop all of it | It serves the Cursor workbench and its protobuf layer, and the model never sees it |
| Builds the description with a reference to the `WRITE` tool that throws when `WRITE` is absent, and counts offsets in UTF-16 code units | Name tools literally in the description, and count bytes | Both exist because of JavaScript; Rust strings are bytes |

Confidence for each row is the confidence of the behavior it names.

## Mapping to Runvil

`Access` already gates, claims and fires every call, so the Pack needs no policy code of its own. The gaps are all in text handling, so most of the work is pure code in `plugin-std` (Table 9). The crate keeps every Rust file at 500 lines or fewer, so the engine needs several sibling files named in the crate's kebab-case style (for example matching, fallback, splice and refusal text) with a `<stem>-tests.rs` beside each. Vinnie owns every row.

**Table 9.** How each behavior maps onto `Access`, whether the VFS covers it, what the gap needs, and the next step as a todo id from the standing plan.

| Behavior | `Access` method | Covered | What the gap needs | Next step |
|---|---|---|---|---|
| 1. Own the matching | `check_write`, `read_string`, `write` | partial | A new module with `apply` and no I/O; keep `Access::str_replace` for Lua; `Access::check_write(path)`, which runs the mode policy only, comes in the first commit of `write-real` | `replace-real`, after `write-real` |
| 2. Exact scan, overlap guard, `replace_all` | none | missing | Pure code in `apply` | `replace-real` |
| 3. Splice by offset | `write` | covered | Pure code; `write` is a whole-file write, and its atomicity belongs to `write-real` | `replace-real` |
| 4. One-to-one line-trimmed fallback | none | missing | Pure code, including the empty-last-line divergence | `replace-real` |
| 5. Refuse invalid UTF-8 | `read_string` (`NotUtf8`) | covered | Map the path and `valid_up_to` into a sentence in the shared `refusal` function | `replace-real` |
| 6. LF view, offset map, BOM | `read_string` (strips no BOM) | partial | Offset map in the Pack; `read-real` drops a UTF-8 mark from what Read shows, so both sides hide it | `replace-real`, `read-real` |
| 7. Refuse on a sibling conflict | any `Access` call (`VfsError::Conflict`) | covered | Map the error in the shared `refusal` function; confirm by test that a conflict returned through a Pack does not end the run | `refusal-text`, `replace-real` |
| 8. `edits` array, in order | none | missing | Parse `edits`, check that exactly one form is present, change the schema | `replace-real` |
| 9. Stale diagnosis | `read_string`, `stat` (size; `modified` is optional) | partial | Read writes the per-run record (`master-read.md`); this tool compares a hash of the raw bytes, and Write replaces the entry while a successful edit replaces the hash and creates none. `Access::canonical_path` comes with `read-real`, and the record's key, `needs` entry and Host provider come with `write-real` | `read-real`, `write-real`, `replace-real` |
| 10. Refusal text | `Anchor` (count only) | missing | Build the texts from `apply`'s results, not from `Anchor`, with the path in each; build every `VfsError` sentence in the shared `refusal` function and every argument error with the shared helper | `refusal-text`, `replace-real` |
| 11. Success text | none | missing | Build from `Applied` (count, fallback flag, changed-file flag), with the shared `plural` helper | `replace-real` |
| 12. Empty needle and identity checks | `str_replace` refuses an empty `old` | partial | Repeat the empty check in the Pack, since the tool no longer reaches `str_replace`; add the identity check; build both with the shared argument-error helper | `replace-real` |
| 13. Whole-line deletion | none | missing | Pure code in the splice | `replace-real` |
| 14. 8 MiB cap, bounded bookkeeping | `stat` (`size`) | covered | One `stat` call before the read | `replace-real` |
| Mode gates and VFS refusals | `check_write`, then `gate(Op::Read, path)` inside `read_string` and `gate(Op::Write, path)` inside `write` | partial | The mode policy refuses only the write, so `check_write` asks it before the match. Pass the policy's reason through the shared `refusal` function, and map `ReadOnly`, `NotFound` and `IsADirectory` there too. The policy's next-step text comes from `vfs-mode-text` | `refusal-text`, `write-real`, `vfs-mode-text` |
| Symlink policy | `read_string` and `write` follow a link that stays inside the root and refuse one that leaves it or dangles (`real/resolve.rs`, `contain`; `real/tests/links.rs`) | covered | none | none |

Tests to add to the crate, each beside the module it covers, with the integration cases in `tests/it/tools.rs`, where the `std/replace` entry flips from the stub text to real behavior:

- The overlap vectors on `aaa`, `aaaa` and `ababa`, with `replace_all` both ways, and the `$&$&` literal vector.
- The three identity vectors, the `a\r\nb` identity case, the empty-needle case, and the empty-file case.
- The final-newline vectors of behavior 3, the bytes-outside vector for the fallback, the tab-versus-space example, the middle-line-omitted refusal, and the eight single-line and trailing-newline vectors from the evidence report's Trap 7.
- The CRLF, mixed-ending, inserted-break, lone-CR and BOM vectors of behavior 6.
- The invalid-UTF-8 refusal with its path and byte offset, the UTF-16 refusal, and the same refusals after a Read has shown each file decoded.
- The deletion vectors of behavior 13, including `abc\n` deleting `b`.
- The ambiguity refusal with 8 line numbers and `and N more`, the not-found excerpt, the CRLF no-identical-text case, the blank-line hint case, the hint tie-break, and the two lone-`\r` excerpt cases that match Read's numbering.
- Every refusal names the path argument verbatim and no message names another tool, with the missing-file, directory, blank-path and unknown-parameter sentences byte for byte.
- The success sentences of behavior 11, including the self-overlap pin, the fallback note and `1 edit` against `2 edits`.
- Two parallel calls on one path in one round, with the conflict sentence byte for byte, and the same pair as one `edits` call.
- An edit of a `.rs` file with an absent needle under `Mode::Plan`, which must answer the Plan refusal.
- The array order pair (`alpha beta`, and `A` to `B` to `C`), the empty `edits` refusal, the both-forms refusal, and an unchanged file after a failing second entry.
- The stale-record cases of behavior 9, including two runs against one `Std` that must not share records, and an edit with no entry that creates none.
- The 8 MiB boundary and the 1 MiB `e` case with `replace_all` both ways.
- A test that a successful edit makes one read and one write and never a `str_replace`.

Schema and description fixes, each a next step under `replace-real`:

- Add an `edits` property to `schemas/replace.json`: an array of objects with `old_string`, `new_string` and optional `replace_all`. Reduce `required` to `path`, because the tool checks that exactly one form is present. Record both changes in `schemas/EDITS.md`, whose current note says the parameters and the `required` list are Cursor's.
- Rewrite the description to say what the tool does: the old string must be unique unless `replace_all` is true; the replacement is inserted as written and is not re-indented; edits to one file in one round belong in one `edits` call. Keep "Read the file first" as advice. Tell the model two facts about what Read shows. Read prefixes each line with a six-wide number and a pipe, as in `     1|text`, and that prefix is not file text, so `old_string` must leave it out. Read does not show whether the file ends in a newline, and an edit keeps the file's final newline as it is. Re-add Cursor's line "When editing text, ensure you preserve the exact indentation (tabs/spaces) as it appears before", which the current wording drops and which tells the model that the fallback is a safety net. Add two sentences from the shared rules. First: "A path that names a symbolic link is read and written through to the file it points to. A link that leaves the run's filesystem is refused." Second: "StrReplace edits any path the call names, including paths that Glob and Grep leave out."
- Keep the shipped `path` text, "The absolute path of the file to modify", because every file tool accepts a relative path, joins it onto `/`, and advertises the absolute form that Glob and Grep print.
- Keep the `new_string` description "(must be different from old_string)", because behavior 12 enforces it.
- Edit the standing plan: replace the `replace-real` wording "reusing `Access::str_replace` where its single-occurrence semantics match and adding the trimmed-line fallback it lacks" with "owning the matching in the Pack over `read_string` and `write`; `Access::str_replace` stays for Lua", and move the open question in the plan's Decision Record to a recorded decision for the Pack. The plan edits are proposals. The standing plan is not edited by this report.
- Record, in the plan's Decision Record, that `Access::str_replace` counts with `str::matches` and so passes a self-overlapping needle, and that Lua's `store.str_replace` still uses it. Fixing it is outside this crate and lands as its own VFS commit under `replace-real`.
- Order the todos as the shared rulings do: `refusal-text`, `read-real`, `write-real`, `replace-real`. `read-real` adds `Access::canonical_path`, and `write-real` adds `Access::check_write`, the record's key, its `needs` entry and the Host provider. `vfs-mode-text` is independent and can land any time.

The `edits` schema change, and the description sentence that mentions `edits`, carry behavior 8's low confidence. The other fixes and the plan edits are high confidence, because they only make the text match the behaviors. The new todo ids `refusal-text` and `vfs-mode-text` are proposals, confidence medium.

## Risks and open limits

### The order of edits in an array is not settled by the evidence

The recommendation for in-order application rests on one fact: the one harness that shipped both orders, rig, chose in-order four days after shipping the other, and pinned the choice with a test. The reason for that reversal is not in rig's repository. I checked the commit, its message, the README change and the doc-comment changes, and I searched the tree at `a83b774d0` for `triobench`. I found a title, a one-line body and a statement of what the new order permits. I found nothing about what went wrong with the old one. The benchmark data is not in the repository. That is why the array behavior is rated low. Matching against the original is easier to reason about for independent edits and fails on `{a to b}` then `{b to c}`. In-order handles chained edits and fails when an earlier edit removes the text a later needle matches. Nothing in the sources says which case models meet more often, so the engine takes a slice, the order is pinned by one test, and a switch changes one code path. Table 10 lists the other limits.

**Table 10.** What the evidence does not settle. Likelihood is the chance that the stated outcome happens. Confidence rates the evidence behind the row.

| Open limit | Likelihood | Confidence | What it rests on |
|---|---|---|---|
| In-order is the wrong order for Runvil's models | roughly even chance | low | rig's reversal has no recorded reason; no transcript or benchmark in the sources |
| A model sends two same-file edits in one round | likely | low | Cursor locks per path so that same-turn calls on one path cannot interleave (bundle, via the evidence report); no Runvil transcript counts them |
| A conflict returned through a Pack ends the run | very unlikely | medium | The tool-call arm turns a model-issued tool's failure into failure text, and only a Lua store answer is made fatal; read, not run |
| A Lua script hits the VFS default's overlap defect | unlikely | medium | The defect is read in source (`str::matches`); how often a script uses a self-overlapping needle is unknown |
| A wrong-depth replacement goes through the fallback despite the note, in a repository with mixed indentation | likely | low | Claude Code issues on tabs and U+00A0 (web); no count of how often models mis-indent |
| The offset map's inserted-break rule surprises a user in a mixed-ending file | roughly even chance | low | Composed from three harnesses' pieces; no harness ships it |
| A NUL-bearing file that is valid UTF-8 is edited as text | unlikely | medium | `read_string` refuses only invalid UTF-8. Read refuses such a file by a content sniff of the first 8,192 bytes (`master-read.md`, "Refuse binary content by sniffing the first 8,192 bytes"), so a model rarely sees one, but this tool has no sniff of its own. Cursor sniffs for NUL and Zed checks its first 1,024 bytes (the evidence report gives no sample size for Cursor) |
| A composed accent in the needle misses a decomposed one in the file, and the hint looks identical | unlikely | medium | No harness except pi (NFKC) and Cline's `apply_patch` normalizes, and rig and Goose print the identical-looking hint (probe) |
| The stale record stays empty because Read and Write do not write it yet | likely | high | The record needs both siblings, and no code writes it today. `read-real` ships without the record and without `needs`, and `write-real` adds the key, the `needs` entry, the Host provider and the `pack-tests.rs` change in one commit (`master-read.md`). The todo order puts `replace-real` after `write-real`, so a release that follows it has the record |
| A GPT-5-family model does worse with `StrReplace` than with `ApplyPatch` | roughly even chance | low | Cursor, OpenCode and Cline all split by model, and the evidence report gives the cause as model tuning; no measurement in the sources |
| A hostile file or needle makes the fallback slow | unlikely | medium | Lines times needle lines, bounded by the 8 MiB cap; no Runvil measurement |

Read refuses a file over 16 MiB (`master-read.md`, "Bound the read with a type check and a 16 MiB ceiling"), and this tool refuses a file over 8 MiB before it reads (behavior 14). A file over 16 MiB is therefore refused by both. A file between 8 MiB and 16 MiB can be read in pages and cannot be edited here.

Several claims come from the evidence report and were not re-run here. Cursor's engine was run from the daemon, and the write-side line-ending restore was read from the bundle and never run, which matters little because Runvil's design replaces it. The web claims (Claude Code issues 26996 and 29870, Aider issue 306) were not re-read. The vectors of behaviors 2, 3, 4, 6, 12 and 13 passed in a throwaway JavaScript prototype of the recommended pipeline (probe). The final-newline vectors added to behavior 3 follow from the rules and were not run. The prototype checks the rules, not the byte arithmetic, the Rust types or the performance, and the entry-index diagnosis of behavior 8 was not prototyped.

Five findings would change the recommendation. A transcript or benchmark showing which edit order models need reverses behavior 8's order. A backend that must edit in place, such as a database, reopens Option C of Table 2. A measured drop in a GPT-5-family model's edits reopens the vendor question through `Pack::render`. A test showing that a conflict returned through a Pack ends the run makes behavior 7 and the `edits` array a requirement for the first release. Evidence that the fallback note does not stop wrong-depth edits adds Aider's indent repair, together with the changed-region excerpt.

## Sources

Evidence reports, in this folder:

- [`tool-str-replace.md`](tool-str-replace.md): the primary ground. Cited by section name: `Decisions you have to make`, `The contract`, `Algorithm and ordering`, `Engineering questions`, `Failure matrix`, `Traps`, `How the harnesses do this`, `What Runvil can support today` and `What remains unknowable`.
- [`file-tools-index.md`](file-tools-index.md): the shared facts and the corrections to the parent spec.

Cursor 3.27.12 (all read through the evidence report unless a line says otherwise):

- `cursor-agent-exec/dist/main.js`, 9,814,287 bytes, the minified bundle (bundle). Offsets are in the evidence report's `Sources`.
- `cursor-agent-host/dist/agent-host-daemon/dist/bin/daemon.cjs`, 20,213,626 bytes, the unminified twin, with Cursor's engine run verbatim for the Trap vectors (daemon, probe).
- "Supplemental findings, edit pipeline", a verified extraction of the shared edit module at bundle offsets 4150000 to 4170000 (bundle).

Other harnesses, each at the pin named in the evidence report:

- Claude Code 2.1.311, win32-x64 bundle: the `Edit` tool, the read-state store, and 126 extracted error strings (bundle).
- Copilot: `microsoft/vscode`, `extensions/copilot/src/extension/tools/node/`, commit `b208bc25` (2026-10-25) (source).
- Zed: `crates/agent/src/tools/`, commit `37faef81f7` (2026-10-27), with `edit_file_tool.rs`, `edit_session.rs` and `edit_session/streaming_fuzzy_matcher.rs` (source, probe).
- pi: `packages/coding-agent/src/core/tools/`, commit `a92283056` (2026-10-27), package 1.4.0, with commit `128330e36` (2026-06-19, issue 5899, pi 0.79.9) (source, probe).
- rig: `crates/rig-tools/src/edit.rs` and `edit/tests.rs` at commit `a83b774d0` (2026-10-25). The reversal commit `c2b6c392383107cbb96f5087182c08f85497b22d` (2026-10-19, PR 2857), its README and doc-comment diff, and a `git grep -il triobench` at `a83b774d0` were read directly for this report (source).
- zeroclaw: `crates/zeroclaw-tools/src/file_edit.rs`, version 0.8.5, commit `2d85371f3` (2026-09-18) (source, probe).
- Codex CLI: `openai/codex`, commit `bea47c06dfac` (2026-10-28), `codex-rs/apply-patch/src/` (source, web).
- Gemini CLI: `google-gemini/gemini-cli`, tag v0.66.0, commit `970413207732`, `packages/core/src/tools/edit.ts` (source).
- OpenCode: `anomalyco/opencode`, tag v1.19.5, commit `810c0eba1a73`, and dev `fa1260d6ca` (source, probe).
- Cline: `cline/cline`, tag v4.1.26, commit `a459f1d` (source, probe).
- Aider: `Aider-AI/aider`, commit `5dc9490bb35f` (2026-05-22), `aider/coders/editblock_coder.py` (source, probe).
- Goose: `aaif-goose/goose`, commit `ee8500d5e060` (2026-10-28), `crates/goose/src/agents/platform_extensions/developer/edit.rs` (source, probe).
- Web, via the evidence report and not re-read: [Aider issue 306](https://github.com/Aider-AI/aider/issues/306), Claude Code issues 26996 and 29870 (web).

Written for this report: a throwaway JavaScript prototype of the recommended pipeline, which ran the vectors of behaviors 2, 3, 4, 6, 12 and 13 (probe).

Runvil, repository `runvil` at commit `08801150f` (2026-10-27, "Add the std Pack with Cursor's six file tools") (source):

- `crates/runvil-internal/vfs/src/handle/operations.rs`, `handle/claims.rs`, `handle/scope.rs`, `handle/forward.rs`, `handle/tests/happens_before.rs` and `router-access.rs`: the `Access` surface and the claims rules.
- `crates/runvil-internal/vfs/src/traits/backend.rs`, `error.rs` and `stat.rs`: the default `str_replace`, the `VfsError` variants and `Stat`.
- `crates/runvil-plugin/src/context.rs` and `pack.rs`: the call's access, the no-wait invariant and `Pack::render`.
- `crates/runvil-internal/types/src/tools/origin.rs`, `crates/runvil-internal/engine/src/execute/scheduler/apply.rs`, `crates/runvil-internal/engine/src/execute/scheduler/tool_call.rs` and `crates/runvil-internal/lua/src/engine_globals.rs`: the run identifier, the conflict handling for a store answer, a tool's failure as untrusted failure text, and Lua's `store.str_replace`.
- `crates/runvil-internal/vfs/src/real/resolve.rs` and `real/tests/links.rs`: how content operations treat a link.
- `crates/plugin-std/src/pack.rs`, `schemas/replace.json` and `schemas/EDITS.md`: the shipped stub and definition.
- `vibe/2026-10-27-2-plugin-std-file-tools.md`: the standing plan, its todo ids and its open question.

*2026-10-27 - Claude Sonnet 5.5*
