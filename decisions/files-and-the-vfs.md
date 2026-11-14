# Files and the VFS: design decisions and their reasons

- Scope: the `store` Lua API, the virtual file system (VFS), real and virtual roots, what a run may read and write, file identities and conflicts, and persistence
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the sentence on the `std` Pack's answer to a conflict describes `runvil` at commit `b0693699d` and `runvil-plugin-std` at `782ad23`, and the Paths bullet names the repositories as of `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 11)
- Update: the first entry of section 1 states the rule of the second change that follows `8de43c2f` in `runvil-harness`: every heading starts a section, and the code before the first heading is the preamble (section 11)
- Update: the text on how a tool call reaches files, on the Packs through which a model reads them, on Workshop's demo mounts, and three planned items describe `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980` (section 11)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil-workshop`

## Summary

A run's files are in a virtual file system, the VFS, which the crate `runvil-vfs` implements. The Host builds one `VfsRef` handle for each run: a table of mounts, each a backend in memory or over a real directory, which holds the files the run's tools work in. The store is a separate value, a `Store`, that the Engine makes for each run, and each `execute` gives the prompt that it runs a store of its own. Lua reads and writes a store through the `store` table. The main walk, each task, and each tool call has an `Access`, an object that performs file operations for one identity: it checks the path, the policy of its handle, and what the others have read and written, and then calls the backend. A `store` call runs on a view of the store, an `Access` with the calling chain's identity that reaches the store alone and asks no Host policy. Sections 1 to 7 describe this design.

A check of this page against the code found two gaps (section 9) and eight questions about reasons (section 10). The larger gap is the comment on the `Vfs` trait: it says that a single synchronous thread drives a run, and the Harness calls `Run::step` from an async function.

Section 8 lists the planned work. Its largest item is a sequence of two steps: the store becomes a tool, and the VFS then becomes async.

## From `store.write` to the backend and back

A Lua block calls `store.write('notes.md', text)`. The function is a shim that yields a table, which the Engine parses into a `Request::Store` with a `VfsOp::Write`. The Engine derives a store view from the run's `Store` under the `Access` of the calling chain, such as the main walk or a task. The view is a second `Access` with the chain's identity and scope, and it reaches the run's store alone. The Engine issues `Effect::Vfs` with the id of the store, the view, and the operation. `Run::step` returns the effect, and the chain waits.

The Harness records the effect and calls `perform_vfs_op`, which calls `Access::write` on the view.

`Access::write` checks `notes.md` against the store's path rules and canonicalizes it to `/notes.md` inside the store. Its policy check goes to the store's own policy, `AllowAll`, which allows every operation, so no Host policy is asked. It checks the store's own claims table for a conflicting access by another identity (section 5) and records a write claim there. The view then calls the store's backend, a `MemoryBackend`, which stores the bytes. The Harness records the answer and resumes the run. The Engine reports the event `VfsWriteSucceeded`, and `store.write` returns `nil`.

A later section of the same walk calls `store.read('notes.md')`, which runs the same steps with a `VfsOp::Read` and a read claim. The walk has one identity for all its sections, so the claim check passes, and Lua receives the text. Had a task written `notes.md` in between, before the walk joined it, the read would fail with `VfsError::Conflict`, and the Engine would end the run with the error code `determinism`.

## 1. The `store` table

### Every Lua block has a `store` table over the files that a prompt's sections and tasks share

The Engine installs a `store` table in the Lua VM of every section entry. The preamble, the Lua before a prompt's first heading, is a section entry like the others ([The prompt language](prompt-language.md)). The table's functions read and write the store, a set of text files that all the sections and tasks of one prompt share while it runs. The run's own prompt has the store that the run starts with, and a prompt that an `execute` runs has a new store for that execution (section 3). A tool call works in the files of the run's handle and sees no file of a store (section 3).

The owner's reason: "Store is always present, its a core feature of the Runvil language because its the way to move data between sections and handle fanout."

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

### Virtual paths have one POSIX form on every operating system

