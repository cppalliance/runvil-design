# The Harness and the Host API: design decisions and their reasons

- Scope: the `runvil` crate, `Host`, `RunSetup`, one Harness per run, the effect loop, inference brokers, Host services, the recorder and run events, the run inbox from the Host's side, stop and cancel
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the walk-through's sentence on filling the `PackRegistry` describes `runvil` at commit `b0693699d`, and the Paths bullet names the repositories as of `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 10)
- Update: the text on how a Host installs Plugins, on `runvil::plugin`, on `ready`, on `HostServices`, and two planned items describe `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, and `runvil-plugin-api` at `03a9fa3` (section 10)
- Update: the text on file operations as effects, on the files of a tool call, on the turn that the effect loop gives its executor, on the record of a tool's file operation, on a stop and a cancel of a tool call that asks for files, the sentence on `ToolFiles` and the count of re-exports in section 1, two planned items, and two questions describe `runvil-harness` at commit `ee199e3a` and `runvil-plugin-api` at `52a4905` (section 10)
- Update: the two entries on the inference broker and on streaming in section 4 describe `runvil-harness` at commit `7a106571` (section 10)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil-workshop`

## Summary

A Host is an application that runs prompts, and it depends on one crate, `runvil`. At startup it builds one `Host` struct: its installed Packs, a prompt finder, and a timer. For each run it fills a `RunSetup` with a recorder, an inference broker, a file system, and the run's settings, and creates one `Harness`. `Harness::run` is one future. It steps the Engine, performs each effect, writes every event, effect, and answer to the recorder, and returns a `RunReport`. A `RunControl` sends messages to the run, stops the work in flight, or cancels the run. Sections 1 to 6 describe these parts.

A check of this page against the code found five gaps and left fourteen questions about reasons for the owner. Section 8 lists the gaps, and section 9 lists the questions. The largest gap is in the language guide: the table in chapter 16 says that the Harness changes the response cap and the receive timeout, and `GatewayBroker::new` always uses the defaults.

Section 7 lists the planned work. Its largest item is replay: the recorder receives the seed, the start time, every answer, every message, and every file request of a tool call, and the code that passes them to a second run is still to be built.

## From a Host's setup to a recorded result

At startup a Host creates a `PackRegistry` and fills it. `insert_plugin_dir` loads each library Plugin of a directory, `insert_plugin` loads a Plugin that is compiled into the Host, and both kinds follow the C contract. Two Plugins, `plugin-web` and `plugin-mcp`, enter through a second API: each is a Rust function named `install`, which the Host calls with the registry, a configuration value, and the Host's services. The move of the two onto the C contract is planned work (section 7). The Host then passes the registry to `Packs::new`. It builds one `Host` with three fields: `packs`, a `PromptFinder` in `prompts`, and a `Timer` in `timer`. The `run-prompt` example in the `runvil` crate runs one prompt this way, with only the built-in Packs.

For each run the Host calls `Host::parse` with the prompt's name. `Host::parse` finds the source, parses it and every prompt that it lists for `execute`, and checks the `uses:` list of each against the installed Packs. The Host then calls `RunSetup::new` with a recorder and a broker, sets the fields this run needs, and calls `Harness::new`. It takes a `RunControl` from `Harness::control` and awaits `Harness::run`.

`Harness::run` first calls the recorder's `begin_run` with a `RunMeta`: a seed drawn from the operating system and the start time from the Host's timer. It asks the broker for its model list and picks the run's model. It creates the run's Pack instances and builds an Engine `Run` with the arguments in `RunSetup.args`.

The effect loop then calls `Run::step`. It appends the step's events to the recorder, then appends each effect and starts it. A `Chat` effect calls `InferenceBroker::chat`, a `ToolCall` or `Render` effect calls the run's Packs, a `Timer` effect calls `Timer::sleep`, and a `Vfs` effect, which is one file operation, runs at once on the same thread. When an answer is ready, the loop appends it and calls `Run::resume`. The same wait also ends on a cancel, a stop, a sent message, or a file request of a tool call in flight. The loop reports such a request with `Run::request_file`, the next step returns it as a `Vfs` effect that names the call, and the loop performs the operation on the file system of `RunSetup.vfs` and hands the outcome to the call.

When `Run::step` returns `Step::Done`, the loop calls the recorder's `end_run` with the result. `Harness::run` returns a `RunReport` with three fields: the run's id in `run_id`, the same result in `result`, and the messages that the run left unread in `unread`.

## 1. The three layers and the `runvil` crate

### The Engine emits effects, the Harness performs them, and the Host decides policy

The root `AGENTS.md` defines the terms. The Engine, the `runvil-*` crates, parses a prompt, steps a run, and emits an effect each time the run needs a model reply, a Pack's render of the tools a section offers it, a Pack tool's result, a timer, or a file operation. The Harness, the `runvil` crate and the `harness-*` crates, steps the Engine, performs every effect, and records every run. A Host is an application such as Workshop, and it "makes every policy decision".

The Harness owns the effect loop because one ordered record of a run exists only if the Harness sees every effect and every answer. Replay and resume, which section 7 lists as planned work, depend on the same condition.

### A prompt names roles and Pack ids, and the Host binds both

