---
produced: 2026-11-22
title: OpenRouter client tools (function calling) and structured output - tool translation per provider, tool_choice support flags, parallel_tool_calls, tool-call ids, strict schemas, streaming tool arguments, response_format json_schema and json_object, response healing, refusals, message-shape rules, Auto Exacto, with vendor rules (Anthropic, Gemini, Mistral, DeepSeek, Meta, Moonshot, OpenAI) and open-translator fills (LiteLLM, agentgateway)
---

# OpenRouter client tools (function calling) and structured output

- **What was read:** OpenRouter's tool calling, structured output, response healing, Exacto, provider integration, parameters, compatibility, routing, error, streaming, and model migration pages, the OpenAPI spec, the model catalog, the per-endpoint records of 19 models, and the official AI SDK provider source. Then the live tool and structured-output documentation of Anthropic, Google, Mistral, DeepSeek, Meta, Moonshot, and OpenAI, and two open translators (LiteLLM, agentgateway) for the points OpenRouter leaves unsaid.
- **How much:** 52 sources, 67 evidence items (T1 to T67), 28 candidate matrix rows.
- **Read date:** 2026-11-22 for every source. Nothing was run against a live model, and every read was public and unauthenticated.

## Candidate matrix rows

Basis labels: `OR-documented`, `OR-inferred`, `Filled` (project named), `Vendor-documented`, `Workspace`. Where a row draws on more than one kind of source, the label for the OpenRouter column comes first.

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Function tool definitions | All | OpenAI `tools[].function`; Anthropic `tools[]` with `input_schema`; Gemini function declarations | `tools` in OpenAI shape. "passed down as-is" for OpenAI-style providers, "transform and map" for custom interfaces, "YAML template" otherwise | The response does not say which path ran. The third path rests on one code comment and conflicts with "the parameter is ignored" | Gateway holds the translation. Stateless. Neutral values. Chat route. Tools must be resent on every request | OR-documented | T1, T2, T3, T5, T67 |
| Tool capability discovery | All providers | n/a | `supported_parameters` per model and per endpoint, set by OpenRouter's capability tests ("never enabled from the feed") | The model-level list behaves as a union across endpoints (`z-ai/glm-5.3` lists `parallel_tool_calls`, and 2 of its 46 endpoints do), so one model has endpoints with and without `tools` or `structured_outputs` | Gateway or caller reads the per-endpoint record to know. No state | OR-documented | T4, T6, T9, T33 |
| `tool_choice` vocabulary | OpenAI, Anthropic, Google, Mistral | OpenAI `auto`/`none`/`required`/named/`allowed_tools`; Anthropic `auto`/`any`/`tool`/`none`; Gemini `auto`/`any`/`none`/`validated`; Mistral `auto`/`any`/`none` | Chat: `"none"`, `"auto"`, `"required"`, `{"type":"function","function":{"name"}}`, plus server-tool form. Per-endpoint `supports_tool_choice` `{none, auto, required, function}` | Gemini `validated` stays on Google's own API and OpenAI `allowed_tools` stays on the Responses route. Value support differs per endpoint of the same model | Neutral words translate one to one. Whether a value works depends on model and endpoint, so the gateway needs the flags or a model table | OR-documented, Vendor-documented | T7, T8, T9, T48, T52, T54, T58 |
| Forced tool choice rejected | Anthropic (Fable 5.2, Mythos 5.2, Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1, and manual extended thinking) | 400 `tool_choice: type "tool" and "any" are not supported for this model.` | Forwarded, and "the provider's 400" comes back. Flags show `required: false, function: false` on every endpoint. Advice: `auto` plus a prompt instruction plus `strict`, or structured outputs | No downgrade and no neutral "must call a tool" that works on every model | Gateway can refuse early or downgrade from a model table. No state. Caller's value stays neutral | OR-documented, Vendor-documented | T10, T48, T60 |
| Forced tool choice in thinking mode | DeepSeek | "`required` and named tool choices are not supported in thinking mode; the API returns a `400` error"; with web tools "`required` is treated as `auto`" | Not in OpenRouter prose. Flags: `deepseek/deepseek-v4-pro` is `none, auto` only on 9 of 13 endpoints | Depends on the thinking switch, which is a separate request setting | Gateway must combine thinking state and `tool_choice`. No state | Vendor-documented, OR-documented (flags) | T9, T55 |
| `tool_choice` limited to `auto` | Meta model API | Only `"auto"`; others return HTTP 400 | Meta-family models on OpenRouter run on third-party hosts whose flags vary per endpoint | n/a | Per-endpoint fact. Gateway holds it | Vendor-documented, OR-documented (flags) | T56, T9 |
| Parallel tool call switch | OpenAI, Anthropic, Mistral | OpenAI and Mistral `parallel_tool_calls`; Anthropic `tool_choice.disable_parallel_tool_use` (inverted) | `parallel_tool_calls` boolean on chat and Responses, default true. Listed as supported on 18 of 476 models. Messages route keeps Anthropic's flag | Translation to Anthropic's flag is undocumented. The catalog flag is too sparse to route on | Neutral boolean. Gateway inverts it for Anthropic | OR-documented, Filled (LiteLLM, agentgateway) | T13, T14, T60 |
| Tool-call id format | OpenAI, Anthropic, Bedrock, Mistral, Meta, Moonshot, Google | OpenAI `call_...`; Anthropic `^[a-zA-Z0-9_-]+$`; Bedrock at most 64 characters; Mistral `^[a-zA-Z0-9]{9}$`; Meta 1 to 64 characters; Kimi K3 opaque (older `name:idx`); Gemini optional ids | Plain string with no documented rule. The vendor's own id reaches the caller on every route (`call_...`, `toolu_bdrk_...`). Gemini ids are minted as `tool_<name>_<random>` | No documented rewrite when history crosses vendors. Third-party captures show Mistral's error 3280 relayed through OpenRouter | Caller round-trips the id unchanged. Cross-vendor replay needs a gateway rewrite applied to the call and its result alike. A deterministic rewrite needs no stored state | OR-inferred, Vendor-documented, Filled (LiteLLM, agentgateway) | T15, T16, T17, T50, T54, T56, T57, T59, T62, T64 |
| Empty or duplicate ids from providers | Some providers | n/a | The official SDK mints a new id in the client when an id is empty or repeated | The server does not guarantee unique, non-empty ids | Gateway should guarantee them before the client sees the reply | OR-documented (SDK source) | T18 |
| Tool result pairing and order | Anthropic, OpenAI, Meta, Moonshot | Anthropic: all results in the next user message, first, with nothing between. Meta: one `tool` message per call, call order when history is manual. Kimi: every call answered, order free | Chat: `role: "tool"` with `tool_call_id`. Responses: `function_call_output` with `call_id`. "Append every output item in order" | Regrouping and repair rules are undocumented. The debug echo is the only window, and it is streaming only | Caller sends one tool message per call right after the assistant message. Gateway regroups for Anthropic | OR-documented, Vendor-documented, Workspace | T19, T39, T49, T56, T57, T66 |
| Tool schema subset and `strict` | OpenAI, Anthropic, Google, DeepSeek, Meta | OpenAI strict: `additionalProperties: false` and all fields required. Anthropic strict subset, 20 strict tools, 24 optional parameters. Gemini "subset of the OpenAPI schema". DeepSeek strict on a beta base URL. Meta strict is validation only | `function.strict` (`boolean \| null`). Schema is an unchecked JSON object. For Anthropic, `strict` is stripped unless `x-anthropic-beta: structured-outputs-2025-11-13` is sent | No documented sanitizing. "See the provider's documentation" | Caller writes a portable schema or the gateway sanitizes per vendor. Neutral flag. No state | OR-documented, OR-inferred, Vendor-documented, Filled (agentgateway, LiteLLM, Anthropic SDK) | T20, T21, T22, T51, T52, T55, T56, T58, T62, T63 |
| Tool-call argument validation | n/a | n/a | Draft 7 validation after the fact, for the Tool Call Error Rate (`InvalidJson`, `UnknownName`, `MissingName`, `SchemaMismatch`, `UnparsedToolCall`) | Bad arguments still reach the caller. Raw tool markup in text reaches the caller | Caller or gateway validates arguments | OR-documented | T23 |
| Streaming of tool arguments | OpenAI, Anthropic, Google | OpenAI incremental. Anthropic buffered unless `eager_input_streaming`. Gemini whole call per chunk (Interactions API can stream partial arguments) | `delta.tool_calls[]` with `index`, `id`, `function.name`, `function.arguments`. OpenRouter sets `eager_input_streaming: true` for Anthropic | Some providers resend full arguments, omit `index`, or close the JSON early | Client accumulates per call and finalizes only at end of stream | OR-documented, OR-inferred | T24, T25, T26, T27, T18 |
| Tool call cut off by the token limit | All | Anthropic `stop_reason: "max_tokens"` with an open block | `finish_reason: "length"` (chat), `response.incomplete` (Responses). Scored as `truncated-arguments` | What the caller receives is undocumented. The official SDK drops unparseable calls in the client | Client must not run a call when the finish reason is `length` | OR-documented, OR-inferred | T28 |
| Finish reason with tool calls | Google | Gemini reports `STOP` with function calls | Normalized `finish_reason: "tool_calls"` plus `native_finish_reason`. The official SDK still overrides `stop` and unknown values when tool calls are present | Normalization has gaps the SDK patches | Client treats the presence of tool calls as the signal | OR-inferred, OR-documented (SDK source) | T17, T18 |
| JSON mode | OpenAI, DeepSeek | OpenAI `json_object` needs the word "JSON" in context. DeepSeek offers `text` and `json_object` only | `response_format: {"type": "json_object"}` | n/a | Neutral value. Gateway only | OR-documented, Vendor-documented | T30, T55, T58 |
| JSON Schema output | OpenAI, Anthropic, Google, Moonshot | OpenAI `json_schema` strict. Anthropic `output_config.format`. Gemini `response_format` with `schema`. Kimi `json_schema` strict | Chat `response_format.json_schema`, Responses `text.format`. Capability flag `structured_outputs` per endpoint. `provider.require_parameters: true` to pin | Enforcement "varies by provider". Unsupported model: "request will fail" in one page, "parameter is ignored" in another | Gateway translates. Caller must validate the result. Support is a per-endpoint fact | OR-documented, Vendor-documented | T31, T32, T33, T34, T51, T53, T57 |
| JSON repair after generation | OpenRouter only | n/a | `plugins: [{"id": "response-healing"}]`, non-streaming only, syntax repair of the message text | Does nothing on streams. Does not check the schema | Runvil always streams, so this plugin never applies. A gateway-side repair would need the full body | OR-documented | T35 |
| Refusals | OpenAI, Anthropic | OpenAI `refusal` field. Anthropic `stop_reason: "refusal"` | `message.refusal` with `finish_reason: "content_filter"`. Policy blocks are HTTP 403 with `error_type: "refusal"` | Anthropic's `stop_details` category stays on the Messages route | Neutral field. No state | OR-documented | T36 |
| Freeform tools and grammars | OpenAI, Meta | Custom tools with `lark` or `regex` grammars, Responses only | Responses `tools[].type: "custom"` and `custom_tool_call` items. Chat has `response_format` types `grammar` and `python` whose supporting providers are unlisted | Custom tools exist on the Responses route alone | Exists only off the chat route | OR-documented, Vendor-documented | T29, T37, T56, T58 |
| System and developer roles, message merging | Anthropic and others | Anthropic takes a top-level `system` and combines consecutive same-role turns | Roles `system`, `developer`, `user`, `assistant`, `tool`. "system messages are concatenated ... user messages are merged". `name` becomes `{name}: {content}` for non-OpenAI models | The merge rules are unpublished | Gateway holds the rules. Neutral input | OR-documented | T38, T39, T40 |
| Assistant prefill | Anthropic, DeepSeek, Moonshot | Newer Claude models reject a final assistant turn. DeepSeek `prefix: true` (beta URL). Kimi `partial: true` | A final `role: "assistant"` message. The provider's 400 is returned where rejected | Vendor flags `prefix` and `partial` are unmapped in the docs | Gateway could map the flags from a model table. No state | OR-documented, Vendor-documented | T41, T65 |
| Mid-conversation system messages and tool changes | Anthropic, Moonshot, DeepSeek | Anthropic `clear_at` and tool-change blocks. Kimi tool definitions in a content-less system message. DeepSeek accepts mid-conversation system messages on chat | Messages route only. Chat gets the `configuration_update` extension on a content-less system message | "the Chat Completions API has no representation for these blocks" | Exists only off the chat route, or needs an extension field | OR-documented, Vendor-documented | T42, T55, T57 |
| Images in tool results | Anthropic, Google, OpenAI | Anthropic `tool_result` content blocks (`text`, `image`, `document`). Gemini 3 multimodal function responses | Chat spec: tool `content` is a string or content parts. Responses: `function_call_output.output` array. "only forwarded to supported multimodal models" | What a text-only model gets is unpublished | Gateway decides per model. No state | OR-documented, Vendor-documented | T38, T43, T49, T52 |
| `phase` on assistant output | OpenAI (GPT-5.3 Codex to GPT-5.5) | Must be replayed on assistant items | Responses route only | "not available in the Chat Completions API" | Round trip through the caller, off the chat route | OR-documented | T45 |
| Async tool calls | OpenAI (GPT-6) | `async: true` on a tool, result delivered later | Responses route only. On chat "the flag is removed before the request is routed" | Chat and Messages run every call synchronously | Exists only off the chat route | OR-documented | T45 |
| Provider variance for one model | Hosts of open-weight models | n/a | Auto Exacto reorders providers on every tool request by Tool Call Error Rate, throughput, and benchmarks. `:exacto` to ask for it, `provider.sort: "price"` or `:floor` to opt out | Reordering can break sticky routing and cost cache hits | Routing concern inside the gateway. The caller's request stays vendor-neutral, and the result quality depends on the host | OR-documented | T46 |
| Thought signature tied to a tool call | Google (Gemini 3) | A function call's signature must come back | `reasoning_details` entry (`reasoning.encrypted`, `google-gemini-v1`) with the same id as the tool call, replayed by the caller | Missing replay returns a 400 that quotes the upstream error | Opaque data makes a round trip through the caller, next to the tool call | OR-inferred, Filled (agentgateway, LiteLLM put it inside the id) | T17, T18, T62 |

