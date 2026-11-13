# Tools and Packs: design decisions and their reasons

- Scope: tools, Packs, Plugins, the Plugin C contract, and the Packs that ship with Runvil
- Code: `runvil` at commit `fa9936f64`, `runvil-docs` at commit `cfd69a4`, and `runvil-plugin-api` at commit `63a2403`
- Update: the text on library Plugins, the C contract, and the `std` Pack describes `runvil` at commit `b0693699d`, `runvil-plugin-api` at `03a9fa3`, and `runvil-plugin-std` at `782ad23`, and the Paths bullet names the repositories as of `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 13)
- Update: the text on the two Plugin APIs, the `bash` Pack, the planned work, and the gaps describes `runvil-harness` at commit `18361d02`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, `runvil-plugin-bashkit` at `9c8e980`, and `runvil-workshop` at `f386801b` (section 13)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil-workshop`

## Summary

A Plugin installs Packs, a Pack installs tools, and a section's Lua offers tools to its model. The Engine asks for each tool call as an effect, and the Harness runs it. Sections 1 to 9 describe that design and its reasons.

A check of this page against the code found five gaps, one of them in the language guide, and left sixteen questions about reasons for the owner. Section 11 lists the gaps, and section 12 lists the questions. The two largest gaps are in the code. Two trust rules overlap, and together they send MCP server text to a model without the untrusted envelope. Two APIs install Packs, the C contract and a Rust API, and each has abilities that the other lacks.

Section 10 lists the planned work. Its two largest items are one Plugin API, which moves `plugin-web` and `plugin-mcp` onto the C contract, and file changes through the C contract, which the `std` tools Write, StrReplace, and Delete and the `bash` shell wait for.

## From a Plugin to a tool call

A Plugin is an entry point that installs Packs. Four Plugins ship with Runvil. Two are dynamic libraries that export the C function `runvil_plugin`: `std` and `bashkit`, each in a repository of its own. The other two are Rust functions named `install`, one in each of `plugin-web` and `plugin-mcp`. A Pack installs tools, and a Rust Pack can also set a Lua prelude. The Pack `runvil/web` installs two tools, `web/search` and `web/fetch`.

At startup the Host creates a `PackRegistry`, calls each `install` function with it, and has the Harness load the libraries of its Plugin directory into it. Each Pack enters the registry together with a factory. When a run starts, the Harness picks the run's Packs, asks the factory of each one for that run's Pack instance, waits for the instance's `ready`, and reads its tool list. The two ways into the registry are two Plugin APIs with different abilities, which is a gap (section 11). The plan is one API, the C contract, for every Pack (section 10).

A prompt lists the Packs it requires under `uses:`. Each section's Lua then offers tools to its model with `tools.offer`. At the section's first `models.loop` round, the Engine builds the tool list from those offers and sends the same list with every later round.

When the model calls a tool, the Engine emits a `ToolCall` effect and waits. The Harness finds the Pack from the first segment of the tool id and calls `Pack::call`. The Pack returns text marked trusted or untrusted. The Engine wraps untrusted text in an envelope and appends the result to the section's messages.

## 1. Plugins, Packs, and names

### "Pack" names the set of tools, and "Plugin" names the entry point that installs it

A Pack installs tools, and a Rust Pack can also set a Lua prelude. A Plugin is an entry point that installs Packs. "Plugin" is the usual word for what a loadable crate or library provides, and the names of the Plugin crates, such as `plugin-std` and `plugin-web`, fit it. A Plugin that follows the C contract reports a name and a version, which the Host lists. A Rust `install` function reports neither.

### A Pack id is the second segment of the Pack's name

A Pack's fully qualified name has two segments, a vendor and a name, as in `runvil/web`. The name segment is the Pack id, and every tool id in the Pack starts with it, as in `web/fetch`.

### Lua uses a tool's id, and the model uses its wire name

Lua code uses the id, such as `web/search`. The model sees the wire name, which by default is the id with `_` in place of each `/` and `.`. A Pack may set a different wire name, and the model sees `std/read` as `Read`.

The tool objects that Lua reads omit wire names, because the Engine may rename them, and prompt prose that names them would then be stale.

### A bare name under `uses:` means the `runvil` vendor

`uses: [web]` means `runvil/web`. A prompt writes another vendor's Pack with both segments, as `acme/web`, and that Pack then replaces `runvil/web` in the run. Under this rule each name has one meaning. If a bare name could mean a third-party Pack, a `runvil` Pack of the same name that ships later would silently take its place in an existing prompt.

The Engine reserves two Pack ids for its own tools. A tool that a prompt defines in Lua gets an id under `adhoc`, and the two discovery tools get ids under `dynamic`.

### A Pack is for a feature that works on something outside the run

The Engine provides `store`, `var`, `log`, and `models` in every section. A Pack provides a feature that works on something outside the run, such as the web or an MCP server.

The test for a new feature is whether it works on something outside the run. The store is part of the language because it is how a prompt moves data between sections and handles fanout. A Pack for budget, compaction, request shaping, or model routing would duplicate work that the Engine, the prompt's Lua, the Gateway, or the Harness owns, in that order.

## 2. Requiring Packs with `uses:`

### A `uses:` entry requires a Pack and runs its prelude

A prompt names Packs under `uses:` in its front matter. For each entry, `Host::parse` checks that the Host has the Pack and refuses the prompt when the Pack is missing. The Engine then runs the Pack's Lua prelude in every section of the prompt. A Rust Pack sets its prelude in its `PackInfo`. Contract 0.4 gives a Pack a name and a description, so a prelude for a library Pack is still to be built, and each shipped Pack leaves its prelude unset.

A prelude runs only for a declared Pack because it defines Lua globals, and so changes what a prompt's Lua can call. The Lua globals of a prompt therefore depend on its own `uses:` list and are the same on every Host.

Every entry is required. A prompt that can run with or without a Pack tests `tools.packs.get('name')`, which returns the Pack object when the run has the Pack.

### A run includes every `runvil` Pack, and a section opts in to the undeclared ones

A run includes the Packs its prompt declares, plus each `runvil` Pack whose id is still free. The guide calls the undeclared ones extras. A section offers them with `tools.offer(tools.packs.extras())`, as the built-in chat prompt does. The owner's rule: "extras should be a per-section setting. Disabled by default, and opt-in".

