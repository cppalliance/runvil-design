# A prompt directory served as MCP tools reaches three of the tool's eight fields

A Runvil prompt's YAML frontmatter already is the tool definition: a name, a description, and an `args:` declaration that derives a JSON Schema exactly. It holds none of the five fields that tell a client how a tool behaves. The gaps that stop a directory of prompts working as a server sit elsewhere, and four outrank the metadata question: nothing validates a name, nothing supplies a declared input file, nothing applies a declared default, nothing validates a call. PR 183 replaces the `args:` DSL with a hand-written `inputSchema`, leaves all four open, and adds one of its own by changing how the Engine reads arguments.

## Executive summary

The frontmatter is the right home for the contract and the `args:` DSL is the right way to fill the one part Runvil can generate. Three of the protocol's eight tool fields are reachable from the nine frontmatter keys: `name` to `Tool.name`, `description` to `Tool.description`, `args` to `Tool.inputSchema`. The other five, `title`, `outputSchema`, `annotations`, `icons`, and `_meta`, have no frontmatter key and no slot on `ToolInfo`, whose seven fields are `id`, `wire_name`, `description`, `parameters_schema`, `structured_output`, `survives_stop`, and `describes_tools` (`crates/runvil-internal/types/src/tools/info.rs:33`).

Annotations matter most, and they are where owning both parser and harness pays least. The protocol defaults `destructiveHint` and `openWorldHint` to `true`, so a silent prompt is advertised as possibly destructive and possibly open-world; silence is a claim, not neutrality. A parser supplying its own defaults looks like the fix, and mostly is not: of the four derivations this report examined, each lands on a protocol default or on a claim the code cannot back, because `uses:` is a requirement list rather than a bound on reach and the harness adds every `runvil` Pack the prompt does not shadow (`crates/harness-internal/runner/src/host.rs:145`). The value of an `annotations:` key is that an author can state what the parser cannot infer, not that the parser computes a better answer.

PR 183 should not be merged; this report should go to issue 179 instead. The pull request is right that JSON Schema is the target format and wrong about who writes it. It deletes the four-type declaration and its parse-time checks, adds a schema the parser can only shape-check, and inverts requiredness so forgetting one array yields an all-optional tool.

Names have no grammar and no uniqueness check, and `local/prompts/` already holds two files declaring `papergate`. A prompt declaring `input:` cannot be called as a tool, because its file arrives through `RunRequest.input_text` and a call brings only arguments. A declared `default` is never applied at run time. Nothing validates an argument, against a protocol rule requiring servers to validate all tool inputs.

## Method

This is a design review, not a test of a running system. No server exists to exercise, since per-prompt tools were removed on 2026-08-03 and the crates deleted on 2026-09-02 (`vibe/2026-09/2026-09-02-5-crate-taxonomy-rename.md:95`), so every execution claim comes from reading the code a server would call.

Three standards apply. The protocol standard is revision 2026-07-28, read from its generated schema rather than its prose, since the schema generates what a client validates against; I read earlier revisions' schemas where a field's shape changed. The implementation standard is rmcp 3.5.1, pinned at `Cargo.toml:117`. The repository standard is the parser, the Engine's argument derivation, the harness prepare path, and the Workshop contract DTO. Three things I could not do: measure context cost for a long description, run a client against a published list, and pin the revision that introduced `Tool._meta`, since the 2024-11-05 page summarizes the type in prose rather than showing its schema. That origin is marked unverified in Table 1.

## The protocol's tool has eight fields and only two are required

Table 1 lists every field of a `Tool` in 2026-07-28 with requiredness, type, and revision of origin, derived from the generated schemas for 2026-07-28, 2025-11-25, 2025-06-18, and 2024-11-05, and the tools page for 2025-03-26.

Table 1. Every field of a protocol `Tool`, with requiredness, type, and revision of origin.

