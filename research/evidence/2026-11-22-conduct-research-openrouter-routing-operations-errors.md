---
produced: 2026-11-22
title: OpenRouter routing and operations evidence - provider preferences, model fallbacks, variants, routers, presets, BYOK, ZDR, rate limits, error envelope, error_type vocabulary, mid-stream errors, failover, keep-alive comments, cancellation, finish_reason and native_finish_reason, context overflow, context-compression, debug echo_upstream_body
---

# OpenRouter routing and operations evidence - provider preferences, model fallbacks, variants, routers, presets, BYOK, ZDR, rate limits, error envelope, error_type vocabulary, mid-stream errors, failover, keep-alive comments, cancellation, finish_reason and native_finish_reason, context overflow, context-compression, debug echo_upstream_body

- What was read: 43 cited pages of OpenRouter's documentation from the mirror, with more searched by text (routing, variants, routers, presets, BYOK, privacy, ZDR, limits, errors and debugging, streaming, Responses error handling, message transforms, service tiers, FAQ, changelog, and the pages they point to), OpenRouter's OpenAPI 3.1 spec queried by schema name, OpenRouter's official AI SDK provider source, three vendor pages fetched live (Anthropic stop reasons and errors, Gemini finish reasons) plus OpenAI's published spec, four open-source projects at pinned commits (LiteLLM, Vercel AI SDK, pi, Portkey gateway), 64 recorded OpenRouter exchanges kept by the rig project, and two workspace reports.
- Counts: 58 sources, 82 evidence items (O1 to O82), 35 candidate matrix rows.
- Read date: 2026-11-22 for every source. All reads were public and unauthenticated. No request was sent to any model inference endpoint.

## Candidate matrix rows

