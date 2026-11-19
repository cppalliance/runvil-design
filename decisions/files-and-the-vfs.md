# Files and the VFS: design decisions and their reasons

- Scope: the `store` Lua API, the virtual file system (VFS), real and virtual roots, what a run may read and write, file identities and conflicts, and persistence
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the Paths bullet names the repositories as of `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 11)
- Update: the first entry of section 1 states the rule of the second change that follows `8de43c2f` in `runvil-harness`: every heading starts a section, and the code before the first heading is the preamble (section 11)
- Update: the text on the list of file operations, on a tool's file requests, on conflicts and claims, on policies and the operation sink, on the `Vfs` effect and its record, and on Workshop's demo mounts, with the planned work, the known gaps, and the questions, describes `runvil-harness` at commit `ee199e3a`, `runvil-workshop` at `956e7800`, `runvil-plugin-api` at `52a4905`, `runvil-plugin-std` at `a806bf6`, and `runvil-plugin-bashkit` at `f348f7a` (section 11)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil-workshop`

## Summary

A run's files are in a virtual file system, the VFS, which the crate `runvil-vfs` implements. The Host builds one `VfsRef` handle for each run: a table of mounts, each a backend in memory or over a real directory, which holds the files the run's tools work in. The store is a separate value, a `Store`, that the Engine makes for each run, and each `execute` gives the prompt that it runs a store of its own. Lua reads and writes a store through the `store` table. The main walk, each task, and each tool call has an `Access`, an object that performs file operations for one identity: it checks the path and the policy of its handle, and then calls the backend. A `store` call runs on a view of the store, an `Access` with the calling chain's identity that reaches the store alone and asks no Host policy. A view also checks what the other identities have read and written in its store, which an `Access` to the Host's files leaves unchecked. A tool holds no `Access`: it asks for each file operation, the Engine issues the operation as a `Vfs` effect, as it does for a `store` call, and the Harness performs it on the `Access` of the tool call and records it. Sections 1 to 7 describe this design.

A check of this page against the code found five gaps (section 9) and seven questions about reasons (section 10). The largest gap is a lost edit: a tool that edits a file as a read followed by a write can lose a change that another task makes between the two, because the Host's files have no conflict check.

Section 8 lists the planned work: a real directory in Workshop, a database mount, replay, and a glob across mounts.

## From `store.write` to the backend and back

A Lua block calls `store.write('notes.md', text)`. The function is a shim that yields a table, which the Engine parses into a `Request::Store` with a `VfsOp::Write`. The Engine derives a store view from the run's `Store` under the `Access` of the calling chain, such as the main walk or a task. The view is a second `Access` with the chain's identity and scope, and it reaches the run's store alone. The Engine issues `Effect::Vfs` with its three fields: `asker` is a `VfsAsker::Lua` that holds the id of the store, `access` is the view, and `op` is the operation. `Run::step` returns the effect, and the chain waits.

The Harness records the effect and calls `perform_vfs_op`, which runs the operation on the view with `Access::perform`. A `VfsOp::Write` runs as `Access::write`.

`Access::write` checks `notes.md` against the store's path rules and canonicalizes it to `/notes.md` inside the store. Its policy check goes to the store's own policy, `AllowAll`, which allows every operation, so no Host policy is asked. It checks the store's own claims table for a conflicting access by another identity (section 5) and records a write claim there. The view then calls the store's backend, a `MemoryBackend`, which stores the bytes. The Harness records the answer and resumes the run. The Engine reports the event `VfsWriteSucceeded`, and `store.write` returns `nil`.

A later section of the same walk calls `store.read('notes.md')`, which runs the same steps with a `VfsOp::Read` and a read claim. The walk has one identity for all its sections, so the claim check passes, and Lua receives the text. Had a task written `notes.md` in between, before the walk joined it, the read would fail with `VfsError::Conflict`, and the Engine would end the run with the error code `determinism`.

A tool's write of a file takes the same path from the effect on. A Rust Pack's tool calls `request` on the files of its call with a `VfsOp::Write`, and the Harness reports the request to the Engine with `Run::request_file`. The Engine issues `Effect::Vfs` with the asker `VfsAsker::Tool`, which names the tool call, and with the call's own `Access` to the run's handle (section 5). On that `Access` the path is rooted at `/` of the handle, the policy is the Host's, and the operation makes no claim. The Harness performs the write, answers the Engine with a receipt, `VfsReceipt::Done`, and hands the outcome to the tool. Had a task written the same file a moment earlier, both writes would succeed, and the later one would be the content.

## 1. The `store` table

### Every Lua block has a `store` table over the files that a prompt's sections and tasks share

The Engine installs a `store` table in the Lua VM of every section entry. The preamble, the Lua before a prompt's first heading, is a section entry like the others ([The prompt language](prompt-language.md)). The table's functions read and write the store, a set of text files that all the sections and tasks of one prompt share while it runs. The run's own prompt has the store that the run starts with, and a prompt that an `execute` runs has a new store for that execution (section 3). A tool call works in the files of the run's handle and sees no file of a store (section 3).

The owner's reason: "Store is always present, its a core feature of the Runvil language because its the way to move data between sections and handle fanout."

The store holds files because a prompt is often a pipeline: a stage runs inference over the files of the stage before it and writes new files. The owner's description: "there's files created at every stage of the pipeline". The files stay for the whole run so that a person can follow the data from stage to stage. The owner's reason: "we want those files to persist during the run so that they can be inspected". The owner's convention for a prompt author is to write a new file at each stage and leave the earlier one as it is: "You want to create a new one from an old one, and this way you can inspect the data flow." The store leaves the convention to the author: `write` replaces a file, and `append` and `str_replace` change one. The files are text because, in the owner's words, "Markdown is the language of AI". State has to live outside Lua, because the Engine discards a section's VM when the section ends, and `var` passes values to the next section in a form that converts as JSON ([The prompt language](prompt-language.md)). The owner's reason for files beside `var`: "JSON's unsatisfying because it's not Markdown".

