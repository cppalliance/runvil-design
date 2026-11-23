---
produced: 2026-11-22
title: OpenRouter model discovery - models list API, endpoints API, supported_parameters vocabulary, reasoning object, per-vendor-family catalog tables, per-provider differences, variants and latest aliases, flagship fact sheet, Runvil provider sheet reader
---

# OpenRouter model discovery - models list API, endpoints API, supported_parameters vocabulary, reasoning object, per-vendor-family catalog tables, per-provider differences, variants and latest aliases, flagship fact sheet, Runvil provider sheet reader

- What was read: the public catalog snapshot (`GET https://openrouter.ai/api/v1/models`, 476 entries), 59 per-model endpoint listings (490 distinct endpoint records from 65 providers), the ZDR endpoint list (936 records), the providers list (123), 21 single-model lookups, 14 filter and count probes, 22 OpenRouter documentation pages, the OpenAPI spec, the API changelog, the official TypeScript SDK, Runvil's OpenRouter provider code, and three harness catalog readers (pi, Zed, Roo Code).
- Sources: 42, listed at the end. Every OpenRouter read was public and sent without a credential.
- Read date: 2026-11-22. The catalog and every endpoint listing are a snapshot of that day. OpenRouter says the catalog "updates continuously as providers deploy new models, adjust pricing, and add hardware endpoints" (M-s3), so every count below is true for 2026-11-22 only.

How to read the counts: "interactive entries" means the 365 catalog entries that are not a `:batch` variant and not a `~` alias (347 base, 11 `:free`, 7 `openrouter/` routers). Data computed from the catalog or from an endpoint listing has basis `OR-documented`, because it is OpenRouter's own published data. Scripts and raw output are in the topic's scratch subfolder `M`.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|-|-|-|-|-|-|-|-|
| Sampling parameters (`temperature`, `top_p`, `top_k`) accepted or removed | Anthropic (Opus 4.7 and later), OpenAI (GPT-5 line and later, o3); every other family lists them | OpenRouter's account of the vendor rule: Claude 4.7 Opus "no longer accepts `temperature`, `top_p`, or `top_k`"; Haiku 5.5 "rejects any `temperature` other than `1`" | `supported_parameters` omits `temperature` on 10 of 16 Anthropic and 43 of 62 OpenAI interactive entries; all Google, xAI, DeepSeek, Mistral, Moonshot, Qwen, Z.ai entries list it | The list is names only: value ranges and "only the default is accepted" are not in the catalog. Unlisted parameters still route unless `provider.require_parameters` is `true` | Caller holds nothing. A neutral setting can be dropped by the gateway per model from catalog data. No round trip. Any route | OR-documented | M13, M33, M35 |
| Reasoning that cannot be turned off | Counts of interactive entries with `mandatory: true`: Anthropic 5 (Fable 5, 5.1, 5.2, Opus 5.5, Sonnet 5.5), OpenAI 27, Google 17, xAI 4, Z.ai 5 (GLM 5.3 line), Meta 9, MiniMax 3, Perplexity 3, Qwen 4, Moonshot 2, DeepSeek 1 | OpenRouter's account: the model "is registered as mandatory-reasoning" | `reasoning.mandatory: true` on 99 of 252 interactive entries that have a `reasoning` object; a disable request on a concrete slug returns 400 `Reasoning is mandatory for this endpoint and cannot be disabled.`; a `~latest` alias coerces it to the lowest effort | The same request field (`reasoning.enabled: false`, `effort: "none"`) is valid on one model and a 400 on the next | A neutral on/off switch translates only when the gateway knows `mandatory` per model. Gateway holds the catalog. No round trip | OR-documented | M15, M36, M55 |
| Reasoning effort vocabulary | All reasoning families | Each vendor has its own level set | `reasoning.supported_efforts` (31 distinct lists in the snapshot, all drawn from `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`), `reasoning.default_effort`; "unsupported non-`none` efforts are mapped to the nearest supported level on all slugs" | One seven-word scale over all vendors. Token-budget reasoning shows only as `supports_max_tokens: true` (8 entries) | A neutral effort word can be translated by the gateway. The per-model list is needed to show valid choices | OR-documented | M15, M37, M55 |
| Reasoning default state | All reasoning families | Vendor default differs (on or off) | `reasoning.default_enabled` (true on 106, false on 20, key absent on 126 of 252) | Half of the reasoning entries publish no default state | A caller that sends nothing gets the vendor default. The gateway can make it explicit only where the key is present | OR-documented | M15, M36 |
| `verbosity` | OpenAI, Anthropic | OpenRouter's account: "Introduced by OpenAI for the Responses API"; "For Anthropic models, this parameter maps to `output_config.effort`" | Listed by 42 of 62 OpenAI and 13 of 16 Anthropic interactive entries, and by 0 entries of Google, xAI, DeepSeek, Mistral, Moonshot, Qwen, Z.ai | One parameter name means output verbosity on OpenAI and effort on Anthropic | Caller picks from `low`, `medium`, `high`, `xhigh`, `max`; which values work depends on the model | OR-documented | M11, M33 |
| Forced tool choice (`required`, named function) | Anthropic (Fable 5.2, Opus 5.5, Sonnet 5.5, Haiku 5.5 per endpoint data), many open-weight hosts | OpenRouter's account: Opus 5.5 rejects forced tool choice with the provider's 400 `tool_choice: type "tool" and "any" are not supported for this model.` | `tool_choice` in `supported_parameters` "only says the parameter is accepted"; the per-endpoint object `supports_tool_choice` `{none, auto, required, function}` says "which of its values passed testing" | The model-level list hides it: all 16 Anthropic entries list `tool_choice`. The value flags exist only on the endpoints route | The caller or gateway must know the model (and for open-weight models the provider) before sending `required`. A neutral "must call a tool" has no translation on these models | OR-documented | M24, M44, M50 |
| `parallel_tool_calls` | All | Not read for this topic | Listed by 14 of 365 interactive entries: 0 Anthropic, 0 OpenAI, 0 Google, 3 DeepSeek, 2 Moonshot, 2 Z.ai, 3 routers, 4 others | The catalog gives no signal for the three largest vendors | The catalog cannot tell a gateway whether the switch works | OR-documented | M33, M34 |
| Structured output per provider | Open-weight models, Claude on Amazon Bedrock, GPT on Amazon Bedrock | Not read for this topic | `structured_outputs` and `response_format` appear at model level when any endpoint lists them; "Support is determined per endpoint, not just per model"; response format is a soft routing preference | A model that lists `structured_outputs` can still be served by an endpoint that lacks it when the caller pins that provider | Gateway-side routing settles it when the caller leaves provider choice open | OR-documented | M12, M13, M50 |
| `max_tokens` versus `max_completion_tokens` | OpenAI direct versus Azure | Not read for this topic | For `openai/gpt-5.2`, `openai/gpt-5.4` and `openai/gpt-4.1` the OpenAI endpoints list `max_tokens` and the Azure endpoint lists `max_completion_tokens`; docs: `max_completion_tokens` "shares `max_tokens` semantics" | The model-level list shows both names | A rename inside the gateway. No caller knowledge needed | OR-documented | M11, M45 |
| `seed`, `stop`, penalties, `logprobs` | Per family | Not read for this topic | `seed`: 0 of 16 Anthropic, 62 of 62 OpenAI; `stop`: 16 of 16 Anthropic, 14 of 62 OpenAI; `frequency_penalty`: 0 Anthropic, 0 xAI, 0 Gemini; `logprobs`: 0 Anthropic, 6 of 6 xAI | Listed names say nothing about limits such as the number of stop sequences | Neutral names. The gateway can drop what a model does not list | OR-documented | M33, M34 |
| Web search discovery | OpenAI (GPT-4o search), Perplexity, and native search tools on Anthropic, Google, OpenAI, xAI | Vendor-run search tools | `web_search_options` in `supported_parameters` (16 entries); `pricing.web_search` (113 interactive entries); per-endpoint `native_tools`, for example `{"openrouter:web_search": {"type": "web_search_20260209"}}` | Three unrelated fields signal one capability. 104 entries have a `web_search` price and do not list `web_search_options` | Which mechanism applies depends on the model and on the serving provider | OR-documented | M18, M25, M34 |
| Prompt cache pricing and implicit caching | Anthropic (two write TTL prices), OpenAI, Google, xAI, open-weight hosts | Not read for this topic | `pricing.input_cache_read`, `pricing.input_cache_write` ("the default (5-minute) cache-write rate"), `pricing.input_cache_write_1h`; per-endpoint `supports_implicit_caching` | The spec gives `supports_implicit_caching` a type and no description. All 29 Claude endpoints and all 24 Gemini endpoints fetched report `false`; GPT-5.2, GPT-5.4 and GPT-4.1 endpoints report `true`; GPT-6.1 and GPT-6.2 endpoints report `false` | Whether a request needs cache markers is a per-endpoint fact. The caller cannot see it in the models list | OR-documented | M18, M30, M40 |
| Context and output limits per provider | Open-weight models most of all | Each host sets its own limits | Endpoint `context_length`, `max_completion_tokens`, `max_prompt_tokens`; model `context_length` equals the largest endpoint value; `top_provider` holds one provider's numbers | For `z-ai/glm-5.3` the 46 endpoints span context 262144 to 1048576 and max output 32768 to 943718; the models list shows one pair | The caller learns the serving provider only from the response. A fixed budget is safe only against the smallest endpoint or with a pinned provider | OR-documented | M19, M31, M49 |
| Quantization | Open-weight models | Each host picks a precision | Endpoint `quantization` enum (`int4`, `int8`, `fp4`, `mxfp4`, `nvfp4`, `fp6`, `fp8`, `mxfp8`, `fp16`, `bf16`, `fp32`, `unknown`) | Quantization is published on the endpoints route only | Provider pinning or a quantization filter is a request-time routing control | OR-documented | M27, M49 |
| Routes a model can be reached on | All | Vendor-native protocols differ | `endpoints` array per model. All 384 non-batch entries list all four of `/api/v1/chat/completions`, `/api/v1/completions`, `/api/v1/messages`, `/api/v1/responses` | The field does not separate vendors. It separates batch, image, video and extraction routes | Route choice is free at catalog level. The field says nothing about what a route loses for a vendor | OR-documented | M16, M38 |
| Model identity: `id`, `canonical_slug`, aliases | All | Vendors publish dated snapshot ids | `id` is "The routable slug used in API requests"; `canonical_slug` is the "Permanent slug for the model that never changes"; known aliases resolve | The vendor's own snapshot id is not a field. The single-model lookup returns 404 for a canonical slug while the endpoints route accepts it | A gateway that stores a model name should store `id` and may log `canonical_slug` | OR-documented | M22, M53, M54 |
| Variant suffixes | All | Not a vendor feature | Catalog variants `:free`, `:batch` (own entries); routing variants `:nitro`, `:floor`, `:exacto`, `:online` (never entries); `:thinking`, `:extended` deprecated with 0 entries | A catalog built from the list misses every routing-variant string a request may use | Resolve metadata by keeping the catalog suffix and dropping routing suffixes. An unknown suffix is dropped by the lookup route | OR-documented | M51, M52, M53 |
| `~author/family-latest` aliases | Anthropic, OpenAI, Google, xAI, DeepSeek, Moonshot, Z.ai, Unbiased | Not a vendor feature | 19 entries with `alias_target` `{slug, name}`; response `model` reports the concrete model; reasoning parameters are remapped only on these slugs | The `pricing` of 5 alias rows differs from the target row in the snapshot. The endpoints route returns an empty list for an alias | A role name that maps to "latest" trades strict validation for continuity. The concrete model arrives in the response | OR-documented | M55 |
| Service tiers as separate endpoints | OpenAI (`flex`, `priority`, `ultrafast`), Anthropic (`priority`), Google, xAI, open-weight hosts | Vendor tier selected by request parameter or by a dedicated model id | Endpoint `service_tier` (`flex`, `priority`, `ultrafast`, `null`) and tags such as `openai/fast`, `anthropic/fast`; each tier has its own price | The models list shows the default price only | Tier is a routing choice with a price effect. The response reports the tier that served | OR-documented | M29, M44, M45 |
| Deprecation date | Any | Vendors announce retirements | `expiration_date` (16 of 476 entries), fed by a provider's `deprecation_date` | The field names a date and names no successor | A gateway can warn. A replacement must come from elsewhere | OR-documented | M21 |
| Router and composite entries | OpenRouter, NVIDIA, TypeSafe | Not a vendor feature | `pricing.prompt: "-1"` on 8 entries; `architecture.tokenizer: "Router"`; the `reasoning` object is omitted | `supported_parameters` of a router is not tied to the model that will answer | A client that sorts by price or reads capabilities must special-case these rows | OR-documented | M18, M56 |
| Key-scoped availability | Any | Not a vendor feature | `GET /api/v1/models/user` needs a key and filters by provider preferences, privacy settings and guardrails | The public catalog says nothing about what a given key can reach | A gateway that lists models for a user needs the authenticated route | OR-documented | M2, M7 |

## Evidence

### A. Discovery routes and which of them are public

**M1.** OpenRouter documents eleven catalog routes and marks all but one as usable without a key.

- Quote (table "Catalog endpoints", columns Method, Endpoint, Auth, Purpose): "`GET` | `/api/v1/models` | Optional | Public catalog with server-side filtering, sorting, and pagination."
- Quote: "`GET` | `/api/v1/models/user` | Required | Catalog filtered to models accessible by your API key, reflecting your workspace guardrails, provider preferences, and privacy settings."
- Quote: "`GET` | `/api/v1/model/{author}/{slug}` | Optional | Detailed record for a single model, resolving aliases and variants automatically."
- Quote: "`GET` | `/api/v1/models/{author}/{slug}/endpoints` | Optional | Per-provider endpoints for a model with quantization, pricing, context limits, and availability."
- The other rows, all "Optional": `/api/v1/models/count`, `/api/v1/endpoints/zdr`, `/api/v1/images/models`, `/api/v1/videos/models`, `/api/v1/embeddings/models`, `/api/v1/audio/speech/models`, `/api/v1/audio/speech/models/{author}/{slug}`.
- Source: M-s3. Basis: OR-documented.

**M2.** Live probe on 2026-11-22 with no `Authorization` header, no cookie and no key: ten discovery routes answer 200 and two answer 401.

| Route (all `GET https://openrouter.ai...`) | Status | Body |
|-|-|-|
| `/api/v1/models` | 200 | 476 entries, `"total_count": 476`, `"links": {"next": null}` |
| `/api/v1/models?output_modalities=all` | 200 | 739 entries |
| `/api/v1/models/count` | 200 | `{"data":{"count":476}}` |
| `/api/v1/models/count?output_modalities=all` | 200 | `{"data":{"count":739}}` |
| `/api/v1/model/openai/gpt-5.2` (and 20 more slugs, see M53) | 200 or 404 by slug | One model entry under `data` |
| `/api/v1/models/meta-llama/llama-3.3-70b-instruct/endpoints` (and 58 more slugs) | 200 for all 59 | Model header plus an `endpoints` array |
| `/api/v1/providers` | 200 | 123 providers |
| `/api/v1/endpoints/zdr` | 200 | 936 endpoint records |
| `/api/v1/embeddings/models` | 200 | 40 entries, `"total_count": 40` |
| `/api/v1/images/models`, `/api/v1/videos/models`, `/api/v1/audio/speech/models` | 200, 200, 200 | 57 KB, 29 KB and 68 KB of JSON (not analyzed) |
| `/api/v1/tools` | 401 | `{"error":{"message":"No cookie auth credentials found","code":401}}` |
| `/api/v1/models/user` | 401 | `{"error":{"message":"Unauthorized","code":401}}` |

- Source: M-s25, M-s26. Basis: OR-documented (live public API responses). This confirms local survey claim LC1 for the catalog route.

**M3.** The OpenAPI spec declares a key for these routes even though they answer without one.

- The spec's top level has `"security": [{"apiKey": []}]`, and `securitySchemes.apiKey.description` is "API key as bearer token in Authorization header". The operations `getModels`, `getModel`, `listModelsCount`, `listEndpoints`, `listEndpointsZdr`, `listProviders`, `listEmbeddingsModels`, `listImageModels` and `listVideosModels` declare no `security` of their own, so they inherit the global one. `listModelsUser`, `listTools` and `getTool` declare `"security": [{"bearer": []}]`. `listSpeechModels` and `getSpeechModel` declare `"security": []`.
- Changelog, November 11, 2026, under "New response codes": the entries for `GET /endpoints/zdr` and `GET /models/{author}/{slug}/endpoints` each read "now returns `401`". Changelog, October 28, 2026, same heading: the entry for `GET /models` reads "now returns `401`".
- Source: M-s24, M-s23. Basis: OR-documented. The mismatch with M2 is listed under Conflicts.

**M4.** The default list holds text-output models only. Other modalities need a query parameter.

- Quote: "When the parameter is omitted, the endpoint returns text-output models only."
- Quote: "Pass `output_modalities=all` to receive the complete multi-modal catalog".
- Snapshot: default 476, `all` 739. In the `all` list the `architecture.output_modalities` values are `["text"]` 457, `["image"]` 61, `["embeddings"]` 45, `["decisions"]` 45, `["speech"]` 42, `["transcription"]` 30, `["video"]` 28, `["rerank"]` 12, and 19 mixed text entries. The documentation's own example output shows `"model_count": 518` for the default list and `774` for `all`, and lists sample ids (`near-ai/qwen3.8-27b-cyber`) that the 2026-11-22 unauthenticated lists do not contain.
- The count route takes the same `output_modalities` filter, in the guide's words "so that counts stay consistent with list results."
- Source: M-s2, M-s3, M-s7, M-s1, M-s25. Basis: OR-documented.

**M5.** The list route filters and sorts on the server. An unknown `endpoints` value is an error and an unknown `supported_parameters` value is accepted.

- Documented query parameters of `GET /api/v1/models` (M-s3): `output_modalities`, `input_modalities`, `endpoints`, `supported_parameters`, `sort`, `min_price`, `max_price`, `context`, `zdr`, `region`, `q`, `offset`, `limit`. The spec adds `category`, `use_rss`, `use_rss_chat_links`, `arch`, `model_authors`, `providers`, `distillable`, `min_output_price`, `max_output_price`, `min_age_days`, `max_age_days`, `min_tool_success_rate`, `max_tool_success_rate`, and a minimum and maximum for each Artificial Analysis index (M-s4).
- Quote for `supported_parameters`: "Comma-separated API parameters required (e.g. `tools`, `structured_outputs`, `reasoning`)."
- Quote for `endpoints`: "Accepts a comma-separated list; a model must serve every listed path." and "an unknown path returns `400`".
- Quote for `zdr` and `region`: "This filters the catalog only; it doesn't restrict routing".
- Live, 2026-11-22: `?endpoints=/api/v1/bogus` returned 400 with a `ZodError` whose message begins `Invalid endpoints value: \"/api/v1/bogus\". Valid values are: /api/v1/chat/completions, /api/v1/completions, /api/v1/messages, /api/v1/responses, /api/v1/embeddings, /api/v1/rerank, /api/alpha/decisions, /api/alpha/extractions, /api/v1/videos, /api/v1/audio/speech, /api/v1/audio/transcriptions, /api/v1/images, /api/v1/batches`. `?supported_parameters=bogus_param` returned 200 with 448 entries: the full list minus the 28 entries whose `architecture.tokenizer` is `Router`. `?supported_parameters=strict_tool_schemas` returned 200 with 0 entries. `?endpoints=/api/v1/messages`, `?endpoints=/api/v1/responses` and `?endpoints=/api/v1/chat/completions` each returned the same 384 entries. `?endpoints=/api/v1/batches` returned 97.
- Source: M-s2, M-s3, M-s4, M-s25. Basis: OR-documented.

**M6.** The response envelope is `data`, `total_count`, `links.next`. Pagination is opt-in and the public list is cached for two minutes.

- Quote: "Pagination is opt-in: when both are omitted, the full list is returned and `links.next` is `null`. When you paginate, `limit` defaults to 500 (max 1000)".
- Quote: "This schema is cached at the edge and designed for reliable integration with production applications."
- Live: `?offset=0&limit=2` returned 2 entries with `"links": {"next": "/api/v1/models?offset=2&limit=2"}` and `"total_count": 476`. Response headers: `/api/v1/models` and `/api/v1/models/count` sent `Cache-Control: public, max-age=120, stale-while-revalidate=3600, stale-if-error=3600`; `/api/v1/providers` sent `max-age=300`; the endpoints routes sent `Cache-Control: private, no-store`.
- Source: M-s2, M-s25. Basis: OR-documented.

**M7.** The key-scoped list is a separate route that requires a key.

- Quote (`GET /models/user` summary): "List models filtered by user provider preferences, privacy settings, and guardrails". Quote (description): "If requesting through a regional hostname, the results will be filtered to models that satisfy in-region routing for that region."
- Live: 401 `{"error":{"message":"Unauthorized","code":401}}` (M2).
- Source: M-s8, M-s25. Basis: OR-documented.

**M8.** `GET /api/v1/providers` lists every provider with policy links and locations.

- Each of the 123 records has exactly these keys: `name`, `slug`, `privacy_policy_url`, `terms_of_service_url`, `status_page_url`, `headquarters`, `datacenters`. The spec describes `datacenters` as "ISO 3166-1 Alpha-2 country codes of the provider datacenter locations". Example record: `{"name": "ParetoAlpha", "slug": "pareto-alpha", ..., "headquarters": "US", "datacenters": ["US"]}`.
- The spec's `ProviderName` enum (used by `provider_name` on an endpoint) has 126 values, including "Anthropic", "Amazon Bedrock", "Claude Platform on AWS", "Google", "Google AI Studio", "Azure", "OpenAI", "xAI", "Mistral", "DeepSeek", "Moonshot AI", "Z.AI", "Alibaba".
- Source: M-s10, M-s28, M-s5. Basis: OR-documented.

**M9.** `GET /api/v1/endpoints/zdr` returns endpoint records in the same shape as the per-model route, plus one extra key.

- The spec summary is "Preview the impact of ZDR on the available endpoints". The 936 records cover 366 model ids and 61 providers. Each record has the endpoint keys of M23 plus `safety_retention` (true on 53, false on 883).
- Source: M-s9, M-s27. Basis: OR-documented.

### B. Every field of a model entry

**M10.** A model entry has 21 documented fields. The spec marks 14 as required.

Spec `Model.required`: `id`, `canonical_slug`, `name`, `created`, `pricing`, `context_length`, `architecture`, `top_provider`, `per_request_limits`, `supported_parameters`, `default_parameters`, `supported_voices`, `links`, `endpoints`.

| Field | Type in spec | Meaning the spec or guide gives (quoted) | Snapshot, 476 entries |
|-|-|-|-|
| `id` | string | Spec: "Unique identifier for the model". Guide: "Unique model identifier used in API requests" | 476 present |
| `canonical_slug` | string | Spec: "Canonical slug for the model". Guide: "Permanent slug for the model that never changes" | 476 present, see M22 |
| `hugging_face_id` | string or null | "Hugging Face model identifier, if applicable" | a value on 185, `null` on 184, empty string on 107 |
| `name` | string | "Display name of the model" | 476 present, shaped "Vendor: Model" |
| `created` | integer | Spec: "Unix timestamp of when the model was created". Guide: "Unix timestamp of when the model was added to OpenRouter" | 476 present |
| `description` | string | "Description of the model" | 424 of 476 end in "..."; longest is 336 characters |
| `context_length` | integer or null | "Maximum context length in tokens" | 476 present, equals the largest endpoint value in all 55 models checked (M12) |
| `architecture` | object | "Model architecture information" | see M17 |
| `pricing` | object | Spec: "Pricing information for the model". Guide: "Pricing from the top provider for this model" | see M18 |
| `top_provider` | object | Spec: "Information about the top provider for this model". Guide: "Configuration details for the primary provider" | see M19 |
| `per_request_limits` | object or null | Spec: "Per-request token limits", with `prompt_tokens` and `completion_tokens`. Guide: "Rate limiting information (null if no limits)" | `null` on 476 |
| `supported_parameters` | array of `Parameter` | Spec: "List of supported parameters for this model". Guide: "Array of supported API parameters for this model" | see M11 |
| `default_parameters` | object or null | Spec: "Default parameters for this model". Guide: "Default parameter values for this model (null if none)" | see M14 |
| `supported_voices` | array or null | "List of supported voice identifiers for TTS models. Null for non-TTS models." | `null` on 476 |
| `knowledge_cutoff` | string or null | "The date up to which the model was trained on data. ISO 8601 date string (YYYY-MM-DD) or null if unknown." | a date on 144 |
| `expiration_date` | string or null | "The date after which the model may be removed. ISO 8601 date string (YYYY-MM-DD) or null if no expiration." | a date on 16, see M21 |
| `links` | object | `details`: "URL for the model details/endpoints API" | 476 present, see M22 |
| `endpoints` | array of string | "Request API paths that serve this model, derived from the same modality and adapter rules the routers use. A model served only through an alpha surface lists only that path." | see M16 |
| `reasoning` | object, optional | "Reasoning effort configuration. Omitted for non-reasoning models and dynamic router models." | present on 354, see M15 |
| `benchmarks` | object, optional | "Third-party benchmark rankings for this model. Omitted when no benchmark data is available." | present on 274 |
| `alias_target` | object, optional | "Concrete model targeted by this tilde-latest alias, when applicable" | present on 19, see M55 |

- Source: M-s4 (spec schema `Model`), M-s2 (guide table "Model Object Schema"), M-s1. Basis: OR-documented.

**M11.** `supported_parameters` draws from a 27-value enum named `Parameter`. The enum is open, and the catalog uses 26 of the values.

