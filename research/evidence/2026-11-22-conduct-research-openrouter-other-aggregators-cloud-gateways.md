---
produced: 2026-11-22
title: "OpenRouter deep dive - other aggregators and cloud gateways (Vercel AI Gateway, Cloudflare AI Gateway, Hugging Face Inference Providers, Amazon Bedrock Converse, Vertex AI, Azure AI Foundry, Anthropic and Gemini OpenAI-compatible layers, LLM Gateway, Requesty, Groq, Cerebras, Together, Fireworks) - how they expose reasoning replay, signatures, prompt caching, server tools, stop reasons, and provider options, which gateways adopted OpenRouter's extension names (reasoning, reasoning_details, native_finish_reason, provider, models, usage.cost, cache_write_tokens, cache_control, annotations, plugins, transforms) and which names compete (reasoning_content, thinking_blocks, reasoning_signature, extra_content.google.thought_signature, reasoning_opaque), and the Open Responses standard"
---

# OpenRouter deep dive: other aggregators and cloud gateways as a second view of the translation problem, adoption of OpenRouter's conventions, and standardization efforts

- What was read: the live public documentation of the hosted gateways and cloud wrappers named in the task (Vercel AI Gateway, Cloudflare AI Gateway, Hugging Face Inference Providers, Amazon Bedrock, Google Vertex AI, Azure AI Foundry), the two first-party compatibility layers (Anthropic, Gemini), OpenAI's own chat-completions reference, inference platforms and local servers that define reasoning fields (Groq, Cerebras, Together, Fireworks, DeepSeek, vLLM, llama.cpp, Ollama), about fifteen smaller gateways and resellers found while tracing field names, the Open Responses specification and its repository at a pinned commit, the MCP sampling page, and one Agent Client Protocol proposal. The 2026-10-28 workspace survey's sections on cloud wrappers, aggregators, and compatibility layers were read first and re-checked.
- How many sources: 94 source entries (see Sources), of which 3 are workspace documents and 31 were read as page excerpts returned by the search tool. 63 evidence items (Q1 to Q63). 24 candidate matrix rows. 15 rows in the adoption table.
- Read date: 2026-11-22 for every source. Nothing in this file describes OpenRouter's own behavior. Every item is labeled `Filled` with the product name, or `Vendor-documented` where a vendor's own native rule is quoted.

## Candidate matrix rows

The column "How OpenRouter exposes it" is left to the OpenRouter evidence files, as instructed. The column "What OpenRouter loses or flattens" is used here for how the other gateways expose the feature and what they lose. None of it is a statement about OpenRouter.

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens (here: how other gateways expose it and what they lose) | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Signed reasoning text | Anthropic (direct, Bedrock, Vertex, Foundry) | Thinking block with `signature` must return complete and unmodified during tool use, or the call errors | see OpenRouter evidence | Vercel and LLM Gateway: `reasoning_details` entry `reasoning.text` with `text` and `signature`. Bedrock Converse: `reasoningContent.reasoningText` with `text` and `signature`. LiteLLM: `thinking_blocks`. Requesty and agentgateway: one `reasoning_signature` string per message. Prisme.ai: native blocks under `reasoning_details` plus `reasoning_model`. Anthropic's own OpenAI layer returns no thinking at all | Caller holds state. Data must round-trip unchanged. Block position matters: LLM Gateway added `block_index` to restore it. Scoped to the issuing provider: LLM Gateway drops the blocks when the provider changes, Prisme.ai forwards only when the model tag matches | Filled (Vercel AI Gateway, LLM Gateway, Amazon Bedrock, LiteLLM, Requesty, agentgateway, Prisme.ai) | Q3, Q19, Q29, Q31, Q32, Q39, Q49, Q50 |
| Redacted or encrypted reasoning | Anthropic (`redacted_thinking`), OpenAI (`encrypted_content`) | Opaque payload, replayed as received. OpenAI rejects a modified payload with `invalid_encrypted_content` | see OpenRouter evidence | Vercel and LLM Gateway: `reasoning.encrypted` entry with `data`, `id`, `format`. Bedrock Converse: `redactedContent` (base64 binary) as the other arm of a union. Open Responses: `encrypted_content` on a `reasoning` item, opted in with `include: ["reasoning.encrypted_content"]` | Caller holds state. Opaque round trip. No vendor values for the caller to choose. Works on chat completions only through an extension field | Filled (Vercel AI Gateway, LLM Gateway, Amazon Bedrock, Open Responses) | Q3, Q19, Q31, Q54 |
| Thought signature on tool calls | Google Gemini 3 and 4 | First function call of each step must return with its signature or the request gets a 400. Parallel calls must not be interleaved with results | see OpenRouter evidence | Google's own OpenAI route: `tool_calls[].extra_content.google.thought_signature`. LLM Gateway accepts that path and also uses `reasoning_details` entries with `format: "google-gemini-v1"` and `google_part` for text turns. GoModel: `extra_content` keyed by vendor. Requesty: `reasoning_signature` | Caller holds state. Round trip attached to the specific tool call, not the message. Escape value `skip_thought_signature_validator` exists for foreign history. A message-level list cannot express it without a per-call key | Vendor-documented (Google), Filled (LLM Gateway, GoModel, Requesty) | Q25, Q26, Q31, Q32, Q51 |
| Plain reasoning text field | DeepSeek, Qwen, Kimi, GLM, MiniMax, open models on vLLM and llama.cpp | Text beside `content`. DeepSeek returns 400 if it is missing on replay when `tools` is present | see OpenRouter evidence | `reasoning`: vLLM, Groq, Cerebras, Together, Vercel, LLM Gateway. `reasoning_content`: DeepSeek, llama.cpp, Fireworks, Requesty, LiteLLM, agentgateway, MiniMax. Bifrost and Infron emit both | Caller holds state. Round trip of plain text. Two competing names, so a reader must accept both | Filled (see evidence) | Q33, Q34, Q35, Q37, Q38 |
| Reasoning control | OpenAI (effort), Anthropic (adaptive effort, or budget on older models), Google (level or budget) | Effort names differ per model. Claude 4.7 and later reject budgets. Gemini cannot turn thinking off on some models | see OpenRouter evidence | Vercel: `reasoning` object (`enabled`, `effort`, `max_tokens`, `exclude`) mapped per target, with a published table. LLM Gateway and Infron: `reasoning.effort` and `reasoning.max_tokens`. Requesty: budget as a numeric string in `reasoning_effort`. Gemini route: `reasoning_effort` or `extra_body.google.thinking_config`, never both. Fireworks: `reasoning_effort` or Anthropic-shaped `thinking`. Bedrock Converse: vendor values in `additionalModelRequestFields` | Gateway can translate a neutral effort. Budgets do not translate cleanly: effort toward OpenAI is approximate, budget toward new Claude is rejected. LLM Gateway errors on an unsupported budget, Azure drops unsupported parameters unless `require_parameters` is set | Filled (Vercel AI Gateway, LLM Gateway, Infron, Requesty, Fireworks, Amazon Bedrock), Vendor-documented (Google) | Q4, Q20, Q27, Q28, Q31, Q32, Q35, Q40 |
| Reasoning visibility and summaries | Anthropic (`display`), OpenAI (`summary`), Google (`include_thoughts`), Groq and Cerebras (`reasoning_format`) | Text may be omitted while reasoning still runs and is billed | see OpenRouter evidence | Vercel: `reasoning.exclude`, and `thinking.display: 'summarized'` through provider options. Gemini route: `include_thoughts` under `extra_body.google`. Groq and Cerebras: `reasoning_format` of `parsed`, `raw`, `hidden`. Vercel extracts `<think>` tags only when not streaming | Gateway only for hide and show. Vendor-specific values to request summaries. Empty text does not prove reasoning was off | Filled (Vercel AI Gateway, Groq, Cerebras), Vendor-documented (Google) | Q4, Q5, Q9, Q27, Q33 |
| Explicit cache breakpoints | Anthropic (`cache_control`), OpenAI GPT-5.6 and later (`prompt_cache_breakpoint`), Alibaba, MiniMax | Marker on a content block. Up to 4 per request. Minimum prefix per model | see OpenRouter evidence | Vercel: `caching: 'auto'` places markers for the caller, or manual `cache_control`. LiteLLM: injects and translates `cache_control` to `prompt_cache_breakpoint`. Bedrock Converse: separate `cachePoint` block, and it cannot express OpenAI's breakpoints at all. Hugging Face: automatic only. Anthropic's OpenAI layer: unsupported | Gateway can place markers by position (last message, message before the last user message). Caller picks nothing vendor-specific in that mode. Manual markers ride inside the message body | Filled (Vercel AI Gateway, LiteLLM, Amazon Bedrock, Hugging Face), Vendor-documented (Anthropic, OpenAI) | Q7, Q18, Q22, Q23, Q29, Q30, Q46 |
| Cache lifetime (TTL) | Anthropic 5m or 1h, OpenAI 30m or 12h | Per marker. Higher write price for the longer lifetime | see OpenRouter evidence | Vercel: `cache_ttl` of `5m` or `1h` on the Responses route only, advisory, with an `unsupported` warning in `provider_metadata.gateway.warnings`. LiteLLM: `control.ttl` honored toward Anthropic and ignored toward OpenAI. Bedrock: `ttl` on `cachePoint`, and a `ValidationException` on models without 1h | Vendor-specific values. No neutral unit exists across families. Gateways drop or warn, they do not convert | Filled (Vercel AI Gateway, LiteLLM, Amazon Bedrock) | Q7, Q22, Q46 |
| Cache usage accounting | Anthropic (disjoint counters), OpenAI (inclusive total with sub-counts) | Anthropic input excludes cached tokens. OpenAI input includes them | see OpenRouter evidence | OpenAI and LLM Gateway: `prompt_tokens_details.cached_tokens` and `cache_write_tokens`. Bedrock Converse: `cacheReadInputTokens`, `cacheWriteInputTokens`, `cacheDetails` per TTL, with `inputTokens` excluding both. Vercel chat route and Hugging Face: only `cached_tokens`. Anthropic's OpenAI layer: token details always empty | Gateway only. Normalization needed in both directions or totals are wrong. Per-TTL write breakdown exists at OpenAI and Bedrock and has no chat-completions slot at the gateways read | Vendor-documented (OpenAI), Filled (LLM Gateway, Amazon Bedrock, Vercel AI Gateway, Hugging Face) | Q7, Q18, Q22, Q23, Q29, Q30, Q45 |
| Cache affinity across providers | Any routed model with per-provider caches | A cache only helps if the next turn reaches the same provider | see OpenRouter evidence | Hugging Face: `X-HF-Session-Id` header with a UUID. Azure model router: session affinity (preview). Open Responses: `prompt_cache_key` | Gateway holds the affinity. Caller supplies one neutral id | Filled (Hugging Face, Azure AI Foundry, Open Responses) | Q18, Q28, Q55 |
| Named cache resources | Google Gemini (`cachedContents`) | Cache created ahead of time and named in the request | see OpenRouter evidence | Gemini and Vertex OpenAI routes: `extra_body.google.cached_content` with a `cachedContents/...` handle | Server holds state. Caller passes a vendor-specific handle. Does not map to breakpoint markers | Vendor-documented (Google) | Q25, Q27 |
| Vendor server tools (web search) | Anthropic, OpenAI, xAI, Google, Alibaba | Each vendor has its own tool entry or flag and its own result blocks | see OpenRouter evidence | Cloudflare: native shape on the vendor's own route only (`web_search_20250305` on Messages, `web_search_preview` on Responses, `enable_search` flag). Gemini grounding is unreachable through the unified tool. Bedrock OpenAI routes on `bedrock-runtime`: server tools unavailable. Vertex chat route: `web_search_options` maps to GoogleSearch with no sub-options. Experiential: runs its own search and returns `url_citation` annotations | Cloudflare's answer is a bypass route per vendor. Results in native blocks need a native route. A gateway-run search can be normalized to OpenAI's `annotations` | Filled (Cloudflare AI Gateway, Amazon Bedrock, Experiential), Vendor-documented (Google) | Q8, Q13, Q24, Q25, Q47 |
| Vendor-specific option pass-through | All | Parameters outside the common set | see OpenRouter evidence | Bedrock Converse: `additionalModelRequestFields` in, `additionalModelResponseFieldPaths` out by JSON Pointer. Vercel: `providerOptions.<provider>`, which overrides the neutral `reasoning` field. Google: `extra_body.google` and `extra_content.google`. Cerebras: `extra_body` | Caller must know the vendor to use it. Every wrapper read has such a field. Vercel documents that it silently wins over the neutral setting | Filled (Amazon Bedrock, Vercel AI Gateway, Cerebras), Vendor-documented (Google) | Q4, Q20, Q25, Q33 |
| Stop and finish reasons | All | Vendor-specific enumerations | see OpenRouter evidence | Bedrock Converse: one closed set of nine values, vendor value available only by JSON Pointer. Vercel generation route: AI SDK values such as `content-filter`, `tool-calls`. Only OpenRouter clones emit `native_finish_reason` | Gateway only. No shared name for the raw value outside OpenRouter's clones | Filled (Amazon Bedrock, Vercel AI Gateway, ModelGates, InteractiveAI) | Q10, Q21, Q41 |
| Errors | All | Status plus vendor code. Errors can arrive after a 200 | see OpenRouter evidence | Vercel: OpenAI-shaped `error` object, 402 for balance or budget. Kilo: turns an upstream 402 into 503. LLM Gateway: forwards upstream errors such as `invalid_encrypted_content` unchanged. Anthropic's OpenAI layer: same format, messages not equivalent. Bedrock: typed exceptions. Open Responses: five error types and a `response.failed` event | Gateway only. Caller sees a class and sometimes the vendor code | Filled (Vercel AI Gateway, Kilo Gateway, LLM Gateway, Amazon Bedrock, Open Responses), Vendor-documented (Anthropic) | Q11, Q21, Q29, Q31, Q57, Q62 |
| Provider routing preferences | Aggregators | Which upstream serves a model | see OpenRouter evidence | Vercel: `providerOptions.gateway` (`order`, `sort`, `byok`, timeouts), `provider.sort` shorthand. Hugging Face: model-id suffix (`:fastest`, `:cheapest`, `:novita`). Azure: `routing_config`. Cloudflare: routes configured on the gateway. Infron and NanoGPT: OpenRouter's `provider` object | Gateway only. A caller that sets these names a provider, which is vendor knowledge by another route | Filled (Vercel AI Gateway, Hugging Face, Azure AI Foundry, Cloudflare AI Gateway, Infron, NanoGPT) | Q6, Q12, Q16, Q28, Q42 |
| Fallback model list | Aggregators, Anthropic | Try another model when the first fails | see OpenRouter evidence | Vercel: top-level `models` array or `providerOptions.gateway.models`. NanoGPT: `models`, up to 4. Anthropic native: `fallbacks`, unavailable on Foundry | Gateway only. Response `model` names the model that answered, which decides whether stored reasoning can be replayed | Filled (Vercel AI Gateway, NanoGPT), Vendor-documented (Anthropic) | Q6, Q28, Q43 |
| Cost reporting | Aggregators | Money charged for a request | see OpenRouter evidence | LLM Gateway and Sargalay: `usage.cost` and `usage.cost_details`. RouterPlus and Prisme.ai: `usage.cost`. Vercel: `GET /v1/generation?id=` and `providerMetadata` | Gateway only. Unit is not guaranteed (Sargalay reports points) | Filled (LLM Gateway, Sargalay, RouterPlus, Prisme.ai, Vercel AI Gateway) | Q10, Q44 |
| System message placement | Anthropic | One top-level `system` field. Mid-conversation system messages are a separate native feature | see OpenRouter evidence | Anthropic's OpenAI layer hoists and concatenates every system and developer message with a newline, wherever it appeared | Gateway only, and lossy: position is discarded | Vendor-documented (Anthropic) | Q29 |
| Structured output strictness | Anthropic, Google | Schema-constrained output is a native feature with its own limits | see OpenRouter evidence | Anthropic's OpenAI layer: only `json_schema` with `strict: true`, anything else is a 400, and `strict` on tools is ignored. Vertex chat route: `json_schema` without fully recursive schemas. Each Vercel route uses a different field (`response_format`, `output_config.format`, `text.format`) | Gateway only. The guarantee differs per upstream and the caller is not told | Vendor-documented (Anthropic, Google), Filled (Vercel AI Gateway) | Q9, Q25, Q29 |
| Unsupported parameter handling | All | Vendors differ on reject versus ignore | see OpenRouter evidence | Ignored silently: Vertex chat route, Gemini route, most fields on Anthropic's layer, Cerebras for some models. Dropped by rule: Azure model router, unless `routing_config.require_parameters` is true. Warned in the reply: Vercel `provider_metadata.gateway.warnings`, Experiential `x-experiential-ignored-parameters`. Rejected: LLM Gateway for an unsupported `reasoning.max_tokens`, Groq for `raw` with tools | Gateway only. Three reporting styles exist: silent, warning in the reply, and hard error | Filled (Azure AI Foundry, Vercel AI Gateway, LLM Gateway, Groq, Cerebras, Experiential), Vendor-documented (Google, Anthropic) | Q7, Q25, Q27, Q28, Q29, Q31, Q33, Q63 |
| Server-held conversation state | OpenAI Responses | `previous_response_id` with `store` | see OpenRouter evidence | Bedrock: stored 30 days by default, `model` required on every request on `bedrock-runtime`. Azure model router: only on the Responses format. Open Responses: server MUST load prior input and output | Server holds state. Exists only on the Responses-shaped route | Filled (Amazon Bedrock, Azure AI Foundry, Open Responses) | Q24, Q28, Q57 |
| Native-format bypass routes | Anthropic, OpenAI | The vendor's own wire format | see OpenRouter evidence | Vercel, Cloudflare, Hugging Face, Azure, Bedrock, LLM Gateway, Requesty, and RouterPlus all serve an Anthropic Messages route next to chat completions. Vercel and Azure tell callers to use it for full Claude features. Cloudflare has deprecated its single unified endpoint for one-model calls | Every large gateway read ended up with a second native route. Features with native-only shapes (server tool result blocks, thinking display, cache markers) are documented there | Filled (Vercel AI Gateway, Cloudflare AI Gateway, Hugging Face, Azure AI Foundry, Amazon Bedrock, LLM Gateway, Requesty, RouterPlus) | Q1, Q8, Q12, Q17, Q24, Q28, Q31, Q32, Q44 |
| A neutral standard for reasoning replay | Open Responses project | `reasoning` item with `summary`, `content`, and opaque `encrypted_content` | see OpenRouter evidence | Covers the OpenAI pattern. Signatures, per-call signatures, cache markers, cost, and routing are left to implementer extensions, which use a `slug:` prefix for new types and optional fields on existing schemas | Standard exists only for the Responses shape. For chat completions the names spread by copying | Filled (Open Responses) | Q53, Q54, Q55, Q56, Q61 |

