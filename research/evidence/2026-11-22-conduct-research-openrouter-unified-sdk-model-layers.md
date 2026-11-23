---
produced: 2026-11-22
title: "Unified model layers in open-source SDKs - how Vercel AI SDK, Pydantic AI, LangChain, TensorZero, rig, genai, pi, opencode, Goose, and Zed hold vendor state (thinking signatures, Gemini thought signatures, OpenAI encrypted reasoning, server tools) behind a neutral message type - fill evidence for the OpenRouter deep dive"
---

# Unified model layers in open-source SDKs: how each holds vendor state behind a neutral message type

- **What was read:** the source of ten open-source projects that put one neutral message type over several model vendors, each at a pinned commit cloned on 2026-11-22: Vercel AI SDK, Pydantic AI, LangChain (two repositories), TensorZero, rig, genai, pi, opencode, Goose, and Zed. One issue report and three workspace documents were also used.
- **How many sources:** 23 (19 source-code areas across 11 repositories, 1 issue report, 3 workspace documents). 73 evidence items, 28 candidate matrix rows, and a 10-row comparison table.
- **Read date and basis:** everything was read on 2026-11-22. Every item is labeled `Filled` with the project named, except H73, which is labeled `Workspace`. None of it shows what OpenRouter's closed translator does. Nothing was run against a model.

## Candidate matrix rows

How to read this table. This file holds fill evidence only, so the column "How OpenRouter exposes it" says "see OpenRouter evidence" in every row. The column "What OpenRouter loses or flattens" is used here for how the surveyed projects hold the feature and what they lose, which is what a fill can show. The native rule in each row is the rule as the projects' own code and comments state it, or as the workspace research recorded it. I did not re-fetch vendor documentation for this file (see Unverified).

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Signed thinking block | Anthropic | A `thinking` block goes back with its `signature`, unmodified and in order. An empty signature is "never valid" (Pydantic AI comment) | see OpenRouter evidence | Projects hold it five ways: `{anthropic: {signature}}` on the part (AI SDK, opencode), a `signature` field plus `provider_name` (Pydantic AI), the raw block as content (LangChain), a `Thought` with `provider_type` (TensorZero), the raw block JSON beside a neutral block (rig). genai pairs text and signature by position and drops both when the counts differ | Caller holds it. Round trip unchanged. No value for the caller to pick. Fits a per-reply block list | Filled (AI SDK, Pydantic AI, LangChain, TensorZero, rig, genai, opencode) | H5, H6, H26, H28, H37, H45, H50, H57, H66 |
| Redacted thinking | Anthropic | A `redacted_thinking` block has `data` and no signature, and must go back | see OpenRouter evidence | Four different encodings: `redactedData` key (AI SDK, opencode), the `signature` field with marker id `'redacted_thinking'` (Pydantic AI), `text: None` plus signature (TensorZero), a non-standard block (LangChain), `redacted: true` with the data in the signature string (pi), a custom raw part (genai). Zed's Gemini and OpenAI paths skip it | Caller holds it. A neutral "signature string" type has to overload a field or add a flag. A raw block list holds it with no special case | Filled (AI SDK, Pydantic AI, LangChain, TensorZero, pi, genai, Zed) | H5, H28, H37, H45, H57, H60, H70 |
| Thinking bound to tools and system prompt | Anthropic (newer models) | A thinking block is tied to the request's `system`, `tools`, and earlier messages. `thinking.block_binding.prefix_mismatch_behavior: "drop_block"` lets the API drop a mismatched block | see OpenRouter evidence | rig stores a fingerprint of tools and system prompt in the turn's `Origin.context` and treats a turn from another context "as if from another model", unless the request sends `drop_block`. AI SDK exposes `blockBinding.prefixMismatchBehavior` as an Anthropic option. Pydantic AI retries once with `drop_block` (workspace report) | Whoever replays must know whether tools or system changed, or must send `drop_block`. A per-model fact (`binds_context`) decides it | Filled (rig, AI SDK) | H12, H53 |
| Thought signature on a function call | Gemini 3 | The first function call in a step must have its `thoughtSignature`. Later parallel calls in the same reply have none. A call with no signature can take the documented value `skip_thought_signature_validator` | see OpenRouter evidence | Per tool call in all projects: part metadata (AI SDK, opencode), `provider_details` on the call (Pydantic AI), a side map keyed by tool call id in `additional_kwargs` (LangChain), a signature-only thought block placed before the call (TensorZero, genai), a field on the call (pi, Zed, Goose metadata). Seven projects inject the skip value when the signature is missing (AI SDK, Pydantic AI, LangChain, genai, opencode, Goose, rig) | State is per tool call, not per message. A per-message block list holds it only if each block keeps its own signature or the list keeps adjacency. A standard chat-completions `tool_calls` entry holds an id, a type, and a function, so the signature needs an extension field | Filled (AI SDK, Pydantic AI, LangChain, TensorZero, genai, pi, opencode, Goose, Zed) | H14, H15, H30, H39, H46, H54, H57, H60, H66, H69, H70 |
| Thought signature on text and other parts | Gemini | A signature "can appear on ANY part type" and must go back "inside its original Part". Parts with signatures must not be merged or moved | see OpenRouter evidence | AI SDK moves a signature from an empty text part onto the previous part and replaces that part's metadata. LangChain replays function calls after all other parts, so order is lost. TensorZero errors when a signature-only block is last, or is followed by a thought or unknown block | Order inside one reply matters. A translator that splits a reply into `content` plus `tool_calls` loses the position unless the block list is kept | Filled (AI SDK, LangChain, TensorZero, pi, Pydantic AI) | H14, H30, H39, H46, H60 |
| Signature validity across Gemini API and Vertex | Google | Disputed between projects (see Conflicts) | see OpenRouter evidence | LangChain treats both as one native family. AI SDK looks up the other namespace as a fallback. Pydantic AI replaces the other transport's signature with the skip value | The tag may need the endpoint, not only the vendor name. Not settled | Filled (LangChain, AI SDK, Pydantic AI) | H15, H24, H30, H40 |
| Reasoning item with encrypted content | OpenAI Responses | A reasoning item has an `id`, a `summary`, and `encrypted_content` when asked for with `include: ["reasoning.encrypted_content"]`. With `store: false` only the encrypted content can be replayed | see OpenRouter evidence | AI SDK and opencode store `{itemId, reasoningEncryptedContent}` per part. Pydantic AI puts the encrypted content in `signature` and the item id in `id`. TensorZero keeps the encrypted content and drops the id. LangChain and rig keep the raw item. pi serializes the whole item into the signature string. One item becomes several parts in AI SDK and Pydantic AI, merged again by id | Caller holds the encrypted item when `store` is false. The same slot as an Anthropic signature can hold it, but it is an item between other items, not a field on text | Filled (AI SDK, Pydantic AI, LangChain, TensorZero, rig, pi, opencode, Goose) | H18, H19, H31, H38, H47, H54, H60, H66, H69 |
| Reasoning item paired with the item after it | OpenAI Responses | A stored message is "bound to the reasoning item that produced" it. A reasoning item without its following item, or a message without its reasoning item, is an error | see OpenRouter evidence | rig: `needs_next` drops a reasoning item when the item after it is not replayed. AI SDK sends message text by content instead of by reference when reasoning was pruned. pi skips errored turns because of "reasoning without following item" and drops item ids on a model change | Pruning or editing one block can invalidate its neighbor. Replay must treat the pair as a unit | Filled (rig, AI SDK, pi) | H20, H24, H54, H61, H62 |
| Server-stored conversation state | OpenAI Responses | `store` (default true), `previous_response_id`, `conversation`, and `item_reference` input items point at state kept by OpenAI | see OpenRouter evidence | AI SDK branches on `store`, `previousResponseId`, and `conversation` and sends `item_reference` when stored. Pydantic AI has `accepts_server_state`, judged by provider name and endpoint URL, apart from encrypted data. opencode and Goose default `store` to false | State is held by the vendor, keyed by ids. It does not survive a provider change or a different endpoint. A stateless caller sets `store: false` and replays items | Filled (AI SDK, Pydantic AI, LangChain, opencode, Goose) | H19, H20, H27, H31, H38, H66, H69 |
| Message item id and phase | OpenAI Responses | Output text has an item id (`msg_...`) and a `phase` (`commentary` or `final_answer`) | see OpenRouter evidence | AI SDK: `{itemId, phase}` on the text part. pi: a `TextSignatureV1` JSON string in `textSignature`. LangChain: `id` and `phase` on the block, and it refuses to send ids that start with `lc_`. Zed reads the phase from `reasoning_details` | Text parts also hold vendor state, not only reasoning. Editing the text of a stored item is ignored by the API (Pydantic AI comment) | Filled (AI SDK, pi, LangChain, Pydantic AI, Zed) | H18, H20, H26, H38, H59, H70 |
| Tool call id format | OpenAI Responses, Anthropic, others | Responses has a `call_id` and a separate item id. Anthropic ids match `[a-zA-Z0-9_-]`, 64 at most | see OpenRouter evidence | pi and rig keep the Responses pair as one string `call_id\|item_id` and rewrite ids when a call crosses to another model. rig cuts OpenAI ids to 40. opencode replaces illegal characters for Anthropic. AI SDK refuses to send function calls as item references | Ids must be rewritten on a switch, and the tool result must follow the rewrite. A gateway can do it if it sees both the call and the result | Filled (pi, rig, opencode, AI SDK) | H20, H54, H62, H66 |
| Server tool call and result | Anthropic | `server_tool_use` plus a `*_tool_result` block in the same assistant turn. Web search results hold `encrypted_content`, and citations hold an encrypted index | see OpenRouter evidence | AI SDK models each tool by name and rebuilds blocks field by field, and warns "is not supported" for any other tool. opencode keeps the raw result payload in metadata and errors on an unknown tool name. TensorZero and rig keep the raw block as an unknown or opaque item. Pydantic AI replays search and fetch results to any accepting client and code execution only to the same provider. pi's three block types leave them out | Raw blocks replay with no per-tool code. Typed neutral blocks need code per tool and per version. A use and its result replay only together (rig) | Filled (AI SDK, opencode, TensorZero, rig, Pydantic AI, LangChain, pi) | H8, H29, H37, H45, H50, H54, H59, H66 |
| Server tool call and result | Gemini, OpenAI Responses | Gemini: `toolCall` and `toolResponse` parts with an id and a tool type. OpenAI: hosted tool items (`web_search_call` and others) | see OpenRouter evidence | AI SDK: with `store` false, "Results for OpenAI tool ... are not sent to the API" except four tool kinds. Pydantic AI echoes Gemini server parts only with Google's own ids. LangChain passes 14 Responses item types through unchanged. opencode keeps hosted results across models of the same provider | Hosted tool data is provider-scoped, not model-scoped (opencode). With `store` false the caller must hold full items | Filled (AI SDK, Pydantic AI, LangChain, opencode) | H16, H20, H30, H38, H65 |
| Code execution container | Anthropic | A response returns `container: {id, expires_at}`. The next request passes the id | see OpenRouter evidence | AI SDK returns it in per-response metadata and ships a helper the application calls to copy the id into the next request's options. LangChain has `reuse_last_container`. Pydantic AI has `anthropic_container` | State is held by the vendor, per account. It is per response, not per block. Application or gateway code must read it | Filled (AI SDK, LangChain, Pydantic AI) | H10, H29, H33, H41 |
| Citations | Anthropic, OpenAI, Gemini | Each vendor returns its own citation shape on text | see OpenRouter evidence | AI SDK, Pydantic AI, and LangChain each have a neutral citation type and also keep the vendor's raw citations. Only the raw copy is sent back. AI SDK drops `source` parts on replay. LangChain Google: "Citations are only handled on output. Can't pass them back" | The neutral form is for reading. Replay needs the raw form. Web search citations only go where the search results go | Filled (AI SDK, Pydantic AI, LangChain) | H9, H29, H36, H37 |
| Vendor compaction block | Anthropic, OpenAI | Anthropic returns a `compaction` block with readable content. OpenAI returns a compaction item with `encrypted_content` | see OpenRouter evidence | Pydantic AI and opencode have a typed `CompactionPart` that names its provider. AI SDK folds Anthropic's into a text part with `type: 'compaction'` metadata and OpenAI's into a custom part. Zed tags it with provider id and format. opencode errors on replay to another provider | Must go back to the provider that made it. Projects error here where they silently drop reasoning | Filled (Pydantic AI, opencode, AI SDK, Zed) | H7, H20, H26, H64, H66, H70 |
| Reasoning effort and thinking budget | All | Anthropic: `thinking` type plus `effort`. OpenAI: `reasoning.effort`. Gemini: `thinkingLevel` or `thinkingBudget` | see OpenRouter evidence | Every project has one neutral setting. AI SDK and opencode: `none, minimal, low, medium, high, xhigh, max`. Pydantic AI: a bool or `minimal` to `max`. rig: `Off`, an effort, or a token budget. The translation is per model. A vendor-specific setting wins over the neutral one in AI SDK and Pydantic AI | A neutral level can be translated in the gateway with a per-model table. The caller does not need vendor values | Filled (AI SDK, Pydantic AI, rig, genai, pi, opencode) | H13, H33, H55, H58, H63, H67 |
| Thinking that cannot be turned off | Anthropic (some models), others | Some models reject a disabled thinking setting | see OpenRouter evidence | AI SDK maps `none` to `effort: 'low'` or to `between_tools` and adds a warning. Pydantic AI: `False` means "use the lowest effort it accepts". rig can answer `Unsupported` as an error or a warning | The neutral off switch becomes "least" on those models. The layer needs a per-model fact and a way to tell the caller | Filled (AI SDK, Pydantic AI, rig) | H13, H33, H55 |
| Cache breakpoints and retention | Anthropic, OpenAI, Gemini | Anthropic: `cache_control` on up to 4 blocks, 5 minute or 1 hour. OpenAI: cache key and retention options. Gemini: a cached content handle | see OpenRouter evidence | In the AI SDK caching is set through the vendor map: the caller puts `providerOptions.anthropic.cacheControl` on a part. Pydantic AI has a neutral `cache` setting with retention snapped to the nearest tier, plus a `CachePoint` marker. opencode has a neutral `CacheHint` per part and an `auto` policy. rig has `CacheRetention` of `None`, `Short`, `Long`. Zed has `cache: bool` per message | A neutral retention level and automatic placement work in four projects. Explicit placement stays vendor-shaped in one | Filled (AI SDK, Pydantic AI, opencode, rig, Zed, genai) | H11, H33, H55, H58, H67, H70 |
| Usage counts | All | Anthropic reports uncached input apart from cache read and write. OpenAI and Gemini report inclusive totals | see OpenRouter evidence | All ten have cache read and cache write fields. Reasoning tokens are a field in seven (AI SDK, Pydantic AI, LangChain, rig, genai, pi, opencode). TensorZero has them only in the raw usage entry, and Goose's and Zed's usage types stop at cache counts. AI SDK, opencode, and TensorZero keep the vendor's raw usage object beside the neutral one | The gateway can normalize. The raw object is needed for anything the neutral type leaves out | Filled (all ten) | H22, H34, H41, H49, H55, H58, H63, H67, H68, H71 |
| Finish reason | All | Each vendor has its own words. Anthropic's `pause_turn` means the caller must send another request | see OpenRouter evidence | Neutral enums have 4 to 7 values. AI SDK, opencode, pi, LangChain, and genai keep the raw value beside the neutral one. AI SDK maps `pause_turn` to `stop`. LangChain adds a separate `finish_state: "suspended"`. Pydantic AI has a separate response state with `suspended` | A normalized reason plus the raw string covers it. "Paused, send again" is a state, and two projects model it apart from the finish reason | Filled (AI SDK, Pydantic AI, LangChain, TensorZero, rig, genai, pi, opencode, Zed) | H23, H34, H41, H49, H55, H58, H63, H67, H71 |
| Reasoning from another vendor or model | All | Vendors reject each other's signatures and encrypted blobs | see OpenRouter evidence | Three answers. Drop it: pi, Goose, TensorZero, LangChain (Anthropic and Gemini targets), AI SDK (Anthropic and OpenAI targets). Turn it into text: rig, Pydantic AI (in a separate user turn for models that copy formatting). Send it as an unsigned thought: AI SDK and Pydantic AI toward Gemini. opencode keeps the text as a reasoning part and lets the target adapter decide | Needs a tag and a rule. Text conversion has a recorded failure (H72). The rule belongs where the target model is known | Filled (all ten) | H6, H15, H19, H28, H30, H37, H40, H44, H51, H61, H65, H69, H72 |
| The tag on vendor state | All | Not a vendor rule. Each project picks what it records | see OpenRouter evidence | Provider name only: AI SDK (the map key), LangChain (one string per message), TensorZero thoughts. Provider name plus endpoint: Pydantic AI. Provider plus model: opencode, Goose (model name), TensorZero unknown blocks. Wire plus provider plus model: pi, rig. Bare strings that each adapter reads as its own: genai signatures, Zed signatures | A provider-only tag replays one model's signature to another model of the same vendor. Projects that tag the model drop it instead | Filled (all ten) | H2, H6, H27, H35, H44, H50, H51, H56, H59, H61, H64, H65, H68, H70 |
| Vendor block with no neutral type | All | Vendors add block types over time | see OpenRouter evidence | Raw passthrough: TensorZero `unknown`, rig `Opaque`, LangChain `non_standard`, genai `Custom`, AI SDK `custom` with a `{provider}.{type}` kind. pi, Goose, and Zed have a closed block set | A raw block type lets new vendor blocks survive with no code change. A closed set drops them | Filled (TensorZero, rig, LangChain, genai, AI SDK, pi, Goose, Zed) | H4, H36, H43, H45, H50, H56, H59, H68, H70 |
| Vendor-only request options | Anthropic, Google, OpenAI | See the option lists | see OpenRouter evidence | AI SDK: 21 call options under `providerOptions.anthropic`, 16 under `google`, 31 under `openai`. Pydantic AI: 19 `anthropic_` settings, 9 `google_`, and more than 30 `openai_`. rig sends anything outside its nine options through `additional_params` | These predict where a caller is pushed toward vendor knowledge: server tools, containers, context management, safety settings, service tiers, stored conversations, fallbacks, citations, task budgets | Filled (AI SDK, Pydantic AI, rig, LangChain) | H12, H17, H21, H33, H41, H55 |
| History edited after the reply | Anthropic, OpenAI | A changed block invalidates its signature, and stored items ignore edited text | see OpenRouter evidence | rig fingerprints each block's neutral form and stops replaying the stored item once the block was edited. Pydantic AI tells application code to clear `id` when it edits text. The other projects replay whatever is stored | Whoever edits history must drop or flag the vendor state for the edited block and what follows | Filled (rig, Pydantic AI) | H26, H52 |
| Interrupted or failed reply | All | A reply cut mid-block has incomplete vendor state | see OpenRouter evidence | pi and rig skip errored turns on replay. opencode does not replay the last block's vendor state of a cut stream and drops reasoning that no output followed. Pydantic AI treats an empty signature as invalid and falls back to text | The stored block list must record how the reply ended, and replay must check it | Filled (pi, rig, opencode, Pydantic AI) | H28, H51, H61, H65 |
| `reasoning_details` as clients hold it | OpenRouter (client side only) | Not a vendor rule. Field names as the clients' codecs declare them: `type` of `reasoning.text`, `reasoning.summary`, `reasoning.encrypted`, plus `format`, `index`, `id`, `signature`, `data` | see OpenRouter evidence | Pydantic AI makes one thinking part per entry, tagged `openrouter`. TensorZero keeps `format` in `extra_data`. pi stores the merged array as a JSON string in the signature slot. rig keeps it verbatim. LangChain merges stream fragments by `(type, index)` and removes ids that start with `rs_` before sending it back | Clients treat it as one opaque array per message, tagged with the gateway's name and not the upstream vendor. Two clients had to repair it before replay | Filled (Pydantic AI, TensorZero, pi, rig, LangChain) | H32, H42, H48, H54, H60 |

