# Tools and Packs: design decisions and their reasons

- Scope: tools, Packs, Plugins, the Plugin C contract, and the Packs that ship with Runvil
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil-workshop`

## Summary

A Plugin installs Packs, a Pack installs tools, and a section's Lua offers tools to its model. The Engine asks for each tool call as an effect, and the Harness runs it. A tool asks for each file operation that it needs, the Engine issues that operation as an effect of its own, and the Harness performs and records it. Sections 1 to 9 describe that design and its reasons.

Section 11 lists the gaps between this page and the code, and section 12 lists the questions about reasons for the owner. The largest gaps are in the code. The trust rules overlap, and together they send MCP server text to a model without the untrusted envelope. The C contract and a Rust API both install Packs, and each has abilities that the other lacks. StrReplace and Delete of `std` change a file in more than one request, and another task can change that file between the requests.

Section 10 lists the planned work. Its largest item is one Plugin API, which moves `plugin-mcp` onto the C contract. Some items wait for a decision of the owner: a thread of its own for bash, a removal in the contract that takes a file only, and an edit that keeps another task's change.

## From a Plugin to a tool call

A Plugin is an entry point that installs Packs. The Plugins that ship with Runvil are `std`, `bashkit`, and `web`, dynamic libraries that export the C function `runvil_plugin`, each in a repository of its own, and the Rust function named `install` in `plugin-mcp`. A Pack installs tools, and a Rust Pack can also set a Lua prelude. The Pack `runvil/web` installs the tools `web/search` and `web/fetch`.

A Host names its Plugins on the builder of its `Host`, and `HostBuilder::build` makes a `PackRegistry` and runs the steps in the order written: it calls the function of a Rust Plugin with the registry, and it has the Harness load each Plugin of the C contract, a library from a directory or a file or an entry point that is compiled into the Host, and insert the Packs that Plugin offers. Each Pack enters the registry together with a factory, and the registry holds one Pack for each Pack id. When a run starts, the Harness asks the factory of each Pack in the registry for that run's Pack instance, waits for the instance's `ready`, and reads its tool list. The ways into the registry, the C contract and the Rust API, have different abilities, which is a gap (section 11). The plan is one API, the C contract, for every Pack (section 10).

A prompt lists the Packs it requires under `uses:`. Each section's Lua then offers tools to its model with `tools.offer`. At the section's first `models.loop` round, the Engine builds the tool list from those offers and sends the same list with every later round.

When the model calls a tool, the Engine emits a `ToolCall` effect and waits. The Harness finds the Pack from the first segment of the tool id and calls `Pack::call`. The effect and the call hold the tool's id, its arguments, and the call's origin, and they hold no file access: a tool asks for each file operation, and the Harness performs it.

Take a call of `std/read`, which the model sees as `Read`. `std` is a library Pack, so the loader runs the call as a series of steps and sends the Plugin one `RV_TOOL_CALL` for each. The first step asks what the path names: it calls the file function `file_stat`, which takes the request, and returns `RV_PENDING`. The loader makes the request through the files of the call, and the Harness reports it to the Engine with `Run::request_file`. The Engine's next step issues the operation as a `Vfs` effect that names the call. The Harness performs it on the run's file system, under the run's policy, records the effect and a receipt of its outcome, and hands the outcome to the call. The call's next step receives the answer and asks for the file's bytes with `file_read`, and the same round trip follows. The step after that calls `finish` with the file's numbered lines, marked untrusted, and returns `RV_OK`.

The Pack returns text marked trusted or untrusted. The Engine wraps untrusted text in an envelope and appends the result to the section's messages.

## 1. Plugins, Packs, and names

### "Pack" names the set of tools, and "Plugin" names the entry point that installs it

A Pack installs tools, and a Rust Pack can also set a Lua prelude. A Plugin is an entry point that installs Packs. "Plugin" is the usual word for what a loadable crate or library provides, and the names of the Plugin crates, such as `plugin-std` and `plugin-web`, fit it. A Plugin that follows the C contract reports a name and a version, which the Host lists. A Rust `install` function reports neither.

### A Pack id is the second segment of the Pack's name

A Pack's fully qualified name has two segments, a vendor and a name, as in `runvil/web`. The name segment is the Pack id, and every tool id in the Pack starts with it, as in `web/fetch`.

### A registry holds one Pack for each Pack id

`PackRegistry` refuses a Pack whose id it already holds, whatever the vendor of either Pack. The first Pack that a Plugin inserts keeps the id, and the second insert fails with the reason `pack_id_taken` and a message that names the holder, as in ``Pack `acme/web` cannot take the Pack id web, which `runvil/web` holds``. A Pack that a Plugin inserted as unavailable holds its id too. The Harness makes the built-in Packs, `subagents` and `user`, itself, so a registry starts empty and holds only what Plugins inserted. The registry still keeps the id of each built-in: it refuses a Pack that takes one of them with the same reason, `pack_id_taken`, and a message that names the built-in, as in ``Pack `acme/user` cannot take the Pack id user, which the built-in Pack `runvil/user` holds``. The owner's rule: "runvil/web and acme/web cannot both be in the host's PackRegistry".

The rule follows from the tool id. A tool id starts with the Pack id and has no vendor, and the Harness finds the Pack for a tool call from that first segment. With one Pack for each id, `web/fetch` names one tool on a Host, and a prompt reaches every Pack of a run by its bare Pack id.

### Lua uses a tool's id, and the model uses its wire name

Lua code uses the id, such as `web/search`. The model sees the wire name, which by default is the id with `_` in place of each `/` and `.`. A Pack may set a different wire name, and the model sees `std/read` as `Read`.

The tool objects that Lua reads omit wire names, because the Engine may rename them, and prompt prose that names them would then be stale.

### A bare name under `uses:` means the `runvil` vendor

`uses: [web]` means `runvil/web`. A prompt writes another vendor's Pack with both segments, as `acme/web`. Under this rule each name has one meaning. If a bare name could mean a third-party Pack, a `runvil` Pack of the same name that ships later would silently take its place in an existing prompt.

The bare name is the one thing that sets the `runvil` vendor apart. A Pack of any vendor is in every run of its Host (section 2), and Lua reaches each one by its Pack id.

An entry must name the vendor that the Host installed the Pack from. On a Host that has `acme/poop`, the entry `poop` is refused with `poop means runvil/poop, which is not installed; write acme/poop to use that Pack`, and on a Host that has `runvil/web`, the entry `acme/web` is refused with `acme/web is not installed`. The Harness leaves the installed Pack of that id out of such a run, because the run's roster lists the id as unavailable, and the run's first step refuses the run.

The Engine reserves Pack ids for its own tools. A tool that a prompt defines in Lua gets an id under `adhoc`, and the discovery tools get ids under `dynamic`.

### A Pack is for a feature that works on something outside the run

The Engine provides `store`, `var`, `log`, and `models` in every section. A Pack provides a feature that works on something outside the run, such as the web or an MCP server.

The test for a new feature is whether it works on something outside the run. The store is part of the language because it is how a prompt moves data between sections and handles fanout. A Pack for budget, compaction, request shaping, or model routing would duplicate work that the Engine, the prompt's Lua, the Gateway, or the Harness owns, in that order.

## 2. Requiring Packs with `uses:`

### A `uses:` entry requires a Pack and runs its prelude

A prompt names Packs under `uses:` in its front matter. For each entry, `Host::parse` checks that the Host has the Pack and refuses the prompt when the Pack is missing. The Engine then runs the Pack's Lua prelude in every section of the prompt. A Rust Pack sets its prelude in its `PackInfo`. The C contract gives a Pack a name and a description, so a prelude for a library Pack is still to be built, and each shipped Pack leaves its prelude unset.

A prelude runs only for a declared Pack because it defines Lua globals, and so changes what a prompt's Lua can call. The Lua globals of a prompt therefore depend on its own `uses:` list and are the same on every Host.

Every entry is required. A prompt that can run with or without a Pack tests `tools.packs.get('name')`, which returns the Pack object when the run has the Pack.

### A run includes every Pack the Host installed, and a section opts in to the undeclared ones

A run includes every Pack in the Host's registry, whatever the Pack's vendor and whatever the prompt declares. A `uses:` entry adds no Pack to a run and removes none. It marks a Pack as required, so that the run is refused when the Pack is missing or cannot serve, and it runs the Pack's prelude. The owner's rule: "Packs are determined by the host, not the prompts. The frontmatter just indicates which packs are *required*."

The guide calls the undeclared Packs extras. A section offers them with `tools.offer(tools.packs.extras())`, as the built-in chat prompt does. The owner's rule: "extras should be a per-section setting. Disabled by default, and opt-in".

The Host decides because it is the party that installs Plugins. With this rule the Packs and tools that a prompt can reach on one Host are the same for every prompt, and the front matter of one prompt has no effect on what another prompt's run holds.

### A linked prompt's `uses:` requires its Packs for the run and runs their preludes in its own sections

A prompt may list other prompts under `prompts:`, and a run of it holds each of them for `execute` ([The prompt language](prompt-language.md)). `Host::parse` and the run's first step read the `uses:` of every one of those prompts. A Pack that a linked prompt declares is required for the run, so the run is refused when the Host lacks the Pack or the Pack cannot serve. The refusal names the linked prompt, as in `- review.md: web is unavailable: runvil/web is not installed`, and a Pack that two prompts declare counts once.

A linked prompt's `uses:` does nothing else to the run. The run still has every Pack in the Host's registry, one instance of each, and a tool of any Pack answers a call from any of the run's prompts. In each section, `tools.packs` marks as required the Packs under the `uses:` of that section's own prompt, and `tools.packs.extras()` lists every other Pack of the run. A caller's Packs are therefore the same before and after it executes a prompt.

A Pack's prelude runs in the sections of each prompt that declares the Pack, and in no section of a prompt that leaves it out. The owner's rule: "A prompt's sections should only have preludes injected for the packs it names. It should not inject preludes for packs named by other prompts in prompts: frontmatter". The Lua globals of a prompt therefore depend on its own `uses:` list, whichever prompt executes it, and two Packs whose preludes define one global collide only in a prompt that declares both.

`RunPacks::preludes` gives the Engine one prelude for each usable Pack that any prompt of the run declares, and the Engine keeps for each prompt the preludes of the Packs that the prompt declares, in the order of its `uses:`. A prelude that fails to load fails the section entry that installs it. For an executed prompt, that entry is its first section, so the failure is the error that the `execute` raises.

### A declared Pack can be refused at install, at parse, or at the run's first step

1. At install. A Rust Plugin whose setup fails inserts its Pack as unavailable, with a reason. The loader drops every Pack of a library Plugin whose load fails, and `Host::parse` then reports each one with the text `is not installed`.
2. At parse. `Host::parse` refuses a prompt when it, or a prompt it links, declares a Pack that the Host lacks or one that a Plugin inserted as unavailable.
3. At the run's first step. A factory that fails or panics, or a `ready` that returns an error, makes the Pack unavailable to that one run. The Harness also marks `runvil/user` unavailable in a run that lacks a user (section 9).

The third case exists because a factory's result is unknown at parse time. The next run calls the factory again.

### The Harness reads each Pack's tools once, when the run starts

The Harness makes the run's instance of each installed Pack, waits for each `ready`, and reads each tool list. The run uses that catalog until it ends. A run waits for every Pack, declared or not, because the tools of an undeclared Pack are in its catalog too. `Pack::ready` resolves within a bound that the Pack owns, so a Pack that cannot start fails its `ready` within that bound, and the run goes on without its tools. An error from `ready` is temporary. A Pack that recovers is available to the runs that start afterward.

### Lua reads one catalog for the whole run, and the Engine builds the model's tool list once for each section entry

The run's catalog lists every Pack tool's `ToolInfo` (its id, wire name, description, parameter schema, and flags) by tool id. `tools.get`, `tools.call`, and `tools.offer` look ids up in it. The Harness builds it before any Lua runs, so each lookup returns the same answer for the whole run. For each section entry, which is one execution of a section, the Engine builds what a model sees: the wire names, the Static Set, and the Dynamic Set (section 3).

The Harness fixes the catalog in advance so that the Engine is deterministic and leaves all I/O to its caller. One catalog covers the whole run because the catalog is the same for every model.

## 3. Offering tools to a model

### A section's Lua offers tools to its model

`tools.offer("web")` offers each tool of the `web` Pack. `tools.offer("web/fetch")` offers one tool. The model's tool list for the section contains the offered tools.

Each entry into a section starts a fresh Lua VM with an empty offer list. A prompt that wants the same offers in several sections puts them in one section and loads that section with `require`.

Offers are per section for isolation: a section's model sees only the tools that the section offered. The rule also stops a prompt with 20 tools from sending 20 schemas to the model of every section.

### `tools.offer` takes tools and Packs, as ids or as objects

Its single argument is a tool, a Pack, or an array of both, and each item is an id string or an object. A string that contains a slash is a tool id, and any other string is a Pack id. The owner's rule: "I think there should only be one api to offer tools".

### The first `models.loop` round builds the tool list, and later rounds reuse it

Each `tools.offer` call checks its argument and records the offer. The section's first `models.loop` round builds the tool list, and every later round of that section sends the same bytes. A call to `tools.offer` after that round raises an error that says "the section's tool list is fixed at its first models.loop round".

The reason is the provider's prompt cache. A tool list rebuilt every round can change the request prefix in the middle of a conversation, and the provider then discards its cache.

### `tools.offer_dynamic` builds a second set, which the model uses through discovery tools

The Static Set is the list of tools offered with `tools.offer`, and every round sends it. The Dynamic Set is the list of tools offered with `tools.offer_dynamic`. When the Dynamic Set has members, the Engine appends tools of its own to the Static Set. `GetDynamicTools` lists and searches the Dynamic Set, and `CallDynamicTool` calls one of its tools.

The names and parameters copy Cursor's pair, and the search pattern is a regex because models that know Cursor send regexes. The Engine answers a mistake in a discovery call with text that tells the model what to do next. The `mcp` Pack has a pair of its own (section 9).

### Offer order sets list order, and a shared wire name fails the round

Tools appear in the list in the order of the offers. The tools of one Pack form one block, at the place of the section's first offer from that Pack, and an adhoc tool keeps its own place. The position of each offer line decides, because a hash order would vary from process to process, which would break prompt-cache hits, replay, and tests.

Two tools that share a wire name across the sets fail the section's model call, with an error that names both. The exception is an adhoc tool: the Engine drops a Pack tool that shares its wire name in the same set. The clash fails the call so that a Pack bug or a name clash shows up as a clear error.

### `Pack::render` lets a Pack shape its tools for each model

At the first loop round, the Harness calls `render` on each Pack the section offered from. The Pack receives the offered tools and the round's model, and returns the tools the model will see. It may rename a tool, rewrite its description and schema, drop it, or add model-only tools. Each shipped Pack uses the default, which returns the tools unchanged. The C contract lacks a render verb, so the loader gives each library Pack the same default, and a render for a library Pack is still to be built.

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

### An adhoc handler follows rules for its result, for `jump`, and for model calls

1. Return one scalar. The Engine uses the first return value as the tool's result: `nil` becomes the empty string, and a string, a number, or a boolean becomes its text. A table fails the call with the error "cannot return a table as a result".
2. Call `jump` from the block, after the tool call returns. Inside a handler `jump` raises an error, because with a jump there `models.loop` would need rules for reporting a batch that the jump abandons partway.
3. Use `models.infer` for a model call. A `models.loop` that a handler starts while the section's loop runs raises an error, because its records would precede the outer batch's.

A block's `return` turns its value into text the same way and refuses a table with the same error. The two differ for `nil`: a handler's `nil` is the empty string, and a block's `return` of `nil` ends its chain with no text ([The prompt language](prompt-language.md)).

## 5. Tool calls

### The Engine emits a `ToolCall` effect, and the Harness calls the Pack

The Engine has each Pack tool's id, schema, and description, and the Pack has the code. When a run needs a Pack tool, `Run::step` returns a `ToolCall` effect and that part of the run waits. The Harness finds the Pack from the first segment of the tool id, calls `Pack::call`, and resumes the run with one answer: the output, the tool's error, or `Dropped`. The effect names the section, the caller, the section entry that made the call, and, for a model's call, the model whose round asked for it.

This split defines the Engine and the Harness. The Engine returns the work it needs, and its caller performs that work. The Engine is therefore independent of any async runtime, and it is a deterministic function of its inputs.

### The model calls offered tools, and Lua calls any Pack tool

The model calls the tools in its section's sets, by wire name. Lua calls any tool in the run's catalog by id with `tools.call`. A Rust Pack can flag a tool `model_only_call`, and a script's call of such a tool raises an error of kind `model_only_tool`. The flag is part of the catalog entry because Lua must be able to name the tool when it offers it. The C contract gives a tool a name, a wire name, a description, a parameter schema, and a flag, which says that the tool changes files, so the other flags of `ToolInfo` are still to be built for a library Pack.

### A Pack tool's failure returns to the model as text and raises in a script

When the model made the call, the Engine converts the failure into the call's result, wrapped as untrusted text, and the loop continues. When a script made the call, the failure raises at the call site with kind `tool`. The model receives the failure as text so that it can read it and recover, because the failure of a Pack tool's backend is outside the prompt author's control.

Some failures raise at `models.loop` itself. A model call to a name outside both sets fails the whole round with `out_of_scope_tool`. A call that the Harness drops, after a stop or a cancel, raises the canceled error.

### The Engine issues one round's Pack tool calls together, and each call that changes files alone

When one model reply asks for several Pack tools, the Engine issues the calls in one step, so they run at the same time. The loop appends the round to the messages as a whole, in the order the model listed the calls. An adhoc tool or a discovery tool runs by itself, between batches. So does a call to a tool whose `ToolInfo` sets `changes_files`: the Engine issues it once every call listed before it has its answer, and issues the calls listed after it once it has its own. The shipped tools that set the flag are Write, StrReplace, and Delete of `std`, and Shell of `bash` (section 9).

The calls run at the same time because they are independent of each other, and the model would otherwise wait for them one at a time. A call that changes files is the exception, because a later call of the same reply can depend on the change. The Host's files have no conflict check, so this order is what keeps two changes of one reply from interleaving: each changing call finishes before the next one starts. The changes of one reply apply one at a time, in the order the model listed them, and a call listed before a change reads the files as they were. A script's `tools.call` is one call and waits for its answer, so the flag changes nothing for it. The loop appends the whole round because model APIs reject a request unless every requested call has a result.

### Each call works under its own file identity

The Engine forks a file identity for each tool call when it issues the call, and ends it when the Harness answers the call or the Engine aborts it. Every file operation of the call runs under that identity, and the file system refuses one that arrives after the end ([Files and the VFS](files-and-the-vfs.md)). The Host's files have no conflict check: the calls that one round runs together, and the calls of two tasks, may read and write one path, and the later write is the content. Only a conflict in a store ends a run, and a store is the prompt's own memory. The owner's reason: "maybe the determinism error should only apply to store operations? and we should otherwise try to behave like regular harnesses, so that two subagents which conflict on file access can just work it out". Of the shipped Packs, `web` and `mcp` ask for no file, and the tools of `std` and `bash` ask through the file functions of the C contract (section 8).

Each call has an identity of its own so that the end of one call stops that call's operations and no other's. A call that the Harness dropped can then change no file afterward.

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

`web/fetch` and `web/search` return untrusted output. `std` marks file content and file names as untrusted. It marks as trusted its fixed stub text, the answers of Write, StrReplace, and Delete, which hold fixed text and counts, and a Grep answer for zero matches that holds only fixed text and counts. `bash/shell` marks the output of each command as untrusted. A discovery listing is untrusted because Packs and their servers write the descriptions. `tasks.receive()` returns the operator's messages as plain text, because a wrapped answer would hide the operator's own words from the script.

### The `mcp` Pack marks its output by caller

The `mcp` Pack reads the caller from its `ToolContext`. For a call the model made, it marks the server's text untrusted. For a script's `tools.call`, it marks the same text trusted, so the script receives the raw string and can decode the listing JSON. The cost is that a script that forwards MCP text into a prompt sends it unwrapped, and that is the script author's decision. Section 11 describes a conflict between this rule and the Engine's rule for tool listings. Only a Rust Pack receives the call's origin: the loader passes a library Pack the tool's name, the arguments, the file functions, the Host's resources, and the call's waker, so the contract needs the origin before `mcp` can move to it (section 10).

## 7. Writing a Pack

### A library Plugin is written against `runvil-plugin-sdk`, and a Rust Plugin against `runvil-plugin`

The crates `runvil-plugin-sdk` and `runvil-plugin` define what a Pack author writes against, and each shipped Plugin uses one of them. `std`, `bashkit`, and `web` are written against `runvil-plugin-sdk`, the safe layer over the C contract (section 8). The author implements the SDK's `Plugin` and `Pack` traits, and `export_plugin!` exports the entry point. Each of these depends on that crate and on libraries from outside Runvil, so it builds independently of the Harness. `plugin-mcp` is written against `runvil-plugin`, which defines the `Pack` trait, `PackFactory`, `PackInfo`, `PackRegistry`, and `ToolContext`, and re-exports the Engine and file-system types an author needs. The `## Invariants` section of that crate's docs is the rule list for its Pack authors. A Host imports the same items from `runvil::plugin`.

