# Runvil's Plugin and Tool Names: The Rename Plan Fixes Seven Problems, and Seven Remain Open

- Superseded: the 2026-10-19 Pack and Plugin rename supersedes this report's package and Plugin names.
- Type: evaluation / review of the naming around Plugins, packages, tools, and models
- Audience: the owner (Vinnie Falco), and the agents that will run the rename plan and design the dynamic-library Plugin API
- Decision served: which naming problems the "Plugin Registry and Info Rename" plan fixes, which remain open, and what to decide before the dynamic-library work starts
- Inputs: the 2026-10-18 design conversation between the owner and a Cursor agent; the `runvil` repository at commit `fd91a35b1`; the `runvil-docs` repository at commit `9519407`; the 2026-10-04 design record "Runvil Plugin and Composition Design"; and eight vendor documentation pages, checked on 2026-10-18 and listed under References
- Status: findings and recommendations; the rename plan is ready to run, and nothing in this report has been implemented

## Executive summary

The rename plan fixes the type-level names, and it should run as written. It replaces `PackageRegistry` with `PluginRegistry`, frees "package" for the unit Plugins ship in, calls a Plugin's `vendor/name` its fully qualified name (FQN), and renames the three static-data types to `PluginInfo`, `ToolInfo`, and `ModelInfo`. Those changes resolve seven of the sixteen findings below, with no behavior change and no change to recorded runs.

The plan left nine findings open. A revision of the mlua wrapper report on 2026-10-18 fixed two of them (F8 and F9), and the owner decided a third (F3). The open findings are about the names prompts and models see, and four of the nine are high severity because they bear on designs that come next. "Tool id" assumes every tool is fixed and nameable from Lua, which per-model rendering breaks. "Plugin id" does three jobs at once. "Native" now means two different things. And the 2026-10-04 rule that "a prelude is just pure Lua" conflicts with native preludes. The industry already has words for the missing concept: Google ADK, GitHub's MCP server, and VS Code all call a group of tools a toolset, and MCP selects tools by behavior with annotations such as `readOnlyHint`.

The recommendations are to run the rename plan now; settle the meaning of "native" and the pure-Lua prelude rule before the dynamic-library work; and resolve tool ids, Plugin ids, and wire names in a toolset design before the tools half of the dynamic-library API. The open findings are likely to change the names prompts use. Confidence in that is medium, because the toolset design is not written yet.

## Contents