The inspection that the owner means happens during the run, at a breakpoint or at any stage. With the evidence in file A and the inference over it in file B, "if we get a breakpoint now we can see A and B", where a B written as a series of edits to A would leave only the event log to show the earlier text. The owner's reason: "Having it in the store means there's a blob there." A debugger that rebuilds each store from the run's records is planned work (section 8), and the record is the only view of a store from outside the run (section 7).

### The `store` table has eight functions, and a prompt composes the rest

The functions are `write`, `append`, `read`, `read_numbered`, `str_replace`, `delete`, `glob`, and `exists`. `store.str_replace` replaces an anchor text that must occur exactly once, because an edit by anchor is the pattern that works for models. `store.delete` succeeds on an absent path.

The table stops at eight because a prompt can write the other VFS operations from them. It leaves out `stat` because timestamps are nondeterministic, and the store's memory backend tracks none.

### `store.read` takes 1-based inclusive line bounds, and `store.read_numbered` numbers the lines

`store.read(path, start, end)` returns lines `start` to `end`, and `store.read(path)` returns the whole file. An `end` past the last line means the last line, and a `start` below 1 raises `invalid_range`. `store.read_numbered` takes the same bounds and prefixes each line with its number in the file, as in `12| text`.

The second bound is an end line because a count would force `end - start + 1` arithmetic at every call site, a documented failure mode of small models.

### The store view accepts relative, nested paths and checks each against strict rules

A store path is relative to the store root, as in `drafts/plan.md`, and a write creates the missing parent directories. The store view checks each path first. A path has 1 to 1024 bytes. The view refuses a leading `/`, a backslash, a colon, a `.` or `..` segment, a name that ends in a dot or a space, and a Windows device name such as `COM0`. A refused path raises `invalid_path`, and `err.rule` names the broken rule.

The backslash, colon, suffix, and device-name rules refuse spellings that some backend would read as a different file.

## 2. The virtual file system

### The file system types are Runvil's own, and a run's tool files and its store are separate values built from them

`runvil-vfs` defines the file system types. A backend implements two traits: `Vfs` for the storage and `VfsAccess` for one identity's session with it. `VfsRef` is a cloneable handle to a set of mounted backends, and an `Access` performs operations through it for one identity. A Host builds a `VfsRef` from these types for the files that a run's tools work in (section 3). A `Store` is a value of its own, built from the same types: an empty set of files in memory under a claims table of its own, outside every `VfsRef`.

A store is built from the file system types because it keeps their claims rules (section 5). The owner's reason: "We need the claim system that the VFS has."

The `vfs` module of each facade crate, `runvil` and `runvil-engine`, exports the public items of `runvil-vfs` except `Store`, so a Host can build the file system that its runs' tools work in and can add a backend of its own. `runvil-vfs` alone exports `Store`, because holding a `Store` is the whole authority over it (section 3).

### A store call is one `Access` call on a view of the calling chain's `Store`

For each `store` call the Engine derives a store view with `Store::view`, from the calling chain's `Store` and the calling chain's `Access`. `Store::view` is the only way into a store. The view is a second `Access` with the chain's identity and scope, and its operations reach that store alone. It checks each path against the store's path rules (section 1) and gives error paths back in the form the caller wrote. `dispatch_vfs` in `runvil-internal/runtime/src/execute/scheduler/dispatch.rs` is the one place in the Engine that makes a store view.

The store follows one rule: it offers a subset of the operations of the VFS. A store is a value and not a mount of a handle, because one scope has to hold many stores. A child prompt that `execute` runs gets a store of its own, and it runs in its caller's scope under its caller's identity (section 3). The owner's rule: "prompts passed to execute or execute_async should get their own store which is completely isolated from the caller's store. they cannot see each others' paths or claims."

### One list of operations serves the `store` functions and a tool's file requests

`runvil-vfs` defines the file operations as values, in `runvil-internal/vfs/src/vfs_op.rs`. A `VfsOp` is one operation with its arguments, `Access::perform` runs it on an `Access`, and a `VfsOutcome` is what it returns. The list has 15 operations. The `store` functions use eight of them, `Write`, `Append`, `Read`, `ReadNumbered`, `StrReplace`, `Delete`, `Glob`, and `Exists`, and each `store` call is one of them. A tool asks for nine through the file functions of the C contract: `Stat`, `List`, `ReadRange`, `Mkdir`, `Remove`, `Rename`, and `Copy`, beside `Write` and `Append` ([Tools and Packs](tools-and-packs.md)). A Rust Pack's tool may ask for any operation of the list. `Write` and `Append` hold bytes, because a tool writes bytes, and a `store` call passes the bytes of its text. `Delete` succeeds on an absent path, and `Remove` fails on one with `VfsError::NotFound`. They are two operations, so a run's record shows which one ran. `VfsOutcome` has seven variants: `Unit`, `Text`, `Paths`, `Bool`, `Stat`, `Entries`, and `Bytes`. `VfsOp` and `VfsOutcome` are closed enums, so an operation or an outcome that the list gains fails the build at each place that must handle it ([The Engine](engine.md)).