## Adoption table for OpenRouter's conventions (Part 2)

Counts are of parties other than OpenRouter for which a source was read today. "Clone" marks products whose documentation reproduces OpenRouter's schema. A count is a floor: other adopters may exist.

| OpenRouter convention | Adopters with the same name and meaning | Competing names | Verdict today | Evidence |
|---|---|---|---|---|
| `reasoning` string on `message` and `delta` | 9: vLLM (renamed from `reasoning_content`), Vercel AI Gateway, Groq, Cerebras, Together, LLM Gateway, Infron, Bifrost, Sargalay | `reasoning_content` (11 parties), `reasoning_text` (GitHub Copilot), `thinking` (Ollama native API), `<think>` tags in content | Shared convention, but one of two. A reader must accept `reasoning` and `reasoning_content` | Q37, Q38 |
| `reasoning_details` array on `message` and `delta` | 5 with the same entry shape: Vercel AI Gateway, LLM Gateway, Bifrost, Infron, MiniMax (a model vendor). 1 with a different entry shape under the same name: Prisme.ai | `thinking_blocks` (LiteLLM), `reasoning_signature` (agentgateway, Requesty), `extra_content` (Google, GoModel), `reasoning_opaque` (GitHub Copilot), `encrypted_content` items (Responses, Open Responses) | The most adopted structured form for chat completions, and the only one copied by a large hosted gateway. Not uniform: LLM Gateway adds `block_index` and `google_part`, Prisme.ai changes the entries | Q3, Q31, Q39 |
| Entry types `reasoning.text`, `reasoning.summary`, `reasoning.encrypted`, and `format` tags | `reasoning.text` and `reasoning.encrypted`: Vercel, LLM Gateway, Bifrost. `reasoning.summary`: Vercel. `reasoning.text` with `format: "MiniMax-response-v1"`: MiniMax. Format tags `anthropic-claude-v1`, `openai-responses-v1`, `google-gemini-v1`: Vercel, LLM Gateway | Anthropic block types `thinking` and `redacted_thinking` (Prisme.ai, LiteLLM) | Shared among the adopters of `reasoning_details` | Q3, Q31, Q39 |
| `reasoning` request object with `effort` and `max_tokens` | `effort`: Vercel AI Gateway, LLM Gateway, Infron, Ollama. `max_tokens`: Vercel, LLM Gateway, Infron. `enabled`: Vercel, Together. `exclude`: Vercel | `reasoning_effort` (OpenAI chat standard), `thinking` object (Anthropic, DeepSeek, Fireworks), `extra_body.google.thinking_config`, `reasoning_format`, `output_config.effort` | Shared among aggregators. `reasoning.effort` is also OpenAI's own name on the Responses route. `reasoning.max_tokens` is an aggregator convention with 3 adopters | Q4, Q28, Q40 |
| `native_finish_reason` | 2 clones: ModelGates, InteractiveAI. Sargalay strips it | Bedrock's JSON Pointer to the vendor field. No other name found | One product's invention. No independent adopter found | Q41 |
| `provider` preferences object | 3 with OpenRouter's member names: Infron, NanoGPT, ModelGates (clone). Partial: Vercel (`provider.sort` only) | `providerOptions.gateway` (Vercel), model-id suffix (Hugging Face), `routing_config` (Azure), gateway-side routes (Cloudflare) | Convention among API-compatible resellers only. The large platforms each use their own shape | Q6, Q16, Q28, Q42 |
| `models` fallback array | 2: Vercel AI Gateway, NanoGPT | `fallbacks` (Anthropic native), `providerOptions.gateway.models` (Vercel alternative) | Small shared convention between two gateways | Q6, Q43 |
| `usage.cost` | 6: LLM Gateway, Sargalay, RouterPlus, Prisme.ai, ModelGates, Experiential | Separate generation lookup (Vercel), `providerMetadata` | Shared convention among small and mid-size gateways. Unit is not uniform | Q10, Q44 |
| `usage.cost_details` | 2: LLM Gateway, Sargalay (both with `upstream_inference_cost` names) | none found | Thin adoption | Q44 |
| `prompt_tokens_details.cache_write_tokens` | OpenAI (first party), LLM Gateway, Amazon Bedrock for OpenAI models | `cache_creation_input_tokens` (Anthropic), `cacheWriteInputTokens` (Bedrock Converse), `cache_write_input_tokens` (Prisme.ai) | Inside the standard's own namespace today | Q23, Q30, Q45 |
| `cache_control` on content parts in a chat-completions body | 4 read directly: Vercel AI Gateway, Infron, LiteLLM, Alibaba (through forwarding gateways). LiteLLM's list adds MiniMax, Z.ai, Databricks | `prompt_cache_breakpoint` (OpenAI standard), `cachePoint` (Bedrock Converse), `cached_content` (Gemini) | De facto convention for Anthropic-style caching outside the Messages API. It now sits beside a standard OpenAI marker with a different name | Q7, Q30, Q46 |
| `annotations` with `url_citation` | OpenAI (first party), Open Responses schema, Experiential | Anthropic `web_search_tool_result` blocks, `citations` arrays at search vendors (not read today) | Standard, not an extension | Q30, Q47 |
| `plugins` | 1 clone (ModelGates). 2 compatibility shims: NanoGPT accepts and ignores, Experiential serves only `web` | `web_search_options` (OpenAI standard), vendor server tools, `:online` suffix | One product's invention | Q48 |
| `transforms` | 0. NanoGPT accepts and ignores | none | One product's invention, and OpenRouter's own plugin page now presents context compression as a plugin | Q48 |
| Generation lookup route with `native_tokens_*` fields (not on the task list, found in passing) | Vercel AI Gateway: `GET /v1/generation?id=` with `native_tokens_prompt`, `native_tokens_completion`, `native_tokens_reasoning`, `native_tokens_cached`, `upstream_inference_cost`, `is_byok` | none found | Copied by one large gateway | Q10 |

## Evidence

Every item in this file describes a product other than OpenRouter. The basis label `Filled (<product>)` means "this is how that product does it". It never means "this is how OpenRouter does it". Read dates are 2026-11-22 unless a line says otherwise. Page dates are the publisher's own "last updated" or `dateModified` stamps where the page has one.

### Part 1A. Vercel AI Gateway

**Q1. Vercel AI Gateway is live and serves four request formats at one host: Chat Completions, Anthropic Messages, OpenAI Responses, and "OpenResponses".**

- "The Chat Completions API implements the same specification as the OpenAI Chat Completions API." Base URL `https://ai-gateway.vercel.sh/v1`. Endpoints listed: `GET /models`, `GET /models/{model}`, `POST /chat/completions`, `POST /embeddings`. (Q-s1)
- "The Anthropic Messages API implements the same specification as the [Anthropic Messages API]". Base URL `https://ai-gateway.vercel.sh`. Endpoints: `POST /v1/messages`, `POST /v1/messages/count_tokens`, and five `/v1/messages/batches` routes. "Deleting batches isn't supported through AI Gateway." (Q-s8)
- "[Responses] and [OpenResponses] share `/v1/responses`." (Q-s5)
- Vercel's own advice on which to pick: "**Using Claude Code or the Anthropic SDK?** Use the Anthropic Messages API for native feature support." and "**Want a provider-agnostic REST API?** Use OpenResponses." (Q-s5)
- Model ids are `creator/model`, for example `anthropic/claude-opus-5`, the same id grammar OpenRouter uses. (Q-s1)
- Basis: Filled (Vercel AI Gateway). Page dates 2026-09-08 to 2026-10-07.

**Q2. Vercel's chat-completions route returns reasoning text in `message.reasoning` and `delta.reasoning`, the same field name OpenRouter uses.**

- "When the provider returns reasoning content, AI Gateway includes it in the response" with the example `"message": { "role": "assistant", "content": "...", "reasoning": "Let me think about this carefully. ..." }`. (Q-s2)
- "Reasoning content is streamed incrementally in the `delta.reasoning` field". (Q-s2)
- Basis: Filled (Vercel AI Gateway). Page date 2026-09-08.

**Q3. Vercel's chat-completions route returns a `reasoning_details` array with the same three entry types and the same `format` tags that OpenRouter documents. This is the one hosted gateway found that copies OpenRouter's structured reasoning shape field for field.**

- "When `reasoning.enabled` is `true` (or when `reasoning.exclude` is not set), responses include a `reasoning_details` array alongside the standard `reasoning` text field. This structured field captures cryptographic signatures, encrypted content, and other verification data that providers include with their reasoning output." (Q-s2)
- Entry types, quoted: "`'reasoning.text'`: Contains the actual reasoning content as plain text in the `text` field. May include a `signature` field (Anthropic models) for cryptographic verification." "`'reasoning.encrypted'`: Contains encrypted or redacted reasoning content in the `data` field. Used by OpenAI models when reasoning is protected, or by Anthropic models when thinking is redacted." "`'reasoning.summary'`: Contains a condensed version of the reasoning process in the `summary` field." (Q-s2)
- Other entry fields, quoted: "`id` (optional)", "`format`: Provider format identifier, such as `'openai-responses-v1'`, `'anthropic-claude-v1'`, `'google-gemini-v1'`, `'xai-responses-v1'`, or `'unknown'`", "`index` (optional): Position in the reasoning sequence". (Q-s2)
- Streaming: "When streaming, reasoning details are delivered incrementally in `delta.reasoning_details`". The Anthropic example chunk has `"type": "reasoning.text"`, `"text"`, `"signature"`, `"format": "anthropic-claude-v1"`, `"index": 0`. (Q-s2)
- What the page claims the array is for: "The AI Gateway preserves reasoning details from models across interactions, normalizing the different formats used by OpenAI, Anthropic, and other providers into a consistent structure. This allows you to switch between models without rewriting your conversation management logic. This is particularly useful during tool calling workflows where the model needs to resume its thought process after receiving tool results." (Q-s2)
- What the page leaves unsaid: it gives no rule for sending `reasoning_details` back (no "unmodified", no ordering rule, no error on a missing entry), and no example of an assistant message in a request that contains the array. The replay rule on this route is therefore undocumented by Vercel.
- Basis: Filled (Vercel AI Gateway). Page date 2026-09-08. Confidence: high on the field names (quoted), low on replay behavior (the page is silent).

**Q4. Vercel's chat-completions route takes a `reasoning` request object with `enabled`, `effort`, `max_tokens`, and `exclude`, the same four members OpenRouter documents, and treats `reasoning_effort` as an alias.**

- Table rows, quoted: "`enabled` | boolean | Request reasoning on or off. Some models require reasoning and can't disable it. This doesn't guarantee visible reasoning text." "`effort` | string | The request schema accepts `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, or `max`. ... Can't be combined with `max_tokens`." "`max_tokens` | number | Request a thinking-token budget. Can't be combined with `effort`." "`exclude` | boolean | When `true`, omit reasoning content from the response. It doesn't turn off internal reasoning." (Q-s2)
- "The `reasoning` object is a Gateway extension to the OpenAI Chat Completions API. Python clients send it through `extra_body`". "the standard `reasoning_effort` field is an alias for `reasoning.effort`. If both are present, the nested value takes precedence." (Q-s2)
- Translation is the gateway's job: "The `reasoning` object works across supported reasoning models, including models from other providers. AI Gateway maps it to the target provider's native reasoning configuration." (Q-s2)
- The mapping table (Q-s6), quoted in part: "Anthropic Claude 4.7 and later | Adaptive thinking `effort` | AI Gateway sets adaptive thinking. `minimal` maps to `low`. Other enabled effort levels, including `xhigh` and `max`, pass through." "Anthropic Claude 4.5 and earlier | `budgetTokens` | Converted to a thinking budget sized as a share of the maximum output tokens." "Google Gemini 3 and later | `thinkingLevel` | `low` maps to `low`; all other effort levels map to `high`." "Google Gemini 2.5 | `thinkingBudget` | Fixed budgets: `none` sets `thinkingBudget: 0`; `minimal` uses the model-specific minimum; `low` uses 1,024 tokens; `medium` uses 8,192; and `high`, `xhigh`, or `max` uses 24,576."
- Budget shares for budget-based Claude models: `minimal` ~10%, `low` ~20%, `medium` ~50%, `high` ~80%, `xhigh` ~80% of the request's maximum output tokens, "clamped between 1,024 and 64,000 tokens". (Q-s6)
- Budgets going the other way are lossy: "For OpenAI reasoning models, AI Gateway translates a token-budget request to a named effort level; this doesn't enforce an exact reasoning-token cap. Claude 4.7 and later don't accept token-budget configurations." (Q-s6)
- A neutral setting can fail on a newer model: "Legacy `reasoning.max_tokens` requests use fixed-budget thinking, which newer adaptive-only Claude models reject. Use shared effort for those models." (Q-s2)
- Visible text is not guaranteed: "On Claude Opus 4.7 and later, reasoning text can be omitted even when thinking runs. Missing reasoning text doesn't establish that the model ignored effort. Check usage when available, or request `thinking.display: 'summarized'` through provider options." (Q-s2)
- Precedence rule: "The top-level `reasoning` option and reasoning-related provider options are never merged. If you set a reasoning-related option in `providerOptions` (like `reasoningEffort`, `thinking`, or `thinkingConfig`), it takes full precedence and the top-level `reasoning` value is ignored". (Q-s6)
- Basis: Filled (Vercel AI Gateway). Page date 2026-09-08.

**Q5. Vercel extracts `<think>` tags only for non-streaming replies. In streams the tags stay in the content.**

- "Automatic extraction: For models that don't natively support reasoning output, the gateway automatically extracts reasoning from `<think>` tags in non-streaming responses. In streaming responses, the tags stay in `delta.content`." (Q-s2, tag name read from the Markdown source of the page)
- Basis: Filled (Vercel AI Gateway).

**Q6. Vercel's provider controls live under a camelCase `providerOptions.gateway` object. A top-level `provider` object exists only as a shorthand for `sort`. A top-level `models` fallback array has the same name and meaning as OpenRouter's.**

- "You can control which providers are used and in what order through the `providerOptions` parameter." Example: `"providerOptions": { "gateway": { "order": ["vertex", "anthropic"] } }`. (Q-s3)
- "`provider` (object): Shorthand for provider routing. Supports `sort` (`'cost'`, `'ttft'`, `'tps'`). Equivalent to setting `providerOptions.gateway.sort`." and "The `provider` shorthand supports `sort` only. ... If you set both, they must resolve to the same value or the request fails." (Q-s3, Q-s4)
- Model fallbacks: "The simplest way is to use the `models` field directly at the top level of your request", example `"model": "openai/gpt-6-astra", "models": ["anthropic/claude-opus-5", "google/gemini-3.6-flash"]`, and "Alternatively, you can specify model fallbacks through the `providerOptions.gateway.models` field". "Both configurations will: 1. Try the primary model ... first". The response `model` field names the model used. (Q-s3)
- Other members of `providerOptions.gateway` seen: `order`, `sort`, `models`, `providerTimeouts`, `byok`, `caching`, `safetyIdentifier`. (Q-s3, Q-s4)
- The same `providerOptions` object is accepted on the Messages and Responses routes: the format table lists "AI Gateway options: `providerOptions.gateway` in the JSON body" for Chat Completions, Messages API, and Responses / OpenResponses. (Q-s5)
- Basis: Filled (Vercel AI Gateway). Page dates 2026-09-08 to 2026-10-07.

**Q7. Vercel prompt caching: a gateway-level `caching: 'auto'` switch places Anthropic-style markers for the caller, and manual `cache_control` is accepted on the chat-completions route. Cache TTL is settable only on the Responses route and is advisory.**

- "When you set `caching: 'auto'` and the request routes to a provider that requires explicit cache markers, AI Gateway adds `cache_control` breakpoints to your messages. This covers Anthropic, MiniMax, and Alibaba. AI Gateway uses the cache markers expected by each provider. For providers with implicit caching (OpenAI, Google, DeepSeek), no modification is needed and caching works automatically." (Q-s7)
- Where the automatic markers go: "On the last message." and "On the message before the last user message (falling back to the system message)." (Q-s7)
- "**Default behavior**: When `caching` is not set, AI Gateway passes your request through without modification. ... For Anthropic, you'll need to set `caching: 'auto'` or manually add cache markers to your messages." (Q-s7)
- Manual markers on chat completions: "For fine-grained control, you can manually mark content with `cache_control`", with the example placing `"cache_control": { "type": "ephemeral" }` as a sibling of `role` and `content` on a message object. "Prompt caching works with Anthropic models across all supported providers (Anthropic, Vertex AI, and Bedrock)." (Q-s3)
- TTL: "you can request a one-hour lifetime with the `cache_ttl` field on the Responses API". "The field accepts `5m` (five minutes) or `1h` (one hour)." "`cache_ttl` is advisory. AI Gateway uses the default lifetime for unsupported values and never rejects the request because of the hint. The response includes an `unsupported` warning in `provider_metadata.gateway.warnings`" with `"type": "unsupported", "feature": "cache_ttl", "details": "Unsupported cache_ttl; the hint was ignored."` (Q-s7)
- Cache usage fields per route, from the page's own examples: chat completions reads `response.usage.prompt_tokens_details.cached_tokens`; Messages reads `response.usage.cache_read_input_tokens` and shows `cache_creation_input_tokens`; Responses reads `response.usage.input_tokens_details.cached_tokens`. (Q-s7, Q-s8) No cache-write token field is shown for the chat-completions route.
- "The AI SDK does not return a separate cache-hit boolean. ... A zero count can mean a cache miss, an ineligible prompt, or unavailable provider usage details." (Q-s7)
- Basis: Filled (Vercel AI Gateway). Page date 2026-09-11.