### Comparison table: one row per project

| Project | Slot shape | Tag (how the layer knows who made the state) | Granularity | Switch rule (replay to another provider or model) | Vendor-specific options the application still has to know |
|---|---|---|---|---|---|
| Vercel AI SDK (spec v4) | One open map per part and per message, keyed by provider name: `providerMetadata` out, `providerOptions` in. Plus a `custom` part with a `{provider}.{type}` kind | The map key (provider name) only | Per part, per tool call, per message, per response | Per provider package. Anthropic: drop with warning `'unsupported reasoning metadata'`. OpenAI: drop with "Non-OpenAI reasoning parts are not supported". Google: send text as an unsigned thought, and inject `skip_thought_signature_validator` on Gemini 3 calls | All of `providerOptions.anthropic`, `.google`, `.openai` (H12, H17, H21): cache breakpoints, containers, context management, `store`, `previousResponseId`, safety settings, provider tool ids |
| Pydantic AI | Typed fields on every part: `id`, `provider_name`, `provider_details` (open map), and `signature` on thinking | `provider_name` per part, plus `provider_url` and `canonical_system` per response. The model name is stored and replay ignores it | Per part and per response | Encrypted data: accepted when the canonical system matches, else thinking becomes tagged text (in a user turn for models that copy formatting). Server ids: accepted only for the same endpoint | Settings with a vendor prefix (`anthropic_`, `google_`, `openai_`, `openrouter_`) for everything past thinking level, cache, citations, service tier, tool choice (H33) |
| LangChain | The vendor's own blocks as message content, with a derived neutral view. In the neutral view: `extras` per block and a `non_standard` wrapper | `response_metadata["model_provider"]`, one string per message. Replay ignores the model name | Per block, except Gemini function-call signatures, which sit in a per-message side map keyed by tool call id | Anthropic target: skip foreign reasoning silently. Gemini target: drop with a warning. OpenAI Responses target: no provider check on reasoning blocks | Constructor fields per chat model class: `thinking`, `betas`, `context_management`, `mcp_servers`, `use_previous_response_id`, `include`, `output_version` (H38, H41) |
| TensorZero | Two block types in the public API: `thought` (`text`, `signature`, `summary`, `provider_type`, `extra_data`) and `unknown` (raw `data`) | `provider_type` on a thought. `model_name` and `provider_name` on an unknown block | Per block. A Gemini per-part signature becomes a separate thought block next to the part | Filter before each request: drop a thought of another provider type, drop an unknown block of another model or provider | `extra_body` and `extra_headers` per provider, and raw `unknown` blocks the application may build itself |
| rig | The raw provider item (JSON) beside each neutral block (`native`), plus `Opaque` items | `Origin` per turn: wire, provider, model, response id, and a fingerprint of tools and system prompt | Per block, in provider order | Exact match of wire, provider, and model, else rebuild from neutral fields: reasoning becomes text, opaque items are dropped, call ids are rewritten | `additional_params` for anything outside nine neutral options, including hosted tools (H55) |
| genai | A bare `ThoughtSignature(String)` part, `thought_signatures` on a tool call, and a raw `Custom` part with the model identity | Signatures are bare strings. Custom parts hold the model identity | Per part, by position | Each adapter reads any signature string as its own. Anthropic drops thinking when text and signature counts differ | `extra_body`, `extra_headers`, and per-adapter behavior |
| pi | A signature string on each of three block types (`textSignature`, `thinkingSignature`, `thoughtSignature`), whose content depends on the wire | `api`, `provider`, `model` on the assistant message | Per block and per tool call | Same provider, wire, and model, else thinking is dropped, signatures are removed, and tool call ids are rewritten | Per-wire options and model compat flags. No server tool or opaque block type |
| opencode | Request layer: a provider-keyed map per part, plus neutral `encrypted` and `cache` fields. Stored layer: one `native` record per block | `Model.Ref` (provider id and model id) on the stored assistant message | Per block and per message | Two levels: model-scoped state needs the same model, hosted tool result data needs the same provider. Other reasoning keeps its text only. Compaction to another provider is an error | `providerOptions` per route, for example `store` and `include` on OpenAI Responses (H66) |
| Goose | Typed `Thinking {thinking, signature}` and `RedactedThinking {data}` blocks, and a free map on tool requests | `InferenceMetadata` on the message: provider, requested model, resolved model | Per block and per tool request | A thinking block is stale when the current model is neither the requested nor the resolved model. One retry without reasoning when OpenAI rejects encrypted content | Request params per provider (for example `store`, `reasoning_mode`) |
| Zed | `Thinking {text, signature}`, `RedactedThinking`, `thought_signature` on tool use, and one opaque `reasoning_details` JSON value per message | Signatures are bare strings. Compaction state holds a provider id and a format | Per block, per tool use, per message | In the model crates read: Gemini takes any signed thinking block, the OpenAI paths skip or textualize thinking. Thread-level filtering was not read | Per-provider settings in the provider crates (not read in full) |

## Evidence

Every item in this file has the basis label `Filled`, with the project named, except H73 (`Workspace`). None of it shows what OpenRouter does. Items H32, H42, H48, H54, and H60 describe client code written for OpenRouter: they show how a client holds OpenRouter's fields, and the lead agent may use them as the basis for an inference. Line numbers are at the pinned commits listed under Sources. All sources were read on 2026-11-22.

### Vercel AI SDK (`vercel/ai`, TypeScript)

**H1. The neutral type is an ordered part list per message, at language model specification v4.**

- The pinned tree has `packages/provider/src/language-model/v2`, `v3`, and `v4`. The three first-party providers import the `LanguageModelV4*` types. Package versions at the commit: `@ai-sdk/provider` 4.0.39, `ai` 7.0.179, `@ai-sdk/anthropic` 4.0.107, `@ai-sdk/google` 4.0.124, `@ai-sdk/openai` 4.0.122.
- `language-model-v4-prompt.ts:18-59`: "export type LanguageModelV4Prompt = Array<LanguageModelV4Message>;" with roles `'system'`, `'user'`, `'assistant'`, `'tool'`. The assistant content is one array of `LanguageModelV4TextPart | LanguageModelV4FilePart | LanguageModelV4CustomPart | LanguageModelV4ReasoningPart | LanguageModelV4ReasoningFilePart | LanguageModelV4ToolCallPart | LanguageModelV4ToolResultPart`.
- `language-model-v4-generate-result.ts:12-15`: "Ordered content that the model has generated." and `content: Array<LanguageModelV4Content>`.
- Source: H-s1. Basis: Filled (Vercel AI SDK).

**H2. The vendor-state slot is one open map keyed by provider name, on every message and every part.**

- `shared-v4-provider-options.ts:13-24`: "The outer record is keyed by the provider name, and the inner record is keyed by the provider-specific metadata key." and `export type SharedV4ProviderOptions = Record<string, JSONObject>;`. `shared-v4-provider-metadata.ts:24` has the same shape for output: `export type SharedV4ProviderMetadata = Record<string, JSONObject>;`.
- `language-model-v4-prompt.ts:52-58` puts `providerOptions?: SharedV4ProviderOptions` on every message, and lines 77, 96, 125, 145, 188, 223, 252, 282 put it on every part type. The comment on each: "Additional provider-specific options. They are passed through to the provider from the AI SDK and enable provider-specific functionality that can be fully encapsulated in the provider."
- The same map is used for two different jobs: replay state that came from the vendor (a signature), and settings the application picked (a cache breakpoint). The type's own example is a setting: `"anthropic": { "cacheControl": { "type": "ephemeral" } }`.
- Source: H-s1. Basis: Filled (Vercel AI SDK).

**H3. Replay is a mechanical copy: each output part's `providerMetadata` becomes the same part's `providerOptions`.**

- `packages/ai/src/generate-text/to-response-messages.ts:66-71`: `case 'reasoning': content.push({ type: 'reasoning', text: part.text, providerOptions: part.providerMetadata });`. The same copy is at lines 52-57 (text), 59-64 (custom), 89-101 (tool-call), 111-117 (tool-result).
- Lines 29-32: "Skip sources - they are response-only content that no provider expects back".
- Lines 42-49: "Skip empty text without provider metadata." An empty text part that has metadata is kept, which is what lets a signature on an empty Gemini text part survive (see H24).
- The core never reads inside the map. Only the provider package named by the key does.
- Source: H-s2. Basis: Filled (Vercel AI SDK).

**H4. A vendor block with no neutral part type goes in a `custom` part whose `kind` names the provider.**

- `language-model-v4-custom-content.ts:3-21`: "A provider-specific content block that does not map to another standardized content part type." and ``kind: `${string}.${string}` `` with the comment "The kind of custom content, in the format `{provider}.{provider-type}`."
- Kinds seen in the three providers: `anthropic.container_upload`, `anthropic.fallback`, `anthropic.message_start` (stream only), `openai.compaction`.
- Source: H-s1, H-s3, H-s5. Basis: Filled (Vercel AI SDK).

**H5. Anthropic thinking is stored as a reasoning part with `{ anthropic: { signature } }` or `{ anthropic: { redactedData } }`.**

- `packages/anthropic/src/anthropic-language-model.ts:1682-1705`: a `thinking` block becomes `{ type: 'reasoning', text: part.thinking, providerMetadata: { anthropic: { signature: part.signature } } }`, and a `redacted_thinking` block becomes `{ type: 'reasoning', text: '', providerMetadata: { anthropic: { redactedData: part.data } } }`.
- `anthropic-api.ts:1591-1598`: `anthropicReasoningMetadataSchema = z.object({ signature: z.string().optional(), redactedData: z.string().optional() })`.
- In a stream the signature arrives as its own delta: line 3190 `case 'signature_delta':` emits a reasoning delta whose `providerMetadata` is `{ anthropic: { signature: value.delta.signature } }`.
- Source: H-s3. Basis: Filled (Vercel AI SDK).

**H6. On replay to Anthropic, a reasoning part without Anthropic metadata is dropped with a warning. There is no model tag, so any Anthropic signature is sent to any Anthropic model.**

- `convert-to-anthropic-prompt.ts:959-1012`: the converter calls `parseProviderOptions({ provider: 'anthropic', providerOptions: part.providerOptions, schema: anthropicReasoningMetadataSchema })`. With a `signature` it pushes `{ type: 'thinking', thinking: part.text, signature }`. With `redactedData` it pushes `{ type: 'redacted_thinking', data }`. Otherwise it pushes the warning `'unsupported reasoning metadata'` and sends nothing for that part.
- The `sendReasoning` option (default true, `anthropic-language-model.ts:756`) turns all reasoning replay off, with the warning `'sending reasoning content is disabled for this model'`.
- The metadata holds the signature alone, so the converter treats a signature from any Claude model the same way.
- Source: H-s3. Basis: Filled (Vercel AI SDK). Confidence: high for the code path, and the "any Anthropic model" statement is an inference from the two-field schema.

**H7. Anthropic blocks that have no neutral part are folded into text or custom parts, with the block type in the metadata.**

- Compaction: `anthropic-language-model.ts:1714-1732` turns a `compaction` block into `{ type: 'text', text: part.content, providerMetadata: { anthropic: { type: 'compaction', signature } } }`. `convert-to-anthropic-prompt.ts:914-939` turns a text part whose metadata has `type === 'compaction'` back into `{ type: 'compaction', content: part.text, signature }` and adds the beta `compact-2026-09-04`.
- Server-side fallback: `anthropic-language-model.ts:2127-2140` emits `{ type: 'custom', kind: 'anthropic.fallback', providerMetadata: { anthropic: { type: 'fallback', from, to } } }`, and `convert-to-anthropic-prompt.ts:1015-1041` sends it back as a `fallback` block.
- Container upload: `{ type: 'custom', kind: 'anthropic.container_upload', providerMetadata: { anthropic: { fileId: part.file_id } } }` (lines 1706-1713).
- Source: H-s3. Basis: Filled (Vercel AI SDK).

**H8. Anthropic server tools are held as a tool-call part with `providerExecuted: true` plus a tool-result part in the same assistant message, and are rebuilt field by field on replay.**

- `anthropic-language-model.ts:1783-1862`: a `server_tool_use` block becomes `{ type: 'tool-call', toolCallId: part.id, toolName, input: JSON.stringify(part.input), providerExecuted: true }`. Lines 1936-1949: a `web_search_tool_result` becomes a tool-result whose `result` is a camelCase copy: `{ url, title, pageAge, encryptedContent: result.encrypted_content, type }`.
- `convert-to-anthropic-prompt.ts:1471-1523` rebuilds the block from that copy: `encrypted_content: result.encryptedContent`. Lines 1118-1159 rebuild `server_tool_use` for the tool names `code_execution`, `web_fetch`, `web_search`, `tool_search_tool_regex`, `tool_search_tool_bm25`, and `advisor`. Any other provider-executed tool gets the warning "provider executed tool call for tool ${part.toolName} is not supported" and is not sent.
- MCP calls keep their extra fields in the slot: `providerMetadata: { anthropic: { type: 'mcp-tool-use', serverName: part.server_name } }` (lines 1864-1880).
- The design consequence: every new server tool needs new code in both directions. The raw block is not stored.
- Source: H-s3. Basis: Filled (Vercel AI SDK).

**H9. Citations have a neutral form and a raw vendor copy, and the raw copy is what goes back.**

- `language-model-v4-text.ts:15-18`: `citations?: Array<LanguageModelV4Citation>` on output text, "References supporting this text, separate from retrieved sources." `language-model-v4-citation.ts:7-19` has `source`, `startIndex`, `endIndex`, `citedText`.
- `anthropic-language-model.ts:1640-1657`: the text part gets both the mapped `citations` and `providerMetadata: { anthropic: { citations } }` with the raw array. `convert-to-anthropic-prompt.ts:950-952` replays `citations: textMetadata.citations` from the metadata.
- The prompt-side text part (`language-model-v4-prompt.ts:64-78`) has `text` and `providerOptions` only, so the neutral citation list does not make the round trip. Separate `source` parts are skipped on replay (H3).
- Source: H-s1, H-s3. Basis: Filled (Vercel AI SDK).

**H10. Message-level Anthropic state (container, context management) is returned in metadata, and application code reads it to send it back.**

- `anthropic-language-model.ts:2162-2228` returns `providerMetadata.anthropic` with `usage`, `stopSequence`, `serviceTier`, `stopDetails`, `iterations`, `container: { expiresAt, id, skills }`, and `contextManagement`.
- `forward-anthropic-container-id-from-last-step.ts:4-36`: "Sets the Anthropic container ID in the provider options based on any previous step's provider metadata." It reads `steps[i].providerMetadata?.anthropic?.container?.id` and returns `{ providerOptions: { anthropic: { container: { id: containerId } } } }`. "You can use this function in `prepareStep` to forward the container ID between steps."
- So the per-part slot is copied without being read, and the per-response slot is read by a helper the application has to call.
- Source: H-s3. Basis: Filled (Vercel AI SDK).

**H11. Cache breakpoints are a vendor option placed in the same slot, counted and capped by the provider.**