| Value | Meaning on the Parameters page (quoted) | Interactive entries listing it, of 365 |
|-|-|-|
| `temperature` | "This setting influences the variety in the model's responses." | 302 |
| `top_p` | "This setting limits the model's choices to a percentage of likely tokens" | 295 |
| `top_k` | "This limits the model's choice of tokens at each step" | 200 |
| `min_p` | "Represents the minimum probability for a token to be considered, relative to the probability of the most likely token." | 97 |
| `top_a` | "Consider only the top tokens with \"sufficiently high\" probabilities based on the probability of the most likely token." | 16 |
| `frequency_penalty` | "This setting aims to control the repetition of tokens based on how often they appear in the input." | 202 |
| `presence_penalty` | "Adjusts how often the model repeats specific tokens already used in the input." | 196 |
| `repetition_penalty` | "Helps to reduce the repetition of tokens from the input." | 138 |
| `max_tokens` | "This sets the upper limit for the number of tokens the model can generate in response." | 348 |
| `max_completion_tokens` | "This parameter shares `max_tokens` semantics" | 85 |
| `logit_bias` | "Accepts a JSON object that maps tokens (specified by their token ID in the tokenizer) to an associated bias value from -100 to 100." | 115 |
| `logprobs` | "Whether to return log probabilities of the output tokens or not." | 148 |
| `top_logprobs` | "An integer between 0 and 20 specifying the number of most likely tokens to return at each token position" | 148 |
| `prediction` | Defined by the enum only; the Parameters page has 25 sections and this value is outside them | 7 |
| `seed` | "If specified, the inferencing will sample deterministically" | 274 |
| `response_format` | "Forces the model to produce specific output format." | 311 |
| `structured_outputs` | "If the model can return structured outputs using response_format json_schema." | 300 |
| `strict_tool_schemas` | Defined by the enum only | 0 |
| `stop` | "Stop generation immediately if the model encounter any token specified in the stop array." | 243 |
| `tools` | "Tool calling parameter, following OpenAI's tool calling request shape. For non-OpenAI providers, it will be transformed accordingly." | 305 |
| `tool_choice` | "Controls which (if any) tool is called by the model." | 300 |
| `parallel_tool_calls` | "Whether to enable parallel function calling during tool use." | 14 |
| `include_reasoning` | "Deprecated alias for reasoning.exclude. When true, reasoning tokens are returned in the response when supported by the model." | 257 |
| `reasoning` | "Controls reasoning behavior for models that support thinking tokens, including whether reasoning is enabled, the reasoning effort, maximum reasoning tokens, and whether reasoning is excluded from the response." | 257 |
| `reasoning_effort` | "OpenAI-style reasoning effort setting." The page lists the enum as "(xhigh, high, medium, low, minimal, none)" | 152 |
| `web_search_options` | "Configures native web search options for models and providers that support web-connected answers." | 16 |
| `verbosity` | "Constrains the verbosity of the model's response." The page lists the enum as "(low, medium, high, xhigh, max)" | 59 |

- The enum has `"x-speakeasy-unknown-values": "allow"`, which tells generated clients to accept values outside the list.
- `strict_tool_schemas` is listed by 0 of 476 catalog entries, 0 of 490 fetched endpoint records and 0 of 936 ZDR endpoint records. The changelog entry for November 19, 2026 says "`Parameter`: enum property_added" and names no value. That this value is the one added three days before the snapshot is an inference (basis OR-inferred, from the changelog line and the zero counts).
- The Models guide has a shorter list of 13 values under "Supported Parameters" (`tools`, `tool_choice`, `max_tokens`, `temperature`, `top_p`, `reasoning`, `include_reasoning`, `structured_outputs`, `response_format`, `stop`, `frequency_penalty`, `presence_penalty`, `seed`) and describes the array as showing "which OpenAI-compatible parameters work with each model".
- Source: M-s4, M-s17, M-s2, M-s23, M-s1, M-s26, M-s27. Basis: OR-documented.

**M12.** The list is built per endpoint: sampling parameters come from the provider's feed and tested capabilities come from OpenRouter's own tests. The model-level list is the union of the endpoint lists.

- Quote (provider feed field `output_modalities[].supported_parameters`, status "Applied"): "Text output: Plain sampling parameters are applied; parameters OpenRouter verifies by test (`tools`, `tool_choice`, `response_format`, `structured_outputs`, `logprobs`, and the rest of the capability suite) are never enabled from the feed, only by the capability tests passing."
- Quote: "OpenRouter discovers testable capabilities by running capability tests against the endpoint, so declaring a feature there does not enable it."
- Quote (feed descriptors): "An absent key means the parameter is unsupported." The feed's descriptors have value domains, for example `"temperature": { "type": "range", "min": 0, "max": 1 }` and `"stop": { "type": "array", "max_items": 4 }`. The public catalog publishes the names only.
- Quote (Structured Outputs guide): "Support is determined per endpoint, not just per model: the same model may be served by multiple providers, and only some of those providers may support structured outputs. Endpoint support can also change over time."
- Computed 2026-11-22: for all 55 fetched models that have a catalog entry and at least one endpoint, the catalog `supported_parameters` set equals the union of the endpoints' `supported_parameters` sets, and the catalog `context_length` equals the largest endpoint `context_length`. So a parameter in the model-level list means "at least one endpoint lists it".
- Source: M-s16, M-s1, M-s26, and `https://openrouter.ai/docs/guides/features/structured-outputs` (M-s42). Basis: OR-documented for the quotes and the data; the union rule itself is OR-inferred from 55 of 55 matches.

**M13.** A request that sends a parameter an endpoint does not list is still routed by default.

- Quote: "can still receive the request, but will ignore unknown parameters. When you set `require_parameters` to `true`, the request won't even be routed to that provider."
- Quote: "a small set of parameters is used as a soft preference when choosing between providers of the same model: `tools`, the response format (`text.format` on the Responses API, `response_format` on Chat Completions, including structured outputs), and `verbosity`."
- Quote: "If none of a model's providers support the parameter, the request is still routed to that model and the parameter is ignored".
- Quote (Parameters page): "When a sampling parameter is absent from your request, OpenRouter omits it upstream rather than substituting a hardcoded value, so the provider applies its own default."
- Source: M-s18, M-s17. Basis: OR-documented. This matches local survey claim LC3 for the documented default. M35 holds the documented cases where a parameter is refused.

**M14.** `default_parameters` holds up to six nullable sampling defaults. In the snapshot it is an object on every entry.

- Spec: "Default parameters for this model", with `additionalProperties: false` and the keys `temperature`, `top_p`, `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, each number or null. The object itself may be null.
- Snapshot: `null` on 0 of 476. Among the 365 interactive entries, `{}` on 168, an object whose listed keys are all `null` on 113, and at least one number on 84. Most frequent values: `top_p` 0.95 (51 entries), `temperature` 1 (42), `temperature` 0.3 (15, all Mistral), `top_k` 20 (12).
- The spec and the guide do not say whether OpenRouter applies these values or only reports them. M13's quote says OpenRouter "omits it upstream rather than substituting a hardcoded value".
- Source: M-s4, M-s2, M-s1. Basis: OR-documented. Per-family table: M41.

**M15.** The `reasoning` object tells a client whether reasoning can be turned off, what the default is, and which effort words the model accepts.

| Key | Spec description (quoted) |
|-|-|
| `mandatory` (required) | "When true, reasoning cannot be disabled and effort \"none\" is rejected." |
| `default_enabled` | "Default reasoning enabled state when the client does not set `reasoning.enabled`." |
| `supported_efforts` | "Allowed reasoning effort values for this model, in descending effort order (highest first)." Followed by "Null means no allowlist" |
| `default_effort` | "Default reasoning effort when the client enables reasoning without specifying effort. Maps to `reasoning.effort` in chat requests. When `\"none\"`, prefer omitting effort unless the user explicitly disables reasoning." |
| `supports_max_tokens` | "Present and `true` when the model accepts `reasoning.max_tokens` in requests (Anthropic-style) instead of or in addition to `reasoning.effort`. Omitted otherwise." |

- `ReasoningEffort` enum: `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`, `null`.
- Guide quotes, for client UIs: "**`supported_efforts`**: Filter effort selectors to these values, returned in descending effort order (highest first). When `null`, all gateway effort values are accepted. When omitted, the model does not expose effort selection." and "Non-reasoning models and dynamic router models (`openrouter/auto`, `openrouter/free`) omit the `reasoning` field."
- Behavior quotes: "Concrete model slugs keep strict validation. Sending `effort: \"none\"` to a model that mandates reasoning returns a `400`." and "unsupported non-`none` efforts are mapped to the nearest supported level on all slugs (for example `xhigh` → `high` on a model without `xhigh`)". The 400 text, from the Claude 5.5 migration guide: "`Reasoning is mandatory for this endpoint and cannot be disabled.`"
- Snapshot: the object is on 354 of 476 entries (252 of 365 interactive). `supported_efforts` is a list on 146 interactive entries and the key is absent on 106; the value `null` appears on 0. `reasoning_effort` in `supported_parameters` coincides exactly with a `supported_efforts` list on every interactive entry that has the object; the six router entries list `reasoning_effort` and have no object.
- Source: M-s4, M-s19, M-s15, M-s21, M-s1. Basis: OR-documented. Per-family tables: M36, M37.

**M16.** `endpoints` lists the request routes that serve a model. In the snapshot it does not separate vendors.

- Spec: "Request API paths that serve this model, derived from the same modality and adapter rules the routers use. A model served only through an alpha surface lists only that path." The changelog adds the field on October 30, 2026: "`Model`: new property `endpoints` (always present)". The `endpoints` query filter follows on November 3, 2026.
- Snapshot: 369 entries list exactly `/api/v1/chat/completions`, `/api/v1/completions`, `/api/v1/messages`, `/api/v1/responses`; 92 `:batch` entries list `/api/v1/batches` alone; 10 add `/api/v1/images`; 3 add `/api/alpha/extractions`; 2 add `/api/v1/videos`. Every entry of every vendor family that is not a batch entry lists all four chat-shaped routes (M38).
- Source: M-s4, M-s23, M-s1. Basis: OR-documented.

**M17.** `architecture` holds the modalities, a tokenizer family and an instruct type.

- Keys: `modality` ("Primary modality of the model", for example `text+image+file->text`), `input_modalities` (enum `text`, `image`, `file`, `audio`, `video`), `output_modalities` (enum `text`, `image`, `embeddings`, `audio`, `video`, `rerank`, `decisions`, `speech`, `transcription`), `tokenizer` (schema `ModelGroup`, "Tokenizer type used by the model": `Router`, `Media`, `Other`, `GPT`, `Claude`, `Gemini`, `Gemma`, `Grok`, `Cohere`, `Nova`, `Qwen`, `Yi`, `DeepSeek`, `Mistral`, `Llama2`, `Llama3`, `Llama4`, `PaLM`, `RWKV`, `Qwen3`), `instruct_type` ("Instruction format type", 22 names such as `chatml`, `llama3`, `deepseek-r1`, `qwen3`, or null).
- Snapshot: `tokenizer` is `Other` on 145, `GPT` 106, `Gemini` 43, `Claude` 32, `Router` 28, `Qwen3` 26, `Qwen` 23, `Mistral` 23, `DeepSeek` 17, `Grok` 9. `instruct_type` is `null` on 448 of 476.
- Source: M-s4, M-s1. Basis: OR-documented.

**M18.** `pricing` values are strings in USD per unit. Thirteen keys appear in the snapshot, plus two sentinel values.

| Key | Spec description (quoted) | Interactive entries with the key, of 365 |
|-|-|-|
| `prompt` | "Price in USD per token for prompt (input) processing" | 365 |
| `completion` | "Price in USD per token for completion (output) generation" | 365 |
| `input_cache_read` | "Price in USD per cached input token (read)" | 245 |
| `input_cache_write` | "Price per cache-write token, in USD per token. For providers with multiple cache TTLs (e.g. Anthropic), this is the default (5-minute) cache-write rate." | 67 |
| `input_cache_write_1h` | "Price per 1-hour cache-write token, in USD per token. Only present for providers that price an extended (1-hour) cache TTL separately, such as Anthropic." | 16, all Anthropic |
| `internal_reasoning` | "Price in USD per internal reasoning token" | 22 (21 Google, 1 Perplexity) |
| `web_search` | "Price in USD per web search" | 113 |
| `image` | "Price in USD per input image" | 21, all Google |
| `image_output` | "Price in USD per output image" | 10 |
| `audio` | "Price in USD per audio input token" | 24 |
| `audio_output` | "Price in USD per audio output token" | 2 |
| `input_audio_cache` | "Price in USD per cached audio input token" | 21 |
| `overrides` | "Conditional overrides of the base pricing (e.g. long-context or time-based pricing)." | 51 |
| `request`, `image_token`, `discount` | In the schema; `discount` is "Fractional discount applied to this endpoint's pricing" | 0 in the text catalog; `discount` is on every endpoint record |

- Sentinels, quoted: "**`-1` (Dynamic Router models)**: Routers such as `openrouter/auto`, `openrouter/auto-code`, `openrouter/fusion`, and `typesafe/jev-router` set `pricing.prompt: \"-1\"` and `pricing.completion: \"-1\"`." and "**`0` (Free models and variants)**". Snapshot: `"-1"` on 8 entries, `"0"` on 14 interactive entries (11 `:free`, `openrouter/free`, and two Google Lyria previews).
- Overrides, quoted: "The top-level pricing keys always reflect the price that applies to a request under default conditions" and "Consumers should skip entries containing condition fields they do not recognize rather than apply their prices". Condition keys: `min_prompt_tokens`, `utc_start`, `utc_end`, `utc_days`. Snapshot, all 476 entries: 86 override entries use `min_prompt_tokens`, 9 use a UTC time window, 1 uses `utc_days` alone. Example, `anthropic/claude-haiku-5.5`: `"overrides": [{"min_prompt_tokens": 100000, "prompt": "0.0000005", "completion": "0.0000025", ...}]` against a base `"prompt": "0.0000001"`.
- Source: M-s4, M-s2, M-s3, M-s1. Basis: OR-documented. Per-family table: M40.

**M19.** `top_provider` holds one provider's context limit, output limit and moderation flag.

- Keys: `context_length` ("Context length from the top provider"), `max_completion_tokens` ("Maximum completion tokens from the top provider. Input and output tokens share the context window, so the effective maximum output for a request is further limited by the context remaining after input tokens."), `is_moderated` ("Whether the top provider moderates content").
- Guide: "`max_completion_tokens` is a ceiling for `max_tokens`, not a guaranteed output size."
- Snapshot: `top_provider.context_length` differs from the model-level `context_length` on 29 entries, for example `qwen/qwen3.8-27b` (1024000 at model level, 262144 at the top provider) and `meta-llama/llama-4-scout` (1310720 and 327680). `max_completion_tokens` is `null` on 6 router entries. `is_moderated` is true on 143 entries: 91 OpenAI, 32 Anthropic, 6 Meta, 5 Cohere, 5 Amazon, 4 others.
- The docs do not say how the top provider is chosen.
- Source: M-s4, M-s2, M-s1. Basis: OR-documented.

**M20.** Four fields are constant or thin in the text catalog.

- `per_request_limits` is `null` on 476 of 476. `supported_voices` is `null` on 476 of 476 (it is for TTS models). `knowledge_cutoff` is a date on 144, of which 56 are OpenAI. `benchmarks` is on 274 and holds `design_arena` rows and an `artificial_analysis` object whose `coding_index` and `agentic_index` the spec calls "Always null".
- Source: M-s4, M-s1. Basis: OR-documented.

**M21.** `expiration_date` is the retirement signal, and it comes from the provider's feed.

- Spec: "The date after which the model may be removed." Guide: "Deprecation date for the model endpoint (null if not deprecated)".
- Provider feed, quoted: "If a model is scheduled for deprecation, include the `deprecation_date` field in ISO 8601 format." and "Models past their deprecation time may be automatically hidden from the marketplace."
- Snapshot: 16 entries have a date: `openai/gpt-5`, `gpt-5-mini`, `gpt-5-nano`, `gpt-5-pro`, `o3`, `o3-pro`, `gpt-5-image` (all 2026-12-11) and five of their `:batch` twins, `openai/gpt-5-image-mini` (2026-12-01), `z-ai/glm-4.5` and `z-ai/glm-4.5v` (2026-12-31), `mistralai/mistral-medium-3-5` (2027-01-31). The entry has a date and names no replacement model.
- Source: M-s4, M-s2, M-s16, M-s1. Basis: OR-documented.

**M22.** `canonical_slug` is the permanent name and `links.details` points at the endpoints route under that name.

- Quote: "`id`: The routable slug used in API requests (e.g. `anthropic/claude-3.5-sonnet`)." and "`canonical_slug`: The permanent canonical reference for the underlying model checkpoint (e.g. `anthropic/claude-3-5-sonnet-20241022`)."
- Snapshot: `links.details` equals `/api/v1/models/<canonical_slug>/endpoints` on 476 of 476. The relation between `id` and `canonical_slug` is in M54.
- Source: M-s3, M-s1. Basis: OR-documented.

### C. Every field of an endpoint entry

**M23.** `GET /api/v1/models/{author}/{slug}/endpoints` returns a short model header and one record per provider endpoint. The spec's `PublicEndpoint` schema has 26 properties and marks 21 as required.

The response's `data` object has `id`, `name`, `created`, `description`, `architecture` and `endpoints`. The model-level `supported_parameters`, `reasoning`, `default_parameters`, `pricing` and `top_provider` are on the models list only.

| Endpoint field | Type in spec | Meaning the spec gives (quoted where it has a description) | Observed in 490 fetched records |
|-|-|-|-|
| `name` | string | Typed only | The provider name and the canonical slug separated by a vertical bar character, for example provider `OpenAI` and slug `openai/gpt-5.2-20251211` |
| `model_id` | string | "The unique identifier for the model (permaslug)" | Holds the routable `id`, for example `openai/gpt-5.2` |
| `model_name` | string | Typed only | The model's display name |
| `provider_name` | `ProviderName` enum | Typed only | 65 distinct providers |
| `tag` | string | Typed only | Provider slug with region, precision or tier suffixes: `anthropic`, `google-vertex/global`, `amazon-bedrock/us-east-1`, `openai/fast`, `xai/zdr/priority`, `mistral/eu`, `deepinfra/fp8/flex` |
| `service_tier` | `flex`, `priority`, `ultrafast` or null | See M29 | null 412, `priority` 53, `flex` 22, `ultrafast` 3 |
| `quantization` | `Quantization` enum or null | See M27 | 8 distinct values |
| `context_length` | integer | Typed only | Per endpoint; differs between providers of one model (M31) |
| `max_completion_tokens` | integer or null | "Maximum completion tokens for this endpoint. Input and output tokens share the context window, so the effective maximum output for a request is further limited by the context remaining after input tokens." | Per endpoint |
| `max_prompt_tokens` | integer or null | Typed only | A number on 33 records, null on 457 |
| `pricing` | object | Same keys as the model `pricing`, plus `discount` | `discount` on 490 of 490 |
| `supported_parameters` | array of `Parameter` | Typed only | 25 distinct values (all but `prediction` and `strict_tool_schemas`) |
| `supports_tool_choice` | object | See M24 | 9 distinct flag combinations |
| `supports_implicit_caching` | boolean | Typed only | true 378, false 112 (M30) |
| `native_tools` | object | See M25 | 7 distinct tool mappings |
| `status` | integer enum | Typed only (M26) | `0` on 475, `-2` on 9, `-5` on 6 |
| `uptime_last_5m`, `uptime_last_30m`, `uptime_last_1d` | number or null | See M28 | null on 96, 52 and 10 records |
| `latency_last_30m`, `throughput_last_30m` | object or null | See M28 | null on 490 of 490 |
| `perf_last_30m_by_workload` | object | "Only visible when authenticated with an API key or cookie." | Key absent on 490 of 490 |
| `supports_diarization`, `supports_voice_cloning`, `supports_multiple_audio_references`, `supports_image_reference` | boolean | Speech-to-text and text-to-speech flags | false on every chat endpoint fetched |

- Quote (Models API page): "Use endpoint metadata when you want to pin requests to specific providers or enforce quantization standards using the `provider` request parameter in Chat Completions."
- Quote (provider selection, about `tag`-style slugs): "To route to **all** endpoints of a provider (across all regions and variants), just use the base slug without a suffix. For example, `\"google-vertex\"` will route across all Vertex AI regions."
- Source: M-s5, M-s3, M-s18, M-s26. Basis: OR-documented.

**M24.** `supports_tool_choice` says which `tool_choice` values an endpoint passed in testing. It is the only public place where forced tool choice is visible.

- Spec, schema `ToolChoiceSupport`: "Per-variant `tool_choice` support. `tool_choice` in `supported_parameters` only says the parameter is accepted; these flags say which of its values passed testing." Keys: `none`, `auto`, `required`, `function`. Example description: "Whether the endpoint supports naming a function, `\"tool_choice\": {\"type\": \"function\", ...}`."
- Changelog, August 28, 2026: "`PublicEndpoint`: required `supports_tool_choice` property added".
- Observed combinations in 490 records, written as four letters for none, auto, required, function with a dot for false: `NARF` 299, `NA..` 95, `.A..` 34, `.ARF` 24, `NA.F` 22, `NAR.` 10, `....` 3, `N.RF` 2, `.A.F` 1.
- Claude by model, from the fetched listings plus the ZDR list: `NA..` on every endpoint of `anthropic/claude-fable-5.2`, `claude-opus-5.5`, `claude-sonnet-5.5` and `claude-haiku-5.5`; `NARF` on every endpoint of `claude-opus-5`, `claude-sonnet-5`, `claude-opus-4.8`, `claude-opus-4.7`, `claude-sonnet-4.6`, `claude-opus-4.6`, `claude-sonnet-4.5`, `claude-opus-4.5`, `claude-haiku-4.5` and `claude-opus-4.1`.
- OpenRouter's Claude 5.5 migration guide, quoted: "Opus 5.5 rejects `tool_choice` set to `{\"type\": \"any\"}` or a named tool, and the Chat Completions forms `tool_choice: \"required\"` and `{\"type\": \"function\", \"function\": {\"name\": ...}}`, with the provider's 400 (`tool_choice: type \"tool\" and \"any\" are not supported for this model.`)."
- Source: M-s5, M-s23, M-s26, M-s27, M-s21. Basis: OR-documented.

**M25.** `native_tools` names the provider's own built-in tool that an OpenRouter server tool is translated to on that endpoint.

- Spec: "The server tools this endpoint accepts as the provider's own built-in tool (`engine: \"native\"`) instead of an OpenRouter engine, keyed by canonical `openrouter:*` name. Each value names the provider tool type the request is translated to. Where that tool runs (provider-side, or returned to the client as with Anthropic bash) is documented per tool. Empty when the provider has none."
- Changelog, September 28, 2026: "`PublicEndpoint`: required `native_tools` property added".
- Observed mappings in 490 records: `openrouter:web_search` to `web_search` (28: OpenAI 16, xAI 8, Azure 3, Meta 1), to `google_search` (21: Google AI Studio 12, Google Vertex 9), to `web_search_20260209` (13: Anthropic 8, Claude Platform on AWS 5); `openrouter:bash` to `bash_20250124` (30); `openrouter:apply_patch` to `apply_patch` (20: OpenAI 15, Azure 5); `openrouter:advisor` to `advisor_20260301` (13); `openrouter:web_fetch` to `web_fetch_20260209` (9).
- The same Claude model differs by provider: on `anthropic/claude-opus-5.5` the `anthropic` and `claude-on-aws` endpoints list web search, web fetch, bash and advisor, and the `amazon-bedrock`, `azure/global` and `google-vertex/global` endpoints list bash alone.
- Source: M-s5, M-s23, M-s26. Basis: OR-documented.

**M26.** `status` is an integer enum that the spec and the official SDK publish without meanings.

- Spec, schema `EndpointStatus`: `"enum": [0, -1, -2, -3, -5, -10]`, `"type": "integer"`, `"x-speakeasy-unknown-values": "allow"`. The schema has an example and no description.
- Official TypeScript SDK, `src/models/endpointstatus.ts` lines 9 to 16: `export const EndpointStatus = { Zero: 0, Minus1: -1, Minus2: -2, Minus3: -3, Minus5: -5, Minus10: -10, } as const;`
- Observed: `0` on 475 of 490 fetched records, `-2` on 9, `-5` on 6. In the ZDR list: `0` on 907, `-2` on 19, `-5` on 10.
- Source: M-s24, M-s29, M-s26, M-s27. Basis: OR-documented. The meaning of each number is an open question.

**M27.** `quantization` is per endpoint and reads `unknown` when a provider declares nothing.

- Spec enum: `int4`, `int8`, `fp4`, `mxfp4`, `nvfp4`, `fp6`, `fp8`, `mxfp8`, `fp16`, `bf16`, `fp32`, `unknown`.
- Provider feed, quoted: "Use `null` (or omit the field) when the precision is undeclared."
- Observed in 490 records: `unknown` 208, `fp8` 139, `fp4` 90, `mxfp4` 22, `nvfp4` 12, `bf16` 10, `int4` 6, `fp16` 3. Every fetched endpoint of a Claude, hosted GPT, Gemini, Grok or Mistral model reads `unknown` (95 records).
- Source: M-s5, M-s16, M-s26. Basis: OR-documented.

**M28.** Latency and throughput are hidden from unauthenticated callers. Uptime is public.

