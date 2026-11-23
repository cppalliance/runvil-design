---
produced: 2026-11-22
title: OpenRouter field evidence part 1 - how TypeScript harnesses and SDKs special-case OpenRouter in source (ai-sdk-provider, opencode, Cline, Roo Code, Kilo Code, pi, Vercel AI SDK) - reasoning_details replay, thinking signatures, Gemini thought signatures, cache_control, usage and cost, tool-call ids, stream errors, provider routing, issue reports
---

# OpenRouter field evidence part 1 - how TypeScript harnesses and SDKs special-case OpenRouter in source

- What was read: seven repositories at pinned commits (OpenRouter's own `@openrouter/ai-sdk-provider`, opencode, Kilo Code, Cline, Roo Code, pi, and the Vercel AI SDK's generic OpenAI-compatible provider), plus 16 public issue and pull request pages (12 fetched in full, 4 as search-result excerpts), OpenRouter's OpenAPI spec and reasoning documentation from the local mirror, and Google's thought signature documentation.
- How many sources: 50 source keys (8 OpenRouter SDK source locations, 22 harness source locations, 1 OpenRouter spec, 1 OpenRouter doc page, 15 issue report keys, 1 vendor doc, 1 third-party status report, 1 key for two workspace documents). The file holds 68 evidence items and 29 candidate matrix rows. 187 special cases were counted across the six harness and SDK projects.
- Read date: 2026-11-22 for everything. All reads were public and unauthenticated. Nothing was run against a live model.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Signed thinking replay on chat completions | Anthropic | A thinking block must come back with its `signature` unchanged | Assistant `reasoning_details[]` entries `{type: "reasoning.text", text, signature, format: "anthropic-claude-v1", index}`. Streamed as many text deltas, then one delta with only `signature` | The client has to rebuild each block from deltas. `index` is unusable for that. A lost signature gives either a 400 or silently dropped thinking (two accounts, see Conflicts) | Caller holds state. Round trip must be exact. Values are opaque. Works on the chat route if the stream assembler is right | OR-documented | F1, F2, F3, F7, F40, F46 |
| Several signed thinking blocks in one answer | Anthropic (Claude 5.x progress updates) | Each block has its own signature and position | Separate `reasoning.text` runs with `index: 0`, `index: 1`, each closed by its own signature-only delta, no `id` | Position among text and tool calls is flattened: all details sit in one array beside `content` and `tool_calls` | Caller holds state. Merging two blocks under one signature gives a 400. A block boundary is "a signature arrived" | OR-inferred (issue report plus SDK fix) | F3, F47 |
| Thinking with parallel tool calls | Anthropic | One thinking block before the tool_use blocks | One `reasoning_details` array per assistant message | The AI SDK part model has no message-level slot, so clients copy the array onto parts and must send it once | Caller holds state. Sending the array twice is rejected. A message-level opaque field avoids the copy problem | OR-documented | F4, F9, F52 |
| Gemini thought signature replay | Google | Signature must return in the same part. First function call of each step needs it or 400 | `reasoning.encrypted` entry `{data, id: <tool call id>, format: "google-gemini-v1", index}`. Gemini `reasoning.text` has no signature | Per-part position becomes an `id` link to a tool call. Text thoughts cannot be verified by the client | Caller holds state. Round trip must be exact. Signature is tied to the upstream provider (AI Studio or Vertex), so the caller must pin the provider | OR-inferred (OpenRouter staff comment in issue) | F8, F10, F44, F45 |
| History from another model sent to Gemini | Google | Dummy signature `skip_thought_signature_validator` is allowed as a last resort | Client injects `{type: "reasoning.encrypted", data: "skip_thought_signature_validator", id: <first tool call id>, format: "google-gemini-v1", index: 0}` | OpenRouter does not synthesize this. Missing signatures surface as upstream 400s | Vendor-specific value chosen by the caller, or by a gateway translator that knows the target is Gemini | Filled (Roo Code, Cline) | F31, F32, F48, F58 |
| Gemini 3 tool calls end with `stop` | Google | Finish reason is STOP even when function calls are present | `finish_reason: "stop"` with `tool_calls` present | The normalized finish reason is wrong for agent loops | A translator can fix this without the caller (tool calls present means tool-calls) | OR-documented | F11 |
| OpenAI encrypted reasoning items | OpenAI, Azure, xAI | Reasoning items have ids and encrypted content tied to the org or project | `reasoning.encrypted` with `id: "rs_..."`, `format: "openai-responses-v1"`, plus `reasoning.summary` deltas that all report `index: 0` | Duplicate ids across turns are rejected. Content minted under one upstream key fails under another | Caller holds state. Exact round trip. Dedupe by id is needed. Some clients strip `id` or drop the blobs | OR-documented | F5, F6, F33, F43, F50 |
| Empty reasoning marker | DeepSeek V4 | Reasoning content must be present on every assistant turn | `reasoning_details: []` returned and expected back. Some clients send `reasoning_content: ""` | An empty array has meaning, so "absent" and "empty" differ | Caller must keep the empty array. Opaque round trip covers it | OR-documented | F12, F38 |
| Reasoning format and type vocabulary | All | n/a | Spec enum `ReasoningFormat` has 12 values and a fourth type `reasoning.server_tool_call` | Clients that validate against an older list drop entries they do not know | Opaque storage must not validate the contents | OR-documented | F13, F14 |
| Reasoning effort and budget | All reasoning models | Each vendor has its own control | `reasoning: {effort, max_tokens, enabled, exclude}`. `effort` values `max, xhigh, high, medium, low, minimal, none` | `max_tokens` goes upstream as the thinking budget and upstream rules still apply. Per-model exceptions remain | Neutral effort translates. Budget needs clamping below the output limit. Some models reject "off" | OR-inferred (client code) | F15, F16, F27, F35, F39 |
| Cache breakpoints | Anthropic, Alibaba Qwen | Up to 4 `cache_control` markers | `cache_control: {type: "ephemeral", ttl?: "5m"|"1h"}` on content parts, on tools, and at the top level of the body | Other upstreams cache without markers. Gemini uses only the last marker | Caller (or gateway) decides marker placement from the model id prefix. No server state | OR-inferred (client code) | F17, F18, F19, F28, F34, F36 |
| Cache hit stability | Anthropic and others | Byte-identical prefix | Same request body rules. `session_id` body field or `x-session-id` header for sticky routing | A re-serialized tool argument object with a different key order misses the cache | Caller must keep stored arguments byte-stable, and send a session id | OR-documented | F20, F28, F36 |
| Usage detail fields | All | Vendor-specific | `usage.prompt_tokens_details.cached_tokens`, `.cache_write_tokens`, `completion_tokens_details.reasoning_tokens` | Some upstreams put usage elsewhere or leave fields empty | Gateway-only normalization | OR-documented | F21, F29, F37 |
| Billed cost | OpenRouter | n/a | `usage.cost`, `usage.is_byok`, `usage.cost_details.upstream_inference_cost` (also on the Anthropic Messages route in `message_delta` usage) | With BYOK, `cost` is only OpenRouter's fee. Catalog prices differ from billed prices when routing falls back | Reported cost is the only exact figure. Gateway-only | OR-inferred (client code, issue report) | F21, F22, F37, F49 |
| Finish reasons | All | Vendor-specific stop reasons | `finish_reason` plus `native_finish_reason`. Values seen: `stop`, `length`, `tool_calls`, `content_filter`, `error`, `network_error`, null | Null with tool calls present. `error` as a finish reason | Translator normalizes. Caller needs the raw value only for diagnostics | OR-inferred (client code) | F11, F23, F30 |
| Errors after HTTP 200 | All | n/a | Three shapes: a body with top-level `error`, a choice with `error` and `finish_reason: "error"`, and a stream chunk with top-level `error {code, message, metadata.raw, metadata.provider_name}` | The top message is often only "Provider returned error". The real text is in `metadata.raw` | Gateway-only. Retry rules need string matching | OR-documented | F23, F24, F25, F30 |
| Tool call stream repair | Many upstreams | n/a | Standard `tool_calls` deltas | Ids can be empty, null or repeated. `index` can be absent or reused. `arguments` can be absent or sent twice | Translator must repair ids and assemble arguments. No caller knowledge needed | OR-documented | F26 |
| Fine-grained tool streaming | Anthropic | `eager_input_streaming` per tool, off by default | OpenRouter sets it to true on every tool of a streamed request and ignores the client's value | Tool arguments can arrive as invalid JSON | The caller cannot opt out through OpenRouter. A direct translator could | OR-inferred (issue report quoting OpenRouter support) | F51 |
| Strict structured output | OpenAI and others | `json_schema.strict` | `response_format: {type: "json_schema", json_schema: {strict}}` | `strict: true` removes providers that lack it and can end in a 404. Response healing works only without streaming | Routing side effect of a request field | OR-documented | F27 |
| Provider routing | OpenRouter | n/a | `provider: {order, only, ignore, allow_fallbacks, require_parameters, data_collection, zdr, quantizations, sort, max_price}`, `models[]` | Fallback to another upstream breaks provider-scoped state (Gemini signatures, prices) | The caller has to know and pin the upstream provider in some cases | OR-inferred (client code, issue report) | F27, F45, F49 |
| Attribution and key headers | OpenRouter | n/a | `HTTP-Referer`, `X-Title` or `X-OpenRouter-Title`, `X-OpenRouter-Categories`, `X-Provider-API-Keys`, `x-session-id`, `x-anthropic-beta` | n/a | Static gateway configuration | OR-inferred (client code) | F28 |
| Model metadata | OpenRouter | n/a | `GET /models` (`supported_parameters`, `pricing.input_cache_read`, `top_provider.max_completion_tokens`, `reasoning.supported_efforts`, `reasoning.mandatory`), `GET /models/{id}/endpoints`, `GET /models/user` | Every harness still patches metadata per model id | Gateway-only, with a per-model override table | OR-inferred (client code) | F35, F39 |
| Context window pre-check | OpenRouter | n/a | 400 "This endpoint's maximum context length is X tokens. However, you requested about Y tokens" before routing | OpenRouter counts input with its own tokenizer, up to 12 percent above the provider's count | Caller or gateway must leave margin when setting `max_tokens` | OR-inferred (issue report, client code) | F41, F53 |
| Second native route for Claude | Anthropic | Messages API | `POST https://openrouter.ai/api/v1/messages` | pi left chat completions for all `anthropic/*` models to keep signed thinking | A harness chose a bypass path over repairing the chat path | Filled (pi) | F40, F46 |
| Responses route for OpenAI reasoning | OpenAI | Responses API with encrypted reasoning | `POST /responses` with `store: false`, `include: ["reasoning.encrypted_content"]` | n/a | opencode ships a second route beside the chat route | Filled (opencode) | F16 |
| Generic OpenAI-compatible client | n/a | n/a | n/a | Reads only `reasoning_content` or `reasoning` text. `reasoning_details`, `cache_write_tokens`, `cost` and `finish_reason: "error"` are not parsed | A client without the extension still works but loses signed reasoning | Filled (Vercel AI SDK) | F42, F43 |
| File parse annotations | OpenRouter | n/a | `message.annotations[]` entries `{type: "file", file: {hash, name, content}}` sent back on the assistant message | n/a | A second opaque round-trip field | OR-documented | F54 |
| Encrypted reasoning placeholder | OpenRouter | n/a | Reasoning text can contain `[REDACTED]` | Display text is polluted | Display-only cleanup | OR-inferred (client code) | F55 |
| Image output | Google and others | n/a | Request needs `modalities: ["image", "text"]`, images arrive in `message.images[]` | n/a | Request flag chosen from model metadata | OR-inferred (client code) | F56 |

## Evidence

Repository homes and pins, confirmed by `git ls-remote` and an HTTP check on 2026-11-22:

- `OpenRouterTeam/ai-sdk-provider` exists under that name. Commit `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33`, 2026-11-20. Package version 3.1.0 plus 35 unreleased changesets.
- `sst/opencode` answers with HTTP 301 to `anomalyco/opencode`. Commit `71cd8c064c8b93c31604c4c4074adb8b45790fc1`, 2026-11-22.
- `cline/cline` exists. Commit `8075d2e35cf5cad67c24f180c80a05d311b74a3e`, 2026-11-20, release v4.1.27.
- `RooCodeInc/Roo-Code` exists and is frozen. Commit `b867ec9145750d0ae1ff7f02d35406e9bf2a0b16`, 2026-05-15 (see F68).
- `Kilo-Org/kilocode` exists. Commit `8045e72da2cf057e42ed70820891d5ca11aa3f4b`, 2026-11-22. Its `.opencode-version` file reads `v1.18.26`: it is a fork of opencode.
- `badlogic/pi-mono` answers with HTTP 301 to `earendil-works/pi`. Commit `9c4652688590abb836b40ebd1e4725ecf9a7c7d1`, 2026-11-22.
- `vercel/ai` exists. Commit `d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d`, 2026-11-20.

Two of the projects no longer look like the task description expected, and that is itself a finding:

- opencode at HEAD replaced `@openrouter/ai-sdk-provider` with its own native provider. `packages/core/src/aisdk-native.ts` line 131 maps `"@openrouter/ai-sdk-provider": "@opencode/ai/providers/openrouter",`. The old `provider/transform.ts` file is gone from opencode and survives in the Kilo Code fork (F65).
- Cline at HEAD moved its providers into `sdk/packages/llms` and talks to OpenRouter through the Vercel AI SDK's generic `createOpenAICompatible` provider (F43). The old `openrouter-stream.ts` file is gone.