The owner's reason for one list: "in principle, the store is just a virtual directory in the file system that is unique to a prompt." A `store` call and a tool's file request are two views of the same operations, and the Engine and the Harness handle the whole list, so either side gains an operation by adding one function. Each of the two reaches the list as a `Vfs` effect, which says who asked (section 7).

### Virtual paths have one POSIX form on every operating system

A virtual path is rooted and uses forward slashes on every operating system. An `Access` canonicalizes each path when it receives it, so the policy, the claims table, and the backend see one spelling. A path without a leading `/` is relative to `/`.

With one canonical form, only `RealBackend` translates a path to the operating system's form. Case is significant. A claims table has one exception: below a rooted real mount on Windows and macOS it compares paths in lowercase, because a false conflict fails safe. A store is memory, and a handle that a Host builds checks no claims (section 5), so the exception is out of the reach of every run.

### `VfsRefBuilder::build` fixes a handle's mount table, and the longest matching prefix wins

`VfsRefBuilder::mount` installs a backend at a path prefix, and `build` returns the handle. The handle's router sends each operation to the mount with the longest matching prefix. `VfsRef::overlay` returns a second handle with one more mount above the first, under the first handle's policy and operation sink.

A table fixed at construction makes routing immutable and a handle cheap to clone for each run.

### The backend traits are synchronous and work on bytes

Every method of `Vfs` and `VfsAccess` is a blocking call, file content is bytes, and `Access` adds the text operations. A `VfsRef` locks its backend for each call, so a backend must be `Send` and may leave locking to the handle.

The traits are synchronous because the backends are synchronous by nature: memory and `std::fs`. Lua is synchronous as well, and that is beside the point. The owner's reason: "at the end of the day, the operating system calls are synchronous. So we keep the storage operations synchronous." `Access` is synchronous for the same reason.

## 3. Real and virtual roots

### The Host builds the file system that a run's tools work in

A Host sets `RunSetup.vfs` to one `VfsRef` for each run: the file system that the run's tool calls work in ([The Harness and the Host API](harness-and-host.md)). The Harness passes the handle to the Engine with `RunParams::with_vfs`, and the Engine makes an `Access` to it for each tool call (section 5). `RunSetup::new` sets `VfsRef::default()`, one empty in-memory mount at `/`. A Host can fill the handle before a run and read it after the run, through its own clone of the `VfsRef`, and concurrent runs can share one handle. When the handle refuses the run its access, or refuses the fork for a task or a tool call, the failure has the handle's own error code and the message `file access failed: {cause}`.

The Host builds the handle because the mounts are the Host's choice: memory, a real directory, or a backend of its own. The handle holds tool files alone, and the Engine and the Harness put nothing into it.

### The Engine makes each run an empty store, which only the run reaches

The Engine makes a run's store with `Store::new()` in `ChainState::root`, in `runvil-internal/runtime/src/execute/context.rs`. A new store is empty. Every chain that runs the run's own prompt shares the one store: the main walk, a `call` chain, a task, and a fanout arm. The Engine never hands the `Store` out: a caller reaches the store only through the view in a `Vfs` effect, for that effect's one operation (section 7). Nothing seeds the store before the run, and nothing reads it afterward. Data goes into a run as `args` and comes out as the result ([The Harness and the Host API](harness-and-host.md)). The store and the handle are disjoint: a tool sees no file of the store, and a `store` call sees no file of the handle.

The owner's reason: "Every prompt starts with a, a memory store that's completely empty. And that's the only type of store there is. And it doesn't map to any file system path in the virtual file system." With no way in from outside, a run's result depends only on its prompt, its arguments, and the answers to its effects.

The owner holds the two halves of this rule with different confidence. Data enters a run as its arguments and leaves as its result, and the owner is sure of that: "we don't want to use the store as a way of getting data in and out of the prompt from the host", and "Using the argument list and using the return value instead of the input and output files, that was definitely the right move." Keeping the store out of reach of a model's file tools is less certain. The owner made that choice for `execute`: "The problem with execute is that the prompt that we're calling needs to map itself into the virtual file system, and I couldn't think of a place to put it." The owner's view of the choice: "it was a big simplification that was useful for execute. I'm not sure that it was the right one." A prompt can give a model access to its store with an adhoc tool whose handler calls a `store` function ([Tools and Packs](tools-and-packs.md)). The rule that a tool sees no file of the store still holds there: the handler is the prompt's own Lua and makes an ordinary `store` call on its chain's store, where the file requests of a Pack tool reach the handle alone. The owner rejects that as the standing way for a model to reach the store, and rejects a second set of tools for it: "There's no reason to make a separate set of store tools. That just burdens the model." The owner's rule is that a model that reaches the store uses the file tools it already has, because "We want as few tools as possible" and "Models already know how to use Bash." That would put the store inside the file system that a tool sees, which the design in this section rules out, so it is a change that is still to be designed.

### Each `execute` gives the prompt it runs a new, empty store

`ChainState::with_prompt`, in the same file, makes a store with `Store::new()` for the chain of each `execute` ([The prompt language](prompt-language.md)). The store is empty and has a claims table of its own. The executed prompt's `call` chains and tasks share it, and a prompt that the child executes in turn gets another. The caller and the child see none of each other's store files: one path in the two stores is two files, and the caller finds no file of the child after the `execute` returns. A second `execute` of the same prompt starts with an empty store again.

A child's store is gone when the child ends. Each chain releases its hold on its store when it finishes or is aborted, and a store is freed once the last chain that shared it has ended.

The child's tool calls are a different matter: they work in the files of the run's handle under the caller's file access, as the caller's own tool calls do, because the executed prompt is a part of the caller's run.

