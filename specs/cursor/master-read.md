# Build Read as a numbered, capped text reader over a bounded read

## Summary

I recommend that Runvil's `Read` be a numbered, capped, bounded text reader in `plugin-std`, built with two added VFS methods, `Access::read_range` and `Access::canonical_path`. Five decisions matter most:

- Number every line with a six-wide pipe prefix, always on, and fix the shipped description, which promises `cat -n` and a 2,000-character cut that Cursor never had.
- Stop at 100,000 bytes of output on a line boundary and end with a trailer that names the next offset, where Cursor errors with empty content. No cut ever splits a UTF-8 code point: the one cut inside a line steps back to a character boundary.
- Bound the read: check the node type with `stat`, then read at most 16 MiB plus one byte, so a link to a huge file cannot exhaust memory.
- Make lines uniform: strip one `\r` from each line, drop the empty element after a final newline, and decode lossily with a note instead of returning `NotUtf8`.
- Name the path in every failure. Cursor's missing-file and denied-read errors omit it.

The largest risk is that none of the twelve harnesses measured how models cope with a read format, so the prefix and the cap rest on attestation, not data. Confidence is medium overall: high on four behaviors, medium on nine. The next step, owned by Vinnie, is the `read-real` todo, which follows a new `refusal-text` todo for the shared failure sentences. Its first commit adds the two `Access` methods and regenerates `crates/runvil/public-api.txt`. Read writes the record that the write and str-replace reports need once `write-real` adds its run service. Table 1 lists all thirteen behaviors.

## Contents