A virtual path is rooted and uses forward slashes on every operating system. An `Access` canonicalizes each path when it receives it, so the policy, the claims table, and the backend see one spelling. A path without a leading `/` is relative to `/`.

With one canonical form, only `RealBackend` translates a path to the operating system's form. Case is significant, with one exception: below a rooted real mount on Windows and macOS the claims table compares paths in lowercase, because a false conflict fails safe.

### `VfsRefBuilder::build` fixes a handle's mount table, and the longest matching prefix wins

`VfsRefBuilder::mount` installs a backend at a path prefix, and `build` returns the handle. The handle's router sends each operation to the mount with the longest matching prefix. `VfsRef::overlay` returns a second handle with one more mount above the first, and the two share one claims table.

A table fixed at construction makes routing immutable and a handle cheap to clone for each run.

### The backend traits are synchronous and work on bytes

Every method of `Vfs` and `VfsAccess` is a blocking call, file content is bytes, and `Access` adds the text operations. A `VfsRef` locks its backend for each call, so a backend must be `Send` and may leave locking to the handle.

The traits are synchronous because the backends are synchronous by nature: memory and `std::fs`. Lua is synchronous as well, and that is beside the point. An async VFS is planned work (section 8).

## 3. Real and virtual roots

### The Host builds the file system that a run's tools work in

A Host sets `RunSetup.vfs` to one `VfsRef` for each run: the file system that the run's tool calls work in ([The Harness and the Host API](harness-and-host.md)). The Harness passes the handle to the Engine with `RunParams::with_vfs`, and each tool call receives an `Access` to it (section 5). `RunSetup::new` sets `VfsRef::default()`, one empty in-memory mount at `/`. A Host can fill the handle before a run and read it after the run, through its own clone of the `VfsRef`, and concurrent runs can share one handle. When the handle refuses the run its access, or refuses the fork for a task or a tool call, the failure has the handle's own error code and the message `file access failed: {cause}`.

The Host builds the handle because the mounts are the Host's choice: memory, a real directory, or a backend of its own. The handle holds tool files alone, and the Engine and the Harness put nothing into it.

### The Engine makes each run an empty store, which only the run reaches

The Engine makes a run's store with `Store::new()` in `ChainState::root`, in `runvil-internal/runtime/src/execute/context.rs`. A new store is empty. Every chain that runs the run's own prompt shares the one store: the main walk, a `call` chain, a task, and a fanout arm. The Engine never hands the `Store` out: a caller reaches the store only through the view in a `Vfs` effect, for that effect's one operation (section 7). Nothing seeds the store before the run, and nothing reads it afterward. Data goes into a run as `args` and comes out as the result ([The Harness and the Host API](harness-and-host.md)). The store and the handle are disjoint: a tool sees no file of the store, and a `store` call sees no file of the handle.

The owner's reason: "Every prompt starts with a, a memory store that's completely empty. And that's the only type of store there is. And it doesn't map to any file system path in the virtual file system." With no way in from outside, a run's result depends only on its prompt, its arguments, and the answers to its effects.

### Each `execute` gives the prompt it runs a new, empty store

`ChainState::with_prompt`, in the same file, makes a store with `Store::new()` for the chain of each `execute` ([The prompt language](prompt-language.md)). The store is empty and has a claims table of its own. The executed prompt's `call` chains and tasks share it, and a prompt that the child executes in turn gets another. The caller and the child see none of each other's store files: one path in the two stores is two files, and the caller finds no file of the child after the `execute` returns. A second `execute` of the same prompt starts with an empty store again.

A child's store is gone when the child ends. Each chain releases its hold on its store when it finishes or is aborted, and a store is freed once the last chain that shared it has ended.

The child's tool calls are a different matter: they work in the files of the run's handle under the caller's file access, as the caller's own tool calls do, because the executed prompt is a part of the caller's run.

The rule is the owner's (section 2). An executed prompt therefore has the one way in and the one way out that a run has: its arguments and its text. Two fanout arms that execute one prompt can each write the same path with no conflict, because each execution has its own store and its own claims.

### `RealBackend` maps a real directory into the namespace and confines paths and links to it

