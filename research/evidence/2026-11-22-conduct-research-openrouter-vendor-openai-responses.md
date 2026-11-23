---
produced: 2026-11-22
title: OpenRouter deep dive - OpenAI vendor-native features and the Responses API (previous_response_id, store, encrypted reasoning items, phase, configuration_update, async tools, built-in tools, background, prompt caching, service tiers, errors) and how OpenRouter exposes or flattens each
---

# OpenRouter deep dive - OpenAI vendor-native features and the Responses API (previous_response_id, store, encrypted reasoning items, phase, configuration_update, async tools, built-in tools, background, prompt caching, service tiers, errors) and how OpenRouter exposes or flattens each

- What was read: OpenAI's live documentation in its Markdown form (the Responses and Chat Completions create references, 66 guides downloaded and searched with about 30 read closely, the changelog), OpenRouter's documentation mirror (the six pages under the Responses API reference, the GPT-5.4, GPT-5.6 and GPT-6 migration guides, reasoning tokens, prompt caching, service tiers, errors, server tools, the Codex guides, the changelog), OpenRouter's OpenAPI spec and model catalog, the official OpenRouter AI SDK provider source, and five open-source projects at pinned commits (Codex, LiteLLM, Vercel AI SDK, llama.cpp, vLLM) plus LM Studio's documentation.
- How many sources: 68 sources (30 OpenRouter, 29 OpenAI, 6 fills, 3 workspace documents), 85 evidence items, 44 candidate matrix rows.
- Read date: 2026-11-22 for every source. Nothing was run against a live model and no credential was used.

## Candidate matrix rows

Basis shorthand in this table: VD = `Vendor-documented`, ORD = `OR-documented`, ORI = `OR-inferred`, F = `Filled` (project named). "Chat route" means OpenRouter's `POST /api/v1/chat/completions`. "Responses route" means OpenRouter's `POST /api/v1/responses`.

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
| - | - | - | - | - | - | - | - |
| Server-held conversation state: `previous_response_id`, `store` | OpenAI | `store` defaults to `true` on Responses, data kept "for at least 30 days". `previous_response_id` chains to a stored response. All earlier input tokens are still billed. | Responses route: "Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error". Spec: `store` is `const: false`. Chat route: neither field is in `ChatRequest`. | The whole stateful mode. The caller replays the full item list every turn. | State holder on OpenAI: the server. On OpenRouter: the caller. A chat-completions superset can hold this only if the gateway keeps a store. The obstacle is server-held state, and stateless replay is the documented substitute. | VD; ORD | OA1, OA24, OA4 |
| Conversations API: `conversation` | OpenAI | A durable conversation object whose items are kept until deleted. "Cannot be used in conjunction with" `previous_response_id`. | The spec's Responses surface is one path, `POST /responses`, and `conversation` is outside its request schema (the accepted fields are in OA4). | The durable conversation object. | Server-held state on a second resource. Route: Responses only, OpenAI only. | VD; ORD (spec) | OA24, OA2, OA4 |
| Encrypted reasoning items for stateless replay: `reasoning` item with `encrypted_content` | OpenAI, Azure OpenAI, Bedrock OpenAI | `encrypted_content` "is populated by default for reasoning items returned by `POST /v1/responses`". The legacy `include: ["reasoning.encrypted_content"]` is accepted and not needed. Use the item from `response.output_item.done`. | Responses route: reasoning item in `output` with `encrypted_content`, plus OpenRouter fields `format` (`openai-responses-v1`), `signature`, `content`. "OpenRouter automatically requests encrypted reasoning content". Chat route: `message.reasoning_details[]` entries `reasoning.summary` and `reasoning.encrypted` (`data`, `id` of form `rs_...`, `format: "openai-responses-v1"`). Messages route: `thinking` plus `redacted_thinking.data`. | On the chat route the item becomes detail entries on one message, so its position among several output items is not on the wire. The blob in every documentation example ends with a dot and a base64 JSON tag naming the endpoint, so the stored value differs from OpenAI's own. | The caller holds the blob and returns it unchanged. This is the opaque-field case: stored unread, returned in order. The caller must not edit, reorder, or trim the sequence. | VD; ORD; ORI (endpoint tag) | OA25, OA14, OA16, OA13, OA15, OA17, OA18 |
| Reasoning next to function calls, and when OpenAI drops reasoning | OpenAI | "pass back the reasoning items returned with the function calls along with the tool outputs", in original order. The API removes reasoning on a model-family switch, a downgrade, an organization or user change, and from GPT-6.2 on a prefix edit or a `safety_identifier` change. `input_transformations` reports each removal with a `reason`. | Responses route result has `input_transformations` ("Present only when the provider reported the field"). The field exists on the Responses and Messages results only. Chat rule: "the entire sequence of consecutive reasoning blocks must match the outputs generated by the model". | A chat caller is not told when OpenAI dropped replayed reasoning. | Round trip must be byte-stable and the history before it must be append-only on GPT-6.2 and later. A gateway that rewrites earlier tool results or system text makes OpenAI drop the reasoning. The organization check means a blob made under one upstream key is dropped under another. | VD; ORD | OA26, OA27, OA21, OA18, OA19 |
| Reasoning summaries: `reasoning.summary` | OpenAI | Opt-in: `summary` is `auto`, `concise` or `detailed`. Raw reasoning text is not returned. | `reasoning.summary` accepted on both routes (`ReasoningSummaryVerbosity`: `auto`, `concise`, `detailed`). Chat route returns summary text in `message.reasoning` and as `reasoning.summary` details. | Nothing documented as lost. `reasoning.exclude` hides it on the chat route only. | Neutral setting is possible (show or hide). Summary deltas all arrive with `index: 0`, so a client must merge by type change. | VD; ORD | OA31, OA7, OA20, OA13, OA33 |
| `reasoning.effort` | OpenAI | Values "`none`, `minimal`, `low`, `medium`, `high`, `xhigh`, and `max`", model-dependent. GPT-6 Astra, GPT-6.1 Sol and GPT-6.2 reject `none` with HTTP 400. | `reasoning.effort` enum `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none` on both routes, plus chat shorthand `reasoning_effort`. Catalog field `reasoning.supported_efforts`, `default_effort`, `mandatory` per model. Mid-conversation updates with an unsupported effort get a 400. | Request-level effort mapping to a model's nearest level is documented for Gemini. For OpenAI the page states only the per-model list. | A neutral setting translates. The caller or gateway needs the per-model list to avoid a 400 (for example "thinking off" on GPT-6 Astra). | VD; ORD | OA29, OA33, OA34 |
| `reasoning.mode` (pro) | OpenAI (GPT-5.6 and later) | `standard` or `pro`, "in the Responses API". Independent of effort. | `reasoning.mode: "pro"` on both routes "reroutes the request to the matching `*-pro` model listing". Stripped for providers that lack it. Honored by OpenAI and Azure, ignored by Bedrock, so pro requests route only to the first two. | On OpenAI this is one model with a mode. On OpenRouter it is a second model listing. | Vendor-specific value. A role-based caller can map a role to the `-pro` slug and never send the field. | VD; ORD | OA30, OA33, OA79 |
| `reasoning.context` | OpenAI (GPT-5.6 and later) | `auto`, `current_turn`, `all_turns`. GPT-5.6 defaults to `all_turns`, earlier models to `current_turn`. Reasoning is reusable only inside one model family. | `reasoning.context` on both routes with the same three values. "only supported by OpenAI GPT-5.6 and newer". | Nothing documented as lost. | Vendor-specific value with a safe default (omit it). It only matters when the caller replays reasoning items. | VD; ORD | OA28, OA33 |
| `verbosity` | OpenAI | Responses: `text.verbosity` `low`, `medium`, `high`. Chat Completions: top-level `verbosity`. | Parameters page: key `verbosity`, enum "low, medium, high, xhigh, max", mapped to `output_config.effort` on Anthropic. Spec: `verbosity` exists only as `text.verbosity` on the Responses route (`low`, `medium`, `high`). Catalog lists `verbosity` as a supported parameter on 114 models. | One knob on OpenRouter stands for two different vendor knobs (OpenAI output length, Anthropic effort). | Neutral value translates, but the meaning differs per vendor. | VD; ORD | OA32, OA33 |
| Chat Completions versus Responses on OpenAI today | OpenAI | "Starting with GPT-5.4, Chat Completions does not support tool calling with `reasoning_effort` values other than `none`." GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra and GPT-6.2: "tool calling requires Responses". "Passing CoT exists only in the Responses API". | OpenRouter's catalog lists `tools` and `tool_choice` as supported parameters for `openai/gpt-6-astra` and lists `/api/v1/chat/completions` among its endpoints. The GPT-6 guide says a chat-route `configuration_update` is translated "into the Responses `configuration_update` item before forwarding to the model". | The chat route hides which upstream API is used. Reasoning continuity between tool calls then depends on the caller returning `reasoning_details`. | An OpenAI-direct translator that forwards chat completions as written loses tool calling with reasoning on GPT-5.4 and later, and loses tool calling on GPT-6. The translator must speak Responses upstream for these models. | VD; ORD; ORI | OA35, OA36, OA37, OA9, OA10, OA11, OA12 |
| `phase` on assistant messages | OpenAI (GPT-5.3 Codex, GPT-5.4 and later) | `phase` is `commentary` or `final_answer` on assistant messages. "preserve and resend phase on all assistant messages"; dropping it "can degrade performance". | Responses route only. "`phase` is **not available** in the Chat Completions API. The Chat Completions format cannot represent multiple output items with distinct phases in a single response." In the spec `phase` is a property of `OutputMessage` and `EasyInputMessage` only. | On the chat route the phase labels and the split into several assistant messages. | Round trip needed, per assistant message. The obstacle on chat completions is an item shape: one response can hold several assistant messages with different phases, and one chat choice holds one message. | VD; ORD | OA40, OA41 |
| Async tool calling: `async: true` | OpenAI (GPT-6 family) | `async: true` on a `function` or `custom` tool. The call item has `async: true`, the model keeps working, and the output arrives in a later request with the same `call_id`. | Responses route only. Chat route: "the flag is removed before the request is routed and the call behaves like any other tool call". Routing narrows to OpenAI and Azure endpoints; Bedrock runs it synchronously. | On the chat route the whole feature. On the Responses route the `previous_response_id` continuation, replaced by replay. | The caller holds the pending call and delivers the result later. Chat completions requires a tool message right after the assistant turn, so a pending call followed by more turns has no chat equivalent. | VD; ORD | OA42, OA43 |
| Mid-conversation effort: `configuration_update` | OpenAI (GPT-6, GPT-6.1), also Claude via OpenRouter | An input item `{"type": "configuration_update", "reasoning": {"effort": ...}}` (also `text.format`). Two adjacent updates are rejected. Rejected with `truncation: "auto"` and with pro mode. | Three forms: Responses item, chat-route extension field `configuration_update` on a content-less `system` or `developer` message, and Messages-route `output_config.effort` on a content-less system message. OpenRouter "normalizes each into one internal representation". Unsupported model: 400. | The `text` (format) half of OpenAI's item: OpenRouter's schema requires `reasoning` and allows only `effort`. | The caller holds the update at a fixed position in history. A neutral form exists (OpenRouter invented one for chat). | VD; ORD | OA44, OA45 |
| Compaction: `context_management`, `compaction_trigger`, `compaction` item, `/responses/compact` | OpenAI | `context_management: [{type: "compaction", compact_threshold}]` runs server-side compaction and emits an encrypted `compaction` item. A `compaction_trigger` input item compacts now. A standalone `/responses/compact` endpoint exists. | Responses route accepts input items `compaction_trigger`, `compaction`, `context_compaction` and returns `compaction` output items. In the spec `context_management` belongs to the Messages request only, and the Responses surface is the single create path. OpenRouter's own overflow tool is the `context-compression` plugin. | The automatic threshold form and the standalone endpoint. | The compaction item is an opaque blob the caller stores and returns. Route: Responses only. | VD; ORD (spec) | OA46, OA47 |
| `truncation` | OpenAI | `auto` drops items from the start of the conversation, `disabled` (default) fails with a 400 on overflow. | Responses route accepts `auto`, `disabled`. Rejected together with a `configuration_update`. | Not documented for non-OpenAI models. | Neutral on or off value. Responses only. | VD; ORD | OA48, OA47 |
| Built-in tool: web search | OpenAI | Responses tool `web_search`, item `web_search_call`, `url_citation` annotations. Chat Completions only through search models such as `gpt-5-search-api`. | Responses route accepts `web_search`, `web_search_preview`, `web_search_2025_08_26` and `openrouter:web_search`. Chat route schema `ChatWebSearchShorthand`: "Web search tool using OpenAI Responses API syntax. Automatically converted to openrouter:web_search." Engine `auto` uses native provider search when present. | OpenAI-only controls depend on the native engine. | No state. A neutral tool name translates. | VD; ORD | OA49, OA50, OA53 |
| Built-in tool: file search | OpenAI | Tool `file_search` with `vector_store_ids`, item `file_search_call`. | Responses route schema `FileSearchServerTool`: "requires BYOK (your own OpenAI key); requests on OpenRouter credits get a 400". | Vector stores belong to an OpenAI account, so the feature exists only with the caller's own OpenAI key. | Server-held resource at the vendor. The caller must know it is OpenAI and hold an OpenAI key. | VD; ORD | OA49, OA51 |
| Built-in tool: code interpreter | OpenAI | Tool `code_interpreter` with `container`, item `code_interpreter_call`. | Responses route: "An existing container id or `container.file_ids` requires BYOK (your own OpenAI key); requests on OpenRouter credits must use `auto` with no `file_ids`." | Reuse of a named container without BYOK. | Server-held container at the vendor. | VD; ORD | OA49, OA51 |
| Built-in tool: computer use | OpenAI | Tools `computer` and `computer_use_preview`, items `computer_call`, `computer_call_output`. | Responses route schema lists `computer_use_preview` and item `computer_call`. The newer `computer` tool type is not in the tool union read today. | The newer tool type, as far as the spec shows. | The caller executes actions and returns screenshots. Responses only. | VD; ORD (spec) | OA49, OA50 |
| Built-in tool: image generation | OpenAI | Tool `image_generation`, item `image_generation_call`. | Responses route accepts `image_generation` and OpenRouter's own `openrouter:image_generation` (an inner image-model generation). | Not documented which engine serves the OpenAI-shaped tool on non-OpenAI models. | No caller state. | VD; ORD | OA49, OA50 |
| Built-in tool: MCP connector | OpenAI | Tool `mcp` (`server_label`, `server_url`, `connector_id`, `require_approval`), items `mcp_list_tools`, `mcp_call`, `mcp_approval_request`, `mcp_approval_response`. | Responses route accepts `mcp` and `openrouter:mcp`, and the four item types. | Not documented which side runs the connector for non-OpenAI models. | Approval items must make a round trip through the caller. Responses only. | VD; ORD (spec) | OA49, OA50 |
| Built-in tool: shell and local shell | OpenAI | Shell is offered through the Responses API: "It's not available via the Chat Completions API." Hosted containers or a local runtime. Items `shell_call`, `shell_call_output`. | Responses route: "On OpenAI models this uses OpenAI's own hosted shell; on any other model, OpenRouter routes the call to its sandbox transparently." `openrouter:shell` works on the Responses and Messages routes. | Local shell mode on OpenRouter's own tool ("Only container-backed environments are supported"). | Hosted state (a container) at the vendor or at OpenRouter. Route: Responses and Messages, not chat. | VD; ORD | OA49, OA52 |
| Built-in tool: apply patch | OpenAI | Tool `apply_patch`, items `apply_patch_call`, `apply_patch_call_output`. The application applies the patch. | Responses route only (`openrouter:apply_patch` "Responses API only"). Native passthrough "Currently supported on OpenAI endpoints", otherwise an OpenRouter validator buffers the diff. | Incremental diff streaming on non-OpenAI endpoints. | Caller executes. Item type with no chat equivalent. | VD; ORD | OA49, OA52 |
| Tool search, namespaces, `defer_loading` | OpenAI (GPT-5.4 and later) | Tool `tool_search` (hosted or client-executed), `namespace` tools, `defer_loading: true`, items `tool_search_call`, `tool_search_output`, `additional_tools`. | Responses route accepts `namespace`, `additional_tools`, `tool_search_call`, `tool_search_output` and `openrouter:tool_search`. OpenRouter keeps deferred-tool state "inside the encrypted reasoning it returns". | Covered in depth by the tool-search evidence file. | OpenRouter's own state rides in the same opaque reasoning field the caller already returns. | VD; ORD | OA49, OA22, OA56 |
| Custom (freeform) tools and grammars | OpenAI | Tool `{"type": "custom", "name", "format"}` with `lark` or `regex` grammars. Item `custom_tool_call` with a plain-text `input`. OpenAI's Chat Completions also has `ChatCompletionCustomTool`. | Responses route accepts `custom` and `custom_tool_call`. Chat route tool union is `function` plus OpenRouter server tools, and `ChatToolCall.type` is `function` only. | On the chat route: custom tools and grammars. | A missing tool type on the chat route. Plain-text tool input needs a second tool-call shape in a chat superset. | VD; ORD (spec) | OA54, OA50, OA53 |
| Programmatic tool calling | OpenAI | Tool `programmatic_tool_calling`. Items `program` (JavaScript `code`, and a `fingerprint` that is an "Opaque program replay fingerprint that must be round-tripped") and `program_output`. | Not in OpenRouter's Responses tool union or input items read today. | The feature. | Another opaque value that must make a round trip. | VD; ORD (spec) | OA55, OA50 |
| Background runs, polling, cancel, webhooks | OpenAI | `background: true` returns an id. Poll with retrieve, cancel with the cancel endpoint, or receive `response.completed` by webhook. `background_options.completion_window` `2h` or `8h` gives 30 or 40 percent off. Background mode stores data even with `store: false`. | `background` is a boolean in `ResponsesRequest` whose only documentation is its type, responses echo `"background": false`, and the spec's Responses surface is the single create path. Polling and cancel are described only on OpenAI's side. | The whole long-running job mode. OpenRouter's separate Batch API has a `callback_url` webhook. | The obstacle is a long-running job with server-held state. It needs a retrieve route, so it lives outside a streaming chat route. | VD; ORD (spec); ORI | OA57, OA58, OA2 |
| Prompt caching: automatic prefix caching and `prompt_cache_key` | OpenAI | Automatic above 1,024 tokens. `prompt_cache_key` helps routing on models before GPT-5.6 and only separates accounting on GPT-5.6 and later. | Automatic, "does not require any additional configuration". `prompt_cache_key` is accepted on both routes and is the fallback sticky-routing key when `session_id` is absent. | Nothing documented as lost. | No caller state. The gateway must keep one upstream endpoint per conversation. | VD; ORD | OA61, OA63 |
| Prompt caching: explicit breakpoints, TTL, retention | OpenAI (GPT-5.6 and later) | `prompt_cache_breakpoint: {mode: "explicit", ttl}` on a content block, `prompt_cache_options: {mode, ttl, prewarm, comparison_response_id}`. `ttl` `12h` exists in the Responses API only; Chat Completions supports `30m` only. `prompt_cache_retention` is deprecated. | `prompt_cache_breakpoint` and `prompt_cache_options` (`mode`, `ttl`) on both routes, with `12h` documented on the chat route too. Converted to and from Anthropic-style `cache_control`. "TTLs are not translated". Stripped for providers that lack them. | `prewarm`, `comparison_response_id`, `prompt_cache_diagnostics`, `prompt_cache_retention`. TTLs across vendors. | Caller-placed markers with a neutral block-level form. Request-level options stay OpenAI-only. | VD; ORD | OA61, OA63, OA64, OA12 |
| Cached-token reporting | OpenAI | Responses: `usage.input_tokens_details.cached_tokens`, `cache_write_tokens`, `cache_write_tokens_details.ttl_30m_tokens` and `ttl_12h_tokens`. Chat Completions: `usage.prompt_tokens_details.cached_tokens` and `cache_write_tokens`. | Same first two fields on both routes. | The per-TTL write breakdown is not in OpenRouter's usage schemas. | Report only. `cache_write_tokens` is now OpenAI's own chat field name too. | VD; ORD | OA61, OA65, OA63, OA64 |
| Top-level `instructions` and the system prompt | OpenAI | From GPT-6.2, "append top-level `instructions` after `input`" and bill them uncached. Reusable instructions belong in a first `developer` message. | Responses route accepts `instructions`. How a chat-route system message is placed upstream is not documented. | Unknown. | A translator that maps the chat system message to `instructions` loses the cache on GPT-6.2 and later. Gateway-only decision. | VD; ORI (gap) | OA62, OA80 |
| Service tiers: `service_tier` | OpenAI | `auto`, `default`, `flex`, `scale`, `priority`, `fast`, `ultrafast`. The response reports the tier used, and reports `priority` when the request said `fast`. Flex is being wound down and is absent on GPT-6.2. | Top-level `service_tier` on all three routes with `auto`, `default`, `fast`, `flex`, `priority`, `scale`, `ultrafast`. Each tier is a separate endpoint (`openai/flex`, `openai/fast`, `openai/ultrafast`). The response `service_tier` reports the tier that served. Also `:nitro` and `:floor` variants and a workspace default. | OpenAI's `auto` project setting is replaced by OpenRouter routing. A flex request can fall back to another endpoint at or below the flex price. | Neutral values exist and OpenRouter says they are not OpenAI-specific. No caller state. | VD; ORD | OA66, OA67 |
| Predicted outputs: `prediction` | OpenAI | Chat Completions only, on `gpt-4o`, `gpt-4o-mini`, `gpt-4.1` families. Usage reports `accepted_prediction_tokens` and `rejected_prediction_tokens`. | `prediction` on `ChatRequest` only. Catalog lists it for nine models. `ChatUsage` has both token fields. | Nothing documented as lost. | Chat-route field, usable only on a few older models. No state. | VD; ORD | OA68, OA69 |
| Structured outputs and `refusal` | OpenAI | Chat: `response_format` with `json_schema`, refusal in `message.refusal`. Responses: `text.format`, refusal as a `refusal` content part. | Chat: `response_format`, `message.refusal` with `finish_reason: "content_filter"`. Responses: `text.format`, `status: "incomplete"` with `incomplete_details.reason: "content_filter"` and a `refusal` part. | OpenRouter fixes the chat `finish_reason` for a model-written refusal at `content_filter`. Which value OpenAI's own chat route sends was not found in the pages read today (see Unverified). | No state. Neutral. | VD; ORD | OA70, OA71 |
| Strict function schemas | OpenAI | Responses attempts strict when `strict` is omitted. "Chat Completions requests remain non-strict by default." | `strict` on function tools on both routes. "Enforcement varies by provider". | A guarantee on non-OpenAI endpoints. | The default differs by upstream API, so a translator that moves chat requests to Responses must send `strict: false` to keep chat behavior. | VD; ORD | OA39, OA71 |
| Response status and `incomplete_details.reason` | OpenAI | `status`: `completed`, `failed`, `in_progress`, `cancelled`, `queued`, `incomplete`. `incomplete_details.reason`: `max_output_tokens`, `max_messages`, `content_filter`, `steered`. | Responses route: same six `status` values in the spec; `IncompleteDetails.reason` enum is `max_output_tokens`, `content_filter`. Upstream length errors become `status: "incomplete"`. Chat route: `finish_reason` `length`. | `max_messages` and `steered`. | Maps to chat `finish_reason` except the two extra reasons. | VD; ORD | OA72, OA76, OA75 |
| Errors: context overflow, spend limits, policy blocks, in-stream failure | OpenAI | Overflow code `context_length_exceeded` (confirmed in Codex source, not in the public error guide). Spend codes `credit_balance_exhausted`, `organization_spend_limit_exceeded`, `project_spend_limit_exceeded`, `organization_usage_limit_exceeded`, with `error.type` possibly `insufficient_quota`. In-stream: `response.failed` and a plain `error` event. | Responses route collapses to five codes, with the exact type in top-level `error_type`. `context_length_exceeded` shows as `invalid_prompt`. Failed streams end with `response.failed`. `bio_policy` and `cyber_policy` become HTTP 403 with `error_type: "refusal"`. | The native `error.code` vocabulary on the Responses route. | The gateway must normalize. A caller reads `error_type`. | F (Codex); VD; ORD | OA73, OA74, OA75 |
| `safety_identifier` and `user` | OpenAI | `safety_identifier`, max 128 characters. `user` "is being replaced by `safety_identifier` and `prompt_cache_key`". From GPT-6.2 a changed identifier makes the API drop replayed reasoning. | Responses route: `safety_identifier`, folded "into the hashed identity sent upstream" and never forwarded raw. Chat route: `user` does that job. | The raw value. | Neutral end-user id. It must stay the same across a conversation on GPT-6.2 and later. | VD; ORD | OA77, OA78, OA27 |
| `metadata` | OpenAI | 16 pairs, keys up to 64 characters, values up to 512. | Same limits on both routes. | Not documented whether it reaches OpenAI. | None. | VD; ORD | OA77, OA78 |
| `logprobs` | OpenAI | Chat: `logprobs`, `top_logprobs`. Responses: `include: ["message.output_text.logprobs"]`, `top_logprobs`. Unsupported with reasoning on GPT-6. | Chat: `logprobs`, `top_logprobs`. Responses: `top_logprobs` only; the include enum's eight values (OA4) cover search, images, code and reasoning. | The Responses include value. | Neutral. | VD; ORD | OA77, OA78 |
| Batch API | OpenAI | A `.jsonl` file of requests, endpoints such as `/v1/responses` and `/v1/chat/completions`, 50 percent off, 24 hours. | OpenRouter's own Batch API: JSON body with `endpoint`, `model`, `requests`, `callback_url`, `completion_window: "24h"`, and `:batch` model variants. | OpenAI's file-based flow. | A separate route with a job id. | VD; ORD | OA59, OA60 |
| File inputs | OpenAI | Responses `input_file` accepts many file types by `file_data`, `file_id`, `file_url`. "Chat Completions accepts only PDF files as `file` content parts." | Covered by the multimodal evidence file. OpenRouter has its own `/files` routes and a file-parser plugin. | Not examined here. | Not examined here. | VD | OA77 |
| Provider differences for one OpenAI model | OpenAI direct, Azure, Bedrock | Not applicable. | Format tags `openai-responses-v1`, `azure-openai-responses-v1`, `bedrock-openai-responses-v1`. Pro mode and async tools: OpenAI and Azure only. ZDR for the OpenAI group "Removes first-party OpenAI endpoints (Azure remains available)". `access_programs` other than `standard`: BYOK only. | A reasoning blob minted by one provider is labeled for that provider. | Sticky routing keeps one provider per conversation. A caller that returns the details unchanged does not need to read the tag. | ORD; ORI | OA79, OA15, OA8 |
| WebSocket transport and mid-turn steering | OpenAI | Responses WebSocket mode with `response.create` events, `stream_id` lanes, and `response.steer`. | Codex guide: `supports_websockets`: "Leave `false` for OpenRouter". | The transport. | A second transport, outside HTTP chat. | VD; ORD | OA6, OA72 |