**Q8. Vercel's native Messages route keeps Anthropic's own shapes for thinking, caching, and web search, and that is where Vercel sends callers who need the full feature set.**

- Thinking reply on the Messages route: `{"type": "thinking", "thinking": "...", "signature": "anthropic-signature-xyz"}` followed by a `text` block. Controls: `thinking.type` `'adaptive'` or `'enabled'`, `budget_tokens`, `display`, and `output_config.effort`. (Q-s8)
- "The gateway passes through the `cache_control` parameter to Anthropic's prompt caching feature." "The `cache_control` parameter is passed through to **Anthropic**, **Vertex AI Anthropic**, and **Amazon Bedrock Anthropic** models for explicit caching." (Q-s8)
- Server web search on the Messages route is Anthropic's own tool entry (`"type": "web_search_20250305", "name": "web_search"`) and the reply has `web_search_tool_result` blocks. (Q-s8)
- Claude Code setting documented for cross-provider routing: "`CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS` | Strips Anthropic-specific `anthropic-beta` headers and beta tool-schema fields from requests. Set to `1` when routing through providers like Bedrock or Vertex AI that reject those fields." (Q-s8)
- Basis: Filled (Vercel AI Gateway). Page dates 2026-09-08 and 2026-09-18.

**Q9. Vercel's Responses and OpenResponses route returns reasoning as an opaque item, and Vercel warns that a successful reply does not prove an optional setting took effect.**

- "The raw chain of thought isn't returned. A `reasoning` item may carry an `encrypted_content` field rather than readable text, so treat it as an opaque value to pass back rather than something to display. Use `summary: "auto"` to request readable text on models that support summaries. An empty summary doesn't prove that reasoning was disabled." (Q-s9)
- "[Reasoning], [search tools], and [cache controls] have model-specific and API-specific options. A successful generation alone doesn't establish that an optional setting took effect. Inspect usage and provider metadata when verifying routing, caching, service tiers, or compliance settings." (Q-s5)
- "Structured output uses `output` in AI SDK 7, ... `response_format` in Chat Completions, `output_config.format` in Messages, and `text.format` in Responses. Tool definitions and tool-result messages also differ. Follow the linked examples in the capability table instead of copying a payload between formats." (Q-s5)
- Basis: Filled (Vercel AI Gateway).

**Q10. Vercel reports cost through a separate lookup route, `GET /v1/generation?id=`, whose field names match OpenRouter's generation route. No `usage.cost` field is documented on the chat-completions reply.**

- "Returns detailed information about a specific generation, including cost, latency, and token usage. Much of this data is also returned in `providerMetadata` on the chat completion response." (Q-s10)
- Sample fields: `total_cost`, `market_cost`, `surcharge_cost`, `gateway_cost`, `upstream_inference_cost`, `usage`, `is_byok`, `provider_name`, `streamed`, `finish_reason`, `latency`, `generation_time`, `tokens_prompt`, `tokens_completion`, `native_tokens_prompt`, `native_tokens_completion`, `native_tokens_reasoning`, `native_tokens_cached`, `native_tokens_cache_creation`, `billable_web_search_calls`. (Q-s10)
- "`finish_reason`: Why the generation ended (for example, `stop`, `length`, `content-filter`, `tool-calls`)". These are the AI SDK's hyphenated values, not OpenAI's underscore values. (Q-s10)
- "Generation IDs are returned on every chat completion response as the `id` field, and on streaming responses are injected into the first content chunk". (Q-s10)
- The reasoning page's response examples show `usage` with only `prompt_tokens`, `completion_tokens`, `total_tokens`, and `completion_tokens_details.reasoning_tokens`. (Q-s2) A search of the 23 downloaded Vercel pages found no `native_finish_reason`, no `usage.cost`, and no `cost_details`.
- Basis: Filled (Vercel AI Gateway). Page date 2026-09-11. Confidence: medium on the absence claim (23 pages searched, the streaming and usage sub-pages were not all fetched).

**Q11. Vercel's error shape and status codes on the chat-completions route.**

- Codes listed: `400`, `401`, `402 Payment Required: No positive credit balance, or a budget reached its limit`, `403`, `404`, `429`, `500`. Body: `{"error": {"message": "...", "type": "invalid_request_error", "param": "model", "code": "invalid_request_error"}}`. (Q-s1)
- Basis: Filled (Vercel AI Gateway).

### Part 1B. Cloudflare AI Gateway

**Q12. Cloudflare has deprecated its one-URL OpenAI-compatible endpoint for single-model calls and now offers four endpoints side by side, one per wire format. It routes by format instead of translating everything into one.**

- On the `/compat/chat/completions` page: "Deprecated for single-model calls. For standard single-model chat completions, this endpoint is deprecated. Use the REST API instead, which provides OpenAI-compatible endpoints at `api.cloudflare.com/client/v4/accounts/{ACCOUNT_ID}/ai/v1/chat/completions`. The `/compat/chat/completions` endpoint will continue to work for existing integrations." (Q-s11, page `dateModified` 2026-11-04)
- The four REST endpoints: "`POST /ai/run` | Envelope with `model`, `input` | All models and modalities (LLM, image, TTS, ASR)", "`POST /ai/v1/chat/completions` | OpenAI chat completions", "`POST /ai/v1/responses` | OpenAI Responses API", "`POST /ai/v1/messages` | Anthropic Messages API". (Q-s12, `dateModified` 2026-10-09)
- "The `/ai/v1/messages` endpoint strictly uses Anthropic's API schema and supports routing to Anthropic and other third-party models. Workers AI models (`@cf/`) do not support this schema." (Q-s12)
- "Claude Code sends requests using the Anthropic Messages API. It cannot use the OpenAI-compatible `/ai/v1/chat/completions` endpoint or the universal `/ai/run` endpoint directly." (Q-s12)
- A third family still exists: provider-native pass-through URLs such as `https://gateway.ai.cloudflare.com/v1/{account_id}/{gateway_id}/anthropic/v1/messages`. (Q-s11a) For Bedrock's Anthropic route Cloudflare states: "AI Gateway passes native Anthropic requests, responses, and server-sent events through without translation." (Q-s11b)
- The older "Universal Endpoint" page is titled "Universal Endpoint (Deprecated)". (Q-s11b)
- Model ids are `author/model` (`openai/gpt-4.1`, `anthropic/claude-sonnet-4`). (Q-s12)
- Basis: Filled (Cloudflare AI Gateway).

**Q13. Cloudflare exposes vendor server tools only in each vendor's own shape on that vendor's own route. There is no translated tool on the chat-completions route.**

- Table: "Anthropic | `POST /ai/v1/messages` | `tools: [{ "type": "web_search_20250305", "name": "web_search", "max_uses": N }]`", "OpenAI | `POST /ai/v1/responses` | `tools: [{ "type": "web_search_preview" }]`", "xAI | `POST /ai/v1/responses` | `tools: [{ "type": "web_search" }]`", "Alibaba | `POST /ai/v1/chat/completions` | top-level `"enable_search": true`". (Q-s13, `dateModified` 2026-06-26)
- "How you enable web search depends on the provider. Activation is either a tool entry on a `tools` array or a top-level flag on the request body." (Q-s13)
- "OpenAI web search is available only on the Responses API endpoint (`POST /ai/v1/responses`). The `/ai/v1/chat/completions` endpoint does not accept the `web_search_preview` tool." (Q-s13)
- "Search invocations and results appear in the response as `server_tool_use` and `web_search_tool_result` content blocks." (Q-s13)
- What is lost, stated by Cloudflare: "**Google Gemini** — not available through the unified `web_search` tool, because Vertex's OpenAI-compatible surface does not translate it into Gemini's native `googleSearch` tool. To use Gemini grounding, pass the native `google_search` tool to the provider-specific Vertex endpoint." (Q-s13, the long dash is in the original)
- Basis: Filled (Cloudflare AI Gateway).

**Q14. Cloudflare's documentation leaves the translation of reasoning, signatures, cache markers, and finish reasons on its chat-completions route undescribed.**

- A search of the 982 KB `llms-full.txt` export of the AI Gateway documentation for `cache_control`, `cached_tokens`, `signature`, `finish_reason`, `native_finish`, and `reasoning_details` found only the SigV4 authentication text and model-catalog blurbs. (Q-s11b)
- Basis: Filled (Cloudflare AI Gateway), negative finding. Confidence: medium (one export searched by keyword).

### Part 1C. Hugging Face Inference Providers

**Q15. Since 2026-11-16 Hugging Face routes language-model requests through OpenRouter. The 2026-10-28 survey's description of Hugging Face as an independent "reshaper" is out of date.**

- Hugging Face blog, "Published November 16, 2026": "Today, we are switching Inference Providers from our own custom router to OpenRouter's. Requests made to language models now go through OpenRouter, which picks the best provider for each call based on live price, speed and reliability, and retries on another provider if the first one fails. Your code stays the same." (Q-s18)
- Changelog, "Nov 17, 26": "Inference Providers now route language models through OpenRouter. ... Custom provider keys no longer work for language models." (Q-s18a)
- Docs index: "Routing is powered by OpenRouter, the unified interface for LLMs trusted by millions". (Q-s19)
- Providers page: "Chat, vision and agent requests go through OpenRouter, which routes each one to a provider serving the model". (Q-s15)
- Basis: Filled (Hugging Face Inference Providers).

**Q16. Hugging Face keeps provider selection as a suffix on the model id and exposes no request field for it.**

- "Append a suffix to the model id. It works in every client, including tools that only let you set a model name." Table: no suffix is "Default routing: the best mix of price, speed and reliability"; `:fastest` is "Highest throughput, in tokens per second"; `:cheapest` is "Lowest price"; `:novita` is "Only Novita, or an error if it doesn't serve the model." (Q-s14)
- "If a provider fails, OpenRouter retries the request on another provider serving the model. A pinned provider (`:novita`) is never replaced." (Q-s14)
- Account-level control replaces per-request allow and deny lists: "In your Inference Providers settings, you can turn off the providers you don't want. The router never sends your requests to them, and pinning one with a suffix returns an error." (Q-s14)
- Basis: Filled (Hugging Face Inference Providers).

**Q17. Hugging Face's router now serves three chat formats, not one.**

- Blog: "The router supports OpenAI's Chat Completions and Responses APIs and Anthropic's Messages API for every chat model, whatever the provider behind it." (Q-s18)
- Messages guide: "The Messages API (from Anthropic) lets you call Inference Providers with the Anthropic SDKs. It works with every chat model, whatever the provider serving it, and it's the API Claude Code uses." "point it at `https://router.huggingface.co`. Pass your token as `auth_token` ...: the router expects an `Authorization: Bearer` header and doesn't accept `x-api-key`." "`max_tokens` is required. Reasoning models return a `thinking` block before the text and spend part of `max_tokens` on it". (Q-s17)
- The docs index still has the older sentence: "**Note**: This OpenAI-compatible endpoint is currently available for chat completion tasks only." (Q-s19) In context it means the endpoint does not serve image, embedding, or speech tasks.
- The Responses guide is titled "Responses API (beta)" and documents `reasoning={"effort": "low" | "medium" | "high"}`. (Q-s17a)
- Basis: Filled (Hugging Face Inference Providers).

**Q18. Hugging Face documents caching as automatic only, reports it in `prompt_tokens_details.cached_tokens`, and keeps a conversation on one provider through a header.**

- "Caching is automatic: there is nothing to turn on. The response tells you how many input tokens came from the cache" with `"prompt_tokens_details": { "cached_tokens": 11500 }`. (Q-s16)
- "A provider's cache only helps if the next turn goes to the same provider. Send a stable UUID per conversation in the `X-HF-Session-Id` header, and the router keeps it on the same provider. Other values are ignored." (Q-s16)
- The chat-completion task reference lists `reasoning_effort` ("Common values: none, minimal, low, medium, high, xhigh. Support and defaults are provider and model-dependent.") and does not list a `reasoning` object, `reasoning_details`, `cache_control`, or a `provider` object. (Q-s20)
- Basis: Filled (Hugging Face Inference Providers). Confidence: medium that OpenRouter's extension fields are accepted or returned when sent through the Hugging Face router. The Hugging Face pages do not say either way.


### Part 1D. Amazon Bedrock (Converse, and the OpenAI-compatible routes)

**Q19. Converse returns reasoning as a typed content block that is a union of signed text or encrypted bytes, and the reference tells the caller to send it back unmodified.**

- `ReasoningContentBlock`: "This data type is a UNION, so only one of the following members can be specified when used or returned." Members: "`reasoningText` : The reasoning that the model used to return the output. Type: ReasoningTextBlock object" and "`redactedContent` : The content in the reasoning that was encrypted by the model provider for safety reasons. The encryption doesn't affect the quality of responses. Type: Base64-encoded binary data object". (Q-s21)
- `ReasoningTextBlock`: "`text` ... Required: Yes" and "`signature` : A token that verifies that the reasoning text was generated by the model. If you pass a reasoning block back to the API in a multi-turn conversation, include the text and its signature unmodified. Type: String Required: No". (Q-s22)
- Streaming: `ReasoningContentBlockDelta` is a union of `redactedContent`, `signature`, and `text`, with the same "include the text and its signature unmodified" sentence on `signature`. (Q-s23)
- Bedrock's Claude guide (Messages shape on InvokeModel) gives the replay rule in prose: "During tool use, you must pass thinking blocks back to the API, and you must include the complete unmodified block back to the API." "Notice that the `thinking_block` is passed in as well as the `tool_use_block`. If this is not passed in, an error occurs." "While you can omit `thinking` blocks from prior `assistant` role turns, we suggest always passing back all thinking blocks to the API for any multi-turn conversation." (Q-s27)
- Basis: Filled (Amazon Bedrock Converse). Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28), which says the Converse signature "hashes all messages and must be replayed with all previous messages or the call errors". That stronger wording was not found on the three reference pages read today, so it stays a `Workspace` claim.

**Q20. Converse has a small fixed set of neutral inference parameters. Everything vendor-specific (including thinking controls) goes through an untyped pass-through field that the caller fills with vendor-specific values.**

- "`additionalModelRequestFields` : Additional inference parameters that the model supports, beyond the base set of inference parameters that `Converse` and `ConverseStream` support in the `inferenceConfig` field. ... Type: JSON value". (Q-s24)
- "`inferenceConfig` : Inference parameters to pass to the model. `Converse` and `ConverseStream` support a base set of inference parameters. If you need to pass additional parameters that the model supports, use the `additionalModelRequestFields` request field." (Q-s24)
- The reverse direction is also opt-in: "`additionalModelResponseFieldPaths` : Additional model parameters field paths to return in the response. `Converse` and `ConverseStream` return the requested fields as a JSON Pointer object in the `additionalModelResponseFields` field." "if the JSON Pointer is valid, but the requested field is not in the model response, it is ignored by `Converse`." Up to 10 pointers. (Q-s24)
- The reference's own example passes `top_k` this way: "the request passess a field (`top_k`) that the `Converse` field doesn't support. You pass the additional field in the `additionalModelRequestFields` field." (Q-s24, spelling as in the original)
- Other top-level request members today: `guardrailConfig`, `outputConfig`, `performanceConfig`, `projectIdentifier`, `promptVariables`, `requestMetadata`, `serviceTier`, `system`, `toolConfig`. (Q-s24)
- Basis: Filled (Amazon Bedrock Converse).

**Q21. Converse normalizes stop reasons to one closed set of nine values. The vendor's own value is not returned unless the caller asks for it by JSON Pointer.**

- "`stopReason` ... Valid Values: `end_turn | tool_use | max_tokens | stop_sequence | guardrail_intervened | content_filtered | malformed_model_output | malformed_tool_use | model_context_window_exceeded`". (Q-s24)
- The pointer example in the reference is exactly this case: `additionalModelResponseFieldPaths` of `[ "/stop_sequence" ]`. (Q-s24)
- Errors are typed exceptions, among them `ModelErrorException`, `ModelNotReadyException`, `ModelTimeoutException`, `ServiceUnavailableException`, `ThrottlingException`, `ValidationException`. (Q-s24)
- Basis: Filled (Amazon Bedrock Converse). Note the set has grown since the workspace survey's "closed normalized set": `malformed_model_output`, `malformed_tool_use`, and `model_context_window_exceeded` are present today.

**Q22. Converse cache markers are separate `cachePoint` blocks with an optional TTL, and usage reports reads and writes as disjoint counters with a per-TTL breakdown.**

- `CachePointBlock`: "`type` ... Valid Values: `default` Required: Yes" and "`ttl` : Optional TTL duration for cache entries. ... Valid Values: `5m | 1h` Required: No". (Q-s25)
- "In the Converse API, add `"ttl": "1h"` to your `cachePoint` object. In the InvokeModel API for Claude models, add `"ttl": "1h"` to your `cache_control` object. For models that support only the 5-minute TTL, omit the `ttl` field. Including the field for these models can result in a `ValidationException`." (Q-s26)
- `TokenUsage` members: `inputTokens`, `outputTokens`, `totalTokens` (all required), `cacheReadInputTokens`, `cacheWriteInputTokens`, and "`cacheDetails` : Detailed breakdown of cache writes by TTL. Empty if no cache creation occurred. Sorted by TTL duration (1h before 5m)." (Q-s28)
- "When prompt caching is enabled, the `inputTokens` field represents only the non-cached input tokens (tokens that were not read from or written to the cache). To calculate the total input tokens sent in a request, use the following formula: `total input tokens = inputTokens + cacheReadInputTokens + cacheWriteInputTokens`". (Q-s26)
- Basis: Filled (Amazon Bedrock Converse). Also in workspace: the 2026-10-28 survey, section 5.5, which lists the same six usage fields.

**Q23. Converse cannot express OpenAI's explicit cache breakpoints. For OpenAI models on Bedrock the caller must leave Converse and use the OpenAI-shaped route. This is a documented case of a unified shape that forces a bypass for one vendor's feature.**