These crates are separate APIs, and the design is one. The Rust API has abilities that the contract lacks (section 11), and the move of `plugin-mcp` onto the contract is planned work (section 10). The rest of this section describes the Rust API, and section 8 describes the contract.

### A Rust Plugin is a function named `install`

A Rust Plugin is a function that inserts its Packs into the Host's `PackRegistry`. The Host names the Plugin with `HostBuilder::with_rust_plugin`, as a function that is lent the registry, and that function calls `install` with the registry and whatever else that Plugin takes, as plain arguments. `HostBuilder::build` makes the registry and calls each such function in the order written. `plugin_mcp::install` takes the Host's list of MCP servers, which is the `mcpServers` object of an `mcp.json` file, and the handle of the Host's tokio runtime. For each Pack, the function builds a factory from its arguments and calls `PackRegistry::insert` with the Pack's `PackInfo` and the factory. When setup fails, it calls `insert_unavailable` with a reason.

The Host owns the registry, and Plugins fill it. The owner's rule: "A host can only install plugins". The `install` function is the Rust-only way into the registry. A library Plugin inserts its Packs through the contract's `insert_pack`, and the loader calls `PackRegistry::insert` for it (section 8).

### The Harness makes one Pack instance for each run

The Harness calls `PackFactory::new_pack(id)` once for each Pack in a run. The instance drops when the run ends, and its `Drop` does the cleanup. The factory owns the state that runs share, such as MCP connections.

