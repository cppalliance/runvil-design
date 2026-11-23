---
produced: 2026-10-20
title: "Runvil tool offering and MCP design: fixed and dynamic offers of Packs and filtered views, GetDynamicTools and CallDynamicTool, the rendered tool catalog, shaped Packs, MCP as a Pack or an Engine feature, the rendezvous, tool metadata, ask mode, reflection, and 23 open questions"
---

# Tool offering and MCP in Runvil

- Type: design report
- Audience: Runvil maintainers, including new ones
- Decision served: how to finish the tool offering design, and whether MCP is a Pack or part of the Engine
- Status: core design settled on 2026-10-20; 23 questions open

Runvil prompts get tools from several sources: Runvil's own Packs, third-party Packs, and MCP servers. A Pack is a named set of tools, such as `web`. Today a section offers tools one at a time or a Pack at a time, and the Engine rebuilds the tool list before every model round. An offer made between rounds changes the list, and a changed list invalidates the prompt cache. MCP tools reach the model only through two generic tools inside one `mcp` Pack.

Vinnie Falco, who owns Runvil, settled a new design on 2026-10-20. Section Lua offers whole Packs, or filtered views that pick a Pack's tools by metadata or name. Each offer is fixed or dynamic. The model sees a fixed tool's schema directly. It finds a dynamic tool through two discovery tools, `GetDynamicTools` and `CallDynamicTool`, which the Engine adds only when something is offered dynamic. At the section's first model call, the Engine renders every offer into one tool list, the rendered catalog, and attaches it to `messages`, the section's single conversation. The catalog then stays the same until the section ends, and an offer that would change it is a hard error.

Twenty-three questions remain open (Table 1). The largest is whether MCP is a Pack or part of the Engine. Make MCP a Pack whose client adds one tool catalog per server at run time, through a capability that any Pack can use. MCP tools then stay ordinary Pack tools. Ids, offers, filters, and reflection keep one code path each, and the front-matter parser stays as it is. Confidence is medium, because only MCP uses the new capability so far.

Two problems need attention soon. Paperweight, Jason's prompt that judges the evidence a WG21 paper gives for standardization, fails to compile against current `runvil`, because the Host types it imports moved to another module. And Runvil's MCP client still uses the handshake from before MCP revision 2026-07-28, so it misses the `instructions` that newer servers publish.

**Table 1.** Open questions and recommended answers.

| Id | Question | Recommended answer | Confidence |
|-|-|-|-|
| A1 | Offer verb names | `packs.offer` and `packs.offer_dynamic` | Low |
| A2 | What the offer verbs accept | Packs and filtered views; each MCP catalog is a view of the `mcp` Pack | Medium |
| A3 | Filter syntax | A table of metadata keys plus `names` and `except`, combined by AND; only `except` excludes | Medium |
| B1 | When the hard error fires | At an offer that would change the catalog; loosen only with a vendor append API and a supporting evaluation | Medium |
| B2 | What the rendered catalog stores | Schemas, shape mapping, dynamic-set snapshot, discovery-tools flag | Medium |
| C1 | What the model sees of the dynamic set | Names, summaries, tool names, and server instructions, capped, in `GetDynamicTools`'s description | Medium |
| C2 | Discovery tool arguments | Cursor's arguments; search names and descriptions; refusals name an alternative; wire names `get_dynamic_tools`, `call_dynamic_tool` | Low |
| D1 | Loading only named Packs | A front-matter opt-out, such as `extras: false` | Low |
| D2 | Requiring one MCP server | A Lua check in the H1 section | Medium |
| D3 | A required Pack or server is missing | Refuse at start; a required server moves the rendezvous to run start | Medium |
| E1 | MCP as a Pack or in the Engine | A Pack the Host links; any Pack can provide catalogs | Medium |
| E2 | MCP ids and wire names | `mcp/<server>/<tool>`, lowercased; loud collision refusal; every wire name capped at 64 characters with a hash | Medium |
| E3 | How long the rendezvous waits | Host-wide connections; per-server cap, 30 seconds by default | Medium |
| E4 | What an MCP call returns | Data if an output schema is declared, else a string; one size cap | Medium |
| E5 | MCP features beyond tools | Tools; then `server/discover` with fallback, resources, elicitation; drop sampling and roots | Medium |
| F1 | A shaped Pack offered dynamic | A hard error | Medium |
| F2 | Where shapes are kept | In each shaped Pack, keyed by family from provider and model-id prefix | Medium |
| F3 | Evidence for reduced rendering | Offline A/B per family first; Anthropic's native editor starts at full rendering | Medium |
| G1 | Tool metadata and trust | MCP's five annotations plus an open bag, MCP's defaults; trusted only on the Host's word | Medium |
| G2 | Checks before a tool call | Fixed order; every refusal is a tool result naming an alternative | Medium |
| H1 | What Lua can read | Read-only userdata extending today's objects | High |
| H2 | Whether reflection waits | Status at once; catalogs wait for the rendezvous | Medium |
| H3 | What a section reads about its tools | Read-only catalog view with placements, plus a dynamic-usage log | Medium |

## The settled design

Vinnie settled 29 decisions on 2026-10-20, in a huddle with Jason that afternoon and a design discussion that evening (Appendix E). The Engine runs a prompt as a pure state machine, the Harness performs its I/O and records the run, and the Host is the application around both, such as Workshop, Runvil's desktop app.

### Offering

Front matter declares the Packs a prompt needs, and section Lua does all the offering [D 5:54]. A filter can narrow a Pack to a single tool, and any Pack can be offered either way [P1].

A tool offered fixed stays out of the dynamic set, so one Pack can be split, with its read-only tools fixed and the rest dynamic. Vinnie's reason is cost, since discovery "consumes an extra model turn and there are 2 more tools competing for attention" [D 6:15].

Only offers change what the model can discover, and a Lua `tools.call` leaves the dynamic set alone. The Engine owns the discovery tools, because "progressive tool discovery is independent of MCP" [D 6:02].

`tools.offer` by id, `tools.required()`, `tools.extras()`, and `tools.allow_tasks` go away [P1].

### The rendered catalog

The Engine renders late, so shaped Packs can fit the model in use [D 5:27]. The catalog survives compaction and even a change of model [D 6:36]. Vinnie allowed loosening the hard-error rule later "with evidence" [D 6:15].

Rendering once keeps the prompt cache valid. Anthropic's documentation says "Modifying tool definitions (names, descriptions, parameters) invalidates the entire cache" [R1].

### One conversation per section

`messages` is the section's only message list. The Engine creates it along with the section's Lua VM, and Lua cannot make another. A list stays in its section, because "the whole point of Runvil is that sections clear the history!" [D 6:55]. To compact, a Lua compactor fills an empty staging list and returns it. The Engine checks that list and swaps its records into `messages`, which keeps its identity and its catalog. "One conversation per section: implementation spec" gives the rules and tests [R2].

### Shaped Packs

A shaped Pack renders its tools in the form a vendor trained its models on, such as OpenAI's `apply_patch`. The set is small, the core file tools and `subagents`, because it "is tied to how the closed-weight frontier model vendors decide to train their models" [D 5:54].

