---
produced: 2026-11-22
title: OpenRouter API surfaces and request superset - chat completions, Responses, and Anthropic Messages routes, field-by-field inventory against the OpenAI and Anthropic references, unsupported parameter handling, headers, versioning, official SDK cross-check
---

# OpenRouter API surfaces and request superset - chat completions, Responses, and Anthropic Messages routes, field-by-field inventory against the OpenAI and Anthropic references, unsupported parameter handling, headers, versioning, official SDK cross-check

- What was read: OpenRouter's OpenAPI 3.1 spec (123 paths, 1071 schemas, walked with Python), 51 OpenRouter documentation pages and the dated changelog, the public model catalog (476 models), four official OpenRouter SDK repositories at pinned commits, the official OpenAI and Anthropic Python SDK type sources at pinned commits, OpenAI's live chat completions reference page, and LiteLLM source for fills.
- Sources: 66, listed in the Sources table at the end. Evidence items: 69 (S1 to S69). Candidate matrix rows: 25.
- Read date: 2026-11-22. Every read was public and unauthenticated. Nothing was run against a model.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Three wire protocols over one model pool | OpenAI, Anthropic | OpenAI: `/chat/completions` and `/responses`. Anthropic: `/messages`. | All three under `https://openrouter.ai/api/v1`, plus legacy `/completions`. Each of the 384 chat models lists all four in its catalog `endpoints`. | Each route keeps its own envelope. Some features sit on one route only (rows 6, 7, and 14 to 17). | Route is the caller's choice and does not depend on the vendor. The chat route reaches every model but not every feature. | OR-documented | S1, S5, S6, S7, S8, S39 |
| Server-held conversation state | OpenAI Responses | `previous_response_id`, `store`, `conversation` keep state at the vendor | `/responses` rejects a non-null `previous_response_id` and `store: true` with a 400. `conversation` is outside the schema. | Vendor-side state. The caller resends the full history, reasoning items included. | State holder: caller. Round trip: full history on every request. | OR-documented | S26, S27 |
| Encrypted reasoning blob | OpenAI, Azure OpenAI | Responses reasoning item `encrypted_content`, replayed unchanged | Chat: `reasoning_details[]` item `type: "reasoning.encrypted"` with `data` and `format: "openai-responses-v1"`. Responses: `encrypted_content` plus `format`. Messages: `redacted_thinking.data`. | The blob gains a `.` plus base64 `{"endpoint_slug":"<model>\|<provider>"}` suffix. On Messages the summary becomes a `thinking` block with `signature: ""`. | Round trip through the caller unchanged: yes. The gateway tags the blob with model version and provider. | OR-documented (fields), OR-inferred (suffix) | S16, S28, S31, S38 |
| Vendor tag on reasoning | All reasoning vendors | Each vendor has its own block format | `format` on every `reasoning_details` item and Responses reasoning item, 12 named values such as `anthropic-claude-v1`, `google-gemini-v1` | Open enum, unknown values allowed | The caller stores the tag unread and sends it back. | OR-documented | S16, S57 |
| Thinking on or off, budget, effort | Anthropic, OpenAI, Google | Anthropic: `thinking` and `output_config.effort`. OpenAI: `reasoning.effort`. | Chat and Responses: `reasoning: {effort, max_tokens, enabled, exclude}`. Messages: native `thinking` and `output_config`, translated to `reasoning` for every model. | `effort` and `max_tokens` cannot both reach non-Claude models. Budgets are not forwarded to newer Claude models. `minimal` is sent to Claude as `low`. | A neutral setting translates. No vendor value is needed for on, off, and effort. | OR-documented | S17, S35, S49 |
| Thinking display, `between_tools`, prefix-lock control | Anthropic | `thinking.display`, `thinking.type: "between_tools"`, `thinking.block_binding` | Messages route only | Chat offers `reasoning.exclude` only | Exists only on the non-chat route. | OR-documented | S35, S36 |
| Mid-conversation tool changes and ephemeral system messages | Anthropic | `tool_addition` and `tool_removal` blocks; system message with `clear_at`; beta header | Messages route only. OpenRouter attaches the beta header itself and routes only to providers that support it. | "the Chat Completions API has no representation for these blocks" | Exists only on the non-chat route. The marker stays in the transcript and is resent. | OR-documented | S36 |
| Mid-conversation effort change | OpenAI (GPT-6), Anthropic (Fable 5.1 and later) | OpenAI: `configuration_update` input item. Anthropic: system message with `output_config.effort`. | Chat: `configuration_update` on a content-less `system` or `developer` message (OpenRouter extension). Native forms on the other two routes. All three interchangeable. | A model without support gets a 400 | The caller places a neutral marker in history and keeps its position. No vendor value. | OR-documented | S13, S39, S47 |
| Anthropic beta headers | Anthropic | `anthropic-beta` header | `x-anthropic-beta` request header, forwarded by a per-provider allowlist. Several betas are attached automatically. `strict` on tools is stripped without the header. | Values off the list are dropped | The caller names vendor beta strings unless the gateway detects the feature. | OR-documented; Filled (LiteLLM) for the mapping design | S51, S64 |
| Cache breakpoints | Anthropic, OpenAI | Anthropic: `cache_control` on blocks. OpenAI: `prompt_cache_breakpoint` on parts. | Both markers on chat text parts. `cache_control` also at the top level of all three routes and on function tools. Converted per provider. | In the spec the markers sit on text parts only | A neutral marker on the message part. The gateway translates. | OR-documented | S10, S14, S15 |
| Unsupported parameter | All | OpenAI and Anthropic reject | Dropped for providers that lack it. `provider.require_parameters: true` filters providers. `tools`, response format, and `verbosity` steer routing by default. 22 documented rejection cases. | Silent drop by default | Decided inside the gateway. Catalog `supported_parameters` gives the per-model list. | OR-documented | S41, S43, S44, S45, S47 |
| Finish reason | All | Vendor-specific strings | `finish_reason` normalized to `tool_calls`, `stop`, `length`, `content_filter`, `error`. Raw value in `native_finish_reason`. | `native_finish_reason` is outside the spec's `ChatChoice` and the generated SDK types. Messages returns `stop_reason` only. | Gateway only. | OR-documented | S18, S19, S37 |
| Serving provider identity | Aggregator feature | Not applicable | Wire `provider` on chat and Messages responses. `openrouter_metadata` with header `X-OpenRouter-Metadata: enabled`. `X-Generation-Id` response header. | `provider` is outside the spec's `ChatResult` | The caller needs it only to pin a replay or to debug. | OR-documented | S19, S37, S50, S52 |
| Parsed PDF reuse | OpenRouter feature over all vendors | Not applicable | Chat: `message.annotations[]` items of `type: "file"` with `hash`, sent back in later requests | Responses returns `file_citation`, "which can't be sent back" | Round trip through the caller, unread. | OR-documented | S24 |
| Context compaction items | OpenAI | Responses `compaction` items with `encrypted_content` | Responses input items `compaction`, `context_compaction`, `compaction_trigger`. Messages has a `compaction` block. | Chat messages cannot express it | Responses or Messages route only. Opaque round trip. | OR-documented | S28, S34 |
| Context editing | Anthropic | `context_management.edits` (beta) | `MessagesRequest.context_management`; response `context_management.applied_edits` | Outside the chat and Responses schemas | Messages route only. LiteLLM applies the edits inside the gateway. | OR-documented; Filled (LiteLLM) | S32, S33, S65 |
| Async tools | OpenAI (GPT-6) | `async: true` on a tool definition | Responses route. Chat accepts the flag and removes it. Messages runs every call synchronously. | Flattened to a normal tool call on chat | Responses route only. | OR-documented | S39 |
| `container`, `mcp_servers` | Anthropic | Request fields on Messages | Outside `MessagesRequest` | Not exposed | Not reachable through OpenRouter's Messages route per the spec. | OR-inferred (fields missing from the spec) | S33 |
| Computer use tools | Anthropic, OpenAI | Vendor tool types | Messages: "rejected at validation". The Responses tool union lists `ComputerUseServerTool`. | Not on chat or Messages | Vendor-specific tool types. | OR-documented | S39, S47 |
| Fast mode and service tier | Anthropic `speed`, OpenAI `service_tier` | Vendor field per API | `service_tier` on all three routes, enum widened with `ultrafast`. `speed` on Messages. Spellings interchangeable. Result in top-level `service_tier` (chat, Responses) or `usage.service_tier` (Messages). | Tier labels normalized, except Messages keeps `standard` | A neutral value translates. | OR-documented | S10, S37, S39 |
| Usage in streams | OpenAI | `stream_options.include_usage` opt-in; final chunk with empty `choices` | Always sent. `include_usage` "has no effect". The final chunk holds one content-free choice. | Differs from OpenAI's chunk shape | Gateway only. The client must accept a non-empty `choices` on the usage chunk. | OR-documented | S10, S22 |
| Typed error per route | All | Vendor-specific codes | `error_type` (27 values) at `error.metadata.error_type` (chat), top-level `error_type` (Responses), `error.error_type` (Messages) | Native codes are lossy on Responses and Messages | Gateway only. | OR-documented | S23, S30, S37 |
| Converting between chat, Responses, and Anthropic message formats | All | Not applicable | Official `@openrouter/agent` helpers `fromChatMessages`, `toChatMessage`, `fromClaudeMessages`, `toClaudeMessage` | Chat to Responses drops `reasoning_details`, images, audio. Responses to chat keeps text only. Claude input is limited to four block types. Encrypted reasoning is parked in `unsupported_content`. | A converter between protocols loses signed and encrypted reasoning unless it has an opaque side channel. | OR-documented (SDK source) | S59, S60, S61, S62 |
| Sampling parameters on the Anthropic shape | Anthropic | SDK types at release 2.12.0 list no `temperature`, `top_p`, `top_k` | `MessagesRequest` keeps all three. Newer Claude catalog entries omit them from `supported_parameters`. | Not applicable | The gateway decides per model from the catalog. | OR-documented, Vendor-documented | S32, S45 |
| API versioning | OpenRouter | Not applicable | One version `v1` in the URL path. Additive changes ship unannounced. Breaking changes go through the changelog. | Not applicable | Clients ignore unknown response fields and unknown enum values. | OR-documented | S53 |

## Evidence

Reading notes for the tables in this section:

- "Spec" means OpenRouter's OpenAPI document (S-s1). Type signatures are generated from the spec by script: `?` marks an optional property, `X|null` a nullable one, `array<T>` an array, and a bare capitalized name a named schema.
- "OpenAI standard" means the field is in OpenAI's official Python SDK types at commit `f63f14b` dated 2026-11-20 (S-s48) and was also found on OpenAI's live chat completions reference page (S-s49). "OpenRouter addition" means the field is in OpenRouter's spec and is not in those two OpenAI sources.
- "Wire" means a field shown in captured output inside OpenRouter's own docs, or in a second schema inside the same spec, and not listed in the named response schema.

### 1. The routes

**S1. The spec declares one server, one version, and three chat-like routes.** `servers[0].url` is `https://openrouter.ai/api/v1`. `info.description` is "OpenAI-compatible API with additional OpenRouter features". `info.version` is `1.0.0`. The three chat-like operations and their descriptions:

| Route | operationId | Request schema | Response schema (JSON / SSE) | Spec description |
|---|---|---|---|---|
| `POST /chat/completions` | `sendChatCompletionRequest` | `ChatRequest` | `ChatResult` / `ChatStreamingResponse` | "Sends a request for a model response for the given chat conversation. Supports both streaming and non-streaming modes." |
| `POST /responses` | `createResponses` | `ResponsesRequest` | `OpenResponsesResult` / `ResponsesStreamingResponse` | "Creates a streaming or non-streaming response using OpenResponses API format" |
| `POST /messages` | `createMessages` | `MessagesRequest` | `MessagesResult` / `MessagesStreamingResponse` | "Creates a message using the Anthropic Messages API format. Supports text, images, PDFs, tools, and extended thinking." |

Source: S-s1. Basis: OR-documented.

**S2. Full list of inference and inference-support routes in the spec.** All paths are relative to `https://openrouter.ai/api/v1`.

| Route | Purpose (spec summary or description) | Tag |
|---|---|---|
| `POST /chat/completions` | "Create a chat completion" | Chat |
| `POST /responses` | "Create a response" | Responses, beta.responses |
| `POST /messages` | "Create a message" | Anthropic Messages |
| `POST /embeddings`, `GET /embeddings/models` | "Submits an embedding request to the embeddings router" | Embeddings |
| `POST /rerank` | "Submits a rerank request to the rerank router" | Rerank |
| `POST /images`, `GET /images/models`, `GET /images/models/{author}/{slug}/endpoints` | "Generates an image from a text prompt via the image generation router" | Images |
| `POST /videos`, `GET /videos/{jobId}`, `GET /videos/{jobId}/content`, `GET /videos/models` | "Submits a video generation request and returns a polling URL to check status" | Video Generation |
| `POST /audio/speech`, `GET /audio/speech/models`, `GET /audio/speech/models/{author}/{slug}` | "Synthesizes audio from the input text. Returns a raw audio bytestream in the requested format (e.g. mp3, pcm, wav)." | TTS |
| `POST /audio/transcriptions` | "Transcribes audio into text. Accepts base64-encoded audio input as JSON, an OpenAI-style multipart/form-data file upload, or a URL the provider downloads directly, and returns the transcribed text." | STT |
| `POST /batches`, `GET /batches`, `GET /batches/{id}`, `DELETE /batches/{id}` | "Creates a batch of requests that run asynchronously against a single endpoint (`/v1/chat/completions`, `/v1/responses`, `/v1/messages`, `/v1/embeddings`, `/v1/audio/speech`)." | Batch |
| `POST /files`, `GET /files`, `GET /files/{file_id}`, `DELETE /files/{file_id}`, `GET /files/{file_id}/content` | "Uploads a file to be referenced in future API calls." | Files |
| `GET /containers/{container_id}/files` and three more container file routes | Files inside a bash or shell tool container | Containers |
| `POST /presets/{slug}/chat/completions`, `POST /presets/{slug}/messages`, `POST /presets/{slug}/responses` | "Creates a preset (or a new version of an existing one) from an inference request body." | Presets |
| `GET /generation`, `GET /generation/content`, `POST /generation/feedback` | "Get request & usage metadata for a generation" | Generations |
| `GET /tools`, `GET /tools/{name}` | "Lists every server tool OpenRouter can run on behalf of a model: accepted `tools[].type` spellings per API format, the engines behind it with their pricing, and how many endpoints run it natively." | Tools |
| `POST /systemone`, `POST /api/alpha/decisions`, `POST /api/alpha/extractions` | Typed-question and extraction routes for "System One" models | SystemOne, alpha.decisions, alpha.extractions |
| `GET /models`, `GET /models/count`, `GET /models/user`, `GET /model/{author}/{slug}`, `GET /models/{author}/{slug}/endpoints` | Model catalog | Models, Endpoints |

The Batch guide's own table lists four batchable endpoints, "`/v1/chat/completions`, `/v1/responses`, `/v1/messages`, `/v1/embeddings`" (S-s54), one fewer than the spec description above. The compatibility matrix adds that image generation is "Mounted at both `/api/v1/images` and `/api/v1/images/generations`" and marks Assistants, Realtime, and Fine-tuning "Not supported" (S-s13).

Source: S-s1, S-s13, S-s54. Basis: OR-documented.

**S3. The legacy text completions route lives in docs prose and in the catalog.** The spec's 123 paths cover `/chat/completions`, `/responses`, and `/messages` for text generation. The route `/api/v1/completions` is named by four doc pages and by the catalog:

- Router metadata guide, list of routes that return `openrouter_metadata`: "`/api/v1/completions` (legacy text completions)" (S-s17).
- FAQ: "OpenRouter implements the OpenAI API specification for the [/responses](https://openrouter.ai/docs/api_reference/responses/overview.md), /chat/completions, and /completions endpoints, allowing you to use any model with the same request/response format." (S-s15).
- Streaming page: "The generation ID is returned in the `X-Generation-Id` response header for all endpoints (chat completions, completions, responses, and messages)" (S-s10).
- API overview, request type: "// Either "messages" or "prompt" is required" followed by `messages?: Message[];` and `prompt?: string;`, and a response subtype `NonChatChoice` with `text: string` (S-s3).
- Catalog: 384 of 476 models list `/api/v1/completions` in `endpoints` (S-s2).

Basis: OR-documented. Confidence note: the request and response field list of the legacy route is documented only by the overview page's TypeScript type.

**S4. One doc sentence names a fifth text-generation request type, Gemini `generateContent`.** Broadcast guide: "`requested_models` is set on text-generation requests (chat completions, Responses, Messages, legacy completions, and Gemini `generateContent`)" (S-s38). A search of all 559 mirrored pages and the spec found this one occurrence of `generateContent`. Basis: OR-documented (a mention only). Confidence note: whether a public Gemini-shaped route exists is unverified, see Unverified.

**S5. Status of the three chat-like routes.**

