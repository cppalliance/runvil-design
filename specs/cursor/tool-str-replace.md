# Cursor's `StrReplace` file tool: an implementation report

Cursor's edit tool is a bare exact-substring replacement with one narrow fallback. What matters about it is the order the checks run and what the fallback does not do. It has no fuzzy matching, no read gate, and no freshness token, and yet it is the tool that most models have been trained on, which is why its surface is worth copying even where its internals are worth replacing.

The section that matters most is `## Decisions you have to make`, because Runvil already has `Access::str_replace` and the question is not how to match a string but where the semantics should live.

## The short version

- **There is no fuzzy matching.** No Levenshtein, no similarity threshold, no block anchor, no `...` elision, no smart-quote folding, no NFC. The only fallback is a line-trimmed block match that requires strict one-to-one line correspondence. Any implementation that adds a similarity score is a different tool with different failure modes.
- **The pipeline order is exact scan first, line-trimmed block match second, and nothing third.** The fallback runs only when the exact scan produced zero matches, so a file with two exact matches never reaches the fallback even if the fallback would have matched something else.
- **The exact scan advances by the needle's length, not by one.** So `aa` in `aaa` is one match. With `replace_all: false` that single match trips a separate overlap guard and throws; with `replace_all: true` it replaces once and the file becomes `Xa`; an overlapping replace would give `XX`.
- **The overlap guard is a second check that only exists because of the first.** It fires when `replace_all` is false, the match came from the exact scan, and `indexOf(oldString, firstMatchStart + 1)` still finds something. That is what catches self-overlapping needles that the non-overlapping scan undercounts. Other harnesses mostly lack it: Claude Code, Copilot, pi, rig, zeroclaw, Gemini CLI, Cline and Goose count the same way and pass `aa` in `aaa` as unique, editing the first site. Only Zed (overlap-counting scan) and OpenCode (`indexOf` against `lastIndexOf` on the candidate text) refuse it (source, probe).
- **`old_string === ""` throws before any search**, and normalized-equal old and new throws too. So `"foo"` to `"foo"` is an error, and so is `"a\r\nb"` to `"a\nb"`, because both normalize to the same text. But `"  foo"` to `"foo"` is a successful de-indent through the fallback, and `"  foo"` to `"    foo"` is a successful re-indent. The identity check compares line-ending-normalized forms and never trims.
- **The fallback trims per-line leading and trailing whitespace and does nothing else.** No internal run collapsing and no blank-line tolerance, so `"foo  bar"` does not match `"foo bar"`. The single-line rule is the non-obvious part, and it was run against the shipped engine (daemon). The exact scan goes first, so `"const a = 1;"` edits `const a = 1; // note` directly as a plain substring, with no fallback. When a one-line needle does need the fallback (a tab where the file has spaces), a trimmed file line that equals it or starts with it matches, and the region runs from column 0 of that line to the end of the needle, so a trailing comment survives and the indentation is replaced. A line that only ends with the needle, or holds it in the middle, is refused (daemon, probe).
- **When the fallback succeeds, `new_string` goes in verbatim, with no re-indentation.** Cursor does not consult the file's indentation and does not adjust the replacement. A model that emits a tab-indented replacement into a space-indented file gets a tab-indented region in a space-indented file.
- **Untouched lines keep their exact bytes, because nothing is ever rebuilt.** The matcher computes character offsets into the original content and the splice touches only `[start, end)`. An implementation that splits into trimmed lines, rejoins, and writes that back re-indents the entire file, which is the worst failure available in this tool.
- **CRLF is normalized in three places, and an LF `old_string` against a CRLF file is an exact match, not a fallback match.** Model strings get `\r\n?` collapsed to `\n`; file content gets `\r\n` collapsed to `\n` before the engine sees it; the write layer restores the file's ending on the way out. The model never has to know about CRLF.
- **The splice has two paths.** A bulk descending-offset splice when the replacement is non-empty and there are two or more matches, and a sequential loop otherwise, including every deletion. The deletion path eats the surrounding newline when the match was alone on its line, which is why a whole-line delete does not leave a blank line behind.
- **There is no read-before-write gate and no freshness check.** No error string exists for a gate; the requirement is prompt text only. `StrReplace` reads fresh and edits fresh, so a stale edit fails only if the anchor no longer exists or is no longer unique, and nothing ties the edit to the bytes the model read. Rig's compare-and-swap on a content hash is the Rust example of the missing piece (see decision 4).
- **The success text is one flat sentence.** No snippet, no line numbers, no diff stat, no count unless `replace_all` was set. `The file <path> has been updated.` or `N occurrences in file <path> have been updated.` That is the whole model-visible result.
- **The failure text includes a hint that costs nothing and is worth keeping.** On zero matches, Cursor picks the longest line of the failed `old_string`, finds the first file line containing it after trimming, and shows a numbered window around it under ` Found a possible fuzzy match, did you mean:`. It never affects whether the edit applies. A hint heuristic can be wrong in ways that cost a turn: Goose's `first_line.contains(line.trim())` is true for every blank line, so its hint is often an unrelated blank-line neighbourhood. Cursor's builder is not open to that (its anchor is non-empty), but its second pass accepts any line containing any word of three or more characters (daemon, probe).
- **The shipped build gates `StrReplace` by model vendor.** Anthropic, Google, xAI, and unknown vendors get `StrReplace` plus `Write`; OpenAI and Cursor vendors with `isGpt5Family` get `ApplyPatch` *instead of* `StrReplace`. A harness that always offers `StrReplace` is not reproducing this build.
- **`pi_edit` is the parallel surface and it is a different, in places better, design.** It takes an array of `{oldText, newText}` edits, applies them all against one snapshot of the original content, is all-or-nothing, and has both an exact path and a whole-file NFKC normalization path. That last path rewrites every untouched line, which is the trap described above. It is a version of pi's own edit tool that pi outgrew: pi wrote the whole normalized file back until commit `128330e36` (2026-06-19), then confined the write to the touched lines, and Cursor's port never picked that fix up (source, probe).

## Decisions you have to make

These are ordered by how much they change the shape of the crate.

### 1. Does `StrReplace` own its matching, or does it call `Access::str_replace` and layer the fallback above it?

This is the open question already recorded in `vibe/2026-10-27-2-plugin-std-file-tools.md`, and it is the one that decides the file layout.

The facts. `Access::str_replace` already implements exact single-occurrence replacement. It refuses an empty `old` before touching the backend, it claims a write, it fires the event, and it delegates to `self.inner().str_replace(&path, old, new)`, which answers `VfsError::Anchor { path, anchor, count }` when the count is not one. What is missing above it is `replace_all`, the trimmed-line fallback, and Cursor's exact error strings.

**Option A: the Pack reads, matches, and writes.**

The Pack calls `Access::read_string`, runs the whole pipeline over the returned text, and calls `Access::write` with the result. `Access::str_replace` becomes unused by this tool.

- What it costs: the read-modify-write window opens up. Between `read_string` and `write` another access can interleave, because the two calls claim separately. You also lose the VFS's own anchor semantics, so a backend that implements `str_replace` natively (a database, an object store, a document service) cannot be used. And the whole edit becomes a full-file write from the backend's point of view, so a backend that does line-level mutation cannot help.
- What it provides: one place for the semantics. The entire pipeline is a pure function over `&str` to `&str`, so the whole thing is testable with no filesystem at all, as Cursor's own `Bbe` is. Matching and splicing live in one module and one test file. Rig ships this split in Rust: `rig_tools::edit::apply(path, text, edits)` is public and pure, `edit()` is the thin read-apply-write wrapper, and a dry-run plugin and a benchmark call `apply` with no file system (source: rig `crates/rig-tools/src/edit.rs`, commit `a83b774d0`).
- What breaks: nothing structural, but `Access::str_replace` is now dead weight in the crate. Leaving a dead method that answers different error text than the tool is a trap for the next person.

**Option B: the Pack calls `Access::str_replace` for the exact single-occurrence case and does its own work for everything else.**

The Pack computes the match itself, then dispatches: if there is one exact match and `replace_all` is false, call `Access::str_replace`; otherwise read, splice, and `Access::write`.

- What it costs: two code paths that must agree. The Pack has to reproduce Cursor's match counting to decide which path it is on, which means it already knows everything `Access::str_replace` knows, which makes the delegation redundant. And the two paths produce different error text for the same user-visible condition (`VfsError::Anchor` versus Cursor's sentence), so the mapping layer has to translate.
- What it provides: the backend's native anchor path stays alive for the common case, and the claim is taken once by the VFS for that call.
- What breaks: the claim granularity becomes inconsistent. One branch claims once inside the VFS, the other claims twice from the Pack. Two calls to `StrReplace` on one path can interleave differently depending on which branch they take.

**Option C: matching lives in the VFS, as a richer anchor operation.**

Extend the VFS with a match policy - exact-unique, exact-all, trimmed-block - and let the backend do the work, returning the edited text or the failure.

- What it costs: the VFS grows a text-editing policy interface, which is a large conceptual addition to a layer that currently speaks in paths and bytes. Every backend must implement it or inherit a default.
- What it provides: the read-modify-write window closes, because the whole operation is one claim inside the VFS.
- What breaks: portability of the VFS. A backend that stores opaque bytes now has to know what a line is.

**Recommendation: Option A, the Pack owns the matching, with `Access::str_replace` retired or redefined.** Confidence: high, because the tool's semantics are text-level and belong in the layer that works in text, and because the read-modify-write window is already the status quo for `Write` and closing it is a separate, orthogonal piece of work. Do not leave `Access::str_replace` in place answering `Anchor` if the tool no longer uses it; either delete it or make it the primitive the Pack calls for one case only.

If you take Option A, take the second half seriously: make the pipeline a pure function, `fn apply(content: &str, edit: &Edit) -> Result<EditOutcome, EditError>`, with no I/O and no awareness of `Access`. Then the only thing the Pack does is read, call it, write, and map the error. That is the same split Cursor has between `Sbe` (read, gate, write) and `Bbe` (the pure engine), and it is the reason Cursor's engine can be reasoned about at all.

### 2. Do you keep the line-trimmed fallback at all?

**Option A: keep it as Cursor has it.** Per-line `trim()` on both sides, strict one-to-one line correspondence, partial anchoring on the first and last line only, single-line searches must match a whole trimmed line.

- What it costs: a class of match the model did not literally ask for. `"  foo"` can match `"        foo"`. Two different regions can become ambiguous only after trimming, and the model gets an "after relaxing whitespace" error it did not expect.
- What it provides: it solves tab-versus-space and U+00A0 at line edges for free, because both vanish under `trim()`. These are the two failure classes Claude Code has open issues for and Cursor does not. It is the single highest-value behavior in the tool after the exact match.
- What breaks: nothing, if the splice is offset-based. It breaks badly if the splice is line-based.

**Option B: drop the fallback, exact match only.** This is Claude Code's design.

- What it costs: every tab-indented needle against a space-indented file fails. Every file with non-breaking spaces fails at those lines.
- What it provides: a successful match is the text the model wrote, byte for byte. There is no class of edit that "sort of" matched.
- What breaks: models will retry, and each retry costs a turn.

**Option C: keep a fallback but make it a normalizing one.** Collapse internal whitespace runs, treat blank lines as optional, maybe normalize Unicode.

- What it costs: it changes bytes the model did not name. `"foo  bar"` matching `"foo bar"` is a silent rewrite of a string that might matter inside a literal.
- What it provides: fewer retries on badly-copied context.
- What breaks: the guarantee that a match means the text is what the model thinks it is.

**Option D: exact only, plus one uniform indentation repair (Aider).** If no exact match, Aider strips the common minimum indent from both SEARCH and REPLACE, looks for a window whose lines match after `lstrip()` and share one identical indent prefix, and adds that prefix to every non-blank REPLACE line (source: Aider `coders/editblock_coder.py:243-293`, commit `5dc9490bb`).

- What it costs: it changes the model's bytes too, because REPLACE is re-indented, and it is stricter than Cursor's per-line `trim()`: every non-blank line must share one offset, trailing whitespace is not tolerated, and the SEARCH `a` / `    b` against `        a` / `    b` fails (probe).
- What it provides: the file's indentation wins over the model's, which is what Cursor's fallback gets backwards (it replaces the file's leading whitespace with the model's). It is the one whitespace tolerance Aider's maintainer calls critical (web: Aider issue 306).
- What breaks: it is the same class as Trap 4. Use it only with a changed-region echo.

**Option E: a graded ladder that labels each tier.** Codex's `apply_patch` seeks four passes (exact, trailing-trimmed, both-ends trimmed, then Unicode-folded) and takes the first position (source: Codex `apply-patch/src/seek_sequence.rs:35-108`). Gemini CLI tries exact, flexible (trimmed lines, re-indented), regex, then bounded fuzzy and labels only the last one. OpenCode's dev branch tries exact, a one-to-one Unicode fold, then trailing whitespace, and keeps offsets so the splice lands on the original bytes (source: OpenCode `packages/core/src/tool/plugin/edit.ts:46-56,164-188`, dev `fa1260d6ca`). The costs are the ones in Option C plus a first-hit-wins rule in Codex and Gemini: Codex's probe edits the first of two identical lines with no warning.

**Recommendation: Option A, keep Cursor's fallback byte for byte.** Confidence: high, because it is the conservative choice that still fixes two failure modes, and because most attempts to make it cleverer ended badly. Aider's `difflib.SequenceMatcher` 0.8 matcher sits behind a bare `return` (disabled). Amazon Q proposed a 0.6 threshold (web). OpenCode's nine matchers (seven fuzzy) are still live at v1.19.5 but needed their thresholds tightened to 0.65 and a disproportion guard added, and its chain still edits the first candidate that looks unique without saying so (probe). What moved it: the other harnesses show the one safe middle design is Aider's uniform offset or OpenCode dev's offset-preserving folds, and neither beats Cursor's two-sided trim enough to justify a second rule. The one thing worth considering is a *flag*, not a different default, so a Host that wants strictness can turn it off.

### 3. Do you copy `replace_all`'s non-overlapping semantics, or use a real global replace?

Cursor's `replace_all` replaces the non-overlapping matches the same scanner found, so `aa` in `aaa` becomes `Xa` and reports one replacement. That example does not separate Cursor from `String.prototype.replaceAll`, which also gives `Xa` (probe). What does separate them is the replacement text: Cursor splices by offset, so `new_string: "$&$&"` lands literally, while `replaceAll` with a string replacement expands `$&`, `$$`, and the other `$` patterns. Run through Cursor's engine, `price: X and X` with `X` to `$&$&` gives `price: $&$& and $&$&`; native `replaceAll` gives `price: XX and XX` (daemon, probe). OpenCode's `content.replaceAll(search, newString)` has that bug, and Cline wraps its replacement in a closure to avoid it (source: OpenCode `tool/edit.ts:715`, Cline `editor.ts:187`).

**Option A: copy it.** `replace_all` means "all the matches this tool found", which is all the non-overlapping ones.

- What it costs: a model that says `replace_all: true` on a self-overlapping needle gets fewer replacements than it asked for, silently. The success message says `1 occurrence ... has been updated`, which is technically accurate and practically confusing.
- What it provides: one scanner, one notion of a match, and the count in the message always equals the number of splices. The overlap guard and the count are consistent with each other.
- What breaks: nothing internally. It surprises anyone who assumes `replace_all` means `replaceAll`.

**Option B: use an overlapping global replace.** Advance by one during the scan when `replace_all` is set.

- What it costs: two scanners, or one scanner with a mode. The count and the overlap guard have to be rethought, because the "more than one place and they overlap" error becomes unreachable under `replace_all`.
- What it provides: `aa` in `aaa` becomes `XX`, which is what a naive reader expects. No library `replaceAll` or `str::replace` does this, so it means a custom scan.
- What breaks: it is a different tool. Models trained on Cursor expect `Xa`.

**What the other harnesses do with the same question.** Uniqueness is where they split.

| Harness | How uniqueness is counted | `aa` in `aaa`, one edit |
|---|---|---|
| Cursor | non-overlapping scan plus the overlap guard | refused with the overlap sentence |
| Zed | overlap-counting KMP scan, stops at two hits | refused as ambiguous (`aaaa` in `aaaaa`) |
| OpenCode | `indexOf` equals `lastIndexOf` on the candidate text | refused as multiple matches |
| zeroclaw | `content.matches(old).count()`, then `replacen(old, new, 1)` | passes as exactly once, edits the first two bytes, `Xa` |
| pi, rig, Gemini CLI, Cline, Goose | `split`, `match_indices` or `matches`, all non-overlapping | pass, `Xa` |
| Claude Code, Copilot | non-overlapping scan, no guard | pass, first site edited |
| Aider, Codex | no uniqueness rule at all, first match wins | pass, first site edited |

(source, probe: zeroclaw `file_edit.rs:219`; OpenCode `edit.ts:705-720`; Zed `streaming_fuzzy_matcher.rs:315-364`; Aider `editblock_coder.py:146-154`; Codex `seek_sequence.rs`.) Zeroclaw is the clean counterexample: its uniqueness proof is a library count that skips overlaps, so the check passes and the edit lands on a site the model may not have meant. Aider adds the other extreme: no `replace_all`, no ambiguity error, and its udiff path replaces every occurrence (source: `coders/search_replace.py:443`).

**Recommendation: Option A, copy the non-overlapping behavior, and make the overlap guard mandatory.** Confidence: high, because the overlap guard is what makes the semantics safe: without it, `replace_all: false` on `aa` in `aaa` would silently edit one of two overlapping sites. The guard is the compensating control for the undercount, and the two must ship together or neither is correct. What moved it: ten of the twelve other harnesses lack the control (two of those, Aider and Codex, have no uniqueness rule at all), so the mistake is easy to make, and only Zed and OpenCode refuse the case.

### 4. Do you add a read-before-write gate?

Cursor has none. Claude Code has a full state machine.

**Option A: no gate, like Cursor.** The tool reads fresh and edits fresh.

- What it costs: a model can edit a file it never read, and will sometimes edit the wrong occurrence because it guessed the anchor.
- What it provides: no state to keep, no false refusals when the harness restarted or the context was compacted, and no way for the gate itself to be the bug.
- What breaks: nothing mechanically.

**Option B: a hard gate, like Claude Code.** Refuse the edit unless this session has read the file.

- What it costs: a state store keyed by normalized path, invalidated on every write, and an error string the model must learn. Partial reads complicate it: Claude Code treats a partial read as fatal for a full-file write and survivable for an edit.
- What it provides: it catches the case where a formatter rewrote the file between read and edit, which a fresh read alone cannot.
- What breaks: agents that legitimately edit a file named in the prompt without reading it, which is a common and reasonable pattern.

**Option C: no gate, but return the changed region in the result.** Neither Cursor nor Claude Code uses this design, and it makes the model's next turn correct without a round trip.

- What it costs: result size, and a decision about how much context to include.
- What it provides: the model sees what it just did, in file coordinates, without spending a Read.
- What breaks: nothing, but it is a departure from Cursor's flat sentence, so a model trained on `The file X has been updated.` sees a different shape.

**Option D: compare-and-swap on a content hash, re-applying on a mismatch (rig).** Rig records a SHA-256 of what `read` showed in a shared `Seen` map. `edit` applies its entries to the cached text and calls `write_if(path, bytes, &hash)`, which returns `None` when the file still held the cached text and `Some(current)` when it did not. On `Some`, rig re-applies the same entries to the current text and writes unconditionally. A miss against the changed text says `changed since it was last read, so read it again`; a hit says nothing (source: rig `rt/edit.rs:175-214`, `rt/workspace.rs:97-118`, commit `a83b774d0`).

- What it costs: a hash map, and a second window. Rig's probe shows an outside write landing between `write_if`'s read and its write is lost, on the fast path and on the slow path (probe S7, S8), because the compare and the write are separate steps. A file the model only partly read has a hash but no kept text, so its first edit is unconditional.
- What it provides: the common stale case (an outside append, the anchor still unique) succeeds with no extra read, because `write_if` hands back the content it already read (probe S2).
- What breaks: a successful edit on a changed file is silent. The model learns nothing unless the anchor is gone.

**Option E: diagnose, never gate (Zed).** Zed records the disk mtime at each read and each agent edit, compares it only when an edit fails to match, and adds one sentence to the error (`The file has changed on disk since you last read it.`). An edit whose `old_text` still matches succeeds after an external change, and an edit with no prior read succeeds (source: Zed `edit_session.rs:971-976`, tests `edit_file_tool.rs:302-326, 2322-2402`, commit `37faef81f7`).

- What it costs: one map from path to `SystemTime` or hash, and one string.
- What it provides: no turn lost on a harmless stale read, and the right explanation when the match fails. Rig applies the same threshold and says it only on a miss (probe S3), so two independent designs converged on it.
- What breaks: the edit that lands on changed text stays unreported, same as rig.

**What the other harnesses mean for this decision.** None of the other twelve has a hard gate except Claude Code (see above), and a design with no check loses a concurrent change. Zeroclaw, Goose and Aider read and write with no lock, hash or mtime between them (source). Cline's two parallel edits of one file both succeeded and one update was lost (probe). Pi and Gemini CLI serialize writes to one path in-process, with a queue and a lock, but keep no hash or mtime (source). Codex re-derives the new contents from disk at apply time, so a file edited during the approval wait is re-matched rather than overwritten from a stale copy (source: `runtimes/apply_patch.rs:183-201`). OpenCode's release build holds its lock across the permission prompt and writes without re-reading, while its V2 core uses `writeIfUnchanged` and the message `File changed after permission approval. Read it again before editing.` (source: `C:edit.ts:113-116,192`). Gemini CLI has a SHA-256 compare only inside its self-correction pass, which is off by default.

**Recommendation: Option A, plus a cheap version of C, plus Option D's compare-and-swap under the VFS claim, plus Option E's sentence on a miss.** Confidence: medium, because the gate's value depends on whether your harness has other agents writing the same files. If it does not, the gate is pure friction. The freshness problem is better solved by making the write compare-and-swap on a content hash, which is a VFS concern rather than a tool concern. Rig's probe is the warning: its compare and write are separate steps, so its window is narrowed but not closed. Runvil's `admit(Claims::claim_write, ...)` can hold one claim across the read, the hash compare and the write inside one process, which closes that window. Make `write_if` return the current contents on a mismatch, as rig does, so the tool can re-apply instead of forcing a second read. Returning the changed region is worth prototyping because Cursor's flat sentence is the weakest part of its contract; rig's reply (`Edited {path}: lines 2-4 (+1 line)` plus the changed lines with 3 lines of context) is the worked example. What moved it: rig's compare-and-swap and Zed's diagnose-on-miss are two working designs that drop the hard gate without losing the stale case.

### 5. Do you keep Cursor's success text, or return the changed region?

**Option A: keep Cursor's exact sentence.** `The file <path> has been updated.`

- What it costs: the model has no confirmation of what changed. On a fallback match it does not know the region it hit was trimmed differently from what it wrote.
- What it provides: byte-identical behavior, which is the whole reason to copy the surface.
- What breaks: nothing, but it is information the harness has and throws away.

**Option B: keep the sentence and append a short numbered excerpt of the changed region.**

- What it costs: a slightly different contract, and a decision about excerpt size.
- What it provides: the model sees the result of a fallback match with the file's indentation, which is the information it lacks.
- What breaks: a model that pattern-matches on the exact sentence. Low risk, since the sentence is still the prefix.

**Other harnesses on the result text.** Rig returns `Edited {path}: lines 2-4 (+1 line).` plus the changed lines with 3 lines of context, at most 60 shown lines and 300 characters per line, with no diff markers (source: `rt/edit.rs:218-284`, tests `rt/edit/tests.rs:80-177`). Gemini CLI returns `Here is the updated code:` with a 5-line-context snippet (probe: 13 lines for a one-line change in a 30-line file, but the whole file on a CRLF file). Cline returns `Edited <path>` plus a diff capped at 200 lines. OpenCode, Goose and zeroclaw return one line (`Edit applied successfully.`, `Edited {path} (a lines -> b lines)`, `Edited {path}: replaced 1 occurrence ({n} bytes)`), and only OpenCode appends LSP errors for the file. Rig's reply went through three rewrites in four days (unified diff, then changed line numbers only, then lines with context), which is the best evidence that the size is a tunable, not a given (source: rig commits `c9c928d4f`, `e034438fa`, `bfc4efca6`).

**Recommendation: Option A for the first implementation, Option B as a deliberate later divergence.** Confidence: high on A as the starting point, because a fresh harness should get the boring behavior right before adding value, and because the fallback-match ambiguity is better fixed by telling the model in the *description* that the replacement is not re-indented. What moved it: rig's numbers make Option B cheap to specify later (bounded at 89 file lines), but no transcript in the sources shows a model skipping a Read because of it.

### 6. Do you keep the two-valued splice, or unify it?

Cursor has a bulk descending splice for non-empty multi-match replacement and a sequential loop for everything else, plus line-eating for empty replacements.

**Option A: copy both paths.** They exist because the deletion path needs per-match lookahead at the surrounding line, which a bulk splice cannot express.

- What it costs: two code paths, and the bulk path has a bail-out (`if (start < 0 || end < start || end > tail) return undefined`) that falls through to the sequential path when offsets are inconsistent. That bail-out is only reachable from the fallback path, whose matches can in principle overlap.
- What it provides: correct deletion behavior, and an O(n) bulk path.
- What breaks: nothing. The bail-out is dead in practice for exact matches because they are always disjoint and ordered.

**Option B: one unified path.** Compute the deletion-adjusted ranges first, then splice them all descending.

- What it costs: you must run the line-eating adjustment for every match before splicing, using the original text for the lookahead. That is straightforward, but it is not what Cursor does, and the two are only equivalent if the adjustments are computed against the original coordinates.
- What it provides: one path, no bail-out, and the guarantee that the same set of matches always produces the same result.
- What breaks: the sequential path's adjustment reads `text` *after* earlier splices, because it reassigns `text` each iteration. Since matches are processed in descending order, earlier splices are all at higher offsets and cannot affect the lookahead of a lower one. So the two are equivalent in practice, which means unifying is safe.

**Recommendation: Option B, one unified path, computing adjustments against the original snapshot.** Confidence: medium-high, because the unified path is provably equivalent to Cursor's two paths for the inputs Cursor can produce, and it removes a bail-out that is otherwise a correctness question you have to keep answering.

### 7. Do you copy the `dsv3-1018` snake_case branch?

**Option A: only the `latest` shape.** `path`, `old_string`, `new_string`, `replace_all`, PascalCase name `StrReplace`.

- What it costs: nothing.
- What it provides: the surface the majority of models see.
- What breaks: nothing, unless you run DeepSeek models.

**Option B: both branches.** Add `search_replace` with `file_path` when the model is a DeepSeek family model.

- What it costs: a second schema, a second description, and the typo decision below.
- What it provides: a model trained on the leaked DeepSeek prompt sees the names it expects.
- What breaks: nothing, but it doubles the surface you test.

**Recommendation: Option A first, with the schema shaped so Option B is a table entry rather than a rewrite.** Confidence: high, because the parameter *names* are the only thing that differs and a per-model table is cheap to add later. One difference is worth preserving if you add it: the `dsv3-1018` description says `ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.` and ends `To create or overwrite a file, you should prefer the write tool.`, while the default branch drops that line and appends `If you want to create a new file, use the Write tool instead.` Those are two different prompts.

### 8. Do you reproduce the typo?

The `dsv3-1018` `replace_all` description reads `Replace all occurences of old_string (default false)` - one `r`, one `s`. The default branch reads `occurrences`.

**Option A: reproduce it on that branch only.** **Option B: fix it everywhere.**

**Recommendation: reproduce it if and only if you ship the `dsv3-1018` branch, and put a comment on the line saying it is deliberate.** Confidence: high, because a one-character difference in a description is invisible to a reader and load bearing for a byte-exact comparison, and the failure mode is that someone "fixes" it in six months and silently changes a prompt. If you do not ship that branch, the question disappears.

### 9. Do you gate the tool by model vendor, as the shipped build does?

The shipped build offers `StrReplace` plus `Write` for Anthropic, Google, xAI, zai, and unknown vendors, and for OpenAI and Cursor vendors only when the model is *not* GPT-5 family. GPT-5 family models get `ApplyPatch` instead.

**Option A: always offer `StrReplace`.** It is one tool, it works, and a harness that is not chasing Cursor's per-model tuning should not copy the tuning.

- What it costs: a GPT-5 class model gets a tool it was not tuned for, which is an unmeasured cost.
- What it provides: one code path, one test surface, no dependency on a vendor enum you have to keep current.
- What breaks: nothing.

**Option B: reproduce the gate.** Offer `ApplyPatch` for GPT-5 family.

- What it costs: you now need `ApplyPatch`, which is a patch parser and a different matcher.
- What it provides: parity with the shipped behavior.
- What breaks: you have two edit tools to keep consistent.

**Recommendation: Option A, and record the divergence.** Confidence: high, because `ApplyPatch` is a whole second tool with its own parser, and the reason Cursor gates is model tuning rather than correctness. If you later measure that a GPT-5 model does worse with `StrReplace`, add `ApplyPatch` as a Pack variant, which is what a Pack system is for.

