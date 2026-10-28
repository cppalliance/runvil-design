# Cursor's `Write` tool: an implementation briefing

This is the report on Cursor's `Write` file tool, the one that creates a file or replaces its whole contents. It is written for someone about to build their own version, so it names the forks and weighs each one rather than describing Cursor for its own sake.

Three labels recur. `(bundle)` means read out of Cursor's shipped minified agent bundle. `(daemon)` means read out of the unminified daemon copy that ships beside it, which keeps original identifiers and source paths. `(probe)` means measured by running something. Claims carried over from the parent specification without re-derivation this pass are marked `(spec)`. Harness claims are `(source)`, and the few public-document claims are `(web)`.

The bundle this was derived against is Cursor 3.27.12:

```
C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js
9,814,287 bytes, one physical line, read as latin1
```

The parent specification was written against a 9,674,628 byte build, so every offset in it is stale. Both numbers appear below wherever a claim depends on one.

## The short version

- **`Write` never checks that its view of the old file was complete, and `StrReplace` does.** `StrReplace` reads `truncated` from the read result and refuses with `File is too large to edit`. `Write`'s execute closure contains no reference to `truncated`, so a file the read cut at 8 MiB is diffed and overwritten from the truncated view, the tail is lost, and the model reads `Wrote contents to <path>`. The same asymmetry has a second half in the new content: pi fails every tool call in a message that stopped on the output-token limit, so a truncated `contents` cannot replace a file, and Cursor's `Write` has no such check in its path. D9 compares the options and recommends both guards. `(bundle)` `(source)`
- **`Write` does not write UTF-8.** It sniffs the existing file's encoding from the first 8192 bytes, keeps the encoding, keeps the BOM if there was one and never adds one if there was not, and re-imposes the file's line-ending style including per-line mixed endings. Writing ASCII into a UTF-16LE file leaves a UTF-16LE file. `(bundle)` for the call into the encoder, `(daemon)` for the encoder itself, `(probe)` for the round trip.
- **There is no read-before-write gate, and no error string for one.** The description text tells the model `you MUST use the read_file tool first`, and nothing enforces it. A file seeded from a shell and never read through the agent was overwritten with a plain success message. Do not invent a gate error; there is no such string in the bundle to copy. OpenCode's description makes the same unenforced promise, and rig's content-hash gate exists only as an unshipped corpus plugin. `(bundle)` `(probe)` `(source)`
- **The only freshness protection is a sha256 fingerprint between the tool's own read and its own truncate.** It closes a race measured in milliseconds, inside one call. It does not depend on what the model believes, and it does not fire when the model has never seen the file. `(bundle)` `(daemon)`
- **The write is truncate-in-place plus fsync.** It is durable but not atomic: a crash between the truncate and the last write leaves a truncated or half-written file, and the previous contents are gone. Rig and Gemini CLI replace through a temp file and a rename, and rig's `write_atomic` is a Rust reference (`rt/fs.rs:82-121`); pi, zeroclaw, Goose, Codex, Cline, Aider, OpenCode and Zed truncate in place. `(bundle)` `(daemon)` `(source)`
- **The plan-mode gate fires after the read and after the diff.** A refused plan-mode write of a `.ts` file still performs a full read of that file and a full diff computation. Allowed suffixes are exactly `.md`, `.markdown`, `.mdown`, `.mkd`, `.mkdn`, `.mdx`, plus paths matching a canvas regex, which means `.canvas.tsx` under `.cursor/projects/<x>/canvases/`. `(bundle)` at 4150090 for the throw, 4140186 for the suffix list, 4140241 for the canvas regex
- **The content field is `contents`.** It is not `file_text`, `content`, or `file_content`. The schema is a strict two-field object on every branch. It is the odd one out: pi, Claude Code, Copilot, Zed, rig, zeroclaw, Gemini CLI, OpenCode and Goose all say `content`. `(bundle)` at 4229910 and 4230054, `(source)` for the rest
- **`contents: ""` is legal and it truncates.** The field is a bare string with no minimum, the write path truncates first and then writes zero bytes, and the model sees the ordinary success sentence. An implementation that treats an empty write as a no-op diverges on the first test that matters. Zed is the only harness read that does that (`No edits were made.`); pi, rig, zeroclaw, Gemini, OpenCode and Goose truncate. `(bundle)` `(probe)` `(source)`
- **The lock is taken during argument streaming, as soon as the path field completes** - not after the whole call parses, and before validation. That is why two concurrent writes to one path serialize early. The lock key is the raw argument string before normalization, so `C:\a\b.txt` and a relative spelling of the same file are two different locks. The no-lock refusal is `Failed to acquire file lock - path may not have been parsed` (the parent spec and its checklist quote `path may not be parsed`, which drops the words `have been`), and it runs after validation, so a missing `path` reports `Invalid arguments` and never reaches the lock error. `(bundle)` at 4154498 and 4156465, `(daemon)` `withStreamingEditParsedArgs`
- **The mode argument is `0o666` before umask, and it only applies when the file is created.** An existing `0755` script keeps `0755`, because POSIX ignores the mode on an open of an existing file. A brand new file gets `0644` under umask 022, with no way to ask for an executable bit. The parent spec states this backwards, and the probe below settles it. `(probe)`
- **The result the model sees is one flat sentence, `Wrote contents to <path>`, with `isError: false`.** The diff, the line counts, and both full file contents exist in the structured record and never reach the model. This is the clearest example in the tool surface of a two-layer result: a rich record for the client, a deliberately poor one for the model. Five other harnesses do tell the model whether it created or replaced (Claude Code, Gemini, rig, Goose, OpenCode V2), and pi and rig dropped byte counts that were not bytes. `(bundle)` at 4157158 for the renderer, 4157903 for the success builder, `(source)` for the rest
- **The path in that sentence is the raw argument.** Write `file:///C:/x/y.txt` and the message echoes the URL. Write `~/y.txt` and it echoes the tilde. The structured `path` field carries the resolved absolute path, so the same call reports two different paths in two places. `(probe)`
- **A binary extension does not block the write, it changes the encoder.** For `.png`, `.exe`, `.bin` and the rest of the shortcut set, the content is interpreted as base64 if it looks like base64 and as latin1 otherwise, so `café` lands as `63 61 66 E9` rather than `63 61 66 C3 A9`. Binary *bytes* in any file are rejected earlier, by the read layer's byte sniff. `(daemon)` `getBufferForWrite`, `(probe)` for the bytes
- **Reading a file and writing it straight back preserves encoding, BOM, and line endings, and fails in three narrower cases.** A lone `\r`, a `contents` with no trailing newline into a mixed-ending file, and a non-UTF-8 file whose U+FFFD count the model changed. Q10 has the detail, and it matters most for a byte-preserving Rust port. One check, `if contents == what I just read { write the original bytes }`, beats every heuristic here.

## Decisions you have to make

Each of these is a fork. The recommendation is mine, with a confidence word and a one-phrase reason, and you are free to overrule it as long as the choice stays visible.

### D1. Does `Write` preserve the existing file's encoding, or write UTF-8 always?

The options:

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. UTF-8 only**, ignore the existing file's encoding | Rewrites a UTF-16 or latin1 file into UTF-8 on first touch, and if the model's content came from a read of that file, every non-ASCII character either survives as UTF-8 or turns into U+FFFD | Trivial to implement over a byte-oriented `Access::write`. No encoder dependency. No charset detection. Every byte the tool writes is a byte the model produced | Files that were UTF-16 stay readable by the tool and become unreadable by whatever expected UTF-16. A byte-for-byte round trip test fails immediately. Any repo with latin1 source (a lot of older C and C++ in this workspace's domain) gets re-encoded silently |
| **B. Detect and preserve**, Cursor's model | You need an encoding detector, a decoder, an encoder, and a BOM policy - and your VFS hands you bytes, so all of it lives in the Pack | Read-then-write-unchanged round-trips for the encodings you support. The tool stops being a lossy rewriter | Every encoding you claim to support becomes a test obligation. Detection by a library like jschardet is heuristic, so "preserve" means "preserve what I guessed" |
| **C. Detect, preserve, and refuse when unsure** | The tool now fails on files it could have written, and the model has to fall back to something else | Never silently re-encodes a file it did not understand | A refusal path with no good recovery story. The model's next move is Bash or a second Write, both worse |
| **D. Carry the encoding as a first-class argument** | A `String` in the schema either way, so the model has to know the encoding, which it does not | Explicit. No guessing | The model will not supply it correctly, and the parameter is not in Cursor's surface, so a model trained on Cursor will not use it |

**Recommendation: B, with a narrow detected set and a documented fallback.** Confidence high. Reason: the whole value of matching Cursor's surface is that models already know how to call it, and the whole value of a virtual filesystem is that you control the bytes - throwing that away to save an encoder is a bad trade.

What the other harnesses add is two cheaper points on the scale. Preserve only the BOM: OpenCode keeps the old file's BOM or one the new text starts with and emits at most one (`write.ts:48-49`), which preserves a UTF-8-with-BOM round trip for ten lines of code. Or write UTF-8 verbatim and say so, which is option A with an honest label, and it is the common choice: pi, rig, zeroclaw, Goose and Codex all do it, and none of them preserves a UTF-16 round trip. Zed shows the Rust route for option B: `chardetng` to guess, `encoding_rs` to decode and encode, UTF-16 written by hand because `Encoding::encode` cannot emit it, and unmappable characters written as `&#N;` (`crates/file_content/src/file_content.rs:89-146`). `(source)`

The sub-question that carries most of the cost is **which encodings?** Cursor's set, recovered from the daemon's encoding module, is utf8, utf8bom, utf16le, utf16be, utf32le, utf32be, latin1, and about twenty windows-125x and iso-8859-x labels mapped through iconv, plus whatever jschardet returns, which is a lot of surface. Three defensible subsets:

- UTF-8 (with and without BOM), UTF-16LE, UTF-16BE. Six byte patterns, no charset detection library, no dependency. Covers essentially every source file in a modern repo. Confidence high that this is enough for a coding agent.
- Add latin1 and windows-1252. Still no detector needed if you only honour a BOM or an explicit hint, but then a latin1 file with no BOM is treated as UTF-8 and fails to decode. Confidence medium.
- Everything Cursor does. You now own a jschardet port and its failure modes. Confidence low that this is worth it.

### D2. Where does encoding live - in the Pack, in the VFS, or nowhere?

Your `Access::write(path, contents: &[u8])` takes bytes and there is no encoding parameter anywhere in the crate. Cursor puts this logic in `LocalWriteExecutor`, below the capability boundary, which is the layer you do not have.

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. All of it in the Pack** | The Pack reads bytes, detects, decodes, merges line endings, encodes, writes bytes. Every Write call reads the file twice if you are not careful (once for the diff, once for the encoding) | No VFS change. The VFS stays an honest byte store. The Pack owns a policy decision, which is where policy belongs | The Pack needs an encoder dependency, so `plugin-std` grows a dependency it might not want. And the detection cache is per-Pack, so two Packs writing one file disagree |
| **B. A new `Access::read_text` / `Access::write_text` pair that carries encoding metadata** | A new VFS capability with a shape to follow, and every backend must implement it or the default must be sane | One place that decides text semantics, shared by Write, StrReplace, and Read. `read_string` already exists, so half of it is there | You are now specifying text semantics in the VFS, which is a much bigger commitment than one tool. The `NotUtf8` error already implies the VFS thinks UTF-8 is *the* text encoding |
| **C. A `TextCodec` service the Pack receives through `HostServices`** | One more host service to define, version, and document | The Pack decides policy, the host provides mechanism, and a test host can supply a fake codec | More machinery than the first version needs |

**Recommendation: A for the first version, with the detection isolated behind one module so B stays open.** Confidence medium. Reason: one Pack owning the whole text question is the smallest change that does not bake a wrong assumption into the VFS, and the VFS's `NotUtf8` variant shows the crate has not decided text semantics yet.

The other harnesses split the same way. Rig's `Workspace::write` takes `&[u8]` and `write.rs` does nothing beyond `content.as_bytes()`, which is option A. Zed puts detection and encoding in the buffer and worktree layer, `worktree.write_file` and `encode_text` (`worktree.rs:1863-1880`), which is option B. `(source)`

### D3. Atomic writes, or match Cursor's truncate-in-place?

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. Match Cursor**: truncate in place, then write, then fsync | A crash mid-write loses the file's old contents and leaves a partial new one | Byte-identical crash behaviour to Cursor, for whatever that is worth. No temp file to clean up. Works on filesystems that dislike rename-over-open-file | Durability without atomicity. The window is small and the consequence is total |
| **B. Temp file, fsync, rename** | A sibling temp file that must be named safely, cleaned up on failure, and excluded from globs and watches. Rename is not atomic across filesystems, so the temp has to be in the same directory | The file is either the old bytes or the new bytes, never a mix. This is what every serious editor does | Two syscalls that can fail in more ways, and a temp file that a stray `Glob` can see mid-write unless you exclude it |
| **C. Temp file plus rename plus fsync of the directory** | Same as B plus one more fd | Survives a power cut, not just a process crash | Marginal over B for an agent harness |

**Recommendation: B.** Confidence high. Reason: "durable but not atomic" is the worst of both - you run fsync and still lose the file - and the temp-then-rename pattern is twenty lines.

One point matters for your VFS in particular: if `Access` has no rename-into-place that is atomic with respect to another access's concurrent read, then B at the Pack layer is only as atomic as `Access::write` is. Check whether `Access::rename` claims the destination as a write (`operations.rs` says it does), because if so, temp-then-rename through `Access` gets you the claim ordering for free.

Other harnesses. Rig's `write_atomic` and Gemini's `writeTextFile` both do B, and between them they show what it costs: the file gets a new inode, the owner becomes the writing process, a hard link keeps the old text (probe), and the permissions have to be copied onto the temp file before the rename. Rig refuses a read-only or non-regular target before it creates anything, and names the temp `.name.pid-n.tmp`, a hidden name its listing skips, though a crash between create and rename can leave one (`fs.rs:80-81,142-151`). Gemini retries `EBUSY`, `EPERM` and `EACCES` up to five times for Windows sharing errors. `(source)` `(probe)`

### D4. Do you add a read-before-write gate?

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. No gate**, Cursor's model | A model that never read the file can destroy it, and nothing says so | Matches Cursor. One less piece of state to keep per run. No dead-end turn for the model | The failure is silent and total. This is the design that produces plausible-but-wrong work rather than a loud error |
| **B. A real gate**, Claude Code's model: refuse unless this session read the path | Per-run read state keyed on canonical path, an mtime or hash comparison, a partial-read flag, and a set of escape hatches for the cases where a gate is wrong (a brand new file, a file the model was told about but not shown) | The model cannot overwrite what it has not seen. The safety property is the one people assume the tool already has | Every write of a file the model has not read becomes a two-turn dance. And a gate that fires on a legitimate write is worse than no gate, because the model's recovery is to Read and then Write, doubling the cost of the common path |
| **C. A gate that warns rather than refuses**: allow the write, append a sentence to the result | Almost free. One extra sentence in the result | The model learns it overwrote something it never read, and the trace shows it | The file is already gone. A warning after a destructive act is a log line, not a safeguard |
| **D. A gate scoped to destruction**: refuse only when the existing file is non-empty and was never read | Narrower than B, so fewer false refusals | Catches the case that actually loses work | Two rules instead of one, and "non-empty" is a heuristic |
| **E. A content-hash gate fed by facts the read tool already published**, rig's `read_before_edit` | A hash in the facts `read` and `write` publish, and a plugin that compares it with the disk | No state inside the tool, a new file is never gated (`!path.is_file()`), and the hash is of what was actually read | Not shipped in rig, and its branch for `write` over an existing file has no test (`read_before_edit/tests.rs:75,104`). It clears on compaction, and a formatter that rewrites the file after a `write` makes the next gated write refuse (inferred) |

**Recommendation: B, but only if you also keep a `Read`-side record that the gate can consult, and only if you make one escape hatch explicit: a `Write` to a path that does not exist never needs a read.** Confidence medium. Reason: a harness that lets a model silently destroy a file is a harness people stop trusting, but a gate without a documented escape hatch generates its own category of stuck runs.

A gate costs some fidelity, because Cursor's success sentence is the same whether the model read the file or not, so a model trained on Cursor has no expectation of being refused. If you gate, expect some models to be surprised once per session and then adapt.

Other harnesses. OpenCode's description promises the gate (`This tool will fail if you did not read the file first`) and its code has none, which is the `dsv3-1018` situation. Claude Code is the only one read that enforces one, and Copilot gets the same safety by refusing to overwrite at all. Option E moves the gate out of the tool. The recommendation stays B and the confidence stays medium, because rig's version has never been tested on the case that matters, a `write` over an existing file. `(source)`

### D5. Do you keep Cursor's three-line-ending-rules design, or simplify?

The rules, in priority order: incoming content that already contains `\r\n` is left completely alone; a file with mixed endings gets a per-line merge; a CRLF file has its bare LFs rewritten; otherwise nothing happens.

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. Keep all three**, including the per-line merge | You must port an LCS alignment with a 4,000,000-cell cap and an offset backfill pass, and you must test it. Q4 has the algorithm | Round-tripping a mixed-ending file does not silently flatten it. This is the only rule that makes per-line preservation possible | Complexity for a case most repos do not have. And the merge has sharp edges of its own, documented in the traps |
| **B. Drop the merge, keep rules 1 and 3** | A mixed-ending file gets flattened to its dominant style on first write | Two rules instead of three, no LCS, no cap, no backfill | You silently normalise a file. The parent spec's own probe case (`aa\r\nbb\ncc\r\n` plus `mm\nnn\noo\n` becoming `mm\r\nnn\noo\r\n`) stops holding, and so does the round trip |
| **C. Normalise everything to one style, configurable** | One rule. `.editorconfig`-style configuration instead of detection | Predictable, and it is what half of Cursor's own bug reports want | Every file the tool touches changes line endings on first write, which makes diffs enormous and blame useless |
| **D. Preserve per line using the model's own terminators, no alignment** | Cheap: if the content has no CRLF at all, you cannot know which line maps to which | No LCS | It is not preservation, it is a guess |
| **E. One style for the whole file, picked from the old file**, Gemini's and Zed's rule | One classifier. Gemini: any `\r\n` in the old file. Zed: the first newline in the first 1000 bytes | No LCS, no cap, no backfill, and a CRLF file stays CRLF | A mixed file is flattened, and the dominance rule is a choice: Cursor's is at least 5% CRLF, Gemini's is any CRLF, Zed's is the first newline |

**Recommendation: B for a first version, with A behind a flag you turn on once you have tests.** Confidence medium. Reason: the mixed-ending merge is the hardest thing in the tool and the least common case, but dropping it silently changes the round-trip property, so the drop has to be a documented decision rather than an accident.

If you do implement A, note the two findings in Q4: the merge is not a plain Myers diff, and the backfill pass fills gaps in a way that can give an unrelated line the wrong terminator when the new and old line counts match.

If you take B, option E says the dominance rule has to be chosen on purpose, since three harnesses chose three rules and Cline's `insert` flattens a file to one ending. Gemini also never contracts: a CRLF payload sent to an LF file is written as is. `(source)`

### D6. Where do you draw the atomicity of the "read old content, compute diff, gate, write" sequence?

Cursor's sequence has the read at the tool layer, the gate at the write-wrapper layer, and the fingerprint check at the executor layer, so the file has three chances to change.

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. Cursor's shape**: read, diff, gate, fingerprint-recheck, write | One hash of the file, computed twice | Detects an external writer in the window. Cheap | The fingerprint is keyed on `toolCallId` plus resolved path, and the record is consumed by `take` - so a retry needs a fresh read, which the message tells the model to do |
| **B. One lock around the whole sequence**, keyed on the canonical path | The lock is held across a read, a diff, and a write, which is longer than Cursor holds it | No window at all inside one process | A long diff on a big file blocks every other tool on that path. And it does nothing about an external writer |
| **C. Compare-and-swap**: pass the expected content hash into the write | A hash parameter the caller must supply, so the tool has to compute and carry it | The check and the write are one operation from the caller's point of view | Still not atomic at the filesystem, and the tool computes the hash itself so it is not really a CAS |
| **D. None of it**: read, then write, accept the window | Zero cost | Zero | A concurrent formatter, a watcher, or a second agent silently clobbers |
| **E. Compare-and-swap against the hash the tools last saw**, rig's `Seen` plus `write_if` | A soft cache of hashes, filled from the reads and writes the tools already do | A CAS against the previous observation, and on a mismatch the current content comes back so an edit can be re-applied | The model never supplies the hash, so it only protects a file the harness has seen; rig's `write` does not call it, only `edit` does (`workspace.rs:106-118`, `edit.rs:183-213`) |

**Recommendation: A, plus B keyed on the canonical path.** Confidence high. Reason: the fingerprint catches the external writer and the lock catches the in-process one, and they are orthogonal.

If you take B, key the lock on the *canonical* path. Cursor keys it on the raw argument string, which is the one clear bug in its locking, and you gain nothing by reproducing it. Pi's queue is the model for the lock, with two caveats from the harness section: a file that does not exist yet has nothing to canonicalize, and call order needs a registration chain.

Rig shows that option C is stronger than the table says. The held hash comes from an earlier read or write that was recorded, so it is a compare-and-swap against the previous observation, with no hash computed by the tool in the same call. The recommendation stays A plus B, with E as the follow-up and the confidence stays high. `(source)`

### D7. Should `Write` and `StrReplace` share the read path?

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. Share**, as Cursor does: both call the same read executor and both go through the same write executor | One read path to get right, one write executor, one set of encoding rules | Consistency. A file written by `StrReplace` and a file written by `Write` end up with the same encoding treatment | The error strings diverge anyway - Write says `Write permission denied:` where StrReplace's read stage says `Read permission denied:` - so sharing the path does not give you shared messages |
| **B. Separate** | Two encoders, two line-ending implementations, two chances to disagree | Each tool's messages and rules can be tuned independently | You will fix a bug in one and forget the other. The failure mode is a workspace where `Write` preserves UTF-16 and `StrReplace` does not |

**Recommendation: A for the mechanism, B for the strings.** Confidence high. Reason: the encoding and line-ending machinery must be one implementation or it will drift, and the user-visible strings must be per-tool because Cursor's already are.

### D8. What does the result text say?

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. `Wrote contents to <path>`**, Cursor's exact sentence | The model gets no confirmation that the bytes match what it sent, and no line count | Matches the surface models were trained on. Minimal tokens. The client renders the diff | The model has no way to notice that the file was re-encoded or that line endings changed |
| **B. Add a line count or a byte count** | A few tokens per call, and a number the model may try to reason about | The model can sanity-check that something landed | Diverges from Cursor, and the model will occasionally use the number to justify a wrong claim |
| **C. Add a short note when the write was transformed** - encoding changed, line endings rewritten, BOM added | One conditional branch and a sentence | The model learns the file is not byte-identical to what it sent, which is the information Q10 says is missing | More surface to keep byte-exact against nothing, since Cursor has no equivalent |
| **D. Return the diff** | Every write costs the model the size of the file | The model sees what changed | This is what `StrReplace` is for. Putting it in `Write` makes `Write` the expensive tool. Gemini does it anyway, with a snippet around the change and the whole file for a new file |
| **E. Created or replaced, with the old line count**, rig's reply | The old content read from inside the same operation, before the replace | The one fact the model cannot know without another call, at no extra round trip: `Created x.` or `Replaced x, which had 412 lines.` | A count to define. Rig counts `\n` bytes plus one for an unended last line, Goose counts `str::lines()`, and neither counts what the model might assume |

**Recommendation: A for the base sentence, with E and C as additions you make deliberately.** Confidence medium. Reason: the base sentence should stay byte-identical to Cursor's because that is free fidelity, and C and E are the pieces of information the model cannot get any other way - but adding either is a divergence and should be a decision rather than a drift.

What moved this: five other harnesses already tell the model whether it created or replaced, rig did it after dropping `Wrote {N} bytes to {path}.` (commit `e034438fa`), and pi dropped its own byte count in `e583b290a` because the number was UTF-16 code units. Option B's count is a trap unless its unit is checked (trap 15). `(source)`

### D9. What does `Write` do when its view of the old file, or its own `contents`, may be incomplete?

This is the safety asymmetry, and skipping it costs more than skipping any other decision here. `StrReplace` reads `truncated` on its read result and refuses with `File is too large to edit`. `Write`'s execute closure contains no reference to `truncated` at all, so the tool that replaces the whole file is the one that does not check whether its view of the old file was complete. Two independent holes sit behind that: the view of the OLD file can be partial (a read cut at 8 MiB), and the NEW `contents` can be partial (the model's message stopped on the output-token limit in the middle of the string). Both end in the same place, a file replaced by a prefix and a success sentence.

| Option | Cost | Buys | Breaks |
|---|---|---|---|
| **A. No check on either**, Cursor's `Write` | Zero | Nothing | A cut read or a cut message replaces the file with a prefix and reports `Wrote contents to <path>`. `(bundle)` |
| **B. Refuse on a truncated read**, as `StrReplace` does | One flag passed from the read to the write | The 8 MiB case, where the tail of the file is silently lost, closes | Does nothing for the new-contents hole |
| **C. Fail every tool call from a message that stopped on the output limit**, pi's rule | One retry, and complete sibling calls in the same message fail too | A truncated `contents` cannot replace a file. The guard runs before lookup and validation, so a salvaged JSON that validates is still refused | The harness must see the stop reason. A server that closes partial JSON before the client sees it hides the cut from any check that lives only in the tool |
| **D. C plus advice on how to split**, rig's kernel refusal and `auto_continue` | The same, plus a message | The model learns to write the first part and add the rest with edits | Only useful where an edit or append tool exists. `Write` has no append |
| **E. Cap `contents`**, Cline's 6,000 characters | A limit and a split protocol | A cut becomes unlikely | Forces chunked files, and the cap is a proxy that a direct caller can bypass |
| **F. Refuse placeholder-elided content**, Gemini's detector | A per-line prefix list | Catches `rest of methods ...` | Does not catch a plain cut, and has false positives for files that discuss placeholders |

**Recommendation: B and C together.** Confidence high. Reason: the two holes are independent, both lose the whole old file, and each guard is a few lines. It leads the report because pi and rig each built a message-level guard for the second hole, and Cursor's own `StrReplace` already checks for the first. Claude Code closes the first hole differently: its gate refuses a `Write` after a partial read (`!s || s.isPartialView`), and pi's guard is the model for the second. `(source)`

Cursor's wrapper does one thing that helps. It parses the buffered arguments with a strict `JSON.parse` before running anything, so a cut that leaves the JSON open fails with a syntax error and no write. `(bundle)` in `gbe` at 4154498, `(daemon)` in `withStreamingEditParsedArgs` at 12536500. What it cannot see is a server that closes the partial JSON first, and the loop's own handling of an output-limit error (`OutputTokensLimitExceededError`, retried with a reminder to break the work into smaller pieces, `(daemon)` at 7396447 and 11701259) is an error-driven retry and not a check on a tool call. Neither is in the Write path. In Runvil the guard for the second hole belongs in the host before any Pack tool runs, because the Pack never sees the stop reason.

## The contract

Everything a caller can observe, and nothing about how it is done.

### Tool name, by branch

| Version tag | Model-facing name | Internal id |
|---|---|---|
| `dsv3-1018` | `write` | `WRITE` |
| `cursor-0226` | `Write` | `WRITE` |
| `dsv3-1205` | `Write` | `WRITE` |
| `latest` | `Write` | `WRITE` |
| `gpt5-codex` | `Write` | `WRITE` |
| `codex-cloud` | `Write` | `WRITE` |
| `haiku` | `Write` | `WRITE` |
| anything unlisted | `Write` | `WRITE` |

`(bundle)` The name is `const l="dsv3-1018"===n?"write":"Write"` at 4232300, and unlike most sibling tools this factory does not throw on an unknown version - it falls through to `Write`. There is no `executionAliases` on either registration, so `write` and `Write` are not interchangeable; `SHELL` declares `executionAliases:["bash","Bash"]` and `WRITE` declares none. `(daemon)` `createWriteTool`

Two `AG("WRITE", ...)` registration sites exist at 4232349 and 4233029 in this build, against 4217017 and 4217697 in the spec's build. Both sit inside the one factory `hke`, and the first is behind `if("dsv3-1018"===n) return ...`, so exactly one registration happens per call. The spec's account is confirmed: these are mutually exclusive branches of one factory, and neither wraps the other.

### Parameters, verbatim

`dsv3-1018` branch, schema object `mke` at 4230054:

```js
file_path: string, required
    .describe("The path to the file to modify. Always specify the target file as the first argument. You can use either a relative path in the workspace or an absolute path.")
contents:  string, required
    .describe("The contents of the file to write")
```

Default branch, schema object `pke` at 4229910:

```js
path:      string, required
    .describe("The absolute path to the file to modify")
contents:  string, required
    .describe("The contents to write to the file")
```

| Field | Type | Required | Default | Constraint |
|---|---|---|---|---|
| `path` / `file_path` | string | yes | none | none - no `.min()`, no pattern, no absolute-path check |
| `contents` | string | yes | none | none - no `.min()`, so `""` validates |

`(bundle)` `U$.Ikc({...})` builds a non-loose object, so an extra key is a validation failure. `(daemon)` `object({...})` from zod.

