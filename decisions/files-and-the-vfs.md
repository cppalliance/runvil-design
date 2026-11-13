# Files and the VFS: design decisions and their reasons

- Scope: the `store` Lua API, the virtual file system (VFS), real and virtual roots, what a run may read and write, file identities and conflicts, and persistence
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the sentence on the `std` Pack's answer to a conflict describes `runvil` at commit `b0693699d` and `runvil-plugin-std` at `782ad23`, and the Paths bullet names the repositories as of `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 11)
- Update: the first entry of section 1 states the rule of the second change that follows `8de43c2f` in `runvil-harness`: every heading starts a section, and the code before the first heading is the preamble (section 11)
- Update: the text on how a tool call reaches files, on the Packs through which a model reads them, on Workshop's demo mounts, and three planned items describe `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980` (section 11)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil-workshop`

## Summary

A run's files are in a virtual file system, the VFS, which the crate `runvil-vfs` implements. The Host builds one `VfsRef` handle for each run: a table of mounts, each a backend in memory or over a real directory, with one mount declared as the store. Lua reads and writes the store through the `store` table. The main walk, each task, and each tool call has an `Access`, an object that performs its file operations: it checks the path, the Host's policy, and what the others have read and written, and then calls the backend. Sections 1 to 7 describe this design.

A check of this page against the code found three gaps (section 9) and eight questions about reasons (section 10). The largest gap is the comment on `OpSink`: it says that store operations call the sink from the blocking pool, and the Harness performs each store operation inline.

Section 8 lists the planned work. Its largest item is a sequence of two steps: the store becomes a tool, and the VFS then becomes async.

## From `store.write` to the backend and back

A Lua block calls `store.write('notes.md', text)`. The function is a shim that yields a table, which the Engine parses into a `Request::Store` with a `VfsOp::Write`. The Engine derives a store view, a second `Access` confined to the store, from the `Access` of the calling chain, such as the main walk or a task, and issues `Effect::Vfs` with the view and the operation. `Run::step` returns the effect, and the chain waits.

The Harness records the effect and calls `perform_vfs_op`, which calls `Access::write` on the view.

`Access::write` checks `notes.md` against the store's path rules and joins it onto the store root, which produces `/notes.md` under the default handle. It asks the handle's `Policy`, checks the claims table for a conflicting access by another identity (section 5), and records a write claim. The view then calls the store's backend, here a `MemoryBackend` at `/`, which stores the bytes. The Harness records the answer and resumes the run. The Engine reports the event `VfsWriteSucceeded`, and `store.write` returns `nil`.

A later section of the same walk calls `store.read('notes.md')`, which runs the same steps with a `VfsOp::Read` and a read claim. The walk has one identity for all its sections, so the claim check passes, and Lua receives the text. Had a task written `notes.md` in between, before the walk joined it, the read would fail with `VfsError::Conflict`, and the Engine would end the run with the error code `determinism`.

## 1. The `store` table

### Every Lua block has a `store` table over the run's shared files

The Engine installs a `store` table in the Lua VM of every section entry. The preamble, the Lua before a prompt's first heading, is a section entry like the others ([The prompt language](prompt-language.md)). The table's functions read and write the store, a set of text files that all the sections and tasks of one run share. A model reads the same files through the `std` Pack and the `bash` Pack ([Tools and Packs](tools-and-packs.md)). Both are library Plugins, and the C contract gives a tool call three file functions, which stat, list, and read. Functions that write and delete a file are planned work (section 8).

The owner's reason: "Store is always present, its a core feature of the Runvil language because its the way to move data between sections and handle fanout."

### The `store` table has eight functions, and a prompt composes the rest

The functions are `write`, `append`, `read`, `read_numbered`, `str_replace`, `delete`, `glob`, and `exists`. `store.str_replace` replaces an anchor text that must occur exactly once, because an edit by anchor is the pattern that works for models. `store.delete` succeeds on an absent path.

The table stops at eight because a prompt can write the other VFS operations from them. It leaves out `stat` because timestamps from a store on real files are nondeterministic.

### `store.read` takes 1-based inclusive line bounds, and `store.read_numbered` numbers the lines