### A. OpenRouter's own AI SDK provider: reasoning replay

**F1. The reference client rebuilds `reasoning_details` from stream deltas by type transition, because `index` is not usable.**

- Quote (`src/chat/index.ts` lines 891-896): "Accumulate reasoning_details to preserve for multi-turn conversations. Merge consecutive same-type text/summary deltas into a single entry. Providers (e.g. OpenAI `openai-responses-v1`) stream a logical reasoning block as many small deltas" and "so `index` cannot be used to reassemble the array".
- Code (lines 906-929): a `reasoning.text` delta is appended to the previous entry when `lastDetail?.type === ReasoningDetailType.Text && !lastDetail.signature`, with `lastDetail.text = (lastDetail.text || '') + (detail.text || '')`. Otherwise `accumulatedReasoningDetails.push({ ...detail })`.
- Quote (lines 945-946): "Encrypted details are opaque blobs (each has its own id); they are discrete and must be pushed as-is, never merged."
- Source: F-s1. Basis: OR-documented.
- Implication (OR-inferred from this code): OpenRouter streams `reasoning_details` as fragments, not as whole entries, and does not give the client a reliable key for reassembly. A client that stores raw deltas unmerged sends back an array that breaks replay.

**F2. The signature arrives last and alone, and can arrive after the visible text has started.**

- Quote (lines 961-963): "Emit even when detail.text is empty/undefined" and "a signature-only delta (no text, just signature) must still be emitted so that the signature propagates to the reasoning part's providerMetadata."
- Quote (lines 951-954): "Only emit reasoning events if text content has not started yet. Late-arriving reasoning_details (e.g. a signature-only delta after text has begun) are still accumulated above for multi-turn roundtrip via the finish event's providerMetadata, but must not start a new reasoning block".
- Quote (lines 998-1001): "Always include accumulated reasoning_details so the AI SDK can update the reasoning part's providerMetadata with the correct signature.  The signature typically arrives in the last delta, but reasoning-start only carries the first delta's metadata."
- Source: F-s1. Basis: OR-documented.
- Confidence: high for the wire order. Pull request 608 (F60, still open) says the late-details path still loses data once the stream is serialized.

**F3. A signature closes its block: several signed Anthropic thinking blocks in one answer must stay separate entries.**

- Quote (lines 907-909): "A signature closes the text detail it signs: once the last text detail is signed, the next text delta starts a new block (e.g. multiple signed Anthropic thinking blocks)."
- Quote (lines 918-919): "Only copy defined values: assigning `undefined` keys breaks AI SDK prompt validation on replay (#540)."
- Changeset `split-signed-reasoning-blocks.md`: "Fix streaming: keep separately signed `reasoning.text` blocks (e.g. multiple Anthropic thinking blocks in one answer) as separate `reasoning_details` entries instead of merging them under the first signature, which made replay fail with \"Invalid signature in thinking block\"."
- Source: F-s1, F-s7. Commit `5fe77f6`, 2026-10-21. Basis: OR-documented.

**F4. Storage shape in the reference client: the whole array is copied into `providerMetadata.openrouter.reasoning_details` on several parts, and only the first tool call gets it.**

- Non-streaming (lines 400-413): each `reasoning.text` detail becomes a part `{ type: 'reasoning', text: detail.text, providerMetadata: { openrouter: { reasoning_details: [detail] } } }`. Each tool call part (lines 484-496) gets `providerMetadata: { openrouter: { reasoning_details: reasoningDetails } }` only when `!reasoningDetailsAttachedToToolCall`.
- Quote (lines 467-468): "Only attach reasoning_details to the first tool call to avoid duplicating thinking blocks for parallel tool calls (Claude)".
- Streaming: `reasoning-end` (lines 1004-1008), the first `tool-call` (lines 1344-1350) and the `finish` event (line 1405) all get `reasoning_details: accumulatedReasoningDetails`, the same mutable array object.
- Response-level shape (`src/schemas/provider-metadata.ts` lines 37-70): `{ provider: string, reasoning_details?: ReasoningDetailUnion[], annotations?: FileAnnotation[], usage: {...} }`.
- Source: F-s1, F-s3. Basis: OR-documented.
- Implication (OR-inferred from this code): OpenRouter's wire unit is one array per assistant message. The AI SDK has no message-level slot for it, so the client spreads copies over parts and then has to pick one copy on replay (F9). Three bugs came from that spread: duplicates (F52), stale copies (F60, F63) and lost signatures (F59).

**F5. OpenAI summary deltas all report `index: 0` and must be concatenated.**

- `e2e/issues/issue-519-streaming-summary-merge.test.ts` lines 5-11: "During streaming, consecutive `reasoning.summary` deltas were pushed to the accumulated reasoning_details as-is (one entry per delta) instead of being merged the way consecutive `reasoning.text` deltas are. Providers such as OpenAI `openai-responses-v1` stream a single logical summary as many small deltas that all carry `index: 0`, so the array fragmented into one entry per delta. Re-submitting that fragmented array on the next turn breaks multi-turn reasoning round-tripping (cf. langchain-ai/langchain#36400)."
- Source: F-s8. Commit `e96af18`, 2026-07-21. Basis: OR-documented.

**F6. The client dedupes `reasoning_details` across all assistant messages in a request, and says its key rule copies OpenRouter's server code.**

- Quote (`src/chat/convert-to-openrouter-chat-messages.ts` lines 69-72): "Track reasoning_details across all messages in this conversion to prevent duplicates. This fixes issue #254 where the same reasoning ID appears in multiple assistant messages during multi-turn conversations, causing the API to reject the request with \"Duplicate item found with id\" error."
- Quote (`src/utils/reasoning-details-duplicate-tracker.ts` lines 13-17): "The canonical key logic matches the OpenRouter API's deduplication exactly (see openrouter-web/packages/llm-interfaces/reasonings/duplicate-tracker.ts): - Summary: key = summary field - Encrypted: key = id field (if truthy) or data field - Text: key = text field (if truthy) or signature field (if truthy)".
- Source: F-s2, F-s4. Basis: OR-documented. This is the only place in the repository that names a file in OpenRouter's closed source.
- Implication (OR-inferred from the comment): OpenRouter's server also dedupes reasoning details, keyed by summary text, encrypted `id` or `data`, and reasoning text or signature. Confidence medium: it is a code comment, not a spec.

**F7. Unsigned Anthropic reasoning is removed before sending, and plain reasoning text is sent only when valid details go with it.**

- Quote (lines 322-325): "Anthropic (issue #423): Uses an explicit `signature` field on reasoning.text entries. When the signature is lost during message serialization, custom pruning, or DB storage, sending the entry back causes \"Invalid signature in thinking block\"."
- Code (lines 344-356): entries of type `reasoning.text` whose `format` is `anthropic-claude-v1` or `google-gemini-v1` are kept only when `!!detail.signature`. A missing `format` counts as Anthropic: `const format = detail.format ?? DEFAULT_REASONING_FORMAT;`.
- Quote (lines 394-397): "Sending reasoning without reasoning_details causes the API to construct thinking blocks without valid signatures, which Anthropic rejects with \"Invalid signature in thinking block\" (issue #423)."
- Code (lines 402-405): `reasoning && finalReasoningDetails && finalReasoningDetails.length > 0 ? reasoning : undefined`.
- Source: F-s2. Commits `a921eb7` (2026-03-13) and `16db454` (2026-03-16). Basis: OR-documented.
- Implication: a per-format rule lives in the client. The client reads `format` and `signature` to decide what is safe to send, so the array is not fully opaque to it.

**F8. Gemini reasoning text is always removed on replay.**

- Quote (lines 327-334): "Google Gemini (issue #418): Does NOT use the SDK `signature` field, but Google internally signs thought tokens. When reasoning text is modified during roundtripping (serialization, encoding changes, field reordering), Google rejects with \"Corrupted thought signature\". Since the SDK cannot verify whether the text is intact, Gemini reasoning.text entries are always stripped on roundtrip".
- Quote (lines 336-337): "Other formats (OpenAI, xAI, Azure) and non-text detail types pass through unchanged."
- Source: F-s2. Commit `aa7b8c6`, 2026-04-08. Basis: OR-documented.
- Confidence note: OpenRouter staff later traced the Gemini failures to upstream provider switching, not to client changes (F45). So this strip is a defensive guess that stayed in the code.

**F9. What the reference client actually sends for an assistant turn.**

- Code (lines 407-417): `messages.push({ role: 'assistant', content: hasPartCacheControl ? textParts : text || null, tool_calls: toolCalls.length > 0 ? toolCalls : undefined, reasoning: effectiveReasoning, reasoning_details: finalReasoningDetails, annotations: messageAnnotations, cache_control: getCacheControl(providerOptions), });`
- Quote (lines 304-306): "Use message-level reasoning_details if available, otherwise find from parts Priority: message-level > first tool call > first reasoning part This prevents duplicate thinking blocks when Claude makes parallel tool calls".
- Source: F-s2. Basis: OR-documented.
- Implication: the wire has one flat array per assistant message, with text in `content` and calls in `tool_calls`. Where a thinking block sat among text and tool calls is not expressed in the request.

**F10. Encrypted details are stored and replayed but never shown.**

- Quote (`src/chat/index.ts` lines 432-435): "Encrypted reasoning is an opaque blob for multi-turn roundtripping. It is preserved in response-level providerMetadata.openrouter.reasoning_details and does not produce a visible reasoning content part."
- Changelog 2.4.1: "Stop emitting `[REDACTED]` as visible reasoning text for encrypted reasoning details."
- Source: F-s1, F-s7. Basis: OR-documented.

**F11. Finish reasons need three client fixes.**

- Quote (lines 549-551): "Fix for Gemini 3 thoughtSignature: when there are tool calls with encrypted reasoning (thoughtSignature), the model returns 'stop' but expects continuation. Override to 'tool-calls' so the SDK knows to continue the conversation."
- Quote (lines 564-565): "Fix for #420: When finishReason is 'other' (unknown/missing) but tool calls were made, infer 'tool-calls' so agentic loops continue correctly."
- `e2e/issues/issue-166-finish-reason-null.test.ts` lines 5-10: "Some models return `finish_reason: null` in streaming responses even when tool calls are present." and "Models mentioned: google/gemini-2.5-flash, various free-tier models."
- `src/utils/map-finish-reason.ts` lines 14-28 maps `stop`, `length`, `content_filter`, `function_call`, `tool_calls` and `error`, and everything else to `other`. Changeset `finish-reason-error.md`: "A response that ends with `error` and contains tool calls is no longer reported as `tool-calls`".
- Source: F-s1, F-s5, F-s7, F-s8. Basis: OR-documented.
- Implication (OR-inferred from this code): OpenRouter's normalized `finish_reason` is not reliable for the tool-call case. It passes Gemini's stop through as `stop` and can send null.

**F12. An empty `reasoning_details` array is a signal that must go back.**

- Quote (lines 1401-1404): "Always include reasoning_details in finish metadata, even when empty. Some providers (e.g. DeepSeek V4) return reasoning_details: [] on turns where they produced no visible reasoning tokens, and they require the field to be sent back in subsequent turns to maintain conversation state."
- Changelog 2.8.1: "They require this empty array to be sent back in subsequent requests to maintain conversation state; omitting it causes 4xx errors on follow-up turns."
- Source: F-s1, F-s7. Commit `4588197`, 2026-04-25. Basis: OR-documented.

**F13. The reference client knows 6 formats and 3 detail types, and drops any entry outside them.**

- `src/schemas/format.ts` lines 1-8: `Unknown = 'unknown'`, `OpenAIResponsesV1 = 'openai-responses-v1'`, `AzureOpenAIResponsesV1 = 'azure-openai-responses-v1'`, `XAIResponsesV1 = 'xai-responses-v1'`, `AnthropicClaudeV1 = 'anthropic-claude-v1'`, `GoogleGeminiV1 = 'google-gemini-v1'`. Lines 10-12: "Anthropic Claude was the first reasoning that we're passing back and forth" and `export const DEFAULT_REASONING_FORMAT = ReasoningFormat.AnthropicClaudeV1;`.
- `src/schemas/reasoning-details.ts` lines 5-9 lists `reasoning.summary`, `reasoning.encrypted`, `reasoning.text`. Line 14: `format: z.enum(ReasoningFormat).nullish(),`. Lines 58-65: entries that fail the union become null through `z.unknown().transform(() => null)` and are filtered out.
- `src/chat/signature-roundtrip.test.ts` lines 636-639: "If the server adds a new format that the SDK doesn't know about yet, the SDK should NOT silently drop ALL reasoning_details. Valid entries should survive even if one entry has an unknown format." Lines 735-736: "azure-openai-responses-v1 exists in the server but was missing from SDK enum."
- Source: F-s3, F-s8. Basis: OR-documented.

**F14. OpenRouter's spec lists 12 formats and a fourth detail type that the reference client does not know.**