- Quote (`PercentileStats`): "Latency percentiles in milliseconds over the last 30 minutes. Latency measures time to first token. Only visible when authenticated with an API key or cookie; returns null for unauthenticated requests."
- Quote (`throughput_last_30m`): "Throughput percentiles in tokens per second over the last 30 minutes. Throughput measures output token generation speed; pN is the speed that N% of requests reached or exceeded, so p90 is lower than p50."
- Quote (`uptime_last_1d`): "Uptime percentage over the last day: the share of minutes in which at least 80% of provider attempts succeeded, counting only minutes with 10 or more attempts. Rate-limited and caller-caused failures are excluded. Null when no minute had enough traffic."
- Observed: `latency_last_30m` and `throughput_last_30m` are null on 490 of 490 records.
- Source: M-s5, M-s26. Basis: OR-documented.

**M29.** A service tier is its own endpoint record with its own price.

- Quote (`PublicServiceTier`): "The service tier this endpoint sells, or null for a standard endpoint. The same value whether the provider selects the tier with a request parameter or with a dedicated model id. Always the canonical tier: priority covers the fast request alias and the turbo, highspeed, and prime provider model-id spellings, none of which this field reports."
- Changelog, October 15, 2026: "`PublicEndpoint`: required `service_tier` property added".
- Observed: `openai/gpt-6.2-sol` has OpenAI endpoints tagged `openai` (2 and 10 dollars per million input and output tokens), `openai/fast` with `service_tier: "priority"` (4 and 20) and `openai/ultrafast` with `"ultrafast"` (6 and 30). `anthropic/claude-fable-5.2` has `anthropic` (8 and 40) and `anthropic/fast` with `"priority"` (16 and 80).
- Source: M-s5, M-s23, M-s26. Basis: OR-documented.

**M30.** `supports_implicit_caching` is a bare boolean in the spec. Its values split by provider and, for OpenAI, by model generation.

- Spec: `{"type": "boolean"}`, with no description.
- Observed in the 59 fetched listings, by model family: all 29 endpoints of Claude models are `false` (Anthropic, Amazon Bedrock, Azure, Google Vertex, Claude Platform on AWS); all 24 endpoints of Gemini models are `false` (Google AI Studio and Vertex); `openai/gpt-4.1`, `gpt-5.2` and `gpt-5.4` endpoints are `true` on OpenAI and Azure; `openai/gpt-6.1-terra`, `gpt-6.2-sol` and `gpt-6.2-astra` endpoints are `false` on OpenAI, Azure and Amazon Bedrock; the xAI endpoints of `x-ai/grok-4.7` and `grok-4.3` are `true` and the Amazon Bedrock endpoint of `grok-4.7` is `false`; endpoints whose provider is Mistral are `true` on 9 and `false` on 2.
- In the ZDR list (936 records): `true` 572, `false` 364.
- Source: M-s24, M-s26, M-s27. Basis: OR-documented. What the flag implies for a request is not stated on the pages read for this topic.

**M31.** Each endpoint has its own context, output and prompt limits.

- `max_prompt_tokens` is a number on 33 of 490 records: OpenAI 15, Alibaba 8, Azure 5, KT 2, Amazon Bedrock 2, Cohere 1. For `openai/gpt-5.2` every endpoint reports `context_length` 400000, `max_prompt_tokens` 272000 and `max_completion_tokens` 128000. For `openai/gpt-5.4` the three numbers are 1050000, 922000 and 128000.
- One model, different limits: `anthropic/claude-sonnet-4.5` has `context_length` 200000 on `azure/global` and 1000000 on the other four endpoints. `google/gemini-4-carbon` has `max_completion_tokens` 32768 on `google-ai-studio` and 943718 on its other four endpoints. `openai/gpt-4.1` has `max_completion_tokens` 942818 on `azure` and 32768 on `openai`. `meta-llama/llama-3.3-70b-instruct` has `context_length` 24000 on `cloudflare/fp8`, 128000 on `coreweave/fp16` and 131072 on its other five endpoints.
- Source: M-s26. Basis: OR-documented.

### D. The catalog by vendor family (computed from the 2026-11-22 snapshot)

All of section D is data computed from the snapshot `models.json` with the scripts `analyze.py` and `extra*.py` in the topic's scratch subfolder. Source: M-s1. Basis: OR-documented. A family is the `id` prefix before the slash, with a leading `~` removed.

**M32.** The 476 entries fall into 18 named families plus 42 smaller prefixes. A quarter of the list is batch twins and aliases.

| Family (id prefix) | All entries | Base | `:free` | `:batch` | `~` alias | Router | Interactive entries (base + free + router) |
|-|-|-|-|-|-|-|-|
| `anthropic` | 36 | 16 | 0 | 16 | 4 | 0 | 16 |
| `openai` | 111 | 62 | 0 | 44 | 5 | 0 | 62 |
| `google` | 51 | 35 | 2 | 12 | 2 | 0 | 37 |
| `x-ai` | 10 | 6 | 0 | 3 | 1 | 0 | 6 |
| `deepseek` | 20 | 14 | 0 | 3 | 3 | 0 | 14 |
| `mistralai` | 22 | 21 | 0 | 1 | 0 | 0 | 21 |
| `moonshotai` | 11 | 8 | 0 | 2 | 1 | 0 | 8 |
| `qwen` | 43 | 40 | 0 | 3 | 0 | 0 | 40 |
| `z-ai` | 22 | 17 | 0 | 3 | 2 | 0 | 17 |
| `meta-llama` | 7 | 7 | 0 | 0 | 0 | 0 | 7 |
| `meta` | 10 | 9 | 0 | 1 | 0 | 0 | 9 |
| `minimax` | 5 | 4 | 0 | 1 | 0 | 0 | 4 |
| `amazon` | 5 | 5 | 0 | 0 | 0 | 0 | 5 |
| `cohere` | 5 | 5 | 0 | 0 | 0 | 0 | 5 |
| `perplexity` | 5 | 5 | 0 | 0 | 0 | 0 | 5 |
| `nvidia` | 11 | 7 | 4 | 0 | 0 | 0 | 11 |
| `tencent` | 7 | 7 | 0 | 0 | 0 | 0 | 7 |
| `openrouter` | 7 | 0 | 0 | 0 | 0 | 7 | 7 |
| `other` | 88 | 79 | 5 | 3 | 1 | 0 | 84 |
| **Total** | 476 | 347 | 11 | 92 | 19 | 7 | 365 |

- `other` prefixes: aion-labs (6), inclusionai (5), thinkingmachines (5), xiaomi (5), morph (4), sakana (4), unbiased (4), bytedance-seed (3), liquid (3), nex-agi (3), thedrummer (3), and 31 prefixes with one or two entries.
- `meta` (9 base entries, the Muse line) and `meta-llama` (7, Llama and Llama Guard) are separate prefixes.

**M33.** `supported_parameters` by family, interactive entries only. Each cell is the number of the family's entries that list the parameter. The header shows the family size.

| Parameter | `anthropic` (16) | `openai` (62) | `google` (37) | `x-ai` (6) | `deepseek` (14) | `mistralai` (21) | `moonshotai` (8) | `qwen` (40) | `z-ai` (17) | All (365) |
|-|-|-|-|-|-|-|-|-|-|-|
| `temperature` | 6 | 19 | 37 | 6 | 14 | 21 | 8 | 40 | 17 | 302 |
| `top_p` | 5 | 19 | 37 | 6 | 14 | 21 | 8 | 40 | 17 | 295 |
| `top_k` | 6 | 3 | 5 | 2 | 14 | 5 | 8 | 39 | 17 | 200 |
| `min_p` | 0 | 2 | 5 | 0 | 11 | 4 | 3 | 17 | 6 | 97 |
| `top_a` | 0 | 1 | 0 | 0 | 3 | 0 | 0 | 2 | 1 | 16 |
| `frequency_penalty` | 0 | 13 | 6 | 0 | 14 | 20 | 8 | 39 | 12 | 202 |
| `presence_penalty` | 0 | 13 | 6 | 0 | 11 | 20 | 8 | 39 | 12 | 196 |
| `repetition_penalty` | 0 | 2 | 6 | 0 | 11 | 4 | 8 | 23 | 11 | 138 |
| `max_tokens` | 16 | 52 | 37 | 6 | 14 | 21 | 8 | 40 | 17 | 348 |
| `max_completion_tokens` | 15 | 46 | 2 | 0 | 3 | 0 | 0 | 1 | 0 | 85 |
| `logit_bias` | 0 | 13 | 5 | 0 | 11 | 4 | 4 | 22 | 6 | 115 |
| `logprobs` | 0 | 16 | 4 | 6 | 9 | 4 | 6 | 33 | 7 | 148 |
| `top_logprobs` | 0 | 16 | 4 | 6 | 9 | 4 | 6 | 33 | 7 | 148 |
| `prediction` | 0 | 5 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 7 |
| `seed` | 0 | 62 | 37 | 6 | 11 | 20 | 8 | 38 | 12 | 274 |
| `response_format` | 15 | 62 | 35 | 6 | 14 | 19 | 7 | 39 | 16 | 311 |
| `structured_outputs` | 15 | 61 | 30 | 6 | 14 | 20 | 7 | 36 | 10 | 300 |
| `strict_tool_schemas` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `stop` | 16 | 14 | 24 | 2 | 11 | 21 | 8 | 39 | 12 | 243 |
| `tools` | 16 | 59 | 26 | 5 | 14 | 19 | 8 | 38 | 17 | 305 |
| `tool_choice` | 16 | 59 | 26 | 5 | 14 | 19 | 8 | 38 | 17 | 300 |
| `parallel_tool_calls` | 0 | 0 | 0 | 0 | 3 | 0 | 2 | 0 | 2 | 14 |
| `include_reasoning` | 16 | 46 | 30 | 6 | 12 | 3 | 6 | 26 | 17 | 257 |
| `reasoning` | 16 | 46 | 30 | 6 | 12 | 3 | 6 | 26 | 17 | 257 |
| `reasoning_effort` | 12 | 41 | 20 | 5 | 7 | 3 | 2 | 5 | 6 | 152 |
| `web_search_options` | 0 | 6 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 16 |
| `verbosity` | 13 | 42 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 59 |
| (empty list) | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 3 |

| Parameter | `meta-llama` (7) | `meta` (9) | `minimax` (4) | `amazon` (5) | `cohere` (5) | `perplexity` (5) | `nvidia` (11) | `tencent` (7) | `openrouter` (7) | `other` (84) | All (365) |
|-|-|-|-|-|-|-|-|-|-|-|-|
| `temperature` | 7 | 9 | 4 | 5 | 5 | 5 | 10 | 7 | 5 | 77 | 302 |
| `top_p` | 7 | 9 | 4 | 5 | 5 | 5 | 10 | 3 | 5 | 75 | 295 |
| `top_k` | 7 | 9 | 3 | 5 | 5 | 5 | 6 | 3 | 5 | 53 | 200 |
| `min_p` | 4 | 2 | 2 | 0 | 0 | 0 | 5 | 2 | 5 | 29 | 97 |
| `top_a` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 4 | 5 | 16 |
| `frequency_penalty` | 6 | 2 | 3 | 0 | 5 | 5 | 6 | 3 | 5 | 55 | 202 |
| `presence_penalty` | 6 | 2 | 3 | 0 | 5 | 5 | 6 | 3 | 5 | 52 | 196 |
| `repetition_penalty` | 6 | 9 | 2 | 0 | 0 | 0 | 6 | 3 | 5 | 42 | 138 |
| `max_tokens` | 7 | 9 | 4 | 5 | 5 | 5 | 10 | 7 | 5 | 80 | 348 |
| `max_completion_tokens` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 7 | 4 | 7 | 85 |
| `logit_bias` | 6 | 2 | 2 | 0 | 0 | 0 | 6 | 2 | 4 | 28 | 115 |
| `logprobs` | 4 | 2 | 3 | 0 | 0 | 0 | 4 | 1 | 5 | 44 | 148 |
| `top_logprobs` | 4 | 2 | 3 | 0 | 0 | 0 | 4 | 1 | 5 | 44 | 148 |
| `prediction` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 7 |
| `seed` | 6 | 5 | 3 | 0 | 5 | 0 | 10 | 3 | 5 | 43 | 274 |
| `response_format` | 4 | 9 | 3 | 0 | 5 | 0 | 6 | 6 | 5 | 60 | 311 |
| `structured_outputs` | 5 | 9 | 3 | 0 | 5 | 1 | 6 | 6 | 5 | 61 | 300 |
| `strict_tool_schemas` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `stop` | 7 | 2 | 4 | 5 | 5 | 0 | 6 | 7 | 5 | 55 | 243 |
| `tools` | 5 | 9 | 4 | 5 | 5 | 0 | 9 | 4 | 5 | 57 | 305 |
| `tool_choice` | 5 | 9 | 4 | 1 | 5 | 0 | 9 | 4 | 5 | 56 | 300 |
| `parallel_tool_calls` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 3 | 4 | 14 |
| `include_reasoning` | 0 | 9 | 4 | 1 | 1 | 3 | 10 | 4 | 5 | 58 | 257 |
| `reasoning` | 0 | 9 | 4 | 1 | 1 | 3 | 10 | 4 | 5 | 58 | 257 |
| `reasoning_effort` | 0 | 9 | 1 | 0 | 0 | 0 | 3 | 3 | 5 | 30 | 152 |
| `web_search_options` | 0 | 0 | 0 | 0 | 0 | 5 | 0 | 0 | 1 | 4 | 16 |
| `verbosity` | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 3 | 1 | 59 |
| (empty list) | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 2 | 0 | 3 |

Two prefixes mix closed-weight and open-weight models, so they are split here:

| Parameter | openai hosted (59) | openai gpt-oss (3) | google gemini (27) | google gemma (8) | google other (2) |
|-|-|-|-|-|-|
| `temperature` | 16 | 3 | 27 | 8 | 2 |
| `top_p` | 16 | 3 | 27 | 8 | 2 |
| `top_k` | 0 | 3 | 0 | 5 | 0 |
| `min_p` | 0 | 2 | 0 | 5 | 0 |
| `top_a` | 0 | 1 | 0 | 0 | 0 |
| `frequency_penalty` | 11 | 2 | 0 | 6 | 0 |
| `presence_penalty` | 11 | 2 | 0 | 6 | 0 |
| `repetition_penalty` | 0 | 2 | 0 | 6 | 0 |
| `max_tokens` | 49 | 3 | 27 | 8 | 2 |
| `max_completion_tokens` | 46 | 0 | 0 | 2 | 0 |
| `logit_bias` | 11 | 2 | 0 | 5 | 0 |
| `logprobs` | 14 | 2 | 0 | 4 | 0 |
| `top_logprobs` | 14 | 2 | 0 | 4 | 0 |
| `prediction` | 5 | 0 | 0 | 0 | 0 |
| `seed` | 59 | 3 | 27 | 8 | 2 |
| `response_format` | 59 | 3 | 25 | 8 | 2 |
| `structured_outputs` | 58 | 3 | 24 | 6 | 0 |
| `strict_tool_schemas` | 0 | 0 | 0 | 0 | 0 |
| `stop` | 11 | 3 | 18 | 6 | 0 |
| `tools` | 56 | 3 | 20 | 6 | 0 |
| `tool_choice` | 56 | 3 | 20 | 6 | 0 |
| `parallel_tool_calls` | 0 | 0 | 0 | 0 | 0 |
| `include_reasoning` | 43 | 3 | 26 | 4 | 0 |
| `reasoning` | 43 | 3 | 26 | 4 | 0 |
| `reasoning_effort` | 39 | 2 | 20 | 0 | 0 |
| `web_search_options` | 6 | 0 | 0 | 0 | 0 |
| `verbosity` | 42 | 0 | 0 | 0 | 0 |

The same tables over all 476 entries and over the 92 batch entries are in the scratch files `B3_params_all_major.md`, `B4_params_all_rest.md` and `B5_params_batch.md`.

**M34.** Patterns in those tables, each checked by script.

- `reasoning` and `include_reasoning` always appear together: 257 of 257 interactive entries, and 359 of 359 over all 476. No entry lists one and not the other, in any family.
- `reasoning_effort` is narrower. Entries that list it, out of the entries that list `reasoning`: Anthropic 12 of 16, OpenAI 41 of 46, Google 20 of 30, xAI 5 of 6, DeepSeek 7 of 12, Mistral 3 of 3, Moonshot 2 of 6, Qwen 5 of 26, Z.ai 6 of 17, Meta 9 of 9, MiniMax 1 of 4, NVIDIA 3 of 10, Tencent 3 of 4. It is listed exactly where the `reasoning` object has a `supported_efforts` list (M15).
- `tool_choice` follows `tools` except on 5 entries that list `tools` alone: `amazon/nova-premier-v1`, `nova-lite-v1`, `nova-micro-v1`, `nova-pro-v1`, and `recursive/flint-v1.0:free`. 57 interactive entries with a non-empty list omit `tools`, among them all 5 Perplexity entries, the image-output Gemini and GPT entries, and `x-ai/grok-4.20-multi-agent`.
- `structured_outputs` and `response_format` diverge on 43 entries: 16 list `structured_outputs` alone (4 Sakana, 2 Nex AGI, 2 Perceptron, 2 Reka and 6 others) and 27 list `response_format` alone (6 Z.ai including `z-ai/glm-5.3-prime`, 5 Google, 3 Qwen, `openai/gpt-4`, and 12 others).
- `max_completion_tokens` is listed by 85 entries: OpenAI 46, Anthropic 15, Tencent 7, `openrouter/` routers 4, DeepSeek 3, Google 2 (Gemma 4), Qwen 1, others 7. Ten OpenAI entries list it and omit `max_tokens`: `gpt-5.2-codex`, `gpt-5.2-chat`, `gpt-5.1-codex-max`, `gpt-5.1-codex`, `gpt-5.1-codex-mini`, `gpt-4.1-nano`, `gpt-4o-2024-05-13`, `gpt-3.5-turbo-0613`, `gpt-3.5-turbo-16k`, `gpt-4`.
- `parallel_tool_calls` is listed by 14 entries: `deepseek/deepseek-v4.1-flash-1017`, `deepseek-v4.1-flash`, `deepseek-v4-flash-0731`, `moonshotai/kimi-k2.7-code`, `kimi-k2.6`, `z-ai/glm-5.3-flash`, `glm-5.3`, `upstage/solar-mini4`, `solar-pro4`, `xiaomi/mimo-v2.6-flash`, and four routers. Anthropic, OpenAI and Google entries that list it: 0.
- `web_search_options` is listed by 16 entries: six `openai/gpt-4o` entries, all 5 Perplexity entries, 4 Sakana entries and `openrouter/auto`. 104 interactive entries have a `pricing.web_search` key and omit `web_search_options`: OpenAI 48, Google 27, Anthropic 16, Meta 7, xAI 6.
- `prediction` is listed by 7 entries: five `openai/gpt-4o` entries, `mistralai/codestral-2508` and `openrouter/auto`.
- `verbosity` is listed by 59 entries: OpenAI 42, Anthropic 13, and the routers `openrouter/auto-code`, `openrouter/auto-beta`, `openrouter/pareto-code`, `typesafe/jev-router`.
- `seed` is listed by 274. It is omitted by all 16 Anthropic entries, all 5 Amazon, all 5 Perplexity, 5 Z.ai, 4 Tencent, 4 Meta and 3 DeepSeek entries.
- `stop` is listed by 243. It is omitted by 48 OpenAI entries (every GPT-5 and GPT-6 entry, the three GPT-4.1 entries, `o3`, `o3-pro`), 13 Google, 7 Meta, 5 Z.ai, 5 Perplexity, 4 xAI, 4 NVIDIA and 3 DeepSeek entries.
- `logprobs` and `top_logprobs` are always paired (148 and 148). Anthropic entries listing them: 0. Google: 4, all Gemma. xAI: 6 of 6. Qwen: 33 of 40.
- `top_k` on closed-weight flagships: the 6 Anthropic entries created before 2026-04 list it; hosted OpenAI and Gemini entries listing it: 0; xAI 2 of 6 (`grok-4.7`, `grok-4.6`).
- Gemini (27 entries) lists a narrow set: `temperature`, `top_p`, `max_tokens`, `seed`, `response_format`, `structured_outputs`, `stop`, `tools`, `tool_choice` and the three reasoning names. Its counts for `top_k`, `min_p`, the three penalties, `logit_bias`, `logprobs`, `verbosity` and `max_completion_tokens` are all 0.
- Three entries have an empty list: `nvidia/switchyard`, `openrouter/fusion`, `openrouter/bodybuilder`.

**M35.** Sampling parameters: the newer Claude and GPT entries stop listing them, and OpenRouter's migration guides describe two different outcomes when a caller sends them anyway.

- Anthropic, by creation date: the 10 entries from `anthropic/claude-opus-4.7` (2026-04-16) through `anthropic/claude-fable-5.2` (2026-11-09) list none of `temperature`, `top_p`, `top_k`. The 6 older entries list `temperature` and `top_k`; five of them list `top_p` (`anthropic/claude-opus-4.5` is the one that omits it).
- OpenAI: 43 of 62 entries omit `temperature` and `top_p`: every entry from `openai/gpt-5` (2025-08-07) on except `gpt-5-image`, `gpt-5-image-mini`, `gpt-audio`, `gpt-audio-mini` and `gpt-oss-safeguard-20b`, plus `o3` and `o3-pro`. The 19 that list them are the GPT-4 family (10 entries), GPT-3.5 (2), the two audio and two image entries, and the three open-weight `gpt-oss` entries.
- Others that omit `temperature`: 4 Sakana entries, 2 ScaleDown entries and `relace/relace-apply-3`. Every entry of Google, xAI, DeepSeek, Mistral, Moonshot, Qwen, Z.ai, Meta, MiniMax, Amazon, Cohere and Perplexity lists `temperature`.
- Quote (Claude 4.7 migration guide): "Claude 4.7 Opus no longer accepts `temperature`, `top_p`, or `top_k`. If you pass these parameters, they will be silently ignored", and the same sentence continues "your request will still succeed, but the parameters will have no effect."
- Quote (Claude 5.5 migration guide): "Haiku 5.5 rejects any `temperature` other than `1`, any `top_p` other than its default, any `top_k`, and requests that set both `temperature` and `top_p`. Remove all three from your requests and steer the model with prompting instead." The guide's summary row reads "`temperature`, `top_p`, `top_k` | Accepted | Non-default values not supported".
- The catalog signal is identical for both models (the names are missing from `supported_parameters`), while the documented outcome is "silently ignored" on one and "rejects" on the other.
- Source: M-s1, M-s22, M-s21. Basis: OR-documented.

**M36.** The `reasoning` object by family, interactive entries.

| Family | Entries | Has `reasoning` object | `mandatory: true` | `mandatory: false` | `default_enabled: true` | `default_enabled: false` | `default_enabled` absent | Has `supported_efforts` list | `supports_max_tokens: true` | Lists `reasoning` in `supported_parameters` | Object present but `reasoning` param not listed | `reasoning` param listed but object absent |
|-|-|-|-|-|-|-|-|-|-|-|-|-|
| `anthropic` | 16 | 16 | 5 | 11 | 2 | 3 | 11 | 12 | 1 | 16 | 0 | 0 |
| `openai` | 62 | 46 | 27 | 19 | 21 | 5 | 20 | 41 | 0 | 46 | 0 | 0 |
| `google` | 37 | 30 | 17 | 13 | 11 | 4 | 15 | 20 | 0 | 30 | 0 | 0 |
| `x-ai` | 6 | 6 | 4 | 2 | 5 | 1 | 0 | 5 | 0 | 6 | 0 | 0 |
| `deepseek` | 14 | 12 | 1 | 11 | 4 | 1 | 7 | 7 | 0 | 12 | 0 | 0 |
| `mistralai` | 21 | 3 | 0 | 3 | 1 | 1 | 1 | 3 | 0 | 3 | 0 | 0 |
| `moonshotai` | 8 | 6 | 2 | 4 | 6 | 0 | 0 | 2 | 0 | 6 | 0 | 0 |
| `qwen` | 40 | 26 | 4 | 22 | 13 | 0 | 13 | 5 | 3 | 26 | 0 | 0 |
| `z-ai` | 17 | 17 | 5 | 12 | 12 | 0 | 5 | 6 | 0 | 17 | 0 | 0 |
| `meta-llama` | 7 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `meta` | 9 | 9 | 9 | 0 | 0 | 0 | 9 | 9 | 0 | 9 | 0 | 0 |
| `minimax` | 4 | 4 | 3 | 1 | 0 | 0 | 4 | 1 | 0 | 4 | 0 | 0 |
| `amazon` | 5 | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 |
| `cohere` | 5 | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 |
| `perplexity` | 5 | 3 | 3 | 0 | 0 | 0 | 3 | 0 | 0 | 3 | 0 | 0 |
| `nvidia` | 11 | 10 | 0 | 10 | 5 | 0 | 5 | 3 | 4 | 10 | 0 | 0 |
| `tencent` | 7 | 4 | 0 | 4 | 4 | 0 | 0 | 3 | 0 | 4 | 0 | 0 |
| `openrouter` | 7 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 5 | 0 | 5 |
| `other` | 84 | 58 | 19 | 39 | 22 | 5 | 31 | 29 | 0 | 58 | 1 | 1 |
| `ALL` | 365 | 252 | 99 | 153 | 106 | 20 | 126 | 146 | 8 | 257 | 1 | 6 |

- The one entry with an object and without the `reasoning` parameter is `rekaai/reka-edge`. The six entries that list `reasoning` without an object are routers: `openrouter/auto`, `auto-code`, `auto-beta`, `pareto-code`, `free`, and `typesafe/jev-router`.
- `supports_max_tokens: true` is on 8 entries: `anthropic/claude-opus-4.6`, `qwen/qwen3.8-omni-flash`, `qwen3.8-flash`, `qwen3.7-flash`, `nvidia/nemotron-3-ultra-550b-a55b` and its `:free` twin, `nvidia/nemotron-3-super-120b-a12b`, `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free`.
- `meta-llama` (7 entries) has the object on 0 entries. Mistral has it on 3 of 21 (`mistral-large-4-0`, `mistral-medium-3-5`, `mistral-small-2603`).

