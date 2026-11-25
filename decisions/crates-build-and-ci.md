# Crates, build, and CI: design decisions and their reasons

- Scope: crate names and families, which crate may depend on which, the facades and the check of their public surface, structural checks, lint rules and rulebooks, the toolchain, CI, releases, and installers
- Paths: code paths start at the root of the `runvil-workshop` repository, and a path in another repository starts with that repository's name

## Summary

The Rust crates of the Engine, the Harness, and Workshop are in workspaces, each in a repository of its own. The `runvil-harness` workspace has the crates of the Engine and the Harness, the Rust Plugin crate, and a `build-xtask` crate. The `runvil-workshop` workspace has the Workshop crates and the build crates, and its root `Cargo.toml` pins crates of `runvil-harness` by commit. Another repository, `runvil`, has a workspace of build crates that build the installers and the docs site. A crate's name prefix sets its family. The facades are crates of re-exports: `runvil-engine` for the Engine, and `runvil` for the Harness, the one crate a Host depends on. Each workspace has a crate `build-xtask` with its structural checks: tests that read manifests and source files and fail on a dependency that the family rules refuse, on a facade that defines an item, and on banned attributes. Every crate inherits the lint table of its workspace. Each repository's `ci.yml` runs its jobs behind one required job, and `installers.yml` in `runvil` builds, tests, and publishes the installers. Sections 1 to 9 describe these parts.

Section 11 lists the gaps between this page and the code, and section 12 lists the questions about reasons for the owner. A gap is in the Gateway's containers: a crate at the root of `crates/` in `runvil-gateway` can depend on one of the private crates and pass every structural check.

Section 10 lists the planned work. Its largest item is platform code signing for the installers.

## From a new crate to a release build

A developer who adds a Workshop crate runs `cargo xtask new-crate` with a name that starts with `workshop-`. The scaffolder in `build-xtask` writes these files under `crates/workshop/`: a `Cargo.toml` with `[lints] workspace = true`, a `src/lib.rs` with an `## Invariants` section, and `tests/it/main.rs`. The developer adds the crate's path to `members` in the root `Cargo.toml`, and adds the package to one of the tier tables in `crates/build-xtask/src/tidy.rs` (section 2).

`cargo test -p build-xtask` then runs every structural check, each in a test of its own. `family` classifies the package by its prefix, and `product_boundary_violations` tests each of its dependencies on a workspace crate against the desktop boundary and the container rule. `harness_pin_violations` tests each dependency that names `runvil-harness`. Before a push, `.githooks/pre-push` runs a scan of rulebook wording, clippy with `CARGO_BUILD_WARNINGS=deny`, and `cargo deny`.

On the pull request, `.github/workflows/ci.yml` runs its jobs. `test` runs `cargo nextest` over the workspace, which includes the `build-xtask` checks. `check-workshop` lints and tests the desktop crates on Windows, and `dylint` runs the custom lints. `ci-green` succeeds when every job it lists succeeded. A change to an Engine or Harness crate is a pull request in `runvil-harness`. The `ci.yml` of that repository has its own jobs: among them, `api-surface` runs `cargo xtask api --check` on the pinned nightly, and `msrv` builds Engine crates with Rust 1.95.

The installers are built in `runvil`, from a checkout of `runvil-workshop`. A tag such as `v0.3.0` in `runvil` starts `runvil/.github/workflows/installers.yml`. Its `prepare` job compares the tag with the workspace version and checks out `runvil-workshop` at the commit that the root `Cargo.toml` of `runvil` pins. `build` stages the Gateway release that `gateway-pin/pin.json` of that checkout names and runs `cargo installer --sign` on a runner for each target, one installer for each. `test` installs each installer on a fresh machine of its kind, once for each component selection. `manifest` writes and verifies `latest.json` and `SHA256SUMS`, and `publish-release` creates the GitHub Release and updates the rolling release `workshop-latest`. In `runvil-workshop`, `.github/workflows/bundle.yml` runs `cargo workshop bundle` on the same targets for a pull request that touches the bundle and for a manual run, and the installers are made from that bundle.

## 1. Crate names and families

### A crate's name prefix sets its family

In each workspace, `family` in `crates/build-xtask/src/product.rs` maps a package name to a family. In `runvil-harness`, the prefix `runvil-` means Engine, the exact name `runvil` and the prefix `harness-` mean Harness, and the prefixes `plugin-` and `build-` name the other families. In `runvil-workshop`, the prefix `workshop-` and the desktop app's package `workshop` mean Workshop, and the prefix `build-` means Build. In `runvil`, the prefix `build-` means Build, because every crate there is build tooling. Any other name is `Unaffiliated`.

The owner's goal: "You can immediately tell which crate belongs to which deliverable." A rename and the edit of `family` belong in one change. A crate whose name `family` fails to recognize is `Unaffiliated`, and the checks then skip its family's rules and still pass.

### `runvil` is the one crate a Host depends on

A Host writes against `runvil`, the Harness facade, which re-exports the Harness, the Engine items a Host reads, and the Plugin contract. [The Harness and the Host API](harness-and-host.md) lists what `runvil` re-exports. Workshop, a Host in the `runvil-workshop` repository, names the crate as a git dependency (section 2).