- Supported-models table, GPT rows: "`prompt_cache_breakpoint` on `input_text` blocks (Responses API) and `text` content parts (Chat Completions and InvokeModel). Converse doesn't support explicit caching for this model." TTL "30 minutes", "4 cache writes". (Q-s26)
- The same guide describes the OpenAI usage fields for those models: "The usage object in the response includes two cache-specific fields: `cached_tokens` — Number of input tokens read from cache ... `cache_write_tokens` — Number of input tokens written to cache (billed at 1.25× the uncached input token rate)." and "In the Responses API, `input_tokens` includes tokens read from and written to cache; `cached_tokens` and `cache_write_tokens` are those parts." (Q-s26, long dashes in the original)
- So on one platform two accounting conventions coexist: Converse's disjoint counters (Q22) and OpenAI's inclusive total with sub-counts.
- Basis: Filled (Amazon Bedrock).

**Q24. Bedrock's OpenAI-compatible routes: `bedrock-runtime` is now the recommended host and `bedrock-mantle` is the compatibility host. What each loses is stated.**

- "Amazon Bedrock provides the OpenAI Responses API on both the `bedrock-runtime` and `bedrock-mantle` endpoints. Use `bedrock-runtime` for new applications. Use `bedrock-mantle` only when a model or capability that you require isn't available on `bedrock-runtime`." Base URLs `https://bedrock-runtime.{region}.amazonaws.com/openai/v1` and `https://bedrock-mantle.{region}.api.aws/v1`. (Q-s29)
- Losses on `bedrock-runtime`, quoted: "Requests are always synchronous. `background=true` is rejected with a 400 error." "`model` is required on every request, including one that supplies `previous_response_id`. This differs from the OpenAI Responses API specification". "Server-side tool use and pre-configured tools aren't available, including web search. Client-side tool use works on both endpoints." "Application inference profiles aren't supported." (Q-s29)
- "`bedrock-runtime` doesn't implement the OpenAI-compatible `GET /models` operation", and listing models there "returns `404 UnknownOperationException`". (Q-s29, Q-s30)
- State: "When `store` is `true` (the default), Amazon Bedrock retains the response, including the input and output, for 30 days." "When `store` is `false` ... The `previous_response_id` parameter cannot be used to continue the conversation." (Q-s29)
- Path quirk on mantle: "the path prefix varies by model. Most models are served under `/v1` ... but some are served under `/openai/v1` instead. A request that uses the wrong prefix for a model is rejected". (Q-s29)
- The Chat Completions page documents no reasoning field, no cache field, and no finish-reason mapping. It defers to OpenAI's documentation and adds only guardrail headers (`X-Amzn-Bedrock-GuardrailIdentifier`, `X-Amzn-Bedrock-GuardrailVersion`, `X-Amzn-Bedrock-Trace`) and the body field `amazon-bedrock-guardrailConfig`. (Q-s30)
- Basis: Filled (Amazon Bedrock). Also in workspace: the 2026-10-28 survey says "AWS runs a second host `bedrock-mantle` for compatibility only". Today's page keeps mantle as the compatibility host but also puts the OpenAI routes on `bedrock-runtime` under `/openai/v1` and recommends that host.

### Part 1E. Google Vertex AI and Azure AI Foundry serving through OpenAI-shaped routes

**Q25. Vertex's Chat Completions route is documented for Gemini and for self-deployed open models. It ignores unsupported parameters and puts Gemini-only features under a `google` key in two extension objects.**

- "The Chat Completions API supports both Gemini models and select self-deployed models from Model Garden." "Parameter support for third-party models varies by model." "If you pass any unsupported parameter, it is ignored." (Q-s31)
- "These features can still be passed in as parameters, but must be contained within an `extra_content` or `extra_body` or they will be ignored." (Q-s31)
- `extra_body.google` members: `safety_settings`, `cached_content`, `thinking_config`, `thought_tag_marker`, `stream_function_call_arguments`, `tools`, `media_resolution`. (Q-s31)
- `extra_content.google` members: "`thought` | This field explicitly marks if a field is a thought and takes precedence over `thought_tag_marker`.", "`thought_signature` | A bytes field that provides a thought signature to validate against thoughts returned by the model.", "`parts` | Specific to a Tool message to pass multi-modal function response parts back to the model." "`extra_content` lets you specify Gemini-specific content that shouldn't be ignored." (Q-s31)
- Reasoning control collision: "`reasoning_effort` ... `low`: 1024, `medium`: 8192, `high`: 24576. As no thoughts are included in the response, only one of `reasoning_effort` or `extra_body.google.thinking_config` may be specified." (Q-s31)
- Google-only enum value inside a standard field: "`tool_choice` ... `validated`: Corresponds to the mode `VALIDATED` in the `FunctionCallingConfig`. This is Google-specific." And "`web_search_options` | Corresponds to the `GoogleSearch` tool. No sub-options are supported." (Q-s31)
- Basis: Filled (Google Vertex AI, now branded Gemini Enterprise Agent Platform).

**Q26. Google's OpenAI-compatible route returns the thought signature on each tool call at `tool_calls[].extra_content.google.thought_signature`, and a request that replays the call without it gets a 400.**

- Vertex page, Chat Completions example, model response: `"tool_calls": [ { "extra_content": { "google": { "thought_signature": "<SIGNATURE_A>" } }, "function": { "arguments": "{\"flight\":\"AA100\"}", "name": "check_flight" }, "id": "function-call-1", "type": "function" } ]`. (Q-s32)
- Rule: "When constructing the next request, you must include the `part` containing the `functionCall` and its `thought_signature` exactly as it was returned by the model. For sequential (multi-step) function calling, validation is performed on all steps in the current turn, and omitting a required `thought_signature` for the first `functionCall` part in any step of the current turn results in a `400` error. A turn begins with the most recent user message that is not a `functionResponse`." (Q-s32)
- Parallel calls: "If the model returns parallel function calls (for example, `FC1+signature`, `FC2`), your response must contain all function calls followed by all function responses ... Interleaving responses (`FC1+signature`, `FR1`, `FC2`, `FR2`) results in a `400` error." (Q-s32)
- Escape hatch for foreign history: "You can set `thought_signature` to `skip_thought_signature_validator`, but, this should be a last resort as it will negatively impact model performance." (Q-s32) The Gemini API page names two accepted dummy values: "`"context_engineering_is_the_way_to_go"` or `"skip_thought_signature_validator"`". (Q-s33)
- Text replies: "Including this signature in the next request is recommended for best performance, but omitting it won't cause an error. When streaming, this signature might be returned in a part with empty text content". (Q-s32)
- The Gemini API compatibility page says only: "Gemini 3 supports OpenAI compatibility for thought signatures in chat completion APIs. You can find the full example on the thought signatures page." (Q-s34) The Markdown export of that thought-signatures page read today shows its OpenAI section with native `thoughtSignature` JSON and no `extra_content` text, so the Vertex page is the live Google source for the field path.
- Third-party confirmation of the wire shape: a pydantic-ai pull request merged 2026-10-14 says Gemini 3 and 4 on OpenAI-compatible endpoints "return each tool call's thought signature in a `tool_calls[].extra_content` vendor extension and reject the next request with a 400 unless it comes back unchanged", and a harness issue quotes the upstream error "Function call is missing a thought_signature in functionCall parts." (Q-s35, Q-s36, both third-party)
- Basis: Filled (Google, Gemini OpenAI-compatible route). Also in workspace: `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20) says Gemini "exposes signatures through `extra_content`". Confirmed live.

**Q27. Gemini API's own OpenAI-compatible page: reasoning effort is mapped per model family, thought summaries need a Google-only switch, and the layer is still labeled beta.**

- Mapping table columns: `reasoning_effort (OpenAI)` to `thinking_level (Gemini 4)`, `thinking_level (Gemini 3.1 Flash-Lite)`, `thinking_level (Gemini 3 Flash)`, `thinking_budget (Gemini 2.5)`. Rows include "`minimal` | `low` | `minimal` | `minimal` | `1,024`" and "`high` | `high` | `high` | `high` | `24,576`", and "`xhigh` | `xhigh` (`gemini-4-neon` and `gemini-4-hydrogen` only) | Not supported | Not supported | Not supported". (Q-s34)
- "If you want to disable thinking, you can set `reasoning_effort` to `"none"` for 2.5 models. Reasoning cannot be turned off for Gemini 2.5 Pro, Gemini 3, or Gemini 4 models." (Q-s34)
- "Note that `reasoning_effort` and `thinking_level`/`thinking_budget` overlap functionality, so they can't be used at the same time." (Q-s34)
- Thought summaries are requested with `extra_body: { "google": { "thinking_config": { "thinking_level": "high", "include_thoughts": true } } }`. (Q-s34)
- Caching: "`cached_content` | Text | Chat | Corresponds to Gemini's general content cache." passed as `extra_body.google.cached_content` with a `cachedContents/...` handle. (Q-s34)
- "Any other parameters not listed here or in the `extra_body` section will be silently ignored by the compatibility layer." (said of image generation) and "Support for the OpenAI libraries is still in beta while we extend feature support." "If you aren't already using the OpenAI libraries, we recommend that you call the Gemini API directly." (Q-s34)
- "The Gemini API matches OpenAI's `service_tier` parameter in name and logic". (Q-s34)
- Basis: Filled (Google Gemini API). Also in workspace: the 2026-10-28 survey says "Gemini's route loses thought signatures". That is wrong as a statement about today's route for tool calls (Q26): the signature is returned and must be replayed through `extra_content`.

**Q28. Azure AI Foundry gives Claude its own Anthropic-shaped path and keeps the OpenAI-shaped path for the rest. Its model router documents which features exist on which format.**

- "Claude models use a separate `/anthropic` endpoint and the Anthropic Messages API, not the OpenAI-compatible endpoint." Endpoint `https://<resource-name>.services.ai.azure.com/anthropic/v1/messages`. (Q-s37)
- Model router, three formats: "Responses, Chat Completions, and Messages (Anthropic format, preview)". "Only Claude models answer Messages requests". (Q-s38)
- Feature table rows, quoted: "Reasoning effort parameter | `reasoning.effort` | `reasoning_effort` | `output_config.effort`"; "Multi-turn state | `previous_response_id` and `conversation` | No (send the full history in `messages`) | No (send the full history in `messages`)"; "Built-in tools (web search, file search, code interpreter, MCP) | Yes | No | Not documented"; "Prompt caching | Yes | Yes | Yes (`cache_control`)". (Q-s38)
- Parameter dropping: "After it selects a model, model router drops optional parameters that the model doesn't support, and sends the rest of the request. For example, it drops `temperature`, `top_p`, `presence_penalty`, `frequency_penalty`, `logit_bias`, `logprobs`, and `stop` for reasoning models, and `reasoning_effort` for non-reasoning models." (Q-s38)
- Opt-out of dropping uses a name OpenRouter also uses inside its `provider` object: "To keep every parameter, set `routing_config.require_parameters` to `true` (preview). ... only models that support every parameter you sent are eligible, so nothing is dropped. If no model qualifies, the request fails with HTTP 400". (Q-s38)
- "Send reasoning effort in each API's native field. `routing_config` has no reasoning-effort field." (Q-s38)
- "Prompt caching works through the underlying models: cache hits happen when the same model serves consecutive requests with overlapping prefixes. Session affinity raises the odds." (Q-s38)
- What Claude loses on Foundry, from Anthropic's page: not supported at all are "Admin API", "Advisor tool", "Claude Managed Agents", "Compliance API", "Models API", "Message Batches API", "Server-side fallback (the `fallbacks` parameter; use the client-side fallback pattern instead)", "User profile endpoints". Not supported when hosted on Azure: "Code execution", "Web search and web fetch tool versions later than `web_search_20250305` and `web_fetch_20250910`", "Agent Skills", "Programmatic tool calling", "Files API". "Requests that use these features against a deployment hosted on Azure return a `400 Bad Request` error by design." (Q-s39)
- Basis: Filled (Microsoft Azure AI Foundry) and Vendor-documented (Anthropic, for the Foundry feature list).

### Part 1F. First-party flattenings: Anthropic's and OpenAI's own pages

**Q29. Anthropic's OpenAI SDK compatibility layer, re-read today. It still never returns thinking and still does not cache. Three behaviors changed from "ignored" to "400" since the 2026-10-28 survey.**

- Purpose statement: "This compatibility layer is primarily intended to test and compare model capabilities, and is not considered a long-term or production-ready solution for most use cases." "For the best experience and access to the full Claude API feature set (PDF processing, citations, thinking, and prompt caching), use the native Claude API." (Q-s40)
- Thinking: "The response doesn't include Claude's thinking. The native Claude API can return summarized thinking." "Setting `thinking` to `{"type": "adaptive"}` returns a 400 error." "On models that reject `enabled` in the native Claude API, the compatibility layer converts `enabled` to adaptive thinking. On those models, `budget_tokens` is required but has no effect." "On Claude 5 models and Claude Mythos Preview, thinking is on by default, so you can omit the `thinking` field." (Q-s40)
- "`reasoning_effort` | Ignored". (Q-s40)
- Caching: "Prompt caching is not supported, but it is supported in the Anthropic SDKs." Usage: "`usage.completion_tokens_details` | Always empty", "`usage.prompt_tokens_details` | Always empty". (Q-s40)
- Changed since the survey: "`response_format` accepts only `json_schema` with `strict: true`. A request that sets `json_object` or `text` returns a 400 error." (the survey recorded `response_format` as ignored). "Audio and file inputs are not supported. A request that includes an `input_audio` or `file` content part returns a 400 error." "`n` | Must be exactly 1. Any other value returns a 400 error." (Q-s40)
- Still silently ignored: `logprobs`, `metadata`, `prediction`, `presence_penalty`, `frequency_penalty`, `seed`, `service_tier`, `audio`, `logit_bias`, `store`, `user`, `modalities`, `top_logprobs`, and `strict` on tools. "Most unsupported fields are silently ignored rather than producing errors." (Q-s40)
- System hoisting: "System/developer messages are hoisted and concatenated into the top-level `system` field, wherever they appear in the conversation." joined "with a single newline (`\n`)". (Q-s40)
- Forced tool choice can fail: "Some models return a 400 error for forced tool use: Claude Fable 5.2, Claude Mythos 5.2, Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1, and Claude Mythos 5.1. Use `"auto"` on those models." (Q-s40)
- Errors: "The compatibility layer maintains consistent error formats with the OpenAI API. However, the detailed error messages are not equivalent." (Q-s40)
- Basis: Vendor-documented (Anthropic). Also in workspace: the 2026-10-28 survey, section 5.2.

**Q30. OpenAI's own Chat Completions reference now defines `prompt_tokens_details.cache_write_tokens`, explicit `prompt_cache_breakpoint` markers on content parts, and `annotations` of type `url_citation`. Three names on OpenRouter's extension list are therefore inside the standard today.**

- Usage: "`prompt_tokens_details: optional object { audio_tokens, cache_write_tokens, cached_tokens, 2 more }` Breakdown of tokens used in the prompt. ... `cache_write_tokens: optional integer (default: 0)` The unadjusted number of prompt tokens written to cache. `cached_tokens: optional integer (default: 0)` Cached tokens present in the prompt." (Q-s41)
- Markers: "`prompt_cache_breakpoint: optional object { mode, ttl }` Marks the exact end of a reusable prompt prefix. When `ttl` is omitted, the breakpoint inherits the request's `prompt_cache_options.ttl` ... `ttl: optional "30m" or "12h"`". Request-level: "`prompt_cache_options: optional object { mode, ttl }` Options for prompt caching. Supported for `gpt-5.6` and later models." (Q-s41)
- Citations: "`type: "url_citation"` The type of the URL citation. Always `url_citation`." "`url_citation: object { end_index, start_index, title, url }` A URL citation when using web search." (Q-s41)
- The reference lists `reasoning_effort` as the reasoning control and defines no `reasoning`, `reasoning_content`, or `reasoning_details` field on the chat-completions message. (Q-s41, keyword search of the page)
- OpenAI's caching guide for the Responses route: "Track `usage.input_tokens_details.cached_tokens`, `usage.input_tokens_details.cache_write_tokens`", and "`input_tokens_details.cache_write_tokens_details` breaks writes into `ttl_30m_tokens` and `ttl_12h_tokens`. These counts do not overlap and sum to `cache_write_tokens`." "Unsupported values return `400 invalid_request_error`." (Q-s42)
- Basis: Vendor-documented (OpenAI). Confidence: high (field names quoted from the live reference).


### Part 1G. More aggregators and inference platforms that document translation behavior

**Q31. LLM Gateway (llmgateway.io, hosted and self-hosted) uses OpenRouter's `reasoning_details` shape on chat completions and documents the replay rules that OpenRouter's translator keeps closed: position restore, provider scoping, and what happens when the conversation moves to another provider.**

