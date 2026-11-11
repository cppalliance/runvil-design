# Runvil design decisions

This folder has fifteen pages, one for each part of Runvil's design. A page says what the part is, what it does, and why it is built that way. Every page describes the code as it is at the commits listed below. The pages are for developers who are new to the code, and the developer who will own a part reads its page first.

- Code: `runvil` at commit `fa9936f64`, `runvil-gateway` at `9369a7d`, `runvil-docs` at `cfd69a4`, `runvil-cloud-providers` at `1817cc4`, `runvil-plugin-api` at `63a2403`, and `runvil-plugin-bashkit` at `6a2e36d`
- Size: 543 entries in about 76,500 words

## The pages

| Page | What it covers | Entries | Gaps | Questions |
|---|---|---|---|---|
| [The prompt language](prompt-language.md) | The prompt file, sections, Lua blocks, the Lua globals, arguments, substitution, `jump` and `call` | 47 | 2 | 10 |
| [Model calls and conversations](models-and-conversations.md) | The `models` API, the model loop, roles, replies, messages, compaction | 45 | 5 | 10 |
| [Tasks and fanout](tasks-and-fanout.md) | Concurrent work in a run, the scheduler's order, determinism, inboxes | 26 | 2 | 5 |
| [Files and the VFS](files-and-the-vfs.md) | The `store` API, the virtual file system, policies, conflicts | 24 | 3 | 8 |
| [The Engine](engine.md) | Parsing, the run state machine, effects, limits, the one error type, the public surface | 40 | 3 | 10 |
| [The Harness and the Host API](harness-and-host.md) | The `runvil` crate, `Host`, `RunSetup`, performing effects, the recorder, the run inbox, stop and cancel | 39 | 7 | 13 |
| [Tools and Packs](tools-and-packs.md) | Tools, Packs, Plugins, the C contract, the shipped Packs | 60 | 10 | 14 |
| [The Gateway](gateway.md) | The Gateway's API, configuration, local models, providers, profiles, boot | 52 | 15 | 10 |
| [Speech](speech.md) | Speech to text, text to speech, voice in Workshop | 39 | 7 | 6 |
| [Workshop: how the app is built](workshop-app.md) | The desktop shell, the server, the protocol, state, the workspace document | 48 | 11 | 9 |
| [Workshop: what the user sees](workshop-ui.md) | Windows, menus, the agent window, the chat box, the look | 32 | 8 | 7 |
| [Crates, build, and CI](crates-build-and-ci.md) | Crate families, dependency rules, structural checks, the toolchain, CI, releases | 48 | 2 | 9 |
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

The pages list 86 gaps and ask 123 questions. These ten gaps matter most.

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

## How the pages were checked

- **Sources.** Each fact on a page was read in a frozen copy of the code at the commits above.
- **Review.** A separate reviewer session for each page opened about 2,150 claims at their sources in the code and the guides, and corrected about 190 of them.
- **Second check.** A further session for each page compared about 700 stated reasons with their sources and corrected 21. It confirmed about 195 facts in the code and fixed 5.
- **The checker.** A script checks each page: every quote is word for word in the code, a guide, or the owner's own words, and the page follows the prose rules of the set.

## Limits

- **Behavior is read from the code.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Error rate.** The reviewers corrected about 9% of the claims they checked. For a sentence outside those checks, the evidence is one session's reading of the code.
- **Currency.** The pages describe the commits listed at the top. `runvil` has 13 later commits. They port the cross-platform installer work, move the custom lints to a repository of their own, move `Timestamp` to the root of both crates of re-exports, and change the type of `RunMeta::started_at`. The entries on installers and lints in [Crates, build, and CI](crates-build-and-ci.md) describe the code before those commits.

*2026-11-10 16:53 - Claude Fable 5.2*