`RealBackend::rooted(dir)` maps the virtual root to one real directory and refuses a path that resolves outside it with `OutsideRoot`. `RealBackend::identity()` maps each virtual path to the same real path. `remove`, `exists`, `stat`, `mkdir`, and `rename` act on a link itself, and the content operations follow a link whose target is inside the root. `with_writability` makes a backend read-only. A failed write leaves the file unchanged, because the backend writes a temporary sibling file and renames it.

The five operations act on the link itself because a `remove` that followed the link would delete the link's target. `RealBackend` is part of `runvil-vfs` itself: `std::fs` belongs to the standard library, so the backend fits the crate's zero-dependency rule ([Crates, build, and CI](crates-build-and-ci.md)).

## 4. What a run may read and write

### Each handle has one policy, which answers Allow, Deny, or Ask

An `Access` asks the handle's `Policy` about every operation, reads included, before it checks claims. `Policy::check` returns `Allow`, `Deny(String)`, or `Ask(String)`. The file system turns `Deny` into `PermissionDenied` and `Ask` into `ApprovalRequired`. The default policy is `AllowAll`. A store is no part of the handle and has `AllowAll` as its own policy, so the handle's policy sees the file operations of tool calls and never a `store` call.

The model reads the `Deny` text, and a person would read the `Ask` text. A tool of a library Plugin receives both answers as one file error of the C contract, of kind `RV_FILE_ERROR_DENIED`, with the policy's text. One policy for a handle is enough, because a Host that needs a richer system can combine its rules inside a single `Policy` object. No Host policy applies to the store, because a policy decides what a run's tools may do to the files of the handle, and the store is the run's own memory, outside every handle.

### `ModePolicy` allows or refuses changes by mode: `Ask`, `Plan`, or `Agent`

`ModePolicy` is the second policy that ships. In `Mode::Ask` it answers `Ask` to every change, in `Mode::Plan` it allows changes to paths that end in `.md` and denies the others, and in `Mode::Agent` it allows everything. It allows every read. A Host changes the mode during a run through a `ModeHandle`. A mode decides the changes that tools make through the handle, and no mode applies to a `store` call (the entry above).

Modes are one implementation of the `Policy` trait, separate from the machinery of the VFS. Workshop's default handle has the `AllowAll` policy, and tests are the only callers of `ModePolicy`.

### An operation sink observes every admitted operation of its handle

A Host installs a callback, the operation sink, with `VfsRefBuilder::on_op`. The handle calls it for each operation that passed the policy and the claims check, with the operation kind, the canonical path, and the `Origin` of the access, a label with a file and line. Tests are the only callers of `on_op`. A `Store` has no sink, so the sink observes the file operations of tool calls and never a `store` call.

Each store operation is an effect instead: the Harness sees it before it runs, and the run's record holds it (section 7).

## 5. Identities and conflicts

### Each operation belongs to an identity, and a conflict in a `store` call ends the run

Every `Access` has an `ExecId`, the identity of one serial thread of execution: the main walk, a task, or a tool call. A claims table records which identity read or wrote which path. When two identities use one path, at least one of them writes, and the two are unordered, the second operation fails with `VfsError::Conflict`. For a `store` call the Engine then ends the run with the error code `determinism` and abandons the suspended chain, together with any `pcall` around the call.

A lock on the backend makes such a pair of accesses memory-safe, and the claims table refuses the second one all the same. The owner's reason: "It's safe, but it's not correct because that's non-deterministic behavior. That means replay is not going to work right."

### Happens-before order decides whether two accesses conflict

Each identity has a vector clock. Fork and join are methods of a public type, `ScopeHandle` in `runvil-internal/vfs/src/handle/scope_handle.rs`, which `VfsRef::acquire_scope` returns. The type has four methods: `spawn`, `join`, `end_access`, and `end`. `ScopeHandle::spawn` forks a child identity, and the fork orders everything the parent did before it ahead of the child's first operation. `ScopeHandle::join` merges a child's clock into its owner's, and the owner may then read what the child wrote. The Engine forks at each task spawn and joins at each delivery of a task's result ([Tasks and fanout](tasks-and-fanout.md)).

