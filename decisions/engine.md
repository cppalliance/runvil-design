# The Engine: design decisions and their reasons

- Scope: the Engine as a library: parsing a prompt, the run state machine, effects and their answers, run limits, the one error type, the public surface, and the internal crates
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the Paths bullet names the repositories as of `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`, after the Engine and the Harness moved (section 12)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil`

## Summary

The Engine is a Rust library that parses a prompt file and runs it as a state machine. `Prompt::parse` compiles each Lua block and returns a `Prompt`, and `Run::new` builds a `Run` from it. Each `Run::step` runs the prompt's Lua until the run waits for answers and returns the effects the run needs: model rounds, Pack renders, Pack tool calls, `store` operations, and timers. The caller performs each effect and passes its answer to `Run::resume`. A run ends with its final text or with one `Error`. Six private crates contain the code, and `runvil-engine` re-exports the 107 names a caller uses. Sections 1 to 8 describe these parts.

A check of this page against the code found three gaps and left ten questions about reasons for the owner. Section 10 lists the gaps, and section 11 lists the questions. The largest gap is in chapter 16 of the language guide, which teaches six parse error kinds and calls the set open, while the code has one closed `ErrorCode` enum.

Section 9 lists the planned work. Its largest item is child prompts: a Lua `execute()` call that runs another prompt by name.

## From a prompt file to a finished run

The `greeter` example in `runvil-engine/examples/greeter.rs` is a whole caller loop. Its prompt has one section with one Lua block, which calls `store.write`, `store.read`, `models.infer`, and `tools.call` and returns the last result. The example calls `Prompt::parse` with a closure that returns the prompt's text. The parser splits the front matter from the body, builds the section tree, and compiles the block into a `LuaProgram`.

The example builds a `RunParams` from a `RunSeed` and a `Timestamp`, adds a tool catalog, a role map, and a `CancelHandle`, and calls `Run::new`, which creates a `Scheduler`. The first `Run::step` checks that the run can start. It then starts the main chain, the scheduler's record of the walk through the sections, builds a `SectionContext` with a fresh `SectionVm`, and starts the block as a coroutine.

`store.write` is a shim, a short Lua function that yields a table. The Lua crate parses the table into `Request::Store`, the scheduler builds an `Effect::Vfs`, and `EffectTable::issue` numbers it and records the waiting chain. `step` returns `Step::Pending` with one `IssuedEffect`. The example performs it with `perform_vfs_op` and calls `Run::resume` with `EffectAnswer::Vfs`.

`Run::resume` converts the answer into an `Answer` and queues the chain, and the next `step` resumes the coroutine. The Engine issues `models.infer` as `Effect::Chat` and `tools.call` as `Effect::ToolCall`, and the example answers each the same way. The block's `return` ends the run, and the last `step` returns `Step::Done` with the returned text as its `result`.

## 1. The Engine and its caller

### The Engine returns the work it needs as an effect, and its caller performs it

An effect is a Rust value that describes one piece of outside work, such as a model round. `Run::step` returns the effects a run needs, and the run waits until the caller passes each answer to `Run::resume`. In production the caller is the Harness ([The Harness and the Host API](harness-and-host.md)). `step` and `resume` are synchronous functions, and a structural check fails when an Engine crate depends on an async runtime or an HTTP client ([Crates, build, and CI](crates-build-and-ci.md)).

The Engine returns this work as values so that it is independent of any runtime and is a deterministic function of its inputs.

### The Engine returns five kinds of work as effects and performs the rest itself

`Effect` has five variants: `Chat` for one model round, `Render` for a Pack's rendering of offered tools, `ToolCall` for one call of a Pack's tool, `Vfs` for one `store` operation, and `Timer` for one sleep. [Tools and Packs](tools-and-packs.md) covers `Render` and `ToolCall`.

The Engine performs four kinds of work itself and reports each as an event: an adhoc tool call, a `GetDynamicTools` call, a refused discovery call, and a `store` call that a Pack prelude makes while it installs.

### The caller supplies the start time and a 256-bit seed

`RunParams::new` takes a `RunSeed` and a `Timestamp`, and nine `with_` setters add a run's other inputs. Lua reads the timestamp as `sys.when`, every timeout is a `Timer` effect, and clippy rejects a clock read in an Engine crate. The same prompt, arguments, params, and answers produce the same effects, events, and result ([Tasks and fanout](tasks-and-fanout.md)).

The seed is 32 bytes, and the Engine derives from it, with SHA-256, the nonce of the envelope that it wraps around untrusted text ([Tools and Packs](tools-and-packs.md)). A 64-bit seed would cap the nonce's unpredictability at 64 bits.

### Plugin code and wire code are outside the Engine

The Engine receives each tool as a `ToolInfo`, each Pack prelude as Lua source, and each model reply as a `Completion`. `runvil-plugin`, the crate with the `Pack` trait, depends on two Engine crates, and the Engine crates are independent of it. The Engine sees only a Pack's descriptors, preludes, and ids. A Pack whose tool calls the Harness performs is a catalog Pack in Engine code. The name leaves the Harness out, because Engine crates name the Harness only to assign it a responsibility.

The Harness's gateway client parses SSE streams and OpenAI bodies, because the Engine is neutral about the transport.

## 2. Parsing a prompt

### `Prompt::parse` takes a `PromptFinder` and a name, and the name identifies the prompt

`PromptFinder` is a trait with one method, `find(name)`, which returns a prompt's source text. `Prompt::parse(finder, name)` calls it and parses the text, and a closure of the form `Fn(&str) -> Result<String, Error>` is a finder. `Prompt::file` and every failure name the prompt by the caller's name for it, such as `research.md`.

Once child prompts exist, prompts will refer to each other by these names, so the name also works as the key for spotting duplicates. Section 9 lists child prompts as planned work.

### `Prompt::parse` compiles every Lua block

The parser compiles each `lua` fence into a `LuaProgram`, which contains the source text and Lua 5.5 bytecode. A syntax error in any block fails the parse with the code `lua_compile`, because a `Prompt` that parses must be executable. The bytecode exists in memory, and the type's docs say it "must not be persisted". `Prompt::frontmatter` returns everything the front matter declares, and the run's first step checks those declarations against the caller's params (section 3).

### A parsed prompt has one shared body string, and the Engine reads it in place

The parser stores the prompt's body, the text after the front matter, as one `Arc<str>`. A `BodyText` is a handle to that string plus a byte range, and each section's raw Markdown, prose blocks, and list items are `BodyText` values. A run reads sections through `Visible`, a borrowed view of the sections that a running section may name.

### One function, `find_visible`, looks up every heading in a visible set

A running section may name its siblings and its direct children by heading, and that list is its visible set ([The prompt language](prompt-language.md)). `find_visible` searches the set for `jump`, `call`, a task spawn, and a `prompt.section` or `prompt.list` read. `read_visible` turns a miss into `nil`, and `resolve_visible` turns it into an error. A heading outside the set and a heading absent from the file get the same answer, because telling them apart would need the whole document, and a miss would then reveal what the visible-set rule hides. `require` searches a wider set with a second function, `find_reachable`.

## 3. The run state machine

### A caller steps a run with four functions

`Run::new(prompt, args, params)` builds a run. `Run::step` advances it until it needs an answer and returns a `Step`. `Run::resume(id, answer)` delivers the answer to one effect. `Run::deliver(text)` queues a message for the main chain's inbox ([The Harness and the Host API](harness-and-host.md)). `Run` is `Send` and owns its prompt through an `Arc`.

The caller answers effects one at a time, in any order. Parallel work in a fanout depends on that: the caller resumes one arm while the effects of the other arms are in flight.

### `Run::new` always succeeds, and the first `step` checks that the run can start

The first `Run::step` calls `check_start`, which makes three checks in order. The prompt declares `runvil: 0`. The params' file system declares a store whose backend passes a probe. The params meet what the prompt requires: each Pack under `uses:` is `Usable` in the roster that the caller passed to `RunParams::with_packs`, and the model in each declared role meets that role's requirements. A requirements failure has the code `requirements_unmet`, and its message names every gap.

### A step returns its effects and its events as values

`Step::Pending` has four fields: `effects`, `events`, `withdrawn`, and `waiting_for_input`. Each element of `effects` is an `IssuedEffect` with an `id`, a `provenance`, and the `effect`. `events` lists what the run reported since the previous step. Because a step returns effects and events as values, the Engine is pure, the Harness receives one ordered stream, and a test asserts on a vector. The caller receives events one step at a time, and a step runs at the speed of its Lua.

An `EffectId` may differ between runs. The `provenance`, a task and a position within it, is the same in every run with the same inputs and answers.

### `Step`, `Effect`, and `EffectAnswer` are closed enums

The three enums omit the `#[non_exhaustive]` attribute, so a caller's `match` names every variant, and a new variant fails every caller's build. The Harness matches these values. With `#[non_exhaustive]` it would need wildcard arms, and it would still compile after the Engine gains a variant.