Lua cannot see shaped tools by name, so it selects them only through metadata filters [D 5:28, 6:02]. When a filter cuts a shaped tool, the Engine renders a reduced form and refuses disallowed calls at dispatch. If the reduced form performs poorly, the tool renders whole, with the same refusals [D 6:15]. MCP tools always render in canonical form.

### MCP

MCP exists in a run only when the Host configures servers. Only the Host supplies that configuration, because of credentials and security [D 6:33]. The MCP client connects to every server in the background when the Host starts, and builds one MCP catalog per server. Lua can offer each catalog fixed or dynamic. MCP inputs are JSON, and outputs can take any form [H 26:14].

The first render, Lua call, or reflection that needs the MCP catalogs waits until every server has listed its tools or failed. That meeting point is the rendezvous. Two details were derived and put to Vinnie, who did not object. A failed server stays out until the Host restarts. And when the Host keeps connections for its lifetime, as Workshop does, only the first run after startup waits.

### Metadata, ask mode, and reflection

Every tool can hold metadata like MCP's annotations [D 5:54]. Ask mode is a read-only mode of the Workshop chat agent. It is a HostService, a typed object the Host hands to Packs, and it applies only to the model's calls. The chat agent's catalog stays fixed for the whole conversation, so ask mode refuses write calls at a dispatch gate, a check the Engine runs before each tool call. Claude Code enforces its plan mode the same way [E1].

Lua can read Packs, their tools, each tool's metadata, and the MCP client's servers [D 6:25]. All of it is read-only, and configuration and credentials stay hidden.

### Rejected options

Vinnie rejected one Pack per MCP server, which he called "broken" [P2]. He also rejected a purpose-built `mcp.promote_server`, offering from front matter, a three-tool discovery bridge, and moving a conversation into another section.

## Design principles

- Building blocks. Runvil offers general pieces, and Lua stays at about 20 percent of a prompt or less. In Vinnie's words, "Runville [Runvil] should offer building blocks not solutions" [H 38:43, 48:26].
- One rule for every case. Every tool source, Pack, and section follows the same rules [P1].
- Same behavior under every Host. A prompt runs the same in Workshop, Paperweight, or any other Host [P1].
- Requirements in front matter. Front matter says what a prompt needs, and Lua decides what the model sees [D 5:54].
- Cache stability. The tool list stays byte-identical for a whole conversation [R1].
- Lean context. Tokens and model turns go only where they improve the result [D 6:15].
- Fit with vendor training. Tools take their trained form where it matters [D 5:54; E2].
- Safe failure. Errors fire at the mistake, and every refusal names an alternative [H 30:18].
- Host control. The Host stays "the master of tools" [H 46:11].
- Replay from the record. A run replays from its record alone, without Plugins or servers [P1].

The sources leave these unranked, and two pairs often conflict. Cache stability and lean context agree on rendering once but disagree about listing the dynamic set up front. Building blocks and one rule for every case conflict when the most general piece needs a special power, as the MCP client does to add catalogs at run time.

## The MCP design

Vinnie asked, "should mcp even be a pack or should be an engine feature?", and wanted both designs worked out [D 6:25, 6:33]. Both keep every settled decision.

In today's code, a Pack's name is fixed at compile time, so a Pack per server is impossible [C1]. A tool id belongs to the Pack its first segment names, so `mcp/paperstore/get_paper` is already a valid id in one `mcp` Pack [C2]. MCP tools join the run late, at the rendezvous, after the Harness has read every other Pack's tools. In both designs the rendezvous becomes an Engine effect, so the Engine stays pure.

### MCP as a Pack

The MCP client stays the Pack `runvil/mcp`, and each server's catalog is a filtered view of it. Unlike other Packs, it adds tools at run time. The Host can link the Pack, as Workshop does today, or every registry can hold it as a built-in, which breaks today's rule that built-ins offer zero tools [C1]. Separately, the power to add catalogs at run time can belong to MCP alone or to any Pack.

With zero servers, the Pack is empty. A Host can also leave a linked Pack out, and then `packs.get("mcp")` returns nil, as for any unknown Pack. A prompt requires a server with a Lua check in its H1 section, the section under the title heading, which runs before all others. A Lua error there stops the run at start, with the author's message [C3].

Lua reaches a catalog through the Pack object, as in `mcp:catalog("paperstore")`, which returns a filtered view. `packs.offer("mcp")` offers every server at once.

Most of the code change is in `plugin-mcp`. It drops its two static tools, adds the server segment to ids, and keeps the annotations and output schemas it discards today [C4]. The Pack contract gains the run-time catalog capability, and the Engine gains the discovery tools.

### MCP built into the Engine

Here the Engine owns MCP's concepts: server catalogs as a second kind of tool source, an `mcp:` front-matter key, and a Lua `mcp` table. The Engine still leaves all I/O to the Harness, so the Harness takes over the client and performs connections and calls as effects [P1].

A prompt requires a server with `mcp: [paperstore]`, and the Harness checks it before the run starts. Front matter rejects unknown keys today, so the parser changes [C5]. Lua lists servers with `mcp.servers()` and gets one with `mcp.server("paperstore")`. Ids can take the form `mcp.paperstore/get_paper`, which is valid today and gives the same wire name.

MCP tools stop being Pack tools, so ids, offers, and reflection each need an MCP path, and a prompt that offers everything needs a second loop for servers. The Engine gains effect kinds, a front-matter field, and a reserved `mcp` global. This is the largest code change.

### Hybrids

One hybrid keeps MCP as a built-in Pack and adds the Engine design's `mcp:` key. Another lets a `uses:` entry be a map, such as `- mcp: { servers: [paperstore] }`, as Continue's configuration does [E3]. MCP would be the first user of that syntax, and Vinnie disliked "mcp-specific syntax to the uses" [D 6:30].

### Trade-offs

Every design renders once, leaves placement to Lua, and records MCP calls for replay. They differ in how a prompt requires a server and whether a Host can leave MCP out (Table 2).

**Table 2.** MCP designs compared.

| | Pack, catalogs for MCP only | Pack, catalogs for any Pack | Built into the Engine | `mcp:` key over a built-in Pack |
|-|-|-|-|-|
| Requiring a server | H1 check | H1 check | `mcp: [paperstore]` | `mcp: [paperstore]` |
| Requirement visible without running | No | No | Yes | Yes |
| Special cases | Catalog power for MCP alone | One general Pack capability | MCP ids, offers, and reflection | A built-in that offers tools, plus a key |
| Host can leave MCP out | Yes | Yes | No | No |

Make MCP a Pack the Host links, and let any Pack provide catalogs. One extras loop then offers every tool source, and the work outside `plugin-mcp`, which every design changes, stays small. A front-matter key can be added later if Hosts need to read requirements, while a second kind of tool source would be hard to take back out. The cost is that the requirement is written in Lua, so a prompt that needs MCP behaves the same under every Host only if it checks for its server. Vinnie's first instinct went the other way, toward "a feature of the language" with an `mcp` key [D 6:30].