- OpenAPI `components.schemas.ReasoningFormat.enum`: `"unknown", "openai-responses-v1", "azure-openai-responses-v1", "bedrock-openai-responses-v1", "bedrock-xai-responses-v1", "xai-responses-v1", "meta-responses-v1", "google-vertex-meta-responses-v1", "google-vertex-xai-responses-v1", "openrouter-stealth-v1", "anthropic-claude-v1", "google-gemini-v1", null`, with `"x-speakeasy-unknown-values": "allow"`.
- `ReasoningDetailUnion` discriminator mapping has four types, including `reasoning.server_tool_call`, described as "Record of an OpenRouter server-tool invocation (e.g. openrouter:fusion), carried in reasoning_details so a prior tool call can be rehydrated into a later turn of the same conversation."
- Source: F-s9. Basis: OR-documented.
- Consequence, by comparing F13 with F14: at the pinned commit, a detail with format `bedrock-openai-responses-v1`, `bedrock-xai-responses-v1`, `meta-responses-v1`, `google-vertex-meta-responses-v1`, `google-vertex-xai-responses-v1` or `openrouter-stealth-v1`, or with type `reasoning.server_tool_call`, fails the SDK's schema and is dropped when the response is parsed. Confidence medium-high: it follows from the two files, and I did not run the SDK.

**F15. Reasoning control in the reference client.**

- `src/types/index.ts` lines 21-38: `reasoning?: { enabled?: boolean; exclude?: boolean; } & ( | { max_tokens: number; } | { effort: | 'max' | 'xhigh' | 'high' | 'medium' | 'low' | 'minimal' | 'none'; } );`
- `src/utils/resolve-reasoning.ts` lines 15-18: "Call-level `reasoning` replaces the model-level effort/max_tokens but keeps the model-level `enabled`/`exclude`. It is ignored when `providerOptions.openrouter.reasoning_effort` is set, because OpenRouter rejects conflicting `reasoning_effort` and `reasoning.effort` values."
- Source: F-s6. Basis: OR-documented.

### B. OpenRouter's own AI SDK provider: caching, usage, tools, errors, options

**F17. `cache_control` is read from four option spellings and written per content block.**

- `convert-to-openrouter-chat-messages.ts` line 31: `export type OpenRouterCacheControl = { type: 'ephemeral'; ttl?: '5m' | '1h' };`. Lines 40-43 read `openrouter?.cacheControl ?? openrouter?.cache_control ?? anthropic?.cacheControl ?? anthropic?.cache_control`.
- System messages are always sent as a one-element text block array so the marker can sit on the block (lines 79-88). Changelog 2.2.0 shows the before and after: message-level `"cache_control": { "type": "ephemeral" }` beside a string `content` became block-level inside `content: [{ type: "text", text, cache_control }]`.
- Changelog 2.0.3: "When message-level cache_control is set on a user message with multiple parts, it now only applies to the last text part."
- Markers are also written on `image_url`, `video_url`, `input_audio` and `file` parts, on assistant text parts, on assistant messages and on tool messages (lines 158, 175, 184, 220, 267, 416, 435-437).
- Source: F-s2, F-s7. Basis: OR-documented.

**F18. Two more placements: top-level automatic caching and per-tool markers.**

- `src/types/openrouter-chat-settings.ts` lines 128-132: "Enable Anthropic automatic prompt caching by setting a top-level cache_control directive on the request body." and "Only works with Anthropic models through OpenRouter."
- Changeset `tool-cache-control.md`: "Tool-level `cache_control` is passed through as-is. OpenRouter does not document it for the OpenAI-format `tools` array; Anthropic supports it on the last tool definition, so place the breakpoint there."
- Source: F-s6, F-s7. Basis: OR-documented.
- Implication: the client offers no neutral cache setting. The caller writes Anthropic's own marker, including Anthropic's TTL values.

**F20. Tool call arguments are serialized with sorted keys to keep cache prefixes stable.**

- Changelog 2.6.0: "When the AI SDK deserializes and re-serializes tool call arguments across turns, the key insertion order may change. `JSON.stringify()` follows insertion order, so semantically identical objects could produce different strings, causing cache misses."
- Code (line 277): `arguments: deterministicStringify(part.input),`.
- Source: F-s2, F-s7. Commit `44ba7ec`, 2026-04-13. Basis: OR-documented.

**F21. Usage and cost fields the reference client reads.**

- `src/chat/schemas.ts` lines 11-39: `prompt_tokens`, `prompt_tokens_details.cached_tokens`, `prompt_tokens_details.cache_write_tokens`, `completion_tokens`, `completion_tokens_details.reasoning_tokens`, `total_tokens`, `cost`, `cost_details.upstream_inference_cost`, `is_byok`.
- `src/utils/compute-token-usage.ts` lines 24-36: `noCache: promptTokens - cacheReadTokens`, `cacheWrite: cacheWriteTokens`, `text: completionTokens - reasoningTokens`.
- Request side: `usage: this.settings.usage` (line 224) and `stream_options` only when `this.config.compatibility === 'strict'` (lines 678-688). `src/provider.ts` line 288: "we default to compatible, because strict breaks providers like Groq:".
- `e2e/issues/issue-419-usage-fallback.test.ts` lines 5-7: "Standard usage object contains undefined values while providerMetadata.openrouter.usage has correct data. Model: z-ai/glm-5:free".
- Source: F-s1, F-s5, F-s6, F-s8. Basis: OR-documented.

**F23. Three error shapes arrive with HTTP 200.**

- Quote (`src/chat/index.ts` line 345): "Check if response is an error (HTTP 200 with error payload)". The response schema is a union of the success shape and `OpenRouterErrorResponseSchema.extend({ user_id: z.string().optional() })` (`schemas.ts` lines 164-167).
- Quote (lines 375-376): "A provider failure after generation started arrives as HTTP 200 with the error on the choice, next to whatever output was produced." Changeset `choice-error-throws.md`: "(`choices[0].error`, sent with HTTP 200 and `finish_reason: \"error\"`)" and "Before, the partial output was returned as a successful result."
- Stream (lines 794-799): `if ('error' in value) { finishReason = createFinishReason('error'); controller.enqueue({ type: 'error', error: value.error }); return; }`.
- Error object shape (`src/schemas/error-response.ts` lines 6-23): `code` (string or number), `message`, `type`, `param`, plus passthrough keys.
- Source: F-s1, F-s5, F-s7. Basis: OR-documented.

**F24. The useful error text is nested in `error.metadata.raw`.**

- Quote (`src/schemas/error-response.ts` lines 28-30): "The top-level `error.message` is often generic (e.g. \"Provider returned error\"), while `error.metadata.raw` contains the actual upstream provider error details."
- Lines 61-62: "The raw field can be a string, a JSON string, or a nested object." Line 43 reads `metadata.provider_name`.
- Source: F-s5. Basis: OR-documented.

**F25. A dropped connection mid-stream needs its own handling.**

- `e2e/issues/issue-412-mid-stream-termination.test.ts` lines 5-9: "When a TLS socket closes mid-stream during an SSE streaming response, the consumer receives a raw `TypeError: terminated` with no HTTP status code, no error code, and no response body."
- Source: F-s8, with `src/utils/with-stream-error-handling.ts` in F-s5. Basis: OR-documented.

**F26. Tool call deltas are repaired in six ways.**

- Quote (lines 471-472): "Track seen tool call IDs to ensure uniqueness. Some providers return empty, null, or duplicate IDs for parallel tool calls." Code: `if (!toolCallId || seenToolCallIds.has(toolCallId)) { toolCallId = generateId(); }`.
- Quote (line 1118): "Tool call start. OpenRouter returns all information except the arguments in the first chunk."
- Quote (lines 1111-1113): "No index (allowed by the schema): a delta with a new id or, without an id, a new function name starts a new tool call; anything else continues the most recent one."
- Quote (line 1212): "Some providers resend the full arguments in a later chunk."
- Missing arguments: `input: toolCall.function.arguments ?? '{}'` (line 488). Changelog 2.0.4: "upstream providers may omit the arguments field for tools with no parameters."
- Changelog 2.9.1: "Send `content: null` instead of `content: \"\"` for assistant messages that contain only tool calls. Fixes AWS Bedrock Nova rejecting requests with \"The text field in the ContentBlock object is blank.\""
- Changelog 2.9.0: a trailing whitespace delta made a second `tool-call` event, "Observed in production with `moonshotai/kimi-k2.6` via OpenRouter, where the user-visible effect was every outbound message being delivered twice."
- Source: F-s1, F-s7. Basis: OR-documented.
- Implication (OR-inferred from this code): OpenRouter forwards upstream tool-call delta quirks largely as they are. It does not normalize ids, indexes or argument framing across providers.

**F27. Structured output, routing and request options.**

- Changelog 2.9.0: "Models like `moonshotai/kimi-k2.6` (routed through Parasail/Venice/Io Net) returned HTTP 404 \"No endpoints available matching your guardrail restrictions and data policy\" because the strict flag eliminated every eligible endpoint." The fix is `structuredOutputs: { strict: false }`. Default stays `strict: this.settings.structuredOutputs?.strict ?? true` (line 203).
- `openrouter-chat-settings.ts` lines 83-85, on the response-healing plugin: "**Important:** This plugin only works with non-streaming requests (e.g., `generateObject`). It has no effect when used with streaming methods like `streamObject` or `streamText`."
- Provider routing type (lines 184-231): `order`, `allow_fallbacks`, `require_parameters`, `data_collection`, `only`, `ignore`, `quantizations`, `sort`, `max_price`, `zdr`. Changeset `merge-chat-provider-routing.md` (fixes issue 587): four sources of `provider` are shallow merged, and `providerOptions.openrouter.provider = null` "omits `provider` from the request entirely".
- Other request fields the client sets: `models`, `plugins`, `web_search_options`, `debug` (`echo_upstream_body`, "Only works with streaming requests."), `include_reasoning` (marked `@deprecated use `reasoning` instead`), and server tools mapped from `openrouter.web_search` to `openrouter:web_search` (lines 1456-1462).
- Source: F-s1, F-s6, F-s7. Basis: OR-documented.

**F54. File parse annotations are a second round-trip field.**

- Changelog 1.4.0: "Annotations from file parsing are now exposed via `providerMetadata.openrouter.annotations`" and "Pass annotations back via `providerOptions.openrouter.annotations` to skip re-parsing costs".
- Shape (`src/schemas/provider-metadata.ts` lines 10-30): `{ type: 'file', file: { hash, name, content?: [{ type, text? }] } }`. They are sent back as `annotations: messageAnnotations` on the assistant message (F9).
- Source: F-s3, F-s7. Basis: OR-documented.

### C. opencode (native `@opencode/ai` provider at HEAD)

**F16. opencode's own OpenRouter provider: request options, budget clamp, and a second route.**

- `packages/ai/src/providers/openrouter.ts` lines 153-165, the assistant message hook: `reasoning_content: undefined, reasoning_text: undefined, reasoning: reasoning && reasoningDetails && reasoningDetails.length > 0 ? reasoning : undefined, reasoning_details: reasoningDetails,`. Plain reasoning text goes out only together with details, the same rule as F7.
- Quote (lines 172-173): "OpenRouter forwards `reasoning.max_tokens` as the upstream thinking budget. Upstreams such as Anthropic and Alibaba reject one that is not below the output limit; 1,024 is Anthropic's minimum budget." The test "fits the reasoning budget to half the output limit" expects `{ max_tokens: 131_071 }` with `maxTokens` 32,000 to become `{ max_tokens: 16_000 }`, and `{ max_tokens: 65_536 }` with `maxTokens` 1,500 to become `{ max_tokens: 1_024 }`.
- Usage accounting is on by default: `openrouter.usage === undefined || known.usage === true ? { include: true }` (lines 185-187).
- Pass-through options (lines 119-131): `models`, `provider`, `plugins`, `debug`, `user`, `usage`, `web_search_options`, `reasoning`, and any unknown key is forwarded as written.
- Second route (lines 251-259): `endpoint: Endpoint.path("/responses", { baseURL })` with `options: { providerOptions: { store: false, include: ["reasoning.encrypted_content"] } }`.
- Reasoning variants (`packages/core/src/variant.ts` lines 438-447): effort becomes `{ reasoning: { effort } }`, toggle becomes `{ reasoning: { enabled: false } }` or `{ reasoning: { enabled: true } }`, budget becomes `{ reasoning: { max_tokens: tokens } }`.
- Source: F-s11, F-s15, F-s16. Basis: Filled (opencode). The max_tokens statement is OR-inferred from opencode's comment and test.

**F57a. opencode's storage shape for OpenRouter reasoning.**

