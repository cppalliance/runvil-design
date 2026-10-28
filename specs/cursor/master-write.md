# Guard Write against incomplete input, then let the VFS do the rest

## Summary

I recommend that Runvil's `Write` be one `Access::write` call behind four guards. The headline finding is a safety asymmetry in Cursor: `StrReplace` refuses a truncated read and `Write` never checks, so a cut read or a cut message replaces a whole file with a prefix and the model reads `Wrote contents to <path>`. The evidence report's decision D9 recommends one guard for each hole, and this report adopts both.

The decisions that matter most:

- The Engine fails every call in a message that stopped on the output limit.
- An overwrite of an existing non-empty file needs a whole-file `Read` whose bytes still match.
- The Pack leaves atomic replacement and parent creation to the VFS, which already does both. A policy-only `Access::check_write` runs first, so a mode refusal comes before any read.
- `Write` stores UTF-8 as sent, keeps a file's BOM and uniform CRLF, refuses a non-UTF-8 old file, and says whether it created or replaced.

The largest risk is the read gate. No transcript shows how often it blocks a run, and it needs a new run service. Confidence is high for the cut-message guard and for leaving atomicity to the VFS, and medium for the gate and the text rules. Next step, owner Vinnie: build `refusal-text`, `read-real` and `write-real` in that order, and add the independent todos `engine-cut-guard` (Engine hook), `vfs-write-mode` (backend mode copy) and `vfs-mode-text` (policy wording).

## Contents

