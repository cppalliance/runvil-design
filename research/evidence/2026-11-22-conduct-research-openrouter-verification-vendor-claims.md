---
produced: 2026-11-22
title: OpenRouter deep dive - independent verification of first-wave claims about vendor native rules (OpenAI Chat Completions tool calling with reasoning, encrypted_content, input_transformations, phase, Anthropic thinking binding and prefix check, stop reasons, forced tool_choice, sampling parameter removal, Gemini thought signatures and missing_thought_signature, cachedContents, DeepSeek reasoning_content, Mistral tool-call ids, Kimi, Meta tool_choice, vLLM and llama.cpp reasoning fields, xAI tool cap)
---

# OpenRouter deep dive - independent verification of first-wave claims about vendor native rules

- What was read: 24 claims that first-wave subagents recorded about the model vendors' own native rules were opened in their evidence files, then each vendor page was fetched again from the vendor's live site and compared word for word.
- How many sources: 60 source keys in the Sources table at the end, covering about 100 pages and files. Every one was fetched without credentials. Raw copies are in this subagent's scratch folder.
- Read date: 2026-11-22, between 08:41 and 08:54 Pacific time (UTC-8). Each section gives its own fetch time.

Verdict words used: `confirmed` (the live page says what the first wave recorded), `confirmed with a correction` (it holds, with the stated change), `not confirmed` (the vendor page does not say this), `conflict` (two vendor pages disagree, both quoted), `could not re-fetch`.

## Verdict table

Counts over 27 rows (claims 13, 20, and 24 each have two rows): `confirmed` 21, `confirmed with a correction` 2, `not confirmed` 3, `conflict` 1, `could not re-fetch` 0.

| Claim | What the first wave recorded | Live source and fetch time | Verdict | Correction or note |
|---|---|---|---|---|
| 1. OA35, OA36, OA37: Chat Completions, tools, and reasoning | From GPT-5.4, Chat Completions "does not support tool calling with `reasoning_effort` values other than `none`". On GPT-6 models tool calling "requires Responses". Chain of thought passes between turns on Responses only | OpenAI migration guide, Using GPT-6, Using GPT-6.2, reasoning guide, function calling guide, changelog, 11 model pages, 08:41 to 08:43 (VV1) | confirmed with a correction | Five GPT-6 models need Responses for any tool call (GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6.2 Astra, GPT-6.2 Sol). GPT-6 Sol and GPT-6 Luna keep function calling on Chat Completions with `reasoning_effort: "none"`. Stated as a limitation in prose. OpenAI prints no status or error body for the tool case |
| 2. OA25: `encrypted_content` by default | "This is populated by default for reasoning items returned by `POST /v1/responses` and WebSocket `response.create` requests." | Responses reference, reasoning guide, migration guide, 08:41 (VV2) | confirmed | The reasoning guide words the default as stateless mode. The reference and migration guide state it for every create request |
| 3. OA27, OA62: dropped reasoning, `input_transformations`, `instructions` on GPT-6.2 | Removal on model family, organization, or user change. From 6.2 also on a `safety_identifier` change or an edited prefix. Five `reason` values. From 6.2 top-level `instructions` go after `input` and bill uncached | Reasoning guide, Responses reference, Using GPT-6.2, prompt caching guide, 08:41 (VV3) | confirmed | Also removed on a move "to a less capable model within the same family" |
| 4. Q30, Q45, surfaces file: `cache_write_tokens`, `prompt_cache_breakpoint`, `url_citation` in OpenAI's chat reference | All three are in OpenAI's own Chat Completions reference | Chat Completions reference, 08:41 (VV4) | confirmed | On Chat Completions the request-level `prompt_cache_options.ttl` accepts `"30m"` only, while the marker's own `ttl` lists `"30m"` or `"12h"` |
| 5. `invalid_encrypted_content` for a cross-organization blob (workspace report, 2026-09-20) | OA: the string is in none of today's OpenAI pages. Docs describe removal reported as `organization_mismatch` | 30 OpenAI pages searched, 08:41 to 08:43 (VV5) | not confirmed | OA's finding holds. Documented outcome is "removed" with reason `organization_mismatch`. The outcome for a blob OpenAI never minted is undocumented |
| 6. OA41 (and OA40): `phase` | Values `commentary` and `final_answer` on assistant messages. Preserve and resend on models from `gpt-5.3-codex` on. Responses API only | Responses reference, reasoning guide, Using GPT-5.4, conversation state guide, 08:41 (VV6) | confirmed | OA41 is the OpenRouter item and OA40 the vendor rule. OpenAI's rule is a recommendation with a quality cost, not a validation error |
| 7. AN5, AN3: account-bound thinking, prefix check | Blocks from Fable 5.2, Mythos 5.2, Sonnet 5.5, Haiku 5.5 work only in the producing account. Account is the organization on the Claude API. Prefix check counts "changing text before a thinking block in its own message". 400 on Fable 5.2, Mythos 5.2, Mythos 5.1. Possible 400 on Fable 5.1, Opus 5.5, Sonnet 5.5, Haiku 5.5 | Preserved thinking page and prefix checks page, 08:43 (VV7) | confirmed | Extra sentence: a block from an account "no longer in good standing" can also be left out |
| 8. `compaction` stop reason, `pause_turn` | Seven stop reasons in the reference, `compaction` as an eighth in the compaction guide. `pause_turn`: send the response back as-is | Stable and beta Messages references, stop reasons guide, compaction guide, server tools page, 08:43 to 08:44 (VV8) | confirmed | The beta reference's enum lists eight values including `"compaction"`. A paused turn continues by re-sending the assistant content unchanged with the same tools and the container id |
| 9. T48: forced `tool_choice` | `any` and `tool` return a 400 on Fable 5.2, Mythos 5.2, Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1, and error under manual extended thinking | Tool use page and errors page, 08:43 to 08:44 (VV9) | confirmed | Error text, from the errors page: `tool_choice: type "tool" and "any" are not supported for this model.` (400 `invalid_request_error`) |
| 10. T51: structured outputs without a beta header | "beta headers are no longer required", `output_config.format` | Structured outputs page, 08:43 (VV10) | confirmed | The old `output_format` field still needs `structured-outputs-2025-11-13` |
| 11. Sampling parameters removed under `2026-10-21` | A request that sends `temperature`, `top_p`, or `top_k` "returns a 400 error on every model". Under `2023-06-01`, 4.7 and later reject non-default values | Versioning page, migration page, thinking page, stable Messages reference, 08:43 (VV11) | confirmed | Under the older version on 4.7 and later, `top_p: 1` is non-default (default `0.99`) and `temperature` with `top_p` together is a 400 even at defaults |
| 12. AN83: forward betas and unknown fields | "Forward the header verbatim; don't allowlist individual values" | Claude Code gateway compatibility guide, 08:43 (VV12) | confirmed | The page has a section titled "Forward as open lists". Translation to a non-Anthropic upstream is named as the gateway's own job |
| 13a. GG2 to GG6: thought signature rule, the 400, the token | First function call part of each step in the current turn must have its signature. Earlier turns are not validated. 400 with the "is missing a `thought_signature`" message. `MISSING_THOUGHT_SIGNATURE` is a finish reason and an Interactions generation error code | AI Studio thought signatures page, `generateContent` reference, both API errors pages, 08:46 to 08:47 (VV13) | confirmed | The 400 body has `code: 400` and `status: "INVALID_ARGUMENT"` |
| 13b. Local survey: "a 400 with code `missing_thought_signature`" | First wave said this is wrong | Same pages (VV13) | not confirmed | The token is a `finishReason` on `generateContent` and a "Generation error code" on the Interactions API. It is not the code of the HTTP 400 |
| 14. GG7, GG13: text part signatures, tool combination | Optional on text parts. With built-in tools plus custom functions every part is signed and all must come back | Thought signatures page, thinking page, tool combination page, 08:46 to 08:47 (VV14) | confirmed | `id` and `tool_type` must come back with the parts too |
| 15. GG8: dummy signature values | `"context_engineering_is_the_way_to_go"` or `"skip_thought_signature_validator"` on AI Studio. Vertex names the second only | AI Studio thought signatures page 08:46, Vertex page 08:48 (VV15) | confirmed | |
| 16. GG25, GG98: OpenAI-compatible route | Signature in `tool_calls[].extra_content.google.thought_signature`. `cached_content` through `extra_body` | Vertex page 08:48, AI Studio OpenAI compatibility page 08:47 (VV16) | confirmed | The path is printed on the Vertex page only. On the wire `cached_content` sits in a top-level `extra_body.google` object |
| 17. GG52 and following, GG41: explicit caching on Gemini 4 and Interactions, continuation token | Explicit caching "is not supported in the Interactions API, or for Gemini 4 models". `finish_reason: CONTINUATION` with a `continuation_token` | Both caching pages, Gemini 4 Carbon page, long decode guide, `generateContent` reference, 08:47 (VV17) | confirmed | REST field name is `continuationToken`. The quotes sit in the items after GG52 |
| 18. Signature portability between AI Studio and Vertex | First wave: Google's pages are silent | AI Studio pages 08:46 to 08:47, Vertex page 08:48 (VV18) | not confirmed | Google names no AI Studio and Vertex rule. The Vertex page does scope a signature to "the same conversation history, model, and Google Cloud project", with omission across projects and a 400 for a corrupted value. New to the evidence |
| 19. OV22, OV23: DeepSeek `reasoning_content` with `tools` | All prior `reasoning_content` must go back when `tools` is present, else "the API will return a 400 error". The page does not print the message | DeepSeek thinking mode guide, tool calls guide, chat reference, error codes page, 08:51 (VV19) | confirmed | The message string rests on third parties |
| 20a. OV30: Mistral tool-call ids | Exactly nine alphanumeric characters, error code 3280 | Mistral docs 08:51, `mistral-common` validator 08:52 (VV20) | confirmed with a correction | Docs say "exactly 9 characters" for fine-tuning data. Mistral's validator enforces `^[a-zA-Z0-9]{9}$` for tokenizer versions 3 to 11 and accepts any non-empty id other than `"null"` for version 13 or newer |
| 20b. OV35: Mistral's two chat endpoints | `/v1/chat/completions` "uses the native Mistral format". `/openai/v1/chat/completions` is OpenAI-compatible and in public preview | Mistral docs, 08:51 (VV20) | confirmed | |
| 21. T57 and the Kimi replay rule | "K3 drops the ``func:index`` format requirement". Pass the complete assistant message back as-is, including `reasoning_content` | Kimi-K3 `encoding_k3.py`, four Kimi platform pages, 08:51 (VV21) | confirmed | Kimi prints no error string for a missing `reasoning_content` |
| 22. OV64: Meta `tool_choice`, and the OpenRouter catalog | Meta: only `"auto"`, other values return HTTP 400. OpenRouter's Meta endpoint shows all four modes true | Meta tool calling page 08:51, three OpenRouter endpoint lists 08:53 (VV22) | confirmed | The conflict holds for `meta/muse-spark-1.3` and `meta/muse-wonder`. For `meta/muse-ember` OpenRouter shows `auto` only |
| 23. Q37, OV73, OV67: reasoning field names | vLLM renamed `reasoning_content` to `reasoning`. llama.cpp returns `reasoning_content` | vLLM docs and llama.cpp server README at their main branches, 08:51 (VV23) | confirmed | The vLLM item is OV73 in the first-wave file, not OV71 |
| 24a. xAI tool cap (W44, C1, OV16) | Workspace documents disagree: 350 against 1,000. OV16 recorded 1,000 | xAI function calling guide, Responses reference, Chat Completions reference, 08:51 (VV24) | conflict | Function calling guide: "max 350 tools per request". References: "A max of 1000 tools are supported" and "A max of 1000 functions are supported" |
| 24b. xAI encrypted reasoning on Responses only (OV14) | Encrypted reasoning exists on the Responses route. Chat Completions "has no field for the ciphertext" | xAI reasoning guide and comparison page, 08:51 (VV24) | confirmed | The Chat Completions reference still lists a plain `reasoning_content` field, against the comparison page's "No reasoning content returned" |