The rule is the owner's (section 2). An executed prompt therefore has the one way in and the one way out that a run has: its arguments and its text. Two fanout arms that execute one prompt can each write the same path with no conflict, because each execution has its own store and its own claims.

### `RealBackend` maps a real directory into the namespace and confines paths and links to it

`RealBackend::rooted(dir)` maps the virtual root to one real directory and refuses a path that resolves outside it with `OutsideRoot`. `RealBackend::identity()` maps each virtual path to the same real path. `remove`, `exists`, `stat`, `mkdir`, and `rename` act on a link itself, and the content operations follow a link whose target is inside the root. `with_writability` makes a backend read-only. A failed write leaves the file unchanged, because the backend writes a temporary sibling file and renames it. The policy of the handle is asked about the path that an operation names, a link's own path included, and never about the link's target (section 4).

The five operations act on the link itself because a `remove` that followed the link would delete the link's target. `RealBackend` is part of `runvil-vfs` itself: `std::fs` belongs to the standard library, so the backend fits the crate's zero-dependency rule ([Crates, build, and CI](crates-build-and-ci.md)).

### A mount is for storage that answers quickly, and a remote system is a tool's job

A Host may mount any backend, a slow one included. Every file operation holds its run until the backend returns, so a run that works on a network drive, or in a folder that a cloud service syncs, waits for that storage, and its stop and its cancel wait with it. A feature that needs a timeout, a retry, a sign-in, or a rule for storage that goes away is a Pack's tool, which the Harness can drop while it waits ([Tools and Packs](tools-and-packs.md)).

The owner's rule for slow storage: "we should allow it, but if the performance suffers, then it suffers." For a remote system: "If you want to access those remote systems, then you need to do it through MCP". The tool then decides, in the owner's words, "what do you do if the volume goes offline, like in the middle of it".

## 4. What a run may read and write

### Each handle has one policy, which answers Allow, Deny, or Ask

An `Access` asks the handle's `Policy` about an operation path by path, reads included: about a path once that path is valid, and before the operation is admitted. The first check that fails and the first question that the policy refuses end the operation, so the policy is asked nothing that would come after either. `Policy::check` returns `Allow`, `Deny(String)`, or `Ask(String)`. The file system turns `Deny` into `PermissionDenied` and `Ask` into `ApprovalRequired`. The default policy is `AllowAll`. A store is no part of the handle and has `AllowAll` as its own policy, so the handle's policy sees the file operations of tool calls and never a `store` call.

A copy and a rename are asked as what they are made of. Each asks about its source and then about its destination under its own kind, `Op::Copy` or `Op::Rename`, and then about its source as `Op::Read` and about its destination as `Op::Write`. The first refusal ends the operation, and nothing is copied or moved. A policy that refuses reads under a path is the plain way for a Host to keep a model from a file, and a copy or a rename out of that path would otherwise hand the content over. The same holds for a policy that refuses writes under a path and a copy or a rename into it. The questions under the operation's own kind come first, so a policy that treats a copy or a rename as one kind of change, as `ModePolicy` does, gives its own verdict and its own reason for it. A rename leaves out the question about its source as `Op::Delete`, so a policy that refuses deletes so that nothing is lost can still allow a move. `StrReplace`, the replacement of a text in a file, is asked the same way: about its path as `Op::Write`, and then as `Op::Read`, because it reads the file and its failure tells how many times the text occurs. A policy that refuses reads under a path therefore refuses a replacement there, and the failure is that refusal and never a count.

The questions name the paths of an operation, or the pattern of a glob, and nothing else. A rename of a directory moves everything in it, and a recursive removal takes everything in it, so a policy that guards a path must also refuse a rename and a removal of each directory above that path. A glob is asked about its pattern and about none of the paths that it returns, so a glob that starts above a guarded path returns the names below it, and no content. A policy is asked about the canonical path of each operation and never about what a link leads to, so where a backend follows a link, as `RealBackend` does for a target inside its root (section 3), an operation through a link under an allowed path reaches a file under a guarded one.

The model reads the `Deny` text, and a person would read the `Ask` text. A tool of a library Plugin receives both answers as one file error of the C contract, of kind `RV_FILE_ERROR_DENIED`, with the policy's text. One policy for a handle is enough, because a Host that needs a richer system can combine its rules inside a single `Policy` object. No Host policy applies to the store, because a policy decides what a run's tools may do to the files of the handle, and the store is the run's own memory, outside every handle.

### `ModePolicy` allows or refuses changes by mode: `Ask`, `Plan`, or `Agent`

`ModePolicy` is the second policy that ships. In `Mode::Ask` it answers `Ask` to every change, in `Mode::Plan` it allows changes to paths that end in `.md` and denies the others, and in `Mode::Agent` it allows everything. It allows every read. A copy or a rename is a change of both of its paths, so in `Mode::Plan` both must end in `.md`. A Host changes the mode during a run through a `ModeHandle`. A mode decides the changes that tools make through the handle, and no mode applies to a `store` call (the entry above).

Modes are one implementation of the `Policy` trait, separate from the machinery of the VFS. Workshop's default handle has the `AllowAll` policy, and tests are the only callers of `ModePolicy`.

### An operation sink observes every admitted operation of its handle

A Host installs a callback, the operation sink, with `VfsRefBuilder::on_op`. The handle calls it once for each path of an operation that it admitted: one that its own policy allowed in every question, from an `Access` whose scope and identity are still live. A handle that is mounted inside it can still refuse the operation afterward. Each call has the operation kind, the canonical path, and the `Origin` of the access, a label with a file and line. The label of a tool call's `Access` is the id of the tool. A copy or a rename has two paths, so two calls under its own kind are one copy or one rename, and the sink is no list of what the policy allowed. Tests are the only callers of `on_op`. A `Store` has no sink, so the sink observes the file operations of tool calls and never a `store` call.