### A declared Pack can be refused at three points

1. At install. A Rust Plugin whose setup fails inserts its Pack as unavailable, with a reason. The loader drops every Pack of a library Plugin whose load fails, and `Host::parse` then reports each one with the text `is not installed`.
2. At parse. `Host::parse` refuses a prompt that declares a Pack that the Host lacks, or one that a Plugin inserted as unavailable.
3. At the run's first step. A factory that fails or panics, or a `ready` that returns an error, makes the Pack unavailable to that one run. The Harness also marks `runvil/user` unavailable in a run that lacks a user (section 9).

The third case exists because a factory's result is unknown at parse time. The next run calls the factory again.

### The Harness reads each Pack's tools once, when the run starts

The Harness picks the run's Packs from `uses:`, makes their instances, waits for each `ready`, and reads each tool list. The run uses that catalog until it ends. The Harness picks first so that a run waits only for its own Packs. An error from `ready` is temporary. A Pack that recovers is available to the runs that start afterward.

### Lua reads one catalog for the whole run, and the Engine builds the model's tool list once for each section entry

The run's catalog lists every Pack tool's `ToolInfo` (its id, wire name, description, parameter schema, and flags) by tool id. `tools.get`, `tools.call`, and `tools.offer` look ids up in it. The Harness builds it before any Lua runs, so each lookup returns the same answer for the whole run. For each section entry, which is one execution of a section, the Engine builds what a model sees: the wire names, the Static Set, and the Dynamic Set (section 3).

The Harness fixes the catalog in advance so that the Engine is deterministic and leaves all I/O to its caller. One catalog covers the whole run because the catalog is the same for every model.

## 3. Offering tools to a model

### A section's Lua offers tools to its model

`tools.offer("web")` offers both tools of the `web` Pack. `tools.offer("web/fetch")` offers one tool. The model's tool list for the section contains the offered tools.

Each entry into a section starts a fresh Lua VM with an empty offer list. A prompt that wants the same offers in several sections puts them in one section and loads that section with `require`.

Offers are per section for isolation: a section's model sees only the tools that the section offered. The rule also stops a prompt with 20 tools from sending 20 schemas to the model of every section.

### `tools.offer` takes tools and Packs, as ids or as objects

Its single argument is a tool, a Pack, or an array of both, and each item is an id string or an object. A string that contains a slash is a tool id, and any other string is a Pack id. The owner's rule: "I think there should only be one api to offer tools".

### The first `models.loop` round builds the tool list, and later rounds reuse it

Each `tools.offer` call checks its argument and records the offer. The section's first `models.loop` round builds the tool list, and every later round of that section sends the same bytes. A call to `tools.offer` after that round raises an error that says "the section's tool list is fixed at its first models.loop round".

The reason is the provider's prompt cache. A tool list rebuilt every round can change the request prefix in the middle of a conversation, and the provider then discards its cache.

### `tools.offer_dynamic` builds a second set, which the model uses through two discovery tools

The Static Set is the list of tools offered with `tools.offer`, and every round sends it. The Dynamic Set is the list of tools offered with `tools.offer_dynamic`. When the Dynamic Set has members, the Engine appends two tools of its own to the Static Set. `GetDynamicTools` lists and searches the Dynamic Set, and `CallDynamicTool` calls one of its tools.

The names and parameters copy Cursor's pair, and the search pattern is a regex because models that know Cursor send regexes. The Engine answers a mistake in a discovery call with text that tells the model what to do next. The `mcp` Pack has a pair of its own (section 9).

### Offer order sets list order, and a shared wire name fails the round

Tools appear in the list in the order of the offers. The tools of one Pack form one block, at the place of the section's first offer from that Pack, and an adhoc tool keeps its own place. The position of each offer line decides, because a hash order would vary from process to process, which would break prompt-cache hits, replay, and tests.

Two tools that share a wire name across the two sets fail the section's model call, with an error that names both. The exception is an adhoc tool: the Engine drops a Pack tool that shares its wire name in the same set. The clash fails the call so that a Pack bug or a name clash shows up as a clear error.

### `Pack::render` lets a Pack shape its tools for each model

At the first loop round, the Harness calls `render` on each Pack the section offered from. The Pack receives the offered tools and the round's model, and returns the tools the model will see. It may rename a tool, rewrite its description and schema, drop it, or add model-only tools. Each shipped Pack uses the default, which returns the tools unchanged. Contract 0.4 lacks a render verb, so the loader gives each library Pack the same default, and a render for a library Pack is still to be built.

One hook covers every Pack, so the Engine treats all tools the same way. The Engine requests the render as an effect: the Harness performs it, so the Engine is still pure, and the run log records it.

### Prompt prose describes tools by what they do

The model sees wire names, and a render can change them. Prose that tells the model to call `web/search` leads to a failed call when the tool list names the tool `web_search`. The shipped research prompt therefore describes each tool by its job, and its prose is true under any render.

## 4. Tools written in Lua

### `tools.adhoc` defines a tool in Lua, and `tools.offer` offers it

`tools.adhoc(attrs, f)` takes a name, a description, optional parameters, and a function, and returns a tool object. The tool's wire name is the name as written, and its id is `adhoc/<name>` with the name in lower case. The prompt passes that object to `tools.offer`, or calls the tool itself with `tools.call`. Defining a tool is separate from offering it, so every tool has one offer path, and a prompt can define a helper tool that only Lua calls.

### An adhoc handler runs inside the Engine, as section Lua

The handler runs in the coroutine of the Lua block that called it, and the Engine answers the call itself. A `tools.call`, a `store` call, or a `models.infer` inside a handler is an ordinary effect, and the Engine counts it as a script call. A handler that Rust called outside any coroutine would fail at every `store` call. Inside the block's coroutine, the handler's calls yield through the same shim as the block's own calls, and a chain needs only one coroutine.

### A failing adhoc handler raises at `models.loop`

When a handler raises an error, the Engine raises the same value at the place the tool was called. For a call the model made, that place is `models.loop`, so the loop ends unless the prompt wraps it in `pcall`. The Engine raises the value unchanged so that a structured error table still has its `kind`. The Engine sends a successful result to the model as written, marked trusted, because the prompt author wrote the handler. The exception is a handler whose own `tools.call` read an untrusted tool listing: the Engine wraps that handler's result and marks it untrusted (section 11).