## Evidence

Every item has one basis label. Quotes are exact. Line numbers for cloned repositories are at the pinned commits listed under Sources.

### A. OpenRouter's Responses route: what it accepts and what it refuses

**OA1. The Responses route is stateless, and it rejects `store: true` and a non-null `previous_response_id` with a 400.**

- Quote (overview page): "This API is **stateless** - each request is independent and no conversation state is persisted between requests. You must include the full conversation history in each request. Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error."
- Quote (spec, `ResponsesRequest.previous_response_id` description): "Not supported on this proxy. Each response request is independent: no responses are stored, so a previous response cannot be referenced. Requests with a non-null value are rejected with a 400 error. Send the full conversation history in `input` instead."
- Spec, `ResponsesRequest.store`: `"type": "boolean", "const": false, "default": false`.
- Source: OA-s1, OA-s17. Basis: `OR-documented`. Confirms survey claim LC4.

**OA2. OpenRouter's spec lists one Responses path.**

- The spec's paths that mention responses are `POST /responses` ("Create a response", operation `createResponses`, described as "Creates a streaming or non-streaming response using OpenResponses API format") and `POST /presets/{slug}/responses`. OpenAI's reference lists Create, Retrieve, Cancel, Compact, Delete, Input Items List, Input Tokens, and a Conversations resource.
- Source: OA-s17, OA-s59. Basis: `OR-documented` (the content of the spec).

**OA3. The Responses route went GA on 2026-07-25 with unchanged schemas.**

- Quote (changelog, July 25, 2026): "The Responses API is now GA. `beta.responses` was an OpenAPI grouping tag and SDK namespace, never a URL. `POST /api/v1/responses` and its request and response schemas are unchanged, so raw HTTP callers and OpenAI-compatible clients (including the OpenAI Agents SDK) need no changes."
- Source: OA-s28. Basis: `OR-documented`.

**OA4. The request fields the Responses route lists, and OpenAI fields it does not list.**

- `ResponsesRequest` properties in the spec: `access_programs`, `background`, `cache_control`, `debug`, `deferred_tools`, `frequency_penalty`, `image_config`, `include`, `input`, `instructions`, `max_output_tokens`, `max_tool_calls`, `metadata`, `modalities`, `model`, `models`, `parallel_tool_calls`, `plugins`, `presence_penalty`, `previous_response_id`, `prompt`, `prompt_cache_key`, `prompt_cache_options`, `provider`, `reasoning`, `route`, `safety_identifier`, `service_tier`, `session_id`, `stop`, `stop_server_tools_when`, `store`, `stream`, `temperature`, `text`, `tool_choice`, `tools`, `top_k`, `top_logprobs`, `top_p`, `trace`, `truncation`, `user`.
- OpenAI's create reference lists these body parameters that are not in that list: `background_options`, `context_management`, `conversation`, `image`, `moderation`, `prompt_cache_retention`, `stream_options`.
- `include` enum on OpenRouter (`ResponseIncludesEnum`): `file_search_call.results`, `message.input_image.image_url`, `computer_call_output.output.image_url`, `reasoning.encrypted_content`, `output.encrypted_content`, `code_interpreter_call.outputs`, `web_search_call.results`, `web_search_call.action.sources`. OpenAI's list has `message.output_text.logprobs` and does not list `output.encrypted_content`.
- `max_tool_calls` has a different meaning on OpenRouter: "Maximum number of server-tool (e.g. `openrouter:web_search`) agent steps the model may take during a request. Defaults to 30, which is also the maximum."
- Source: OA-s17, OA-s31. Basis: `OR-documented`.

**OA5. Non-OpenAI models are reachable on the Responses route.**

- Catalog: each model has an `endpoints` array of route paths. 384 of 476 models list `/api/v1/responses` (369 list exactly `/api/v1/chat/completions`, `/api/v1/completions`, `/api/v1/messages`, `/api/v1/responses`; 15 more add an images, videos, or extractions path). 322 of the 384 are not `openai/` models. The other 92 models are `:batch` variants that list only `/api/v1/batches`.
- Documentation examples on the Responses route use `anthropic/claude-sonnet-4.5`, `~anthropic/claude-sonnet-latest`, `google/gemini-3.1-pro-preview`, `deepseek/deepseek-r1-0528`, and `anthropic/claude-fable-5.2`.
- Quote (overview): "OpenRouter's Responses API provides OpenAI-compatible access to multiple AI models through a unified interface, designed to be a drop-in replacement for OpenAI's Responses API."
- Source: OA-s18, OA-s9, OA-s1. Basis: `OR-documented`.

**OA6. Codex talks to OpenRouter over the Responses wire only, and the WebSocket transport stays off.**

- Quote (Codex desktop guide, configuration table, row `model_providers.openrouter.wire_api`): "Protocol Codex speaks to the provider. `responses` is the only supported value".
- Quote (same table, row `model_providers.openrouter.supports_websockets`): "Whether the provider supports the Responses API WebSocket transport. Leave `false` for OpenRouter".
- Quote (Codex CLI guide): "Point `model` at any OpenRouter slug (e.g. `~openai/gpt-sol-latest`, `~anthropic/claude-sonnet-latest`) or a pinned version to switch models without changing your Codex installation".
- Source: OA-s19, OA-s20. Basis: `OR-documented`.

**OA7. The Responses route always returns reasoning items, and `reasoning.exclude` works on the chat route only.**

- Quote: "`exclude` is only honored on Chat Completions (`/api/v1/chat/completions`). The Responses API ignores it and always returns `reasoning` items in `output`; filter them out client-side if you do not want them."
- Quote: "For backward compatibility, Chat Completions (`/api/v1/chat/completions`) still accepts the following legacy parameters. The Responses API does not accept them" (the parameters are `include_reasoning: true` and `include_reasoning: false`).
- Source: OA-s9. Basis: `OR-documented`.

### B. Does the chat route call OpenAI's Responses API upstream, and how do encrypted reasoning items come back and go back

**OA8. The chat route's `reasoning_details` entries have a `format` tag, and three of the values name the OpenAI Responses API by provider.**

- The reasoning tokens page has a section "Reasoning Details API Shape" that opens: "This section documents the Chat Completions (`/api/v1/chat/completions`) response shape." Under "Common Fields" it lists "`format` (string): The format of the reasoning detail, with possible values:" and then one bullet per value. Four of the bullets, each quoted whole:
  - "`"openai-responses-v1"` - OpenAI responses format version 1"
  - "`"azure-openai-responses-v1"` - Azure OpenAI responses format version 1"
  - "`"bedrock-openai-responses-v1"` - Amazon Bedrock OpenAI responses format version 1"
  - "`"anthropic-claude-v1"` - Anthropic Claude format version 1 (default)"
- Spec `ReasoningFormat` enum: `unknown`, `openai-responses-v1`, `azure-openai-responses-v1`, `bedrock-openai-responses-v1`, `bedrock-xai-responses-v1`, `xai-responses-v1`, `meta-responses-v1`, `google-vertex-meta-responses-v1`, `google-vertex-xai-responses-v1`, `openrouter-stealth-v1`, `anthropic-claude-v1`, `google-gemini-v1`, `null`.
- Quote (changelog, November 18, 2026, on a renamed value): "It identifies encrypted reasoning minted by Meta models served on Google Vertex ... `reasoning_details[].format` in responses now carries the new value. Requests that replay stored `reasoning_details` with the old value keep working".
- Source: OA-s9, OA-s17, OA-s28. Basis: `OR-documented`. The tag tells which provider minted the reasoning. It is a statement about the reasoning format, not a sentence that says "the chat route calls the Responses API".

**OA9. The one documented sentence that a chat-route request to an OpenAI model is forwarded in Responses form is about `configuration_update` on GPT-6.**

- Quote (GPT-6 guide): "All three OpenRouter APIs expose the update in the shape native to that API. OpenRouter normalizes each into one internal representation and re-emits it as the Responses `configuration_update` input item that GPT-6 expects."
- Quote (same page, Chat Completions section): "OpenAI's Chat Completions API has no per-message effort control, so this form is an OpenRouter extension. Place `configuration_update` on a content-less system message (`content: ""`) directly before the user message it applies to. OpenRouter translates it into the Responses `configuration_update` item before forwarding to the model."
- Quote (same page): "Other models and other OpenAI Responses transports are unaffected."
- Source: OA-s8. Basis: `OR-documented`. Scope: the sentence covers GPT-6 requests that have an update. A general statement that every chat-route request to an OpenAI reasoning model goes to OpenAI's Responses API was searched for across the mirror, the changelog, and the spec, and the search found only the sentences quoted here and in OA45.