Appendix A shows three real prompts in both designs.

## Open questions

### A1. What should the offer verbs be called?

Every author types these calls. Once `tools.offer` by id goes, they are the only way to offer tools.

- `packs.offer` and `packs.dynamic_offer`, which Vinnie proposed with "Maybe?" [D 6:15]. Adjective first.
- `packs.offer` and `packs.offer_dynamic`. Verb first, like `tools.offer_local`.
- `packs.offer(p, "dynamic")`. One verb, plus a string to misspell.

Use `packs.offer` and `packs.offer_dynamic`. Verb first matches `tools.offer_local`, and the pair sorts together. Confidence is low, because this is a matter of taste. Next, pick the spelling before the language guide's tools chapter is rewritten.

### A2. What can the offer verbs accept?

MCP catalogs and local Lua tools each need a defined way into the offer verbs.

- Packs and filtered views, with each MCP catalog a view of the `mcp` Pack. Works if MCP is a Pack.
- MCP catalogs as Pack-like values. Works if MCP is in the Engine.
- Separate `mcp.offer` verbs. Doubles the verbs.

Accept Packs and filtered views only, and treat each MCP catalog as a view of the `mcp` Pack. Keep `tools.offer_local` fixed only, because a local tool exists for the model in its own section. Confidence is medium, because the answer depends on the MCP design. Next, settle it together with that decision.

### A3. What syntax should filters use?

Filters pick tools by metadata or name. On shaped Packs they can match only metadata.

- A table, such as `p:where{ readOnlyHint = true }`. The Engine can inspect and record it.
- A method chain, `p:only(...)` and `p:except(...)`. More methods to learn.
- A Lua predicate. Flexible, but opaque in the run record.
- Glob strings. Blind to metadata.

Use the table. Keys combine by AND, `names` is an allow list, and `except` is a deny list that wins. Every surveyed harness excludes tools through a separate deny rule like `except`. OpenAI's Responses API filters MCP tools with a similar table, `{"read_only": true, "tool_names": [...]}` [E4]. Confidence is medium, because the shape is untried in Runvil. Next, write the grammar beside the metadata vocabulary.

### B1. Where does the hard error fire?

The open part is where the Engine raises the error. A library section that several sections load may offer `web` each time, and that repeat is harmless.

- At any offer after first render. Breaks shared library sections.
- At the next round, if the catalog would differ. Reports the error at the loop, away from the offer that caused it.
- At the offer, only when it would change the catalog.

Raise at the offer when it would change the catalog, and treat an identical re-offer as a no-op. Loosen the rule only when a vendor API can append tools after the cached prefix, as Anthropic's and OpenAI's tool search do [E5], and an evaluation shows accuracy holds. Confidence is medium, because "identical" still needs a definition. Next, define offer identity as the same Pack, filter, and placement.

### B2. What does the rendered catalog store?

Dispatch must map a shaped call back to its Pack's tool, even after a model change. Replay needs the exact bytes sent.

- The schemas alone. A later model's calls lose their route back to the canonical tools.
- Schemas, shape mapping, dynamic-set snapshot, and a flag for the discovery tools.
- Tool ids and filters, re-rendered on demand. Fragile, since the bytes must match forever.

Store schemas, mapping, snapshot, and flag as one run-record entry at first render, so dispatch can route a later model's calls through the stored mapping. Confidence is medium, because the mapping's format depends on where shapes are kept. Next, define the entry beside the run record's existing entries.

### C1. What does the model see of the dynamic set?

A listing costs tokens on every request. Without one, the model spends a turn searching or decides a tool does not exist. In one Cursor session the name list alone cost about 2,884 tokens [R1].

- Names, one-line summaries, and tool names per Pack or catalog, as Cursor shows.
- Names only.
- An empty description. Hermes added a listing after models kept "declaring a capability nonexistent instead of calling `tool_search`" [E6].

List each dynamic Pack and MCP catalog with a one-line summary, its tool names, and its server instructions, inside `GetDynamicTools`'s description. Apply a hard cap with a truncation marker. The description belongs to the rendered catalog, so the cache stays valid and the system prompt stays the author's. Confidence is medium, because the cap needs measuring. Next, measure the listing for Workshop's Packs plus four MCP servers.

### C2. What do the discovery tools accept and return?

Their wording affects how well models use discovery, and their refusals affect whether a model recovers from a wrong call.

- Cursor's arguments, `GetDynamicTools(namespace?, toolName?, pattern?)` and `CallDynamicTool(namespace, toolName, arguments)`. Today's `mcp` Pack copies them, minus `pattern`.
- Search first, like Anthropic's tool search.
- A plain listing by namespace. Long for a big dynamic set.

Use Cursor's arguments, with `pattern` matching descriptions as well as names, and `namespace` taking Pack ids and catalog names. Results are canonical schemas under a size cap. A call outside the dynamic set gets a refusal that names an alternative, such as "web_fetch is offered directly; call it by name". The wire names are `get_dynamic_tools` and `call_dynamic_tool`, the tool names today's `mcp` Pack already uses. Confidence is low, because these shapes are unmeasured. Next, write both descriptions and test them on one Anthropic and one OpenAI model.

### D1. Can a prompt load only the Packs it names?

A run gets its declared Packs plus every undeclared `runvil` Pack. The undeclared ones are its extras. Vinnie raised a Paperweight Host pointed at an installer's Plugin directory, which would pick up Packs the prompt never uses [H 18:08].

- Keep today's behavior.
- A front-matter opt-out such as `extras: false`, after Vinnie's "only load named packs" [D 6:15].
- Flip the default. Safer for pipelines, but the chat agent relies on extras.

Add the opt-out and keep today's default, so existing prompts keep their meaning. Goose recipes that name extensions load only those [E7]. The flag controls loading only, and built-in Packs such as `subagents` stay available. Confidence is low, because today only Runvil's own Packs join undeclared, and a prompt can ignore extras by leaving them unoffered. Next, add the key to the front-matter schema and apply it where the Harness picks a run's Packs.

### D2. How does a prompt require one MCP server?

Paperweight needs `paperstore`, an MCP server for WG21 papers, and should stop at start without it. Jason and Andrew Wheeler's rule is that "A prompt that depends on a tool should fail loudly where the tool is missing" [R1].

- A Lua check in the H1 section.
- An `mcp: [paperstore]` front-matter key. Readable without running, but a key for one kind of source.
- Options on `uses:` entries. Only MCP would use them, and Vinnie dislikes MCP syntax there [D 6:30].

Use the H1 check, which raises when the catalog is missing. A Lua error in the H1 section already stops the run before any other section starts [C3]. The same check serves other requirements, and the parser stays as it is. The cost is that a Host learns the requirement only by running the prompt, which goes against the principle that requirements belong in front matter. Confidence is medium for that reason. Next, write Paperweight's check once its Host can configure `paperstore`.

### D3. What happens when a required Pack or server is missing?

A pipeline should stop fast with a clear reason. A chat agent should keep working when an optional server is absent.