### Each family's private crates are in a container directory

A container is a directory under `crates/` that lacks a `Cargo.toml` of its own. In `runvil-harness`, `crates/runvil-internal/` contains the private Engine crates, and `crates/harness-internal/` the private Harness crates. In `runvil-workshop`, `crates/workshop/` contains the Workshop crates, the desktop app among them. A container crate's directory drops the family prefix, so `runvil-harness/crates/runvil-internal/lua` is the package `runvil-lua`. The root `Cargo.toml` of `runvil-harness` and of `runvil-workshop` excludes its containers and lists each container crate in `members`.

The containers exist so that a maintainer or a coding agent who browses `crates/` can tell from the tree which crates are public API and which are internal machinery.

### Both workspaces prefer many small crates to a few large ones

The Engine's private crates sit behind its facade: `runvil-types`, `runvil-vfs`, `runvil-markdown`, `runvil-prompt`, `runvil-model-client`, `runvil-lua`, and `runvil-runtime`. The root `AGENTS.md` of each repository tells authors to "prefer more, smaller crates to fewer, larger ones".

A larger crate has more interactions between its own parts. A smaller crate is more focused, and the crate is the natural unit of context for a coding session. The boundaries between the Engine's private crates also let Cargo enforce their layering and compile them in parallel.

### The Gateway is a separate product in its own repository

The Gateway's crates are in the `runvil-gateway` repository. `runvil-harness` and `runvil-workshop` contain the clients. In `runvil-harness`, `harness-gateway-client` sends model requests over HTTP. In `runvil-workshop`, `workshop-gateway` is Workshop's HTTP client, and `workshop-gateway-discovery` finds a running Gateway. `runvil-workshop` also contains the tooling that stages a Gateway binary for Workshop's builds, and `runvil` builds the installers that hold that binary, which sections 7 and 9 describe. [The Gateway](gateway.md) covers the Gateway's API, configuration, and model serving.

### Every crate sets `publish = false`, and formats and APIs change freely

Every manifest in both workspaces sets `publish = false`. The owner's reason: "someone is already squatting on runvil in crates.io so I have decided to never publish there, and make signed installers the primary distribution mechanism."

`AGENTS.md` states the rule for compatibility: "There are no users and no recorded runs, logs, or stored data to preserve, so backward compatibility does not matter". Authors change formats, schemas, and APIs freely, with one limit: "never bump a format, schema, or crate version number". An author who removes a feature also removes every trace of it from tests, lints, and comments.

## 2. Which crate may depend on which

### One check reads every manifest against a family matrix

In each workspace, `product_boundary_violations` reads each crate's `Cargo.toml` and tests every dependency on another crate of that workspace: normal, dev, build, and target-specific. In `runvil-harness` it reports each of these dependencies:

1. An Engine crate on a Harness crate.
2. A Harness crate on a `build-*` or unaffiliated crate.
3. A Plugin crate on a workspace crate other than `runvil-plugin`.
4. An Engine or Harness crate on a Plugin crate.

In `runvil-workshop` it reports a dependency that crosses the desktop boundary and a dependency from outside into the `crates/workshop/` container, which later entries describe. The matrix of `runvil` holds only the container rule. A dependency on a crate of a different repository is outside every matrix, and `runvil-workshop` has a second check for the crates it takes from `runvil-harness`.

Each check is a test of its `build-xtask`, so cargo itself enforces the structure of each workspace.

### A crate outside a family depends on that family's public crates

The Engine family's public crates are `runvil-engine` and `runvil-plugin`. The Harness family's are `runvil` and `harness-gateway-client`. The matrix of `runvil-harness` reports a crate outside a family that names any other crate of that family.

A second rule covers the containers. In `runvil-harness`, `runvil-engine` and `runvil-plugin` may depend into `crates/runvil-internal/` from outside it, and `runvil` may depend into `crates/harness-internal/`. In `runvil-workshop`, a `build-*` crate may depend into `crates/workshop/`, and the build crates leave that permission unused.

For the Engine family, the goal of the rule is zero leakage of its private crates. `harness-gateway-client` is a public Harness crate so that a Host can depend on it directly.

### `runvil-workshop` takes crates from `runvil-harness`, all at one commit

The root `Cargo.toml` of `runvil-workshop` names `runvil`, `harness-gateway-client`, and `plugin-mcp` in `[workspace.dependencies]` as git dependencies on `runvil-harness`, each with the same `rev`. `harness_pin_violations` in `crates/build-xtask/src/harness_pin.rs` reads that table and the manifest of every workspace member. It reports an entry from that repository that names any other crate, an entry that lacks a `rev`, a second `rev` among the entries, and a member that names the repository in a manifest of its own. A member takes each of them with `workspace = true`.

