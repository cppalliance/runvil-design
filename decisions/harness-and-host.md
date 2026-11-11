# The Harness and the Host API: design decisions and their reasons

- Scope: the `runvil` crate, `Host`, `RunSetup`, one Harness per run, the effect loop, inference brokers, Host services, the recorder and run events, the run inbox from the Host's side, stop and cancel
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Paths: code paths start at the `crates/` folder of the `runvil` repository

## Summary

A Host is an application that runs prompts, and it depends on one crate, `runvil`. At startup it builds one `Host` struct: its installed Packs, a prompt finder, and a timer. For each run it fills a `RunSetup` with a recorder, an inference broker, a file system, and the run's settings, and creates one `Harness`. `Harness::run` is one future. It steps the Engine, performs each effect, writes every event, effect, and answer to the recorder, and returns a `RunReport`. A `RunControl` sends messages to the run, stops the work in flight, or cancels the run. Sections 1 to 6 describe these parts.

A check of this page against the code found seven gaps and left thirteen questions about reasons for the owner. Section 8 lists the gaps, and section 9 lists the questions. The largest gap is in the language guide: chapters 5 and 17 document `sys.execution` as the run's name, and a section that reads it raises an error.

Section 7 lists the planned work. Its largest item is replay: the recorder receives the seed, the start time, every answer, and every message of a run, and the code that passes them to a second run is still to be built.

## From a Host's setup to a recorded result

At startup a Host creates a `PackRegistry`, calls each Plugin's `install` with it, and passes the registry to `Packs::new`. It builds one `Host` with three fields: `packs`, a `PromptFinder` in `prompts`, and a `Timer` in `timer`. The `run-prompt` example in the `runvil` crate runs one prompt this way, with only the built-in Packs.

For each run the Host calls `Host::parse` with the prompt's name. `Host::parse` finds the source, parses it, and checks its `uses:` list against the installed Packs. The Host then calls `RunSetup::new` with a recorder and a broker, sets the fields this run needs, and calls `Harness::new`. It takes a `RunControl` from `Harness::control` and awaits `Harness::run`.

`Harness::run` first calls the recorder's `begin_run` with a `RunMeta`: a seed drawn from the operating system and the start time from the Host's timer. It asks the broker for its model list and picks the run's model. It writes `input_text` at the prompt's declared `input:` path, creates the run's Pack instances, and builds an Engine `Run`.

The effect loop then calls `Run::step`. It appends the step's events to the recorder, then appends each effect and starts it. A `Chat` effect calls `InferenceBroker::chat`, a `ToolCall` or `Render` effect calls the run's Packs, a `Timer` effect calls `Timer::sleep`, and a `Vfs` effect runs at once on the same thread. When an answer is ready, the loop appends it and calls `Run::resume`. The same wait also ends on a cancel, a stop, or a sent message.

When `Run::step` returns `Step::Done`, the loop calls the recorder's `end_run` with the result. `Harness::run` reads the declared `output:` file and returns a `RunReport`.

## 1. The three layers and the `runvil` crate

### The Engine emits effects, the Harness performs them, and the Host decides policy

The root `AGENTS.md` defines the terms. The Engine, the `runvil-*` crates, parses a prompt, steps a run, and emits an effect each time the run needs a model reply, a tool result, a timer, or a file. The Harness, the `runvil` crate and the `harness-*` crates, steps the Engine, performs every effect, and records every run. A Host is an application such as Workshop, and it "makes every policy decision".

The Harness owns the effect loop because one ordered record of a run exists only if the Harness sees every effect and every answer. Replay and resume, which section 7 lists as planned work, depend on the same condition.

### A prompt names roles and Pack ids, and the Host binds both

A prompt file names a model by a role, such as `strong`, and a set of tools by a Pack id under `uses:`. The Host supplies the rest: its selected model fills the roles, its `PackRegistry` supplies the Packs, and `RunSetup.vfs` is the run's file system.

### A Host depends on one crate, `runvil`

`runvil` re-exports three groups of items: the Harness, the Engine items a Host reads and builds, and the Plugin contract. `Harness`, `Host`, `RunSetup`, `RunReport`, and `RunControl` are at the crate root, the rest of the Harness is in `runvil::harness`, and the Plugin contract is in `runvil::plugin`. The crate has 122 re-exports, each at one path. `cargo xtask api` checks that every type from this repository that a `runvil` signature names is among them.

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