A Rust Pack is per run because the C contract has the same design for library Plugins, with verbs that create and delete a per-run Pack (section 8).

### `new_pack` returns at once, and `install` builds what the runs share

The Harness calls `new_pack` while it prepares the run, so a factory returns quickly: it clones the parts that `install` built and stores them in a new instance. The `mcp` factory clones its list of servers and the handle of the Host's runtime. `new_pack` runs inside the run's future, so the crate's docs forbid it to block, do I/O, or panic.

### The `Pack` trait has the methods `tools`, `call`, `ready`, and `render`

`tools()` returns the Pack's tool list, which the Harness reads once per run. `call()` performs one tool call. `ready()` resolves when the tool list is complete. `render()` shapes the offered tools for a model.

### `ToolContext` lends each call the tool id, its files, and the call's origin

`cx.tool()` returns the id of the called tool. `cx.files()` returns the call's files, a `ToolFiles`: the call asks for one operation of the `VfsOp` list with `request` and awaits its `VfsOutcome`. A request works under the call's own identity, on paths rooted at `/` of the file system that the Host gave the run. It reaches no store, so a tool sees no file that `store.write` wrote ([Files and the VFS](files-and-the-vfs.md)). The context holds no file access, so a request is the call's one way to the run's files. `cx.origin()` says which section and which caller made the call. The context's fields are borrows that end with the call, so the Rust compiler rejects a Pack that stores the files.