The matrix of `runvil-workshop` reads only the crates of its workspace, so the pin has a check of its own. The pinned crates are the public Harness crates and the Rust Plugin that Workshop links in, so Workshop reaches the Engine only through `runvil`, the facade. The Plugin crate is in the pin because `mcp` enters the Host through the Rust `install` API, whose types the Harness shares with it. The `std` Plugin, the bashkit Plugin, and the web Plugin follow the C contract and have pins of their own. The move of `mcp` onto the contract is planned work ([Tools and Packs](tools-and-packs.md)). One `rev` is the rule because two commits of `runvil-harness` in one build are two copies of every type that the pinned crates share. To take new Engine or Harness work, a developer changes the `rev` of every entry in one commit.

### Outside its family, a Harness crate names `runvil-engine` and `runvil-plugin`

The matrix of `runvil-harness` reports a Harness crate that depends on a `build-*` crate or an unaffiliated crate. Outside its family, a Harness crate may name only `runvil-engine` and `runvil-plugin`.

The rule exists so that the Harness can be built, published, or moved out of the workspace by itself, and so that its documentation tree needs only what the docs site contains. The matrix of `runvil-harness` enforces it.

### Workshop crates form tiers below one desktop boundary

`crates/build-xtask/src/tidy.rs` lists each Workshop server crate in one of its tier tables: vocabulary and library Plugins, services, features, and server. [Workshop: how the app is built](workshop-app.md) describes them. A crate depends on crates of lower tiers, and a new `workshop-*` crate joins a tier table before `workshop-server` may depend on it. The library Plugin crates, `workshop-plugin-std`, `workshop-plugin-bashkit`, and `workshop-plugin-web`, depend only on crates outside the Workshop family, and `workshop-server` alone may depend on them. The desktop app, package `workshop`, depends on one Workshop crate, `workshop-server-api`, which depends only on `workshop-server`.

The pin on `runvil-harness`, which an earlier entry of this section describes, keeps the Engine's own crates out of every Workshop manifest, so Workshop imports Engine types from `runvil`.

### A Rust Plugin crate depends on `runvil-plugin`, and no Engine or Harness crate names a Plugin crate

In `runvil-harness`, a `plugin-*` crate may depend on `runvil-plugin` and on libraries from outside the workspace. The matrix also reports an Engine or Harness crate that depends on a Plugin crate, dev-dependencies included.

These checks place every `plugin-*` crate below the Harness: `runvil-plugin`, which defines the Rust Plugin API, is the only Engine or Harness crate that a `plugin-*` crate may depend on. The rule covers the Rust Plugin, `plugin-mcp`. A Plugin that follows the C contract depends on `runvil-plugin-sdk` or `runvil-plugin-sys` from `runvil-plugin-api` and on zero crates of `runvil-harness`, as the `std` Plugin, the bashkit Plugin, and the web Plugin do. One API for both kinds of Plugin is planned work. [Tools and Packs](tools-and-packs.md) covers `runvil-plugin` and the C contract crate `runvil-plugin-sys`.

Each workspace pins its git dependencies by commit. `runvil-harness` has `runvil-plugin-sys` from `runvil-plugin-api`. `runvil-workshop` has the crates it takes from `runvil-harness`, `runvil-plugin-sdk` from `runvil-plugin-api`, the `std` Plugin from `runvil-plugin-std`, the bashkit Plugin from `runvil-plugin-bashkit`, and the web Plugin from `runvil-plugin-web`. `runvil-plugin-sys` sets `links`, and Cargo then refuses two commits of the contract in one build, so `runvil-workshop` pins `runvil-plugin-sdk` at the commit of `runvil-plugin-api` that `runvil-harness` pins.

## 3. Bans on dependencies and features

### An Engine crate that depends on a banned crate fails the check

The banned crates are `tokio`, `tokio-util`, `async-trait`, `reqwest`, `getrandom`, and `fastrand`. `runvil-harness/crates/build-xtask/src/engine_deps.rs` applies the ban to `runvil-engine`, `runvil-plugin`, and every crate under `crates/runvil-internal/`. It scans every dependency table of each manifest, dev-dependencies included. It then runs `cargo tree` for each Engine crate with all features and all targets, and reports a banned crate anywhere in the resolved graph.

The owner's rule: "tokio is not allowed anywhere in the engine". The check makes tokio and operating system entropy impossible in every Engine crate, tests included. [The Engine](engine.md) explains why the Engine leaves all I/O to its caller.

### The Harness facade and its container declare `tokio` only for tests

`runvil-harness/crates/build-xtask/src/harness_bans.rs` reads the `[dependencies]` tables of `runvil` and of every crate under `crates/harness-internal/`, and reports `tokio` or `tokio-util`. Dev-dependencies pass, because the test suites run a Host on tokio. `harness-gateway-client` is outside the ban and depends on `tokio` and `reqwest`.

The facade and its container exclude tokio so that a Host can await a run on any executor.

### A crate enables `test-support` only from `[dev-dependencies]`

Some Engine crates declare a Cargo feature, `test-support`, and compile their test drivers and fixtures under it. Both facades omit a `[features]` table. `runvil-harness/crates/build-xtask/src/test_support_leak.rs` scans the `[dependencies]`, `[build-dependencies]`, and `[features]` tables of every crate and the root `[workspace.dependencies]` table. It reports an entry that enables `test-support` on an Engine or Harness crate, because that entry "would pull the test drivers into a shipping binary".