### Every effect gets exactly one answer, and `Dropped` is an answer

`EffectAnswer` has one variant for each effect kind, plus `Dropped`. A caller that abandons an effect answers it `Dropped`, and the chain that waited resumes with the canceled error. `Run::step` returns `Step::Done` once the caller has answered every issued effect, so a store operation still in progress finishes inside the run's file scope.

Three caller mistakes end the run with an `internal` error: an answer of the wrong kind, an answer for an unknown id, and a second answer for one effect. The Engine discards an answer for an effect whose chain stopped waiting.

### A step lists the effects it withdraws

`withdrawn` lists the ids of effects whose answers the run will discard: an effect of a canceled or abandoned task, a `models.loop` round that a delivered message replaces, and every outstanding effect in the step that decides the run's result. The caller may skip or abort each one, and it still owes each the answer `Dropped`.

The step names the withdrawn effects in a list, because a shared cancel flag inside each effect would stop effects from being plain recorded data and would hide the withdrawal from logs and tests. A flag would also leave a running call running. The list covers every kind of effect, so the caller can abort a model round of a canceled task and save the cost of a reply that the run would discard.

### The caller cancels a run through the `CancelHandle` in its params

A caller passes a `CancelHandle` to `RunParams::with_cancel` and sets the flag, from any thread, to cancel the run. The scheduler polls the flag before each chain step, and a hook in every Lua VM polls it too. The next `step` ends every chain and lists every outstanding effect in `withdrawn`, and the run ends with the code `canceled` once the caller has answered them.