- Chat completions: the compatibility matrix marks it "Supported" with the note "Full support for streaming, tools, structured outputs, and vision across 400+ models." (S-s13).
- Responses: the changelog entry dated July 25, 2026 says "The Responses API is now GA. `beta.responses` was an OpenAPI grouping tag and SDK namespace, never a URL. `POST /api/v1/responses` and its request and response schemas are unchanged, so raw HTTP callers and OpenAI-compatible clients (including the OpenAI Agents SDK) need no changes." (S-s39). The spec still lists the tag pair `['Responses', 'beta.responses']` on the operation, and describes the second tag as "Deprecated alias of responses. Use responses instead; scheduled for removal (sunset date TBD)." (S-s1).
- Messages: the spec tag description is "Anthropic Messages endpoints" (S-s1). The Claude Code cookbook says "OpenRouter exposes an input that is compatible with the Anthropic Messages API." and "OpenRouter's "Anthropic Skin" behaves exactly like the Anthropic API. It handles model mapping and passes through advanced features like "Thinking" blocks and native tool use." (S-s19). The stability statement that covers it is the versioning page's general one: "All endpoints documented in the [API Reference](https://openrouter.ai/docs/api_reference/overview.md) are part of `v1`." (S-s5).
- The API overview page names two shapes: "OpenRouter supports two request shapes: the [Responses API](https://openrouter.ai/docs/api_reference/responses/overview.md) (`POST /api/v1/responses`) and the Chat Completions API (`POST /api/v1/chat/completions`). Where this page shows both, the Responses API comes first." (S-s3). The errors page counts three: "OpenRouter exposes three API skins." (S-s9).
- In the spec the Messages operation has `x-speakeasy-ignore: true`. In the generated TypeScript SDK, `src/sdk/` holds client files `chat.ts` and `responses.ts`, and the `MessagesRequest` type is reachable only through the preset function `src/funcs/presetsCreatePresetsMessages.ts` (S-s44).

Basis: OR-documented.

**S6. Every chat model in the catalog lists all four text routes.** Script result over `models.json`: 384 models have `/api/v1/chat/completions` in `endpoints`, and all 384 also list `/api/v1/completions`, `/api/v1/messages`, and `/api/v1/responses`. The main group (369 models with exactly those four) spans 67 author prefixes, led by `openai` 59, `qwen` 40, `google` 28, `mistralai` 21, `z-ai` 17, `anthropic` 16. The other 92 catalog entries are `:batch` variants that list only `/api/v1/batches`. Source: S-s2. Basis: OR-documented (catalog data). Confidence note: the catalog states route availability per model; it does not state feature parity per route.

**S7. The Messages route serves non-Anthropic models.** Quotes and captured output:

- Reasoning guide: "Requests sent through the [Anthropic Messages API](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md) control reasoning with Anthropic's native `thinking` and `output_config.effort` fields. OpenRouter normalizes both into the unified `reasoning` parameter, so they work on every reasoning model, not only Claude." (S-s18).
- Same guide: "Claude receives both fields unchanged, while a model such as GPT-5 or MiniMax receives `reasoning_effort: "high"` in its own vocabulary, translated the same way as request-level `reasoning.effort`." (S-s18).
- Advisor guide: "On `/api/v1/messages`, request the advisor with the native Anthropic tool shape, and it works with **any executor model**, not just Anthropic ones" (S-s37).
- GPT-6 migration guide, Messages example body: `"model": "openai/gpt-6-sol"` with "The Messages API uses Anthropic's per-message `output_config.effort` on a content-less system message. OpenRouter translates it into the Responses `configuration_update` item when the request routes to a GPT-6 model." (S-s25).
- Gemini 4 migration guide, Messages example body: `"model": "google/gemini-4-argon"` (S-s26).
- Service tiers guide: a `curl https://openrouter.ai/api/v1/messages` request with `"model": "openai/gpt-5-mini"` and a captured response that ends `"context_management":null,"provider":"OpenAI"}` (S-s27).
- Limits the docs state: "Claude Code with OpenRouter is only guaranteed to work with the Anthropic first-party provider." and "Claude Code is optimized for Anthropic models and may not work correctly with other providers." (S-s19). "The Messages API (`/api/v1/messages`) does not support video inputs." (S-s59).

Basis: OR-documented.

**S8. The Responses route serves non-OpenAI models.**

- Responses overview: "OpenRouter's Responses API provides OpenAI-compatible access to multiple AI models through a unified interface, designed to be a drop-in replacement for OpenAI's Responses API." (S-s6).
- Reasoning guide, Responses example body: `"model": "anthropic/claude-fable-5.2"` with a `configuration_update` input item, and "A Responses `configuration_update` sent to a Claude model becomes a per-message `output_config`" (S-s18).
- Gemini 4 migration guide, Responses example body: `"model": "google/gemini-4-argon"` (S-s26).
- Agent SDK: "`callModel` is built on OpenRouter's Responses API which uses an **items-based model** rather than the messages-based model used by OpenAI Chat or Vercel AI SDK." (S-s43). Its examples call `anthropic/claude-sonnet-4.5` through `https://openrouter.ai/api/v1/responses` (S-s42).

Basis: OR-documented.

**S9. Preset routes reuse the three request schemas and keep only configuration.** Presets guide: "Each inference skin has its own endpoint. Send the same JSON body you would send to the matching inference route. OpenRouter persists only the fields that overlap with the preset config (e.g. `model`, `temperature`, `provider`, `top_p`, `system`, `tools`). Transient fields like `messages`, `input`, `prompt`, and `stream` are silently ignored." and "These endpoints require a read-write [management key](https://openrouter.ai/docs/guides/overview/auth/management-api-keys.md), like other workspace configuration endpoints; inference API keys are rejected with a 401." (S-s29). The spec gives `ChatRequest`, `MessagesRequest`, and `ResponsesRequest` as the three request bodies (S-s1). Basis: OR-documented.

### 2. Chat completions request, field by field

**S10. All 43 top-level properties of `ChatRequest`.** Generated from the spec. `required` is `["messages"]` only.

| Field | Required | OpenRouter spec type | Status | Note |
|---|---|---|---|---|
| `cache_control` | no | `AnthropicCacheControlDirective` | OpenRouter addition | Anthropic-shaped `{type: "ephemeral", ttl?: "5m"\|"1h"}` at the top level. Spec: "the system automatically applies cache breakpoints to the last cacheable block in the request". |
| `debug` | no | `ChatDebugOptions` | OpenRouter addition | `{echo_upstream_body?: boolean}`. Spec: "Debug options for inspecting request transformations (streaming only)". |
| `deferred_tools` | no | `DeferredToolsControl` | OpenRouter addition | Router-level deferred tool protocol: `protocol: "v1"`, `profile: "portable"\|"universal"`, `search`, `validation: "runtime"`. |
| `frequency_penalty` | no | `number\|null` | OpenAI standard |  |
| `image_config` | no | `ImageConfig` | OpenRouter addition | Free map. Spec: "Provider-specific image configuration options. Keys and values vary by model/provider." |
| `logit_bias` | no | `object\|null` | OpenAI standard |  |
| `logprobs` | no | `boolean\|null` | OpenAI standard |  |
| `max_completion_tokens` | no | `integer\|null` | OpenAI standard |  |
| `max_tokens` | no | `integer\|null` | OpenAI standard | Spec: "Maximum tokens (deprecated, use max_completion_tokens). Note: some providers enforce a minimum of 16." |
| `messages` | yes | `array<ChatMessages> [minItems=1]` | OpenAI standard | Message objects have OpenRouter additions, see the message tables below. |
| `metadata` | no | `map<string>` | OpenAI standard | Spec: "Key-value pairs for additional object information (max 16 pairs, 64 char keys, 512 char values)". |
| `min_p` | no | `number\|null` | OpenRouter addition | Spec: "Not all providers support this parameter." |
| `modalities` | no | `array<enum["text", "image", "audio"]>` | OpenAI standard name, widened enum | OpenAI enum is `["text", "audio"]` (S-s48 L217). OpenRouter adds `"image"`. |
| `model` | no | `ModelName` | OpenAI standard | Optional in OpenRouter's spec (`required: ["messages"]`). Docs: "If the `model` parameter is omitted, the user or payer's default is used." Required by OpenAI. |
| `models` | no | `ChatModelNames` | OpenRouter addition | Fallback list of model slugs. |
| `parallel_tool_calls` | no | `boolean\|null` | OpenAI standard |  |
| `plugins` | no | `array<(AutoRouterPlugin \| AutoBetaRouterPlugin \| ModerationPlugin \| WebSearchPlugin \| WebFetchPlugin \| FileParserPlugin \| ...8 more)>` | OpenRouter addition | 14 plugin ids in the spec: `auto-router`, `auto-beta-router`, `moderation`, `web`, `web-fetch`, `file-parser`, `response-healing`, `context-compression`, `pareto-router`, `fusion`, `switchyard-router`, `jev-router`, `alignment`, `armor`. |
| `prediction` | no | `Prediction` | OpenAI standard |  |
| `presence_penalty` | no | `number\|null` | OpenAI standard |  |
| `prompt_cache_key` | no | `string\|null` | OpenAI standard |  |
| `prompt_cache_options` | no | `PromptCacheOptions` | OpenAI standard name, different shape | OpenAI: `mode: "implicit"\|"explicit"`, `ttl: "30m"` (S-s48 L530-546). OpenRouter spec: `mode: enum["explicit"]` required, `ttl: string\|null`; "Only supported by OpenAI GPT-5.6 and newer." |
| `provider` | no | `ProviderPreferences` | OpenRouter addition | `ProviderPreferences`, 14 fields, `additionalProperties: false`: `allow_fallbacks`, `data_collection`, `enforce_distillable_text`, `ignore`, `max_price`, `only`, `options`, `order`, `preferred_max_latency`, `preferred_min_throughput`, `quantizations`, `require_parameters`, `sort`, `zdr`. |
| `reasoning` | no | `{context?: ReasoningContext, effort?: enum["max", "xhigh", "high", "medium", "low", "minimal", "none", null], enabled?: boolean, exclude?: boolean,...` | OpenRouter addition | Object with `context`, `effort`, `enabled`, `exclude`, `max_tokens`, `mode`, `summary`. OpenAI has a `reasoning` object on Responses only; on chat completions OpenAI has just `reasoning_effort`. |
| `reasoning_effort` | no | `enum["max", "xhigh", "high", "medium", "low", "minimal", "none", null]` | OpenAI standard | Same seven values as OpenAI's `ReasoningEffort` (S-s48 `shared/reasoning_effort.py` L8). Spec: "Cannot be used simultaneously with reasoning.effort if they differ." |
| `repetition_penalty` | no | `number\|null` | OpenRouter addition | Spec: "Not all providers support this parameter." |
| `response_format` | no | `(ChatFormatTextConfig \| ChatFormatJsonObjectConfig \| ChatFormatJsonSchemaConfig \| ChatFormatGrammarConfig \| ChatFormatPythonConfig)` | OpenAI standard name, widened union | OpenAI union: `text`, `json_schema`, `json_object`. OpenRouter adds `{type: "grammar", grammar: string}` and `{type: "python"}`. |
| `route` | no | `DeprecatedRoute` | OpenRouter addition | Deprecated. Spec: "Use providers.sort.partition instead". Values `fallback`, `sort`. |
| `seed` | no | `integer\|null` | OpenAI standard |  |
| `service_tier` | no | `enum["auto", "default", "fast", "flex", "priority", "scale", "ultrafast", null]` | OpenAI standard name, widened enum | OpenAI enum: `auto`, `default`, `flex`, `scale`, `priority`, `fast` (S-s48 L349). OpenRouter adds `ultrafast`. |
| `session_id` | no | `string [maxLength=256]` | OpenRouter addition | Spec: "OpenRouter uses it as the sticky routing key". Max 256 chars. Also accepted as the `x-session-id` header; "the body value takes precedence". |
| `stop` | no | `(string \| array<string> [maxItems=4] \| null)` | OpenAI standard |  |
| `stop_server_tools_when` | no | `StopServerToolsWhen` | OpenRouter addition | Stop conditions for the server-tool agent loop: `step_count_is`, `has_tool_call`, `max_tokens_used`, `max_cost`, `finish_reason_is`. |
| `stream` | no | `boolean [default=false]` | OpenAI standard |  |
| `stream_options` | no | `ChatStreamOptions` | OpenAI standard | OpenRouter spec has only `include_usage`, marked deprecated: "This field has no effect. Full usage details are always included." OpenAI's `include_obfuscation` is absent from the OpenRouter spec. |
| `temperature` | no | `number\|null` | OpenAI standard |  |
| `tool_choice` | no | `ChatToolChoice` | OpenAI standard name, different union | `none`, `auto`, `required`, and `{type: "function", function: {name}}` are OpenAI standard. OpenRouter adds `ChatServerToolChoice` `{type: string}`. OpenAI's `allowed_tools` and `custom` choices are absent from the OpenRouter spec. |
| `tools` | no | `array<ChatFunctionTool>` | OpenAI standard name, different union | `{type: "function", function}` is OpenAI standard. OpenRouter adds 12 more variants (10 `openrouter:*` server tools, the `web_search*` shorthand, a generic `{type: string, parameters?}` envelope) and `cache_control`, `defer_loading` on function tools. OpenAI's `{type: "custom"}` tool is absent from the OpenRouter spec. |
| `top_a` | no | `number\|null` | OpenRouter addition | Spec: "Not all providers support this parameter." |
| `top_k` | no | `integer\|null` | OpenRouter addition | Spec: "Not all providers support this parameter." |
| `top_logprobs` | no | `integer\|null` | OpenAI standard |  |
| `top_p` | no | `number\|null` | OpenAI standard |  |
| `trace` | no | `TraceConfig` | OpenRouter addition | `trace_id`, `trace_name`, `span_name`, `generation_name`, `parent_span_id`, plus free keys passed to broadcast destinations. |
| `user` | no | `string` | OpenAI standard | Spec: "OpenRouter folds it into the hashed identity sent upstream and never forwards it raw." |

Source: S-s1 for the OpenRouter columns, S-s48 and S-s49 for the status column. Basis: OR-documented.

**S11. The schema is open, and ten OpenAI chat fields are outside it.** `ChatRequest` leaves `additionalProperties` unset, which in JSON Schema means extra properties validate. The parameters page says "You may send any parameters from the following list, as well as others, to OpenRouter." (S-s4). OpenAI request fields at S-s48 that the OpenRouter spec's `ChatRequest` does not list: `audio`, `function_call`, `functions`, `moderation`, `n`, `prompt_cache_retention`, `safety_identifier`, `store`, `verbosity`, `web_search_options`. Two of those ten are documented by OpenRouter outside the spec, see S12. Basis: OR-documented for the spec content; OR-inferred for what happens to the other eight (inferred from the missing fields plus the general "ignored" rule in S41).

**S12. Chat request fields that OpenRouter documents outside the spec.**

| Field | Where documented | Quote |
|---|---|---|
| `verbosity` | Parameters page; catalog `supported_parameters` (114 models) | "Optional, **enum** (low, medium, high, xhigh, max)" and "For Anthropic models, this parameter maps to `output_config.effort`." (S-s4) |
| `web_search_options` | Parameters page; catalog `supported_parameters` (18 models) | "Configures native web search options for models and providers that support web-connected answers." (S-s4) |
| `include_reasoning` | Parameters page; catalog `supported_parameters` (359 models) | "Deprecated alias for reasoning.exclude. When true, reasoning tokens are returned in the response when supported by the model." (S-s4) |
| `structured_outputs` | Parameters page; catalog `supported_parameters` (407 models) | "Optional, **boolean**" and "If the model can return structured outputs using response\_format json\_schema." (S-s4) |
| `preset` | Presets guide | "The `preset` field takes the same `@preset/{slug}` reference as the model field. A bare slug without the `@preset/` prefix is ignored." (S-s29). Also usable inside `model`: `"model": "@preset/email-copywriter"` and `"model": "openai/gpt-4@preset/email-copywriter"`. |
| `usage` | Usage accounting cookbook | "On Chat Completions, the `usage: { include: true }` and `stream_options: { include_usage: true }` parameters are deprecated and have no effect. The Responses API has no equivalent parameter. Full usage details are always included automatically in every response on both APIs." (S-s31) |
| `prompt` | API overview type | "// Either "messages" or "prompt" is required" (S-s3). This is the legacy completions input. |
| `x_search_filter` | Web search docs | "The legacy top-level `x_search_filter` request field still enables X search for backward compatibility." (S-s60, Responses web search page) |
| `safe_prompt`, `raw_mode` | Parameters page | "OpenRouter will also transmit some provider-specific parameters, such as `safe_prompt` for Mistral or `raw_mode` for Hyperbolic directly to the respective providers if specified." (S-s4) |
| `transforms` | Zero occurrences in the spec and in all mirrored doc pages other than the page title "Message Transforms" | The page now documents a plugin: "OpenRouter supports a context compression [plugin](https://openrouter.ai/docs/guides/features/plugins.md) that can be enabled per-request" with `plugins: [{ id: "context-compression" }]` (S-s30). |

Basis: OR-documented. Confidence note: the parameters page lists `structured_outputs` under the same "Key" heading as request parameters, and the catalog uses the same word as a capability flag. Which of the two it is on the wire is unverified.

**S13. Message objects by role.** `ChatMessages` is a union discriminated on `role`.

| Role | Fields in the spec | OpenAI standard fields | OpenRouter additions |
|---|---|---|---|
| `system` | `role`, `content` (string or array of text parts), `name?`, `configuration_update?` | `role`, `content`, `name` | `configuration_update` |
| `developer` | `role`, `content` (string or array of text parts), `name?`, `configuration_update?` | `role`, `content`, `name` | `configuration_update` |
| `user` | `role`, `content` (string or array of `ChatContentItems`), `name?` | `role`, `content`, `name` | Content part types, see S14 |
| `assistant` | `role`, `content?` (string, array of `ChatContentItems`, or null), `name?`, `refusal?`, `tool_calls?`, `audio?`, `reasoning?`, `reasoning_details?`, `images?`, `model?` | `role`, `content`, `name`, `refusal`, `tool_calls`, `audio` | `reasoning` (`string\|null`), `reasoning_details` (array of `ReasoningDetailUnion`), `images` (array of `{image_url: {url}}`), `model` (string, "Model that generated this assistant message") |
| `tool` | `role`, `content` (string or array of `ChatContentItems`), `tool_call_id` | `role`, `content`, `tool_call_id` | Content may hold any part type; OpenAI limits tool content to text parts (S-s48 `chat_completion_tool_message_param.py` L14) |