The ordering test is the one that the FastTrack race detector uses, and with it a verdict depends only on the structure of the program. A claim that lasted for the life of its identity would make the verdict depend on which model call returns first.

### A read claims the whole region it observes

`read` and `exists` claim a path, `glob` claims its pattern, and `list` claims a directory's children. A write claims its path and each parent directory it may create. A `glob` therefore conflicts with an unordered write of a matching file, whichever runs second. Two unordered writes below one new directory pass.

A `glob` or an `exists` that runs before a sibling task creates a matching file has to conflict as well, because its result would otherwise depend on timing. The claims are stricter than some cases need. That extra strictness is deterministic, and a missed conflict would be nondeterministic.

### A scope is one acquire and its forks, and the run closes its scope at `Done`

`VfsRef::acquire` starts a scope: a new identity plus every identity later forked from it. Accesses in two live scopes are always unordered, so a write in one conflicts with an overlapping access in the other. A run is one scope, which the Engine starts with `VfsRef::acquire_scope` on the run's handle. A store view joins the scope of the `Access` it comes from and records its claims in the store's own claims table. The forks, joins, and ends of the scope therefore order store operations as they order file operations. A store and the handle share no claim, and two stores share none, even under one identity. The Engine closes the run's scope when the run ends: each table then ignores the scope's claims, and an `Access` of the scope, a store view included, refuses each operation with `ScopeEnded`. A backend that wraps another backend passes the `AcquireContext` of `Vfs::acquire` on unchanged, so a wrapped `VfsRef` joins the caller's scope.

Scopes let a Host write files before a run and read them after it, and let concurrent runs share a base of real files. A store view joins its caller's scope so that the ordering test for a `store` call is the one that file operations use.

### A tool call works under its own identity, which the Engine ends with the call

The Engine forks an identity for each Pack tool call and passes its `Access`, rooted at `/` of the run's handle, in the `ToolCall` effect ([Tools and Packs](tools-and-packs.md)). The `Access` reaches the files of the handle and no store. A Rust Pack calls the `Access` itself. For a Pack that follows the C contract, the loader keeps the `Access` and answers the contract's three file functions from it: `file_stat`, `file_list`, and `file_read`. Contract 0.4 reads only, so the tools of `std` and `bash` read files, and the two Packs that ship through the Rust API, `web` and `mcp`, leave the `Access` unused. When the Harness answers the call or the Engine aborts it, `ScopeHandle::end_access` joins the identity into the calling chain, and every later operation through that `Access` fails with `ScopeEnded`. The Engine ends the identity because a call that it aborted could otherwise still write.

When an operation inside a tool call conflicts, the `Access` returns `VfsError::Conflict` to the Pack, which decides how to answer the call. For the `std` Pack, a library Plugin, the loader passes the conflict on as a file error of the C contract, and `refusal` in `crates/plugin-std/src/refusal.rs` of the `runvil-plugin-std` repository converts it into a sentence that tells the model to call the tool again. In the shell of the `bash` Pack the same file error fails the command that read the file, and the model reads the reason in the command's output. The Engine resumes the chain with the Pack's answer. The Engine ends the run only for a conflict in a `store` call: `classify_vfs_failure` in `runvil-internal/runtime/src/execute/scheduler/apply.rs` applies the fatal rule only to answers of `Effect::Vfs`.

## 6. Errors

### `VfsError` is the one error type, and each variant has a stable tag

A VFS operation that fails returns a `VfsError`, an enum of 20 variants whose details are public fields. `VfsError::tag()` returns one fixed string for each variant, such as `not_found`. Each kind of refusal has its own variant, such as `ReadOnly` and `OutsideRoot`.

### A failed `store` call raises a Lua error value of kind `store`

In that value, `err.kind` is `store`, `err.reason` is the variant's tag, and `err.path` is the path as the prompt wrote it. An anchor failure adds `anchor` and `count`, and an invalid path adds `rule`. A `pcall` catches the value, and an uncaught failure ends the run with the error code of its reason, such as `not_found`. A wrong argument type raises an error of kind `lua`.