- `packages/ai/src/protocols/openai-chat.ts` lines 114-116: "`reasoning_details` carries two dialects. OpenRouter's `reasoning.*` entries must be replayed unmodified (`index` included), so they keep every field they arrived with."
- Stored on each reasoning part (lines 1106-1117): `{ [providerMetadataKey]: { reasoningField?, reasoningDetails?, reasoningOpaque? } }`, where the key is `"openrouter"`. The test at `packages/ai/test/provider/openrouter.test.ts` line 540 shows `providerMetadata: { openrouter: { reasoningField: "reasoning", reasoningDetails: details } }`.
- Schema (lines 121-147): `id`, `format` (any string), `index`, `signature` are typed, and every other field is kept through `Schema.Record(Schema.String, Schema.Unknown)`. Unknown `type` values are dropped: the test "drops unrecognized details and preserves duplicate continuation details" sends `{ type: "reasoning.future", ... }` and expects it gone, and expects two identical `reasoning.encrypted` entries to both remain.
- Merge rule (lines 1054-1101): consecutive text or summary deltas join, encrypted entries never join, and a join is refused when `id`, `index` or `format` differ or when both sides have different signatures. Quote (lines 1083-1084): "The first non-empty signature and format win; a later delta may carry the signature for text that streamed earlier."
- Replay guard (lines 580-582): "A model that declares a different reasoning field reads reasoning there, so structured details and opaque values from earlier turns (including another model served under the same provider metadata key) must not be replayed."
- Source: F-s12, F-s16. Basis: Filled (opencode).
- Two differences from OpenRouter's own SDK: opencode keeps `format` as an open string (so the six formats in F14 survive), and it does not dedupe. Both clients drop `reasoning.server_tool_call`.

**F19. opencode chooses cache markers for OpenRouter from the model id prefix.**

- Quote (`packages/ai/src/cache-policy.ts` lines 81-83): "OpenRouter and Vercel AI Gateway upstreams other than Anthropic and Alibaba Qwen cache without breakpoints. Gemini uses only the last breakpoint, so a conversation-tail breakpoint writes a new cache every step and costs more than none. Qwen ignores breakpoints on tool definitions and caches tools with the system prompt."
- Code (lines 85-91): `const id = modelID.replace(/^~/, "")`, then `anthropic/` gets the full policy (last tool, first and last system parts, conversation tail), `qwen/` or `alibaba/qwen` gets system and tail only, everything else gets no markers. Line 86: "`~anthropic/claude-sonnet-latest` style IDs are OpenRouter aliases for the latest model in a family."
- `packages/ai/README.md` line 1678: "OpenRouter | emits up to 4 `cache_control` markers". Line 1652: "`ttl` on every `cache_control` marker". The marker written is `{ type: "ephemeral" as const, ttl: cache.ttl }` (`protocols/utils/cache.ts`).
- The chat model also declares `capabilities: { promptCacheKey: true }` (line 301) and sends `prompt_cache_key`.
- Source: F-s11, F-s13. Basis: OR-inferred from opencode's comments and policy code. The statements about Gemini and Qwen are opencode's observations, not OpenRouter documentation.

**F29. opencode treats `usage.cost` as the authoritative charge.**

- Quote (`packages/ai/src/schema/events.ts` lines 75-77): "USD the endpoint says it charged for this request, as OpenRouter, LiteLLM and OpenCode Zen report it in `usage.cost`. `0` is authoritative: the request was free or a subscription covered it. Absent, callers price the tokens themselves."
- `mapUsage` (`openai-chat.ts` lines 977-983) reads `prompt_tokens_details.cached_tokens` and `prompt_tokens_details.cache_write_tokens`, and keeps the raw usage object under `providerMetadata.openrouter`.
- Source: F-s12, F-s14. Basis: Filled (opencode).

**F30. opencode's error and finish handling for this wire.**

- `step` (lines 1121-1129): a chunk with `event.error` fails the stream, and a numeric `error.code` is used as the HTTP status for classification. The test "fails on a mid-stream provider error" feeds `error: { code: 502, message: "Provider disconnected" }` and expects `ProviderInternal`.
- `mapFinishReason` (lines 931-943): `"error"` raises "Provider reported an error (finish_reason: error)" and `"network_error"` raises "Provider reported a network error (finish_reason: network_error)".
- The raw reason keeps the upstream value (line 1143): `raw: choice?.native_finish_reason ?? rawFinishReason`. The test "preserves the upstream provider finish reason" expects `{ normalized: "stop", raw: "end_turn" }`.
- Quote (`packages/ai/src/provider-error.ts` lines 122-123): "OpenRouter tags provider failures with a typed `error_type`; its Responses skin also emits `image_content_policy_violation` as the native code."
- Source: F-s12, F-s14, F-s16. Basis: OR-inferred from opencode's code and tests.

**F55. Reasoning text can contain a `[REDACTED]` placeholder.**

- Quote (`packages/tui/src/routes/session/message-parts.tsx` lines 147-148): "OpenRouter encrypts some reasoning blocks; drop the placeholder." with `return part.text.replace("[REDACTED]", "").trim()`.
- OpenRouter's own SDK emitted the same string until 2.4.1 (F10), so the string may come from the old SDK and not from the wire.
- Source: F-s15. Basis: OR-inferred. Confidence low on where the placeholder originates.

**F66. opencode disables two model aliases and renames headers for OpenRouter.**

- Quote (`packages/core/src/plugin/provider/openrouter.ts` lines 22-23): "These are OpenRouter-specific OpenAI chat aliases that do not work on the generic path." The ids are `gpt-5-chat-latest` and `openai/gpt-5-chat`.
- Source: F-s15. Basis: Filled (opencode).

### D. Kilo Code (fork of opencode v1.18.26, still on `@openrouter/ai-sdk-provider`)

Kilo's provider plugin loads OpenRouter's SDK (`evt.sdk = mod.createOpenRouter(evt.options)`, `packages/core/src/plugin/provider/openrouter.ts` lines 32-34). The tree also holds an early native OpenRouter route in `packages/llm/src/providers/openrouter.ts`, which forwards only `usage`, `reasoning` and `prompt_cache_key`.

**F65. The older opencode transform layer, as kept by Kilo Code, has these OpenRouter branches.**

- Reasoning options are written in OpenRouter's shape (`packages/opencode/src/provider/transform.ts` lines 1361-1362 and 1462-1463): `return { reasoning: { effort } }` and `return { reasoning: { max_tokens: budget } }`.
- Defaults (lines 1535-1545): `result["usage"] = { include: true, }` and `if (input.model.api.id.includes("gemini-3")) { result["reasoning"] = { effort: "high" } }`.
- Small-model calls (lines 1704-1707): for provider `openrouter` with a model id containing `google`, `return { reasoning: { enabled: false } }`.
- Quote (line 889): "OpenRouter maps xhigh to GLM-5.2's native max effort."
- Cache markers (lines 416-422): `openrouter: { cacheControl: { type: "ephemeral" }, }` is placed on the first two system messages and the last two other messages, when the model id contains `anthropic` or `claude` (lines 555-571).
- The generic interleaved-reasoning rewrite is skipped for OpenRouter (lines 331-335): `model.api.npm !== "@openrouter/ai-sdk-provider"`.
- Tool call ids are rewritten by model id, not by provider, so they apply through OpenRouter too (lines 234-235 and 263-272): for ids containing `claude`, `id.replace(/[^a-zA-Z0-9_-]/g, "_")`. For Mistral family ids, non-alphanumerics are removed and the id is cut or padded to 9 characters.
- Headers (`packages/core/src/plugin/provider/openrouter.ts` lines 16-17): `"HTTP-Referer"` and `"X-Title"`.
- Source: F-s17, F-s20. Basis: Filled (Kilo Code, opencode lineage).

**F63. Kilo Code repairs stale reasoning details after each step.**

- Quote (`packages/opencode/src/kilocode/session/reasoning-replay.ts` lines 7-13): "OpenRouter's AI SDK provider attaches its live, still-accumulating `openrouter.reasoning_details` array to early tool-call and reasoning-end events. Later SSE chunks can append details such as `reasoning.encrypted` (Gemini thought signatures) after those parts were already persisted, so the stored replay state is a stale prefix. The finish-step metadata carries the completed array; reconcile it into the parts of that step before the next request is built from durable history."
- When a step produced neither a reasoning part nor a tool part, Kilo writes a reasoning part with empty text to hold the array (lines 53-66): `metadata: { openrouter: { reasoning_details: structuredClone(input.details) } }`.
- Source: F-s18. Basis: Filled (Kilo Code).
- Implication (OR-inferred from this code and from F60): an encrypted detail can arrive after the tool call delta it belongs to. A client that persists parts as they stream stores an incomplete array.

**F22. Kilo Code's cost rule for BYOK.**

- Quote (`packages/opencode/src/kilocode/session/index.ts` lines 200-204): "For OpenRouter BYOK routing, `cost` is what OpenRouter charged the account ($0, or only its routing fee) and `upstreamInferenceCost` is billed to the user's own key. True spend is the sum. A non-BYOK response always bills the account at least the upstream cost, so summing only when upstream exceeds the billed amount never changes non-BYOK sessions."
- Code (line 244): `if (upstream !== undefined && upstream > (regular ?? -Infinity)) return upstream + (regular ?? 0)`.
- Source: F-s19. Basis: OR-inferred from Kilo Code's comment and code.

**F56. Image output needs a request flag.**

- Quote (`packages/opencode/src/kilocode/provider/image-output.ts` lines 4-6): "OpenRouter-compatible chat endpoints (OpenRouter and the Kilo Gateway) only return generated images when the request asks for the image output modality. Without it, image models such as Gemini \"Nano Banana\" answer with text only." The function returns `["image", "text"]`, and returns nothing for ids that start with `openrouter/`.
- Source: F-s20. Basis: OR-inferred from Kilo Code's comment.

### E. Cline (SDK rewrite at HEAD, v4.1.27)

**F43. Cline now reaches OpenRouter through the generic OpenAI-compatible provider, and signed reasoning does not survive that path.**

- `sdk/packages/llms/src/providers/vendors/openai-compatible.ts` lines 243-251: `const provider = createOpenAICompatible({ name: context.provider.id, apiKey, ... includeUsage: true, transformRequestBody: withMaxCompletionTokensForReasoningModels, } as never);`. OpenRouter's own `createOpenRouter` is built only when `context.provider.metadata?.imageTransport === "openrouter" && modelProducesImages(context.model)` (lines 252-268).
- History replay (`sdk/packages/llms/src/providers/ai-sdk.ts` lines 766-790): a stored reasoning part is sent as `{ type: "reasoning", text, providerOptions: { anthropic: { signature, redactedData } } }`. The generic provider turns reasoning parts into one string, `reasoning_content` (F42), and does not read the `anthropic` options.
- A ripgrep over `sdk/packages` (generated catalog excluded) finds `reasoning_details` only at `sdk/packages/llms/src/catalog/catalog-live.ts` line 333, as a catalog value for `interleavedReasoningField`. No code on this path writes a `reasoning_details` field to the wire.
- Tool calls (lines 816-834): a stored `thoughtSignature`, `signature` or `thought_signature` is sent as `providerOptions: { google: { thoughtSignature } }`, which the generic provider writes as `extra_content.google.thought_signature` (F42).
- Legacy extension storage, still in the tree (`apps/vscode/src/shared/messages/content.ts`): `export const REASONING_DETAILS_PROVIDERS = ["cline", "openrouter"]` (line 21). The array was stored on the text block and on the tool_use block: "reasoning_details only exists for providers listed in REASONING_DETAILS_PROVIDERS" (lines 29 and 45), with element type `{ type: "reasoning.text" | string, text: string, signature: string, format: "anthropic-claude-v1" | string, index: number }` (lines 8-14). Line 122: "Removes thinking block that has no signature (invalid thinking block that's incompatible with Anthropic API)".
- Source: F-s22, F-s24, F-s32. Basis: Filled (Cline).
- Confidence: medium. I read the code path, and did not run it. The conclusion that Claude signatures and OpenAI encrypted reasoning are not replayed through OpenRouter on this path rests on the generic provider having zero matches for `reasoning_details` (F42).

**F64. Cline's OpenRouter rules.**

- Reasoning (`routing/provider-option-rules.ts` line 248): "OpenRouter expects reasoning controls under its first-class reasoning object." `routing/reasoning-codecs.ts` lines 8-9: "OpenRouter separates total output limits (`max_tokens` / `max_output_tokens`) from `reasoning.max_tokens`, which caps only the reasoning-token portion." Off is `{ effort: "none" }` (line 49). On without a budget is `{ enabled: true, max_tokens: ... }` with `OPENROUTER_REASONING_BUDGET_FRACTION = 0.6` of the output limit (line 13).
- Routing (lines 152-171): "OpenRouter and Cline enable provider failover for Anthropic models." with `bucketOptions: { provider: { allow_fallbacks: true } }`.
- Vendor-family thinking rules are switched off for OpenRouter: the Kimi K2.6 and DeepSeek `thinking.type` rules both require `input.request.providerId !== "openrouter"` (lines 420 and 437).
- Sticky session (`builtins.ts` lines 89-95): `stickySession: { transport: "json-body", field: "session_id", metadataKey: "sessionId" }`.
- Cost (`ai-sdk.ts` lines 1241-1271): reads `cost_details.upstream_inference_cost`, `market_cost`, `cost`, `is_byok`, and adds upstream cost to `cost` when `is_byok` is true.
- Errors (`error-classification.ts` lines 324-328): "The sole message fallback. OpenRouter forwards an upstream failure mid-stream as a bare \"Provider returned error\" string with no HTTP status and no typed error to inspect, so there is nothing else to key on." The pattern is `/provider returned error/i`. `sdk/packages/agents/src/agent-runtime.ts` retries such a turn up to 3 times (`PROVIDER_ERROR_MAX_RETRIES = 3`).
- Empty turns (`middleware/retry-empty-response.ts` lines 5-10): "providers intermittently return a response that finishes normally but carries no content at all. This was first observed on local backends (Ollama especially), but production telemetry shows hosted backends (openrouter, cline, generic OpenAI-compatible endpoints) do the same."
- Source: F-s21, F-s22, F-s23. Basis: OR-inferred from Cline's code and comments.