The owner's rule: "The tool can't keep the access. ... it has to give it up, after the call is done". What a Rust Pack needs for longer than one call, its Plugin takes as an argument of `install` and stores in the Pack's factory.

### `Pack::call` follows rules for polling, failure, trust, being dropped, and finishing

1. Return to the poll quickly. A call sends blocking or CPU-heavy work to the Host's runtime, because the Harness polls every effect of a run on the same task. A call awaits a file operation that it asks for through its files, and the Harness performs it between two polls of the call.
2. Report failure as an `Error`. The Harness answers a panicking call as `Dropped` and logs the panic.
3. Mark as untrusted any output that embeds data an attacker can influence.
4. Expect to be dropped. The Harness drops the call's future on a stop, on a cancel, and when the calling task ends.
5. Finish independently of the run's other calls. A call that waits for a sibling queued behind the limit of 10 can stall the run.

### A tool keeps nothing from one call that changes what a later call returns

A tool's result depends on the call's arguments, the run's files, and what the tool reaches outside the run. Whatever one call would leave for a later call goes into the run's files, and the record holds each such change, because a tool changes a file of the run only by a request through the files of its call. A connection or a client that a Pack's factory shares between runs changes no result, so the rule leaves it alone.

The owner's rule: "Tools are not allowed to have internal states, and if they do, they have to be recorded." The reason is replay, which is planned work ([The Harness and the Host API](harness-and-host.md)): a replay feeds each recorded tool result back to the Engine and runs no tool. The owner's words: "On a replay, the, the shell tool is not invoked. The shell tool never executes." A run that went on live after a replay would call a tool that holds none of the state its earlier calls left. The shell of the `bash` Pack follows the rule with a shell for each call: what a script leaves in its shell ends with the call, and the next call finds the run's files alone (section 9).

### A tool asks for a file operation, and the Harness performs and records it

A tool asks for a file operation and waits for the outcome. A Rust Pack's tool asks through `cx.files()` and awaits the outcome. A library Pack's tool asks through a file function of the C contract, and the loader makes the same request for it through the files of the call (section 8). The Harness reports the request to the Engine with `Run::request_file`, and the Engine issues it as a `Vfs` effect that names the call. The Harness performs the operation, records it and its answer, and wakes the call with the outcome ([The Harness and the Host API](harness-and-host.md)). The call stays in progress the whole time, and its one answer is still the tool's result. A request is made when the call first polls it, and it goes out when the Harness takes it, between two polls of the call, so a request that the call drops before then sends nothing. The Harness performs the requests of one call in the order the call made them, and it drops what a call that has ended still asks for. The owner's rule: "every VFS operation that's performed by a tool has to turn into an effect in the engine."

A tool that changes files sets `changes_files` in its `ToolInfo`, and the Engine then runs each of its calls alone, in the order the model listed them (section 5).

The operation itself is synchronous, because the storage underneath is synchronous ([Files and the VFS](files-and-the-vfs.md)): the Harness performs it with one blocking call. The Harness is also independent of any particular async runtime, so each part of the context has to be independent of tokio. A request is a future that the Harness itself wakes, so a tool awaits it on any runtime.

The shipped Rust Pack, `mcp`, asks for no file operation. The shipped tools that touch files are in the library Plugins `std` and `bashkit`, and they ask through the file functions of the contract, so the record holds each of their file operations.

A long file job yields so that the Harness can cancel it. Each file request of a library Pack's tool ends a step of its call, and Grep, in the `std` library, also ends its step after every 32 files or directories, so a stop or a cancel drops a long search between two steps.

### The run's mode policy decides each file change that a tool makes

The Harness performs every file operation of a tool on the file access of its call, and the file system applies the run's mode policy to each one. In Ask mode it refuses each change, and in Plan mode it accepts changes to markdown paths. The check is in the file system because every operation passes through the access, so the check covers every tool, including each tool written later. A `store` call is outside the policy, because the run's store is no part of the file system that tools work in. Workshop's runs use the file system's default policy, `AllowAll`, and tests are the callers of `ModePolicy` ([Files and the VFS](files-and-the-vfs.md)).

### A Rust Pack runs on the Host's async runtime, and a library Plugin owns a runtime

A Rust Pack that needs an async runtime uses the Host's. Its Plugin takes the handle of that runtime as an argument of `install`, and the Pack's factory keeps the handle. The `mcp` Pack spawns the connection to each server, and each call of a server's tool, on it. A library Plugin links a copy of tokio of its own, so the Host's handle is of no use to it, and a Rust object stays on the Host's side of the contract. A library Plugin that needs a runtime therefore owns one: the bashkit Plugin's has one worker thread for its timers, and the web Plugin runs its requests on its own. What the Host hands a library Plugin is its resources, which are JSON values that each of the Plugin's tool calls is lent (section 8).

### A Pack's Lua prelude runs once in each section VM

For a prompt that declares the Pack, the Engine runs the Pack's prelude in each section VM, before `require` and before the author's code. The prelude receives the Pack id through `...` and defines globals, which the Engine then seals. Each Rust Pack that ships leaves its prelude unset. A Pack from the C contract has a name and a description, and a prelude for it is still to be built (section 8).

## 8. The C contract

### The contract is one exported function, and each call names a verb

A dynamic library exports one function, `runvil_plugin`, and each call passes an `RvCall` struct that names a verb. In the contract, verbs 1 and 2 load and unload the Plugin, and a load names the Plugin and inserts its Packs. Verbs 3 and 4 create and delete one run's instance of a Pack, and the instance lists its tools when it is created. Verb 5 runs one step of a tool call, and verb 6 frees the state of a call that the Host abandoned. Numbers 7 and 8 are reserved for later versions.

A single function means one exported symbol, and a later version of the contract adds verbs.

### The contract has its own repository, and each 0.x version is a separate contract

`runvil-plugin-api` contains the C header, the Rust mirror `runvil-plugin-sys`, and a safe SDK, `runvil-plugin-sdk`. While the major version is 0, a Plugin built for 0.6 loads only in a Host that reports 0.6. The Plugin reads the Host's version first in `RV_PLUGIN_LOAD` and answers any other version with `RV_ERR_UNSUPPORTED`. The contract has a repository of its own so that a Plugin depends on that repository only.

### Each callback belongs to one verb and takes a `ctx`