**M37.** Effort lists by family. The seven words are shared, and the subsets are specific to a vendor and often to a model generation.

| Family | `supported_efforts` list (highest first) | `default_effort` values | Models | Example ids |
|-|-|-|-|-|
| `anthropic` | max,xhigh,high,medium,low | high x5 | 5 | `anthropic/claude-fable-5.2`, `anthropic/claude-sonnet-5.5`, `anthropic/claude-opus-5.5` |
| `anthropic` | max,xhigh,high,medium,low,none | high x4, medium x1 | 5 | `anthropic/claude-haiku-5.5`, `anthropic/claude-opus-5`, `anthropic/claude-sonnet-5` |
| `anthropic` | (key absent) | (absent) x4 | 4 | `anthropic/claude-opus-4.5`, `anthropic/claude-haiku-4.5`, `anthropic/claude-sonnet-4.5` |
| `anthropic` | max,high,medium,low,none | medium x1, high x1 | 2 | `anthropic/claude-sonnet-4.6`, `anthropic/claude-opus-4.6` |
| `openai` | max,xhigh,high,medium,low,none | medium x11 | 11 | `openai/gpt-6.1-terra`, `openai/gpt-6-luna-pro`, `openai/gpt-6-luna` |
| `openai` | max,xhigh,high,medium,low | medium x9 | 9 | `openai/gpt-6.2-astra-pro`, `openai/gpt-6.2-astra`, `openai/gpt-6.2-sol-pro` |
| `openai` | xhigh,high,medium,low,none | medium x7 | 7 | `openai/gpt-5.5`, `openai/gpt-5.4-image-2`, `openai/gpt-5.4-nano` |
| `openai` | (key absent) | (absent) x5 | 5 | `openai/gpt-oss-safeguard-20b`, `openai/gpt-5-image-mini`, `openai/gpt-5-image` |
| `openai` | high,medium,low | medium x4 | 4 | `openai/gpt-5.1-codex`, `openai/gpt-5.1-codex-mini`, `openai/gpt-oss-120b` |
| `openai` | xhigh,high,medium | medium x3 | 3 | `openai/gpt-5.5-pro`, `openai/gpt-5.4-pro`, `openai/gpt-5.2-pro` |
| `openai` | high,medium,low,minimal | medium x3 | 3 | `openai/gpt-5`, `openai/gpt-5-mini`, `openai/gpt-5-nano` |
| `openai` | xhigh,high,medium,low | medium x2 | 2 | `openai/gpt-5.2-codex`, `openai/gpt-5.1-codex-max` |
| `openai` | high,medium,low,none | medium x1 | 1 | `openai/gpt-5.1` |
| `openai` | high | high x1 | 1 | `openai/gpt-5-pro` |
| `google` | (key absent) | (absent) x10 | 10 | `google/gemini-3-pro-image`, `google/gemma-4-26b-a4b-it`, `google/gemma-4-26b-a4b-it:free` |
| `google` | high,medium,low | medium x6 | 6 | `google/gemini-4-carbon`, `google/gemini-4-argon`, `google/gemini-3.8-flash` |
| `google` | high,minimal,none | minimal x4 | 4 | `google/gemini-nano-banana-2.1`, `google/gemini-3.1-flash-lite-image`, `google/gemini-3.1-flash-image` |
| `google` | high,medium,low,minimal | medium x2, minimal x1 | 3 | `google/gemini-3.6-flash`, `google/gemini-3.5-flash-lite`, `google/gemini-3.5-flash` |
| `google` | high,medium,low,minimal,none | minimal x2, medium x1 | 3 | `google/gemini-3.1-flash-lite`, `google/gemini-3.1-flash-lite-preview`, `google/gemini-3-flash-preview` |
| `google` | xhigh,high,medium,low | medium x2 | 2 | `google/gemini-4-neon`, `google/gemini-4-hydrogen` |
| `google` | high,low | low x2 | 2 | `google/gemini-omni-1.2-flash`, `google/gemini-omni-1.1-flash` |
| `x-ai` | xhigh,high,medium,low | high x2, medium x1 | 3 | `x-ai/grok-4.7`, `x-ai/grok-4.6`, `x-ai/grok-4.20-multi-agent` |
| `x-ai` | high,medium,low | high x1 | 1 | `x-ai/grok-4.5` |
| `x-ai` | high,medium,low,none | low x1 | 1 | `x-ai/grok-4.3` |
| `x-ai` | (key absent) | (absent) x1 | 1 | `x-ai/grok-4.20` |
| `deepseek` | max,high,low,none | high x5 | 5 | `deepseek/deepseek-v4.1-flash-1017`, `deepseek/deepseek-v4.1-flash`, `deepseek/deepseek-v4-flash-vision-exp` |
| `deepseek` | (key absent) | (absent) x5 | 5 | `deepseek/deepseek-v3.2`, `deepseek/deepseek-v3.2-exp`, `deepseek/deepseek-v3.1-terminus` |
| `deepseek` | xhigh,high,none | high x2 | 2 | `deepseek/deepseek-v4-pro`, `deepseek/deepseek-v4-flash` |
| `mistralai` | high,none | high x3 | 3 | `mistralai/mistral-large-4-0`, `mistralai/mistral-medium-3-5`, `mistralai/mistral-small-2603` |
| `moonshotai` | (key absent) | (absent) x4 | 4 | `moonshotai/kimi-k2.7-code`, `moonshotai/kimi-k2.6`, `moonshotai/kimi-k2.5` |
| `moonshotai` | max,high,low,none | max x2 | 2 | `moonshotai/kimi-k3.1-pro`, `moonshotai/kimi-k3` |
| `qwen` | (key absent) | (absent) x21 | 21 | `qwen/qwen3.8-omni-flash`, `qwen/qwen3.8-flash`, `qwen/qwen3.7-flash` |
| `qwen` | xhigh,high,medium,low,minimal | xhigh x2 | 2 | `qwen/qwen3.8-max-prime`, `qwen/qwen3.8-max-0902` |
| `qwen` | xhigh,high,low,none | high x1 | 1 | `qwen/qwen4-max` |
| `qwen` | xhigh,medium,low,none | xhigh x1 | 1 | `qwen/qwen3.8-27b` |
| `qwen` | xhigh,medium,low | xhigh x1 | 1 | `qwen/qwen3.8-2.4t-a95b` |
| `z-ai` | (key absent) | (absent) x11 | 11 | `z-ai/glm-5.1`, `z-ai/glm-5v-turbo`, `z-ai/glm-5-turbo` |
| `z-ai` | max,high,low | max x5 | 5 | `z-ai/glm-5.3-flashx-legal`, `z-ai/glm-5.3-prime`, `z-ai/glm-5.3-flashx` |
| `z-ai` | xhigh,high,none | high x1 | 1 | `z-ai/glm-5.2` |
| `meta` | max,xhigh,high,medium,low,minimal | medium x3, max x1 | 4 | `meta/muse-ember`, `meta/muse-wonder`, `meta/muse-spark-1.3-contributor` |
| `meta` | xhigh,high,medium,low,minimal | medium x3 | 3 | `meta/muse-spark-1.2-contributor`, `meta/muse-spark-1.2`, `meta/muse-spark-1.1` |
| `meta` | xhigh,high,medium,low | medium x2 | 2 | `meta/muse-spark-open`, `meta/muse-glimmer-30b` |
| `minimax` | (key absent) | (absent) x3 | 3 | `minimax/minimax-m3`, `minimax/minimax-m2.7`, `minimax/minimax-m2.5` |
| `minimax` | max,xhigh,high,medium,low | max x1 | 1 | `minimax/minimax-m3.1-flash` |
| `amazon` | (key absent) | (absent) x1 | 1 | `amazon/nova-2-lite-v1` |
| `cohere` | (key absent) | (absent) x1 | 1 | `cohere/command-a-plus` |
| `perplexity` | (key absent) | (absent) x3 | 3 | `perplexity/sonar-pro-search`, `perplexity/sonar-reasoning-pro`, `perplexity/sonar-deep-research` |
| `nvidia` | (key absent) | (absent) x7 | 7 | `nvidia/nemotron-3.5-super-vl`, `nvidia/nemotron-3.5-super-vl:free`, `nvidia/nemotron-3.5-lightning` |
| `nvidia` | high,medium,none | high x2 | 2 | `nvidia/nemotron-3-ultra-550b-a55b`, `nvidia/nemotron-3-ultra-550b-a55b:free` |
| `nvidia` | medium,low,none | medium x1 | 1 | `nvidia/nemotron-3-super-120b-a12b` |
| `tencent` | high,low,none | high x3 | 3 | `tencent/hy4`, `tencent/hy4-preview`, `tencent/hy3` |
| `tencent` | (key absent) | (absent) x1 | 1 | `tencent/hy4-flash` |
| `other` | (key absent) | (absent) x29 | 29 | `inclusionai/ling-3.2-flash:free`, `liquid/lfm-3-1.7b`, `liquid/lfm-3-1.7b:free` |
| `other` | max,xhigh,high,medium,low,minimal,none | high x2, medium x2 | 4 | `particle-ai/particle-1`, `particle-ai/particle-1:free`, `upstage/solar-mini4` |
| `other` | max,high,medium,low,minimal,none | high x3 | 3 | `thinkingmachines/inkling-2-mini`, `thinkingmachines/inkling-small`, `thinkingmachines/inkling` |
| `other` | high,medium,low,minimal,none | medium x2, high x1 | 3 | `perceptron/perceptron-mk1.5`, `bytedance-seed/seed-2.0-lite`, `bytedance-seed/seed-2.0-mini` |
| `other` | max,xhigh,high | xhigh x2, high x1 | 3 | `sakana/fugu-ultra-v2`, `sakana/fugu-max`, `sakana/fugu-ultra` |
| `other` | max,high,low,none | max x2 | 2 | `baidu/ernie-light-dt-1.0-intl`, `fireworks/ember-1` |
| `other` | high,medium,low | medium x2 | 2 | `stepfun/step-5-preview`, `stepfun/step-3.7-flash` |
| `other` | max,high,low | high x2 | 2 | `aion-labs/aion-3.5-mini`, `aion-labs/aion-3.5` |
| `other` | high,medium,none | high x2 | 2 | `nex-agi/nex-n2.5-mini`, `nex-agi/nex-n2.5-pro` |
| `other` | xhigh,medium,low,none | xhigh x1 | 1 | `radixark/daisyrose-27b` |
| `other` | high | high x1 | 1 | `baidu/ernie-5.5-flash-preview:free` |
| `other` | high,medium | high x1 | 1 | `nex-agi/nex-n2.6` |
| `other` | max,high,medium,low,minimal | high x1 | 1 | `thinkingmachines/inkling-2-small` |
| `other` | max,xhigh,high,medium,low | max x1 | 1 | `reflection/beam-1.0-501b-a23b` |
| `other` | xhigh,medium,none | xhigh x1 | 1 | `prism-ml/ternary-bonsai-2-27b` |
| `other` | high,low,none | high x1 | 1 | `ibm-granite/granite-4.2-8b` |
| `other` | high,none | high x1 | 1 | `sakana/sakana-namazu` |

- 31 distinct lists appear across 146 interactive entries. Word frequency: `high` 141, `low` 121, `medium` 107, `xhigh` 75, `none` 73, `max` 65, `minimal` 33.
- `default_effort` values: `medium` 69, `high` 47, `max` 12, `xhigh` 8, `minimal` 7, `low` 3.
- In all 146 lists, `none` is in the list exactly when `mandatory` is false.
- Same vendor, different generations: Anthropic goes from a missing list (4.1 through 4.5) to `max,high,medium,low,none` (4.6) to `max,xhigh,high,medium,low,none` (4.7 through 5, Haiku 5.5) to `max,xhigh,high,medium,low` with `mandatory: true` (Fable, Opus 5.5, Sonnet 5.5). DeepSeek goes from a missing list (V3.x) to `xhigh,high,none` (V4 April) to `max,high,low,none` (V4 July and later). Z.ai goes from a missing list to `xhigh,high,none` (GLM 5.2) to `max,high,low` (GLM 5.3).

**M38.** The `endpoints` field by family, all 476 entries.

| Family | `chat + completions + messages + responses` | `batches` | `chat + completions + messages + responses + images` | `chat + completions + messages + responses + extractions` | `chat + completions + messages + responses + videos` |
|-|-|-|-|-|-|
| `anthropic` | 20 | 16 | 0 | 0 | 0 |
| `openai` | 64 | 44 | 3 | 0 | 0 |
| `google` | 30 | 12 | 7 | 0 | 2 |
| `x-ai` | 7 | 3 | 0 | 0 | 0 |
| `deepseek` | 17 | 3 | 0 | 0 | 0 |
| `mistralai` | 21 | 1 | 0 | 0 | 0 |
| `moonshotai` | 9 | 2 | 0 | 0 | 0 |
| `qwen` | 40 | 3 | 0 | 0 | 0 |
| `z-ai` | 19 | 3 | 0 | 0 | 0 |
| `meta-llama` | 7 | 0 | 0 | 0 | 0 |
| `meta` | 9 | 1 | 0 | 0 | 0 |
| `minimax` | 4 | 1 | 0 | 0 | 0 |
| `amazon` | 5 | 0 | 0 | 0 | 0 |
| `cohere` | 5 | 0 | 0 | 0 | 0 |
| `perplexity` | 5 | 0 | 0 | 0 | 0 |
| `nvidia` | 11 | 0 | 0 | 0 | 0 |
| `tencent` | 7 | 0 | 0 | 0 | 0 |
| `openrouter` | 7 | 0 | 0 | 0 | 0 |
| `other` | 82 | 3 | 0 | 3 | 0 |
| `ALL` | 369 | 92 | 10 | 3 | 2 |

- The 15 entries with a fifth route: 10 image-output entries (7 Google, 3 OpenAI) add `/api/v1/images`; `google/gemini-omni-1.2-flash` and `gemini-omni-1.1-flash` add `/api/v1/videos`; `scaledown/extract`, `inference-net/schematron-v2-turbo` and `schematron-v2-small` add `/api/alpha/extractions`.

**M39.** Input and output modalities by family, interactive entries.

| Family | Entries | in: text | in: image | in: file | in: audio | in: video | out: text | out: image | out: audio | out: video |
|-|-|-|-|-|-|-|-|-|-|-|
| `anthropic` | 16 | 16 | 16 | 16 | 0 | 0 | 16 | 0 | 0 | 0 |
| `openai` | 62 | 62 | 54 | 50 | 2 | 0 | 62 | 3 | 2 | 0 |
| `google` | 37 | 37 | 36 | 18 | 18 | 23 | 37 | 7 | 2 | 2 |
| `x-ai` | 6 | 6 | 6 | 6 | 0 | 0 | 6 | 0 | 0 | 0 |
| `deepseek` | 14 | 14 | 3 | 0 | 0 | 0 | 14 | 0 | 0 | 0 |
| `mistralai` | 21 | 21 | 12 | 11 | 1 | 0 | 21 | 0 | 0 | 0 |
| `moonshotai` | 8 | 8 | 5 | 0 | 0 | 2 | 8 | 0 | 0 | 0 |
| `qwen` | 40 | 40 | 24 | 0 | 1 | 19 | 40 | 0 | 0 | 0 |
| `z-ai` | 17 | 17 | 6 | 0 | 0 | 5 | 17 | 0 | 0 | 0 |
| `meta-llama` | 7 | 7 | 3 | 0 | 0 | 0 | 7 | 0 | 0 | 0 |
| `meta` | 9 | 9 | 9 | 8 | 3 | 8 | 9 | 0 | 0 | 0 |
| `minimax` | 4 | 4 | 2 | 0 | 0 | 2 | 4 | 0 | 0 | 0 |
| `amazon` | 5 | 5 | 4 | 1 | 0 | 1 | 5 | 0 | 0 | 0 |
| `cohere` | 5 | 5 | 1 | 0 | 0 | 0 | 5 | 0 | 0 | 0 |
| `perplexity` | 5 | 5 | 4 | 0 | 0 | 0 | 5 | 0 | 0 | 0 |
| `nvidia` | 11 | 11 | 4 | 0 | 1 | 3 | 11 | 0 | 0 | 0 |
| `tencent` | 7 | 7 | 2 | 0 | 0 | 0 | 7 | 0 | 0 | 0 |
| `openrouter` | 7 | 7 | 4 | 3 | 3 | 3 | 7 | 3 | 0 | 0 |
| `other` | 84 | 84 | 41 | 6 | 8 | 15 | 84 | 0 | 0 | 0 |
| `ALL` | 365 | 365 | 236 | 119 | 37 | 81 | 365 | 13 | 4 | 2 |

**M40.** Pricing keys by family, interactive entries.

| Family | Entries | `input_cache_read` | `input_cache_write` | `input_cache_write_1h` | `internal_reasoning` | `web_search` | `image` | `image_output` | `audio` | `audio_output` | `input_audio_cache` | `overrides` | `request` | `image_token` | `discount` | prompt = "-1" | prompt = "0" |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| `anthropic` | 16 | 16 | 16 | 16 | 0 | 16 | 0 | 0 | 0 | 0 | 0 | 2 | 0 | 0 | 0 | 0 | 0 |
| `openai` | 62 | 50 | 20 | 0 | 0 | 48 | 0 | 3 | 2 | 2 | 0 | 24 | 0 | 0 | 0 | 0 | 0 |
| `google` | 37 | 24 | 21 | 0 | 21 | 27 | 21 | 7 | 21 | 0 | 21 | 4 | 0 | 0 | 0 | 0 | 4 |
| `x-ai` | 6 | 6 | 0 | 0 | 0 | 6 | 0 | 0 | 0 | 0 | 0 | 6 | 0 | 0 | 0 | 0 | 0 |
| `deepseek` | 14 | 9 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| `mistralai` | 21 | 15 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `moonshotai` | 8 | 6 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `qwen` | 40 | 23 | 9 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 8 | 0 | 0 | 0 | 0 | 0 |
| `z-ai` | 17 | 16 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `meta-llama` | 7 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `meta` | 9 | 9 | 1 | 0 | 0 | 7 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `minimax` | 4 | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| `amazon` | 5 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `cohere` | 5 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `perplexity` | 5 | 0 | 0 | 0 | 1 | 5 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| `nvidia` | 11 | 4 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 4 |
| `tencent` | 7 | 4 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 |
| `openrouter` | 7 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 6 | 1 |
| `other` | 84 | 56 | 0 | 0 | 0 | 4 | 0 | 0 | 0 | 0 | 0 | 4 | 0 | 0 | 0 | 1 | 5 |
| `ALL` | 365 | 245 | 67 | 16 | 22 | 113 | 21 | 10 | 24 | 2 | 21 | 51 | 0 | 0 | 0 | 8 | 14 |

- `input_cache_write_1h` is on all 16 Anthropic entries and on 0 entries of any other family.
- `input_cache_write` among OpenAI entries is on the 20 entries of the GPT-5.6, GPT-6, GPT-6.1 and GPT-6.2 lines; the older OpenAI entries have `input_cache_read` alone.
- `internal_reasoning` is a Gemini key (21 entries) plus `perplexity/sonar-deep-research`.
- Override conditions over all 476 entries: OpenAI 52 entries with overrides, xAI 10, Qwen 8, Anthropic 5, Google 5, others 7. Time-window overrides are on `minimax/minimax-m3.1-flash`, `deepseek/deepseek-v4.1-flash-1017` and `tencent/hy4-preview`.

**M41.** `default_parameters` by family, interactive entries.

| Family | Entries | `{}` (empty object) | All listed keys null | Has a non-null value | Non-null values seen |
|-|-|-|-|-|-|
| `anthropic` | 16 | 12 | 3 | 1 | temperature=1 x1, top_p=1 x1 |
| `openai` | 62 | 22 | 40 | 0 |  |
| `google` | 37 | 13 | 20 | 4 | temperature=1 x4, top_p=0.95 x4, top_k=64 x4 |
| `x-ai` | 6 | 2 | 3 | 1 | temperature=0.7 x1, top_p=0.95 x1 |
| `deepseek` | 14 | 8 | 2 | 4 | temperature=1 x3, top_p=1 x2, top_p=0.95 x2, temperature=0.6 x1 |
| `mistralai` | 21 | 2 | 2 | 17 | temperature=0.3 x15, temperature=0.0645 x1, temperature=0.2 x1, top_p=0.95 x1 |
| `moonshotai` | 8 | 2 | 4 | 2 | top_p=0.95 x2 |
| `qwen` | 40 | 19 | 10 | 11 | top_p=0.95 x8, top_k=20 x8, temperature=1 x5, temperature=0.6 x3, temperature=0.7 x3, top_p=0.8 x3, repetition_penalty=1 x1 |
| `z-ai` | 17 | 0 | 0 | 17 | temperature=1 x12, top_p=0.95 x12, temperature=0.75 x3, temperature=0.8 x1, top_p=0.6 x1, temperature=0.6 x1 |
| `meta-llama` | 7 | 7 | 0 | 0 |  |
| `meta` | 9 | 8 | 0 | 1 | temperature=1 x1, top_p=0.95 x1, top_k=64 x1 |
| `minimax` | 4 | 1 | 0 | 3 | temperature=1 x3, top_p=0.95 x3 |
| `amazon` | 5 | 3 | 2 | 0 |  |
| `cohere` | 5 | 5 | 0 | 0 |  |
| `perplexity` | 5 | 4 | 1 | 0 |  |
| `nvidia` | 11 | 5 | 2 | 4 | top_p=0.95 x4, temperature=1 x3, temperature=0.6 x1 |
| `tencent` | 7 | 6 | 0 | 1 | temperature=0.9 x1, top_p=1 x1 |
| `openrouter` | 7 | 3 | 4 | 0 |  |
| `other` | 84 | 46 | 20 | 18 | top_p=0.95 x13, temperature=1 x10, top_k=20 x4, temperature=0.7 x3, top_k=40 x3, temperature=0.1 x3, top_k=50 x3, frequency_penalty=0 x3 |
| `ALL` | 365 | 168 | 113 | 84 | top_p=0.95 x51, temperature=1 x42, temperature=0.3 x15, top_k=20 x12, temperature=0.7 x7, temperature=0.6 x6, top_k=64 x5, top_p=1 x4 |

**M42.** A `:batch` or `:free` entry is its own row with its own limits and parameter list.

- 92 `:batch` entries: OpenAI 44, Anthropic 16, Google 12, and 20 across ten other prefixes. 89 have a base entry with the same `canonical_slug`; `openai/o4-mini:batch`, `openai/o3-mini:batch` and `openai/gpt-4-turbo:batch` are the only entries for their models. 17 of the 89 match the base entry in parameters, context and output limit.
- Parameters the base lists and the batch twin omits: `max_completion_tokens` on 51 twins, `logprobs` and `top_logprobs` on 9, `top_k` on 6, `parallel_tool_calls` on 5, `top_p` on 5, `temperature` on 4 (the Gemini 3.5 Flash Lite, 3.6, 3.7 and 3.8 Flash twins). Context differs on 5 twins, for example `qwen/qwen3.8-max-0902` 1000000 and its twin 262144. The batch prompt price is half the base price on 69 twins.
- 11 `:free` entries. Four are the only entry for their model: `inclusionai/ling-3.2-flash:free`, `baidu/ernie-5.5-flash-preview:free`, `recursive/flint-v1.0:free`, `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free`. Of the seven with a paid twin, five list fewer parameters and different limits: `google/gemma-4-31b-it:free` omits 11 parameters the paid entry lists (among them `structured_outputs`, `stop`, `top_k`, `logprobs`), and `nvidia/nemotron-3-ultra-550b-a55b:free` reports context 1000000 against 262144 on the paid entry.
- Docs quote that matches: "a `:free` entry can have a shorter context window than its base model, and its pricing is zero."
- Source for the quote: M-s11.

### E. Per-provider differences for the same model

**M43.** Method. The endpoints route was fetched for 59 slugs on 2026-11-22, with no credential, 0.4 seconds apart. All 59 returned 200. They hold 490 distinct endpoint records from 65 providers. Eighteen models are tabulated below; the raw JSON for all 59 is in the scratch subfolder `M\endpoints`, and tables for 17 more models are in `M\out` (`JF_*.md`, `JS_*.md`).

How to read the tables: "Parameters every endpoint lists" is the common set. The column "beyond the common set" shows what one endpoint lists in addition. The `tool_choice` column shows four letters for `none`, `auto`, `required`, `function`, with a dot where the endpoint's `supports_tool_choice` flag is false. Prices are dollars per million tokens. Source for all of section E: M-s26. Basis: OR-documented.

**M44.** Claude: Anthropic direct, Claude Platform on AWS, Amazon Bedrock, Google Vertex and Azure.

