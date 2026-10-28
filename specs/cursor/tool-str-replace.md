# Cursor's `StrReplace` file tool: an implementation report

Cursor's edit tool is a bare exact-substring replacement with one narrow fallback. Everything interesting about it is in the order the checks run and in what the fallback does not do. It has no fuzzy matching, no read gate, and no freshness token, and yet it is the tool that most models have been trained on, which is why its surface is worth copying even where its internals are worth replacing.

The section that matters most is `## Decisions you have to make`, because Runvil already has `Access::str_replace` and the question is not how to match a string but where the semantics should live.

## The short version

- **There is no fuzzy matching.** No Levenshtein, no similarity threshold, no block anchor, no `...` elision, no smart-quote folding, no NFC. The only fallback is a line-trimmed block match that requires strict one-to-one line correspondence. Any implementation that adds a similarity score is a different tool with different failure modes.
- **The pipeline order is exact scan first, line-trimmed block match second, and nothing third.** The fallback runs only when the exact scan produced zero matches, so a file with two exact matches never reaches the fallback even if the fallback would have matched something else.
- **The exact scan advances by the needle's length, not by one.** So `aa` in `aaa` is exactly one match. With `replace_all: false` that single match trips a separate overlap guard and throws; with `replace_all: true` it replaces once and the file becomes `Xa`, not `XX`.
- **The overlap guard is a second check that only exists because of the first.** It fires when `replace_all` is false, the match came from the exact scan, and `indexOf(oldString, firstMatchStart + 1)` still finds something. That is what catches self-overlapping needles that the non-overlapping scan undercounts.
- **`old_string === ""` throws before any search**, and normalized-equal old and new throws too. So `"  foo"` to `"foo"` is an error, while `"  foo"` to `"    foo"` is a successful re-indent through the fallback. The identity check is on the normalized forms, not the raw ones.
- **The fallback trims per-line leading and trailing whitespace and does nothing else.** No internal run collapsing, no blank-line tolerance, and no partial matching for single-line searches. That last constraint is the non-obvious one: a one-line `old_string` must equal a whole line after trimming, so `"const a = 1;"` will not match `"const a = 1; // note"` through the fallback, and `"foo  bar"` will not match `"foo bar"` because internal runs are untouched.
- **When the fallback succeeds, `new_string` goes in verbatim, with no re-indentation.** Cursor does not consult the file's indentation and does not adjust the replacement. A model that emits a tab-indented replacement into a space-indented file gets a tab-indented region in a space-indented file.
- **Untouched lines keep their exact bytes, because nothing is ever rebuilt.** The matcher computes character offsets into the original content and the splice touches only `[start, end)`. An implementation that splits into trimmed lines, rejoins, and writes that back re-indents the entire file, which is the single worst failure available in this tool.
- **CRLF is normalized in three places, and an LF `old_string` against a CRLF file is an exact match, not a fallback match.** Model strings get `\r\n?` collapsed to `\n`; file content gets `\r\n` collapsed to `\n` before the engine sees it; the write layer restores the file's ending on the way out. The model never has to know about CRLF.
- **The splice has two paths.** A bulk descending-offset splice when the replacement is non-empty and there are two or more matches, and a sequential loop otherwise, including every deletion. The deletion path eats the surrounding newline when the match was alone on its line, which is why a whole-line delete does not leave a blank line behind.
- **There is no read-before-write gate.** No error string exists for one. The requirement is prompt text only, and the tool re-reads the file itself so a gate would be redundant.
- **There is also no freshness check against what the model last saw.** `StrReplace` reads fresh and edits fresh, so a stale edit fails only if the anchor text no longer exists or is no longer unique. Nothing ties the edit to the bytes the model read.
- **The success text is one flat sentence.** No snippet, no line numbers, no diff stat, no count unless `replace_all` was set. `The file <path> has been updated.` or `N occurrences in file <path> have been updated.` That is the whole model-visible result.
- **The failure text carries a free hint that costs nothing and is worth keeping.** On zero matches, Cursor picks the longest line of the failed `old_string`, finds the first file line containing it after trimming, and shows a numbered window around it under ` Found a possible fuzzy match, did you mean:`. It never affects whether the edit applies.
- **The shipped build gates `StrReplace` by model vendor.** Anthropic, Google, xAI, and unknown vendors get `StrReplace` plus `Write`; OpenAI and Cursor vendors with `isGpt5Family` get `ApplyPatch` *instead of* `StrReplace`. A harness that always offers `StrReplace` is not reproducing this build.
- **`pi_edit` is the parallel surface and it is a different, in places better, design.** It takes an array of `{oldText, newText}` edits, applies them all against one snapshot of the original content, is all-or-nothing, and has both an exact path and a whole-file NFKC normalization path. That last path rewrites every untouched line, which is exactly the trap described above, in shipped code.

## Decisions you have to make

These are ordered by how much they change the shape of the crate, not by how hard they are.

### 1. Does `StrReplace` own its matching, or does it call `Access::str_replace` and layer the fallback above it?

This is the open question already recorded in `vibe/2026-10-27-2-plugin-std-file-tools.md`, and it is the one that decides the file layout.

The facts. `Access::str_replace` already implements exact single-occurrence replacement. It refuses an empty `old` before touching the backend, it claims a write, it fires the event, and it delegates to `self.inner().str_replace(&path, old, new)`, which answers `VfsError::Anchor { path, anchor, count }` when the count is not one. What is missing above it is `replace_all`, the trimmed-line fallback, and Cursor's exact error strings.

**Option A: the Pack reads, matches, and writes.**

The Pack calls `Access::read_string`, runs the whole pipeline over the returned text, and calls `Access::write` with the result. `Access::str_replace` becomes unused by this tool.

- What it costs: the read-modify-write window opens up. Between `read_string` and `write` another access can interleave, because the two calls claim separately. You also lose the VFS's own anchor semantics, so a backend that implements `str_replace` natively (a database, an object store, a document service) cannot be used. And the whole edit becomes a full-file write from the backend's point of view, so a backend that does line-level mutation cannot help.
- What it buys: one place for the semantics. The entire pipeline is a pure function over `&str` to `&str`, which means the whole thing is testable with no filesystem at all, which is exactly what Cursor's own `Bbe` is. Matching and splicing live in one module and one test file.
- What breaks: nothing structural, but you have to be honest that `Access::str_replace` is now dead weight in the crate. Leaving a dead method that answers different error text than the tool is a trap for the next person.

**Option B: the Pack calls `Access::str_replace` for the exact single-occurrence case and does its own work for everything else.**

The Pack computes the match itself, then dispatches: if there is exactly one exact match and `replace_all` is false, call `Access::str_replace`; otherwise read, splice, and `Access::write`.

- What it costs: two code paths that must agree. The Pack has to reproduce Cursor's match counting to decide which path it is on, which means it already knows everything `Access::str_replace` knows, which makes the delegation redundant. And the two paths produce different error text for the same user-visible condition (`VfsError::Anchor` versus Cursor's sentence), so the mapping layer has to translate.
- What it buys: the backend's native anchor path stays alive for the common case, and the claim is taken once by the VFS for that call.
- What breaks: the claim granularity becomes inconsistent. One branch claims once inside the VFS, the other claims twice from the Pack. Two calls to `StrReplace` on one path can interleave differently depending on which branch they take.

**Option C: matching lives in the VFS, as a richer anchor operation.**

Extend the VFS with a match policy - exact-unique, exact-all, trimmed-block - and let the backend do the work, returning the edited text or the failure.

- What it costs: the VFS grows a text-editing policy interface, which is a large conceptual addition to a layer that currently speaks in paths and bytes. Every backend must implement it or inherit a default.
- What it buys: the read-modify-write window closes, because the whole operation is one claim inside the VFS.
- What breaks: portability of the VFS. A backend that stores opaque bytes now has to know what a line is.

**Recommendation: Option A, the Pack owns the matching, with `Access::str_replace` retired or redefined.** Confidence: high, because the tool's semantics are genuinely text-level and belong in the layer that thinks in text, and because the read-modify-write window is already the status quo for `Write` and closing it is a separate, orthogonal piece of work. Do not leave `Access::str_replace` in place answering `Anchor` if the tool no longer uses it; either delete it or make it the primitive the Pack calls for exactly one case and nothing else.

If you take Option A, take the second half seriously: make the pipeline a pure function, `fn apply(content: &str, edit: &Edit) -> Result<EditOutcome, EditError>`, with no I/O and no awareness of `Access`. Then the only thing the Pack does is read, call it, write, and map the error. That is the same split Cursor has between `Sbe` (read, gate, write) and `Bbe` (the pure engine), and it is the reason Cursor's engine can be reasoned about at all.

### 2. Do you keep the line-trimmed fallback at all?

**Option A: keep it exactly as Cursor has it.** Per-line `trim()` on both sides, strict one-to-one line correspondence, partial anchoring on the first and last line only, single-line searches must match a whole trimmed line.

- What it costs: a class of match the model did not literally ask for. `"  foo"` can match `"        foo"`. Two different regions can become ambiguous only after trimming, and the model gets an "after relaxing whitespace" error it did not expect.
- What it buys: it solves tab-versus-space and U+00A0 at line edges for free, because both vanish under `trim()`. These are the two failure classes Claude Code has open issues for and Cursor does not. It is the single highest-value behavior in the tool after the exact match.
- What breaks: nothing, if the splice is offset-based. It breaks badly if the splice is line-based.

**Option B: drop the fallback, exact match only.** This is Claude Code's design.

- What it costs: every tab-indented needle against a space-indented file fails. Every file with non-breaking spaces fails at those lines.
- What it buys: a successful match is exactly what the model wrote. There is no class of edit that "sort of" matched.
- What breaks: models will retry, and each retry costs a turn.

**Option C: keep a fallback but make it a normalizing one.** Collapse internal whitespace runs, treat blank lines as optional, maybe normalize Unicode.

- What it costs: it changes bytes the model did not name. `"foo  bar"` matching `"foo bar"` is a silent rewrite of a string that might matter inside a literal.
- What it buys: fewer retries on badly-copied context.
- What breaks: the guarantee that a match means the text is what the model thinks it is.

**Recommendation: Option A, keep Cursor's fallback byte for byte.** Confidence: high, because it is the conservative choice that still fixes two real failure modes, and because every attempt to make it cleverer (opencode's eight replacers, Amazon Q's 0.6 threshold, Aider's disabled 0.8) either got disabled or became a different product. The one thing worth considering is a *flag*, not a different default, so a Host that wants strictness can turn it off.

### 3. Do you copy `replace_all`'s non-overlapping semantics, or use a real global replace?

Cursor's `replace_all` is not `String.replaceAll`. It replaces the non-overlapping matches the same scanner found, so `aa` in `aaa` becomes `Xa` and reports one replacement.

**Option A: copy it.** `replace_all` means "all the matches this tool found", which is all the non-overlapping ones.

- What it costs: a model that says `replace_all: true` on a self-overlapping needle gets fewer replacements than it asked for, silently. The success message says `1 occurrence ... has been updated`, which is technically accurate and practically confusing.
- What it buys: one scanner, one notion of a match, and the count in the message always equals the number of splices. The overlap guard and the count are consistent with each other.
- What breaks: nothing internally. It surprises anyone who assumes `replace_all` means `replaceAll`.

**Option B: use a real overlapping global replace.** Advance by one during the scan when `replace_all` is set.

- What it costs: two scanners, or one scanner with a mode. The count and the overlap guard have to be rethought, because the "more than one place and they overlap" error becomes unreachable under `replace_all`.
- What it buys: `aa` in `aaa` becomes `XX`, which is what a naive reader expects.
- What breaks: it is a different tool. Models trained on Cursor expect `Xa`.

**Recommendation: Option A, copy the non-overlapping behavior, and make the overlap guard mandatory.** Confidence: high, because the overlap guard is what makes the semantics safe: without it, `replace_all: false` on `aa` in `aaa` would silently edit one of two overlapping sites. The guard is the compensating control for the undercount, and the two must ship together or neither is correct.

### 4. Do you add a read-before-write gate?

Cursor has none. Claude Code has a full state machine.

**Option A: no gate, like Cursor.** The tool reads fresh and edits fresh.

- What it costs: a model can edit a file it never read, and will sometimes edit the wrong occurrence because it guessed the anchor.
- What it buys: no state to keep, no false refusals when the harness restarted or the context was compacted, and no way for the gate itself to be the bug.
- What breaks: nothing mechanically.

**Option B: a hard gate, like Claude Code.** Refuse the edit unless this session has read the file.

- What it costs: a state store keyed by normalized path, invalidated on every write, and an error string the model must learn. Partial reads complicate it: Claude Code treats a partial read as fatal for a full-file write and survivable for an edit.
- What it buys: it catches the case where a formatter rewrote the file between read and edit, which a fresh read alone cannot.
- What breaks: agents that legitimately edit a file named in the prompt without reading it, which is a common and reasonable pattern.

**Option C: no gate, but return the changed region in the result.** This is the design neither Cursor nor Claude Code uses, and it is interesting because it makes the model's next turn correct without a round trip.