Each store operation is an effect instead: the Harness sees it before it runs, and the run's record holds it (section 7). Each file operation of a tool is an effect as well (section 5), so the run's record holds it too, whether the policy allows it or refuses it.

## 5. Identities and conflicts

### Each operation belongs to an identity, and a conflict in a store ends the run

Every `Access` has an `ExecId`, the identity of one serial thread of execution: the main walk, a task, or a tool call. Each store has a claims table, which records which identity read or wrote which path of that store. When two identities use one store path, at least one of them writes, and the two are unordered, the second operation fails with `VfsError::Conflict`. The Engine then ends the run with the error code `determinism` and abandons the suspended chain, together with any `pcall` around the call.

A lock on the backend makes such a pair of accesses memory-safe, and the claims table refuses the second one all the same. The owner's reason: "It's safe, but it's not correct because that's non-deterministic behavior. That means replay is not going to work right."

### The claims check applies to a store alone, and the Host's files have none

An operation on the files of the run's handle makes no claim and checks none, whoever asked for it: a model's tool call, or the prompt's own `tools.call`. Two tasks that work on one file both succeed. A reader in one task holds back no writer in another, and the later write is the content. The test `two_tasks_whose_models_write_one_path_both_succeed_and_the_later_write_is_the_content` in `harness-internal/runner/tests/it/harness-files-shared.rs` runs that case. A handle that a Host builds checks no claims at all, so two runs over one handle, and the Host's own access beside them, share its files in the same way. `Access::admit` in `runvil-internal/vfs/src/handle/access.rs` makes a claim only on a handle that checks claims. On every handle it refuses an operation whose scope or whose identity has ended.

The owner's reason: "maybe the determinism error should only apply to store operations? and we should otherwise try to behave like regular harnesses, so that two subagents which conflict on file access can just work it out". On the prompt's own `tools.call`, the owner's words: "I think I am ok with Lua's own tools.call on Host files being unchecked. We can always put it back in later." The store is the prompt's own memory, and every party in it is the prompt's Lua, so the check stays there. The Host's files are the outside world: with a real folder mounted, another program can change a file, and a claims table knows the run's own identities alone.

The record of a run stays whole without the check, because each operation that a tool asks for is a recorded effect that the Harness performs in order (the last entry of this section). The changes that one model reply asks for apply one at a time, in the order the reply lists them, because a call of a tool whose `ToolInfo` sets `changes_files` runs alone ([Tools and Packs](tools-and-packs.md)). One gap is known: a tool that edits a file as a read followed by a write can lose an edit that another task makes between the two (section 9).

The claims code stays in `runvil-vfs` with its tests, and only that crate can make a handle check claims: `VfsRefBuilder::checking_claims` is private to the crate, and `Store::new` calls it. The check on the Host's files can therefore return.

### Happens-before order decides whether two accesses to a store conflict

Each identity has a vector clock. Fork and join are methods of a public type, `ScopeHandle` in `runvil-internal/vfs/src/handle/scope_handle.rs`, which `VfsRef::acquire_scope` returns. The type has four methods: `spawn`, `join`, `end_access`, and `end`. `ScopeHandle::spawn` forks a child identity, and the fork orders everything the parent did before it ahead of the child's first operation. `ScopeHandle::join` merges a child's clock into its owner's, and the owner may then read what the child wrote to the store. The Engine forks at each task spawn and joins at each delivery of a task's result ([Tasks and fanout](tasks-and-fanout.md)).

The ordering test is the one that the FastTrack race detector uses, and with it a verdict depends only on the structure of the program. A claim that lasted for the life of its identity would make the verdict depend on which model call returns first.

### A read claims the whole region it observes

Where claims are checked, `read` and `exists` claim a path, `glob` claims its pattern, and `list` claims a directory's children. A write claims its path and each parent directory it may create. A `glob` therefore conflicts with an unordered write of a matching file, whichever runs second. Two unordered writes below one new directory pass.

A `glob` or an `exists` that runs before a sibling task creates a matching file has to conflict as well, because its result would otherwise depend on timing. The claims are stricter than some cases need. That extra strictness is deterministic, and a missed conflict would be nondeterministic.

### A scope is one acquire and its forks, and the run closes its scope at `Done`

`VfsRef::acquire` starts a scope: a new identity plus every identity later forked from it. Accesses in two live scopes are always unordered, so in one store a write in one scope conflicts with an overlapping access in the other. A run is one scope, which the Engine starts with `VfsRef::acquire_scope` on the run's handle. A store view joins the scope of the `Access` it comes from and records its claims in the store's own claims table. The forks, joins, and ends of the scope therefore order store operations. Two stores share no claim, even under one identity, and the handle holds none. The Engine closes the run's scope when the run ends: each store's table then ignores the scope's claims, and an `Access` of the scope, a store view included, refuses each operation with `ScopeEnded`. A backend that wraps another backend passes the `AcquireContext` of `Vfs::acquire` on unchanged, so a wrapped `VfsRef` joins the caller's scope.

The scope bounds what a run can do: once it is closed, every access of the run is refused, so the run changes no file of the handle or of a store after it. A store view joins its caller's scope so that the identities of the run's chains order its `store` calls.

### A tool call works under its own identity, which the Engine ends with the call