**`anthropic/claude-fable-5.2`** - 4 endpoints. Parameters every endpoint lists (10): `max_tokens`, `response_format`, `structured_outputs`, `stop`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`, `verbosity`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Google | `google-vertex/global` | - | unknown | 1000000 | 128000 | null | false | 8 | 40 | 0.2 | 10 / 16 | (none) | NA.. | bash=bash_20250124 | 0 |
| Azure | `azure` | - | unknown | 1000000 | 128000 | null | false | 8 | 40 | 0.2 | 10 / 16 | `max_completion_tokens` | NA.. | bash=bash_20250124 | 0 |
| Anthropic | `anthropic` | - | unknown | 1000000 | 128000 | null | false | 8 | 40 | 0.2 | 10 / 16 | (none) | NA.. | web_search=web_search_20260209, web_fetch=web_fetch_20260209, bash=bash_20250124, advisor=advisor_20260301 | 0 |
| Anthropic | `anthropic/fast` | priority | unknown | 1000000 | 128000 | null | false | 16 | 80 | 0.4 | 20 / 32 | (none) | NA.. | web_search=web_search_20260209, web_fetch=web_fetch_20260209, bash=bash_20250124, advisor=advisor_20260301 | 0 |

**`anthropic/claude-sonnet-5.5`** - 5 endpoints. Parameters every endpoint lists (9): `max_tokens`, `response_format`, `stop`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`, `verbosity`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Google | `google-vertex/global` | - | unknown | 1000000 | 128000 | null | false | 2 | 10 | 0.1 | 2.5 / 4 | `structured_outputs` | NA.. | bash=bash_20250124 | 0 |
| Amazon Bedrock | `amazon-bedrock` | - | unknown | 1000000 | 128000 | null | false | 2 | 10 | 0.1 | 2.5 / 4 | (none) | NA.. | bash=bash_20250124 | 0 |
| Azure | `azure/global` | - | unknown | 1000000 | 128000 | null | false | 2 | 10 | 0.1 | 2.5 / 4 | `max_completion_tokens`, `structured_outputs` | NA.. | bash=bash_20250124 | 0 |
| Claude Platform on AWS | `claude-on-aws` | - | unknown | 1000000 | 128000 | null | false | 2 | 10 | 0.1 | 2.5 / 4 | (none) | NA.. | web_search=web_search_20260209, web_fetch=web_fetch_20260209, bash=bash_20250124, advisor=advisor_20260301 | 0 |
| Anthropic | `anthropic` | - | unknown | 1000000 | 128000 | null | false | 2 | 10 | 0.1 | 2.5 / 4 | `structured_outputs` | NA.. | web_search=web_search_20260209, web_fetch=web_fetch_20260209, bash=bash_20250124, advisor=advisor_20260301 | 0 |

**`anthropic/claude-sonnet-4.5`** - 5 endpoints. Parameters every endpoint lists (9): `temperature`, `top_p`, `top_k`, `max_tokens`, `structured_outputs`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Claude Platform on AWS | `claude-on-aws` | - | unknown | 1000000 | 64000 | null | false | 3 | 15 | 0.3 | 3.75 / 6 | `response_format`, `stop` | NARF | web_search=web_search_20260209, bash=bash_20250124, advisor=advisor_20260301 | 0 |
| Azure | `azure/global` | - | unknown | 200000 | 64000 | null | false | 3 | 15 | 0.3 | 3.75 / 6 | `max_completion_tokens`, `response_format` | NARF | bash=bash_20250124 | 0 |
| Amazon Bedrock | `amazon-bedrock` | - | unknown | 1000000 | 64000 | null | false | 3 | 15 | 0.3 | 3.75 / 6 | `response_format`, `stop` | NARF | bash=bash_20250124 | 0 |
| Anthropic | `anthropic` | - | unknown | 1000000 | 64000 | null | false | 3 | 15 | 0.3 | 3.75 / 6 | `response_format`, `stop` | NARF | web_search=web_search_20260209, bash=bash_20250124, advisor=advisor_20260301 | 0 |
| Google | `google-vertex/global` | - | unknown | 1000000 | 64000 | null | false | 3 | 15 | 0.3 | 3.75 / 6 | `stop` | NARF | bash=bash_20250124 | 0 |

- In these three tables and in the Opus 5.5 and Haiku 5.5 tables in scratch, price, context and output limit are the same on every standard endpoint of one model, with one exception: `anthropic/claude-sonnet-4.5` on `azure/global` reports `context_length` 200000 where the other four report 1000000.
- `structured_outputs` is listed by some hosts and omitted by others for the same model: `anthropic/claude-opus-5.5` lists it on `anthropic`, `anthropic/fast`, `claude-on-aws`, `azure/global` and `google-vertex/global`, and the `amazon-bedrock` endpoint omits it. `anthropic/claude-sonnet-5.5` and `anthropic/claude-haiku-5.5` list it on `anthropic`, `azure/global` and `google-vertex/global`, and their `amazon-bedrock` and `claude-on-aws` endpoints omit it.
- `max_completion_tokens` is listed by the Azure endpoint alone on all six Claude models fetched (Fable 5.2, Opus 5.5, Sonnet 5.5, Haiku 5.5, Sonnet 4.5, Opus 4.6).
- Native tools differ by host: web search, web fetch and advisor are on the `anthropic` and `claude-on-aws` endpoints; Bedrock, Vertex and Azure endpoints list `bash` alone.
- `supports_tool_choice` is `NA..` on every endpoint of the four newest models and `NARF` on every endpoint of `claude-sonnet-4.5` (M24).
- `anthropic/claude-fable-5.2` has 4 endpoints (Anthropic standard and priority, Vertex, Azure). Its listing has 0 Amazon Bedrock and 0 Claude Platform on AWS records on 2026-11-22.

**M45.** OpenAI: OpenAI direct, Azure and Amazon Bedrock.

**`openai/gpt-6.2-sol`** - 5 endpoints. Parameters every endpoint lists (6): `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`, `verbosity`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| OpenAI | `openai` | - | unknown | 1050000 | 128000 | 922000 | false | 2 | 10 | 0.1 | 2.5 | `max_tokens`, `seed`, `response_format`, `structured_outputs` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |
| Azure | `azure` | - | unknown | 1050000 | 128000 | 922000 | false | 2 | 10 | 0.1 | 2.5 | `max_completion_tokens`, `seed`, `response_format`, `structured_outputs` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |
| Amazon Bedrock | `amazon-bedrock/global` | - | unknown | 1050000 | 128000 | 922000 | false | 2 | 10 | 0.1 | 2.5 | `max_tokens` | NARF | - | 0 |
| OpenAI | `openai/fast` | priority | unknown | 1050000 | 128000 | 922000 | false | 4 | 20 | 0.2 | 5 | `max_tokens`, `seed`, `response_format`, `structured_outputs` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |
| OpenAI | `openai/ultrafast` | ultrafast | unknown | 1050000 | 128000 | 922000 | false | 6 | 30 | 0.3 | 7.5 | `max_tokens`, `seed`, `response_format`, `structured_outputs` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |

**`openai/gpt-5.4`** - 4 endpoints. Parameters every endpoint lists (9): `seed`, `response_format`, `structured_outputs`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`, `verbosity`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| OpenAI | `openai/flex` | flex | unknown | 1050000 | 128000 | 922000 | true | 1.25 | 7.5 | 0.125 | - | `max_tokens` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |
| Azure | `azure` | - | unknown | 1050000 | 128000 | 922000 | true | 2.5 | 15 | 0.25 | - | `max_completion_tokens` | NARF | apply_patch=apply_patch | 0 |
| OpenAI | `openai` | - | unknown | 1050000 | 128000 | 922000 | true | 2.5 | 15 | 0.25 | - | `max_tokens` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |
| OpenAI | `openai/fast` | priority | unknown | 1050000 | 128000 | 922000 | true | 5 | 30 | 0.5 | - | `max_tokens` | NARF | web_search=web_search, apply_patch=apply_patch | 0 |

- On `openai/gpt-5.4`, `openai/gpt-5.2` and `openai/gpt-4.1` the lists differ in one name: the OpenAI endpoints list `max_tokens` and the Azure endpoint lists `max_completion_tokens`.
- On `openai/gpt-6.2-sol` the `amazon-bedrock/global` endpoint omits `seed`, `response_format` and `structured_outputs`, which the OpenAI and Azure endpoints list, and its `native_tools` object is empty.
- Azure's `native_tools` on `gpt-5.4` and `gpt-5.2` hold `apply_patch` alone; the OpenAI endpoints hold `web_search` and `apply_patch`.
- `openai/gpt-4.1` reports `max_completion_tokens` 942818 on `azure` and 32768 on `openai`.
- One model has up to four OpenAI price points: `openai/flex` (half price), `openai`, `openai/fast` (`priority`, double), `openai/ultrafast` (triple on `gpt-6.2-sol`).

**M46.** Gemini: Google AI Studio and Google Vertex.

**`google/gemini-4-carbon`** - 5 endpoints. Parameters every endpoint lists (11): `temperature`, `top_p`, `max_tokens`, `seed`, `response_format`, `structured_outputs`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Google AI Studio | `google-ai-studio/flex` | flex | unknown | 1048576 | 943718 | null | false | 1 | 5 | 0.05 | 0.0938 | (none) | NARF | web_search=google_search | 0 |
| Google | `google-vertex/global` | - | unknown | 1048576 | 943718 | null | false | 2 | 10 | 0.1 | 0.1875 | `stop` | NARF | web_search=google_search | 0 |
| Google AI Studio | `google-ai-studio` | - | unknown | 1048576 | 32768 | null | false | 2 | 10 | 0.1 | 0.1875 | (none) | NARF | web_search=google_search | 0 |
| Google | `google-vertex/global/priority` | priority | unknown | 1048576 | 943718 | null | false | 3.6 | 18 | 0.18 | 0.3375 | `stop` | NARF | web_search=google_search | 0 |
| Google AI Studio | `google-ai-studio/priority` | priority | unknown | 1048576 | 943718 | null | false | 3.6 | 18 | 0.18 | 0.3375 | (none) | NARF | web_search=google_search | 0 |

**`google/gemini-4-hydrogen`** - 5 endpoints. Parameters every endpoint lists (11): `temperature`, `top_p`, `max_tokens`, `seed`, `response_format`, `structured_outputs`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Google AI Studio | `google-ai-studio/flex` | flex | unknown | 1048576 | 943718 | null | false | 0.75 | 3.75 | 0.0375 | 0.0417 | (none) | NARF | web_search=google_search | 0 |
| Google | `google-vertex/global` | - | unknown | 1048576 | 943718 | null | false | 1.5 | 7.5 | 0.075 | 0.0833 | `stop` | NARF | web_search=google_search | 0 |
| Google AI Studio | `google-ai-studio` | - | unknown | 1048576 | 65536 | null | false | 1.5 | 7.5 | 0.075 | 0.0833 | (none) | NARF | web_search=google_search | 0 |
| Google | `google-vertex/global/priority` | priority | unknown | 1048576 | 943718 | null | false | 2.7 | 13.5 | 0.135 | 0.15 | `stop` | NARF | web_search=google_search | 0 |
| Google AI Studio | `google-ai-studio/priority` | priority | unknown | 1048576 | 943718 | null | false | 2.7 | 13.5 | 0.135 | 0.15 | (none) | NARF | web_search=google_search | 0 |

- `stop` is listed by the two Vertex endpoints and omitted by the three AI Studio endpoints on both models.
- The standard `google-ai-studio` endpoint reports a smaller output limit than its siblings: 32768 on `gemini-4-carbon` and 65536 on `gemini-4-hydrogen`, against 943718.
- Every endpoint in both tables has a cache-write price and `supports_implicit_caching: false`.

**M47.** xAI and Mistral.

**`x-ai/grok-4.7`** - 5 endpoints. Parameters every endpoint lists (10): `temperature`, `top_p`, `max_tokens`, `response_format`, `structured_outputs`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Amazon Bedrock | `amazon-bedrock/global` | - | unknown | 500000 | 450000 | null | false | 2 | 6 | 0.5 | 0 | `top_k`, `stop` | NARF | - | 0 |
| xAI | `xai/zdr` | - | unknown | 500000 | 450000 | null | true | 2 | 6 | 0.5 | - | `logprobs`, `top_logprobs`, `seed` | NARF | web_search=web_search | 0 |
| xAI | `xai` | - | unknown | 500000 | 450000 | null | true | 2 | 6 | 0.5 | - | `logprobs`, `top_logprobs`, `seed` | NARF | web_search=web_search | 0 |
| xAI | `xai/zdr/priority` | priority | unknown | 500000 | 450000 | null | true | 4 | 12 | 1 | - | `logprobs`, `top_logprobs`, `seed` | NARF | web_search=web_search | 0 |
| xAI | `xai/priority` | priority | unknown | 500000 | 450000 | null | true | 4 | 12 | 1 | - | `logprobs`, `top_logprobs`, `seed` | NARF | web_search=web_search | 0 |

**`mistralai/mistral-large-4-0`** - 2 endpoints. Parameters every endpoint lists (16): `temperature`, `top_p`, `frequency_penalty`, `presence_penalty`, `max_tokens`, `logprobs`, `top_logprobs`, `seed`, `response_format`, `structured_outputs`, `stop`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`, `reasoning_effort`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Mistral | `mistral/zdr` | - | unknown | 1048576 | 262144 | null | true | 0.7 | 2.1 | 0.07 | - | (none) | NARF | - | 0 |
| Mistral | `mistral/eu` | - | unknown | 524288 | 262144 | null | true | 0.77 | 2.31 | 0.077 | - | (none) | NARF | - | 0 |

- `x-ai/grok-4.7`: the four xAI endpoints list `logprobs`, `top_logprobs` and `seed`; the `amazon-bedrock/global` endpoint omits those three and lists `top_k` and `stop`. `supports_implicit_caching` is true on xAI and false on Bedrock. `native_tools` has web search on xAI and is empty on Bedrock.
- `mistralai/mistral-large-4-0`: both endpoints are Mistral's own. The `mistral/eu` endpoint reports half the context (524288 against 1048576) and a price 10 percent higher.

**M48.** Open-weight models, full tables.

**`meta-llama/llama-3.3-70b-instruct`** - 7 endpoints. Parameters every endpoint lists (4): `temperature`, `top_p`, `max_tokens`, `stop`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| DeepInfra | `deepinfra/turbo` | - | fp8 | 131072 | 16384 | null | false | 0.1 | 0.32 | - | - | `top_k`, `min_p`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `logit_bias`, `seed`, `response_format`, `structured_outputs`, `tools`, `tool_choice` | NA.. | - | 0 |
| AkashML | `akashml/fp8` | - | fp8 | 131072 | 128000 | null | true | 0.2 | 0.52 | 0.1 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `logprobs`, `top_logprobs`, `seed`, `response_format`, `structured_outputs`, `tools`, `tool_choice` | NARF | - | 0 |
| Parasail | `parasail/fp8` | - | fp8 | 131072 | 16384 | null | true | 0.22 | 0.5 | 0.11 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `logit_bias`, `logprobs`, `top_logprobs`, `seed`, `response_format`, `structured_outputs` | NARF | - | 0 |
| Cloudflare | `cloudflare/fp8` | - | fp8 | 24000 | 21600 | null | false | 0.293 | 2.253 | - | - | `top_k`, `min_p`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `logit_bias`, `logprobs`, `top_logprobs`, `seed`, `response_format` | NARF | - | 0 |
| Groq | `groq` | - | unknown | 131072 | 32768 | null | true | 0.59 | 0.79 | 0.295 | - | `seed`, `response_format`, `tools`, `tool_choice` | NARF | - | 0 |
| CoreWeave | `coreweave/fp16` | - | fp16 | 128000 | 115200 | null | true | 0.71 | 0.71 | 0.71 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `logprobs`, `top_logprobs`, `seed`, `response_format`, `structured_outputs`, `tools`, `tool_choice` | NARF | - | 0 |
| Amazon Bedrock | `amazon-bedrock/us-east-1` | - | unknown | 131072 | 8192 | null | false | 0.72 | 0.72 | - | - | `top_k`, `tools`, `tool_choice` | NA.. | - | 0 |

**`deepseek/deepseek-v3.2`** - 4 endpoints. Parameters every endpoint lists (5): `temperature`, `top_p`, `max_tokens`, `include_reasoning`, `reasoning`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| SiliconFlow | `siliconflow/fp8` | - | fp8 | 163840 | 147456 | null | true | 0.259 | 0.8 | 0.14 | - | `top_k`, `frequency_penalty`, `response_format`, `structured_outputs`, `tools`, `tool_choice` | NARF | - | 0 |
| DeepInfra | `deepinfra/fp4` | - | fp4 | 163840 | 147456 | null | true | 0.26 | 0.38 | 0.13 | - | `top_k`, `min_p`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `logit_bias`, `seed`, `response_format`, `structured_outputs`, `stop`, `tools`, `tool_choice` | NARF | - | 0 |
| Baidu | `baidu/fp8` | - | fp8 | 131072 | 65536 | null | true | 0.28 | 0.42 | 0.028 | - | `seed`, `response_format`, `structured_outputs`, `stop` | NARF | - | 0 |
| DigitalOcean | `digitalocean` | - | unknown | 163840 | 147456 | null | true | 0.3 | 0.96 | 0.09 | - | `logprobs`, `top_logprobs` | NARF | - | 0 |

**`z-ai/glm-5.1`** - 8 endpoints. Parameters every endpoint lists (7): `temperature`, `top_p`, `max_tokens`, `tools`, `tool_choice`, `include_reasoning`, `reasoning`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| Baidu | `baidu/fp8` | - | fp8 | 202752 | 131072 | null | true | 0.8988 | 2.8248 | 0.1669 | - | `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `response_format`, `structured_outputs`, `stop` | NA.. | - | 0 |
| Chutes | `chutes/fp8` | - | fp8 | 202752 | 65535 | null | true | 0.9 | 2.9 | 0.09 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `seed`, `response_format`, `structured_outputs`, `stop` | NA.F | - | 0 |
| StreamLake | `streamlake/fp8` | - | fp8 | 200000 | 128000 | null | true | 0.966 | 3.036 | 0.1794 | - | `logprobs`, `top_logprobs`, `response_format`, `structured_outputs`, `stop` | NA.. | - | 0 |
| SiliconFlow | `siliconflow/fp8` | - | fp8 | 204800 | 131072 | null | true | 1.19 | 3.74 | 0.6 | - | `top_k`, `frequency_penalty` | NAR. | - | 0 |
| Alibaba | `alibaba/fp8` | - | fp8 | 202745 | 131072 | 169984 | true | 1.33 | 4.18 | 0.247 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `logprobs`, `top_logprobs`, `seed`, `response_format`, `structured_outputs`, `stop` | NA.. | - | 0 |
| Novita | `novita/fp8` | - | fp8 | 204800 | 131072 | null | true | 1.38 | 4.4 | 0.26 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty`, `seed`, `stop` | NA.. | - | 0 |
| Z.AI | `z-ai/fp8` | - | fp8 | 202752 | 131072 | null | true | 1.4 | 4.4 | 0.26 | - | `top_k`, `response_format` | .A.. | - | 0 |
| Venice | `venice/fp8` | - | fp8 | 200000 | 80000 | null | true | 1.4014 | 4.4044 | 0.2603 | - | `top_k`, `frequency_penalty`, `presence_penalty`, `response_format`, `structured_outputs`, `stop` | .A.. | - | 0 |

**`qwen/qwen3.5-397b-a17b`** - 8 endpoints. Parameters every endpoint lists (8): `temperature`, `top_p`, `presence_penalty`, `max_tokens`, `response_format`, `stop`, `include_reasoning`, `reasoning`.

| Provider | Tag | Tier | Quantization | Context | Max output | Max prompt | Implicit caching | Input $/M | Output $/M | Cache read $/M | Cache write $/M (5m / 1h) | Parameters this endpoint lists beyond the common set | `tool_choice` values (N A R F) | Native tools | Status |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
| DeepInfra | `deepinfra/fp8/flex` | flex | fp8 | 262144 | 81920 | null | true | 0.36 | 2.4 | 0.176 | - | `top_k`, `min_p`, `frequency_penalty`, `repetition_penalty`, `logit_bias`, `seed`, `structured_outputs`, `tools`, `tool_choice` | NARF | - | 0 |
| Alibaba | `alibaba` | - | unknown | 262144 | 65536 | 258048 | false | 0.39 | 2.34 | - | - | `top_k`, `frequency_penalty`, `logprobs`, `top_logprobs`, `seed`, `tools`, `tool_choice` | NA.. | - | 0 |
| DeepInfra | `deepinfra/fp8` | - | fp8 | 262144 | 81920 | null | true | 0.45 | 3 | 0.22 | - | `top_k`, `min_p`, `frequency_penalty`, `repetition_penalty`, `logit_bias`, `seed`, `structured_outputs`, `tools`, `tool_choice` | .A.. | - | 0 |
| Parasail | `parasail/fp8` | - | fp8 | 262144 | 235929 | null | true | 0.5 | 3.6 | 0.3 | - | `top_k`, `frequency_penalty`, `repetition_penalty`, `logit_bias`, `logprobs`, `top_logprobs`, `seed`, `structured_outputs`, `tools`, `tool_choice` | .ARF | - | 0 |
| StreamLake | `streamlake` | - | unknown | 256000 | 64000 | null | true | 0.6 | 3.6 | 0.12 | - | `logprobs`, `top_logprobs`, `structured_outputs` | NARF | - | 0 |
| Novita | `novita` | - | unknown | 262144 | 65536 | null | false | 0.6 | 3.6 | - | - | `top_k`, `frequency_penalty`, `repetition_penalty`, `logprobs`, `top_logprobs`, `seed`, `tools`, `tool_choice` | NA.. | - | 0 |
| DeepInfra | `deepinfra/fp8/priority` | priority | fp8 | 262144 | 81920 | null | true | 0.675 | 4.5 | 0.33 | - | `top_k`, `min_p`, `frequency_penalty`, `repetition_penalty`, `logit_bias`, `seed`, `structured_outputs`, `tools`, `tool_choice` | NARF | - | 0 |
| Venice | `venice` | - | unknown | 128000 | 32768 | null | false | 0.75 | 4.5 | - | - | `top_k`, `frequency_penalty`, `logprobs`, `top_logprobs`, `structured_outputs`, `tools`, `tool_choice` | NARF | - | -2 |

**M49.** Open-weight models with many hosts, summarized per parameter.

**`deepseek/deepseek-v4.1-flash`** - 37 endpoints from 28 providers.

| Parameter | Endpoints listing it | Providers whose endpoint does not list it |
|-|-|-|
| `temperature` | 37 of 37 | (all list it) |
| `top_p` | 37 of 37 | (all list it) |
| `top_k` | 30 of 37 | lacking: baidu/fp8, baseten, baseten/us, digitalocean, modal, river, streamlake/fp8 |
| `min_p` | 14 of 37 | only: cloudflare, crusoe/fp8, deepinfra/fp8, deepinfra/fp8/flex, deepinfra/fp8/priority, inference-net/fp8, inference-net/us, makora/fp8, mancer/fp8, open-inference/fp4, relace/fp4, together, wafer/mxfp4, wafer/us |
| `top_a` | 1 of 37 | only: mancer/fp8 |
| `frequency_penalty` | 32 of 37 | lacking: baidu/fp8, digitalocean, modal, river, streamlake/fp8 |
| `presence_penalty` | 31 of 37 | lacking: baidu/fp8, digitalocean, modal, river, siliconflow/fp8, streamlake/fp8 |
| `repetition_penalty` | 23 of 37 | lacking: alibaba, baidu/fp8, baseten, baseten/us, digitalocean, ionstream, modal, nebius/fp8, reka, river, siliconflow/fp8, streamlake/fp8, venice/fp8, venice/ultrafast |
| `max_tokens` | 37 of 37 | (all list it) |
| `logit_bias` | 21 of 37 | lacking: alibaba, baidu/fp8, baseten, baseten/us, cohere/fp8, coreweave/fp8, digitalocean, google-vertex/global, modal, novita/fp8, reka, river, siliconflow/fp8, streamlake/fp8, venice/fp8, venice/ultrafast |
| `logprobs` | 19 of 37 | lacking: alibaba, baidu/fp8, baseten, baseten/us, crusoe/fp8, deepinfra/fp8, deepinfra/fp8/flex, deepinfra/fp8/priority, inference-net/fp8, inference-net/us, modal, open-inference/fp4, parasail/fp8, relace/fp4, siliconflow/fp8, venice/fp8, venice/ultrafast, wafer/us |
| `top_logprobs` | 19 of 37 | lacking: alibaba, baidu/fp8, baseten, baseten/us, crusoe/fp8, deepinfra/fp8, deepinfra/fp8/flex, deepinfra/fp8/priority, inference-net/fp8, inference-net/us, modal, open-inference/fp4, parasail/fp8, relace/fp4, siliconflow/fp8, venice/fp8, venice/ultrafast, wafer/us |
| `seed` | 23 of 37 | lacking: baidu/fp8, baseten, baseten/us, digitalocean, fireworks, fireworks/spot, fireworks/us, modal, relace/fp4, siliconflow/fp8, streamlake/fp8, together, venice/fp8, venice/ultrafast |
| `response_format` | 36 of 37 | lacking: baseten |
| `structured_outputs` | 34 of 37 | lacking: alibaba, siliconflow/fp8, streamlake/fp8 |
| `stop` | 35 of 37 | lacking: digitalocean, siliconflow/fp8 |
| `tools` | 37 of 37 | (all list it) |
| `tool_choice` | 37 of 37 | (all list it) |
| `parallel_tool_calls` | 3 of 37 | only: baseten, baseten/us, cohere/fp8 |
| `include_reasoning` | 37 of 37 | (all list it) |
| `reasoning` | 37 of 37 | (all list it) |
| `reasoning_effort` | 37 of 37 | (all list it) |