The Engine and the Harness allow a Cargo feature only for testing, and only inside the workspace. The root `AGENTS.md` states the general rule: a Cargo feature "gates a real constraint such as a toolchain requirement or heavy native build, not product shape".

### Each crate declares the dependency features it uses

A crate that needs a feature of a shared dependency names the feature on its own dependency line, or on its dev-dependency line when only tests need it. The root `[workspace.dependencies]` table pins the versions.

A feature set shared by the whole workspace would compile the test-only feature `test-util` into release binaries, and would put tokio, reqwest, hyper, and GUI crates in the dependency graphs of the Engine crates.

### Workshop links one rustls crypto backend

Both workspaces pin `reqwest` at 0.13, whose `rustls` feature selects the `aws-lc-rs` backend. The `supply-chain` CI job of `runvil-workshop` runs `cargo tree` on the `workshop` package and fails when `ring` appears in its normal dependency graph. The same job of `runvil-harness` runs the command on its whole workspace, so a Host that links those crates gets the one backend too.

Two crypto libraries in one binary would mean two sets of security advisories to follow, and rustls is unable to choose a `CryptoProvider` by itself when both are present. The version pin enforces the rule through feature resolution, because `reqwest` 0.13 lacks a `ring` option.

## 4. The facades and the check of their surface

### A facade's source contains `pub use` lines and documented `pub mod` blocks

Both facades are in `runvil-harness`. The files under `crates/runvil-engine/src` and `crates/runvil/src` of that repository contain re-exports and comments. Each line of code is a single-item re-export of the form `pub use internal_crate::path::Item;`, or belongs to a `pub mod` block with a doc comment that groups such re-exports. `runvil-harness/crates/build-xtask/src/facade_shape.rs` parses both crates and reports a glob, a grouped list, an item definition, or a path that starts outside the facade's `[dependencies]`.

A re-export of a whole module or crate would expose every item that the module gets later, and a second path to an item would make the API ambiguous. [The Engine](engine.md) and [The Harness and the Host API](harness-and-host.md) describe the surfaces.

### `cargo xtask api` proves each surface closed and compares it with a listing

The command belongs to the `build-xtask` of `runvil-harness`. It builds rustdoc JSON for `runvil-engine` and for `runvil`. Its closure check reports a surface item whose signature, fields, bounds, impls, or doc links name something outside the facade's re-exports, the standard library, and the serde crates it allows. Its listing check compares one line per item with the `public-api.txt` in the facade's directory. `--check` fails on a violation or a difference, and `--bless` rewrites the listing.

A developer who adds a `pub use` therefore changes `public-api.txt`, so the change to the surface gets a review. The workspace uses its own command because the `cargo-public-api` tool leaves out the items inside cross-crate re-exports.

### The surface command runs on one pinned nightly toolchain

Every other build uses stable Rust. `PINNED` in `runvil-harness/crates/build-xtask/src/api/toolchain.rs` pairs one dated nightly with `rustdoc-types` 0.61.0, and the command refuses any other toolchain. A developer runs `cargo +<nightly> xtask api --check` with the nightly that `PINNED` names and with `CARGO_BUILD_WARNINGS` unset.

The pin exists because only a nightly toolchain emits rustdoc JSON, and the format of that JSON changes at roughly every release. The `api-surface` CI job reads the nightly's name from that file so that "the two cannot drift".

### `doc(hidden)` fails the check in both facades and both containers

`runvil-harness/crates/build-xtask/src/doc_hidden.rs` reads every attribute in every `.rs` file under the facades, `crates/runvil-internal/`, and `crates/harness-internal/`. It reports `doc(hidden)` on an item, a field, a variant, a method, or a re-export, and inside a macro body.

The dependency rules and the surface check make internal items unavailable to outside code, so `doc(hidden)` would only hide them from a family's own docs, which hurts contributors.

## 5. Structural checks and lints

### `cargo test -p build-xtask` runs every structural check

Each workspace's `build-xtask` has its own checks, and each check has a test that asserts an empty list of violations for its workspace. `cargo xtask tidy` runs them through `all_violations` in `crates/build-xtask/src/tidy.rs` and prints the violations. These checks are in both `runvil-harness` and `runvil-workshop`: the matrix, the `## Invariants` marker, lint inheritance, the unsafe allowlist, the doctest ban, the size limits on types, and the spelling `fmt::Debug` for the `Debug` trait in every `.rs` file, a rule that clippy lacks a lint for. `runvil` has the same checks except the marker. `runvil-harness` adds the Engine guards, the Harness ban on `tokio`, the shape of the facades, and the `doc(hidden)` ban. `runvil-workshop` adds the Workshop tiers and the pin on `runvil-harness`. The entries of sections 2 to 5 describe the other checks.

`AGENTS.md` states what a check may test: "Gate on yes/no properties, never on a score threshold that ordinary work edits."

### Every crate inherits one lint table from the root `Cargo.toml`