- Refuse the run at start, as Runvil does for required Packs.
- Continue with a notice.
- Fail at the first call to the missing tool, as today.

Refuse at start, and let a required server trigger the rendezvous at run start, so the run waits for a slow server and stops for a failed one. The H1 check does both, because reading the catalogs there is their first use. Codex's `required = true`, the only such flag among twelve surveyed harnesses, exists to "make startup fail if this enabled server can't initialize" [E8]. Confidence is medium, because the wait limit is a separate question. Next, test Paperweight with `paperstore` missing, and with it slow.

### E1. Should the MCP client be a Pack or part of the Engine?

The answer determines whether MCP tools are ordinary Pack tools or a second kind of tool source, and where a prompt states that it needs a server.

- A linked Pack, with run-time catalogs for MCP alone.
- A built-in Pack, which every Host then includes.
- A linked Pack, with run-time catalogs open to any Pack.
- Built into the Engine, with an `mcp:` key and a Lua `mcp` table.
- A hybrid of an `mcp:` key and a built-in Pack.

Make MCP a Pack the Host links, and let any Pack provide catalogs, for the reasons under "The MCP design". Confidence is medium. Next, write the catalog capability into the Pack contract, beside the planned per-run Pack instance [P1].

### E2. What ids and wire names do MCP tools get?

MCP tool names may use uppercase letters, which tool ids refuse. OpenAI caps function names at 64 characters, while Runvil's wire names can be any length [R3]. When two wire names collide today, the Engine drops the later tool with only a log line [C6].

- A reserved segment, `mcp/paperstore/get_paper`. Fits today's types.
- A prefix, `mcp.paperstore/get_paper`. Needs Pack ids named at run time.
- A separate id type that keeps case. A second identity scheme.

Use the reserved segment. Lowercase the name, keep the original for the call, and refuse a colliding tool loudly, naming both. Cap every wire name at 64 characters, ending longer names with an 8-character hash, as Codex, Cline, and Qwen Code do [E8]. Confidence is medium, because 64 is the strictest common cap. Next, add the cap and the refusal where the Engine binds wire names.

### E3. How long may the rendezvous wait?

The rendezvous waits for every server, so one slow server delays the first answer of every prompt that uses MCP. Today each server gets two minutes [C7].

- Keep two minutes.
- A per-server timeout in the Host's configuration.
- One cap over all servers.

Keep connections Host-wide, and cap each server at 30 seconds by default, as Claude Code does (Appendix C). A server that misses the cap stays out until the Host restarts. Confidence is medium, because 30 seconds is unmeasured on Runvil's servers. Next, add the cap as a field of the Host's MCP configuration.

### E4. What does an MCP call return?

The client joins text blocks and uses `structuredContent` only when the content list is empty. An image-only result becomes a placeholder, and a script gets a string even when the server sent typed data [C8].

- A string for Lua and the model.
- Data for Lua when the tool declares an `outputSchema`.
- A result object with text, data, and content blocks.

Give Lua `structuredContent` as data when the tool declares an `outputSchema`, since MCP says "Servers MUST provide structured results that conform to this schema" [E9]. Otherwise give Lua a string. Give the model the text blocks, or the serialized `structuredContent` when text is missing. Apply one size cap to both, and keep results marked untrusted. Confidence is medium, because servers vary outside declared schemas. Next, keep `outputSchema` when the client lists tools.

### E5. Which MCP features beyond tools should the client support?

The client uses only tools. It connects with `rmcp`'s older `initialize` handshake, so it skips `server/discover` and misses the `instructions` that modern servers publish there [C9].

- Tools only.
- Resources and prompts.
- Elicitation, through the user-input Pack.
- Sampling and roots.

MCP revision 2026-07-28 deprecates sampling, roots, and logging. Elicitation is its only remaining core client feature, and stateless requests replace the `initialize` handshake [E9]. Support tools now, and switch the client to `rmcp`'s `Auto` lifecycle, which tries `server/discover` first and falls back to `initialize`. Add resources next, then elicitation. Drop sampling and roots. Confidence is medium, because how existing servers answer a discovery-first client is untested. Next, switch to `ClientLifecycleMode::Auto` and test against the GitHub and paper store servers.

### F1. Can a shaped Pack be offered dynamic?

A call through `CallDynamicTool` loses the top-level form the model was trained on. Subagent tools are the hard case, because they are large. Cursor's `Task` tool costs 1,946 tokens, and Cursor keeps it behind discovery [R1].

- A hard error whenever a shaped Pack is offered dynamic.
- Allowed, with canonical or shaped descriptions in lookups.
- Per Pack, with file tools always fixed and subagent tools allowed either way.

Make it a hard error that says to offer the Pack fixed or leave it out, as Vinnie's instinct suggested [D 6:15]. Every surveyed harness keeps its file tools loaded. Claude Code deferred its file tools and its `Agent` subagent tool in v2.1.69, users reported failures, and v2.1.72 restored them [E8]. That is the only report on deferring a subagent tool. Confidence is medium, because the evidence is strong for file tools and thin for subagent tools. Next, add the check to the dynamic offer verb.

### F2. Where are shapes kept, and how does the Engine learn the model family?

The Engine needs the model's family at first render. It knows only an optional `provider` string, and only the Gateway, the process that routes model calls, knows the tool dialect [C10].

- In each shaped Pack, keyed by family.
- A central Engine table, which then has to know every shaped Pack's tools.
- The Gateway's tool dialect. A dialect names a wire protocol, which several model families can share.

Store shapes as data in each shaped Pack, so a Pack's shapes change with its tools. At first render, the Engine resolves the family from `provider` plus the longest matching model-id prefix, as Codex does [E8]. An unknown model gets the canonical form. Confidence is medium, because prefix matching breaks when vendors rename models. Next, pass the resolved model id and provider into the render step.

### F3. What evidence decides between reduced and full rendering?

The settled fallback to full rendering applies on "poor performance" [D 6:15], and that term still needs a test. Published tests compare whole formats only. Edit format alone moved one model from 20 to 61 percent on Aider's benchmark [E10].

- Case by case, when someone reports a problem.
- An offline A/B evaluation per model family before a reduced form ships.
- Runtime telemetry that sets a per-model choice for later runs.

Run an offline A/B per family first, measuring pass rate, malformed calls, and tokens over at least 30 runs per arm [R3]. Telemetry corrects mistakes more slowly. For Anthropic's native editor, start at full rendering with refusals. That editor is schema-less and "can't be modified" [E11], so a reduced form would have to imitate it. Confidence is medium, because the method is standard but has yet to run. Next, build the A/B harness on the file-tools Pack.

### G1. What metadata does `ToolInfo` gain, and does Runvil trust it?

Filters and the ask-mode gate need metadata that `ToolInfo` lacks [C11]. The MCP client also drops the annotations servers send, so a tool its server marks read-only still fails a read-only filter [C4].

- MCP's five annotations, `readOnlyHint`, `destructiveHint`, `idempotentHint`, `openWorldHint`, and `title`.
- Runvil's own names, mapped from MCP.
- MCP's five as typed fields, plus an open key-value bag.