- What it costs: result size, and a decision about how much context to include.
- What it buys: the model sees what it just did, in file coordinates, without spending a Read.
- What breaks: nothing, but it is a departure from Cursor's flat sentence, so a model trained on `The file X has been updated.` sees a different shape.

**Recommendation: Option A plus a cheap version of C.** Confidence: medium, because the gate's value depends on whether your harness has other agents writing the same files. If it does not, the gate is pure friction. The freshness problem is better solved by making the write atomic and compare-and-swap on a content hash, which is a VFS concern rather than a tool concern. Returning the changed region is worth prototyping because Cursor's flat sentence is the weakest part of its contract.

### 5. Do you keep Cursor's success text, or return the changed region?

**Option A: keep Cursor's exact sentence.** `The file <path> has been updated.`

- What it costs: the model has no confirmation of what changed. On a fallback match it does not know the region it hit was trimmed differently from what it wrote.
- What it buys: byte-identical behavior, which is the whole reason to copy the surface.
- What breaks: nothing, but it is information the harness has and throws away.

**Option B: keep the sentence and append a short numbered excerpt of the changed region.**

- What it costs: a slightly different contract, and a decision about excerpt size.
- What it buys: the model sees the result of a fallback match with the file's real indentation, which is exactly the information it lacks.
- What breaks: a model that pattern-matches on the exact sentence. Low risk, since the sentence is still the prefix.

**Recommendation: Option A for the first implementation, Option B as a deliberate later divergence.** Confidence: high on A as the starting point, because a fresh harness should get the boring behavior right before adding value, and because the fallback-match ambiguity is better fixed by telling the model in the *description* that the replacement is not re-indented.

### 6. Do you keep the two-valued splice, or unify it?

Cursor has a bulk descending splice for non-empty multi-match replacement and a sequential loop for everything else, plus line-eating for empty replacements.

**Option A: copy both paths.** They exist because the deletion path needs per-match lookahead at the surrounding line, which a bulk splice cannot express.

- What it costs: two code paths, and the bulk path has a bail-out (`if (start < 0 || end < start || end > tail) return undefined`) that falls through to the sequential path when offsets are inconsistent. That bail-out is only reachable from the fallback path, whose matches can in principle overlap.
- What it buys: correct deletion behavior, and an O(n) bulk path.
- What breaks: nothing. The bail-out is dead in practice for exact matches because they are always disjoint and ordered.

**Option B: one unified path.** Compute the deletion-adjusted ranges first, then splice them all descending.

- What it costs: you must run the line-eating adjustment for every match before splicing, using the original text for the lookahead. That is straightforward, but it is not what Cursor does, and the two are only equivalent if the adjustments are computed against the original coordinates.
- What it buys: one path, no bail-out, and the guarantee that the same set of matches always produces the same result.
- What breaks: the sequential path's adjustment reads `text` *after* earlier splices, because it reassigns `text` each iteration. Since matches are processed in descending order, earlier splices are all at higher offsets and cannot affect the lookahead of a lower one. So the two are equivalent in practice, which means unifying is safe.

**Recommendation: Option B, one unified path, computing adjustments against the original snapshot.** Confidence: medium-high, because the unified path is provably equivalent to Cursor's two paths for the inputs Cursor can produce, and it removes a bail-out that is otherwise a correctness question you have to keep answering.

### 7. Do you copy the `dsv3-1018` snake_case branch?

**Option A: only the `latest` shape.** `path`, `old_string`, `new_string`, `replace_all`, PascalCase name `StrReplace`.

- What it costs: nothing.
- What it buys: the surface the majority of models actually see.
- What breaks: nothing, unless you run DeepSeek models.

**Option B: both branches.** Add `search_replace` with `file_path` when the model is a DeepSeek family model.

- What it costs: a second schema, a second description, and the typo decision below.
- What it buys: a model trained on the leaked DeepSeek-flavored prompt sees the names it expects.
- What breaks: nothing, but it doubles the surface you test.

**Recommendation: Option A first, with the schema shaped so Option B is a table entry rather than a rewrite.** Confidence: high, because the parameter *names* are the only thing that differs and a per-model table is cheap to add later. Note one real difference worth preserving if you do add it: the `dsv3-1018` description says `ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.` and ends `To create or overwrite a file, you should prefer the write tool.`, while the default branch drops that line and appends `If you want to create a new file, use the Write tool instead.` Those are two different prompts, not two spellings of one.

### 8. Do you reproduce the typo?

The `dsv3-1018` `replace_all` description reads `Replace all occurences of old_string (default false)` - one `r`, one `s`. The default branch reads `occurrences`.

**Option A: reproduce it on that branch only.** **Option B: fix it everywhere.**

**Recommendation: reproduce it if and only if you ship the `dsv3-1018` branch, and put a comment on the line saying it is deliberate.** Confidence: high, because a one-character difference in a description is invisible to a reader and load bearing for a byte-exact comparison, and the failure mode is that someone "fixes" it in six months and silently changes a prompt. If you do not ship that branch, the question disappears.

### 9. Do you gate the tool by model vendor, as the shipped build does?

The shipped build offers `StrReplace` plus `Write` for Anthropic, Google, xAI, zai, and unknown vendors, and for OpenAI and Cursor vendors only when the model is *not* GPT-5 family. GPT-5 family models get `ApplyPatch` instead.

**Option A: always offer `StrReplace`.** It is one tool, it works, and a harness that is not chasing Cursor's per-model tuning should not carry the tuning.

- What it costs: a GPT-5 class model gets a tool it was not tuned for, which is a real but unmeasured cost.
- What it buys: one code path, one test surface, no dependency on a vendor enum you have to keep current.
- What breaks: nothing.

**Option B: reproduce the gate.** Offer `ApplyPatch` for GPT-5 family.

- What it costs: you now need `ApplyPatch`, which is a patch parser and a different matcher.
- What it buys: parity with the shipped behavior.
- What breaks: you have two edit tools to keep consistent.

**Recommendation: Option A, and record the divergence.** Confidence: high, because `ApplyPatch` is a whole second tool with its own parser, and the reason Cursor gates is model tuning rather than correctness. If you later measure that a GPT-5 model does worse with `StrReplace`, add `ApplyPatch` as a Pack variant, which is what a Pack system is for.

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

Every other file tool names its `dsv3-1018` branch in snake_case too (`write`, `delete_file`, `edit_notebook`, `list_dir`), so the snake_case set is a whole DeepSeek-flavored surface rather than a quirk of this tool.

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

`dsv3-1018` differences: `file_path` instead of `path`, a longer `file_path` description, `occurences` misspelled, and an actual `.default(false)`.

The absent default is load bearing. On the default branch `replace_all` arrives as `undefined`, and the code normalizes with a triple-equals test:

```js
const replaceAll = edit.replace_all === true;
```

That is equivalent to `Boolean` for a zod-validated boolean, but it is *not* equivalent for a value that slipped past validation, and it means `replace_all: false` and an omitted `replace_all` produce the same behavior through a different route. On the `dsv3-1018` branch the field is genuinely `false` by the time the code sees it.

`U$.Ikc` builds a plain `ZodObject` with no catchall, so unknown keys are stripped rather than rejected. A model that sends `explanation` gets it silently dropped.

### Parameters that do not exist

Worth stating because implementations keep adding them:

| Name | Present on `STR_REPLACE`? | Where it actually lives |
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

Three byte-level facts about that block.

The dash in the `Optional parameter` line is a real U+2014 EM DASH, three bytes `0xE2 0x80 0x94` in the file. Verified at bundle offset 4226537.

The `Write` name is not a literal. It is resolved with `Bj(t, "WRITE")`, which throws `` `Required tool ${t} not found in allTools` `` if `WRITE` is absent from the tool set. So a harness that offers `StrReplace` without `Write` hard-fails while building the description, before any tool call. If you are building a subset, either keep the reference or substitute a literal and note it.

There is a conditional line the default branch adds when `isComposer15` is set:

```
- The edit will FAIL if path isn’t given as the first argument. Always provide path first.
```

with a U+2019 RIGHT SINGLE QUOTATION MARK in `isn’t`, verified as three bytes `0xE2 0x80 0x99`. This is model-family-conditional, not a global line.

`dsv3-1018` differs in three places: it adds `- ALWAYS prefer editing existing files in the codebase. NEVER write new files unless explicitly required.`, it ends with `- To create or overwrite a file, you should prefer the write tool.` instead of the `If you want to create a new file...` paragraph, and it carries the `occurences` typo in the parameter description rather than the description body.

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

The 100,000-character cap deserves a note. `StrReplace` does **not** check `exceededLimit`. It checks `truncated`. Those are different fields produced by different layers: `exceededLimit` is a success record with empty content, and `truncated` is the executor saying it only read part of the file. A file of 99,999 characters with a `limit` supplied would trip `exceededLimit` on a Read but not on this tool, because the tool supplies no `limit`. A file above 8 MiB trips `truncated` and gets `File is too large to edit`. So the practical editable ceiling is the read executor's streaming cap, not the 100,000-character read cap.

That check is the one place the shared edit pipeline is safer here than in `Write`. `Write` runs the same read and the same write executor but never inspects `truncated`, so it will overwrite a file it only partly saw. See the edit-pipeline section of `tool-write.md`. If you build one shared pipeline for both tools, put this check in the shared layer and let `Write` inherit it.

### Result cases

The result is an `EditToolCall` carrying an `EditSuccess` on success. The model-visible text is the `message` field, rendered through a shared renderer.

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

The `<path>` in the success text is the raw argument, not the resolved path. In the diff it is the raw argument too, which means the unified diff headers read `--- a//Users/x/f.ts` with a doubled slash on an absolute path.

Note the two different success grammars. `1 occurrence ... has been updated` is correct, `3 occurrences ... have been updated` is correct, and `replace_all: false` never reports a count even though it always modified exactly one match. There is no trailing space and no diff stat. The sentence is the entire payload.

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

So the shape is `{ path, edits: [{ oldText, newText }, ...] }`, and three things in that schema's own words are decisive:

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

Answers to the questions that make this surface interesting:

| Question | Answer |
|---|---|
| Shape of each edit | `{ oldText, newText }`, both required strings, no `replaceAll` flag anywhere |
| Applied in order or atomically? | **Atomically.** All edits are matched against one snapshot, then spliced in one pass. |
| What happens when one fails? | **Nothing is written.** The throw happens during matching, before the write. All-or-nothing. |
| Do offsets refer to the original or the mutated text? | **The original.** Verified: `{a→b}` then `{b→c}` on a file containing `a` throws `oldText was not found in the file.` for the second edit, because `b` does not exist in the original. |
| Is there a `replace_all` equivalent? | **No.** Duplicate `oldText` in one call yields `edits must not overlap.` rather than a bulk replace. |
| Matching rule | Exact substring; if *any* edit misses, the whole call retries against a fuzzy-normalized copy of the file |
| Fuzzy normalization | `NFKC`, then U+2018/U+2019 to `'`, U+201C/U+201D to `"`, U+2013/U+2014 to `-`, U+00A0 to a space, then `trimEnd()` per line |
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
| Line endings | dominant ending detected (`\r\n` if any `\r\n` present else `\n`), content normalized to LF, then every `\n` rewritten to the ending |

The atomic all-or-nothing array is genuinely a better contract than `StrReplace`'s single edit for a model that wants to make four changes to one file, and it is what Copilot's `multi_replace_string_in_file` and Zed's `edit_file` both do. But there is one shipped bug in it that you must not copy.

**On the fuzzy path, `pi_edit` rewrites the entire file.** `fuzzyNormalize` runs `NFKC`, folds curly quotes to ASCII, folds en and em dashes to hyphens, replaces U+00A0 with a space, and applies `trimEnd()` to every line. The matches are computed against that normalized copy and spliced into *that copy*, which is then written back. Verified on a file with a curly-quoted string, a non-breaking space, and a trailing-space line: an edit that only touched `const s = 'quoted';` also removed the trailing spaces from an unrelated line and replaced a U+00A0 in a third line with a plain space. Input and output, exactly as the reimplementation produced them:

```
input : "const a = 1;   \nconst s = ‘quoted’;\nconst n = 1\u00a0+ 2;\ntarget();\n"
fuzzy : true
output: "const a = 1;\nconst s = 'renamed';\nconst n = 1 + 2;\ntarget();\n"
```

Three lines changed, one of them the intended one. This is precisely the failure a commit in `zhayujie/CowAgent` had to be written to fix, and it is present in shipped Cursor code on the `pi_edit` path. The lesson is not that `pi_edit` is bad - the array shape is good - it is that a normalizing matcher must compute offsets in the original text and splice there, and here it does not.

**Recommendation for a fresh implementation: take `pi_edit`'s parameter shape and reject its matcher.** Confidence: high. The array is the better contract: it makes a multi-site edit one call, one claim, one write, one diff, and it makes atomicity the default rather than something the model has to think about. The matcher should be the `StrReplace` one, run per edit against the original snapshot, with the offsets spliced into the original. That combination - array of edits, exact-then-trimmed matching, offset-based splice into the original - is strictly better than either shipped surface.

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

