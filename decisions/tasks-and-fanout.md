# Tasks and fanout: design decisions and their reasons

- Scope: concurrent work inside a run: tasks, fanout, the scheduler's order, limits, determinism, and a task's inbox
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the Paths bullet names the repositories as of `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 10)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil`

## Summary

A chain is one walk through a prompt's sections. A run has a chain for the main walk, one for each `call`, and one for each task, and a task is a chain that runs beside the chain that spawned it. The Engine's scheduler runs the chains in turns on one thread. `tasks.spawn` starts a task, `tasks.join_any` waits for one, and `tasks.fanout` is a Lua function that spawns one task for each member of a table. Sections 1 to 6 describe this design.

A check of this page against the code found two gaps and left five questions about reasons for the owner. Section 8 describes the gaps, and section 9 lists the questions. The larger gap is in the language guide: chapter 15 lists the execution among the fields of the `TaskStarted` event, and the event in the code has the fields `section`, `provenance`, `task`, `target`, `input`, `item`, `index`, and `var`.

Section 7 lists the planned work. Its largest item is replay: the Harness driver that runs a recorded run again from its seed, start time, and answers is still to be built.

## From a `tasks.fanout` call to its results

A section's Lua calls `tasks.fanout('### Worker', items)`, a Lua function in `__impl_fanout.lua` that the code's comments call the `fanout` shim. The shim calls `collection_members` to list the members of `items`, copies the caller's `var`, and yields one `spawn` request for each member.

For each request, the scheduler's `dispatch_spawn` checks the depth cap, forks a file identity from the caller's (section 5), and assigns the next child id, such as `0.0`. It adds a `TaskEntry` to the `TaskArena`, queues the new chain in `Admission`, and resumes the shim.

The shim then yields a `join_any` request for all the tasks it spawned, which are the fanout's arms. `drain`, the scheduler's loop over the ready queue, finds the queue empty and calls `admit`, which starts queued chains in spawn order while slots are free. The scheduler emits `TaskStarted` for each arm, and the arm runs the worker section with its member as `item`. When an arm's Lua yields a model call, the scheduler queues the call as an effect for `Run::step` to return and runs the next arm.

When an arm's chain ends, `complete_task` stores the outcome in the arm's entry and emits `TaskSucceeded` or `TaskFailed`. `deliver` then joins the arm's file identity into the caller's and takes the outcome, and the scheduler wakes the caller with it. The shim stores each result at its member's index and returns the results after the last arm.

## 1. Chains and the scheduler

### Chains take turns on one thread, first in, first out

The scheduler resumes one chain's Lua coroutine at a time. The chain runs until its Lua yields a request, such as a model call, a store call, or a wait. `drain` then pops the next chain from the `ready` queue. A request for outside work becomes an effect that `Run::step` returns ([The Engine](engine.md)), and the scheduler itself answers a request about another chain. A long Lua loop between two requests therefore blocks every other chain.

One thread is enough to dispatch every chain, because a fanout spends most of its time waiting for inference.

### `TaskArena` owns the task records, and `Admission` owns the queues and the slots

`TaskArena` is a map from task id to `TaskEntry`. It checks ownership and enforces the task lifecycle. Its entries are permanent, so a late `tasks.status` still reports how a task ended. `Admission` owns the queues of waiting chains and the set of chains that have slots (section 4). The methods that write to several owners are `Scheduler` methods, such as `finish`, `abort_subtree`, and `cancel_task`, and they report a subtree's events before its owner's terminal event.

They are methods of `Scheduler` because the owners depend on each other through the order of these writes. The `ready` queue is a field of `Scheduler` itself, because first in, first out is its only rule. [The Engine](engine.md) describes the scheduler's other owners.

## 2. Tasks

### `tasks.spawn` starts a chain over a section and returns a handle at once