The Engine raises because a handler is the author's own Lua, so its failure is a bug in the prompt. A prompt that wants the model to read a handler's failure calls `pcall` inside the handler and returns the text.

### An adhoc handler follows three rules

1. Return one scalar. The Engine uses the first return value as the tool's result: `nil` becomes the empty string, and a string, a number, or a boolean becomes its text. A table fails the call with the error "cannot return a table as a result".
2. Call `jump` from the block, after the tool call returns. Inside a handler `jump` raises an error, because with a jump there `models.loop` would need rules for reporting a batch that the jump abandons partway.
3. Use `models.infer` for a model call. A `models.loop` that a handler starts while the section's loop runs raises an error, because its records would precede the outer batch's.

A block's `return` turns its value into text the same way and refuses a table with the same error. The two differ for `nil`: a handler's `nil` is the empty string, and a block's `return` of `nil` ends its chain with no text ([The prompt language](prompt-language.md)).

## 5. Tool calls

### The Engine emits a `ToolCall` effect, and the Harness calls the Pack

The Engine has each Pack tool's id, schema, and description, and the Pack has the code. When a run needs a Pack tool, `Run::step` returns a `ToolCall` effect and that part of the run waits. The Harness finds the Pack from the first segment of the tool id, calls `Pack::call`, and resumes the run with one answer: the output, the tool's error, or `Dropped`. The effect names the section, the caller, the section entry that made the call, and, for a model's call, the model whose round asked for it.

This split defines the Engine and the Harness. The Engine returns the work it needs, and its caller performs that work. The Engine is therefore independent of any async runtime, and it is a deterministic function of its inputs.

### The model calls offered tools, and Lua calls any Pack tool

The model calls the tools in its section's two sets, by wire name. Lua calls any tool in the run's catalog by id with `tools.call`. A Rust Pack can flag a tool `model_only_call`, and a script's call of such a tool raises an error of kind `model_only_tool`. The flag is part of the catalog entry because Lua must be able to name the tool when it offers it. Contract 0.4 gives a tool a name, a wire name, a description, and a parameter schema, so the four flags are still to be built for a library Pack.

### A Pack tool's failure returns to the model as text and raises in a script

When the model made the call, the Engine converts the failure into the call's result, wrapped as untrusted text, and the loop continues. When a script made the call, the failure raises at the call site with kind `tool`. The model receives the failure as text so that it can read it and recover, because the failure of a Pack tool's backend is outside the prompt author's control.

Two failures raise at `models.loop` itself. A model call to a name outside both sets fails the whole round with `out_of_scope_tool`. A call that the Harness drops, after a stop or a cancel, raises the canceled error.

### The Engine issues one round's Pack tool calls together

When one model reply asks for several Pack tools, the Engine issues all the calls in one step, so they run at the same time. The loop appends the round to the messages as a whole, in the order the model listed the calls. An adhoc tool or a discovery tool runs by itself, between batches.

The calls run at the same time because they are independent of each other, and the model would otherwise wait for them one at a time. The loop appends the whole round because model APIs reject a request unless every requested call has a result.

### Each call works under its own file identity

The Engine forks a file identity for each tool call when it issues the call, and joins it back when the Harness answers. The calls of one round therefore count as concurrent. When two of them touch the same path and one writes, the file system returns `VfsError::Conflict` to the operation that ran second, and the run continues ([Files and the VFS](files-and-the-vfs.md)). A Rust Pack receives that error, and the loader hands a library Pack the kind `RV_FILE_ERROR_CONFLICT` with a sentence that names the path. Concurrent calls that shared one identity would race, and the file system would miss the conflict. Each shipped tool reads files only: `web` and `mcp` leave the access unused, and contract 0.4 has three file functions, which stat, list, and read (section 8).

### The Harness runs at most 10 calls at once and starts waiting calls oldest first

A run has at most 10 Pack tool calls in progress. The Harness queues the rest and starts the call with the lowest effect id when a slot frees. A Host sets the limit through `RunSetup::max_tool_calls_in_flight`.

The default follows other harnesses, which cluster at 8 to 10: Claude Code 10, Mastra 10, and CrewAI 8. The Harness owns the queue because a stop has to drop waiting calls as well. A queue in the Engine would hide waiting calls from the Harness, and a stop would miss them. Waiting calls start oldest first. The owner's reason: "If we get a batch, then we want to finish that batch."

### A stop drops the calls in flight, and a message waits for them

A stop drops every tool call in progress or waiting and leaves the run's cancel flag clear. Each dropped call raises the canceled error at its call site, so the run continues when the prompt catches that error with `pcall`, as the built-in chat prompt does. A cancel drops every call and ends the run. A message sent while tool calls run waits: the tool calls finish, and the loop appends the message before its next model request. A message sent while a model request is unanswered withdraws that request, and the loop sends it again with the message appended. A message waits because a tool that is cut off may have changed something, and a stop is the way to cut tools short.

A tool whose `ToolInfo` sets `survives_stop` continues to run through a stop. The shipped Packs leave the flag unset.

### A loop runs up to 24 rounds, and the Host sets a ceiling

`models.loop` runs up to 24 model rounds by default. A prompt declares its own limit with `max_tool_iterations`, because the budget belongs with the prompt that needs it. The Host's ceiling, 1000 by default, caps the declared value.

## 6. Trust of tool output

### The Pack marks each output, and the Engine wraps untrusted text

`Pack::call` returns `ToolOutput::trusted(text)` or `ToolOutput::untrusted(text)`. A library Plugin ends a call with the result kind `RV_RESULT_TRUSTED` or `RV_RESULT_UNTRUSTED`, and the loader builds the same `ToolOutput` from it. A Pack marks as untrusted any output that embeds data an attacker can influence. The Engine wraps untrusted text in an envelope before the model or the script receives it. The exception is a script's call of a tool flagged `describes_tools`: the script receives the untrusted listing as written (section 11). The code's docs describe the envelope as "defense in depth, not a security boundary".

### The envelope is a tag named with a per-run nonce

The envelope is an XML-style tag whose name includes a nonce. A sentence at its top tells the model that the enclosed text is data. A run has one nonce, derived from the run's seed with SHA-256.

Each result includes the sentence because a rule stated next to the data aids compliance. The nonce is the same across a run because a fresh nonce for each call would break the byte-identical envelopes that prompt caching needs. The hash is one-way because a reversible derivation would let a reader of one nonce compute the seed.

