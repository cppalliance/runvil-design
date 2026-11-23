---
produced: 2026-11-22
title: OpenRouter deep dive - Google Gemini vendor-native features (thought signatures, thinking levels, long decode continuation token, cachedContents, safety settings, grounding, code execution, Files API, Interactions API, function calling, structured output, OpenAI-compatible route) and how OpenRouter exposes or flattens each
---

# OpenRouter deep dive - Google Gemini vendor-native features and how OpenRouter exposes or flattens each

- What was read: Google's live Gemini API documentation in both its `generateContent` and Interactions API variants, the Gemini API reference, Google Cloud's (Vertex) thought signature page, the local mirror of OpenRouter's documentation, OpenRouter's OpenAPI spec, model catalog and public per-model endpoint lists, OpenRouter's official AI SDK provider source, and four open-source projects read at pinned commits as fills (LiteLLM, Vercel AI SDK, Gemini CLI, agentgateway).
- How many sources: 57 (listed in the Sources table), giving 108 evidence items and 29 candidate matrix rows.
- Read date: 2026-11-22 for every source. Google's `https://ai.google.dev/gemini-api/docs/<page>` URLs now redirect to `https://aistudio.google.com/docs/<page>`. Each page has two variants: the default one documents the Interactions API, and `?generate-content=true` documents `generateContent`. The Markdown form is the same URL with `.md` before the query string.

## Candidate matrix rows