A prompt file names a model by a role, such as `strong`, and a set of tools by a Pack id under `uses:`. The Host supplies the rest: its selected model fills the roles, its `PackRegistry` supplies the Packs, and `RunSetup.vfs` is the file system that the run's tool calls work in.

### A Host depends on one crate, `runvil`

`runvil` re-exports three groups of items: the Harness, the Engine items a Host reads and builds, and the Rust Plugin API. `Harness`, `Host`, `RunSetup`, `RunReport`, and `RunControl` are at the crate root, and the rest of the Harness is in `runvil::harness`, which includes the loader of the C contract: `insert_plugin`, `insert_plugin_dir`, `Plugin`, `PluginReport`, and `PluginSource`. The same module has three stock prompt finders: `FindTextPrompt`, `FindFilePrompt`, and `PromptSet` (section 2). `runvil::plugin` has the Rust Plugin API of the crate `runvil-plugin`, such as `PackRegistry`, `Pack`, and `HostServices`. `ToolFiles` is in that module too: a tool call asks for each file operation through it, as a `VfsOp` of `runvil::vfs` that resolves with a `VfsOutcome` (section 3). The types of the C contract are in `runvil-plugin-sys`, outside `runvil`. The crate has 124 re-exports, each at one path. `cargo xtask api` checks that every type from this repository that a `runvil` signature names is among them.

The owner's goal: "I want the outside world to only see one runvil crate, not runvil-*."

### `runvil` leaves out the Engine's stepping API

`runvil` omits 17 Engine names, the items that step a run and perform its effects, among them `Run`, `Step`, `RunParams`, `Effect`, `EffectAnswer`, and the `cancel` module. Code that steps a run itself imports them from `runvil-engine`, as the Harness does, and a Host cancels through `RunControl`. `runvil` re-exports every other Engine name, including names that the existing Hosts leave unused.

### `harness-gateway-client` is the Harness family's second public crate

`harness-gateway-client` contains what a Host needs to call the Gateway: `GatewayBroker`, which implements `InferenceBroker`; `GatewayChat`, the HTTP client it uses; `fetch_model_catalog`; and `GatewaySearch`, the search provider for the `web` Pack. It is the one Harness crate besides `runvil` that an outside crate may depend on.

The client is a crate of its own for two reasons. Inference is a Host service and `runvil` excludes tokio, so `runvil` leaves HTTP to the Host. Every Host still needs a standard Gateway client. [The Gateway](gateway.md) covers the routes it calls.

### The Harness's async code uses `futures-util`, and any executor can poll a run

`harness-runner`, the crate with the effect loop, uses `futures-util` for its async work, and a structural check fails when `runvil` or a crate under `crates/harness-internal` lists `tokio` under `[dependencies]`. A Host therefore awaits `Harness::run` on whichever executor it has.

A Host's own parts may still need a runtime: the docs of `GatewayBroker` state that its futures need tokio.

## 2. `Host`, `RunSetup`, and one Harness per run

### A Host builds one `Host` at startup and one `RunSetup` for each run

`Host` is a struct with three public fields: `packs`, `prompts`, and `timer`. A Host builds one and shares it behind an `Arc`. `RunSetup` is a struct with nine public fields that the Host fills for each run. The two structs sort every input by lifetime: `Host` contains what every run of one Host shares, and `RunSetup` contains what one run gets on top of the Host.

### `Harness::new` takes the `Host`, a parsed `Prompt`, and the `RunSetup`, and `run` consumes the Harness

`Harness::new(host, prompt, setup)` creates the Harness for one run. `Harness::control()` returns a `RunControl`. `Harness::run()` takes the Harness by value and returns a `Send` future, so the Host takes the control first. A chat conversation is Workshop's own code around one run; see [Workshop: how the app is built](workshop-app.md).

### `Host::parse` parses the prompt before a run exists, and a parse failure returns to the caller unrecorded

`Host::parse(name)` calls `Prompt::parse` with the Host's `PromptFinder`, which parses the named prompt and every prompt that the `prompts:` lists reach from it ([The Engine](engine.md)). It then checks the `uses:` list of each of those prompts against the installed Packs. It returns the parse error unchanged, or a `RequirementsUnmet` error that names each declared Pack that the Host lacks or that a Plugin inserted as unavailable. The line of a Pack that only a linked prompt declares starts with that prompt's name, as in `- review.md: web is unavailable: runvil/web is not installed`. Only a Rust `install` inserts a Pack as unavailable. The loader drops every Pack of a contract Plugin whose load fails, so `Host::parse` reports each of them with the text `is not installed`, and the reason is in the `PluginReport` that `insert_plugin_dir` returned. The recorder's first call for a run is `begin_run`, inside `Harness::run`.

### A prompt that lists other prompts needs a finder that serves each listed name

The Harness calls the Host's finder only inside `Host::parse`. One parse finds the named prompt and then each prompt that the `prompts:` lists reach, so the finder has to serve every one of those names. A listed name that the finder lacks fails the parse, and the failure is placed on the `prompts:` line of the prompt that lists the name.

`runvil::harness` has three stock finders. `FindTextPrompt` serves one prompt from text, and `FindFilePrompt` serves one from a file. Each fails a find for any other name with the code `not_found` and the message `no prompt named {name}`, so the two fit a prompt that lists no prompt, or only itself. `PromptSet` serves the prompts that a Host inserts as text, as files, and as directories, each file of a directory under its file name, such as `greet.md`. A Host whose prompts execute each other therefore gives its `Host` a `PromptSet`, or a finder of its own that serves each name.

