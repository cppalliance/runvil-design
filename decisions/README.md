# Runvil design decisions

This folder has fifteen pages, one for each part of Runvil's design. A page says what the part is, what it does, and why it is built that way. Every page describes the code as it is at the commits listed below. The pages are for developers who are new to the code, and the developer who will own a part reads its page first.

- Code: `runvil` at commit `fa9936f64`, `runvil-gateway` at `9369a7d`, `runvil-docs` at `cfd69a4`, `runvil-cloud-providers` at `1817cc4`, `runvil-plugin-api` at `63a2403`, and `runvil-plugin-bashkit` at `6a2e36d`
- Repositories: the Engine and the Harness moved to the `runvil-harness` repository after that commit of `runvil`. The text on repositories, crates, dependency rules, and CI describes `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9` (see Limits)
- Plugins: Workshop then moved to the `runvil-workshop` repository. The text on Plugins, Packs, and file access by tools describes `runvil-harness` at `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980` (see Limits)
- Size: 545 entries in about 80,000 words

## The pages

| Page | What it covers | Entries | Gaps | Questions |
|---|---|---|---|---|
| [The prompt language](prompt-language.md) | The prompt file, sections, Lua blocks, the Lua globals, arguments, substitution, `jump` and `call` | 47 | 2 | 9 |
| [Model calls and conversations](models-and-conversations.md) | The `models` API, the model loop, roles, replies, messages, compaction | 45 | 5 | 10 |
| [Tasks and fanout](tasks-and-fanout.md) | Concurrent work in a run, the scheduler's order, determinism, inboxes | 26 | 2 | 5 |
| [Files and the VFS](files-and-the-vfs.md) | The `store` API, the virtual file system, policies, conflicts | 24 | 3 | 8 |
| [The Engine](engine.md) | Parsing, the run state machine, effects, limits, the one error type, the public surface | 40 | 3 | 9 |
| [The Harness and the Host API](harness-and-host.md) | The `runvil` crate, `Host`, `RunSetup`, performing effects, the recorder, the run inbox, stop and cancel | 39 | 7 | 13 |
| [Tools and Packs](tools-and-packs.md) | Tools, Packs, Plugins, the C contract, the shipped Packs | 61 | 5 | 16 |
| [The Gateway](gateway.md) | The Gateway's API, configuration, local models, providers, profiles, boot | 52 | 15 | 10 |
| [Speech](speech.md) | Speech to text, text to speech, voice in Workshop | 39 | 7 | 6 |
| [Workshop: how the app is built](workshop-app.md) | The desktop shell, the server, the protocol, state, the workspace document | 48 | 11 | 9 |
| [Workshop: what the user sees](workshop-ui.md) | Windows, menus, the agent window, the chat box, the look | 32 | 8 | 7 |
| [Crates, build, and CI](crates-build-and-ci.md) | The two workspaces, crate families, dependency rules, structural checks, the toolchain, CI, releases | 49 | 2 | 9 |
| [Testing and debugging](testing-and-debugging.md) | Test support, scripted models, rules for test code, logs and traces | 21 | 4 | 5 |
| [Guides and docs](guides-and-docs.md) | The language guide, the docs site, crate docs | 15 | 5 | 3 |
| [Prompts that ship](shipped-prompts.md) | The sample prompts and the built-in chat prompt | 7 | 2 | 4 |

## Where to start

- **Everyone.** [The prompt language](prompt-language.md), then [The Engine](engine.md) and [The Harness and the Host API](harness-and-host.md). These three define the terms that the other pages use.
- **Packs and Plugins.** [Tools and Packs](tools-and-packs.md) and [Files and the VFS](files-and-the-vfs.md).
- **Agent behavior.** [Model calls and conversations](models-and-conversations.md) and [Tasks and fanout](tasks-and-fanout.md).
- **Workshop.** [Workshop: how the app is built](workshop-app.md), then [Workshop: what the user sees](workshop-ui.md), and [Speech](speech.md) for voice.
- **The Gateway.** [The Gateway](gateway.md) and [Speech](speech.md).
- **Build and release.** [Crates, build, and CI](crates-build-and-ci.md), [Testing and debugging](testing-and-debugging.md), and [Guides and docs](guides-and-docs.md).

## The parts of a page

Every page has the same parts in the same order.