`tasks.spawn(target, opts)` queues a new chain that starts at the target section, and returns before that chain runs. The handle is a plain table, `{ task = id }`, and every `tasks` function that names a task takes the handle or the id string. The `tasks` table has thirteen functions: `spawn`, `join_any`, `join`, `ready`, `status`, `pending`, `note`, `cancel`, `concurrency`, `receive`, `send`, `reply`, and `fanout`.

### The chain that spawns a task owns it

A task belongs to the chain that spawned it. A `jump` or a fall-through to the next section leaves the task running, and the tasks of the H1 pass, which runs the Lua under the prompt's title, become the main walk's. Only the owner may wait on, send to, or cancel the task, and any other chain receives a `task_not_owned` error. A chain may also call `tasks.status` and `tasks.note` for the task it runs inside, whose id is `sys.taskid`.

A task is bound to the lifetime of `var`, the table that the sections of one chain share. Authors know that rule from `var`, and it matches `ChainArena`, the scheduler's record of chains, exactly.

### `tasks.join_any` is the one wait in the Engine, and `tasks.join` is Lua over it

`tasks.join_any(set)` suspends the caller until one task in the set ends or replies. `tasks.join(set)` is a Lua loop over `join_any` that returns one entry for each member. A wait takes each result once, and a second wait for the same result raises `task_consumed`. `opts.timeout`, in seconds, starts a timer that the wait cancels itself.

The names state the contract: after a join, the task's writes are visible to the caller. Both waits return a member's failure as data. A `tasks.join` that raised on a member's failure would have to cancel the other members or leak them, and each choice is wrong for half the callers.

### A chain that ends with a running task fails with `tasks_live`

When a chain ends normally while a task it spawned is still running, the chain's result becomes a `tasks_live` error that names the task. The Engine ends the task with the state `abandoned`. A task that its owner stops with `tasks.cancel` has the state `canceled`. When the owner ends, the Engine also abandons an idle task, one that waits after `tasks.reply`, and leaves the owner's result unchanged.

The owner's rule: "subagents spawned by Lua become a hard error on chain termination". A task left running at its owner's end is a bug in the prompt. An idle task is exempt from the error, and guide chapter 15 states the reason: "It already handed its owner a reply, so an owner may end while it has idle tasks". The two end states are separate because a task that lost its owner and a task that someone stopped are different facts for the log and the UI.

### Every started task reports one start event and one terminal event

The scheduler emits `TaskStarted` at admission, under the section that spawned the task. One terminal event follows: `TaskSucceeded`, `TaskFailed`, `TaskCanceled`, or `TaskAbandoned` with an `AbandonReason`. The scheduler skips both events for a task that ends while still queued.

When a run ends as a whole, by a cancel or a fatal answer, `Scheduler::end` first abandons every live task. The reason is `run_terminated`, or `owner_aborted` for a task nested under another. If `Scheduler::end` skipped this step, `TaskStarted` would be the last event of such a task, and its record would read `running` forever.

## 3. Fanout

### `tasks.fanout` is a Lua function over `spawn`, `join_any`, and `cancel`

`tasks.fanout(worker, collection)` runs the worker section once for each member of the collection, and each run is an arm. The function is the Lua in `__impl_fanout.lua`. It yields the same requests that `tasks.spawn`, `tasks.join_any`, and `tasks.cancel` yield, so every concurrency rule in the Engine is a rule about tasks.

### The collection is a Lua table, and the shim lists its members in a fixed order

`collection_members` lists the array part in index order and then the keyed part sorted by key. A keyed member becomes a pair table that the arm reads as `item.key` and `item.value`. Any other value in place of the table raises an error that points to `prompt.list(heading)`.

A heading in place of the table is one of the refused values. `prompt.list(heading)` returns the items of a list section, so a heading as the argument would be a second way to do one thing, and a bare string is ambiguous against a section name. The sort of the keyed part makes every process spawn the arms in the same order.

### Each arm is a task that runs the worker section with `item` and `sys.index`

An arm's chain starts at the worker section in a fresh Lua VM. The member is the global `item` and the value of `{{ item }}` in prose, its 1-based position is `sys.index`, and `var` is a copy of the caller's. The worker section can do what any section does, including `call`, `jump`, and a nested `tasks.fanout`.