### 10. Does one call take one edit or an array, and are the entries matched against the original or the running text?

`### The pi_edit counterpart` recommends the array shape; the open part is the matching text. The other harnesses split four ways.

| Option | Who | Failure it causes or removes |
|---|---|---|
| A. One edit per call | Cursor `StrReplace`, Claude Code, Gemini CLI, OpenCode, Cline `editor`, Goose, zeroclaw | ten edits are ten read-modify-write passes |
| B. Array, one snapshot of the original, all-or-nothing | pi, Cursor `pi_edit`, Copilot `multi_replace_string_in_file` (range-overlap check), rig until 2026-10-19 | `{a to b}` then `{b to c}` on a file holding `a` fails, because `b` is not in the original (probe) |
| C. Array, in order on the running text, all-or-nothing | rig since 2026-10-19 (`c2b6c3923`), test `edits_apply_in_order_even_where_they_overlap` | a later anchor can be changed or duplicated by an earlier edit; rig names it: `edits[1]: old_text was in the file, but edits[0], applied before it, changed that text` |
| D. Array, in order, not atomic | Zed `edit_file` | edit N fails, edits 0 to N-1 stay applied and the buffer is saved; the error shows the partial diff |

A fifth shape is a patch grammar (Codex, Cline `apply_patch`, Aider's blocks): Codex verifies every hunk against the current files before any write, so one bad hunk means zero writes, but applies hunks one file at a time with no transaction after that (source: Codex `core/tests/suite/apply_patch_cli.rs:1471-1489`, `apply-patch/src/lib.rs:468-683`).

**Recommendation: Option C, in-order and all-or-nothing, with a pinned test either way.** Confidence: low, because rig's benchmark and the reason for its reversal are not in the sources; the only evidence is that the one harness that shipped B and C chose C four days later and kept a test that pins it. In-order is also the composition of the single-edit tool a model already knows (N `StrReplace` calls, one claim, one write), so no new matching rule and no overlap rule is needed. Matching against the original suits order-independent edits, as in pi, and not a model that reasons step by step. Option D is the one to reject: a half-applied file is only safe if the result shows the diff, as Zed's does.

## The contract

### Names

The internal id is always `STR_REPLACE`. The model-facing name has exactly two values.

| Version tag | Name |
|---|---|
| `dsv3-1018` | `search_replace` |
| `cursor-0226` | `StrReplace` |
| `dsv3-1205` | `StrReplace` |
| `latest` | `StrReplace` |
| `gpt5-codex` | `StrReplace` |
| `codex-cloud` | `StrReplace` |
| `haiku` | `StrReplace` |

The generator is a two-armed function, not a switch with a default that returns `StrReplace`:

```js
function (e) {
  if ("dsv3-1018" === e) return "search_replace";
  return "StrReplace";
}
```

Every other file tool names its `dsv3-1018` branch in snake_case too (`write`, `delete_file`, `edit_notebook`, `list_dir`), so the snake_case names form a whole DeepSeek surface rather than a quirk of this tool.

There is no `edit_file` name for this tool. `edit_file` appears in the bundle only inside old prompt text and protobuf identifiers, never as `STR_REPLACE`'s model-facing name.

**And there is a vendor gate on whether the tool exists at all.** The tool list builder switches on `modelInfo.vendor`:

```js
switch (e.modelInfo.vendor) {
  case "anthropic": case "google": case "xai": case "zai": case "unknown":
    t.push(n()), t.push(r()); break;          // StrReplace, Write
  case "openai": case "cursor":
    e.modelInfo.isGpt5Family ? t.push(s())     // ApplyPatch only
                            : (t.push(n()), t.push(r()));
    break;
  default: unreachable
}
```

So on a GPT-5 family model `StrReplace` is absent and `ApplyPatch` takes its place. The parent spec describes `StrReplace` as always present, which is wrong for this build. See `## Where this build differs from the parent spec`.

### Parameters

Default branch, verbatim:

```js
const oke = U$.Ikc({
  path: U$.YjP().describe("The absolute path to the file to modify"),
  old_string: U$.YjP().describe("The text to replace"),
  new_string: U$.YjP().describe("The text to replace it with (must be different from old_string)"),
  replace_all: U$.zMY().optional().describe("Replace all occurrences of old_string (default false)"),
});
```

`dsv3-1018` branch, verbatim:

```js
const ike = U$.Ikc({
  file_path: U$.YjP().describe("The path to the file to modify. Always specify the target file as the first argument. You can use either a relative path in the workspace or an absolute path."),
  old_string: U$.YjP().describe("The text to replace"),
  new_string: U$.YjP().describe("The text to replace it with (must be different from old_string)"),
  replace_all: U$.zMY().optional().describe("Replace all occurences of old_string (default false)").default(!1),
});
```

| Parameter | Type | Required | Default | `.describe()` text (default branch) |
|---|---|---|---|---|
| `path` | string | yes | none | `The absolute path to the file to modify` |
| `old_string` | string | yes | none | `The text to replace` |
| `new_string` | string | yes | none | `The text to replace it with (must be different from old_string)` |
| `replace_all` | boolean | optional | **none** | `Replace all occurrences of old_string (default false)` |

`dsv3-1018` differences: `file_path` instead of `path`, a longer `file_path` description, `occurences` misspelled, and a `.default(false)`.

The absent default is load bearing. On the default branch `replace_all` arrives as `undefined`, and the code normalizes with a triple-equals test:

```js
const replaceAll = edit.replace_all === true;
```

That is equivalent to `Boolean` for a zod-validated boolean, but it is *not* equivalent for a value that slipped past validation, and it means `replace_all: false` and an omitted `replace_all` produce the same behavior through a different route. On the `dsv3-1018` branch the field is `false` by the time the code sees it.

`U$.Ikc` builds a plain `ZodObject` with no catchall, so unknown keys are stripped rather than rejected. A model that sends `explanation` gets it silently dropped.

### Parameters that do not exist

Implementations keep adding these:

| Name | Present on `STR_REPLACE`? | Where it lives |
|---|---|---|
| `explanation` | no | `SEMANTIC_SEARCH`, `SWITCH_MODE`, old `run_terminal_cmd` |
| `line_range` | no | `READ` on the `haiku` branch, `AI_ATTRIBUTION` |
| `old_str` / `new_str` | no | Anthropic-native `str_replace_editor` provider schemas only |
| `target_file` | no | `READ` on `haiku`, and `DELETE` |
| `contents` | no | `WRITE` |

### Description, verbatim

`cursor-0226` gets one line and nothing else:

```
Performs exact string replacements in files.
```

The default branch renders as (quoted verbatim, including the em dash Cursor uses, which this document forbids in its own prose):

```
Performs exact string replacements in files.

Usage:
- When editing text, ensure you preserve the exact indentation (tabs/spaces) as it appears before.
- Only use emojis if the user explicitly requests it. Avoid adding emojis to files unless asked.
- The edit will FAIL if old_string is not unique in the file. Either provide a larger string with more surrounding context to make it unique or use replace_all to change every instance of old_string.
- Use replace_all for replacing and renaming strings across the file. This parameter is useful if you want to rename a variable for instance.
- Optional parameter: replace_all (boolean, default false) — if true, replaces all occurrences of old_string in the file.

If you want to create a new file, use the Write tool instead.
```

The dash in the `Optional parameter` line is U+2014 EM DASH, three bytes `0xE2 0x80 0x94` in the file. Verified at bundle offset 4226537.

The `Write` name is not a literal. It is resolved with `Bj(t, "WRITE")`, which throws `` `Required tool ${t} not found in allTools` `` if `WRITE` is absent from the tool set. So a harness that offers `StrReplace` without `Write` hard-fails while building the description, before any tool call. If you are building a subset, either keep the reference or substitute a literal and note it.

There is a conditional line the default branch adds when `isComposer15` is set:

```
- The edit will FAIL if path isn’t given as the first argument. Always provide path first.
```

with a U+2019 RIGHT SINGLE QUOTATION MARK in `isn’t`, verified as three bytes `0xE2 0x80 0x99`. This line is model-family-conditional.

`dsv3-1018` differs in three places: it adds `- ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.`, it ends with `- To create or overwrite a file, you should prefer the write tool.` instead of the `If you want to create a new file...` paragraph, and it has the `occurences` typo in the parameter description rather than the description body.

### Caps and limits

| Limit | Value | Unit | Enforced by |
|---|---|---|---|
| whole-file readability | 100,000 | JS string length (UTF-16 code units) | the read stage's `exceededLimit`, which `StrReplace` does not check |
| read truncation | 8 MiB | JavaScript string length (UTF-16 code units) | the read executor's streaming cap, surfaced as `truncated` |
| full content in the result | 524,288 (512 KiB) | chars | `vbe`, which drops `before`/`after` when exceeded |
| diff computation worker threshold | 262,144 | chars, summed over both sides | `Lbe`, above which a worker thread runs |
| diff telemetry threshold | 2,000 | Myers edit length | `Ube`, telemetry only |
| fuzzy hint context | 2 before, needle-lines + 4 total | lines | the did-you-mean builder |
| longest-word filter in the hint | `length > 2` | characters | the did-you-mean builder's second pass |
| lock wait | unbounded | wall clock | the lock manager, aborts only |
| plan-mode allowlist | 5 extensions plus canvas | - | `x_e` |

The 100,000-character cap is enforced by `exceededLimit`, which `StrReplace` does **not** check. It checks `truncated`. Those are different fields produced by different layers: `exceededLimit` is a success record with empty content, and `truncated` is the executor saying it only read part of the file. A file of 99,999 characters with a `limit` supplied would trip `exceededLimit` on a Read but not on this tool, because the tool supplies no `limit`. A file above 8 MiB trips `truncated` and gets `File is too large to edit`. So the practical editable ceiling is the read executor's streaming cap; the 100,000-character read cap does not apply.

That check is the one place the shared edit pipeline is safer here than in `Write`. `Write` runs the same read and the same write executor but never inspects `truncated`, so it will overwrite a file it only partly saw. See the edit-pipeline section of `tool-write.md`. If you build one shared pipeline for both tools, put this check in the shared layer and let `Write` inherit it.

### Result cases

The result is an `EditToolCall` holding an `EditSuccess` on success. The model-visible text is the `message` field, rendered through a shared renderer.

| Case | Model-visible text | `isError` |
|---|---|---|
| success, `replace_all` false | `The file <path> has been updated.` | false |
| success, `replace_all` true, 1 match | `1 occurrence in file <path> has been updated.` | false |
| success, `replace_all` true, N matches | `N occurrences in file <path> have been updated.` | false |
| `fileNotFound` | `File not found: <path>` | true |
| `readPermissionDenied` | `Read permission denied: <path>` | true |
| `writePermissionDenied` | `Write permission denied: <path>` or `Write permission denied: <path>: <error>` | true |
| `writePermissionDenied` with `isReadonly` | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` | true |
| `rejected` with a non-plan-mode reason | `Rejected: <reason>` | true |
| `rejected` with the plan-mode reason | the long plan-mode paragraph naming `SwitchMode` | true |
| `error` | `Error: <modelVisibleError>` | true |
| undefined case | `Unknown error` | true |

The `<path>` in the success text is the unresolved raw argument. In the diff it is the raw argument too, which means the unified diff headers read `--- a//Users/x/f.ts` with a doubled slash on an absolute path.

The success grammar agrees in number: `1 occurrence ... has been updated`, `3 occurrences ... have been updated`. `replace_all: false` never reports a count even though it always modified one match. There is no trailing space and no diff stat, so the sentence is the entire payload.

### The `pi_edit` counterpart

The bundle registers a parallel `pi_*` family beside the seven tools. `PI_EDIT` is at 4559169 (shipped build; the parent spec does not cover this family at all), named `pi_edit`, described as:

```
Edit a single file using pi-compatible exact text replacements.
```

Its schema, verbatim:

```js
const KRe = U$.Ikc({
  oldText: U$.YjP().describe("Exact text for one targeted replacement. It must be unique in the original file and must not overlap with any other edits[].oldText in the same call."),
  newText: U$.YjP().describe("Replacement text for this targeted edit."),
});
const QRe = U$.Ikc({
  path: U$.YjP().describe("Path to the file to edit (relative or absolute)"),
  edits: U$.YOg(KRe).describe("One or more targeted replacements."),
});
```

So the shape is `{ path, edits: [{ oldText, newText }, ...] }`, and the schema's own words settle three things:

- `oldText` "must be unique in the original file". That is the atomicity contract stated in the parameter description itself: uniqueness is checked against the pre-edit content, not against a running result.
- `oldText` "must not overlap with any other `edits[].oldText` in the same call". Overlap is a call-level constraint, not a per-edit one.
- `path` is `relative or absolute`, with none of the default branch's "absolute path" claim.

The executor is `LocalPiEditExecutor` in the daemon, and it is short enough to reproduce almost in full:

```js
const resolvedPath = resolvePiPath(args.path, this.workspacePath);
return withPiFileMutationQueue(resolvedPath, async () => {
  const shouldBlockRead = await this.permissionsService.shouldBlockRead(resolvedPath);
  if (shouldBlockRead) return this.error(`Read blocked: ${blockReasonToString(shouldBlockRead)}`);
  const stats = await stat(resolvedPath);
  if (stats.isDirectory()) return this.error(`Path is a directory: ${resolvedPath}`);
  const rawContent = (await readFile(resolvedPath)).toString("utf8");
  const editResult = applyPiEdits(args.path, rawContent, args.edits);
  const writeResult = await this.writeExecutor.execute(...);
  ...
});
```

| Question | Answer |
|---|---|
| Shape of each edit | `{ oldText, newText }`, both required strings, no `replaceAll` flag anywhere |
| Applied in order or atomically? | **Atomically.** All edits are matched against one snapshot, then spliced in one pass. (Rig did the same until 2026-10-19, then switched to in-order application on the running text.) |
| What happens when one fails? | **Nothing is written.** The throw happens during matching, before the write. All-or-nothing. |
| Do offsets refer to the original or the mutated text? | **The original.** Verified: `{a→b}` then `{b→c}` on a file containing `a` throws `oldText was not found in the file.` for the second edit, because `b` does not exist in the original. |
| Is there a `replace_all` equivalent? | **No.** Duplicate `oldText` in one call yields `edits must not overlap.` rather than a bulk replace. |
| Matching rule | Exact substring; if *any* edit misses, the whole call retries against a fuzzy-normalized copy of the file |
| Fuzzy normalization | `NFKC`, then U+2018/U+2019 to `'`, U+201C/U+201D to `"`, U+2013/U+2014 to `-`, U+00A0 to a space, then `trimEnd()` per line. The same function also runs on `newText`, so a fuzzy edit rewrites the replacement too. Pi's own fold is wider (U+201A, U+201B, U+201E, U+201F, U+2010 to U+2012, U+2015 and U+2212 as well), so nine characters pi matches fail here (daemon `fuzzyNormalize`; pi `edit-diff.ts:34-54`; probe) |
| Empty `oldText` | rejected: `oldText must not be empty.` |
| Empty `edits` | rejected: `edits must contain at least one replacement.` |
| No-op edit | rejected: `edits did not change the file.` |
| Missing `oldText` | `oldText was not found in the file.` |
| Ambiguous `oldText` | `oldText must be unique in the file.` |
| Overlapping ranges | `edits must not overlap.` |
| Success text | `Successfully replaced N block(s) in <path>.` with a literal `(s)`, no pluralization |
| Diff context | 4 lines (`{ context: 4 }`), versus Cursor's 3 on the main surface |
| Lock | a per-realpath promise chain (`withPiFileMutationQueue`), not the agent's `fileOperationLockManager` |
| BOM | stripped before matching, re-prepended after |
| Line endings | `\r\n` if any `\r\n` is present in the file, else `\n` (not a dominant ending: pi uses the first ending in the file, Cursor uses any CRLF, and neither counts a majority). Content is normalized to LF, a lone `\r` becomes `\n` on every edit, then every `\n` is rewritten to the chosen ending, so a mixed file is homogenized (daemon `detectLineEnding`; pi `edit-diff.ts:11-25`; probe) |

The atomic all-or-nothing array is a better contract than `StrReplace`'s single edit for a model that wants to make four changes to one file. Copilot's `multi_replace_string_in_file` is also an array and refuses edits whose ranges overlap. Zed's `edit_file` is an array too, but it is not atomic: edits apply one after another to the live buffer, and edits before a failing one stay applied and saved. Rig started as an atomic array against one snapshot and switched to in-order application on 2026-10-19 (see decision 10). There is one shipped weakness in `pi_edit` that you must not copy, and it is a version problem rather than a design one: pi's own edit tool had it and fixed it.

**On the fuzzy path, Cursor's `pi_edit` rewrites the entire file, which is what pi's own tool did until commit `128330e36` (2026-06-19, issue 5899, released in pi 0.79.9).** Cursor ships a version pi outgrew. Pi's comment at that commit's parent admitted the design ("the output will have normalized whitespace/quotes/dashes, which is acceptable since we're fixing minor formatting differences anyway"), and the fix added regression tests with untouched lines ending in trailing spaces (source: pi `edit-diff.ts` at `128330e36^`, `tools.test.ts:1325,1340`). Cursor's port is not a copy of any pi revision: none of its four distinct error strings appears in any pi commit (`git log --all -S`), so it is an independent reimplementation that lacks the fix (probe). The full comparison of pi's original, pi at HEAD and Cursor's port is under `### pi` in the harness section. In Cursor, `fuzzyNormalize` runs `NFKC`, folds curly quotes to ASCII, folds en and em dashes to hyphens, replaces U+00A0 with a space, and applies `trimEnd()` to every line. The matches are computed against that normalized copy and spliced into *that copy*, which is then written back. Verified on a file with a curly-quoted string, a non-breaking space, and a trailing-space line: an edit that only touched `const s = 'quoted';` also removed the trailing spaces from an unrelated line and replaced a U+00A0 in a third line with a plain space. Input and output, verbatim from the reimplementation:

```
input : "const a = 1;   \nconst s = ‘quoted’;\nconst n = 1\u00a0+ 2;\ntarget();\n"
fuzzy : true
output: "const a = 1;\nconst s = 'renamed';\nconst n = 1 + 2;\ntarget();\n"
```

Three lines changed, one of them the intended one. A commit in `zhayujie/CowAgent` (web) was written to fix the same failure, and pi fixed it in its own tool. Pi at HEAD is line-confined, not clean: it rewrites the whole touched line from normalized text, and it throws on a whitespace-only last line (see `### pi` below). The array shape is good; the lesson is that a normalizing matcher must compute offsets in the original text and splice there, and Cursor's port does not.

**Recommendation for a fresh implementation: take `pi_edit`'s parameter shape and reject its matcher.** Confidence: high, because the array makes a multi-site edit one call, one claim, one write, one diff, and the matcher defect is shown in code and by a probe. Take pi's own `edits` description with it (`Each edit is matched against the original file, not incrementally...`), which Cursor's port drops, and pi's `prepareArguments` repairs for a stringified `edits` or a single edit object, which Cursor lacks (source: pi `edit.ts:35-38,103-134`). The matcher should be the `StrReplace` one, run per edit, with the offsets spliced into the original text. Whether each entry matches the original or the running text is decision 10. Matching against the original is defensible but not strictly better than in-order: rig shipped it and reversed it on 2026-10-19.

## Algorithm and ordering

Two layers. A wrapper that owns the lock, the read, the diff, the plan-mode gate, and the write; and a pure engine over a string. The engine's order is the contract.

The plan-mode gate sits in `Rbe`, the shared write executor, as its first statement, and `Sbe` also calls it before its read. So the read, the transform, and the diff all happen above the gate, and `StrReplace` adds one early gate that `Write` does not have. See step 1 and step 10 in Layer 2.

### Layer 1: the streaming wrapper and the lock

Steps 1 through 4 run before the arguments are even parsed.

```
 1  STREAM   read the raw argument JSON as it arrives, chunk by chunk
 2  DETECT   watch for the `path` field (or `file_path` on dsv3-1018) to appear
             complete in the partial object
 3  LOCK     on the first complete path value and only once:
             lock = await lockManager.waitForLock(ctx, path)
             keyed on the RAW argument string, before any normalization
 4  EMIT     emit a partial tool call carrying the path so the UI can show it
 5  PARSE    schema.safeParse(JSON.parse(buffer))
             on failure THROW "Invalid arguments:\n" + issues
 6  CHECK    if no lock was acquired THROW
             "Failed to acquire file lock - path may not be parsed"
 7  EXECUTE  the core, with the lock held
 8  RELEASE  dispose the lock in `finally`
```

In step 3 the lock is taken on the raw argument string, so `C:\a\f.ts` and `c:/a/f.ts` are two different keys and do not serialize against each other. Cursor accepts that. If you key on the canonical path instead, you close a hole and you gain a normalization step before the claim, which means the claim happens after path validation and a malformed path never takes a lock. That is better, and it is a deliberate divergence.

Step 6 exists because the stream can end without `path` ever completing, which happens when the model emits malformed JSON or gets cut off. The lock variable stays undefined and the call fails with a string that names the cause.

The `streamField` is `new_string`, not `path`, in the streaming config:

```js
{ path: "path", streamField: "new_string", fileOperationLockManager }
```

That is what drives the live `editToolCallDelta`, which holds `streamContentDelta` and is how the UI renders a diff while the model is still writing the replacement.

### Layer 2: read, gate, diff, write

The gate is not in one place. `Sbe` calls `W_e(r, o)` as its first statement, before the read, and `Rbe` calls `W_e(r, i)` again before the write. So the plan-mode check runs before the read *and* before the write, and the first one usually wins. That is the one difference from `Write`, whose wrapper `Rbe` is reached only after the read and the diff, so a refused plan-mode write there still pays for both. Step 1 below is the early check; step 10 is the second one inside `Rbe`.

```
 1  GATE     if mode is PLAN and the path is not markdown or a canvas path
                THROW plan-mode refusal            (checked TWICE: here and again in step 10)
 2  READ     service.read(path, toolCallId)
 3  SWITCH   on read result
                fileNotFound      -> THROW "File not found"
                permissionDenied  -> THROW "Read permission denied: <path>"
                invalidFile       -> THROW "<reason or default>: <path>"
                rejected          -> THROW "Edit rejected: <reason>"
                error             -> THROW passthrough, with two special cases:
                                     a worktree-guard sentinel rethrown as-is,
                                     and "Unable to resolve nonexistent file" remapped to fileNotFound
                success           -> continue
 4  TEXT     if output is not `content`  THROW "File is not a text file (binary data)"
 5  SIZE     if the read was truncated    THROW "File is too large to edit ..."
 6  NORMAL   contentsLF = original with /\r\n/g -> "\n"        // note: NOT /\r\n?/
 7  ENGINE   edited = editEngine(contentsLF, old, new, replaceAll, eol="\n", fallback=true)
 8  DIFF     computeDiff(original = contentsLF, new = edited, type = "modified", tool = "str_replace")
 9  MESSAGE  build the success sentence from the RAW `replace_all` argument and `occurrencesModified`
             (the raw argument, not the normalized replaceAll: the helper tests `t` truthily, so a
             schema-valid `false` and an omitted value both take the singular branch)
10  GATE     plan-mode check again, at the top of the write stage
11  WRITE    service.write(path, edited)
12  SWITCH   on write result
                success          -> recordFileState(path, new, original); return
                permissionDenied -> THROW, with the readonly variant
                noSpace          -> THROW "No space left on device"
                rejected         -> THROW "Edit rejected: <reason>"
                error            -> THROW passthrough
```

Step 6 preserves a subtle asymmetry. The engine's own normalizer uses `/\r\n?/g`, which collapses lone `\r` as well as `\r\n`. The wrapper's normalization uses `/\r\n/g`, which collapses only `\r\n`. So for a classic-Mac file with lone `\r` endings:

- the file content keeps its lone `\r`
- the model's `old_string` has its lone `\r` collapsed to `\n`
- the exact scan therefore cannot match across a line boundary
- the trimmed fallback can still match, because `trim()` eats a trailing `\r` at the line end
- the result reports `usedWhitespaceInsensitive: true` for what should have been an exact match

This is a wart. Reproduce it or fix it, but do not fix it by accident.

Step 8 uses `contentsLF` as the diff's original, rather than `original`. On a CRLF file that means the diff compares LF against LF and comes out clean, which is why the diff is not full of phantom changes. The original CRLF text is kept separately as `originalContent` and passed through to `recordFileState` and to `beforeFullFileContent`.

Step 10 is the same gate as step 1, which means a plan-mode refusal can fire twice in one call: once before the read and once before the write. In practice the first one always wins, so the second is unreachable unless the mode changes mid-call. It costs nothing and it is not worth reproducing.

Step 12's `recordFileState(path, newContents, original)` is the harness's undo record. It is not a model-facing feature.

### Layer 3: the pure engine

```
editEngine(content, oldRaw, newRaw, replaceAll, eol, fallback)

 1  newStr = normalizeEol(newRaw, eol)      // /\r\n?/ -> \n when eol is "\n"
    oldStr = normalizeEol(oldRaw, eol)
 2  if (oldRaw === "")            THROW "old_string is empty"        // raw, not normalized
 3  if (oldStr === newStr)        THROW "old_string and new_string are exactly the same"
 4  { matches, used } = findMatches(content, oldStr, fallback, eol)
 5  if matches is empty
        hint = buildDidYouMean(oldStr, content.split(eol))
        THROW "The string to replace was not found in the file"
              + (used ? " (even after relaxing whitespace)" : "") + "."
              + (hint ? " Found a possible fuzzy match, did you mean:\n" + render(hint) : "")
 6  if count(matches) > 1 and replaceAll is false
        THROW "The string to replace was found multiple times in the file"
              + (used ? " after relaxing whitespace" : "") + ". "
              + "Use replace_all to replace all occurrences, or include more context to only edit one occurrence."
 7  if replaceAll is false and used is false
    and content.indexOf(oldStr, matches[0].start + 1) !== -1
        THROW the overlap sentence
 8  return { contentsAfterEdit: splice(content, matches, newStr), used, occurrencesModified: count(matches) }
```

The emptiness test in step 2 is on `oldRaw`, the raw argument, not on the normalized `oldStr`. `normalizeEol` can never turn a non-empty string into an empty one, so the two are equivalent in practice, but the asymmetry matters if you ever add a normalizer that *can* empty a string, because the guard then moves.

Steps 2 and 3 both run before any search. `"  foo"` to `"foo"` normalizes to two strings that are not equal, so step 3 passes, then the search runs and finds nothing under the exact scan (the file has `foo` at column 0), then the fallback trims both sides to `foo`, they match, and the file becomes `foo`. That is a successful de-indent, because step 3 compares `oldStr` to `newStr` *without* trimming. The rule is narrower than "normalized equal is rejected".

The identity check is on EOL-normalized forms only. It does not trim. So:

- `old_string: "  foo"`, `new_string: "foo"` -> EOL-normalized forms differ -> step 3 passes -> exact scan finds nothing -> fallback trims and matches -> **the edit succeeds and de-indents the line.**
- `old_string: "foo"`, `new_string: "foo"` -> identical -> step 3 throws.

The parent spec says the de-indent attempt fails with `IdenticalStringsError`. That is wrong: the spec's worked example contradicts its own pseudocode, and the pseudocode is right. Verified against the shipped matcher: `runSingleEdit({contentsBeforeEdit: "foo\n", old_string: "  foo", new_string: "foo"})` returns `usedWhitespaceInsensitive: true` and produces `"foo\n"`. The outdent direction verifies the same way: `"    foo"` to `"  foo"` against a file line with four spaces produces `"  foo\n"`.

So the accurate statement of the rule is: **an `old_string` and `new_string` that are identical after line-ending normalization are rejected, and anything else runs the pipeline.** A line-ending difference alone does not save the pair, so `old_string: "a\r\nb"` with `new_string: "a\nb"` is an identity error: both normalize to the same text. Trimming is not part of normalization, which is why `"  foo"` to `"foo"` passes.

### Layer 4: the match finder

```
findMatches(content, search, enabled, eol)

 1  if content === ""            return { matches: search === "" ? [zero-width at 0] : [], used: false }
 2  if search === ""             log "[editV2SearchReplace] Search string is empty but text is not"
                                 return { matches: [], used: false }
 3  hits = []; i = 0
    loop  i = content.indexOf(search, i)
          if i === -1 break
          hits.push({ start: i, end: i + search.length })
          i = i + search.length                       // NOT i + 1
 4  if hits is non-empty         return { matches: hits, used: false }
 5  if enabled is false          return { matches: [], used: false }
 6  return { matches: trimmedBlockMatch(search, content, eol), used: true }
```

Step 1 and step 2 are unreachable from `StrReplace` because the engine's step 2 throws on an empty `old_string` first, and because a non-empty search against empty content simply finds nothing. They exist for the notebook caller, which handles the empty-cell case itself. A reimplementation can drop them if it has no notebook tool, but keep step 6's `used: true` even when the fallback returns zero, because that flag drives the `(even after relaxing whitespace)` suffix.

In step 3, `i += search.length` means overlapping occurrences do not count. Verified:

```
findMatches("aaa", "aa")  ->  [{ start: 0, end: 2 }]              count 1
findMatches("aaaa", "aa") ->  [{ start: 0, end: 2 }, { start: 2, end: 4 }]   count 2
findMatches("ababa", "aba") -> [{ start: 0, end: 3 }]             count 1
```

### Layer 5: the trimmed block matcher, in full

```
trimmedBlockMatch(search, content, eol)

 1  crlf = (eol === "\r\n")                       // always false for StrReplace
    lf   = s => s.replace(/\r\n/g, "\n")
    content = lf(content)
    search  = lf(search)
    needle  = search.split("\n").map(trim)
    if needle.length === 0 return []

 2  fileLines  = content.split("\n")               // raw, untrimmed
    fileTrim   = fileLines.map(trim)
    lineStart  = []; acc = 0
    for line in fileLines: lineStart.push(acc); acc += line.length + (crlf ? 2 : 1)

 3  out = []
    outer: for (start = 0; start <= fileLines.length - needle.length; start++)
        startOff = 0; endOff = 0
        for (n = 0; n < needle.length; n++)
            ft = fileTrim[start + n]; fr = fileLines[start + n]
            isFirst = (n === 0); isLast = (n === needle.length - 1)
            ok = (ft === needle[n])
            if isFirst
                ok = ft.endsWith(needle[n])
                if ok
                    startOff = fr.lastIndexOf(needle[n])
                    if ft === needle[n] startOff = 0
            if isLast
                if isFirst and ok and ft !== needle[n] continue outer   // reject a partial single line
                ok = ft.startsWith(needle[n])
                if ok
                    endOff = fr.indexOf(needle[n]) + needle[n].length
                    if ft === needle[n] endOff = fr.length
            if not ok continue outer
        out.push({ start: lineStart[start] + startOff,
                   end:   lineStart[start + needle.length - 1] + endOff })

 4  return out
```

The `lineStart` accumulator uses `crlf ? 2 : 1` for the newline width, and `crlf` is always false here because the engine is always called with `eol = "\n"` and the content has already had `\r\n` collapsed. So +1 is correct for this caller. A caller that passed `"\r\n"` would get wrong offsets unless the content still had CRLF; since the accumulator and the split disagree in that case, treat `crlf` as vestigial.

The loop bound is `start <= fileLines.length - needle.length`, and no inner loop can consume a differing number of lines, which makes the correspondence **strictly one-to-one**. A three-line search can only match three consecutive file lines. That is why a search missing one intermediate line simply fails rather than matching around the gap, and it is why there is no `...` elision.

### Worked example: tabs against spaces

File on disk:

```text
function foo() {
    if (x) {
        return 1;
    }
}
```

Model sends, where the escapes stand for literal characters:

```json
{"path": "...", "old_string": "\tif (x) {\n\t\treturn 1;\n\t}", "new_string": "\tif (y) {\n\t\treturn 2;\n\t}"}
```

Trace.

1. `oldStr` and `newStr` are unchanged by the EOL normalizer, since neither contains `\r`.
2. `oldRaw` is not empty. `oldStr !== newStr`. Both pass.
3. Exact scan: `content.indexOf("\tif (x) {\n...")`. The file contains no tab anywhere, so zero hits.
4. Fallback. `needle` after trim is `["if (x) {", "return 1;", "}"]`.
5. Window 0: `fileTrim[0]` is `"function foo() {"`, which does not end with `"if (x) {"`. Reject.
6. Window 1: `fileTrim[1]` is `"if (x) {"`, which ends with `"if (x) {"`. `startOff = fileLines[1].lastIndexOf("if (x) {")` = 4. But `fileTrim[1]` equals `needle[0]`, so `startOff` is forced to 0. That gives a region starting at the beginning of line 1, which is index 17; the first non-space character is at index 21.

That is the subtle part: a whole-line match under the fallback starts at column 0 of the raw line, rather than at the first non-space character. Consequence: the replaced region includes the file's leading indentation. Replacing it with a tab-indented `new_string` therefore *removes* the file's four spaces and substitutes one tab. That is what Cursor does, and it is what the parent spec's example shows:

```text
function foo() {
	if (y) {
		return 2;
	}
}
```

Verified by running the shipped matcher: the output is `function foo() {\n\tif (y) {\n\t\treturn 2;\n\t}\n}`. The replaced region takes the model's indentation. Cursor does not re-indent and does not preserve the file's indentation for a matched line. It only preserves the bytes *outside* the matched region.

The rule is therefore: **the fallback replaces whole trimmed lines including their original leading whitespace, and inserts `new_string` verbatim. Nothing is re-indented, in either direction.** A model that emits a replacement at the wrong depth produces a file at the wrong depth, and the tool reports success.

### Worked example: differing line counts

Same file. Model sends a two-line search that omits the closing brace:

```json
{"old_string": "\tif (x) {\n\t\treturn 1;", "new_string": "\tif (y) {\n\t\treturn 2;"}
```

`needle` is `["if (x) {", "return 1;"]`, two lines. Window 1:

- `n = 0`, first line, `endsWith("if (x) {")` true, `startOff = 0`.
- `n = 1`, last line, `startsWith("return 1;")` on `"return 1;"` true, `endOff = 0 + 9 = 9`.

So the region is line 1 start through the 9th character of line 2. The match is two lines. It succeeds, and the output is:

```text
function foo() {
	if (y) {
		return 2;
    }
}
```

The `    }` on the fourth line survives with its four spaces, because it was outside the region. That is a correct partial match under the fallback's first/last-line anchoring, and it is also a good example of why the tool cannot be trusted to produce consistent indentation.

Now a search that omits a *middle* line, over a four-line file:

```json
{"old_string": "\tif (x) {\n\t}", "new_string": "X"}
```

`needle` is `["if (x) {", "}"]`, two lines. The only window containing an `if (x) {` is window 1 (`"if (x) {"`, `"return 1;"`). `startsWith("}")` fails on `"return 1;"`, so no match at window 1. And window 0 is `"function foo() {"` which fails `endsWith`. So the result is zero matches:

```
Error: The string to replace was not found in the file (even after relaxing whitespace).
```

That is the one-to-one line correspondence at work, and it makes a good test vector.

### Worked example: `aa` in `aaa`, both settings

File content is `aaa`.

`replace_all: false`: exact scan finds one match at `[0,2)`. `count(matches)` is 1, so step 6 does not fire. `replaceAll` is false and `used` is false, so step 7 runs: `content.indexOf("aa", 0 + 1)` is 1, which is not -1, so the overlap guard throws:

```
Error: The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.
```

`replace_all: true`: same single match, step 6 skipped because `replaceAll` is true, step 7 skipped because `replaceAll` is true. Splice with `newStr = "X"`. The bulk path requires `newStr !== "" && count >= 2`, and count is 1, so it takes the sequential path and produces `Xa`. The message is `1 occurrence in file <path> has been updated.`

A `String.prototype.replaceAll` implementation produces `XX`. A scan that advanced by one would report two matches and, with `replace_all: false`, throw the "multiple times" sentence instead of the overlap sentence. Both divergences are observable, and both are worth a test.

### Layer 6: the splice

```
splice(text, matches, replacement)

 1  desc = matches sorted by start descending
 2  if replacement !== "" and desc.length >= 2
        tail = text.length; parts = []
        for m in desc
            if m.start < 0 or m.end < m.start or m.end > tail: bail to step 3
            parts.push(text.slice(m.end, tail)); parts.push(replacement)
            tail = m.start
        parts.push(text.slice(0, tail))
        return parts.reverse().join("")
 3  out = text
    for m in desc
        a = m.start; b = m.end
        if replacement === ""
            lineStart = out.lastIndexOf("\n", a - 1) + 1
            if out[lineStart ..< a] is all whitespace   a = lineStart
            lineEnd = out.indexOf("\n", b)
            if lineEnd !== -1 and out[b ..< lineEnd] is all whitespace   b = lineEnd + 1
            else if lineEnd === -1 and out[b ..] is all whitespace
                b = out.length
                if a > 0 and out.charCodeAt(a-1) === 10
                    a -= 1
                    if a > 0 and out.charCodeAt(a-1) === 13   a -= 1
        out = out.slice(0, a) + replacement + out.slice(b)
    return out
```

The empty-replacement clause is the whole-line-deletion behavior, and it has three cases:

- content before the match on its line is all whitespace, so eat back to line start
- content after the match on its line is all whitespace and a newline follows, so eat through the newline
- content after the match is all whitespace and there is no newline (last line), so eat to EOF and then eat back over one preceding `\n` and one preceding `\r`

Verified vectors:

| file | old | new | result |
|---|---|---|---|
| `line1\nline2\nline3\nline4` | `line2` | `` | `line1\nline3\nline4` |
| `line1\nline2\nline3\nline4` | `line2\n` | `` | `line1\nline3\nline4` |
| `line1\nline2\nline3` | `line3` | `` | `line1\nline2` |
| `line1\nline2\nline3` | `line1\n` | `` | `line2\nline3` |
| `line1\nline2` | `line1\nline2` | `` | `` |
| `a\nb\na\nb\n` | `a`, `replace_all` | `` | `b\nb\n` |

In the last row, with `replace_all: true` and an empty replacement, the bulk path is skipped (`replacement === ""`), so the sequential path runs twice, and each deletion eats its own line's newline. The result is `b\nb\n`, with no blank lines.

### Worked example: re-indentation of `new_string`

File `foo\n` at column 0. Model sends `old_string: "  foo"`, `new_string: "    foo"`.

1. Normalized forms: `"  foo"` and `"    foo"`. Not equal, so step 3 passes.
2. Exact scan on `"foo\n"` for `"  foo"`: zero hits.
3. Fallback: `needle = ["foo"]`, one line. Window 0: `fileTrim[0] = "foo"`, `endsWith("foo")` true, `startOff = 0` because `ft === needle[0]`. `isLast` also true, and `isFirst && ok && ft !== needle[0]` is false, so no rejection. `endOff = fr.length = 3`.
4. Region is `[0,3)`, the whole line without the newline.
5. Splice inserts `"    foo"`, giving `"    foo\n"`.

Verified: output is `"    foo\n"`. So a deliberate re-indent works, and it works through the fallback even though the file had no leading whitespace at all.

Now the case that fails:

File `foo\n`. Model sends `old_string: "foo"`, `new_string: "foo"`.

Step 3 sees `"foo" === "foo"` and throws `old_string and new_string are exactly the same`. So a model cannot "re-indent" by sending the same text, and cannot probe whether a line exists by sending it back unchanged. That is a deliberate no-op refusal, and it is the one failure in the tool that a model is likely to hit by accident when it second-guesses itself.

## Engineering questions

### Q1 Pagination and chunking

`StrReplace` does not page. There is no `offset`, no `limit`, no cursor. The whole file is read, matched against, and written back in one operation.

What a caller gets instead is a hard refusal: if the read executor had to truncate the file, the tool throws `File is too large to edit ...` rather than editing a prefix. That is the right choice, because editing a prefix of a file and writing the prefix back would truncate the file. The refusal exists because there is no way to page an edit.

The one place a boundary could fall mid-character is the read executor's streaming path, which decodes incrementally and cuts the retained text. Its `selectedTruncated` / `truncated: true` is what reaches the tool as `truncated`, and the tool refuses on that flag. So the case is handled by refusing rather than by reassembling, which is safe. Where it could still bite is a file just under 8 MiB with an astral character straddling the cut; the executor's `line.text.slice(0, remainingBytes)` can split a surrogate pair. That produces a lone surrogate in the truncated content, which the tool then refuses anyway, so it is invisible.

That `slice` is also the clearest example of a naming trap in this code path. The local is `remainingBytes`, the accumulator is `selectedBytes`, and the budget is `MAX_TEXT_SIZE`, but every value fed into the comparison is `line.text.length`, which is a JavaScript string length. So the unit is UTF-16 code units and only the names say bytes. A Rust port that reads the names and counts bytes will cut at a different point on any non-ASCII file.

The `truncated` flag has a second producer: when the executor's buffer threshold is passed it spills to a temp file, and a failure to create that temp file sets `truncated: true`. So a disk-full condition during the read surfaces as `File is too large to edit`, which is a misleading message for a disk problem, so fix it rather than copy it.

Other harnesses. Nothing else pages an edit. Rig refuses a file over 16 MiB (bytes) and cuts its reply at 60 lines; Cline caps each string at 6,000 characters and the diff at 200 lines; Goose and zeroclaw read any file whole with no cap; Zed accepts partial tool JSON as it streams and loads files up to 6 GiB (source: each harness H3).

### Q2 Units

| Quantity | Unit | Where |
|---|---|---|
| match offsets | JS string index (UTF-16 code units) | the engine, throughout |
| `old_string.length` in the overlap guard | UTF-16 code units | engine step 7 |
| the read cap that produces `exceededLimit` | UTF-16 code units, 100,000 | read result shaping, not checked by this tool |
| the read cap that produces `truncated` | JavaScript string length (UTF-16 code units), 8 MiB | read executor |
| `beforeFullFileContent` ceiling | UTF-16 code units, 524,288 | `vbe` |
| diff worker threshold | UTF-16 code units summed, 262,144 | `Lbe` |
| the "too large to edit" byte figure | bytes, from `fileSize` (a BigInt) | the throw's interpolation |
| the fuzzy hint window | lines | did-you-mean builder |
| `replace_all` count | matches, non-overlapping | `occurrencesModified` |

The 100,000-character read cap and the 8 MiB truncation cap measure different things at different layers, and this tool reacts only to the second.

The exact comparison for the read cap, from the shared read shaper:

```js
content.length > LIe ? { /* exceededLimit: true, value: "" */ } : { /* normal */ }
```

with `LIe = 1e5`. That is a pure length test after windowing, in code units rather than bytes or lines.

Other harnesses. Units differ: Zed matches exact text in bytes and fuzzy text in lines, pi, Gemini CLI, OpenCode and Cline in UTF-16 code units, rig, zeroclaw and Goose in bytes (Rust `match_indices`), Aider by whole lines. A Rust port that copies Cursor's UTF-16 offsets must convert at the boundary (source).

### Q3 Text encoding

**Invalid UTF-8.** `StrReplace` never sees bytes. The read executor decodes and hands the tool a JS string. Its behavior on invalid UTF-8, from the daemon: it tries a strict UTF-8 decoder, and on `InvalidUtf8Error` it falls back to a lossy `TextDecoder("utf-8", { fatal: false })`, and if the lossy result is "mostly valid UTF-8" it uses that, otherwise it falls back to `latin1`. So a file that is Latin-1 gets decoded as Latin-1, every byte maps to a character, and a round trip is lossless. A file that is mostly UTF-8 with a few bad bytes gets lossy decoding, and the bad bytes become `U+FFFD` replacement characters, which are then written back. **That is a silent corruption path**, and it is inherited by `StrReplace` because it shares the read stage. Decide deliberately: refuse rather than lossy-decode, or round-trip as bytes.

**BOM.** The read executor strips a leading U+FEFF and remembers it (`hadLeadingBom`). Whether the write path re-adds it depends on the encoding it settles on: the caching line is `cacheEncoding = decoderConfig.cacheEncoding === "utf8" && hadLeadingBom ? "utf8bom" : ...`, so a BOM'd file is remembered as `utf8bom` and the write encodes it back as UTF-8 with a BOM. So a BOM survives a `StrReplace` edit. The model never sees the BOM, so an `old_string` starting at the first character of the file matches against content with the BOM already removed. That is the right behavior.

**Unicode normalization.** None. No NFC, no NFD, no NFKC. Two visually identical strings can fail to match if one is composed and the other decomposed, and the fallback does not help because `trim()` does not normalize. This is a gap: a model reading a macOS-created filename's contents will see whatever the file contains, so the common case is fine, but a model that types a composed accent against a decomposed file fails.

**U+00A0.** `trim()` strips it at line edges, because JS `trim` covers the full `WhiteSpace` set which includes U+00A0. So a non-breaking space at the start or end of a line is neutralized by the fallback. A non-breaking space in the middle of a line is not, and fails. Verified:

```
content "  \u00a0const a = 1;\n", needle "const a = 1;"  -> exact hit, used false
content "x\u00a0y = 1;\n",        needle "x y = 1;"     -> no hit at all
```

The first succeeds because the exact scan finds `const a = 1;` as a substring after the NBSP; trimming plays no part.

**Smart quotes.** No folding, in either direction. `‘` does not match `'`. This is Cursor's one matching gap that Claude Code closes, and it is a deliberate difference: Claude Code folds the four quote characters, Cursor does not.

**`\uXXXX` escapes.** No handling. Claude Code tries both directions (unescape the needle, and re-escape literal characters into a regex); Cursor does neither. A model that emits `\u00e9` where the file has `é` fails.

Other harnesses. Strict UTF-8 with refusal: rig, zeroclaw, Goose, Codex. Lossy rewrite of bad bytes to U+FFFD: pi, Gemini CLI, OpenCode, Cline, Zed (after `chardetng`). Cursor falls back to latin1 and Claude Code also handles utf16 and latin1. Folding: pi (NFKC), rig's loose pass, Codex's fourth pass and OpenCode dev fold quotes, dashes and spaces, and none folds for the exact scan. NFC against NFD fails in rig, zeroclaw, Goose, OpenCode and Cline `editor`, and rig and Goose then print a hint that looks identical; only pi (NFKC) and Cline `apply_patch` (`canonicalize`) normalize it (probe).

### Q4 Line endings

This is where the tool is most subtle, because normalization happens in three places and one of them has a hole.

**Model strings.** `normalizeEol(s, "\n")` applies `/\r\n?/g -> "\n"`, which collapses both `\r\n` and a lone `\r`.

**File content.** The wrapper applies `/\r\n/g -> "\n"`, which collapses `\r\n` only. A lone `\r` survives into the engine.

**The write path.** On the way out, the write executor detects the file's format and decides. From `dr`:

```js
const u = t.includes("\r\n") ? void 0 : await detectMixedEndings(...);
if (void 0 !== u) t = At({ content: t, existing: u, defaultEnding: ... });
else if (s.lineEnding === Ft.CRLF) t = t.replace(/(?<!\r)\n/g, "\r\n");
```

So:

- If the content the tool produced already contains `\r\n`, nothing is done. That is Cursor's own tool never producing `\r\n`, but it matters for other writers.
- If the file on disk has mixed endings, `At` does a line-level LCS alignment against the existing content and restores each line's original ending. That is a per-line merge.
- If the file on disk is uniformly CRLF, a blunt `/(?<!\r)\n/g -> "\r\n"` runs.
- Otherwise the LF content is written as-is.

`At` is more careful than it looks. It splits both the incoming content and the existing content into `lines` plus per-line `ends` (either `\r\n` or `\n`), runs a diff similar to patience diff to align the line vectors, and then for each incoming line, if its ending is `\n`, substitutes the ending of the aligned existing line. So an edit inside a CRLF file that leaves 99% of lines untouched gets each untouched line's original ending back, and the inserted lines get the ending of the line they were aligned with. That is why a `StrReplace` on a mixed-ending file does not homogenize it.

**Net answer.** An LF `old_string` against a CRLF file is an **exact** match, with `usedWhitespaceInsensitive: false`, because the file was normalized to LF before the engine ran. The model never has to emit CRLF. The file's endings are restored on the way out, per-line for a mixed file and globally for a uniform one.

The hole is a lone-`\r` file. Trace:

1. Model sends `old_string: "a\nb"`. The normalizer collapses nothing, because there is no `\r`.
2. The file `"a\rb\rc\r"` is normalized with `/\r\n/g`, which does nothing. Content stays `"a\rb\rc\r"`.
3. Exact scan for `"a\nb"` fails, because the file has `\r` between `a` and `b`.
4. Fallback splits on `\n`, giving one "line" `"a\rb\rc\r"`, trims it to `"a\rb\rc"`, and compares against needle `["a", "b"]`. Two needle lines cannot match one file "line", so zero matches.
5. Result: `The string to replace was not found in the file (even after relaxing whitespace).`

Verified. So a classic-Mac file is effectively uneditable across line boundaries. A single-line needle inside it does work, because the exact scan is plain substring, and the result keeps the lone `\r`:

```
content "a\rb\rc", old "b", new "X"  ->  "a\rX\rc"
```

Verified. Whether the lone `\r` is later converted depends on the write path: `s.lineEnding` for such a file would be detected as LF (the sniffer looks for `\r\n`), so no conversion happens, and the lone `\r` survives.

**Recommendation for a Rust implementation.** Normalize `\r\n` and lone `\r` to `\n` for matching, and restore with the same per-line alignment for a mixed file. The lone-`\r` hole is a bug, not a design choice, and closing it costs one character change in a regex. Do not copy it. Do copy the per-line merge, because it is the reason a `StrReplace` never homogenizes a mixed file.

One more hole: the global `/(?<!\r)\n/g` rewrite on a uniformly-CRLF file will also convert a bare LF that the model deliberately put inside a string literal. A file with a literal newline byte inside a quoted string is pathological, but the regex cannot distinguish it from a line break. A per-line approach cannot fix this either; only a parser can.

Other harnesses. Gemini CLI and pi make a mixed file uniform; OpenCode forces CRLF onto the replacement in a mixed file; Cline edits the matched span only, but its `insert_line` flattens the whole file; Aider writes the platform ending, so a CRLF file becomes LF on Linux and an LF file becomes CRLF on Windows, touched or not. Codex keeps each untouched line's ending; rig converts only an all-CRLF file; zeroclaw and Goose do nothing, so a multi-line LF needle never matches a CRLF file (probe). The per-line merge is Cursor's alone (source, probe).

### Q5 Binary detection

Three layers, in order.

**Extension shortcut.** A binary extension list is tested before any read:

```js
const aTe = /\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i;
```

Case-insensitive, anchored at the end. A hit reaches a verdict without touching the file.

**Content sniff.** For an extension not on the list, the format sniffer reads a sample and decides. It checks, in order: BOM patterns for UTF-16LE, UTF-16BE, UTF-8 BOM, and UTF-32 variants; a heuristic on the first 50 two-byte pairs for UTF-16 without a BOM (`> 80%` of high bytes zero with printable low bytes means UTF-16LE, and vice versa); and then a NUL-byte scan. A NUL in the sample means binary.

**Decode failure.** If neither fires, the decode path above tries strict UTF-8, then lossy, then Latin-1. So a file is only "not text" if the extension list or the NUL scan says so.

What the model sees on a binary file:

```
Error: File is not a text file (binary data)
```

produced by the check `if (output.case !== "content") throw new BinaryFileError(...)`. A `dataBlobId` or `contentBlobId` output also lands here, so a file large enough to route through the blob store would be reported as binary if the read stage ever returned a blob instead of inline content. Whether that path is reachable for a text file is not determinable from the bundle; see `## What remains unknowable`.

`StrReplace` does not have its own binary rule. It inherits Read's, which is the right factoring.

Other harnesses. Of the others, only Zed sniffs content (the first 1,024 bytes, with a NUL and text-likeness ratio). Everyone else relies on UTF-8 decoding or does nothing; Aider crashes while building the failure text for an undecodable file (`AttributeError` on `None`) (source, probe).

### Q6 Empty input and empty result

| Situation | Behavior |
|---|---|
| empty file, `old_string` non-empty | `The string to replace was not found in the file.` with **no** relaxed-whitespace suffix, because the empty-content guard returns before the fallback runs. Verified. |
| empty file, `old_string: ""` | `old_string is empty`, thrown by the engine's step 2 before the matcher is reached. The matcher's own empty case is unreachable from this tool. |
| `new_string: ""` | a deletion. `new_string` is required, so it must be present, but an empty string is allowed. Triggers the line-eating path. |
| `old_string` is the whole file | matches, replaces entirely. With `new_string: ""` the file becomes 0 bytes. Verified. |
| `old_string` is the whole file plus a trailing newline | matches when the file ends with that newline, and the match includes the terminator. `"a\nb\n"` with `old_string: "a\nb\n"` and `new_string: ""` produces 0 bytes. Verified. |
| CR-only file, single-line needle | matches by exact substring; the `\r` bytes survive. `"a\rb\rc"` with `old_string: "b"` produces `"a\rX\rc"`. Verified. |
| CR-only file, needle spanning a line break | **not found**, even after relaxation, because the wrapper only collapses `\r\n` so the `\r` survives into the matcher and defeats both paths. Verified. |
| `old_string` ending with one newline, file line after it not blank | the fallback matches and includes the newline, so the replacement abuts the next line. `"\tb\n"` against `"a\n    b\nc\n"` produces `"a\nXc\n"`. Verified. |
| zero matches | `The string to replace was not found in the file (even after relaxing whitespace).` |
| zero matches with the fallback disabled | `The string to replace was not found in the file.` The suffix tracks whether the fallback *ran*, not whether it could have helped. |
| `old_string === new_string` | `old_string and new_string are exactly the same` |
| `replace_all: true` with zero matches | the same not-found error; `replace_all` does not change the zero case |
| empty `path` | the read stage fails; the string is whatever the read stage produces (`File not found` or an invalid-file message), not a dedicated empty-path error |
| `old_string` is a single whitespace character | matches literally. `"   "` against a file containing three spaces finds it by exact scan; verified `used: false`. |

The last row is the one place where an all-whitespace needle behaves differently from what a reader expects: `"   "` is not treated as empty and is not trimmed. Only `""` is refused.

Other harnesses. An empty `old_string` is refused by Cursor, rig, zeroclaw, OpenCode and Gemini CLI (on an existing file), but Goose treats it as a match on an empty file and as `chars + 1` matches otherwise, and Gemini CLI and OpenCode create the file when it is absent. Cline reports `text not found`. Deletion: Cursor eats the line's newline when a deletion leaves the line empty; pi and rig, which were probed, leave a blank line unless `old_text` took the newline.

### Q7 Cost

**Memory.** The whole file is held as a JS string twice at peak: `originalContent` (with whatever endings it had) and `contentsLF` (normalized). Then the edit produces a third, and the diff produces a fourth if it is materialized as a string. On an 8 MiB file that is roughly 16 to 48 MiB of UTF-16 (so up to ~96 MiB of bytes) before the diff. The 512 KiB `beforeFullFileContent`/`afterFullFileContent` cap only limits what goes into the result record; it does not limit what is held in memory.

**Time.** Three passes over the file at minimum: the read, the exact scan, the write. The exact scan is `indexOf` in a loop, which is effectively linear for a needle that does not match often and quadratic in the pathological case of a self-similar needle. The fallback is worse: it splits the file into lines, trims every line, and then does an O(lines × needle-lines) window scan with a `trim()`-comparison inner loop. On a 100,000-line file with a 50-line needle that is 5 million line comparisons, each of which is a string equality. It only runs when the exact scan found nothing, which is the case where a model is retrying, so it runs when the user is already waiting.

**The worst case a hostile repository can force.** A file just under 8 MiB, all on one line, with no trailing newline. Then:

- the read succeeds (under the truncation cap)
- the fallback's `split("\n")` produces one element, so the window loop runs once, but each `trim()` on an 8 MiB string copies it, so you allocate 8 MiB twice
- if the needle is a long repeated substring, the exact scan's `indexOf` loop is quadratic

The pathological `indexOf` case is bounded by the non-overlapping advance: a needle of length L in a file of length N does at most N/L full searches, each of which is O(N) worst case, so O(N²/L). For L = 1 that is O(N²). An 8 MiB file of a single repeated character with a 1-character needle is the worst case, and the scan does not stop at the first hit; it collects every match before deciding. So it builds an array of 8 million match objects, and memory rather than time is the hostile case here.

**Recommendation.** Cap the number of matches collected. You only need to know whether there are 0, 1, or more than 1 matches for the `replace_all: false` path, and for `replace_all: true` you need all of them, so cap `replace_all` too or stream the splice. Cursor does neither, and a model that sends `old_string: "e"` with `replace_all: true` on a large file will allocate a match object per `e`.

Other harnesses. A one-character needle is the hostile case everywhere: Cursor builds 1,048,576 match objects for 1 MiB of `e` in 139 ms (daemon, probe); Goose collects every position before the count check; pi builds a `split` array (8 MiB of `e` fails after 126 ms); rig's ambiguous path computes `line_of` for every match, which is quadratic (512 KiB of `a` lines took 110 s) (probe).

### Q8 Paths

**Relative or absolute.** The default branch's `path` description says `The absolute path to the file to modify`, but nothing enforces it. The path goes straight into the read request:

```js
const m = "path" in o ? o.path : o.file_path;
const h = new Ol.g$({path: m, streamContent: o.new_string});
```

No resolution, no `realpath`, no workspace join. Whatever the model sends is what the read executor receives, and the executor resolves it against the workspace if it is relative. The `dsv3-1018` branch documents this: `You can use either a relative path in the workspace or an absolute path.`

**`..`.** Not handled by the tool. The read executor resolves and the capability service decides whether the result is inside the workspace. The tool neither knows nor cares.

**Symlinks.** Resolved by the read executor, and again by the write executor. The tool does not resolve. `StrReplace` on a symlink edits the target, because both the read and the write follow. There is no no-follow flag on this path, unlike `Write`'s open flags. So a symlinked leaf is followed by `StrReplace` and refused by `Write`, which is an inconsistency between two tools that share the same write executor.

**Paths outside the workspace.** Allowed if the executor allows them. There is no worktree-escape check in this tool's own code; the worktree guard lives in the read/write executors and surfaces as a sentinel error that `StrReplace` rethrows verbatim rather than mapping.

**Windows drive letters, UNC, case sensitivity.** Nothing special in the tool. The lock keys on the raw string, so `C:\a\b.ts` and `c:\a\b.ts` are different keys on a case-insensitive filesystem, which means two calls can hold "the lock" for the same file simultaneously.

**Normalization order.** The path is used for the lock *before* any normalization (step 3 of layer 1), and used for the read and write *without* normalization. So the lock key and the operation target are both the raw string, and they agree with each other. The divergence is between two different raw strings that name one file.

**The `worktries` repair.** This quirk is adjacent to paths. The write path runs the argument through:

```js
const cke = /(^\/(?:Users|home)\/[^/]+\/\.cursor\/)worktries(?=\/|$)/g;
function lke(e) { return e.replace(cke, "$1worktrees"); }
```

So a path under a `.cursor/` directory with a `worktries` segment gets that segment rewritten to `worktrees`. The success message uses the repaired path. That is a targeted repair for one misspelling in one directory layout, and a fresh implementation has no reason to keep it.

Other harnesses. Gemini CLI silently retargets a missing relative path to a unique same-basename file (`correctPath`, a breadth-first search over at most 50 directories), pi cannot address a file whose name holds U+00A0, and Aider silently redirects an edit to the closest chat-file name or to any chat file that contains the block (source).

### Q9 Concurrency

**What is locked.** A per-path mutex from `fileOperationLockManager`, acquired during argument streaming as soon as the `path` field completes (layer 1, step 3), and released in `finally` (step 8).

**Keyed on what.** The raw argument string, before any normalization. See Q8.

**For how long.** From the moment the path field completes in the stream, through the read, the engine, the diff, and the write. So the lock covers the entire read-modify-write window inside this tool.

**What race that closes.** Two `StrReplace` calls in the same turn on the same path cannot interleave their read-modify-write sequences. Without the lock, call A could read, call B could read, A could write, and B could write, losing A's edit. This is the classic lost update, and the lock closes it for calls within one agent loop.

**What races it leaves open.**

1. **Across agents.** The lock manager is a per-turn instance. Ten `new lne` sites, all method-local; two matter, one per turn and one per step. Two chat tabs, two background agents, or a parent and a subagent each have their own empty lock set. So the lost update is fully available across agents.
2. **Across processes.** No OS-level lock at all.
3. **Against a non-`StrReplace` writer.** `Shell` takes an *exclusive* lock, so a shell command does serialize against edits. But an external editor, a file watcher, or a `git checkout` does not.
4. **Spellings.** Two spellings of one path do not serialize.

**The read-modify-write window and TOCTOU.** Inside the tool the lock covers it, so there is no internal TOCTOU. But there is no content check either: the tool reads, computes, and writes without re-verifying that the file still contains what it read. Compare `Write`, which computes a sha256 fingerprint of the content it read and re-verifies it immediately before the truncate, refusing with `{path} changed on disk after this tool call read it...` if it moved. `StrReplace` has nothing equivalent. Its only defense is that a concurrent change usually invalidates the anchor.

That defense is weak in the case that matters. Consider a file where the model's anchor is in a region the concurrent writer did not touch. The concurrent writer appends a function, or reformats a different part of the file. `StrReplace`'s read happened before that write; its own write happens after; the anchor still matches; the edit lands on top of a file that has changed, and the tool's write clobbers the concurrent change entirely, because `StrReplace` writes the whole file. So a concurrent modification outside the matched region is silently lost.

**What a Rust implementation should do differently.** Four things, in order of value.

1. **Compare-and-swap on a content hash.** Re-read or hash immediately before the write and refuse if the content is not what the engine matched against. This is `Write`'s existing check, applied here. Cost: one extra read, or one hash of content already in memory plus a stat-based shortcut. Value: closes the silent clobber.
2. **A splice operation in the VFS instead of a whole-file write.** If the backing store can apply a byte-range replacement, the concurrent-change window narrows to the range. This is the argument for Option C in decision 1.
3. **Atomic write.** Temp file plus rename. Cursor truncates in place, so a crash mid-write leaves a partially written file.
4. **A lock keyed on the canonical path, and scoped above the agent.** If multiple agents share a process, the per-agent lock is not enough, and the content check is the only thing that saves you.

One more concurrency behavior looks like a race and is not: `StrReplace` is in the read-retry whitelist. If a `Read` in the same step misses, it waits up to 120,000 ms for earlier calls in this set to settle and retries once. So a read issued in the same model response as an edit will wait for the edit rather than racing it. The set is:

```js
["ApplyPatch","AwaitShell","CopyToBox","EditNotebook","MultiEdit","MultiStrReplace","Shell","StrReplace","Write","edit_notebook","run_terminal_cmd","search_replace","write"]
```

`MultiStrReplace` appears in that set. There is a `MULTI_STR_REPLACE` tool id in the bundle's wire-case map (`MULTI_STR_REPLACE: "edit_tool_call"`) and in its team map (`MULTI_STR_REPLACE: "agent-quality"`), but **there is no `AG("MULTI_STR_REPLACE", ...)` registration in this build**. The id is referenced by the read-retry list and by two lookup tables, and nothing registers it. So a multi-edit tool existed at some point, or is behind a flag that is off, and in this build it is absent.

Other harnesses. In-process write serialization exists in pi (a per-realpath queue), Gemini CLI (a path lock), OpenCode (a semaphore held across the permission prompt) and rig only as footprint ordering inside one reply; Zed serializes on the `Buffer` entity; zeroclaw, Goose, Cline, Aider and Codex have none. Nobody has a cross-process lock. Besides Claude Code's read-state check and Cursor's own `Write` fingerprint, only rig's `write_if` and OpenCode V2's `writeIfUnchanged` check content at write time (source).

### Q10 Fidelity

If a caller reads a file and writes it back unchanged, what survives? For `StrReplace`, the more useful question is what survives an edit that touches one region.

| Property | Preserved? | Mechanism |
|---|---|---|
| bytes outside the matched region | **yes** | the splice is `slice(0, a) + new + slice(b)`; nothing else is touched |
| bytes inside the matched region | replaced by `new_string` verbatim, including its own indentation | there is no re-indentation |
| line endings, uniform LF | yes | nothing to do |
| line endings, uniform CRLF | yes, restored globally | `/(?<!\r)\n/g -> "\r\n"` |
| line endings, mixed | yes, restored **per line** | `At` line-level LCS alignment against the existing content |
| line endings, lone `\r` | preserved inside a matched single line; unmatchable across lines | the wrapper's `/\r\n/g` does not touch a lone `\r` |
| trailing newline | yes, unless the match consumed it | plain substring semantics |
| BOM | yes | stripped on read and re-added on write when the file is `utf8bom` |
| encoding, Latin-1 | yes, round-tripped through a Latin-1 decoder | every byte maps to one character |
| encoding, mostly-UTF-8 with bad bytes | **no**, the bad bytes become U+FFFD | lossy fallback decoder |
| encoding, UTF-16 | yes, decoded and re-encoded | format sniffer and encoder |
| invalid UTF-8 in a file that is mostly valid | **no**, silently corrupted | lossy decode then re-encode |
| file mode | not touched by this tool | no `chmod`, no mode preservation logic |
| ownership | not touched | - |
| mtime | updated by the write | - |
| hard links | the link is followed and its target rewritten in place, so other links see the change | truncate-in-place, no rename |
| extended attributes | not touched, but truncate-in-place on some platforms can drop them | below the tool |

The two silent-rewrite rows are the ones to decide about. The mixed-ending per-line merge is careful and worth copying. The lossy-decode path is a corruption bug and worth refusing instead.

**The naive-implementation trap.** An implementation that does `content.split('\n').map(trim)` to match, then `join('\n')` to write, will re-indent every line in the file. Cursor avoids this because the matcher returns offsets into the original string and the splice touches only those offsets. The test for it is one line: edit a file whose untouched lines have trailing spaces and mixed indentation, then assert the untouched bytes are byte-identical to the input.

Other harnesses. Bytes outside the match survive in zeroclaw, Goose, OpenCode, Cline `editor` and rig (exact path); Aider and Codex rewrite the whole file and add a missing final newline; Gemini CLI pins the final-newline status to the original; pi homogenizes endings on the exact path (source, probe).

### Q11 Determinism

**Output text.** Fully deterministic. The success sentence is a pure function of the path, `replace_all`, and the count. The failure sentences are pure functions of the case and the flag.

**Match order.** The exact scan is left-to-right by construction. The fallback scans windows in ascending `start` order. The splice sorts descending by start. All deterministic.

**The sort.** `matches.sort((a, b) => b.start - a.start)` is a numeric comparator, so it is a total order except for equal starts, which cannot happen for disjoint matches. Where two matches do share a start, which the fallback can in principle produce, the tie is unresolved and `Array.prototype.sort` stability decides. In practice the fallback's window loop pushes at most one match per window, so equal starts do not arise.

**The fuzzy hint.** Deterministic but tie-prone: `[...lines].sort((a, b) => b.length - a.length)[0]` picks the longest line, and among equal-length lines the sort is stability-dependent. A different JS engine could pick a different anchor line for a symmetric `old_string`. That does not change whether the edit applies, only which region the hint shows.

**The diff.** Myers, so deterministic for a given input pair. `linesAdded` and `linesRemoved` come from filtering hunk lines for a leading `+` or `-`, which is deterministic.

**Byte-identical reruns.** Yes for the file content and the model-visible text, given the same input bytes and the same `replace_all`. The variable is the line-ending restoration, and only if the file is being changed between the two runs.

So: two identical runs on the same tree produce byte-identical files and byte-identical messages. The one non-determinism is the fuzzy hint's anchor choice on a tie, which is cosmetic.

Other harnesses. Aider's fallback file is the first match in a Python `set`, so when two chat files hold the same block the winner changes across processes (probe); everything else in these tools is deterministic for the same bytes.

### Q12 Truncation and omission

The tool truncates nothing in its model-visible output, because its model-visible output is one sentence: there is no marker, no count, no "use offset to see more", and no way for the success message to lie about completeness.

But there are four places where information is dropped without the model being told.

1. **`beforeFullFileContent` / `afterFullFileContent`** are set to `undefined` when either side exceeds 524,288 characters. Those fields are client-only, so no model-visible lie, but a UI that renders a diff from them shows nothing and the model is unaffected.
2. **The diff itself** is skipped entirely when `omitEditDiffString` is set, in which case `diffString: ""`, `linesAdded: 0`, `linesRemoved: 0`. Again client-only.
3. **The fuzzy hint's window** is 2 lines before and needle-lines + 4 total, clipped at the ends of the file. A model reading the hint sees a region that may not include the mismatch, and there is no marker saying the region was clipped. It is labelled as a possible match, so it does not claim completeness.
4. **`linesAdded` / `linesRemoved`** are computed and stored on the record but never reach the model. The model gets the sentence.

So the answer to the question "does the tool lie about completeness anywhere" is: to the model, no. To a UI reading the proto fields without checking, three fields can be silently empty or zero, and there is no flag distinguishing "the diff was omitted because there was none" from "the diff was omitted because a flag suppressed it."

One more omission: the **success message does not say whether the fallback was used.** A model that sent a tab-indented replacement into a space-indented file has no way to learn that its replacement went in verbatim. The telemetry counter `agent.tools.str_replace.whitespace_insensitive` records it, and the model sees nothing. That is information the harness has and discards, and it is the strongest argument for appending a short excerpt to the success text.

Other harnesses. Rig marks every cut it makes (`[… N more lines]`, `and N more`) except the 300-character line cut; Cursor's did-you-mean window has no marker. Gemini CLI labels only a fuzzy match, so an exact, flexible or regex success looks the same to the model; OpenCode never says a match was fuzzy; Cline's `Edited` line never says how many replacements (always one); and a success on a changed file is silent in rig, Codex and Cursor (source, probe).

## Failure matrix

Every failure this tool can produce, in check order. `Case` is the structured result case; the model-visible column is what the renderer emits, including the `Error: ` prefix the renderer adds.

### Before any matching

| Condition | Case | Model-visible string | Layer |
|---|---|---|---|
| arguments fail schema validation | `error` | `Invalid arguments:\n<key>: <message>` | tool |
| the stream ended and `path` never completed | `error` | `Failed to acquire file lock - path may not be parsed` | tool, streaming wrapper |
| plan mode, path not markdown or canvas | `error`, remapped | the long plan-mode paragraph | tool, gate |

### Read stage, before the engine runs

| Condition | Case | Model-visible string | Layer |
|---|---|---|---|
| read returns `fileNotFound` | `fileNotFound` | `File not found` | capability service |
| read returns `permissionDenied` | `readPermissionDenied` | `Read permission denied: <path>` | capability service |
| read returns `invalidFile` | `error` | `<reason or "Path is not a valid file to read">: <path>` | capability service |
| read returns `invalidFile` for a directory | `error` | `Path is a directory, not a file: <path>` | capability service |
| read returns `rejected` | `rejected` | `Edit rejected: <reason>` | capability service |
| read returns `rejected` with `Failed to find tool call context` | `error` | the raw reason, with no prefix | tool |
| read returns `error`, message contains `Unable to resolve nonexistent file` | `fileNotFound` | that same message | tool, remapped |
| read returns `error`, message is the worktree guard sentinel | `error` | the sentinel, rethrown unchanged | tool |
| read output is not text | `error` | `File is not a text file (binary data)` | tool |
| read was truncated | `error` | `File is too large to edit (<bytes> bytes): only part of it could be read, so writing the edit would truncate it. Edit it with a shell command instead.` | tool |

### Engine

| Condition | Case | Model-visible string | Layer |
|---|---|---|---|
| `old_string === ""` | `error` | `old_string is empty` | engine |
| normalized old equals normalized new | `error` | `old_string and new_string are exactly the same` | engine |
| zero matches, exact scan only | `error` | `The string to replace was not found in the file.` | engine |
| zero matches, fallback also ran | `error` | `The string to replace was not found in the file (even after relaxing whitespace).` | engine |
| zero matches with a fuzzy hint | `error` | the not-found sentence plus ` Found a possible fuzzy match, did you mean:\n<numbered snippet>` | engine |
| more than one match, `replace_all` false | `error` | `The string to replace was found multiple times in the file. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` | engine |
| more than one match only after relaxation | `error` | `The string to replace was found multiple times in the file after relaxing whitespace. Use replace_all to replace all occurrences, or include more context to only edit one occurrence.` | engine |
| self-overlapping needle, `replace_all` false | `error` | `The string to replace was found at more than one place in the file, and those places overlap, so replace_all would edit only the first. Include more context to only edit one occurrence.` | engine |

### Write stage

| Condition | Case | Model-visible string | Layer |
|---|---|---|---|
| write returns `permissionDenied` | `writePermissionDenied` | `Write permission denied: <path>` | capability service |
| write returns `permissionDenied` with an error detail | `writePermissionDenied` | `Write permission denied: <path>: <error>` | capability service |
| write returns `permissionDenied` with `isReadonly` | `writePermissionDenied` | `You are in ask mode and cannot run non read-only tools. Ask the user to switch to agent mode if edits are required.` | renderer |
| write returns `noSpace` | `error` | `No space left on device` | capability service |
| write returns `rejected` | `rejected` | `Edit rejected: <reason>` | capability service |
| write returns `error` | `error` | `Error: <error>` | capability service |
| the result case is undefined | `error` | `Unknown error` | renderer |

### Failures that look like successes

| Condition | Case | Model-visible string | Why it looks like success |
|---|---|---|---|
| fallback matched a differently-indented region | `success` | `The file <path> has been updated.` | the model gets no signal that a relaxed match happened, or that its replacement went in at its own indentation |
| `replace_all: true` on a self-overlapping needle | `success` | `1 occurrence in file <path> has been updated.` | the count is honest but a model expecting all overlapping sites is misled by `replace_all` |
| `replace_all: true` with one match | `success` | `1 occurrence in file <path> has been updated.` | identical semantics to `replace_all: false` for one match, different text |
| the anchor matched a region the model did not mean | `success` | `The file <path> has been updated.` | uniqueness was satisfied against the wrong thing |
| a concurrent write was clobbered | `success` | `The file <path> has been updated.` | `StrReplace` writes the whole file, so a lost concurrent change is invisible |
| a lone-`\r` file was silently unmatchable | `error` | `The string to replace was not found in the file (even after relaxing whitespace).` | reported as not-found, not as an encoding problem |
| a mostly-UTF-8 file with bad bytes was lossily decoded | `success` | `The file <path> has been updated.` | the bad bytes became U+FFFD and were written back |

The last two entries stand apart from the rest: one failure misreports its cause, and one success corrupts data. Both are inherited from the read and write stages rather than produced by the engine.

## Traps

Each of these is a place a faithful-looking implementation goes wrong, with the test that catches it. Traps 1 to 11 were run against Cursor's own engine, loaded verbatim from the daemon: 28 of 32 sub-cases agree with the expectation as first written, and the ones that did not are corrected in place (Trap 1's matched line, Trap 7's two single-line cases). Rig's engine agrees on 12 of 22 of the same inputs (daemon, probe). Traps 16 to 20 come from the other harnesses.

### Trap 1: rebuilding the file from trimmed lines

**What goes wrong.** The implementation splits the file into lines, trims each one to compare, then joins the matched region back and writes. Every untouched line loses its trailing whitespace, and any line whose leading whitespace was normalized loses that too. A single-line edit to a 3,000-line file rewrites 3,000 lines.

**Why it is easy.** The trimmed-line matcher naturally produces trimmed lines, and joining them is the shortest path from "I found the region" to "I have the new file". It looks like the same operation.

**The test.** File with three lines: `"\tconst a = 1;  \n    const b = 2;\t\nconst c = 3;\n"`. The `old_string` must need the fallback, so `b = 2` will not do: it is an exact substring and never reaches the fallback (daemon, probe). Use `old_string` `"\tconst b = 2;"` and `new_string` `"    const b = 9;\t"`. Assert the result is `"\tconst a = 1;  \n    const b = 9;\t\nconst c = 3;\n"` - lines 1 and 3 keep their tabs and trailing spaces byte for byte, and the matched line becomes what the model sent, because a fallback match replaces the whole line including its own indentation and trailing tab. With `new_string` `"\tconst b = 9;"` the result is `"\tconst a = 1;  \n\tconst b = 9;\nconst c = 3;\n"` (daemon, probe). Assert the byte length of the untouched prefix and suffix separately.

This is the `zhayujie/CowAgent` commit (web) in one sentence: locate the match in the original content and return offsets into it, so only the matched region is replaced. Cursor gets it right by construction. Cursor's `pi_edit` gets it wrong on its fuzzy path, and pi's own tool did too until 2026-06-19, which proves it is possible to ship both ways.

### Trap 2: advancing the exact scan by one

**What goes wrong.** `i += 1` instead of `i += oldString.length` reports two matches for `aa` in `aaa`. With `replace_all: false` that throws the "multiple times" sentence instead of the overlap sentence. With `replace_all: true` it produces `XX` instead of `Xa`.

**Why it is easy.** Advancing by one is the standard "find all occurrences including overlapping" idiom, and a reader who has not thought about the overlap guard will consider it more correct.

**The test.** Three assertions on a file whose content is `aaa`:
- `old_string: "aa"`, `new_string: "X"`, `replace_all: false` -> the overlap sentence, not the multiple-times sentence.
- `old_string: "aa"`, `new_string: "X"`, `replace_all: true` -> file is `Xa`, message says `1 occurrence`.
- `old_string: "aa"` against `aaaa` with `replace_all: true` -> file is `XX`, message says `2 occurrences`.

The third assertion is the one that distinguishes a correct non-overlapping scan from one that simply stops after the first hit.

### Trap 3: dropping the overlap guard

**What goes wrong.** The non-overlapping scan finds one match in `aaa` for `aa`, `replace_all` is false, so the uniqueness check passes, and the edit applies to offset 0. The model intended offset 1, or intended both, and it is never told there was a second site.

**Why it is easy.** The guard is a second check that looks redundant with the multiple-match check, since both throw the same error class. A reader implementing from the error strings alone will implement one and think the other is a duplicate.

**The test.** `ababa` with `old_string: "aba"`, `new_string: "X"`, `replace_all: false` -> must throw the overlap sentence. And the same call with `replace_all: true` -> must produce `Xba`, not `XbX`, because the scan found one match and replaced it.

### Trap 4: re-indenting `new_string` to match the file

**What goes wrong.** The implementation notices the matched region was at column 4 and the model's replacement starts at column 0, so it prefixes four spaces. Or it notices the model sent tabs and the file has spaces, so it converts. Either way the model's bytes are changed.

**Why it is easy.** It feels like the helpful thing to do, and several editors' AI features do this. Three harnesses re-indent on purpose. Zed's `edit_file` has a whole `Reindenter` with per-line indent deltas. Aider adds one matched indent prefix to every non-blank REPLACE line, the older and more widely used precedent (source: Aider `editblock_coder.py:262-272`; probe: REPLACE `x = 1` / `z = 3` against `    x = 1` gave `    z = 3`). Gemini CLI's flexible tier re-indents the replacement to the first matched line and erases differences in inner indentation (probe). That makes it look like the standard approach.

**The test.** File `"function f() {\n    if (x) {\n        return 1;\n    }\n}"`. Send `old_string` with tabs for the three inner lines and `new_string` with **two** spaces of indentation. Assert the result contains `"\n  if (y) {\n    return 2;\n  }\n"` - two spaces, because that is what the model sent. The fallback matched, and the replacement is verbatim.

### Trap 5: applying the whole-file LF normalization and forgetting to restore

**What goes wrong.** The engine works on LF and the result is written back as LF. A CRLF file becomes an LF file, and every line shows as changed in `git diff`.

**Why it is easy.** The normalization is necessary and the restoration is a layer away, in the write executor. If you implement the engine and the write as two independent pieces, the restoration is easy to omit.

**The test.** File `"a\r\nb\r\nc\r\n"`. Edit `b` to `B`. Assert the file on disk is `"a\r\nB\r\nc\r\n"` - byte for byte, three `\r\n` sequences, none missing and none added.

And the mixed-ending variant: file `"a\r\nb\nc\r\nd\n"`. Edit `b` to `B`. Assert `"a\r\nB\nc\r\nd\n"` - every untouched line kept its original ending, including the bare LF on line 2.

### Trap 6: treating the empty replacement as a normal splice

**What goes wrong.** `new_string: ""` splices the match away and leaves the newline, so deleting a whole line leaves a blank line where it was. The model then issues a second edit to remove the blank line, and the turn costs twice as much.

**Why it is easy.** The general splice handles the empty string correctly as a replacement; what it does not do is look at whether anything else shares the matched line.

**The test.** File `"line1\nline2\nline3\nline4"`. `old_string: "line2"`, `new_string: ""`. Assert the result is `"line1\nline3\nline4"` - three lines, with no hole. Then the same on the last line: `old_string: "line4"`, `new_string: ""` on `"line1\nline2\nline3\nline4"` -> `"line1\nline2\nline3"`, no trailing newline, no trailing blank line.

And the counter-test: a match sharing its line with other content must not eat the line. `old_string: "b"`, `new_string: ""` on `"abc\n"` -> `"ac\n"`.

### Trap 7: assuming the fallback is more permissive than the exact scan in every respect

**What goes wrong.** The fallback is stricter in several ways, and each surprises.

- **A single-line search matches a trimmed line that equals it or starts with it, and never one that only ends with it.** For a one-line needle the first-line `endsWith` result is overwritten by the last-line `startsWith` result. The only survivor of the first-line test is the rejection `if (isFirst && ok && ft !== needle[n]) continue outer`, which refuses a line that ends with the needle without being equal to it. Run through Cursor's own engine (daemon, probe), with a tab in the needle to push it past the exact scan:

  | File line | Needle | Result |
  |---|---|---|
  | `  foo` | `\tfoo` | matches, `X` |
  | `  foo bar` | `\tfoo` (a prefix) | matches, `X bar`: column 0 to the end of the needle is replaced |
  | `  const a = 1; // note` | `\tconst a = 1;` | matches, `X // note` |
  | `  foo bar` | `\tbar` (a suffix) | not found, with a did-you-mean hint |
  | `  foo bar baz` | `\tbar` (the middle) | not found |
  | `  aXa` | `\ta` (starts and ends with it, not equal) | not found |

  `"const a = 1;"` will not match `"    const  a = 1;"` (an extra internal space). Without a tab it never reaches the fallback at all: the exact scan finds `const a = 1;` inside `const a = 1; // note` and edits it directly, `X // note`, with `usedWhitespaceInsensitive` false. Likewise `"a = 1"` against `"    const a = 1;"` succeeds immediately as a substring.

- **Internal whitespace is never collapsed.** `"foo  bar"` does not match `"  foo bar"`, even though both trim to the same shape, because the trimmed forms still differ by one space.

- **A single trailing newline in `old_string` becomes "and the next line begins as an empty prefix".** Under the fallback, a trailing `\n` produces a trailing empty needle line, whose `startsWith("")` is always true and whose `endOff` is `indexOf("") + 0 = 0`. So the match ends at column 0 of the following line, which means it *includes* the newline after the matched text but does not consume anything from the next line. Verified: `"\tb\n"` against `"a\n    b\nc\n"` matches `"    b\n"`, and replacing with `"X"` gives `"a\nXc\n"` - the replacement is glued to `c` with no separator.

- **Two trailing newlines require a blank following line.** With two, the needle has two empty lines at the end. The first is matched by the `startsWith("")` rule ending at column 0 of the next line; the second must correspond to another following line whose trimmed form is empty. So `"b\n\n"` matches `"b\n\n"` in the file and not `"b\nc\n"`.

- **A blank line in the middle of `old_string` must correspond to a blank line in the file.** No blank-line tolerance at all. `"b\n\nc"` matches a region with a blank third line, and fails against a region where `b` and `c` are adjacent.

- **A leading newline in `old_string` produces the mirror case.** The first needle line is empty, `endsWith("")` is true, and `startOff = rawLine.lastIndexOf("") = rawLine.length`, so the match begins at the end of the preceding line - i.e. on the newline itself. Verified: `"\nb"` against `"a\n    b\nc\n"` matches `"\n    b"`.

**Why it is easy.** "Whitespace insensitive" suggests a superset relation. It is not; it is a different relation with its own strictness.

**The test.** Eight assertions, all run against the shipped engine:
- `"    const  a = 1;"` in the file, `"const a = 1;"` as the needle -> not found after relaxation.
- `"const a = 1; // note"` in the file, `"const a = 1;"` as the needle -> succeeds through the exact scan, file becomes `X // note`, no relaxation flag. This is the assertion an exact-first pipeline must satisfy. Reading only the fallback rule predicts "not found", which is wrong.
- `"  const a = 1; // note\n"` in the file, `"\tconst a = 1;"` as the needle -> succeeds through the fallback as a prefix match, file becomes `X // note\n`, relaxation flag true.
- `"  foo bar\n"` in the file, `"\tbar"` as the needle (a suffix) -> not found.
- `"  foo bar"` in the file, `"foo  bar"` as the needle -> not found, because internal runs are not collapsed.
- `"\tb\n"` against `"a\n    b\nc\n"` -> matches the region `"    b\n"`, so the file becomes `"a\nXc\n"` with no separator between the replacement and `c`.
- `"\tb\n\n"` against `"a\n    b\nc\n"` -> not found, because the second empty needle line needs a blank file line.
- `"\tb\n\n\tc"` against a file where `b` and `c` are adjacent lines -> not found.

### Trap 8: getting the identity check wrong in either direction

**What goes wrong.** Two symmetric mistakes.

- Comparing the raw forms: then `old_string: "a\r\nb"` with `new_string: "a\nb"` is not an identity error, the exact scan finds `a\r\nb` after normalization, and the edit replaces `a\nb` with `a\nb`, a no-op reported as success.
- Comparing the trimmed forms: then `old_string: "  foo"` with `new_string: "foo"` becomes an identity error, and deliberate re-indentation stops working.

**Why it is easy.** "Reject a no-op" sounds like it should use the strictest comparison available. It should not; it uses the EOL-normalized comparison and nothing else.

**The test.** Three assertions:
- `old_string: "foo"`, `new_string: "foo"` -> `old_string and new_string are exactly the same`.
- `old_string: "  foo"`, `new_string: "foo"` against a file with `foo` at column 0 -> success, file becomes `foo`.
- `old_string: "  foo"`, `new_string: "    foo"` against the same file -> success, file becomes `    foo`.

The parent spec gets the second one wrong, which is why it is worth a test.

### Trap 9: normalizing the needle but not the file, or vice versa

**What goes wrong.** If only the needle's `\r\n` is collapsed and the file's is not, an LF needle against a CRLF file goes through the fallback instead of matching through the exact scan. That is mostly invisible, except: the reported `usedWhitespaceInsensitive` flips, the "after relaxing whitespace" suffix appears on errors, and the fallback's stricter rules (trap 7) apply where the exact match would have succeeded.

**Why it is easy.** The two normalizations are in different functions at different layers, with different regexes.

**The test.** CRLF file `"function f() {\r\n    return 1;\r\n}\r\n"` with an LF `old_string` of `"    return 1;\n"`. Assert the edit succeeds **and** that the code path taken was the exact one. The observable proxy is a counter or a returned flag; assert on the flag.

### Trap 10: using `/\r\n?/g` on the file content

**What goes wrong.** Collapsing lone `\r` in the file content makes a classic-Mac file look like an LF file to the matcher but not to the writer, so line endings can be lost or duplicated. Cursor uses `/\r\n/g` on the content and `/\r\n?/g` on the model strings, and the asymmetry is what makes a lone-`\r` file unmatchable across lines rather than mishandled.

**Why it is easy.** One regex is tidier than two.

**The test.** File `"a\rb\rc\r"`, `old_string: "a\nb"`, `new_string: "X"`. Cursor's behavior is not-found. If you deliberately fix the hole, assert success and assert the file is `"X\rc\r"` or `"X\r\nc\r\n"` - decide which, and pin it. What you must not do is produce `"Xc\r"` with a lost separator.

### Trap 11: forgetting that `replace_all` has no schema default

**What goes wrong.** Reading `args.replace_all` directly instead of normalizing with `=== true`. With a plain truthy test, a string `"false"` or a number `0` behaves differently. With a `Boolean` coercion, `"false"` becomes `true`.

**Why it is easy.** The description says `default false`, so a reader assumes the default is applied.

**The test.** Send `replace_all` absent -> treat as false. Send `replace_all: false` -> false. Send `replace_all: true` -> true. Send `replace_all: "true"` -> schema validation rejects it as not a boolean, so the tool never sees it; assert the refusal rather than a behavior.

### Trap 12: copying `pi_edit`'s fuzzy path wholesale

**What goes wrong.** Cursor's fuzzy path matches against a whole-file NFKC-normalized copy and splices into that copy. Every untouched line gets NFKC, quote folding, dash folding, NBSP replacement, and `trimEnd()` applied, and then the whole file is written back. Three lines change when one was intended. This is the version pi shipped until 2026-06-19.

**Why it is easy.** `pi_edit`'s array shape is better and it is tempting to take the whole implementation. The normalization looks like a feature.

**The test.** File with a curly-quoted string, a U+00A0 in a third line, and a trailing-space line. Send an edit whose needle requires the fuzzy path (an ASCII apostrophe against the curly quotes). Assert the output differs from the input **only** inside the intended region: the trailing spaces survive, the U+00A0 survives, and the curly quotes outside the region survive. Cursor's `pi_edit` fails it. Pi at HEAD passes it, because pi now copies untouched lines from the original, so this case alone no longer separates a good implementation from a bad one. Cursor's `StrReplace` has no folding tier, so it refuses the ASCII needle (`not found`) and changes nothing (daemon, probe); with the curly needle, only the intended line changes. Two more cases catch what pi HEAD still gets wrong (probe):
- **Same-line damage.** File `"x = \u2018a\u2019; y = \u2018b\u2019;   \nkeep \u2018c\u2019   \n"`, edit `x = 'a';` to `x = 'A';`. Assert that `y = \u2018b\u2019;` and its trailing spaces survive. Pi HEAD returns `"x = 'A'; y = 'b';\nkeep \u2018c\u2019   \n"`: the other lines are safe, but everything else on the touched line is normalized.
- **Whitespace-only last line.** File `"foo\nbar  \n   "` (no final newline), edit `bar\n` to `BAR\n` through the fuzzy path. Assert success and `"foo\nBAR\n   "` or a clear refusal. Pi HEAD throws `Cannot preserve unchanged lines because the base content has a different line count.`, because `trimEnd` empties the last line and the normalized copy has one line fewer; the same file is fine on the exact path. Cursor's `StrReplace` succeeds on this input but returns `"foo\nBAR\n"`, so it does not meet the first alternative: the fallback consumes the whitespace-only last line along with the match (daemon, probe).

### Trap 13: locking on the resolved path when the tool locks on the raw one, or the reverse

**What goes wrong.** If the lock key and the operation target disagree about which string is canonical, two calls that name the same file under different spellings either both take "the lock" (and interleave) or fail to take it at all.

**Why it is easy.** Both strings look like paths. Only one of them is the one the read executor will use.

**The test.** Two concurrent calls, one with `C:\work\a.ts` and one with `C:/work/a.ts`. Assert they serialize. Cursor fails this test, deliberately. If you fix it, assert the serialization explicitly rather than relying on it.

### Trap 14: assuming the tool exists on every model

**What goes wrong.** The harness registers `StrReplace` unconditionally, and a GPT-5 family model gets a tool Cursor would not have given it. Or the reverse: a harness with a per-model tool table drops `StrReplace` and the description generator hard-fails on the missing `WRITE`.

**Why it is easy.** The parent spec presents `StrReplace` as always present, and the vendor gate is one `switch` in a tool list builder that a reader focused on the matcher will never open.

**The test.** Assert the tool list for a GPT-5 family model contains `ApplyPatch` and not `StrReplace`. Assert the tool list for an Anthropic model contains `StrReplace` and `Write`. Assert that generating the description with `WRITE` absent throws, if you reproduce the `Bj` reference.

### Trap 15: letting the success text claim something the tool did not do

**What goes wrong.** `replace_all: true` with a self-overlapping needle reports `1 occurrence ... has been updated` after changing one of two overlapping sites. The sentence is accurate about what happened and misleading about what was asked. Similarly, a fallback match reports plain success with no indication that the match was relaxed or that the replacement kept the model's indentation.

**Why it is easy.** The message is generated from the match count, which is the number of splices, which is correct by construction. The gap is between "how many did I change" and "how many did the model think it was asking for".

**The test.** Assert the exact sentence for the self-overlap case is `1 occurrence in file <path> has been updated.` Then decide whether to add a clause. If you keep Cursor's text, this test is what stops someone from "fixing" the count to 2 and breaking the splice.

### Trap 16: letting a library `replaceAll` expand `$` patterns in `new_string`

**What goes wrong.** The tool replaces with a string-replacement API, and `$&`, `$$`, `` $` `` and `$'` in the model's text are treated as patterns. A model that writes code containing `$&` (a regex replacement, a shell variable, a template) gets different bytes than it sent. OpenCode's `content.replaceAll(search, newString)` does this (probe: `price: X and X` with `X` to `$&$&` gives `price: XX and XX`).

**Why it is easy.** `replaceAll` looks like the exact tool for `replace_all`, and the failure needs a `$` in the replacement to show. Cursor splices by offset and Cline replaces through a closure, which is why both pass.

**The test.** `price: X and X`, `old_string` `X`, `new_string` `$&$&`, `replace_all` true -> `price: $&$& and $&$&`. And `a a`, `old_string` `a`, `new_string` `$$`, `replace_all` true -> `$$ $$` (both run through Cursor's engine, daemon, probe). In Rust, `str::replace` has no such patterns, so the trap is only open to a port that goes through a regex crate.

### Trap 17: a did-you-mean builder that anchors on a blank line, or that hides the real difference

**What goes wrong.** Goose's hint takes the first file line where `line.contains(first_line) || first_line.contains(line.trim())`. The second clause is true for every blank line, so a miss shows the neighbourhood of the first blank line, unrelated to the search (probe). Separately, a hint built with `lines()` hides a `\r`, and a hint that prints text for an NFC against NFD mismatch looks identical to the needle (rig, Goose), so the model cannot see what to fix.

**Why it is easy.** The hint is advisory, so no test fails when it is wrong, and a bad hint costs a turn only when a model trusts it.

**The test.** A file `line one\n\nline three\nline four\n` and an `old_string` of `zzz not present` -> the error must show no hint, or a hint that shares a token with the needle. Cursor passes: its anchor is the longest non-empty trimmed line and a blank file line cannot contain it (daemon, probe). For the hidden difference, a CRLF file and a multi-line LF needle that misses by one word -> the error must not print text that is byte-identical to the needle.

### Trap 18: a tier that picks one candidate and deletes by position

**What goes wrong.** A thresholded match is accepted, and the edit then removes the lines it expected by count rather than by content (Cline `apply_patch` at 0.66), or replaces a whole window including a buffer line the model never named (Zed at 0.8). Both report success. A chain that treats two candidates as unique because their text differs by whitespace edits the first and ignores the second (OpenCode, probe: a tab line and a two-space line against `    foo()` edits the tab line and leaves the other, also with `replaceAll`).

**Why it is easy.** Each tier looks like a small relaxation of the one before, and the damage is invisible in the success text.

**The test.** A buffer where the needle omits one similar middle line -> the edit must refuse, and the omitted line must survive. A file with `if a {\n\tfoo()\n}\nif b {\n  foo()\n}\n` and `old_string` `    foo()` -> an ambiguity error and no edit. Cursor refuses with `found multiple times in the file after relaxing whitespace`, and with `replace_all` it edits both and says 2 (daemon, probe).

### Trap 19: a BOM that `read` shows and `edit` strips

**What goes wrong.** The matcher removes a leading U+FEFF before matching and restores it on write, but `read` shows it, so an `old_text` copied from the first line contains a character the matcher has already removed and misses; the closest-text hint then prints `abc` for `{BOM}abc`, which looks identical (rig, probe D2).

**Why it is easy.** Each side is correct in isolation. Runvil's `read_string` does not strip a BOM, so a Pack that matches against its output sees U+FEFF in the string (see `What Runvil can support today`).

**The test.** A file `{BOM}abc\n` with `old_string` `abc` -> succeeds and the BOM survives. And with `old_string` `{BOM}abc` copied from a read -> succeeds, or the read must not show the BOM. Strip on both sides or on neither.

### Trap 20: in-order entries where an earlier edit takes away a later anchor

**What goes wrong.** In an array applied in order on the running text, entry 0 removes or changes the text entry 1 needs, or adds a second copy that makes entry 1 ambiguous. Without a diagnosis the model is told only `not found`, and without all-or-nothing the file is left half edited (Zed).

**Why it is easy.** The two orders both look right in isolation, and each works for different models: matching against the original suits order-independent edits, in-order suits a model that reasons step by step (decision 10).

**The test.** `alpha beta` with edits `alpha` to `gamma`, then `alpha beta` to `X` -> an error naming `edits[0]` as the cause, nothing written. `A` with edits `A` to `B`, then `B` to `C` -> `C` in an in-order tool, `not found` in an original-snapshot tool; pin whichever you chose (rig, probe E1, E2).

## How the harnesses do this

Twelve harnesses are compared: Claude Code, Copilot, Zed, pi, rig, zeroclaw, Codex CLI, Gemini CLI, OpenCode, Cline, Aider and Goose. Each H3 gives the source identity, the parameters, the caps with units, an output example, and one idea to take and one to reject. Where a harness has no str-replace tool, its H3 says what it does instead. Tier labels: `(source)` code read at a pinned commit, `(probe)` code run, `(web)` documentation or an API response, `(bundle)` a shipped bundle, `(daemon)` Cursor's unminified daemon.

### Claude Code

**Source.** Claude Code 2.1.311, win32-x64 bundle: the `Edit` tool (`edit.js`), the read-state store (`read-file-state.js`) and 126 extracted error strings (`error-strings.json`), extracted with 0 anchor failures (bundle).

**Tool name.** `Edit`.

**Parameters.**

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `file_path` | string | yes | absolute; `path` is accepted as an alias and rewritten |
| `old_string` | string | yes | `old_str` accepted as an alias |
| `new_string` | string | yes | `new_str` accepted as an alias |
| `replace_all` | boolean | optional, default false | `replace_name` accepted as an alias |

The alias layer is `coerceInput`, which reports which aliases it rewrote (`alias_replace_name`, `path`, `old_str`, `new_str`). Cursor has no such layer.

**Caps.** `maxResultSizeChars: 100000`. A separate editable-file ceiling of 1 GiB (`ZWn = 1073741824`), enforced by a `stat` before the edit:

```
File is too large to edit (${size}). Maximum editable file size is ${max}.
```

There is also a perforce read-only refusal and an `isPartialView` rule.

**Output shape.** A snippet, not a sentence:

```
The file <path> has been updated successfully.
```

with, when `replace_all`:

```
The file <path> has been updated. All occurrences were successfully replaced.
```

and optional trailing notes for `userModified`, `staleRecovered`, and `contentNotInModelContext`. The period placement differs between the two branches - `has been updated. All occurrences...` versus `has been updated successfully.` - which only matters if you are diffing transcripts.

**Matching rule.** Exact `includes`, non-overlapping advance identical to Cursor's. Then quote folding on four characters (`\u2018 \u2019 \u201C \u201D`), offset-preserving. Then `\uXXXX` unescaping in the needle, and then the reverse: build a `\uXXXX` regex from literal characters and match escaped source. No whitespace fallback at all.

**Gating.** A read-before-write state machine. `readFileState` is an LRU keyed by normalized path, holding `content` (Bun.hash, base36), `timestamp`, `offset`, `limit`, `isPartialView`, and optionally `contentNotInModelContext` and `staleRecovered`. The gate is:

```
File has not been read yet. Read it first before writing to it.
```

and the staleness detectors:

```
File content has changed since it was last read. This commonly happens when a linter or formatter run via Bash rewrites the file. Call Read on this file to refresh, then retry the edit.
```

```
File has been modified since read, either by the user or by a linter. Read it again before attempting to write it.
```

There is a `staleRecovered` escape hatch: if the `old_string` still applies to the changed content, the edit is allowed to proceed and the result includes a note that the file contains other changes not in the model's context. That is a good idea, and Cursor has no analogue because it always reads fresh.

**The one design idea worth taking.** The `staleRecovered` recovery. A stale read is not always a reason to refuse: if the anchor still resolves uniquely, applying the edit is very likely correct and refusing costs a full read plus a turn. Refusing only when the anchor is gone or ambiguous is the right threshold, and it is strictly better than either "always refuse" or "never check".

**The one design idea worth rejecting.** The quote folding. Folding `‘` to `'` means a model's ASCII apostrophe silently matches a curly one in the source, and then the replacement is written with whatever the model sent. On a file that deliberately uses typographic quotes, an edit can silently typographize or de-typographize a region. Cursor's refusal to fold is the safer default, because a failed match is recoverable and a silent character substitution is not.

### VS Code Copilot

Copilot is the richest source here because it has three edit surfaces plus a repair pass, and they are all in the same directory.

**Source.** `microsoft/vscode`, `extensions/copilot/src/extension/tools/node/`, local checkout at HEAD `b208bc25` (2026-10-25) (source).

**The edit surfaces.**

| Tool | File | Shape |
|---|---|---|
| `replace_string_in_file` | `replaceStringTool.tsx` + `abstractReplaceStringTool.tsx` | `{ explanation, filePath, oldString, newString }` |
| `multi_replace_string_in_file` | `multiReplaceStringTool.tsx` | `{ explanation, replacements: [{ filePath, oldString, newString, explanation }] }` |
| `insert_edit_into_file` | `insertEditTool.tsx` | a natural-language instruction, not strings |
| `apply_patch` | `applyPatchTool.tsx` | a patch document, including `*** Delete File:` |

The names `filePath`, `oldString`, `newString` are camelCase, and `explanation` is a required field on the single-file tool's TS interface. That is a different parameter set from Cursor's.

**Parameters, single-file tool.**

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `explanation` | string | yes in the interface | Cursor has no such field |
| `filePath` | string | yes | absolute path, per the description |
| `oldString` | string | yes | "MUST be the exact literal text" |
| `newString` | string | yes | - |

**The description** is the most prescriptive edit description in the set:

```
This is a tool for making edits in an existing file in the workspace. For moving or renaming files, use run in terminal tool with the 'mv' command instead. For larger edits, split them into smaller edits and batch them with multi_replace_string_in_file in a single call. Before editing, always ensure you have the context to understand the file's contents and context. To edit a file, provide: 1) filePath (absolute path), 2) oldString (MUST be the exact literal text to replace including all whitespace, indentation, newlines, and surrounding code etc), and 3) newString (MUST be the exact literal text to replace `oldString` with (also including all whitespace, indentation, newlines, and surrounding code etc.). Ensure the resulting code is correct and idiomatic.). Each use of this tool replaces exactly ONE occurrence of oldString.

CRITICAL for `oldString`: Must uniquely identify the single instance to change. Include at least 3 lines of context BEFORE and AFTER the target text, matching whitespace and indentation precisely. If this string matches multiple locations, or does not match exactly, the tool will fail. Never use 'Lines 123-456 omitted' from summarized documents or ...existing code... comments in the oldString or newString.
```

"at least 3 lines of context BEFORE and AFTER" is an unusual instruction: it pushes toward larger anchors than Cursor does, and it is the opposite of Claude Code's newer "keep `old_string` minimal, usually 1-3 lines" guidance. Two products giving opposite advice on anchor size is a signal that neither has measured it.

**Caps.** No explicit character or line cap on the edit itself. Two relevant thresholds:

- `NoChangeError` when `updatedFile === originalFile`
- `trySimilarityMatch` skips when `oldStr.length > 1000`, when `oldStr` has more than 20 lines, or when the file has more than 1000 lines

The output is capped by the renderer's token budget, passed as `tokenizationOptions ?? { tokenBudget: 5000, countTokens: (t) => Promise.resolve(t.length * 3 / 4) }`.

**Matching rule.** Four strategies in order, in `findAndReplaceOne`:

1. `tryExactMatch` - non-overlapping `indexOf` scan, same as Cursor's.
2. `tryWhitespaceFlexibleMatch` - split on `eol`, trim each line, and append a `''` to the needle with the comment `// trailing newline to match until the end of a line`. Then a window comparison on the joined trimmed lines.
3. `tryFuzzyMatch` - build a regex where each line is escaped literally and followed by `[ \t]*\r?\n`, so only trailing whitespace and newline style are flexible.
4. `trySimilarityMatch` - a Levenshtein similarity with a default threshold of `0.95`, applied per line in a sliding window, only for needles under 1,000 characters and 20 lines and files under 1,000 lines.

So Copilot has *four* strategies where Cursor has one fallback, and the fourth has a similarity threshold. This is a counterexample to any claim that nobody uses thresholds: Microsoft ships 0.95, and it is not the only one. Cline's `apply_patch` has a 0.66 similarity tier (fuzz factor 1000) and then deletes the hunk's `-` lines by position, so a similar-looking block elsewhere is replaced and the only signal is `Note: Patch applied with fuzz factor 1000` (source: Cline `apply-patch-parser.ts:404-415`, `apply-patch.ts:162-193`, tag v4.1.26; probe). Gemini CLI's fourth tier is a bounded fuzzy Levenshtein: windows of the same number of lines, a weighted score at or under 0.1, a 10-character floor on `old_string`, and a hard cost bound of 400,000,000 on lines times characters squared. It is the only tier that tells the model what it did (`Applied fuzzy match at line(s) S-E.`) (source: Gemini CLI `edit.ts:68-70,1404-1542`, v0.66.0). Zed ships a 0.8 per-line version and OpenCode's block-anchor replacer uses 0.65. So five of the twelve other harnesses ship a live thresholded tier (Aider's is disabled), and what separates them is whether it can delete by position (Cline), skip lines (Zed), or only substitute (Gemini).

The whitespace strategy has a detail Cursor does not: it appends an empty string to the needle so that a search matches "until the end of a line", and it computes `startIdx` / `endIdx` from line positions via `OffsetLineColumnConverter` with `endIdx = convert.positionToOffset(end) - 1` and the comment `// -1 to include the last EOL`. It also trims `oldStr` and `newStr` before splitting (`oldStr.trim().split(eol)`), which Cursor never does.

**Healing.** `editFileHealing.tsx` is a repair pass that runs when the first attempt produced `NoMatchError` and the model is flagged as needing healing. It tries, in order: unescape the needle (the Gemini over-escaping bug), then ask a `copilot-utility-small` endpoint to produce the corrected `oldString` given the file content, then ask it to adjust `newString` to match the corrected `oldString`. Both LLM calls have JSON schemas and both fall back to the original on any failure. There is also `trimPairIfPossible`, which trims the needle if the trimmed form matches the expected number of times and trims `newString` to match.

**Output shape.** `EditFileResult` rendered from `fileResults`, with per-file entries holding `operation`, `uri`, `isNotebook`, `existingDiagnostics`, `error`, and `healed`. The unified diff is streamed separately into the chat as a markdown code block with a `diff:` language tag and a `<vscode_codeblock_uri>` line. So Copilot's model-visible result includes diagnostics and a heal note, and Cursor's is one sentence.

**The one design idea worth taking.** `_errorConflictingEdits`. When two edits in one multi-replace call land on the same file, it sorts all their `textEdits` by start position and refuses if any two overlap:

```js
const hasOverlap = allEdits.some((e2, i) => {
  if (i === 0) { return false; }
  const e1 = allEdits[i - 1];
  return !e1.range.end.isBeforeOrEqual(e2.range.start);
});
```

with the error `Edit at index ${i} conflicts with another replacement in ${path}. You can make another call to try again.` Cursor's `pi_edit` also checks overlap (`edits must not overlap.`), and Cursor's `StrReplace` cannot have the problem because it takes one edit. The idea is worth taking because an array tool without an overlap check silently produces nonsense: two edits whose ranges touch can double-apply or lose text depending on splice order.

**The one design idea worth rejecting.** The similarity threshold. 0.95 on a per-line Levenshtein is high enough that it mostly catches whitespace and case differences, which the earlier strategies already handle, and low enough that it can match a line the model did not mean. It also has a quadratic cost per window. Cursor's decision not to score similarity is the safer one, and the Aider history (a `difflib.SequenceMatcher` ratio of 0.8 over whole blocks, shipped and then disabled behind a bare `return`) is the evidence (source: Aider `editblock_coder.py:183-185,296-329`, tests disabled by a `__test_` prefix and the comment `fuzzy logic disabled v0.11.2-dev`).

### Zed

**Source.** Zed `crates/agent/src/tools/edit_file_tool.rs` (121,530 bytes), `edit_session.rs` (43,656), `edit_session/streaming_fuzzy_matcher.rs`, `edit_session/streaming_parser.rs`, `edit_session/reindent.rs`, the `streaming_diff` crate and `tool_permissions.rs`, at commit `37faef81f7` (2026-10-27, clean tree); sizes counted with LF endings (source). Line cites below are to that commit. The older checkout this report was first written against, `d9ad6aff67` (2026-08-22), is reconciled under `### Zed checkout drift`.

**Tool name.** `edit_file`. There is no `StrReplace`, `replace_in_file` or `apply_patch` equivalent: the `tools!` list has `EditFileTool` and `WriteFileTool` and nothing else that edits text (`tools.rs:199-224`).

**Parameters.**

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `path` | `PathBuf` | yes | the description says to start with a project root or use `~/.agents/skills`; that is advice. The code also accepts an absolute path inside a worktree and a bare worktree-relative path (`find_project_path`, test `edit_file_tool.rs:1436-1455`) |
| `edits` | `Vec<Edit>` | yes | `deserialize_maybe_stringified`, so a JSON string containing the array is also accepted |

and each `Edit`:

| Field | Type | Required | `.describe()` in the doc comment |
|---|---|---|---|
| `old_text` | string | yes | "The exact text to find in the file. This will be matched using fuzzy matching to handle minor differences in whitespace or formatting.", then "Be minimal with replacements:" with two bullets ("For unique lines, include only those lines", "For non-unique lines, include enough context to identify them") (`edit_session.rs:47-52`) |
| `new_text` | string | yes | "The text to replace it with" |

The `Edit` type itself is described as "A single edit operation that replaces old text with new text. Properly escape all text fields as valid JSON strings. Remember to escape special characters like newlines (`\n`) and quotes (`"`) in JSON strings." There is one description and one schema for every model, and it tells the model to strip the `read_file` line-number prefix (`{line_number:>6}\t`) from anything it copies. Wrong field names fail with serde's bare message, for example ``missing field `old_text` `` (test `edit_file_tool.rs:3172-3182`).

So `edit_file` is already an array-of-edits tool, like `pi_edit` and Copilot's `multi_replace_string_in_file`, but unlike them it is not atomic (see below).

**Caps.**

| Cap | Value | Unit | Layer |
|---|---|---|---|
| edits per call, `old_text` and `new_text` length | none | - | not enforced |
| exact matches collected | 2 | entries | matcher stops at the second hit (`MAX_EXACT_MATCHES`, `streaming_fuzzy_matcher.rs:9`) |
| fuzzy line similarity, window acceptance | 0.8 and 0.8 | ratio per line; aligned query lines over the larger side | matcher (`:253-256`, `:369-378`) |
| file load ceiling | 6 GiB | bytes, `metadata.len >= FILE_SIZE_MAX`, message `File is too large to load` | worktree loader (`worktree.rs:1721-1726`) |
| binary sniff window | 1,024 | bytes | `FILE_ANALYSIS_BYTES`; binary gives `Binary files are not supported` |
| unified diff context | 3 | lines | `unified_diff` |

Loading refuses files of 6 GiB or more and binary files, so "no cap" is true only for the edit itself. The permission gate is `tool_permissions.rs`.

**The edit session, which is the whole design.** `EditSession::new` resolves the path to a `ProjectPath`, opens a buffer, and then:

```rust
let file_changed_since_last_read = ensure_buffer_saved(&buffer, &abs_path, mode, &context, event_stream, cx).await?;
```

`ensure_buffer_saved` compares the action log's `file_read_time(abs_path)` against the buffer's `disk_state().mtime()`, and if they differ it sets `file_changed_since_last_read`. It also handles a dirty buffer by prompting the user with `Save` or `Discard`. A third option, `Keep`, exists only in the overwrite prompt that `write_file` uses (`tool_permissions.rs:795-860`).

Edits are then processed **as they stream in**, before the call completes. `process_streaming_edits` loops over `ToolInput::Partial` payloads and feeds each `PartialEdit` into `session.process_edit`, which runs the events through `EditPipeline::process_event`. The partial-JSON diffing that makes this work is `edit_session/streaming_parser.rs`, and the character-level alignment that turns a replace into minimal inserts and deletes is the `streaming_diff` crate. So the buffer is edited incrementally while the model is still emitting `new_text`, and the UI shows a live diff through a `Diff` entity. The next edit builds a new matcher over the buffer as the previous edit left it, so **edits apply sequentially and are not atomic**: if edit N fails, edits 0 to N-1 stay applied and the buffer is still saved (test `test_streaming_edit_failure_mid_stream`, `edit_file_tool.rs:1236-1319`).

The old text resolution is `StreamingFuzzyMatcher`, a different matcher:

- `finish()` first tries `find_exact_matches`, a KMP scan over the rope that **detects overlapping matches on purpose**, with `MAX_EXACT_MATCHES = 2` because "two matches are enough to prove ambiguity". Zed counts overlapping occurrences, so `aaaa` against `aaaaa` yields two ranges `[0..4, 1..5]` and is reported as ambiguous. That is the opposite of Cursor's scan.
- If there is no exact match, it runs a line-level dynamic program over `(query_line, buffer_line)` pairs. Both lines are `.trim()`med before comparison. Costs are `REPLACEMENT_COST = 1`, `INSERTION_COST = 3`, `DELETION_COST = 10`, with `fuzzy_eq` as a normalized-Levenshtein check at `THRESHOLD = 0.8` per line, behind a length prefilter.
- It accepts a window only if `matched_ratio >= 0.8`, where `matched_ratio = line_pairs.len() / max(matched_buffer_row_count, query_line_count)`. That ratio allows a query to **skip buffer lines**: the test `test_line_pairs_skip_unmatched_buffer_line` shows a four-line query matching buffer rows `3, 5, 6, 7` with `self.extra` at row 4 skipped, and Zed's own test asserts the skipped row is deleted (`edit_file_tool.rs:402-470`; probe: a five-line query with one unrelated line is accepted at ratio 0.8, a four-line query is rejected at 0.75).
- `LINE_HINT_TOLERANCE = 200` exists, but the shipped tool never supplies a hint: both `matcher.push` calls pass `None` (`edit_session.rs:482,510`), so every ambiguous match is an error. The proximity tiebreak is reached only by a unit test.

So Zed has fuzzy matching with a similarity threshold and line-skipping, and a proximity tiebreak that is dead code in the shipped tool.

**Re-indentation.** `Reindenter` with separate `first_line_delta` and `rest_delta`, computed from `compute_indent_delta(buffer_indent, query_indent)` and `compute_rest_indent_delta(...)`. It adjusts the leading whitespace of each line of `new_text` by the delta between the buffer's indentation and the query's, clamping at zero, and choosing spaces versus tabs based on whether the buffer's indent has any tabs. So Zed *does* re-indent, deliberately, and it has the tests to prove the edge cases (outdent tabs, clamp negative, whitespace-only lines pass through verbatim, a distinct first-line delta when the model stripped only the first line's indent). It also applies to exact matches: for a single-line exact `old_text` inside an indented line, the continuation lines of `new_text` get the buffer line's indent, with `first_line_delta` forced to 0 (`edit_session.rs:547-569`, test `edit_file_tool.rs:329-399`). `strip_trailing_newline` is separate: it removes one `\n` from the end of `new_text` when the matched range is followed by a newline (`edit_session.rs:571,648-652`); it is not line-ending handling.

**Failure text.** Two strings name the edit index (0-based); the path, permission, binary and dirty-buffer failures do not:

```
Could not find matching text for edit at index {}. The old_text did not match any content in the file.{} Please read the file again to get the current content.
```

with `{}` filled by `" The file has changed on disk since you last read it."` when the mtime check fired, and:

```
Edit {} matched multiple locations in the file at lines: {}. Please provide more context in old_text to uniquely identify the location.
```

That second one lists line numbers (1-based), which neither Cursor nor Claude Code does. For exact matches the scan stops at two hits, so it lists at most two lines and cannot give a count; fuzzy ties list every window. It is a better hint than Cursor's for a model that must choose where to add context, but it does not tell the model how many places there are.

**Output shape.** The model sees `output.to_string()` only (`edit_session.rs:130-134`): `Edited {path} successfully`, or `No edits were made.` when the diff is empty (empty `edits`, `old_text == new_text`, or a change that only adds or removes the final newline). On error the text is `{error}\nNo edits were made.`, or `{error}\nEdited {path}:` followed by a fenced unified diff when earlier edits did apply. `old_text` and `new_text` (the whole files) go to `raw_output`, which the type documents as "often for debugging or extra state for replay"; the model does not see them. The success text does not say that a fuzzy window was used, that skipped lines were replaced, that `new_text` was re-indented, or that format-on-save rewrote other lines. Composed from the `Display` code and the fields Zed's test pins (`edit_file_tool.rs:1310-1317`), the partial-failure result is:

````
Could not find matching text for edit at index 1. The old_text did not match any content in the file. Please read the file again to get the current content.
Edited root/file.txt:

```diff
@@ -1,3 +1,3 @@
-line 1
+MODIFIED
 line 2
 line 3

```
````

**Other behavior that changes a design.** Line endings: the loader turns `\r\n` and lone `\r` into `\n`, `LineEnding::detect` reads the first `\n` in the first 1,000 bytes, and save writes every `\n` with that one ending, so a mixed file is rewritten uniformly. On a Windows build a file with no `\n` in its first 1,000 bytes is saved as CRLF (`worktree.rs:7399-7425`, `text.rs:3621-3661`, derived). Encoding: a BOM selects the encoding, plain UTF-8 streams in, and anything else goes through `chardetng` and `encoding_rs`, re-encoded on save, so invalid bytes become U+FFFD (`worktree.rs:7272-7312`, `file_content.rs:89-145`). Format-on-save runs on every edit call for languages that enable it, so formatter changes appear in the result diff (`edit_session.rs:186-214`). There is no tool-level lock; the serialization point is the `Buffer` entity.

**The one design idea worth taking.** Staleness as a diagnosis, never a gate. Zed records the disk mtime at each read and each agent edit, compares it only when an edit fails to match, and adds one sentence to the error (`edit_session.rs:971-976`). An edit whose `old_text` still matches succeeds after an external change (test `edit_file_tool.rs:2322-2402`), and an edit with no prior read succeeds (`:302-326`). The model loses no turn on a harmless stale read and gets the right explanation when the match fails. Runvil can copy it with a map from path to `SystemTime` or a content hash from `Access::stat`, written by Read and by a successful StrReplace and consulted only on the zero-match path. Confidence: high, because it costs one map and one string and closes the "why did this fail" gap.

Two smaller ideas are worth taking in a stateless design. First, **naming the indices and line numbers in the failure text**: `Edit 2 matched multiple locations in the file at lines: 41, 97.` is more useful than `The string to replace was found multiple times in the file.` and costs one line of code, provided you also count all matches so the model can be told how many. Second, the live-buffer design: **the edit applies to a live buffer, not to bytes just read**, so a user's open editor sees the change and a user can interrupt. Whether the language server also sees it is not verified here: the LSP sync path was not read, and the tool result has no diagnostics field.

**The one design idea worth rejecting.** Accepting a fuzzy window and replacing the whole window. Zed accepts a window when 80 percent of the larger side aligns, then replaces the entire window including buffer lines the model never mentioned, and reports plain success, which is the failure mode Aider's maintainer cited when disabling fuzzy matching: it let the model silently delete semantically significant lines. The deletion is pinned by Zed's own test and by a probe. Cursor's fallback refuses unless trimmed lines match one to one, which cannot drop a line. Confidence: high. A runner-up reject is the non-atomic multi-edit with a save after a partial failure: it works for Zed because the diff in the error shows what landed, and a tool without that diff would leave the model guessing. Confidence: medium, since it depends on whether the result has a diff.

The re-indentation is a closer call. Zed's version is careful and well tested, and it produces better-looking diffs when a model copies a fragment without its leading indent. But it also means the model's bytes are not the bytes that land, which is a class of surprise Cursor's design does not have. Take Zed's re-indentation only if you also return the changed region in the result, so the model can see what it got. Without that, verbatim insertion is safer. Confidence: medium, since both the careful test suite and the silent-rewrite risk are real.

### pi

**Source.** pi `packages/coding-agent/src/core/tools/edit.ts` and `edit-diff.ts`, with `file-mutation-queue.ts`, at commit `a92283056` (2026-10-27, clean tree), package `@earendil-works/pi-coding-agent` 1.4.0. The `packages/durable` twin has the same matcher. History of the matcher: fuzzy matching `0c135d014` (2026-01-14), NFKC `700bcf345` (2026-03-15), multi-edit `20a57e759`, `edits[]` as the only shape `e773527b3` (2026-03-28), and `128330e36` (2026-06-19, "preserve untouched lines in fuzzy edit", closes issue 5899, released in pi 0.79.9) (source). Cursor's own `pi_edit` is described under `### The pi_edit counterpart`; this section is the one account that compares the two.

**Pi's original against Cursor's port.** The whole-file normalized write-back that the contract section calls a trap is a version pi outgrew. Pi's tool wrote the whole normalized file back until `128330e36`; at HEAD it is line-confined, which is better but not clean. Cursor's port is not a copy of any pi revision (none of its four distinct error strings is in any pi commit) and has not picked up the fix. I sliced the pure matcher out of pi HEAD, out of `128330e36^`, and out of Cursor's daemon, and ran the same inputs through all three (probe).

| Question | pi before `128330e36` | pi at HEAD | Cursor `pi_edit` |
|---|---|---|---|
| fuzzy path writes back | the whole normalized file, by design (a code comment says so) | only the lines the matches touch; other lines are copied from the original by index (`edit-diff.ts:132-173`) | the whole normalized file |
| text on a touched line but outside the match | normalized | normalized (the whole touched line is rewritten) | normalized |
| whitespace-only last line, no final newline, fuzzy edit | succeeds | throws `Cannot preserve unchanged lines because the base content has a different line count.` | succeeds, last line consumed |
| characters folded | NFKC, U+2018 to U+201B, U+201C to U+201F, U+2010 to U+2015, U+2212, and the space family | same | NFKC, U+2018/9, U+201C/D, U+2013/4, U+00A0; nine characters pi matches fail |
| `newText` normalized on the fuzzy path | no | no | yes |
| uniqueness counted in | normalized space, even for an exact hit | same | exact count first, normalized only on zero |
| line ending chosen | the first ending in the file | same | `\r\n` if any `\r\n` exists |
| error text | names the path and the edit index | same | short, no path, no index |

The input from the contract section, with the three results side by side (probe):

```
content : "const a = 1;   \nconst s = \u2018quoted\u2019;\nconst n = 1\u00a0+ 2;\ntarget();\n"
edit    : oldText "const s = 'quoted';"  newText "const s = 'renamed';"
pi@HEAD   : "const a = 1;   \nconst s = 'renamed';\nconst n = 1\u00a0+ 2;\ntarget();\n"
pi@parent : "const a = 1;\nconst s = 'renamed';\nconst n = 1 + 2;\ntarget();\n"
cursor    : "const a = 1;\nconst s = 'renamed';\nconst n = 1 + 2;\ntarget();\n"
```

Pi's fix is pinned by tests with untouched lines that end in trailing spaces (`tools.test.ts:1325,1340`); both come out right at HEAD and wrong at the parent. Two things pi HEAD still gets wrong: a line that holds the match is normalized end to end (`x = 'a'; y = 'b';` loses its curly quotes), and the last-line crash above, which gives the model no hint that an exact `oldText` with the trailing spaces would work. Which side holds the bug depends on the question: the whole-file rewrite is Cursor's, line endings are both sides' (neither preserves a mixed file), and the last-line crash is pi's only (probe).

**Parameters.**

| Parameter | Type | Required | Description (pi's own text) |
|---|---|---|---|
| `path` | string | yes | `Path to the file to edit (relative or absolute)` |
| `edits` | array | yes | `One or more targeted replacements. Each edit is matched against the original file, not incrementally. Do not include overlapping or nested edits. If two changes touch the same block or nearby lines, merge them into one edit instead.` |
| `edits[].oldText`, `edits[].newText` | string | yes | `Exact text for one targeted replacement. It must be unique in the original file and must not overlap with any other edits[].oldText in the same call.` and `Replacement text for this targeted edit.` |

Cursor's port keeps the field names and the `oldText` and `path` descriptions word for word, and cuts `edits` to `One or more targeted replacements.`. Pi also repairs arguments before validation (`prepareEditArguments`, `edit.ts:103-134`): a JSON string holding the array, a string holding one edit object, one bare edit object, and top-level `oldText`/`newText` are all accepted. Cursor has no counterpart (searched the daemon).

**Caps.** None found: no file size cap, no edits-per-call cap, no `oldText` length cap. The diff context is 4 lines and feeds only `details`. Matches and lengths are UTF-16 code units; the mutation queue allows one active edit or write per realpath in-process (`file-mutation-queue.ts:32-61`). `Found N occurrences` can report a huge N: 8 MiB of `e` with needle `e` gives `Found 8388608 occurrences` after building a split array that large (probe).

**Output shape.** One sentence, pinned by tests (`edit-tool-legacy-input.test.ts:87`), with a literal `(s)`:

```
Successfully replaced 1 block(s) in legacy.txt.
Could not find edits[1] in f.txt. The oldText must match exactly including all whitespace and newlines.
Found 3 occurrences of the text in f.txt. The text must be unique. Please provide more context to make it unique.
```

**One idea to take.** Confine the fuzzy write-back to the lines the match touches, and prove it with a bytes-outside test: match in normalized space, widen each replacement to its lines, rewrite only those from the normalized base, copy every other line from the original (about 120 lines with helpers). Take it with two repairs: do not throw when the line counts differ, and consider narrowing from lines to the matched span. Confidence: medium, because the idea is sound and tested but the shipped version crashes on the last line and writes whole lines. Since Runvil recommends no folding tier at all (decision 2), the practical take is the test.

**One idea to reject.** One detected line ending applied to the whole file on every edit. `detectLineEnding` picks the first ending, a lone `\r` becomes `\n`, and `restoreLineEndings` rewrites every `\n`: on `a\nb\r\nc\r\n` an edit to `b` returns `a\nB\nc\n`, losing the CR on two untouched lines, on the plain exact path (probe). It undermines the guarantee the June fix added. Confidence: high, because untouched bytes change on the exact path.

### rig

**Source.** Rig `crates/rig-tools/src/edit.rs` (904 lines, 32,681 bytes) and `edit/tests.rs` (seven tests) at commit `a83b774d0` (2026-10-25, "feat(rig-traffic): record every provider HTTP exchange", checkout matches the commit). The tool is named `edit`, one name and no per-model variants. The engine is the public pure function `rt::edit::apply(path, text, edits)`, and the tool is a thin read, apply, write wrapper (`rt/edit.rs:175-214, 398-400`) (source). History: created 2026-10-15 matching against the original; switched to in-order application on the running text 2026-10-19 (`c2b6c3923`, "triobench round 4: edits apply in order"); reply changed twice (`e034438fa`, `bfc4efca6`). The benchmark data is not in the repo, so the reason for the switch is not in the sources.

**Parameters.** Closed schema (`deny_unknown_fields` plus `additionalProperties: false`), at least one edit (`minItems: 1`):

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | yes | `The file to edit.` (a `HOST:PATH` hint is appended when remote hosts are configured) |
| `edits` | array, `minItems` 1 | yes | `The replacements, in order: each matches the text the ones before it left.` |
| `edits[].old_text` | string | yes | `The text to replace, copied from the file.` |
| `edits[].new_text` | string | yes | `The replacement.` |
| `edits[].replace_all` | boolean | no, default false | `Replace every match instead of exactly one.` |

**Caps.** File size 16 MiB (bytes; `{path} is {N} MB; the tools open files up to 16 MB`). No cap on `old_text`, `new_text`, number of edits or matches. Ambiguity lists 8 line numbers then `and {N} more`; the closest-text hint shows 20 lines, each cut to 300 characters; the reply shows 3 lines of context, stops starting new runs at 60 shown lines, and shows a run over 30 lines as its first 15 and last 15 (at most 89 file lines). A diff deadline of 100 ms. No timeout on the tool itself.

**Matching.** Per entry, in order, on the running text. Exact first (`match_indices`, non-overlapping, so `aa` in `aaa` is one match and passes, with no overlap guard). If there is no exact match and `replace_all` is off, a loose pass compares whole lines after `trim_end` plus a fold of quotes (U+2018 to U+201B, U+201C to U+201F), dashes (U+2010 to U+2015, U+2212) and spaces; it needs exactly one place and says so in the reply. Leading whitespace, inner runs, case and Unicode normal forms are not touched, so tab against spaces fails with `nothing in the file is close to it` (probe). A failed entry does not stop the loop, so one reply names every failure; nothing is written unless all entries apply. `old_text == new_text`, an empty `old_text`, and entries that cancel out are refused.

**Stale protection.** `Seen` holds the SHA-256 of what `read` showed and, within an 8 MiB total text budget, the text itself. `edit` applies to the cached text and calls `write_if(path, bytes, &hash)`: `None` means the file still held the cached text and the bytes are written; `Some(current)` means it did not, nothing was written, and `current` becomes the text to edit. The slow path then re-applies the same entries to `current` and writes unconditionally. A miss after a change says `... changed since it was last read, so read it again and copy old_text from what it holds now`; a hit on a changed file says nothing (probe: appended line kept, edit applied, no note). The compare and the write are separate steps, so an outside write between them is lost (probe S7, S8). Decision 4 states what this means for Runvil.

**Output shape.** Changed line numbers, then the changed lines with context, no diff markers (tests `rt/edit/tests.rs:80-177`):

```
Edited {path}: lines 2-4, line 10 (+1 line).
Edited {path}: line 11.
Edited {path}: 3 replacements, lines 1, 3, 5.
```

A miss, composed from the code and pinned for its fragments (`rt/edit/tests.rs:212-227`):

```
edits[1]: `old_text` was not found, not even ignoring trailing whitespace; copy it from the closest text in the file, shown below. No edit was applied, so `{path}` was not changed. Send all 2 edits again, with this one fixed.
The closest text to the `old_text` of edits[1] is line 2:
    let total = add(1, 2);
```

**One idea to take.** The reply that shows the result, built from the same pure `apply`: a first line with the changed lines and a bounded excerpt, plus the closest-text hint on a miss. Rig reached it in three rewrites in four days. For Runvil: a Pack-level `apply(&str, &[Edit]) -> Result<Applied, EditError>` returning text, change and notes. Confidence: medium, since the mechanism is pinned by tests but the benchmark that motivated it is not in the repo.

**One idea to reject.** All-or-nothing CRLF detection and the loose pass a mixed file falls into. A file is CRLF only when every `\n` has a `\r`; one stray bare LF makes it mixed, and then a multi-line `old_text` that crosses a CRLF boundary cannot match because the model's text is LF. It falls to the loose pass, which replaces the region with LF and drops the `\r` of the last matched line (probe C6, C7, C11). The fix is per line: match on LF-normalized text and give each written line its own original ending. Confidence: high on the defect, medium on how often it bites. Smaller rejects: no overlap guard, `line_of` computed for every match of an ambiguous needle (quadratic: 512 KiB of `a` lines takes 110 s), and a hint that prints text that looks identical when the difference is NFC against NFD (probe).

### zeroclaw

**Source.** zeroclaw `crates/zeroclaw-tools/src/file_edit.rs`, workspace version 0.8.5, commit `2d85371f3` (2026-09-18; the checkout is about five weeks behind, newer commits are not in the sources). The tool is `file_edit`, one name and no per-model variants, wrapped in a rate limiter and a path guard, and not auto-approved (source).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `path` | string | yes | `Path to the file. Relative paths resolve from workspace; outside paths require policy allowlist.` |
| `old_string` | string | yes | `The exact text to find and replace (must appear exactly once in the file)` |
| `new_string` | string | yes | `The replacement text (empty string to delete the matched text)` |

**Caps.** None in the tool: no size limit on the file it reads (unlike `file_read`'s 10 MiB), no cap on strings. The agent loop cuts any result over 50,000 bytes; a default budget of 20 successful actions per sender per hour applies to every wrapped tool, reads included.

**Matching and uniqueness.** `content.matches(old_string).count()`, then `replacen(old_string, new_string, 1)`, then a whole-file write (`file_edit.rs:219,239-241`). `str::matches` does not count overlaps, so `aa` in `aaa` counts as one, the uniqueness check passes, and the first two bytes are replaced, giving `Xa` (probe). The same happens for `aba` in `abab`. There is no identity check: `old_string == new_string` rewrites the file and reports success. There is no fallback, but a failed match is re-tried under a weaker rule for the message only: both strings have leading whitespace stripped per line, and the count of the stripped needle picks the text (0, 1, or n).

**Output shape.** Reply `Edited {path}: replaced 1 occurrence ({n} bytes)`, where `n` is the size of the whole new file rather than the change. Failures (probe through the verbatim `no_match_diagnostic`):

```
Error: old_string not found in file
Error: old_string not found exactly: a block matching it ignoring leading whitespace exists exactly once. The difference is indentation (width, or tabs vs spaces). Re-read the target region and copy its leading whitespace exactly, then retry.
Error: old_string matches 2 times; must match exactly once
```

**One idea to take.** A no-match message that names the class of mistake by re-matching under a weaker rule: the model gets "indentation only" or "ambiguous under whitespace" instead of a bare not-found. It is cheap, pure and unit-testable (`file_edit.rs:260-286`, tests `442-465`). Confidence: high, since it is reproduced on the shipped helper. **One idea to reject.** Using `str::matches(...).count()` as the uniqueness proof, and a diagnostic that cannot see CR: overlapping candidates slip through as unique, and a CRLF file is reported as an indentation problem because `lines()` hides the `\r` on both sides (probe). Confidence: high.

### Codex CLI

**Source.** `openai/codex` at commit `bea47c06dfac` (committed 2026-10-28T05:46Z, workspace version `0.0.0`, so no release number); read from raw files at that sha, with the removed tools read at the parents of PRs 15773, 15775 and 21170 (web, source). **Codex has no str-replace tool.** Its model-visible file tools are `exec_command` (shell: `cat`, `rg`, `rm`), the freeform `apply_patch`, and `view_image`; three dedicated file tools (`read_file`, `grep_files`, `list_dir`) existed behind an experimental flag and were removed. Every model in `models.json` gets `apply_patch` as a `freeform` Lark-grammar tool. What this implies: Codex trusts the model with a shell for reads and one patch grammar for every write, and the gate is the OS sandbox plus an approval policy; there is no per-tool check.

**The nearest mechanism.** An `*** Update File: <path>` hunk. There is no old-string and new-string pair, no `replace_all`, and no uniqueness requirement; the unit is whole lines.

| Element | Syntax | Meaning |
|---|---|---|
| file | `*** Update File: <path>` | the file must exist |
| rename | `*** Move to: <path>` | before the first chunk only |
| anchor | `@@` or `@@ <line>` | one line searched from the cursor, which then moves past it |
| context, removal, addition | a leading space, `-` or `+` | must match and is kept; must match and is deleted; inserted |
| end anchor | `*** End of File` | the old lines must match at the end of the file |

**Caps.** 512 MiB read per file (`512 * 1024 * 1024` bytes); the result shown to the model is capped at 10,000 approximate tokens (4 bytes each). Patch size, hunk count and files per patch are not in the sources.

**Matching.** Four passes over the range, strictest first: exact, trailing whitespace trimmed, both ends trimmed, then both ends trimmed with dashes U+2010 to U+2015 and U+2212, curly quotes and wide spaces folded. Strictness wins over position, and within a pass the first position at or after the cursor wins, with no check that it is the only one (`seek_sequence.rs:35-108`). Only the changed lines are replaced; context lines stay as the original text, but a trim-matched line takes the model's indentation (probe). Every hunk is verified against the current files before any write, then derived again from disk at apply time; after that hunks apply one file at a time with no transaction. CRLF, LF and bare CR are recognized per line, untouched lines keep their endings, and inserted lines use the first ending in the file.

**Output shape.** The block formatted as shell output (`apply_patch_cli.rs:459-467`), grouped Added, Modified, Deleted whatever the patch order:

```
Exit code: 0
Wall time: 0.1 seconds
Output:
Success. Updated the following files:
A nested/new.txt
M modify.txt
D delete.txt
```

Verification failure: `apply_patch verification failed: Failed to find expected lines in <absolute path>:` then the old lines the model sent, joined and uncapped.

**One idea to take.** Verify every hunk against the current files before any write, then derive again from disk at apply time: a bad hunk anywhere means zero writes (`apply_patch_cli.rs:1471-1489`), and a stale file is re-matched instead of overwritten from a cached copy. Confidence: high, since it gives all-or-nothing for the common failure without a hash store. **One idea to reject.** Fuzzy matching that takes the first hit and writes the model's text over the file's text: of two identical lines the first is edited silently, and a trim match drops the file's indentation (probe). Confidence: high, both look like success.

### Gemini CLI

**Source.** `google-gemini/gemini-cli` tag `v0.66.0`, commit `970413207732`, `packages/core/src/tools/edit.ts` plus `utils/textUtils.ts`, `tools/diff-utils.ts` and `utils/pathCorrector.ts` (source). Tool `replace` (display name Edit). Two schema sets exist (`gemini-3` and `default-legacy` for every other model); parameter names are identical and descriptions differ.

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `file_path` | string | yes | `The path to the file to modify.` |
| `instruction` | string | yes | a semantic description of the change for an LLM assistant; Cursor has no such field |
| `old_string` | string | yes | `The exact literal text to replace, unescaped.` The legacy text asks for at least 3 lines of context before and after |
| `new_string` | string | yes | the exact replacement; placeholders such as `(rest of methods ...)` are refused unless the same placeholder is in `old_string` |
| `allow_multiple` | boolean | no, default false | `If true, the tool will replace all occurrences of old_string. If false (default), it will only succeed if exactly one occurrence is found.` |

**Caps.** Fuzzy tier skips `old_string` under 10 characters (UTF-16 code units), accepts a weighted distance over search length of at most 0.1, and skips when `lines * old_string.length^2 > 400,000,000`. Tool output over 64 KiB is cut by the scheduler.

**Matching.** Four tiers, first hit wins: exact (`content.split(search).length - 1`, non-overlapping, so `aa` in `aaa` is 1 and gives `Xa`; replacement through `safeLiteralReplace`, so `$&` lands literally); flexible (windows of lines after `trim()`, then the replacement is re-indented to the first matched line); regex (tokens joined with `\s*`, anchored at a line start); then bounded fuzzy Levenshtein. Differences in inner indentation are erased in the flexible tier (probe). File and model strings are normalized to LF, and if the file held any `\r\n` the whole result is expanded to CRLF, so a mixed file becomes uniform. An empty `old_string` creates the file when it is absent. A self-correction pass that asks a model to repair the edit exists and is off by default.

**Output shape.** From `edit.ts:1063-1086`:

```
Successfully modified file: <abs> (1 replacements). Here is the updated code:
...
line 14
line 15 CHANGED
line 16
...
```

(the snippet is five lines of context; the example is shortened). A flexible, regex or exact success is not labelled; only fuzzy adds `Applied fuzzy match at line(s) S-E.` On a CRLF file the snippet echoes the whole file, because the helper compares LF-normalized and CRLF-expanded text (probe). Failures: `Could not find an exact match for 'old_string' in '<file_path>'. If previous edits modified the file or you are modifying lines outside your recent read window, please use ReadFile to inspect the target lines before retrying with an exact 'old_string'.` and `Failed to edit, Expected 1 occurrence but found <n> for old_string in file: <file_path>. If you intended to replace multiple occurrences, set 'allow_multiple' to true.`

**One idea to take.** A graded matcher that tells the model when it guessed, with a hard cost bound; pair it with the lesson that every non-exact tier should be reported, the flexible and regex tiers included. Confidence: medium, because the value depends on how often models send near-misses, which is not measured. **One idea to reject.** Silent path retargeting: a relative path that does not exist is replaced by a unique same-basename file elsewhere in the tree, and the success text shows only the absolute path (`pathCorrector.ts:49-81`). Confidence: high, since read and write do not do it, so one path means different files in different tools.

### OpenCode

**Source.** `anomalyco/opencode` (formerly `sst/opencode`) tag `v1.19.5`, commit `810c0eba1a73`, `packages/opencode/src/tool/edit.ts` (the set the session loop uses), with the V2 rewrite in `packages/core/src/tool/edit.ts` and the dev branch at `fa1260d6ca` for the next generation (source). Three implementations coexist and a verdict about OpenCode has to name which one. For GPT-family models (`gpt-` but not `oss` or `gpt-4`) the model gets `apply_patch` and no `edit` or `write`.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `filePath` | string | yes | `The absolute path to the file to modify` |
| `oldString` | string | yes | `The text to replace` |
| `newString` | string | yes | `The text to replace it with (must be different from oldString)` |
| `replaceAll` | boolean | no, default false | `Replace all occurrences of oldString (default false)` |

The description opens with the same sentence as Cursor's (`Performs exact string replacements in files.`), promises errors the code does not raise, and adds a "must Read first, will error" rule with no check behind it.

**Caps.** No file size check; the reply is cut at 2,000 lines or 51,200 bytes by the generic wrapper. Block-anchor similarity 0.65 for one candidate and for several (mean of per-line `1 - distance / maxLen` over the middle lines), a line delta of `max(1, floor(0.25 * searchLines))`, and a disproportion guard that refuses a span of `>= max(oldLines + 3, oldLines * 2)` lines.

**Matching.** `replace()` runs nine replacers in order: `SimpleReplacer`, `LineTrimmedReplacer`, `BlockAnchorReplacer`, `WhitespaceNormalizedReplacer`, `IndentationFlexibleReplacer`, `EscapeNormalizedReplacer`, `TrimmedBoundaryReplacer`, `ContextAwareReplacer`, `MultiOccurrenceReplacer` (`edit.ts:694-704`). Two are exact and seven are fuzzy, and all nine are live at v1.19.5. They were tightened, not disabled: both thresholds are 0.65, the disproportion guard throws `Refusing replacement because the matched span is much larger than oldString...`, and tests pin the rejections. Only the V2 tool is exact-only (with a TODO to port "edit fuzzy parity"), and the dev branch replaced the chain with exact, then a one-to-one Unicode fold, then trailing whitespace, keeping offsets. Uniqueness is tested on the text of each candidate span (`indexOf === lastIndexOf`), not on the number of candidates, so two spans that differ in whitespace are each unique and the first wins (probe: a tab line and a two-space line against `    foo()` edits the tab line, no ambiguity error, also with `replaceAll`). `aa` in `aaa` is refused as multiple matches, and with `replaceAll` gives `Xa`. `content.replaceAll(search, newString)` expands `$&` and `$$` in the replacement (probe: `$&$&` on `price: X and X` gives `price: XX and XX`). Any `\r\n` makes the file CRLF and the replacement is forced to CRLF, including in a mixed file. The per-path semaphore is held across the permission prompt, and the file is not re-read after approval.

**Output shape.** One line, plus LSP errors for the edited file:

```
Edit applied successfully.

LSP errors detected in this file, please fix:
<block>
```

Failures: `Could not find oldString in the file. It must match exactly, including whitespace, indentation, and line endings.` and `Found multiple matches for oldString. Provide more surrounding context to make the match unique.`

**One idea to take.** The dev matcher: exact first, then a one-to-one Unicode fold, then per-line trailing whitespace, every tier keeping offsets so the splice lands on the original bytes, with the description telling the model which retries exist. The one-to-one rule is what keeps offsets valid. Confidence: medium, since it is on the dev branch and not measured. **One idea to reject.** The nine-replacer chain in the release build: it resolves ambiguity silently, loosens matches without telling the model, expands `$` patterns under `replaceAll`, and its own description promises errors the code does not raise. Confidence: high, reproduced by probe.

### Cline

**Source.** `cline/cline` tag `v4.1.26`, commit `a459f1d` (2026-10-25), `sdk/packages/core/src/extensions/tools/executors/editor.ts` (268 lines) and `apply-patch.ts`, `apply-patch-parser.ts`; the legacy `replace_in_file` line is tag `v4.0.12` (`f267bf4`) (source). Two mechanisms chosen per model: `editor` for most models, and `apply_patch` (the Codex format) for `openai-native`, `gpt*` and `codex*` models; never both.

| Parameter (`editor`) | Type | Required | Description |
|---|---|---|---|
| `path` | string, min 1 | yes | `The absolute path for the action to be performed on` |
| `old_text` | string, optional | no | `Exact text to replace (must match exactly once). Omit this when creating a missing file or inserting via insert_line. Keep this at or below 6000 characters when possible...` |
| `new_text` | string | yes | the replacement, or the new file content, or the inserted text |
| `insert_line` | integer, optional | no | a positive one-based boundary; `insert_line` plus `old_text` silently ignores `old_text` |

**Caps.** 6,000 characters each for `old_text` and `new_text` (UTF-16 units; a hard error despite the word "recommended"), 200 diff lines in the result, 30,000 ms timeout, and a second cut of 8,000 characters per string applied when the result is sent to the provider. `apply_patch` has a similarity floor of 0.66.

**Matching.** `editor`: normalize both strings to the file's detected ending (CRLF if `\r\n` appears anywhere), `occurrences = content.split(needle).length - 1`, zero or more than one throws, exactly one does `content.replace(old, () => new)`. The closure is load bearing: without it `$&`, `$'` and `$$` in `new_text` expand (test `editor.test.ts:266-283`; probe: `$& $$ $1` landed literally). `aa` in `aaa` counts as one. `apply_patch`: four tiers per hunk (exact after `canonicalize`, `trimEnd`, `trim`, then Levenshtein similarity at 0.66, with NFC and smart-quote folding in `canonicalize`), and hunks are deleted by position, not by re-comparing the `-` lines, so a similar block elsewhere is replaced; the only signal is `Note: Patch applied with fuzz factor 1000` (probe). Writes are sequential and not atomic. Two parallel edits of one file both succeeded and one update was lost (probe).

**Output shape.**

````
Edited /tmp/x/example.txt
```diff
-2: b
+2: B
```
````

Failures: `Editor operation failed: No replacement performed: text not found in <abs>.` and `Editor operation failed: No replacement performed: multiple occurrences of text found in <abs>.` The legacy `replace_in_file` failure echoed the whole original file back in `<file_content>`.

**One idea to take.** Normalize the model's text to the file's detected ending before counting matches, then count non-overlapping occurrences and replace through a closure. Reads strip `\r`, so the model's view is always LF; counting first would make every CRLF file unmatchable. Confidence: high, both behaviors are pinned by tests and reproduced. **One idea to reject.** The 0.66 Levenshtein tier that deletes by position: the `-` lines are not re-verified and the only signal is a bare integer. Confidence: medium, the position-based deletion is source-confirmed, but no case was built where it picked a wrong location.

### Aider

**Source.** `Aider-AI/aider` commit `5dc9490bb35f` (2026-05-22, `__version__` `0.86.3.dev`), `aider/coders/editblock_coder.py` and `editblock_prompts.py`; all 80 `aider/**/*.py` files outside `website/` were searched, so "not found" claims cover the package (source). **Aider has no str-replace tool and no function schema for file work.** The model writes SEARCH/REPLACE blocks in its reply and Aider parses them, matches and writes. The format is chosen per model (of 333 explicit entries in the model table: 289 `diff`, 35 `diff-fenced`, 4 `whole`, 4 `udiff`, 1 `architect`). What this implies: a human chooses the files and the model only edits them, which removes read gates, search tools and staleness hashes and moves those problems onto the human and onto prompt text.

| Element | Form | Rule |
|---|---|---|
| file path | line before the fence | decoration stripped; the previous block's path is reused if missing |
| head, divider, end | `<<<<<<< SEARCH`, `=======`, `>>>>>>> REPLACE` | 5 to 9 marker characters each |
| SEARCH text | whole lines | empty means create or append |
| REPLACE text | whole lines | empty means delete the matched lines |

**Caps.** `max_reflections = 3` retries; filename fuzzy match at 0.8 against chat files; the did-you-mean hint triggers at 0.6 over lists of lines with 5 lines of context. The fuzzy chunk matcher (`similarity_thresh = 0.8`) is unreachable.

**Matching.** A ladder, first hit wins: whole-line equality; then `replace_part_with_missing_leading_whitespace` (remove the common minimum indent from SEARCH and REPLACE, find a window whose lines match after `lstrip()` and share one identical indent prefix, add that prefix to every non-blank REPLACE line); then retries without a spurious leading blank line; then `...` elision; then a bare `return` at `editblock_coder.py:183` so the fuzzy code below it never runs. That disabled matcher is `difflib.SequenceMatcher(None, chunk, part).ratio()` over windows of whole lines, not a Levenshtein distance; the name `replace_closest_edit_distance` and the maintainer's phrase "Levenshtein-style" are where that label comes from (source: `coders/editblock_coder.py:296-316`). First match wins and nothing checks for a second (`a\na\na\n` with SEARCH `a\n` gave `X\na\na\n`), and each block is written before the next is tried, so a reply with one bad block leaves the file partly edited. The matching is by whole lines; SEARCH `foo\n` does not match the line `foobar`.

**Output shape.** What the model sees for one failed block with a near miss (probe, unmodified function):

````
# 1 SEARCH/REPLACE block failed to match!

## SearchReplaceNoExactMatch: This SEARCH block failed to exactly match lines in a.txt
<<<<<<< SEARCH
def b():
    x = 1
    return 3
=======
def b():
    return 4
>>>>>>> REPLACE

Did you mean to match some of these actual lines from a.txt?

```
def a():
    return 1

def b():
    x = 1
    return 2
```

The SEARCH section must exactly match an existing block of lines including all white space, comments, indentation, docstrings, etc
````

With one good block and one bad block it also says `# The other 1 SEARCH/REPLACE block were applied successfully.` then `Don't re-send them.`

**One idea to take.** The failure text: it echoes the model's own block, shows the nearest lines in the file, says whether the REPLACE text is already present, and reports which blocks applied, so a near miss becomes a retry the model can answer from the message alone. Confidence: high, from the probe output and the maintainer's reasoning on issue 306 (web: fuzzy whole-block matching "allows GPT to silently delete some important lines"). **One idea to reject.** First-match-wins with no ambiguity error, per-block writes with no atomicity, and a truthiness test on the result: an edit that empties a file is reported as a failure. Confidence: high, each is reproduced by a probe.

### Goose

**Source.** `aaif-goose/goose` (formerly `block/goose`) commit `ee8500d5e060` (2026-10-28), `crates/goose/src/agents/platform_extensions/developer/edit.rs` (handler `file_edit_with_cwd`, matcher `string_replace`); tag v1.56.0 differs only by the removal of a `read` path (source). The developer extension exposes exactly `write`, `edit`, `shell`, `tree` and `read_image`, so Goose has a str-replace tool but no read, delete, glob or grep tool. The `text_editor` tool with `str_replace` was deleted on 2026-02-26 (PR 7466).

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `path` | string | yes | no `~` expansion, no confinement |
| `before` | string | yes | not `old_string`; the description is `Edit a file by finding and replacing text. The before text must match exactly and uniquely. Use empty after text to delete.` |
| `after` | string | yes | a model trained on other harnesses must adapt |

**Caps.** `NO_MATCH_PREVIEW_LINES = 20` lines shown on a miss; ambiguity reports two sites with one line of context each way. No file size cap, no match cap: `content.match_indices(before).collect()` stores every match position before the count is checked, so `before = "a"` on a huge file of `a` allocates one entry per character.

**Matching.** Exact only. `match_indices` is non-overlapping, so `aa` in `aaa` is one match and the edit gives `Xa`. Zero matches gives `No match found for the specified text.` plus an optional `Did you mean:` block plus a numbered preview of the first 20 lines; one match does `replacen`; two or more give the ambiguity report. The count is checked before the write, so an ambiguous edit leaves the file untouched. No identity check: `before == after` rewrites and answers success. Byte-exact line endings: a multi-line LF `before` cannot match a CRLF file, and the hint and preview use `lines()`, which hides `\r`, so the text looks identical to the model's.

**The Did-you-mean hint is broken.** `find_similar_context` takes the first line of `before`, trimmed, and returns the neighbourhood of the first file line where `line.contains(first_line) || first_line.contains(line.trim())`. Because `first_line.contains("")` is always true, any blank line matches, so when no earlier line contains the search text the hint is the neighbourhood of the first blank line, unrelated to the search (`dev/edit.rs:198-210`; probe, below). This is Goose's hint, and it is the failure Cursor's builder is not subject to: Cursor's anchor is the longest non-empty trimmed line of the needle and a blank file line cannot `include` a non-empty string (daemon, probe). Cursor's second pass has its own weakness: any file line containing any token of three or more characters is accepted, so a common word can show an unrelated region.

**Output shape.** A miss in `line one\n\nline three\nline four\n` for `zzz not present`, with the unrelated hint caused by the blank line (probe):

````
No match found for the specified text.

Did you mean:
```
line one

line three
line four
```

File preview:
```
   1: line one
   2: 
   3: line three
   4: line four
```
````

Ambiguity (`x` twice): `Found 2 matches. Please provide more context to identify a unique match:` then `Match 1 (line 1):` and `Match 2 (line 2):` with fenced context and, for three or more, `...and 1 more`. Success: `Edited {path as given} ({a} lines -> {b} lines)`, where the counts are the sizes of the two snippets rather than of the file.

**One idea to take.** The failure payload: a numbered preview on a miss, and an ambiguity that names the line numbers of two sites with a line of context each way and an exact count, so the model can extend its anchor in one retry; the count check runs before any write (test `dev/edit.rs:338-353`). Confidence: high, tiny, deterministic and test-pinned. **One idea to reject.** The `Did you mean` heuristic as written, and `match_indices(...).collect()` before the count check for a hostile file. What the broken hint means for Runvil: a hint is part of the stale-protection and uniqueness story, because the not-found error is the moment a changed file or a wrong anchor shows up (Zed and rig both append their stale note only on a miss). A hint that can point at the wrong region, or hide a CRLF or NFD difference, defeats the diagnosis it is there for. Confidence: high, reproduced by probe.

### Comparison across all harnesses

Three tables with the same rows, split at seven columns. Cursor repeats as the first column of the second and third so each reads alone. A cell says `not in the sources` where the notes found nothing.

| Dimension | Cursor `StrReplace` | Claude Code `Edit` | Copilot `replace_string_in_file` | Zed `edit_file` | pi `edit` | rig `edit` |
|---|---|---|---|---|---|---|
| parameter shape | `path`, `old_string`, `new_string`, `replace_all` | `file_path`, `old_string`, `new_string`, `replace_all`, plus an alias layer | `explanation`, `filePath`, `oldString`, `newString` | `path`, `edits[]` of `{old_text, new_text}` | `path`, `edits[]` of `{oldText, newText}` | `path`, `edits[]` of `{old_text, new_text, replace_all}` |
| multi-edit in one call | no (a `MULTI_STR_REPLACE` id exists, unregistered) | no | yes, via `multi_replace_string_in_file` | yes, sequential and not atomic | yes, atomic against the original | yes, in order on the running text, all or nothing |
| paging | none; refuses a truncated file | none; 1 GiB editable ceiling | none | none | none | none |
| cap and unit | 8 MiB read truncation (UTF-16 units); 100,000 read chars not checked | 100,000 result chars; 1 GiB file (bytes) | none on the edit; 1,000 chars and 20 lines to try similarity | none on the edit; 6 GiB load ceiling (bytes) | none | 16 MiB file (bytes); no cap on strings or edits |
| matching rule | exact, then line-trimmed block, strict 1:1 lines | exact, then quote folding, then `\uXXXX` both ways | exact, whitespace-flexible, trailing-space regex, 0.95 similarity | exact (overlap-detecting KMP), then line DP at 0.8 per line with line skipping | exact, then whole-file NFKC fold, written back per touched line | exact, then whole-line loose key (`trim_end`, quote, dash, space folds), unique only |
| uniqueness and overlap | non-overlapping scan plus an overlap guard | non-overlapping, no guard | non-overlapping, no guard; multi-call checks range overlap | overlapping scan, so overlap is ambiguity | counted in normalized space; `aa` in `aaa` passes | `match_indices`, no guard; lists up to 8 lines |
| identity check | EOL-normalized old equals new | raw `old_string === new_string` | `oldString === newString`, then `NoChangeError` | none; a no-op renders `No edits were made.` | `No changes made` | refused, plus entries that cancel out |
| encoding | lossy UTF-8 fallback to latin1; BOM preserved | utf8, utf16, latin1 with BOM | the editor's document decoding | BOM, UTF-8, else `chardetng`; re-encoded on save | UTF-8; invalid bytes become U+FFFD; BOM restored | strict UTF-8; BOM stripped before matching |
| line endings | normalize to LF, restore per line for mixed | normalize to the document's EOL | `oldString.replace(/\r?\n/g, eol)` to the document's | LF in the buffer, one ending restored on save; lone `\r` becomes `\n` | first ending wins; lone `\r` to `\n`; mixed homogenized | only an all-CRLF file is converted; mixed edited as is |
| stale-read protection | none; prompt text only | LRU read state with hash and mtime; `staleRecovered` | none | mtime note on a miss only; dirty-buffer prompt | none; in-process queue per realpath | SHA-256 `write_if` compare-and-swap, re-apply on a mismatch |
| gating | none; vendor gate on whether the tool exists | hard read-before-write | confirmation for sensitive paths | permission prompt for `.zed`, outside paths | none | none shipped |
| re-indentation of the replacement | none, verbatim | none | none | yes, `Reindenter`, also on exact matches | none | none, verbatim |
| output format | one sentence plus a client-only diff | `The file X has been updated successfully.` plus notes | `EditFileResult` with diagnostics and a streamed diff | `Edited X successfully` or `No edits were made.` | `Successfully replaced N block(s) in X.` | `Edited X: lines 2-4 (+1 line).` plus changed lines |
| failure text names the site | no | no | no | yes, line numbers (at most two for exact) | edit index and count | yes, up to 8 lines plus the closest text |

Cursor with Claude Code, Copilot, Zed, pi and rig: the matching rule and the stale-read protection are the rows that most change a design.

| Dimension | Cursor `StrReplace` | zeroclaw `file_edit` | Codex `apply_patch` | Gemini CLI `replace` | OpenCode `edit` | Cline `editor` |
|---|---|---|---|---|---|---|
| parameter shape | `path`, `old_string`, `new_string`, `replace_all` | `path`, `old_string`, `new_string` | a patch document | `file_path`, `instruction`, `old_string`, `new_string`, `allow_multiple` | `filePath`, `oldString`, `newString`, `replaceAll` | `path`, `old_text`, `new_text`, `insert_line` |
| multi-edit in one call | no | no | yes, verified whole before any write | no | no | no (parallel calls instead) |
| paging | none | none | none | none | none | none |
| cap and unit | 8 MiB read truncation (UTF-16 units) | none; 50,000-byte result cut | 512 MiB read (bytes); 10,000-token result | fuzzy skips under 10 chars; cost bound 400,000,000; 64 KiB output | none; 2,000 lines or 51,200 bytes reply | 6,000 chars per string; 200 diff lines |
| matching rule | exact, then line-trimmed block | exact only; indentation diagnostic in the message | four passes: exact, rstrip, trim, Unicode fold; first position wins | exact, flexible, regex, bounded fuzzy at 0.1 | nine replacers, seven fuzzy, in order | exact once; `apply_patch` has four tiers to 0.66 |
| uniqueness and overlap | non-overlapping plus a guard | `str::matches().count()`; `aa` in `aaa` passes | none; first match wins | `split` count; `aa` in `aaa` passes | `indexOf === lastIndexOf` on the candidate text | `split` count; `aa` in `aaa` passes |
| identity check | EOL-normalized old equals new | none; rewrites and reports success | n/a | `No changes to apply` | `No changes to apply` | not in the sources |
| encoding | lossy UTF-8 fallback to latin1 | strict UTF-8 | UTF-8 only; a BOM blocks line 1 | UTF-8; BOM kept; invalid bytes become U+FFFD | non-fatal UTF-8; invalid bytes become U+FFFD | UTF-8; invalid bytes become U+FFFD |
| line endings | normalize to LF, restore per line | none; byte-exact | per line preserved; inserted lines use the first ending | any CRLF makes the whole file CRLF | any CRLF forces the replacement to CRLF, mixed too | CRLF if any, applied to the matched span only |
| stale-read protection | none | none | re-derive from disk at apply time | none (a hash only in an off-by-default pass) | none in release; `writeIfUnchanged` in V2 | none |
| gating | none | approval, path guard | sandbox plus approval policy | policy: ask, or allow in auto-edit | permission `edit`; lock held across the prompt | approval policy |
| re-indentation of the replacement | none | none | none; a trim match takes the model's indentation | yes, in flexible, regex and fuzzy tiers | yes, `IndentationFlexibleReplacer` | none in `editor` |
| output format | one sentence plus a client-only diff | `Edited X: replaced 1 occurrence (N bytes)` | `Success. Updated the following files:` | `Successfully modified file` plus a 5-line snippet | `Edit applied successfully.` plus LSP errors | `Edited X` plus a diff capped at 200 lines |
| failure text names the site | no | the count only | the old lines the model sent | no; fuzzy names its lines | no | no |

Cursor with zeroclaw, Codex, Gemini CLI, OpenCode and Cline; the Codex column is `apply_patch`, its nearest mechanism, because Codex has no str-replace tool.

| Dimension | Cursor `StrReplace` | Aider SEARCH/REPLACE | Goose `edit` |
|---|---|---|---|
| parameter shape | `path`, `old_string`, `new_string`, `replace_all` | text blocks in the reply, no tool call | `path`, `before`, `after` |
| multi-edit in one call | no | yes, many blocks, each written as it applies | no |
| paging | none | none | none |
| cap and unit | 8 MiB read truncation (UTF-16 units) | 3 retries; no size cap | 20-line miss preview; no size cap |
| matching rule | exact, then line-trimmed block | whole-line equality, then a uniform indent offset, then `...` elision; the `difflib` fuzzy tier is disabled | exact only |
| uniqueness and overlap | non-overlapping plus a guard | first match wins, no ambiguity error | `match_indices`; `aa` in `aaa` passes |
| identity check | EOL-normalized old equals new | not in the sources | none; rewrites and reports success |
| encoding | lossy UTF-8 fallback to latin1 | decoded `str`; an undecodable file crashes the failure text | strict UTF-8 |
| line endings | normalize to LF, restore per line | matched as LF, written with the platform ending, so a CRLF file changes wholly | none; byte-exact |
| stale-read protection | none | none; commits dirty files first | none |
| gating | none | the human chooses the files; permission prompts per file | tool-name permission mode |
| re-indentation of the replacement | none, verbatim | yes, one matched prefix on every non-blank line | none |
| output format | one sentence plus a client-only diff | no success text; the failure text echoes the block with a near-miss hint | `Edited X (a lines -> b lines)` |
| failure text names the site | no | the nearest real lines | yes, two sites with line numbers and a 20-line preview |

Cursor with Aider and Goose; Aider has no tool call, only text blocks in the reply.

The matching rule and the stale-read protection are the two rows where the twelve harnesses disagree most. There is no consensus to copy, only considered positions.

### Zed checkout drift

The Zed claims in this report were checked against the canonical checkout at commit `37faef81f7` (2026-10-27, clean tree): of 50 claims, 27 held, 10 were thin, 12 were wrong or misleading and 1 is unverifiable, and I re-read the load-bearing cites in that source myself (source). The report was first written against an older checkout at `d9ad6aff67` (2026-08-22). The rows below are claims that changed and behavior that changed between the two commits (source, probe: `git diff --no-index` over the five edit files and `git log d9ad6aff67..37faef81f7` for those paths).

| Claim as first written | What it said | What is true at `37faef81f7` |
|---|---|---|
| multi-edit is atomic | "the atomic all-or-nothing array ... is what Copilot's `multi_replace_string_in_file` and Zed's `edit_file` both do" | Zed applies edits in order to the live buffer; edits before a failing one stay applied and the buffer is saved (`edit_file_tool.rs:1236-1319`) |
| `path` must start with a project root | stated as a rule | advice only; an absolute path inside a worktree and a bare worktree-relative path are also accepted (`project.rs:5541-5581`, test `edit_file_tool.rs:1436-1455`) |
| dirty-buffer prompt | `Save / Discard / Keep` | edit mode offers `Save` and `Discard`; `Keep` exists only in the overwrite prompt of `write_file` (`tool_permissions.rs:795-860`) |
| ambiguity resolved by `line_hint` | `LINE_HINT_TOLERANCE = 200` gives a proximity tiebreak | the shipped tool passes `None` at both `matcher.push` calls (`edit_session.rs:482,510`), so there is no tiebreak and every ambiguity is an error |
| failure strings | "three, all of which name the index" | two name the edit index (0-based): not found and ambiguous; the path, permission, binary and dirty-buffer failures do not |
| ambiguity message | "tells the model exactly how many places and where" | for exact matches the scan stops at two hits, so it lists at most two lines and gives no count (`streaming_fuzzy_matcher.rs:9,357-361`) |
| result shape | the model gets full file contents in the proto | the model gets `Display` text only; `old_text` and `new_text` go to `raw_output`, "often for debugging or extra state for replay" (`edit_session.rs:130-134`) |
| line endings | "the buffer's, with `strip_trailing_newline` handling" | LF in the buffer; the first `\n` in the first 1,000 bytes picks the ending written on save; `strip_trailing_newline` is separate (`worktree.rs:7399-7425`, `text.rs:3646-3661`) |
| read-before-write gate | mtime comparison plus a dirty-buffer prompt | no gate: the mtime only adds a sentence to a not-found error and no prior read is required (`edit_session.rs:971-976`, tests `edit_file_tool.rs:302-326,2322-2402`) |
| caps | "no cap" | none on the edit; loading refuses files of 6 GiB or more and binary files (`worktree.rs:1721-1726,7283-7288`) |
| encoding | "the buffer's own decoding" | BOM, then plain UTF-8, then `chardetng`, re-encoded on save (`worktree.rs:7272-7312`) |
| matcher detail | similarity threshold, line skipping | both lines are trimmed before comparison and a length prefilter precedes the Levenshtein check; Zed's own test pins that a skipped buffer row is deleted (`edit_file_tool.rs:402-470`) |
| re-indentation | applies when the match was fuzzy | also applies to exact matches, with `first_line_delta` forced to 0 (`edit_session.rs:547-569`) |
| live language server and diagnostics | stated as fact | not verified: the LSP sync path was not read, and the tool result has no diagnostics field |
| **drift** `StreamingFuzzyMatcher::refresh` | absent | added by `0849124bc2` (2026-10-06): an existing matcher is refreshed when the buffer version changed, so a match is re-resolved when the edit finalizes; fewer stale-range failures |
| **drift** tool-call location | `line` | `line + 1`, ACP numbers from 1 (`23973bd49f`, 2026-10-26); UI only |
| **drift** ACP types, `Diff::finalized` | v1 types, eager diffs | v2 types (`7fb155e9a2`, 2026-10-09), lazy diffs (`9e65fc7682`, 2026-10-21); not model-visible |
| **drift** `reindent.rs`, `streaming_parser.rs` | - | unchanged between the two commits |

The older checkout stores every line with CRLF, so raw byte deltas misstate the change; counted as LF, `edit_file_tool.rs` grew by 4,287 bytes and `edit_session.rs` by 80 (probe).

## What Runvil can support today

The `Access` surface as it exists, in `crates/runvil-internal/vfs/src/handle/operations.rs`:

| Behavior the report specifies | `Access` method | Status | What the gap needs |
|---|---|---|---|
| read the whole file as text | `read_string` | covered | fails as `NotUtf8 { valid_up_to }` rather than lossy-decoding, which is a better default than Cursor's |
| read the whole file as bytes | `read` | covered | - |
| write the edited result back | `write` | covered | whole-file truncate-in-place; see the atomicity note below |
| exact single-occurrence replacement | `str_replace` | **covered, and then some** | rejects an empty anchor before any I/O; answers `Anchor { path, anchor, count }` for 0 or >1 |
| exact replacement of every occurrence | `str_replace` | **missing** | `Access` has no `replace_all`. Either a new capability or the Pack reads, splices, and writes |
| line-trimmed block fallback | none | **missing** | has to live somewhere above the VFS unless the VFS grows a match policy |
| verbatim insertion of `new_string` | n/a | n/a | a consequence of the splice, not an `Access` call |
| preserve untouched bytes | `str_replace` (partial) | **partial** | the backend's `str_replace` may or may not be offset-based; the Pack's own splice definitely is |
| Cursor's exact error strings | none | **missing** | `Anchor` holds `count`, which is enough to generate all three sentences, but nothing generates them today |
| line-ending detection and restoration | none | **missing** | no `stat`-level line-ending info in `crates/runvil-internal/vfs/src/stat.rs`; the Pack would have to detect from `read_string` output and restore by rewriting |
| mixed-ending per-line merge | none | **missing** | same; this is the `At` algorithm and it is a pure function, so it could live in the Pack |
| BOM detection and preservation | none | **missing** | `read_string` strips nothing, so a BOM arrives as U+FEFF in the string; the Pack would match against it, which is a behavior difference from Cursor |
| binary detection | none | **missing** | no NUL scan, no extension list. A binary file read as `read_string` fails as `NotUtf8`, which is arguably better than a separate binary path |
| file-too-large refusal | none | **missing** | no size check before read; `stat` could supply it |
| plan-mode gating | the handle's `gate(Op::Write, path)` | **covered** | the VFS already refuses a non-markdown write in Plan mode, so the Pack does not need `x_e` |
| ask-mode refusal | `gate(Op::Write, ...)` | **covered** | answers `PermissionDenied` or `ApprovalRequired`, which the Pack maps to a sentence |
| exclusivity against a concurrent writer | `admit(Claims::claim_write, ...)` | **partial** | the claim is per-access and epoch-ordered, which is stronger than Cursor's per-turn set, but it is still in-process |
| content compare-and-swap | none | **missing** | no hash parameter on `write`; a fresh implementation would need `write_if(path, expected_hash, contents)` or a `read`-then-`write` with a claim held across both. Rig's shape is `Workspace::write_if(path, bytes, &held) -> io::Result<Option<Contents>>`: `None` means written, `Some(current)` hands back what the file holds so the tool re-applies without a second read (`rt/workspace.rs:97-118`, rig `a83b774d0`) |
| pure apply with a typed result | none | **missing** | rig's public `rt::edit::apply(path, &str, &[Edit]) -> Result<Applied, ToolExecutionError>`, with the notes and the changed lines in `Applied`, is the model for the Pack-level `apply` of decision 1; a dry-run plugin and a benchmark reuse it with no I/O |
| per-path read record | none | **missing** | rig's `Seen` (`rt/read.rs:65`) maps a path to a SHA-256 and, within an 8 MiB budget, the text; Zed keeps only an mtime in its `ActionLog`. A map keyed by canonical path next to the Pack is enough, written by Read and by a successful StrReplace |
| changed-region excerpt | none | **missing** | rig builds its reply from the `similar` crate with a 100 ms diff deadline and 3 context lines (`rt/fs.rs:309-315`); the same crate would serve decision 5's Option B |
| symlink policy | backend-defined | **partial** | `str_replace` goes through the backend, which may follow or refuse; the Pack cannot see which |

`Access` never touches the disk directly. Every method follows the same four-step shape: `gate(Op::X, path)`, then `admit(Claims::claim_x, &path)`, then `fire(Op::X, &path)`, then `self.inner().x(&path, ...)`. So a new capability has a template to copy, and a Pack that does its own I/O would be breaking a rule the whole crate is built around.

`Access::str_replace` already refuses an empty anchor, before gating, with a comment explaining that the store view validates the path before the anchor. That means the existing VFS has already made the decision that an empty anchor is an argument error rather than a not-found, which is Cursor's decision too, so the Pack does not need to re-check emptiness if it delegates.

The gap that matters most is not in the table. It is that `str_replace` takes `&str, &str, &str` and returns `Result<(), VfsError>`, so it cannot report **which** region it replaced, **how many** matches there were on success, or whether the match needed relaxation. Every one of those is needed for Cursor's success text and for the telemetry counter. So even in the delegating design, the Pack has to run its own matcher to produce the message, which is the argument for Option A in decision 1.

## What remains unknowable

**Whether the UI renders the diff.** The agent emits an `editToolCall` with `diffString`, `linesAdded`, `linesRemoved`, `beforeFullFileContent`, and `afterFullFileContent`. The React component that paints it lives in the Cursor workbench, not in `cursor-agent-exec`, and could not be found in this bundle. Claims about what a user sees are claims about a layer outside the artifact.

**Whether a remote or cloud executor adds checks.** `StrReplace` talks to a `resourceAccessor`-provided read and write executor. Whether a remote executor adds a sandbox, an agent-store barrier, or a stricter path check is outside this artifact. The `F_e` agent-store conflict decorator is visible and its path regexes are visible, but the resource identities it consults are not resolvable from the bundle.

**Whether a text file can ever arrive as a blob.** `Sbe` accepts only `output.case === "content"`, and a `dataBlobId` or `contentBlobId` output would be rejected as `BinaryFileError`. The read shaper has a blob window and a `LIe` ceiling that serves double duty, so the plumbing exists. Whether it is reachable for a text file under the truncation cap could not be determined. If it is, a large text file is reported as binary, which would be a bug.

**What `sparseLineNumbers` is at the edit tools.** The fuzzy hint renders through `fH`, which honours `sparseLineNumbers` by emitting only every Nth line number. The value is threaded from the agent config. In this build the config builder derives it from `promptVersion === "cursor-0226" ? 10 : undefined`, so the hint would show every tenth line number for that one prompt version. Whether an edit tool ever receives a non-zero value in practice depends on config paths outside the bundle, so the hint's exact rendering for `cursor-0226` is inferred rather than measured.

**Whether the vendor gate is the whole gate.** The `switch` in the tool list builder is visible and complete for the seven vendors it names, with an unreachable default. But whether another layer filters the list afterwards (a feature flag, a per-agent-type filter, a model capability check) could not be ruled out. The `isGemini3` check immediately after the switch shows the list is still being mutated, so treat the vendor switch as the primary gate rather than the only one.

**Why `multiFile` is a parameter.** `Bbe` takes `multiFile = false` and never reads it, and the parameter occurs once in the whole function. It is dead, and a reimplementation should not add it. Whether some other caller passes `true` (the notebook path, maybe) could not be determined, but since the parameter is unread it does not matter.

**Whether `MultiStrReplace` ever ships.** The id exists in two lookup tables and in the read-retry whitelist, and there is no registration. Either it is dead code from a removed feature or it is behind a flag that is off in this build. The shape of its parameters, and therefore whether it was an array-of-edits tool, is not recoverable from this artifact.

**Whether the write layer restores line endings the way this report says, run end to end.** The engine was run verbatim from the daemon for every Trap test, but the write-side restore (`At`, `dr`) lives in the `cursor-agent-exec` bundle, not in the daemon, so Traps 5 and 9 and the mixed-ending cases were exercised on the engine side only. The restore was read from the bundle and was not re-run.

**Why rig reversed to in-order application.** The commit title is `triobench round 4: edits apply in order` (2026-10-19), and the benchmark data is not in the repo, so decision 10's low confidence stands. A successful stale edit on a local file is pinned by tests only for rig's `Remote`; the local evidence is a probe (source, probe).

**Text the notes could not reach.** Pi's issue 5899 and the `zhayujie/CowAgent` commit were not read (pi's own commit and tests were). Whether Zed's language server sees a buffer edit was not verified. Codex's `apply_patch` instruction text returns 404 at the pinned commit, so what the model is told about it is unknown. How OpenCode's dev branch relates to its release line, and whether its TUI drives the V2 core, is not in the sources. Gemini CLI's self-correction pass and Cline's `apply_patch` fuzz tier were read but not run end to end.

## Where this build differs from the parent spec

These are the disagreements between `agent-file-tools.md` and Cursor 3.27.12's shipped bundle. Each one is a finding rather than a nuisance, because a reimplementation built from the spec would reproduce the wrong behavior.

**1. The de-indent example is wrong, and the pseudocode is right.** The spec's worked-examples table says `old_string: "  foo"` with `new_string: "foo"` produces `Error: old_string and new_string are exactly the same`, and the prose repeats it as "so a de-indent attempt fails while a re-indent with differing forms succeeds". It does not fail. The identity check compares EOL-normalized forms and nothing else; `"  foo"` and `"foo"` are not equal, so the check passes, the fallback matches, and the edit de-indents the line. Verified against the shipped matcher: `runSingleEdit({contentsBeforeEdit: "foo\n", old_string: "  foo", new_string: "foo"})` returns `usedWhitespaceInsensitive: true` and `"foo\n"`. The spec's own pseudocode (`IF oldStr === newStr THROW`) is correct and contradicts its example.

**2. The spec says `StrReplace` is always registered. It is not.** The shipped build switches on model vendor: Anthropic, Google, xAI, zai, and unknown get `StrReplace` plus `Write`; OpenAI and Cursor vendors get `ApplyPatch` *instead of* `StrReplace` when `isGpt5Family` is true. Bundle offset 4582200 and daemon offset 13939958 both show the switch. A harness reproducing this build must not offer `StrReplace` to a GPT-5 family model.

**3. The spec's offset for the registration is stale, and so is its offset for the engine.** The spec gives `STR_REPLACE` at 4209511 and the edit engine at 4150373. The shipped build has the registration at **4224839** and the engine (`Bbe`) at **4165672**, with the read-then-write wrapper (`Sbe`) at 4158713, the write stage (`Rbe`) at 4163006, the error classes at 4149970 through 4150290, the result packer (`vbe`) at 4157903, the renderer (`ybe`) at 4157164, the planner gate (`W_e`) at 4150449, and the 512 KiB constant (`wbe`) at 4157892. Every offset in the spec's StrReplace section is about 15,000 bytes low.

**4. The spec says `formattingOptions` for the edit tools are hardcoded, so the hint always uses the default line-number format.** The research file states that `gpt5StyleLineNumbers` and `gpt5CodexCatN` are "hardcoded false for the edit tools" and that the snippet therefore "always uses the default `%6d|` format, regardless of model". In the shipped build the tools receive `e.formattingOptions` from the agent config (bundle 4583105), which is built by `buildFormattingOptions` (daemon 14044652) as `{ shouldUseFormatCodeblock: true, gpt5StyleLineNumbers: modelInfo.isGpt5Family, gpt5CodexCatN: false, ... }`. So on a GPT-5 family model the hint uses `L<n>:<text>` rather than the pipe format. This only matters on a model that never reaches this tool in the first place, because of finding 2, which is probably why the discrepancy was not noticed.

**5. `fH` has a second arity the spec does not describe.** The spec's description of the formatter covers the string form. The shipped `fH` (2529940) accepts either a string or an array of `{lineNumber, text}` objects, and in the array form an entry with a non-integer `lineNumber` renders as `"...".padStart(6, " ")` in the cat-n form, `"..."` in the GPT-5 form, and (in the default form) as a line that skips the number. That array form is used by Read for sparse numbering, not by `StrReplace`, but a reimplementation that shares one formatter needs to know both signatures exist.

**6. The spec's `K_e` client-visible string is not quoted anywhere.** The failure matrix shows `The model provided an ambiguous search string to replace` for the client half of the ambiguous error, and that is correct (verified at 4150055), but the spec's `### Failure modes` section only says "Client text" for the two engine-before-matching rows and leaves the ambiguous row's client text out. Minor, but it is a byte-exact string that a reimplementation will need if it renders a UI toast.

**7. The spec's claim that there is "no `MULTI_STR_REPLACE`" is right, but its evidence is incomplete.** The id appears in `K.Dm_.fields` as `MULTI_STR_REPLACE: "edit_tool_call"` (2742376), in the team map as `MULTI_STR_REPLACE: "agent-quality"` (3475096), and in the read-retry whitelist `iTe` (bundle 4338xxx, daemon 13939958 region), and there is no `AG("MULTI_STR_REPLACE", ...)` anywhere. So the id is referenced by three tables and registered by none. It is recorded here because it is the closest thing in this build to the array tool that `pi_edit` and Copilot both ship.

**8. The spec says the tool's read has "no read-before-write gate" and the prompt text is at offset 2750288.** The string exists; the offset in the shipped build is **2764730** for `at least once before editing`, with `You MUST use the ` at 2764674. The behavior claim is correct.

**9. The spec describes five error classes but the shipped build has nine.** The spec names `H_e K_e V_e Y_e` (string-not-found, ambiguous, empty, identical). The shipped build also has `kbe` (file not found), `xbe` (invalid file), `Ebe` (read permission denied), `Ibe` (write permission denied), `Tbe` (write rejected), `Cbe` (binary), `Pbe` (file too large), and `Abe` (invalid edit args). Those are described in the spec's failure tables but not enumerated as classes, which matters if you are mapping a Rust error enum onto them one-to-one.

**10. The spec's `### No read-before-write gate` says the freshness read means "a stale edit fails with `File is too large to edit` or a genuine not-found rather than a stale-write error".** The first half is misleading: a stale edit fails with a not-found or an ambiguity error, and `File is too large to edit` has nothing to do with staleness. That sentence conflates two mechanisms.

**11. The spec gives the `dsv3-1018` description as ending with `To create or overwrite a file, you should prefer the write tool.`** Correct, and verified. But the spec's rendering of the default branch omits the `isComposer15` conditional line entirely, so a reader would not know that branch can have six bullets instead of five. The line is `- The edit will FAIL if path isn’t given as the first argument. Always provide path first.` with a U+2019.

**12. Nothing in the spec covers `pi_edit` at all.** The whole parallel family is undocumented. This report's `### The \`pi_edit\` counterpart` is the first description of it, including the fuzzy-path whole-file rewrite, which pi's own tool had until 2026-06-19.

## Conformance checklist

The twenty items from the parent spec's `### StrReplace` entry, each marked confirmed or corrected.

| # | Claim | Status | Reason |
|---|---|---|---|
| 1 | Parameters are exactly `path`, `old_string`, `new_string`, `replace_all`, and no `explanation`, `line_range`, `old_str`, `new_str`, or `target_file` exists | **confirmed** | two schemas, `oke` and `ike`; the second uses `file_path`; none of the five absent names appears anywhere in the tool's code |
| 2 | `replace_all` has no schema default on the default branch, arriving as `undefined` and normalized with a triple-equals test against `true` | **confirmed** | `.optional()` with no `.default()`; normalized by `const l = !0 === s.replace_all` |
| 3 | `replace_all` has a `false` default only on the `dsv3-1018` branch | **confirmed** | `.default(!1)` on `ike` only |
| 4 | Both model strings and the file content have `\r\n` normalized to `\n` before matching | **confirmed, with a caveat** | model strings use `/\r\n?/g`, file content uses `/\r\n/g`; a lone `\r` survives in the file and is unmatchable across lines |
| 5 | An empty `old_string` throws before any search runs | **confirmed** | engine step 2, tested on the raw `old_string`, before `findMatches` |
| 6 | Normalized old equal to normalized new throws, so a de-indent attempt fails while a re-indent with differing forms succeeds | **corrected** | the throw is confirmed, but the example is wrong. The comparison is EOL-normalized only and does not trim, so `"  foo"` to `"foo"` is not an identity error and de-indents successfully. Verified against the shipped matcher |
| 7 | The exact scan advances by the needle length, not by one, so `aa` in `aaa` is exactly one match | **confirmed** | `i += r.length`; verified `count === 1` for `aaa`/`aa`, `count === 2` for `aaaa`/`aa` |
| 8 | The whitespace fallback runs only when the exact scan found nothing, and only when enabled on the call path | **confirmed** | `if (matches.length > 0) return` before the `if (!enabled) return`; `StrReplace` always passes `true` |
| 9 | The fallback trims per-line whitespace on both sides and does nothing else: no internal-run collapse, no Unicode normalization, no quote folding | **confirmed** | the matcher body contains only `trim()`, `split("\n")`, `endsWith`, `startsWith`, `lastIndexOf`, `indexOf`; verified that `"foo  bar"` does not match `"foo bar"` |
| 10 | The fallback requires a strict one-to-one line correspondence and offers no ellipsis elision and no block-anchor strategy | **confirmed** | loop bound `start <= fileLines.length - needle.length`, no inner consumption; verified that a two-line needle cannot span a three-line region |
| 11 | Zero matches report the relaxed-whitespace suffix only when the fallback ran | **confirmed** | the suffix is keyed on `usedWhitespaceInsensitive`, which is set true on the branch that calls the fallback, even when it returns zero matches |
| 12 | The zero-match throw sends the short fixed form to the client and the long form with the fuzzy hint to the model | **confirmed** | `H_e`'s constructor hardcodes `clientVisibleErrorMessage: "The string to replace was not found in the file."` and takes the model string as its second argument |
| 13 | The fuzzy suggestion is advisory only and never changes whether the edit applies | **confirmed** | the hint is built inside the throw expression after `occurrences === 0`; `buildDidYouMean` returns a value or `null` and has no effect on control flow |
| 14 | Multiple matches with `replace_all` false throw, and the message inserts ` after relaxing whitespace` before the period when the fallback found them | **confirmed** | template `... in the file${d ? " after relaxing whitespace" : ""}. Use replace_all ...` |
| 15 | The overlap guard fires only when `replace_all` is false, the match came from the exact scan, and the needle starts anywhere after the first match start plus one | **confirmed** | condition `!l && !d && -1 !== r.indexOf(c, u[0].startCharIndexZeroIndexed + 1)` |
| 16 | The splice recovers offsets into the original content, so untouched lines keep their exact original bytes | **confirmed** | `slice(0, a) + replacement + slice(b)` over the original string; no normalized copy is ever written back |
| 17 | The bulk splice runs only when the replacement is non-empty and there are two or more matches, walking offsets in descending order | **confirmed** | `if ("" !== r && n.length >= 2)` after `sort` descending, with a bail-out on inconsistent offsets |
| 18 | An empty `new_string` eats surrounding blank whitespace so a whole-line deletion removes its newline | **confirmed** | the three-branch clause in the sequential loop; verified five deletion vectors |
| 19 | `replace_all` on `aa` in `aaa` with `X` produces `Xa` and reports one replacement | **confirmed** | verified: `"Xa"`, `occurrencesModified: 1`, message `1 occurrence ... has been updated.` |
| 20 | Success text is one flat sentence with no diff, no line numbers, and no count unless `replace_all` was set | **confirmed** | the message builder is the only contributor; `linesAdded`, `linesRemoved`, and `diffString` are proto fields never read by the renderer |

Nineteen confirmed, one corrected. The correction matters: an implementation built from the spec's example would reject a legitimate de-indent and would need a workaround.

## Sources

### Bundle, Cursor 3.27.12

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-exec\dist\main.js`, 9,814,287 bytes, one line, read as latin1 and sliced. All offsets below are into this build.

- `AG("STR_REPLACE", ...)` registration and `ake` factory: 4224839
- default schema `oke` and `dsv3-1018` schema `ike`: 4224390 and 4224590 approximately, both inside the registration slice
- description generator, all three branches: 4224900 through 4226700
- the U+2014 EM DASH between `(default false)` and `if true` in the description: 4226537
- the U+2019 RIGHT SINGLE QUOTATION MARK in `isn’t`: 4226325
- `Bbe` (the edit engine): 4165672
- `findMatches` (anonymous IIFE, inside `Bbe`): 4165994
- `trimmedBlockMatch` (anonymous, nested inside `findMatches`): 4166867
- the did-you-mean builder (anonymous, inside `Bbe`): 4167414
- `splice` (anonymous `m`, inside `Bbe`): 4168802
- `Mbe` (EOL normalizer): 4165324
- `Sbe` (read-then-write wrapper): 4158713
- `Rbe` (write stage): 4163006
- `W_e` (plan-mode gate) and `G_e`/`z_e` (its strings): 4150449, 4150090, 4150140
- `S_e` (markdown extension allowlist) and `k_e` (canvas regex): 4140186, 4140241
- error classes `H_e`, `K_e`, `V_e`, `Y_e`: 4150000 through 4150300
- error classes `kbe`, `xbe`, `Ebe`, `Ibe`, `Tbe`, `Cbe`, `Pbe`, `Abe`: 4159000 through 4165200
- `vbe` (result packer) and `wbe = 524288`: 4157903, 4157892
- `ybe` (renderer) and `JB` (result wrapper): 4157164, 2309224
- `fH` (line-number renderer): 2529940
- `Lbe` (diff driver) and `ube = 262144`: 4168900 approximately, 4152120
- `Bbe` call site with `eolSequence: "\n"`: 4227189
- vendor gate tool list builder: 4582200, with `tJe` at 4581900 and `NSe` (`APPLY_PATCH`) at 4202223
- `formattingOptions` threaded from agent config: 4583105
- `JB` result wrapper: 2309224
- `buildFormattingOptions` (bundle copy): 5079870, 5080100; daemon copy 14044652
- `Kze` (`sparseLineNumbers` derivation): 5078455
- read shaper with `LIe`: 4343900
- read truncation and `MAX_TEXT_SIZE = 8*1024*1024`: 8647921, daemon 18017 for the constant
- `At` (EOL harmonizer) and its LCS inner function: 8605364, 8605460
- `dr` (file buffer builder): 8609588
- `detectMixedEndings` inline check for mixed endings: 8609100 approximately
- binary extension regex `aTe`: 4339800 approximately
- `isBinaryFile` format sniffer `mr`: 8611841
- read-first prompt text: 2764674 and 2764730
- `iTe` (read-retry whitelist) containing `StrReplace` and `MultiStrReplace`: 4339800 region
- `MULTI_STR_REPLACE` wire-case map entry: 2742376
- `MULTI_STR_REPLACE` team map entry: 3475096
- `PI_EDIT` registration `hMe`: 4559169
- `pi_edit` schema `KRe` (replacement) and `QRe` (args): 4553926, 4554595
- `LocalPiEditExecutor` class: 9196579 (latin1 index; the earlier 9196200 was stale)
- `bE` / `applyPiEdits` in the bundle: 9192653
- `yE` (pi LF normalization) at 9192136 and `wE` (pi fuzzy normalization) at 9192202; the earlier 9191800 was stale
- `EditSuccess` proto shape: 5970095
- `EditToolCall` and `EditArgs` proto shapes: 5972459, 5969181
- `EditToolCallDelta`: 5972839
- `ReadSuccess` proto shape: 6093936
- `PiEditToolSuccess` and `PiEditToolCall`: 6062332, 6061134

### Daemon, unminified twin

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, original identifiers intact. Offsets are character indices in the UTF-8-decoded file; the engine and pi offsets were re-measured in this pass (source).

- `createStrReplaceTool`: 12625769
- `performEdit`: 12510955
- `async function performWrite`: 12515656
- `runSingleEdit`: 12524268
- `findStringMatches`: 12521327 (the `function` keyword; add 9 for the name)
- `findOccurrencesIgnoringWhitespace`: 12519369
- `replaceAllMatches`: 12522738
- `joinDisjoint`: 12522361
- `normalizeLineEndings`: 12523750
- `getClosestFuzzyMatchWithVSCodeDiff`: 12526414
- `formatStrReplaceResultForModel`: 12619463
- `createStrReplaceTool`'s registration tail with `createZodAgentTool("STR_REPLACE", ...)`: 12628930
- `PI_EDIT` tool definition `createPiEditTool`: 13899225
- `renderPiResult` and `extractResultText`: 13895487, 13895107
- `toPiEditToolResult`: 13906724
- `pi_edit` schema `editSchema`: 13911658
- `applyPiEdits`: 16836811
- `findMatches` (pi): 16835288
- `applyMatches` (pi): 16836361
- `fuzzyNormalize` (pi): 16834590
- `stripBom`, `detectLineEnding`, `normalizeToLf`, `restoreLineEnding`: 16834000 through 16834400
- `withPiFileMutationQueue`: 16837822 (16843029 is the `applyPiEdits(` call inside the executor)
- `resolvePiPath`: 16838486
- `buildFormattingOptions`: 14044652
- `MAX_TEXT_SIZE = 8*1024*1024`: 18017
- vendor gate `switch(props.modelInfo.vendor)`: 13939958
- `PI_EDIT` in the per-tool identifier map: 10316166 and 13899225

### Extractions and probes

- Claude Code 2.1.311, win32-x64 bundle (latin1 decode, 14 files, 0 anchor failures): `edit.js` (the `Edit` registration, prompt, schema, validateInput, call, and `mapToolResultToToolResultBlockParam`), `read-file-state.js` (the LRU state store) and `error-strings.json` (126 extracted strings, with offsets) (bundle).
- Cursor 3.27.12 `cursor-agent-exec` bundle: a prettified slice of `STR_REPLACE`. The prettier pass failed on a fragment boundary, so the slice begins with a fragment of unrelated disposable code and the authoritative text is the minified region (bundle).
- Earlier probes of the shipped bundle and daemon: offset and string location, latin1 slicing, a verbatim reimplementation of `findStringMatches`, `findOccurrencesIgnoringWhitespace`, `replaceAllMatches`, `joinDisjoint`, `normalizeLineEndings` and `runSingleEdit` with the mangled names restored (34 named vectors covering the fallback, `replace_all`, deletion, CRLF, identity, empty input and Unicode), match-region dumps, a reimplementation of `applyPiEdits` with the fuzzy whole-file rewrite, and byte-level inspection of the em dash and U+2019 (probe).
- This pass, Cursor's own engine: `findOccurrencesIgnoringWhitespace`, `findStringMatches`, `joinDisjoint`, `replaceAllMatches`, `normalizeLineEndings`, `runSingleEdit`, `getClosestFuzzyMatchWithVSCodeDiff` and `addLineNumbers` loaded as source text from the unminified daemon, with only the error classes and a logger stubbed, fed the wrapper's `\r\n` to `\n` normalization, and run on Trap 1 to 11 and the Trap 12 cases (daemon, probe). Of 32 Trap 1 to 11 sub-cases, 28 agree with the report's expectation, 3 differ and 1 is informational; the differences are Trap 1's matched line when the needle needs the fallback and Trap 7's `const a = 1; // note`, which is an exact hit. Rig's probe of the same traps on its own engine found 12 of 22 sub-cases agree and 10 differ. The write-layer line-ending restore lives in the exec bundle, not the daemon, and was not run.
- Pi: the pure matcher sliced from pi HEAD and from `128330e36^`, and the `applyPiEdits` helpers sliced from the daemon, run on the same inputs (probe). Rig: the verbatim engine, `Seen` and `write_if` compiled with the stable Rust toolchain on this machine, with the hash, the diff and the atomic write shimmed: 71 engine cases, 12 staleness scenarios on temporary files, 22 Trap sub-cases and cost timings (probe).

### Harness sources

Copilot: `microsoft/vscode`, `extensions/copilot/src/extension/tools/node/`, local checkout at HEAD `b208bc25` (2026-10-25) (source).

- `abstractReplaceStringTool.tsx` - the shared matcher dispatch, `_errorConflictingEdits`, `applyAllEdits`, `generateEdit`, the healing call
- `replaceStringTool.tsx` - `replace_string_in_file` and its description
- `multiReplaceStringTool.tsx` - `multi_replace_string_in_file`, the merge and sort logic
- `editFileHealing.tsx` - the repair pass
- `editFileToolUtils.tsx` - `findAndReplaceOne` and its four strategies, `applyEdit`, `assertPathIsSafe`, `makeUriConfirmationChecker`
- `insertEditTool.tsx` - read for the natural-language-instruction surface
- `applyPatchTool.tsx` - read for the delete-by-patch surface

Zed: `crates/agent/src/tools/` in the local checkout at `37faef81f7` (2026-10-27, clean), with the older `d9ad6aff67` (2026-08-22) for the drift table (source).

- `edit_file_tool.rs` - `EditFileToolInput`, `process_streaming_edits`, the streaming partial-input loop, the tests that pin overlap rejection, mid-edit failure and the skipped-row deletion
- `edit_session.rs` - `EditSession`, `EditPipeline`, `extract_match`, `ensure_buffer_saved`, `resolve_dirty_buffer`, the `Display` text and the error strings
- `edit_session/streaming_fuzzy_matcher.rs` - the KMP exact scan, the line-level DP, the costs, the `0.8` similarity threshold, the `LINE_HINT_TOLERANCE`
- `edit_session/streaming_parser.rs` and the `streaming_diff` crate - the partial-JSON diffing and the character-level alignment
- `edit_session/reindent.rs` - `Reindenter`, `compute_indent_delta`, `compute_rest_indent_delta`, and the tests pinning clamping and whitespace-only lines
- `tool_permissions.rs` - the permission gate and the dirty-buffer prompts; also `worktree.rs`, `text.rs`, `file_content.rs` and `language_model_core/src/request.rs` for loading, line endings, decoding and the `raw_output` doc

Others, each at the version named in its H3 (source unless marked):

- pi: local checkout at `a92283056` (2026-10-27), `packages/coding-agent/src/core/tools/edit.ts`, `edit-diff.ts`, `file-mutation-queue.ts`, `CHANGELOG.md`, `tools.test.ts`; history at `128330e36` (2026-06-19) and its parent.
- rig: local checkout at `a83b774d0` (2026-10-25), `crates/rig-tools/src/edit.rs`, `edit/tests.rs`, `workspace.rs`, `read.rs`, `fs.rs`; commits `c9c928d4f`, `e034438fa`, `2fe8609ed`, `c2b6c3923`, `0358ebe16`, `bfc4efca6`.
- zeroclaw: local checkout at `2d85371f3` (2026-09-18), `crates/zeroclaw-tools/src/file_edit.rs`, version 0.8.5.
- Codex CLI: `openai/codex` at `bea47c06dfac` (2026-10-28), `codex-rs/apply-patch/src/` (`seek_sequence.rs`, `file_update.rs`, `text_file.rs`, `invocation.rs`), with the removed tools at the parents of PRs 15773, 15775 and 21170 (web: raw files at the pinned sha and the GitHub API).
- Gemini CLI: `google-gemini/gemini-cli` tag `v0.66.0`, `970413207732`, `packages/core/src/tools/edit.ts` and helpers; docs at https://geminicli.com/docs/tools/file-system/ (web).
- OpenCode: `anomalyco/opencode` tag `v1.19.5`, `810c0eba1a73`, and dev `fa1260d6ca`; docs at https://opencode.ai/docs/tools/ (web).
- Cline: `cline/cline` tag `v4.1.26`, `a459f1d`, legacy `v4.0.12`, `f267bf4`; docs at https://docs.cline.bot/tools-reference/all-cline-tools (web).
- Aider: `Aider-AI/aider` at `5dc9490bb35f` (2026-05-22), `aider/coders/editblock_coder.py`; issue 306 at https://github.com/Aider-AI/aider/issues/306 and https://aider.chat/2023/12/21/unified-diffs.html (web).
- Goose: `aaif-goose/goose` at `ee8500d5e060` (2026-10-28), tag v1.56.0 (`9949cf9`), `dev/edit.rs`; PRs 13144 and 7466 (web).

### Repository

- `C:\Users\Vinnie\cursor\runvil\crates\runvil-internal\vfs\src\handle\operations.rs` - the `Access` surface, including `str_replace` at line 74
- `C:\Users\Vinnie\cursor\runvil\crates\runvil-internal\vfs\src\error.rs` - `VfsError`, including `Anchor` at line 76 and its `Display` at line 229
- `C:\Users\Vinnie\cursor\runvil\crates\runvil-internal\vfs\src\stat.rs` - for the absence of line-ending and encoding metadata
- `C:\Users\Vinnie\cursor\runvil\crates\plugin-std\src\` - `lib.rs`, `pack.rs`, `check.rs`, `spec.rs` and the three test files
- `C:\Users\Vinnie\cursor\runvil\crates\plugin-std\schemas\replace.json` - the current definition, with the description recorded as the crate's own wording in `schemas\EDITS.md`
- `C:\Users\Vinnie\cursor\runvil\vibe\2026-10-27-2-plugin-std-file-tools.md` - the plan, including the open question this report answers

### Web

Used only for corroboration and for the lineage argument, and labeled inline where it appears. The specific claims:

- The leaked Cursor 2.0 prompt and the `asgeirtj/system_prompts_leaks` dump, which corroborate the two description branches and the absence of a runtime read gate.
- `forum.cursor.com` thread 74000 on whitespace-only edits being ignored by the **apply model**, which is a different subsystem and must not be conflated with `StrReplace`.
- `anomalyco/opencode` issues and PRs describing fuzzy replacers (the code at v1.19.5 has nine matchers, seven fuzzy, all live) and a `WhitespaceNormalizedReplacer` that collapses internal runs, which Cursor does not.
- `aws/amazon-q-developer-cli` PR 3725 proposing a three-strategy chain with a 0.6 Levenshtein threshold, which does not exist in Cursor.
- Aider's `replace_closest_edit_distance` at 0.8, a `difflib.SequenceMatcher` ratio behind a dead-code `return`, plus maintainer issue 306 on why fuzzy whole-block matching was disabled.
- Claude Code issues 26996 and 29870 on tabs-to-spaces and U+00A0 failures, which Cursor's fallback happens to neutralize.
- `zhayujie/CowAgent` commit a94f4e3 on locating a fuzzy match in the original content rather than a normalized copy.

Where the web and the bundle disagree, the bundle wins and this report says so at the point of disagreement. The clearest case is the claim that Cursor uses a similarity threshold: no such constant exists anywhere in `Bbe`, and the only numeric literals in the match path are `0`, `1`, and the hint's context-line count of `2`.

*2026-10-27 - DeepSeek V4.1 Flash; enriched with twelve-harness evidence 2026-10-28 - Claude Sonnet 5.5*