**OA10. Inference: the chat route reaches OpenAI reasoning models through the Responses API.**

- Basis: `OR-inferred`. Inferred from five things together:
  - OA8: the `openai-responses-v1` tag on chat-route reasoning details.
  - OA9: the GPT-6 `configuration_update` translation.
  - OA11: OpenRouter's catalog advertises tool calling on the chat route for models where OpenAI's own Chat Completions refuses tool calling.
  - OA12: OpenRouter documents a 12-hour cache TTL on the chat route, which OpenAI offers on the Responses API only.
  - OA13: the official SDK's chat-route fixture has an encrypted reasoning detail whose `id` is a Responses reasoning item id (`rs_...`). OpenAI's Chat Completions API returns no reasoning items ("Passing CoT exists only in the Responses API", OA37).
- Confidence: high for GPT-5.4 and later and for any OpenAI model that returns encrypted reasoning on the chat route. Not shown for non-reasoning models such as `openai/gpt-4o`, which could be served through either upstream API. A single authenticated request with `debug.echo_upstream_body` would settle it (not done, see Open questions).

**OA11. OpenRouter's catalog lists tools on the chat route for models that require Responses for tools on OpenAI.**

- Catalog entry `openai/gpt-6-astra`: `endpoints` is `["/api/v1/chat/completions", "/api/v1/completions", "/api/v1/messages", "/api/v1/responses"]` and `supported_parameters` is `["include_reasoning", "max_completion_tokens", "max_tokens", "reasoning", "reasoning_effort", "response_format", "seed", "structured_outputs", "tool_choice", "tools", "verbosity"]`. The entries for `openai/gpt-6.2-astra` and `openai/gpt-6.1-sol` are the same.
- Compare OA36: OpenAI says tool calling on these models "requires Responses".
- Source: OA-s18. Basis: `OR-documented` (the catalog content). The conclusion drawn from it is OA10.

**OA12. OpenRouter documents a 12-hour explicit cache TTL on the chat route. OpenAI's own Chat Completions reference lists `30m` as the only value.**

- Quote (OpenRouter prompt caching page): "12-hour TTL for a long-lived system prompt, with the rest of the conversation on the 30-minute default (Chat Completions shown; the Responses API takes the same fields on `input_text` blocks)".
- Quote (OpenAI Chat Completions reference, `prompt_cache_options.ttl`): "`ttl: optional "30m"` ... Defaults to `30m`, which is currently the only supported value."
- Quote (OpenAI Responses reference, `prompt_cache_options.ttl`): "`ttl: optional "30m" or "12h"`".
- Source: OA-s10, OA-s32, OA-s31. Basis: `OR-documented` (the claim is OpenRouter's statement; the two OpenAI quotes are given for comparison and feed the inference in OA10).

**OA13. The chat-route streaming shape of OpenAI reasoning, from OpenRouter's official AI SDK provider.**

- File `e2e/issues/issue-519-streaming-summary-merge.test.ts`, lines 72 and 90, a fixture for model `openai/gpt-5.4-nano`: `"choices":[{"index":0,"delta":{"reasoning_details":[{"type":"${ReasoningDetailType.Encrypted}","data":"gAAAAA...Fgw=","id":"rs_0fcb21ad7ba2c5f7","format":"openai-responses-v1","index":0}]},"logprobs":null,"finish_reason":null}]`.
- Same file, lines 5 to 15: "Providers such as OpenAI `openai-responses-v1` stream a single logical summary as many small deltas that all carry `index: 0`, so the array fragmented into one entry per delta. Re-submitting that fragmented array on the next turn breaks multi-turn reasoning round-tripping ... Encrypted details remain discrete (each is an opaque blob with its own id)."
- File `src/chat/index.ts`, lines 893 to 899: "Providers (e.g. OpenAI `openai-responses-v1`) stream a logical reasoning block as many small deltas that all carry the same `index` (frequently `index: 0`), so `index` cannot be used to reassemble the array".
- Spec: `ReasoningDetailEncrypted` has `type: "reasoning.encrypted"`, `data`, `id`, `format`, `index`; `ReasoningDetailSummary` has `type: "reasoning.summary"`, `summary`, `id`, `format`, `index`.
- Source: OA-s30, OA-s17. Basis: `OR-documented` (official SDK source). Note: the JSON is a test fixture written by OpenRouter's team to model the server, so it shows the intended wire shape and is one step from a live capture. OpenRouter's prose documentation illustrates the chat-route shape with Claude only: the three example entries on the reasoning page all use `"format": "anthropic-claude-v1"`.

**OA14. OpenRouter asks OpenAI for encrypted reasoning on the caller's behalf.**

- Quote: "When using the Responses API with manual state management (echoing output items back as input), set `reasoning.context`. OpenRouter automatically requests encrypted reasoning content from reasoning-capable providers, so replay the returned `encrypted_content` unchanged; no explicit `include` field is required".
- Source: OA-s9. Basis: `OR-documented`.

**OA15. Every OpenAI encrypted reasoning payload in OpenRouter's documentation examples ends with a dot and a base64url JSON object that names the model endpoint and the provider.**

- Found by scanning the documentation mirror for the suffix: 22 payloads on 8 pages. Decoded suffixes:

```text
{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|openai"}      (responses/basic-usage, service-tiers)
{"endpoint_slug":"openai/gpt-5.2-20251211|openai"}           (api_reference/overview)
{"endpoint_slug":"openai/o3-2025-04-16|openai"}              (responses/web-search)
{"endpoint_slug":"openai/gpt-5-nano-2025-08-07|openai"}      (agent-sdk/call-model/text-generation)
{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|azure"}       (routing/provider-selection)
{"endpoint_slug":"openai/gpt-5.1-codex-mini-20251113|azure"} (server-tools/apply-patch)
{"endpoint_slug":"google/gemini-3-flash-preview-20251217|google-vertex/global"} (multimodal/image-understanding)
```

- The value appears in `encrypted_content` (Responses route), `encryptedContent` (SDK output), and `redacted_thinking.data` (Messages route). The part before the dot starts with `gAAAAAB`, the usual start of an OpenAI encrypted reasoning blob.
- When Azure serves the request, the reasoning item's `format` is `"azure-openai-responses-v1"` (provider-selection page, four examples).
- Source: OA-s4, OA-s5, OA-s11, OA-s16, OA-s25, OA-s24. Basis: `OR-inferred`. Inferred from the bytes in the documentation examples. No OpenRouter page describes the suffix. Reading: OpenRouter tags each blob with the endpoint that minted it, so that a replayed blob can be matched to an upstream that can decrypt it (OpenAI ties a blob to the organization that created it, OA27). Whether OpenRouter strips the suffix before forwarding, and what it does when the next request routes to a different endpoint, is not documented.

**OA16. On the Responses route the reasoning item has three OpenRouter extension fields.**

- Spec, `OutputReasoningItem` and `ReasoningItem`: OpenAI's fields (`id`, `summary`, `encrypted_content`, `status`, `type`) plus `content` (array of `ReasoningTextContent`), `format` (`ReasoningFormat`), and `signature` ("A signature for the reasoning content, used for verification"). `ReasoningItem` is described as "Reasoning output item with signature and format extensions".
- Example (Responses reasoning page, model `openai/gpt-5-mini`, no `include` in the request): the output item has `"type": "reasoning"`, `"summary": []`, `"encrypted_content": "gAAAAABq_LoL..."`, `"format": "openai-responses-v1"`.
- Source: OA-s17, OA-s3. Basis: `OR-documented`.

**OA17. On the Messages route an OpenAI model's reasoning arrives as Anthropic-shaped blocks.**

- Example (service tiers page, model `openai/gpt-5-mini`, `provider: "OpenAI"`): `"content":[{"type":"thinking","thinking":"**Exploring the meaning of life** ...","signature":""},{"type":"redacted_thinking","data":"gAAAAABq_MTy..."},{"type":"text","text":"..."}]`. The `data` value ends with the endpoint tag from OA15.
- Source: OA-s11. Basis: `OR-documented`.

**OA18. The chat-route replay rule.**

- Quote: "Use `reasoning_details` when working with models that return special reasoning types (such as encrypted or summarized) - this preserves the full structure needed for those models."
- Quote: "The `reasoning_details` functionality works identically across all supported reasoning models. You can easily switch between OpenAI reasoning models (like `~openai/gpt-sol-latest`) and Anthropic reasoning models (like `~anthropic/claude-sonnet-latest`) without changing your code structure."
- Quote: "When providing reasoning\_details blocks, the entire sequence of consecutive reasoning blocks must match the outputs generated by the model during the original request; you cannot rearrange or modify the sequence of these blocks."
- Source: OA-s9. Basis: `OR-documented`. Confirms survey claim LC6 and extends it to OpenAI models.

**OA19. The Responses-route replay rule.**

- Quote: "To preserve reasoning across turns, send the previous response's `output` items back in the next request's `input`, in the order they were returned, and append your `function_call_output` items after them. Keep each `reasoning` item's `content`, `summary`, `signature`, and `encrypted_content` unchanged".
- Source: OA-s9. Basis: `OR-documented`.

**OA20. Some OpenAI models return no reasoning text.**

- Quote: "While most models and providers make reasoning tokens available in the response, some (like the OpenAI o-series) do not."
- The Responses example for `openai/o3` with `"summary": "detailed"` prints an empty `REASONING:` line.
- Source: OA-s9. Basis: `OR-documented`.

**OA21. OpenRouter passes OpenAI's reasoning-removal report through on the Responses route.**

- Spec, `OpenResponsesResult.input_transformations`: "Input items the upstream provider changed before inference (OpenAI `input_transformations`). Present only when the provider reported the field; empty when it changed nothing." The schema `ResponsesInputTransformation` first appears in the changelog entry of November 17, 2026.
- `ChatResult` properties are `alignment`, `choices`, `created`, `id`, `model`, `object`, `openrouter_metadata`, `safeguard_results`, `service_tier`, `system_fingerprint`, `usage`.
- Source: OA-s17, OA-s28. Basis: `OR-documented`.

**OA22. OpenRouter uses the encrypted reasoning field as its own state channel for deferred tools.**

- Quote (tool search page, "Continuation State"): "OpenRouter carries each turn's deferred-tool changes inside the encrypted reasoning it returns: a `reasoning_details` entry on Chat Completions, a reasoning item's `encrypted_content` on Responses, and a `redacted_thinking` block on Messages. Each turn records only what changed, such as the ID of a tool it loaded. There is no separate state field."
- Quote (same page): "**Send assistant messages back exactly as returned**, including the encrypted reasoning described above. Each turn's state is chained to the turns before it, so editing, removing, reordering, trimming, or compacting earlier messages breaks the chain."
- Quote (same page, known clients): "**Codex** replays encrypted reasoning as-is; its default `web_search` tool is accepted as a hosted tool." and "**pi** needs pi-ai `0.84.3` or later on Chat Completions. Earlier versions drop the encrypted `reasoning_details` entry."
- Source: OA-s22. Basis: `OR-documented`.

### C. OpenAI's native rules for state and reasoning

**OA23. OpenAI steers reasoning models to the Responses API.**

- Quote (reasoning guide): "**Reasoning models work better with the [Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses)**. While the Chat Completions API is still supported, you'll get improved model intelligence and performance by using Responses."
- Quote (migration guide): "**While Chat Completions remains supported, Responses is recommended for all new projects.**"
- Source: OA-s34, OA-s36. Basis: `Vendor-documented`.

**OA24. OpenAI's server-held state: `store`, `previous_response_id`, and `conversation`.**

- Quote (Responses reference, `store`, default `true`): "Whether to store the generated model response for later retrieval via API. Defaults to true when omitted. If set to true, response data will be stored for at least 30 days".
- Quote (Responses reference, `previous_response_id`): "The unique ID of the previous response to the model. Use this to create multi-turn conversations." and "Cannot be used in conjunction with `conversation`."
- Quote (Responses reference, `conversation`): "The conversation that this response belongs to. Items from this conversation are prepended to `input_items` for this response request. Input items and output items from this response are automatically added to this conversation after this response completes."
- Quote (conversation state guide): "Conversation objects and items in them are not subject to the 30 day TTL." and "Even when using `previous_response_id`, all previous input tokens for responses in the chain are billed as input tokens in the API."
- Quote (Chat Completions reference, `store`, default `false`): "Whether or not to store the output of this chat completion request for use in our [model distillation](/api/docs/guides/supervised-fine-tuning#distilling-from-a-larger-model) or [evals](/api/docs/guides/evals) products."
- Quote (data controls guide): "the `store` parameter for `/v1/responses` and `v1/chat/completions` will always be treated as `false`, even if the request attempts to set the value to `true`" (under Zero Data Retention).
- Source: OA-s31, OA-s35, OA-s32, OA-s58. Basis: `Vendor-documented`.

**OA25. `encrypted_content` is on by default now.**

- Quote (Responses reference, reasoning item `encrypted_content`): "The encrypted content of the reasoning item. This is populated by default for reasoning items returned by `POST /v1/responses` and WebSocket `response.create` requests. When streaming, use the completed reasoning item and its `encrypted_content` from the `response.output_item.done` event in subsequent requests. The `encrypted_content` in `response.output_item.added` may be incomplete. This is especially important when `store` is `false` or when using Zero Data Retention."
- Quote (reasoning guide): "When you create a response in stateless mode, reasoning items in the response's `output` array include an `encrypted_content` property by default. Stateless mode applies when `store` is `false` or when your organization uses Zero Data Retention (ZDR). The API still accepts the legacy `reasoning.encrypted_content` value in `include` for compatibility, but doesn't require it."
- Quote (migration guide): "When a request includes `encrypted_content`, it is decrypted in memory, used for generating the next response, and then securely discarded."
- The reasoning text itself stays hidden: "The reasoning items remain opaque, and the API does not return their reasoning text."
- Source: OA-s31, OA-s34, OA-s36. Basis: `Vendor-documented`. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28, OA-s66), which recorded the same default. The reference says the default holds for every create request. The guide words it as a stateless-mode default. Both agree for `store: false`.

**OA26. The rule for reasoning items next to function calls.**

- Quote (reasoning guide, "Keeping reasoning items in context"): "When doing [function calling](https://developers.openai.com/api/docs/guides/function-calling) with a reasoning model in the [Responses API](https://developers.openai.com/api/reference/resources/responses), pass back the reasoning items returned with the function calls along with the tool outputs. This lets the model continue reasoning after a tool call."
- Quote (same section): "If you manage conversation history yourself, include the original input and all [output](https://developers.openai.com/api/reference/resources/responses#responses/object-output) items from each response in the [input](https://developers.openai.com/api/reference/resources/responses/methods/create#responses-create-input) of the next request. Keep those items in their original order and append new items."
- Quote (conversation state guide): "For stateless reasoning-model requests, preserve every item in the response's `output` array. The Responses API returns encrypted reasoning items by default. Replaying the complete output keeps reasoning items and assistant `phase` values intact."
- Source: OA-s34, OA-s35. Basis: `Vendor-documented`.

**OA27. OpenAI drops replayed reasoning in five cases and reports each in `input_transformations`.**

- Quote (reasoning guide): "The API removes reasoning items if you switch model families, downgrade to a less capable model, or switch organizations or users mid-conversation as one of our safeguards against distillation. Starting with 6.2, reasoning items with a `safety_identifier` change or a modified conversation history (the prefix) will also be removed."
- Quote (Responses reference, response field `input_transformations`): "Reasoning inputs withheld from the model, in input order." Each entry has `id`, `action: "removed"`, `type: "reasoning"`, and `reason`, one of `"organization_mismatch"`, `"end_user_mismatch"`, `"model_mismatch"`, `"prefix_mismatch"`, `"turn_boundary"`.
- Quote (reasoning guide table, `organization_mismatch`): "You reuse an item from another organization, or from a different workspace under a different user."
- Quote (same table, `prefix_mismatch`): "On GPT-6.2 and later, you insert or edit content before the reasoning item. For example, rewriting an earlier tool result changes the prefix. Deleting whole items without editing the remaining items, or appending new items after the reasoning item, does not by itself trigger this reason."
- Quote (same table, `end_user_mismatch`): "On GPT-6.2 and later, the `safety_identifier` on the request that created the item differs from the one on the request that reuses it. This includes supplying an identifier on only one of the two requests."
- Quote (keep-the-prefix guide): "You can change the prefix unintentionally by updating Responses API parameters such as `tools` or `text.format`, changing `instructions` on pre-GPT-6.2 models, or editing earlier `input` items."
- Quote (OpenAI changelog, Nov 17, 2026): "GPT-6.2 changes how the Responses API preserves reasoning when you edit a conversation’s prefix and how it handles the top-level `instructions` field."
- Source: OA-s34, OA-s31, OA-s41, OA-s40. Basis: `Vendor-documented`.

**OA28. `reasoning.context`.**

- Quote (Responses reference): "Controls which reasoning items are rendered back to the model on later turns. If omitted or set to `auto`, the model determines the context mode. The `gpt-5.6` model family defaults to `all_turns`; earlier models default to `current_turn`." Values: `"auto"`, `"current_turn"`, `"all_turns"`.
- Quote (reasoning guide): "Persisted reasoning can be reused only within the same model family. For example, `gpt-5.6-sol`, `gpt-5.6-terra`, and `gpt-5.6-luna` can reuse each other's reasoning, but reasoning does not carry between the GPT-5.6 and GPT-5.5 families."
- Quote (reasoning guide): "`all_turns` has an effect only when the request has access to earlier response items. Use `previous_response_id`, attach the response to a conversation, or manually replay the complete response history."
- Source: OA-s31, OA-s34. Basis: `Vendor-documented`.