The root `Cargo.toml` files of `runvil-harness`, `runvil-workshop`, and `runvil` have the same lint table. Every crate declares `[lints] workspace = true`, and `lint_inheritance_violations` reports a crate that omits it. The table denies `unsafe_code`, clippy's `all` and `pedantic` groups, `unwrap_used`, `expect_used`, `todo`, and `exit`. It also denies `allow_attributes`, so an author writes a suppression as `#[expect(lint, reason = "...")]`.

The level of `unsafe_code` is `deny` because a `forbid` blocks a module-level exception, and a crate with unsafe code would then need a copy of the table. The owner's rule: "I do not want copies of workspace lint. I want the strictest possible settings that still lets us vibe." The allowlist of the next entry limits the exception to the crates it lists.

### Only the allowlisted crates may contain unsafe code

In `runvil-workshop` and `runvil-harness`, `OWNED` in `crates/build-xtask/src/unsafe_allowlist.rs` lists the crates that may: `crates/workshop/desktop` and `crates/workshop/gateway-discovery` in `runvil-workshop`, and `crates/harness-internal/plugin-loader` and `crates/harness-internal/test-plugin` in `runvil-harness`. Each relaxes the lint with `#[expect(unsafe_code, reason = ...)]` on the module that needs it. The check reports an `expect` or `allow` of `unsafe_code` in any other crate, and clippy requires a `SAFETY` comment before each unsafe block.

Any crate can relax a `deny`, so the allowlist check makes the lint as strict as a `forbid` in every other crate. All Plugin loading is in `harness-plugin-loader`, so its unsafe code is in one crate that a reviewer can audit. The loader is outside the facade because a facade contains only re-exports.

### Clippy refuses process-global installers outside a binary entry point

The root `clippy.toml` lists the `disallowed-methods`: `std::panic::set_hook`, the `log` installers, the `tracing` installers, and `rustls::crypto::CryptoProvider::install_default`. The lint table denies `disallowed_methods` and `exit`. A binary's entry point calls one under `#[expect]` with a reason.

A library returns a failure to its caller, and only a binary entry point exits the process or installs process-global state. The rule is a clippy setting because the project prefers a structural check to a line of `AGENTS.md`.

### The Engine `clippy.toml` files ban sleeps, clock reads, and timed waits in Engine code

In `runvil-harness`, `crates/runvil-internal/clippy.toml`, `crates/runvil-engine/clippy.toml`, and `crates/runvil-plugin/clippy.toml` repeat the root entries and add these bans: `std::thread::sleep`, `SystemTime::now`, `Instant::now`, `RandomState::new`, and the timed waits. The bans apply to tests, benches, and examples. `crates/build-xtask/src/engine_clippy.rs` of that repository fails when an Engine crate resolves to another file, when a file lacks an entry, or when Engine source suppresses `clippy::disallowed_methods`.

The files are copies because clippy reads only the nearest `clippy.toml` above a crate's manifest and ignores the root file. [Testing and debugging](testing-and-debugging.md) covers the tests that these bans constrain.

### A doc comment with a compiled code block fails the check

In each workspace, `crates/build-xtask/src/no_doctests.rs` reads every doc comment in every workspace crate. It reports a code block that rustdoc would compile, and a doc attribute that reads its text with `include_str!`. A block tagged `text`, `json`, or `toml` passes.

Doctest steps would add about 67 seconds to every local run of the checks, so the test commands leave them out, and an example is an ordinary test. The check reports a doctest because every test command would skip it.

### Size limits apply to functions and types, and a file may have any length

Clippy fails a function with more than 100 lines or 7 arguments. In each workspace, `crates/build-xtask/src/type_size.rs` fails a type with more than 25 inherent methods in its crate, and a struct with more than 12 named fields. `ALLOWED` lists the types over a limit in each workspace, each with a recorded ceiling and a reason, and `AGENTS.md` calls a new entry "a plan decision with a reason, never a way to make a change pass".

`AGENTS.md` states the rule for files: "A file holds one concept and has no length limit." The limits apply to functions and types because a caller depends on what a type does, whatever file contains it. A limit on the length of a file fails to measure structure, and it causes splits whose only purpose is to fit the limit.

### Engine and Harness crates use the standard module layout

In Engine and Harness crates a production module is the file `foo.rs`, and its children are files such as `foo/bar.rs`. An author merges a module that has exactly one child file, or makes the child a peer. Test code is a `mod tests` block at the bottom of its file, or a sibling `foo-tests.rs` that the module declares with `#[path]`, so a hyphen in a file name marks test code.

The other crates use a flat layout: a subdirectory exists for three or more files, and a group of one or two files is a set of `foo-bar.rs` siblings. `AGENTS.md` leaves those crates on that layout "until their own pass", so their conversion to the standard layout is still to be done.

### Every `workshop-*`, `harness-*`, and `plugin-*` crate has an `## Invariants` section in its crate docs

`marker_violations` reports such a crate when its `src/lib.rs` or `src/main.rs` lacks the line `//! ## Invariants`. The check of `runvil-workshop` binds the `workshop-*` crates, and the check of `runvil-harness` binds the `harness-*` and `plugin-*` crates. The section states the crate's tier, its allowed dependencies, and its structural rules. The check confirms the marker, and authors update the text below it by hand.