`store.read(path, start, end)` returns lines `start` to `end`, and `store.read(path)` returns the whole file. An `end` past the last line means the last line, and a `start` below 1 raises `invalid_range`. `store.read_numbered` takes the same bounds and prefixes each line with its number in the file, as in `12| text`.

The second bound is an end line because a count would force `end - start + 1` arithmetic at every call site, a documented failure mode of small models.

### The store view accepts relative, nested paths and checks each against strict rules

A store path is relative to the store root, as in `drafts/plan.md`, and a write creates the missing parent directories. The store view checks each path first. A path has 1 to 1024 bytes. The view refuses a leading `/`, a backslash, a colon, a `.` or `..` segment, a name that ends in a dot or a space, and a Windows device name such as `COM0`. A refused path raises `invalid_path`, and `err.rule` names the broken rule.

The backslash, colon, suffix, and device-name rules refuse spellings that some backend would read as a different file.

## 2. The virtual file system

### Lua, tools, and the Host share one file system of Runvil's own design

`runvil-vfs` defines one file system for prompt Lua, tool calls, and the Host. A backend implements two traits: `Vfs` for the storage and `VfsAccess` for one identity's session with it. `VfsRef` is a cloneable handle to a set of mounted backends, and an `Access` performs operations through it for one identity.

The facade crates `runvil` and `runvil-engine` export the whole `vfs` module, so a Host can add a backend of its own. The owner's reason: "without the vfs, a host can't get data in or out of the prompt".

### A store call is one `Access` call on a store view

The store is one mount of the VFS. For each `store` call the Engine derives a store view from the calling chain's `Access`. The view is a second `Access` with the chain's identity, rooted at the store root, and its mount table contains only the store's mount.

The store follows one rule: it offers a subset of the operations of the VFS. The view is confined to the store's mount because a store at `/` would otherwise let Lua access any real directory mounted below it.

### Virtual paths have one POSIX form on every operating system

A virtual path is rooted and uses forward slashes on every operating system. An `Access` canonicalizes each path when it receives it, so the policy, the claims table, and the backend see one spelling. A relative path joins the root of its `Access`.

With one canonical form, only `RealBackend` translates a path to the operating system's form. Case is significant, with one exception: below a rooted real mount on Windows and macOS the claims table compares paths in lowercase, because a false conflict fails safe.

### `VfsRefBuilder::build` fixes a handle's mount table, and the longest matching prefix wins

`VfsRefBuilder::mount` installs a backend at a path prefix, and `build` returns the handle. The handle's router sends each operation to the mount with the longest matching prefix. `VfsRef::overlay` returns a second handle with one more mount above the first, and the two share one claims table.

A table fixed at construction makes routing immutable and a handle cheap to clone for each run.

### The backend traits are synchronous and work on bytes

Every method of `Vfs` and `VfsAccess` is a blocking call, file content is bytes, and `Access` adds the text operations. A `VfsRef` locks its backend for each call, so a backend must be `Send` and may leave locking to the handle.

The traits are synchronous because the backends are synchronous by nature: memory and `std::fs`. Lua is synchronous as well, and that is beside the point. An async VFS is planned work (section 8).

## 3. Real and virtual roots

### The Host builds each run's whole file system

A Host sets `RunSetup.vfs` to one `VfsRef` for each run. The Harness writes the prompt's declared input into that handle's store and reads the declared output from it ([The Harness and the Host API](harness-and-host.md)). `RunSetup::new` sets `VfsRef::default()`, one in-memory mount at `/` that is the store.

The Engine and the Harness find the store through the handle, so a Host is free to mount the store at any path.

### The handle declares which mount is the store

`VfsRefBuilder::store(root, backend)` mounts a backend at any path and declares that mount the store. A handle made with `VfsRef::new` lacks a declaration. The Engine probes the store at the run's first step, and a run whose handle lacks one ends there with the error code `no_store`.

The declaration is explicit because a handle with an implied store would let `VfsRef::new(RealBackend::identity())` turn the whole disk into the store.

### `RealBackend` maps a real directory into the namespace and confines paths and links to it

`RealBackend::rooted(dir)` maps the virtual root to one real directory and refuses a path that resolves outside it with `OutsideRoot`. `RealBackend::identity()` maps each virtual path to the same real path. `remove`, `exists`, `stat`, `mkdir`, and `rename` act on a link itself, and the content operations follow a link whose target is inside the root. `with_writability` makes a backend read-only. A failed write leaves the file unchanged, because the backend writes a temporary sibling file and renames it.