- `get-cache-control.ts:7-8`: "Anthropic allows a maximum of 4 cache breakpoints per request" and `const MAX_CACHE_BREAKPOINTS = 4;`. Lines 21-25 read `cacheControl` or `cache_control` from the configured namespace and then from `anthropic`.
- Lines 56-63: on a thinking block the result is the warning "cache_control cannot be set on ${context.type}. It will be ignored." Lines 68-74: past four, "Maximum 4 cache breakpoints exceeded (found N). This breakpoint will be ignored."
- The neutral call options (H13) stop at reasoning, sampling, tools, and response format, so caching is always set through this vendor map.
- Source: H-s3. Basis: Filled (Vercel AI SDK).

**H12. The Anthropic options that have no neutral equivalent.**

- `anthropic-language-model-options.ts:143-519` defines `providerOptions.anthropic` for a call: `thread`, `sendReasoning`, `structuredOutputMode`, `includePings`, `thinking` (types `adaptive`, `enabled`, `disabled`, `between_tools`, with `display` and `blockBinding.prefixMismatchBehavior`), `disableParallelToolUse`, `cacheControl`, `metadata`, `mcpServers`, `container`, `toolStreaming`, `effort`, `taskBudget`, `speed`, `serviceTier`, `inferenceGeo`, `fallbacks`, `anthropicBeta`, `safeguards`, `compaction`, `contextManagement` (edits `clear_tool_uses_20250919`, `clear_thinking_20251015`, `compact_20260112`).
- Per file part (lines 16-52): `containerUpload`, `citations.enabled`, `title`, `context`. Per system message (lines 61-137): `clearAt`, `effort`, `toolChanges`.
- Source: H-s3. Basis: Filled (Vercel AI SDK).

**H13. The neutral controls: one reasoning enum, sampling parameters, a tool-choice enum, and a response format.**

- `language-model-v4-call-options.ts:119-131`: "Reasoning effort level for the model. ... Defaults to 'provider-default'." with values `'provider-default' | 'none' | 'minimal' | 'low' | 'medium' | 'high' | 'xhigh' | 'max'`. The same file has `maxOutputTokens`, `temperature`, `stopSequences`, `topP`, `topK`, `presencePenalty`, `frequencyPenalty`, `responseFormat`, `seed`, `tools`, `toolChoice`.
- `language-model-v4-tool-choice.ts:1-5`: `{ type: 'auto' } | { type: 'none' } | { type: 'required' } | { type: 'tool'; toolName: string }`.
- `language-model-v4-provider-tool.ts:8-28`: a vendor-run tool is `{ type: 'provider', id: '<provider-id>.<unique-tool-name>', name, args }`. The application names the vendor in the tool id.
- Translation of the reasoning enum is per model. `anthropic-language-model.ts:3757-3836`: on a model that cannot turn thinking off, `'none'` becomes `{ effort: 'low' }` with the warning "reasoning 'none' is not supported by ${modelId}; it always uses adaptive thinking. Using effort 'low' to minimize thinking instead." On adaptive models an effort level becomes `{ thinking: { type: 'adaptive', display: 'summarized' }, effort }`. On older models it becomes `{ thinking: { type: 'enabled', budgetTokens } }`. A vendor option wins: the mapping runs only when `anthropicOptions?.effort == null` (line 826).
- Google maps the same enum to `thinkingLevel` or `thinkingBudget` (`google-language-model.ts:1645-1721`), and OpenAI resolves it with `resolveOpenAIReasoningEffort` against the model's supported efforts (`openai-responses-language-model.ts:349-354`).
- Source: H-s1, H-s3, H-s4. Basis: Filled (Vercel AI SDK).

**H14. Gemini thought signatures are stored per part, on any part type that had one.**

- `packages/google/src/google-language-model.ts:597-616`: a text or thought part becomes `{ type: part.thought === true ? 'reasoning' : 'text', text, providerMetadata: thoughtSignatureMetadata }` where the metadata is `{ thoughtSignature: part.thoughtSignature }`. Lines 634-645: a function call becomes a tool-call with `providerMetadata: part.thoughtSignature ? wrapProviderMetadata({ thoughtSignature: part.thoughtSignature }) : undefined`. Lines 646-658 do the same for inline data (a `file` or `reasoning-file` part).
- Lines 542-545: `wrapProviderMetadata` writes the same payload under every name in `providerOptionsNames`, so a Vertex response is stored under both `googleVertex` and `vertex`.
- Lines 605-609: a signature on an empty text part is moved onto the previous content part: `lastContent.providerMetadata = thoughtSignatureMetadata;`. That assignment replaces whatever metadata the previous part had.
- Source: H-s4. Basis: Filled (Vercel AI SDK).

**H15. On replay to Gemini, a signature goes back on the part it came from, a foreign reasoning part goes back as an unsigned thought, and a Gemini 3 function call with no signature gets the documented sentinel.**

- `convert-to-google-messages.ts:461-485`: for a reasoning part the converter returns `{ text: part.text, thought: true, thoughtSignature }`. `thoughtSignature` is undefined when the part has no Google metadata, so reasoning text from another vendor is sent to Gemini as a thought with no signature. This is the opposite of the Anthropic and OpenAI providers, which drop it (H6, H19).
- Lines 30-37: "Sentinel value Google documents for replaying functionCall parts whose original thoughtSignature is not available to the client." `SKIP_THOUGHT_SIGNATURE_VALIDATOR = 'skip_thought_signature_validator'`.
- Lines 593-606: "Gemini 3 returns a single signature for a parallel function-call response on the first standard function call. Subsequent standard function calls in the same model response legitimately have no signature." The sentinel is injected only for a Gemini 3 model and only when no earlier call in the same reply was signed.
- Lines 795-810, the warning: "Replayed N `functionCall` part(s) for a Gemini 3 model without a `thoughtSignature` ... Injected the documented `skip_thought_signature_validator` sentinel to keep the request from failing with HTTP 400. The likely cause is application code that drops `providerOptions.google.thoughtSignature` when persisting or serializing assistant tool-call messages."
- Lines 331-348: metadata is looked up under the provider's own names first, then across namespaces: "Cross-namespace fallback (gateway interop): Vertex providers may receive metadata under `google`, and the Google provider may receive metadata under `googleVertex`/`vertex`."
- Source: H-s4. Basis: Filled (Vercel AI SDK).

**H16. Gemini server tools are a tool-call and tool-result pair with the vendor's call id and tool type in the slot.**

- `google-language-model.ts:659-699`: a `toolCall` part becomes ``{ type: 'tool-call', toolCallId, toolName: `server:${part.toolCall.toolType}`, providerExecuted: true, dynamic: true, providerMetadata: { thoughtSignature, serverToolCallId, serverToolType } }``, and a `toolResponse` part becomes a tool-result with the same three metadata keys.
- `convert-to-google-messages.ts:612-624` and `:659-669` rebuild `{ toolCall: { toolType, args, id }, thoughtSignature }` and `{ toolResponse: { toolType, response, id }, thoughtSignature }` from them.
- Code execution uses `executableCode` and `codeExecutionResult` parts and a generated id (lines 571-596).
- Source: H-s4. Basis: Filled (Vercel AI SDK).

**H17. The Google options that have no neutral equivalent.**

- `google-language-model-options.ts:14-245`: `responseModalities`, `thinkingConfig` (`thinkingBudget`, `includeThoughts`, `thinkingLevel`), `cachedContent`, `structuredOutputs`, `safetySettings`, `threshold`, `audioTimestamp`, `labels`, `mediaResolution`, `imageConfig`, `retrievalConfig`, `streamFunctionCallArguments`, `serviceTier`, `continuationToken`, `sharedRequestType`, `requestType`.
- Source: H-s4. Basis: Filled (Vercel AI SDK).

**H18. An OpenAI Responses reasoning item is split into one reasoning part per summary, each holding `itemId` and `reasoningEncryptedContent`.**

- `packages/openai/src/responses/openai-responses-language-model.ts:992-1011`: "when there are no summary parts, we need to add an empty reasoning part", then for each summary `{ type: 'reasoning', text: summary.text, providerMetadata: { [providerOptionsName]: { itemId: part.id, reasoningEncryptedContent: part.encrypted_content ?? null } } }`.
- `providerOptionsName` is `'azure'` when the provider name includes `azure`, otherwise `'openai'` (lines 337-339).
- `openai-responses-provider-metadata.ts:25-72` lists the per-part shapes: reasoning `{ itemId, reasoningEncryptedContent }`, tool call `{ itemId, async, namespace, caller }`, compaction `{ type: 'compaction', itemId, encryptedContent }`, text `{ itemId, phase, annotations }`. The per-response shape (lines 16-23) has `responseId`, `logprobs`, `serviceTier`, `reasoningContext`, `promptCacheDiagnostics`, `inputTransformations`.
- Source: H-s5. Basis: Filled (Vercel AI SDK).

**H19. Replay to the Responses API has four branches, chosen by `store`, `previousResponseId`, and what the slot holds.**

- `convert-to-openai-responses-input.ts:1350-1454`, in order:
  - With a conversation or a previous response id, and an item id: the part is skipped (lines 1361-1366).
  - `store` true and an item id: `input.push({ type: 'item_reference', id: reasoningId })`, once per id (lines 1371-1383).
  - `store` false and an item id: a `{ type: 'reasoning', id, encrypted_content, summary }` item, with later parts of the same id merged into its summary (lines 1384-1420).
  - No item id: the converter falls back to the encrypted content. The comment: "The OpenAI Responses API accepts reasoning items without an id when encrypted_content is provided". If the part has neither, the warning is "Non-OpenAI reasoning parts are not supported. Skipping reasoning part: ..." (lines 1421-1452).
- Lines 1942-1963: "when store is false, remove reasoning parts without encrypted content", with the warning "Reasoning parts without encrypted content are not supported when store is false. Skipping reasoning parts."
- `openai-responses-language-model.ts:470`: `store: openaiOptions?.store ?? true`. Lines 634-639: "store defaults to true in the OpenAI responses API, so check for false exactly" and then `addInclude('reasoning.encrypted_content')` for a reasoning model.
- Source: H-s5. Basis: Filled (Vercel AI SDK).

**H20. Text, function calls, and server tool items also hold a server item id, and with `store` false most server tool results cannot be sent back.**

- Text: `convert-to-openai-responses-input.ts:817-852` reads `itemId` and `phase` (`'commentary' | 'final_answer'`). With `store` true it sends an `item_reference` only when the text follows reasoning in the same message. The comment at lines 808-812: "Stored messages of reasoning models are bound to the reasoning item that produced them. When the reasoning has been removed (e.g. with `pruneMessages`), referencing the message fails with "Item 'msg_...' of type 'message' was provided without its required 'reasoning' item", so it is sent with its content instead."
- Function calls: lines 1031-1044: "Plain client-executed function calls must NOT be [sent as an item reference]: the matching `function_call_output` can only reference the call by `call_id` (`call_...`), which the API cannot reconcile with an item id (`fc_...`) or an `item_reference`."
- Server tool results: with `store` true every result is an `item_reference` (lines 1332-1339). With `store` false, web search, tool search, programmatic tool calling, and shell are rebuilt from the stored result (lines 1206-1330), and every other tool gets the warning "Results for OpenAI tool ${part.toolName} are not sent to the API when store is false" (lines 1340-1345).
- Compaction: a `custom` part of kind `openai.compaction` is replayed as an `item_reference` or as `{ type: 'compaction', id, encrypted_content }` (lines 1456-1484).
- Source: H-s5. Basis: Filled (Vercel AI SDK).

**H21. The OpenAI Responses options that have no neutral equivalent.**

- `openai-responses-language-model-options.ts:34-346`: `conversation`, `include`, `includeWebSearchSources`, `instructions`, `logprobs`, `maxToolCalls`, `metadata`, `parallelToolCalls`, `previousResponseId`, `promptCacheKey`, `promptCacheOptions`, `promptCacheRetention`, `reasoningEffort`, `reasoningEffortUpdate`, `reasoningMode`, `reasoningContext`, `reasoningSummary`, `safetyIdentifier`, `serviceTier`, `store`, `passThroughUnsupportedFiles`, `strictJsonSchema`, `textVerbosity`, `truncation`, `user`, `systemMessageMode`, `forceReasoning`, `contextManagement`, `compactionTrigger`, `allowedTools`, `additionalTools`.
- `openai-responses-language-model.ts:367-372`: `conversation` and `previousResponseId` together give the warning detail "conversation and previousResponseId cannot be used together".
- Source: H-s5. Basis: Filled (Vercel AI SDK).

**H22. Neutral usage has cache read, cache write, and reasoning tokens, and keeps the vendor's raw usage object beside them.**

- `language-model-v4-usage.ts:6-59`: `inputTokens: { total, noCache, cacheRead, cacheWrite }`, `outputTokens: { total, text, reasoning }`, and `raw?: JSONObject` with "This is the usage information in the shape that the provider returns."
- The three vendors count differently and the converters fix it. Anthropic (`convert-anthropic-usage.ts:100-113`): `total: inputTokens + cacheCreationTokens + cacheReadTokens`. OpenAI Responses (`convert-openai-responses-usage.ts:44-56`): `noCache: inputTokens - cachedTokens - (cacheWriteTokens ?? 0)`. Google (`convert-google-usage.ts:38-51`): `cacheWrite: undefined` and `total: candidatesTokens + thoughtsTokens` for output.
- Source: H-s1, H-s3, H-s4, H-s5. Basis: Filled (Vercel AI SDK).

**H23. The neutral finish reason is a seven-value enum plus the raw vendor string.**

- `language-model-v4-finish-reason.ts:8-35`: `unified: 'stop' | 'length' | 'content-filter' | 'tool-calls' | 'error' | 'interrupted' | 'other'` and `raw: string | undefined`, "This is the original finish reason from the provider."
- `map-anthropic-stop-reason.ts:13-29`: `pause_turn`, `end_turn`, and `stop_sequence` map to `'stop'`. `refusal` maps to `'content-filter'`. `max_tokens` and `model_context_window_exceeded` map to `'length'`. `compaction` maps to `'other'`. A caller that reads only the unified value cannot tell `pause_turn` from `end_turn`.
- `map-google-finish-reason.ts:10-31`: `STOP` is `'tool-calls'` when the reply had tool calls, `MALFORMED_FUNCTION_CALL` is `'error'`, and six safety values are `'content-filter'`.
- `map-openai-responses-finish-reason.ts:14-27`: the Responses API has no finish reason on a normal reply, so a null value becomes `'tool-calls'`, `'content-filter'` (when the model refused), or `'stop'`.
- Source: H-s1, H-s3, H-s4, H-s5. Basis: Filled (Vercel AI SDK).

**H24. Known failures recorded in the SDK's own changelogs.**

- `packages/google/CHANGELOG.md:2265-2267` (entry 218bba1): "When using @ai-sdk/google-vertex provider with Gemini thinking models, multi-step tool calls would fail with "function call is missing a thought_signature" error. This was because thoughtSignature was stored under providerOptions.vertex but retrieved using hardcoded providerOptions.google."
- Line 1940 (7ba09a4): "Fix gateway failover losing thoughtSignature when failing over from Vertex to Google AI Studio."
- Lines 1168-1172 (cfca634): a no-argument function call "was dropped along with any `thoughtSignature` it carried. For Gemini 3 thinking models this caused the next multi-turn step to 400 with `missing thought_signature`."
- Lines 130-134, version 4.0.112 (fd7e8a7): "Preserve the order of text and complete function calls in streamed Google responses, including standalone thought signatures on empty text or reasoning parts." and "Omit foreign-provider empty signature text when replaying history through Anthropic or Bedrock".
- `packages/anthropic/CHANGELOG.md:991` (19c5ee2): "fix(anthropic): reorder assistant content b/w client and provider tool use".
- `packages/openai/CHANGELOG.md:349`: "With reasoning models and `store: true`, pruning reasoning (e.g. `pruneMessages({ reasoning: 'all' })`) caused follow-up requests to fail with "Item 'msg\_...' of type 'message' was provided without its required 'reasoning' item"." Line 1252 (a71d345): "drop reasoning parts without encrypted content when store: false". Line 1316 (17b5597): "skip passing reasoning items when using previous response id".
- These are the SDK's own release notes, so they are a primary source for what was fixed. They do not say how long each failure was live.
- Source: H-s6. Basis: Filled (Vercel AI SDK).

### Pydantic AI (`pydantic/pydantic-ai`, Python)

**H25. The neutral reply is a `ModelResponse` with an ordered `parts` list, tagged with provider name, provider URL, model name, and a "canonical system".**

- `pydantic_ai_slim/pydantic_ai/messages.py:5720-5809`: `parts: Sequence[ModelResponsePart]`, `model_name` ("The name of the model that generated the response."), `provider_name` ("The name of the LLM provider that generated the response."), `provider_url` ("The base URL of the LLM provider that generated the response."), `provider_details`, `provider_response_id`, `finish_reason`.
- Lines 5779-5792, `canonical_system`: "The system whose keys verify the stateless signed or encrypted data in this response, when it isn't evident from `provider_name`." and "Models read it to decide whether they can send that data, such as encrypted reasoning and thinking signatures, back to their own provider".
- Part kinds in the file: `TextPart`, `ThinkingPart`, `CompactionPart`, `FilePart`, `SpeechPart`, `ToolCallPart`, `NativeToolCallPart` (`part_kind` `'builtin-tool-call'`), and `NativeToolReturnPart` (`'builtin-tool-return'`).
- Source: H-s7. Basis: Filled (Pydantic AI).