Use MCP's five as typed fields plus an open bag, and let `tools.offer_local` accept optional metadata. A missing field takes MCP's cautious default, so a tool counts as destructive until it says otherwise. MCP says "clients MUST consider tool annotations to be untrusted unless they come from trusted servers" [E9], so a server's annotations count only when the Host marks it trusted. Confidence is medium, because an untrusted server's read-only tools then stay out of ask mode until a Host vouches for it. Next, add the fields to `ToolInfo` with serde defaults, so recorded runs still load.

### G2. What does the Engine check before a tool call, and what does a refusal say?

Several settled rules refuse calls at dispatch. Today a call to an unknown name ends `models.loop` with an `out_of_scope_tool` error [C12].

- A fixed order, or one ordered by cost.
- A refusal the model reads as a tool result, or an error that ends the loop.
- `CallDynamicTool` calls counted under the underlying tool, or under the discovery tool.

Check in a fixed order. The name must be in the catalog, a `CallDynamicTool` target must be in the dynamic set, the call must use an allowed part of a shaped tool, and ask mode must allow it. Leave argument checks to the Pack or server. Make every refusal a tool result, in text the Engine writes, that names an alternative, as Cline's plan-mode refusal does [E8]. Count `CallDynamicTool` calls under the underlying tool's id. Confidence is medium, because changing the out-of-scope outcome breaks tests that pin it today. Next, write the four refusal texts and test them on one Anthropic and one OpenAI model.

### H1. What can Lua read about Packs, tools, and MCP servers?

A prompt could check whether a newer paper store adds a function before calling it [H 41:49]. Today a tool object shows three fields.

- Read-only userdata with new fields. Two reads give the same object.
- Table snapshots. Lua can write to them, and the writes would mislead.
- JSON strings. Awkward to read.

Extend today's userdata objects through `__index`, `__len`, and `__pairs`. Lua code cannot write to such objects by any documented route, and the sandbox removes `rawset` [C13]. Add status, metadata, schemas, wire names, placement, and server instructions, and keep configuration, credentials, and shaped tool names hidden. Confidence is high, because this extends the shape the code already uses. Next, add `metadata`, `placement`, and `status`.

### H2. Does reading MCP catalogs wait for the rendezvous?

Reading a catalog is a first use, so it waits. A prompt that only shows server status should get its answer at once.

- Every MCP read waits.
- Status reads return at once, and catalog reads wait.
- An explicit `mcp.wait()`.
- Catalogs read as empty until the rendezvous ends.

Return server names and status at once, with status `loading` before the rendezvous, and make catalog reads wait. Authors would forget an explicit wait, and empty catalogs would make a required-server check depend on timing. Confidence is medium, because a status can change a moment after it is read. Next, match the `status` values to today's `ready`, `loading`, and `error`.

### H3. What can a section read about its own tools?

Authors need to see what the model had and what it used. The team plans to develop with extras offered dynamic, watch which ones the model calls, then declare those Packs and offer them fixed [R1].

- Today's `tools.calls` counters only.
- A read-only view of the rendered catalog, with each tool's placement.
- That view plus a log of the dynamic tools the model called.
- The run record only, read after the run.

Give each section the catalog view plus the dynamic-usage log. A section can adapt only to what it reads while running, so the run record alone falls short. The run record stores the same data anyway, because replay needs it. Confidence is medium, because the record grows with the catalog, which the listing cap bounds. Next, add a dynamic-usage entry beside today's tool-call records.

## What to do first

1. Fix Paperweight's import. It imports `HostContext`, `HostServices`, and `PackageRegistry` from `harness::plugin`, which now holds the library-Plugin loader. The types moved to `harness::pack`, and `PackageRegistry` became `PackRegistry` [C14]. The next bump of Paperweight's `vendor/runvil` submodule, pinned at `07e7e01b1`, fails to compile until the import changes. In the same change, replace Paperweight's bare `fanout` call with `tasks.fanout`, the only fan-out function the Engine installs [C15]. Confidence is high, because both fixes are mechanical and checked against the code.
2. Switch the MCP client to `rmcp`'s `Auto` lifecycle. This change stands alone.
3. Settle the MCP design, since the offer verbs, loading, and the other MCP questions depend on it.
4. Build one conversation per section from its spec. Attach today's tool list, rendered once, so this ships before the new offers exist. Confidence is high for the rule and medium for its details.
5. Render once, attach, and raise on change. Reserve an empty place for the shape mapping in the catalog's record.
6. Add tool metadata, then the offer verbs and the filter table.
7. Add the discovery tools and the dispatch checks.
8. Move MCP to run-time catalogs, with the H1 checks, the naming rule, the rendezvous cap, and the result handling.
9. Add reflection and the run-record entries.
10. Shape Packs last, after the A/B evaluation.

## Limits

Most outside evidence comes from vendors. Of 36 quantified claims in the 2026-10-14 harness report, 31 were vendor-reported or self-published, and every vendor figure is still unreplicated [R3]. The 64-character cap and the 30-second default come from other harnesses, and the listing cap still needs a value. Code claims come from reading the code at pinned commits, chiefly `runvil` at `b1bbe33eb`, without running prompts. Later `runvil` commits removed `tools.allow_tasks`, the model's task tools, and task notices. Quotes from the auto-generated huddle transcript fix known garbles in square brackets.

These points remain unverified:

- The MCP protocol version that `plugin-mcp` sends, probably `2025-11-25`.
- How Runvil's older client fares against a server that speaks only the 2026-07-28 protocol.
- That the chat agent's `tools.offer(tools.extras())` offers only the two MCP tools.
- That Paperweight fails at its bare `fanout` call at run time.
- Four harness behaviors, including why Claude Code restored its `Agent` tool in v2.1.72.

## Appendix A. Worked examples

This Lua is illustrative. `packs.offer_dynamic` stands for the dynamic offer verb, whose name is open, and the catalog methods and the file-tools Pack `files` are placeholders.

### Paperweight

Paperweight's command-line Host builds a registry of native built-ins only, so the Host must first add the MCP Plugin with a `paperstore` entry. That Host starts fresh in each process, so every Paperweight run waits at the rendezvous.

With MCP as a Pack, the H1 section checks for the catalog. That read is the catalog's first use, so the rendezvous happens at run start.

```lua
models.default("writer")
local mcp = packs.get("mcp")
if not (mcp and mcp:catalog("paperstore")) then
  error("paperweight needs the paperstore MCP server in the Host's MCP configuration")
end
```

A library section wraps the tool, a pattern Vinnie kept from his Slack sketches because "we can see like PS.get paper, that looks a lot nicer" [H 47:46]. Any section loads it with `require`.

````markdown
## paperstore

```lua
local ps = {}
function ps.get_paper(id)
  return tools.call("mcp/paperstore/get_paper", { paper_id = id })
end
return ps
```
````

```lua
local ps = require("## paperstore")
local md = ps.get_paper("p4003")
```

With MCP in the Engine, the requirement moves to front matter, and the wrapper calls through the server object.

```yaml
name: paperweight
runvil: 0
mcp: [paperstore]
```