`Host` is a struct with three public fields: `packs`, `prompts`, and `timer`. A Host builds one and shares it behind an `Arc`. `RunSetup` is a struct with ten public fields that the Host fills for each run. The two structs sort every input by lifetime: `Host` contains what every run of one Host shares, and `RunSetup` contains what one run gets on top of the Host.

### `Harness::new` takes the `Host`, a parsed `Prompt`, and the `RunSetup`, and `run` consumes the Harness

`Harness::new(host, prompt, setup)` creates the Harness for one run. `Harness::control()` returns a `RunControl`. `Harness::run()` takes the Harness by value and returns a `Send` future, so the Host takes the control first. A chat conversation is Workshop's own code around one run; see [Workshop: how the app is built](workshop-app.md).

### `Host::parse` parses the prompt before a run exists, and a parse failure returns to the caller unrecorded

`Host::parse(name)` calls `Prompt::parse` with the Host's `PromptFinder`, then checks the prompt's `uses:` list against the installed Packs. It returns the parse error unchanged, or a `RequirementsUnmet` error that names each declared Pack that the Host lacks or that a Plugin inserted as unavailable. The recorder's first call for a run is `begin_run`, inside `Harness::run`.

### `RunSetup::new` takes the recorder and the broker, and every other field has a default

`RunSetup::new(recorder, broker)` sets the two required fields. The other eight start at defaults: a fresh in-memory store in `vfs`, empty values in `selected_model`, `workspace_roots`, `args`, and `input_text`, `RunLimits::default()` in `limits`, 10 in `max_tool_calls_in_flight`, and `false` in `has_user`. Workshop leaves `limits` and `max_tool_calls_in_flight` at their defaults, among them 24 loop rounds and 8 concurrent tasks.

The broker is a required argument because every run needs inference. A Host that leaves it out fails to compile. With an optional broker, the Harness would refuse each run of such a Host as it prepares the run.

### The Harness reads each `RunSetup` field once, and the input, the run, and the output use one file system

The Harness reads each field once per run. This matters most for `vfs`, the run's whole file system: the Harness stages the input in it, the run works in it, and the Harness reads the declared output from it. A prompt reads `selected_model` and the first of `workspace_roots` through `ui()`.

The Host chooses where a run's files are stored, because the Engine only executes the prompt and works in the file system that its caller supplies. [Files and the VFS](files-and-the-vfs.md) covers the file system.

### The Harness writes `input_text` at the declared `input:` path and reads the declared `output:` file

Before the first step, `stage_input` writes `RunSetup.input_text` at the path the prompt declares under `input:`. Text for a prompt that lacks the declaration ends the run with the reason `input_undeclared`. After a completed run, the Harness reads the declared `output:` path into `RunReport.output`. When that file is missing after the run, its result stands and `output` is a `NotFound` error.

### `Harness::run` returns a `RunReport` for every run, and its one error is `RecorderFailed`

`RunReport` has four fields: `run_id`, `result`, `output`, and `unread`. `result` is the final text or the `Error` that ended the run, and a cancel is the error whose code is `canceled`. `output` is the declared output file's text or the reason it is absent. The record begins before the model lookup, so a failed lookup and an early cancel each produce a report. `Harness::run` returns `Err` in one case, a recorder that refused a write, with the code `RecorderFailed`.

The Harness delivers a failed prompt as a result that says why the prompt failed, the same way it delivers a text that the prompt produced through inference.

### The Harness draws each run's seed from the operating system

`begin` in `prepare.rs` draws 32 random bytes with `rand::random()`, builds a `RunSeed`, and passes it to the recorder in `RunMeta` and to the Engine in `RunParams`. The Engine derives the nonce of the untrusted-output envelope from the seed, and a structural check forbids random sources in every Engine crate, so the Harness makes the draw. A seed read from a clock would be guessable, and a guessable seed makes a guessable nonce. [Tools and Packs](tools-and-packs.md) describes the envelope.

### The Harness waits for each Pack's `ready`, and a run that fails its start checks ends at the first step