**H26. The slot is three typed fields on every part: an id, a provider name, and an open details map. Thinking adds a `signature` field.**

- `messages.py:4015-4056`, `ThinkingPart`: `content: str`, `id: str | None`, `signature: str | None`, `provider_name: str | None`, `provider_details: dict[str, Any] | None`.
- The `signature` docstring (lines 4030-4040): "Supported by: Anthropic (corresponds to the `signature` field), Bedrock (corresponds to the `signature` field), Google (corresponds to the `thought_signature` field), OpenAI (corresponds to the `encrypted_content` field). When this field is set, `provider_name` is required to identify the provider that generated this data."
- The `provider_name` docstring (lines 4042-4047): "Signatures are only sent back to the same provider. Required to be set when `provider_details`, `id` or `signature` is set."
- `provider_details` (lines 4049-4054): "This is used for data that is required to be sent back to APIs, as well as data users may want to access programmatically."
- `TextPart` (lines 3966-4002) and `BaseToolCallPart` (lines 4317-4363) have the same `id`, `provider_name`, and `provider_details` trio. On a tool call: "An optional identifier of the tool call part, separate from the tool call ID. This is used by some APIs like OpenAI Responses." and "Native tool calls are only sent back to the same provider."
- A warning to application code that edits history, on `TextPart.id` (lines 3979-3981): "If you edit `content` ... set this to `None` too: the OpenAI Responses API uses its stored copy of an item sent with its ID and ignores the text sent alongside."
- Source: H-s7. Basis: Filled (Pydantic AI).

**H27. The switch rule separates two kinds of vendor state: data the API checks with its own key, and ids the API looks up on a server.**

- `pydantic_ai/_canonical_system.py:1-14`: "Stateless round-trip data is data the API verifies on replay by its own key and format, not by looking up stored state: OpenAI encrypted reasoning and compaction items, Anthropic thinking signatures and redacted thinking, Gemini thought signatures, xAI encrypted reasoning and tool results, and OpenRouter or Snowflake reasoning details." and "Server-side state (response, item, container and conversation ids) is judged by endpoint instead".
- `models/__init__.py:1431-1460`, `accepts_stateless_artifacts`: the data "is accepted when that is this model's `canonical_system` or one of its `compatible_systems`" and "Data that isn't accepted gets the model's fallback for foreign data, such as thinking rendered as text." Also: "A part without a `provider_name` is never accepted."
- `models/__init__.py:1462-1482`, `accepts_server_state`: "That's state the API stores and looks up by ID, rather than verifying by its own key: stored responses (`previous_response_id`), output item IDs, conversations and code execution containers. It's accepted when the response has this model's `system` as its `provider_name` and was created at an endpoint this model's provider `shares_server_state_with`." and "a response from another endpoint of the same API still has its encrypted reasoning sent back, without the IDs."
- The rule compares the provider system and the endpoint. It does not compare the model name, so a signature from one Claude model is sent to another Claude model.
- Source: H-s7. Basis: Filled (Pydantic AI).

**H28. Anthropic: a thinking block is a `ThinkingPart` with the signature, redacted thinking reuses the `signature` field with a marker id, and foreign thinking becomes tagged text.**

- `models/anthropic/__init__.py:2519-2523`: `ThinkingPart(id='redacted_thinking', content='', signature=item.data, provider_name=self.system)` and `ThinkingPart(content=item.thinking, signature=item.signature, provider_name=self.system)`.
- Lines 3471-3496, `_map_thinking_part`: "An empty signature (e.g. from an interrupted stream) is never valid, so fall back to tagged text rather than triggering a 400 from the API." With a signature that this model accepts, the part becomes a `thinking` or `redacted_thinking` block. Otherwise `render_replayed_thinking` turns the content into a text block.
- `_thinking_part.py:41-76`, `render_replayed_thinking`: "a `ThinkingPart` that arrived unsigned or from another provider has no native block to ride, so the only way to keep it in the history at all is to write it out as text." The text is wrapped in the model's thinking tags, or moved to a user message in an `assistant_thinking` wrapper for models that copy assistant formatting.
- `models/anthropic/__init__.py:3411-3416` gives the reason for the user-message placement: "Reasoning replayed in an assistant turn teaches a model that imitates assistant formatting to write `<thinking>` tags into its visible answers (https://github.com/pydantic/pydantic-ai/issues/5869)".
- Source: H-s7. Basis: Filled (Pydantic AI).

**H29. Anthropic server tool blocks are sent back by two different rules, depending on whether the result holds its state inline.**

- `models/anthropic/__init__.py:3498-3512`, `_replays_native_tool_part`: "Search, fetch and advisor results carry their state inline, encrypted where it's sensitive, so they're sent to any client that accepts the response's stateless data. Code execution and MCP results refer to state kept by the account that ran them, such as a container, so they only go back to the same provider."
- Lines 3434-3452: web search citations are sent back only when the model accepts the response's stateless data: "Web search citations point into the search results by an encrypted index, so they go wherever those results do". The raw citations sit in `provider_details['citations']`, and the neutral list sits in `TextPart.citations` (`messages.py:3997-4002`).
- A compaction block is a `CompactionPart` with `encrypted_content` and `signature` in `provider_details` (lines 3455-3469).
- Source: H-s7. Basis: Filled (Pydantic AI).

**H30. Gemini: the thought signature sits in `provider_details['thought_signature']` on whichever part it arrived on, and goes back on the same part.**

- `models/google.py:2297-2307`, quoting Google's rules in the docstring: "Always send the thought_signature back to the model inside its original Part. Don't merge a Part containing a signature with one that does not. This breaks the positional context of the thought. Don't combine two Parts that both contain signatures, as the signature strings cannot be merged." The value stored is `{'thought_signature': base64.b64encode(part.thought_signature).decode('utf-8')}`.
- Lines 2310-2334: the details are merged into the part and `item.provider_name = provider_name` is set, for text, thinking, function call, native tool, and file parts alike.
- Lines 2113-2172, `_content_model_response`: "thought signatures, and the native tool parts that carry them, only go back to the transport that issued them, as the Gemini API and Google Cloud reject each other's even though both providers can be named `google`."
- Lines 2206-2218: "the documented dummy `skip_thought_signature_validator` works for both Gemini API and Google Cloud. It's also what a signature from the other transport is replaced with: both reject each other's, and both reject a first function call without one (verified live, 2026-10-27)."
- Lines 2200-2203: a foreign or unsigned `ThinkingPart` with content is sent as `{'text': item.content, 'thought': True}`.
- Lines 2279-2294: a server tool part is echoed only on models that support tool combination and only when the id is Google's own: "Gemini rejects unknown ids".
- Source: H-s7. Basis: Filled (Pydantic AI). The "verified live" statement is the project's own note, not something this research ran.

**H31. OpenAI Responses: the encrypted content goes in `signature`, the reasoning item id in `id`, and one item becomes several parts that are merged again on replay.**

- `models/openai/_reasoning.py:154-189`, `map_reasoning_item`: `signature = item.encrypted_content`, then one `ThinkingPart(content=summary.text, id=item.id, signature=signature, provider_name=provider_name, ...)` per summary, with the comments "We use the same id for all summaries so that we can merge them on the round trip." and "We only need to store the signature and raw_content once."
- `models/openai/_responses_model.py:3108-3148`, `_map_response_thinking_part`: "Encrypted reasoning carries its state inline, so it's also sent back to a provider that accepts this one's encrypted data, with the item ID it was issued under." The signature is sent only when the profile has `openai_supports_encrypted_reasoning_content` and `accepts_stateless_artifacts` is true. Otherwise the thinking is rendered as text.
- Lines 2052-2056: `include.append('reasoning.encrypted_content')`, with a note that begins "OpenAI rejects `reasoning.encrypted_content` on any retrieve of a background".
- `models/openai/_settings.py:324` and `:367`: `openai_previous_response_id: Literal['auto'] | str` and `openai_conversation_id: Literal['auto'] | str`. `_responses_model.py:2429-2459` walks the history backwards to the newest response that passes `accepts_server_state` and sends only the messages after it.
- Source: H-s7. Basis: Filled (Pydantic AI).

**H32. Through OpenRouter, each `reasoning_details` entry becomes one `ThinkingPart` tagged `openrouter`, and the entry's `format`, `index`, and `type` ride in `provider_details`.**

- `models/_reasoning_details.py:1-8`: "The format originated with OpenRouter and has been adopted by other providers (e.g. Snowflake Cortex). Each detail carries a `type` (`reasoning.text`, `reasoning.summary`, or `reasoning.encrypted`) and a `format` naming the upstream provider's reasoning representation (`anthropic-claude-v1`, `openai-responses-v1`, ...)".
- Lines 19-29: the `format` literal is `'unknown', 'openai-responses-v1', 'anthropic-claude-v1', 'xai-responses-v1', 'google-gemini-v1'` or any string.
- Lines 57-78: `provider_details = reasoning.model_dump(include={'format', 'index', 'type'})`. A `reasoning.text` entry keeps `text` and `signature`. A `reasoning.encrypted` entry becomes `ThinkingPart(content='', signature=reasoning.data, ...)`.
- `models/openrouter.py:1490-1496` builds the parts with `self.system` as the provider name. Lines 1520-1539 send them back as `message_param['reasoning_details']` only when `accepts_stateless_artifacts(item)` is true.
- What the tag records: the provider name `openrouter`, plus OpenRouter's `format` string. The upstream vendor OpenRouter chose is stored at response level as `provider_details['downstream_provider']` (line 696), and the replay rule does not read it.
- This is what Pydantic AI's client does when it talks to OpenRouter. It shows how a client holds the field, and it does not show what OpenRouter's translator does with it.
- Source: H-s7. Basis: Filled (Pydantic AI).

**H33. Neutral controls: a thinking level, a cache setting, a citations switch, a service tier, and a tool choice. Vendor settings use a prefix per vendor and win over the neutral field.**

- `settings.py:17-20`: `ThinkingEffort = Literal['minimal', 'low', 'medium', 'high', 'xhigh', 'max']` and `ThinkingLevel = bool | ThinkingEffort`.
- Lines 614-628: "`False`: Disable thinking. On a model that can't turn thinking off, use the lowest effort it accepts" and "Provider-specific thinking settings (e.g., `anthropic_thinking`, `openai_reasoning_effort`) take precedence over this unified field."
- Lines 675-702, `cache: CacheSetting`: "`'5m'`/`'30m'`/`'1h'`/`'12h'`: Cache with a specific retention, snapped to the nearest tier the provider supports" and "Provider-specific cache settings ... take precedence: if any is set, including to `False`, this unified field is ignored entirely for that request." Supported by Anthropic, Bedrock, OpenRouter ("Anthropic and Gemini models only"), and OpenAI ("GPT-5.6 and later").
- `messages.py:1018`: a `CachePoint` marker the application can place in user content.
- Every vendor settings class says the same thing, for example `models/anthropic/__init__.py:578`: "ALL FIELDS MUST BE `anthropic_` PREFIXED SO YOU CAN MERGE THEM WITH OTHER MODELS."
- Vendor-only settings, by prefix. Anthropic (`models/anthropic/__init__.py:580-830`): `anthropic_metadata`, `anthropic_thinking`, `anthropic_cache_tool_definitions`, `anthropic_service_tier`, `anthropic_cache_instructions`, `anthropic_cache_messages`, `anthropic_cache`, `anthropic_cache_diagnostics`, `anthropic_previous_message_id`, `anthropic_effort`, `anthropic_task_budget`, `anthropic_container`, `anthropic_code_execution_tool_version`, `anthropic_web_tools_allowed_callers`, `anthropic_eager_input_streaming`, `anthropic_betas`, `anthropic_speed`, `anthropic_inference_geo`, `anthropic_context_management`. Google (`models/google.py:274-346`): `google_safety_settings`, `google_thinking_config`, `google_labels`, `google_video_resolution`, `google_cached_content`, `google_logprobs`, `google_top_logprobs`, `google_cloud_service_tier`, `google_model_armor_config`. OpenAI Responses (`models/openai/_settings.py:65-451`): `openai_reasoning_effort`, `openai_store`, `openai_text_verbosity`, `openai_service_tier`, `openai_prompt_cache_key`, `openai_prompt_cache_retention`, `openai_prompt_cache_options`, `openai_reasoning_mode`, `openai_reasoning_context`, `openai_reasoning_summary`, `openai_send_reasoning_ids`, `openai_truncation`, `openai_previous_response_id`, `openai_conversation_id`, `openai_context_management`, `openai_background`, `openai_transport`, and others.
- Source: H-s7. Basis: Filled (Pydantic AI).

**H34. Neutral usage and finish reason. The raw vendor finish reason goes in `provider_details`.**

- `usage.py:122-194`, `UsageBase`: `input_tokens`, `cache_write_tokens`, `cache_write_1h_tokens`, `cache_read_tokens`, `output_tokens`, `output_reasoning_tokens`, `input_audio_tokens`, `cache_audio_read_tokens`, `output_audio_tokens`, a `details` map, and `cost`.
- `messages.py:176-187`: `FinishReason = Literal['stop', 'length', 'content_filter', 'tool_call', 'error']`, "Mostly normalized to OpenTelemetry semantic convention values. Whether the agent should automatically continue is determined by `ModelResponse.state`, not by this field."
- Line 189: `ModelResponseState = Literal['complete', 'incomplete', 'suspended', 'interrupted']`. A paused or continuable reply is a state, not a finish reason.
- `models/openrouter.py:696-698`: `provider_details['downstream_provider'] = response.provider` and `provider_details['finish_reason'] = native_finish_reason`.
- Source: H-s7. Basis: Filled (Pydantic AI).

### LangChain (`langchain-ai/langchain` and `langchain-ai/langchain-google`, Python)

**H35. LangChain stores the vendor's own block list as the message content, and derives the neutral blocks from it on demand.**

- `libs/core/langchain_core/messages/content.py:8-14`: "A message to or from a model is simply a list of content blocks, allowing for the natural interleaving of text, images, and other content in a single ordered sequence. An adapter for a specific provider is responsible for translating this standard list of blocks into the format required by its API."
- `libs/core/langchain_core/messages/ai.py:243-271`, the `content_blocks` property: when `response_metadata.get("output_version") == "v1"` the content already is the standard list. Otherwise it reads `model_provider = self.response_metadata.get("model_provider")` and calls that provider's translator. With no provider it falls back to "best-effort parsing".
- `libs/core/langchain_core/messages/block_translators/__init__.py:3-10`: "`AIMessage` will first attempt to use a provider-specific translator if `model_provider` is set in `response_metadata` on the message." Registered translators: `anthropic`, `bedrock`, `bedrock_converse`, `google_genai`, `google_vertexai`, `groq`, `openai`.
- So in the default mode the stored thing is the raw vendor shape (an Anthropic block list, or Responses output items), and the tag is one string per message: `response_metadata["model_provider"]`. `libs/partners/anthropic/langchain_anthropic/chat_models.py:2637` sets `"model_provider": "anthropic"`, and `libs/partners/openai/langchain_openai/chat_models/base.py:5310` sets `response_metadata["model_provider"] = "openai"`.
- Source: H-s8, H-s9, H-s10. Basis: Filled (LangChain).

**H36. In the standard block list, vendor fields sit in an `extras` map on each block, and a block with no standard type is wrapped whole.**

- `content.py:22-36`: "provider-specific fields **within** a standard block are fully supported by default in the `extras` field of each block. ... For example, Google's thought signature" with the example `{"type": "text", "text": "J'adore la programmation.", "extras": {"signature": "EpoWCpc..."}}`.
- `content.py:456-492`, `ReasoningContentBlock`: `type: Literal["reasoning"]`, `id`, `reasoning` ("Either the thought summary or the raw reasoning text itself."), `index`, and `extras` ("Provider-specific metadata.").
- `content.py:817-855`, `NonStandardContentBlock`: "This block contains data for which there is not yet a standard type. The purpose of this block should be to simply hold a provider-specific payload." with `value: dict[str, Any]`.
- `content.py:372-453`: `ServerToolCall` (`type: "server_tool_call"`, `id`, `name`, `args`, `extras`) and `ServerToolResult` (`type: "server_tool_result"`, `tool_call_id`, `status: "success" | "error"`, `output`, `extras`).
- `content.py:126-180`: `Citation` with `type: "citation"`, an `extras` map, and a `NonStandardAnnotation` beside it.
- The `extras` map is flat. The provider is known only from the message-level tag.
- Source: H-s8. Basis: Filled (LangChain).

**H37. Anthropic: the signature is whatever non-standard key the thinking block had, copied into `extras`. Replay checks the message tag.**

- `block_translators/anthropic.py:270-283`: a `thinking` block becomes `{"type": "reasoning", "reasoning": block.get("thinking", "")}`, and every key outside `{"type", "thinking", "index", "extras"}` is copied into `extras`. That is where `signature` lands.
- Lines 497-511: any block type the translator does not know (a `redacted_thinking` block is one) becomes `{"type": "non_standard", "value": block}`.
- Lines 474-495: a block whose type ends in `_tool_result` becomes a `server_tool_result` with `"extras": {"block_type": block_type}` and the raw `content` as `output`.
- `langchain_anthropic/_compat.py:182-190`: `elif block["type"] == "reasoning" and model_provider == "anthropic":` builds a `thinking` block and takes `signature` from `block.get("extras", {}).get("signature")`. Lines 233-238: a `non_standard` block is unwrapped only when `model_provider == "anthropic"`.
- `langchain_anthropic/chat_models.py:793-798`, the default path: `if block["type"] in ("reasoning", "function_call") and (not isinstance(message, AIMessage) or message.response_metadata.get("model_provider") != "anthropic"): continue`. Reasoning from another provider is dropped without a warning.
- Lines 882-898: a native `thinking` block is sent back with the keys `type`, `thinking`, `cache_control`, `signature` only, and a `redacted_thinking` block with `type`, `cache_control`, `data` only.
- The tag is the provider name. The model name is in `response_metadata` too, and the replay check does not read it.
- Source: H-s8, H-s9. Basis: Filled (LangChain).