## 6. Rulebooks

### An `AGENTS.md` file states the rules that checks leave unenforced

The root `AGENTS.md` of `runvil-workshop` has sections from Definitions to SPA and CSS Rules, the root `AGENTS.md` of `runvil-harness` has sections from Definitions to Structural Rules, and the root `AGENTS.md` of `runvil` has Workshop's sections from Definitions to Structural Rules. A crate's own `AGENTS.md` is a title and a few bullets, and most crates lack one. A line belongs in a rule file when it states policy that the code leaves unstated, when the checks and tests leave it unenforced, and when a breach is costly.

Rule files are short because the code changes faster than they do, and a stale file steers a coding session toward a design that the code has abandoned. A corrected line becomes stale again at the next refactor, while a test fails when the code changes.

### A planned design overrides a rule line that conflicts with it

When a line in an `AGENTS.md` or a `.cursor/rules` file conflicts with the planned design of a change, the design wins, and the commit that creates the conflict deletes the line. The owner's rule: "If any AGENTS.md rule would conflict, then remove the rule. Prefer shrinking AGENTS.md over modifying the planned design."

### Each defined term has one meaning, and a Node test scans the rulebooks

Each root `AGENTS.md` defines the same terms, from Engine to Dynamic Set. Each repository has a copy of the Node test: `crates/workshop/ui/test/docs-claims.mjs` in `runvil-workshop`, and `tools/docs-claims.mjs` in `runvil-harness` and in `runvil`. The test reads every `AGENTS.md`, every `.cursor/rules` file, and every crate doc that has an `## Invariants` section. It fails on a lowercase `host`, `engine`, `harness`, or `plugin` used as a prose word. A second test, which only the copy in `runvil-harness` has, makes Engine crates say "the caller" for the code that steps a run.

Code names follow the same terms. The scan runs in the `ui` CI job of `runvil-workshop`, in the `wording` CI job of `runvil-harness` and of `runvil`, and in each repository's `.githooks/pre-push`. The hook runs it because the cargo commands that check Rust work skip the scan, and a change would otherwise include text that the rule forbids.

### A feature reuses an existing facility before it adds a new one

Under the Principles section of `AGENTS.md`, an author first checks whether an existing facility covers the work, then makes the smallest improvement that enables the feature, and adds a new facility when it has "a material benefit beyond tidiness". The reason is that a person can understand in full a small set of primitives that are used in ten ways, and loses track of ten mechanisms that are each used once.

### The repository's written conventions override the general Rust rulebook

A convention that this repository wrote down overrides the general Rust rulebook, a set of rules from outside these repositories. The overrides include the doctest ban, `pedantic` at deny, error messages written for a model, and container directories. Each one is a documented, forced, or deliberate choice of the repository. [The Engine](engine.md) covers one more written convention, the universal `Error` type of Engine and Harness crates.

## 7. Toolchain and build

### `rust-toolchain.toml` pins Rust 1.99 for local builds and CI

Each repository has the file. It names channel `1.99`, and rustup installs that release for anyone who builds in the repository. The jobs of each `ci.yml` use it too, with these exceptions. `api-surface` and `msrv` in `runvil-harness` set a toolchain of their own, and each `dylint` job builds the lint library on the nightly that the library's own `rust-toolchain.toml` names.

The owner's rule: "I want a rule: local builds and CI use the same environment. I never want CI to fail when local succeeds." The file names an exact release, because a build on the `stable` channel fails on a machine whose stable release is out of date.

### The Engine crates declare Rust 1.95 as their floor

`[workspace.package]` in the root manifest of `runvil-harness` sets `rust-version` to 1.95, and the Engine facade and the internal crates inherit it. The `msrv` CI job of that repository reads the value from the manifest and builds the facade and the internal crates by name with that toolchain, except `runvil-markdown`, which Cargo builds as a dependency of the listed crates. The manifest's comment explains the value: "`runvil-lua` calls the atomics' `try_update`, stable since 1.95". The root manifests of `runvil-workshop` and `runvil` omit `rust-version`.

### A plain `cargo build` in `runvil-workshop` builds every crate except the desktop crates

`default-members` in the root manifest of `runvil-workshop` lists every crate except the desktop crates. It omits the desktop app `workshop`, `workshop-server`, and `workshop-server-api`, which are the crates that need Node.js or the Tauri system packages, so a fresh clone on macOS or Linux builds the rest with only Rust. A developer builds the desktop app by name, with `cargo build -p workshop`. The root manifest of `runvil-harness` omits the setting, and a plain `cargo build` there builds every crate of that workspace.

The setting exists because a build of every crate of `runvil-workshop` fails on a macOS or Linux machine that has only Rust.

### A build writes the UI bundle to `OUT_DIR`

The `build.rs` of `workshop-server` calls `build_sibling` in the `build-ui` crate, which runs esbuild on `crates/workshop/ui` and writes the bundle to `$OUT_DIR/ui-dist/`. `rust-embed` reads that folder from disk in a debug build and embeds it in a release build. A build needs Node.js 22 and one `npm ci` in `crates/workshop`, the root of an npm workspace with the members `ui`, `look`, and `platform`.