One row per vendor-specific feature met in this topic. "Caller" means the program that calls OpenRouter. Evidence ids point into the Evidence section.

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
| - | - | - | - | - | - | - | - |
| Stop reason vocabulary | Anthropic | `stop_reason` is one of `end_turn`, `max_tokens`, `stop_sequence`, `tool_use`, `pause_turn`, `refusal`, `model_context_window_exceeded` | Chat: `finish_reason` normalized to `tool_calls`, `stop`, `length`, `content_filter`, `error`, plus `native_finish_reason` with the raw string (`end_turn`, `tool_use` recorded). Messages route: Anthropic values kept (`ORAnthropicStopReason`, adds `compaction`) | No published table for `pause_turn`, `model_context_window_exceeded`, `stop_sequence`. `stop_details` category not described on Chat. `native_finish_reason` is absent from the spec's chat schemas and from the Messages and Responses routes | No state. Neutral values cover stop, length, tool calls, filter. A caller that needs "context full" versus "my cap" or "paused, resend" must read a vendor value | OR-documented (the missing rows are filled from LiteLLM and the Vercel AI SDK in O57) | O52, O53, O54, O55, O56, O57, O77 |
| Stop reason vocabulary | Google Gemini | `finishReason` has 23 values; `STOP` covers tool calls too (no tool-call value) | `finish_reason` normalized; `native_finish_reason` holds `STOP`, `MAX_TOKENS`, `CONTINUATION`. A filter-level `SAFETY` block is an error with `error_type: "content_policy_violation"`, status 403 | Ten filter reasons become one. For tool calls with a thought signature OpenRouter returns `stop`, and its own SDK rewrites it to tool calls | No state. Detect tool calls from `tool_calls` content, not from `finish_reason` | OR-documented (the `stop` beside tool calls case is read from official SDK code, O58) | O54, O56, O58, O77 |
| Stop status on the Responses protocol | OpenAI | `status: "completed"` or `"incomplete"` with `incomplete_details.reason` in `max_output_tokens`, `content_filter`; no tool-call status | Chat route: `finish_reason` derived (`stop` or `tool_calls` beside `native_finish_reason: "completed"`; `length` beside `max_output_tokens`). Responses route: native fields kept | `native_finish_reason` vocabulary differs between OpenAI models served over Responses and over Chat Completions | No state. The native value cannot be read without knowing the upstream protocol, which the response does not name | OR-inferred (from third-party recordings; the Responses route fields themselves are documented) | O54, O77 |
| Truncated tool call | OpenAI (recorded), any | Vendor reports `length` or `max_output_tokens` when output is cut, even inside a tool call | Recorded: `finish_reason: "tool_calls"` with `native_finish_reason: "length"` or `"max_output_tokens"`, arguments cut mid-string | The neutral field hides the truncation | No state. Caller must validate tool arguments as JSON or read the native value | OR-inferred (third-party recordings) | O78 |
| Refusal | Anthropic, OpenAI | Anthropic: `stop_reason: "refusal"` on HTTP 200 with `stop_details`. OpenAI: `refusal` content | Chat: `message.refusal` with `finish_reason: "content_filter"`. Responses: `status: "incomplete"`, `incomplete_details.reason: "content_filter"`, `refusal` part. Refusal sent as an error: `error_type: "refusal"`, 403. Model fallback runs first when allowed | Refusal and filter share `content_filter`; Anthropic's category detail not described on Chat | No state. Neutral value exists | OR-documented | O17, O54 |
| Long-output continuation | Google Gemini | `CONTINUATION` finish with a `continuation_token`; budget is cumulative across hops | `finish_reason: "length"`, `native_finish_reason: "CONTINUATION"` or `"MAX_TOKENS"`, token returned as a reasoning detail (Responses item id `gemini-continuation`, Messages `redacted_thinking`); caller resends the stopped turn | The two native reasons need different budgets; one documented rule avoids reading them | Caller holds the opaque token and sends it back unchanged. Neutral trigger (`length`). Works on all three routes | OR-documented | O59 |
| Error types | Anthropic, OpenAI, all | Anthropic: `error.type` strings with statuses 400 to 529. OpenAI: `error.code` strings | `error_type` with 27 values; location differs per route: `error.metadata.error_type` (Chat), `error.error_type` (Messages), top-level `error_type` (Responses). Vendor code in `error.metadata.provider_code` "when available", verbatim payload in `metadata.raw` | Vendor status replaced by a canonical status. 500 masks message and vendor code. Spend-cap 429 and ordinary 429 share `rate_limit_exceeded`. Responses and Messages native codes are shared buckets | No state. Neutral class plus optional native code. Caller reads a different path on each route | OR-documented (the per-vendor correspondence in O73 is an inference) | O33, O35, O36, O38, O39, O40, O41, O73 |
| Error after HTTP 200 | Anthropic (`event: error`), OpenAI (`response.failed`), all | Vendors report failures inside an already-open stream | Chat: a `chat.completion.chunk` with top-level `error` and `choices[0].finish_reason: "error"`; non-streaming: 200 body with `error`, or `error` on the choice beside partial content. Responses: `response.failed`. Messages: SSE `error` event | Whether `[DONE]` follows is not stated. One guide page shows a bare `{"error": ...}` chunk | No state. Caller must treat any chunk or 200 body with `error` as failure. Runvil's reader must not class it as transport failure | OR-documented | O42, O43, O44, O45, O74, O75 |
| Failover across providers | all multi-host models | A vendor does not fail over to another host | Automatic retry on another provider until the first token reaches the caller, also after the 200 is sent; `provider.allow_fallbacks: false` turns it off; trail in `openrouter_metadata.attempts` and `provider_responses` | After partial output there is no recovery rule | Gateway holds all state. Caller sees nothing unless it opts in | OR-documented | O5, O47, O48 |
| Context overflow signal | Anthropic (message text), OpenAI (`context_length_exceeded`), all | Each vendor signals prompt overflow differently | `error_type: "context_length_exceeded"` (400). Output-side overflow becomes `finish_reason: "length"` on Chat and `status: "incomplete"` on Responses. With `models`, overflow can trigger a model fallback instead | Message text differs per backend. "Context full" and "own cap reached" both show as `length`. Fallback hides the overflow | No state. Neutral typed value exists for input overflow | OR-documented (the message text is from client code, O81) | O15, O63, O64, O76, O81 |
| Server-side prompt trimming | OpenRouter feature (vendors: none equivalent on Chat) | Vendors reject an over-long prompt | `plugins: [{"id": "context-compression"}]`, engine `middle-out`; on by default for endpoints of 8,192 tokens or less; off for deferred-tool requests; reported only in `openrouter_metadata.pipeline` | Middle messages are dropped without a signal unless the caller opts in to metadata | Gateway mutates the prompt. Caller loses knowledge of what the model saw | OR-documented | O60, O61, O62 |
| Rate-limit signals | all | Vendor 429 plus vendor headers | 429 with `error_type: "rate_limit_exceeded"`; `Retry-After` when every attempted provider gave a hint; `X-RateLimit-*` only on OpenRouter's own free-model and key-management 429. Limits are per account, not per key | Vendor rate-limit headers are not described as forwarded | No state. Neutral | OR-documented (that vendor headers are dropped is an inference from silence) | O30, O31, O32 |
| Credit and budget exhaustion | OpenRouter billing (vendors: 402 or 429 variants) | Vendors mix billing failures into 402, 429, 400 | 402 with `error_type: "payment_required"` and `error.metadata.limit_source` (`openrouter_in_flight_budget`, `openrouter_key_limit`, `openrouter_credits`) and `reason`; 403 for guardrail, workspace, and key budgets | A vendor's own quota error under BYOK is not described | No state. Caller branches on `limit_source`; only the in-flight case is retryable | OR-documented | O37, O48 |
| Choice of host for one model | Anthropic direct, Amazon Bedrock, Google Vertex; OpenAI, Azure; open-weight hosts | Each host has its own endpoint, price, limits | `provider` object: `order`, `only`, `ignore`, `allow_fallbacks`, `sort`, `quantizations`, `max_price`, `preferred_min_throughput`, `preferred_max_latency`, `require_parameters`; slug shape `provider[/variant][/region][/tier]` | Default is a weighted random pick, so two requests can hit hosts with different behavior | Gateway holds routing state. Hard filters (`only`, `ignore`, `max_price`, `zdr`, `data_collection`, `quantizations`) fail the request; preferences only reorder | OR-documented | O1 to O12 |
| Vendor beta headers | Anthropic | `anthropic-beta` header turns on beta features | `x-anthropic-beta` request header, forwarded through a per-provider allowlist; documented values `interleaved-thinking-2025-05-14`, `structured-outputs-2025-11-13`; some features turned on by OpenRouter itself | Values off the list are dropped silently | Caller must know the vendor and its header values | OR-documented | O13 |
| Vendor-native parameters | all | Each vendor has parameters with no neutral name | `provider.options[<provider slug>]`, forwarded only to the matched provider, filtered by a per-endpoint allowlist; unknown keys dropped on chat | The chat allowlist is not published | Caller writes vendor-specific names and must know the provider slug | OR-documented | O3 |
| Service tier | OpenAI (`flex`, `priority`), others (`fast`, `turbo`, `highspeed`, `prime`) | Each provider names its tiers differently | Neutral `service_tier` (`auto`, `default`, `fast`, `flex`, `priority`, `scale`, `ultrafast`); suffixes `:nitro` and `:floor` admit tier endpoints; response `service_tier` reports the tier served | Provider label spellings appear only in endpoint slugs | No state. Neutral values translated per provider | OR-documented | O12, O20, O72 |
| Cancellation and billing | 24 providers stop billing on abort, 30 do not | Depends on the provider | Abort the connection of a streaming request; `cancelled` on the generation record | Same model costs differently on abort depending on the host chosen (Anthropic stops, Bedrock and Google do not) | No field. Caller cannot know the outcome without knowing the provider | OR-documented | O51 |
| Stream keep-alive | Anthropic (`ping` events), others silent | Vendors differ in how they keep a slow stream open | SSE comment `: OPENROUTER PROCESSING`, before and between data chunks; no documented interval | Vendor ping events are not described as forwarded on Chat | No state. Caller's parser must skip comment lines | OR-documented (position between chunks is from recordings, O79) | O49, O79, O82 |
| Request timeouts | all | Vendors publish their own limits | Statuses 408, 504 (`timeout`), 524 in the spec; a silent provider is cancelled and another is tried | No number is published | Gateway decides. Caller cannot set it | OR-inferred (no number found by text search; the statuses are documented) | O50 |
| Data retention and training policy | all | Each provider has its own retention and training terms | `provider.data_collection` (`allow`, `deny`), `provider.zdr`, account and guardrail settings; three tiers (ZDR, Safety Retention, Retention and Training) | Tools and plugins are outside ZDR enforcement | Gateway holds the per-endpoint policy table. Neutral values | OR-documented | O8, O28, O29 |
| Data residency | Anthropic (`inference_geo`), OpenAI regional hosts, AWS and Google regions | Region is a vendor-specific setting | Chosen by hostname: `eu.openrouter.ai`, `us.openrouter.ai`; region segment in provider slugs; per-key region declaration for BYOK | No request field | Caller picks a base URL, not a field | OR-documented | O12, O29 |
| BYOK credentials | Azure, Amazon Bedrock, Google Vertex, others | Azure resource or deployment, AWS access key or Bedrock key with region, Google service account JSON | Stored per workspace; tried before shared capacity unless `order` is set; fallback to shared capacity by default; 5 percent fee above 25,000 dollars per month; `usage.is_byok` in the response | Vendor quota errors under BYOK fall through to the next key or to shared capacity | Gateway holds the keys. No request field | OR-documented | O25, O26, O27, O28 |
| Model fallback list | Anthropic (server-side `fallbacks`), others none | Anthropic has its own fallback feature on Messages | Chat and Responses: `models` array. Messages: `fallbacks` (at most 3 entries, `model` only), handled by OpenRouter, not by Anthropic. Billed and named by the model that ran | Per-attempt overrides rejected with 400 | Gateway holds state. Caller must read `model` in the response to know what ran | OR-documented | O15, O16, O17, O18 |
| Reasoning effort chosen by the router | OpenAI, Anthropic, Google (effort levels differ) | Each vendor has its own effort or budget control | Router models (`openrouter/auto`, `openrouter/auto-code`, `typesafe/jev-router`, `<author>/auto`) pick model and effort; the caller's `reasoning.effort` is a hint; `reasoning.max_tokens` returns 400 on Jev | The caller's effort setting is not guaranteed | Gateway decides per request; can switch model between turns of one session | OR-documented | O21, O22 |
| Model variants as suffixes | OpenRouter naming (`:thinking` was for Anthropic-style reasoning endpoints) | Vendors publish separate model ids or parameters | `:free`, `:batch` (catalog); `:nitro`, `:floor`, `:exacto` (routing); `:thinking`, `:extended`, `:online` deprecated | Catalog and routing suffixes resolve differently for metadata lookups | No state. A suffix stands for a request setting, so a neutral field can replace it | OR-documented | O19, O20, O23 |
| Named server-side configuration | OpenRouter feature | Vendors have stored prompts (not surveyed here) | `@preset/<slug>` as the model, as a `preset` field, or appended to a model; request fields win by shallow merge; tools unioned | `preset` is not in the spec | Server holds the configuration | OR-documented | O24 |
| Inspecting the translated upstream body | all | Not applicable | `debug: {"echo_upstream_body": true}`, streaming only; first chunk with `choices: []` and `debug.echo_upstream_body` on Chat, `response.debug` event on Responses; one per provider attempt | Best-effort redaction; not for production; Messages route not named | Shows the vendor-native body, so it documents the translation | OR-documented | O65, O66, O67, O68 |
| Required output cap | Anthropic (`max_tokens` required) | Request fails without `max_tokens` | OpenRouter fills `max_tokens: 65536` when the caller omits it (seen in the echoed body; cookbook states "default output budget of 65,536 tokens") | The caller's absence of a cap becomes a concrete cap | Gateway default. No caller action | OR-documented | O67 |
| System prompt placement | Anthropic (top-level `system` blocks) | System text is not a message | A `system` role message is lifted into `system: [{"type": "text", ...}]` (seen in the echoed body) | Nothing in the documented example | Gateway translation | OR-documented | O67 |
| End-user identity | Anthropic (`metadata.user_id`), OpenAI (safety identifier) | Vendors accept an end-user id for abuse handling | `user` request field; OpenRouter "folds it into the hashed identity sent upstream and never forwards it raw"; echoed body shows `metadata.user_id` redacted | The raw id is not sent | Neutral field. Gateway hashes | OR-documented | O67, O69 |
| Session affinity for prompt caches | all caching providers | Cache hits need the same provider endpoint | `session_id` (body or `x-session-id` header, 256 characters), else a hash of the first messages, else `prompt_cache_key`; 60 minutes of inactivity | Not used when `provider.order` is set | Gateway holds the session-to-provider map. Neutral field | OR-documented | O4, O69 |
| Which provider and upstream id served | all | Vendors return their own request ids | `model` in the response; `provider` on chat chunks (guide, not spec); `X-Generation-Id` header; `GET /api/v1/generation` with `provider_name`, `upstream_id`, `provider_responses`; opt-in `openrouter_metadata` via `X-OpenRouter-Metadata: enabled` | Vendor request id only through a second authenticated request | Caller needs the provider name to interpret `native_finish_reason` and cancellation billing | OR-documented | O70, O71 |
| Encrypted reasoning tied to an endpoint | OpenAI via Azure (sample), others | The blob is decryptable only by the vendor that issued it | Responses reasoning item has `format` (for example `azure-openai-responses-v1`) and an `encryptedContent` ending in a base64 JSON suffix `{"endpoint_slug": ...}` | The blob the caller stores is OpenRouter's wrapper, not the vendor's bytes | Caller round-trips it unchanged. Gateway reads its own tag to route the replay | OR-inferred (sample in OpenRouter's documentation) | O14 |
| Tool argument streaming and strictness | Anthropic | `eager_input_streaming` per tool; `strict` needs a beta header | OpenRouter sets `eager_input_streaming: true` on every streamed request with tools; strips `strict` unless `x-anthropic-beta: structured-outputs-2025-11-13` is sent | `strict` silently dropped without the header | Gateway default for one, caller vendor knowledge for the other | OR-documented | O13 |

## Evidence

Each item gives a claim, the exact quote or quotes, the source key, and the basis label. Quotes that are split into fragments are split to keep them word for word.

### A. Provider preferences (the `provider` object)

**O1. The `provider` object is accepted on Chat Completions and on the Responses route, and its documented fields and defaults are listed in one table.**

- Quote: "You can customize how your requests are routed using the `provider` object in the request body for the [Responses API] and [Chat Completions]."
- Quote (the table, field / type / default / description, links removed):
  - "`order` | string\[] | - | List of provider slugs to try in order (e.g. `["anthropic", "openai"]`)."
  - "`allow_fallbacks` | boolean | `true` | Whether to allow backup providers when the primary is unavailable."
  - "`require_parameters` | boolean | `false` | Only use providers that support all parameters in your request."
  - "`data_collection` | "allow" \| "deny" | "allow" | Control whether to use providers that may store data."
  - "`zdr` | boolean | - | Restrict routing to only ZDR (Zero Data Retention) endpoints."
  - "`enforce_distillable_text` | boolean | - | Restrict routing to only models that allow text distillation."
  - "`only` | string\[] | - | List of provider slugs to allow for this request."
  - "`ignore` | string\[] | - | List of provider slugs to skip for this request."
  - "`quantizations` | string\[] | - | List of quantization levels to filter by (e.g. `["int4", "int8"]`)."
  - "`sort` | string \| object | - | Sort providers by price, throughput, or latency. Can be a string (e.g. `"price"`) or an object with `by` and `partition` fields."
  - "`preferred_min_throughput` | number \| object | - | Preferred minimum throughput (tokens/sec). Can be a number or an object with percentile cutoffs (p50, p75, p90, p99)."
  - "`preferred_max_latency` | number \| object | - | Preferred maximum latency (seconds). Can be a number or an object with percentile cutoffs (p50, p75, p90, p99)."
  - "`max_price` | object | - | The maximum pricing you want to pay for this request."
- Source: O-s1. Basis: OR-documented.

**O2. The OpenAPI schema `ProviderPreferences` is closed (`additionalProperties: false`) and holds two things the guide's table leaves out: an `options` object and an `audio` key inside `max_price`. The sort enum has a fourth value, `exacto`.**

- Quote (`ProviderPreferences`): `"additionalProperties": false`, `"description": "When multiple model providers are available, optionally indicate your routing preference."`
- Quote (`ProviderPreferences.max_price.properties`): `audio` "Maximum price in USD per audio unit", `completion` "Maximum price in USD per million completion tokens", `image` "Maximum price in USD per image", `prompt` "Maximum price in USD per million prompt tokens", `request` "Maximum price in USD per request". Each is `"type": "string"`.
- Quote (`ProviderSort`): `"enum": ["price", "throughput", "latency", "exacto"]`.
- Quote (`ProviderPreferences.sort.description`): "The sorting strategy to use for this request, if "order" is not specified. When set, no load balancing is performed."
- Source: O-s2. Basis: OR-documented.

**O3. `provider.options` is a per-provider pass-through object keyed by provider slug: the one place where a caller writes vendor-native parameter names. Only the matched provider's options are forwarded, each endpoint has an allowlist of pass-through keys that the provider declares, and on the chat routes unknown keys are dropped without an error. It entered the `ProviderPreferences` schema on 2026-09-24. The guides document it for speech, image, video, and extraction models, and the provider routing guide for chat does not mention it.**

- Quote (`ProviderOptions.description`): "Provider-specific options keyed by provider slug. The provider segment is case-insensitive; an endpoint tag after `/` must match the endpoint slug exactly. Only options for the matched provider are forwarded; the rest are ignored. Unrecognized keys are silently dropped."
- Quote (`ProviderOptions.example`): `{"openai": {"max_tokens": 1000}}`. Each provider key is typed `{"additionalProperties": {}, "type": "object"}`, so the spec does not say which keys each provider accepts.
- Quote (changelog, under the heading "September 24, 2026"): "`ProviderPreferences`: new property `options`".
- Quote (speech-to-text guide): "Pass provider-specific parameters through `provider.options`, keyed by the provider slug from the endpoints API. Only the options for the provider that serves the request are forwarded, and they are sent under the provider's own field names, so use the names and shapes from that provider's transcription API reference. Parameters that OpenRouter normalizes across providers (`language`, `prompt`, `temperature`, `response_format`, `timestamp_granularities`) stay at the top level of the request."
- Quote (provider integration guide, addressed to providers): "OpenRouter merges the parameter names from every scope into the endpoint's passthrough allowlist, which controls the `provider.options` keys forwarded to your API." The example declaration is `"passthrough_parameters": {"service_tier": {"type": "enum", "values": ["standard", "priority"]}, "safety_settings": {"type": "object"}}`.
- Quote (extraction models guide, a stricter rule than the spec's): "Provider-specific knobs go under `provider.options.<provider slug>`, and each model accepts only its own keys; anything else fails with a `400` naming the accepted keys."
- A text search of the mirrored provider routing guide, parameters reference, tool calling guide, and reasoning guide for `provider.options` found no mention (searched 2026-11-22), so which keys a chat endpoint forwards is visible only per endpoint through the endpoints API.
- Source: O-s2, O-s27, O-s34, O-s35, O-s36. Basis: OR-documented.

**O4. Default routing is a weighted random pick over providers, weighted by measured effective price, throughput, time to first token, 429 rate, reliability, and tool-call quality. It is bypassed by `sort`, `order`, the `:floor` and `:nitro` suffixes, and BYOK.**

- Quote: "Providers close to the best share most of the traffic, and each request picks one at random, weighted by those shares."
- Quote: "If a request fails, the retry is picked the same way from the providers that are left. Follow-up requests in a conversation stay with the same provider while its [prompt cache] is warm."
- Quote: "Default routing applies whenever you leave the choice of provider to OpenRouter. It is bypassed when you set `sort` (or a default provider sort in your [account settings]), use the `:floor` or `:nitro` shortcuts, set `order`, or use your own provider key. If you set `only` or `ignore` and two or more providers remain, default routing picks among them."
- Quote (provider-facing page): "**Breaker.** If a degradation signal like reliability, quality or 429s is far worse than your peers', routing pauses your traffic until it recovers."
- Source: O-s1, O-s3. Basis: OR-documented.

**O5. `order` sets which providers are tried first. With fallbacks left on, OpenRouter moves on to other providers after the list. `allow_fallbacks: false` restricts the request to the list (or, without `order`, to the single top-ranked provider) and returns the upstream error.**

- Quote: "OpenRouter will try them one at a time and proceed to other providers if none are operational. If you don't want to allow any other public providers, you should [disable fallbacks] as well."
- Quote: "To stop OpenRouter from falling back to other providers, you can disable fallbacks. Without `order`, the request is served only by the top-ranked provider."
- Quote (spec, `allow_fallbacks.description`): "- true: (default) when the primary provider (or your custom providers in "order") is unavailable, use the next best provider." and, on the next line of the same description, "- false: use only the primary/custom provider, and return the upstream error if it's unavailable."
- Quote (spec, `order.description`): "If no providers are available, the request will fail with an error message."
- Source: O-s1, O-s2. Basis: OR-documented.

**O6. `only` and `ignore` are hard filters. Both merge with account-wide lists, and a request left with zero providers fails with HTTP 404 and a message that names the providers and the preference that excluded them.**

- Quote: "your account-wide allowed providers act as the ceiling, and the request's `only` list narrows within it. If no provider satisfies both, the request fails with a 404."
- Quote: "when you ignore providers for a specific request, the list of ignored providers is merged with your account-wide ignored providers."
- Quote (error example): `{"error": {"code": 404, "message": "No allowed providers are available for the selected model. Providers serving openai/gpt-4o-mini: openai, but your request's provider.only preference permits only: azure."}, ...}`
- Source: O-s1, O-s15. Basis: OR-documented.

**O7. By default a provider that does not support a parameter still gets the request and ignores the parameter. `require_parameters: true` turns that into a hard filter. Three parameters act as a soft preference even when it is false. (Bears on LC3.)**

- Quote: "With the default routing strategy, providers that don't support all the [LLM parameters] specified in your request can still receive the request, but will ignore unknown parameters. When you set `require_parameters` to `true`, the request won't even be routed to that provider."
- Quote (spec): "If this setting is omitted or set to false, then providers will receive only the parameters they support, and ignore the rest."
- Quote: "Even when `require_parameters` is `false`, a small set of parameters is used as a soft preference when choosing between providers of the same model: `tools`, the response format (`text.format` on the Responses API, `response_format` on Chat Completions, including structured outputs), and `verbosity`. If some of a model's providers support one of these parameters and others don't, the request is only routed to the supporting providers. If none of a model's providers support the parameter, the request is still routed to that model and the parameter is ignored"
- Quote: "if you set `max_output_tokens` (or `max_tokens` on Chat Completions), then OpenRouter will only route to providers that support a response of that length."
- Source: O-s1, O-s2. Basis: OR-documented.

**O8. `data_collection`, `zdr`, and `enforce_distillable_text` are hard filters on which endpoints may serve the request.**

- Quote: "`allow`: (default) allow providers which store user data non-transiently and may train on it" and "`deny`: use only providers which do not collect user data".
- Quote (spec, `data_collection.description`): "If no available model provider meets the requirement, your request will return an error."
- Quote: "When `zdr` is set to `true`, the request will only be routed to ZDR endpoints. Safety Retention endpoints (which may retain prompts and responses from any request, only to prevent abuse, and don't train on them) are also eligible unless your privacy settings or guardrail are set to **ZDR only**."
- Quote: "The per-request `zdr` parameter operates as an "OR" with your account-wide and guardrail ZDR settings". "The request-level parameter can only ensure ZDR is enabled, not override account-wide or guardrail enforcement."
- Quote: "When `enforce_distillable_text` is set to `true`, the request will only be routed to models where the author has explicitly enabled text distillation."
- Source: O-s1, O-s2. Basis: OR-documented.

**O9. `quantizations` filters by quantization level. The value list is the same in the guide and in the spec.**

- Quote (spec, `Quantization.enum`): `["int4", "int8", "fp4", "mxfp4", "nvfp4", "fp6", "fp8", "mxfp8", "fp16", "bf16", "fp32", "unknown"]`.
- Quote: "To filter providers by quantization level, specify the `quantizations` field in the `provider` parameter".
- Source: O-s1, O-s2. Basis: OR-documented.

**O10. `sort` takes `"price"`, `"throughput"`, or `"latency"` (and `"exacto"` in the spec), or an object `{by, partition}`. `partition` is `"model"` by default and `"none"` sorts endpoints across all models in the `models` list. The old top-level `route` field is a deprecated alias for the partition.**

- Quote: "`sort.partition` | string | `"model"` | How to group endpoints for sorting: `"model"` (default) or `"none"`."
- Quote: "By default, when you specify multiple models (fallbacks), OpenRouter groups endpoints by model before sorting. This means the primary model's endpoints are always tried first, regardless of their performance characteristics. Setting `partition` to `"none"` removes this grouping, allowing endpoints to be sorted globally across all models."
- Quote (spec, `DeprecatedRoute`): "**DEPRECATED** Use providers.sort.partition instead. Backwards-compatible alias for providers.sort.partition. Accepts legacy values: "fallback" (maps to "model"), "sort" (maps to "none")."
- Source: O-s1, O-s2. Basis: OR-documented.

**O11. LC8 holds. The performance preferences only reorder. `max_price` is a hard ceiling that stops the request.**

- Quote: "Endpoints that don't meet these thresholds are deprioritized (moved to the end of the list) rather than excluded entirely."
- Quote: "`preferred_max_latency` and `preferred_min_throughput` do *not* guarantee you will get a provider or model with this performance level. However, providers and models that hit your thresholds will be preferred. Specifying these preferences should therefore never prevent your request from being executed. This is different than `max_price`, which will prevent your request from running if the price is not available."
- Quote: "OpenRouter tracks latency and throughput metrics for each model and provider using percentile statistics calculated over a rolling 5-minute window." The percentiles are p50, p75, p90, p99, and "When you specify multiple percentile cutoffs, all specified cutoffs must be met".
- Quote (spec, `PreferredMaxLatency.description`): "Can be a number (applies to p50) or an object with percentile-specific cutoffs. Endpoints above the threshold(s) may still be used, but are deprioritized in routing. When using fallback models, this may cause a fallback model to be used instead of the primary model if it meets the threshold."
- Quote: "the value `{"prompt": 1, "completion": 2}` will route to any provider with a price of `<= $1/m` prompt tokens, and `<= $2/m` completion tokens or less."
- Source: O-s1, O-s2. Basis: OR-documented.

**O12. A provider slug has the shape `provider[/variant][/region][/tier]`. A base slug matches every variant and region but never a service-tier endpoint.**

- Quote: "Every endpoint slug has the shape `provider[/variant][/region][/tier]`, with each segment present only when the endpoint needs it: the variant (a quantization such as `fp4`, or a provider-specific label such as `zdr`), then the region, then the [service tier] in the provider's spelling (`priority`, `fast`, `turbo`, `flex`), always last."
- Quote: "A base slug does not by itself opt into a [service tier endpoint] (e.g. `openai/fast`, `google-vertex/flex`); select one with the `service_tier` parameter or its tier-suffixed slug."
- Source: O-s1. Basis: OR-documented.

**O13. One vendor header passes through: `x-anthropic-beta`, filtered by a per-provider allowlist. OpenRouter also turns on Anthropic fine-grained tool streaming by itself, and it strips `strict` from tools unless the caller sends the beta header.**

- Quote: "OpenRouter keeps an allowlist of other `x-anthropic-beta` values for each provider and forwards only the values on the list for the provider that serves the request. Providers that serve Anthropic's Messages API directly have the longest list, and Google Vertex and Amazon Bedrock receive only the values they accept. OpenRouter drops every other value, and some values it recognizes are withheld on purpose."
- Quote: "for every streamed request (`stream: true`) that includes user-defined tools, OpenRouter sets `eager_input_streaming: true` on each tool so Anthropic emits tool arguments as a sequence of incremental chunks rather than buffering them."
- Quote: "For **strict tool use** (`strict: true` on tools), you must explicitly pass the `structured-outputs-2025-11-13` header. Without this header, OpenRouter will strip the `strict` field and route normally."
- The two header values the page lists are `interleaved-thinking-2025-05-14` and `structured-outputs-2025-11-13`.
- Source: O-s1. Basis: OR-documented.

**O14. On the Responses route, the encrypted reasoning item that OpenRouter returns ends with a dot and a base64 JSON suffix that names the endpoint that produced it, and the item has a `format` tag naming the upstream protocol.**

- The sample outputs on the provider routing page show reasoning items whose `encryptedContent` ends in `.eyJlbmRwb2ludF9zbHVnIjoib3BlbmFpL2dwdC01LW1pbmktMjAyNS0wOC0wN3xhenVyZSJ9`. Decoded with base64 on 2026-11-22 this is `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|azure"}`. The same items have `"format": "azure-openai-responses-v1"`.
- What this shows: OpenRouter wraps the vendor's encrypted reasoning blob with its own routing tag, so the blob that the caller stores and sends back is not byte-identical to the vendor's blob, and it records which provider endpoint can decrypt it.
- Source: O-s1. Basis: OR-inferred (inferred from a sample response printed in OpenRouter's own documentation; the documentation does not describe the suffix). Confidence: high that the suffix exists in real responses, medium on its purpose.

### B. Model fallbacks (the `models` array)

**O15. `models` is an array of model ids in priority order. Any error from the first model can trigger the next one, and the documented triggers include context-length validation errors and moderation refusals.**

- Quote: "The `models` parameter lets you automatically try other models if the primary model's providers are down, rate-limited, or refuse to reply due to content moderation."
- Quote: "If the model you selected returns an error, OpenRouter will try to use the fallback model instead. If the fallback model is down or returns an error, OpenRouter will return that error."
- Quote: "By default, any error can trigger the use of a fallback model, including:" followed by the list "Context length validation errors", "Moderation flags for filtered models", "Rate-limiting", "Downtime".
- Quote (spec, `ChatRequest.models` is `ChatModelNames`): "Models to use for completion", an array of model names. On the OpenAI SDK the field goes in `extra_body`.
- Source: O-s4, O-s2. Basis: OR-documented.

**O16. The caller is billed for the model that actually ran, and that model is named in the response `model` field. A free model followed by a paid model can produce a paid request.**

- Quote: "Requests are priced using the model that was ultimately used, which will be returned in the `model` attribute of the response body."
- Quote: "If a `:free` model is followed by a paid model in `models` and OpenRouter falls back to the paid model, you're billed at the paid model's price. To keep a request free, list only `:free` models."
- Source: O-s4. Basis: OR-documented.

**O17. Provider fallback runs inside each model first, and model fallback runs after the first model's providers are exhausted. `sort.partition: "none"` removes that ordering.**

- Quote (limits page): "[fallback routing] retries other providers for the same model automatically before the error reaches you. You can also specify [fallback models] to try a different model when all providers for the first are exhausted."
- Quote (latency page): "By default, OpenRouter tries all endpoints of Model A before attempting Model B, even if Model B is currently idle and faster."
- Quote (provider routing page): "This means the primary model's endpoints are always tried first, regardless of their performance characteristics."
- Quote (a refusal that the model returns as normal output still lets fallback run): "Fallback to another model still runs before any output is returned when routing allows it."
- Source: O-s21, O-s30, O-s1, O-s22. Basis: OR-documented.

**O18. The Messages route takes a different field name for the same feature: `fallbacks`, an array of at most 3 entries that each hold only `model`. It maps to `models` routing and does not use Anthropic's own server-side fallback.**

- Quote: "The Anthropic Messages API endpoint (`/api/v1/messages`) accepts the `fallbacks` parameter, matching the shape used by Anthropic's SDKs. Each entry names a fallback model to try in order, and the list maps to OpenRouter's `models` routing, so fallbacks trigger on the same errors listed above (rate limits, downtime, moderation refusals), not only on refusals."
- Quote: "OpenRouter handles this fallback routing itself. The `fallbacks` parameter does not use Anthropic's server-side fallback feature."
- Quote: "Each `fallbacks` entry accepts only the `model` field. Per-attempt overrides such as `max_tokens`, `thinking`, `speed`, or `output_config` are rejected with a 400 error." "`fallbacks` cannot be combined with the `models` parameter; sending both returns a 400 error." "`fallbacks` accepts at most 3 entries; longer lists return a 400 error."
- Source: O-s4. Basis: OR-documented.

### C. Variants, routers, presets

**O19. A suffix after a colon is either a catalog variant (its own entry in the model list, with its own price and context length) or a routing variant (accepted on any model id, never listed). A client that looks up model metadata must keep catalog suffixes and drop routing suffixes.**

- Quote: "**Catalog variants** are separate entries in `GET /api/v1/models` with their own metadata. Only the models that list them support them." "**Routing variants** are accepted on any model ID at request time. They are not listed in `GET /api/v1/models` and change only how the request is routed. The model's metadata is the base model's."
- Quote: "Keep the catalog variant if one is present (`:free`, `:batch`, `:thinking`, `:extended`). Drop every routing variant (`:nitro`, `:floor`, `:exacto`, `:online`)."
- Quote: "Do not strip every suffix. Stripping `:free` resolves to the paid entry, which can report a larger context window and non-zero pricing, neither of which describes the free entry that serves the request."
- Quote: "Sending a catalog suffix on a model that has no such entry does not fall back to the base model. The single-model lookup returns `404`, the endpoint lookup returns `200` with an empty `endpoints` array, and inference requests fail because there is no endpoint to route to."
- Quote: "Suffixes can be combined in any order, separated by colons. A model ID carries at most one catalog variant and any number of routing variants." "When more than one sorting variant is present (`:nitro`, `:floor`, `:exacto`), the last one in the ID determines the sort."
- Source: O-s5. Basis: OR-documented.

**O20. What each suffix stands for, and which are deprecated today.**

| Suffix | Kind | Status | Request setting it stands for | Quote |
| - | - | - | - | - |
| `:free` | catalog | Active | A separate free entry with its own limits and endpoints | "A free-tier version of the model with its own rate limits and endpoints." |
| `:batch` | catalog | Active | The batch-priced entry used by the Batch API | "The Batch API takes the base model slug and resolves the `:batch` entry itself." |
| `:thinking` | catalog | Deprecated | Reasoning on by default; replaced by the `reasoning` parameter | "The `:thinking` variant is deprecated and is being removed. Use the [`reasoning` parameter] instead" |
| `:extended` | catalog | Deprecated | A larger context window; no model offers it | "The `:extended` variant is deprecated and no model on OpenRouter currently offers it. Requests to a `model:extended` slug have no endpoints to route to and will fail." |
| `:nitro` | routing | Active | `provider.sort: "throughput"` plus admission of priority-tier endpoints | "it is a superset of setting `provider.sort` to `"throughput"` (which only sorts)" |
| `:floor` | routing | Active | `provider.sort: "price"` plus admission of flex-tier endpoints | "it is a superset of setting `provider.sort` to `"price"` (which only sorts)" |
| `:exacto` | routing | Active | Provider sort by tool-calling quality signals | "This is a shortcut for setting the provider sort to Exacto on that model." |
| `:online` | routing | Deprecated | The `web` plugin; replaced by the `openrouter:web_search` server tool | "This is a shortcut for using the `web` plugin, and is exactly equivalent to:" `{"model": "openrouter/auto", "plugins": [{ "id": "web" }]}` |

- Note on `:nitro` billing: "a `:nitro` request served by a priority endpoint is charged that endpoint's priority pricing."
- Note on the `:online` example: the documentation's "exactly equivalent" body names `openrouter/auto` as the model, not the model the suffix was put on. This reads as a documentation slip, recorded under Conflicts.
- Source: O-s5, O-s6, O-s7, O-s8, O-s9, O-s10, O-s11, O-s12, O-s1. Basis: OR-documented.

**O21. The router model slugs, and the fact that some choose a reasoning effort for the caller.**

- Quote (overview table): `openrouter/auto` (Auto Router), `openrouter/auto-code` (Auto Code), `openrouter/free` (Free Models Router), `openrouter/pareto-code` (Pareto Router), `openrouter/fusion` (Fusion Router), `nvidia/switchyard` (Switchyard), `typesafe/jev-router` (Jev Router). The Auto Router page adds `openrouter/auto-beta`, and the Jev page adds author-scoped aliases: "`openai/auto`, `anthropic/auto`, and other `<author>/auto` models run Jev with that author's models."
- Quote: "A fast decision model rates each request, and a routing policy weighs that rating against live prices, cache status, and benchmark results to pick a model and reasoning effort."
- Quote (Auto Router): "It picks a model and a reasoning effort for each request, from the models your guardrails, zero data retention, provider restrictions, and BYOK keys allow."
- Quote (Jev Router): "The router normally chooses reasoning effort. You can supply the standard top-level `reasoning.effort` field as a hint, independently of `cost_tier`". "The router can select a different effort or adapt it across turns, so a hint is not a guarantee of the setting sent to the final model. `reasoning.max_tokens` and `reasoning.mode: "pro"` are not supported and return `400`."
- Quote: "The response's `model` field tells you which model served it." "Routers don't add a routing fee. You pay the standard price of the model that serves the request, and the context length is that model's."
- Source: O-s13, O-s14, O-s38. Basis: OR-documented.

**O22. Router settings ride in the `plugins` array, a router can switch models between turns of one conversation, and a restriction that leaves zero models fails with 404.**

- Quote: `"plugins": [{"id": "auto-router", "allowed_models": ["anthropic/*", "openai/gpt-5.1"]}]`, with `excluded_models` and `cost_tier` (`low`, `medium`, `high`) as the other settings.
- Quote: "Each slug only reads settings sent under its own plugin id. Settings sent under the other slug's plugin id are accepted but silently ignored".
- Quote: "Unlike a fixed model slug, the Auto Router can pick a different model on every turn." "OpenRouter recognizes the conversation from an explicit `session_id`, or from a fingerprint of your messages if you don't send one."
- Quote: "If your restrictions leave no eligible model, the request fails with `404`, and the error names the fields to change, such as `widen allowed_models` or `remove excluded_models`."
- Quote (limits of the router): "The router requires conversational input", "The router is available only in the global data region."
- Source: O-s14. Basis: OR-documented.

**O23. `~author/family-latest` is a model alias that resolves to the newest model of a family at request time.**

- Quote: "`~author/family-latest` slugs always resolve to the newest concrete model in a given family, so you can ship code against a stable alias and pick up new releases without redeploying."
- Source: O-s31. Basis: OR-documented.

**O24. A preset is a named, server-stored request configuration. It is referenced in three ways, and request fields win over preset fields by a shallow merge.**

- Quote: the three forms are `"model": "@preset/email-copywriter"`, `"model": "openai/gpt-4", "preset": "@preset/email-copywriter"`, and `"model": "openai/gpt-4@preset/email-copywriter"`. "A bare slug without the `@preset/` prefix is ignored."
- Quote: "If you provide parameters in the request, they take priority over the preset's values. The two are shallow-merged, meaning request-level fields override matching preset fields, but preset fields not present in the request are preserved."
- Quote (what a preset can hold): "Provider routing preferences", "Model selection (specific model or array of models with fallbacks)", "System prompts", "Generation parameters", "Provider inclusion/exclusion rules", "Tools".
- Quote (tools merge): "the two arrays are unioned, and a request tool overrides the preset's tool of the same identity".
- Quote: "when addressing a preset through the API, the latest version is always used."
- The spec's `ChatRequest` and `ResponsesRequest` schemas have no `preset` property (checked 2026-11-22), so the `preset` body field is documented in the guide only.
- Source: O-s16, O-s2. Basis: OR-documented.

### D. BYOK, data policy, ZDR

**O25. A BYOK key is stored encrypted in the workspace and used for every request routed to that provider. It changes which credential goes upstream and which rate limits apply. By default it is tried before OpenRouter's shared capacity and the request falls back to shared capacity when the key fails.**

- Quote: "Your provider keys are securely encrypted and used for all requests routed through the specified provider."
- Quote: "When you use OpenRouter credits, your rate limits for each provider are managed by OpenRouter." "Using provider keys enables direct control over rate limits and costs via your provider account."
- Quote: "**Prioritized**, attempted in order, before falling back to OpenRouter endpoints, unless the request sets [`order`]." "**Fallback**, tried only after OpenRouter endpoints have been attempted, in order."
- Quote: "By default, if all keys in both sections encounter a rate limit or failure, OpenRouter will fall back to using shared OpenRouter endpoints."
- Quote (the three "Shared capacity fallback" levels): "**Use shared capacity** (default)", "**Never use shared capacity for models this key applies to**", "**Never use shared capacity for any model on this provider**". The management API names are `is_required` and `is_byok_only`.
- Quote: "When a request sets no `order`, OpenRouter prioritizes BYOK endpoints first".
- The response says which path served the request: `usage.is_byok` ("Whether request used Bring Your Own Key") and, with router metadata on, `openrouter_metadata.is_byok`.
- Source: O-s17, O-s26, O-s15. Basis: OR-documented.

**O26. LC15 holds, with the allowance now stated in dollars. Inference has no markup, buying credits costs 5.5 percent with a 0.80 dollar minimum, and BYOK costs 5 percent of the normal price above a monthly allowance of 25,000 dollars of list-price inference on the pay-as-you-go plan.**

- Quote (FAQ): "OpenRouter charges a 5.5% (\$0.80 minimum) fee when you purchase credits. We pass through the pricing of the underlying model providers without any markup, so you pay the same rate as you would directly with the provider." "Crypto payments are charged a fee of 5%."
- Quote (BYOK page): "The cost of using custom provider keys on OpenRouter is **5% of what the same model/provider would cost normally on OpenRouter** and will be deducted from your OpenRouter credits. The free allowance is plan-dependent and measured by list-price inference cost, not request count: Pay-as-you-go includes \$25,000 per month, while Enterprise allowances are custom."
- Source: O-s32, O-s17. Basis: OR-documented.

**O27. Per-provider key formats that the documentation lists: Azure, Amazon Bedrock, and Google Vertex take structured JSON, and other providers take a plain key string.**

- Azure (Foundry form): `[{"api_key": "...", "resource_name": "...", "resource_type": "ai_foundry"}]`, where `resource_type` is "Either `"ai_foundry"` for Azure AI Foundry resources (`*.services.ai.azure.com`) or `"openai"` for Azure OpenAI resources (`*.openai.azure.com`). Defaults to `"openai"` if omitted." The legacy per-deployment form has `model_slug`, `endpoint_url`, `api_key`, `model_id`.
- Amazon Bedrock: a Bedrock API key string, or `{"accessKeyId": "...", "secretAccessKey": "...", "region": "..."}`, with optional `crisScope` ("one of `global`, `us`, `eu`, or `apac`") and `crisScopeOverrides`. Required permissions: `bedrock:InvokeModel` and `bedrock:InvokeModelWithResponseStream`.
- Google Vertex: the service account JSON (`"type": "service_account"`, `project_id`, `private_key`, `client_email`, and so on) plus an optional `"region"`.
- The management API example for a plain key is `{"provider": "openai", "key": "sk-...", "name": "...", "allowed_models": [...], "allowed_api_key_hashes": [...], "allowed_user_ids": [...]}`.
- Source: O-s17. Basis: OR-documented.

**O28. LC18 holds. A BYOK key does not exempt a provider from the caller's data policies. The one exception is a ZDR declaration set on the key, which OpenRouter trusts without checking.**

- Quote: "Your data policies apply to BYOK endpoints. A BYOK key changes the credential for the upstream request. It does not change which endpoints you can route to."
- Quote: "A BYOK key does not exempt a provider from your `data_collection` restrictions. For [Zero Data Retention] the same is true by default." "The one exception is a key with a ZDR declaration in its **Provider agreement** section."
- Quote: "Without a ZDR declaration, OpenRouter removes the endpoint that retains prompts. If no ZDR endpoint remains, the request fails. Your valid key for that provider does not change this."
- Quote: "The declaration is your statement about your own provider agreement. OpenRouter does not check it." The management API field is `declared_zdr` (`true`, `false`, or `null`).
- Source: O-s17. Basis: OR-documented.

**O29. Data policy has three tiers per endpoint, and the per-request controls are `provider.data_collection` and `provider.zdr`. ZDR enforcement covers model inference endpoints only, not tools and plugins.**

- Quote (tiers): "**ZDR** | No routine storage after processing", "**Safety Retention** | OpenRouter or a provider may retain prompts and responses from any request, whether flagged or not, only to detect, investigate, and prevent abuse.", "**Retention and Training** | Data may be stored and used for training under the applicable provider policy."
- Quote: "You can enforce ZDR globally, per model group, per guardrail, or per request."
- Quote: "ZDR enforcement only applies to provider routing for inference requests. It does not apply to [plugins] and tools you choose to enable, such as [web search]."
- Quote: "If OpenRouter is not able to establish or ascertain a clear policy for a provider or endpoint, we take a conservative stance and assume that the endpoint both retains and trains on data and mark it as such."
- Quote (what OpenRouter itself keeps): "OpenRouter does not store your prompts or responses, *unless* you opt in to one or both of the following, or content is flagged as potential CSAM". "OpenRouter does store metadata (e.g. number of prompt and completion tokens, latency, etc) for each request."
- Quote (in-region routing is chosen by hostname, not by a field): "send the request to `https://eu.openrouter.ai` or `https://us.openrouter.ai`, and its prompts and completions are processed entirely within that region."
- Source: O-s20, O-s18, O-s1. Basis: OR-documented.

### E. Rate limits

**O30. LC14 holds in substance. Rate limits are governed per account, not per API key, and a paid model has no platform-level request cap. A key can still have a spend cap, which is a credit limit and not a rate limit.**

- Quote: "Making additional accounts or API keys will not affect your rate limits, as we govern capacity globally. We do however have different rate limits for different models, so you can share the load that way if you do run into issues."
- Quote: "switch to the paid variant of the model, which has no platform-level request cap."
- Quote (the three kinds of limit): "Credit limits" ("How much you can spend (account balance, per-key credit caps, and the in-flight spending budget)"), "Budget limits" ("Spend caps an org admin configured"), "Rate limits" ("How many requests you can make (free-model request caps, key-management caps, per-IP burst limits, and DDoS protection)").
- Source: O-s21. Basis: OR-documented.

**O31. Free-model limits: 20 requests per minute, and 50 per day below 10 credits purchased or 1000 per day at 10 or more.**

- Quote: "If you're using a free model variant (with an ID ending in `:free`), the following limits apply:" with rows "Less than 10 | 20 | 50" and "At least 10 | 20 | 1000" under the headings "Credits purchased (all time) | Requests per minute | Requests per day".
- Quote: "The `free_model_daily_requests` field in the `GET /api/v1/key` response above reports the daily counter and ceiling". "The per-minute limit is not reported there."
- Source: O-s21, O-s32. Basis: OR-documented.

**O32. Successful inference responses have no rate-limit headers. `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` appear only on OpenRouter's own 429 for a free-model or key-management limit. `Retry-After` appears on some 429, 503, and in-flight 402 responses.**

- Quote: "Successful inference responses do not include `X-RateLimit-*` headers. When OpenRouter itself returns a  error for a free-model or key-management limit, the error response carries `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` headers describing the limit that was hit. When every attempted provider returned a retry hint, the error response also carries a `Retry-After` header." (The mirrored Markdown drops the status number after "returns a"; the section heading is "Handling 429 errors".)
- Quote: "The `X-RateLimit-Reset` header gives the reset time in milliseconds since the Unix epoch."
- Quote: "On `429` and `503` responses, and on `402` responses whose `error.metadata.limit_source` is `openrouter_in_flight_budget`", "OpenRouter may include a standard HTTP `Retry-After` response header indicating how many seconds to wait before retrying."
- Quote (two origins of a 429): "**OpenRouter**, when you hit one of the platform limits above" and "**The upstream provider**, when the provider serving your request is rate limiting or at capacity. In this case `error.metadata.provider_code` carries the provider's original error code when available".
- What is lost: the vendor's own rate-limit headers (for example Anthropic's `anthropic-ratelimit-*` or OpenAI's `x-ratelimit-*`) are not described as forwarded anywhere in the pages read.
- Source: O-s21, O-s22. Basis: OR-documented. Confidence note: the headers that exist are documented. That vendor rate-limit headers are not forwarded is an inference from their absence in the documentation, confidence medium.

### F. The error envelope

**O33. On Chat Completions the error body is `{"error": {"code": <number>, "message": <string>, "metadata"?: {...}}}`, and the HTTP status equals `error.code` only for errors raised before the response is committed. The spec adds two optional top-level siblings, `openrouter_metadata` and `user_id`.**

- Quote: `type ErrorResponse = { error: { code: number; message: string; metadata?: Record<string, unknown>; }; };`
- Quote: "The HTTP Response will have the same status code as `error.code`, forming a request error if:" "Your original request is invalid" and "Your API key/account is out of credits". "Otherwise, the returned HTTP response status will be `200` and any error occurred while the LLM is producing the output will be emitted in the response body or as an SSE data event."
- Quote (spec, `BadGatewayResponse`, the same shape for every status): properties `error` (required, with `code` integer, `message` string, `metadata` object or null), `openrouter_metadata` (object or null), `user_id` (string or null).
- Source: O-s22, O-s2. Basis: OR-documented.

**O34. Documented HTTP statuses. The guide lists eight, and the spec lists fifteen for `POST /chat/completions`.**

- Guide list: "**400**: Bad Request (invalid or missing params, CORS)", "**401**: Invalid credentials (OAuth session expired, disabled/invalid API key)", "**402**: Your account or API key has insufficient credits. Add more credits and retry the request.", "**403**: Forbidden (insufficient permissions, guardrail block, or moderation flag)", "**408**: Your request timed out", "**429**: You are being rate limited", "**502**: Your chosen model is down or we received an invalid response from it", "**503**: There is no available model provider that meets your routing requirements".
- Spec list for `POST /chat/completions` and `POST /responses`: 200, 400, 401, 402, 403, 404, 408, 413, 422, 429, 500, 502, 503, 524 ("Infrastructure Timeout - Provider request timed out at edge network"), 529 ("Provider Overloaded - Provider is temporarily overloaded").
- Spec list for `POST /messages`: 200, 400, 401, 403, 404, 429, 500, 503, 529.
- Other statuses that the typed table assigns: 404 (`not_found`, `image_not_found`), 412 (`precondition_failed`), 413 (`payload_too_large`), 422 (`unprocessable`), 500 (`server`, `unmapped`), 504 (`timeout`).
- 404 is also the status when routing filters leave no provider (O6) or no router model (O22).
- Source: O-s22, O-s2. Basis: OR-documented.

**O35. `error_type` is the stable, typed error vocabulary. It has 27 values, the same in the guide and in the spec enum `ApiErrorType`. OpenRouter states that the native codes of each route are lossy and that `error_type` is the field to rely on.**

- Quote: "OpenRouter normalizes every upstream provider error into the stable, typed `error_type` vocabulary". "Native protocol codes (the Anthropic `error.type`, the Responses `error.code`) are best-effort and can differ between formats".
- Quote (spec, `ApiErrorType`): "Canonical OpenRouter error type, stable across all API formats", `"enum": ["context_length_exceeded", "max_tokens_exceeded", "token_limit_exceeded", "string_too_long", "authentication", "permission_denied", "payment_required", "rate_limit_exceeded", "provider_overloaded", "provider_unavailable", "invalid_request", "invalid_prompt", "not_found", "precondition_failed", "payload_too_large", "unprocessable", "content_policy_violation", "refusal", "invalid_image", "image_too_large", "image_too_small", "unsupported_image_format", "image_not_found", "image_download_failed", "server", "timeout", "unmapped"]`.
- The guide's status for each value: 400 for `context_length_exceeded`, `max_tokens_exceeded`, `token_limit_exceeded`, `string_too_long`, `invalid_request`, `invalid_prompt`, `invalid_image`, `image_too_large`, `image_too_small`, `unsupported_image_format`, `image_download_failed`; 401 `authentication`; 402 `payment_required`; 403 `permission_denied`, `content_policy_violation`, `refusal`; 404 `not_found`, `image_not_found`; 412 `precondition_failed`; 413 `payload_too_large`; 422 `unprocessable`; 429 `rate_limit_exceeded`; 500 `server`, `unmapped`; 502 `provider_unavailable`; 503 `provider_overloaded`; 504 `timeout`.
- Selected descriptions: `context_length_exceeded` "The combined input and output tokens exceed the model's context window."; `max_tokens_exceeded` "Generation stopped because the output token limit (`max_output_tokens` on the Responses API, `max_tokens` or `max_completion_tokens` on Chat Completions) was reached."; `token_limit_exceeded` "A token budget enforced by OpenRouter (e.g. credit-based cap) was exceeded."; `provider_unavailable` "The upstream provider returned an invalid or empty response. OpenRouter may auto-retry with another provider if fallback routing is enabled."; `unmapped` "An upstream error that doesn't map to any known category. `error.metadata.provider_code` may contain the original code."
- Source: O-s22, O-s2. Basis: OR-documented.

**O36. Where `error_type` sits differs per route: `error.metadata.error_type` on Chat Completions, `error.error_type` on Messages, and a top-level `error_type` beside `error` on Responses.**

- Quote: "**Chat Completions**: `error.metadata.error_type`", "**Anthropic Messages**: `error.error_type` on the SSE `error` event and the non-streaming error envelope.", "**Responses**: top-level `error_type` on the failed response, for both the streaming `response.failed` event and the non-streaming JSON body."
- Quote (Chat Completions example): `{"error": {"code": 429, "message": "Rate limit exceeded", "metadata": {"error_type": "rate_limit_exceeded", "provider_code": "rate_limited"}}}`
- Source: O-s22. Basis: OR-documented.

**O37. LC13 holds. A 402 has `error_type` `payment_required`, the in-flight case is `error.metadata.limit_source` equal to `openrouter_in_flight_budget`, budget rejections are 403 with a `limit_source`, and context overflow is `error_type` `context_length_exceeded`.**

- Quote: "`payment_required` | 402 | The account or API key has insufficient credits."
- Quote (in-flight 402 body): `{"error": {"code": 402, "message": "This request would exceed your available credits given your current in-flight requests. Retry after in-flight requests settle, or add credits.", "metadata": {"reason": "in_flight_budget_exhausted", "limit_source": "openrouter_in_flight_budget", "remedy_hint": "..."}}}`
- Quote: "`"reason": "weight_exceeds_budget"` with `"limit_source": "openrouter_credits"`: this single request's estimated cost is larger than your whole budget, so retrying will not help."
- Quote (402 `limit_source` values): "`openrouter_in_flight_budget` means your running and recently completed requests filled your [in-flight spending budget]", "`openrouter_key_limit` means the API key's credit limit is exhausted. `openrouter_credits` means your balance cannot cover the request, or the single request is too expensive for your in-flight budget."
- Quote (403 budget body): `{"error": {"code": 403, "message": "Budget limit exceeded (weekly limit). Contact your org admin. $52.00 of $50.00 used; resets 2026-10-12T00:00:00.000Z.", "metadata": {"limit_source": "openrouter_guardrail_budget", "remedy_hint": "...", "scope": "workspace_default_member", "limit_usd": 50, "spend_usd": 52, "resets_at": "2026-10-12T00:00:00.000Z"}}}`
- Quote: "`limit_source` | `openrouter_guardrail_budget`, `openrouter_workspace_budget`, or `openrouter_key_limit`". "`remedy_hint` | A one-line next step for people reading logs; branch on `limit_source`, not on this text".
- One oddity to note: `openrouter_key_limit` is listed both as a 402 `limit_source` (limits page, "Handling 402 errors") and as a 403 `limit_source` (budget errors). The pages do not say which status a spent key limit returns in which case.
- Source: O-s22, O-s21. Basis: OR-documented.

**O38. `error.metadata` shapes: moderation errors name the flagged text and the provider, guardrail blocks list matched patterns, and provider errors can hold the vendor's own code and verbatim payload.**

- Quote (moderation): `type ModerationErrorMetadata = { reasons: string[]; flagged_input: string; provider_name: string; model_slug: string; };` with the comments "Why your input was flagged", "The text segment that was flagged, limited to 100 characters.", "The name of the provider that requested moderation".
- Quote (guardrail): `{"error": {"code": 403, "message": "Request blocked: prompt injection patterns detected", "metadata": {"patterns": ["ignore all previous instructions"]}}}`
- Quote (provider error fields): "`metadata.provider_name`, `metadata.provider_code`, and `metadata.raw` identify who declined and carry the provider's verbatim payload."
- Quote (response type comment): "Contains additional error information such as provider details, the raw error message, etc."
- Quote (a worked case in the cookbook): "Moondream returns the same error for every malformed command: `Provider returned error`, with `"message":"Invalid request."` inside `error.metadata.raw`."
- Quote (OpenRouter's own AI SDK provider, a code comment): "The top-level `error.message` is often generic (e.g. "Provider returned error"), while `error.metadata.raw` contains the actual upstream provider error details." The code reads `metadata.provider_name` and treats `metadata.raw` as "a string, a JSON string, or a nested object".
- The spec types `metadata` on the chat stream chunk as only `error_type` (required) and `provider_code` ("Upstream provider-specific error code, when available"). `raw` and `provider_name` are in the guide and the SDK, not in the spec.
- Source: O-s22, O-s26, O-s44, O-s45 (`src/schemas/error-response.ts` lines 27-58), O-s2. Basis: OR-documented.

**O39. Does the vendor's own error code survive? Sometimes. `provider_code` holds it "when available" on non-500 errors, `raw` can hold the verbatim payload, and a 500 masks everything except `error_type`.**

- Quote: "For non-500 errors, the upstream provider's own error code is surfaced in `error.metadata.provider_code` when available."
- Quote: "When a request fails with a `500`, the `message` is replaced with a generic string and `provider_code` and `openrouter_metadata` are omitted, but `error_type` is still present (`server`)."
- Quote: "On 500-class errors, `error.message` is replaced with a generic string and `provider_code` is omitted to prevent leaking upstream details".
- What is lost: the vendor's HTTP status is replaced by OpenRouter's status for the `error_type` (for example content policy: "it is a 403 regardless of whether the model or a filter around it made the call, and regardless of the provider's own wire status"). The vendor's error `type` string (Anthropic's `overloaded_error`, OpenAI's `insufficient_quota`) is kept only if it lands in `provider_code` or `raw`, which the documentation does not promise. The vendor's request id is exposed after the fact as `upstream_id` on the generation record ("Upstream provider's identifier for this generation") and the per-attempt status as `provider_responses[].status` ("HTTP status code from the provider").
- Source: O-s22, O-s2 (`GenerationResponse`, `ProviderResponse`). Basis: OR-documented.

**O40. The Responses route uses the OpenAI Responses error codes, which collapse most types into `server_error`, and adds the canonical `error_type` outside the native `error` object.**

- Quote (mapping table): "`rate_limit_exceeded` | `rate_limit_exceeded`", "`context_length_exceeded`, `invalid_request`, `refusal` | `invalid_prompt`", "`content_policy_violation` | `image_content_policy_violation`", "`authentication`, `provider_overloaded`, `provider_unavailable`, `timeout`, `server` | `server_error`", "All others (including `invalid_prompt`) | `server_error`".
- Quote: `{"id": "resp_abc123", "status": "failed", "error": {"code": "server_error", "message": "Invalid credentials"}, "error_type": "authentication"}`
- Quote (Responses error handling page, a second shape): `{"error": {"code": "invalid_prompt", "message": "Detailed error description"}, "metadata": null}` with the codes `invalid_prompt` (400), `rate_limit_exceeded` (429), `image_content_policy_violation` (403), `permission_denied` (403), `server_error` (500+).
- Quote (spec, `ResponsesErrorField.code` enum, 24 values): `server_error`, `rate_limit_exceeded`, `invalid_prompt`, `vector_store_timeout`, `invalid_image`, `invalid_image_format`, `invalid_base64_image`, `invalid_image_url`, `image_too_large`, `image_too_small`, `image_parse_error`, `image_content_policy_violation`, `invalid_image_mode`, `image_file_too_large`, `unsupported_image_media_type`, `empty_image_file`, `failed_to_download_image`, `image_file_not_found`, `bio_policy`, `cyber_policy`, `misalignment_policy_violation`, `data_residency_mismatch`, `flex_unavailable`, `permission_denied`.
- Source: O-s22, O-s24, O-s2. Basis: OR-documented.

**O41. The Messages route returns Anthropic's error envelope with Anthropic `error.type` strings, adds `error_type` inside `error`, and sets `request_id` to the `gen-` generation id or to null.**

- Quote (mapping table): "`authentication` | `authentication_error`", "`permission_denied` | `permission_error`", "`payment_required` | `billing_error`", "`not_found`, `image_not_found` | `not_found_error`", "`rate_limit_exceeded` | `rate_limit_error`", "`provider_overloaded`, `timeout` | `overloaded_error`", "`context_length_exceeded`, `content_policy_violation`, `refusal`, `invalid_request`, ... | `invalid_request_error`", "`provider_unavailable`, `server`, `unmapped`, ... | `api_error`".
- Quote: `{"type": "error", "error": {"type": "authentication_error", "message": "Invalid credentials", "error_type": "authentication"}, "request_id": null}` and "pre-Router errors such as authentication failures carry a `null` `request_id`; post-Router errors carry the `gen-` generation ID".
- Quote (guardrail blocks on this route): "on `/messages` the block is returned as an Anthropic `permission_error` envelope".
- Quote (spec, `AnthropicMessagesErrorResponse.error.type` enum): `["invalid_request_error", "authentication_error", "permission_error", "not_found_error", "rate_limit_error", "api_error", "overloaded_error", "billing_error", "timeout_error"]`.
- Source: O-s22, O-s2. Basis: OR-documented.

### G. Errors after HTTP 200

**O42. LC11 holds. The 200 status is sent when the provider accepts the request, before any token, so every later failure is reported inside the body. This applies to non-streaming requests too.**

- Quote: "OpenRouter sends you the HTTP `200 OK` status and headers as soon as the provider accepts the request. That happens before the model produces a single token, and an HTTP status is final once sent."
- Quote: "Every failure after that point is therefore reported inside the response instead of in the status. A streaming request receives an SSE error event; a non-streaming request receives an error body. This includes provider errors that arrive before any token."
- Quote (streaming page): "Once the provider has returned response headers, the `200 OK` status is committed even if no token has been produced yet."
- Quote (non-streaming): "If the provider returns headers and then fails, you receive a `200 OK` whose JSON body holds only an `error` object and no `choices`; its `id` identifies the generation when you report the failure. Check the body for an `error` field even on a `200`, rather than relying on the status alone."
- Source: O-s22, O-s23. Basis: OR-documented.

**O43. The mid-stream error chunk on Chat Completions: a normal `chat.completion.chunk` with a top-level `error` object and one choice whose `finish_reason` is `"error"`.**

- Quote (type): `type MidStreamError = { id: string; object: 'chat.completion.chunk'; created: number; model: string; provider: string; error: { code: number; message: string; metadata?: { error_type: string; provider_code?: string; }; }; choices: [{ index: 0; delta: { content: '' }; finish_reason: 'error'; native_finish_reason?: string; }]; };` with the comments "HTTP status code (e.g. 400, 429, 502)" on `code` and "Original upstream error code (omitted on 500s)" on `provider_code`.
- Quote (example): `data: {"id":"gen-abc123","object":"chat.completion.chunk","created":1234567890,"model":"openai/gpt-4o","provider":"OpenAI","error":{"code":429,"message":"Rate limit exceeded","metadata":{"error_type":"rate_limit_exceeded"}},"choices":[{"index":0,"delta":{"content":""},"finish_reason":"error"}]}`
- Quote (characteristics): "The error appears at the **top level** alongside standard response fields", "A `choices` array is included with `finish_reason: "error"` to properly terminate the stream", "The HTTP status remains `200 OK`, because the headers were sent before the error occurred", "The stream is terminated after this event".
- Quote: "The error can be the first and only event in the stream, so treat a `200` carrying an `error` chunk with no content as a failure, not a success".
- Quote (documented causes): "**Provider disconnect**", "**Provider timeout**", "**Token limit hit during generation**" ("the model reaches `max_tokens` or the context window fills up while producing output"), "**Output content filter**", "**Provider overload**".
- Whether `data: [DONE]` follows the error chunk is not stated. The pages say only that the stream is "terminated after this event". See Open questions.
- Source: O-s22, O-s23. Basis: OR-documented.

**O44. A non-streaming request that fails after partial output returns HTTP 200 with the error on the choice, beside the partial content.**

- Quote: `{"choices": [{"message": {"role": "assistant", "content": "partial output..."}, "finish_reason": "error", "error": {"code": 502, "message": "Provider disconnected mid-stream", "metadata": {"error_type": "provider_unavailable"}}}]}`
- Quote (response types): `NonStreamingChoice` and `StreamingChoice` both have `error?: ErrorResponse`.
- Quote (OpenRouter's AI SDK provider, code comments): "Check if response is an error (HTTP 200 with error payload)" and "A provider failure after generation started arrives as HTTP 200 with the error on the choice, next to whatever output was produced."
- Source: O-s22, O-s26, O-s45 (`src/chat/index.ts` lines 345-390). Basis: OR-documented.

**O45. On the Responses route a failed stream ends with `response.failed` (and the schema also allows a plain `error` event). On the Messages route it is an SSE `error` event in Anthropic's shape.**

- Quote: `data: {"type":"response.failed","response":{"id":"gen-abc123", ... ,"status":"failed","output":[],"error":{"code":"server_error","message":"Provider disconnected unexpectedly"},"error_type":"provider_unavailable"},"sequence_number":12}`
- Quote: "OpenRouter ends a failed stream with a `response.failed` event. The stream schema also defines a plain `error` event (upstream OpenAI behavior), so handling both is safe".
- Quote (Messages): `{"type": "error", "error": {"type": "overloaded_error", "message": "Provider is temporarily overloaded", "error_type": "provider_overloaded"}}`
- Source: O-s22, O-s23. Basis: OR-documented.

**O46. "No content" cases: a cold or scaling model can return nothing, and a reasoning model can spend its whole budget on reasoning and return `finish_reason: "length"` with empty content. The first is retried by the caller, the second is not fixed by a retry.**

- Quote: "Occasionally, the model may not generate any content. This typically occurs when:" "The model is warming up from a cold start", "The system is scaling up to handle more requests", "A reasoning model spent the whole `max_tokens` budget on reasoning. The response is a `200` with `finish_reason: "length"`, empty `content`, and `usage.completion_tokens_details.reasoning_tokens` close to `usage.completion_tokens`. Retrying does not help; raise `max_tokens` or cap reasoning instead."
- Quote: "Warm-up times usually range from a few seconds to a few minutes, depending on the model and provider." "If you encounter persistent no-content issues, consider implementing a simple retry mechanism or trying again with a different provider or model that has more recent activity."
- Quote (billing): "When a response contains no output tokens and either has a blank finish reason or an error, you will not be charged for the model's token usage". The two conditions are "The response has zero completion tokens AND a blank/null finish reason" and "The response has an error finish reason".
- Source: O-s22, O-s28. Basis: OR-documented.

### H. Failover and retries

**O47. OpenRouter retries another provider only while no output has reached the caller. After the 200 is sent it still retries until the first token, and it stops for good once part of the answer has been delivered.**

- Quote: "Failover also stops once part of the answer has reached you, since your application already holds output from the first provider."
- Quote: "If an attempt fails before any tokens reach you, OpenRouter automatically tries a backup provider. The `200 OK` has already been sent by then, so the status stays `200` even when every provider fails". The last error then "reaches you in the response body."
- Quote (before the response is committed): OpenRouter can "Silently retry with a different provider endpoint if [fallback routing] is enabled".
- Quote (which errors count against a provider and so trigger a move): "5xx responses", "No response: dropped connections, timeouts", "Setup errors: 401, 402, 404, 405, 408, 410, 421, 423, 424, 426, 498", "Responses that fail partway through". Not counted: "Bad requests: 400, 413, 422", "Geographic or legal blocks: 403, 451", "User disconnects: 499".
- Quote (FAQ): "If a provider returns an error OpenRouter will automatically fall back to the next provider. This happens transparently to the user".
- The attempt trail is visible on request: `openrouter_metadata.attempt` ("1-indexed attempt number that succeeded. Greater than 1 means earlier attempts failed and fell back."), `openrouter_metadata.attempts` ("Per-attempt provider/model/status when the router retried against fallbacks."), and `provider_responses` on the generation record.
- Source: O-s22, O-s3, O-s32, O-s15. Basis: OR-documented.

**O48. Retries the caller is told to do itself: back off on 429 and 503 using `Retry-After`, retry an in-flight 402 by hand because no SDK does, and retry no-content responses.**

- Quote: "The OpenAI SDK, Anthropic SDK, Vercel AI SDK, and OpenRouter SDK already respect this header for backoff on the statuses each SDK retries automatically. None of them retries a `402` on its own, so for the in-flight budget case catch the error, read `Retry-After`, and retry explicitly."
- Quote: "**Retry with exponential backoff.** Rate limits are transient; wait and retry rather than immediately re-sending. Honor the `Retry-After` header when present."
- Quote (cases where retrying cannot work): a 403 budget error ("Retrying before it will fail the same way; there is no `Retry-After` header because the wait is usually hours or days"), `weight_exceeds_budget` ("retrying will not help"), an input overflow on Responses ("Retrying cannot succeed, so it is a terminal failure").
- A mid-stream failure after partial output has no documented automatic recovery. The caller holds partial output and an error chunk, and the pages give no continuation rule for that case.
- Source: O-s22, O-s21. Basis: OR-documented. Confidence note: the last bullet is an inference from the absence of any rule, confidence medium.

### I. Keep-alive comments and timeouts

**O49. The keep-alive is the SSE comment line `: OPENROUTER PROCESSING`. It is sent "occasionally" or "periodic"ally while the caller waits for the first token. No interval is documented.**

- Quote: "For SSE (Server-Sent Events) streams, OpenRouter occasionally sends comments to prevent connection timeouts. These comments look like:" `: OPENROUTER PROCESSING`
- Quote: "If you parse the stream by hand, skip lines that start with `:` before calling `JSON.parse`."
- Quote (compatibility page): "When streaming completions from models that have a long time-to-first-token, OpenRouter sends periodic SSE comment lines to prevent intermediate proxies and cloud load balancers from timing out".
- Quote (server tools): "With streaming (`stream: true`), the server sends SSE comment lines as heartbeats while workers execute."
- Source: O-s23, O-s37. Basis: OR-documented.

**O50. No numeric request or stream timeout is documented for inference. What is documented: a 408 status, a 504 `timeout` type, a 524 edge timeout in the spec, and that OpenRouter cancels a silent provider and moves to another.**

- Quote: "**408**: Your request timed out". "`timeout` | 504 | The provider did not respond within the allowed time." Spec, status 524: "Infrastructure Timeout - Provider request timed out at edge network".
- Quote (provider integration guide, addressed to providers): "If processing takes time (e.g. reasoning models), send SSE comments as keep-alives so we know you're still working on the request. Otherwise we may cancel with a fetch timeout and fallback to another provider".
- Quote (mid-stream cause): "**Provider timeout**" where "the model stops responding mid-generation and the read deadline expires".
- A search of the guides and API reference pages for a timeout with a number of seconds or minutes found none for inference requests (searched 2026-11-22).
- Source: O-s22, O-s2, O-s35. Basis: OR-inferred (that no number is published is inferred from a text search of the documentation and spec; the statuses quoted are documented). Confidence medium.

### J. Cancellation

**O51. Aborting the connection cancels a streaming request, and billing stops only on providers that support it. The documentation lists both groups by name.**

- Quote: "Streaming requests can be cancelled by aborting the connection. For supported providers, this immediately stops model processing and billing."
- Quote (**Supported**): "OpenAI, Azure, Anthropic", "Fireworks, Mancer, Recursal", "AnyScale, Lepton, OctoAI", "Novita, DeepInfra, Together", "Cohere, Hyperbolic, Infermatic", "Avian, XAI, Cloudflare", "SFCompute, Nineteen, Liquid", "Friendli, Chutes, DeepSeek".
- Quote (**Not Currently Supported**): "AWS Bedrock, Groq, Modal", "Google, Google AI Studio, Minimax", "HuggingFace, Replicate, Perplexity", "Mistral, AI21, Featherless", "Lynn, Lambda, Reflection", "SambaNova, Inflection, ZeroOneAI", "AionLabs, Alibaba, Nebius", "Kluster, Targon, InferenceNet".
- Quote: "Cancellation only works for streaming requests with supported providers. For non-streaming requests or unsupported providers, the model will continue processing and you will be billed for the complete response."
- The generation record has `cancelled` ("Whether the generation was cancelled").
- Consequence for the Anthropic model family: the same Claude model stops billing on cancel when served by Anthropic and keeps billing when served by AWS Bedrock or Google, so the caller's cost of an abort depends on which provider the router picked.
- Source: O-s23, O-s2. Basis: OR-documented.

### K. Finish reasons

**O52. LC10 holds for Chat Completions. `finish_reason` is normalized to five values and `native_finish_reason` holds the provider's raw string. Both are on the non-streaming choice, on streaming chunks, and on the final usage chunk.**

- Quote: "OpenRouter normalizes each model's `finish_reason` to one of the following values: `tool_calls`, `stop`, `length`, `content_filter`, `error`."
- Quote: "Some models and providers may have additional finish reasons. The raw finish\_reason string returned by the model is available via the `native_finish_reason` property."
- Quote (response types): `type NonStreamingChoice = { finish_reason: string | null; native_finish_reason: string | null; message: {...}; error?: ErrorResponse; };` and the same two fields on `StreamingChoice`.
- Quote (example): `"finish_reason": "stop", // Normalized finish_reason` and `"native_finish_reason": "stop", // The raw finish_reason from the provider`.
- Quote (final usage chunk): "the usage chunk contains one choice with a content-free `delta` that repeats the `finish_reason` (and `native_finish_reason`) of the stream." "This means the terminal `finish_reason` appears twice".
- Quote (spec, `ChatFinishReasonEnum`): `"enum": ["tool_calls", "stop", "length", "content_filter", "error", null]`.
- Source: O-s26, O-s23, O-s2. Basis: OR-documented.

**O53. Where `native_finish_reason` is absent: the spec's `ChatChoice` and `ChatStreamChoice` schemas do not declare it, the Messages route has no such field, and the Responses route reports only `status` and `incomplete_details.reason`. The generation record has both fields for any route.**

- Spec check (2026-11-22): `ChatChoice` properties are `finish_reason`, `index`, `logprobs`, `message`. `ChatStreamChoice` properties are `delta`, `finish_reason`, `index`, `logprobs`. A search of all 1071 schemas finds `native_finish_reason` only in `BatchObject` and `GenerationResponse`. So typed clients generated from the spec do not see the field on a chat choice.
- Quote (spec, `GenerationResponse`): `finish_reason` "Reason the generation finished", `native_finish_reason` "Native finish reason as reported by provider".
- Quote (cookbook): "Messages has no native finish reason field."
- Quote (spec, `IncompleteDetails.reason` enum on Responses): `["max_output_tokens", "content_filter"]`.
- OpenRouter's own AI SDK provider does not read `native_finish_reason` at all (zero matches in `src` outside one test file, checked at the pinned commit). It keeps the normalized string as `raw`.
- Source: O-s2, O-s42, O-s45. Basis: OR-documented. Confidence note: that typed clients miss the field is an inference from the spec gap, confidence high.

**O54. OpenRouter publishes no complete mapping table from vendor stop reasons to `finish_reason`. The mappings it does state are listed here.**

| Vendor value | What OpenRouter returns | Quote | Source |
| - | - | - | - |
| Anthropic `stop_reason: "refusal"` (a successful response) | Chat: `message.refusal` with `finish_reason: "content_filter"`. Responses: `status: "incomplete"`, `incomplete_details.reason: "content_filter"`, and a `refusal` content part | "When the provider answers successfully with a refusal (e.g. Anthropic's `stop_reason: "refusal"`), each API keeps its native successful shape: Chat Completions returns `message.refusal` with `finish_reason: "content_filter"`, and the Responses API returns `status: "incomplete"` with `incomplete_details.reason: "content_filter"` and a `refusal` content part." | O-s22 |
| Anthropic refusal reported as an error envelope | `error_type: "refusal"`, status 403 | "The provider reported a model refusal as an error rather than as output (e.g. Anthropic's "refused to respond" / usage-policy error envelopes)." | O-s22 |
| Anthropic `max_tokens`, Gemini `MAX_TOKENS`, OpenAI Responses `incomplete` | `finish_reason: "length"` | "any native finish reason that normalizes to `length`, such as `max_tokens` or `MAX_TOKENS`, or a bare Responses `incomplete`" | O-s41 |
| Gemini `MAX_TOKENS` | `finish_reason: "length"` and `native_finish_reason: "MAX_TOKENS"` | "the response ends with `finish_reason: "length"` and `native_finish_reason: "MAX_TOKENS"` and returns a continuation token" | O-s42 |
| Gemini `CONTINUATION` | `finish_reason: "length"` and `native_finish_reason: "CONTINUATION"`; Responses `status: "incomplete"` with `max_output_tokens`; Messages `stop_reason: "max_tokens"` | "the response ends with `finish_reason: "length"`, `native_finish_reason: "CONTINUATION"`, and a continuation token" | O-s42 |
| Gemini `SAFETY` block by a filter outside the model | an error: `error_type: "content_policy_violation"`, status 403 | "The input or output was flagged by a content filter that runs separately from the model (provider- or OpenRouter-level), such as Gemini's `SAFETY` block." | O-s22 |
| Upstream token-limit errors (`max_tokens_exceeded`, `token_limit_exceeded`, `string_too_long`, output-side `context_length_exceeded`) | Chat: `finish_reason: "length"`. Responses: `status: "incomplete"` with `max_output_tokens` | "on Chat Completions the same case is `finish_reason: "length"`" | O-s22 |
| A provider failure mid-generation | `finish_reason: "error"` with an `error` object | O43, O44 | O-s22 |

- Not stated anywhere in the pages read: what `finish_reason` Anthropic's `pause_turn`, `model_context_window_exceeded`, `stop_sequence`, and `compaction` become on Chat Completions, and what Gemini's `RECITATION`, `MALFORMED_FUNCTION_CALL`, `PROHIBITED_CONTENT`, and the other 15 values become. The text search for `pause_turn`, `model_context_window_exceeded`, and `RECITATION` across the guides, API reference, FAQ, and cookbook returned no mapping (searched 2026-11-22).
- Basis: OR-documented for each row. The gap is an open question, see the fills in O56 and O57.

**O55. The Messages route keeps Anthropic's stop-reason vocabulary, including values Anthropic's own quick reference does not list.**

- Quote (spec, `ORAnthropicStopReason`): `"enum": ["end_turn", "max_tokens", "model_context_window_exceeded", "stop_sequence", "tool_use", "pause_turn", "refusal", "compaction", null]`. The changelog records "`ORAnthropicStopReason`: enum property\_added" under "July 24, 2026".
- Anthropic's own quick reference today lists seven: `end_turn`, `max_tokens`, `stop_sequence`, `tool_use`, `pause_turn`, `refusal`, `model_context_window_exceeded`.
- Source: O-s2, O-s27, O-s47. Basis: OR-documented. Confidence note: the comparison list is Anthropic's own page, read live today.

**O56. Vendor rules, read live, that the five-value field has to absorb.**

- Anthropic: "`pause_turn`" is "A server-tool loop reached its iteration limit." and the handling is "Send the assistant content back to continue." "`model_context_window_exceeded`" is "The response filled the model's context window." and "Treat the response as truncated." "`refusal`": "Claude declined to respond." "Read `stop_details`". Refusal detail: "Safety classifiers return this stop reason as a normal HTTP 200 response, not an error." On `model_context_window_exceeded`: "On Sonnet 4.5 (deprecated) and newer models, the API returns this value without a beta header."
- Gemini `FinishReason` has 23 values: `FINISH_REASON_UNSPECIFIED`, `STOP`, `MAX_TOKENS`, `SAFETY`, `RECITATION`, `LANGUAGE`, `OTHER`, `BLOCKLIST`, `PROHIBITED_CONTENT`, `SPII`, `MALFORMED_FUNCTION_CALL`, `IMAGE_SAFETY`, `IMAGE_PROHIBITED_CONTENT`, `IMAGE_OTHER`, `NO_IMAGE`, `IMAGE_RECITATION`, `UNEXPECTED_TOOL_CALL`, `TOO_MANY_TOOL_CALLS`, `MISSING_THOUGHT_SIGNATURE`, `MALFORMED_RESPONSE`, `ESCALATION`, `PUP_LIMITED_DISABLED`, `CONTINUATION`. Gemini has no value that means "the model called a tool": `STOP` is "Natural stop point of the model or provided stop sequence."
- OpenAI Chat Completions `finish_reason`: "`stop` if the model hit a natural stop point or a provided stop sequence", "`length` if the maximum number of tokens specified in the request was reached", "`content_filter` if content was omitted due to a flag from our content filters", "`tool_calls` if the model called a tool, or `function_call` (deprecated)". OpenAI Responses `incomplete_details.reason` enum: `max_output_tokens`, `content_filter`.
- Source: O-s47, O-s49, O-s50. Basis: Vendor-documented.

**O57. Fill for the missing table: two open translators map the same vendor values to OpenAI finish reasons and disagree with each other on three of them.**

| Vendor value | LiteLLM (`_FINISH_REASON_MAP`) | Vercel AI SDK (unified reason) |
| - | - | - |
| Anthropic `end_turn`, `stop_sequence` | `stop` | `stop` |
| Anthropic `max_tokens` | `length` | `length` |
| Anthropic `tool_use` | `tool_calls` | `tool-calls` (or `stop` when the tool is the JSON-response tool) |
| Anthropic `refusal` | `content_filter` | `content-filter` |
| Anthropic `model_context_window_exceeded` | `length` | `length` |
| Anthropic `pause_turn` | `length` | `stop` |
| Anthropic `compaction` | `length` | `other` |
| Gemini `STOP` | `stop` | `tool-calls` when the reply has tool calls, else `stop` |
| Gemini `MAX_TOKENS`, `CONTINUATION` | `length` | `length` |
| Gemini `SAFETY`, `RECITATION`, `BLOCKLIST`, `PROHIBITED_CONTENT`, `SPII`, `IMAGE_SAFETY` | `content_filter` | `content-filter` |
| Gemini `MALFORMED_FUNCTION_CALL` | `stop` | `error` |
| Gemini `OTHER` | `content_filter` | `other` |
| Gemini `MISSING_THOUGHT_SIGNATURE`, `TOO_MANY_TOOL_CALLS`, `UNEXPECTED_TOOL_CALL`, `MALFORMED_RESPONSE` | `stop` | `other` (default branch) |
| OpenAI Responses `max_output_tokens` | not in this table | `length` |
| OpenAI Responses `content_filter` | `content_filter` | `content-filter` |
| OpenRouter `error` | `stop` | not applicable |

- LiteLLM's comment on its last row: "Generic error passthrough (OpenRouter and other OpenAI-compatible providers emit lowercase "error" when a provider fails mid-stream)". An unknown value logs "Unmapped finish_reason '%s', defaulting to 'stop'".
- What the fill shows: every vendor stop reason can be folded into the five OpenAI-style values, and the fold is a judgment call (`pause_turn` is `length` in one project and `stop` in the other). It does not show which fold OpenRouter uses.
- What each fold loses: `pause_turn` (the caller must send the assistant content back to continue) becomes indistinguishable from a normal stop or a truncation; `model_context_window_exceeded` becomes the same `length` as an ordinary `max_tokens` stop, so "the context is full" and "the answer was cut by my own cap" look alike; `refusal` loses Anthropic's `stop_details` category; Gemini's ten filter reasons become one `content_filter`.
- Source: O-s46 (`litellm/litellm_core_utils/core_helpers.py` lines 514-580), O-s54 (`packages/anthropic/src/map-anthropic-stop-reason.ts`, `packages/google/src/map-google-finish-reason.ts`, `packages/openai/src/responses/map-openai-responses-finish-reason.ts`). Basis: Filled (LiteLLM, Vercel AI SDK).

**O58. OpenRouter's own client patches a normalization gap: for Gemini tool calls with a thought signature, OpenRouter returns `finish_reason: "stop"`, and the official AI SDK provider rewrites it to tool calls on the client.**

- Quote (code comment): "Fix for Gemini 3 thoughtSignature: when there are tool calls with encrypted reasoning (thoughtSignature), the model returns 'stop' but expects continuation. Override to 'tool-calls' so the SDK knows to continue the conversation."
- Quote (code): `const shouldOverrideFinishReason = hasToolCalls && hasEncryptedReasoning && choice.finish_reason === 'stop';`
- Quote (second patch): "Fix for #420: When finishReason is 'other' (unknown/missing) but tool calls were made, infer 'tool-calls' so agentic loops continue correctly."
- What this shows: a caller of OpenRouter cannot rely on `finish_reason: "tool_calls"` alone to detect a tool call. It has to look at `tool_calls` in the message. This matches the Gemini rule in O56 (no tool-call finish value) passing through OpenRouter's normalizer unchanged in at least some cases.
- Source: O-s45 (`src/chat/index.ts` lines 549-569 and 1270-1288, `src/utils/map-finish-reason.ts` lines 11-29). Basis: OR-inferred (from OpenRouter's official client code special-casing its own API; confidence high that the case occurs, unknown how often).

**O59. One documented case where the caller must read a vendor value out of `native_finish_reason` to act correctly: Gemini long decode.**

- Quote: "`max_tokens` is cumulative across hops. A response that ended with `MAX_TOKENS` has spent its whole budget but still returns a continuation token: resend it with a higher `max_tokens` to continue. Resending it with the same `max_tokens` returns a 400 error."
- Quote: "A hop with empty content and `native_finish_reason: "CONTINUATION"` is normal: resume it."
- Quote: "keep the stopped turn with its continuation items (reasoning details or Responses API reasoning items whose `id` is `gemini-continuation`)."
- Quote (Messages): "Resume only when `stop_reason` is `"max_tokens"`, and pass the assistant content back unchanged: continuation blocks look like other `redacted_thinking` blocks", and "The `data` value wraps the Gemini token in an OpenRouter envelope so its format survives the round trip."
- Deciding facts: the continuation token is opaque state that the caller stores and sends back unchanged; the neutral `finish_reason: "length"` is enough to trigger the resume loop; the vendor value is needed only to tell a spent budget (`MAX_TOKENS`, raise the cap) from a per-request cap (`CONTINUATION`, resend as is), and the documentation gives one rule that avoids reading it ("set `max_tokens` above the total output of every hop so far").
- Source: O-s42. Basis: OR-documented.

### L. Context overflow

**O60. LC16 holds. The documented way to deal with an over-long prompt is the `context-compression` plugin, not a `transforms` field. The plugin's only engine is still named `middle-out`.**

- Quote: "To help with prompts that exceed the maximum context size of a model, OpenRouter supports a context compression [plugin] that can be enabled per-request:" `"plugins": [{ "id": "context-compression" }]`
- Quote (spec, `ContextCompressionPlugin`): `id` enum `["context-compression"]`, `enabled` "Set to false to disable the context-compression plugin for this request. Defaults to true.", `engine` "The compression engine to use. Defaults to "middle-out"." with enum `["middle-out"]`.
- The request field `transforms` appears nowhere in the spec (zero matches in 1071 schemas) and nowhere in the mirrored guides and API reference outside the page title "Message Transforms" (searched 2026-11-22). Whether the server still accepts `transforms: ["middle-out"]` is not documented. LiteLLM's OpenRouter adapter still forwards a `transforms` parameter (`extra_body["transforms"] = transforms`), which shows clients in the field still send it.
- The changelog cannot date the change: it starts on July 3, 2026 and has no entry mentioning transforms or compression.
- Source: O-s25, O-s2, O-s33, O-s27, O-s46 (`litellm/llms/openrouter/chat/transformation.py` lines 98-102). Basis: OR-documented. Confidence note: the plugin is documented. The status of `transforms` is an inference from its absence in the spec and from client code, confidence medium.

**O61. What the plugin does: it removes or truncates messages from the middle until the prompt fits, and when the message count is the problem it keeps half from the start and half from the end. It also changes which models are eligible.**

- Quote: "The plugin works by removing or truncating messages from the middle of the prompt, until the prompt fits within the model's context window."
- Quote: "For instance, Anthropic's Claude models enforce a maximum of 100000 messages. When this limit is exceeded with context compression enabled, the plugin will keep half of the messages from the start and half from the end of the conversation."
- Quote: "When context compression is enabled, OpenRouter will first try to find models whose context length is at least half of your total required tokens (input + completion)." "If no models meet this criteria, OpenRouter will fall back to using the model with the highest available context length."
- Quote: "The middle of the prompt is compressed because [LLMs pay less attention] to the middle of sequences."
- It reports itself when router metadata is on: a pipeline stage `{"type": "context_compression", "name": "context-compression", "data": {"engine": "middle-out", "input_type": "messages", "original_count": 42, "compressed_count": 30}}`. "A plugin only emits a stage when it actually ran".
- Source: O-s25, O-s15. Basis: OR-documented.

**O62. Default behavior: compression is on by default for endpoints with a context of 8,192 tokens or less, and off otherwise. It is always off for requests that use deferred tools and for image-only output models.**

- Quote: "[All OpenRouter endpoints] with 8k (8,192 tokens) or less context length will default to using context compression. To disable this, pass `plugins: [{"id": "context-compression", "enabled": false}]` in the request body."
- Quote (tool search page): "**Context compression is off.** OpenRouter does not apply `middle-out` or other context compression to deferred-tool requests, so a conversation that outgrows the model's context window fails instead of being trimmed."
- Quote: "Context compression is automatically skipped for [image-generation models] that only output images (no text output)."
- Plugins can also be turned on account-wide: "Plugins can be enabled per-request via the API or configured as defaults for all your API requests through the [Plugins settings page]". So a caller cannot tell from its own request alone whether compression is active.
- Source: O-s25, O-s43, O-s33. Basis: OR-documented.

**O63. What comes back when the prompt is too long and compression is off: a 400-class error with `error_type` `context_length_exceeded`. The message text is described but not quoted, and the same overflow can also arrive as a model fallback, as `finish_reason: "length"`, or as a mid-stream error.**

- Quote: "If context compression is disabled and your total tokens exceed the model's context length, the request will fail with an error message suggesting you either reduce the length or enable context compression." The pages do not print the message text.
- Quote: "`context_length_exceeded` | 400 | The combined input and output tokens exceed the model's context window."
- Quote (Responses route split by which side overflowed): "`context_length_exceeded` (with `param: "input"`, or an overflow message) | terminal failure", "`context_length_exceeded` (output budget) | `status: "incomplete"`, `incomplete_details.reason: "max_output_tokens"`". "The exception is a length error reported against the `input` itself (the prompt overflowed the context window). Retrying cannot succeed, so it is a terminal failure: a non-streaming request gets an error status, and a stream ends with `response.failed` carrying `error.code: "invalid_prompt"`."
- Quote (Chat Completions, output side): "the same token-limit errors become `finish_reason: "length"`".
- Quote (it can trigger a model fallback instead of an error): "any error can trigger the use of a fallback model, including:" "Context length validation errors".
- Quote (it can arrive mid-stream): "**Token limit hit during generation**" where "the model reaches `max_tokens` or the context window fills up while producing output".
- Quote (routing avoids some overflows up front): "if you set `max_output_tokens` (or `max_tokens` on Chat Completions), then OpenRouter will only route to providers that support a response of that length."
- Source: O-s25, O-s22, O-s23, O-s4, O-s1. Basis: OR-documented.

**O64. How a caller detects overflow reliably, assembled from the documented signals.**

- Input overflow, any route: `error_type` equal to `context_length_exceeded`, read from the route's location (O36), on an HTTP 400 or inside a 200 body or stream event. On Responses the native code is `invalid_prompt`, shared with `invalid_request` and `refusal`, so the native code alone does not identify overflow. On Messages the native type is `invalid_request_error`, shared with content policy and invalid requests.
- Output-side exhaustion: `finish_reason: "length"` on Chat, `status: "incomplete"` with `max_output_tokens` on Responses, `stop_reason: "max_tokens"` on Messages. These do not separate "the caller's own `max_tokens` cap" from "the context window filled", unless `native_finish_reason` holds a vendor value such as Anthropic's `model_context_window_exceeded` (not documented by OpenRouter, see O54).
- Silent trimming: with the header `X-OpenRouter-Metadata: enabled`, a `context_compression` stage in `openrouter_metadata.pipeline` says the prompt was trimmed. Without the header the caller gets a normal answer to a shortened prompt and no signal.
- Silent model switch: with `models` set, a context-length error on the first model moves the request to the next model, and the only signal is the `model` field of the response.
- Source: O-s22, O-s15, O-s4. Basis: OR-inferred (a synthesis of documented behaviors; each part is documented in the items it cites).

### M. The debug option

**O65. LC2's debug claim holds. The field is `debug.echo_upstream_body`. It returns the transformed request body that OpenRouter sent upstream, in the vendor's own wire shape, as the first chunk of the stream.**

- Quote: "OpenRouter provides a `debug` option that allows you to inspect the exact request body that was sent to the upstream provider. This works with both the Chat Completions API (`/api/v1/chat/completions`) and the Responses API (`/api/v1/responses`)."
- Quote: `type DebugOptions = { echo_upstream_body?: boolean; // If true, returns the transformed request body sent to the provider };`
- Quote (Chat Completions): "When `debug.echo_upstream_body` is set to `true`, OpenRouter sends a debug chunk as the **first chunk** in the streaming response. This chunk has an empty `choices` array and includes a `debug` field with the transformed request body".
- Quote (Responses): "On the Responses API, debug data arrives as a `response.debug` SSE event" with `"sequence_number": 0`.
- Quote (one chunk per provider attempt): "When using provider fallbacks, a debug chunk will be sent for **each attempted provider**, allowing you to see which providers were tried and what parameters were sent to each."
- The Messages route is not named as supporting it.
- Source: O-s22. Basis: OR-documented.

**O66. Limits of the debug option: streaming only, not for production, and redacted on a best-effort basis.**

- Quote: "The debug option **only works with streaming mode** (`stream: true`). Non-streaming requests will ignore the debug parameter."
- Quote: "The debug flag should **not be used in production environments**. It is intended for development and debugging purposes only, as it may potentially return sensitive information included in the request that was not intended to be visible elsewhere."
- Quote: "OpenRouter will make a best effort to automatically redact potentially sensitive or noisy data from debug output."
- Source: O-s22. Basis: OR-documented.

**O67. The documentation prints one transformed upstream body, and it is direct evidence of the Chat Completions to Anthropic Messages translation.**

- The request sent to OpenRouter: `{"model": "anthropic/claude-haiku-5.5", "stream": true, "messages": [{ "role": "system", "content": "You are a helpful assistant." }, { "role": "user", "content": "Hello!" }], "debug": { "echo_upstream_body": true }}`
- The echoed upstream body: `{"system": [{"type": "text", "text": "You are a helpful assistant."}], "messages": [{"role": "user", "content": "Hello!"}], "model": "claude-haiku-5-5", "stream": true, "metadata": {"user_id": "<redacted>"}, "max_tokens": 65536, "stop_sequences": []}`
- What the pair shows, field by field:
  - The `system` role message is lifted out of `messages` into Anthropic's top-level `system` field as an array of text blocks.
  - The model slug `anthropic/claude-haiku-5.5` becomes the vendor id `claude-haiku-5-5`.
  - `max_tokens`, which the caller did not send, is filled with `65536` (Anthropic requires the field). The cookbook confirms the default for another vendor: "When `max_tokens` is omitted, OpenRouter sends its default output budget of 65,536 tokens".
  - `metadata.user_id` is added by OpenRouter and shown redacted. This matches the `user` field rule in O69: the identity sent upstream is a hash.
  - `stop_sequences: []` is sent even though the caller sent no `stop`.
  - The debug chunk itself has `"provider": "Anthropic"` and `"model": "anthropic/claude-haiku-5.5"`.
- The same request on the Responses route echoes the same Anthropic body without `system`, since that request had no instructions.
- Quote (the documented use cases): "See how OpenRouter maps your parameters to provider-specific formats", "how system messages are concatenated, how user messages are merged", "See what default values OpenRouter applies when parameters are not specified in your request."
- Source: O-s22, O-s42. Basis: OR-documented.

**O68. The spec shows a second debug payload that the guide does not describe: timing milestones.**

- Quote (spec, `DebugEvent.description`): "Debug event emitted when debug.echo_upstream_body is true. Contains the transformed upstream request body or timing milestones."
- Quote (spec, `DebugEvent.debug.timings`): properties `start_ms`, `epoch_ms`, and `event` with enum `["adapter_request", "upstream_headers_received", "first_token_received", "upstream_body_ended"]`.
- Quote (spec, `ChatDebugOptions`): "Debug options for inspecting request transformations (streaming only)".
- Source: O-s2. Basis: OR-documented.

### N. Session, user, metadata, generation id, router metadata, service tier

**O69. `session_id`, `user`, and `metadata` on the request.**

- Quote (spec, `ChatRequest.session_id`): "A unique identifier for grouping related requests (e.g., a conversation or agent workflow). When provided, OpenRouter uses it as the sticky routing key, routing all requests in the session to the same provider to maximize prompt cache hits. Also used for observability grouping. If provided in both the request body and the x-session-id header, the body value takes precedence. Maximum of 256 characters."
- Quote (sticky routing without it): "By default, OpenRouter identifies conversations by hashing the first system (or developer) message and the first non-system message in each request". "Sticky sessions expire after **60 minutes** of inactivity." "If neither is set, OpenRouter falls back to the OpenAI-style `prompt_cache_key` request field as the sticky routing key." "Sticky routing is not used when you specify a manual [provider order] via `provider.order`".
- Quote (spec, `ChatRequest.user`): "Per-end-user identifier for abuse isolation. Use a stable ID, hash, or pseudonym. When a provider requires a user identity, OpenRouter folds it into the hashed identity sent upstream and never forwards it raw. If omitted, requests use an account-level identity, so provider policy blocks can affect the whole account."
- Quote (spec, `ChatRequest.metadata`): "Key-value pairs for additional object information (max 16 pairs, 64 char keys, 512 char values)". A separate `trace` object holds observability keys: "Known keys (trace_id, trace_name, span_name, generation_name, parent_span_id) have special handling."
- Source: O-s2, O-s40. Basis: OR-documented.

**O70. The generation id is the response `id` (prefix `gen-`), is also returned in the `X-Generation-Id` header on every route, and keys a record that holds what the response body leaves out.**

- Quote: "The generation ID is returned in the `X-Generation-Id` response header for all endpoints (chat completions, completions, responses, and messages), which can be useful for debugging and correlating requests."
- Quote: "You can also use the returned `id` to query for the generation stats (including token counts and cost) after the request is complete via the `/api/v1/generation` endpoint."
- Quote (spec, `GenerationResponse` fields): `provider_name` "Name of the provider that served the request", `provider_responses` "List of provider responses for this generation, including fallback attempts", `upstream_id` "Upstream provider's identifier for this generation", `finish_reason`, `native_finish_reason`, `cancelled`, `streamed`, `is_byok`, `router` "Router used for the request (e.g., openrouter/auto)", `service_tier`, `session_id`, `request_id` "Unique identifier grouping all generations from a single API request".
- Reading the record needs an authenticated second request, so it is an after-the-fact audit path and not part of the response.
- Source: O-s23, O-s26, O-s2. Basis: OR-documented.

**O71. Router metadata is an opt-in block, `openrouter_metadata`, switched on per request with the header `X-OpenRouter-Metadata: enabled`. It tells the caller which provider served the request, which attempt succeeded, and which plugins changed the request.**

- Quote: "Router metadata is a **per-request opt-in** that adds an `openrouter_metadata` field to successful responses, capturing exactly what the router did."
- Quote (fields): `requested`, `strategy` ("`direct`, `auto`, `free`, `latest`, `alias`, `fallback`, `pareto`, `bodybuilder`, `fusion`"), `region`, `summary`, `attempt`, `is_byok`, `endpoints`, `params`, `attempts`, `pipeline`.
- Quote: "For streaming responses, `openrouter_metadata` is delivered on the **final chunk** before `data: [DONE]` (Chat Completions / Responses) or as part of the terminal `message_stop` event (Anthropic Messages)."
- Quote: "Opt-in error responses surface `openrouter_metadata` at the **top level** of the error envelope". "Responses with a `500` status are scrubbed to a generic message, and `openrouter_metadata` is omitted from those envelopes by design." "Other 5xx classes (`502`, `503`, `504`, `529`) still include the metadata when the client opted in."
- Quote: "Cache hits never include `openrouter_metadata`."
- Quote: "The previous header name `X-OpenRouter-Experimental-Metadata` is still accepted for backward compatibility."
- Without the header, the guide's chunk examples still name the provider (`"provider": "OpenAI"` on the mid-stream error chunk, `"provider": "Anthropic"` on the debug chunk) and the model in `model`. The spec's `ChatResult` and `ChatStreamChunk` schemas do not declare a `provider` property (checked 2026-11-22), so this field is in the guide and not in the typed contract.
- Source: O-s15, O-s22, O-s2. Basis: OR-documented.

**O72. `service_tier` is a neutral request value that OpenRouter maps to each provider's own tier label, and the response reports the tier that actually served.**

- Quote (spec, `ChatRequest.service_tier`): enum `["auto", "default", "fast", "flex", "priority", "scale", "ultrafast", null]`, "The service tier to use for processing this request. `fast` is accepted as an alias for `priority`. `ultrafast` prefers ultrafast endpoints and falls back to `priority`, then default endpoints."
- Quote: "Providers label their priority endpoints differently (`fast`, `turbo`, `highspeed`, `prime`), and an endpoint's badge and slug use the provider's label (for example `deepinfra/turbo` or `alibaba/prime`), while `service_tier: "priority"` or `"fast"` requests all of them."
- Quote: "These values work the same for every provider that sells more than one tier; the examples on this page use OpenAI models, but nothing about the parameter is OpenAI-specific."
- Quote (response): `service_tier` "The service tier used by the upstream provider for this request" on `ChatResult` and `ChatStreamChunk`. "the response reports the tier that actually served the request, and you're billed at that endpoint's price."
- Source: O-s2, O-s29. Basis: OR-documented.

### O. Vendor error vocabularies and what the flattening loses

**O73. Anthropic's native error vocabulary, read live, against OpenRouter's canonical types.**

- Anthropic's list: "400 - `invalid_request_error`", "401 - `authentication_error`", "402 - `billing_error`", "403 - `permission_error`", "404 - `not_found_error`", "409 - `conflict_error`", "413 - `request_too_large`", "429 - `rate_limit_error`", "500 - `api_error`", "504 - `timeout_error`", "529 - `overloaded_error`". Error shape: "a top-level `error` object that always includes a `type` and `message` value. The response also includes a `request_id` field".
- Anthropic on in-stream errors: "When receiving a [streaming] response over server-sent events (SSE), an error can occur after the API returns a 200 response."
- Anthropic on the spend-cap 429: "A tier spend-cap 429 has no `retry-after` header and keeps failing until access resumes".
- What OpenRouter's flattening would do with these, matching the two published tables: Anthropic's 529 `overloaded_error` corresponds to `provider_overloaded` (canonical status 503); Anthropic's 504 `timeout_error` to `timeout`; Anthropic's prompt-too-long error, which the workspace report of 2026-09-30 records as "400 `invalid_request_error` plus text "prompt is too long"", to `context_length_exceeded`; the spend-cap 429 and the ordinary 429 both to `rate_limit_exceeded` unless `provider_code` or `raw` tells them apart; `conflict_error` has no canonical counterpart.
- What is lost: Anthropic tells overflow apart only by message text, and OpenRouter has to do that text match on the caller's behalf, so the caller depends on OpenRouter's matcher keeping up with Anthropic's wording. The two kinds of 429 need opposite handling (wait, or stop) and share one `error_type`.
- Source: O-s48 (Anthropic), O-s22 (OpenRouter's types), O-s51 (the prompt-too-long wording, not re-checked live today). Basis: Vendor-documented. Confidence note: Anthropic's list is the vendor's own. The per-value correspondence to OpenRouter's types is an inference made by matching the two published tables (OpenRouter publishes the reverse mapping for its Messages route only, see O41), confidence medium.

### P. How open clients handle OpenRouter's error and finish signals

**O74. OpenRouter's official AI SDK provider treats any chunk with an `error` key as a stream error, treats an HTTP 200 body with a top-level `error` as an API error with status 200, and treats a choice-level `error` the same way.**

- Quote (`src/chat/index.ts` lines 794-799): `// handle error chunks:` `if ('error' in value) { finishReason = createFinishReason('error'); controller.enqueue({ type: 'error', error: value.error }); return; }`
- Quote (lines 345-361): `// Check if response is an error (HTTP 200 with error payload)` then `throw new APICallError({ message: errorData.message, ..., statusCode: 200, ... })`.
- The provider's error schema accepts `code` as a string or a number (`z.union([z.string(), z.number()])`) plus optional `type` and `param`. The guide's type says `code: number`. The looser schema suggests that string codes reach clients in practice, but the code does not say when.
- A hand-written test in the same repository models a stream in which a chunk with `"finish_reason":"error"` is followed by a usage chunk with `"choices":[]` and then `data: [DONE]` (`src/chat/index.test.ts` lines 2742-2755). This is a test fixture, not a capture, so it is weak evidence about the real stream tail.
- Source: O-s45. Basis: OR-documented (official SDK source). Confidence note: the handling is read from the code. The stream tail in the last bullet is an inference from a test fixture, confidence low.

**O75. LiteLLM's OpenRouter adapter raises an exception from the error chunk using the code inside the chunk, defaults a choice-level error with no usable code to 502, and its general finish-reason table turns `error` into `stop`.**

- Quote (`litellm/llms/openrouter/chat/transformation.py` lines 303-319): `## HANDLE ERROR IN CHUNK ##` `if "error" in chunk:` ... `raise OpenRouterException(message=error_message["message"], status_code=error_message["code"], headers=error_message["metadata"].get("headers", {}))`. The message is built as `"Message: {}, Metadata: {}, User ID: {}"`, reading `error_chunk.get("user_id", "")`.
- Quote (lines 37-57): `choice.get("finish_reason") == "error" or choice.get("error")`, then `status_code=code if isinstance(code, int) and 400 <= code < 600 else 502` and the default message "OpenRouter returned finish_reason 'error': the provider failed during generation".
- Quote (`core_helpers.py` line 571): `"error": "stop",`
- What this shows: a client that maps OpenRouter's fifth value through an OpenAI-only vocabulary loses the failure, so the error object, not the finish reason, has to be the signal.
- Source: O-s46. Basis: Filled (LiteLLM).

### Q. Comparison with the workspace report of 2026-09-30

**O76. The workspace report's three OpenRouter claims all still hold today, and today's documentation adds detail that the report did not have.**

- Report claim 1: "OpenRouter sends `error_type: context_length_exceeded`". Holds (O35, O63). Added today: the field's location differs per route (`error.metadata.error_type`, `error.error_type`, top-level `error_type`), and on the Responses and Messages routes the native code is a shared bucket (`invalid_prompt`, `invalid_request_error`), so only `error_type` identifies overflow (O36, O40, O41).
- Report claim 2: OpenRouter delivers errors inside a 200 stream as "a chunk with `finish_reason: "error"`". Holds (O43). Added today: the same happens on non-streaming requests (a 200 body with only `error`, or an `error` on the choice beside partial content, O42 and O44); the error can be "the first and only event in the stream"; failover continues after the 200 until the first token (O47).
- Report claim 3: "OpenRouter changed its overflow advice from the "middle-out" transform to its `context-compression` plugin between January and April 2026." Today's pages agree on the end state (O60). The date range cannot be re-checked from today's sources because the changelog starts on July 3, 2026. The engine inside the plugin is still called `middle-out`.
- Added today and absent from the report: output-side overflow is turned into a normal truncated completion (`finish_reason: "length"`, O63); a context-length error can silently trigger a model fallback (O15); compression is on by default for endpoints of 8,192 tokens or less and off for deferred-tool requests (O62); zero-completion insurance means an errored or empty response is not billed (O46).
- The report's table row "429 with `insufficient_quota` or `credit_balance_exhausted` is `QuotaExhausted`; 503 `server_is_overloaded` is `Overloaded`" groups OpenRouter with OpenAI, Azure, and Groq. For OpenRouter itself the quota signal is HTTP 402 with `error_type` `payment_required` and a `limit_source`, or HTTP 403 for budgets (O37), and overload is `provider_overloaded` (503 in the typed table, 529 in the spec). The row is right for OpenAI and does not describe OpenRouter's own signals.
- Source: O-s51, and the items cited. Basis: OR-documented. Also in workspace: `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30), whose wording is quoted above.

### R. Recorded OpenRouter streams in an open-source project (third-party captures)

The rig project (Rust agent library) keeps recorded HTTP exchanges with OpenRouter as test cassettes: 63 files under `crates/rig-cassette/fixtures/cassettes/openrouter/` plus `crates/rig-cassette/fixtures/bank/openrouter.yaml`. The generation ids in them begin `gen-1790...`, which as Unix time is late September 2026. The recordings are redacted (`"created":0`, `user_REDACTED`). They are third-party captures, so they show what the wire looked like on the day of recording and for the models recorded, not a contract. All 64 files were scanned with two small scripts on 2026-11-22 that parse every `data:` line as JSON and tally the finish fields, error objects, and comment lines.

**O77. Observed pairs of normalized and native finish reason. The native value depends on the upstream protocol as well as on the vendor: OpenAI models answer with `completed` and `max_output_tokens` (Responses API vocabulary) for some models and `stop`, `length`, `tool_calls` (Chat Completions vocabulary) for others.**

| Model | `provider` | `finish_reason` | `native_finish_reason` |
| - | - | - | - |
| `anthropic/claude-haiku-4.5` | Anthropic | `stop` | `end_turn` |
| `anthropic/claude-haiku-4.5` | Anthropic | `tool_calls` | `tool_use` |
| `anthropic/claude-haiku-4.5` | Amazon Bedrock | `stop` | `end_turn` |
| `anthropic/claude-haiku-4.5` | Amazon Bedrock | `tool_calls` | `tool_use` |
| `anthropic/claude-sonnet-4.6` | Claude Platform on AWS | `stop` | `end_turn` |
| `google/gemini-2.5-flash`, `google/gemini-3-flash-preview` | Google | `stop` | `STOP` |
| `openai/gpt-4o`, `openai/gpt-4o-mini` | OpenAI, Azure | `stop` | `stop` |
| `openai/gpt-4o`, `openai/gpt-4o-mini` | OpenAI | `tool_calls` | `tool_calls` |
| `openai/gpt-4o-mini` | OpenAI | `length` | `length` |
| `openai/gpt-4.1-mini`, `openai/gpt-5.2`, `openai/o4-mini` | OpenAI | `stop` | `completed` |
| `openai/gpt-4.1-mini`, `openai/gpt-5.2`, `openai/o4-mini` | OpenAI | `tool_calls` | `completed` |
| `openai/gpt-4.1-mini` | OpenAI | `length` | `max_output_tokens` |
| `openai/gpt-4.1` (non-streaming refusal) | OpenAI | `content_filter` | `content_filter` |

- Every chunk has `"native_finish_reason":null` until the last content chunk, a `"provider"` name, and an id of the form `gen-<unix time>-<random>`.
- Stream tail as recorded: `data: {"choices":[{"delta":{"content":"","reasoning":null,"role":"assistant"},"finish_reason":"stop","index":0,"native_finish_reason":"completed"}],...}` then a second chunk with the same finish fields plus `"service_tier":"default"` and `"usage":{...}`, then `data: [DONE]`. This matches LC12 and the documented final usage chunk (O52).
- What this shows: `native_finish_reason` is not one vocabulary per vendor. A caller that wants to read it has to know which upstream protocol OpenRouter chose for that model, which the response does not say. For OpenAI models served over the Responses protocol, OpenRouter derives `tool_calls` itself, since the native value is `completed`.
- Source: O-s55 (`crates/rig-cassette/fixtures/cassettes/openrouter/reasoning_roundtrip/streaming.yaml`, `crates/rig-cassette/fixtures/bank/openrouter.yaml`, and the other cassettes). Basis: OR-inferred (from third-party recordings of OpenRouter responses; confidence high for the recorded models and date, unknown for others).

**O78. A truncated tool call is reported as `finish_reason: "tool_calls"`. Only `native_finish_reason` shows that the output hit the token limit.**

- Recorded stream (`bank/openrouter.yaml` lines 1406-1420, model `openai/gpt-4o-mini`, provider OpenAI): argument fragments `"\":\""`, `"The"`, `" cache"`, `" warmer"`, `" raced"`, then `data: {"choices":[{"delta":{"content":"","role":"assistant"},"finish_reason":"tool_calls","index":0,"native_finish_reason":"length"}],...}`, then the usage chunk with `"completion_tokens":16` and the same two finish fields, then `data: [DONE]`. The arguments stop in the middle of a JSON string.
- The same pattern for `openai/gpt-4.1-mini`: `"finish_reason":"tool_calls"` with `"native_finish_reason":"max_output_tokens"` (lines 2156-2158).
- OpenRouter's own page on tool-call quality describes the case from the server side: "When generation hits the request's token limit (any native finish reason that normalizes to `length`, such as `max_tokens` or `MAX_TOKENS`, or a bare Responses `incomplete`), the last tool call is recorded as `truncated-arguments`".
- What is lost: the neutral field says "run the tools" when the vendor said "cut off". A caller that trusts `finish_reason` alone parses incomplete JSON arguments. The reliable checks are `native_finish_reason` (vendor and protocol specific values: `length`, `max_output_tokens`, `max_tokens`, `MAX_TOKENS`) or validating the arguments as JSON.
- Source: O-s55, O-s41. Basis: OR-inferred (third-party recording; confidence high that it happened as recorded, medium that it is the general rule).

**O79. Keep-alive comments arrive both before the first data chunk and between data chunks.**

- Five of the 64 recorded files contain `: OPENROUTER PROCESSING`. In `reasoning_roundtrip/streaming.yaml` there are 14 before the first `data:` line and 15 after it. In `reasoning_usage_matrix/transports_agree_on_reasoning_tokens.yaml` there are 6 before and 11 after. The other three have them only before the first data line (3, 8, and 34 lines).
- Each comment is its own SSE event: the line `: OPENROUTER PROCESSING` followed by a blank line.
- The recordings have no timestamps, so they do not give the interval.
- Source: O-s55. Basis: OR-inferred (third-party recording; confidence high).

**O80. The pre-stream error envelope as recorded has a top-level `user_id` beside `error`.**

- Recorded body: `{"error":{"code":400,"message":"openai/gpt-nonexistent-routed-model is not a valid model ID"},"user_id":"user_REDACTED"}`. A second recorded body, for embeddings: `{"error":{"code":400,"message":"Model no-such/embedding-model does not exist"}}`.
- This matches the spec's `user_id` property on every error response schema (O33) and LiteLLM's code reading `error_chunk.get("user_id", "")` (O75). Neither recorded body has `metadata.error_type`, so the typed code is not on every error.
- Source: O-s55, O-s2. Basis: OR-inferred (from a third-party recording). Confidence note: high, since the spec declares the same `user_id` property.

**O81. Clients in the field detect OpenRouter context overflow by message text. The text names "this endpoint" and, for some backends, is the upstream provider's own wording.**

- Quote (pi, `packages/ai/src/utils/overflow.ts` lines 20-21): "OpenRouter: "This endpoint's maximum context length is X tokens. However, you requested about Y tokens"" and "OpenRouter/Poolside: "Input length X exceeds the maximum allowed input length of Y tokens.""
- Quote (the two regular expressions, lines 50-51): `/maximum context length is \d+ tokens/i, // OpenRouter (most backends)` and `/exceeds (?:the )?maximum allowed input length of [\d,]+ tokens?/i, // OpenRouter/Poolside`.
- What this shows: the overflow message is not uniform across OpenRouter's providers, which is the problem `error_type` exists to solve. The pi code does not read `error_type`.
- Source: O-s56. Basis: OR-inferred (from client code that special-cases OpenRouter; confidence medium, since the strings are a client author's observations).

**O82. Fill: how another gateway treats OpenRouter's keep-alive comment. Portkey's OpenRouter adapter turns it into an empty content chunk.**

- Quote (`src/providers/openrouter/chatComplete.ts` lines 254-268, two fragments): `if (chunk.includes('OPENROUTER PROCESSING')) {` and, inside the replacement chunk it builds, `object: 'chat.completion.chunk',` with `delta: { role: 'assistant', content: '' },` and `finish_reason: null,`.
- A fill shows one way to handle it. Forwarding the comment line unchanged is the other, and the SSE specification lets clients ignore it.
- Source: O-s57. Basis: Filled (Portkey gateway).

## Conflicts and disagreements

### Between OpenRouter's own sources

- **`native_finish_reason` is in the guide and on the wire, and not in the spec's chat schemas.** The API overview documents it on every choice (O52) and recorded streams contain it (O77). The spec's `ChatChoice` and `ChatStreamChoice` do not declare it (O53). The guide and the recordings are right about what is sent. The spec is incomplete, so SDKs generated from it drop the field.
- **`provider` on chat chunks, `preset` on the request, and `error.metadata.raw` and `provider_name`** are in the same position: described in guide prose or seen on the wire, and missing from the spec (O24, O38, O71).
- **The status for an overloaded provider.** The typed table says "`provider_overloaded` | 503". The spec lists a separate 529 response, "Provider Overloaded - Provider is temporarily overloaded", for `/chat/completions`, and the router metadata page names `529` among the 5xx classes. The sources do not say when a caller sees 503 and when 529.
- **The status for a timeout.** The typed table says "`timeout` | 504". The spec's list for `/chat/completions` has 408 and 524 and has no 504. The guide's status list has 408 only.
- **The Messages error type for a timeout.** The mapping table sends `timeout` to `overloaded_error`. The spec's `AnthropicMessagesErrorResponse` enum includes `timeout_error`. Either the table or the enum is behind.
- **The shape of the mid-stream error chunk.** The errors page and the streaming page show a full chunk with `id`, `object`, `model`, `provider`, a top-level `error`, and `choices` with `finish_reason: "error"` (O43). The OpenAI compatibility page shows a bare `data: {"error": {"code": 502, "message": "Provider disconnected mid-stream"}}` with no `choices`. The streaming page's own example also lacks `metadata.error_type` while the errors page's example has it. A client should accept both: any chunk with an `error` key is a failure.
- **Sort values.** The guide says "The three sort options are" `"price"`, `"throughput"`, `"latency"`. The spec's `ProviderSort` enum has a fourth, `"exacto"`, and the Exacto page calls the suffix "a shortcut for setting the provider sort to Exacto" (O2, O20).
- **Retention and routing.** The provider logging page says "OpenRouter does not have routing rules that change based on data retention policies of providers". The ZDR page and the provider routing page describe exactly such rules (`provider.zdr`, account and guardrail ZDR enforcement, O8 and O29). The ZDR pages are newer in content (they mention the October 27, 2026 CSAM update), so the logging page's sentence is stale. Source for the stale sentence: O-s19.
- **`openrouter_key_limit` appears under both 402 and 403.** The limits page lists it as a 402 `limit_source` and the budget error table lists it as a 403 `limit_source` (O37).
- **The `:online` equivalence.** The page says the suffix "is exactly equivalent to" a body whose model is `openrouter/auto` (O20). That cannot be exact for a suffix placed on another model. It reads as a slip in the example.
- **Sticky session lifetime.** The prompt caching page says "Sticky sessions expire after **60 minutes** of inactivity." The Pareto Router page says of its model and provider stickiness "the cache expires after **5 minutes** of inactivity" (O-s39). These may be two different caches. The pages do not say.
- **Unknown `provider.options` keys.** The spec says "Unrecognized keys are silently dropped." The extraction models guide says "anything else fails with a `400` naming the accepted keys." (O3).
- **Is `error.code` always a number?** The guide's type says `code: number`. The official AI SDK provider's schema accepts a string or a number (O74).

### Between today's sources and the local Runvil documents

- **LC8** (provider controls, hard `max_price`, soft performance preferences): holds (O11). Additions the survey did not list: `only` and `ignore` fail with 404 (O6), `enforce_distillable_text`, `provider.options`, and the `exacto` sort (O2, O3).
- **LC10** (both `finish_reason` and `native_finish_reason`): holds on Chat Completions, with three limits the survey did not state. The field is not in the spec's chat schemas, it does not exist on the Messages and Responses routes, and its vocabulary changes with the upstream protocol, not only the vendor (O53, O77). The survey's recommendation to "Emit a normalized class and the native code together, the way OpenRouter emits both `finish_reason` and `native_finish_reason`" should be read with O78 in mind: OpenRouter's normalized value can contradict the native one (`tool_calls` beside `length`).
- **LC11** (200 as soon as the provider accepts, failover stops once output has reached the caller, mid-stream error as a chunk with `finish_reason: "error"`): holds, quotes unchanged (O42, O43, O47). Addition: the same applies to non-streaming requests (O42, O44).
- **LC13** (402 `payment_required`, `limit_source` `openrouter_in_flight_budget`, 403 for budgets, `context_length_exceeded`): holds (O37). One sentence beside it in the survey no longer holds: "The older `in_flight_budget_exhausted` and `insufficient_credits` reason strings no longer appear in the docs". Today `in_flight_budget_exhausted` is a documented value of `error.metadata.reason`, next to `weight_exceeds_budget` (O37). `insufficient_credits` appears today only as a failure reason of the fusion server tool.
- **LC14** (no per-key rate limit): holds in substance (O30). The precise statement is that rate limits are governed per account and making more keys does not raise them. A key can still have a spend cap and a budget, which fail with 402 or 403, not 429.
- **LC15** (no markup, 5.5 percent on credits, 5 percent on BYOK above an allowance): holds. Today's detail: a 0.80 dollar minimum on the credit fee, and the allowance is 25,000 dollars per month of list-price inference on pay-as-you-go (O26).
- **LC16** (overflow advice moved from "middle-out" to the `context-compression` plugin): the end state holds (O60). The "between January and April 2026" dating cannot be confirmed from today's sources. The name `middle-out` survives as the plugin's engine.
- **LC18** (BYOK does not exempt a provider from retention policy unless ZDR is declared on the key): holds, word for word (O28).
- **LC2 and LC3**, as far as this topic touches them: the debug switch holds (O65). "Unsupported parameters are silently ignored" holds as the default for provider support of a parameter (O7), with exceptions that reject: router-specific reasoning fields return 400 (O21), extra keys in a Messages `fallbacks` entry return 400 (O18), and extraction models return 400 for unknown `provider.options` keys (O3).
- **The workspace report of 2026-09-30**: no disagreement on its three OpenRouter claims. Its exhaustion table row that groups OpenRouter with OpenAI's 429 quota codes does not describe OpenRouter's own 402 and 403 signals (O76).

## Open questions blocked by closed source

1. **The full table from vendor stop reasons to `finish_reason`** (Anthropic `pause_turn`, `model_context_window_exceeded`, `stop_sequence`, `compaction`; Gemini `RECITATION`, `MALFORMED_FUNCTION_CALL`, and the rest). Tried: the guides, the spec, the cookbook, the official AI SDK provider. Found only the rows in O54. Fill found: LiteLLM's `_FINISH_REASON_MAP` and the Vercel AI SDK's three mappers, which disagree on `pause_turn`, `compaction`, and `MALFORMED_FUNCTION_CALL` (O57). Partial ground truth from third-party recordings for `end_turn`, `tool_use`, `STOP`, `completed`, `max_output_tokens` (O77).
2. **Whether `data: [DONE]` and a usage chunk follow a mid-stream error chunk.** Tried: the errors and streaming pages (they say only "The stream is terminated after this event"), the official AI SDK provider (a hand-written test models an error finish followed by a usage chunk and `[DONE]`, O74), the rig recordings (they contain no mid-stream error). No reliable fill found. Both open clients read (the official provider and LiteLLM) treat the error chunk itself as the end and do not depend on `[DONE]`.
3. **How OpenRouter decides between `tool_calls` and `length` when both apply, and between `stop` and `tool_calls` for Gemini.** Tried: the guides and spec (silent). Evidence found: a recorded truncated tool call reported as `tool_calls` (O78) and the official client's override for Gemini (O58). Fill found: the Vercel AI SDK derives `tool-calls` from the presence of tool calls for Gemini `STOP` and for OpenAI Responses (O57), which is the same rule OpenRouter appears to apply.
4. **The keep-alive interval and any request, idle, or total stream timeout.** Tried: the guides, the API reference, the spec, the provider integration guide (it says a silent provider is cancelled "with a fetch timeout" and gives no number), the rig recordings (no timestamps). No fill found.
5. **Whether the server still accepts `transforms: ["middle-out"]`.** Tried: the spec (zero matches) and the guides (plugin only). Evidence found: LiteLLM still forwards the field (O60). No authoritative fill found.
6. **When `error.metadata.provider_code` and `error.metadata.raw` are present, and their shape per vendor.** Tried: the errors page ("when available") and the spec (types `provider_code` only). Fill found: the official AI SDK provider's extractor, which treats `raw` as a string, a JSON string, or a nested object and looks for `message`, `error`, `detail`, `details`, `msg` (O38).
7. **How OpenRouter recognizes a vendor's overflow error** (by vendor code, by message text, or by its own token count before sending), and so how it tracks vendor wording changes. Tried: the guides ("Context length validation errors" suggests a pre-send check exists for some cases). Fill found: pi's regular-expression list and LiteLLM's substring list show the text-matching approach that clients use (O81, and the workspace report of 2026-09-30).
8. **Which `provider.options` keys each chat endpoint forwards.** Tried: the spec (free-form objects), the guides (documented for speech, image, video, and extraction only, where an `allowed_passthrough_parameters` field lists the keys), and the public endpoints listing for one chat model (`GET /api/v1/models/google/gemini-2.5-flash/endpoints`, HTTP 200 without credentials on 2026-11-22, O-s58), whose endpoint records have no `allowed_passthrough_parameters` field. So the allowlist for chat endpoints is not published where a caller can read it. No fill found.
9. **Whether vendor response headers (vendor rate-limit headers, vendor request ids) are forwarded.** Tried: the limits and errors pages (they describe only OpenRouter's own `X-RateLimit-*`, `Retry-After`, and `X-Generation-Id`). The vendor request id is available afterwards as `upstream_id` on the generation record (O39). No fill found for headers.

## Unverified

- From memory, not found in today's sources: older OpenRouter documentation described a top-level request field `transforms: ["middle-out"]` and a `ProviderErrorMetadata` type with `provider_name` and `raw`. Today's pages mention `metadata.raw` and `metadata.provider_name` in prose (O38) and do not print that type.
- The date of the rig recordings (late September 2026) is inferred from the Unix time inside the generation ids, not stated by the project.
- Anthropic's prompt-too-long wording ("prompt is too long") was taken from the workspace report of 2026-09-30 and from pi's source comment, and was not re-fetched from Anthropic today.
- The OpenAI finish-reason text was read from the `openai/openai-openapi` repository's `manual_spec` branch. Whether that branch matches OpenAI's live reference today was not checked.
- That vendor rate-limit headers are dropped is an inference from silence (O32).
- The correspondence between Anthropic's error types and OpenRouter's canonical types in O73 is assembled from two published tables, not stated by OpenRouter for the Chat Completions route.
- No request was sent to any OpenRouter inference endpoint, so every wire behavior here rests on documentation, the spec, SDK source, or third-party recordings.

## Workspace documents used

- `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (2026-09-30). Took: its three OpenRouter claims (the `error_type` overflow value, the in-stream error chunk, the move from middle-out to the plugin) and Anthropic's overflow wording. Re-checked live: the first two fully, the third for its end state only (O76).
- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took: the LC claims to verify, and its line on reason strings. Re-checked live: LC8, LC10, LC11, LC13, LC14, LC15, LC16, LC18 (see Conflicts).
- `runvil-design`, `decisions/gateway.md` (read 2026-11-22, searched for error, finish-reason, and retry terms). Took: only that the Gateway already answers a loading model with 503, a code, and `Retry-After: 5`. Not an OpenRouter claim, so nothing to re-check.

## Sources

All sources were read on 2026-11-22. OpenRouter documentation pages were read from the Markdown mirror fetched that day; the URL given is the human page.

| Key | URL, or repository at commit with file path | Read date | Kind |
| - | - | - | - |
| O-s1 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| O-s2 | https://openrouter.ai/openapi.json (OpenAPI 3.1, 123 paths, 1071 schemas; schemas named in each item: `ProviderPreferences`, `ProviderOptions`, `ProviderSort`, `ProviderSortConfig`, `PreferredMaxLatency`, `PercentileLatencyCutoffs`, `Quantization`, `DeprecatedRoute`, `ChatRequest`, `ChatModelNames`, `ChatChoice`, `ChatStreamChoice`, `ChatStreamChunk`, `ChatResult`, `ChatFinishReasonEnum`, `FinishReason`, `ORAnthropicStopReason`, `IncompleteDetails`, `ApiErrorType`, `BadGatewayResponse` and the other per-status error schemas, `AnthropicMessagesErrorResponse`, `MessagesErrorEvent`, `ResponsesErrorField`, `OpenResponsesErrorField`, `ChatDebugOptions`, `DebugEvent`, `ContextCompressionPlugin`, `ContextCompressionEngine`, `GenerationResponse`, `ProviderResponse`, `OpenRouterMetadata`, `TraceConfig`) | 2026-11-22 | OpenRouter spec |
| O-s3 | https://openrouter.ai/docs/guides/providers/traffic-routing | 2026-11-22 | OpenRouter doc |
| O-s4 | https://openrouter.ai/docs/guides/routing/model-fallbacks | 2026-11-22 | OpenRouter doc |
| O-s5 | https://openrouter.ai/docs/guides/routing/model-variants/overview | 2026-11-22 | OpenRouter doc |
| O-s6 | https://openrouter.ai/docs/guides/routing/model-variants/nitro | 2026-11-22 | OpenRouter doc |
| O-s7 | https://openrouter.ai/docs/guides/routing/model-variants/floor | 2026-11-22 | OpenRouter doc |
| O-s8 | https://openrouter.ai/docs/guides/routing/model-variants/free | 2026-11-22 | OpenRouter doc |
| O-s9 | https://openrouter.ai/docs/guides/routing/model-variants/exacto | 2026-11-22 | OpenRouter doc |
| O-s10 | https://openrouter.ai/docs/guides/routing/model-variants/online | 2026-11-22 | OpenRouter doc |
| O-s11 | https://openrouter.ai/docs/guides/routing/model-variants/thinking | 2026-11-22 | OpenRouter doc |
| O-s12 | https://openrouter.ai/docs/guides/routing/model-variants/extended | 2026-11-22 | OpenRouter doc |
| O-s13 | https://openrouter.ai/docs/guides/routing/routers/overview | 2026-11-22 | OpenRouter doc |
| O-s14 | https://openrouter.ai/docs/guides/routing/routers/auto-router | 2026-11-22 | OpenRouter doc |
| O-s15 | https://openrouter.ai/docs/guides/routing/routers/router-metadata | 2026-11-22 | OpenRouter doc |
| O-s16 | https://openrouter.ai/docs/guides/features/presets | 2026-11-22 | OpenRouter doc |
| O-s17 | https://openrouter.ai/docs/guides/overview/auth/byok | 2026-11-22 | OpenRouter doc |
| O-s18 | https://openrouter.ai/docs/guides/privacy/data-collection | 2026-11-22 | OpenRouter doc |
| O-s19 | https://openrouter.ai/docs/guides/privacy/provider-logging | 2026-11-22 | OpenRouter doc |
| O-s20 | https://openrouter.ai/docs/guides/features/zdr | 2026-11-22 | OpenRouter doc |
| O-s21 | https://openrouter.ai/docs/api_reference/limits | 2026-11-22 | OpenRouter doc |
| O-s22 | https://openrouter.ai/docs/api_reference/errors-and-debugging | 2026-11-22 | OpenRouter doc |
| O-s23 | https://openrouter.ai/docs/api_reference/streaming | 2026-11-22 | OpenRouter doc |
| O-s24 | https://openrouter.ai/docs/api_reference/responses/error-handling | 2026-11-22 | OpenRouter doc |
| O-s25 | https://openrouter.ai/docs/guides/features/message-transforms | 2026-11-22 | OpenRouter doc |
| O-s26 | https://openrouter.ai/docs/api_reference/overview | 2026-11-22 | OpenRouter doc |
| O-s27 | https://openrouter.ai/docs/changelog (entries from July 3, 2026 to November 21, 2026, generated from spec diffs) | 2026-11-22 | OpenRouter doc |
| O-s28 | https://openrouter.ai/docs/guides/features/zero-completion-insurance | 2026-11-22 | OpenRouter doc |
| O-s29 | https://openrouter.ai/docs/guides/features/service-tiers | 2026-11-22 | OpenRouter doc |
| O-s30 | https://openrouter.ai/docs/guides/best-practices/latency-and-performance | 2026-11-22 | OpenRouter doc |
| O-s31 | https://openrouter.ai/docs/guides/features/latest-resolution | 2026-11-22 | OpenRouter doc |
| O-s32 | https://openrouter.ai/docs/faq | 2026-11-22 | OpenRouter doc |
| O-s33 | https://openrouter.ai/docs/guides/features/plugins | 2026-11-22 | OpenRouter doc |
| O-s34 | https://openrouter.ai/docs/guides/overview/multimodal/stt | 2026-11-22 | OpenRouter doc |
| O-s35 | https://openrouter.ai/docs/guides/providers/integration | 2026-11-22 | OpenRouter doc |
| O-s36 | https://openrouter.ai/docs/guides/community/extractions | 2026-11-22 | OpenRouter doc |
| O-s37 | https://openrouter.ai/docs/guides/features/compatibility | 2026-11-22 | OpenRouter doc |
| O-s38 | https://openrouter.ai/docs/guides/routing/routers/jev-router | 2026-11-22 | OpenRouter doc |
| O-s39 | https://openrouter.ai/docs/guides/routing/routers/pareto-router | 2026-11-22 | OpenRouter doc |
| O-s40 | https://openrouter.ai/docs/guides/best-practices/prompt-caching | 2026-11-22 | OpenRouter doc |
| O-s41 | https://openrouter.ai/docs/guides/routing/auto-exacto | 2026-11-22 | OpenRouter doc |
| O-s42 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gemini-4 | 2026-11-22 | OpenRouter doc |
| O-s43 | https://openrouter.ai/docs/guides/features/server-tools/tool-search | 2026-11-22 | OpenRouter doc |
| O-s44 | https://openrouter.ai/docs/cookbook/image-understanding/find-objects-in-images | 2026-11-22 | OpenRouter doc |
| O-s45 | https://github.com/OpenRouterTeam/ai-sdk-provider at commit `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20): `src/utils/map-finish-reason.ts`, `src/chat/index.ts`, `src/schemas/error-response.ts`, `src/chat/index.test.ts` | 2026-11-22 | OpenRouter SDK source |
| O-s46 | https://github.com/BerriAI/litellm at commit `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22): `litellm/litellm_core_utils/core_helpers.py`, `litellm/llms/openrouter/chat/transformation.py` | 2026-11-22 | gateway source |
| O-s47 | https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons | 2026-11-22 | vendor doc |
| O-s48 | https://platform.claude.com/docs/en/api/errors | 2026-11-22 | vendor doc |
| O-s49 | https://ai.google.dev/api/generate-content (Markdown form, redirected to https://aistudio.google.com/api-reference/generate-content.md) | 2026-11-22 | vendor doc |
| O-s50 | https://github.com/openai/openai-openapi, branch `manual_spec` at commit `498c71ddf6f1c45b983f972ccabca795da211a3e`: `openapi.yaml` | 2026-11-22 | vendor doc |
| O-s51 | `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (dated 2026-09-30) | 2026-11-22 | workspace document |
| O-s52 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (dated 2026-10-28) | 2026-11-22 | workspace document |
| O-s53 | `runvil-design`, `decisions/gateway.md` | 2026-11-22 | workspace document |
| O-s54 | https://github.com/vercel/ai at commit `d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d` (2026-11-20): `packages/anthropic/src/map-anthropic-stop-reason.ts`, `packages/google/src/map-google-finish-reason.ts`, `packages/openai/src/responses/map-openai-responses-finish-reason.ts` | 2026-11-22 | harness source |
| O-s55 | https://github.com/0xPlaygrounds/rig at commit `a8673f775ca78004f011e5ad5b615147fdcd4bff` (2026-11-22): `crates/rig-cassette/fixtures/cassettes/openrouter/` (63 files) and `crates/rig-cassette/fixtures/bank/openrouter.yaml` | 2026-11-22 | harness source (third-party recordings of OpenRouter responses) |
| O-s56 | https://github.com/badlogic/pi-mono at commit `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (2026-11-22): `packages/ai/src/utils/overflow.ts` | 2026-11-22 | harness source |
| O-s57 | https://github.com/Portkey-AI/gateway at commit `669825cbe89ee51569918b8f78a9db486fd69dd4` (2026-05-25): `src/providers/openrouter/chatComplete.ts` | 2026-11-22 | gateway source |
| O-s58 | https://openrouter.ai/api/v1/models/google/gemini-2.5-flash/endpoints (public endpoints listing, HTTP 200 without credentials) | 2026-11-22 | OpenRouter spec (live catalog response) |