**OA29. `reasoning.effort` values and the per-model exceptions.**

- Quote (reasoning guide): "Supported values are model-dependent and can include `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, and `max`."
- Quote: "[GPT-6 Astra](https://developers.openai.com/api/docs/models/gpt-6-astra) does not support `none` reasoning effort. Setting `reasoning.effort` (Responses) or `reasoning_effort` (Chat Completions) to `none` returns HTTP 400. [GPT-6.1 Sol](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6-astra#gpt-61-sol) does not support `none` or `minimal` and defaults to `medium`."
- Quote: "[GPT-6.2 Astra](https://developers.openai.com/api/docs/models/gpt-6.2-astra) and [GPT-6.2 Sol](https://developers.openai.com/api/docs/models/gpt-6.2-sol) reject requests that set `reasoning.effort` (Responses) or `reasoning_effort` (Chat Completions) to `none`."
- Quote (Using GPT-6 guide): "When reasoning effort is not `none`, remove `temperature`, `top_p`, and `top_logprobs`. For Chat Completions, also remove `logprobs`."
- Source: OA-s34, OA-s37. Basis: `Vendor-documented`.

**OA30. `reasoning.mode`.**

- Quote (reasoning guide): "GPT-5.6 and GPT-6 models support `standard` and `pro` reasoning modes in the Responses API. `standard` is the default. Set `reasoning.mode` to `pro` for difficult tasks that need more model work and can tolerate higher latency and token usage."
- Quote: "Reasoning mode and reasoning effort are independent." and "Existing Pro model IDs keep their current behavior and pricing."
- Source: OA-s34. Basis: `Vendor-documented`.

**OA31. Reasoning summaries are opt-in.**

- Quote (reasoning guide): "While we don't expose the raw reasoning tokens emitted by the model, you can view a summary of the model's reasoning using the `summary` parameter." and "This output will not be included unless you explicitly opt in to including reasoning summaries."
- Quote (Responses reference, `reasoning.summary`): "One of `auto`, `concise`, or `detailed`." and "`concise` is supported for `computer-use-preview` models and all reasoning models after `gpt-5`." The older name `generate_summary` is marked "**Deprecated:** use `summary` instead."
- Source: OA-s34, OA-s31. Basis: `Vendor-documented`.

**OA32. `verbosity` lives in two places on OpenAI.**

- Responses reference, inside `text`: "`verbosity: optional "low" or "medium" or "high" or null (default: "medium")`" with "Constrains the verbosity of the model's response."
- Chat Completions reference, top-level body parameter: "`verbosity: optional "low" or "medium" or "high" or null (default: "medium")`".
- Source: OA-s31, OA-s32. Basis: `Vendor-documented`.

**OA33. How OpenRouter exposes effort, mode, context, summary and verbosity.**

- Quote (reasoning tokens page, request example comment): "`"effort": "high", // Can be "max", "xhigh", "high", "medium", "low", "minimal" or "none" (OpenAI-style)`".
- Quote (same page, effort level note): "Currently supported by OpenAI reasoning models (o1 series, o3 series, GPT-5 series) and Grok models".
- Quote (same page, model catalog fields): "**`supported_efforts`**: Filter effort selectors to these values, returned in descending effort order (highest first). When `null`, all gateway effort values are accepted. When omitted, the model does not expose effort selection."
- Quote (GPT-5.6 guide): "OpenRouter reroutes the request to the matching `*-pro` model listing." and "Pro mode is only honored by OpenAI and Azure. Amazon Bedrock's OpenAI-compatible API accepts `reasoning.mode` but silently ignores it, so OpenRouter only routes pro requests to providers that honor mode selection."
- Quote (GPT-5.6 guide): "You don't need to filter these fields per model. OpenRouter strips `reasoning.mode`, `prompt_cache_breakpoint`, and `prompt_cache_options` before forwarding requests to providers that don't support them, so requests that fall back to other models keep working."
- Quote (reasoning tokens page): "`reasoning.context` is only supported by OpenAI GPT-5.6 and newer."
- Quote (parameters page, Verbosity, enum "(low, medium, high, xhigh, max)"): "Constrains the verbosity of the model's response. Lower values produce more concise responses, while higher values produce more detailed and comprehensive responses. Introduced by OpenAI for the Responses API." and "For Anthropic models, this parameter maps to `output_config.effort`."
- Quote (reasoning tokens page): "OpenAI models expose two knobs, `reasoning.effort` and `text.verbosity`, and the Messages API request shape has one. `output_config.effort` maps to `reasoning.effort` only, and it is not sent as `text.verbosity`."
- Spec: the chat request's `reasoning` object has `context`, `effort`, `enabled`, `exclude`, `max_tokens`, `mode`, `summary`. The Responses request's `reasoning` (`BaseReasoningConfig`) has `context`, `effort`, `mode`, `summary`. `verbosity` appears in the spec only inside `TextConfig` (`high`, `low`, `medium`); `ChatRequest` has `reasoning_effort` as its one top-level reasoning shorthand. The catalog lists `verbosity` under `supported_parameters` for 114 models.
- Source: OA-s9, OA-s7, OA-s13, OA-s17, OA-s18. Basis: `OR-documented`.

**OA34. The catalog's per-model effort lists for OpenAI models.**

- `openai/gpt-6-astra`, `openai/gpt-6.2-astra`, `openai/gpt-6.2-sol`, `openai/gpt-6.1-sol`: `supported_efforts` `["max", "xhigh", "high", "medium", "low"]`, `default_effort` `medium`, `mandatory` `true`.
- `openai/gpt-6-sol`, `openai/gpt-6-luna`, `openai/gpt-6.1-terra`, `openai/gpt-5.6-sol`: `["max", "xhigh", "high", "medium", "low", "none"]`, `mandatory` `false`.
- `openai/gpt-5.4`, `openai/gpt-5.5`: `["xhigh", "high", "medium", "low", "none"]`. `openai/gpt-5`: `["high", "medium", "low", "minimal"]`, `mandatory` `true`. `openai/gpt-5-pro`: `["high"]`. `openai/o3`: `supported_efforts` is `null`, `mandatory` `true`.
- Source: OA-s18. Basis: `OR-documented`.

### D. The gap between Chat Completions and Responses on OpenAI's own platform

**OA35. From GPT-5.4, OpenAI's Chat Completions refuses tool calling together with reasoning.**

- Quote (migration guide, "Additional differences"): "[Reasoning](https://developers.openai.com/api/docs/guides/reasoning) models have a richer experience in the Responses API with [improved tool usage](https://developers.openai.com/api/docs/guides/reasoning#keeping-reasoning-items-in-context). Starting with GPT-5.4, Chat Completions does not support tool calling with `reasoning_effort` values other than `none`."
- Source: OA-s36. Basis: `Vendor-documented`.

**OA36. On GPT-6 models, tool calling exists on the Responses API only.**

- Quote (Using GPT-6 guide): "**Tool calling:** Use the [Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses#migrating-from-chat-completions). GPT-6 Astra, GPT-6.1 Sol, and GPT-6.1 Terra support Chat Completions, but tool calling requires Responses. GPT-6 Sol and GPT-6 Luna support function calling in Chat Completions only with `reasoning_effort: "none"`. Use Responses for reasoning with tools."
- Quote (Using GPT-6.2 guide): "GPT-6.2 Astra and GPT-6.2 Sol support Chat Completions, but tool calling requires Responses."
- Quote (reasoning guide): "Chat Completions does not support function calling with GPT-6 Astra or GPT-6.1 Sol."
- Quote (OpenAI changelog, Sep 3, GPT-6 Astra release): "Tool calling requires the Responses API. If you use tools with Chat Completions, follow the [Responses migration guide](https://developers.openai.com/api/docs/guides/migrate-to-responses)."
- Source: OA-s37, OA-s38, OA-s34, OA-s40. Basis: `Vendor-documented`.

**OA37. What a reasoning model loses on Chat Completions: reasoning continuity.**

- Quote (Using GPT-5.4 guide): "The biggest difference, and main reason to migrate from Chat Completions to the Responses API for GPT-5.4, is support for passing chain of thought (CoT) between turns." and "Passing CoT exists only in the Responses API, and we've seen improved intelligence, fewer generated reasoning tokens, higher cache hit rates, and lower latency as a result of doing so."
- Quote (migration guide): "**Better performance**: Using reasoning models, like GPT-5, with Responses will result in better model intelligence when compared to Chat Completions. Our internal evals reveal a 3% improvement in SWE-bench with same prompt and setup." and "**Lower costs**: Results in lower costs due to improved cache utilization (40% to 80% improvement when compared to Chat Completions in internal tests)."
- Quote (migration guide, on shapes): "Unlike a Chat Completions Message, where many concerns are glued together into one object, Items are distinct from one another and better represent the basic unit of model context." and "Chat Completions can return multiple parallel generations as `choices`, using the `n` parameter. The Responses API omits this parameter and returns only one generation."
- Source: OA-s39, OA-s36. Basis: `Vendor-documented`.

**OA38. Other things OpenAI offers on Responses and not on Chat Completions.**

- Hosted tools. Quote (migration guide): "With Chat Completions, you cannot use OpenAI-hosted tools natively and have to write your own tool integration." Quote (web search guide): "The Chat Completions API supports only specialized search models for web search." Quote (shell guide): "Shell is available through the [Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses). It's not available via the Chat Completions API."
- Server state. Quote (migration guide): "In Chat Completions, conversation state must be managed manually. The Responses API supports the [Conversations API](https://developers.openai.com/api/docs/guides/conversation-state?api-mode=responses#using-the-conversations-api) for persistent conversations, or you can pass a `previous_response_id` to chain responses together."
- Reusable prompts. Quote (OpenAI changelog): "Reusable prompts are not available in Chat Completions."
- File inputs. Quote (file inputs guide): "The Responses API accepts the file types listed below as `input_file` items. Chat Completions accepts only PDF files as `file` content parts."
- Pro mode (OA30), the 12-hour cache TTL (OA12), background mode (OA57), `phase` (OA40), async tools (OA42), `configuration_update` (OA44), compaction (OA46), and the WebSocket transport are documented for the Responses API.
- What Chat Completions has and Responses lacks: `n`, `prediction` (OA68), `logit_bias`, `seed`, `stop`, `frequency_penalty`, `presence_penalty` (body parameters in the chat reference that are not in the Responses reference).
- Source: OA-s36, OA-s53, OA-s40, OA-s55, OA-s31, OA-s32. Basis: `Vendor-documented`.

**OA39. Strict mode defaults differ between the two OpenAI APIs.**

- Quote (function calling guide): "If you omit `strict`, the default depends on the API: Responses requests will attempt to normalize your schema into strict mode when possible, and will fall back to non-strict, best-effort function calling if the schema cannot be made compatible with strict mode. When fallback happens, the response tool will show `strict: false`. Chat Completions requests remain non-strict by default."
- Quote (migration guide): "In Chat Completions, function definitions are externally tagged. In Responses, they are internally tagged."
- Source: OA-s50, OA-s36. Basis: `Vendor-documented`.

### E. `phase`, async tools, `configuration_update`, compaction, truncation

**OA40. OpenAI's `phase` field.**

- Quote (Responses reference, on `EasyInputMessage` and on the output message): "`phase: optional "commentary" or "final_answer" or null`" with "Labels an `assistant` message as intermediate commentary (`commentary`) or the final answer (`final_answer`)." and "Not used for user messages." The same paragraph tells callers of "models like `gpt-5.3-codex` and beyond" to "preserve and resend phase on all assistant messages".
- Quote (reasoning guide): "For long-running or tool-heavy flows with GPT-5.5 and GPT-5.4 in the Responses API, use the assistant message `phase` field to avoid early stopping and other misbehavior." and "Missing or dropped `phase` can cause preambles to be treated as final answers in those workflows."
- Quote (reasoning guide): "Using `previous_response_id` is usually the simplest path because prior assistant state is preserved. If you replay assistant history manually, preserve each original `phase` value."
- Source: OA-s31, OA-s34. Basis: `Vendor-documented`.

**OA41. OpenRouter supports `phase` on the Responses route only and says why the chat route cannot hold it.**

- Quote (GPT-5.4 guide): "`phase` is **not available** in the Chat Completions API. The Chat Completions format cannot represent multiple output items with distinct phases in a single response. Use the Responses API for full `phase` support."
- Quote (same page): "The Chat Completions API does **not** support `phase` in responses. A single chat completion response can only contain one message per choice, so there is no way to represent the separate commentary and final answer output items that models like GPT-5.4 produce."
- Quote (same page): "When using the Responses API, assistant output items include `phase`. You must persist these items verbatim and pass them back in subsequent requests."
- Supported-models table rows: `openai/gpt-5.5`, `openai/gpt-5.5-pro`, `openai/gpt-5.4`, `openai/gpt-5.4-pro`, `openai/gpt-5.3-codex` "Supported"; "Other OpenAI models | Silently ignored (safe to pass)"; "Non-OpenAI models | Not applicable".
- Spec: `phase` is a property of `OutputMessage` and `EasyInputMessage`. `ChatAssistantMessage` properties are `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`.
- Source: OA-s6, OA-s17. Basis: `OR-documented`. The page does not say what the chat route does with the text of a `commentary` message (see Open questions).

**OA42. OpenAI's async tool calling.**

- Quote (async tool calling guide): "A normal [function call](https://developers.openai.com/api/docs/guides/function-calling) pauses the model's turn to wait for a tool response. Set `async: true` on a function or custom tool definition to let the model continue working after issuing that call, before your application returns the output."
- Quote: "Your application still executes the tool. Async tools don't move execution to OpenAI or manage your background jobs."
- Quote: "When a job finishes, include its output in a later Responses request. Use the original API `call_id` to match the result to its call".
- Quote: "The response can contain both the async call and an answer. If other conversation turns happen before the job finishes, update `latest_response_id` to continue from the latest response while keeping the original tool `call_id`."
- Source: OA-s49. Basis: `Vendor-documented`.

**OA43. OpenRouter exposes async tools on the Responses route only.**

- Quote (GPT-6 guide): "OpenRouter supports async tools on the [Responses API](https://openrouter.ai/docs/api/api-reference/responses/create-a-response.md). Chat Completions and Anthropic Messages requests run every tool call synchronously. A Chat Completions tool that carries `async` is accepted, but the flag is removed before the request is routed and the call behaves like any other tool call."
- Quote: "OpenAI's async tool guide continues the conversation with `previous_response_id`, which makes OpenAI prepend the stored transcript. OpenRouter's Responses API is stateless and rejects `previous_response_id` with a 400, so you replay the transcript yourself."
- Quote: "A call whose output has not arrived yet stays pending. You can append further user turns and assistant output after a pending `function_call` and deliver its `function_call_output` in a later request, as long as the `call_id` matches."
- Quote (routing): "OpenAI and Azure OpenAI honor async tools for the GPT-6 family. When a request declares an async tool and one of those endpoints is a routing candidate, OpenRouter narrows routing to them. When none is, the request keeps its normal candidate pool and the `async` flag is removed before the request is sent, so the call runs synchronously and the returned `function_call` carries no `async` field. Amazon Bedrock accepts the flag but runs the tool synchronously, so it is treated as a non-async transport."
- Quote: "OpenAI rejects `async: true` with a 400 on earlier models, so add the flag only on requests to GPT-6 models."
- Spec, `async` on tools: "Only honored by providers whose Responses API supports async tools; ignored elsewhere."
- Source: OA-s8, OA-s17. Basis: `OR-documented`.

**OA44. OpenAI's `configuration_update` item.**

- Quote (Responses reference): "An update to the conversation's response configuration. Supply reasoning, text, or both. Each setting stays in effect until a later update replaces it." The `reasoning` part: "Updates to reasoning configuration. Only effort is supported."
- Quote (reasoning guide): "On supported GPT-6 and later models, you can update reasoning effort and response format in standard, single-agent mode."
- Quote: "The response's `reasoning.effort` continues to report the request-level setting, not the effort selected by the update."
- Quote: "Automatic truncation and the standalone `/responses/compact` endpoint still reject histories containing updates."
- Source: OA-s31, OA-s34. Basis: `Vendor-documented`.

**OA45. OpenRouter exposes the update on all three routes and invented a chat-route form.**

- Quote (reasoning tokens page): "The three forms are interchangeable: OpenRouter normalizes each into one internal representation and re-emits it in the form the selected provider accepts. A Responses `configuration_update` sent to a Claude model becomes a per-message `output_config`, and a Chat Completions or Messages API update sent to an OpenAI model becomes a Responses `configuration_update` item."
- Quote (GPT-6 guide): "The field is accepted only on `system` and `developer` messages. A `configuration_update` on a user, assistant, or tool message is rejected with a 400."
- Quote (GPT-6 guide): "On OpenRouter, a request that carries an update is routed only to endpoints that accept it, and a request for a model that does not support it is rejected with a 400 rather than sent with the update silently dropped."
- Quote (GPT-6 guide, pro models): "OpenAI rejects the item with `The 'configuration_update' item type is not supported with pro or tournament models.`"
- Spec: `ChatSystemMessage.configuration_update` is described as "OpenRouter extension." `ConfigurationUpdateItem` requires `type` and `reasoning`. `ConfigurationUpdateReasoning` has `additionalProperties: false` and one property, `effort` (`max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`).
- Changelog: `ConfigurationUpdateItem` and `ChatSystemMessage.configuration_update` first appear on September 4, 2026, and `ChatDeveloperMessage.configuration_update` on September 8, 2026.
- Source: OA-s9, OA-s8, OA-s17, OA-s28. Basis: `OR-documented`. OpenAI's item also takes `text` (format); OpenRouter's schema has the `reasoning` part only.