## Claim sections

### OpenAI

All OpenAI pages below were fetched from `developers.openai.com` in their Markdown form on 2026-11-22 between 08:41 and 08:44 Pacific time. Every fetch returned HTTP 200.

#### VV1. Claim 1 (OA35, OA36, OA37): Chat Completions, function tools, and reasoning from GPT-5.4 on

Verdict: `confirmed with a correction`. The first-wave quotes are word for word on the live pages. The correction is to the short form of the claim ("on the GPT-6 models tool calling exists on the Responses API only"): that holds for five GPT-6 models, while GPT-6 Sol and GPT-6 Luna keep function calling on Chat Completions when `reasoning_effort` is `"none"`. OpenAI states all of this as a limitation in prose. The pages print a status code only for the effort value `none`, and they print no error body for the tool case.

What OpenAI says, per model family:

- GPT-5.4, GPT-5.5, GPT-5.6. Migration guide, "Additional differences": "Starting with GPT-5.4, Chat Completions does not support tool calling with `reasoning_effort` values other than `none`." (VV-s1)
- GPT-6 Sol and GPT-6 Luna. Using GPT-6 guide: "GPT-6 Sol and GPT-6 Luna support function calling in Chat Completions only with `reasoning_effort: "none"`. Use Responses for reasoning with tools." (VV-s2). The GPT-6 Sol and GPT-6 Luna model pages repeat it: "Use the Responses API for built-in tools and function calling. Chat Completions supports function calling only with `reasoning_effort` set to `none`." (VV-s8)
- GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra. Using GPT-6 guide: "GPT-6 Astra, GPT-6.1 Sol, and GPT-6.1 Terra support Chat Completions, but tool calling requires Responses." (VV-s2). The GPT-6.1 Sol and GPT-6.1 Terra model pages each say: "Tool calling requires the Responses API." (VV-s8). OpenAI changelog, Sep 3, GPT-6 Astra release: "Tool calling requires the Responses API. If you use tools with Chat Completions, follow the [Responses migration guide](https://developers.openai.com/api/docs/guides/migrate-to-responses)." (VV-s6)
- GPT-6.2 Astra and GPT-6.2 Sol. Using GPT-6.2 guide: "GPT-6.2 Astra and GPT-6.2 Sol support Chat Completions, but tool calling requires Responses." (VV-s3). OpenAI changelog, Nov 17: "Both models support the Responses API and Chat Completions. Tool calling requires the Responses API." (VV-s6)
- Reasoning guide: "Use the [Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses) for function calling. Chat Completions does not support function calling with GPT-6 Astra or GPT-6.1 Sol." (VV-s5)
- Function calling guide, top note: "GPT-6 Astra and [GPT-6.1 Sol](https://developers.openai.com/api/docs/guides/latest-model?model=gpt-6-astra#gpt-61-sol) require the Responses API for tool calling. The Chat Completions examples use GPT-5.6 Terra with reasoning disabled." (VV-s7)

Hard error or documented limitation:

- Every sentence above uses "does not support", "requires", or "only with". None of the 30 OpenAI pages fetched for this file prints an HTTP status or an error message for a Chat Completions request that sends `tools` to one of these models.
- The Chat Completions reference describes `tools` as "A list of tools the model may call. You can provide either [custom tools](/api/docs/guides/function-calling#custom-tools) or [function tools](/api/docs/guides/function-calling)." with no model note, and describes `reasoning_effort` with "Not all reasoning models support every value. See the [reasoning guide](/api/docs/guides/reasoning) for model-specific support." (VV-s9)
- The nearest printed status is for effort, not tools. Reasoning guide: "[GPT-6 Astra](https://developers.openai.com/api/docs/models/gpt-6-astra) does not support `none` reasoning effort. Setting `reasoning.effort` (Responses) or `reasoning_effort` (Chat Completions) to `none` returns HTTP 400." and "[GPT-6.2 Astra](https://developers.openai.com/api/docs/models/gpt-6.2-astra) and [GPT-6.2 Sol](https://developers.openai.com/api/docs/models/gpt-6.2-sol) reject requests that set `reasoning.effort` (Responses) or `reasoning_effort` (Chat Completions) to `none`." (VV-s5)
- So the documents support "documented limitation". Whether the API answers with a 400, and with what words, is unverified here because nothing was run against a live model.

Three details that matter to a gateway that forwards chat completions as written:

- Default effort differs per model, and the default decides whether a request that says nothing about effort falls inside the limitation. Model pages: GPT-5.4 "Reasoning.effort supports: none (default), low, medium, high and xhigh."; GPT-5.5 "none, low, medium (default), high and xhigh"; GPT-5.6 Sol and GPT-5.6 Terra "none, low, medium (default), high, xhigh, and max"; GPT-6 Sol and GPT-6 Luna "`none`, `low`, `medium` (default), `high`, `xhigh`, and `max`" (VV-s8). OpenAI's own Chat Completions tool examples for GPT-5.6 set `reasoning_effort: "none"` explicitly (VV-s1, VV-s7). Inference, not a vendor statement: on a model whose default is `medium`, a chat request with tools that leaves `reasoning_effort` out is at an effort other than `none`.
- The lists of models differ between pages. The reasoning guide and the function calling guide name two models (GPT-6 Astra, GPT-6.1 Sol). The Using GPT-6 guide names three (it adds GPT-6.1 Terra, whose model page agrees). The Using GPT-6.2 guide and the changelog add GPT-6.2 Astra and GPT-6.2 Sol. The longer lists are the newer pages, so the full set today is five models: GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6.2 Astra, GPT-6.2 Sol.
- GPT-6.1 Terra accepts effort `none` (Using GPT-6 guide: "GPT-6.1 Terra, GPT-6 Sol, and GPT-6 Luna support `none`.") and still needs Responses for tools, so on that model setting effort to `none` does not bring tools back on Chat Completions.

OA37 (what a reasoning model loses on Chat Completions): confirmed word for word. Using GPT-5.4 guide: "The biggest difference, and main reason to migrate from Chat Completions to the Responses API for GPT-5.4, is support for passing chain of thought (CoT) between turns." and "Passing CoT exists only in the Responses API, and we've seen improved intelligence, fewer generated reasoning tokens, higher cache hit rates, and lower latency as a result of doing so." (VV-s4)

#### VV2. Claim 2 (OA25): reasoning items return `encrypted_content` by default

Verdict: `confirmed`.

- Responses reference, reasoning item: "`encrypted_content: optional string or null`" with "The encrypted content of the reasoning item. This is populated by default for reasoning items returned by `POST /v1/responses` and WebSocket `response.create` requests." and "When streaming, use the completed reasoning item and its `encrypted_content` from the `response.output_item.done` event in subsequent requests. The `encrypted_content` in `response.output_item.added` may be incomplete. This is especially important when `store` is `false` or when using Zero Data Retention." (VV-s10)
- Reasoning guide: "When you create a response in stateless mode, reasoning items in the response's `output` array include an `encrypted_content` property by default. Stateless mode applies when `store` is `false` or when your organization uses Zero Data Retention (ZDR). The API still accepts the legacy `reasoning.encrypted_content` value in `include` for compatibility, but doesn't require it." (VV-s5)
- Migration guide: "Preserve and replay every returned reasoning item. Each item includes `encrypted_content` by default when you create a response." (VV-s1)
- Note: the reference and the migration guide state the default for every create request. The reasoning guide words it as the stateless-mode default. All three agree for `store: false`. The `include` list in the reference still documents the value `reasoning.encrypted_content` (VV-s10).

#### VV3. Claim 3 (OA27, OA62): dropped reasoning items, `input_transformations`, and `instructions` from GPT-6.2

Verdict: `confirmed` for both.

- Reasoning guide, section "Reasoning item removal across requests": "The API removes reasoning items if you switch model families, downgrade to a less capable model, or switch organizations or users mid-conversation as one of our safeguards against distillation. Starting with 6.2, reasoning items with a `safety_identifier` change or a modified conversation history (the prefix) will also be removed." and "You can use `input_transformations` in the response to diagnose unintended removal:" (VV-s5)
- The table under it, word for word: `prefix_mismatch` "On GPT-6.2 and later, you insert or edit content before the reasoning item. For example, rewriting an earlier tool result changes the prefix. Deleting whole items without editing the remaining items, or appending new items after the reasoning item, does not by itself trigger this reason."; `end_user_mismatch` "On GPT-6.2 and later, the `safety_identifier` on the request that created the item differs from the one on the request that reuses it. This includes supplying an identifier on only one of the two requests. Using the same identifier on both requests, or omitting it from both, passes this check."; `model_mismatch` "You switch model families or move to a less capable model within the same family. Moving to a more capable model within the same family preserves compatible reasoning if the other checks pass."; `organization_mismatch` "You reuse an item from another organization, or from a different workspace under a different user."; `turn_boundary` "The item comes from an earlier turn and the model uses only current-turn reasoning. Models released before GPT-5.6 use current-turn reasoning by default. On supported models, `reasoning.context` controls whether earlier turns are eligible." (VV-s5)
- Responses reference, response object: "`input_transformations: optional array of object { id, action, reason, type }`" with "Reasoning inputs withheld from the model, in input order.", `action: "removed"`, `type: "reasoning"`, and `reason` one of `"organization_mismatch"`, `"end_user_mismatch"`, `"model_mismatch"`, `"prefix_mismatch"`, `"turn_boundary"`. (VV-s10)
- OA62. Using GPT-6.2 guide: "Models starting with GPT-6.2 append top-level `instructions` after `input` in the Responses API. To make reusable instructions eligible for prompt caching, put them in a `developer` message as the first item in `input`." (VV-s3). Prompt caching guide: "In the Responses API, models starting with GPT-6.2 append top-level `instructions` after `input` and bill those tokens at the uncached input rate, even on a cache hit. Put reusable instructions in the first `developer` message in `input`." and "OpenAI bills those instruction tokens at the uncached input-token rate on every request, even if you repeat identical instructions." The same guide says top-level instructions are not kept from one request to the next when a caller chains with `previous_response_id`. (VV-s11)
- Note that widens the first-wave wording a little: the removal also applies to a move "to a less capable model within the same family", and `organization_mismatch` also covers "a different workspace under a different user".