### The Engine escapes `<`, control markup, and the nonce inside wrapped text

The Engine writes each `<` in wrapped text as `&lt;`, so a fake closing tag inside a page is plain text, and the data block ends only at the real tag. It puts a space after the first character of each chat-template delimiter in a fixed list, so `[INST]` becomes `[ INST]`. It breaks each copy of the run's nonce the same way, because mimicry of the delimiter is the documented counter-attack against nonce envelopes.

The list of delimiters is closed on purpose. The `AGENTS.md` of the `runvil-types` crate states the rule: "The control-markup inventory is closed: add table entries, each with a family rationale, and never generalize the matcher."

### The envelope lowers the risk of injection, and a section's offers decide which tools a misled model can call

A model can still follow an instruction inside the envelope. The envelope only lowers the probability that it does. The checks of `web/fetch` stop a request to an internal address. They leave the query string of a public URL as written, so a model that follows an instruction in a fetched page can put run data into it. The URL check accepts such a request because the destination is public and the payload looks the same as an ordinary query.

The control that works is the prompt author's, applied with `tools.offer`: the author leaves the tools that could exfiltrate text out of each section that reads untrusted text.

### Lua reads metadata and decoded JSON as plain text

Lua reads `tool.description`, `tool.parameters`, and a Pack's `name` and `tools` as the declared text, outside the envelope. The owner's rule: "when Lua asks for tool descriptions and such, it should never arrive in the untrusted envelope". Strings from `json.decode` are plain Lua strings as well, and an author who sends them to a model wraps them with `untrusted(s)`.

### The shipped Packs mark outside text as untrusted

`web/fetch` and `web/search` return untrusted output. `std` marks file content and file names as untrusted. It marks as trusted its fixed stub text, and a Grep answer for zero matches that holds only fixed text and counts. `bash/shell` marks the output of each command as untrusted. A discovery listing is untrusted because Packs and their servers write the descriptions. `tasks.receive()` returns the operator's messages as plain text, because a wrapped answer would hide the operator's own words from the script.

### The `mcp` Pack marks its output by caller

The `mcp` Pack reads the caller from its `ToolContext`. For a call the model made, it marks the server's text untrusted. For a script's `tools.call`, it marks the same text trusted, so the script receives the raw string and can decode the listing JSON. The cost is that a script that forwards MCP text into a prompt sends it unwrapped, and that is the script author's decision. Section 11 describes a conflict between this rule and the Engine's rule for tool listings. Only a Rust Pack receives the call's origin: the loader passes a library Pack the tool's name, the arguments, and the file access, so the contract needs the origin before `mcp` can move to it (section 10).

## 7. Writing a Pack

### A library Plugin is written against `runvil-plugin-sdk`, and a Rust Plugin against `runvil-plugin`

Two crates define what a Pack author writes against, and each shipped Plugin uses one of them. `std` and `bashkit` are written against `runvil-plugin-sdk`, the safe layer over the C contract (section 8). The author implements the SDK's `Plugin` and `Pack` traits, and `export_plugin!` exports the entry point. Each of the two depends on that crate and on libraries from outside Runvil, so it builds independently of the Harness's commit. `plugin-web` and `plugin-mcp` are written against `runvil-plugin`, which defines the `Pack` trait, `PackFactory`, `PackInfo`, `PackRegistry`, `ToolContext`, and `HostServices`, and re-exports the Engine and file-system types an author needs. The `## Invariants` section of that crate's docs is the rule list for its Pack authors. A Host imports the same items from `runvil::plugin`.

The two crates are two APIs, and the design is one. The Rust API is the older one, and it has abilities that the contract lacks (section 11). The move of the two Rust Plugins onto the contract is planned work (section 10). The rest of this section describes the Rust API, and section 8 describes the contract.

### A Rust Plugin is a function named `install`

The Host calls `install(packs, config, services)`. For each Pack, the function reads `config[<pack id>]` and the Host's services, builds a factory, and calls `PackRegistry::insert` with the Pack's `PackInfo` and the factory. When setup fails, it calls `insert_unavailable` with a reason.

The Host owns the registry, and Plugins fill it. The owner's rule: "A host can only install plugins". The `install` function is the Rust-only way into the registry. A library Plugin inserts its Packs through the contract's `insert_pack`, and the loader calls `PackRegistry::insert` for it (section 8).

### The Harness makes one Pack instance for each run

The Harness calls `PackFactory::new_pack(id)` once for each Pack in a run. The instance drops when the run ends, and its `Drop` does the cleanup. The factory owns the state that runs share, such as MCP connections and HTTP clients.

A Rust Pack is per run because the C contract has the same design for library Plugins, with verbs that create and delete a per-run Pack (section 8).

### `new_pack` returns at once, and `install` builds what the runs share

The Harness calls `new_pack` while it prepares the run, so a factory returns quickly: it clones the parts that `install` built and stores them in a new instance. The `web` factory clones its two clients, and the `mcp` factory clones its list of servers. `new_pack` runs inside the run's future, so the crate's docs forbid it to block, do I/O, or panic.

### The `Pack` trait has four methods

`tools()` returns the Pack's tool list, which the Harness reads once per run. `call()` performs one tool call. `ready()` resolves when the tool list is complete. `render()` shapes the offered tools for a model.

### `ToolContext` lends each call the tool id, a file access, and the call's origin

`cx.tool()` returns the id of the called tool. `cx.access()` returns the call's file access, rooted at `/`. `cx.origin()` says which section and which caller made the call. The context's fields are borrows that end with the call, so the Rust compiler rejects a Pack that stores the access.

The owner's rule: "The tool can't keep the access. ... it has to give it up, after the call is done". A Rust Pack reads the services it needs at `install` and stores them in its factory.

### `Pack::call` follows five rules

1. Return to the poll quickly. A call sends blocking or CPU-heavy work to the Host's runtime, because the Harness polls every effect of a run on the same task.
2. Report failure as an `Error`. The Harness answers a panicking call as `Dropped` and logs the panic.
3. Mark as untrusted any output that embeds data an attacker can influence.
4. Expect to be dropped. The Harness drops the call's future on a stop, on a cancel, and when the calling task ends.
5. Finish independently of the run's other calls. A call that waits for a sibling queued behind the limit of 10 can stall the run.