An error has three readers: Rust code matches the variant, Lua code branches on `err.reason` and the fields, and the model reads the message.

## 7. Running a store call, and persistence

### Each `store` call is one `Vfs` effect, which the Harness performs inline

A `store` function yields its operation, and the Engine issues one `Effect::Vfs` for it ([The Engine](engine.md)). The Harness performs the effect at once with `perform_vfs_op`, before it waits for any other effect. The Engine reports one of sixteen events for each call, such as `VfsWriteSucceeded`, which names the section and omits the path and the store. The run's record holds every store operation: each `Vfs` effect record with the id of its store and its operation, and each answer record with its outcome, in order. The record is the only view of a store outside its run, and the records of one store are enough to rebuild it. The test `a_runs_vfs_records_rebuild_its_store` in `harness-internal/runner/tests/it/effect_loop-vfs.rs` applies each `Vfs` record of a run that has an `Ok` answer to an empty memory handle, and the file it rebuilds reads as the text that the run returned.

### A `Vfs` effect and its record name the store by a chain id

`Effect::Vfs` has a `store` field, a `ChainId`, and the effect's record holds it beside the operation, as in `{"Vfs":{"store":"0","op":{...}}}`. The id `0`, the root chain's, names the store that the run starts with. A store that an `execute` made has the id of the executed prompt's chain, such as `0.0`, and every operation on that store has this id, whichever chain of the executed prompt made the call. `ChainStore` in `runvil-internal/runtime/src/execute/context.rs` keeps a store together with the id of the chain it was made for.

The record names the store because an executed prompt reports under its caller's task, and two stores can each have a file at one path. Without the id, a reader of the log could not tell the caller's `a.md` from the child's, and neither store could be rebuilt. With it, the records of each store rebuild that store. The test `the_vfs_records_of_each_store_id_rebuild_that_store` in `runvil-internal/runtime/src/execute/tests/execute/store.rs` applies the records of a run of three prompts to one empty store for each id, and each rebuilt store has the files that its prompt wrote. The id is a chain id because a chain id is the same in every run with the same inputs and answers ([Tasks and fanout](tasks-and-fanout.md)), which a replay needs.

The Harness performs the call inline because the store is memory: a `Vfs` effect is one operation that touches no disk, so it has no future for the effect loop to wait on. `answer_vfs` in `harness-internal/runner/src/effect_loop.rs` performs the operation through the effect's view, and the view drops when the function returns, so the Harness holds no way to the store between effects. The loop answers every `Vfs` effect of a step this way and then steps the run once. VFS operations from different chains run one at a time, in the order the Engine issues them.

Store operations are effects for the owner's reason: "We want to be able to debug prompts and replay them and see how the contents of the store evolves". The record holds every store call with its arguments and its outcome, the Harness sees each call before it happens, and a replay would issue the same effects again (section 8).

### A run's store ends with the run, and the files of its tools last as long as the Host's mounts

A run's store is memory: it is empty when the run starts and is gone when the run ends, so no run finds a file of an earlier run in its store. The store of an executed prompt is gone sooner, when that prompt ends (section 3). A prompt that built a file returns the file's text, as in `return store.read('report.md')`. The files of a run's tools are in the mounts of the run's handle. A `MemoryBackend` drops with its handle, and the files below a `RealBackend::rooted` mount are in a real directory, which outlasts the run. Workshop builds `VfsRef::default()` for each run, so the files of a Workshop run's tools exist in memory until the run ends. Workshop's workspace document is a separate file ([Workshop: how the app is built](workshop-app.md)).

A run has one way in, its arguments, and one way out, its result. The owner's reason: "If a prompt wants to return a file, it should return it as the string of its return value."

## 8. Planned work

Each of these items is still to be built.