```lua
function ps.get_paper(id)
  return mcp.server("paperstore"):call("get_paper", { paper_id = id })
end
```

Either way `ps.get_paper` returns a string, since the paper arrives as Markdown text.

### A web-research prompt

Jason's Agora prompt offers the web Pack fixed and MCP dynamic, so its model uses whatever data sources the Host has [H 32:14]. That is Vinnie's case for discovery, "When you don't know ahead of time what the data sources are" [H 25:08].

```lua
packs.offer("web")
local mcp = packs.get("mcp")
if mcp then
  packs.offer_dynamic(mcp)
end
messages:user(prose)
models.loop(messages)
```

With MCP in the Engine, the prompt offers each server's catalog in a loop.

```lua
packs.offer("web")
for _, server in ipairs(mcp.servers()) do
  packs.offer_dynamic(server.catalog)
end
```

### The Workshop chat agent

The chat agent offers the file tools, web, and subagents fixed, and every extra Pack dynamic. With MCP as a Pack, the extras loop covers `mcp` too. With MCP in the Engine, the agent also needs the server loop from the web-research prompt.

```lua
packs.offer({ "files", "web", "subagents" })
for _, pack in ipairs(packs.extras()) do
  packs.offer_dynamic(pack)
end
while true do
  messages:user(input.ask())
  pcall(function() return models.loop(messages) end)
end
```

### Split offers and filters

The `github` Pack is hypothetical, after Vinnie's 18-tool example [D 6:02]. The first offer places the read-only tools fixed. The second places the rest dynamic, because a tool offered fixed stays out of the dynamic set.

```lua
local gh = packs.get("github")
packs.offer(gh:where{ readOnlyHint = true })
packs.offer_dynamic(gh)
```

This pair offers one tool fixed and everything else dynamic except `delete_repo`.

```lua
packs.offer(gh:where{ names = { "list_commits" } })
packs.offer_dynamic(gh:where{ except = { "delete_repo" } })
```

A review section offers the shaped file tools read-only, through a metadata filter.

```lua
packs.offer(packs.get("files"):where{ readOnlyHint = true })
```

### Reflection

Server status reads at once. The catalog read waits for the rendezvous. Here a prompt checks for a newer paper store function before using it.

```lua
for _, server in ipairs(packs.get("mcp"):servers()) do
  log(server.name .. ": " .. server.status)
end
local has_search = false
for _, tool in ipairs(packs.get("mcp"):catalog("paperstore").tools) do
  if tool.name == "search_papers" then has_search = true end
end
```

## Appendix B. One MCP catalog from configuration to call

The URLs and the `examples` server are illustrative. MCP revision 2026-07-28 gives `getUser` and `admin.tools.list` as valid tool names [E9].

```json
{
  "mcpServers": {
    "paperstore": { "url": "https://paperstore.example/mcp" },
    "github": {
      "url": "https://github.example/mcp",
      "headers": { "Authorization": "Bearer ${env:GITHUB_TOKEN}" }
    },
    "examples": { "url": "https://examples.example/mcp" }
  }
}
```

The Host hands `mcpServers` to the MCP client, as Workshop does today. At the rendezvous, each server's tool list becomes an MCP catalog that keeps every tool's annotations and output schema. Each tool gets a lowercased id, and its wire name replaces `/` and `.` with `_` [C6]. The client keeps the server's own name for the call.

**Table B1.** Sample MCP tools as ids and wire names.

| Server | Tool as listed | Id | Wire name |
|-|-|-|-|
| `paperstore` | `get_paper` | `mcp/paperstore/get_paper` | `mcp_paperstore_get_paper` |
| `github` | `list_commits` | `mcp/github/list_commits` | `mcp_github_list_commits` |
| `examples` | `getUser` | `mcp/examples/getuser` | `mcp_examples_getuser` |
| `examples` | `admin.tools.list` | `mcp/examples/admin.tools.list` | `mcp_examples_admin_tools_list` |

If the `examples` server also listed `getuser`, it would collide with `getUser` after lowercasing, and the client would refuse it and name both. A wire name over 64 characters keeps its first 55 characters and adds an underscore and an 8-character hash, as Cline does [E8].

A model's call to `mcp_paperstore_get_paper` maps through the rendered catalog to `mcp/paperstore/get_paper` and passes the dispatch gates. The Harness routes it to the `mcp` Pack by the id's first segment, and the Pack sends `tools/call` to `paperstore` with the name `get_paper`. A Lua `tools.call` of the same id takes the same route, minus the model.

## Appendix C. Per-harness details

Surveyed on 2026-10-20 [E8]. "Undocumented" means the sources read are silent on the point.

**Table C1.** Per-harness practice.

| Harness | Deferred tools | Modes | MCP names and cap | Per-model tools |
|-|-|-|-|-|
| Claude Code | MCP; file tools and `Agent` loaded again since v2.1.72 | Fixed list; plan mode enforced at call time | `mcp__<server>__<tool>`, 128 | Undocumented |
| Codex CLI | MCP, when search is on | Approval and OS sandbox at execution | `mcp__<server>__<tool>`, 128 with hash; per-server `required` | Model catalog, longest slug prefix |
| Cursor | `Task` and MCP, behind discovery | Read-only Ask mode; mechanism undocumented | Undocumented | Undocumented |
| Gemini CLI | Loads everything | Policy engine excludes and refuses | `mcp_<server>_<tool>`, 63 | Undocumented |
| OpenCode | MCP, through Code Mode | Tool list per agent, recomputed per request | `<server>_<tool>`, uncapped | `patch` for GPT models, `edit` and `write` otherwise |
| Cline | Loads everything | Plan preset removes editors | `<server>__<tool>`, 64 with hash | Routing table by provider and model id |
| Qwen Code | MCP and rarely used built-ins, including `agent` | Not surveyed | `mcp__<server>__<tool>`, 63 with hash | Not surveyed |
| VS Code | All but about 30 core tools | Tool list per mode | `mcp_<prefix>_<tool>`, 64 | Not surveyed |
| Goose | Loads everything | Swap to a read-only extension | `<extension>__<tool>`, uncapped | Not surveyed |

Startup limits differ. Claude Code's `MCP_TIMEOUT` defaults to 30 seconds per server, Codex's `startup_timeout_sec` to 10 seconds with a 1-second grace for optional servers, and Qwen Code's remote discovery to 5 seconds. Goose waits for its extensions before the first message, and Gemini CLI starts without waiting.

When a trained tool both reads and writes, such as an editor, every surveyed harness renders the whole tool. Every one also accepts tool definitions as servers send them, without pinning.

## Appendix D. Glossary