### F. Roo Code (archived 2026-05-15)

**F68. Roo Code is frozen.** The last commit is `b867ec9`, 2026-05-15, "Remove roocode.com web app (#12375)". Third-party pages report the repository as archived on that date with final release v3.54.0 (F-s49). Its code is evidence of what worked in May 2026, not of current behavior. Basis: Filled (Roo Code).

**F31. Roo Code injects a fake encrypted reasoning entry for Gemini tool calls.**

- Quote (`src/api/providers/openrouter.ts` lines 248-255): "Inject fake reasoning.encrypted block for tool calls without existing encrypted reasoning. This is required when switching from other models to Gemini to satisfy API validation. Per OpenRouter documentation (conversation with Toven, Nov 2025): - Create ONE reasoning_details entry per assistant message with tool calls - Set `id` to the FIRST tool call's ID from the tool_calls array - Set `data` to \"skip_thought_signature_validator\" to bypass signature validation - Set `index` to 0 See: https://github.com/cline/cline/issues/8214".
- Code (lines 273-279): `const fakeEncrypted = { type: "reasoning.encrypted", data: "skip_thought_signature_validator", id: toolCalls[0].id, format: "google-gemini-v1", index: 0, }`.
- Source: F-s25. Basis: Filled (Roo Code). OR-inferred for the mapping: the source names a conversation with an OpenRouter person, and the string `skip_thought_signature_validator` has zero matches in the local mirror of OpenRouter's 559 documentation pages.
- Implication: OpenRouter maps a Gemini `reasoning.encrypted` entry's `id` to a tool call and its `data` to that call's `thought_signature`. Confidence medium.

**F32. Roo Code drops Gemini tool calls that have no matching reasoning detail, and consolidates details by index.**

- Quote (`src/api/transform/openai-format.ts` lines 152-157): "Gemini models require thought signatures for tool calls. When switching providers mid-conversation, historical tool calls may not include Gemini reasoning details, which can poison the next request. This function: 1. Filters reasoning_details to only include entries matching tool call IDs 2. Drops tool_calls that lack any matching reasoning_details 3. Removes corresponding tool result messages for dropped tool calls".
- Quote (lines 50-52): "Drop corrupted encrypted reasoning blocks that would otherwise trigger: \"Invalid input: expected string, received undefined\" for reasoning_details.*.data".
- The stream accumulator keys entries by type and index (`openrouter.ts` line 391): ``const key = `${detail.type}-${index}` ``, and concatenates `text`, `summary` and `data`.
- Source: F-s25, F-s26. Basis: Filled (Roo Code).
- Note: keying by `index` contradicts F1 and F5, where OpenRouter's SDK says `index` repeats. Roo's approach would merge the two signed blocks of F3 only if they shared an index, and would merge all OpenAI summaries at index 0, which is the wanted result there.

**F33. Roo Code strips `id` from OpenAI reasoning entries.**

- Quote (lines 288-291): "Strip `id` from openai-responses-v1 blocks because OpenAI's Responses API requires `store: true` to persist reasoning blocks. Since we manage conversation state client-side, we don't use `store: true`, and sending back the `id` field causes a 404 error."
- Source: F-s26. Basis: Filled (Roo Code). OR-inferred: OpenRouter passed the reasoning item id through to OpenAI's Responses API as written.

**F34. Roo Code's caching for OpenRouter is an allowlist of model ids.**

- `packages/types/src/providers/openrouter.ts` line 21: `OPEN_ROUTER_PROMPT_CACHING_MODELS`, 34 ids, 25 `anthropic/...` and 9 `google/gemini-...`.
- Anthropic strategy (`src/api/transform/caching/anthropic.ts`): system prompt as a text block with `cache_control: { type: "ephemeral" }`, plus "Add `cache_control: ephemeral` to the last two user messages." Gemini strategy (`caching/gemini.ts`): "*Always* cache the system prompt." plus a marker on every 10th user message.
- Source: F-s26, F-s27. Basis: Filled (Roo Code).

**F35. Roo Code's model metadata fetch and reasoning parameters.**

- Fetches `GET {baseURL}/models` and `GET {baseURL}/models/${modelId}/endpoints` (`src/api/providers/fetchers/openrouter.ts` lines 102 and 150). Reads `context_length`, `top_provider.max_completion_tokens`, `supported_parameters`, `architecture.input_modalities`, `pricing.prompt`, `pricing.completion`, `pricing.input_cache_write`, `pricing.input_cache_read`. Endpoints are keyed by `endpoint.tag ?? endpoint.provider_name`.
- Line 208: `const supportsPromptCache = typeof cacheReadsPrice !== "undefined" // some models support caching but don't charge a cacheWritesPrice, e.g. GPT-5`. Line 220: `supportsReasoningEffort: supportedParameters ? supportedParameters.includes("reasoning") : undefined,`.
- Hand-kept sets beside the fetch: `OPEN_ROUTER_REASONING_BUDGET_MODELS`, `OPEN_ROUTER_REQUIRED_REASONING_BUDGET_MODELS`, and per-id `maxTokens` patches for six Claude ids and two `openrouter/horizon-*` ids (lines 236-281).
- Quote (`openrouter.ts` lines 212-215): "OpenRouter sends reasoning tokens by default for Gemini 2.5 Pro models even if you don't request them. This is not the default for other providers (including Gemini), so we need to explicitly disable them unless the user has explicitly configured reasoning." The fix is `reasoning = { exclude: true }`.
- Provider pin (lines 311-319): `provider: { order: [x], only: [x], allow_fallbacks: false }`.
- Anthropic beta header through OpenRouter (line 327): `{ headers: { "x-anthropic-beta": "fine-grained-tool-streaming-2025-05-14" } }` for ids starting with `anthropic/`.
- Cost (line 494): `totalCost: (lastUsage.cost_details?.upstream_inference_cost || 0) + (lastUsage.cost || 0),`.
- Stream error (line 363): "OpenRouter returns an error object instead of the OpenAI SDK throwing an error." The thrown text is ``OpenRouter API Error ${error?.code}: ${rawErrorMessage}``, with the message taken from `metadata.raw` first.
- Source: F-s25, F-s27. Basis: Filled (Roo Code). The Gemini 2.5 Pro default and the header pass-through are OR-inferred from Roo's comments.

**F57b. Roo Code's storage shape.** The handler returns the consolidated array through `getReasoningDetails()`, and the task stores it as a property on the saved assistant message (`src/core/task/Task.ts` lines 887-890): "Store reasoning_details array if present (for models like Gemini 3)" with `messageWithTs.reasoning_details = reasoningDetails`. On replay the message goes out as `{ role: "assistant", content, reasoning_details }` (lines 4404-4408), and the property is placed before `tool_calls` because "Property order matters when sending messages back to some APIs." (`openai-format.ts` lines 478-480). Source: F-s26, F-s27. Basis: Filled (Roo Code).

### G. pi

**F40. pi sends every `anthropic/*` model on OpenRouter to the Anthropic Messages route, not to chat completions.**

- Code (`packages/ai/catalog/sources/openrouter.ts` lines 71-78): "Includes the `~anthropic/claude-*-latest` aliases, which OpenRouter's Messages endpoint accepts." then `const useAnthropicMessages = /^~?anthropic\//.test(model.id) && !model.id.endsWith(":batch");` and `api: useAnthropicMessages ? "anthropic-messages" : "openai-completions", baseUrl: useAnthropicMessages ? "https://openrouter.ai/api" : "https://openrouter.ai/api/v1",`.
- Provider type (`packages/ai/src/providers/openrouter.ts` line 62): `Provider<"anthropic-messages" | "openai-completions">`.
- Changelog, release 1.5.0 dated 2026-11-02 (`packages/ai/CHANGELOG.md` line 164): "Fixed OpenRouter's `~anthropic/claude-*-latest` aliases losing Claude's earlier thinking after each tool call. They used the Chat Completions API, where pi dropped the thinking signature, and OpenRouter then ignored the unsigned thinking. They now use the Anthropic Messages API with adaptive thinking, like the versioned Claude models".
- Quote (`packages/ai/catalog/providers/openrouter.ts` lines 8-10): "OpenRouter rejects `configuration_update` system messages on Opus 5 (\"Mid-conversation reasoning effort (configuration_update) is not supported on anthropic/claude-opus-5-20260723\") while accepting them on Fable 5.1."
- Source: F-s29, F-s31. Basis: Filled (pi).
- Deciding fact: a mature harness took the bypass route for one vendor even though the chat route has a documented replay field. The stated reason is a client bug on the chat path, not a missing feature in OpenRouter.

**F57c. pi's storage shape on the chat route: the whole array as a JSON string in the signature slot.**

- Quote (`packages/ai/src/api/openai-completions.ts` lines 339-345): "`reasoning_details` are replay metadata, not user-visible stream deltas. Keep them in memory during streaming and serialize once when the block is finalized." with `block.thinkingSignature = JSON.stringify(streamedReasoningDetails);`.
- Quote (lines 619-621): "Keep provider replay data in the existing signature slot. OpenRouter streams reasoning_details as deltas: consecutive text/summary deltas are merged into logical entries, while encrypted entries remain opaque and discrete."
- The same slot holds a plain field name for other hosts: the thinking block's `thinkingSignature` is `"reasoning_content"`, `"reasoning"` or `"reasoning_text"` when the text came from that field (lines 584-602), and on replay the text is written back under that name (lines 1337-1343).
- Replay (lines 1307-1314 and 1378-1380): the first thinking block whose signature parses as a non-empty array of valid details wins, and goes out as `assistantMsg.reasoning_details`. A legacy shape is still read: one `reasoning.encrypted` detail stored as JSON in `toolCall.thoughtSignature`.
- Validation (lines 141-158) accepts only the three types of F13. `reasoning.text` requires `typeof detail.text === "string"`, which is the check that dropped signature-only deltas (F46).
- Source: F-s28. Basis: Filled (pi).

**F36. pi's caching and session affinity for OpenRouter.**

- `detectCompat` (line 1644): `const cacheControlFormat = provider === "openrouter" && model.id.startsWith("anthropic/") ? "anthropic" : undefined;`. Markers go on the first system or developer message, the last tool, and the last user, assistant or tool message (lines 1085-1087). TTL: `const ttl = cacheRetention === "long" && compat.supportsLongCacheRetention ? "1h" : undefined;` (line 1076).
- Session header (lines 761-763): `if (compat.sessionAffinityFormat === "openrouter") { headers["x-session-id"] = sessionId; }`. Changelog line 473: "Fixed OpenRouter requests to send `x-session-id` from `sessionId` for Chat Completions and Anthropic Messages models when prompt caching is enabled".
- Source: F-s28, F-s31. Basis: Filled (pi).

**F37. pi's usage and cost reading.**

- Quote (lines 1533-1543): "Follow documented OpenAI/OpenRouter semantics: cached_tokens is cache-read tokens (hits)." and "OpenAI does not document or emit cache_write_tokens, but OpenRouter-compatible providers can include it as a separate write count." and "Do not subtract writes from cached_tokens, otherwise spec-compliant providers are under-reported." pi computes `input = promptTokens - cacheReadTokens - cacheWriteTokens`.
- `packages/ai/src/utils/openrouter-cost.ts` lines 29-54: when `is_byok === false` the total is `reported.cost`. When `is_byok === true` the total is `reported.cost + upstreamCost`, where upstream cost is `cost_details.upstream_inference_cost` or, when that is null, `upstream_inference_prompt_cost + upstream_inference_completions_cost`. The type also lists `server_tool_cost`: "Metered server-tool cost already included in OpenRouter's charged `cost`."
- Quote (`packages/ai/src/api/anthropic-messages.ts` lines 1131-1132): "OpenRouter adds billing fields to streamed message_delta usage at runtime, although its OpenAPI schema and the Anthropic SDK do not type those fields."
- Source: F-s28, F-s30. Basis: OR-inferred from pi's code and comments.

**F38. pi's reasoning replay rules for DeepSeek and Kimi through OpenRouter.**

- Changelog line 1430: "Fixed OpenAI-compatible DeepSeek V4 model replay to include empty `reasoning_content` on assistant messages when needed, preventing OpenRouter DeepSeek V4 sessions from failing after responses without reasoning deltas". Code (lines 1381-1387): `assistantMsg.reasoning_content = "";`.
- `catalog/providers/openrouter.ts` lines 26-29: ids starting with `moonshotai/kimi-k2.6` get `requiresReasoningContentOnAssistantMessages: true`.
- Source: F-s28, F-s29, F-s31. Basis: Filled (pi).
- Compare with F12: OpenRouter's SDK solves the same DeepSeek case by sending back `reasoning_details: []`. Two clients, two different fields for the same upstream rule.

**F39. pi's model catalog and reasoning control for OpenRouter.**