- **The store as a tool.** The `store` functions would become calls of a tool, which would remove the `Vfs` effect. Each `store` call is one `Vfs` effect (section 7).
- **An async VFS.** The backend traits would become async, which would work well with Turso and with Bashkit. The step follows the store tool. Every method of `Vfs` and `VfsAccess` is a blocking call (section 2). The shell of the `bash` Pack ships on the blocking calls: its file system sends each read to the tool call that runs the script.
- **File changes for a Plugin's tools.** Functions of the C contract with which a tool writes, deletes, and renames a run's files. Contract 0.4 has three file functions, which stat, list, and read, so the Write, StrReplace, and Delete tools of `std` are stubs, and the shell of `bash` refuses each change. `Access` has the operations, and a Rust Pack can call them ([Tools and Packs](tools-and-packs.md)).
- **A real directory in Workshop.** Workshop would mount a real directory into each run's file system, and the choice of the directory is a product decision. Workshop builds `VfsRef::default()` for each run. Its code has two real mounts, each read-only and each behind a constant that is `false`: `ENABLED` mounts a demo of the language guide, and `C_DRIVE` mounts the `C:` drive at `/c`, so that the shell and the file tools can be tried on real files.
- **A database mount.** A mount would show an agent's run database as files, once the VFS is async. The backends that ship are `MemoryBackend` and `RealBackend`.
- **Replay.** Replay would read a run's records back into a run. A debugger would rebuild each store of a run from the same records, at any step. The Harness records each `Vfs` effect and its answer.
- **Glob across mounts.** A glob would also search the backends mounted below the mount that matches its pattern. A glob runs inside that one mount and skips a backend mounted below it.

## 9. Known gaps

Two doc comments in `runvil-vfs` differ from the code.

1. `VfsAccess` in `runvil-internal/vfs/src/traits/backend.rs`. The comment says that paths are "validated, canonicalized, and interned". An `Access` canonicalizes each path (section 2), and the crate lacks an interner.
2. `Vfs` in the same file. The comment says the operations are synchronous "because the Lua VM and the single thread that drives a run are both synchronous". The Harness calls `Run::step` from an async function, `Driver::drive` in `harness-internal/runner/src/effect_loop.rs`. `Run` is `Send`, so the thread that calls `Run::step` may change between calls.

## 10. Questions for the owner

Each question asks for the reason behind a part of the design.

**The store API**

1. Why does a run store its bulk state as files?
2. Why does `store.delete` succeed on an absent path?
3. Why do store calls report sixteen event kinds, one pair for each operation?

**Conflicts**

4. Why does a conflict inside a tool call return to the Pack, while the same conflict in a `store` call ends the run?
5. Why does the claims table fold case below a rooted real mount and compare exactly below an identity mount?

**Hosts and backends**

6. Why does Plan mode allow changes only to `.md` paths?
7. Why is read-only a flag of the backend, separate from the policy?
8. Why do `mount` and `overlay` panic on a bad prefix, where they could return an error?

## 11. Method and limits

- **Code.** The page describes the commits of the `Code:` bullet.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-11 the sentence on the `std` Pack's answer to a conflict, in the entry on conflicts inside a tool call, was brought to `runvil` at commit `b0693699d` and `runvil-plugin-std` at `782ad23`, where `std` is a library Plugin. The rest of the page still describes the commits of the `Code:` bullet.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The text of the page still describes the code at the commits above, when those crates were in `runvil`.
- **Every heading is a section.** On 2026-11-13 the first entry of section 1 was brought to the second change that follows `8de43c2f` in `runvil-harness`, whose plan is `plan/2026-11-13-2-every-heading-is-a-section.plan.md` there. Before it, the Lua blocks under a prompt's one H1 ran once outside every section, as the H1 pass, in a VM of their own that had the `store` table too. That session ran the build and its tests.
- **Update.** On 2026-11-13 the text on file access by tools was brought to `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980`. Between the commits, Workshop moved to the `runvil-workshop` repository, and the bashkit Plugin with its `bash` Pack began to ship. The update covers the first entry of section 1, the policy entry of section 4, the entry on a tool call's identity in section 5, three items of section 8, and the Paths bullet. That session read the code and left the build and the tests unrun. The rest of the page still describes the commits above.

*2026-11-10 16:40 - Claude Fable 5.2*