| Field | Required | Type | Revision of origin |
|---|---|---|---|
| `name` | Yes | `string` | 2024-11-05 |
| `inputSchema` | Yes | `{ $schema?: string; type: "object"; [key: string]: unknown }` | 2024-11-05 |
| `description` | No | `string` | 2024-11-05; optional from 2025-06-18, earlier optionality unverified |
| `title` | No | `string` | 2025-06-18 |
| `outputSchema` | No | `{ $schema?: string; [key: string]: unknown }` | 2025-06-18 |
| `annotations` | No | `ToolAnnotations`, five optional fields | 2025-03-26 (PR #185) |
| `icons` | No | `Icon[]` | 2025-11-25 (SEP-973) |
| `_meta` | No | `MetaObject` | Present by 2025-06-18; introducing revision unverified |

One field a design document is likely to copy is absent. `execution.taskSupport` existed only in 2025-11-25, and 2026-07-28 removed it when tasks moved to the `io.modelcontextprotocol/tasks` extension, so setting it now writes a field the schema does not define.

Runvil can fill both required fields today. The reachability problem is a missing slot rather than a hard mapping: `ToolInfo` has no place for five fields, and only three of its seven concern MCP. The in-tree client keeps those three plus a title fallback, reading `description`, else `title`, else `name` (`crates/plugin-mcp/src/server.rs:52`), so this repository cannot read a title, annotations, or an output schema, and no in-tree test would catch a mistake in them.

## The mapping is exact where it exists and empty where it does not

`name` and `description` map with no transformation. The `args:` to `inputSchema` conversion is exact rather than approximate, because each of the DSL's four parts has a JSON Schema counterpart of the same meaning. The types are a subset: `string`, `boolean`, `integer`, and `number` are exactly the `ArgType` variants (`crates/runvil-internal/parser/src/contract/args.rs:21`), and each writes the same keyword because `Display` emits the lower-case name verbatim (`crates/runvil-internal/parser/src/contract/args.rs:36`). `optional: false` becomes a member of the root `required` array and `optional: true` omits the name. `default` becomes the property's `default`, already type-checked at parse (`crates/runvil-internal/parser/src/contract/args.rs:106`), which JSON Schema does not require. `description` becomes the property's `description`.

The one asymmetry is requiredness, and it is where PR 183 bites. The DSL requires a field unless it says otherwise, so its default is strict; JSON Schema makes a field optional unless the array names it, so its default is lenient. A generator writing `required` from the declaration keeps the strict default, while a human must remember `required: [topic]`, and forgetting produces no error at all: the tool accepts a call with no arguments. PR 183's fixture absorbs this by hand (`crates/workshop/server/src/routes/prompts-tests.rs`, `inputSchema:` hunk).

What the closed DSL gives up should be named: `array`, `object`, `null`, `enum`, `const`, `format`, `oneOf`, `anyOf`, numeric bounds, pattern, and length. The question is whether that buys more than losing a parse error for a misspelled `optinal`. A typo in the DSL fails the parse because `ArgDeclWire` is `deny_unknown_fields` (`crates/runvil-internal/parser/src/contract/args.rs:90`); a typo in an open schema is a valid document, because JSON Schema ignores keywords it does not know.
## Findings, ranked by severity

Seven findings follow, ordered by what they cost when they fire. The first three block the design as stated.

### Finding 1: Nothing validates a name, and the sample directory already has a duplicate

The parser reads `name` into a `String` and never inspects it (`crates/runvil-internal/parser/src/frontmatter.rs:47`). `parse.rs` validates the H1 count, fences, section names, and the `uses:` list, and not the name (`crates/runvil-internal/parser/src/parse.rs:100`). The crate's only grammar function is `is_valid_alias` at `contract.rs:38`, applied to arg names and model labels through `deserialize_contract_map` (`crates/runvil-internal/parser/src/contract.rs:101`), never to a prompt's own name. A prompt can be named with a space, at 300 characters, twice, or not at all.

The protocol's constraints are prose-only: `name` SHOULD be 1 to 128 characters, case-sensitive, ASCII letters, digits, underscore, hyphen, and dot only, unique within a server. The 2026-07-28 schema types it as a bare `string` with no `pattern`, `minLength`, or `maxLength`, so no client can enforce it. The in-tree client enforces nothing either: rmcp's name validation sits behind the `server` feature (`src/handler/server.rs:19`, gated at `src/handler.rs:3`), and this workspace enables only `client` and a transport (`crates/plugin-mcp/Cargo.toml:16`).

The files exist already. `local/prompts/papergate.md` and `local/prompts/drafts/papergate-copy.md` both declare `name: papergate` at line 2. The directory is gitignored (`.gitignore:6`), so the duplicate is not in the repository, but it is the directory this design targets, and it would yield two tools under one name with no error. The deleted server refused exactly this at boot with `^[a-z][a-z0-9_]{0,47}$` (`vibe/2026-08/2026-08-02-2-mcp-server.md:340`). The fix is a grammar and a uniqueness check at parse time, sized to the protocol's SHOULD set rather than the deleted rule: `[A-Za-z][A-Za-z0-9._-]{0,127}` keeps `papergate`, `research_person`, and `admin.tools.list` valid, where the old rule rejected the last two.
### Finding 2: A prompt declaring `input:` cannot be called as a tool

The input file arrives through a Host field, not an argument. `RunRequest` carries `input_text: Option<String>` beside `args: String` (`crates/harness-internal/runner/src/harness.rs:73`), and prepare passes it to `stage_input` before the run starts (`crates/harness-internal/runner/src/prepare.rs:296`). `stage_input` handles no declaration with no text, no declaration with text as an error, and a declaration with text by writing the file (`crates/harness-internal/runner/src/files.rs:90`). The fourth case decides the finding: a declaration with no text checks whether the store already holds the file and refuses with `InputFileError::Missing` when it does not (`crates/harness-internal/runner/src/files.rs:107`).

A prompt declaring `input:` therefore needs staged text or a pre-populated store, and `tools/call` carries `arguments` typed `{ [key: string]: unknown }` and nothing else. Publishing every parsed prompt publishes prompts that fail on every call, as `PrepareError::Input` rather than a tool error the model can correct. Three ways out exist and the design should pick one: exclude such prompts, contradicting the decision to publish every parsed prompt; publish with a file-content argument; or route an argument into `RunRequest.input_text` while keeping `input.path` as the target.

### Finding 3: A declared `default` is never applied at run time

`default` is parsed and type-checked (`crates/runvil-internal/parser/src/contract/args.rs:106`) and read by exactly two production sites: the Workshop DTO at `crates/workshop/server/src/routes/prompts.rs:142` and the UI prefill at `crates/workshop/ui/src/parts/run/run-rows.ts:126`. The Engine's `derive_argv` reads `is_default()` on the declaration, which asks whether the whole `args:` key was absent, not whether an arg has a default (`crates/runvil-internal/engine/src/execute/context-bound.rs:92`).

A prompt declaring `tone: { type: string, default: terse }` and called without `tone` therefore sees `argv.tone` as nil. That matches the module comment that the declaration "advertises, documents, and derives the tool schema; it does not enforce" (`crates/runvil-internal/parser/src/contract/args.rs:3`), and it does not match what a model reading the schema will infer, since omitting `tone` reasonably reads as yielding `terse`. In JSON Schema `default` is an annotation with no validation effect, so publishing it is technically correct and practically misleading. Apply the default in `derive_argv`, filling each missing declared arg before the value reaches Lua and keeping absent-versus-empty-string for args that declare no default.

### Finding 4: The protocol requires input validation and Runvil performs none

The 2026-07-28 tools page lists "Validate all tool inputs" among server MUSTs and routes input validation failures to `isError: true` so a model can self-correct, rather than to a protocol error. That reclassification landed in 2025-11-25 as SEP-1303; before it, 2024-11-05 and 2025-06-18 listed invalid arguments among protocol errors.

Runvil cannot meet the MUST and says so. `derive_argv` parses the string, returns nil on failure, and notes that "the executor never hard-errors on shape" (`crates/runvil-internal/engine/src/execute/context-bound.rs:88`). The design record repeats it for the deferred prompt-pack adapter: "The adapter does not validate args: the call's JSON passes straight to the sub-run" (`vibe/2026-09/2026-09-13-1-capabilities-global-naming.md:528`). This ranks fourth because the gap predates the change, and above the metadata question because publishing widens it: today a human types the string and a mistyped field yields a confusing nil, while a model reading a schema believes it constrains the call and receives no correction.

The fix is a validator in the tool-call path returning `isError: true` with the failing argument named. `plugin-std`'s `check` already validates `type`, `properties`, `required`, `enum`, and `items` against a definition file (`crates/plugin-std/src/check.rs:15`) and distinguishes absent from null in `required` (`crates/plugin-std/src/check.rs:29`); it is not general enough for an open schema and is ample for four types.
### Finding 5: The implicit prose wrap is a wire hazard a client cannot see

A prompt with no `args:` key gets one optional string field named `prose`, described as "Freeform input for this prompt" (`crates/runvil-internal/parser/src/contract/args.rs:254`). `derive_argv` wraps the whole args string as `{"prose": args}` for that case (`crates/runvil-internal/engine/src/execute/context-bound.rs:93`), and an explicit `args:` key never wraps, because `is_default` is true only when the key was absent (`crates/runvil-internal/parser/src/contract/args.rs:181`).

The wrap decides the input shape and the wire cannot say which shape applies. A call to a no-args prompt sending `{"prose": "..."}` reaches Lua as that object stringified inside `prose` unless the call path reconstructs the raw string first, so two prompts look identical in `tools/list` and behave differently with nothing in the published schema recording which is which. PR 183 sharpens it: the branch becomes `input_schema().is_none()`, so a prompt declaring `inputSchema: { type: object }` with no properties moves from wrapping to parsing because it now has a schema, which is a behavior change from a metadata edit. The fix is to publish the branch as data: emit the single `prose` property for the implicit declaration, and have the call path join a lone `prose` back into the raw string.

### Finding 6: A hand-written schema loses the close-world guarantee the DSL gives free

`Frontmatter` is `deny_unknown_fields` (`crates/runvil-internal/parser/src/frontmatter.rs:44`), `ArgDeclWire` is too (`crates/runvil-internal/parser/src/contract/args.rs:90`), and arg keys must match `[A-Za-z][A-Za-z0-9_-]{0,63}` (`crates/runvil-internal/parser/src/contract.rs:38`). A typo anywhere in an args block is a parse error naming the key (`crates/runvil-internal/parser/src/tests/frontmatter.rs:29`).

A JSON Schema has no such guarantee, in three ways. A misspelled keyword is ignored, so `propeties` constrains nothing and PR 183's `check` accepts it, since that function only verifies a present `properties` is a mapping, each property is an object or boolean, and `required` names declared properties. Requiredness inverts, so a forgotten `required` array silently makes every argument optional. And closure cannot be stated: omitting `additionalProperties: false` accepts extra arguments, while setting it rejects arguments a prompt's Lua might read. A closed DSL over four types fails loudly on every malformed instance, and an open schema fails silently on most.

### Finding 7: Nothing decides what a published prompt that fails to parse does

The deleted server chose one answer: a prompt failing validation becomes a broken entry carrying its error, keeps its place in the list, and returns that error when called, while every other prompt keeps serving (`vibe/2026-08/2026-08-02-2-mcp-server.md`, Reload section around line 334). Publishing nothing for it is defensible instead. The choice must be made before the server is written, because otherwise the first broken file makes it by accident.

## Deriving annotation defaults looks like the fix, and the one field worth changing is the one that cannot be derived safely

The parser should supply defaults, and the section's finding is what those defaults can honestly be. Derivation from `uses:` earns its place only as a lower bound labeled as one, and where being wrong costs data, the pessimistic protocol default must stand.

`ToolAnnotations` has five optional fields, unchanged since 2025-03-26: `title`, `readOnlyHint` defaulting `false`, `destructiveHint` defaulting `true`, `idempotentHint` defaulting `false`, `openWorldHint` defaulting `true`. The specification calls them hints, says they "are not guaranteed to provide a faithful description of tool behavior", and says clients MUST treat them as untrusted unless the server is trusted. An annotation is advisory, not a security boundary.

The defaults describe a shape of silence: a bare tool reads as not read-only, possibly destructive, not idempotent, and open-world. For `std/rm` that is accurate. For a prompt that reads a paper and writes a report, `readOnlyHint: false` and `idempotentHint: false` are right, and `openWorldHint: true` is right in fact because the harness adds Packs the declaration does not name. Only `destructiveHint: true` is too pessimistic, since that prompt replaces nothing.
That single field sets the ceiling on what a derivation buys, and the ceiling is that nothing can be relaxed. The field an author would want relaxed is the one a parser cannot safely relax, because the std Pack can replace a file and a `uses: [std]` declaration does not say which of its six tools the prompt calls. A parser overriding it would trade a small annoyance for a data risk. Derivation is still worth doing, because the parser sees the whole declaration and can state what the protocol leaves implicit, and an `annotations:` key earns its place because an author can state what the parser cannot infer. Using either is not a security claim.

### Derive `readOnlyHint` and `openWorldHint`, and leave the other two alone

`readOnlyHint` has the cleanest signal, because the Engine's model of a run's product is its output file: `RunReport` carries `output` and prepare resolves `output_path` from the declaration (`crates/harness-internal/runner/src/prepare.rs:303`). A prompt declaring no output and requiring no writing Pack looks read-only, but that inference breaks in two places. A run's VFS holds the declared store plus any mounted real directories (`crates/harness-internal/runner/src/harness.rs:76`), so a prompt with no `output:` can write anywhere its grants allow. And `readOnlyHint: true` is stronger than `destructiveHint: false`, since it says nothing is modified at all. So the derivation runs only in the safe direction: emit `false` when the prompt declares `output:` or requires a Pack that can write, and omit the field otherwise. The output declaration can prove a prompt is not read-only, and cannot prove it is.

`openWorldHint` has the better signal, because a Pack's identity says where its tools reach. The web Pack fetches URLs and runs searches (`crates/plugin-web/src/lib.rs:6`); the std Pack holds six file tools bounded by the store and granted directories (`crates/plugin-std/src/lib.rs:9`). A prompt declaring `uses: [web]` reaches outside the machine. That is a lower bound rather than a fact, because `uses:` is a requirement list rather than a capability list and the harness adds more than the list names: `HostContext::pick` walks every built `runvil` Pack whose id no entry replaces (`crates/harness-internal/runner/src/host.rs:145`). Lua can also call any catalog tool by id, since `tools.call` resolves an id rather than checking it against the declaration (`crates/runvil-internal/lua/src/tools/decode.rs:30`), and `tools.packs.extras()` enumerates the usable undeclared Packs (`crates/runvil-internal/lua/src/packs.rs:185`). Deriving `false` is therefore a claim the code does not support, while deriving `true` from a web Pack is one it does. The rule is the asymmetry: derive `true` when a known open Pack is named, keep `true` when silent, and never derive `false`.

The other two need no derivation. `destructiveHint` must stay `true` unless the prompt says otherwise, because a silent prompt has promised nothing and the std Pack ships `std/write` and `std/replace` (`crates/plugin-std/src/lib.rs:12`). The failure a derivation would introduce is a prompt declaring `uses: [std]` for `std/read` only, receiving `destructiveHint: false` from a rule treating std as safe, then calling `std/replace` from Lua. And `idempotentHint` must stay `false`, because a run is a sequence of model calls seeded from `RunSeed::from_bytes(rand::random())` (`crates/harness-internal/runner/src/prepare.rs:253`), so two runs with identical arguments may produce different output. Only the prompt can claim determinism.

### Let the prompt declare, and derive only what silence leaves

The recommended shape is a frontmatter key mapping onto `ToolAnnotations` field for field, with the parser filling defaults for a silent prompt. A prompt that reads a paper and writes a report can defensibly claim to be non-destructive and closed-world, and it says so:

```yaml
---
name: papergate
description: Report on the evidence a WG21 paper provides for its need of standardization
runvil: 0
uses: [std]
input: { path: paper.md, description: The paper to assess }
output: { path: report.md, description: The assessment }
annotations:
  readOnlyHint: false
  destructiveHint: false
  idempotentHint: false
  openWorldHint: false
---
```

The same prompt with no `annotations:` key gets `readOnlyHint: false` because it declares an output, `destructiveHint: true` because it did not say otherwise, `idempotentHint: false` because a model call is not reproducible, and `openWorldHint: true` because the declaration names std rather than an open Pack. Those are the protocol defaults, so the derivation changed nothing for this prompt, which is what the four impossibilities above predict.

The value of the key is that the declaration in the block is stronger than any derivation: `destructiveHint: false` and `openWorldHint: false` are claims a parser cannot make and an author can. That is the recommendation, and it is the opposite of deriving more. The parser supplies the protocol defaults, refuses to relax the two that cost data, and gets out of the way when an author knows better.

A declaration is also not a bound on behavior. A `uses:` list is a lower bound, because the harness adds undeclared `runvil` Packs and Lua can call any catalog tool by id (`crates/harness-internal/runner/src/host.rs:145`). A prompt's Lua can define tools at run time through `tools.adhoc`, which appear in no frontmatter (`crates/runvil-internal/lua/src/tools.rs:143`). And a Pack's behavior depends on its Host configuration, not only on the prompt. An `annotations:` block is a statement of intent the Engine cannot enforce, which is exactly what the protocol says a hint is.

### A prompt that calls another inherits its callee's annotations as a floor

The composition direction is not the same for each field, which is why "union" is the right instinct and the wrong word. For `readOnlyHint` the composition is AND: a parent is read-only only if everything it reaches is, so a single writing callee makes the parent not read-only. For `destructiveHint` and `openWorldHint` it is OR, and those two are what the intended union describes, since one destructive callee makes the parent destructive and one network callee makes it open-world. For `idempotentHint` there is no operator at all. A parent that calls A then B is not idempotent even when A and B each are, because the second call repeats both and any accumulation in the store happens twice. `title` does not compose, because a parent's display name is its own.

What the parent publishes is therefore a floor on how bad the call can be, not a summary of its callees. That framing is what makes propagation correct rather than merely convenient. A prompt that calls something which writes is not read-only whatever its own Lua does, and a prompt that calls something which reaches the network is open-world. The published annotation describes the whole subtree because one tool call runs the whole subtree.

The obstacle is completeness, and it has a clean answer rather than a fuzzy one. `execute()` takes a path, and that argument is a Lua value the parser cannot evaluate, so a prompt can name a document the parser never saw; the deferred contract resolves and freezes that source at call time (`vibe/2026-09/2026-09-10-1-unified-prompt-model.md:119` and `:194`). So propagate over declared callees, and let a prompt that calls dynamically inherit the pessimistic default. The join is sound over declared callees because a static `calls:` list bounds one direction only, so declaring becomes self-interested rather than enforced: a prompt that names its callees inherits their floor, and one that does not inherits the worst case.

The leaf case is the one complete case, and it is where derivation is a fact rather than an inference. A prompt with no declared callees and no writing Pack is closed and read-only by construction. Every other derivation remains a guess dressed as a fact, which is this section's existing conclusion, and propagation does not weaken it: a parent's floor is only as trustworthy as its callees' declarations.

The section's finding that a `uses:` list is a requirement list rather than a bound on reach applies to a callee exactly as to the parent, and the Engine already treats a nested call as a bounded but real escalation: the design record guards prompt-tool nesting at `Environment.max_depth`, default 3 and deliberately low, accruing only on model-orchestrated hops (`vibe/2026-09/2026-09-13-1-capabilities-global-naming.md:599`).

## The description has no limit on either side, and the real cost is caller context

Nothing in the protocol constrains a description's length. The 2026-07-28 schema types it as an optional `string` with no bounds, and the tools page calls it "a hint to the model" without mentioning length, verbosity, or context cost. The specification is silent, so there is no rule to satisfy and none to break. Nothing in the parser constrains it either: `description` is a `String` on `Frontmatter` (`crates/runvil-internal/parser/src/frontmatter.rs:49`), and no path truncates or measures it.

The cost falls on whoever lists the tools. The protocol's non-normative client best-practices guidance warns that loading every tool definition into context "wastes tokens, increases latency, and degrades model performance", and recommends progressive discovery once definitions take a threshold of the window, citing 1 to 5 percent as an example. That is guidance for clients and should be quoted as such. The server-side consequence is that a directory with long descriptions makes a large `tools/list`: nothing for a few dozen one-line descriptions, and the failure the guidance describes for hundreds. No limit is enforced, so the convention should be one sentence with detail in the H1 or the args descriptions.

## A long description folds, and the feared column comes from nesting rather than style

All three YAML scalar styles are available. A plain scalar folds, joining `description:` with an indented continuation and a space, which I confirmed by parsing all three styles: a two-line plain scalar returned as one line. A literal block (`|`) preserves newlines and keeps the trailing one. A folded block (`>`) folds single newlines to spaces and turns a blank line into a paragraph break, returning `para one continues\npara two\n` from two paragraphs. The parser test at `crates/runvil-internal/parser/src/tests/frontmatter.rs:22` confirms the literal case, since only a block that keeps its newlines makes an indented `---` inside one meaningful.

A 500-character description therefore need not be a 500-character line. Wrap it, and a plain scalar or `>` yields one line, or one line per paragraph, while the wire sees one long string either way.

The feared column comes from position rather than style. Inside an `args:` block a description sits four levels deep (key, arg name, arg's fields, description), which at two spaces per level is eight columns before the text begins, and a long line there becomes an indented ribbon that a block scalar cannot fix, since its content is indented relative to the key. This argues for the closed DSL: it caps an author at four levels, while a hand-written schema can nest `properties` inside `items` inside `properties` and push a description arbitrarily far right.

One more fact matters. `split_frontmatter` requires the opening `---` at column zero (`crates/runvil-internal/parser/src/frontmatter.rs:226`) and treats any line trimming to `---` as the closing delimiter (`crates/runvil-internal/parser/src/frontmatter.rs:241`), so a description containing such a line closes the frontmatter early unless it sits in a block scalar.

## A prompt cannot read its own metadata, and `sys.frontmatter` would cost a version bump

The answer is no. `sys` is a sealed JSON table holding `when`, `id`, `taskid`, `section_name`, `execution`, `section_count`, then `index` when a chain has one and `model` once the binding is fixed (`crates/runvil-internal/engine/src/execute/section_context.rs:53`). Its seal raises on an unknown field name, so an unheld field is an error rather than nil (`crates/runvil-internal/lua/src/sys.rs:33`). The `prompt` global holds exactly two functions, `section` and `list`, both reading Markdown text (`crates/runvil-internal/lua/src/vm/install.rs:261`). Neither exposes frontmatter, so a prompt cannot ask its own name, its output declaration, or its Packs.

Adding it would be consistent, since `to_json` already builds `sys` from one struct (`crates/runvil-internal/engine/src/execute/section_context.rs:77`). The cost is the one the owner named: a key in `sys` is read, and its shape is frozen for as long as a prompt reads it, whereas a frontmatter key nothing reads is inert. A prompt that wants its own declaration can declare it instead: `var.input_path = "paper.md"` costs a line and no contract. Introspection earns its place only for something unknowable at authoring time, such as which args the caller supplied, and that is already in `argv`.

## Each alternative loses on a specific ground

### MCP `prompts` loses the typed declaration and control of execution

`PromptArgument` is `name`, optional `title`, optional `description`, and optional `required`, with no type at all, and `prompts/get` arguments are typed `{ [key: string]: string }` where a tool's are `{ [key: string]: unknown }`. The four-type declaration has no representation; everything would arrive as a string and the prompt's Lua would parse `"true"` and `"3"`.

The larger loss is execution. `prompts/get` returns `messages` with roles limited to user and assistant, which the client's model then runs: the prompt's Lua never executes, the Engine never walks sections, no tool call goes through Runvil, and the store is never involved. A Runvil prompt is an executable program, and served as a template it is a block of text asking the client's model to do what the Lua would have.

### The Skills extension fits the shape and not the execution model

The Skills extension (`io.modelcontextprotocol/skills`, SEP-2640, Final) is built for this shape: a directory whose `SKILL.md` carries YAML frontmatter with `name` and `description`, served through `resources/read`, with `skills/list` returning every frontmatter field unchanged and a manifest of files with SHA-256 digests and sizes. It runs nothing. Reading `SKILL.md` "does not itself activate a skill"; the content becomes instructions in the model's context, so adopting it would make a Runvil prompt advice rather than a run.

### PR 183's raw `inputSchema` loses on who owns the schema

The proposal replaces `args:` with a hand-written schema the parser converts from YAML to JSON, checks for `type: "object"`, checks that `properties` is a map of schemas, and checks that `required` names declared properties. It deletes `contract/args.rs` (278 lines), adds `contract/input_schema.rs` (174), and rewrites `derive_argv` to branch on `input_schema().is_none()`. Its argument is real: JSON Schema is the wire format, so publishing an author-written schema is one conversion fewer, and an author who needs an array or an enum gets it.

What it costs is the four points established above: inverted requiredness and lost closure (Finding 6), a moved wrap boundary (Finding 5), a human-maintained schema that can drift from an Engine that validates nothing (Finding 4), and a diff unreviewable for behavior, since 20 files built around replacing a contract module mean a reviewer cannot confirm the old behavior survives without reconstructing the mapping by hand.

## The reversal changes what the tool list is for, and the August argument still half holds

This reverses a decision from 2026-08-03 and should be stated as a reversal. On 2026-08-02 a server was designed and built publishing per-prompt tools, with an `expose` configuration, a promotion workflow, a discovery tool, and a boot pass enforcing the name grammar and refusing duplicates (`vibe/2026-08/2026-08-02-2-mcp-server.md:340`). On 2026-08-03 a correction deleted exposure on a principle: "A Runvil prompt is a command. It is invoked because someone named it... It is never something a model reaches for because it noticed a tool that looked relevant" (`vibe/2026-08/2026-08-03-1-mcp-server-correction.md:69`). Decision 3 accepted the cost: "a prompt can never be called under its own name as a tool, and per-prompt typed argument schemas are permanently off the table" (`vibe/2026-08/2026-08-03-1-mcp-server-correction.md:87`). On 2026-09-02 the crates were deleted as unused (`vibe/2026-09/2026-09-02-5-crate-taxonomy-rename.md:95`, step 1 at line 131).

Two things changed. The August decision governed a server executing prompts a caller names, where a caller who names a prompt needs no typed schema, while this proposal publishes a directory where the tool list is the menu and typed schemas are the point. And the protocol caught up with the mitigation, since the client best-practices guidance now documents progressive discovery. That cuts both ways, since it justifies the worry, because the failure is real enough to document, and supplies the mitigation, because a client following it never sees the whole list at once.

What did not change is the part that still bites. A prompt is still a command, publishing it still lets a model reach for it because a description looked relevant, and none of the available mitigations is a prohibition. The design should accept that cost rather than claim the August concern was mistaken, because the concern was right about the mechanism and wrong only about whether it disqualified the design.

## Recommendations

Do not merge PR 183; post this report to issue 179. Owner: the project owner. Next step: comment on the issue linking the report, and note on the pull request that the design question is answered there. Confidence high, because the pull request patches a rejected design and the request was for exposition.

Keep `args:` and add a schema generator emitting `inputSchema`. Owner: the parser. Next step: a function from `ArgsDecl` to a `serde_json::Value` with `type: "object"`, `properties`, and a `required` array built from args whose `optional` is false, tested by golden JSON. Confidence high, because every mapping is one-to-one.

Add a name grammar and a uniqueness check at parse time. Owner: the parser for the grammar, the directory resolver for duplicates. Next step: `is_valid_tool_name` beside `is_valid_alias` accepting `[A-Za-z][A-Za-z0-9._-]{0,127}`, plus a duplicate check. Confidence high, because the duplicate already exists and the check is a few lines.

Decide how a prompt declaring `input:` is called. Owner: the project owner for the decision, the server for the code. Next step: a written choice among excluding such prompts, adding a file-content argument, or routing an argument into `RunRequest.input_text`. Confidence medium, because all three work and the choice depends on directory usage this review cannot see.

Apply declared defaults in `derive_argv`, and add run-time validation against the generated schema returning `isError: true` naming the failing argument. Owner: the Engine for the first, the server for the second. Next step: a fill pass over missing keys with declared defaults, and a validator over the four DSL types plus `required`. Confidence high on both, because each is local and the closed DSL keeps them small.

Add an `annotations:` key and a slot for it on `ToolInfo`. Owner: the parser for the key, `ToolInfo` for the slot. Next step: the key plus the derivation rules, with a declared value overriding any derived one. Confidence medium, because the derivation is sound but the gain is mostly honesty and unmeasured friction.

Publish the prose wrap as data, and keep tool descriptions to one sentence. Owner: the schema generator for the first, the prompt authors for the second. Next step: a single `prose` property for the implicit declaration with an unwrap on the call path, and a line in the frontmatter documentation. Confidence high, because both are cheap and the alternatives are a wire hazard and a tool list of accidental length.

## Limitations, and what would change the conclusion

Three assumptions carry the conclusion. The first is that the protocol stays at 2026-07-28 long enough to ship; if the annotation defaults change, the annotations section loses its premise. The defaults have held since 2025-03-26, which is evidence and not a guarantee.

The second is that this publishes to clients beyond this repository. If the only consumer is the in-tree client, much of the field analysis is moot, because that client reads three fields and drops the rest (`crates/plugin-mcp/src/server.rs:52`).

The third is that no server code exists to test against, so every execution claim is a reading. A server staging input text or applying defaults above `derive_argv` would reduce three findings to documentation defects, and that would be the better design. Separately, the sample prompts are not representative: the tracked `prompts/` directory declares `runvil: 0` in all five files, which passes the gate (`crates/runvil-internal/engine/src/execute/run.rs:309`), while the gitignored `local/prompts/` directory declares `runvil: 1` in all four, which the same gate refuses at line 310. The prompts actually run are in a directory the Engine would reject if the check were reached, so the directory this design targets has never run against the code it targets.

What would change the conclusion: a revision making the annotation defaults optimistic or defining a machine-checkable name constraint; a decision that the only client is in-tree; or a server that owns validation and input staging above the Engine.

## Appendix

Table 2 gives every frontmatter key with its type, requiredness, and whether it reaches a protocol tool field.

Table 2. Every frontmatter key in `Frontmatter`, with type, requiredness, and protocol reachability.

| Key | Type | Required | Reaches a protocol tool field |
|---|---|---|---|
| `name` | `String` | Yes | `Tool.name` |
| `description` | `String` | Yes | `Tool.description` |
| `runvil` | `Option<u32>` | No | No; inclusion predicate and version gate |
| `max_tool_iterations` | `MaxToolIterations`, default or 1 to 1000 | No | No; a run limit |
| `input` | `Option<FileDecl>` of `path` and `description` | No | No slot; its comment claims it feeds schema generation and nothing consumes it |
| `output` | `Option<FileDecl>` | No | No slot; candidate source for a derived `readOnlyHint` |
| `uses` | `Vec<PackRef>` | No, empty | No slot; candidate source for a derived `openWorldHint` |
| `args` | `ArgsDecl`, default one optional `prose` | No | `Tool.inputSchema` |
| `models` | `ModelRoles` | No, empty | No slot |

Table 3 gives the five annotation fields with the protocol default and the default this report recommends.

Table 3. Every `ToolAnnotations` field, with the protocol default and the recommended default.

| Field | Protocol default | Recommended default | Why |
|---|---|---|---|
| `title` | Absent; display falls back to `annotations.title`, then `name` | Absent | No title key exists and `name` already displays; a second name invites confusion |
| `readOnlyHint` | `false` | `false`; stated explicitly when the prompt declares `output:` or a writing Pack, omitted otherwise | Omission leaves the conservative protocol default standing |
| `destructiveHint` | `true` | `true`, never derived to `false` from silence | `std/write` and `std/replace` exist, and a false claim here can cost data |
| `idempotentHint` | `false` | `false`, declarable to `true` by a prompt that means it | A run is randomly seeded and a model call is not reproducible |
| `openWorldHint` | `true` | `true`; stated explicitly when the declaration names an open Pack | `uses:` is a lower bound, so `false` is a claim the code cannot support |

*2026-10-29 11:13 - DeepSeek V4.1 Flash 1017*