1. [Five criteria judge every name](#1-five-criteria-judge-every-name)
2. [The review covered the conversation, both repositories, and vendor documentation](#2-the-review-covered-the-conversation-both-repositories-and-vendor-documentation)
3. [Sixteen findings, ranked by severity](#3-sixteen-findings-ranked-by-severity)
4. [Four high-severity findings block or mislead the next designs](#4-four-high-severity-findings-block-or-mislead-the-next-designs)
5. [Five medium-severity findings cause confusion but block nothing](#5-five-medium-severity-findings-cause-confusion-but-block-nothing)
6. [The rename plan resolves seven findings](#6-the-rename-plan-resolves-seven-findings)
7. [Eight observations that are not problems](#7-eight-observations-that-are-not-problems)
8. [Recommendations: run the rename now, and settle the open names in order](#8-recommendations-run-the-rename-now-and-settle-the-open-names-in-order)
9. [Limits of this review](#9-limits-of-this-review)
10. [References](#references)

## 1. Five criteria judge every name

Each finding below fails at least one of these criteria. They come from plain-language practice ("use one term per concept") and from the design constraints the owner set during the conversation.

1. **One term per concept.** A concept has one name, and a name means one concept.
2. **The name says what the thing is.** A reader can tell from the name alone what the thing holds or does.
3. **No collision.** The name does not already mean something else in the repository or in the language guide.
4. **It survives the planned designs.** The name still fits after the changes already decided: one install per Plugin, tools rendered per model just before each request, and Plugins loaded from dynamic libraries.
5. **It matches the industry where an industry term exists.** A reader who knows other AI tooling does not have to relearn a word.

## 2. The review covered the conversation, both repositories, and vendor documentation

The findings draw on four kinds of evidence. The design conversation of 2026-10-18 supplied the owner's requirements and decisions, quoted where they matter. The `runvil` code at `fd91a35b1` and the `runvil-docs` language guide at `9519407` supplied every repository fact, each cited by file and line. The 2026-10-04 design record supplied the earlier prelude rule. Eight vendor pages, checked on 2026-10-18, supplied the industry terms.

The review did not survey users or test any name with a new reader, and it covers only the names that touch Plugins, packages, tools, and models.

## 3. Sixteen findings, ranked by severity

Severity follows impact on the work ahead. A high finding blocks or misleads a design that comes next. A medium finding causes confusion or a collision but blocks nothing. A low finding was a real problem that the rename plan now fixes. Table 1 lists every finding before sections 4 to 6 explain each.

Table 1. Every naming finding, its severity, the criteria it fails, and its status on 2026-10-18

| # | Finding | Severity | Fails | Status |
|---|---|---|---|---|
| F1 | "Tool id" assumes tools are fixed and nameable from Lua | High | 2, 4 | Open |
| F2 | "Plugin id" names three different things | High | 1, 4 | Open |
| F3 | "Native" means an Engine-built Plugin and compiled package code | High | 1, 3 | Decided: `Builtin`; not yet in the plan |
| F4 | "Prelude" is defined as pure Lua, but native preludes are not | High | 1, 4 | Open, needs an owner ruling |
| F5 | "Wire name" is the industry's tool "name" under another word | Medium | 5 | Open |
| F6 | The vendor vanishes at install, and tool ids follow the Host's choice | Medium | 2 | Open |
| F7 | `ModelInfo` will be the third type of that name in the repository | Medium | 3 | Open, accepted |
| F8 | "Library" means three different things | Medium | 1, 3 | Fixed by the plan and the report revision |
| F9 | The mlua wrapper report uses names the rename supersedes | Medium | 1 | Fixed by the report revision |
| R1 | `PackageRegistry` holds Plugins, not packages | Low | 2 | Fixed by the plan |
| R2 | `Package` describes one Plugin, so the container has no name | Low | 1, 2 | Fixed by the plan |
| R3 | `vendor/name` is called a "package name" | Low | 2 | Fixed by the plan |
| R4 | Static data has three different suffixes | Low | 1 | Fixed by the plan |
| R5 | An install error calls a Plugin id a "Plugin name" | Low | 1 | Fixed by the plan |
| R6 | Installing one Plugin twice goes undocumented | Low | 4 | Documented by the plan |
| R7 | The language guide says "model descriptor" | Low | 1 | Fixed by the plan |

## 4. Four high-severity findings block or mislead the next designs

### F1. "Tool id" assumes tools are fixed and nameable from Lua

A tool id names a tool as its Plugin id, a slash, and the tool's own name, such as `web/fetch` (`runvil-docs`, `src/language/12-tools.md`, line 200). Lua offers one tool by that id with `tools.offer('web/search')`, and the call reaches the tool by id whatever name the model saw (`12-tools.md`, lines 816-820). The design assumes every tool exists before the run and keeps one shape for every model.

The owner's requirement breaks that assumption. Tools are rendered for each model just before each request, and they can differ in name, parameters, and semantics. One example is a diff tool for some models and a string-replace tool for others. In the owner's words: "The Lua can never name the tools because it, it can be rendered late." Even selecting a subset by name fails, for example offering only the read-only file tools.

- Evidence: the tool-id rule (`12-tools.md`, line 200), `tools.offer` taking a tool id (line 820), and the owner's statement above.
- Fix: give the model-facing tools an internal identity that the Harness routes by and records for replay, and that prompts never write. Lua offers toolsets and selects subsets by behavior traits, the way MCP marks tools with `readOnlyHint` and GitHub's MCP server exposes only read tools under `--read-only`. Lua's own calls go to stable functions the Plugin defines in its prelude, such as `web.fetch(url)`, rather than to model tools.
- Status: open. It belongs to the toolset design, before the tools half of the dynamic-library API.

### F2. "Plugin id" names three different things

The Plugin id `web` is, at once, the name a Host installs a Plugin under, the unit Lua hands to a model with `plugins.offer('web')`, and the first segment of every tool id. The owner rejected the second use outright: "The plugin ID web, that's like the wrong name because, we need something like a tool bag or a tool set". He gave examples that are sets in their own right: the web tools (search and fetch); a core set of read, write, grep, and string-replace tools that changes per model; and the subagent tools.

- Evidence: `plugins.offer` puts every tool of one Plugin in scope at once (`12-tools.md`, lines 828-830), and `tools.allow_tasks` turns on the task built-ins as a group (`runvil-docs`, `src/language/17-quick-reference.md`, line 210).
- Industry usage: Google ADK has a `BaseToolset` whose `get_tools` returns tools "based on the provided context", plus a `tool_filter`. GitHub's MCP server groups tools into toolsets enabled with `--toolsets`. VS Code defines a tool set as "a reusable collection of agent tools that you can reference as a single entity". LangChain calls the same idea a toolkit.
- Fix: introduce the toolset as its own named concept. A Plugin offers one or more toolsets, and the Plugin id goes back to meaning only where the Plugin is installed.
- Status: open, for the toolset design.

### F3. "Native" means an Engine-built Plugin and compiled package code

`PluginKind::Native` marks a Plugin "built into every registry" whose "behavior will live in the Engine", and today's one native is `runvil/subagents` (`crates/harness-internal/runner/src/packages.rs`, lines 33-42). `AGENTS.md` calls the same kind "Engine-native" (line 12). The dynamic-library design then introduced the "native prelude", meaning compiled code in a package that runs once per section VM and defines globals. A reader who meets "native Plugin" and "native prelude" side by side will assume they are related, and they are not.

- Evidence: the two uses above, and the owner's own uncertainty about the second: "Not sure what you mean by a native prelude".
- Fix: keep "native" for compiled code, which is its common meaning in programming. Rename the Engine-built kind `PluginKind::Builtin` and call it "built-in" in `AGENTS.md`, which matches the existing doc's own words, "Built into every registry". The language guide already says "built-in Plugin packages" for the bundled linked Plugins such as `runvil/web` (`runvil-docs`, `src/language/12-tools.md`, line 36), so that line should say "bundled Plugins" instead, leaving "built-in" for the Engine-built kind alone.
- Status: decided on 2026-10-18. The owner chose `Builtin` ("change Native to Builtin in the rename plan"), to go into the rename plan's commit 3, but the plan does not include it yet.

### F4. "Prelude" is defined as pure Lua, but native preludes are not

The 2026-10-04 design record states an owner decision in section 5.6: "A prelude is just pure Lua. So the prelude can't talk to the Rust code that the capability provides." A native prelude breaks that sentence by design, because Lua calls its functions directly and they run Rust or C. So the word "prelude" would have two incompatible definitions.

- Evidence: the 2026-10-04 record, section 5.6, and the native-prelude design in the mlua wrapper report.
- Fix: the owner rules on it. The agent's recommendation is to allow native preludes only in Plugins that offer no tools. The rule's recorded reason, that anything a prelude does can be done with the same tools, still holds for tool Plugins, and a native prelude's functions touch no capability.
- Status: open, and it needs the owner's ruling before the next step.

## 5. Five medium-severity findings cause confusion but block nothing

### F5. "Wire name" is the industry's tool "name" under another word

Runvil calls the name a model sees the wire name, such as `web_fetch` (`12-tools.md`, lines 192-200). Every major interface calls it the tool's name. An OpenAI function tool has `function.name`, an Anthropic tool has `name`, and an MCP tool has `name`. The id in those interfaces is the id of one call: OpenAI documents a tool call's `id` as "The ID of the tool call", and Anthropic's `tool_use` block has an `id` beside its `name`.

- Fix: once F1 settles tool identity, call the model-facing name the tool name and retire "wire name". The owner already expects the two to drift apart: "Sometimes the wire name will match, other times it won't."
- Status: open, for the toolset design.

### F6. The vendor vanishes at install, and tool ids follow the Host's choice

The vendor exists only as the first segment of a Plugin's FQN. The default Plugin id is the FQN's second segment (`packages.rs`, lines 156-162), so `runvil/web` and `acme/web` both default to `web`. The Host must then install one of them under another id, and prompts never see which vendor they got. Tool ids follow the Plugin id, so a Host that installs `runvil/web` as `browse` turns `web/fetch` into `browse/fetch`.

- Fix: decide in the toolset design whether a Plugin id stays a Host choice, becomes fixed to the FQN's second segment, or is replaced by FQN-based names. The 2026-10-04 record floated tool ids like `runvil/web/fetch`, but nobody decided that.
- Status: open.

### F7. `ModelInfo` will be the third type of that name in the repository

The rename gives the Engine a `ModelInfo`, but the gateway family already has two: `gateway_api_types::ModelInfo` (`crates/gateway-api-types/src/metadata.rs`, line 181) and `gguf::ModelInfo` (`crates/gateway/local/src/gguf.rs`, line 80). No file will see two of them. No file that uses the Engine's `ModelDescriptor` imports either gateway type, and the Harness's gateway client decodes the model list with its own private `ModelsListEntry` (`crates/harness-gateway-client/src/catalog.rs`, lines 12-31).

- Fix: accept the overlap, because the owner chose `ModelInfo` and the families never share a file. Mention the gateway types in the `ModelInfo` doc comment so a search result explains itself.
- Status: open, accepted as a known risk.

### F8. "Library" means three different things

In the language guide, a library is a prompt section loaded with `require` (`runvil-docs`, `src/language/03-blocks-and-prose.md`, "Sharing code with `require`", line 1891). In `AGENTS.md`, a library Plugin is one "loaded from a dynamic library" (line 12). The mlua wrapper report added a third sense: a "library Plugin" as a Plugin with a native prelude and no tools.

- Fix: the rename plan makes "library" describe only how a Plugin is delivered. Its new `AGENTS.md` entry says "library package", and Host-facing text says "dynamic library" in full. The mlua report's third sense goes with F9.
- Status: fixed. The plan removes the delivery sense's ambiguity, and the 2026-10-18 revision of the mlua wrapper report replaced "library Plugin" with "Plugin with a native prelude".

### F9. The mlua wrapper report uses names the rename supersedes

The mlua wrapper report, first titled "Native Library Plugins Should Reach Lua Through a Handle-Based C API over the Engine's mlua" (2026-10-18), declared `runvil_bundle_init`, `RvBundle`, `RvLibraryDesc`, and `register_library`, and called a Plugin with a native prelude a "library Plugin". After the rename, those become:

- `runvil_package_init` and `RvPackage`, since the DLL is a package
- `RvPluginInfo`, matching `PluginInfo`, since it describes one Plugin
- `register_plugin`
- no special name for a Plugin with a native prelude

- Fix: update the report before the next step uses it as input. Settle F3 first, because its outcome decides whether the report keeps the phrase "native prelude".
- Status: fixed on 2026-10-18. F3 kept "native" for compiled code, so the revised report, now titled "Native Preludes Should Reach Lua Through a Handle-Based C API over the Engine's mlua", keeps "native prelude". It uses the names above, renames `RvFunctionDesc` to `RvFunctionInfo` to match, calls the Plugin's name its `fqn`, and passes `plugin_data` and `plugin_id` to `open`. It also moves the version from each Plugin to the package, through a new `RvPackageInfo` and `RvHost::describe_package`, because one build has one version.

## 6. The rename plan resolves seven findings

These were real problems on 2026-10-18. The plan "Plugin Registry and Info Rename" fixes each one, with the owner's words recorded beside the decision.

- **R1. `PackageRegistry` holds Plugins, not packages.** Its own doc calls it "The packages a Host can install, by package name" (`packages.rs`, lines 21-27), yet every entry builds one Plugin. It becomes `PluginRegistry`. Owner: "PackageRegistry was the wrong name. It should have been PluginRegistry".
- **R2. `Package` describes one Plugin, so the container has no name.** The struct is "A Plugin crate's label" (`crates/runvil-plugin/src/plugin.rs`, lines 106-155). The dynamic library needs a name, and one library can hold several Plugins. The struct becomes `PluginInfo`, and "package" names the container: a `plugin-*` crate is a linked package, and a dynamic library will be a library package. Owner: "We have to have a name for the bundle" and "A DLL can have multiple plugins".
- **R3. `vendor/name` is called a "package name".** It becomes the Plugin's fully qualified name, FQN. "Plugin name" was rejected because it sits too close to "Plugin id". Owner: "FQN fully qualified name".
- **R4. Static data has three different suffixes.** A Plugin had a "label" (`Package`), while tools and models had `ToolDescriptor` and `ModelDescriptor`. All three become `Info`: `PluginInfo`, `ToolInfo`, and `ModelInfo`. Owner: "Then I want PluginInfo and ToolInfo" and "I want ModelInfo". The `Plugin` trait keeps its name, because it already owns that name in every Plugin crate and in the runner. `IPlugin` was rejected, because no trait in `crates/` uses an `I` prefix.
- **R5. An install error calls a Plugin id a "Plugin name".** `NameTaken` reads "Plugin name {name} is taken by the installed {existing}" (`crates/harness-internal/runner/src/host.rs`, lines 266-268). After R3, that phrase reads as the FQN, so the message says "Plugin id".
- **R6. Installing one Plugin twice goes undocumented.** Workshop installs `plugin-mcp` once per MCP server (`crates/plugin-mcp/src/lib.rs`, line 4). The owner set the opposite rule: "There can only be one plugin instance. You can't build the same plugin twice." The plan writes the rule into `AGENTS.md`, names `plugin-mcp` as the exception, and defers the MCP redesign and enforcement. Owner: "Now, we're not changing it here."
- **R7. The language guide says "model descriptor".** Chapter 10 describes each catalogued model "by a model descriptor" (`runvil-docs`, `src/language/10-models.md`, lines 354 and 356). It becomes "model info", to match R4.

## 7. Eight observations that are not problems

These came up in the review and need no change, but each one explains why a name works or where it could mislead later.

- **O1. A linked package is a Cargo package.** "Package" therefore agrees with Cargo for linked code, and `build-xtask`'s Cargo sense of the word stays as it is (`crates/build-xtask/src/product.rs`, line 4). A library package is not a Cargo package, so Host-facing docs should say which kind they mean.
- **O2. Lua's `package` library never meets Runvil's packages.** The sandbox never loads the `package` library (`runvil-docs`, `src/language/05-lua-environment.md`, lines 567-570), and prompt authors never see Runvil packages, because `plugins:` takes Plugin ids.
- **O3. `PluginInfo` and the live `Plugin` are one to one.** Under the one-install rule, each `PluginInfo` builds exactly one Plugin object. That repeats the split `ToolInfo` already makes between a tool's static data and the code that performs it.
- **O4. Toolsets already exist without the name.** `plugins.offer` offers a whole Plugin's tools at once, and `tools.allow_tasks` turns on the task built-ins as a group (F2's evidence). The toolset design would name and generalize what is already there.
- **O5. The wording scan fixes the capitalization.** `crates/workshop/ui/test/docs-claims.mjs` (lines 100-118) refuses a lowercase "plugin" in rulebooks, so prose writes "Plugin registry", never "plugin registry".
- **O6. Semantic Kernel's "plugin" is Runvil's missing toolset.** Semantic Kernel defines a plugin as "a group of functions that can be exposed to AI apps and services". A reader coming from Semantic Kernel will expect a Runvil Plugin to be a toolset, which supports giving the toolset its own name (F2).
- **O7. Plurals stay readable.** The rename plan writes "`ToolInfo` values" in prose, never "infos", and uses `info` and plural nouns in identifiers.
- **O8. `PluginKind` keeps its name.** It says where a Plugin comes from, and the next step adds `Library` beside `Builtin` and `Linked`, the names F3's decision gives the other two kinds.

## 8. Recommendations: run the rename now, and settle the open names in order

Each recommendation names an owner and a next step. "The next step" means the API that loads Plugins from dynamic libraries.

1. **Run the rename plan as written.** Owner: Vinnie approves, and an agent runs the three code commits and the docs commit. Confidence is high, because the plan fixes R1 to R7 without changing behavior or recorded runs.
2. **Rename `PluginKind::Native` to `PluginKind::Builtin`, and "Engine-native" to "built-in" in `AGENTS.md` (F3).** Decided: Vinnie chose `Builtin` on 2026-10-18, to go into the rename plan's commit 3, along with "bundled Plugins" for the language guide's "built-in Plugin packages". Next step: an agent updates the plan. Confidence is medium: "native" for compiled code matches common usage, but the change touches a public Harness enum and the `AGENTS.md` definitions.
3. **Rule on the pure-Lua prelude rule (F4) before the next step.** Owner: Vinnie. Confidence is high that the ruling must come first, because the next step's Lua API rests on it.
4. **Correct the mlua wrapper report's names (F9) once F3 is settled.** Done on 2026-10-18; section 5 lists the changes under F9.
5. **Write the toolset design before the tools half of the dynamic-library API, and settle F1, F2, F5, and F6 in it.** Owner: Vinnie, with an agent drafting. On direction, confidence is medium: toolsets offered by name, subsets chosen by behavior traits, and stable Lua functions for direct calls all match the vendor precedents and the owner's requirements, but nobody has designed them in detail. Confidence is high that the design must come before the tools API, because that API would otherwise freeze fixed tool lists across a C boundary.
6. **Mention the gateway's `ModelInfo` types in the Engine's `ModelInfo` doc comment (F7).** Owner: the agent running the rename plan, as part of its second commit. Confidence is high that it is cheap and prevents a confusing search result.

## 9. Limits of this review

One agent judged these names against five criteria, with the owner's statements as the requirements, and no new reader tested any name. Findings F1, F2, F5, and F6 rest on requirements the owner gave in conversation, not on a written toolset design, so their fixes are directions rather than decisions. The vendor pages describe current versions, MCP specification 2025-06-18 among them, and may change. Every line number is from `runvil` at `fd91a35b1` and `runvil-docs` at `9519407`.

## References

Repository facts, from `runvil` at `fd91a35b1`:

- `crates/harness-internal/runner/src/packages.rs`, lines 21-27 (the registry doc), 33-42 (`PluginKind`), and 156-162 (the default Plugin id)
- `crates/harness-internal/runner/src/host.rs`, lines 266-268 (the `NameTaken` message)
- `crates/runvil-plugin/src/plugin.rs`, lines 21-105 (the `Plugin` trait) and 106-155 (`Package`)
- `crates/runvil-internal/types/src/tools/descriptor.rs`, line 24, and `crates/runvil-internal/types/src/models.rs`, line 158
- `crates/gateway-api-types/src/metadata.rs`, line 181; `crates/gateway/local/src/gguf.rs`, line 80; `crates/harness-gateway-client/src/catalog.rs`, lines 12-31
- `crates/plugin-mcp/src/lib.rs`, line 4, and `crates/workshop/server/src/agents.rs`, lines 179-189
- `crates/workshop/ui/test/docs-claims.mjs`, lines 100-118, and `crates/build-xtask/src/product.rs`, line 4
- `AGENTS.md`, line 12 (the Plugin definition)

Language guide, from `runvil-docs` at `9519407`:

- `src/language/03-blocks-and-prose.md`, line 1891, "Sharing code with `require`"
- `src/language/05-lua-environment.md`, lines 567-570
- `src/language/10-models.md`, lines 354 and 356
- `src/language/12-tools.md`, lines 192-200, 816-820, and 828-830
- `src/language/17-quick-reference.md`, line 210

Design records:

- "Runvil Plugin and Composition Design: Conclusions of the 2026-10-04 Exploration", section 5.6
- "Native Preludes Should Reach Lua Through a Handle-Based C API over the Engine's mlua", 2026-10-18, revised the same day to the rename's vocabulary
- The plan "Plugin Registry and Info Rename", 2026-10-18

Vendor documentation, checked on 2026-10-18:

- OpenAI, [Chat Completions API reference](https://platform.openai.com/docs/api-reference/chat/create)
- Anthropic, [Implementing tool use](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/implement-tool-use)
- Model Context Protocol, specification 2025-06-18: [Tools](https://modelcontextprotocol.io/specification/2025-06-18/server/tools) and [Schema](https://modelcontextprotocol.io/specification/2025-06-18/schema)
- Google ADK, [`base_toolset.py`](https://raw.githubusercontent.com/google/adk-python/main/src/google/adk/tools/base_toolset.py) and [Custom tools](https://google.github.io/adk-docs/tools-custom/)
- GitHub, [github-mcp-server README](https://github.com/github/github-mcp-server)
- Visual Studio Code, [Tool sets](https://code.visualstudio.com/docs/agent-customization/tool-sets)
- LangChain, [Tools and toolkits](https://docs.langchain.com/oss/python/integrations/tools/index)
- Microsoft, [Semantic Kernel plugins](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/)

---

*2026-10-18 15:50 - Claude Opus 5.5 (Cursor agent)*\
*2026-10-18 16:00 - Claude Opus 5.5 (Cursor agent), statuses of F3, F8, and F9 updated*