A run that has started holds its parsed prompt and every prompt that one links, and an `execute` runs one of those. `PromptSet::rescan` lists the set's directories again, which changes what later parses find and changes no run in progress. The test `a_run_executes_the_prompt_its_parse_linked_though_the_set_has_lost_that_prompts_file` in `harness-internal/runner/tests/it/harness-execute.rs` removes a listed prompt's file after the parse and rescans: the run still executes the prompt, and the next `Host::parse` fails with `no prompt named greet.md`.

Every find happens in `Host::parse` because a stock finder reads a file with a blocking `std::fs` call, and a parse runs before the run's future exists. The rule holds for a prompt that executes others because the parse has read them all.

### `RunSetup::new` takes the recorder and the broker, and every other field has a default

`RunSetup::new(recorder, broker)` sets the two required fields. The other seven start at defaults: an empty in-memory file system in `vfs`, empty values in `selected_model` and `workspace_roots`, the empty object `{}` in `args`, `RunLimits::default()` in `limits`, 10 in `max_tool_calls_in_flight`, and `false` in `has_user`. Workshop leaves `limits` and `max_tool_calls_in_flight` at their defaults, among them 24 loop rounds and 8 concurrent tasks.

The default of `args` is `{}`, which is what a caller with nothing to pass sends. A Host that leaves the field alone therefore starts a valid run of every prompt that requires no argument.

The broker is a required argument because every run needs inference. A Host that leaves it out fails to compile. With an optional broker, the Harness would refuse each run of such a Host as it prepares the run.

### The Harness reads each `RunSetup` field once, and the run's tool calls work in the one file system that the setup holds

The Harness reads each field once per run. This matters most for `vfs`, the file system that the run's tool calls work in: the Harness performs every file request of the run's tool calls on that one file system. The run's store is no part of it. The Engine makes each run an empty store of its own, which no setup fills and no report returns. The `Policy` of that file system, such as a `ModePolicy` in its Ask, Plan, or Agent mode, sees the file operations of tool calls. So does the operation sink that `VfsRefBuilder::on_op` installs. Neither sees a store operation. A prompt reads `selected_model` and the first of `workspace_roots` through `ui()`.

The Host chooses where the files of a run's tool calls are stored, because the Engine only executes the prompt, and each file request of a tool call works on the file system that the Engine's caller supplies. No Host policy applies to the store, because a store is no part of that file system: a policy decides what a run's tools may do to the Host's files, and the store is the run's own memory. [Files and the VFS](files-and-the-vfs.md) covers the file system.

### The Harness passes `args` to the Engine as the Host set it, and the run's result is the report's `result`

`RunSetup.args` is the text of one JSON object, and the Harness passes it to `Run::new` unread. The Engine parses and checks the text at the run's first step ([The prompt language](prompt-language.md)), so text that fails the check ends the run with the code `args_invalid`, and the Host reads that error in `RunReport.result`. When the run ends with no error, `RunReport.result` is its final text: the value that the prompt returned, or the text `done` when no section returned a value. A prompt that built a file returns the file's text, and the Harness reads no file after a run.

The Harness leaves the check to the Engine so that one piece of code decides what arguments a prompt takes, whichever caller starts the run.

A run has one way in, its arguments, and one way out, its result. The owner's rule: "If a prompt wants to return a file, it should return it as the string of its return value."

### `Harness::run` returns a `RunReport` for every run, and its one error is `RecorderFailed`

`RunReport` has three fields: `run_id`, `result`, and `unread`. `result` is the final text or the `Error` that ended the run, and a cancel is the error whose code is `canceled`. The record begins before the model lookup, so a failed lookup and an early cancel each produce a report. `Harness::run` returns `Err` in one case, a recorder that refused a write, with the code `RecorderFailed`.

The Harness delivers a failed prompt as a result that says why the prompt failed, the same way it delivers a text that the prompt produced through inference.

### The Harness draws each run's seed from the operating system

`begin` in `prepare.rs` draws 32 random bytes with `rand::random()`, builds a `RunSeed`, and passes it to the recorder in `RunMeta` and to the Engine in `RunParams`. The Engine derives the nonce of the untrusted-output envelope from the seed, and a structural check forbids random sources in every Engine crate, so the Harness makes the draw. A seed read from a clock would be guessable, and a guessable seed makes a guessable nonce. [Tools and Packs](tools-and-packs.md) describes the envelope.

### The Harness waits for each Pack's `ready`, and a run that fails its start checks ends at the first step

Before the first step the Harness creates the run's Pack instances and awaits every instance's `ready` together, until all resolve or the Host cancels the run. Each Pack bounds its own `ready`, because a timeout in the Harness would be a policy that belongs to each Pack. `ready` is a method of the Rust trait `Pack`, and its default resolves at once. The C contract omits a readiness call, so a Pack of a contract Plugin is ready when `RV_PACK_NEW` returns, and the Packs that ship keep the default.

The first `Run::step` checks the run's requirements. For a shortfall, such as a Pack whose factory refused this run, it returns `Step::Done` with a `RequirementsUnmet` error that lists every gap. When the cancel flag is set, the step ends the run with the canceled error. A cancel that ends the wait early leaves a Pack that is still starting with zero tools, and the check would otherwise end a canceled run as refused. [The Engine](engine.md) lists the checks.