**H38. OpenAI Responses: output items are stored as content blocks in their own shape and sent back as they are, with one switch on `store`.**

- `langchain_openai/chat_models/_compat.py:35-59`: "`'responses/v1'` is only applicable to the Responses API. It retains information about response item sequencing and accommodates multiple reasoning items by representing these items in the content sequence", with the example block `{"type": "reasoning", "summary": [{"type": "summary_text", "text": "Reasoning summary"}], "id": "rs_123"}`.
- The older format, in the same docstring: "`'v0'` ... stores reasoning and tool outputs in `AIMessage.additional_kwargs`". `_convert_to_v03_ai_message` (lines 100-107) has the comment "Store a reasoning item in additional_kwargs (overwriting as in ...)": the old format held one reasoning item per message.
- `base.py:1338-1340`: under the heading "Behavior changed in `langchain-openai` 1.0.0", the text "Default updated to `"responses/v1"`."
- `base.py:5013-5031`, `_construct_responses_api_input`: "When `False`, the server does not persist response items, so previously returned item IDs cannot be resolved on the next turn. Assistant message IDs (`msg_*`) are therefore omitted and reasoning blocks are dropped unless they carry `encrypted_content` (which can be replayed without server-side storage)."
- Lines 5137-5156: `elif block_type == "reasoning": if store is not False or block.get("encrypted_content"): input_.append(_pop_index_and_sub_index(block))`. Fourteen other item types are passed through unchanged: `compaction`, `web_search_call`, `file_search_call`, `function_call`, `computer_call`, `custom_tool_call`, `code_interpreter_call`, `mcp_call`, `mcp_list_tools`, `mcp_approval_request`, `tool_search_call`, `tool_search_output`, `apply_patch_call`, `apply_patch_call_output`. Any other block type falls to `else: pass` and is dropped.
- Lines 5126-5133: "LangChain-auto-generated (`lc_`) IDs are not provider-issued; the Responses API rejects them (expects `msg_...`), so don't forward."
- `base.py:1284-1289`: `use_previous_response_id: bool = False`, "If `True`, always pass `previous_response_id` using the ID of the most recent response. ... Input messages up to the most recent response will be dropped from request payloads."
- This path sends blocks by their type alone and never reads `model_provider`. `_compat.py:241-293`, `_implode_reasoning_blocks`, turns any standard `reasoning` block that has text into a Responses reasoning item with that text as `summary_text`. A reasoning block from another vendor would be sent as an OpenAI reasoning item. I did not run it, so what the API answers is not recorded here.
- Source: H-s10. Basis: Filled (LangChain). Confidence: medium on the last point, which is read from code only.

**H39. Gemini: the signature rides in `extras` on text blocks, but a function call's signature is stored outside the block list, keyed by tool call id.**

- `langchain-google`, `libs/genai/langchain_google_genai/chat_models.py:1858-1887`: "Extract thought signature if present (can be on any Part type). Signatures are binary data, encode to base64 string for JSON serialization". A thought becomes `{"type": "thinking", "thinking": part.text, "signature": thought_sig}`. A text part becomes `{"type": "text", "text": ..., "extras": {"signature": thought_sig}}`.
- Lines 161-162 and 2039-2049: `_FUNCTION_CALL_THOUGHT_SIGNATURES_MAP_KEY = "__gemini_function_call_thought_signatures__"`, and "If this function_call Part has a signature, track it separately": the signature is written to `additional_kwargs[...][tool_call_id]`.
- Lines 1542-1580, replay: "First, convert all non-function-call content (text, thinking, reasoning, media, etc.) through the unified conversion path." and "Then, add function call parts". The function calls are appended after all other parts, so the reply's original order of text and function calls is not kept. Each call gets its signature back by tool call id.
- Lines 1246-1272: an older layout is still read: "Index legacy function-call signature sidecars by tool call." and "Early signature sidecars did not include an index and were appended after ...".
- Lines 1175-1185: "Placeholder injected by `_parse_chat_history` when a function-call part for a Gemini 3+ model lacks a thought signature (e.g. replayed history or a cross-provider fallback). ... a real signature copied from an unrelated response is rejected as invalid." `SKIP_THOUGHT_SIGNATURE_VALIDATOR = "skip_thought_signature_validator"`.
- Source: H-s11. Basis: Filled (LangChain).

**H40. Gemini switch rule: three provider kinds, and foreign reasoning is dropped with a warning.**

- `langchain_google_genai/_compat.py:11-27`: "Vertex and the Gemini Developer API share content and signature formats." `_NATIVE_MODEL_PROVIDERS = frozenset({"google_genai", "google_vertexai"})`, and `_classify_model_provider` returns `"native"`, `"foreign"`, or `"unknown"` (when the message has no tag).
- Lines 222-242: for a foreign provider, "Another provider's chain-of-thought (for example AWS Bedrock or OpenAI summary blocks) is not a Gemini thought; sending it back would leak foreign reasoning text into Gemini requests." with the warning "Dropping v1 reasoning block from provider %r; foreign reasoning is not replayed as a Gemini thought part." For a native block with no signature: "Dropping v1 reasoning block with no thought signature; its text will not be sent back to the model."
- Lines 213-217: a text block's signature is sent only when `provider_kind != "foreign"`. `chat_models.py:1291-1310`, `_strip_foreign_signature`: "Return a block without another provider's thought signature."
- `chat_models.py:1406-1413`: a foreign `non_standard` block is dropped with "Dropping non-standard content block that cannot be represented as a Gemini part".
- This disagrees with Pydantic AI on one fact. LangChain says Vertex and the Gemini Developer API "share content and signature formats" and treats both as native. Pydantic AI says "the Gemini API and Google Cloud reject each other's" signatures (H30). See Conflicts.
- Source: H-s11. Basis: Filled (LangChain).

**H41. Neutral usage has cache and reasoning details. The finish reason is a set of keys in `response_metadata`, raw and normalized side by side.**

- `libs/core/langchain_core/messages/ai.py:38-153`: `UsageMetadata` has `input_tokens`, `output_tokens`, `total_tokens`, `input_token_details` (`audio`, `cache_creation`, `cache_read`), and `output_token_details` (`audio`, `reasoning`).
- `langchain_anthropic/chat_models.py:3404-3423`: `_FINISH_REASONS` maps `end_turn` and `stop_sequence` to `"stop"`, `max_tokens` to `"length"`, `tool_use` to `"tool_call"`, `refusal` to `"refusal"`, and `model_context_window_exceeded` to `"context_exceeded"`. `_finish_reason_fields` returns three keys: `"finish_reason": stop_reason` (the raw value), `"normalized_finish_reason"`, and `"finish_state": "suspended" if stop_reason == "pause_turn" else "complete"`, with the comment "`pause_turn` means the caller must send another request to finish."
- The Anthropic chat model's vendor-only fields (lines 1483-1628): `betas`, `thinking`, `output_config`, `mcp_servers`, `context_management`, `container`, `reuse_last_container`, `inference_geo`. `reuse_last_container`: "Automatically reuse container from most recent response (code execution)."
- Source: H-s8, H-s9. Basis: Filled (LangChain).

**H42. LangChain's OpenRouter client keeps `reasoning_details` whole in `additional_kwargs`, and has two repairs it applies before sending it back.**

- `libs/partners/openrouter/langchain_openrouter/chat_models.py:1369-1370`: `additional_kwargs["reasoning_details"] = reasoning_details`, with `response_metadata={"model_provider": "openrouter"}` (line 1387). Lines 1327-1330 send it back as `message_dict["reasoning_details"]`.
- Lines 1209-1232, `_merge_reasoning_details`: "During streaming, `AIMessageChunk.__add__` list-concatenates `reasoning_details` in `additional_kwargs`, fragmenting a single entry into many. When serialized back to the API ... these fragments cause `BadRequestResponseError` on multi-turn conversations (the provider rejects the malformed thinking block with `Invalid signature`)." The repair groups fragments by `(type, index)`, joins `text`, and takes the last non-null value of `format` and `signature`.
- Lines 1196-1206, `_strip_ephemeral_reasoning_ids`: "Remove OpenAI Responses reasoning item IDs from outbound payloads." It deletes any `id` that starts with `rs_`.
- Lines 94-95: `generation_info["native_finish_reason"] = native_finish_reason`.
- This is client code written for OpenRouter. It shows two things a client had to do to make the field replay. It does not show the translator's own logic, and the lead agent may treat it as a basis for an inference about OpenRouter.
- Source: H-s12. Basis: Filled (LangChain).

### TensorZero (`tensorzero/tensorzero`, Rust gateway)

**H43. The neutral reply is an ordered list of content blocks with two vendor-state block types: `thought` and `unknown`.**

- `crates/tensorzero-types/src/content.rs:243-265`, `Thought`: `text: Option<String>`, `signature: Option<String>` ("An optional signature - used with Anthropic and OpenRouter for multi-turn reasoning conversations. Other providers will ignore this field."), `summary: Option<Vec<ThoughtSummaryBlock>>`, `provider_type: Option<String>`, and `extra_data: Option<Value>` ("Provider-specific opaque data for multi-turn reasoning support. For example, OpenRouter stores encrypted reasoning blocks with `{"format": "...", "encrypted": true}` structure.").
- Lines 102-117, `Unknown`: "Struct that represents an unknown provider-specific content block. We pass this along as-is without any validation or transformation." with `data: Value`, `model_name: Option<String>`, `provider_name: Option<String>`.
- Both types are part of the public inference API. `docs/gateway/api-reference/inference.mdx:484-496`: for `thought`, "Pass through the `signature` from the model's response to continue a reasoning conversation." For `unknown`: "`data`: The original content block from the provider, without any validation or transformation by TensorZero." and "If set, the content block will only be provided to this specific model." and "If neither `model_name` nor `provider_name` is set, the content block is passed to all model providers."
- The repository's `main` branch on 2026-11-22 is commit `62eb8f6` dated 2026-06-04, the same commit the workspace report of 2026-11-17 pinned.
- Source: H-s13. Basis: Filled (TensorZero).

**H44. The switch rule is a filter that runs before each provider request: a thought is dropped when its provider type differs, and an unknown block is dropped when its model or provider name differs.**

- `content.rs:251-259`: "When set, this `Thought` block will only be used for providers matching this type (e.g. `anthropic`). Other providers will emit a warning and discard the block."
- `crates/tensorzero-core/src/model.rs:726-743`: "When a thought is scoped to a particular provider type, we discard if it doesn't match our target provider. Thoughts without a `provider_type` are used for all providers."
- `model.rs:649-669`, `should_filter_unknown_block`: "If model_name is specified and doesn't match, filter it out" and "If provider_name is specified and doesn't match, filter it out".
- `model.rs:1617-1645`, `thought_block_provider_type`: each provider has its own type string, so `anthropic`, `aws_bedrock`, `gcp_vertex_anthropic`, and `gcp_vertex_gemini` are four different types. A thought from Anthropic's own API is dropped when the next request goes to Claude on Vertex.
- The thought's tag is the provider type only. The unknown block's tag is the model name plus the provider name.
- Source: H-s13. Basis: Filled (TensorZero).

**H45. Anthropic: a thinking block and a redacted block both become a `Thought`, told apart by whether `text` is present.**

- `crates/tensorzero-core/src/providers/anthropic.rs:1244-1262`: thinking becomes `Thought { text: Some(thinking), signature: Some(signature), provider_type: Some("anthropic") }`, and redacted thinking becomes `Thought { text: None, signature: Some(data), ... }`.
- Lines 828-843, replay: with text, `AnthropicMessageContent::Thinking { thinking, signature }`. With no text and a signature, `AnthropicMessageContent::RedactedThinking { data: signature }`.
- Lines 1263-1267: any block type the provider does not know becomes `ContentBlockOutput::Unknown(Unknown { data, model_name: Some(...), provider_name: Some(...) })`, and line 844 sends an unknown block's `data` back as it is. This is how a server tool block survives: unread, tagged with the model and provider that produced it.
- A thinking block whose text is empty and a redacted block both arrive at replay as "signature with no text" only if the provider stored `text: None`. The Anthropic path stores `Some("")` for empty thinking, so the two stay distinct. This is a reading of the code, not something I ran.
- Source: H-s13. Basis: Filled (TensorZero).

**H46. Gemini: the neutral type has no place for a signature on a function call, so the signature becomes a separate thought block next to the part, and replay merges them again.**

- `crates/tensorzero-core/src/providers/google_ai_studio_gemini.rs:791-811`: "Google AI Studio can emit `thoughtSignature` attached to arbitrary parts (including function calls). Their API expects us to pass back the part with the 'thoughtSignature' attached. Since the TensorZero model only supports standalone thought blocks, we emit a Thought block with just the signature, immediately following the original part. When constructing the input, we merge these blocks with their predecessor".
- The code emits the signature-only thought before it processes the part's data (line 801: "Add a thought chunk to the output, then continue on to process 'part.data'"), and the input side attaches it to the block that follows. `crates/tensorzero-core/src/providers/gcp_vertex_gemini/mod.rs:2311-2356`: a thought with a signature and no text takes `iter.next()` and builds `GCPVertexGeminiContentPart { thought: false, thought_signature: Some(signature.clone()), data: gcp_part }`. So the comment says "following" and "predecessor" while the code puts the thought first and merges it with its successor.
- The same function has three error strings for a block list that was edited: "Thought block with signature must be followed by a content block in {provider_type}", "Thought block with signature cannot be followed by another thought block in {provider_type}", and "Thought block with signature cannot be followed by an unknown block in {provider_type}".
- So a flat block list can hold a per-part signature by position, and the price is that any reordering or removal between the two blocks turns into an error or a signature on the wrong part.
- Source: H-s13. Basis: Filled (TensorZero).

**H47. OpenAI Responses: a reasoning item becomes one `Thought` with the encrypted content as the signature. The item id is discarded.**

- `crates/tensorzero-core/src/providers/openai/responses.rs:211-239`: `OpenAIResponsesOutputInner::Reasoning { encrypted_content, summary }` becomes `Thought { text: None, signature: encrypted_content, summary: Some(...), provider_type: Some("openai") }`.
- Lines 1014-1062, replay: a thought is sent back only `if let Some(encrypted_content) = &thought.signature`, as `OpenAIResponsesReasoning { encrypted_content, summary }`. A thought with no encrypted content is not sent.
- Lines 240-246 and 1064-1069: every other output item type becomes an `Unknown` block and is sent back as `OpenAIResponsesInput::Unknown(data)`.
- Line 76: the include value `reasoning.encrypted_content` is a named option.
- Source: H-s13. Basis: Filled (TensorZero).

**H48. Through OpenRouter, TensorZero keeps the `format` string in `extra_data` and replays the thought only to the OpenRouter provider type.**

- `crates/tensorzero-core/src/providers/openrouter.rs:1774-1798`: a text or summary detail becomes a thought with `extra_data: format.map(|f| json!({"format": f}))`, and an encrypted detail one with `extra_data: Some(json!({"format": format, "encrypted": true}))`.
- Lines 1214-1218: `if thought.provider_type.as_deref() == Some(PROVIDER_TYPE)` the thought is sent, otherwise `warn_discarded_thought_block(PROVIDER_TYPE, thought)`.
- This is TensorZero's client side of OpenRouter. It shows a gateway treating OpenRouter as one provider type, and the block records OpenRouter's `format` string in place of the upstream vendor's name.
- Source: H-s13. Basis: Filled (TensorZero).

**H49. Usage is four counters, and the raw vendor usage and raw response are available on request. The finish reason is a six-value enum, and the vendor's own word is only in the raw response.**

- `crates/tensorzero-types/src/usage.rs:26-39`: `input_tokens`, `output_tokens`, `provider_cache_read_input_tokens`, `provider_cache_write_input_tokens`, `cost`.
- Lines 110-120, `RawUsageEntry`: "This preserves the original provider-specific usage object for fields that TensorZero normalizes away (e.g., OpenAI's `reasoning_tokens`, Anthropic's `cache_read_input_tokens`)." It has `provider_type`, `api_type`, and `data`. The inference API returns it "only when `include_raw_usage` is `true`" (`inference.mdx:1014-1022`). Reasoning tokens are only in the raw entry.
- Lines 140-151: `FinishReason` is `Stop`, `StopSequence`, `Length`, `ToolCall`, `ContentFilter`, `Unknown`.
- `model.rs:1406-1410`: `discard_unknown_chunks`, "If `true`, we emit a warning and discard chunks that we don't recognize ... By default, unknown chunks are forwarded as-is in the stream."
- Source: H-s13. Basis: Filled (TensorZero).

### rig (`0xPlaygrounds/rig`, Rust)

**H50. rig holds each provider item whole beside its neutral block, and tags the turn with the wire, the provider, and the model.**