The Engine forks an identity for each Pack tool call, in `prepare_catalog_call` in `runvil-internal/runtime/src/execute/scheduler/tool_call.rs`. The identity's `Access`, rooted at `/` of the run's handle, reaches the files of the handle and no store, and every file operation of the call runs on it ([Tools and Packs](tools-and-packs.md)). The Engine keeps the `Access` in the continuation of the call, a `ToolCallContinuation`, and passes it in each `Vfs` effect that it issues for a request of the call, and the Harness performs the operation on it. A tool has no way to the `Access`: the `ToolCall` effect holds none, and neither the Rust API nor the C contract lends one to a call. When the Harness answers the call, or the Engine aborts it together with the chain that awaits it, `ScopeHandle::end_access` joins the identity into the calling chain, and every later operation through that `Access` fails with `ScopeEnded`. When a run ends, the Engine withdraws every effect that is still out, and the identities of the open calls end with the run's scope, at `Done` (the entry above). The Engine ends the identity because a file operation of a call that it aborted could otherwise still run.

A tool call's operations make no claim, so none of them conflicts with another access, and the end of the call is what its identity still decides: the `Access` in a `Vfs` effect refuses the operation when the call has ended before a caller performs it. A failure of an operation, such as a path that is absent or a policy's denial, goes to the tool alone, and its Pack decides how to answer the call. The loader gives a library Pack's tool the failure as a file error of the C contract, and it gives no failure the kind `RV_FILE_ERROR_CONFLICT`. The Engine ends a run for a conflict only in the answer to a `store` call: `classify_vfs_failure` in `runvil-internal/runtime/src/execute/scheduler/apply.rs` applies the fatal rule to an `EffectAnswer::Vfs` alone, and the answer to a tool's request, an `EffectAnswer::VfsReceipt`, resumes no chain.

### A tool's file request is a `Vfs` effect, which the Harness performs and records

A tool asks for a file operation and waits for the outcome. A Rust Pack's tool asks through `ToolContext::files`, a `ToolFiles` whose method `request` takes a `VfsOp` and resolves with a `VfsOutcome` or a `VfsError`. A library Pack's tool asks through one of the nine file functions of the C contract, and the loader passes the request on through the same `ToolFiles` ([Tools and Packs](tools-and-packs.md)). The Harness reports the request to the Engine with `Run::request_file`, the Engine issues a `Vfs` effect under the call's identity, the Harness performs and records it, and the tool goes on with the answer ([The Engine](engine.md)). The tool receives the whole outcome, and the Engine receives a receipt of it, which resumes no chain, because the chain still waits for the tool call. A request is a tool's one way to the run's files, so the record holds every file operation of a tool. The file tools of `std` and the shell of `bash` read and change a run's files this way. The owner's rule: "every VFS operation that's performed by a tool has to turn into an effect in the engine."

The Harness performs a tool's request as it performs a `store` call: inline, with one blocking call of `Access::perform`, in the order in which the Engine issues the effects (section 7). It runs a tool's operation inside a guard, so a panic in the Host's policy or backend reaches the tool as a `VfsError::Backend`. A call that has returned or was aborted has no one to take an outcome, so the Harness drops its later requests and answers its outstanding `Vfs` effect as `Dropped`, with the operation left unperformed ([The Harness and the Host API](harness-and-host.md)).

The reason is the record. The owner's questions about a tool that performs its file operations itself: "how's replay gonna work? How are we gonna do replay? How are we gonna handle debugging?" One ordered record of a run exists only if the Harness sees every effect and every answer ([The Harness and the Host API](harness-and-host.md)).

## 6. Errors

### `VfsError` is the one error type, and each variant has a stable tag

A VFS operation that fails returns a `VfsError`, an enum of 20 variants whose details are public fields. `VfsError::tag()` returns one fixed string for each variant, such as `not_found`. Each kind of refusal has its own variant, such as `ReadOnly` and `OutsideRoot`.

### A failed `store` call raises a Lua error value of kind `store`

In that value, `err.kind` is `store`, `err.reason` is the variant's tag, and `err.path` is the path as the prompt wrote it. An anchor failure adds `anchor` and `count`, and an invalid path adds `rule`. A `pcall` catches the value, and an uncaught failure ends the run with the error code of its reason, such as `not_found`. A wrong argument type raises an error of kind `lua`.

An error has three readers: Rust code matches the variant, Lua code branches on `err.reason` and the fields, and the model reads the message.

## 7. Running a store call, and persistence

### Each `store` call is one `Vfs` effect, which the Harness performs inline

A `store` function yields its operation, and the Engine issues one `Effect::Vfs` for it ([The Engine](engine.md)). The Harness performs the effect at once with `perform_vfs_op`, before it waits for any other effect. The Engine reports one of sixteen events for each call, such as `VfsWriteSucceeded`, which names the section and omits the path and the store. The run's record holds every store operation: each `Vfs` effect record with the id of its store and its operation, and each answer record with its outcome, in order. The record is the only view of a store outside its run, and the records of one store are enough to rebuild it. The test `a_runs_vfs_records_rebuild_its_store` in `harness-internal/runner/tests/it/effect_loop-vfs.rs` applies each `Vfs` record of a run that has an `Ok` answer to an empty memory handle, and the file it rebuilds reads as the text that the run returned.

The Harness performs the call inline because file operations are synchronous by design: the storage underneath is synchronous, so a `Vfs` effect has no future for the effect loop to wait on, and the run waits for the operation. The owner's reason: "all file I/O synchronizes the, the harness, and that's in line with our design." `answer_vfs` in `harness-internal/runner/src/effect_loop.rs` performs the operation through the effect's view, and the view drops when the function returns, so the Harness holds no way to the store between effects. The loop answers every `Vfs` effect of a step this way and then steps the run once. VFS operations run one at a time, in the order the Engine issues them, whether a `store` call or a tool's request asked for each.