## 3. The effect loop

### `Harness::run` is one future that polls every effect

The loop in `effect_loop.rs` puts the future of each effect in flight into one `FuturesUnordered` and polls them inside the future that `Harness::run` returned, on whichever thread polls that future.

The design sets a rule for every performer, which is a broker, a Pack, or a timer: return to the poll quickly, and send blocking work to the Host's own runtime. A performer that blocks stalls every other effect of the run, and the run's stop and cancel with them. The rule covers a performer's own work, such as a network request or a long computation. A file operation is synchronous by design, and the run waits for each one ([Files and the VFS](files-and-the-vfs.md)).

### The Harness answers each `Vfs` effect inline, on the thread that polls the run

A `Vfs` effect is one file operation, a `VfsOp`. Its `asker` says who asked for it, the prompt's Lua or a tool call, and its `access` is what the loop performs the operation on ([The Engine](engine.md)). The loop appends the effect's record, performs the operation, appends the answer, and resumes the run before it waits again. When one step issues several `Vfs` effects, the loop answers all of them and then steps once. The operations run one at a time, in the order the Engine issued them, whichever chain or tool call asked for each.

The prompt's Lua asks with a `store` call, and the operation works on a store of the run. A run has the store that it starts with, and one more for each `execute` ([Files and the VFS](files-and-the-vfs.md)). The asker is `VfsAsker::Lua`, which names the store by the id of its chain, and the access is a view of that store under the calling chain's identity. The loop performs the operation with `perform_vfs_op` on that view and answers with `EffectAnswer::Vfs`, the whole outcome. The Harness reaches a run's store only to perform the one operation of a `Vfs` effect, through that effect's view. It puts nothing in the store before a run and reads nothing from it after.

A tool call asks through the files of its `ToolContext` ([Tools and Packs](tools-and-packs.md)). `ToolContext::files` returns a `ToolFiles`, a trait of the crate `runvil-plugin` whose one method, `request`, takes a `VfsOp` and returns a future of its outcome. The loop gives each tool call a `CallDesk` as its files, and a request joins the desk at the first poll of its future. The loop reads the requests of the calls in flight beside their answers and reports each one to the run with `Run::request_file`. The run issues the request at its next step as a `Vfs` effect whose asker is `VfsAsker::Tool` with the id of the call, and the loop performs it on the file system of `RunSetup.vfs`, under the call's own identity and the Host's policy. The loop then resumes the run with `EffectAnswer::VfsReceipt`, a receipt of the outcome, and hands the whole outcome to the call, which stays in flight throughout. A call that has landed or was dropped gets no further operation: the loop drops a request it still makes, and answers `Dropped`, unperformed, a `Vfs` effect that the run issued for it. A panic of the Host's policy or storage inside an operation reaches the tool as a failed operation, a `VfsError::Backend` with the message `the file operation panicked`, and the run goes on.

A tool of a library Plugin asks through the same files. The loader, the crate `harness-plugin-loader`, follows version 0.5 of the C contract, in which the `RvToolCallArgs` of a step has nine file functions: `file_stat`, `file_list`, and `file_read` read, and `file_write`, `file_append`, `file_mkdir`, `file_remove`, `file_rename`, and `file_copy` change files. A file function takes the one request of its step and fills nothing, and a step that made a request returns `RV_PENDING`. The loader asks the `ToolFiles` of the call for the operation, and the call's next step receives the answer as an `RvFileAnswer`. The loader reaches the run's files in no other way, so the Harness performs and records every file operation of a library tool, and the Host's policy applies to each one.

While the recorder answers at once, the loop goes from one file request to the next without a return to the executor that polls the run. So once in every 32 file requests of the run's tool calls, the value of `REQUESTS_PER_TURN` in `tool_files.rs`, the run's future wakes itself and returns pending once, and the loop reports the request after that return. Which task the executor polls next is the executor's choice, so the return is one more chance for the Host's other tasks, one that stops or cancels the run among them, and no promise. An executor that takes its tasks in turn runs them then, so a tool call that keeps asking for files gives up the thread at each such return. A file operation is one blocking call, so the loop returns between operations and never in the middle of one, however long a backend takes. Only the file requests of tool calls count: a Lua loop of `store` calls, and a Lua loop of calls to a tool whose future is ready at its first poll, run on without such a return while the recorder answers at once.

The loop performs the operation itself because file operations are synchronous by design, so the call has nothing for the loop to wait on. The owner's reason: "all file I/O synchronizes the, the harness, and that's in line with our design. Our whole design of Runvil is a single-threaded. It's cooperative asynchrony." A store is memory. The owner's rule: "Every prompt starts with a, a memory store that's completely empty. And that's the only type of store there is. And it doesn't map to any file system path in the virtual file system." With no way in from outside, a run's result depends only on its prompt, its arguments, and its answers.

### The loop answers `Dropped` for a performer that panics

The loop catches a panic from a broker, a Pack, or a timer, logs it with the effect id, and answers the effect `Dropped`. The Lua call that waited raises the canceled error, which a `pcall` can catch, and an uncaught one ends the run as canceled.

### The Harness sends each tool call to the Pack named by the id's first segment, and at most 10 run at once