- `crates/rig-core/src/completion/message.rs:35-52`, `AssistantMessage`: "`content` holds one block per provider output item, in the order the provider produced them. `origin` names the wire, provider and model that produced the turn; a hand-built turn has none and always replays from its canonical fields." Fields: `content: Vec<AssistantContent>`, `origin: Option<Origin>`, `stop: Option<StopReason>`.
- Lines 192-208, `AssistantContent`: `Text`, `ToolCall`, `Reasoning`, `Image`, and `Opaque` ("A provider item with no canonical meaning.").
- Lines 382-399, `Reasoning`: "Signatures, encrypted payloads and ids are provider data and live in `native`." Fields: `text`, `redacted`, `native: Option<Native>`.
- `crates/rig-core/src/completion/message/native.rs:232-262`, `Native`: "One provider item, verbatim in its API's JSON shape, beside the canonical block (or message) it was decoded to." Fields: `item: serde_json::Value`, `fingerprint`, `baseline`, `edited`, and `blob_slot` ("Where the item keeps its reasoning blob, as a JSON pointer such as `/signature`").
- Lines 378-388, `Opaque`: "A provider item with no canonical meaning, such as a hosted-tool step or a compaction record. Only the API that produced it reads it." Fields: `item` and `replay` ("A client-executed call nothing answers is kept but never sent.").
- Lines 55-81, `Origin`: `api` (examples in the doc: `"anthropic.messages"`, `"openai.responses"`, `"openai.chat"`), `provider`, `model` ("The model the request named."), `response_model`, `response_id`, and `context: Option<Fingerprint>`.
- This is one slot shape for every vendor: raw JSON per block, plus one tag per turn. The commit read is `a8673f7` of 2026-11-22. The workspace report of 2026-11-17 read `5393668`, and the types above (`Origin`, `Native`, `Opaque`) are the form at today's commit.
- Source: H-s14. Basis: Filled (rig).

**H51. The switch rule is an exact match on wire, provider, and model. A turn from anywhere else is rebuilt from the neutral fields.**

- `native.rs:55-59`: "Replay compares it, with `api` and `provider`, against the target: only an exact match replays the turn's provider items." Lines 97-101, `same_model`: "The one sameness rule replay uses."
- `crates/rig-core/src/completion/history.rs:463-490`, the `adapt` function's rules: "A turn whose origin names `target`'s API, provider and model keeps its provider items." and "Any other turn keeps only canonical fields: provider items are cleared, reasoning becomes plain text, or stays reasoning when the request asks for it and the model reads it ...; redacted or empty reasoning is dropped; `Opaque` items are dropped and call ids go through `ReplayTarget::normalize_tool_call_id`, made distinct when two calls would share one."
- Also there: "so is a turn that ended in an error, with the results answering it. An aborted turn is read as the text it had sent, as a turn from no known model".
- Source: H-s14. Basis: Filled (rig).

**H52. rig detects an edit to a block and stops replaying the stored item for it.**

- `native.rs:3-5`: "Its baseline or `Fingerprint` detects edits so an edited block stops replaying its stale item."
- `history.rs:398-408`, `Replay`: `Item` ("The provider item, still current, with a call's id slot spelled."), `Identity` ("The block was edited: rebuild it from its canonical fields, keeping these keys of its stale item."), and `Rebuild`.
- `native.rs:132-136`: the fingerprint is "A 64-bit FNV-1a hash of a value's JSON serialization with every object's keys in sorted order and every whole number written as an integer, so a store that reorders keys (Postgres `jsonb`, sorted-key dumps) or writes `20.0` as `20` never changes it."
- Source: H-s14. Basis: Filled (rig).

**H53. rig binds a turn to the request's tools and system prompt for the vendors that check them.**

- `native.rs:74-80`, `Origin.context`: "The fingerprint of the request's tools and system prompt. A wire that binds its items to them (`ReplayTarget::binds_context`) replays a turn made under another context as if from another model."
- `history.rs:290-304`: `binds_context`, "Whether `model` binds its provider items to the request's tools and system prompt", and `drops_unbound_items`, "Whether `request` asks the provider to drop an item bound to another context itself (Anthropic's `drop_block`), so a turn made under another context replays verbatim rather than as another model's."
- `crates/rig-core/src/providers/anthropic/wire.rs:862-864` implements `binds_context` from a per-model catalog flag.
- Source: H-s14. Basis: Filled (rig).

**H54. Per-vendor replay facts are small declared values on one trait, and they show where the vendors differ.**

- `history.rs:266-274`, `blob_slots`: "Where this wire's items keep a reasoning blob (a signature or an encrypted copy of the reasoning), as JSON pointers tried in order." The values: Anthropic `["/signature", "/data"]` (`providers/anthropic/wire.rs:887-889`), Gemini `["/thoughtSignature"]` (`providers/gemini/completion.rs:315-317`), OpenAI Responses `["/encrypted_content"]` (`providers/openai/responses_api/wire.rs:228-230`), and OpenAI chat `["/thinking_blocks/*/signature", "/thinking_blocks/*/data"]` (`providers/openai/wire/chat.rs:999-1002`).
- `history.rs:312-318`, `needs_next`: "Whether this provider item requires the item after it in its turn (Responses reasoning): when that one is not replayed, neither is this." Implemented at `providers/openai/responses_api/wire.rs:248-250` as `item.str("type") == Some("reasoning")`.
- `history.rs:320-328`, `hosted_pair`: "A use and its result replay only together, in one turn or across the model's turns. A use still running when the last turn ended (a paused turn, or one whose client call the hosted tool made) replays alone."
- `history.rs:224-231`, `normalize_tool_call_id`: "The id `id` takes when a call another model made is sent to `model` on this wire." Anthropic (`providers/anthropic/wire.rs:829-837`): "Anthropic takes call ids of `[a-zA-Z0-9_-]`, at most 64 long." OpenAI chat (`providers/openai/wire/chat.rs:944-978`): "A `call|item` id joins its sanitized halves with `_`, ending a result over 40 characters in a hash of the whole id; OpenAI's own ids are cut to 40".
- `providers/gemini/completion.rs:824`: for a Gemini 3 model a call with no stored signature gets `json!("skip_thought_signature_validator")`.
- `history.rs:233-240`, `continues_stored`: "Whether `request` continues a conversation the provider stores, such as one naming a previous interaction."
- `providers/openai/responses_api/wire.rs:232-234`: "An edited block keeps its item's `id` and `type`, a message its `phase`, and reasoning its ciphertext, so the items after it stay paired."
- `providers/openai/wire/chat.rs:1102-1105`: on the chat-completions wire rig keeps two extension fields verbatim: "`reasoning_details`, and the signed `thinking_blocks` a LiteLLM proxy returns for Claude." Lines 980-983: "Signed data such as `reasoning_details` or `thinking_blocks` never survives an edit."
- Source: H-s14. Basis: Filled (rig).

**H55. Neutral controls are a closed set, and each wire must answer for each one.**

- `crates/rig-core/src/completion/options/mapping.rs:17-36`, `OptionFields`: `reasoning`, `cache`, `service_tier`, `verbosity`, `parallel_tool_calls`, `top_p`, `seed`, `stop`, `cache_key`. Lines 1-4: "Neither has `..` or a default, so a new option fails to compile in every wire until it answers for it."
- `crates/rig-core/src/completion/options.rs:225-264`: `Reasoning` is `Off`, `Effort(Effort)`, or `Budget { tokens }`, and `Effort` is `Minimal`, `Low`, `Medium`, `High`, `XHigh`, `Max`. Lines 289-299: `CacheRetention` is `None`, `Short` ("typically minutes"), `Long` ("typically an hour or more").
- `mapping.rs:108-127`, `Mapping`: `Nothing`, `Send(Value)`, `Omit` ("Honour the option by sending nothing: the provider default already does what was asked."), `Place` ("Valid for `cache` only."), and `Unsupported(String)` ("an error under `OnUnsupported::Error`, a warning under `OnUnsupported::Ignore`").
- Anything else goes through `additional_params`, merged over the body. `history.rs:598-606`: hosted tools are passed "as is in the `tools` of its `additional_params`".
- `native.rs:104-123`, `StopReason`: `Stop`, `Length`, `ToolUse`, `Error(String)`, `Aborted(String)`.
- `crates/rig-core/src/completion/request.rs:687-720`, `Usage`: `input_tokens`, `output_tokens`, `total_tokens`, `cached_input_tokens`, `cache_creation_input_tokens`, `tool_use_prompt_tokens`, `reasoning_tokens`, `cost`.
- Source: H-s14. Basis: Filled (rig).

### genai (`jeremychone/rust-genai`, Rust)

**H56. genai's slot is a bare signature string as its own content part, plus a raw-JSON custom part tagged with the model.**

- `src/chat/content_part/common.rs:12-38`, `ContentPart`: `Text(String)`, `Binary`, `ToolCall`, `ToolResponse`, `ThoughtSignature(String)`, `ReasoningContent(String)`, `Custom(CustomPart)`.
- `src/chat/content_part/custom_part.rs:8-14`: "Custom Content Part containing the raw json value of that part from the provider/model. `.model_iden` will always be captured if this is part of a response". Fields: `model_iden: Option<ModelIden>` and `data: Value`.
- `src/chat/tool/tool_call.rs:20-27`: a tool call also has `thought_signatures: Option<Vec<String>>`.
- `ThoughtSignature` holds a bare string. Which provider or model wrote it is known only from where it sits in the history.
- Source: H-s15. Basis: Filled (genai).

**H57. The one signature part means three different things, one per adapter, and the pairing is by position.**

- Anthropic, `src/adapter/adapters/anthropic/adapter_shared.rs:672-677`: a thinking block becomes `ContentPart::ThoughtSignature(signature)` followed by `ContentPart::ReasoningContent(reasoning)`. Lines 296-330, replay: "Anthropic only accepts a thinking block when it carries both its reasoning text and the matching signature, so the two lists must line up one-to-one. They can legitimately fail to line up: a conversation carried over from a provider that returns unsigned reasoning (OpenAI, DeepSeek, ...) ... Pairing by position anyway would sign the wrong text, which Anthropic rejects, so drop the thinking blocks and send the rest of the message instead." The warning: "anthropic - assistant message has unpaired reasoning content and thought signatures; omitting thinking blocks from this message".
- Gemini, `src/adapter/adapters/gemini/adapter_impl.rs:195-198`: "Parts keep the order the API returned them in, and a thought signature sits immediately before the part it arrived on (a call also mirrors its own in `thought_signatures`)." Line 18: `SKIP_THOUGHT_SIGNATURE_VALIDATOR`.
- OpenAI Responses, `src/adapter/adapters/openai_resp/adapter_impl.rs:568-603`: "encrypted reasoning blobs from prior turns must be carried back as top-level `{type: "reasoning"}` input items to keep the Responses-API prefix cache warm." Each `ThoughtSignature(blob)` is sent as `{"type": "reasoning", "encrypted_content": blob, "summary": []}`. Lines 216-219: the adapter asks for `include: ["reasoning.encrypted_content"]` as an opt-in.
- Since the part is untagged, a signature that Anthropic produced is, after a switch to an OpenAI Responses model, sent as `encrypted_content`. The count check protects the Anthropic direction only. This is read from the code. I did not run it.
- A `redacted_thinking` block is kept as a `Custom` part (`adapter_shared.rs:950-955`), and a Gemini server tool part is kept as `ContentPart::Custom(CustomPart { model_iden: Some(...), data: raw })` (`gemini/adapter_impl.rs:231-237`).
- Source: H-s15. Basis: Filled (genai). Confidence: medium on the cross-vendor statement.

**H58. genai's neutral controls and its stop reason, which keeps the raw string inside each variant.**

- `src/chat/chat_options.rs:30-117`: `temperature`, `max_tokens`, `top_p`, `stop_sequences`, `response_format`, `tool_choice`, `normalize_reasoning_content`, `reasoning_effort`, `verbosity`, `seed`, `service_tier`, `extra_headers`, `cache_control`, `prompt_cache_key`, `extra_body`.
- Lines 311-325, `ReasoningEffort`: `Zero`, `Low`, `Medium`, `High`, `XHigh`, `Max`, and a budget variant ("Provider-specific reasoning token budget.").
- `src/chat/chat_message.rs:130-140`, `CacheControl`: `Ephemeral`, `Memory`, `Ephemeral5m`, and a one hour variant, set per message.
- `src/chat/chat_response.rs:22-35`, `StopReason`: `Completed(String)`, `MaxTokens(String)`, `ToolCall(String)`, `ContentFilter(String)`, `StopSequence(String)`, `Other(String)`. Each variant holds the vendor's own word.
- `src/chat/usage.rs:15-98`: an OpenAI-shaped usage with `prompt_tokens_details` (`cache_creation_tokens`, `cache_creation_details` with `ephemeral_5m_tokens` and `ephemeral_1h_tokens`, `cached_tokens`) and `completion_tokens_details` (`reasoning_tokens`).
- Source: H-s15. Basis: Filled (genai).

### pi (`earendil-works/pi`, package `@earendil-works/pi-ai` 1.6.0, TypeScript)

**H59. pi's reply has three block types, a signature string on each, and the wire, provider, and model on the message.**

- `packages/ai/src/types.ts:595-612`, `AssistantMessage`: `content: (TextContent | ThinkingContent | ToolCall)[]`, `api: Api`, `provider: ProviderId`, `model: string`, `responseModel?`, `responseId?`, `usage`, `stopReason`, `rawStopReason?`.
- Lines 432-467: `TextContent.textSignature?` ("e.g., for OpenAI responses, message metadata (legacy id string or TextSignatureV1 JSON)"), `ThinkingContent.thinkingSignature?` ("Provider-specific opaque or serialized reasoning replay data") with `redacted?: boolean` ("The opaque encrypted payload is stored in `thinkingSignature` so it can be passed back to the API for multi-turn continuity."), and `ToolCall.thoughtSignature?` ("Google-specific: opaque signature for reusing thought context").
- Lines 66-76, `KnownApi`: `openai-completions`, `mistral-conversations`, `openai-responses`, `azure-openai-responses`, `openai-codex-responses`, `anthropic-messages`, `bedrock-converse-stream`, `google-generative-ai`, `google-vertex`, `pi-messages`.
- The block union is closed at these three types. A vendor block of any other kind, such as a server tool block, is left out of a pi message.
- The repository answers at `earendil-works/pi`, and the clone's origin URL is `https://github.com/earendil-works/pi.git`.
- Source: H-s16. Basis: Filled (pi).

**H60. What the signature string holds depends on the wire that wrote it.**

- Anthropic, `packages/ai/src/api/anthropic-messages.ts:992-1001`: `thinkingSignature: event.content_block.signature ?? ""`, and for redacted thinking `thinking: "[Reasoning redacted]", thinkingSignature: event.content_block.data, redacted: true`. Lines 1708-1713 send `{ type: "redacted_thinking", data: block.thinkingSignature! }` back.
- OpenAI Responses, `packages/ai/src/api/openai-responses-shared.ts:333-337`: `const reasoningItem = JSON.parse(block.thinkingSignature) as ResponseReasoningItem; output.push(reasoningItem);`. The whole reasoning item, id and encrypted content included, is serialized into the string.
- Chat completions with `reasoning_details` (OpenRouter), `packages/ai/src/api/openai-completions.ts:613-623`: "Keep provider replay data in the existing signature slot. OpenRouter streams reasoning_details as deltas: consecutive text/summary deltas are merged into logical entries, while encrypted entries remain opaque and discrete." Line 344 stores `block.thinkingSignature = JSON.stringify(streamedReasoningDetails)`, and lines 1307-1314 and 1378-1380 parse it back into `assistantMsg.reasoning_details`.
- Gemini, `packages/ai/src/api/google-shared.ts:118-126`: "`thoughtSignature` can appear on ANY part type (text, functionCall, etc.) - it does NOT indicate the part itself is thinking content." and "When persisting/replaying model outputs, signature-bearing parts must be preserved as-is; do not merge/move signatures across parts." A text part's signature goes in `textSignature`, a thought's in `thinkingSignature`, and a call's in `thoughtSignature` (lines 238-276).
- A tool call id from the Responses API is the pair `call_id|item_id` in one string: ``id: `${item.call_id}|${item.id}` `` (`openai-responses-shared.ts:560`).
- Source: H-s16. Basis: Filled (pi).

**H61. The switch rule: same provider, same wire, same model, or the vendor state is removed.**

- `packages/ai/src/api/transform-messages.ts:122-125`: `const isSameModel = assistantMsg.provider === model.provider && assistantMsg.api === model.api && assistantMsg.model === model.id;`.
- Lines 128-137: "Thinking from another model is dropped. Signatures and encrypted reasoning are only valid for the model that produced them, and sending readable thinking as plain text makes the new model treat it as part of the visible reply." and "Keep redacted and signed blocks even if the thinking text is empty (OpenAI encrypted reasoning); they are needed for replay."
- Lines 140-146: text from another model loses `textSignature`. Lines 152-155: a tool call from another model loses `thoughtSignature`.
- Lines 216-224: "Skip errored/aborted assistant messages entirely. ... Replaying them can cause API errors (e.g., OpenAI "reasoning without following item")".
- Lines 179-207: an unanswered tool call gets a synthetic result with the text "No result provided" and `isError: true`.
- `google-shared.ts:157-162`: "Only keep signatures from the same provider/model and with valid base64."
- Source: H-s16. Basis: Filled (pi).

**H62. Tool call ids are rewritten when a call crosses to another model.**