| Term | Meaning |
|-|-|
| Agora | Jason's web-research prompt, known here from the huddle |
| Ask mode | A read-only mode of the Workshop chat agent, provided as a HostService |
| Canonical form | A tool as its Pack defines it, before any shaping |
| Compactor | The Lua function that fills a staging list when a conversation overflows |
| Discovery tools | `GetDynamicTools` and `CallDynamicTool`, Engine built-ins present only while the dynamic set holds tools |
| Dispatch gate | A check before a tool call that can refuse it with text the model reads |
| Dynamic offer | An offer that puts tools in the dynamic set |
| Dynamic set | Tools the model reaches only through the discovery tools; older documents say "withheld" |
| Engine | Runvil's pure, deterministic interpreter, which leaves all I/O to the Harness |
| Extras | A run's Packs beyond those its front matter declares |
| Filtered view | A subset of a Pack's tools, chosen by metadata or name |
| Fixed offer | An offer whose tool schemas go into the rendered catalog |
| Gateway | The separate process that holds vendor credentials and routes model calls |
| H1 section | The section under a prompt's title heading, which runs first; only `argv` and `var` outlive it |
| Harness | The part of Runvil that performs model calls, tool calls, and other effects, and records the run |
| Host | The application that runs a prompt and decides which Packs exist, such as Workshop |
| HostService | A typed object the Host hands to Packs |
| MCP catalog | One MCP server's tool list |
| MCP client | The code in the `plugin-mcp` crate that talks to MCP servers and provides the `mcp` Pack |
| Pack | A named set of tools with a one-segment id, such as `web` |
| Paper store | `paperstore`, an MCP server for WG21 papers |
| Paperweight | Jason's prompt, with its own command-line Host, that judges the evidence a WG21 paper gives for standardization |
| Plugin | A loaded crate or library that adds Packs |
| Rendered catalog | The tool list sent with a section's model requests, fixed at the first one and attached to `messages` |
| Rendezvous | The point where the first use of MCP catalogs waits for every server to list its tools or fail |
| Run catalog | Every tool the run's Packs can serve, fixed at run start |
| Run record | What the Harness records so a run replays without Plugins or servers |
| Shape mapping | The record, kept with the rendered catalog, that turns a shaped call back into its canonical tool |
| Shaped Pack | A Pack whose tools render in the form a vendor trained its models on |
| Staging list | The empty list the Engine hands the compactor to fill |
| Tool id | A tool's canonical name, lowercase segments with the Pack id first, such as `web/fetch` |
| Wire name | The name the model sees, such as `web_fetch` |
| Workshop | Runvil's desktop app, whose chat agent is the main interactive prompt |

## Appendix E. Decision log

Vinnie Falco's design statements in the discussion of 2026-10-20, Pacific time. Spelling is as written. Ellipses mark cuts, and drafting instructions are left out. The 6:36 PM and 6:50 PM statements answered question dialogs and were confirmed verbatim separately.

- 5:27 PM: "tools.offer adds the tool to a running "offer catalog" which is rendered once for the agent session, just before the tools are first offered to the OpenAI-compatible endpoint (or equivalent). They are rendered late for the case where a Plugin Pack's tools are model-shaped (tools whose name, description, parameters, or semantics are modified to adapt to a specific vendor's model, due to training)"
- 5:28 PM: "shaped tools will be anonymous, they wont be callable in the Lua"
- 5:54 PM: "1. no front matter solutions allowed. prefer solutions which are general and flexible. "withhold" is kind of specific. ... 2. my gut tells me, not to tie the discovery to the actual usage of tools in Lua. Only tools.offer should affect discovery, in the sense that if a tool or pack is explicitly offered then it should not be also discoverable. and if there is no discoverable tool set then the discovery tools (GetDynamicTools, CallDynamicTool) should be squelched from the late-rendered tool catalog 3. MCP Tools are never model-shaped. Only specific packs are model-shaped: core tools like ReadFile, WriteFile, Grep, Rg, the subagents pack (those are not even individually offerable). and actually that's all I can think of. The number of model-shaped packs is small and enumerable. It is tied to how the closed-weight frontier model vendors decide to train their models. 4. I was thinking MCP annotations should be applicable to pack tools as well, not just MCP tools (i.e. allow each pack tool to have metadata). ... 5. Ask-mode is a specific feature for a specific prompt (i.e. the general Agent window prompt used in Workshop). it is a HostService probably only offered to that one prompt. ... It would not be used in things like paperweight. So it would not need to apply to Lua calls. And we just need to trust."
- 6:02 PM: "shaped packs have to be filterable. or else how could we apply readOnlyHint=true as a filter to the core tools (allow ReadFile, disallow WriteFile, disallow StrReplace for example) progressive tool discovery is independent of MCP. The 2 discovery tools are built-in to the engine and they appear only when a model is offered the withheld tool set. The idea here is to make the mcp client treat its catalog as just another source of tools, the server acts like a "pack", and the Lua controls whether these tools are progressive or fixed. And I want to shape Runvil so that tools that do not come from mcp can become discoverable. For example a "github" pack that has 18 tools for interacting with github. the Lua should be able to offer the entire pack as a rendered catalog, or it should be able to offer the pack as progressive discovery."
- 6:15 PM: "for now, any attempt to change the tool catalog after it is rendered the first time should result in a hard error. we can always loosen this later, with evidence. what this means when a filter splits a shaped tool, is that we should render it reduced, and refuse disallowed calls at dispatch. however, if this results in poor performance (because the model was trained on the full description) then we have to fall back to full rendering, and refusing the disallowed operation at dispatch. For 2 I agree, filters already allow matching a tool subset so why not a subset of 1 named tool. the shaped-tool packs will just not offer any named tools to the Lua, so it will not be able to select one in the first place. for 3. how about packs.offer and packs.dynamic_offer ? Maybe? I am thinking maybe the frontmatter needs to have a flag that tells the harness "only load named packs. ignore available packs that are not in 'uses'. 4. if a tool is offered fixed, it never can be discovered. this prevents wasteful overlap and waste in general, because discovery consumes an extra model turn and there are 2 more tools competing for attention (the discovery tools) 5. my instinct is that shaped packs are never progressive. ... subagents also, they have to be explicit offerings I think. ..."
- 6:20 PM: "The mcp client is going to be either a builtin pack or a linked pack or whatever, because it is special, it creates tool catalogs and puts them in the dynamic set which is something a regular tool cannot do. The mcp client on startup will connect to the servers in the background, but if it gets called then it has to wait until all the servers are finished. so it doesnt have to block on startup but it does have to block on the first toolcall. Not block, but you know, introduce latency in the first response. This helps startup go faster because the mcp client is connecting servers asynchronously while the prompt is getting started. and then the prompt and the mcp client rendezvous the first time the mcp client's tool catalog is needed. ... I like dynamic tool set / dynamic set / DynamicTools much better than "withheld {anything}" naming ..."
- 6:25 PM: "there is an open question: should mcp even be a pack or should be an engine feature? and the availability of the feature depends on if there are servers. no servers = no mcp. I also want reflection designed. ... The pack. The pack's list of tools. The tool's data. Metadata. The mcp client's list of servers."
- 6:30 PM: ""uses: [mcp.paperstore]" I dont know if I like that. Shouldn't we instead make mcp a feature of the language, and then make "mcp" a front-matter key? because you adding mcp-specific syntax to the uses which feels wrong in general"
- 6:33 PM: "Now that I think about it.. the dynamic MCP config is a bad idea., cut it. I forgot about credentials. and security. its a headache we don't need. I am not certain MCP should be a built-in language feature. ..."
- 6:36 PM: "The rendered tool catalog is attached to the messages array the first time the catalog is rendered and then it sticks. if the model changes, so be it but we can't redo the tools. If the messages are compacted, the rendered catalog has to be transfered to the new messages array."
- 6:49 PM: "a message list can't cross a section boundary"
- 6:50 PM: "I might even say, that you cannot create more than one message list in a section?" Then: "Consider it as an option. The Engine would create `messages` when it creates the VM. compaction would receive a 2nd empty messages array to insert into or something like that"
- 6:53 PM: "if messages becomes the section's list itself, then there is no constructor. the Lua cannot create a new one. the compaction function will be given a 2nd constructed empty messages object that it can fill, which it is then required to return. ..."
- 6:55 PM: "no one is "ferrying history to a new section" - the whole point of Runvil is that sections clear the history! if someone wants history preserved, they stay in the section"
- 6:55 PM: "I like your swap idea for compaction"
- 6:57 PM, on one conversation per section: "we can settle it yes ..."