The two polls follow one rule: the Engine honors a cancel while a block runs and while a chain waits on a request.

### A run ends with its final text or with one `Error`

`Step::Done` has three fields: `result`, `events`, and `unread`. `result` is a `Result<String, Error>`: `Ok` is the run's final text, a canceled run ends with the error whose code is `canceled`, and any other error is a failure. `unread` lists the messages still in the main chain's inbox.

A prompt declines its task by returning its reason from Lua, so the run completes and `result` is `Ok` with that text. A prompt refuses to start in a different way: a block under its H1 title raises a Lua error, and the run ends with the code `requirements_unmet` ([The prompt language](prompt-language.md)).

### A run where every chain waits ends as `stalled` or as `internal`

A run stalls when its ready queue is empty, all its awaited effects have their answers, and its main chain waits on something other than its inbox. The scheduler then ends it. When tasks wait on their inboxes, the error has the code `stalled` and names them. Otherwise the error is `internal`, because only an Engine fault produces that state.

A main chain that waits on its inbox is a different case: `step` returns `Pending` with `waiting_for_input` set, and the caller continues with `Run::deliver` or a cancel. The case differs because the main chain's inbox is the only one that code outside the Engine can fill.

## 4. Effects and their records

### `Effect::Chat` contains the whole request

A `Chat` effect has five fields: the model `binding`, the `messages` in wire order, the `tools` schemas, the `round`, and `after`. The messages are the full request, so the caller can perform the round from the effect. The request settings are in the binding only: a second copy in the effect could differ from the binding, and the effect's record would then describe a request that differs from the one sent.