## Evidence

Every OpenRouter documentation page below was read from the 2026-11-22 mirror of `https://openrouter.ai/docs`. The spec is `https://openrouter.ai/openapi.json` and the catalog is `https://openrouter.ai/api/v1/models`, both fetched 2026-11-22. Source keys (`T-s1` and so on) are listed in the Sources table at the end.

### A. Tool calling translation per provider (LC5)

**T1. The three-implementation sentence is in today's docs, as a comment in the Chat Completions request type on the API Reference overview page.**

- Quote: "// Tool calling / // Will be passed down as-is for providers implementing OpenAI's interface. / // For providers with custom interfaces, we transform and map the properties. / // Otherwise, we transform the tools into a YAML template. The model responds with an assistant message. / // See models supporting tool calling: openrouter.ai/models?supported_parameters=tools / tools?: Tool[]; / tool_choice?: ToolChoice;" (the slashes mark line breaks in the code block)
- Source: T-s10. Basis: `OR-documented`.
- Note: this is the only place in the 559 mirrored pages where the words "YAML template" appear. A search of the whole mirror for `YAML template`, `passed down as-is`, `custom interfaces`, and `transform the tools` hits this page only.

**T2. The Parameters page states the second implementation (transformation) and nothing about the third.**

- Quote: "Tool calling parameter, following OpenAI's tool calling request shape. For non-OpenAI providers, it will be transformed accordingly. Learn more in the [tool calling guide]"
- Source: T-s8. Basis: `OR-documented`.

**T3. The Client Tools guide describes one standardized interface and never mentions a prompt template, emulation, or YAML.**

- Quote: "OpenRouter standardizes the tool calling interface across models and providers, making it easy to integrate external tools with any supported model."
- Quote: "**Supported Models**: You can find models that support tool calling by filtering on [openrouter.ai/models?supported_parameters=tools]"
- Quote: "**Note**: On both APIs, the `tools` parameter must be included in every request (Steps 1 and 3) so the router can validate the tool schema on each call."
- Source: T-s1. Basis: `OR-documented`.

**T4. Whether an endpoint counts as supporting `tools`, `tool_choice`, `response_format`, or `structured_outputs` is decided by OpenRouter's own capability tests, never by the provider's declaration.**

- Quote: "parameters OpenRouter verifies by test (`tools`, `tool_choice`, `response_format`, `structured_outputs`, `logprobs`, and the rest of the capability suite) are never enabled from the feed, only by the capability tests passing."
- Quote: "The legacy `supported_features` list (`tools`, `json_mode`, `structured_outputs`, `logprobs`, and so on) is parsed and ignored: OpenRouter discovers testable capabilities by running capability tests against the endpoint, so declaring a feature there does not enable it."
- Quote (provider API): "`POST /api/v1/providers/{provider_slug}/endpoints/{endpoint_id}/capabilities/disable` removes capabilities from an endpoint ... and excludes them so automated capability tests do not re-enable them."
- Source: T-s7. Basis: `OR-documented`.

**T5. The third implementation (YAML template) sits next to routing text that says the opposite for a model with no tool-capable provider: the parameter is ignored.**

- Quote (provider selection, "Default parameter preferences"): "Even when `require_parameters` is `false`, a small set of parameters is used as a soft preference when choosing between providers of the same model: `tools`, the response format (`text.format` on the Responses API, `response_format` on Chat Completions, including structured outputs), and `verbosity`. If some of a model's providers support one of these parameters and others don't, the request is only routed to the supporting providers. If none of a model's providers support the parameter, the request is still routed to that model and the parameter is ignored"
- Quote (API overview, "Non-standard parameters"): "If the chosen model doesn't support a request parameter (such as `logit_bias` in non-OpenAI models, or `top_k` for OpenAI), then the parameter is ignored. The rest are forwarded to the underlying model API."
- Sources: T-s14, T-s10. Basis: `OR-documented`.
- Confidence note: the two statements conflict (see "Conflicts and disagreements"). The docs give no example of a YAML-rendered tool prompt, no changelog entry about it, and no field that tells a caller which of the three paths served a request. The catalog lists 62 of 476 models without `tools` in `supported_parameters` (T-s12), which is the population the third path would apply to.

**T6. The catalog counts for the tool and structured-output parameters (476 models, `supported_parameters` per model, fetched 2026-11-22).**

- `tools`: 414. `tool_choice`: 409. `parallel_tool_calls`: 18. `structured_outputs`: 407. `response_format`: 419.
- `tools` without `tool_choice`: 5 models (`recursive/flint-v1.0:free`, `amazon/nova-premier-v1`, `amazon/nova-lite-v1`, `amazon/nova-micro-v1`, `amazon/nova-pro-v1`). `tool_choice` without `tools`: 0.
- `response_format` without `structured_outputs`: 28. `structured_outputs` without `response_format`: 16. `tools` without `structured_outputs`: 42 (among them `anthropic/claude-opus-4.1`, `moonshotai/kimi-k2`, `z-ai/glm-4.5`, `openai/gpt-4`, `minimax/minimax-m2.7`, the Amazon Nova family).
- No `tools`: 62 models (image, audio, translation, safety, and roleplay models, plus `microsoft/phi-4`, `google/gemma-3-4b-it`, `nousresearch/hermes-4-405b`, `perplexity/sonar*`, `mistralai/pixtral-large-2502`).
- `strict_tool_schemas` is a value of the spec's `Parameter` enum (T-s11) and appears on 0 of 476 models and on 0 of the 194 endpoints sampled in T9.
- Source: T-s12 (computed with a script over the saved catalog). Basis: `OR-documented`.

### B. `tool_choice`

**T7. Accepted values on each of the three routes.**

- Chat Completions, Parameters page: "Controls which (if any) tool is called by the model. 'none' means the model will not call any tool and instead generates a message. 'auto' means the model can pick between generating a message or calling one or more tools. 'required' means the model must call one or more tools. Specifying a particular tool via `{"type": "function", "function": {"name": "my_function"}}` forces the model to call that tool." (T-s8)
- Chat Completions, spec `ChatToolChoice`: `anyOf` of the strings `"none"`, `"auto"`, `"required"`, `ChatNamedToolChoice` (`{"type": "function", "function": {"name"}}`), and `ChatServerToolChoice`, described as "OpenRouter extension: force a specific server tool by naming it directly in `tool_choice.type` instead of wrapping it in `{ type: \"function\", function: { name } }`." (T-s11)
- Chat Completions, API overview TypeScript type: `type ToolChoice = | 'none' | 'auto' | { type: 'function'; function: { name: string; }; };` This type leaves out `'required'`, which the Parameters page and the spec both list. (T-s10)
- Responses, spec `OpenAIResponsesToolChoice`: `"auto"`, `"none"`, `"required"`, `{"type": "function", "name"}`, `{"type": "web_search_preview" | "web_search_preview_2025_03_11"}`, `ToolChoiceAllowed` (`{"type": "allowed_tools", "mode": "auto" | "required", "tools": [...]}`), `{"type": "apply_patch"}`, `{"type": "shell"}`, and `ToolChoiceMcp` (`{"type": "mcp", "server_label", "name"}`). (T-s11)
- Messages, spec `MessagesRequest.tool_choice`: `{"type": "auto"}`, `{"type": "any"}`, `{"type": "none"}`, `{"type": "tool", "name"}`. The `auto`, `any`, and `tool` forms each take `disable_parallel_tool_use: boolean`. (T-s11)
- Basis: `OR-documented`.

**T8. `tool_choice` in `supported_parameters` only means the parameter is accepted. Which values work is a separate per-endpoint record, `supports_tool_choice`, filled in by testing.**

- Quote (spec `ToolChoiceSupport`): "Per-variant `tool_choice` support. `tool_choice` in `supported_parameters` only says the parameter is accepted; these flags say which of its values passed testing."
- Fields, all required booleans: `none` ("Whether the endpoint supports `\"tool_choice\": \"none\"`."), `auto`, `required`, `function` ("Whether the endpoint supports naming a function, `\"tool_choice\": {\"type\": \"function\", ...}`.").
- Where it lives: `PublicEndpoint.supports_tool_choice`, a required property, returned by `GET /api/v1/models/{author}/{slug}/endpoints`. The model-level catalog entry (`/api/v1/models`) returns `supported_parameters` alone.
- Changelog: "**August 28, 2026** ... `PublicEndpoint`: required `supports_tool_choice` property added" and new schema `ToolChoiceSupport`.
- Sources: T-s11, T-s23. Basis: `OR-documented`.

**T9. Live `supports_tool_choice` values, read without credentials on 2026-11-22 for 19 models (194 endpoints). The same model gets different flags on different providers.**

Each cell is the count of endpoints with that flag pattern. Pattern letters: N = `none`, A = `auto`, R = `required`, F = `function`, and a dot means the flag is `false`.

| Model | Endpoints | Flag patterns |
|---|---|---|
| `anthropic/claude-fable-5.2` | 4 | `NA..` 4 (Anthropic, Anthropic fast, Azure, Google Vertex) |
| `anthropic/claude-opus-5.5` | 6 | `NA..` 6 (Anthropic, Anthropic fast, Claude Platform on AWS, Azure, Google Vertex, Amazon Bedrock) |
| `anthropic/claude-sonnet-5.5` | 5 | `NA..` 5 |
| `anthropic/claude-sonnet-4.5` | 5 | `NARF` 5 |
| `openai/gpt-5.5` | 4 | `NARF` 4 |
| `openai/gpt-6.2-sol` | 5 | `NARF` 5 |
| `google/gemini-3.5-flash` | 6 | `NARF` 6 (Vertex and AI Studio) |
| `x-ai/grok-4.7` | 5 | `NARF` 5 |
| `mistralai/mistral-large-2512` | 2 | `.ARF` 2 (Mistral: `none` is false) |
| `meta-llama/llama-4-maverick` | 1 | `.ARF` 1 (Parasail) |
| `meta-llama/llama-3.3-70b-instruct` | 7 | `NARF` 5, `NA..` 2 (DeepInfra turbo, Amazon Bedrock) |
| `deepseek/deepseek-v4-pro` | 13 | `NA..` 9, `.A..` 3 (StreamLake, Relace, DigitalOcean), `NARF` 1 (Azure) |
| `deepseek/deepseek-v4.1-flash` | 37 | `NARF` 30, `NA.F` 2, `.A..` 2, `NA..` 1, `.ARF` 1, `....` 1 |
| `moonshotai/kimi-k3` | 29 | `NARF` 22, `N.RF` 2, `NAR.` 2, `NA..` 1, `.ARF` 1, `....` 1 |
| `moonshotai/kimi-k2.6` | 15 | `NARF` 9, `NA.F` 2, `.A..` 2, `NA..` 1, `.ARF` 1 |
| `z-ai/glm-5.3` | 46 | `NARF` 27, `NA.F` 9, `NA..` 8, `.A..` 1 (Z.AI itself), `NAR.` 1 (Amazon Bedrock) |
| `qwen/qwen3.7-flash` | 1 | `NA..` 1 (Alibaba) |
| `minimax/minimax-m2.7` | 1 | `.A..` 1 (Groq) |
| `amazon/nova-pro-v1` | 2 | `NARF` 2, while `supported_parameters` on both endpoints lists `tools` and leaves out `tool_choice` |

- Source: T-s13. Basis: `OR-documented` (public API data).
- Confidence note: two rows look inconsistent with their own `supported_parameters` (Amazon Nova lists all four flags true with no `tool_choice` parameter, and two `meta-llama/llama-3.3-70b-instruct` endpoints list all four flags true with no `tools` parameter). The flags may default to `true` where no test ran. That reading is an inference.

**T10. A forced tool choice on a model that rejects it is passed to the provider and comes back as the provider's 400. OpenRouter's docs describe no downgrade to `auto`.**

- Quote (Claude 5.5 guide, Opus 5.5): "Opus 5.5 rejects `tool_choice` set to `{"type": "any"}` or a named tool, and the Chat Completions forms `tool_choice: "required"` and `{"type": "function", "function": {"name": ...}}`, with the provider's 400 (`tool_choice: type "tool" and "any" are not supported for this model.`). `{"type": "auto"}` (the default) and `{"type": "none"}` are unaffected."
- Quote (Sonnet 5.5): "Sonnet 5.5 rejects forced tool choice. Chat Completions `tool_choice: "required"` or a named function, and Messages API `{"type": "any"}` or `{"type": "tool"}`, return the provider's 400 (`tool_choice: type "tool" and "any" are not supported for this model.`). Send `auto`, say in the prompt when to call the tool, and mark the tool `strict: true` so its input matches the schema."
- Quote (breaking-changes table): "Forced `tool_choice` (`required`, named, `any`, `tool`) | Accepted | 400 upstream"
- Quote (Fable 5.1 guide): "Starting with Fable 5.1, requests with `tool_choice` set to `{"type": "any"}` or a named tool return a 400. `{"type": "auto"}` (the default) and `{"type": "none"}` are unaffected." and table row "`tool_choice: {"type": "any"}` or named tool | Allowed | Rejected upstream (400)"
- Quote (Haiku 5.5): "Forced tool use still works on Haiku 5.5 (`tool_choice: "required"`, `{"type": "any"}`, or a named tool), but the model skips thinking before a forced call."
- Quote (the workaround OpenRouter recommends): "use `tool_choice: {"type": "auto"}` and state the expectation in the prompt ... Because `auto` does not guarantee a call, check that one was made and retry if not." and "if you were forcing a tool call to get JSON back, use [structured outputs] instead"
- Sources: T-s15, T-s16. Basis: `OR-documented`.
- Cross-check: this matches the `NA..` flags on every Claude Fable 5.2, Opus 5.5, and Sonnet 5.5 endpoint in T9.

**T11. Routing treats `tools` as a soft filter by default and as a hard filter with `require_parameters`. The docs name the parameter, never the individual `tool_choice` values.**