### File operations through the access are synchronous

A Rust Pack's tool reads and writes files through `cx.access()`, and each operation runs inside the call. The reason is that the Harness is independent of any particular async runtime, so each part of the context has to be independent of tokio. The shipped Rust Packs, `web` and `mcp`, leave the access unused. The shipped tools that touch files are in the library Plugins `std` and `bashkit`, and they read through the three file functions of the contract, which the loader answers from the same access (section 8). A long file job yields so that the Harness can cancel it. Grep, in the `std` library, ends its step after every 32 files or directories, or after 2 MiB read, so a stop or a cancel drops a long search between two steps.

### The run's mode policy decides each file change

A tool performs every file operation on the call's access, and the file system applies the run's mode policy to each one. In Ask mode it refuses each change, and in Plan mode it accepts changes to markdown paths. The check is in the file system because every operation passes through the access, so the check covers every tool, including each tool written later. Workshop's runs use the file system's default policy, `AllowAll`, and tests are the callers of `ModePolicy` ([Files and the VFS](files-and-the-vfs.md)).

### Host services are named, typed objects that `install` reads

A service has a two-segment id, such as `runvil/tokio-runtime`, and a `ServiceKey<T>` binds the id to a Rust type. The Host fills one `HostServices` map and passes it to each Plugin's `install`. A Rust Pack that needs an async runtime reads the Host's runtime handle from that map. The C contract passes a Plugin the contract version alone, so the bashkit Plugin starts a runtime of its own, with one worker thread, for its timers.

### A Pack's Lua prelude runs once in each section VM

For a prompt that declares the Pack, the Engine runs the Pack's prelude in each section VM, before `require` and before the author's code. The prelude receives the Pack id through `...` and defines globals, which the Engine then seals. Each Rust Pack that ships leaves its prelude unset. A Pack from the C contract has a name and a description, and a prelude for it is still to be built (section 8).

## 8. The C contract

### Contract 0.4 has six verbs

A dynamic library exports one function, `runvil_plugin`, and each call passes an `RvCall` struct that names a verb. Verbs 1 and 2 load and unload the Plugin, and a load names the Plugin and inserts its Packs. Verbs 3 and 4 create and delete one run's instance of a Pack, and the instance lists its tools when it is created. Verb 5 runs one step of a tool call, and verb 6 frees the state of a call that the Host abandoned. Numbers 7 and 8 are reserved for later versions.

A single function means one exported symbol, and a new version of the contract adds verbs. Contract 0.2 had the first two verbs, which proved the loading mechanism before the rest was designed. Version 0.3 added Packs and tool calls, and version 0.4 added the file functions.

### The contract has its own repository, and each 0.x version is a separate contract

`runvil-plugin-api` contains the C header, the Rust mirror `runvil-plugin-sys`, and a safe SDK, `runvil-plugin-sdk`. While the major version is 0, a Plugin built for 0.4 loads only in a Host that reports 0.4. The Plugin reads the Host's version first in `RV_PLUGIN_LOAD` and answers any other version with `RV_ERR_UNSUPPORTED`. The contract has a repository of its own so that a Plugin depends on that repository only.

### Each callback belongs to one verb and takes a `ctx`

The functions a Plugin may call back are fields of a verb's arguments struct. For a load, `RvLoadArgs` has `ctx`, `refuse`, `describe_plugin`, and `insert_pack`. `RvPackNewArgs` has `refuse` and `add_tool`, and `RvToolCallArgs` has `finish` and three file functions. Each function takes the `ctx` first and answers only while its call runs, on that call's thread. A callback belongs to one verb because each verb has its own lifetime and thread rules.

A tool call is a series of steps. The Host sends `RV_TOOL_CALL` once for each step, a step that returns `RV_PENDING` asks for another, and a step ends the call by calling `finish` once with its result. The call has steps because a step that blocked would stall its run, and the Host can abandon a call only between two steps. A step reads the run's files through `file_stat`, `file_list`, and `file_read`, so the run's policy applies to every read. Functions that change a file are still to come: contract 0.4 reads only.

### A library Plugin and a C entry point compiled into the Host receive the same calls

The loader has two functions: `insert_plugin_dir` loads every dynamic library in a directory, and `insert_plugin` loads a C entry point that is compiled into the Host. The loader names the two kinds `PluginSource::Library` and `PluginSource::Linked`. Each entry point gets the same calls, so one path loads both kinds. The owner's rule: "those have to be treated exactly the same. In other words ... the entry point has the same shape". For each Pack that a Plugin inserted, the loader puts a factory in the registry. The factory sends `RV_PACK_NEW` for each run, the instance sends one `RV_TOOL_CALL` each time a call's future is polled, and its drop sends `RV_PACK_DELETE`.

`std` and `bashkit` are library Plugins, and tests are the only callers of `insert_plugin`. The shipped Plugins that are compiled into the Host, `plugin-web` and `plugin-mcp`, are a third kind. Each is a Rust `install` function written against `runvil-plugin` (section 7), which the Host calls itself, so the contract's calls go to `std` and `bashkit` alone. The docs of `runvil-plugin` call that third kind a linked Plugin as well. Section 11 describes the gap, and section 10 lists the move of the two Plugins onto the contract.

A Pack from the contract has tools only, and a tool has a name, a wire name, a description, and parameters. The four `ToolInfo` flags, the prelude, `ready`, `render`, the call's origin, a configuration, Host services, and a Pack inserted as unavailable exist in the Rust API alone. The contract is still to gain the ones that a shipped Pack uses (section 10).

A loaded Plugin is reference counted. The `PluginReport`, each of its Packs in the registry, and each Pack instance of a run hold it, so it unloads after its last Pack instance is deleted. `insert_plugin_dir` records a library that fails to load in the `PluginReport` and continues with the next one.

Loading happens in the Harness, and the Engine receives data: the tool catalog, the roster of the run's Packs, and the prelude text. The reason is that the Engine is pure: tool calls are effects that the Harness performs, and a replay can feed recorded results back to the Engine without the Plugins present.

## 9. The Packs that ship with Runvil

### `std` copies Cursor's file tools