Step 3 is order critical in a specific way. The lock is taken on the raw argument string, so `C:\a\f.ts` and `c:/a/f.ts` are two different keys and do not serialize against each other. Cursor accepts that. If you key on the canonical path instead, you close a real hole and you gain a normalization step before the claim, which means the claim happens after path validation and a malformed path never takes a lock. That is better, and it is a deliberate divergence.

Step 6 exists because the stream can end without `path` ever completing, which happens when the model emits malformed JSON or gets cut off. The lock variable stays undefined and the call fails with a string that names the actual cause.

The `streamField` is `new_string`, not `path`, in the streaming config:

```js
{ path: "path", streamField: "new_string", fileOperationLockManager }
```

That is what drives the live `editToolCallDelta` carrying `streamContentDelta`, which is how the UI renders a diff while the model is still writing the replacement.

### Layer 2: read, gate, diff, write

The gate is not in one place. `Sbe` calls `W_e(r, o)` as its very first statement, before the read, and `Rbe` calls `W_e(r, i)` again before the write. So the plan-mode check runs before the read *and* before the write, and the first one usually wins. That is the one real difference from `Write`, whose wrapper `Rbe` is reached only after the read and the diff, so a refused plan-mode write there still pays for both. This report's step 1 below is the early check; step 10 is the second one inside `Rbe`.

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

Step 6 preserves a subtle asymmetry you should know about. The engine's own normalizer uses `/\r\n?/g`, which collapses lone `\r` as well as `\r\n`. The wrapper's normalization uses `/\r\n/g`, which collapses only `\r\n`. So for a classic-Mac file with lone `\r` endings:

- the file content keeps its lone `\r`
- the model's `old_string` has its lone `\r` collapsed to `\n`
- the exact scan therefore cannot match across a line boundary
- the trimmed fallback can still match, because `trim()` eats a trailing `\r` at the line end
- the result reports `usedWhitespaceInsensitive: true` for what should have been an exact match

This is a genuine wart, not a feature. Reproduce it or fix it, but do not fix it by accident.

Step 8 uses `contentsLF` as the diff's original, not `original`. On a CRLF file that means the diff compares LF against LF and comes out clean, which is why the diff is not full of phantom changes. The original CRLF text is kept separately as `originalContent` and passed through to `recordFileState` and to `beforeFullFileContent`.

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

Read step 2 carefully. The emptiness test is on `oldRaw`, the raw argument, not on the normalized `oldStr`. `normalizeEol` can never turn a non-empty string into an empty one, so the two are equivalent in practice, but the asymmetry is real: if you ever add a normalizer that *can* empty a string, the guard moves.

Steps 2 and 3 both run before any search. That ordering is the whole reason a de-indent attempt fails and a re-indent succeeds. `"  foo"` to `"foo"` normalizes to two strings that are not equal, so step 3 passes, then the search runs and finds nothing under the exact scan (the file has `foo` at column 0), then the fallback trims both sides to `foo`, they match, and the file becomes `foo`. Wait - that would be a successful de-indent. It is not, because step 3 compares `oldStr` to `newStr` *without* trimming: `"  foo"` is not `"foo"`, so step 3 passes. Let me state the real rule, which is narrower than "normalized equal is rejected".

The identity check is on EOL-normalized forms only. It does not trim. So:

- `old_string: "  foo"`, `new_string: "foo"` -> EOL-normalized forms differ -> step 3 passes -> exact scan finds nothing -> fallback trims and matches -> **the edit succeeds and de-indents the line.**
- `old_string: "foo"`, `new_string: "foo"` -> identical -> step 3 throws.

The parent spec says the de-indent attempt fails with `IdenticalStringsError`. That is wrong, and it is the one place where the spec's worked example contradicts its own pseudocode. The pseudocode is right and the example is wrong. Verified against the shipped matcher: `runSingleEdit({contentsBeforeEdit: "foo\n", old_string: "  foo", new_string: "foo"})` returns `usedWhitespaceInsensitive: true` and produces `"foo\n"`. The outdent direction verifies the same way: `"    foo"` to `"  foo"` against a file line with four spaces produces `"  foo\n"`.

So the accurate statement of the rule is: **a byte-identical `old_string` and `new_string` is rejected, and anything else runs the pipeline.** An EOL difference alone makes them non-identical, so `old_string: "a\r\nb"` with `new_string: "a\nb"` is not an identity error even though the normalized forms are equal - which is why the check uses the normalized forms after all. The two facts hold together: the check is on normalized forms, and trimming is not part of normalization.

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

Step 3's advance is the single most consequential line in the tool. `i += search.length` means overlapping occurrences do not count. Verified:

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

The `lineStart` accumulator uses `crlf ? 2 : 1` for the newline width, and `crlf` is always false here because the engine is always called with `eol = "\n"` and the content has already had `\r\n` collapsed. So +1 is correct for this caller. A caller that passed `"\r\n"` would get wrong offsets unless the content genuinely still had CRLF; since the accumulator and the split disagree in that case, treat `crlf` as vestigial.

The one structural constraint worth restating: the loop bound is `start <= fileLines.length - needle.length`, and there is no inner loop that can consume a differing number of lines. **Strict one-to-one line correspondence.** A three-line search can only match three consecutive file lines. That is why a search missing one intermediate line simply fails rather than matching around the gap, and it is why there is no `...` elision.

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
6. Window 1: `fileTrim[1]` is `"if (x) {"`, which ends with `"if (x) {"`. `startOff = fileLines[1].lastIndexOf("if (x) {")` = 4. Then `fileTrim[1] === "if (x) {"` so `startOff = 0`... no. `fileTrim[1]` is the trimmed line `"if (x) {"`, and `needle[0]` is `"if (x) {"`, so they are equal, so `startOff` is forced to 0. That gives a region starting at the beginning of line 1, which is index 17, not 21.

That is the subtle part, and it is worth pausing on, because it means a whole-line match under the fallback starts at column 0 of the raw line, not at the first non-space character. Consequence: the replaced region includes the file's leading indentation. Replacing it with a tab-indented `new_string` therefore *removes* the file's four spaces and substitutes one tab. Which is exactly what Cursor does, and it is what the parent spec's example shows:

```text
function foo() {
	if (y) {
		return 2;
	}
}
```

Verified by running the shipped matcher: the output is `function foo() {\n\tif (y) {\n\t\treturn 2;\n\t}\n}`. The indentation of the replaced region is the model's, not the file's. Cursor does not re-indent and does not preserve the file's indentation for a matched line. It only preserves the bytes *outside* the matched region.

This is the single most consequential detail in the tool for its usefulness, so state it precisely: **the fallback replaces whole trimmed lines including their original leading whitespace, and inserts `new_string` verbatim. Nothing is re-indented, in either direction.** A model that emits a replacement at the wrong depth produces a file at the wrong depth, and the tool reports success.

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

`needle` is `["if (x) {", "}"]`, two lines. The only windows containing an `if (x) {` are window 1 (`"if (x) {"`, `"return 1;"`) and no others. `startsWith("}")` fails on `"return 1;"`, so no match at window 1. And window 0 is `"function foo() {"` which fails `endsWith`. So the result is zero matches:

```
Error: The string to replace was not found in the file (even after relaxing whitespace).
```

That is the one-to-one line correspondence biting, and it is worth a test vector.

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

The last row is the interesting one: with `replace_all: true` and an empty replacement, the bulk path is skipped (`replacement === ""`), so the sequential path runs twice, and each deletion eats its own line's newline. The result is `b\nb\n`, not `\n b\n\n\n` or anything with blank lines.

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

What a caller gets instead is a hard refusal: if the read executor had to truncate the file, the tool throws `File is too large to edit ...` rather than editing a prefix. That is the right choice, because editing a prefix of a file and writing the prefix back would truncate the file. The refusal exists precisely because there is no way to page an edit.

The one place a boundary could fall mid-character is the read executor's streaming path, which decodes incrementally and cuts the retained text. Its `selectedTruncated` / `truncated: true` is what reaches the tool as `truncated`, and the tool refuses on that flag. So the case is handled by refusing rather than by reassembling, which is safe. Where it could still bite is a file just under 8 MiB with an astral character straddling the cut; the executor's `line.text.slice(0, remainingBytes)` can split a surrogate pair. That produces a lone surrogate in the truncated content, which the tool then refuses anyway, so it is invisible.

That `slice` is also the clearest example of a naming trap in this code path. The local is `remainingBytes`, the accumulator is `selectedBytes`, and the budget is `MAX_TEXT_SIZE`, but every value fed into the comparison is `line.text.length`, which is a JavaScript string length. So the unit is UTF-16 code units and only the names say bytes. A Rust port that reads the names and counts bytes will cut at a different point on any non-ASCII file.

The `truncated` flag has a second producer worth knowing: when the executor's buffer threshold is passed it spills to a temp file, and a failure to create that temp file sets `truncated: true`. So a disk-full condition during the read surfaces as `File is too large to edit`, which is a misleading message for a real disk problem. Worth fixing rather than copying.

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

The mismatch between the 100,000-character read cap and the 8 MiB truncation cap is the one to internalize. They measure different things at different layers, and this tool reacts only to the second.

The exact comparison for the read cap, from the shared read shaper:

```js
content.length > LIe ? { /* exceededLimit: true, value: "" */ } : { /* normal */ }
```

with `LIe = 1e5`. That is a pure length test after windowing, not a byte test and not a line test.

### Q3 Text encoding

**Invalid UTF-8.** `StrReplace` never sees bytes. The read executor decodes and hands the tool a JS string. Its behavior on invalid UTF-8, from the daemon: it tries a strict UTF-8 decoder, and on `InvalidUtf8Error` it falls back to a lossy `TextDecoder("utf-8", { fatal: false })`, and if the lossy result is "mostly valid UTF-8" it uses that, otherwise it falls back to `latin1`. So a file that is genuinely Latin-1 gets decoded as Latin-1, and every byte maps to a character, and a round trip is lossless. A file that is mostly UTF-8 with a few bad bytes gets lossy decoding, and the bad bytes become `U+FFFD` replacement characters, which are then written back. **That is a silent corruption path**, and it is inherited by `StrReplace` because it shares the read stage. It is worth deciding deliberately: refuse rather than lossy-decode, or round-trip as bytes.

**BOM.** The read executor strips a leading U+FEFF and remembers it (`hadLeadingBom`). Whether the write path re-adds it depends on the encoding it settles on: the caching line is `cacheEncoding = decoderConfig.cacheEncoding === "utf8" && hadLeadingBom ? "utf8bom" : ...`, so a BOM'd file is remembered as `utf8bom` and the write encodes it back as UTF-8 with a BOM. So a BOM survives a `StrReplace` edit. The model never sees the BOM, so an `old_string` starting at the very first character of the file matches against content with the BOM already removed. That is the right behavior.

**Unicode normalization.** None. No NFC, no NFD, no NFKC. Two visually identical strings can fail to match if one is composed and the other decomposed, and the fallback does not help because `trim()` does not normalize. This is a real gap: a model reading a macOS-created filename's contents will see whatever the file contains, so the common case is fine, but a model that types a composed accent against a decomposed file fails.

**U+00A0.** `trim()` strips it at line edges, because JS `trim` covers the full `WhiteSpace` set which includes U+00A0. So a non-breaking space at the start or end of a line is neutralized by the fallback. A non-breaking space in the middle of a line is not, and fails. Verified:

```
content "  \u00a0const a = 1;\n", needle "const a = 1;"  -> exact hit, used false
content "x\u00a0y = 1;\n",        needle "x y = 1;"     -> no hit at all
```

The first succeeds because the exact scan finds `const a = 1;` as a substring after the NBSP, not because of trimming.

**Smart quotes.** No folding, in either direction. `‘` does not match `'`. This is Cursor's one matching gap that Claude Code closes, and it is a deliberate difference: Claude Code folds the four quote characters, Cursor does not.

**`\uXXXX` escapes.** No handling. Claude Code tries both directions (unescape the needle, and re-escape real characters into a regex); Cursor does neither. A model that emits `\u00e9` where the file has `é` fails.

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
- If the file on disk has mixed endings, `At` does a line-level LCS alignment against the existing content and restores each line's original ending. That is a per-line merge, not a global rewrite.
- If the file on disk is uniformly CRLF, a blunt `/(?<!\r)\n/g -> "\r\n"` runs.
- Otherwise the LF content is written as-is.

`At` is worth a line of explanation because it is more careful than it looks. It splits both the incoming content and the existing content into `lines` plus per-line `ends` (either `\r\n` or `\n`), runs a patience-style diff to align the line vectors, and then for each incoming line, if its ending is `\n`, substitutes the ending of the aligned existing line. So an edit inside a CRLF file that leaves 99% of lines untouched gets each untouched line's original ending back, and the inserted lines get the ending of the line they were aligned with. That is why a `StrReplace` on a mixed-ending file does not homogenize it.

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