`after` names the earlier round whose request this one extends, with the number of leading messages the two share, and the effect's record omits those messages.

### The Engine numbers the model rounds of a run from 0

One counter numbers every model round of a run in issue order, and loop rounds and nested `models.infer` rounds share it. Each `Effect::Chat` has a `Round` with that `RoundId` and an origin, `Chat` or `Infer`. The Engine writes the same id in the round's thinking, reply, and tool-call events, so a caller that streams partial output matches it to the finished events. A `Round` has only these two values, because a caller needs only a round's id and whether the round is user-facing.

### `Effect::Vfs` is one `store` operation

The Engine issues a `Vfs` effect for each of the eight `store.*` calls. The effect has an `access`, the calling chain's view of the run's store, and a `VfsOp` with the call's arguments. A caller answers it with `perform_vfs_op(&access, op)`, a synchronous function ([Files and the VFS](files-and-the-vfs.md)).

Lua says `store`, and the names below the Lua layer say `Vfs`, because every one of these operations is a VFS operation made through the run's store view.

### `Effect::record` and `EffectAnswer::record` return JSON for the run log

`Effect::record` returns the effect as a `serde_json::Value`, minus its live handles. A `ToolCall` record omits the file access, and a `Timer` record stores seconds, though the effect has a `Duration`. `EffectAnswer::record` returns the answer's outcome, and a failure is an object with a `kind`, the tag of its `ErrorCode` or `VfsError`, and a `message`. The typed records behind the two functions, `EffectRecord` and `AnswerRecord`, are private to `runvil-runtime`.

## 5. Inside a step: chains, section VMs, and requests

### The scheduler builds one `SectionContext` with a fresh Lua VM for each section entry

A chain is the scheduler's record of one sequence of sections: the H1 pass, which runs the blocks under the title once, the main walk, a `call` child, or a task ([Tasks and fanout](tasks-and-fanout.md)). Each time a chain enters a section, the scheduler builds a `SectionContext`, which the code calls a frame. The frame owns a new `SectionVm`, a sandboxed Lua state that runs every block of the section, and the frame's `Drop` destroys it. The next section's VM starts with the Engine globals and the chain's `var`.

The rule is one `SectionContext` for each section entry, whichever way the chain entered the section: by falling through from the section before, by `jump`, or by `call`. One VM shared across sections would create hidden globals that depend on the path the run took, uncloneable state, and inconsistent section inputs.

### Section VM setup runs in one fixed order, with the limits first

The frame constructor creates the VM and applies the run's memory and log limits. `setup_section_vm` then runs its installs in this order:

1. It injects `args`, `sys`, `var`, and the store access.
2. It installs the Engine globals, such as `log` and `store`.
3. It installs `jump` and the `prompt` reads.
4. It installs the coroutine shims, `models.loop` among them.
5. It runs each Pack prelude.
6. It installs `require`.
7. It replaces the store functions with yielding shims.

A prelude runs before step 7 because it runs outside a coroutine, where a yield fails. The limits apply before any install, so each prelude runs under the run's own limits.

### A Lua call that waits yields a request from a coroutine

The scheduler starts every Lua block as a coroutine with `start_block_coro`. A call that waits, such as `models.infer`, is a Lua shim that yields a table. The Lua crate parses the table into a `Request`, and the scheduler answers it at once, or issues an effect and records what the chain waits on. Later, `resume_block_coro_answer` resumes the coroutine with an `(ok, result)` pair. A yield from author code fails the block with the message "scripts may not yield directly". The shims are Lua, in files named `__impl_*.lua`, because a yield across the C boundary fails.

One thread runs a whole run. A run needs only one, because every Lua block executes fast.

### The Engine uses mlua's synchronous API on Lua 5.5

The workspace builds `mlua` 0.12 with the features `lua55`, `vendored`, `serialize`, and `send`, and the Engine calls its synchronous API. The `async` feature of mlua is off, permanently and unconditionally. Runvil's needs are modest, and for them a small request protocol that the project owns beats a general-purpose mechanism that the project would depend on and that is outside its control. The owner's rule: "I never want async mlua."