Before the first step the Harness creates the run's Pack instances and awaits every instance's `ready` together, until all resolve or the Host cancels the run. Each Pack bounds its own `ready`, because a timeout in the Harness would be a policy that belongs to each Pack.

The first `Run::step` checks the run's requirements. For a shortfall, such as a Pack whose factory refused this run, it returns `Step::Done` with a `RequirementsUnmet` error that lists every gap. When the cancel flag is set, the step ends the run with the canceled error. A cancel that ends the wait early leaves a Pack that is still starting with zero tools, and the check would otherwise end a canceled run as refused. [The Engine](engine.md) lists the checks.

## 3. The effect loop

### `Harness::run` is one future that polls every effect

The loop in `effect_loop.rs` puts the future of each effect in flight into one `FuturesUnordered` and polls them inside the future that `Harness::run` returned, on whichever thread polls that future.

The design sets a rule for every performer, which is a broker, a Pack, or a timer: return to the poll quickly, and send blocking work to the Host's own runtime. A performer that blocks stalls every other effect of the run, and the run's stop and cancel with them.

### The Harness answers each `Vfs` effect inline, on the thread that polls the run

A `Vfs` effect is a file operation that the Engine asks for. The loop appends the effect's record, runs the operation with `perform_vfs_op`, appends the answer, and resumes the run before it waits again. When one step issues several `Vfs` effects, the loop answers all of them and then steps once.

The loop calls the file system directly because a performer for file effects would only forward each operation to it, and `spawn_blocking` exists only in tokio, which `harness-runner` excludes. The cost is that an operation on a real disk blocks the Host's executor thread, and file operations from different chains run one at a time, in the order the Engine issued them. The design accepts that cost. The owner's reason: "its ok to block a little. what's 100ms really?"

### The loop answers `Dropped` for a performer or a file backend that panics

The loop catches a panic from a broker, a Pack, a timer, or a file backend, logs it with the effect id, and answers the effect `Dropped`. The Lua call that waited raises the canceled error, which a `pcall` can catch, and an uncaught one ends the run as canceled.

### The Harness sends each tool call to the Pack named by the id's first segment, and at most 10 run at once

`RunPacks`, the run's snapshot of its Pack instances, performs every `ToolCall` and `Render` effect. It finds the Pack from the first segment of the tool id and calls `Pack::call`. The lookup skips the run's tool catalog, because a model can call a model-only tool, which a Pack's `render` adds outside the catalog.

At most `RunSetup.max_tool_calls_in_flight` tool calls run at once, 10 by default, and the loop starts waiting calls lowest effect id first. Chat, render, and timer effects start at once and are outside the count. [Tools and Packs](tools-and-packs.md) covers the cap's reasons and `Pack::call`.

### A pending run whose effects are all answered ends with an `Internal` error, unless its main chain waits for a message

A `Step::Pending` with an empty effect list, while every earlier effect is answered, shows that the loop lost an effect. The loop ends the run with an `Internal` error: "the effect loop has nothing to await for a pending run". The Engine ends its own stalls with the code `stalled` or `internal`; see [The Engine](engine.md).

When the main chain, the run's root task, waits on its inbox of sent messages, `Step::Pending` sets `waiting_for_input`, and the loop waits for a `RunControl::send` or a cancel. The same applies to a run whose `has_user` is `false`, since a Host may still call `RunControl::send` from code.

## 4. Inference, the clock, and Host services

### The Host supplies model inference as an `InferenceBroker` with two methods

`InferenceBroker` is a trait with two methods. `models()` lists the models the Host offers. `chat(binding, messages, tools, options, round)` runs one model round and returns the finished reply, and the loop calls it for each `Chat` effect. The broker decides where it sends each round, so a Gateway change applies to the next round.

The Host supplies inference so that the Host's broker owns the Gateway's URL, its key, and its model catalog, and `RunSetup` needs only the broker. [Model calls and conversations](models-and-conversations.md) covers the request a round sends.

### The Harness picks the run's model once, and that model fills all five roles

Before it prepares the run, the Harness awaits `models()` until the broker answers or the Host cancels the run, so a broker may wait until it has a model. The Harness picks the model that `RunSetup.selected_model` names, or the first listed model when the field is `None`. A selection absent from the list ends the run with `RequirementsUnmet` and the reason `model_absent`. The Harness passes the Engine a role map with one entry, `default`, so the picked model fills all five roles.