- [Summary](#summary)
- [Contents](#contents)
- [The recommendation in brief](#the-recommendation-in-brief)
- [Criteria](#criteria)
- [Options and the do-nothing case](#options-and-the-do-nothing-case)
- [Behaviors to implement](#behaviors-to-implement)
  - [1. Number every line with a six-wide pipe prefix, and say so in the description](#number-every-line-with-a-six-wide-pipe-prefix-and-say-so-in-the-description)
  - [2. Stop at a 100,000-byte cap on a line boundary and name the next offset](#stop-at-a-100000-byte-cap-on-a-line-boundary-and-name-the-next-offset)
  - [3. Bound the read with a type check and a 16 MiB ceiling](#bound-the-read-with-a-type-check-and-a-16-mib-ceiling)
  - [4. Read a window by one-based offset and limit, and count a negative offset from the end](#read-a-window-by-one-based-offset-and-limit-and-count-a-negative-offset-from-the-end)
  - [5. Name the path and the next step in every failure](#name-the-path-and-the-next-step-in-every-failure)
  - [6. Report the same line total whether or not the file ends in a newline](#report-the-same-line-total-whether-or-not-the-file-ends-in-a-newline)
  - [7. Strip one trailing carriage return from every line](#strip-one-trailing-carriage-return-from-every-line)
  - [8. Decode lossily, hide the byte order mark, and say when bytes were replaced](#decode-lossily-hide-the-byte-order-mark-and-say-when-bytes-were-replaced)
  - [9. Refuse binary content by sniffing the first 8,192 bytes](#refuse-binary-content-by-sniffing-the-first-8192-bytes)
  - [10. Turn a sibling conflict into a retry instruction and never retry inside Read](#turn-a-sibling-conflict-into-a-retry-instruction-and-never-retry-inside-read)
  - [11. Validate arguments in the body: integers, a limit of at least 1, no undeclared parameters](#validate-arguments-in-the-body-integers-a-limit-of-at-least-1-no-undeclared-parameters)
  - [12. State every limit in the description and pin the text with a test](#state-every-limit-in-the-description-and-pin-the-text-with-a-test)
  - [13. Publish a read record to a per-run service, and keep the render pure](#publish-a-read-record-to-a-per-run-service-and-keep-the-render-pure)
- [Where Runvil should differ from Cursor](#where-runvil-should-differ-from-cursor)
- [Mapping to Runvil](#mapping-to-runvil)
- [Risks and open limits](#risks-and-open-limits)
- [Sources](#sources)

## The recommendation in brief

| Behavior | Recommendation | Confidence |
|---|---|---|
| [Number every line with a six-wide pipe prefix, and say so in the description](#number-every-line-with-a-six-wide-pipe-prefix-and-say-so-in-the-description) | Prefix every line with a six-wide number and a pipe, always on, and describe it | Medium, the best-attested format, but four of nine other tools number nothing |
| [Stop at a 100,000-byte cap on a line boundary and name the next offset](#stop-at-a-100000-byte-cap-on-a-line-boundary-and-name-the-next-offset) | Stop at 100,000 rendered bytes on a line boundary, never inside a character, and end with a trailer that names the next offset | Medium, six of nine other tools cut and mark, but the number is unmeasured |
| [Bound the read with a type check and a 16 MiB ceiling](#bound-the-read-with-a-type-check-and-a-16-mib-ceiling) | Check the node type with `stat`, then read at most 16 MiB plus one byte through a new `Access::read_range` | Medium, Goose tests the same idea, but 16 MiB is a judgment among 10, 16 and 20 MiB |
| [Read a window by one-based offset and limit, and count a negative offset from the end](#read-a-window-by-one-based-offset-and-limit-and-count-a-negative-offset-from-the-end) | Offset counts from 1, `-1` is the last line, clamp when the window overlaps the file, fail when it selects nothing, refuse `limit` below 1 | Medium, Cursor's own arithmetic, but negative offsets exist in one harness behind a flag |
| [Name the path and the next step in every failure](#name-the-path-and-the-next-step-in-every-failure) | Put the path in every failure, name the next step, and answer a zero-byte file with `File is empty.` as a success | High, Cursor's omissions are observed in the bundle |
| [Report the same line total whether or not the file ends in a newline](#report-the-same-line-total-whether-or-not-the-file-ends-in-a-newline) | Drop the empty element after a final newline and use one total everywhere | Medium, pi's fix and Runvil's own helper against Cursor's count |
| [Strip one trailing carriage return from every line](#strip-one-trailing-carriage-return-from-every-line) | Remove one trailing `\r` from every line, with no vote | High, nine harnesses reach the same answer |
| [Decode lossily, hide the byte order mark, and say when bytes were replaced](#decode-lossily-hide-the-byte-order-mark-and-say-when-bytes-were-replaced) | Decode lossily, drop a UTF-8 mark, decode UTF-16 by its mark, and add a note when bytes changed | Medium, the decode ladder is settled but the note has no precedent |
| [Refuse binary content by sniffing the first 8,192 bytes](#refuse-binary-content-by-sniffing-the-first-8192-bytes) | Refuse on a NUL or 5% control bytes in the first 8,192 bytes, with no extension list | High, four harnesses sniff content |
| [Turn a sibling conflict into a retry instruction and never retry inside Read](#turn-a-sibling-conflict-into-a-retry-instruction-and-never-retry-inside-read) | Map `Conflict` to a read-again sentence and never retry inside Read | Medium, documented in `ToolContext` but untested |
| [Validate arguments in the body: integers, a limit of at least 1, no undeclared parameters](#validate-arguments-in-the-body-integers-a-limit-of-at-least-1-no-undeclared-parameters) | Change `offset` and `limit` to `integer`, refuse `limit` below 1 and undeclared parameters, read `null` as absent | Medium, only rig refuses unknown parameters |
| [State every limit in the description and pin the text with a test](#state-every-limit-in-the-description-and-pin-the-text-with-a-test) | Fix the three false lines in `read.json`, state every limit, and pin the text with a test | High, the false lines are observed in the shipped file |
| [Publish a read record to a per-run service, and keep the render pure](#publish-a-read-record-to-a-per-run-service-and-keep-the-render-pure) | Write a hash, the shown range, the total and a whole-file flag to a run service, and keep the render a pure function | Medium, two sibling reports and two harnesses converge, but the service does not exist yet |

*Table 1. The thirteen behaviors, the recommendation for each, and its confidence.*

## Criteria

Every option below is judged against six criteria. Each term means the same thing wherever it appears.

- **Accuracy.** The model gets what it asked for in few calls, and no result is silently wrong or partial. Runvil gives a model 24 tool rounds per `models.loop` call by default, so a wasted round draws on a fixed budget (`RunLimits::default`, source).
- **File safety.** A read changes nothing on disk, and nothing a read shows can lead a later write to destroy content.
- **Anchor fidelity.** Text the model copies out of a read, minus the line prefix, either matches the file or fails loudly when an edit tool uses it as an anchor.
- **Replay.** The same bytes and arguments give the same output, byte for byte, so a recorded run replays.
- **Bounded cost.** Tokens, memory and blocking time per call have a ceiling, and the description tells the model what it is. The Harness polls every effect on one task and a real-disk read blocks that task, so memory and time matter as much as tokens (`effect_loop.rs` and the plan's constraints, source).
- **Fit.** The tool is implementable on Runvil's VFS, and the report names each gap.

Three terms stay fixed. A *line* is the text between newlines. The *cap* is the most output one read returns. The *ceiling* is the largest file Read will open.

Each claim states its basis. An observed claim names a tier: `(source)` for code read at a named commit, `(web)` for a fetched page, `(bundle)` and `(daemon)` for Cursor's shipped files, `(probe)` for something I ran. An assumption begins "I assume" and a judgment begins "I judge". Confidence is high, medium or low and describes the evidence. Likelihood uses one ladder: very unlikely, unlikely, roughly even chance, likely, very likely, almost certain. The `(probe)` claims I made myself come from two runs on this Windows machine: string and offset checks against Cursor 3.27.12's bundle and daemon, and a standalone Rust model of the recommended behaviors that passed 84 checks. I also re-read constants in the Copilot, pi, rig and Zed checkouts `(source)`. Every other tier label comes from the evidence report and is cited by its section.

## Options and the do-nothing case

Nine decisions have real alternatives. Each subsection states the recommended option first, then weighs it against the others, with the do-nothing option named in every table. The column "Who does the work" names the component that performs the step: the Pack (`plugin-std`), the VFS, the Harness or the model.

### Number every line, because the prefix costs 7 bytes and lets the model cite and page

The recommended option prefixes every line with a six-wide number and a pipe. Table 2 sets it against the shipped promise, an optional argument, plain text with a typed record, and a tab separator.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: keep `read.json` as shipped | Pack | no formatter to write | the description promises `cat -n` and a 2,000-character cut that the code would not do | Reject |
| Always numbered, six-wide pipe | Pack | one rule, citable lines, Cursor's format, a fixed width | 7 bytes per line; the model strips the prefix before using an anchor | **Recommended** |
| Numbered behind an `include_line_numbers` argument | Pack and model | a model can skip the cost | a fourth parameter; with Cursor's flag on, an omitted argument means no numbers | Reject |
| Plain lines plus a typed record (rig) | Pack and host | no prefix tokens; the host can draw numbers | the model has nothing to cite; `ToolOutput` has no side channel | Reject for now |
| Number and tab (Zed, Claude Code) | Pack | parses without ambiguity | width moves, or a tab collides with tab-indented text | Reject |

*Table 2. Five ways to number lines. The do-nothing row keeps the stub's description.*

The shipped promise loses because it is false: Cursor's build never says `cat -n` and never cuts long lines (`tool-read.md`, "What the stub gets wrong today"). The argument loses because Cursor's own version of it is a feature-flag experiment whose default silently removes the numbers. Plain text with a record is a sound design in rig, where a terminal draws the numbers from the record, but a model reads no numbers there and Runvil's `ToolOutput` cannot hold a record. A tab separator is what Zed and `cat -n` print, and Claude Code switches to `12:content` when the text itself starts with a tab, which shows that the tab collides with content. The winner costs 7 bytes per line, about 17% on lines of 40 bytes (derived from the format), and it asks the model to strip the prefix, which the description must say.

### Cut on a line boundary and name the next offset, because an error returns nothing

The recommended option returns a page and a trailer. Table 3 compares it with no cap, Cursor's error, an outline, and two ways of cutting inside a line.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: no cap | Pack | no trailer to write | a 5 MB file is about 1.25 million tokens (estimate) | Reject |
| Error with empty content (Cursor) | Pack | nothing partial is ever returned | one round, no content, no next offset named | Reject |
| Cut on a line boundary, trailer names the next offset | Pack | a page and the next call in one round | a model that ignores the trailer believes it saw the whole file | **Recommended** |
| Outline instead of content (Zed) | Pack and a parser | the shape of a large file | a grammar per language; a ranged read has no cap | Reject |
| Cut each line at 2,000 characters (Copilot, Gemini CLI, OpenCode, Cline) | Pack | a minified line cannot flood a page | long prose lines are cut and the rest is unreachable | Reject |
| Fit one line width to the window (rig) | Pack | files of long lines read further | the same line renders differently by window; cut lines unreachable | Reject |

*Table 3. Six ways to handle a read that is too large. The do-nothing row applies no cap.*

A cap is needed because the alternative is unbounded cost. Cursor's error is unambiguous but uses a round, and the model's only escape is to guess a window. Zed's outline is a good answer for code and needs a parser and a grammar race. The per-line cut loses here for a reason specific to this workspace. Its rule says markdown files do not hard-wrap, which makes lines over 2,000 characters ordinary, and Runvil's Plan mode writes only markdown paths (workspace rule `always.mdc` and the std plan, source). In rig, a fitted width shows the failure at scale: a 3,000-line file with lines of 101 characters or more shows 478 lines cut at 100 characters, where whole lines at the same budget would show 501 (probe; `tool-read.md`, "rig"). The winner can mislead a model that skips the last line, so the trailer sits on the output's last line, and a single line over the cap stays unreachable, which the risks section records.

### Bound the read with a range read, because a size check cannot see through a symlink

The recommended option checks the node type with `stat` and reads at most the ceiling plus one byte. Table 4 compares it with reading the whole file, a `stat` size check, and streaming.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: `Access::read` the whole file | Pack | no VFS change | memory equals file size; one blocked task; a 4 GB link loads in full | Reject |
| `stat` size check, then `read` | Pack | no VFS change | `stat` reports a link's own size; a gap between check and read | Reject as the only guard |
| `stat` for node type, then `read_range(0, ceiling + 1)` | VFS (one method) and Pack | bounded memory for a file or a link; honors read policy | one `Access` method and one `public-api.txt` line | **Recommended** |
| Stream a line window in chunks (OpenCode, Cline, pi's durable package) | VFS and Pack | memory bounded by the window, no ceiling | the total still needs a scan to the end; more code | Defer |

*Table 4. Four ways to bound a read. The do-nothing row is the stub's whole-file read.*

Reading the whole file is what Cursor does, and it silently cuts at 8 MiB while doing it. Runvil cannot copy that on one polling task. A `stat` check alone fails on a link, because `RealBackend::stat` does not follow it. Streaming is the better design if Runvil later needs to read multi-gigabyte logs, and I defer it because under a 16 MiB ceiling the memory it saves is small. A read of 16 MiB took a median 5.7 ms in my model (probe). The cost of the winner is a change outside the Pack: `Access::read_range` must exist before `read-real` can use it, and `crates/runvil/public-api.txt` changes.

### Strip each carriage return and drop the final empty line, because a vote leaks one and a phantom line miscounts

The recommended option splits on `\n`, drops the empty element after a final newline, and strips one `\r` from each line. Table 5 compares it with four other rules.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: `str::lines` | Pack | two lines of code; the same count as the recommendation | a bare final `\r` survives; `NotUtf8` on bad bytes | Close; add the final-`\r` strip |
| Newline count plus one, then strip (Cursor's count) | Pack | the format models know; the final newline stays visible | a numbered blank line the file does not have; a total one too high | Reject |
| A 5% CRLF vote (Cursor) | Pack | leaves mixed files alone | `\r` reaches the model below 5%; two paths can disagree | Reject |
| Pass `\r` through (pi, Goose) | Pack | no code | the model copies `\r` into anchors | Reject |
| Split on `\n`, drop the final empty element, strip one `\r` | Pack | one rule; the same total with or without a final newline | the final newline is no longer visible | **Recommended** |

*Table 5. Five line models. The do-nothing row is `str::lines`.*

`str::lines` is the natural first attempt and it is nearly right. The recommended option is that rule plus a strip for a final bare `\r`, written out so the count and the paging use one split. Cursor's count has a real benefit, since its last numbered line is blank exactly when the file ends in a newline. I set it aside because pi counted the same way until it fixed the phantom line on 2026-05-21 (commit `f95306781`). The 5% vote and the pass-through lose on anchor fidelity.

### Decode lossily with a note, because refusing leaves the model nothing to do

The recommended option decodes bytes lossily, understands byte order marks, and adds a note when bytes were replaced. Table 6 compares it with the stub's `read_string`, refusing, silent loss and full detection.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: `read_string`, return `NotUtf8` | Pack | no decode code; an honest VFS error | a Latin-1 file is unreadable; the error names a byte offset | Reject |
| Refuse with a clear message (rig, Aider) | Pack | no silent loss | the same unreadable files | Reject |
| Lossy UTF-8 with no report (pi, OpenCode, Cline, zeroclaw) | Pack | readable text | an edit near a replaced byte fails with no explanation | Reject |
| Lossy, byte order mark aware, with a note | Pack | readable text and an explanation | the note is new and unmeasured | **Recommended** |
| Detect the encoding (Zed's `chardetng`, Cursor's Latin-1 rescue) | Pack and a dependency | legacy encodings read as text | code and a dependency; mojibake on a wrong guess; Write must re-encode | Defer |

*Table 6. Five decoding policies. The do-nothing row is the stub's `read_string`.*

Runvil's `NotUtf8` is the right error for a VFS and the wrong answer for a tool, because the model can do nothing with it. Silent replacement keeps files readable and hides the cause of a later edit failure. Detection is where Zed ends up, with tests for Windows-1252, GBK and Windows-1251 round trips, and it is useful only when Write can write the same encoding back. The write report sets that aside for the first version (`master-write.md`, "Refuse to rewrite a file that is not valid UTF-8, and write UTF-8 as sent").

### Sniff content for binary, because an extension list adds nothing once the bytes are in memory

The recommended option refuses on a NUL byte or a high share of control bytes in the first 8,192 bytes. Table 7 compares it with no check, Cursor's two-stage check, and a hexdump.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: treat every file as text (pi, rig, Cline) | none | no code | NULs and control bytes reach the model | Reject |
| Extension list, then sniff (Cursor) | Pack | skips I/O for known types | a text `.bin` is refused; `.jar` and `.wasm` rely on the sniff anyway | Reject |
| Sniff only | Pack | one rule on bytes already read | UTF-16 without a mark is refused | **Recommended** |
| Hexdump with byte offsets (Copilot) | Pack | binary files become readable | `offset` and `limit` change units by content | Reject |

*Table 7. Four binary policies. The do-nothing row skips the check.*

Copilot's hexdump reuses two parameters in two units chosen by inspecting the content, so a model cannot predict what `limit: 100` returns; I take that as the idea to reject. The refusal text for binary can still follow zeroclaw and name the next move when the run offers one, which I leave to a later description edit.

### Clamp a window that overlaps the file, because a model's tail request should work

The recommended option keeps negative offsets and fails only when no requested line exists. Table 8 compares it with no negative offsets and two stricter rules.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: no negative offsets (all nine other tools) | Pack | the simplest schema | no one-call tail; Runvil has no shell | Reject |
| Negative offset, clamp when the window overlaps, error when it selects nothing | Pack | one sentence; the tail request works on any file | the asymmetry needs a description sentence | **Recommended** |
| Negative offset, error when the magnitude exceeds the file | Pack | a symmetric rule | a reasonable request fails and costs a round | Reject |
| Positive overshoot returns an empty success or a note (Zed, Cline, rig, zeroclaw) | Pack | no error to handle | a model reads an empty file or the wrong lines | Reject |

*Table 8. Four rules for windows and offsets. The do-nothing row drops negative offsets.*

Dropping the feature is what the other nine tools do. Codex CLI and Goose, which have no read tool, get the tail of a log from `tail` in a shell, and Runvil has no shell. A symmetric error is what the evidence report suggests, and it fails the plain case of asking for the last 100 lines of a short file. The winner keeps Cursor's asymmetry and gives it a reason.

### Translate a conflict, because Runvil reports the race that Cursor retries

The recommended option answers a sibling conflict with a sentence that tells the model to read again next round. Table 9 compares it with raw text, Cursor's retry, and ordering by footprint.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: show the raw `conflicting claim: {detail}` | none | no code | claim identities the model cannot use | Reject |
| Translate and let the next round retry | Pack | a clear instruction; no wait | one round when the race fires | **Recommended** |
| Retry once inside Read after waiting (Cursor) | Pack and a Harness settle signal | hides the race from the model | a signal the Pack lacks; a wait of up to 120,000 ms | Reject |
| Order the batch by declared footprint (rig) | Harness | no round lost | a scheduler that sees every call first | Defer |

*Table 9. Four answers to a Read that collides with a sibling Write. The do-nothing row shows the raw error.*

Ordering by footprint is the strongest design in the evidence, since no round is lost, and it is a Harness change outside this crate. I defer it and revisit it if the conflict fires often.

### Publish a read record, because the write and str-replace reports both need what Read last showed

The recommended option has Read write a hash, a shown range and a whole-file flag to a per-run service, and leaves every gate to the tools that write. Table 10 compares it with recording nothing, a gate inside Read, transcript rewriting and re-sending the bytes.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| Do nothing: record nothing (pi, Gemini CLI, OpenCode) | none | no state, full replay | Write and StrReplace cannot tell a partial view from a whole one | Fallback if both write reports drop their record |
| Read-before-write gate (Claude Code) | Read records, Write refuses | refuses a blind write | a read round before every first write; model cooperation | Write's decision; Read supplies the record |
| Record a hash, shown range and whole flag in a run service (rig, Claude Code) | Read writes; Write and StrReplace read | no refusal in Read; one record serves two consumers | a service key, a `needs` entry, a Host provider, a `sha2` dependency | **Recommended** |
| Rewrite stale reads in the transcript (Cline) | Harness | a smaller context | keyed on path and range, so an edit does not invalidate it | Reject |
| Re-send current bytes each turn (Aider) | Harness | the model never sees stale text | tokens every turn; a small file set | Reject |

*Table 10. Five ways to track what a read showed. The do-nothing row records nothing.*

Two sibling reports move the answer away from do nothing. `master-write.md` adopts a gate that needs a whole-file flag and a hash from Read, and `master-str-replace.md` adopts a stale diagnosis that needs the same hash. Claude Code's gate adds a round trip for every first write, which is Write's cost to weigh, not Read's. rig's hash is satisfied by a read that displayed nothing, so the entry records the shown range and sets `whole` only for a complete, uncut read. The winner costs one pass over the bytes for the hash and a new run service. If the two sibling reports drop their records, the do-nothing row becomes the right answer and this behavior disappears.

## Behaviors to implement

Thirteen behaviors follow, ordered by what an error costs. The first three set the contract the model sees and the limits that protect the run. Table 1 lists all thirteen with their confidence.

### Number every line with a six-wide pipe prefix, and say so in the description

**Recommendation.** Prefix every line of a successful read with its one-based line number, right-aligned in six characters, then a pipe, then the text with no space: `     1|const x = 1;`. Always emit the prefix, with no argument to turn it off. Let a number wider than six digits grow wider instead of cutting it.

**Why.** A model that wants to cite line 41, or ask for `offset: 41`, reads the number off the page instead of counting, which serves accuracy. The prefix is not file text, so the description must say where it ends, which serves anchor fidelity. A fixed width renders a given line the same way in every window, which serves replay. Runvil's own Lua helper `store.read_numbered` sizes the number to the largest in the window, so line 9 prints one way in a window that ends at 9 and another in a window that ends at 12 (source). The prefix adds 7 bytes per line, or 14,000 bytes for 2,000 lines, and the cap in the next behavior counts those bytes. I assume models trained on Cursor transcripts parse this prefix without help, but no source measures it.

**Evidence.** Cursor's default formatter pads the number to six and adds a pipe, and its shipped description says `using following format: LINE_NUMBER|LINE_CONTENT` (bundle, probe: two occurrences of each). The string `cat -n` occurs zero times in the 9,814,287-byte bundle (probe), so the line in the shipped `read.json` that promises `cat -n` describes no Cursor build (`tool-read.md`, "The pipe is not `cat -n`"). Six of the ten read tools in the evidence number their lines: Cursor, Zed (six wide and a tab), Claude Code (an unpadded number and a tab, or `12:content` for tab-led text), OpenCode and zeroclaw (`1: text`) and Cline (`   1 | text`). Copilot, pi, rig and Gemini CLI print none and anchor edits by text (source; `tool-read.md`, "Across all the harnesses"). Cursor's `include_line_numbers` argument exists only behind a flag, and with the flag on an omitted argument means no numbers (bundle; `tool-read.md`, "Decision 1" and "Traps").

**Without it.** A model told `cat -n` and shown a pipe may paste anchors with the prefix attached, and I judge the resulting edit failures would look like matching bugs. With plain text and no numbers, a model that needs line 41 of a long file counts by eye or asks for a window and misjudges it.

**Test.** Read a three-line file and assert that bytes 0 to 5 of the output are five spaces and `1`, that byte 6 is `|`, that the output holds no tab, and that `cat -n` does not appear. Read a 12-line file twice, lines 1 to 9 and lines 1 to 12, and assert that line 9 renders identically in both. Format line number 1,000,000 and assert that it prints as `1000000|`.

**Confidence.** Medium. The format has the widest attestation and the description fix removes a known mismatch, but four of the nine other tools number nothing and no source measures how models cope with any format.

### Stop at a 100,000-byte cap on a line boundary and name the next offset

**Recommendation.** Count the bytes of the numbered lines the read will return, newlines included, and stop before the line that would pass 100,000 bytes, so a cut never splits a line. The cap counts the numbered lines only. The trailer and any note come after them and are not counted. When a read stops before the last line, end the output with one trailer line that names the next offset: `[Showing lines 1-2083 of 5000 (100000-byte limit). Use offset=2084 to continue.]`. When the caller's `limit` ended the read, leave out the parenthesis. When the first line alone passes the cap, keep the longest prefix of that line that fits the cap and ends on a character boundary, and say so: `[Line 1 is 150000 bytes and is cut after 99992 bytes. No parameter reads the rest of a line.]`, with ` Use offset=2 to continue.` before the closing bracket when more lines follow. The number in the notice is the bytes kept, not the bytes allowed. The allowance is the cap less the 7-byte prefix and the newline, 99,992 bytes. An ASCII line keeps all 99,992. A line of 3-byte characters, such as CJK text, keeps 99,990, so its notice says `cut after 99990 bytes`.

**A cut never splits a code point.** Read shortens text in exactly one place, the first-line cut above. Every other stop is at a line boundary, where no character can be split. The first-line cut works on decoded text and not on raw bytes. It takes a `&str` and returns a `&str`, steps back from the byte limit until `str::is_char_boundary` is true, and never slices by a byte index without that check, so the type system rules out invalid UTF-8 in its result. Three other paths cannot split a character either:

- The 16 MiB ceiling refuses the file and never truncates it, so decoding never sees a buffer that ends mid-sequence because of Read.
- A sequence that a file itself leaves unfinished at its end is one invalid sequence and shows as U+FFFD under the lossy decode. That is damage in the file and not a cut Read made.
- UTF-16 is decoded in full before any cut, so a surrogate pair is already one character when the cut looks for a boundary.

The cut protects code points, not grapheme clusters. A combining mark or an emoji joined with a zero-width joiner can be separated from its base. I accept that, because the cut applies only to a single line that is already longer than the whole cap. The cut function lives in the shared `text` module that Grep also calls (`clip_to_boundary`), so Grep's long-line window and context-line clip use the same function and the same rule. The cap constant is far larger than any code point, so the kept prefix is never empty.

**Why.** A cut that names its next offset costs the model one more call and leaves nothing to guess. An error costs one call and returns nothing, and a silent cut lets the model reason from part of a file without knowing it, which breaks accuracy. The cap counts the numbered lines the model receives, prefixes included, where Cursor counts the text before it adds the prefixes (derived from `tool-read.md`, "Algorithm and ordering", steps 17 and 18). Bytes need no second pass over the text in Rust. A last page that starts after line 1 prints no trailer, because its line numbers already show where the file ends. Glob and Grep page over results with no running number, so they print the range. Applying the cap after the window keeps the one good property of Cursor's order: a two-line window into a large file always succeeds. I judge 100,000 because it matches Cursor's figure for ASCII text, the number the model's habits formed on; that is an assumption, and pi, rig and OpenCode stop at 51,200 bytes.

**Evidence.** Cursor errors when the window passes 100,000 UTF-16 units: the record is a success with empty content and the error flag set, and the message names `offset` and `limit` but no next value (bundle; `tool-read.md`, "Q12 Truncation and omission"). Six of the nine other tools cut the window and say so: Copilot, pi, rig, Gemini CLI, OpenCode and Cline. Claude Code gives an unranged read a first page ending `Do NOT answer from this page alone if the answer may be further in the file` and errors only on a ranged read. Zed answers an unranged read of a buffer over 16,384 bytes with an outline (source). pi's trailer `[Showing lines {start}-{end} of {total}. Use offset={end + 1} to continue.]` and its `(50.0KB limit)` cause form are pinned by four tests, and a probe that followed `Use offset=N` page by page rebuilt the original text in four cases (source, probe). Cline wrote its marker and a later stage cut 8,000 characters from the result, so the marker lied (source, probe). In my Rust model, 5,000 lines of 40 bytes give a first page of 2,083 lines and 100,064 bytes with the trailer, and following the trailers rebuilt the original in all 60 combinations of 6 line counts, 2 final-newline states and 5 cap and limit settings (probe).

**Without it.** With no cap, a 5 MB file arrives as about 1.25 million tokens at the four-bytes-per-token estimate Codex CLI uses (source). With Cursor's error, a model that reads a 120 KB file with no arguments gets nothing back, uses one of its 24 rounds, and must pick a window with no hint.

**Test.** Read 5,000 lines of 40 `x` and assert 2,083 numbered lines, the trailer above, and a length of at most 100,120 bytes. For files of 1, 2, 5, 37, 2,000 and 2,001 lines, each with and without a final newline, follow `Use offset=N` page by page under caps of 100,000, 300 and 90 bytes and limits of none, 50 and 7, and assert that the rebuilt lines equal the original and no page is empty. Read a 150,000-byte line and assert 99,992 kept bytes. Read one line of 50,000 CJK characters and assert 99,990 kept bytes, a notice that says `cut after 99990 bytes`, and valid UTF-8. Do the same with a line of 2-byte characters (99,992 kept) and a line of 4-byte emoji (99,992 kept, which is 24,998 characters). Sweep the cap from 12 to 60 bytes over a first line that mixes 1-, 2-, 3- and 4-byte characters, and at every cap assert that the kept bytes are valid UTF-8 (`String::from_utf8` succeeds), that they are a prefix of the line, that they fit the allowance, and that the notice's count equals their length. Read a file whose last bytes are the first two bytes of a 3-byte sequence and assert one U+FFFD and the invalid-sequence note, with no panic. Pass the output through the Harness's result wrapper and assert that the trailer survives.

**Confidence.** Medium. Six of nine other tools reach the same shape of cut, mark and next offset, but the number 100,000 has no measurement behind it, and a single line over the cap stays unreachable.

### Bound the read with a type check and a 16 MiB ceiling

**Recommendation.** Call `Access::stat` first. Refuse a directory by name, and refuse a FIFO, socket or device by name. For a regular file or a symlink, call a new `Access::read_range(path, 0, 16_777_217)`. If it returns more than 16,777,216 bytes, refuse with `File is too large to read: {path} is over the limit of 16777216 bytes.` Give `Access::read_range` the same four steps as `read` (gate, admit `claim_read`, fire `Op::Read`, delegate), so a policy that denies reads denies it too. The ceiling is a refusal and never a cut: Read decodes only a buffer it read to the end, so the bound cannot split a character.

**Why.** `Access::read` loads the whole file, and the Harness polls every effect on one task where a real-disk read blocks, so an unbounded read is a memory and latency hazard that a hostile repository can force; that is bounded cost. A `stat` check cannot guard it alone. `RealBackend::stat` uses `symlink_metadata`, so for a symlink `Stat.size` is the link's own size and a link to a multi-gigabyte file looks small (source; `tool-read.md`, "Q8 Paths"). A read of at most ceiling plus one bytes bounds the memory for a regular file and for a link to one. A link to a FIFO would still block, which the risks section names. In my model, reading a 16 MiB file took a median of 5.7 ms and rendering its first page 25.4 ms (probe, one machine, warm cache), so the worst case blocks the task for about 30 ms. For fit, `VfsAccess::read_range` already exists with a seeking override in `RealBackend` and delegation through three layers, and only `Access` lacks it (source).

**Evidence.** Cursor never seeks: it reads the whole file, decodes it and splits it, so a 100 MB ASCII file costs about 100 MB of buffer plus 200 MB of string, and its 8 MiB cut is silent because the `truncated` flag is set and never rendered (bundle, daemon; `tool-read.md`, "Q7 Cost"). File ceilings elsewhere are rig 16,777,216 bytes, Gemini CLI 20 MiB, zeroclaw 10 MiB and Cline 100,000,000 bytes, each tested from a `stat` (source). pi has none, and its read of a 100 MB file with `limit 10` took 136 ms and 165 to 308 MB of memory (probe). Goose's `read_bounded` is `take(max + 1).read_to_end` plus a length check, tested with an infinite source (source). OpenCode, Cline, pi's durable package and Codex CLI's removed tool stream a line window instead. I did not adopt that, because under a 16 MiB ceiling the memory it saves is small, and counting the total still scans to the end on the same blocking task.

**Without it.** A symlink to a 4 GB log passes a `stat`-based size check and then loads in full. A link to a device such as `/dev/zero` would make `fs::read` grow until memory runs out. I ran neither case; the first follows from `symlink_metadata` and the second from how `fs::read` works.

**Test.** Use a counting backend that records the `len` it receives. A file of 16,777,216 bytes reads, and one byte more is refused with the message above. A symlink to a file over the ceiling is refused, and the backend saw `len` of 16,777,217 and nothing larger. A directory gets its own message and triggers no `read_range` call. A mount that keeps the default whole-file `read_range` still works. On the real backend, a link to a file inside the run's filesystem reads as that file, and a link that leaves it answers the `OutsideRoot` sentence from the shared `refusal` function (the file-link cases are ignored on Windows, which needs the symlink privilege for a file link).

**Confidence.** Medium. The bounded read is the right shape and Goose tests the same idea, but 16 MiB sits among three harness ceilings of 10, 16 and 20 MiB, and nothing measures which files models need.

### Read a window by one-based offset and limit, and count a negative offset from the end

**Recommendation.** Take `offset` (a line number, default 1) and `limit` (a line count, default the rest of the file). Treat `offset: 0` as 1. Count a negative offset from the end, so `-1` is the last line, and read `|offset|` lines when `limit` is absent. Clamp a window that overlaps the file to the file. Fail only when the window selects no line: `Offset 11 is beyond the end of /notes.md, which has 10 lines.` Refuse a `limit` below 1 with `` std_Read: `limit` must be at least 1, and the call passed 0 ``, built by the shared argument-error helper.

**Why.** A window that has something to show should show it, because a model that asks for the last 100 lines of a 10-line file wants the file. A window that selects nothing must fail, because an empty success looks like an empty file, which is what Zed and Cline return. That one rule explains Cursor's asymmetry, where `offset: -100` clamps and `offset: 11` throws. `tool-read.md`, Decision 5, recommends dropping the asymmetry. I keep it because the rule above is one sentence in the description and a symmetric error would turn a reasonable tail request into a wasted round. Treating `offset: 0` as 1 saves a round for a model that counts from zero. Negative offsets matter more here than elsewhere: Runvil has no shell, and `tail` is how Codex CLI and Goose read the end of a log, so `offset: -50` is the only one-call tail. The `read-real` todo already lists it.

**Evidence.** On a 10-line file Cursor's `computeReadSlice` gives lines 1 to 10 for `offset: 0`, line 10 for `offset: 10`, lines 9 to 10 for `offset: -2`, lines 1 to 10 for `offset: -100`, lines 9 to 10 for `offset: -2, limit: 5`, and a throw for `offset: 11` (daemon, probe; `tool-read.md`, "The window arithmetic, exactly"). It accepts `offset: 0` and refuses `limit: 0` with `Limit must be >= 1.` (bundle). Its offset message prints `(1 lines)` for a one-line file. No other harness accepts a negative offset: Claude Code is `nonnegative`, Zed is `u32`, rig is `usize`, and Gemini CLI and Cline set `minimum 1` (source; `tool-read.md`, Decision 5). Cursor, Copilot, Zed, pi, rig and OpenCode read `offset: 0` as line 1; Gemini CLI and Cline refuse it. Offset past the end has five behaviors across harnesses: a thrown message (Cursor, pi, OpenCode, Copilot), a system reminder (Claude Code), a success note with the line count (rig, zeroclaw), an empty string (Zed, Cline), and a banner with an impossible range such as `Showing lines 11-10 of 10` (Gemini CLI) (source, probe). zeroclaw ignores a bad offset and starts at line 1 (source, probe). My Rust model reproduces the Cursor window cases and returns the error text above (probe).

**Without it.** If `offset: -100` errors, the natural tail request fails on short files and costs a round. If `offset: 11` returns an empty success, the model concludes the file is empty. If a bad offset is ignored, as in zeroclaw, the model reads the wrong lines and does not know.

**Test.** On a 10-line file assert these windows: no arguments give 1 to 10; `offset: 0` gives 1 to 10; `offset: 10` gives 10 to 10; `offset: -2` gives 9 to 10; `offset: -100` gives 1 to 10; `offset: -2, limit: 5` gives 9 to 10; `offset: -1` gives 10 to 10. Assert the exact text for `offset: 11`, and that `offset: 2` on a one-line file says `which has 1 line.` Assert that `limit: 0` answers the text above byte for byte, that `limit: -3` fails naming `limit` with the same prefix, and that an empty file with `offset: 500` still answers `File is empty.`

**Confidence.** Medium. The window arithmetic is Cursor's own and I ran its cases against a model, but negative offsets exist in one harness behind a flag and no transcript shows models using them.

### Name the path and the next step in every failure

**Recommendation.** Put the path, as the model wrote it, in every failure, and say what to do next when the tool can know. The shared `refusal` function in `plugin-std` builds the sentence for every `VfsError`, so one variant reads the same in all six tools (the `refusal-text` todo writes it before `read-real`). Read uses these texts, and the argument checks keep the `std_Read:` prefix the shared check already adds:

```text
File not found: {path}
Path is a directory, not a file: {path}
Path is neither a file nor a directory: {path}
File content is binary data, not text: {path}
File is too large to read: {path} is over the limit of 16777216 bytes.
Offset {offset} is beyond the end of {path}, which has {total} line(s).
{path} was changed, or is being changed, by another call in this round, so it cannot be read in this round. Call Read again in your next message.
I/O failure on {path} ({kind}): {message}
```

A zero-byte file is a success that answers `File is empty.` For a denied read, and for an approval-required refusal, the shared function passes the policy's reason through unchanged. Any other variant that Read can meet, such as `OutsideRoot` for a link that leaves the run's filesystem, gets the VFS `Display` text with its leading label dropped, and so does any variant added later. Refuse an empty or whitespace-only `path` before any I/O, with the helper that the four required-path tools share: `` std_Read: `path` is empty or only whitespace, and the call passed "{value}" ``. Name no other tool in a failure message, because a prompt can offer `std/read` alone. Every file tool follows this rule, and the one exception is the read-gate refusal in `master-write.md`, "Refuse to overwrite an existing file the run has not read whole and current", which has to tell the model to read. A description may name another tool.

**Why.** Cursor's `File not found` and `Permission denied`, which I judge are the failures a model meets most, contain no path, so a model that reads a batch of failed reads cannot tell which file each one means (bundle, probe). Runvil's `VfsError::PermissionDenied` holds the policy's reason text, which Cursor's boolean `isReadonly` loses (source). The VFS's own reasons name the operation and the path, as in `{op:?} on {path} is refused: ...`, so passing a reason through unchanged still names the file. A zero-byte file is a fact, not a failure, and an empty file with any offset gets the same text because the empty-file message tells the model more than an offset error would. The shared `plural(n, noun)` helper in `plugin-std` fixes Cursor's `(1 lines)`. One shared function for the sentences, and not one per tool, keeps a variant from reading six ways and gives a variant added later a readable default.

**Evidence.** `File not found` and `Permission denied` are bare `new DG("...")` throws in the bundle, one occurrence each (probe). Every other Cursor failure names the path, echoing the argument and not the resolved path (`tool-read.md`, "Result cases"). Cursor's cap message sends the model to a `grep` tool that a port may not have, and rig's shell note points at a spill path that its own `read` refuses under confinement (source; `tool-read.md`, "A message that names a recovery path the tool will refuse"). zeroclaw's binary refusal names the next move: `Use the image_info tool for images, or encoding="base64" to read the raw bytes.` (source). OpenCode adds up to three sibling paths when a file is missing, which would cost a directory listing that `read` does not do, so I did not adopt it.

**Without it.** In a batch of three reads where two fail with `File not found`, the model cannot tell which paths failed and reads all three again. A hint that names a tool the run lacks sends the model to call something that is not offered.

**Test.** For each failure above, assert that the message contains the path argument verbatim and matches the sentence byte for byte. For a denied read, use a test policy whose reason names the path, and assert that the message is that reason unchanged. Read a missing relative path `notes/a.md` and assert the message shows `notes/a.md` and not `/notes/a.md`. Read a zero-byte file with `offset: 500` and assert `File is empty.` as a success. Read with `path: "   "` and assert the blank-path text above with `"   "` as the value. Assert that no message names another tool, `Write`, `StrReplace`, `Delete`, `Glob`, `Grep` and `Shell` included.

**Confidence.** High. Cursor's own omissions are observed in the bundle and naming the path costs nothing.

### Report the same line total whether or not the file ends in a newline

**Recommendation.** Split the text on `\n`. If the text ends with `\n`, drop the empty element after it. The line total is what remains, so `a\nb\n` and `a\nb` both have 2 lines, `\n` has 1 empty line, and a zero-byte file has no lines and answers `File is empty.` Use that one count for the line numbers, for `of {total}` in the trailer, and for the offset error.

**Why.** A total of 3 for a two-line file, which Cursor reports for any file that ends in a newline, gives the model a wrong answer to "how many lines" and a numbered blank line it did not write. That breaks accuracy. I judge that agreeing with the rest of Runvil and with pi's fix is worth more than agreeing with Cursor here. Runvil's `store.read_numbered` helper already builds lines with `str::lines`, so two Runvil formats would otherwise count differently (source). The cost is information: Cursor's final numbered blank line tells the model that the file ended in a newline, and Read will not. The risks section records that, and the write report decides whether Write keeps the final newline itself.

**Evidence.** Cursor counts `\n` plus one (`countNewlines` in the daemon): `""` is 1, `"a\n"` is 2, `"\n"` is 2, and an empty file short-circuits to a literal 0 (daemon, probe; `tool-read.md`, "The line counter"). With that count a page never ends empty, because the phantom line is numbered. Until commit `f95306781` (2026-05-21) added `splitLinesForCounting`, pi counted the empty element after a final newline, and Cursor's `pi_read` port still has the bug: a file of 2,000 lines plus a newline gets a trailer that offers `offset=2001` and an empty page (source, probe; `tool-read.md`, "A continuation offset can point at an empty page"). rig and zeroclaw use `str::lines`. My model gives the same total with and without a final newline and passes every rebuild run (probe).

**Without it.** Counting with `str::lines` for the total and with `\n` for paging reproduces pi's bug: a trailer that points at an empty page, and a total off by one.

**Test.** Assert that `a\nb\n` and `a\nb` render the same two numbered lines. Assert that `\n` renders one numbered blank line and that a zero-byte file and a file holding only a UTF-8 BOM both answer `File is empty.` Assert that a 2,000-line file with a final newline, read with `limit: 2000`, ends with no trailer.

**Confidence.** Medium. pi's fix and Runvil's own helper support the count, but Cursor, whose format the model knows, counts the other way, and no source shows which count models handle better.

### Strip one trailing carriage return from every line

**Recommendation.** After splitting on `\n`, remove one trailing `\r` from every line, including a last line that has no `\n`. Never decide by a vote. Leave a `\r` inside a line alone, so a file whose only terminator is `\r` reads as one long line.

**Why.** Nine harnesses hide CRLF from the model, and only pi passes `\r` through (source; `tool-read.md`, Decision 7). A `\r` the model can see ends up inside an `old_string` that fails to match, and a per-line strip gives the model one rule with no exceptions, which serves anchor fidelity. The strip also settles a cross-tool contract. Because Read hides the `\r`, StrReplace must normalize the anchor and the file the same way before it matches, and Write must decide what ends a line. Those belong to the str-replace and write reports.

**Evidence.** Cursor calls a file CRLF only when at least 5% of its line endings are CRLF, measured over the first 8,192 bytes on the whole-file path and over the whole file on the streaming path, so the two paths can disagree. Below 5% the `\r` reaches the model, and `a\r\nb\nc\r\nd` windowed to three lines still shows `a\r\nb\nc\r` (daemon, probe; `tool-read.md`, "Q4 Line endings"). Claude Code strips a trailing `\r` per line. rig's and zeroclaw's `str::lines` strips a `\r` only before a `\n`, so a bare final `\r` survives, and a port must strip it itself (source, probe). My model renders `a\r\nb\nc\r\nd` as four clean lines and strips a final bare `\r` (probe).

**Without it.** A file with 4% CRLF lines shows a `\r` at the end of those lines. The model copies one into an `old_string`, and the edit then fails or matches against different bytes than the model believes (judgment).

**Test.** Assert that `a\r\nb\nc\r\nd` renders four lines with no `\r`, and that a last line `b\r` with no newline renders `b`. Assert that a file of 95 LF lines and 5 CRLF lines renders the same as its all-LF twin, and so does a file of 90 and 10. Assert that a file whose only terminator is `\r` renders one numbered line.

**Confidence.** High. Nine harnesses reach the same answer and Cursor's vote is the odd rule out.

### Decode lossily, hide the byte order mark, and say when bytes were replaced

**Recommendation.** Read bytes, not `read_string`, and decode in this order. Drop a UTF-8 byte order mark. If the file starts with a UTF-16 mark (`FF FE` or `FE FF`), decode it with `String::from_utf16_lossy` and add the trailer line `[Note: the file is UTF-16LE and was decoded to text.]`, naming LE or BE. Treat anything else as UTF-8 and decode it lossily. When `n` sequences were invalid, add `[Note: n invalid UTF-8 sequences in this file were shown as U+FFFD.]`, with the singular form for 1 from the shared `plural(n, noun)` helper. Grep's note uses this shape too. Never return `VfsError::NotUtf8` to the model, and do not guess Latin-1 or any other encoding. Put `decode`, `sniff_binary`, `split_lines` and `clip_to_boundary` (the one cut that never splits a code point; see "Stop at a 100,000-byte cap on a line boundary and name the next offset") in one shared `text` module of `plugin-std`, so that Grep calls the same code and a Grep line number names the same line as a Read line number (`master-grep.md`, "Decode text as UTF-8 with replacement, strip a BOM, and split lines the way Read does"). The splitter and the line formatter are pure functions of that module, and `StrReplace` builds its refusal excerpts with them (`master-str-replace.md`, "Name the cause and the next move in every refusal").

**Why.** `NotUtf8` is a hard refusal that gives the model nothing to act on, while text with replacement characters and a note lets it reason, and it warns the model that an edit near those bytes may not match. Cursor never refuses on encoding, so models trained on it never meet this failure. UTF-16 decoding costs a few lines with `std` alone, since `encoding_rs` is a workspace dependency that `plugin-std` does not need for this. Full detection, which Zed does with `chardetng`, is a fair amount of code for a case that mostly yields mojibake either way (`tool-read.md`, "Q3 Text encoding"). None of the three harnesses that decode UTF-16 tells the model which encoding it found. The write and str-replace reports refuse to edit a file that is not valid UTF-8 (`master-write.md`, "Refuse to rewrite a file that is not valid UTF-8, and write UTF-8 as sent"; `master-str-replace.md`, behavior 5), so a file that Read decodes this way is read-only through the Pack, and the edit tool says why when the model tries. The note is my addition, so I rate the behavior one step lower than the decode ladder.

**Evidence.** The harnesses follow four policies. rig refuses, and Aider drops an undecodable file with a terminal-only warning. pi, Codex CLI's removed tool, OpenCode, Cline, zeroclaw and Goose decode lossily and report nothing. Gemini CLI decodes UTF-8, UTF-16 and UTF-32 by their marks and drops them. Zed reads the mark, guesses BOM-less UTF-16, then runs `chardetng` (source; `tool-read.md`, "Q3 Text encoding"). Cursor strips a UTF-8 mark, selects UTF-16 by its mark, and for other bytes falls back to lossy UTF-8 when the file is mostly valid and to Latin-1 otherwise, reporting nothing (daemon). Only Zed, Gemini CLI and Cursor decode UTF-16 to readable text. My model turns `61 FF 62 0A` into `a` U+FFFD `b` with a note, decodes UTF-16LE and UTF-16BE files with a mark, hides a UTF-8 mark, and refuses UTF-16 without a mark as binary (probe).

**Without it.** Returning `NotUtf8` makes a Latin-1 source file, or a log with one stray byte, unreadable, and Runvil's message for it (`not UTF-8: /f.txt, valid up to byte 4812`) names an offset the model cannot use. Lossy decoding with no note lets the model edit a line that holds a replacement character, and the anchor then never matches.

**Test.** Assert that `61 FF 62 0A` renders `     1|a` U+FFFD `b` plus the note, and that a valid UTF-8 file renders no note. Assert that UTF-16LE and UTF-16BE files with a mark render their text and the UTF-16 note. Assert that a UTF-8 mark file renders its first line without U+FEFF and that a file holding only a mark answers `File is empty.`

**Confidence.** Medium. Cursor, Zed and Gemini CLI settle the decode ladder, but the note has no precedent, and a file that Read decodes this way stays read-only through the other tools.

### Refuse binary content by sniffing the first 8,192 bytes

**Recommendation.** After the byte order mark check, refuse the file if any of its first 8,192 bytes is NUL, or if at least 5% of those bytes are control bytes other than tab, LF and CR. Answer `File content is binary data, not text: {path}`. Do not keep Cursor's extension list.

**Why.** The bytes are already in memory from the bounded read, so the sniff is free, and a PNG or PDF read as text would waste a round and a large part of the cap on replacement characters. Cursor's extension list exists to throw before it reads a file that it would otherwise read whole. Under a bounded read it saves at most one 16 MiB read, about 6 ms (probe). It also refuses a text file that happens to end in `.bin`, and it misses `.jar`, `.wasm` and `.class`, which Cursor leaves to the content sniff.

**Evidence.** Cursor checks sixteen extensions before any I/O, then NUL in the first 8,192 bytes, then at least 5% control bytes (daemon; `tool-read.md`, "Q5 Binary detection"). Zed sniffs 1,024 bytes. OpenCode combines 28 extensions, a NUL in the first 4,096 bytes and 30% control bytes. zeroclaw tests an 8,192-byte sample after a UTF-8 failure. Copilot scans for a NUL and serves a hexdump. pi, rig, Cline, Goose and Aider have no binary check, and a valid-UTF-8 file full of NULs reads as text in rig, zeroclaw and Aider (source, probe). The NUL rule has no percentage, so one NUL in 8,192 bytes refuses a file. It also refuses UTF-16 without a mark, as Cursor does. My model refuses a PNG header and UTF-16 without a mark and reads a UTF-8 mark file (probe).

**Without it.** A model that asks for `logo.png` receives up to 100,000 bytes of replacement characters and control bytes, uses a round on it, and may try to interpret them. pi returns 50 KiB of garbage for a `.zip` in the same situation (source; `tool-read.md`, "pi").

**Test.** Assert that a file with a NUL at index 8,191 is refused and one with its first NUL at index 8,192 is read. Assert that a file with 5% control bytes in the first 8,192 is refused and one with 4.9% is read. Assert that a PNG header and a UTF-16 file without a mark are refused, and that a text file named `data.bin` is read.

**Confidence.** High. Four harnesses sniff content for NUL and control bytes, and the extension list adds nothing once the sniff is free.

### Turn a sibling conflict into a retry instruction and never retry inside Read

**Recommendation.** Map `VfsError::Conflict`, through the shared `refusal` function, to `{path} was changed, or is being changed, by another call in this round, so it cannot be read in this round. Call Read again in your next message.` Do not retry inside the tool and do not wait for earlier calls.

**Why.** Cursor retries a missing file once because its calls can run before a sibling Write lands. The retry releases the lock, waits up to 120,000 ms for earlier mutating calls, re-acquires the lock and reads again (bundle, daemon; `tool-read.md`, "Algorithm and ordering", step 9). Runvil's model differs. `ToolContext` documents that calls a model asks for in one round are concurrent with each other and that one call's write to a path conflicts with a sibling's read or write of it (source). `Access::gate` canonicalizes before it claims, so two spellings of a path are one claim key. The race therefore surfaces as `Conflict` and not as `NotFound`, and the model's next round is the retry. The lock dance has no counterpart, because the Pack holds no lock. A claim is a record in the VFS ledger that stays until the run's scope ends. The VFS decides a conflict by what each call has seen, so a sibling write from the same round conflicts with this read even after the write has finished (source, `claims.rs`). That is why the sentence says "in this round" and sends the retry to the next message. A retry inside Read would therefore still fail, which is a second reason to drop it. I assume a call made in a later round has seen the claims of earlier rounds. This differs from `tool-read.md`, Decision 10, which keeps the retry at high confidence. I judge the retry targets a race that Runvil reports as a conflict, and I have not run a Write and a Read in one round to see it.

**Evidence.** None of the other read paths retries a missing file. rig orders a batch by declared footprint, so a read waits behind an earlier write of an overlapping path. pi's write and edit queue on the real path, and its read never joins the queue (source; `tool-read.md`, Decision 10 and "Q9 Concurrency"). The `VfsError::Conflict` documentation says two overlapping claims conflict when at least one is a write and they are concurrent (source). The sibling reports reach the same shape for their own tools: refuse, and do not wait (`master-str-replace.md`, behavior 7; `master-write.md`, "Leave concurrent calls to the claims system and report the conflict"). Cursor's lock is keyed on the raw argument string, so `./foo.ts` and `foo.ts` do not serialize (bundle).

**Without it.** Passing the raw text, `conflicting claim: <detail>`, shows the model claim identities it cannot use, and the model may decide the file is unreadable. Copying Cursor's retry could add a wait of up to 120,000 ms on a task that polls every effect.

**Test.** In one round, issue a Write and a Read of the same new path. Assert that the Read fails with the retry text above, byte for byte, and never answers `NotFound`. Call Read again in the next round and assert success. Issue two Reads of one path in one round and assert that both succeed.

**Confidence.** Medium. The concurrency model is documented in `ToolContext`, but the Conflict path for a Read is untested.

### Validate arguments in the body: integers, a limit of at least 1, no undeclared parameters

**Recommendation.** Change `offset` and `limit` in `read.json` from `number` to `integer`, so the shared check refuses `1.5` and names the parameter. In the body, refuse a `limit` below 1. Refuse any parameter other than `path`, `offset` and `limit`, naming it and listing the three allowed, with the text `` std_Read: `{name}` is not a parameter of this tool. The parameters are `path`, `offset` and `limit`. ``, which the shared helper in `check.rs` (`refuse_unknown`) builds for every tool in the same shape. Every argument error in this tool has the shape `` std_Read: `{name}` {problem} ``, as in the `limit` refusal of the window behavior. Read a `null` value as absent. Do not coerce numeric strings.

**Why.** The shared check honors only `type`, `properties`, `required`, `enum` and `items`, so `number` accepts a fraction, and a `limit` of 0 passes unless the body refuses it (source). Undeclared parameters pass through by design. A model that sends `start_line`, `end_line`, `file_path` or `line_range`, the names Zed, Gemini CLI, Claude Code and Cursor's `haiku` branch use, would otherwise read the whole file with its range ignored, a silent wrong result for accuracy. rig refuses unknown keys for that reason. Cursor coerces `"12"` to 12; I do not, because the shared check already refuses a string with a message that names the value, and a coercion in one tool would break the contract of the shared check. That costs a round when a model sends a string, so my confidence in that one part is low.

**Evidence.** In Cursor `offset: 0` is accepted and rewritten to 1, `limit: 0` is refused, `"12"` is coerced, `"abc"` fails, and `1e20` is clamped to 2147483647 (bundle; `tool-read.md`, "Validation quirks, exactly"). rig sets `deny_unknown_fields`, so a misspelled `file_path` fails (source). zeroclaw silently ignores an `offset` that is negative, a float or a string (source, probe). Seven harnesses use `offset` and `limit`: Cursor, Claude Code, Copilot v2, pi, rig, zeroclaw and OpenCode. Zed and Gemini CLI use `start_line` and an inclusive `end_line`, Cline puts both in each entry of a `files` array, and Cursor's `haiku` branch takes a `line_range` array (`tool-read.md`, "Across all the harnesses").

**Without it.** `offset: 1.5` reaches the body and the body must guess. A `limit` of 0 renders no lines and still counts as a read, which is rig's behavior. A call with `line_range: [10, 20]` is ignored and returns the file from line 1, and only a model that checks the numbers will notice.

**Test.** Assert that `offset: 1.5` fails naming `offset`, that `limit: 0` fails, and that `"offset": "12"` fails with the shared check's message naming the value. Assert that `line_range: [10, 20]` answers `` std_Read: `line_range` is not a parameter of this tool. The parameters are `path`, `offset` and `limit`. `` byte for byte. Assert that `{"path": "/a", "offset": null}` reads the whole file.

**Confidence.** Medium. The integer and limit rules are Cursor's own and the check's gap is observed, but only rig refuses undeclared parameters.

### State every limit in the description and pin the text with a test

**Recommendation.** Edit `read.json` so each sentence is true. Drop `Any lines longer than 2000 characters will be truncated`. Replace the `cat -n` sentence with the real format and say that the prefix is not part of the line. State the 100,000-byte cap and the trailer, the 16777216-byte ceiling, and that binary files are refused. State that Read opens any path the call names, including paths that Glob and Grep leave out, and that a path naming a symbolic link is read through to the file it points to unless the link leaves the run's filesystem. Keep the negative-offset sentence and the empty-file sentence. Add a spec test that asserts each enforced constant appears in the description text.

**Why.** A model plans around what the description promises. Today it promises `cat -n` and a 2,000-character cut that Cursor never did and Runvil will not do (`tool-read.md`, "What the stub gets wrong today"). A description that names a limit the code does not enforce, or the reverse, leaves holes the model cannot see. Cline's description promised about 47k characters and a later stage cut 8,000, so every long read lost its middle (source, probe). Gemini CLI builds its description from the constants it enforces and pins the rendered text with a snapshot test, so contract and enforcement cannot drift (source, probe). The plan's definition test already refuses descriptions that promise PDFs, and `schemas/EDITS.md` records each edit (source).

**Evidence.** The shipped `read.json` says `Any lines longer than 2000 characters will be truncated` and `Results are returned using cat -n format, with line numbers starting at 1` (source). Cursor's `latest` description says `Lines in the output are numbered starting at 1, using following format: LINE_NUMBER|LINE_CONTENT`, with the article missing, and a leaked prompt from November 2025 has the same words (bundle, web; `tool-read.md`, "Description text, verbatim, `latest`"). Cursor's description never states the cap. A model learns it from the error, after the call.

**Without it.** A model that expects a tab after the number writes anchors with a tab. A model that believes long lines are cut distrusts a long line it can read whole. A model that does not know the cap discovers it only from the trailer.

**Test.** Assert that the description contains `100000` and `16777216` as plain digits and a six-wide example, and that it does not contain `cat -n`, `2000` or `16 MiB`. Better, build the description string from the constants and assert that the stored JSON equals it.

**Confidence.** High. The false lines are observed in the shipped file, and Gemini CLI shows that pinning works.

### Publish a read record to a per-run service, and keep the render pure

**Recommendation.** After a successful read, write one entry to a per-run read record, keyed by the canonical path. The record lives in a run service whose key `plugin-std` declares and whose provider the Host supplies for each run, beside the input broker. The Pack lists the key in `needs`, so a run without the provider cannot use the Pack and Read never meets a missing service. The service is already per run, so key each entry by the canonical path alone. The entry holds the SHA-256 of the file bytes the read returned, the first and last line shown, the line total, and `whole`, which is true only when the read showed lines 1 to the total with no cut. A file that decodes to no text, whether it holds zero bytes or only a byte order mark, still gets an entry: the hash of its raw bytes, lines 0 to 0, total 0 and `whole` true. After a successful `Write`, that tool replaces the entry with the hash of the bytes it wrote, `whole` true, lines 1 to the new total (0 to 0 when the new total is 0) and that total. After a successful `StrReplace`, that tool replaces only the hash and keeps the other fields, so an edit never turns a partial view into a whole one. When no entry exists for the path, a successful `StrReplace` creates none.

The `read-real` todo ships Read without the record and without the `needs` entry. The `write-real` todo adds the key, the `needs` entry, the Host provider in `crates/workshop/agents/src/conversation-run.rs` and the `pack-tests.rs` change in one commit, and Read writes the record from then on. Write the formatter as a pure function from bytes, arguments and constants to text, shown range and total, and take the entry from its result. Do not gate anything inside Read.

**Why.** The write report gates an overwrite on a record that the last read of the path covered the whole file and still matches the disk, and the str-replace report diagnoses a stale edit against the same kind of record (`master-write.md`, "Refuse to overwrite an existing file the run has not read whole and current"; `master-str-replace.md`, behavior 9). Both name Read as the writer, so Read owns this half of the contract. The gate protects files from a rewrite made from a partial or stale view, which serves file safety. The `whole` flag matters because Claude Code's partial-view rule shows the failure: a `limit: 50` read of a 200-line file must not satisfy a later write. rig's hash is recorded even for a read that showed nothing, so the entry here also holds the shown range and refuses to call a partial read whole. The record cannot live in the Pack, because `Std` is shared by every run and a ledger there would leak state between runs (source). `ToolOutput` holds text and a trust level only, so a run service is the channel. A pure render keeps the output the same for the same inputs, which serves replay. A file whose rendered text passes the cap can never be recorded whole, and the write report accepts that limit.

**Evidence.** Claude Code gates writes on an LRU of content hashes with a 5,000 ms window and a partial-view flag. rig publishes `FilesRead { path, hash, lines }` and compare-and-swaps in `edit`, but its gate is satisfied by a read that showed nothing (`limit: 0`, an offset past the end, or a cut read). Zed keeps a file's mtime and uses it to annotate a failed edit. pi, Gemini CLI and OpenCode keep no read state. Cline rewrites older reads in the transcript, and Aider re-reads its files at send time (source, probe; `tool-read.md`, Decision 11). Cursor records only a set of paths keyed by the resolved path, with no hash (bundle). `ToolContext::service` lends a run's services, and the Pack declares no `needs` today (source). `PackInfo::needs` says that a run without every listed service cannot use the Pack, and `ConversationRun::run_services` supplies only the input broker today, so the Host change is part of the cost (source). `sha2` is a workspace dependency that `plugin-std` does not list (source). The `Access` facade exposes no canonical-path function (`public-api.txt`, source), so `Access` gains `canonical_path(path) -> Result<String, VfsError>`, which canonicalizes against the access root with no policy check, claim or event. No Pack normalizes a path itself.

**Without it.** The write gate cannot tell a window of 200 lines from a whole 900-line file, and a model that read part of a file rewrites all of it from that view and drops the rest.

**Test.** Read a three-line file whole and assert an entry with its SHA-256, lines 1 to 3, total 3 and `whole` true. Read it with `limit: 2` and assert `whole` false. Read a file in a capped page and assert `whole` false. Read an empty file and a file of only a byte order mark, and assert an entry with total 0 and `whole` true for each. After a read with `limit: 2` of a five-line file, run a successful `StrReplace` on line 1 and assert that the hash changed and `whole` is still false. Fail a read, or refuse a binary file, and assert no entry. Run a successful `StrReplace` on a path with no entry and assert that no entry appears. After a `Write` of empty contents, assert that the entry equals the one Read records for an empty file. Build a run with no provider for the key and assert that the Host refuses the Pack and names the service. Read one path under two spellings and assert one entry. Run two runs against one `Std` and assert separate records. Call the render function twice with the same inputs and assert equal text. The record tests start with the `write-real` commit, because `read-real` ships without the service.

**Confidence.** Medium. Two sibling reports and two harnesses (Claude Code, rig) converge on the record, but the service key and the `needs` entry do not exist yet, and no transcript shows how often a gate blocks a run.

## Where Runvil should differ from Cursor

Runvil should differ from Cursor in eighteen places. Table 11 lists them, and several rows drop machinery that exists only because of JavaScript, the proto layer or Cursor's per-model routing.

| What Cursor does | What Runvil should do | Why |
|---|---|---|
| Names the tool `read_file`, `ReadFile`, `Read` or `ViewImage` by prompt version, and names the path `target_file` on one branch | One wire name, `Read`, and one `path` parameter | the Pack has one wire name; the variants serve model families Runvil does not route (`tool-read.md`, "Tool name, per branch") |
| Accepts `line_range` on the `haiku` branch, `include_line_numbers` behind a flag, and `machineId` when machines are registered | Accept `path`, `offset` and `limit` only, and refuse the rest by name | one parameter shape; the shared check lets undeclared parameters through, so the body refuses them |
| Converts PDFs to text, returns images as content parts, and renders `.ipynb` cells | Read text only and refuse binary | the std plan lists images and PDFs as a non-goal; `ToolOutput` holds text |
| Stores large results as blobs and returns a record with `exceededLimit`, `isEmpty` and `readRange` flags | Return one text result, with failures as `ToolError` | the record exists for proto transport; a renderer that tests content before the flag prints `File is empty.` for a 200 KB file (`tool-read.md`, "Emptiness is a flag on the record, not a property of the content") |
| Clamps numbers to 32 bits and coerces numeric strings with `lenientNumber` | Refuse non-integers and strings with the shared check's message, and refuse an integer outside the 64-bit range by name | the clamp is a JavaScript number limit, and coercion in one tool would break the shared check's contract |
| Rewrites `~/.cursor/worktries/` to `worktrees` with a regex | Do nothing | it repairs a typo in a Cursor-specific path |
| Has three line-number styles: the default, a GPT-5 `L<n>:` style, and a sparse style; a fourth, `gpt5CodexCatN`, is unreachable | One style | the others serve models Runvil does not route, and the fourth is dead code in 3.27.12 (`tool-read.md`, "The three line-number styles") |
| Strips `\r` only when at least 5% of line endings are CRLF | Strip one `\r` from every line | nine harnesses hide CRLF, and the vote leaks `\r` below 5% |
| Counts newlines plus one, so a final newline adds a numbered blank line | Drop the empty element after a final newline | the total then matches what the file holds and agrees with `str::lines` and pi's fix |
| Errors when the window passes 100,000 UTF-16 units, counted before the prefixes | Cut on a line boundary at 100,000 bytes of rendered output and name the next offset | six of nine other tools cut and mark; bytes need no second pass |
| Prints head and tail notices, `... N lines not shown ...` | Print one trailer on the last line | the line numbers already show the head, and one trailer names the next offset |
| Refuses sixteen extensions before reading | Sniff content only | the sniff is free once the bytes are read |
| Reads the whole file and silently cuts at 8 MiB | Read at most 16 MiB plus one byte and refuse above | an unbounded read blocks the one polling task; a silent cut hides loss |
| Keys a lock on the raw argument and retries a miss after a wait of up to 120,000 ms | Rely on claims keyed on the canonical path, and translate `Conflict` | `Access::gate` canonicalizes first, and the ledger decides a conflict by what each call has seen |
| Detects encodings with `jschardet` and rescues with Latin-1, reporting nothing | Decode lossily and add a note | cheaper, and the model learns that bytes changed |
| Omits the path from `File not found` and `Permission denied`, and prints `(1 lines)` | Name the path in every failure and use the right plural | the omission is observed in the bundle and costs nothing to fix |
| Names `grep` in the cap message and mixes characters with bytes (`The file is 300003 bytes` beside `100000 characters`) | Name no other tool in a failure message, and count bytes everywhere | a prompt may offer `Read` alone, and one unit cannot contradict itself |
| Records the resolved path of every read in `readPaths`, with no hash | Record a hash, the shown range, the total and a whole-file flag in a per-run service | the write and str-replace reports consume it, and a path set cannot say whether a read was whole |

*Table 11. Eighteen differences between Cursor's Read and the recommended Runvil Read.*

Confidence for each row is the confidence of the behavior it names.

## Mapping to Runvil

Table 12 maps each behavior to the `Access` method that covers it. Two methods are missing, `Access::read_range` and `Access::canonical_path`, and everything else is a pure function in the Pack, a message mapping or the run service for the record. Vinnie owns every row. Every next step is a todo id, from the standing plan or proposed here (`refusal-text`).

| Behavior | `Access` method | Status | What the gap needs | Owner | Next step |
|---|---|---|---|---|---|
| Number every line | none | missing | a pure formatter in the Pack | Vinnie | `read-real` |
| Stop at the byte cap | none | missing | a pure function over the window and the cap constant, and `clip_to_boundary` in the shared `text` module for the first-line cut, so no cut splits a code point | Vinnie | `read-real` |
| Bound the read | `stat`, and `read_range` below `Access` | partial | `stat` is covered; `Access::read_range` is missing, so add it with the four steps of `read` and regenerate `crates/runvil/public-api.txt` | Vinnie | `read-real`, first commit |
| Read a window | none | missing | a pure window function over `i64` offsets | Vinnie | `read-real` |
| Name the path in every failure | `VfsError` variants | partial | the shared `refusal` function builds the texts for `NotFound`, `IsADirectory`, `PermissionDenied`, `ApprovalRequired`, `OutsideRoot`, `Io` and `Conflict`; the sentences come from the shared `refusal-text` module, and real approval waits on the `approval` todo | Vinnie | `refusal-text`, `read-real` |
| Report the same line total | none | missing | one split used by numbering, paging and the offset error, in the shared `text` module | Vinnie | `read-real` |
| Strip one carriage return | none | missing | a pure function | Vinnie | `read-real` |
| Decode lossily and say so | `read` | covered | `String::from_utf8_lossy`, `<[u8]>::utf8_chunks` and `String::from_utf16_lossy`; no new dependency; in the shared `text` module | Vinnie | `read-real` |
| Refuse binary content | `read` | covered | a pure function over the first 8,192 bytes, in the shared `text` module | Vinnie | `read-real` |
| Translate a sibling conflict | claims inside `read` | covered | the shared `refusal` function maps `VfsError::Conflict` to the retry sentence | Vinnie | `refusal-text`, `read-real` |
| Validate arguments | the shared check in `check.rs` | partial | change `number` to `integer` in `read.json`; refuse `limit` below 1 in the body, and undeclared parameters with the shared `refuse_unknown` helper, so every argument error has the `std_Read:` shape | Vinnie | `read-real` |
| State and pin the limits | `schemas/read.json`, `spec-tests.rs` | partial | the description edits below and a test that pins the constants | Vinnie | `read-real` |
| Publish a read record | `ToolContext::service`, plus `Access::canonical_path` for the key | missing | a service key that `plugin-std` declares, a `needs` entry on the Pack, a Host provider, a `sha2` dependency, and a canonical path (`Access::canonical_path`, added in the first `read-real` commit with `Access::read_range`) | Vinnie | `read-real` for the path method, then `write-real` for the service, the `needs` entry and the provider, then `replace-real` |
| Mark output untrusted | `ToolOutput::untrusted` | covered | return `untrusted` for every read that embeds file text | Vinnie | `read-real` |

*Table 12. Mapping from behavior to `Access` method, gap, owner and todo.*

The standing plan stays as it is, and the plan edits below are proposals. The `read-real` todo text in the std plan needs one amendment. It says `the 100,000-character cap with its exact error` and `the not-shown markers`. Replace both with the 100,000-byte cap that cuts on a line boundary and ends with a trailer that names the next offset, and drop the head and tail markers. Four more edits follow from the rulings in this report:

- Add the todo `refusal-text` before `read-real`. It writes one function from `VfsError` to a model-readable sentence, with a `Display` arm for any variant it does not name, and every tool uses it. The `approval` todo then only wires real approval.
- Add `Access::canonical_path` to the first `read-real` commit, beside `Access::read_range`, and let the plan's non-goal about `crates/runvil/public-api.txt` allow these `Access` additions, each regenerating the file in its own commit.
- Say that `read-real` ships Read without the record and without `needs`, and that `write-real` adds the key, the `needs` entry, the Host provider and the `pack-tests.rs` change in one commit. Let the dependency line allow `sha2` from the workspace for the record.
- Order the todos `refusal-text`, `read-real`, `write-real`, `replace-real`, so the record and the canonical path exist before the tools that consume them.

Confidence in these plan edits is high, because they only make the plan text match the behaviors above. The new todo id `refusal-text` is a proposal, with medium confidence.

Tests to add to the crate:

- Unit tests beside the render module for numbering, the window table, the cap and the paging rebuild, the cut on a character boundary, the line total, carriage returns, decoding and the binary sniff. The Rust model I wrote for this report passes 84 such checks, and its cases port directly.
- A VFS test for `Access::read_range`: a read policy that denies `Op::Read` denies it, it takes a read claim, it fires the read event, and a counting backend sees `len` of 16,777,217.
- A VFS test for `Access::canonical_path`: two spellings of one path give one canonical path, and a policy that denies `Op::Read` does not refuse it, because it runs no policy check, claim or event.
- An argument test for each rule in the validation behavior, and a description test that pins the constants.
- An integration test in `tests/it/tools.rs` that flips `Read` from the stub text to real output, and a second test that issues a Write and a Read of one path in one round and asserts that the Read gets the conflict sentence and never `NotFound`.
- A trust test: every successful read returns `OutputTrust::Untrusted`.

Schema and description fixes, all in `schemas/read.json` with a matching entry in `schemas/EDITS.md`: change `offset` and `limit` to `integer`, and give `limit` the sentence `Must be at least 1`. In the description, keep the bullet about offset and limit, the batching bullet and the empty-file bullet. Delete the `Any lines longer than 2000 characters` bullet and the `cat -n` bullet, and add these five:

```text
- Each line is numbered from 1 in the form LINE_NUMBER|LINE_CONTENT, with the number right-aligned in six characters, as in "     1|text". The number and the pipe are not part of the line.
- Output stops on a line boundary after 100000 bytes. A page that stops early ends with a line that says what was left out and, when more lines follow, the offset to continue from.
- Files larger than 16777216 bytes and binary files are refused.
- Read opens any path the call names, including paths that Glob and Grep leave out.
- A path that names a symbolic link is read through to the file it points to. A link that leaves the run's filesystem is refused.
```

## Risks and open limits

Table 13 lists what the evidence does not settle. The likelihood column gives the chance that the event in the first column happens in real use, on the fixed ladder. The confidence column rates the evidence behind that likelihood. None of the twelve harnesses ships a transcript or benchmark of how models cope with its read format, so no row rises above medium confidence (`tool-read.md`, "What remains unknowable").

| Open limit | Likelihood | Confidence in the evidence | What would change the recommendation |
|---|---|---|---|
| A model treats a capped page as the whole file | roughly even chance | low | Claude Code added `Do NOT answer from this page alone`, which suggests the failure was seen; if run logs show it, add that sentence to the trailer |
| 100,000 bytes is the wrong cap for real runs | likely | low | measure how many reads reach the trailer; move toward 51,200 bytes (pi, rig, OpenCode) if pages are too large, or up if models page too often |
| The hidden final newline lets Write drop a file's last newline | roughly even chance | medium | the write report decides; if Write rewrites whole files from model text, add a `[No newline at end of file]` marker to Read |
| A symlink to a FIFO or device blocks the polling task | very unlikely | low | untested, because `stat` does not follow links; test it, and if it blocks, refuse links whose target type cannot be known |
| One line over the cap hides the rest of a minified file | unlikely | medium | a run that needs it; add a byte-range parameter or return the match from Grep; Cursor has the same dead end |
| The first-line cut separates a combining mark or a joined emoji from its base, because the cut protects code points and not grapheme clusters | very unlikely | medium | it applies only to a single line longer than the whole cap; if it shows up in real output, add a grapheme-aware step back to `clip_to_boundary` |
| A Write and a Read in one round behave differently than documented | unlikely | low | the integration test decides; if the Read does not get `Conflict`, the identities are not forked together, and the Harness orders the batch |
| The 16 MiB ceiling blocks a real task such as a large log | unlikely | low | add the streaming window scan from Table 4 |
| A model sees two numbered formats in one run, this tool's and the Lua helper's | unlikely | low | align `store.read_numbered` with the tool, or document the difference |
| `read-real` ships before the run service that holds the read record exists | likely | medium | ship Read without the record and without `needs`; `write-real` adds the key, the `needs` entry, the Host provider in `crates/workshop/agents/src/conversation-run.rs` and the `pack-tests.rs` change in one commit; the pure render makes that a small change |
| A model sends numeric strings for `offset` or `limit` and loses a round | roughly even chance | low | Cursor built `lenientNumber` for it, which suggests the case occurs; if logs show it, add coercion to the shared check |
| A custom policy refuses a read with a reason that omits the path | unlikely | low | the shared `refusal` function passes the policy's reason through, so a reason without the path gives a refusal without it; have the function append the argument path when the reason lacks it |
| A Harness stage cuts tool output after Read, as Cline's request builder does | very unlikely | low | my grep of the Harness found no such cut, which is a negative search; the trailer-survival test settles it |
| The maintainers refuse a new `Access` method | unlikely | medium | fall back to a `stat` ceiling for regular files, accept the symlink gap, and keep the check in the Pack |

*Table 13. Open limits, with likelihood and confidence kept apart.*

Three findings would change the main recommendation. A measurement showing that models read plain text as accurately as numbered text would remove the prefix and its 7 bytes per line. A run log showing that models act on capped pages as whole files would move the cap from a trailer toward an error, or toward Claude Code's stronger wording. A decision by the write and str-replace reports to drop their read record would return Read to a stateless render.

I found these limits in the evidence report and did not resolve them. The service below Cursor's tool boundary varies by bundle, so the exact decode ladder in other builds is unknown. Claude Code's default line count lives outside the extracted region, so its 2,000-line figure is the parent spec's claim and not an observation. The Zed grammar race that returns a raw 1 KiB preview has no test. I did not run a Write and a Read in one round on Runvil, and I did not open a FIFO through a link. A `Conflict` that the Pack maps to a `ToolError` reaches the model as failure text and does not end the run (`scheduler/tool_call.rs`, read from source, not run).

## Sources

Tier labels follow the section on criteria. Facts about the twelve harnesses and about Cursor's internals come from the evidence report and are cited by its section name; the entries below say which of them I re-read myself.

**Evidence reports.**

- `tool-read.md`, the primary ground, cited by section name throughout.
- `file-tools-index.md`, for the shared line-number format, the lock ordering, the two ripgrep builders and the corrections to the parent spec.

**Cursor 3.27.12 on Windows.**

- `cursor-agent-exec/dist/main.js`, 9,814,287 bytes, under `C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\`. `(bundle)`. I re-checked the line-number format string, the `LINE_NUMBER|LINE_CONTENT` description line, the cap message, the bare `File not found` and `Permission denied` throws, and the offset message. `(probe)`
- `cursor-agent-host/dist/agent-host-daemon/dist/bin/daemon.cjs`, 20,213,626 bytes, same parent folder. `(daemon)`. I re-checked `computeReadSlice`, `countNewlines`, the 5% CRLF vote, `MAX_TEXT_SIZE` and its silent `substring` cut. `(probe)`
- A leaked Cursor prompt of 2025-11-01 at `github.com/EliFuzz/awesome-system-prompts/blob/main/leaks/cursor/2025-11-01_prompt_agent.md`, used only through the evidence report for the missing article in `using following format`. `(web)`

**Harness sources, as the evidence report cites them.** I re-read the constants marked here, and the rest I took from the report.

- Claude Code 2.1.311, the `Read` region of `claude.exe` (win32-x64, 298,870,432 bytes). `(source)`
- VS Code Copilot, `extensions/copilot/src/extension/tools/node/readFileTool.tsx` in `microsoft/vscode` at commit `b208bc2501` (2026-10-25), Copilot Chat 0.72.0. I re-read lines 82, 83, 105, 716 and 720. `(source)`
- Zed, `crates/agent/src/tools/read_file_tool.rs` and `outline.rs` at commit `37faef81f7` (2026-10-27). I re-read the `cat -n` doc comment. `(source)`
- pi, `packages/coding-agent/src/core/tools/read.ts` and `truncate.ts` at commit `a92283056` (2026-10-27), and the counting fix at commit `f95306781` (2026-05-21). I re-read `truncate.ts` lines 11, 12 and 47. `(source)`
- rig, `crates/rig-tools/src/read.rs` at commit `a83b774d0` (2026-10-25). I re-read `MAX_LINE_CHARS`, `MIN_LINE_CHARS`, the empty-file and past-the-end texts. `(source)`
- zeroclaw, `crates/zeroclaw-runtime/src/tools/file_read.rs` at commit `2d85371f3` (2026-09-18). `(source)`
- Codex CLI at commit `bea47c06dfac`, with the removed `read_file` at commit `c6ffe9abab`. `(source)`, `(web)`
- Gemini CLI tag `v0.66.0` (commit `970413207732`), OpenCode tag `v1.19.5` (commit `810c0eba1a73`), Cline tag `v4.1.26` (commit `a459f1d`), Aider commit `5dc9490bb35f` and Goose commit `ee8500d5e060`. `(source)`, `(web)`

**Runvil, repository `runvil` at commit `08801150f` (2026-10-27), working tree clean.** `(source)`

- `crates/runvil-internal/vfs/src/handle/operations.rs`, `error.rs`, `stat.rs`, `traits/backend.rs` and `real.rs`, for what `Access` and `VfsAccess` do, the `VfsError` variants, `Stat`, `read_range` and the real backend's `read` and `read_range`.
- `crates/runvil-plugin/src/context.rs`, for the `ToolContext` statement that calls in one round are concurrent, and `crates/runvil-internal/types/src/tools/output.rs`, for `ToolOutput` and `ToolError`.
- `crates/plugin-std/src/check.rs`, `src/pack.rs`, `schemas/read.json` and `schemas/EDITS.md`, for the shared check, the stub and the shipped description.
- `crates/runvil-internal/lua/src/engine_globals.rs`, for `store.read_numbered`. `crates/runvil-internal/engine/src/execute/config-limits.rs` and `scheduler/tool_call.rs`, and `crates/harness-internal/runner/src/effect_loop.rs`, for the 24-round default, concurrent calls and blocking disk reads.
- `crates/runvil/public-api.txt`, for the absence of `Access::read_range`.
- `vibe/2026-10-27-2-plugin-std-file-tools.md`, the standing plan, for the todo ids and the crate's constraints.

**Workspace and method.**

- The workspace rule file `.cursor/rules/always.mdc` for the no-hard-wrap rule on markdown. `(source)`
- `tools-public/rulebooks/reports-rulebook.md` and `prose-rulebook.md`, for the report structure and prose passes.
- A standalone Rust model of the recommended behaviors, written for this report and run with `rustc` 1.99.1 on Windows. It passed 84 checks and its timings come from one machine with a warm cache. It is not in the repository. `(probe)`

*2026-10-27 - claude-sonnet-5-5*