`std` is a library Plugin. Its code is in the `runvil-plugin-std` repository, written against `runvil-plugin-sdk`, and Workshop loads its library from the Plugin directory ([Workshop: how the app is built](workshop-app.md)). The Pack has seven tools. `std/read`, `std/dir`, and `std/grep` work. `std/write`, `std/replace`, `std/rm`, and `std/glob` answer a fixed text that says the tool is missing from this build. The model sees Cursor's names, such as `Read`, `Grep`, and `StrReplace`. Six definitions are Cursor's with the edits that `crates/plugin-std/schemas/EDITS.md` in that repository lists, and ListDir's is the crate's own. All seven are JSON files.

The Pack copies Cursor's text because the model was trained against that exact text, and a stored copy can be diffed against Cursor's. A stub answers as a success so that calls to the Pack exercise both success and failure from end to end. Walks skip `.git` and list the other dotfiles, because a model often needs to edit `.gitignore` and similar files. A walk leaves out a name that contains a control character and counts it, because a line feed in a name would forge a line of the answer. The shell is in a Pack of its own, `bash`, so a run that wants a shell declares it.

### `bash` runs a shell inside the run, and the shell reads the run's files

`bash` is the Pack of a library Plugin. The bashkit Plugin has a repository of its own, `runvil-plugin-bashkit`, and is written against `runvil-plugin-sdk`. It reports the name `bashkit` and inserts one Pack, `runvil/bash`. The Pack has one tool, `bash/shell`, and the model sees it as `Shell`, which is Cursor's name for its shell tool. The definition is Cursor's, with the edits that `crates/plugin-bashkit/schemas/EDITS.md` in that repository lists. Workshop pins the Plugin by commit, builds its library through the crate `workshop-plugin-bashkit`, and loads it from the Plugin directory, beside the library of `std` ([Workshop: how the app is built](workshop-app.md)). The built-in chat prompt requires `bash` under `uses:` and offers it.

The shell is a bash interpreter from the `bashkit` crate, and it runs inside the run. Every command is built into bashkit, so the shell works without a child process and without a network. A run has one session, which the first call makes, and the working directory, variables, and functions that one command leaves are there for the next call. A call that the Host abandons takes the shell with it, and the next call starts a new session, because a script that is cut off partway may leave the shell in a state that an ordinary command would avoid. A command ends after 30 seconds, or after the `block_until_ms` of its call when that is less. The answer is the exit code and the output, marked untrusted.

The shell's files are the run's files. Bashkit keeps one `'static` file system for as long as the shell lives, and a tool call lends its files only while one of its steps runs. The Pack's file system, `RunFiles`, therefore holds a channel in place of a call: each read waits in the channel as a request, and the call that runs the script performs it through the contract's file functions and replies. Every read goes through the Host, so the run's policy applies. A step ends after 32 reads, as a Grep step ends after 32 files, so the Host can abandon a script between two steps. A command that changes a file fails with one fixed sentence, the constant `READ_ONLY` in `crates/plugin-bashkit/src/files.rs`, because contract 0.4 reads only (section 8). The Pack's plan calls this the first of two stages, and the second stage adds writing (section 10).

### `web` provides search and fetch together

`web/search` calls a search provider that the Host supplies as a service, and the provider has the search key. `web/fetch` runs in process on the Host's runtime. When either service is missing, the Plugin inserts the whole Pack as unavailable, because a research prompt wants both tools together.

A fetch reports a failure of the target, such as a 404 or a timeout, as text the model can act on. The text states the status and the URL, and the Pack drops the error page's body, because the body is untrusted HTML and unhelpful for recovery.

### `web/fetch` connects to public addresses, and checks each address when it connects

The model supplies the URL, so `web/fetch` checks it twice. Before a request, it refuses a URL that uses plain `http`, contains a user name or password, names a port other than 80 or 443, or has an IP address as its host. When the client connects, a resolver of its own drops each DNS answer in a blocked range, such as a private or loopback address. Both checks run again on each redirect, up to 5 redirects. The client omits cookies and credentials.

The second check runs at connect time because a check that reads only the URL string is defeated by a name that resolves to an internal address and by DNS rebinding.

### A fetch returns a page's text under a three-line header

`web/fetch` takes a `url` and two optional arguments: `raw` renders the whole page, and `max_chars` lowers the limit of 40,000 characters. The tool extracts the main article of an HTML page as markdown, returns JSON, XML, and plain text as written, and refuses every other type and a response that omits `Content-Type`. It reads up to 8 MiB of a body, counted after decompression, so a gzip bomb hits the same cap.

The result starts with the lines `url:`, `truncated:`, and `extraction:`. The first is the final URL after redirects, so a citation names the address that returned the bytes, which can differ from the one the model supplied.

### `mcp` is one Pack with two tools for remote servers

The Pack takes the whole `mcpServers` object as its config and connects to each server in the background. It offers `mcp/get_dynamic_tools`, which lists the servers and their tools, and `mcp/call_dynamic_tool`, which calls one. The listing shows each server as `ready`, `loading`, or `error`. The owner's rule: "there should just be 2 static tools that the mcp pack offers".

The client speaks Streamable HTTP through the `rmcp` SDK and returns results as text. An entry for a local server, one started with a `command`, shows as `error`. Local servers are still to be built, because each one needs a launcher program (`npx`, `uvx`), a job object or process group so that the server dies with Workshop, and handling for Windows `.cmd` shims.

### A server entry has the format of Cursor's `mcp.json`

A remote entry has a `url` and optional `headers`. The Pack expands `${env:NAME}` and `${userHome}` in an entry's strings, refuses an entry that names an unset variable, and leaves every other `${...}` form unchanged, as Cursor does. It ignores unknown fields because the format is defined outside this repository.

The Pack refuses the legacy `sse` type and `headers` that set `accept`, `Mcp-Session-Id`, or `Last-Event-Id`. The `rmcp` SDK sets those three headers, and the Pack refuses them at install, since `rmcp` would refuse such an entry only on the first request. A refused entry shows as `error` in the listing, with the reason, and the other servers still connect.

### The `mcp` Pack names the keys of `env` and `headers` and omits their values

A value in an entry's `env` or `headers` is often a secret. Each refusal reason, log line, `Debug` output, and tool error of the Pack may name a key and omits the value. The Pack leaves the values out because a reason appears in the notice of a refusal, and a secret inside it would appear there too.

The Pack uses `rmcp`'s own HTTP client, which refuses redirects, so the client sends the configured headers to the configured server only. One case is still open: `rmcp`'s own logs can name a URL that contains an expanded variable.