**OA46. OpenAI's compaction.**

- Quote (compaction guide): "You can enable server-side compaction in a Responses create request (`POST /responses` or `client.responses.create`) by setting `context_management` with `compact_threshold`." and "The response stream includes the encrypted compaction item."
- Quote: "The returned compaction item carries forward key prior state and reasoning into the next run using fewer tokens. It is opaque and not intended to be human-interpretable."
- Quote: "To compact now, append `{"type": "compaction_trigger"}` as the last input item in a `POST /v1/responses` request."
- Quote (Responses reference, `Compaction` input item): "A compaction item generated by the [`v1/responses/compact` API](/api/reference/resources/responses/methods/compact)." with `encrypted_content` "The encrypted content of the compaction summary."
- Quote (Responses reference, `context_management`): "The context management entry type. Currently only 'compaction' is supported." and `compact_threshold` "`minimum: 1000`".
- Quote (keep-the-prefix guide): "If you use [OpenAI compaction](https://developers.openai.com/api/docs/guides/compaction) via `/responses/compact` or `context_management`, our systems retain subsequent reasoning items despite the prefix being changed."
- Source: OA-s48, OA-s31, OA-s41. Basis: `Vendor-documented`.

**OA47. OpenRouter accepts compaction items and `truncation`, and lists no threshold field on the Responses route.**

- Spec: `truncation` is `OpenAIResponsesTruncation`, enum `auto`, `disabled`, `null`.
- Quote (GPT-6 guide): "Updates cannot be combined with `truncation: "auto"` in the Responses API. Automatic truncation could drop the update and silently revert the effort."

- Spec input items: `CompactionTriggerItem` (`type: "compaction_trigger"`, "Asks the model to compact the preceding input into a compaction item"), `CompactionItem` (`type: "compaction"`, "A context compaction marker with encrypted summary"), `ContextCompactionItem` (`type: "context_compaction"`, "A context compaction marker with an optional encrypted summary"). Output: `OutputCompactionItem` ("A compaction output item carrying the encrypted summary of the compacted conversation. Emitted in reply to a `compaction_trigger` input item.") with fields `created_by`, `encrypted_content`, `id`, `type`.
- In the spec `context_management` is a property of `MessagesRequest` and `MessagesResult` only.
- Changelog: `CompactionTriggerItem` first appears on November 4, 2026.
- Source: OA-s17, OA-s28, OA-s8. Basis: `OR-documented`. The spec is where these items are documented; the guides describe only the `truncation` rule quoted above.

**OA48. OpenAI's `truncation`.**

- Quote (Responses reference): "`auto`: If the input to this Response exceeds the model's context window size, the model will truncate the response to fit the context window by dropping items from the beginning of the conversation." and "`disabled` (default): If the input size will exceed the context window size for a model, the request will fail with a 400 error."
- Source: OA-s31. Basis: `Vendor-documented`.

### F. Built-in tools and custom tools

**OA49. OpenAI's Responses tool types and the item types each produces.**

- The Responses reference lists 16 tool shapes: `Function` (`function`), `FileSearch` (`file_search`), `Computer` (`computer`), `ComputerUsePreview` (`computer_use_preview`), `WebSearch` (`web_search`), `Mcp` (`mcp`), `CodeInterpreter` (`code_interpreter`), `ProgrammaticToolCalling` (`programmatic_tool_calling`), `ImageGeneration` (`image_generation`), `LocalShell` (`local_shell`), `Shell` (`shell`), `Custom` (`custom`), `Namespace` (`namespace`), `ToolSearch` (`tool_search`), `WebSearchPreview` (`web_search_preview`), `ApplyPatch` (`apply_patch`).
- It lists 33 input item shapes: `EasyInputMessage`, `Message`, `ResponseOutputMessage`, `FileSearchCall`, `ComputerCall`, `ComputerCallOutput`, `WebSearchCall`, `FunctionCall`, `FunctionCallOutput`, `ToolSearchCall`, `ToolSearchOutput`, `AdditionalTools`, `ConfigurationUpdate`, `Reasoning`, `Compaction`, `ImageGenerationCall`, `CodeInterpreterCall`, `LocalShellCall`, `LocalShellCallOutput`, `ShellCall`, `ShellCallOutput`, `ApplyPatchCall`, `ApplyPatchCallOutput`, `McpListTools`, `McpApprovalRequest`, `McpApprovalResponse`, `McpCall`, `CustomToolCallOutput`, `CustomToolCall`, `CompactionTrigger`, `ItemReference`, `Program`, `ProgramOutput`.
- Item `type` strings in the tool guides: web search `web_search_call` (and `url_citation` annotations); file search `file_search_call` (and `file_citation`); code interpreter `code_interpreter_call` (and `container_file_citation`); computer use `computer_call`, `computer_call_output`; image generation `image_generation_call`; MCP `mcp_list_tools`, `mcp_call`, `mcp_approval_request`, `mcp_approval_response`; shell `shell_call`, `shell_call_output`; apply patch `apply_patch_call`, `apply_patch_call_output`; tool search `tool_search_call`, `tool_search_output`, `additional_tools`; programmatic tool calling `program`, `program_output`; custom tools `custom_tool_call`, `custom_tool_call_output`.
- Quote (tool search guide): "In the Responses API, only `gpt-5.4` and later models support `tool_search`." and "**Hosted tool search:** OpenAI searches across the deferred tools you declared in the request and returns the loaded subset in the same response." and "**Client-executed tool search:** The model emits a `tool_search_call`, your application performs the lookup, and you return a matching `tool_search_output`."
- Quote (Responses reference, `max_tool_calls`): "The maximum number of total calls to built-in tools that can be processed in a response."
- Source: OA-s31, OA-s53. Basis: `Vendor-documented`.

**OA50. The tool and item types OpenRouter's Responses route lists.**

- `ResponsesRequest.tools` union, OpenAI-shaped members: `function`, `web_search_preview`, `web_search_preview_2025_03_11`, `web_search`, `web_search_2025_08_26`, `file_search`, `computer_use_preview`, `code_interpreter`, `mcp`, `image_generation`, `local_shell`, `shell`, `apply_patch`, `custom`, `namespace`. OpenRouter members: `openrouter:advisor`, `openrouter:subagent`, `openrouter:datetime`, `openrouter:files`, `openrouter:fusion`, `openrouter:image_generation`, `openrouter:experimental__search_models`, `openrouter:web_fetch`, `openrouter:web_search`, `openrouter:apply_patch`, `openrouter:bash`, `openrouter:shell`, `openrouter:tool_search`, `openrouter:mcp`.
- OpenAI tool types from OA49 that are not in that union: `computer`, `programmatic_tool_calling`, `tool_search`.
- OpenAI-shaped input item types listed: `reasoning`, `message`, `function_call`, `function_call_output`, `apply_patch_call`, `apply_patch_call_output`, `web_search_call`, `file_search_call`, `image_generation_call`, `code_interpreter_call`, `computer_call`, `local_shell_call`, `local_shell_call_output`, `shell_call`, `shell_call_output`, `mcp_list_tools`, `mcp_approval_request`, `mcp_approval_response`, `mcp_call`, `custom_tool_call`, `custom_tool_call_output`, `compaction_trigger`, `compaction`, `context_compaction`, `item_reference`, `additional_tools`, `agent_message`, `configuration_update`, `tool_search_call`, `tool_search_output`. OpenAI input item types not listed: `computer_call_output`, `program`, `program_output`.
- `item_reference` is accepted in the schema ("A reference to a previous response item by ID") although the route stores no responses (OA1).
- Spec, `CustomToolCallItem` description: "Used for tools like Codex CLI's `apply_patch` whose payload is opaque text rather than JSON arguments."
- Source: OA-s17. Basis: `OR-documented`. The spec is generated and may lag the server.

**OA51. File search and code interpreter need the caller's own OpenAI key.**

- Quote (spec, `FileSearchServerTool`): "File search tool configuration. `vector_store_ids` names OpenAI vector stores, so it requires BYOK (your own OpenAI key); requests on OpenRouter credits get a 400."
- Quote (spec, `CodeInterpreterServerTool`): "Code interpreter tool configuration. An existing container id or `container.file_ids` requires BYOK (your own OpenAI key); requests on OpenRouter credits must use `auto` with no `file_ids`."
- Source: OA-s17. Basis: `OR-documented`.

**OA52. Shell and apply patch: native on OpenAI endpoints, an OpenRouter engine elsewhere.**

- Quote (shell page): "On the Responses API you can also send OpenAI's native tool shape (`{ "type": "shell" }`, or the legacy Codex `local_shell`) instead of `openrouter:shell`. On OpenAI models this uses OpenAI's own hosted shell; on any other model, OpenRouter routes the call to its sandbox transparently. The response emits the native `shell_call` output item either way."
- Quote (spec, `ShellServerToolEnvironment`): "Server-side execution environment for the shell tool. Only container-backed environments are supported; "local" shells are not."
- Quote (apply patch page): "**Native passthrough** streams the diff incrementally via `response.apply_patch_call_operation_diff.delta` events, matching OpenAI's streaming format. Currently supported on OpenAI endpoints." and "**HITL (human-in-the-loop)** buffers the complete diff and delivers it as a single atomic `apply_patch_call` output item."
- Server tools table rows: "`openrouter:apply_patch` | Propose file edits via V4A diff patches (Responses API only)", "`openrouter:shell` | Run commands in a hosted, sandboxed shell (Responses and Messages APIs)", "`openrouter:tool_search` | Discover and load deferred tools on demand (Responses and Messages APIs)".
- Source: OA-s23, OA-s24, OA-s21, OA-s17. Basis: `OR-documented`.

**OA53. The chat route's tool union has `function` and OpenRouter server tools.**

- Spec, `ChatFunctionTool`: "Tool definition for function calling (regular function or OpenRouter built-in server tool)". Members: the `function` shape, then `AdvisorServerTool_OpenRouter`, `BashServerTool`, `DatetimeServerTool`, `FilesServerTool`, `FusionServerTool_OpenRouter`, `ImageGenerationServerTool_OpenRouter`, `ChatSearchModelsServerTool`, `SubagentServerTool_OpenRouter`, `WebFetchServerTool`, `OpenRouterWebSearchServerTool`, `ChatWebSearchShorthand`, `ChatDynamicServerTool`.
- Spec, `ChatWebSearchShorthand`: "Web search tool using OpenAI Responses API syntax. Automatically converted to openrouter:web_search."
- Spec, `ChatToolCall.type`: enum `["function"]`.
- Source: OA-s17. Basis: `OR-documented`.

**OA54. OpenAI's custom tools and grammars.**

- Quote (function calling guide): "Custom tools work in much the same way as JSON schema-driven function tools. But rather than providing the model explicit instructions on what input your tool requires, the model can pass an arbitrary string back to your tool as input."
- Quote: "You can provide a custom CFG using the `grammar` parameter when configuring a custom tool. Currently, we support two forms of CFG syntax when defining grammars: `lark` and `regex`."
- Output item example in the guide: `"type": "custom_tool_call"`, `"call_id": "call_aGiFQkRWSWAIsMQ19fKqxUgb"`, `"input": "print(\"hello world\")"`, `"name": "code_exec"`.
- OpenAI's Chat Completions reference also lists `ChatCompletionCustomTool object { custom, type }` and, on assistant messages, `ChatCompletionMessageCustomToolCall object { id, custom, type }` ("A call to a custom tool created by the model.").
- Source: OA-s50, OA-s32. Basis: `Vendor-documented`.

**OA55. Programmatic tool calling adds an opaque value that must make a round trip.**

- Quote (Responses reference, `Program` item): `code` is "The JavaScript source executed by programmatic tool calling." and `fingerprint` is "Opaque program replay fingerprint that must be round-tripped."
- Function and custom tools have `allowed_callers: optional array of "direct" or "programmatic" or null` ("The tool invocation context(s).").
- Source: OA-s31. Basis: `Vendor-documented`.

**OA56. Under OpenRouter's deferred tools, two OpenAI hosted tools pass through and the others are refused.**

- Quote (tool search page): "Freeform (custom) tools that are not deferred, such as Codex `apply_patch`, are sent to the model unchanged and their calls are passed back without validation. Hosted tools the provider executes itself are also sent unchanged when their executed calls can be replayed with the conversation: `web_search` (every variant) and `code_interpreter`."
- Quote: "Other hosted tool types (`file_search`, `mcp`, `image_generation`, `computer_use_preview`, shells) return a `400` with `deferred_tools` until their replayed items are supported."
- Source: OA-s22. Basis: `OR-documented`.

### G. Background runs, webhooks, batch

**OA57. OpenAI's background mode.**

- Quote (background guide): "Use background mode to run long-running Responses API requests asynchronously, including multi-step agent runs."
- Quote: "Set `background` to `true` and save the response ID returned by the API. Use a webhook to learn when the request finishes and retrieve the response by ID."
- Quote: "Background mode temporarily stores response data even when you set `store` to `false` or use Zero Data Retention."
- Quote (polling): "Keep polling while the status is `queued` or `in_progress`. When the status changes to a terminal state, stop polling."
- Quote (cancel): "Canceling twice is idempotent. Later calls return the final `Response` object. For a synchronous response, close the connection instead."
- Quote (streaming): "If the connection drops, resume the stream by passing the last event's `sequence_number` as `starting_after`. You can't stream a response later if it was created without streaming."
- Quote (Responses reference, `background_options`): "Options for background execution on HTTP requests. The `2h` and `8h` completion windows require `background: true`, Standard processing, and `stream: false` or omitted." and, for `continuation_id`, "A customer-provided opaque identifier used to prioritize related background requests. It does not add prior history to the input."
- Quote (OpenAI changelog, Nov 17): "Choose a two-hour or eight-hour [background completion window](https://developers.openai.com/api/docs/guides/background#use-discounted-completion-windows) in the Responses API public beta. Model token rates are 30% lower than Standard for the two-hour window and 40% lower for the eight-hour window."
- Webhooks: the webhook reference lists `response.completed`, `response.cancelled`, `response.failed`, `response.incomplete`, `batch.completed`, `batch.cancelled`, `batch.expired` among its events. Quote (webhooks guide): "Webhooks are delivered to an HTTP endpoint you control, following the [Standard Webhooks specification](https://github.com/standard-webhooks/standard-webhooks/blob/main/spec/standard-webhooks.md)."
- Source: OA-s42, OA-s31, OA-s40, OA-s57. Basis: `Vendor-documented`.

**OA58. Inference: OpenAI's background mode is not usable through OpenRouter.**

- Basis: `OR-inferred`. Inferred from three things:
  - The spec lists `background` on `ResponsesRequest` as `"type": ["boolean", "null"]`, and the type is all the spec says about it. The documentation's example responses show `"background": false`.
  - The spec has one Responses path, `POST /responses` (OA2). Polling and cancel need `GET /responses/{id}` and `POST /responses/{id}/cancel`.
  - A search of the documentation mirror (outside the generated API reference) for `background mode`, `background: true`, `background_options` and `completion_window` finds only OpenRouter's own Batch API pages. The route is described as storing nothing (OA1), and background mode stores the response (OA57).
- Confidence: medium. The docs do not say what the server does with `background: true` (ignore, reject, or run in the foreground).

**OA59. OpenAI's Batch API.**