Notes:

- `ChatAssistantMessage` is one schema used both ways. Spec description: "Assistant message for requests and responses".
- `configuration_update` spec description: "OpenRouter extension. Changes reasoning effort from this point in the conversation onward without invalidating the prompt cache for the preceding turns. Place it on a content-less system message (`content: ""`) directly before the user message it should apply to, and keep it at that position in later requests. Equivalent to the OpenAI Responses `configuration_update` input item and the Anthropic Messages per-message `output_config.effort`." Shape: `{reasoning: {effort}}`, `additionalProperties: false`.
- OpenAI message fields outside the OpenRouter spec: the deprecated `function_call` on assistant messages, the `function` role message, and `annotations` on assistant messages (see S19 and S24 for where `annotations` appears on OpenRouter's wire).
- The auto-router plugin reads the `model` addition. Spec, `AutoRouterPlugin.pin_model`: "When true, reuses the model from the most recent assistant message's `model` attribute for subsequent turns." (S-s1).

Sources: S-s1, S-s48. Basis: OR-documented.

**S14. Content part types and cache markers.** `ChatContentItems` is a union discriminated on `type` with six members.

| Part `type` | Fields | Status |
|---|---|---|
| `text` | `text`, `cache_control?`, `prompt_cache_breakpoint?` | OpenAI standard type. `prompt_cache_breakpoint` (`{mode: "explicit", ttl?}`) is OpenAI standard (S-s48 `chat_completion_content_part_text_param.py` L32). `cache_control` (`{type: "ephemeral", ttl?: "5m"\|"1h"}`) is an OpenRouter addition in Anthropic's shape. |
| `image_url` | `image_url: {url, detail?}` with `detail` enum `auto`, `low`, `high`, `original` | OpenAI standard |
| `input_audio` | `input_audio: {data, format}` with `format` a free string | OpenAI standard type; OpenAI's `format` enum is `wav`, `mp3` (S-s48) |
| `file` | `file: {file_data?, file_id?, filename?}` | OpenAI standard |
| `video_url` | `video_url: {url, processing?}` with `processing` enum `agentic`, `static` | OpenRouter addition |
| `input_video` | Same inner object under `video_url` | OpenRouter addition, marked deprecated: "Video input content part (legacy format - deprecated)" |

Notes:

- In the spec, both cache markers sit on text parts only. OpenAI's SDK types also put `prompt_cache_breakpoint` on image, audio, and file parts (S-s48 `chat_completion_content_part_image_param.py` L45, `..._input_audio_param.py` L41, `chat_completion_content_part_param.py` L52).
- `ChatContentCacheControl` spec description: "Anthropic-style cache breakpoint for the content part. Interchangeable with the OpenAI-style `prompt_cache_breakpoint` marker: OpenRouter converts between the two based on the provider serving the request."
- `cache_control` also appears at the request top level (S10), on function tool definitions (S15), and inside `FusionServerToolConfig`.
- OpenAI's `refusal` content part for assistant messages is outside the OpenRouter union.

Sources: S-s1, S-s48. Basis: OR-documented.

**S15. Tool definitions and `tool_choice`.** `tools` items are `ChatFunctionTool`, described as "Tool definition for function calling (regular function or OpenRouter built-in server tool)". It is an `anyOf` with 13 members.

| `type` value | Member | Other fields |
|---|---|---|
| `function` | inline | `function: {name (max 64), description?, parameters?, strict?, defer_loading?}`, tool-level `cache_control?`, tool-level `defer_loading?` |
| `openrouter:advisor` | `AdvisorServerTool_OpenRouter` | `parameters?` |
| `openrouter:bash` | `BashServerTool` | `parameters?` |
| `openrouter:datetime` | `DatetimeServerTool` | `parameters?` |
| `openrouter:files` | `FilesServerTool` | `parameters?` |
| `openrouter:fusion` | `FusionServerTool_OpenRouter` | `parameters?` |
| `openrouter:image_generation` | `ImageGenerationServerTool_OpenRouter` | `parameters?` |
| `openrouter:experimental__search_models` | `ChatSearchModelsServerTool` | `parameters?` |
| `openrouter:subagent` | `SubagentServerTool_OpenRouter` | `parameters?` |
| `openrouter:web_fetch` | `WebFetchServerTool` | `parameters?` |
| `openrouter:web_search` | `OpenRouterWebSearchServerTool` | `parameters?` |
| `web_search`, `web_search_preview`, `web_search_preview_2025_03_11`, `web_search_2025_08_26` | `ChatWebSearchShorthand` | "Web search tool using OpenAI Responses API syntax. Automatically converted to openrouter:web_search." |
| any string | `ChatDynamicServerTool` | `parameters?`. "Generic OpenRouter server-tool envelope. `type` names a registered server tool (canonical `openrouter:*` form or a registered shorthand)" |

`tool_choice` (`ChatToolChoice`) accepts `"none"`, `"auto"`, `"required"`, `{type: "function", function: {name}}`, and `ChatServerToolChoice` `{type: string}`, described as "OpenRouter extension: force a specific server tool by naming it directly in `tool_choice.type` instead of wrapping it in `{ type: "function", function: { name } }`."

Tool calls on the assistant message (`ChatToolCall`): `id`, `type: enum["function"]`, `function: {name, arguments}`. All three are OpenAI standard. OpenAI's second tool call variant `{type: "custom", custom: {name, input}}` and its `{type: "custom"}` tool definition (S-s48 `chat_completion_custom_tool_param.py` L61-67) are outside the OpenRouter chat union.

Sources: S-s1, S-s48. Basis: OR-documented.

**S16. The `reasoning_details` union.** `ChatReasoningDetails` is an array of `ReasoningDetailUnion`, discriminated on `type`.

| `type` | Required fields | Optional fields |
|---|---|---|
| `reasoning.text` | `type` | `text` (`string\|null`), `signature` (`string\|null`), `format`, `id` (`string\|null`), `index` (integer) |
| `reasoning.summary` | `type`, `summary` | `format`, `id`, `index` |
| `reasoning.encrypted` | `type`, `data` | `format`, `id`, `index` |
| `reasoning.server_tool_call` | `type`, `tool_name`, `arguments`, `result` | `tool_call_id`, `format`, `id`, `index` |

`format` is `ReasoningFormat`, an open enum (`x-speakeasy-unknown-values: "allow"`) with 12 named values plus null: `unknown`, `openai-responses-v1`, `azure-openai-responses-v1`, `bedrock-openai-responses-v1`, `bedrock-xai-responses-v1`, `xai-responses-v1`, `meta-responses-v1`, `google-vertex-meta-responses-v1`, `google-vertex-xai-responses-v1`, `openrouter-stealth-v1`, `anthropic-claude-v1`, `google-gemini-v1`. The fourth union member is a record of an OpenRouter server tool run (for example `openrouter:fusion`) kept inside `reasoning_details` so a later turn can replay it. Source: S-s1. Basis: OR-documented.

**S17. The chat `reasoning` request object.** Properties: `context` (`auto`, `all_turns`, `current_turn`), `effort` (`max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`), `enabled` (boolean), `exclude` (boolean), `max_tokens` (integer), `mode` (`standard`, `pro`), `summary` (`auto`, `concise`, `detailed`). The spec marks `context` and `mode` "Only supported by OpenAI GPT-5.6 and newer." Source: S-s1. Basis: OR-documented.

### 3. Chat completions response and stream, field by field

**S18. Non-streaming response (`ChatResult`) in the spec.**

| Object | Field | Spec type | Status |
|---|---|---|---|
| `ChatResult` | `id` | string | OpenAI standard |
| `ChatResult` | `object` | `enum["chat.completion"]` | OpenAI standard |
| `ChatResult` | `created` | integer | OpenAI standard |
| `ChatResult` | `model` | string | OpenAI standard |
| `ChatResult` | `choices` | `array<ChatChoice>` | OpenAI standard |
| `ChatResult` | `system_fingerprint` | `string\|null`, required | OpenAI standard |
| `ChatResult` | `service_tier?` | `string\|null`, "The service tier used by the upstream provider for this request" | OpenAI standard name, free string here |
| `ChatResult` | `usage?` | `ChatUsage` | OpenAI standard with additions, see S20 |
| `ChatResult` | `openrouter_metadata?` | `OpenRouterMetadata` | OpenRouter addition, opt-in by header |
| `ChatResult` | `safeguard_results?` | `array<ArmorSafeguardResult>` | OpenRouter addition |
| `ChatResult` | `alignment?` | `Alignment` | OpenRouter addition, marked beta |
| `ChatChoice` | `index` | integer | OpenAI standard |
| `ChatChoice` | `message` | `ChatAssistantMessage` | OpenAI standard with additions, see S13 |
| `ChatChoice` | `finish_reason` | `ChatFinishReasonEnum`: `tool_calls`, `stop`, `length`, `content_filter`, `error`, null | OpenAI standard name. OpenAI's enum is `stop`, `length`, `tool_calls`, `content_filter`, `function_call` (S-s48 `chat_completion.py` L40). OpenRouter adds `error`. |
| `ChatChoice` | `logprobs?` | `ChatTokenLogprobs`: `content`, `refusal?`, each an array of `{token, logprob, bytes, top_logprobs}` | OpenAI standard |
| `ChatToolCall` | `id`, `type`, `function.name`, `function.arguments` | strings; `type` is `enum["function"]` | OpenAI standard |
| `OpenRouterMetadata` | `attempt`, `attempts?`, `endpoints`, `generation_time?`, `is_byok`, `params?`, `pipeline?`, `region`, `requested`, `strategy`, `summary` | `attempts` items are `{model, provider, status}`; `strategy` enum is `direct`, `auto`, `free`, `latest`, `alias`, `fallback`, `pareto`, `bodybuilder`, `fusion` | OpenRouter addition |

OpenAI response fields outside this schema: top-level `metadata` and `moderation`, and `message.function_call` (S-s48 `chat_completion.py` L215-225, `chat_completion_message.py` L82). Sources: S-s1, S-s48. Basis: OR-documented.

**S19. Three wire fields live outside `ChatResult`: `provider`, `native_finish_reason`, and `message.annotations`.** The spec's `ChatResult`, `ChatChoice`, and `ChatAssistantMessage` schemas list none of the three. They are documented in four other places:

- A second, inline copy of the chat completion body inside the spec, at `BatchObject.results[].response.body.anyOf[0]`. Its top-level properties are `alignment`, `choices`, `created`, `debug`, `id`, `model`, `object`, `openrouter_metadata`, `provider` (`string\|null`), `safeguard_results`, `service_tier`, `system_fingerprint`, `usage`. Its choice items add `native_finish_reason` (`string\|null`). Its message adds `annotations` (three variants: `file`, `url_citation`, `web_search_citation`), a `type: "image_url"` discriminator on `images[]` items, and `index` on each `tool_calls[]` item (S-s1).
- The API overview's response type: `native_finish_reason: string | null;` on both `NonStreamingChoice` and `StreamingChoice`, with the example lines `"finish_reason": "stop", // Normalized finish_reason` and `"native_finish_reason": "stop", // The raw finish_reason from the provider` (S-s3).
- Captured output in the PDF guide: `"model": "google/gemma-3-27b-it",` `"provider": "NextBit",` `"system_fingerprint": "vllm-0.32.0-8349ae92",` `"service_tier": null,` then a choice with `"finish_reason": "stop",` `"native_finish_reason": "stop",` and a message with `"refusal": null,` `"reasoning": null,` `"annotations": [` (S-s28).
- The generation record schema: `native_finish_reason` ("Native finish reason as reported by provider") and `provider_name` (S-s1, `GenerationResponse.data`).

The `url_citation` annotation in the batch copy has `url`, `title`, `start_index`, `end_index` (the four OpenAI standard fields, S-s48 `chat_completion_message.py` L13-25) plus `content?` and `uncited?` (`const true`). The `web_search_citation` variant has `url`, `title`, `cited_text?`, `encrypted_index?`. Basis: OR-documented. Confidence note: the three official SDKs are generated from the named schemas, so their typed chat result omits these three fields (S58).

**S20. Usage object (`ChatUsage`).**

| Field | Spec type | Status |
|---|---|---|
| `prompt_tokens`, `completion_tokens`, `total_tokens` | integer | OpenAI standard |
| `prompt_tokens_details.cached_tokens` | integer | OpenAI standard |
| `prompt_tokens_details.cache_write_tokens` | integer | In OpenAI's SDK type too: `cache_write_tokens: Optional[int] = None` (S-s48 `completion_usage.py` L43), and found 12 times on OpenAI's live reference page (S-s49) |
| `prompt_tokens_details.audio_tokens` | integer | OpenAI standard |
| `prompt_tokens_details.video_tokens` | integer | OpenRouter addition |
| `prompt_tokens_details.file_tokens` | integer or null, in the batch copy only | OpenRouter addition |
| `completion_tokens_details.reasoning_tokens`, `.audio_tokens`, `.accepted_prediction_tokens`, `.rejected_prediction_tokens` | integer or null | OpenAI standard |
| `completion_tokens_details.image_tokens` | In the overview type and the batch copy | OpenRouter addition on the completion side (OpenAI has `image_tokens` on the prompt side) |
| `cost` | `number\|null`, "Cost of the completion" | OpenRouter addition |
| `cost_details` | `{upstream_inference_cost?, upstream_inference_prompt_cost, upstream_inference_completions_cost, server_tool_cost?}` | OpenRouter addition |
| `is_byok` | boolean | OpenRouter addition |
| `server_tool_use_details` | `{tool_calls_requested?, tool_calls_executed?, web_search_requests?}` | OpenRouter addition |
| `deferred_tools_details` | `{prompt_tokens, completion_tokens, total_tokens, turns}` | OpenRouter addition |

OpenAI usage fields outside the OpenRouter schema: `prompt_tokens_details.image_tokens`, `.text_tokens`, and `completion_tokens_details.text_tokens` (S-s48 `completion_usage.py` L33-52). Sources: S-s1, S-s3, S-s48, S-s49. Basis: OR-documented.

**S21. Streaming chunk (`ChatStreamChunk`).** The SSE stream ends with the sentinel `[DONE]` (`x-speakeasy-sse-sentinel`).

| Object | Field | Spec type | Status |
|---|---|---|---|
| `ChatStreamChunk` | `id`, `created`, `model`, `choices` | string, integer, string, `array<ChatStreamChoice>` | OpenAI standard |
| `ChatStreamChunk` | `object` | `enum["chat.completion.chunk"]` | OpenAI standard |
| `ChatStreamChunk` | `system_fingerprint?`, `service_tier?`, `usage?` | string, `string\|null`, `ChatUsage` | OpenAI standard |
| `ChatStreamChunk` | `error?` | `{code: integer, message: string, metadata?: {error_type: ApiErrorType, provider_code?: string}}` | OpenRouter addition |
| `ChatStreamChunk` | `openrouter_metadata?`, `safeguard_results?`, `alignment?` | as in S18 | OpenRouter addition |
| `ChatStreamChoice` | `index`, `delta`, `finish_reason`, `logprobs?` | `finish_reason` is the same six-value enum as S18 | OpenAI standard |
| `ChatStreamDelta` | `role?` | `enum["assistant"]` | OpenAI standard |
| `ChatStreamDelta` | `content?`, `refusal?` | `string\|null` | OpenAI standard |
| `ChatStreamDelta` | `tool_calls?` | `array<ChatStreamToolCall>`: `index`, `id?`, `type?`, `function?: {name?, arguments?}` | OpenAI standard |
| `ChatStreamDelta` | `audio?` | `ChatAudioOutput`: `id?`, `data?`, `expires_at?`, `transcript?` | OpenAI standard |
| `ChatStreamDelta` | `reasoning?` | `string\|null`, "Reasoning content delta" | OpenRouter addition |
| `ChatStreamDelta` | `reasoning_details?` | `array<ReasoningDetailUnion>` | OpenRouter addition |

Wire fields outside this schema, from docs: `native_finish_reason` on each choice (S-s10 sample lines), and on the debug chunk a top-level `"provider": "Anthropic"` and `debug.echo_upstream_body` (S-s9). OpenAI chunk fields outside the OpenRouter schema: `obfuscation`, `moderation`, and `delta.function_call` (S-s48 `chat_completion_chunk.py` L118, L331-337). Sources: S-s1, S-s9, S-s10, S-s48. Basis: OR-documented.

**S22. Stream framing differences from OpenAI.**

- Final usage chunk: "OpenAI's spec emits this chunk with an empty `choices` array, but many clients crash when accessing `choices[0].delta` on it, so OpenRouter intentionally deviates: the usage chunk contains one choice with a content-free `delta` that repeats the `finish_reason` (and `native_finish_reason`) of the stream." and "the terminal `finish_reason` appears twice: once on the last content-bearing chunk and again on the usage chunk." (S-s10).
- Debug chunk: "OpenRouter sends a debug chunk as the **first chunk** in the streaming response. This chunk has an empty `choices` array and includes a `debug` field with the transformed request body" and "a debug chunk will be sent for **each attempted provider**" (S-s9).
- Keepalive: "OpenRouter sends periodic SSE comment lines", shown as `: OPENROUTER PROCESSING` (S-s13).
- Usage placement on the other routes: "the Responses API (`/api/v1/responses`) reports usage in the `response.completed` event, and the Messages API (`/api/v1/messages`) reports it in the `message_delta` event before `message_stop`." (S-s10).

Basis: OR-documented.

**S23. Error object on chat completions.** HTTP error bodies are `{error: {code: integer, message: string, metadata?: object\|null}, openrouter_metadata?: object\|null, user_id?: string\|null}` (for example `BadRequestResponse`). The operation lists 14 error statuses: 400, 401, 402, 403, 404, 408, 413, 422, 429, 500, 502, 503, 524, 529. OpenAI's error object is `{code: Optional[str], message: str, param: Optional[str], type: str}` (S-s48 `shared/error_object.py` L44-51), so OpenRouter's `code` is a number where OpenAI's is a string, `type` and `param` are replaced by `metadata`, and the typed discriminator is `error.metadata.error_type`. `ApiErrorType` is described as "Canonical OpenRouter error type, stable across all API formats" and has 27 values: `context_length_exceeded`, `max_tokens_exceeded`, `token_limit_exceeded`, `string_too_long`, `authentication`, `permission_denied`, `payment_required`, `rate_limit_exceeded`, `provider_overloaded`, `provider_unavailable`, `invalid_request`, `invalid_prompt`, `not_found`, `precondition_failed`, `payload_too_large`, `unprocessable`, `content_policy_violation`, `refusal`, `invalid_image`, `image_too_large`, `image_too_small`, `unsupported_image_format`, `image_not_found`, `image_download_failed`, `server`, `timeout`, `unmapped`. Where the typed code sits on each route: "**Chat Completions**: `error.metadata.error_type`", "**Anthropic Messages**: `error.error_type` on the SSE `error` event and the non-streaming error envelope.", "**Responses**: top-level `error_type` on the failed response" (S-s9). A provider failure after generation started returns a `200` whose choice holds `"finish_reason": "error"` and an `error` object next to partial content (S-s9). Sources: S-s1, S-s9, S-s48. Basis: OR-documented.

**S24. `annotations` is data the caller sends back.** PDF guide: "When you send a PDF to the Chat Completions API (`/api/v1/chat/completions`), the response may include file annotations in the assistant's message." and "By sending these annotations back in subsequent requests, you can avoid re-parsing the same PDF document multiple times, which saves both processing time and costs. The Responses API returns `file_citation` annotations instead, which can't be sent back to skip parsing". Annotation type: `type: 'file'` with `file: {hash, name?, content}` where "The `hash` field uniquely identifies the parsed file content and is used to skip re-parsing when you include the annotation in subsequent requests." (S-s28). Basis: OR-documented.

### 4. The Responses route

**S25. All 43 top-level properties of `ResponsesRequest`.** Generated from the spec. `required` is unset, so the schema marks every property optional. The status column compares against OpenAI's `ResponseCreateParamsBase` (S-s48 `responses/response_create_params.py` L47-347).

| Field | Required | OpenRouter spec type | Status | Note |
|---|---|---|---|---|
| `access_programs` | no | `{cyber?: enum["standard", "daybreak_blue", "daybreak_red"]}` | OpenAI Responses standard | Spec: a program other than `standard` "is supported only on BYOK requests that run on your own OpenAI key enrolled in that program; any other endpoint ... rejects it with a 400 error." |
| `background` | no | `boolean\|null` | OpenAI Responses standard | Type `boolean\|null` in the spec. The spec and docs give it no description. Sample responses echo `"background": false`. |
| `cache_control` | no | `AnthropicCacheControlDirective` | OpenRouter addition | Same top-level directive as chat completions. |
| `debug` | no | `ChatDebugOptions` | OpenRouter addition |  |
| `deferred_tools` | no | `DeferredToolsControl` | OpenRouter addition |  |
| `frequency_penalty` | no | `number\|null` | OpenRouter addition | Not in OpenAI's `ResponseCreateParamsBase` (S-s48). |
| `image_config` | no | `ImageConfig` | OpenRouter addition |  |
| `include` | no | `array<ResponseIncludesEnum>\|null` | OpenAI Responses standard | Items are `ResponseIncludesEnum`: `file_search_call.results`, `message.input_image.image_url`, `computer_call_output.output.image_url`, `reasoning.encrypted_content`, `output.encrypted_content`, `code_interpreter_call.outputs`, `web_search_call.results`, `web_search_call.action.sources`. |
| `input` | no | `Inputs` | OpenAI Responses standard | String or array of 52 item variants, including `ItemReferenceItem` (`{type: "item_reference", id}`), `CompactionItem`, `ConfigurationUpdateItem`, and OpenRouter server tool items. |
| `instructions` | no | `string\|null` | OpenAI Responses standard |  |
| `max_output_tokens` | no | `integer\|null` | OpenAI Responses standard |  |
| `max_tool_calls` | no | `integer\|null` | OpenAI standard name, OpenRouter meaning | Spec: "Maximum number of server-tool (e.g. `openrouter:web_search`) agent steps the model may take during a request. Defaults to 30, which is also the maximum. Ignored when `stop_server_tools_when` is set." |
| `metadata` | no | `RequestMetadata` | OpenAI Responses standard |  |
| `modalities` | no | `array<OutputModalityEnum>` | OpenRouter addition | Spec: "Supported values are "text" and "image"." |
| `model` | no | `string` | OpenAI Responses standard | Optional in the spec (`required` is unset on `ResponsesRequest`). |
| `models` | no | `array<string>` | OpenRouter addition |  |
| `parallel_tool_calls` | no | `boolean\|null` | OpenAI Responses standard |  |
| `plugins` | no | `array<(AutoRouterPlugin \| AutoBetaRouterPlugin \| ModerationPlugin \| WebSearchPlugin \| WebFetchPlugin \| FileParserPlugin \| ...8 more)>` | OpenRouter addition |  |
| `presence_penalty` | no | `number\|null` | OpenRouter addition | Not in OpenAI's `ResponseCreateParamsBase` (S-s48). |
| `previous_response_id` | no | `any` | OpenAI standard, rejected | Spec: "Not supported on this proxy. Each response request is independent: no responses are stored, so a previous response cannot be referenced. Requests with a non-null value are rejected with a 400 error. Send the full conversation history in `input` instead." |
| `prompt` | no | `StoredPromptTemplate` | OpenAI Responses standard | `StoredPromptTemplate` `{id, variables?}`. The spec and docs give it no description. |
| `prompt_cache_key` | no | `string\|null` | OpenAI Responses standard |  |
| `prompt_cache_options` | no | `PromptCacheOptions` | OpenAI Responses standard |  |
| `provider` | no | `ProviderPreferences` | OpenRouter addition |  |
| `reasoning` | no | `ReasoningConfig` | OpenAI standard, widened | `BaseReasoningConfig` (`context`, `effort`, `mode`, `summary`) plus OpenRouter's `enabled` and `max_tokens`. |
| `route` | no | `DeprecatedRoute` | OpenRouter addition |  |
| `safety_identifier` | no | `string\|null` | OpenAI Responses standard |  |
| `service_tier` | no | `enum["auto", "default", "fast", "flex", "priority", "scale", "ultrafast", null] [default="auto"]` | OpenAI standard name, widened enum | Adds `ultrafast`. Default `auto`. |
| `session_id` | no | `string [maxLength=256]` | OpenRouter addition |  |
| `stop` | no | `(string \| array<string> [maxItems=4] \| null)` | OpenRouter addition | Not in OpenAI's `ResponseCreateParamsBase` (S-s48). |
| `stop_server_tools_when` | no | `StopServerToolsWhen` | OpenRouter addition |  |
| `store` | no | `const false [default=false]` | OpenAI standard, restricted | Spec type is `const false`, default `false`. Docs: "Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error." |
| `stream` | no | `boolean [default=false]` | OpenAI Responses standard |  |
| `temperature` | no | `number\|null` | OpenAI Responses standard |  |
| `text` | no | `TextExtendedConfig` | OpenAI standard, widened enum | `verbosity` enum is `low`, `medium`, `high`, `xhigh`, `max`; OpenAI's is `low`, `medium`, `high`. |
| `tool_choice` | no | `OpenAIResponsesToolChoice` | OpenAI Responses standard |  |
| `tools` | no | `array<((FunctionTool & {allowed_callers?: array<enum["direct", "programmatic"]>\|null, async?: boolean, defer_loading?: boolean, output_schema?: obj...` | OpenAI standard name, widened union | 29 variants: OpenAI's function, web search, file search, computer use, code interpreter, MCP, image generation, local shell, shell, apply patch, custom, namespace tools, plus 14 OpenRouter server tools. Function tools gain `allowed_callers`, `async`, `defer_loading`, `output_schema`. |
| `top_k` | no | `integer` | OpenRouter addition | Not in OpenAI's `ResponseCreateParamsBase` (S-s48). |
| `top_logprobs` | no | `integer\|null` | OpenAI Responses standard |  |
| `top_p` | no | `number\|null` | OpenAI Responses standard |  |
| `trace` | no | `TraceConfig` | OpenRouter addition |  |
| `truncation` | no | `OpenAIResponsesTruncation` | OpenAI Responses standard | Enum `auto`, `disabled`, `null`. Docs: mid-conversation effort updates "cannot be combined with `truncation: "auto"`". |
| `user` | no | `string [maxLength=256]` | OpenAI Responses standard |  |

Source: S-s1, S-s48. Basis: OR-documented.

**S26. The route is stateless and says so with a 400.** Docs banner on the Responses overview, re-fetched live the same day: "This API is **stateless** - each request is independent and no conversation state is persisted between requests. You must include the full conversation history in each request. Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error." (S-s6). Basic usage page: "Since the Responses API is stateless, you must include the full conversation history in each request to maintain context. Replayed assistant content uses `output_text`, while user content uses `input_text`. Stateful parameters like `store: true` and `previous_response_id` are not supported and are rejected with a `400` error" (S-s8). Spec: `store` is `{"const": false, "default": false, "type": "boolean"}`; `previous_response_id` has a description and no type (quoted in the S25 table). Gemini 4 guide: "This is the stateless pattern the Responses API already uses for encrypted reasoning. OpenRouter does not store the response, so `previous_response_id` cannot be used to continue." (S-s26). Basis: OR-documented.

**S27. Status of each OpenAI Responses field the task asked about.**

| OpenAI field | In `ResponsesRequest`? | What the spec or docs say |
|---|---|---|
| `previous_response_id` | Yes | Non-null value is rejected with a 400 (S26) |
| `store` | Yes | `const false`; `true` is rejected with a 400 (S26) |
| `background` | Yes, `boolean\|null` | The spec and docs give it no description. Sample responses echo `"background": false` (S-s6). Behavior of `true` is unverified. |
| `conversation` | Outside the schema | OpenAI: `conversation: Optional[Conversation]` (S-s48 L67) |
| `include` | Yes | Eight enum values, including `reasoning.encrypted_content` (S25 table) |
| `truncation` | Yes, `auto`, `disabled`, null | "Updates cannot be combined with `truncation: "auto"` in the Responses API" (S-s18) |
| `prompt_cache_key` | Yes, `string\|null` | Also on chat completions |
| `service_tier` | Yes | Enum widened with `ultrafast`; "`fast` is accepted as an alias for `priority`" |
| `context_management` | Outside the schema | OpenAI: `context_management: Optional[Iterable[ContextManagement]]` with `compact_threshold` (S-s48 L64, L396-400). OpenRouter's input union does accept `compaction`, `context_compaction`, and `compaction_trigger` items (S28). |
| `background_options`, `image`, `moderation`, `prompt_cache_retention`, `stream_options` | Outside the schema | Present in S-s48 L57, L75, L154, L195, L268 |
| `prompt` | Yes, `{id, variables?}` | No description in spec or docs |
| `access_programs` | Yes | Non-`standard` values need a BYOK OpenAI key, else a 400 |

Sources: S-s1, S-s48. Basis: OR-documented for rows found in the spec; OR-inferred for the "outside the schema" rows (inferred from a field missing in the spec; the general "ignored" rule in S41 would apply to them).

**S28. Input items: OpenAI's item list plus OpenRouter extensions.** `Inputs` is a string or an array with 52 item variants. Points that matter for round trips:

- `ReasoningItem` is "Reasoning output item with signature and format extensions": OpenAI's reasoning item (`id`, `type: "reasoning"`, `summary`, `content?`, `encrypted_content?`, `status?`) plus `format?` (`ReasoningFormat`), `signature?` (`string\|null`), and a nullable `content`.
- `CompactionItem` (`{type: "compaction", encrypted_content, id?}`, "A context compaction marker with encrypted summary"), `ContextCompactionItem` (`{type: "context_compaction", encrypted_content?, id?}`), and `CompactionTriggerItem` (`{type: "compaction_trigger"}`, "Asks the model to compact the preceding input into a compaction item").
- `ItemReferenceItem` (`{type: "item_reference", id}`, "A reference to a previous response item by ID") is in the input union of a route that stores no responses. What the server does with it is unverified.
- `ConfigurationUpdateItem` (`{type: "configuration_update", reasoning: {effort}, id?}`): "Two adjacent configuration updates are rejected. Only supported by models that accept mid-conversation effort changes."
- 17 `openrouter:*` output item types can be replayed as input, among them `openrouter:advisor`, `openrouter:subagent`, `openrouter:fusion`, `openrouter:bash`, `openrouter:web_search`, `openrouter:armor_blocked`.

Source: S-s1. Basis: OR-documented.

**S29. Response additions on the Responses route.** `OpenResponsesResult` is OpenAI's response object (`BaseResponsesResult`) plus: `access_programs`, `alignment`, `error` (with an OpenRouter `metadata` member), `error_type` (`ApiErrorType`), `input_transformations`, `openrouter_metadata`, `safeguard_results`, `service_tier` (free string), and a `usage` object extended with `cost`, `cost_details` (`upstream_inference_cost`, `upstream_inference_input_cost`, `upstream_inference_output_cost`, `server_tool_cost`), `is_byok`, `server_tool_use_details`, `deferred_tools_details`. The response `id` in captured output starts with `gen-` (for example `"id": "gen-1794947580-aB47fruOzZLs4Q1yYYEw"`), which is OpenRouter's generation id (S-s6). Debug output on this route "arrives as a `response.debug` SSE event" (S-s9). Source: S-s1, S-s6, S-s9. Basis: OR-documented.

**S30. Error codes on the Responses route are a lossy mapping, repaired by `error_type`.** "These codes are a reduced mapping of the [internal typed error codes](https://openrouter.ai/docs/api_reference/errors-and-debugging.md#typed-error-codes)." and "Because the native `error.code` vocabulary is lossy (many distinct errors collapse to `server_error`), failed responses also include a top-level `error_type` field" (S-s7). Mapping table from the errors page: `rate_limit_exceeded` maps to `rate_limit_exceeded`; `context_length_exceeded`, `invalid_request`, `refusal` map to `invalid_prompt`; `content_policy_violation` maps to `image_content_policy_violation`; `authentication`, `provider_overloaded`, `provider_unavailable`, `timeout`, `server` map to `server_error` (S-s9). Output-limit errors become a completed response: "On the Responses API the response has `status: "incomplete"` and `incomplete_details.reason: "max_output_tokens"`" and "on Chat Completions the same case is `finish_reason: "length"`" (S-s9). Basis: OR-documented.

**S31. OpenRouter appends a routing suffix to OpenAI encrypted reasoning.** In 20 blob values found in captured output across 7 doc pages, an OpenAI `encrypted_content` value (Responses route) or `redacted_thinking.data` value (Messages route) ends with a dot and a base64url segment. Decoded examples:

- API overview, Responses output for `openai/gpt-5.2`: suffix `eyJlbmRwb2ludF9zbHVnIjoib3BlbmFpL2dwdC01LjItMjAyNTEyMTF8b3BlbmFpIn0` decodes to `{"endpoint_slug":"openai/gpt-5.2-20251211|openai"}` (S-s3).
- Service tiers guide, Messages output for `openai/gpt-5-mini`: suffix decodes to `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|openai"}` (S-s27).
- Provider selection guide: five values decode to `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|azure"}` (S-s14).

The same overview output also tags the reasoning item with `"format": "openai-responses-v1"`. Basis: OR-inferred, from captured output in OpenRouter's own docs; no doc sentence explains the suffix. Confidence: high that the suffix exists and names the model version and provider; the purpose (routing a replay back to the endpoint that can decrypt it) is an inference. Related documented behavior for another vendor: "The `data` value wraps the Gemini token in an OpenRouter envelope so its format survives the round trip." (S-s26).

### 5. The Messages route

**S32. All 29 top-level properties of `MessagesRequest`.** Generated from the spec. `required` is `["model", "messages"]`. The status column compares against Anthropic's official Python SDK types at release 2.12.0 (S-s50 `types/message_create_params.py` L34-305 for GA, `types/beta/message_create_params.py` L38-394 for beta).

| Field | Required | OpenRouter spec type | Status | Note |
|---|---|---|---|---|
| `cache_control` | no | `AnthropicCacheControlDirective` | Anthropic standard |  |
| `context_management` | no | `{edits?: array<({clear_at_least?: AnthropicInputTokensClearAtLeast, clear_tool_inputs?: (boolean \| array<string> \| null), exclude_tools?: array<str...` | Anthropic beta field | Spec lists three edit types: `clear_tool_uses_20250919`, `clear_thinking_20251015`, `compact_20260112`. |
| `deferred_tools` | no | `DeferredToolsControl` | OpenRouter addition |  |
| `fallbacks` | no | `array<MessagesFallbackParam>\|null` | Anthropic beta name, OpenRouter meaning | Spec: "Handled by OpenRouter multi-model routing rather than Anthropic server-side fallbacks; cannot be combined with `models`. Each entry accepts only `model`. Maximum of 3 entries." |
| `max_tokens` | no | `integer` | Anthropic standard | Optional in OpenRouter's spec. `Required[int]` in Anthropic's SDK (S-s50 `message_create_params.py` L35). |
| `messages` | yes | `array<MessagesMessageParam>\|null` | Anthropic standard | `MessagesMessageParam`: `role` enum is `user`, `assistant`, `system`; message-level `clear_at` and `output_config`. |
| `metadata` | no | `{user_id?: string\|null}` | Anthropic standard | `{user_id?: string\|null}`. |
| `model` | yes | `string` | Anthropic standard |  |
| `models` | no | `array<string>` | OpenRouter addition |  |
| `output_config` | no | `MessagesOutputConfig` | Anthropic standard | `effort` (`low`..`max`), `format` (`json_schema`), `task_budget` (`{type: "tokens", total, remaining?}`). `task_budget` is in Anthropic's beta params only (S-s50 `beta/beta_output_config_param.py` L28). |
| `plugins` | no | `array<(AutoRouterPlugin \| AutoBetaRouterPlugin \| ModerationPlugin \| WebSearchPlugin \| WebFetchPlugin \| FileParserPlugin \| ...8 more)>` | OpenRouter addition |  |
| `provider` | no | `ProviderPreferences` | OpenRouter addition |  |
| `route` | no | `DeprecatedRoute` | OpenRouter addition |  |
| `safeguards` | no | `array<AnthropicSafeguard>\|null` | Not in Anthropic's SDK types | Spec: "A server-side safeguard Anthropic evaluates alongside the completion". No `safeguards` field in `MessageCreateParamsBase`, GA or beta, at S-s50. |
| `service_tier` | no | `string` | Anthropic standard name, wider type | Spec type is plain `string`. Anthropic's enum is `auto`, `standard_only` (S-s50 L158). OpenRouter docs send `"flex"` and `"priority"` here (S-s27). |
| `session_id` | no | `string [maxLength=256]` | OpenRouter addition |  |
| `speed` | no | `(AnthropicSpeed & any)` | Anthropic standard |  |
| `stop_sequences` | no | `array<string>` | Anthropic standard |  |
| `stop_server_tools_when` | no | `StopServerToolsWhen` | OpenRouter addition |  |
| `stream` | no | `boolean` | Anthropic standard |  |
| `system` | no | `(string \| array<AnthropicTextBlockParam>)` | Anthropic standard |  |
| `temperature` | no | `number` | Not in Anthropic's SDK types | No `temperature` field anywhere under `src/anthropic/types` at S-s50. OpenRouter's spec keeps it. |
| `thinking` | no | `({block_binding?: AnthropicThinkingBlockBinding, budget_tokens: integer, display?: AnthropicThinkingDisplay, type: enum["enabled"]} \| {type: enum["...` | Anthropic standard | Four variants: `enabled` (`budget_tokens`, `display?`, `block_binding?`), `disabled`, `adaptive` (`display?`, `block_binding?`), `between_tools`. `display` enum: `summarized`, `omitted`, `updates`. |
| `tool_choice` | no | `({disable_parallel_tool_use?: boolean, type: enum["auto"]} \| {disable_parallel_tool_use?: boolean, type: enum["any"]} \| {type: enum["none"]} \| {dis...` | Anthropic standard | `auto`, `any`, `none`, `tool`, with `disable_parallel_tool_use`. |
| `tools` | no | `array<({cache_control, defer_loading, description, input_schema, name, strict, ...} \| {cache_control?: AnthropicCacheControlDirective, name: enum["...` | Anthropic standard name, different union | 17 variants: Anthropic `custom`, `bash_20250124`, `text_editor_20250124`, `web_search_20250305`, `web_search_20260209`, `advisor_20260301`, tool search tools, plus OpenRouter `openrouter:*` server tools and a generic `{type: string}`. |
| `top_k` | no | `integer` | Not in Anthropic's SDK types | No `top_k` field anywhere under `src/anthropic/types` at S-s50. OpenRouter's spec keeps it. |
| `top_p` | no | `number` | Not in Anthropic's SDK types | No `top_p` field anywhere under `src/anthropic/types` at S-s50. OpenRouter's spec keeps it. |
| `trace` | no | `TraceConfig` | OpenRouter addition |  |
| `user` | no | `string [maxLength=256]` | OpenRouter addition | OpenRouter end-user identifier, max 256 chars. |

Source: S-s1, S-s50. Basis: OR-documented for the OpenRouter columns; Vendor-documented (Anthropic SDK source) for the status column.

**S33. Status of each Anthropic field the task asked about.**

| Anthropic field | In `MessagesRequest`? | Notes |
|---|---|---|
| `thinking` | Yes | Four variants (S32 table). Normalized into `reasoning` for every model (S35). |
| `cache_control` (top level and on blocks) | Yes | Top level, on text, image, document, tool use, tool result blocks, and on tool definitions |
| `context_management` | Yes | `edits[]` of type `clear_tool_uses_20250919`, `clear_thinking_20251015`, `compact_20260112`. Response returns `context_management.applied_edits`. |
| `container` | Outside the request schema | Anthropic: `container: Optional[MessageCreateParamsContainerParam]` (S-s50 L135). OpenRouter's response schema does list `container` (`BaseMessagesResult.container`), and sample output shows `"container":null` (S-s27). |
| `mcp_servers` | Outside the request schema | Anthropic beta: `mcp_servers: Iterable[BetaRequestMCPServerURLDefinitionParam]` (S-s50 beta L226). Zero occurrences of `mcp_servers` in the OpenRouter spec. |
| `output_config` | Yes | `effort`, `format`, `task_budget` |
| `tool_choice` | Yes | `auto`, `any`, `none`, `tool` |
| `fallbacks` | Yes | Remapped to OpenRouter model routing (S32 table) |
| `speed` | Yes | `fast`, `standard` |
| `diagnostics`, `inference_geo`, `user_profile_id`, `workspace_id` | Outside the request schema | Anthropic GA fields at S-s50 L138, L146, L299, L305 |
| `billing_policy`, `compaction`, `fallback_credit_token`, `thread` | Outside the request schema | Anthropic beta fields at S-s50 beta L133, L156, L186, L289 |
| Beta headers | See S51 | `x-anthropic-beta` is forwarded by allowlist; several betas are attached by OpenRouter itself |

OpenRouter additions on this route: `models`, `provider`, `plugins`, `route`, `session_id`, `trace`, `user`, `deferred_tools`, `stop_server_tools_when`. Sources: S-s1, S-s50. Basis: OR-documented; OR-inferred for the "outside the request schema" rows.

**S34. Message-level and block-level shape on the Messages route.** `MessagesMessageParam` ("Anthropic message with OpenRouter extensions"): `role` enum `user`, `assistant`, `system`; `content`; `clear_at?` (`next_user_message`, `never`); `output_config?` (`{effort}`, `additionalProperties: false`). Input content block types: `text`, `image`, `document`, `tool_use`, `tool_result` (with `is_error?` and nested `text`, `image`, `tool_reference`, `search_result`, `document`), `thinking` (`{thinking, signature}`), `redacted_thinking` (`{data}`), `server_tool_use`, `web_search_tool_result`, `search_result`, `compaction` (`{content, encrypted_content?}`), and OpenRouter blocks `MessagesAdvisorToolResultBlock`, `MessagesToolAdditionBlock`, `MessagesToolRemovalBlock`, `MessagesShellToolResultBlock`, `MessagesBashToolResultBlock`, `ORAnthropicToolSearchResultParam`. Response content block types (`ORAnthropicContentBlock`, 17 members): `text`, `tool_use`, `thinking`, `redacted_thinking`, `server_tool_use`, `web_search_tool_result`, `web_fetch_tool_result`, `code_execution_tool_result`, `bash_code_execution_tool_result`, `text_editor_code_execution_tool_result`, `tool_search_tool_result`, `container_upload`, `compaction`, `advisor_tool_result`, `openrouter_shell_tool_result`, `openrouter_bash_tool_result`, `openrouter_tool_search_result`. Source: S-s1. Basis: OR-documented.

**S35. `thinking` and `output_config.effort` are translated to the unified `reasoning` setting.** Reasoning guide table:

| Messages API request | Unified `reasoning` equivalent |
|---|---|
| `output_config: { effort }` | `reasoning: { effort }` |
| `thinking: { type: "enabled", budget_tokens }` | `reasoning: { max_tokens: budget_tokens }` |
| `thinking: { type: "adaptive" }` | `reasoning: { enabled: true }` |
| `thinking: { type: "disabled" }` | `reasoning: { enabled: false }` |

Loss stated on the same page: "The unified `reasoning` parameter accepts either `effort` or `max_tokens` but not both, so every other model receives `reasoning.effort` and the budget is not forwarded." and "OpenAI models expose two knobs, `reasoning.effort` and `text.verbosity`, and the Messages API request shape has one. `output_config.effort` maps to `reasoning.effort` only, and it is not sent as `text.verbosity`." (S-s18). Controls that exist on this route only: `thinking: {"type": "between_tools"}` ("Send it through OpenRouter's [Messages API](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md).", S-s24), `thinking.display: "updates"` ("If you're using the Anthropic Messages API format directly, you can control this with `thinking.display`", S-s18), and `thinking.block_binding.prefix_mismatch_behavior` (S-s22). Basis: OR-documented.

**S36. Features that exist on the Messages route only, with beta headers attached by OpenRouter.**

- Fable 5.1 guide: "The new request controls are available on OpenRouter's [Messages API](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md) (`/api/v1/messages`) only. You do **not** need to send the Anthropic beta headers for any of them: OpenRouter detects requests that use each feature, attaches the corresponding beta for you, and routes those requests only to providers that support it." The controls: ephemeral mid-conversation system messages (`clear_at`), per-turn effort, `thinking.display: "updates"`, and prefix-lock handling (S-s22).
- Opus 5 guide, on `tool_addition` and `tool_removal` blocks: "the Chat Completions API has no representation for these blocks. You do **not** need to send the `anthropic-beta: mid-conversation-tool-changes-2026-07-01` header: OpenRouter detects requests that use tool changes and attaches the beta for you, and routes those requests only to providers that support it." (S-s23).
- Per-turn effort is the one control of that set with a chat completions form: "The same change is available through the Chat Completions API (`configuration_update` on a content-less system message) and the Responses API (a `configuration_update` input item). OpenRouter translates each form into the `output_config` message Fable expects." (S-s22).

Basis: OR-documented.

**S37. Response additions on the Messages route.** `MessagesResult` ("Non-streaming response from the Anthropic Messages API with OpenRouter extensions") is Anthropic's message plus `provider` (string), `openrouter_metadata`, `safeguard_results`, `context_management.applied_edits`, and `usage` extended with `cost`, `cost_details`, `is_byok`, `deferred_tools_details`, `iterations`, `server_tool_use` (`web_search_requests`, `web_fetch_requests`, `tool_calls_requested?`, `tool_calls_executed?`), `service_tier`, `speed`. `stop_reason` enum: `end_turn`, `max_tokens`, `model_context_window_exceeded`, `stop_sequence`, `tool_use`, `pause_turn`, `refusal`, `compaction`, null. Error body: `{type: "error", error: {type, message, error_type?}, request_id, metadata?, openrouter_metadata?}` with `error.type` enum `invalid_request_error`, `authentication_error`, `permission_error`, `not_found_error`, `rate_limit_error`, `api_error`, `overloaded_error`, `billing_error`, `timeout_error`. Gemini 4 guide: "Messages has no native finish reason field." (S-s26). Service tiers guide: "**Messages API** (`/api/v1/messages`): `service_tier` is returned inside the **`usage` object**, matching Anthropic's native format." and "OpenRouter normalizes provider-equivalent base tier labels, such as Google's `standard`, to `default`. The exception is the Anthropic Messages API, which preserves `standard` to match Anthropic's spec" (S-s27). Sources: S-s1, S-s26, S-s27. Basis: OR-documented.

**S38. Reasoning from a non-Anthropic model on the Messages route arrives as `thinking` with an empty signature plus `redacted_thinking`.** Captured output for `"model": "openai/gpt-5-mini"` on `/api/v1/messages` has a `content` array of three blocks in this order: `{"type":"thinking","thinking":<summary text>,"signature":""}`, then `{"type":"redacted_thinking","data":<blob starting gAAAAABq_MTy>}`, then `{"type":"text","text":<answer>}`. The text and blob are shortened here; the key names, order, and the empty `signature` are as captured (S-s27). For Gemini, a hop that stops at a limit returns `stop_reason: "max_tokens"` and a `redacted_thinking` block holding the continuation token, and the guide says "pass the assistant content back unchanged: continuation blocks look like other `redacted_thinking` blocks, such as thought signatures, and OpenRouter resumes from the last continuation block." (S-s26). Basis: OR-documented (captured output and prose in OpenRouter docs). Confidence note: the empty `signature` is seen in one captured output; no doc sentence states it as a rule.

**S39. Which features sit on which route.**

| Feature | Chat completions | Responses | Messages | Quote |
|---|---|---|---|---|
| `openrouter:bash` server tool | 400 | 400 | Yes | "`openrouter:bash` is only available on the Anthropic Messages API. Requesting it on the Chat Completions or Responses API returns a `400` error." (S-s33) |
| Shell server tool | 400 | Yes | Yes | "Requesting it on the Chat Completions API returns a `400` error." (S-s34) |
| Async tools (`async: true`) | Accepted, flag removed | Yes | Synchronous | On a chat completions tool, `async` "is accepted, but the flag is removed before the request is routed and the call behaves like any other tool call." (S-s25) |
| Subagent `inherit_functions` | 400 | Yes | 400 | "Supported on the Responses API (`/api/v1/responses`) only; other APIs reject it with a `400`." (S-s35) |
| Mid-conversation tool changes, `clear_at` | Not expressible | Not documented | Yes | S36 |
| Mid-conversation effort | `configuration_update` on a system or developer message | `configuration_update` item | System message with `output_config` | "All three OpenRouter APIs expose this, each in the shape native to that API." (S-s18) |
| Computer use tools | Not in the tool union | `ComputerUseServerTool` is in the tool union | Rejected | "OpenRouter's Messages API does not accept either computer tool form, so computer-use requests through OpenRouter are rejected at validation." (S-s24) |
| Video input | `video_url` part | Not checked | Not supported | "The Messages API (`/api/v1/messages`) does not support video inputs." (S-s59) |
| Reusable PDF `file` annotations | Yes | `file_citation` only | Not documented | S24 |
| `debug.echo_upstream_body` | Yes | Yes | Not in `MessagesRequest` | "This works with both the Chat Completions API (`/api/v1/chat/completions`) and the Responses API (`/api/v1/responses`)." (S-s9) |
| Advisor streaming | No effect | Yes | Planned | "Streaming has **no effect on the Chat Completions API**" and "Streaming the advice in the **Anthropic Messages API** is a planned fast-follow" (S-s37) |
| Fast mode spelling | `service_tier` | `service_tier` | `speed`, `service_tier` | "Send `speed: "fast"`, `service_tier: "fast"`, or `service_tier: "priority"`" and "the three spellings are fully interchangeable on all APIs" (S-s19) |
| Armor `output_mode` | `text` | `custom_item` | `claude_code_compatibility` | "`custom_item` (Responses API only)" and "`claude_code_compatibility` (Messages API only" (S-s1, `ArmorPlugin.output_mode`) |

Basis: OR-documented. Conflict noted: the chat tool union in the spec lists `BashServerTool` (S15) while the bash guide says the chat route returns a 400 for it, see Conflicts.

**S40. How coding harnesses are pointed at each route.**

- Claude Code and the Anthropic Agent SDK use the Messages route: `export ANTHROPIC_BASE_URL="https://openrouter.ai/api"`, `export ANTHROPIC_AUTH_TOKEN="$OPENROUTER_API_KEY"`, `export ANTHROPIC_API_KEY="" # Important: Must be explicitly empty`. "Claude Code sends `ANTHROPIC_AUTH_TOKEN` as `Authorization: Bearer <token>`" (S-s19, S-s20). "OpenRouter strips the marker before routing", said of the `[1m]` model name suffix (S-s19).
- Codex CLI uses the `/api/v1` base: `base_url="https://openrouter.ai/api/v1"` with `model="~openai/gpt-sol-latest"` (S-s21). The changelog names the Responses route as the one "OpenAI-compatible clients (including the OpenAI Agents SDK)" call (S-s39).

Basis: OR-documented.

### 6. Unsupported parameters: ignored, routed around, or rejected

**S41. The default rule is "ignored".** API overview: "If the chosen model doesn't support a request parameter (such as `logit_bias` in non-OpenAI models, or `top_k` for OpenAI), then the parameter is ignored. The rest are forwarded to the underlying model API." (S-s3). Provider selection guide: "With the default routing strategy, providers that don't support all the [LLM parameters](https://openrouter.ai/docs/api_reference/parameters.md) specified in your request can still receive the request, but will ignore unknown parameters." (S-s14). Spec, `ProviderPreferences.require_parameters`: "If this setting is omitted or set to false, then providers will receive only the parameters they support, and ignore the rest." (S-s1). Basis: OR-documented.

**S42. An absent sampling parameter stays absent upstream.** "When a sampling parameter is absent from your request, OpenRouter omits it upstream rather than substituting a hardcoded value, so the provider applies its own default. The "Default" listed for each parameter below is the conventional value, not one OpenRouter injects. Explicitly sending it (e.g. `temperature: 1.0`) is still forwarded and may differ from omitting it (for example, it can affect provider-side cache keys)." (S-s4). One documented default that OpenRouter does inject: "When `max_tokens` is omitted, OpenRouter sends its default output budget of 65,536 tokens" for Gemini 4 Argon (S-s26), and the debug sample for a Claude Haiku request shows `"max_tokens": 65536` in the upstream body (S-s9). Basis: OR-documented.

**S43. `provider.require_parameters` turns "ignored" into "routed around, or failed".** "When you set `require_parameters` to `true`, the request won't even be routed to that provider." (S-s14). Compatibility guide: "Setting `provider: { require_parameters: true }` ensures OpenRouter never routes requests with `json_schema` or `tools` to backend providers that don't support those parameters: if no endpoint supports them, the request fails instead of falling back to one that would drop them. It doesn't guarantee that the selected provider enforces `strict: true` exactly." (S-s13). Basis: OR-documented.

**S44. Three parameters are a routing preference even without `require_parameters`.** "Even when `require_parameters` is `false`, a small set of parameters is used as a soft preference when choosing between providers of the same model: `tools`, the response format (`text.format` on the Responses API, `response_format` on Chat Completions, including structured outputs), and `verbosity`. If some of a model's providers support one of these parameters and others don't, the request is only routed to the supporting providers. If none of a model's providers support the parameter, the request is still routed to that model and the parameter is ignored" (S-s14). Also: "When you send a request with `tools` or `tool_choice`, OpenRouter makes a best effort to route to providers known to support tool use. Similarly, if you set `max_output_tokens` (or `max_tokens` on Chat Completions), then OpenRouter will only route to providers that support a response of that length." (S-s14). Basis: OR-documented.

**S45. The catalog publishes the per-model parameter list that drives this.** Each model has `supported_parameters`. Across 476 models the vocabulary has 26 values: `max_tokens` (459 models), `response_format` (419), `tools` (414), `tool_choice` (409), `structured_outputs` (407), `seed` (362), `include_reasoning` (359), `reasoning` (359), `temperature` (354), `top_p` (343), `stop` (306), `reasoning_effort` (245), `frequency_penalty` (228), `top_k` (225), `presence_penalty` (222), `logprobs` (169), `top_logprobs` (169), `repetition_penalty` (158), `logit_bias` (138), `min_p` (117), `verbosity` (114), `max_completion_tokens` (95), `parallel_tool_calls` (18), `web_search_options` (18), `top_a` (17), `prediction` (9). Example: `anthropic/claude-haiku-5.5` lists `include_reasoning`, `max_completion_tokens`, `max_tokens`, `reasoning`, `reasoning_effort`, `response_format`, `stop`, `structured_outputs`, `tool_choice`, `tools`, `verbosity`, while `anthropic/claude-haiku-4.5` also lists `temperature`, `top_k`, `top_p` (S-s2). The migration guide says of the same model: "Haiku 5.5 rejects any `temperature` other than `1`, any `top_p` other than its default, any `top_k`, and requests that set both `temperature` and `top_p`. Remove all three from your requests" (S-s24). Basis: OR-documented for the catalog data. OR-inferred that OpenRouter drops `temperature` for Haiku 5.5 before the provider sees it (inferred from the catalog list plus the rule in S41; the guide's wording "rejects" describes the model and does not say which side acts).

**S46. Provider-specific extras are forwarded only to the matching provider.** Parameters page: "OpenRouter will also transmit some provider-specific parameters, such as `safe_prompt` for Mistral or `raw_mode` for Hyperbolic directly to the respective providers if specified." (S-s4). Spec, `provider.options` (`ProviderOptions`, one optional object per provider slug, 158 slugs listed): "Provider-specific options keyed by provider slug. The provider segment is case-insensitive; an endpoint tag after `/` must match the endpoint slug exactly. Only options for the matched provider are forwarded; the rest are ignored. Unrecognized keys are silently dropped." (S-s1). Basis: OR-documented.

**S47. Documented exceptions where OpenRouter rejects with an error.** Each row is OpenRouter's own rejection, taken from its docs or spec.

| Case | Status | Quote | Source |
|---|---|---|---|
| `previous_response_id` non-null, or `store: true`, on `/responses` | 400 | "are rejected with a `400` error" | S-s6 |
| Mid-conversation effort update sent to a model that lacks support | 400 | "a request for a model that does not support it is rejected with a 400 rather than sent with the update silently dropped" | S-s25 |
| Effort value in an update with no equivalent on the model | 400 | "An update whose effort has no supported equivalent is rejected with a 400." | S-s25 |
| `configuration_update` on a user, assistant, or tool message | 400 | "A `configuration_update` on a user, assistant, or tool message is rejected with a 400." | S-s25 |
| Two adjacent updates; update plus `truncation: "auto"`; update plus `reasoning.mode: "pro"` | 400 | "that combination is rejected with a 400" | S-s18 |
| GPT-6 features on an endpoint without support | 400 | "OpenRouter sends a request that uses these features only to endpoints that support it and rejects it with a 400 otherwise." | S-s25 |
| Disabling reasoning on a model where it is mandatory | 400 | "OpenRouter returns a 400 (`Reasoning is mandatory for this endpoint and cannot be disabled.`) for `enabled: false` or `effort: "none"`." | S-s24 |
| Replayed thinking blocks when `provider.only` or `provider.ignore` excludes the provider that can read them | 400 | "OpenRouter returns a 400 asking you to allow that provider or start a new conversation without the thinking blocks." | S-s24 |
| `openrouter:bash` on chat completions or Responses; shell tool on chat completions | 400 | S39 quotes | S-s33, S-s34 |
| `inherit_functions` or `inherited_function_names` off the Responses route | 400 | "other APIs reject it with a `400`" | S-s35 |
| Function tools inside a subagent's nested `tools` | 400 | "function tools are rejected with a `400` because the worker has no way to execute them" | S-s35 |
| Computer use tools on `/messages` | Rejected at validation | S39 quote | S-s24 |
| `fallbacks` with `models`, more than 3 entries, or per-attempt overrides, on `/messages` | 400 | "`fallbacks` cannot be combined with the `models` parameter; sending both returns a 400 error." | S-s32 |
| `deferred_tools` with a forcing `tool_choice` | 400 | "Rather than silently overriding what you asked for, the request fails with a `400`" | S-s36 |
| `deferred_tools` on the Batch API; effort updates on the Batch API | Rejected | "Batch requests that set `deferred_tools` are rejected." | S-s36 |
| `access_programs.cyber` other than `standard` without an enrolled BYOK OpenAI key | 400 | "rejects it with a 400 error" (spec); on a batch, "the line is rejected before anything reaches the provider" | S-s1, S-s55 |
| Invalid `x_search` object or legacy `x_search_filter` | 400 | "is rejected with a 400 rather than sent as an unfiltered X search" | S-s56 |
| Web search: `engine: "native"` with domain filters on Google; both domain lists on Perplexity | 400 | "With `engine: "native"`, returns a 400 error" | S-s57 |
| `gs://` file reference in a system or tool message | 400 | "returns a 400 rather than being dropped from the prompt" | S-s58 |
| Explicit `service_tier`, `speed: "fast"`, or a tier endpoint slug naming a tier the workspace admin has disallowed | 403 | "A request that selects the tier explicitly is rejected with a `403`" | S-s27 |
| `reasoning_effort` and `reasoning.effort` both set and different | Stated as not allowed | "Cannot be used simultaneously with reasoning.effort if they differ." | S-s1 |
| Unknown keys on the alpha extractions route | 400 | "Unknown keys at the top level or inside a field are rejected with a `400`, not ignored, so a provider option in the wrong place never silently does nothing." | S-s61 |

Basis: OR-documented. Confidence note: the list comes from a text search of all mirrored doc pages for "rejected", "400", and "silently"; a rejection that the docs word differently would be missed.

**S48. Some rejections are the provider's own 400, passed through.** "Opus 5.5 rejects `tool_choice` set to `{"type": "any"}` or a named tool, and the Chat Completions forms `tool_choice: "required"` and `{"type": "function", "function": {"name": ...}}`, with the provider's 400 (`tool_choice: type "tool" and "any" are not supported for this model.`)." and, for assistant prefill on Haiku 5.5, "OpenRouter returns the provider's 400 (`This model does not support assistant message prefill. The conversation must end with a user message.`)." and, for prefix-locked thinking, "OpenRouter passes the upstream error through and does not drop thinking blocks for you" (S-s24, S-s22). Basis: OR-documented.

**S49. Some settings are silently reshaped, neither ignored nor rejected.** "**Budgets are not forwarded.** `reasoning.max_tokens` and `thinking.budget_tokens` are accepted by OpenRouter, but the request goes upstream as adaptive thinking with no budget" (S-s24). "Without this header, OpenRouter will strip the `strict` field and route normally." (S-s14, on `strict: true` tools for Anthropic). "Claude accepts `low`, `medium`, `high`, `xhigh`, and `max` (`minimal` is sent as `low`, `none` is rejected)." (S-s18). "The debug option **only works with streaming mode** (`stream: true`). Non-streaming requests will ignore the debug parameter." (S-s9). Basis: OR-documented.

### 7. Headers and versioning

**S50. Request headers.**

| Header | Scope | What the source says | Source |
|---|---|---|---|
| `Authorization: Bearer <credential>` | All routes | "Send one credential per request in the `Authorization` header as `Bearer <credential>`." Accepts an `sk-or-v1-` API key or an OpenRouter-issued JWT. | S-s11 |
| `x-api-key` | All routes | "For clients built on the Anthropic SDK, you can also send an API key (not an access token) in the `x-api-key` header." | S-s11 |
| `HTTP-Referer` | Global spec parameter `AppIdentifier` | "The app identifier should be your app's URL and is used as the primary identifier for rankings." | S-s1, S-s16 |
| `X-OpenRouter-Title` | Global spec parameter `AppDisplayName` | "`X-Title` is still supported for backwards compatibility." | S-s1, S-s16 |
| `X-OpenRouter-Categories` | Global spec parameter `AppCategories` | "Comma-separated list of app categories (e.g. "cli-agent,cloud-agent"). Used for marketplace rankings." | S-s1 |
| `X-OpenRouter-App-Visibility` | Docs only | "Send `hidden` to create the app hidden" | S-s16 |
| `X-OpenRouter-Metadata` | Spec parameter on `/chat/completions`, `/responses`, `/messages` | "Opt-in to surface routing metadata on the response under `openrouter_metadata`. Defaults to `disabled`. The legacy header `X-OpenRouter-Experimental-Metadata` is also accepted for backward compatibility." Enum `disabled`, `enabled`. | S-s1 |
| `x-session-id` | Named in the `session_id` description | "If provided in both the request body and the x-session-id header, the body value takes precedence." | S-s1 |
| `x-anthropic-beta` | Docs, provider selection guide | See S51 | S-s14 |
| `anthropic-version` | Docs examples; Files API | The Messages fallback example sets `req.Header.Set("anthropic-version", "2023-06-01")`. Files API: "Requests with an `anthropic-version` header get the Anthropic shape." | S-s32, S-s52 |
| `X-OpenRouter-Cache`, `X-OpenRouter-Cache-TTL`, `X-OpenRouter-Cache-Clear` | Response caching | "`X-OpenRouter-Cache` \| `true` \| Enable caching for this request" | S-s53 |
| `x-openrouter-callmodel` | Sent by the Agent SDK | "`agent-sdk` when the `x-openrouter-callmodel` header is set or the `User-Agent` identifies the agent SDK" | S-s38 |
| `x-openrouter-advisor-depth`, `x-openrouter-subagent-depth`, `x-openrouter-fusion-depth` | Set by OpenRouter on its own inner calls | The advisor guide says each inner advisor call has this header and "the advisor tool is stripped from any sub-call, so an advisor sub-agent can never re-enter the advisor". The subagent and fusion guides say the same for their headers. | S-s37, S-s35, S-s66 |

The official TypeScript SDK's chat call sends exactly four of these: `HTTP-Referer`, `X-OpenRouter-Metadata`, `X-OpenRouter-Categories`, `X-OpenRouter-Title` (S-s44 `src/funcs/chatSend.ts` L225-241). Basis: OR-documented.

**S51. `x-anthropic-beta` is forwarded through a per-provider allowlist, and some betas are attached by OpenRouter.** "When using Anthropic models (Claude), you can request specific beta features by including the `x-anthropic-beta` header in your request. OpenRouter will pass through supported beta features to Anthropic." The page lists two values, `interleaved-thinking-2025-05-14` and `structured-outputs-2025-11-13`, and shows a comma-separated form `x-anthropic-beta: interleaved-thinking-2025-05-14,structured-outputs-2025-11-13`. "OpenRouter keeps an allowlist of other `x-anthropic-beta` values for each provider and forwards only the values on the list for the provider that serves the request. Providers that serve Anthropic's Messages API directly have the longest list, and Google Vertex and Amazon Bedrock receive only the values they accept. OpenRouter drops every other value, and some values it recognizes are withheld on purpose." Managed automatically: "**Prompt caching and extended context** are enabled based on model capabilities", the structured outputs header for JSON schema response formats, and fine-grained tool streaming: "for every streamed request (`stream: true`) that includes user-defined tools, OpenRouter sets `eager_input_streaming: true` on each tool". Caller-driven: "For **strict tool use** (`strict: true` on tools), you must explicitly pass the `structured-outputs-2025-11-13` header." The doc's own example sends the header on a Responses route call (`openRouter.responses.send`). Fast mode: "The required beta header is injected automatically whenever the fast tier endpoint serves the request." (S-s19). Source: S-s14. Basis: OR-documented. Confidence note: the docs spell the request header `x-anthropic-beta`; whether the Messages route also honors Anthropic's native `anthropic-beta` header name is an open question below.

**S52. Response headers.**

| Header | When | Quote or observation | Source |
|---|---|---|---|
| `X-Generation-Id` | Every inference response | "The generation ID is returned in the `X-Generation-Id` response header for all endpoints (chat completions, completions, responses, and messages)" | S-s10 |
| `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` | 429 from OpenRouter's own free-model or key-management limits | "Successful inference responses do not include `X-RateLimit-*` headers." | S-s12 |
| `Retry-After` | 429 when providers gave a hint | Sent "When every attempted provider returned a retry hint". The errors page adds: "Respect the `Retry-After` header before retrying." | S-s12, S-s9 |
| `X-OpenRouter-Cache-Status`, `X-OpenRouter-Cache-Age`, `X-OpenRouter-Cache-TTL`, `X-OpenRouter-Cache-Source-Id` | Response caching | "`X-OpenRouter-Cache-Status` \| `HIT` or `MISS`" | S-s53 |
| `X-Provider-Name`, `x-openrouter-error-origin`, `x-openrouter-error-tool`, `request-id` | Named in a live header | A live `GET https://openrouter.ai/api/v1/models` returned `Access-Control-Expose-Headers: WWW-Authenticate,PAYMENT-REQUIRED,Payment-Receipt,PAYMENT-RESPONSE,Location,X-Generation-Id,x-openrouter-error-origin,x-openrouter-error-tool,X-Provider-Name,request-id,cf-ray` and `Vary: Host, User-Agent, Originator, Anthropic-Version, Authorization, X-Api-Key` | S-s2 |

Basis: OR-documented for the first four rows. OR-inferred for the last row: the header names are exposed to browsers by the edge, the docs mirror has zero mentions of `X-Provider-Name`, `x-openrouter-error-origin`, or `x-openrouter-error-tool`, and no inference response was observed.

**S53. Versioning and breaking-change policy.** "The OpenRouter API has a single stable version, `v1`, selected by the URL path" and "There are no version headers and no date-based version pinning." Non-breaking changes "ship without prior notice" and include "New endpoints", "New optional request parameters", "New fields in responses", "New response status codes", "New schemas, and new optional properties or union variants on existing schemas". Client guidance: "Write clients defensively: ignore response fields you don't recognize, and don't fail on unknown enum values." Breaking changes "are reviewed by a human before publication" and include "Removing or renaming an endpoint, parameter, or response field", "Changing a field's type", "Widening a response field to allow `null` when it was previously always present", "Making an optional parameter required". Deprecations: "Removal of a deprecated feature is a breaking change and follows the breaking-change process above." Changelog: "generated automatically from OpenAPI specification diffs on every release". Source: S-s5. The changelog mirror holds 105 dated entries from July 3, 2026 to November 21, 2026 (S-s39). Basis: OR-documented.

**S54. Region is chosen by host name.** "send the request to `https://eu.openrouter.ai` or `https://us.openrouter.ai`, and its prompts and completions are processed entirely within that region." (S-s14). Basis: OR-documented.

### 8. Official SDKs as a cross-check, and OpenRouter's own format converters

**S55. The official SDK repositories and their pins.** The docs name the packages: `@openrouter/sdk`, `openrouter` (PyPI), `github.com/OpenRouterTeam/go-sdk`, `@openrouter/agent`, `openrouter-agent-sdk` (PyPI), `github.com/OpenRouterTeam/go-agent` (S-s40, S-s41). "All three SDKs are auto-generated from the OpenRouter OpenAPI spec, so new models, parameters, and endpoints appear immediately after each API release." (S-s40). "The Agent SDK is built on top of the TypeScript [Client SDK](https://openrouter.ai/docs/client-sdks/overview.md)" and "The Python and Go packages are ports of `@openrouter/agent` kept in sync automatically." (S-s41). Cloned shallow on 2026-11-22:

| Repository | Commit | Commit date | Version |
|---|---|---|---|
| `OpenRouterTeam/typescript-sdk` | `7ad8458755a042a1ccf1fc96046dee3dd9055849` | 2026-11-21 | `@openrouter/sdk` 1.5.128 |
| `OpenRouterTeam/python-sdk` | `b48773eed16735206dae1d6b2af4e96182819568` | 2026-11-21 | 1.3.203 |
| `OpenRouterTeam/go-sdk` | `654d9d4d4b182a6daf12786495194d3c235c4616` | 2026-11-21 | v0.9.127 |
| `OpenRouterTeam/typescript-agent` | `8a2d96e541cb49ecc1869d490ce54b4bcf3b5a55` | 2026-11-17 | `@openrouter/agent` 1.2.2 |

Basis: OR-documented.

**S56. The generated chat message type matches the spec in all three languages.** TypeScript, `src/models/chatassistantmessage.ts` L57-95: `audio?`, `content?: string | Array<ChatContentItems> | null`, `images?: Array<ChatAssistantImages>`, `model?`, `name?`, `reasoning?: string | null`, `reasoningDetails?: Array<ReasoningDetailUnion>`, `refusal?`, `role`, `toolCalls?`. The wire names in the parser (L145-157) are `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls` (S-s44). Python, `src/openrouter/components/chatassistantmessage.py` L63-89: the same ten fields (S-s45). Go, `models/components/chatassistantmessage.go` L136-153: JSON tags `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls` (S-s46). Basis: OR-documented.

**S57. The generated reasoning-details union matches the spec.** TypeScript, `src/models/reasoningdetailunion.ts` L39-55: a union of `ReasoningDetailEncrypted`, `ReasoningDetailServerToolCall`, `ReasoningDetailSummary`, `ReasoningDetailText`, and `discriminatedUnionTypes.Unknown<"type">`, discriminated on `"reasoning.encrypted"`, `"reasoning.server_tool_call"`, `"reasoning.summary"`, `"reasoning.text"`. `src/models/reasoningdetailtext.ts` L18-25: `format?`, `id?`, `index?`, `signature?`, `text?`, `type: "reasoning.text"`. `src/models/reasoningformat.ts` L9-22: the same 12 format strings as S16, typed as `OpenEnum` (S-s44). Go JSON tags: text `format`, `id`, `index`, `signature`, `text`, `type`; encrypted `data`, `format`, `id`, `index`, `type`; summary `format`, `id`, `index`, `summary`, `type`; server tool call `arguments`, `format`, `id`, `index`, `result`, `tool_call_id`, `tool_name`, `type` (S-s46). The TypeScript union accepts an unknown `type` value on input; the format enum is open in all three. Basis: OR-documented.

**S58. The SDK's typed chat result stops at the spec's named schema.** In the TypeScript SDK, `native_finish_reason` appears only in `src/models/batchobject.ts` (L505, L1740) and `src/models/generationresponse.ts` (L119, L278), and `annotations` appears in `batchobject.ts`, `responseoutputtext.ts`, and four unrelated models. `src/models/chatchoice.ts` and `src/models/chatresult.ts` hold neither, and `chatresult.ts` holds no `provider` (S-s44). Go, `models/components/chatchoice.go` L12-18: `finish_reason`, `index`, `logprobs`, `message`; `chatresult.go` L38-56: `alignment`, `choices`, `created`, `id`, `model`, `object`, `openrouter_metadata`, `safeguard_results`, `service_tier`, `system_fingerprint`, `usage` (S-s46). Basis: OR-documented. Confidence note: whether the SDK parsers pass unknown JSON keys through to the caller was not tested.

**S59. `fromChatMessages`: chat completions messages into Responses input items.** Source: `packages/agent/src/lib/chat-compat.ts` L114-161 (S-s47).

| Chat input | Output item | Kept | Dropped or flattened |
|---|---|---|---|
| `tool` message | `{type: "function_call_output", callId, output}` | `tool_call_id`, content | Non-string content is turned into a JSON string by `contentToString` (L83-91) |
| `assistant` message | A message item with `role: "assistant"` when it has text or has zero tool calls, then one `{type: "function_call", callId, id, name, arguments, status: "completed"}` per tool call | `content` as a string, `tool_calls[].id`, `.function.name`, `.function.arguments` ("already a JSON string in the chat format, so it is forwarded as-is") | The function reads only `msg.content` and `msg.toolCalls`. `reasoning`, `reasoning_details`, `images`, `audio`, `refusal`, `name`, and `model` are left behind. |
| `system`, `user`, `developer` message | A message item with the same role | Role, content as a string | A content array (image, audio, file, video parts) becomes one JSON string; `name` and `configuration_update` are left behind |

Basis: OR-documented (official SDK source).

**S60. `toChatMessage`: a Responses result into one chat assistant message.** `toChatMessage` is `extractMessageFromResponse`, which takes the first `message` output item and calls `convertToAssistantMessage` (`packages/agent/src/lib/stream-transformers.ts` L727-759). That function returns exactly `{role: 'assistant', content: textContent || null}`, where the text is the joined `output_text` parts. Reasoning items, function calls, annotations, refusal parts, and every other output item stay out of the result. A response with no message item raises `No message found in response output` (S-s47). Basis: OR-documented (official SDK source).

**S61. `fromClaudeMessages`: Anthropic messages into Responses input items.** Source: `packages/agent/src/lib/anthropic-compat.ts` L83-235 and `packages/agent/src/api-shape-helpers/claude-message.ts` L243-256 (S-s47). The code comment at L64-66 says: "Note: Some Claude features are lost in conversion as OpenRouter doesn't support them:", "- cache_control on content blocks", "- is_error flag on tool_result blocks".

| Claude input | Output item | Notes |
|---|---|---|
| String content | Message item with role `user` or `assistant` | |
| `text` blocks | Joined into one string message (or `input_text` parts when the message also has images) | `cache_control` dropped |
| `image` block (`base64` or `url` source) | `{type: "input_image", detail: "auto", imageUrl}`; base64 becomes a `data:` URL | `cache_control` dropped |
| `tool_use` block | `{type: "function_call", callId, name, arguments: JSON.stringify(input), id, status: "completed"}` | Emitted before the message's text, so block order inside a message changes |
| `tool_result` block | `{type: "function_call_output", callId, output}` holding the joined text | `is_error` dropped. An image inside a tool result becomes a separate `{type: "image_generation_call", id: "<tool_use_id>-image-<i>", result, status}` item. A result with no text emits no output item. |
| Any other block type | Throws `Unhandled content block type` | The accepted input type lists only `text`, `image`, `tool_use`, `tool_result`, so `thinking`, `redacted_thinking`, `document`, and server tool blocks are outside it. The accepted `role` values are `user` and `assistant`. |

The docs page states the loss more briefly: "Some Claude features are not preserved in conversion. e.g. `is_error` flag on tool\_result blocks" and "These features are Claude-specific and not supported by OpenRouter." (S-s42). Basis: OR-documented (official SDK source and docs).

**S62. `toClaudeMessage`: a Responses result into an Anthropic message.** `toClaudeMessage` is `convertToClaudeMessage` (`stream-transformers.ts` L1106-1300, S-s47).

| Responses output | Claude output | Notes |
|---|---|---|
| `message` with `output_text` parts | `{type: "text", text, citations?}` | `url_citation` becomes `web_search_result_location` with `cited_text: ''` and `encrypted_index: ''`; `file_citation` and `file_path` become `char_location` with zeroed offsets (L1005-1070) |
| `refusal` part | Entry in `unsupported_content` | Reason string: "Claude does not have a native refusal content type" |
| `function_call` | `{type: "tool_use", id: callId, name, input}` | Arguments are parsed; a parse failure yields `{_raw_arguments: ...}` |
| `reasoning` with summary text | `{type: "thinking", thinking: summaryItem.text, signature: ''}` (L1205-1213) | The signature is an empty string |
| `reasoning` with `encryptedContent` | Entry in `unsupported_content` with `original_type: 'reasoning_encrypted'` and `data: {id, encrypted_content}` (L1217-1225) | Reason string: "Encrypted reasoning content preserved for round-trip". It sits beside the message `content`, outside the Anthropic block list. |
| `web_search_call` | `{type: "server_tool_use", name: "web_search", input: {status}}` | |
| `file_search_call` | `{type: "tool_use", name: "file_search", input: {queries, status}}` | |
| `image_generation_call` | Entry in `unsupported_content` | Reason string: "Claude does not support image outputs in assistant messages" |
| Any other item type | Skipped | Comment: "Unknown output types (e.g. new server tools) are skipped during Claude format conversion" |
| Status | `stop_reason`: `tool_use` when any function call exists, `max_tokens` for `incomplete` with `max_output_tokens`, else `end_turn` | `stop_sequence` is always `null` (L1075-1100, L1289) |
| Usage | `input_tokens`, `output_tokens`, `cache_creation_input_tokens: inputTokensDetails.cachedTokens ?? 0`, `cache_read_input_tokens: 0` (L1290-1295) | The cached-read count is written into the cache-creation field and the cache-read field is fixed at 0 |

Basis: OR-documented (official SDK source). Confidence note: this is the client-side converter in `@openrouter/agent`. It shows how OpenRouter's SDK team maps the formats; it is separate code from the closed server-side translator behind `/messages`. One point where the two agree is the empty `signature` on a `thinking` block built from a summary (S38).

### 9. Fills from other projects for points the OpenRouter docs leave open

**S63. LiteLLM runs its multi-vendor Messages route as native pass-through for Anthropic-family providers and as a chat completions round trip for everything else.** `litellm/llms/anthropic/messages/architecture.md` at commit `90dce4b`: the flowchart sends `anthropic`, `azure_ai`, `bedrock invoke`, and `vertex_ai` to provider-specific Messages configs and "Other providers" to `LiteLLMAnthropicMessagesAdapter`, whose steps are `translate_anthropic_to_openai`, `litellm.completion`, `translate_openai_response_to_anthropic`. The sequence diagram notes what the adapter maps: "messages, tools, thinking,<br/>output_format → response_format" (S-s51). Basis: Filled (LiteLLM). This shows one way to serve a Messages route over non-Anthropic models. It does not show what OpenRouter does.

**S64. LiteLLM forwards Anthropic beta headers through a per-provider mapping file.** `litellm/anthropic_beta_headers_manager.py` L10-14: "JSON config contains mapping of beta headers for each provider", "Keys are input header names, values are provider-specific header names (or null if unsupported)", "Only headers present in mapping keys with non-null values can be forwarded". The bundled `litellm/anthropic_beta_headers_config.json` has sections `anthropic` (77 keys), `azure_ai` (77), `bedrock_converse` (34), `bedrock` (37), `bedrock_mantle` (36), `vertex_ai` (77), `vertex_ai_header_only` (18), `databricks` (34). Example: `advanced-tool-use-2025-11-20` maps to itself for `anthropic` and to `tool-search-tool-2025-10-19` for `bedrock` and `vertex_ai` (S-s51). Basis: Filled (LiteLLM). It matches the design OpenRouter describes in prose in S51 (an allowlist per provider) and adds a detail OpenRouter's docs leave unsaid: a beta can need a different name per provider.

**S65. LiteLLM emulates Anthropic `context_management` edits inside the gateway.** `litellm/llms/anthropic/messages/context_management/dispatcher.py` L1 and L30-33: "Dispatch ``context_management`` edits to registered polyfill editors." with a registry of `CLEAR_TOOL_USES_EDIT_TYPE: apply_clear_tool_uses_20250919` and `COMPACT_EDIT_TYPE: apply_compact_20260112` (S-s51). Basis: Filled (LiteLLM). It shows that a gateway can apply these edits itself for providers without native support. Whether OpenRouter does the same for non-Anthropic models on `/messages` is an open question below.

### 10. Local survey claims LC1 to LC4

**S66. LC1 holds.** "`https://openrouter.ai/api/v1` serves Chat Completions, a Responses-shaped route, and an Anthropic Messages surface. The model catalog returns 200 with no auth header." Confirmed by the spec (S1), and by a live `GET https://openrouter.ai/api/v1/models` sent with no `Authorization` header on 2026-11-22 at 16:04 UTC: status 200, 925,527 bytes, 476 models, `Cache-Control: public, max-age=120, stale-while-revalidate=3600, stale-if-error=3600` (S-s2). Additions since the survey: the legacy `/completions` route and the per-model `endpoints` list (S3, S6). Basis: OR-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (S-s62); `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/openrouter.rs` (S-s63: "no auth on the listing endpoint (verified live 2026-09-14, a 718 KB payload)").

**S67. LC2 holds, with a longer list.** "The request is a superset: extra sampling controls, `models` as a fallback list, a structured `provider` block, plugins, `session_id`, and a debug switch that returns the transformed body sent upstream." Each named item is in today's spec: `top_k`, `top_a`, `min_p`, `repetition_penalty`; `models`; `provider`; `plugins`; `session_id`; `debug.echo_upstream_body` ("If true, includes the transformed upstream request body in a debug chunk at the start of the stream. Only works with streaming mode."). The full set of 16 OpenRouter-only top-level chat fields is in S10: `cache_control`, `debug`, `deferred_tools`, `image_config`, `min_p`, `models`, `plugins`, `provider`, `reasoning`, `repetition_penalty`, `route`, `session_id`, `stop_server_tools_when`, `top_a`, `top_k`, `trace`. Basis: OR-documented. Also in workspace: the 2026-10-28 survey (S-s62).

**S68. LC3 holds as the default and needs a list of exceptions.** "Unsupported parameters are silently ignored, not rejected." The default rule is quoted in S41. It is bounded by three things the survey sentence leaves out: `provider.require_parameters` (S43), the soft routing preference for `tools`, response format, and `verbosity` (S44), and a set of at least 22 documented rejections (S47), several of which OpenRouter words as deliberate departures from silent dropping ("rather than sent with the update silently dropped", "Rather than silently overriding what you asked for", "rather than being dropped from the prompt"). Basis: OR-documented. Also in workspace: the 2026-10-28 survey (S-s62), see Conflicts item 9.

**S69. LC4 holds.** "The `/responses` route rejects `previous_response_id` with a 400 when it is non-null, and `store` must be absent or `false`." Confirmed word for word by the spec and the docs banner (S26), and re-fetched live on 2026-11-22. Basis: OR-documented. Also in workspace: the 2026-10-28 survey (S-s62), which cites the same API reference page read on 2026-10-28.

## Conflicts and disagreements

Conflicts between OpenRouter sources:

1. **Named response schema versus the wire.** The spec's `ChatResult`, `ChatChoice`, and `ChatAssistantMessage` leave out `provider`, `native_finish_reason`, and `annotations`. The API overview, the streaming page, the PDF guide's captured output, and a second inline schema in the same spec (`BatchObject.results[].response.body`) all show them (S19). The captured output and the inline batch schema are right about the wire. The named schema is incomplete, and the three generated SDKs inherit the gap (S58).
2. **Named request schema versus the parameters page.** The parameters page and the catalog list `verbosity`, `web_search_options`, `include_reasoning`, and `structured_outputs`; the presets guide lists `preset`. `ChatRequest` lists none of the five (S12). Both can be true because the schema is open (S11). No source read today settles which of the five still act on the chat route.
3. **`openrouter:bash` on chat completions and Responses.** The spec puts `BashServerTool` in the chat tool union and in the Responses tool union (S15, S25). The bash guide says "Requesting it on the Chat Completions or Responses API returns a `400` error." (S-s33). Not resolved here. The guide is the more specific statement.
4. **How many request shapes.** The API overview says "two request shapes" (S-s3), the errors page says "three API skins" (S-s9), the router metadata guide lists four routes (S-s17), and the broadcast guide names five text-generation request types (S-s38). The spec has three chat-like operations. These read as pages written at different times, each counting what it documents.
5. **The API overview's TypeScript request type is older than the spec.** It lists `route?: 'fallback'` under "OpenRouter-only parameters" while the spec marks `route` deprecated. Its `Message` type has roles `user`, `assistant`, `system`, `tool` and a `name?` on tool messages, while the spec adds `developer` and gives tool messages three fields. Its `ToolChoice` has `none`, `auto`, and a named function, while the spec adds `required` and a server tool choice (S-s3 against S-s1). The spec is the newer source.
6. **`reasoning_effort` values.** The parameters page says "(xhigh, high, medium, low, minimal, none)" (S-s4). The spec enum also has `max` (S10). The spec is the newer source; the same parameters page lists `max` for `verbosity`.
7. **`beta.responses` tag.** The July 25, 2026 changelog entry says the tag was removed and the route is GA (S-s39). Today's spec lists the tag again on the operation, defined as a "Deprecated alias of responses" (S-s1). The GA statement stands; the tag is back as an SDK alias.

Disagreement between OpenRouter and a vendor source:

8. **Sampling parameters on the Messages shape.** OpenRouter's `MessagesRequest` lists `temperature`, `top_p`, `top_k` (S32). Anthropic's official Python SDK at release 2.12.0 has no field of those names anywhere under `src/anthropic/types` (S-s50, checked by text search). OpenRouter's route serves many vendors and older Claude models whose catalog entries still list `temperature` (S45), so the wider schema is consistent with its own catalog.

Where these findings differ from local Runvil documents:

9. **`runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (S-s62), on LC3.** The survey calls the `/responses` rejection of `previous_response_id` "the opposite of the general rule" and treats it as the one hard edge. Today's docs give 22 rejection cases (S47), and several are worded as a deliberate choice against silent dropping. The survey's default rule is still right. The survey's sentence "A gateway that models "aggregator ignores unsupported fields" as a blanket rule is wrong on this call" is right and applies to more calls than the one it names.
10. **The 2026-11-17 report on the Anthropic Messages API and harnesses (S-s64), on the cache-write field name.** It calls `prompt_tokens_details.cache_write_tokens` "an extension name that agentgateway and OpenRouter both use". OpenAI's official Python SDK at commit `f63f14b` (2026-11-20) now defines `cache_write_tokens: Optional[int] = None` in its own `PromptTokensDetails` (S-s48 `completion_usage.py` L43), and OpenAI's live chat reference page contains the name 12 times (S-s49). As of today the name is in OpenAI's own schema. This also bears on LC17, which another evidence file covers.
11. **`runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/openrouter.rs` (S-s63).** Its header comment describes a 718 KB payload verified on 2026-09-14. Today's payload is 925 KB, and each entry includes `endpoints`, `reasoning`, `benchmarks`, `knowledge_cutoff`, and `hugging_face_id`, which the file's comments leave unmentioned. The file's claims (keyless listing, base URL `https://openrouter.ai/api/v1`) still hold.

## Open questions blocked by closed source

1. **Which chat request fields outside the spec does the server act on, forward, or drop?** This covers OpenAI's `n`, `store`, `audio`, `safety_identifier`, `prompt_cache_retention`, `moderation`, `functions`, `function_call`, and OpenRouter's own older `transforms`. Tried: the spec, the parameters page, a text search of all 559 doc pages. Found: the general rule that unsupported parameters are ignored (S41) and the statement that extra parameters may be sent (S11). Fill: none found for the individual fields.
2. **Does the Messages route honor Anthropic's native `anthropic-beta` and `anthropic-version` header names?** Tried: docs search. Found: only `x-anthropic-beta` is documented (S51); `anthropic-version` appears in one Messages example and selects the response shape on the Files API (S50); the live edge lists `Anthropic-Version` in `Vary` (S52). Fill: LiteLLM's per-provider beta header mapping (S64) shows how another gateway forwards, renames, or withholds each beta per provider. It does not show OpenRouter's list.
3. **On `/messages` with a non-Anthropic model, what happens to `context_management`, `safeguards`, `speed`, `cache_control` TTLs, and `thinking.display`?** Tried: the spec, the reasoning guide, the migration guides. Found: `thinking` and `output_config.effort` are normalized for every model (S35); beta-gated features are routed "only to providers that support it" (S36). Nothing found for `context_management` on other vendors. Fill: LiteLLM applies `clear_tool_uses_20250919` and `compact_20260112` edits inside the gateway through polyfill editors (S65).
4. **How does the server-side translator lay out non-Anthropic reasoning as Anthropic blocks, and how does it read them back?** Tried: docs. Found: one captured output with a `thinking` block that has `"signature":""` followed by a `redacted_thinking` block holding the OpenAI blob (S38), and the Gemini wording about an "OpenRouter envelope" (S31). Fill: OpenRouter's own client-side converter writes the same empty signature and parks encrypted reasoning outside the block list (S62); LiteLLM routes non-Anthropic providers through a chat completions round trip (S63). Neither is the server code.
5. **What does the `{"endpoint_slug": ...}` suffix on encrypted reasoning do on replay, and what happens when a replay lands on a different provider?** Tried: a search of the docs and the spec for `endpoint_slug` (zero hits as a key). Found: the suffix itself on 20 blob values in captured output (S31), and for Claude a documented rule that replays stay on the provider that can read the blocks or fail with a 400 (S47). Fill: none found.
6. **Does `/responses` do anything with `item_reference` input items, `prompt` templates, or `background: true`?** Tried: the spec and the four Responses doc pages. Found: all three are in the schema with no description (S27, S28). Fill: none found.
7. **Is there a public Gemini `generateContent`-shaped route?** Tried: a search of the spec and all doc pages. Found: one sentence (S4). Fill: none found.
8. **What is the full request and response shape of the legacy `/completions` route?** Tried: the spec and docs. Found: `prompt`, `NonChatChoice.text` (S3). Fill: none sought; OpenAI's own spec still defines `/completions`, which this file did not compare.

## Unverified

- Whether `provider` and `native_finish_reason` appear on every chat completions response and every stream chunk. Seen in docs samples and one inline schema only (S19, S21). No inference call was made.
- Whether the three official SDKs pass unknown response keys through to the caller or strip them (S58). Not tested.
- Whether `structured_outputs` is a request field or only a catalog capability flag (S12).
- Whether `background: true`, `prompt`, and `item_reference` are acted on by `/responses` (S27, S28).
- Whether OpenRouter drops `temperature` for a model whose catalog entry omits it, or forwards it and returns the provider's error (S45). The rule in S41 says dropped; no doc sentence states it for that model.
- Whether a Gemini-shaped route exists (S4).
- The reason Anthropic's SDK types no longer list `temperature`, `top_p`, `top_k`. Only the absence in the type files at one commit was checked. Anthropic's prose docs were not read for this.
- That the empty `signature` on `thinking` blocks from non-Anthropic models is a rule and not a property of one sample (S38).
- The response headers `X-Provider-Name`, `x-openrouter-error-origin`, and `x-openrouter-error-tool` on inference responses. Only their names in a CORS header were observed (S52).
- From memory, not checked today: that LiteLLM has a `drop_params` setting for unsupported parameters; that OpenAI's API returns a 400 for an unknown top-level parameter. The local survey states the second point; this file did not re-read OpenAI's docs for it.

## Workspace documents used

- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took claims LC1 to LC4 (its lines on OpenRouter as "the reference implementation of the role"). Re-checked live today: all four hold, LC3 with exceptions (S66 to S69).
- `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/openrouter.rs` (header comment dated 2026-09-14). Took the keyless model listing and the `openai_base_url` value. Re-checked live today (S66).
- Report on the Anthropic Messages API and harness survey, cabinet output, 2026-11-17 (file date 2026-11-19). Took its statement that three conventions exist for returning thinking through chat completions and that "None is an OpenAI standard", and its statement on `cache_write_tokens`. Re-checked live: the first holds (OpenAI's live reference page has zero occurrences of `reasoning_details`, S-s49); the second is corrected in Conflicts item 10.
- `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30, S-s65). Took its note that OpenRouter reports overflow as `error_type: context_length_exceeded` and a mid-stream error as a chunk with `finish_reason: "error"`. Re-checked against today's errors page: both match (S23).

## Sources

All OpenRouter doc pages were read from the Markdown mirror fetched on 2026-11-22; the URL is the page's `.md` address.

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| S-s1 | `https://openrouter.ai/openapi.json` (OpenAPI 3.1.0, 123 paths, 1071 schemas, 3,118,352 bytes, SHA-256 starting `e6614516bb3f4059`; a second download at 16:04 UTC was byte-identical) | 2026-11-22 | OpenRouter spec |
| S-s2 | `https://openrouter.ai/api/v1/models` (476 models; live unauthenticated GET, status 200, response headers saved) | 2026-11-22 | OpenRouter spec (model catalog) |
| S-s3 | `https://openrouter.ai/docs/api_reference/overview.md` | 2026-11-22 | OpenRouter doc |
| S-s4 | `https://openrouter.ai/docs/api_reference/parameters.md` | 2026-11-22 | OpenRouter doc |
| S-s5 | `https://openrouter.ai/docs/api_reference/versioning.md` | 2026-11-22 | OpenRouter doc |
| S-s6 | `https://openrouter.ai/docs/api_reference/responses/overview.md` (mirror, plus a live re-fetch that matched) | 2026-11-22 | OpenRouter doc |
| S-s7 | `https://openrouter.ai/docs/api_reference/responses/error-handling.md` | 2026-11-22 | OpenRouter doc |
| S-s8 | `https://openrouter.ai/docs/api_reference/responses/basic-usage.md` | 2026-11-22 | OpenRouter doc |
| S-s9 | `https://openrouter.ai/docs/api_reference/errors-and-debugging.md` | 2026-11-22 | OpenRouter doc |
| S-s10 | `https://openrouter.ai/docs/api_reference/streaming.md` | 2026-11-22 | OpenRouter doc |
| S-s11 | `https://openrouter.ai/docs/api_reference/authentication.md` | 2026-11-22 | OpenRouter doc |
| S-s12 | `https://openrouter.ai/docs/api_reference/limits.md` | 2026-11-22 | OpenRouter doc |
| S-s13 | `https://openrouter.ai/docs/guides/features/compatibility.md` | 2026-11-22 | OpenRouter doc |
| S-s14 | `https://openrouter.ai/docs/guides/routing/provider-selection.md` | 2026-11-22 | OpenRouter doc |
| S-s15 | `https://openrouter.ai/docs/faq.md` | 2026-11-22 | OpenRouter doc |
| S-s16 | `https://openrouter.ai/docs/app-attribution.md` | 2026-11-22 | OpenRouter doc |
| S-s17 | `https://openrouter.ai/docs/guides/routing/routers/router-metadata.md` | 2026-11-22 | OpenRouter doc |
| S-s18 | `https://openrouter.ai/docs/guides/best-practices/reasoning-tokens.md` | 2026-11-22 | OpenRouter doc |
| S-s19 | `https://openrouter.ai/docs/cookbook/coding-agents/claude-code-integration.md` | 2026-11-22 | OpenRouter doc |
| S-s20 | `https://openrouter.ai/docs/guides/community/anthropic-agent-sdk.md` | 2026-11-22 | OpenRouter doc |
| S-s21 | `https://openrouter.ai/docs/cookbook/coding-agents/codex-cli.md` | 2026-11-22 | OpenRouter doc |
| S-s22 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/fable-5-1.md` | 2026-11-22 | OpenRouter doc |
| S-s23 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/opus-5.md` | 2026-11-22 | OpenRouter doc |
| S-s24 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5.md` | 2026-11-22 | OpenRouter doc |
| S-s25 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-6.md` | 2026-11-22 | OpenRouter doc |
| S-s26 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gemini-4.md` | 2026-11-22 | OpenRouter doc |
| S-s27 | `https://openrouter.ai/docs/guides/features/service-tiers.md` | 2026-11-22 | OpenRouter doc |
| S-s28 | `https://openrouter.ai/docs/guides/overview/multimodal/pdfs.md` | 2026-11-22 | OpenRouter doc |
| S-s29 | `https://openrouter.ai/docs/guides/features/presets.md` | 2026-11-22 | OpenRouter doc |
| S-s30 | `https://openrouter.ai/docs/guides/features/message-transforms.md` | 2026-11-22 | OpenRouter doc |
| S-s31 | `https://openrouter.ai/docs/cookbook/administration/usage-accounting.md` | 2026-11-22 | OpenRouter doc |
| S-s32 | `https://openrouter.ai/docs/guides/routing/model-fallbacks.md` | 2026-11-22 | OpenRouter doc |
| S-s33 | `https://openrouter.ai/docs/guides/features/server-tools/bash.md` | 2026-11-22 | OpenRouter doc |
| S-s34 | `https://openrouter.ai/docs/guides/features/server-tools/shell.md` | 2026-11-22 | OpenRouter doc |
| S-s35 | `https://openrouter.ai/docs/guides/features/server-tools/subagent.md` | 2026-11-22 | OpenRouter doc |
| S-s36 | `https://openrouter.ai/docs/guides/features/server-tools/tool-search.md` | 2026-11-22 | OpenRouter doc |
| S-s37 | `https://openrouter.ai/docs/guides/features/server-tools/advisor.md` | 2026-11-22 | OpenRouter doc |
| S-s38 | `https://openrouter.ai/docs/guides/features/broadcast.md` | 2026-11-22 | OpenRouter doc |
| S-s39 | `https://openrouter.ai/docs/changelog.md` (105 dated entries, July 3, 2026 to November 21, 2026) | 2026-11-22 | OpenRouter doc (changelog) |
| S-s40 | `https://openrouter.ai/docs/client-sdks/overview.md` | 2026-11-22 | OpenRouter doc |
| S-s41 | `https://openrouter.ai/docs/agent-sdk/overview.md` | 2026-11-22 | OpenRouter doc |
| S-s42 | `https://openrouter.ai/docs/agent-sdk/call-model/message-formats.md` | 2026-11-22 | OpenRouter doc |
| S-s43 | `https://openrouter.ai/docs/agent-sdk/call-model/items.md` | 2026-11-22 | OpenRouter doc |
| S-s44 | `https://github.com/OpenRouterTeam/typescript-sdk` at `7ad8458755a042a1ccf1fc96046dee3dd9055849` (2026-11-21): `src/models/chatassistantmessage.ts`, `reasoningdetailunion.ts`, `reasoningdetailtext.ts`, `reasoningformat.ts`, `chatresult.ts`, `chatchoice.ts`, `chatstreamchunk.ts`, `batchobject.ts`, `generationresponse.ts`; `src/funcs/chatSend.ts`; `src/sdk/` | 2026-11-22 | OpenRouter SDK source |
| S-s45 | `https://github.com/OpenRouterTeam/python-sdk` at `b48773eed16735206dae1d6b2af4e96182819568` (2026-11-21): `src/openrouter/components/chatassistantmessage.py` | 2026-11-22 | OpenRouter SDK source |
| S-s46 | `https://github.com/OpenRouterTeam/go-sdk` at `654d9d4d4b182a6daf12786495194d3c235c4616` (2026-11-21): `models/components/chatassistantmessage.go`, `chatchoice.go`, `chatresult.go`, `reasoningdetail*.go` | 2026-11-22 | OpenRouter SDK source |
| S-s47 | `https://github.com/OpenRouterTeam/typescript-agent` at `8a2d96e541cb49ecc1869d490ce54b4bcf3b5a55` (2026-11-17): `packages/agent/src/lib/chat-compat.ts`, `anthropic-compat.ts`, `stream-transformers.ts`; `packages/agent/src/api-shape-helpers/claude-message.ts` | 2026-11-22 | OpenRouter SDK source |
| S-s48 | `https://github.com/openai/openai-python` at `f63f14b34754fa77fc839ac2cf95da3ef42e1f6a` (2026-11-20): `src/openai/types/chat/*.py`, `src/openai/types/completion_usage.py`, `src/openai/types/shared/reasoning_effort.py`, `src/openai/types/shared/error_object.py`, `src/openai/types/responses/response_create_params.py` | 2026-11-22 | Vendor doc (OpenAI official SDK source) |
| S-s49 | `https://platform.openai.com/docs/api-reference/chat/create` (fetched as text, 187 KB) | 2026-11-22 | Vendor doc |
| S-s50 | `https://github.com/anthropics/anthropic-sdk-python` at `8baaef1c4d23b2e4861d05da0dca65252e879eb8` (2026-11-20, release 2.12.0): `src/anthropic/types/message_create_params.py`, `types/beta/message_create_params.py`, `types/output_config_param.py`, `types/beta/beta_output_config_param.py`, `types/thinking_config_*.py` | 2026-11-22 | Vendor doc (Anthropic official SDK source) |
| S-s51 | `https://github.com/BerriAI/litellm` at `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22): `litellm/llms/anthropic/messages/architecture.md`, `litellm/anthropic_beta_headers_manager.py`, `litellm/anthropic_beta_headers_config.json`, `litellm/llms/anthropic/messages/context_management/dispatcher.py` | 2026-11-22 | Gateway source |
| S-s52 | `https://openrouter.ai/docs/guides/features/files-api.md` | 2026-11-22 | OpenRouter doc |
| S-s53 | `https://openrouter.ai/docs/guides/features/response-caching.md` | 2026-11-22 | OpenRouter doc |
| S-s54 | `https://openrouter.ai/docs/guides/features/batch-api.md` | 2026-11-22 | OpenRouter doc |
| S-s55 | `https://openrouter.ai/docs/batch-quickstart.md` | 2026-11-22 | OpenRouter doc |
| S-s56 | `https://openrouter.ai/docs/guides/features/plugins/web-search.md` | 2026-11-22 | OpenRouter doc |
| S-s57 | `https://openrouter.ai/docs/guides/features/server-tools/web-search.md` | 2026-11-22 | OpenRouter doc |
| S-s58 | `https://openrouter.ai/docs/guides/overview/auth/byok.md` | 2026-11-22 | OpenRouter doc |
| S-s59 | `https://openrouter.ai/docs/guides/overview/multimodal/videos.md` | 2026-11-22 | OpenRouter doc |
| S-s60 | `https://openrouter.ai/docs/api_reference/responses/web-search.md` | 2026-11-22 | OpenRouter doc |
| S-s61 | `https://openrouter.ai/docs/guides/community/extractions.md` | 2026-11-22 | OpenRouter doc |
| S-s62 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28) | 2026-11-22 | Workspace document |
| S-s63 | `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/openrouter.rs` (comment dated 2026-09-14) | 2026-11-22 | Workspace document |
| S-s64 | Report on the Anthropic Messages API and harness survey, cabinet output, 2026-11-17 | 2026-11-22 | Workspace document |
| S-s65 | `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30) | 2026-11-22 | Workspace document |
| S-s66 | `https://openrouter.ai/docs/guides/features/server-tools/fusion.md` | 2026-11-22 | OpenRouter doc |