`RunPacks`, the run's snapshot of its Pack instances, performs every `ToolCall` and `Render` effect. It finds the Pack from the first segment of the tool id and calls `Pack::call`. The lookup skips the run's tool catalog, because a model can call a model-only tool, which a Pack's `render` adds outside the catalog.

At most `RunSetup.max_tool_calls_in_flight` tool calls run at once, 10 by default, and the loop starts waiting calls lowest effect id first. The count is one for the whole run, so the tool calls of a prompt that an `execute` runs are in it with the calls of the run's own prompt. Chat, render, and timer effects start at once and are outside the count. [Tools and Packs](tools-and-packs.md) covers the cap's reasons and `Pack::call`.

### A pending run whose effects are all answered ends with an `Internal` error, unless its main chain waits for a message

A `Step::Pending` with an empty effect list, while every earlier effect is answered, shows that the loop lost an effect. The loop ends the run with an `Internal` error: "the effect loop has nothing to await for a pending run". The Engine ends its own stalls with the code `stalled` or `internal`; see [The Engine](engine.md).

When the main chain, the run's root task, waits on its inbox of sent messages, `Step::Pending` sets `waiting_for_input`, and the loop waits for a `RunControl::send` or a cancel. The same applies to a run whose `has_user` is `false`, since a Host may still call `RunControl::send` from code.

## 4. Inference, the clock, and Host services

### The Host supplies model inference as an `InferenceBroker`

`InferenceBroker` is a trait with the methods `models()`, `supports_streaming()`, and `chat(binding, messages, tools, options, round, on_piece)`. `models()` lists the models the Host offers. `supports_streaming()` says whether the broker hands the pieces of a reply to `on_piece` as the reply forms. `chat` runs one model round, takes an optional callback for the reply's pieces, and returns the finished reply, and the loop calls it for each `Chat` effect. The broker decides where it sends each round, so a Gateway change applies to the next round.

The Host supplies inference so that the Host's broker owns the Gateway's URL, its key, and its model catalog, and `RunSetup` needs only the broker. [Model calls and conversations](models-and-conversations.md) covers the request a round sends.

### The Harness picks the run's model once, and that model fills all five roles

Before it prepares the run, the Harness awaits `models()` until the broker answers or the Host cancels the run, so a broker may wait until it has a model. The Harness picks the model that `RunSetup.selected_model` names, or the first listed model when the field is `None`. A selection absent from the list ends the run with `RequirementsUnmet` and the reason `model_absent`. The Harness passes the Engine a role map with one entry, `default`, so the picked model fills all five roles.

The map has one entry because `RunSetup` has one `selected_model`, and section 7 lists a model for each role as planned work. Workshop's broker uses the wait: it answers once its catalog has a chat-capable model.

### A Host streams through a callback on its setup, and each Host owns its queue

A Host that shows a reply as it forms sets `RunSetup::streaming` to a callback. The Harness hands the callback each piece of a section's own model round, with the round, in the order the model produced the pieces. A round that a nested `models.infer` runs hands the Host its finished reply alone. The Harness passes every piece on, and it neither buffers nor skips one. A broker that streams calls the `on_piece` callback of `InferenceBroker::chat` once for each piece, never after the round ends, and `supports_streaming()` tells a Host whether its broker does, for its display. The record contains the finished reply, as an `assistant_reply` event, and omits the pieces.

The callback is the simpler primitive, and each Host owns its queue. The Host chooses the queue's type, its capacity, what happens when it is full, and how much memory it may use, where a stream from the Harness would choose one answer for every Host. A Host that lags stops taking pieces in its own code, keeps the first part of the reply, and takes the rest from the finished reply.

The callback returns at once and never calls into the Harness. It runs inside the future that drives the run, so a callback that waits stalls every effect of the run. The Harness is not reentrant, so it promises nothing about a call to `RunControl::send`, `RunControl::stop_round`, or `RunControl::cancel` from the callback. A Host that reacts to a piece does so from its own code, outside the callback: the callback hands the piece to a channel, a flag, or a broadcast and returns, and the Host's own task then stops the round or cancels the run. A callback that panics ends its round as a broker that panics does: the Harness answers the round `Dropped`.

### The Host's `Timer` performs each sleep and stamps each run's start

`Timer` is a trait with two methods. `sleep(duration)` returns the future that the loop polls for a `Timer` effect. `now()` returns the time that the Harness records as the run's start and passes to the Engine, and its default reads the system clock.

The timer is a required field of `Host`. As an entry of `HostServices` it could be missing, and a missing timer would show only at a timed wait inside a run.

### The Host fills one `HostServices` map, and the `install` of each Rust Plugin reads it

`HostServices` is a map from a two-segment id, such as `runvil/tokio-runtime`, to a typed object. The Host fills one map at startup and passes it to the `install` function of each Rust Plugin, and a Pack factory stores what its instances need. The C contract lacks Host services: a contract Plugin receives the contract's version and the functions of its load call, so the bashkit Plugin owns a tokio runtime of its own. Services for contract Plugins are planned work (section 7). The recorder, the broker, and the timer are required values outside the map.

A `ServiceKey` binds each id to the type of its object, so a Host provides only an object of that type under the key. [Tools and Packs](tools-and-packs.md) covers a Pack's side.

## 5. The recorder and run events