- Reasoning is sent as `reasoning: { effort }`, and off is `{ effort: model.thinkingLevelMap?.off ?? "none" }` (lines 929-938). Changelog line 1672: "Fixed OpenRouter reasoning requests to use the provider's nested `reasoning.effort` payload instead of OpenAI's `reasoning_effort`".
- Changelog line 537: "Fixed OpenRouter reasoning controls by deriving `off` support and available effort levels from OpenRouter's model metadata, preventing reasoning-mandatory models from receiving `effort: \"none\"`". The catalog reads `reasoning.supported_efforts` and `reasoning.mandatory` (`catalog/sources/openrouter.ts` lines 14-25).
- Quote (`catalog/providers/openrouter.ts` lines 89-92): "Mercury 2 in instant mode (reasoning_effort: \"none\") disables tool calling. Mark \"off\" unsupported so the openai-completions provider omits the reasoning param instead of defaulting to {reasoning:{effort:\"none\"}}."
- Only models with tools are listed: `if (!model.supported_parameters?.includes("tools")) continue;` (line 63). Limits come from `model.top_provider?.context_length` and `model.top_provider?.max_completion_tokens`.
- Changelog line 249: "Fixed OpenRouter router models such as `openrouter/auto` showing a cost of -$1,000,000 per 1M tokens; OpenRouter's negative placeholder prices are now 0".
- `personalizeOpenRouterCatalog()` calls `GET /models/user` to narrow the catalog to what a key may use (`src/providers/openrouter.ts` lines 24-35).
- Routing: `(params as any).provider = model.compat.openRouterRouting;` (lines 979-981).
- Source: F-s28, F-s29, F-s31. Basis: Filled (pi).

**F41. pi's error, retry and overflow rules that name OpenRouter.**

- `mapStopReason` (lines 1563-1586): any unknown finish reason becomes an error with the text ``Provider finish_reason: ${reason}``. `packages/ai/src/utils/retry.ts` lines 124-126: "An OpenAI-compatible stream that ended with `finish_reason: \"error\"`, which OpenRouter sends when its provider fails mid-stream".
- Retry patterns (lines 61-64 and 127-130): `"provider.?returned.?(?:an.?)?error"`, `"returned an empty response"`, `"stopped before completing the response"`, with the comment "OpenRouter: \"Provider returned an empty response\", and its wrapper for a provider that gave up, e.g. \"Upstream error from Relace: The model stopped before completing the response.\""
- Error detail (lines 704-711): "Some providers via OpenRouter give additional information in this field." reading `error.error.metadata.raw`.
- Overflow (`packages/ai/src/utils/overflow.ts` lines 20-21): "OpenRouter: \"This endpoint's maximum context length is X tokens. However, you requested about Y tokens\"" and "OpenRouter/Poolside: \"Input length X exceeds the maximum allowed input length of Y tokens.\""
- Output clamp (`packages/ai/src/api/simple-options.ts` lines 16-18): "OpenRouter counts the input with its own tokenizer and rejects input + max_tokens above the window before routing. It counted prose up to 12% above the provider's count (#10497)." with `const CONTEXT_SAFETY_RATIO = 0.15;`.
- Source: F-s28, F-s30. Basis: OR-inferred from pi's code, comments and issue 10497 (F53).

### H. Vercel AI SDK generic provider and cross-project items

**F42. The generic OpenAI-compatible provider reads and writes only plain reasoning text.**

- `packages/openai-compatible` has zero matches for `reasoning_details` (ripgrep count over the package at the pinned commit).
- Read side (`src/chat/openai-compatible-chat-language-model.ts` lines 449-450): `const reasoning = choice.message.reasoning_content ?? choice.message.reasoning;`.
- Write side (`src/chat/convert-to-openai-compatible-chat-messages.ts` line 310): `...(reasoning.length > 0 ? { reasoning_content: reasoning } : {}),`.
- Gemini signature on tool calls (lines 279-300): "TODO: thoughtSignature should be abstracted once we add support for other providers", written as `extra_content: { google: { thought_signature: String(thoughtSignature) } }`.
- Usage (`convert-openai-compatible-chat-usage.ts`): `cacheWrite: undefined,`. `cost` is reachable only through `raw: usage`.
- Finish reason (`map-openai-compatible-finish-reason.ts`): the switch has no `error` case, and the default is `return 'other';`.
- An escape hatch exists: `getOpenAIMetadata` spreads `providerOptions.openaiCompatible` onto messages and parts (lines 23-27), which is how the older opencode and Cline place `cache_control` and a reasoning field on a message.
- OpenRouter's reasoning documentation says of the plain fields: "You can also use `reasoning_content` as an alias - it functions identically to `reasoning`."
- Source: F-s32, F-s10. Basis: Filled (Vercel AI SDK).
- Implication: a standard client still gets text and tool calls from OpenRouter, and sends plain reasoning back. It keeps no signature, no encrypted blob, no cache-write count, and reads an `error` finish as `other`.

**F28. Headers and session identifiers sent to OpenRouter, by project.**

- OpenRouter SDK (`src/provider.ts` lines 299-305): `'X-OpenRouter-Title'` from `appName`, `'HTTP-Referer'` from `appUrl`, `'X-Provider-API-Keys': JSON.stringify(options.api_keys)`, and a user agent ``ai-sdk/openrouter/${VERSION}``.
- opencode plugin: `"HTTP-Referer": "https://opencode.ai/", "X-Title": "opencode"`. Its legacy option mapper also writes `"X-OpenRouter-Title"`, `"HTTP-Referer"` and `"X-Provider-API-Keys"` (`packages/core/src/aisdk-native.ts` lines 330-342).
- Kilo Code: `"HTTP-Referer"` = `https://kilo.ai/`, `"X-Title"` = `Kilo Code`.
- Cline (`sdk/packages/llms/src/providers/request-headers.ts` lines 46-47): `"HTTP-Referer": "https://cline.bot", "X-Title": "Cline",`. Session id goes in the JSON body as `session_id` (F64).
- Roo Code (`src/api/providers/constants.ts`): `"HTTP-Referer": "https://github.com/RooVetGit/Roo-Cline", "X-Title": "Roo Code", "User-Agent": ...`, plus `x-anthropic-beta` for Claude (F35).
- pi (`packages/ai/src/utils/identity.ts` lines 111-117): `HTTP-Referer`, `"X-OpenRouter-Title"`, `"X-OpenRouter-Categories"`, plus `x-session-id` (F36).
- Source: F-s6, F-s15, F-s20, F-s22, F-s27, F-s30. Basis: OR-inferred from client code. Two title header names are in use: `X-Title` (four projects) and `X-OpenRouter-Title` (OpenRouter's SDK, pi, opencode's mapper).

**F57. Storage shapes for the opaque reasoning state, side by side.**

| Project | Where the array lives | Granularity | Known failure of that shape |
|---|---|---|---|
| OpenRouter SDK | `providerMetadata.openrouter.reasoning_details` on reasoning parts, on the first tool-call part, and on the response | Copies on several parts, one chosen on replay | Stale or duplicate copies (F52, F60, F63) |
| Kilo Code | Same as the SDK, then rewritten after each step from finish metadata. An empty reasoning part is added when no part exists | Per step | Needs a reconcile pass (F63) |
| opencode HEAD | `providerMetadata.openrouter.reasoningDetails` plus `reasoningField` on reasoning parts | All parts' arrays are concatenated on replay | Unknown types dropped (F57a) |
| pi | `thinkingSignature` string on the thinking block holds `JSON.stringify(details)` | One array per assistant message, first valid block wins | Validation dropped signature-only deltas (F46) |
| Roo Code | `reasoning_details` property on the stored assistant message | One array per message | Consolidation by index, entries with undefined `data` (F48) |
| Cline (legacy) | `reasoning_details` on the text block and on the tool_use block | Per block | Entries with undefined `data` (F48) |

- Source: F-s1, F-s12, F-s18, F-s24, F-s27, F-s28. Basis: Filled (each project named in the table).
- Fact for the decision: the two projects that keep one array per assistant message outside the part list (pi, Roo Code) have no duplicate or stale-copy bugs on record here. The projects that spread copies across parts have three.

### I. Issue reports (third-party accounts)

Each item below is an issue report or pull request read on its public page on 2026-11-22. Items F50 (second half), F52 and F62 were read as search-result excerpts of the issue page, not as full page fetches, and are marked so.

**F59. Anthropic signature lost while streaming.** `OpenRouterTeam/ai-sdk-provider` issue 423, created 2026-02-19, closed. The title begins "Anthropic thinking block `signature` lost during streaming" and names the result, "`Invalid signature in thinking block` on multi-turn conversations". Error text: "invalid_request_error: messages.1.content.0: Invalid `signature` in `thinking` block". The report gives the wire order: "Many deltas with text: `{ type: \"reasoning.text\", text: \"fragment...\" }`" then "A final delta with only the signature: `{ type: \"reasoning.text\", signature: \"eyJ...\", format: \"anthropic-claude-v1\" }`". Fixes followed in pull requests 427, 442, 445 and 458 over seven weeks, and the issue was reopened as 453 in between. Source: F-s33. Basis: OR-inferred from an issue report in OpenRouter's own repository.

**F44. Gemini "Corrupted thought signature".** Issue 418, created 2026-02-14, closed. Response body quoted in the report: `"message": "Provider returned error", "code": 400, "metadata": { "raw": "{\n  \"error\": {\n    \"code\": 400,\n    \"message\": \"Corrupted thought signature.\",\n    \"status\": \"INVALID_ARGUMENT\"\n  }\n}\n", "provider_name": "Google AI Studio", "is_byok": false }`. The reporter: "why random? sending the same prompt chain again works." Source: F-s34. Basis: OR-inferred from an issue report. It also shows the error envelope of F24 on a real response.

**F45. Gemini thought signatures are tied to the upstream provider, and OpenRouter staff say the fix belongs in OpenRouter's backend.** Issue 491, created 2026-04-30, closed 2026-06-17. Reporter's matrix: "google-vertex -> google-vertex: success", "google-ai-studio -> google-ai-studio: success", "google-vertex -> google-ai-studio: fails", "google-ai-studio -> google-vertex: fails". Reply signed "the OpenRouter SDK team" (user mattapperson, 2026-06-17): "The 400 reproduces with a raw curl request and **zero SDK involvement**." and "switching between `Google AI Studio` and `Google` (Vertex) mid-tool-call returns 400 (`Unable to submit request because Thought signature is not valid.` / `Corrupted thought signature.`). Gemini thought signatures are provider-scoped, so they can't be validated by a different upstream provider." and "A practical workaround in the meantime: pin the provider for a multi-turn tool-call conversation (e.g. `provider: { order: [...], allow_fallbacks: false }`) so all turns route to the same Google endpoint." The report also shows the entry shape: `{ "type": "reasoning.encrypted", "format": "google-gemini-v1", "data": "<large opaque blob>", "id": "tool_get-available-images_123", "index": 0 }`. Source: F-s36. Basis: OR-inferred from an issue report that quotes OpenRouter staff. Confidence high for the behavior as of June 2026. Whether OpenRouter's router now pins the upstream when `reasoning_details` are present is not known.

**F47. Two signed thinking blocks merged into one.** Issue 582, created 2026-10-21, closed the same day. "Claude Opus 5.5, Fable 5.1 and Sonnet 5.5 return the text they write between tool calls as progress-update thinking blocks, each with its own signature, right before the tool call it introduces." Delta sequence quoted: `{"type":"reasoning.text","text":"...","format":"anthropic-claude-v1","index":0}` ... `{"type":"reasoning.text","signature":"<signature A>","format":"anthropic-claude-v1","index":0}` then the same with `"index":1` and `<signature B>`. "None of these details carries an `id`." Replay test by the reporter: "two separate details, each with its own signature, were accepted (200); the merged detail was refused (400); the first detail alone was accepted." Error: "400 ... messages.N.content.0: Invalid signature in thinking block". Source: F-s37. Basis: OR-inferred from an issue report.

**F60. Late reasoning details are still lost over a real wire (open).** Pull request 608, opened 2026-10-23, state open on 2026-11-22. "Any `reasoning_details` that arrive after that point were added to `accumulatedReasoningDetails`, but no later persisted part carried them. Examples are a late `reasoning.encrypted` blob from OpenAI and a signature-only delta from Anthropic." and "In process it only looked like it worked. `reasoning-end` (and the first `tool-call`) held the same mutable array object, so later mutation showed through." and "The UI message stream serializes `reasoning-end` right away, so the late blob or signature was lost, and the next turn replayed `[summary]` only. For Anthropic it replayed `[]`, because the unsigned text gets stripped." and "Finish metadata is not persisted into messages." Source: F-s38. Basis: OR-inferred from a pull request in OpenRouter's repository. This is the same defect Kilo Code patches on its side (F63).

**F46. Unsigned reasoning is silently dropped by OpenRouter, with a measured cost.** `earendil-works/pi` issue 9534, created 2026-09-12, closed 2026-10-28. Title: "OpenRouter: signature-only reasoning chunks dropped for Claude". "All continuation requests succeeded". Measurement by user jkytomaki, 2026-10-25: "Replaying turn 1's reasoning to Opus 5.5 via OpenRouter → Anthropic, the second call's prompt_tokens were 590 with no reasoning, 590 with pi's unsigned `reasoning.text`, and 806 with the signature included. So OpenRouter silently drops unsigned reasoning, and the model (Claude Opus 5.5 in our test) never sees its earlier thinking." A/B table by user jacksonsstm2, 2026-10-14 (3 runs per arm, median): wall time 1403 s against 296 s, output tokens 121k against 12k, reasoning tokens 107k against 6.0k, cost $3.54 against $0.79. Source: F-s42. Basis: OR-inferred from an issue report with measurements. Confidence medium-high: two independent commenters, no OpenRouter statement.