- Quote: "When you send a request with `tools` or `tool_choice`, OpenRouter makes a best effort to route to providers known to support tool use."
- Quote: "With the default routing strategy, providers that don't support all the [LLM parameters] specified in your request can still receive the request, but will ignore unknown parameters. When you set `require_parameters` to `true`, the request won't even be routed to that provider."
- Quote (compatibility page): "Setting `provider: { require_parameters: true }` ensures OpenRouter never routes requests with `json_schema` or `tools` to backend providers that don't support those parameters: if no endpoint supports them, the request fails instead of falling back to one that would drop them."
- Sources: T-s14, T-s9. Basis: `OR-documented`.
- Confidence note: no page says whether `require_parameters` or default routing reads the `supports_tool_choice` flags, so whether `tool_choice: "required"` avoids an endpoint whose `required` flag is `false` is undocumented. See "Open questions".

**T12. Two places where OpenRouter itself rejects or rewrites a `tool_choice`.**

- Tool search: "Any other `tool_choice` conflicts with deferral ... Rather than silently overriding what you asked for, the request fails with a `400`: `tool_choice` conflicts with `openrouter:tool_search`. Deferred tools are revealed through `tool_choice`, so it must be omitted or set to `{"type": "allowed_tools", ...}`." (T-s25)
- Broadcast metadata describes a normalized, applied choice: "`tool_choice` is the tool choice the router applies: `auto`, `none`, or `required`, or the choice's type (such as `function`) when the request names a tool. For the Responses API, an `allowed_tools` choice is reported as its `mode`. A Messages request that defines tools without a `tool_choice` reports `auto`, the default the router applies to it. `tool_count` is the number of tools the router passes on after normalizing the request" (T-s24)
- Private endpoints: "now returns 400 `parameters_invalid` when `declared_parameters` includes `tool_choice` without `tools`" (T-s23)
- Basis: `OR-documented`.

### C. Parallel tool calls and tool-call ids

**T13. `parallel_tool_calls` is a plain boolean on Chat Completions and Responses. On the Messages route the same idea is the inverted flag `disable_parallel_tool_use` inside `tool_choice`.**

- Quote (Parameters page): "Key: `parallel_tool_calls` / Optional, **boolean** / Default: **true** / Whether to enable parallel function calling during tool use. If true, the model can call multiple functions simultaneously. If false, functions will be called sequentially. Only applies when tools are provided."
- Quote (Client Tools guide): "Control whether multiple tools can be called simultaneously with the `parallel_tool_calls` parameter (default is true for most models)" and "When `parallel_tool_calls` is `false`, the model will only request one tool call at a time instead of potentially multiple calls in parallel."
- Spec: `ChatRequest.parallel_tool_calls` is `boolean | null`, "Whether to enable parallel function calling during tool use. When true, the model may generate multiple tool calls in a single response." `ResponsesRequest.parallel_tool_calls` is `boolean | null`. `MessagesRequest.tool_choice` variants `auto`, `any`, and `tool` each have `disable_parallel_tool_use: boolean`.
- Sources: T-s8, T-s1, T-s11. Basis: `OR-documented`.

**T14. The catalog lists `parallel_tool_calls` as a supported parameter on only 18 of 476 models, and on none of the OpenAI, Anthropic, Google, xAI, or Mistral models.**

- The 18: `deepseek/deepseek-v4.1-flash-1017`, `deepseek/deepseek-v4.1-flash`, `~deepseek/deepseek-flash-latest`, `~deepseek/deepseek-v4-flash-latest`, `deepseek/deepseek-v4-flash-0731`, `upstage/solar-mini4`, `upstage/solar-pro4`, `xiaomi/mimo-v2.6-flash`, `~z-ai/glm-flash-latest`, `z-ai/glm-5.3-flash`, `~z-ai/glm-latest`, `z-ai/glm-5.3`, `moonshotai/kimi-k2.7-code`, `moonshotai/kimi-k2.6`, `openrouter/auto-code`, `typesafe/jev-router`, `openrouter/auto-beta`, `openrouter/pareto-code`.
- At endpoint level the flag is rarer still: of the 194 endpoints in T9, 6 list it (for example 2 of 46 `z-ai/glm-5.3` endpoints: Cohere and Inceptron).
- Source: T-s12, T-s13. Basis: `OR-documented` (catalog data).
- Confidence note: the docs say the default is true "for most models" (T13), and the catalog says the parameter is supported on about 4 percent of models. Either the catalog flag is filled in for few endpoints, or the parameter is forwarded without being tracked. No page explains the gap. An inference from the `require_parameters` rule in T11: a request that sets both `parallel_tool_calls` and `provider.require_parameters: true` could lose every OpenAI and Anthropic endpoint. That consequence is untested here.

**T15. The spec puts no format, length, or character rule on tool-call ids. They are plain strings in both directions.**

- `ChatToolCall.id`: "Tool call identifier", `type: string`, required, with no `pattern`, `minLength`, or `maxLength`.
- `ChatToolMessage.tool_call_id`: "ID of the assistant message tool call this message responds to", `type: string`, required.
- `ChatStreamToolCall`: only `index` is required. `id`, `type`, `function.name`, and `function.arguments` are optional.
- `FunctionCallItem` (Responses): `call_id` required string, `id` optional (`string | null`).
- By contrast the spec does document a rule for names: `ChatFunctionTool.function.name` is "Function name (a-z, A-Z, 0-9, underscores, dashes, max 64 chars)" with `maxLength: 64`, and `ChatJsonSchemaConfig.name` is "Schema name (a-z, A-Z, 0-9, underscores, dashes, max 64 chars)".
- A search of the mirrored docs for id normalization, rewriting, or remapping of tool-call ids finds no statement either way.
- Source: T-s11. Basis: `OR-documented` (what the spec says) and, for the missing rule, `OR-inferred` from the absence of any id constraint in the spec and docs.

**T16. Examples in OpenRouter's own docs show the upstream vendor's id reaching the caller unchanged, on every route, while the item ids around it are minted by OpenRouter.**

- Responses route, `openai/gpt-4o-mini`: `"id": "fc_tmp_obdhb66hlx", "type": "function_call", ... "call_id": "call_gtKR0Mt9UXdJJQmARAwfQVCg"` (T-s2). The `call_...` value is OpenAI's native format. The `fc_tmp_...` item id is OpenRouter's (message items use `msg_tmp_...`, reasoning items `rs_tmp_...`, server-tool items `st_tmp_...`).
- Responses route, `anthropic/claude-sonnet-5.5`: `"id": "fc_tmp_cawpoqzocd", "type": "function_call", "status": "completed", "call_id": "toolu_bdrk_01XKFdFuUDfRLf6fBX4Ay6CX"` (T-s22). The `toolu_...` value is an Anthropic-family native id, surfaced as an OpenAI-shaped `call_id`. The `bdrk` infix suggests the Amazon Bedrock endpoint served it, which is a reading of the id itself and not something the page states.
- Messages route, `openai/gpt-4.1`: `{ "type": "tool_use", "id": "call_bPQO3MRISnlDU09qwfs1xkZv", "name": "bash", "input": { "command": "git push origin main" } }` (T-s26). An OpenAI-format id inside an Anthropic-shaped `tool_use` block.
- Basis: `OR-inferred`, from response examples printed in OpenRouter's documentation. Confidence: medium to high that ids pass through as issued. The examples do not show what happens to an id on the request path when the next request is routed to a different vendor.

**T17. A third-party stream capture shows the one case where OpenRouter has to mint the id itself: Gemini, whose native function calls have optional ids.**

- Quote (gptel issue 1190, dated 2025-12-18, a raw OpenRouter SSE chunk for `google/gemini-3-flash-preview`): `"delta":{"role":"assistant","content":null,"reasoning":null,"reasoning_details":[{"id":"tool_ls_0anw0Z1C02HL0ck4g5OR","type":"reasoning.encrypted","data":"ZTI0ODMwYTctNWNkNi00MmZlLTk5OGItZWU1MzllNzJiOWMz","format":"google-gemini-v1","index":0}],"tool_calls":[{"index":0,"id":"tool_ls_0anw0Z1C02HL0ck4g5OR","type":"function","function":{"name":"ls","arguments":"{\"path\":\"~\"}"}}]},"finish_reason":"tool_calls","native_finish_reason":"STOP"`
- What it shows: the id has the shape `tool_<function name>_<random>`, the whole `arguments` string arrives in one chunk, the native finish reason `STOP` is normalized to `tool_calls`, and the encrypted reasoning entry (the thought signature) is keyed by the same id as the tool call.
- Source: T-s40. Basis: `OR-inferred` from an issue report (third-party), about 11 months old. Confidence: medium that the id shape still holds today.

**T18. OpenRouter's official AI SDK provider has client-side repairs for ids, arguments, and finish reasons, each with a comment that names the upstream behavior it repairs. The repairs sit in the client, which means the server sends these shapes.**