### `Request` is one closed enum, and every `Answer` variant is a success

Every call that suspends a block is one variant of `Request` in the Lua crate, with typed fields that the parse checks. The enum has 20 variants. A leaf request, such as `Infer` or `Store`, becomes an effect. A structural request, such as `Call` or `Spawn`, changes which chains the scheduler runs. The enum is closed because it is the audit surface: one file lists everything a script can cause.

The scheduler resumes a block with a `Result` over `Answer`, and every failure is the `Err` side, because the Lua side reads the variant of an answer only when the answer is a success.

### The rules of `models.loop` run in Rust, and a Lua trampoline performs each action

`models.loop` is a short Lua function over `drive`, the trampoline. `drive` calls a Rust step function, performs the one action the step returns, and calls the step again with the outcome. The actions are: yield a request, call a tool handler, call the compactor, raise an error, and return. The Rust side is a state machine named `Machine`.

The trampoline is Lua because a Rust function can yield only with mlua's `async` feature, and its call to a Lua function that yields fails. In the wider division, Lua in the prompt decides policy, such as compaction, and Rust runs the mechanics ([Model calls and conversations](models-and-conversations.md)).

### The `Scheduler` owns five state objects, and a one-chain request runs in a `ChainOps` view

A `Run` owns one `Scheduler`, and the scheduler owns five objects, each with its own state and rules. `ChainArena` stores the chains, `EffectTable` lists the effects that await an answer, `TaskArena` stores the tasks, `Inboxes` stores unread messages, and `Admission` queues the chains that wait for a concurrency slot. The scheduler itself owns the ready queue and the run's phase, and it performs every action that writes to two of the five.

A request that changes one chain and the effect table, such as a model round, runs in `ChainOps`, a view that borrows that chain and the table. `Scheduler::with_chain` is the one entry to the view. Functions in the `execute` module take the chain's `ChainState` as their first parameter, `ctx`.

### A chain's position is a stack of `SectionPos` values

A `SectionPos` names a section by the path to its list of siblings and an index into that list. Each chain has a `Walk`: the current position on top and, below it, the sections whose jumps descended into child sections. A jump to a sibling replaces the top, a jump to a child pushes, and the end of a child level pops.

## 6. Run limits

### `RunLimits` is a struct with six public fields

A caller changes a limit by assigning a field of `RunLimits::default()` and passing the struct to `RunParams::with_limits`. The fields and their defaults:

- `max_tool_iterations`: 24 model rounds in each `models.loop` call.
- `tool_iterations_ceiling`: 1000, the cap on the field above and on a prompt's own `max_tool_iterations`.
- `max_concurrency`: 8 tasks admitted at once.
- `max_idle_tasks`: 8 tasks idle after a reply.
- `lua_memory_bytes`: 64 MiB for each Lua VM.
- `lua_log_events`: 1024 `log` calls for each Lua VM.

The fields are public because `RunLimits` is a plain `Copy` value whose only invariant is in its `NonZero` types. The call depth cap is a scheduler constant, `MAX_CALL_DEPTH`, with the value 8, and a deeper `call` or task fails with the code `call_depth_exceeded`.

### A Lua block may run any number of instructions, and a hook polls the cancel flag

Each Lua VM has a hook that fires every 10,000 instructions and reads the run's cancel flag, and a set flag aborts the block. An endless loop in a prompt therefore runs until the caller cancels the run. The owner's reason: "If the user codes an infinite loop that's their problem".

The Engine sets two quotas, both on `log`: a count of events and a count of bytes. A call past either one raises an error with the code `lua_quota`.

## 7. The one error type

### Every Engine and Harness failure is one `Error` record with an `ErrorCode`

`Error` is a struct in `runvil-types` with five public fields: `code`, `message`, `prose`, `at`, and `facts`. `ErrorCode` is an enum of 69 codes, each with a number and a tag such as `lua_runtime` (201). The hundreds digit names the family: 1xx parse, 2xx Lua, 3xx tasks, 4xx model, 5xx tools, 6xx store, 7xx run, and 8xx caller. The enum's docs add: "A number is never reused." The root `AGENTS.md` makes `Error` the one error type of the Engine and Harness crates, with `VfsError` as the main exception.