The five operations act on the link itself because a `remove` that followed the link would delete the link's target. `RealBackend` is part of `runvil-vfs` itself: `std::fs` belongs to the standard library, so the backend fits the crate's zero-dependency rule ([Crates, build, and CI](crates-build-and-ci.md)).

## 4. What a run may read and write

### Each handle has one policy, which answers Allow, Deny, or Ask

An `Access` asks the handle's `Policy` about every operation, reads included, before it checks claims. `Policy::check` returns `Allow`, `Deny(String)`, or `Ask(String)`. The file system turns `Deny` into `PermissionDenied` and `Ask` into `ApprovalRequired`. The default policy is `AllowAll`.

The model reads the `Deny` text, and a person would read the `Ask` text. A tool of a library Plugin receives both answers as one file error of the C contract, of kind `RV_FILE_ERROR_DENIED`, with the policy's text. One policy for a handle is enough, because a Host that needs a richer system can combine its rules inside a single `Policy` object.

### `ModePolicy` allows or refuses changes by mode: `Ask`, `Plan`, or `Agent`

`ModePolicy` is the second policy that ships. In `Mode::Ask` it answers `Ask` to every change, in `Mode::Plan` it allows changes to paths that end in `.md` and denies the others, and in `Mode::Agent` it allows everything. It allows every read. A Host changes the mode during a run through a `ModeHandle`.

Modes are one implementation of the `Policy` trait, separate from the machinery of the VFS. Workshop's default handle has the `AllowAll` policy, and tests are the only callers of `ModePolicy`.

### An operation sink observes every admitted operation

A Host installs a callback, the operation sink, with `VfsRefBuilder::on_op`. The handle calls it for each operation that passed the policy and the claims check, with the operation kind, the canonical path, and the `Origin` of the access, a label with a file and line. Tests are the only callers of `on_op`.

## 5. Identities and conflicts

### Each operation belongs to an identity, and a conflict in a `store` call ends the run

Every `Access` has an `ExecId`, the identity of one serial thread of execution: the main walk, a task, or a tool call. The claims table records which identity read or wrote which path. When two identities use one path, at least one of them writes, and the two are unordered, the second operation fails with `VfsError::Conflict`. For a `store` call the Engine then ends the run with the error code `determinism` and abandons the suspended chain, together with any `pcall` around the call.

The handle's lock makes such a pair of accesses memory-safe, and the claims table refuses the second one all the same. The owner's reason: "It's safe, but it's not correct because that's non-deterministic behavior. That means replay is not going to work right."

### Happens-before order decides whether two accesses conflict

Each identity has a vector clock. Fork, join, and the store view are methods of a public type, `ScopeHandle` in `runvil-internal/vfs/src/handle/scope_handle.rs`, which `VfsRef::acquire_scope` returns. `ScopeHandle::spawn` forks a child identity, and the fork orders everything the parent did before it ahead of the child's first operation. `ScopeHandle::join` merges a child's clock into its owner's, and the owner may then read what the child wrote. The Engine forks at each task spawn and joins at each delivery of a task's result ([Tasks and fanout](tasks-and-fanout.md)).

The ordering test is the one that the FastTrack race detector uses, and with it a verdict depends only on the structure of the program. A claim that lasted for the life of its identity would make the verdict depend on which model call returns first.

### A read claims the whole region it observes

`read` and `exists` claim a path, `glob` claims its pattern, and `list` claims a directory's children. A write claims its path and each parent directory it may create. A `glob` therefore conflicts with an unordered write of a matching file, whichever runs second. Two unordered writes below one new directory pass.

A `glob` or an `exists` that runs before a sibling task creates a matching file has to conflict as well, because its result would otherwise depend on timing. The claims are stricter than some cases need. That extra strictness is deterministic, and a missed conflict would be nondeterministic.

### A scope is one acquire and its forks, and the run closes its scope at `Done`

`VfsRef::acquire` starts a scope: a new identity plus every identity later forked from it. Accesses in two live scopes are always unordered, so a write in one conflicts with an overlapping access in the other. A run is one scope, and the Harness writes the input and reads the output in scopes of its own. The Engine closes the run's scope when the run ends: the table then ignores the scope's claims, and an `Access` of the scope refuses each operation with `ScopeEnded`. A backend that wraps another backend passes the `AcquireContext` of `Vfs::acquire` on unchanged, so a wrapped `VfsRef` joins the caller's scope.