The map has one entry because `RunSetup` has one `selected_model`, and section 7 lists a model for each role as planned work. Workshop's broker uses the wait: it answers once its catalog has a chat-capable model.

### The broker returns each reply whole, and a Host streams inside its own broker

`InferenceBroker::chat` returns one finished `Completion`. A Host that shows a reply as it forms streams the pieces from inside its broker. `GatewayBroker::chat_streaming` runs the same round and passes each `StreamDelta` to a callback. The record contains the finished reply, as an `assistant_reply` event, and omits the pieces.

The owner's rule: "the engine, the harness would not know anything about streaming. only the host, through the inference broker. and for headless hosts, they dont need to stream at all".

### The Host's `Timer` performs each sleep and stamps each run's start

`Timer` is a trait with two methods. `sleep(duration)` returns the future that the loop polls for a `Timer` effect. `now()` returns the time that the Harness records as the run's start and passes to the Engine, and its default reads the system clock.

The timer is a required field of `Host`. As an entry of `HostServices` it could be missing, and a missing timer would show only at a timed wait inside a run.

### The Host fills one `HostServices` map, and each Plugin's `install` reads it

`HostServices` is a map from a two-segment id, such as `runvil/tokio-runtime`, to a typed object. The Host fills one map at startup and passes it to each Plugin's `install`, and a Pack factory stores what its instances need. The recorder, the broker, and the timer are required values outside the map.

A `ServiceKey` binds each id to the type of its object, so a Host provides only an object of that type under the key. [Tools and Packs](tools-and-packs.md) covers a Pack's side.

## 5. The recorder and run events

### The Host supplies the `RunRecorder` and receives a run's events through it

`RunRecorder` is a trait with three methods: `begin_run(meta)` returns a `RunId`, `append(run, record)` adds one record, and `end_run(run, result)` closes the run. The Host chooses the storage: `MemoryRecorder` stores runs in memory, and Workshop's recorder writes a Turso database. Calls for different runs may overlap, so each recorder locks its own state.

A Host sees a run's events through its recorder: `append` is the only call in which the Harness passes an event to the Host.

### The Harness writes every event, effect, and answer in loop order

The loop writes a step's events before it starts the step's effects, each effect before its performer starts, and each answer before the run resumes with it. Every effect record has exactly one answer record, and a dropped effect's answer is `Dropped`. The loop also records an effect that the same step issued and withdrew. A recorder that returns an error stops its run.

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

`stop_round` raises a flag. When the loop sees it, the loop aborts every effect in flight and every tool call waiting for a slot, answers each `Dropped`, and lowers the flag. A tool call whose `ToolInfo` sets `survives_stop` continues. The cancel flag is still clear, so each Lua call that waited raises the canceled error, and a `pcall` around it lets the prompt continue.

The loop lowers the flag even when the set of effects in flight is empty, so a stop leaves every later effect untouched. A stop drops tool calls as well as model rounds, because a tool call in flight inside `models.loop` would otherwise continue, and the agent would still be busy after the stop.

### A stop raised before a send takes effect first

For a Host that calls `stop_round` and then `send`, the stop applies first: the loop drops the round in flight, `models.loop` raises the canceled error, and the message waits in the inbox for the chain's next read. The loop checks for a stop before each step and before it delivers a message. The Engine withdraws a model round that is in flight when a message is delivered, and issues it again with the message. A stop that the loop first sees after that step drops the reissued round unstarted, because the reissued round is the request that the Host stopped.

A message leaves tool calls running. To cut a long tool call short, the Host stops the round and then sends.

### A cancel drops every effect and ends the run with the `canceled` error

`cancel` sets the run's `CancelHandle`. The loop answers every effect `Dropped`, including calls that survive stops and calls that wait for a slot, and steps the run to `Step::Done`. The Engine reads the same flag, and its `pcall` raises a caught error again while the flag is set, because a cancel is the only way to stop a runaway loop.

`CancelHandle::child()` returns a handle that its parent also cancels. It exists so that an orchestrator can cancel its sub-agents. Tests are its only callers, and the Harness creates one handle for each run.