One more hole worth naming: the global `/(?<!\r)\n/g` rewrite on a uniformly-CRLF file will also convert a bare LF that the model deliberately put inside a string literal. A file with a literal newline byte inside a quoted string is pathological, but the regex cannot distinguish it from a line break. A per-line approach cannot fix this either; only a real parser can.

### Q5 Binary detection

Three layers, in order.

**Extension shortcut.** A binary extension list is tested before any read:

```js
const aTe = /\.(zip|tar|gz|exe|dll|so|dylib|bin|mp4|webm|mov|avi|mkv|wmv|flv|m4v)$/i;
```

Case-insensitive, anchored at the end. A hit reaches a verdict without touching the file.

**Content sniff.** For an extension not on the list, the format sniffer reads a sample and decides. It checks, in order: BOM patterns for UTF-16LE, UTF-16BE, UTF-8 BOM, and UTF-32 variants; a heuristic on the first 50 two-byte pairs for UTF-16 without a BOM (`> 80%` of high bytes zero with printable low bytes means UTF-16LE, and vice versa); and then a NUL-byte scan. A NUL in the sample means binary.

**Decode failure.** If neither fires, the decode path above tries strict UTF-8, then lossy, then Latin-1. So a file is only "not text" if the extension list or the NUL scan says so, or if the extension list catches it first.

What the model sees on a binary file:

```
Error: File is not a text file (binary data)
```

produced by the check `if (output.case !== "content") throw new BinaryFileError(...)`. Worth noting that a `dataBlobId` or `contentBlobId` output also lands here, so a file large enough to route through the blob store would be reported as binary if the read stage ever returned a blob instead of inline content. Whether that path is reachable for a text file is not determinable from the bundle; see `## What remains unknowable`.

`StrReplace` does not have its own binary rule. It inherits Read's, which is the right factoring.

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

The last row is worth pinning because it is the one place where an all-whitespace needle behaves differently from what a reader expects: `"   "` is not treated as empty and is not trimmed. Only `""` is refused.

### Q7 Cost

**Memory.** The whole file is held as a JS string twice at peak: `originalContent` (with whatever endings it had) and `contentsLF` (normalized). Then the edit produces a third, and the diff produces a fourth if it is materialized as a string. On a 8 MiB file that is roughly 16 to 48 MiB of UTF-16 (so up to ~96 MiB of actual bytes) before the diff. The 512 KiB `beforeFullFileContent`/`afterFullFileContent` cap only limits what goes into the result record, not what is held in memory.

**Time.** Three passes over the file at minimum: the read, the exact scan, the write. The exact scan is `indexOf` in a loop, which is effectively linear for a needle that does not match often and quadratic in the pathological case of a self-similar needle. The fallback is worse: it splits the file into lines, trims every line, and then does an O(lines × needle-lines) window scan with a `trim()`-comparison inner loop. On a 100,000-line file with a 50-line needle that is 5 million line comparisons, each of which is a string equality. It only runs when the exact scan found nothing, which is the case where a model is retrying, so it runs exactly when the user is already waiting.

**The worst case a hostile repository can force.** A file just under 8 MiB, all on one line, with no trailing newline. Then:

- the read succeeds (under the truncation cap)
- the fallback's `split("\n")` produces one element, so the window loop runs once, but each `trim()` on a 8 MiB string copies it, so you allocate 8 MiB twice
- if the needle is a long repeated substring, the exact scan's `indexOf` loop is quadratic

Actually the pathological `indexOf` case is bounded by the non-overlapping advance: a needle of length L in a file of length N does at most N/L full searches, each of which is O(N) worst case, so O(N²/L). For L = 1 that is O(N²). A 8 MiB file of a single repeated character with a 1-character needle is the worst case, and it is also the case where the scan succeeds on the first hit and stops counting... no, it does not stop counting; it collects every match before deciding. So it builds an array of 8 million match objects. That is the real hostile case: memory, not time.

**Recommendation.** Cap the number of matches collected. You only need to know whether there are 0, 1, or more than 1 matches for the `replace_all: false` path, and for `replace_all: true` you need all of them, so cap `replace_all` too or stream the splice. Cursor does neither, and a model that sends `old_string: "e"` with `replace_all: true` on a large file will allocate a match object per `e`.

### Q8 Paths

**Relative or absolute.** The default branch's `path` description says `The absolute path to the file to modify`, but nothing enforces it. The path goes straight into the read request:

```js
const m = "path" in o ? o.path : o.file_path;
const h = new Ol.g$({path: m, streamContent: o.new_string});
```

No resolution, no `realpath`, no workspace join. Whatever the model sends is what the read executor receives, and the executor resolves it against the workspace if it is relative. The `dsv3-1018` branch documents this honestly: `You can use either a relative path in the workspace or an absolute path.`

**`..`.** Not handled by the tool. The read executor resolves and the capability service decides whether the result is inside the workspace. The tool neither knows nor cares.

**Symlinks.** Resolved by the read executor, and again by the write executor. The tool does not resolve. `StrReplace` on a symlink edits the target, because both the read and the write follow. There is no no-follow flag on this path, unlike `Write`'s open flags. So a symlinked leaf is followed by `StrReplace` and refused by `Write`, which is an inconsistency between two tools that share the same write executor.

**Paths outside the workspace.** Allowed if the executor allows them. There is no worktree-escape check in this tool's own code; the worktree guard lives in the read/write executors and surfaces as a sentinel error that `StrReplace` rethrows verbatim rather than mapping.

**Windows drive letters, UNC, case sensitivity.** Nothing special in the tool. The lock keys on the raw string, so `C:\a\b.ts` and `c:\a\b.ts` are different keys on a case-insensitive filesystem, which means two calls can hold "the lock" for the same file simultaneously.

**Normalization order.** The path is used for the lock *before* any normalization (step 3 of layer 1), and used for the read and write *without* normalization. So the lock key and the operation target are both the raw string, and they agree with each other. The divergence is between two different raw strings that name one file.

**The `worktries` repair.** This is a real quirk adjacent to paths. The write path runs the argument through:

```js
const cke = /(^\/(?:Users|home)\/[^/]+\/\.cursor\/)worktries(?=\/|$)/g;
function lke(e) { return e.replace(cke, "$1worktrees"); }
```

So a path under a `.cursor/` directory with a `worktries` segment gets that segment rewritten to `worktrees`. The success message uses the repaired path. That is a targeted repair for one misspelling in one directory layout, and a fresh implementation has no reason to carry it.

### Q9 Concurrency

**What is locked.** A per-path mutex from `fileOperationLockManager`, acquired during argument streaming as soon as the `path` field completes (layer 1, step 3), and released in `finally` (step 8).

**Keyed on what.** The raw argument string, before any normalization. See Q8.

**For how long.** From the moment the path field completes in the stream, through the read, the engine, the diff, and the write. So the lock covers the entire read-modify-write window inside this tool.

**What race that closes.** Two `StrReplace` calls in the same turn on the same path cannot interleave their read-modify-write sequences. Without the lock, call A could read, call B could read, A could write, and B could write, losing A's edit. This is the classic lost update, and the lock closes it for calls within one agent loop.

**What races it leaves open.**

1. **Across agents.** The lock manager is a per-turn instance, not a singleton. Ten `new lne` sites, all method-local; two matter, one per turn and one per step. Two chat tabs, two background agents, or a parent and a subagent each have their own empty lock set. So the lost update is fully available across agents.
2. **Across processes.** No OS-level lock at all.
3. **Against a non-`StrReplace` writer.** `Shell` takes an *exclusive* lock, so a shell command does serialize against edits. But an external editor, a file watcher, or a `git checkout` does not.
4. **Spellings.** Two spellings of one path do not serialize.

**The read-modify-write window and TOCTOU.** Inside the tool the lock covers it, so there is no internal TOCTOU. But there is no content check either: the tool reads, computes, and writes without re-verifying that the file still contains what it read. Compare `Write`, which computes a sha256 fingerprint of the content it read and re-verifies it immediately before the truncate, refusing with `{path} changed on disk after this tool call read it...` if it moved. `StrReplace` has nothing equivalent. Its only defense is that a concurrent change usually invalidates the anchor.

That defense is weak in exactly the case that matters. Consider a file where the model's anchor is in a region the concurrent writer did not touch. The concurrent writer appends a function, or reformats a different part of the file. `StrReplace`'s read happened before that write; its own write happens after; the anchor still matches; the edit lands on top of a file that has changed, and the tool's write clobbers the concurrent change entirely, because `StrReplace` writes the whole file. So a concurrent modification outside the matched region is silently lost.

**What a Rust implementation should do differently.** Four things, in order of value.

1. **Compare-and-swap on a content hash.** Re-read or hash immediately before the write and refuse if the content is not what the engine matched against. This is `Write`'s existing check, applied here. Cost: one extra read, or one hash of content already in memory plus a stat-based shortcut. Value: closes the silent clobber.
2. **A splice operation in the VFS, not a whole-file write.** If the backing store can apply a byte-range replacement, the concurrent-change window narrows to the range. This is the real argument for Option C in decision 1.
3. **Atomic write.** Temp file plus rename. Cursor truncates in place, so a crash mid-write leaves a partially written file.
4. **A lock keyed on the canonical path, and scoped above the agent.** If multiple agents share a process, the per-agent lock is not enough, and the content check is the only thing that saves you.

There is one more concurrency behavior worth knowing because it looks like a race and is not: `StrReplace` is in the read-retry whitelist. If a `Read` in the same step misses, it waits up to 120,000 ms for earlier calls in this set to settle and retries once. So a read issued in the same model response as an edit will wait for the edit rather than racing it. The set is:

```js
["ApplyPatch","AwaitShell","CopyToBox","EditNotebook","MultiEdit","MultiStrReplace","Shell","StrReplace","Write","edit_notebook","run_terminal_cmd","search_replace","write"]
```

Note `MultiStrReplace` in there. There is a `MULTI_STR_REPLACE` tool id in the bundle's wire-case map (`MULTI_STR_REPLACE: "edit_tool_call"`) and in its team map (`MULTI_STR_REPLACE: "agent-quality"`), but **there is no `AG("MULTI_STR_REPLACE", ...)` registration in this build**. The id is referenced by the read-retry list and by two lookup tables, and nothing registers it. So a multi-edit tool existed at some point, or is behind a flag that is off, and in this build it is absent. That is a finding, not a documented feature.

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

The two silent-rewrite rows are the ones to decide about. The mixed-ending per-line merge is genuinely careful and worth copying. The lossy-decode path is a corruption bug and worth refusing instead.

**The naive-implementation trap.** An implementation that does `content.split('\n').map(trim)` to match, then `join('\n')` to write, will re-indent every line in the file. Cursor avoids this because the matcher returns offsets into the original string and the splice touches only those offsets. This is the single most important thing to get right, and the test is one line: edit a file whose untouched lines have trailing spaces and mixed indentation, then assert the untouched bytes are byte-identical to the input.

### Q11 Determinism

**Output text.** Fully deterministic. The success sentence is a pure function of the path, `replace_all`, and the count. The failure sentences are pure functions of the case and the flag.

**Match order.** The exact scan is left-to-right by construction. The fallback scans windows in ascending `start` order. The splice sorts descending by start. All deterministic.

**The sort.** `matches.sort((a, b) => b.start - a.start)` is a numeric comparator, so it is a total order except for equal starts, which cannot happen for disjoint matches. Where two matches do share a start, which the fallback can in principle produce, the tie is unresolved and `Array.prototype.sort` stability decides. In practice the fallback's window loop pushes at most one match per window, so equal starts do not arise.

**The fuzzy hint.** Deterministic but tie-prone: `[...lines].sort((a, b) => b.length - a.length)[0]` picks the longest line, and among equal-length lines the sort is stability-dependent. A different JS engine could pick a different anchor line for a symmetric `old_string`. That does not change whether the edit applies, only which region the hint shows.

**The diff.** Myers, so deterministic for a given input pair. `linesAdded` and `linesRemoved` come from filtering hunk lines for a leading `+` or `-`, which is deterministic.

**Byte-identical reruns.** Yes for the file content and the model-visible text, given the same input bytes and the same `replace_all`. The variable is the line-ending restoration, and only if the file is being changed between the two runs.

So: two identical runs on the same tree produce byte-identical files and byte-identical messages. The one non-determinism is the fuzzy hint's anchor choice on a tie, which is cosmetic.

### Q12 Truncation and omission

The tool truncates nothing in its model-visible output, because its model-visible output is one sentence. That is the short answer, and it is true: there is no marker, no count, no "use offset to see more", and no way for the success message to lie about completeness.

But there are four places where information is dropped without the model being told.

1. **`beforeFullFileContent` / `afterFullFileContent`** are set to `undefined` when either side exceeds 524,288 characters. Those fields are client-only, so no model-visible lie, but a UI that renders a diff from them shows nothing and the model is unaffected.
2. **The diff itself** is skipped entirely when `omitEditDiffString` is set, in which case `diffString: ""`, `linesAdded: 0`, `linesRemoved: 0`. Again client-only.
3. **The fuzzy hint's window** is 2 lines before and needle-lines + 4 total, clipped at the ends of the file. A model reading the hint sees a region that may not include the actual mismatch, and there is no marker saying the region was clipped. It is labelled as a possible match, so it does not claim completeness.
4. **`linesAdded` / `linesRemoved`** are computed and carried on the record but never reach the model. The model gets the sentence.

