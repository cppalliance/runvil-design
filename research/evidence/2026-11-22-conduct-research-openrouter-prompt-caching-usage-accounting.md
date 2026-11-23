---
produced: 2026-11-22
title: OpenRouter prompt caching and usage accounting - cache_control, prompt_cache_breakpoint, sticky routing, usage fields, generation stats, cache pricing, per-vendor native rules
---

# OpenRouter prompt caching and usage accounting - cache_control, prompt_cache_breakpoint, sticky routing, usage fields, generation stats, cache pricing, per-vendor native rules

- What was read: OpenRouter's mirrored documentation (33 pages), its OpenAPI spec and model catalog queried with Python, its official Vercel AI SDK provider source, live documentation from nine vendors (Anthropic, OpenAI, Google, DeepSeek, xAI, Groq, Moonshot, Alibaba, Z.AI), two open-source gateways at pinned commits, and three workspace documents.
- Sources: 51 source keys (32 OpenRouter doc keys, 1 spec, 1 catalog, 1 OpenRouter SDK repository, 11 vendor doc keys, 2 gateway repositories, 3 workspace documents). Evidence items: 85. Candidate matrix rows: 19.
- Read date: 2026-11-22 for everything. Three key OpenRouter pages (prompt caching, usage accounting, streaming) were re-fetched live and matched the mirror byte for byte.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Explicit per-block cache breakpoints | Anthropic (and Claude on Bedrock, Vertex, Azure) | `cache_control: {"type":"ephemeral","ttl":"5m"\|"1h"}` on blocks in `tools`, `system`, `messages`. At most 4 breakpoints. 20-position lookback per breakpoint. Per-model minimum of 512 to 4,096 tokens, silently uncached below it. Thinking blocks cannot take the marker. | Chat Completions: `cache_control` on `text` content parts and on function tools. `/messages`: Anthropic's own positions (text, image, document, `tool_use`, `tool_result`, tools, system blocks). `/responses`: per-block `cache_control` is replaced by `prompt_cache_breakpoint`. | On the chat route the spec lists the marker on text parts and function tools only. The Responses route drops the per-block TTL. The 4-breakpoint limit and the lookback stay the caller's problem. | The vendor holds the cache, keyed by the exact prefix. Nothing makes a round trip. The marker is a neutral value. Placement can be done by the caller or by the gateway. Works on the chat route. | OR-documented, Vendor-documented | C6, C7, C8, C17, C62, C80 |
| Automatic caching (one top-level field) | Anthropic | One top-level `cache_control`. The API puts the breakpoint on the last cacheable block and moves it forward each turn. Uses one of the 4 slots. Bedrock `bedrock-runtime` answers 400 to the top-level field. | Top-level `cache_control` on `/chat/completions`, `/responses`, and `/messages`. On Bedrock OpenRouter rewrites it into a trailing breakpoint. | Listed as supported on Anthropic, Vertex, Azure, Bedrock, and Claude Platform on AWS. The page does not say what happens on other vendors (the general rule is that an unsupported parameter is ignored). | A gateway can add this field with zero input from the caller. One field, nothing to count. Chat route. | OR-documented, Vendor-documented | C6, C7, C23, C62, C82 |
| Cache lifetime (TTL) | Anthropic `5m`, `1h`. OpenAI GPT-5.6+ `30m`, `12h`. Gemini explicit caches any `ttl` (default 1 hour). Moonshot `5m`, `1h`. Alibaba fixed 5 minutes. | Each vendor has its own value set and its own write price per TTL. | `cache_control.ttl` (enum `5m`, `1h`), `prompt_cache_breakpoint.ttl` and `prompt_cache_options.ttl` (`"30m"`, `"12h"`). Gemini writes get a fixed 5-minute TTL. | TTLs are dropped when a marker crosses vendor families. They are not converted. Gemini's TTL is fixed by OpenRouter. | TTL values are vendor-specific. A neutral short or long setting would need a map inside the gateway. | OR-documented, Vendor-documented | C3, C6, C11, C19, C22, C62, C64, C65, C69, C70 |
| Automatic prefix caching (no marker) | OpenAI, DeepSeek, xAI Grok, Groq, Moonshot, Z.AI, Gemini 2.5+ implicit | The vendor caches a repeated prefix by itself. OpenAI GPT-5.6+ bills the write at 1.25x. Minimums differ per vendor and model. | Nothing to send. Reads show as `cached_tokens`, writes as `cache_write_tokens`. | OpenRouter's per-vendor multipliers and minimums are stale in places against the vendor pages and against its own catalog. | The vendor holds the state. The caller only has to keep the prefix byte-stable. Gateway-only. | OR-documented, Vendor-documented | C1, C2, C10, C12, C58, C64 to C71 |
| OpenAI explicit breakpoints and cache options | OpenAI GPT-5.6 and later | `prompt_cache_breakpoint: {"mode":"explicit","ttl"?}` on a content block. `prompt_cache_options: {"mode":"explicit"\|"implicit","ttl"}` at the root. Up to four cache writes per request. | Same two names on Chat Completions (`text` parts) and Responses (`input_text` parts). Stripped before forwarding to providers that do not support them. | The spec's `mode` enum lists `explicit` only. `prompt_cache_options` is never translated. The per-TTL write split `cache_write_tokens_details` is not in OpenRouter's usage schema. | Vendor holds state. Values `30m` and `12h` are vendor-specific. The block marker is interchangeable with `cache_control`. Chat route. | OR-documented, Vendor-documented | C3, C4, C21, C64 |
| Cache routing key | OpenAI `prompt_cache_key`. xAI `x-grok-conv-id` header (chat) or `prompt_cache_key` (responses). Z.AI session affinity. | A caller-chosen stable string that steers requests to the machine holding the cache. | `prompt_cache_key` is accepted on chat and responses and doubles as the sticky routing key when no `session_id` is sent. Z.AI gets an OpenRouter-derived affinity key. Private endpoints get an OpenRouter-derived `prompt_cache_key`, not the caller's. | Whether the caller's `prompt_cache_key` reaches OpenAI, and whether `x-grok-conv-id` is set for xAI, is not written down. | One stable conversation id from the caller is the only input. A gateway can derive it. | OR-documented, OR-inferred | C5, C12, C16, C27, C64, C67 |
| `prompt_cache_retention` | OpenAI models before GPT-5.6 | `in_memory` or `24h`. | The field is in no request schema of the spec. | An unknown parameter is ignored, so the vendor default applies. | Vendor-specific value. Not needed for a neutral API. | OR-inferred (spec lists no such field) | C5, C23, C64, C79 |
| Gemini explicit caching (named cache objects) | Google Gemini 2.5 and 3 on `generateContent` | The caller creates a `cachedContents` resource, pays storage by TTL (default 1 hour), and sends its `name` as `cachedContent`. System instruction, tools, and tool config are immutable in the cache. Not offered on Gemini 4 or the Interactions API. | A `cache_control` breakpoint in message content. OpenRouter creates and reuses the cache object. Only the last breakpoint counts. Writes get a 5-minute TTL that does not refresh. | Cache name, TTL choice, update, and delete are hidden. A dynamic tail inside the first system message cannot stay uncached. Skipped on Gemini 4 continuation hops. | The gateway holds the state (cache name per prefix hash). The caller sends the same neutral marker as for Anthropic. Chat route. | OR-documented, Filled (LiteLLM) | C11, C15, C65, C72 |
| Alibaba explicit caching | Alibaba Qwen | `cache_control: {"type":"ephemeral"}` in messages, up to four markers, 20-block lookback, 5-minute validity, 1,024-token minimum. | Same `cache_control` syntax, on a listed set of models. | TTL is fixed at 5 minutes. | Same neutral marker. Vendor holds state. | OR-documented, Vendor-documented | C13, C70 |
| Marker translation between conventions | OpenAI, Anthropic, Google | Two block-marker spellings exist: `cache_control` and `prompt_cache_breakpoint`. | A text block with `cache_control` gets a `prompt_cache_breakpoint` on a supporting OpenAI model. A block with `prompt_cache_breakpoint` gets a default 5-minute `cache_control` toward Anthropic or Google. | TTL in both directions. Request-level `prompt_cache_options`. | One block marker is enough for the caller. The gateway picks the spelling from the serving vendor. | OR-documented | C19, C20, C21, C22 |
| Sticky provider routing | OpenRouter (many providers per model) | Vendor caches are per provider and often per machine. | Automatic after a cache hit. Key is account + model + conversation (hash of the first system or developer message and the first non-system message), or `session_id`, `x-session-id`, or `prompt_cache_key`. Expires after 60 minutes idle (another page says a 10-minute pin). | Switched off by `provider.order`. Overridden by Auto Exacto on tool-calling requests. A changed model slug changes the key. | The gateway holds routing state. The caller may send a session id but need not. | OR-documented | C25 to C32, C83 |
| Cached and written token counts | All | Anthropic: `input_tokens` excludes cache, plus `cache_read_input_tokens`, `cache_creation_input_tokens`, `cache_creation.ephemeral_5m_input_tokens` and `ephemeral_1h_input_tokens`. OpenAI: `cached_tokens`, `cache_write_tokens`, `cache_write_tokens_details`. DeepSeek: `prompt_cache_hit_tokens`, `prompt_cache_miss_tokens`. Alibaba: `cache_creation_input_tokens`. | Chat: `prompt_tokens` is the whole prompt, with `prompt_tokens_details.cached_tokens` and `.cache_write_tokens`. Responses: `input_tokens_details.cached_tokens` and `.cache_write_tokens`. `/messages`: Anthropic's shape. | The per-TTL write split shows on `/messages` and in Broadcast only. The vendor's raw usage object shows in Broadcast only. | The gateway normalizes. `cache_write_tokens` sits inside the standard `prompt_tokens_details` object, so a client that ignores it still works. | OR-documented, Filled (LiteLLM, agentgateway) for the sum rule | C34, C36, C37, C41, C42, C47, C63, C73, C74, C78, C84 |
| Usage delivery and the final stream chunk | OpenAI | Usage in a stream only with `stream_options.include_usage`, in a last chunk with empty `choices`. | Usage is always sent. `usage: {include: true}` and `stream_options.include_usage` are deprecated and do nothing. The last chunk has one choice with a content-free delta that repeats `finish_reason`. | The OpenAI shape of the usage chunk. | A client parser has to accept a usage chunk with or without choices. | OR-documented | C33, C38, C39 |
| Cost in the usage object | No vendor reports cost | Vendors report tokens only. | `usage.cost`, `usage.cost_details.upstream_inference_cost` with a prompt and completions split (input and output on Responses), `usage.is_byok`, `cost_details.server_tool_cost`. | Nothing lost. This is an addition. | Gateway-only. Needs a price table with cache read and write rates per model. | OR-documented | C34, C36, C43 |
| Reasoning tokens in usage and price | OpenAI, Anthropic, Google | Each vendor counts thinking tokens under its own name. | `completion_tokens_details.reasoning_tokens` (chat), `output_tokens_details.reasoning_tokens` (responses), `output_tokens_details.thinking_tokens` (`/messages`). Counted inside completion tokens and billed as output. Catalog key `internal_reasoning` exists on Google models and equals the completion price. | Nothing found. | Gateway-only. | OR-documented | C34, C37, C41, C49, C59 |
| Generation stats API | OpenRouter | No vendor equivalent. | `GET /generation?id=` returns native and normalized token counts, `cache_discount`, `upstream_inference_cost`, provider name, finish reasons, latency fields, and the list of provider attempts. | Cache writes have no field of their own there (`native_tokens_cached` is the only cache count). `upstream_inference_cost` is filled for BYOK only. | The gateway stores a record per generation id. | OR-documented | C50 to C55, C85 |
| Cache prices in the catalog | All | Per-vendor price lists. | `pricing.input_cache_read`, `input_cache_write`, `input_cache_write_1h`, `internal_reasoning`, `input_audio_cache`, and `overrides` for long-context tiers. | OpenAI's 12-hour write rate and Gemini's storage rate have no catalog key. | A gateway that reports cost needs these rates per model and per TTL. | OR-documented | C57 to C61 |
| Cache-safe mid-conversation changes | Anthropic, OpenAI | Anthropic: `tool_addition` and `tool_removal` blocks, `clear_at` system messages, per-turn effort. OpenAI GPT-6: `configuration_update` item. Each keeps the cached prefix unchanged. | Anthropic tool changes: on the `/messages` route only. Effort change: an item interchangeable across the three routes. | The Anthropic tool-change blocks are not offered on the chat route. | A feature that exists only on a non-chat route. | OR-documented | C56 |
| Response cache (OpenRouter's own, not prompt caching) | OpenRouter | Not a vendor feature. | `X-OpenRouter-Cache` request headers, `X-OpenRouter-Cache-Status` response header, zeroed usage on a hit. | Not applicable. | A separate gateway feature. Do not confuse it with vendor prompt caching. | OR-documented | C45 |

## Evidence

Source keys (C-s1 and so on) are listed in the Sources table at the end. Every item was read on 2026-11-22. Quotes are word for word, with one typographic change: a long dash in the source is written here as a single hyphen with spaces, because this workspace's files use plain dashes only. Where a quote was taken from a hard-wrapped source or from HTML with the tags removed, the item says so.

### A. How caching is requested through OpenRouter

**C1. Most providers cache by themselves. Alibaba and Anthropic need a marker per message.**
- Quote: "Most providers automatically enable prompt caching, but note that some (see Alibaba and Anthropic below) require you to enable it on a per-message basis."
- Quote (who holds the cached data, from the ZDR page): "Some endpoints/models provide implicit caching of prompts. This keeps repeated prompt data in an in-memory cache in the provider's datacenter, so that the repeated part of the prompt does not need to be re-processed." and "OpenRouter has taken the stance that in-memory caching of prompts is *not* considered "retaining" data, and we therefore allow endpoints/models with implicit caching to be hit when a ZDR routing policy is in effect."
- Sources: C-s1, C-s23. Basis: OR-documented.

**C2. OpenAI caching is automatic, with a 1024-token minimum, and GPT-5.6 and later charge for the write.**
- Quote: "Prompt caching with OpenAI is automated and does not require any additional configuration. There is a minimum prompt size of 1024 tokens."
- Quote: "**Cache writes**: no cost on models before the GPT-5.6 family. GPT-5.6 and later charge cache writes at 1.25x the price of the original input pricing, even with automatic caching - no opt-in required."
- Quote: "**Cache reads**: (depending on the model) charged at 0.25x or 0.50x the price of the original input pricing"
- Source: C-s1. Basis: OR-documented. Note: the read multipliers disagree with the catalog and with OpenAI's page, see C58 and Conflicts.

**C3. OpenAI explicit caching uses two fields, spelled the same as OpenAI spells them, on both the chat and responses routes.**
- Quote: "`prompt_cache_breakpoint`: placed on an individual text content block (`input_text` in Responses, `text` in Chat Completions) to mark the end of a reusable prefix. Everything through that block becomes the candidate cached prefix. Automatic caching remains enabled."
- Quote: "`prompt_cache_options`: placed at the request root. Setting `mode` to `"explicit"` disables OpenAI-managed breakpoints so only blocks marked with `prompt_cache_breakpoint` participate in caching. Use `ttl` to set the minimum cache lifetime for every breakpoint the request writes: `"30m"` (default) or `"12h"`. A `prompt_cache_breakpoint` can carry its own `ttl` to override the request-level value for that boundary".
- Quote: "OpenAI explicit prompt caching is only supported by OpenAI GPT-5.6 and newer."
- Quote (prices): "**Cache writes (30-minute TTL, default)**: charged at 1.25x", "**Cache writes (12-hour TTL)**: charged at 2.5x".
- Spec: schema `PromptCacheBreakpoint` has `mode` (enum `["explicit"]`, required) and `ttl` (string or null). Schema `PromptCacheOptions` has the same two properties. `prompt_cache_breakpoint` appears on exactly two schemas: `ChatContentText` and `InputText`. `prompt_cache_options` appears on `ChatRequest`, `ResponsesRequest`, and the Responses result object.
- Sources: C-s1, C-s2, C-s30. Basis: OR-documented.

**C4. Caching is turned off for one request by asking for explicit mode and marking nothing.**
- Quote: "To turn caching off for a request, set `prompt_cache_options.mode` to `"explicit"` and do not mark any content block with `prompt_cache_breakpoint`."
- Quote: "When caching is disabled, `cache_write_tokens` in the response usage reads `0`."
- Source: C-s1. Basis: OR-documented.

**C5. `prompt_cache_key` is accepted on chat and responses. `prompt_cache_retention` is in no schema.**
- Spec: `ChatRequest.properties.prompt_cache_key` and `ResponsesRequest.properties.prompt_cache_key` are `{"type": ["string", "null"]}` with no description. `MessagesRequest` lists `cache_control` and `session_id` but not `prompt_cache_key`.
- Spec: a search of the whole spec for the key `prompt_cache_retention` returns zero occurrences.
- Quote (the one documented use): "If neither is set, OpenRouter falls back to the OpenAI-style `prompt_cache_key` request field as the sticky routing key. Clients that already send `prompt_cache_key` get session-pinned routing without any changes."
- Sources: C-s30, C-s1. Basis: OR-documented. The inference drawn from the missing field is item C79.

**C6. Anthropic caching has two forms through OpenRouter: one top-level field, or per-block breakpoints.**
- Quote: "**Automatic caching**: Add a single `cache_control` field at the top level of your request. The system automatically applies the cache breakpoint to the last cacheable block and advances it forward as conversations grow. Best for multi-turn conversations."
- Quote: "**Explicit cache breakpoints**: Place `cache_control` directly on individual content blocks for fine-grained control over exactly what gets cached. There is a limit of four explicit breakpoints."
- Quote: "**Automatic caching** (top-level `cache_control`) is supported on the **Anthropic**, **Google Vertex AI**, **Azure**, and **Amazon Bedrock** providers, as well as Claude Platform on AWS. On Amazon Bedrock, OpenRouter translates the top-level field into a trailing cache breakpoint ([Bedrock's simplified cache management](https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html#prompt-caching-simplified)), since Bedrock's InvokeModel API does not accept the top-level field directly. Explicit per-block `cache_control` breakpoints work across all Anthropic-compatible providers including Bedrock and Vertex."
- Quote (TTL): "By default, the cache expires after 5 minutes, but you can extend this to 1 hour by specifying `"ttl": "1h"` in the `cache_control` object." and "**5 minutes** (default): `"cache_control": { "type": "ephemeral" }`", "**1 hour**: `"cache_control": { "type": "ephemeral", "ttl": "1h" }`".
- Quote (prices): "**Cache writes (5-minute TTL)**: charged at 1.25x", "**Cache writes (1-hour TTL)**: charged at 2x", "**Cache reads**: charged at 0.1x".
- Source: C-s1. Basis: OR-documented.

**C7. Where the spec allows `cache_control`, per route.**
- Shared schema: `AnthropicCacheControlDirective` = `{type: enum ["ephemeral"] (required), ttl: AnthropicCacheControlTtl}`. `AnthropicCacheControlTtl` = enum `["5m", "1h"]` with `x-speakeasy-unknown-values: allow`.
- Chat Completions: top-level `ChatRequest.cache_control`; `ChatContentText.cache_control` (text content parts); `ChatFunctionTool.cache_control` (function tools). The other chat content part schemas (`ChatContentImage`, `ChatContentFile`, `ChatContentAudio`, `ChatContentVideo`) list only their own payload and `type`. Message objects (`ChatSystemMessage`, `ChatUserMessage`, `ChatAssistantMessage`, `ChatToolMessage`) list no message-level `cache_control`. System and developer content may be an array of `ChatContentText`, and tool message content may be a string or an array of content parts, so a marked text part can sit in a system, user, assistant, or tool message.
- Responses: top-level `ResponsesRequest.cache_control` only. `InputText` lists `prompt_cache_breakpoint`, `text`, `type`.
- Messages: top-level `MessagesRequest.cache_control`; `system` as an array of `AnthropicTextBlockParam`; message content blocks of type `text`, `image`, `document`, `tool_use`, `tool_result`, `server_tool_use`, `web_search_tool_result`, `search_result`, `compaction`, `tool_addition`, `tool_removal`; tools of type `custom`, `bash_20250124`, `text_editor_20250124`, `web_search_20250305`, `web_search_20260209`, `advisor_20260301` (which also has a `caching` property of the same type), and the two tool-search tools. Blocks of type `thinking` and `redacted_thinking` list no `cache_control`.
- Source: C-s30 (queried with Python: 32 occurrences of the key `cache_control` outside examples). Basis: OR-documented.

**C8. On the Responses route the Anthropic per-block marker is not offered. The OpenAI marker stands in for it.**
- Quote: "**Responses API support:** The [Responses API](https://openrouter.ai/docs/api/api-reference/responses/create-a-response.md) supports **automatic caching** via top-level `cache_control`. Anthropic-style per-block `cache_control` inside `input` items is **not** exposed through the Responses API - instead use OpenAI's per-block [`prompt_cache_breakpoint`](#explicit-prompt-caching), which OpenRouter converts to a default `cache_control` breakpoint when the request is routed to Anthropic or Google. Note that `prompt_cache_breakpoint` carries no `ttl`; if you need to set a cache `ttl`, use the [Chat Completions](https://openrouter.ai/docs/api/api-reference/chat/create-a-chat-completion.md) or [Anthropic Messages](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md) API with `cache_control`."
- Source: C-s1. Basis: OR-documented. See C22 for the part of this note that is out of date.

**C9. OpenRouter's list of Anthropic minimum cacheable lengths.**
- Quote: "**512 tokens**: Claude Fable 5.1, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Fable 5, Claude Haiku 5.5", "**4,096 tokens**: Claude Opus 4.6, Claude Opus 4.5, Claude Haiku 4.5", "**2,048 tokens**: Claude Opus 4.7, Claude Haiku 3.5", "**1,024 tokens**: Claude Opus 4.8, Claude Sonnet 5, Claude Sonnet 4.6, Claude Sonnet 4.5, Claude Opus 4.1, Claude Opus 4, Claude Sonnet 4", "Prompts shorter than these minimums will not be cached."
- Source: C-s1. Basis: OR-documented. It matches Anthropic's live list (C62) for the models both name. Anthropic's list also names Fable 5.2 and the Mythos models at 512.

**C10. Gemini implicit caching needs nothing in the request.**
- Quote: "Gemini 2.5 series models and newer support **implicit caching**, providing automatic caching functionality similar to OpenAI’s automatic caching. Implicit caching works seamlessly - no manual setup or additional `cache_control` breakpoints required."
- Quote: "No cache write or storage costs." and "Cached tokens are charged at 0.25x the original input token cost."
- Quote: "Note that the TTL is on average 3-5 minutes, but will vary. Requests must also meet a minimum prompt size to be eligible for caching, which varies by model: 1024 tokens for Gemini 2.5 Flash and 4096 tokens for Gemini 2.5 Pro."
- Source: C-s1. Basis: OR-documented. The minimums disagree with Google's live table (C65).

**C11. Gemini explicit caching is requested with the Anthropic marker, and OpenRouter runs the cache objects.**
- Quote: "OpenRouter simplifies Gemini cache management, abstracting away complexities:" then "You **do not** need to manually create, update, or delete caches." and "You **do not** need to manage cache names or TTL explicitly."
- Quote: "Gemini caching in OpenRouter requires you to insert `cache_control` breakpoints explicitly within message content, similar to Anthropic."
- Quote: "There is not a limit on the number of `cache_control` breakpoints you can include in your request. OpenRouter will use only the last breakpoint for Gemini caching across normal message content. Including multiple breakpoints is safe and can help maintain compatibility with Anthropic, but only the final one will be used for Gemini."
- Quote: "Cache Writes have a 5 minute Time-to-Live (TTL) that does not update. After 5 minutes, the cache expires and a new cache must be written."
- Quote (price): "Cache write cost = Input token price + (Cache storage price × (5 minutes / 60 minutes))" and "**Cache Reads:** Charged at 0.25× the original input token cost."
- Quote (system prompt limit): "Gemini has a single `systemInstruction` field, and cached Gemini content treats that `systemInstruction` as immutable. On OpenRouter, this means `cache_control` inside the first `system` or `developer` message can cache the normalized system prompt, but it cannot preserve an uncached dynamic tail inside that same message. If you need part of your prompt to stay dynamic, move that dynamic content into a later `user` message instead of appending it after a cached block in the first `system` message."
- Quote: "Gemini models have typically have a 4096 token minimum for cache write to occur. Cached tokens count towards the model's maximum token usage."
- Source: C-s1. Basis: OR-documented. How the cache object is found again on the next request is not described (see Open questions, with a LiteLLM fill at C72).

**C12. DeepSeek, Grok, Moonshot, Groq, and Z.AI cache automatically.**
- DeepSeek: "**Cache writes**: charged at the same price as the original input pricing", "**Cache reads**: charged at 0.1x", "Prompt caching with DeepSeek is automated and does not require any additional configuration."
- Grok: "**Cache writes**: no cost", "**Cache reads**: charged at 0.25x", "Prompt caching with Grok is automated and does not require any additional configuration."
- Moonshot AI: "**Cache writes**: no cost", "**Cache reads**: charged at 0.25x", "Prompt caching with Moonshot AI is automated and does not require any additional configuration."
- Groq: "**Cache writes**: no cost", "**Cache reads**: charged at 0.5x", "Prompt caching with Groq is automated and does not require any additional configuration. Currently available on Kimi K2 models."
- Z.AI: "Prompt caching with Z.AI is automated and does not require any additional configuration. Cache reads are reported in the `cached_tokens` field of `prompt_tokens_details` in the usage response." and "To improve cache hit rates, OpenRouter sends Z.AI a session affinity key with each request, derived from your account and, when provided, your [`session_id`](#using-session_id-for-sticky-sessions). Passing a `session_id` in multi-turn conversations keeps requests from the same session on the same cache."
- Source: C-s1. Basis: OR-documented. The Moonshot and Groq statements are out of date against the vendors' pages (C68, C69).

**C13. Alibaba Qwen needs explicit breakpoints, written exactly like Anthropic's.**
- Quote: "Alibaba prompt caching requires explicit cache breakpoints. Add `cache_control: { "type": "ephemeral" }` to content blocks you want to cache, using the same syntax as Anthropic explicit caching. Cache writes use a 5-minute TTL."
- Quote: "Alibaba explicit caching is available on `deepseek/deepseek-v3.2`, `qwen/qwen3-max`, `qwen/qwen-plus`, `qwen/qwen3.6-plus`, `qwen/qwen3-coder-plus`, and `qwen/qwen3-coder-flash`. Snapshot endpoints, including `qwen/qwen3.5-plus-02-15` and `qwen/qwen3.5-flash-02-23`, do not support explicit caching."
- Quote (prices): "**Cache writes**: charged at 1.25x", "**Cache reads**: charged at 0.1x".
- Source: C-s1. Basis: OR-documented.

**C14. In the Batch API a cache written by one line may be invisible to other lines of the same batch.**
- Quote: "`cache_control` breakpoints work on Anthropic `:batch` endpoints the same way as on the sync API, but the requests inside a single batch may process concurrently and in any order - a cache written by one line is not guaranteed to be visible to other lines in the same batch."
- Source: C-s1. Basis: OR-documented.

**C15. On Gemini 4 long answers, the explicit marker is skipped on continuation hops.**
- Quote: "Implicit caching can cover part of that prompt, and cached tokens bill at the cached input rate. Explicit caching with `cache_control` is skipped on continuation hops."
- Source: C-s21. Basis: OR-documented. Google's own page says explicit caching is not offered for Gemini 4 at all (C65). OpenRouter's prompt caching page does not say what a `cache_control` marker does on a Gemini 4 model.

**C16. For private endpoints OpenRouter sends its own derived cache key, not the caller's.**
- Spec, `PrivateEndpointPromptCaching.cache_control`: "Forward top-level, message, and tool cache_control markers. Defaults on after endpoint validation accepts these fields; otherwise off. Set false to opt out, or true to declare support explicitly."
- Spec, `PrivateEndpointPromptCaching.prompt_cache_key`: "Send an OpenRouter-derived, customer-isolated prompt_cache_key, not the caller-supplied key. Defaults on after endpoint validation accepts the field; otherwise off. Set false to opt out, or true to declare support explicitly. HIPAA and unknown workspace posture withhold OpenRouter caching directives and the derived key; this does not control upstream retention."
- Source: C-s30. Basis: OR-documented. This is the only place where the spec says what `prompt_cache_key` value goes upstream. The words "top-level, message, and tool cache_control markers" name three positions.

**C17. OpenRouter's official AI SDK provider sends `cache_control` in more places than the spec lists.**
- Type: `export type OpenRouterCacheControl = { type: 'ephemeral'; ttl?: '5m' | '1h' };` (`src/chat/convert-to-openrouter-chat-messages.ts` line 31).
- User message parts: the marker is spread onto `text`, `image_url`, `video_url`, `input_audio`, and `file` parts, for example `type: 'image_url' as const, image_url: { url, ...(detail && { detail }) }, ...(cacheControl && { cache_control: cacheControl })` (same file, lines 138 to 221). A message-level marker goes to the last text part only (lines 114 to 136), with the comment "For multi-part messages, don't add cache_control at the root level" (line 234).
- Assistant message: `cache_control: getCacheControl(providerOptions)` at message level (line 416). Tool message: `cache_control: getCacheControl(providerOptions) ?? getCacheControl(toolResponse.providerOptions)` at message level (lines 435 to 437).
- Tools: `...(cacheControl != null && { cache_control: cacheControl })` on each function tool (`src/chat/index.ts` line 264). Top level: the comment "// Anthropic automatic caching:" then `cache_control: this.settings.cache_control` (lines 234 to 235).
- The chat path has no typed support for `prompt_cache_breakpoint`, `prompt_cache_options`, or `prompt_cache_key` (a search of `src` outside tests finds none).
- Source: C-s32 at commit `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33`, package version 3.1.0. Basis: OR-documented (official SDK source). What this implies about the server is item C80. See also Conflicts.

**C18. The catalog marks implicit caching per endpoint, and has no "supported parameter" for cache markers.**
- Spec: `PublicEndpoint.supports_implicit_caching` is a boolean. An endpoint listing quoted in the docs shows `"supports_implicit_caching":true` for an OpenAI endpoint.
- Catalog: the 26 distinct values of `supported_parameters` across 476 models hold no cache-related name. Cache support shows only through the pricing keys (C57).
- Sources: C-s30, C-s31, C-s51. Basis: OR-documented. The consequence for `provider.require_parameters` is item C81.

### B. Translation between conventions

**C19. The block markers are interchangeable, and TTLs are dropped, not converted (the current text behind LC7).**
- Quote, in full: "The block-level markers are interchangeable: a text block marked with Anthropic-style `cache_control` gets a `prompt_cache_breakpoint` when routed to a supporting OpenAI model, and a block marked with `prompt_cache_breakpoint` gets a default (5-minute) `cache_control` when routed to Anthropic or Google. TTLs are not translated - a `cache_control` `ttl` is dropped toward OpenAI, a `prompt_cache_breakpoint` `ttl` is dropped toward Anthropic or Google, and the request-level `prompt_cache_options` stays OpenAI-only."
- What is dropped: `cache_control.ttl` toward OpenAI (so a `1h` request becomes OpenAI's default 30 minutes). `prompt_cache_breakpoint.ttl` toward Anthropic and Google (so a `12h` request becomes 5 minutes). All of `prompt_cache_options` toward every vendor but OpenAI.
- Source: C-s1. Basis: OR-documented.

**C20. The spec says the same in its schema descriptions.**
- `ChatContentCacheControl`: "Anthropic-style cache breakpoint for the content part. Interchangeable with the OpenAI-style `prompt_cache_breakpoint` marker: OpenRouter converts between the two based on the provider serving the request."
- `PromptCacheBreakpoint`: "Marks an explicit prompt-cache boundary on this content block (OpenAI-style). Everything through the block carrying this marker is part of the candidate cached prefix. Supported natively by OpenAI GPT-5.6 and newer; on providers that use Anthropic-style `cache_control`, OpenRouter converts the marker to that format automatically. `ttl` ("30m" or "12h") overrides the request-level `prompt_cache_options.ttl` for this boundary; 12h writes bill at the higher cache-write rate."
- `AnthropicCacheControlDirective`: "Enable automatic prompt caching. When set at the top level, the system automatically applies cache breakpoints to the last cacheable block in the request. When set on an individual content block, it marks an explicit cache breakpoint; block-level markers also work on OpenAI models that support explicit prompt caching - OpenRouter converts them to the provider's native format."
- `PromptCacheOptions`: "Request-level prompt-cache controls. `mode: "explicit"` disables OpenAI-managed breakpoints so only blocks marked with `prompt_cache_breakpoint` are cached. `ttl` sets the minimum cache lifetime for every breakpoint the request writes: "30m" (default) or "12h" (billed at the higher 12h cache-write rate). Only supported by OpenAI GPT-5.6 and newer."
- Source: C-s30. Basis: OR-documented.

**C21. OpenAI-style cache fields are stripped before a request goes to a provider that does not support them.**
- Quote: "You don't need to filter these fields per model. OpenRouter strips `reasoning.mode`, `prompt_cache_breakpoint`, and `prompt_cache_options` before forwarding requests to providers that don't support them, so requests that fall back to other models keep working."
- Quote: "The block-level markers are interchangeable with the Anthropic-style `cache_control` blocks Chat Completions also accepts: `cache_control` becomes a `prompt_cache_breakpoint` on GPT-5.6+ models, and `prompt_cache_breakpoint` becomes a default 5-minute `cache_control` on Anthropic and Google. The request-level `prompt_cache_options` stays OpenAI-only."
- Source: C-s2. Basis: OR-documented.

**C22. The breakpoint-level `ttl` arrived on 2026-10-21, and one note on the page still describes the earlier state.**
- Changelog entry dated "October 21, 2026": "`PromptCacheBreakpoint`: description updated; example modified; new property `ttl`" and "`PromptCacheOptions`: description updated".
- The Responses note (C8) still says "Note that `prompt_cache_breakpoint` carries no `ttl`", while the explicit caching section of the same page says "A `prompt_cache_breakpoint` can carry its own `ttl`".
- Sources: C-s18, C-s1. Basis: OR-documented. The local survey (2026-10-28) was written one week after this change.

**C23. A parameter the chosen model or provider does not support is ignored.**
- Quote: "If the chosen model doesn't support a request parameter (such as `logit_bias` in non-OpenAI models, or `top_k` for OpenAI), then the parameter is ignored. The rest are forwarded to the underlying model API."
- Quote: "With the default routing strategy, providers that don't support all the [LLM parameters](https://openrouter.ai/docs/api_reference/parameters.md) specified in your request can still receive the request, but will ignore unknown parameters."
- Sources: C-s12, C-s6. Basis: OR-documented. What this means for `cache_control` on a model without cache support is item C82.

**C24. With deferred tools, `cache_control` on a tool is part of the fixed tool catalog.**
- Quote: "**Keep the tool catalog fixed.** Send the same `tools`, `tool_choice`, and `deferred_tools` settings on every turn. Tool order, descriptions, and `cache_control` are part of the catalog, so changing any of them requires a new conversation."
- Source: C-s25. Basis: OR-documented.

### C. Routing for cache hits (sticky provider routing)

**C25. After a cached request, later requests for the same model go to the same provider endpoint.**
- Quote: "To maximize cache hit rates, OpenRouter uses **provider sticky routing** to route your subsequent requests to the same provider endpoint after a cached request. This works automatically with both implicit caching (e.g. OpenAI, DeepSeek, Gemini 2.5) and explicit caching (e.g. Anthropic `cache_control` breakpoints)."
- Quote: "After a request that uses prompt caching, OpenRouter remembers which provider served your request."
- Quote: "Sticky routing only activates when the provider's cache read pricing is cheaper than regular prompt pricing, ensuring you always benefit from cost savings."
- Quote: "If the sticky provider becomes unavailable, OpenRouter automatically falls back to the next-best provider."
- Quote: "Sticky routing is not used when you specify a manual [provider order](https://openrouter.ai/docs/guides/routing/provider-selection.md) via `provider.order` - in that case, your explicit ordering takes priority."
- Quote: "Sticky sessions expire after **60 minutes** of inactivity. Each successful request resets the timer. If the sticky provider returns an error, the cache is not updated, allowing the next request to be re-routed."
- Source: C-s1. Basis: OR-documented.

**C26. The default key is account, model, and a hash of the opening messages.**
- Quote: "Sticky routing is tracked at the account level, per model, and per conversation. By default, OpenRouter identifies conversations by hashing the first system (or developer) message and the first non-system message in each request, so requests that share the same opening messages are routed to the same provider. This means different conversations naturally stick to different providers, improving load-balancing and throughput while keeping caches warm within each conversation."
- Source: C-s1. Basis: OR-documented.

**C27. `session_id` replaces the hash, and makes stickiness start one request earlier.**
- Quote: "When a `session_id` is present, OpenRouter uses it directly as the sticky routing key instead of deriving one from message hashing. This is especially useful for multi-turn agentic workflows where the opening messages may change between requests but you still want to route to the same provider."
- Quote: "**Request body**: Include `session_id` as a top-level field in your request body. If both are provided, the body value takes precedence." and "**Header**: Set the `x-session-id` HTTP header." and "The `session_id` must be at most 256 characters."
- Quote: "If neither is set, OpenRouter falls back to the OpenAI-style `prompt_cache_key` request field as the sticky routing key."
- Quote: "When `session_id` is set, sticky routing activates on any successful request - even before cache usage is observed - so that subsequent requests in the same session benefit from prompt caching from the start. Without `session_id`, sticky routing only activates after a cache hit is detected."
- Spec, `ChatRequest.session_id` (same text on `MessagesRequest` and `ResponsesRequest`): "A unique identifier for grouping related requests (e.g., a conversation or agent workflow). When provided, OpenRouter uses it as the sticky routing key, routing all requests in the session to the same provider to maximize prompt cache hits. Also used for observability grouping. If provided in both the request body and the x-session-id header, the body value takes precedence. Maximum of 256 characters." with `maxLength: 256`.
- Sources: C-s1, C-s30. Basis: OR-documented.

**C28. With router models the resolved model is reused too, on a best-effort basis.**
- Quote: "When using router models like [Auto Router](https://openrouter.ai/docs/guides/routing/routers/auto-router.md) or [Pareto Router](https://openrouter.ai/docs/guides/routing/routers/pareto-router.md), sticky routing also reuses the **resolved model** on a best-effort basis when it remains in the current candidate set, not just the provider. The cache hint is ignored when it falls out of that set, so the router may select a different model on a later turn."
- Source: C-s1. Basis: OR-documented.

**C29. A second page gives a 10-minute pin, and says a changed model slug breaks the key.**
- Quote: "**Pass a Consistent `session_id`:** OpenRouter stores a 10-minute best-effort pin directing follow-up turns back to the exact provider endpoint holding the warm KV cache."
- Quote: "**Keep the Model Slug Identical:** Changing from `~anthropic/claude-sonnet-latest` to `~anthropic/claude-sonnet-latest:nitro` mid-conversation invalidates the sticky key."
- Quote: "**Avoid Hardcoded `provider.order`:** Setting explicit `provider.order` disables sticky-session reordering and default routing."
- Quote: "**Keep the Prefix Static:** Ensure system prompts, repository maps, and tool definitions appear at the beginning of the prompt and remain byte-identical across turns. Dynamic timestamps or session metadata should be placed at the end."
- Source: C-s13. Basis: OR-documented. The 10-minute figure conflicts with the 60 minutes in C25 (see Conflicts).

**C30. Auto Exacto can override the pin on tool-calling requests.**
- Quote: "Auto Exacto reorders providers on every tool-calling request, which can conflict with the sticky routing used by [prompt caching](https://openrouter.ai/docs/guides/best-practices/prompt-caching.md). Sticky routing pins subsequent requests to the same provider endpoint to keep KV caches warm, while Auto Exacto may deprioritize that pinned provider mid-session when another endpoint's throughput or tool-calling success rate improves. In tool-calling agent loops, this can cause cache misses mid-conversation."
- Quote: "If your agent loop sends large tool definitions on every turn and cache hit rate is your primary concern, disable Auto Exacto by setting `sort: "price"` in the `provider` object or using the `:floor` model variant."
- Source: C-s14. Basis: OR-documented.

**C31. Default provider choice already weighs each provider's real cache hit rate.**
- Quote: "**Effective price**: the price the provider actually charges, reflecting its real cache hit rate and its typical mix of input and output tokens."
- Quote: "If a request fails, the retry is picked the same way from the providers that are left. Follow-up requests in a conversation stay with the same provider while its [prompt cache](https://openrouter.ai/docs/guides/best-practices/prompt-caching.md) is warm."
- Quote (provider-facing page): "Effective price is the price your endpoint actually charged over the last 30 minutes, across all its traffic. It reflects your real **cache hit rate** and the mix of input and output tokens, so it can differ a lot from your list price."
- Sources: C-s6, C-s28. Basis: OR-documented.

**C32. What a provider or model switch costs, and what the `user` field does.**
- Quote: "Unlike a fixed model slug, the Auto Router can pick a different model on every turn. Switching models costs extra while the new model rebuilds the prompt cache, so the router is restrained about it."
- Quote: "Re-computing attention over large contexts without a warm cache typically adds 5-10 seconds of prefill delay on every step."
- No page puts a price on a provider switch. The cost inferred from the pricing rules is item C83.
- Spec, `ChatRequest.user`: "Per-end-user identifier for abuse isolation. Use a stable ID, hash, or pseudonym. When a provider requires a user identity, OpenRouter folds it into the hashed identity sent upstream and never forwards it raw. If omitted, requests use an account-level identity, so provider policy blocks can affect the whole account." The description names abuse isolation and says nothing of routing. The sticky keys named in the docs are the message hash, `session_id`, `x-session-id`, and `prompt_cache_key`.
- Sources: C-s24, C-s13, C-s30. Basis: OR-documented.

### D. Usage fields in the response

**C33. Usage is always returned. The two opt-in parameters are deprecated and do nothing.**
- Quote: "OpenRouter automatically returns detailed usage information with every response" and "No additional parameters are required."
- Quote: "On Chat Completions, the `usage: { include: true }` and `stream_options: { include_usage: true }` parameters are deprecated and have no effect. The Responses API has no equivalent parameter. Full usage details are always included automatically in every response on both APIs."
- Spec, `ChatStreamOptions.include_usage`: `"deprecated": true`, description "Deprecated: This field has no effect. Full usage details are always included." `ChatRequest` lists `stream_options` among its 43 properties and lists no `usage` property.
- Sources: C-s4, C-s30. Basis: OR-documented.

**C34. Every field of `usage` on Chat Completions (schema `ChatUsage`, used by `ChatResult.usage` and `ChatStreamChunk.usage`).**
- Required: `prompt_tokens` ("Number of tokens in the prompt"), `completion_tokens` ("Number of tokens in the completion"), `total_tokens` ("Total number of tokens").
- `prompt_tokens_details` (object or null): `cached_tokens` ("Cached prompt tokens"), `cache_write_tokens` ("Tokens written to cache. Only returned for models with explicit caching and cache write pricing."), `audio_tokens` ("Audio input tokens"), `video_tokens` ("Video input tokens").
- `completion_tokens_details` (object or null): `reasoning_tokens` ("Tokens used for reasoning"), `audio_tokens` ("Tokens used for audio output"), `accepted_prediction_tokens`, `rejected_prediction_tokens`.
- `cost` (number or null, "Cost of the completion").
- `cost_details` (schema `CostDetails`, "Breakdown of upstream inference costs"): `upstream_inference_cost` (number or null), `upstream_inference_prompt_cost` (required), `upstream_inference_completions_cost` (required), `server_tool_cost` ("Metered server-tool execution cost (for example, shell sandbox time) billed for this request, in USD. Matches the billed checkpoint and settlement amounts exactly. 0 when a metered server tool ran but settled at zero dollars; absent when no metered server tool ran.").
- `is_byok` ("Whether a request was made using a Bring Your Own Key configuration").
- `server_tool_use_details` (schema `ServerToolUseDetails`, "Usage for server-side tool execution (e.g., web search)"): `tool_calls_requested`, `tool_calls_executed`, `web_search_requests`. The name is `server_tool_use_details`, not `server_tool_use`. (`server_tool_use` is the name on the `/messages` route, C37.)
- `deferred_tools_details` (C44).
- Source: C-s30. Basis: OR-documented.

**C35. The API overview page shows the same object as a TypeScript type, with one field the spec does not list.**
- Quote: "prompt_tokens: number;" under the comment "/** Including images, input audio, and tools if any */".
- Quote: "completion_tokens_details?: { reasoning_tokens?: number; // Tokens generated for reasoning audio_tokens?: number; // Tokens generated for audio output image_tokens?: number; // Tokens generated for image output };" (line breaks removed).
- Quote: "cached_tokens: number; // Tokens cached by the endpoint" and "cache_write_tokens?: number; // Tokens written to cache (models with explicit caching)".
- The page lists `completion_tokens_details.image_tokens`. The spec's `ChatUsage` lists `accepted_prediction_tokens` and `rejected_prediction_tokens` there and does not list `image_tokens`. See Conflicts.
- Source: C-s12. Basis: OR-documented.

**C36. Usage on the Responses route (schema `Usage` = `OpenAIResponsesUsage` plus OpenRouter additions).**
- `OpenAIResponsesUsage` (all five required): `input_tokens`, `input_tokens_details` with `cached_tokens` (required) and `cache_write_tokens` (integer or null), `output_tokens`, `output_tokens_details` with `reasoning_tokens` (required), `total_tokens`.
- Additions: `cost`, `cost_details` with `upstream_inference_cost`, `upstream_inference_input_cost`, `upstream_inference_output_cost`, `server_tool_cost`; `is_byok`; `server_tool_use_details`; `deferred_tools_details`.
- Quote: "`cost_details.upstream_inference_cost`: The actual cost charged by the upstream AI provider, split into `upstream_inference_input_cost` and `upstream_inference_output_cost` on the Responses API (`upstream_inference_prompt_cost` and `upstream_inference_completions_cost` on Chat Completions)".
- OpenAI's own per-TTL object `cache_write_tokens_details` (C64) is not among the properties of `input_tokens_details` in this schema.
- Sources: C-s30, C-s4. Basis: OR-documented.

**C37. Usage on the `/messages` route keeps Anthropic's shape, for any model, with OpenRouter fields added.**
- `AnthropicUsage`: `input_tokens`, `output_tokens`, `output_tokens_details` (`thinking_tokens`), `cache_creation_input_tokens`, `cache_read_input_tokens`, `cache_creation` (`ephemeral_5m_input_tokens`, `ephemeral_1h_input_tokens`), `inference_geo`, `server_tool_use` (`web_search_requests`, `web_fetch_requests`), `service_tier`. All nine are required, several nullable.
- Additions on `MessagesResult.usage`: `cost`, `cost_details` (the same `CostDetails` as chat), `is_byok`, `iterations`, `deferred_tools_details`, `service_tier`, `speed`, and `server_tool_use` widened with `tool_calls_requested` and `tool_calls_executed`.
- Example in the docs, an OpenAI model answered on `/messages`: `"model":"openai/gpt-5-mini","stop_reason":"end_turn","stop_details":null,"stop_sequence":null,"usage":{"input_tokens":13,"output_tokens":750,"output_tokens_details":{"thinking_tokens":256},"cache_creation_input_tokens":null,"cache_read_input_tokens":0,"cache_creation":null,"inference_geo":null,"server_tool_use":null,"service_tier":"flex","speed":"standard","cost":0.000751625,"is_byok":false,"cost_details":{"upstream_inference_cost":0.000751625,"upstream_inference_prompt_cost":0.000001625,"upstream_inference_completions_cost":0.00075}}`.
- Example, a Claude model on `/messages`: `usage: { input_tokens: 748, output_tokens: 263, output_tokens_details: null, cache_creation_input_tokens: 1124, cache_read_input_tokens: null, cache_creation: null, inference_geo: null, ...`.
- Sources: C-s30, C-s8, C-s26. Basis: OR-documented. No page says whether `input_tokens` on this route leaves out cached tokens when the serving vendor is not Anthropic (see Open questions).

**C38. The final usage chunk on a chat stream has a non-empty `choices` array (LC12 holds).**
- Quote: "On the Chat Completions endpoint (`/api/v1/chat/completions`), every stream ends with an extra chunk that carries the `usage` object for the request, sent just before the `[DONE]` message. OpenAI's spec emits this chunk with an empty `choices` array, but many clients crash when accessing `choices[0].delta` on it, so OpenRouter intentionally deviates: the usage chunk contains one choice with a content-free `delta` that repeats the `finish_reason` (and `native_finish_reason`) of the stream."
- Quote (wire example): `data: {"id":"gen-abc123",...,"choices":[{"index":0,"delta":{"content":"","role":"assistant"},"finish_reason":"stop","native_finish_reason":"stop"}],"usage":{...}}`
- Quote: "This means the terminal `finish_reason` appears twice: once on the last content-bearing chunk and again on the usage chunk. Clients that validate streams should treat the usage chunk as an accounting frame rather than a second terminal event."
- A captured chunk elsewhere in the docs: `data: {"choices":[{"delta":{"content":"","role":"assistant"},"finish_reason":"stop","index":0}],"created":1794949778,"id":"gen-1794949778-riLGFRdE0YzhPTQFTzBI","model":"openai/gpt-5.2","object":"chat.completion.chunk","service_tier":"default","usage":{"completion_tokens":14,"prompt_tokens":8,"total_tokens":22,"completion_tokens_details":{"audio_tokens":0,"reasoning_tokens":0},"cost":0.00021,"cost_details":{"upstream_inference_completions_cost":0.000196,"upstream_inference_prompt_cost":0.000014,"upstream_inference_cost":0.00021},"is_byok":false,"prompt_tokens_details":{"audio_tokens":0,"cache_write_tokens":0,"cached_tokens":0,"video_tokens":0}}}`
- Spec: `ChatStreamChunk` requires `id`, `choices`, `created`, `model`, `object`, and has an optional `usage`.
- Sources: C-s11, C-s12, C-s27, C-s30. Basis: OR-documented.

**C39. Where usage arrives on the other two routes.**
- Quote: "This shape is specific to Chat Completions. Other endpoints follow their own specs: the Responses API (`/api/v1/responses`) reports usage in the `response.completed` event, and the Messages API (`/api/v1/messages`) reports it in the `message_delta` event before `message_stop`."
- Quote: "This information is included in the terminal `response.completed` or `response.incomplete` event for streaming responses, or in the terminal response for non-streaming requests."
- Sources: C-s11, C-s4. Basis: OR-documented.

**C40. Token counts are the model's native counts, taken from the provider's report, with estimates as a fallback.**
- Quote: "The token counts returned in the API response are calculated using the model's native tokenizer. Credit usage and model pricing are based on these native token counts."
- Quote: "Input and output token counts using the model's native tokenizer".
- Quote (Broadcast page): "Its `promptTokens` and `completionTokens` are the token counts used by OpenRouter's accounting layer, normally derived from provider-reported usage." and "These can contain fallback estimates when provider usage is missing."
- Quote (FAQ): "When you make a request to OpenRouter, we receive the total number of tokens processed by the provider. We then calculate the corresponding cost and deduct it from your credits."
- Sources: C-s12, C-s4, C-s15, C-s10. Basis: OR-documented.

**C41. `prompt_tokens` is the whole prompt with cached tokens inside it, and `completion_tokens` has reasoning tokens inside it.**
- Official SDK, `src/utils/compute-token-usage.ts` lines 24 to 36: `inputTokens: { total: promptTokens, noCache: promptTokens - cacheReadTokens, cacheRead: cacheReadTokens, cacheWrite: cacheWriteTokens, }, outputTokens: { total: completionTokens, text: completionTokens - reasoningTokens, reasoning: reasoningTokens, },`. The subtraction only makes sense if `cached_tokens` is a subset of `prompt_tokens` and `reasoning_tokens` a subset of `completion_tokens`.
- Doc example: `"prompt_tokens": 10339, "completion_tokens": 60, "total_tokens": 10399, "prompt_tokens_details": { "cached_tokens": 10318, "cache_write_tokens": 0 }`.
- Quote (reasoning): "To detect this case, subtract `usage.output_tokens_details.reasoning_tokens` from `usage.output_tokens`. The difference is the number of visible output tokens".
- Quote (Broadcast cost allocation): "`input_tokens` | All billed input tokens, including cache reads and cache writes: `non_cache_input_tokens + cache_read_input_tokens + cache_write_input_tokens`."
- Sources: C-s32, C-s1, C-s22, C-s15. Basis: OR-documented. The specific claim about Anthropic-served requests is item C84.
- Detail: the SDK's `noCache` subtracts cache reads only. OpenAI's own formula subtracts both reads and writes (C64). So in the SDK's numbers, written tokens are counted inside `noCache`.

**C42. `cache_write_tokens` is the name for cache writes (LC17 holds), and it is no longer only an OpenRouter name.**
- Quote: "`cached_tokens` is the number of tokens that were *read* from the cache. `cache_write_tokens` is the number of tokens that were *written* to the cache (only returned for models with explicit caching and cache write pricing)."
- Quote: "`cache_write_tokens`: Number of tokens written to the cache. This appears on the first request when establishing a new cache entry."
- Quote (OpenAI 12-hour writes): "Tokens written with a 12-hour TTL are billed at the 12-hour cache-write rate; the rest of the write is billed at the 30-minute rate. Both are included in `cache_write_tokens` in the response usage."
- Quote: "Cache activity is reported in `usage.input_tokens_details` (Responses) and `usage.prompt_tokens_details` (Chat Completions): `cache_write_tokens` counts prompt tokens written to the cache, and `cached_tokens` counts prompt tokens read from it."
- The same name is now native at OpenAI (`usage.input_tokens_details.cache_write_tokens`, C64) and at Moonshot (`usage.prompt_tokens_details.cache_write_tokens`, C69). LiteLLM uses a different name, `cache_creation_tokens` (C73). Alibaba uses `cache_creation_input_tokens` inside `prompt_tokens_details` (C70).
- Sources: C-s4, C-s1, C-s2. Basis: OR-documented.

**C43. Cost fields.**
- Quote: "`cost`: The total amount charged to your account".
- Quote: "Cost in credits" (item 2 of the list of what usage includes).
- Quote (generation endpoint): "When obtaining usage information via generation ID, the `upstream_inference_cost` field is only available for BYOK (Bring Your Own Key) requests. For all other requests it will be 0 or null."
- In the response `usage`, the captured chunk in C38 shows `upstream_inference_cost` filled on a request with `"is_byok":false`, equal to `cost`.
- The page's own JSON examples show `"cost": 0.95` beside `"upstream_inference_cost": 19`, which reads as placeholder data. The numbers do not relate to each other.
- Sources: C-s4, C-s27. Basis: OR-documented.

**C44. With deferred tools, the token fields cover the final upstream call only, while cost covers every call.**
- Spec, `DeferredToolsDetails`: "Totals across every upstream model call of a deferred-tools run. The native usage fields report only the final call; cost covers every call." Fields: `turns`, `prompt_tokens`, `completion_tokens`, `total_tokens`.
- Changelog: "`ChatUsage`: new property `deferred_tools_details`" under "November 5, 2026".
- Sources: C-s30, C-s18. Basis: OR-documented.

**C45. OpenRouter's own response cache is a different feature, and a hit reports zero usage.**
- Quote: "Response caching allows you to cache responses for identical API requests. When a cached response is available, OpenRouter returns it immediately from cache with no billing (all billable usage counters are reported as `0`), reducing both latency and cost."
- Quote: "Two requests are considered identical when they share the same API key, model, endpoint type, streaming mode, and request body (including all parameters)."
- Quote: "**Provider caching**: Some providers offer their own prompt caching (e.g. [Anthropic prompt caching](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching), [OpenAI cached context](https://platform.openai.com/docs/guides/prompt-caching)). Provider caching is separate from OpenRouter response caching and the two can be used together."
- Headers: request `X-OpenRouter-Cache`, `X-OpenRouter-Cache-TTL`, `X-OpenRouter-Cache-Clear`; response `X-OpenRouter-Cache-Status` (`HIT` or `MISS`), `X-OpenRouter-Cache-Age`, `X-OpenRouter-Cache-TTL`, `X-OpenRouter-Cache-Source-Id`. Default TTL 300 seconds, range 1 to 86400.
- Generation API: `response_cache_source_id` ("If this generation was served from response cache, contains the original generation ID. Null otherwise.").
- Sources: C-s3, C-s30. Basis: OR-documented.

**C46. A response with zero output and a blank or error finish reason is not billed.**
- Quote: "When a response contains no output tokens and either has a blank finish reason or an error, you will not be charged for the model's token usage, even if the underlying provider charges for prompt processing."
- Source: C-s7. Basis: OR-documented.

**C47. The vendor's raw usage object, and the per-TTL write split, are exposed through Broadcast (the observability export), not through the API response.**
- Quote: "With the **Cost** metadata opt-in enabled, `upstream_raw_response_usage` carries the provider's own usage values as they arrived, before OpenRouter normalizes them. Unknown provider fields are preserved, so this object can change whenever a provider changes its API."
- Quote: "This field is provider evidence, not an invoice calculation, and not every provider reports usage. Its values are not OpenRouter's accounting inputs".
- Quote: "`promptTokensDetails.cacheWriteTokens` | Tokens written to the provider's prompt cache. Uses the provider-reported aggregate when available; otherwise the pricing layer's cache-write quantity."
- Quote: "`promptTokensDetails.cacheCreation.ephemeral_5m_input_tokens` | Cache-write tokens with a five-minute TTL, when the provider reports a complete breakdown."
- Quote: "`usage_is_estimated` | `true` when accounting used a fallback prompt or completion quantity, or fallback pricing after a usage-calculation failure."
- Quote: "Provider-reported cache-write totals can differ from the quantities normalized by OpenRouter's pricing layer."
- Quote: "Counts are the quantities OpenRouter billed, so they can differ from `promptTokens` when the provider's own report is inconsistent. For example, when Gemini reports cache reads above an uncached input, it bills both, and `input_tokens` follows what was billed."
- Example of the raw object in the docs: `"upstream_raw_response_usage": { "input_tokens": 4, "output_tokens": 373, "cache_creation_input_tokens": 300, "cache_read_input_tokens": 0, "cache_creation": { "ephemeral_5m_input_tokens": 100, "ephemeral_1h_input_tokens": 200 } }`.
- Source: C-s15. Basis: OR-documented. This is the one place where OpenRouter says in words that it normalizes vendor usage ("before OpenRouter normalizes them").

**C48. On Gemini 4 multi-hop answers each hop is a separate generation with its own usage.**
- Quote: "Every hop is its own generation with its own usage and cost. A continuation hop's prompt tokens include all the reasoning from earlier hops, and each hop bills its full prompt, so input cost grows with every hop." and "for context-window or compaction checks, use the first hop's `prompt_tokens`, because the summed prompt tokens count earlier output once per hop and are much larger than the context the model uses."
- Source: C-s21. Basis: OR-documented.

**C49. Reasoning tokens are output tokens for billing.**
- Quote: "Reasoning tokens are considered output tokens and charged accordingly."
- Quote: "Reasoning tokens are counted as output tokens for billing purposes."
- Quote (Broadcast): "`reasoning_tokens` appears only for providers that price reasoning separately; elsewhere reasoning is billed, and counted, as output."
- Sources: C-s22, C-s15. Basis: OR-documented.

### E. The generation stats API

**C50. `GET /generation?id=<generation id>` returns 45 fields under `data`.**
- Query parameter: `id` ("The generation ID", pattern `^gen-[0-9A-Za-z-]+$`, max length 128). Sibling paths: `/generation/content` and `/generation/feedback`.
- Token fields: `tokens_prompt` ("Number of tokens in the prompt"), `tokens_completion` ("Number of tokens in the completion"), `native_tokens_prompt` ("Native prompt tokens as reported by provider"), `native_tokens_completion` ("Native completion tokens as reported by provider"), `native_tokens_reasoning` ("Native reasoning tokens as reported by provider"), `native_tokens_cached` ("Native cached tokens as reported by provider"), `native_tokens_completion_images` ("Native completion image tokens as reported by provider").
- Cost fields: `total_cost` ("Total cost of the generation in USD"), `usage` ("Usage amount in USD"), `cache_discount` ("Discount applied due to caching"), `upstream_inference_cost` ("Cost charged by the upstream provider"), `is_byok` ("Whether this used bring-your-own-key").
- Outcome fields: `finish_reason` ("Reason the generation finished"), `native_finish_reason` ("Native finish reason as reported by provider"), `cancelled`, `streamed`.
- Timing fields, all in milliseconds: `latency` ("Total latency in milliseconds"), `generation_time` ("Time taken for generation in milliseconds"), `moderation_latency` ("Moderation latency in milliseconds").
- Routing fields: `provider_name` ("Name of the provider that served the request"), `provider_responses` ("List of provider responses for this generation, including fallback attempts"), `router`, `service_tier` ("Service tier the upstream provider reported running this request on, or null if it did not report one."), `data_region` (enum `global`, `europe`, `us`), `upstream_id` ("Upstream provider's identifier for this generation"), `model`.
- Identity and grouping fields: `id`, `request_id` ("Unique identifier grouping all generations from a single API request"), `session_id`, `app_id`, `external_user`, `preset_id`, `workspace_id`, `origin`, `http_referer`, `user_agent`, `created_at`, `api_type` (enum `completions`, `embeddings`, `rerank`, `decisions`, `extractions`, `video`, `tts`, `stt`, `image`, `tools_invoke`, null).
- Counters: `num_media_prompt`, `num_input_audio_prompt`, `num_media_completion`, `num_search_results`, `num_fetches`, `web_search_engine`, `response_cache_source_id`.
- Source: C-s5, C-s30. Basis: OR-documented.

**C51. `cache_discount` goes negative on Anthropic cache writes (this part of LC7 holds).**
- Quote: "The `cache_discount` field in the response body will tell you how much the response saved on cache usage. Some providers, like Anthropic, will have a negative discount on cache writes, but a positive discount (which reduces total cost) on cache reads."
- Spec: `cache_discount` is a number or null, described as "Discount applied due to caching", and it is a property of exactly one schema, `GenerationResponse.data`. The chat, responses, and messages `usage` objects do not list it. So "the response body" in the quote is the body of `GET /generation`.
- Sources: C-s1, C-s30. Basis: OR-documented.

**C52. The generation record separates the provider's counts from OpenRouter's own estimates, and has one cache counter.**
- Quote (Broadcast page): "The existing `openrouter_generation.tokens_prompt` and `tokens_completion` fields are OpenRouter's own token estimates, not the provider's token counters."
- `native_tokens_cached` is the only cache count in `GenerationResponse.data`. A cache-write count is not a listed field there. Writes show through a negative `cache_discount` (C51) and through `prompt_tokens_details.cache_write_tokens` in the original response.
- Per vendor: the spec gives one field for every vendor and no per-vendor notes. How each vendor's read count maps to `native_tokens_cached` is not written down (see Open questions).
- Sources: C-s15, C-s30. Basis: OR-documented. The link from the Broadcast field to the API field of the same name is item C85.

**C53. On the generation endpoint, `upstream_inference_cost` is filled for BYOK only.**
- Quote: "When obtaining usage information via generation ID, the `upstream_inference_cost` field is only available for BYOK (Bring Your Own Key) requests. For all other requests it will be 0 or null."
- Source: C-s4. Basis: OR-documented.

**C54. Each provider attempt, including fallbacks, is listed.**
- Schema `ProviderResponse`: `endpoint_id`, `id` ("Upstream provider response identifier"), `is_byok`, `latency` ("Response latency in milliseconds"), `model_permaslug`, `provider_name` (an enum of 154 provider names), `routed_service_tier` (enum `flex`, `priority`, `ultrafast`), `status` ("HTTP status code from the provider", required).
- Source: C-s30. Basis: OR-documented.

**C55. The generation id arrives in a header on every route, and analytics token metrics are native.**
- Quote: "The generation ID is returned in the `X-Generation-Id` response header for all endpoints (chat completions, completions, responses, and messages), which can be useful for debugging and correlating requests."
- Quote: "Spend metrics (`total_usage`, `usage_*`) are in USD. Token metrics are native tokens. `cache_hit_rate` is a 0 to 1 ratio."
- Sources: C-s11, C-s29. Basis: OR-documented.

### E2. Cache-preserving controls that depend on the route

**C56. Some cache-preserving controls exist on the `/messages` route only.**
- Quote: "**Mid-conversation tool changes (beta)** - add and remove tools between turns without invalidating the prompt cache, via OpenRouter's [Messages API](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md) only".
- Quote (effort change): "To change effort partway through a conversation without invalidating the prompt cache, add a `configuration_update` input item before the user message it should apply to. The item is interchangeable with the Chat Completions and Anthropic Messages API forms."
- Spec: `ChatSystemMessage` and `ChatDeveloperMessage` list a `configuration_update` property. `tool_addition` and `tool_removal` are content block types of `MessagesMessageParam` only.
- Sources: C-s48, C-s49, C-s30. Basis: OR-documented. Other subagents cover these features in depth. They are listed here because each one exists to keep the cached prefix unchanged.

### F. Prices that touch accounting

**C57. Cache prices in the public model catalog.**
- Pricing keys across the 476 models, with the number of models that have each: `prompt` 476, `completion` 476, `input_cache_read` 342, `web_search` 198, `input_cache_write` 117, `overrides` 87, `audio` 38, `internal_reasoning` 36, `input_cache_write_1h` 36, `image` 35, `input_audio_cache` 33, `image_output` 10, `audio_output` 2.
- Spec descriptions (`PublicPricing`): `input_cache_read` "Price in USD per cached input token (read)"; `input_cache_write` "Price per cache-write token, in USD per token. For providers with multiple cache TTLs (e.g. Anthropic), this is the default (5-minute) cache-write rate."; `input_cache_write_1h` "Price per 1-hour cache-write token, in USD per token. Only present for providers that price an extended (1-hour) cache TTL separately, such as Anthropic."; `internal_reasoning` "Price in USD per internal reasoning token".
- By model prefix: `input_cache_write_1h` is on all 32 `anthropic/` models and the 4 `~anthropic/` aliases and nowhere else. `input_cache_write` is on 36 of 106 `openai/` models, all 32 `anthropic/`, 28 of 49 `google/`, and 9 of 43 `qwen/`.
- Example: `anthropic/claude-sonnet-5.5` has `{"prompt": "0.000002", "completion": "0.00001", "web_search": "0.01", "input_cache_read": "0.0000001", "input_cache_write": "0.0000025", "input_cache_write_1h": "0.000004"}`.
- A key for OpenAI's 12-hour write rate (2.5x) is not among the catalog keys. `openai/gpt-5.6-sol` has one `input_cache_write` of `0.000005`, which is 1.25x its prompt price.
- Sources: C-s31, C-s30, C-s16. Basis: OR-documented.

**C58. Catalog ratios against the multipliers stated on the prompt caching page.**
- Computed from the catalog (cache price divided by prompt price): `anthropic/claude-sonnet-5.5` read 0.05, write 1.25, 1-hour write 2.0. `anthropic/claude-opus-5.5` read 0.05. `openai/gpt-5.6-sol` read 0.1, write 1.25. `openai/gpt-5.2` read 0.1. `openai/gpt-4o` read 0.5. `google/gemini-2.5-flash` read 0.1, write 0.2778. `google/gemini-2.5-pro` read 0.1, write 0.3. `deepseek/deepseek-v3.2` read 0.5. `x-ai/grok-4.7` read 0.25. `moonshotai/kimi-k3` read 0.948. `z-ai/glm-5.3` read 0.986. `qwen/qwen-plus` read 0.2, write 1.25.
- The page says Anthropic reads are 0.1x, OpenAI reads "0.25x or 0.50x", Gemini reads 0.25x, DeepSeek reads 0.1x, Moonshot reads 0.25x (C2, C6, C10, C12).
- The migration guides explain the Anthropic gap: "**Pricing**: cache reads cost less than on Sonnet 5." and "prompt cache reads cost a quarter of the Fable 5 rate."
- The top-level catalog price is one endpoint's price. A model served by several providers (for example `moonshotai/kimi-k3`, `deepseek/deepseek-v3.2`) can show a third party's rate, so a ratio near 1.0 there says little about the vendor's own rate.
- Sources: C-s31, C-s1, C-s19, C-s20. Basis: OR-documented for the raw prices. The ratios are my arithmetic.

**C59. Reasoning tokens have their own catalog price on Google models, equal to the completion price.**
- `internal_reasoning` is on 33 of 49 `google/` models, 2 `~google/` aliases, and `perplexity/sonar-deep-research`. On every Google entry it equals `completion` (for example `google/gemini-2.5-flash`: `"completion": "0.0000025"`, `"internal_reasoning": "0.0000025"`). On `perplexity/sonar-deep-research` it differs: `internal_reasoning=0.000003`, `completion=0.000008`.
- Quote (FAQ): "There is usually a different price for prompt and completion tokens. There are also models that charge per request, for images and for reasoning tokens."
- Sources: C-s31, C-s10. Basis: OR-documented.

**C60. Long-context tiers override the cache prices too.**
- Example: `openai/gpt-5.6-sol` has `"overrides": [{"min_prompt_tokens": 272000, "prompt": "0.000008", "completion": "0.00003", "input_cache_read": "0.0000008", "input_cache_write": "0.00001"}]`.
- Quote: "An entry applies when all of its condition fields match the request. When multiple entries apply, later entries win per key. Price keys absent from an entry inherit the base price."
- Sources: C-s31, C-s16. Basis: OR-documented.

**C61. How providers declare cache prices to OpenRouter, and what OpenRouter cannot bill yet.**
- Quote: "Cache prices are first-class SKUs in a modality's `pricing` array. Multiple entries may have the same `type` when their qualifier fields differ. The effective identity of an entry is its `type` together with its qualifiers, so two `cache_write` entries with different TTLs are separate SKUs."
- Quote: "`ttl_seconds` is the cache lifetime the price applies to." and "`implicit` marks provider-initiated caching that the request does not ask for and defaults to `false`. Explicit and implicit cache modes can coexist on one model."
- Pricing types: `cached_prompt` ("Cost per unit read from prompt cache") and `cache_write` ("Cost per unit written to prompt cache").
- Quote: "Entries OpenRouter cannot represent yet (for example search pricing, TTL cache tiers, or non-token units) are ignored with validator warnings `unsupported_pricing_entry`, `unsupported_cache_qualifier`, or `unsupported_override`."
- Source: C-s17. Basis: OR-documented.

### G. Each vendor's native rule, read live from the vendor

**C62. Anthropic: prefix order, 4 breakpoints, 20-position lookback, silent minimum, TTL and prices.**
- Quote: "Prompt caching covers the full prefix: `tools`, `system`, and `messages` (in that order), up to and including the block designated with `cache_control`."
- Quote: "**Cache writes occur only at your breakpoint.** Marking a block with `cache_control` writes exactly one cache entry: a hash of the prefix ending at that block."
- Quote: "**The lookback window is 20 blocks.** The system checks at most 20 positions for each breakpoint, counting the breakpoint itself as the first." and "On the Claude API, a run of consecutive `tool_use` blocks counts as one position, and so does a run of consecutive `tool_result` blocks, so a turn with many parallel tool calls doesn’t push the previous request’s entry out of the window on its own."
- Quote: "You can define up to 4 cache breakpoints".
- Quote (automatic): "Add a single `cache_control` field at the top level of your request. The system automatically applies the cache breakpoint to the last cacheable block and moves it forward as conversations grow."
- Quote: "When used together, the automatic cache breakpoint uses one of the 4 available breakpoint slots." and "If 4 explicit block-level breakpoints already exist, the API returns a 400 error (no slots left for automatic caching)."
- Quote: "Automatic caching is available on every platform except [Amazon Bedrock (`bedrock-runtime`)](https://platform.claude.com/docs/en/build-with-claude/claude-on-amazon-bedrock-legacy). There, the API returns a 400 error for a top-level `cache_control` field".
- Quote: "Shorter prompts cannot be cached, even if marked with `cache_control`. Any requests to cache fewer than this number of tokens are processed without caching, and no error is returned."
- Minimums: "512 tokens for Claude Fable 5.2, [Claude Mythos 5.2] ... Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Fable 5, [Claude Mythos 5] ... and Claude Haiku 5.5"; "2,048 tokens for [Claude Mythos Preview] ... and Claude Opus 4.7"; "4,096 tokens for Claude Opus 4.6 and Claude Opus 4.5"; "1,024 tokens for Claude Opus 4.8, Claude Sonnet 5, Claude Sonnet 4.6, Claude Sonnet 4.5"; "4,096 tokens for Claude Haiku 4.5" (link targets shortened with "...").
- Quote: "Thinking blocks cannot be cached directly with `cache_control`."
- Quote (TTL): "By default, automatic caching uses a 5-minute TTL. You can specify a 1-hour TTL at 2x the base input token price:" then `{ "cache_control": { "type": "ephemeral", "ttl": "1h" } }`.
- Quote (prices): "Cache write tokens with a 5-minute TTL are 1.25 times the base input tokens price.", "Cache write tokens with a 1-hour TTL are 2 times the base input tokens price.", "Cache read tokens are 0.1 times the base input tokens price (see the table footnote for per-model exceptions)."
- Footnotes: "Cache hits and refreshes on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, and Claude Mythos 5.1 are priced at 0.025x the base input price." and "Cache hits and refreshes on Claude Opus 5.5 and Claude Sonnet 5.5 are priced at 0.05x the base input price."
- Source: C-s33. Basis: Vendor-documented. Also in workspace: the 2026-11-17 report on Anthropic's Messages API (C-s47, rule 6 there gives the same numbers).

**C63. Anthropic: `input_tokens` is only the part after the last breakpoint.**
- Quote: "The `input_tokens` field represents only the tokens that come **after the last cache breakpoint** in your request, not all the input tokens you sent."
- Quote: "total_input_tokens = cache_read_input_tokens + cache_creation_input_tokens + input_tokens".
- Fields: "`cache_creation_input_tokens`: Number of tokens written to the cache when creating a new entry", "`cache_read_input_tokens`: Number of tokens retrieved from the cache for this request", "`input_tokens`: Number of input tokens that were not read from or used to create a cache (that is, tokens after the last cache breakpoint)".
- Source: C-s33. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey (C-s46), section 5.5, and the 2026-11-17 report (C-s47), usage mapping table.

**C64. OpenAI: implicit and explicit modes, four writes, TTL, keys, retention, and usage fields.**
- Quote: "The minimum cacheable prompt length is 1,024 tokens for GPT-5.6 and later and varies by request settings for earlier models."
- Quote: "For GPT-5.6 and later, cache writes cost 1.25× the standard, uncached input-token rate for the default 30-minute lifetime."
- Quote: "Set `prompt_cache_options.mode` to `explicit` to use only developer-selected breakpoints and mark each desired breakpoint by adding `prompt_cache_breakpoint: { "mode": "explicit" }` to a supported content block inside an input message."
- Quote: "When no explicit breakpoints are placed, the request does not use prompt caching or create cache writes."
- Quote: "Multiple explicit breakpoints can preserve prefixes that change at different rates. Each request can create up to four cache writes."
- Quote: "When `prompt_cache_options.mode` is `implicit`, OpenAI places a breakpoint at the end of the latest eligible message."
- Quote: "You can add explicit breakpoints without turning off the implicit breakpoint; an implicit breakpoint uses one of the four cache write slots to leave three usable explicit cache write slots."
- Quote: "Top-level `instructions` cannot contain an explicit breakpoint."
- Quote (TTL): "In the Responses API, set `prompt_cache_options.ttl` to choose the minimum cache lifetime:" with rows `"30m"` (default) at "1.25× the uncached input-token rate" and `"12h"` at "2.5× the uncached input-token rate". Summary table row: "Cache write charge | 1.25× for `"30m"`; 2.5× for `"12h"` in the Responses API".
- Quote: "Unsupported values return `400 invalid_request_error`. These settings do not change the older-model `prompt_cache_retention` parameter."
- Quote (older models): "Use `prompt_cache_retention`, with supported values that depend on the model:" then "`in_memory`: Entries typically remain active for around 5 to 10 minutes of inactivity, up to one hour." and "`24h`: Extended retention typically keeps entries available for around 30 minutes and can retain them for up to 24 hours."
- Quote (keys): "Keys influence routing; they do not pin requests to a machine or guarantee a cache hit." and "On GPT-5.6 and later, OpenAI handles cache routing automatically; the key is not needed to optimize caching. You can use separate keys to maintain separate cache accounting for customers or users within your application."
- Quote: "Cached states live on individual machines, where traffic above 15 requests per minute can lead to overflow routing."
- Quote (usage): "For Responses API models that support `prompt_cache_options`, `input_tokens_details.cache_write_tokens_details` breaks writes into `ttl_30m_tokens` and `ttl_12h_tokens`. These counts do not overlap and sum to `cache_write_tokens`." and "In this usage example, 1,000 input tokens are ordinary input: `12000 - 4000 - 7000`."
- Quote (older models): "Reported `cached_tokens` is calculated by subtracting the hidden system tokens from the last matched breakpoint, then rounding down to the nearest multiple of 128."
- Quote (read price): "Cache read charge | 0.1× uncached input (0.05× for GPT-6.2 Astra, GPT-6.2 Sol, GPT-6.1 Sol, and GPT-6.1 Terra)".
- Source: C-s34. Basis: Vendor-documented.

**C65. Google Gemini: implicit caching everywhere, named cache objects on `generateContent` for Gemini 2.5 and 3 only.**
- Quote: "The **Interactions API** only supports implicit caching. Explicit caching (manually creating and managing cache objects) is not supported in the Interactions API, or for Gemini 4 models. To use explicit caching on supported Gemini 3 and 2.5 models, switch to the [generateContent API](/docs/caching?generate-content=true)." (source line wraps joined)
- Quote: "Implicit caching is enabled by default for all Gemini 2.5 and newer models."
- Minimum input tokens for implicit caching, from Google's table: Gemini 4 Carbon, Argon, Neon, Hydrogen 4,096; Gemini 3.8 Flash 4,096; Gemini 3.6 Flash 4,096; Gemini 3.1 Pro Preview 4,096; Gemini 2.5 Flash 2,048; Gemini 2.5 Pro 2,048.
- Quote: "**Multi-turn and agent loops on Gemini 4:** Gemini caches the full previous request only if the new request adds at least 2,050 tokens. Model output, thought, and function results all count toward this total." (source line wraps joined)
- Quote (explicit): "This caching duration is called the *time to live* (TTL). If not set, the TTL defaults to 1 hour."
- Quote: "Explicit context caching is currently in Beta. Endpoints and SDK methods are available under `v1beta`. Explicit caching is not supported for Gemini 4 models; Gemini 4 models use [implicit caching](/docs/caching#implicit-caching) by default."
- Native calls: `curl -X POST "https://generativelanguage.googleapis.com/v1beta/cachedContents"` to create, then `"cachedContent": "'"$CACHE_NAME"'"` in the `generateContent` body.
- Quote: "**Storage duration:** The amount of time cached tokens are stored (TTL), billed based on the TTL duration of cached token count. There are no minimum or maximum bounds on the TTL."
- Quote: "Implicit caching doesn't apply when a request uses an explicit cache. Any additional input tokens sent alongside the explicit cache are billed at the standard input rate."
- Quote: "If you're using an [OpenAI library](/docs/openai), you can enable explicit caching using the `cached_content` property on [`extra_body`](/docs/openai#extra-body)."
- API reference, fields of the cache resource: "Optional. Input only. Immutable. Developer set system instruction. Currently text only.", "Optional. Input only. Immutable. A list of `Tools` the model may use to generate the next response", "Optional. Input only. Immutable. Tool config. This config is shared for all tools."
- Usage: "You can see the number of tokens which were cache hits in the response object's `usage_metadata` field." (generateContent page) and "`usage.total_cached_tokens`" (Interactions page).
- Sources: C-s35, C-s36, C-s37. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey (C-s46), section 5.5 (it gave 2,048 for Gemini 2 and 2.5 and 4,096 for Gemini 3, which matches).

**C66. DeepSeek: on by default, reported under its own two field names.**
- Quote: "The DeepSeek API Context Caching on Disk Technology is enabled by default for all users, allowing them to benefit without needing to modify their code."
- Fields in `usage`: `prompt_cache_hit_tokens` ("The number of tokens in the input of this request that resulted in a cache hit.") and `prompt_cache_miss_tokens` ("The number of tokens in the input of this request that did not result in a cache hit.").
- Quote: "The cache system works on a "best-effort" basis and does not guarantee a 100% cache hit rate." and "Once the cache is no longer in use, it will be automatically cleared, usually within a few hours to a few days."
- Source: C-s38 (text taken from the HTML page with tags removed). Basis: Vendor-documented.

**C67. xAI: automatic, with a conversation header that pins the server.**
- Quote: "The xAI API performs prompt caching **automatically**. However, we recommend setting the `x-grok-conv-id` HTTP header to maximize your cache hit rate."
- Quote: "The `x-grok-conv-id` HTTP header routes requests with the same conversation ID to the same server. Since cache entries are stored per-server, this maximizes your cache hit rate."
- Quote: "For the Responses API, use the `prompt_cache_key` field directly in the request body. It functions identically to setting `x-grok-conv-id` - it routes requests to the same server for cache reuse."
- Quote: "Cached tokens appear in `usage.prompt_tokens_details.cached_tokens`:" (chat) and "Cached tokens appear in `usage.input_tokens_details.cached_tokens`:" (responses).
- Quote: "Prompt caching is not 100% guaranteed. Cache entries can be evicted due to memory pressure, and requests may be routed to different servers."
- Source: C-s39. Basis: Vendor-documented.

**C68. Groq: automatic, on three GPT-OSS models, 2-hour expiry, 50 percent off.**
- Quote: "Prompt caching works automatically on all your API requests to supported models with no code changes required and no additional fees."
- Quote: "Prompt caching is currently only supported for the following models:" then a table with `openai/gpt-oss-20b`, `openai/gpt-oss-120b`, `openai/gpt-oss-safeguard-20b`.
- Quote: "**Automatic Expiration**: All cached data automatically expires after 2 hours without use." and "There is a 50% discount for cached input tokens." and "Cached tokens do not count towards your rate limits."
- Source: C-s40. Basis: Vendor-documented. OpenRouter's page says "Currently available on Kimi K2 models" (C12). The two lists differ.

**C69. Moonshot: writes are now billed, with a TTL choice under the OpenAI field name `prompt_cache_options`.**
- Quote: "With this platform upgrade, Cache Write is billed as a separate item, and two cache TTL (time to live) options are offered: 5m and 1h."
- Quote: "The Chat Completions API and Responses API write to the cache with a 5m TTL by default when prompt_cache_options is omitted."
- Request example: `"prompt_cache_options": {"mode": "implicit", "ttl": "1h"}`. Quote: "prompt_cache_options.mode currently supports only implicit , and ttl supports 5m and 1h ."
- Quote (its Anthropic-shaped route): "cache_control is effective only at the top level; the same marker inside the messages body is ignored."
- Usage table: Chat Completions total input `usage.prompt_tokens`, cache read `usage.prompt_tokens_details.cached_tokens`, cache write `usage.prompt_tokens_details.cache_write_tokens`. Quote: "For the Chat Completions and Responses APIs, cache reads, cache writes, and the uncached remainder are mutually exclusive and sum to total input tokens."
- Quote: "For streaming Chat Completions API requests, set stream_options.include_usage=true for the complete cache read/write breakdown to appear in the usage field of the final chunk." and "kimi-k3.1-pro and kimi-k3 support Cache Write."
- Prices on the page for kimi-k3: input $3.00, cache write (5m) $3.00, cache write (1h) $6.00, cached input $0.30 per 1M tokens.
- Source: C-s41 (text taken from the HTML page with tags removed, so code formatting is lost and a stray space sits before some punctuation). Basis: Vendor-documented. OpenRouter's page says Moonshot cache writes have "no cost" and need no configuration (C12).

**C70. Alibaba: an Anthropic-style marker with the same limits, and its own write counter name.**
- Quote: "Add a "cache_control": {"type": "ephemeral"} marker to the messages array. The system then searches backward from each cache_control marker and examines up to 20 preceding content blocks to find a cache hit. A single request supports up to four cache markers."
- Quote: "The new cache block has a validity period of 5 minutes." and "A cache block contains at least 1,024 tokens." and "Explicit cache and implicit cache are mutually exclusive."
- Quote: "Tokens used to create the cache are typically billed at 125% of the standard input token price, while subsequent cache hits are typically billed at only 10% of that price."
- Usage in the page's sample code: `first_completion.usage.prompt_tokens_details.cache_creation_input_tokens` and `first_completion.usage.prompt_tokens_details.cached_tokens`.
- Source: C-s42 (page marked "Last Updated:Nov 21, 2026", text taken from HTML). Basis: Vendor-documented.

**C71. Z.AI: implicit caching, OpenAI-shaped read counter.**
- Quote: "**Automatic Cache Recognition**: Implicit caching that intelligently identifies repeated context content without manual configuration" and "**Transparent Billing**: Detailed display of cached token counts in response field `usage.prompt_tokens_details.cached_tokens`".
- Source: C-s43. Basis: Vendor-documented.

### H. Fills from open translators (how another project does what OpenRouter does not describe)

**C72. LiteLLM turns `cache_control` markers into a Gemini named cache by hashing the marked prefix and using the hash as the cache's display name.**
- Split: "Returns the first continuous block of cached messages, plus every system message once any message is cached, since generateContent rejects a system_instruction beside a cachedContent, and the remaining messages." (`litellm/llms/vertex_ai/context_caching/transformation.py`, `separate_cached_messages`, lines 113 to 117)
- Key: `generated_cache_key: Final = local_cache_obj.get_cache_key(messages=cached_messages, tools=tools, tool_choice=tool_choice, model=target.model)` (`vertex_ai_context_caching.py`, lines 485 to 487).
- Lookup: "Currently, checks cache list, for cache key == displayName, since Google doesn't let us set the name of the cache (their API docs are out of sync with actual implementation)." (same file, `check_cache` docstring, line 228). The code pages through `cachedContents` and compares each item's `displayName`.
- Create: `displayName=cache_key` in the request body (`transformation.py`, lines 190 to 194), with `cached_content_request_body["tools"] = tools` (`vertex_ai_context_caching.py`, line 521). The tool parameters are removed from the main request once a cache is used (lines 505 to 506). The call returns the cache `name`, which goes into the `generateContent` body.
- TTL: the Anthropic-style `ttl` string is converted to Gemini's seconds form by `_normalize_ttl_to_seconds`, pattern `^([0-9]*\.?[0-9]+)([smh])$` (`transformation.py`, lines 92 to 107).
- Limits it handles: "The cachedContents API rejects contents ending on a model turn, which is how it classifies both assistant messages and tool results, with HTTP 400 "Requests ending with a model turn are not supported"." (`transformation.py`, lines 134 to 137), and a skip when "cached content is below minimum token count" (`vertex_ai_context_caching.py`, line 147).
- Source: C-s44 at commit `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22). Basis: Filled (LiteLLM). This shows one working design: the gateway holds no table of its own and finds the cache again by listing the vendor's caches. It does not show what OpenRouter does. Two visible differences from OpenRouter's documented behavior: LiteLLM caches the first continuous marked block and converts the TTL, while OpenRouter says it uses "only the last breakpoint" and a fixed 5-minute TTL (C11).

**C73. LiteLLM adds Anthropic's cache counters into `prompt_tokens`, and names the write counter `cache_creation_tokens`.**
- Code: `if "cache_creation_input_tokens" in _usage and _usage["cache_creation_input_tokens"] is not None: cache_creation_input_tokens = _usage["cache_creation_input_tokens"]; prompt_tokens += cache_creation_input_tokens` and the same for `cache_read_input_tokens` (`litellm/llms/anthropic/chat/transformation.py`, `calculate_usage`, lines 3186 to 3192, line breaks shown as ";").
- Code: `prompt_tokens_details: Final = PromptTokensDetailsWrapper(cached_tokens=cache_read_input_tokens, cache_creation_tokens=cache_creation_input_tokens, cache_creation_token_details=cache_creation_token_details, text_tokens=raw_input_tokens,)` (lines 3218 to 3223).
- Source: C-s44. Basis: Filled (LiteLLM).

**C74. agentgateway does the same sum when it answers a chat-completions caller from an Anthropic reply.**
- Code: `let input_tokens = accounting.input_tokens.saturating_add(accounting.cache_read_input_tokens.unwrap_or_default()).saturating_add(accounting.cache_creation_input_tokens.unwrap_or_default());` then `prompt_tokens: input_tokens as u32,` under the comment "// Convert usage from Anthropic format to OpenAI format" (`crates/llm/src/conversion/messages.rs`, lines 1002 to 1010).
- Its Responses usage type holds `cached_tokens`, `cache_write_tokens`, and `cache_write_tokens_details` (`crates/llm/src/types/responses.rs`, lines 718 to 723).
- Source: C-s45 at commit `888badbbe91f2a57ec8e7bf5a5ef3acda34b1fe5` (2026-11-20), read from the clone made by another subagent on this project. Basis: Filled (agentgateway). Also in workspace: the 2026-11-17 report (C-s47) says agentgateway and OpenRouter both use `cache_write_tokens`.

**C75. LiteLLM's client for OpenRouter shows what a third party believes OpenRouter accepts.**
- It keeps `cache_control` only for model names containing one of five strings: `CLAUDE = "claude"`, `GEMINI = "gemini"`, `MINIMAX = "minimax"`, `GLM = "glm"`, `ZAI = "z-ai"`, and strips the marker for all other models (`litellm/llms/openrouter/chat/transformation.py`, lines 60 to 72 and 119 to 128).
- Docstring: "Move cache_control from message level to content blocks. OpenRouter requires cache_control to be inside content blocks, not at message level. To avoid exceeding Anthropic's limit of 4 cache breakpoints, cache_control is only added to the LAST content block in each message." (lines 131 to 137)
- Code: `# ALWAYS add usage parameter to get cost data from OpenRouter` then `if "usage" not in response: response["usage"] = {"include": True}` (lines 193 to 196). OpenRouter now documents this parameter as deprecated with no effect (C33).
- Source: C-s44. Basis: OR-inferred (inferred from client code that special-cases OpenRouter). Confidence: low to medium. The model list is narrower than OpenRouter's own page, which also names Alibaba Qwen models and OpenAI GPT-5.6 and later.

**C78. agentgateway does the reverse subtraction when it answers a `/messages` caller from an OpenAI-shaped reply.**
- Code: `input_tokens: usage.as_ref().map(|u| u.prompt_tokens as usize).unwrap_or(0).saturating_sub(cache_creation_input_tokens.unwrap_or(0)).saturating_sub(cache_read_input_tokens.unwrap_or(0)),` where the two cache counts come from `prompt_tokens_details.cache_write_tokens` and `prompt_tokens_details.cached_tokens` (`crates/llm/src/conversion/completions.rs`, lines 193 to 225, line breaks removed).
- Source: C-s45. Basis: Filled (agentgateway). It shows one way to keep Anthropic's rule (C63) true on a Messages-shaped route when the upstream reports an inclusive total. It does not show what OpenRouter does on `/messages` for a non-Anthropic model (C37).

### I. Fees, and the local survey claims

**C76. No inference markup, 5.5 percent on credit purchases, 5 percent on BYOK above an allowance (LC15 holds).**
- Quote: "We pass through the pricing of the underlying providers; there is no markup on inference pricing (however we do charge a [fee](https://openrouter.ai/docs/faq.md#pricing-and-fees) when purchasing credits)." (source line wraps joined)
- Quote: "OpenRouter charges a 5.5% (\$0.80 minimum) fee when you purchase credits. We pass through the pricing of the underlying model providers without any markup, so you pay the same rate as you would directly with the provider." and "Crypto payments are charged a fee of 5%."
- Quote: "Yes. BYOK has a plan-dependent free allowance measured by list-price inference cost, not request count. Pay-as-you-go includes \$25,000 per month with no BYOK fee, while Enterprise allowances are custom. Usage above the allowance has a fee of 5% of what the same model and provider would normally cost on OpenRouter. This fee is deducted from your OpenRouter credits."
- Quote (BYOK page): "The cost of using custom provider keys on OpenRouter is **5% of what the same model/provider would cost normally on OpenRouter** and will be deducted from your OpenRouter credits."
- Quote (Broadcast, BYOK accounting): "`byok_usage_inference` | Reference inference cost for BYOK, not an additional OpenRouter charge or your provider's invoice amount."
- Sources: C-s10, C-s9, C-s15. Basis: OR-documented. Also in workspace: the 2026-10-28 survey (C-s46), section on aggregator economics.

**C77. Status of the four local survey claims in this topic.**
- LC7 (translation both ways, TTLs dropped, negative discount on Anthropic writes): holds. Refinements: the translation is chosen by the serving model and provider, not by the route called ("when routed to a supporting OpenAI model"); a `prompt_cache_breakpoint` has had its own `ttl` since 2026-10-21 and that TTL is dropped too; the "discount field" is `cache_discount` on `GET /generation` only. Evidence: C19, C20, C22, C51.
- LC12 (final usage chunk has non-empty `choices`): holds, word for word. Evidence: C38.
- LC15 (no markup, 5.5 percent on credits, 5 percent on BYOK above an allowance): holds. Added detail: a $0.80 minimum on the credit fee, 5 percent on crypto payments, and a pay-as-you-go allowance of $25,000 per month. Evidence: C76.
- LC17 (`prompt_tokens_details.cache_write_tokens` is the name for cache writes): holds. Refinement: OpenAI's Responses API and Moonshot's API now use the same name natively, so calling it an OpenRouter extension name is out of date. Evidence: C42, C64, C69.
- The claims come from the 2026-10-28 survey (C-s46). Basis: OR-documented. Each line names the items that hold the evidence.

### J. Inferences about OpenRouter's behavior

Each item here is an inference, kept apart from the documented facts it rests on.

**C79. OpenRouter does not expose OpenAI's `prompt_cache_retention`.**
- Inferred from: the field's absence from every request schema in the spec (C5), set against OpenAI's live page, which keeps the parameter for models before GPT-5.6 (C64).
- Source: C-s30. Basis: OR-inferred. Confidence: medium. The chat route ignores unknown fields without an error (C23), so a silent pass-through to OpenAI cannot be ruled out from the spec alone.

**C80. The chat route accepts `cache_control` in more positions than its spec lists, at least without an error.**
- Inferred from: the official SDK, which sends the marker on image, video, audio, and file parts and at message level on assistant and tool messages (C17), and the private-endpoint text that names "top-level, message, and tool cache_control markers" (C16).
- Source: C-s32, C-s30. Basis: OR-inferred. Confidence: medium that the positions are accepted, low on whether each one takes effect. LiteLLM's client holds the opposite belief about message-level markers (C75).

**C81. `provider.require_parameters` cannot select providers by cache-marker support.**
- Inferred from: the catalog's `supported_parameters` values, which hold no cache-related name (C18), and the rule that `require_parameters` filters on listed parameters (C23).
- Source: C-s31, C-s6. Basis: OR-inferred. Confidence: medium.

**C82. A `cache_control` marker sent to a model or provider without cache support is dropped silently.**
- Inferred from: the general rule that an unsupported parameter "is ignored" (C23), the Gemini text that calls extra breakpoints "safe" (C11), and the statement that the OpenAI-style markers are stripped for providers that do not support them (C21). No page states the outcome for `cache_control` by name.
- Source: C-s12, C-s1, C-s2. Basis: OR-inferred. Confidence: medium.

**C83. A provider switch costs one uncached prompt, plus a write charge where the vendor bills writes.**
- Inferred from: the pricing rules. After a switch the new provider has no entry for the prefix, so the next request pays the full input price, plus the write price where one applies (1.25x on Anthropic 5-minute writes and on OpenAI GPT-5.6 and later, per C2 and C6). No page puts a number on a switch. The docs give the latency side only (C32).
- Source: C-s1. Basis: OR-inferred. Confidence: high on the mechanism.

**C84. For Anthropic-served requests OpenRouter adds Anthropic's separate cache counters into `prompt_tokens`.**
- Inferred from: the official SDK's single formula for every model, `noCache: promptTokens - cacheReadTokens` (C41); the doc example with `prompt_tokens` 10339 and `cached_tokens` 10318 (C41); Anthropic's native rule that `input_tokens` excludes cached tokens (C63); and two open gateways that do this sum (C73, C74). No OpenRouter page states the sum in words.
- Source: C-s32, C-s1. Basis: OR-inferred. Confidence: high.

**C85. The generation API's `tokens_prompt` and `tokens_completion` are OpenRouter's own estimates, and the `native_*` fields are the provider's counts.**
- Inferred from: the Broadcast page's statement about `openrouter_generation.tokens_prompt` and `tokens_completion` (C52), linked to the API fields by the shared names, and the API descriptions "as reported by provider" on the `native_*` fields (C50).
- Source: C-s15, C-s30. Basis: OR-inferred. Confidence: medium.

## Conflicts and disagreements

Conflicts inside OpenRouter's own material:

1. **How long a sticky pin lasts.** The prompt caching page says "Sticky sessions expire after **60 minutes** of inactivity" (C25). The latency page says "OpenRouter stores a 10-minute best-effort pin" (C29). Both pages are current. I could not tell which is right. The 60-minute figure sits on the page that defines the feature.
2. **Whether `prompt_cache_breakpoint` has a TTL.** The Responses note says it "carries no `ttl`" (C8). The explicit caching section, the spec, and the changelog of 2026-10-21 say it has one (C3, C20, C22). The newer statement is right, and the note is a leftover.
3. **Cache price multipliers on the prompt caching page against the catalog.** The page gives Anthropic reads as 0.1x, OpenAI reads as "0.25x or 0.50x", Gemini reads as 0.25x, DeepSeek reads as 0.1x. The catalog gives 0.05 for Claude Sonnet 5.5 and Opus 5.5, 0.1 for GPT-5.2 and GPT-5.6, 0.5 for GPT-4o, and 0.1 for Gemini 2.5 (C58). Anthropic's and OpenAI's own pages agree with the catalog (C62, C64). The catalog is the better source for a price. The page's multipliers are stale for newer models.
4. **`completion_tokens_details.image_tokens`.** The API overview page lists it (C35). The spec's `ChatUsage` lists `accepted_prediction_tokens` and `rejected_prediction_tokens` in that object and leaves `image_tokens` out (C34). The Broadcast page lists `completionTokensDetails.imageTokens`, and the generation record has `native_tokens_completion_images` (C50). The field most likely exists on the wire and the spec is behind. I did not see a live response.
5. **Where `cache_control` may sit on the chat route.** The spec lists three places: top level, text parts, function tools (C7). The official SDK also puts it on image, video, audio, and file parts and at message level on assistant and tool messages (C17). LiteLLM's client says "OpenRouter requires cache_control to be inside content blocks, not at message level" (C75). The private-endpoint text speaks of "top-level, message, and tool cache_control markers" (C16). The safe reading is the spec's three places. The other positions are accepted by the server without an error at the least, since the official SDK sends them, but whether they take effect is not written down.
6. **`prompt_cache_options.mode` values.** OpenRouter's spec has enum `["explicit"]` only (C3). OpenAI documents `explicit` and `implicit` (C64), and Moonshot documents `implicit` only (C69). So `mode: "implicit"` is a legal vendor value that OpenRouter's schema does not list.
7. **`upstream_inference_cost` for non-BYOK requests.** The usage accounting page says the generation endpoint fills it for BYOK only (C53). A captured stream chunk shows it filled in the response `usage` of a non-BYOK request (C38, C43). These are two different surfaces, so both can be true, but the difference is easy to miss.

Conflicts between OpenRouter's pages and the vendors' live pages:

8. **Gemini implicit caching minimums.** OpenRouter: "1024 tokens for Gemini 2.5 Flash and 4096 tokens for Gemini 2.5 Pro" (C10). Google: 2,048 for both, and 4,096 for the Gemini 3 and 4 models listed (C65). Google's table is the native rule.
9. **Gemini explicit caching on Gemini 4.** Google says explicit caching "is not supported in the Interactions API, or for Gemini 4 models" (C65). OpenRouter's prompt caching page describes `cache_control`-driven Gemini caching without naming that limit, and its Gemini 4 guide says only that it "is skipped on continuation hops" (C15). What OpenRouter does with a `cache_control` marker on a Gemini 4 model is not stated.
10. **Moonshot.** OpenRouter: cache writes "no cost", automated, and `prompt_cache_options` "stays OpenAI-only" (C12, C19). Moonshot: cache writes are a separate billed item, with a `5m` or `1h` TTL chosen through `prompt_cache_options` (C69). Moonshot prices the 5-minute write at the plain input price, so the default case costs the same. The 1-hour option is not reachable through OpenRouter as documented.
11. **Groq.** OpenRouter: "Currently available on Kimi K2 models" (C12). Groq: supported "only" on three GPT-OSS models (C68). Groq's page is the native rule.
12. **OpenAI's 12-hour TTL and the chat route.** OpenAI ties `"12h"` to the Responses API ("2.5× for `"12h"` in the Responses API", C64). OpenRouter shows a `"ttl": "12h"` breakpoint on a Chat Completions request and says "the Responses API takes the same fields" (C3). Either OpenRouter sends chat requests to OpenAI through the Responses API, or the 12-hour value is dropped or rejected upstream on the chat route. Not resolved.

Where my findings differ from the local Runvil documents:

13. **LC7 wording.** The 2026-10-28 survey says `cache_control` becomes `prompt_cache_breakpoint` "on the OpenAI route". The current text ties the conversion to the model that serves the request ("when routed to a supporting OpenAI model"), not to the route the caller used. The survey also predates by one week the change that gave `prompt_cache_breakpoint` its own `ttl` (C22). The claim itself holds.
14. **LC17 and the 2026-11-17 report's "extension name".** The report calls `prompt_tokens_details.cache_write_tokens` "an extension name that agentgateway and OpenRouter both use". OpenAI now reports `input_tokens_details.cache_write_tokens` on its Responses API, and Moonshot reports `prompt_tokens_details.cache_write_tokens` on its chat API (C64, C69). The name is turning into a shared convention. The report is right that it is absent from OpenAI's classic chat-completions usage as far as OpenAI's caching page shows.
15. No finding contradicts LC12 or LC15.

## Open questions blocked by closed source

1. **How does OpenRouter find a Gemini named cache again on the next request, and where does it keep that state?** Tried: the prompt caching page, the spec, the SDK source, the changelog. The page says only that the caller does "not need to manage cache names or TTL explicitly". Fill found: LiteLLM (C72) hashes the marked prefix plus tools and model, stores the hash as the cache's `displayName`, and lists the vendor's caches to find it, so the state lives at the vendor.
2. **How is `prompt_tokens` built for an Anthropic-served request?** Tried: usage accounting page, spec, Broadcast page. The inclusive shape is documented through the SDK formula (C41). The sum itself is not stated. Fill found: LiteLLM (C73) and agentgateway (C74) both add `cache_read_input_tokens` and `cache_creation_input_tokens` to `input_tokens`.
3. **On `/messages` with a non-Anthropic model, does `input_tokens` leave out cached tokens the way Anthropic's does?** Tried: the spec, the Messages examples in the docs (C37, which show `cache_read_input_tokens: 0`, so they do not decide it). Fill found: agentgateway (C78) subtracts reads and writes from `prompt_tokens` when it answers in Messages shape.
4. **Is the caller's `prompt_cache_key` forwarded to OpenAI, replaced by a derived key, or dropped? Is `x-grok-conv-id` set for xAI?** Tried: the prompt caching page, the spec, provider-specific header section. Partial evidence: for private endpoints the spec says OpenRouter sends "an OpenRouter-derived, customer-isolated prompt_cache_key, not the caller-supplied key" (C16), and for Z.AI it sends a derived "session affinity key" (C12). No fill found for the public OpenAI and xAI endpoints. xAI's own page (C67) states what a gateway has to send.
5. **What happens to `cache_control` on a provider with no cache support, and to more than four markers on Anthropic?** Tried: prompt caching page, parameters page, provider selection page. Only the general "parameter is ignored" rule exists (C23). The page says "There is a limit of four explicit breakpoints" and does not say whether OpenRouter trims extra markers or passes Anthropic's 400 through. Fill found (Workspace, 2026-11-17 report, failure table row 8): the native error text is "A maximum of 4 blocks with cache_control may be provided. Found 5.", and the report's advice from Bifrost and opencode is to give breakpoints one owner and count before sending. LiteLLM's OpenRouter client (C75) puts a message-level marker on the last block only, for the same reason.
6. **What does a `cache_control` on a function tool become toward OpenAI?** The translation note names "a text block" only (C19). OpenAI allows `prompt_cache_breakpoint` on content blocks inside input messages and says "`additional_tools` input items do not currently accept `prompt_cache_breakpoint`". No fill found. The likely outcome is that the tool marker is dropped, since OpenAI's implicit breakpoint already covers tools as part of the prefix.
7. **Does a top-level `cache_control` do anything for non-Anthropic models?** The page lists Anthropic providers only (C6). No fill found. Moonshot's own Anthropic-shaped route honors only the top-level form (C69), which shows the question is real for other vendors.
8. **How are `cache_discount` and `native_tokens_cached` computed per vendor?** Tried: the generation reference, the Broadcast page. The spec gives one description for all vendors (C50, C52). No fill found. Not searched further, since it is an accounting detail of OpenRouter's own ledger.
9. **Where is the sticky pin stored, what hash is used, and which of the two lifetimes is real?** Tried: prompt caching page, latency page, provider selection page, routers pages. No fill found. Vendor pages show what the pin protects: OpenAI and xAI caches live on single machines (C64, C67).

## Unverified

- Nothing in this file was tested against a live model, so every statement about behavior rests on documentation, spec, or source code.
- The exact shape of a usage object for a real cached Anthropic, Gemini, or DeepSeek request through OpenRouter was not seen. The examples quoted are from OpenRouter's docs.
- OpenRouter's public pricing page (`https://openrouter.ai/pricing`) was not fetched. The fee figures come from the FAQ and the BYOK page.
- AWS Bedrock's own caching page was not fetched. The statement that Bedrock rejects a top-level `cache_control` rests on Anthropic's page (C62) and OpenRouter's page (C6).
- The Mistral caching page was located (`https://docs.mistral.ai/inference/conversations/advanced/prompt-caching.md`) and not read. OpenRouter's prompt caching page has no Mistral section, while 15 of 22 `mistralai/` catalog entries have an `input_cache_read` price.
- From memory, not checked today: OpenAI's classic chat-completions usage reports `prompt_tokens_details.cached_tokens`, and a stream there sends usage only with `stream_options.include_usage` in a last chunk with empty `choices`. The local survey says the same and OpenRouter's streaming page says the same of "OpenAI's spec" (C38). I did not open OpenAI's API reference for it.
- Whether OpenAI reports cache writes on its chat-completions route under `prompt_tokens_details.cache_write_tokens`. OpenAI's caching page shows the field under `input_tokens_details` only (C64).

## Workspace documents used

- C-s46: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took: claims LC7, LC12, LC15, LC17, and section 5.5's table of four caching designs with the Gemini minimums. Re-checked live: yes, all four claims against OpenRouter's pages and spec, and the Anthropic, OpenAI, and Gemini rules against the vendors' pages. All hold, with the refinements in C77.
- C-s47: Cabinet output dated 2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build". Took: rule 6 (4 breakpoints, 20-block lookback, minimums, TTL, prices, top-level field), the usage mapping table (`input_tokens` + `cache_read_input_tokens` + `cache_creation_input_tokens` to `prompt_tokens`), the failure table row for a fifth breakpoint, and the note that the Runvil Gateway's stream parser drops the usage-only chunk and that `Usage` has no field for cache writes. Re-checked live: the Anthropic rules, yes (C62, C63). The Runvil code references were not re-read.
- C-s50: `runvil-design`, `decisions/models-and-conversations.md` (last commit 2026-11-13). Took: the standing decision that prefix changes must be explicit, in the owner's words "Because of prefix cache invalidation, changes to the prefix of the messages should be visible and explicit", and "A request that repeats the first messages of the previous request can use the provider's prompt cache." Not a claim about OpenRouter, so no live re-check applies.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| C-s1 | https://openrouter.ai/docs/guides/best-practices/prompt-caching (re-fetched live, identical to the mirror) | 2026-11-22 | OpenRouter doc |
| C-s2 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-5-6 | 2026-11-22 | OpenRouter doc |
| C-s3 | https://openrouter.ai/docs/guides/features/response-caching | 2026-11-22 | OpenRouter doc |
| C-s4 | https://openrouter.ai/docs/cookbook/administration/usage-accounting (re-fetched live, identical to the mirror) | 2026-11-22 | OpenRouter doc |
| C-s5 | https://openrouter.ai/docs/api/api-reference/generations/get-request-&-usage-metadata-for-a-generation | 2026-11-22 | OpenRouter doc |
| C-s6 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| C-s7 | https://openrouter.ai/docs/guides/features/zero-completion-insurance | 2026-11-22 | OpenRouter doc |
| C-s8 | https://openrouter.ai/docs/guides/features/service-tiers | 2026-11-22 | OpenRouter doc |
| C-s9 | https://openrouter.ai/docs/guides/overview/auth/byok | 2026-11-22 | OpenRouter doc |
| C-s10 | https://openrouter.ai/docs/faq | 2026-11-22 | OpenRouter doc |
| C-s11 | https://openrouter.ai/docs/api_reference/streaming (re-fetched live, identical to the mirror) | 2026-11-22 | OpenRouter doc |
| C-s12 | https://openrouter.ai/docs/api_reference/overview | 2026-11-22 | OpenRouter doc |
| C-s13 | https://openrouter.ai/docs/guides/best-practices/latency-and-performance | 2026-11-22 | OpenRouter doc |
| C-s14 | https://openrouter.ai/docs/guides/routing/auto-exacto | 2026-11-22 | OpenRouter doc |
| C-s15 | https://openrouter.ai/docs/guides/features/broadcast and https://openrouter.ai/docs/guides/features/broadcast/s3 | 2026-11-22 | OpenRouter doc |
| C-s16 | https://openrouter.ai/docs/guides/overview/models | 2026-11-22 | OpenRouter doc |
| C-s17 | https://openrouter.ai/docs/guides/providers/integration | 2026-11-22 | OpenRouter doc |
| C-s18 | https://openrouter.ai/docs/changelog (entries dated 2026-10-21 and 2026-11-05) | 2026-11-22 | OpenRouter doc |
| C-s19 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5 | 2026-11-22 | OpenRouter doc |
| C-s20 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/fable-5-1 | 2026-11-22 | OpenRouter doc |
| C-s21 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gemini-4 | 2026-11-22 | OpenRouter doc |
| C-s22 | https://openrouter.ai/docs/guides/best-practices/reasoning-tokens | 2026-11-22 | OpenRouter doc |
| C-s23 | https://openrouter.ai/docs/guides/features/zdr | 2026-11-22 | OpenRouter doc |
| C-s24 | https://openrouter.ai/docs/guides/routing/routers/auto-router | 2026-11-22 | OpenRouter doc |
| C-s25 | https://openrouter.ai/docs/guides/features/server-tools/tool-search | 2026-11-22 | OpenRouter doc |
| C-s26 | https://openrouter.ai/docs/guides/features/server-tools/bash | 2026-11-22 | OpenRouter doc |
| C-s27 | https://openrouter.ai/docs/guides/community/tanstack-ai | 2026-11-22 | OpenRouter doc |
| C-s28 | https://openrouter.ai/docs/guides/providers/traffic-routing | 2026-11-22 | OpenRouter doc |
| C-s29 | https://openrouter.ai/docs/cookbook/administration/analytics-cost-control | 2026-11-22 | OpenRouter doc |
| C-s30 | https://openrouter.ai/openapi.json (schemas `ChatUsage`, `CostDetails`, `Usage`, `OpenAIResponsesUsage`, `AnthropicUsage`, `ChatStreamChunk`, `ChatStreamOptions`, `ChatRequest`, `ResponsesRequest`, `MessagesRequest`, `ChatContentText`, `InputText`, `AnthropicCacheControlDirective`, `AnthropicCacheControlTtl`, `PromptCacheBreakpoint`, `PromptCacheOptions`, `PrivateEndpointPromptCaching`, `GenerationResponse`, `ProviderResponse`, `PublicPricing`, `PublicEndpoint`, `DeferredToolsDetails`, `ServerToolUseDetails`) | 2026-11-22 | OpenRouter spec |
| C-s31 | https://openrouter.ai/api/v1/models (476 models, pricing keys tallied with Python) | 2026-11-22 | OpenRouter spec |
| C-s32 | https://github.com/OpenRouterTeam/ai-sdk-provider at `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20), `src/chat/convert-to-openrouter-chat-messages.ts`, `src/chat/index.ts`, `src/utils/compute-token-usage.ts` | 2026-11-22 | OpenRouter SDK source |
| C-s33 | https://platform.claude.com/docs/en/build-with-claude/prompt-caching (fetched as `.md`) | 2026-11-22 | vendor doc |
| C-s34 | https://developers.openai.com/api/docs/guides/prompt-caching (fetched as `.md`) | 2026-11-22 | vendor doc |
| C-s35 | https://ai.google.dev/gemini-api/docs/caching (fetched as `.md.txt`, Interactions API view) | 2026-11-22 | vendor doc |
| C-s36 | https://ai.google.dev/gemini-api/docs/caching?generate-content=true (fetched as `.md.txt`) | 2026-11-22 | vendor doc |
| C-s37 | https://ai.google.dev/api/caching (fetched as `.md.txt`) | 2026-11-22 | vendor doc |
| C-s38 | https://api-docs.deepseek.com/guides/kv_cache | 2026-11-22 | vendor doc |
| C-s39 | https://docs.x.ai/developers/advanced-api-usage/prompt-caching and its subpages `/how-it-works`, `/maximizing-cache-hits`, `/usage-and-pricing` (fetched as `.md`) | 2026-11-22 | vendor doc |
| C-s40 | https://console.groq.com/docs/prompt-caching (fetched as `.md`) | 2026-11-22 | vendor doc |
| C-s41 | https://platform.moonshot.ai/docs/guide/use-context-caching-feature-of-kimi-api | 2026-11-22 | vendor doc |
| C-s42 | https://www.alibabacloud.com/help/en/model-studio/context-cache (page dated Nov 21, 2026) | 2026-11-22 | vendor doc |
| C-s43 | https://docs.z.ai/guides/capabilities/cache (fetched as `.md`) | 2026-11-22 | vendor doc |
| C-s44 | https://github.com/BerriAI/litellm at `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22), `litellm/llms/vertex_ai/context_caching/transformation.py`, `litellm/llms/vertex_ai/context_caching/vertex_ai_context_caching.py`, `litellm/llms/anthropic/chat/transformation.py`, `litellm/llms/openrouter/chat/transformation.py` | 2026-11-22 | gateway source |
| C-s45 | https://github.com/agentgateway/agentgateway at `888badbbe91f2a57ec8e7bf5a5ef3acda34b1fe5` (2026-11-20), `crates/llm/src/conversion/messages.rs`, `crates/llm/src/conversion/completions.rs`, `crates/llm/src/types/responses.rs` | 2026-11-22 | gateway source |
| C-s46 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28) | 2026-11-22 | workspace document |
| C-s47 | Cabinet output dated 2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build" | 2026-11-22 | workspace document |
| C-s48 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/opus-5 | 2026-11-22 | OpenRouter doc |
| C-s49 | https://openrouter.ai/docs/api_reference/responses/reasoning | 2026-11-22 | OpenRouter doc |
| C-s50 | `runvil-design`, `decisions/models-and-conversations.md` (last commit 2026-11-13) | 2026-11-22 | workspace document |
| C-s51 | https://openrouter.ai/docs/guides/routing/model-variants/overview (endpoint listing example) | 2026-11-22 | OpenRouter doc |