- Quote (batch guide): "Learn how to use OpenAI's Batch API to send asynchronous groups of requests with 50% lower costs, a queued-token limit separate from per-minute limits, and a clear 24-hour turnaround time."
- Quote: "Batches start with a `.jsonl` file where each line contains the details of an individual request to the API." The endpoints listed are `/v1/responses`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/completions`, `/v1/moderations`, `/v1/images/generations`, `/v1/images/edits`.
- Quote: "Note that each input file can only include requests to a single model."
- Source: OA-s54. Basis: `Vendor-documented`.

**OA60. OpenRouter has its own Batch API with a different shape.**

- Quote (batch quickstart, field table): "`/v1/chat/completions`, `/v1/responses`, `/v1/messages`, or `/v1/embeddings`. Every request uses this endpoint." (field `endpoint`); "`24h`, which is the default and the only accepted value." (field `completion_window`); "A public HTTPS URL that receives a webhook when the batch finishes." (field `callback_url`).
- Quote (batch page): "Batch requests are typically billed at 50% of the model's standard per-token price."
- Quote (batch page, limitations): "Use the sync API. Batch keeps data for 30 days, so it cannot run when your privacy settings require ZDR for the model's provider."
- Quote (reasoning tokens page): "Mid-conversation effort updates are not accepted on the [Batch API](https://openrouter.ai/docs/batch-quickstart.md) and are rejected per line."
- Catalog: 92 `:batch` model variants (44 of them `openai/`) list `/api/v1/batches` as their only endpoint.
- Source: OA-s27, OA-s9, OA-s18. Basis: `OR-documented`.

### H. Prompt caching

**OA61. OpenAI's prompt caching rules.**

- Quote (prompt caching guide): "The minimum cacheable prompt length is 1,024 tokens for GPT-5.6 and later and varies by request settings for earlier models."
- Quote (GPT-5.6 and later): "Set `prompt_cache_options.mode` to `explicit` to use only developer-selected breakpoints and mark each desired breakpoint by adding `prompt_cache_breakpoint: { "mode": "explicit" }` to a supported content block inside an input message." and "Each request can create up to four cache writes."
- Quote (implicit mode): "When `prompt_cache_options.mode` is `implicit`, OpenAI places a breakpoint at the end of the latest eligible message."
- Quote (earlier models): "Only implicit caching is supported."
- Summary table row "Cache write charge": "1.25× for `"30m"`; 2.5× for `"12h"` in the Responses API" for GPT-5.6 and later, "No additional cache-write charge" for earlier models. Row "Supported retention values": "`"30m"` (default), or `"12h"` in the Responses API" against "`"24h"` only" for GPT-5.5 and "`"in_memory"` or `"24h"`" for other earlier models.
- Quote (`prompt_cache_key`): "On models before GPT-5.6, use a stable [`prompt_cache_key`] ... for requests that share a reusable prefix to help route related requests to the same cache." (link target omitted) and "On GPT-5.6 and later, OpenAI handles cache routing automatically; the key is not needed to optimize caching. You can use separate keys to maintain separate cache accounting for customers or users within your application."
- Quote: "Caches are not shared across organizations and cannot be reused across [regional processing boundaries](https://developers.openai.com/api/docs/guides/data-residency)."
- Quote (Responses reference, `prompt_cache_retention`): "Deprecated. Use `prompt_cache_options.ttl` instead." and "For `gpt-5.5`, `gpt-5.5-pro`, and future models, only `24h` is supported."
- Responses reference, `prompt_cache_options`: "`optional object { comparison_response_id, mode, prewarm, ttl }`". `prewarm`: "Prepares the prompt cache without generating output."
- Quote (reporting): "For Responses API models that support `prompt_cache_options`, `input_tokens_details.cache_write_tokens_details` breaks writes into `ttl_30m_tokens` and `ttl_12h_tokens`." and "`cached_tokens` counts billed reads and excludes tokens charged as writes".
- Quote (earlier models): "Reported `cached_tokens` is calculated by subtracting the hidden system tokens from the last matched breakpoint, then rounding down to the nearest multiple of 128."
- Source: OA-s43, OA-s31. Basis: `Vendor-documented`.

**OA62. From GPT-6.2, top-level `instructions` sit after `input` and are billed uncached.**

- Quote (Using GPT-6.2 guide): "Models starting with GPT-6.2 append top-level `instructions` after `input` in the Responses API. To make reusable instructions eligible for prompt caching, put them in a `developer` message as the first item in `input`."
- Quote (prompt caching guide): "In the Responses API, models starting with GPT-6.2 append top-level `instructions` after `input` and bill those tokens at the uncached input rate, even on a cache hit. Put reusable instructions in the first `developer` message in `input`."
- Quote (prompt caching guide): "Top-level `instructions` cannot contain an explicit breakpoint."
- Source: OA-s38, OA-s43. Basis: `Vendor-documented`.

**OA63. How OpenRouter exposes OpenAI caching.**

- Quote: "Prompt caching with OpenAI is automated and does not require any additional configuration. There is a minimum prompt size of 1024 tokens."
- Quote: "`prompt_cache_breakpoint`: placed on an individual text content block (`input_text` in Responses, `text` in Chat Completions) to mark the end of a reusable prefix."
- Quote: "`prompt_cache_options`: placed at the request root. Setting `mode` to `"explicit"` disables OpenAI-managed breakpoints so only blocks marked with `prompt_cache_breakpoint` participate in caching."
- Quote: "The block-level markers are interchangeable: a text block marked with Anthropic-style `cache_control` gets a `prompt_cache_breakpoint` when routed to a supporting OpenAI model, and a block marked with `prompt_cache_breakpoint` gets a default (5-minute) `cache_control` when routed to Anthropic or Google." The same note ends: "a `cache_control` `ttl` is dropped toward OpenAI, a `prompt_cache_breakpoint` `ttl` is dropped toward Anthropic or Google, and the request-level `prompt_cache_options` stays OpenAI-only."
- Quote: "Cache activity is reported in `usage.input_tokens_details` (Responses) and `usage.prompt_tokens_details` (Chat Completions): `cache_write_tokens` counts prompt tokens written to the cache, and `cached_tokens` counts prompt tokens read from it."
- Quote (sticky routing): "If neither is set, OpenRouter falls back to the OpenAI-style `prompt_cache_key` request field as the sticky routing key. Clients that already send `prompt_cache_key` get session-pinned routing without any changes."
- Quote (turning caching off): "To turn caching off for a request, set `prompt_cache_options.mode` to `"explicit"` and do not mark any content block with `prompt_cache_breakpoint`."
- Source: OA-s10. Basis: `OR-documented`. Confirms survey claim LC7 (translation both ways, TTLs dropped).

**OA64. What the spec shows OpenRouter leaves out of OpenAI's caching surface.**

- `PromptCacheOptions` in the spec has `mode` (enum `["explicit"]`, required) and `ttl`. OpenAI's Responses object has `comparison_response_id`, `mode` (`implicit` or `explicit`), `prewarm`, `ttl`.
- The spec has no schema or property named `prompt_cache_retention`.
- Usage: `OpenAIResponsesUsage.input_tokens_details` has `cache_write_tokens` and `cached_tokens`. `ChatUsage.prompt_tokens_details` has `audio_tokens`, `cache_write_tokens` ("Tokens written to cache. Only returned for models with explicit caching and cache write pricing."), `cached_tokens`, `video_tokens`. Neither has `cache_write_tokens_details`. The response field `prompt_cache_diagnostics` is in OpenAI's reference and not in `BaseResponsesResult`.
- Source: OA-s17, OA-s31. Basis: `OR-documented`.

**OA65. `cache_write_tokens` is now OpenAI's own Chat Completions field too.**

- Quote (OpenAI Chat Completions reference, `usage.prompt_tokens_details`): "`cache_write_tokens: optional integer (default: 0)`" with "The unadjusted number of prompt tokens written to cache."
- Source: OA-s32. Basis: `Vendor-documented`. This changes how to read survey claim LC17 (see Conflicts).

### I. Service tiers

**OA66. OpenAI's `service_tier`.**

- Responses reference values: `"auto"`, `"default"`, `"flex"`, `"scale"`, `"priority"`, `"fast"`, `"ultrafast"`.
- Quote: "To opt-in to [Fast mode](/api/docs/guides/fast-mode) at the request level, include the `service_tier=fast` or `service_tier=priority` parameter for Responses or Chat Completions. The response will show `service_tier=priority` regardless of if you specify `service_tier=fast` or `priority` in your request."
- Quote: "When the `service_tier` parameter is set, the response body will include the `service_tier` value based on the processing mode actually used to serve the request. This response value may be different from the value set in the parameter."
- Quote (fast mode guide): "Priority processing was renamed Fast mode on July 30, 2026." and "If your traffic ramps too fast, the system may downgrade some Fast mode requests to standard speeds and charge standard rates. When this happens, the response contains `service_tier: "default"`."
- Quote (flex guide): "Flex is unavailable on GPT-6.2 Astra and GPT-6.2 Sol" and "Flex processing may sometimes lack sufficient resources to handle your requests, resulting in a `429 Resource Unavailable` error code. **You will not be charged when this occurs.**"
- Quote (ultrafast guide): "Ultrafast mode is the fastest service tier in the OpenAI API. It supports GPT-6.2 Astra, GPT-6.2 Sol, GPT-6 Astra, and GPT-6.1 Sol."
- Quote (error codes guide): "The API returns the message "Invalid `service_tier` argument: The requested service tier is not allowed for this project." as an `invalid_request_error` with `error.param` set to `service_tier`".
- Source: OA-s31, OA-s46, OA-s45, OA-s47, OA-s52. Basis: `Vendor-documented`.

**OA67. OpenRouter turns each tier into an endpoint and keeps the field name.**

- Quote: "OpenRouter lists each tier as its own endpoint of the model. You can let tier endpoints compete with standard endpoints for your traffic, or request a tier explicitly. Either way, the response reports the tier that actually served the request, and you're billed at that endpoint's price."
- Quote: "These values work the same for every provider that sells more than one tier; the examples on this page use OpenAI models, but nothing about the parameter is OpenAI-specific."
- Quote: "**Reported tier.** The response always reports the tier as `priority`, never `fast`. This matches OpenAI's own API, which returns `priority` even when the request spelled the tier `fast`."
- Quote: "**Responses API** (`/api/v1/responses`): `service_tier` is returned at the **top level** of the response object, matching OpenAI's native format." The same sentence is given for the chat route, and for the Messages route the field is "inside the **`usage` object**".
- Quote (flex): "Fallback is bounded by price: the request only falls back to endpoints whose input and output prices are both at or below a flex endpoint's".
- Quote: "In every case, billing follows the tier that actually served the request: if a provider sheds a tier request to its default tier, you're billed the default rate."
- Quote (workspace policy): "A request that selects the tier explicitly is rejected with a `403`".
- Spec: request enum `auto`, `default`, `fast`, `flex`, `priority`, `scale`, `ultrafast`, `null` on `ChatRequest` and `ResponsesRequest`; result `ServiceTier` enum `auto`, `default`, `flex`, `priority`, `scale`, `ultrafast`, `null`. `ResponsesErrorField.code` includes `flex_unavailable`.
- Source: OA-s11, OA-s17. Basis: `OR-documented`.

### J. Predicted outputs, structured outputs, refusals

**OA68. Predicted outputs are a Chat Completions feature on OpenAI.**

- Quote (predicted outputs guide): "**Predicted Outputs** enable you to speed up API responses from [Chat Completions](https://developers.openai.com/api/reference/resources/chat) when many of the output tokens are known ahead of time." and "Predicted Outputs are available today using the latest `gpt-4o`, `gpt-4o-mini`, `gpt-4.1`, `gpt-4.1-mini`, and `gpt-4.1-nano` models."
- Quote: "Note both the `accepted_prediction_tokens` and `rejected_prediction_tokens` in the `usage` object."
- The Responses create reference has no `prediction` body parameter.
- Source: OA-s44, OA-s31. Basis: `Vendor-documented`.

**OA69. OpenRouter takes `prediction` on the chat route.**

- Spec, `Prediction` (a property of `ChatRequest` only): "Static predicted output content. Supported models can use this to reduce latency when much of the response is known in advance." Shape: `type: "content"`, `content` string or text parts.
- Catalog: `prediction` is in `supported_parameters` for nine models: `mistralai/codestral-2508`, `openai/gpt-4o-2024-11-20`, `openai/gpt-4o-2024-08-06`, `openai/gpt-4o-mini`, `openai/gpt-4o-mini-2024-07-18`, `openai/gpt-4o-mini:batch`, `openai/gpt-4o`, `openai/gpt-4o:batch`, `openrouter/auto`. The `openai/gpt-4.1` entries do not list it.
- `ChatUsage.completion_tokens_details` has `accepted_prediction_tokens` and `rejected_prediction_tokens`.
- Source: OA-s17, OA-s18. Basis: `OR-documented`.

**OA70. OpenAI's structured outputs and refusals.**

- Quote (structured outputs guide): "Since a refusal does not necessarily follow the schema you have supplied in `response_format`, the API response will include a new field called `refusal` to indicate that the model refused to fulfill the request."
- Quote (migration guide): "Structured Outputs API shape is different. Instead of `response_format`, use `text.format` in Responses."
- Chat Completions reference: assistant message field "`refusal: optional string or null`" ("The refusal message by the assistant.") and a content part `ChatCompletionContentPartRefusal object { refusal, type }`. Responses: a `refusal` content part on the output message, with stream events `response.refusal.delta` and `response.refusal.done`.
- Source: OA-s51, OA-s36, OA-s32, OA-s33. Basis: `Vendor-documented`.

**OA71. OpenRouter's refusal and strict-mode contract.**

- Quote (errors page): "A refusal that the model produces as its own completed output is not an error. When the provider answers successfully with a refusal (e.g. Anthropic's `stop_reason: "refusal"`), each API keeps its native successful shape: Chat Completions returns `message.refusal` with `finish_reason: "content_filter"`, and the Responses API returns `status: "incomplete"` with `incomplete_details.reason: "content_filter"` and a `refusal` content part."
- Quote (structured outputs page): "**Use strict mode**: Set `strict: true` so that providers with a native strict mode enforce your schema exactly. Enforcement varies by provider: some guarantee schema-conforming output, while others translate your schema into their own structured-output format or treat it as a strong hint, so exact compliance is not guaranteed on every endpoint."
- Spec: `ChatAssistantMessage.refusal` ("Refusal message if content was refused"); `OutputMessage.content` items are `ResponseOutputText` or `OpenAIResponsesRefusalContent`.
- Source: OA-s14, OA-s12, OA-s17. Basis: `OR-documented`.

### K. Status, incomplete reasons, errors

**OA72. OpenAI's response status, incomplete reasons, and stream failure events.**

- Quote (Responses reference, `status`): "The status of the response generation. One of `completed`, `failed`, `in_progress`, `cancelled`, `queued`, or `incomplete`."
- `incomplete_details.reason`: "`optional "max_output_tokens" or "max_messages" or "content_filter" or "steered"`". Quote: "`steered` means the response stopped at a safe output boundary after a WebSocket `response.steer` event."
- `error.code` on the response object: `server_error`, `rate_limit_exceeded`, `invalid_prompt`, `data_residency_mismatch`, `bio_policy`, `misalignment_policy_violation`, `vector_store_timeout`, and 14 image codes (`invalid_image` through `image_file_not_found`). The error object also has optional `misalignment`, `param`, and `type`.
- Stream events: `response.failed` ("An event that is emitted when a response fails."), `response.incomplete` ("An event that is emitted when a response finishes as incomplete."), and a plain `error` event (`ResponseErrorEvent` with `code`, `message`, `param`, `sequence_number`, `type: "error"`).
- Quote (reasoning guide): "If the generated tokens reach the context window limit or the `max_output_tokens` value you've set, you'll receive a response with a `status` of `incomplete` and `incomplete_details` with `reason` set to `max_output_tokens`."
- Source: OA-s31, OA-s33, OA-s34. Basis: `Vendor-documented`.

**OA73. OpenAI's spend-limit and quota codes.**

- Error codes guide table: "429 - Credit balance exhausted | **Code:** `credit_balance_exhausted`", "429 - Slow down | **Type:** `rate_limit_error` <br /> **Code:** `slow_down`", "429 - Organization spend limit reached | **Code:** `organization_spend_limit_exceeded`", "429 - Project spend limit reached | **Code:** `project_spend_limit_exceeded`", "429 - Organization usage limit reached | **Code:** `organization_usage_limit_exceeded`", "503 - Model temporarily overloaded | **Type:** `service_unavailable_error` <br /> **Code:** `server_is_overloaded`".
- Quote: "For billing-related errors, inspect `error.code` to identify the specific cause. The broader `error.type` can still be `insufficient_quota`."
- Quote: "Retrying billing, spend, or quota errors won't restore API access."
- Quote (WebSocket errors): "`previous_response_not_found`: The `previous_response_id` cannot be resolved from available state. Retry with full input context and `previous_response_id` set to `null`."
- Source: OA-s52. Basis: `Vendor-documented`.

**OA74. The context overflow code is `context_length_exceeded`, confirmed from OpenAI's own client, not from its public guides.**

- A search of all 80 OpenAI documentation pages downloaded today (66 guides, the changelog, the deprecations and models pages, and 11 reference pages covering Responses create, retrieve, cancel, compact, input tokens, streaming events, Chat Completions create and streaming, conversations, and webhooks) finds no occurrence of `context_length_exceeded`. The Responses reference's `error.code` list (OA72) does not include it.
- Codex, `codex-rs/codex-api/src/error.rs`, lines 140 to 142: `("context_length_exceeded", None | Some(StatusCode::BAD_REQUEST)) => { Some(ApiError::ContextWindowExceeded) }`.
- Codex, `codex-rs/codex-api/src/sse/responses.rs`, line 1197, a test fixture of the in-stream form: `{"type":"response.failed","sequence_number":3,"response":{"id":"resp_5c66275b97b9baef1ed95550adb3b7ec13b17aafd1d2f11b","object":"response","created_at":1759510079,"status":"failed","background":false,"error":{"code":"context_length_exceeded","message":"Your input exceeds the context window of this model. Please adjust your input and try again."},"usage":null,"user":null,"metadata":{}}}`.
- Same file `error.rs`, lines 106 and 123 to 131: `("insufficient_quota" | "credit_balance_exhausted", None) => Some(ApiError::QuotaExceeded)`, and three spend codes, `"api_key_spend_limit_exceeded"`, `"project_spend_limit_exceeded"`, `"organization_spend_limit_exceeded"`, each matched with `None | Some(StatusCode::TOO_MANY_REQUESTS)`. The API-key code is not in OpenAI's public guide.
- Source: OA-s60. Basis: `Filled` (Codex, OpenAI's own Responses client). Also in workspace: `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30, OA-s68), which reached the same result from an earlier Codex commit and said "OpenAI's own public error guide does not list the code". That still holds today.

**OA75. OpenRouter's error mapping on the Responses route.**