So the honest answer to the question "does the tool lie about completeness anywhere" is: to the model, no. To a UI reading the proto fields without checking, three fields can be silently empty or zero, and there is no flag distinguishing "the diff was omitted because there was none" from "the diff was omitted because a flag suppressed it."

One more omission worth naming: the **success message does not say whether the fallback was used.** A model that sent a tab-indented replacement into a space-indented file has no way to learn that its replacement went in verbatim. The telemetry counter `agent.tools.str_replace.whitespace_insensitive` records it, and the model sees nothing. That is information the harness has and discards, and it is the strongest argument for appending a short excerpt to the success text.

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

The last two are the honest entries in this matrix: one failure misreports its cause, and one success corrupts data. Both are inherited from the read and write stages rather than produced by the engine.

## Traps

Each of these is a place a faithful-looking implementation goes wrong, with the test that catches it.

### Trap 1: rebuilding the file from trimmed lines

**What goes wrong.** The implementation splits the file into lines, trims each one to compare, then joins the matched region back and writes. Every untouched line loses its trailing whitespace, and any line whose leading whitespace was normalized loses that too. A single-line edit to a 3,000-line file rewrites 3,000 lines.

**Why it is easy.** The trimmed-line matcher naturally produces trimmed lines, and joining them is the shortest path from "I found the region" to "I have the new file". It looks like the same operation.

**The test.** File with three lines: `"\tconst a = 1;  \n    const b = 2;\t\nconst c = 3;\n"`. Edit `b = 2` to `b = 9` with an `old_string` that needs the fallback. Assert the result is exactly `"\tconst a = 1;  \n    const b = 9;\t\nconst c = 3;\n"` - same tabs, same trailing spaces, same order on the untouched lines. Assert the byte length of the untouched prefix and suffix separately.

This is the `zhayujie/CowAgent` commit in one sentence: locate the match in the original content and return offsets into it, so only the matched region is replaced. Cursor gets it right by construction, and `pi_edit` gets it wrong on its fuzzy path, which proves it is possible to ship both ways.

### Trap 2: advancing the exact scan by one

**What goes wrong.** `i += 1` instead of `i += oldString.length` reports two matches for `aa` in `aaa`. With `replace_all: false` that throws the "multiple times" sentence instead of the overlap sentence. With `replace_all: true` it produces `XX` instead of `Xa`.

**Why it is easy.** Advancing by one is the standard "find all occurrences including overlapping" idiom, and a reader who has not thought about the overlap guard will consider it more correct.

**The test.** Three assertions on a file of exactly `aaa`:
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

**Why it is easy.** It feels like the helpful thing to do, and several editors' AI features do exactly this. Zed's `edit_file` deliberately does re-indent, with a whole `Reindenter` and an indent-delta computation, which makes it look like the standard approach.

**The test.** File `"function f() {\n    if (x) {\n        return 1;\n    }\n}"`. Send `old_string` with tabs for the three inner lines and `new_string` with **two** spaces of indentation. Assert the result contains exactly `"\n  if (y) {\n    return 2;\n  }\n"` - two spaces, not four, because that is what the model sent. The fallback matched, and the replacement is verbatim.

### Trap 5: applying the whole-file LF normalization and forgetting to restore

**What goes wrong.** The engine works on LF and the result is written back as LF. A CRLF file becomes an LF file, and every line shows as changed in `git diff`.

**Why it is easy.** The normalization is necessary and the restoration is a layer away, in the write executor. If you implement the engine and the write as two independent pieces, the restoration is easy to omit.

**The test.** File `"a\r\nb\r\nc\r\n"`. Edit `b` to `B`. Assert the file on disk is `"a\r\nB\r\nc\r\n"` - byte for byte, four `\r\n` sequences, none missing and none added.

And the mixed-ending variant: file `"a\r\nb\nc\r\nd\n"`. Edit `b` to `B`. Assert `"a\r\nB\nc\r\nd\n"` - every untouched line kept its original ending, including the bare LF on line 2.

### Trap 6: treating the empty replacement as a normal splice

**What goes wrong.** `new_string: ""` splices the match away and leaves the newline, so deleting a whole line leaves a blank line where it was. The model then issues a second edit to remove the blank line, and the turn costs twice as much.

**Why it is easy.** The general splice handles the empty string correctly as a replacement; what it does not do is look at whether anything else shares the matched line.

**The test.** File `"line1\nline2\nline3\nline4"`. `old_string: "line2"`, `new_string: ""`. Assert the result is `"line1\nline3\nline4"` - three lines, not four with a hole. Then the same on the last line: `old_string: "line4"`, `new_string: ""` on `"line1\nline2\nline3\nline4"` -> `"line1\nline2\nline3"`, no trailing newline, no trailing blank line.

And the counter-test: a match sharing its line with other content must not eat the line. `old_string: "b"`, `new_string: ""` on `"abc\n"` -> `"ac\n"`, not `"\n"`.

### Trap 7: assuming the fallback is more permissive than the exact scan in every respect

**What goes wrong.** The fallback is stricter in several ways, and each surprises.

- **A single-line search must equal a whole line after trimming.** `startsWith` and `endsWith` are overridden by the single-line rejection: `if (isFirst && ok && ft !== needle[n]) continue outer`. So a one-line needle can only match a line whose trimmed form is *exactly* the needle. `"const a = 1;"` will not match `"    const  a = 1;"` (an extra internal space makes the trimmed line longer), and it will not match `"const a = 1; // note"` either. Note this only matters when the exact scan failed first: `"a = 1"` against `"    const a = 1;"` succeeds immediately as a substring, never reaching the fallback.
- **Internal whitespace is never collapsed.** `"foo  bar"` does not match `"  foo bar"`, even though both trim to the same shape, because the trimmed forms still differ by one space.
- **A single trailing newline in `old_string` becomes "and the next line begins as an empty prefix".** Under the fallback, a trailing `\n` produces a trailing empty needle line, whose `startsWith("")` is always true and whose `endOff` is `indexOf("") + 0 = 0`. So the match ends at column 0 of the following line, which means it *includes* the newline after the matched text but does not consume anything from the next line. Verified: `"\tb\n"` against `"a\n    b\nc\n"` matches `"    b\n"`, and replacing with `"X"` gives `"a\nXc\n"` - the replacement is glued to `c` with no separator.
- **Two trailing newlines require a genuinely blank following line.** With two, the needle has two empty lines at the end. The first is matched by the `startsWith("")` rule ending at column 0 of the next line; the second must correspond to another following line whose trimmed form is empty. So `"b\n\n"` matches `"b\n\n"` in the file and not `"b\nc\n"`.
- **A blank line in the middle of `old_string` must correspond to a blank line in the file.** No blank-line tolerance at all. `"b\n\nc"` matches a region with a blank third line, and fails against a region where `b` and `c` are adjacent.
- **A leading newline in `old_string` produces the mirror case.** The first needle line is empty, `endsWith("")` is true, and `startOff = rawLine.lastIndexOf("") = rawLine.length`, so the match begins at the end of the preceding line - i.e. on the newline itself. Verified: `"\nb"` against `"a\n    b\nc\n"` matches `"\n    b"`.

**Why it is easy.** "Whitespace insensitive" suggests a superset relation. It is not; it is a different relation with its own strictness.

**The test.** Six assertions:
- `"    const  a = 1;"` in the file, `"const a = 1;"` as the needle -> not found after relaxation.
- `"const a = 1; // note"` in the file, `"const a = 1;"` as the needle -> not found after relaxation, because the trimmed line is longer.
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

**What goes wrong.** If only the needle's `\r\n` is collapsed and the file's is not, an LF needle against a CRLF file goes through the fallback instead of matching exactly. That is mostly invisible, except: the reported `usedWhitespaceInsensitive` flips, the "after relaxing whitespace" suffix appears on errors, and the fallback's stricter rules (trap 7) apply where the exact match would have succeeded.

**Why it is easy.** The two normalizations are in different functions at different layers, with different regexes.

**The test.** CRLF file `"function f() {\r\n    return 1;\r\n}\r\n"` with an LF `old_string` of `"    return 1;\n"`. Assert the edit succeeds **and** that the code path taken was exact, not fallback. The observable proxy is a counter or a returned flag; assert on the flag.

### Trap 10: using `/\r\n?/g` on the file content

**What goes wrong.** Collapsing lone `\r` in the file content makes a classic-Mac file look like an LF file to the matcher but not to the writer, so line endings can be lost or duplicated. Cursor uses `/\r\n/g` on the content and `/\r\n?/g` on the model strings, and the asymmetry is what makes a lone-`\r` file unmatchable across lines rather than mishandled.

**Why it is easy.** One regex is tidier than two.

**The test.** File `"a\rb\rc\r"`, `old_string: "a\nb"`, `new_string: "X"`. Cursor's behavior is not-found. If you deliberately fix the hole, assert success and assert the file is `"X\rc\r"` or `"X\r\nc\r\n"` - decide which, and pin it. What you must not do is produce `"Xc\r"` with a lost separator.

### Trap 11: forgetting that `replace_all` has no schema default

**What goes wrong.** Reading `args.replace_all` directly instead of normalizing with `=== true`. With a plain truthy test, a string `"false"` or a number `0` behaves differently. With a `Boolean` coercion, `"false"` becomes `true`.

**Why it is easy.** The description says `default false`, so a reader assumes the default is applied.

**The test.** Send `replace_all` absent -> treat as false. Send `replace_all: false` -> false. Send `replace_all: true` -> true. Send `replace_all: "true"` -> schema validation rejects it as not a boolean, so the tool never sees it; assert the refusal rather than a behavior.

### Trap 12: copying `pi_edit`'s fuzzy path wholesale

**What goes wrong.** The fuzzy path matches against a whole-file NFKC-normalized copy and splices into that copy. Every untouched line gets NFKC, quote folding, dash folding, NBSP replacement, and `trimEnd()` applied, and then the whole file is written back. Three lines change when one was intended.

**Why it is easy.** `pi_edit`'s array shape is genuinely better and it is tempting to take the whole implementation. The normalization looks like a feature.

**The test.** File with a curly-quoted string, a U+00A0 in a third line, and a trailing-space line. Send an edit whose needle requires the fuzzy path (an ASCII apostrophe against the curly quotes). Assert the output differs from the input **only** inside the intended region: the trailing spaces survive, the U+00A0 survives, and the curly quotes outside the region survive. Cursor's `StrReplace` passes this; `pi_edit` fails it.

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

## How three harnesses do this

### Claude Code

**Source.** `C:\Users\Vinnie\cursor\cabinet\_scratch\claude-code\tools\edit.js`, plus `read-file-state.js` for the state store and `error-strings.json` for the exact strings.

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

**Output shape.** A real snippet, not a sentence:

```
The file <path> has been updated successfully.
```

with, when `replace_all`:

```
The file <path> has been updated. All occurrences were successfully replaced.
```

and optional trailing notes for `userModified`, `staleRecovered`, and `contentNotInModelContext`. Note the period placement differs between the two branches - `has been updated. All occurrences...` versus `has been updated successfully.` - which is the kind of thing that only matters if you are diffing transcripts.

**Matching rule.** Exact `includes`, non-overlapping advance identical to Cursor's. Then quote folding on four characters (`\u2018 \u2019 \u201C \u201D`), offset-preserving. Then `\uXXXX` unescaping in the needle, and then the reverse: build a `\uXXXX` regex from real characters and match escaped source. No whitespace fallback at all.

**Gating.** A real read-before-write state machine. `readFileState` is an LRU keyed by normalized path, holding `content` (Bun.hash, base36), `timestamp`, `offset`, `limit`, `isPartialView`, and optionally `contentNotInModelContext` and `staleRecovered`. The gate is:

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

There is a `staleRecovered` escape hatch: if the `old_string` still applies to the changed content, the edit is allowed to proceed and the result carries a note that the file contains other changes not in the model's context. That is a genuinely good idea, and Cursor has no analogue because it always reads fresh.

**The one design idea worth taking.** The `staleRecovered` recovery. A stale read is not always a reason to refuse: if the anchor still resolves uniquely, applying the edit is very likely correct and refusing costs a full read plus a turn. Refusing only when the anchor is gone or ambiguous is the right threshold, and it is strictly better than either "always refuse" or "never check".

**The one design idea worth rejecting.** The quote folding. Folding `‘` to `'` means a model's ASCII apostrophe silently matches a curly one in the source, and then the replacement is written with whatever the model sent. On a file that deliberately uses typographic quotes, an edit can silently typographize or de-typographize a region. Cursor's refusal to fold is the safer default, because a failed match is recoverable and a silent character substitution is not.

### VS Code Copilot

Copilot is the richest source here because it has three edit surfaces plus a repair pass, and they are all in the same directory.