- Quantization: fp8 x19, unknown x14, fp4 x2, mxfp4 x2
- Context length: 1048576 x31, 1040000 x2, 1024000 x1, 1000000 x3
- Max output (`max_completion_tokens`): 943718 x20, 393216 x11, 384000 x2, 262144 x3, 131072 x1
- `supports_implicit_caching`: true x37, false x0
- `supports_tool_choice` (none, auto, required, function): NARF x30, NA.F x2, .A.. x2, .... x1, NA.. x1, .ARF x1
- Service tier: None x33, priority x2, flex x1, ultrafast x1
- Input price range: 0.0498 to 1.5 $/M; output price range: 0.1992 to 6 $/M; endpoints with an `input_cache_read` price: 37 of 37
- Status: 0 x37

**`moonshotai/kimi-k3`** - 29 endpoints from 20 providers.

| Parameter | Endpoints listing it | Providers whose endpoint does not list it |
|-|-|-|
| `temperature` | 27 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2 |
| `top_p` | 27 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2 |
| `top_k` | 23 of 29 | lacking: alibaba, baseten/fp8, digitalocean, modal/mxfp4, moonshotai, river |
| `min_p` | 9 of 29 | only: deepinfra/mxfp4, inference-net/fast, inference-net/fp4, inference-net/us, inference-net/us/fast, together, wafer, wafer/fast, wafer/us |
| `frequency_penalty` | 23 of 29 | lacking: alibaba, amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, digitalocean, modal/mxfp4, river |
| `presence_penalty` | 24 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, digitalocean, modal/mxfp4, river |
| `repetition_penalty` | 19 of 29 | lacking: alibaba, amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, baseten/fp8, digitalocean, modal/mxfp4, moonshotai, nebius/fp4, river, sail-research/fp4 |
| `max_tokens` | 29 of 29 | (all list it) |
| `logit_bias` | 15 of 29 | lacking: alibaba, amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, baseten/fp8, chutes/mxfp4, digitalocean, io-net/fp4, kt, makora/fp4, modal/mxfp4, moonshotai, relace/fp4, river, sail-research/fp4 |
| `logprobs` | 17 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, baseten/fp8, chutes/mxfp4, deepinfra/mxfp4, inference-net/fast, inference-net/fp4, inference-net/us, inference-net/us/fast, moonshotai, relace/fp4, sail-research/fp4 |
| `top_logprobs` | 17 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, baseten/fp8, chutes/mxfp4, deepinfra/mxfp4, inference-net/fast, inference-net/fp4, inference-net/us, inference-net/us/fast, moonshotai, relace/fp4, sail-research/fp4 |
| `seed` | 19 of 29 | lacking: baseten/fp8, digitalocean, fireworks, fireworks/fast, fireworks/us, kt, modal/mxfp4, moonshotai, relace/fp4, together |
| `response_format` | 25 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, baseten/fp8, relace/fp4 |
| `structured_outputs` | 25 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, baseten/fp8, relace/fp4 |
| `stop` | 26 of 29 | lacking: alibaba, digitalocean, kt |
| `tools` | 26 of 29 | lacking: chutes/mxfp4, parasail/fast, river |
| `tool_choice` | 23 of 29 | lacking: amazon-bedrock/us-east-1, amazon-bedrock/us-east-2, chutes/mxfp4, modal/mxfp4, parasail/fast, river |
| `include_reasoning` | 29 of 29 | (all list it) |
| `reasoning` | 29 of 29 | (all list it) |
| `reasoning_effort` | 29 of 29 | (all list it) |

- Quantization: fp4 x11, unknown x11, mxfp4 x6, fp8 x1
- Context length: 1048576 x27, 1048320 x1, 1000000 x1
- Max output (`max_completion_tokens`): 1048575 x1, 943718 x22, 900000 x1, 262144 x1, 131072 x1, 128000 x2, 65535 x1
- `supports_implicit_caching`: true x27, false x2
- `supports_tool_choice` (none, auto, required, function): NARF x22, N.RF x2, NAR. x2, NA.. x1, .ARF x1, .... x1
- Service tier: None x24, priority x5
- Input price range: 0.153 to 4.5 $/M; output price range: 9.5 to 22.5 $/M; endpoints with an `input_cache_read` price: 29 of 29
- Status: 0 x29

**`z-ai/glm-5.3`** - 46 endpoints from 34 providers.

| Parameter | Endpoints listing it | Providers whose endpoint does not list it |
|-|-|-|
| `temperature` | 46 of 46 | (all list it) |
| `top_p` | 46 of 46 | (all list it) |
| `top_k` | 35 of 46 | lacking: baidu/fp8, baseten/fast, baseten/fp4, baseten/fp4/us, baseten/us/fast, digitalocean, inceptron/fp4, mistral/nvfp4/eu, mistral/zdr, modal, modelrun/fp4 |
| `min_p` | 14 of 46 | only: cloudflare, crusoe/fp4, decart/fp4, deepinfra/fp4, deepinfra/fp4/flex, doubleword/nvfp4, featherless, friendli, inference-net/fp4, inference-net/us, morph/fp8, together, wafer/fp4, wafer/us |
| `frequency_penalty` | 41 of 46 | lacking: amazon-bedrock/us-east-1, digitalocean, modal, sambanova, z-ai/fp8 |
| `presence_penalty` | 40 of 46 | lacking: amazon-bedrock/us-east-1, digitalocean, modal, sambanova, siliconflow/fp8, z-ai/fp8 |
| `repetition_penalty` | 28 of 46 | lacking: alibaba, alibaba/prime, amazon-bedrock/us-east-1, baseten/fast, baseten/fp4, baseten/fp4/us, baseten/us/fast, digitalocean, inceptron/fp4, mistral/nvfp4/eu, mistral/zdr, modal, sail-research/fp8, sail-research/us, sambanova, siliconflow/fp8, venice, z-ai/fp8 |
| `max_tokens` | 46 of 46 | (all list it) |
| `logit_bias` | 18 of 46 | only: cloudflare, crusoe/fp4, deepinfra/fp4, deepinfra/fp4/flex, doubleword/nvfp4, fireworks, fireworks/fast, fireworks/spot, inceptron/fp4, inference-net/fp4, inference-net/us, morph/fp8, nebius/fp4, parasail/fast, parasail/fp4, together, wafer/fp4, wafer/us |
| `logprobs` | 24 of 46 | lacking: amazon-bedrock/us-east-1, baidu/fp8, baseten/fast, baseten/fp4, baseten/fp4/us, baseten/us/fast, crusoe/fp4, decart/fp4, deepinfra/fp4, deepinfra/fp4/flex, doubleword/nvfp4, featherless, friendli, inference-net/fp4, inference-net/us, modelrun/fp4, relace/fp4, sail-research/fp8, sail-research/us, siliconflow/fp8, venice, z-ai/fp8 |
| `top_logprobs` | 24 of 46 | lacking: amazon-bedrock/us-east-1, baidu/fp8, baseten/fast, baseten/fp4, baseten/fp4/us, baseten/us/fast, crusoe/fp4, decart/fp4, deepinfra/fp4, deepinfra/fp4/flex, doubleword/nvfp4, featherless, friendli, inference-net/fp4, inference-net/us, modelrun/fp4, relace/fp4, sail-research/fp8, sail-research/us, siliconflow/fp8, venice, z-ai/fp8 |
| `seed` | 30 of 46 | lacking: baidu/fp8, baseten/fast, baseten/fp4, baseten/fp4/us, baseten/us/fast, digitalocean, fireworks, fireworks/fast, fireworks/spot, modal, modelrun/fp4, sambanova, siliconflow/fp8, together, venice, z-ai/fp8 |
| `response_format` | 44 of 46 | lacking: baseten/fast, baseten/us/fast |
| `structured_outputs` | 38 of 46 | lacking: alibaba, alibaba/prime, baseten/fast, baseten/us/fast, featherless, inceptron/fp4, relace/fp4, siliconflow/fp8 |
| `stop` | 43 of 46 | lacking: digitalocean, siliconflow/fp8, z-ai/fp8 |
| `tools` | 46 of 46 | (all list it) |
| `tool_choice` | 45 of 46 | lacking: amazon-bedrock/us-east-1 |
| `parallel_tool_calls` | 2 of 46 | only: cohere/nvfp4, inceptron/fp4 |
| `include_reasoning` | 46 of 46 | (all list it) |
| `reasoning` | 46 of 46 | (all list it) |
| `reasoning_effort` | 46 of 46 | (all list it) |

- Quantization: fp4 x18, unknown x14, fp8 x9, nvfp4 x5
- Context length: 1048576 x40, 1048575 x1, 1000000 x3, 786432 x1, 262144 x1
- Max output (`max_completion_tokens`): 943718 x21, 943717 x1, 707788 x1, 524288 x1, 262144 x5, 131072 x14, 128000 x2, 32768 x1
- `supports_implicit_caching`: true x45, false x1
- `supports_tool_choice` (none, auto, required, function): NARF x27, NA.F x9, NA.. x8, .A.. x1, NAR. x1
- Service tier: None x40, priority x5, flex x1
- Input price range: 0.0699 to 2.1 $/M; output price range: 1.4 to 6.6 $/M; endpoints with an `input_cache_read` price: 46 of 46
- Status: 0 x46

**`openai/gpt-oss-120b`** - 24 endpoints from 21 providers.

| Parameter | Endpoints listing it | Providers whose endpoint does not list it |
|-|-|-|
| `temperature` | 24 of 24 | (all list it) |
| `top_p` | 24 of 24 | (all list it) |
| `top_k` | 18 of 24 | lacking: baseten/fp4, baseten/fp4/us, cerebras/fp16, dekallm/bf16, digitalocean, groq |
| `min_p` | 5 of 24 | only: crusoe/bf16, deepinfra/bf16, mancer/fp8, primeintellect/fp4, together |
| `top_a` | 2 of 24 | only: mancer/fp8, primeintellect/fp4 |
| `frequency_penalty` | 18 of 24 | lacking: amazon-bedrock, amazon-bedrock/eu-west-1, digitalocean, google-vertex/global, groq, sambanova |
| `presence_penalty` | 17 of 24 | lacking: amazon-bedrock, amazon-bedrock/eu-west-1, digitalocean, google-vertex/global, groq, sambanova, siliconflow/fp8 |
| `repetition_penalty` | 13 of 24 | lacking: amazon-bedrock, amazon-bedrock/eu-west-1, baseten/fp4, baseten/fp4/us, cerebras/fp16, dekallm/bf16, digitalocean, groq, sambanova, siliconflow/fp8, venice/int4 |
| `max_tokens` | 24 of 24 | (all list it) |
| `logit_bias` | 11 of 24 | only: cerebras/fp16, crusoe/bf16, deepinfra/bf16, dekallm/bf16, fireworks, mancer/fp8, nebius/fp4, parasail/fast, parasail/fp4, primeintellect/fp4, together |
| `logprobs` | 10 of 24 | only: akashml/fp4, cerebras/fp16, coreweave/fp4, dekallm/bf16, digitalocean, fireworks, mancer/fp8, near-ai/fp4, parasail/fast, parasail/fp4 |
| `top_logprobs` | 10 of 24 | only: akashml/fp4, cerebras/fp16, coreweave/fp4, dekallm/bf16, digitalocean, fireworks, mancer/fp8, near-ai/fp4, parasail/fast, parasail/fp4 |
| `seed` | 14 of 24 | lacking: amazon-bedrock, amazon-bedrock/eu-west-1, baseten/fp4, baseten/fp4/us, digitalocean, fireworks, sambanova, siliconflow/fp8, together, venice/int4 |
| `response_format` | 19 of 24 | lacking: amazon-bedrock, amazon-bedrock/eu-west-1, digitalocean, sambanova, venice/int4 |
| `structured_outputs` | 19 of 24 | lacking: amazon-bedrock, amazon-bedrock/eu-west-1, digitalocean, sambanova, venice/int4 |
| `stop` | 21 of 24 | lacking: digitalocean, google-vertex/global, siliconflow/fp8 |
| `tools` | 21 of 24 | lacking: digitalocean, google-vertex/global, siliconflow/fp8 |
| `tool_choice` | 21 of 24 | lacking: digitalocean, google-vertex/global, siliconflow/fp8 |
| `include_reasoning` | 24 of 24 | (all list it) |
| `reasoning` | 24 of 24 | (all list it) |
| `reasoning_effort` | 24 of 24 | (all list it) |

- Quantization: fp4 x9, unknown x8, bf16 x3, fp8 x2, int4 x1, fp16 x1
- Context length: 131072 x20, 128072 x2, 128000 x2
- Max output (`max_completion_tokens`): 117964 x16, 115264 x2, 65536 x2, 40960 x1, 32768 x1, 8192 x1, 4096 x1
- `supports_implicit_caching`: true x18, false x6
- `supports_tool_choice` (none, auto, required, function): NARF x10, .A.. x9, .ARF x4, .... x1
- Service tier: None x23, priority x1
- Input price range: 0.03 to 0.35 $/M; output price range: 0.15 to 0.95 $/M; endpoints with an `input_cache_read` price: 18 of 24
- Status: 0 x22, -5 x2

**`qwen/qwen3.8-27b`** - 23 endpoints from 23 providers.

| Parameter | Endpoints listing it | Providers whose endpoint does not list it |
|-|-|-|
| `temperature` | 23 of 23 | (all list it) |
| `top_p` | 23 of 23 | (all list it) |
| `top_k` | 19 of 23 | lacking: cerebras/fp16, dekallm, modelrun/fp4, streamlake |
| `min_p` | 5 of 23 | only: cloudflare, deepinfra/fp8, mancer/fp8, primeintellect/fp8, reka/fp8 |
| `top_a` | 2 of 23 | only: mancer/fp8, primeintellect/fp8 |
| `frequency_penalty` | 19 of 23 | lacking: modal, near-ai/fp8, streamlake, tenstorrent/fp8 |
| `presence_penalty` | 19 of 23 | lacking: modal, near-ai/fp8, streamlake, tenstorrent/fp8 |
| `repetition_penalty` | 13 of 23 | lacking: alibaba, cerebras/fp16, dekallm, modal, near-ai/fp8, nebius/fp4, siliconflow/fp8, streamlake, tenstorrent/fp8, venice/fp8 |
| `max_tokens` | 23 of 23 | (all list it) |
| `logit_bias` | 10 of 23 | only: cerebras/fp16, cloudflare, deepinfra/fp8, dekallm, ionstream/fp8, mancer/fp8, nebius/fp4, parasail/fp8, primeintellect/fp8, reka/fp8 |
| `logprobs` | 16 of 23 | lacking: chutes/fp8, deepinfra/fp8, modelrun/fp4, primeintellect/fp8, siliconflow/fp8, tenstorrent/fp8, venice/fp8 |
| `top_logprobs` | 16 of 23 | lacking: chutes/fp8, deepinfra/fp8, modelrun/fp4, primeintellect/fp8, siliconflow/fp8, tenstorrent/fp8, venice/fp8 |
| `seed` | 18 of 23 | lacking: modal, modelrun/fp4, siliconflow/fp8, streamlake, venice/fp8 |
| `response_format` | 22 of 23 | lacking: cerebras/fp16 |
| `structured_outputs` | 18 of 23 | lacking: alibaba, cerebras/fp16, novita, siliconflow/fp8, streamlake |
| `stop` | 22 of 23 | lacking: siliconflow/fp8 |
| `tools` | 23 of 23 | (all list it) |
| `tool_choice` | 23 of 23 | (all list it) |
| `include_reasoning` | 23 of 23 | (all list it) |
| `reasoning` | 23 of 23 | (all list it) |
| `reasoning_effort` | 23 of 23 | (all list it) |

- Quantization: fp8 x13, unknown x6, fp4 x3, fp16 x1
- Context length: 1024000 x1, 1000000 x3, 262144 x18, 65536 x1
- Max output (`max_completion_tokens`): 900000 x1, 235929 x11, 131072 x4, 128000 x1, 65536 x5, 32768 x1
- `supports_implicit_caching`: true x21, false x2
- `supports_tool_choice` (none, auto, required, function): NARF x11, .ARF x6, NA.. x5, .A.. x1
- Service tier: None x23
- Input price range: 0.035 to 0.99 $/M; output price range: 1.05 to 3.2 $/M; endpoints with an `input_cache_read` price: 22 of 23
- Status: 0 x23

**M50.** Concrete cases where one model slug supports a parameter on one provider and not on another. Each line is read off the tables above or the scratch tables.