### The Host supplies the `RunRecorder` and receives a run's events through it

`RunRecorder` is a trait with three methods: `begin_run(meta)` returns a `RunId`, `append(run, record)` adds one record, and `end_run(run, result)` closes the run. The Host chooses the storage: `MemoryRecorder` stores runs in memory, and Workshop's recorder writes a Turso database. Calls for different runs may overlap, so each recorder locks its own state.

A Host sees a run's events through its recorder: `append` is the only call in which the Harness passes an event to the Host.

### The Harness writes every event, effect, and answer in loop order

The loop writes a step's events before it starts the step's effects, each effect before its performer starts, and each answer before the run resumes with it. Every effect record has exactly one answer record, and a dropped effect's answer is `Dropped`. The loop also records an effect that the same step issued and withdrew. The records are the only view of a run's store outside the run: each `Vfs` effect record of a `store` call names its store under the key `store` and holds its operation, and its answer record holds the whole outcome, which is enough to rebuild the store. The record of a `Vfs` effect that a tool call asked for names the call under the key `call` and holds the operation, with the bytes of a write. Its answer record holds a receipt, such as the number of bytes of a read, and the tool's own result is the answer record of its `ToolCall` ([The Engine](engine.md)). A recorder that returns an error stops its run.

The loop records a withdrawn effect because each `Chat` record names an earlier round by id, in its `after` field. With every effect recorded, the `Chat` records of one message list, in record order, are exactly its sends, and every `after` names a recorded round.

### Events are values that `Run::step` returns, and dropping them leaves the run unchanged

An `Event` is one thing that happened in a run, such as a section start, a model reply, or a tool result. `Run::step` returns the events beside the effects, and each has a `section` and a `provenance`, the task id with a sequence number. The Engine's docs state the contract: dropping every event "leaves a run's outputs, errors, and ordering" unchanged.

Events exist to show a run's progress: a run that reported only its result would be one silent call that lasts minutes. [The Engine](engine.md) lists the event kinds.

### A reply from `models.infer` is an `assistant_reply` event with `origin` set to `infer`

Every model reply produces one `assistant_reply` event. Its `origin` field is `chat` for a chat turn and `infer` for a `models.infer` call, so a Host reads one event kind and uses `origin` to separate the two.

A second event kind for `models.infer` replies would duplicate the reply payload, and every Host would have to learn it.

### `RunMeta` has the seed, a reserved flags field, and the start time

`begin_run` receives a `RunMeta` with three fields: `seed`, `flags`, and `started_at`. `flags` is reserved for replay, and the Harness writes 0. The field exists so that an Engine change that alters an exit rule or a protocol detail sets a flag, which a replay of the run reads. The recorder issues the `RunId`, and every later record names the run by that id.

The recorder receives every record under its run's id, so an event omits a name for its run. Workshop's recorder stores its own label for a run in an `agent` column.

### Recorded JSON reads back as the identical value

Every payload that the Harness passes to a recorder is JSON that parses back to the same value and prints as the same text. Object keys are sorted, numbers are finite, and the workspace enables the `float_roundtrip` feature of `serde_json`. `AGENTS.md` states the rule, and a fidelity test in Workshop's run log checks it.

The rule exists for replay, which is still to be built: a replay compares the results of a second run against the stored payloads, so each payload has to read back as the identical JSON value. With the feature off, the float `3.9078000000000004` reads back as `3.9078`.

### Record formats change in place, and every reader changes with them

A commit that changes an event, a record, or `RunMeta` also changes every reader, and it leaves stored data as it is. `AGENTS.md` states the principle for the whole repository: "There are no users and no recorded runs, logs, or stored data to preserve, so backward compatibility does not matter".

## 6. The run inbox, stop, and cancel

### `RunControl` has three calls: `send`, `stop_round`, and `cancel`

`RunControl` is a handle that the Host clones and calls from any thread. `send(text)` queues a message for the run's main chain. `stop_round()` drops the work in flight, and the run continues. `cancel()` ends the run.

A stop is a call of its own because a cancel ends the run. After a cancel, Workshop would have to start a new run with an empty history, and the model would forget a conversation that Workshop still shows. The handle omits an unsend, since a message that `send` accepted is committed.

### `RunControl::send` queues text for the main chain, and `RunReport::unread` returns what the run left unread

`send` pushes the text on a queue and returns `true`. The loop takes the queue the next time it waits, calls `Run::deliver` for each message, records the events that `deliver` returns, and steps the run. Once the run has ended, `send` drops the text and returns `false`. `RunReport::unread` lists the messages still in the inbox at the end and the ones that the loop left undelivered, so that a Host can return them to the person who sent them.

`send` applies only to the main chain: a sub-agent is a task in the same run, and only a run has a `RunControl`. [Tasks and fanout](tasks-and-fanout.md) covers what the main chain does with a message.

### Two events tell the Host whether the main chain waits for input

The Engine emits `inbox_message` when a message enters an inbox and `inbox_waiting` when a chain starts to wait on an empty inbox. Both name the chain in their `provenance`, and the main chain is task `0`. A Host reads them as its recorder receives them: after `inbox_waiting` on task `0` the run waits for the person, and after the next `inbox_message` it continues. Workshop converts them to its `awaiting_input` and `user_message` agent events.

### `RunSetup.has_user` says whether a person answers the main chain