An arm is normal flow: its chain follows the rules of every other chain.

### The shim spawns every arm before it waits for any

The shim yields a `spawn` request for every member before its first `join_any`.

Under the join rule of section 5, an arm spawned after a join would be ordered after the arm that the join returned and could read that arm's files, so its result would depend on timing. With every arm spawned before the first join, the arms are unordered among themselves.

### Results return in collection order as objects with four fields

`tasks.fanout` returns an array with one result at each member's position. `text` is the arm's final text, `ok` says whether the arm completed, `item` is the member, and `exhausted` says whether the arm ran out of model rounds. An assignment to a result raises an error that reads "tasks.fanout results are read-only".

The caller combines the results itself. `item` lets the caller match each result with its member when the member is a rich value, such as a record.

### The first failing arm cancels the others, and an exhausted arm becomes a stub

When an arm fails, the shim cancels the arms still live and raises that arm's own error at the `tasks.fanout` call. An arm that fails with `tool_loop_exhausted` is the exception: the shim fills its position with a stub text, `ok` false, and `exhausted` true, and the other arms continue, so that one stuck arm leaves the evidence of its siblings intact. An empty collection raises an error before any spawn, and the error's text states the reason: "tasks.fanout over an empty collection: no work is likely a bug".

### The model-first form runs every arm on one model

`tasks.fanout(model, worker, collection)` takes a model handle first. The shim sets the caller's current model to that handle, spawns the arms, and restores the caller's model before the first wait. Each arm copies the model at its spawn ([Model calls and conversations](models-and-conversations.md)).

The form exists only in the shim: both forms of the call yield the same requests to the scheduler. It spares authors a manual save and restore of the model.

## 4. Admission and limits

### One limit of 8 bounds the tasks that run at once in the whole run

`RunLimits::max_concurrency` is the most tasks the scheduler admits at once. The default is 8, and the Host sets it. A running task counts against the limit of its owner and of every ancestor chain, and each such count is a slot. A fanout inside an arm therefore runs within that arm's share.

One limit covers the whole run. A limit inside each fanout would multiply when fanouts nest, since 8 arms that each run 8 arms are 64 live tasks, and it would leave tasks from `tasks.spawn` unbounded. The Harness has a separate limit of 10 Pack tool calls in flight ([Tools and Packs](tools-and-packs.md)), because one shared limit would couple the task limit and the call limit.

### A spawned task starts at admission, in spawn order, after the chains that resume

A spawn forks the file identity, assigns the task id, and queues the chain. Admission is the moment the scheduler starts a queued chain. The chain gets its Lua VM then, and until then `tasks.status` reports `blocked` as `queued`. `drain` runs an admission pass each time the `ready` queue is empty. The pass admits chains that resume from a wait first, then new spawns in spawn order.

A queued task costs very little, so the arms of a fanout over 500 members have at most as many Lua VMs at one time as the limit allows. The pass admits resumed chains first, which protects a resume from starving behind a long queue of spawns. Admission order changes timing only: the ids and the file system's decisions on conflicts (section 5) are the same under every admission order. A collection may be as large as the author likes, because a large collection only takes longer, and whether that is worth it is the author's call.

### A task that waits on tasks or on its inbox releases its slots

A task that waits in `tasks.join_any` or `tasks.join` releases its slot at its owner and at every ancestor, and takes the slots again before it resumes. A task that waits on its inbox does the same. A task that waits on a model call or a tool call still counts against the limit.

The release prevents a deadlock: 8 outer arms that wait on their own inner arms would otherwise fill every slot.

### `tasks.concurrency` sets a chain's limit and clamps it to the parent's

`tasks.concurrency(n)` sets the limit for the tasks the calling chain spawns from then on and returns the limit in effect. The scheduler clamps `n` to the limit of the enclosing chain, or to `max_concurrency` on the main walk.