**Source.** `C:\Users\Vinnie\src\vscode\extensions\copilot\src\extension\tools\node\`.

**The three surfaces.**

| Tool | File | Shape |
|---|---|---|
| `replace_string_in_file` | `replaceStringTool.tsx` + `abstractReplaceStringTool.tsx` | `{ explanation, filePath, oldString, newString }` |
| `multi_replace_string_in_file` | `multiReplaceStringTool.tsx` | `{ explanation, replacements: [{ filePath, oldString, newString, explanation }] }` |
| `insert_edit_into_file` | `insertEditTool.tsx` | a natural-language instruction, not strings |
| `apply_patch` | `applyPatchTool.tsx` | a patch document, including `*** Delete File:` |

Note the naming: `filePath`, `oldString`, `newString` in camelCase, and `explanation` is a required field on the single-file tool's TS interface. That is a different parameter set from Cursor's, not a spelling variant.

**Parameters, single-file tool.**

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `explanation` | string | yes in the interface | Cursor has no such field |
| `filePath` | string | yes | absolute path, per the description |
| `oldString` | string | yes | "MUST be the exact literal text" |
| `newString` | string | yes | - |

**The description is worth quoting** because it is the most prescriptive edit description in the set:

```
This is a tool for making edits in an existing file in the workspace. For moving or renaming files, use run in terminal tool with the 'mv' command instead. For larger edits, split them into smaller edits and batch them with multi_replace_string_in_file in a single call. Before editing, always ensure you have the context to understand the file's contents and context. To edit a file, provide: 1) filePath (absolute path), 2) oldString (MUST be the exact literal text to replace including all whitespace, indentation, newlines, and surrounding code etc), and 3) newString (MUST be the exact literal text to replace `oldString` with (also including all whitespace, indentation, newlines, and surrounding code etc.). Ensure the resulting code is correct and idiomatic.). Each use of this tool replaces exactly ONE occurrence of oldString.

CRITICAL for `oldString`: Must uniquely identify the single instance to change. Include at least 3 lines of context BEFORE and AFTER the target text, matching whitespace and indentation precisely. If this string matches multiple locations, or does not match exactly, the tool will fail. Never use 'Lines 123-456 omitted' from summarized documents or ...existing code... comments in the oldString or newString.
```

"at least 3 lines of context BEFORE and AFTER" is an unusual instruction: it pushes toward larger anchors than Cursor does, and it is the opposite of Claude Code's newer "keep `old_string` minimal, usually 1-3 lines" guidance. Two products giving opposite advice on anchor size is a real signal that neither has measured it.

**Caps.** No explicit character or line cap on the edit itself. Two relevant thresholds:

- `NoChangeError` when `updatedFile === originalFile`
- `trySimilarityMatch` skips when `oldStr.length > 1000`, when `oldStr` has more than 20 lines, or when the file has more than 1000 lines

The output is capped by the renderer's token budget, passed as `tokenizationOptions ?? { tokenBudget: 5000, countTokens: (t) => Promise.resolve(t.length * 3 / 4) }`.

**Matching rule.** Four strategies in order, in `findAndReplaceOne`:

1. `tryExactMatch` - non-overlapping `indexOf` scan, same as Cursor's.
2. `tryWhitespaceFlexibleMatch` - split on `eol`, trim each line, and append a `''` to the needle with the comment `// trailing newline to match until the end of a line`. Then a window comparison on the joined trimmed lines.
3. `tryFuzzyMatch` - build a regex where each line is escaped literally and followed by `[ \t]*\r?\n`, so only trailing whitespace and newline style are flexible.
4. `trySimilarityMatch` - a Levenshtein similarity with a default threshold of `0.95`, applied per line in a sliding window, only for needles under 1,000 characters and 20 lines and files under 1,000 lines.

So Copilot has *four* strategies where Cursor has one fallback, and the fourth has a real similarity threshold. This is the concrete counterexample to any claim that nobody uses thresholds; Microsoft ships 0.95.

The whitespace strategy has a detail Cursor does not: it appends an empty string to the needle so that a search matches "until the end of a line", and it computes `startIdx` / `endIdx` from line positions via `OffsetLineColumnConverter` with `endIdx = convert.positionToOffset(end) - 1` and the comment `// -1 to include the last EOL`. It also trims `oldStr` and `newStr` before splitting (`oldStr.trim().split(eol)`), which Cursor never does.

**Healing.** `editFileHealing.tsx` is a repair pass that runs when the first attempt produced `NoMatchError` and the model is flagged as needing healing. It tries, in order: unescape the needle (the Gemini over-escaping bug), then ask a `copilot-utility-small` endpoint to produce the corrected `oldString` given the file content, then ask it to adjust `newString` to match the corrected `oldString`. Both LLM calls have JSON schemas and both fall back to the original on any failure. There is also `trimPairIfPossible`, which trims the needle if the trimmed form matches the expected number of times and trims `newString` to match.

**Output shape.** `EditFileResult` rendered from `fileResults`, with per-file entries carrying `operation`, `uri`, `isNotebook`, `existingDiagnostics`, `error`, and `healed`. The unified diff is streamed separately into the chat as a markdown code block with a `diff:` language tag and a `<vscode_codeblock_uri>` line. So Copilot's model-visible result includes diagnostics and a heal note, and Cursor's is one sentence.

**The one design idea worth taking.** `_errorConflictingEdits`. When two edits in one multi-replace call land on the same file, it sorts all their `textEdits` by start position and refuses if any two overlap:

```js
const hasOverlap = allEdits.some((e2, i) => {
  if (i === 0) { return false; }
  const e1 = allEdits[i - 1];
  return !e1.range.end.isBeforeOrEqual(e2.range.start);
});
```

with the error `Edit at index ${i} conflicts with another replacement in ${path}. You can make another call to try again.` Cursor's `pi_edit` also checks overlap (`edits must not overlap.`), and Cursor's `StrReplace` cannot have the problem because it takes one edit. The idea is worth taking because an array-shaped tool without an overlap check silently produces nonsense: two edits whose ranges touch can double-apply or lose text depending on splice order.

**The one design idea worth rejecting.** The similarity threshold. 0.95 on a per-line Levenshtein is high enough that it mostly catches whitespace and case differences, which the earlier strategies already handle, and low enough that it can match a line the model did not mean. It also has a quadratic cost per window. Cursor's decision not to score similarity is the better one, and the Aider history (a 0.8 threshold shipped and then disabled behind a bare `return`) is the evidence.

### Zed

**Source.** `C:\Users\Vinnie\src\zed\crates\agent\src\tools\edit_file_tool.rs` (120 KB), `edit_session.rs` (45 KB), `edit_session/streaming_fuzzy_matcher.rs`, `edit_session/reindent.rs`, `tool_permissions.rs`.

**Tool name.** `edit_file`. There is no `StrReplace` equivalent, and this is the most interesting divergence in the set.

**Parameters.**

| Parameter | Type | Required | Notes |
|---|---|---|---|
| `path` | `PathBuf` | yes | must start with a project root, or be under `~/.agents/skills` |
| `edits` | `Vec<Edit>` | yes | `deserialize_maybe_stringified`, so a JSON string containing the array is also accepted |

and each `Edit`:

| Field | Type | Required | `.describe()` in the doc comment |
|---|---|---|---|
| `old_text` | string | yes | "The exact text to find in the file. This will be matched using fuzzy matching to handle minor differences in whitespace or formatting." |
| `new_text` | string | yes | "The text to replace it with" |

So `edit_file` is already an array-of-edits tool, like `pi_edit` and Copilot's `multi_replace_string_in_file`.

**Caps.** No character or line cap on the edit. The permission gate is `tool_permissions.rs`. The session opens a real `Buffer` in the project, so the file's language server, diagnostics, and dirty state are all live.

**The edit session, which is the whole design.** `EditSession::new` resolves the path to a `ProjectPath`, opens a buffer, and then:

```rust
let file_changed_since_last_read = ensure_buffer_saved(&buffer, &abs_path, mode, &context, event_stream, cx).await?;
```

`ensure_buffer_saved` compares the action log's `file_read_time(abs_path)` against the buffer's `disk_state().mtime()`, and if they differ it sets `file_changed_since_last_read`. It also handles a dirty buffer by prompting the user with Save / Discard / Keep.

Edits are then processed **as they stream in**, not after the call completes. `process_streaming_edits` loops over `ToolInput::Partial` payloads and feeds each `PartialEdit` into `session.process_edit`, which runs the events through `EditPipeline::process_event`. So the buffer is edited incrementally while the model is still emitting `new_text`, and the UI shows a live diff through a real `Diff` entity.

The old text resolution is `StreamingFuzzyMatcher`, and it is worth describing because it is a genuinely different matcher:

- `finish()` first tries `find_exact_matches`, a KMP scan over the rope that **detects overlapping matches on purpose**, with `MAX_EXACT_MATCHES = 2` because "two matches are enough to prove ambiguity". So Zed's overlap handling is the opposite of Cursor's: Zed counts overlapping occurrences, so `aaaa` against `aaaaa` yields two ranges `[0..4, 1..5]` and is reported as ambiguous.
- If there is no exact match, it runs a line-level dynamic program over `(query_line, buffer_line)` pairs with costs `REPLACEMENT_COST = 1`, `INSERTION_COST = 3`, `DELETION_COST = 10`, and `fuzzy_eq` as a normalized-Levenshtein check at `THRESHOLD = 0.8` per line.
- It accepts a fuzzy window only if `matched_ratio >= 0.8`, where `matched_ratio = line_pairs.len() / max(matched_buffer_row_count, query_line_count)`. That ratio is what allows a query to **skip buffer lines**: the test `test_line_pairs_skip_unmatched_buffer_line` proves a four-line query matches buffer rows `3, 5, 6, 7` with `self.extra` at row 4 skipped.
- Ambiguity is resolved by a `line_hint` with `LINE_HINT_TOLERANCE: u32 = 200`, so multiple fuzzy matches within 200 lines of the hint pick the nearest.

So Zed has fuzzy matching with a similarity threshold, line-skipping, and a proximity tiebreak, all three of which Cursor rejects.

**Re-indentation.** `Reindenter` with separate `first_line_delta` and `rest_delta`, computed from `compute_indent_delta(buffer_indent, query_indent)` and `compute_rest_indent_delta(...)`. It adjusts the leading whitespace of each line of `new_text` by the delta between the buffer's indentation and the query's, clamping at zero, and choosing spaces versus tabs based on whether the buffer's indent has any tabs. So Zed *does* re-indent, deliberately, and it has the tests to prove the edge cases (outdent tabs, clamp negative, whitespace-only lines pass through verbatim, a distinct first-line delta when the model stripped only the first line's indent).

**Failure text.** Three, all of which name the index:

```
Could not find matching text for edit at index {}. The old_text did not match any content in the file.{} Please read the file again to get the current content.
```

with `{}` filled by `" The file has changed on disk since you last read it."` when the mtime check fired, and:

```
Edit {} matched multiple locations in the file at lines: {}. Please provide more context in old_text to uniquely identify the location.
```

That second one lists the actual line numbers, which neither Cursor nor Claude Code does. It is a better hint than Cursor's, because it tells the model exactly how many places and where.

**Output shape.** `EditSessionOutput::Success` carries `input_path`, `new_text` (the whole new file), `old_text` (the whole old file), and `diff` (a unified diff). The `Display` impl renders `Edited {path} successfully`, and on error `{error}\nEdited {path}:\n\n```diff\n{diff}\n````, so a failed edit that still applied some earlier edits shows the partial diff. The model gets the full file contents in the proto even though the rendered text is short.

**The one design idea worth taking.** The session, specifically two properties of it. First, **the edit applies to a live buffer, not to bytes just read**, so the file's language server sees the change and diagnostics are live. Second, **edits are applied as they stream**, so the UI shows the change being written and a user can interrupt. Neither is available to a stateless string-in-string-out tool.

The second property that is worth taking even in a stateless design is **naming the indices and line numbers in the failure text**. `Edit 2 matched multiple locations in the file at lines: 41, 97, 210.` is strictly more useful than `The string to replace was found multiple times in the file.` and costs one line of code.

**The one design idea worth rejecting.** The 0.8 per-line similarity with line-skipping. It is powerful and it is also the design that lets a query silently match a region with a line missing, which is the failure mode Aider's maintainer cited when disabling Levenshtein matching: it let the model silently delete semantically significant lines. Zed mitigates it with the `matched_ratio >= 0.8` threshold and the ambiguity check, but the mitigation is a threshold on a heuristic, and Cursor's answer - refuse and show the neighbourhood - is the one that cannot silently delete a line.

The re-indentation is a closer call. Zed's version is careful and well tested, and it produces better-looking diffs when a model copies a fragment without its leading indent. But it also means the model's bytes are not the bytes that land, which is a class of surprise Cursor's design does not have. My read: take Zed's re-indentation only if you also return the changed region in the result, so the model can see what it actually got. Without that, verbatim insertion is safer.

### Comparison across all four