1. A summary of the design in three paragraphs.
2. A walk-through that follows one concrete thing through the code, such as a tool call, a chat request, or a user's message.
3. Numbered sections of entries. An entry's heading states one part of the design, and its body says what the code does and why.
4. Planned work: what is still to be built.
5. Known gaps: places where a guide or a doc comment differs from the code, features that are half built, and code that has only tests as its callers.
6. Questions for the owner: parts of the design whose reason is still to be written down.
7. Method and limits.

## The gaps that matter most

The pages list 81 gaps and ask 123 questions. These eleven gaps matter most.

1. **MCP server text marked trusted.** An adhoc tool that wraps `mcp/get_dynamic_tools` returns the servers' text to the model outside the untrusted envelope and marked trusted, because two trust rules overlap. The evidence is a reading of the code and two tests, and a test with the real `mcp` Pack would confirm it. ([Tools and Packs](tools-and-packs.md))
2. **A language guide that documents fields the code lacks.** The guide documents `sys.execution`, a run's name, and `prompt_hash`. The code lacks all three, and a section that reads `sys.execution` raises an error. ([The prompt language](prompt-language.md), [The Harness and the Host API](harness-and-host.md))
3. **A Gateway guide that differs from the code.** Most of the Gateway page's fifteen gaps are places where the guide says one thing and the code does another. The Speech page lists seven more in the guide's speech chapter, which describes loudness detection where the code runs Silero. ([The Gateway](gateway.md), [Speech](speech.md))
4. **A docs site that leaves out the Gateway guide.** `runvil-docs` has the guide's 11 chapters, and the site build lists only the other two books. ([Guides and docs](guides-and-docs.md))
5. **A restart feature that is half built.** `POST /admin/restart` answers 409 on every Gateway that a shipped launcher starts. The route, the supervised flag, and exit code 75 exist, and the supervisor process is still to be built. ([The Gateway](gateway.md))
6. **Server state that the Workshop UI ignores.** The server computes `chat_ready`, and the UI ignores it, so the chat box can offer send and the microphone while the server reports chat as unavailable. The UI also lacks a sender for the `switch_profile` frame. ([Workshop: what the user sees](workshop-ui.md), [Workshop: how the app is built](workshop-app.md))
7. **Three paths that close unsaved editors.** Open Workspace, Close Window, and Exit skip the save dialog that the other close paths show. The evidence is a reading of the code. ([Workshop: what the user sees](workshop-ui.md))
8. **A toolchain pin that two workflows override.** The repository pins Rust 1.99, and `nightly.yml` and `workshop-installer-smoke.yml` set `RUSTUP_TOOLCHAIN: stable`. ([Crates, build, and CI](crates-build-and-ci.md))
9. **Logs and debug data that Workshop drops.** Workshop lacks a `tracing` subscriber, so its process discards the Harness's log lines. `RunSetup` lacks a switch for the Engine's events that record each model request and response body. ([Testing and debugging](testing-and-debugging.md))
10. **A demo that names one developer's machine.** `language_guide.rs` in Workshop's server mounts the language guide from a folder on one developer's machine, behind a switch that is off. The built-in chat prompt describes a system message that its own Lua has commented out. ([Workshop: how the app is built](workshop-app.md), [Prompts that ship](shipped-prompts.md))
11. **Two APIs that install Packs.** `std` and `bash` reach a Host through the C contract, and `web` and `mcp` reach it through a Rust `install` function, with abilities that the contract lacks: a configuration, Host services, an unavailable Pack with a reason, the call's origin, a tool flag, a wait on outside work, and error causes. The design is one Plugin API, and `insert_plugin`, the function for a Plugin that is compiled into the Host, has tests as its only callers. ([Tools and Packs](tools-and-packs.md), [The Harness and the Host API](harness-and-host.md))

## How the pages were checked

- **Sources.** Each fact on a page was read in a frozen copy of the code at the commits above.
- **Review.** A separate reviewer session for each page opened about 2,150 claims at their sources in the code and the guides, and corrected about 190 of them.
- **Second check.** A further session for each page compared about 700 stated reasons with their sources and corrected 21. It confirmed about 195 facts in the code and fixed 5.
- **The checker.** A script checks each page: every quote is word for word in the code, a guide, or the owner's own words, and the page follows the prose rules of the set.

## Limits

