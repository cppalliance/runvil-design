---
produced: 2026-11-22
title: OpenRouter verification pass - independent live re-fetch of 27 first-wave claims about OpenRouter docs, OpenAPI spec, and public catalog (reasoning_details replay, error_type, supports_tool_choice, cache marker translation, sticky routing, encrypted blob suffix, deferred tools, Gemini continuation)
---

# OpenRouter verification pass: independent live re-fetch of 27 first-wave claims about OpenRouter's docs, spec, and public catalog

- What was read: 51 OpenRouter documentation pages in Markdown form, the full docs dump `llms-full.txt` (388 pages), the OpenAPI spec, the model catalog, the endpoints listing for 25 models (198 endpoint records), and five more discovery routes. That is 84 live sources. Every source was fetched fresh from `openrouter.ai` with `curl.exe -sL` and no key, header, or cookie. The local mirror under `pages\` was not used as a source.
- Fetch times: docs, spec, and catalog at 16:41 UTC on 2026-11-22 (one page, the PDF guide, at 17:02 UTC); endpoints listings and discovery routes at 16:47 UTC on 2026-11-22. Raw files and the fetch log (time, HTTP status, bytes, SHA-256 prefix, URL) are in the verification scratch folder.
- Drift check: the live spec is byte-identical to the one the first wave used (3,118,352 bytes, SHA-256 prefix `e6614516bb3f4059`). All 51 doc pages are byte-identical to their copies in the first-wave mirror. The live catalog is 925,527 bytes (the first wave's copy was 925,524), still 476 models. So any difference below is a reading difference, not a change at OpenRouter between the two fetches.
- Quoting: quotes are word for word from the live source. Where the live text has an em dash I write a plain dash, and Markdown link targets are left out. Those are the only changes made inside quotation marks. Nothing here was run against a live model, so every verdict is about what OpenRouter documents and publishes, not about observed wire behavior.

Verdict words: `confirmed`, `confirmed with a correction`, `not confirmed`, `conflict` (two OpenRouter sources disagree), `could not re-fetch`.

Counts over the 27 numbered claims: confirmed 21, confirmed with a correction 4 (claims 5, 16, 21, 22), conflict 2 (claim 10, and the Haiku 5.5 part of claim 17), not confirmed 0, could not re-fetch 0.

## Verdict table

Doc paths are relative to `https://openrouter.ai/docs/` and were fetched with `.md` appended. "Spec" is `https://openrouter.ai/openapi.json`. "Catalog" is `https://openrouter.ai/api/v1/models`. "Endpoints" is `https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints`.