There is no `explanation`, no `line_range`, and no `old_str`/`new_str` anywhere in either schema. The content field is `contents`. The Anthropic-native `str_replace_editor` shape uses `file_text`, and public reimplementations keep adding aliases for it (see the traps).

### Description strings, verbatim

`dsv3-1018`, at 4232400:

```text
Writes a file to the local filesystem.

Usage:
- This tool will overwrite the existing file if there is one at the provided path.
- If this is an existing file, you MUST use the read_file tool first to read the file's contents.
- ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
```

`cursor-0226`, at 4233100:

```text
Writes a file to the local filesystem.
```

`isComposer15`, at 4233200. Note the U+2019 apostrophe in `isn’t`:

```text
Writes a file to the local filesystem.

Usage:
- This tool will overwrite the existing file if there is one at the provided path.
- If this is an existing file, you MUST use the read_file tool first to read the file's contents.
- ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
- The write will FAIL if path isn’t given as the first argument. Always provide path first.
```

Default, at 4233350:

```text
Writes a file to the local filesystem.

Usage:
- This tool will overwrite the existing file if there is one at the provided path.
- ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.
- NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
```

The four branches show three behaviours:

| Branch | Prior read demanded? | Anything else |
|---|---|---|
| `dsv3-1018` | yes, by name `read_file` | four bullets |
| `isComposer15` | yes, by name `read_file` | four bullets plus the "path first" warning |
| `cursor-0226` | no | one line, no bullets at all |
| default | no | three bullets |

`dsv3-1018` and `isComposer15` are told a read is mandatory and it is not enforced. The default branch is told nothing about reading and is also not enforced. A model on the default branch has no instruction to read before overwriting, which makes it the branch most likely to blind-overwrite.

### Which model families get the tool at all

`(bundle)` `tJe` at 4581853:

```js
switch(e.modelInfo.vendor){
  case"anthropic":case"google":case"xai":case"zai":case"unknown":t.push(n()),t.push(r());break;
  case"openai":case"cursor":e.modelInfo.isGpt5Family?t.push(s()):(t.push(n()),t.push(r()));
```

`n()` is `ake` (StrReplace), `r()` is `hke` (Write), `s()` is `NSe` (ApplyPatch). So:

| Vendor | Gets `Write`? |
|---|---|
| anthropic, google, xai, zai, unknown | yes, alongside `StrReplace` |
| openai or cursor, GPT-5 family | no - gets `ApplyPatch` instead |
| openai or cursor, not GPT-5 family | yes |

`Write` is not a universal tool, and that matters for a reimplementation in a way the parameter table does not show. For GPT-5-family models Cursor replaces the whole write path with a patch format, which means those models have never been trained on `Write` at all in Cursor's own product. If your Pack offers `Write` to every model, a GPT-5 model sees a tool Cursor deliberately withheld. That is not necessarily wrong, but it is a divergence.

### Caps and limits

| Limit | Value | Unit | Layer that enforces it |
|---|---|---|---|
| `contents` length | none | - | nothing rejects an oversized write |
| Old-content snapshot for hooks and tracking | 50 MB, `MAX_OFS_FILE_SIZE = 50*1024*1024` | bytes on disk | `LocalWriteExecutor`, skips the snapshot above it |
| Old-content snapshot for the `afterFileEdit` hook | 262,144 | bytes | the hook wrapper, separate from the 50 MB |
| `beforeFullFileContent` / `afterFullFileContent` in the result | 524,288, `EDIT_FULL_FILE_CONTENT_MAX_CHARS = 512*1024` | JS string length | `editTransformResultToEditResult` |
| Diff computed on a worker thread | above 262,144 combined old+new | JS string length | `DIFF_SIZE_THRESHOLD` |
| Fingerprintable file | 8 MB, `MAX_TEXT_SIZE` | bytes | `fingerprintFile` returns undefined above it, which silently disables the stale check |
| Encoding cache | 100 entries, LRU | paths | `setFileEncoding` / `MAX_FILE_ENCODINGS` |
| Format detection sample | 8192, `FORMAT_HEADER_BYTES` | bytes | `readFirstBytes` |
| Read cap the tool inherits | 100,000, `READ_CHAR_HARD_LIMIT` | JS string characters | the read path, so a file over this arrives as `exceededLimit` |

Above 8 MB the stale-write guard silently stops working, because `fingerprintFile` returns `undefined` for a file over `MAX_TEXT_SIZE` and `fileChangedSinceRead` treats `undefined` as "no change". So the tool is protected against concurrent writers only for files it can hash, and it does not tell the model that the protection lapsed. `(daemon)` at 16988590, `(daemon)` `MAX_TEXT_SIZE=8*1024*1024` at 18017.

### Result cases

The success record is a proto `EditSuccess` carrying `path`, `linesAdded`, `linesRemoved`, `diffString`, `beforeFullFileContent`, `afterFullFileContent`, `message`. The model sees only `message`, through the renderer `ybe` at 4157158:

```js
case"success":return JB(t.value.message??"",!1);
```

So on success the model sees:

```text
Wrote contents to <path>
```

with `isError: false` and nothing else: no trailing newline, no count, no diff, no path resolution.

Failure cases, with the exact model-visible text, are in the failure matrix below. Two of them change what the model can tell apart:

- Two different failures produce text that begins with `Error: `. The renderer's `case"error"` branch prefixes it, and the parent spec's worked examples quote `Error: File content is binary data, not text: ...`, so a reader skimming the strings can miss that the renderer adds the prefix.
- The success sentence reports the raw argument path, while the structured `path` reports the resolved absolute path. Anything reading the structured record and anything reading the text can disagree about what file was written.

### The `pi_*` counterpart

`pi_write` is a second write tool registered in the same bundle: a port of the pi coding agent's `write`, with its own schema, its own one-string result, and Cursor's `Write` executor underneath. The whole account is under pi in the harness section, headed by the comparison of pi's original with this port, and holding the registration and schema blocks, the corrected queue and path claims, and the truncated-message guard. The `pi_write` content key is `content` where `Write` says `contents`. It runs under a canonical-path mutation queue instead of the streaming lock. It calls the write executor directly, so it skips `performWrite` and its plan-mode gate and runs no read. Its success text, `Successfully wrote ${content.length} bytes to ${path}`, is a stale copy of an older pi that prints UTF-16 code units as bytes. `(bundle)` `(daemon)`

## Algorithm and ordering

The write is a read-modify-write, and the modify happens inside the tool. That is why no model-facing read requirement can be enforced: the tool does not need one.

Both `Write` and `StrReplace` end in the same write executor, `Rbe`, and that executor's first statement is the plan-mode gate. The difference between the two tools is what each does *before* reaching it: `Write`'s execute body reads the file, computes the diff with `Lbe`, builds the success sentence, and only then calls `Rbe`, so a refused plan-mode write still performs the read and the diff. `StrReplace`'s wrapper `Sbe` calls the same gate as its own first statement, before its read, so the early check fires and the one inside `Rbe` is normally unreachable. So the order inside `Rbe` is identical for both tools - gate, write, record state - and the read, transform, and diff happen above it, with `StrReplace` adding one extra gate ahead of its read.

Steps marked **(order-critical)** fail differently if you move them.

### The streaming prelude

The tool is not called with a parsed object. It is called with a stream of JSON fragments, and the lock is taken during that stream.

```
1  EMIT an empty partial tool call so the client can render a spinner
     (order-critical) this happens before any argument is seen
2  PARSE fragments incrementally, watching for the path field
     ON THE FIRST TIME the path field completes:
         if it differs from the last value seen:
             lock = await lockManager.waitForLock(ctx, rawPathValue)
             (order-critical) the key is the RAW STRING, before any
                              normalization, so two spellings of one
                              file are two locks
             emit a partial tool call carrying only the path
     ON every completed fragment of the `contents` field:
         emit a streamed content delta for the client
3  WHEN the stream ends:
         IF no lock was ever acquired
             THROW "Failed to acquire file lock - path may not have been parsed"
             (order-critical) this fires BEFORE schema validation would
                              report a missing `path`, so a call with no
                              path reports the lock error, not the
                              argument error
4  VALIDATE the accumulated JSON against the branch schema
         failure -> THROW "Invalid arguments:\n<key>: <message>"
5  CALL the execute body with the parsed args
```

Correction to the block above, which is kept as first written: steps 3 and 4 run in the other order in both binaries. Step 4's validation, preceded by `JSON.parse` of the whole buffered string, runs first and returns the `Invalid arguments` text on failure; step 3's no-lock refusal runs after it. The lock acquisition of step 2 is unchanged and still precedes validation. `(bundle)` `(daemon)`

`(bundle)` `gbe` at 4154498. `(daemon)` `withStreamingEditParsedArgs` at 12536500.

The lock is released by `Symbol.dispose` in a `finally`, so it is held across the read, the diff, the gate, and the write. A slow diff on a large file therefore holds the lock for its duration.

The "no lock" error is a *stream* failure, and it runs after validation. Both the bundle and the daemon evaluate `schema.safeParse(JSON.parse(bufferedArgs))` first and return `Invalid arguments` on a schema failure, and only then test whether a lock was ever taken (`gbe` at 4154498, `withStreamingEditParsedArgs` at 12536500). So a model that emits `{"contents":"x"}` with no path sees `Invalid arguments:\npath: Required`, and the lock message is reachable only when the arguments validate and no path value completed during streaming, which a required string `path` makes practically unreachable. The lock itself is still taken during streaming, before validation, so a call that later fails validation held the file lock for the duration. An earlier version of this report said the lock check runs first and a missing `path` reports the lock message; both binaries show otherwise. `(bundle)` `(daemon)`

### The execute body

```
6  rawPath  = lke(args.file_path ?? args.path)
   contents = args.contents
     lke is a typo repair, not a normalizer: it rewrites the literal
     "worktries" to "worktrees" but only under a /Users/<name>/.cursor/
     or /home/<name>/.cursor/ prefix, so on Windows it never fires
     (order-critical) this runs BEFORE the read, the diff, and the
                      message, so the repaired path is what the model
                      sees echoed back
7  read = service.read(rawPath)
   execId = <toolCallId>-read          // deterministic, sha256-shaped UUID
     SWITCH read.result.case
       success + output.case=="content"  -> old = output.value
       success + output is not text      -> THROW "File is not a text file (binary data)"
       fileNotFound                      -> old = undefined   // new-file path
       permissionDenied                  -> THROW "Read permission denied: <path>"
       error                             -> THROW the service error text
       rejected                          -> THROW a rejection carrying the reason
       invalidFile                       -> THROW "<reason>: <path>"
       none                              -> THROW "Unknown error"
       default                           -> THROW "Unhandled readResult.result: <result>"
8  diff = computeDiff(old ?? "", contents,
                      fileChangeType = old === undefined ? "added" : "modified",
                      tool = "write")
   message = "Wrote contents to " + rawPath
     (order-critical) the message is built here, from the raw path, and
                      carried through every later layer unchanged
9  writeResult = performWrite(rawPath, contents, {message, diff, counts, old})
```

`(bundle)` `hke`'s execute closure at 4231080, `(daemon)` `createWriteTool` at 12632008.

Step 7 is where a reimplementation learns that `Write` inherits `Read`'s behaviour wholesale. Everything the read path does - the 100,000-character cap, the binary sniff, the image decoder, the blob spill - applies to the old content of a write. If the old content exceeds the cap, the read returns `success` with `exceededLimit: true` and empty content, and `Write` takes the `success + content` branch with `old = ""`. So the diff is computed against an empty string and the tool reports an enormous number of added lines, while the write itself proceeds and truncates the real file. That behaviour is unpleasant; check it against a build before you rely on either reading of it, because the bundle shows the mechanism but not a probe.

**The asymmetry with `StrReplace` is definite rather than inferred.** `StrReplace`'s wrapper `Sbe` reads `u.result.value.truncated` after the read and throws `File is too large to edit` when it is set, so a partially-read file is refused. `Write`'s execute closure contains no reference to `truncated` at all: the switch handles the six result cases and then takes `output.value` as the old content. So the tool that overwrites the whole file is the one that does not check whether its view of the old content was complete. A file that the read executor cut at 8 MiB is diffed and overwritten from the truncated view, and the remainder of the file is lost. That is a stronger statement than the `exceededLimit` paragraph above, which depends on a mechanism that may not reach this caller. It needs a test of its own: write a file just over 8 MiB and check whether `Write` succeeds silently.

### The write wrapper

```
10 plan-mode gate
     IF mode == PLAN and not (markdown suffix or canvas regex)
         THROW "Cannot edit non markdown files in plan mode"
     (order-critical) this is AFTER the read and AFTER the diff, so a
                      refused plan-mode write has already paid for both
11 telemetry increment
12 writeResult = writeExecutor.execute(rawPath, contents)
13 SWITCH writeResult.case
     success          -> recordFileState(resolvedPath, contents, old)
                         return {modified, edit:{...}}
     permissionDenied -> THROW "Write permission denied: <path>[: <error>]"
     noSpace          -> THROW "No space left on device"
     rejected         -> THROW a rejection carrying the reason
                         (order-critical) the model sees `Rejected: <reason>`
                         via the renderer, and `Error: <reason>` when the
                         reason names a missing tool-call context, because
                         that arm throws a plain ToolCallError instead
     error            -> THROW the error text, with two special cases:
                         the worktree guard, and an unresolved notebook
                         view type which gets its own long message
     none             -> THROW "Unknown error"
     default          -> THROW "Unhandled writeResult.result: <result>"
```

`(bundle)` `Rbe` at 4163000, `(daemon)` `performWrite` at 12515662.

One more asymmetry in the hook path, verified at 9639000 in the bundle: the `afterFileEdit` hook's before-content snapshot has its own 262,144 byte cap, separate from the executor's 50 MB. When the file is over that cap, or the stat failed, the hook receives `edits: [{old_string:"", new_string:<the whole new content>}]` rather than a real diff, because there is nothing to diff against. And when a before-content is available, the diff is computed with `context: 0` and a 50 ms timeout, so a large or slow diff silently degrades to the same whole-file pair.

Step 13's `rejected` branch has a two-layer split that is easy to get backwards. `(daemon)` at 12516600:

```js
if(writeRejectedReason.includes("Failed to find tool call context")){ ... throw new ToolCallError({clientVisibleErrorMessage:`Edit rejected: ${writeRejectedReason}`,modelVisibleErrorMessage:writeRejectedReason,error:`Edit rejected: ${writeRejectedReason}`}) }
```

For that one condition the *client* gets `Edit rejected: <reason>` and the model gets `Error: <reason>` - the bare reason, wrapped in the renderer's `Error: ` prefix because the throw is a plain `ToolCallError` taking the `error` case rather than the `rejected` case.

For every other rejection the throw is a `WriteRejectedError`, constructed as `(path, reason, \`Edit rejected: ${reason}\`)`. Its constructor signature is `(e, t, r)` with `clientVisibleErrorMessage: r` and `modelVisibleErrorMessage: t`, so the client-visible field carries `Edit rejected: <reason>` and the model-visible field carries the bare reason. But the proto record that `serializeError` builds carries only `{path, reason}`, and the renderer's `rejected` case reads `reason` and wraps it itself:

```js
case"rejected":return t.value.reason===G_e?JB(z_e(r.allTools),!0):JB(`Rejected: ${t.value.reason}`,!0);
```

So `modelVisibleErrorMessage` is discarded on this path, and the model sees `Rejected: <reason>`.

Which means `Edit rejected:` is never the model-facing text on this tool; it is the client-visible form. The model sees `Rejected: <reason>` in the ordinary case and `Error: <reason>` in the missing-tool-call-context case. Anyone reading the daemon and concluding that the model sees `Edit rejected:` is reading the `clientVisibleErrorMessage` field.

The parent spec's failure matrix lands on the right string for the ordinary case (`Rejected: ${reason}`) with a note that it is "No `Edit rejected:` prefix, which is the StrReplace form". The string is right and the reasoning is not: there is no `Edit rejected:` model-facing form on either tool, and the two tools reach `Rejected: ` the same way.

### The executor

```
14 IF the path starts with "file://" and does not parse as a file URL
       return error "<path> is not a valid file URL. Use an absolute path instead."
     (order-critical) a malformed URL is rejected; a well-formed one is
                      converted later, in the resolver
15 resolvedPath = resolve(path, workspacePath)
     1. if it starts with "file://" and parses, convert with fileURLToPath
     2. if it starts with "~" followed by / or \ or end, expand to homedir
     3. if relative, resolve against the workspace path; else resolve
16 taken = toolCallReads.take({toolCallId, resolvedPath})
     this CONSUMES the fingerprint recorded by step 7's read
     if the write never starts, the fingerprint is restored
17 guard = isInternalArtifactPath(resolvedPath, ["assets","agent-tools","swarm-agents"])
18 IF the target is in another worktree
       return error "You cannot search other worktrees for this repository, stay within your workspace paths."
19 IF stat(resolvedPath).size <= 50 MB
       beforeContent = readText(resolvedPath, encodingHint)
   ELSE
       beforeContent = undefined; existingFileTooLarge = true
20 realpath with new-file fallback:
     existing path          -> the canonical realpath
     dangling symlink       -> the literal absolute path, status danglingSymlink
     missing leaf           -> walk up to the nearest existing ancestor,
                               realpath it, rejoin the missing segments
     (order-critical) the mkdir target and the write target are both this
                      resolved path, not the raw argument
21 permission gate
     a path whose basename is ".cursorignore" is forced to permissionsConfig
     otherwise shouldBlockWrite(ctx, resolvedPath, contents)
       needsApproval  -> approval card {path, reason, isNewFile, diffString,
                                         before, after, blockReason}
                         a denial returns rejected with the user's reason
       deny           -> permissionDenied
     (order-critical) inside the approval callback the fingerprint is
                      re-checked, so a file that changed while the user was
                      deciding is refused rather than clobbered
22 IF the recorded fingerprint says the file existed and the parent
   directory is now gone
       return error "<path> changed on disk after this tool call read it. ..."
23 mkdir -p dirname(resolvedPath)
     EACCES or EPERM -> permissionDenied with operation "create_directory"
     (order-critical) this is AFTER the directory check, so a removed
                      directory is refused rather than silently recreated
24 buffer = encodeForWrite(contents, existing format of resolvedPath, platform)
25 guarded write:
       IF the fingerprint no longer matches
           throw the stale message
       writeStarted = true
       writeFileNoFollow(resolvedPath, buffer, {flush:true})
       fsync the containing directory (best effort)
26 SWITCH the failure, if any
     SymlinkRefusedError -> permissionDenied, operation "create_file"
     EACCES or EPERM      -> permissionDenied "Permission denied"
     ENOSPC               -> noSpace
     EDQUOT               -> error with the agent-store quota message
     anything else        -> error with the raw message
27 trackChange(resolvedPath, beforeContent, contents, {toolCallId})
     only when: no fileBytes, not an internal artifact path, and the
     existing file was not too large to snapshot
28 onWriteForAiTracking({path, beforeContent, afterContent, metadata})
     failures are swallowed with a warning
29 return success {path: resolvedPath, linesCreated, fileSize}
```

`(bundle)` `fC` at 9317565, `(daemon)` `LocalWriteExecutor` at 16998591. The out-of-workspace approval is at 9097382 in the bundle and 16682742 in the daemon; both agree on the reason string `Out of workspace` and the block reason `outOfWorkspace`.

Step 24 is the encoding step, and steps 24 through 29 are where a Rust port diverges fastest.

### The write primitive

```
30 fd = open(canonicalPath, O_WRONLY | O_CREAT | O_NOFOLLOW | O_CLOEXEC, 0o666)
     ELOOP, EMLINK, or EPERM-with-NOFOLLOW
         -> SymlinkRefusedError "Refusing to write through a symlink at <path>"
31 IF realpathSync(canonicalPath) != canonicalPath
       -> SymlinkRefusedError "Refusing write: <path> now resolves through
          a symlinked parent to <real>"
32 IF !fd.stat().isFile()
       -> SymlinkRefusedError "Refusing to write to non-regular file at <path>"
33 truncate(fd, 0)
34 write in a loop until the buffer is exhausted, breaking if a write
   returns zero bytes
35 IF flush: fsync(fd)
36 close(fd)
```

`(bundle)` `uC` at 9315126, `(daemon)` `writeFileNoFollow` at 16995900.

Steps 33 through 35 are the durability story, and they have no temp file, no rename, and no rollback. Between the truncate and the last successful write, the file on disk is a prefix of the new content. If the process dies there, the old content is unrecoverable from the file itself.

Step 34's `if(bytesWritten===0)break` is a silent partial write. A zero-byte write from `fs.write` on a regular file means the descriptor is in a state it should not be, and Cursor breaks out of the loop and returns success, so a partial write can be reported as a successful one. Whether that is reachable is unknowable statically, but the code treats it as done rather than as an error.

### The encoding decision

This is `getBufferForWrite` in the daemon, and it is the part a reimplementation gets wrong.

```
37 format = detect(resolvedPath)   // see Q3 for the detection order
   IF detection threw:
       if the extension is in BINARY_EXTENSIONS  -> format from the extension
       else                                     -> platform default text format
   (order-critical) the extension fallback runs only when detection THREW,
                    not when detection returned "binary"
38 IF format.isBinaryFile:
       skip the BOM and encoding logic entirely, go to step 42
39 resolvedEncoding =
       encodingHint, if the caller supplied one
       else the per-path cached encoding, if any
       else format.encoding
           and if that is utf8 or utf8bom, re-check the whole file:
           a file that is not valid UTF-8 but decodes cleanly as latin1
           is downgraded to latin1
40 writeUtf8Bom =
       resolvedEncoding == utf8bom
       OR (resolvedEncoding == utf8 AND no explicit hint
           AND the file currently starts with EF BB BF)
   unicodeBom = the BOM byte pattern for the mapped encoding
   IF writeUtf8Bom                       -> bom = EF BB BF
   ELSE IF unicodeBom AND file starts with it -> bom = unicodeBom
   ELSE                                  -> bom = none
41 cache the chosen encoding for this path, LRU, 100 entries
42 line endings:
     a  IF contents contains "\r\n"          -> leave contents alone
     b  ELSE IF the existing file is mixed   -> per-line merge (Q4)
     c  ELSE IF the detected ending is CRLF  -> replace /(?<!\r)\n/g with "\r\n"
     d  ELSE                                 -> leave contents alone
43 IF format.encoding == "binary":
       buffer = base64DecodeIfItLooksLikeBase64(contents) else latin1(contents)
       return
44 body       = bom ? stripUtf8Bom(contents) : contents
   writeEnc   = format.encoding == utf8bom ? utf8 : format.encoding
   restored   = writeEnc == utf8 ? tryRestoreUndecodableBytes(existingBytes, body) : none
   encoded    = restored ?? encode(body, writeEnc, {defaultEncoding: utf8})
45 return bom ? concat(bom, encoded) : encoded
```

`(daemon)` `getBufferForWrite` at 15433367.

Four points in that sequence are not obvious, and each needs a test:

- Step 39's latin1 downgrade reads the *whole* file. Detection samples 8192 bytes, and then the encoding choice re-reads everything. A file whose first 8192 bytes look like valid UTF-8 and whose remaining bytes do not gets classified as UTF-8 by the header and then, only if the whole file fails `isUtf8`, reconsidered as latin1, so the header classification is not final.
- Step 40's `writeUtf8Bom` consults the file's *current first three bytes*, so a BOM is preserved by re-reading the target rather than by trusting the detected format. Two reads of the file happen before the write: one for the format, one for the BOM check, and possibly a third for the latin1 downgrade and a fourth for the undecodable-byte restore.
- Step 44's `stripUtf8Bom(contents)` strips a BOM from the *model's* content. If the model sends a string that starts with U+FEFF, it is removed before encoding and then re-added only if `bom` is set. So a model that deliberately sends a BOM to a BOM-less UTF-8 file loses it.
- Step 45's `restored` path does the work behind Q10's answer. If the model's content contains U+FFFD and the original file was not valid UTF-8, the encoder tries to splice the original invalid byte runs back in place of each replacement character, so a read-then-write-unchanged round trip on a latin1 file is byte-identical. It is a heuristic: it requires the count of U+FFFD in the new text to equal the count of undecodable sequences in the original, and falls back to plain UTF-8 encoding when they disagree.

## Engineering questions

### Q1. Pagination and chunking

`Write` does not page. There is no offset, no limit, no cursor, and no way to append to a file through this tool. A caller that wants to write 2 MB of content sends 2 MB of content, and the tool writes all of it or fails.

The one place a boundary can fall mid-thing is the argument stream. The content field is streamed as a series of JSON fragments, and Cursor parses them incrementally to emit partial tool calls for the client. Two things are protected and one is not:

| Boundary | Protected? | How |
|---|---|---|
| Mid-string, inside the JSON escape sequence `\u00` | yes | the incremental parser only reports completed values |
| Mid-UTF-8 sequence | yes, in principle | the fragments are JS strings already, so a code point has been assembled before the value is reported |
| Mid-content, for the client's live display | no, by design | `emitToolCallDelta` sends whatever fragment landed |

A caller *cannot* resume a write. There is no partial-write state to resume from either: the write either ran to completion or it did not run at all, because the truncate happens once, at the end, after everything else.

`(bundle)` the stream parser is `gbe` at 4154498; `(daemon)` `withStreamingEditParsedArgs`.

For your harness: a resumable large write would be a new tool. An `Append` exists in `Access` and Cursor has no equivalent.

**Other harnesses.** None pages and none has an append mode in its write tool. Cline appends only through `insert_line = line_count + 1` in `editor` and refuses a `new_text` over 6,000 characters (`helpers.ts:21-33`). Rig and its `auto_continue` plugin tell the model to write the first part with `write` and add the rest with `edit` (`feedback.rs:57-62`), and Zed has no such text. `(source)`

### Q2. Units

| Quantity | Unit | Where the comparison is |
|---|---|---|
| `contents` size | unbounded | nothing compares it |
| Old-content snapshot threshold | bytes on disk | `stats.size <= MAX_OFS_FILE_SIZE`, `MAX_OFS_FILE_SIZE = 52428800` |
| Fingerprint threshold | bytes on disk | `stats.size > MAX_TEXT_SIZE`, `MAX_TEXT_SIZE = 8388608` |
| Full-content elision in the result | JS string length, UTF-16 code units | `(before?.length ?? 0) > EDIT_FULL_FILE_CONTENT_MAX_CHARS`, `512*1024` |
| Diff worker threshold | JS string length | `original.length + new.length > DIFF_SIZE_THRESHOLD`, `256*1024` |
| Encoding sample | bytes | `FORMAT_HEADER_BYTES = 8192` |
| Encoding cache | paths | `MAX_FILE_ENCODINGS = 100` |
| Hook snapshot threshold | bytes on disk | `size > 262144` |
| The read cap the tool inherits | JS string characters | `content.length > READ_CHAR_HARD_LIMIT`, `100000` |
| The read truncation the tool reacts to | JS string characters, 8 MiB | `truncated` on the read result, set by the executor's own `MAX_TEXT_SIZE` budget |

The units do not match. The diff threshold and the elision threshold count UTF-16 code units while the snapshot thresholds count bytes, so a file of 500 KB of CJK text is *under* the 524,288 "character" elision limit only if it has fewer than 524,288 code units - which for 3-byte UTF-8 characters is about 1.5 MB on disk. Two thresholds that look like the same kind of limit behave differently on the same file.

`(daemon)` `MAX_OFS_FILE_SIZE` at 14536957, `MAX_TEXT_SIZE` at 18017, `EDIT_FULL_FILE_CONTENT_MAX_CHARS` at 12539479, `DIFF_SIZE_THRESHOLD` at 12534208. `(bundle)` `wbe=524288` at 4157903.

For a Rust port: `String::len()` is bytes and `chars().count()` is code points, and Cursor's `.length` is neither. If you want Cursor's behaviour you need UTF-16 code unit counts, which means `encode_utf16().count()`. If you want something sane, pick bytes and document the divergence. My advice is bytes: every one of these limits is a proxy for memory or tokens, and bytes is the better measure for both.

**Other harnesses.** Units diverge in the same places. Pi's old reply printed UTF-16 code units as bytes and was removed in `e583b290a`; rig's old line count is `\n` bytes plus one for an unended last line (probe: `FF 0A FE 0A` counts 2); zeroclaw's reply counts decoded bytes (`file_write.rs:229`); Goose counts `str::lines()` (`dev/edit.rs:63`); Cline's guard counts UTF-16 units (`helpers.ts:29`); Zed's parser tracks bytes (`streaming_parser.rs:58`). `(source)` `(probe)`

### Q3. Text encoding

Q3 splits into three claims with three different evidence levels, and conflating them is how a reimplementation ends up confidently wrong.

**What is directly observable.** The tool's call into the encoder, the strings it passes, and the strings it produces. `(bundle)` `dr` at 8603900 calls into `bt.encode(d, p, {defaultEncoding:"utf8"})` and prefixes a BOM buffer when it decided to. That much is byte-exact.

**What is observable in the daemon but not in the minified bundle.** The encoder itself, under original identifiers and with its source path preserved: `../utils/dist/encoding.js`, `../utils/dist/line-endings.js`, `../utils/dist/lossy-utf8.js`. The daemon is 20,213,626 bytes against the bundle's 9,814,287, and it keeps module boundaries. So `getFormatForBuffer`, `detectUTF16Encoding`, `detectUTF32Encoding`, `isBufferText`, `decodeBufferWithEncoding`, `encodeUtf8RestoringUndecodableBytes`, `withExistingLineEndings` and `splitLines` are all readable. `(daemon)`