Every build step writes outside the folders that git tracks. The helper refuses `npx`, because `npx` can download a different version and produce different output. CI jobs of `runvil-workshop` end with a `Clean tree` step, which fails when `git status` reports a change.

### `cargo workshop` builds the desktop app with a staged Gateway binary

`cargo workshop` is an alias for the `build-workshop` crate. It places a Gateway binary where Workshop finds it, which is the sidecar slot under `crates/workshop/desktop/binaries/` for a Windows target and the target directory beside Workshop elsewhere, and it builds the `workshop` package. It takes the binary from `--gateway-binary`, then from `RUNVIL_GATEWAY_SIDECAR`, then from a file already staged. With all three absent, the command stops and names `node tools/fetch-gateway-sidecar.mjs`, which downloads the release in `gateway-pin/pin.json` and verifies its checksum. `cargo workshop bundle` builds one platform's Tauri bundle with the same Gateway sources, and `cargo installer` in `runvil` makes the installer from that bundle.

The root `AGENTS.md` states the rule for run time: "Runtime and serve paths never compile native dependencies or invoke build tools". [The Gateway](gateway.md) covers how the Gateway obtains its `llama-server` program.

## 8. CI

### Each `ci.yml` runs its jobs behind one required job

Each repository's workflow runs on a push to `master` or `main` and on every pull request. In `runvil-workshop` its jobs are `fmt`, `clippy`, `test`, `docs`, `check-workshop`, `check-workshop-linux`, `ui`, `supply-chain`, and `dylint`. In `runvil-harness` its jobs are `fmt`, `clippy`, `test`, `test-windows`, `docs`, `wording`, `supply-chain`, `api-surface`, `dylint`, and `msrv`. In `runvil` its jobs are `fmt`, `clippy`, `test`, `docs`, `wording`, `supply-chain`, and `dylint`. `ci-green` lists the jobs of its workflow in `needs:` and fails when any result differs from success. The workflow's comment warns that the list is static, so "every newly added job must be added to it". Branch protection then requires one status, `ci-green`.

### The desktop crates have their own Windows job

In `runvil-workshop`, the workspace-wide clippy, test, and docs commands exclude `workshop`, `workshop-server`, and `workshop-server-api`. The `check-workshop` job lints and tests them on `windows-latest`. `check-workshop-linux` compiles the desktop app on Ubuntu 22.04 with the Tauri system packages. Both jobs stage a stub Gateway binary with `node tools/fetch-gateway-sidecar.mjs --stub`.

Their job runs on Windows because the desktop crates need GUI packages that the Ubuntu runner lacks, and Workshop ships on Windows. The Linux job compiles the app so that CI catches the failures of a Linux build, which the Windows job misses.

### CI denies rustdoc warnings and documents the private items of some crates

Each repository's `docs` job sets `RUSTDOCFLAGS` to `-D warnings` and runs `cargo doc` for its workspace. The job of `runvil-harness` also runs it for each facade with default features, and for `runvil-runtime` with `--document-private-items`. The `check-workshop` job of `runvil-workshop` does the same for `workshop-server`. A doc link to an item that does not exist then fails CI.

The steps for private items are in CI because a line in `AGENTS.md` would rely on each author to remember it. [Guides and docs](guides-and-docs.md) covers the build of the docs site.

### CI and the pre-push hook deny warnings through `CARGO_BUILD_WARNINGS`

CI and `.githooks/pre-push` set `CARGO_BUILD_WARNINGS=deny` on the clippy commands, and a local build shows a warning as a warning. Both leave `RUSTFLAGS` unset: on Windows the variable replaces the `+crt-static` setting in `.cargo/config.toml`, and on every system it changes the compiler flags and discards the build cache.

The setting is limited to those two so that a new Rust lint leaves every local build working.

## 9. Releases and installers

### A `v` tag in `runvil` builds an installer for each target

`runvil/.github/workflows/installers.yml` starts on a tag such as `v0.3.0`. Its `build` job runs `cargo installer --sign` on a runner for each target: Windows x64, macOS ARM and Intel, and Linux x64 and ARM64. `cargo installer` runs `cargo workshop bundle` in the `runvil-workshop` checkout and makes the installer from the bundle. The Windows installer is the NSIS setup from that bundle, with the Gateway inside. The macOS installers are DMGs and the Linux installers are `.run` files, built with the Qt Installer Framework, and each installs Workshop and the Gateway as separate components. Each runner first stages the Gateway release that `gateway-pin/pin.json` of the checkout names, and the build refuses a Gateway of another version.

Each product has its own tag prefix, so a tag starts the release of one product. The Gateway's releases use cargo-dist and `gateway-v` tags, which `dist-workspace.toml` in the `runvil-gateway` repository sets.

### A release publishes after every installer passes an install test

The `test` job downloads each installer to a fresh runner of the same kind and runs one cell for each component selection: all, Workshop only, and Gateway only on every platform, and Gateway with Speech to Text on Windows. A cell installs the selection and checks that exactly the selected components are present. It runs `--version` on each installed program and fails when Workshop prints another version than the workspace version or the Gateway another than the pinned one. It checks that Workshop loads its library Plugins, starts the app, and requests its page and `app.js`, which must stay under 3,000,000 bytes, and it requests the Gateway's `/config/` page. The `publish-release` job needs `test` and `manifest`, so a failed cell stops the release.