A Host sets `has_user` to `true` when a person can answer the run's main chain, and the run then has the built-in `runvil/user` Pack. With `false`, the Harness marks that Pack unavailable with the reason "this run has no user", and the first step refuses a prompt that declares `uses: [user]`. [Tools and Packs](tools-and-packs.md) covers the `user` Pack.

### A stop drops the effects in flight, and the run continues

`stop_round` raises a flag. When the loop sees it, the loop aborts every effect in flight and every tool call waiting for a slot, answers each `Dropped`, and lowers the flag. A tool call whose `ToolInfo` sets `survives_stop` continues, and keeps its files. The cancel flag is still clear, so each Lua call that waited raises the canceled error, and a `pcall` around it lets the prompt continue.

The loop answers each `Vfs` effect before it waits again, so a stop finds none in flight. A tool call that a stop drops gets no further file operation: the loop drops a request that the call still makes, and answers `Dropped`, unperformed, a `Vfs` effect that the run issued for the call. A stop raised while the loop writes the record of such an effect comes after that one operation: the loop performs and records it, and then drops the call.

The loop lowers the flag even when the set of effects in flight is empty, so a stop leaves every later effect untouched. A stop drops tool calls as well as model rounds, because a tool call in flight inside `models.loop` would otherwise continue, and the agent would still be busy after the stop.

### A stop raised before a send takes effect first

For a Host that calls `stop_round` and then `send`, the stop applies first: the loop drops the round in flight, `models.loop` raises the canceled error, and the message waits in the inbox for the chain's next read. The loop checks for a stop before each step and before it delivers a message. The Engine withdraws a model round that is in flight when a message is delivered, and issues it again with the message. A stop that the loop first sees after that step drops the reissued round unstarted, because the reissued round is the request that the Host stopped.

A message leaves tool calls running. To cut a long tool call short, the Host stops the round and then sends.

### A cancel drops every effect and ends the run with the `canceled` error

`cancel` sets the run's `CancelHandle`. The loop answers every effect `Dropped`, including calls that survive stops and calls that wait for a slot, and steps the run to `Step::Done`. The Engine reads the same flag, and its `pcall` raises a caught error again while the flag is set, because a cancel is the only way to stop a runaway loop. A cancel treats the file operations of a tool call as a stop does: a `Vfs` effect that the run issued for a dropped call is answered `Dropped`, unperformed, and a cancel raised while the loop writes the record of such an effect comes after that one operation.

`CancelHandle::child()` returns a handle that its parent also cancels. It exists so that an orchestrator can cancel its sub-agents. Tests are its only callers, and the Harness creates one handle for each run.

## 7. Planned work

Each of these items is still to be built.

- **Replay and resume.** A driver that builds a run from a recorded seed and start time, passes the recorded answers to `Run::resume`, the messages to `Run::deliver`, and the file requests of tool calls to `Run::request_file`, each in its recorded order and at its recorded step, and reports where the second run diverges. The recorder receives the seed, the start time, every answer, every message, and every file request of a tool call, which is the record of its `Vfs` effect.
- **A model for each role.** `RunSetup` has one `selected_model`, which fills all five roles.
- **Small records for large answers.** An answer record stores the whole tool output, or the whole text that a `store` call read, so the record contains a copy of everything a script reads. The answer record of a tool's file operation holds a receipt, and the effect record of a tool's write holds the bytes. When those bytes are among the tool's arguments, the record holds them twice: in the `ToolCall` effect record and in the `Vfs` effect record.
- **Request limits for a Host.** `GatewayBroker::new` always uses a receive timeout of 120 seconds and a reply cap of 16 MiB.
- **A step hook.** Pack code that rewrites an effect or its answer between `Run::step` and the performers. The loop starts each effect as `Run::step` returned it and resumes the run with each answer as its performer returned it.
- **More Host services.** Secrets, a schedule, a model client for Packs, the parent environment of a sub-run, and a way for a contract Plugin to read a service. The two Rust Plugins read two services: `plugin-web` and `plugin-mcp` read `runvil/tokio-runtime`, the Host's runtime handle, and `plugin-web` reads `runvil/search-provider`, the search provider for the `web` Pack. A contract Plugin reads none.
- **One Plugin API.** `web` and `mcp` as Plugins that follow the C contract, which a Host inserts with `insert_plugin`, and with them contract functions for configuration, Host services, an unavailable Pack with its reason, the origin of a call, and a wait on outside work. The two Plugins are Rust functions named `install`, and `insert_plugin` has tests as its only callers ([Tools and Packs](tools-and-packs.md)).
- **Approvals.** An approval as an effect of its own, with the dialog in the Host. The effect kinds are `Chat`, `Render`, `ToolCall`, `Vfs`, and `Timer`. When the `Policy` of `RunSetup.vfs` returns `Ask` for a file operation of a tool call, the file system refuses it with `VfsError::ApprovalRequired`. The `Policy` never sees a store operation (section 2).
- **Requirement gaps before a run.** A call that returns every gap as data before a run exists. `Host::parse` reports missing Packs, and every other gap ends a run with a `RequirementsUnmet` error.

## 8. Known gaps

Each item names a place where a guide or a doc comment differs from the code, or code that only tests call.

**The language guide**