### `user` marks a prompt that needs a person

`runvil/user` is built into every registry. A prompt that needs a person declares `uses: [user]` and reads the person's messages with `tasks.receive()`. When `RunSetup.has_user` is false, the Harness marks the Pack unavailable, and the run's first step refuses a prompt that declares it. The Pack's tool list is empty. A model asks the person a question in its reply: the reply ends `models.loop`, and the prompt then returns to `tasks.receive()`.

### `subagents` reserves a name

`runvil/subagents` is built into every registry and reserves its name for subagent tools that the Engine will provide. It is a Pack so that a prompt can name it under `uses:`. `tasks` is a language feature, because an option to leave the `tasks` table out of a run would be useless.

## 10. Planned work

Each of these items is still to be built.

- **One Plugin API.** The Rust Plugins `plugin-web` and `plugin-mcp` written against the C contract and inserted with `insert_plugin`, and the additions to the contract that they need first. Both are `install` functions written against `runvil-plugin` (section 7), Workshop calls both, and `insert_plugin` has tests as its only callers. The two Packs use seven things that contract 0.4 lacks. `mcp` uses a configuration, for its list of servers, the call's origin, to mark its output by caller, and the `describes_tools` flag, on its listing tool. `web` uses the Host's search provider. Both use the Host's runtime handle, a Pack inserted as unavailable with a reason, a call that waits on work outside its step, and error causes. After the move the `install` functions are removed, and a Pack that is compiled into the Host goes through the same contract as a Pack from a library (section 8).
- **File changes through the C contract.** Functions with which a Plugin's tool writes, deletes, and renames the run's files. Contract 0.4 has three file functions, which stat, list, and read. Two shipped Packs wait for the rest. The `std` tools Write, StrReplace, and Delete answer a fixed stub text, and the `bash` shell refuses each change with one fixed sentence (section 9). The Rust API has the change operations on `cx.access()` (section 7), and the shipped Rust Packs, `web` and `mcp`, leave the access unused.
- **The `std` tools that are stubs.** Write, StrReplace, Delete, and Glob. Each answers a fixed text that says the tool is missing from this build. The first three wait for the file changes above.
- **Approval by a person.** The step that asks the operator to approve a file change. When the run's policy answers `Ask`, the file system refuses the operation with `VfsError::ApprovalRequired` and a reason. A Rust Pack's tool receives that error, and the loader hands a library Pack's tool a denied file error with the same reason. The shipped tools read files only, and Workshop's runs use `AllowAll`, so the error is out of reach of a run under Workshop.
- **Ignore files.** Applying `.gitignore`, so that Grep skips a folder such as `target/` and spends its budget of 100,000 files elsewhere. Walks skip `.git` and names that contain a control character.
- **Writing and a thread of its own for the `bash` shell.** Shell commands that change the run's files, and bash on a thread of the Plugin's own, with a wake function in the contract. The `bash` Pack runs commands and reads the run's files (section 9). Bash runs inside the steps of its call: a heavy script holds its step, and the Stop button with it, until the script ends or reaches one of bashkit's limits, and while a script sleeps the Host steps the call with nothing to do. Writing waits for the file changes above.
- **Subagent tools.** The tools of `runvil/subagents`, and the wrapping of text that one model writes for another. The Pack only reserves its name (section 9).
- **MCP in the Dynamic Set.** The `mcp` Pack rendering its servers' tools into the Dynamic Set, which retires its own two tools. Local servers and OAuth are still to be built as well. The Pack offers its own two tools, for remote servers.
- **Filters and labels.** An offer narrowed by a hint such as read-only, and labels on `ToolInfo`, modeled on MCP's, for a future Ask mode. An offer names each tool or Pack, and `ToolInfo` has four flags: `structured_output`, `survives_stop`, `describes_tools`, and `model_only_call`.
- **Partial rounds.** Saving the finished results when a stop cuts a round short. The loop discards the whole round.
- **Fetch settings.** A way for a Host to change the fetch limits. The fetch policy has a setter for each limit, and the Pack accepts `null` or `{}` as its configuration, so each Host runs the defaults.

## 11. Known gaps

This section has five gaps: one conflict inside the code, two APIs that install Packs, one place where the language guide differs from the code, and a flag and a prelude that only tests set.

### Two trust rules overlap, and the model receives MCP server text unwrapped

Two rules decide the trust of a tool listing that a script reads. The `mcp` Pack marks its output trusted for a script's call (section 6). The Engine has a rule for tools flagged `describes_tools`: a script receives the tool's untrusted listing unwrapped, and the Engine then wraps the result of an adhoc tool that fetched the listing. Under that rule an adhoc wrapper of a listing tool produces the same envelope as a direct call by the model.

`mcp/get_dynamic_tools` is the one shipped tool with the flag. The Pack marks a script's listing trusted, so the Engine treats it as trusted text and skips its listing rule. An adhoc tool whose handler returns `tools.call('mcp/get_dynamic_tools')` therefore returns the servers' text to the model unwrapped and reported as trusted.

The evidence is a reading of `dispatch_body` in `runvil-internal/lua/src/dispatch.rs`, of `marked` and `Mcp::call` in `plugin-mcp/src/dynamic.rs`, and of two tests: `a_script_listing_is_trusted_json_equal_to_the_models` and `a_failed_listing_call_and_a_trusted_listing_mark_nothing`. The Engine's tests of the wrapper rule use a stand-in tool that returns untrusted text. A test with the real `mcp` Pack would confirm the behavior.

### Two APIs install Packs, and each has abilities that the other lacks

A Pack enters a Host's registry in one of three ways. A Rust `install` function, written against `runvil-plugin`, inserts it: `plugin-web` and `plugin-mcp` work this way, and Workshop calls both functions. A C entry point that is compiled into the Host inserts it, when the Host hands the entry point to `insert_plugin`. A dynamic library that exports `runvil_plugin` inserts it, when the Host loads the library with `insert_plugin_dir`: `std` and `bashkit` work this way. The second and third ways are one API, the C contract, and one private function of the loader loads both. The first way is a second API, built from Rust types of the Engine and the Harness.