Store operations are effects for the owner's reason: "We want to be able to debug prompts and replay them and see how the contents of the store evolves". The record holds every store call with its arguments and its outcome, the Harness sees each call before it happens, and a replay would issue the same effects again (section 8).

The `store` functions stay in the Engine. Serving them from a tool would only restructure the code to make the Engine smaller, and in the owner's words, "This has nothing to do with capabilities." The owner's verdict on serving them from a tool: "That turned out to be not a good idea". A store that a tool serves is more work, in the owner's words "to turn it into a tool and to create all the rigmarole", and the store is part of the language: "It just needs to be a language feature." The owner's rule is that a feature that is deeply part of the language stays in the Engine.

### A `Vfs` effect and its record name a store by a chain id, and a tool call by its effect id

`Effect::Vfs` has three fields: `asker`, `access`, and `op`. The asker of a `store` call is a `VfsAsker::Lua`, which names the store by a `ChainId`, and the effect's record holds that id beside the operation, as in `{"Vfs":{"store":"0","op":{...}}}`. The id `0`, the root chain's, names the store that the run starts with. A store that an `execute` made has the id of the executed prompt's chain, such as `0.0`, and every operation on that store has this id, whichever chain of the executed prompt made the call. `ChainStore` in `runvil-internal/runtime/src/execute/context.rs` keeps a store together with the id of the chain it was made for.

The record names the store because an executed prompt reports under its caller's task, and two stores can each have a file at one path. Without the id, a reader of the log could not tell the caller's `a.md` from the child's, and neither store could be rebuilt. With it, the records of each store rebuild that store. The test `the_vfs_records_of_each_store_id_rebuild_that_store` in `runvil-internal/runtime/src/execute/tests/execute/store.rs` applies the records of a run of three prompts to one empty store for each id, and each rebuilt store has the files that its prompt wrote. The id is a chain id because a chain id is the same in every run with the same inputs and answers ([Tasks and fanout](tasks-and-fanout.md)), which a replay needs.

The asker of a tool's request is a `VfsAsker::Tool`, which names the tool call by the id of its `ToolCall` effect, and the record holds that id beside the operation, as in `{"Vfs":{"call":7,"op":{...}}}`. Its files are the files of the run's handle, which the record leaves unnamed because a run has one handle. The effect record holds the operation with the bytes of a write, as text under `contents` when they are UTF-8 and as hex under `contents_hex` otherwise. The answer record holds a receipt in place of content, a `VfsReceipt`, such as `{"VfsReceipt":{"Ok":{"Bytes":12}}}` for a read of 12 bytes ([The Engine](engine.md)). The tool's own result is the answer of its `ToolCall` effect. The Engine reports no event for a tool's file operation, so the two records are the view of it.

### A run's store ends with the run, and the files of its tools last as long as the Host's mounts

A run's store is memory: it is empty when the run starts and is gone when the run ends, so no run finds a file of an earlier run in its store. The store of an executed prompt is gone sooner, when that prompt ends (section 3). A prompt that built a file returns the file's text, as in `return store.read('report.md')`. The files of a run's tools are in the mounts of the run's handle. A `MemoryBackend` drops with its handle, and the files below a `RealBackend::rooted` mount are in a real directory, which outlasts the run. Workshop builds `VfsRef::default()` for each run, so the files of a Workshop run's tools exist in memory until the run ends. Workshop's workspace document is a separate file ([Workshop: how the app is built](workshop-app.md)).

A run has one way in, its arguments, and one way out, its result. The owner's reason: "If a prompt wants to return a file, it should return it as the string of its return value."

## 8. Planned work

Each of these items is still to be built.

- **A real directory in Workshop.** Workshop would mount a real directory into each run's file system, and the choice of the directory is a product decision. Workshop builds `VfsRef::default()` for each run, so the files that a run's tools write and change are in memory and end with the run. Its code has two real mounts in `workshop/server/src/agents/language_guide.rs`, each read-only and each behind a constant that is `false`: `ENABLED` mounts a demo of the language guide, and `C_DRIVE` mounts the `C:` drive at `/c`, so that the shell and the file tools can be tried on real files. A tool's change under such a mount fails with `VfsError::ReadOnly`.
- **A database mount.** A mount would show an agent's run database as files. Its backend would be synchronous like every other, so a backend over a database client with an async interface would drive that client on a thread of its own. The backends that ship are `MemoryBackend` and `RealBackend`.
- **Replay.** Replay would read a run's records back into a run. A debugger would rebuild each store of a run from the same records, at any step. The Harness records each `Vfs` effect and its answer, for a `store` call and for a tool's file request.
- **Glob across mounts.** A glob would also search the backends mounted below the mount that matches its pattern. A glob runs inside that one mount and skips a backend mounted below it.

## 9. Known gaps

Each item names a doc comment in `runvil-vfs` that differs from the code, or a fault in the file system that is known and still open.

**Doc comments**

1. `VfsAccess` in `runvil-internal/vfs/src/traits/backend.rs`. The comment says that paths are "validated, canonicalized, and interned". An `Access` canonicalizes each path (section 2), and the crate lacks an interner.

**Known faults**