Repository `OpenRouterTeam/ai-sdk-provider` at commit `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20), file `src/chat/index.ts`.

- Lines 471-480: "// Track seen tool call IDs to ensure uniqueness. Some providers / // return empty, null, or duplicate IDs for parallel tool calls." followed by `if (!toolCallId || seenToolCallIds.has(toolCallId)) { toolCallId = generateId(); }`. The streaming path repeats it at lines 1134-1141.
- Line 488: `input: toolCall.function.arguments ?? '{}',` (arguments can be missing).
- Line 1118: "// Tool call start. OpenRouter returns all information except the arguments in the first chunk."
- Lines 1111-1114: "// No index (allowed by the schema): a delta with a new id / // or, without an id, a new function name starts a new tool / // call; anything else continues the most recent one."
- Lines 1212-1218: "// Some providers resend the full arguments in a later chunk. / // When the accumulated arguments are already a complete JSON / // object/array and the delta is an identical copy (ignoring / // surrounding whitespace), appending it could only produce / // invalid JSON, so ignore it."
- Lines 1239-1244: "// The tool call is intentionally not finalized here even when / // the accumulated arguments happen to parse as JSON: a valid / // prefix can be followed by more fragments (e.g. an early / // root close from the upstream), which would yield a / // truncated tool call followed by stray deltas (#574)."
- Lines 549-551: "// Fix for Gemini 3 thoughtSignature: when there are tool calls with encrypted / // reasoning (thoughtSignature), the model returns 'stop' but expects continuation. / // Override to 'tool-calls' so the SDK knows to continue the conversation."
- Lines 564-565: "// Fix for #420: When finishReason is 'other' (unknown/missing) but tool calls / // were made, infer 'tool-calls' so agentic loops continue correctly."
- Lines 467-468: "// Only attach reasoning_details to the first tool call to avoid / // duplicating thinking blocks for parallel tool calls (Claude)"
- Source: T-s27. Basis: `OR-documented` (official SDK source). The step from "the SDK repairs it" to "the server does not normalize it" is an inference with high confidence, since the SDK talks only to OpenRouter.

**T19. What the caller must send back, as far as OpenRouter's docs state it.**

- Chat Completions: "function tools are nested under `function`, the model returns `tool_calls` on an assistant message, and tool results go back as `role: "tool"` messages." The example replays the assistant message with `"content": null` and its `tool_calls`, then one `{"role": "tool", "tool_call_id": "call_abc123", "content": "..."}` message. (T-s1)
- Responses: "`call_id` is what pairs the output with its originating `function_call`" and "The `id` field is optional for `function_call_output` objects. Only `type`, `call_id`, and `output` are required". (T-s2)
- Order and completeness: "Append every output item in order so the LLM has the full context. It's easy to forget this step!" and, in every streaming example, "Preserve every completed item in order. Reasoning and assistant items are required context for some providers when sending tool results." (T-s1)
- Tools on every turn: "the `tools` parameter must be included in every request (Steps 1 and 3) so the router can validate the tool schema on each call." (T-s1)
- The official SDK provider sends one `role: "tool"` message per tool result, sets `name` to the tool name, and sends assistant `content` as `text || null` next to `tool_calls` (`src/chat/convert-to-openrouter-chat-messages.ts` lines 407-438, T-s27).
- Not stated anywhere in the docs: whether tool messages must directly follow the assistant message, whether their order must match the call order, whether several results may share one message, and what happens to a call with no result. Those are vendor rules (section I) and the docs leave them to the caller or to the closed translator.
- Basis: `OR-documented`.

### D. Schemas and `strict`

**T20. `strict` is an optional per-tool flag on all three routes, and a per-format flag on structured output.**

- `ChatFunctionTool.function.strict`: "Enable strict schema adherence", `boolean | null`. `FunctionTool.strict` (Responses): `boolean | null`. `ChatJsonSchemaConfig.strict`: "Enable strict schema adherence", `boolean | null`. `FormatJsonSchemaConfig.strict` (Responses `text.format`): `boolean | null`.
- `function.parameters` is "Function parameters as JSON Schema object" with `additionalProperties: {}`, meaning the spec accepts any JSON object as the schema and checks nothing inside it.
- Source: T-s11. Basis: `OR-documented`.

**T21. For Anthropic models OpenRouter removes `strict` from tools unless the caller sends a beta header. This is the one documented case of OpenRouter editing a tool definition.**

- Quote: "For **strict tool use** (`strict: true` on tools), you must explicitly pass the `structured-outputs-2025-11-13` header. Without this header, OpenRouter will strip the `strict` field and route normally."
- Quote (same note): "**Structured outputs for JSON schema response format** (`text.format.type: "json_schema"`, or `response_format.type: "json_schema"` on Chat Completions) — the header is automatically applied"
- Quote (table): "Structured Outputs | `structured-outputs-2025-11-13` | Enables the strict tool use feature for supported Claude models, validating tool parameters against your schema to ensure correctly-typed arguments"
- The header is sent as `x-anthropic-beta`. "OpenRouter keeps an allowlist of other `x-anthropic-beta` values for each provider and forwards only the values on the list for the provider that serves the request ... OpenRouter drops every other value".
- Source: T-s14. Basis: `OR-documented`.
- Cross-check against Anthropic today (T-s29, see T51): "The `output_format` parameter has moved to `output_config.format`, and beta headers are no longer required." OpenRouter's page still describes the beta header. Which is current on the wire is not checked here.

**T22. No OpenRouter page says that tool or response schemas are sanitized, rewritten, or validated before they go upstream. The pages say enforcement and translation differ by provider and send the reader to the provider's docs.**

- Quote: "Enforcement varies by provider: some guarantee schema-conforming output, while others translate your schema into their own structured-output format or treat it as a strong hint, so exact compliance is not guaranteed on every endpoint. Strict modes may also restrict which JSON Schema features you can use. See the provider's documentation for details" (T-s3)
- Quote: "**Invalid schema**: The model will return an error if your JSON Schema is invalid" (T-s3)
- Quote: "It doesn't guarantee that the selected provider enforces `strict: true` exactly. Some providers translate the schema into their own format or treat it as a strong hint, so validate the parsed object against your schema before you use it." (T-s9)
- "so the router can validate the tool schema on each call" (T-s1) is the only mention of router-side validation, and it does not say what is validated.
- The official SDK provider sends `parameters: tool.inputSchema` as written, adding `strict` only when the tool sets it (`src/chat/index.ts` lines 253-260, T-s27). Its settings doc says the `structuredOutputs.strict` option exists "to opt out of strict mode for models whose providers don't advertise support for it (e.g. open-source models routed through non-OpenAI-compatible providers)" and that the SDK "defaults to `true`" (`src/types/openrouter-chat-settings.ts` lines 161-179).
- Basis: `OR-documented`. The conclusion "schemas pass as written, apart from the Anthropic `strict` strip" is `OR-inferred` from the lack of any contrary statement, with medium confidence.

**T23. OpenRouter does validate tool-call arguments against the caller's schema, after the fact, to score providers. It does not repair or reject on that basis.**

- Quote: "For each request that includes tools, OpenRouter inspects every tool call the model returned and validates it against the schemas the caller supplied."
- Quote: "Tool call `arguments` are validated against the corresponding `tools[].function.parameters` schema using `@cfworker/json-schema`, pinned to **JSON Schema Draft 7**"
- Quote: "Tools whose `parameters` schema is absent or fails to compile are treated as having no schema and are always considered valid"
- Error buckets, word for word: "`InvalidJson`. `JSON.parse(arguments)` throws.", "`UnknownName`. `function.name` is not present in the request's `tools[]`.", "`MissingName`. The call carries no `function.name` at all.", "`SchemaMismatch`. The validator returns `valid: false` against the resolved schema.", plus two response-level issues: "`NameLeakedInContent`. A structured tool call was emitted, and the assistant text also begins with a requested tool name." and "`UnparsedToolCall`. No structured tool call was emitted, and the assistant text begins with a requested tool name in the model's native tool-call syntax."
- Source: T-s5. Basis: `OR-documented`.
- Why the last bucket matters: it is OpenRouter saying that some provider endpoints return the model's raw tool-call markup as text with no structured `tool_calls`, and that this reaches the caller.

### E. Streaming of tool arguments

**T24. The Chat Completions delta shape for a tool call is in the spec only. The streaming guide covers text, usage, and errors.**

- Spec `ChatStreamDelta.tool_calls`: array of `ChatStreamToolCall`, described as "Tool call delta for streaming responses", with `index` (integer, required, "Tool call index in the array"), `id` ("Tool call identifier"), `type` (`"function"`), `function.name`, and `function.arguments` ("Function arguments as JSON string"). Example in the spec: `{"function": {"arguments": "{\"location\": \"...\"}", "name": "get_weather"}, "id": "call_abc123", "index": 0, "type": "function"}`.
- A search of the streaming guide (`api_reference/streaming`) for `tool` returns no hits.
- The official SDK provider states the contract in a comment: "OpenRouter returns all information except the arguments in the first chunk." (T18)
- Sources: T-s11, T-s21, T-s27. Basis: `OR-documented`.

**T25. The Responses stream uses OpenAI's event names, and the docs tell callers to take the arguments from the completed item.**

- Events used in the examples: `response.output_item.added` (with `item.type === 'function_call'`), `response.function_call_arguments.delta` (spec: `delta`, `item_id`, `output_index`, `sequence_number`), `response.function_call_arguments.done` (spec: `arguments`, `name`, `item_id`, `output_index`), `response.output_item.done`, `response.completed`, `response.incomplete`, `response.failed`.
- Quote (code comment in every streaming example): "A completed function_call item carries the full arguments string."
- Sources: T-s1, T-s2, T-s11. Basis: `OR-documented`.

**T26. OpenRouter switches on Anthropic's incremental argument streaming for every streamed request with tools, to make Anthropic behave like the other providers.**

- Quote: "**Fine-grained tool streaming** — for every streamed request (`stream: true`) that includes user-defined tools, OpenRouter sets `eager_input_streaming: true` on each tool so Anthropic emits tool arguments as a sequence of incremental chunks rather than buffering them. This matches the streaming behavior already exposed by other providers."
- Source: T-s14. Basis: `OR-documented`.
- The official SDK provider also lets a caller pass a per-tool `eager_input_streaming` value through `providerOptions.openrouter` (`src/chat/index.ts` lines 247-263, T-s27). The field is absent from `ChatFunctionTool` in the spec.

**T27. Which providers send the arguments in one piece is not listed in the docs. Two sources show it happens.**

- Gemini through OpenRouter: the 2025-12-18 capture in T17 has the full `arguments` string in the same chunk as the id and name.
- The official SDK provider handles "Some providers resend the full arguments in a later chunk" and an "early root close from the upstream" followed by more fragments (T18).
- Basis: `OR-inferred` (issue report and official SDK source). Confidence: medium.

**T28. A tool call cut off by the token limit: the docs describe how OpenRouter scores it, and say nothing about what the caller receives.**

- Quote (Auto Exacto): "When generation hits the request's token limit (any native finish reason that normalizes to `length`, such as `max_tokens` or `MAX_TOKENS`, or a bare Responses `incomplete`), the last tool call is recorded as `truncated-arguments` instead of `InvalidJson` or `SchemaMismatch` if its arguments fail to parse or never arrived, and it does not count as an error. A caller's `max_tokens` cutting off a tool call does not count against the provider."
- Quote (streaming guide): "**Responses API**: Upstream token-limit errors (such as `max_tokens_exceeded`) end the stream with a `response.incomplete` event (`incomplete_details.reason: "max_output_tokens"`) rather than an error" and "**Chat Completions API**: ... the same token-limit errors become `finish_reason: "length"`"
- The Client Tools streaming examples treat `response.incomplete` as fatal: each one throws an error whose message starts with "Response incomplete:" followed by `incomplete_details.reason`.
- The official SDK provider's rule (lines 1290-1305): "When the model signalled completion ('tool-calls' or 'stop'), forward every tool call, including invalid JSON ... Otherwise (e.g. 'length', 'error', 'content-filter') only forward tool calls whose arguments are complete, parsable JSON." So a partial tool call with unparseable arguments does arrive on the wire with `finish_reason: "length"`, and the client is the one that drops it.
- Sources: T-s5, T-s21, T-s1, T-s27. Basis: `OR-documented` for the quotes, `OR-inferred` (from the SDK's handling) for what arrives on the wire.

### F. Structured output

**T29. `response_format` on Chat Completions accepts five types. Two of them are not in the prose docs.**

- Spec `ChatRequest.response_format`: `oneOf` with discriminator `type`: `text` ("Default text response format"), `json_object` ("JSON object response format"), `json_schema` ("JSON Schema response format for structured outputs"), `grammar` ("Custom grammar response format", fields `type: "grammar"` and `grammar: string`, example `root ::= "yes" | "no"`), and `python` ("Python code response format").
- The API overview type lists only `{ type: 'json_object' }` and `{ type: 'json_schema'; json_schema: { name: string; strict?: boolean; schema: object } }`.
- Broadcast metadata confirms all five are real values: "`response_format_type` is the `response_format` type (`json_schema`, `json_object`, `text`, `grammar`, or `python`), without the schema."
- No page says which providers honor `grammar` or `python`.
- Sources: T-s11, T-s10, T-s24. Basis: `OR-documented`.

**T30. JSON mode.**

- Quote: "Forces the model to produce specific output format. Setting to `{ "type": "json_object" }` enables JSON mode, which guarantees the message the model generates is valid JSON. **Note**: when using JSON mode, you should also instruct the model to produce JSON yourself via a system or user message." (T-s8)
- Quote: "`{ type: 'json_object' }`: Basic JSON mode - the model will return valid JSON" (T-s10)
- Basis: `OR-documented`.

**T31. JSON Schema mode has a different field path on each route.**

- Chat Completions: `response_format: {"type": "json_schema", "json_schema": {"name", "strict", "schema", "description"}}`. `name` is required, "Schema name (a-z, A-Z, 0-9, underscores, dashes, max 64 chars)".
- Responses: `text.format: {"type": "json_schema", "name", "strict", "schema", "description"}` (flat, `name` and `schema` required).
- Messages: `output_config` on `MessagesRequest` (Anthropic's shape).
- Quote: "On both APIs, the model will respond with a JSON object that strictly follows your schema"
- Quote (streaming): "Structured outputs are also supported with streaming responses. The model will stream valid partial JSON that, when complete, forms a valid response matching your schema."
- Sources: T-s3, T-s11. Basis: `OR-documented`.

**T32. `structured_outputs` is a capability name, not a request field. `response_format` in `supported_parameters` means the field is accepted, and `structured_outputs` means the JSON Schema form works.**

- Quote: "Key: `structured_outputs` / Optional, **boolean** / If the model can return structured outputs using response\_format json\_schema." (T-s8)
- In the spec the name appears only in the `Parameter` enum used by `supported_parameters`, never as a `ChatRequest` property (T-s11).
- Counts: 419 models list `response_format`, 407 list `structured_outputs`, 28 list `response_format` only (for example `openai/gpt-4`, `z-ai/glm-4.5`, `qwen/qwen3.7-flash`, `google/gemma-4-31b-it:free`), 16 list `structured_outputs` only (T6).
- Basis: `OR-documented`.

**T33. Support is per endpoint, and the caller finds out from the model page or the endpoints API, then pins with `require_parameters`.**

- Quote: "Support is determined per endpoint, not just per model: the same model may be served by multiple providers, and only some of those providers may support structured outputs. Endpoint support can also change over time. To see which providers support structured outputs for a specific model, check the `structured_outputs` parameter in the Providers section of the model's page."
- Quote: "To ensure your request is only routed to endpoints that support structured outputs: 1. Check the model's supported parameters on the models page 2. Set `require_parameters: true` in your provider preferences 3. Include `text.format` (Responses API) or `response_format` (Chat Completions) with `type: json_schema` in your request"
- Vendors the page links to for "each provider's implementation": OpenAI, Google Gemini, Anthropic, Fireworks.
- Live example of the split (T9): `anthropic/claude-sonnet-5.5` lists `structured_outputs` on 3 of 5 endpoints (Anthropic, Azure, Google Vertex) and leaves it off Amazon Bedrock and Claude Platform on AWS. `z-ai/glm-5.3` lists it on 38 of 46 endpoints. `deepseek/deepseek-v4-pro` lists it on 8 of 13.
- Sources: T-s3, T-s13. Basis: `OR-documented`.

**T34. What happens on a model or endpoint that lacks support is stated two ways.**

- Structured outputs page: "**Model doesn't support structured outputs**: The request will fail with an error indicating lack of support"
- Provider selection page: the response format is one of the soft-preference parameters, and "If none of a model's providers support the parameter, the request is still routed to that model and the parameter is ignored"
- Compatibility page: with `require_parameters: true`, "if no endpoint supports them, the request fails instead of falling back to one that would drop them."
- Sources: T-s3, T-s14, T-s9. Basis: `OR-documented`. The first two statements conflict. See "Conflicts and disagreements".

**T35. Response healing is an opt-in plugin that repairs JSON syntax after a non-streaming response. It does not check the schema.**

- Quote: "The plugin activates for non-streaming requests when you set `text.format` in a Responses API request (or `response_format` on Chat Completions) to either `type: "json_schema"` or `type: "json_object"`, and include the response-healing plugin in your `plugins` array."
- Request form: `"plugins": [{ "id": "response-healing" }]`. Spec `ResponseHealingPlugin.enabled`: "Set to false to disable the response-healing plugin for this request. Defaults to true."
- What it fixes, by the page's own headings: "Missing closing bracket", "Markdown Code Blocks" ("Extracts JSON from markdown code blocks"), "Mixed Text and JSON" (text before the JSON), "Trailing Commas", "Unquoted Keys".
- Limits: "**Non-Streaming Requests Only** Response Healing only applies to non-streaming requests." and "Some malformed JSON responses may still be unrepairable. In particular, if the response is truncated by `max_output_tokens`, the plugin will not be able to repair it."
- The code comment in the example reads "The plugin attempts to repair malformed JSON syntax". Nothing on the page says the repaired JSON is validated against the schema, and nothing says it applies to tool-call `arguments`.
- Sources: T-s4, T-s11. Basis: `OR-documented`.

**T36. Refusals have three shapes: a field on a successful message, a content part on Responses, and a 403 error type.**

- Quote: "A refusal that the model produces as its own completed output is not an error. When the provider answers successfully with a refusal (e.g. Anthropic's `stop_reason: "refusal"`), each API keeps its native successful shape: Chat Completions returns `message.refusal` with `finish_reason: "content_filter"`, and the Responses API returns `status: "incomplete"` with `incomplete_details.reason: "content_filter"` and a `refusal` content part."
- Quote (error table): "`refusal` | 403 | The provider reported a model refusal as an error rather than as output (e.g. Anthropic's "refused to respond" / usage-policy error envelopes)." and "`content_policy_violation` | 403 | The input or output was flagged by a content filter that runs separately from the model (provider- or OpenRouter-level), such as Gemini's `SAFETY` block."
- Spec: `ChatAssistantMessage.refusal` ("Refusal message if content was refused", `string | null`), `ChatStreamDelta.refusal` ("Refusal message delta"), `OpenAIResponsesRefusalContent` (`{"type": "refusal", "refusal": string}`), stream events `RefusalDeltaEvent` and `RefusalDoneEvent`.
- GPT-6 policy blocks: "A policy block is an HTTP 403 with `error_type: "refusal"` on every OpenRouter API ... On Chat Completions the provider code is `error.metadata.provider_code`" and "OpenRouter does not resend a declined prompt to another endpoint serving the same model".
- Sources: T-s20, T-s11, T-s17. Basis: `OR-documented`.

**T37. OpenAI's custom (freeform) tools and grammars are exposed on the Responses route only.**

- Spec `CustomTool` (in `ResponsesRequest.tools`): `{"type": "custom", "name", "description", "format", "async", "defer_loading"}` where `format` is `{"type": "text"}` or `{"type": "grammar", "definition": string, "syntax": "lark" | "regex"}`.
- Spec `CustomToolCallItem`: "A call to a custom (freeform-grammar) tool created by the model — distinct from `function_call`. Used for tools like Codex CLI's `apply_patch` whose payload is opaque text rather than JSON arguments." Fields `type: "custom_tool_call"`, `call_id`, `name`, `input` (string). The reply item is `custom_tool_call_output`.
- `ChatRequest.tools` items (`ChatFunctionTool`) are a function tool or one of OpenRouter's server tools. A custom tool can be declared on the Responses route alone.
- Sources: T-s11, T-s17. Basis: `OR-documented`.

### G. Message-shape rules

**T38. Roles and content shapes accepted on Chat Completions.**

- Spec `ChatMessages`: discriminated on `role` with `system`, `user`, `developer`, `assistant`, `tool`.
- `ChatSystemMessage.content` and `ChatDeveloperMessage.content`: a string or an array of text parts only. `ChatUserMessage.content`: string or array of `ChatContentItems` (text, image, audio, video, file).
- `ChatAssistantMessage.content`: string, array of content items, or `null`. Other properties: `tool_calls`, `refusal`, `reasoning`, `reasoning_details`, `name`, `audio`, `images`.
- `ChatToolMessage.content`: "Tool response content", a string or an array of `ChatContentItems`. So the spec allows images and files in a tool result on Chat Completions, while the API overview's TypeScript type says `role: 'tool'; content: string;`.
- Error type for a bad message: "`invalid_prompt` | 400 | A specific message in the request (`input` on the Responses API, `messages` on Chat Completions) is invalid (e.g. unsupported role, empty content)."
- Sources: T-s11, T-s10, T-s20. Basis: `OR-documented`.

**T39. OpenRouter combines messages for providers that need it, and the only way for a caller to see how is the debug echo.**

- Quote: "**Verifying Message Formatting**: Check how OpenRouter combines and formats your messages for different providers (e.g., how system messages are concatenated, how user messages are merged)."
- The echo example for `anthropic/claude-haiku-5.5` shows the system message lifted into Anthropic's top-level field: `"system": [ { "type": "text", "text": "You are a helpful assistant." } ], "messages": [ { "role": "user", "content": "Hello!" } ], "model": "claude-haiku-5-5"`.
- Limits of the echo: "The debug option **only works with streaming mode** (`stream: true`). Non-streaming requests will ignore the debug parameter." and "The debug flag should **not be used in production environments**."
- Source: T-s20. Basis: `OR-documented`.
- Not documented: the rules themselves (which system messages are concatenated, what happens to a system message in the middle of a conversation on a vendor that only takes a leading one, how consecutive assistant messages are treated).

**T40. The `name` field is folded into the text for non-OpenAI models.**

- Quote (comment in the request type): "// If "name" is included, it will be prepended like this / // for non-OpenAI models: `{name}: {content}`"
- Source: T-s10. Basis: `OR-documented`.

**T41. Assistant prefill is offered as a general feature and passed through. Where the vendor rejects it, the vendor's 400 comes back.**

- Quote: "OpenRouter supports asking models to complete a partial response ... simply include a message with `role: "assistant"` at the end of your `input` array (Responses API) or `messages` array (Chat Completions)."
- Quote (Claude 5.5 guide): "Haiku 5.5 rejects it on every provider, even with thinking off, and OpenRouter returns the provider's 400 (`This model does not support assistant message prefill. The conversation must end with a user message.`)."
- Quote: "**Sampling parameters and assistant prefill are removed.** Non-default `temperature`, `top_p`, and `top_k` are not supported, and neither is a final assistant turn (already the case on Opus 5)"
- Sources: T-s10, T-s15. Basis: `OR-documented`.

**T42. Mid-conversation system messages, tool-set changes, and per-turn settings live on the Messages route. Chat Completions gets one narrow extension.**

- Quote: "**Mid-conversation system messages and `clear_at`**: append a `role: "system"` message instead of editing the system prompt ..." under "These Messages API controls work on all three models through OpenRouter's Messages API (`/api/v1/messages`). You do **not** need to send Anthropic beta headers: OpenRouter detects each feature, attaches the beta, and routes only to providers that support it." (T-s15)
- Quote (tool changes): "This feature is only supported on OpenRouter's Messages API (`/api/v1/messages`) — the Chat Completions API has no representation for these blocks." (T-s19)
- The Chat Completions extension: spec `ChatSystemMessage.configuration_update`, "OpenRouter extension. Changes reasoning effort from this point in the conversation onward without invalidating the prompt cache for the preceding turns. Place it on a content-less system message (`content: \"\"`) directly before the user message it should apply to". "A `configuration_update` on a user, assistant, or tool message is rejected with a 400." (T-s11, T-s17)
- Basis: `OR-documented`.