Scopes let the Harness write the input before a run and read the output after it, and let concurrent runs share a base of real files.

### A tool call works under its own identity, which the Engine ends with the call

The Engine forks an identity for each Pack tool call and passes its `Access`, rooted at `/`, in the `ToolCall` effect ([Tools and Packs](tools-and-packs.md)). A Rust Pack calls the `Access` itself. For a Pack that follows the C contract, the loader keeps the `Access` and answers the contract's three file functions from it: `file_stat`, `file_list`, and `file_read`. Contract 0.4 reads only, so the tools of `std` and `bash` read files, and the two Packs that ship through the Rust API, `web` and `mcp`, leave the `Access` unused. When the Harness answers the call or the Engine aborts it, `ScopeHandle::end_access` joins the identity into the calling chain, and every later operation through that `Access` fails with `ScopeEnded`. The Engine ends the identity because a call that it aborted could otherwise still write.

When an operation inside a tool call conflicts, the `Access` returns `VfsError::Conflict` to the Pack, which decides how to answer the call. For the `std` Pack, a library Plugin, the loader passes the conflict on as a file error of the C contract, and `refusal` in `crates/plugin-std/src/refusal.rs` of the `runvil-plugin-std` repository converts it into a sentence that tells the model to call the tool again. In the shell of the `bash` Pack the same file error fails the command that read the file, and the model reads the reason in the command's output. The Engine resumes the chain with the Pack's answer. The Engine ends the run only for a conflict in a `store` call: `classify_vfs_failure` in `runvil-internal/runtime/src/execute/scheduler/apply.rs` applies the fatal rule only to answers of `Effect::Vfs`.

## 6. Errors

### `VfsError` is the one error type, and each variant has a stable tag

A VFS operation that fails returns a `VfsError`, an enum of 21 variants whose details are public fields. `VfsError::tag()` returns one fixed string for each variant, such as `not_found`. Each kind of refusal has its own variant, such as `ReadOnly`, `OutsideRoot`, and `NoStore`.

### A failed `store` call raises a Lua error value of kind `store`

In that value, `err.kind` is `store`, `err.reason` is the variant's tag, and `err.path` is the path as the prompt wrote it. An anchor failure adds `anchor` and `count`, and an invalid path adds `rule`. A `pcall` catches the value, and an uncaught failure ends the run with the error code of its reason, such as `not_found`. A wrong argument type raises an error of kind `lua`.

An error has three readers: Rust code matches the variant, Lua code branches on `err.reason` and the fields, and the model reads the message.

## 7. Running a store call, and persistence

### Each `store` call is one `Vfs` effect, which the Harness performs inline

A `store` function yields its operation, and the Engine issues one `Effect::Vfs` for it ([The Engine](engine.md)). The Harness performs the effect at once with `perform_vfs_op`, before it waits for any other effect. The Engine reports one of sixteen events for each call, such as `VfsWriteSucceeded`, which names the section and omits the path. One phase differs: a Pack prelude runs outside any coroutine, so during prelude install each `store` function calls a direct closure on the store view. `route_store_to_shims` then switches every function to its yielding shim.

The inline call has a cost: a real-disk operation blocks the Host's executor thread, and VFS operations from different chains run one at a time, in the order the Engine issues them.

### A run's files remain after the run when the Host mounts lasting storage

A `MemoryBackend` drops with its handle, and a `RealBackend::rooted` store writes each file into a real directory, where the next run finds it. Workshop builds `VfsRef::default()` for each run, so a Workshop run's files exist in memory until the run ends. Workshop's workspace document is a separate file ([Workshop: how the app is built](workshop-app.md)).

Resume is Lua in the prompt: a section tests `store.exists(path)` and skips work whose file is present.

## 8. Planned work

Each of these items is still to be built.