A manual run of the workflow builds every platform, runs every cell, and stops before `publish-release`, and the installers stay as workflow artifacts for 7 days. A pull request that touches the installer paths runs the `all` and `workshop` cells for each platform. Every mode builds with `--sign`: outside a release the key is one that the `prepare` job generates and nothing trusts, so the `manifest` job checks the same signatures that a release has.

### The rolling `workshop-latest` release has the updater manifest and version-free installer names

After it publishes a versioned release, the `publish-release` job uploads these files to the release named `workshop-latest`. One is `latest.json`, the manifest that the Tauri updater in an installed Workshop reads, which `cargo installer manifest` writes after it verifies each updater signature. Another is a copy of each installer under a fixed name, such as `Runvil-setup.exe` and `Runvil-arm64.dmg`. A release build signs the updater files with a minisign key from GitHub secrets.

The fixed names make each direct download URL stable across versions. The names are on the release that has `latest.json`, because a tag of their own would be a second rolling release to look after. [Workshop: how the app is built](workshop-app.md) covers the updater.

### The Windows installer offers Workshop, the Gateway, and speech to text as components

`crates/workshop/desktop/installer.nsi` is a custom NSIS template with a components page and these sections: `Runvil Workshop`, `Runvil Gateway`, and `Speech to Text`, whose label also states the size of the download. The installer includes the Gateway binary as a Tauri `externalBin`. The speech section has an empty body: the installer records the choice in the registry value `InstallSTT`, which the Gateway reads when it generates its first config. An update run restores the saved choices, stops a running Gateway, and restarts it.

The components allow partial installs: the Gateway only, which is the headless server install, and Workshop only, which is a client that attaches to a Gateway over the local network. The custom template repeats the updater's command-line switches, because the updater otherwise fails silently.

## 10. Planned work

Each of these items is still to be built.

- **Platform code signing.** A Windows certificate and a macOS Developer ID that sign the installers. The installers workflow signs only the updater files, with the minisign key. Unsigned installers block a launch in managed environments.
- **One check command.** A `cargo xtask check` that runs every command of the Verification list in `AGENTS.md`, and Cursor hooks that run it. A developer runs each command of that list, and `.githooks/pre-push` runs a scan of rulebook wording, clippy, and `cargo deny` before a push.
- **Surface listings for more crates.** A `public-api.txt` for `runvil-plugin` and one for `workshop-server-api`. `cargo xtask api` checks the facades.
- **Private-item docs for more crates.** The private-item docs step of section 8 for `runvil-lua` and the Harness crates. CI documents the private items of `runvil-runtime` and `workshop-server`.
- **More determinism bans for Engine crates.** Clippy's `iter_over_hash_type` lint, and bans for `Instant::elapsed`, `RandomState::default`, and `HashMap::new`. The Engine `clippy.toml` files have the determinism bans that section 5 describes.
- **A scaffold for other families.** `cargo xtask new-crate` for a crate outside the Workshop family. The command accepts a `workshop-*` name and refuses any other.

## 11. Known gaps

Each item names two parts of the code or its configuration that disagree.

1. The Gateway's containers. The root `Cargo.toml` of `runvil-gateway` describes `crates/gateway/` and `crates/gateway/stt/` as containers of "the gateway's private crates". `all_violations` in `crates/build-xtask/src/tidy.rs` of that repository runs lint inheritance, a check that each `build.rs` calls the file-size limit, the unsafe allowlist, the walled admin tier, the doctest ban, and the `Debug` spelling. A crate at the root of `crates/` can depend on one of the private crates and pass every check.

## 12. Questions for the owner

Each question asks for the reason behind a part of the design.

**Crates and dependencies**

1. Why does the matrix refuse an Engine or Harness crate that names a Plugin crate?
2. Why is the Harness facade named `runvil`, while the other Harness crates have the prefix `harness-` and the prefix `runvil-` means Engine?
3. Why does `runvil-plugin`, an Engine crate by name, omit `rust-version`, and why does the `msrv` job leave it out?

**Checks and rules**

4. Why does a library leave process-global state to the binary?
5. Why does `AGENTS.md` say "Gate on yes/no properties", and does that rule cover `ALLOWED` in `type_size.rs`? Each entry there records a ceiling for one type.
6. Why do the rule files omit the rule that a planned design overrides a conflicting rule line?

**Toolchain and CI**

7. Why is `CARGO_BUILD_WARNINGS=deny` limited to CI and the pre-push hook? With the toolchain pinned, a new lint appears only when the pin changes, so `.cargo/config.toml` could set it for every build.

**Releases**

8. Why does `installers.yml` require a release tag to name the workspace version 0.3.0? `AGENTS.md` tells authors to "never bump a format, schema, or crate version number".

## 13. Method and limits

- **Checks.** The facts on this page were read in the code. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.

*2026-11-10 16:47 - Claude Fable 5.2*