**What is not observable from either.** Whether the daemon and the bundle are the same build. The daemon embeds source paths and unminified code; the bundle is the shipped agent. They agree on every string I cross-checked, and disagree on nothing I could test, but they are not provably the same revision. Where a claim below rests only on the daemon, it says so.

#### Detection, in the order the code runs it

`(daemon)` `getFormatForBuffer` at 15437700. The input is the first 8192 bytes, and when the header comes back full at exactly 8192 bytes the last partial UTF-8 character is trimmed before detection, because the sample cut it in half.

| Order | Test | Result |
|---|---|---|
| 1 | header is zero length | platform default: `{encoding:"utf8", lineEnding: win32 ? CRLF : LF}` |
| 2 | `FF FE` and **not** followed by `00 00` | UTF-16LE |
| 3 | `FE FF` | UTF-16BE |
| 4 | more than 80% of 50 sampled UTF-16LE pairs have a NUL in the high byte and a printable ASCII, tab, LF, or CR in the low byte | UTF-16LE |
| 5 | the mirror condition, NUL in the low byte | UTF-16BE |
| 6 | `00 00 FE FF` | UTF-32BE |
| 7 | `FF FE 00 00` | UTF-32LE |
| 8 | more than 80% of 50 sampled UTF-32BE quads | UTF-32BE |
| 9 | the mirror condition | UTF-32LE |
| 10 | not `isBufferText` | `binary`, and `isImageFile` from a magic-number sniff |
| 11 | valid UTF-8, and (contains ESC or is not pure ASCII) | detected UTF-8 |
| 12 | not valid UTF-8 but "mostly valid UTF-8 with stray bytes" | detected UTF-8 |
| 13 | jschardet with `confidence > 0.7`, else `utf-8` | that encoding |
| 14 | a non-latin single-byte charset whose header still looks like latin1 | downgraded to `windows-1252` |

The order inside rows 2 through 9 is what makes `.txt` files full of NUL bytes work: `61 00 62 00 63 00` is claimed by the UTF-16LE heuristic at row 4, so it never reaches the NUL scan at row 10. Reverse those two and every UTF-16 file becomes "binary".

`detectUTF16Encoding` requires at least 4 bytes and `detectUTF32Encoding` requires at least 8, so a 3-byte file skips rows 2 through 9 entirely. `FF FE 00 00` is excluded from row 2 by the explicit `!(buffer[2]===0&&buffer[3]===0)` test, which is the only reason row 7 is reachable.

Row 11's `(!header.includes(ESCAPE_BYTE) || !isAscii(header))` is a strange condition. A pure-ASCII file with no ESC byte fails it, falls through to row 13, and gets `utf-8` from jschardet anyway - so the effect is a wasted detector call and the same answer. The condition exists to make pure ASCII reach the charset detector, presumably for a case that no longer exists.

Row 10's `isBufferText` is the binary sniff, and it is what `Write` rejects on:

```js
function isBufferText(buffer){
  const len=Math.min(FORMAT_HEADER_BYTES,buffer.length);
  if(len===0){return true}
  for(let i=0;i<len;i++){if(buffer[i]===0){return false}}
  let nonPrintableCount=0;
  for(let i=0;i<len;i++){const byte=buffer[i];
    if(byte<32&&byte!==9&&byte!==10&&byte!==13){nonPrintableCount++}}
  const nonPrintablePercentage=nonPrintableCount/len*100;
  return nonPrintablePercentage<5
}
```

Any NUL byte in the first 8192 fails. More than 5% control bytes other than tab, LF, CR fails. The check runs *after* the UTF-16 and UTF-32 tests, so a UTF-16LE file full of NULs is classified UTF-16LE before the NUL scan can call it binary.

#### BOM

`(daemon)` `getBufferForWrite` at 15433367, and the BOM rules are in step 40 of the algorithm above. Restated as a table:

| Existing file | Encoding chosen | BOM written |
|---|---|---|
| BOM-less, ASCII | utf8 | none |
| BOM-less, UTF-8 with non-ASCII | utf8 | none |
| `EF BB BF` + UTF-8 | utf8 | `EF BB BF` |
| `FF FE` + UTF-16LE | utf16le | `FF FE` |
| `FF FE 00 00` + UTF-32LE | utf32le | `FF FE 00 00` |
| new file, no hint | utf8 | none |
| any file, caller passes an `encodingHint` of `utf8bom` | utf8bom | `EF BB BF` |
| any file, caller passes an `encodingHint` | that hint | that encoding's BOM if the file had one |

The `encodingHint` column is not reachable from the model. `WriteArgs` carries an `encodingHint` field, but the tool constructs it as `new WriteArgs({path, fileText, toolCallId})` with no hint, so on the `Write` path the hint is always undefined. It exists for other callers.

`(bundle)` `Rbe` at 4163000 constructs `new TW.hY({path:r, fileText:n, toolCallId:o.toolCallId})`.

Consequence: no path through the `Write` tool can add a BOM to a file that did not have one, or remove one from a file that did. The BOM is preserved, which is a stronger property than "the model's BOM is respected".

#### Unicode normalization

None. There is no NFC, NFD, NFKC or NFKD call anywhere in the write path. `normalizeEncodingName` normalizes an *encoding label* to lowercase and maps `latin-1` to `latin1` and `utf-8-bom` to `utf8bom`; it does not touch content.

So two visually identical strings can differ in bytes, and a file whose paths contain combining characters will be written under the byte sequence the model sent. On macOS this interacts badly with the filesystem's own NFD normalization, which is outside the write path.

`pi_write` is the exception in the family, and it is the port's behavior, not pi's: `resolvePiPath` in the port replaces only U+00A0 with a plain space (after a `trim()`), where pi's own `normalizePath` folds `[\u00A0\u2000-\u200A\u202F\u205F\u3000]`, five classes covering 15 code points (`src/utils/paths.ts:7`). The port's `macReadVariants` helper tries NFD forms, a curly-apostrophe substitution, and a replacement of every space with U+202F for reads (daemon 16839032), where pi rewrites only the space before `AM.` or `PM.` (`path-utils.ts:7-9`). That is the pi surface's problem. `(daemon)` `(source)`

#### Lone surrogates and invalid UTF-8

A lone surrogate cannot exist in a valid JSON string, and the argument arrives as parsed JSON, so a model cannot send one. What can arrive is U+FFFD, because a model that read a latin1 file saw replacement characters in its Read result and may echo them back.

U+FFFD is the hook the encoder uses. `(daemon)` `encodeUtf8RestoringUndecodableBytes` at 15436236 is transcribed in full in Q10, and it is the only mechanism that makes a non-UTF-8 round trip work.

#### The honest limit

Everything above the encoder is confirmed in the shipped bundle by string and call-site. Everything in the encoder is confirmed in the daemon. What is not confirmed is that the installed daemon and the installed bundle are the same revision, and there is no probe in this pass that would settle it - the probes I ran exercised the *documented* behaviour through a reimplementation of the algorithm, and none of them ran Cursor itself.

Cursor's shipped *tool* calls an encoder that behaves as described, and the encoder is the one in the daemon. If the daemon lags the bundle, the detection order might differ in a detail. One live test on your side with a UTF-16 file would settle it, and it gives you a byte-exact fixture.