- [Summary](#summary)
- [The recommendation in brief](#the-recommendation-in-brief)
- [Criteria](#criteria)
  - [Confidence and likelihood use two separate scales](#confidence-and-likelihood-use-two-separate-scales)
- [Options and the do-nothing case](#options-and-the-do-nothing-case)
  - [Incomplete input: add the host guard and the read-state gate](#incomplete-input-add-the-host-guard-and-the-read-state-gate)
  - [Atomic replacement: keep the Pack to one VFS call](#atomic-replacement-keep-the-pack-to-one-vfs-call)
  - [Text policy: write UTF-8 as sent and refuse an old file that is not UTF-8](#text-policy-write-utf-8-as-sent-and-refuse-an-old-file-that-is-not-utf-8)
  - [Result text: keep Cursor's sentence and add the one fact the model lacks](#result-text-keep-cursors-sentence-and-add-the-one-fact-the-model-lacks)
  - [Concurrency: leave it to the claims system](#concurrency-leave-it-to-the-claims-system)
- [Behaviors to implement](#behaviors-to-implement)
  - [Refuse any Write from a message that stopped on the output limit](#refuse-any-write-from-a-message-that-stopped-on-the-output-limit)
  - [Refuse to overwrite an existing file the run has not read whole and current](#refuse-to-overwrite-an-existing-file-the-run-has-not-read-whole-and-current)
  - [Write through one `Access::write` and let the VFS own atomicity and parent creation](#write-through-one-accesswrite-and-let-the-vfs-own-atomicity-and-parent-creation)
  - [Tell the model whether the call created or replaced, and how many lines it replaced](#tell-the-model-whether-the-call-created-or-replaced-and-how-many-lines-it-replaced)
  - [Refuse to rewrite a file that is not valid UTF-8, and write UTF-8 as sent](#refuse-to-rewrite-a-file-that-is-not-valid-utf-8-and-write-utf-8-as-sent)
  - [Keep a file's BOM and uniform CRLF, and use LF for a new file](#keep-a-files-bom-and-uniform-crlf-and-use-lf-for-a-new-file)
  - [Treat empty `contents` as a real write that empties the file](#treat-empty-contents-as-a-real-write-that-empties-the-file)
  - [Leave concurrent calls to the claims system and report the conflict](#leave-concurrent-calls-to-the-claims-system-and-report-the-conflict)
  - [Name the path and the next step in every refusal](#name-the-path-and-the-next-step-in-every-refusal)
  - [Make the description say what the tool enforces, and keep `contents`](#make-the-description-say-what-the-tool-enforces-and-keep-contents)
- [Where Runvil should differ from Cursor](#where-runvil-should-differ-from-cursor)
- [Mapping to Runvil](#mapping-to-runvil)
- [Risks and open limits](#risks-and-open-limits)
- [Sources](#sources)

## The recommendation in brief

Table 1 gives one row per behavior, in the order of the Behaviors section. Each row names the recommendation and its confidence.

| Behavior | Recommendation | Confidence |
|---|---|---|
| Refuse any Write from a message that stopped on the output limit | An Engine guard fails every call in a cut message before dispatch | High |
| Refuse to overwrite an existing file the run has not read whole and current | A read-record gate with a partial-view rule; new files and files with no text need no read | Medium |
| Write through one `Access::write` and let the VFS own atomicity and parent creation | `Access::check_write` first; no `mkdir`, temp file, or lock in the Pack; copy the old mode in the backend as a follow-up | High for the Pack, medium for `check_write` and the mode copy |
| Tell the model whether the call created or replaced, and how many lines it replaced | Keep the `Wrote contents to <path>` prefix and add one sentence | Medium |
| Refuse to rewrite a file that is not valid UTF-8, and write UTF-8 as sent | A refusal naming the offset, then no rewrite of the model's text | Medium |
| Keep a file's BOM and uniform CRLF, and use LF for a new file | Three line-ending rules and one BOM rule, in a pure text module | Medium |
| Treat empty `contents` as a real write that empties the file | Accept `""` and report the replaced line count | High |
| Leave concurrent calls to the claims system and report the conflict | No Pack lock; the shared `refusal` function maps `VfsError::Conflict` to a sentence | High |
| Name the path and the next step in every refusal | The shared `refusal` function writes one sentence per `VfsError`, the Pack one per guard; no path rewriting | High |
| Make the description say what the tool enforces, and keep `contents` | Rewrite `schemas/write.json` to match the shipped behavior | High |

*Table 1. The ten recommendations with their confidence.*

## Criteria

Five criteria judge every option and every behavior below. Each keeps one name through the report.

- **Loud failure.** The model reaches the file it wants in few calls, and no outcome is a silent wrong result. A refusal that names the cause and the next step passes. A success sentence over a lost or altered file fails.
- **File survival.** A write loses nothing the model did not choose to lose. Write has two cases. Old-content loss is the file the call replaces. New-content loss is the file the call writes arriving cut or altered.
- **Replay.** The same call against the same files yields the same bytes and the same reply on every host, so a recorded run replays. Anything that depends on the host platform, a clock, or a race fails.
- **Bounded cost.** Memory and tokens per call have a stated bound, and the model is told when a bound applies. One read of the old file passes. Four reads fail.
- **VFS fit.** The behavior works on Runvil's `Access` as it stands, the gap is named when it does not, and each policy is decided once in the layer that owns it: mode gates in the VFS policy, atomic replacement in the backend, text rules in the Pack.

When two criteria conflict, the order is File survival, then Loud failure, then Replay, then Bounded cost. VFS fit is a constraint rather than a preference. File survival ranks first because Write replaces a whole file in one call, so one wrong call loses more than any other file tool can.

### Confidence and likelihood use two separate scales

Every recommendation gets a confidence word: high, medium, or low. Confidence rates the evidence behind the recommendation. Likelihood rates an event and uses a fixed ladder: very unlikely, unlikely, roughly even chance, likely, very likely, almost certain. The two never share a sentence.

The evidence uses tier labels. `(bundle)` and `(daemon)` mean read out of Cursor 3.27.12's shipped agent bundle and its unminified daemon twin. `(probe)` means measured by running something. `(source)` means read from a pinned commit of another harness or of Runvil. `(web)` means a public document. One limit applies to every Cursor line-ending vector: [tool-write.md](tool-write.md) (section Engineering questions, Q4) says the vectors come from a literal transcription of the daemon source and not from Cursor itself, so they describe the algorithm and are not live measurements. Each claim below says whether it was observed, assumed, or judged.

## Options and the do-nothing case

Five decisions have real alternatives. Each gets one table that names the do-nothing option and marks the recommended one. Table 2 covers the decision that matters most.

### Incomplete input: add the host guard and the read-state gate

Cursor's `Write` checks nothing about whether its picture of the old file or its own `contents` is complete. `StrReplace` refuses a read flagged `truncated`; `Write`'s execute closure never mentions the flag (`(bundle)`, offset 4231080 in 3.27.12, per [tool-write.md](tool-write.md), section Algorithm and ordering). The evidence report's decision D9 recommends two guards, one for each hole. They are options C and E in Table 2.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| A. Do nothing (Cursor's `Write`) | Nobody | No state, no refused turn | A cut read or a cut message replaces the file with a prefix and reports `Wrote contents to <path>` | Set aside: fails File survival and Loud failure |
| B. Refuse a truncated read only (`StrReplace`'s rule) | Pack, reading one flag | Closes old-content loss from a capped read | Catches a page that `Read` cut at its cap and misses a window the model chose with `offset` and `limit` | Set aside alone; E contains it |
| C. Fail every call in a message that hit the output limit (pi, rig) | Engine, before dispatch | Closes new-content loss for every tool, not just Write | One retry, and complete sibling calls in the message fail too | **Adopt, with E** |
| D. Cap `contents` at a fixed size (Cline, 6,000 characters) | Tool wrapper | A cut becomes unlikely | Forces files into chunks, and a direct caller bypasses it | Set aside |
| E. Read-state gate with a partial-view rule (Claude Code) | Pack, with a run service that Read fills | One rule closes never-read, stale, and partial views | A refused turn, a new run service, and Read must publish a record | **Adopt, with C** |
| F. Compare-and-swap on the hash of the last observation (rig `write_if`) | Pack or VFS | No extra model turn | Protects only files the harness already saw; rig's own `write` never calls it | Fold the hash into E; set aside alone |
| G. Refuse placeholder text such as `rest of methods ...` (Gemini) | Pack | Catches elided code | Misses a plain cut; flags files that discuss placeholders | Set aside |
| H. Refuse every overwrite (Copilot's `create_file`) | Tool surface | The destructive case cannot happen | No whole-file replace; the model needs Delete then Write | Set aside; the fallback if E blocks too many runs |

*Table 2. Seven ways to handle an incomplete view or an incomplete message. The do-nothing option is A.*

Option E costs a turn when the model has not read the file, and it caps overwrites at files the model can read whole. Claude Code has the refused-turn cost. Option C costs one retry per cut message and fails complete sibling calls. Each cost is one failed call, and a replaced file may not be recoverable.

**Why B alone lost.** Cursor's flag exists because its read executor cuts a file at 8 MiB without telling anyone. Runvil's `Access::read` returns the whole file or an error, so a Pack that reads through `Access` never holds a truncated view. The partial view that matters in Runvil is the model's own: it read lines 1 to 200 of a 900-line file through a windowed `Read`, or it read a first page that `Read` cut at its 100,000-byte cap, and then writes a new whole file. Option E sees both, because the whole-file flag in the read record is false for each. Claude Code's `bA(entry)` predicate refuses a `Write` after a `limit: 50` read of a 200-line file for the same reason.

**Why F lost alone.** Rig's `write_if` compares the file against the hash of the last observation, so it protects only files the harness has already seen, and rig's own `write` never calls it (`workspace.rs:106-118`, `edit.rs:183-213`). The gate keeps the hash and adds a rule that refuses unseen files.

**Why D and G lost.** Both guess at a cut from the shape of the content. Cline's cap is a proxy the model works around by splitting files. Gemini's detector looks for one family of phrases and misses a message that simply stops. Option C reads the cause directly.

**Why H lost, and when it wins.** Copilot's `create_file` makes the destructive case impossible by refusing to overwrite a file with content (`File already exists. You must use an edit tool to modify it.`). With that rule, no `Write` can cause old-content loss. It also removes whole-file replacement, which forces a model to rewrite a 400-line file through Delete then Write. Option H becomes the better choice if the gate in E blocks too many runs, which the Risks section rates.

### Atomic replacement: keep the Pack to one VFS call

Cursor truncates the file in place and then runs fsync. Table 3 weighs four owners for the replacement step.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| A. One `Access::write`, nothing else (the do-nothing option) | VFS backend | `RealAccess::write` already writes a sibling temp file, runs `sync_all`, then renames; ancestors are created; the policy gates one path | The new inode loses the old mode, owner, and hard links (derived from reading the code, not run); no directory fsync | **Adopt** |
| B. Pack writes a temp file and renames it through `Access` | Pack | Independent of the backend | Plan mode refuses a temp path that does not end in `.md`, so the name needs care; once approval is wired, each mutation asks separately; the backend already does this | Set aside |
| C. Backend copies the old mode onto the temp before the rename (rig, Gemini) | VFS, in `real/files.rs` | Keeps `0755` on overwrite | One `set_permissions` call in a crate outside `plugin-std` | **Adopt as a follow-up** |
| D. Truncate in place, then fsync (Cursor) | Pack | Keeps mode, owner, inode | A crash between truncate and last write leaves a prefix | Set aside |

*Table 3. Four owners for atomic replacement. The do-nothing option is A, and here it is also the winner.*

The evidence report's decision D3 recommends option B at the Pack layer ("the temp-then-rename pattern is twenty lines"). That recommendation does not account for the backend's own temp-file write. `RealAccess::write` calls `atomic_write`, whose own doc says it writes "a sibling temp file plus rename, so a failed write leaves the destination unchanged and no temp file behind" (`(source)`, `crates/runvil-internal/vfs/src/real/files.rs`). The memory backend inserts into a map under a mutex. Writing the pattern again in the Pack would produce two temp files and add two mutations that the Mode policy gates separately.

**Why D lost.** Truncating in place keeps the inode, mode, and owner, and a crash between the truncate and the last write leaves a prefix of the new content with the old content gone. The backend's replace gives up the first property to avoid the second, which is the right trade for File survival.

**What the winner costs.** The new inode loses the mode, owner, and hard links of the old file, and option C repairs the mode. No directory fsync follows the rename in `atomic_write`, and rig has none either, while Cursor syncs the containing directory on a best-effort basis (`(bundle)`, `fC` at 9317565). The evidence report rates the directory sync as marginal for an agent harness (decision D3, option C).

### Text policy: write UTF-8 as sent and refuse an old file that is not UTF-8

Table 4 weighs six policies for the bytes `Write` puts on disk.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| A. Bytes as sent, no look at the old file (pi, rig, zeroclaw, Goose; the do-nothing option) | Nobody | Simplest | A CRLF file becomes LF, a BOM is dropped, and a non-UTF-8 file is destroyed, all reported as success | Set aside: fails File survival |
| B. UTF-8 as sent, keep the old BOM and uniform CRLF, refuse a non-UTF-8 old file | Pack text module | The UTF-8 files that Read can show survive a rewrite, and the rest fail loudly | Refuses a latin1 rewrite; a mixed-ending file flattens to LF | **Adopt** |
| C. Detect and preserve encodings (Cursor, Zed) | Pack, plus a detector and `encoding_rs` | UTF-16 and latin1 round trips | One test obligation per encoding; the model cannot rewrite faithfully text that Read decoded lossily | Set aside for the first version |
| D. Per-line CRLF merge (Cursor) | Pack | A mixed-ending file keeps each terminator | An LCS port with a 4,000,000-cell cap, a backfill pass, and a result that depends on a trailing newline | Set aside |
| E. One style for the whole file, taken from the old file (Gemini: any CRLF; Zed: first newline) | Pack | CRLF survives, one classifier | Three harnesses chose three dominance rules; a mixed file flattens | B is the strict form of this |
| F. Write the original bytes when `contents` equals what Read returned (evidence report, Q10 option 2) | Pack | A total round trip for unchanged text | The model rewrites numbered text and seldom echoes it unchanged | Set aside |

*Table 4. Six text policies. The do-nothing option is A.*

The evidence report recommends option C with high confidence for decision D1 and adds option F in its Q10 answer. Runvil's `Read` changes the answer. `master-read.md` reads bytes, decodes UTF-16 by its mark, decodes everything else as lossy UTF-8 with a note, and never returns `VfsError::NotUtf8` to the model ("Decode lossily, hide the byte order mark, and say when bytes were replaced"). A model therefore does see a latin1 or UTF-16 file, with each bad sequence shown as U+FFFD. The hash in the read record covers the raw file bytes and not the decoded text, so the gate in Table 2 passes for such a file, and the text check in option B is the only guard. The text the model sends back does not map to the old bytes unless an encoder guesses the encoding, and `master-read.md` defers detection partly for that reason. If a later `Read` detects legacy encodings, or `Write` learns to encode UTF-16, option C returns to the table.

**Why D and E lost.** The merge in option D needs an LCS port with a 4,000,000-cell cap and a backfill pass, and its result changes with a trailing newline: the same three lines get different terminators with and without a final LF (vectors from a transcription of the daemon source). Option E has no such quirk, but three harnesses chose three dominance rules (Cursor at least 5% CRLF, Gemini any CRLF, Zed the first newline in the first 1,000 bytes), so the rule is arbitrary. Option B takes the strict form: only a file whose every LF has a CR counts as CRLF, so one scan decides.

**Why F lost.** A model that rewrites a file seldom reproduces the text it read, and the text it read carried a six-wide line number and a pipe on every line, which is not file text, so the model had to strip them. The check would pass in rare cases and add a branch to test. This is a judgment with no harness measurement behind it, so the evidence report's high confidence for the option stands as the opposing view.

**What the winner costs.** Option B refuses a rewrite of any latin1 or UTF-16 file, which matters for older C and C++ source in latin1 that [tool-write.md](tool-write.md) names as common in this workspace's domain. `Read` shows such a file, lossily for latin1 and faithfully for UTF-16 with a mark, so the model can read a file that this tool then refuses to rewrite. It also flattens a mixed-ending file to the endings the model sent, and the reply says so.

### Result text: keep Cursor's sentence and add the one fact the model lacks

Cursor's reply is one flat sentence. Table 5 weighs five reply shapes.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| A. `Wrote contents to <path>` only (Cursor; the do-nothing option) | Pack | Matches the text models know | The model cannot tell a new file from a replaced one | Keep as the prefix |
| B. The sentence, plus `created` or `replaced` and the old line count (rig, Goose) | Pack, from the old bytes it already read | One fact the model cannot get without another call | A count to define and test | **Adopt** |
| C. A byte or line count of the new content | Pack | A number to check | Pi's count printed UTF-16 units as bytes and was removed in commit `e583b290a`; the model knows what it sent | Set aside |
| D. Return a diff (Gemini) | Pack | The model sees the change | Cost grows with file size; `StrReplace` exists for this | Set aside |
| E. A note when the call changed what the model sent (BOM kept, endings converted) | Pack | The one transformation the model cannot see | One conditional sentence | **Adopt inside B** |

*Table 5. Five reply shapes. The do-nothing option is A.*

**Why C and D lost.** A count of the new content tells the model what it already knows, and the unit is easy to get wrong: pi's reply printed UTF-16 code units under the label bytes until commit `e583b290a`, and `café` reported 4 bytes while 5 landed on disk (`(probe)`). A diff makes every write cost the size of the file, and `StrReplace` exists to show a change. Gemini returns one anyway, with a snippet around the change and the whole file for a new file.

**What the winner costs.** Option B adds one sentence of about six words per write, a line-count function to define and test, and a `stat` before the write. Its count is advisory, since the word `created` is decided before a write that another call could precede, a point Goose's `exists()` check shares.

### Concurrency: leave it to the claims system

Table 6 weighs three owners for ordering concurrent writes to one path.

| Option | Who does the work | What it provides | What it costs | Verdict |
|---|---|---|---|---|
| A. Nothing in the Pack; the VFS claims decide (the do-nothing option) | VFS | Sibling calls in one round conflict loudly; a call's own read then write does not | The model must send a conflicting call again in its next message | **Adopt** |
| B. A Pack mutex on a canonical path (pi's queue, Gemini's lock) | Pack | Serializes siblings instead of refusing them | The mutex sees only `Write` calls, while a claim also covers a sibling read, glob, grep, list or recursive delete; a blocked mutex stalls the one task that polls every call; serialization hides the conflict, and order depends on scheduling (pi's durable queue ran 133 of 200 trials out of call order, an `(probe)` measurement) | Set aside |
| C. Cursor's streaming lock keyed on the raw argument | Tool wrapper | Early serialization | `C:\a\b.txt` and `..\a\b.txt` are two keys for one file | Set aside |

*Table 6. Three concurrency owners. The do-nothing option is A.*

**Why B and C lost.** A Pack mutex would order `Write` calls only, so a sibling `Glob` or `Read` of the path would still meet a claim that the mutex knows nothing about, and the Pack would hold a second ordering rule beside the claims. Pi's queue shows how such a rule drifts: a new file reached through a junction alias or a case-different spelling overlapped. A blocking lock also sits badly with the plan's rule that a call must not block while polled. Cursor's raw-string key is the bug the evidence report names: two spellings of one path are two locks.

**What the winner costs.** Option A refuses the second of two sibling writes to one path, so a model that batched them must send one again in its next message. The refusal is deterministic, and its text says what to do.

## Behaviors to implement

Ten behaviors, ordered by importance. The first two close the safety asymmetry in the title: Cursor's `StrReplace` refuses a truncated read and its `Write` never checks. The next four set what the call does to the bytes and to the reply, and the last four fix the edges.

### Refuse any Write from a message that stopped on the output limit

**Recommendation.** Add a guard in the Engine, before it dispatches a tool-call batch, that fails every call in a model message whose finish reason says the output limit cut it off. The failure text says that the response hit the output limit, that the call did not run because its arguments may be cut, and tells the model to send it again with complete arguments, in smaller pieces if the content is long. The text names no other tool, because a prompt can offer a subset of the file tools. The Pack needs no code for this, because the Pack never sees the finish reason. The todo is `engine-cut-guard`, a proposal, and it is independent of the other todos.

**Why.** A cut message can arrive as arguments that parse and validate. The new file body is the whole argument of `Write`, so a prefix replaces the old file and the reply reads as success. That is new-content loss, and it breaks File survival and Loud failure together. The guard costs one retry. The Engine is the only layer that holds the finish reason, so the guard belongs there.

**Evidence.** Pi fails every tool call in a message with `stopReason === "length"` before lookup, validation, or execution, with the text `Tool call "write" was not executed: the response hit the output token limit, so its arguments may be truncated. Re-issue the tool call with complete arguments.` (`(source)`, pi at `a92283056`, `agent-loop.ts:274-279`, added in commit `351efc828` on 2026-07-07). Its code comment gives the reason: streamed arguments are finished by a salvage parser, so a cut message can produce a call whose arguments "parse and validate but are silently incomplete" (`agent-loop.ts:481-487`). Rig answers a cut call with an `invalid_args` result at spawn time, so the tool never runs, and its text tells the model to split the work "such as writing part of a file and adding the rest with edits" (`(source)`, rig at `a83b774d0`, `rig-ecs/src/turn.rs:1412-1431`, `rig-core/src/tool/feedback.rs:57-62`). Cursor's `Write` has no such check. Its wrapper runs a strict `JSON.parse` on the buffered arguments, which catches a cut that leaves the JSON open, and its loop retries an `OutputTokensLimitExceededError`, but neither inspects a call that survived (`(bundle)` `gbe` at 4154498; `(daemon)` at 7396447 and 11701259). Whether a server upstream closes partial JSON before the client sees it is not in the sources. In Runvil, the Engine's chat scheduler builds `ChatOutcome::ToolCalls(events)` with `finish_reason: served.finish_reason.clone()`, and the batch "resumes unexecuted; the shim dispatches each call" (`(source)`, `crates/runvil-internal/engine/src/execute/scheduler/chat.rs`, function `tool_calls`). Observed: the finish reason is present there as an `Option<String>`. Not verified: the string each provider uses for an output limit.

**Without it.** A model writes a 600-line module, stops at the output limit on line 340, and the salvaged call overwrites a working file with 340 lines. The reply says `Wrote contents to /src/module.rs`. The model moves on, and the loss surfaces at the next build.

**Test.** Feed the Engine a scripted model message with the output-limit finish reason and one `Write` call whose arguments validate. Assert that no `Write` runs, that the model sees one failure per call naming the cut, that no failure names another tool, and that the target file is byte-identical afterwards. Add a vector with a sibling `Read` in the same message and pin the outcome (pi fails the sibling too).

**Confidence.** High, because pi and rig built this guard independently and the cost is one retry.

### Refuse to overwrite an existing file the run has not read whole and current

**Recommendation.** Gate `Write` over an existing non-empty file on the read record that `Read` publishes (`master-read.md`, "Publish a read record to a per-run service, and keep the render pure"). The record holds a hash, a shown range, a total and a whole-file flag, and the gate uses two of them: the run's last `Read` of that path must have set the whole-file flag, and the hash of the file's current raw bytes must equal the recorded hash. The hash covers the raw file bytes and not the decoded text, so a file that `Read` decoded still matches the disk. A path that does not exist, or an existing file with no text after its byte order mark, needs no read. Look the entry up by the path that `Access::canonical_path` returns, so two spellings of one file share one record. The refusal is one sentence that names the path and tells the model to read the whole file and send the call again: `{path} has not been read in full since it last changed. Read the whole file, then send this call again.` This is the one failure message in the report that names a read, because the call cannot succeed without one. After a successful `Write`, replace the entry with the hash of the bytes written, the whole-file flag set, a shown range of every line (0 to 0 for an empty file) and the new total, as `master-read.md` specifies, so the model can write again without another read. Call `Access::check_write` before the lookup (behavior 3), then look up the record before reading the old bytes, so a refused call never loads a large file.

**Why.** Three failures share one cause: the model's picture of the file is wrong. It never read the file, it read part of it, or the file changed after it read. One rule catches all three, which serves File survival and Loud failure. A content hash keeps the verdict deterministic on every backend, where a timestamp rule would go blind on a backend that tracks no times (`Stat.modified` is an `Option<SystemTime>`). The same transcript and files give the same verdict, so Replay holds. The cost is a refused turn when the model skipped the read, and the refusal text names the fix. `Read` stops on a line boundary at 100,000 rendered bytes, prefixes and newlines counted, and sets the whole-file flag false for a cut read (`master-read.md`, "Stop at a 100,000-byte cap on a line boundary and name the next offset"). It also refuses a file over 16 MiB (`master-read.md`, "Bound the read with a type check and a 16 MiB ceiling"). No `Read` of a file whose numbered text passes the cap can satisfy the gate, so a whole-file rewrite of such a file is impossible; the model uses `StrReplace`, or `Delete` then `Write`.

**Evidence.** Claude Code is the one harness of twelve that enforces a gate. It keeps a read-state store, refuses with `File has not been read yet. Read it first before writing to it.`, and refuses a changed file with `File has been modified since read, either by the user or by a linter. Read it again before attempting to write it.` (`(source)`, `claude.exe` 2.1.311, `write.js`). Its partial-view predicate `bA(entry)` is false when `offset > 1`, when `isPartialView` is set, or when a `limit` did not cover the file, so a `limit: 50` read of a 200-line file does not satisfy a later `Write`. When the modification time moved, it still allows the write if the entry is a full view and the content hash matches the disk. A path-shape test, `O4t(path)`, exempts a file that does not exist. Cursor has no gate: two of its four description branches tell the model `you MUST use the read_file tool first` and nothing enforces it, and a file seeded from a shell and never read was overwritten with a plain success (`(bundle)` `(probe)`). OpenCode's description makes the same unenforced promise (`(source)`, v1.19.5). Rig's content-hash `read_before_edit` gate is an unshipped corpus plugin whose `write`-over-existing branch has no test (`read_before_edit/tests.rs:75,104`). Copilot avoids the question by refusing to overwrite a file that has content.

Runvil's shipped `schemas/write.json` already says `you MUST first read it before writing`, so today the promise is unenforced. Observed in the crate: `Std` holds only the tool list and is shared by every run, `ToolContext::service` lends the run's services, and the Pack declares no `needs` (`(source)`, `crates/plugin-std/src/pack.rs`, `crates/runvil-plugin/src/context.rs`). Judged: the record belongs in a required per-run service that `Read` fills. `plugin-std` declares the key, the Pack lists it in `needs`, and the Host supplies it for each run beside the input broker. Entries are keyed by canonical path alone, because the service is already per run. `read-real` ships without the record and without `needs`. `write-real` adds the key, the `needs` entry, the Host provider and the `pack-tests.rs` change in one commit, and `Read` writes the record from then on. The service is required, so the gate always has a record to consult and has no gate-off mode. `master-read.md` adopts the same record and has `Read` write it after each successful read. Its record behavior also assigns the post-write update: `Write` replaces the whole entry and a successful `StrReplace` replaces only the hash.

**Without it.** A model skims lines 1 to 50 of a 300-line config, writes a new version of the whole file from that view, and drops 250 lines. Or a formatter rewrites the file between the model's read and its write, and the write erases the formatter's change. In both cases the reply is the success sentence.

**Test.** Five vectors against a stub run service. A three-line file never read: `Write` fails, file unchanged. The same file read whole, then written: succeeds, and a second `Write` with no new read also succeeds. The same file read with a `limit` that covers two lines, then written: fails. The file read whole, then changed through another handle, then written: fails with the same text. A path that does not exist, an existing empty file, and a file holding only `EF BB BF`: all succeed with no read. The refusal reads byte for byte as the sentence above, and its path is the argument as given. Two spellings of one path share one record. After a `Write` of `""`, the entry equals the one `Read` records for an empty file.

**Confidence.** Medium, because one of twelve harnesses enforces a gate, no transcript measures how often it blocks a run, and the run service is new design.

### Write through one `Access::write` and let the VFS own atomicity and parent creation

**Recommendation.** Call `Access::check_write(path)` first, so a Plan or Ask refusal comes before the read gate and before any read. This is a new `Access` method that runs the policy only, with no claim and no event, and it lands in the first commit of `write-real`. Then call `Access::write(path, bytes)` once per `Write`. Do not call `mkdir`, do not write a temp file, do not hold a lock, and do not re-implement the plan-mode check. Add one follow-up in the VFS backend, the todo `vfs-write-mode`: copy the old file's mode onto the temp file before the rename.

**Why.** Each of those jobs already has an owner. The Mode policy runs in `gate`, and `Access::write` is the only call that meets it, because `stat`, `read` and `read_string` gate `Op::Stat` and `Op::Read`, which every mode allows. Without `check_write`, a Plan-mode `Write` to an existing unread `/src/a.rs` gets "read it first" from the gate, then reads, then gets the Plan refusal: two wasted rounds. With it, the Pack needs no plan-mode check and runs no diff. A refused write costs one `check_write` call, where Cursor's plan-mode throw fires after the read and after the diff, so a refused write there has already done both (`(bundle)` `Rbe` at 4163000). `Access::write` gates one path, the file, so a Plan-mode write of `/docs/new/plan.md` works without `mkdir`, while a separate `mkdir` of `/docs/new` is refused because the directory path does not end in `.md` (derived from `is_markdown` and `gate`). The backend replaces atomically and creates ancestors. That serves File survival and VFS fit, and it removes the Cursor failure where a crash between truncate and last write leaves a prefix.

**Evidence.** Observed in source: `RealAccess::write` checks writability, resolves the path, refuses a directory, calls `create_parent`, then `atomic_write`, whose doc says it writes "a sibling temp file plus rename, so a failed write leaves the destination unchanged and no temp file behind"; the steps are `File::create`, `write_all`, `sync_all`, `rename`, with cleanup on failure (`(source)`, `crates/runvil-internal/vfs/src/real.rs`, `real/files.rs`). `real/tests/atomicity.rs` pins that a failed write leaves the destination unchanged and no `.vfs-tmp-` file behind. The memory backend does `check_file_destination`, `create_ancestors`, then `files.insert` under one mutex, and `memory-tests.rs` test `write_creates_overwrites_and_materializes_ancestor_directories` has the comment "No mkdir was needed; the ancestors exist." The standing plan's `write-real` todo says "parent creation through `Access::mkdir`", and [tool-write.md](tool-write.md) (decision D3) recommends a Pack-level temp file; neither accounts for the backend's own write path. Nine harnesses truncate in place, Cursor among them; rig's `write_atomic` and Gemini's temp-and-rename are the two that replace atomically, and both copy permissions onto the temp before the rename (`(source)`; rig `rig-tools/src/fs.rs:82-121`, Gemini `fileSystemService.ts:35,53-138`). The gap, derived and not run: `atomic_write` never calls `set_permissions`, so an overwrite creates a new inode with the default mode. Cursor keeps the mode only by accident, because an in-place open ignores the mode argument for an existing file (`(probe)`: an existing `755` file stayed `755`, a new file became `644` under umask 0022).

**Without it.** A Pack that calls `mkdir` first fails every Plan-mode write into a new directory with a refusal about a directory that is not markdown. A Pack that writes its own temp file makes a second temp file beside the backend's, and Plan mode refuses any temp name that does not end in `.md`. A Pack that skips the mode follow-up turns an executable script into `0644` on its first overwrite.

**Test.** In Plan mode, `Write` of `/docs/new/plan.md` succeeds when `/docs` does not exist, and `Write` of `/docs/new/plan.rs` fails naming the Plan mode. In Ask mode, `Write` to a new path fails with the approval refusal and a recording backend sees no read. In Plan mode, `Write` of an existing unread `/src/a.rs` fails with the Plan refusal and not the gate's sentence, and the backend sees no read. On Unix, create a `0755` file, `Write` over it, and expect `0755`; mark this test ignored with a pointer to the VFS follow-up until the backend copies the mode.

**Confidence.** High that the Pack should not repeat atomicity or `mkdir`, because source and tests show the backend does both. Medium on the mode follow-up, because the loss is derived from reading and has not been run, and medium on `check_write`, because the method and its name are my proposal and no code calls it yet.

### Tell the model whether the call created or replaced, and how many lines it replaced

**Recommendation.** Keep `Wrote contents to <path>` as the start of the reply, end it with a period, and append one sentence: `Created a new file.`, `Replaced an empty file.`, or `Replaced a file of N lines.` Write `1 line` for one, through the shared `plural(n, noun)` helper in `plugin-std`. Drop a leading UTF-8 byte order mark first. If no text is left, the file counts as empty and the reply says `Replaced an empty file.` Otherwise count LF bytes, plus one when the text does not end in LF. Add `Created directory <parent>.` when the parent did not exist, and add one sentence for each transformation named in the BOM and line-ending behavior below. The path is the argument as given. Return the reply as trusted text, because it holds only the caller's path and counts the tool computed.

**Why.** After a write the model cannot learn whether it made a file or replaced one without another call. That is the one fact it cannot reconstruct, and it costs a stat and one count of bytes already in hand. It serves Loud failure: a model that believes it created a file will not tell the user it replaced one. A fixed unit serves Replay and keeps the number honest. The unit is the line total that `Read` reports, so `a\nb\n` and `a\nb` both count 2 and the number matches what the model saw in a trailer (`master-read.md`, "Report the same line total whether or not the file ends in a newline"). The new-directory sentence is a judgment: a mistyped directory name silently creates a new directory under Cursor, rig, and Goose alike, and the sentence makes the typo visible. Counting bytes rather than decoded characters works on any file. A file of only a byte order mark counts as empty because `Read` answers `File is empty.` for it, and the reply states the same fact.

**Evidence.** Cursor's model sees only `message`. The diff, the line counts, and both file contents exist in the structured record and never reach the model, because the renderer is `case"success":return JB(t.value.message??"",!1)` (`(bundle)` `ybe` at 4157158; reply built at 4157903). Five harnesses tell the model whether it created or replaced: Claude Code (`File created successfully at: <filePath>` against `The file <filePath> has been updated successfully.`), Gemini, rig, Goose, and OpenCode's V2 rewrite (`(source)`). Rig replies `Created x/y/new.txt.` or `Replaced two.txt, which had 2 lines.`, and its commit `e034438fa` (2026-10-17, "tool replies carry only what the model can't know") replaced an earlier `Wrote {N} bytes to {path}.`. Pi removed its byte count in commit `e583b290a` (2026-09-02) because the number was UTF-16 code units labeled bytes. Line counts differ by harness: rig counts `\n` bytes plus one for an unended last line, so three CR-ended lines count as 1 and bytes `FF 0A FE 0A` count as 2 (`(probe)`); Goose counts `str::lines()`, where `""` is 0, `"a\n"` is 1, and `"a\n\n"` is 2 (`(probe)`). Observed in Runvil: `VfsPath` canonicalizes dot segments on receipt and `Access` returns no resolved path (`(source)`, `real/resolve.rs`, `handle/operations.rs`), so a reply that echoes the argument can differ from the canonical path for a spelling such as `/a/./b.md`. The method `Access::canonical_path`, which `master-read.md` has the first `read-real` commit add, gives the Pack the canonical path for the record key, and the reply still echoes the argument. The standing plan's `write-real` todo says to answer "the exact `Wrote contents to <path>` success text"; that wording has to change.

**Without it.** A model asked to "create notes.md" overwrites an existing file, replies that it created the file, and the user learns otherwise from a diff. In the opposite case the model re-reads the file after every write to check what happened, which costs a round trip each time.

**Test.** New path: the reply starts with `Wrote contents to /n.md.` and says `Created a new file.` A three-line file ending in LF replaced: `Replaced a file of 3 lines.` The file `a\nb` with no final LF: 2 lines, the same as `a\nb\n`. A lone-CR file `a\rb\r`: `Replaced a file of 1 line.`, which pins the unit and the singular. An empty file replaced: `Replaced an empty file.` An old file of only `EF BB BF`: the reply says `Replaced an empty file.`, the same fact as `Read`'s `File is empty.`, and the gate asks for no read. A write into a missing directory: the reply adds `Created directory`. Every reply passes `starts_with("Wrote contents to ")`.

**Confidence.** Medium, because five harnesses report the fact and rig's own commit message supports it, but no measurement shows it changes model behavior.

### Refuse to rewrite a file that is not valid UTF-8, and write UTF-8 as sent

**Recommendation.** When the old file exists and is not valid UTF-8, refuse as soon as the old bytes are read and before any rule looks at them, with this sentence: `File is not valid UTF-8 text: {path}. The first bad byte is at offset {valid_up_to}. This tool rewrites UTF-8 text only, even when a read showed the file decoded.` The sentence is the `NotUtf8` arm of the shared `refusal` function, with "rewrites" where `StrReplace` says "edits", and `{path}` is the argument as given. It says "a read" and not the tool name, because a failure message names no other tool. Otherwise encode `contents` as UTF-8 with no normalization, no unescaping, no comment stripping, and no formatting.

**Why.** `Read` never fails on encoding. It decodes a UTF-16 file by its mark and any other non-UTF-8 file lossily, shows U+FFFD for each bad sequence, adds a note, and publishes a record for the file (`master-read.md`, "Decode lossily, hide the byte order mark, and say when bytes were replaced"). A model can therefore read a latin1 or UTF-16 file whole, pass the gate, and rewrite it from text that does not map back to the old bytes. A write that converts such a file to UTF-8 changes bytes the model never saw and reports success. A refusal costs one failed call and protects File survival. The record's hash covers the raw file bytes, so the gate matches the disk and passes, and this check is the only guard. Writing the bytes as sent serves Replay and Loud failure, because every rewrite that Gemini and Copilot add is a change the model is not told about. JSON strings cannot hold a lone surrogate, so `contents` is always valid Unicode and the encode cannot fail.

**Evidence.** Cursor detects and preserves: a 14-row detector over the first 8,192 bytes, an encoding set of utf8, utf8bom, utf16le, utf16be, utf32le, utf32be, latin1, about twenty windows-125x and iso-8859-x labels, and jschardet output above 0.7 confidence (`(daemon)` `getBufferForWrite` at 15433367, `getFormatForBuffer` at 15437700). Writing ASCII into a UTF-16LE file leaves a UTF-16LE file (`(probe)`). Its latin1 round trip depends on a splice that only works when the count of U+FFFD characters in the new text equals the count of undecodable sequences in the old file (`encodeUtf8RestoringUndecodableBytes` at 15436236). Pi, rig, zeroclaw, Goose, and Codex write UTF-8 verbatim, and a pi overwrite turns a UTF-16LE file into UTF-8 (`(source)` `(probe)`). Claude Code refuses legacy encodings, NUL bytes in the first 8 KB, and UTF-16 big-endian (`error-strings.json`), which is a refusal precedent. Zed alone preserves encodings in Rust, with `chardetng`, `encoding_rs`, and a hand-written UTF-16 encoder (`crates/file_content/src/file_content.rs:89-146`). Gemini runs `unescapeStringForGeminiBug` over `content` for some model families, and a probe turns the source text `console.log("a\nb");` into a line break; Copilot strips a leading `filepath:` comment (`createFileTool.tsx:111,116`). Observed in Runvil: `read_string` returns `VfsError::NotUtf8 { path, valid_up_to }`, displayed as `not UTF-8: {path}, valid up to byte {valid_up_to}`, which is the shape of the refusal here. `Read` does not use it: it reads bytes and decodes them as `master-read.md` says, so the model sees a file that this tool will not rewrite. [tool-write.md](tool-write.md) (decision D1) recommends preserving encodings with high confidence and does not assume a `Read` that decodes lossily.

**Without it.** A latin1 header with `é` stored as `E9` reaches the model as U+FFFD, and a rewrite from that view stores `EF BF BD` where `E9` stood, so the diff shows a change on every accented line. A UTF-16 file becomes UTF-8, and whatever expected UTF-16 stops reading it.

**Test.** With a stub record seeded to match the old bytes and set whole, an old file with bytes `E9 E8 EA` plus ASCII: `Write` fails with the sentence above, the message names the offset, and the bytes are unchanged. An old UTF-16LE file with a BOM: same. With no record, the same files fail at the gate and the bytes are unchanged. With a whole `Read` first, `Read` shows the text and publishes a record, the gate passes, the UTF-8 check refuses, and the bytes are unchanged. Assert that the refusal names no other tool. `contents` of `café` over a UTF-8 file writes `63 61 66 C3 A9`. `contents` holding a backslash then `n` as two characters is written as those two characters. `contents` starting with `// filepath: x` is written with that line intact.

**Confidence.** Medium, because the evidence report reaches the opposite answer for a `Read` that detects encodings, and a later encoder could round-trip the UTF-16 files that `Read` now decodes.

### Keep a file's BOM and uniform CRLF, and use LF for a new file

**Recommendation.** Apply three rules in order to the bytes to write. If `contents` contains `\r\n`, write it as sent, because the model chose its endings. Else if the old file has at least one LF and every LF is preceded by CR, convert each bare LF in `contents` to CRLF. Otherwise write as sent. Then, if the old file starts with `EF BB BF` and `contents` does not start with U+FEFF, prepend the BOM. A new file gets LF and no BOM. Add no final newline and drop none: the new file ends in a newline if and only if `contents` does. When a rule changes the bytes, say so in the reply, and when the old file mixed endings, say that the new file uses the endings sent.

**Why.** `Read` shows the model LF text with no UTF-8 mark, and it gives no way to tell whether the file ended in a newline. It strips one `\r` from every line, even in a file with mixed endings, drops the mark, and drops the empty element after a final newline (`master-read.md`, "Strip one trailing carriage return from every line", "Decode lossily, hide the byte order mark, and say when bytes were replaced" and "Report the same line total whether or not the file ends in a newline"). A model that rewrites a CRLF file from that view sends bare LF text and no BOM. Rewriting a CRLF file with LF text changes every line, which makes the diff unreadable, and a dropped BOM changes how other tools read the file. Both are File survival failures that a success sentence hides, so the rules above put the endings and the mark back. The final newline is the one thing the model cannot see and the tool does not restore: the model decides it by what it sends, and the description says so. Uniform-only classification can be checked by one scan and leaves the hard case, the mixed file, to a reported write as sent. A new file gets LF everywhere so the same call writes the same bytes on every host, which is Replay. Cursor's default for a new file is CRLF on Windows and LF elsewhere, and Cline, Aider, and Gemini also take the host's ending.

**Evidence.** Cursor applies four rules in order: content with `\r\n` stays untouched, a mixed file gets a per-line merge, a file classified CRLF (at least 5% of terminators) has bare LFs rewritten with `/(?<!\r)\n/g`, and anything else stays untouched (`(daemon)` `getBufferForWrite` at 15433367, `withExistingLineEndings` at 15428106). The merge runs a prefix pass, a suffix pass, an LCS under a 4,000,000-cell cap, and a backfill pass, and its result changes with a trailing newline: existing `aaR bbN ccR` with content `mmN nnN ooN` gives `mmR nnN ooR`, while `mmN nnN oo` gives `mmR nnR oo` (vectors from a transcription of the daemon source, not live measurements). Cursor keeps a BOM the file had and never adds one, because `Write` builds its arguments with no `encodingHint` (`(bundle)` `Rbe` at 4163000). OpenCode keeps the old BOM or one the new text starts with and emits at most one (`write.ts:48-49`). Gemini makes a whole file CRLF when it contains any `\r\n` (`textUtils.ts:93-97`), Zed follows the first newline in the first 1,000 bytes (`text.rs:3646-3661`), Claude Code always writes LF, Codex cannot write CRLF, and pi, rig, zeroclaw, Goose, and OpenCode write bytes as sent (`(source)`). A Cursor forum thread titled "Agent Write tool inserts blank line between every line in file contents" describes doubled newlines in Go files on Windows, which fits what an unconditional LF-to-CRLF replace does to content that already holds CRLF (`(web)`, corroboration only). Settled in `master-read.md`: `Read` strips one `\r` from every line and drops a UTF-8 mark, so the model sees LF text and no BOM for every file it reads.

**Without it.** A CRLF C++ source file overwritten with LF text flips all its line endings. An unconditional `replace("\n", "\r\n")` turns model-sent `x\r\ny\n` into `x\r\n\r\ny\r\n` and doubles every blank line.

**Test.** Old `a\r\nb\r\n` with content `x\ny\n` gives `x\r\ny\r\n`. Old `a\r\nb\r\n` with content `x\r\ny\n` stays `x\r\ny\n`. Old `a\r\nb\n` with content `x\ny\n` gives `x\ny\n` and a reply that says the old file mixed endings. Old file with a BOM and content without one gets the BOM prepended, and content that starts with U+FEFF over a BOM file gets one BOM. A new file with content `x\ny\n` is LF on every host. A lone-CR old file is written as sent. Old `a\nb\n` with content `x\ny` writes `x\ny` with no final LF, and old `a\nb` with content `x\ny\n` writes `x\ny\n`; the reply adds no note for either.

**Confidence.** Medium, because Cursor, Gemini, Zed, and Cline keep CRLF in some form and Cursor and OpenCode keep the BOM, but the strict classification, the mixed-file rule and the as-sent final newline are judgment calls.

### Treat empty `contents` as a real write that empties the file

**Recommendation.** Accept `contents: ""`. On an existing file it empties the file, and on a missing path it creates an empty file. Emptying a non-empty file passes the same read gate as any overwrite. The reply says `Replaced a file of N lines.` or `Created a new file.`, so the emptying is visible.

**Why.** `Write` is the one file tool where empty input is legal and destructive, and a model that asks for an empty file expects one. Skipping the write on empty input leaves the old content in place while the reply says success, which breaks Loud failure. The gate already protects File survival for the destructive case, and the reply makes the effect visible.

**Evidence.** Cursor's `contents` is a bare string with no minimum, the write path truncates and then writes zero bytes, and the model reads the ordinary success sentence (`(bundle)` `(probe)`). Seven other harnesses truncate: pi, rig, zeroclaw, Gemini (`write-file.ts:655-715` has no empty check), OpenCode (`write.test.ts:215-226`), Goose, and Aider's empty block. Rig and Goose say so in the reply (`which had N lines`, `(0 lines)`). Zed is the one that does nothing: `finalize_content("")` emits no event, so the old text stays and the model reads `No edits were made.`, derived from `streaming_parser.rs:260-271` and `edit_session.rs:421-440` with no test pinning it (`(source)`). Copilot allows an empty string only for a new or empty file (`createFileTool.tsx:92`).

**Without it.** An `if contents.is_empty() { return }` guard turns a request to empty a file into a silent nothing, and the model proceeds on a false belief.

**Test.** After a whole-file read, write 120 bytes, then write `contents: ""` to the same path: `stat` reports size 0 and the reply starts with `Wrote contents to`. A missing path with `contents: ""` yields an empty file.

**Confidence.** High, because eight harnesses truncate and the schema allows the empty string.

### Leave concurrent calls to the claims system and report the conflict

**Recommendation.** Take no lock in the Pack. Run the read, the checks, and the write as one synchronous body with no await between them. When `Access` returns `VfsError::Conflict`, tell the model which path conflicted and to send the call again in its next message, with this sentence from the shared `refusal` function: `{path} is also used by another call in this round (a read, write, glob, grep, list or delete that covers it), so it cannot be changed in this round. Send this call again in your next message.` The conflict can come from any sibling that holds a claim over the path in the same round: a `Read` or `Write` of it, a `Glob` whose pattern matches it, a `Grep` or list that read it or listed its directory, or a recursive `Delete` above it, and the sentence says so in its parenthesis. A claim holds until its scope ends, so a finished sibling still conflicts.

**Why.** Runvil forks the access identities of calls in one round together so the calls are concurrent, and one call's write to a path conflicts with a sibling's read or write of it. That is a loud and deterministic refusal, which serves Loud failure and Replay. A mutex would serialize the calls instead, hide the conflict, and let scheduling pick the winner. It could also block the one task that polls every call, and the standing plan says a call must not block while polled. A call's own read followed by its own write is ordered by its identity, so the read-check-write inside one call needs no extra protection.

**Evidence.** The `ToolContext` documentation says "Calls a model asks for in one round fork their identities from the chain together, so they are concurrent with each other: one call's write to a path conflicts with a sibling's read or write of it, as between fanout arms" (`(source)`, `crates/runvil-plugin/src/context.rs`). VFS tests pin the pieces: `one_identity_never_conflicts_with_itself`, `two_writes_by_two_identities_conflict`, and `alias_spellings_of_one_file_land_on_one_claim_key`, where a read of `/a/./b.txt` conflicts with a write of `/a//b.txt` and the message names the canonical key `/a/b.txt` (`crates/runvil-internal/vfs/src/handle/tests/claims.rs`). Cursor takes its lock while the arguments stream, keyed on the raw argument string, so `C:\a\b.txt` and `..\a\b.txt` are two keys for one file; [tool-write.md](tool-write.md) calls that "the one clear bug in its locking" (`(bundle)` `gbe` at 4154498). Pi's queue keys on `realpath` and works for an existing file, but probes on Windows showed a new file reached through a junction alias or a case-different spelling overlapping, and its durable queue ran 133 of 200 trials out of call order for an existing file and 137 of 200 for a new one (`(probe)`). Gemini locks per realpath, and zeroclaw, Codex, OpenCode, Cline, Aider, and Goose take no lock (`(source)`).

A `Conflict` that the Pack maps to a `ToolError` reaches the model as failure text and does not end the run: the tool-call arm resumes a model-issued call with content and turns a tool's own failure into untrusted failure text (`scheduler/tool_call.rs`), and the Engine makes a conflict fatal only for a Lua store answer (`classify_vfs_failure` in `scheduler/apply.rs`). This is read from source and not run. Not run in this pass: a two-call round through the Pack, because `Write` is still a stub.

**Without it.** A Pack mutex produces a last-writer-wins file whose winner changes from run to run, and the model never learns that two of its calls collided. Cursor's raw-string key lets two spellings of one path race.

**Test.** Send two `Write` calls to `/a.md` in one scripted round. Assert that exactly one reply is the conflict sentence above, byte for byte, naming `/a.md`, and repeat the round 20 times to assert the same call wins each time. Send a `Write` and a sibling `Read` of one path in one round and assert the conflict. Do the same with a sibling `Glob` whose pattern matches the path, and assert that the later of the two calls fails with a conflict in both orders. Assert that the run continues after a conflict reply. Send two `Write` calls to the same path in two separate rounds and assert both succeed.

**Confidence.** High, because the contract is documented and the claim tests exist; the Pack-level behavior waits on `write-real`.

### Name the path and the next step in every refusal

**Recommendation.** Build every `VfsError` sentence in the shared `refusal(tool, given_path, &VfsError)` function in `plugin-std`, which the todo `refusal-text` writes before `read-real`, and add one sentence in the Pack for each guard refusal. Every sentence has three parts: what failed, the path as the model gave it, and what to do next. Never return a bare variant text. Do not normalize, expand, or repair the path in the Pack. Argument errors come from the shared check and the body in one style, ``std_Write: `{name}` {problem}``. Cover these cases:

- Plan mode (`PermissionDenied`) and Ask mode (`ApprovalRequired`): pass the policy's reason through unchanged, because the reason carries the next step. Today it does not, so the todo `vfs-mode-text` edits `ModePolicy` in `lib.rs`. Plan then reads `{op:?} on {path} is refused: the Plan mode allows mutations only to markdown paths, which end in .md (case-sensitive)`, and Ask reads `{op:?} on {path} needs user approval: the Ask mode refuses all mutations, and retrying will not help`.
- `IsADirectory`: `Path is a directory, not a file: {path}`
- `NotADirectory`: `Path is a file, not a directory: {ancestor}. {path} cannot be used under it.`
- `ReadOnly` and `OutsideRoot`: the VFS text with its leading label dropped, for example `{path} is served read-only and cannot change.`
- `Conflict`: the sentence in the concurrency behavior above.
- A blank `path`, refused before any I/O: ``std_Write: `path` is empty or only whitespace, and the call passed "{value}"``
- An undeclared parameter: ``std_Write: `{name}` is not a parameter of this tool. The parameters are `path` and `contents`.`` The shared check passes undeclared keys through, so the body refuses them. A call that also lacks `contents` meets the check's required-key refusal first.
- A guard refusal from the cut-message guard, the UTF-8 check, or the read gate: the sentences given in their behaviors.

**Why.** The model repairs a call only from the text it receives. Runvil's policy texts say what happened and leave out the fix: `permission denied: Write on /x.rs is refused: the Plan mode allows mutations only to markdown paths` does not say that `.markdown` and `.MD` are also refused. The policy owns its wording, so `vfs-mode-text` puts the fix in the policy reason and every tool passes it through, and the Pack holds no mode text of its own. One shared function gives each variant one sentence in all six tools, where six renderers would give it six. A refusal that names the next step serves Loud failure and keeps retries to one. Rewriting the path serves nobody, because the model then believes it wrote a file it did not.

**Evidence.** Cursor's mode refusals do name the next step. The plan-mode text ends with `use the SwitchMode tool with target_mode_id=agent first.`, and the ask-mode text ends with `Ask the user to switch to agent mode if edits are required.`. Its weaker texts are the failure matrix's `Error: Unknown error`, a user denial with no reason that renders as `Rejected: `, and `Error: <raw message>` for any other executor error (`(bundle)`; [tool-write.md](tool-write.md), section Failure matrix). The shipped Runvil crate states the rule: "Error messages are written for a model to read: concise, factual, and naming the parameter and the value" (`(source)`, standing plan, constraints). On rewriting paths, pi strips a leading `@`, folds five Unicode space classes covering 15 code points, and maps `/x` to `X:\` on Windows, then answers with the typed path. Probes landed `@types/node/index.d.ts` at `types/node/index.d.ts` and created a plain-space twin of a file named with U+202F while the original stayed untouched (`(source)` `(probe)`). Rig and Goose leave `~` literal, so `~/notes.md` creates a directory named `~` (`(probe)`). Gemini runs `decodeURIComponent` over the path, which turns `100%25.txt` into `100%.txt`. Cursor's `lke` repairs `worktries` to `worktrees`, but only under a `/Users/<name>/.cursor/` or `/home/<name>/.cursor/` prefix, so it never fires on Windows (`(bundle)`).

**Without it.** A model told only `permission denied` in Plan mode retries the same path, then tries `.markdown`, and loses turns. A model whose `@a/b.txt` was quietly written as `a/b.txt` reports a file at the wrong path.

**Test.** For each case above, build the failure with a stub backend and assert that the reply contains the path as given and one imperative next step, and that it matches the sentence above byte for byte where one is given. The Plan-mode reply must name the `.md` rule once `vfs-mode-text` lands; until then the test pins today's reason. `Write` to `@a/b.txt` replies with `@a/b.txt`, and `Write` to `/~/x.txt` replies with `/~/x.txt`; neither path is rewritten. `Write` with `path: "   "` fails with the blank-path sentence and creates no file. `Write` with an extra `mode` key fails naming `mode` and listing `path` and `contents`.

**Confidence.** High, because the crate's own invariant requires it and every failure table in the evidence report shows the cost of a vague refusal. The shared function and the `vfs-mode-text` edit are proposals, and the rule holds without them.

### Make the description say what the tool enforces, and keep `contents`

**Recommendation.** Rewrite the `Write` description in `schemas/write.json` so that each sentence is true of the shipped tool. Say that overwriting an existing non-empty file needs a whole-file `Read` first and that the call fails with instructions otherwise. The description edit lands in the same commit as the gate, so the description never promises what the tool does not enforce. Add the facts the model cannot infer: parent directories are created, empty `contents` empties the file, and the reply says whether the file was created or replaced. Add three facts about paths. First, `Write` creates or replaces any path the call names, including paths that `Glob` and `Grep` leave out. Second, a path that names a symbolic link is written through to the file it points to, and a link that leaves the run's filesystem is refused. Third, keep the shipped `path` text, `The absolute path of the file to write`: every file tool advertises the absolute form, because `Glob` and `Grep` print it, though each also accepts a relative path. Add two facts that come from `Read`. First, `contents` is plain file text, without the six-wide `LINE_NUMBER|` prefix that `Read` puts on each line and without the bracketed trailer and note lines it adds. Second, `Read` does not show whether a file ended in a newline, and `Write` stores `contents` as sent, so the file ends in a newline only when `contents` does. Keep the field names `path` and `contents`, and record the edit in `schemas/EDITS.md`.

**Why.** The model treats a description as a contract. A promise the tool does not enforce misleads, as the evidence report says of OpenCode: a model that trusts the promise skips nothing and a model that ignores it loses nothing. The shipped Runvil text makes that promise today. Keeping `contents` costs at most one retry for a model that sends `content`, because the argument check already fails with the real key named, and the retry is loud. An alias would loosen `required` in a schema that is otherwise Cursor's, and the Pack's invariant is that the stored text matches Cursor's except for listed edits.

**Evidence.** Observed: `schemas/write.json` says `If the destination file already exists, you MUST first read it before writing, so a change you did not make is never overwritten`, and `EDITS.md` records that the sentence was added "to require a read before overwriting, matching Cursor's own guidance. No behavior was added; only the wording is ours" (`(source)`). Cursor's own four branches disagree: `dsv3-1018` and `isComposer15` tell the model a prior `read_file` is mandatory, `cursor-0226` has one line, and the default branch says nothing about reading, so the default is "the branch most likely to blind-overwrite" (`(bundle)` offsets 4232400 to 4233350). OpenCode promises `This tool will fail if you did not read the file first` and a search of its source finds no check (`(source)`, v1.19.5). The field `contents` is the odd one out: pi, Claude Code, Copilot, Zed, rig, zeroclaw, Gemini CLI, OpenCode, and Goose all use `content` (`(source)`). Rig closes its schema and answers a wrong key with `` `write` has no argument `contents`. Its arguments are: `path`, `content`. Call it again with only those. `` (`(probe)`). In Runvil the argument check fails a missing required key with ``std_Write: `contents` is required, and the call left it out`` and passes undeclared keys through, so the body refuses them (`(source)`, standing plan, Functional Specification; behavior 9).

**Without it.** If the description ships before the gate, it keeps promising protection that does not exist, and a model that trusts it skips reads it would otherwise make.

**Test.** In `spec-tests.rs`, assert that the `Write` description names `StrReplace`, parent directories, the empty-contents rule, the created-or-replaced reply, the whole-file `Read` rule for an overwrite, the `LINE_NUMBER|` prefix, the final-newline rule, the any-path sentence and the symbolic-link sentence. Add an integration case where `{"path": "/a.md", "content": "x"}` fails as invalid arguments naming `contents`, because the check's required-key refusal comes before the body's refusal of an undeclared key.

**Confidence.** High, because the unenforced promise is observed in the shipped file and the fix is local to one JSON file and one test.

## Where Runvil should differ from Cursor

Table 7 lists where this report leaves Cursor's design. Rows that begin `Drop:` are machinery that exists only because of Cursor's JavaScript runtime, its proto result layer, or its editor client. The confidence word closes each last cell.

| What Cursor does | What Runvil should do | Why |
|---|---|---|
| `Write` never reads the read result's `truncated` flag, while `StrReplace` does | Gate on a whole, current read record (behavior 2) | Closes the safety asymmetry for the model's partial view as well as the tool's (medium) |
| Nothing inspects a message cut by the output limit | An Engine guard fails every call in a cut message (behavior 1) | New-content loss; pi and rig built the same guard (high) |
| Sniffs encoding in the first 8,192 bytes, keeps encoding and BOM, re-imposes line endings | Write UTF-8 as sent, refuse a non-UTF-8 old file, keep BOM and uniform CRLF (behaviors 5 and 6) | `Read` decodes UTF-16 by its mark and everything else lossily, so for most such files the text the model saw does not map back to the old bytes, and the first version builds no encoder (medium) |
| Drop: per-line CRLF merge with an LCS capped at 4,000,000 cells and a backfill pass | Write mixed-ending files as sent and say so | The result changes with a trailing newline, and the case is rare (medium) |
| Truncate in place, then fsync | One `Access::write`; the backend replaces through a temp file | `RealAccess` already does it (high) |
| `mkdir -p` as a separate step before the write | No `mkdir` | Both backends create ancestors on write, and Plan mode refuses a `mkdir` of a non-`.md` directory (high) |
| A sha256 fingerprint keyed on tool-call id and path, 60-second TTL, 1,000 entries, silently off above 8 MB | The hash and whole-file flag from the last read of the path, in a run record with no TTL and no silent size lapse; a file `Read` refuses for size gets no record, so the gate refuses it | Cursor's window is milliseconds inside one call; the record covers the seconds between the model's read and its write (medium) |
| Takes the file lock while arguments stream, keyed on the raw argument string | No Pack lock; the VFS claims, keyed on the canonical path, decide | Two spellings of one path are one claim in Runvil and two locks in Cursor (high) |
| Reads the old file four or more times per write | One `Access::read` threaded through every check | Bounded cost (high) |
| Drop: snapshots of the old content up to 50 MB, a 262,144-byte hook snapshot, full-content elision at 524,288 UTF-16 units, a diff on a worker thread above 262,144 | No diff, no snapshot, no hooks | These feed Cursor's client and tracking, and the model never sees them (high) |
| Drop: a two-layer result, with a rich `EditSuccess` for the client and `message` for the model | One reply the model reads | Runvil has no client for the structured record; every fact in the reply must earn its tokens (high) |
| Replies `Wrote contents to <path>` and nothing more | Keep that prefix, add created-or-replaced and the old line count (behavior 4) | The one fact the model cannot rebuild without another call (medium) |
| Echoes the raw argument path while writing the resolved path | Echo the argument as given; use `Access::canonical_path` for the record key (`master-read.md`) | The two differ only for a non-canonical spelling such as `/a/./b.md`, because `VfsPath` canonicalizes on receipt, and the model wrote the argument, so it recognizes it (medium) |
| Drop: the `lke` repair of `worktries`, `file://` conversion, `~` expansion, the worktree guard, `.cursorignore` forcing a permission check | Write only to the path the model named | These are Cursor product features, and path leniency on a write picks a different file than the model meant (high) |
| Drop: a binary-extension list that switches the encoder to base64 or latin1 | Text only; a non-UTF-8 old file is refused | Two extension lists disagree with each other and with the byte sniff (high) |
| A plan-mode throw after the read and the diff; six allowed suffixes and a canvas regex | `Access::check_write` runs the VFS policy first, so the Pack adds no plan-mode check of its own and runs no diff; keep `.md` only and let the policy reason name the rule | The policy belongs to the VFS and widening it changes every tool; the canvas path is a Cursor feature (medium) |
| Drop: `Rejected:` and `Error:` renderer prefixes, and `Edit rejected:` for the client | One plain sentence per refusal (behavior 9) | The prefixes exist because of a proto union of result cases (high) |
| Three description branches by model; the tool is withheld from GPT-5-family models, which get `ApplyPatch` instead of both `StrReplace` and `Write` | One description, offered to every model | Per-model selection is a product decision, and this Pack has no `ApplyPatch` (medium) |
| Drop: the `encodingHint` argument of `WriteArgs`, which the model cannot reach | Do not build it | A parameter with no caller (high) |
| Drop: a write loop that breaks on a zero-byte write and reports success | None | A Node loop artifact; the backend returns an error or completes (medium) |
| Drop: a second tool, `pi_write`, with its own key, queue, and byte count | Build one write tool | One tool keeps one set of rules (high) |
| Keeps an existing file's mode, because an in-place open ignores the mode argument | Have the backend copy the old mode onto the temp file | Rig and Gemini copy permissions for the same reason (medium) |
| Field name `contents` | Keep `contents` | The argument check names the key on a miss, so a model that sends `content` retries once (high) |

*Table 7. Where Runvil leaves Cursor's design. Twenty-three rows; `Drop:` marks machinery tied to Cursor's runtime or client.*

## Mapping to Runvil

Table 8 maps each behavior to the `Access` method that covers it. The owner is Vinnie for every row. A todo id that the standing plan lacks is marked new, and each new id is a proposal.

| Behavior | `Access` method | Status | What the gap needs | Next step |
|---|---|---|---|---|
| 1. Refuse a Write from a cut message | None; the Engine holds `finish_reason` | Missing | An Engine hook before dispatch that fails every call in a cut batch, and the provider strings for an output limit | `engine-cut-guard` (new, independent) for the hook; `write-real` for the Pack-side test |
| 2. Gate an overwrite on a whole, current read | `read` for bytes, `stat` for existence, `canonical_path` for the key | Partial | A required per-run service holding the entry `Read` publishes (hash, shown range, total and whole-file flag), keyed by canonical path; `Read` fills it and `Write` replaces the entry after its own write; the key declared in `plugin-std`, a `needs` entry on the Pack and a Host provider; a SHA-256 dependency | `read-real` for `Access::canonical_path`, without the record; then `write-real`, which adds the key, the `needs` entry, the Host provider and the `pack-tests.rs` change in one commit |
| 3. `Access::check_write`, then one `Access::write`, no `mkdir`, no temp file | `check_write` (new), `write` | Covered | A new `Access::check_write(path)` that runs the policy only; mode copy in the backend's `atomic_write`, outside `plugin-std` | `write-real` for the call, with `check_write` in its first commit; `vfs-write-mode` (new, independent) for the mode copy |
| 4. Reply created or replaced with the old line count | `stat`, `read`, then `write` | Partial | A byte-counting line function with the byte order mark rule, and the shared `plural` helper | `write-real`; `Access::canonical_path` comes with `read-real` |
| 5. Refuse a non-UTF-8 old file; write UTF-8 as sent | `read` | Covered | Decode the old bytes with `String::from_utf8` in the Pack and keep `valid_up_to` | `write-real` |
| 6. Keep BOM and uniform CRLF; LF for a new file | `read` for bytes | Covered | A `text` module of pure functions in the Pack | `write-real` |
| 7. Empty `contents` empties the file | `write` | Covered | Nothing | `write-real` |
| 8. Report a claim conflict | `read`, `write`, `glob` and `list` admit claims; `VfsError::Conflict` | Covered | The shared `refusal` function's conflict sentence, which names the path and says to send the call again in the next message | `refusal-text`, then `write-real` |
| 9. Name path and next step in every refusal | The 21 `VfsError` variants | Covered | The shared `refusal` function for every reachable variant; the Plan and Ask next-step text in the policy reason, an edit to `ModePolicy` in `lib.rs`; an undeclared-parameter refusal in the body | `refusal-text`, then `write-real`; `vfs-mode-text` (new, independent) for the policy wording |
| 10. Honest description | `schemas/write.json`, `EDITS.md`, `spec-tests.rs` | Fix | Rewrite the description, record the edit, extend the spec test | `write-real` |

*Table 8. Mapping from behavior to `Access` coverage. Owner is Vinnie for every row.*

Tests to add to `plugin-std`, each in a `<stem>-tests.rs` file or in `tests/it/`, with every Rust file under 500 lines:

- A `text-tests.rs` file for the pure functions: the CRLF, BOM and final-newline vectors from behavior 6, the line count vectors from behavior 4, and the UTF-8 check from behavior 5.
- A `gate-tests.rs` file for the read gate: the five vectors in behavior 2, with a stub run service.
- An integration case in `tests/it/tools.rs` per behavior, replacing the stub case for `Write`: create, replace, empty, Plan mode into a new directory, Plan mode over an unread existing file, Ask mode, conflict between sibling calls including a sibling `Glob`, a blank path, an undeclared parameter, and each refusal text.
- A `spec-tests.rs` case that holds the description to what the tool enforces, and an invalid-arguments case for `content`.
- A `plural` case in the shared helper's test file, with `Replaced a file of 1 line.` as one vector.
- In the Engine: a scripted cut-message test for behavior 1, in the chat scheduler's test tree.
- In the VFS: an ignored Unix test that overwrites a `0755` file and expects `0755`, until the backend copies the mode. A test of `Access::check_write` that shows no claim, no event and no backend call. A test of the Plan and Ask wording in `ModePolicy`.

Schema and description fixes, none made in this pass:

- `schemas/write.json`: replace the description as behavior 10 says, in the same commit as the gate.
- `schemas/EDITS.md`: record each wording change, since the file's rule is that every line differing from Cursor's text is listed.

Plan edits, none made in this pass, proposed for the standing plan:

- The `write-real` todo: replace "parent creation through `Access::mkdir`, and the exact `Wrote contents to <path>` success text" with "no `mkdir`, the four guards of `master-write.md`, and the `Wrote contents to <path>` prefix with a created-or-replaced sentence".
- Needs: append "From `write-real` the Pack declares one `needs` entry, the read record, and the Host supplies it for each run." In the testing plan, replace "no per-run needs" in the `pack-tests.rs` line with "and, from `write-real`, the one per-run need".
- Scope: the Engine and VFS lines change only through the todos `engine-cut-guard`, `vfs-write-mode` and `vfs-mode-text`, each with its own tests, and `Access::check_write` and `Access::canonical_path` are `Access` additions that regenerate `crates/runvil/public-api.txt` in their own commits.
- The Decision Record records the new todo ids `refusal-text`, `engine-cut-guard`, `vfs-write-mode` and `vfs-mode-text`.

Confidence in these plan edits is high, because they only make the plan text match the behaviors above. The new todo ids are proposals, confidence medium.

## Risks and open limits

Table 9 lists what the evidence does not settle. Likelihood rates the event on the fixed ladder. Confidence rates the evidence behind that rating. Items marked *observed* were read in source, and items marked *derived* follow from reading without a run.

| Risk | Likelihood | Confidence in the evidence | Basis |
|---|---|---|---|
| A provider's output-limit finish reason is not recognized, so a cut message passes the guard in behavior 1 | Roughly even chance | Low | Observed: the Engine holds `finish_reason` as an `Option<String>` beside the batch. Not verified: the strings each provider sends, or whether a cut call's arguments reach the Engine as parsed JSON |
| The read gate blocks a run: a file whose numbered text passes `Read`'s 100,000-byte cap, a file over its 16 MiB ceiling, a binary file, or a prompt that offers `Write` without `Read` can never satisfy it | Likely | Low | Expected at least once in a long-running agent. Derived from the cap, the ceiling, the binary refusal and the gate rule; no transcript measures how often models hit it |
| A model bypasses the gate with `Delete` then `Write`, since a missing path needs no read | Unlikely | Medium | By accident; the gate guards against accident and does not stop intent. Derived from the new-file exemption |
| An overwrite through the backend loses the old file mode on Unix | Very likely | Medium | For a file with a non-default mode. Derived: `atomic_write` never calls `set_permissions`; not run on Unix |
| A mistyped path creates a new file in a new directory and passes the gate, because new files need no read | Likely | Medium | Cline's probe created `ghost.txt` from a mistyped path; rig and Goose create a directory named `~`; the reply's created-and-directory sentences only make the typo visible |
| A rewrite from `Read`'s view drops what `Read` hides: the final newline, the BOM or the `\r` of each line | Roughly even chance | Low | `Read` hides all three (`master-read.md`). The BOM rule and the uniform-CRLF rule in behavior 6 restore the mark and the endings of a uniform file. The final newline survives only when `contents` ends in one, and the description says so. No transcript shows how often models drop it |
| The run's read record is lost on compaction or resume, so a valid write is refused | Likely | Low | If Runvil compacts a run. Rig's gate clears on compaction (inferred); Claude Code's store survives a resume; Runvil's compaction was not checked |
| A symlink leaf behaves differently from this reading | Unlikely | Low | Content operations follow an in-root link and refuse an escaping or dangling one (`real/resolve.rs`, `contain`; `real/tests/links.rs`). The file-link tests are ignored on Windows, which needs the symlink privilege |
| Cursor's 8 MiB outcome differs from the code reading: a cut read feeds `Write` and the tail is lost | Unlikely | Medium | Observed in both binaries: `Write`'s closure never mentions `truncated`. Inferred: the file is lost, because the evidence report never ran a file just over 8 MiB through `Write` |
| Cursor's line-ending vectors differ from a live run | Unlikely | Low | The vectors come from a transcription of the daemon source, and no live probe of the merge exists |

*Table 9. What the evidence leaves open, with likelihood and confidence rated apart.*

Five limits sit outside the table. The sources do not show whether a server upstream of Cursor closes partial JSON when the output limit cuts a message, so Cursor's strict `JSON.parse` may protect more or less than the evidence report assumes. The evidence report found no probe that settles whether Cursor's installed daemon and bundle are the same build, and one live write to a UTF-16LE file would settle it. This pass ran no probe of Cursor or Runvil, so every Runvil statement is a reading of source at the commit named under Sources, and each behavior waits on `write-real` for a test that runs. The fourth limit is the size of the old file. `Read` refuses a file over 16 MiB, so no whole-file record exists for one and the gate refuses the overwrite before `Write` loads it. For a file that grew past 16 MiB after a whole read, `Write` loads the old file whole through `Access::read`, and this report sets no bound on that read. The fifth limit is whether a conflict ends a run. A `Conflict` that the Pack maps to a `ToolError` reaches the model as failure text and does not end the run (`scheduler/tool_call.rs`, read, not run).

**What would change the recommendation.** Five findings would move it.

- A measured need to rewrite UTF-16 files, which `Read` already decodes by their mark, or a `Read` that detects legacy encodings, moves behavior 5 toward option C in Table 4, because `Write` could then encode the text back.
- A measured rate of runs blocked by the gate, or a transcript showing models looping on the refusal, moves behavior 2 toward option H in Table 2 (refuse every overwrite) or toward a warning in the reply.
- An Engine that never sees a finish reason before dispatch moves behavior 1 to the model-client layer.
- A VFS compare-and-swap, such as a write that succeeds only while the file still hashes to a given value, replaces the Pack's hash compare in behavior 2 and closes the window between the check and the write.
- A live probe that contradicts the Cursor vectors changes only the Cursor column of Table 7, because Runvil drops the merge.

## Sources

Every Cursor offset, string, and line-ending vector below is quoted from the evidence report, which derived them from the two Cursor artifacts. This pass did not reopen the bundle or the daemon. Runvil statements come from files read directly in this pass.

**Evidence reports and parent specification.**

- [tool-write.md](tool-write.md), the evidence report for `Write`, enriched with twelve harnesses and edited for prose. The primary ground, cited by section name.
- [file-tools-index.md](file-tools-index.md), the cross-tool facts and the corrections to the parent specification. Cited for the safety-asymmetry row.
- [master-read.md](master-read.md), the upstream report for `Read`. Cited by behavior name for the cap, the 16 MiB ceiling, the line model, the decode ladder, the read record and `Access::canonical_path`.
- [tool-read.md](tool-read.md) and [tool-str-replace.md](tool-str-replace.md), named only because they hold the read cap and the `StrReplace` half of the asymmetry. Not reread for this report.
- `agent-file-tools.md`, the parent specification `(spec)`. Reached only through the evidence report. Not reread.

**Cursor 3.27.12.**

- `(bundle)` `cursor-agent-exec/dist/main.js`, 9,814,287 bytes. Offsets in this report are for this build; the parent specification's offsets are for a 9,674,628-byte build and are stale.
- `(daemon)` `cursor-agent-host/dist/agent-host-daemon/dist/bin/daemon.cjs`, 20,213,626 bytes, unminified.
- `(probe)` The file-mode measurement (Git Bash, umask 0022: existing `755` stayed `755`, a new file became `644`), reported in the evidence report.
- `(web)` A Cursor forum thread titled "Agent Write tool inserts blank line between every line in file contents". The evidence report records no URL, so none is given here. Corroboration only.

**Other harnesses, `(source)` unless marked.**

- Claude Code 2.1.311, `claude.exe` decoded as latin1: the `write.js`, `read-file-state.js`, `freshness-checks.js`, and `error-strings.json` extracts.
- VS Code Copilot: `https://github.com/microsoft/vscode` at `b208bc2501` (2026-10-25), `createFileTool.tsx`.
- Zed: `https://github.com/zed-industries/zed` at `37faef81f7` (2026-10-27).
- pi: `git@github.com:earendil-works/pi.git` at `a92283056` (2026-10-27), version 1.4.0, with commits `351efc828` and `e583b290a`. `(probe)` Queue probes on Windows with Node v24.19.0.
- rig: `https://github.com/0xPlaygrounds/rig` at `a83b774d0` (2026-10-25), with commit `e034438fa`. `(probe)` Compiled copies of the function bodies with `rustc` 1.99.1 on Windows.
- zeroclaw: `https://github.com/zeroclaw-labs/zeroclaw` at `2d85371f3` (2026-09-18), version 0.8.5.
- Codex CLI: `https://github.com/openai/codex` at `bea47c06dfac12ecce362090418e12a6faeedbf9`.
- Gemini CLI: `https://github.com/google-gemini/gemini-cli` tag `v0.66.0`, commit `970413207732f5d4d8c50540c7fc359c44fdaaf2`.
- OpenCode: `https://github.com/anomalyco/opencode` tag `v1.19.5`, commit `810c0eba1a7348036f8bdf61e6632d0c36c6876c`.
- Cline: `https://github.com/cline/cline` tag `v4.1.26` (`a459f1d`). `(probe)` The `ghost.txt` run.
- Aider: `https://github.com/Aider-AI/aider` at `5dc9490bb35f9729ef2c95d00a19ccd30c26339c`.
- Goose: `https://github.com/aaif-goose/goose` at `ee8500d5e060eea6f2bcee92e8a5e351f12c3a70`, tag `v1.56.0`.

**Runvil, `(source)`, repository at commit `08801150f` (2026-10-27).**

- `crates/runvil-internal/vfs/src/handle/operations.rs`, `error.rs`, `stat.rs`, `lib.rs` (the `ModePolicy` and `is_markdown`), `real.rs`, `real/files.rs`, `real/resolve.rs`, `memory.rs`, `traits/backend.rs`.
- Tests read: `memory-tests.rs`, `real/tests/atomicity.rs`, `real/tests/links.rs`, `handle/tests/claims.rs`.
- `crates/runvil-plugin/src/context.rs`, for `ToolContext`.
- `crates/runvil-internal/engine/src/execute/scheduler/chat.rs`, for the finish reason beside a tool-call batch, and `tool_call.rs` and `apply.rs` in the same folder, for how a tool's own failure and a conflict resume.
- `crates/plugin-std/src/pack.rs`, `lib.rs`, `schemas/write.json`, `schemas/EDITS.md`, `tests/it/tools.rs`.
- The standing plan, "The std Pack, Cursor's six file tools" (2026-10-27), `vibe/2026-10-27-2-plugin-std-file-tools.md`, for the todo ids and the constraints.

*2026-10-27 - Claude Sonnet 5.5*