- Quote (Responses error handling page): "These codes are a reduced mapping of the [internal typed error codes](https://openrouter.ai/docs/api_reference/errors-and-debugging.md#typed-error-codes)." and "`context_length_exceeded` and `invalid_request` both surface as `invalid_prompt` here".
- The page's code table: `invalid_prompt` (400), `rate_limit_exceeded` (429), `image_content_policy_violation` (403), `permission_denied` (403), `server_error` (500+).
- Quote: "Because the native `error.code` vocabulary is lossy (many distinct errors collapse to `server_error`), failed responses also include a top-level `error_type` field that carries the precise canonical error type".
- Quote (errors page): "OpenRouter ends a failed stream with a `response.failed` event. The stream schema also defines a plain `error` event (upstream OpenAI behavior), so handling both is safe".
- Quote (errors page, transformations): "Certain token/length error codes reported by the upstream provider are transformed into a completed-but-truncated response instead of a failure." Table: "`context_length_exceeded` (with `param: "input"`, or an overflow message) | terminal failure", "`context_length_exceeded` (output budget) | `status: "incomplete"`, `incomplete_details.reason: "max_output_tokens"`".
- Quote (errors page): "a stream ends with `response.failed` carrying `error.code: "invalid_prompt"`" (for an input overflow).
- Quote (streaming page): "**Responses API**: Upstream token-limit errors (such as `max_tokens_exceeded`) end the stream with a `response.incomplete` event (`incomplete_details.reason: "max_output_tokens"`) rather than an error; other failures end with `response.failed`".
- Quote (GPT-6 guide, policy blocks): "A policy block is an HTTP 403 with `error_type: "refusal"` on every OpenRouter API." and "On the Responses API the same block is a `response.failed` event with `response.error.code` set to the provider code and `error_type: "refusal"` on the response."
- Typed codes on the errors page include `context_length_exceeded` (400), `payment_required` (402), `refusal` (403), `content_policy_violation` (403).
- Source: OA-s2, OA-s14, OA-s15, OA-s8. Basis: `OR-documented`. Agrees with survey claim LC13 on `error_type: context_length_exceeded` and `payment_required`, and adds that on the Responses route the native `error.code` for an overflow is `invalid_prompt`.

**OA76. The enums in OpenRouter's Responses result.**

- `OpenAIResponsesResponseStatus`: `completed`, `incomplete`, `in_progress`, `failed`, `cancelled`, `queued`.
- `IncompleteDetails.reason`: `max_output_tokens`, `content_filter`.
- `ResponsesErrorField.code`: `server_error`, `rate_limit_exceeded`, `invalid_prompt`, `vector_store_timeout`, 14 image codes, `bio_policy`, `cyber_policy`, `misalignment_policy_violation`, `data_residency_mismatch`, `flex_unavailable`, `permission_denied`.
- `ChatFinishReasonEnum`: `tool_calls`, `stop`, `length`, `content_filter`, `error`, `null`.
- Source: OA-s17. Basis: `OR-documented`.

### L. `safety_identifier`, `metadata`, `logprobs`, file inputs

**OA77. OpenAI's rules.**

- Quote (Responses reference, `safety_identifier`): "A stable identifier used to help detect users of your application that may be violating OpenAI's usage policies. The IDs should be a string that uniquely identifies each user, with a maximum length of 128 characters."
- Quote (`user`): "This field is being replaced by `safety_identifier` and `prompt_cache_key`. Use `prompt_cache_key` instead to maintain caching optimizations."
- Quote (`metadata`): "Set of 16 key-value pairs that can be attached to an object." and "Keys are strings with a maximum length of 64 characters. Values are strings with a maximum length of 512 characters."
- Logprobs: Responses `include` value "`message.output_text.logprobs`: Include logprobs with assistant messages." plus `top_logprobs`; Chat Completions `logprobs` and `top_logprobs`.
- Quote (file inputs guide): "File input support depends on the API endpoint. The Responses API accepts the file types listed below as `input_file` items. Chat Completions accepts only PDF files as `file` content parts." The table row for "External file URL (`file_url`)" reads "Not supported" for Chat Completions.
- Source: OA-s31, OA-s32, OA-s55, OA-s56. Basis: `Vendor-documented`.

**OA78. OpenRouter's handling.**

- Quote (spec, `ResponsesRequest.safety_identifier`): "Recommended per-end-user identifier for abuse isolation. Use a stable ID, hash, or pseudonym. When a provider requires a user identity, OpenRouter folds it into the hashed identity sent upstream and never forwards it raw. If omitted, requests use an account-level identity, so provider policy blocks can affect the whole account."
- Quote (spec, `ChatRequest.user`): "Per-end-user identifier for abuse isolation. Use a stable ID, hash, or pseudonym. When a provider requires a user identity, OpenRouter folds it into the hashed identity sent upstream and never forwards it raw."
- Quote (user tracking page): "On the Responses API, `safety_identifier` sets only the upstream abuse-isolation identity."
- Quote (spec, `RequestMetadata`): "Metadata key-value pairs for the request. Keys must be ≤64 characters and cannot contain brackets. Values must be ≤512 characters. Maximum 16 pairs allowed."
- Spec: `ChatRequest` has `logprobs` ("Return log probabilities") and `top_logprobs` ("Number of top log probabilities to return (0-20)"); `ResponsesRequest` has `top_logprobs`; OpenAI's `message.output_text.logprobs` include value is outside OpenRouter's `include` enum (OA4). Catalog: `logprobs` is in `supported_parameters` for 19 `openai/` models, none of them in the GPT-5.x or GPT-6 text families.
- Source: OA-s17, OA-s29, OA-s18. Basis: `OR-documented`.

### M. Provider differences inside OpenRouter for one OpenAI model

**OA79. What the documentation says differs between OpenAI direct, Azure, and Bedrock.**

- Reasoning format tags per provider (OA8), and an Azure-served `openai/gpt-5-mini` example whose reasoning item has `"format": "azure-openai-responses-v1"` and an endpoint tag ending in `|azure` (OA15).
- Pro mode: "`reasoning.mode` is only supported by OpenAI GPT-5.6 and newer, when served by OpenAI or Azure. Amazon Bedrock's OpenAI-compatible API accepts the field but silently ignores it, so pro reasoning is not available on Bedrock" (reasoning tokens page).
- Async tools: OpenAI and Azure honor them, Bedrock runs them synchronously (OA43).
- ZDR group row (ZDR page): "**OpenAI** | Removes first-party OpenAI endpoints (Azure remains available)".
- Access programs (spec, `ResponsesRequest.access_programs`): "A program other than `standard` is supported only on BYOK requests that run on your own OpenAI key enrolled in that program; any other endpoint, including OpenRouter's own OpenAI credentials, rejects it with a 400 error."
- Sticky routing (prompt caching page): "Subsequent requests for the same model are routed to the same provider, keeping your cache warm." and "Sticky routing is not used when you specify a manual [provider order](https://openrouter.ai/docs/guides/routing/provider-selection.md) via `provider.order`".
- Source: OA-s9, OA-s25, OA-s26, OA-s17, OA-s10. Basis: `OR-documented`. The documentation does not say whether a reasoning blob minted on the OpenAI endpoint is accepted, dropped, or rejected when the next turn is served by Azure (see Open questions).

### N. Fills from other projects (how others solve what OpenRouter's documentation leaves unsaid)

A fill shows that a solution exists and how that project does it. It does not show what OpenRouter does.

**OA80. LiteLLM's bridge from chat completions to the Responses API keeps Responses state on the chat message in three places.**

- Repository `BerriAI/litellm` at commit `90dce4b93bd2b090621a6b9083311294d3358bc6`, folder `litellm/completion_extras/litellm_responses_transformation/`.
- Reasoning items on a chat message, `transformation.py` lines 373 to 386: function `_reasoning_input_items` reads `msg.get("reasoning_items")` and falls back to `msg.get("thinking_blocks")`. Its docstring: "Stored reasoning items win because they carry an id the Responses API minted; thinking blocks are the fallback for turns that arrived over another API surface." Lines 409 to 414 build each stored item as `{"id": item_id, "type": "reasoning", "encrypted_content": encrypted_content, "summary": summary}`.
- An order map with a checksum, `history.py` lines 11 to 12 and 58 to 75: `REPLAY_FIELD: Final = "openai_responses_replay"` and `PHASE_FIELD: Final = "phase"`; class `_Replay` has `version: Literal[1]`, `items: tuple[_Entry, ...]`, `checksum: str`; an entry is either `_MessageEntry` (`type: Literal["message"]`, `parts` holding the text length of each output part, `phase`) or `_IndexedEntry` (`type: Literal["reasoning", "function_call", "custom_tool_call", "tool_search_call", "tool_search_output"]`, `index`). The field lives in the message's `provider_specific_fields`.
- The checksum, `history.py` lines 228 to 242: a SHA-256 over `content`, `tool_calls` (id, name, arguments), `reasoning` (id, `encrypted_content`, summary texts), `tool_search_items`, `phase`, and the entries. Lines 309 to 310 skip the replay record unless the output items match the chat message: `if not completed or not items or len(items) != len(parsed) or chat is None or not _matches(items, chat): return phase_fields`.
- Which providers get replay, `transformation.py` lines 271 to 272: `_HISTORY_REPLAY_PROVIDERS: Final = frozenset({"openai", "azure"})` and `_SYSTEM_INPUT_PROVIDERS: Final = frozenset({"openai", "azure", "azure_ai", "bedrock_mantle"})`.
- System messages, lines 1218 to 1224: leading system strings are joined into `instructions` ("# Concatenate multiple system prompts with a space") when `preserve_system_instructions` is true. Line 1482 sets it with `preserve_system_instructions=preserve_system_instructions or provider not in _SYSTEM_INPUT_PROVIDERS`, so for OpenAI and Azure the system text stays in `input` as a message item unless `truncation` is `auto`.
- `previous_response_id` on the chat route, lines 273 to 276 (two adjacent string literals, joined here as Python joins them): "`previous_response_id` is only supported on the Responses API (`/v1/responses`). Continue the stored response there, or send the earlier turns in `messages`."
- `strict` is passed through as given, line 2116: `strict=function_tool.get("strict"),`.
- Source: OA-s61. Basis: `Filled` (LiteLLM).

**OA81. The Vercel AI SDK keeps item ids, phase, and encrypted reasoning in per-part provider metadata.**

- Repository `vercel/ai` at commit `d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d`, file `packages/openai/src/responses/convert-to-openai-responses-input.ts`.
- Lines 820 to 825, on a text part: `const id = providerOptions?.itemId as string | undefined;` and `const phase = providerOptions?.phase as | 'commentary' | 'final_answer' | null | undefined;`. Lines 845 to 850 send `role: 'assistant', content: part.text, ...(phase != null && { phase })`.
- Lines 832 to 841, when the response was stored: "// item references reduce the payload size" and `input.push({ type: 'item_reference', id });`.
- Lines 1403 to 1410, when it was not stored, on a reasoning part: `reasoningMessages[reasoningId] = { type: 'reasoning', id: reasoningId, encrypted_content: providerOptions?.reasoningEncryptedContent, summary: summaryParts, };`.
- Lines 1423 to 1426, comment: "The OpenAI Responses API accepts reasoning items without an id when encrypted_content is provided, enabling multi-turn reasoning even when server-side item persistence is not used or when itemId has been stripped from providerOptions." (comment markers removed)
- Line 1449, for reasoning from another vendor: "Non-OpenAI reasoning parts are not supported. Skipping reasoning part".
- Source: OA-s62. Basis: `Filled` (Vercel AI SDK).

**OA82. Codex, OpenAI's reference Responses client, runs stateless over HTTP and stores every output item.**

- Repository `openai/codex` at commit `227fa4e805d44988b0888390f41afe1358e9d24d`.
- `codex-rs/core/src/client.rs` lines 1326 to 1329: `let mut include = vec!["reasoning.encrypted_content".to_string()];` and `if self.state.output_token_replay_enabled && is_openai { include.push("output.encrypted_content".to_string()); }`.
- Same file lines 1367 to 1383, the request: `tool_choice: "auto".to_string()`, `store: false`, `stream: true`, `include`, `service_tier`, `prompt_cache_key: self.prompt_cache_key(responses_metadata)`, `prompt_cache_options: prompt.prompt_cache_ttl.map(|ttl| codex_api::PromptCacheOptions { ttl })`.
- `codex-rs/protocol/src/models.rs` lines 1187 to 1200: the stored item `Reasoning { id, summary, content, encrypted_content: Option<String>, ... }`. Lines 1152 to 1170 and 1218 to 1221: `Message` and `FunctionCall` items also have an `encrypted_content: Option<String>` field, and `Message` has `phase: Option<MessagePhase>`.
- Same file lines 1053 to 1066: `pub enum MessagePhase { Commentary, PartialAnswer, FinalAnswer }` with the comment "Providers do not emit this consistently, so callers must treat `None` as "phase unknown" and keep compatibility behavior for legacy models."
- Source: OA-s60. Basis: `Filled` (Codex). `partial_answer` is a third phase value that OpenAI's public reference does not list (see Conflicts).

**OA83. llama.cpp serves `/v1/responses` by converting to chat completions and refuses server state.**

- Repository `ggml-org/llama.cpp` at commit `8d60873dfb8619f2b5ed31adfbab07cccb40db5c`, file `tools/server/server-chat.cpp`.
- Lines 32 to 38: `json server_chat_convert_responses_to_chatcmpl(const json & response_body)` and `throw std::invalid_argument("llama.cpp does not support 'previous_response_id'.");`.
- Lines 45 to 51: `instructions` becomes a `{"role", "system"}` message. Lines 257 to 281: a `reasoning` input item becomes `reasoning_content` on the assistant message, taken from `item.at("content")[0].at("text")`, and an item whose `content` is empty is rejected with "item['content'] is empty". Lines 305 to 311: "// Only "effort" is handled so far" and `chatcmpl_body["reasoning_effort"] = reasoning.at("effort");`.
- `tools/server/README.md` line 1709: "`extra_params`: llama.cpp parameters that the Responses API has no field for, such as `seed`, `min_p` or `repeat_penalty`, as an object".
- Source: OA-s63. Basis: `Filled` (llama.cpp). An OpenAI reasoning item with only `encrypted_content` would be rejected here, since the conversion requires plain reasoning text.

**OA84. vLLM implements `store`, `previous_response_id`, and background mode with an in-memory store that is off by default.**

- Repository `vllm-project/vllm` at commit `e2052f5c7ad64564f00dd4592e77c27785c9333f`, file `vllm/entrypoints/openai/responses/serving.py`.
- Lines 165 to 174: "NOTE(woosuk): This may not be intuitive for users, as the default behavior in OpenAI's Responses API is to store the response, but vLLM's default behavior is not." and the warning "`VLLM_ENABLE_RESPONSES_API_STORE` is enabled. This may cause a memory leak since we never remove responses from the store." (adjacent string literals in the source, joined here)
- Lines 259 to 271 (adjacent string literals, joined here): "This vLLM engine does not support `store=True` and therefore does not support the background mode. To enable these features, set the environment variable `VLLM_ENABLE_RESPONSES_API_STORE=1` when launching the vLLM server."
- Lines 349 to 356: "Although returning an error is possible, we opted to implicitly disable store and process the request anyway, as we assume most users do not intend to actually store the response (i.e., their request's `store=True` just because it's the default value)."
- Lines 359 to 364: an unknown `previous_response_id` returns a not-found error.
- Source: OA-s64. Basis: `Filled` (vLLM). vLLM downgrades `store: true` silently, where OpenRouter rejects it with a 400 (OA1).

**OA85. LM Studio's `/v1/responses` supports `previous_response_id`.**

- Quote: "Create responses with support for streaming, reasoning, prior response state, and optional Remote MCP tools." and "Use the `id` from a previous response as `previous_response_id`."
- Source: OA-s65. Basis: `Filled` (LM Studio).

## Conflicts and disagreements

**Between this file and the local Runvil documents**

- **`invalid_encrypted_content` (workspace, 2026-09-20) against today's OpenAI documentation.** `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (OA-s67) says of OpenAI Responses: "`invalid_encrypted_content` for cross-org or cross-provider blobs; missing reasoning is silent degradation". Today's OpenAI documentation describes a different outcome for a blob from another organization: the API removes the reasoning item and reports it in `input_transformations` with reason `organization_mismatch` (OA27). The string `invalid_encrypted_content` appears in none of the 80 OpenAI pages downloaded today and in none of the Codex source folders cloned. Which is right today: OpenAI's documentation is the primary source, so "dropped and reported" is the documented behavior. A blob that is not an OpenAI blob at all (for example one minted by another vendor) may still produce a hard error; that case is not documented and was not tested. Treat the workspace claim as unverified.
- **Survey claim LC17 needs a new reading.** The claim is that `prompt_tokens_details.cache_write_tokens` is "the extension name OpenRouter uses for cache writes". OpenRouter does use that name (OA63). OpenAI's own Chat Completions reference now has the same field with the same name (OA65), so it is a shared name rather than an OpenRouter extension.
- **Survey claim LC3 has documented exceptions in this topic.** "Unsupported parameters are silently ignored, not rejected" holds for `async` on the chat route, `reasoning.mode`, `prompt_cache_breakpoint`, `prompt_cache_options`, and `phase` on older OpenAI models, all of which are stripped or ignored (OA33, OA41, OA43). These are rejected with a 400 instead: `store: true` and `previous_response_id` (OA1), a `configuration_update` for a model that does not support it or on a non-system message (OA45), `file_search` on OpenRouter credits (OA51), a non-standard `access_programs` value without BYOK (OA79), and several hosted tools together with `deferred_tools` (OA56).
- **Survey claim LC1 undercounts by one.** The catalog's `endpoints` arrays list four inference routes per model: `/api/v1/chat/completions`, `/api/v1/completions`, `/api/v1/messages`, `/api/v1/responses` (OA5).
- **Survey claim LC4 holds** (OA1). **LC6 holds** and extends to OpenAI models (OA18). **LC7 holds** (OA63). **LC13 holds** for the typed `error_type` values, with the Responses-route detail in OA75.
- **The survey's description of Responses needs one update.** `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` describes Responses with "the system prompt is top-level `instructions`". From GPT-6.2 (2026-11-17) OpenAI appends `instructions` after `input` and bills it uncached, and tells callers to put reusable instructions in a first `developer` message (OA62).

**Inside OpenRouter's own documentation**