Reading notes: "native" means Google's `generateContent` API unless the cell says Interactions. The Basis column labels the "How OpenRouter exposes it" cell; every "Native rule" cell is Vendor-documented.

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
| - | - | - | - | - | - | - | - |
| Thought signature on function calls | Google (Gemini 3, Gemini 4; optional on 2.5) | `thoughtSignature` on the first `functionCall` part of each step. Must come back in the same part. Missing in the current turn: HTTP 400 "Function call ... is missing a `thought_signature`". Only the current turn is checked. | Entries in `message.reasoning_details` with `type: "reasoning.encrypted"`, `format: "google-gemini-v1"`, opaque `data`. Caller sends the whole array back on the assistant message. Tool call object holds `id`, `type`, `function` only. | Position: the signature sits beside the tool calls, not on one. The value is wrapped (a sample shows a second segment naming the issuing endpoint). Server-side pairing rule is unpublished. | Caller holds state. Opaque round trip, byte for byte. No vendor-specific values to pick. Works on chat completions. Caller code is vendor-neutral if it replays `reasoning_details` for every model. | OR-documented (spec plus official SDK source) | GG1 to GG5, GG17 to GG22 |
| Thought signature on text, final, and image parts | Google | Last part of a reply without function calls may hold a signature. Not validated; "performance may degrade" without it. Image models sign every image part. Streaming may deliver it on an empty text part. | Same `reasoning_details` array. OpenRouter's SDK drops Gemini `reasoning.text` entries on replay and keeps `reasoning.encrypted`. | Per-part placement. Image-part signatures: handling not documented. | Caller holds state. Round trip optional for correctness, needed for quality. Same opaque field. | OR-documented (official SDK source) | GG7, GG10, GG14, GG20, GG96 |
| Dummy signature for history from another model | Google | Set the signature field to `"skip_thought_signature_validator"` (AI Studio also lists `"context_engineering_is_the_way_to_go"`). Vertex: "last resort", "will negatively impact model performance". | Not documented. OpenRouter states `reasoning_details` lets callers switch models "without changing your code structure". | Unknown whether OpenRouter injects it or lets the 400 through. | A translator can do this alone: it sees a `functionCall` with no signature and fills the dummy on the first call of each step. No caller state. | Filled (LiteLLM, Vercel AI SDK, Gemini CLI) | GG8, GG24, GG28, GG30, GG31 |
| Signature validity across Google AI Studio and Vertex | Google | Google's pages are silent. Vertex: Flash-Lite drops thoughts made by non-Flash-Lite models. Interactions API: "The backend manages compatibility" on model switch. | Wrapped signature sample records an `endpoint_slug` that names the model version and `google-vertex/global`. Continuation tokens are pinned: "A continuation token only works on the provider that issued it". Ordinary routing is soft sticky and can change provider. | Not stated for ordinary signatures. | The opaque value must be tagged with the issuing provider and model, and the translator must drop or replace it (dummy) when the target differs. Caller stays vendor-blind only if the gateway does this. | OR-inferred (decoded sample in OpenRouter docs; LiteLLM comment as fill) | GG15, GG16, GG22, GG23, GG29, GG30, GG101 |
| Thinking level and budget | Google | Gemini 3 and 4: `thinkingConfig.thinkingLevel` (`minimal`, `low`, `medium`, `high`, `xhigh`; set differs per model). Gemini 2.5: `thinkingBudget` (0 off, -1 dynamic). Gemini 4, Gemini 3, and 2.5 Pro cannot turn thinking fully off (`minimal` is near-off on some Gemini 3 Flash models). Budget plus level on Gemini 4: 400. | `reasoning.effort` maps to `thinkingLevel`, nearest supported level; `reasoning.max_tokens` maps to `thinkingBudget`; catalog `reasoning` object lists `supported_efforts`, `default_effort`, `mandatory`. Gemini 4 Argon: `reasoning.max_tokens` "is not supported". | Token budgets on Gemini 3 and 4. `effort: "none"` is rejected when `mandatory` is true. | Gateway-only translation of a neutral setting. No state. A plain on/off switch cannot be honored on models where thinking is mandatory; the gateway needs a per-model table (the Gemini model list gives a boolean only). | OR-documented | GG33, GG34, GG38, GG39, GG108 |
| Thought summaries | Google | `includeThoughts: true` returns parts with `thought: true`. Interactions: `thinking_summaries` `auto` or `none`, `thought` steps. | Returned as `message.reasoning` and `reasoning.text` details; `reasoning.exclude: true` hides them. | The upstream flag is not exposed; summaries arrive by default. | Gateway-only. No state. Neutral on/off for display. | OR-documented | GG35, GG36, GG40 |
| Long decode continuation token | Google (Gemini 4) | Hop cap 65,536 (32,768 non-streaming REST on Carbon and Argon). `finishReason: CONTINUATION` plus `continuationToken`. Next request: same `contents` plus top-level `continuationToken`; no partial output added. `maxOutputTokens` is the total; after `MAX_TOKENS` it must rise or 400. Stream checkpoints every 5,000 tokens. | `finish_reason: "length"`, `native_finish_reason: "CONTINUATION"` or `"MAX_TOKENS"`, token as `reasoning_details` entry with `id: "gemini-continuation"`, `format: "google-gemini-v1"`. Caller appends the stopped assistant turn and resends; OpenRouter strips it before forwarding. | Token position (top-level field becomes a history item). Usage is per hop. Token "can reach several MB". Explicit caching skipped on continuation hops. | Caller holds state (the token). Opaque round trip inside the same reasoning field. Caller needs a loop keyed on finish reason, must raise `max_tokens`, and must stay on one provider. A caller that ignores it gets a normal length stop. | OR-documented | GG41 to GG50 |
| Implicit caching | Google (2.5 and later) | On by default. Minimum 4,096 tokens (3.x, 4) or 2,048 (2.5). Gemini 4: 95 percent discount, full previous request cached only if 2,050 tokens were added. `cachedContentTokenCount` in usage. | Automatic. Sticky provider routing (60 minutes, or `session_id`). Reported in `usage.prompt_tokens_details.cached_tokens`. | Nothing the caller sets. OpenRouter's stated figures (0.25x, 1024 minimum) differ from Google's current ones. | Gateway-only. State is on Google's side. Benefit depends on stable prefixes and on staying with one provider. | OR-documented | GG51, GG55, GG57, GG58 |
| Named caches (`cachedContents`) | Google (Gemini 3 and 2.5 only) | Create a cache object, reference it as `cachedContent: "cachedContents/{id}"`. TTL default 1 hour; only expiry is updatable; `model`, `systemInstruction`, `contents`, `tools`, `toolConfig` immutable. Storage billed per million tokens per hour. Unavailable on Gemini 4 and on the Interactions API. | Caller marks content with Anthropic-style `cache_control`; OpenRouter creates and manages the cache object. "only the last breakpoint" counts. Fixed 5 minute TTL. Write billed as input plus 5 minutes of storage. | Cache names, TTL choice, explicit update and delete, reuse across conversations by name. `ttl` is dropped toward Google. | Gateway holds the cache handle. No caller state. Caller uses a neutral marker. Behavior on Gemini 4 undocumented. | OR-documented | GG52 to GG54, GG56, GG57 |
| Safety settings | Google | `safetySettings` list of `category` and `threshold`; default Off on 2.5 and later; core harms always blocked. | No chat field for it. `provider.options` keyed by `google-ai-studio` or `google-vertex` exists in the spec; forwarded keys are not published for chat models. | The whole control, as far as the documentation shows. | Vendor-specific values (Google's category and threshold names). A caller that sets them must know it talks to Gemini; a gateway default avoids that. | OR-inferred (spec has `ProviderOptions` and no `safety_settings`) | GG59, GG65 |
| Blocked prompt with no candidate | Google | HTTP 200, zero candidates, `promptFeedback.blockReason` (`SAFETY`, `OTHER`, `BLOCKLIST`, `PROHIBITED_CONTENT`, `IMAGE_SAFETY`). | Error with `error_type: "content_policy_violation"`, status 403, with `metadata.provider_name`, `metadata.provider_code`, `metadata.raw`. | `safetyRatings` detail moves into `metadata.raw`. Exact mapping unpublished. | Gateway-only normalization. No state. | OR-documented (one example names Gemini) | GG60, GG64, GG67 |
| Finish reasons | Google | 23 `finishReason` values, including `SAFETY`, `RECITATION`, `PROHIBITED_CONTENT`, `MALFORMED_FUNCTION_CALL`, `MAX_TOKENS`, `MISSING_THOUGHT_SIGNATURE`, `CONTINUATION`. A function call ends with `STOP`. | `finish_reason` normalized to `tool_calls`, `stop`, `length`, `content_filter`, `error`; raw value in `native_finish_reason`. OpenRouter's SDK notes Gemini 3 tool-call replies arriving as `stop`. | The distinction among Google's values, except through `native_finish_reason`. | Gateway-only. Translator must set the tool-call reason itself when the reply holds a function call. | OR-documented | GG61, GG21, GG47, GG64, GG67 |
| Google Search grounding | Google | Tool `google_search`. Reply has `groundingMetadata` (`webSearchQueries`, `searchEntryPoint`, `groundingChunks`, `groundingSupports`). Search Suggestions display is required by terms. $14 per 1,000 queries. | `{"type": "openrouter:web_search"}` server tool or `web` plugin with engine `auto` or `native`; endpoint `native_tools` shows `google_search`. Results as `annotations` of type `url_citation`. Catalog price `web_search: 0.014`. | Search entry point HTML, queries, segment-to-source index. Domain filters (400 or Exa fallback). | Gateway-run or vendor-run tool behind a neutral tool name. No caller state for search alone. | OR-documented | GG68, GG71, GG72 |
| URL context | Google | Tool `url_context`; up to 20 URLs, 34MB each; `url_context_metadata` in the reply. | Not documented. OpenRouter's own `openrouter:web_fetch` tool is the nearest neutral feature; Gemini endpoints list no native fetch tool. | Native tool and its metadata. | Would need a vendor tool passthrough or a gateway-run fetch. | OR-inferred (endpoint `native_tools`, spec search) | GG69, GG72 |
| Maps grounding | Google | Tool `googleMaps`; attribution rules; `google_maps_widget_context_token`; $25 per 1,000 grounded prompts. | Not documented. | Whole feature. | Vendor-specific tool with display duties on the application. | OR-inferred (terms missing in docs) | GG69, GG72 |
| Code execution | Google | Tool `code_execution`; reply parts `executableCode` and `codeExecutionResult` with `id`; Python only; 30 second runtime. | Not documented for Gemini. | Whole feature, including its part types. | Part types have no chat completions slot; they would need the opaque block list to survive a round trip. | OR-inferred (endpoint `native_tools`, server tool list) | GG70, GG72 |
| Built-in tools combined with custom functions | Google (Gemini 3 and 4) | `include_server_side_tool_invocations: true`; reply holds `toolCall` and `toolResponse` parts with `id`, `tool_type`, and a signature on every part; all must come back or "the model will error out". | Not documented. Native search results arrive as annotations, not parts. | Server-side tool parts and their signatures. | Caller would hold state it cannot read. Round trip of the full ordered part list is required. Only an opaque block list covers it. | OR-inferred (no slot in spec) | GG13, GG72 |
| Files API and file URIs | Google | Upload once, reference as `file_data.file_uri`; 2 GB per file, 20 GB per project, deleted after 48 hours; free. Also public or signed HTTPS URLs up to 15 MB. | Images, PDFs, audio, video as URL or base64 parts. `gs://bucket/object` forwarded only on the caller's own Vertex key. | Files API handles. | State is in the Google project that uploaded. A gateway that owns the key must do the upload and hold the handle. | OR-documented (for `gs://`), OR-inferred (Files API terms missing) | GG73, GG74, GG77, GG78 |
| Video, audio, YouTube input | Google | Inline up to 100 MB; YouTube URL as `file_uri` (preview, public videos only). | `video_url` part: YouTube links on Google AI Studio only; Vertex takes base64 only. `processing: "agentic"` or `"static"` on the video part, routed to an endpoint that supports it. | Nothing for YouTube on AI Studio. Provider must be known or pinned for URL input. | Caller must know which Google provider serves the request to choose URL or base64, unless the gateway converts. `processing` is a Gemini-only value. | OR-documented | GG74, GG76, GG102 |
| Media resolution | Google | `media_resolution` global and per part (`MEDIA_RESOLUTION_LOW`, `MEDIUM`, `HIGH`, per part `ULTRA_HIGH`). | Not documented for chat models. | Whole control. | A neutral image `detail` hint could be translated by the gateway. No state. | OR-inferred (terms missing in docs) | GG75, GG78 |
| Interactions API | Google | Default API since June 2026. Server stores interactions (55 days paid); `previous_interaction_id`; typed `steps`; `store: false` for stateless; background execution. `tools`, `system_instruction`, `generation_config` are sent again each time. Explicit caching and batch are unavailable on it. | Not exposed. Gemini is served on `/api/v1/chat/completions`, `/api/v1/completions`, `/api/v1/messages`, `/api/v1/responses`, all stateless. | Server-held state, background runs, step vocabulary. | Server-held state exists only on a non-chat-completions route. The stateless mode has the same replay duty as `generateContent` ("You **MUST** always resend all `thought` blocks"). | OR-documented (catalog routes; upstream API inferred) | GG16, GG79 to GG82 |
| Function calling modes | Google | `functionCallingConfig.mode`: `AUTO`, `ANY`, `NONE`, `VALIDATED`; `allowedFunctionNames` with `ANY` or `VALIDATED`. | Standard `tool_choice`; endpoints report `none`, `auto`, `required`, `function` all supported. | `VALIDATED` mode and multi-name allow lists. | Gateway-only translation of neutral values. | OR-documented (support flags), mapping OR-inferred | GG83, GG84 |
| Function call ids and parallel ordering | Google | Gemini 3 and later return an `id` per call, to be echoed. Parallel: all calls, then all responses; interleaving is a 400. Vertex matches by `id` or by name and order. | Standard `tool_calls[].id` and `role: "tool"` messages with `tool_call_id`. | Whether ids are Google's or synthesized is unpublished. | Gateway-only: regroup tool messages into one user content in call order. No state. | OR-inferred (standard shape; agentgateway and LiteLLM as fills) | GG9, GG85, GG86 |
| Function schema subset, roles, system instruction | Google | Declarations use a subset of OpenAPI schema; complex schemas can be rejected with `400 INVALID_ARGUMENT`. Roles are `user` and `model`. One text-only `systemInstruction`. Names: letters, digits, underscores, dashes, 128 maximum. | Standard `tools` and `messages`. OpenRouter notes "Gemini has a single `systemInstruction` field". | Schema down-conversion rules unpublished. | Gateway-only. No state. Failure mode is a vendor 400 on rich schemas. | OR-inferred | GG87, GG88, GG89 |
| Structured output | Google | `responseMimeType: "application/json"` with `response_json_schema`; JSON Schema subset; unsupported properties ignored; too-complex schemas rejected. Works with function calling on Gemini 3 and 4. | Standard `response_format` and `structured_outputs`, support marked per endpoint. | Subset handling unpublished. | Gateway-only translation of a standard field. | OR-documented | GG90, GG91 |
| Sampling parameters ignored | Google (3.6 and later, 4) | `temperature`, `top_p`, `top_k` ignored with deprecation warning headers, 400 announced for a future release. `candidateCount` unsupported on 3 and 4. | Catalog still lists `temperature`, `top_p`, `seed` as supported on Gemini 4; `stop` listed on Vertex endpoints only. | The warning headers. | Gateway-only: drop the fields for these models. Temperature set by a prompt has no effect on Gemini 4. | OR-documented (catalog) | GG92, GG93, GG100 |
| Image output | Google | `responseModalities`; image parts each with a signature. | Images returned in `message.images[].image_url.url`; image models also listed under `/api/v1/images`. | Per-image signature handling unpublished. | Opaque round trip needed for multi-turn editing. Non-standard response field. | OR-documented (official SDK source, catalog) | GG14, GG96 |
| `usageMetadata` | Google | `promptTokenCount` (includes cached), `cachedContentTokenCount`, `candidatesTokenCount` (excludes thoughts), `thoughtsTokenCount`, `toolUsePromptTokenCount`, `totalTokenCount`. | `usage.completion_tokens_details.reasoning_tokens`, `usage.prompt_tokens_details.cached_tokens` and `cache_write_tokens`. | `toolUsePromptTokenCount` and per-modality detail not shown. | Gateway-only arithmetic: completion tokens are candidates plus thoughts. Long decode needs first-hop prompt size for context checks. | OR-documented | GG94, GG95, GG45 |
| Google's own OpenAI-compatible route | Google | `/v1beta/openai/chat/completions`. Signatures in `tool_calls[].extra_content.google.thought_signature`. `reasoning_effort` mapped. `extra_body.google.thinking_config` and `cached_content` on chat. Long decode unavailable. Beta. | Not used by OpenRouter as far as shown (its `native_finish_reason` values and field names are `generateContent` ones). | Not applicable. | A first-party example of flattening: signature rides on the tool call, text-part signatures have no slot, long decode is lost, safety settings on chat are undocumented. An existing forward-as-written gateway keeps signatures only if the client echoes `extra_content`. | Filled (Google, first-party) | GG25, GG26, GG97 to GG99 |

## Evidence

### A. Thought signatures - Google's native rule

**GG1. On `generateContent` a signature is a field on a content part, and the general rule is to send it back unchanged.**
- Quote: "When using thinking models (such as the Gemini 4, Gemini 3, and 2.5 series), the API may return a `thoughtSignature` field within the [content parts] of the response (e.g., `text` or `functionCall` parts)."
- Quote: "As a general rule, if you receive a thought signature in a model response, you should pass it back exactly as received when sending the conversation history in the next turn."
- Quote (API reference, `Part`): "**thoughtSignature**: `string` Optional. An opaque signature for the thought so it can be reused in subsequent requests."
- Source: GG-s20, GG-s30. Basis: Vendor-documented.

**GG2. On Gemini 4 and Gemini 3 the signature is mandatory during function calling, at every thinking level.**
- Quote: "**When using Gemini 4 and Gemini 3 models, you must pass back thought signatures during function calling, otherwise you will get a validation error** (4xx status code). This includes when using the `minimal` [thinking level] setting for Gemini 3 Flash."
- Source: GG-s20. Basis: Vendor-documented.

**GG3. Which part holds the signature.**
- Quote: "When there are [parallel function] calls, the first function call part returned by the model response will have a thought signature."
- Quote: "When there are sequential function calls (multi-step), each function call will have a signature and you must pass all signatures back."
- Quote: "Model responses without a function call will return a thought signature inside the last part returned by the model."
- Quote: "**Parallel Function Calls**: If the model generates parallel function calls in a response, the `thought_signature` is attached **only to the first** `functionCall` part. Subsequent `functionCall` parts in the same response will **not** contain a signature."
- Source: GG-s20. Basis: Vendor-documented.

**GG4. What is validated: only the current turn, and only the first `functionCall` part of each step in it.**
- Quote: "**Validation**: Strict validation is enforced for all function calls within the current turn . (Only current turn is required; we don't validate on previous turns)"
- Quote: "The API goes back in the history (newest to oldest) to find the most recent **User** message that contains standard content (e.g., `text`) (which would be the start of the current turn). This will not **be** a `functionResponse`."
- Quote: "The **first** `functionCall` part in **each step** of the current turn **must** include its `thought_signature`."
- Quote: "If you omit a `thought_signature` for the first `functionCall` part in any step of the current turn, the request will fail with a 400 error."
- Quote: "**Requirement**: You **must** return this signature in the exact part where it was received when sending the conversation history back."
- Source: GG-s20. Basis: Vendor-documented.

**GG5. The error for a missing signature on `generateContent` is an HTTP 400 with this message form.**
- Quote: "Gemini 3 models: Failure to include signatures will result in a 400 error. The verbiage will be of the form: Function call `<Function Call>` in the `<index of contents array>` content block is missing a `thought_signature`. For example, *Function call `FC1` in the `1.` content block is missing a `thought_signature`.*"
- Source: GG-s20. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (same message text).

**GG6. The token `missing_thought_signature` exists in two other places, and neither is the HTTP 400 above.**
- Quote (`generateContent` reference, `Candidate.finishReason` enum): "`MISSING_THOUGHT_SIGNATURE`: Request has at least one thought signature missing."
- Quote (Interactions API errors page, "Generation error codes", described as codes that "indicate a structural issue with the model's generated output"): "| `missing_thought_signature` | The response is missing a required thought signature. |"
- Source: GG-s30, GG-s31. Basis: Vendor-documented. Confidence note: the two descriptions disagree on whether the request or the response lacks the signature. Google's `generateContent` error page lists no code of this name among its HTTP errors (its body uses `code: 400`, `status: "INVALID_ARGUMENT"`).

**GG7. Signatures on parts that are not function calls: sending them back is recommended, the API does not check them, and quality can drop without them.**
- Quote: "**Behavior**: The final content part (`text, inlineData…`) returned by the model may contain a `thought_signature`."
- Quote: "**Recommendation**: Returning these signatures is **recommended** to ensure the model maintains high-quality reasoning, especially for complex instruction following or simulated agentic workflows."
- Quote: "**Validation**: The API does **not** strictly enforce validation. You won't receive a blocking error if you omit them, though performance may degrade."
- Source: GG-s20. Basis: Vendor-documented.

**GG8. The documented dummy value for history that did not come from Gemini.**
- Quote (AI Studio page): "While injecting custom function call blocks into the request is strongly discouraged, in cases where it can't be avoided, e.g. providing information to the model on function calls and responses that were executed deterministically by the client, or transferring a trace from a different model that does not include thought signatures, you can set the following dummy signatures of either `"context_engineering_is_the_way_to_go"` or `"skip_thought_signature_validator"` in the thought signature field to skip validation."
- Quote (Vertex page): "You can set `thought_signature` to `skip_thought_signature_validator`, but, this should be a last resort as it will negatively impact model performance."
- Source: GG-s20, GG-s47. Basis: Vendor-documented. Note: the Vertex page names one value, the AI Studio page names two.

**GG9. Parallel calls and their responses must be grouped, not interleaved.**
- Quote (AI Studio): "When the API returns parallel function calls "FC1 + signature, FC2", the user response expected is "FC1+ signature, FC2, FR1, FR2". If you have them interleaved as "FC1 + signature, FR1, FC2, FR2" the API will return a 400 error."
- Quote (Vertex): "Return `functionResponse` parts in the same order as the corresponding `functionCall` parts. The API matches parallel function responses to function calls by `id` (if provided in `v1beta1`), or by `name` and sequential order in the `parts` array."
- Source: GG-s20, GG-s47. Basis: Vendor-documented.

**GG10. In a streamed reply with no function call, the signature can arrive in a part whose text is empty.**
- Quote: "During a model response not containing a FC with a streaming request, the model may return the thought signature in a part with an empty text content part. It is advisable to parse the entire request until the `finish_reason` is returned by the model."
- Source: GG-s20. Basis: Vendor-documented.

**GG11. The rule per model generation.**
- Quote: "If there are function calls in a response, Gemini 4 and Gemini 3 will always have the signature on the first function call part. It is **mandatory** to return that part. Gemini 2.5 will have the signature in the first part (regardless of type). It is **optional** to return that part."
- Quote: "If there are no function calls in a response, Gemini 4 and Gemini 3 will have the signature on the last part if the model generates a thought. Gemini 2.5 won't have a signature in any part."
- Quote (thinking page): "**Gemini 2.5 models** return thought signatures when thinking is enabled and the request includes [function calling], specifically [function declarations]."
- Source: GG-s20, GG-s22. Basis: Vendor-documented.

**GG12. Parts that hold signatures must not be merged or joined, and replayed signatures are billed as input.**
- Quote: "Don't concatenate parts with signatures together." and "Don't merge one part with a signature with another part without a signature."
- Quote (function calling page): "Don't merge a `Part` containing a signature with one that does not. This breaks the positional context of the thought." and "Don't combine two `Parts` that both contain signatures, as the signature strings cannot be merged."
- Quote: "Thought signatures increase the input tokens billed when sent back in subsequent requests."
- Source: GG-s22, GG-s33. Basis: Vendor-documented.

**GG13. When built-in tools are combined with custom functions, every returned part has a signature, including server-side tool parts, and all of them must come back.**
- Quote: "You must set the `include_server_side_tool_invocations` flag to `true` to enable tool context circulation."
- Quote: "In a single response, the API returns the `toolCall` and `toolResponse` parts for the built-in tool call."
- Quote: "`thought_signature`: The actual encrypted context embedded in **each part returned by the API**. Context can't be reconstructed without thought signatures; if you don't return the thought signatures for all parts in every turn, the model will error out." and "Found in *all* parts."
- Quote (API reference, `toolCall`): "Server-side tool call. This field is populated when the model predicts a tool invocation that should be executed on the server. The client is expected to echo this message back to the API."
- Source: GG-s34, GG-s30. Basis: Vendor-documented.

**GG14. Image models put a signature on every image part, and the two Google sites describe the failure differently.**
- Quote (AI Studio image generation page): "All responses include a `thought_signature` field." and "Failure to circulate thought signatures may cause the response to fail."
- Quote (same page, example comments): "The first non-thought part always has a signature" and "All image parts have a signature".
- Quote (Vertex): "Gemini 3 Pro Image doesn't return a `400` error if a thought signature isn't returned."
- Source: GG-s46, GG-s47. Basis: Vendor-documented.

**GG15. Vertex states two rules about thoughts across models and turns that the AI Studio page does not state.**
- Quote: "Gemini Flash-Lite models can't read thoughts generated by non-Flash-Lite models. If a request to a Flash-Lite model includes thought signatures generated by a non-Flash-Lite model, the request doesn't fail. Instead, the service drops those thoughts from the model's context."
- Quote: "For Gemini 3.5 Flash and newer models, thoughts from previous turns are preserved by default. The service retains thought history in the conversation context before passing it to the model."
- Quote: "When sending conversation history, be consistent with previous turns: either include the full context (thoughts, function calls, and responses) or omit it entirely. Providing partial context can degrade model performance."
- Source: GG-s47. Basis: Vendor-documented.

**GG16. On the Interactions API a signature lives on a `thought` step or a built-in tool step, never on a function call, and the server can hold it.**
- Quote: "In the Interactions API, thoughts are a first-class representation as dedicated `thought` steps. Because of this signatures are limited exclusively to two known locations, `thought` steps, or built-in tool steps (like `google_search_call`/ `google_search_result`). They never appear on user inputs, model outputs, or standard function calls."
- Quote (field table, `signature`, required): "An encrypted representation of the model's internal reasoning state. Always present, even when the model performs minimal reasoning."
- Quote (stateful mode): "the server automatically manages the conversation state, including all thought blocks and signatures. In this mode, you do not need to do anything regarding signatures. They are handled entirely on the server side."
- Quote (stateless mode): "You **MUST** always resend all `thought` blocks exactly as they were received from the model." and "When switching models within a session, you should still resend the previous model's thought blocks. The backend manages compatibility."
- Source: GG-s23, GG-s21. Basis: Vendor-documented.

### B. Thought signatures - how OpenRouter exposes them

**GG17. OpenRouter's chat completions message has one slot that can hold a Gemini signature: an entry in `reasoning_details` with the format tag `google-gemini-v1`.**
- Quote (reasoning tokens page, `format` values): "`"google-gemini-v1"` - Google Gemini format version 1"
- Quote (same page): "**2. Encrypted Type (`reasoning.encrypted`)** Contains encrypted reasoning data that may be redacted or protected"
- Spec facts (queried from the OpenAPI JSON): `ReasoningDetailEncrypted` has properties `data`, `format`, `id`, `index`, `type` with `type` and `data` required. `ChatToolCall` has properties `function` (`name`, `arguments`), `id`, `type`. A text search of the whole spec returns 0 matches for `extra_content`, 0 for `thought_signature`, and 0 for `thoughtSignature`.
- Source: GG-s2, GG-s5. Basis: OR-documented. Confidence note: the spec and page define the slot. That a Gemini function call signature is what fills it comes from GG19.

**GG18. OpenRouter's documentation states the replay rule in general terms and names thought signatures only once, in passing.**
- Quote (reasoning tokens page): "Use `reasoning_details` when working with models that return special reasoning types (such as encrypted or summarized) - this preserves the full structure needed for those models."
- Quote (same page): "When providing reasoning\_details blocks, the entire sequence of consecutive reasoning blocks must match the outputs generated by the model during the original request; you cannot rearrange or modify the sequence of these blocks."
- Quote (Gemini 4 guide, Anthropic Messages section, the only match for "thought signature" in the 559-page mirror): "continuation blocks look like other `redacted_thinking` blocks, such as thought signatures, and OpenRouter resumes from the last continuation block."
- Source: GG-s2, GG-s1. Basis: OR-documented.

**GG19. OpenRouter's own AI SDK provider shows the wire shape: a Gemini reply with a tool call returns `reasoning_details` entries of type `reasoning.encrypted` with format `google-gemini-v1`, and the whole array goes back on the assistant message.**
- Quote (`e2e/gemini/reasoning-multiturn.test.ts`, lines 4-10): "the reasoning_details (thoughtSignature) are properly preserved in tool calls and can be sent back in multi-turn conversations. The providerOptions containing reasoning_details must be preserved when passing response.messages from a first request to a second request, otherwise Gemini will reject the continuation."
- Quote (same file, lines 99-101): `expect(firstReasoningDetail?.type).toBe('reasoning.encrypted');` `expect(firstReasoningDetail?.data).toBeDefined();` `expect(firstReasoningDetail?.format).toBe('google-gemini-v1');`
- Quote (`src/chat/convert-to-openrouter-chat-messages.ts`, lines 407-417): the outgoing assistant message is built as `role: 'assistant'`, `content`, `tool_calls`, `reasoning: effectiveReasoning`, `reasoning_details: finalReasoningDetails`.
- Quote (`src/chat/index.ts`, lines 467-468): "Only attach reasoning_details to the first tool call to avoid duplicating thinking blocks for parallel tool calls (Claude)"
- Source: GG-s19. Basis: OR-documented (official SDK source). Confidence note: this shows where the signature travels. It does not show how OpenRouter's server pairs each `reasoning.encrypted` entry with a `functionCall` part when it rebuilds the Gemini request.

**GG20. The same provider drops Gemini plain-text reasoning on replay, because sending altered thought text back fails.**
- Quote (`src/chat/convert-to-openrouter-chat-messages.ts`, lines 327-334): "Google Gemini (issue #418): Does NOT use the SDK `signature` field, but Google internally signs thought tokens. When reasoning text is modified during roundtripping (serialization, encoding changes, field reordering), Google rejects with "Corrupted thought signature". Since the SDK cannot verify whether the text is intact, Gemini reasoning.text entries are always stripped on roundtrip" and, from the same sentence, "the encrypted reasoning blob (reasoning.encrypted) handles multi-turn continuity instead."
- Quote (`CHANGELOG.md`, PR #468): "Fix Gemini "Corrupted thought signature" error in multi-turn conversations (issue #418)"
- Source: GG-s19. Basis: OR-documented (official SDK source).

**GG21. Through OpenRouter, a Gemini 3 reply that holds tool calls can report `finish_reason: "stop"`.**
- Quote (`src/chat/index.ts`, lines 549-551): "Fix for Gemini 3 thoughtSignature: when there are tool calls with encrypted reasoning (thoughtSignature), the model returns 'stop' but expects continuation. Override to 'tool-calls' so the SDK knows to continue the conversation."
- Quote (`CHANGELOG.md`, PR #288): "Fix Gemini 3 tool-call conversations stopping prematurely when thoughtSignature (encrypted reasoning) is present. Override finishReason from 'stop' to 'tool-calls' when tool calls and encrypted reasoning are detected."
- Source: GG-s19. Basis: OR-documented (official SDK source). Confidence note: the fix dates from an earlier provider release, so the server may have changed since. Gemini's native `finishReason` is `STOP` for a turn that ends in a function call (the enum in GG61 has no tool-call value), which matches.

**GG22. A Gemini reasoning item printed in OpenRouter's documentation is a signature with a second segment that names the model version and the provider endpoint that issued it.**
- Quote (image understanding page, sample Responses API output for `google/gemini-3-flash-preview`): `"type": "reasoning"`, `"summary": []`, `"encrypted_content": "AY89a1+LwO7p...NxhpqjI/.eyJlbmRwb2ludF9zbHVnIjoiZ29vZ2xlL2dlbWluaS0zLWZsYXNoLXByZXZpZXctMjAyNTEyMTd8Z29vZ2xlLXZlcnRleC9nbG9iYWwifQ"`, `"format": "google-gemini-v1"`
- Decoding the segment after the dot as base64url gives exactly: `{"endpoint_slug":"google/gemini-3-flash-preview-20251217|google-vertex/global"}`. The segment before the dot decodes to 132 bytes of binary data.
- Source: GG-s4. Basis: OR-inferred (from a sample response in OpenRouter's documentation, decoded locally with Python). Confidence note: one sample. It shows OpenRouter wraps the Gemini signature and records the issuing endpoint. What the server does with a wrapped signature whose endpoint differs from the endpoint now serving the request is not shown.

**GG23. For the Gemini 4 continuation token, OpenRouter says in words that a token is bound to the provider that issued it, and that it wraps the token.**
- Quote: "**Continuations stay on one provider.** A continuation token only works on the provider that issued it. OpenRouter routes a continuation request back to that provider and does not fall back to another one. If that provider is unavailable, the request fails instead of restarting the answer elsewhere."
- Quote (Messages section): "The `data` value wraps the Gemini token in an OpenRouter envelope so its format survives the round trip."
- Source: GG-s1. Basis: OR-documented. Confidence note: OpenRouter states this for continuation tokens only. No page states the same rule for ordinary thought signatures.

**GG24. OpenRouter's documentation claims model switching works without code changes and says nothing about a dummy signature.**
- Quote: "The `reasoning_details` functionality works identically across all supported reasoning models. You can easily switch between OpenAI reasoning models (like `~openai/gpt-sol-latest`) and Anthropic reasoning models (like `~anthropic/claude-sonnet-latest`) without changing your code structure."
- A search of the mirror for `skip_thought_signature_validator` and `context_engineering_is_the_way_to_go` returns 0 matches.
- Source: GG-s2. Basis: OR-documented (the quote). The behavior when a history with tool calls from another model reaches Gemini 3 or 4 is listed under open questions.

### C. Thought signatures - fills from other projects (none of these shows what OpenRouter does)

**GG25. Google's own OpenAI-compatible route puts the signature on the tool call, in `extra_content.google.thought_signature`.**
- Quote (Vertex page, Chat Completions example, model response): `"tool_calls": [ { "extra_content": { "google": { "thought_signature": "<SIGNATURE_A>" } }, "function": { "arguments": "{\"flight\":\"AA100\"}", "name": "check_flight" }, "id": "function-call-1", "type": "function" } ]`
- Quote (same page, parallel example): the first entry of `tool_calls` has `extra_content`, and the second entry has only `function`, `id`, and `type`.
- The tool result goes back as `"role": "tool"` with `"name"`, `"tool_call_id"`, and `"content"`.
- Source: GG-s47. Basis: Filled (Google Cloud, first-party flattening of its own API). Also in workspace: `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` ("Gemini's exposes signatures through `extra_content`").

**GG26. The AI Studio documentation points at this but no longer prints it.**
- Quote (OpenAI compatibility page): "Gemini 3 supports OpenAI compatibility for thought signatures in chat completion APIs. You can find the full example on the [thought signatures] page."
- Observation: the "Signatures for OpenAI compatibility" section of the AI Studio thought signatures page, as served on 2026-11-22, shows only native-shaped JSON (`"functionCall"`, `"thoughtSignature"`), and a search of all 112 AI Studio page files downloaded for this work (both variants of each guide plus the API reference) for `extra_content` returns 0 matches.
- Source: GG-s42, GG-s20. Basis: Vendor-documented.

**GG27. LiteLLM hides the signature inside the tool call id, and also keeps a copy in a side field.**
- Quote (`litellm/litellm_core_utils/prompt_templates/factory.py`, line 103): `THOUGHT_SIGNATURE_SEPARATOR: Final = "__thought__"`
- Quote (same file, lines 1262-1271): "Embed thought signature into tool call ID for OpenAI client compatibility." and "Format: call_<uuid>__thought__<base64_signature>"
- Quote (same file, lines 1308-1312): "Extract thought signature from tool call's provider_specific_fields. If not provided try to extract thought signature from tool call id"
- Quote (`litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py`, line 2530): `{**_function_chunk, "provider_specific_fields": {"thought_signature": thought_signature}}`
- Quote (`litellm/utils.py`, lines 1149 and 1166): "Messages with Gemini thought signatures cut from tool call ids, or None when the call's target keeps them." and "Removing thought signatures from tool call IDs for non-Gemini model %s"
- Source: GG-s48. Basis: Filled (LiteLLM).

**GG28. LiteLLM handles foreign history with Google's dummy value, on the first function call only, and retries with it when Gemini rejects a signature.**
- Quote (`factory.py`, lines 1366-1379): "Generate a dummy thought signature for models that require it. This is used when transferring conversation history from older models (like gemini-2.5-flash) to gemini-3, which requires thought_signature for strict validation." followed by `dummy_data: Final = b"skip_thought_signature_validator"` and `return base64.b64encode(dummy_data).decode("utf-8")`
- Quote (same file, lines 1450-1455): "Gemini signs only the first functionCall part of a parallel batch, so scope the placeholder fallback to that part instead of fabricating one per sibling call" followed by `if not thought_signature and is_first_function_call and needs_dummy_signature:`
- Quote (`litellm/llms/vertex_ai/gemini/rejected_thought_signature_retry.py`, lines 114-115): "Gemini rejected a replayed thought signature, resending with the skip_thought_signature_validator placeholder in place of the replayed signatures"
- Source: GG-s48. Basis: Filled (LiteLLM). Note: LiteLLM sends the base64 encoding of the dummy string, while Google's page shows the plain string.

**GG29. LiteLLM states that a signature is only valid on the Google backend that issued it, and quotes the two error strings.**
- Quote (`rejected_thought_signature_retry.py`, lines 22-26): "A signature only validates on the backend that minted it. AI Studio answers a replayed Vertex signature with ``400 Corrupted thought signature.`` and Vertex answers an AI Studio one with ``400 Invalid thought signature.``"
- Source: GG-s48. Basis: Filled (LiteLLM). Confidence note: a code comment by a third party, not a Google statement. Neither Google page read today says whether signatures move between AI Studio and Vertex. Also in workspace: `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` ("same endpoint only; AI Studio to Vertex is HARD").

**GG30. Vercel AI SDK keeps the signature per part in provider metadata, injects the dummy value for Gemini 3, and has a changelog entry that treats a Vertex signature as usable on AI Studio.**
- Quote (`packages/google/src/convert-to-google-messages.ts`, lines 30-37): "Sentinel value Google documents for replaying functionCall parts whose original thoughtSignature is not available to the client." and `export const SKIP_THOUGHT_SIGNATURE_VALIDATOR = 'skip_thought_signature_validator';`
- Quote (same file, lines 594-606): "Gemini 3 returns a single signature for a parallel function-call response on the first standard function call. Subsequent standard function calls in the same model response legitimately have no signature." followed by `thoughtSignature ?? (isGemini3Model && !shouldSkipMissingSignatureMitigation ? injectSkipSignature(part.toolName) : undefined)`
- Quote (same file, lines 800-808, the warning text): "Injected the documented `skip_thought_signature_validator` sentinel to keep the request from failing with HTTP 400. The likely cause is application code that drops `providerOptions.google.thoughtSignature` when persisting or serializing assistant tool-call messages."
- Quote (`packages/openai-compatible/src/chat/convert-to-openai-compatible-chat-messages.ts`, lines 291-299): "Include extra_content for Google Gemini thought signatures" followed by `extra_content: { google: { thought_signature: String(thoughtSignature) } }`
- Quote (`packages/google/CHANGELOG.md`, entry 7ba09a4): "Fix gateway failover losing thoughtSignature when failing over from Vertex to Google AI Studio. The Google provider now falls back to checking the vertex namespace for thoughtSignature, matching the existing Vertex-to-Google fallback behavior."
- Source: GG-s49. Basis: Filled (Vercel AI SDK).

**GG31. Gemini CLI, Google's reference client, uses the same dummy value and lists the history rules it enforces before each request.**
- Quote (`packages/core/src/utils/historyHardening.ts`, line 12): `export const SYNTHETIC_THOUGHT_SIGNATURE = 'skip_thought_signature_validator';`
- Quote (same file, lines 43-48): "Invariants enforced: 1. Role Alternation: user -> model -> user -> model 2. Start Constraint: Must start with a 'user' turn. 3. End Constraint: Must end with a 'user' turn (usually for follow-up prompts). 4. Tool Pairing: Every model functionCall must be followed by a user functionResponse. 5. Signatures: The first functionCall in a model turn must have a thoughtSignature."
- Quote (same file, lines 249-252): "[HistoryHardener] Missing thought signature on first function call in model turn. Injecting synthetic signature."
- Source: GG-s50. Basis: Filled (Gemini CLI, Google).

**GG32. agentgateway uses the same tool-call-id trick as LiteLLM on Vertex, loses the signature of a turn that has no tool call, and sends AI Studio chat traffic to Google's OpenAI-compatible route.**
- Quote (`crates/llm/src/conversion/vertex_gemini.rs`, lines 12-15): "Vertex rejects an `id` on functionCall/functionResponse parts and Gemini 3 hard-400s if a functionCall's signature is not echoed back, but OpenAI clients drop unknown fields while reliably echoing `tool_call_id`. So the signature rides inside the id as `<base-id>__thought__<signature>`, recovered before the outbound Vertex request."
- Quote (same file, lines 1365-1367): "Known limitation: a Gemini-3 `thoughtSignature` on a reasoning-only turn (no tool call to carry it in a tool_call id) is dropped, since the OpenAI shape has no signature slot. Tool-call signatures still round-trip via the tool_call id."
- Quote (`crates/llm/src/gemini.rs`, lines 27-30): `RouteType::Embeddings => "/v1beta/openai/embeddings"`, and the default arm `_ => "/v1beta/openai/chat/completions"`
- Source: GG-s51. Basis: Filled (agentgateway).

### D. Thinking controls

**GG33. `thinkingLevel` is the control for Gemini 4 and Gemini 3, the set of levels differs per model, and Gemini 4 cannot stop thinking.**
- Quote: "The `thinkingLevel` parameter, required for Gemini 4 and recommended for Gemini 3 models, lets you control reasoning behavior. Gemini 4 Carbon and Argon (`gemini-4-carbon` and `gemini-4-argon`) support `low`, `medium` (default), and `high`, and Gemini 4 Neon and Hydrogen (`gemini-4-neon` and `gemini-4-hydrogen`) support `low`, `medium`, `high` (default), and `xhigh`."
- Quote: "You cannot disable thinking for Gemini 4 Carbon, Gemini 4 Argon, Gemini 4 Neon, or Gemini 4 Hydrogen. Gemini 3 Flash and Flash-Lite also do not support full thinking-off, but the `minimal` setting means the model likely will not think (though it still potentially can)."
- Quote (`minimal` row): "Not supported on Gemini 4 models or `gemini-3.8-flash`."
- Table rows on the same page: `gemini-3.8-flash` "low, medium, high"; `gemini-3.6-flash` "minimal, low, medium, high"; `gemini-3.5-flash-lite` default "On (minimal)"; `gemini-3.1-pro-preview` "low, medium, high"; `gemini-3-pro-preview` "low, high".
- Quote (API reference enum): "`THINKING_LEVEL_UNSPECIFIED`", "`MINIMAL`", "`LOW`", "`MEDIUM`", "`HIGH`", "`XHIGH`", with "Recommended for Gemini 3 or later models. Use with earlier models results in an error."
- Source: GG-s22, GG-s30. Basis: Vendor-documented.

**GG34. `thinkingBudget` is the Gemini 2.5 control, and on Gemini 4 it is deprecated and cannot be combined with a level.**
- Quote: "Gemini 2.5 series models don't support `thinkingLevel`; use `thinkingBudget` instead."
- Quote: "You can disable thinking by setting `thinkingBudget` to 0." and "Setting the `thinkingBudget` to -1 turns on **dynamic thinking**"
- Table rows: 2.5 Pro range "`128` to `32768`" with "N/A: Cannot disable thinking"; 2.5 Flash "`0` to `24576`"; 2.5 Flash Lite "Model does not think" by default, range "`512` to `24576`".
- Quote: "In Gemini 4 models (`gemini-4-carbon`, `gemini-4-argon`, `gemini-4-neon`, and `gemini-4-hydrogen`), `thinkingBudget` (`thinking_budget`) is deprecated. If passed alone, it maps to the nearest `thinkingLevel` (`thinking_level`) and returns a warning; sending `thinkingBudget` and `thinkingLevel` together returns a `400` error."
- Source: GG-s22. Basis: Vendor-documented.

**GG35. Thought summaries are opt-in with `includeThoughts`, arrive as parts flagged `thought`, and are billed by the full thoughts behind them.**
- Quote: "You can enable thought summaries by setting `includeThoughts` to `true` in your request configuration. You can then access the summary by iterating through the `response` parameter's `parts`, and checking the `thought` boolean."
- Quote: "thinking levels and budgets apply to the model's raw thoughts and not to thought summaries."
- Quote: "pricing is based on the full thought tokens the model needs to generate to create a summary, despite only the summary being output from the API."
- Source: GG-s22. Basis: Vendor-documented.

**GG36. The Interactions API uses a different vocabulary for the same things.**
- Quote: "You can control this behavior using the `thinking_level` parameter."
- Quote: "A thought block may contain **only a signature with no summary** in these cases:" followed by three bullets, one of which reads "`thinking_summaries: "none"`, where summaries are explicitly disabled". The request examples on the page set `"thinking_summaries": "auto"`.
- Quote (streaming delta table): "`thought_summary` | Text or image summary content" and "`thought_signature` | The cryptographic signature | the last delta before `step.stop`"
- Quote: "You can get the total number of generated thinking tokens from the `total_thought_tokens` field."
- The Interactions level table also lists `gemini-2.5-pro`, `gemini-2.5-flash`, and `gemini-2.5-flash-lite` with levels "low, medium, high", where the `generateContent` page says 2.5 models take `thinkingBudget` only.
- Source: GG-s23. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (same vocabulary change, read 2026-10-28).

**GG37. The output limit includes thought tokens and cuts off without changing how much the model thinks.**
- Quote: "The [`max_output_tokens`] generation parameter sets the maximum number of tokens a response can generate, including thought tokens."
- Quote: "If the model hits this limit while reasoning, it stops generating with `finish_reason: MAX_TOKENS` and returns truncated or empty output (while still billing for any thinking tokens generated)."
- Source: GG-s22. Basis: Vendor-documented.

**GG38. OpenRouter translates its neutral `reasoning` object to Gemini's two controls.**
- Quote: "OpenRouter maps the `reasoning.effort` parameter directly to Google's `thinkingLevel` values" with table rows `"minimal"` to `"minimal"`, `"low"` to `"low"`, `"medium"` to `"medium"`, `"high"` to `"high"`, and "`"xhigh"` | `"high"` (mapped down)".
- Quote: "If a model doesn't support a specific effort level (for example, if a model only supports `low` and `high`), OpenRouter will map your requested effort to the nearest supported level."
- Quote: "If you specify `reasoning.max_tokens` explicitly, OpenRouter will pass it through as `thinkingBudget` to Google's API. However, for Gemini 3 models, Google internally maps this budget value to a `thinkingLevel`, so you will not get precise token control."
- Quote (Gemini 4 guide): "Gemini 4 Argon always reasons. `reasoning.effort` accepts `low`, `medium`, and `high`. `reasoning.max_tokens` is not supported, and reasoning cannot be turned off."
- Quote (reasoning page, on the catalog flag): "**`mandatory`**: When `true`, hide disable controls and do not send `effort: "none"`" and, from the same bullet, "the model rejects it."
- Source: GG-s2, GG-s1. Basis: OR-documented. Note: the "mapped down" row was written for Gemini 3. The catalog (GG39) lists `xhigh` as supported on `google/gemini-4-neon` and `google/gemini-4-hydrogen`.

**GG39. OpenRouter's public catalog publishes, per Google model, which efforts exist and whether thinking is mandatory.**
- Catalog values read from `models.json`: `google/gemini-4-neon` and `google/gemini-4-hydrogen` `{"mandatory": true, "supported_efforts": ["xhigh", "high", "medium", "low"], "default_effort": "medium"}`; `google/gemini-4-carbon` and `google/gemini-4-argon` `{"mandatory": true, "supported_efforts": ["high", "medium", "low"], "default_effort": "medium"}`; `google/gemini-3.6-flash` `{"mandatory": true, "default_enabled": true, "supported_efforts": ["high", "medium", "low", "minimal"], "default_effort": "medium"}`; `google/gemini-3.1-flash-lite` `{"mandatory": false, "default_enabled": true, "supported_efforts": ["high", "medium", "low", "minimal", "none"], "default_effort": "minimal"}`; `google/gemini-2.5-pro` `{"mandatory": true}`; `google/gemini-2.5-flash` `{"mandatory": false}`.
- Source: GG-s6. Basis: OR-documented. Note: Google's page gives `high` as the default level for Neon and Hydrogen (GG33), while the catalog gives `default_effort: "medium"` for both.

**GG40. OpenRouter exposes thought summaries through its ordinary reasoning fields, with one switch.**
- Quote: "Reasoning tokens are included in the response by default if the model decides to output them." and "On Chat Completions they appear in the `reasoning` field of each message, and `reasoning.exclude` can omit them."
- Quote (Gemini 4 guide): "**`reasoning.exclude` keeps the token.** With `reasoning: { "exclude": true }`, reasoning text is hidden but the continuation item is still returned"
- Source: GG-s2, GG-s1. Basis: OR-documented. Confidence note: no OpenRouter page says whether it sets `includeThoughts: true` upstream by default for Gemini. The sample outputs on the reasoning page show Gemini summary text returned without the caller asking for it.

### E. Long decode and the continuation token (Gemini 4)

**GG41. Google's native rule: one hop is capped, a hop that stops with budget left returns a token, and the budget is one total for all hops.**
- Quote: "**A cap on each hop:** One hop returns up to 65,536 output tokens (or up to 32,768 on direct non-streaming REST calls to `gemini-4-carbon` and `gemini-4-argon`)."
- Quote: "**A continuation signal:** If the model reaches the cap and has tokens left, the response ends with `finish_reason: CONTINUATION` and a `continuation_token`."
- Quote: "**One total for all hops:** `max_output_tokens` sets the total thinking and output tokens for all hops together." and "**A stop signal:** When the model uses all of `max_output_tokens`, the response ends with `finish_reason: MAX_TOKENS`."
- Quote: "The encrypted `continuation_token` carries the model's state between requests, so nothing is stored on the server."
- Quote (latest models page, each of the three Gemini 4 cards): "*Max output* **64K default / 1M [long decode]**"
- Source: GG-s24, GG-s55. Basis: Vendor-documented.

**GG42. Google's native loop: the token is a top-level request field next to an unchanged request, and partial output stays out of `contents`.**
- Quote: "**Send the same request on every hop.** Send the original `contents`, `system_instruction`, `tools`, and `generation_config`. Keep `max_output_tokens` the same." / "**Do not add partial output to `contents`.** The `continuation_token` carries the model's state, including its thinking." / "**Send only the latest token.** Each token contains all earlier progress." / "**Continue while `finish_reason` is `CONTINUATION`.** Stop on any other finish reason." / "**Join the parts after the last hop.** Combine the parts from all hops into one model turn for the conversation history."
- Quote (REST example, hop 2): `"continuationToken": "TOKEN_FROM_HOP_1"` as a sibling of `"contents"` and `"generationConfig"`; the hop response holds `"finishReason": "CONTINUATION"` and `"continuationToken": "CiQ4ZmQ3..."` inside the candidate.
- Quote (API reference, request field): "Note that `contents` is mandatory and must identically replicate the prompt and conversation history from the initial request when continuation token is provided in a request."
- Source: GG-s24, GG-s30. Basis: Vendor-documented.

**GG43. A spent budget can be resumed only with a larger one.**
- Quote: "When a response ends with `MAX_TOKENS`, it still includes a `continuation_token`." and "**Continue:** Send the same request with the token and a higher `max_output_tokens`. The value must be above the tokens used so far. Otherwise, the API returns a 400 error."
- Quote (troubleshooting table): "400: "Continuation token has already used output tokens..." | `max_output_tokens` is not above the tokens used so far. Raise it."
- Quote (`finishReason` enum, `CONTINUATION`): "Note, the `max_output_tokens` is cumulative across continuation hops and must be monotonically increased or omitted."
- Source: GG-s24, GG-s30. Basis: Vendor-documented.

**GG44. Streams save checkpoints, and a dropped stream resumes from the last one.**
- Quote: "After the first 32,768 tokens on `gemini-4-carbon` and `gemini-4-argon` (or after 65,536 tokens on `gemini-4-neon` and `gemini-4-hydrogen`, starting on the second hop), the stream sends a checkpoint token every 5,000 tokens. Keep new chunks in a pending list. When a chunk has a `continuation_token`, move the pending chunks to your committed output and save the token. If the stream drops, discard the pending chunks and resume from the last token."
- Source: GG-s24. Basis: Vendor-documented.

**GG45. Usage across hops overstates the prompt, and Google addresses gateways directly.**
- Quote: "`prompt_token_count` and `total_token_count` are the sum of every hop, not the size of your context. Don't use them to decide when to summarize or trim your conversation history (context compaction), or to enforce a context limit."
- Quote: "**Proxies and gateways that call REST:** Each REST response reports only its own hop. Add each field across hops for cost, and keep the first hop's `promptTokenCount` as the prompt size. Otherwise, your users see different numbers than SDK users see for the same request."
- Quote: "The SDK calculates `initial_prompt_token_count` on the client. It isn't in the REST response"
- Quote: "**Cache discount:** Implicit caching discounts the repeated input in each hop by 95%." and "**Cache expiry:** The cache expires after 15 to 20 minutes without use."
- Source: GG-s24. Basis: Vendor-documented.

**GG46. Where long decode exists and where it does not.**
- Quote (`generateContent` page, limitations): "**OpenAI compatibility endpoint:** Long decode is not available. Output stops at the per-request cap."
- Quote (Interactions page): "On the Interactions API, you can run long decode with [background execution] so our servers run every hop for you, or in the foreground with [client-side continuation]"
- Quote (Interactions status table): "`requires_continuation` | A foreground hop reached its single-request cap" and "In REST, send the next hop with `previous_interaction_id` and `continuation_token`."
- Quote (Interactions troubleshooting): "400: "store=true is required for background interactions""
- Quote (timeouts): "**Non-streaming calls:** The server closes the connection after 10 minutes." and "**Streaming calls:** The server closes the stream after 20 minutes."
- Source: GG-s24, GG-s25. Basis: Vendor-documented.

**GG47. OpenRouter's field: the token is an encrypted reasoning item with a fixed id, returned on a length stop.**
- Quote: "On OpenRouter, the continuation token is an encrypted reasoning item. Clients that already pass reasoning back between turns get long decode with no new fields."
- Quote (response shape): `"finish_reason": "length"`, `"native_finish_reason": "CONTINUATION"`, and in `message.reasoning_details` an entry `{"type": "reasoning.encrypted", "id": "gemini-continuation", "format": "google-gemini-v1", "data": "<opaque continuation token>", "index": 0}`
- Quote: "If the model hits the limit, the response ends with `finish_reason: "length"` and `native_finish_reason: "MAX_TOKENS"` and returns a continuation token."
- Quote: "When `max_tokens` is omitted, OpenRouter sends its default output budget of 65,536 tokens, the same as Argon's per-request limit."
- Source: GG-s1. Basis: OR-documented.

**GG48. OpenRouter's loop differs from Google's on the surface: the caller appends the stopped assistant turn, and OpenRouter removes it again before the upstream call.**
- Quote: "Resend the same conversation and parameters with the stopped turn appended as the last assistant turn. Set `max_tokens` above the total output of every hop so far, or the request returns a 400 error after a `MAX_TOKENS` hop."
- Quote: "Only a continuation item in the last assistant turn resumes the decode. Each hop replaces the previous continuation turn, and OpenRouter removes continuation items from earlier turns, so a finished long answer in your history does not affect later requests. Visible text from the stopped turn does not need to be resent: OpenRouter drops the trailing continuation turn before forwarding the request, because the token already carries its state. Treat the token as opaque and pass it back byte for byte."
- Quote: "Do not add a new user message in the middle of a continuation."
- Quote: "A hop that finishes normally can still carry checkpoint items, so decide whether to continue from the finish reason, not from whether an item is present."
- Source: GG-s1. Basis: OR-documented.

**GG49. OpenRouter's stated limits for the token.**
- Quote: "**Token size grows with the decode** and can reach several MB for a long answer. Your client must accept and resend bodies of that size on later hops."
- Quote: "Explicit caching with `cache_control` is skipped on continuation hops."
- Quote: "Every hop is its own generation with its own usage and cost." and "for context-window or compaction checks, use the first hop's `prompt_tokens`"
- Quote: "A dropped stream reports no `usage` or finish reason, so you cannot tell whether the dropped hop had already spent its budget. If the resume returns a 400 error saying `max_output_tokens` has been exhausted, raise `max_tokens` and resend."
- Quote (breaking changes): "A client that does not resume gets a partial answer, as with any length stop."
- Source: GG-s1. Basis: OR-documented.

**GG50. The same token on OpenRouter's other two routes, and the catalog's output limit.**
- Quote (Responses): "returns `status: "incomplete"` with `incomplete_details.reason: "max_output_tokens"` and a reasoning output item that carries the token in `encrypted_content`" with `"id": "gemini-continuation"`.
- Quote (Responses): "OpenRouter does not store the response, so `previous_response_id` cannot be used to continue."
- Quote (Messages): "returns `stop_reason: "max_tokens"` and a `redacted_thinking` content block that carries the token."
- Catalog values: `google/gemini-4-argon`, `-carbon`, `-neon`, `-hydrogen` report `context_length` 1048576 and `top_provider.max_completion_tokens` 943718.
- Source: GG-s1, GG-s6. Basis: OR-documented.

### F. Caching

**GG51. Implicit caching is on by default from Gemini 2.5, with a per-model minimum and a special rule for Gemini 4 agent loops.**
- Quote: "Implicit caching is enabled by default for all Gemini 2.5 and newer models. We automatically pass on cost savings if your request hits caches. You don't need to configure anything to turn it on."
- Minimum token table on the page: Gemini 4 Carbon, Argon, Neon, Hydrogen "4,096"; Gemini 3.8 Flash, 3.6 Flash, 3.1 Pro Preview "4,096"; Gemini 2.5 Flash "2,048"; Gemini 2.5 Pro "2,048".
- Quote: "**Multi-turn and agent loops on Gemini 4:** Gemini caches the full previous request only if the new request adds at least 2,050 tokens. Model output, thought, and function results all count toward this total. For smaller additions, Gemini re-processes the last 2,050 tokens of the previous request at the standard input rate."
- Quote: "Implicit caching (automatically enabled on Gemini 2.5 and newer models, no cost saving guarantee)"
- Source: GG-s26. Basis: Vendor-documented.

**GG52. Named caches (`cachedContents`) are created ahead of time, referenced by name, and almost wholly immutable.**
- Quote: "If not set, the TTL defaults to 1 hour." and "There are no minimum or maximum bounds on the TTL."
- Quote: "You can set a new `ttl` or `expire_time` for a cache. Changing anything else about the cache isn't supported."
- Quote: "It's not possible to retrieve or view cached content, but you can retrieve cache metadata"
- Quote (API reference): `model` "Immutable. The name of the `Model` to use for cached content"; `systemInstruction` "Optional. Input only. Immutable. Developer set system instruction. Currently text only."; `contents` "Optional. Input only. Immutable. The content to cache."; `tools` "Optional. Input only. Immutable."; `toolConfig` "Optional. Input only. Immutable. Tool config. This config is shared for all tools."
- Quote (request field): "**cachedContent**: `string` Optional. The name of the content [cached] to use as context to serve the prediction. Format: `cachedContents/{cachedContent}`"
- Quote: "Cached content is a prefix to the prompt." and "Implicit caching doesn't apply when a request uses an explicit cache. Any additional input tokens sent alongside the explicit cache are billed at the standard input rate."
- Quote: "The *minimum* input token count for context caching varies by model."
- Source: GG-s26, GG-s28, GG-s30. Basis: Vendor-documented.

**GG53. Named caches are unavailable on Gemini 4 and on the Interactions API.**
- Quote (Interactions variant of the caching page): "The **Interactions API** only supports implicit caching. Explicit caching (manually creating and managing cache objects) is not supported in the Interactions API, or for Gemini 4 models."
- Quote (`generateContent` variant): "Explicit caching (can be manually enabled on Gemini 3 and 2.5 models, cost saving guarantee; not supported for Gemini 4 models)"
- Quote (Gemini 4 Carbon migration list): "**Move off explicit context caching:** Explicit caching (`cachedContents` / `caches.create`) is not supported on Gemini 4 Carbon and returns an error. Remove calls that create or pass `cached_content` in `GenerateContentConfig`. [Implicit caching] is enabled by default for prompts of at least 4,096 tokens and discounts cached input tokens by 95% on cache hits."
- Source: GG-s26, GG-s44. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` ("absent on the Interactions API and on Gemini 4 Argon+").

**GG54. Storage is billed by the hour for named caches, and Gemini 4 price rows list implicit caching only.**
- Quote (pricing page, the Flash model row described as "Our most intelligent Flash model"): "Context caching price ... $0.075 through December 31, 2026. $0.15 starting January 1, 2027. $0.50 / 1,000,000 tokens per hour (storage price) through December 31, 2026. $1.00 / 1,000,000 tokens per hour (storage price) starting January 1, 2027." (the ellipsis stands for the "Free of charge" free-tier cell)
- Quote (Gemini 4 Carbon row): "Context caching price ... $0.10 through November 30, 2026. $0.20 starting December 1, 2026. Implicit caching only (explicit caching is not supported)."
- Source: GG-s27. Basis: Vendor-documented.

**GG55. OpenRouter describes Gemini implicit caching with figures that differ from Google's page today.**
- Quote: "Gemini 2.5 series models and newer support **implicit caching**, providing automatic caching functionality similar to OpenAI’s automatic caching. Implicit caching works seamlessly" 
- Quote: "No cache write or storage costs." and "Cached tokens are charged at 0.25x the original input token cost."
- Quote: "Note that the TTL is on average 3-5 minutes, but will vary. Requests must also meet a minimum prompt size to be eligible for caching, which varies by model: 1024 tokens for Gemini 2.5 Flash and 4096 tokens for Gemini 2.5 Pro."
- Source: GG-s3. Basis: OR-documented. See Conflicts for the differences (2,048 minimum, 95 percent discount on Gemini 4).

**GG56. For explicit caching OpenRouter holds the cache object itself, and the caller marks content with Anthropic-style `cache_control`.**
- Quote: "OpenRouter simplifies Gemini cache management, abstracting away complexities: You **do not** need to manually create, update, or delete caches. You **do not** need to manage cache names or TTL explicitly."
- Quote: "Gemini caching in OpenRouter requires you to insert `cache_control` breakpoints explicitly within message content, similar to Anthropic."
- Quote: "OpenRouter will use only the last breakpoint for Gemini caching across normal message content. Including multiple breakpoints is safe and can help maintain compatibility with Anthropic, but only the final one will be used for Gemini."
- Quote: "Cache Writes have a 5 minute Time-to-Live (TTL) that does not update. After 5 minutes, the cache expires and a new cache must be written."
- Quote: "Cache write cost = Input token price + (Cache storage price × (5 minutes / 60 minutes))" and "**Cache Reads:** Charged at 0.25× the original input token cost."
- Quote: "Gemini has a single `systemInstruction` field, and cached Gemini content treats that `systemInstruction` as immutable. On OpenRouter, this means `cache_control` inside the first `system` or `developer` message can cache the normalized system prompt, but it cannot preserve an uncached dynamic tail inside that same message."
- Source: GG-s3. Basis: OR-documented.

**GG57. Cache markers translate across vendors with the TTL dropped, and OpenRouter keeps a conversation on one provider to keep caches warm (confirms LC7 for Google).**
- Quote: "a block marked with `prompt_cache_breakpoint` gets a default (5-minute) `cache_control` when routed to Anthropic or Google. TTLs are not translated" and, from the same note, "a `prompt_cache_breakpoint` `ttl` is dropped toward Anthropic or Google"
- Quote: "This works automatically with both implicit caching (e.g. OpenAI, DeepSeek, Gemini 2.5) and explicit caching (e.g. Anthropic `cache_control` breakpoints)."
- Quote: "If the sticky provider becomes unavailable, OpenRouter automatically falls back to the next-best provider." and "Sticky sessions expire after **60 minutes** of inactivity."
- Quote: "Sticky routing is not used when you specify a manual [provider order] via `provider.order`"
- Source: GG-s3. Basis: OR-documented.

**GG58. OpenRouter's catalog prices cache writes on Gemini 4 models, and its endpoint API reports implicit caching as false for every Gemini endpoint checked.**
- Catalog values: `google/gemini-4-argon` pricing has `"input_cache_read": "0.0000001"` and `"input_cache_write": "0.0000001875"` against `"prompt": "0.000002"`. `google/gemini-4-hydrogen` has `"input_cache_read": "0.000000075"` and `"input_cache_write": "0.0000000833333333333333"` against `"prompt": "0.0000015"`.
- Endpoint API values for `google/gemini-4-argon`, `google/gemini-4-hydrogen`, `google/gemini-3.1-pro-preview`, `google/gemini-3.5-flash`, `google/gemini-2.5-flash`: every endpoint object has `"supports_implicit_caching": false`.
- Source: GG-s6, GG-s7. Basis: OR-documented. See Conflicts.

### G. Safety settings, blocked prompts, and finish reasons

**GG59. `safetySettings` is a per-request list of category and threshold pairs, and the default threshold is Off.**
- Quote (REST example): `"safetySettings": [ {"category": "HARM_CATEGORY_HATE_SPEECH", "threshold": "BLOCK_LOW_AND_ABOVE"} ]`
- Thresholds on the page: `OFF`, `BLOCK_NONE`, `BLOCK_ONLY_HIGH`, `BLOCK_MEDIUM_AND_ABOVE`, `BLOCK_LOW_AND_ABOVE`, `HARM_BLOCK_THRESHOLD_UNSPECIFIED`.
- Quote: "If the threshold is not set, the default block threshold is **Off** for Gemini 2.5 and later models."
- Quote: "In addition to the adjustable safety filters, the Gemini API has built-in protections against core harms, such as content that endangers child safety. These types of harm are always blocked and cannot be adjusted."
- Quote (API reference): "Harm categories HARM_CATEGORY_HATE_SPEECH, HARM_CATEGORY_SEXUALLY_EXPLICIT, HARM_CATEGORY_DANGEROUS_CONTENT, HARM_CATEGORY_HARASSMENT, HARM_CATEGORY_CIVIC_INTEGRITY, HARM_CATEGORY_JAILBREAK are supported."
- Source: GG-s29, GG-s30. Basis: Vendor-documented.

**GG60. A blocked prompt returns HTTP 200 with zero candidates and a `promptFeedback.blockReason`.**
- Quote: "Returns no candidates at all only if there was something wrong with the prompt (check `prompt_feedback`)"
- Quote: "**blockReason**: `string enum` Optional. If set, the prompt was blocked and no candidates are returned. Rephrase the prompt."
- Enum values: `BLOCK_REASON_UNSPECIFIED`, `SAFETY`, `OTHER`, `BLOCKLIST`, `PROHIBITED_CONTENT`, `IMAGE_SAFETY`.
- Quote (safety page): "If response content was blocked and the `finishReason` was `SAFETY`, you can inspect `safetyRatings` for more details. The content that was blocked is not returned."
- Source: GG-s30, GG-s29. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`.

**GG61. The `finishReason` enum on `generateContent`, in full.**
- Values: `FINISH_REASON_UNSPECIFIED`, `STOP`, `MAX_TOKENS`, `SAFETY`, `RECITATION`, `LANGUAGE`, `OTHER`, `BLOCKLIST`, `PROHIBITED_CONTENT`, `SPII`, `MALFORMED_FUNCTION_CALL`, `IMAGE_SAFETY`, `IMAGE_PROHIBITED_CONTENT`, `IMAGE_OTHER`, `NO_IMAGE`, `IMAGE_RECITATION`, `UNEXPECTED_TOOL_CALL`, `TOO_MANY_TOOL_CALLS`, `MISSING_THOUGHT_SIGNATURE`, `MALFORMED_RESPONSE`, `ESCALATION`, `PUP_LIMITED_DISABLED`, `CONTINUATION`.
- Quotes: "`STOP`: Natural stop point of the model or provided stop sequence." / "`RECITATION`: The response candidate content was flagged for recitation reasons." / "`PROHIBITED_CONTENT`: Token generation stopped for potentially containing prohibited content." / "`MALFORMED_FUNCTION_CALL`: The function call generated by the model is invalid." / "`UNEXPECTED_TOOL_CALL`: Model generated a tool call but no tools were enabled in the request."
- The enum has no value for "stopped to call a function": a function call ends with `STOP`.
- Source: GG-s30. Basis: Vendor-documented.

**GG62. The Interactions API reports blocks and malformed output as snake_case error codes.**
- "Generation blocked codes": `safety`, `recitation`, `language`, `prohibited_content`, `spii`, `blocklist`, `image_safety`, `image_prohibited_content`, `image_recitation`, `image_other`, `content_blocked`.
- "Generation error codes": `malformed_function_call`, `malformed_tool_call`, `unexpected_tool_call`, `no_image`, `too_many_tool_calls`, `missing_thought_signature`.
- Quote: "All errors from the Interactions API return an `error` object containing a `code` and `message`." and, for streams, "the API sends error events over the Server-Sent Events (SSE) stream with `event_type` set to `"error"`."
- Source: GG-s31. Basis: Vendor-documented.

**GG63. Google documents a known cause of `MALFORMED_FUNCTION_CALL` and a prompt-level fix.**
- Quote: "**Issue:** If your prompt requires the model to output structured text (XML, YAML, JSON, etc.) (e.g., `<UPDATE>...</UPDATE>`) immediately before making a tool call, the tool call may occasionally fail with `Malformed_Function_Call`."
- Quote: "**PREFERRED:** Instruct the model to put its pre-tool notes inside a dedicated `update()` function call instead of raw text"
- Source: GG-s33. Basis: Vendor-documented.

**GG64. OpenRouter turns a Gemini safety block into a typed 403 error and keeps Google's payload in metadata.**
- Quote: "`content_policy_violation` | 403 | The input or output was flagged by a content filter that runs separately from the model (provider- or OpenRouter-level), such as Gemini's `SAFETY` block."
- Quote: "`metadata.provider_name`, `metadata.provider_code`, and `metadata.raw` identify who declined and carry the provider's verbatim payload. The status is the same for every provider and skin, whether the block is the HTTP response or arrives mid-stream (in which case it is the `code` inside the error body, since the HTTP line was already committed)."
- Quote (API overview): "OpenRouter normalizes each model's `finish_reason` to one of the following values: `tool_calls`, `stop`, `length`, `content_filter`, `error`." and "The raw finish\_reason string returned by the model is available via the `native_finish_reason` property."
- Source: GG-s8, GG-s18. Basis: OR-documented (confirms LC10 for Gemini: `native_finish_reason` holds values such as `"CONTINUATION"` and `"MAX_TOKENS"`, GG47). Confidence note: no OpenRouter page lists how each Gemini `finishReason` or `blockReason` maps to the five values, or whether `RECITATION` and `PROHIBITED_CONTENT` become the 403 error or a `content_filter` finish.

**GG65. OpenRouter's request schema has a provider-keyed passthrough object, and its documentation does not say which Google keys it forwards for chat models.**
- Quote (spec, `ProviderOptions`, referenced from `ProviderPreferences.options`): "Provider-specific options keyed by provider slug. The provider segment is case-insensitive; an endpoint tag after `/` must match the endpoint slug exactly. Only options for the matched provider are forwarded; the rest are ignored. Unrecognized keys are silently dropped." Its properties include `google-ai-studio` and `google-vertex`, each `additionalProperties: {}`.
- Quote (provider integration page, an example of what a provider can declare): `"passthrough_parameters": { "service_tier": { "type": "enum", "values": ["standard", "priority"] }, "safety_settings": { "type": "object" } }` and "OpenRouter merges the parameter names from every scope into the endpoint's passthrough allowlist, which controls the `provider.options` keys forwarded to your API."
- A text search of the spec returns 0 matches for `safety_settings` and `safetySettings`. The catalog and the endpoint objects for the five Gemini models checked hold no passthrough allowlist field.
- Source: GG-s5, GG-s15, GG-s6, GG-s7. Basis: OR-documented (the mechanism). Whether `safetySettings` reaches Google is listed under open questions.

**GG66. In OpenRouter's batch documentation, a Vertex line blocked for safety or recitation is not billed.**
- Quote: "A line that Vertex blocked, at the prompt or with a finish reason such as `SAFETY` or `RECITATION`, is not billed either, even when it reports token counts."
- Source: GG-s14. Basis: OR-documented.

**GG67. Two open-source translators show a way to flatten Gemini's stop and block signals.**
- LiteLLM map (`litellm/litellm_core_utils/core_helpers.py`, lines 534-555): `"STOP": "stop"`, `"MAX_TOKENS": "length"`, `"CONTINUATION": "length"`, `"SAFETY": "content_filter"`, `"RECITATION": "content_filter"`, `"MALFORMED_FUNCTION_CALL": "stop"`, `"OTHER": "content_filter"`, `"PROHIBITED_CONTENT": "content_filter"`, `"SPII": "content_filter"`, `"UNEXPECTED_TOOL_CALL": "stop"`, `"MISSING_THOUGHT_SIGNATURE": "stop"`.
- agentgateway (`crates/llm/src/conversion/vertex_gemini.rs`, lines 1275-1295): when `resp.candidates.is_empty()` and `prompt_feedback.block_reason` is set, it builds one choice with `content: Some(String::new())` and `FinishReason::ContentFilter`. Lines 1399-1405: a mapped `Stop` becomes `FinishReason::ToolCalls` when the reply held a tool call.
- Source: GG-s48, GG-s51. Basis: Filled (LiteLLM, agentgateway).

### H. Grounding, URL context, Maps, code execution, and tool combination

**GG68. Google Search grounding is a server-run tool that returns structured `groundingMetadata` with a display obligation.**
- Quote (REST): `"tools": [ { "google_search": {} } ]`
- Quote: "`webSearchQueries` : Array of the search queries used." / "`searchEntryPoint` : Contains the HTML and CSS to render the required Search Suggestions. Full usage requirements are detailed in the [Terms of Service]." / "`groundingChunks` : Array of objects containing the web sources (`uri` and `title`)." / "`groundingSupports` : Array of chunks to connect model response `text` to the sources in `groundingChunks`. Each chunk links a text `segment` (defined by `startIndex` and `endIndex`) to one or more `groundingChunkIndices`."
- Quote: "When you use Grounding with Google Search with Gemini 3 and newer models, your project is billed for **each** search query that the model decides to execute." and "then costs **$14 per 1,000 search requests**."
- Quote: "Older models use a `google_search_retrieval` tool. For all current models, use the `google_search` tool"
- Source: GG-s35. Basis: Vendor-documented.

**GG69. URL context and Maps grounding are two more server-run tools with their own metadata.**
- Quote (URL context): "When the model uses the URL context tool, the response includes a `url_context_metadata` object." / "Request limit: The tool can process up to 20 URLs per request." / "URL content size: The maximum size for content retrieved from a single URL is 34MB." / "The content retrieved from the URLs you specify in your prompt is counted as part of the input tokens." / "you will get an `url_retrieval_status` of `URL_RETRIEVAL_STATUS_UNSAFE`."
- Quote (Maps, REST): `"tools": [{"googleMaps": {}}]`
- Quote (Maps): "The Google Maps sources must immediately follow the generated content that" (sentence continues on the page) and "The Google Maps sources must be viewable within one user interaction."
- Maps tool response data listed on the tool combination page: `places` and `google_maps_widget_context_token`. Maps pricing line: "($25 per 1,000 grounded prompts)".
- Source: GG-s36, GG-s37, GG-s34. Basis: Vendor-documented.

**GG70. Code execution runs Python on Google's side and returns two part types of its own.**
- Quote: "`executableCode`: Code generated by the model that is meant to be executed" and "`codeExecutionResult`: Result of the executable code"
- Quote (chat example): `"executable_code": { "id": "a1b2c3d4", ... }, "thought_signature": "..."` then `"code_execution_result": { "id": "a1b2c3d4", "outcome": "OUTCOME_OK", ... }`
- Quote: "Gemini is only able to execute code in Python." / "The maximum runtime of the code environment is 30 seconds." / "You can't install your own libraries."
- Quote: "You must pass back the `id` and `thought_signature` fields for tool combination to work."
- Source: GG-s38. Basis: Vendor-documented.

**GG71. OpenRouter exposes Google Search through its own web search tool and returns citations in OpenAI's annotation shape.**
- Quote: "When `engine` is `"auto"` (the default) or `"native"`, OpenRouter uses the provider's built-in search for supported models." with the Google line "Gemini 3 Flash, Gemini 3 Pro, Gemini 3.1 Flash/Lite, Gemini 3.5 Flash".
- Quote (spec description of the native tool type): "The provider tool type the request is translated to when this tool runs natively, e.g. `web_search_20260209` on Anthropic or `google_search` on Gemini."
- Endpoint API values: the endpoints of `google/gemini-4-argon`, `google/gemini-4-hydrogen`, `google/gemini-3.1-pro-preview`, and `google/gemini-3.5-flash` report `"native_tools": {"openrouter:web_search": {"type": "google_search"}}`. The three endpoints of `google/gemini-2.5-flash` report `"native_tools": {}`.
- Quote (plugin page): "Web search results for all models (including native-only models like Perplexity and OpenAI Online) are available in the API and standardized by OpenRouter to follow the same annotation schema in the [OpenAI Chat Completion Message type]" with the shape `"annotations": [ { "type": "url_citation", "url_citation": { ...`
- Quote (domain filter table): "**Native (Google)** | No | No | Not supported. With `engine: "auto"`, falls back to Exa when filters are set. With `engine: "native"`, returns a 400 error"
- Quote: "`user_location` ... Currently only supported by native provider search" (parameter table row, shortened) and "With native provider search, forwarded only to Anthropic (as `max_uses`); other native search providers ignore it"
- Catalog value: every Google chat model lists `"web_search": "0.014"` in `pricing`, which equals Google's $14 per 1,000 queries.
- Source: GG-s9, GG-s10, GG-s5, GG-s7, GG-s6. Basis: OR-documented.

**GG72. OpenRouter's documentation shows one Gemini built-in tool mapped (search) and is silent on the rest.**
- The only key in `native_tools` on the Gemini endpoints checked is `openrouter:web_search`. OpenRouter's server tool pages are: advisor, apply-patch, bash, datetime, fusion, image-generation, search-models, shell, subagent, tool-search, web-fetch, web-search. A text search of the spec returns 0 matches for `url_context`, and `google_search` appears once (the description quoted in GG71).
- The mirror has 0 matches for `groundingMetadata`, `searchEntryPoint`, `groundingSupports`, `webSearchQueries`, and `googleMaps`.
- Quote (batch page): "Google Vertex rejects web search tools."
- Source: GG-s7, GG-s11, GG-s5, GG-s14. Basis: OR-inferred (from the endpoint lists, the spec, and missing terms in the documentation).

### I. The Files API and media

**GG73. The Files API stores uploads for 48 hours under a URI the request then references.**
- Quote: "The Files API lets you store up to 20 GB of files per project, with a per-file maximum size of 2 GB. Files are stored for 48 hours."
- Quote: "you can't download user-uploaded files." and "The Files API is available at no cost in all regions where the Gemini API is available."
- Quote (REST): `{"file_data":{"mime_type": "'"${MIME_TYPE}"'", "file_uri": '$file_uri'}}`
- Quote (file input methods page): "File API uploads are temporary and expire after 48 hours."
- Source: GG-s39. Basis: Vendor-documented.

**GG74. Media can also arrive inline, by public or signed URL, or as a YouTube link.**
- Input method table: inline data "100 MB per request or payload (**50 MB for PDFs**)".
- Quote: "You can pass publicly accessible HTTPS URLs or pre-signed URLs directly in your request. The Gemini API will fetch the content securely during processing." and "Each file can be up to 15 MB."
- Quote (YouTube, REST): `"file_uri": "https://www.youtube.com/watch?v=9hE5-98ZeCg"` with "Preview: The YouTube URL feature is in preview and is available at no charge." and "You can only upload public videos (not private or unlisted videos)." and "For Gemini 2.5 and later models, you can upload a maximum of 10 videos per request."
- Source: GG-s39. Basis: Vendor-documented.

**GG75. Media resolution is a Gemini-specific token control with a global and a per-part form.**
- Quote: "The `media_resolution` parameter controls how the Gemini API processes media" and the list "[Per part] (Gemini 4 and Gemini 3 models)" and "[Globally] for an entire" request.
- Levels: `MEDIA_RESOLUTION_UNSPECIFIED`, `MEDIA_RESOLUTION_LOW`, `MEDIA_RESOLUTION_MEDIUM`, `MEDIA_RESOLUTION_HIGH`, and "`MEDIA_RESOLUTION_ULTRA_HIGH` (Per part only)".
- Quote: "Per-part media resolution is an experimental feature."
- Source: GG-s40. Basis: Vendor-documented.

**GG76. OpenRouter passes video by URL or base64 and documents that the two Google providers differ.**
- Quote: "Video URL support varies by provider. OpenRouter only sends video URLs to providers that explicitly support them. For example, Google Gemini on AI Studio only supports YouTube links (not Vertex AI)."
- Quote: "**Google Gemini (AI Studio)**: Only supports YouTube links" and "**Google Gemini (Vertex AI)**: Does not support video URLs. Use base64-encoded data URLs instead"
- Quote: "Video processing modes are currently a **Google Gemini** feature. When a processing mode is set, OpenRouter routes the request to an endpoint that supports it. Other providers ignore the `processing` field." with values "`agentic`" and "`static`".
- Quote (spec): "Video processing mode. `agentic` enables agentic video processing and `static` forces fixed-rate frame sampling on providers that support it (currently Google Gemini)."
- Source: GG-s12, GG-s5. Basis: OR-documented.

**GG77. OpenRouter forwards one kind of Google-held file reference, and only on the caller's own Vertex key.**
- Quote: "On a Vertex BYOK request to a Gemini model, an image or PDF in a user message can be a `gs://bucket/object` reference instead of a URL or base64 data. OpenRouter forwards the reference to Vertex as-is and never downloads the object"
- Quote: "a `gs://` reference that reaches an OpenRouter-funded Vertex endpoint, or any other provider, is rejected with a 400 by OpenRouter or fails at the provider."
- Quote (batch page, provider table row for Google AI Studio): "No. Public YouTube URLs work as `video_url` parts."
- Source: GG-s13, GG-s14. Basis: OR-documented.

**GG78. OpenRouter's documentation has no term for Gemini Files API URIs or for media resolution on chat models.**
- The mirror has 0 matches for `fileUri`, `file_uri`, `media_resolution`, and `mediaResolution`. The one statement about image resolution is for a different product: "Gemini Decision 1 forwards `low` and `high` as Google's image resolution and drops `auto`."
- Source: GG-s56, GG-s12. Basis: OR-inferred (from terms missing in the documentation). Confidence note: a Files API URI belongs to the Google project that uploaded it, and on OpenRouter-funded requests that project is OpenRouter's, so a caller's own URI has no upload path shown in the documentation.

### J. The Interactions API

**GG79. Google now treats the Interactions API as the default and `generateContent` as legacy.**
- Quote (documentation index): "**API Choice:** Use the Interactions API (`client.interactions`) as the default across all workflows. `generateContent` is still supported for existing integrations, but is considered a legacy API."
- Quote: "As of June 2026, it is Generally Available and the recommended interface for all new projects."
- Quote: "**Where new things launch**: Going forward, new models and capabilities beyond the core mainline family, along with new agentic capabilities and tools, will launch on the Interactions API."
- Source: GG-s45, GG-s43. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`.

**GG80. The server holds the conversation by default, and three request fields are not remembered.**
- Quote: "By default, the Interactions API stores requests so you can leverage the server-side state management features by using `previous_interaction_id`. You can opt into stateless behavior by setting `store=false`."
- Quote: "The other parameters are **interaction-scoped** and apply only to the specific interaction you are currently generating: `tools`, `system_instruction`, `generation_config` (including `thinking_level`, `temperature`, etc.)" (the three names are a bullet list on the page) and "This means you must re-specify these parameters in each new interaction"
- Quote: "**Paid Tier**: The system retains interactions for **55 days**." and "**Free Tier**: The system retains interactions for **1 day**."
- Quote: "`store=false` is incompatible with [background execution] and prevents using `previous_interaction_id` for subsequent turns."
- Source: GG-s43. Basis: Vendor-documented.

**GG81. An interaction is a list of typed steps, and three `generateContent` features are still missing from it.**
- Quote: "These steps include model thoughts, server-side or client-side tool calls and results (like `function_call` and `function_result`), and the final `model_output`."
- Quote: "The following features are supported by the [`generateContent`] API but are **not yet available** in the Interactions API: **[Batch API]**, **[Automatic function calling (Python)]**, **[Explicit caching]**" (a bullet list on the page).
- Quote (errors page example, the tool types the API accepts): "Supported values: 'function', 'code_execution', 'mcp_server', 'filesystem', 'google_maps', 'google_search', 'bash', 'computer_use', 'file_search', 'url_context'."
- Quote (limitations): "When mixing different models in a conversation (either stateful or stateless), subsequent models must support the output modalities of the previous models as input."
- Source: GG-s43, GG-s31. Basis: Vendor-documented.

**GG82. OpenRouter serves Gemini on its own stateless routes and has no page about the Interactions API.**
- Catalog value: `google/gemini-4-argon` `endpoints` is `["/api/v1/chat/completions", "/api/v1/completions", "/api/v1/messages", "/api/v1/responses"]`.
- The mirror has 0 matches for `previous_interaction_id` and `thinking_summaries`.
- Quote (Gemini 4 guide): "This is the stateless pattern the Responses API already uses for encrypted reasoning. OpenRouter does not store the response, so `previous_response_id` cannot be used to continue."
- Source: GG-s6, GG-s1. Basis: OR-documented (the routes and the stateless statement, which matches LC4). Which Google API OpenRouter calls upstream is not stated; the `native_finish_reason` values `CONTINUATION` and `MAX_TOKENS` and the `thinkingLevel` and `thinkingBudget` field names in OpenRouter's pages are `generateContent` vocabulary (OR-inferred).

### K. Function calling rules

**GG83. Function calling modes, including one that the usual OpenAI vocabulary lacks.**
- Quote (API reference, `functionCallingConfig.mode`): "`AUTO`: Default model behavior, model decides to predict either a function call or a natural language response." / "`ANY`: Model is constrained to always predicting a function call only. If "allowed_function_names" are set, the predicted function call will be limited to any one of "allowed_function_names", else the predicted function call will be any one of the provided "function_declarations"." / "`NONE`: Model will not predict any function call." / "`VALIDATED`: Model decides to predict either a function call or a natural language response, but will validate function calls with constrained decoding."
- Quote: "**allowedFunctionNames**: `string array` Optional. A set of function names that, when provided, limits the functions the model will call. This should only be set when the Mode is ANY or VALIDATED."
- Quote (guide): "`VALIDATED`: Default mode for tool combination (when built-in tools or structured outputs also enabled)." and "`AUTO`: Default mode when only function_declarations tool enabled."
- Source: GG-s30, GG-s33. Basis: Vendor-documented.

**GG84. OpenRouter reports all four OpenAI `tool_choice` forms as supported on Gemini endpoints.**
- Endpoint API value on every Gemini endpoint checked: `"supports_tool_choice": {"none": true, "auto": true, "required": true, "function": true}`.
- Source: GG-s7. Basis: OR-documented. Confidence note: the natural mapping is `none` to `NONE`, `auto` to `AUTO`, `required` to `ANY`, and a named function to `ANY` with one entry in `allowedFunctionNames`. OpenRouter's pages do not state the mapping (OR-inferred from the vendor enum).

**GG85. Function call ids exist on Gemini 3 and later and must be echoed, and Vertex describes a fallback match by name and order.**
- Quote: "**Always map function IDs:** Gemini 3 now always returns a unique `id` with every `functionCall`. Include this exact `id` in your `functionResponse` so the model can accurately map your result back to the original request."
- Quote (API reference, `FunctionCall.id`): "Optional. Unique identifier of the function call. If populated, the client to execute the `function_call` and return the response with the matching `id`."
- Quote (Vertex): "The API matches parallel function responses to function calls by `id` (if provided in `v1beta1`), or by `name` and sequential order in the `parts` array."
- Quote (agentgateway source comment, a third-party statement): "Vertex rejects an `id` on functionCall/functionResponse parts"
- Source: GG-s33, GG-s30, GG-s47, GG-s51. Basis: Vendor-documented, with one Filled (agentgateway) statement that conflicts in part (see Conflicts).

**GG86. Google's function calling page gives two different answers on the order of parallel results.**
- Quote: "When the model initiates multiple function calls in a single turn, you don't need to return the `function_result` objects in the same order that the `function_call` objects were received. The Gemini API maps each result back to its corresponding call using the `id` from the model's output."
- Quote (same section, later): "To send the results back, include the responses in the same order as they were requested."
- Source: GG-s33. Basis: Vendor-documented. The grouping rule in GG9 (all calls, then all responses) is stated without contradiction.

**GG87. Function declarations use a subset of OpenAPI schema, and complex schemas can be rejected.**
- Quote: "You define functions using JSON, specifically with a [select subset] of the [OpenAPI schema] format."
- Quote: "Only a [subset of the OpenAPI schema] is supported." and "For `ANY` mode, very large or deeply nested schemas may be rejected."
- Quote: "When you use `ANY` mode or define complex parameter and response schemas, the API compiles your function declarations into a state machine that constrains token generation." and "If the combined complexity of your tools exceeds these limits, the API rejects the request with a `400 INVALID_ARGUMENT` error."
- API reference: a function declaration has both `parameters` (a `Schema`) and `parametersJsonSchema` (`object`).
- Quote (name rule): "The name of the function to call. Must be a-z, A-Z, 0-9, or contain underscores and dashes, with a maximum length of 128."
- Source: GG-s33, GG-s30. Basis: Vendor-documented.

**GG88. Roles and the system instruction.**
- Quote (`Content.role`): "Optional. The producer of the content. Must be either 'user' or 'model'."
- Quote (`systemInstruction`): "Optional. Developer set [system instruction(s)]. Currently, text only."
- Function results go back as `functionResponse` parts inside a `"role": "user"` content (thought signature page examples).
- Quote (OpenRouter caching page): "Gemini has a single `systemInstruction` field"
- Source: GG-s30, GG-s20, GG-s3. Basis: Vendor-documented.

**GG89. A function result can hold media on Gemini 3 and later, and a function call is not always the last part.**
- Quote: "For Gemini 3 series models, you can include multimodal content in the function response parts that you send to the model."
- Quote: "the model may return a mix of `functionCall`, `toolCall`, and `toolResponse` parts in a single turn. Because of this, don't assume the `functionCall` will always be the last item in the parts array."
- Source: GG-s33. Basis: Vendor-documented.

### L. Structured output, sampling, image output, and usage

**GG90. Structured output takes a JSON Schema subset, ignores what it does not support, and rejects schemas that are too complex.**
- Quote: "To generate a JSON object, set `response_json_schema` (or `responseFormat.text.schema` in REST) in the generation configuration."
- Quote: "**Schema subset:** Not all features of the JSON Schema specification are supported. The model ignores unsupported properties." and "**Schema complexity:** Very large or deeply nested schemas may be rejected."
- Quote: "If your schema exceeds these limits, the API rejects the request with a `400 INVALID_ARGUMENT` error."
- Quote (API reference): "**responseMimeType**: `string` Optional. MIME type of the generated candidate text. Supported MIME types are: `text/plain`: (default) Text output. `application/json`: JSON response in the response candidates. `text/x.enum`: ENUM as a string response in the response candidates."
- Quote: "For Gemini 4 and Gemini 3 series models, you can use function calling with [structured output]."
- Source: GG-s41, GG-s30, GG-s33. Basis: Vendor-documented. Note: the method reference page lists the field `responseJsonSchema` with the text "An internal detail. Use `responseJsonSchema` rather than this field.", which reads like a description meant for an older field (`responseSchema`).

**GG91. OpenRouter exposes structured output through the standard `response_format` and marks support per endpoint.**
- Quote: "Support is determined per endpoint, not just per model: the same model may be served by multiple providers, and only some of those providers may support structured outputs."
- Catalog and endpoint values: Gemini chat models list `response_format` and `structured_outputs` in `supported_parameters` on both the Google AI Studio and the Vertex endpoints checked.
- Source: GG-s57, GG-s6, GG-s7. Basis: OR-documented. Confidence note: no OpenRouter page says how an OpenAI-style schema is cut down to Gemini's subset.

**GG92. Sampling parameters are ignored on Gemini 3.6 and later and on Gemini 4, with an error announced for the future.**
- Quote (troubleshooting): "Gemini 4 and Gemini 3.6+ models ignore custom values for `temperature`, `top_p`, and `top_k` and return deprecation warning headers (and will return `400` errors in a future release). Remove `temperature`, `top_p`, and `top_k` from your request configurations."
- Quote (parameter table): "Candidate count | 1-8 (integer, unsupported on Gemini 3 and 4)" and "Temperature | 0.0-1.0 (ignored on Gemini 3.6+ and 4; keep at default on earlier Gemini 3 models)"
- Quote (API reference, `temperature`): "This field is no longer supported and is silently ignored on Gemini 3.6 Flash and newer models."
- Quote (Gemini 4 Carbon migration list): "**Remove deprecated parameters:** Remove `temperature`, `top_p`, `top_k`, and `candidate_count` from requests. If provided, sampling parameters are ignored."
- Source: GG-s32, GG-s30, GG-s44. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`.

**GG93. OpenRouter's catalog still lists the sampling parameters as supported on Gemini 4.**
- Catalog value: `google/gemini-4-argon` `supported_parameters` is `include_reasoning, max_tokens, reasoning, reasoning_effort, response_format, seed, stop, structured_outputs, temperature, tool_choice, tools, top_p`.
- Source: GG-s6. Basis: OR-documented. Confidence note: "supported" here means accepted by the endpoint. Google ignores the value upstream (GG92). This fits LC3 (unsupported parameters are not rejected).

**GG94. `usageMetadata` reports thoughts and cached tokens as their own counts.**
- Quote: "**promptTokenCount**: `integer` Number of tokens in the prompt. When `cached_content` is set, this is still the total effective prompt size meaning this includes the number of tokens in the cached content." / "**cachedContentTokenCount**: `integer` Number of tokens in the cached part of the prompt (the cached content)" / "**candidatesTokenCount**: `integer` Total number of tokens across all the generated response candidates." / "**toolUsePromptTokenCount**: `integer` Output only. Number of tokens present in tool-use prompt(s)." / "**thoughtsTokenCount**: `integer` Output only. Number of tokens of thoughts for thinking models." / "**totalTokenCount**: `integer` Total token count for the generation request (prompt + tool-use prompt + thoughts + response candidates)."
- Source: GG-s30. Basis: Vendor-documented. Note: `candidatesTokenCount` excludes thoughts (the long decode example shows `"candidatesTokenCount": 0` beside `"thoughtsTokenCount": 32768`), while OpenAI's `completion_tokens` includes reasoning tokens.

**GG95. OpenRouter reports the same quantities in OpenAI-shaped usage fields.**
- Quote: "To detect this case, subtract `usage.completion_tokens_details.reasoning_tokens` from `usage.completion_tokens`."
- Quote: "Cache activity is reported in `usage.input_tokens_details` (Responses) and `usage.prompt_tokens_details` (Chat Completions): `cache_write_tokens` counts prompt tokens written to the cache, and `cached_tokens` counts prompt tokens read from it."
- Catalog value: Google models price reasoning separately as `"internal_reasoning"`, equal to the `completion` price on the models checked.
- Source: GG-s2, GG-s3, GG-s6. Basis: OR-documented (the field names, which match LC17). That OpenRouter adds `thoughtsTokenCount` to `candidatesTokenCount` to form `completion_tokens` is OR-inferred from the subtraction rule.

**GG96. Image output: Gemini returns image parts with signatures, and OpenRouter returns images in a message field of its own.**
- Vendor side: see GG14 for the signature rule on image parts. The `generateContent` request selects output types with `responseModalities`.
- OpenRouter side (`src/chat/index.ts` of its AI SDK provider, lines 501-511): the provider reads `choice.message.images` and takes each `image.image_url.url`. The catalog lists `/api/v1/images` among the endpoints of the Gemini image models, for example `google/gemini-3-pro-image`, whose `reasoning` object is `{"mandatory": true}`.
- Source: GG-s46, GG-s19, GG-s6. Basis: OR-documented (official SDK source and catalog). Confidence note: how the per-image signatures come back for multi-turn image editing through OpenRouter was not found in the documentation.

### M. Google's own OpenAI-compatible route

**GG97. The route, its thinking mapping, and its escape field.**
- Quote: "`base_url="https://generativelanguage.googleapis.com/v1beta/openai/"`"
- Mapping table: `reasoning_effort` `minimal` maps to `low` on Gemini 4 and to a budget of `1,024` on Gemini 2.5; `low`, `medium`, `high` map to the same names on Gemini 4; `xhigh` maps to "`xhigh` (`gemini-4-neon` and `gemini-4-hydrogen` only)".
- Quote: "If you want to disable thinking, you can set `reasoning_effort` to `"none"` for 2.5 models. Reasoning cannot be turned off for Gemini 2.5 Pro, Gemini 3, or Gemini 4 models."
- Quote (REST): `"extra_body": { "google": { "thinking_config": { "thinking_level": "high", "include_thoughts": true } } }`
- Quote: "Note that `reasoning_effort` and `thinking_level`/`thinking_budget` overlap functionality, so they can't be used at the same time."
- Source: GG-s42. Basis: Vendor-documented.

**GG98. What the route keeps through `extra_body`, and for which endpoint.**
- Table rows: "**`cached_content`** | Text | Chat | Corresponds to Gemini's general content cache." / "**`thinking_config`** | Object | Chat | Corresponds to Gemini's ThinkingConfig." / "**`safety_settings`** | List | Images | Custom safety threshold filters" / "**`tools`** | List | Images | Enables grounding (e.g., `[{"google_search": {}}]`). Only for `gemini-3-pro-image-preview`."
- Quote (image section): "Any other parameters not listed here or in the [`extra_body`] section will be silently ignored by the compatibility layer."
- Quote (caching page): "If you're using an [OpenAI library], you can enable explicit caching using the `cached_content` property on [`extra_body`]."
- Source: GG-s42, GG-s26. Basis: Vendor-documented.

**GG99. What the route loses or leaves undocumented.**
- Quote: "**OpenAI compatibility endpoint:** Long decode is not available. Output stops at the per-request cap."
- Quote: "Support for the OpenAI libraries is still in beta while we extend feature support."
- The table in GG98 lists `safety_settings` and `tools` for the Images endpoint only. The page has no row for chat-route safety settings, Google Search on chat, URL context, code execution, `promptFeedback`, or `groundingMetadata`.
- Thought signatures on this route: see GG25 (Vertex page shows `extra_content.google.thought_signature` on tool calls) and GG26.
- Source: GG-s24, GG-s42. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (its list of losses is corrected under Conflicts).

### N. Provider differences inside OpenRouter (Google AI Studio and Google Vertex)

**GG100. One Gemini model is served by two Google providers with different slugs, tiers, and parameter lists.**
- Endpoint API values for `google/gemini-4-argon`: five endpoints with `provider_name` and `tag` pairs "Google AI Studio" `google-ai-studio`, "Google AI Studio" `google-ai-studio/flex`, "Google AI Studio" `google-ai-studio/priority`, "Google" `google-vertex/global`, "Google" `google-vertex/global/priority`.
- For `google/gemini-4-argon`, `google/gemini-4-hydrogen`, `google/gemini-3.1-pro-preview`, and `google/gemini-3.5-flash`, every Vertex endpoint lists `stop` in `supported_parameters` and every AI Studio endpoint leaves it out.
- `google/gemini-2.5-flash` had three endpoints on the read date, all "Google AI Studio", and all three list `stop`.
- Quote (provider selection page): "For example, `"google-vertex"` matches `google-vertex`, `google-vertex/us-east5`, `google-vertex/us-central1`, and so on."
- Source: GG-s7, GG-s16. Basis: OR-documented.

**GG101. A conversation can move between the two providers between turns unless the caller pins one.**
- Quote: "After a request that uses prompt caching, OpenRouter remembers which provider served your request." and "If the sticky provider becomes unavailable, OpenRouter automatically falls back to the next-best provider."
- Quote: "When `session_id` is set, sticky routing activates on any successful request" and "Without `session_id`, sticky routing only activates after a cache hit is detected."
- Quote (batch example): `"provider": { "only": ["google-vertex"], "allow_fallbacks": false }`
- Source: GG-s3, GG-s14. Basis: OR-documented. Confidence note: the pinning rule OpenRouter states for continuation tokens (GG23) is a hard pin. For ordinary thought signatures and for caches the documented mechanism is the soft sticky routing above. What OpenRouter does with a signature after such a move is an open question (GG22 shows it knows the issuing endpoint).

**GG102. Documented feature differences between the two providers.**
- Video: YouTube links on AI Studio only, base64 only on Vertex (GG76).
- Files: `gs://` references on Vertex with the caller's own key only (GG77).
- Quote (batch): "Public YouTube URLs work only on Google AI Studio, as `video_url` parts on `/v1/chat/completions`." / "Google Vertex rejects web search tools." / "Google AI Studio allows 48 hours, and Google Vertex allows 96 hours."
- Parameters: `stop` is listed on Vertex endpoints only for Gemini 3.x and 4 (GG100).
- Source: GG-s12, GG-s13, GG-s14, GG-s7. Basis: OR-documented.

**GG103. OpenRouter's reasoning format list has separate tags for other vendors' models on Vertex, and one tag for Gemini on both Google providers.**
- Quote: "`"google-vertex-meta-responses-v1"` - Meta on Google Vertex responses format version 1" / "`"google-vertex-xai-responses-v1"` - xAI on Google Vertex responses format version 1" / "`"google-gemini-v1"` - Google Gemini format version 1"
- Quote (changelog): "the value was renamed to `google-vertex-meta-responses-v1`. It identifies encrypted reasoning minted by Meta models served on Google Vertex"
- Source: GG-s2, GG-s17. Basis: OR-documented. Note: the sample in GG22 has `"format": "google-gemini-v1"` on a reply served by `google-vertex/global`, so the provider that issued a Gemini signature is recorded inside the wrapped value, not in the format tag.

**GG104. OpenRouter's changelog has almost no Gemini entries.**
- A search of `changelog.md` (730 KB) for `gemini`, `vertex`, `google`, `thought`, `signature`, `safety`, `grounding`, and `continuation` returns nine lines: the `ReasoningFormat` rename above, one `x-ori-invoke-signature-v2` property, and guardrail properties named `allow_safety_retention_google` and `allow_safety_retention_other`.
- Source: GG-s17. Basis: OR-documented. So the changelog gives no dates for when Gemini signature or continuation behavior changed.

**GG105. OpenRouter also runs a separate Google AI Studio adapter for the Gemini Decisions product.**
- Quote (multimodal decisions page, adapter table): "`GoogleAIStudioDecisionsAdapter` | Google AI Studio"
- Source: GG-s56. Basis: OR-documented. This is the only place the documentation names an OpenRouter adapter for Google.

### O. Workspace material

**GG106. The 2026-10-28 survey's Gemini statements, checked against today's pages.**
- Survey text: "thought signatures are mandatory: for parallel calls only the first `functionCall` carries one, and omitting it is a 400 with code `missing_thought_signature`."
- Survey text: "The OpenAI route at `/v1beta/openai/` loses thought signatures, `alt=sse` behavior, `continuationToken`, safety settings and cache handles."
- Survey text: "The explicit-cache minimum is **per model, not universal**: 4,096 tokens for the Gemini 3 family and 2,048 for Gemini 2 and 2.5"
- Source: GG-s52. Basis: Workspace (document dated 2026-10-28). Re-checked live: the parallel rule, the blocked-prompt rule, the `CONTINUATION` rule, the Interactions vocabulary, and the cache immutability rule hold (GG3, GG60, GG41, GG36, GG52). The error code claim and the OpenAI route claim need correction (see Conflicts).

**GG107. The 2026-09-20 report's Gemini rows.**
- Report text: "Gemini 3 | `thought_signature` on the first `functionCall` of each step in the current turn | ids optional | "Function call `FC1` in the `1.` content block is missing a `thought_signature`""
- Report text: "Gemini's exposes signatures through `extra_content`" and, in the replay table, "same endpoint only; AI Studio to Vertex is HARD"
- Source: GG-s53. Basis: Workspace (document dated 2026-09-20). Re-checked live: the validation rule and error text match GG4 and GG5, `extra_content` matches the Vertex page (GG25), and the AI Studio to Vertex statement matches LiteLLM's comment (GG29) but not a Google page. "ids optional" needs an update: Gemini 3 "now always returns a unique `id`" and asks for it back (GG85).

**GG108. Runvil's cloud provider code for Gemini reads the model list and already records both Google base URLs.**
- Code: `base_url: "https://generativelanguage.googleapis.com"` and `openai_base_url: Some("https://generativelanguage.googleapis.com/v1beta/openai")`; the list call is `GET {base_url}/v1beta/models` with header `x-goog-api-key` and `pageToken` paging; it reads `inputTokenLimit`, `outputTokenLimit`, `supportedGenerationMethods`, and the boolean `thinking`.
- Code comment: "The endpoint reports no effort levels, pricing, release date, or deprecation status."
- Source: GG-s54, GG-s32. Basis: Workspace (code, not dated in the file; its fixture is named `2026-09-14-gemini.json`). Not re-checked live, because the model list call needs an API key. Related live facts: Google says `models.list` "only validate that your API key exists" (troubleshooting page), and thinking levels per model come from the documentation table (GG33) or OpenRouter's catalog (GG39), not from this endpoint.

## Conflicts and disagreements

1. **The error code `missing_thought_signature` (local survey of 2026-10-28, and this task's own brief).** The survey says omitting a signature "is a 400 with code `missing_thought_signature`". Today's `generateContent` page gives the 400 as a message ("Function call `<Function Call>` in the `<index of contents array>` content block is missing a `thought_signature`", GG5). The token appears as a `finishReason` value on `generateContent` and as a "generation error code" on the Interactions API (GG6), and those two descriptions disagree with each other (request missing it, or response missing it). The live pages are right about what they say. The survey joined two separate things. Whether the 400 body also holds this code was not tested, because nothing was run against a model.
2. **What Google's OpenAI-compatible route loses (local survey of 2026-10-28).** The survey lists "thought signatures, `alt=sse` behavior, `continuationToken`, safety settings and cache handles". Today: signatures on tool calls are kept in `extra_content.google.thought_signature` (GG25), and cache handles are kept as `cached_content` in `extra_body` (GG98). Long decode is confirmed lost (GG99). Chat-route safety settings have no documented field (GG98 lists `safety_settings` for Images only). The 2026-09-20 report already said signatures travel in `extra_content`, so the two workspace documents disagree and the 2026-09-20 one is right on this point.
3. **"ids optional" for Gemini 3 (report of 2026-09-20).** Google now says Gemini 3 "always returns a unique `id` with every `functionCall`" and tells the caller to echo it (GG85). The report's row is out of date.
4. **OpenRouter's caching page against Google's caching page.** OpenRouter: cached tokens cost "0.25x", minimums are "1024 tokens for Gemini 2.5 Flash and 4096 tokens for Gemini 2.5 Pro", TTL "on average 3-5 minutes" (GG55). Google today: minimum 2,048 for both 2.5 models and 4,096 for 3.x and 4 (GG51), a 95 percent discount on Gemini 4 (GG53), and "The cache expires after 15 to 20 minutes without use" for Gemini 4 hops (GG45). OpenRouter's own catalog agrees with Google on price for Gemini 4 (cache read is 5 percent of the prompt price, GG58). Google's page is the newer statement; OpenRouter's text was written for Gemini 2.5.
5. **Explicit caching on Gemini 4.** Google: named caches are "not supported for Gemini 4 models" and return an error on Gemini 4 Carbon (GG53). OpenRouter: its caching page describes `cache_control` for Gemini without a model limit (GG56), its catalog lists an `input_cache_write` price for Gemini 4 models (GG58), and its Gemini 4 guide says only that explicit caching "is skipped on continuation hops" (GG49). Not resolved: what OpenRouter does with a `cache_control` marker on a Gemini 4 request.
6. **Implicit caching flag.** OpenRouter's endpoint API reports `"supports_implicit_caching": false` on every Gemini endpoint checked (GG58), while OpenRouter's caching page and Google both say implicit caching is on for Gemini 2.5 and later (GG55, GG51). Not resolved. The flag may mean something narrower than its name.
7. **Default thinking level and `xhigh`.** Google gives `high` as the default for Gemini 4 Neon and Hydrogen (GG33); OpenRouter's catalog gives `default_effort: "medium"` for both (GG39). OpenRouter's reasoning page says `xhigh` is "mapped down" to `high` (GG38, written for Gemini 3); the catalog lists `xhigh` as supported on Neon and Hydrogen, and Google accepts `xhigh` natively on those two.
8. **Gemini 2.5 and thinking levels inside Google's own documentation.** The `generateContent` page says "Gemini 2.5 series models don't support `thinkingLevel`" (GG34). The Interactions page lists the three 2.5 models with levels "low, medium, high" (GG36). Both can be true if the Interactions API converts a level to a budget for 2.5, but neither page says so.
9. **Order of parallel function results inside Google's own documentation.** One paragraph says the order does not matter because results map by `id`; a later paragraph says "include the responses in the same order as they were requested" (GG86); the Vertex page says to return them in the same order (GG9). All three agree that calls and responses must be grouped, not interleaved.
10. **Function call `id` on Vertex.** Google Cloud's page says results match "by `id` (if provided in `v1beta1`), or by `name` and sequential order" (GG9). agentgateway's source says "Vertex rejects an `id` on functionCall/functionResponse parts" and strips it (GG32). Vercel AI SDK has an option named `includeFunctionCallIds` for the same reason. This looks like a difference between Vertex API versions. Not resolved here.
11. **Do signatures move between AI Studio and Vertex?** Google's pages are silent. LiteLLM says no and quotes two 400 messages (GG29). Vercel AI SDK's changelog describes keeping a Vertex signature when failing over to AI Studio as a fix (GG30). OpenRouter records the issuing endpoint inside the wrapped signature (GG22) and pins continuation tokens to one provider (GG23). The LiteLLM statement is the most specific, and it matches the 2026-09-20 report, but it is a third-party code comment.
12. **The dummy signature.** The AI Studio page names two strings, the Vertex page names one (GG8). Vercel AI SDK and Gemini CLI send the plain string `skip_thought_signature_validator`; LiteLLM sends its base64 encoding (GG28). Which forms each backend accepts was not tested.
13. **Per-request output cap for Gemini 4 Argon.** OpenRouter says Argon "returns at most 65,536 output tokens per request". Google says a direct non-streaming REST call to Carbon or Argon returns up to 32,768 and streaming returns up to 65,536 (GG41). The two fit together if OpenRouter always streams upstream, which the brief's Runvil notes say its own gateway client also does.
14. **A defect in Google's page.** The OpenAI compatibility page sends readers to the thought signature page for "the full example" in chat completions form, and that section today shows native JSON only (GG26). The Vertex page has the working example (GG25).
15. **Local survey claims that still hold for Gemini.** LC3 (unsupported parameters are not rejected: GG93), LC4 (`previous_response_id` unusable: GG50), LC6 (`reasoning_details` with a vendor `format` tag, replayed unmodified: GG17 to GG19), LC7 (TTL dropped toward Google: GG57), LC10 (`native_finish_reason`: GG47, GG64), LC17 (`cache_write_tokens`: GG95).

## Open questions blocked by closed source

1. **How does OpenRouter's server put each `reasoning.encrypted` entry back onto the right Gemini part (first `functionCall` of a step, a final text part, an image part)?** Tried: reasoning tokens page, Gemini 4 guide, the spec, OpenRouter's AI SDK provider (which only stores and returns the array). Fill found: three designs exist. Google's own OpenAI route puts the signature on the tool call (`extra_content.google.thought_signature`, GG25). LiteLLM and agentgateway hide it in the tool call id as `<id>__thought__<signature>` (GG27, GG32). Vercel AI SDK keeps it per part in `providerOptions.google.thoughtSignature` (GG30).
2. **What does OpenRouter do when the history holds tool calls from another model and the target is Gemini 3 or 4?** Tried: the whole mirror (0 matches for either dummy string, GG24). Fill found: Google documents the dummy value (GG8); LiteLLM, Vercel AI SDK, and Gemini CLI all inject `skip_thought_signature_validator` on the first function call of a step when the signature is missing (GG28, GG30, GG31).
3. **What does OpenRouter do when a caller sends back the tool calls but drops `reasoning_details`?** Tried: same pages. Fill found: same as question 2. The OpenRouter provider test comment says "otherwise Gemini will reject the continuation" (GG19), which suggests the error passes through, but that is a comment in a test file.
4. **What happens to a signature when routing moves from AI Studio to Vertex or back between turns?** Tried: caching page (sticky routing), provider selection page, the sample envelope (GG22). Fill found: LiteLLM retries the request with the dummy value after a 400 that mentions "thought signature" (GG28), and states signatures are backend-bound (GG29). Vercel AI SDK points the other way (GG30).
5. **Can a caller set Gemini `safetySettings` through OpenRouter, and which thresholds does OpenRouter send by default?** Tried: spec (`provider.options` exists with `google-ai-studio` and `google-vertex` keys, no allowlist published for chat models, GG65), parameters page, catalog, endpoint API. No fill found for OpenRouter's defaults. Google's own OpenAI route documents `safety_settings` for Images only (GG98).
6. **How do Gemini's `finishReason` values and a no-candidate blocked prompt map onto OpenRouter's five finish reasons and its 403 `content_policy_violation` error?** Tried: errors page (one example, "Gemini's `SAFETY` block"), API overview. Fill found: LiteLLM's map and agentgateway's synthetic empty choice with `content_filter` (GG67).
7. **How does a `cache_control` marker become a `cachedContents` object: when is it created, reused, and deleted, and what happens on Gemini 4?** Tried: caching page (GG56), Gemini 4 guide. Fill found in part: LiteLLM has a Gemini context caching module (seen as an import of `..context_caching.rejected_cache_retry` in the file quoted in GG28); it was not read in depth here because another subagent reads LiteLLM.
8. **Does anything from `groundingMetadata` beyond `url_citation` annotations reach the caller (search queries, the `searchEntryPoint` HTML that Google requires to be shown, the segment-to-source index)?** Tried: both web search pages, the spec, the mirror (0 matches for the field names, GG72). No fill found.
9. **Can URL context, Maps grounding, or code execution be switched on through OpenRouter?** Tried: server tools pages, `native_tools` on endpoints, spec. No fill found. Google's rule that all `toolCall` and `toolResponse` parts with their signatures must come back (GG13) has no visible slot in a chat completions message.
10. **Which Google API does OpenRouter call (`generateContent` or Interactions), and does it ask for thought summaries by default?** Tried: all Gemini pages in the mirror. Inferred `generateContent` from vocabulary (GG82). No fill needed for the first part; no fill found for the second.
11. **How does OpenRouter cut an OpenAI-style JSON Schema (tools and `response_format`) down to Gemini's subset?** Tried: structured outputs page, tool calling page. No fill read here; other subagents read LiteLLM and the open-source gateways on this point.
12. **Does OpenRouter pass Gemini's own `functionCall.id` through as the tool call id, or make its own?** Tried: tool calling page, spec. Fill found: agentgateway uses the native id when present and otherwise builds `call_{seed}_{index}` (`vertex_gemini.rs`, lines 1262-1264), and LiteLLM forwards the id only when a flag (`forward_function_call_id`) is set (GG28 file, lines 1443-1445).

## Unverified

- Nothing was run against a live model, so every error status and message here is as documented, not as observed.
- From memory, not found in any source today: that OpenRouter's Gemini `reasoning.encrypted` entries have an `id` equal to the tool call id they belong to. The spec allows an `id`, and the only documented id value is `gemini-continuation`.
- From memory, not read today: that LiteLLM accepts a `safety_settings` request parameter for Gemini, and that Vercel AI SDK returns `groundingMetadata` and `safetyRatings` in `providerMetadata.google`.
- The minimum size for a named cache per model. Today's page gives the per-model table under implicit caching and says of explicit caching only that the minimum "varies by model". The survey's figures (4,096 for Gemini 3, 2,048 for 2 and 2.5) match the implicit table but were not confirmed for named caches.
- Vertex differences for caching, safety settings, grounding, and the Interactions API. Only the Vertex thought signature page was read. Google's index points to "Interactions API on Gemini platform" for the Vertex form, which was not read.
- Google's Terms of Service text behind "the required Search Suggestions". Only the pointer on the grounding page was read.
- Whether `"supports_implicit_caching": false` in OpenRouter's endpoint API is a data error or a narrower flag.
- The Runvil model list code for Gemini was not re-checked against the live `GET /v1beta/models` response, because that call needs an API key.
- OpenRouter's `xhigh` handling for Gemini 4 Neon and Hydrogen: the catalog says supported, the reasoning page's mapping table predates Gemini 4.

## Workspace documents used

- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, dated 2026-10-28. Taken: its Gemini paragraph (signatures, blocked prompts, `CONTINUATION`, sampling parameters, Interactions vocabulary, cache rules, the list of what Google's OpenAI route loses). Re-checked live today: yes. Two corrections (Conflicts 1 and 2); the rest holds.
- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, dated 2026-09-20. Taken: the Gemini 3 replay rule and error text, the `extra_content` statement, the "AI Studio to Vertex is HARD" entry in its replay table, the parallel call ordering rule. Re-checked live today: yes. One correction (Conflict 3). The AI Studio to Vertex entry has a third-party source only (GG29).
- `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/gemini.rs`, undated (fixture named 2026-09-14). Taken: how Runvil reads Gemini's model list, the two base URLs it records, and the fact that the list reports a boolean `thinking` flag and no effort levels. Re-checked live today: no (needs a key).

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
| - | - | - | - |
| GG-s1 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gemini-4 | 2026-11-22 | OpenRouter doc |
| GG-s2 | https://openrouter.ai/docs/guides/best-practices/reasoning-tokens | 2026-11-22 | OpenRouter doc |
| GG-s3 | https://openrouter.ai/docs/guides/best-practices/prompt-caching | 2026-11-22 | OpenRouter doc |
| GG-s4 | https://openrouter.ai/docs/guides/overview/multimodal/image-understanding (sample output block) | 2026-11-22 | OpenRouter doc |
| GG-s5 | https://openrouter.ai/openapi.json (schemas `ReasoningFormat`, `ReasoningDetailEncrypted`, `ChatToolCall`, `ProviderOptions`, `ProviderPreferences`; text searches of the whole file) | 2026-11-22 | OpenRouter spec |
| GG-s6 | https://openrouter.ai/api/v1/models (41 entries whose id starts with `google/gemini` or `~google`) | 2026-11-22 | OpenRouter spec |
| GG-s7 | https://openrouter.ai/api/v1/models/google/{gemini-4-argon, gemini-4-hydrogen, gemini-3.1-pro-preview, gemini-3.5-flash, gemini-2.5-flash}/endpoints | 2026-11-22 | OpenRouter spec |
| GG-s8 | https://openrouter.ai/docs/api_reference/errors-and-debugging | 2026-11-22 | OpenRouter doc |
| GG-s9 | https://openrouter.ai/docs/guides/features/server-tools/web-search | 2026-11-22 | OpenRouter doc |
| GG-s10 | https://openrouter.ai/docs/guides/features/plugins/web-search | 2026-11-22 | OpenRouter doc |
| GG-s11 | https://openrouter.ai/docs/guides/features/server-tools | 2026-11-22 | OpenRouter doc |
| GG-s12 | https://openrouter.ai/docs/guides/overview/multimodal/videos | 2026-11-22 | OpenRouter doc |
| GG-s13 | https://openrouter.ai/docs/guides/overview/auth/byok | 2026-11-22 | OpenRouter doc |
| GG-s14 | https://openrouter.ai/docs/batch-quickstart | 2026-11-22 | OpenRouter doc |
| GG-s15 | https://openrouter.ai/docs/guides/providers/integration (section "Passthrough parameters") | 2026-11-22 | OpenRouter doc |
| GG-s16 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| GG-s17 | https://openrouter.ai/docs/changelog | 2026-11-22 | OpenRouter doc |
| GG-s18 | https://openrouter.ai/docs/api_reference/overview | 2026-11-22 | OpenRouter doc |
| GG-s19 | https://github.com/OpenRouterTeam/ai-sdk-provider at commit 7a235a4cad7624ed60db913b96f5f0f6bd8c8d33 (2026-11-20): `src/chat/index.ts`, `src/chat/convert-to-openrouter-chat-messages.ts`, `src/schemas/format.ts`, `e2e/gemini/reasoning-multiturn.test.ts`, `e2e/issues/issue-418-gemini-thought-signature.test.ts`, `CHANGELOG.md` | 2026-11-22 | OpenRouter SDK source |
| GG-s20 | https://aistudio.google.com/docs/thought-signatures?generate-content=true (reached from https://ai.google.dev/gemini-api/docs/thought-signatures) | 2026-11-22 | vendor doc |
| GG-s21 | https://aistudio.google.com/docs/thought-signatures (Interactions variant) | 2026-11-22 | vendor doc |
| GG-s22 | https://aistudio.google.com/docs/thinking?generate-content=true | 2026-11-22 | vendor doc |
| GG-s23 | https://aistudio.google.com/docs/thinking (Interactions variant) | 2026-11-22 | vendor doc |
| GG-s24 | https://aistudio.google.com/docs/long-decode?generate-content=true | 2026-11-22 | vendor doc |
| GG-s25 | https://aistudio.google.com/docs/long-decode (Interactions variant) | 2026-11-22 | vendor doc |
| GG-s26 | https://aistudio.google.com/docs/caching and https://aistudio.google.com/docs/caching?generate-content=true | 2026-11-22 | vendor doc |
| GG-s27 | https://aistudio.google.com/docs/pricing | 2026-11-22 | vendor doc |
| GG-s28 | https://aistudio.google.com/api-reference/caching | 2026-11-22 | vendor doc |
| GG-s29 | https://aistudio.google.com/docs/safety-settings?generate-content=true | 2026-11-22 | vendor doc |
| GG-s30 | https://aistudio.google.com/api-reference/generate-content and https://aistudio.google.com/api-reference/generate-content/generate-content | 2026-11-22 | vendor doc |
| GG-s31 | https://aistudio.google.com/docs/api-errors and https://aistudio.google.com/docs/api-errors?generate-content=true | 2026-11-22 | vendor doc |
| GG-s32 | https://aistudio.google.com/docs/troubleshooting | 2026-11-22 | vendor doc |
| GG-s33 | https://aistudio.google.com/docs/function-calling?generate-content=true | 2026-11-22 | vendor doc |
| GG-s34 | https://aistudio.google.com/docs/tool-combination?generate-content=true | 2026-11-22 | vendor doc |
| GG-s35 | https://aistudio.google.com/docs/google-search?generate-content=true | 2026-11-22 | vendor doc |
| GG-s36 | https://aistudio.google.com/docs/url-context?generate-content=true | 2026-11-22 | vendor doc |
| GG-s37 | https://aistudio.google.com/docs/maps-grounding?generate-content=true | 2026-11-22 | vendor doc |
| GG-s38 | https://aistudio.google.com/docs/code-execution?generate-content=true | 2026-11-22 | vendor doc |
| GG-s39 | https://aistudio.google.com/docs/files?generate-content=true, https://aistudio.google.com/docs/file-input-methods, https://aistudio.google.com/docs/video-understanding?generate-content=true | 2026-11-22 | vendor doc |
| GG-s40 | https://aistudio.google.com/docs/media-resolution?generate-content=true | 2026-11-22 | vendor doc |
| GG-s41 | https://aistudio.google.com/docs/structured-output?generate-content=true | 2026-11-22 | vendor doc |
| GG-s42 | https://aistudio.google.com/docs/openai | 2026-11-22 | vendor doc |
| GG-s43 | https://aistudio.google.com/docs/interactions-overview | 2026-11-22 | vendor doc |
| GG-s44 | https://aistudio.google.com/docs/whats-new-gemini-4-carbon?generate-content=true | 2026-11-22 | vendor doc |
| GG-s45 | https://aistudio.google.com/llms.txt | 2026-11-22 | vendor doc |
| GG-s46 | https://aistudio.google.com/docs/image-generation?generate-content=true (section "Thought Signatures") | 2026-11-22 | vendor doc |
| GG-s47 | https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/thinking/thought-signatures (reached from https://cloud.google.com/vertex-ai/generative-ai/docs/thought-signatures) | 2026-11-22 | vendor doc |
| GG-s48 | https://github.com/BerriAI/litellm at commit 90dce4b93bd2b090621a6b9083311294d3358bc6 (2026-11-22): `litellm/litellm_core_utils/prompt_templates/factory.py`, `litellm/llms/vertex_ai/gemini/rejected_thought_signature_retry.py`, `litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py`, `litellm/utils.py`, `litellm/litellm_core_utils/core_helpers.py` | 2026-11-22 | gateway source |
| GG-s49 | https://github.com/vercel/ai at commit d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d (2026-11-20): `packages/google/src/convert-to-google-messages.ts`, `packages/google/CHANGELOG.md`, `packages/openai-compatible/src/chat/convert-to-openai-compatible-chat-messages.ts` | 2026-11-22 | harness source |
| GG-s50 | https://github.com/google-gemini/gemini-cli at commit eecccfcd5aff25de5e78135ca20f7abe3f566955 (2026-11-20): `packages/core/src/utils/historyHardening.ts`, `packages/core/src/core/geminiChat.ts` | 2026-11-22 | harness source |
| GG-s51 | https://github.com/agentgateway/agentgateway at commit 888badbbe91f2a57ec8e7bf5a5ef3acda34b1fe5 (2026-11-20): `crates/llm/src/conversion/vertex_gemini.rs`, `crates/llm/src/gemini.rs` | 2026-11-22 | gateway source |
| GG-s52 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (dated 2026-10-28) | 2026-11-22 | workspace document |
| GG-s53 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (dated 2026-09-20) | 2026-11-22 | workspace document |
| GG-s54 | `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/gemini.rs` | 2026-11-22 | workspace document |
| GG-s55 | https://ai.google.dev/gemini-api/docs/gemini-3 (redirects to the "Latest Gemini models" page, indexed as https://aistudio.google.com/docs/latest-model) | 2026-11-22 | vendor doc |
| GG-s56 | https://openrouter.ai/docs/guides/community/multimodal-decisions | 2026-11-22 | OpenRouter doc |
| GG-s57 | https://openrouter.ai/docs/guides/features/structured-outputs | 2026-11-22 | OpenRouter doc |