**F48. Corrupted encrypted entry rejected by schema.** `cline/cline` issue 8214, created 2025-12-19, closed 2026-01-10. Error: `{"error":{"message":"Invalid input: expected string, received undefined","type":"invalid_request_error","param":"messages.2.reasoning_details.0.data","code":"invalid_request_error"}}`, seen for `google/gemini-3-pro-preview` and `xai/grok-code-fast-1` through the Cline provider ("from Vercel" in the message). Maintainer: "the issue was that corrupted reasoning details with undefined `data` fields were being sent to the API." Source: F-s41. Basis: Filled (Cline). The 400 came from Cline's own gateway ("from Vercel" in the message), not from OpenRouter. It shows that a `reasoning.encrypted` entry is validated as having a string `data`, which matches the spec (F14: `"required": ["type", "data"]`).

**F50. OpenAI reasoning ids: duplicates and unverifiable blobs.** (a) `ai-sdk-provider` issue 254, created 2025-11-26, closed: "Getting Duplicate item found with id rs_0ad20f1f8629dc53016924443203408193abb5b3d0b4301e26. Remove duplicate items from your input and try again." with model `gpt-5-codex`. Source: F-s35. (b) `anomalyco/opencode` issue 4608, created 2025-11-21, closed 2026-01-16, read as a search-result excerpt: "The encrypted content for item rs_0aa9cdb85cd23c86016920cad04e4c8196bf0bdb7dbce6a4df could not be verified." and the commenter's diagnosis that the blobs "no longer match the OpenAI project/key OpenRouter is using". Source: F-s45. Basis: OR-inferred from issue reports.

**F52. Duplicated or rewritten thinking blocks on tool parts.** `anomalyco/opencode` issue 14716, created 2026-02-22, closed 2026-05-22, read as a search-result excerpt. Error: "messages.1.content.6: `thinking` or `redacted_thinking` blocks in the latest assistant message cannot be modified. These blocks must remain as they were in the original response." Reporter: "tool Parts have duplicated reasoning attached to them in the session." The envelope in the log: `"responseBody": "{\"error\":{\"message\":\"Provider returned error\",\"code\":400,\"metadata\":{\"raw\":...`. A related pull request (19048, 2026-03-25, also read as an excerpt) lists three causes: "`trimEnd()` on reasoning text mutates the text after Anthropic signs it", duplicate pushes into the accumulated details, and "`providerMetadata` assignment (`=`) overwrites previously stored signature data". Source: F-s46, F-s47. Basis: OR-inferred from issue reports.

**F62. The SDK's strip warning reached users.** `anomalyco/opencode` issue 21263, created 2026-04-07, closed, read as a search-result excerpt: users of Opus 4.6 through OpenRouter saw "Some reasoning details entries were removed because they were missing signatures." in the terminal UI. Source: F-s47. Basis: OR-inferred from an issue report. It confirms the F7 strip fired in normal use, which means signatures were going missing in real sessions in April 2026.

**F51. OpenRouter forces fine-grained tool streaming on Anthropic tools.** `ai-sdk-provider` issue 574, created 2026-10-09, state open. Reporter, 2026-10-14: "OpenRouter support has confirmed the cause. On streamed requests with function tools, OpenRouter sets `eager_input_streaming: true` on every tool and doesn't read the value the client sends: **Anthropic:** an explicit `false` is overwritten with `true`. **Amazon Bedrock:** the per-tool field is removed and fine-grained tool streaming is turned on for the whole request." Measured: streamed through OpenRouter 18 of 36 calls parsed cleanly, non-streamed 36 of 36, direct to Anthropic with `false` 16 of 16. "All 11 failures ended with `finish_reason: \"tool_calls\"`." Source: F-s39. Basis: OR-inferred from an issue report that relays OpenRouter support. Confidence medium: the support statement is second-hand.

**F61. Cache misses from argument key order.** `ai-sdk-provider` issue 171, created 2025-09-06, closed 2026-04-13: "when the `arguments` is serialized in a different order than previous messages, there is no cache hit. I observed this with Anthropic / Claude 4." Source: F-s40. Basis: OR-inferred from an issue report. Fixed in the client (F20).

**F49. Catalog prices differ from billed prices.** `earendil-works/pi` issue 9980, created 2026-09-23, closed 2026-10-22. Measurement by user denispol, 2026-10-20, for `deepseek/deepseek-v4.1-flash-1017`, 256 calls under default routing: billed $0.5231 against pi's catalog estimate $0.3671, ratio 1.42. "The whole gap comes from calls served by an upstream other than the one whose price the catalog holds." and "Even with a pin, a single fallback moves the price". Source: F-s43. Basis: OR-inferred from an issue report with measurements.

**F53. Context limit rejected before routing.** `earendil-works/pi` issue 10497, created 2026-10-05, closed 2026-11-02. Error: `400: {"message":"This endpoint's maximum context length is 1048576 tokens. However, you requested about 1049560 tokens (442154 of text input, 2749 of tool input, 604657 in the output). Please reduce the length of either one, or use the context-compression plugin to compress your prompt automatically.","code":400,"metadata":{"provider_name":null}}`. Maintainer analysis, 2026-11-02: "OpenRouter counted 226,998 + 2,749 = 229,747 input tokens, about 4% more than the provider-reported figure." Source: F-s44. Basis: OR-inferred from an issue report.

### J. Vendor rule behind the Gemini dummy signature

**F58. Google documents the dummy signature.** Google Cloud documentation, "Thought signatures" page, read 2026-11-22: "There are rare cases where you need to provide `functionCall` parts that were not generated by the API and therefore don't have an associated thought signature (for example, when transferring history from a model that does not include thought signatures). You can set `thought_signature` to `skip_thought_signature_validator`, but, this should be a last resort as it will negatively impact model performance." Same page: "In cases of parallel function calls in a single response, only the first `functionCall` part will contain the `thought_signature`." and "Don't merge a `Part` containing a signature with one that does not." The Gemini API page at `ai.google.dev/gemini-api/docs/thought-signatures` now describes the Interactions API and, as fetched today, does not show the dummy value. Source: F-s48. Basis: Vendor-documented.

### K. Counts of special cases

**F67. Special cases recorded per project and per axis.** These are the special cases I read and listed for this file, counted once each. They are a floor, not a census: generic OpenAI-compatible handling that also happens to serve OpenRouter is left out. Reasoning effort and budget controls are counted under "routing and provider options".

| Axis | OpenRouter SDK | opencode | Kilo Code | Cline | Roo Code | pi | Total |
|---|---|---|---|---|---|---|---|
| Reasoning replay | 15 | 8 | 3 | 3 | 7 | 7 | 43 |
| Caching | 8 | 4 | 1 | 2 | 3 | 3 | 21 |
| Usage and cost | 4 | 3 | 2 | 3 | 2 | 3 | 17 |
| Tools | 9 | 0 | 2 | 0 | 4 | 2 | 17 |
| Structured output | 3 | 0 | 0 | 0 | 0 | 0 | 3 |
| Errors and stream failures | 8 | 4 | 0 | 3 | 2 | 5 | 22 |
| Routing and provider options | 6 | 4 | 7 | 5 | 4 | 6 | 32 |
| Headers | 3 | 2 | 1 | 1 | 1 | 1 | 9 |
| Model metadata | 0 | 1 | 1 | 1 | 7 | 5 | 15 |
| Other (annotations, media, images) | 4 | 2 | 1 | 1 | 0 | 0 | 8 |
| Total | 60 | 28 | 18 | 19 | 30 | 32 | 187 |

- The Vercel AI SDK's generic provider has no OpenRouter branch. It shows five gaps instead (F42).
- Reasoning replay is the largest axis in every project that implements it itself, and in OpenRouter's own SDK 9 of the 15 reasoning items are dated bug fixes from 2026 (F3, F5, F6, F7, F8, F10, F12 and the two in F2).
- Source: all source-code keys. Basis: counts are my own tally of the items in sections A to H.

## Conflicts and disagreements