The functions a Plugin may call back are fields of a verb's arguments struct. For a load, `RvLoadArgs` has `ctx`, `refuse`, `describe_plugin`, and `insert_pack`. `RvPackNewArgs` has `refuse` and `add_tool`, and `RvToolCallArgs` has `finish`, the file functions, `answer`, a pointer to the answer to the file request that the step before made, `resources`, the Host's resources as text, and `waker`, the call's waker. The resources are the JSON text of one object with a key for each resource that the Host provides, such as `runvil/web-search`, and `{}` when it provides none. A Host fills a `HostResources` and passes it to the builder of its `Host` with `with_resources`. Each function takes the `ctx` first and answers only while its call runs, on that call's thread. The exception is the `wake` of a call's waker, which takes the waker's own `ctx` and answers on any thread until the Plugin is unloaded. A callback belongs to one verb because each verb has its own lifetime and thread rules.

A tool call is a series of steps. The Host sends `RV_TOOL_CALL` once for each step, a step that returns `RV_PENDING` parks the call, and a step ends the call by calling `finish` once with its result and returning `RV_OK`. The Host sends a parked call its next step after the Plugin wakes the call through the call's waker, or, when the step made a file request, when the answer is ready. A call that waits on something outside its run, such as a reply from the network or a timer, stays parked until a thread of the Plugin wakes it. The call has steps because a step that blocked would stall its run, and the Host can abandon a call only between two steps.

### A file function takes a request, and the call's next step receives the answer

A step reaches the run's files through the file functions of its arguments: `file_stat`, `file_list`, and `file_read` read, and `file_write`, `file_append`, `file_mkdir`, `file_remove`, `file_rename`, and `file_copy` change the files. A file function takes one request and fills nothing. The step returns `RV_PENDING`, the Harness performs the operation, and the call's next step receives the answer, an `RvFileAnswer`: what the request asked for, a status alone for a change, or the error of an operation that failed. The Host sends that step when the answer is ready, and no step before then, even when the Plugin wakes the call. A call has one request outstanding, so a second request in one step is a misuse, and so is a request in a step that calls `finish`. An operation that fails leaves the call running, and the tool decides what its result says. A write or an append takes at most 16 MiB, the constant `RV_MAX_WRITE`, and a tool writes a longer file as one write and then one append for each further part. The contract has no function that asks whether a path exists, because a stat answers that.

The loader makes each request through the files of the call, as a Rust Pack's tool does (section 7). Each file operation of a library Pack's tool, a read as well as a change, is therefore a `Vfs` effect that the Harness performs under the run's policy and records ([Files and the VFS](files-and-the-vfs.md)). The owner's rule: "every VFS operation that's performed by a tool has to turn into an effect in the engine." The answer waits for the next step because the Harness performs each operation as an effect of the run, between two steps of the call. A function that returned its result inside the step would perform the operation there, outside the run's record.

The file functions ask for some of the operations of `VfsOp`. The others have no file function, and `StrReplace`, which replaces a text in a file in one operation, is one of them. A library Pack's tool therefore edits a file as a read and then a write (section 9).

The SDK puts a future over each request. A file method of the SDK's `ToolCall`, such as `read_range` or `write`, returns a `FileFuture`, and a tool awaits it inside the future that its `Pack::call` returned. The first poll of a `FileFuture` queues its request. The step that polled it hands the oldest queued request of the call to the Host and returns `RV_PENDING`, and the next step copies the answer out of the Host's memory before it polls the tool's future again. A tool may wait on several requests at once, and the SDK sends them one with each step, in the order of their first polls.

The contract has an error kind for a conflict between two tool calls, `RV_FILE_ERROR_CONFLICT`, and the Harness sends it to no tool, because the files that a tool reaches have no conflict check (section 11).

A tool whose calls change files sets the flag `RV_TOOL_CHANGES_FILES` when its Pack instance adds it. The loader then sets `changes_files` in the tool's `ToolInfo`, and the Engine runs each call of the tool alone and in order (section 5).

### A library Plugin and a C entry point compiled into the Host receive the same calls

A Host names each Plugin of the contract on the builder of its `Host`: `with_plugin_dir` names every dynamic library in a directory, `with_plugin_file` names the library in one file, and `with_plugin` names a C entry point that is compiled into the Host. The loader has a function for each, `insert_plugin_dir`, `insert_plugin_file`, and `insert_plugin`, and it names the kinds `PluginSource::Library` and `PluginSource::Linked`. Each entry point gets the same calls, so one path loads both kinds. The owner's rule: "those have to be treated exactly the same. In other words ... the entry point has the same shape". For each Pack that a Plugin inserted, the loader puts a factory in the registry. The factory sends `RV_PACK_NEW` for each run, the instance sends one `RV_TOOL_CALL` for each step of a call, and its drop sends `RV_PACK_DELETE`.

`std`, `bashkit`, and `web` are library Plugins, and tests are the only callers of `with_plugin`. The shipped Plugin that is compiled into the Host, `plugin-mcp`, is a third kind. It is a Rust `install` function written against `runvil-plugin` (section 7), which the Host calls itself, so the contract's calls go to the library Plugins alone. The docs of `runvil-plugin` call that third kind a linked Plugin as well. Section 11 describes the gap, and section 10 lists the move of `plugin-mcp` onto the contract.

A Pack from the contract has tools only, and a tool has a name, a wire name, a description, parameters, and the flag that says it changes files. The other `ToolInfo` flags, the prelude, `ready`, `render`, the call's origin, a configuration at `install`, and a Pack inserted as unavailable exist in the Rust API alone. The contract is still to gain the ones that a shipped Pack uses (section 10).

A loaded Plugin is reference counted. The `PluginReport`, each of its Packs in the registry, and each Pack instance of a run hold it, so it unloads after its last Pack instance is deleted. `insert_plugin_dir` records a library that fails to load in the `PluginReport` and continues with the next one.

Loading happens in the Harness, and the Engine receives data: the tool catalog, the roster of the run's Packs, and the prelude text. The reason is that the Engine is pure: tool calls are effects that the Harness performs, and a replay can feed recorded results back to the Engine without the Plugins present.

## 9. The Packs that ship with Runvil

### `std` copies Cursor's file tools

`std` is a library Plugin. Its code is in the `runvil-plugin-std` repository, written against `runvil-plugin-sdk`, and Workshop loads its library from the Plugin directory ([Workshop: how the app is built](workshop-app.md)). The Pack's tools are `std/read`, `std/dir`, `std/grep`, `std/write`, `std/replace`, `std/rm`, and `std/glob`. All of them work except `std/glob`, which answers a fixed text that says the tool is missing from this build. The model sees Cursor's names, such as `Read`, `Grep`, and `StrReplace`. Each definition is Cursor's, with the edits that `crates/plugin-std/schemas/EDITS.md` in that repository lists, except ListDir's, which is the crate's own. The definitions are JSON files.

Write, StrReplace, and Delete change the run's files, and each sets `changes_files`, so the Engine runs each of their calls alone, in the order the model listed them (section 5). Write makes a file, or replaces the whole content of one, with one write, and the Host makes each absent directory on the way to the file. It refuses contents over 16 MiB, the most that one write of the contract takes.