| Claim | What the first wave recorded | Live source and fetch time | Verdict | Correction or note |
| - | - | - | - | - |
| 1. R51, R52, R53 | On chat completions reasoning goes back as the `reasoning` string or as the whole `reasoning_details` array; the array must match the original sequence, unmodified and in order; `reasoning_content` is an input alias | `guides/best-practices/reasoning-tokens`, 16:41 UTC | confirmed | The rule is worded for "consecutive reasoning blocks" and sits in the tool-calling part of the page. See V1 |
| 2. R32, R33, R34 | Four entry types; twelve `format` values | Spec (`ReasoningDetailUnion`, `ReasoningFormat`) and the reasoning guide, 16:41 UTC | confirmed | Both enums listed in V2. The guide still says "three possible types" while the spec maps four |
| 3. R1, R2, R3 | `reasoning` request object has seven properties; seven effort values; `effort` with `max_tokens` is rejected with a documented string | Spec (`ChatRequest.reasoning`, `ReasoningEffort`), 16:41 UTC | confirmed | String: `Only one of "reasoning.effort" and "reasoning.max_tokens" can be specified` |
| 4. R14, R15 | Disabling reasoning on a mandatory model is OpenRouter's own 400; on a `~latest` alias it is coerced | `cookbook/evaluate-and-optimize/model-migrations/claude-5-5`, spec (`ModelReasoning`), catalog, 16:41 UTC | confirmed | The 400 and the alias coercion are written for Opus 5.5 and Sonnet 5.5. The catalog marks both aliases `mandatory: true` too, so the catalog flag alone does not tell a client which behavior it gets |
| 5. R62, AN10, AN12 | OpenRouter passes the prefix-mismatch 400 through and does not drop blocks; `drop_block` and `input_transformations` are on Messages (report also on Responses), not on chat completions | `claude-5-5`, `fable-5-1`, spec, 16:41 UTC | confirmed with a correction | The `drop_block` request control is on the Messages route only. The Responses `input_transformations` is a different report with a different shape (OpenAI's own, `id`, `type`, `action`, `reason`). See V5 |
| 6. R63, AN11 | Account-bound thinking is handled by keeping the request on a provider that can read the blocks, or a 400 | `claude-5-5`, 16:41 UTC | confirmed | Haiku 5.5: same provider only. Sonnet 5.5: Anthropic, Claude Platform on AWS, and Azure share an account group |
| 7. R42, P50, P51 | Deferred-tools state lives in returned encrypted reasoning; dropping it gives `deferred_tools_state_missing` and three sibling codes; fields of `deferred_tools` and `defer_loading` | `guides/features/server-tools/tool-search`, spec (`DeferredToolsControl`), 16:41 UTC | confirmed | Guide says `portable` is the only profile; the spec enum also lists `universal` |
| 8. P26 | On chat completions a server tool call is a `reasoning_details` entry of type `reasoning.server_tool_call` | Spec (`ReasoningDetailServerToolCall`, `ServerToolOutputName`), 16:41 UTC | confirmed | Spec only. The string `reasoning.server_tool_call` occurs 3 times in the spec and 0 times in the 388-page docs dump |
| 9. C19, C20, C21, C22 | `cache_control` and `prompt_cache_breakpoint` are converted by serving model; TTLs dropped both ways; `prompt_cache_options` "stays OpenAI-only" | `guides/best-practices/prompt-caching`, `gpt-5-6`, changelog, spec, 16:41 UTC | confirmed | The caching page still has one sentence saying the breakpoint "carries no `ttl`" and another saying it "can carry its own `ttl`" |
| 10. C25 against C29 | Sticky lifetime is 60 minutes on one page and a "10-minute best-effort pin" on another | `guides/best-practices/prompt-caching` and `guides/best-practices/latency-and-performance`, 16:41 UTC | conflict | Both quotes are live today. These are the only two lifetime figures for sticky routing in the whole docs dump. See V10 |
| 11. C33, C38 | Usage is always sent, the opt-in parameters are deprecated; the final streaming usage chunk has a non-empty `choices` array | `cookbook/administration/usage-accounting`, `api_reference/streaming`, spec, 16:41 UTC | confirmed | |
| 12. C11 | Gemini explicit caching uses the `cache_control` marker; OpenRouter manages cache objects, uses only the last breakpoint, TTL fixed at 5 minutes | `guides/best-practices/prompt-caching`, 16:41 UTC | confirmed | |
| 13. OA1, OA2, S26 | `/responses` returns 400 on non-null `previous_response_id` and on `store: true` | `api_reference/responses/overview`, `api_reference/responses/basic-usage`, spec, 16:41 UTC | confirmed | OA2 is the item about the spec listing one Responses path, which also holds (`POST /responses`, plus the preset form) |
| 14. O35, O36 | `error_type` has 27 values; location differs per route | Spec (`ApiErrorType`), `api_reference/errors-and-debugging`, 16:41 UTC | confirmed | All 27 values in V14. The spec also has it on batch result errors |
| 15. O42, O43, O44, O47 | Mid-stream error chunk shape; HTTP 200 with an `error` body on non-streaming; failover continues after the 200 until the first token | `api_reference/errors-and-debugging`, `api_reference/streaming`, `guides/providers/traffic-routing`, 16:41 UTC | confirmed | The choice-level `error` and the chunk's top-level `provider` are documented in the guide and are outside the spec's `ChatChoice` and `ChatStreamChunk` property lists |
| 16. O51, O67 | Cancellation stops billing on 24 named providers and not on 30; debug echo example for Claude | `api_reference/streaming`, `api_reference/errors-and-debugging`, 16:41 UTC | confirmed with a correction | The "Not Currently Supported" list has 24 names, not 30. The names the first wave quoted are right; only the count in its matrix row is off. O67 is exact |
| 17. T10, T8, T9, M24 | Forced `tool_choice` on newer Claude models returns the provider's 400; per-endpoint `supports_tool_choice` | `claude-5-5`, `fable-5-1`, spec (`ToolChoiceSupport`), endpoints for 25 models, 16:41 and 16:47 UTC | conflict (Haiku 5.5 only); the rest confirmed | Guide: "Forced tool use still works on Haiku 5.5". Catalog: all five Haiku 5.5 endpoints report `required: false`, `function: false`. Every flag pattern the first wave recorded for the models I re-fetched matches. See V17 |
| 18. T21, T35 | `strict` is stripped from Anthropic tools without the `structured-outputs-2025-11-13` beta header; response healing is non-streaming only | `guides/routing/provider-selection`, `guides/features/plugins/response-healing`, 16:41 UTC | confirmed | |
| 19. P10, P11, P13, P14, P2, P3, P4 | Web search `engine` field, `auto` using the native tool, settings ignored with native; thirteen server tools and per-route acceptance | `guides/features/server-tools`, `guides/features/server-tools/web-search`, six tool pages, spec, endpoints, 16:41 and 16:47 UTC | confirmed | The spec's chat and Responses tool unions list `openrouter:bash`, while the bash guide says those two routes return a 400. The first wave recorded that disagreement and it is still there |
| 20. GG41 to GG49 | Gemini 4 continuation token is a reasoning item with `id: "gemini-continuation"`; OpenRouter strips the trailing turn and pins the provider | `cookbook/evaluate-and-optimize/model-migrations/gemini-4`, 16:41 UTC | confirmed | GG41 to GG46 are Google's own rules and are outside this pass. On the Messages route the token is a `redacted_thinking` block with a `data` field only, so the `gemini-continuation` id applies to chat completions and Responses |
| 21. R41, OA15, GG22 | Encrypted blobs in doc examples end with a dot and base64 JSON naming the endpoint | Nine doc pages and `llms-full.txt`, 16:41 UTC | confirmed with a correction | I decoded 23 blobs on 9 pages and reach the same values. Counts differ from R41 (ten on the Responses web search page, not five) and from OA15 (five Azure-format examples on the provider selection page, not four). Of the 22 `endpoint_slug` examples, 21 are on the Responses route, 1 on Messages, 0 on chat completions. See V21 |
| 22. S41 to S47 | Unsupported parameters are ignored by default, with 22 documented rejections | 14 doc pages and the spec, 16:41 UTC | confirmed with a correction | All 22 quotes are live word for word. One row is mislabeled: the forcing-`tool_choice` 400 is documented for `openrouter:tool_search` with `defer_loading`, and it also covers forbidding calls. See V22 |
| 23. M12, M2 | Catalog `supported_parameters` equals the union of the endpoints' lists; which discovery routes answer without a key | Catalog and endpoints for 25 models, five discovery routes, 16:41 and 16:47 UTC | confirmed | Union rule held on 25 of 25 models. `/models`, `/models/count`, `/providers`, `/endpoints/zdr` answer 200; `/tools` and `/models/user` answer 401 |
| 24. OA41, OA43, OA45 | `phase` is Responses-route only; async tools are Responses-route only; `configuration_update` has a chat form on a content-less system message | `gpt-5-4`, `gpt-6`, reasoning guide, spec, 16:41 UTC | confirmed | |
| 25. S5, S7, S8 | "The Responses API is now GA" (July 25, 2026); Messages serves non-Anthropic models; Responses serves non-OpenAI models | Changelog, `gpt-6`, `gemini-4`, `guides/features/service-tiers`, reasoning guide, spec, 16:41 UTC | confirmed | |
| 26. AN82, AN40, AN85 | `x-anthropic-beta` goes through a per-provider allowlist; native web search and web fetch only on the Anthropic and "Claude Platform on AWS" endpoints | `guides/routing/provider-selection`, endpoints for eight Claude models, 16:41 and 16:47 UTC | confirmed | AN85 is the fast-mode item, not a web-tools item. It also holds. The web-tools statement is in AN40 (and AN95) |
| 27. O53, S19 | `native_finish_reason`, `provider`, and `message.annotations` are outside the spec's chat response schemas | Spec, 16:41 UTC | confirmed | Property lists in V27 |

## Per-claim sections

Each section gives the live quote or the live data, then says how it compares with the first-wave record. Evidence ids for this file use the prefix `V`.

### V1. Claim 1 - the replay rule for `reasoning_details`, and `reasoning` as a second lane (R51, R52, R53)

- Source: `https://openrouter.ai/docs/guides/best-practices/reasoning-tokens.md`, section "Preserving Reasoning", subsection "Chat Completions". Fetched 2026-11-22 16:41 UTC.
- Live quote: "To preserve reasoning context across multiple turns, you can pass it back to the API in one of two ways:" then "1. **`message.reasoning`** (string): Pass the plaintext reasoning as a string field on the assistant message" and "2. **`message.reasoning_details`** (array): Pass the full reasoning\_details block".
- Live quote: "Use `reasoning_details` when working with models that return special reasoning types (such as encrypted or summarized) - this preserves the full structure needed for those models."
- Live quote: "For models that only return raw reasoning strings, you can use the simpler `reasoning` field. You can also use `reasoning_content` as an alias - it functions identically to `reasoning`."
- Live quote (note titled "Important for Reasoning Models"): "When providing reasoning\_details blocks, the entire sequence of consecutive reasoning blocks must match the outputs generated by the model during the original request; you cannot rearrange or modify the sequence of these blocks."
- Live quote (Go example): `messages = append(messages, msg) // includes reasoning_details as-is`
- Live quote (same subsection): "The `reasoning_details` functionality works identically across all supported reasoning models. You can easily switch between OpenAI reasoning models (like `~openai/gpt-sol-latest`) and Anthropic reasoning models (like `~anthropic/claude-sonnet-latest`) without changing your code structure."
- The Responses subsection of the same page states the same rule for that route: "send the previous response's `output` items back in the next request's `input`, in the order they were returned, and append your `function_call_output` items after them. Keep each `reasoning` item's `content`, `summary`, `signature`, and `encrypted_content` unchanged".
- Comparison: every first-wave quote is on the live page. One quote (the cURL comment "Send the assistant turn back unmodified (including reasoning_details) plus the tool results, so Claude continues reasoning from where it left off.") is split over two shell comment lines on the page; the words are the same.
- Reading note: the page frames the rule around tool calling ("Preserving reasoning blocks is useful specifically for tool calling"), and the rule's own words are "the entire sequence of consecutive reasoning blocks". "On the assistant message" comes from the field names (`message.reasoning`, `message.reasoning_details`) and the examples, which append the returned message.
- Verdict: confirmed.

### V2. Claim 2 - four entry types and twelve `format` values (R32, R33, R34)

- Source: `https://openrouter.ai/openapi.json` (OpenAPI 3.1.0, 123 paths, 1071 schemas), fetched 2026-11-22 16:41 UTC, read with Python.
- `ReasoningDetailUnion.discriminator.mapping`, every key: `reasoning.encrypted`, `reasoning.server_tool_call`, `reasoning.summary`, `reasoning.text`. That is four. The `oneOf` list has the same four schemas: `ReasoningDetailSummary`, `ReasoningDetailEncrypted`, `ReasoningDetailText`, `ReasoningDetailServerToolCall`.
- `ReasoningFormat.enum`, every value in spec order: `unknown`, `openai-responses-v1`, `azure-openai-responses-v1`, `bedrock-openai-responses-v1`, `bedrock-xai-responses-v1`, `xai-responses-v1`, `meta-responses-v1`, `google-vertex-meta-responses-v1`, `google-vertex-xai-responses-v1`, `openrouter-stealth-v1`, `anthropic-claude-v1`, `google-gemini-v1`, plus `null`. That is twelve string values. The schema has `"x-speakeasy-unknown-values": "allow"` and `"example": "unknown"`.
- Fields per type, from the live spec:
  - `ReasoningDetailText`: `type` (required), `text`, `signature`, `id` (each string or null), `format`, `index` (integer).
  - `ReasoningDetailSummary`: `type`, `summary` (both required), `id`, `format`, `index`.
  - `ReasoningDetailEncrypted`: `type`, `data` (both required), `id`, `format`, `index`.
  - `ReasoningDetailServerToolCall`: `type`, `tool_name`, `arguments`, `result` (all required), `tool_call_id`, `id`, `format`, `index`.
- Live quote (guide, same URL as V1): "Each object in the array represents a specific piece of reasoning information and follows one of three possible types." The guide then lists twelve `format` values with the same names as the spec, including "`"google-vertex-meta-responses-v1"` - Meta on Google Vertex responses format version 1 (requests may still send the retired `"google-vertex-responses-v1"`, which maps to this value)".
- Changelog check (`https://openrouter.ai/docs/changelog.md`): "`ReasoningFormat`: enum property\_added" appears under July 27, August 20, November 3, November 10, and November 18, 2026, as R34 says.
- Comparison: same four types, same twelve values, same field lists.
- Verdict: confirmed. The guide's "three possible types" against the spec's four is a small disagreement between two OpenRouter sources that the first wave already recorded.

### V3. Claim 3 - the `reasoning` request object (R1, R2, R3)

- Source: live spec, `ChatRequest.properties.reasoning`.
- Properties, all seven: `context`, `effort`, `enabled`, `exclude`, `max_tokens`, `mode`, `summary`. Description: "Configuration options for reasoning models". Example: `{"effort": "medium", "summary": "concise"}`.
- `effort` enum (also `ReasoningEffort`): `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`, `null`.
- Live quote (`max_tokens` description): "Maximum number of tokens the model may spend on reasoning, for models that take a reasoning token budget instead of an effort level. Cannot be combined with `reasoning.effort` or the top-level `reasoning_effort`: a request that sets both is rejected with "Only one of "reasoning.effort" and "reasoning.max_tokens" can be specified"." (The inner quotation marks are part of the spec's string.)
- Live quote (guide): `"effort": "high", // Can be "max", "xhigh", "high", "medium", "low", "minimal" or "none" (OpenAI-style)` and "// One of the following (not both):".
- The Responses route's `ReasoningConfig` is `BaseReasoningConfig` (`context`, `effort`, `mode`, `summary`) plus `enabled` and `max_tokens`; `exclude` is on the chat request only. Same as R1.
- Verdict: confirmed.

### V4. Claim 4 - disabling reasoning on a mandatory-reasoning model (R14, R15)

- Source: `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5.md`, fetched 16:41 UTC.
- Live quote (Opus 5.5): "**Disabling reasoning fails at OpenRouter, not upstream.** The model is registered as mandatory-reasoning, so `reasoning: {"enabled": false}`, `reasoning: {"effort": "none"}`, and Messages API `thinking: {"type": "disabled"}` return a 400 (`Reasoning is mandatory for this endpoint and cannot be disabled.`) before the request is routed. The exception is the `~anthropic/claude-opus-latest` alias, where OpenRouter coerces a disable request to the lowest supported effort instead."
- Live quote (Sonnet 5.5): "**Reasoning can't be disabled on Chat Completions.** Sonnet 5 accepted `reasoning: {"enabled": false}`. On Sonnet 5.5, OpenRouter returns a 400 (`Reasoning is mandatory for this endpoint and cannot be disabled.`) for `enabled: false` or `effort: "none"`. The exception is the `~anthropic/claude-sonnet-latest` alias, where OpenRouter runs adaptive thinking instead of returning the 400 (`effort: "none"` becomes `low`)."
- Live quote (spec, `ModelReasoning.mandatory`): "When true, reasoning cannot be disabled and effort "none" is rejected." Guide: "**`mandatory`**: When `true`, hide disable controls and do not send `effort: "none"` - the model rejects it."
- Live catalog (476 models): 354 have a `reasoning` object, 151 have `mandatory: true`, 9 have `supports_max_tokens: true`. Same three numbers as R14.
- Extra fact from the live catalog: `~anthropic/claude-opus-latest` and `~anthropic/claude-sonnet-latest` both report `"mandatory": true`, the same as the pinned slugs. `anthropic/claude-haiku-5.5` reports `"mandatory": false` with `none` in `supported_efforts`.
- Reading note: the 400 string and the alias coercion are documented for these two Claude models. The spec sentence covers all mandatory models, and it does not say whether OpenRouter or the provider does the rejecting for the other 149.
- Verdict: confirmed.

### V5. Claim 5 - prefix-bound Claude thinking, `drop_block`, and `input_transformations` (R62, AN10, AN12)

- Sources: `claude-5-5.md` and `.../model-migrations/fable-5-1.md`, and the live spec. Fetched 16:41 UTC.
- Live quote (both pages, same words): "Whether it is enforced through OpenRouter depends on the provider that serves the request. Some providers already return this 400 for requests through OpenRouter, and the rest can start enforcing it, so do not rely on history edits succeeding. OpenRouter passes the upstream error through and does not drop thinking blocks for you; set `drop_block` yourself if you want that."
- Live quote (Claude 5.5 page): "Upstream, replaying a thinking block after editing any part of that prefix (injected or removed messages, in-place summarization, a changed system prompt, a changed tool list) returns a 400 `invalid_request_error` on enforced accounts (Anthropic API accounts created on or after August 31, 2026), or drops the affected blocks if you opt in to `drop_block`."
- Live example, first line of the code block: "// Messages API - drop mismatched thinking blocks instead of erroring", then `"thinking": { "type": "adaptive", "block_binding": { "prefix_mismatch_behavior": "drop_block" } }`.
- Live quote: "With `drop_block`, each removal is reported in the response's `input_transformations`, which makes it a good audit tool. Run a session with it set, log `input_transformations`, and fix any `prefix_binding_mismatch` your harness produces. (`model_binding_mismatch` entries after a model switch are expected.)"
- Live spec, the request control: `AnthropicThinkingBlockBinding` has `prefix_mismatch_behavior` (enum `error`, `drop_block`, `null`) and `mismatch_behavior` ("Deprecated: legacy alias of prefix_mismatch_behavior. Send only one of the two."). The only schema that references it is `MessagesRequest`, under `thinking` in the `enabled` and `adaptive` variants. The property name `block_binding` occurs nowhere else in the spec.
- Live spec, the report: a property named `input_transformations` is on four schemas. Three use `AnthropicInputTransformation` ("A server-side transformation Anthropic applied to the request input", example `{"path": "messages.1.content.0", "reason": "prefix_binding_mismatch", "type": "thinking_dropped"}`): `BaseMessagesResult`, `MessagesStartEvent`, and the Messages body variant inside `BatchObject` results. The fourth is `OpenResponsesResult`, which uses a different schema, `ResponsesInputTransformation` (required `id`, `type`, `action`, `reason`; example `{"action": "removed", "id": "rs_0ca73592c5f68037016afcc009779887", "reason": "prefix_mismatch", "type": "reasoning"}`), described as "Input items the upstream provider changed before inference (OpenAI `input_transformations`). Present only when the provider reported the field; empty when it changed nothing."
- Live spec, chat completions: `ChatResult` has eleven properties (`alignment`, `choices`, `created`, `id`, `model`, `object`, `openrouter_metadata`, `safeguard_results`, `service_tier`, `system_fingerprint`, `usage`). `ChatRequest` has 43 properties; its reasoning control is the seven-property `reasoning` object in V3, and `thinking` is a property of `MessagesRequest` only.
- Changelog: "`BaseMessagesResult`: new property `input_transformations`" is under September 1, 2026, and "`AnthropicThinkingBlockBinding`: new property `mismatch_behavior`" under September 9, 2026, as R62 and AN12 say.
- Verdict: confirmed with a correction. The correction is to R62's closing line, which calls the `drop_block` switch and the report "Messages and Responses features". The switch is a Messages request field only. The Responses route has a report of the same name, and it is OpenAI's own report in a different shape, present "only when the provider reported the field". The spec ties the Claude `thinking_dropped` report to Messages results (and Messages batch results). A fourth location, `BatchObject`, was not in the first-wave list.

### V6. Claim 6 - account-bound thinking on Sonnet 5.5 and Haiku 5.5 (R63, AN11)

- Source: `claude-5-5.md`, fetched 16:41 UTC.
- Live quote (Haiku): "Thinking blocks from Haiku 5.5 are only readable by the provider account that produced them. Replaying one through a different account is not an error upstream: Anthropic drops the block and the model loses that reasoning."
- Live quote (Haiku): "OpenRouter handles this in routing. When a request replays Haiku 5.5 thinking blocks, OpenRouter keeps it on the provider that produced them instead of falling back to a different provider. If you pass thinking blocks back unchanged, nothing changes for you. If `provider.only` or `provider.ignore` excludes the provider that produced the blocks, OpenRouter returns a 400 asking you to allow that provider or start a new conversation without the thinking blocks."
- Live quote (Sonnet): "Sonnet 5.5 thinking blocks are readable only by the account that produced them or an account linked to it. Anthropic, Claude Platform on AWS, and Azure share an account group, so OpenRouter can fall back among them when you replay Sonnet 5.5 thinking blocks. It does not send those replays to Amazon Bedrock or Google Vertex. If `provider.only` or `provider.ignore` leaves no provider that can read the blocks, OpenRouter returns a 400 asking you to allow one of them or start a new conversation without the thinking blocks."
- Comparison: word for word the same as R63 and AN11. How OpenRouter learns which account produced a block is left unstated on the page ("OpenRouter handles this in routing").
- Verdict: confirmed.

### V7. Claim 7 - the deferred-tools protocol (R42, P50, P51)

- Source: `https://openrouter.ai/docs/guides/features/server-tools/tool-search.md` and the live spec. Fetched 16:41 UTC.
- Live quote: "The `deferred_tools` request field opts a conversation into OpenRouter's versioned deferred-tool protocol. It works on the Chat Completions, Responses, and Messages APIs. Every field is required:" followed by `{"deferred_tools": {"protocol": "v1", "profile": "portable", "validation": "runtime", "search": { "type": "regex", "max_results": 5 }}}`.
- Live quote (Continuation State): "OpenRouter carries each turn's deferred-tool changes inside the encrypted reasoning it returns: a `reasoning_details` entry on Chat Completions, a reasoning item's `encrypted_content` on Responses, and a `redacted_thinking` block on Messages. Each turn records only what changed, such as the ID of a tool it loaded. There is no separate state field."
- Live quote: "**Send assistant messages back exactly as returned**, including the encrypted reasoning described above. Each turn's state is chained to the turns before it, so editing, removing, reordering, trimming, or compacting earlier messages breaks the chain."
- Live quote: "an assistant message that contains only text and carries no state is accepted as ordinary conversation content, and the next turn continues from the last complete one. An assistant message with tool calls but no state is rejected."
- Live error table, all four codes: `deferred_tools_state_missing` ("An assistant message with tool calls carries no deferred-tool state, usually because the client did not send the encrypted reasoning back, or carries state altered until it is no longer recognisable."), `deferred_tools_state_invalid` ("The deferred-tool state fails verification: it was modified, or was issued for a different workspace."), `deferred_tools_history_changed` ("Earlier messages, or the state attached to them, were edited, removed, or reordered."), `deferred_tools_catalog_changed` ("The tools, `tool_choice`, or `deferred_tools` settings differ from the ones the conversation started with."). Location: "in `error.metadata.code` on Chat Completions and Responses and in `metadata.code` on Messages". Status: "returns a `400`".
- Live quote (named clients): "**pi** needs pi-ai `0.84.3` or later on Chat Completions. Earlier versions drop the encrypted `reasoning_details` entry."
- `defer_loading`, live table: Chat Completions "`tools[].function.defer_loading: true`", Responses "`tools[].defer_loading: true` on a function tool", Messages "`tools[].defer_loading: true`".
- Live spec: `deferred_tools` is a `$ref` to `DeferredToolsControl` on `ChatRequest`, `ResponsesRequest`, and `MessagesRequest`. Required: `protocol`, `profile`, `validation`, `search`. Other properties: `announce_catalog_changes`, `cascade_mode` (`keywords`, `native`), `hint`, `interface` (`wrappers`, `code_mode`), `min_deferred_tools`, `scope` (`all`, `mcp`). `profile` enum: `portable`, `universal`. `ChatUsage.deferred_tools_details` refers to `DeferredToolsDetails` (`turns`, `prompt_tokens`, `completion_tokens`, `total_tokens`).
- Comparison: same as R42, P50, P51.
- Verdict: confirmed. The guide's "`"portable"` is the only profile in this release" against the spec's `portable`, `universal` enum is a disagreement between two OpenRouter sources that the first wave already recorded.

### V8. Claim 8 - server tool calls on chat completions (P26)

- Source: live spec.
- `ReasoningDetailServerToolCall.description`: "Record of an OpenRouter server-tool invocation (e.g. openrouter:fusion), carried in reasoning_details so a prior tool call can be rehydrated into a later turn of the same conversation." Example: `{"arguments": "{\"prompt\":\"Compare carbon tax proposals\"}", "result": "{\"status\":\"ok\",\"models\":[\"openai/gpt-4o\"]}", "tool_call_id": "call_abc123", "tool_name": "openrouter:fusion", "type": "reasoning.server_tool_call"}`.
- `ServerToolOutputName.type.description`: "Output item `type` on Responses, content block `type` on Anthropic Messages, `reasoning_details[].type` on Chat Completions". `ServerToolInputName.output_names.description`: "What the call surfaces as on each accepting API format when requested under this spelling; a format is absent when the call is not visible to the caller".
- Coverage check: the string `reasoning.server_tool_call` occurs 3 times in the live spec and 0 times in the live 388-page docs dump (`https://openrouter.ai/docs/llms-full.txt`). The route that would list which tools produce the entry, `GET /api/v1/tools`, answers 401 without a key (V23).
- Verdict: confirmed, on the spec alone. The only place such an entry is printed is the spec's own example, which matches P26's confidence note.

### V9. Claim 9 - cache marker translation (C19, C20, C21, C22)

- Sources: `https://openrouter.ai/docs/guides/best-practices/prompt-caching.md`, `.../model-migrations/gpt-5-6.md`, the changelog, the live spec. Fetched 16:41 UTC.
- Live quote (caching page): "The block-level markers are interchangeable: a text block marked with Anthropic-style `cache_control` gets a `prompt_cache_breakpoint` when routed to a supporting OpenAI model, and a block marked with `prompt_cache_breakpoint` gets a default (5-minute) `cache_control` when routed to Anthropic or Google. TTLs are not translated - a `cache_control` `ttl` is dropped toward OpenAI, a `prompt_cache_breakpoint` `ttl` is dropped toward Anthropic or Google, and the request-level `prompt_cache_options` stays OpenAI-only."
- Live quote (GPT-5.6 page): "You don't need to filter these fields per model. OpenRouter strips `reasoning.mode`, `prompt_cache_breakpoint`, and `prompt_cache_options` before forwarding requests to providers that don't support them, so requests that fall back to other models keep working." and "`cache_control` becomes a `prompt_cache_breakpoint` on GPT-5.6+ models, and `prompt_cache_breakpoint` becomes a default 5-minute `cache_control` on Anthropic and Google. The request-level `prompt_cache_options` stays OpenAI-only."
- Live spec: `ChatContentCacheControl`: "Anthropic-style cache breakpoint for the content part. Interchangeable with the OpenAI-style `prompt_cache_breakpoint` marker: OpenRouter converts between the two based on the provider serving the request." `PromptCacheBreakpoint`: "Supported natively by OpenAI GPT-5.6 and newer; on providers that use Anthropic-style `cache_control`, OpenRouter converts the marker to that format automatically. `ttl` ("30m" or "12h") overrides the request-level `prompt_cache_options.ttl` for this boundary". `PromptCacheOptions`: "Only supported by OpenAI GPT-5.6 and newer." `prompt_cache_options` is a property of `ChatRequest` and `ResponsesRequest`; `prompt_cache_breakpoint` is a property of `ChatContentText` and `InputText`.
- Changelog: "`PromptCacheBreakpoint`: description updated; example modified; new property `ttl`" sits under October 21, 2026.
- The two sentences C22 pointed at are both on the live caching page: "A `prompt_cache_breakpoint` can carry its own `ttl` to override the request-level value for that boundary" (explicit caching section) and "Note that `prompt_cache_breakpoint` carries no `ttl`; if you need to set a cache `ttl`, use the Chat Completions or Anthropic Messages API with `cache_control`." (the Responses note in the Anthropic section).
- Verdict: confirmed.

### V10. Claim 10 - sticky routing lifetime: 60 minutes against 10 minutes (C25 against C29)

- Page one: `https://openrouter.ai/docs/guides/best-practices/prompt-caching.md`, section "Provider Sticky Routing", fetched 16:41 UTC. Live quote: "Sticky sessions expire after **60 minutes** of inactivity. Each successful request resets the timer. If the sticky provider returns an error, the cache is not updated, allowing the next request to be re-routed."
- Same page, on `session_id`: "When a `session_id` is present, OpenRouter uses it directly as the sticky routing key instead of deriving one from message hashing." and "When `session_id` is set, sticky routing activates on any successful request - even before cache usage is observed - so that subsequent requests in the same session benefit from prompt caching from the start. Without `session_id`, sticky routing only activates after a cache hit is detected."
- Page two: `https://openrouter.ai/docs/guides/best-practices/latency-and-performance.md`, fetched 16:41 UTC, under "**Solution:** Preserve sticky routing and provider KV caching across turns." Live quote: "**Pass a Consistent `session_id`:** OpenRouter stores a 10-minute best-effort pin directing follow-up turns back to the exact provider endpoint holding the warm KV cache."
- Same page: "**Keep the Model Slug Identical:** Changing from `~anthropic/claude-sonnet-latest` to `~anthropic/claude-sonnet-latest:nitro` mid-conversation invalidates the sticky key." and "**Avoid Hardcoded `provider.order`:** Setting explicit `provider.order` disables sticky-session reordering and default routing."
- Coverage check: I searched the live 388-page docs dump for every minute figure next to "sticky", "pin", "session", "affinity", or "routing". These two sentences are the only lifetime figures for sticky routing.
- Both pages describe the same mechanism (a `session_id` keyed pin to one provider endpoint, lost on `provider.order`). One says 60 minutes of inactivity with a reset on each success, the other says a 10-minute best-effort pin.
- Verdict: conflict. The first wave recorded it accurately. Which figure is current is left open by both pages.

### V11. Claim 11 - usage is always sent, and the final streaming usage chunk (C33, C38)

- Sources: `https://openrouter.ai/docs/cookbook/administration/usage-accounting.md`, `https://openrouter.ai/docs/api_reference/streaming.md`, live spec. Fetched 16:41 UTC.
- Live quote (usage page): "OpenRouter automatically returns detailed usage information with every response, including:" and "No additional parameters are required."
- Live quote (usage page note): "On Chat Completions, the `usage: { include: true }` and `stream_options: { include_usage: true }` parameters are deprecated and have no effect. The Responses API has no equivalent parameter. Full usage details are always included automatically in every response on both APIs."
- Live spec: `ChatStreamOptions.include_usage` has `"deprecated": true` and the description "Deprecated: This field has no effect. Full usage details are always included." `ChatRequest` has 43 properties; `stream_options` is one of them and `usage` is not among them.
- Live quote (streaming page, "The final usage chunk (Chat Completions)"): "On the Chat Completions endpoint (`/api/v1/chat/completions`), every stream ends with an extra chunk that carries the `usage` object for the request, sent just before the `[DONE]` message. OpenAI's spec emits this chunk with an empty `choices` array, but many clients crash when accessing `choices[0].delta` on it, so OpenRouter intentionally deviates: the usage chunk contains one choice with a content-free `delta` that repeats the `finish_reason` (and `native_finish_reason`) of the stream."
- Live wire example, three lines in order: `data: {"id":"gen-abc123",...,"choices":[{"index":0,"delta":{"content":"","role":"assistant"},"finish_reason":"stop","native_finish_reason":"stop"}]}` then `data: {"id":"gen-abc123",...,"choices":[{"index":0,"delta":{"content":"","role":"assistant"},"finish_reason":"stop","native_finish_reason":"stop"}],"usage":{...}}` then `data: [DONE]`.
- Live quote: "This means the terminal `finish_reason` appears twice: once on the last content-bearing chunk and again on the usage chunk. Clients that validate streams should treat the usage chunk as an accounting frame rather than a second terminal event." and "This shape is specific to Chat Completions."
- Live spec: `ChatStreamChunk` requires `id`, `choices`, `created`, `model`, `object`; `usage` is optional.
- Verdict: confirmed.

### V12. Claim 12 - Gemini explicit caching (C11)

- Source: `prompt-caching.md`, section "Google Gemini", fetched 16:41 UTC.
- Live quote: "Gemini caching in OpenRouter requires you to insert `cache_control` breakpoints explicitly within message content, similar to Anthropic."
- Live quote: "OpenRouter simplifies Gemini cache management, abstracting away complexities:" then "You **do not** need to manually create, update, or delete caches." and "You **do not** need to manage cache names or TTL explicitly."
- Live quote: "There is not a limit on the number of `cache_control` breakpoints you can include in your request. OpenRouter will use only the last breakpoint for Gemini caching across normal message content. Including multiple breakpoints is safe and can help maintain compatibility with Anthropic, but only the final one will be used for Gemini."
- Live quote: "Cache Writes have a 5 minute Time-to-Live (TTL) that does not update. After 5 minutes, the cache expires and a new cache must be written."
- Live quote (price): "Cache write cost = Input token price + (Cache storage price × (5 minutes / 60 minutes))" and "**Cache Reads:** Charged at 0.25× the original input token cost."
- Live quote (system prompt limit): "Gemini has a single `systemInstruction` field, and cached Gemini content treats that `systemInstruction` as immutable. On OpenRouter, this means `cache_control` inside the first `system` or `developer` message can cache the normalized system prompt, but it cannot preserve an uncached dynamic tail inside that same message."
- Verdict: confirmed. The page gives the 5 minute figure as a fixed property of Gemini cache writes on OpenRouter. The "Cache TTL Options" section of the same page sits under the Anthropic Claude heading.

### V13. Claim 13 - `/responses` is stateless and says so with a 400 (OA1, OA2, S26)

- Sources: `https://openrouter.ai/docs/api_reference/responses/overview.md`, `.../responses/basic-usage.md`, live spec. Fetched 16:41 UTC.
- Live quote (overview): "This API is **stateless** - each request is independent and no conversation state is persisted between requests. You must include the full conversation history in each request. Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error."
- Live quote (basic usage): "Since the Responses API is stateless, you must include the full conversation history in each request to maintain context. Replayed assistant content uses `output_text`, while user content uses `input_text`. Stateful parameters like `store: true` and `previous_response_id` are not supported and are rejected with a `400` error".
- Live spec, `ResponsesRequest.previous_response_id.description`: "Not supported on this proxy. Each response request is independent: no responses are stored, so a previous response cannot be referenced. Requests with a non-null value are rejected with a 400 error. Send the full conversation history in `input` instead." `ResponsesRequest.store`: `{"const": false, "default": false, "type": "boolean"}`.
- Two more live pages repeat it: GPT-6 guide, "OpenRouter's Responses API is stateless and rejects `previous_response_id` with a 400, so you replay the transcript yourself."; Gemini 4 guide, "OpenRouter does not store the response, so `previous_response_id` cannot be used to continue."
- OA2 (one Responses path): the live spec's paths containing "responses" are `POST /responses` (summary "Create a response", operation `createResponses`, tags `Responses` and `beta.responses`, "Creates a streaming or non-streaming response using OpenResponses API format") and `POST /presets/{slug}/responses`.
- Verdict: confirmed.

### V14. Claim 14 - `error_type` values and where the field sits (O35, O36)

- Sources: live spec, `https://openrouter.ai/docs/api_reference/errors-and-debugging.md`. Fetched 16:41 UTC.
- `ApiErrorType` description: "Canonical OpenRouter error type, stable across all API formats". `type: string`, `"x-speakeasy-unknown-values": "allow"`, example `rate_limit_exceeded`.
- All 27 values, in spec order: `context_length_exceeded`, `max_tokens_exceeded`, `token_limit_exceeded`, `string_too_long`, `authentication`, `permission_denied`, `payment_required`, `rate_limit_exceeded`, `provider_overloaded`, `provider_unavailable`, `invalid_request`, `invalid_prompt`, `not_found`, `precondition_failed`, `payload_too_large`, `unprocessable`, `content_policy_violation`, `refusal`, `invalid_image`, `image_too_large`, `image_too_small`, `unsupported_image_format`, `image_not_found`, `image_download_failed`, `server`, `timeout`, `unmapped`.
- The guide's seven tables list the same 27 (4 token and length, 3 authentication, 3 rate and availability, 6 request validation, 2 content policy, 6 image, 3 generic), with the statuses O35 recorded.
- Live quote (location): "**Chat Completions**: `error.metadata.error_type` - on the mid-stream error chunk (see Mid-Stream Errors) and on the non-streaming response when a provider error interrupts generation." / "**Anthropic Messages**: `error.error_type` on the SSE `error` event and the non-streaming error envelope." / "**Responses**: top-level `error_type` on the failed response, for both the streaming `response.failed` event and the non-streaming JSON body."
- Live quote: "Native protocol codes (the Anthropic `error.type`, the Responses `error.code`) are best-effort and can differ between formats - `error_type` is the field to rely on across all of them."
- Live spec, every place that refers to `ApiErrorType`: `ChatStreamChunk.error.metadata.error_type`; `AnthropicMessagesErrorResponse.error.error_type`; `MessagesErrorEvent.error.error_type`; `OpenResponsesResult.error_type` (top level); `BatchObject.results[].error.error_type`.
- Small spec detail: for chat HTTP error bodies (`BadRequestResponse` and its siblings) `error.metadata` is an open object, so `error_type` is typed in the spec for the stream chunk only.
- Verdict: confirmed.

### V15. Claim 15 - errors after the 200 (O42, O43, O44, O47)

- Sources: `errors-and-debugging.md`, `streaming.md`, `https://openrouter.ai/docs/guides/providers/traffic-routing.md`, live spec. Fetched 16:41 UTC.
- Live quote: "OpenRouter sends you the HTTP `200 OK` status and headers as soon as the provider accepts the request. That happens before the model produces a single token, and an HTTP status is final once sent."
- Live quote: "Every failure after that point is therefore reported inside the response instead of in the status. A streaming request receives an SSE error event; a non-streaming request receives an error body. This includes provider errors that arrive before any token. Failover also stops once part of the answer has reached you, since your application already holds output from the first provider."
- Live quote (note): "If an attempt fails before any tokens reach you, OpenRouter automatically tries a backup provider. The `200 OK` has already been sent by then, so the status stays `200` even when every provider fails - the last error reaches you in the response body."
- Live type (`MidStreamError`, code comments left out): `id: string; object: 'chat.completion.chunk'; created: number; model: string; provider: string; error: { code: number; message: string; metadata?: { error_type: string; provider_code?: string; }; }; choices: [{ index: 0; delta: { content: '' }; finish_reason: 'error'; native_finish_reason?: string; }];`
- Live example: `data: {"id":"gen-abc123","object":"chat.completion.chunk","created":1234567890,"model":"openai/gpt-4o","provider":"OpenAI","error":{"code":429,"message":"Rate limit exceeded","metadata":{"error_type":"rate_limit_exceeded"}},"choices":[{"index":0,"delta":{"content":""},"finish_reason":"error"}]}`
- Live quote (characteristics): "The error appears at the **top level** alongside standard response fields", "A `choices` array is included with `finish_reason: "error"` to properly terminate the stream", "The HTTP status remains `200 OK`, because the headers were sent before the error occurred", "The stream is terminated after this event".
- Live quote (non-streaming, headers then failure): "If the provider returns headers and then fails, you receive a `200 OK` whose JSON body holds only an `error` object and no `choices`; its `id` identifies the generation when you report the failure. Check the body for an `error` field even on a `200`, rather than relying on the status alone."
- Live example (non-streaming, partial output): `{"choices": [{"message": { "role": "assistant", "content": "partial output..." }, "finish_reason": "error", "error": {"code": 502, "message": "Provider disconnected mid-stream", "metadata": { "error_type": "provider_unavailable" }}}]}`
- Live quote (streaming page): "Once the provider has returned response headers, the `200 OK` status is committed even if no token has been produced yet." and "The error can be the first and only event in the stream, so treat a `200` carrying an `error` chunk with no content as a failure, not a success".
- Live list (traffic routing page), what counts against a provider: "5xx responses", "Setup errors: 401, 402, 404, 405, 408, 410, 421, 423, 424, 426, 498", "Responses that fail partway through". Not counted: "Bad requests: 400, 413, 422", "Geographic or legal blocks: 403, 451", "User disconnects: 499".
- Spec cross-check: `ChatStreamChunk` declares a top-level `error`. `ChatChoice` declares `finish_reason`, `index`, `logprobs`, `message`, so the choice-level `error` in the guide's non-streaming example is a documented field outside the spec. The same goes for the top-level `provider` on the error chunk (V27).
- Verdict: confirmed.

### V16. Claim 16 - cancellation and billing, and the debug echo example (O51, O67)

- Source: `streaming.md`, section "Stream cancellation", fetched 16:41 UTC.
- Live quote: "Streaming requests can be cancelled by aborting the connection. For supported providers, this immediately stops model processing and billing."
- Live list, "**Supported**", eight lines of three: "OpenAI, Azure, Anthropic" / "Fireworks, Mancer, Recursal" / "AnyScale, Lepton, OctoAI" / "Novita, DeepInfra, Together" / "Cohere, Hyperbolic, Infermatic" / "Avian, XAI, Cloudflare" / "SFCompute, Nineteen, Liquid" / "Friendli, Chutes, DeepSeek". That is 24 names.
- Live list, "**Not Currently Supported**", eight lines of three: "AWS Bedrock, Groq, Modal" / "Google, Google AI Studio, Minimax" / "HuggingFace, Replicate, Perplexity" / "Mistral, AI21, Featherless" / "Lynn, Lambda, Reflection" / "SambaNova, Inflection, ZeroOneAI" / "AionLabs, Alibaba, Nebius" / "Kluster, Targon, InferenceNet". That is 24 names.
- Live quote: "Cancellation only works for streaming requests with supported providers. For non-streaming requests or unsupported providers, the model will continue processing and you will be billed for the complete response."
- Correction: the first wave's matrix row says "24 providers stop billing on abort, 30 do not". The second list has 24 names. The names quoted in O51 are complete and correct.
- Side note: the providers route lists 123 providers (V23) and these two lists name 48, so most providers are in neither list.
- O67 source: `errors-and-debugging.md`, section "Debugging". The live request: `{"model": "anthropic/claude-haiku-5.5", "stream": true, "messages": [{ "role": "system", "content": "You are a helpful assistant." }, { "role": "user", "content": "Hello!" }], "debug": { "echo_upstream_body": true }}` (sent to `https://openrouter.ai/api/v1/chat/completions`).
- The live echoed upstream body, in full:

```json
{
  "system": [
    {
      "type": "text",
      "text": "You are a helpful assistant."
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": "Hello!"
    }
  ],
  "model": "claude-haiku-5-5",
  "stream": true,
  "metadata": {
    "user_id": "<redacted>"
  },
  "max_tokens": 65536,
  "stop_sequences": []
}
```

- So the system message is lifted to a top-level `system` array of text blocks, `max_tokens` is filled with 65536, the slug becomes `claude-haiku-5-5`, and `metadata.user_id` is added and printed as `<redacted>`. The page prints `<redacted>`; that the value is a hash comes from another first-wave item (O69) and is not shown in this example.
- The same request on the Responses route (`"input": "Hello!"`) echoes the same body without `system`. The "Debug response format" block on the same page shows a shorter body (`system`, `messages`, `model`, `stream`, `max_tokens`) inside a chunk with `"provider": "Anthropic"`, `"model": "anthropic/claude-haiku-5.5"`, and `"choices": []`.
- Live quotes on limits: "The debug option **only works with streaming mode** (`stream: true`). Non-streaming requests will ignore the debug parameter." and "a debug chunk will be sent for **each attempted provider**".
- Verdict: confirmed with a correction (the count of 30 is 24).

### V17. Claim 17 - forced `tool_choice` on newer Claude models, and `supports_tool_choice` (T10, T8, T9, M24)

- Sources: `claude-5-5.md`, `fable-5-1.md` (16:41 UTC); live spec; the endpoints route for the models below (16:47 UTC, HTTP 200 without a key).
- Live quote (Opus 5.5): "Opus 5.5 rejects `tool_choice` set to `{"type": "any"}` or a named tool, and the Chat Completions forms `tool_choice: "required"` and `{"type": "function", "function": {"name": ...}}`, with the provider's 400 (`tool_choice: type "tool" and "any" are not supported for this model.`). `{"type": "auto"}` (the default) and `{"type": "none"}` are unaffected."
- Live quote (Sonnet 5.5): "Sonnet 5.5 rejects forced tool choice. Chat Completions `tool_choice: "required"` or a named function, and Messages API `{"type": "any"}` or `{"type": "tool"}`, return the provider's 400 (`tool_choice: type "tool" and "any" are not supported for this model.`). Send `auto`, say in the prompt when to call the tool, and mark the tool `strict: true` so its input matches the schema."
- Live table row (both models): "Forced `tool_choice` (`required`, named, `any`, `tool`) | Accepted | 400 upstream". Fable 5.1 page: "Starting with Fable 5.1, requests with `tool_choice` set to `{"type": "any"}` or a named tool return a 400." and the row "`tool_choice: {"type": "any"}` or named tool | Allowed | Rejected upstream (400)".
- Live quote (Haiku 5.5): "Forced tool use still works on Haiku 5.5 (`tool_choice: "required"`, `{"type": "any"}`, or a named tool), but the model skips thinking before a forced call."
- Live spec, `ToolChoiceSupport`: "Per-variant `tool_choice` support. `tool_choice` in `supported_parameters` only says the parameter is accepted; these flags say which of its values passed testing." Four required booleans: `none`, `auto`, `required`, `function`. `supports_tool_choice` is in `PublicEndpoint.required`. Changelog: "`PublicEndpoint`: required `supports_tool_choice` property added" under August 28, 2026.
- Live endpoint data. The newest Claude flagship in the catalog by `created` date is `anthropic/claude-fable-5.2` (2026-11-09). The Meta model served by the provider named "Meta" is `meta/muse-ember` (the newest `meta/` entry, 2026-11-09); I added `meta/muse-spark-1.3` and `meta-llama/llama-4-maverick` because they differ. The Mistral model is `mistralai/mistral-large-2512`, with `mistralai/mistral-large-4-0` added for the same reason.

| Model | Provider (`provider_name`) | Endpoint `tag` | `none` | `auto` | `required` | `function` |
| - | - | - | - | - | - | - |
| `anthropic/claude-sonnet-5.5` | Amazon Bedrock | `amazon-bedrock` | true | true | false | false |
| `anthropic/claude-sonnet-5.5` | Anthropic | `anthropic` | true | true | false | false |
| `anthropic/claude-sonnet-5.5` | Azure | `azure/global` | true | true | false | false |
| `anthropic/claude-sonnet-5.5` | Claude Platform on AWS | `claude-on-aws` | true | true | false | false |
| `anthropic/claude-sonnet-5.5` | Google | `google-vertex/global` | true | true | false | false |
| `anthropic/claude-haiku-5.5` | Amazon Bedrock | `amazon-bedrock` | true | true | false | false |
| `anthropic/claude-haiku-5.5` | Anthropic | `anthropic` | true | true | false | false |
| `anthropic/claude-haiku-5.5` | Azure | `azure/global` | true | true | false | false |
| `anthropic/claude-haiku-5.5` | Claude Platform on AWS | `claude-on-aws` | true | true | false | false |
| `anthropic/claude-haiku-5.5` | Google | `google-vertex/global` | true | true | false | false |
| `anthropic/claude-fable-5.2` | Anthropic | `anthropic` | true | true | false | false |
| `anthropic/claude-fable-5.2` | Anthropic | `anthropic/fast` | true | true | false | false |
| `anthropic/claude-fable-5.2` | Azure | `azure` | true | true | false | false |
| `anthropic/claude-fable-5.2` | Google | `google-vertex/global` | true | true | false | false |
| `meta/muse-ember` | Google | `google-vertex/global` | false | true | false | false |
| `meta/muse-ember` | Meta | `meta` | false | true | false | false |
| `meta/muse-spark-1.3` | Google | `google-vertex/global` | true | true | true | true |
| `meta/muse-spark-1.3` | Meta | `meta` | true | true | true | true |
| `meta-llama/llama-4-maverick` | Parasail | `parasail/fp8` | false | true | true | true |
| `mistralai/mistral-large-2512` | Mistral | `mistral/eu` | false | true | true | true |
| `mistralai/mistral-large-2512` | Mistral | `mistral/zdr` | false | true | true | true |
| `mistralai/mistral-large-4-0` | Mistral | `mistral/eu` | true | true | true | true |
| `mistralai/mistral-large-4-0` | Mistral | `mistral/zdr` | true | true | true | true |

- Other Claude models fetched the same way: `claude-opus-5.5` (6 endpoints) and `claude-fable-5.1` (3) are all `true, true, false, false`; `claude-opus-5` (6), `claude-opus-4.8` (6), and `claude-sonnet-4.5` (5) are all four `true`.
- Cross-check against T9: I re-fetched 15 of T9's 19 models (`claude-fable-5.2`, `claude-opus-5.5`, `claude-sonnet-5.5`, `claude-sonnet-4.5`, `gpt-5.5`, `gpt-6.2-sol`, `gemini-3.5-flash`, `grok-4.7`, `mistral-large-2512`, `llama-4-maverick`, `llama-3.3-70b-instruct`, `deepseek-v4.1-flash`, `kimi-k3`, `glm-5.3`, `qwen3.7-flash`). For all 15, every endpoint count and every flag pattern count equals the first-wave table, for example `deepseek/deepseek-v4.1-flash` 37 endpoints: 30 all true, 2 without `required`, 2 `auto` only, 1 `none` and `auto`, 1 without `none`, 1 all false.
- T9's confidence note also reproduces: eight endpoints across `meta-llama/llama-3.3-70b-instruct`, `moonshotai/kimi-k3`, and `z-ai/glm-5.3` report true flags while their own `supported_parameters` list leaves out `tool_choice` (five of them leave out `tools` too).
- Verdict: T10 confirmed for Opus 5.5, Sonnet 5.5, Fable 5.1 and later. T8, T9, M24 confirmed. Conflict on Haiku 5.5: the guide says forced tool use "still works", and the catalog reports `required: false` and `function: false` on all five Haiku 5.5 endpoints. The first wave recorded this disagreement in its model catalog file and its Anthropic file; it is still live. Which of the two is right is left open by both sources, and only a live request could settle it.

### V18. Claim 18 - `strict` on Anthropic tools, and response healing (T21, T35)

- Source: `https://openrouter.ai/docs/guides/routing/provider-selection.md`, section "Provider-Specific Headers" then "Anthropic Beta Features". Fetched 16:41 UTC.
- Live quote: "For **strict tool use** (`strict: true` on tools), you must explicitly pass the `structured-outputs-2025-11-13` header. Without this header, OpenRouter will strip the `strict` field and route normally."
- Live table row: "Structured Outputs | `structured-outputs-2025-11-13` | Enables the strict tool use feature for supported Claude models, validating tool parameters against your schema to ensure correctly-typed arguments". The header name is in the sentence above the table: "you can request specific beta features by including the `x-anthropic-beta` header in your request".
- Live quote (what OpenRouter manages itself): "**Structured outputs for JSON schema response format** (`text.format.type: "json_schema"`, or `response_format.type: "json_schema"` on Chat Completions) - the header is automatically applied" and "**Fine-grained tool streaming** - for every streamed request (`stream: true`) that includes user-defined tools, OpenRouter sets `eager_input_streaming: true` on each tool".
- Source for T35: `https://openrouter.ai/docs/guides/features/plugins/response-healing.md`. Live quote: "The plugin activates for non-streaming requests when you set `text.format` in a Responses API request (or `response_format` on Chat Completions) to either `type: "json_schema"` or `type: "json_object"`, and include the response-healing plugin in your `plugins` array." Live quote: "Response Healing only applies to non-streaming requests." Live quote: "Some malformed JSON responses may still be unrepairable. In particular, if the response is truncated by `max_output_tokens`, the plugin will not be able to repair it."
- Verdict: confirmed.

### V19. Claim 19 - web search engine selection, and the server tool list (P10, P11, P13, P14, P2, P3, P4)

- Sources: `https://openrouter.ai/docs/guides/features/server-tools.md`, `.../server-tools/web-search.md`, the bash, shell, apply patch, advisor, subagent, and fusion pages, live spec, endpoints route. Fetched 16:41 and 16:47 UTC.
- Live table (web search parameters): "`engine` | string | `auto` | Search engine to use: `auto`, `native`, `exa`, `firecrawl`, `parallel`, or `perplexity`".
- Live quote: "**`auto`** (default): Uses native search if the provider supports it, otherwise falls back to Exa" and "**`native`**: Prefers the provider's built-in web search; falls back to Exa if the model doesn't support native search."
- Live quote: "When `engine` is `"auto"` (the default) or `"native"`, OpenRouter uses the provider's built-in search for supported models." and "To check programmatically, read `native_tools` on the endpoint object."
- Live quote (overview page): "When the model's endpoint has a "native" version of a tool available, OpenRouter can request the tool to be executed through the provider instead of running its own engine. This is the `native` engine in the server tool configuration." and "Each endpoint object returned by the endpoints API has a `native_tools` field. The key is the canonical OpenRouter server tool name. For example, for web search, it is `openrouter:web_search`. The value is the name of the provider's tool."
- Live spec, `PublicEndpoint.native_tools`: "The server tools this endpoint accepts as the provider's own built-in tool (`engine: "native"`) instead of an OpenRouter engine, keyed by canonical `openrouter:*` name." It is in `PublicEndpoint.required`.
- Live endpoint data for `openrouter:web_search`: Anthropic and Claude Platform on AWS endpoints `web_search_20260209`; OpenAI and Azure endpoints of `openai/gpt-5.5` and `openai/gpt-6.2-sol` `web_search`; all Google endpoints of `google/gemini-3.5-flash` and `google/gemini-4-argon` `google_search`; xAI endpoints of `x-ai/grok-4.7` `web_search`; the Meta endpoint of `meta/muse-ember` `web_search`. The Amazon Bedrock endpoints of `gpt-6.2-sol` and `grok-4.7` have an empty `native_tools`.
- Settings with the native engine, live table cells: `max_results` "Applies to Exa, Firecrawl, Parallel, and Perplexity engines; ignored with native provider search"; `max_uses` "With native provider search, forwarded only to Anthropic (as `max_uses`); other native search providers ignore it"; `search_context_size` "Ignored with native provider search and Firecrawl."; `max_characters` "ignored with native provider search and Firecrawl"; `user_location` "Currently only supported by native provider search; ignored with Exa, Firecrawl, Parallel, and Perplexity".
- Domain filter rows, live: "**Native (Anthropic)** | Yes | Yes | Mutually exclusive", "**Native (OpenAI)** | Yes | No | `excluded_domains` ignored (no error)", "**Native (Google)** | No | No | Not supported. With `engine: "auto"`, falls back to Exa when filters are set. With `engine: "native"`, returns a 400 error", "**Native (SpaceXSI)** | Yes | Yes | Mutually exclusive".
- P2, the live "Available Server Tools" table has thirteen rows: `openrouter:web_search`, `openrouter:datetime`, `openrouter:image_generation`, `openrouter:web_fetch`, `openrouter:apply_patch` ("Responses API only"), `openrouter:shell` ("Responses and Messages APIs"), `openrouter:bash` ("Messages API only"), `openrouter:fusion`, `openrouter:advisor`, `openrouter:subagent`, `openrouter:experimental__search_models`, `openrouter:tool_search` ("Responses and Messages APIs"), `openrouter:files`.
- P3, `tools[].type` values extracted from the live spec with a script:
  - `ChatRequest`: `function`, `openrouter:advisor`, `openrouter:bash`, `openrouter:datetime`, `openrouter:files`, `openrouter:fusion`, `openrouter:image_generation`, `openrouter:experimental__search_models`, `openrouter:subagent`, `openrouter:web_fetch`, `openrouter:web_search`, the shorthand `web_search`, `web_search_preview`, `web_search_preview_2025_03_11`, `web_search_2025_08_26`, and a generic envelope with a free string `type` (`ChatDynamicServerTool`).
  - `ResponsesRequest`: `function`, `web_search_preview`, `web_search_preview_2025_03_11`, `web_search`, `web_search_2025_08_26`, `file_search`, `computer_use_preview`, `code_interpreter`, `mcp`, `image_generation`, `local_shell`, `shell`, `apply_patch`, `custom`, `namespace`, `openrouter:advisor`, `openrouter:subagent`, `openrouter:datetime`, `openrouter:files`, `openrouter:fusion`, `openrouter:image_generation`, `openrouter:experimental__search_models`, `openrouter:web_fetch`, `openrouter:web_search`, `openrouter:apply_patch`, `openrouter:bash`, `openrouter:shell`, `openrouter:tool_search`, `openrouter:mcp`.
  - `MessagesRequest`: `custom`, `bash_20250124`, `text_editor_20250124`, `web_search_20250305`, `web_search_20260209`, `advisor_20260301`, `tool_search_tool_bm25_20251119`, `tool_search_tool_bm25`, `tool_search_tool_regex_20251119`, `tool_search_tool_regex`, `openrouter:bash`, `openrouter:datetime`, `openrouter:image_generation`, `openrouter:experimental__search_models`, `openrouter:web_fetch`, `openrouter:web_search`, `openrouter:shell`, `openrouter:tool_search`, and one generic variant with a free string `type`.
  - These three lists equal P3 value for value.
- P4, live quotes: "The apply patch server tool is only available through the Responses API. It is not supported via the Chat Completions API." / "The shell server tool is available through the Responses API and the Messages API. Requesting it on the Chat Completions API returns a `400` error." / "`openrouter:bash` is only available on the Anthropic Messages API. Requesting it on the Chat Completions or Responses API returns a `400` error." / "The Chat Completions API does not accept it in `tools` and returns a `400`; on Chat Completions, use the `deferred_tools` protocol instead." / "Fusion in the `tools` array also works on `/chat/completions`, but it's slower there than on the Responses API" / "the tool is available on the Chat Completions, Responses, and Anthropic Messages APIs" (advisor) / "Supported on the Responses API (`/api/v1/responses`) only; other APIs reject it with a `400`." (subagent function inheritance).
- Disagreements between OpenRouter's own sources, both already in the first wave: (a) the spec lists `openrouter:bash` in the chat and Responses unions while the bash guide says those routes return a 400; (b) the overview page's `native_tools` example is labeled "Anthropic | anthropic/claude-sonnet-4.5" and includes `openrouter:web_fetch`, while the live endpoints route for that model lists `web_search`, `bash`, and `advisor` on the Anthropic endpoint and leaves `web_fetch` out.
- Verdict: confirmed.

### V20. Claim 20 - Gemini 4 long decode through OpenRouter (GG47, GG48, GG49)

- Source: `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gemini-4.md`, fetched 16:41 UTC. GG41 to GG46 quote Google's own documentation and are outside this pass.
- Live quote: "On OpenRouter, the continuation token is an encrypted reasoning item. Clients that already pass reasoning back between turns get long decode with no new fields."
- Live chat completions response shape: `"finish_reason": "length"`, `"native_finish_reason": "CONTINUATION"`, and in `message.reasoning_details` the entry `{"type": "reasoning.encrypted", "id": "gemini-continuation", "format": "google-gemini-v1", "data": "<opaque continuation token>", "index": 0}`. Responses shape: `{"type": "reasoning", "id": "gemini-continuation", "summary": [], "encrypted_content": "<opaque continuation token>", "format": "google-gemini-v1"}` under `"status": "incomplete"`, `"incomplete_details": { "reason": "max_output_tokens" }`. Messages shape: `"stop_reason": "max_tokens"` with `{ "type": "redacted_thinking", "data": "<opaque continuation data>" }`.
- Live quote (the loop): "keep the stopped turn with its continuation items (reasoning details or Responses API reasoning items whose `id` is `gemini-continuation`)" and "Resend the same conversation and parameters with the stopped turn appended as the last assistant turn. Set `max_tokens` above the total output of every hop so far, or the request returns a 400 error after a `MAX_TOKENS` hop."
- Live quote (stripping): "Only a continuation item in the last assistant turn resumes the decode. Each hop replaces the previous continuation turn, and OpenRouter removes continuation items from earlier turns, so a finished long answer in your history does not affect later requests. Visible text from the stopped turn does not need to be resent: OpenRouter drops the trailing continuation turn before forwarding the request, because the token already carries its state. Treat the token as opaque and pass it back byte for byte."
- Live quote (pinning): "**Continuations stay on one provider.** A continuation token only works on the provider that issued it. OpenRouter routes a continuation request back to that provider and does not fall back to another one. If that provider is unavailable, the request fails instead of restarting the answer elsewhere."
- Live quotes (limits): "**Token size grows with the decode** and can reach several MB for a long answer. Your client must accept and resend bodies of that size on later hops." / "Explicit caching with `cache_control` is skipped on continuation hops." / "Every hop is its own generation with its own usage and cost." / "**`reasoning.exclude` keeps the token.**" / "Messages has no native finish reason field." / "The `data` value wraps the Gemini token in an OpenRouter envelope so its format survives the round trip."
- Catalog check: `google/gemini-4-argon` is in the live catalog with five endpoints (Google AI Studio and Google Vertex).
- Verdict: confirmed. The pinning sentence was not among the GG47 to GG49 quotes I was given, and it is on the live page as the claim says.

### V21. Claim 21 - the suffix on encrypted reasoning blobs in documentation examples (R41, OA15, GG22)

- Method: I searched the live docs dump (`https://openrouter.ai/docs/llms-full.txt`, 388 pages, 16:41 UTC) and the nine individually fetched pages for a long base64 run, a dot, and a segment starting `eyJ`, then decoded each segment with Python `base64` (URL-safe alphabet, padding restored). Both searches gave the same 23 blobs on the same 9 pages.

| Page (under `https://openrouter.ai/docs/`) | Examples | Blob vendor and start | Field | Route | Decoded suffix |
| - | - | - | - | - | - |
| `api_reference/overview` | 1 | OpenAI, `gAAAAAB` | `encrypted_content`, `format` `openai-responses-v1` | Responses | `{"endpoint_slug":"openai/gpt-5.2-20251211\|openai"}` |
| `api_reference/responses/basic-usage` | 1 | OpenAI, `gAAAAAB` | `encrypted_content`, `openai-responses-v1` | Responses | `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07\|openai"}` |
| `api_reference/responses/web-search` | 10 | OpenAI, `gAAAAAB` | `encrypted_content`, `openai-responses-v1` | Responses | `{"endpoint_slug":"openai/o3-2025-04-16\|openai"}` |
| `agent-sdk/call-model/text-generation` | 1 | OpenAI, `gAAAAAB` | `encrypted_content` in a Python SDK object print, `format_='openai-responses-v1'` | Responses (`response.output`) | `{"endpoint_slug":"openai/gpt-5-nano-2025-08-07\|openai"}` |
| `guides/features/server-tools/apply-patch` | 1 | OpenAI served by Azure, `gAAAAAB` | `encrypted_content`, `azure-openai-responses-v1` | Responses | `{"endpoint_slug":"openai/gpt-5.1-codex-mini-20251113\|azure"}` |
| `guides/routing/provider-selection` | 5 | OpenAI served by Azure, `gAAAAAB` | `encryptedContent` (SDK output), `azure-openai-responses-v1` | Responses | `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07\|azure"}` |
| `guides/overview/multimodal/image-understanding` | 2 | Google Gemini on Vertex, `AY89a1` | `encrypted_content` and `encryptedContent`, `google-gemini-v1` | Responses | `{"endpoint_slug":"google/gemini-3-flash-preview-20251217\|google-vertex/global"}` |
| `guides/features/service-tiers` | 1 | OpenAI, `gAAAAAB` | `data` of a `redacted_thinking` block | Messages (`/api/v1/messages` with `openai/gpt-5-mini`) | `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07\|openai"}` |
| `guides/features/server-tools/shell` | 1 | OpenRouter's own, literal prefix `openrouter-server-tool` | `encrypted_content` | Responses | `{"server_tool_round":{"v":1,"members":[{"item_id":"msg_tmp_lk25kz4h0w"},{"item_id":"st_tmp_2foofw1c4sf","call_id":"toolu_01NiP9rEbn67fjNzu8Smb3QG"}]}}` |

- Totals: 22 `endpoint_slug` suffixes on 8 pages (20 on OpenAI blobs, 2 on Gemini blobs), plus 1 server-tool suffix. By route: of the 22 `endpoint_slug` examples, 21 are on Responses and 1 is on Messages; the server-tool example is on Responses; chat completions has 0. Every OpenAI blob starts with `gAAAAAB`.
- Chat completions check: the string `reasoning.encrypted` occurs six times in the docs dump. The only `data` values printed next to it are `eyJlbmNyeXB0ZWQiOiJ0cnVlIiwiY29udGVudCI6IltSRURBQ1RFRF0ifQ==` in the reasoning guide, which decodes to `{"encrypted":"true","content":"[REDACTED]"}` (a made-up placeholder), and `<opaque continuation token>` in the Gemini 4 guide. So the live docs print real encrypted blobs for the Responses and Messages routes only, and whether the suffix is inside chat completions `reasoning_details[].data` stays unshown, as R41's confidence note says.
- Comparison with the first wave: every decoded value in R41, OA15, and GG22 is reproduced. OA15's total of 22 payloads on 8 pages matches. Two counts are off: R41 says the Responses web search page has "five examples" and it has ten (two reasoning items in each of five outputs); OA15 says the provider selection page has four examples with `"format": "azure-openai-responses-v1"` and it has five. R41 lists five pages plus the shell page; the full set is the eight pages above plus the shell page.
- Eight of the 23 printed blobs have a clip marker in the middle of the value (the pages print a marker of the form `[2180 chars clipped]`). The segment after the last dot is whole in every case, so the decode is unaffected.
- The only prose about the suffix on any OpenRouter page is the one sentence in the Gemini 4 guide about an "OpenRouter envelope" on the Messages route (V20). So the meaning of `endpoint_slug` is an inference from the decoded bytes, here as in the first wave.
- Verdict: confirmed with a correction (the two counts, and the route breakdown).

### V22. Claim 22 - ignored by default, with documented rejections (S41 to S47)

- The default, live quotes: "If the chosen model doesn't support a request parameter (such as `logit_bias` in non-OpenAI models, or `top_k` for OpenAI), then the parameter is ignored. The rest are forwarded to the underlying model API." (`api_reference/overview`) and "With the default routing strategy, providers that don't support all the LLM parameters specified in your request can still receive the request, but will ignore unknown parameters." (`guides/routing/provider-selection`).
- S42 to S46, live: "When a sampling parameter is absent from your request, OpenRouter omits it upstream rather than substituting a hardcoded value, so the provider applies its own default." / "When you set `require_parameters` to `true`, the request won't even be routed to that provider." / "Even when `require_parameters` is `false`, a small set of parameters is used as a soft preference when choosing between providers of the same model: `tools`, the response format (`text.format` on the Responses API, `response_format` on Chat Completions, including structured outputs), and `verbosity`." / "OpenRouter will also transmit some provider-specific parameters, such as `safe_prompt` for Mistral or `raw_mode` for Hyperbolic directly to the respective providers if specified." The live catalog's `supported_parameters` vocabulary has 26 values with the same per-value model counts as S45 (`max_tokens` 459 down to `prediction` 9).
- Eight rejections, each quoted from the live page:
  1. Responses state: "Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error." (`api_reference/responses/overview`)
  2. Wrong role for an effort update: "The field is accepted only on `system` and `developer` messages. A `configuration_update` on a user, assistant, or tool message is rejected with a 400." (`model-migrations/gpt-6`)
  3. Effort update on a model without support: "On OpenRouter, a request that carries an update is routed only to endpoints that accept it, and a request for a model that does not support it is rejected with a 400 rather than sent with the update silently dropped." (`gpt-6`)
  4. Disabling mandatory reasoning: "return a 400 (`Reasoning is mandatory for this endpoint and cannot be disabled.`) before the request is routed" (`model-migrations/claude-5-5`)
  5. Bash and shell on the wrong route: "`openrouter:bash` is only available on the Anthropic Messages API. Requesting it on the Chat Completions or Responses API returns a `400` error." and "Requesting it on the Chat Completions API returns a `400` error." (`server-tools/bash`, `server-tools/shell`)
  6. Messages fallbacks: "`fallbacks` cannot be combined with the `models` parameter; sending both returns a 400 error." (`guides/routing/model-fallbacks`)
  7. Web search filters: "**Native (Google)** | No | No | Not supported. With `engine: "auto"`, falls back to Exa when filters are set. With `engine: "native"`, returns a 400 error" and "**Perplexity** | Yes | Yes | Mutually exclusive (when both provided, the request is rejected with a 400)" (`server-tools/web-search`)
  8. Unknown keys on extractions: "Unknown keys at the top level or inside a field are rejected with a `400`, not ignored, so a provider option in the wrong place never silently does nothing." (`guides/community/extractions`)
- The other fourteen rows of the S47 table: I found each row's quoted words on the live page or in the live spec too. That covers the effort value with no equivalent, the update combinations, GPT-6 features on an endpoint without support, replayed thinking blocks with an excluded provider, subagent function inheritance, function tools in a subagent's nested `tools`, computer use on Messages ("OpenRouter's Messages API does not accept either computer tool form, so computer-use requests through OpenRouter are rejected at validation."), tool search with `tool_choice`, `deferred_tools` on the Batch API, `access_programs.cyber`, an invalid `x_search`, `gs://` references, admin-disallowed service tiers (403), and `reasoning_effort` against `reasoning.effort` ("Cannot be used simultaneously with reasoning.effort if they differ."). So the count of 22 rows stands.
- Correction to one row. The S47 row "`deferred_tools` with a forcing `tool_choice`" quotes the tool search page's section "Controlling Tool Choice". The live text there is about the `openrouter:tool_search` tool with `defer_loading` tools, and it covers more than forcing: "Any other `tool_choice` conflicts with deferral - forcing a specific tool, requiring a call, or forbidding calls entirely all contradict "reveal these tools gradually." Rather than silently overriding what you asked for, the request fails with a `400`:" followed by "`tool_choice` conflicts with `openrouter:tool_search`. Deferred tools are revealed through `tool_choice`, so it must be omitted or set to `{"type": "allowed_tools", ...}`." For the `deferred_tools` protocol field the same page states a different rule, that `tool_choice` must stay the same on every turn (code `deferred_tools_catalog_changed`).
- Small precision note on another row: the status "400" is printed for an effort update combined with `mode: "pro"` ("that combination is rejected with a 400"). For two adjacent updates and for `truncation: "auto"` the pages say "Adjacent updates are rejected." and "Updates cannot be combined with `truncation: "auto"` in the Responses API." without printing a status.
- Verdict: confirmed with a correction (the label of the tool search row).

### V23. Claim 23 - the union rule, and which discovery routes answer without a key (M12, M2)

- Sources: live catalog (16:41 UTC) and the endpoints route for 25 models (16:47 UTC), compared with a script. `https://openrouter.ai/docs/guides/providers/integration.md` and `.../guides/features/structured-outputs.md` for the quotes.
- Live quote: "Text output: Plain sampling parameters are applied; parameters OpenRouter verifies by test (`tools`, `tool_choice`, `response_format`, `structured_outputs`, `logprobs`, and the rest of the capability suite) are never enabled from the feed, only by the capability tests passing." Live quote: "Support is determined per endpoint, not just per model: the same model may be served by multiple providers, and only some of those providers may support structured outputs. Endpoint support can also change over time."
- The five models I picked for the check, plus the twenty others fetched for other claims:

| Model | Endpoints | Catalog `supported_parameters` count | Union of endpoint lists count | Equal sets | Distinct endpoint lists |
| - | - | - | - | - | - |
| `deepseek/deepseek-v4.1-flash` | 37 | 22 | 22 | yes | 21 |
| `moonshotai/kimi-k3` | 29 | 20 | 20 | yes | 19 |
| `z-ai/glm-5.3` | 46 | 21 | 21 | yes | 24 |
| `meta-llama/llama-3.3-70b-instruct` | 7 | 17 | 17 | yes | 6 |
| `openai/gpt-6.2-sol` | 5 | 11 | 11 | yes | 3 |
| `anthropic/claude-fable-5.1` | 3 | 11 | 11 | yes | 2 |
| `anthropic/claude-fable-5.2` | 4 | 11 | 11 | yes | 2 |
| `anthropic/claude-haiku-5.5` | 5 | 11 | 11 | yes | 3 |
| `anthropic/claude-opus-4.8` | 6 | 11 | 11 | yes | 3 |
| `anthropic/claude-opus-5` | 6 | 11 | 11 | yes | 3 |
| `anthropic/claude-opus-5.5` | 6 | 11 | 11 | yes | 3 |
| `anthropic/claude-sonnet-4.5` | 5 | 12 | 12 | yes | 3 |
| `anthropic/claude-sonnet-5.5` | 5 | 11 | 11 | yes | 3 |
| `google/gemini-3.5-flash` | 6 | 12 | 12 | yes | 2 |
| `google/gemini-4-argon` | 5 | 12 | 12 | yes | 2 |
| `meta-llama/llama-4-maverick` | 1 | 16 | 16 | yes | 1 |
| `meta/muse-ember` | 2 | 13 | 13 | yes | 2 |
| `meta/muse-spark-1.3` | 2 | 13 | 13 | yes | 2 |
| `meta/muse-wonder` | 2 | 13 | 13 | yes | 2 |
| `mistralai/mistral-large-2512` | 2 | 11 | 11 | yes | 1 |
| `mistralai/mistral-large-4-0` | 2 | 16 | 16 | yes | 1 |
| `mistralai/mistral-medium-3-5` | 2 | 14 | 14 | yes | 1 |
| `openai/gpt-5.5` | 4 | 11 | 11 | yes | 2 |
| `qwen/qwen3.7-flash` | 1 | 12 | 12 | yes | 1 |
| `x-ai/grok-4.7` | 5 | 15 | 15 | yes | 2 |

- Result: 25 of 25 equal, over 198 endpoint records. On all 25 the catalog `context_length` also equals the largest endpoint `context_length`. The rule is still an inference from data alone, as M12 says.
- What the union hides, from the same data: on `moonshotai/kimi-k3` the model-level list includes `tools`, `tool_choice`, `temperature`, and `top_p`, and each of those four is missing from at least one of its 29 endpoints. On `meta-llama/llama-3.3-70b-instruct` both `tools` and `tool_choice` are missing from at least one of 7 endpoints.
- M2, live probes with no `Authorization` header, cookie, or key, 16:41 and 16:47 UTC:

| Route (`GET https://openrouter.ai...`) | Status | Body |
| - | - | - |
| `/api/v1/models` | 200 | 476 entries, `"total_count": 476`, 925,527 bytes |
| `/api/v1/models/count` | 200 | `{"data":{"count":476}}` |
| `/api/v1/providers` | 200 | 123 providers |
| `/api/v1/endpoints/zdr` | 200 | 936 endpoint records |
| `/api/v1/models/{author}/{slug}/endpoints` | 200 for all 25 models tried | Model header plus an `endpoints` array |
| `/api/v1/tools` | 401 | `{"error":{"message":"No cookie auth credentials found","code":401}}` |
| `/api/v1/models/user` | 401 | `{"error":{"message":"Unauthorized","code":401}}` |

- Same statuses and same bodies as M2 for every route I tried.
- Verdict: confirmed.

### V24. Claim 24 - `phase`, async tools, and `configuration_update` (OA41, OA43, OA45)

- Sources: `.../model-migrations/gpt-5-4.md`, `.../model-migrations/gpt-6.md`, the reasoning guide, live spec. Fetched 16:41 UTC.
- Live quote (`phase`): "`phase` is **not available** in the Chat Completions API. The Chat Completions format cannot represent multiple output items with distinct phases in a single response. Use the Responses API for full `phase` support."
- Live quote: "The Chat Completions API does **not** support `phase` in responses. A single chat completion response can only contain one message per choice, so there is no way to represent the separate commentary and final answer output items that models like GPT-5.4 produce."
- Live quote: "When using the Responses API, assistant output items include `phase`. You must persist these items verbatim and pass them back in subsequent requests." The live table rows: `openai/gpt-5.5`, `openai/gpt-5.5-pro`, `openai/gpt-5.4`, `openai/gpt-5.4-pro`, `openai/gpt-5.3-codex` "Supported"; "Other OpenAI models | Silently ignored (safe to pass)"; "Non-OpenAI models | Not applicable".
- Live spec: `phase` is a property of `OutputMessage`, `EasyInputMessage`, and the message variant of `BaseInputs` (all Responses shapes), plus an unrelated `phase` on `ImageGenTextChunkEvent`. `ChatAssistantMessage` has ten properties: `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`.
- Live quote (async tools): "OpenRouter supports async tools on the Responses API. Chat Completions and Anthropic Messages requests run every tool call synchronously. A Chat Completions tool that carries `async` is accepted, but the flag is removed before the request is routed and the call behaves like any other tool call."
- Live quote (routing): "OpenAI and Azure OpenAI honor async tools for the GPT-6 family. When a request declares an async tool and one of those endpoints is a routing candidate, OpenRouter narrows routing to them. When none is, the request keeps its normal candidate pool and the `async` flag is removed before the request is sent, so the call runs synchronously and the returned `function_call` carries no `async` field. Amazon Bedrock accepts the flag but runs the tool synchronously, so it is treated as a non-async transport."
- Live spec: `async` is a property of the Responses function tool, `CustomTool`, `NamespaceFunctionTool`, and the Responses call items (`FunctionCallItem`, `CustomToolCallItem`, and their output forms). Description on the tool: "Only honored by providers whose Responses API supports async tools; ignored elsewhere." The chat tool schema (`ChatFunctionTool`) lists `type`, `function`, `cache_control`, `defer_loading`, and inside `function`: `name`, `description`, `parameters`, `strict`, `defer_loading`. So in the spec `async` is a Responses-side property only, which fits the guide's "accepted, but the flag is removed".
- Live quote (`configuration_update`, chat form, GPT-6 guide): "OpenAI's Chat Completions API has no per-message effort control, so this form is an OpenRouter extension. Place `configuration_update` on a content-less system message (`content: ""`) directly before the user message it applies to. OpenRouter translates it into the Responses `configuration_update` item before forwarding to the model." Example message: `{"role": "system", "content": "", "configuration_update": { "reasoning": { "effort": "high" } }}`.
- Live quote: "The field is accepted only on `system` and `developer` messages. A `configuration_update` on a user, assistant, or tool message is rejected with a 400."
- Live quote (reasoning guide): "The three forms are interchangeable: OpenRouter normalizes each into one internal representation and re-emits it in the form the selected provider accepts. A Responses `configuration_update` sent to a Claude model becomes a per-message `output_config`, and a Chat Completions or Messages API update sent to an OpenAI model becomes a Responses `configuration_update` item."
- Live spec: `configuration_update` is a property of `ChatSystemMessage` and `ChatDeveloperMessage` only. `ChatSystemMessage.configuration_update.description` starts "OpenRouter extension. Changes reasoning effort from this point in the conversation onward without invalidating the prompt cache for the preceding turns. Place it on a content-less system message (`content: ""`) directly before the user message it should apply to". `ConfigurationUpdateReasoning` has one property, `effort`, with the seven effort values and `additionalProperties: false`.
- Changelog: "`ChatSystemMessage`: new property `configuration_update`" is under September 4, 2026 and "`ChatDeveloperMessage`: new property `configuration_update`" under September 8, 2026, as OA45 says.
- Verdict: confirmed.

### V25. Claim 25 - Responses is GA, and both native-shaped routes serve other vendors' models (S5, S7, S8)

- Source for S5: `https://openrouter.ai/docs/changelog.md` (105 dated entries, July 3 to November 21, 2026), fetched 16:41 UTC. Under the heading "**July 25, 2026**": "**Migration: no action for HTTP clients.** The Responses API is now GA. `beta.responses` was an OpenAPI grouping tag and SDK namespace, never a URL. `POST /api/v1/responses` and its request and response schemas are unchanged, so raw HTTP callers and OpenAI-compatible clients (including the OpenAI Agents SDK) need no changes."
- Live spec: the `/responses` operation still has both tags, `Responses` and `beta.responses`; the second tag's description is "Deprecated alias of responses. Use responses instead; scheduled for removal (sunset date TBD)." The `/messages` operation has `"x-speakeasy-ignore": true` and the tag "Anthropic Messages" ("Anthropic Messages endpoints").
- Live quotes on how many shapes there are: "OpenRouter supports two request shapes: the Responses API (`POST /api/v1/responses`) and the Chat Completions API (`POST /api/v1/chat/completions`). Where this page shows both, the Responses API comes first." (API overview) and "OpenRouter exposes three API skins." (errors page).
- S7, Messages with non-Anthropic models, live:
  - Reasoning guide: "Requests sent through the Anthropic Messages API control reasoning with Anthropic's native `thinking` and `output_config.effort` fields. OpenRouter normalizes both into the unified `reasoning` parameter, so they work on every reasoning model, not only Claude."
  - GPT-6 guide, section "Anthropic Messages API", example body with `"model": "openai/gpt-6-sol"`, `"max_tokens": 4096`, and a message `{ "role": "system", "content": [], "output_config": { "effort": "high" } }`: "The Messages API uses Anthropic's per-message `output_config.effort` on a content-less system message. OpenRouter translates it into the Responses `configuration_update` item when the request routes to a GPT-6 model."
  - Gemini 4 guide, section "Anthropic Messages API", example body with `"model": "google/gemini-4-argon"`.
  - Service tiers guide: a Go example that posts `"model": "openai/gpt-5-mini"` to `https://openrouter.ai/api/v1/messages`, with printed output that starts `{"id":"msg_fNtJ3JZ98rD2kxnDp7MH","type":"message","role":"assistant"` and ends `"context_management":null,"provider":"OpenAI"}`.
  - Advisor guide: "On `/api/v1/messages`, request the advisor with the native Anthropic tool shape, and it works with **any executor model**, not just Anthropic ones".
  - The stated limit: "Claude Code with OpenRouter is only guaranteed to work with the Anthropic first-party provider."
- S8, Responses with non-OpenAI models, live:
  - "OpenRouter's Responses API provides OpenAI-compatible access to multiple AI models through a unified interface, designed to be a drop-in replacement for OpenAI's Responses API."
  - Reasoning guide: a Responses body with `"model": "anthropic/claude-fable-5.2"` and a `{"type": "configuration_update", ...}` input item, and a Responses "Preserving Reasoning" body with `"model": "anthropic/claude-sonnet-4.5"`.
  - Gemini 4 guide: Responses body with `"model": "google/gemini-4-argon"`.
  - Agent SDK: "`callModel` is built on OpenRouter's Responses API which uses an **items-based model** rather than the messages-based model used by OpenAI Chat or Vercel AI SDK."
  - The printed Responses outputs decoded in V21 include a Gemini model served by Google Vertex.
- Verdict: confirmed.

### V26. Claim 26 - the beta header allowlist, and where Claude's native web tools exist (AN82, AN40, AN85)

- Source for AN82: `guides/routing/provider-selection.md`, fetched 16:41 UTC. Live quote: "OpenRouter keeps an allowlist of other `x-anthropic-beta` values for each provider and forwards only the values on the list for the provider that serves the request. Providers that serve Anthropic's Messages API directly have the longest list, and Google Vertex and Amazon Bedrock receive only the values they accept. OpenRouter drops every other value, and some values it recognizes are withheld on purpose."
- The page names two values, `interleaved-thinking-2025-05-14` and `structured-outputs-2025-11-13`, and keeps the rest of the allowlist unpublished.
- Source for AN40: the endpoints route for eight Claude models, fetched 16:47 UTC. `native_tools` per endpoint tag:

| Model | Tags that list `openrouter:web_search` and `openrouter:web_fetch` | Tags that list `openrouter:bash` only |
| - | - | - |
| `anthropic/claude-fable-5.2` | `anthropic`, `anthropic/fast` | `azure`, `google-vertex/global` |
| `anthropic/claude-fable-5.1` | `anthropic` | `azure`, `google-vertex/global` |
| `anthropic/claude-opus-5.5` | `anthropic`, `anthropic/fast`, `claude-on-aws` | `amazon-bedrock`, `azure/global`, `google-vertex/global` |
| `anthropic/claude-sonnet-5.5` | `anthropic`, `claude-on-aws` | `amazon-bedrock`, `azure/global`, `google-vertex/global` |
| `anthropic/claude-haiku-5.5` | `anthropic`, `claude-on-aws` | `amazon-bedrock`, `azure/global`, `google-vertex/global` |
| `anthropic/claude-opus-5` | `anthropic`, `anthropic/fast`, `claude-on-aws` | `amazon-bedrock`, `azure/global`, `google-vertex/global` |
| `anthropic/claude-opus-4.8` | `anthropic`, `anthropic/fast`, `claude-on-aws` | `amazon-bedrock`, `azure/global`, `google-vertex/global` |
| `anthropic/claude-sonnet-4.5` | `anthropic`, `claude-on-aws` (web search only; `web_fetch` is on neither) | `amazon-bedrock`, `azure/global`, `google-vertex/global` |

- The native types are `web_search_20260209`, `web_fetch_20260209`, `bash_20250124`, and `advisor_20260301`. The tags with web search also list `openrouter:advisor`. The provider name on the `claude-on-aws` tag is "Claude Platform on AWS". The live listings for `claude-fable-5.2` and `claude-fable-5.1` consist of Anthropic, Azure, and Google Vertex endpoints only.
- So on all eight models, native web search exists only on endpoints whose provider is Anthropic or Claude Platform on AWS, and native web fetch on the same endpoints for seven of the eight. Same as AN40.
- About the id: in the first-wave file AN85 is the fast mode item. It holds too. Live quotes (`cookbook/coding-agents/claude-code-integration.md`): "the three spellings are fully interchangeable on all APIs. The fast tier endpoint is tried first, with fallback to standard endpoints (billed at their standard rates)." / "The required beta header is injected automatically whenever the fast tier endpoint serves the request." / "Fast mode is only served by the Anthropic first-party provider, since other providers (e.g. Amazon Bedrock, Google Vertex) do not support it." The live listings show an `anthropic/fast` tag with `service_tier: priority` on Fable 5.2, Opus 5.5, Opus 5, and Opus 4.8.
- Verdict: confirmed.

### V27. Claim 27 - fields outside the spec's chat response schemas (O53, S19)

- Source: live spec, read with Python. Fetched 16:41 UTC.
- `ChatChoice` properties: `finish_reason`, `index`, `logprobs`, `message`. `ChatStreamChoice` properties: `delta`, `finish_reason`, `index`, `logprobs`.
- `ChatResult` properties: `alignment`, `choices`, `created`, `id`, `model`, `object`, `openrouter_metadata`, `safeguard_results`, `service_tier`, `system_fingerprint`, `usage`. `ChatStreamChunk` has the same eleven plus `error`.
- `ChatAssistantMessage` properties: `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`. `ChatStreamDelta`: `audio`, `content`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`.
- `native_finish_reason` is declared as a property in exactly two places in all 1071 schemas: the inline chat body inside `BatchObject` (`results[].response.body.anyOf[0].choices[]`, type string or null) and `GenerationResponse.data` ("Native finish reason as reported by provider").
- `provider` as a response property: the inline chat body inside `BatchObject` (string or null), `MessagesResult` (string), `MessagesStartEvent.message` (an enum of provider names), plus `GenerationResponse.data.provider_name`. The chat result and chat stream chunk schemas list the properties above, and `provider` is not one of them.
- `annotations` on a chat message: declared only on the inline chat body inside `BatchObject`. (`ResponseOutputText.annotations` is the Responses shape.)
- None of these response schemas sets `additionalProperties: false`, so the extra fields are legal on the wire; they are just untyped for a client generated from the spec.
- The fields are documented outside the spec, live: the API overview's types have `native_finish_reason: string | null;` on both `NonStreamingChoice` and `StreamingChoice`, with the example lines `"finish_reason": "stop", // Normalized finish_reason` and `"native_finish_reason": "stop", // The raw finish_reason from the provider`. The PDF guide's printed chat output (`https://openrouter.ai/docs/guides/overview/multimodal/pdfs.md`, fetched 17:02 UTC) has `"provider": "NextBit",` `"system_fingerprint": "vllm-0.32.0-8349ae92",` `"service_tier": null,` then `"native_finish_reason": "stop",` and `"annotations": [`.
- Comparison: same lists as O53 and S19. One addition: `MessagesResult` and `MessagesStartEvent` do declare `provider`, so the gap is specific to the chat completions schemas.
- Verdict: confirmed.

## Claims that are not plainly confirmed

Verdicts other than `confirmed`:

1. **Claim 5 (R62, AN12) - confirmed with a correction.** The `drop_block` request control (`thinking.block_binding.prefix_mismatch_behavior`) exists on the Messages route only. The Responses route has a result field with the same name, `input_transformations`, and it is a different report: OpenAI's own, shaped `id`, `type`, `action`, `reason`, "Present only when the provider reported the field". The Claude `thinking_dropped` report is specified for Messages results, the Messages stream start event, and Messages batch results. R62's closing line calls both the switch and the report "Messages and Responses features", which overstates the Responses side.
2. **Claim 10 (C25 against C29) - conflict.** "Sticky sessions expire after **60 minutes** of inactivity." (prompt caching page) against "OpenRouter stores a 10-minute best-effort pin" (latency and performance page). Both are live today, and no third page gives a figure.
3. **Claim 16 (O51) - confirmed with a correction.** The "Not Currently Supported" cancellation list has 24 provider names, not 30. The supported list has 24. The names in O51 are right.
4. **Claim 17 (T10, M24) - conflict, for Haiku 5.5 only.** Guide: "Forced tool use still works on Haiku 5.5 (`tool_choice: "required"`, `{"type": "any"}`, or a named tool)". Catalog: all five `anthropic/claude-haiku-5.5` endpoints report `"required": false, "function": false`. Everything else in the claim is confirmed.
5. **Claim 21 (R41, OA15) - confirmed with a correction.** All decoded values reproduce. The Responses web search page has ten suffixed blobs (R41 says five). The provider selection page has five examples with `azure-openai-responses-v1` (OA15 says four). Of the 22 `endpoint_slug` examples, 21 are on the Responses route and 1 is on the Messages route; the only chat completions `reasoning.encrypted` values the docs print are placeholders.
6. **Claim 22 (S47) - confirmed with a correction.** All 22 rejection quotes are live. The row labeled "`deferred_tools` with a forcing `tool_choice`" quotes a rule the page states for `openrouter:tool_search` with `defer_loading`, and that rule also rejects forbidding calls. For two adjacent effort updates and for `truncation: "auto"` the pages say "rejected" and "cannot be combined" without printing a status code.

Disagreements between OpenRouter's own sources that sit inside confirmed claims (all were already recorded by the first wave, and all are still live):

- Claim 2: the reasoning guide says entries follow "one of three possible types"; the spec maps four.
- Claim 7: the tool search guide says `portable` "is the only profile in this release"; the spec's `profile` enum is `portable`, `universal`.
- Claim 9: the caching page says both that `prompt_cache_breakpoint` "carries no `ttl`" and that it "can carry its own `ttl`".
- Claim 19: the spec lists `openrouter:bash` in the chat completions and Responses tool unions; the bash guide says those routes return a 400. The overview page's `native_tools` example for Claude Sonnet 4.5 includes `openrouter:web_fetch`; the live endpoints route for that model does not.
- Claim 25: the API overview says "two request shapes"; the errors page says "three API skins".

Notes on evidence ids in the task list:

- OA2 is the item about the spec listing one Responses path. The 400 on `previous_response_id` and `store: true` is in OA1 and S26.
- AN85 is the fast mode item. The statement about native web search and web fetch endpoints is in AN40 and AN95.
- GG41 to GG46 are Google's own rules, quoted from Google's documentation. Only GG47 to GG49 are OpenRouter claims.

What this pass cannot settle: every verdict is about documented behavior. No request was sent to a model, so whether the wire matches the documents (for example whether a chat completions `reasoning.encrypted` blob has the endpoint suffix, or which of the two sticky lifetimes is real) is still unobserved.

## Sources

All fetched on 2026-11-22 with no credentials. Doc pages were fetched as `<url>.md`.

| Key | URL | Fetch time (UTC) | Kind |
| - | - | - | - |
| V-s1 | `https://openrouter.ai/openapi.json` (3,118,352 bytes, SHA-256 prefix `e6614516bb3f4059`, 123 paths, 1071 schemas) | 16:41 | OpenRouter spec |
| V-s2 | `https://openrouter.ai/api/v1/models` (925,527 bytes, 476 models) | 16:41 | OpenRouter public catalog |
| V-s3 | `https://openrouter.ai/docs/llms-full.txt` (7,332,832 bytes, 388 pages) | 16:41 | OpenRouter doc (full dump) |
| V-s4 | `https://openrouter.ai/docs/guides/best-practices/reasoning-tokens` | 16:41 | OpenRouter doc |
| V-s5 | `https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/` pages `claude-5-5`, `fable-5-1`, `opus-5`, `gpt-5-4`, `gpt-5-6`, `gpt-6`, `gemini-4` | 16:41 | OpenRouter doc |
| V-s6 | `https://openrouter.ai/docs/guides/best-practices/prompt-caching` and `.../latency-and-performance` | 16:41 | OpenRouter doc |
| V-s7 | `https://openrouter.ai/docs/changelog` (730,262 bytes, 105 dated entries) | 16:41 | OpenRouter doc (changelog) |
| V-s8 | `https://openrouter.ai/docs/api_reference/` pages `overview`, `parameters`, `streaming`, `errors-and-debugging`, `responses/overview`, `responses/basic-usage`, `responses/web-search`, `responses/reasoning` | 16:41 | OpenRouter doc |
| V-s9 | `https://openrouter.ai/docs/guides/features/server-tools` and its pages `web-search`, `web-fetch`, `tool-search`, `shell`, `bash`, `apply-patch`, `advisor`, `subagent`, `fusion` | 16:41 | OpenRouter doc |
| V-s10 | `https://openrouter.ai/docs/guides/routing/provider-selection`, `.../routing/model-fallbacks`, `.../providers/traffic-routing`, `.../providers/integration` | 16:41 | OpenRouter doc |
| V-s11 | `https://openrouter.ai/docs/guides/features/` pages `plugins/response-healing`, `plugins/web-search`, `service-tiers`, `structured-outputs`, `compatibility`, `batch-api` | 16:41 | OpenRouter doc |
| V-s12 | `https://openrouter.ai/docs/guides/overview/multimodal/videos`, `.../image-understanding`, `.../pdfs` (17:02), `https://openrouter.ai/docs/guides/overview/auth/byok` | 16:41 | OpenRouter doc |
| V-s13 | `https://openrouter.ai/docs/cookbook/administration/usage-accounting`, `.../cookbook/coding-agents/claude-code-integration`, `https://openrouter.ai/docs/faq`, `https://openrouter.ai/docs/batch-quickstart`, `https://openrouter.ai/docs/guides/community/extractions` | 16:41 | OpenRouter doc |
| V-s14 | `https://openrouter.ai/docs/agent-sdk/call-model/text-generation`, `.../items`, `.../message-formats` | 16:41 | OpenRouter doc |
| V-s15 | `https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints` for 25 models: `anthropic/claude-fable-5.2`, `claude-fable-5.1`, `claude-opus-5.5`, `claude-sonnet-5.5`, `claude-haiku-5.5`, `claude-opus-5`, `claude-opus-4.8`, `claude-sonnet-4.5`; `meta/muse-ember`, `muse-spark-1.3`, `muse-wonder`; `meta-llama/llama-4-maverick`, `llama-3.3-70b-instruct`; `mistralai/mistral-large-4-0`, `mistral-large-2512`, `mistral-medium-3-5`; `deepseek/deepseek-v4.1-flash`; `moonshotai/kimi-k3`; `google/gemini-3.5-flash`, `gemini-4-argon`; `openai/gpt-6.2-sol`, `gpt-5.5`; `x-ai/grok-4.7`; `qwen/qwen3.7-flash`; `z-ai/glm-5.3` (all HTTP 200) | 16:47 | OpenRouter public catalog |
| V-s16 | `https://openrouter.ai/api/v1/models/count`, `/providers`, `/endpoints/zdr` (HTTP 200), `/tools`, `/models/user` (HTTP 401) | 16:47 | OpenRouter public API response |

One probe returned 404 and was not used: `https://openrouter.ai/docs/guides/features/server-tools/overview.md` (the overview page lives at `.../server-tools.md`).