**T43. Images in tool results: documented for Responses, present in the Chat Completions spec, and forwarded only to models that take them.**

- Quote: "`function_call_output.output` accepts either a string or an array of input content parts (`input_text`, `input_image`, `input_file`) — the same shape as user-message content. Use the array form to return images or files from a tool; the non-text parts are only forwarded to supported multimodal models."
- The official SDK provider maps a tool result with content parts to `image_url`, `video_url`, `input_audio`, or `file` parts inside the `role: "tool"` message (`mapToolResultContentParts`, lines 474-540, T-s27).
- Sources: T-s2, T-s27. Basis: `OR-documented`.
- Not documented: what a text-only model receives in place of the dropped parts.

**T44. Empty content next to tool calls.**

- The docs' own replay example sends `"role": "assistant", "content": null, "tool_calls": [...]` (T-s1). The Alignment page states the response side: "On a message that consists of tool calls, `content` is `null`" (T-s22).
- The official SDK provider sends `content: ... text || null` (T19).
- Basis: `OR-documented`.

**T45. Two OpenAI-specific item features do not fit Chat Completions, and the docs say so.**

- `phase`: "`phase` is **not available** in the Chat Completions API. The Chat Completions format cannot represent multiple output items with distinct phases in a single response." and "If `phase` metadata is dropped when reconstructing conversation history, significant performance degradation can occur — including early stopping on longer-running tasks." (T-s18)
- Async tools: "OpenRouter supports async tools on the Responses API. Chat Completions and Anthropic Messages requests run every tool call synchronously. A Chat Completions tool that carries `async` is accepted, but the flag is removed before the request is routed and the call behaves like any other tool call." Routing: "OpenAI and Azure OpenAI honor async tools for the GPT-6 family. When a request declares an async tool and one of those endpoints is a routing candidate, OpenRouter narrows routing to them. When none is ... the `async` flag is removed before the request is sent ... Amazon Bedrock accepts the flag but runs the tool synchronously, so it is treated as a non-async transport." (T-s17)
- Basis: `OR-documented`.

### H. Per-vendor caveats and Exacto

**T46. Auto Exacto exists because the same model calls tools with different reliability on different providers.**

- Quote (Exacto variant): "Providers serving the same model can vary meaningfully in tool-use behavior. Exacto gives you an explicit, request-level way to prefer higher-quality providers when you care more about tool-calling reliability than default routing."
- Quote (Auto Exacto): "Auto Exacto is a routing step that automatically optimizes provider ordering for all requests that include tools. It runs by default on every tool-calling request, requiring no configuration."
- Signals: "**Throughput.**", "**Tool-calling success rate.** How reliably each provider completes tool calls", "**Benchmark data.** Results from OpenRouter's own benchmark harness" (GPQA Diamond and Tau2-Bench Airline, scores over "a **rolling 32-day window**").
- Metric: "requests_with_tool_call_errors / requests_where_finish_reason_is_tool_calls", shown as "**Tool Call Error Rate** on the Performance tab of every model page".
- Provider-side thresholds: "**Tool-calling success rate** -- providers falling more than **2 standard deviations** below the median are deprioritized." and "An endpoint needs at least 100 general requests (30-minute window) and 200 tool-call requests (2-hour window) before it can be evaluated." (T-s7)
- Opt out: "disable Auto Exacto by setting `sort: "price"` in the `provider` object or using the `:floor` model variant."
- Cost of it: "Auto Exacto reorders providers on every tool-calling request, which can conflict with the sticky routing used by prompt caching ... In tool-calling agent loops, this can cause cache misses mid-conversation."
- Sources: T-s5, T-s6, T-s7. Basis: `OR-documented`.

**T47. Every place the tool and structured-output docs name a vendor or say support differs by provider.**

| Where | What it says | Source |
|---|---|---|
| Parameters, `tools` | "For non-OpenAI providers, it will be transformed accordingly." | T-s8 |
| Parameters, intro | "OpenRouter will also transmit some provider-specific parameters, such as `safe_prompt` for Mistral or `raw_mode` for Hyperbolic directly to the respective providers if specified." | T-s8 |
| Structured outputs | "Support is determined per endpoint, not just per model"; links to OpenAI, Google Gemini, Anthropic, Fireworks | T-s3 |
| Structured outputs, strict | "Enforcement varies by provider ..." | T-s3 |
| Provider selection, Anthropic beta | `structured-outputs-2025-11-13` header for strict tools; `strict` stripped without it; `eager_input_streaming` set automatically; `interleaved-thinking-2025-05-14` | T-s14 |
| Client Tools streaming examples | "Reasoning and assistant items are required context for some providers when sending tool results." | T-s1 |
| Responses tool calling | "the non-text parts are only forwarded to supported multimodal models" | T-s2 |
| Claude 5.5, Fable 5.1 guides | forced `tool_choice` returns the provider's 400; prefill rejected; computer-use tools "rejected at validation" on OpenRouter's Messages API | T-s15, T-s16 |
| Opus 5 guide | mid-conversation tool changes only on Messages; "As of launch, Amazon Bedrock supports tool changes on Claude Opus 5 only." | T-s19 |
| GPT-6 guide | async tools honored by OpenAI and Azure OpenAI only; Amazon Bedrock runs them synchronously | T-s17 |
| GPT-5.4 guide | `phase` on Responses only | T-s18 |
| `CustomTool.async` in the spec | "Only honored by providers whose Responses API supports async tools; ignored elsewhere." | T-s11 |
| Endpoints API | `supports_tool_choice` per endpoint | T-s11, T-s13 |
| Auto Exacto, Exacto | provider variance in tool-use reliability | T-s5, T-s6 |

Basis: `OR-documented`.

### I. Vendor rules a translator has to satisfy (read from vendor sources on 2026-11-22)

**T48. Anthropic `tool_choice`: four values, and forced use is rejected on six models and under manual extended thinking.**

- Quote: "When working with the `tool_choice` parameter, there are four possible options: `auto` allows Claude to determine whether to call any provided tools. This is the default value when `tools` are provided. `any` tells Claude that it must use one of the provided tools, but doesn't force a particular tool. `tool` forces Claude to always use a particular tool. `none` prevents Claude from using any tools. This is the default value when no `tools` are provided."
- Quote: "Not every model and setting supports forced tool use. Where it isn't supported, `tool_choice: {"type": "any"}` and `tool_choice: {"type": "tool", "name": "..."}` fail, whereas `tool_choice: {"type": "auto"}` (the default) and `tool_choice: {"type": "none"}` still work"
- Quote (table): "Manual extended thinking (`thinking: {type: "enabled"}`) | `any` and `tool` are not supported and result in an error" and "Claude Fable 5.2, Claude Mythos 5.2, Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1, and Claude Mythos 5.1 | `any` and `tool` return a 400 error."
- Quote: "Forcing a tool also skips thinking on every model that accepts forced tool use: the response starts with the `tool_use` block and has no `thinking` block."
- Quote: "When using prompt caching, changes to the `tool_choice` parameter will invalidate cached message blocks."
- Source: T-s28. Basis: `Vendor-documented`. Also in workspace: the 2026-11-17 Anthropic Messages report ("A forced `tool_choice` is a 400 on six models"), which this confirms.

**T49. Anthropic pairing and ordering of `tool_use` and `tool_result`.**