#### VV4. Claim 4 (Q30, Q45, and the surfaces file's usage table): `cache_write_tokens`, `prompt_cache_breakpoint`, and `url_citation` in OpenAI's Chat Completions reference

Verdict: `confirmed`.

- Usage: "`prompt_tokens_details: optional object { audio_tokens, cache_write_tokens, cached_tokens, 2 more }`", "`cache_write_tokens: optional integer (default: 0)`" with "The unadjusted number of prompt tokens written to cache.", and "`cached_tokens: optional integer (default: 0)`" with "Cached tokens present in the prompt." (VV-s9)
- Markers on content parts: "`prompt_cache_breakpoint: optional object { mode, ttl }`" with "Marks the exact end of a reusable prompt prefix. When `ttl` is omitted, the breakpoint inherits the request's `prompt_cache_options.ttl`; the boundary is not rounded to a token block.", "`mode: "explicit" (default: "explicit")`", "`ttl: optional "30m" or "12h"`". The field is on `ChatCompletionContentPartText`, `ChatCompletionContentPartImage`, `ChatCompletionContentPartInputAudio`, and `FileContentPart`. (VV-s9)
- Request level: "`prompt_cache_options: optional object { mode, ttl }`" with "Options for prompt caching. Supported for `gpt-5.6` and later models." and "Each request can write up to four breakpoints." (VV-s9)
- Citations on the assistant message: "`annotations: optional array of object { type, url_citation }`" with "Annotations for the message, when applicable, as when using the [web search tool](/api/docs/guides/tools-web-search).", "`type: "url_citation"`", and "`url_citation: object { end_index, start_index, title, url }`" with "A URL citation when using web search." (VV-s9)
- Note, an inconsistency inside the same reference page: the breakpoint's own `ttl` lists `"30m"` or `"12h"`, while the request-level `prompt_cache_options.ttl` on Chat Completions is "`ttl: optional "30m"`" with "Defaults to `30m`, which is currently the only supported value." The caching guide's model table gives "`"30m"` (default), or `"12h"` in the Responses API" (VV-s11). So on Chat Completions the 12-hour lifetime is listed on the marker and ruled out at request level.
- The chat reference page contains zero occurrences of `phase`, `reasoning_details`, or `reasoning_content` (text search of the fetched page).

#### VV5. Claim 5: `invalid_encrypted_content` against "dropped and reported as `organization_mismatch`"

Verdict: `not confirmed` for the 2026-09-20 workspace claim, and `confirmed` for the first wave's finding.