`Error`, `ErrorCode`, and `ErrorFact` omit the `#[non_exhaustive]` attribute, so a new code breaks the build of a Host whose `match` on `ErrorCode` names every variant.

### An error's `message` includes each cause once, and its `prose` is the text a model reads

`message` is the full diagnostic for logs and people, and `Display` prints it. `Error::with_cause` appends the text of each cause and skips text the message contains. The owner's rule: "Keep the full self-contained message in Display".

`prose` is a second text for a model: what failed and what to do next. Each `ErrorCode` has a default prose. The field exists because a model may read any string that the system produces.

### One location shape describes a place in a prompt or in Rust source

`at` is an optional `SourceLoc`: a file, a start line and column, and an end line and column. The file is the prompt's name, or for an `internal` failure the Rust source file that built the error. The field's name, `file`, is generic, so one shape describes both kinds of place.

`Error::located_message` returns the message followed by the location, as `(at file:line)` for most codes. The Lua crate's `make_internal_error` is `#[track_caller]`, so an internal failure names the line that found the fault.

### A Lua failure the author caused has the code `lua_runtime`, and a broken Engine rule has `internal`

The Lua crate builds a failure with `make_lua_runtime_error` when the prompt author caused it and can fix it in Lua, such as an empty `messages` list. It builds one with `make_internal_error` when an Engine rule broke, such as a poisoned lock. The code names the party at fault, and a failure with the wrong one of the two codes would blame the wrong party.

## 8. The public surface and the internal crates

### `runvil-engine` contains re-exports, and each item has one path

The crate's one source file has 107 `pub use` lines, at the root and in ten `pub mod` blocks, and private crates define every item. The code calls such a crate a facade. Each line re-exports one item by name from the crate that defines it. A structural check rejects a glob, a list, and an item definition, and `cargo xtask api` rejects a re-exported module ([Crates, build, and CI](crates-build-and-ci.md)). The `runvil` crate, which a Host depends on, is a second facade ([The Harness and the Host API](harness-and-host.md)).

A caller finds every item in this one crate. The owner's reason: "I dont want types scattered around different crates. I have discovered it becomes hard to browse the cargo doc that way." Each line names a single item because a re-exported module or crate would expose everything that the module later gains, and a second path to an item would make the API ambiguous.

### The facade re-exports an item that non-test code outside the Engine uses

The facade re-exports an item when production code of a crate outside the Engine uses it. A use in a test, a bench, or a doc example is outside the rule, and the author rewrites the test. The goal is a minimal surface in which every item is necessary. The owner's rule: "I want ZERO public API surface growth just for tests." Every change to the surface appears as a difference in `runvil-engine/public-api.txt`.

So the Engine builds every `ToolSchema`: a public constructor would widen the facade only for tests.

### A signature on the surface names facade items, the standard library, and four serde crates

`cargo xtask api` reads rustdoc's JSON for the facade and fails when a signature, field, or impl on the surface names anything else. The four crates are `serde`, `serde_core`, `serde_json`, and `serde_yaml_ng`. The limit applies to the surface only: the internal crates may use many external crates, and the surface is restricted to a few basic ones.

### An operation reserved for Engine crates is a free function in a `detail` module

Some operations on a public type are reserved for Engine crates, such as building a `ToolSchema` or reading a prompt's sections. Each is a free function in a module named `detail` in the crate that defines the type. `runvil-model-client` and `runvil-prompt` each have one, and the facade omits both modules.

The operations are free functions because a free function in a module outside the facade is invisible on the type's docs page. A hidden trait implemented on a public type can render there as an impl that links into an internal crate.

### The Engine is six private crates, each with one job

The crates are in `runvil-internal/`, and code outside that folder imports them through `runvil-engine` and `runvil-plugin` ([Crates, build, and CI](crates-build-and-ci.md)).