- **`verbosity` on the chat route.** The parameters page documents a chat key `verbosity` with enum "(low, medium, high, xhigh, max)", and the catalog lists it for 114 models. In the OpenAPI spec the only `verbosity` is `TextConfig.verbosity` on the Responses route, with `high`, `low`, `medium`, so the spec and the parameters page disagree about the chat route and about the two higher values (OA33).
- **The error code for a policy block on the Responses route.** The errors page table maps internal `refusal` to Responses code `invalid_prompt`. The GPT-6 guide says "`response.error.code` set to the provider code" (`bio_policy` or `cyber_policy`), and the spec's code enum includes both (OA75, OA76).
- **Effort levels.** The Responses reasoning page lists four (`minimal`, `low`, `medium`, `high`). The reasoning tokens page and the spec list seven (OA33).
- **The `phase` model table is behind the GPT-6 guide.** The GPT-5.4 page lists five supported models and "Other OpenAI models | Silently ignored". The GPT-6 guide's live output from `openai/gpt-6-astra` has `"phase": "final_answer"` (OA41, OA43 source page).
- **`include`.** The reasoning tokens page says "no explicit `include` field is required". The GPT-5.6 guide's example still sends `"include": ["reasoning.encrypted_content"]`. Both work per OA25.
- **The reasoning summary shape.** One example on the Responses reasoning page shows `summary` as an array of plain strings. The spec and OpenAI define it as objects with `type: "summary_text"` and `text`.

**Inside OpenAI's own material**

- **Which models use `phase`.** The reference says "models like `gpt-5.3-codex` and beyond". The reasoning guide says "GPT-5.5 and GPT-5.4". Codex's enum has a third value, `PartialAnswer` (OA82), that the public reference does not list.
- **When `encrypted_content` is on by default.** The reference says for every `POST /v1/responses`. The guide says in stateless mode. The migration guide's sample output shows a reasoning item with no `encrypted_content` field (OA25).
- **The overflow code.** Codex handles `context_length_exceeded` inside `response.failed`, and the Responses reference's `error.code` list does not include it (OA72, OA74).

## Open questions blocked by closed source

1. **Does the chat route send every OpenAI model to the Responses API upstream, or only reasoning models?** Tried: the documentation mirror, the changelog, the OpenAPI spec, the catalog, the official AI SDK provider. Result: one documented sentence for GPT-6 `configuration_update` (OA9) and an inference from five signals (OA10). The `debug.echo_upstream_body` option would show the upstream body, and it needs an authenticated request, which this research does not make. Fill found: LiteLLM routes chat requests through a Responses bridge and replays history for `openai` and `azure` (OA80), which shows the design is workable.
2. **Where does the chat route put the text of several assistant messages (`commentary` then `final_answer`) and several reasoning items from one Responses output?** OpenRouter says only that the chat format "cannot represent" them (OA41). Whether the texts are concatenated into `content`, and whether the order of reasoning items against tool calls survives in `reasoning_details`, is not documented. Fill found: LiteLLM concatenates message text, keeps `reasoning_items`, and records the original item order in `openai_responses_replay` with a checksum (OA80). The Vercel AI SDK keeps separate parts with `itemId` and `phase` each (OA81).
3. **What is the endpoint tag appended to encrypted reasoning for, is it removed before the blob goes to OpenAI, and what happens when the next turn is served by a different provider (OpenAI then Azure, or after failover)?** Tried: every page that mentions encrypted reasoning, the changelog, the SDK source. The tag is only visible in examples (OA15). Fill found: OpenAI's own rule is that a blob from another organization is removed and reported (OA27). LiteLLM replays Responses history only when the target provider is `openai` or `azure` (OA80). The Vercel AI SDK skips reasoning parts that are not OpenAI's with a warning (OA81). No fill shows OpenRouter's own behavior.
4. **How does the chat route place a system message upstream for OpenAI: top-level `instructions` or a `developer` message in `input`?** This decides cache behavior on GPT-6.2 and later (OA62). Not documented. Fill found: LiteLLM keeps system text in `input` for `openai`, `azure`, `azure_ai`, and `bedrock_mantle`, and uses `instructions` for other providers (OA80).
5. **What does the Responses route do with `background: true`?** The schema accepts the boolean and nothing else is said (OA58). Fill found: vLLM ties background mode to an in-memory response store and refuses it when the store is off (OA84). No fill shows OpenRouter's behavior.
6. **Does OpenRouter send `store: false` to OpenAI, and which `include` values?** Documented in part: "no responses are stored" (OA1) and "automatically requests encrypted reasoning content" (OA14). The exact upstream fields are not shown. Fill found: Codex sends `store: false` with `include: ["reasoning.encrypted_content"]` (OA82).
7. **When OpenRouter moves a chat request to the Responses API, does it keep chat's non-strict default for functions?** OpenAI's two APIs differ (OA39). Not documented by OpenRouter. Fill found: LiteLLM passes `strict` through as the caller gave it (OA80), so an omitted value takes the Responses default.
8. **Which engine serves OpenAI-shaped hosted tools (`mcp`, `image_generation`, `computer_use_preview`, `code_interpreter` with `auto`) on the Responses route for a non-OpenAI model?** The shell page answers for `shell` (OA52) and the spec answers for BYOK limits (OA51). The others have schemas and no prose. No fill found for OpenRouter; the server-tools evidence file may hold more.

## Unverified

- **Remembered, not found in today's sources:** that OpenAI's own Chat Completions returns `finish_reason: "stop"` together with `message.refusal` for a model-written refusal. The pages read today do not state the finish reason. OpenRouter documents `content_filter` for its own chat route (OA71).
- **Remembered, not found:** that a reasoning item must be followed by its paired output item, or OpenAI returns a 400 that names the reasoning item and says its required following item is absent. The wording is from memory and is not a quote. Today's documentation only says to keep items in order (OA26). The workspace research on tool-call replay may hold a source for this.
- **Not verified live:** every statement about what OpenRouter's server does comes from documentation, the spec, the catalog, or SDK fixtures. No request was sent.
- **Not verified:** whether the OpenRouter spec's missing tool types (`computer`, `programmatic_tool_calling`, native `tool_search`) are rejected by the server or accepted as unknown values. The tool search page says "Each API's native spelling is accepted as an alias", which may cover `tool_search`.
- **Not verified:** OpenAI's capability table in the migration guide (Chat Completions against Responses). The Markdown form of the page drops the check marks, so the table cells came through empty. The surrounding prose was used instead (OA38).
- **Not examined:** the Conversations reference (985 KB), the WebSocket mode guide, the multi-agent guide, the steering guide, the deprecations page, and the data residency guide were downloaded and only searched for specific strings.
- **Model coverage:** OpenAI's statements about GPT-6.1 Terra and GPT-6 Sol and Luna were quoted as written. Per-model tool-calling support on Chat Completions was not cross-checked against each model page.

## Workspace documents used

- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took: the Responses overview (typed items, `instructions`, `status` plus `incomplete_details.reason`), the `encrypted_content` default, and the OpenRouter `/responses` rejection of `previous_response_id` and `store`. Re-checked live: yes, all three confirmed (OA25, OA1), with one update on `instructions` for GPT-6.2 (OA62).
- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20). Took: Appendix B's row for OpenAI Responses ("Replay `reasoning` items with `encrypted_content`; omission degrades, foreign blobs fail", "`call_id` pairing", `invalid_encrypted_content`). Re-checked live: replay and `call_id` pairing confirmed (OA26, OA42). The `invalid_encrypted_content` string was not found in today's OpenAI documentation, see Conflicts.
- `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30). Took: the finding that OpenAI's public error guide does not list `context_length_exceeded` and that Codex confirms it, and the OpenRouter `error_type` value. Re-checked live: yes, against today's OpenAI pages and a fresh Codex commit (OA74), and against OpenRouter's errors page (OA75).
- 2026-11-22 research series "conduct-research-fake-llm" (cabinet research, title "fake LLM agent harness tests"). Seen in a workspace search for `context_length_exceeded`; nothing was taken from it.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
| - | - | - | - |
| OA-s1 | https://openrouter.ai/docs/api_reference/responses/overview | 2026-11-22 | OpenRouter doc |
| OA-s2 | https://openrouter.ai/docs/api_reference/responses/error-handling | 2026-11-22 | OpenRouter doc |
| OA-s3 | https://openrouter.ai/docs/api_reference/responses/reasoning | 2026-11-22 | OpenRouter doc |
| OA-s4 | https://openrouter.ai/docs/api_reference/responses/basic-usage | 2026-11-22 | OpenRouter doc |
| OA-s5 | https://openrouter.ai/docs/api_reference/responses/tool-calling and https://openrouter.ai/docs/api_reference/responses/web-search (read for coverage; encrypted reasoning examples) | 2026-11-22 | OpenRouter doc |
| OA-s6 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-5-4 | 2026-11-22 | OpenRouter doc |
| OA-s7 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-5-6 | 2026-11-22 | OpenRouter doc |
| OA-s8 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-6 | 2026-11-22 | OpenRouter doc |
| OA-s9 | https://openrouter.ai/docs/guides/best-practices/reasoning-tokens | 2026-11-22 | OpenRouter doc |
| OA-s10 | https://openrouter.ai/docs/guides/best-practices/prompt-caching | 2026-11-22 | OpenRouter doc |
| OA-s11 | https://openrouter.ai/docs/guides/features/service-tiers | 2026-11-22 | OpenRouter doc |
| OA-s12 | https://openrouter.ai/docs/guides/features/structured-outputs | 2026-11-22 | OpenRouter doc |
| OA-s13 | https://openrouter.ai/docs/api_reference/parameters | 2026-11-22 | OpenRouter doc |
| OA-s14 | https://openrouter.ai/docs/api_reference/errors-and-debugging | 2026-11-22 | OpenRouter doc |
| OA-s15 | https://openrouter.ai/docs/api_reference/streaming | 2026-11-22 | OpenRouter doc |
| OA-s16 | https://openrouter.ai/docs/api_reference/overview | 2026-11-22 | OpenRouter doc |
| OA-s17 | https://openrouter.ai/openapi.json (version 1.0.0, 123 paths, 1071 schemas) | 2026-11-22 | OpenRouter spec |
| OA-s18 | https://openrouter.ai/api/v1/models (476 models) | 2026-11-22 | OpenRouter spec (model catalog) |
| OA-s19 | https://openrouter.ai/docs/cookbook/coding-agents/codex-cli | 2026-11-22 | OpenRouter doc |
| OA-s20 | https://openrouter.ai/docs/cookbook/coding-agents/codex-desktop | 2026-11-22 | OpenRouter doc |
| OA-s21 | https://openrouter.ai/docs/guides/features/server-tools | 2026-11-22 | OpenRouter doc |
| OA-s22 | https://openrouter.ai/docs/guides/features/server-tools/tool-search | 2026-11-22 | OpenRouter doc |
| OA-s23 | https://openrouter.ai/docs/guides/features/server-tools/shell | 2026-11-22 | OpenRouter doc |
| OA-s24 | https://openrouter.ai/docs/guides/features/server-tools/apply-patch | 2026-11-22 | OpenRouter doc |
| OA-s25 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| OA-s26 | https://openrouter.ai/docs/guides/features/zdr | 2026-11-22 | OpenRouter doc |
| OA-s27 | https://openrouter.ai/docs/batch-quickstart and https://openrouter.ai/docs/guides/features/batch-api | 2026-11-22 | OpenRouter doc |
| OA-s28 | https://openrouter.ai/docs/changelog (entries dated 2026-07-25, 2026-09-04, 2026-09-08, 2026-09-11, 2026-09-23, 2026-11-04, 2026-11-17, 2026-11-18) | 2026-11-22 | OpenRouter doc |
| OA-s29 | https://openrouter.ai/docs/cookbook/administration/user-tracking | 2026-11-22 | OpenRouter doc |
| OA-s30 | `OpenRouterTeam/ai-sdk-provider` at `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20): `e2e/issues/issue-519-streaming-summary-merge.test.ts`, `src/chat/index.ts`, `src/schemas/format.ts`, `src/chat/signature-roundtrip.test.ts` | 2026-11-22 | OpenRouter SDK source |
| OA-s31 | https://developers.openai.com/api/reference/resources/responses/methods/create.md | 2026-11-22 | vendor doc |
| OA-s32 | https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create.md | 2026-11-22 | vendor doc |
| OA-s33 | https://developers.openai.com/api/reference/resources/responses/streaming-events.md | 2026-11-22 | vendor doc |
| OA-s34 | https://developers.openai.com/api/docs/guides/reasoning.md | 2026-11-22 | vendor doc |
| OA-s35 | https://developers.openai.com/api/docs/guides/conversation-state.md | 2026-11-22 | vendor doc |
| OA-s36 | https://developers.openai.com/api/docs/guides/migrate-to-responses.md | 2026-11-22 | vendor doc |
| OA-s37 | https://developers.openai.com/api/docs/guides/latest-model/gpt-6-astra.md | 2026-11-22 | vendor doc |
| OA-s38 | https://developers.openai.com/api/docs/guides/latest-model/gpt-6.2-astra.md | 2026-11-22 | vendor doc |
| OA-s39 | https://developers.openai.com/api/docs/guides/latest-model/gpt-5.4.md | 2026-11-22 | vendor doc |
| OA-s40 | https://developers.openai.com/api/docs/changelog.md (entries Sep 3 and Nov 17, 2026) | 2026-11-22 | vendor doc |
| OA-s41 | https://developers.openai.com/api/docs/guides/keep-prefix-unchanged.md | 2026-11-22 | vendor doc |
| OA-s42 | https://developers.openai.com/api/docs/guides/background.md | 2026-11-22 | vendor doc |
| OA-s43 | https://developers.openai.com/api/docs/guides/prompt-caching.md | 2026-11-22 | vendor doc |
| OA-s44 | https://developers.openai.com/api/docs/guides/predicted-outputs.md | 2026-11-22 | vendor doc |
| OA-s45 | https://developers.openai.com/api/docs/guides/flex-processing.md | 2026-11-22 | vendor doc |
| OA-s46 | https://developers.openai.com/api/docs/guides/fast-mode.md | 2026-11-22 | vendor doc |
| OA-s47 | https://developers.openai.com/api/docs/guides/ultrafast-mode.md | 2026-11-22 | vendor doc |
| OA-s48 | https://developers.openai.com/api/docs/guides/compaction.md | 2026-11-22 | vendor doc |
| OA-s49 | https://developers.openai.com/api/docs/guides/async-tool-calling.md | 2026-11-22 | vendor doc |
| OA-s50 | https://developers.openai.com/api/docs/guides/function-calling.md | 2026-11-22 | vendor doc |
| OA-s51 | https://developers.openai.com/api/docs/guides/structured-outputs.md | 2026-11-22 | vendor doc |
| OA-s52 | https://developers.openai.com/api/docs/guides/error-codes.md | 2026-11-22 | vendor doc |
| OA-s53 | https://developers.openai.com/api/docs/guides/tools-web-search.md, tools-file-search.md, tools-code-interpreter.md, tools-computer-use.md, tools-image-generation.md, tools-connectors-mcp.md, tools-shell.md, tools-apply-patch.md, tools-tool-search.md, tools-programmatic-tool-calling.md (same folder) | 2026-11-22 | vendor doc |
| OA-s54 | https://developers.openai.com/api/docs/guides/batch.md | 2026-11-22 | vendor doc |
| OA-s55 | https://developers.openai.com/api/docs/guides/file-inputs.md | 2026-11-22 | vendor doc |
| OA-s56 | https://developers.openai.com/api/docs/guides/safety-best-practices.md | 2026-11-22 | vendor doc |
| OA-s57 | https://developers.openai.com/api/docs/guides/webhooks.md and https://developers.openai.com/api/reference/resources/webhooks.md | 2026-11-22 | vendor doc |
| OA-s58 | https://developers.openai.com/api/docs/guides/your-data.md | 2026-11-22 | vendor doc |
| OA-s59 | https://developers.openai.com/api/docs/llms.txt and https://developers.openai.com/api/reference/llms.txt (indexes; `https://platform.openai.com/docs/llms.txt` redirects to the first) | 2026-11-22 | vendor doc |
| OA-s60 | `openai/codex` at `227fa4e805d44988b0888390f41afe1358e9d24d` (2026-11-22): `codex-rs/core/src/client.rs`, `codex-rs/codex-api/src/error.rs`, `codex-rs/codex-api/src/sse/responses.rs`, `codex-rs/protocol/src/models.rs` | 2026-11-22 | harness source |
| OA-s61 | `BerriAI/litellm` at `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22): `litellm/completion_extras/litellm_responses_transformation/transformation.py`, `history.py` | 2026-11-22 | gateway source |
| OA-s62 | `vercel/ai` at `d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d` (2026-11-20): `packages/openai/src/responses/convert-to-openai-responses-input.ts` | 2026-11-22 | harness source |
| OA-s63 | `ggml-org/llama.cpp` at `8d60873dfb8619f2b5ed31adfbab07cccb40db5c` (2026-11-22): `tools/server/server-chat.cpp`, `tools/server/README.md` | 2026-11-22 | gateway source (local server) |
| OA-s64 | `vllm-project/vllm` at `e2052f5c7ad64564f00dd4592e77c27785c9333f` (2026-11-23 +08:00): `vllm/entrypoints/openai/responses/serving.py` | 2026-11-22 | gateway source (local server) |
| OA-s65 | https://lmstudio.ai/docs/developer/openai-compat/responses | 2026-11-22 | vendor doc (LM Studio) |
| OA-s66 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28) | 2026-11-22 | workspace document |
| OA-s67 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20) | 2026-11-22 | workspace document |
| OA-s68 | `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30) | 2026-11-22 | workspace document |