- `transform-messages.ts:86-90`: "OpenAI Responses API generates IDs that are 450+ chars with special characters like `|`. Anthropic APIs require IDs matching ^[a-zA-Z0-9_-]+$ (max 64 chars)."
- Lines 157-163: the rewrite runs only when `!isSameModel`, and a map applies the new id to the matching tool result.
- `openai-responses-shared.ts:202-225`: each half of the id is cut to 64 characters of `[a-zA-Z0-9_-]`. A foreign item id becomes `fc_` plus a short hash, because "OpenAI Responses API requires item id to start with "fc"".
- Lines 365-374: "For different-model messages, set id to undefined to avoid pairing validation. OpenAI tracks which item IDs were paired with rs_xxx reasoning items. By omitting the id, we avoid triggering that validation (like cross-provider does)."
- Source: H-s16. Basis: Filled (pi).

**H63. pi's neutral controls, usage, and stop reason.**

- `types.ts:223-279` and `:387-394`: `temperature`, `maxTokens`, `cacheRetention`, `sessionId`, `metadata`, and in the simple options `toolChoice`, `reasoning?: ThinkingLevel`, `thinkingBudgets`.
- Lines 469-497, `Usage`: `input`, `output`, `cacheRead`, `cacheWrite`, `cacheWrite1h` ("Only Anthropic reports this split."), `reasoning` ("a subset of `output`"), `totalTokens`, and a `cost` object with `totalFromProvider`.
- Line 499: `StopReason = "pending" | "stop" | "length" | "toolUse" | "error" | "aborted" | "deferred"`. The vendor's own word is kept in `rawStopReason` (line 612).
- Lines 603-606: the message records `providerThinkingLevel` ("Exact provider-native effort level used for this response.") beside the neutral `thinkingLevel`.
- Source: H-s16. Basis: Filled (pi).

### opencode (`anomalyco/opencode`, package `@opencode/ai` 2.3.0 and the session store, TypeScript)

**H64. opencode has two layers: a request-time message type with a provider-keyed metadata map per part, and a stored session message with one `native` record per block.**

- Request layer, `packages/ai/src/schema/options.ts:42-45`: `ProviderMetadata = Schema.Record(Schema.String, Schema.Record(Schema.String, Schema.Unknown))`. `packages/ai/src/schema/messages.ts:41-47`, `:139-149`, `:187-193`: `TextPart`, `ToolCallPart`, `ToolResultPart`, and `ReasoningPart` each have `cache: Schema.optional(CacheHint)` and `providerMetadata`. `ReasoningPart` also has `encrypted: Schema.optional(Schema.String)`, and tool parts have `providerExecuted`.
- Lines 262-271: `AssistantMessage.content` is an array of `TextPart, ReasoningPart, ToolCallPart, ToolResultPart, MediaPart, CompactionPart`.
- Lines 201-208, `CompactionPart`: `provider: ProviderID`, `id`, and either `encrypted` or `text`. It is the one part that names its provider in a typed field.
- Stored layer, `packages/schema/src/session-message.ts:37-44`, `Native`: "Provider-owned replay data, keyed by the producing provider's metadata fields and echoed back verbatim. A tool's result data is merged into its call data, so adapters must not reuse a key with a different value across the two." It is `Schema.Record(Schema.String, Schema.Unknown)`.
- Lines 171-203 and 222-238: `AssistantText`, `AssistantReasoning`, and `AssistantTool` each have `native`, and the `Assistant` message has `model: Model.Ref`, `finish`, `rawFinish`, and its own `native` ("Step-level native data, such as a provider's reason for blocking a response.").
- Source: H-s17. Basis: Filled (opencode).

**H65. The switch rule has two levels: model-scoped state and provider-scoped state.**

- `packages/core/src/session/runner/to-llm-message.ts:146-148`: `const sameProvider = message.model.providerID === model.providerID` and `const sameModel = sameProvider && message.model.id === model.id`.
- Lines 181-200: "Let the destination adapter handle readable reasoning after a model/provider switch." With the same model, reasoning goes with its metadata. Otherwise it goes as `{ type: "reasoning", text: item.text }` when it has text, and is dropped when it has none.
- Lines 202-205: "Call-side metadata is model-scoped proof of generation (Gemini thought signatures, OpenAI encrypted reasoning): only the producing model may replay it."
- Lines 207-210: "Hosted tools (e.g. google_search) run inside the provider, so their native data is provider-format result data rather than model-scoped proof: it stays replayable across models of the same provider."
- Lines 159-170: "A stream cut by Esc, a crash or a failure may end mid-block, so its last block's provider state is not replayed, and reasoning is only replayed once the model moved on to output."
- The workspace research of 2026-09-20 recorded "OpenCode's `differentModel` guard strips signatures". Today's code has the two named levels above. The idea is the same and the names changed.
- Source: H-s17. Basis: Filled (opencode).

**H66. Per-vendor shapes in opencode's metadata.**

- Anthropic, `packages/ai/src/protocols/anthropic-messages.ts:1276-1289`: "Redacted thinking surfaces as an empty reasoning part carrying the opaque payload as `redactedData` metadata (same model as Vercel's @ai-sdk/anthropic)." Lines 913-929: `const signature = part.encrypted ?? signatureFromMetadata(part.providerMetadata, providerMetadataKey)`.
- Anthropic server tools, lines 614-636: "Server tool result blocks are typed by name. Anthropic ships three today; extend this list when new server tools land. The block content is the structured payload returned by the provider, which we round-trip as-is." The payload is read from `part.providerMetadata?.[providerMetadataKey]?.["result"]`. Any other tool name fails with "Anthropic Messages does not know how to round-trip server tool result for ${part.name}".
- Anthropic compaction, lines 902-906: "Compaction state must be replayed to its originating provider and API" is an error, not a silent drop.
- Line 598: `scrubToolCallID = (id: string) => id.replace(/[^a-zA-Z0-9_-]/g, "_")`.
- Gemini, `packages/ai/src/protocols/gemini.ts:285-294` and `:361-369`: `providerMetadata(key, { thoughtSignature: signature })`, and on replay `thoughtSignature: signature ?? (!hasSignedToolCall ? SKIP_THOUGHT_SIGNATURE_VALIDATOR : undefined)`.
- OpenAI Responses, `packages/ai/src/protocols/open-responses.ts:1082`: `providerMetadata(state, { itemId: itemID, reasoningEncryptedContent: item.encryptedContent ?? null })`, the same two key names as the Vercel provider. `packages/ai/src/protocols/openai-responses.ts:438`: the adapter's default options are `{ providerOptions: { store: false, include: ["reasoning.encrypted_content"] } }`.
- Source: H-s17. Basis: Filled (opencode).

**H67. opencode's neutral controls, usage, and finish reason.**

- `packages/ai/src/schema/options.ts:194-195`: `ReasoningEfforts = ["none", "minimal", "low", "medium", "high", "xhigh", "max"]`, as an open string type.
- Lines 408-445: `CacheHint` is `{ type: "ephemeral" | "persistent", ttl? }`, and `CachePolicy` is `"auto"`, `"none"`, or an object with `tools`, `system`, `messages`, and `ttl`. The comment says `"auto"` is the default for agent loops, and that "it places breakpoints at the last tool definition, the first and last distinct system parts, and the conversation tail".
- `packages/ai/src/schema/events.ts:64-80`, `Usage`: `contextTokens`, `inputTokens`, `outputTokens`, `nonCachedInputTokens`, `cacheReadInputTokens`, `cacheWriteInputTokens`, `reasoningTokens`, `totalTokens`, `cost`, `providerMetadata`. Lines 59-62: "`providerMetadata` always carries the provider's raw usage payload", and it is "keyed by provider name". Lines 75-78, `cost`: "USD the endpoint says it charged for this request, as OpenRouter, LiteLLM and OpenCode Zen report it in `usage.cost`."
- Lines 269-276, `FinishReasonDetails`: `normalized`, `raw`, `category`, `explanation`. `packages/schema/src/llm.ts:5`: `FinishReason` is `"stop", "length", "tool-calls", "content-filter", "error", "unknown"`.
- Source: H-s17. Basis: Filled (opencode).

### Goose (`block/goose`, Rust)

**H68. Goose has typed thinking and redacted-thinking blocks with no tag on them, a free metadata map on tool requests, and the provider and model on the message.**

- `crates/oi-kit-provider-types/src/conversation/message.rs:319-331`, `MessageContentBlock`: `Text`, `Image`, `Document`, `ToolRequest`, `ToolResponse`, `ToolConfirmationRequest`, `ActionRequired`, `Thinking`, `RedactedThinking`, `SystemNotification`, `Error`.
- Lines 232-240: `ThinkingContentBlock { thinking: String, signature: String }` and `RedactedThinkingContentBlock { data: String }`.
- Lines 81-83: "Provider-specific metadata for tool requests/responses. Allows providers to store custom data without polluting the core model." `pub type ProviderMetadata = serde_json::Map<String, serde_json::Value>;`. Lines 133-141: `ToolRequest` has `metadata: Option<ProviderMetadata>`.
- Lines 769-776, `InferenceMetadata`: `provider`, `requested_model`, `resolved_model`, `provider_session_id`, held in `MessageMetadata.inference`.
- Lines 780-801, `MessageUsage`: `input_tokens`, `output_tokens`, `total_tokens`, `cache_read_tokens`, `cache_write_tokens`, `cost`, `cost_source`, `elapsed_ms`, `time_to_first_token_ms`, `is_compaction`. Line 837: the finish signal on the message is one flag, `output_token_limit_reached`.
- The crate names at this commit are `oi-kit-provider-types` and `oi-kit-providers`. The workspace report of 2026-11-17 read the same code under `goose-provider-types` and `goose-providers` at commit `5ac7ccb`.
- Source: H-s18. Basis: Filled (Goose).

**H69. Goose's switch rule compares the model name, and Goose retries without reasoning when the vendor rejects it.**

- `crates/oi-kit-provider-types/src/formats/anthropic.rs:146-159`, `thinking_block_is_stale`: the block is stale when `current_model != requested && current_model != resolved`. A message with no inference metadata is never stale.
- `crates/oi-kit-provider-types/src/formats/openai_responses.rs:65-78`: "The thinking signature slot also carries Anthropic signatures and reasoning imported from other accounts, which the API rejects, so only reasoning that `current_model` produced in this session is replayed." The encrypted content of a Responses reasoning item is stored in `ThinkingContentBlock.signature` (lines 48-62).
- Lines 91-102: `is_encrypted_reasoning_error` matches a message that contains "encrypted content" and "could not be verified", and "Rejected reasoning stays in the session, so callers retry once without any reasoning items rather than failing every later request."
- Line 779: `let store = model_config.request_param::<bool>("store").unwrap_or(false);`.
- Gemini, `crates/oi-kit-provider-types/src/formats/google.rs:23-37`: `THOUGHT_SIGNATURE_KEY = "thoughtSignature"` in the tool request's metadata map, and `SYNTHETIC_THOUGHT_SIGNATURE = "skip_thought_signature_validator"` for a call that has none (lines 147-153).
- Source: H-s18. Basis: Filled (Goose).

### Zed (`zed-industries/zed`, Rust)

**H70. Zed's content enum has thinking with an optional signature, redacted thinking, a thought signature on tool use, and one opaque JSON value per message.**

- `crates/language_model_core/src/request.rs:264-275`, `MessageContent`: `Text(String)`, `Thinking { text: String, signature: Option<String> }`, `RedactedThinking(String)`, `Image`, `ToolUse`, `ToolResult`, `Compaction(CompactedContext)`.
- `crates/language_model_core/src/language_model_core.rs:631-640`, `LanguageModelToolUse`: `thought_signature: Option<String>`, "Thought signature the model sent us. Some models require that this signature be preserved and sent back in conversation history for validation."
- `request.rs:357-367`, `LanguageModelRequestMessage`: `role`, `content: Vec<MessageContent>`, `cache: bool`, and `reasoning_details: Option<Arc<serde_json::Value>>`. The stream event is `ReasoningDetails(serde_json::Value)` (`language_model_core.rs:66`).
- `crates/open_ai/src/completion.rs:623-655`: for the Responses API the same `reasoning_details` value is parsed into reasoning items and replayed, with a map by item id so one item is sent once.
- `request.rs:277-300`: compaction state is the one tagged slot: `ProviderCompactionState { provider_id, format, payload }`, "Opaque context produced by a provider's native compaction mechanism."
- The thinking signature and the tool-use thought signature are bare strings. The message type holds a role, content, a cache flag, and `reasoning_details`, so the producing provider and model are tracked outside it, if at all.
- `crates/google_ai/src/completion.rs:37-50`: a `Thinking` block with a non-empty signature is sent to Gemini as `TextPart { text, thought: true, thought_signature: Some(signature) }`. A `Thinking` block with no signature, a `RedactedThinking` block, and a `Compaction` block are skipped. This converter does not check which vendor wrote the signature. Whether Zed's thread code filters blocks before this point was not read (the sparse checkout holds the model crates only).
- `crates/open_ai/src/completion.rs:97-126`, the chat-completions path: a `Thinking` block's text is sent as `reasoning_content` when the model takes interleaved reasoning, and otherwise as plain text content (`MessageContent::Text(text) | MessageContent::Thinking { text, .. }` share one arm). `RedactedThinking` is skipped. Line 475, the Responses path, skips `Thinking` and `RedactedThinking` and replays reasoning from `reasoning_details` instead.
- Source: H-s19. Basis: Filled (Zed).

**H71. Zed's stop reason and usage.**

- `language_model_core.rs:550-555`: `StopReason` is `EndTurn`, `MaxTokens`, `ToolUse`, `Refusal(Option<RefusalDetails>)`.
- Lines 564-573: `TokenUsage` has `input_tokens`, `output_tokens`, `cache_creation_input_tokens`, `cache_read_input_tokens`. These four fields are the whole type.
- Source: H-s19. Basis: Filled (Zed).

### Known failures reported outside the source trees

**H72. Turning another model's reasoning into tagged text made Claude print its reasoning to users, for more than 90 turns in one reported case.**

- Pydantic AI issue 5869, opened 2026-06-11 by a user, state closed, last updated 2026-11-17. The title says the Anthropic mapper re-renders unsigned or foreign `ThinkingPart`s as literal tagged text in the prompt, and ends "model mimics it and leaks reasoning into user-visible answers". (The page fetch stripped the tag text from the title, so the title is not quoted whole.)
- The report describes a `ThinkingPart` "**without a valid signature**", or one with a `provider_name` different from the serving model, such as one "produced by an OpenAI model in a `FallbackModel` chain", being sent to an Anthropic model as a plain text block wrapped in thinking tags. Then: "We hit this in production: 90+ user-visible turns where Claude printed its full reasoning".
- From the report, on why it persists: "If the application persists chat history, the leaked text is stored and replayed, making the contamination **self-sustaining for the whole thread**."
- The issue timeline lists the follow-up changes by title: PR 5920 "Drop unsigned/foreign `ThinkingPart`s from history by default across Anthropic, Bedrock, Groq, HuggingFace & xAI", PR 6692 "Carry non-native `ThinkingPart`s in a user turn for models that imitate assistant formatting", and PR 10392 "Send other providers' `ThinkingPart`s to `GoogleModel` as tagged text, not as Gemini `thought` parts".
- The code at the pinned commit matches PR 6692's title (H28). It does not match PR 10392's title: `models/google.py:2200-2203` still returns `{'text': item.content, 'thought': True}` for any thinking part with content (H30). I did not open the pull requests, so their merge state is not recorded.
- pi's code comment states the same risk as its reason to drop: "sending readable thinking as plain text makes the new model treat it as part of the visible reply" (H61).
- Source: H-s20. Basis: Filled (Pydantic AI). This is a third-party issue report. The production figure is the reporter's own.

**H73. An earlier workspace document recorded two harness failures on a model switch.**

- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, line 213: "OpenCode's `differentModel` guard strips signatures; Codex replays `encrypted_content` verbatim and fails on switch; Zed stored OpenAI reasoning as `RedactedThinking` and broke GPT-to-Claude switches until fixed."
- Lines 86-87 of the same document give the vendor error strings: OpenAI's `invalid_encrypted_content` "for cross-org or cross-provider blobs", and Gemini's "Function call `FC1` in the `1.` content block is missing a `thought_signature`".
- Today's code is consistent with the first and third statements. opencode has the `sameModel` and `sameProvider` checks (H65). Zed's OpenAI Responses path replays reasoning from the per-message `reasoning_details` value and skips `RedactedThinking` (H70). I did not re-check the Codex statement or find the Zed fix commit.
- Source: H-s22. Basis: Workspace (`runvil-design`, 2026-09-20). Not re-checked against a live issue tracker.

## Conflicts and disagreements

**Between the projects**