- **The store as a tool.** The `store` functions would become calls of a tool, which would remove the `Vfs` effect and the direct closures. Each `store` call is one `Vfs` effect, or a direct closure during prelude install (section 7).
- **An async VFS.** The backend traits would become async, which would work well with Turso and with Bashkit. The step follows the store tool. Every method of `Vfs` and `VfsAccess` is a blocking call (section 2). The shell of the `bash` Pack ships on the blocking calls: its file system sends each read to the tool call that runs the script.
- **File changes for a Plugin's tools.** Functions of the C contract with which a tool writes, deletes, and renames a run's files. Contract 0.4 has three file functions, which stat, list, and read, so the Write, StrReplace, and Delete tools of `std` are stubs, and the shell of `bash` refuses each change. `Access` has the operations, and a Rust Pack can call them ([Tools and Packs](tools-and-packs.md)).
- **A real directory in Workshop.** Workshop would mount a real directory into each run's file system, and the choice of the directory is a product decision. Workshop builds `VfsRef::default()` for each run. Its code has two real mounts, each read-only and each behind a constant that is `false`: `ENABLED` mounts a demo of the language guide, and `C_DRIVE` mounts the `C:` drive at `/c`, so that the shell and the file tools can be tried on real files.
- **A database mount.** A mount would show an agent's run database as files, once the VFS is async. The backends that ship are `MemoryBackend` and `RealBackend`.
- **Replay.** Replay would read a run's records back into a run. The Harness records each `Vfs` effect and its answer.
- **Glob across mounts.** A glob would also search the backends mounted below the mount that matches its pattern. A glob runs inside that one mount and skips a backend mounted below it.

## 9. Known gaps

Three doc comments in `runvil-vfs` differ from the code.

1. `OpSink` in `runvil-internal/vfs/src/observe.rs`. The comment says that store operations call the sink from the blocking pool. The Harness performs each store operation inline with `perform_vfs_op` (section 7).
2. `VfsAccess` in `runvil-internal/vfs/src/traits/backend.rs`. The comment says that paths are "validated, canonicalized, and interned". An `Access` canonicalizes each path (section 2), and the crate lacks an interner.
3. `Vfs` in the same file. The comment says the operations are synchronous "because the Lua VM and the single thread that drives a run are both synchronous". The Harness calls `Run::step` from an async function, `Driver::drive` in `harness-internal/runner/src/effect_loop.rs`. `Run` is `Send`, so the thread that calls `Run::step` may change between calls.

## 10. Questions for the owner

Each question asks for the reason behind a part of the design.

**The store API**

1. Why does a run store its bulk state as files that Lua and the model share?
2. Why does `store.delete` succeed on an absent path?
3. Why do store calls report sixteen event kinds, one pair for each operation?

**Conflicts**

4. Why does a conflict inside a tool call return to the Pack, while the same conflict in a `store` call ends the run?
5. Why does the claims table fold case below a rooted real mount and compare exactly below an identity mount?

**Hosts and backends**

6. Why does Plan mode allow changes only to `.md` paths?
7. Why is read-only a flag of the backend, separate from the policy?
8. Why do `mount`, `store`, and `overlay` panic on a bad prefix, where they could return an error?

## 11. Method and limits

- **Code.** The page describes the commits of the `Code:` bullet.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-11 the sentence on the `std` Pack's answer to a conflict, in the entry on conflicts inside a tool call, was brought to `runvil` at commit `b0693699d` and `runvil-plugin-std` at `782ad23`, where `std` is a library Plugin. The rest of the page still describes the commits of the `Code:` bullet.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The text of the page still describes the code at the commits above, when those crates were in `runvil`.
- **Every heading is a section.** On 2026-11-13 the first entry of section 1 was brought to the second change that follows `8de43c2f` in `runvil-harness`, whose plan is `plan/2026-11-13-2-every-heading-is-a-section.plan.md` there. Before it, the Lua blocks under a prompt's one H1 ran once outside every section, as the H1 pass, in a VM of their own that had the `store` table too. That session ran the build and its tests.
- **Update.** On 2026-11-13 the text on file access by tools was brought to `runvil-harness` at commit `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980`. Between the commits, Workshop moved to the `runvil-workshop` repository, and the bashkit Plugin with its `bash` Pack began to ship. The update covers the first entry of section 1, the policy entry of section 4, the entry on a tool call's identity in section 5, three items of section 8, and the Paths bullet. That session read the code and left the build and the tests unrun. The rest of the page still describes the commits above.

*2026-11-10 16:40 - Claude Fable 5.2*