1. Request limits. The table in chapter 16 says the Harness changes the response cap and the receive timeout. `GatewayChat::with_request_limits` sets them, and `GatewayBroker::new` always uses the defaults.
2. Start time. Chapter 4 says the start is stamped "from the system wall clock". The Harness calls `now()` on the Host's `Timer`, whose default reads the system clock.

**Doc comments and code that only tests call**

3. The doc comment on `RunMeta.seed` says that the run's id derives from the seed. The recorder issues each `RunId`.
4. The doc comment on `HostServices::provide` names the reason of an invalid id as `make_invalid_id_error`. The code writes `invalid_id`.
5. `CancelHandle::child()` has tests as its only callers. Section 6 describes it.

## 9. Questions for the owner

Each question asks for the reason behind a part of the design.

**The API**

1. Why does a prompt name roles and Pack ids while the Host binds them?
2. Why does a parse failure return to the caller, outside the run's record?
3. Why does `runvil` leave out the stepping API and still re-export Engine names that the existing Hosts leave unused?
4. What will set `RunMeta.flags`? The field is reserved, and the Harness writes 0 in it for every run.

**The effect loop**

5. Why do only tool calls count toward the cap of 10, while model rounds start at once?
6. Why does a performer that panics end the run as canceled?

**Inference and the clock**

7. Why does `GatewayBroker` fix the receive timeout and the reply cap, while `GatewayChat::with_request_limits` is public?
8. Why does the Host's `Timer` stamp the run's start?

**The inbox and cancel**

9. Why does a Host learn whether the run waits for input from two events, through its recorder?
10. Why does `send` return `false` after the run has ended, where it could return an error?
11. What will call `CancelHandle::child()`?
12. Why does every run get a record, including one that ends before its first step?

**The file operations of tool calls**

13. Why do only the file requests of tool calls count toward the turn that the loop gives its executor? A Lua loop of `store` calls, and a Lua loop of calls to a tool whose future is ready at its first poll, run on without a return to the executor while the recorder answers at once (section 3).
14. Why does a file operation of a tool call still run when a stop or a cancel is raised while the loop writes the record of its effect? The loop performs and records that one operation and then drops the call, and a `store` operation runs in the same case (section 6).

## 10. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-11 the walk-through's sentence on filling the `PackRegistry` was brought to `runvil` at commit `b0693699d`, where a Host also loads library Plugins with `insert_plugin_dir`. The rest of the page still describes the commits above.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. Workshop, the Host in `runvil`, takes the `runvil` crate from that repository as a git dependency at a pinned commit ([Crates, build, and CI](crates-build-and-ci.md)). The text of the page still describes the code at the commits above, when those crates were in `runvil`.
- **Update.** On 2026-11-13 the text on Plugins and Host services was brought to `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, and `runvil-plugin-api` at `03a9fa3`. Workshop moved to the `runvil-workshop` repository after the commits of the Move note. The update covers the first paragraph of the walk-through, the entry on the one crate in section 1, the entries on `Host::parse` and on `ready` in section 2, the entry on `HostServices` in section 4, two items of section 7, and the Paths bullet. It states that `web` and `mcp` enter a Host through the Rust `install` API and that one Plugin API is planned work. That session read the code and left the build and the tests unrun. The rest of the page still describes the commits above.
- **Update.** On 2026-11-19 the text on the file operations of tool calls was brought to `runvil-harness` at commit `ee199e3a` and `runvil-plugin-api` at `52a4905`, where the C contract is at version 0.5. At those commits a tool call holds no file access: its tool asks for each file operation, the Engine issues the operation as a `Vfs` effect, and the Harness performs it and records it. The update covers two sentences of the summary, the fourth paragraph of the walk-through, the entries on the three layers and on the one crate in section 1, the entry on the `RunSetup` fields in section 2, the entries on the one future and on the `Vfs` effect in section 3, the entry on loop order in section 5, the entries on a stop and on a cancel in section 6, the items on replay and on small records in section 7, questions 13 and 14 of section 9, and the third Update bullet at the top of the page. It states that the loop returns to its executor once in every 32 file requests of tool calls, that the record of a tool's file operation holds the operation and a receipt of its outcome, and that a replay has to repeat the file requests of tool calls beside the answers and the messages. The crate `runvil` has 124 re-exports where it had 121, and `ToolFiles`, `VfsOp`, and `VfsOutcome` are the difference. The count of 17 Engine names that `runvil` omits was taken again and is the same: `VfsOp` and `VfsOutcome` left that group, and `VfsAsker` and `VfsReceipt` joined it. That session read the code and left the build and the tests unrun. The update leaves the rest of the page as it was.
- **Update.** On 2026-11-24 the two entries of section 4 on the inference broker and on streaming were brought to `runvil-harness` at commit `7a106571`, the head of pull request 76, where Workshop's pull request 38 at `runvil-workshop` commit `411e6656` sets the callback. The update covers those two entries and the fourth Update bullet at the top of the page. It states that `chat` takes an optional `on_piece` callback and that `supports_streaming` is a required method, that a Host sets `RunSetup::streaming` and the Harness hands it each piece of a section's own model round with its round, that the callback returns at once and never calls into the Harness, and that a callback that panics ends its round as `Dropped`. At those commits the whole verification list of each repository passes. The rest of the page still describes the commits above.

*2026-11-10 16:41 - Claude Fable 5.2*