1. **Are Gemini API and Vertex thought signatures interchangeable?** LangChain says "Vertex and the Gemini Developer API share content and signature formats" and treats both as native (H40). The AI SDK reads a signature from the other namespace as a fallback, and its changelog calls losing the signature on a Vertex to AI Studio failover a bug it fixed (H15, H24). Pydantic AI says "the Gemini API and Google Cloud reject each other's" and "both reject each other's ... (verified live, 2026-10-27)", and it replaces the other transport's signature with the skip value (H30). These cannot all be right for the same models. I did not test it and did not find a Google statement in these sources. Pydantic AI's note is the only one that claims a dated live check.
2. **What to do with another model's reasoning.** pi, Goose, TensorZero, and LangChain drop it. rig and Pydantic AI turn it into text. The AI SDK's Google provider and Pydantic AI's Google model send it to Gemini as a thought with no signature, while the AI SDK's Anthropic and OpenAI providers drop it (H6, H15, H19, H28, H30, H37, H40, H44, H51, H61). pi's comment and Pydantic AI issue 5869 both report that plain-text conversion makes the next model imitate it (H61, H72). Pydantic AI's answer is to keep the text and move it to a user turn for models that copy formatting. rig's answer is plain text by default, with an opt-in to keep it as reasoning where the wire has an unsigned reasoning field.
3. **How narrow the tag is.** The AI SDK, LangChain, and TensorZero's thought block tag by provider. Pydantic AI tags by provider and endpoint. opencode and Goose tag by provider and model. pi and rig tag by wire, provider, and model (H2, H27, H35, H44, H50, H59, H64, H68). A provider-only tag sends one Claude model's signature to another Claude model. The model-tagged projects say that is invalid: pi, "Signatures and encrypted reasoning are only valid for the model that produced them", and opencode, "only the producing model may replay it". None of the provider-tagged projects' code says the opposite in words. The sources here do not settle which vendors accept a sibling model's signature.
4. **TensorZero's comment and code disagree on position.** The comment says the signature-only thought block is emitted "immediately following the original part" and merged "with their predecessor". The code emits it before the part and merges it with the next block (H46). The code is what runs.
5. **LangChain has two rules in one family of packages.** `langchain-anthropic` and `langchain-google-genai` check the message's `model_provider` before replaying reasoning. `langchain-openai`'s Responses input builder does not (H37, H38, H40).

**With the workspace documents**

6. **The 2026-11-17 report on Anthropic's Messages API, Appendix E.** Its one-line summaries still hold: rig "Raw provider JSON per block, same model only", genai "Text and signature paired by position", Vercel AI SDK "Signature in provider options", Pydantic AI "Signed part with a provider name", LangChain "Blocks kept in the message. Foreign reasoning dropped", opencode "Signature in part metadata", pi "Signature on the block, same provider and model only". Three details moved since its pinned commits. rig's sameness rule now also compares the wire (`Origin.api`) and can compare a fingerprint of tools and system prompt (H50, H51, H53). TensorZero has two vendor-state block types, not "One thought type": the second is `unknown`, a raw block tagged with model and provider (H43). Goose's crates were renamed to `oi-kit-provider-types` and `oi-kit-providers` (H68).
7. **The same report's low-confidence judgment about OpenAI.** It said: "It is likely that the same field can hold OpenAI's encrypted reasoning items later. My confidence in that judgment is low, because no subagent read the Responses API." This file adds evidence on that point. Five projects hold Responses reasoning items in the same per-reply ordered slot they use for Anthropic blocks: LangChain (`responses/v1` content blocks), rig (`native` items), TensorZero (`thought` and `unknown`), pi (a JSON string in the signature slot), and Pydantic AI (`ThinkingPart` with `id` and `signature`) (H31, H38, H47, H50, H60). The same sources show three rules that an Anthropic-only design does not need: a reasoning item is tied to the item after it (H54), item ids must be dropped or rewritten when the model changes (H62), and server-stored ids are a second kind of state with a different validity rule (H27).
8. **The same report's rule 2 for the field** says the translator "replays the blocks only when both [protocol and model] match" and cites Goose, pi, and rig. That matches pi and rig today, which compare wire, provider, and model (H51, H61). Goose compares the model name only (H69).
9. **The 2026-10-28 survey says "reasoning items now carry `encrypted_content` **by default**".** The SDKs read here still ask for it: the AI SDK adds `include: ['reasoning.encrypted_content']` when `store` is false, opencode sets it in the adapter's default options, Pydantic AI appends it, and genai offers it as an opt-in (H19, H31, H57, H66). This does not contradict the survey, since asking for a default is harmless. I did not re-fetch OpenAI's documentation, so I cannot say which is current.
10. **Local survey claim LC6** ("`reasoning_details` ... must replay unmodified and in order. Each entry has a vendor `format` tag such as `anthropic-claude-v1`. It is a replay contract, not a conversion contract."). The client code read here agrees on the `format` tag and on replay without conversion: Pydantic AI's codec lists `'unknown', 'openai-responses-v1', 'anthropic-claude-v1', 'xai-responses-v1', 'google-gemini-v1'` (H32), TensorZero stores `format` unread (H48), and rig says such signed data "never survives an edit" (H54). One client does modify the array before replay: LangChain's OpenRouter package merges stream fragments and removes ids that start with `rs_` (H42). That is client-side evidence only. It suggests "unmodified" has at least one practical exception in a shipping client, and it does not show what OpenRouter requires. The lead agent should check LC6 against the OpenRouter documentation evidence.
11. **The 2026-09-20 document's line on opencode** ("`differentModel` guard strips signatures") describes the same behavior under an older name. Today's code has two levels, `sameModel` and `sameProvider` (H65, H73).

## Open questions blocked by closed source

These are questions about OpenRouter's translator that came up in this topic. None can be answered from the projects read here. Each has a fill that shows how an open project solves the same problem.

1. **When a conversation's next request is routed to a different upstream than the one that produced its `reasoning_details`, what does OpenRouter do with them: drop, convert, forward, or fail?** Tried: client code for OpenRouter in Pydantic AI, LangChain, TensorZero, pi, and rig. All five tag the state with the name `openrouter` and send it back to OpenRouter whole, so none of them makes this decision. Fill found: yes. pi and rig (exact match on wire, provider, model, else drop or rebuild, H51, H61), opencode (model-scoped and provider-scoped levels, H65), Pydantic AI (canonical system, H27), TensorZero (provider type filter, H44).
2. **How does a per-message array hold state that the vendor attaches to one part, such as a Gemini thought signature on one function call?** Tried: the clients' codecs. Pydantic AI's has `index` and a `google-gemini-v1` format value, and pi has a legacy path that reads an encrypted reasoning detail from a tool call's `thoughtSignature` (H32, H60). Neither shows OpenRouter's mapping. Fill found: yes. Per-part metadata (AI SDK, opencode, H14, H66), a side map keyed by tool call id (LangChain, H39), a signature-only block adjacent to the part (TensorZero, genai, H46, H57), the raw part per block (rig, H50).
3. **Does OpenRouter keep the vendor's order of thinking, text, and tool calls within one reply?** Tried: the same client code. The chat-completions shape the clients parse has `content`, `tool_calls`, and `reasoning_details` as three separate fields. Fill found: yes. Ordered part lists in the AI SDK, Pydantic AI, rig, pi, opencode, and TensorZero keep order (H1, H25, H43, H50, H59, H64). LangChain's Gemini path shows the loss when calls are stored apart from the block list (H39).
4. **How are vendor-run tool calls, their encrypted results, and citations replayed through a chat-completions route?** Tried: not present in the client code read. Fill found: yes. Rebuild per tool (AI SDK, H8), raw payload in metadata (opencode, H66), raw unknown or opaque block (TensorZero, rig, H45, H50), two replay rules by where the state lives (Pydantic AI, H29).
5. **What does OpenRouter do with OpenAI reasoning item ids when it replays to OpenAI?** Tried: LangChain's OpenRouter client, which removes `rs_` ids before sending (H42). That hints that sending them back caused a failure, and it is not proof. Fill found: yes. The AI SDK's four branches by `store` and `previousResponseId` (H19), pi's rule to drop item ids for a different model (H62), rig's `needs_next` (H54).
6. **Which tag decides whether stored state is still valid: vendor, endpoint, model, or account?** Tried: the clients record `provider_details['downstream_provider']` (Pydantic AI) and nothing more specific on the state itself (H32). Fill found: partly. Pydantic AI separates data checked by key from ids looked up on a server (H27). rig adds the request context (H53). The projects disagree on whether the model is part of the tag (Conflicts, item 3).

## Unverified

- **Vendor rules.** The "native rule" cells in the matrix are as stated in the projects' code comments and in the workspace research. I did not fetch Anthropic's, Google's, or OpenAI's documentation for this file. Other evidence files in this series cover the vendors.
- **Behavior that was read and not run.** Nothing here was executed. Three statements rest on reading one code path and could be wrong if another layer filters first: that `langchain-openai` would send another vendor's reasoning block as a Responses reasoning item (H38), that genai sends an Anthropic signature as `encrypted_content` after a switch to a Responses model (H57), and that Zed's Gemini converter accepts any signed thinking block (H70). Zed's thread code and LangChain's agent layer were not read.
- **"Verified live" notes.** Phrases such as "verified live, 2026-10-27" (Pydantic AI) and "checked live on eight models" (rig) are the projects' own claims.
- **Pull request state.** The Pydantic AI pull requests named in issue 5869's timeline were not opened. The code at the pinned commit is the evidence for what is in the tree.
- **TensorZero's branch.** `main` at the read date is a commit dated 2026-06-04. I did not check whether development moved to another branch or repository.
- **Not read.** The AI SDK's UI message layer (how `providerMetadata` survives storage in an application), Pydantic AI's Bedrock and xAI models, LangChain's Bedrock and Vertex packages, rig's Gemini interactions wire beyond its blob slots, Goose's and Zed's agent layers. Goose and Zed were read for their types and replay rules only.
- **Counts of options** (21, 16, 31, 19, 9) are counts of the top-level keys I listed in H12, H17, H21, and H33 at the pinned commits.
- **From memory.** Nothing in this file is recorded from memory.

## Workspace documents used

- **"How agent harnesses handle Anthropic's Messages API, and what Runvil should build"** (cabinet report dated 2026-11-17, source key H-s21). Used: section 2 (the four options and the opaque block-list field), section 3 (the six rules for the field), section 4 (the six harness rules), and Appendix E (pinned commits and one-line summaries for 21 harnesses). I read it first and did not repeat its Anthropic-path findings. Re-checked: yes, against today's source for nine of its projects (rig, genai, TensorZero, pi, opencode, AI SDK, Pydantic AI, LangChain, Goose). Its summaries hold, with the three updates in Conflicts item 6.
- **`runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`** (2026-09-20, source key H-s22). Used: the replay table rows for OpenAI Responses and Gemini 3 (lines 86-87, 207) and the line on harness behavior at a model switch (line 213). Re-checked: the opencode and Zed statements against today's source (H65, H70, H73). The vendor error strings were not re-checked.
- **`runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`** (2026-10-28, source key H-s23). Used: its statements that OpenAI's `encrypted_content` is returned by default, that Gemini thought signatures are required with code `missing_thought_signature`, that Google's default documented surface is the Interactions API, and its recommendation text at line 371 ("Store the assistant's whole native block list as engine-owned data keyed by provider and model"). Re-checked: not against vendor documentation. Compared with SDK behavior in Conflicts item 9. Claim LC6 is discussed in Conflicts item 10.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| H-s1 | `vercel/ai` at `d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d` (commit date 2026-11-20): `packages/provider/src/language-model/v4/` and `packages/provider/src/shared/v4/` | 2026-11-22 | harness source |
| H-s2 | `vercel/ai` at `d55f3cf`: `packages/ai/src/generate-text/to-response-messages.ts` | 2026-11-22 | harness source |
| H-s3 | `vercel/ai` at `d55f3cf`: `packages/anthropic/src/` (`anthropic-language-model.ts`, `convert-to-anthropic-prompt.ts`, `anthropic-language-model-options.ts`, `anthropic-api.ts`, `get-cache-control.ts`, `map-anthropic-stop-reason.ts`, `convert-anthropic-usage.ts`, `forward-anthropic-container-id-from-last-step.ts`) | 2026-11-22 | harness source |
| H-s4 | `vercel/ai` at `d55f3cf`: `packages/google/src/` (`google-language-model.ts`, `convert-to-google-messages.ts`, `google-language-model-options.ts`, `map-google-finish-reason.ts`, `convert-google-usage.ts`) | 2026-11-22 | harness source |
| H-s5 | `vercel/ai` at `d55f3cf`: `packages/openai/src/responses/` (`openai-responses-language-model.ts`, `convert-to-openai-responses-input.ts`, `openai-responses-language-model-options.ts`, `openai-responses-provider-metadata.ts`, `map-openai-responses-finish-reason.ts`, `convert-openai-responses-usage.ts`) | 2026-11-22 | harness source |
| H-s6 | `vercel/ai` at `d55f3cf`: `packages/google/CHANGELOG.md`, `packages/anthropic/CHANGELOG.md`, `packages/openai/CHANGELOG.md` | 2026-11-22 | harness source |
| H-s7 | `pydantic/pydantic-ai` at `463a6d519b7e4c8e038d5a2559b0d26281df6937` (2026-11-22): `pydantic_ai_slim/pydantic_ai/` (`messages.py`, `_canonical_system.py`, `_thinking_part.py`, `settings.py`, `usage.py`, `models/__init__.py`, `models/anthropic/__init__.py`, `models/google.py`, `models/openai/_reasoning.py`, `models/openai/_responses_model.py`, `models/openai/_settings.py`, `models/_reasoning_details.py`, `models/openrouter.py`) | 2026-11-22 | harness source |
| H-s8 | `langchain-ai/langchain` at `8ff3cd30eebfe37712a91f11e3f4b61a545e7a37` (2026-11-20): `libs/core/langchain_core/messages/` (`content.py`, `ai.py`, `block_translators/__init__.py`, `block_translators/anthropic.py`) | 2026-11-22 | harness source |
| H-s9 | `langchain-ai/langchain` at `8ff3cd3`: `libs/partners/anthropic/langchain_anthropic/` (`chat_models.py`, `_compat.py`) | 2026-11-22 | harness source |
| H-s10 | `langchain-ai/langchain` at `8ff3cd3`: `libs/partners/openai/langchain_openai/chat_models/` (`base.py`, `_compat.py`) | 2026-11-22 | harness source |
| H-s11 | `langchain-ai/langchain-google` at `227abc8efd3c357407556002b164f89f893a8c09` (2026-11-19): `libs/genai/langchain_google_genai/` (`chat_models.py`, `_compat.py`) | 2026-11-22 | harness source |
| H-s12 | `langchain-ai/langchain` at `8ff3cd3`: `libs/partners/openrouter/langchain_openrouter/chat_models.py` | 2026-11-22 | harness source |
| H-s13 | `tensorzero/tensorzero` at `62eb8f63e8ec62018d70420dbf1a8c5d1c026315` (2026-06-04): `crates/tensorzero-types/src/` (`content.rs`, `usage.rs`), `crates/tensorzero-core/src/model.rs`, `crates/tensorzero-core/src/providers/` (`anthropic.rs`, `google_ai_studio_gemini.rs`, `gcp_vertex_gemini/mod.rs`, `openai/responses.rs`, `openrouter.rs`), `docs/gateway/api-reference/inference.mdx` | 2026-11-22 | gateway source |
| H-s14 | `0xPlaygrounds/rig` at `a8673f775ca78004f011e5ad5b615147fdcd4bff` (2026-11-22): `crates/rig-core/src/completion/` (`message.rs`, `message/native.rs`, `history.rs`, `options.rs`, `options/mapping.rs`, `request.rs`), `crates/rig-core/src/providers/` (`anthropic/wire.rs`, `gemini/completion.rs`, `openai/responses_api/wire.rs`, `openai/wire/chat.rs`) | 2026-11-22 | harness source |
| H-s15 | `jeremychone/rust-genai` at `2a8a5b39c96028d2143d8754ee2bba42a79e32c5` (2026-11-18): `src/chat/` (`content_part/common.rs`, `content_part/custom_part.rs`, `tool/tool_call.rs`, `chat_options.rs`, `chat_message.rs`, `chat_response.rs`, `usage.rs`), `src/adapter/adapters/` (`anthropic/adapter_shared.rs`, `gemini/adapter_impl.rs`, `openai_resp/adapter_impl.rs`) | 2026-11-22 | harness source |
| H-s16 | `earendil-works/pi` at `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (2026-11-22): `packages/ai/src/types.ts`, `packages/ai/src/api/` (`transform-messages.ts`, `anthropic-messages.ts`, `openai-responses-shared.ts`, `openai-completions.ts`, `google-shared.ts`) | 2026-11-22 | harness source |
| H-s17 | `anomalyco/opencode` at `71cd8c064c8b93c31604c4c4074adb8b45790fc1` (2026-11-22): `packages/ai/src/schema/` (`messages.ts`, `options.ts`, `events.ts`), `packages/ai/src/protocols/` (`anthropic-messages.ts`, `gemini.ts`, `open-responses.ts`, `openai-responses.ts`), `packages/schema/src/` (`session-message.ts`, `llm.ts`), `packages/core/src/session/runner/to-llm-message.ts` | 2026-11-22 | harness source |
| H-s18 | `block/goose` at `36b29113aff691e1a46a21a5c25900824caf9949` (2026-11-20): `crates/oi-kit-provider-types/src/conversation/message.rs`, `crates/oi-kit-provider-types/src/formats/` (`anthropic.rs`, `google.rs`, `openai_responses.rs`) | 2026-11-22 | harness source |
| H-s19 | `zed-industries/zed` at `d31d1074a07b7bbaa43a52fef7847a00dd491baf` (2026-11-22): `crates/language_model_core/src/` (`request.rs`, `language_model_core.rs`), `crates/google_ai/src/completion.rs`, `crates/open_ai/src/completion.rs` | 2026-11-22 | harness source |
| H-s20 | https://github.com/pydantic/pydantic-ai/issues/5869 | 2026-11-22 | issue report (third-party) |
| H-s21 | Cabinet report "How agent harnesses handle Anthropic's Messages API, and what Runvil should build", dated 2026-11-17 | 2026-11-22 | workspace document |
| H-s22 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, dated 2026-09-20 | 2026-11-22 | workspace document |
| H-s23 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, dated 2026-10-28 | 2026-11-22 | workspace document |