- **Behavior is read from the code.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Error rate.** The reviewers corrected about 9% of the claims they checked. For a sentence outside those checks, the evidence is one session's reading of the code.
- **Currency.** The pages describe the commits listed at the top. The update bullets below name the text that was brought to later code, and the last bullet names the text that still describes older code.
- **Update of 2026-11-11.** The text on Plugins, crates, and dependency rules was brought to `runvil` at `b0693699d`, `runvil-plugin-api` at `03a9fa3`, and `runvil-plugin-std` at `782ad23`, where `std` is a library Plugin that Workshop loads from a Plugin directory and the `shared-*` crate family is gone. The update is on five pages: [Tools and Packs](tools-and-packs.md), [Workshop: how the app is built](workshop-app.md), [Crates, build, and CI](crates-build-and-ci.md), [The Harness and the Host API](harness-and-host.md), and [Files and the VFS](files-and-the-vfs.md).
- **Update of 2026-11-12.** The text on repositories was brought to `runvil` at `48a4dbc08` and `runvil-harness` at `5bca6f9`, where the Engine, Harness, and Plugin crates are in the `runvil-harness` repository and `runvil` pins four of them by commit. The history of `runvil-harness` was written a second time that day, to include the plan and vibe records of the Engine and the Harness, and commit `48a4dbc08` of `runvil` moved the pin to `5bca6f9` in that history. [Crates, build, and CI](crates-build-and-ci.md) has most of it: the two workspaces, their crates, the dependency rules, the place of each structural check, and the CI jobs, with one new entry on the pin. [Workshop: how the app is built](workshop-app.md), [Guides and docs](guides-and-docs.md), and [Prompts that ship](shipped-prompts.md) have the sentences that the move changed. Eight pages have a new Paths bullet, because the code they describe is in `runvil-harness`: [The prompt language](prompt-language.md), [Model calls and conversations](models-and-conversations.md), [Tasks and fanout](tasks-and-fanout.md), [Files and the VFS](files-and-the-vfs.md), [The Engine](engine.md), [The Harness and the Host API](harness-and-host.md), [Tools and Packs](tools-and-packs.md), and [Testing and debugging](testing-and-debugging.md). Each page lists its updated parts in its last section.
- **Update of 2026-11-13.** The text on Plugins, Packs, and file access by tools was brought to `runvil-harness` at `18361d02`, `runvil-workshop` at `f386801b`, `runvil-plugin-api` at `03a9fa3`, `runvil-plugin-std` at `ee04d10`, and `runvil-plugin-bashkit` at `9c8e980`. Workshop is in the `runvil-workshop` repository, which pins `runvil-harness` at `7147e709`, `runvil-plugin-std` at `782ad23`, and `runvil-plugin-bashkit` at `0864f1f`. The update is on [Tools and Packs](tools-and-packs.md) and on six more pages: [Files and the VFS](files-and-the-vfs.md), [The Harness and the Host API](harness-and-host.md), [Crates, build, and CI](crates-build-and-ci.md), [Workshop: how the app is built](workshop-app.md), [Testing and debugging](testing-and-debugging.md), and [Prompts that ship](shipped-prompts.md). The pages state three things. `web` and `mcp` enter a Host through a Rust `install` API, and one Plugin API through the C contract is planned work. The Plugin surface is unfinished and grows a piece at a time. The C contract reads files, and its functions that write and delete a file are still to be built. [Tools and Packs](tools-and-packs.md) gained one entry, on the `bash` Pack, one gap, on the two APIs, and two questions. Its list of places where the language guide differs from the code went from seven to one, after the same session corrected six of them in the working tree of `runvil-docs` over commit `c03ec91`. That session read the code and left the build and the tests unrun. Each page lists its updated parts in its last section.
- **Still to update.** Three groups of text describe older code. The entries on installers, releases, and lints in [Crates, build, and CI](crates-build-and-ci.md) describe the code at the commits listed at the top, and gap 8 above names two workflows that `runvil` has since replaced with `installers.yml`. The text on sections, `jump` targets, and the `prompt` reads in [The prompt language](prompt-language.md) also describes that code. Workshop moved to the `runvil-workshop` repository after the update of 2026-11-12, and the text outside the update of 2026-11-13 describes Workshop's crates as part of `runvil`: the workspaces, crate counts, families, pins, and checks in [Crates, build, and CI](crates-build-and-ci.md), and the Paths bullets and repository names of the pages that the update left alone.

*2026-11-10 16:53 - Claude Fable 5.2*