1. **What happens to unsigned Claude reasoning.** OpenRouter's SDK comment (March 2026, F7) says plain reasoning without details "causes the API to construct thinking blocks without valid signatures, which Anthropic rejects". A pi issue (October 2026, F46) measured the opposite: prompt tokens were 590 with and without the unsigned text and 806 with the signature, so the commenter concludes "OpenRouter silently drops unsigned reasoning". pi's changelog repeats it: "OpenRouter then ignored the unsigned thinking" (F40). The two accounts are seven months apart and may describe different inputs (a bare `reasoning` string against a `reasoning.text` entry without `signature`). I cannot tell which holds today. Both end badly for the caller: one is a 400, the other is silent loss that tripled run cost in the reported test.
2. **Whether `index` identifies a block.** OpenRouter's SDK says "`index` cannot be used to reassemble the array" (F1). Issue 582 shows `index: 0` and `index: 1` separating two Anthropic blocks (F47). Roo Code keys its accumulator by type and index (F32). opencode refuses to merge deltas whose `index` differs (F57a). The accounts fit together only per format: Anthropic blocks have distinct indexes, OpenAI summary deltas all say 0. No source gives a rule that holds for every format.
3. **Dedupe.** OpenRouter's SDK dedupes details across messages and says the server does the same (F6). opencode's test expects two identical encrypted entries to both be sent (F57a). A later SDK pull request is titled "fix: scope reasoning_details dedupe keys by type" (listed on issue 254's timeline as pull request 610, state not read), which suggests the SDK's key rule collided across types.
4. **Gemini reasoning text.** OpenRouter's documentation says the sequence of reasoning blocks "must match the outputs generated by the model during the original request; you cannot rearrange or modify the sequence of these blocks" (F-s10). OpenRouter's own SDK removes every Gemini `reasoning.text` entry before replay (F8). OpenRouter staff later wrote that the SDK is "faithfully round-tripping `reasoning_details`" and that the Gemini 400s came from upstream switching (F45). The strip is still in the code at the pinned commit.
5. **The DeepSeek V4 rule.** OpenRouter's SDK satisfies it with `reasoning_details: []` (F12). pi satisfies it with `reasoning_content: ""` (F38). Both are reported as fixes. Either OpenRouter accepts both, or one of them only works on some upstreams.
6. **Cache markers for Gemini.** Roo Code sends markers for 9 Gemini ids (F34). opencode says Gemini "uses only the last breakpoint" and sends none (F19). pi sends markers only for `anthropic/` ids (F36).
7. **BYOK cost arithmetic.** Roo Code always adds `upstream_inference_cost` to `cost` (F35). Kilo Code adds only when upstream exceeds `cost` (F22). pi adds only when `is_byok` is true (F37). Cline adds when `is_byok` is true and both numbers exist, and also reads `market_cost` (F64).
8. **Title header.** `X-Title` (opencode, Kilo Code, Cline, Roo Code) against `X-OpenRouter-Title` (OpenRouter's SDK since 2.4.0, pi). Both are in use (F28).

Against the local survey of 2026-10-28:

- **LC6** (reasoning details must replay unmodified and in order, a replay contract): the contract statement still matches OpenRouter's documentation. The field evidence adds that OpenRouter's own client does not treat the array as opaque. It merges deltas, strips entries by format and signature, dedupes, and drops unknown formats (F1, F6, F7, F8, F13). The `anthropic-claude-v1` tag is confirmed, and the spec now lists 12 format values (F14).
- **LC3** (unsupported parameters are silently ignored): three counterexamples in client code and changelogs. `json_schema.strict: true` filters providers and can end in "No endpoints available matching your guardrail restrictions and data policy" (F27). Conflicting `reasoning_effort` and `reasoning.effort` are rejected (F15). A `configuration_update` system message is rejected for one Claude model with a named error (F40). Also, OpenRouter overwrites a client's `eager_input_streaming: false` instead of ignoring or honoring it (F51).
- **LC7** (cache TTLs dropped, not converted): clients do send `ttl: "1h"` on `cache_control` to OpenRouter for Anthropic models (F17, F19, F36). One user report, kept as a regression test header in OpenRouter's SDK (`e2e/issues/issue-196-anthropic-1h-cache-ttl.test.ts` lines 5-7), says: "when using Anthropic's 1-hour prompt caching via OpenRouter with ttl: \"1h\", they observed 1.25x cache write cost instead of 2x." The issue number (196) falls between issues dated 2025-09-06 and 2025-11-26, so the report is from autumn 2025. It fits "TTL dropped" for the Anthropic direction too. I did not find a later statement that it changed.
- **LC10** (`finish_reason` and `native_finish_reason`): confirmed by opencode's test (F30). The normalized value is still wrong for Gemini 3 tool calls (F11).
- **LC11** (a mid-stream error arrives as a chunk with `finish_reason: "error"`): clients key on a top-level `error` object in the chunk, and also on `finish_reason: "error"` (F23, F30, F41). Both shapes are handled, so the survey's statement is right and incomplete.
- **LC1, LC2, LC4, LC13, LC17**: consistent with client code. pi uses the Messages surface and opencode the Responses surface with `store: false` (F40, F16). `debug.echo_upstream_body` exists and the SDK comment adds "Only works with streaming requests." (F27). `error_type` appears in a real error body, `"metadata": { "error_type": "invalid_request" }` (issue 491, F-s36). `cache_write_tokens` is in the SDK schema (F21).

## Open questions blocked by closed source

1. **Does OpenRouter drop an unsigned Claude reasoning entry, or forward it and let Anthropic reject it?** Tried: the SDK source and changelog, pi's issue and changelog, OpenRouter's reasoning documentation. Fill found: two conflicting accounts (F7, F46). No fill settles it.
2. **Where does OpenRouter put each `reasoning_details` entry when it rebuilds Anthropic's content list?** The request has one flat array beside `content` and `tool_calls` (F9). Issue 582 shows that two signed entries are accepted (F47), so OpenRouter can rebuild more than one block, but no source states how it orders blocks against text and tool calls. Fill: none for OpenRouter. The workspace report of 2026-11-17 notes the same guess problem for LiteLLM and agentgateway.
3. **Does OpenRouter's router now keep the same upstream when provider-scoped state is present?** In June 2026 it did not (F45), and staff called it a backend fix. Tried: SDK changelog and changesets after June, no mention. Fill: the client workaround is `provider: { order: [...], allow_fallbacks: false }` (F45, F35).
4. **What are the server's dedupe rules for reasoning details?** Only an SDK comment naming `openrouter-web/packages/llm-interfaces/reasonings/duplicate-tracker.ts` (F6). Fill: the SDK's copy of the key rule.
5. **How does a Gemini `reasoning.encrypted` entry map to request parts?** Roo Code's comment and one issue example say `id` is the tool call id and `data` is the signature (F31, F45). What `id` a text-only signature gets is not stated anywhere I read. Fill: Roo Code's injection recipe, partial.
6. **Is `cache_control` on a chat-format tool definition honored?** The SDK changeset says OpenRouter "does not document it" (F18). pi and opencode send it for Claude. Fill: none. pi avoids the question for Claude by using the Messages route (F40).
7. **Does OpenRouter still force `eager_input_streaming: true`?** The issue is open and the statement is relayed from support (F51). Fill: none.
8. **Where does `[REDACTED]` in reasoning text come from?** opencode strips it and blames OpenRouter (F55). OpenRouter's SDK produced the same string before 2.4.1 (F10). Fill: none.

## Unverified

- I did not run any of the code. The claim that OpenRouter's SDK drops details with formats outside its six-value enum (F14) follows from reading two files. The same holds for the claim that Cline's current path does not replay Claude signatures through OpenRouter (F43).
- Four issue pages were read only as search-result excerpts: opencode issues 4608, 14716, 21263 and pull request 19048 (F50, F52, F62). Dates, states and quoted strings come from those excerpts.
- Roo Code's archive status comes from third-party pages (F-s49). The repository page answered HTTP 200 and the last commit date matches, but I did not read the archive banner on GitHub myself.
- That the removed opencode `provider/transform.ts` matched what Kilo Code's fork now holds is an assumption from the fork relationship and the `.opencode-version` file. I did not read opencode's history. Lines marked `kilocode_change` in that file are Kilo's own.
- The task description's expectation that Cline has an `openrouter-stream.ts` transform was not checked against history. It is not in the tree at the pinned commit.
- Survey claim LC12 (the final usage chunk has a non-empty `choices` array) was not checked. The SDK tolerates both shapes (`if (choice?.delta == null) { return; }`).
- The special-case counts (F67) are my own tally, and the split of items between axes involves judgment.
- From memory, not sourced today: that OpenRouter's SDK defaulted `usage` accounting to off in older versions, and that `X-Title` was OpenRouter's originally documented title header.

## Workspace documents used

- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, dated 2026-10-28. Took the OpenRouter claims LC1 to LC19 as the list to test. Re-checked against live client source: LC1, LC2, LC3, LC4, LC6, LC7, LC10, LC11, LC13, LC17. LC3 needs qualifying, the others hold with the additions listed under Conflicts. Not re-checked by me: LC5, LC8, LC9, LC12, LC14, LC15, LC16, LC18, LC19.
- "Report: Anthropic Messages API harness survey", dated 2026-11-17 (cabinet output). Took three points: that three conventions exist for returning thinking through chat completions (`thinking_blocks`, `reasoning_details`, `reasoning_signature`), that the translator has to guess block positions, and its table row for opencode at commit `2aed822` ("Signature in part metadata"). Re-checked live: opencode is now at `71cd8c0` and has its own native OpenRouter provider, and the part-metadata statement still holds with the field names in F57a. Its statement that pi drops thinking produced by another model was not re-checked.
- `workspace-index.tsv` in the shared scratch folder, 2026-11-22, used to find the two documents above.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| F-s1 | `OpenRouterTeam/ai-sdk-provider` at `7a235a4c`, `src/chat/index.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s2 | same, `src/chat/convert-to-openrouter-chat-messages.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s3 | same, `src/schemas/reasoning-details.ts`, `src/schemas/format.ts`, `src/schemas/provider-metadata.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s4 | same, `src/utils/reasoning-details-duplicate-tracker.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s5 | same, `src/chat/schemas.ts`, `src/schemas/error-response.ts`, `src/utils/map-finish-reason.ts`, `src/utils/compute-token-usage.ts`, `src/utils/with-stream-error-handling.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s6 | same, `src/provider.ts`, `src/types/index.ts`, `src/types/openrouter-chat-settings.ts`, `src/types/openrouter-chat-completions-input.ts`, `src/utils/resolve-reasoning.ts`, `src/utils/merge-provider-routing.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s7 | same, `CHANGELOG.md` and `.changeset/*.md` (35 pending changesets) | 2026-11-22 | OpenRouter SDK source |
| F-s8 | same, `e2e/issues/*.test.ts` headers and `src/chat/signature-roundtrip.test.ts` | 2026-11-22 | OpenRouter SDK source |
| F-s9 | `https://openrouter.ai/openapi.json`, `components.schemas.ReasoningFormat`, `ReasoningDetailUnion`, `ReasoningDetailEncrypted`, `ReasoningDetailServerToolCall` | 2026-11-22 | OpenRouter spec |
| F-s10 | `https://openrouter.ai/docs/guides/best-practices/reasoning-tokens` | 2026-11-22 | OpenRouter doc |
| F-s11 | `anomalyco/opencode` at `71cd8c06`, `packages/ai/src/providers/openrouter.ts` | 2026-11-22 | harness source |
| F-s12 | same, `packages/ai/src/protocols/openai-chat.ts` | 2026-11-22 | harness source |
| F-s13 | same, `packages/ai/src/cache-policy.ts`, `packages/ai/src/protocols/utils/cache.ts`, `packages/ai/README.md` | 2026-11-22 | harness source |
| F-s14 | same, `packages/ai/src/provider-error.ts`, `packages/ai/src/protocols/shared.ts`, `packages/ai/src/schema/events.ts` | 2026-11-22 | harness source |
| F-s15 | same, `packages/core/src/plugin/provider/openrouter.ts`, `packages/core/src/aisdk-native.ts`, `packages/core/src/variant.ts`, `packages/tui/src/routes/session/message-parts.tsx` | 2026-11-22 | harness source |
| F-s16 | same, `packages/ai/test/provider/openrouter.test.ts` | 2026-11-22 | harness source |
| F-s17 | `Kilo-Org/kilocode` at `8045e72d`, `packages/opencode/src/provider/transform.ts` | 2026-11-22 | harness source |
| F-s18 | same, `packages/opencode/src/kilocode/session/reasoning-replay.ts`, `packages/opencode/src/session/processor.ts` | 2026-11-22 | harness source |
| F-s19 | same, `packages/opencode/src/kilocode/session/index.ts`, `packages/opencode/src/session/session.ts` | 2026-11-22 | harness source |
| F-s20 | same, `packages/core/src/plugin/provider/openrouter.ts`, `packages/opencode/src/kilocode/provider/image-output.ts`, `packages/llm/src/providers/openrouter.ts`, `.opencode-version` | 2026-11-22 | harness source |
| F-s21 | `cline/cline` at `8075d2e3`, `sdk/packages/llms/src/providers/routing/provider-option-rules.ts`, `routing/reasoning-codecs.ts` | 2026-11-22 | harness source |
| F-s22 | same, `sdk/packages/llms/src/providers/vendors/openai-compatible.ts`, `ai-sdk.ts`, `builtins.ts`, `request-headers.ts`, `vendors/types.ts` | 2026-11-22 | harness source |
| F-s23 | same, `sdk/packages/llms/src/providers/error-classification.ts`, `middleware/retry-empty-response.ts`, `sdk/packages/agents/src/agent-runtime.ts` | 2026-11-22 | harness source |
| F-s24 | same, `apps/vscode/src/shared/messages/content.ts` | 2026-11-22 | harness source |
| F-s25 | `RooCodeInc/Roo-Code` at `b867ec91`, `src/api/providers/openrouter.ts` | 2026-11-22 | harness source |
| F-s26 | same, `src/api/transform/openai-format.ts`, `src/api/transform/caching/anthropic.ts`, `src/api/transform/caching/gemini.ts`, `src/api/transform/reasoning.ts` | 2026-11-22 | harness source |
| F-s27 | same, `src/api/providers/fetchers/openrouter.ts`, `packages/types/src/providers/openrouter.ts`, `src/api/providers/constants.ts`, `src/core/task/Task.ts` | 2026-11-22 | harness source |
| F-s28 | `earendil-works/pi` at `9c465268`, `packages/ai/src/api/openai-completions.ts` | 2026-11-22 | harness source |
| F-s29 | same, `packages/ai/catalog/providers/openrouter.ts`, `packages/ai/catalog/sources/openrouter.ts`, `packages/ai/src/providers/openrouter.ts` | 2026-11-22 | harness source |
| F-s30 | same, `packages/ai/src/utils/openrouter-cost.ts`, `utils/retry.ts`, `utils/overflow.ts`, `utils/identity.ts`, `api/simple-options.ts`, `api/anthropic-messages.ts`, `packages/ai/catalog/families/deepseek.ts` | 2026-11-22 | harness source |
| F-s31 | same, `packages/ai/CHANGELOG.md` | 2026-11-22 | harness source |
| F-s32 | `vercel/ai` at `d55f3cfc`, `packages/openai-compatible/src/chat/*.ts` | 2026-11-22 | harness source |
| F-s33 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/423` (created 2026-02-19, closed) | 2026-11-22 | issue report (third-party) |
| F-s34 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/418` (created 2026-02-14, closed) | 2026-11-22 | issue report (third-party) |
| F-s35 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/254` (created 2025-11-26, closed) | 2026-11-22 | issue report (third-party) |
| F-s36 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/491` (created 2026-04-30, closed 2026-06-17) | 2026-11-22 | issue report (third-party) |
| F-s37 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/582` (created 2026-10-21, closed) | 2026-11-22 | issue report (third-party) |
| F-s38 | `https://github.com/OpenRouterTeam/ai-sdk-provider/pull/608` (opened 2026-10-23, open) | 2026-11-22 | issue report (third-party) |
| F-s39 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/574` (created 2026-10-09, open) | 2026-11-22 | issue report (third-party) |
| F-s40 | `https://github.com/OpenRouterTeam/ai-sdk-provider/issues/171` (created 2025-09-06, closed 2026-04-13) | 2026-11-22 | issue report (third-party) |
| F-s41 | `https://github.com/cline/cline/issues/8214` (created 2025-12-19, closed 2026-01-10) | 2026-11-22 | issue report (third-party) |
| F-s42 | `https://github.com/earendil-works/pi/issues/9534` (created 2026-09-12, closed 2026-10-28) | 2026-11-22 | issue report (third-party) |
| F-s43 | `https://github.com/earendil-works/pi/issues/9980` (created 2026-09-23, closed 2026-10-22) | 2026-11-22 | issue report (third-party) |
| F-s44 | `https://github.com/earendil-works/pi/issues/10497` (created 2026-10-05, closed 2026-11-02) | 2026-11-22 | issue report (third-party) |
| F-s45 | `https://github.com/anomalyco/opencode/issues/4608` (created 2025-11-21, closed 2026-01-16), search-result excerpt | 2026-11-22 | issue report (third-party) |
| F-s46 | `https://github.com/anomalyco/opencode/issues/14716` (created 2026-02-22, closed 2026-05-22), search-result excerpt | 2026-11-22 | issue report (third-party) |
| F-s47 | `https://github.com/anomalyco/opencode/issues/21263` (created 2026-04-07, closed) and opencode pull request 19048 (2026-03-25, read through a mirror excerpt) | 2026-11-22 | issue report (third-party) |
| F-s48 | `https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/thinking/thought-signatures`, and `https://ai.google.dev/gemini-api/docs/thought-signatures` | 2026-11-22 | vendor doc |
| F-s49 | `https://www.edgewisely.com/cline-vs-roo-code/` (2026-10-31), `https://taku.ai/blog/roo-code-alternatives` (2026-11-10), `https://github.com/stacklok/toolhive/issues/5017` | 2026-11-22 | blog (third-party) |
| F-s50 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, and the 2026-11-17 report "Anthropic Messages API harness survey" | 2026-11-22 | workspace document |