The scheduler clamps a value above the limit, where it could raise an error, so that a prompt is portable wherever it runs, whatever the limit. The function is named after tasks because tasks are what it limits.

### `call`, `tasks.spawn`, and fanout arms share one depth cap of 8

A `call` child and a spawned task each run one level deeper than the chain that started them, and `MAX_CALL_DEPTH` is 8. The start that would run at level 9 raises an error with the code `call_depth_exceeded` and the text "recursion exceeded cap of 8". `make_call_depth_exceeded_error` in `runvil-internal/runtime/src/error.rs` formats "recursion exceeded cap of {cap}".

A fanout arm counts as a level, so the depth of a recursion accumulates across a fanout, where it would otherwise start again in each arm.

## 5. Determinism

### The same seed, start time, and answers produce the same run

For each run the Harness generates a random seed, reads the clock, and passes both to the Engine in `RunParams`. With the same two values and the same answers in the same order, the Engine emits the same effects, events, and ids. The order of answers is an input. Among waiting tool calls, the Harness starts the one with the lowest effect id ([Tools and Packs](tools-and-packs.md)), and `Run` lacks a call that reports a start. That choice therefore affects the Engine only through the order of answers, so a replay (section 7) is independent of the Harness's start order. Three results depend on the order of answers: which task `tasks.join_any` returns, whether a timed wait runs out, and how far a canceled task got. The guide adds a fourth: where a `tasks.send` message applies in a task's work.

Because of this rule, replay is a job for the Harness: the Engine is deterministic, so the Harness can match a recorded run exactly. The rule covers Lua's table walks too. The Engine's `pairs` and `next` walk a table in sorted order, where the order of stock Lua's walk depends on a hash seed.

### Chain ids, task ids, and `sys.id` are dot-separated paths

A chain's id is its parent's id followed by a number from the parent's own child counter. The main walk is `0`, its first child is `0.0`, and a task's id is the id of its chain. `sys.id` is the chain's id followed by the chain's entry counter, so the first walked section reads `0.1`. Each counter belongs to one chain, so the ids are the same in every run, whatever order the chains finish in.

An id is a path of numbers, and paths can name any number of arms. The number of a fanout's arms is unbounded, so a packing of the numbers into one fixed-width integer would have an overflow rule to document and test.

### A spawn is a fork, and every delivery is a join

Each task has a file identity, an `ExecId`, under which the file system records its reads and writes. A spawn orders everything the parent did before it ahead of the child's first step. A delivery orders everything the task did ahead of the owner's next step: `join_any` joins the task it returns, `join` joins each member, and a chain's end joins every task that the chain owns. The file system uses this order to decide whether two accesses conflict ([Files and the VFS](files-and-the-vfs.md)).

The result is one rule: after a join, a chain can read what the task wrote. A wait that returned a task unjoined would be a second concept, and a trap: a caller could take a task's result and leave the task unjoined. Determinism here means that the file system's decisions on conflicts and the contents of the files depend only on program order and on explicit points of synchronization, and are independent of scheduling.

## 6. Inboxes

### Every task has an inbox, and `tasks.receive()` reads it

An inbox is a queue of text messages, oldest first. The Engine has one for the main chain, which is the H1 pass and the main walk, and one for each task. `tasks.receive()` returns the oldest message, or suspends the chain until a message is queued. A `call` child reads the inbox of the task it runs inside. A chain inside `models.loop` reads its messages before its next model request ([Model calls and conversations](models-and-conversations.md)).

The inbox functions are members of `tasks` for two reasons: `tasks.send` is there, and `tasks.note` is a member that also acts on the caller's own task.

### The Host queues messages for the main chain, and a task's owner queues messages for the task

`Run::deliver` queues a message for the main chain, and the Harness calls it for each message the Host sends ([The Harness and the Host API](harness-and-host.md)). `tasks.send(task, text)` queues a message for a task that the caller spawned. It returns true, or false when the task has ended, and it raises `task_not_owned` for any other task.