| Model | Parameter or flag | Endpoints that list it | Endpoints that omit it |
|-|-|-|-|
| `meta-llama/llama-3.3-70b-instruct` | `tools`, `tool_choice` | `deepinfra/turbo`, `akashml/fp8`, `groq`, `coreweave/fp16`, `amazon-bedrock/us-east-1` | `parasail/fp8`, `cloudflare/fp8` |
| `meta-llama/llama-3.3-70b-instruct` | `structured_outputs` | `deepinfra/turbo`, `akashml/fp8`, `parasail/fp8`, `coreweave/fp16` | `cloudflare/fp8`, `groq`, `amazon-bedrock/us-east-1` |
| `deepseek/deepseek-v3.2` | `tools`, `tool_choice` | `siliconflow/fp8`, `deepinfra/fp4` | `baidu/fp8`, `digitalocean` |
| `deepseek/deepseek-v4-pro-0813` | `structured_outputs` | 16 of 21 endpoints | `deepseek` (the vendor's own endpoint), `baseten/fp4`, `relace/fp4`, `siliconflow/fp8`, `streamlake` |
| `moonshotai/kimi-k3` | `temperature`, `top_p` | 27 of 29 endpoints | `amazon-bedrock/us-east-1`, `amazon-bedrock/us-east-2` |
| `moonshotai/kimi-k3` | `tools` | 26 of 29 endpoints | `chutes/mxfp4`, `parasail/fast`, `river` |
| `moonshotai/kimi-k3` | `seed`, `logprobs`, `top_k` | Most third-party hosts | `moonshotai` (the vendor's own endpoint) among others |
| `z-ai/glm-5.3` | `tool_choice` | 45 of 46 endpoints | `amazon-bedrock/us-east-1` |
| `z-ai/glm-5.3` | `stop`, `seed`, `frequency_penalty` | Most third-party hosts | `z-ai/fp8` (the vendor's own endpoint) among others |
| `z-ai/glm-5.3` | `parallel_tool_calls` | `cohere/nvfp4`, `inceptron/fp4` | The other 44 |
| `openai/gpt-oss-120b` | `tools`, `tool_choice` | 21 of 24 endpoints | `digitalocean`, `google-vertex/global`, `siliconflow/fp8` |
| `qwen/qwen3.8-27b` | `structured_outputs` | 18 of 23 endpoints | `alibaba`, `cerebras/fp16`, `novita`, `siliconflow/fp8`, `streamlake` |
| `anthropic/claude-opus-5.5` | `structured_outputs` | `anthropic`, `anthropic/fast`, `claude-on-aws`, `azure/global`, `google-vertex/global` | `amazon-bedrock` |
| `anthropic/claude-sonnet-4.5` | `stop` | `anthropic`, `claude-on-aws`, `amazon-bedrock`, `google-vertex/global` | `azure/global` |
| `openai/gpt-6.2-sol` | `seed`, `response_format`, `structured_outputs` | `openai`, `openai/fast`, `openai/ultrafast`, `azure` | `amazon-bedrock/global` |
| `openai/gpt-5.2` | `max_tokens` | `openai`, `openai/flex`, `openai/fast` | `azure` (lists `max_completion_tokens`) |
| `google/gemini-4-carbon` | `stop` | `google-vertex/global`, `google-vertex/global/priority` | the three `google-ai-studio` endpoints |
| `x-ai/grok-4.7` | `logprobs`, `top_logprobs`, `seed` | the four `xai` endpoints | `amazon-bedrock/global` |

- Limits spread across hosts of one open-weight model: `z-ai/glm-5.3` context 262144 to 1048576 and output 32768 to 943718 over 46 endpoints; `qwen/qwen3.8-27b` context 65536 to 1024000 over 23; `openai/gpt-oss-120b` output 4096 to 117964 over 24; `moonshotai/kimi-k3` output 65535 to 1048575 over 29.
- Price spread: `deepseek/deepseek-v4.1-flash` input 0.0498 to 1.5 and output 0.1992 to 6 over 37 endpoints; `z-ai/glm-5.2` input 0.007 to 2.31 over 30.
- Quantization spread: `moonshotai/kimi-k2.6` is served at `fp4` (5 endpoints), `int4` (4), `fp8` (2), `bf16` (1) and undeclared (3).
- `supports_tool_choice` spread on `openai/gpt-oss-120b`: `NARF` on 10 endpoints, `.A..` on 9, `.ARF` on 4, `....` on 1.

### F. Variants, aliases, routers and naming

**M51.** The docs split suffixes into catalog variants, which are their own entries, and routing variants, which are never entries.

- Quote: "**Catalog variants** are separate entries in `GET /api/v1/models` with their own metadata. Only the models that list them support them."
- Quote: "**Routing variants** are accepted on any model ID at request time. They are not listed in `GET /api/v1/models` and change only how the request is routed. The model's metadata is the base model's."
- Quote: "`GET /api/v1/models` is a catalog of models and catalog variants. It is not an exhaustive list of every model string a request can use."
- Catalog suffixes and status: `:free` Active; `:batch` Active ("The Batch API takes the base model slug and resolves the `:batch` entry itself."); `:thinking` Deprecated ("Use the `reasoning` parameter instead."); `:extended` Deprecated ("No model currently offers it.").
- Routing suffixes, quoted from the Models API page: "`:nitro`: Prioritizes the lowest latency providers.", "`:floor`: Selects the cheapest available provider for the model.", "`:exacto`: Sorts providers by quality signals tuned for tool-calling reliability (mapped to `ProviderSort.Exacto`).", "`:online` (deprecated): Attaches web search results (use the `openrouter:web_search` server tool instead)." The variants overview page words `:nitro` as "Sorts providers by throughput" and adds that it admits priority-tier endpoints and that `:floor` admits flex-tier endpoints.
- One model ID may have at most one catalog variant and any number of routing variants (the overview page, paraphrased). Quote: "When more than one sorting variant is present (`:nitro`, `:floor`, `:exacto`), the last one in the ID determines the sort." and "Only the suffixes listed on this page are variants. Do not rely on any other suffix."
- Quote: "Because `:nitro` and `:floor` can select a service tier endpoint, the price actually charged for a request can differ from the base entry's `pricing`."
- Quote (Thinking page): "Reasoning is now requested per call through the `reasoning` parameter".
- Quote (Free page): "Free variants provide access to models without cost, but may have different rate limits or availability compared to paid versions."
- Source: M-s11, M-s3, M-s12, M-s13, M-s14. Basis: OR-documented.

**M52.** In the snapshot, two suffixes exist as entries.

- `:batch` 92 entries, `:free` 11 entries. `:thinking`, `:extended`, `:exacto`, `:nitro`, `:floor`, `:online`: 0 entries each.
- A variant's `name` has the suffix in parentheses: `anthropic/claude-fable-5.2:batch` is "Anthropic: Claude Fable 5.2 (batch)" and `google/gemma-4-31b-it:free` is "Google: Gemma 4 31B (free)".
- 19 entries have an `id` that starts with `~`, and 9 entries are routers or composites (M56).
- Source: M-s1. Basis: OR-documented.

**M53.** The docs give a three-step rule for resolving a requested model ID to a catalog entry, and two routes apply it on the server.

- Quote: "1. Split the ID on `:`. The first segment is the base slug. 2. Keep the catalog variant if one is present (`:free`, `:batch`, `:thinking`, `:extended`). Drop every routing variant (`:nitro`, `:floor`, `:exacto`, `:online`). 3. The result is the `id` of the catalog entry to read metadata from."
- Quote: "Send the model ID exactly as the user wrote it in the request `model` field, and resolve metadata separately."
- Quote (single-model route description): "Supports variant suffixes (e.g. openai/gpt-4:free) and resolves known slug aliases."
- Quote: "Do not strip every suffix. Stripping `:free` resolves to the paid entry, which can report a larger context window and non-zero pricing, neither of which describes the free entry that serves the request."
- Quote: "The single-model and endpoint lookup routes accept any suffix and apply this rule server-side, so a client can pass the requested ID through unchanged".
- Quote: "the single-model route returns `404` and the endpoints route returns `200` with `\"endpoints\": []`. Treat either as an unavailable model, not as a signal to retry with the base slug."
- Live, 2026-11-22, `GET /api/v1/model/<slug>` with no credential:

| Requested slug | Status | Returned `id` or error message |
|-|-|-|
| `openai/gpt-5.2` | 200 | `openai/gpt-5.2` (`canonical_slug` `openai/gpt-5.2-20251211`) |
| `openai/gpt-5.2:nitro`, `:floor`, `:exacto`, `:online` | 200 each | `openai/gpt-5.2` |
| `openai/gpt-5.2:bogus` | 200 | `openai/gpt-5.2` |
| `openai/gpt-5.2:batch` | 200 | `openai/gpt-5.2:batch` |
| `openai/gpt-5.2:free` | 404 | `Model not found: openai/gpt-5.2:free` |
| `openai/gpt-5.2:thinking`, `openai/gpt-5.2:extended`, `anthropic/claude-fable-5.2:thinking` | 404 each | `Model not found: ...` |
| `anthropic/claude-sonnet-4-5` (dash alias) | 200 | `anthropic/claude-sonnet-4.5` |
| `anthropic/claude-fable-5.2-20261109` (the canonical slug) | 404 | `Model not found: anthropic/claude-fable-5.2-20261109` |
| `google/gemma-4-31b-it:free`, `google/gemma-4-31b-it:free:nitro` | 200 each | `google/gemma-4-31b-it:free` |
| `~anthropic/claude-fable-latest`, `~anthropic/claude-opus-latest` | 200 each | The alias row itself, with `alias_target` |
| `openrouter/auto`, `openrouter/free` | 200 each | The router row |
| `does-not/exist` | 404 | `Model not found: does-not/exist` |

- Live, endpoints route: `openai/gpt-5.2:nitro` returned the same 4 records as `openai/gpt-5.2`. `openai/gpt-5.2:free` returned 200 with `"endpoints": []`. The canonical slug `anthropic/claude-fable-5.2-20261109` returned 200 with `"id": "anthropic/claude-fable-5.2"` and 4 records. `anthropic/claude-sonnet-4-5` returned `"id": "anthropic/claude-sonnet-4.5"` and 5 records.
- Source: M-s11, M-s2, M-s6, M-s25, M-s26. Basis: OR-documented.

**M54.** `canonical_slug` cannot be derived from `id` by a rule, and the two lookup routes treat it differently.

- Computed over the 457 entries that are not `~` aliases, comparing `canonical_slug` with the `id` minus its variant suffix: 307 are the `id` plus a dash suffix (a date such as `-20261109`, or a version such as `-2603` or `-05-2026`); 117 are identical; 33 have a different stem.
- Different-stem examples: `anthropic/claude-opus-4.8` and `anthropic/claude-4.8-opus-20260528`; `anthropic/claude-fable-5` and `anthropic/claude-5-fable-20260609`; `deepseek/deepseek-v4-pro-0813` and `deepseek/deepseek-v4-pro-20260813`; `mistralai/mistral-medium-3-5` and `mistralai/mistral-medium-3.5-20260430`; `cognitivecomputations/dolphin-mistral-24b-venice-edition` and `venice/uncensored`.
- A base entry and its `:batch` or `:free` twin share one `canonical_slug` (89 of 89 batch twins, 7 of 7 free twins).
- The single-model route returned 404 for a canonical slug and the endpoints route returned 200 for the same slug (M53). `links.details` on every entry uses the canonical slug (M22).
- Source: M-s1, M-s25, M-s26. Basis: OR-documented.

**M55.** `~author/family-latest` aliases are catalog rows that point at a concrete model, and they relax validation of reasoning parameters.

- Quote: "`~author/family-latest` slugs always resolve to the newest concrete model in a given family, so you can ship code against a stable alias and pick up new releases without redeploying."
- Quote: "The response's `model` field reflects the concrete model that actually served the request, not the alias you sent."
- Quote, about alias rows in `/api/v1/models` responses: they "report the pricing, context length, modalities, and supported parameters of the target they currently resolve to, not a frozen snapshot."
- Quote: "`~latest` slugs trade exactness for continuity. Your existing request shape keeps working when the alias retargets, even if the new model doesn't support every parameter you send. Instead of rejecting the request, OpenRouter remaps unsupported reasoning parameters to the nearest supported value."
- Quote: "`reasoning: { effort: \"none\" }` is upgraded to the lowest effort the model supports (for example `minimal` or `low`)." and "`reasoning: { enabled: false }` is flipped to enabled, again at the lowest supported effort." and "This remapping only applies to `~latest` slugs. Concrete model slugs keep strict validation."
- Quote: "**Aliases and hidden models are excluded**: The router never resolves to another alias slug or to models that have been hidden."
- Spec, `alias_target`: "Concrete model targeted by this tilde-latest alias, when applicable", with `slug` ("Routable model ID of the concrete target, matching that model row's id") and `name`. The changelog adds it on July 29, 2026.
- Snapshot, 19 alias rows and their `alias_target.slug`:

| Alias `id` | Target |
|-|-|
| `~anthropic/claude-fable-latest` | `anthropic/claude-fable-5.2` |
| `~anthropic/claude-opus-latest` | `anthropic/claude-opus-5.5` |
| `~anthropic/claude-sonnet-latest` | `anthropic/claude-sonnet-5.5` |
| `~anthropic/claude-haiku-latest` | `anthropic/claude-haiku-5.5` |
| `~openai/gpt-astra-latest` | `openai/gpt-6.2-astra` |
| `~openai/gpt-sol-latest` | `openai/gpt-6.2-sol` |
| `~openai/gpt-terra-latest` | `openai/gpt-6.1-terra` |
| `~openai/gpt-luna-latest` | `openai/gpt-6-luna` |
| `~openai/gpt-mini-latest` | `openai/gpt-5.4-mini` |
| `~google/gemini-pro-latest` | `google/gemini-4-carbon` |
| `~google/gemini-flash-latest` | `google/gemini-4-hydrogen` |
| `~x-ai/grok-latest` | `x-ai/grok-4.7` |
| `~deepseek/deepseek-pro-latest` | `deepseek/deepseek-v4-pro-0813` |
| `~deepseek/deepseek-flash-latest` | `deepseek/deepseek-v4.1-flash-1017` |
| `~deepseek/deepseek-v4-flash-latest` | `deepseek/deepseek-v4-flash-0731` |
| `~moonshotai/kimi-latest` | `moonshotai/kimi-k3` |
| `~z-ai/glm-latest` | `z-ai/glm-5.3` |
| `~z-ai/glm-flash-latest` | `z-ai/glm-5.3-flash` |
| `~unbiased/pareto-latest` | `unbiased/pareto-26.10` |

- On an alias row, `canonical_slug` equals the alias `id`, `architecture.tokenizer` is `Router`, and `created` is the alias's own date. `supported_parameters`, `context_length`, `reasoning`, `default_parameters` and `endpoints` equal the target row's on 19 of 19. `pricing` differs from the target row on 5: the three DeepSeek aliases, `~x-ai/grok-latest` and `~moonshotai/kimi-latest` (for example `~deepseek/deepseek-pro-latest` shows `"prompt": "0.0000000999"` and its target shows `"prompt": "0.00000132"`). `top_provider.max_completion_tokens` differs on the three DeepSeek aliases, and `top_provider.is_moderated` differs on 7 aliases.
- Live: the endpoints route for `~anthropic/claude-fable-latest` and `~anthropic/claude-opus-latest` returned 200 with `"endpoints": []`. A `supported_parameters` filter on the list route leaves alias and router rows out (M5).
- Source: M-s15, M-s4, M-s23, M-s1, M-s25, M-s26. Basis: OR-documented.

**M56.** Router and composite entries sit in the same list with sentinel values.

- Quote: "A model router picks which model handles each request. You send a router slug, such as `openrouter/auto`, as the `model`, and the router chooses a model from the catalog for that request. The response's `model` field tells you which model served it."
- Quote: "Routers don't add a routing fee. You pay the standard price of the model that serves the request, and the context length is that model's."
- Snapshot: 9 entries: `openrouter/auto`, `openrouter/auto-code`, `openrouter/auto-beta`, `openrouter/fusion`, `openrouter/pareto-code`, `openrouter/free`, `openrouter/bodybuilder`, `nvidia/switchyard`, `typesafe/jev-router`. `pricing.prompt` is `"-1"` on 8 and `"0"` on `openrouter/free`. `architecture.tokenizer` is `Router` on all 9. The `reasoning` object is omitted on all 9. `supported_parameters` is empty on 3 (`fusion`, `bodybuilder`, `switchyard`) and holds 20 to 24 names on the other 6. `context_length` is 2000000 on `auto`, `auto-code`, `auto-beta` and `pareto-code`. The endpoints route for `openrouter/auto` returned `"endpoints": []`.
- Source: M-s20, M-s3, M-s1, M-s26. Basis: OR-documented.

### G. Flagship fact sheet

**M57.** The newest three or four entries of nine vendors, read from the snapshot. Names are the catalog's own.

All 28 entries list the same four routes in `endpoints`: `/api/v1/chat/completions`, `/api/v1/completions`, `/api/v1/messages`, `/api/v1/responses`. `default_parameters` shown as "six keys, all null" means the object lists `temperature`, `top_p`, `top_k`, `frequency_penalty`, `presence_penalty`, `repetition_penalty` with `null` values. The last column comes from the endpoints route (M43); "not fetched" means the slug was outside the 59.

| Vendor | Model `id` (created, `canonical_slug` suffix) | `context_length` / `top_provider.max_completion_tokens` | `reasoning` object | `default_parameters` | `supported_parameters` (in spec enum order) | Pricing keys; input / output $ per M | Endpoints fetched: count, providers, `supports_tool_choice` flags (none, auto, required, function) |
|-|-|-|-|-|-|-|-|
| Anthropic | `anthropic/claude-fable-5.2` (2026-11-09, `claude-fable-5.2-20261109`) | 1000000 / 128000 | `{"mandatory":true,"supported_efforts":["max","xhigh","high","medium","low"],"default_effort":"high"}` | `{}` | max_tokens, max_completion_tokens, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, input_cache_write_1h; 8 / 40 | 4; Anthropic x2, Google, Azure; NA.. x4 |
| Anthropic | `anthropic/claude-haiku-5.5` (2026-10-07, `claude-haiku-5.5-20261007`) | 1000000 / 128000 | `{"mandatory":false,"supported_efforts":["max","xhigh","high","medium","low","none"],"default_effort":"medium"}` | `{}` | max_tokens, max_completion_tokens, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, input_cache_write_1h, overrides; 0.1 / 0.5 | 5; Google, Claude Platform on AWS, Azure, Anthropic, Amazon Bedrock; NA.. x5 |
| Anthropic | `anthropic/claude-sonnet-5.5` (2026-09-28, `claude-sonnet-5.5-20260928`) | 1000000 / 128000 | `{"mandatory":true,"supported_efforts":["max","xhigh","high","medium","low"],"default_effort":"high"}` | `{}` | max_tokens, max_completion_tokens, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, input_cache_write_1h; 2 / 10 | 5; Google, Amazon Bedrock, Azure, Claude Platform on AWS, Anthropic; NA.. x5 |
| Anthropic | `anthropic/claude-opus-5.5` (2026-09-22, `claude-opus-5.5-20260921`) | 1000000 / 128000 | `{"mandatory":true,"supported_efforts":["max","xhigh","high","medium","low"],"default_effort":"high"}` | `{}` | max_tokens, max_completion_tokens, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, input_cache_write_1h; 4 / 20 | 6; Anthropic x2, Amazon Bedrock, Azure, Google, Claude Platform on AWS; NA.. x6 |
| OpenAI | `openai/gpt-6.2-astra` (2026-11-17, `gpt-6.2-astra-20261117`) | 1050000 / 128000 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["max","xhigh","high","medium","low"],"default_effort":"medium"}` | `{}` | max_tokens, max_completion_tokens, seed, response_format, structured_outputs, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, overrides; 8 / 40 | 5; OpenAI x3, Azure, Amazon Bedrock; NARF x5 |
| OpenAI | `openai/gpt-6.2-sol` (2026-11-17, `gpt-6.2-sol-20261117`) | 1050000 / 128000 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["max","xhigh","high","medium","low"],"default_effort":"medium"}` | `{}` | max_tokens, max_completion_tokens, seed, response_format, structured_outputs, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, overrides; 2 / 10 | 5; OpenAI x3, Azure, Amazon Bedrock; NARF x5 |
| OpenAI | `openai/gpt-6.1-terra` (2026-10-26, `gpt-6.1-terra-20261026`) | 1050000 / 128000 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["max","xhigh","high","medium","low","none"],"default_effort":"medium"}` | `{}` | max_tokens, max_completion_tokens, seed, response_format, structured_outputs, tools, tool_choice, include_reasoning, reasoning, reasoning_effort, verbosity | web_search, input_cache_read, input_cache_write, overrides; 1 / 5 | 4; OpenAI x3, Azure; NARF x4 |
| Google | `google/gemini-4-neon` (2026-11-18, `gemini-4-neon-20261118`) | 1048576 / 943718 | `{"mandatory":true,"supported_efforts":["xhigh","high","medium","low"],"default_effort":"medium"}` | `{}` | temperature, top_p, max_tokens, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | image, audio, input_audio_cache, web_search, internal_reasoning, input_cache_read, input_cache_write; 3 / 15 | 5; Google AI Studio x3, Google x2; NARF x5 |
| Google | `google/gemini-4-hydrogen` (2026-11-12, `gemini-4-hydrogen-20261112`) | 1048576 / 943718 | `{"mandatory":true,"supported_efforts":["xhigh","high","medium","low"],"default_effort":"medium"}` | `six keys, all null` | temperature, top_p, max_tokens, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | image, audio, input_audio_cache, web_search, internal_reasoning, input_cache_read, input_cache_write; 1.5 / 7.5 | 5; Google AI Studio x3, Google x2; NARF x5 |
| Google | `google/gemini-4-carbon` (2026-11-10, `gemini-4-carbon-20261110`) | 1048576 / 943718 | `{"mandatory":true,"supported_efforts":["high","medium","low"],"default_effort":"medium"}` | `six keys, all null` | temperature, top_p, max_tokens, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | image, audio, input_audio_cache, web_search, internal_reasoning, input_cache_read, input_cache_write; 2 / 10 | 5; Google AI Studio x3, Google x2; NARF x5 |
| xAI | `x-ai/grok-4.7` (2026-09-21, `grok-4.7-20260916`) | 500000 / 450000 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["xhigh","high","medium","low"],"default_effort":"high"}` | `{"temperature":0.7,"top_p":0.95}` | temperature, top_p, top_k, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | web_search, input_cache_read, overrides; 2 / 6 | 5; xAI x4, Amazon Bedrock; NARF x5 |
| xAI | `x-ai/grok-4.6` (2026-08-12, `grok-4.6-20260810`) | 500000 / 450000 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["xhigh","high","medium","low"],"default_effort":"high"}` | `{}` | temperature, top_p, top_k, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | web_search, input_cache_read, overrides; 2 / 6 | not fetched |
| xAI | `x-ai/grok-4.5` (2026-07-08, `grok-4.5-20260708`) | 500000 / 450000 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["high","medium","low"],"default_effort":"high"}` | `{}` | temperature, top_p, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | web_search, input_cache_read, overrides; 2 / 6 | not fetched |
| DeepSeek | `deepseek/deepseek-v4.1-flash-1017` (2026-10-17, `deepseek-v4.1-flash-20261017`) | 1048576 / 393216 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["max","high","low","none"],"default_effort":"high"}` | `{}` | temperature, top_p, top_k, min_p, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, max_completion_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, parallel_tool_calls, include_reasoning, reasoning, reasoning_effort | input_cache_read, overrides; 0.15 / 0.6 | not fetched |
| DeepSeek | `deepseek/deepseek-v4.1-flash` (2026-09-10, `deepseek-v4.1-flash-20260910`) | 1048576 / 393216 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["max","high","low","none"],"default_effort":"high"}` | `{}` | temperature, top_p, top_k, min_p, top_a, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, parallel_tool_calls, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.0498 / 0.1992 | 37; DeepInfra x3, Fireworks x3, Wafer x2, InferenceNet x2, BaseTen x2, Parasail x2 ...; NARF x30, NA.F x2, .A.. x2, .... x1, NA.. x1, .ARF x1 |
| DeepSeek | `deepseek/deepseek-v4-pro-0813` (2026-08-12, `deepseek-v4-pro-20260813`) | 1048576 / 943718 | `{"mandatory":false,"supported_efforts":["max","high","low","none"],"default_effort":"high"}` | `{"temperature":1,"top_p":1}` | temperature, top_p, top_k, min_p, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 1.32 / 3.96 | 21; DeepInfra x3, Wafer x2, Alibaba x2, Parasail x2, Relace, Baidu ...; NARF x12, NA.. x5, .ARF x3, .A.. x1 |
| Mistral | `mistralai/mistral-large-4-0` (2026-10-06, `mistral-large-4-0-20261006`) | 1048576 / 262144 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["high","none"],"default_effort":"high"}` | `{}` | temperature, top_p, frequency_penalty, presence_penalty, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.7 / 2.1 | 2; Mistral x2; NARF x2 |
| Mistral | `mistralai/mistral-medium-3-5` (2026-04-30, `mistral-medium-3.5-20260430`) | 262144 / 209715 | `{"mandatory":false,"supported_efforts":["high","none"],"default_effort":"high"}` | `six keys, all null` | temperature, top_p, frequency_penalty, presence_penalty, max_tokens, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | prompt and completion only; 1.5 / 7.5 | 2; Mistral x2; NARF x2 |
| Mistral | `mistralai/mistral-small-2603` (2026-03-16, `mistral-small-2603`) | 262144 / 209715 | `{"mandatory":false,"default_enabled":false,"supported_efforts":["high","none"],"default_effort":"high"}` | `six keys, all null` | temperature, top_p, frequency_penalty, presence_penalty, max_tokens, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.15 / 0.6 | 3; Mistral x3; NARF x3 |
| Moonshot | `moonshotai/kimi-k3.1-pro` (2026-10-23, `kimi-k3.1-pro-20261023`) | 1048576 / 943718 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["max","high","low","none"],"default_effort":"max"}` | `{"top_p":0.95}` | temperature, top_p, top_k, min_p, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.2403 / 8.4023 | 28; InferenceNet x4, Wafer x2, Sail Research x2, Alibaba x2, Moonshot AI x2, Amazon Bedrock x2 ...; NARF x20, NAR. x4, .ARF x2, NA.. x1, NA.F x1 |
| Moonshot | `moonshotai/kimi-k3` (2026-07-16, `kimi-k3-20260715`) | 1048576 / 943718 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["max","high","low","none"],"default_effort":"max"}` | `{"temperature":null,"top_p":0.95,"top_k":null,"frequency_penalty":null,"presence_penalty":null,"repetition_penalty":null}` | temperature, top_p, top_k, min_p, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.192 / 16 | 29; InferenceNet x4, Wafer x3, Fireworks x3, Amazon Bedrock x2, Parasail x2, Relace ...; NARF x22, N.RF x2, NAR. x2, NA.. x1, .ARF x1, .... x1 |
| Moonshot | `moonshotai/kimi-k2.7-code` (2026-06-12, `kimi-k2.7-code-20260612`) | 262144 / 235929 | `{"mandatory":true,"default_enabled":true}` | `six keys, all null` | temperature, top_p, top_k, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, parallel_tool_calls, include_reasoning, reasoning | input_cache_read; 0.6 / 3 | not fetched |
| Qwen | `qwen/qwen4-max` (2026-11-16, `qwen4-max-20261115`) | 1000000 / 131072 | `{"mandatory":false,"default_enabled":true,"supported_efforts":["xhigh","high","low","none"],"default_effort":"high"}` | `{}` | temperature, top_p, top_k, frequency_penalty, presence_penalty, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read, input_cache_write; 1.425 / 4.275 | 1; Alibaba; NARF x1 |
| Qwen | `qwen/qwen3.8-max-prime` (2026-09-23, `qwen3.8-max-prime-20260923`) | 1000000 / 131072 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["xhigh","high","medium","low","minimal"],"default_effort":"xhigh"}` | `{}` | temperature, top_p, top_k, frequency_penalty, presence_penalty, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 4 / 12 | 1; Alibaba; NA.. x1 |
| Qwen | `qwen/qwen3.8-omni-flash` (2026-09-21, `qwen3.8-omni-flash-20260918`) | 1000000 / 131072 | `{"mandatory":false,"default_enabled":true,"supports_max_tokens":true}` | `{}` | temperature, top_p, top_k, frequency_penalty, presence_penalty, max_tokens, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, include_reasoning, reasoning | input_cache_read; 0.15 / 0.47 | not fetched |
| Z.ai | `z-ai/glm-5.3-flashx-legal` (2026-11-02, `glm-5.3-flashx-legal-20261102`) | 1048576 / 131072 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["max","high","low"],"default_effort":"max"}` | `{"temperature":1,"top_p":0.95}` | temperature, top_p, top_k, max_tokens, response_format, structured_outputs, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.37 / 1.25 | not fetched |
| Z.ai | `z-ai/glm-5.3-prime` (2026-09-23, `glm-5.3-prime-20260921`) | 1000000 / 131072 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["max","high","low"],"default_effort":"max"}` | `{"temperature":1,"top_p":0.95}` | temperature, top_p, top_k, frequency_penalty, presence_penalty, max_tokens, logprobs, top_logprobs, seed, response_format, stop, tools, tool_choice, include_reasoning, reasoning, reasoning_effort | input_cache_read; 1.68 / 5.28 | 1; Alibaba; NA.. x1 |
| Z.ai | `z-ai/glm-5.3` (2026-08-18, `glm-5.3-20260816`) | 1048576 / 943718 | `{"mandatory":true,"default_enabled":true,"supported_efforts":["max","high","low"],"default_effort":"max"}` | `{"temperature":1,"top_p":0.95}` | temperature, top_p, top_k, min_p, frequency_penalty, presence_penalty, repetition_penalty, max_tokens, logit_bias, logprobs, top_logprobs, seed, response_format, structured_outputs, stop, tools, tool_choice, parallel_tool_calls, include_reasoning, reasoning, reasoning_effort | input_cache_read; 0.0699 / 1.4 | 46; BaseTen x4, Fireworks x3, Wafer x2, InferenceNet x2, DeepInfra x2, Sail Research x2 ...; NARF x27, NA.F x9, NA.. x8, .A.. x1, NAR. x1 |

- Modalities (`architecture.modality`): the four Claude entries, the three GPT entries and the three Grok entries are `text+image+file->text`; the three Gemini entries are `text+image+file+audio+video->text`; DeepSeek V4.1 Flash is `text+image->text` and V4 Pro 0813 is `text->text`; Kimi K3 and K3.1 Pro, Qwen4 Max and Qwen3.8 Max Prime are `text+image+video->text`; GLM 5.3 and GLM 5.3 Prime are `text->text`.
- `knowledge_cutoff` is `null` on all 28. `expiration_date` is set on one: `mistralai/mistral-medium-3-5`, `2027-01-31`.
- `mistralai/pixtral-large-2502` has the newest `created` value among Mistral entries (2026-11-20) and is a non-reasoning entry; the sheet lists the three newest Mistral entries that have a `reasoning` object.
- Source: M-s1, M-s26. Basis: OR-documented.

### H. How Runvil already reads this catalog

All of section H is read from the repository `runvil-cloud-providers` at commit `50961b2cb837b51525651b42f62d269fd6f05f18` (2026-11-13, branch `master`, clean working tree for these files), crate path `crates/gateway-cloud-providers/src/`. Nothing was run. Basis: Workspace.

**M58.** Runvil's OpenRouter provider file reads ten catalog fields from the default list, with no credential.

- `providers/openrouter.rs` line 36: `const MODELS_PATH: &str = "/api/v1/models";`. Lines 45 to 51 issue `client.get(format!("{base_url}{MODELS_PATH}"))` with no query string and no auth header. The test `fetch_sends_no_credential` (lines 441 to 482) asserts that the request has no `authorization` header.
- Module comment, lines 1 to 9: "no auth on the listing endpoint (verified live 2026-09-14, a 718 KB payload) and the richest card of any surveyed provider" and "The default response is the full catalog, so the `links.next` pagination is not engaged."
- The wire structs (lines 60 to 96) declare exactly these fields: `id`, `name`, `created`, `context_length`, `architecture.input_modalities`, `architecture.output_modalities`, `pricing.prompt`, `pricing.completion`, `top_provider.max_completion_tokens`, `expiration_date`.

| Catalog field | Sheet field (`ModelEntry`) | Rule in `normalize_model` (lines 122 to 172) |
|-|-|-|
| `id` | `id`, and `family` through the taxonomy | Copied |
| `name` | `display_name` | Falls back to the `id` when null |
| `created` | `released_at` | Unix seconds to a calendar date (`openai_shape.rs` `base_entry`) |
| `context_length` | `context_window` | Copied |
| `top_provider.max_completion_tokens` | `max_output` | Copied |
| `architecture.input_modalities` | `images`, `pdf_input`, `video_input`, `audio_input` | True when the list holds `image`, `file`, `video`, `audio` |
| `architecture.output_modalities` | `kind` | First non-text output decides: `embeddings`, `image`, `video`, `audio` or `speech`, `transcription`, `rerank`; text-only is `Chat` (`openrouter-taxonomy.rs` lines 18 to 33) |
| `pricing.prompt`, `pricing.completion` | `pricing.prompt_per_mtok`, `pricing.completion_per_mtok`, `currency: "USD"` | String parsed to a float and multiplied by 1,000,000; an unparseable prompt price drops the pricing object |
| `expiration_date` | `deprecation` | `status: "expires"`, the parsed date, `replacement: None` |

- Source: M-s30, M-s31, M-s34.

**M59.** The reader skips the fields that describe capabilities, so every OpenRouter entry in the sheet shows the conservative defaults for them.

- Struct comment, lines 57 to 59: "`canonical_slug`, `description`, `per_request_limits`, `supported_parameters`, `default_parameters`, `supported_voices`, and `links` are ignored." Lines 72 to 74: "`modality`, `instruct_type`, and `tokenizer` duplicate the lists and are not parsed." Lines 81 to 83: "The cache, image, audio, and request price variants are ignored." Lines 90 to 92: "Its `context_length` duplicates the top-level field and `is_moderated` is ignored."
- Catalog fields that are outside both the wire structs and those comments: `reasoning`, `endpoints`, `alias_target`, `hugging_face_id`, `knowledge_cutoff`, `benchmarks`, `pricing.overrides`.
- `base_entry` (`providers/openai_shape.rs` lines 43 to 72) sets `batch: false`, `citations: false`, `code_execution: false`, `structured_outputs: false`, `tool_calling: false`, `thinking: Thinking::default()`, `effort_levels: Vec::new()`, `default_effort: None`, and `normalize_model` assigns none of them. The sheet type has these fields for exactly this data (`sheet-types.rs` lines 218 to 235: "Honors response schemas.", "Emits tool calls.", "The provider's own level names, e.g. `[\"low\", \"high\", \"max\"]`; never mapped to a cross-provider scale.").
- Where the catalog has the matching data: `supported_parameters` holds `tools` (305 interactive entries) and `structured_outputs` (300); the `reasoning` object holds `supported_efforts` and `default_effort` (146 interactive entries with a list), `mandatory` and `default_enabled`; a `:batch` entry and its `endpoints` value `/api/v1/batches` mark batch support.
- Reading `price_per_mtok` (lines 101 to 106) against M18: a router entry's `"-1"` parses as a float, so its sheet price would be -1,000,000 per million tokens. This is an inference from the code and the snapshot; it was not run. Confidence: high, the function has one path.
- The reader requests the default list. On 2026-11-22 that list holds text-output entries only (M4), so the entries it yields match only the `Chat`, `Image`, `Video` and `Speech` branches of `model_kind`.
- Source: M-s30, M-s33, M-s34, M-s1.