## References

[D 6:15] is Vinnie Falco's design discussion of 2026-10-20 at that time, Pacific, PM; Appendix E quotes it. [H 29:43] is an offset into the auto-generated recording of his huddle with Jason that afternoon. Code paths are in `runvil` at `b1bbe33eb` unless an entry says otherwise.

Code

- C1: `crates/runvil-plugin/src/registry.rs` lines 32-34 and 195-198, and `crates/runvil-plugin/src/pack.rs` lines 128-132 and 150-154. Pack names are fixed at compile time, and built-ins offer zero tools.
- C2: `crates/runvil-internal/types/src/tools/ids.rs` lines 10-13 and 26-32, and `crates/runvil-internal/types/src/packs.rs` lines 67-76. Tool id segments, their charset, and Pack ownership by first segment.
- C3: `crates/runvil-internal/engine/src/execute/scheduler/h1.rs` lines 1-10. A Lua failure in the H1 section stops the run as `Error::RequirementsUnmet`.
- C4: `crates/plugin-mcp/src/dynamic.rs` lines 294-315 and `crates/plugin-mcp/src/server.rs` lines 48-68. The two static MCP tools, and the fields the client keeps from each listed tool.
- C5: `crates/runvil-internal/parser/src/frontmatter.rs` line 43, `deny_unknown_fields`.
- C6: `crates/runvil-internal/engine/src/execute/context-bound.rs` lines 40-79. Wire names, and colliding tools dropped with a log line.
- C7: `crates/plugin-mcp/src/connect.rs` lines 18-19 and 84-85, and `Cargo.toml` line 116. The two-minute startup deadline, `client_config().serve(transport)`, and `rmcp` pinned at 3.5.1.
- C8: `crates/plugin-mcp/src/result.rs` lines 72-82, MCP call results.
- C9: `rmcp` 3.5.1, `src/service/client.rs` lines 639-655 and `src/model.rs` lines 156-201. Legacy initialization under `serve`, and `server/discover` only in the `Discover` or `Auto` lifecycle.
- C10: `crates/runvil-internal/types/src/models.rs` lines 163-169, the optional `provider`.
- C11: `crates/runvil-internal/types/src/tools/info.rs` lines 24-43, `ToolInfo`.
- C12: `crates/runvil-internal/engine/src/execute/scheduler/chat.rs` lines 349-391, the scope gate and `out_of_scope_tool`.
- C13: `crates/runvil-internal/lua/src/globals.rs` lines 22-24, the sandbox's removal of `rawset`.
- C14: `wg21-paperflow` at `693ef8ab`, `crates/paperweight/src/app.rs` line 22, and `runvil` `crates/harness/src/lib.rs` lines 28-52. Paperweight's import, and the modules that now hold those types.
- C15: `crates/runvil-internal/lua/src/coro.rs` line 259 (`tasks.fanout`), and `wg21-paperflow` `crates/paperweight/paperweight.md` line 534 (the bare `fanout` call).

Plans and reports

- P1: "Library Plugin loading", 2026-10-20 (`runvil/vibe/2026-10-20-1-library-plugin-loading.md`).
- P2: "Pack and Plugin rename", 2026-10-19 (`runvil/vibe/2026-10-19-2-pack-plugin-rename.md`), lines 45 and 623-628.
- R1: "Tool Offering in runvil: Three States per Plugin, a Built-in Discovery Facade, and a Frozen Tools Array", 2026-10-14 (`runvil-design/research/`).
- R2: [One conversation per section: implementation spec](2026-10-21-one-conversation-per-section-spec.md), 2026-10-21 (`runvil-design/research/2026-10-21-one-conversation-per-section-spec.md`).
- R3: "What to steal: how popular harnesses handle MCP servers and dynamic tools, and how models are trained for them", 2026-10-14 (`runvil-design/research/`).

External, accessed 2026-10-20

- E1: Claude Code, [prompt caching](https://code.claude.com/docs/en/prompt-caching) and [permissions](https://code.claude.com/docs/en/permissions).
- E2: Anthropic, [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works); OpenAI, [Codex prompting guide](https://developers.openai.com/cookbook/examples/gpt-5/codex_prompting_guide).
- E3: Continue, [Configuring models, rules, and tools](https://docs.continue.dev/guides/configuring-models-rules-tools).
- E4: OpenAI, [connectors and MCP guide](https://platform.openai.com/docs/guides/tools-connectors-mcp), `allowed_tools`.
- E5: Anthropic, [tool search](https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool); OpenAI, [tool search](https://developers.openai.com/api/docs/guides/tools-tool-search).
- E6: Hermes, [tool search](https://hermes-agent.nousresearch.com/docs/user-guide/features/tool-search).
- E7: Goose, [issue 6678](https://github.com/block/goose/issues/6678).
- E8: Per-harness sources, held with line-level citations in the 2026-10-20 research collections. Main ones: Claude Code [MCP](https://code.claude.com/docs/en/mcp), [env-vars](https://code.claude.com/docs/en/env-vars), and [issue 31002](https://github.com/anthropics/claude-code/issues/31002); Codex [MCP docs](https://developers.openai.com/codex/mcp) and `openai/codex` at `d571eaf`; Cline `cline/cline` at `627e51e`; Gemini CLI at `d00859d`; OpenCode at `aadd4d8`; Qwen Code at `6e6598e`; VS Code [release 1.118](https://code.visualstudio.com/updates/v1_118); Goose at `7143ddd`; Cursor's [Ask mode](https://cursor.com/help/ai-features/ask-mode) and first-hand observation.
- E9: MCP specification, revision 2026-07-28: [tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools), [changelog](https://modelcontextprotocol.io/specification/2026-07-28/changelog), and [discover](https://modelcontextprotocol.io/specification/2026-07-28/server/discover).
- E10: Aider, [unified diffs](https://aider.chat/docs/unified-diffs.html).
- E11: Anthropic, [text editor tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/text-editor-tool).

*2026-10-21 07:11 - Claude Opus 5.5 (Cursor agent)*