| Dimension | Cursor `StrReplace` | Claude Code `Edit` | Copilot `replace_string_in_file` | Zed `edit_file` |
|---|---|---|---|---|
| parameter shape | `path`, `old_string`, `new_string`, `replace_all` | `file_path`, `old_string`, `new_string`, `replace_all`, plus an alias layer | `explanation`, `filePath`, `oldString`, `newString` | `path`, `edits[]` of `{old_text, new_text}` |
| multi-edit in one call | no (a `MULTI_STR_REPLACE` id exists but is unregistered) | no | yes, via `multi_replace_string_in_file` | yes, natively |
| paging | none; refuses a truncated file | none; 1 GiB editable ceiling | none | none |
| cap and unit | 8 MiB read truncation (UTF-16 code units); 100,000 read chars not checked | 100,000 result chars; 1 GiB file (bytes) | none on the edit; 1,000 chars and 20 lines to attempt similarity | none |
| matching rule | exact, then line-trimmed block, strict 1:1 lines | exact, then quote folding, then `\uXXXX` both ways | exact, then whitespace-flexible, then trailing-space regex, then 0.95 similarity | exact (overlap-detecting KMP), then line-level DP with 0.8 per-line similarity and line skipping |
| overlap handling | non-overlapping scan plus an overlap guard that throws | non-overlapping scan, no guard | non-overlapping scan, no guard; multi-call checks range overlap | overlapping scan, so overlap becomes ambiguity |
| identity check | EOL-normalized old equals new | raw `old_string === new_string` | `oldString === newString` then `NoChangeError` | no identity check; a no-op is caught by `edits did not change` nowhere, and `diff.is_empty()` renders `No edits were made.` |
| encoding | lossy UTF-8 fallback to latin1; BOM preserved | utf8 / utf16 / latin1 with BOM handling | uses the VS Code document, so the editor's decoding | the buffer's own decoding |
| line endings | normalize to LF, restore per line for mixed | normalize to the document's EOL | `oldString.replace(/\r?\n/g, eol)` against the document's EOL | the buffer's, with `strip_trailing_newline` handling |
| read-before-write gate | none; prompt text only | yes, an LRU state machine with hashes and timestamps | no gate on read; a confirmation prompt for sensitive paths | mtime comparison, plus a dirty-buffer prompt |
| re-indentation of the replacement | none, verbatim | none | none | yes, `Reindenter` with per-line deltas |
| output format | one sentence, plus a client-only diff | `The file X has been updated successfully.` plus notes | a `EditFileResult` with diagnostics and a streamed diff block | full `old_text` and `new_text` in the proto; `Edited X successfully` when rendered |
| failure text names the site | no | no | no | **yes**, the line numbers of every ambiguous match |

The two rows that most change a design are "matching rule" and "read-before-write gate", and the four products disagree on both. There is no consensus to copy, only four considered positions.

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
| Cursor's exact error strings | none | **missing** | `Anchor` carries `count`, which is enough to generate all three sentences, but nothing generates them today |
| line-ending detection and restoration | none | **missing** | no `stat`-level line-ending info in `crates/runvil-internal/vfs/src/stat.rs`; the Pack would have to detect from `read_string` output and restore by rewriting |
| mixed-ending per-line merge | none | **missing** | same; this is the `At` algorithm and it is a pure function, so it could live in the Pack |
| BOM detection and preservation | none | **missing** | `read_string` strips nothing, so a BOM arrives as U+FEFF in the string; the Pack would match against it, which is a behavior difference from Cursor |
| binary detection | none | **missing** | no NUL scan, no extension list. A binary file read as `read_string` fails as `NotUtf8`, which is arguably better than a separate binary path |
| file-too-large refusal | none | **missing** | no size check before read; `stat` could supply it |
| plan-mode gating | the handle's `gate(Op::Write, path)` | **covered** | the VFS already refuses a non-markdown write in Plan mode, so the Pack does not need `x_e` |
| ask-mode refusal | `gate(Op::Write, ...)` | **covered** | answers `PermissionDenied` or `ApprovalRequired`, which the Pack maps to a sentence |
| exclusivity against a concurrent writer | `admit(Claims::claim_write, ...)` | **partial** | the claim is per-access and epoch-ordered, which is stronger than Cursor's per-turn set, but it is still in-process |
| content compare-and-swap | none | **missing** | no hash parameter on `write`; a fresh implementation would need `write_if(path, expected_hash, contents)` or a `read`-then-`write` with a claim held across both |
| symlink policy | backend-defined | **partial** | `str_replace` goes through the backend, which may follow or refuse; the Pack cannot see which |

Two facts to keep in mind, both from the brief and both visible in the code.

`Access` never touches the disk directly. Every method follows the same four-step shape: `gate(Op::X, path)`, then `admit(Claims::claim_x, &path)`, then `fire(Op::X, &path)`, then `self.inner().x(&path, ...)`. So a new capability has a template to copy, and a Pack that does its own I/O would be breaking a rule the whole crate is built around.

`Access::str_replace` already refuses an empty anchor, before gating, with a comment explaining that the store view validates the path before the anchor. That means the existing VFS has already made the decision that an empty anchor is an argument error rather than a not-found, which is Cursor's decision too. Good agreement, and it means the Pack does not need to re-check emptiness if it delegates.

The gap that matters most is not in the table. It is that `str_replace` takes `&str, &str, &str` and returns `Result<(), VfsError>`, so it cannot report **which** region it replaced, **how many** matches there were on success, or whether the match needed relaxation. Every one of those is needed for Cursor's success text and for the telemetry counter. So even in the delegating design, the Pack has to run its own matcher to produce the message, which is the argument for Option A in decision 1.

## What remains unknowable

**Whether the UI renders the diff.** The agent emits an `editToolCall` with `diffString`, `linesAdded`, `linesRemoved`, `beforeFullFileContent`, and `afterFullFileContent`. The React component that paints it lives in the Cursor workbench, not in `cursor-agent-exec`, and could not be found in this bundle. Claims about what a user sees are claims about a layer outside the artifact.

**Whether a remote or cloud executor adds checks.** `StrReplace` talks to a `resourceAccessor`-provided read and write executor. Whether a remote executor adds a sandbox, an agent-store barrier, or a stricter path check is outside this artifact. The `F_e` agent-store conflict decorator is visible and its path regexes are visible, but the resource identities it consults are not resolvable from the bundle.

**Whether a text file can ever arrive as a blob.** `Sbe` accepts only `output.case === "content"`, and a `dataBlobId` or `contentBlobId` output would be rejected as `BinaryFileError`. The read shaper has a blob window and a `LIe` ceiling that serves double duty, so the plumbing exists. Whether it is reachable for a text file under the truncation cap could not be determined. If it is, a large text file is reported as binary, which would be a real bug.

**What `sparseLineNumbers` actually is at the edit tools.** The fuzzy hint renders through `fH`, which honours `sparseLineNumbers` by emitting only every Nth line number. The value is threaded from the agent config. In this build the config builder derives it from `promptVersion === "cursor-0226" ? 10 : undefined`, so the hint would show every tenth line number for that one prompt version. Whether an edit tool ever receives a non-zero value in practice depends on config paths outside the bundle, so the hint's exact rendering for `cursor-0226` is inferred rather than measured.

**Whether the vendor gate is the whole gate.** The `switch` in the tool list builder is visible and complete for the seven vendors it names, with an unreachable default. But whether another layer filters the list afterwards (a feature flag, a per-agent-type filter, a model capability check) could not be ruled out. The `isGemini3` check immediately after the switch shows the list is still being mutated, so treat the vendor switch as the primary gate rather than the only one.

**Why `multiFile` is a parameter.** `Bbe` takes `multiFile = false` and never reads it. One occurrence in the whole function. It is dead, and a reimplementation should not add it. Whether some other caller passes `true` (the notebook path, maybe) could not be determined, but since the parameter is unread it does not matter.

**Whether `MultiStrReplace` ever ships.** The id exists in two lookup tables and in the read-retry whitelist, and there is no registration. Either it is dead code from a removed feature or it is behind a flag that is off in this build. The shape of its parameters, and therefore whether it was an array-of-edits tool, is not recoverable from this artifact.

## Where this build differs from the parent spec

These are the disagreements between `agent-file-tools.md` and Cursor 3.27.12's shipped bundle. Each one is a finding rather than a nuisance, because a reimplementation built from the spec would reproduce the wrong behavior.

**1. The de-indent example is wrong, and the pseudocode is right.** The spec's worked-examples table says `old_string: "  foo"` with `new_string: "foo"` produces `Error: old_string and new_string are exactly the same`, and the prose repeats it as "so a de-indent attempt fails while a re-indent with differing forms succeeds". It does not fail. The identity check compares EOL-normalized forms and nothing else; `"  foo"` and `"foo"` are not equal, so the check passes, the fallback matches, and the edit de-indents the line. Verified against the shipped matcher: `runSingleEdit({contentsBeforeEdit: "foo\n", old_string: "  foo", new_string: "foo"})` returns `usedWhitespaceInsensitive: true` and `"foo\n"`. The spec's own pseudocode (`IF oldStr === newStr THROW`) is correct and contradicts its example.

**2. The spec says `StrReplace` is always registered. It is not.** The shipped build switches on model vendor: Anthropic, Google, xAI, zai, and unknown get `StrReplace` plus `Write`; OpenAI and Cursor vendors get `ApplyPatch` *instead of* `StrReplace` when `isGpt5Family` is true. Bundle offset 4582200 and daemon offset 13939958 both show the switch. A harness reproducing this build must not offer `StrReplace` to a GPT-5 family model.

**3. The spec's offset for the registration is stale, and so is its offset for the engine.** The spec gives `STR_REPLACE` at 4209511 and the edit engine at 4150373. The shipped build has the registration at **4224839** and the engine (`Bbe`) at **4165672**, with the read-then-write wrapper (`Sbe`) at 4158713, the write stage (`Rbe`) at 4163006, the error classes at 4149970 through 4150290, the result packer (`vbe`) at 4157903, the renderer (`ybe`) at 4157164, the planner gate (`W_e`) at 4150449, and the 512 KiB constant (`wbe`) at 4157892. Every offset in the spec's StrReplace section is about 15,000 bytes low.

**4. The spec says `formattingOptions` for the edit tools are hardcoded, so the hint always uses the default line-number format.** The research file states that `gpt5StyleLineNumbers` and `gpt5CodexCatN` are "hardcoded false for the edit tools" and that the snippet therefore "always uses the default `%6d|` format, regardless of model". In the shipped build the tools receive `e.formattingOptions` from the agent config (bundle 4583105), which is built by `buildFormattingOptions` (daemon 14044652) as `{ shouldUseFormatCodeblock: true, gpt5StyleLineNumbers: modelInfo.isGpt5Family, gpt5CodexCatN: false, ... }`. So on a GPT-5 family model the hint uses `L<n>:<text>` rather than the pipe format. This only matters on a model that never reaches this tool in the first place, because of finding 2, which is probably why the discrepancy was not noticed.

**5. `fH` has a second arity the spec does not describe.** The spec's description of the formatter covers the string form. The shipped `fH` (2529940) accepts either a string or an array of `{lineNumber, text}` objects, and in the array form an entry with a non-integer `lineNumber` renders as `"...".padStart(6, " ")` in the cat-n form, `"..."` in the GPT-5 form, and (in the default form) as a line that skips the number. That array form is used by Read for sparse numbering, not by `StrReplace`, but a reimplementation that shares one formatter needs to know both signatures exist.

**6. The spec's `K_e` client-visible string is not quoted anywhere.** The failure matrix shows `The model provided an ambiguous search string to replace` for the client half of the ambiguous error, and that is correct (verified at 4150055), but the spec's `### Failure modes` section only says "Client text" for the two engine-before-matching rows and leaves the ambiguous row's client text out. Minor, but it is a byte-exact string that a reimplementation will need if it renders a UI toast.

**7. The spec's claim that there is "no `MULTI_STR_REPLACE`" is right, but its evidence is incomplete.** The id appears in `K.Dm_.fields` as `MULTI_STR_REPLACE: "edit_tool_call"` (2742376), in the team map as `MULTI_STR_REPLACE: "agent-quality"` (3475096), and in the read-retry whitelist `iTe` (bundle 4338xxx, daemon 13939958 region), and there is no `AG("MULTI_STR_REPLACE", ...)` anywhere. So the id is referenced by three tables and registered by none. Worth recording because it is the closest thing in this build to the array-shaped tool that `pi_edit` and Copilot both ship.

**8. The spec says the tool's read has "no read-before-write gate" and the prompt text is at offset 2750288.** The string is real; the offset in the shipped build is **2764730** for `at least once before editing`, with `You MUST use the ` at 2764674. The behavior claim is correct.

**9. The spec describes five error classes but the shipped build has nine.** The spec names `H_e K_e V_e Y_e` (string-not-found, ambiguous, empty, identical). The shipped build also has `kbe` (file not found), `xbe` (invalid file), `Ebe` (read permission denied), `Ibe` (write permission denied), `Tbe` (write rejected), `Cbe` (binary), `Pbe` (file too large), and `Abe` (invalid edit args). Those are described in the spec's failure tables but not enumerated as classes, which matters if you are mapping a Rust error enum onto them one-to-one.