- `runvil-types` defines ids, events, metrics, `Error`, the cancel flag, and the untrusted envelope.
- `runvil-vfs` is the virtual file system.
- `runvil-model-client` defines messages, tool schemas, completions, and model bindings.
- `runvil-lua` runs the Lua sandbox and defines the Engine globals and the request protocol.
- `runvil-prompt` parses a prompt and compiles its Lua, and uses `runvil-lua` for `LuaProgram`.
- `runvil-runtime` contains `Run`, the scheduler, and the effects.

The owner's goal: "I want as many internal crates as possible and I want maximum internal structure." A crate boundary is what stops `runvil-lua`, for example, from using the scheduler's internals.

## 9. Planned work

Each of these items is still to be built.

- **Child prompts.** A Lua `execute()` call that runs another prompt by the name the Host uses for it. A run executes one prompt, and `Prompt::parse` finds that prompt by name through a `PromptFinder`.
- **Replay.** Harness code that runs a recorded run a second time and compares the records of the two runs. The Engine attaches a `Provenance` to each effect and event, and `Effect::record` and `EffectAnswer::record` return the JSON that a run log stores.
- **The store as a tool.** The `store` API as a tool, which would move the `Vfs` effect and the store probe out of the Engine. The Engine issues a `Vfs` effect for each `store.*` call, and the first `step` probes the store's backend.
- **The round id on `ModelTurnFailed`.** A round id in the `ModelTurnFailed` event, so that a Host can drop exactly the failed round's streamed pieces. The Engine writes the round id in a round's thinking, reply, and tool-call events.
- **Bounds on JSON conversion.** A size budget at every site that converts a Lua table. `json.encode` and `var` writes have a size budget. Six other sites in the Lua crate convert a table of any size, and one encode runs as a single Rust callback between cancel polls.

## 10. Known gaps

Each item names a place where the language guide differs from the code.

1. Run limits. Chapter 16 has a section titled "The five run limits" and says that a prompt's own value "replaces the run's round cap". `RunLimits` in `runvil-internal/runtime/src/execute/params.rs` has six fields, and `ChainState::max_tool_iterations` caps the prompt's value at `tool_iterations_ceiling`.
2. Parse error kinds. Chapter 16 teaches six kinds, such as `Frontmatter` and `Structure`, and says "The set of parse error kinds is open". The code has one closed enum, `ErrorCode` in `runvil-internal/types/src/error.rs`, and `Prompt::parse` returns one of six codes, such as `frontmatter_invalid` or `lua_compile`.
3. `sys.execution`. Chapter 5 lists `sys.execution` among six fields of every section entry. `SectionSys` in `runvil-internal/runtime/src/execute/section_context.rs` has `when`, `id`, `taskid`, `section_name`, and `section_count`, plus the optional `index` and `model`.

## 11. Questions for the owner

Each question asks for the reason behind a part of the design.

**The run**

1. Why do the blocks under the H1 title run once for a whole run?
2. Why does `Run::new` always succeed, while the first `step` makes the start checks?
3. Why is the cancel a flag in `RunParams`, where `Run` could have a method for it?
4. Why does a prompt that declines its task complete, while an unmet requirement ends the run with an error?

**Lua and limits**

5. Why does each Lua block run as its own coroutine, when one coroutine could run a whole section?
6. Why is `models.loop` the one suspending call whose rules run in Rust?
7. Why is the call depth cap 8, and why is `call_depth_exceeded` in the Lua family of codes?
8. Why are the defaults 8 concurrent tasks, 64 MiB of Lua memory, and 1024 log events, and why does the cancel hook fire every 10,000 instructions?

**Errors and the surface**

9. Why do all Engine and Harness crates report failures as one `Error` type?
10. Why are `Error`, `ErrorCode`, and `ErrorFact` closed, when a new code then breaks the build of a Host whose `match` on `ErrorCode` names every variant?

## 12. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The text of the page still describes the code at the commits above, when those crates were in `runvil`.

*2026-11-10 16:37 - Claude Fable 5.2*