- "The Chat Completions format has no reasoning items, so the gateway carries the same payloads on the assistant message as `reasoning_details`", with the example entry `{"type": "reasoning.encrypted", "data": "gAAAAABqfkEO3uEFtXeGSNE9...", "id": "rs_057d93d1ba22357e01...", "format": "openai-responses-v1", "index": 0}`. (Q-s47)
- "To preserve the reasoning, append the assistant message back into `messages` exactly as you received it (with `reasoning_details` intact) before the matching `tool` result. No opt-in parameter is required — the field is always present when the model produced one. When streaming, the entries arrive as a `delta.reasoning_details` chunk." (Q-s47, long dash in the original)
- "The gateway converts supported `reasoning_details` formats to native provider fields and strips these entries from other upstream requests." (Q-s47)
- Claude: "On Chat Completions, the same blocks arrive as `reasoning_details` entries with `format: "anthropic-claude-v1"`: `reasoning.text` with `text` and `signature`, or `reasoning.encrypted` with `data`." "Replay the assistant turn unchanged, including each entry's `block_index`. The gateway replays thinking only in the position where Claude produced it. ... Signatures are scoped to the provider that issued them: when a conversation moves to another provider, the gateway drops that provider's thinking blocks instead of failing the request." (Q-s47)
- Gemini: "Gemini text turns carry signatures in `reasoning_details` entries with `type: "reasoning.text"` and `format: "google-gemini-v1"`. ... Preserve each entry's `google_part` metadata: it lets the gateway restore the original signed text boundaries. Streaming clients must collect `delta.reasoning_details`, including entries arriving with empty text." "For Chat Completions, `content[]` text parts and `tool_calls[]` also accept `extra_content.google.thought_signature`." "Only replay signatures with the original content and the model/provider that issued them." (Q-s47)
- Errors are forwarded: "A modified, truncated, or foreign payload is rejected upstream and the error is forwarded unchanged" with `"code": "invalid_encrypted_content"`. (Q-s47)
- Reasoning control: `reasoning_effort` or "the unified `reasoning` configuration object with an `effort` field", plus `reasoning.max_tokens`: "When `reasoning.max_tokens` is specified, it overrides `reasoning.effort` and `reasoning_effort`." An unsupported budget is an error, not a silent drop: "Model gpt-4o does not support reasoning.max_tokens. Remove the reasoning.max_tokens parameter or use a model that supports explicit reasoning token budgets". "Effort tiers are never downgraded by the gateway". (Q-s47)
- "The response will include a `reasoning` field in the message object". (Q-s47)
- Basis: Filled (LLM Gateway). This gateway had to add two fields of its own (`block_index`, `google_part`) to OpenRouter's entry shape to restore block position and signed text boundaries. That is evidence that the plain OpenRouter entry shape does not by itself say where a block sat.

**Q32. Requesty returns reasoning text as `reasoning_content` and the signature as one top-level `reasoning_signature` string on the assistant message, and overloads `reasoning_effort` to accept a token budget as a string.**

- "When using Deepseek and Anthropic, the reasoning content in the response will be under 'reasoning_content'." "OpenAI does NOT share the actual reasoning tokens. You will not see them in the response." (Q-s43)
- "Some reasoning models sign or encrypt their reasoning, for example Anthropic thinking signatures, Gemini thought signatures and OpenAI encrypted reasoning. On `/v1/chat/completions`, Requesty returns this value as a top-level `reasoning_signature` field on the assistant message. When streaming, it arrives as `reasoning_signature` on a `delta`." (Q-s43)
- "When you send the conversation back, keep `reasoning_signature` on the assistant message it came with, including assistant messages that only carry `tool_calls`". "Requesty passes it back to the provider in that provider's own format. If you drop it, the model can lose its earlier reasoning. On Gemini, a tool loop still returns no error, but `reasoning_tokens` drops to 0 after the first one or two calls." (Q-s43)
- Effort and budget in one field: the Python example sends `reasoning_effort="10000"`. "If the client specifies a reasoning budget string (e.g. "10000"), Requesty converts it to an effort": "0-1024 -> "low"", "1025-8192 -> "medium"", "8193 or higher -> "high"". Toward Anthropic: ""min" / "none" / "low" -> 1024", ""medium" -> 8192", ""high" -> 16384", ""max" -> max output tokens for model minus 1". (Q-s43)
- Route limit: "`xhigh` and `max` are available on the `/v1/chat/completions` and `/v1/responses` endpoints. It is not reachable via the Anthropic-style `/v1/messages` endpoint". (Q-s43)
- Basis: Filled (Requesty). One signature string per message cannot represent several signed thinking blocks or a redacted block next to a signed one. The page does not say how those cases are packed.

**Q33. Groq and Cerebras return parsed reasoning in `message.reasoning` and select the presentation with a `reasoning_format` request parameter.**

- Groq: "`parsed` | Separates reasoning into a dedicated `message.reasoning` field while keeping the response concise." "`raw` | Includes reasoning within `<think>` tags in the main text content." "`hidden` | Returns only the final answer." "If reasoning is explicitly set to `raw` with JSON mode or tool use enabled, we will return a 400 error." "The `include_reasoning` parameter cannot be used together with `reasoning_format`. These parameters are mutually exclusive." (Q-s44, the tag name was stripped by the fetch tool and is inferred from the surrounding sentence)
- Cerebras: "`parsed` | Returns reasoning in `choices[0].message.reasoning` and the final answer in `choices[0].message.content`". "Reasoning is returned separately in `message.reasoning` or streaming `delta.reasoning`." (Q-s45)
- Cerebras replay: "Chat Completions is stateless. Resend conversation history with each request, including any assistant reasoning and tool calls required by the model." For Kimi K3: "Replay the complete assistant response, including reasoning and tool calls." For Qwen and GLM: "`clear_thinking` controls whether the model receives historical assistant reasoning." (Q-s45)
- Cerebras hides model-family parameters behind its own: "Do not send Qwen-native parameters such as `disable_reasoning`, `enable_thinking`, `preserve_thinking`, or `thinking_budget`. Use `reasoning_effort` and `clear_thinking` instead." And some values are accepted then ignored: "[`kimi-k2.7-code`] always enables reasoning. The API accepts `reasoning_effort` for client compatibility but ignores its value." (Q-s45)
- "`reasoning_effort` is a standard OpenAI client parameter. Pass Cerebras-specific parameters such as `reasoning_format` and `clear_thinking` through `extra_body`". (Q-s45)
- Basis: Filled (Groq, Cerebras).

**Q34. Together uses both field names, depending on the model, and has a `reasoning` request object with an `enabled` member.**

- "they first produce a chain of thought, visible as tokens in a dedicated output field (`reasoning` or `reasoning_content`, depending on the model)". Stream examples read `delta.reasoning`. (Q-s46)
- "Hybrid models let you toggle reasoning on or off using the `reasoning` parameter." Example: `reasoning: { enabled: true }`. (Q-s46)
- "DeepSeek-R1 uses a different format. It outputs reasoning inside `<think>` tags within the `content` field rather than a separate `reasoning` field." (Q-s46)
- Preserved thinking: "Enable preserved thinking by setting `clear_thinking` to `false`" inside `chat_template_kwargs`, and "include the unmodified `reasoning_content` from previous turns back in the conversation". (Q-s46)
- Basis: Filled (Together AI). Also in workspace: the 2026-10-28 survey says Together's "reasoning has three patterns across the catalogue that must be symmetric on replay".

**Q35. Fireworks uses `reasoning_content` and accepts either OpenAI's `reasoning_effort` or an Anthropic-shaped `thinking` object on its chat-completions route, but not both.**

- "Fireworks provides access to the model's reasoning process through the `reasoning_content` field." (Q-s48)
- "You can control the reasoning token length using either the `reasoning_effort` parameter or the Anthropic-compatible `thinking` parameter." "You cannot specify both `thinking` and `reasoning_effort` in the same request. If both are provided, a validation error will be raised." (Q-s48)
- "When building multi-turn tool-calling agents with models that support interleaved thinking, you **MUST** include the `reasoning_content` from previous assistant turns in subsequent requests." (Q-s48)
- "Coming from NVIDIA NIM or vLLM? Fields like `chat_template_kwargs.enable_thinking` are mapped onto `reasoning_effort` automatically." (Q-s48)
- Basis: Filled (Fireworks AI).

**Q36. GitHub Copilot's chat-completions route uses a third pair of names: `reasoning_text` and `reasoning_opaque`.**

- Pydantic AI's Copilot page: "Copilot returns Anthropic and Google reasoning in a `reasoning_text` field rather than in either of the field names OpenAI-compatible providers usually use ... Copilot returns a `reasoning_opaque` signature alongside it, which Pydantic AI does not carry; Copilot accepts follow-up turns without it." (Q-s49, third-party account of GitHub's wire format)
- Zed's Copilot client declares both fields on the assistant message: `reasoning_opaque: Option<String>` and `reasoning_text: Option<String>` in `crates/copilot_chat/src/copilot_chat.rs` at commit `be52d3b7`. (Q-s50, read through a search-engine excerpt, generic parameters restored from context)
- Basis: Filled (GitHub Copilot, through two client projects). GitHub's own public documentation of these fields was not found. Confidence: medium.


### Part 2. Who else uses OpenRouter's extension names, and which names compete

The OpenRouter side of each name is in the OpenRouter evidence files. The items below record only the other parties. "Clone" means a product whose public documentation reproduces OpenRouter's schema nearly word for word.

**Q37. `reasoning` as a string on the assistant message and on stream deltas. Used by at least nine other parties. This is the most widely shared of the non-standard reasoning names today, and vLLM switched to it from `reasoning_content`.**

- vLLM documentation (main branch): "Reasoning models return an additional `reasoning` field in their outputs". "`reasoning` used to be called `reasoning_content`. To migrate, directly replace `reasoning_content` with `reasoning`. It is important that you also update your client code. Otherwise, your client code could silently read an empty `reasoning_content`, even when `reasoning` is populated." (Q-s52)
- Vercel AI Gateway: `message.reasoning`, `delta.reasoning`. (Q2)
- Groq: "a dedicated `message.reasoning` field". Cerebras: "`choices[0].message.reasoning`", "streaming `delta.reasoning`". (Q33)
- Together: "(`reasoning` or `reasoning_content`, depending on the model)". (Q34)
- LLM Gateway: "The response will include a `reasoning` field in the message object". (Q31)
- Infron: "Reasoning information may appear in `message.reasoning`, `message.reasoning_content` or `message.reasoning_details`." (Q-s58)
- Bifrost emits all three at once, per a client bug report: "Bifrost also emits `reasoning`, `reasoning_content`, and `reasoning_details: [{"type":"reasoning.text","text":...,"signature":...}]` deltas". (Q-s59, third-party issue report)
- Sargalay's response schema lists `"reasoning": "string"` on the message. (Q-s60)
- Basis: Filled (vLLM, Vercel AI Gateway, Groq, Cerebras, Together, LLM Gateway, Infron, Bifrost, Sargalay). Count: 9 adopters with a source each. Also in workspace: the 2026-10-28 survey lists vLLM under `reasoning_content`. vLLM's current documentation says the opposite.

**Q38. `reasoning_content` is the competing name with the longest list of native vendors. It is plain text only and has no place for a signature.**