**10. The spec's `### No read-before-write gate` says the freshness read means "a stale edit fails with `File is too large to edit` or a genuine not-found rather than a stale-write error".** The first half is misleading: a stale edit fails with a not-found or an ambiguity error, and `File is too large to edit` has nothing to do with staleness. That sentence conflates two mechanisms.

**11. The spec gives the `dsv3-1018` description as ending with `To create or overwrite a file, you should prefer the write tool.`** Correct, and verified. But the spec's rendering of the default branch omits the `isComposer15` conditional line entirely, so a reader would not know that branch can carry six bullets instead of five. The line is `- The edit will FAIL if path isn’t given as the first argument. Always provide path first.` with a U+2019.

**12. Nothing in the spec covers `pi_edit` at all.** The whole parallel family is undocumented. This report's `### The \`pi_edit\` counterpart` is the first description of it, including the fuzzy-path whole-file rewrite bug.

## Conformance checklist

The twenty items from the parent spec's `### StrReplace` entry, each marked confirmed or corrected.

| # | Claim | Status | Reason |
|---|---|---|---|
| 1 | Parameters are exactly `path`, `old_string`, `new_string`, `replace_all`, and no `explanation`, `line_range`, `old_str`, `new_str`, or `target_file` exists | **confirmed** | two schemas, `oke` and `ike`; the second uses `file_path`; none of the five absent names appears anywhere in the tool's code |
| 2 | `replace_all` has no schema default on the default branch, arriving as `undefined` and normalized with a triple-equals test against `true` | **confirmed** | `.optional()` with no `.default()`; normalized by `const l = !0 === s.replace_all` |
| 3 | `replace_all` carries an actual `false` default only on the `dsv3-1018` branch | **confirmed** | `.default(!1)` on `ike` only |
| 4 | Both model strings and the file content have `\r\n` normalized to `\n` before matching | **confirmed, with a caveat** | model strings use `/\r\n?/g`, file content uses `/\r\n/g`; a lone `\r` survives in the file and is unmatchable across lines |
| 5 | An empty `old_string` throws before any search runs | **confirmed** | engine step 2, tested on the raw `old_string`, before `findMatches` |
| 6 | Normalized old equal to normalized new throws, so a de-indent attempt fails while a re-indent with differing forms succeeds | **corrected** | the throw is confirmed, but the example is wrong. The comparison is EOL-normalized only and does not trim, so `"  foo"` to `"foo"` is not an identity error and de-indents successfully. Verified against the shipped matcher |
| 7 | The exact scan advances by the needle length, not by one, so `aa` in `aaa` is exactly one match | **confirmed** | `i += r.length`; verified `count === 1` for `aaa`/`aa`, `count === 2` for `aaaa`/`aa` |
| 8 | The whitespace fallback runs only when the exact scan found nothing, and only when enabled on the call path | **confirmed** | `if (matches.length > 0) return` before the `if (!enabled) return`; `StrReplace` always passes `true` |
| 9 | The fallback trims per-line whitespace on both sides and does nothing else: no internal-run collapse, no Unicode normalization, no quote folding | **confirmed** | the matcher body contains only `trim()`, `split("\n")`, `endsWith`, `startsWith`, `lastIndexOf`, `indexOf`; verified that `"foo  bar"` does not match `"foo bar"` |
| 10 | The fallback requires a strict one-to-one line correspondence and offers no ellipsis elision and no block-anchor strategy | **confirmed** | loop bound `start <= fileLines.length - needle.length`, no inner consumption; verified that a two-line needle cannot span a three-line region |
| 11 | Zero matches report the relaxed-whitespace suffix only when the fallback actually ran | **confirmed** | the suffix is keyed on `usedWhitespaceInsensitive`, which is set true on the branch that calls the fallback, even when it returns zero matches |
| 12 | The zero-match throw carries the short fixed form to the client and the long form with the fuzzy hint to the model | **confirmed** | `H_e`'s constructor hardcodes `clientVisibleErrorMessage: "The string to replace was not found in the file."` and takes the model string as its second argument |
| 13 | The fuzzy suggestion is advisory only and never changes whether the edit applies | **confirmed** | the hint is built inside the throw expression after `occurrences === 0`; `buildDidYouMean` returns a value or `null` and has no effect on control flow |
| 14 | Multiple matches with `replace_all` false throw, and the message inserts ` after relaxing whitespace` before the period when the fallback found them | **confirmed** | template `... in the file${d ? " after relaxing whitespace" : ""}. Use replace_all ...` |
| 15 | The overlap guard fires only when `replace_all` is false, the match came from the exact scan, and the needle starts anywhere after the first match start plus one | **confirmed** | condition `!l && !d && -1 !== r.indexOf(c, u[0].startCharIndexZeroIndexed + 1)` |
| 16 | The splice recovers offsets into the original content, so untouched lines keep their exact original bytes | **confirmed** | `slice(0, a) + replacement + slice(b)` over the original string; no normalized copy is ever written back |
| 17 | The bulk splice runs only when the replacement is non-empty and there are two or more matches, walking offsets in descending order | **confirmed** | `if ("" !== r && n.length >= 2)` after `sort` descending, with a bail-out on inconsistent offsets |
| 18 | An empty `new_string` eats surrounding blank whitespace so a whole-line deletion removes its newline | **confirmed** | the three-branch clause in the sequential loop; verified five deletion vectors |
| 19 | `replace_all` on `aa` in `aaa` with `X` produces `Xa` and reports one replacement | **confirmed** | verified: `"Xa"`, `occurrencesModified: 1`, message `1 occurrence ... has been updated.` |
| 20 | Success text is one flat sentence with no diff, no line numbers, and no count unless `replace_all` was set | **confirmed** | the message builder is the only contributor; `linesAdded`, `linesRemoved`, and `diffString` are proto fields never read by the renderer |

Nineteen confirmed, one corrected. The one correction is not a small one: an implementation built from the spec's example would reject a legitimate de-indent and would need a workaround.

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
- `LocalPiEditExecutor` class: 9196200
- `bE` / `applyPiEdits` in the bundle: 9192653
- `wE` (pi fuzzy normalization) and `yE` (pi LF normalization): 9191800
- `EditSuccess` proto shape: 5970095
- `EditToolCall` and `EditArgs` proto shapes: 5972459, 5969181
- `EditToolCallDelta`: 5972839
- `ReadSuccess` proto shape: 6093936
- `PiEditToolSuccess` and `PiEditToolCall`: 6062332, 6061134

### Daemon, unminified twin

`C:\Users\Vinnie\AppData\Local\Programs\Cursor\resources\app\extensions\cursor-agent-host\dist\agent-host-daemon\dist\bin\daemon.cjs`, 20,213,626 bytes, original identifiers intact.

- `createStrReplaceTool`: 12625769
- `performEdit`: 12510955
- `performWrite`: 12514600 approximately
- `runSingleEdit`: 12524268
- `findStringMatches`: 12520700 approximately
- `findOccurrencesIgnoringWhitespace`: 12519100 approximately
- `replaceAllMatches`: 12521900 approximately
- `joinDisjoint`: 12521800 approximately
- `normalizeLineEndings`: 12522000 approximately
- `getClosestFuzzyMatchWithVSCodeDiff`: 12526414
- `formatStrReplaceResultForModel`: 12619463
- `createStrReplaceTool`'s registration tail with `createZodAgentTool("STR_REPLACE", ...)`: 12628930
- `PI_EDIT` tool definition `createPiEditTool`: 13899225
- `renderPiResult` and `extractResultText`: 13895487, 13895107
- `toPiEditToolResult`: 13906724
- `pi_edit` schema `editSchema`: 13911658
- `applyPiEdits`: 16836811
- `findMatches` (pi): 16836200 approximately
- `applyMatches` (pi): 16836600 approximately
- `fuzzyNormalize` (pi): 16834300 approximately
- `stripBom`, `detectLineEnding`, `normalizeToLf`, `restoreLineEnding`: 16834000 through 16834400
- `withPiFileMutationQueue`: 16843029
- `resolvePiPath`: 16843500 approximately
- `buildFormattingOptions`: 14044652
- `MAX_TEXT_SIZE = 8*1024*1024`: 18017
- vendor gate `switch(props.modelInfo.vendor)`: 13939958
- `PI_EDIT` in the per-tool identifier map: 10316166 and 13899225

### Scratch corpus

`C:\Users\Vinnie\cursor\cabinet\_scratch\cursor-tools\`

- `tools\STR_REPLACE.js` - the prettified slice; the prettier pass failed on a fragment boundary, so the file begins with a fragment of unrelated disposable code and the authoritative text is the minified region in the bundle
- `BRIEFING.md` - the identifier-mangling conventions

`C:\Users\Vinnie\cursor\cabinet\_scratch\claude-code\tools\`

- `edit.js` - the `Edit` registration, prompt, schema, validateInput, call, and `mapToolResultToToolResultBlockParam`
- `read-file-state.js` - the LRU state store
- `error-strings.json` - the 126 extracted strings, with offsets
- `_index.json` - extraction metadata: version 2.1.311, win32-x64, latin1 decode, 14 files, 0 anchor failures

### Probes

Run against the shipped bundle and daemon in `cabinet\_scratch\six-tool-reports\strreplace\`:

- `loc.js`, `find.js`, `dfind.js`, `rx.js`, `ag.js` - offset and string location
- `sl.js`, `sl2.js` - latin1 slicing
- `engine.js` - a verbatim reimplementation of `findStringMatches`, `findOccurrencesIgnoringWhitespace`, `replaceAllMatches`, `joinDisjoint`, `normalizeLineEndings`, and `runSingleEdit`, used for every behavioral vector in this report
- `vectors.js` - 34 named vectors covering the fallback, `replace_all`, deletion, CRLF, identity, empty input, and Unicode
- `dbg.js`, `dbg2.js`, `dbg3.js` - match-region dumping and the CRLF pipeline
- `pi.js`, `pi3.js`, `pi5.js` - reimplementation of `applyPiEdits`, including the fuzzy whole-file rewrite demonstration
- `bytes.js` - byte-level inspection of the em dash and the typographic apostrophe

The `engine.js` reimplementation is byte-for-byte the shipped logic with the mangled names restored; every claim in this report that says "verified" was produced by running it.

### Harness sources

`C:\Users\Vinnie\src\vscode\extensions\copilot\src\extension\tools\node\`

- `abstractReplaceStringTool.tsx` - the shared matcher dispatch, `_errorConflictingEdits`, `applyAllEdits`, `generateEdit`, the healing call
- `replaceStringTool.tsx` - `replace_string_in_file` and its description
- `multiReplaceStringTool.tsx` - `multi_replace_string_in_file`, the merge and sort logic
- `editFileHealing.tsx` - the repair pass
- `editFileToolUtils.tsx` - `findAndReplaceOne` and its four strategies, `applyEdit`, `assertPathIsSafe`, `makeUriConfirmationChecker`
- `insertEditTool.tsx` - read for the natural-language-instruction surface
- `applyPatchTool.tsx` - read for the delete-by-patch surface

`C:\Users\Vinnie\src\zed\crates\agent\src\tools\`

- `edit_file_tool.rs` - `EditFileToolInput`, `process_streaming_edits`, the streaming partial-input loop, the tests that pin overlap rejection and mid-edit failure
- `edit_session.rs` - `EditSession`, `EditPipeline`, `extract_match`, `ensure_buffer_saved`, `resolve_dirty_buffer`, the error strings
- `edit_session\streaming_fuzzy_matcher.rs` - the KMP exact scan, the line-level DP, the costs, the `0.8` similarity threshold, the `LINE_HINT_TOLERANCE`
- `edit_session\reindent.rs` - `Reindenter`, `compute_indent_delta`, `compute_rest_indent_delta`, and the tests pinning clamping and whitespace-only lines
- `tool_permissions.rs` - read for the gate

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
- `anomalyco/opencode` issues and PRs describing eight fuzzy replacers and a `WhitespaceNormalizedReplacer` that collapses internal runs, which Cursor does not.
- `aws/amazon-q-developer-cli` PR 3725 proposing a three-strategy chain with a 0.6 Levenshtein threshold, which does not exist in Cursor.
- Aider's `replace_closest_edit_distance` at 0.8 and its dead-code `return`, plus maintainer issue 306 on why Levenshtein matching was disabled.
- Claude Code issues 26996 and 29870 on tabs-to-spaces and U+00A0 failures, which Cursor's fallback happens to neutralize.
- `zhayujie/CowAgent` commit a94f4e3 on locating a fuzzy match in the original content rather than a normalized copy.

Where the web and the bundle disagree, the bundle wins and this report says so at the point of disagreement. The clearest case is the claim that Cursor uses a similarity threshold: no such constant exists anywhere in `Bbe`, and the only numeric literals in the match path are `0`, `1`, and the hint's context-line count of `2`.

*2026-10-27 - DeepSeek V4.1 Flash*