The Host's `RunControl` sends only to the main chain: a sub-agent is a task in the same run, and only a run has a `RunControl`. When every chain that remains waits on a task or on a task's inbox, the run ends with the error `stalled` ([The Engine](engine.md)).

### `tasks.reply` publishes a result, and the task then waits for its owner's next message

`tasks.reply(value)` publishes `value` as the task's latest result and then waits on the task's inbox. The task is then idle: its Lua state and its conversation are intact, `tasks.status` reports `blocked` as `inbox`, and the owner's next `tasks.send` resumes it.

A task can therefore return a result and remain open for follow-ups with its conversation intact, up to the cap on idle tasks in the next entry. The fanout shim cancels each arm after it takes the arm's result, so an arm that replied ends as `canceled`. The shim cancels the arm whether it is live or has ended, since a cancel of an ended task is free and silent.

### A run has at most 8 idle tasks, and one more reply closes the one idle longest

`RunLimits::max_idle_tasks` is 8 by default. A reply that makes a ninth idle task closes another: the one idle longest among those whose own tasks have all ended, or the one idle longest of all. The closed task ends as `done` with its latest reply as its result.

The cap bounds memory: an idle task still has its Lua VM, which the Engine is unable to rebuild from the task's conversation. The closed task ends as `done` because its owner is still alive and can still take that reply, while `abandoned` means that the owner ended.

## 7. Planned work

Each of these items is still to be built.

- **Replay and resume.** A Harness driver that builds `RunParams` from a recorded run's seed and start time and passes the recorded answers to `Run::resume`, and a way to spawn a canceled task again under its original id. The Harness builds `RunParams` for every run from a new seed and the clock, and a spawn assigns the next child id.
- **A read of task history.** A Lua call that returns the events that a task has reported. The design waits for a concrete use. A prompt reads a task's state at the time of the call with `tasks.status`.
- **Task tools for models.** Tools with which a model starts tasks itself, as the tools of the Pack `runvil/subagents` ([Tools and Packs](tools-and-packs.md)). The Pack reserves the name, and only Lua starts a task, with `tasks.spawn` or `tasks.fanout`.
- **Inbox messages other than text.** Images, files, audio, and video as inbox messages. An inbox message is a text string.

## 8. Known gaps

### The guide lists `execution` among the fields of `TaskStarted`, and the code omits it

Guide chapter 15, in its section on the `TaskStarted` event, says the event includes "the execution, the section, the provenance, the task id", the target, and the four spawn seeds. The `events!` macro in `runvil-internal/types/src/event.rs` adds `section` and `provenance` to every variant, and `TaskStarted` declares `task`, `target`, `input`, `item`, `index`, and `var`. The test `no_serialized_event_has_an_execution_key` in `event-tests.rs` asserts that every serialized event lacks an `execution` key. [The Harness and the Host API](harness-and-host.md) lists the other places where the guide names an execution.

### A doc comment in `ids.rs` calls a run's depth unbounded, and the code caps it at 8

The doc comment at the top of `runvil-internal/types/src/ids.rs` says that a path is "chosen over a packed integer" because the depth and the width of a run "are both unbounded (call nesting, fanout arm count)". `MAX_CALL_DEPTH` in `runvil-internal/runtime/src/execute/scheduler/chain_record.rs` is 8, and a `call` or a spawn that would run at level 9 raises `call_depth_exceeded` (section 4).

## 9. Questions for the owner

Each question asks for the reason behind a part of the design.

**Fanout**

1. Why does the first failing arm cancel the others?
2. Why does a result have these four fields, and why is `exhausted` separate from `ok`?

**Tasks**

3. Why does every task have an inbox?
4. Why is the model-first form limited to `tasks.fanout`?
5. Why is 8 the value of `max_concurrency`, of `max_idle_tasks`, and of the depth cap?

## 10. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The text of the page still describes the code at the commits above, when those crates were in `runvil`.

*2026-11-10 16:38 - Claude Fable 5.2*