**Other harnesses.** Most write UTF-8 verbatim: pi, rig, zeroclaw, Goose, Codex, and Cline (which turns invalid bytes into U+FFFD on a rewrite). OpenCode keeps only the BOM. Gemini drops it and, for some model families, unescapes the content. Claude Code keeps the existing encoding and refuses legacy encodings and UTF-16 BE (`error-strings.json`). Aider writes the session encoding, strict, and a lone surrogate raises after the file was truncated to 0 bytes (probe). Zed is the only Rust implementation that preserves encodings: it detects from a BOM, a null-byte heuristic or `chardetng`, and encodes with `encoding_rs`, writing UTF-16 by hand because `Encoding::encode` cannot, and unmappable characters come out as `&#N;` entities (`file_content.rs:112-146`; `(web)` https://docs.rs/encoding_rs/0.8.35/encoding_rs/struct.Encoding.html). `(source)`

### Q4. Line endings

Four rules, first match wins. `(daemon)` `getBufferForWrite` at 15433367, and `withExistingLineEndings` at 15428106.

```
a  contents contains "\r\n"              -> untouched
b  existing file has mixed endings       -> per-line merge
c  detected ending is CRLF              -> every bare LF becomes CRLF
d  otherwise                            -> untouched
```

#### Classification

```js
function lineEndingFromCounts(crlfCount,lfOnlyCount){
  if(crlfCount===0&&lfOnlyCount===0){return LineEnding.LF}
  const total=crlfCount+lfOnlyCount;
  const crlfPercentage=crlfCount/total*100;
  return crlfPercentage>=5?LineEnding.CRLF:LineEnding.LF
}
```

CRLF needs at least 5% of the terminators. One CRLF in twenty LFs is classified LF, and rule c then does not fire, so the lone CRLF survives as-is. A file that is 4% CRLF is a "mixed" file for the purposes of `hasMixedLineEndings` (which only needs at least one of each) but classified LF, so rule b fires before rule c is ever consulted and the merge runs with a default ending of `\n`.

#### Mixed detection

```js
function hasMixedLineEndings(text){
  let crlf=false;let lf=false;
  for(let at=text.indexOf("\n");at!==-1;at=text.indexOf("\n",at+1)){
    if(text.charCodeAt(at-1)===CARRIAGE_RETURN){crlf=true}else{lf=true}
    if(crlf&&lf){return true}
  }
  return false
}
```

Only LF-terminated lines count. A file that is all bare CR (`\r` with no `\n`) has no `\n`, so it is not mixed, and its classification counts zero terminators, so it is LF, and it is left untouched by rule d.
#### The merge, which is the hard part

The merge is a three-phase algorithm, not a plain diff.

```
split(text) -> {lines, ends}
  every \n is a terminator; the preceding \r, if any, belongs to ends not lines
  the final segment after the last \n is a line with ends[j] == ""

pair(before, after) -> pairs[afterIndex] = beforeIndex or -1
  phase 1: common prefix, matched element by element
  phase 2: common suffix, matched from the end backwards
  phase 3: only if the unmatched interior is non-empty on both sides AND
           (rows+1)*(columns+1) <= 4,000,000:
             build an LCS length table over the interior,
             then walk it greedily forward, matching equal elements and
             otherwise stepping toward the larger of the two neighbours
  phase 4: THE BACKFILL PASS
             walk afterIndex 0..after.length inclusive;
             skip unmatched entries;
             whenever the gap in before indices equals the gap in after
             indices since the last matched pair, fill the in-between
             after entries with consecutive before indices

merge(content, existing, defaultEnding)
  previousEnd = defaultEnding
  for each line j of content:
      end = content.ends[j]
      if end == "\n":
          end = pairs[j] != -1 && existing.ends[pairs[j]] != ""
                ? existing.ends[pairs[j]]
                : previousEnd
      if end != "": previousEnd = end
      emit content.lines[j] + end
```

Phase 4 is the part nobody would write from a description, and it is what makes the parent spec's probe case come out right. Without it, a three-line replacement of a three-line mixed file has no matching lines at all, every line falls back to `previousEnd`, and you get uniform CRLF - not the per-line result the spec documents.

With it, three unrelated lines inherit the three old terminators positionally, because after the prefix and suffix passes leave nothing matched, the backfill sees a gap of 3 on both sides and fills pairs `[0,1,2]`.

I ported the four phases literally and ran vectors against the port, with `R` for CRLF and `N` for LF in the table. The results come from a faithful transcription of the daemon source and not from Cursor itself, so treat them as the algorithm's behaviour rather than as probes:

| Existing | Content | Branch | Result |
|---|---|---|---|
| `aaR bbN ccR` | `mmN nnN ooN` | b | `mmR nnN ooR` |
| `aaR bbN ccR` | `xxN yyN zzN` | b | `xxR yyN zzR` |
| `aaR bbN ccR` | `mmN nnN oo` | b | `mmR nnR oo` |
| `aaR bbN ccR ddR` | `xxN yyN zzN` | b | `xxR yyR zzR` |
| `aaR bbN ccR ddR` | `wwN xxN yyN zzN` | b | `wwR xxN yyR zzR` |
| `aaR bbN ccR ddR` | `wwN xxN yyN zz` | b | `wwR xxR yyR zz` |
| `aaR bbN ccR` | `newN aaN bbN ccN` | b | `newR aaR bbN ccR` |
| `aaR bbN ccR` | `aaN ccN` | b | `aaR ccR` |
| `aaR bbN ccR` | `mmR nnN` | a | `mmR nn` (untouched) |
| `aR bR` | `alphaN betaN` | c | `alphaR betaR` |

Row 4 is where the backfill's shape shows. Four old lines, three new lines, nothing in common: the interior gap is 4 on the before side and 3 on the after side, so the backfill condition (`i - matchedBefore === j - matchedAfter`) never holds and no pairs are made. `previousEnd` starts at CRLF because the file classifies CRLF, and all three new lines inherit CRLF. Row 5, with four lines on each side, backfills `[0,1,2,3]` and gives `wwR xxN yyR zzR` - the middle line inherits the bare LF of `bb`. Two vectors that differ only by one line produce structurally different endings, which is the kind of thing a test suite needs a vector for.

In row 3, content `mmN nnN oo` against existing `aaR bbN ccR` produces `mmR nnR oo` - uniform CRLF, no inheritance at all - while the same three lines *with* a trailing newline produce `mmR nnN ooR` and inherit per line. The presence of a trailing newline in the content changes which terminators the earlier lines get.

The mechanism is the phantom final line, which also explains the algorithm's overall shape. `splitLines` always appends one final entry after the last terminator:

```js
lines.push(text.slice(start));ends.push("");
```

A file ending in a newline therefore splits to `["aa","bb","cc",""]` with ends `["\r\n","\n","\r\n",""]`. That trailing empty line is an element of the array, and the LCS phase matches it against the content's own trailing empty line, which the suffix pass finds before the interior is even considered. With that anchor at index 3 on both sides, the backfill's condition (`i - matchedBefore === j - matchedAfter`) holds for the interior and fills `[0,1,2]`.

Without the trailing newline, the content splits to `["mm","nn","oo"]` with ends `["\n","\n",""]`. There is no phantom on the content side, the suffix pass finds nothing, the LCS finds nothing, and the backfill's only anchor is the synthetic `j === after.length` step, where the gap is 5 on the before side against 4 on the after side. No fill. Every line takes `previousEnd`, which the first line set to CRLF.

The per-line merge only reaches the backfill when the content ends with a newline, so a model that omits the final newline gets uniform endings instead of preserved ones. The behaviour is undocumented and not defensible, but it is what the algorithm does.

Row 3 argues for treating this merge as a known-quirky heuristic rather than a specification: it does not do what a reasonable person would predict, and the source has no comment explaining the intent. For a test suite, every mixed-ending vector needs a trailing-newline variant and a no-trailing-newline variant, because they take different branches inside the merge.

#### Trailing newline

There is no trailing-newline logic anywhere in the write path. `contents` ending without a newline produces a file ending without a newline; `contents` ending with one produces exactly one. There is no `insert_final_newline` and no `.editorconfig` handling.

The one place a trailing newline gets added in Cursor's source tree is the settings-file editor, which uses `{insertFinalNewline:true}`, and that is a different subsystem.

#### What happens to a `\r` on its own

Rule c's regex is `/(?<!\r)\n/g`, which only rewrites LF not preceded by CR. A bare `\r` with no following `\n` is never touched, so a file with CR-only line endings survives rule c intact - and, as noted, is not even classified as anything but LF.

**Other harnesses.** Gemini restores CRLF all or nothing (`write-file.ts:468-475`), so mixed files become uniform. Cline restores it on the matched span of an edit and picks the host's ending for a new file unless the content has `\r`. Zed normalizes on the way in and re-imposes the style of the first newline in the first 1000 bytes on the way out (`text.rs:3646-3661`). Claude Code writes LF. Codex strips a trailing `\r` and always ends lines with `\n`. Aider writes `os.linesep` by default. Pi, rig, zeroclaw, Goose and OpenCode write the bytes as sent, so a CRLF file overwritten with LF text becomes LF. Only Cursor merges per line. `(source)` `(probe)`

### Q5. Binary detection

Three mechanisms, and they disagree, so the answer depends on which file you mean.

**(a) Binary bytes in a text-extension file are rejected by the read.** `(bundle)` `isBufferText`, applied to the first 8192 bytes. Any NUL, or more than 5% of bytes below 0x20 that are not tab/LF/CR, gets `invalidFile` with the reason `File content is binary data, not text`. `Write` turns that into:

```text
Error: File content is binary data, not text: <path>
```

`(bundle)` at 9136219 for the reason, `(probe)` for the rendered form.

**(b) A binary extension changes the encoder, and the extension list that matters is not the one you would expect.**

There are two extension sets in play, and only one of them applies to `Write`.

The set you will find first is `UNSUPPORTED_BINARY_EXTENSIONS` at 13569705:

```js
/\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i
```

That one is used by the **Read tool**, thrown from inside its own `execute` at 13574854 as `Cannot read binary files of type ${ext}`. `Write` does not call the Read tool. It calls the read *executor* directly - `(bundle)` `i.execute(t,n,...)` where `i = e.get(X$._As)` at 4231080 - so this rejection never runs on the `Write` path. A model that does `Read` on `x.bin` is refused; the same model doing `Write` on `x.bin` is not.

The set that does apply is `BINARY_EXTENSIONS` in the encoder, `(daemon)` at 15450802:

```js
new Set([".png",".jpg",".jpeg",".gif",".webp",".ico",".bmp",".svg",".pdf",
         ".zip",".tar",".gz",".exe",".dll",".so",".dylib",".bin",
         ...VIDEO_EXTENSION_LIST])
```

For a file with one of these extensions, `getFormatForFile` returns `{encoding:"binary", isBinaryFile:true}` without reading the file at all, which sends the write down the base64-or-latin1 branch instead of the ordinary encoder.

So the behaviour, with the correct layer for each row:

| Target | Enforced where | Model sees |
|---|---|---|
| `.bin`, `.zip`, `.exe` containing real archive bytes | executor's `isBufferText` sniff, on the actual bytes | `Error: File content is binary data, not text: <path>` |
| `.bin`, `.zip`, `.exe` containing ASCII text | nothing rejects it | `Wrote contents to <path>`, content written latin1 |
| `.bin`, `.zip`, `.exe` containing non-ASCII text | nothing rejects it | `Wrote contents to <path>`, non-ASCII mangled to single bytes |
| `.png`, `.jpg` with a decodable image | the read layer's image path | `Error: File is not a valid image: ...`, a different error family |
| `.png`, `.jpg` containing ASCII text | nothing rejects it | `Wrote contents to <path>`, content written latin1 |
| `.pdf` | the read layer's PDF text extraction | depends on whether extraction succeeds |
| `.txt` containing NUL bytes | the executor's UTF-16 heuristic claims it first | `Wrote contents to <path>`, file becomes UTF-16LE |
| new file with a binary extension | nothing rejects it | `Wrote contents to <path>`, content written latin1 |

The `.txt`-with-NULs row looks like it contradicts (a), but it does not: the UTF-16 and UTF-32 heuristics run *before* the binary sniff, so `61 00 62 00 63 00` is classified UTF-16LE and never reaches the NUL test. That ordering is what makes (a) survivable at all.

**(c) Image extensions get a decoder error.** `(probe)` from the parent spec, a valid-signature PNG that would not decode:

```text
Error: File is not a valid image: its data could not be decoded (There are some read requests waitng on finished stream). A truncated or hand-copied file does this; write it again from the original bytes with code.: <path>
```

Cursor's own text has the typo `waitng` and doubled punctuation before the path. It is a different error family from (a), produced by a different layer.

**What the model sees, summarised:**

| Target | Model sees |
|---|---|
| text extension, binary bytes | `Error: File content is binary data, not text: <path>` |
| image extension, undecodable payload | `Error: File is not a valid image: ... : <path>` |
| image or binary extension, ASCII text | `Wrote contents to <path>`, with the content possibly latin1-encoded |
| binary extension, non-ASCII text | `Wrote contents to <path>`, with non-ASCII mangled to single bytes |
| new file with a binary extension | `Wrote contents to <path>`, with the same mangling |

For your Rust port, this is a decision rather than a behaviour to copy. Three coherent designs:

- No binary concept at all: `Write` writes bytes, the model is responsible for what it sends. Simplest, and it fits a virtual filesystem.
- Refuse by content sniff only: NUL bytes in the first 8 KB mean binary, refuse. One rule, no extension lists, no disagreement between layers.
- Refuse by content sniff and offer a separate `WriteBytes` for binary. The cleanest, and two tools where Cursor has one.

My recommendation is the second for the first version. The extension lists exist in Cursor because a single tool has to serve a model that might write a PNG, and the disagreement between the two lists is evidence that the design was never coherent.

**Other harnesses.** Few check. Claude Code refuses a file with NUL bytes in its first 8 KB, and Zed sniffs an existing file with `analyze_byte_content` (1024 bytes) and refuses `Binary files are not supported`. Pi, rig, zeroclaw, Gemini, OpenCode, Goose and Aider write whatever string they are given, NUL included. Zeroclaw is the only one with an explicit binary channel, `encoding: "base64"`. `(source)`

### Q6. Empty input and empty result

| Case | Behaviour |
|---|---|
| `contents: ""` on an existing file | **success**, the file becomes 0 bytes, the model sees `Wrote contents to <path>`. Not a no-op, not a refusal |
| `contents: ""` on a missing file | **success**, an empty file is created, parents included |
| `contents: ""` with a binary extension | **success**, 0 bytes, the base64 path is skipped because `cleanedContent.length` is 0 |
| `contents: ""` on a file whose read was `exceededLimit` | success, 0 bytes, and the diff reports the whole old file as removed |
| A missing `contents` key | `Invalid arguments:\ncontents: Required` |
| `contents: null` | `Invalid arguments:\ncontents: <zod type message>` |
| A missing `path` key | `Invalid arguments:\npath: Required`, because validation runs before the no-lock check; the lock message needs valid arguments and no streamed path |
| An empty `path` string | the schema accepts it; resolution then treats it as the workspace root, and the write fails downstream |
| A whitespace-only `path` | same as empty; there is no trim-and-refuse, unlike `Delete` |
| The file is empty and `contents` is empty | success, still 0 bytes, the diff is empty, `fileChangeType` is `modified` because the read returned `success` not `fileNotFound` |

`(bundle)` `contents` is `U$.YjP()` with no `.min()`, and the write path calls `truncate(0)` before writing zero bytes, so the last row is a distinction: an empty existing file reports `modified`, while a missing file reports `added`. The model cannot see either, because `fileChangeType` only reaches the diff, and the diff only reaches the client.

**Other harnesses.** Cursor truncates on `contents: ""`. Zed does not: `finalize_content("")` emits no event, so the old text stays and the model reads `No edits were made.` (derived from `streaming_parser.rs:260-271`, no test). Pi, rig, zeroclaw, Gemini (`write-file.ts:655-715` has no empty check), OpenCode (`write.test.ts:215-226`), Goose and Aider's empty block all truncate, and rig and Goose say so (`which had N lines`, `(0 lines)`). Copilot's `create_file` allows an empty string only for a new or empty file. `(source)`

### Q7. Cost

The tool is expensive in a way the description never hints at, because it reads the whole file before writing.

| Step | Cost |
|---|---|
| The read | the whole file into memory, decoded to a JS string |
| The fingerprint, recorded at read time | the whole file streamed through sha256, 64 KB at a time |
| The diff | O(n·m) in the worst case over the changed region, capped at a 4,000,000-cell interior for the line-ending merge and at 262,144 combined characters before it moves to a worker thread |
| The executor's snapshot | the whole file again, if under 50 MB, for hooks and tracking |
| Format detection | 8192 bytes |
| The BOM check | the first 8192 bytes again |
| The latin1 downgrade | the *whole* file again, if the detected encoding was utf8 or utf8bom |
| The undecodable-byte restore | the whole file again, if the model's content contains U+FFFD |
| The fingerprint recheck | the whole file again, streamed through sha256 |
| The write | the whole buffer |

So a single `Write` to a 1 MB UTF-8 file reads it at least four times: once for the read executor, once for the snapshot, once for the latin1 check, once for the fingerprint. Each of those is a separate `readFile`, which argues for a design that reads once and threads the bytes through.

The worst case a hostile repository can force:

- A 100 MB file: the read executor streams it (its streaming path kicks in above 8 MB), the read reports `exceededLimit` if the decoded content exceeds 100,000 characters, `Write` takes the empty-old-content branch, and the write truncates and replaces a 100 MB file with whatever the model sent. The fingerprint is above 8 MB so the stale guard is disabled. The snapshot is above 50 MB so `beforeContent` is undefined and the tracking hooks are skipped. Nothing refuses.
- A file just under 8 MB: every threshold is crossed in the expensive direction. The whole file is read and hashed four times, the diff goes to a worker, and the before/after elision kicks in so the model still sees only the sentence.
- A file with mixed line endings and a large diff: the line-ending merge runs an LCS over the unmatched interior, capped at 4,000,000 cells. Above that cap, phase 3 is skipped entirely and only the prefix, suffix, and backfill passes run, which means a large reorganisation of a mixed-ending file gets positional terminators rather than aligned ones.

`(daemon)` `getBufferForWrite` at 15433367, `fingerprintFile` at 16988590, `withExistingLineEndings` at 15428106, `MAX_OFS_FILE_SIZE` at 14536957, `MAX_TEXT_SIZE` at 18017. `(bundle)` `READ_CHAR_HARD_LIMIT` at 12792688, `DIFF_SIZE_THRESHOLD` at 12534208.

**Other harnesses.** Cost spreads widely. Pi, zeroclaw, Goose and Codex write once without reading the old file. Rig reads up to 16 MiB of it, hashes the new text twice, and runs a 100 ms diff (a probe read 16 MiB of a 17 MiB file and wrote 2 bytes in 18 ms, without the hash or diff). Gemini computes its correction twice and echoes the whole new file for a new file. Zed holds the old file as a rope, as a baseline string, and again in the UI payload (derived from `edit_session.rs:762-768` and `thread.rs:5594-5601`). `(source)` `(probe)`

### Q8. Paths

`(daemon)` `resolvePath` at 8151620, four steps in this order:

```
1  if it starts with "file://" and parses with fileURLToPath  -> convert
2  if it starts with "~" followed by / or \ or end of string  -> expand to homedir
3  if it is relative and a workspace path is known            -> resolve(workspace, p)
4  otherwise                                                  -> resolve(p)
```

Before any of that, `lke` repairs the misspelling:

```js
const cke=/(^\/(?:Users|home)\/[^/]+\/\.cursor\/)worktries(?=\/|$)/g;
function lke(e){return e.replace(cke,"$1worktrees")}
```

It only matches under a `/Users/<name>/.cursor/` or `/home/<name>/.cursor/` prefix, so on Windows it never fires.

| Path form | Behaviour | Message echoes |
|---|---|---|
| `C:\work\a.txt` | absolute, resolved as-is | `C:\work\a.txt` |
| `src\a.txt` | resolved against the workspace root | `src\a.txt` |
| `~/a.txt` | expanded to homedir | `~/a.txt` |
| `file:///C:/work/a.txt` | converted, written at `C:\work\a.txt` | `file:///C:/work/a.txt` |
| `file:///not%20a/url%zz` | rejected before resolution | the malformed URL |
| `\\server\share\a.txt` (UNC) | absolute on Windows, resolved as-is; no special handling | the UNC path |
| `C:\WORK\A.TXT` on Windows | no case folding; the OS decides | as typed |
| `..\..\etc\passwd` | resolved, and **not** refused for leaving the workspace | as typed |
| `/Users/x/.cursor/worktries/a` | repaired to `worktrees` before anything else | the repaired path |

**There is no workspace-boundary rejection.** A path outside the workspace goes through `shouldBlockWrite`, which returns `{type:"needsApproval", approvalReason:"Out of workspace", blockReason:"outOfWorkspace"}` - an approval card instead of a failure. Some paths skip even the approval: anything matching `["assets","agent-notes","agent-tools","swarm-agents"]` under a `.cursor/projects` internal path, `.md`/`.mdc`/`.cursorrules` files directly under `<homedir>/.cursor/`, and paths the settings classifier calls protected. `(daemon)` at 16681800.

The only hard refusal is the worktree guard, which is much narrower than a workspace boundary: it fires only when every workspace path contains `/.cursor/worktrees/` and the target is in a different worktree of the same repo, or in the main worktree while the workspace is a worktree.

**The lock key is the raw argument, before any resolution.** This is the one clear bug in the tool's concurrency story. `C:\a\b.txt`, `C:/a/b.txt`, and `..\a\b.txt` relative to the workspace are three lock keys for one file, so two concurrent writes to one file with different spellings do not serialize. On Windows the case-insensitive filesystem makes this worse: `C:\A\B.TXT` and `c:\a\b.txt` are the same file and two locks.

`pi_write` fixes this for a file that already exists, and only partly. `(daemon)` `queueKey` at 16837707, not 16837500:

```js
async function queueKey(path){try{return await realpath(path)}catch{return path}}
```

Canonical where possible, raw where not, and the fallback fires on every error, where pi falls back only for `ENOENT` and `ENOTDIR` and rethrows the rest (`file-mutation-queue.ts:16-26`). For a file that does not exist yet there is nothing to canonicalize, so a new file reached through a junction or a case-different spelling is not serialized, in the port or in pi's own queue, and pi's docs say so (`docs/extensions.md:2166-2168`). The shape is right for existing files and incomplete for the case `Write` meets most often. See pi for the probes. `(source)` `(probe)`

**Other harnesses.** Pi folds five space classes, strips a leading `@`, and maps `/x` to `X:\` on Windows, rewriting the target (trap 16). Rig and Goose leave `~` literal and create a directory named `~` (probes). Gemini runs `decodeURIComponent` over the whole path, so `100%25.txt` becomes `100%.txt`. Cline's lexical check refuses a name that starts with `..` and lets a junction inside the working directory escape. Zeroclaw runs `create_dir_all` before its canonical boundary check. Zed refuses a path outside every worktree before any approval. `(source)` `(probe)`

### Q9. Concurrency

Three separate mechanisms, and none of them is a lock in the sense a database engineer would recognize.

**The in-process file lock.** `(bundle)` class `lne` at 3455600. A `Set` of locked file keys plus a FIFO wait queue plus an exclusive-lock flag. `waitForLock(ctx, key)` either adds the key or queues, with abort support. The instance is created per turn and per step - `(spec)` says ten `new lne` sites, all method-local - so parallel tool calls inside one turn share a lock set and two chat tabs do not.

The key, again, is the raw argument string.

**The mutation guard.** `(daemon)` the executor optionally wraps the write in `writeFileMutationGuard.runExclusive(resolvedPath, ...)`, which is a second, canonical-path-keyed serialization inside the executor. Whether it is present depends on how the executor was constructed.

**The fingerprint.** `(daemon)` `fileChangedSinceRead` at 16988207 and `fingerprintFile` at 16988590. A sha256 plus byte length, recorded when the read executor returned content, keyed `${toolCallId}\0${resolvedPath}` in an LRU of 1,000 entries with a 60-second TTL, and consumed by `take`.

What each closes:

| Race | Closed by |
|---|---|
| two tool calls in one turn writing one file | the in-process lock, if the spellings match |
| two tool calls writing one file with different spellings | nothing, unless the mutation guard is present |
| an external process modifying the file between the read and the write | the fingerprint, if the file is under 8 MB |
| a file appearing where the read said not-found | the fingerprint's `"absent"` branch, which returns "exists now" as a change |
| a formatter or watcher rewriting the file just after the write | nothing |
| two agents, two windows, or an agent and an editor | nothing |

What it leaves open, stated as a list a test suite can hit:

- The model has a stale view and writes anyway. No mechanism compares against what the model saw, because the tool never records what the model saw.
- The file is over 8 MB. `fingerprintFile` returns `undefined`, and `fileChangedSinceRead` returns false on `undefined`, so the guard is silently off.
- The `toolCallId` differs. Two different calls that both read the file have two independent fingerprints, and each write only checks its own.
- The TTL expires. 60 seconds is a long time for a turn, but a long approval prompt can exceed it, and an expired entry means `take` returns undefined and the guard does not fire.
- The write is attempted through `pi_write`, which uses its own queue and no fingerprint.
- The write comes from a shell command, which takes the exclusive lock in Cursor's design but does nothing about the fingerprint.

For your Rust port, use one lock keyed on the canonical path, held across read-through-write, plus a content hash comparison at the point of write. Cursor has the pieces but not assembled that way.

**Other harnesses.** Pi serializes with a `realpath`-keyed queue and call order, and a new file reached through a different spelling overlaps (probe). Gemini holds an in-process per-path lock keyed on the realpath, with the real parent plus basename for a new file (`pathMutex.ts:33-103`). Rig orders conflicting calls inside one reply by footprint and has no lock. Rig's `Seen` plus `write_if` is a compare-and-swap against the previous observation, used by `edit` and not by `write`. Zeroclaw, Codex, OpenCode, Cline, Aider and Goose have no lock. `(source)`

### Q10. Fidelity

**The question:** a caller reads a file and writes it back unchanged. Do the bytes survive?

**The answer for Cursor: mostly, and the exceptions are the ones that matter for byte-exact work.**

| Property | Preserved? | Mechanism | Evidence |
|---|---|---|---|
| Encoding | yes, for the encodings it detects | the format detector plus the per-path cache | `(daemon)` `getBufferForWrite` |
| BOM presence | yes | re-reads the first 8192 bytes and re-emits the same BOM | `(daemon)` same |
| BOM absence | yes | the BOM is only written when `utf8bom` or when the file had one | `(daemon)` same |
| Uniform line endings | yes | rules a, c, d | `(daemon)` same |
| Mixed line endings | yes, positionally | rule b, the merge | `(daemon)` `withExistingLineEndings` |
| Trailing newline | yes | there is no trailing-newline logic at all | `(daemon)` same |
| Non-UTF-8 bytes decoded to U+FFFD | yes, via a splicing heuristic | `encodeUtf8RestoringUndecodableBytes` | `(daemon)` at 15436236 |
| File mode | **no, for new files**; unchanged for existing files | `open(..., 0o666)` | `(probe)` below |
| Ownership | unchanged, because the file is opened in place, not replaced | no rename | `(daemon)` |
| Lone CR line endings | untouched but also unclassified | the classifier only counts `\n` | `(daemon)` `hasMixedLineEndings` |
| mtime | changed, obviously | the write | - |
| Extended attributes, ACLs | unchanged, because the inode is unchanged | no rename | `(daemon)` |

#### The mode claim, corrected

The parent spec says: "Fixed `0o666` before umask. **No executable-bit preservation.** A write to a script that was `0755` will lose its executable bit."

That is wrong for existing files, and right for new ones, and the difference is a POSIX rule rather than a Cursor choice. `open(path, O_CREAT|..., mode)` applies `mode` **only when the file is created**. When the file already exists, the mode argument is ignored and the existing mode is untouched.

I measured it, with the exact flag set and the exact mode Cursor uses, in Git Bash on this machine under umask 0022:

```
umask:        0022
before script 755: 755
before notes  600: 644
after script  755: 755
after notes   600: 755
new file (0666&~umask): 644
```

Two rows in that output are artifacts of the probe's own `chmod` happening through Git Bash's emulation: `notes.txt` was `chmod 600` and reported as `644` both before and after, and the second `after` column reports `755` because the earlier probe run in the same directory tree. The rows that matter are the first and the last, and they are unambiguous:

- An existing `755` file survives a `0o666` open as `755`, because the open does not touch the mode of an existing file.
- A brand new file gets `644`, which is `0o666 & ~0o022`.

The `chmod 600` row is unreliable on this host and I am not claiming it. The POSIX rule it would demonstrate - an existing `600` file staying `600` - is the same rule the `755` row already demonstrates, so nothing is lost.

- An existing file's mode is preserved by accident, because the open ignores the mode for an existing file.
- A new file gets `0o666 & ~umask`, which on a typical system is `0644`. There is no executable bit, and no way to ask for one.
- Nothing in the write path ever calls `chmod` or `fchmod`. `(bundle)` has `fchmodSync` calls at 5128762 and elsewhere, all in the conflict-journal writer, none in the write executor.

For your Rust port: if `Access::write` creates with `0644` and leaves existing modes alone, you have matched Cursor. If you want to preserve an executable bit on a *new* file - which is what matters for a script the agent writes from scratch - you need a mode parameter that Cursor does not have. That is a deliberate addition.

#### The undecodable-byte restore, transcribed

`(daemon)` at 15436236, this is the mechanism that makes a latin1 read-then-write round trip:

```js
function encodeUtf8RestoringUndecodableBytes(text,original){
  if(!text.includes(REPLACEMENT_CHARACTER)||isUtf8(original)){return void 0}
  const sequences=undecodableSequences(original);
  const parts=text.split(REPLACEMENT_CHARACTER);
  if(sequences===void 0||parts.length-1!==sequences.length){return void 0}
  const pieces=[];
  for(let i=0;i<parts.length;i++){
    pieces.push(Buffer.from(parts[i],"utf8"));
    if(i<sequences.length){
      pieces.push(sequences[i]??Buffer.from(REPLACEMENT_CHARACTER,"utf8"))
    }
  }
  return Buffer.concat(pieces)
}
```

It walks the original bytes with a lossy UTF-8 decoder, records the byte sequence behind each U+FFFD, then splits the new text on U+FFFD and re-inserts the original byte sequences positionally. It requires:

- the new text to contain at least one U+FFFD,
- the original not to be valid UTF-8,
- the count of U+FFFD in the new text to equal the count of undecodable sequences in the original,
- the byte walk to consume the whole original.

If any of those fails it returns undefined and the caller falls back to plain UTF-8 encoding, which means the invalid bytes become `EF BF BD` and the file is permanently changed.

So the round trip works if the model echoes the replacement characters back unchanged and in the same positions. It breaks if the model "fixes" one, if it reorders them, or if it removes or merges a U+FFFD. Adding a new non-ASCII character does not break it: that character is valid UTF-8 and does not affect the U+FFFD count, and because the split is on U+FFFD in the text and the sequences are consumed positionally, the spliced bytes still land in the right place relative to the *text*.

#### What this means for your Rust port

The model-facing surface does not tell you how to round-trip. There is no encoding parameter, no BOM parameter, no line-ending parameter, and no report in the result. Cursor's round trip happens entirely below the capability boundary, in a layer you do not have, using a heuristic you can only see in the daemon.

So you have to decide, and the options have different costs:

**Option 1: byte-preserving, but only for the encodings you detect explicitly.** Read bytes, decode with a decoder you trust, remember the encoding and BOM in a per-call struct, encode with the same, merge line endings the same way. Round trip holds for UTF-8, UTF-8-BOM, UTF-16LE, UTF-16BE. It fails for latin1 unless you add the U+FFFD splice, because a latin1 byte like `E9` does not decode to a character under any Rust UTF-8 decoder.

**Option 2: pass the bytes through when the text did not change.** If `contents` is exactly the string the read returned, write the original bytes back verbatim. This is a one-line check and it makes the round trip total, for every encoding, including ones you cannot decode. It costs nothing and it is strictly better than Cursor, which does not do it. The only case it misses is a model that reproduces the text with a trivial change, which is not a round trip.

**Option 3: refuse to write a file you could not decode as UTF-8.** The read returns `NotUtf8`, the tool reports it, the model is told to use a different mechanism. Simple and explicit, and it loses the ability to edit latin1 files at all.

**Option 4: write bytes with no interpretation.** `contents` is a string, so it has to be encoded somehow, and UTF-8 is the only sane choice. This is where Cursor would be if it had no encoding layer, and it means every non-UTF-8 file is converted on first write.

**Recommendation: Option 1 plus Option 2, in that order.** Confidence high. Reason: Option 2 makes the round trip total for the exact-echo case at the cost of one comparison, and Option 1 makes it work for the encodings that matter. Option 3 is the fallback for anything neither covers, and it should be a distinct error the model can read rather than a silent re-encode.

Option 2 is not available to Cursor, because the write path never sees the original bytes and the new text together at a point where it can compare them cheaply. It has both, and it already does something like this through the U+FFFD splice, but only for the invalid-byte case. You can do it directly because `Access::read` gives you `Vec<u8>` and `Access::write` takes `&[u8]`.

**Other harnesses.** Rig and Gemini replace through a temp file, so they copy permissions and give up the inode: the owner changes and a hard link keeps the old text (probe). Zeroclaw, Goose, Codex, pi and Zed truncate in place, which should keep inode, mode and owner (derived from `File::create`, `fs::write` and `writeFile` semantics and not probed on POSIX). Rig's `read` strips `\r` and ends the file with `\n`, so a read-then-write round trip converts CRLF there, and Gemini's all-or-nothing rule would flatten a mixed file. Nobody besides Cursor attempts a byte-exact round trip. `(source)` `(probe)`

### Q11. Determinism

The tool produces no list, so there is no sort order to define. What it does produce, and whether two runs agree:

| Output | Deterministic? |
|---|---|
| The model-visible message | yes, given the argument |
| The bytes written | yes, given the content, the existing file, and the platform default |
| The diff string | yes, given the same content and the same diff library version |
| `linesAdded` / `linesRemoved` | yes, from the diff |
| The traceback of which rules fired | yes |
| The error message for a given failure | yes |
| Which of two concurrent writers wins | **no** - it depends on which lock was acquired first, and Cursor's lock key is spelling-dependent |
| Whether the stale check fires | **no** - it depends on timing, and on whether the file was under 8 MB |

The one cross-run nondeterminism is the platform default. A new file's line ending is CRLF on Windows and LF elsewhere, and the encoding cache is per-process, so a file written once on Windows and once on Linux from the same content differs.

`(daemon)` `getDefaultTextFormatForOS` at 15443978:

```js
return{encoding:UTF8_ENCODING,
       lineEnding:process.platform==="win32"?LineEnding.CRLF:LineEnding.LF,
       isBinaryFile:false,isImageFile:false,isVideoFile:false}
```

For a Rust port with a virtual filesystem, this is a decision: a VFS has no platform, so the default should be explicit rather than derived from `cfg!(windows)`. My advice is LF, always, and document it, because a virtual filesystem that produces different bytes on different hosts is a reproducibility problem for every test that touches it.

**Other harnesses.** Cursor is not alone in producing different bytes on different hosts. Cline writes CRLF for a new file on Windows, Aider uses `os.linesep`, Gemini uses `os.EOL` for a new file, and Zed's default line ending is CRLF on every non-Unix target. Rig, Goose and pi write the same bytes everywhere. `(source)`

### Q12. Truncation and omission

The model sees one sentence. So the question is not "what does truncation look like" but "where does the tool silently drop information".

| Information | Where it goes | Threshold |
|---|---|---|
| The diff | structured record, rendered by the client | dropped entirely when `omitEditDiffString` is set |
| `linesAdded` / `linesRemoved` | structured record | never dropped |
| `beforeFullFileContent` | structured record | dropped above 512,288 characters, or when `omitBeforeAfterFileStates` is set |
| `afterFullFileContent` | structured record | same |
| Byte count | kept inside the executor's `WriteSuccess.fileSize`, never copied into `EditSuccess` | always lost |
| Line count | `linesCreated` inside the executor's `WriteSuccess`, never copied out | always lost |
| The resolved path | `EditSuccess.path` | never dropped, but never spoken |
| The raw argument path | `EditSuccess.message` | what the model sees |
| Whether the file was re-encoded | nowhere | always lost |
| Whether the line endings were rewritten | nowhere | always lost |
| Whether the stale check was disabled by size | nowhere | always lost |

The answer to "does the tool lie about completeness" is that it does not claim completeness, it claims success. The one case where that claim is arguably a lie is the silent fingerprint disable above 8 MB, where the tool reports `Wrote contents to ...` for a write that had no freshness protection at all.

One more omission: `linesCreated` is computed as `countLines(text)`, which is newlines plus one, so a three-line file with a trailing newline reports 4. It is zero when the write came from `fileBytes`. None of this reaches the model, so it is only a trap for anyone reading the structured record and assuming it is a line count in the usual sense.

`(daemon)` `WriteSuccess` is constructed at 17009089 with `{path, linesCreated, fileSize, fileContentAfterWrite}`; `(bundle)` `vbe` at 4157903 with `wbe=524288` builds `EditSuccess` from the edit struct and never reads either count.

**Other harnesses.** Silent omissions recur. Zed's save errors vanish (trap 14), OpenCode's formatter rewrites the file after the write and the reply does not say so (`write.ts:65-67`), pi echoes a typed path for a rewritten target, and rig's `Replaced <path>.` cannot tell an unreadable old file from one over 16 MiB. Rig is the only harness whose reply states what it replaced, and Gemini's is the only one that shows the model what changed. `(source)`

## Failure matrix

Every failure the tool can produce, with the layer that produces it. Where the bundled text is a throw and the rendered text differs, both are given. The `Error: ` and `Rejected: ` prefixes are added by the renderer.

| Condition | Result case | Exact model-visible text | Layer |
|---|---|---|---|
| Arguments validate but no path value completed during streaming (practically unreachable) | `error` | `Failed to acquire file lock - path may not have been parsed` | stream wrapper |
| Argument JSON does not match the schema | `error` | `Invalid arguments:\n<key>: <message>` | stream wrapper |
| `contents` key absent | `error` | `Invalid arguments:\ncontents: Required` | stream wrapper |
| `path` key absent but a `contents` value present | `error` | `Invalid arguments:\npath: Required` - validation precedes the no-lock check | stream wrapper |
| An extra key in the arguments | `error` | `Invalid arguments:\n<key>: <zod message>` | stream wrapper |
| The call is aborted while streaming | `error` | `Aborted` | stream wrapper |
| The stream is aborted before the lock is taken | `error` | `Aborted`, after an `info` or `warn` log | stream wrapper |
| Internal read returns `fileNotFound` | `success` | `Wrote contents to <path>` | tool - not a failure |
| Internal read returns non-text output | `error` | `Error: File is not a text file (binary data)` | tool |
| Internal read returns `permissionDenied` | `permissionDenied` | `Read permission denied: <path>` | tool |
| Internal read returns `error` | `error` | `Error: <service error text>` | tool |
| Internal read returns `rejected` | `rejected` | `Rejected: <reason>` | tool, rendered with the same prefix as every other rejection |
| Internal read returns `invalidFile` with a reason | `error` | `Error: <reason>: <path>` | tool |
| Internal read returns `invalidFile` with no reason | `error` | `Error: Path is not a valid file to read: <path>` | tool |
| Internal read returns no case | `error` | `Error: Unknown error` | tool |
| Internal read returns an unrecognized case | `error` | `Error: Unhandled readResult.result: <result>` | tool |
| Target is a directory | `error` | `Error: Path is a directory, not a file: <path>` | read layer, via `invalidFile` |
| Target is a device, FIFO, or socket | `error` | `Error: Path is neither a file nor a directory: <path>` | read layer, via `invalidFile` |
| Target sniffs as binary | `error` | `Error: File content is binary data, not text: <path>` | read layer, via `invalidFile` |
| Target is an undecodable image | `error` | `Error: File is not a valid image: its data could not be decoded (<reason>). A truncated or hand-copied file does this; write it again from the original bytes with code.: <path>` | read layer |
| Plan mode, target not markdown and not a canvas path | `error` | `Plan mode can only edit markdown files and canvas files. This edit was blocked because the target is not allowed in plan-mode. If the user wants implementation changes outside markdown or canvas files, use the SwitchMode tool with target_mode_id=agent first.` | write wrapper |
| The same, thrown text | n/a | `Cannot edit non markdown files in plan mode` | write wrapper - the terse string is what `Rbe` throws, and neither the `rejected` nor the `error` case ever shows it to the model |
| Path is a malformed `file://` URL | `error` | `Error: <path> is not a valid file URL. Use an absolute path instead.` | write executor |
| Target is in another worktree | `error` | `Error: You cannot search other worktrees for this repository, stay within your workspace paths.` | write executor |
| Parent directory disappeared since the read | `error` | `Error: <path> changed on disk after this tool call read it. Nothing was written. Read the file again and retry.` | write executor |
| File changed between the read and the truncate | `error` | the same string | write executor |
| File appeared where the read said not-found | `error` | the same string | write executor |
| File changed while an approval card was open | `error` | the same string | write executor |
| Write permission denied | `permissionDenied` | `Write permission denied: <path>: <error>` | write wrapper |
| Write permission denied with no error detail | `permissionDenied` | `Write permission denied: <path>` | write wrapper |
| Read-only mode | `permissionDenied` | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` | renderer |
| Read-only subagent | `permissionDenied` | `This operation is not allowed in readonly mode. The subagent was launched with readonly: true, which restricts write operations.` | tool stub |
| Symlinked leaf | `permissionDenied` | `Write permission denied: <path>: Refusing to write through a symlink at <path>` | write executor, wrapped by the wrapper |
| Parent resolves through a symlink | `permissionDenied` | `Write permission denied: <path>: Refusing write: <path> now resolves through a symlinked parent to <real>` | write executor, wrapped |
| Target is not a regular file | `permissionDenied` | `Write permission denied: <path>: Refusing to write to non-regular file at <path>` | write executor, wrapped |
| `realpath` failed before the open | `permissionDenied` | `Write permission denied: <path>: Cannot verify canonical path before writing: <path>` | write executor, wrapped |
| `mkdir` failed with EACCES or EPERM | `permissionDenied` | `Write permission denied: <path>: Permission denied` | write executor |
| No space left | `error` | `Error: No space left on device` | write wrapper |
| Storage quota exceeded | `error` | `Error: Agent-store quota exceeded: this write was rejected because the storage quota is full. Nothing was written; your local copy is intact. Delete or truncate files to free space (including resolved *.conflict* files), then retry. Do not retry the same write without freeing space first.` | write executor |
| User denies the approval | `rejected` | `Rejected: <reason>` | renderer |
| User denies with an empty reason | `rejected` | `Rejected: ` | renderer |
| Rejection whose reason names a missing tool-call context | `error` | `Error: <reason>` | write wrapper |
| Any other executor error | `error` | `Error: <raw message>` | write wrapper |
| Unresolved notebook view type | `error` | `Error: This notebook could not be opened for editing because the editor environment has no notebook provider for it (for example, Jupyter/notebook support is unavailable). This is an environment limitation, not a problem with the edit itself - retrying will not help.` | write wrapper |
| Result case unrecognized | `error` | `Error: Unhandled result case: <case>` | renderer |

Failures that look like successes:

| Case | What actually happened |
|---|---|
| `contents: ""` | The file is now 0 bytes. `Wrote contents to <path>`, `isError: false` |
| Write to a file over 8 MB | The stale-write guard was silently disabled. Success reported normally |
| Write to a file the read executor truncated | The old content was a prefix. The diff is wrong and the write discards whatever the read did not return. Success reported normally. `StrReplace` refuses this case; `Write` does not |
| Write to a file that was never read by the model | No gate exists. Success reported normally |
| Write where the content was re-encoded | The bytes on disk are not the bytes the model sent. Success reported normally |
| Write where the line endings were rewritten | Same. Success reported normally |
| Write where the buffer was partly written because a zero-byte write broke the loop | Success reported normally. Reachability unknown |

Three of those strings appear in the parent spec with different text:

| Spec text | Shipped text |
|---|---|
| `Failed to acquire file lock - path may not have been parsed` | `Failed to acquire file lock - path may not have been parsed` |
| `Cannot edit non markdown files in plan mode` only | correct as the throw, but the model sees the long SwitchMode paragraph |
| `Write permission denied: {path}: Permission denied` as the general form | correct only when the detail is absent or is literally `Permission denied`; a symlink refusal appends the symlink sentence instead |

## Traps

Seventeen ways to write something that looks right and is not. Each has the exact test that catches it. Traps 1 to 12 come from Cursor's own build; 13 to 17 come from what the other harnesses do wrong.

### 1. Writing UTF-8 and calling it done

**What goes wrong.** The tool takes a `contents` string and writes UTF-8 bytes. A UTF-16LE file becomes UTF-8 on first touch. A latin1 file with `é` becomes `C3 A9` where it used to be `E9`. The model sees success, the file is changed in a way nobody asked for, and the change is invisible until something else reads the file.

**Why it is easy to get wrong.** The parameter is a `string`. Every language's default for "string to bytes" is UTF-8. Nothing in the description, the schema, or the result mentions encoding. The only evidence that encoding matters is in a layer you cannot see from the tool's signature.

**The test.** Create a UTF-16LE file with a BOM, two lines, CRLF endings, and one non-ASCII character. Read it through your tool, then write the exact string you got back. Compare bytes. They must be identical: `FF FE` prefix, same byte pairs, same `0D 00 0A 00` terminators.

### 2. Treating an empty `contents` as a no-op

**What goes wrong.** An `if contents.is_empty() { return }` guard, added because writing nothing seems pointless, turns a destructive truncate into a silent nothing. The model asked the file be emptied, the file is not emptied, and the tool says it worked.

**Why it is easy to get wrong.** Every other tool in the surface treats an empty input as an error. `StrReplace` throws on an empty `old_string`, `Grep` rejects an empty pattern, `Glob` rejects a pattern that normalises to `**/`. `Write` is the one tool where empty is legal and destructive. That holds inside Cursor's surface and not across the harnesses, where it is the norm for a write tool: Gemini CLI's `write_file` also accepts `content: ""` (its `replace` treats an empty `old_string` as create-file when the path is absent and errors when the file exists, `edit.ts:700-757`), and so do pi, rig, zeroclaw, OpenCode and Goose. Zed is the one that makes it a no-op. `(source)`

**The test.** Write 120 bytes to a file, then `Write` with `contents: ""`. `stat` must report 0. The message must be exactly `Wrote contents to <path>`, `isError: false`.

### 3. Adding a read-before-write gate because the description asks for one

**What goes wrong.** The description says `you MUST use the read_file tool first`. You implement the gate. Now a model that has never read a file gets a refusal where Cursor gives a success, and the model's recovery is a Read then a Write, which doubles the cost of a path Cursor handles in one call.

**Why it is easy to get wrong.** The prompt text is the most visible artefact and it reads like a specification. It is not one. Cursor's own two branches disagree about whether to mention reading at all, and the default branch does not mention it.

**The test.** Seed a file from outside the tool, never read it, then `Write` over it. Cursor returns `Wrote contents to <path>` with `isError: false`. If your build refuses, you added something.

### 4. Keying the lock on the resolved path when you meant to match Cursor, or on the raw path when you meant to be correct

**What goes wrong.** Two ways, one bug. Cursor keys on the raw argument string, so `C:\a\b.txt` and `..\a\b.txt` do not serialize and two concurrent writes to one file race. If you copy that, you inherit the bug. If you fix it by keying on the resolved path, you diverge from Cursor - which is the right divergence, but it has to be deliberate.

**Why it is easy to get wrong.** The lock is taken in a place where only the raw argument is available, because the resolution happens later, inside the executor. Moving the lock after resolution means moving it out of the streaming wrapper, which changes when it is acquired relative to the stream.

**The test.** Start two concurrent `Write` calls to the same file, one with an absolute path and one relative. With Cursor's keying, both proceed and the file ends up with one of the two contents, nondeterministically. With canonical keying, the second waits and the file ends up with the second's content. Either is defensible; only one is Cursor.

### 5. Assuming the mode argument preserves or changes the executable bit

**What goes wrong.** Two symmetric errors. If you believe `0o666` "strips the executable bit", you add a mode-preservation step that Cursor does not have and that changes behaviour on new files. If you believe the mode is irrelevant, you never notice that a new script is written `0644` and cannot be executed.

**Why it is easy to get wrong.** `open(..., 0o666)` looks like it sets the mode. It sets the mode *on creation only*. On an existing file it is ignored. The parent spec states the wrong half of this as a bug, which makes it worse: a reader who checks will find the spec wrong and may conclude the whole mode section is wrong.

**The test.** `chmod 755` a file, `Write` it, `stat` the mode. It must still be `755`. Then `Write` a new file that does not exist and `stat` it: `0644` under umask 022, never `0755`, with no way to ask for `0755`.

### 6. Porting the line-ending merge as a plain diff

**What goes wrong.** The merge has a backfill phase that fills unmatched gaps positionally when the before and after gaps are equal. Without it, a whole-file replacement of a mixed-ending file falls back to `previousEnd` for every line and produces uniform endings, which is the case the parent spec documents as producing per-line preservation.

**Why it is easy to get wrong.** "Myers-style line alignment" describes phases 1 through 3. Phase 4 is six lines of code that only fires when the gaps match, and if you diff the source against a standard LCS implementation you will not notice it because it reads like cleanup.

**The test.** Existing `aa\r\nbb\ncc\r\n`, content `mm\nnn\noo\n`. Result must be `mm\r\nnn\noo\r\n` - the middle line keeping the bare LF. Then the same content without the trailing newline: `mm\nnn\noo`, which produces `mm\r\nnn\r\noo`. Two vectors, different results, both required.

### 7. Forgetting that the content's own CRLF wins outright

**What goes wrong.** You implement CRLF rewriting and apply it unconditionally, so a model that sends `\r\n` deliberately gets its endings rewritten and, in the CRLF-into-CRLF case, doubled.

**Why it is easy to get wrong.** Rule a is checked first and it is a plain `includes("\r\n")`, which means *one* CRLF anywhere in the content disables all rewriting for the whole file. A model that sends mostly LF with one CRLF gets an entirely untouched file, including the LFs.

**The test.** Existing `a\r\nb\r\n`, content `x\r\ny\n`. Result must be `x\r\ny\n` - the second line keeps its bare LF, because rule a fired and nothing was rewritten. This is the vector that catches a `replace("\n", "\r\n")` implementation immediately, since it produces `x\r\n\r\ny\r\n`.

### 8. Believing the success message names the file that was written

**What goes wrong.** The message echoes the raw argument. The file is written at the resolved path. A test that parses the message to find the file finds a path that may not exist, and a client that displays the message shows the model's spelling rather than the truth.

**Why it is easy to get wrong.** Most tools normalise before reporting. `StrReplace` reports the path it was given too, so the pattern is consistent - but `Read` reports the resolved path, which makes the family inconsistent and makes a reasonable reader assume they all resolve.

**The test.** `Write` to `file:///C:/work/a.txt` and to `~/a.txt`. The message must contain the literal `file:///C:/work/a.txt` and the literal `~/a.txt`. The structured `path` must contain the absolute resolved path in both cases.

### 9. Trusting the stale-write guard above 8 MB

**What goes wrong.** `fingerprintFile` returns `undefined` for a file over `MAX_TEXT_SIZE`, and `fileChangedSinceRead` treats `undefined` as "unchanged". So the guard silently stops working at the size where a concurrent write is most expensive to lose.

**Why it is easy to get wrong.** The comparison is in a `try` with a `catch` that returns false, and the size check returns `undefined` rather than throwing, so nothing anywhere logs or surfaces the downgrade. A reader who checks "is there a stale check" finds one; a reader who checks "when does it run" finds the answer is "below 8 MB, silently".

**The test.** Create a 9 MB file, start a `Write`, and modify the file from outside between the read and the write. Cursor overwrites without complaint. Below 8 MB the same sequence is refused with the changed-on-disk message.

### 10. Passing the model's `contents` string through a lossy decode before writing it

**What goes wrong.** If your pipeline ever converts `contents` to bytes and back through a lossy UTF-8 decoder, U+FFFD sequences become replacement characters and the byte level changes. Cursor's own encoder does this deliberately and then repairs it, which means the repair has to be present or the file is silently mangled.

**Why it is easy to get wrong.** In Rust, `String` is guaranteed valid UTF-8, so a lossy decode has to be explicit - but the same mistake shows up as accepting a `Vec<u8>` and calling `String::from_utf8_lossy`, which is the idiomatic thing to do and is the wrong thing here.

**The test.** A latin1 file containing `E9 E8 EA` plus ASCII. Read it, write the string back unchanged. The bytes must still be `E9 E8 EA`. If they became `EF BF BD EF BF BD EF BF BD`, the replacement characters were materialised. If they became `C3 A9 C3 A8 C3 AA`, the file was re-encoded as UTF-8.

### 11. Assuming the binary-extension list you found is the one that applies

**What goes wrong.** Two extension regexes exist, one in the Read tool and one in the encoder, with different membership and different consequences. Copying the Read tool's list into a Write implementation refuses files Cursor writes, and copying the encoder's list into a read-side check accepts files Cursor refuses.

**Why it is easy to get wrong.** The Read tool's list is right next to the Read tool and looks authoritative. `Write` never calls the Read tool, so that list is dead code for this path.

**The test.** `Write` to a `.bin` file containing ASCII text. Cursor accepts it and writes the content as latin1. `Write` to a `.bin` file containing `00 01 02 03 00 00 FF`. Cursor refuses with the binary-data message, and the refusal comes from the byte sniff rather than the extension.

### 12. Copying the plan-mode suffix list from memory

**What goes wrong.** The allowlist is six suffixes plus a regex, and `.md` alone is not it. Miss `.mkd` and a plan-mode write to a `.mkd` file is refused. Add `.txt` because it seems harmless and a plan-mode write gets through that Cursor would block.

**Why it is easy to get wrong.** The list is in a constant called `S_e` in the minified bundle and `MARKDOWN_SUFFIXES` in the daemon, and it is checked with `trim().toLowerCase().endsWith(...)` so case and surrounding whitespace do not matter. Getting the trim right and the list wrong gives a plausible-looking implementation.

**The test.** Six vectors, one per suffix, all of which must be allowed in plan mode: `a.md`, `a.markdown`, `a.mdown`, `a.mkd`, `a.mkdn`, `a.mdx`, plus one uppercase form `A.MD`. Then `.MDX.bak`, which must be refused.

### 13. Overwriting from an incomplete view or an incomplete message

**What goes wrong.** The old content the tool saw was a prefix of the file, or the `contents` it received was a prefix of what the model meant to send. The write succeeds and the rest is gone. Cursor's `Write` does not check the read's `truncated` flag, and nothing in its path looks at why the model's message ended. `(bundle)` `(source)`

**Why it is easy to get wrong.** `StrReplace` has the check, so the safety property looks like part of the shared machinery. It is not. And in pi a cut message can arrive as arguments that parse and validate, because streamed JSON is finalized by a salvage parser (`agent-loop.ts:481-487`). Whether a server upstream of Cursor does the same is not in the sources.

**The test.** Two vectors. First, make the read executor report `truncated` for a file just over 8 MiB, then `Write` over it: the tool must refuse, and `StrReplace` already does. Second, hand the harness an assistant message whose stop reason is the output limit and whose `Write` arguments validate: no tool call in that message may run, and the file must be byte-identical afterwards. Pi's text for the refusal is a usable template.

### 14. Reporting success for a write that did not reach the disk

**What goes wrong.** The save step's error is logged and dropped, and the result is built from an in-memory copy. Zed ends `save_buffer` and `format` in `.log_err()` (`edit_session.rs:204,210`) and then returns `Edited <path> successfully`, while its system prompt tells the model not to re-read because "the tool call will fail if it didn't work". A read-only file or a full disk looks like a normal write. `(source)`

**Why it is easy to get wrong.** A harness with an editor buffer has two states, buffer and disk, and the result is naturally built from the one that always works. Rig avoids it by building its reply after `replace` returns (`write.rs:111-120`).

**The test.** Make the target read-only, `Write` to it, and require an error result. Then fill the quota and require `No space left on device`. A result that names a path must be built after the write returned.

### 15. A count in the result that is not what it says

**What goes wrong.** `Successfully wrote ${content.length} bytes` prints UTF-16 code units under a `bytes` label. Cursor's `pi_write` still does. Pi removed the count in `e583b290a` after issue 8979, and rig replaced its `Wrote {N} bytes` reply with the old file's line count because the model could not otherwise know it. Goose's `(N lines)` uses `str::lines().count()`, which ignores a trailing newline, and rig's old line count is the number of `\n` bytes plus one when the last line is unended, so a file of three CR-ended lines counts as 1. `(daemon)` `(source)` `(probe)`

**Why it is easy to get wrong.** The cheapest number to compute is the one the runtime already has, and its unit is the runtime's.

**The test.** Write `caf�` and `??` to new files. A `bytes` count must read 5 and 4, `chars().count()` must not be what produced it, and a line count must say what it counts on a file with CRLF, a lone CR, and no final newline.

### 16. Rewriting the target path and echoing the typed one

**What goes wrong.** Pi strips a leading `@`, folds Unicode spaces and maps `/x` to `X:\` on Windows before writing, then replies with the typed path. `@types/node/index.d.ts` was written to `types/node/index.d.ts`, and an overwrite of a name containing U+202F created a plain-space twin and left the original alone. The opposite failure exists too: rig and Goose do not expand `~`, and `~/notes.md` creates a directory literally named `~`. Cursor sits between them and echoes the raw argument while writing the resolved path (trap 8). `(source)` `(probe)`

**Why it is easy to get wrong.** Leniency is added for reads, where a wrong guess costs nothing, and the same function is reused for writes, where it silently picks a different file.

**The test.** `Write` to `@a/b.txt`, to a name containing U+202F, and to `~/x.txt`. The reply must name the file that was actually written, and no variant may be tried for a write.

### 17. Changing the content on its way to the disk

**What goes wrong.** Gemini CLI runs `unescapeStringForGeminiBug` over `content` for some model families, so source text holding a backslash and an `n` becomes a line break. Copilot's `create_file` strips a leading `filepath:` comment from the content before inserting it (`createFileTool.tsx:111,116`, `util/common/markdown.ts:66-73`). Zed formats on save when the language asks for it and writes unmappable characters of a legacy encoding as `&#N;`. None of these tells the model. `(source)` `(web)`

**Why it is easy to get wrong.** Each rewrite fixes a model habit, so each looks like a feature. The cost is a file that differs from what the model sent with a reply that says it did not.

**The test.** Write source containing `"a\nb"` as eight characters, a leading `// filepath: x` comment, and a character outside a legacy file's encoding. Read the bytes back. Anything that changed must either be refused or reported in the reply.

## How the harnesses do this

Twelve harnesses, in one order: Claude Code, Copilot, Zed, pi, rig, zeroclaw, Codex CLI, Gemini CLI, OpenCode, Cline, Aider, Goose. Each entry names the source it was read from, the parameters, the caps with their units, an observed output, and one idea to take and one to reject. Harnesses with no write tool say what they do instead. Tier labels are `(source)` for code read at a pinned commit, `(probe)` for something run, `(web)` for documentation, and `(bundle)` and `(daemon)` for Cursor's shipped artifacts.

### Claude Code

**Source.** The `write.js` extract of `claude.exe` 2.1.311 (win32-x64), decoded as latin1, at byte offsets 256072762 to 256087301. Supporting extracts of the same binary: `read-file-state.js` (the read-state store), `freshness-checks.js` (the comparison), `limits.js` (41 constants), `error-strings.json` (126 strings), `input-schemas.md`. `(source)`

**Name.** `Write`. Constant `Yt` at offset 247297811. There is one name, no per-model branches, no aliases.

**Parameters.**

| Parameter | Type | Required | Description, verbatim |
|---|---|---|---|
| `file_path` | string | yes | `The absolute path to the file to write (must be absolute, not relative)` |
| `content` | string | yes | `The content to write to the file` |

The key is `content` where Cursor has `contents`; the two-field shape is otherwise the same.

**Caps.**

| Limit | Value | Unit |
|---|---|---|
| `maxResultSizeChars` | 100,000 | characters of the tool result |
| Read-state store maximum size | `pu` = 26,214,400 | bytes, the LRU's `maxSize` |
| Read-state per-entry content cap | `gu` = 4,096 | bytes retained in the model-visible store |
| Commit race timeout | `l1o` = 10,000 | ms |
| Memory-file name cap | 32,768 | bytes |

**Encoding and line endings.** `Write` reads the existing file with detection (`lt?.encoding ?? "utf8"`) and writes with the same encoding and a fixed `"LF"` ending, `Yat(ioPath, content, encoding, "LF")`, so a CRLF file comes back LF (`write.js:454-486`). It refuses a target that is not text, and its string table names three refusals: a file that is not valid UTF-8 and uses a legacy encoding such as Windows-1252, Shift-JIS or GBK; NUL bytes in the first 8 KB, which covers UTF-16 and UTF-32 without a byte-order mark; and UTF-16 big-endian, "which these tools cannot write" (`error-strings.json`). The BOM strip and CRLF normalization (`Ev`, `Hy` in `freshness-checks.js`) belong to the freshness comparison and to the stored read state, and play no part in the write. `(source)`

**Output shape.** Two cases, and the text differs:

```text
File created successfully at: <filePath>
```

```text
The file <filePath> has been updated successfully.
```

Both may be followed by ` The user modified your proposed content before accepting it.` and by ` (file state is current in your context - no need to Read it back)`. The structured output is much richer than Cursor's: `type` (`create` or `update`), `filePath`, `content`, `structuredPatch`, `originalFile`, `gitDiff`, `userModified`, `staged`, `memdirStamped`.

**The gate.** Claude Code has a read-before-write state machine, the sharpest contrast in the set:

```
validateInput:
  read the store entry for the normalized path
  if the entry is missing OR the entry is a partial view:
      if the entry is missing
         AND the path is not in the new-or-empty set
         AND no pre-read guard applies
         AND reads are not auto-allowed:
             if a remote baseline exists and does not match: refuse (stale, remote form)
             allow                // unread, but trusted for this reason
      refuse "File has not been read yet. Read it first before writing to it."
      // with a remote baseline present, the refusal is the remote-form stale message
  if floor(mtime) > entry.timestamp:
      if the entry is a full view AND its content hash matches the disk: allow
      refuse "File has been modified since read, either by the user or by a linter. ..."
  allow
```

`bA(entry)` is the partial-view predicate, which returns false when `offset > 1`, when `isPartialView` is set, or when a `limit` was applied that did not cover the whole file. So a Read with `limit: 50` on a 200-line file leaves a partial view, and a `Write` after it is refused even though the model did read something. That is the same class of sharp edge as Cursor's empty-content rule, in the opposite direction.

`O4t(path)` is the new-or-empty escape, and it is a path-shape test rather than a content test: it strips trailing dots and spaces, then asks whether the result names something that does not exist. So a model writing a brand-new file is not gated, which is the escape hatch D4 recommends.

The store is an LRU keyed on the normalized path, holding `content`, `contentHash` (Bun.hash base36), `contentLength`, `readBaseline` (sha256 plus byte length), `offset`, `limit`, `isPartialView`, `timestamp`, and `contentNotInModelContext`. It is dumped and reloaded across process boundaries and merged on resume. That is a state machine, where Cursor has a fingerprint.

The exact strings:

```text
File has not been read yet. Read it first before writing to it.
```

```text
File content has changed since it was last read. This commonly happens when a linter or formatter run via Bash rewrites the file. Call Read on this file to refresh, then retry the edit.
```

```text
File has been modified since read, either by the user or by a linter. Read it again before attempting to write it.
```

**The one design idea worth taking** is the read-state store as a dumpable, mergeable artefact rather than a per-process cache. Cursor's fingerprint store has a 60-second TTL and dies with the process; Claude Code's survives a resume, which is the difference between a gate that works across a long session and one that quietly stops working. Confidence high.

**The one design idea worth rejecting** is the gate's dead-end shape. A refusal with no path forward except "Read, then Write" costs a full round trip and a full file read, on a tool where Cursor's whole design is built on the premise that the tool reads the file itself anyway. If you take the gate, give it an escape: allow the write when the model passes the exact content it believes is there, which is a cheap check and turns a refusal into a correctness check. Confidence medium.

### VS Code Copilot

**Source.** `extensions/copilot/src/extension/tools/node/createFileTool.tsx` (12 KB) in `https://github.com/microsoft/vscode`, plus `applyPatchTool.tsx` (43 KB) and `common/toolNames.ts`, read from the checkout `C:\Users\Vinnie\src\vscode` at `b208bc2501` (2026-10-25). `(source)`

**Name.** `create_file`, and `copilot_createFile` in the internal-prefixed namespace. Related: `insert_edit_into_file` (`copilot_insertEdit`), `replace_string_in_file` (`copilot_replaceString`), `multi_replace_string_in_file` (`copilot_multiReplaceString`), `apply_patch`.

**Parameters.**

| Parameter | Type | Required | Description, verbatim |
|---|---|---|---|
| `filePath` | string | yes by convention | no `.describe()`; the schema is `ICreateFileParams` with `filePath: string` and `content?: string` |
| `content` | string | **optional in the type** | checked at run time by `if (!options.input.filePath \|\| options.input.content === undefined)` |

Copilot's `content` is declared optional and validated by hand, where Cursor's `contents` is schema-required.

**Content handling.** The non-notebook path strips a leading `filepath:` comment from `content` (`removeLeadingFilepathComment`, `createFileTool.tsx:116`, `util/common/markdown.ts:66-73`) and inserts the text through the response stream's `textEdit`, so the editor applies the write and the user keeps or undoes it. An existing file is refused only when it has content (`fileExists && !!doc?.getText()`, `:92`), so `content: ""` is accepted for a new or an empty file. `(source)`

**Caps.** None found in the create path. `editFileToolUtils.tsx` contains no size limit constant. The one relevant cap is in `editFileToolResult.tsx`'s diagnostics timeout, 2000 ms, which is a UI budget and limits no content.

**Output shape.**

```text
File created at <path>
```

That is the whole result on the non-notebook path. On the notebook path the result is a rendered `EditFileResult` component instead.

**Creation is split from editing.** `create_file` refuses to touch an existing file with content:

```text
File already exists. You must use an edit tool to modify it.
```

or, for a notebook:

```text
File already exists. You must use the edit_notebook tool to modify it.
```

So there is no whole-file overwrite tool at all in Copilot. Whole-file replacement goes through `apply_patch`'s `*** Add File:` directive (which itself refuses an existing path with `Add File Error: File already exists: <path>`), or through `insert_edit_into_file` with an empty `oldString`, or through the replace tools.

Copilot also has no delete tool. `apply_patch` deletes through `*** Delete File: <path>`.

**The one design idea worth taking** is `create_file`'s refusal to overwrite. It makes the destructive case impossible rather than merely unlikely, and it means the model has to choose a different tool with different semantics when it wants to replace something. That is a stronger safety property than Cursor's, achieved by removing a capability rather than gating one. Confidence high.

**The one design idea worth rejecting** is the absence of a whole-file replace. A model that needs to rewrite a 400-line file should not have to construct a patch or an empty-anchor edit. The cost is visible in `apply_patchTool.tsx`'s 43 KB and its healing logic, which exists partly to make whole-file replacement work through a patch format. Confidence medium.

### Zed

**Source.** `crates/agent/src/tools/write_file_tool.rs` (55 KB) in `https://github.com/zed-industries/zed` at `37faef81f7` (2026-10-27), plus `edit_session.rs` (44 KB), `tool_permissions.rs` (53 KB) and `edit_session/streaming_parser.rs`, read from the canonical checkout `C:\Users\Vinnie\cursor\zed`. The first pass read the 2026-08-22 checkout `d9ad6aff67`; see Zed checkout drift below. `(source)`

**Name.** `write_file`, from `const NAME: &'static str = "write_file"` at line 206 of the `impl AgentTool for WriteFileTool` block.

**Parameters.**

| Parameter | Type | Required | Description, verbatim (first sentence) |
|---|---|---|---|
| `path` | `PathBuf` | yes | `The full path of the file to create or overwrite in the project.` |
| `content` | `String` | yes | `The entire content for the file.` |

The `path` doc comment is long and prescriptive about multi-root projects: a path must start with one of the project's root directories, or be under `~/.agents/skills` for a global skill. That is a stricter path contract than Cursor's, which accepts anything and asks for approval after the fact. The description is stricter than the code. The code accepts a root-less relative path when the parent directory exists (tests at `write_file_tool.rs:504-530`), and a path outside every worktree is refused with `Can't create file: parent directory doesn't exist` before any approval is asked; `~/.agents/skills` is the one exception (`edit_session.rs:1262-1287`). `(source)`

**Caps.** None on content. The tool streams. An existing file of 6 GiB or more is refused (`File is too large to load`, `worktree.rs:1721-1727`), and so is an existing file that sniffs as binary within its first 1024 bytes (`Binary files are not supported`, `worktree.rs:7284-7287`, `file_content.rs:239-310`). `(source)`

**Output shape.** `EditSessionOutput` is an untagged enum with `Success { input_path, new_text, old_text, diff }` and `Error { error, input_path, diff }`. The `Display` impl produces:

```text
Edited <input_path> successfully
```

or, when the diff is empty:

```text
No edits were made.
```

On success that sentence is all the model sees: the diff, `new_text` and `old_text` go to the UI as structured `raw_output` and never reach the model (`edit_session.rs:96-134`, `thread.rs:5593-5601`). The diff reaches the model only inside an error that happens after the session exists. On error the text is the error and a newline.

```text
<error>
```

That is followed by the block below when the session had an input path and the diff is non-empty. Otherwise it is followed by the sentence `No edits were made.` A pre-session error therefore reads as its own line and then that sentence, as in `Can't create file: parent directory doesn't exist`. `(source)` The block, when there is a diff:

````text
Edited <input_path>:

```diff
<diff>
```
````

Note this is `write_file` reporting "Edited", which is odd; the text comes from the shared `edit_session.rs` machinery rather than the write tool, and the same enum is used by `edit_file`.

**The shared edit session, and what it provides.** Zed's `write_file` and `edit_file` both construct an `EditSession` with a different `EditSessionMode`:

```rust
EditSession::new(path, EditSessionMode::Write, Self::NAME, ...)
```

The session resolves the path inside the project tree, authorises through `authorize_file_edit`, opens a `Buffer`, prompts if the buffer is dirty, and then streams the content *into the buffer* as it arrives. It also computes whether the file changed on disk since the model last read it, and the Write pipeline discards that answer (`edit_session.rs:396-405`), so Zed has no stale-write check for `write_file` either. A bad path fails before the permission gate, so it never costs the user a prompt, and the gate runs before the buffer opens, so a user can approve a write that then fails as binary or too large (`edit_session.rs:700-760`). `(source)`

```rust
fn process_event(&mut self, event: &WriteEvent, buffer: &Entity<Buffer>, ...) {
    let WriteEvent::ContentChunk { chunk } = event;
    let edit_range = if self.content_written { buffer_len..buffer_len } else { 0..buffer_len };
    agent_edit_buffer(buffer, [(edit_range, chunk.as_str())], &context.action_log, cx);
```

Four things this provides that Cursor cannot do, two of them with caveats from the checkout:

- The diff is live. `event_stream.update_diff(diff.clone())` is called before any content arrives, and the diff entity is updated as chunks land, so the user watches the file change.
- Each streamed chunk is its own buffer transaction reported to the action log, and the review bar offers Accept and Reject. Reverting goes through the action log, not through the editor's undo stack: `buffer_created` records the old file content as the baseline and `reject_all_edits` restores it, or deletes a file the agent created (`crates/action_log/src/action_log.rs:120-147,712-795`, test `test_streaming_reject_created_file_deletes_it` at `write_file_tool.rs:1118`). Whether the editor's own undo reverses a whole write in one step is not in the sources. `(source)`
- The file is saved through the project's save path, and format-on-save runs only when `format_on_save` is not `off` for the buffer's language (`ensure_buffer_saved`, `edit_session.rs:186-205`). The shipped global default is `"off"` (`assets/settings/default.json:1653`), and several language blocks set it to `"on"` (`default.json:2425,2458,2467,2471`). Trailing-whitespace trimming and the final-newline rule live inside that format pass (`crates/project/src/lsp_store.rs:1772-1792`), so with `format_on_save` off the agent tool does neither. `test_streaming_format_on_save` and `test_streaming_remove_trailing_whitespace` set their settings explicitly (`write_file_tool.rs:609,724`), so they do not show a default. A manual save in the editor takes the format path even with `format_on_save` off (`editor/src/items.rs:2456-2459`), which is why trimming looks universal in the editor and is not in this tool. The diff in the result describes the text on disk after formatting, which can differ from what the model sent. `(source)`
- A dirty buffer is handled with a prompt. `resolve_dirty_buffer` offers `keep` and `discard` options, labelled Cancel and Overwrite, and `test_streaming_write_dirty_buffer_discard` and `test_streaming_write_dirty_buffer_keep` pin both. A third path, the user saving the buffer while the prompt is open, cancels with its own error and its own test (`write_file_tool.rs:1317`). The prompt fires once at session creation, before the content streams, and is not re-checked (`edit_session.rs:744`). Cursor reads from disk and never sees the buffer at all. `(source)`

The cost is that `write_file` cannot do anything the buffer API cannot do: it cannot write outside a project except to `~/.agents/skills`, and it goes through the project's file-system abstraction rather than raw POSIX calls. An existing file keeps its mode, owner and links because `File::create` truncates in place; a new file gets the OS default. The save is not atomic and not fsynced: truncate, buffered write, `flush`, and no `sync_all` anywhere in the `fs` crate (`crates/fs/src/fs.rs:1034-1041`). `(source)`

**What Zed does silently.** Three behaviors the model cannot see, all derived from the code with no test pinning them. `content: ""` on an existing file writes nothing: `finalize_content("")` emits no event because `0 > 0` is false, so the first-chunk replace never runs, the old text stays, and the model reads `No edits were made.` as a success (`streaming_parser.rs:260-271`, `edit_session.rs:421-440`). It is not possible to truncate a file to zero bytes with this tool, the opposite of Cursor's trap 2. Save failures vanish: `project.save_buffer` and `project.format` end in `.log_err()`, so a read-only file or a full disk returns `Edited <path> successfully` from the in-memory buffer while the buffer stays dirty in the editor (`edit_session.rs:204,210`), and the system prompt tells the model not to re-read because the call "will fail if it didn't work" (`crates/agent/src/templates/system_prompt.hbs:44`). And an interrupted stream saves what it streamed: on any failure after the session exists, `run_session` still saves the buffer, so a model that runs out of output tokens halfway through a large `content` can leave a truncated file on disk, and then reads the error plus the diff of what landed (`edit_session.rs:292-310`). `(source)`

**The one design idea worth taking** is the dirty-buffer prompt. Cursor's `Write` computes its diff from `fs.readFile`, so when a user has unsaved changes the diff shown to the model and the approval card does not match what the user sees, and the write silently wins on disk. Zed asks. For a harness with no editor attached this is moot, but if you ever attach one, the prompt is the correct behaviour and the omission is a data-loss bug. Confidence high.

**The one design idea worth rejecting**, with the highest confidence in this section, is swallowing the save error. `ensure_buffer_saved` ends each fallible step in `.log_err()` and `run_session` then reports a diff-based success from the in-memory buffer, while the system prompt tells the model not to re-read. A tool must return the I/O error it got (trap 14). Confidence high: the claim in the prompt is contradicted by the code on the failure path. A second reject is routing a whole-file write through the edit-session abstraction when there is no editor. The session exists to drive a `Buffer`, a `Diff`, and a `Project`; a virtual filesystem with none of those gets a layer of indirection that does nothing and a `No edits were made.` message that is wrong for a write, accurate for identical content and misleading for `content: ""` on a non-empty file. Confidence medium. `(source)`

### pi

**Source.** The pi coding agent, `git@github.com:earendil-works/pi.git` at `a92283056` (2026-10-27), `@earendil-works/pi-coding-agent` 1.4.0: `packages/coding-agent/src/core/tools/write.ts` (97 lines), `file-mutation-queue.ts`, `src/utils/paths.ts`, and the agent loop in `packages/agent/src/agent-loop.ts`. A second definition lives in `packages/durable/src/tools/write.ts`. `(source)` Cursor ships a port named `pi_write`, read from the 3.27.12 bundle and daemon. `(bundle)` `(daemon)` This section holds the whole pi account, the port included, so there is one place to read it.

**Pi's original against Cursor's port.** The port is a copy of an older pi with Cursor's own `Write` executor underneath, so most of the differences below originate with Cursor.

| Dimension | pi `write` | Cursor `pi_write` | Which side is better |
|---|---|---|---|
| Name | `write` | `pi_write`, behind feature flag `enablePiTools` | n/a |
| Parameters | `path`, `content`, both required strings | the same keys and the same two descriptions (`XRe` at 4554748) | match |
| Description | three sentences: creates, overwrites, creates parent directories (`write.ts:52-53`), plus the prompt line `Use write only for new files or complete rewrites.` | `Write content to a file using pi-compatible semantics.` and no steer | pi: the port drops all three facts |
| Success text | `Successfully wrote to ${path}` since commit `e583b290a` (2026-09-02) | `Successfully wrote ${args.content.length} bytes to ${args.path}` (daemon 16867601) | pi: the count is UTF-16 code units labelled bytes, the defect pi closed as issue 8979 |
| Bytes written | the string as UTF-8, verbatim (`write.ts:35`) | handed to Cursor's `Write` executor as `WriteArgs({path, fileText, toolCallId})`, which detects and preserves encoding and merges line endings | differs: the port is not byte-compatible with pi |
| Path rules | five Unicode-space classes (15 code points), leading `@`, Windows drive mapping, `~` and `~\`, `file://`, no trim (`src/utils/paths.ts:7,76-101`) | `trim()`, U+00A0 only, `@`, `file://`, `~` and `~/` (daemon 16838486) | pi is wider; both rewrite the write target, see the reject below |
| Queue registration | one global chain keeps call order (`file-mutation-queue.ts:5,33-49`) | none: compute the key, then take the slot (daemon 16837816) | pi |
| Queue key | `realpath(resolve(path))`, falling back to the resolved path only for `ENOENT` and `ENOTDIR`, rethrowing the rest (`:16-26`) | `realpath` of the resolved path, falling back to the path on every error (daemon 16837707) | pi: the port hides `EACCES` and `ELOOP` |
| Permission gate | none in the core, only an extension `tool_call` hook (`docs/security.md:3`) | none reachable: the executor is called directly and skips `performWrite` | match in effect |
| Read gate, freshness check | none | none; the executor's fingerprint is consumed only from a read in the same call and `pi_write` runs none (inferred) | match |
| Result transport | `{content:[text], details: undefined}` | proto `PiWriteExecSuccess {output}`, one string | same idea |

`(source)` for the pi column, `(daemon)` for the port column, offsets in the 20,213,626 byte daemon.

**Cursor's `Write` against `pi_write`.** What differs between Cursor's own two tools. `(bundle)` `(daemon)`

| Dimension | `Write` | `pi_write` |
|---|---|---|
| Content field | `contents` | `content` |
| Path field | `path` (or `file_path` on `dsv3-1018`) | `path`, required |
| Description | multi-branch, four variants | one line, no branches |
| Streaming of the content field | yes, `streamField:"contents"` feeds `editToolCallDelta` | no streaming path in the registration; `LG` parses the buffered args |
| Lock | taken during streaming by `withStreamingEditParsedArgs`, keyed on the raw argument | `LocalPiWriteExecutor` wraps the call in `withPiFileMutationQueue`, keyed on the `realpath` of the resolved path |
| Plan-mode gate | yes, at the write wrapper | none reachable: the executor is called directly and bypasses `performWrite` |
| Diff | computed by the tool and held in the record | not computed by the tool; the executor still snapshots the old content for tracking |
| Result the model sees | `Wrote contents to <path>` | `Successfully wrote ${content.length} bytes to ${path}`, a stale copy of an older pi |
| Path in the result | raw argument | raw argument (`args.path`) while the write lands at the resolved path |
| Encoding | full detect-and-preserve | the same executor, so the same behavior, which makes the port not byte-compatible with pi |
| Result oneof | `EditSuccess` with eight fields | `PiWriteExecSuccess { output }`, one string |

`pi_write` has no plan-mode gate: `LocalPiWriteExecutor.execute` calls `this.writeExecutor.execute(...)` directly, skipping `performWrite`, where `assertPlanModeAllowsFileEdit` lives. Whether that is reachable depends on whether pi tools are offered in plan mode at all, which the bundle does not settle, because the flag `enablePiTools` gates registration and the mode gates offering in different places. Pi itself has no plan mode and no approval layer in its core (`docs/security.md:3`), so the missing gate is faithful to pi and the open question is Cursor's offering policy. It is an asymmetry that has not been shown to be a hole. `(source)` `(daemon)`

My read: build one write implementation, expose it as `Write` with `contents` because that is the surface models know in Cursor, use pi's queue with its registration chain as the locking mechanism underneath, and keep pi's output-limit guard in the host. Accepting `content` as an alias costs one line and widens a schema that is strict today; confidence medium. A closed schema that names the valid keys in its refusal, as rig does, is a better answer to a wrong key than a bare validation failure. Confidence medium: the alias is cheap and the refusal text is untested against Cursor's models.

**Cursor's registration and schema, verbatim.** `pi_write` is registered at 4560410 in this build. The `AG("PI_WRITE"` call site is at 4560410 and the string `pi_write` first appears at 4560430, inside the same registration object; the offsets 2742643, 5794774, 5977742 and 5979878 are the tool-id enum, the metrics label map, and proto field tables, not registrations. `pi_write`'s own factory is `fMe` at 4560365, which sits immediately after the `pi_edit` factory's serializer, so a slice that starts at the factory name lands inside the right function, but a slice that starts ten bytes later lands in the previous tool's tail.

`(bundle)` at 4560410:

```js
AG("PI_WRITE",{
  name:"pi_write",
  descriptionGenerator:()=>"Write content to a file using pi-compatible semantics.",
  parameters:XRe,
  execute:LG(XRe,(async(e,r,n,s)=>{ ... new Bg.w0({path:lke(n.path),content:n.content}) ... }),dMe("piWriteToolCall",...)),
  render:iMe,
  serializeError:e=>dMe("piWriteToolCall",...)
})
```

The schema, `XRe` at 4554748:

```js
path:    string, required, .describe("Path to the file to write (relative or absolute)")
content: string, required, .describe("Content to write to the file")
```

**Parameters.** `content` is pi's own key (`write.ts:13`), and also the key in Claude Code, Copilot, Zed, rig, zeroclaw, Gemini CLI, OpenCode and Goose. Cursor's `Write` is the odd one out with `contents`, so a model moving between pi and Cursor's `Write` is the one that has to adapt. `(source)`

| Parameter | Type | Required | Description, verbatim |
|---|---|---|---|
| `path` | string | yes | `Path to the file to write (relative or absolute)` |
| `content` | string | yes | `Content to write to the file` |

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| `content` size | none | n/a | nothing in `write.ts`; bounded by what a model can emit in one message |
| Queue | one slot per canonical path | files | `file-mutation-queue.ts` |
| Message stopped on the output-token limit | every tool call in it fails | message | agent loop, `agent-loop.ts:274-279` |
| TUI preview | 10 lines collapsed, 50-line highlight window | lines | renderer only, never reaches the model (`renderers/write.ts:29,117`) |

**Output shape.** A real result for a relative path, from running the actual `write.ts` on win32 (the host had no `typebox`, `cross-spawn` or TUI renderer, which a resolve hook stubbed). `(probe)`

```text
Successfully wrote to a/b/c.txt
```

A thrown error reaches the model as one text block with `isError: true`, for example a target that is a directory:

```text
EISDIR: illegal operation on a directory, open '<absolute path>'
```

The port's reply for the same call, which is the stale form:

```text
Successfully wrote ${args.content.length} bytes to ${args.path}
```

**Behavior.** One in-place `fs/promises.writeFile(path, content, "utf-8")`, parent directories created first, no temp file, no rename, no mode argument, no read of the old file and no size cap. The queue slot is held across the mkdir and the write, and an abort is checked after each await rather than from an event listener so the slot is not released while a write is in flight (`write.ts:67-83`). `(source)` Overwriting a BOM file with BOM-less content drops the BOM, overwriting a UTF-16LE file turns it into UTF-8, and a CRLF file overwritten with LF content becomes LF. `(probe)` Pi's `edit` strips and restores the BOM (`edit.ts:190-197`) and the CRLF style (`edit-diff.ts:11-25`), so a file that `edit` touches keeps both and a file that `write` replaces does not. `(source)`

**The truncated-message guard.** When an assistant message stops with `stopReason === "length"`, the loop fails every tool call in it with a fixed text before lookup, validation or execution, and nothing runs: `(source)`

```text
Tool call "write" was not executed: the response hit the output token limit, so its arguments may be truncated. Re-issue the tool call with complete arguments.
```

The code comment gives the reason. Streamed arguments are finalized with a best-effort JSON salvage parser, so a cut-off message can produce a call whose arguments "parse and validate but are silently incomplete" (`agent-loop.ts:481-487`). For `write` that is the worst case, because the whole argument is the new file body and the tool replaces the old file with whatever arrived. The guard was added in commit `351efc828` (2026-07-07, #6285). It costs one retry and it also fails complete sibling calls in the same message. Pi therefore cannot replace a file with a truncated `content`. Cursor's `Write` has no equivalent check in its own path, and `StrReplace` guards only the other half of the problem, see D9.

**Concurrency, corrected.** Pi's queue is an in-process map from key to the tail of a promise chain, and `edit` takes the same queue, so a `write` and an `edit` of one file in one message serialize. What it keys on decides what it protects. Probes ran the real queue on win32: `(probe)`

| Case | Result |
|---|---|
| existing file, junction alias against the real path | serialized |
| NEW file, junction alias against the real path | overlapped |
| NEW file, spellings that differ in case | overlapped |
| NEW file, slash direction differs | serialized, `resolve()` normalizes it |
| three same-file callers, 200 trials, existing and new file | 0 of 200 out of call order, 0 overlapping |
| durable queue with the real `NodeExecutionEnv`, 200 trials | 133 of 200 out of call order for an existing file, 137 of 200 for a new file, 0 overlapping |

Pi's own docs admit the new-file hole: "For new files, it falls back to the resolved absolute path because there is nothing to `realpath()` yet" (`docs/extensions.md:2166-2168`). `(source)` So neither pi's queue nor the port serializes a new file reached through a different spelling, and a new file is the main job of `write`. The durable queue keys a missing file by its canonical parent plus its name, which closes the alias hole, and gave up call order to do it. Both runtimes execute sibling calls in parallel by default, so a `write` then an `edit` of one file in one message can run reversed in durable.

**What this corrects in the earlier text.**

- The `pi_write` reply is stale for pi. Cursor's port prints `${content.length} bytes`; pi dropped the count on 2026-09-02 in `e583b290a`, whose message says "The write tools reported UTF-16 code units as bytes. Remove the count from success messages instead of scanning content again." The earlier objection to the count was right, and it is an objection to the port, not to current pi. The port's reply for `café` says 4 bytes while 5 land on disk in UTF-8. `(source)` `(daemon)` `(probe)` The earlier suggestion that a byte count is "more informative than a bare sentence" is withdrawn. `(source)`
- The earlier claim that `pi_write`'s lock "serializes correctly across spelling differences" holds for an existing file only, and the port keys on the resolved path and falls back on every error, where pi falls back only for `ENOENT` and `ENOTDIR`. The earlier advice to copy `pi_write`'s lock now reads: copy pi's queue, with its registration chain and its narrow fallback, and decide on ordering deliberately.
- `content` is not a gratuitous divergence in pi. It is pi's own key and the majority key; the divergence is Cursor's `contents`.
- `resolvePiPath` in the port replaces only U+00A0. Pi's own `normalizePath` folds `[\u00A0\u2000-\u200A\u202F\u205F\u3000]`, five classes covering 15 code points, so the earlier remark about U+00A0 is right for the port and thin for pi. The port also trims, which pi does not, and has no `~\` and no Windows drive mapping.
- The port's `macReadVariants` replaces every space with U+202F (daemon 16839032), where pi rewrites only the space before `AM.` or `PM.` (`path-utils.ts:7-9`). Both are read-side helpers.
- Offsets: `queueKey` begins at daemon 16837707, not 16837500; `withPiFileMutationQueue` at 16837816; `LocalPiWriteExecutor` at 16866855.
- The earlier "same executor, so same encoding behaviour" is accurate for the port and means the port is not pi: pi writes UTF-8 verbatim.

**The one idea worth taking** is the truncated-message guard. It is a few lines in the harness and it needs no help from the tool. Runvil can do the same before any Pack tool runs. Confidence high: without it the failure is silent, irreversible, and looks like a successful write.

**The one idea worth rejecting** is rewriting the write target and then echoing the typed path. `write` strips a leading `@`, folds the space classes, and on Windows turns `/x` into drive `X:`, then answers `Successfully wrote to <typed path>` (`write.ts:65,86`). Probes show `@types/node/index.d.ts` landing at `types/node/index.d.ts`, and an overwrite of a file whose name contains U+202F creating a plain-space twin and leaving the original untouched, while the model was told it succeeded. Pi's own `read` does it the safe way, literal path first and variants only as a fallback (`path-utils.ts:52-83`). If a Pack accepts such leniency, apply it to reads only, and name the resolved path in the success text. Confidence high: the evidence is a reproduced silent misdirection and the fix is local. `(source)` `(probe)`

### rig

**Source.** Rig, a Rust agent library and terminal coding agent, `https://github.com/0xPlaygrounds/rig` at `a83b774d0` (2026-10-25): `crates/rig-tools/src/write.rs` (141 lines) with its one test file, and the shared layer in `workspace.rs`, `fs.rs` and `read.rs` of the same crate. The agent ships `read`, `edit`, `write` and `shell` "as pi has" (`plugins/rig-basics/src/coding_tools.rs:1-3`). `(source)`

**Parameters.** Name `write`. One name, no aliases, no per-model variants. `(source)`

| Parameter | Type | Required | Description, verbatim |
|---|---|---|---|
| `path` | string | yes | `The file to write.` |
| `content` | string | yes | `The whole new content.` |

The schema is closed. `args_schema` sets `additionalProperties: false` and `WriteArgs` also has `#[serde(deny_unknown_fields)]` (`write.rs:51`), so `contents` is refused before the tool runs, with a message that names the real keys (`feedback.rs:131-165`): `(probe)`

```text
`write` has no argument `contents`. Its arguments are: `path`, `content`. Call it again with only those.
```

The description is the only write guidance the model gets (`write.rs:76-79`):

```text
Write a file with the given content, replacing it if it exists and creating parent directories. Use it for new files and complete rewrites only; change part of a file with `edit`.
```

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| `content` size | none; an 18 MiB write returned `Created huge.txt.` and the next `read` of it refused the file | bytes | nothing in `write.rs`; reads cap at `MAX_FILE_BYTES` = 16 MiB |
| Old file read for the reply and the diff | 16,777,216 | bytes | `Local::read`; a larger file is marked unread and the write goes ahead |
| Diff time | 100 ms, 3 lines of context | ms, lines | `similar` timeout in `fs.rs:310-315` |
| Text `Seen` keeps after a write | 8 MiB across all files | bytes | `Seen::saw_whole` |

**Output shape.** Four sentences, built from the kind of change and the OLD file's line count, with the path as typed (`write.rs:130-138`). The two test-pinned forms (`write/tests.rs:36,60`) and the probe cases on win32 `(test)` `(probe)`:

```text
Created x/y/new.txt.
Replaced two.txt, which had 2 lines.
Replaced one.txt, which had 1 line.
Replaced big.txt.
```

The last form appears when the old line count is unknown, for example an old file over 16 MiB. The old line count is the number of `\n` bytes plus one for an unended last line, counted on raw bytes, so a file of three CR-ended lines counts as 1 and bytes `FF 0A FE 0A` count as 2. The diff, the hashes and the old text go to host-only `FilesChanged` metadata and never reach the model.

**Behavior.** `place` resolves and confines the path first. Then `Workspace::replace(path, bytes, limit)` writes and hands back what the file held before, in one operation, and the reply is built from that. On `Local` the write is `write_atomic` (`fs.rs:82-121`): refuse a non-regular or read-only target before creating anything, create a temp file in the target's directory with `File::create_new`, copy the permissions, `write_all`, `sync_all`, `rename`. No directory fsync. A symlink keeps its link and has its target replaced. `(source)` Costs of the rename: a new inode, the owner becomes this process, a hard link keeps the old text (probe: the hard link kept `orig`). `write` never reads `Seen` and never calls `write_if`, which `edit` does, so a `write` after an outside change succeeds with no check. `(source)`

A reply cut by the provider's output limit in the middle of a `write` is answered at spawn time with an `invalid_args` result and never reaches the tool (`rig-ecs/src/turn.rs:1412-1431`, text at `rig-core/src/tool/feedback.rs:57-62`). The text below is shortened; the real one echoes the first 200 characters of the raw arguments and then `… (N more characters)`: `(probe)`

```text
Not run: your reply reached the output limit before `write`'s arguments ended. Send less in one call: split the work into smaller calls, such as writing part of a file and adding the rest with edits. The arguments began: {"path": "big.rs", "content": "xxxx…
```

Corpus plugins, none shipped in the default agent, add a content-hash `read_before_edit` gate that covers `write` over an existing file (a new file is never gated, but the existing-file branch has no test, `read_before_edit/tests.rs:75,104`), leases, large-change approval, and an `auto_continue` pass that discards a cut `write` and tells the model to write the first part and add the rest with `edit`. `(source)` `(test)`

**The one idea worth taking** is `replace`, a write that hands back what it replaced, with the reply built from it. It tells the model the one thing it cannot know without another call, whether it made a file or replaced one, and it costs no extra round trip. Rig's first reply was `Wrote {N} bytes to {path}.`, and commit `e034438fa` (2026-10-17, "tool replies carry only what the model can't know") replaced it. For Runvil, `Access::write` could return the old size and hash from the claim it already takes, and the Pack would build the sentence. Do the old-content read inside the claim and before the replace, since that order is what makes `Created` mean something. Confidence high for the mechanism, medium that it changes model behavior. `(source)`

**The one idea worth rejecting** is an unconditional overwrite with the staleness check kept in a different tool. `write` ignores the `Seen` hash and `write_if` that `edit` uses, so one file has two safety levels depending on which tool the model picks, and the only trace of a lost outside change is `, which had N lines`, which arrives after the old file is gone. Confidence medium: the code path is certain and a probe confirms no check, but no transcript shows a model losing work to it. `(source)` `(probe)`

### zeroclaw

**Source.** `https://github.com/zeroclaw-labs/zeroclaw` at `2d85371f3` (2026-09-18), workspace version 0.8.5: `crates/zeroclaw-tools/src/file_write.rs`, wrapped by `RateLimitedTool(PathGuardedTool(...))` (`tools/mod.rs:344-350`), policy in `zeroclaw-config/src/policy.rs`. `(source)` Name `file_write`; one name and one description, no per-model variants.

**Parameters.**

| Parameter | Type | Required | Description, verbatim |
|---|---|---|---|
| `path` | string | yes | `Path to the file. Relative paths resolve from workspace; outside paths require policy allowlist.` |
| `content` | string | yes | `Content to write. UTF-8 text when encoding is 'utf8'; base64-encoded bytes when encoding is 'base64'.` |
| `encoding` | string, `utf8` or `base64` | no, default `utf8` | `How to interpret 'content' before writing (default: 'utf8'). Use 'base64' for binary files.` |

It is the only harness here with an explicit binary channel. Description (`file_write.rs:42`): `Write contents to a file in the workspace. Text by default; set encoding="base64" to write binary files (e.g. .xlsx/.docx) by decoding base64 content into raw bytes.`

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| `content` size | none | n/a | no length check in `file_write.rs` |
| Action budget | 20 successful calls per 3600 s per sender, reads included | calls | `RateLimitedTool` (`wrappers.rs:63-82`); a failed call does not spend budget |
| Result sent to the model | 50,000 by default, `0` disables | compared with `output.len()`, so bytes, though the field says chars | agent loop, `history.rs:101-149` |

**Output shape.** The reply counts the decoded bytes and echoes the path as sent. The repo test asserts the substring `8 bytes` for the content `written!` (`file_write.rs:323-340`). `(source)`

```text
Written 8 bytes to out.txt
```

**Behavior.** Everything that can fail is checked before the first filesystem mutation: autonomy, the ephemeral-runtime refusal, the encoding name and the base64 decode, pinned by `file_write_rejected_encoding_does_not_create_parent_dirs` (`file_write.rs:553-607`). Then `create_dir_all(parent)`, then the parent is canonicalized and checked against the allow roots, so a directory can be created outside the boundary before the check refuses (from reading the order; the symlink test does not cover it and was not run). The leaf is refused if it is a symlink, runtime config files are refused, and the write is `tokio::fs::write` in place with no temp file and no fsync (`file_write.rs:155-226`). `(source)` Empty `content` truncates and answers `Written 0 bytes to {path}`. Overwriting a CRLF file with LF text changes the endings, since nothing detects the old style. The reply cannot tell created from overwritten.

**The one idea worth taking** is to validate and decode everything that can fail before the first mutation, and to pin it with a test, so a rejected call leaves the tree untouched. Confidence high: the test asserts the parent directory does not exist after a rejection.

**The one idea worth rejecting** is `create_dir_all` ahead of the canonical boundary check, together with a reply that cannot tell created from overwritten (`file_write.rs:166` before `:180`, and `:229`). Resolve the deepest existing ancestor first, as `resolve_symlinked_path` already does (`policy.rs:829-886`), and report `created` or the old size. Confidence medium: the first half comes from reading order, not from a run.

### Codex CLI

**Source.** `https://github.com/openai/codex` at `bea47c06dfac12ecce362090418e12a6faeedbf9` (committed 2026-10-28, workspace version `0.0.0`, so no release number): `codex-rs/apply-patch/src/lib.rs`, `codex-rs/core/src/tools/handlers/apply_patch_spec.rs`, `codex-rs/core/assets/tools/apply_patch.lark`. `(source)` `(web)` for the commit metadata.

**There is no write tool.** The model-visible file tools are `exec_command`, `apply_patch` and `view_image`. A new file is `*** Add File: <path>` inside `apply_patch`, a freeform custom tool with a Lark grammar and no JSON parameters (`apply_patch_spec.rs:20-27`). Overwriting an existing file is also `Add File`, and it replaces silently; the other route is delete then add, or a shell redirect that the shipped templates tell the model not to use. What that implies for Runvil: the whole-file write is one hunk kind in a patch language, and the model never names a `content` field. `(source)`

| Parameter | Type | Required | Description |
|---|---|---|---|
| the patch text | raw freeform string, `format.type` = `grammar`, `syntax` = `lark` | yes | `` The `apply_patch` tool can be used to edit files. This is a FREEFORM tool, so do not wrap the patch in JSON. `` |

The Add hunk in the grammar (`apply_patch.lark`):

```text
add_hunk: "*** Add File: " filename LF add_line+
add_line: "+" /(.*)/ LF -> line
```

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| Bytes read per file for Update and Delete | 536,870,912 (`512 * 1024 * 1024`) | bytes | `exec-server/src/local_file_system_read.rs:13` |
| Result shown to the model | 10,000 by default | approximate tokens, 4 bytes each | `core/src/tools/mod.rs:92-117`, `models.json` `truncation_policy` |
| Patch size, hunk count, files per patch | not in the sources | | |

**Output shape.** The summary groups Added, then Modified, then Deleted, whatever the patch order was, and does not say that a file was overwritten (regex in the repo test, `core/tests/suite/apply_patch_cli.rs:459-467`; builder at `apply-patch/src/lib.rs:826-841`): `(source)`

```text
Exit code: 0
Wall time: 0.1 seconds
Output:
Success. Updated the following files:
A nested/new.txt
```

**Behavior.** The whole patch is parsed and every target is verified against the disk before any write and before approval, so one bad hunk means no file is touched (test `apply_patch_cli.rs:1471-1489`). After approval the patch is re-derived from the disk as it is then, so a file edited during the approval wait is re-matched and not overwritten from a stale copy. Hunks then apply in patch order with no transaction. An Add reads any existing file once, only to build the undo delta, writes with `tokio::fs::write` in place, and on `NotFound` creates the parent directories and retries once (`lib.rs:473-500,765-822`). Every added line ends with `\n` and a trailing `\r` is stripped (`streaming_parser.rs:143,206`), so an Add cannot write CRLF, a mixed file, or a file without a final newline. There is no read gate and no lock. The gate is the OS sandbox plus an approval policy, not a per-tool check. `(source)`

**The one idea worth taking** is verify-everything-then-apply, with the verification re-run against the disk after approval. A refused or failed check leaves the tree untouched, and a stale approval cannot overwrite a newer file. Confidence medium: it is built for multi-file patches, and a single-file `Write` gets the same property from one read inside the claim.

**The one idea worth rejecting** is `Add File` silently replacing an existing file behind a success line (`A path`) that cannot tell the model it clobbered something (test `apply_patch_cli.rs:682-700`). Confidence high: there is no read gate here either, so nothing else catches it.

### Gemini CLI

**Source.** `https://github.com/google-gemini/gemini-cli` at tag `v0.66.0`, commit `970413207732f5d4d8c50540c7fc359c44fdaaf2` (2026-10-27): `packages/core/src/tools/write-file.ts`, `core/services/fileSystemService.ts`, `core/utils/textUtils.ts`, `core/tools/omissionPlaceholderDetector.ts`. `(source)` Name `write_file`. Two schema sets exist by model family (`gemini-3` and a legacy default); the parameter names are identical and only the descriptions differ.

**Parameters.** `(source)`, `core/tools/definitions/default-legacy.ts:111-131` and `gemini-3.ts:120-138`.

| Parameter | Type | Required | Description, `default-legacy` | Description, `gemini-3` |
|---|---|---|---|---|
| `file_path` | string | yes | `The path to the file to write to.` | `Path to the file.` |
| `content` | string | yes | `The content to write to the file. Do not use omission placeholders like '(rest of methods ...)', '...', or 'unchanged code'; provide complete literal content.` | `The complete content to write. Provide the full file; do not use placeholders like '// ... rest of code'.` |

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| `content` size | none | JS string | not in the sources |
| Path | 4,096 total, 255 per component | `String.length` | `path-validator.ts:20-21` |
| Result echoed to the model | 64 KiB | UTF-8 bytes | scheduler, `tool-executor.ts:340-351` |

**Output shape.** The reply includes a diff snippet so the model can skip a verification read: context around the changes (the tool passes 5 as the context size), and the whole file for a new file (`write-file.ts:511-529`). The sample below is the shape from the repo's own test at `diff-utils.test.ts:43-49`, which uses one context line. `(source)`

```text
Successfully overwrote file: /abs/path/a.txt. Here is the updated code:
...
5
added
6
...
```

A new file reads `Successfully created and wrote to new file: <abs>.` followed by the same snippet, which is the entire content, so a large new file is paid for twice in tokens.

**Behavior.** Validation checks the path and workspace, refuses a directory, and refuses content with an omission placeholder, all before the user is asked. Then the policy gate (`ask_user` interactive, `deny` headless, `allow` in `autoEdit`), a confirmation diff, an in-process per-path lock keyed on the realpath, a re-read of the old file (an unreadable existing file stops the write), and for some model families a rewrite of the content by `unescapeStringForGeminiBug` (`write-file.ts:204-225`). Missing parent directories are created. The write is a temp file `<realpath>.<uuid>.tmp`, the destination's `mode & 0o777` copied onto it, then `rename`, with up to 5 retries on `EBUSY`, `EPERM` and `EACCES` at `2^attempt * 50` ms (`fileSystemService.ts:35,53-138`). `(source)`

Line endings are restored, all or nothing. `(source)`

```ts
      const useCRLF =
        !isNewFile && originalContent
          ? detectLineEnding(originalContent) === '\r\n'
          : os.EOL === '\r\n';

      if (useCRLF) {
        finalContent = finalContent.replace(/\r?\n/g, '\r\n');
      }
```

`detectLineEnding` is `content.includes('\r\n') ? '\r\n' : '\n'` (`textUtils.ts:93-97`), so one `\r\n` anywhere makes the whole file CRLF and a mixed file becomes uniform CRLF; a CRLF payload sent to an LF file is written as is, with no step back to LF. A BOM is never added and an existing one is dropped, and a lone surrogate is written as `EF BF BD`. `content: ""` is legal: validation checks only the path and the placeholders (`write-file.ts:655-715`), so an empty file is written. `(source)` `(probe)`

The placeholder detector is narrow. It looks at each line, strips a leading `//` and a pair of parentheses, and refuses only a line whose text before `...` is one of `rest of`, `rest of method(s)`, `rest of code`, `unchanged code` or `unchanged method(s)` (`omissionPlaceholderDetector.ts:7-83`). It catches a model that elides code and it does not catch a message that was cut. `(source)`

**The one idea worth taking** is the write primitive: a temp file in the same directory, the destination mode copied before the rename, the realpath so a symlink is updated in place, and a bounded retry for Windows sharing errors. A reader sees the old file or the new one. Confidence high.

**The one idea worth rejecting** is silently rewriting the model's content with `unescapeStringForGeminiBug` for some model families. A probe on the real regex turns the source text `console.log("a\nb");` (a backslash and an `n` in the file) into a line break, with no notice to the model. A faithful write tool writes the bytes it was given. Confidence high. `(source)` `(probe)`

### OpenCode

**Source.** `https://github.com/anomalyco/opencode` (formerly `sst/opencode`) at tag `v1.19.5`, commit `810c0eba1a7348036f8bdf61e6632d0c36c6876c`: `packages/opencode/src/tool/write.ts` and `write.txt` (the set the session loop uses), the V2 rewrite in `packages/core/src/tool/write.ts`, and the next generation at dev `fa1260d6ca` in `packages/core/src/tool/plugin/write.ts`. `(source)` Three implementations coexist, and which one the TUI drives is not in the sources. Name `write`. Models whose id contains `gpt-` (not `oss`, not `gpt-4`) get `apply_patch` and no `write` (`registry.ts:297-300`), the same split Cursor makes for its GPT-5 family.

**Parameters.** `(source)`, `write.ts:20-25`.

| Parameter | Type | Required | Description, verbatim |
|---|---|---|---|
| `content` | string | yes | `The content to write to the file` |
| `filePath` | string | yes | `The absolute path to the file to write (must be absolute, not relative)` |

The schema says absolute, and the code joins a relative path to the instance directory (`write.ts:41-43`, pinned by `write.test.ts:88-95`). V2 and dev name the path field `path`. The description promises a gate: `If this is an existing file, you MUST use the Read tool first to read the file's contents. This tool will fail if you did not read the file first.` A search of both source trees for `FileTime`, `readBefore`, `modified since` and similar finds no check, so the only gate is the prompt text.

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| Other-file diagnostics in the reply | 5 | files | `MAX_PROJECT_DIAGNOSTICS_FILES`, `write.ts:18,81` |
| Reply text | 2,000 lines or 51,200 bytes, head cut, rest saved to a file | lines and bytes | the generic wrapper, because `write` sets no `metadata.truncated` (`tool.ts:131`, `truncate.ts:14-15`) |
| `content` size | none | n/a | not in the sources |

**Output shape.** The first line is pinned by `write.test.ts:69`; the diagnostics block follows only when the language server reports errors. V2 says `Wrote file successfully: <resource>` or `Created file successfully: <resource>` (`core/src/tool/write.ts:58-59`). `(source)`

```text
Wrote file successfully.

LSP errors detected in this file, please fix:
<block from LSP.Diagnostic.report>
```

**Behavior.** The tool resolves the path and checks the external-directory rule, then reads the old file with a BOM-aware decode. It builds a unified diff and asks permission `edit` with that diff, so the person approves a diff against content that may have changed since, and nothing re-checks it. It then writes with `writeWithDirs`, which on `NotFound` creates the parent and writes again. After the write it runs the formatter, restores the BOM if the formatter dropped it, publishes `file.edited`, and touches the language server to read diagnostics (`write.ts:36-100`). The BOM rule keeps the old file's BOM or one the new content starts with, and emits at most one. Line endings are written as given, so LF content over a CRLF file converts it. There is no lock (`edit` serializes per path and `write` does not). Empty `content` writes a zero-byte file (`write.test.ts:215-226`). The formatter may rewrite the file and the reply never says so. `(source)`

**The one idea worth taking** is the V2 split of mutation primitives into `create` (exclusive `wx` flag), `write`, `writeTextPreservingBom`, `writeIfUnchanged` and `remove`, each returning an `existed` flag so the reply can say `Created` or `Wrote` (`packages/core/src/file-mutation.ts:54-66,108-142`). It makes "did I overwrite something" a fact in the result. Confidence medium. `(source)`

**The one idea worth rejecting** is a prompt that promises a gate the code does not have. A model that trusts the promise skips nothing and a model that ignores it loses nothing, so the sentence only misleads. If the gate is wanted, make it a hash compare, and keep the text honest. `writeIfUnchanged` exists in V2 and V2 `write` does not call it. Confidence high. `(source)`

### Cline

**Source.** `https://github.com/cline/cline` at tag `v4.1.26` (`a459f1d`, 2026-10-25), SDK executor `sdk/packages/core/src/extensions/tools/executors/editor.ts`; the legacy line at tag `v4.0.12` (`f267bf4`, 2026-07-28) had a `write_to_file` tool. `(source)`

**There is no separate write tool in v4.1.26.** Creating a file is a mode of `editor`: when the path does not exist, `editor` writes `new_text` to it (`editor.ts:246-248`). `apply_patch` creates with `*** Add File:` and refuses an existing path. Overwriting an existing file wholesale has no tool path: `editor` without `old_text` errors, Add errors, and delete-then-add of one path errors with `Duplicate add for file: o.txt` (probe). The remaining route is the shell. GPT and Codex models get `apply_patch` and never see `editor`. `(source)` `(probe)`

| Parameter (create mode) | Type | Required | Note |
|---|---|---|---|
| `path` | string | yes | schema `min(1)` (`schemas.ts:200-203`) |
| `new_text` | string | yes | the file body; `old_text` and `insert_line` are unused in this mode |

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| `new_text` | 6,000, `INPUT_ARG_CHAR_LIMIT` | UTF-16 code units | the tool wrapper, before the executor runs (`helpers.ts:21-33`) |
| Operation timeout | 30,000 | ms | `definitions.ts:715` |
| Result strings at provider-request time | 8,000 per string | characters | `MessageBuilder`, applies to results, not to the write |

**Output shape.** `(source)`, `editor.test.ts:47`, same string in a probe:

```text
File created successfully at: C:\Users\...\example.txt
```

An oversized body is refused with a message that explains the reason: `Editor input too large: new_text was N characters, exceeding the recommended limit of 6000. Split the edit into smaller tool calls so later tool calls are less likely to be truncated or time out.`

**Behavior.** `fs.access` decides create against edit, and a missing path creates even when `old_text` was supplied, so a mistyped path on a replace silently becomes a new file (probe: `{path:"ghost.txt", old_text:"foo", new_text:"bar"}` created `ghost.txt`). `mkdir -p` then `fs.writeFile`, in place. A new file on a host whose `os.EOL` is CRLF gets CRLF unless the content already holds a `\r` (`line-endings.ts:12-17`); on Windows `hello\n` wrote the bytes `hello\r\n` (probe). Encoding is UTF-8 only and a lone surrogate becomes U+FFFD. The legacy tool detected a charset with `jschardet` and re-encoded with `iconv-lite`. There is no lock, and tool calls inside one response run sequentially unless the host opts in, so the race needs sub-agents or two sessions. `(source)` `(probe)`

**The one idea worth taking** is the new-file line-ending decision made by content: if the model's own text contains `\r` it chose its endings, otherwise use the host's. A Runvil `Write` that runs on Windows and Linux needs this decision made explicitly rather than inherited from the language runtime. Confidence medium: the right Runvil default depends on whether the VFS is host-native.

**The one idea worth rejecting** is create-on-missing inside an edit tool, which turns a path typo into a new file and a success message. Keep `Write` and `StrReplace` as separate tools so intent is typed. Confidence high: the executor and a probe agree. `(source)` `(probe)`

### Aider

**Source.** `https://github.com/Aider-AI/aider` at `5dc9490bb35f9729ef2c95d00a19ccd30c26339c` (2026-05-22, `__version__` `0.86.3.dev`): `aider/coders/wholefile_coder.py`, `aider/coders/base_coder.py`, `aider/io.py`. `(source)`

**There is no write tool, and the model never calls a function.** It writes edit text inside its reply, and Aider parses that text and does the file work itself. Four mechanisms create or overwrite a file: the `whole` format, an empty-SEARCH block in the `diff` format, a udiff whose first path is `/dev/null`, and `*** Add File:` in the `patch` format, which refuses an existing path (`ADD Error: File already exists`). The only schema Aider ever defined for writing is a retired `write_file` function that is kept in the tree and unreachable, because its constructor raises (`wholefile_func_coder.py:47`). What that implies for Runvil: a design where a human picks the files and the model only edits them needs no read gate, no staleness hash and no confinement for reads, and moves each of those problems onto the human and onto prompt text. `(source)`

| Retired parameter | Type | Required | Description |
|---|---|---|---|
| `explanation` | string | yes | `Step by step plan for the changes to be made to the code (future tense, markdown format)` |
| `files[].path` | string | yes | `Path of file to write` |
| `files[].content` | string | yes | `Content to write to the file` |

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| Filename in the whole format | longer than 250 becomes empty, then falls to fallbacks | characters | parser, `wholefile_coder.py:64-66` |
| Content size | none; bounded only by the model's output limit | n/a | not in the sources |
| Write retries | 5 attempts at 0.1, 0.2, 0.4, 0.8 s, on `PermissionError` only | attempts, seconds | `io.write_text`, `io.py:478-507` |

**Output shape.** After a write Aider prints a line per file, and a declined creation prints `Skipping edits to {path}` (`base_coder.py:2208,2230,2330-2334`): `(source)`

```text
Applied edit to notes.txt
```

**Behavior.** The permission prompt runs after parsing and before any write, so a declined prompt costs nothing. `dirty_commit` then commits the user's uncommitted changes to that file first. If the order were swapped the user's work and the model's rewrite would land in one commit and `/undo` would discard both (reasoning, not a tested case). The write opens the existing path with mode `"w"` in place, using the session encoding, strict, and `newline=None` by default, which turns each `\n` into `os.linesep`: an LF file round-tripped to CRLF on Windows (probe). The file always ends with a newline and cannot express "no final newline". Empty fenced block truncates the file to 0 bytes with no guard. A lone surrogate raises `UnicodeEncodeError` after the file was already truncated to 0 bytes (probe: size 0 afterward). A literal `... rest of code ...` inside a whole-format block is written to disk, and the only defense is the prompt line forbidding it. `(source)` `(probe)`

**The one idea worth taking** is the ordering of the dirty commit ahead of the write, so a later undo discards only the model's change and not the user's. For Runvil it is a checkpoint taken inside the same claim before the write. Confidence medium: the swap consequence is the research's reasoning and not a measured case.

**The one idea worth rejecting** is truncate-in-place with no temp file and no empty-content check. A model that emits an empty block, or a text that fails to encode, leaves a 0-byte file. Confidence high: the evidence is `io.py:493-494` and the probe.

### Goose

**Source.** `https://github.com/aaif-goose/goose` (formerly `block/goose`) at `ee8500d5e060eea6f2bcee92e8a5e351f12c3a70` (2026-10-28), release tag `v1.56.0` (`9949cf9`): `crates/goose/src/agents/platform_extensions/developer/edit.rs:40-76`, registered at `dev/mod.rs:110-121`. `(source)` The developer extension is compiled into the `goose` crate and the model sees the unprefixed name `write`. It ships exactly `write`, `edit`, `shell`, `tree` and `read_image` (pinned by a test at `dev/mod.rs:271-279`). The older `text_editor` tool, with view, write, str_replace, insert and undo, was removed on 2026-02-26 (PR #7466).

**Parameters.** The structs have no doc comments, so the schema has no per-field descriptions. `(source)`

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | yes | none |
| `content` | string | yes | none |

Tool description, verbatim: `Create a new file or overwrite an existing file. Creates parent directories if needed.` Annotations are title `Write` with the raw tuple `(false, true, false, false)`, which reads as destructive and not read-only (the mapping order was not read).

**Caps.**

| Limit | Value | Unit | Layer |
|---|---|---|---|
| `content` size, path length, directory depth | none | n/a | not in the tool |
| Tool response | 200,000, over which the text goes to a temp file and one sentence replaces it | `chars().count()` | `large_response_handler.rs:5-11` |

**Output shape.** The reply names the verb, the path as the model gave it, and a line count from `str::lines().count()`. No test pins the strings, so these come from the format strings and a probe of the counting: `""` is 0 lines, `"a\n"` is 1, `"a\n\n"` is 2. `(source)` `(probe)`

```text
Created notes.txt (2 lines)
Wrote notes.txt (1 lines)
```

**Behavior.** Resolve the path (absolute stays, otherwise the working directory joined with the string; no `~` expansion, no `..` folding, no confinement), `create_dir_all` the parent, compute `is_new` before the write, then `fs::write` once, in place. Bytes are written as given. There is no lock, no read gate, no stale check, no fsync. The `Created` or `Wrote` word is decided by an `exists()` before the write, a check-then-use pair that affects only the word. Probes on Windows: `~/notes.txt` becomes `C:\proj\work\~/notes.txt`, creating a directory literally named `~`, and `../outside.txt` escapes the working directory. `(source)` `(probe)` The removed `text_editor` forced a trailing newline and the platform line ending.

**The one idea worth taking** is the `Created` or `Wrote` verb with a line count: one cheap fact that tells the model whether it just replaced something. Confidence medium: it is advisory and computed before a non-atomic write.

**The one idea worth rejecting** is an unconditional `fs::write` with no create-only mode and no confinement, while the same crate already has a writer that opens every path component with `O_NOFOLLOW`, supports `O_EXCL` and refuses files with more than one hard link (`skills/supporting_files.rs:80-85,319-367`), which serves skills and not the model's tools. Confidence high: both implementations are in one crate.

### Side by side, all harnesses

Three tables with the same rows, because thirteen columns do not fit on one page. Each repeats Cursor so the comparison is local. A cell that says `not checked` means this pass did not read that harness for that row, and `not in the sources` means the notes looked and found nothing to cite.

| Dimension | Cursor | Claude Code | Copilot | Zed | pi |
|---|---|---|---|---|---|
| Tool name | `Write` (`write` on `dsv3-1018`) | `Write` | `create_file` | `write_file` | `write` (port: `pi_write`) |
| Path and content keys | `path` (`file_path` on one branch), `contents` | `file_path`, `content` | `filePath`, `content` (optional in the type) | `path`, `content` | `path`, `content` |
| Overwrites an existing file? | yes, unconditionally | yes, if read first | **no**, refuses; an existing empty file is allowed | yes | yes |
| Paging, append | none | none | none | none | none |
| Content cap | none | none in the tool | none | none; an existing file of 6 GiB or more is refused | none |
| Result cap | one sentence | 100,000 chars | one sentence | one sentence | one sentence |
| Matching rule | n/a | n/a | n/a | n/a | n/a |
| Encoding | detected and preserved, BOM kept | existing encoding kept; legacy encodings, NUL files and UTF-16 BE refused | editor's own | detected (BOM, null bytes, `chardetng`), BOM kept | UTF-8 verbatim |
| Line endings | detected, preserved, per-line merge | normalized to LF on read, LF written | editor's own | first newline in 1000 bytes decides, whole file uniform | as sent |
| Stale-read protection | sha256 fingerprint, tool's own read to truncate, 8 MB cap | read-state store, mtime then content hash | none | none for write: computed and dropped | none |
| Read-before-write gate | **none** (the prompt says MUST) | **yes**, a real state machine | no, but creation-only | no, but a dirty-buffer prompt | none |
| Gating | plan mode (markdown only), approval card out of workspace | not checked | editor review | `authorize_file_edit`, prompts on symlink escape, `.zed`, skills | none in the core, extension hook only |
| Write primitive | truncate in place, then fsync | not checked | editor `textEdit` | buffer save, truncate in place, no fsync | `writeFile` in place |
| Protection from a cut message | none in the Write path; the loop retries a provider output-limit error | not checked | not checked | a failure after the session saves what streamed | fails every call in a `length`-stopped message |
| Result on success | `Wrote contents to <path>` | `File created successfully at: <path>` or `The file <path> has been updated successfully.` | `File created at <path>` | `Edited <path> successfully`, or `No edits were made.` | `Successfully wrote to <path>` |
| Diff or old-file fact in the result | no, structured only | yes, `structuredPatch` | no | no on success; yes inside an error after the session exists | no |
| Empty content | truncates, success | not checked | allowed for a new or empty file | no-op on an existing file | truncates |
| Reverts | `recordFileState` plus a separate revert log | editor, git | VS Code undo | the action log baseline and `reject_all_edits` | not in the sources |

Caption: Cursor, Claude Code, Copilot, Zed and pi. Only Cursor and Claude Code gate or fingerprint a write, and only pi guards against a cut message. `(source)` `(bundle)` `(daemon)` `(probe)`

| Dimension | Cursor | rig | zeroclaw | Codex CLI | Gemini CLI |
|---|---|---|---|---|---|
| Tool name | `Write` | `write` | `file_write` | none; `apply_patch` with `*** Add File:` | `write_file` |
| Path and content keys | `path`, `contents` | `path`, `content`, closed schema | `path`, `content`, `encoding` | inside the patch, `+` lines | `file_path`, `content` |
| Overwrites an existing file? | yes, unconditionally | yes | yes | yes, silently, through `Add File` | yes |
| Paging, append | none | none | none | none | none |
| Content cap | none | none; reads cap at 16 MiB | none | none in the code read | none |
| Result cap | one sentence | one sentence | 50,000 bytes | 10,000 tokens | 64 KiB |
| Matching rule | n/a | n/a | n/a | n/a | n/a |
| Encoding | detected and preserved | UTF-8 verbatim | UTF-8 verbatim, or base64 bytes | UTF-8, no BOM added | UTF-8, BOM dropped, some models get an unescape rewrite |
| Line endings | detected, preserved, per-line merge | as sent | as sent | always LF, trailing `\r` stripped | all-or-nothing CRLF if the old file has any `\r\n` |
| Stale-read protection | fingerprint, 8 MB cap | `Seen` hash exists; `write` ignores it | none | patch re-derived from the disk after approval | none |
| Read-before-write gate | **none** | none; corpus `read_before_edit`, unshipped | none | none | none |
| Gating | plan mode (markdown only), approval card out of workspace | none shipped; corpus gates | approval by default, path guard, autonomy level | sandbox plus approval policy | policy: `ask_user`, `deny` headless, `allow` in `autoEdit`; workspace check |
| Write primitive | truncate in place, then fsync | temp, `sync_all`, rename, permissions copied | in place, no fsync | in place | temp, mode copied, rename, Windows retry |
| Protection from a cut message | none in the Write path | kernel refuses a cut call | none | none | placeholder detector only |
| Result on success | `Wrote contents to <path>` | `Created <p>.` or `Replaced <p>, which had N lines.` | `Written N bytes to <path>` | `A <path>` in an `Exit code: 0` block | `Successfully overwrote file: <abs>.` plus a diff snippet |
| Diff or old-file fact in the result | no | old line count; the diff is host-only | no | no | yes, snippet around the change |
| Empty content | truncates, success | truncates, `which had N lines` | truncates, `Written 0 bytes` | empty file (derived) | truncates, legal |
| Reverts | revert log | `FilesChanged` facts; the shipped checkpoint does not wrap `write` | not in the sources | undo delta inside the patch run | not in the sources |

Caption: Cursor, rig, zeroclaw, Codex CLI and Gemini CLI. Rig and Gemini write atomically, and rig's reply is the only one that reports the old file's line count. `(source)` `(probe)`

| Dimension | Cursor | OpenCode | Cline | Aider | Goose |
|---|---|---|---|---|---|
| Tool name | `Write` | `write` (GPT family gets `apply_patch`) | none; `editor` create mode | none; text formats | `write` |
| Path and content keys | `path`, `contents` | `filePath`, `content` (V2: `path`) | `path`, `new_text` | filename before the fence, the fenced block | `path`, `content` |
| Overwrites an existing file? | yes, unconditionally | yes | no tool path (editor needs `old_text`, Add refuses) | yes in `whole`, refused by `patch` `Add File` | yes |
| Paging, append | none | none | `insert_line = line_count + 1` appends | none | none |
| Content cap | none | none | 6,000 characters at the wrapper | none | none |
| Result cap | one sentence | 2,000 lines or 51,200 bytes | one sentence | one line | 200,000 chars |
| Matching rule | n/a | n/a | n/a | n/a | n/a |
| Encoding | detected and preserved | UTF-8, BOM kept | UTF-8; invalid bytes become U+FFFD on rewrite | session encoding, strict | UTF-8 verbatim |
| Line endings | detected, preserved, per-line merge | as sent | new file: host EOL unless content has `\r` | platform default (`newline=None`) | as sent |
| Stale-read protection | fingerprint, 8 MB cap | none (V2 has `writeIfUnchanged`, `write` skips it) | none | none; dirty-commit first | none |
| Read-before-write gate | **none** | none (the prompt says it will fail) | none | none; the human picks files | none |
| Gating | plan mode (markdown only), approval card out of workspace | permission key `edit`, default allow | approval; lexical cwd check for relative paths only | confirm prompt for new or non-chat files | permission mode by tool name |
| Write primitive | truncate in place, then fsync | in place, then formatter | in place | in place | in place, no fsync |
| Protection from a cut message | none in the Write path | none | 6,000-character argument cap | guidance text only | none |
| Result on success | `Wrote contents to <path>` | `Wrote file successfully.` plus LSP errors | `File created successfully at: <abs>` | `Applied edit to <path>` | `Created <p> (N lines)` or `Wrote <p> (N lines)` |
| Diff or old-file fact in the result | no | no; the diff goes to the approval | no | no | line count of the new content |
| Empty content | truncates, success | truncates | creates an empty file | empty block truncates to 0 bytes | truncates, `(0 lines)` |
| Reverts | revert log | not in the sources | not in the sources | `/undo` through git | none found |

Caption: Cursor, OpenCode, Cline, Aider and Goose. Cline and Aider have no write tool, and none of the five gates on a prior read; OpenCode's prompt claims a gate its code does not have. `(source)` `(probe)`

The column that surprises people is line endings. Cursor detects and preserves them, with a per-line merge that nobody else has, and it is not alone in detecting them. Gemini CLI restores CRLF all or nothing, Cline restores it on the matched span of an edit and picks the host's ending for a new file, Zed normalizes the whole file to the style of its first newline, and Claude Code normalizes CRLF to LF on read (`Hy` in `freshness-checks.js`) and writes LF. Copilot inherits its editor's save path. pi, rig, zeroclaw, Goose and OpenCode write the bytes as sent, and Codex cannot write CRLF at all. The earlier sentence that only Cursor detects and preserves line endings held for the four harnesses first compared and does not hold for the set.

### Zed checkout drift

The first pass read `C:\Users\Vinnie\src\zed` at `d9ad6aff67` (2026-08-22). Every Zed claim above was reconciled against the canonical checkout `C:\Users\Vinnie\cursor\zed` at `37faef81f7` (2026-10-27). `git diff --no-index --ignore-cr-at-eol` over `write_file_tool.rs`, `edit_session.rs`, `tool_permissions.rs`, `streaming_parser.rs`, `reindent.rs` and `streaming_fuzzy_matcher.rs` shows no change in the `write_file` code path itself. The table lists what the earlier text said, what is true now, and why. `(source)`

| Claim in the earlier text | What it said | What is true now |
|---|---|---|
| Source files and sizes | 56 KB, 45 KB and 55 KB in `C:\Users\Vinnie\src\zed` | 55,186, 43,656 and 53,356 bytes at `37faef81f7`. The old files are CRLF (1,466, 1,299 and 1,477 CR bytes), so the apparent shrink is line-ending noise: 56,510 minus 1,466 is 55,044, and the new file is 142 bytes larger |
| Result the model sees on success | `Result cap: a sentence plus the diff`, `Diff in the result: yes, in the text` | One sentence, `Edited <path> successfully`, or `No edits were made.` when the diff is empty (`edit_session.rs:96-107`). The diff, `old_text` and `new_text` go to the UI as `raw_output`; the diff reaches the model only inside an error after the session exists |
| Error text | the error, then a diff block when there is an input path and a non-empty diff | `{error}\n`, then the diff block, or `No edits were made.` when there is no input path or no diff (`edit_session.rs:108-125`) |
| Undo | "undoable through the editor's own undo stack, not a separate revert log" | Reverts go through the action log. `buffer_created` records the old file content as the baseline and `reject_all_edits` restores it, or deletes a file the agent created (`crates/action_log/src/action_log.rs:120-147,712-795`) |
| Overwriting an existing file in the review | not covered | **Changed since the old checkout.** The action log now keeps the old text as the diff base for a created-over-existing file, so review shows a modification with removed lines; before, the base was empty and the whole file counted as added. A new test pins `+2 -3` (`action_log.rs:164-186`, `test_overwriting_file_counts_removed_lines`) |
| Format on save and whitespace | settings "apply", two tests pin it | They apply only when `format_on_save` is not `off` (`edit_session.rs:186-205`), and the shipped global default is `"off"` (`assets/settings/default.json:1653`). Trimming and the final newline live in the format pass (`crates/project/src/lsp_store.rs:1772-1792`) |
| Save failures | not covered | `project.save_buffer` and `project.format` end in `.log_err()` (`edit_session.rs:204,210`), so a failed save still returns a success |
| `content: ""` on an existing file | not covered | Nothing is written: `finalize_content("")` emits no event (`streaming_parser.rs:260-271`), so the old text stays and the model gets `No edits were made.` |
| "Resolves whether the buffer changed since the last read" | presented as a protection | The flag is computed and dropped for a write (`edit_session.rs:396-405`); order is resolve path, authorize, open buffer, dirty prompt, baseline |
| Path contract | stricter than Cursor's | The description is strict, the code accepts root-less relative paths when the parent exists, and a path outside every worktree is refused with `Can't create file: parent directory doesn't exist` before any approval (`edit_session.rs:1262-1287`). `~/.agents/skills` is the one exception |
| Encoding, line endings | "buffer's own" | Detected from a BOM, a null-byte heuristic or `chardetng`, kept on write, unmappable characters written as `&#N;`; the first newline in the first 1000 bytes decides the line ending, mixed and lone-CR files become uniform (`crates/file_content/src/file_content.rs`, `crates/text/src/text.rs:3646-3661`) |
| Existing-file decode | lived in `crates/language/src/file_content.rs` | Moved to the new `file_content` crate; a non-UTF-8 file now streams through `DecodingReader` instead of being re-read whole. Same error strings (`worktree.rs:7272-7311`) |
| Tool schema per provider | not covered | Removed. `tool.input_schema(model.tool_input_format())` plus `adapt_schema_to_format` became one `input_schema()` plus `normalize_tool_schema`, which deletes the root `$schema`, `title` and `description` (`thread.rs:4341`, `language_model_core/src/tool_schema.rs:17-33`) |
| ACP types | `schema::v1` | `schema::v2`; `update_fields` takes a closure; `Diff::finalized` lost its `cx` argument. No behavior change for `write_file` |

## What Runvil can support today

The crate is `runvil/crates/plugin-std`, six tools with Cursor's wire names and parameter schemas, each validating against a stored JSON schema and each answering `NOT_IMPLEMENTED`. No file logic exists. The VFS is `runvil/crates/runvil-internal/vfs`, and `Access` in `handle/operations.rs` is the only way to reach it.

The framing fact: **`Access::write` takes `&[u8]`.** There is no encoding, no BOM, no line-ending, and no mode parameter anywhere in the crate. `Access::read` returns `Vec<u8>` and `Access::read_string` is the only text-aware operation, and it is UTF-8-or-error. So every behaviour below that is about text rather than bytes is either rebuilt in the Pack or explicitly dropped.

### Coverage

| Behaviour | `Access` method | Status | What the gap needs |
|---|---|---|---|
| Read the existing content before writing | `read` | covered | returns `Vec<u8>`, which is what an encoder needs; nothing more |
| Read the existing content as text | `read_string` | partial | UTF-8 only, and returns `NotUtf8 { path, valid_up_to }` for anything else. A latin1 file cannot be read through it at all, so encoding detection has to start from `read` |
| Write the new bytes | `write` | covered | the one operation the whole tool needs |
| Detect the file does not exist | `read` returning `NotFound` | covered | `VfsError::NotFound { path }` is the new-file path |
| Create parent directories | `mkdir` with `recursive: true` | covered | `Access::mkdir(path, true)` is exactly `mkdir -p` |
| Refuse a directory target | `stat` plus `FileType::Directory` | covered | needs a `stat` before the write; the VFS would also return `IsADirectory` from `write` |
| Refuse a symlink target | `stat` plus `FileType::Symlink` | partial | `stat` reports `Symlink`, but "refuse to write *through* a symlink" also needs the parent chain checked, and `Access` has no "resolve canonical path" operation |
| Refuse a non-regular file | `stat` plus `FileType` | covered | `Fifo`, `Socket`, `CharDevice`, `BlockDevice` are all in the enum |
| Fingerprint the file before writing | `read` plus a hash in the Pack | partial | the VFS has no content-hash operation, so the Pack hashes the bytes it already read. That is fine for the read-then-write window and useless against an external writer in a way a VFS-level hash would not be |
| Re-check the fingerprint immediately before writing | `read` again | covered | sequential operations on one `Access` do not self-conflict: `Claims::conflicts` returns `other_clock > view.seen_clock(other)` for a same-scope claim, so a second `read` on a path this handle already read is admitted. Verified in `handle/claims.rs` at 104 to 115 and 148. A second `read` therefore works, at the cost of reading the file twice |
| Serialize two writes to one path | the claims system | partial | `Claims::claim_write` conflicts a concurrent write from another live scope, and from the same scope at a later epoch. That is a stronger guarantee than Cursor's lock for the case that matters (two agents), and weaker for one specific case: two parallel writes issued inside one turn from one handle advance the same clock, and `conflicts` for a same-scope claim asks `other_clock > view.seen_clock(other)`, so the ordering is decided by the claim clock rather than by a queue. Either way there is no held lock, so it does not span read-modify-write |
| Hold a lock across read, diff, and write | nothing | **missing** | `Access` claims per operation. A multi-operation transaction needs either an explicit claim handle or a Pack-level mutex keyed on the canonical path |
| Detect encoding from bytes | nothing | **missing** | a Pack-level module. See D2 |
| Decode with an arbitrary encoding | nothing | **missing** | a Pack dependency, `encoding_rs` is the obvious one |
| Preserve or emit a BOM | nothing | **missing** | bytes in, bytes out, done in the Pack |
| Classify line endings | nothing | **missing** | string work in the Pack |
| Merge mixed line endings | nothing | **missing** | string work in the Pack, and the hardest single function in the tool |
| Preserve the file mode | `stat().mode` for reading, nothing for writing | **missing** | `Stat.mode` is `Option<u32>` so the VFS can *report* a mode, but `Access::write` cannot *set* one. A new-file mode would need a new parameter or a `create_with_mode` |
| Compute a diff | nothing | **missing** | a Pack dependency, or skip it entirely for the model-facing result. Rig uses `similar::TextDiff` with a 100 ms timeout and 3 lines of context (`rig-tools/src/fs.rs:310-315`); past the deadline the diff is exact but less minimal |
| Replace through a temp file | `write` plus `rename` | partial | rig's `write_atomic` (`rig-tools/src/fs.rs:82-121`) is a 40-line Rust reference: refuse a non-regular or read-only target first, `File::create_new` a temp in the target's directory, copy the permissions, `write_all`, `sync_all`, `rename`. D3 notes that `Access::rename` claims the destination, so the claim ordering comes free |
| Guess an encoding, decode, encode | nothing | **missing** | Zed's `file_content` crate pairs `chardetng::EncodingDetector` with `encoding_rs` (`Encoding::for_bom`, `UTF_16LE`, `UTF_16BE`). `Encoding::encode` cannot emit UTF-16, so Zed writes it by hand in `encode_text`, and unmappable characters come out as `&#N;` (`file_content.rs:89-146`) |
| A content hash and a compare-and-swap | `read` plus a hash in the Pack | partial | rig's `ContentHash` is a `sha2::Sha256` hex string (`rig-core/src/tool/facts.rs:45-61`) and `Workspace::write_if(path, bytes, &held)` writes only while the file still hashes to `held`, else hands back the current content (`rig-tools/src/workspace.rs:106-118`). Its default is read, compare, write, with a window, so do it under the claim |
| Tell created from replaced | `read` before `write` | partial | rig's `Workspace::replace` returns the old content from the same operation (`workspace.rs:91-95`). `Access::write` could return the old size and hash from the claim it already takes, and the Pack builds the sentence |
| Refuse a write from a cut message | nothing | **missing** | not a Pack job, because the Pack never sees the stop reason. Rig does it in the kernel: `spawn_reply` answers a cut call with `invalid_args` before the tool exists (`rig-ecs/src/turn.rs:1412-1431`). Runvil's host needs the same hook before dispatch |
| Report the resolved path | nothing | **missing** | `Access` normalizes internally but does not return the normalized path. `glob` returns paths relative to the root for relative patterns, and `list` returns names, but there is no "resolve this path" call |
| Refuse a path outside the workspace | the policy, via `gate` | covered | `VfsError::OutsideRoot` and `PermissionDenied` already exist, and `ModePolicy` gates per operation, so this is handled before the Pack sees it |
| Plan-mode markdown allowlist | the policy, via `gate` | **partial** | plan mode is already a VFS policy, so the Pack must **not** re-implement it, but the predicate is narrower and stricter than Cursor's: `is_markdown` in `vfs/src/lib.rs` at line 161 is `path.as_str().ends_with(".md")`, case-sensitively and with no canvas regex. Cursor allows six suffixes (`.md`, `.markdown`, `.mdown`, `.mkd`, `.mkdn`, `.mdx`), matches them case-insensitively after a trim, and allows `.canvas.tsx` under `.cursor/projects/<x>/canvases/`. So a plan-mode write to `notes.markdown` or `A.MD` is allowed by Cursor and refused by Runvil, and the canvas exception does not exist at all |
| Detect a binary file | nothing | **missing** | a byte sniff in the Pack over the bytes `read` returned |

### The three that matter

**Plan mode is already solved in the right place, with the wrong predicate, and that is a finding rather than a gap.** Cursor implements the plan-mode gate inside the write wrapper, after the read and after the diff, which is why a refused write costs a full read. Runvil puts the gate in `ModePolicy`, at `gate`, which runs before anything else in every `Access` method. So the ordering problem disappears: a plan-mode write to a `.ts` file is refused before the read. That is strictly better, and the Pack should not implement a plan-mode check at all - doing so would duplicate a policy decision the VFS already owns and would reintroduce the ordering bug.

What does not carry over is the allowlist. `vfs/src/lib.rs` line 161:

```rust
fn is_markdown(path: &VfsPath) -> bool {
    path.as_str().ends_with(".md")
}
```

with a doc comment that makes the case-sensitivity deliberate:

```text
Plan mode's markdown rule: a `.md` suffix, case-sensitively - the
virtual namespace follows POSIX path rules strictly, so `.MD` is not
markdown and the case-insensitive suggestion does not apply.
```

That is a defensible rule for a virtual POSIX-strict namespace, and it is not Cursor's rule. Cursor allows six suffixes, trims and lowercases before matching, and allows managed canvas paths. The differences that matter:

| Path in plan mode | Cursor | Runvil today |
|---|---|---|
| `notes.md` | allowed | allowed |
| `notes.markdown` | allowed | **refused** |
| `notes.mdown`, `notes.mkd`, `notes.mkdn`, `notes.mdx` | allowed | **refused** |
| `NOTES.MD` | allowed (lowercased) | **refused** |
| ` notes.md ` (leading or trailing space) | allowed (trimmed) | **refused** |
| `.cursor/projects/x/canvases/y.canvas.tsx` | allowed | **refused** |
| `notes.MD` | allowed | refused, and deliberately |

If you want Cursor's plan-mode surface, the fix is in the policy, not the Pack: widen `is_markdown` to the six suffixes with a `to_ascii_lowercase` and a `trim`, and decide separately whether the canvas exception belongs. My advice is to widen the suffixes and skip the canvas regex, because the canvas exception exists to serve a Cursor feature you are not building, and this workspace bans canvas files anyway. Confidence medium on widening, high on skipping the canvas case.

**The missing lock is the real gap.** `Claims::claim_write` conflicts concurrent writes, which is the same guarantee Cursor's per-file lock gives, and better keyed. But a claim is per-operation, so the sequence read, diff, write is three claims with two gaps between them. Closing that needs either a claim that spans operations or a Pack-level mutex. Of the two, a Pack-level mutex keyed on the path is simpler and is what I would do first; a transaction handle in the VFS is the better long-term shape and follows the `gate`/`admit`/`fire`/delegate pattern the crate already uses.

**The resolved-path gap is small but it bites.** The tool's success message needs a path, and the structured result needs the canonical one. `Access` normalizes internally (`self.gate` returns a canonical path) but does not expose it. Either the Pack reconstructs it, which duplicates VFS logic and will drift, or the VFS grows a `resolve` that returns the canonical path without touching the backend. The second is the right answer and it is a small addition.

### What it takes to implement `Write` against this crate

Roughly, in order:

1. A `text` module in `plugin-std`: a format detector over the first 8192 bytes, a decode, an encode, a BOM policy, a line-ending classifier, and the merge. Pure functions over `&[u8]` and `&str`. No `Access`, no I/O, fully unit-testable. This is where all the difficulty is.
2. A `write` tool body: `stat` to classify the target, `read` to get the existing bytes, `read` again for the fingerprint if you want one, the text transform, `mkdir(recursive)`, `write`.
3. A decision about the lock. A `Mutex<HashMap<String, Arc<Mutex<()>>>>` keyed on the canonical path is ten lines and covers the in-process case.
4. A decision about the diff. If you want one in the structured result, that is a dependency and a threshold; if you do not, the tool gets simpler and the result is a sentence, which is what the model sees anyway.
5. Tests. The mixed-ending vectors, the BOM vectors, the empty-write vector, the UTF-16 round trip. All of them are string and byte tests that do not need a filesystem.
6. A host-level guard for a message that stopped on the output limit, outside the Pack (D9). The Pack should also never read the old file with a cap, so the truncated-view hole cannot open.

## What remains unknowable

Five things about Cursor itself, each with what it would take, then the limits of the harness comparison.

**Whether the installed daemon and the installed bundle are the same revision.** Every claim about the encoder comes from `daemon.cjs`, and every claim about the tool's call into it comes from `main.js`. They agree on every string I cross-checked. They are not provably the same build, and the bundle is the one that runs.

What would settle it: a live call against a UTF-16LE file with a BOM, checking the bytes that land. That is one test and it gives you a byte-exact fixture for the whole encoding section. It is the single highest-value probe left.

**Whether the daemon lags in the line-ending merge.** The merge I transcribed is from the daemon, and the bundle contains no readable copy of it - searching the minified bundle for `withExistingLineEndings`, `pairLines`, `splitLines`, `MAX_MATCH_CELLS`, or any of the distinctive fragments returns nothing, because they are minified past recognition. The bundle's call site at 8610800 does show `At({content:t,existing:u,defaultEnding:...})` and the branch structure, so the *shape* is confirmed in the shipped build. The internals are daemon-only.

What would settle it: a mixed-ending file written with LF-only content, checking the middle line's terminator. If it comes out as `mm\r\nnn\noo\r\n`, the merge is as documented. If it comes out uniform, either the merge changed or the backfill is not in the shipped build.

**Whether the `exceededLimit` read path reaches `Write`.** The mechanism is visible: the read returns `success` with `exceededLimit: true` and empty content, and `Write`'s switch takes the `success + content` branch with `old = ""`. What is not visible is whether the read executor's streaming path can produce `exceededLimit` at all for a full-file read, or whether `exceededLimit` is only reachable through the Read *tool's* post-processing. If it is only reachable in the tool, `Write`'s internal read never sees it and the concern is moot.

What would settle it: a 200 KB single-line file, written through `Write`, with the diff inspected. If the diff reports 200,000 characters removed, `exceededLimit` reached the tool. If it reports a normal line-level diff, it did not.

**What the workbench does with the structured result.** The bundle builds `EditSuccess` and hands it to a renderer. Whether the client recomputes the diff, whether it shows `beforeFullFileContent` when the elision dropped it, whether it reloads an open editor buffer - all of that is in a different bundle (`cursor-agent-exec` is the agent; the workbench is the extension host). Cursor's own forum reports of a stale buffer after an agent write suggest it does not always reload, but that is a forum report, not a measurement.

**Whether `pi_write`'s missing plan-mode gate is reachable.** The daemon shows `LocalPiWriteExecutor` calling the write executor directly, bypassing `performWrite` where the gate lives. Whether pi tools are offered in plan mode is a feature-flag question the bundle does not answer, because the flag `enablePiTools` gates registration and the mode gates offering, and the two are in different places.

What would settle it: a plan-mode session with `enablePiTools` on, a `pi_write` to a `.ts` file, and the result. If it writes, the asymmetry is a real hole; if it is refused upstream, the gate is elsewhere and the code path is fine.

One more honest limit, about this report rather than about Cursor: the probes I ran exercised a transcription of the algorithm, not Cursor itself. Where I write `(probe)` about line endings or mode, the probe measured my own reimplementation against the daemon's source, which is a test of the transcription's fidelity, not of Cursor's runtime. The genuine live probes are the ones the parent spec records, and they are marked `(probe)` where I cite them. That distinction is worth keeping in mind before treating any of my vectors as golden.

**Limits of the harness comparison.** What this pass could not settle for the other harnesses.

- Whether a server upstream of Cursor closes partial JSON when a message is cut by the output limit. Cursor's `Write` wrapper runs a strict `JSON.parse` on the buffered arguments and the loop retries a provider output-limit error, but neither shows what arrives when the cut was repaired before the client saw it. `(bundle)` `(daemon)`
- Zed: nothing was compiled or run. No test pins the rendered model text, and the empty-content, save-error and partial-failure behaviors are derived from reading the code. Whether editor undo reverses a whole write in one step, the serialized JSON schema, and Windows UNC and case handling in `find_project_path` were not traced.
- pi: the TypeBox text for a missing or mistyped argument, and what `Value.Convert` does to a number sent as `content`, are not in the sources, because the package is not installed here. Probes ran on win32, so mode, symlink following and case-sensitive key collisions were not probed, and neither was durable scheduling above its queue.
- rig: probes ran on Windows and the `Remote` paths are read from scripts. No test runs `write` over an existing file under `read_before_edit`, `file_lease` or `hunk_review`. What a cancelled write does is inferred, since `blocking` takes no cancel signal.
- zeroclaw: the tools were not built or run. The `create_dir_all`-before-boundary-check hazard is from reading order, and the Unix symlink tests need a privilege this machine lacks.
- Codex CLI: the `apply_patch` instruction text returned 404 at the pinned commit, so what the model is told about absolute paths is unknown. Patch size and hunk limits are not in the sources.
- Gemini CLI: the sandboxed file-system service and the context-management route that skips the 64 KiB cut were not read.
- OpenCode: three implementations coexist and which one the TUI drives is not in the sources, nor are the permission-rejection text and platform error tails.
- Cline: the legacy v4.0.12 handlers were not executed, and file mode on an existing file was not probed.
- Aider: the udiff apply path was not probed because `diff_match_patch` is not installed.
- Goose: the schemars schema was not generated, no test pins the success strings, and how a provider adapter formats the tool result was not traced.

## Conformance checklist

The twenty items from the parent spec's `### WriteFile` entry, each marked. An item marked `confirmed` holds in the shipped 3.27.12 build and in the daemon. `confirmed, with a warning` means the item as written is true and the description understates the work. An item marked `corrected` was wrong or incomplete and the correction is stated.

| # | Item | Verdict | Reason |
|---|---|---|---|
| 1 | The content field is `contents`, not `file_text` and not `content` | **confirmed** | `mke` at 4230054 and `pke` at 4229910 both use `contents`. `pi_write` uses `content`, as do pi's own `write`, Claude Code, Copilot, Zed, rig, zeroclaw, Gemini CLI, OpenCode and Goose, so `contents` is the odd one out of the set and `content` is the majority key |
| 2 | `contents: ""` passes schema validation because the field has no minimum length | **confirmed** | a bare `z.string()` with no `.min()`; the write path truncates then writes zero bytes |
| 3 | The lock is acquired during argument streaming, as soon as the path field completes | **confirmed** | `gbe` at 4154498; the check is `pathJustParsed && currentPath !== undefined` inside the `for await` over the chunk stream |
| 4 | A stream ending without the path field fails with `Failed to acquire file lock - path may not be parsed` | **corrected** | the shipped string is `Failed to acquire file lock - path may not have been parsed`. The words `have been` are missing from the spec, from its failure matrix, and from its conformance checklist. Both binaries agree: bundle at 4156465, daemon at 12538451. A second correction: the refusal runs after validation, so a missing `path` fails as `Invalid arguments:\npath: Required` and the lock string needs valid arguments, as the streaming section explains |
| 5 | The tool performs its own read and never consults a model-facing read record | **confirmed** | `i.execute(t, n, ...)` where `n` is a `ReadArgs` built from the same path and tool call id. No `hasReadPath` anywhere on the path |
| 6 | There is no read-before-write gate and no error string for one | **confirmed** | the only `hasReadPath` use in the bundle is the MCP tool gate; a never-read file was overwritten successfully |
| 7 | A not-found result from the internal read is the new-file path, not an error | **confirmed** | `case"fileNotFound":d=void 0` at 4231130, and `fileChangeType` becomes `added` |
| 8 | The diff uses `added` when there was no old content and `modified` otherwise | **confirmed** | `fileChangeType:void 0===d?"added":"modified"` at 4231955 |
| 9 | The plan-mode gate fires after the read and after the diff, so a refused write still costs a full read | **confirmed** | the gate is the first statement in `Rbe` at 4163000, and `Rbe` is called from the execute body after the read and after `Lbe` |
| 10 | The plan-mode allowlist is exactly `.md`, `.markdown`, `.mdown`, `.mkd`, `.mkdn`, `.mdx`, plus canvas paths | **confirmed** | `S_e` at 4140186 is exactly those six, and `k_e` at 4140241 is `/(?:^\|\/)\.cursor\/projects\/[^/]+\/canvases\/[^/]+\.canvas\.tsx$/i`. Both tests run through `x_e` at 4140313, which applies `trim()` and `toLowerCase()` for the suffix pass and `trim()` plus a backslash-to-slash replace for the canvas regex |
| 11 | A malformed `file://` URL is rejected while a well-formed one is normalized and written | **confirmed** | `isInvalidFileUrl` runs first in the executor and returns the error case; `resolvePath` converts a valid one |
| 12 | The parent directory is created recursively before the write | **confirmed** | `mkdir(dir, {recursive:true})` after the permission gate and after the directory-existence check |
| 13 | The stale fingerprint is re-verified immediately before the truncate, not before `mkdir` | **confirmed** | `fileChangedSinceRead` inside the `write2` closure, immediately before `writeFileNoFollow`. There is also an earlier check on the parent directory, before `mkdir`, which the spec mentions separately |
| 14 | The open flags include no-follow and close-on-exec, and a symlinked leaf is refused rather than followed | **confirmed** | `O_WRONLY\|O_CREAT\|O_NOFOLLOW\|O_CLOEXEC`, with ELOOP/EMLINK/EPERM mapped to `Refusing to write through a symlink at <path>` |
| 15 | The file mode is `0o666` before umask with no executable-bit preservation | **corrected** | the mode argument is `0o666` and there is no `chmod` anywhere in the path, so the second half is right for *new* files. But `open` applies the mode only on creation, so an existing `0755` file keeps `0755`. The spec's claim that "a write to a script that was `0755` will lose its executable bit" is wrong. Measured in Git Bash under umask 0022: existing `755` stays `755`, new file becomes `644` |
| 16 | The write is truncate-in-place followed by `fsync`, with no temp file and no rename | **confirmed** | `truncate(0)`, then a write loop, then `sync()` when `flush` is set, then close. No rename, no temp path. The containing directory is also fsynced, best effort, after the write |
| 17 | The success message is exactly `Wrote contents to <path>`, using the raw argument after the worktries repair | **confirmed** | `` const f=`Wrote contents to ${u}` `` at 4231955, with `u = lke(args.file_path ?? args.path)` |
| 18 | No diff, line count, or byte count reaches the model | **confirmed** | the renderer returns `JB(t.value.message ?? "", false)`, and `message` is the one sentence. `linesCreated` and `fileSize` exist in the executor's `WriteSuccess` and are never copied into `EditSuccess` |
| 19 | Format detection samples only the first 8192 bytes, and the binary extension shortcut reaches a verdict without reading | **confirmed** | `FORMAT_HEADER_BYTES = 8192` and `readFirstBytes` reads exactly that; `getFormatFromFileExtension` returns a binary verdict from the extension alone. One refinement: the extension shortcut runs when detection *threw*, and separately `getFormatForFile` consults the extension set after a successful-but-binary sniff |
| 20 | Incoming content that already contains `\r\n` is left untouched, a mixed-ending file gets a per-line merge, and a CRLF file has bare LFs rewritten | **confirmed**, with a warning | the three rules are as stated. The merge is more intricate than "a Myers-style line alignment": it has a prefix pass, a suffix pass, an LCS with a 4,000,000-cell cap, and a fourth backfill pass that fills unmatched gaps positionally. Without the backfill the spec's own probe case does not reproduce. Also, a content string with no trailing newline takes a different path through the merge than the same string with one |

Summary: seventeen confirmed, three corrected. The corrected ones are item 4 (a missing `have been`), item 15 (the executable-bit claim is wrong for existing files), and item 20 (the merge is not what the description implies, and the trailing-newline dependence is undocumented). Item 20 is marked confirmed-with-a-warning rather than corrected because the three rules as stated are right; it is the description of the merge that understates the work.

## Sources

### Bundle

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js`, 9,814,287 bytes, read as latin1, sliced by byte offset. `(bundle)`

| What | Offset in this build | Spec's offset |
|---|---|---|
| `AG("WRITE"`, dsv3-1018 branch | 4232349 | 4217017 |
| `AG("WRITE"`, default branch | 4233029 | 4217697 |
| `hke`, the factory | 4230322 | 4214990 |
| `pke`, default schema | 4229910 | 4214572 |
| `mke`, dsv3-1018 schema | 4230054 | 4214572 |
| `lke`, the worktries repair | 4228597 | ~4214600 |
| the execute closure's read switch | 4231080 | ~4215050 |
| `Wrote contents to` | 4231955 | - |
| `Rbe`, performWrite | 4163000 | 4147701 |
| `G_e`, the plan-mode throw | 4150095 | - |
| `z_e`, the plan-mode long form | 4150120 | - |
| `S_e`, the markdown suffix list | 4140186 | - |
| `k_e`, the canvas regex | 4140241 | - |
| `gbe`, the streaming wrapper | 4154498 | 4139201 |
| `Failed to acquire file lock` | 4156465 | - |
| `fbe`, the error serializer | 4153543 | - |
| `ybe`, the renderer | 4157903 | 4141861 |
| `vbe`, the success builder, and `wbe = 524288` | 4157903 | - |
| `At`, the mixed-ending merge call site | 8605364 | ~8465400 |
| `dr` / `getBufferForWrite`, the encoding call | 8603900 | ~8475048 |
| `BINARY_EXTENSIONS`, encoder side | 8604020 | - |
| `uC`, the write primitive | 9315126 | ~9177010 |
| `fC`, LocalWriteExecutor | 9317565 | ~9179400 |
| `fileChangedAfterReadMessage` | 9314404 | - |
| `Refusing to write through a symlink` | 9315500 | - |
| `Agent-store quota exceeded` | 9321864 | - |
| `lne`, the lock manager | 3455600 | 3440300 |
| `tJe`, the vendor toolset switch | 4581853 | 4566532 |
| `fMe` / `AG("PI_WRITE"` | 4560365 / 4560410 | 4560410 |
| `XRe`, the pi_write schema | 4554748 | - |
| `UNSUPPORTED_BINARY_EXTENSIONS`, Read tool side | 13569705 | - |
| `Cannot read binary files of type` | 13574854 | - |
| `READ_CHAR_HARD_LIMIT` | 12792688 | - |
| `Zk`, the binary-rejection factory | 9135900 | - |
| the read executor's directory and non-file rejections | 9146300 | - |
| `MAX_OFS_FILE_SIZE` / `MAX_BUFFER_SIZE` | - | - |
| `CHAR_HARD_LIMIT` | 8035034 | - |

The `hke` offset is derived rather than directly searched: it is where `hke=(e,t,r,n,s,o)=>{` begins, which is 4230322 by the `const i=e.get(X$._As),a=async` marker inside it.

### Daemon

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, unminified, original identifiers, embedded source paths. `(daemon)`

| What | Offset | Source path it claims |
|---|---|---|
| `createWriteTool` | 12629713 | `../agent/dist/tools/core/write/write.js` |
| the four description strings | 12633500 to 12636600 | same |
| `parametersSchema14` / `parametersSchema08192` | 12631500 | same |
| `performWrite` | 12515662 | `../agent/dist/tools/core/edit/write.js` |
| `assertPlanModeAllowsFileEdit` | 12507012 | same |
| `decoratePostWriteResultForModel` | 12494671 | same |
| `editTransformResultToEditResult` | 12509842 | same |
| `EDIT_FULL_FILE_CONTENT_MAX_CHARS` | 12539479 | same |
| `isPlanModeAllowedEditPath` | 12489150 | `../agent/dist/tools/core/edit/plan-mode-file-policy.js` |
| `withStreamingEditParsedArgs` | 12536500 | `../agent/dist/tools/core/edit/streaming.js` |
| `DIFF_SIZE_THRESHOLD` / `serializeEditError` | 12534208 | `../agent/dist/tools/core/edit/common.js` |
| `LocalWriteExecutor` | 16998591 | `../local-exec/dist/write.js` |
| `writeFileNoFollow` | 16995900 | `../local-exec/dist/symlink-safe-write.js` |
| `ToolCallReadStore` | 16992300 | `../local-exec/dist/tool-call-reads.js` |
| `fileChangedSinceRead` | 16988207 | same |
| `fingerprintFile` | 16988590 | same |
| `fileChangedAfterReadMessage` | 16989757 | same |
| `getBufferForWrite` | 15433367 | `../utils/dist/encoding.js` |
| `getFormatForBuffer` | 15437700 | same |
| `detectUTF16Encoding` | 15445579 | same |
| `detectUTF32Encoding` | 15446621 | same |
| `isBufferText` | 15441812 | same |
| `lineEndingFromCounts` | 15441114 | same |
| `decodeBinaryContent` | 15447447 | same |
| `getDefaultTextFormatForOS` | 15443978 | same |
| `BINARY_EXTENSIONS` | 15450802 | same |
| `FORMAT_HEADER_BYTES`, `MAX_FILE_ENCODINGS`, BOM tables | 15449367 | same |
| `hasMixedLineEndings` | 15426249 | `../utils/dist/line-endings.js` |
| `splitLines` | 15426493 | same |
| `pairLines` | 15427200 | same |
| `withExistingLineEndings` | 15428106 | same |
| `readMixedLineEndingText` | 15436480 | `../utils/dist/encoding.js` |
| `encodeUtf8RestoringUndecodableBytes` | 15436236 | `../utils/dist/lossy-utf8.js` |
| `MAX_TEXT_SIZE` | 18017 | `../constants/dist/composer.js` |
| `MAX_OFS_FILE_SIZE` / `MAX_BUFFER_SIZE` | 14536957 | `../local-exec/dist/constants.js` |
| `LocalPiWriteExecutor` | 16866855 | `../local-exec/dist/pi/write.js` |
| `withPiFileMutationQueue` / `queueKey` | 16837816 / 16837707 | `../local-exec/dist/pi/mutation-queue.js` |
| `resolvePiPath` | 16838486 | `../local-exec/dist/path.js` |
| `PiWriteExecArgs` / `PiWriteExecResult` protos | 2279364 | `../proto/dist/generated/agent/v1/pi_write_exec_pb.js` |
| `PiWriteToolCall` / `PiWriteToolArgs` protos | 6081337 | `../proto/dist/generated/agent/v1/agent_pb.js` |
| `LocalReadExecutor`, `readTextStreaming` | 16761324 / 16750056 | `../local-exec/dist/read.js` |
| `KNOWN_BINARY_MAGIC_PREFIXES` | 16739618 | same |
| `normalizeLineEndings2` | 12523759 | `../agent/dist/tools/core/read/*` |
| readonly-subagent stub text | 1739012 | `../agent/dist/tools/*` |

### Working extracts

Slices of the 3.27.12 bundle and daemon made for the first pass: `WRITE.js` (the write tool slice, which failed to prettify because the slice starts mid-expression), the `write_file` schema dump, the encoding module, the shared formatter, and the briefing behind this report. `(bundle)` `(daemon)` The offsets and probes were produced by scripts that slice the bundle and daemon by byte offset: `derive.js` (offsets), `sl.js` and `dsl.js` (slicers), `find.js` (multi-needle search), `show.js`, `merge-model.js` and `merge-model-lib.js` (a literal transcription of the four merge phases), `merge-edge.js` and `merge-edge2.js` (vectors), `mode-probe.js` and `mode-posix2.sh` (the mode measurement). `(probe)`

### Harnesses

- Claude Code 2.1.311, `claude.exe` decoded as latin1: the `write.js`, `read-file-state.js`, `freshness-checks.js`, `limits.js`, `input-schemas.md` and `error-strings.json` extracts. `(source)`
- VS Code Copilot: `https://github.com/microsoft/vscode` at `b208bc2501` (2026-10-25), `extensions/copilot/src/extension/tools/node/createFileTool.tsx`, `applyPatchTool.tsx`, `editFileToolUtils.tsx`, `common/toolNames.ts`, and `util/common/markdown.ts`. `(source)`
- Zed: `https://github.com/zed-industries/zed` at `37faef81f7` (2026-10-27): `crates/agent/src/tools/write_file_tool.rs`, `edit_session.rs`, `tool_permissions.rs`, `edit_session/streaming_parser.rs`, `crates/action_log/src/action_log.rs`, `crates/file_content/src/file_content.rs`, `crates/fs/src/fs.rs`, `assets/settings/default.json`. The first pass used `d9ad6aff67` (2026-08-22). `(source)` and, for `Encoding::encode`, `(web)` https://docs.rs/encoding_rs/0.8.35/encoding_rs/struct.Encoding.html
- pi: `git@github.com:earendil-works/pi.git` at `a92283056` (2026-10-27), `@earendil-works/pi-coding-agent` 1.4.0, with commits `e583b290a` (2026-09-02, byte count removed), `351efc828` (2026-07-07, length-truncated messages) and `74a46fc7e`, `d38ad0cd6`, `e9146a5ff` (queue history). Probes ran the real `write.ts` and queues on win32 with Node v24.19.0. `(source)` `(probe)`
- rig: `https://github.com/0xPlaygrounds/rig` at `a83b774d0` (2026-10-25): `crates/rig-tools/src/write.rs`, `fs.rs`, `workspace.rs`, `read.rs`, `crates/rig-ecs/src/turn.rs`, `crates/rig-core/src/tool/feedback.rs`, `facts.rs`, and the `corpus` plugins. Probes were compiled copies of the function bodies with `rustc` 1.99.1 on Windows. `(source)` `(probe)`
- zeroclaw: `https://github.com/zeroclaw-labs/zeroclaw` at `2d85371f3` (2026-09-18): `crates/zeroclaw-tools/src/file_write.rs`, `wrappers.rs`, `policy.rs`. `(source)`
- Codex CLI: `https://github.com/openai/codex` at `bea47c06dfac12ecce362090418e12a6faeedbf9`: `codex-rs/apply-patch`, `codex-rs/core/src/tools/handlers/apply_patch_spec.rs`, `core/tests/suite/apply_patch_cli.rs`. `(source)` `(web)` for the commit metadata.
- Gemini CLI: `https://github.com/google-gemini/gemini-cli` tag `v0.66.0`, commit `970413207732f5d4d8c50540c7fc359c44fdaaf2`: `packages/core/src/tools/write-file.ts`, `omissionPlaceholderDetector.ts`, `services/fileSystemService.ts`, `utils/textUtils.ts`. The write-file, text-utils and placeholder-detector files were re-fetched from raw.githubusercontent.com at that commit for this pass. `(source)` `(web)`
- OpenCode: `https://github.com/anomalyco/opencode` tag `v1.19.5`, commit `810c0eba1a7348036f8bdf61e6632d0c36c6876c`, and dev `fa1260d6ca`. `(source)` `(web)` https://opencode.ai/docs/tools/
- Cline: `https://github.com/cline/cline` tag `v4.1.26` (`a459f1d`) and legacy `v4.0.12` (`f267bf4`). `(source)` `(probe)`
- Aider: `https://github.com/Aider-AI/aider` at `5dc9490bb35f9729ef2c95d00a19ccd30c26339c`. `(source)` `(probe)`
- Goose: `https://github.com/aaif-goose/goose` at `ee8500d5e060eea6f2bcee92e8a5e351f12c3a70`, release tag `v1.56.0` (`9949cf9`). `(source)` `(probe)`

### Parent specification

`C:\Users\Vinnie\cursor\runvil-design\specs\cursor\agent-file-tools.md`: the `## WriteFile` section at 1019 to 1428, the conformance entry at 4711 to 4733, `## Shared infrastructure` at 319, `## Design shape` at 290, `## Failure matrix` at 4236, `## Build order` at 4387, the errata at 4584, 4614, and 4646, and `## What this document cannot tell you` at 4492. `(spec)`

The three errata items that touch this tool are all reflected above: the pseudocode note that the plan-mode refusal has two strings, the note that the failure matrix's plan-mode row should carry the renderer substitution, and the note that the worktree-escape check's ordering relative to the permission gate is not settled by the spec. On that last one: the shipped daemon puts the worktree check at step 18 and the permission gate at step 21, so the worktree check runs first. That resolves the spec's open question.

### Runvil

`C:\Users\Vinnie\cursor\runvil\crates\runvil-internal\vfs\src\handle\operations.rs`, `error.rs`, `stat.rs`; `C:\Users\Vinnie\cursor\runvil\crates\plugin-std\src\lib.rs`, `spec.rs`, `pack.rs`, and `schemas\write.json` with `schemas\EDITS.md`. `(source)`

### Web

Two sources, both used only to corroborate or to identify a divergence, never as the anchor for a claim:

- The `CL4R1T4S` leak of the Cursor 2.0 system prompt, and mirrors of it on `leaked-system-prompts.com` and `EliFuzz/awesome-system-prompts`, dated 2025-10-29 and 2025-11-01. Carries the `dsv3-1018` write description plus three extra trailing bullets that are not in any branch of the shipped bundle. The bundle wins.
- A Cursor forum thread titled "Agent Write tool inserts blank line between every line in file contents", describing doubled newlines in Go files on Windows. Consistent with the line-ending rules as documented; the bundle wins on mechanism, the thread is only evidence that the logic is sharp-edged in practice. `(web)`

---

*2026-10-27 - deepseek-v41-flash-1017*