- DeepSeek: "In thinking mode, the chain-of-thought content is returned via the reasoning_content parameter, at the same level as content." "for requests carrying the tools parameter, the reasoning_content must be fully passed back to the API in all subsequent requests — even for turns where the model did not perform a tool call. If your code does not correctly pass back reasoning_content, the API will return a 400 error." "If the request does not carry the tools parameter: reasoning_content does not need to be passed back; even if passed to the API, it will be ignored". (Q-s53, long dash in the original)
- llama.cpp server: "`--reasoning-format FORMAT` ... - none: leaves thoughts unparsed in `message.content` - deepseek: puts thoughts in `message.reasoning_content` - deepseek-legacy: keeps `<think>` tags in `message.content` while also populating `message.reasoning_content` (default: auto)". (Q-s54)
- Fireworks: "through the `reasoning_content` field". (Q35) Requesty: "the reasoning content in the response will be under 'reasoning_content'". (Q32)
- LiteLLM: "LiteLLM will standardize the `reasoning_content` in the response and `thinking_blocks` in the assistant message." (Q-s55)
- agentgateway source comment, which is itself a survey: "There is no consistent standard for OpenAI compatible endpoints in how to express 'reasoning' / Deepseek: reasoning_content ... / z.ai: reasoning_content ... / OpenRouter: `reasoning` and `reasoning_details` ... / LiteLLM: `reasoning_content` and `thinking_blocks` ... / Since 3/4 of these use `reasoning_content`, it seems like a reasonable default." (Q-s56, `crates/llm/src/types/completions.rs` at commit `77e45c73`, read through a search-engine excerpt)
- MiniMax (through a reseller's API manual): with `reasoning_split` true, "Thinking content is split into `choices[].message.reasoning_content` and `reasoning_details`". (Q-s57, third-party copy of a vendor schema)
- Together uses it for some models (Q34). Bifrost and Infron emit it next to `reasoning` (Q37).
- Basis: Filled (DeepSeek, llama.cpp, Fireworks, Requesty, LiteLLM, agentgateway, MiniMax, Z.ai, Together, Bifrost, Infron). Count: 11 with a source each (Z.ai only through agentgateway's comment).

**Q39. `reasoning_details` as an array of typed entries. Five other parties use OpenRouter's entry shape. One model vendor (MiniMax) emits it natively. One gateway uses the same field name with a different entry shape.**

- Same shape, hosted gateways: Vercel AI Gateway (Q3) and LLM Gateway (Q31), both with `reasoning.text`, `reasoning.encrypted`, and the `anthropic-claude-v1` and `openai-responses-v1` format tags. Vercel also documents `reasoning.summary`, `google-gemini-v1`, `xai-responses-v1`, and `unknown`.
- Same shape, open-source gateway: Bifrost's Anthropic provider guide lists "`thinking` blocks → `reasoning_details`" among its response mappings (Q-s61, read through a search-engine excerpt), and the client bug report in Q37 shows the wire form.
- Same shape, API-compatible reseller: Infron reads `message.get("reasoning_details")` in its reference example. (Q-s58)
- Same shape from a model vendor: MiniMax's OpenAI-compatible schema, as published in a reseller's manual: "`reasoning_details` : type: array / description: Structured thinking content. Only returned when `reasoning_split=true` is set in the request", with entry properties `type` ("Thinking content type, e.g. `reasoning.text`"), `id`, `format` ("Format identifier for the thinking content, e.g. `MiniMax-response-v1`"), `index`, and `text`. (Q-s57) A copy of MiniMax's own guide says: "the entire `response_message` — including the `reasoning_details` field — must be preserved in the message history and passed back to the model in the next round of interaction." (Q-s62, third-party copy of a vendor page; long dashes in the original)
- Same name, different contents: Prisme.ai's gateway returns "`reasoning_details` : type: array / description: Anthropic thinking blocks produced with this assistant message (`thinking` blocks carry `thinking` and `signature`, `redacted_thinking` blocks carry `data`). Returned by the gateway and meant to be sent back verbatim when the turn is replayed." with entry `type` enum `thinking`, `redacted_thinking`, plus a sibling "`reasoning_model` : Gateway model id that produced `reasoning_details`. On replay the blocks are forwarded only when this matches the model being called; the tag itself is never sent to a provider." "Anthropic thinking blocks are never streamed." (Q-s63)
- Client support named in passing: Pydantic AI's maintainers list "Anthropic and Bedrock signatures, OpenRouter `reasoning_details` and the Responses `namespace`" as provider-gated replay data. (Q-s35)
- Basis: Filled (Vercel AI Gateway, LLM Gateway, Bifrost, Infron, MiniMax, Prisme.ai). Count: 5 with the OpenRouter entry shape, 1 with a different shape under the same name. Also in workspace: the 2026-11-17 report on the Anthropic Messages API and harness survey names OpenRouter and Bifrost as the readers of `reasoning_details`. Vercel AI Gateway, LLM Gateway, Infron, and MiniMax are additions.

**Q40. The `reasoning` request object. Four other chat-completions gateways accept `reasoning.effort`, and three of them accept `reasoning.max_tokens`. The name matches OpenAI's own Responses route, where `reasoning.effort` is standard.**

- Vercel AI Gateway: `enabled`, `effort`, `max_tokens`, `exclude`, and "the standard `reasoning_effort` field is an alias for `reasoning.effort`". (Q4)
- LLM Gateway: "the unified `reasoning` configuration object with an `effort` field" and `reasoning.max_tokens`. "You cannot use both `reasoning_effort` and `reasoning.effort` in the same request." (Q31)
- Infron: "Chat Completions accepts a unified `reasoning` field, which Infron converts to the provider's parameters." Rows: "`reasoning.effort` | Options: `xhigh`, `high`, `medium`, `low`, `minimal`, `none`" and "`reasoning.max_tokens` | Set an upper limit for reasoning tokens, not an exact consumption target". "Choose either `effort` or `max_tokens`." (Q-s58)
- Ollama's OpenAI-compatible `/v1/chat/completions` lists as supported: "`reasoning_effort` (model-defined names and compatibility aliases)" and "`reasoning` / `effort` (model-defined names and compatibility aliases)". "`reasoning_effort` and `reasoning.effort` control model thinking." (Q-s64)
- Together: `reasoning: { enabled: true }`. (Q34)
- The standard's own names: OpenAI chat completions has "`reasoning_effort: optional ReasoningEffort or null (default: "medium")`" (Q-s41). Azure's table gives the three native names side by side: "Reasoning effort parameter | `reasoning.effort` | `reasoning_effort` | `output_config.effort`" for Responses, Chat Completions, and Messages. (Q28)
- Competing shapes: Requesty puts a budget into `reasoning_effort` as a numeric string (Q32). Fireworks accepts an Anthropic-shaped `thinking` object on chat completions (Q35). DeepSeek uses `thinking: {type: "enabled/disabled"}` plus `reasoning_effort` on chat completions (Q-s53). Google uses `extra_body.google.thinking_config` (Q27). Groq and Cerebras add `reasoning_format` (Q33).
- Basis: Filled (Vercel AI Gateway, LLM Gateway, Infron, Ollama, Together). Count: 4 with `reasoning.effort` on chat completions, 3 with `reasoning.max_tokens`, 2 with `reasoning.enabled`.

**Q41. `native_finish_reason`. Found only in OpenRouter clones. The independent gateways read return `finish_reason` alone.**

- ModelGates: "`native_finish_reason: string | null;`" on both choice types, and "The raw finish_reason string returned by the model is available via the `native_finish_reason` property." Its normalized set is "`tool_calls`, `stop`, `length`, `content_filter`, `error`". (Q-s65)
- InteractiveAI Router: "Access the provider's original finish reason through `native_finish_reason` when debugging provider-specific behavior." (Q-s66)
- Sargalay removes it: "OpenAI-shaped, with two differences: `usage.cost` is in points, and the provider-specific `native_finish_reason` is stripped." (Q-s60)
- Not present in: Vercel AI Gateway's 23 downloaded pages (Q10), Kilo Gateway's API reference (response type lists only `finish_reason: "stop" | "length" | "tool_calls" | "content_filter"`, Q-s67), Bedrock Converse (returns one normalized `stopReason`, Q21), Cloudflare's export (Q14).
- Another design for the same need: Bedrock lets the caller request the vendor's raw field by JSON Pointer through `additionalModelResponseFieldPaths` (Q20, Q21).
- Basis: Filled (ModelGates, InteractiveAI, Sargalay). Count: 2 clones emit it, 1 reseller strips it, 0 independent adopters.

**Q42. The `provider` preferences object. Adopted by API-compatible resellers with the same member names. Larger platforms use their own shapes.**

- Infron: "`model` selects a model; `provider` selects serving providers." Example `"provider": {"sort": "price", "allow_fallbacks": true}`, with sections `provider.sort`, `provider.order`, `provider.only`. (Q-s58)
- NanoGPT's migration table: "`provider.order`, `only`, `ignore`, `allow_fallbacks`, `sort`, `max_price`, `quantizations`, `zdr` | Work on models that support provider selection"; "`provider.require_parameters` | Applies only on routes that can check parameters"; "`provider.data_collection: "deny"` | Refused with `400`". (Q-s68)
- ModelGates reproduces OpenRouter's request schema. (Q-s65, Q-s69)
- Vercel AI Gateway: top-level `provider` exists but "supports `sort` only". The full control surface is `providerOptions.gateway` with `order`, `sort`, `models`, `byok`, `providerTimeouts`, `caching`. (Q6)
- Azure model router: `routing_config.require_parameters`. (Q28) Hugging Face: a model-id suffix. (Q16) Cloudflare: dynamic routes and model aliases configured on the gateway, invoked as `dynamic/{route}`. (Q12)
- Client code that hard-codes OpenRouter's member names: Zed's `OpenRouterProvider` struct has `order`, `allow_fallbacks`, `require_parameters`, `data_collection`, `only`, `ignore`, `quantizations`, `sort`. (Q-s70, `crates/settings_content/src/language_model.rs` on main, read through a search-engine excerpt)
- Basis: Filled (Infron, NanoGPT, ModelGates, Vercel AI Gateway, Azure AI Foundry, Hugging Face, Cloudflare). Count: 3 adopters of the object with OpenRouter's member names, 1 partial (Vercel, `sort` only), 3 competing designs.

**Q43. The `models` fallback array. Same name and meaning at Vercel and NanoGPT. Anthropic's native API uses `fallbacks`.**

- Vercel AI Gateway: "use the `models` field directly at the top level of your request". (Q6)
- NanoGPT: "`models` fallback list | Works, with up to 4 IDs". (Q-s68)
- A proxy that forwards to OpenRouter documents both names: "OpenRouter receives `fallbacks` for Claude Messages or `models` for Chat Completions, and runs the fallback itself." (Q-s71, third-party project documentation)
- Competing name from a vendor: Anthropic lists "Server-side fallback (the `fallbacks` parameter; use the client-side fallback pattern instead)" among Claude API features. (Q-s39)
- Basis: Filled (Vercel AI Gateway, NanoGPT). Count: 2 adopters.

**Q44. `usage.cost` and `usage.cost_details`. Adopted by at least five other gateways, three of them with OpenRouter's `upstream_inference_cost` sub-field names.**

- LLM Gateway: `"cost": 0.000125, "cost_details": { "upstream_inference_cost": 0.000125, "upstream_inference_prompt_cost": 0.000025, "upstream_inference_completions_cost": 0.0001, "total_cost": ..., "input_cost": ..., "output_cost": ..., "cached_input_cost": ..., "request_cost": ..., "web_search_cost": ..., ... }`. "Every billed reply carries `usage.cost`, what the request was billed in USD, with `usage.cost_details` breaking it down ... There is nothing to opt into." In streams "The cost fields are included in the final usage chunk sent before the `[DONE]` message". (Q-s51)
- Sargalay: `"cost": 1, "cost_details": { "upstream_inference_cost": 1, "upstream_inference_prompt_cost": 1, "upstream_inference_completions_cost": 1 }`, with the unit changed: "`usage.cost` and every `usage.cost_details.*` field come back in points". (Q-s60)
- RouterPlus: "each billed response states its cost in `usage.cost`". It serves two formats side by side: "OpenAI-compatible | `https://api.routerplus.com/v1` | `POST /v1/chat/completions`, and `POST /v1/responses` for OpenAI's models" and "Anthropic-compatible | `https://api.routerplus.com` | `POST /v1/messages`". (Q-s72)
- Prisme.ai: "The non-streaming response is enriched with `usage.cost`, `usage.duration_ms`, and `usage.carbon` (Prisme.ai extensions over the standard OpenAI shape)." (Q-s63)
- ModelGates example: `"cost": 0.00014` inside `usage`. (Q-s65) Experiential: a per-search fee "is included in the response's `usage.cost`". (Q-s73)
- Infron keeps OpenRouter's older opt-in: "Add `"usage": {"include": true}` to Chat requests." (Q-s58)
- Vercel AI Gateway reports cost through `GET /v1/generation?id=` and `providerMetadata`, not through `usage.cost`. (Q10)
- Basis: Filled (LLM Gateway, Sargalay, RouterPlus, Prisme.ai, ModelGates, Experiential). Count: 6 adopters of `usage.cost`, 2 of `cost_details` with `upstream_inference_*` names. Caution: Sargalay shows the same field name with a different unit.

**Q45. `prompt_tokens_details.cache_write_tokens`. This is now OpenAI's own field, so it is a name inside the standard and no longer only an aggregator extension.**

- OpenAI chat-completions reference: "`cache_write_tokens: optional integer (default: 0)` The unadjusted number of prompt tokens written to cache." (Q30)
- LLM Gateway: `"prompt_tokens_details": { "cached_tokens": 0, "cache_write_tokens": 0, "image_tokens": 0, "audio_tokens": 0, "video_tokens": 0 }`. (Q-s51)
- Amazon Bedrock documents the same name for OpenAI models on its OpenAI-shaped routes. (Q23)
- Competing names for the same count: Anthropic Messages `cache_creation_input_tokens` (shown on Vercel's Messages route, Q7), Bedrock Converse `cacheWriteInputTokens` (Q22), Prisme.ai `cache_write_input_tokens` (Q-s63), Vercel's generation route `native_tokens_cache_creation` (Q10).
- Not reported at all: Vercel's chat-completions examples and Hugging Face's caching page show only `cached_tokens`. (Q7, Q18)
- Basis: Vendor-documented (OpenAI) and Filled (LLM Gateway, Amazon Bedrock). Local claim LC17 calls this "the extension name OpenRouter uses". As of today the name is also in OpenAI's reference. Which party used it first was not established here.

**Q46. `cache_control` inside a chat-completions body. Widely accepted, including by one model vendor's own OpenAI-compatible API. The placement differs between products.**

- Vercel AI Gateway accepts it and documents it at message level: `{"role": "user", "content": "...", "cache_control": {"type": "ephemeral"}}`. (Q7)
- Infron: "Add `cache_control: {"type": "ephemeral"}` to a content block. The cached prefix runs from the start of the prompt through the end of that block." (Q-s58)
- LiteLLM's supported-provider list for the "`cache_control` marker" includes "Anthropic API", "Azure AI - Claude", "OpenRouter - Claude, Gemini, MiniMax, GLM, z-ai routes", "Databricks - Claude", "DashScope / Qwen", "MiniMax", "Z.ai / GLM". LiteLLM translates it for OpenAI: "`"cache_control": {"type": "ephemeral"}` on the targeted block" becomes "`"prompt_cache_breakpoint": {"mode": "explicit"}` on the targeted block", and "`control.ttl` picks the 5m or 1h cache" on the Anthropic side while "`control` is ignored; set `prompt_cache_options` yourself for `ttl`" on the OpenAI side. (Q-s74)
- Alibaba's OpenAI-compatible API accepts the marker natively, according to two gateways that forward it: a gateway guide says "Explicit caching supports up to four `cache_control: {"type": "ephemeral"}` markers on system/user text content blocks. This path is restricted to text blocks" (Q-s75, third-party), and Vercel lists Alibaba and MiniMax as "Explicit" caching providers that receive "`cache_control` breakpoints" (Q7).
- Azure model router and Vercel accept `cache_control` only in its home format on the Messages route for Claude ("Prompt caching | Yes | Yes | Yes (`cache_control`)", Q28).
- Competing markers: OpenAI `prompt_cache_breakpoint` on a content part plus `prompt_cache_options` at the root (Q30), Bedrock Converse `cachePoint` as a separate block (Q22), Gemini `extra_body.google.cached_content` naming a stored cache (Q27), Hugging Face a session header only (Q18).
- Basis: Filled (Vercel AI Gateway, Infron, LiteLLM, Alibaba through forwarding gateways). Count: 4 with a source each, plus MiniMax, Z.ai, and Databricks named in LiteLLM's list.

**Q47. `annotations` with `url_citation`. This is OpenAI's own chat-completions shape, not an OpenRouter invention.**

- OpenAI reference: "`type: "url_citation"` ... `url_citation: object { end_index, start_index, title, url }` A URL citation when using web search." (Q30)
- Open Responses schema: `type: "url_citation" = "url_citation";` in `schema/content.tsp` lines 88 and 140 at commit `dce9ddb`. (Q-s76)
- Another gateway that emits it for gateway-run search: "`url_citation` annotations on Chat (`choices[0].message.annotations`, one `delta.annotations` chunk on a stream)". (Q-s73)
- Vendors that do not map their search results into it on a gateway: Cloudflare returns Anthropic's `server_tool_use` and `web_search_tool_result` blocks on the Messages route. (Q13)
- Basis: Vendor-documented (OpenAI) and Filled (Open Responses, Experiential).

**Q48. `plugins` and `transforms`. Adoption is limited to one clone that reproduces `plugins` and two gateways that accept the fields for compatibility, one of which ignores them.**

- ModelGates: "Plugins are enabled by adding a `plugins` array to your chat completions request. Each plugin is identified by its `id`", with the same ids (`web`, `response-healing`, `auto-router`, context compression with engine `middle-out`). (Q-s69)
- NanoGPT: "`transforms: ["middle-out"]` | Accepted, no effect | Trim long prompts yourself" and "`plugins` with `web` | Accepted, no effect | Use the `:online` suffix". "`plugins: [{"id": "web"}]` runs no search and raises no error." (Q-s68)
- Experiential treats the web plugin as one of four spellings of the same request: "Chat `web_search_options`, the OpenRouter `plugins: [{"id": "web"}]` object and the `:online` model suffix, the Responses `web_search` tool, and the Anthropic `web_search_*` server tool all ask for the same thing." "Only the `web` plugin id is served; other plugin ids are a named `400`." (Q-s73)
- Basis: Filled (ModelGates, NanoGPT, Experiential). Count: 1 clone, 2 compatibility shims, 0 independent adopters.

**Q49. `thinking_blocks` (LiteLLM). One product. LiteLLM's own documentation says compatible clients ignore it.**

- "`"thinking_blocks": [ # only returned for Anthropic models { "type": "thinking", "thinking": "The capital of France is Paris.", "signature": "EqoBCkgIARABGAIiQL2UoU0b1OHYi+..." } ]`". (Q-s55)
- "When using Anthropic models with `thinking` enabled and tool calling, you must include `thinking_blocks` from the previous assistant response when sending tool results back. Failure to do so will result in a `400 Bad Request` error." (Q-s55)
- "1. OpenAI's Chat Completions spec has no field for `thinking_blocks` 2. OpenAI-compatible clients (LibreChat, Open WebUI, Vercel AI SDK, etc.) ignore the `thinking_blocks` field in responses". (Q-s55)
- LiteLLM's fallback when the field is missing: "Set `litellm.modify_params = True` and LiteLLM will automatically handle this incompatibility by dropping the `thinking` param when `thinking_blocks` are missing". (Q-s55)
- Basis: Filled (LiteLLM). Also in workspace: the 2026-11-17 report on the Anthropic Messages API and harness survey, which lists OpenHands as passing `thinking_blocks` back through LiteLLM.

**Q50. `reasoning_signature`. Two independent products: agentgateway and Requesty.**

- agentgateway: "`reasoning_signature` — the Bedrock/Anthropic cryptographic attestation that must accompany a thinking block when replayed in a subsequent turn. Extracted from the Anthropic SignatureDelta event and forwarded verbatim so downstream consumers can preserve thinking blocks across conversation turns." Request side: "Agentgateway: the Bedrock/Anthropic cryptographic attestation for `reasoning_content`. Required for the provider to accept a replayed thinking block." (Q-s56, long dash in the original)
- Requesty: "a top-level `reasoning_signature` field on the assistant message". (Q32)
- Basis: Filled (agentgateway, Requesty). Count: 2. Requesty's use covers Anthropic, Gemini, and OpenAI. agentgateway's comment names Bedrock and Anthropic.

**Q51. `extra_content.google.thought_signature`. Google's own name, now accepted or reused by at least three other gateways and client libraries, and the subject of an open request to add it to OpenAI's published spec.**

- Google: Q25, Q26.
- LLM Gateway: "`content[]` text parts and `tool_calls[]` also accept `extra_content.google.thought_signature`." (Q31)
- GoModel gateway generalizes the container: "Such state has no OpenAI-compatible field, so GoModel carries it in one place: an `extra_content` object keyed by vendor." "The shape matches Google's own OpenAI-compatible endpoint, so clients built against it work unchanged." It places it at "`tool_calls[].extra_content`; on the assistant message for turn-wide state such as thinking blocks". (Q-s77)
- Pydantic AI stores it and sends it back only to the same provider. (Q-s35)
- An issue on OpenAI's spec repository, open since 2026-01-22: "Google introduced an additional attribute that is **not part of the OpenAPI spec**, which causes incompatibilities for clients that strictly follow the spec." "this was applied **only to the Python SDK**." (Q-s78, third-party issue report)
- A reporter's test recorded in the Pydantic AI pull request: sending `extra_content` and a made-up key inside `tool_calls` to `api.openai.com` returned 200, so "Chat Completions ignores unknown keys there." (Q-s35, third-party)
- Basis: Vendor-documented (Google) and Filled (LLM Gateway, GoModel, Pydantic AI). Count: 3 adopters besides Google.

**Q52. `reasoning_text` plus `reasoning_opaque` (GitHub Copilot). One product.** See Q36.


### Part 3. Standardization efforts

**Q53. One published, governed specification for a neutral inference API exists: Open Responses. It standardizes OpenAI's Responses shape, not chat completions.**

- "Open Responses is an open-source specification and ecosystem for building multi-provider, interoperable LLM interfaces based on the OpenAI Responses API." (Q-s79)
- Governance: the charter page is headed "for Open Responses a Series of LF Projects, LLC" and "Adopted October 21, 2026". The TSC chair "will serve as the primary communication contact between the Project and Agentic AI Foundation (AAIF), a directed fund of The Linux Foundation." Code is Apache License 2.0, documentation is Creative Commons Attribution 4.0. (Q-s80)
- Maintainers, who are the initial voting TSC members, from `MAINTAINERS.md` at commit `dce9ddb91921427c2ba06901c7dfaf0cc7056799` (2026-10-29): "Steve Coffey, OpenAI", "Ankit Mathur, Databricks", "Anthony Liguori, Amazon", "Ben Burtenshaw, HuggingFace", "Devon Rifkin, Ollama", "Ji Huang, OpenAI", "Matt Apperson, OpenRouter", "Shaun Smith, HuggingFace", "Kedar Thakkar, Ramp". (Q-s76)
- Versions, from `CHANGELOG.md`: "2026-01-15 ... Launched Open Responses as an open, multi-provider specification with a shared schema, semantic streaming events, item lifecycle rules, and extensible tooling." and "2026-04-24 ... Added WebSocket transport for `/v1/responses`", "Added `POST /v1/responses/compact`", "Added the optional assistant-message `phase` field with `commentary` and `final_answer` values." (Q-s76)
- Launch partners according to an OpenAI developer-relations post dated 2026-01-15: "Honored to partner with Vercel, @LMStudio, Hugging Face, Ollama, NVIDIA AI, OpenRouter and vLLM in bringing this spec to life!" (Q-s81, third-party social post). No Anthropic or Google name appears in the maintainer list or the launch list.
- The project ships a conformance check: the site says "Validate your API using the acceptance tests" (Q-s79) and the repository README lists "Compliance tests: `bin/compliance-test.ts`" and "an interactive compliance tester in the docs site (`/compliance`)" (Q-s76).
- Basis: Filled (Open Responses project). OpenRouter holds one of nine maintainer seats.

**Q54. Open Responses defines reasoning replay as an opaque `encrypted_content` string on a `reasoning` item. Signed text and per-tool-call signatures fall outside the schema, and one gateway puts them in non-standard fields.**

- "`encrypted_content` (protected reasoning content): Providers MAY choose to never expose raw reasoning to the client. Instead, they can return an `encrypted_content` payload whose format and cryptographic properties are provider-specific. Clients are expected to treat this field as opaque and non-inspectable; it is intended for providers to protect reasoning content while allowing it to be round-trippable." (Q-s79)
- "`content` (raw reasoning content): Providers MAY surface the model's raw reasoning trace directly through the `content` field on `reasoning` items." "`summary` (reasoning summaries)". "All three fields are optional". (Q-s79)
- Opt-in: `IncludeEnum` value `"reasoning.encrypted_content"`, described as "includes encrypted reasoning content so that it may be rehydrated on a subsequent request." (Q-s76, `schema/configuration.tsp` lines 9 to 13)
- Input schema `ReasoningItemParam`: `id?`, `type: "reasoning"`, `summary`, `content?: null`, `encrypted_content?: string | null`. (Q-s76, `schema/items.tsp` lines 15 to 30) So a client may send back a summary and the encrypted payload, and may not send raw reasoning text as input under this schema.
- Reasoning control: `reasoning.effort` with `enum ReasoningEffortEnum { none, low, medium, high, xhigh, }`. The description map next to it also describes a `minimal` value that the enum itself does not list. (Q-s76, `schema/configuration.tsp` lines 56 to 74) No reasoning token-budget field appears in the schema: the word "budget" occurs only in two status descriptions about the output-token budget.
- A search of the schema folder for `signature` returned no match. (Q-s76)
- How a real gateway fills the gap: LLM Gateway's Responses route puts Gemini signatures in non-standard fields. "function call items carry `extra_content.google.thought_signature`. ... Gemini signatures are not converted into OpenAI `encrypted_content`." (Q-s47)
- Basis: Filled (Open Responses project).

**Q55. Open Responses defines cache-read reporting and a cache routing key. Breakpoints, TTLs, and cache-write counts are left to each implementer.**

- Request: "`prompt_cache_key?: BoundedString<64> | null;`" described as "A key to use when reading from or writing to the prompt cache." Usage: `InputTokensDetails` has only "`cached_tokens: integer;`" ("The number of input tokens that were served from cache."). (Q-s76, `schema/responses.tsp` lines 86 to 87 and 127 to 133)
- A search of the schema for `cache_write`, `breakpoint`, `ttl`, and `cache_control` returned no match. The word "cost" appears once, in a comment ("increase latency and cost"), and not as a field. (Q-s76)
- The only cache-related behavior rule is about tools: "`allowed_tools` ... is primarily a cache-preserving control surface: instead of mutating the `tools` field (which typically invalidates any prompt or schema caches), a client can use `allowed_tools` to dynamically narrow or expand the effective tool menu". (Q-s79)
- Basis: Filled (Open Responses project).

**Q56. Open Responses has an explicit extension rule: new item types, event types, and hosted tools get a provider-slug prefix. Extra fields on existing schemas are allowed, must be optional, and are called non-portable.**

- "Model providers may emit their own types of items not contained in the Open Responses spec. These items MUST be prefixed with the canonical provider slug", example `"type": "openai:web_search_call"`. (Q-s79)
- "New item types that are not part of this specification MUST be prefixed with the implementor slug (for example, `acme:search_result`)". "clients SHOULD be prepared to encounter unknown item types (for example, by ignoring them gracefully or treating them as opaque records)". (Q-s79)
- Hosted tools: `"type": "implementor_slug:custom_document_search"`. "The item's schema should allow for enough information to pass to the developer that it can be round-tripped back to the API for lossless rehydration on a followup request." (Q-s79)
- Fields: "implementors MAY extend existing schemas with additional, implementation-specific fields." Guidelines: "Keep all standard fields intact and unchanged in meaning; extensions MUST NOT alter required core behavior." "Prefer optional fields so that portable clients can ignore them safely." "Avoid building cross-vendor contracts on top of non-standard fields; instead, propose commonly-used patterns for inclusion in the core spec via the TSC process." (Q-s79)
- Compliance definition: "Any implementer can be considered to have a Open Responses-compliant API if they have an API that implements this spec directly or is a proper superset of Open Responses." (Q-s79)
- The spec defines no provider-routing object, no fallback model list, no cost field, and no vendor-native stop reason. (Q-s76: the words `provider`, `routing`, `fallback`, `finish_reason`, and `stop_reason` have zero matches in the `.tsp` schema files)
- Basis: Filled (Open Responses project).

**Q57. Open Responses errors, overflow, and state.**

- Error types: `server_error` (500), `invalid_request` (400), `not_found` (404), `model_error` (500), `too_many_requests` (429). Streaming: "Any error incurred while streaming will be followed by a `response.failed` event." (Q-s79)
- Overflow: "`"disabled"`: The server MUST NOT truncate any input. If the combined context exceeds the model's maximum context window, the request MUST fail with an error instead of silently dropping content." No error code is named for that case. (Q-s79)
- State: "When `previous_response_id` is provided, the server MUST load both the input and output associated with that prior response". On WebSocket with `store=false`, a missing response is the error code `previous_response_not_found`. (Q-s79)
- Stream end: "The stream MUST include a terminal response event (`response.completed`, `response.failed`, or `response.incomplete`). Servers MAY send the literal string `[DONE]` after that event." (Q-s79)
- Basis: Filled (Open Responses project). Also in workspace: `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30), which says "The Open Responses specification requires that overflow fail when truncation is disabled but names no error code." Confirmed live.

**Q58. Adoption of Open Responses by hosted gateways found today.**

- Vercel AI Gateway serves it at `/v1/responses` and positions it as the neutral choice: "**Want a provider-agnostic REST API?** Use OpenResponses." Its parameter list for the route is short: `model`, `input`, `stream`, `temperature`, `top_p`, `max_output_tokens`, `tools`, `tool_choice`, `reasoning` ("Reasoning configuration with `effort` level"), `providerOptions`. (Q-s9, Q-s5)
- Infron: "The LLM gateway is OpenAI-compatible, Anthropic-compatible, and OpenResponses-compatible". (Q-s58)
- Hugging Face documents a "Responses API (beta)" on its router. (Q17)
- Basis: Filled (Vercel AI Gateway, Infron, Hugging Face).

**Q59. MCP sampling is a neutral message format for model calls, and it is deprecated.**

- "Deprecated: The Sampling feature is deprecated as of protocol version `2026-07-28` (SEP-2577). ... New implementations SHOULD NOT adopt it; existing implementations SHOULD migrate to integrating directly with LLM provider APIs." Earliest removal: "First revision released on or after 2027-07-28". (Q-s82, Q-s83)
- Shape: `sampling/createMessage` with `messages` (role plus content blocks), `modelPreferences` (`hints`, `costPriority`, `intelligencePriority`, `speedPriority`), `systemPrompt`, `maxTokens`, `tools`, `toolChoice`. Content block types in the 2026-07-28 schema: text, image, audio, `tool_use`, `tool_result`. Results have `stopReason` values such as `endTurn` and `toolUse`. (Q-s82, Q-s84)
- The whole vocabulary is those five content block types, a model name, and a stop reason. Reasoning, signatures, cache markers, and token usage sit outside it. (Q-s82, by reading the page's examples)
- Basis: Filled (Model Context Protocol). Also in workspace: the 2026-09-30 compaction report says MCP sampling "is deprecated as of its 2026-07-28 protocol version". Confirmed live.

**Q60. The Agent Client Protocol does not define a model wire format. A proposal lets a client tell an agent which existing vendor protocol and base URL to use.**

- RFD "Configurable LLM Providers": "Add the ability for clients to discover and configure agent LLM providers (identified by `id`) via dedicated provider methods: `providers/list`, `providers/set`, `providers/disable`. This allows clients to route LLM requests through their own infrastructure (proxies, gateways, or self-hosted models)". (Q-s85)
- "`type LlmProtocol = "anthropic" | "openai" | "azure" | "vertex" | "bedrock" | string;`" with the comment "Well-known API protocol identifiers for LLM providers. This is an open string type". Configuration is `apiType`, `baseUrl`, `headers`. (Q-s85)
- Basis: Filled (Agent Client Protocol). The page is a request for discussion. Its acceptance status was not checked.

**Q61. Chat-completions extension names spread by copying between products. The search found one request to add a field to OpenAI's published spec, still open after ten months, and found neither a registry nor a conformance suite for such extensions.**

- The open request to add Google's field to OpenAI's published spec states the gap: "there is nowhere an attribute reflecting on the SDK change `extra_content.google.thought_signature` nor there is an indication about extensions for a specific provider." (Q-s78, third-party issue report, open since 2026-01-22)
- agentgateway's source says the same from the implementer's side: "There is no consistent standard for OpenAI compatible endpoints in how to express 'reasoning'". (Q-s56)
- A commentator at the Open Responses launch: "I was hoping for one based on their older Chat Completions API since so many other products have cloned the already, but basing it on Responses does make sense". (Q-s86, third-party blog)
- Searches run on 2026-11-22 that returned no such document: "standard specification neutral LLM inference API chat completions superset", "OpenAI-compatible extension registry", plus keyword reads of the Open Responses schema, the Vercel AI Gateway pages, and the Hugging Face Inference Providers index.
- Basis: Filled (negative finding across the sources above). Confidence: medium. A negative search result is weaker than a found document.


### Additional items found while tracing names

**Q62. Kilo Gateway's own API reference documents a plain OpenAI chat-completions schema with none of OpenRouter's extension fields, repairs tool-call history, and hides upstream billing errors.**

- Response type: `finish_reason: "stop" | "length" | "tool_calls" | "content_filter"`, `usage` with only `prompt_tokens`, `completion_tokens`, `total_tokens`. Model ids are `anthropic/claude-sonnet-4.5` style and response ids look like `gen-abc123`. (Q-s67)
- "The gateway automatically handles common tool calling issues: Deduplication: Removes duplicate tool calls with the same ID. Orphan cleanup: Removes tool result messages without matching tool calls. Missing results: Inserts placeholder results for tool calls without responses. ID normalization: Normalizes tool call IDs per provider requirements (Anthropic, Mistral)". (Q-s67)
- "When the gateway receives a 402 (Payment Required) from an upstream provider, it returns 503 to the client to avoid exposing internal billing details." Context overflow is a 400 with the message "This request exceeds the model's context window of 200000 tokens. Your request contains approximately 250000 tokens." (Q-s67)
- One endpoint is stated to forward to OpenRouter: "The gateway forwards each request to OpenRouter using the platform credential". (Q-s67, said of the `/systemone` route)
- A third-party client's documentation calls the gateway "OpenRouter-compatible". (Q-s87, third-party)
- Basis: Filled (Kilo Gateway).

**Q63. One gateway (Experiential) reports dropped parameters in the response body, which is a third style beside silent ignore and hard reject.**

- "Dropped with disclosure when the serving rung cannot honor it but the request is otherwise servable: the field is removed and named in the top-level response body array `x-experiential-ignored-parameters` (on a stream the same key rides in the chunk that carries the finish)." "Refused before the request is admitted, as a `400` with `code: unsupported_parameter` (no route serves it) or `code: invalid_parameter` (malformed) and `param` naming the field." (Q-s73)
- Vercel's equivalent is the `provider_metadata.gateway.warnings` array (Q7).
- Basis: Filled (Experiential).

## Conflicts and disagreements

Disagreements with the local Runvil documents:

- **Hugging Face is no longer an independent router (2026-10-28 survey, aggregators section).** The survey calls Hugging Face Inference Providers "the clearest reshaper", says "The compatible route is chat-only", and says its suffix design "is the exact opposite of OpenRouter's structured provider object". Since 2026-11-16 Hugging Face routes language models through OpenRouter, and its router serves Chat Completions, Responses, and Messages (Q15, Q17). The suffix grammar is still there (Q16). The live pages are right: the change is 19 days newer than the survey.
- **"Gemini's route loses thought signatures" (2026-10-28 survey, "What is provably lost").** Google's Vertex page documents that the chat-completions route returns `tool_calls[].extra_content.google.thought_signature` and returns a 400 when a required one is missing on replay (Q26). The older workspace document `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20) says Gemini "exposes signatures through `extra_content`", and that one is right. What the route does lose for a client that follows the OpenAI schema strictly is the signature, because the field is outside the schema.
- **vLLM and `reasoning_content` (2026-10-28 survey, "The extensions that became de facto standard").** The survey lists vLLM among servers whose output field is `reasoning_content`. vLLM's current documentation says the field is `reasoning` and that `reasoning_content` is the old name (Q37). llama.cpp, Fireworks, and DeepSeek still use `reasoning_content` (Q38).
- **Anthropic's compatibility layer and `response_format` (2026-10-28 survey, section 5.2).** The survey lists `response_format` as ignored. Today it "accepts only `json_schema` with `strict: true`" and other values return a 400. Audio and file parts and `n` other than 1 also return 400 now (Q29). The rest of the survey's list still holds.
- **Bedrock hosts (2026-10-28 survey, cloud wrappers section).** The survey says "AWS runs a second host `bedrock-mantle` for compatibility only". Today AWS serves the OpenAI-shaped routes on `bedrock-runtime` under `/openai/v1` and recommends that host, with `bedrock-mantle` as the fallback for features `bedrock-runtime` lacks (Q24). The survey's list of what the runtime host rejects still matches.
- **Bedrock stop reasons "where the vendor value never survives" (2026-10-28 survey).** Mostly right, with one qualification: `additionalModelResponseFieldPaths` lets a caller pull a vendor field such as `/stop_sequence` out of the raw response (Q20, Q21). The closed set also has nine values now.
- **LC17, `prompt_tokens_details.cache_write_tokens` as "the extension name OpenRouter uses".** The name is in OpenAI's own chat-completions reference today (Q30). It is a standard field that OpenRouter also fills, not only an extension. The survey's sentence that cost fields "exist nowhere in OpenAI's spec and are purely an aggregator invention" still holds for `usage.cost`.
- **"A structured provider-preference object ... is now the reference design for routing" (2026-10-28 survey).** The adopters found are API-compatible resellers and one clone (Q42). Vercel, Azure, Cloudflare, and Hugging Face each use a different shape. The evidence supports "copied by resellers", not "reference design" for the large platforms.
- **Three conventions for thinking through chat completions (2026-11-17 report on the Anthropic Messages API and harness survey).** That report names `thinking_blocks`, `reasoning_details`, and `reasoning_signature`. Two more exist: Google's `extra_content` keyed by vendor, which GoModel uses as its general container (Q51), and GitHub Copilot's `reasoning_text` with `reasoning_opaque` (Q36). The report credits `reasoning_details` to OpenRouter and Bifrost. Vercel AI Gateway, LLM Gateway, Infron, and MiniMax also use it (Q39).

Conflicts between sources:

- **Google's two pages on the OpenAI route's signature field.** The Vertex thought-signatures page shows `extra_content.google.thought_signature` in its Chat Completions examples (Q26). The Markdown export of the Gemini API thought-signatures page, in its "Signatures for OpenAI compatibility" section, shows only native `thoughtSignature` JSON and no `extra_content` text. The Gemini compatibility page only links to that section. Third-party reports of the wire format agree with the Vertex page. The Gemini export looks like a documentation rendering defect, but that is an inference.
- **Google's dummy signature values.** The Vertex page names one (`skip_thought_signature_validator`). The Gemini API page names two (`"context_engineering_is_the_way_to_go"` or `"skip_thought_signature_validator"`) (Q26).
- **Does one assistant message have one signature or many?** Requesty and agentgateway use a single `reasoning_signature` string per message (Q32, Q50). Bedrock, Vercel, LLM Gateway, LiteLLM, and Prisme.ai use a list of blocks, each with its own signature (Q3, Q19, Q31, Q39, Q49). Google attaches signatures to individual tool calls (Q26). These three placements are not interchangeable.
- **Same field name, different contents.** `reasoning_details` entries are `reasoning.text` and `reasoning.encrypted` objects at Vercel and LLM Gateway, and Anthropic-native `thinking` and `redacted_thinking` blocks at Prisme.ai (Q39). `usage.cost` is USD at LLM Gateway and RouterPlus and "points" at Sargalay (Q44). `cache_control` is shown on the message object at Vercel and on a content block at Infron and LiteLLM (Q46).
- **Cloudflare's own pages.** The Unified API page says the `/compat/chat/completions` endpoint is "Deprecated for single-model calls" (dated 2026-11-04), while the section index still describes connecting through "the unified API, provider-native endpoints, or WebSockets" (dated 2026-04-20) (Q12).
- **Finish reason spelling at Vercel.** The generation lookup route documents `content-filter` and `tool-calls` (Q10). The chat-completions examples use OpenAI's `stop`. The spelling on the chat-completions route for filter and tool cases was not shown on the pages read.
- **Kilo Gateway.** A third-party client calls it "OpenRouter-compatible". Kilo's own reference documents no OpenRouter extension field (Q62).

## Open questions blocked by closed source

Each question is about OpenRouter's translator. Each fill shows how another product does it and is not evidence of what OpenRouter does.

- **When `reasoning_details` comes back, how does the translator know where each thinking block sat among the text and tool-use blocks?** Tried: other gateways' documentation. Fill found: LLM Gateway adds a `block_index` to each entry and "replays thinking only in the position where Claude produced it" (Q31). Prisme.ai keeps Anthropic's native block entries (Q39). Bedrock Converse keeps reasoning as an ordered content block in the message, so position is never lost (Q19).
- **What happens to stored reasoning when the next request is served by a different provider or model?** Fill found: LLM Gateway "drops that provider's thinking blocks instead of failing the request" and "strips these entries from other upstream requests" (Q31). Prisme.ai forwards the blocks "only when this matches the model being called", using a `reasoning_model` tag it never sends upstream (Q39). Pydantic AI replays `extra_content` only when the stored provider name matches (Q26, Q51).
- **How are Gemini's signed text boundaries and per-call signatures represented in a message-level list?** Fill found: LLM Gateway uses `google_part` and `google_response` metadata on `reasoning.text` entries and also accepts `extra_content.google.thought_signature` on tool calls and text parts (Q31). Google's own route puts the signature on the tool call (Q26).
- **How is a neutral effort turned into a vendor budget, and a budget into an effort?** Fill found: Vercel's mapping table and share-of-max-output rule (Q4), Requesty's two conversion tables (Q32), and Google's own table for its compatible route (Q27).
- **Where does a gateway put automatic cache breakpoints, and what does it do with a TTL the target does not support?** Fill found: Vercel's two marker positions and its advisory `cache_ttl` with a warning (Q7). LiteLLM's rule that `control.ttl` is ignored toward OpenAI (Q46). Bedrock's `ValidationException` for an unsupported TTL (Q22).
- **How does a gateway tell the caller that a parameter or feature was dropped?** Fill found: Vercel's `provider_metadata.gateway.warnings` (Q7), Experiential's `x-experiential-ignored-parameters` array (Q63), Azure's `routing_config.require_parameters` filter (Q28), LLM Gateway's hard error for an unsupported budget (Q31).
- **How can the vendor's raw stop reason be exposed without a custom field name?** Fill found: Bedrock's JSON Pointer mechanism (`additionalModelResponseFieldPaths`) (Q20, Q21). No other gateway outside OpenRouter's clones exposes the raw value.
- **Does a request sent through Hugging Face's router keep OpenRouter's extension fields (`reasoning_details`, `provider`, `cache_control`) in both directions?** Tried: the Hugging Face routing, caching, chat-completion, Messages, and Responses pages. No fill found. The pages are silent (Q18).
- **What is Vercel's replay rule for `reasoning_details`?** Tried: the 23 downloaded Vercel pages. No fill found. Vercel's translator is also closed and its page gives no replay rule (Q3).

## Unverified

- **`provider_specific_fields` (LiteLLM).** From memory: LiteLLM returns provider-only data on the message and on tool calls under this name, including Gemini thought signatures. No source was read for it today, so it has no evidence item and no count in the adoption table.
- **Perplexity's `usage.cost` object.** From memory: Perplexity returns a cost object inside `usage`. Not fetched today.
- **Bedrock Converse signature binding.** The 2026-10-28 survey says the Converse signature "hashes all messages and must be replayed with all previous messages or the call errors". The three reference pages read today say only to "include the text and its signature unmodified". Not re-found.
- **MiniMax's own documentation.** The `reasoning_details` and `reasoning_split` quotes come from a reseller's API manual and a GitHub copy of a MiniMax guide. MiniMax's own page was not fetched.
- **GitHub Copilot's `reasoning_text` and `reasoning_opaque`.** Known only through two client projects. GitHub's own documentation of these fields was not found.
- **Ollama's response field on its OpenAI-compatible route.** The request side (`reasoning.effort`) is confirmed. The name of the field that holds reasoning text in the reply was not confirmed. Ollama's native API uses `thinking`.
- **xAI and Mistral reasoning field names.** The xAI guide URL returned a 404 and the Mistral page was not parsed. Neither is counted.
- **OpenTelemetry GenAI message schema.** Two attempts to download the JSON schema returned a 404 page. Its treatment of reasoning parts is not recorded here. The workspace's 2026-09-30 compaction report mentions these conventions.
- **Vercel AI SDK's provider specification and Hugging Face's task specification as standards.** Not examined in this file. Other evidence files on the AI SDK cover its model layer.
- **GitHub Models and Poe.** Named in the task as optional. Not fetched.
- **Which party used `cache_write_tokens` first, OpenRouter or OpenAI.** Not established.
- **Agent Client Protocol proposal status.** The "Configurable LLM Providers" page is a request for discussion. Whether it has been accepted was not checked.
- **Sources read as excerpts.** Sources marked "excerpt" in the Sources table were read as page text returned by the search tool on 2026-11-22, not through a separate fetch of the page. The quotes are exact as returned. For source code, generic type parameters were stripped by the tool.
- **Launch-partner list for Open Responses.** Taken from a social post and a blog. The maintainer list is from the repository and is firm.

## Workspace documents used

- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Read the sections on cloud platform wrappers, aggregators and routers, compatibility layers (5.2), reasoning (5.4), caching (5.5), and "The OpenAI shape as an industry default". Used as the baseline for Part 1 and for the claim list. Re-checked live: Hugging Face (changed), Bedrock hosts and Converse fields (partly changed), Anthropic's compatibility layer (partly changed), Gemini's compatible route (survey wrong on signatures), vLLM's field name (changed), `cache_write_tokens` (context changed). Not re-checked: Azure classic versus v1 routing detail, OCI, watsonx, Databricks, NVIDIA, Cohere.
- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20). Took the statement that Gemini's compatible route exposes signatures through `extra_content`. Re-checked live and confirmed (Q26).
- `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30). Took two statements: Open Responses requires overflow to fail when truncation is disabled and names no code, and MCP sampling is deprecated as of 2026-07-28. Both re-checked live and confirmed (Q57, Q59).
- Report on the Anthropic Messages API and harness survey (cabinet output, dated 2026-11-17 in its text, file date 2026-11-19). Took the list of three conventions for returning thinking through chat completions and the LiteLLM strict-upstream failure. Re-checked the three names live (Q39, Q49, Q50) and found two more conventions and four more adopters.
- `runvil-design`, `research/2026-09/2026-09-12-everruns-feature-survey.md` (2026-09-12). Noted only that a surveyed harness ships an "OpenResponses" driver. Not re-checked.

## Sources

"Excerpt" means the page text was returned by the search tool rather than fetched separately. All read dates are 2026-11-22.

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| Q-s1 | https://vercel.com/docs/ai-gateway/sdks-and-apis/openai-chat-completions (also reached as /docs/ai-gateway/openai-compat) | 2026-11-22 | gateway doc (Vercel) |
| Q-s2 | https://vercel.com/docs/ai-gateway/sdks-and-apis/openai-chat-completions/reasoning (page date 2026-09-08) | 2026-11-22 | gateway doc (Vercel) |
| Q-s3 | https://vercel.com/docs/ai-gateway/sdks-and-apis/openai-chat-completions/advanced (page date 2026-09-08) | 2026-11-22 | gateway doc (Vercel) |
| Q-s4 | https://vercel.com/docs/ai-gateway/sdks-and-apis/openai-chat-completions/chat-completions (page date 2026-09-08) | 2026-11-22 | gateway doc (Vercel) |
| Q-s5 | https://vercel.com/docs/ai-gateway/sdks-and-apis (page date 2026-10-07) | 2026-11-22 | gateway doc (Vercel) |
| Q-s6 | https://vercel.com/docs/ai-gateway/models-and-providers/reasoning (page date 2026-09-08) | 2026-11-22 | gateway doc (Vercel) |
| Q-s7 | https://vercel.com/docs/ai-gateway/models-and-providers/automatic-caching (page date 2026-09-11) | 2026-11-22 | gateway doc (Vercel) |
| Q-s8 | https://vercel.com/docs/ai-gateway/sdks-and-apis/anthropic-messages-api and its /reasoning and /advanced pages (page dates 2026-09-18, 2026-09-08) | 2026-11-22 | gateway doc (Vercel) |
| Q-s9 | https://vercel.com/docs/ai-gateway/sdks-and-apis/openresponses and its /reasoning page (page date 2026-09-08) | 2026-11-22 | gateway doc (Vercel) |
| Q-s10 | https://vercel.com/docs/ai-gateway/sdks-and-apis/rest-api (page date 2026-09-11) | 2026-11-22 | gateway doc (Vercel) |
| Q-s11 | https://developers.cloudflare.com/ai-gateway/usage/chat-completion/ (dateModified 2026-11-04) | 2026-11-22 | gateway doc (Cloudflare) |
| Q-s11a | https://developers.cloudflare.com/ai-gateway/usage/providers/anthropic/ (dateModified 2026-10-27) | 2026-11-22 | gateway doc (Cloudflare) |
| Q-s11b | https://developers.cloudflare.com/ai-gateway/llms-full.txt and https://developers.cloudflare.com/ai-gateway/llms.txt | 2026-11-22 | gateway doc (Cloudflare) |
| Q-s12 | https://developers.cloudflare.com/ai-gateway/usage/rest-api/ (dateModified 2026-10-09) | 2026-11-22 | gateway doc (Cloudflare) |
| Q-s13 | https://developers.cloudflare.com/ai-gateway/usage/web-search/ (dateModified 2026-06-26) | 2026-11-22 | gateway doc (Cloudflare) |
| Q-s14 | https://huggingface.co/docs/inference-providers/routing | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s15 | https://huggingface.co/docs/inference-providers/providers | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s16 | https://huggingface.co/docs/inference-providers/prompt-caching | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s17 | https://huggingface.co/docs/inference-providers/guides/messages-api | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s17a | https://huggingface.co/docs/inference-providers/guides/responses-api | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s18 | https://huggingface.co/blog/openrouter (published 2026-11-16), excerpt | 2026-11-22 | vendor announcement (Hugging Face) |
| Q-s18a | https://huggingface.co/changelog/inference-by-openrouter (dated 2026-11-17), excerpt | 2026-11-22 | vendor changelog (Hugging Face) |
| Q-s19 | https://huggingface.co/docs/inference-providers/index | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s20 | https://huggingface.co/docs/inference-providers/tasks/chat-completion | 2026-11-22 | gateway doc (Hugging Face) |
| Q-s21 | https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ReasoningContentBlock.html | 2026-11-22 | vendor doc (AWS) |
| Q-s22 | https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ReasoningTextBlock.html | 2026-11-22 | vendor doc (AWS) |
| Q-s23 | https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ReasoningContentBlockDelta.html | 2026-11-22 | vendor doc (AWS) |
| Q-s24 | https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html | 2026-11-22 | vendor doc (AWS) |
| Q-s25 | https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_CachePointBlock.html | 2026-11-22 | vendor doc (AWS) |
| Q-s26 | https://docs.aws.amazon.com/bedrock/latest/userguide/prompt-caching.html | 2026-11-22 | vendor doc (AWS) |
| Q-s27 | https://docs.aws.amazon.com/bedrock/latest/userguide/claude-messages-extended-thinking.html | 2026-11-22 | vendor doc (AWS) |
| Q-s28 | https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_TokenUsage.html | 2026-11-22 | vendor doc (AWS) |
| Q-s29 | https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-mantle.html (titled "Responses API") | 2026-11-22 | vendor doc (AWS) |
| Q-s30 | https://docs.aws.amazon.com/bedrock/latest/userguide/inference-chat-completions.html | 2026-11-22 | vendor doc (AWS) |
| Q-s31 | https://docs.cloud.google.com/vertex-ai/generative-ai/docs/migrate/openai/overview | 2026-11-22 | vendor doc (Google) |
| Q-s32 | https://docs.cloud.google.com/vertex-ai/generative-ai/docs/thought-signatures (also at /gemini-enterprise-agent-platform/models/thinking/thought-signatures) | 2026-11-22 | vendor doc (Google) |
| Q-s33 | https://ai.google.dev/gemini-api/docs/thought-signatures?generate-content=true (Markdown export) | 2026-11-22 | vendor doc (Google) |
| Q-s34 | https://ai.google.dev/gemini-api/docs/openai | 2026-11-22 | vendor doc (Google) |
| Q-s35 | https://github.com/pydantic/pydantic-ai/pull/10235 (merged 2026-10-14), excerpt | 2026-11-22 | issue report (third-party) |
| Q-s36 | https://github.com/deepseek-ai/deepseek-harness/discussions/10210, excerpt | 2026-11-22 | issue report (third-party) |
| Q-s37 | https://learn.microsoft.com/en-us/azure/foundry/how-to/develop/sdk-overview | 2026-11-22 | vendor doc (Microsoft) |
| Q-s38 | https://learn.microsoft.com/en-us/azure/foundry/openai/how-to/model-router-apis | 2026-11-22 | vendor doc (Microsoft) |
| Q-s39 | https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry | 2026-11-22 | vendor doc (Anthropic) |
| Q-s40 | https://platform.claude.com/docs/en/api/openai-sdk | 2026-11-22 | vendor doc (Anthropic) |
| Q-s41 | https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create | 2026-11-22 | vendor doc (OpenAI) |
| Q-s42 | https://developers.openai.com/api/docs/guides/prompt-caching | 2026-11-22 | vendor doc (OpenAI) |
| Q-s43 | https://docs.requesty.ai/features/reasoning | 2026-11-22 | gateway doc (Requesty) |
| Q-s44 | https://console.groq.com/docs/reasoning | 2026-11-22 | vendor doc (Groq) |
| Q-s45 | https://inference-docs.cerebras.ai/capabilities/reasoning | 2026-11-22 | vendor doc (Cerebras) |
| Q-s46 | https://docs.together.ai/docs/reasoning-models-guide | 2026-11-22 | vendor doc (Together AI) |
| Q-s47 | https://docs.llmgateway.io/features/reasoning | 2026-11-22 | gateway doc (LLM Gateway) |
| Q-s48 | https://docs.fireworks.ai/guides/reasoning | 2026-11-22 | vendor doc (Fireworks AI) |
| Q-s49 | https://pydantic.dev/docs/ai/models/github-copilot/, excerpt | 2026-11-22 | harness doc (third-party account of GitHub Copilot) |
| Q-s50 | zed-industries/zed at commit `be52d3b7`, `crates/copilot_chat/src/copilot_chat.rs`, excerpt | 2026-11-22 | harness source |
| Q-s51 | https://docs.llmgateway.io/features/cost-breakdown, excerpt | 2026-11-22 | gateway doc (LLM Gateway) |
| Q-s52 | vllm-project/vllm, branch `main` on 2026-11-22, `docs/features/reasoning_outputs.md` (raw file) | 2026-11-22 | local server doc |
| Q-s53 | https://api-docs.deepseek.com/guides/thinking_mode | 2026-11-22 | vendor doc (DeepSeek) |
| Q-s54 | ggml-org/llama.cpp, branch `master` on 2026-11-22, `tools/server/README.md` (raw file) | 2026-11-22 | local server doc |
| Q-s55 | https://docs.litellm.ai/docs/reasoning_content | 2026-11-22 | gateway doc (LiteLLM) |
| Q-s56 | agentgateway/agentgateway at commit `77e45c73`, `crates/llm/src/types/completions.rs`, excerpt | 2026-11-22 | gateway source |
| Q-s57 | https://evolink.ai/docs/en/api-manual/language-series/minimax-m3/minimax-m3-chat, excerpt | 2026-11-22 | reseller doc (third-party copy of MiniMax schema) |
| Q-s58 | https://infron.ai/models/openai/gpt-5-chat/api-reference | 2026-11-22 | gateway doc (Infron) |
| Q-s59 | vercel-labs/fx issue 1173, read at https://safaia.net/vercel-labs/fx/issues/1173, excerpt | 2026-11-22 | issue report (third-party) |
| Q-s60 | https://console.sargalay.com/docs, excerpt | 2026-11-22 | gateway doc (Sargalay) |
| Q-s61 | https://www.getmaxim.ai/bifrost/guides/providers/anthropic, excerpt | 2026-11-22 | gateway doc (Bifrost) |
| Q-s62 | https://raw.githubusercontent.com/muratcankoylan/Agent-Skills-for-Context-Engineering/HEAD/examples/interleaved-thinking/docs/interleavedthinking.md, excerpt | 2026-11-22 | third-party copy of a MiniMax guide |
| Q-s63 | https://docs.prisme.ai/llm-gateway/completions/create-a-chat-completion | 2026-11-22 | gateway doc (Prisme.ai) |
| Q-s64 | https://docs.ollama.com/api/openai-compatibility | 2026-11-22 | local server doc |
| Q-s65 | https://modelgates.ai/docs/api/reference/overview, excerpt | 2026-11-22 | gateway doc (ModelGates) |
| Q-s66 | https://docs.interactive.ai/llm-router, excerpt | 2026-11-22 | gateway doc (InteractiveAI) |
| Q-s67 | https://kilo.ai/docs/gateway/api-reference | 2026-11-22 | gateway doc (Kilo) |
| Q-s68 | https://docs.nano-gpt.com/migrate-from-openrouter, excerpt | 2026-11-22 | gateway doc (NanoGPT) |
| Q-s69 | https://modelgates.ai/docs/guides/features/plugins/overview and https://modelgates.ai/docs/api/api-reference/chat/send-chat-completion-request, excerpt | 2026-11-22 | gateway doc (ModelGates) |
| Q-s70 | zed-industries/zed, branch `main` on 2026-11-22, `crates/settings_content/src/language_model.rs`, excerpt | 2026-11-22 | harness source |
| Q-s71 | https://gproxy.leenhawk.com/guides/providers, excerpt | 2026-11-22 | gateway doc (third-party proxy) |
| Q-s72 | https://app.routerplus.com/docs/overview, excerpt | 2026-11-22 | gateway doc (RouterPlus) |
| Q-s73 | https://platform.experientiallabs.ai/docs/openai-compatibility, excerpt | 2026-11-22 | gateway doc (Experiential) |
| Q-s74 | https://docs.litellm.ai/docs/tutorials/prompt_caching, excerpt | 2026-11-22 | gateway doc (LiteLLM) |
| Q-s75 | https://docs.preloop.ai/guide/alibaba-model-studio/, excerpt | 2026-11-22 | gateway doc (third-party account of Alibaba) |
| Q-s76 | openresponses/openresponses at commit `dce9ddb91921427c2ba06901c7dfaf0cc7056799` (2026-10-29): `schema/*.tsp`, `MAINTAINERS.md`, `CHANGELOG.md`, `README.md` | 2026-11-22 | specification source |
| Q-s77 | https://gomodel.enterpilot.io/docs/advanced/extra-content, excerpt | 2026-11-22 | gateway doc (GoModel) |
| Q-s78 | https://github.com/openai/openai-openapi/issues/517 (opened 2026-01-22, open), excerpt | 2026-11-22 | issue report (third-party) |
| Q-s79 | https://www.openresponses.org/ and https://www.openresponses.org/specification | 2026-11-22 | specification |
| Q-s80 | https://www.openresponses.org/governance (charter adopted 2026-10-21) | 2026-11-22 | specification governance |
| Q-s81 | LinkedIn post by an OpenAI developer-relations employee dated 2026-01-15, and https://www.heise.de/en/news/Open-Responses-Unified-LLM-interface-instead-of-adapter-chaos-11150330.html (2026-01-22), excerpt | 2026-11-22 | blog (third-party) |
| Q-s82 | https://modelcontextprotocol.io/specification/2026-07-28/client/sampling, excerpt | 2026-11-22 | specification |
| Q-s83 | modelcontextprotocol/modelcontextprotocol, branch `main`, `docs/specification/2026-07-28/deprecated.mdx`, and https://modelcontextprotocol.io/seps/2577-deprecate-roots-sampling-and-logging, excerpt | 2026-11-22 | specification |
| Q-s84 | modelcontextprotocol/typescript-sdk at commit `cc4b4161`, `packages/core-internal/src/wire/rev2026-07-28/buildSchemas.ts`, excerpt | 2026-11-22 | SDK source |
| Q-s85 | https://agentclientprotocol.com/rfds/custom-llm-endpoint | 2026-11-22 | specification proposal |
| Q-s86 | https://simonwillison.net/2026/jan/15/open-responses/, excerpt | 2026-11-22 | blog (third-party) |
| Q-s87 | https://docs.openclaw.ai/providers/kilocode/, excerpt | 2026-11-22 | harness doc (third-party) |
| Q-s88 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28) | 2026-11-22 | workspace document |
| Q-s89 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20) and `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30) | 2026-11-22 | workspace document |
| Q-s90 | Report on the Anthropic Messages API and harness survey (cabinet output, 2026-11-17) | 2026-11-22 | workspace document |