The `web` and `mcp` Packs use seven things that contract 0.4 lacks (section 10). The contract has what `install` lacks: a Plugin reports a name and a version, and the `PluginReport` lists them. A call to `install` returns only a result, so Workshop's listing of the Plugins it loaded names `std` and `bashkit` and omits `web` and `mcp`. `insert_plugin` has tests as its only callers. Four more abilities of the Rust API have tests as their only users: the flags `structured_output` and `model_only_call`, and overrides of `Pack::ready` and `Pack::render`.

The word "linked" has two meanings in the code. The docs of `runvil-plugin` use "linked Plugin" for the `install` function of a Plugin crate. The loader's `PluginSource::Linked` and `insert_plugin` mean a C entry point that is compiled into the Host. Section 8 uses the second meaning.

The design is one API, and the move is planned work (section 10). A Pack written against the Rust API can use each ability that API gains, and a Pack from the contract waits for the contract to gain it, so the two sets of Packs drift apart with each addition.

The evidence is a reading of the two `install` functions, in `plugin-web/src/web.rs` and `plugin-mcp/src/lib.rs`, of `with_plugins` in `workshop/server/src/agents.rs`, and of every struct of `runvil-plugin-sys`, and a search for the callers of `insert_plugin`, which finds the loader's unit tests alone.

### The language guide differs from the code in one place

Chapters 4, 12, and 13 show the refusal of a missing Pack at the run's first step, as `missing required Pack: web (at file:line)`. Under Workshop, `Host::parse` refuses earlier, as `web is unavailable: runvil/web is not installed`. The guide's text is correct for a caller that steps the Engine itself.

### `survives_stop` and the Pack prelude have tests as their only users

A Rust Pack can flag a tool with `survives_stop`, and it can set a Lua prelude. The Engine and the Harness act on both, and each shipped Rust Pack leaves both unset. A Pack from the C contract has neither (section 8).

1. `survives_stop`. `ToolInfo::survives_stop` in `runvil-internal/types/src/tools/info.rs` sets the flag, and the Engine copies it to each `ToolCall` effect. On a stop, the Harness drops every tool call except a flagged one, through `Reach::AllButSurvivors` in `harness-internal/runner/src/effect_loop.rs`. `ToolInfo::new` sets the flag to `false`, each tool of `web` and `mcp` leaves it at `false`, the loader builds each tool of `std` and `bash` with `ToolInfo::new` and a wire name alone, and tests are the only callers of the setter.
2. The Pack prelude. `PackInfo::prelude` in `runvil-plugin/src/pack.rs` sets the prelude. `RunPacks::preludes` in `harness-internal/runner/src/packs.rs` collects the preludes of the declared Packs, and `install_preludes` in `runvil-internal/lua/src/prelude.rs` runs them in each section VM. `PackInfo::new` sets the prelude to `None`, the `PackInfo` values of `web`, `mcp`, `user`, and `subagents` leave it at `None`, the loader builds the `PackInfo` of `std` and of `bash` from a name and a description, and tests are the only callers of the setter.

## 12. Questions for the owner

Each question asks for the reason behind a part of the design.

**Tool calls**

1. Why does a handler that returns a table fail the call?
2. Why does a call to an unknown name fail the whole round, while a call to a Dynamic Set name returns text?
3. Why do adhoc tools run one at a time?
4. Why do events fire in the order of the answers while the messages are in call order?
5. Why is a script's `tools.call` a single call, outside the batch path?
6. Why is the default 24 rounds?
7. Why does a file conflict inside a tool return to the model, while the same conflict in `store.*` ends the run?
8. What is `survives_stop` for, when the shipped Packs leave it unset?

**Packs**

9. Why does the registry leave a Plugin's Packs in place when its `install` fails partway?
10. Why does `Host::parse` check `uses:` when the run's first step checks it too?
11. Why is the need for a person expressed as a Pack?
12. Why is every `uses:` entry required?
13. Why do `web`, `std`, and `bash` wrap output for scripts while `mcp` returns it raw?

**The C contract**

14. Why do verbs 7 and 8 and status codes 1 to 3 stay reserved after the header dropped the uses that it named for them?
15. What does a new Pack that is compiled into the Host use until the contract has what `web` and `mcp` need: a Rust `install` function, or a C entry point handed to `insert_plugin`?
16. What happens to the `Pack` trait, `HostServices`, and `ToolContext` of `runvil-plugin` once `plugin-web` and `plugin-mcp` are on the contract?

## 13. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64`, `runvil-docs` at commit `cfd69a4`, and `runvil-plugin-api` at commit `63a2403`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-11 the text on library Plugins, the C contract, and the `std` Pack was brought to `runvil` at commit `b0693699d`, `runvil-plugin-api` at `03a9fa3`, and `runvil-plugin-std` at `782ad23`. Between the two `runvil` commits, `std` became a library Plugin in a repository of its own, and the contract went from 0.2 to 0.4. The update covers the summary, the walk-through, three entries of section 7, section 8, the `std` entry of section 9, three items of section 10, and question 14. The rest of the page still describes the commits above.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The two Rust Plugins, `plugin-web` and `plugin-mcp`, are among them, and Workshop takes both from that repository as git dependencies at a pinned commit ([Crates, build, and CI](crates-build-and-ci.md)). The text of the page still describes the code at the commits above, when those crates were in `runvil`.
- **Update.** On 2026-11-13 the text on the two Plugin APIs, the `bash` Pack, the planned work, and the gaps was brought to `runvil-harness` at commit `18361d02`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, `runvil-plugin-bashkit` at `9c8e980`, and `runvil-workshop` at `f386801b`. Between the earlier update and these commits, the bashkit Plugin gained the Pack `runvil/bash`, Workshop began to ship that Plugin's library and to require `bash` in the built-in chat prompt, and Workshop's crates moved to the repository `runvil-workshop`. The update covers the Paths bullet, the summary, the walk-through, entries in each of sections 1 to 8, a new entry on `bash` and one sentence of the `std` entry in section 9, four items of section 10, a new gap, the gap on the language guide, and the gap on `survives_stop` and the prelude in section 11, and questions 13, 15, and 16. The earlier text listed seven places where the language guide differs from the code. The same session corrected six of them in chapters 11, 12, and 16 of the guide, in the working tree of `runvil-docs` over commit `c03ec91`, and the seventh is the one that section 11 keeps. That session read the code and left the build and the tests unrun. The rest of the page still describes the commits above.

*2026-11-10 16:46 - Claude Fable 5.2*