- Quote: "Add a `user` message with one `tool_result` block for each `tool_use` block in that content, even if the tool failed."
- Quote: "Tool result blocks must immediately follow their corresponding tool use blocks in the message history. You cannot include any messages between the assistant's tool use message and the user's tool result message."
- Quote: "In the user message containing tool results, the `tool_result` blocks must come FIRST in the content array. Any text must come AFTER all tool results."
- Quote: "If you receive an error such as "tool_use ids were found without tool_result blocks immediately after", check that your tool results are formatted correctly."
- Quote: "`content` (optional): The result of the tool as a string or a list of content blocks. These content blocks can use the `text`, `image`, `document`, or `search_result` types." and "`is_error` (optional): Set to `true` if the tool failed or the call was invalid."
- Quote: "Unlike APIs that separate tool use or use special roles like `tool` or `function`, the Claude API integrates tools directly into the `user` and `assistant` message structure."
- Source: T-s30. Basis: `Vendor-documented`. Also in workspace: `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, Appendix B ("Anthropic requires all `tool_result` blocks in one user message immediately after the `tool_use` message"), which this confirms.

**T50. Anthropic name and id patterns.**

- Tool name, live: "`name` | The name of the tool. Must match the regex `^[a-zA-Z0-9_-]{1,128}$`." (T-s28, `Vendor-documented`)
- Tool-call id pattern `^[a-zA-Z0-9_-]+$`: from the 2026-11-17 workspace report (its source A11, Anthropic's API reference) and from the error string it records, "`tool_use.id`: String should match pattern `^[a-zA-Z0-9_-]+$`". Not re-fetched from Anthropic today. Basis: `Workspace`, corroborated by LiteLLM's docstring "Anthropic requires tool_use_id to match ^[a-zA-Z0-9_-]+$." (T-s44).

**T51. Anthropic structured output and strict tool use: one grammar pipeline, one list of schema limits, and a beta header that is no longer needed.**

- Quote: "JSON outputs (`output_config.format`): Get Claude's response in a specific JSON format" and "Strict tool use (`strict: true`): Guarantee schema validation on tool names and inputs."
- Quote: "The `output_format` parameter has moved to `output_config.format`, and beta headers are no longer required. The `output_format` parameter is deprecated. To use it anyway, add the `structured-outputs-2025-11-13` beta header. Without it, the API returns a 400 error."
- Quote: "Structured outputs are available on every model"
- Supported, word for word: "All basic types: object, array, string, integer, number, boolean, null", "`enum` and `const` (strings, numbers, booleans, or nulls only)", "`anyOf` and `allOf`, except the uses listed under "Not supported"", "`$ref`, `$defs`, and `definitions`", "`required` and `additionalProperties` (must be set to `false` for objects)", "Array `minItems` (only values 0 and 1 supported)".
- Not supported, word for word: "Recursive schemas", "`oneOf` (use `anyOf` instead)", "`not`", "`anyOf` in the same schema object as keywords such as `type` or `properties`", "External `$ref`", "Numerical constraints (such as `minimum`, `maximum`, `multipleOf`)", "String constraints (`minLength`, `maxLength`)", "Array constraints beyond `minItems` of 0 or 1", "`additionalProperties` set to anything other than `false`". "If you use an unsupported feature, you'll receive a 400 error with details."
- Limits: "Strict tools per request | 20", "Optional parameters | 24 | Total optional parameters across all strict tool schemas and JSON output schemas", "Parameters with union types | 16". Over the internal limits: "a 400 error with the message "Schema is too complex for compilation.""
- Not compatible: "Message prefilling: Prefilling is incompatible with JSON outputs." and citations (400).
- Output may still miss the schema: "Refusals (`stop_reason: "refusal"`) ... You'll receive a 200 status code ... The output might not match your schema" and "Token limit reached (`stop_reason: "max_tokens"`) ... The output might be incomplete and not match your schema."
- Source: T-s29. Basis: `Vendor-documented`.
- The "root must be an object" rule for tool `input_schema` is from the workspace report (its Table B1: "Copy `parameters`. The root must be an object schema ... pi notes a root `anyOf` is a 400"). Not re-fetched today. Basis: `Workspace`.

**T52. Gemini function calling. Google's page now documents the Interactions API, where calls have ids and results name them.**

- Modes: "Control how the model uses tools using `tool_choice` in `generation_config`: `auto` (Default): Model decides whether to call a function or respond directly. `any`: Model is constrained to always predict a function call. `none`: Model is prohibited from making function calls. `validated`: Model ensures function schema adherence." A named subset is `{"tool_choice": {"allowed_tools": {"mode": "any", "tools": ["get_current_temperature"]}}}`.
- Replay: "In stateless mode, you must pass the full history of the conversation in the `input` field of each subsequent request. This history must include: ... All model-generated steps returned in Turn 1 (including `thought` and `function_call` steps) exactly as received. The `function_result` step containing the output of your executed function." Results carry `"call_id": fc_step.id`.
- Tools every turn: "Tool declarations aren't stored with the interaction. Send the same `tools` array on every request".
- Schema: "Only a subset of the OpenAPI schema is supported." and "For `any` mode, very large or deeply nested schemas may be rejected." Over the limits "the API rejects the request with a `400 INVALID_ARGUMENT` error."
- Streaming: "the model generates function calls as a sequence of `step.delta` events on the stream. Tool arguments can be streamed as partial arguments using `arguments`."
- Tool results: "For Gemini 3 and later models, you can include multimodal content in the function response parts that you send to the model."
- Thought signatures page today: "When you use the Interactions API, you don't need to manually extract or pass signatures on individual content parts." and, in stateless mode, "Signatures stay cleanly inside these steps instead of attaching to user inputs, model text outputs, or function calls."
- Sources: T-s31, T-s51. Basis: `Vendor-documented`.
- Not re-checked: the `generateContent` rules in the workspace matrix (a `thought_signature` on the first `functionCall` of each step, all parallel `functionCall`s before their `functionResponse`s, optional ids). Google's current pages describe the Interactions API and no longer show them. They stay `Workspace` (2026-09-20). The 2025-12-18 issue in T17 shows the upstream error text OpenRouter relayed at the time: "Function call is missing a thought_signature in functionCall parts."

**T53. Gemini structured output: a schema subset, and since this version of the page the server strips the keywords it cannot enforce.**

- Quote: "Gemini's structured output mode supports a subset of the JSON Schema specification."
- Quote: "`additionalProperties`: Controls whether properties not listed in `properties` are allowed. Can be a boolean or a schema."
- Quote: "Unenforced constraints: `minLength`, `maxLength`, `minProperties`, `maxProperties`, `uniqueItems`, `contains`, `minContains`, and `maxContains` (and their snake_case variants) are not enforced by the constrained decoding grammar engine. If these keywords are present in your schema ... the API automatically strips and ignores them during request normalization rather than returning an error. `minItems` and `maxItems` are enforced."
- Quote: "Schema complexity: Very large or deeply nested schemas may be rejected." with "`400 INVALID_ARGUMENT`".
- Request form: `response_format: {"type": "text", "mime_type": "application/json", "schema": ...}`.
- Source: T-s32. Basis: `Vendor-documented`.

**T54. Mistral: three `tool_choice` words, and a hard id format enforced by Mistral's own validator library.**

- Quote (docs): "Developers can use `tool_choice` to specify how tools are used: "auto": default mode. Model decides if it uses the tool or not. "any": forces tool use. "none": prevents tool use." and "`parallel_tool_calls` ... true: default mode. The model decides if it uses parallel tool calls or not. false: forces the model to use single tool calling." The docs' example id is `"id": "D681PevKs"`. (T-s33)
- Validator source, `mistralai/mistral-common` at blob `7edf6f65`, `src/mistral_common/protocol/instruct/validator.py`, class `MistralRequestValidatorV3`: `if not re.match(r"^[a-zA-Z0-9]{9}$", message.tool_call_id): raise InvalidToolMessageException(f"Tool call id was {message.tool_call_id} but must be a-z, A-Z, 0-9, with a length of 9.")`, the same check on `tool_call.id`, and `raise InvalidFunctionCallException("Tool call id has to be defined in serving mode.")`. Function and tool names: `^[a-zA-Z0-9_-]{1,64}$`. (T-s34)
- Basis: `Vendor-documented` (vendor docs and the vendor's own open-source validator). Also in workspace: Appendix B ("Mistral ... exactly nine alphanumerics ... error code 3280"), which this confirms.
- OpenRouter's own endpoint flags for `mistralai/mistral-large-2512` say `none: false` (T9), which does not match Mistral's docs listing `"none"`.

**T55. DeepSeek: forced tool choice is a 400 in thinking mode, JSON Schema output is not offered natively, and strict tools need a beta base URL.**

- Quote: "`required` and named tool choices are not supported in thinking mode; the API returns a `400` error. Disable thinking mode first to use them. With the web tools declared, `required` is treated as `auto`." and "`none` is the default when no tools are present. `auto` is the default if tools are present." (T-s37)
- Quote: "Thinking mode is enabled by default" (T-s36)
- Quote (`response_format.type`): "Possible values: `text`, `json_object`" and "Must be one of `text` or `json_object`." (T-s37)
- Quote: "To use `strict` mode, you need to: 1. Use `base_url="https://api.deepseek.com/beta"` to enable Beta features 2. In the `tools` parameter，all `function` need to set the `strict` property to `true` 3. The server will validate the JSON Schema of the Function provided by the user. If the schema does not conform to the specifications or contains JSON schema types that are not supported by the server, an error message will be returned" and "All properties of every `object` must be set as `required`, and the `additionalProperties` attribute of the `object` must be set to `false`." (T-s35)
- Quote: "The Chat Completion API does not support inserting tool calls mid-conversation but does support inserting `system` messages mid-conversation; to insert tool calls, use the Anthropic API or the Responses API instead." (T-s35)
- Quote: "for requests carrying the `tools` parameter, the `reasoning_content` must be fully passed back to the API in all subsequent requests — even for turns where the model did not perform a tool call. If your code does not correctly pass back `reasoning_content`, the API will return a 400 error." (T-s36)
- Function name: "Must be a-z, A-Z, 0-9, or contain underscores and dashes, with a maximum length of 128." Example ids: `call_00_kw66qNnNto11bSfJVIdlV5Oo`. (T-s37, T-s36)
- Basis: `Vendor-documented`. Also in workspace: Appendix B (the `reasoning_content` rule), which this confirms. The `tool_choice` limit matches the `NA..` flags on 9 of 13 `deepseek/deepseek-v4-pro` endpoints in T9.

**T56. Meta's model API: `tool_choice` must be `auto`, and ids have a length limit.**

- Quote: "`tool_choice` must be `"auto"`: Only `"auto"` (the default) is supported on both Chat Completions and the Responses API; `"none"`, `"required"`, and named function choices return `HTTP 400` ("only `"auto"` is supported for `tool_choice`")."
- Quote: "Keep `call_id` within 1–64 characters: Every `call_id` on both `function_call` and `function_call_output` items must be 1 to 64 characters. An empty or overlength `call_id` returns `HTTP 400`."
- Quote: "Function tool names must match `^[a-zA-Z0-9_.-]+$` (alphanumeric, underscores, hyphens, and dots) and contain at most one dot. Names with two or more dots return `HTTP 400`."
- Quote: "If a function tool's `parameters` schema omits the root `"type": "object"`, Model API adds it before the schema reaches the model"
- Quote: "`strict` defaults to `false` ... Set `strict: true`: the server validates your schema against the strict subset ... A schema that violates the subset returns `HTTP 400`" and "Regardless of `strict`, the model's generated arguments are guided by your `parameters` schema but are not guaranteed to validate against it."
- Quote: "Execute every call, then append one `tool` message per call, each with its own `tool_call_id`, before the next request." and "Always include the full assistant message ... Omitting it returns `HTTP 400`." and "When you build history manually without `previous_response_id`, keep tool results in the same order as their corresponding tool calls."
- Quote: "Custom tools are Responses-API-only; sending `{"type": "custom", ...}` to Chat Completions returns `HTTP 400`."
- Source: T-s38. Basis: `Vendor-documented`.
- Note: these are the rules of Meta's own API. The `meta-llama/*` models on OpenRouter are served by third-party hosts (Parasail, DeepInfra, Groq, Amazon Bedrock and others in T9), whose flags differ.

**T57. Moonshot Kimi: the `functions.name:idx` id rule no longer applies to K3.**

- Quote (Moonshot's `encoding_k3.py` in the `moonshotai/Kimi-K3` repository): tool results are "matched against the most recent preceding assistant ``tool_calls`` by opaque ``tool_call_id`` == ``tool_calls[].id`` (K3 drops the ``func:index`` format requirement) and sorted by the matched 1-based position." (T-s42)
- Quote (platform docs): "If the assistant message has `tool_calls`, the next messages must be exactly the same `role=tool` messages as the `tool_calls`, and `tool_call_id` must be aligned one-to-one with the previous `tool_calls.id`", "If there are multiple `tool_calls`, the order is not sensitive", "The uniqueness requirement is only local to the `tool_calls`-response in this round, not for the entire conversation or globally.", and "you must return results for all `tool_calls` for the model to continue, otherwise the request will be considered invalid and rejected". Example id: `"id": "web_search:0"`. (T-s43)
- Quote (K3 page): "Use `tool_choice="required"` on the first turn to require at least one tool call.", "K3 always has thinking mode enabled", "Use `json_schema` with `strict: true` to constrain the final `message.content`.", "Return the complete assistant message unchanged in multi-turn conversations and tool calls.", and dynamic loading: "Place a complete tool definition in a `system` message without `content`." (T-s52)
- Basis: `Vendor-documented`. This corrects the workspace matrix. See "Conflicts and disagreements".

**T58. OpenAI: the reference shape, with its own strict rules.**

- Quote: "Auto: (Default) Call zero, one, or multiple functions. `tool_choice: "auto"`", "`tool_choice: "required"`", "`tool_choice: {"type": "function", "name": "get_weather"}`", "Allowed tools: Restrict the tool calls the model can make to a subset", "You can also set `tool_choice` to `"none"` to imitate the behavior of passing no functions."
- Quote: "The model may choose to call multiple functions in a single turn. You can prevent this by setting `parallel_tool_calls` to `false`, which ensures exactly zero or one tool is called."
- Quote: "1. `additionalProperties` must be set to `false` for each object in the `parameters`. 2. All fields in `properties` must be marked as `required`." and "If you omit `strict`, the default depends on the API: Responses requests will attempt to normalize your schema into strict mode when possible, and will fall back to non-strict, best-effort function calling if the schema cannot be made compatible with strict mode. When fallback happens, the response tool will show `strict: false`. Chat Completions requests remain non-strict by default."
- Quote (custom tools): "You can provide a custom CFG using the `grammar` parameter when configuring a custom tool. Currently, we support two forms of CFG syntax when defining grammars: `lark` and `regex`." Output item type `custom_tool_call`.
- Structured outputs: "Root objects must not be `anyOf` and must be an object", "A schema may have up to 5000 object properties total, with up to 10 levels of nesting.", "If you turn on Structured Outputs by supplying `strict: true` and call the API with an unsupported JSON Schema, you will receive an error."
- JSON mode: "the API will throw an error if the string "JSON" does not appear somewhere in the context."
- Sources: T-s39, T-s41. Basis: `Vendor-documented`.

### J. Fills: how open translators and vendor SDKs do what OpenRouter's docs leave unsaid

A fill shows that a solution exists and how that project does it. It does not show what OpenRouter does.

**T59. LiteLLM rewrites tool-call ids to fit the target vendor, per vendor.** (`BerriAI/litellm` at commit `90dce4b93bd2b090621a6b9083311294d3358bc6`, 2026-11-22)

- Anthropic, `litellm/litellm_core_utils/prompt_templates/factory.py` lines 1825-1837: `_ANTHROPIC_TOOL_USE_ID_INVALID_CHARS: Final = re.compile(r"[^a-zA-Z0-9_-]")` and `def sanitize_anthropic_tool_use_id(tool_use_id: str) -> str: """Anthropic requires tool_use_id to match ^[a-zA-Z0-9_-]+$."""`. Invalid characters become `_`, and an empty result becomes `"tool_use_id"`.
- Bedrock, same file lines 1840-1850: "Bedrock Converse requires toolUseId to match [a-zA-Z0-9_.:-]+ and be at most 64 chars. Ids that need rewriting get a short hash of the original appended so two ids that only differ in a replaced char or past the cut still map to distinct values."
- Gemini signatures in ids, `litellm/llms/anthropic/common_utils.py` lines 2365-2375: "Strips Gemini thought-signature suffixes (``__thought__``) first, then replaces any remaining invalid characters with underscores."
- Mistral, `litellm/llms/mistral/chat/transformation.py` lines 802-817: the tool call is rebuilt with `id=_tool.get("id")`, the id as given. A search of the tree for a nine-character rewrite finds none.
- Basis: `Filled` (LiteLLM).

**T60. LiteLLM and agentgateway translate `tool_choice` value by value and do not downgrade a forced choice for models that reject it.**

- LiteLLM, `litellm/llms/anthropic/chat/transformation.py` lines 1207-1248 (`_map_tool_choice`): `"auto"` to `type="auto"`, `"required"` to `type="any"`, `"none"` to `type="none"`, a named function to `type="tool"` with `name`. "Anthropic uses 'disable_parallel_tool_use' flag to determine if parallel tool use is allowed / this is the inverse of the openai flag."
- agentgateway (`agentgateway/agentgateway` at commit `888badbbe91f2a57ec8e7bf5a5ef3acda34b1fe5`, 2026-11-20), `crates/llm/src/conversion/messages.rs` lines 783-808: `let disable_parallel_tool_use = req.parallel_tool_calls.map(|p| !p);`, `Required` to `messages::ToolChoice::Any`, a named function to `ToolChoice::Tool`, and with no `tool_choice` but a parallel flag and tools, `ToolChoice::Auto { disable_parallel_tool_use }`.
- Neither function looks at the model name. A forced choice on Claude Opus 5.5 would reach Anthropic and fail there, the same outcome OpenRouter documents in T10.
- Basis: `Filled` (LiteLLM, agentgateway).

**T61. LiteLLM's prompt emulation for models with no native function calling uses a JSON instruction in the system message.**

- `litellm/litellm_core_utils/prompt_templates/factory.py` lines 6646-6662 (`function_call_prompt`): the header is `"""To call a function, reply with JSON ONLY in this format {"name": "function_name", "arguments":{"argument_name": "argument_value"}}. Once a function result answers the request, reply to the user in plain text instead of calling a function again. The following functions are available to you:"""`, followed by each function definition, appended to the system message or added as a new one.
- It runs when `litellm.add_function_to_prompt` is set (`litellm/utils.py` lines 5786-5792), and LiteLLM sets it on its own for some Ollama models while forcing `optional_params["format"] = "json"`.
- Basis: `Filled` (LiteLLM). It shows one way to build the third path in T1. OpenRouter's comment says YAML and shows no template.

**T62. agentgateway normalizes tool schemas for Gemini, passes response schemas through, synthesizes missing ids, and hides the thought signature inside the id.** (`crates/llm/src/conversion/vertex_gemini.rs`)

- Lines 763-766: "Gemini's functionDeclarations[].parameters accept only a subset of JSON Schema (the generativelanguage v1beta content.proto Schema message), so tool parameters are normalized below. Structured-output schemas bypass this: they egress as raw JSON Schema via `responseJsonSchema`, which Google recommends over the legacy `responseSchema` subset."
- Lines 797-798: "Normalize an OpenAI JSON Schema into the subset Gemini accepts for tool parameters: inline `$ref`/`$defs`, rewrite unsupported keywords, and strip `additionalProperties`."
- Lines 1261-1265: `fn tool_call_id(native: Option<&str>, seed: &str, index: u32) -> String { native.map(str::to_string).unwrap_or_else(|| format!("call_{seed}_{index}")) }`
- Lines 12-16: "Vertex rejects an `id` on functionCall/functionResponse parts and Gemini 3 hard-400s if a functionCall's signature is not echoed back, but OpenAI clients drop unknown fields while reliably echoing `tool_call_id`. So the signature rides inside the id as `<base-id>__thought__<signature>`, recovered before the outbound Vertex request."
- Lines 319-320: "Carry the base id as a transient correlation key, the post-loop pass orders the responses to match the call order, then strips it."
- LiteLLM has the same schema work in `litellm/llms/vertex_ai/common_utils.py` (`_filter_anyof_fields` at line 1081: "When anyof is present, only keep the anyof field and its contents - otherwise VertexAI will throw an error"; `_convert_schema_types` at line 1302: "Convert type arrays and lowercase types for Vertex AI compatibility"), and sends a tool whose root Vertex cannot take as `parametersJsonSchema`.
- Basis: `Filled` (agentgateway, LiteLLM). Contrast with OpenRouter, which per T17 and T18 keeps the Gemini signature in `reasoning_details` keyed by the tool-call id, a separate field the caller must replay.

**T63. Anthropic's own SDKs define a schema-sanitizing recipe for its strict subset.**

- Quote: "Most SDK helpers transform schemas that use unsupported features. The transformation steps: 1. Remove unsupported constraints (for example, `minimum`, `maximum`, `minLength`, `maxLength`) 2. Update descriptions by adding each unsupported constraint to the field's description (for example, `{minimum: 100}`) 3. Add `additionalProperties: false` to all objects"
- Quote: "A helper that validates responses still checks the response against `minimum: 100`."
- Source: T-s29. Basis: `Filled` (Anthropic, vendor guidance usable by a gateway author).

**T64. Third-party reports of a Mistral id error arriving through OpenRouter's error envelope.**

- Quote (project note, undated, read 2026-11-22): `Error code: 400 - {'error': {'message': 'Provider returned error', 'code': 400, 'metadata': {'raw': '{"object":"error","message":"Tool call id was call_98660886 but must be a-z, A-Z, 0-9, with a length of 9.","type":"invalid_function_call","param":null,"code":"3280"}', 'provider_name': 'Mistral'}}`. The outer `'Provider returned error'` with `metadata.raw` and `metadata.provider_name` is OpenRouter's error shape. (T-s46)
- Quote (Zed issue 53034, opened 2026-04-03, direct Mistral API): `"Tool call id was toolu_01VpEm654HvfMoRcrE3Hgdc6 but must be a-z, A-Z, 0-9, with a length of 9.","type":"invalid_function_call","param":null,"code":"3280"`, and a commenter on 2026-04-09: "At first this happened only when I used a Mistral model right after using an OpenRouter model". (T-s47)
- Basis: `OR-inferred` from third-party reports. They suggest OpenRouter forwarded a non-conforming id to Mistral unchanged and relayed the 400. The first report is undated, so whether this still happens is unknown.

### K. Cross-checks

**T65. Two vendors offer prefill through their own flag on the last assistant message. OpenRouter's docs describe only the plain form.**

- DeepSeek: "`prefix` bool (Beta) Set this to `true` to force the model to start its answer by the content of the supplied prefix in this `assistant` message. You must set `base_url="https://api.deepseek.com/beta"` to use this feature." (T-s37)
- Kimi: "Add an assistant message with `partial=True` at the end of `messages` to continue from a text prefix. Prepend that prefix when displaying the final result." (T-s52)
- OpenRouter: "simply include a message with `role: "assistant"` at the end" (T41). No page mentions `prefix` or `partial`.
- Basis: `Vendor-documented`.

**T66. The workspace matrix (`runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, Table 2 and Appendix B), row by row, against today's sources.**

| Matrix claim (2026-09-20) | Status on 2026-11-22 | Evidence |
|---|---|---|
| Anthropic: `tool_result.tool_use_id` must match a prior `tool_use`; all results in one user message immediately after | Confirmed live | T49 |
| Anthropic: dangling calls are a 400 | Confirmed by the live error text "tool_use ids were found without tool_result blocks immediately after" | T49 |
| OpenAI Chat Completions: one `tool` message per `tool_call_id` immediately following | Not re-fetched from OpenAI today. Meta's API states the same rule for its own endpoint | T56 |
| Gemini 3: `thought_signature` on the first `functionCall` of each step; ids optional; all parallel `functionCall`s before their `functionResponse`s | Not re-checkable on Google's current pages, which now document the Interactions API (calls have `id`, results have `call_id`, signatures sit in `thought` steps) | T52 |
| DeepSeek: `reasoning_content` on every historical assistant message once `tools` is present, else 400 | Confirmed live | T55 |
| Kimi K3 / k2.7-code: ids must be `functions.name:idx` | No longer true for K3: Moonshot's encoder says "K3 drops the ``func:index`` format requirement". Docs examples show ids like `web_search:0`. K2-family templates not re-checked | T57 |
| Mistral: exactly nine alphanumerics, error code 3280 | Confirmed in Mistral's validator source and in two third-party error captures | T54, T64 |
| llama-server: the named-function `tool_choice` object is silently downgraded to `auto` | Not re-checked (local server, outside this topic). Meta's hosted API is different: it returns HTTP 400 for anything except `"auto"` | T56 |

Basis: `Workspace` for the left column, with the right column's basis per the cited items.

**T67. LC5 verdict.**

- LC5 says tool calling has three implementations behind one field name: native pass-through, transformation to provider format, and a YAML template injected into the prompt for providers with neither.
- The sentence is in today's docs word for word (T1), on `https://openrouter.ai/docs/api_reference/overview`, as a code comment above `tools?: Tool[]`.
- It is the only source. The Client Tools guide (T3), the Parameters page (T2), the spec, the changelog, and the provider integration guide do not mention the third path, and the routing page says a model with no tool-capable provider gets the parameter "ignored" (T5).
- So LC5 is confirmed as documented, with low confidence that the third path describes current behavior.

## Conflicts and disagreements

### Inside OpenRouter's own material

1. **Models with no tool support: YAML template or parameter ignored.** The API overview comment says "Otherwise, we transform the tools into a YAML template" (T1). The provider selection page says "If none of a model's providers support the parameter, the request is still routed to that model and the parameter is ignored" (T5). Both are live today. The second is prose in a maintained routing guide with a worked rule, and the first is a comment in a type listing that also lags the spec elsewhere (item 3). Neither can be confirmed without calling a model.
2. **Structured output on an unsupported model: error or ignored.** The structured outputs page says "The request will fail with an error indicating lack of support" (T34). The provider selection page lists the response format among parameters that are "ignored" when no provider supports them (T5). With `require_parameters: true` the request fails (T11). Which default applies is unresolved.
3. **`tool_choice: "required"`.** The Parameters page and the spec list it (T7). The API overview's `ToolChoice` type and the Responses tool calling table leave it out. The changelog is "generated from the OpenAPI specification on every release" (T-s23), which makes the spec the maintained artifact and the better source.
4. **Tool message content type.** The API overview type says `content: string` for `role: 'tool'`. The spec says a string or an array of content parts (T38). Same reasoning as item 3: the spec wins.
5. **`parallel_tool_calls` support.** The Client Tools guide says "default is true for most models" (T13). The catalog lists the parameter on 18 of 476 models and on no OpenAI or Anthropic model (T14). Not resolved.
6. **Streaming `index`.** The spec marks `ChatStreamToolCall.index` as required (T15). The official SDK provider handles deltas with "No index (allowed by the schema)" (T18), meaning its own response schema treats `index` as optional. The SDK reflects what arrives.
7. **Anthropic strict tools and the beta header.** OpenRouter says `strict` on tools is stripped unless the caller sends `x-anthropic-beta: structured-outputs-2025-11-13` (T21). Anthropic says "beta headers are no longer required" (T51). OpenRouter's page may be stale, or OpenRouter may still gate the field on the header. Not resolved.
8. **Mistral and `tool_choice: "none"`.** OpenRouter's endpoint flags for `mistralai/mistral-large-2512` say `none: false` (T9). Mistral's docs list `"none": prevents tool use` (T54). OpenRouter's flags come from its own tests (T8), so the flag may record a test failure and not a documented limit.
9. **Flags that contradict their own parameter list.** `amazon/nova-pro-v1` endpoints have all four `supports_tool_choice` flags true and no `tool_choice` in `supported_parameters`. Two `meta-llama/llama-3.3-70b-instruct` endpoints have all four flags true and no `tools` (T9). The flags look like defaults in those cases.

### Against the local Runvil documents

10. **LC5 (2026-10-28 survey).** Holds as a quote (T67). Two corrections to how the survey uses it. First, the survey's later sentence that "tool reliability cannot be inferred from `tools` appearing in a supported-parameters list" undersells the list: OpenRouter fills `tools` in from its own capability tests and ignores provider declarations (T4), and it publishes a per-endpoint Tool Call Error Rate (T46). Second, the survey presents the three implementations as current behavior, and the third rests on one code comment that conflicts with the routing guide (item 1).
11. **LC3 (unsupported parameters are silently ignored).** Outside this topic's assignment, but this file has two exceptions worth passing on: a forced `tool_choice` on Claude Fable 5.1, Opus 5.5, and Sonnet 5.5 is forwarded and returns the provider's 400 (T10), and `tool_choice` with deferred tools is rejected by OpenRouter itself with a 400 (T12).
12. **Workspace matrix, Kimi id rule.** The 2026-09-20 report says Kimi K3 and k2.7-code ids "must be `functions.name:idx`". Moonshot's own K3 encoder says the format requirement is dropped and ids are opaque (T57). The live source is right for K3. The rule may still hold for K2-family chat templates, which were not re-checked.
13. **Workspace matrix, Gemini rules.** The report's Gemini rows describe `generateContent`. Google's current function calling and thought signature pages describe the Interactions API, with a different shape (T52). The report is not shown wrong, but its Gemini rows can no longer be confirmed from Google's default documentation.
14. **Workspace Anthropic report (2026-11-17).** Confirmed on every point re-checked (six models reject forced choice, results first and immediately after, name pattern). One addition: Anthropic now says structured outputs need no beta header and are "available on every model" (T51).

## Open questions blocked by closed source

| # | Question | What was tried | Fill |
|---|---|---|---|
| 1 | Does the YAML-template path still run, for which models, and what does the injected prompt look like? | All 559 doc pages, the spec, the changelog, the catalog, the official AI SDK provider | LiteLLM `function_call_prompt` shows a JSON-instruction version of the same idea (T61). No OpenRouter template found |
| 2 | Are tool-call ids rewritten on the request path when history from one vendor is sent to another (for example `call_...` or `toolu_...` ids sent to Mistral, or ids with dots sent to Anthropic)? | Docs, spec, SDK source, doc examples (T16), third-party reports (T64) | LiteLLM sanitizes for Anthropic and Bedrock and leaves Mistral alone (T59). agentgateway synthesizes ids for Gemini (T62). Third-party reports suggest OpenRouter forwarded bad ids to Mistral (T64) |
| 3 | Does OpenRouter mint ids when the provider gives none, and guarantee they are unique? | Spec, SDK source, one stream capture | Gemini ids are minted as `tool_<name>_<random>` (T17). The official SDK re-mints empty or duplicate ids in the client, so uniqueness is not guaranteed by the server (T18) |
| 4 | Are tool and response schemas sanitized for each vendor's subset? | Docs (only the Anthropic `strict` strip is documented, T21), SDK source (passes schemas as written, T22) | agentgateway and LiteLLM normalize for Gemini tool parameters (T62). Anthropic's SDK recipe (T63). Gemini's server now strips the keywords it cannot enforce (T53) |
| 5 | Does routing read the per-value `supports_tool_choice` flags, so that `tool_choice: "required"` avoids endpoints where `required` is `false`? Does `require_parameters` consider them? | Provider selection page, compatibility page, spec, migration guides | No fill found. The Claude guides only cover models where every endpoint rejects the value |
| 6 | Is a forced `tool_choice` ever downgraded to `auto`? | Migration guides (provider 400 is returned, T10) | LiteLLM and agentgateway map values and do not downgrade (T60). DeepSeek itself treats `required` as `auto` when web tools are declared (T55) |
| 7 | How are tool results regrouped for Anthropic (one user message, results first), and what happens to a call without a result or to consecutive same-role messages? | Docs give only "system messages are concatenated ... user messages are merged" and the streaming-only debug echo (T39) | The 2026-11-17 workspace report, Table B1: LiteLLM and Bifrost group consecutive `tool` messages into one user message |
| 8 | Is `parallel_tool_calls: false` translated to Anthropic's `disable_parallel_tool_use` and to other vendors' switches? | Docs, spec | LiteLLM and agentgateway invert it into `disable_parallel_tool_use` (T60) |
| 9 | What does the caller receive when a tool call is cut off by the token limit: a partial call, or nothing? | Auto Exacto page, streaming page, SDK source (T28) | The official SDK drops unparseable calls in the client when the finish reason is `length`, which implies partial calls are delivered. The workspace report's failure 17 gives the same client rule |
| 10 | How is `response_format: json_schema` handled for a provider with no native schema mode ("translate your schema into their own structured-output format or treat it as a strong hint")? | Structured outputs page, compatibility page | agentgateway sends it as Gemini `responseJsonSchema` unchanged (T62). The workspace report maps it to Anthropic `output_config.format`. No fill for prompt-level hints |
| 11 | Which providers honor `response_format` types `grammar` and `python`? | Spec, broadcast page, catalog | No fill found |
| 12 | What replaces image parts in a tool result when the model is text-only? | Responses tool calling page (T43) | No fill found |
| 13 | Does response healing touch tool-call `arguments`, and does it validate against the schema? | Response healing page (T35) | No fill found. The page describes syntax repair of the message text only |

## Unverified

- Nothing in this file was tested against a live model. Every statement about what OpenRouter sends upstream comes from documentation, the spec, the catalog, SDK source, or third-party captures.
- Whether the debug echo (`debug.echo_upstream_body`) shows translated tool definitions, ids, and `tool_choice`. The docs show it for messages and parameters only. It would answer open questions 2, 4, 6, 7, and 8 in one streamed request per vendor.
- The date of the Mistral error capture in T64 (T-s46). The page is undated.
- That the `tool_<name>_<random>` id shape for Gemini (T17) is still current. The capture is from 2025-12-18.
- Whether the server accepts a per-tool `eager_input_streaming` field. The official SDK sends it and the spec does not list it (T26).
- What the `strict_tool_schemas` parameter flag means. It is in the spec enum and on no model (T6).
- Vendor rules not re-checked live in this pass: xAI, Cohere, Qwen, GLM, and MiniMax tool rules. OpenAI's "one `tool` message per call immediately following" rule for Chat Completions. Anthropic's id pattern and root-object rule (taken from the 2026-11-17 workspace report). Gemini's `generateContent` function calling rules.
- From memory, not sourced: older Gemini `generateContent` endpoints rejected `additionalProperties` and `$ref` in function declarations with a 400. The agentgateway comment in T62 says the same ("`additionalProperties` is rejected"), which is a fill and not a vendor statement.

## Workspace documents used

- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20). Took Table 2 and Appendix B (replay and id rules per vendor) as the starting matrix. Re-checked live: Anthropic, DeepSeek, Mistral, and Kimi rows (Kimi K3 corrected). Not re-checked: Gemini `generateContent` rows, OpenAI Responses, Qwen and GLM, and Appendix C (llama-server).
- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took the LC5 wording and its context (lines 138 and 339). Re-checked live: LC5 confirmed as a quote, with the caveats in conflict 10.
- "Anthropic Messages API harness survey", a finished workspace report dated 2026-11-17. Took Appendix A (name and id patterns, tool result rules), Table B1 (chat-completions to Messages translation rules and which translator sets each precedent), and Table C1 (failure 10, 17, 21). Re-checked live: forced `tool_choice` models, tool result ordering, tool name pattern. Not re-checked: the id pattern and the root-object rule.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| T-s1 | https://openrouter.ai/docs/guides/features/tool-calling | 2026-11-22 | OpenRouter doc |
| T-s2 | https://openrouter.ai/docs/api_reference/responses/tool-calling | 2026-11-22 | OpenRouter doc |
| T-s3 | https://openrouter.ai/docs/guides/features/structured-outputs | 2026-11-22 | OpenRouter doc |
| T-s4 | https://openrouter.ai/docs/guides/features/plugins/response-healing | 2026-11-22 | OpenRouter doc |
| T-s5 | https://openrouter.ai/docs/guides/routing/auto-exacto | 2026-11-22 | OpenRouter doc |
| T-s6 | https://openrouter.ai/docs/guides/routing/model-variants/exacto | 2026-11-22 | OpenRouter doc |
| T-s7 | https://openrouter.ai/docs/guides/providers/integration | 2026-11-22 | OpenRouter doc |
| T-s8 | https://openrouter.ai/docs/api_reference/parameters | 2026-11-22 | OpenRouter doc |
| T-s9 | https://openrouter.ai/docs/guides/features/compatibility | 2026-11-22 | OpenRouter doc |
| T-s10 | https://openrouter.ai/docs/api_reference/overview | 2026-11-22 | OpenRouter doc |
| T-s11 | https://openrouter.ai/openapi.json (schemas `ChatRequest`, `ChatFunctionTool`, `ChatToolChoice`, `ChatNamedToolChoice`, `ChatServerToolChoice`, `ChatToolCall`, `ChatStreamToolCall`, `ChatToolMessage`, `ChatAssistantMessage`, `ChatStreamDelta`, `ChatMessages`, `ChatSystemMessage`, `ChatJsonSchemaConfig`, `ChatFormatGrammarConfig`, `ChatFormatPythonConfig`, `ToolChoiceSupport`, `ToolChoiceAllowed`, `ToolChoiceMcp`, `OpenAIResponsesToolChoice`, `PublicEndpoint`, `Parameter`, `FunctionTool`, `CustomTool`, `CustomToolCallItem`, `FunctionCallItem`, `FunctionCallOutputItem`, `FormatJsonSchemaConfig`, `ResponseHealingPlugin`, `ResponsesRequest`, `MessagesRequest`) | 2026-11-22 | OpenRouter spec |
| T-s12 | https://openrouter.ai/api/v1/models (476 models) | 2026-11-22 | OpenRouter spec (public catalog data) |
| T-s13 | https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints for 19 models: `anthropic/claude-fable-5.2`, `anthropic/claude-opus-5.5`, `anthropic/claude-sonnet-5.5`, `anthropic/claude-sonnet-4.5`, `openai/gpt-5.5`, `openai/gpt-6.2-sol`, `google/gemini-3.5-flash`, `x-ai/grok-4.7`, `mistralai/mistral-large-2512`, `meta-llama/llama-4-maverick`, `meta-llama/llama-3.3-70b-instruct`, `deepseek/deepseek-v4-pro`, `deepseek/deepseek-v4.1-flash`, `moonshotai/kimi-k3`, `moonshotai/kimi-k2.6`, `z-ai/glm-5.3`, `qwen/qwen3.7-flash`, `minimax/minimax-m2.7`, `amazon/nova-pro-v1` (unauthenticated) | 2026-11-22 | OpenRouter spec (public catalog data) |
| T-s14 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| T-s15 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5 | 2026-11-22 | OpenRouter doc |
| T-s16 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/fable-5-1 | 2026-11-22 | OpenRouter doc |
| T-s17 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-6 | 2026-11-22 | OpenRouter doc |
| T-s18 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-5-4 | 2026-11-22 | OpenRouter doc |
| T-s19 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/opus-5 | 2026-11-22 | OpenRouter doc |
| T-s20 | https://openrouter.ai/docs/api_reference/errors-and-debugging | 2026-11-22 | OpenRouter doc |
| T-s21 | https://openrouter.ai/docs/api_reference/streaming | 2026-11-22 | OpenRouter doc |
| T-s22 | https://openrouter.ai/docs/guides/features/plugins/alignment | 2026-11-22 | OpenRouter doc |
| T-s23 | https://openrouter.ai/docs/changelog (entries of 2026-07-07, 2026-08-19, 2026-08-28, 2026-09-11, 2026-09-23, 2026-10-01, 2026-11-05) | 2026-11-22 | OpenRouter doc |
| T-s24 | https://openrouter.ai/docs/guides/features/broadcast | 2026-11-22 | OpenRouter doc |
| T-s25 | https://openrouter.ai/docs/guides/features/server-tools/tool-search | 2026-11-22 | OpenRouter doc |
| T-s26 | https://openrouter.ai/docs/guides/features/plugins/armor | 2026-11-22 | OpenRouter doc |
| T-s27 | `OpenRouterTeam/ai-sdk-provider` at `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20): `src/chat/index.ts`, `src/chat/convert-to-openrouter-chat-messages.ts`, `src/chat/get-tool-choice.ts`, `src/types/openrouter-chat-settings.ts` | 2026-11-22 | OpenRouter SDK source |
| T-s28 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use | 2026-11-22 | vendor doc (Anthropic) |
| T-s29 | https://platform.claude.com/docs/en/build-with-claude/structured-outputs | 2026-11-22 | vendor doc (Anthropic) |
| T-s30 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls | 2026-11-22 | vendor doc (Anthropic) |
| T-s31 | https://ai.google.dev/gemini-api/docs/function-calling | 2026-11-22 | vendor doc (Google) |
| T-s32 | https://ai.google.dev/gemini-api/docs/structured-output | 2026-11-22 | vendor doc (Google) |
| T-s33 | https://docs.mistral.ai/capabilities/function_calling/ | 2026-11-22 | vendor doc (Mistral) |
| T-s34 | `mistralai/mistral-common` at blob `7edf6f65`: `src/mistral_common/protocol/instruct/validator.py` (https://github.com/mistralai/mistral-common/blob/7edf6f65/src/mistral_common/protocol/instruct/validator.py) | 2026-11-22 | vendor doc (Mistral, vendor source) |
| T-s35 | https://api-docs.deepseek.com/guides/tool_calls | 2026-11-22 | vendor doc (DeepSeek) |
| T-s36 | https://api-docs.deepseek.com/guides/thinking_mode | 2026-11-22 | vendor doc (DeepSeek) |
| T-s37 | https://api-docs.deepseek.com/api/create-chat-completion | 2026-11-22 | vendor doc (DeepSeek) |
| T-s38 | https://llama.developer.meta.com/docs/features/tool-calling | 2026-11-22 | vendor doc (Meta) |
| T-s39 | https://developers.openai.com/api/docs/guides/function-calling | 2026-11-22 | vendor doc (OpenAI) |
| T-s40 | https://github.com/karthink/gptel/issues/1190 (opened 2025-12-18) | 2026-11-22 | issue report (third-party) |
| T-s41 | https://developers.openai.com/api/docs/guides/structured-outputs | 2026-11-22 | vendor doc (OpenAI) |
| T-s42 | https://huggingface.co/moonshotai/Kimi-K3/blob/main/encoding_k3.py | 2026-11-22 | vendor doc (Moonshot, vendor source) |
| T-s43 | https://platform.kimi.ai/docs/guide/use-official-tools | 2026-11-22 | vendor doc (Moonshot) |
| T-s44 | `BerriAI/litellm` at `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22): `litellm/litellm_core_utils/prompt_templates/factory.py`, `litellm/llms/anthropic/common_utils.py`, `litellm/llms/anthropic/chat/transformation.py`, `litellm/llms/mistral/chat/transformation.py`, `litellm/llms/vertex_ai/common_utils.py`, `litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py`, `litellm/utils.py` | 2026-11-22 | gateway source |
| T-s45 | `agentgateway/agentgateway` at `888badbbe91f2a57ec8e7bf5a5ef3acda34b1fe5` (2026-11-20): `crates/llm/src/conversion/messages.rs`, `crates/llm/src/conversion/vertex_gemini.rs` | 2026-11-22 | gateway source |
| T-s46 | https://huggingface.co/spaces/arterm-sedov/cmw-copilot/blob/main/docs/MISTRAL_TOOL_CALL_ID_FIX.md (undated) | 2026-11-22 | blog (third-party) |
| T-s47 | https://github.com/zed-industries/zed/issues/53034 (opened 2026-04-03, read through a search result excerpt) | 2026-11-22 | issue report (third-party) |
| T-s48 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20) | 2026-11-22 | workspace document |
| T-s49 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28) | 2026-11-22 | workspace document |
| T-s50 | "Anthropic Messages API harness survey", finished workspace report (2026-11-17) | 2026-11-22 | workspace document |
| T-s51 | https://ai.google.dev/gemini-api/docs/thought-signatures (read as `.md.txt`) | 2026-11-22 | vendor doc (Google) |
| T-s52 | https://platform.kimi.ai/docs/guide/kimi-k3-quickstart | 2026-11-22 | vendor doc (Moonshot) |

The repositories in T-s27, T-s44, and T-s45 were read from shallow clones made on 2026-11-22 by other research subagents in the shared scratch area. Commit hashes and commit dates were read with `git rev-parse HEAD` and `git log -1`.