**M60.** Runvil's taxonomy groups by vendor prefix and folds a suffixed entry onto its base when the base is in the list.

- `providers/openrouter-taxonomy.rs` lines 12 to 16: the family is "the vendor segment before the slash (`openai`, `anthropic`, `z-ai`, ...), and the whole id otherwise." Lines 35 to 42: `apply` sets the family and then calls `collapse_variants(entries, sku_suffix)`.
- `taxonomy.rs` lines 117 to 123: `sku_suffix` "Splits a `:free`/`:batch`-style SKU suffix off `id`, on the last colon". Lines 125 to 151: `collapse_variants` marks an entry as `variant_of` the base "when the base id exists in the same list", and "An entry whose base id is absent stays canonical".
- Applied by reading to the 2026-11-22 snapshot (not run): 89 `:batch` and 7 `:free` entries become variants of a base entry; the 3 batch and 4 free entries listed in M42 stay canonical. The 19 `~` alias ids get families such as `~anthropic`, because `vendor_prefix` keeps the text before the first slash, and the alias's `alias_target` is unread, so the sheet does not link an alias to its target.
- Source: M-s31, M-s32, M-s1.

**M61.** The Runvil plan that introduced the file recorded the richer fields and scoped the file to five of them.

- `runvil`, `vibe/2026-09/2026-09-14-2-provider-model-sheets.md` (2026-09-14), line 101: "Verified live 2026-09-14: 718 KB payload; each model has `id`, `canonical_slug`, `name`, `created`, `description`, `context_length`, `architecture` (modality, input/output modalities, tokenizer), `pricing` (prompt/completion USD per token, cache read), `top_provider` (`max_completion_tokens`, `is_moderated`), `supported_parameters`, plus server-side filtering and pagination. A viable complement or fallback source."
- Same document, line 310: "only Anthropic and the OpenRouter aggregate expose effort levels in the list response". Line 309: "`effort_levels` is a list of the provider's own level names as strings - the observed union across all surveyed providers is exactly `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, `max` - and no cross-provider ordinal scale is ever invented." That union is the same seven words as OpenRouter's `ReasoningEffort` enum (M15).
- `runvil`, `vibe/2026-09/2026-09-14-4-provider-expansion-local-run.md` (2026-09-14), line 85: "normalize `context_length`, `architecture` input/output modalities, `pricing` (USD per token to per-million-token), `top_provider.max_completion_tokens`, and `expiration_date` into `Deprecation`. Tier: `aggregator`."
- `runvil-design`, `decisions/gateway.md` (last changed 2026-11-10), section 8: "The sheet records that a model has a setting and copies the provider's own values for it unchanged." Section on `GET /v1/models`: "Each routed entry adds `kind`, `description`, `context`, `thinking`, and the capability keys from the config, such as `max_output`, `images`, `effort_levels`, and `provider`."
- Source: M-s35, M-s36, M-s37.

### I. Fills: how three harnesses with a unified model layer read the same catalog

These show what other projects do with OpenRouter's catalog. They are not evidence of what OpenRouter does.

**M62.** pi reads `supported_parameters`, the `reasoning` object and `top_provider`, chooses the wire protocol from the `id` prefix, and patches catalog metadata per model id.

- Repository `badlogic/pi-mono` at commit `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (2026-11-22).
- `packages/ai/catalog/sources/openrouter.ts` lines 141 to 147 fetch three lists: the default, `?output_modalities=image` and `?output_modalities=decisions`. Comment, lines 48 to 52: "The default listing omits image-only and decision models, so those come from the other listings."
- Line 63: `if (!model.supported_parameters?.includes("tools")) continue;`
- Lines 71 to 78: `const useAnthropicMessages = /^~?anthropic\//.test(model.id) && !model.id.endsWith(":batch");`, then `api: useAnthropicMessages ? "anthropic-messages" : "openai-completions"` and `baseUrl: useAnthropicMessages ? "https://openrouter.ai/api" : "https://openrouter.ai/api/v1"`. The comment reads "Includes the `~anthropic/claude-*-latest` aliases, which OpenRouter's Messages endpoint accepts."
- Line 80: `reasoning: model.supported_parameters?.includes("reasoning") || false,`. Lines 15 to 26 turn `reasoning.supported_efforts` into pi's thinking levels, ending with `return { ...map, off: reasoning.mandatory === true ? null : "none" };`.
- Lines 84 to 85: `contextWindow: model.top_provider?.context_length || model.context_length || 4096,` and `maxTokens: model.top_provider?.max_completion_tokens || 4096,`.
- `packages/ai/catalog/providers/openrouter.ts` lines 13 to 33, function `correctUpstream`, comment "Keep selected model metadata stable until upstream settles.": it overrides `maxTokens` for the Kimi K3 family ("Keep Kimi K3's canonical output limit when gateway metadata is missing or incorrect."), overrides cost and `maxTokens` for `moonshotai/kimi-k2.5`, overrides cost for `z-ai/glm-5`, and sets `requiresReasoningContentOnAssistantMessages: true` for ids that start with `moonshotai/kimi-k2.6`.
- Same file, lines 72 to 78: every model gets `thinkingFormat: "openrouter"`, and ids matching `/^~?anthropic\//` also get `cacheControlFormat: "anthropic"`. Lines 8 to 11 hold a per-model exception with this comment: "OpenRouter rejects `configuration_update` system messages on Opus 5 (\"Mid-conversation reasoning effort (configuration_update) is not supported on anthropic/claude-opus-5-20260723\") while accepting them on Fable 5.1."
- Same file, lines 88 to 94, for ids starting `inception/mercury-2`: "Mercury 2 in instant mode (reasoning_effort: \"none\") disables tool calling." and "Mark \"off\" unsupported so the openai-completions provider omits the reasoning param".
- Source: M-s39. Basis: Filled (pi).

**M63.** Zed lists models from the key-scoped route and reads the whole `reasoning` object.

- Repository `zed-industries/zed` at commit `d31d1074a07b7bbaa43a52fef7847a00dd491baf` (2026-11-22), file `crates/open_router/src/open_router.rs`.
- Line 486: `let uri = format!("{api_url}/models/user");`, sent with `.header("Authorization", format!("Bearer {}", api_key))`.
- Line 534: `supports_tools: Some(entry.supported_parameters.contains(&"tools".to_string())),`. Lines 542 to 549 set `ModelMode::Adaptive` when `supported_parameters` contains `"reasoning"`.
- Lines 411 to 423 declare `ModelReasoning` with `default_effort`, `default_enabled`, `mandatory`, `supported_efforts`, `supports_max_tokens`. Lines 550 to 568 copy `supported_efforts` (reversed), `default_effort`, `supports_max_tokens` and `mandatory` into Zed's model.
- Line 533: `max_tokens: entry.context_length.unwrap_or(2000000),`.
- Source: M-s40. Basis: Filled (Zed).

**M64.** Roo Code reads the models list and the per-model endpoints route, and keeps hard-coded lists of model ids for what the catalog did not tell it.

- Repository `RooCodeInc/Roo-Code` at commit `b867ec9145750d0ae1ff7f02d35406e9bf2a0b16` (2026-05-15), file `src/api/providers/fetchers/openrouter.ts`.
- Line 102 fetches `${baseURL}/models`; line 150 fetches `${baseURL}/models/${modelId}/endpoints`; line 166 keys each endpoint by `endpoint.tag ?? endpoint.provider_name`.
- Line 208: `const supportsPromptCache = typeof cacheReadsPrice !== "undefined" // some models support caching but don't charge a cacheWritesPrice, e.g. GPT-5`.
- Line 211: `maxTokens: maxTokens || Math.ceil(model.context_length * 0.2),`. Line 220: `supportsReasoningEffort: supportedParameters ? supportedParameters.includes("reasoning") : undefined,`.
- Lines 224 to 230 consult two id sets, `OPEN_ROUTER_REASONING_BUDGET_MODELS` and `OPEN_ROUTER_REQUIRED_REASONING_BUDGET_MODELS`. Lines 236 to 281 override values for named ids, for example `if (id === "anthropic/claude-haiku-4.5") { modelInfo.supportsReasoningBudget = true; modelInfo.supportsReasoningEffort = false }` (lines 268 to 271, reflowed onto one line).
- Source: M-s41. Basis: Filled (Roo Code).

## Conflicts and disagreements

### Between OpenRouter's own sources

1. **Auth on the discovery routes.** The docs table says "Optional" for ten routes (M1) and the live routes answer 200 with no credential (M2). The OpenAPI spec gives those operations the global `apiKey` security, and the changelog records `401` as a new response code for `GET /models` (October 28, 2026) and for the two endpoints routes (November 11, 2026) (M3). The docs table and the live behavior agree with each other. The spec's security block is the outlier on 2026-11-22.
2. **Forced tool choice on Claude Haiku 5.5.** The Claude 5.5 migration guide says "Forced tool use still works on Haiku 5.5 (`tool_choice: \"required\"`, `{\"type\": \"any\"}`, or a named tool)". The endpoints route reports `"required": false` and `"function": false` on all five Haiku 5.5 endpoints fetched (M24). The two sources disagree and no inference call was made to settle it.
3. **Alias pricing.** The Latest Resolution page says alias rows "report the pricing, context length, modalities, and supported parameters of the target". In the snapshot 5 of 19 alias rows show a `pricing` object that differs from the target row (M55). Context length and supported parameters do match on 19 of 19.
4. **`model_id` on an endpoint.** The spec describes it as "The unique identifier for the model (permaslug)". The live value is the routable `id` (`openai/gpt-5.2`), and the canonical slug appears inside `name` (M23).
5. **Empty `default_parameters`.** The guide says "(null if none)". The snapshot uses `{}` on 215 of 476 entries and `null` on 0 (M14).
6. **`created`.** The spec says "when the model was created" and the guide says "when the model was added to OpenRouter" (M10).
7. **`:nitro`.** The Models API page says "Prioritizes the lowest latency providers" and the variants overview says "Sorts providers by throughput" (M51).
8. **Empty `endpoints` list.** The variants overview tells a client to treat a 200 with `"endpoints": []` as "an unavailable model". The same response comes back for a `~latest` alias and for `openrouter/auto`, which are routable rows (M55, M56). The guidance is written for a missing catalog variant; applied to every slug it would mark aliases and routers as unavailable.
9. **Catalog size.** The Models API page's sample output shows 518 text models and 774 over all modalities, with ids such as `near-ai/qwen3.8-27b-cyber`. The unauthenticated lists on 2026-11-22 hold 476 and 739 and do not contain those ids (M4). The sample was produced with an `Authorization` header; whether that or the passage of time explains the gap is unverified.

### With the local Runvil documents

1. **LC1 holds.** The catalog returns 200 with no auth header, and so do the single-model, endpoints, count, providers, ZDR and embeddings routes (M2). The survey's sentence "OpenRouter's catalogue is unauthenticated, so a 200 says nothing about your key" (M-s38) still holds; the key-scoped answer is `GET /api/v1/models/user`, which returned 401 without a key (M7).
2. **LC3 holds as the documented routing default, and it is not a blanket rule.** The docs still say an endpoint that lacks a parameter "can still receive the request, but will ignore unknown parameters" (M13). The same docs record refusals: a 400 from OpenRouter when a caller disables reasoning on a mandatory-reasoning model (M15), the provider's 400 for forced `tool_choice` on Opus 5.5 and Sonnet 5.5 (M24), and "Haiku 5.5 rejects any `temperature` other than `1`" (M35). So "unsupported parameters are silently ignored" is right for a parameter the serving endpoint does not list, and wrong for a listed parameter with a value the model refuses, and for reasoning controls.
3. **Runvil's module comment on the default list.** `openrouter.rs` says "The default response is the full catalog". On 2026-11-22 the default response is the text-output list: 476 of 739 entries, with 0 entries whose output is `embeddings` (M4, M59). The file's test fixture presents an embeddings model as part of "the verified 2026-09-14 live payload". The docs read today say embedding models "are not in the default response". For today, the docs and the live list are right. Whether the default changed after 2026-09-14 could not be established: the changelog shows "description updated" for `GET /models` on September 1 and September 17, 2026 with no detail.
4. **Effort levels.** The 2026-09-14 plan says "only Anthropic and the OpenRouter aggregate expose effort levels in the list response". That is true of the catalog today (`reasoning.supported_efforts`, M15). The provider file written from that plan leaves `effort_levels` empty for every OpenRouter entry (M59). The plan and the code agree on the file's scope (M61); the catalog has more than the file reads.
5. **Payload size.** The plan and the code comment record "718 KB" on 2026-09-14. The 2026-11-22 snapshot is 925 KB.

## Open questions blocked by closed source

| Question | What was tried | Fill |
|-|-|-|
| What do the endpoint `status` values `-1`, `-2`, `-3`, `-5`, `-10` mean? | The spec schema `EndpointStatus` (an enum with no description), the official TypeScript SDK (member names `Minus1` to `Minus10` only), a search of the 559-page docs mirror for `EndpointStatus`, `deranked` and status text (0 hits outside the API reference) | No fill found. pi, Zed and Roo Code do not read `status` |
| How is `top_provider` chosen, and is the model-level `pricing` always that provider's? | The Models guide ("Pricing from the top provider for this model", "Configuration details for the primary provider"), the spec, the provider integration guide | No fill found; this is internal to OpenRouter. Observation: model `context_length` equals the largest endpoint value on 55 of 55 models, and `top_provider.context_length` differs from it on 29 entries (M12, M19) |
| When the serving endpoint does not list a parameter, does OpenRouter remove it before the upstream call, or forward it and let the provider ignore or refuse it? | Provider selection page ("will ignore unknown parameters"), Parameters page, the Claude 4.7 guide ("silently ignored") and the Claude 5.5 guide ("rejects") | Fill from pi and Roo Code: both keep per-model-id rules in the client. pi marks a thinking level unsupported "so the openai-completions provider omits the reasoning param" for one model family, and Roo Code keeps id sets for reasoning budgets (M62, M64). This shows the workaround other projects use, not what OpenRouter does |
| What does `supports_implicit_caching: false` require of a request, and why do GPT-6.1 and GPT-6.2 endpoints report `false` while GPT-5.4 endpoints report `true`? | The spec (`{"type": "boolean"}`), the endpoints data (M30). The prompt caching guide belongs to another topic and was not read here | Fill from Roo Code: it derives cache support from the price keys instead of the flag: `const supportsPromptCache = typeof cacheReadsPrice !== "undefined"` (M64) |
| How does a `~latest` alias pick its target, and why do 5 alias rows show prices that differ from the target? | Latest Resolution page ("The newest visible model in that family is selected"), the snapshot (M55) | No fill found. pi special-cases one alias by id (`~moonshotai/kimi-latest`, M62) |
| Does a request with a key see a different or larger public list? | Docs sample output versus the live unauthenticated list (M4). No key was used, by rule | Fill from Zed and pi: both use the key-scoped `GET /models/user` to decide what to offer (M63; pi's test `openrouter-user-models.test.ts` notes "/models/user lists text models only") |
| Which tests decide that an endpoint lists `tools`, `structured_outputs`, or each `tool_choice` value? | Provider integration guide ("the capability suite", "baseline test suite") | No fill found |
| Are `default_parameters` applied by OpenRouter or only reported? | Spec, Models guide, Parameters page ("OpenRouter omits it upstream rather than substituting a hardcoded value") | Fill: pi, Zed and Roo Code all leave the field unread in the files inspected (M62, M63, M64) |

## Unverified

- That `strict_tool_schemas` is the enum value the November 19, 2026 changelog entry added. It is the only enum value with zero uses; the changelog names no value.
- Whether the default `GET /api/v1/models` response included embedding models on 2026-09-14.
- Whether an authenticated call returns entries the unauthenticated call does not.
- The reason the four newest Claude models report `supports_tool_choice` as `NA..`. The Claude 5.5 guide gives a reason for Opus 5.5 ("On models with thinking always enabled, forcing a tool call makes the model skip its thinking"); Haiku 5.5 has `mandatory: false` and the same flags.
- The rule "model-level `supported_parameters` is the union of the endpoint lists" was checked on 55 models, not on all 476.
- The statement that Runvil's reader would publish a price of -1,000,000 for router entries comes from reading the code, not from running it.
- Vendor-native rules were not read from vendor documentation for this topic. Where the matrix states a vendor rule, it is OpenRouter's account of it. From memory and not checked today: Anthropic's own API refuses forced tool choice when extended thinking is on, and OpenAI's reasoning models refuse a non-default `temperature`.
- The `/api/v1/images/models`, `/api/v1/videos/models` and `/api/v1/audio/speech/models` responses were fetched for status only and not analyzed.
- The Roo Code clone's newest commit is dated 2026-05-15. It is the head of the default branch on the clone day; its OpenRouter handling may predate catalog fields added since (`endpoints`, `alias_target`, `supports_tool_choice`).

## Workspace documents used

- `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/openrouter.rs`, `providers/openrouter-taxonomy.rs`, `providers/openai_shape.rs`, `taxonomy.rs`, `sheet-types.rs` (commit `50961b2`, 2026-11-13). Took: which catalog fields the provider sheet reads, ignores and maps. Re-checked live: yes, every field against the 2026-11-22 snapshot. Correction: the default list is the text-output list, not the full catalog.
- `runvil`, `vibe/2026-09/2026-09-14-2-provider-model-sheets.md` (2026-09-14). Took: the field list verified on that day, the effort-level statement, the seven-word effort union. Re-checked live: yes. The fields are all still present; `alias_target` (2026-07-29), `supports_tool_choice` (2026-08-28), `native_tools` (2026-09-28), `service_tier` (2026-10-15) and `endpoints` (2026-10-30) are dated by the changelog.
- `runvil`, `vibe/2026-09/2026-09-14-4-provider-expansion-local-run.md` (2026-09-14). Took: the stated scope of `openrouter.rs`. Not a live claim; not re-checked.
- `runvil-design`, `decisions/gateway.md` (last changed 2026-11-10). Took: what the model sheet and `GET /v1/models` are for. A decision record; not re-checked.
- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took: claims LC1 and LC3 and the sentence about the unauthenticated catalogue. Re-checked live: yes. LC1 confirmed; LC3 confirmed as the documented default with documented exceptions.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|-|-|-|-|
| M-s1 | `https://openrouter.ai/api/v1/models` (snapshot `models.json`, 476 entries, 925,524 bytes) | 2026-11-22 | OpenRouter public API response |
| M-s2 | `https://openrouter.ai/docs/guides/overview/models` | 2026-11-22 | OpenRouter doc |
| M-s3 | `https://openrouter.ai/docs/api_reference/models` | 2026-11-22 | OpenRouter doc |
| M-s4 | `https://openrouter.ai/docs/api/api-reference/models/list-all-models-and-their-properties` | 2026-11-22 | OpenRouter doc (embeds the spec for `GET /models`) |
| M-s5 | `https://openrouter.ai/docs/api/api-reference/endpoints/list-all-endpoints-for-a-model` | 2026-11-22 | OpenRouter doc (embeds the spec for the endpoints route) |
| M-s6 | `https://openrouter.ai/docs/api/api-reference/models/get-a-model-by-its-slug` | 2026-11-22 | OpenRouter doc |
| M-s7 | `https://openrouter.ai/docs/api/api-reference/models/get-total-count-of-available-models` | 2026-11-22 | OpenRouter doc |
| M-s8 | `https://openrouter.ai/docs/api/api-reference/models/list-models-filtered-by-user-provider-preferences-privacy-settings-and-guardrails` | 2026-11-22 | OpenRouter doc |
| M-s9 | `https://openrouter.ai/docs/api/api-reference/endpoints/preview-the-impact-of-zdr-on-the-available-endpoints` | 2026-11-22 | OpenRouter doc |
| M-s10 | `https://openrouter.ai/docs/api/api-reference/providers/list-all-providers` | 2026-11-22 | OpenRouter doc |
| M-s11 | `https://openrouter.ai/docs/guides/routing/model-variants/overview` | 2026-11-22 | OpenRouter doc |
| M-s12 | `https://openrouter.ai/docs/guides/routing/model-variants/free` | 2026-11-22 | OpenRouter doc |
| M-s13 | `https://openrouter.ai/docs/guides/routing/model-variants/thinking` | 2026-11-22 | OpenRouter doc |
| M-s14 | `https://openrouter.ai/docs/guides/routing/model-variants/extended` | 2026-11-22 | OpenRouter doc |
| M-s15 | `https://openrouter.ai/docs/guides/features/latest-resolution` | 2026-11-22 | OpenRouter doc |
| M-s16 | `https://openrouter.ai/docs/guides/providers/integration` | 2026-11-22 | OpenRouter doc |
| M-s17 | `https://openrouter.ai/docs/api_reference/parameters` | 2026-11-22 | OpenRouter doc |
| M-s18 | `https://openrouter.ai/docs/guides/routing/provider-selection` | 2026-11-22 | OpenRouter doc |
| M-s19 | `https://openrouter.ai/docs/guides/best-practices/reasoning-tokens` | 2026-11-22 | OpenRouter doc |
| M-s20 | `https://openrouter.ai/docs/guides/routing/routers/overview` | 2026-11-22 | OpenRouter doc |
| M-s21 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5` | 2026-11-22 | OpenRouter doc |
| M-s22 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-4-7` | 2026-11-22 | OpenRouter doc |
| M-s23 | `https://openrouter.ai/docs/changelog` (entries July 3, 2026 to November 21, 2026) | 2026-11-22 | OpenRouter doc (changelog) |
| M-s24 | `https://openrouter.ai/openapi.json` (123 paths, 1071 schemas) | 2026-11-22 | OpenRouter spec |
| M-s25 | Live unauthenticated probes of `https://openrouter.ai/api/v1/models` (with and without query parameters), `/models/count`, `/model/{author}/{slug}` for 21 slugs, `/models/user`, `/tools`, `/embeddings/models`, `/images/models`, `/videos/models`, `/audio/speech/models`; raw bodies and headers in scratch `M\probes`, `M\probes2`, `M\single` | 2026-11-22 | OpenRouter public API response |
| M-s26 | `https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints` for 59 slugs; raw bodies and `_fetch_log.tsv` in scratch `M\endpoints` | 2026-11-22 | OpenRouter public API response |
| M-s27 | `https://openrouter.ai/api/v1/endpoints/zdr` (936 records) | 2026-11-22 | OpenRouter public API response |
| M-s28 | `https://openrouter.ai/api/v1/providers` (123 records) | 2026-11-22 | OpenRouter public API response |
| M-s29 | `https://github.com/OpenRouterTeam/typescript-sdk` at `7ad8458755a042a1ccf1fc96046dee3dd9055849` (2026-11-21), `src/models/endpointstatus.ts` | 2026-11-22 | OpenRouter SDK source |
| M-s30 | `runvil-cloud-providers` at `50961b2cb837b51525651b42f62d269fd6f05f18` (2026-11-13), `crates/gateway-cloud-providers/src/providers/openrouter.rs` | 2026-11-22 | workspace document |
| M-s31 | Same repository and commit, `crates/gateway-cloud-providers/src/providers/openrouter-taxonomy.rs` | 2026-11-22 | workspace document |
| M-s32 | Same repository and commit, `crates/gateway-cloud-providers/src/taxonomy.rs` | 2026-11-22 | workspace document |
| M-s33 | Same repository and commit, `crates/gateway-cloud-providers/src/sheet-types.rs` | 2026-11-22 | workspace document |
| M-s34 | Same repository and commit, `crates/gateway-cloud-providers/src/providers/openai_shape.rs` | 2026-11-22 | workspace document |
| M-s35 | `runvil`, `vibe/2026-09/2026-09-14-2-provider-model-sheets.md` (document dated 2026-09-14) | 2026-11-22 | workspace document |
| M-s36 | `runvil`, `vibe/2026-09/2026-09-14-4-provider-expansion-local-run.md` (document dated 2026-09-14) | 2026-11-22 | workspace document |
| M-s37 | `runvil-design`, `decisions/gateway.md` (last changed 2026-11-10) | 2026-11-22 | workspace document |
| M-s38 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28) | 2026-11-22 | workspace document |
| M-s39 | `https://github.com/badlogic/pi-mono` at `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (2026-11-22), `packages/ai/catalog/sources/openrouter.ts`, `packages/ai/catalog/providers/openrouter.ts`, `packages/ai/test/openrouter-user-models.test.ts` | 2026-11-22 | harness source |
| M-s40 | `https://github.com/zed-industries/zed` at `d31d1074a07b7bbaa43a52fef7847a00dd491baf` (2026-11-22), `crates/open_router/src/open_router.rs` | 2026-11-22 | harness source |
| M-s41 | `https://github.com/RooCodeInc/Roo-Code` at `b867ec9145750d0ae1ff7f02d35406e9bf2a0b16` (2026-05-15), `src/api/providers/fetchers/openrouter.ts` | 2026-11-22 | harness source |
| M-s42 | `https://openrouter.ai/docs/guides/features/structured-outputs` | 2026-11-22 | OpenRouter doc |