- What today's documentation says about a reasoning item from another organization: the API removes it and reports it. Reasoning guide: "The API removes reasoning items if you switch model families, downgrade to a less capable model, or switch organizations or users mid-conversation as one of our safeguards against distillation." and the table row `organization_mismatch`: "You reuse an item from another organization, or from a different workspace under a different user." (VV-s5). The reference lists `"organization_mismatch"` as a `reason` inside `input_transformations` with `action: "removed"` (VV-s10).
- The string `invalid_encrypted_content` appears in none of the 30 OpenAI pages fetched for this file. That set includes the error codes guide (VV-s12), the Responses create reference (560 KB), the Responses streaming events reference (2.3 MB), the reasoning guide, the conversation state guide, the migration guide, the keep-the-prefix guide, the data controls guide, and the changelog.
- What stays open: a blob that OpenAI did not mint at all (for example another vendor's signature placed in `encrypted_content`). The pages read describe removal only for OpenAI's own items that fail one of five checks. The outcome for a malformed or foreign blob is not stated anywhere in them.

#### VV6. Claim 6 (OA41, and OA40 for OpenAI's own text): the `phase` field

Verdict: `confirmed`. Note on ids: in the first-wave file OA41 is the OpenRouter item and OA40 is the OpenAI item. The vendor rule checked here is OA40, and every OA40 quote is on the live pages. A point for whoever reads OA41 beside it: OpenAI names GPT-5.3-Codex "and beyond" in the reference, and GPT-5.4 and GPT-5.5 in the guides, while OpenRouter's table lists five model slugs and calls other OpenAI models "Silently ignored".

- Values and where it sits. Responses reference, on `EasyInputMessage` and on the output message: "`phase: optional "commentary" or "final_answer" or null`" with "Labels an `assistant` message as intermediate commentary (`commentary`) or the final answer (`final_answer`)." and "Not used for user messages." (VV-s10)
- Which models and the rule for sending it back. Same paragraph: "For models like `gpt-5.3-codex` and beyond, when sending follow-up requests, preserve and resend phase on all assistant messages" followed by "dropping it can degrade performance." (VV-s10)
- Reasoning guide, section "`phase` parameter": "For long-running or tool-heavy flows with GPT-5.5 and GPT-5.4 in the Responses API, use the assistant message `phase` field to avoid early stopping and other misbehavior." and "`phase` is optional at the API level, but OpenAI recommends using it. Use `phase: "commentary"` for intermediate assistant updates, such as preambles before tool calls, and `phase: "final_answer"` for the completed answer. Don't add `phase` to user messages." and "Using `previous_response_id` is usually the simplest path because prior assistant state is preserved. If you replay assistant history manually, preserve each original `phase` value." and "Missing or dropped `phase` can cause preambles to be treated as final answers in those workflows." (VV-s5)
- Using GPT-5.4 guide: "For GPT-5.4, `gpt-5.3-codex`, and later Responses models, the `phase` field can" (sentence continues on the page) and "`phase` is optional at the API level, but it is highly recommended. Best-effort inference may exist server-side, but explicit round-tripping of `phase` is strictly better." (VV-s4)
- Conversation state guide: "For stateless reasoning-model requests, preserve every item in the response's `output` array. The Responses API returns encrypted reasoning items by default. Replaying the complete output keeps reasoning items and assistant `phase` values intact." (VV-s13)
- Route: every OpenAI mention of `phase` is on the Responses API. The Chat Completions reference has zero occurrences of the word (VV-s9). That matches the reason OpenRouter gives in OA41.
- Strength of the rule: OpenAI words it as a recommendation with a quality cost ("can degrade performance", "can cause preambles to be treated as final answers"), not as a validation error.

### Anthropic

All Anthropic pages below were fetched from `platform.claude.com` and `code.claude.com` in their Markdown form on 2026-11-22 between 08:43 and 08:45 Pacific time. Every fetch returned HTTP 200.

#### VV7. Claim 7 (AN5, AN3): account-bound thinking, and what the prefix check counts

Verdict: `confirmed` for both.

AN5, which models and what "account" means:

- Preserved thinking page: "Thinking blocks from Claude Fable 5.2, Claude Mythos 5.2, Claude Sonnet 5.5, and Claude Haiku 5.5 work only in the account that produced them or a linked one. In any other account, the API drops them, with no error." (VV-s20)
- Same page, table "What counts as your account on each platform": "Claude API, Claude Platform on AWS, Claude Models on Google Cloud (Anthropic managed), or Microsoft Foundry | Your organization", "Amazon Bedrock | Your AWS account", "Claude Models on Google Cloud (Google managed) | Your Google Cloud project". (VV-s20)
- So on the Claude API the account is the organization. The page does not use the workspace or the API key as the unit.
- One more sentence on the same page that the first wave did not quote: "The API can also leave out a thinking block that another account produced, if that account is no longer in good standing. This applies to requests to Claude Fable and Claude Mythos models, Claude Opus 5, Claude Opus 5.5, Claude Sonnet 5.5, and Claude Haiku 5.5. The request still succeeds, and the model answers without that block's reasoning." (VV-s20, apostrophe typed plain here)
- Model switches: "A thinking block that the new model or account can't use never causes an error. The API drops it for that request and doesn't bill it, and Claude answers without that reasoning." (VV-s20, apostrophes typed plain here)

AN3, the prefix check:

- What it compares: "A prefix check fails if `system`, `tools`, or an earlier message changed after Claude produced a block. It also fails if a thinking block between others was left out or moved." (VV-s21)
- The sentence the task asked about, word for word: "A change can be as small as one word of a tool description. Changing, inserting, moving, or deleting an earlier message counts, whatever its role. So does changing text before a thinking block in its own message." (VV-s21)
- Which models, with `prefix_mismatch_behavior` unset, table "What each model does when the prefix check fails": Claude Fable 5.2 "400 error", Claude Mythos 5.2 "400 error", Claude Mythos 5.1 "400 error", Claude Fable 5.1 "Possible 400 error", Claude Opus 5.5 "Possible 400 error", Claude Sonnet 5.5 "Possible 400 error", Claude Haiku 5.5 "Possible 400 error". (VV-s21)
- What "possible" means: "Where the table says "Possible 400 error," there are two cases: a request that fails the prefix check might get a 400 error or no error. When no error is returned, the failing blocks reach the model." (VV-s21, quotation marks typed plain here)
- The error text: "the 400 error says that a failing block `is bound to a different conversation`." The preserved thinking page gives the full string as `The block is bound to a different conversation`. (VV-s21, VV-s20)
- Scope of the check: "The prefix check looks only at content, not at sessions, IDs, or users. The same content in a new request or a fork passes it." and "On the Claude API, other models, such as Claude Opus 5, don't run it, and it doesn't cover their blocks." (VV-s21, apostrophes typed plain here)
- What does not count, table rows: "`model`, `effort`, `max_tokens`, `tool_choice`, `metadata`, `thinking.display`, and other parameters", "JSON formatting or key order", "`cache_control` markers", "A tool with `defer_loading: true` that nothing has referred to". (VV-s21)
- The setting: "The [beta header](https://platform.claude.com/docs/en/api/beta-headers#beta-features-on-other-platforms) `thinking-binding-controls-2026-08-01` adds the setting `thinking.block_binding.prefix_mismatch_behavior` and the response's `input_transformations` array." Values `"error"`, `"drop_block"`, and unset. (VV-s21)
- Batches: "[Message Batches items] that leave `prefix_mismatch_behavior` unset get no error, and the API can drop the failing blocks." (VV-s21, link target omitted)

#### VV8. Claim 8: the `compaction` stop reason and `pause_turn`

Verdict: `confirmed`, with one added fact about where `compaction` is listed.

- The stable Messages reference lists seven values in its `stop_reason` enum: `"end_turn"`, `"max_tokens"`, `"stop_sequence"`, `"tool_use"`, `"pause_turn"`, `"refusal"`, `"model_context_window_exceeded"`. (VV-s22)
- The stop reasons guide's quick reference table has the same seven rows. (VV-s23)
- The beta Messages reference (`BetaStopReason`) lists eight enum values: the seven above plus `"compaction"`, placed between `"pause_turn"` and `"refusal"`. Its prose list above the enum still describes seven. The same reference says of its `compaction` request parameter: "the response holds only the resulting `compaction` block (`stop_reason` `"compaction"`)". (VV-s24)
- The compaction guide (beta header `compact-2026-01-12`): "When enabled, the API returns a message with the `compaction` stop reason after generating the compaction block:" and "If safeguards block the summary, `content` is also `null` and the stop reason is `refusal` instead of `compaction`." and the advice "look for the `compaction` block instead of testing `stop_reason`". (VV-s25)
- So `compaction` is an eighth value that exists in the beta schema and the compaction guide, beside the seven in the stable schema. The first wave credited it to the guide only. The beta reference enum is the stronger source.
- `pause_turn`, the reference: "`"pause_turn"`: we paused a long-running turn. You may provide the response back as-is in a subsequent request to let the model continue." (VV-s22)
- `pause_turn`, the stop reasons guide table: "A server-tool loop reached its iteration limit." and "Send the assistant content back to continue." Also: "the API might return `pause_turn` if the server-side sampling loop reaches its iteration limit (default 10). Handle this by adding the paused response to your messages unchanged and sending the request again." (VV-s23)
- `pause_turn`, the server tools page, what the caller sends: "**Continue the conversation:** Pass the paused response back as-is in a subsequent request to let Claude continue its turn." / "**Preserve tool state:** Include the same tools in the continuation request. A paused turn can end with a `server_tool_use` block whose tool has not run yet, and the API returns a validation error if that tool is missing from the continuation." / "**Keep the container:** If the response includes a `container`, as it does with the [code execution tool](https://platform.claude.com/docs/en/agents-and-tools/tool-use/code-execution-tool), send its ID back in the continuation request." / "**Repeat as needed:** A continued turn can pause again." and "Keep every paused response in `messages`, in order. A continuation holds only new content, so a request that leaves out an earlier paused response loses that work and can return a 400 error." (VV-s26)
- What it is not: "When Claude stops to call a tool that you run, the stop reason is [`tool_use`](https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons#tool-use), never `pause_turn`." (VV-s23)
- So the caller continues a paused turn by appending the paused assistant message unchanged as the last message and sending the same request again (same tools, the container id if one came back), with no new user message.

#### VV9. Claim 9 (T48): models that reject a forced `tool_choice`, and the error text

Verdict: `confirmed`, and the error text is now quoted from Anthropic's errors page (T48 did not quote it).

- The list: "Claude Fable 5.2, [Claude Mythos 5.2](https://platform.claude.com/docs/en/models/mythos-5-2/overview), Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1, and [Claude Mythos 5.1](https://platform.claude.com/docs/en/models/mythos-5-1/overview) | `any` and `tool` return a 400 error. See [Forced tool use not supported](https://platform.claude.com/docs/en/api/errors#forced-tool-use-not-supported)". (VV-s27)
- The errors page, section "Forced tool use not supported": "Claude Fable 5.2, [Claude Mythos 5.2](https://platform.claude.com/docs/en/models/mythos-5-2/overview), Claude Opus 5.5, Claude Sonnet 5.5, Claude Fable 5.1, and [Claude Mythos 5.1](https://platform.claude.com/docs/en/models/mythos-5-1/overview) don't support forced tool use. Sending `tool_choice: {"type": "any"}` or `tool_choice: {"type": "tool", "name": "..."}` to any of these models, including on the [token counting endpoint](https://platform.claude.com/docs/en/build-with-claude/token-counting), returns a 400 `invalid_request_error`:" followed by the message `tool_choice: type "tool" and "any" are not supported for this model.` (VV-s28, apostrophe typed plain here)
- Same section: "`tool_choice: {"type": "auto"}` (the default) and `{"type": "none"}` are accepted." (VV-s28)
- Manual extended thinking row: "Manual [extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) (`thinking: {type: "enabled"}`) | `any` and `tool` are not supported and result in an error". (VV-s27)
- Which models still accept it: "models with thinking on by default, such as Claude Opus 5 and Claude Haiku 5.5, accept it, but a forced call skips thinking: the response starts with the tool call and has no `thinking` block". (VV-s27)
- Note: Claude Haiku 5.5 is on the prefix-check list (VV7) and is not on this list. The two lists differ by that one model.

#### VV10. Claim 10 (T51): structured outputs without a beta header

Verdict: `confirmed`.

- "The `output_format` parameter has moved to `output_config.format`, and beta headers are no longer required. The `output_format` parameter is deprecated. To use it anyway, add the `structured-outputs-2025-11-13` beta header. Without it, the API returns a 400 error." (VV-s29)
- "Structured outputs are available on every model and eligible for HIPAA readiness." (VV-s29)
- The Claude Code gateway page adds a caveat for upstreams that are not the Claude API. Its row for effort and structured outputs says the `output_config` body field holds effort, structured-output format, and task budget settings, that "each pairs with its own beta header", and that the failure is "`400` naming `output_config`, often `Extra inputs are not permitted`, on Amazon Bedrock and Google Cloud's Agent Platform upstreams". (VV-s30). So "no beta header" is the rule on the Claude API, and Claude Code still sends paired beta values that matter on other platforms.

#### VV11. Claim 11: `temperature`, `top_p`, `top_k` removed under API version `2026-10-21`

Verdict: `confirmed`.

- Versioning page, under `2026-10-21`: "The `temperature`, `top_p`, and `top_k` request parameters are removed. A request that sends any of them returns a 400 error on every model." (VV-s31)
- Migration page: "In `2026-10-21`, a request that sends `temperature`, `top_p`, or `top_k` returns a 400 error, even at the parameter's default value." and "Models that already accept only the defaults, such as Claude 4.7 and later models, sample as before. On models that accept other values, such as Claude Sonnet 4.6, a request that lowered `temperature` for more consistent output loses that effect. No other setting replaces it. If a request relies on a non-default value, either stop sending the parameter, or keep that request on `2023-06-01`." (VV-s32, apostrophe typed plain here)
- What the older version does on newer models, from the thinking page: "On Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, Claude Mythos 5.1, Claude Fable 5, Claude Mythos 5, Claude Mythos Preview, Claude Opus 5.5, Claude Opus 5, Claude Opus 4.8, Claude Opus 4.7, Claude Sonnet 5.5, Claude Sonnet 5, and Claude Haiku 5.5, non-default `temperature`, `top_p`, or `top_k` values return a 400 error on every request, regardless of whether thinking is used. On these models, the `top_p` default is `0.99`, so a `top_p` of `1` returns a 400 error, and so does a request that includes both `temperature` and `top_p`, even at their defaults." (VV-s33)
- Same paragraph, older models on older versions: "On older models in API versions before `2026-10-21`, the restriction applies only while thinking is on: `temperature` and `top_k` are incompatible with thinking, and `top_p` is allowed at values between 0.95 and 1." (VV-s33)
- Availability: "On Amazon Bedrock, it's available for Claude 4.7 and later models. For earlier Claude models on Bedrock, [keep the API version that you use today](https://platform.claude.com/docs/en/api/migration-2026-10-21#availability)." (VV-s31, apostrophe typed plain here)
- The stable Messages reference fetched today contains zero occurrences of `temperature`, `top_p`, or `top_k` (text search of the page). (VV-s22)
- A detail that sharpens the first wave's summary ("Under `2023-06-01`, 4.7 and later reject non-default values"): on those models under the older version, `top_p: 1` is itself a non-default value (the default is `0.99`), and sending `temperature` together with `top_p` is a 400 even when both are at their defaults.

#### VV12. Claim 12 (AN83): Anthropic's guidance to gateway authors

Verdict: `confirmed`. The page has a section titled "Forward as open lists".

- Section "Forward as open lists": "When forwarding to an Anthropic-format upstream, pass `anthropic-*` request headers and request body fields through unchanged rather than allowlisting the ones you see today. A gateway pinned to an observed list strips the next capability's header or field and breaks it on the release that introduces it." and "The exception is a non-Anthropic upstream such as Amazon Bedrock or Google Cloud's Agent Platform, where bridging the schema difference is the gateway's job; see [feature pass-through](#feature-pass-through)." (VV-s30)
- Header table, `anthropic-beta`: "Comma-separated capability values for the request. Forward the header verbatim; don't allowlist individual values, because the set changes with Claude Code releases." (VV-s30)
- "Capabilities that add body fields pair them with a beta header, and the pair travels together. A gateway that strips the header while passing the body, or forwards an Anthropic-format body to an upstream with a different schema, produces `400` errors." (VV-s30)
- "When the client speaks the Anthropic Messages format, Claude Code sends the full set, even if your gateway forwards to an Amazon Bedrock or Google Cloud's Agent Platform upstream" and "Bridging that difference is your gateway's job." (VV-s30)
- "Most of this retry logic matches on the upstream's error wording, so forward error response bodies unmodified. A gateway that wraps upstream errors in its own envelope breaks the recovery path, even when it preserves the status code," with one exception the page gives: an envelope whose message has a stable `capability_rejected:` token. (VV-s30)
- Prompt caching row: "Forward `cache_control` unchanged wherever it appears, and don't convert block-form `system` or message content to plain strings". (VV-s30)
- Scope note: this page is written for gateways that serve Claude Code over the Anthropic Messages format. It is guidance for a pass-through gateway, and it names translation to a different upstream schema as the gateway's own job.

### Google

All Google pages below were fetched on 2026-11-22 between 08:46 and 08:49 Pacific time. The AI Studio pages were fetched in Markdown form from `aistudio.google.com` (the `ai.google.dev/gemini-api/docs/...` addresses redirect there). The `generateContent` variant of a guide is the same address with `?generate-content=true`. The Vertex page was fetched as HTML from `docs.cloud.google.com` and has the stamp "Last updated 2026-11-20 UTC."

#### VV13. Claim 13 (GG2 to GG6): the thought signature rule, the HTTP 400, and where `missing_thought_signature` appears

Verdict: `confirmed` for GG2 to GG6. The local survey's wording ("omitting it is a 400 with code `missing_thought_signature`") is `not confirmed`.

The rule (AI Studio thought signatures page, `generateContent` variant, VV-s40):

- "**When using Gemini 4 and Gemini 3 models, you must pass back thought signatures during function calling, otherwise you will get a validation error** (4xx status code). This includes when using the `minimal` [thinking level](/docs/thinking#thinking-levels) setting for Gemini 3 Flash."
- "**Parallel Function Calls**: If the model generates parallel function calls in a response, the `thought_signature` is attached **only to the first** `functionCall` part. Subsequent `functionCall` parts in the same response will **not** contain a signature."
- "**Requirement**: You **must** return this signature in the exact part where it was received when sending the conversation history back."
- "**Validation**: Strict validation is enforced for all function calls within the current turn . (Only current turn is required; we don't validate on previous turns)"
- "The API goes back in the history (newest to oldest) to find the most recent **User** message that contains standard content (e.g., `text`) (which would be the start of the current turn). This will not **be** a `functionResponse`."
- "The **first** `functionCall` part in **each step** of the current turn **must** include its `thought_signature`."
- "If you omit a `thought_signature` for the first `functionCall` part in any step of the current turn, the request will fail with a 400 error."

The HTTP 400 and its message (same page): "Gemini 3 models: Failure to include signatures will result in a 400 error. The verbiage will be of the form:" then "Function call `<Function Call>` in the `<index of contents array>` content block is missing a `thought_signature`. For example, *Function call `FC1` in the `1.` content block is missing a `thought_signature`.*" (VV-s40)

Where the token `missing_thought_signature` appears:

- `generateContent` API reference, `Candidate.finishReason` enum: "`MISSING_THOUGHT_SIGNATURE`: Request has at least one thought signature missing." It sits in the list of finish reasons between `TOO_MANY_TOOL_CALLS` and `MALFORMED_RESPONSE`. (VV-s41)
- Interactions API errors page, table "Generation error codes", which the page introduces with "These error codes indicate a structural issue with the model's generated output (such as a malformed function call or an undeclared tool call).": "| `missing_thought_signature` | The response is missing a required thought signature. |" (VV-s42)
- The same Interactions page has a separate table, "Standard API error codes", where each code has an HTTP status (`invalid_request` 400, `failed_precondition` 400, `parameter_unknown` 400, and so on). `missing_thought_signature` is not in that table. It is listed with `malformed_function_call`, `unexpected_tool_call`, `no_image`, and `too_many_tool_calls`, which are the lower-case forms of `generateContent` finish reasons. (VV-s42)
- `generateContent` errors page: its HTTP table has one 400 row for a bad body, "| 400 | INVALID_ARGUMENT | The request body is malformed. |", and the error body format is `"code": 400`, `"status": "INVALID_ARGUMENT"`, with the fields described as "`code` | integer | The HTTP status code." and "`status` | string | The gRPC status code in `SCREAMING_CASE`." The token `missing_thought_signature` is absent from this page. (VV-s43)

Settled reading: on `generateContent`, a missing signature on the first function call part of a step in the current turn is an HTTP 400 whose machine-readable fields are the integer `400` and the status `INVALID_ARGUMENT`, and whose message has the "is missing a `thought_signature`" form. `MISSING_THOUGHT_SIGNATURE` is a finish reason on `generateContent` and a generation error code on the Interactions API. It is not the code of the HTTP 400. The first wave is right and the local survey's phrase is wrong on this point. The two Google descriptions also differ: the `generateContent` enum says the request lacks a signature, the Interactions table says the response lacks one.

#### VV14. Claim 14 (GG7, GG13): signatures on text parts, and tool combination

Verdict: `confirmed`.

- Text and other parts (VV-s40): "**Behavior**: The final content part (`text, inlineData…`) returned by the model may contain a `thought_signature`." / "**Recommendation**: Returning these signatures is **recommended** to ensure the model maintains high-quality reasoning, especially for complex instruction following or simulated agentic workflows." / "**Validation**: The API does **not** strictly enforce validation. You won't receive a blocking error if you omit them, though performance may degrade."
- Thinking page: "**Gemini 4 and Gemini 3 models** may return thought signatures for all types of [parts](/api-reference/generate-content#Resource:Content). We recommend you always pass all signatures back as received, but it's *required* for function calling signatures." (VV-s44)
- Tool combination page (VV-s45): "You must set the `include_server_side_tool_invocations` flag to `true` to enable tool context circulation." / "In a single response, the API returns the `toolCall` and `toolResponse` parts for the built-in tool call." / "You must return all parts, including all the [fields](/docs/tool-combination#critical-fields) they contain, back to the model on each turn to maintain context and enable tool combinations." / "`thought_signature`: The actual encrypted context embedded in **each part returned by the API**. Context can't be reconstructed without thought signatures; if you don't return the thought signatures for all parts in every turn, the model will error out." / "Found in *all* parts."
- Same page, two more fields that must come back with the parts: "`id`: A unique identifier that maps a call to its response." ("You *must* provide the same `id` in the function response that the API provides in the function call.") and "`tool_type`: Identifies the specific tool being used". (VV-s45)

#### VV15. Claim 15 (GG8): the dummy signature values

Verdict: `confirmed`. Two strings on the AI Studio page, one on the Vertex page.

- AI Studio, FAQ 1: "While injecting custom function call blocks into the request is strongly discouraged, in cases where it can't be avoided, e.g. providing information to the model on function calls and responses that were executed deterministically by the client, or transferring a trace from a different model that does not include thought signatures, you can set the following dummy signatures of either `"context_engineering_is_the_way_to_go"` or `"skip_thought_signature_validator"` in the thought signature field to skip validation." (VV-s40)
- Vertex: "There are rare cases where you need to provide functionCall parts that were not generated by the API and therefore don't have an associated thought signature (for example, when transferring history from a model that does not include thought signatures). You can set thought_signature to skip_thought_signature_validator, but, this should be a last resort as it will negatively impact model performance." (VV-s46, read from the HTML page with tags removed, so code formatting is not shown)

#### VV16. Claim 16 (GG25, GG98): signatures and `cached_content` on Google's OpenAI-compatible route

Verdict: `confirmed`, with one detail on the exact nesting of `cached_content`.

- Vertex page, Chat Completions example, model response: `"tool_calls": [ { "extra_content": { "google": { "thought_signature": "<SIGNATURE_A>" } }, "function": { "arguments": "{\"flight\":\"AA100\"}", "name": "check_flight" }, "id": "function-call-1", "type": "function" } ]`. The caller's next request repeats that assistant entry with `extra_content` in place, followed by `{"role": "tool", "name": "check_flight", "tool_call_id": "function-call-1", "content": "..."}`. (VV-s46)
- Vertex page, parallel example: the first `tool_calls` entry has `extra_content.google.thought_signature`, and the second entry has only `function`, `id`, and `type`. Caption: "You must preserve <SIGNATURE_A> on the first part exactly as received." (VV-s46)
- AI Studio: the OpenAI compatibility page says "Gemini 3 supports OpenAI compatibility for thought signatures in chat completion APIs. You can find the full example on the [thought signatures](/docs/thought-signatures?generate-content=true#openai) page." (VV-s47). The section it points to ("Signatures for OpenAI compatibility") shows only native-shaped JSON today (`"functionCall"`, `"thoughtSignature"`). The string `extra_content` has zero occurrences in the AI Studio thought signatures page, OpenAI compatibility page, thinking page, function calling page, and both `generateContent` reference pages fetched today, and it is on seven lines of the Vertex page. So the field path is documented by Google Cloud and is only pointed at by AI Studio. This matches GG26.
- `cached_content` (AI Studio OpenAI compatibility page, section "Enable Gemini features with `extra_body`"): "| **`cached_content`** | Text | Chat | Corresponds to Gemini's general content cache. |" and "| **`thinking_config`** | Object | Chat | Corresponds to Gemini's ThinkingConfig. |" (VV-s47)
- The nesting in Google's own example is three levels in the Python SDK call: `extra_body={ 'extra_body': { 'google': { 'cached_content': "cachedContents/0000aaaa1111bbbb2222cccc3333dddd4444eeee" } } }`. On the wire the JSON body has a top-level key named `extra_body` that holds `google`. Google's own REST sample for the same route shows it: `"extra_body": { "google": { "thinking_config": { "thinking_level": "high", "include_thoughts": true } } }` sits beside `"model"` and `"messages"` in the request body. (VV-s47)
- Caching guide: "If you're using an [OpenAI library](/docs/openai), you can enable explicit caching using the `cached_content` property on [`extra_body`](/docs/openai#extra-body)." (VV-s48)
- What the route lacks, long decode guide: "**OpenAI compatibility endpoint:** Long decode is not available. Output stops at the per-request cap." (VV-s49)

#### VV17. Claim 17 (GG52 and the Gemini 4 caching items, GG41): explicit caching on Gemini 4 and Interactions, and the continuation token

Verdict: `confirmed` for both.

- Caching guide, Interactions variant: "The **Interactions API** only supports implicit caching. Explicit caching (manually creating and managing cache objects) is not supported in the Interactions API, or for Gemini 4 models. To use explicit caching on supported Gemini 3 and 2.5 models, switch to the [generateContent API](/docs/caching?generate-content=true)." (VV-s50)
- Caching guide, `generateContent` variant: "Explicit caching (can be manually enabled on Gemini 3 and 2.5 models, cost saving guarantee; not supported for Gemini 4 models)" and "Explicit caching is not supported for Gemini 4 models; Gemini 4 models use [implicit caching](/docs/caching#implicit-caching) by default." (VV-s48)
- Gemini 4 Carbon migration list: "**Move off explicit context caching:** Explicit caching (`cachedContents` / `caches.create`) is not supported on Gemini 4 Carbon and returns an error. Remove calls that create or pass `cached_content` in `GenerateContentConfig`." (VV-s51)
- Note on ids: in the first-wave file the heading of GG52 is about named caches in general, and these three quotes sit in the items right after it. The content is confirmed whichever id it is filed under.
- Long decode guide: "**A cap on each hop:** One hop returns up to 65,536 output tokens (or up to 32,768 on direct non-streaming REST calls to `gemini-4-carbon` and `gemini-4-argon`)." / "**A continuation signal:** If the model reaches the cap and has tokens left, the response ends with `finish_reason: CONTINUATION` and a `continuation_token`." / "**One total for all hops:** `max_output_tokens` sets the total thinking and output tokens for all hops together." / "**A stop signal:** When the model uses all of `max_output_tokens`, the response ends with `finish_reason: MAX_TOKENS`." (VV-s49)
- Field names on the wire: the guide's prose and Python use `continuation_token`. Its REST and JavaScript samples, and the API reference, use `continuationToken`. Reference, request field: "**continuationToken**: `string`" with "Optional. An opaque continuation token used to resume generation from a previous `GenerateContent` or `StreamGenerateContent` response that stopped with `finish_reason` set to `CONTINUATION`. Note that `contents` is mandatory and must identically replicate the prompt and conversation history from the initial request when continuation token is provided in a request." Reference, candidate field: "An optional opaque continuation token returned when `finish_reason` is `CONTINUATION`." (VV-s41)
- Reference, finish reason: "`CONTINUATION`: Token generation was stopped because the response reached the per-request token limit, but generation is not yet complete. The model response can be continued via a subsequent request using the returned `continuation_token`. Note, the `max_output_tokens` is cumulative across continuation hops and must be monotonically increased or omitted." (VV-s41)
- A wording difference between two Google pages on one point: the guide's loop rule 1 says "Keep `max_output_tokens` the same.", and the reference says it "must be monotonically increased or omitted". Both agree that the value is one running total and that it must not go down.
- The Interactions API uses different names for the same thing: status `requires_continuation`, with `previous_interaction_id` and `continuation_token` on the next request. (VV-s52)

#### VV18. Claim 18: are signatures portable between the Gemini API (AI Studio) and Vertex AI?

Verdict: `not confirmed` as a stated rule in either direction. Neither Google site names the AI Studio and Vertex pair. The Vertex page does state the scope a signature is valid in, and that statement was not in the first-wave file.

- Vertex page, "Constraints and limits": "Conversation, model, and project scope: Return thought signatures unmodified within the same conversation history, model, and Google Cloud project. Modifying earlier turns in contents, changing system instructions or tool declarations mid-conversation, or passing signatures across incompatible models or projects causes the service to omit mismatched thoughts from the model's context rather than failing the request. However, omitting a required thought_signature on a functionCall part in the current turn on Gemini 3 and later models returns a 400 error." (VV-s46, tags removed)
- Same section: "No expiration or TTL: Thought signatures don't expire and have no platform-enforced time to live (TTL) or validity window." and "Store thought_signature values as variable-length opaque payloads (such as unbounded text or binary columns), and don't truncate, split, merge, or modify them. Sending a truncated or corrupted signature returns a 400 (INVALID_ARGUMENT) error." (VV-s46)
- Vertex page, "Thought signatures from other models": "Thought signatures are compatible only with the model family and tier that generated them. For example, Gemini Flash-Lite models can't process thoughts generated by Gemini Flash or Gemini Pro models, and earlier model families can't process thoughts generated by later model families. If a request includes thought signatures from an incompatible model, the request still succeeds, but the service omits incompatible thoughts from the model's context and preserves visible text and tool calls." (VV-s46)
- The AI Studio thought signatures page, thinking page, and function calling page contain no sentence about projects, Vertex, or moving signatures between endpoints (text search of the three pages fetched today).
- What follows from Google's own words: a signature is scoped to the conversation history, the model, and the Google Cloud project. A signature that reaches a different project is described as omitted from the model's context while the request succeeds, and a signature that is damaged is described as a 400. Whether an AI Studio key and a Vertex endpoint in the same Google Cloud project count as the same scope is left unstated. LiteLLM's code comment that the move fails with a 400 (first-wave item GG29) is a third-party account and was not re-read here.
- This section also shows a prefix rule on Vertex that the first wave did not record: editing earlier turns, system instructions, or tool declarations mid-conversation makes the service drop the mismatched thoughts without an error.

### Other vendors and local servers

All pages below were fetched on 2026-11-22 between 08:51 and 08:54 Pacific time. Every fetch returned HTTP 200 unless stated.

#### VV19. Claim 19 (OV22, OV23): DeepSeek and `reasoning_content` when `tools` is present

Verdict: `confirmed`. The vendor page states the status 400. It does not print the error message, which is what OV23 recorded.

- Thinking mode guide: "If the request carries the tools parameter: the reasoning_content of all previous turns should be passed back to the API and will be concatenated into the context." (VV-s60, read from HTML with tags removed, so code formatting is not shown)
- "If the request does not carry the tools parameter: reasoning_content does not need to be passed back; even if passed to the API, it will be ignored and will not be concatenated into the context." (VV-s60)
- "Please note that for requests carrying the tools parameter, the reasoning_content must be fully passed back to the API in all subsequent requests" and, after a dash in the same sentence, "even for turns where the model did not perform a tool call. If your code does not correctly pass back reasoning_content, the API will return a 400 error." (VV-s60)
- "Additionally, in the Turn 2 request, we still pass the reasoning_content generated in Turn 1 to the API." (VV-s60)
- Error text: the thinking mode guide, the tool calls guide, the chat completion reference, and the error codes page were searched for "must be passed back" and for "in the thinking mode". The only hit is an unrelated sentence in the tool calls guide ("From DeepSeek-V3.2, the API supports tool use in the thinking mode."). The error codes page lists "400 - Invalid Format" with "Cause: Invalid request body format." and "422 - Invalid Parameters". (VV-s61, VV-s62, VV-s63)
- So the string "The reasoning_content in the thinking mode must be passed back to the API." rests on LiteLLM's docstring and two third-party reports, as OV23 says. It was not re-checked here.

#### VV20. Claim 20 (OV30, OV35): Mistral tool-call ids and the two chat endpoints

Verdict: OV30 `confirmed with a correction`. OV35 `confirmed`.

OV30:

- Mistral docs, fine-tuning data format: "Both `"id"` and `"tool_call_id"` are randomly generated strings of exactly 9 characters." (VV-s64). This sentence is about fine-tuning data. It says "9 characters" and does not say "alphanumeric".
- The alphanumeric rule and the error text are in Mistral's own open-source validator, `mistralai/mistral-common`, file `src/mistral_common/protocol/instruct/validator.py` on `main` (latest commit touching the file: `fc079a099f6c2a6186c9829fe4405ee589aac576`, 2026-11-18): line 50 `_TOOL_CALL_ID_REGEX = re.compile(r"^[a-zA-Z0-9]{9}$")`, and in `MistralRequestValidatorV3` lines 550 to 553 and 587 to 590 the messages `f"Tool call id was {message.tool_call_id} but must be a-z, A-Z, 0-9, with a length of 9."` and `f"Tool call id was {tool_call.id} but must be a-z, A-Z, 0-9, with a length of 9."`. That wording matches the third-party error bodies OV30 quoted. (VV-s65)
- The correction: the same file relaxes the rule for newer tokenizer versions. Lines 51 to 53: `_INVALID_TOOL_CALL_ID_MESSAGE = ( f"Tool call id must be a non-empty string other than '{_NULL_TOOL_CALL_ID}' for tokenizer version 13 or newer." )`. Lines 738 to 747: `class MistralRequestValidatorV13(MistralRequestValidatorV11):` overrides both id checks with `if not tool_call.id or tool_call.id == _NULL_TOOL_CALL_ID: raise InvalidFunctionCallException(_INVALID_TOOL_CALL_ID_MESSAGE)`. The factory at lines 782 to 793 maps `TokenizerVersion.v3`, `v7`, and `v11` to validators that keep the nine-character check, and `v13` and `v15` to validators that accept any non-empty id other than `"null"`. (VV-s65)
- So in Mistral's own validator the nine-alphanumeric rule holds for tokenizer versions 3 to 11, and from version 13 on the rule is "a non-empty string other than 'null'". Which hosted models use tokenizer version 13 or newer is not stated in the pages read, and the hosted API's behavior was not tested. A translator that always rewrites ids to nine alphanumeric characters satisfies both rules.
- The error glossary and code 3280 were not re-read here. The number 3280 rests on the third-party reports.

OV35:

- "The OpenAI-compatible endpoint is in public preview. Its behavior, supported parameters, and error responses can change during the preview." (VV-s64)
- "Use the OpenAI-compatible endpoint to call Mistral-hosted models from the OpenAI SDK or another OpenAI-compatible client. The endpoint is available at `https://api.mistral.ai/openai/v1/chat/completions` and uses the OpenAI Chat Completions request and response format. The standard `https://api.mistral.ai/v1/chat/completions` endpoint uses the native Mistral format and works with the Mistral SDKs." (VV-s64)
- "To use the OpenAI SDK, set its base URL to `https://api.mistral.ai/openai/v1` and use your Mistral API key:" (VV-s64)

#### VV21. Claim 21 (T57, and OV40 for the replay rule): Kimi K3 tool-call ids and `reasoning_content`

Verdict: `confirmed` for both.

- Moonshot's `encoding_k3.py` in the `moonshotai/Kimi-K3` repository on Hugging Face (repository revision `1f5d44f794ef598de7b6f529ef7923729b7e5408`, last modified 2026-11-20), lines 393 to 397: tool messages are "matched against the most recent preceding assistant ``tool_calls`` by opaque ``tool_call_id`` == ``tool_calls[].id`` (K3 drops the ``func:index`` format requirement) and sorted by the matched 1-based position." (VV-s66)
- Hosted platform, official tools guide: "If the assistant message has `tool_calls`, the next messages must be exactly the same `role=tool` messages as the `tool_calls`, and `tool_call_id` must be aligned one-to-one with the previous `tool_calls.id`:", "If there are multiple `tool_calls`, the order is not sensitive;", "The uniqueness requirement is only local to the `tool_calls`-response in this round, not for the entire conversation or globally.", and "The model may return more than one `tool_calls`; you must return results for all `tool_calls` for the model to continue, otherwise the request will be considered invalid and rejected;". Example id: `"id": "web_search:0"`. (VV-s67)
- Hosted platform, chat reference: example id `"id": "get_weather_0_4d9e2559"` and "The `tool_call_id` must match the tool-call `id` in the assistant message:". (VV-s68)
- The string `functions.` has zero occurrences in the four Kimi platform pages fetched (chat reference, thinking guide, K3 quickstart, official tools guide).
- The replay rule. Thinking guide: "In both single-task tool-call loops and multi-turn conversations, pass the complete assistant message back as-is, including `reasoning_content`. `kimi-k3.1-pro` and `kimi-k3` always enable Preserved Thinking." and "`kimi-k3.1-pro` and `kimi-k3` always enable Preserved Thinking. In multi-turn conversations and tool calls, append the complete assistant message returned by the API to `messages` as-is, including `reasoning_content`; do not keep only the final answer text." and "For K3.1 Pro and K3, multi-turn conversations and tool calls must pass the complete assistant message returned by the API back in `messages` as-is." (VV-s69)
- K3 quickstart: "Return the complete assistant message unchanged in multi-turn conversations and tool calls." and "For multi-turn conversations and tool calls, add the complete assistant message returned by the API to the next request. Do not keep only `content`." (VV-s70)
- Older models, chat reference table: `thinking.keep` on `kimi-k2.6` "Defaults to `null`, which discards `reasoning_content` from previous turns. Pass `"all"` to enable Preserved Thinking."; on `kimi-k2.7-code` and `kimi-k2.7-code-highspeed` "The model preserves `reasoning_content` from previous turns whether you omit this field, pass `null`, or pass `"all"`. Other values return an error." (VV-s68)
- The Kimi pages state the rule with "must" and print no status code or error string for a request that leaves `reasoning_content` out.

#### VV22. Claim 22 (OV64, OV66): Meta `tool_choice`, and OpenRouter's endpoint data

Verdict: OV64 `confirmed`. The conflict OV recorded is `confirmed`, with one addition: OpenRouter's data differs between Meta models.

- Meta tool calling page, "Guidelines": "**`tool_choice` must be `"auto"`**: Only `"auto"` (the default) is supported on both Chat Completions and the [Responses API](https://dev.meta.ai/docs/protocols/responses); `"none"`, `"required"`, and named function choices return `HTTP 400` ("only `"auto"` is supported for `tool_choice`"). With `"auto"`, the model decides whether to call a tool, so your app should handle both a tool call and a plain text response." (VV-s71)
- OpenRouter, `GET https://openrouter.ai/api/v1/models/meta/muse-spark-1.3/endpoints` (no key, HTTP 200, 08:53 Pacific): two endpoints. Provider `Meta`, tag `meta`: `"supports_tool_choice": {"none": true, "auto": true, "required": true, "function": true}`, and `tool_choice` is in its `supported_parameters`. Provider `Google`, tag `google-vertex/global`: the same four values. (VV-s72)
- `meta/muse-wonder/endpoints`: provider `Meta`, tag `meta`: `{"none": true, "auto": true, "required": true, "function": true}`. (VV-s72)
- `meta/muse-ember/endpoints`: provider `Meta`, tag `meta`: `{"none": false, "auto": true, "required": false, "function": false}`, and the Google endpoint shows the same. (VV-s72)
- So for two of the three Meta models checked, OpenRouter's catalog says the Meta provider supports `none`, `required`, and a named function, which Meta's page says return HTTP 400. For the third model the catalog agrees with Meta's page. Meta's page states the rule for the whole API and names no model exception. Either OpenRouter rewrites the other values before calling Meta for those two models, or the flag is wrong for them. No document read settles which.

#### VV23. Claim 23 (Q37 and OV73 for vLLM, OV67 for llama.cpp): the reasoning response field name

Verdict: `confirmed` for both. Note on ids: in the first-wave file the vLLM item is OV73. OV71 is about llama.cpp's Messages and Responses routes.

- vLLM, `docs/features/reasoning_outputs.md` on `main` (latest commit touching the file: `beb72da79fe63262086418324c1101ad537e6601`, 2026-11-05): "Reasoning models return an additional `reasoning` field in their outputs, which contains the reasoning steps that led to the final conclusion. This field is not present in the outputs of other models." and "`reasoning` used to be called `reasoning_content`. To migrate, directly replace `reasoning_content` with `reasoning`. It is important that you also update your client code. Otherwise, your client code could silently read an empty `reasoning_content`, even when `reasoning` is populated." (VV-s73)
- llama.cpp, `tools/server/README.md` on `master` (latest commit touching the file: `0b51bb2fed322bc884f96f98db340e2b0f6ea176`, 2026-11-22): "The server supports parsing and returning reasoning via the `reasoning_content` field, similar to Deepseek API." and the row for the server flag `reasoning-format` (written here without its leading dashes): "controls whether thought tags are allowed and/or extracted from the response, and in which format they're returned; one of:" "- none: leaves thoughts unparsed in `message.content`" "- deepseek: puts thoughts in `message.reasoning_content`" "- deepseek-legacy: keeps `<think>` tags in `message.content` while also populating `message.reasoning_content`" "(default: auto)". (VV-s74, `<br/>` tags removed)
- So today the two main local servers use different field names for the same thing: vLLM `reasoning`, llama.cpp `reasoning_content`.

#### VV24. Claim 24 (W44 and C1, OV16, OV14): xAI tool cap, and encrypted reasoning on the Responses API only

Verdict: `conflict` for the tool cap (two xAI pages disagree, and each workspace document quoted one of them). `confirmed` for encrypted reasoning.

The cap:

- xAI function calling guide, "Tool Schema Reference" table, row `name`: "Unique identifier (max 350 tools per request)". (VV-s75)
- xAI Responses API reference, `tools`: "A list of tools the model may call in JSON-schema. Currently, only functions and web search are supported as tools. A max of 1000 tools are supported." (VV-s76)
- xAI Chat Completions API reference, `tools`: "A list of tools the model may call in JSON-schema. Currently, only functions are supported as a tool. Use this to provide a list of functions the model may generate JSON inputs for. A max of 1000 functions are supported." (VV-s77)
- So the 2026-10-14 workspace document (350, from the function calling page) and the 2026-10-28 survey and OV16 (1,000, from the API references) each quoted a real xAI page. xAI's own pages disagree today. The lower number is the safe one for a gateway to enforce, and which one the API applies is unverified.

Encrypted reasoning:

- Reasoning guide: "The reasoning content is encrypted by us and can be returned if you pass `include: ["reasoning.encrypted_content"]` to the Responses API. You can send the encrypted content back to provide more context to a previous conversation." (VV-s78)
- Same page: "On the Responses API, `grok-4.7` returns `reasoning.encrypted_content` on every response, whether or not `include` lists it, together with the encrypted outputs of any server-side tools." The page then says the reasoning items have an `encrypted_content` field and tells the caller to "pass them back unchanged in the next request's `input` so the model keeps its reasoning across turns even when you manage conversation history yourself." and "Chat Completions is unaffected; it has no field for the ciphertext." (VV-s78)
- Comparison page, row "Reasoning Models": Responses API "Full support with encrypted reasoning content", Chat Completions API (Deprecated) "No reasoning content returned". (VV-s79)
- A second disagreement between xAI pages, already listed by the first wave: the Chat Completions reference documents a response field "`reasoning_content` (string | null)" described as "The reasoning trace generated by the model." (VV-s77), while the comparison page says "No reasoning content returned". Both agree that the encrypted form exists on the Responses API only.
- Naming: xAI's pages now call the vendor "SpaceXSI" ("The Responses API is the recommended way to interact with SpaceXSI models."). (VV-s79)

## Claims that are not plainly confirmed

1. **Claim 1, short form (OA36).** "On the GPT-6 models tool calling exists on the Responses API only" is true for GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6.2 Astra, and GPT-6.2 Sol. GPT-6 Sol and GPT-6 Luna keep function calling on Chat Completions with `reasoning_effort: "none"`. OpenAI documents the whole rule as a limitation and prints no error body for it. (VV1)
2. **Claim 5, the 2026-09-20 workspace claim of an `invalid_encrypted_content` error.** Not in any of the 30 OpenAI pages fetched. Today's documented outcome for a cross-organization item is removal, reported as `organization_mismatch` in `input_transformations`. The outcome for a blob OpenAI never minted is undocumented. (VV5)
3. **Claim 13, the local survey's phrase "a 400 with code `missing_thought_signature`".** Wrong. The HTTP 400 on `generateContent` has `code: 400` and `status: "INVALID_ARGUMENT"`. `MISSING_THOUGHT_SIGNATURE` is a `finishReason` on `generateContent` and a "Generation error code" on the Interactions API. The first wave's GG6 is right. (VV13)
4. **Claim 18.** Google states no rule that names the AI Studio and Vertex pair. The Vertex page scopes a signature to "the same conversation history, model, and Google Cloud project", says a cross-project or cross-model signature is omitted without failing the request, and says a truncated or corrupted signature returns "400 (INVALID_ARGUMENT)". This scope statement, and the prefix rule beside it, were not in the first-wave file. (VV18)
5. **Claim 20 (OV30).** The docs sentence says "exactly 9 characters" for fine-tuning data. The alphanumeric rule and the error wording are in Mistral's `mistral-common` validator, which keeps the rule for tokenizer versions 3 to 11 and relaxes it to "a non-empty string other than 'null'" for version 13 or newer. (VV20)
6. **Claim 24, the tool cap.** Two xAI pages disagree: "max 350 tools per request" on the function calling guide, "A max of 1000 tools are supported" on the Responses reference and "A max of 1000 functions are supported" on the Chat Completions reference. (VV24)

Confirmed claims where the re-fetch added a detail the first wave did not have:

- **Claim 6 (OA41 against OA40).** OpenAI's rule for `phase` covers "models like `gpt-5.3-codex` and beyond" and is a recommendation with a quality cost. OpenRouter's five-slug table is narrower than OpenAI's wording. OA41 is the OpenRouter item, and OA40 is the vendor rule. (VV6)
- **Claim 7.** A second account rule on the same page: a block from an account "no longer in good standing" can be left out, on a wider model list that includes Claude Opus 5 and Claude Opus 5.5. (VV7)
- **Claim 8.** `compaction` is in the beta Messages schema's `stop_reason` enum (eight values) as well as in the compaction guide. The stable schema and the stop reasons guide list seven. (VV8)
- **Claim 9.** The error text is on Anthropic's errors page: `tool_choice: type "tool" and "any" are not supported for this model.` with type `invalid_request_error`, and the token counting endpoint returns it too. (VV9)
- **Claim 11.** Under the older API version on Claude 4.7 and later, `top_p: 1` is a non-default value (the default is `0.99`), and `temperature` together with `top_p` is a 400 even at defaults. (VV11)
- **Claim 16.** The `extra_content.google.thought_signature` path is printed on the Vertex page only. AI Studio points at it and prints native-shaped JSON. `cached_content` goes inside a top-level `extra_body.google` object in the request body. (VV16)
- **Claim 17.** The REST field is `continuationToken`. The reference says `max_output_tokens` "must be monotonically increased or omitted" across hops, where the guide says to keep it the same. (VV17)
- **Claim 19, the error text (OV23).** DeepSeek's pages print the status 400 and no message string. The quoted message rests on third parties, as OV23 says. (VV19)
- **Claim 22, the conflict between Meta's page and OpenRouter's catalog.** It holds for `meta/muse-spark-1.3` and `meta/muse-wonder` (all four `tool_choice` modes shown as supported on the Meta provider endpoint). For `meta/muse-ember` OpenRouter shows `auto` only, which agrees with Meta. (VV22)

## Unverified

- No request was sent to any model, so every statement here about what an API does is what the vendor's page says it does. In particular the HTTP status and message for a Chat Completions request with tools on GPT-5.4 and later, the DeepSeek and Mistral error bodies, and xAI's real tool cap are unobserved.
- Which Mistral hosted models use tokenizer version 13 or newer.
- Whether an AI Studio key and a Vertex endpoint inside one Google Cloud project share signature scope.

## Sources

All read on 2026-11-22 (Pacific time, UTC-8). Kind is "vendor doc" unless stated.

| Key | URL, or repository at commit with file path | Fetch time | Kind |
|---|---|---|---|
| VV-s1 | https://developers.openai.com/api/docs/guides/migrate-to-responses.md | 08:41 | vendor doc |
| VV-s2 | https://developers.openai.com/api/docs/guides/latest-model/gpt-6-astra.md | 08:41 | vendor doc |
| VV-s3 | https://developers.openai.com/api/docs/guides/latest-model/gpt-6.2-astra.md (identical to https://developers.openai.com/api/docs/guides/latest-model.md today) | 08:41 | vendor doc |
| VV-s4 | https://developers.openai.com/api/docs/guides/latest-model/gpt-5.4.md | 08:41 | vendor doc |
| VV-s5 | https://developers.openai.com/api/docs/guides/reasoning.md | 08:41 | vendor doc |
| VV-s6 | https://developers.openai.com/api/docs/changelog.md (entries Nov 17 and Sep 3, 2026) | 08:41 | vendor doc |
| VV-s7 | https://developers.openai.com/api/docs/guides/function-calling.md | 08:41 | vendor doc |
| VV-s8 | https://developers.openai.com/api/docs/models/{gpt-5.4, gpt-5.5, gpt-5.6-sol, gpt-5.6-terra, gpt-6-sol, gpt-6-luna, gpt-6-astra, gpt-6.1-sol, gpt-6.1-terra, gpt-6.2-astra, gpt-6.2-sol}.md | 08:43 | vendor doc |
| VV-s9 | https://developers.openai.com/api/reference/resources/chat/subresources/completions/methods/create.md | 08:41 | vendor doc |
| VV-s10 | https://developers.openai.com/api/reference/resources/responses/methods/create.md | 08:41 | vendor doc |
| VV-s11 | https://developers.openai.com/api/docs/guides/prompt-caching.md | 08:41 | vendor doc |
| VV-s12 | https://developers.openai.com/api/docs/guides/error-codes.md, https://developers.openai.com/api/docs/guides/keep-prefix-unchanged.md, https://developers.openai.com/api/docs/guides/your-data.md, https://developers.openai.com/api/reference/resources/responses/streaming-events.md, https://developers.openai.com/api/docs/guides/latest-model/gpt-5.5.md, https://developers.openai.com/api/docs/guides/latest-model/gpt-5.6.md, https://developers.openai.com/api/docs/llms.txt (searched for `invalid_encrypted_content` and for tool limits) | 08:41 to 08:43 | vendor doc |
| VV-s13 | https://developers.openai.com/api/docs/guides/conversation-state.md | 08:41 | vendor doc |
| VV-s20 | https://platform.claude.com/docs/en/build-with-claude/preserved-thinking.md | 08:43 | vendor doc |
| VV-s21 | https://platform.claude.com/docs/en/build-with-claude/preserved-thinking/prefix-checks.md | 08:43 | vendor doc |
| VV-s22 | https://platform.claude.com/docs/en/api/messages/create.md | 08:43 | vendor doc |
| VV-s23 | https://platform.claude.com/docs/en/build-with-claude/handling-stop-reasons.md | 08:43 | vendor doc |
| VV-s24 | https://platform.claude.com/docs/en/api/beta/messages/create.md | 08:44 | vendor doc |
| VV-s25 | https://platform.claude.com/docs/en/build-with-claude/compaction-threshold.md | 08:43 | vendor doc |
| VV-s26 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/server-tools.md | 08:43 | vendor doc |
| VV-s27 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use.md (same content as `define-tools.md`) | 08:43 | vendor doc |
| VV-s28 | https://platform.claude.com/docs/en/api/errors.md | 08:44 | vendor doc |
| VV-s29 | https://platform.claude.com/docs/en/build-with-claude/structured-outputs.md | 08:43 | vendor doc |
| VV-s30 | https://code.claude.com/docs/en/llm-gateway-protocol.md (indexed in https://code.claude.com/docs/llms.txt as "Claude Code gateway compatibility guide") | 08:43 | vendor doc (guidance to gateway authors) |
| VV-s31 | https://platform.claude.com/docs/en/api/versioning.md | 08:43 | vendor doc |
| VV-s32 | https://platform.claude.com/docs/en/api/migration-2026-10-21.md | 08:43 | vendor doc |
| VV-s33 | https://platform.claude.com/docs/en/build-with-claude/thinking.md | 08:43 | vendor doc |
| VV-s40 | https://aistudio.google.com/docs/thought-signatures?generate-content=true (Markdown; `https://ai.google.dev/gemini-api/docs/thought-signatures.md.txt` redirects to the Interactions variant at `https://aistudio.google.com/docs/thought-signatures.md`) | 08:46 | vendor doc |
| VV-s41 | https://aistudio.google.com/api-reference/generate-content.md and https://aistudio.google.com/api-reference/generate-content/generate-content.md | 08:47 | vendor doc |
| VV-s42 | https://aistudio.google.com/docs/api-errors.md (Interactions API) | 08:47 | vendor doc |
| VV-s43 | https://aistudio.google.com/docs/api-errors.md?generate-content=true | 08:47 | vendor doc |
| VV-s44 | https://aistudio.google.com/docs/thinking.md?generate-content=true | 08:47 | vendor doc |
| VV-s45 | https://aistudio.google.com/docs/tool-combination.md?generate-content=true | 08:47 | vendor doc |
| VV-s46 | https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/thinking/thought-signatures (HTML; reached also from https://cloud.google.com/vertex-ai/generative-ai/docs/thought-signatures; page stamp "Last updated 2026-11-20 UTC.") | 08:48 | vendor doc |
| VV-s47 | https://aistudio.google.com/docs/openai.md | 08:47 | vendor doc |
| VV-s48 | https://aistudio.google.com/docs/caching.md?generate-content=true | 08:47 | vendor doc |
| VV-s49 | https://aistudio.google.com/docs/long-decode.md?generate-content=true | 08:47 | vendor doc |
| VV-s50 | https://aistudio.google.com/docs/caching.md (Interactions API) | 08:47 | vendor doc |
| VV-s51 | https://aistudio.google.com/docs/whats-new-gemini-4-carbon.md?generate-content=true | 08:47 | vendor doc |
| VV-s52 | https://aistudio.google.com/docs/long-decode.md (Interactions API) | 08:47 | vendor doc |
| VV-s60 | https://api-docs.deepseek.com/guides/thinking_mode | 08:51 | vendor doc |
| VV-s61 | https://api-docs.deepseek.com/guides/tool_calls | 08:51 | vendor doc |
| VV-s62 | https://api-docs.deepseek.com/api/create-chat-completion | 08:51 | vendor doc |
| VV-s63 | https://api-docs.deepseek.com/quick_start/error_codes | 08:51 | vendor doc |
| VV-s64 | https://docs.mistral.ai/llms-full.txt (sections "OpenAI compatibility" under Chat completions, and the fine-tuning data format) | 08:51 | vendor doc |
| VV-s65 | `mistralai/mistral-common`, branch `main`, `src/mistral_common/protocol/instruct/validator.py` (latest commit on the file `fc079a099f6c2a6186c9829fe4405ee589aac576`, 2026-11-18), raw file from `raw.githubusercontent.com` | 08:52 | vendor doc (Mistral, vendor source) |
| VV-s66 | https://huggingface.co/moonshotai/Kimi-K3/raw/main/encoding_k3.py (repository revision `1f5d44f794ef598de7b6f529ef7923729b7e5408`, 2026-11-20) | 08:51 | vendor doc (Moonshot, vendor source) |
| VV-s67 | https://platform.kimi.ai/docs/guide/use-official-tools.md | 08:51 | vendor doc |
| VV-s68 | https://platform.kimi.ai/docs/api/chat.md | 08:51 | vendor doc |
| VV-s69 | https://platform.kimi.ai/docs/guide/use-thinking-models.md | 08:51 | vendor doc |
| VV-s70 | https://platform.kimi.ai/docs/guide/kimi-k3-quickstart.md | 08:51 | vendor doc |
| VV-s71 | https://dev.meta.ai/docs/tool-calling.md | 08:51 | vendor doc |
| VV-s72 | https://openrouter.ai/api/v1/models/meta/muse-spark-1.3/endpoints, https://openrouter.ai/api/v1/models/meta/muse-wonder/endpoints, https://openrouter.ai/api/v1/models/meta/muse-ember/endpoints (unauthenticated GET, HTTP 200) | 08:53 | OpenRouter spec (public catalog) |
| VV-s73 | `vllm-project/vllm`, branch `main`, `docs/features/reasoning_outputs.md` (latest commit on the file `beb72da79fe63262086418324c1101ad537e6601`, 2026-11-05), raw file | 08:51 | vendor doc (local server) |
| VV-s74 | `ggml-org/llama.cpp`, branch `master`, `tools/server/README.md` lines 240 and 1588 (latest commit on the file `0b51bb2fed322bc884f96f98db340e2b0f6ea176`, 2026-11-22), raw file | 08:51 | vendor doc (local server) |
| VV-s75 | https://docs.x.ai/developers/tools/function-calling.md | 08:51 | vendor doc |
| VV-s76 | https://docs.x.ai/developers/rest-api-reference/inference/responses.md | 08:51 | vendor doc |
| VV-s77 | https://docs.x.ai/developers/rest-api-reference/inference/chat-completions.md | 08:51 | vendor doc |
| VV-s78 | https://docs.x.ai/developers/model-capabilities/text/reasoning.md | 08:51 | vendor doc |
| VV-s79 | https://docs.x.ai/developers/model-capabilities/text/comparison.md | 08:51 | vendor doc |

First-wave evidence files opened for the claims: vendor-openai-responses (OA), vendor-anthropic (AN), vendor-google-gemini (GG), vendor-others-and-local-servers (OV), tools-structured-output (T), other-aggregators-cloud-gateways (Q), api-surfaces-request-superset (S), workspace-research-sweep (W), all dated 2026-11-22. Workspace document checked for the local survey's wording: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28), line 96.