StrReplace reads a file and writes it back with one text in place of another. It loads the file as Read does, with a stat and then one read, so it refuses a directory, a pipe or a device, and a file over 16 MiB as Read does. The file must hold UTF-8 text, and the text to replace must occur exactly once unless the call sets `replace_all`. When the text occurs zero times, or more than once without `replace_all`, the failure states how many times it occurs. The count is of the occurrences that do not overlap, taken from the start of the file, and a call with `replace_all` replaces those. A text that starts at more than one place does not occur once, even where the places overlap, as `ana` starts at two places in `banana`. When the count for such a text is one, a call without `replace_all` is refused with a sentence that says the text starts at more than one place, and that failure states no count. StrReplace refuses an edit that would leave the file over 16 MiB, the most that one write takes, and it refuses before it builds the edited text, because a call with `replace_all` could otherwise ask for far more memory than the file and its arguments hold. The edit is a read and then a write, because the contract has no function that replaces text in one step, so it can lose a change that another task makes to the file between the two ([Files and the VFS](files-and-the-vfs.md), [runvil-plugin-std issue 4](https://github.com/cppalliance/runvil-plugin-std/issues/4)).

Delete removes a file, or a link and not what the link leads to, and it refuses a directory. It asks what the path names before it asks for the removal, because the removal of the contract also takes an empty directory. The two are requests of their own, so a directory with no entries that another task puts at the path between them is removed. A directory with entries is never removed. A removal in the contract that takes a file only would close that gap ([runvil-plugin-std issue 3](https://github.com/cppalliance/runvil-plugin-std/issues/3)). Section 11 lists both windows as gaps.

The Pack copies Cursor's text because the model was trained against that exact text, and a stored copy can be diffed against Cursor's. A stub answers as a success so that calls to the Pack exercise both success and failure from end to end. Walks skip `.git` and list the other dotfiles, because a model often needs to edit `.gitignore` and similar files. A walk leaves out a name that contains a control character and counts it, because a line feed in a name would forge a line of the answer. The shell is in a Pack of its own, `bash`, so a run that wants a shell declares it.

### `bash` runs a shell inside the run, and the shell reads and changes the run's files

`bash` is the Pack of a library Plugin. The bashkit Plugin has a repository of its own, `runvil-plugin-bashkit`, and is written against `runvil-plugin-sdk`. It reports the name `bashkit` and inserts the Pack `runvil/bash`. The Pack has the tool `bash/shell`, and the model sees it as `Shell`, which is Cursor's name for its shell tool. The definition is Cursor's, with the edits that `crates/plugin-bashkit/schemas/EDITS.md` in that repository lists. Workshop pins the Plugin by commit, builds its library through the crate `workshop-plugin-bashkit`, and loads it from the Plugin directory, beside the library of `std` ([Workshop: how the app is built](workshop-app.md)). The built-in chat prompt requires `bash` under `uses:` and offers it.

The shell is a bash interpreter from the `bashkit` crate, and it runs inside the run. Every command is built into bashkit, so the shell works without a child process and without a network. Each call builds a shell of its own, which starts at the root of the run's files and goes when the call ends. The working directory, variables, and functions of one call end with it, so a call finds the run's files as the earlier calls left them and nothing else of an earlier call. A tool keeps nothing from one call that changes what a later call returns (section 7), and a shell for each call is how the shell follows that rule. The owner's reason: "Its the easiest to implement and doesn't expand the API surface." A command ends after 30 seconds, or after the `block_until_ms` of its call when that is less. The answer is the exit code and the output, marked untrusted.

The shell's files are the run's files. Bashkit holds its file system as a value that borrows nothing, and the SDK's `ToolCall` borrows from its call. The Pack's file system, `RunFiles`, therefore holds a channel in place of a call: each file operation of the shell, a read or a change, waits in the channel as a request, and the call that runs the script performs it through the contract's file functions and replies. Every operation goes through the Host: the run's policy applies to it, and the run's record holds it. Redirects, `tee`, `sed -i`, `cp -r`, `mv`, `rm -r`, `mkdir -p`, `touch`, and `mktemp` work. The tool sets `changes_files`, so the Engine runs the Shell calls of one reply one after another, in the order the model listed them (section 5). Each operation that goes to the Host ends a step of the call, so the Host can abandon a script between two operations, and a change that reached the Host before then has happened.

The contract reports what a path names and how long a file is, and it has no function for a mode, a time, a link, or a named pipe. The shell answers those itself and keeps no state to do so: a directory shows mode 755 and every other path 644, `chmod` to another mode fails with the sentence of the constant `FIXED_MODE` in `crates/plugin-bashkit/src/files.rs`, a time that a command sets is not kept, and `ln` and `mkfifo` fail. No file has an execute bit, so a script runs as `bash script.sh`. The shell reads at most 16 MiB of one file and writes at most 16 MiB at a time, and a longer write or append fails with nothing written. On every platform it refuses a write or an append to a path under `/dev/`, with the sentence of the constant `NO_DEVICE` in the same file, because bashkit on Windows sends a redirect to `/dev/stdout` or `/dev/stderr` to its file system ([bashkit issue 2933](https://github.com/everruns/bashkit/issues/2933)), where the write would make a file of that name among the run's files. The Pack builds against `bashkit` 0.21 from crates.io. A redirect to `/dev/null` discards its output all the same, because bashkit answers that one itself. `mktemp` makes its file under `/tmp` among the run's files, and makes that directory when it is absent.

### `web` provides search and fetch together

`web` is the Pack of a library Plugin. The web Plugin has a repository of its own, `runvil-plugin-web`, and is written against `runvil-plugin-sdk`, and Workshop loads its library from the Plugin directory. It inserts the Pack `runvil/web`, with the tools `web/search` and `web/fetch`. The Plugin owns a tokio runtime, because a library links its own copy of tokio, so the Host's runtime is out of its reach. Each search and each fetch runs on that runtime, and its call is parked until the request ends (section 8).

`web/search` sends each search to the Gateway, which has the search vendor's key. The tool reads the `runvil/web-search` entry of the Host's resources that its call is lent: the address of the Gateway's web search route and the Gateway's key. When the entry is absent, because the Host offers none or has no Gateway right now, the search fails with a text that says so, and `web/fetch` works all the same. The Pack has both tools in every run, because the resources are read as each call starts, so only a call finds out whether a search can run, and a research prompt gets both tools together.

A fetch reports a failure of the target, such as a 404 or a timeout, as text the model can act on. The text states the status and the URL, and the Pack drops the error page's body, because the body is untrusted HTML and unhelpful for recovery.

### `web/fetch` connects to public addresses, and checks each address when it connects

The model supplies the URL, so `web/fetch` checks it more than once. Before a request, it refuses a URL that uses plain `http`, contains a user name or password, names a port other than 80 or 443, or has an IP address as its host. When the client connects, a resolver of its own drops each DNS answer in a blocked range, such as a private or loopback address. Both checks run again on each redirect, up to 5 redirects. The client omits cookies and credentials.

The second check runs at connect time because a check that reads only the URL string is defeated by a name that resolves to an internal address and by DNS rebinding.

### A fetch returns a page's text under a header

`web/fetch` takes a `url` and the optional arguments `raw`, which renders the whole page, and `max_chars`, which lowers the limit of 40,000 characters. The tool extracts the main article of an HTML page as markdown, returns JSON, XML, and plain text as written, and refuses every other type and a response that omits `Content-Type`. It reads up to 8 MiB of a body, counted after decompression, so a gzip bomb hits the same cap.

The result starts with the lines `url:`, `truncated:`, and `extraction:`. The first is the final URL after redirects, so a citation names the address that returned the bytes, which can differ from the one the model supplied.

### `mcp` is a Pack with discovery tools for remote servers

The Pack takes the whole `mcpServers` object as its config and connects to each server in the background. It offers `mcp/get_dynamic_tools`, which lists the servers and their tools, and `mcp/call_dynamic_tool`, which calls one. The listing shows each server as `ready`, `loading`, or `error`. The owner's rule: "there should just be 2 static tools that the mcp pack offers".

The client speaks Streamable HTTP through the `rmcp` SDK and returns results as text. An entry for a local server, one started with a `command`, shows as `error`. Local servers are still to be built, because each one needs a launcher program (`npx`, `uvx`), a job object or process group so that the server dies with Workshop, and handling for Windows `.cmd` shims.

### A server entry has the format of Cursor's `mcp.json`

A remote entry has a `url` and optional `headers`. The Pack expands `${env:NAME}` and `${userHome}` in an entry's strings, refuses an entry that names an unset variable, and leaves every other `${...}` form unchanged, as Cursor does. It ignores unknown fields because the format is defined outside this repository.

The Pack refuses the legacy `sse` type and `headers` that set `accept`, `Mcp-Session-Id`, or `Last-Event-Id`. The `rmcp` SDK sets those headers, and the Pack refuses them at install, since `rmcp` would refuse such an entry only on the first request. A refused entry shows as `error` in the listing, with the reason, and the other servers still connect.

### The `mcp` Pack names the keys of `env` and `headers` and omits their values

A value in an entry's `env` or `headers` is often a secret. Each refusal reason, log line, `Debug` output, and tool error of the Pack may name a key and omits the value. The Pack leaves the values out because a reason appears in the notice of a refusal, and a secret inside it would appear there too.

The Pack uses `rmcp`'s own HTTP client, which refuses redirects, so the client sends the configured headers to the configured server only. A case is still open: `rmcp`'s own logs can name a URL that contains an expanded variable.

### `user` marks a prompt that needs a person

`runvil/user` is a Pack of every Host, and the Harness makes it itself. A prompt that needs a person declares `uses: [user]` and reads the person's messages with `tasks.receive()`. When `RunSetup.has_user` is false, the Harness marks the Pack unavailable, and the run's first step refuses a prompt that declares it. The Pack's tool list is empty. A model asks the person a question in its reply: the reply ends `models.loop`, and the prompt then returns to `tasks.receive()`.

### `subagents` reserves a name

`runvil/subagents` is a Pack of every Host, and the Harness makes it itself. It reserves its name for subagent tools that the Engine will provide. It is a Pack so that a prompt can name it under `uses:`. `tasks` is a language feature, because an option to leave the `tasks` table out of a run would be useless.

## 10. Planned work

Each of these items is still to be built, and the ones that begin with the word Undecided wait for a decision of the owner.

- **One Plugin API.** The Rust Plugin `plugin-mcp` written against the C contract and named with `with_plugin`, and the additions to the contract that it needs first. It is an `install` function written against `runvil-plugin` (section 7), Workshop calls it, and `with_plugin` has tests as its only callers. The `mcp` Pack uses things that the C contract lacks. It uses a configuration for its list of servers, the call's origin to mark its output by caller, the `describes_tools` flag on its listing tool, and the Host's runtime handle. It also uses a Pack inserted as unavailable with a reason, and error causes. After the move the `install` function is removed, and a Pack that is compiled into the Host goes through the same contract as a Pack from a library (section 8).
- **Glob in `std`.** A `std/glob` that finds files by a name pattern and returns their paths. The tool answers a fixed text that says it is missing from this build (section 9).
- **Approval by a person.** The step that asks the operator to approve a file change. When the run's policy answers `Ask`, the file system refuses the operation with `VfsError::ApprovalRequired` and a reason. A Rust Pack's tool receives that error, and the loader hands a library Pack's tool a denied file error with the same reason. Workshop's runs use `AllowAll`, so the error is out of reach of a run under Workshop.
- **Ignore files.** Applying `.gitignore`, so that Grep skips a folder such as `target/` and spends its budget of 100,000 files elsewhere. Walks skip `.git` and names that contain a control character.
- **Shell state that lasts across calls.** A shell that saves its state to a file of the run and loads it at its next call. Each `bash/shell` call builds a new shell, so the working directory, variables, and functions of one call end with it (section 9).
- **Undecided: a thread of its own for bash.** Bash on a thread of the Plugin's own, with a wake function in the contract. Bash runs inside the steps of its call: a heavy script holds its step, and the Stop button with it, until the script ends or reaches one of bashkit's limits, and while a script sleeps the Host steps the call with nothing to do. The owner has yet to decide on it.
- **Undecided: a removal that takes a file only.** A removal in the contract that refuses a directory, so that Delete needs no stat before it ([runvil-plugin-std issue 3](https://github.com/cppalliance/runvil-plugin-std/issues/3)). Delete asks what the path names and then asks for the removal, so a directory with no entries that another task puts at the path between the two is removed (section 11). The owner has yet to decide on it.
- **Undecided: an edit that keeps another task's change.** A way in the contract for a tool to edit a file without losing a change that another task makes to it, such as an edit as one operation or a write that fails when the file changed since the read ([runvil-plugin-std issue 4](https://github.com/cppalliance/runvil-plugin-std/issues/4)). StrReplace reads a file and then writes it back, so a change that another task makes to the file between the two is lost (section 11). The owner has yet to decide on it.
- **Subagent tools.** The tools of `runvil/subagents`, and the wrapping of text that one model writes for another. The Pack only reserves its name (section 9).
- **MCP in the Dynamic Set.** The `mcp` Pack rendering its servers' tools into the Dynamic Set, which retires its own tools. Local servers and OAuth are still to be built as well. The Pack offers tools of its own, for remote servers.
- **Filters and labels.** An offer narrowed by a hint such as read-only, and labels on `ToolInfo`, modeled on MCP's, for a future Ask mode. An offer names each tool or Pack, and `ToolInfo` has the flags `structured_output`, `survives_stop`, `describes_tools`, `model_only_call`, and `changes_files`.
- **Partial rounds.** Saving the finished results when a stop cuts a round short. The loop discards the whole round.
- **Fetch settings.** A way for a Host to change the fetch limits. The fetch policy has a setter for each limit, and the Plugin takes no configuration, so each Host runs the defaults.

## 11. Known gaps

The gaps are a conflict inside the code, the APIs that install Packs, the tools of `std` that change a file in more than one request, an error kind that only tests send, a place where the language guide differs from the code, and a flag and a prelude that only tests set.

### The trust rules overlap, and the model receives MCP server text unwrapped

The trust of a tool listing that a script reads follows rules in the `mcp` Pack and in the Engine. The `mcp` Pack marks its output trusted for a script's call (section 6). The Engine has a rule for tools flagged `describes_tools`: a script receives the tool's untrusted listing unwrapped, and the Engine then wraps the result of an adhoc tool that fetched the listing. Under that rule an adhoc wrapper of a listing tool produces the same envelope as a direct call by the model.

`mcp/get_dynamic_tools` is a shipped tool with the flag. The Pack marks a script's listing trusted, so the Engine treats it as trusted text and skips its listing rule. An adhoc tool whose handler returns `tools.call('mcp/get_dynamic_tools')` therefore returns the servers' text to the model unwrapped and reported as trusted.

The evidence is a reading of `dispatch_body` in `runvil-internal/lua/src/dispatch.rs`, of `marked` and `Mcp::call` in `plugin-mcp/src/dynamic.rs`, and of the tests `a_script_listing_is_trusted_json_equal_to_the_models` and `a_failed_listing_call_and_a_trusted_listing_mark_nothing`. The Engine's tests of the wrapper rule use a stand-in tool that returns untrusted text. A test with the real `mcp` Pack would confirm the behavior.

### The C contract and the Rust API both install Packs, and each has abilities that the other lacks

A Pack enters a Host's registry in one of these ways. A Rust `install` function, written against `runvil-plugin`, inserts it: `plugin-mcp` works this way, and Workshop calls the function. A C entry point that is compiled into the Host inserts it, when the Host names the entry point with `with_plugin`. A dynamic library that exports `runvil_plugin` inserts it, when the Host names the library with `with_plugin_dir` or `with_plugin_file`: `std`, `bashkit`, and `web` work this way. The second and third ways are one API, the C contract, and one private function of the loader loads both. The first way is a second API, built from Rust types of the Engine and the Harness.

The `mcp` Pack uses things that the C contract lacks (section 10). A Rust Pack's tool may also ask for any operation of `VfsOp`, a replacement of text in one operation among them, where the file functions of the contract ask for only some of them (section 8). The contract has what `install` lacks: a Plugin reports a name and a version, and the `PluginReport` lists them. A call to `install` returns only a result, so Workshop's listing of the Plugins it loaded names `std`, `bashkit`, and `web` and omits `mcp`. `with_plugin` has tests as its only callers. More abilities of the Rust API have tests as their only users: the flags `structured_output` and `model_only_call`, and overrides of `Pack::ready` and `Pack::render`.

The word "linked" has two meanings in the code. The docs of `runvil-plugin` use "linked Plugin" for the `install` function of a Plugin crate. The loader's `PluginSource::Linked` and `insert_plugin` mean a C entry point that is compiled into the Host. Section 8 uses the second meaning.

The design is one API, and the move is planned work (section 10). A Pack written against the Rust API can use each ability that API gains, and a Pack from the contract waits for the contract to gain it, so the two sets of Packs drift apart with each addition.

The evidence is a reading of the `install` function in `plugin-mcp/src/lib.rs`, of `with_plugins` in `workshop/server/src/agents.rs`, of `HostBuilder` in `harness-internal/runner/src/host.rs`, and of every struct of `runvil-plugin-sys`, and a search for the callers of `with_plugin`, which finds the tests of `harness-test-plugin` alone.

### StrReplace and Delete each take more than one request, and another task can change the file between them

The Host's files have no conflict check (section 5), and each file request of a tool is an operation of its own. StrReplace and Delete of `std` decide on one request and act on a later one.

1. StrReplace. The tool reads the file, builds the edited text, and writes the whole file back. A change that another task makes to the file between the read and the write is lost, because the write holds the file as the read found it. The contract has no function that replaces text in one operation (section 8).
2. Delete. The tool asks what the path names, refuses a directory, and then asks for the removal, which also takes a directory with no entries. A directory with no entries that another task puts at the path between the two requests is removed. A directory with entries stays, because the removal refuses it.

The calls of one model reply are safe from both, because the Engine runs the changing calls of a reply one at a time (section 5). The window is open between the calls of two tasks. Each fix is a change to the contract, and the owner has yet to decide on either (section 10).

The evidence is a reading of `crates/plugin-std/src/replace.rs` and `crates/plugin-std/src/rm.rs` in `runvil-plugin-std`, whose module docs state both windows, and of [issue 4](https://github.com/cppalliance/runvil-plugin-std/issues/4) and [issue 3](https://github.com/cppalliance/runvil-plugin-std/issues/3) of that repository. The tests of `std` in `crates/plugin-std/tests/it/changes.rs` step through each call, and they leave the file alone between the read and the write of a StrReplace call and between the stat and the removal of a Delete call. A test that changes the file there would confirm the behavior.

### The contract's error kind for a conflict has tests as its only senders

The contract has the file error kind `RV_FILE_ERROR_CONFLICT`, for a path that another tool call of the run changed or is changing. The SDK hands it to a tool as `FileErrorKind::Conflict`, and `std` has text for it: a sentence that tells the model to call the tool again in its next message, and a count in Grep's line of skipped entries. The Harness sends the kind to no tool, because the files that a tool reaches have no conflict check (section 5). In the loader, `describe` gives the kind `RV_FILE_ERROR_OTHER` to each `VfsError` that the contract has no kind for, and a `VfsError::Conflict` from a Host's own backend is one of them. In `std`, the unit tests of the refusal sentences build the error themselves.

The evidence is a reading of `describe` in `harness-internal/plugin-loader/src/file.rs`, of its test `each_error_the_contract_has_no_kind_for_reads_as_other_and_a_text_a_model_can_read`, and of `refusal` in `crates/plugin-std/src/refusal.rs` in `runvil-plugin-std`.

### The language guide differs from the code about the refusal of a missing Pack

Chapters 4, 12, and 13 show the refusal of a missing Pack at the run's first step, as `missing required Pack: web (at file:line)`. Under Workshop, `Host::parse` refuses earlier, as `web is unavailable: runvil/web is not installed`. The guide's text is correct for a caller that steps the Engine itself.

### `survives_stop` and the Pack prelude have tests as their only users

A Rust Pack can flag a tool with `survives_stop`, and it can set a Lua prelude. The Engine and the Harness act on both, and each shipped Rust Pack leaves both unset. A Pack from the C contract has neither (section 8).

1. `survives_stop`. `ToolInfo::survives_stop` in `runvil-internal/types/src/tools/info.rs` sets the flag, and the Engine copies it to each `ToolCall` effect. On a stop, the Harness drops every tool call except a flagged one, through `Reach::AllButSurvivors` in `harness-internal/runner/src/effect_loop.rs`. `ToolInfo::new` sets the flag to `false`, each tool of `mcp` leaves it at `false`, the loader builds each tool of `std`, `bash`, and `web` with `ToolInfo::new`, the `changes_files` flag, and a wire name alone, and tests are the only callers of the setter.
2. The Pack prelude. `PackInfo::prelude` in `runvil-plugin/src/pack.rs` sets the prelude. `RunPacks::preludes` in `harness-internal/runner/src/packs.rs` collects the preludes of the Packs that the run's prompts declare, and `install_preludes` in `runvil-internal/lua/src/prelude.rs` runs a prompt's own in each of its section VMs. `PackInfo::new` sets the prelude to `None`, the `PackInfo` values of `mcp`, `user`, and `subagents` leave it at `None`, the loader builds the `PackInfo` of `std`, of `bash`, and of `web` from a name and a description, and tests are the only callers of the setter.

## 12. Questions for the owner

Each question asks for the reason behind a part of the design.

**Tool calls**

1. Why does a handler that returns a table fail the call?
2. Why does a call to an unknown name fail the whole round, while a call to a Dynamic Set name returns text?
3. Why do adhoc tools run one at a time?
4. Why do events fire in the order of the answers while the messages are in call order?
5. Why is a script's `tools.call` a single call, outside the batch path?
6. Why is the default 24 rounds?
7. What is `survives_stop` for, when the shipped Packs leave it unset?

**Packs**

8. Why does the registry leave a Plugin's Packs in place when its `install` fails partway?
9. Why does `Host::parse` check `uses:` when the run's first step checks it too?
10. Why is the need for a person expressed as a Pack?
11. Why is every `uses:` entry required?
12. Why do `web`, `std`, and `bash` wrap output for scripts while `mcp` returns it raw?

**The C contract**

13. Why does the header reserve verbs 7 and 8 and status codes 1 to 3, when it names no use for them?
14. What does a new Pack that is compiled into the Host use until the contract has what `mcp` needs: a Rust `install` function, or a C entry point named with `with_plugin`?
15. What happens to the `Pack` trait and `ToolContext` of `runvil-plugin` once `plugin-mcp` is on the contract?

**The shipped Packs**

16. Why does the shell of `bash` refuse a write and an append under `/dev/` and send a copy, a move, and a new directory under `/dev/` to the Host?
17. Why does `mktemp` make its file under `/tmp` among the run's files, where a Host that mounts a real directory at the root gains a `tmp` directory in it?
18. Why does an argument error of `std` name its tool as `std_Read`, when the model calls the tool `Read` and Lua calls it `std/read`?

## 13. Method and limits

- **Checks.** The facts on this page were read in the code. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.

*2026-11-10 16:46 - Claude Fable 5.2*
