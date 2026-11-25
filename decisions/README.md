# Runvil design decisions

This folder has one page for each part of Runvil's design. A page says what the part is, what it does, and why it is built that way. Every page describes the code as it is. The pages are for developers who are new to the code, and the developer who will own a part reads its page first.

## The pages

| Page | What it covers |
|---|---|
| [The prompt language](prompt-language.md) | The prompt file, sections, Lua blocks, the Lua globals, arguments, substitution, `jump`, `call`, and `execute` |
| [Model calls and conversations](models-and-conversations.md) | The `models` API, the model loop, roles, replies, messages, compaction |
| [Tasks and fanout](tasks-and-fanout.md) | Concurrent work in a run, the scheduler's order, determinism, inboxes |
| [Files and the VFS](files-and-the-vfs.md) | The `store` API, the virtual file system, policies, conflicts |
| [The Engine](engine.md) | Parsing, the run state machine, effects, limits, the one error type, the public surface |
| [The Harness and the Host API](harness-and-host.md) | The `runvil` crate, `Host`, `RunSetup`, performing effects, the recorder, the run inbox, stop and cancel |
| [Tools and Packs](tools-and-packs.md) | Tools, Packs, Plugins, the C contract, the shipped Packs |
| [The Gateway](gateway.md) | The Gateway's API, configuration, local models, providers, profiles, boot |
| [Speech](speech.md) | Speech to text, text to speech, voice in Workshop |
| [Workshop: how the app is built](workshop-app.md) | The desktop shell, the server, the protocol, state, the workspace document |
| [Workshop: what the user sees](workshop-ui.md) | Windows, menus, the agent window, the chat box, the look |
| [Crates, build, and CI](crates-build-and-ci.md) | The workspaces, crate families, dependency rules, structural checks, the toolchain, CI, releases |
| [Testing and debugging](testing-and-debugging.md) | Test support, scripted models, rules for test code, logs and traces |
| [Guides and docs](guides-and-docs.md) | The language guide, the docs site, crate docs |
| [Prompts that ship](shipped-prompts.md) | The sample prompts and the built-in chat prompt |

## Where to start

- **Everyone.** [The prompt language](prompt-language.md), then [The Engine](engine.md) and [The Harness and the Host API](harness-and-host.md). These pages define the terms that the other pages use.
- **Packs and Plugins.** [Tools and Packs](tools-and-packs.md) and [Files and the VFS](files-and-the-vfs.md).
- **Agent behavior.** [Model calls and conversations](models-and-conversations.md) and [Tasks and fanout](tasks-and-fanout.md).
- **Workshop.** [Workshop: how the app is built](workshop-app.md), then [Workshop: what the user sees](workshop-ui.md), and [Speech](speech.md) for voice.
- **The Gateway.** [The Gateway](gateway.md) and [Speech](speech.md).
- **Build and release.** [Crates, build, and CI](crates-build-and-ci.md), [Testing and debugging](testing-and-debugging.md), and [Guides and docs](guides-and-docs.md).

## The parts of a page

Every page has the same parts in the same order.

1. A summary of the design.
2. A walk-through that follows one concrete thing through the code, such as a tool call, a chat request, or a user's message.
3. Numbered sections of entries. An entry's heading states one part of the design, and its body says what the code does and why.
4. Planned work: what is still to be built.
5. Known gaps: places where a guide or a doc comment differs from the code, features that are half built, code that has only tests as its callers, and faults of the code that are known and still open.
6. Questions for the owner: parts of the design whose reason is still to be written down.
7. Method and limits.

## The gaps that matter most

1. **MCP server text marked trusted.** An adhoc tool that wraps `mcp/get_dynamic_tools` returns the servers' text to the model outside the untrusted envelope and marked trusted, because two trust rules overlap. The evidence is a reading of the code and two tests, and a test with the real `mcp` Pack would confirm it. ([Tools and Packs](tools-and-packs.md))
2. **A Gateway guide that differs from the code.** Most of the Gateway page's gaps are places where the guide says one thing and the code does another. The Speech page lists more in the guide's speech chapter, which describes loudness detection where the code runs Silero. ([The Gateway](gateway.md), [Speech](speech.md))
3. **A docs site that leaves out the Gateway guide.** `runvil-docs` has the guide's chapters, and the site build lists only the other books. ([Guides and docs](guides-and-docs.md))
4. **A restart feature that is half built.** The route `POST /admin/restart`, the supervised flag, exit code 75, and the supervisor exist. The supervisor never restores the `.prev` copies that Apply writes, and the config UI lacks a Restart to Apply button, so nothing calls the route. ([The Gateway](gateway.md))
5. **Server state that the Workshop UI ignores.** The server computes `chat_ready`, and the UI ignores it, so the chat box can offer send and the microphone while the server reports chat as unavailable. The UI also lacks a sender for the `switch_profile` frame. ([Workshop: what the user sees](workshop-ui.md), [Workshop: how the app is built](workshop-app.md))
6. **Paths that close unsaved editors.** Open Workspace, Close Window, and Exit skip the save dialog that the other close paths show. The evidence is a reading of the code. ([Workshop: what the user sees](workshop-ui.md))
7. **Logs and debug data that Workshop drops.** Workshop lacks a `tracing` subscriber, so its process discards the Harness's log lines. `RunSetup` lacks a switch for the Engine's events that record each model request and response body. ([Testing and debugging](testing-and-debugging.md))
8. **A demo that names one developer's machine.** `language_guide.rs` in Workshop's server mounts the language guide from a folder on one developer's machine, behind a switch that is off. The built-in chat prompt describes a system message that its own Lua has commented out. ([Workshop: how the app is built](workshop-app.md), [Prompts that ship](shipped-prompts.md))
9. **Two APIs that install Packs.** `std`, `bash`, and `web` reach a Host through the C contract, and `mcp` reaches it through a Rust `install` function, with abilities that the contract lacks: a configuration, an unavailable Pack with a reason, the call's origin, the `describes_tools` flag, the Host's runtime handle, and error causes. The design is one Plugin API, and `with_plugin`, the builder method for a Plugin that is compiled into the Host, has tests as its only callers. ([Tools and Packs](tools-and-packs.md), [The Harness and the Host API](harness-and-host.md))

## How the pages were checked

- **Sources.** Each fact on a page was read in a frozen copy of the code.
- **Review.** A separate reviewer session for each page opened each claim at its source in the code and the guides, and corrected the claims that were wrong.
- **Second check.** A further session for each page compared each stated reason with its source and corrected the ones that were wrong. It also confirmed facts in the code.
- **The checker.** A script checks each page: every quote is word for word in the code, a guide, or the owner's own words, and the page follows the prose rules of the set.

## Limits

- **Behavior is read from the code.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Evidence.** The reviewers corrected some of the claims they checked. For a sentence outside those checks, the evidence is one session's reading of the code.

*2026-11-10 16:53 - Claude Fable 5.2*