2. A lost edit. A tool that edits a file as a read followed by a write can lose a change that another task makes between the two, because an operation on the Host's files checks no claim (section 5). StrReplace of the `std` Pack edits this way, with a stat, a read, and a write, each a request of its own. The list of operations has `StrReplace`, which replaces a text in one operation, and a Rust Pack's tool can ask for it. The C contract has no function for it, which leaves the tools of `std` and `bash` with the read and the write, and whether to add one is the owner's decision. The gap is tracked as [runvil-plugin-std issue 4](https://github.com/cppalliance/runvil-plugin-std/issues/4). Delete of `std` has the same shape with its stat and its removal: it removes a directory with no entries that another task puts at the path between the two ([runvil-plugin-std issue 3](https://github.com/cppalliance/runvil-plugin-std/issues/3)).
3. The removal of a directory that a mount sits under. With a mount at `/project/assets` and no `/project` in the backend that serves `/`, the directory `/project` exists, stats as a directory, and lists `assets`. A removal of it goes to the serving backend, which has no such path: a `Delete` succeeds, a `Remove` fails with `NotFound`, and the directory is still there. A store has one mount, so the fault shows only on a handle that a Host builds with one mount below another. The evidence is a reading of `RoutingAccess` in `runvil-internal/vfs/src/router.rs`, and the tests of that file leave the case out.
4. A write below a file. The two backends that ship answer differently for a write to a path directly below a file. `MemoryBackend` fails with `NotADirectory` for the parent. `RealBackend` asks the standard library's `create_dir_all` to make the parent, and the plan of the change, `plan/2026-11-16-1-tool-file-effects.plan.md` in `runvil-harness`, records from a reading of that function that the write fails with `AlreadyExists` for the written path. That session and this one left the case unrun.
5. A read far past the end of a file. The doc of `Access::read_range` promises an empty answer for an offset at or past the end of a file. `RealBackend` seeks to the offset before it reads, and the loader caps the offset of a tool's read at `i64::MAX`. The same plan records, from a reading of the code and of the kernel's rule, that ext4 refuses a seek past its largest file size, so such a read would fail there with an I/O error. That session and this one left the case unrun.

## 10. Questions for the owner

Each question asks for the reason behind a part of the design.

**The store API**

1. Why does `store.delete` succeed on an absent path?
2. Why do store calls report sixteen event kinds, one pair for each operation?

**Conflicts**

3. Why does the claims table fold case below a rooted real mount and compare exactly below an identity mount?

**Hosts and backends**

4. Why does Plan mode allow changes only to `.md` paths?
5. Why is read-only a flag of the backend, separate from the policy?
6. Why do `mount` and `overlay` panic on a bad prefix, where they could return an error?
7. Why is a policy asked about the paths that an operation names and about nothing else, where a rename or a recursive removal of a directory takes every file below it, a link leads to a second path, and a glob returns paths?

## 11. Method and limits

- **Code.** The page describes the commits of the `Code:` bullet.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-11 the sentence on the `std` Pack's answer to a conflict, in the entry on conflicts inside a tool call, was brought to `runvil` at commit `b0693699d` and `runvil-plugin-std` at `782ad23`, where `std` is a library Plugin. The rest of the page still describes the commits of the `Code:` bullet.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The text of the page still describes the code at the commits above, when those crates were in `runvil`.
- **Every heading is a section.** On 2026-11-13 the first entry of section 1 was brought to the second change that follows `8de43c2f` in `runvil-harness`, whose plan is `plan/2026-11-13-2-every-heading-is-a-section.plan.md` there. Before it, the Lua blocks under a prompt's one H1 ran once outside every section, as the H1 pass, in a VM of their own that had the `store` table too. That session ran the build and its tests.
- **Update.** On 2026-11-13 the text on file access by tools was brought to `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980`. Between the commits, Workshop moved to the `runvil-workshop` repository, and the bashkit Plugin with its `bash` Pack began to ship. The update covers the first entry of section 1, the policy entry of section 4, the entry on a tool call's identity in section 5, three items of section 8, and the Paths bullet. That session read the code and left the build and the tests unrun. The rest of the page still describes the commits above.
- **Update.** On 2026-11-19 the text on file operations by tools was brought to `runvil-harness` at commit `ee199e3a`, `runvil-workshop` at `956e7800`, `runvil-plugin-api` at `52a4905`, `runvil-plugin-std` at `a806bf6`, and `runvil-plugin-bashkit` at `f348f7a`. The code of `runvil-harness` at `ee199e3a` is the code at `4afed45f`, which `runvil-workshop` pins. Between the commits of the update above and these, a tool lost its file access: it asks for each file operation, the Engine issues the operation as a `Vfs` effect, and the Harness performs and records it. The C contract went from version 0.4 to 0.5 and gained six functions that change files, and the claims check moved to the store alone. The update covers the summary, the walk-through, the first entry of section 1, four entries of section 2, four entries of section 3, the three entries of section 4, the seven entries of section 5, the first two entries of section 7, sections 8 to 10, and the Update bullets at the top of the page. Four of those entries are new: on the one list of operations, on slow storage and remote systems, on the claims check, and on a tool's file request. Three items left section 8: the store as a tool and an async VFS, which the owner decided against, and file changes for a Plugin's tools, which are built. The gap on the comment of the `Vfs` trait left section 9, because the comment gives the reason of section 2 at these commits, and four known faults joined that section. Two questions left section 10, because the entries of sections 1 and 5 hold their answers, and one question joined it. The update replaces the text that the updates of 2026-11-11 and 2026-11-13 brought, except the Paths bullet. The owner's words that it adds are quoted from the decision pages of `runvil-harness` at `ee199e3a`. That session read the code and left the build and the tests unrun. It also read the entries that it left as they were against the same commits and found nothing to change in them.

*2026-11-10 16:40 - Claude Fable 5.2*