## 7. Planned work

Each of these items is still to be built.

- **Replay and resume.** A driver that builds a run from a recorded seed and start time, passes the recorded answers to `Run::resume` and the messages to `Run::deliver`, and reports where the second run diverges. The recorder receives the seed, the start time, every answer, and every message of a run.
- **A model for each role.** `RunSetup` has one `selected_model`, which fills all five roles.
- **Small records for large answers.** An answer record stores the whole tool output or file text, so the record contains a copy of everything a script reads.
- **Request limits for a Host.** `GatewayBroker::new` always uses a receive timeout of 120 seconds and a reply cap of 16 MiB.
- **An offload for blocking file work.** A hook, supplied by the Host, that runs file operations off the thread that polls the run. The loop runs each file operation inline on that thread.
- **A step hook.** Pack code that rewrites an effect or its answer between `Run::step` and the performers. The loop starts each effect as `Run::step` returned it and resumes the run with each answer as its performer returned it.
- **More Host services.** Secrets, a schedule, a model client for Packs, and the parent environment of a sub-run. The Plugins in the repository read two services: `runvil/tokio-runtime`, the Host's runtime handle, and `runvil/search-provider`, the search provider for the `web` Pack.
- **Approvals.** An approval as an effect of its own, with the dialog in the Host. The effect kinds are `Chat`, `Render`, `ToolCall`, `Vfs`, and `Timer`. When the file system's `Policy` returns `Ask` for an operation, the file system refuses it with `VfsError::ApprovalRequired`.
- **Requirement gaps before a run.** A call that returns every gap as data before a run exists. `Host::parse` reports missing Packs, and every other gap ends a run with a `RequirementsUnmet` error.

## 8. Known gaps

Each item names a place where a guide or a doc comment differs from the code, or code that only tests call.

**The language guide**

1. Run name and prompt hash. Chapter 4 lists `name` and `prompt_hash` among the values of a run's record and says that every event has an `execution` field. `RunMeta` has `seed`, `flags`, and `started_at`, and each `Event` has a `section` and a `provenance`.
2. `sys.execution`. Chapters 5 and 17 document it as the run's name, "which the Harness assigns". A section that reads it raises `unknown sys field 'execution'`, and the test `sys_holds_no_execution_label` checks that.
3. Request limits. The table in chapter 16 says the Harness changes the response cap and the receive timeout. `GatewayChat::with_request_limits` sets them, and `GatewayBroker::new` always uses the defaults.
4. Start time. Chapter 4 says the start is stamped "from the system wall clock". The Harness calls `now()` on the Host's `Timer`, whose default reads the system clock.

**Doc comments and code that only tests call**

5. The doc comment on `RunMeta.seed` says that the run's id derives from the seed. The recorder issues each `RunId`.
6. The doc comment on `HostServices::provide` names the reason of an invalid id as `make_invalid_id_error`. The code writes `invalid_id`.
7. `CancelHandle::child()` has tests as its only callers. Section 6 describes it.

## 9. Questions for the owner

Each question asks for the reason behind a part of the design.

**The API**

1. Why does a prompt name roles and Pack ids while the Host binds them?
2. Why does a parse failure return to the caller, outside the run's record?
3. Why does `runvil` leave out the stepping API and still re-export Engine names that the existing Hosts leave unused?
4. What will set `RunMeta.flags`? The field is reserved, and the Harness writes 0 in it for every run.

**The effect loop**

5. Why do only tool calls count toward the cap of 10, while model rounds start at once?
6. Why does a completed run with a missing output file still succeed?
7. Why does a performer that panics end the run as canceled?

**Inference and the clock**

8. Why does `GatewayBroker` fix the receive timeout and the reply cap, while `GatewayChat::with_request_limits` is public?
9. Why does the Host's `Timer` stamp the run's start?

**The inbox and cancel**

10. Why does a Host learn whether the run waits for input from two events, through its recorder?
11. Why does `send` return `false` after the run has ended, where it could return an error?
12. What will call `CancelHandle::child()`?
13. Why does every run get a record, including one that ends before its first step?

## 10. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.

*2026-11-10 16:41 - Claude Fable 5.2*
