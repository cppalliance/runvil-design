---
produced: 2026-11-22
title: OpenRouter deep dive - verification of first-wave claims about open-source code at pinned commits (rig recorded OpenRouter fixtures, endpoint_slug signature suffix, reasoning_details replay in Zed, OpenRouter AI SDK provider, pi, Cline, LangChain, LiteLLM thought signatures and Responses bridge, agentgateway, Bifrost, Envoy AI Gateway, Portkey, Pydantic AI, rig replay rules)
---

# OpenRouter deep dive - verification of first-wave claims about open-source code at pinned commits

- What was read: 48 first-wave evidence items (prefixes F, G, H, J, K, O), grouped into 18 claims. For each one I opened the first-wave item, then opened the source file in the pinned clone and read the code around the cited lines. For the rig fixtures I parsed all 63 recorded OpenRouter cassettes and the 111-entry reply bank with Python and decoded every signature suffix myself.
- How many sources: 14 repositories at the commits the first wave pinned, one commit patch fetched from github.com without credentials (Zed `7fab4c0`), and OpenRouter's OpenAPI spec from the local mirror. Every file I needed was present in a first-wave clone, so I made zero new clones and the HEAD comparison is the same commit in every case.
- Read date: 2026-11-22. Nothing was run against a model. One local run happened: OpenRouter's AI SDK schema files, copied byte for byte into my scratch folder (SHA-256 equal to the pinned files), were executed with zod 4.6.7 on hand-made entries to settle what the parser does with an unknown `format` or `type`.

## Verdict table

Verdict words: `confirmed`, `confirmed with a correction`, `not confirmed`, `overstated`, `could not check`. Counts by evidence id: 39 confirmed, 7 confirmed with a correction, 2 overstated, 0 not confirmed, 0 could not check.

| Claim | What the first wave recorded | File and lines read, commit | Verdict | Correction or note |
|---|---|---|---|---|
| 1. G37 | Claude signatures in rig's recorded OpenRouter replies end in `.` plus base64 JSON `{"endpoint_slug":...}`; fixtures of 2026-09-22 lack it, those from 2026-10-02 have it | rig `a8673f7`: all 63 files under `crates/rig-cassette/fixtures/cassettes/openrouter/`, parsed and decoded | confirmed with a correction | The window is right for Claude signatures only. OpenAI `reasoning.encrypted` entries already had the same suffix on their `data` field on 2026-09-22 (4 replies in 3 fixtures). So the suffix was live before the window, and what changed in the window is that Claude signatures began to get it. Fields and routes are listed in VF1 |
| 2. G39 | Signed Claude `reasoning_details` went back with HTTP 200 on the Anthropic, Bedrock, and Vertex routes after the tool list changed | rig `a8673f7`: `cassette/context_binding.rs:1-169`; `context_binding/anthropic.yaml`, `amazon_bedrock.yaml`, `google_vertex.yaml` | confirmed | Each replay stayed on the route that issued the signature (`"only":[route]`, `"allow_fallbacks":false`). A 200 shows OpenRouter accepted the request. The recordings do not show what OpenRouter sent upstream |
| 3. G41 | OpenAI reasoning models arrive on the chat route as `reasoning.encrypted` entries with `rs_` ids, `format: "openai-responses-v1"`, `native_finish_reason: "completed"` | rig `a8673f7`: `reasoning_roundtrip/*.yaml`, `reasoning_tool_roundtrip/nonstreaming.yaml`, `reasoning_usage_matrix/*.yaml`, `streaming_tools/stream_encrypted_reasoning_survives_into_the_next_turn.yaml`, `reasoning_delta_hook/streaming.yaml` | confirmed | A `reasoning.summary` entry at `index: 0` comes first and the encrypted entry is at `index: 1`. With `"exclude":true` only the encrypted entry arrives, at `index: 0` |
| 4. O78 | A truncated tool call is reported as `finish_reason: "tool_calls"` beside `native_finish_reason: "length"`, arguments cut mid-string | rig `a8673f7`: `fixtures/bank/openrouter.yaml:1384-1420`, `:2106-2160`, `:1162-1175`, `:2509-2522` | confirmed | Four recordings, two models, blocking and streaming, all 2026-09-23 |
| 4. O77 | `native_finish_reason` differs for OpenAI models: `stop`/`length`/`tool_calls` for some, `completed`/`max_output_tokens` for others | same bank file plus all cassettes | confirmed | The split is by model on the same day. That the cause is the upstream protocol stays an inference. A second marker points the same way: `system_fingerprint` is `fp_...` on the chat-vocabulary models and `null` on the others |
| 5. G1 | Zed withholds `reasoning_details` for `anthropic/` models on OpenRouter; its comment calls the field output-only | zed `d31d107`: `crates/language_models/src/provider/open_router.rs:449-504` | confirmed | |
| 5. G122 | Commit `7fab4c0` of 2026-11-12 extended the rule to `~anthropic/` aliases, untested against the API | `https://github.com/zed-industries/zed/commit/7fab4c0527e10a466439b819d4e9e7fe34cd989c.patch` | confirmed | Commit date `Thu, 12 Nov 2026 16:04:51 +0000`. Its release note is about prompt caching; the echo of `reasoning_details` is listed as part of the same defect |
| 6. F1 | OpenRouter's AI SDK provider says `index` "cannot be used to reassemble the array" and merges streamed entries by type transition | ai-sdk-provider `7a235a4`: `src/chat/index.ts:890-949` | confirmed with a correction | The quote and the merge are exact. The comment's stated reason is that fragments of one block "all carry the same `index`". Two other clients use `index` as part of the merge key (rig `merge_details`, LangChain `_merge_reasoning_details`), and rig's recorded replays of arrays merged that way got 200. So "index is unusable" is this SDK's choice, and the recordings do not show it as a fact about OpenRouter |
| 6. F6 | De-duplicates `reasoning_details` across all assistant messages before replay | same repo: `src/chat/convert-to-openrouter-chat-messages.ts:69-73`, `:373-386`; `src/utils/reasoning-details-duplicate-tracker.ts:1-68` | confirmed | An entry with an empty key (text entry with empty text and empty signature) is also removed |
| 6. F7 | Strips unsigned Anthropic `reasoning.text` entries | same file `:319-356`, `:389-405` | confirmed | A missing `format` counts as `anthropic-claude-v1` |
| 6. F8 | Gemini reasoning text is always removed on replay | same file `:327-356`; `src/chat/signature-roundtrip.test.ts:251-340` | overstated | True for a `google-gemini-v1` `reasoning.text` entry whose `signature` is empty. The filter keeps one that has a `signature`. The SDK's own test names that case "Hypothetical", yet rig recorded exactly such an entry from `google/gemini-3-flash-preview` on 2026-10-02 |
| 6. F13 | Schema knows 6 `format` values and 3 entry types; others are dropped on parse | same repo: `src/schemas/format.ts:1-12`, `src/schemas/reasoning-details.ts:1-91`, `src/schemas/provider-metadata.ts:79-92`, `src/chat/schemas.ts:44-55`, `:171-181` | confirmed | Settled by running the schemas: the single entry is dropped, the rest of the array survives, and the parse succeeds. Same result on the stream path and on the replay path |
| 6. F14 | OpenRouter's spec has 12 formats and 4 entry types | `https://openrouter.ai/openapi.json`, `components.schemas.ReasoningFormat`, `ReasoningDetailUnion` (local mirror read 2026-11-22) | confirmed | 12 string values plus `null`; fourth type is `reasoning.server_tool_call` |
| 7. F40 | pi sends every `anthropic/*` model on OpenRouter to the Anthropic Messages route | pi `9c46526`: `packages/ai/catalog/sources/openrouter.ts:61-87`; `packages/ai/catalog/providers/openrouter.ts:66-104`; `packages/ai/CHANGELOG.md:106`, `:164` | confirmed with a correction | Two limits on "every": ids ending in `:batch` stay on chat completions, and the catalog only holds models that list `tools`. The `~anthropic/` aliases are included |
| 8. F43 | Cline reaches OpenRouter through a generic OpenAI-compatible provider that does not write `reasoning_details` | cline `8075d2e`: `sdk/packages/llms/src/providers/vendors/openai-compatible.ts:225-307`; `builtins.ts:1074-1091`; `ai-sdk.ts:757-835`, `:2078-2082`; vercel/ai `d55f3cf` `packages/openai-compatible` | confirmed | For a model that produces images the language model is OpenRouter's own provider (`openRouterImageProvider?.chat(modelId) ?? provider(modelId)`). Cline's history formatter still sends reasoning under `providerOptions.anthropic`, which that provider does not read |
| 9. H42 | LangChain's OpenRouter client removes ids starting with `rs_` before replay | langchain `8ff3cd3`: `libs/partners/openrouter/langchain_openrouter/chat_models.py:1196-1206`, `:1209-1268`, `:1321-1330` | confirmed | It removes the `id` key only. The entry and its `data` stay |
| 10. K10 | LiteLLM stores the Gemini thought signature inside the tool call id as `call_...__thought__<base64>` | litellm `90dce4b`: `litellm/litellm_core_utils/prompt_templates/factory.py:103`, `:1261-1277`; `litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py:2502-2558` | confirmed with a correction | The `call_` prefix holds only when Gemini sent no id of its own. Code comment: "Gemini 3.5+ returns a stable native `id`; prefer it over the synthetic call_<uuid>", so the id is then `<Gemini id>__thought__<signature>`. On by default, switched off by `thought_signature_in_tool_call_id` |
| 10. K12 | On the request the signature is looked up in three places, and Gemini 3+ gets a placeholder on the first call when it is missing | same `factory.py:1299-1379`, `:1437-1459` | confirmed | |
| 10. K13 | On a model or backend switch it resends with Google's placeholder after a 400 | `litellm/llms/vertex_ai/gemini/rejected_thought_signature_retry.py:1-235`; `litellm/utils.py:1146-1167` | confirmed with a correction | The first-wave item is accurate. The one-line summary merges three cases. The resend after a 400 fires on any 400 whose text contains "thought signature", which is how the Vertex and AI Studio mismatch shows up. History with no signature sent to Gemini 3+ gets the placeholder up front, with no 400. History sent to a non-Gemini model has the signature cut from the ids, with no placeholder |
| 11. K16 | The chat-to-Responses bridge refuses `previous_response_id` | `litellm/completion_extras/litellm_responses_transformation/transformation.py:271-276`, `:1440-1450` | confirmed | Raised as `litellm.UnsupportedParamsError` |
| 11. K17 | The bridge returns `reasoning_items` plus an order map with a sha256 checksum in `provider_specific_fields.openai_responses_replay` | same folder: `history.py:11`, `:72-75`, `:228-242`, `:300-313`, `:449-456`; `transformation.py:794-821` | confirmed | `reasoning_items` is a message field. The map is in `provider_specific_fields`. On for `openai` and `azure` only |
| 11. K19 | The caller must pass `include: ["reasoning.encrypted_content"]` on the bridge | same `transformation.py:1340-1378`, `:2140-2165`; `litellm/types/llms/openai.py:1404-1414`; `litellm/llms/chatgpt/responses/transformation.py:120-149`; text search of `litellm/` | confirmed | Settled for `openai` and `azure`: the bridge copies a caller's `include` through and adds nothing itself. One provider differs: the `chatgpt` Responses transform appends the value and sets `store` to `False` |
| 12. K6 | Anthropic reply: text joined into one string, thinking kept in a separate list | `litellm/llms/anthropic/chat/transformation.py:2961-3033` | confirmed | |
| 12. K7 | Anthropic request: block order rebuilt by a fixed pattern | `factory.py:3516-3750` | confirmed | The pattern applies when `content` is a string or null, which is what LiteLLM itself returns. When the caller sends `content` as a list, the list is walked in the caller's order |
| 12. K72 | Streamed thinking arrives as one fragment per delta tagged with `index`; whether LiteLLM's rebuilder merges was open | `litellm/llms/anthropic/chat/handler.py:878-900`; `litellm/litellm_core_utils/streaming_chunk_builder_utils.py:806-862`, `:1697-1744`, `:1843-1851` | confirmed with a correction | Settled: `stream_chunk_builder` merges them. It joins `thinking` text until a fragment with a `signature` arrives and then closes the block. It ignores `index`, the merged block has the keys `type`, `thinking`, `signature`, and a run that never gets a signature is discarded. A caller that reads raw chunks still has to merge |
| 13. K24 | LiteLLM's extension fields are stripped in six provider transforms; the shared OpenAI-compatible path sends them as given | `litellm/litellm_core_utils/prompt_templates/common_utils.py:1342-1367`; six callers found by search; `litellm/llms/openai/chat/gpt_transformation.py:476-488`, `:694-730` | confirmed | Callers: `together_ai`, `mistral`, `groq`, `friendliai`, `fireworks_ai`, `databricks`. `hosted_vllm` removes `thinking_blocks` inline |
| 13. K49 | An unsupported parameter is an error by default | `litellm/utils.py:5973-6035` | confirmed | `UnsupportedParamsError` with `status_code=500` |
| 13. K32 | LiteLLM creates a Gemini `cachedContents` object from `cache_control` marks, keyed by a hash stored in `displayName` | `litellm/llms/vertex_ai/context_caching/vertex_ai_context_caching.py:211-308`, `:413-562`; `transformation.py:110-203`; `litellm/caching/caching.py:586-609` | confirmed | The key is a sha256 over messages, tools, tool choice, and model |
| 13. K35 | `prompt_tokens` is made to include cache reads and writes for every vendor | `litellm/llms/anthropic/chat/transformation.py:3179-3223`; `litellm/llms/bedrock/chat/converse_transformation.py:3021`; `vertex_and_google_ai_studio_gemini.py:2832-2843` | overstated | Read in code for Anthropic, Bedrock Converse, and Gemini (plus DeepSeek's field mapping in the first wave). "Every vendor" rests on a sentence in LiteLLM's documentation, not on code read for each vendor |
| 14. J8 | agentgateway's `reasoning_signature` survives only a reply with exactly one thinking block; redacted thinking is dropped | agentgateway `888badb`: `crates/llm/src/conversion/messages.rs:917-990` | confirmed | |
| 14. J34 | agentgateway copies tool-call ids unchanged; Bifrost rewrites them | same file `:479-500`, `:731`, `:931-949`; search of `crates/llm/src` and `crates/agentgateway/src/llm`; bifrost `cae639c`: `core/providers/utils/utils.go:46-95`, `core/providers/anthropic/chat.go:1147-1155` | confirmed | Also read: Portkey (`id: toolCall.id`) and Envoy (`ID: *toolCall.ID`) copy the id on their Anthropic paths. So of the four gateways, Bifrost is the one that rewrites |
| 15. J27 | Bifrost's `reasoning_details` has OpenRouter's name and three type strings; `format` is the field OpenRouter has and Bifrost lacks | bifrost `cae639c`: `core/schemas/chatcompletions.go:1962-1980` | confirmed | A fourth constant, `reasoning.content_blocks`, appears only at its declaration in the whole repository, tests included |
| 15. J31 | Bifrost puts the Gemini signature in the tool call id (`<id>_ts_<signature>`) and also in `reasoning_details` | `core/providers/gemini/chat.go:187-258`; `core/providers/utils/utils.go:41-44` | confirmed | |
| 15. J38 (with J39) | Bifrost treats encrypted reasoning as bound to its issuer, strips it and retries once on a 400 | `core/providers/utils/utils.go:107-192`; `core/encryptedreasoning.go:151-243`; `core/bifrost.go:7933-7961` | confirmed | The strip-and-retry text is recorded in the first wave as J39. J38 is the reasoning item id stored inside a signature |
| 16. J9 | agentgateway replays thinking first, then text, then tool calls | agentgateway `messages.rs:432-503` | confirmed | |
| 16. J30 | Bifrost builds reasoning first, then text, then tool calls | bifrost `core/providers/anthropic/chat.go:1099-1175` | confirmed | |
| 16. J74 | Portkey moves tool uses to the end | portkey `669825c`: `src/providers/anthropic/chatComplete.ts:141-182`, `:565-603` | confirmed | With strict mode off, the non-tool blocks keep Anthropic's order in `content_blocks` |
| 16. J86 | Envoy AI Gateway's non-streaming Anthropic reply loses blocks | agent-router (formerly envoyproxy/ai-gateway) `9cf8505`: `internal/translator/anthropic_helper.go:1438-1490` | confirmed | |
| 16. J100 | No gateway keeps Anthropic's block order: agentgateway, Bifrost, and Envoy rebuild reasoning, then text, then tool calls | same Envoy file `:404-489` plus the three above | confirmed with a correction | Right for agentgateway and Bifrost. Envoy's request path writes the caller's content parts (text, thinking, redacted thinking) in the order the caller gave them and then appends tool uses, with no reasoning-first rule. Envoy's reply path splits the blocks into `content` (first text only), `tool_calls`, and `reasoning_content` (last thinking wins), which is where the order goes |
| 16. J87 | Envoy's streaming path writes thinking text into `content`, with no signature case | same file `:1232-1242`, `:1283-1317`; search of `internal/translator` | confirmed | |
| 17. H50 to H54 | rig keeps raw provider JSON beside each neutral block, tags the turn with wire, provider, and model, replays only on an exact match, fingerprints blocks and context | rig `a8673f7`: `crates/rig-core/src/completion/message.rs:35-52`; `message/native.rs:1-101`, `:132-138`, `:232-262`; `history.rs:266-328`, `:398-439`, `:463-495`, `:1179-1197`; `providers/anthropic/wire.rs:862-889`; `providers/openai/responses_api/wire.rs:228-250`; `providers/openai/wire/chat.rs:1000-1002`; `providers/gemini/completion.rs:315-317` | confirmed | |
| 17. H27 | Pydantic AI judges client-held vendor state by a "canonical system" name and server-stored ids by endpoint URL | pydantic-ai `463a6d5`: `pydantic_ai/_canonical_system.py:1-14`; `models/__init__.py:1410-1482` | confirmed | |
| 18. H30 | Pydantic AI: the Gemini API and Google Cloud reject each other's signatures | pydantic-ai `models/google.py:2113-2119`, `:2206-2218` | confirmed | |
| 18. H40 | LangChain: Vertex and the Gemini Developer API share signature formats | langchain-google `227abc8`: `libs/genai/langchain_google_genai/_compat.py:11-27`, `:210-242` | confirmed | The two projects disagree on a fact. LiteLLM's code sides with Pydantic AI (VF18) |

## Method notes that apply to every section

- Clones read: the first-wave clones, read only. The only git commands I ran on them read state: `git rev-parse HEAD`, `git log -1`, and a config read. Every HEAD equals the commit in the first wave's sources table.
- Recording dates for rig's fixtures come from the reply id. rig writes `"created":0` into every recorded body, and the id has the form `gen-<number>-<random>`. I read the number as a Unix time. An independent check agrees: each OpenAI encrypted reasoning blob begins with byte `0x80` followed by eight bytes that read as a Unix time, and in all 11 recorded blobs that time is 3 to 16 seconds after the id's time.
- Scripts and their output are in my scratch folder: `rig_census.py`, `rig_detail.py`, `rig_bank.py`, `rig_more.py`, `rig_fernet.py`, and `zodtest/parse-test.ts`.
- Searches for things code does not do are stated with the pattern and the folder searched.

## VF1. G37 - the `endpoint_slug` suffix in rig's recorded OpenRouter traffic

Verdict: confirmed with a correction.

**Which field holds it, and on which route.** I walked every string in every recorded request and reply and tried to decode the part after the last `.` as base64url JSON. It decoded in exactly these places:

| Route | Body | Field | Entry it sits on |
|---|---|---|---|
| `POST /api/v1/chat/completions` | reply, non-streaming | `choices[].message.reasoning_details[].signature` | `"type":"reasoning.text"`, `"format":"anthropic-claude-v1"` |
| `POST /api/v1/chat/completions` | reply, streaming | `choices[].delta.reasoning_details[].signature` | same; the signature arrives alone in its own delta entry with keys `format`, `index`, `signature`, `type` |
| `POST /api/v1/chat/completions` | reply, both | `choices[].message.reasoning_details[].data` and `choices[].delta.reasoning_details[].data` | `"type":"reasoning.encrypted"`, `"format":"openai-responses-v1"`, `id` starting `rs_` |
| `POST /api/v1/chat/completions` | request (replay) | `messages[].reasoning_details[].signature` and `messages[].reasoning_details[].data` | the same entries sent back unchanged |
| `POST /api/v1/responses` | reply, non-streaming | `output[].signature` | item with `"type":"reasoning"`, `"format":"anthropic-claude-v1"` |
| `POST /api/v1/responses` | reply, streaming | `item.signature` and `response.output[].signature` | same item inside stream events |
| `POST /api/v1/responses` | request (replay) | `input[].signature` | same item sent back |

The fixtures hold these routes: `/api/v1/chat/completions` (52 files), `/api/v1/responses` (4), `/api/v1/embeddings` (2), `/api/v1/audio/transcriptions` (2), `/api/v1/systemone` (2), `/api/v1/models` (1). The Anthropic Messages route (`/api/v1/messages`) is not recorded, so the fixtures say nothing about that route.

**Claude signatures, every recorded reply.** Decoded with Python `base64.urlsafe_b64decode` after padding.

| Fixture and exchange | Recorded (UTC, from reply id) | Route | Model | `provider` in reply | Decoded value after the last `.` |
|---|---|---|---|---|---|
| `reasoning_tool_order_matrix/blocking_single.yaml` #0 | 2026-09-22 16:48:17 | chat | `anthropic/claude-haiku-4.5` | Anthropic | the signature is 868 characters of standard base64 with zero `.` characters |
| `reasoning_tool_order_matrix/streaming_single.yaml` #0 | 2026-09-22 16:48:24 | chat, stream | `anthropic/claude-haiku-4.5` | Anthropic | same shape, 724 characters |
| `upstream_switch_matrix/responses_switch_streamed.yaml` #0, #2 | 2026-10-02 20:14:36, 20:14:41 | responses, stream | `anthropic/claude-haiku-4.5` | n/a (this body names the model only) | `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` |
| `upstream_switch_matrix/responses_switch_unary.yaml` #0, #2 | 2026-10-02 20:14:54, 20:15:00 | responses | `anthropic/claude-haiku-4.5` | n/a (this body names the model only) | `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` |
| `upstream_switch_matrix/switch_streamed.yaml` #0, #2 | 2026-10-02 20:15:13, 20:15:16 | chat, stream | `anthropic/claude-haiku-4.5` | Amazon Bedrock | `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` |
| `upstream_switch_matrix/switch_unary.yaml` #0, #2 | 2026-10-02 20:15:29, 20:15:33 | chat | `anthropic/claude-haiku-4.5` | Amazon Bedrock | `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` |
| `reasoning_tool_order_matrix/streaming_signed_agent_roundtrip.yaml` #0 | 2026-10-02 22:00:40 | chat, stream | `anthropic/claude-haiku-4.5` | Anthropic | `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|anthropic"}` |
| `context_binding/amazon_bedrock.yaml` #0, #1 | 2026-10-03 15:34:23, 15:34:29 | chat | `anthropic/claude-opus-5.5` | Amazon Bedrock | `{"endpoint_slug":"anthropic/claude-opus-5.5-20260921\|amazon-bedrock"}` |
| `context_binding/anthropic.yaml` #0, #1 | 2026-10-03 15:34:38, 15:34:45 | chat | `anthropic/claude-opus-5.5` | Anthropic | `{"endpoint_slug":"anthropic/claude-opus-5.5-20260921\|anthropic"}` |
| `context_binding/google_vertex.yaml` #0, #1 | 2026-10-03 15:34:55, 15:35:02 | chat | `anthropic/claude-opus-5.5` | Google | `{"endpoint_slug":"anthropic/claude-opus-5.5-20260921\|google-vertex/global"}` |

So for Claude the first wave's dates hold: 2 replies on 2026-09-22 with a bare signature, 15 replies in 8 fixtures from 2026-10-02 20:14 UTC on with the suffix. The same provider (Anthropic direct, pinned with `"provider":{"allow_fallbacks":false,"order":["Anthropic"]}`) appears on both sides of the change, so the difference follows the date and not the route.

**The correction: OpenAI `data` already had the suffix on 2026-09-22.**

| Fixture and exchange | Recorded (UTC, from reply id) | Time inside the blob | Model | Field | Decoded value after the last `.` |
|---|---|---|---|---|---|
| `reasoning_delta_hook/streaming.yaml` #0 | 2026-09-22 16:47:12 | 2026-09-22 16:47:15 | `openai/gpt-5.2` | `delta.reasoning_details[].data` | `{"endpoint_slug":"openai/gpt-5.2-20251211\|openai"}` |
| `reasoning_usage_matrix/blocking_excluded_reasoning_still_counts_tokens.yaml` #0 | 2026-09-22 16:49:09 | 2026-09-22 16:49:22 | `openai/o4-mini` | `message.reasoning_details[].data` | `{"endpoint_slug":"openai/o4-mini-2025-04-16\|openai"}` |
| `reasoning_usage_matrix/transports_agree_on_reasoning_tokens.yaml` #0, #1 | 2026-09-22 16:51:57, 16:52:04 | 16:52:04, 16:52:16 | `openai/o4-mini` | `message...data`, `delta...data` | `{"endpoint_slug":"openai/o4-mini-2025-04-16\|openai"}` |
| `reasoning_roundtrip/streaming.yaml` #0, #1 | 2026-10-02 04:59:29, 04:59:35 | 04:59:33, 04:59:51 | `openai/gpt-5.2` | `delta...data` | `{"endpoint_slug":"openai/gpt-5.2-20251211\|openai"}` |
| `streaming_tools/stream_encrypted_reasoning_survives_into_the_next_turn.yaml` #0, #1 | 2026-10-02 05:00:44, 05:00:47 | 05:00:47, 05:00:50 | `openai/o4-mini` | `delta...data` | `{"endpoint_slug":"openai/o4-mini-2025-04-16\|openai"}` |
| `reasoning_roundtrip/nonstreaming.yaml` #0, #1 | 2026-10-02 20:12:23, 20:12:29 | 20:12:27, 20:12:45 | `openai/gpt-5.2` | `message...data` | `{"endpoint_slug":"openai/gpt-5.2-20251211\|openai"}` |
| `reasoning_tool_roundtrip/nonstreaming.yaml` #0 | 2026-10-02 20:13:01 | 20:13:04 | `openai/gpt-5.2` | `message...data` | `{"endpoint_slug":"openai/gpt-5.2-20251211\|openai"}` |

Four replies on 2026-09-22, recorded within five minutes of the two bare Claude signatures, already end in the suffix. What the fixtures date is therefore the day Claude signatures joined a wrapping that OpenAI blobs already had. The first-wave item's phrase "from 2026-10-02 the signature has an OpenRouter suffix" is right for the Claude `signature` field and should not be read as the start date of the suffix itself.

**Gemini, one recorded signature, with no suffix.** `upstream_switch_matrix/switch_streamed.yaml` #1 (2026-10-02 20:15:15, `google/gemini-3-flash-preview`, provider Google) has one delta entry: `{"format": "google-gemini-v1", "index": 0, "signature": "AY89a19QiA2xoKvfVf9xoeiRvpyOPIzSO188TDrn/lJkGATbpvhpjLr/62Z2pxCwkOegYrTKtpgQmdYpeyJaswt4r0VRJCqYtD2JpNNmvsjx/G12bJ3BhA7v", "type": "reasoning.text"}`. It is 120 characters with zero `.` characters. The non-streaming twin (`switch_unary.yaml` #1) has a `reasoning.text` entry with `text` only, and the Responses-route twins have a `reasoning` item with `format: "google-gemini-v1"` and the keys `content`, `format`, `id`, `status`, `summary`, `type`.

**Other first-wave details in G37, checked.** The reply repeats the reasoning text in `"reasoning"` and in the entry's `"text"` (seen in `blocking_single.yaml:15`). The decoded tail matches the reply's `provider` field in every chat-route row above. Lengths 868 and 724 for the two bare signatures are exact.

## VF2. G39 - signed Claude reasoning accepted after the tool list changed, on three routes

Verdict: confirmed.

The test file states the purpose (`crates/rig-cassette/tests/providers/openrouter/cassette/context_binding.rs:1-5`):

```
//! Live fact behind Rig replaying OpenRouter Claude turns verbatim: on each
//! route OpenRouter serves Claude Opus 5.5 from (Anthropic, Amazon Bedrock and
//! Google Vertex), a turn's signed `reasoning_details` go back after the tool
//! list changes, and the request succeeds. So Chat does not bind context on
//! OpenRouter.
```

The recorded assertions (`context_binding.rs:128-138`): `assert_eq!(&assistant["reasoning_details"], first, "{scenario}: the signed reasoning goes back exactly as received");` and `assert_eq!(*status, 200, "{scenario}: OpenRouter accepts every turn");` and `assert_eq!(reply["provider"], served, "{scenario}: the pinned route serves it");`.

What I read in the three fixtures with my own parser:

| Fixture | Exchange | Status | `tools` in request | Request `provider` | Reply `provider` | Replayed entry equals first reply's entry | `finish_reason`, `native_finish_reason` |
|---|---|---|---|---|---|---|---|
| `context_binding/anthropic.yaml` | 0 | 200 | `calc` | `{"allow_fallbacks": false, "only": ["anthropic"]}` | Anthropic | n/a | `tool_calls`, `tool_use` |
| same | 1 | 200 | `calc`, `plot` | same | Anthropic | yes | `stop`, `end_turn` |
| `context_binding/amazon_bedrock.yaml` | 0 | 200 | `calc` | `{"allow_fallbacks": false, "only": ["amazon-bedrock"]}` | Amazon Bedrock | n/a | `tool_calls`, `tool_use` |
| same | 1 | 200 | `calc`, `plot` | same | Amazon Bedrock | yes | `stop`, `end_turn` |
| `context_binding/google_vertex.yaml` | 0 | 200 | `calc` | `{"allow_fallbacks": false, "only": ["google-vertex"]}` | Google | n/a | `tool_calls`, `tool_use` |
| same | 1 | 200 | `calc`, `plot` | same | Google | yes | `stop`, `end_turn` |

All six were recorded 2026-10-03 between 15:34:23 and 15:35:02 UTC with model `anthropic/claude-opus-5.5` and `"reasoning":{"effort":"high"}`. The replayed assistant message has the keys `content`, `reasoning`, `reasoning_details`, `role`, `tool_calls`, and the tool call ids keep the upstream's own shape (`toolu_...`, `toolu_bdrk_...`, `toolu_vrtx_...`).

Also confirmed: `reasoning_tool_order_matrix/streaming_signed_agent_roundtrip.yaml` exchange 1 (2026-10-02 22:00:41) replays the streamed signature, pinned with `{"allow_fallbacks": false, "order": ["Anthropic"]}`, and returns 200.

Two limits worth stating. Each replay went back to the route that issued the signature, so these fixtures do not test a signature from one route sent to another. And a 200 shows that OpenRouter accepted the request. The body recorded is OpenRouter's reply, so what OpenRouter forwarded to the upstream is outside the recording. One nearby case exists: in `upstream_switch_matrix/switch_unary.yaml` exchange 2, a Claude entry signed by Bedrock goes back after a Gemini turn in between, with no provider pin, is served by Amazon Bedrock again, and returns 200.

## VF3. G41 - OpenAI reasoning models on the chat route

Verdict: confirmed.

`reasoning_roundtrip/nonstreaming.yaml` exchange 0 (`POST /api/v1/chat/completions`, model `openai/gpt-5.2`, request `"reasoning":{"effort":"medium"}`, recorded 2026-10-02 20:12:23, provider OpenAI), reply fields as recorded, long values trimmed by me:

```
"finish_reason": "stop", "native_finish_reason": "completed"
"reasoning_details": [
  {"format": "openai-responses-v1", "index": 0, "summary": "**Calculating meeting time**\n\nI need to ...", "type": "reasoning.summary"},
  {"data": "gAAAAABqwBArBYP7e3J1TDVWIkg9lsqZABsxphWK...tMjAyNTEyMTF8b3BlbmFpIn0", "format": "openai-responses-v1", "id": "rs_02de745d9b1e8fa7016ac01027c6e887d18f8d7f5699976b67", "index": 1, "type": "reasoning.encrypted"}
]
```

Exchange 1 of the same fixture sends both entries back unchanged in `messages[2].reasoning_details` (message keys `content`, `reasoning`, `reasoning_details`, `role`), `rs_` id included, and gets 200.

Other recorded cases that match the first wave:

- `reasoning_tool_roundtrip/nonstreaming.yaml` #0: `finish_reason` `tool_calls` with `native_finish_reason` `completed`, tool call id `call_qPoLPPBC0vBsH6iFIawlPX7B`.
- `reasoning_usage_matrix/blocking_excluded_reasoning_still_counts_tokens.yaml` #0 (`openai/o4-mini`, request `"reasoning":{"effort":"medium","exclude":true}`, 2026-09-22): `finish_reason` `length`, `native_finish_reason` `max_output_tokens`, `reasoning` null, and one entry `{"data": "gAAAAABqsrGS...", "format": "openai-responses-v1", "id": "rs_0e10fb98fcc7651d016ab2b186d32c87d19b03c3a636eadf99", "index": 0, "type": "reasoning.encrypted"}`.
- Streaming (`reasoning_roundtrip/streaming.yaml` #0, 441 chunks): 102 `reasoning.summary` delta entries, all with `index: 0`, then one `reasoning.encrypted` entry with `index: 1` in chunk 102, whole, with its `rs_` id. For `openai/o4-mini` with no summary the encrypted entry is in chunk 0 with `index: 0`.
- Stream tail in the three OpenAI reasoning streams I dumped: a chunk with `finish_reason` and `native_finish_reason` and delta keys `content`, `reasoning`, `role`, then a second chunk with the same two finish fields, delta keys `content`, `role`, and `usage`.

## VF4. O78 and O77 - truncated tool call, and native finish reasons

Verdict: O78 confirmed. O77 confirmed.

**O78.** The bank file header says what it holds (`crates/rig-cassette/fixtures/bank/openrouter.yaml:1-4`): "The reply bank, written by `cargo xtask cassette bank`. One real recorded reply per provider, completion encoder, reply shape and called tools, the smallest the corpus holds, with the interaction it came from."

Streaming, `openai/gpt-4o-mini`, source `openrouter/tool_truncation_contract_matrix/streaming_gpt4o_low_agent.yaml#0`, recorded 2026-09-23 08:37:04 (`bank/openrouter.yaml:1406-1420`, lines trimmed by me to the delta and finish fields):

```
1406| data: {"choices":[{"delta":{"content":null,"role":"assistant","tool_calls":[{"function":{"arguments":"\":\""},"index":0}]},"finish_reason":null,"index":0,"native_finish_reason":null}], ...
1408| ... "arguments":"The" ...
1410| ... "arguments":" cache" ...
1412| ... "arguments":" warmer" ...
1414| ... "arguments":" raced" ...
1416| data: {"choices":[{"delta":{"content":"","role":"assistant"},"finish_reason":"tool_calls","index":0,"native_finish_reason":"length"}],"created":0,"id":"gen-1790152624-yW3HHZPqZWBquQCuHWtL","model":"openai/gpt-4o-mini","object":"chat.completion.chunk","provider":"OpenAI","system_fingerprint":"fp_253a61362d"}
1418| (same finish fields, plus "service_tier":null and "usage")
1420| data: [DONE]
```

Non-streaming, same model (`bank/openrouter.yaml:2522`): `"finish_reason":"tool_calls"` ... `"tool_calls":[{"function":{"arguments":"{\"summary\":\"The cache warmer raced","name":"file_report"},"id":"call_CHyBBxRD7t1ONiAYScP5UwbF","index":0,"type":"function"}]},"native_finish_reason":"length"` with `"completion_tokens":16`.

`openai/gpt-4.1-mini`, same day: streaming at lines 2156 and 2158 has `"finish_reason":"tool_calls"` with `"native_finish_reason":"max_output_tokens"`, and non-streaming at line 1175 has arguments `"{\"summary\":\"The cache warmer raced the artifact uploader, the retry storm saturated the"` with the same pair. I joined the argument fragments of each of the four replies and gave them to `json.loads`: all four fail to parse. The first wave's line references (1406-1420 and 2156-2158) are exact.

One note on reach: the source cassettes named by these bank entries (`tool_truncation_contract_matrix/...`) are not among the 63 cassette files at this commit, so the request bodies (the token limit that was set) are not available. `"completion_tokens":16` in all four suggests a limit of 16.

**O77.** Pairs I counted myself across the 63 cassettes and the bank, chat route only:

| Model | `provider` | Recorded dates | (`finish_reason`, `native_finish_reason`) pairs | `system_fingerprint` |
|---|---|---|---|---|
| `openai/gpt-4o` | OpenAI, Azure | 2026-09-23, 10-02, 10-03 | (`stop`, `stop`), (`tool_calls`, `tool_calls`) | `fp_...` |
| `openai/gpt-4o-mini` | OpenAI, Azure | 2026-09-23, 10-02, 10-03 | (`stop`, `stop`), (`tool_calls`, `tool_calls`), (`length`, `length`), (`tool_calls`, `length`) | `fp_...` |
| `openai/gpt-4.1-mini` | OpenAI | 2026-09-23, 10-03 | (`stop`, `completed`), (`tool_calls`, `completed`), (`length`, `max_output_tokens`), (`tool_calls`, `max_output_tokens`) | `null` |
| `openai/gpt-4.1` | OpenAI | 2026-09-23 | (`content_filter`, `content_filter`) | `null` |
| `openai/gpt-5.2` | OpenAI | 2026-09-22, 10-02 | (`stop`, `completed`), (`tool_calls`, `completed`) | `null` |
| `openai/o4-mini` | OpenAI | 2026-09-22, 10-02 | (`stop`, `completed`), (`tool_calls`, `completed`), (`length`, `max_output_tokens`) | `null` |
| `anthropic/claude-haiku-4.5`, `anthropic/claude-opus-5.5` | Anthropic, Amazon Bedrock, Google | 2026-09-22 to 10-03 | (`stop`, `end_turn`), (`tool_calls`, `tool_use`) | `null` |
| `anthropic/claude-sonnet-4.6` | Claude Platform on AWS | 2026-10-02 | (`stop`, `end_turn`) | `null` |
| `google/gemini-2.5-flash`, `google/gemini-3-flash-preview` | Google | 2026-09-23, 10-02 | (`stop`, `STOP`) | `null` |
| `deepseek/deepseek-v3.2`, `deepseek/deepseek-r1-0528` | AtlasCloud, Friendli, GMICloud, SiliconFlow, DeepInfra | 2026-09-23 | (`stop`, `stop`), (`length`, `length`) | `null` |

Every row of the first wave's O77 table is in the recordings. The two vocabularies for OpenAI models appear seconds apart on the same day (`openai/gpt-4o-mini` with `length` at 2026-09-23 08:36:20 and `openai/gpt-4.1-mini` with `max_output_tokens` at 08:36:15), so the split is by model and not by date. The recordings do not name the upstream protocol. Two things in them fit the first wave's reading that the second group is served from OpenAI's Responses API: the `openai-responses-v1` format tag on the reasoning entries of `gpt-5.2` and `o4-mini`, and the `system_fingerprint` column above, which has a value only on the first group. `openai/gpt-4.1-mini` is in the second group although all 188 of its recorded choices (chunks included) hold zero `reasoning_details` entries, so the split is not "reasoning models only".

## VF5. G1 and G122 - Zed withholds `reasoning_details` for Claude on OpenRouter

Verdict: G1 confirmed. G122 confirmed.

`zed-industries/zed` at `d31d1074`, `crates/language_models/src/provider/open_router.rs:459-468`:

```
    // Anthropic models via OpenRouter don't accept reasoning_details being echoed back
    // in requests - it's an output-only field for them. However, Gemini models require
    // the thought signatures to be echoed back for proper reasoning chain continuity.
    // Note: OpenRouter's model API provides an `architecture.tokenizer` field (e.g. "Claude",
    // "Gemini") which could replace this ID prefix check, but since this is the only place
    // we need this distinction, we're just using this less invasive check instead.
    // If we ever have a more formal distionction between the models in the future,
    // we should revise this to use that instead.
    // OpenRouter aliases are prefixed with `~` (e.g. `~anthropic/claude-sonnet-latest`).
    let is_anthropic_model = model.id().trim_start_matches('~').starts_with("anthropic/");
```

Lines 477-481: `let reasoning_details_for_message = if is_anthropic_model { None } else { message.reasoning_details.clone() };`. Lines 502-504: `MessageContent::Thinking { .. } => {}`, `MessageContent::RedactedThinking(_) => {}`, `MessageContent::Compaction(_) => {}`. The value is then passed to every `add_message_content_part` call and to the assistant message built for a tool call (line 539), so for an `anthropic/` or `~anthropic/` id the request holds text, images, and tool calls only.

The commit, read as a patch from github.com:

- Header: `From 7fab4c0527e10a466439b819d4e9e7fe34cd989c`, `Date: Thu, 12 Nov 2026 16:04:51 +0000`, `Subject: [PATCH] language_models: Detect OpenRouter Anthropic aliases (#66857)`.
- Summary text: "`into_open_router` only checked for the `anthropic/` prefix, so aliases were not treated as Anthropic models: no `cache_control` was sent (no prompt caching), and `reasoning_details` was echoed back in requests."
- Testing text: "I did not test this manually against the OpenRouter API."
- Release note: "Fixed prompt caching for OpenRouter Anthropic aliases (e.g. `~anthropic/...`)".
- Diff: `-    let is_anthropic_model = model.id().starts_with("anthropic/");` replaced by the two lines quoted above, plus a test `test_anthropic_alias_model_uses_anthropic_request_handling`.

So the rule was touched ten days before the read date and widened to aliases. The commit treats sending `reasoning_details` to an Anthropic alias as part of a defect, and says the change was not tried against the API. Zed's comment and rig's recordings (VF2) disagree about what OpenRouter accepts, and the recordings are the direct evidence.

## VF6. F1, F6, F7, F8, F13, F14 - OpenRouter's AI SDK provider

Verdicts: F1 confirmed with a correction. F6 confirmed. F7 confirmed. F8 overstated. F13 confirmed. F14 confirmed.

All from `OpenRouterTeam/ai-sdk-provider` at `7a235a4c` (package version 3.1.0).

**F1.** `src/chat/index.ts:890-899`, quoted in three pieces to leave out one punctuation mark: "Accumulate reasoning_details to preserve for multi-turn conversations. Merge consecutive same-type text/summary deltas into a single entry. Providers (e.g. OpenAI `openai-responses-v1`) stream a logical reasoning block as many small deltas that all carry the same `index` (frequently `index: 0`), so `index` cannot be used to reassemble the array", then "reassembly is driven by the `type` transition instead.", then "Without merging summary deltas the array fragments into one entry per delta, which breaks multi-turn round-tripping when re-submitted (see issue #519, langchain #36400)."

The merge itself (`:906-948`): a text delta joins the previous entry when `lastDetail?.type === ReasoningDetailType.Text && !lastDetail.signature`, with the comment "A signature closes the text detail it signs: once the last text detail is signed, the next text delta starts a new block (e.g. multiple signed Anthropic thinking blocks)." A summary delta joins when `lastDetail?.type === ReasoningDetailType.Summary`. Anything else: "Encrypted details are opaque blobs (each has its own id); they are discrete and must be pushed as-is, never merged."

The correction is about what this shows of OpenRouter. The comment's own premise is that the fragments of one block share an `index`. Two other clients merge on exactly that:

- rig, `crates/rig-core/src/providers/openai/wire/chat.rs:1766-1787`: "A text or summary fragment continues the last entry of its type and index", implemented as `Some("reasoning.text" | "reasoning.summary") => { last.get("index") == detail.get("index") }`.
- LangChain, `libs/partners/openrouter/langchain_openrouter/chat_models.py:1221-1225`: "Streaming deltas tag each fragment with the `index` of the entry it belongs to in the original (non-streamed) array, so this function groups consecutive entries by `(type, index)` and merges each group into one."

In rig's recordings the indexes are consistent with that reading: Claude text fragments and their signature fragment all have `index: 0`, OpenAI summary fragments have `index: 0` and the encrypted entry `index: 1`. rig's recorded replays of arrays merged by type and index returned 200 (`streaming_signed_agent_roundtrip.yaml` #1, `reasoning_roundtrip/streaming.yaml` #1). So the first-wave implication "OpenRouter ... does not give the client a reliable key for reassembly" goes further than the code shows. What is shown: the SDK does not trust `index`, two other clients do, and all three agree that fragments must be merged before replay.

**F6.** `src/chat/convert-to-openrouter-chat-messages.ts:69-73`, declared once, outside the per-message loop:

```
  // Track reasoning_details across all messages in this conversion to prevent duplicates.
  // This fixes issue #254 where the same reasoning ID appears in multiple
  // assistant messages during multi-turn conversations, causing the API
  // to reject the request with "Duplicate item found with id" error.
  const reasoningDetailsTracker = new ReasoningDetailsDuplicateTracker();
```

Use at `:377-382`: `if (reasoningDetailsTracker.upsert(detail)) { uniqueDetails.push(detail); }`. Keys, `src/utils/reasoning-details-duplicate-tracker.ts:13-17`: "The canonical key logic matches the OpenRouter API's deduplication exactly (see openrouter-web/packages/llm-interfaces/reasonings/duplicate-tracker.ts): - Summary: key = summary field - Encrypted: key = id field (if truthy) or data field - Text: key = text field (if truthy) or signature field (if truthy)". `upsert` returns false when the key is null, so a text entry with empty text and empty signature is removed too.

**F7.** Same file `:344-356`:

```
          const validDetails = candidateReasoningDetails.filter((detail) => {
            if (detail.type !== ReasoningDetailType.Text) {
              return true;
            }
            const format = detail.format ?? DEFAULT_REASONING_FORMAT;
            if (
              format !== ReasoningFormat.AnthropicClaudeV1 &&
              format !== ReasoningFormat.GoogleGeminiV1
            ) {
              return true;
            }
            return !!detail.signature;
          });
```

`DEFAULT_REASONING_FORMAT = ReasoningFormat.AnthropicClaudeV1` (`src/schemas/format.ts:12`). Lines `:402-405`: `reasoning && finalReasoningDetails && finalReasoningDetails.length > 0 ? reasoning : undefined`. The filter runs before the de-duplication ("This runs BEFORE deduplication so that signatureless entries are never registered in the tracker").

**F8, overstated.** The comment (`:327-333`) says "Since the SDK cannot verify whether the text is intact, Gemini reasoning.text entries are always stripped on roundtrip". The filter quoted under F7 does something narrower: a `google-gemini-v1` text entry is removed when `detail.signature` is empty and kept when it is set. The SDK's own tests say so (`src/chat/signature-roundtrip.test.ts:296-297`): `it('should preserve Google Gemini reasoning.text if it has an explicit signature', () => {` with the comment "Hypothetical future case: if Gemini ever adds SDK-level signatures,". That case is in rig's recorded traffic: a streamed `reasoning.text` entry from `google/gemini-3-flash-preview` with a 120-character `signature`, recorded 2026-10-02 (VF1). The SDK's stream handler would store that entry (as a new text entry, or merged into a preceding unsigned text entry at `index.ts:920-922` when one exists), the entry schema accepts it (`text: z.string().nullish()`, `signature: z.string().nullish()`), and on the next request it passes the filter and goes back. So the accurate statement is: unsigned Gemini reasoning text is removed, signed Gemini reasoning text is replayed.

**F13.** `src/schemas/format.ts:1-8` has six values: `'unknown'`, `'openai-responses-v1'`, `'azure-openai-responses-v1'`, `'xai-responses-v1'`, `'anthropic-claude-v1'`, `'google-gemini-v1'`. `src/schemas/reasoning-details.ts:5-9` has three types. The entry schema uses `format: z.enum(ReasoningFormat).nullish(),` (`:14`). The fallback (`:58-65`):

```
const ReasoningDetailsWithUnknownSchema = z.union([
  ReasoningDetailUnionSchema,
  z.unknown().transform(() => null),
]);

export const ReasoningDetailArraySchema = z
  .array(ReasoningDetailsWithUnknownSchema)
  .transform((d) => d.filter((d): d is ReasoningDetailUnion => !!d));
```

Where it is used: reply parsing, `src/chat/schemas.ts:55` and `:181` (`reasoning_details: ReasoningDetailArraySchema.nullish(),`), and replay parsing, `src/schemas/provider-metadata.ts:83-87`: "Use ReasoningDetailArraySchema (with unknown fallback) instead of z.array(ReasoningDetailUnionSchema) so that a single malformed entry (e.g., a future format not yet in the enum) is individually dropped rather than causing the entire array to fail parsing."

What the parser really does, by running those schema files on six hand-made entries (index 0 known Claude text entry, 1 encrypted entry with `format: "bedrock-openai-responses-v1"`, 2 text entry with `format: "openrouter-stealth-v1"`, 3 entry of type `reasoning.server_tool_call`, 4 summary entry with the `format` key left out, 5 encrypted entry with `format: null`):

```
A. ReasoningDetailArraySchema success=true kept indexes: [0,4,5]
B. non-stream response success=true kept indexes: [0,4,5]
C. stream chunk success=true kept indexes: [0,4,5]
D. provider options (replay parse) success=true kept indexes: [0,4,5]
E. provider metadata (strict array) success=false ["reasoning_details.1: invalid_union","reasoning_details.2: invalid_union","reasoning_details.3: invalid_union"]
```

So an entry with a format or type outside the SDK's lists is dropped by itself, the field and the other entries survive, and the parse succeeds, on the reply path, the stream path, and the replay path. Row E is the stricter `OpenRouterProviderMetadataSchema` (`provider-metadata.ts:40`), which the chat model calls only on entries that already passed rows B or C (`src/chat/index.ts:577`). A dropped `reasoning.encrypted` entry means the next request goes out without that blob, and the SDK raises nothing when it happens.

**F14.** From OpenRouter's OpenAPI spec (local mirror of `https://openrouter.ai/openapi.json`, read 2026-11-22): `components.schemas.ReasoningFormat.enum` is `'unknown', 'openai-responses-v1', 'azure-openai-responses-v1', 'bedrock-openai-responses-v1', 'bedrock-xai-responses-v1', 'xai-responses-v1', 'meta-responses-v1', 'google-vertex-meta-responses-v1', 'google-vertex-xai-responses-v1', 'openrouter-stealth-v1', 'anthropic-claude-v1', 'google-gemini-v1', null` with `'x-speakeasy-unknown-values': 'allow'`. That is 12 strings plus null. `ReasoningDetailUnion.discriminator.mapping` has four keys: `reasoning.encrypted`, `reasoning.server_tool_call`, `reasoning.summary`, `reasoning.text`.

## VF7. F40 - pi routes Claude on OpenRouter to the Messages route

Verdict: confirmed with a correction.

`earendil-works/pi` at `9c465268`, `packages/ai/catalog/sources/openrouter.ts:61-79`:

```
	for (const model of listed) {
		// Only include models that support tools
		if (!model.supported_parameters?.includes("tools")) continue;
		...
		// Includes the `~anthropic/claude-*-latest` aliases, which OpenRouter's Messages endpoint accepts.
		const useAnthropicMessages = /^~?anthropic\//.test(model.id) && !model.id.endsWith(":batch");
		chat.push({
			type: "chat",
			id: model.id,
			name: model.name,
			api: useAnthropicMessages ? "anthropic-messages" : "openai-completions",
			baseUrl: useAnthropicMessages ? "https://openrouter.ai/api" : "https://openrouter.ai/api/v1",
			provider: "openrouter",
```

The reason, from the changelog under `## [1.5.0] - 2026-11-02` (`packages/ai/CHANGELOG.md:164`): "Fixed OpenRouter's `~anthropic/claude-*-latest` aliases losing Claude's earlier thinking after each tool call. They used the Chat Completions API, where pi dropped the thinking signature, and OpenRouter then ignored the unsigned thinking. They now use the Anthropic Messages API with adaptive thinking, like the versioned Claude models".

The correction: "every `anthropic/*` model" has two limits. An id that ends in `:batch` stays on `openai-completions`, and only models whose `supported_parameters` include `tools` enter the catalog at all. For the chat-completions path the provider file still marks Claude ids (`packages/ai/catalog/providers/openrouter.ts:76`): `...(/^~?anthropic\//.test(model.id) ? { cacheControlFormat: "anthropic" } : {}),`.

## VF8. F43 - Cline's path to OpenRouter

Verdict: confirmed.

`cline/cline` at `8075d2e3`. The provider entry (`sdk/packages/llms/src/providers/builtins.ts:1074-1091`): `id: "openrouter"`, `family: "openai-compatible"`, `defaults: { baseUrl: "https://openrouter.ai/api/v1" }`, `metadata: { ...ANTHROPIC_AND_QWEN_CACHE_ROUTING_METADATA, ...OPENROUTER_STICKY_SESSION_METADATA, imageTransport: "openrouter", }`. The family dispatch (`ai-sdk.ts:2078-2082`): `case "openai-compatible": {` ... `return createOpenAICompatibleProviderModule(config, context);`.

Inside that module (`vendors/openai-compatible.ts:243-268`, `:293-298`):

```
	const provider = createOpenAICompatible({
		name: context.provider.id,
		apiKey,
		...
		includeUsage: true,
		transformRequestBody: withMaxCompletionTokensForReasoningModels,
	} as never);
	const useOpenRouterImageTransport =
		context.provider.metadata?.imageTransport === "openrouter" &&
		modelProducesImages(context.model);
	...
			language: (modelId) =>
				wrapLanguageModel({
					model: (openRouterImageProvider?.chat(modelId) ??
						provider(modelId)) as LanguageModelV4,
```

So a text model goes through `@ai-sdk/openai-compatible` (the package Cline pins as `^3.0.67`), and a model that produces images goes through `@openrouter/ai-sdk-provider`.

How I searched for `reasoning_details` handling: `rg -c "reasoning_details"` over `sdk/packages` with `node_modules` excluded. Hits: the generated catalog (102), a recorded test JSON (1), `sdk/packages/shared/src/llms/model-info.ts:74` (the enum `.enum(["reasoning_content", "reasoning_details"])` for `interleavedReasoningField`), and `sdk/packages/llms/src/catalog/catalog-live.ts:333` (copies that catalog value). I then traced `interleavedReasoningField`: the only branch that reads its value is `ai-sdk.ts:403`, `context.model.metadata?.interleavedReasoningField === "reasoning_content"`. No code in `sdk/packages` writes a `reasoning_details` key into a request. The same count over `vercel/ai` at `d55f3cfc`, `packages/openai-compatible` (version 3.0.94), returns zero files.

History replay (`ai-sdk.ts:766-789`) sends a stored reasoning part as `{ type: "reasoning", text, providerOptions: { anthropic: { signature, redactedData } } }`. The options are filed under the `anthropic` key. OpenRouter's own provider reads `providerOptions.openrouter.reasoning_details` (VF6, `OpenRouterProviderOptionsSchema`), so on the image-model path too the stored signature stays out of the request's `reasoning_details`. The first wave missed the `model-info.ts:74` hit, which changes nothing.

## VF9. H42 - LangChain removes `rs_` ids

Verdict: confirmed.

`langchain-ai/langchain` at `8ff3cd30`, `libs/partners/openrouter/langchain_openrouter/chat_models.py:1196-1206`:

```
def _strip_ephemeral_reasoning_ids(value: Any) -> Any:
    """Remove OpenAI Responses reasoning item IDs from outbound payloads."""
    if isinstance(value, list):
        return [_strip_ephemeral_reasoning_ids(item) for item in value]
    if isinstance(value, dict):
        return {
            key: _strip_ephemeral_reasoning_ids(item)
            for key, item in value.items()
            if not (key == "id" and isinstance(item, str) and item.startswith("rs_"))
        }
    return value
```

Its one call site (`:1327-1330`): `message_dict["reasoning_details"] = _strip_ephemeral_reasoning_ids(_merge_reasoning_details(message.additional_kwargs["reasoning_details"]))`. It removes the `id` key and keeps the entry with its `data`. rig does the opposite and sends the `rs_` id back (VF3), and both shapes are in use. rig's shape is the one with a recorded 200.

## VF10. K10, K12, K13 - LiteLLM and Gemini thought signatures

Verdicts: K10 confirmed with a correction. K12 confirmed. K13 confirmed with a correction to the one-line summary.

All from `BerriAI/litellm` at `90dce4b9`.

**K10.** The separator (`litellm/litellm_core_utils/prompt_templates/factory.py:103`): `THOUGHT_SIGNATURE_SEPARATOR: Final = "__thought__"`. The encoder (`factory.py:1261-1277`): docstring "Embed thought signature into tool call ID for OpenAI client compatibility." and "Format: call_<uuid>__thought__<base64_signature>", body `return f"{tool_call_id}{THOUGHT_SIGNATURE_SEPARATOR}{thought_signature}"`.

Where the reply is built (`litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py:2535-2552`):

```
                    _tool_response_chunk: ChatCompletionToolCallChunk = {
                        "id": f"call_{uuid.uuid4().hex[:28]}",
                        "type": "function",
                        "function": _function_chunk,
                        "index": cumulative_tool_call_idx,
                    }
                    # Gemini 3.5+ returns a stable native `id`; prefer it over
                    # the synthetic call_<uuid> so the same value can be echoed
                    # back on the matching `functionResponse`.
                    if gemini_call_id:
                        _tool_response_chunk["id"] = gemini_call_id
                    if thought_signature:
                        _tool_response_chunk["provider_specific_fields"] = {"thought_signature": thought_signature}
                    if thought_signature and thought_signature_in_tool_call_id:
                        _tool_response_chunk["id"] = encode_tool_call_id_with_signature(
                            _tool_response_chunk["id"] or "", thought_signature
                        )
```

The correction: the id is `call_<28 hex characters>__thought__<signature>` only when Gemini sent no id of its own. When it did, the id is `<Gemini's id>__thought__<signature>`. The same signature is also written to `provider_specific_fields.thought_signature` on the tool call, so the id is one of two places. The parameter default is `thought_signature_in_tool_call_id: bool = True` (line 2507), read from deployment settings by `embeds_thought_signature_in_tool_call_id` (`litellm/litellm_core_utils/model_param_helper.py:49-51`).

**K12.** The lookup order (`factory.py:1307-1359`): `tool.provider_specific_fields["thought_signature"]`, then `tool.function.provider_specific_fields["thought_signature"]`, then the id: `if tool_call_id and THOUGHT_SIGNATURE_SEPARATOR in tool_call_id:` ... `return signature if _is_strict_base64(signature) else None`. The placeholder (`:1365-1379`): `dummy_data: Final = b"skip_thought_signature_validator"`, returned base64 encoded. Its use (`:1437`, `:1449-1457`):

```
        needs_dummy_signature: Final = model is not None and VertexGeminiConfig.is_gemini_3_or_newer(model)
        ...
                        thought_signature = get_thought_signature_from_tool(dict(tool))
                        # Gemini signs only the first functionCall part of a parallel batch, so scope the
                        # placeholder fallback to that part instead of fabricating one per sibling call:
                        ...
                        is_first_function_call = len(_parts_list) == 0
                        if not thought_signature and is_first_function_call and needs_dummy_signature:
                            thought_signature = get_dummy_thought_signature()
```

The id sent to Gemini has the signature removed (`:1254-1257`): `clean_id: Final = tool_call_id.split(THOUGHT_SIGNATURE_SEPARATOR, 1)[0]`.

**K13.** The whole of `litellm/llms/vertex_ai/gemini/rejected_thought_signature_retry.py` (235 lines) was read. The trigger (`:17`, `:22-26`):

```
_REJECTED_SIGNATURE_MARKER: Final = "thought signature"
...
def names_rejected_thought_signature(status_code: int, message: str) -> bool:
    """A signature only validates on the backend that minted it. AI Studio answers a replayed Vertex signature
    with ``400 Corrupted thought signature.`` and Vertex answers an AI Studio one with ``400 Invalid thought
    signature.``"""
    return status_code == 400 and _REJECTED_SIGNATURE_MARKER in message.lower()
```

The resend (`:62-68`): "Swaps every thought signature in the history for Google's documented skip-validator placeholder, or returns ``None`` when the body carries nothing the backend could have rejected." The log line (`:113-116`): "Gemini rejected a replayed thought signature, resending with the skip_thought_signature_validator placeholder in place of the replayed signatures". It is wired into the Gemini chat path at `vertex_and_google_ai_studio_gemini.py:3867-3900`, `:3980-4025`, `:4175-4258`, and into the native `generateContent` pass-through at `litellm/llms/custom_httpx/llm_http_handler.py:12779` and `:12886`. When a `cachedContent` is in play there can be a second resend with the history sent inline (`:193-211`).

The non-Gemini case (`litellm/utils.py:1146-1167`): "Messages with Gemini thought signatures cut from tool call ids, or None when the call's target keeps them." and `verbose_logger.debug("Removing thought signatures from tool call IDs for non-Gemini model %s", model)`.

The correction to the one-line summary ("on a model or backend switch it resends with Google's placeholder signature after a 400"): three separate things happen.

- A backend switch between Vertex and AI Studio is not detected ahead of time. The request goes out with the real signature, and the resend fires on any 400 whose message contains "thought signature".
- A switch to a Gemini 3+ model from history that has no signature gets the placeholder on the first call of each message up front, with no 400 involved.
- A switch to a non-Gemini model gets the signature cut out of the tool call ids, and no placeholder.

The first-wave item K13 says the same three things and is accurate as written.

## VF11. K16, K17, K19 - LiteLLM's chat-to-Responses bridge

Verdicts: K16 confirmed. K17 confirmed. K19 confirmed, and the first wave's doubt is settled.

**K16.** `litellm/completion_extras/litellm_responses_transformation/transformation.py:273-276` and `:1440-1450`:

```
_PREVIOUS_RESPONSE_ID_UNSUPPORTED: Final = (
    "`previous_response_id` is only supported on the Responses API (`/v1/responses`). Continue the stored "
    "response there, or send the earlier turns in `messages`."
)
...
        if any(params.get("previous_response_id") is not None for params in sent_params):
            raise litellm.UnsupportedParamsError(
                message=_PREVIOUS_RESPONSE_ID_UNSUPPORTED, model=model, llm_provider=_served_provider(litellm_params)
            )
```

`sent_params` is the request's optional parameters, their `extra_body`, the deployment parameters, and their `extra_body`, so the parameter is refused wherever the caller puts it.

**K17.** `history.py:11`: `REPLAY_FIELD: Final = "openai_responses_replay"`. The stored shape (`:58-75`): a `_MessageEntry` has `type: Literal["message"]`, `parts` (a tuple of integers, one text length per part), and `phase`. An `_IndexedEntry` has `type` (one of `"reasoning"`, `"function_call"`, `"custom_tool_call"`, `"tool_search_call"`, `"tool_search_output"`) and `index`. `_Replay` has `version: Literal[1]`, `items: tuple[_Entry, ...]`, and `checksum: str`. The checksum (`:228-242`):

```
    bound: Final = {
        "content": message.content or "",
        "tool_calls": [[call.id, call.function.name, call.function.arguments] for call in message.tool_calls or ()],
        "reasoning": [
            [item.id, item.encrypted_content, [part.text for part in item.summary or ()]]
            for item in message.reasoning_items or ()
        ],
        "tool_search_items": list((fields.tool_search_items if fields else None) or ()),
        "phase": message.phase or (fields.phase if fields else None),
        "items": [entry.model_dump(mode="json", exclude_none=True) for entry in entries],
    }
    encoded: Final = json.dumps(bound, sort_keys=True, separators=(",", ":"), ensure_ascii=False)
    return hashlib.sha256(encoded.encode("utf-8", "surrogatepass")).hexdigest()
```

Where each part lands on the reply: `reasoning_items` is an argument of `Message(...)` (`transformation.py:799-806`), and the map is merged into `provider_specific_fields` (`transformation.py:813-821`, `_with_replay_fields`). On the next request the map is honoured only when the message is unchanged (`history.py:449-456`): `if chat is None or stored is None or stored.checksum != _checksum(chat, stored.items): return None`. The map holds positions and text lengths, and the content stays in the normal fields. `_HISTORY_REPLAY_PROVIDERS: Final = frozenset({"openai", "azure"})` (`transformation.py:271`).

**K19.** How I searched: `rg "reasoning\.encrypted_content" litellm`. Python hits: `litellm/llms/anthropic/messages/responses_adapters/transformation.py:66` and `:625`, `litellm/router_utils/pre_call_checks/encrypted_content_affinity_check.py:4` (a docstring), and `litellm/llms/chatgpt/responses/transformation.py:147-148`. The bridge folder `litellm/completion_extras/` has zero hits. I also searched `litellm/completion_extras`, `litellm/responses`, `litellm/llms/openai/responses`, and `litellm/llms/azure/responses` for any assignment to an `include` key: the only matches are the list-input-items query and function signatures that pass a caller's `include` along.

How a caller's value gets through (`transformation.py:1373-1374`): `elif key in ResponsesAPIOptionalRequestParams.__annotations__:` / `responses_api_request[key] = value`, and `include: list[ResponseIncludable] | None` is one of those annotations (`litellm/types/llms/openai.py:1407`). A value inside `extra_body` is lifted to the top level first (`transformation.py:2140-2162`). The bridge also leaves `store` to the caller.

So for `openai` and `azure` the caller has to send `include: ["reasoning.encrypted_content"]` to get `encrypted_content` back. Without it the reasoning items that come back have `id` and `summary` (`history.py:197-205` writes `encrypted_content` only when it is set). What OpenAI does with a replayed reasoning item that has an `id` and no `encrypted_content` is outside this code, and I did not check it against OpenAI's documentation in this pass.

One provider does it itself, which the first wave listed as a search hit without saying what it does (`litellm/llms/chatgpt/responses/transformation.py:144-149`):

```
        request["store"] = False
        request["stream"] = True
        include: Final = list(request.get("include") or [])
        if "reasoning.encrypted_content" not in include:
            include.append("reasoning.encrypted_content")
        request["include"] = include
```

That is the `chatgpt` provider's Responses transform. I read the transform and did not trace a bridge call into it.

## VF12. K6, K7, K72 - LiteLLM and Anthropic block order, and the stream rebuilder

Verdicts: K6 confirmed. K7 confirmed. K72 confirmed with a correction, and the open point is settled.

**K6.** `litellm/llms/anthropic/chat/transformation.py:2969-3026`: `if content["type"] == "text": text_content += content["text"]`, then `tool_use` and `server_tool_use` blocks go to `tool_calls`, `*_tool_result` blocks go to `web_search_results` or `tool_results`, `thinking_blocks.append(cast(ChatCompletionThinkingBlock, content))`, `thinking_blocks.append(cast(ChatCompletionRedactedThinkingBlock, content))`, `compaction_blocks.append(content)`. Then `reasoning_content` is the `thinking` text of every block joined with no separator. Each kind of block lands in its own list, and the position of a block among the other kinds is not recorded.

**K7.** `litellm/litellm_core_utils/prompt_templates/factory.py`. Compaction first (`:3526-3527`). The interleave branch is entered when (`:3575-3579`) `thinking_blocks is not None and _has_server_tool_calls and isinstance(assistant_content_block.get("content", None), (str, type(None)))`, where server tool calls are recognised by `_tc_id.startswith("srvtoolu_")` (`:3545`). The rule (`:3626-3629`):

```
                # Interleave: each thinking block precedes its server tool group.
                # Pattern: thinking[0], group[0], thinking[1], group[1], ...
                # Any remaining thinking blocks (after all groups) go before text.
                # Any remaining groups (after all thinking blocks) go after.
```

followed by the text block (`:3663-3665`) and then "Add regular tool calls, then a paused turn's unrun server tool call, at the end" (`:3667`). Otherwise (`:3688-3689`): `if not _content_is_list:  # IMPORTANT: ADD THIS FIRST, ELSE ANTHROPIC WILL RAISE AN ERROR` / `assistant_content.extend(replayed_thinking_blocks)`, then text, then tool uses (`:3738-3750`).

One addition: when the caller sends assistant `content` as a list, the code walks that list in the caller's order (`:3690-3717`, `for m in _with_missing_thinking_blocks(_content_list, replayed_thinking_blocks):`), handling thinking, text, `server_tool_use`, and `*_tool_result` items where they stand. So the fixed pattern is what applies to the message shape LiteLLM itself returns (string content plus separate lists). The comment's own example order (`:3582-3583`: "[thinking_1, server_tool_use_1, result_1, thinking_2, text, server_tool_use_2, result_2, ...]") puts text between two tool groups, and the rebuild puts text after all groups, as the first wave noted.

**K72.** The per-delta fragments (`litellm/llms/anthropic/chat/handler.py:878-899`):

```
        elif "thinking" in content_block["delta"] or "signature" in content_block["delta"]:
            thinking_content: Final = content_block["delta"].get("thinking")
            if isinstance(thinking_content, str) and thinking_content:
                ...
                thinking_blocks = [
                    ChatCompletionThinkingBlock(
                        type="thinking",
                        thinking=thinking_content,
                        index=self.thinking_block_index,
                    )
                ]
                provider_specific_fields["thinking_blocks"] = thinking_blocks

            signature: Final = content_block["delta"].get("signature")
            if isinstance(signature, str) and signature:
                thinking_blocks = [
                    ChatCompletionThinkingBlock(
                        type="thinking", thinking="", signature=signature, index=self.thinking_block_index
                    )
                ]
```

The rebuilder, which settles the open point. `stream_chunk_builder` (`litellm/litellm_core_utils/streaming_chunk_builder_utils.py:1843`, re-exported from `litellm/main.py:9932`) calls `_assembled_stream_response` (`:1851`), which calls `_assembled_message_fields` (`:1946`), which yields (`:1743-1744`) `"thinking_blocks", processor.get_combined_thinking_content(thinking_blocks)`. That method (`:806-862`):

```
        def _flush_thinking_block() -> None:
            nonlocal current_thinking_text_parts, current_signature
            if current_signature:
                thinking_blocks.append(
                    ChatCompletionThinkingBlock(
                        type="thinking",
                        thinking="".join(current_thinking_text_parts),
                        signature=current_signature,
                    )
                )
            current_thinking_text_parts = []
            current_signature = None
        ...
                        else:
                            thinking_text = thinking_block.get("thinking", None)
                            if thinking_text:
                                current_thinking_text_parts.append(thinking_text)
                            signature = thinking_block.get("signature", None)
                            if signature:
                                current_signature = signature
                                _flush_thinking_block()
```

So LiteLLM's own rebuilder does merge the fragments into whole signed blocks. Three details: it closes a block when a signature arrives and never reads `index`; the merged block has the keys `type`, `thinking`, `signature`; and text that is never followed by a signature is thrown away at the final flush, because `_flush_thinking_block` appends only `if current_signature`. A `redacted_thinking` fragment is appended whole. The correction to the first-wave wording ("a streaming caller has to merge fragments into whole signed blocks before it can send them back"): that is true for a caller that consumes raw chunks, and a caller that passes the chunks to `stream_chunk_builder` gets the merged blocks.

## VF13. K24, K49, K32, K35 - LiteLLM extension fields, unsupported parameters, Gemini caching, usage

Verdicts: K24 confirmed. K49 confirmed. K32 confirmed. K35 overstated.

**K24.** The field sets (`litellm/litellm_core_utils/prompt_templates/prompt_cache_breakpoint_utils.py:218-223`): `LITELLM_RESPONSE_ONLY_MESSAGE_FIELDS: Final = frozenset({"thinking_blocks", "reasoning_items", "provider_specific_fields"})` and `LITELLM_INTERNAL_MESSAGE_FIELDS: Final = LITELLM_RESPONSE_ONLY_MESSAGE_FIELDS | {"reasoning_content"}`. The helpers (`common_utils.py:1359-1367`): "Drop the fields litellm attaches to a returned message (e.g. Anthropic thinking blocks or Responses API items) that no OpenAI-compatible endpoint accepts back, so a reply can be sent to a strict one." They also remove `provider_specific_fields` from each tool call (`:1342-1356`).

How I searched: `rg "strip_litellm_response_only_fields|strip_litellm_internal_message_fields" litellm -g "*.py"`. Callers: `litellm/llms/friendliai/chat/transformation.py:38`, `fireworks_ai/chat/transformation.py:546`, `together_ai/chat/transformation.py:178`, `databricks/chat/transformation.py:749`, `mistral/chat/transformation.py:729`, `groq/chat/transformation.py:158`. Six, as recorded. A second search for inline removals found `litellm/llms/hosted_vllm/chat/transformation.py:184` (`message.pop("thinking_blocks", None)`), also as recorded.

The shared OpenAI path (`litellm/llms/openai/chat/gpt_transformation.py:476-488`, `:694-730`): `_transform_messages` runs `drop_tool_reference_parts_from_tool_messages`, `hoist_images_from_tool_messages`, and a per-message user-content rewrite, and `transform_request` returns `{"model": model, "messages": transformed_messages, **optional_params, ...}`. A search for `thinking_blocks`, `reasoning_items`, and `provider_specific_fields` over `litellm/llms/openai/chat`, `litellm/llms/openai/openai.py`, `litellm/llms/openai/common_utils.py`, `litellm/llms/openai_like/chat`, and `litellm/llms/openrouter/chat` finds one hit, in reply code (`gpt_transformation.py:858`). So a message that still has those keys is sent with them, and that includes a request LiteLLM sends to OpenRouter.

**K49.** `litellm/utils.py:5985-6013`:

```
        unsupported_params: Final[MappingProxyType[str, object]] = MappingProxyType(
            {
                k: non_default_params[k]
                for k in non_default_params
                if k not in supported_params and k not in PROVIDER_UNVALIDATED_PARAMS and not (k == "n" and n == 1)
            }
        )

        if unsupported_params:
            if litellm.drop_params is True or (drop_params is not None and drop_params is True):
                for k in unsupported_params:
                    non_default_params.pop(k, None)
                ...
            else:
                raise UnsupportedParamsError(
                    status_code=500,
                    message=f"{custom_llm_provider} does not support parameters: {list(unsupported_params.keys())}, for model={model}. To drop these, set `litellm.drop_params=True` or for proxy:\n\n`litellm_settings:\n drop_params: true`\n. \n If you want to use these params dynamically send allowed_openai_params={list(unsupported_params.keys())} in your request.",
                )
```

and `supported_params.extend(allowed_openai_param_names(allowed_openai_params))` at `:6031`. The check covers OpenAI-named parameters. The error's status code is 500.

**K32.** `litellm/llms/vertex_ai/context_caching/vertex_ai_context_caching.py:445-448`: `cached_messages, non_cached_messages = separate_cached_messages(messages=messages)` and a return with nothing done when the list is empty. The key (`:485-487`): `generated_cache_key: Final = local_cache_obj.get_cache_key(messages=cached_messages, tools=tools, tool_choice=tool_choice, model=target.model)`. `get_cache_key` hashes with sha256 (`litellm/caching/caching.py:586-609`, `:240-243`). The create body (`context_caching/transformation.py:190-194`): `CachedContentRequestBody(contents=transformed_messages, model=model_name, displayName=cache_key,)`. The lookup (`vertex_ai_context_caching.py:228`, `:293-302`): "Currently, checks cache list, for cache key == displayName, since Google doesn't let us set the name of the cache (their API docs are out of sync with actual implementation)." with `if display_name is None or display_name != cache_key: continue`. A process-local map is asked first (`:248`). The create call is a POST to the `cachedContents` endpoint (`:536-541`), and tools are moved into the cache body (`:505-506`, `:521`). What marks a message as cached is `cache_control` with `"type": "ephemeral"` on the message or a content part (`litellm/utils.py:9686-9707`).

**K35, overstated.** The code read:

- Anthropic (`litellm/llms/anthropic/chat/transformation.py:3186-3192`): `prompt_tokens += cache_creation_input_tokens` and `prompt_tokens += cache_read_input_tokens`. With server-tool iterations (`:3179-3184`): `prompt_tokens += cache_creation_input_tokens + cache_read_input_tokens`. Details (`:3217-3223`): `cached_tokens=cache_read_input_tokens`, `cache_creation_tokens=cache_creation_input_tokens`, `text_tokens=raw_input_tokens`.
- Bedrock Converse (`litellm/llms/bedrock/chat/converse_transformation.py:3021`): `input_tokens: Final = raw_input_tokens + cache_read_input_tokens + cache_creation_input_tokens`.
- Gemini (`vertex_and_google_ai_studio_gemini.py:2832-2840`): `prompt_tokens=usage_metadata.get("promptTokenCount", 0) + billable_tool_use_prompt_tokens + cache_creation_tokens`, with `cache_read_input_tokens=cached_tokens`. Here `cache_creation_tokens` is LiteLLM's own count from the `cachedContents` object it created for this call.

That is three vendors in code (the first wave adds DeepSeek's field mapping). The phrase "for every vendor" rests on LiteLLM's documentation sentence quoted in the first wave, and neither wave read the usage code of the other providers. A file count shows where cached-token fields are handled under `litellm/llms`: the Anthropic, Bedrock, and Vertex Gemini folders, plus cost calculators for `dashscope` and `perplexity` and a few realtime transforms. The narrower statement that holds: for the vendors whose native usage reports cache tokens outside the input count (Anthropic and Bedrock), LiteLLM adds them into `prompt_tokens`.

## VF14. J8 and J34 - agentgateway's single signature, and tool-call ids

Verdicts: J8 confirmed. J34 confirmed.

**J8.** `agentgateway/agentgateway` at `888badbb`, `crates/llm/src/conversion/messages.rs:955-970` and `:980-989`:

```
				// Chat Completions carries one reasoning text, so several blocks are joined. The signature
				// attests to one block and only survives a response with exactly one.
				messages::ContentBlock::Thinking {
					thinking,
					signature,
				} => {
					match reasoning_content.as_mut() {
						Some(text) => {
							text.push_str("\n\n");
							text.push_str(&thinking);
						},
						None => reasoning_content = Some(thinking),
					}
					reasoning_signatures.push(signature);
				},
				messages::ContentBlock::RedactedThinking { .. } => {},
...
			reasoning_signature: match reasoning_signatures.as_slice() {
				[signature] if !signature.is_empty() => Some(signature.clone()),
				_ => None,
			},
```

With two or more thinking blocks the reply has the joined text and `reasoning_signature` is `None`. A redacted block produces nothing. Text blocks are concatenated into one `content` string (`:926-930`).

**J34.** The chat-to-Messages request path (`messages.rs:479-489`): `out.push(messages::ContentBlock::ToolUse { id: call.id.clone(), name: call.function.name.clone(), input, cache_control: None, });`. Tool results (`:731`): `tool_use_id: tool.tool_call_id.clone(),`. The reply path (`:940-942`): `id: id.clone(),`.

How I searched for a rewrite: `rg -i "sanitiz|normalize.*id|rewrite.*id|valid.*tool.*id|\^\[a-zA-Z0-9_-\]|tool_id"` over `crates/llm/src` (tests excluded) and `rg -i "tool_call_id|tool_use_id|call\.id|sanitiz"` over `crates/agentgateway/src/llm`. The only sanitizers found work on Bedrock tool names (`conversion/bedrock.rs:115-178`) and on document names (`:2657`), plus Google Model Armor content checks. Nothing rewrites a tool-call id on the Anthropic path. So at this commit the id goes to Anthropic as the caller sent it, and the 2026-11-17 workspace report's credit to agentgateway does not match this commit. Whether it matched the older commit that report read (`bdb36de0`) I could not check, since the clone is shallow.

Bifrost's rewrite (`maximhq/bifrost` at `cae639c5`, `core/providers/utils/utils.go:46-85`):

```
var anthropicUnsafeToolUseIDCharRegex = regexp.MustCompile(`[^A-Za-z0-9_-]+`)
...
const maxSanitizedAnthropicToolUseIDLen = 64
...
	hash := fmt.Sprintf("%016x", xxhash.Sum64String(id))
	semantic := strings.Trim(anthropicUnsafeToolUseIDCharRegex.ReplaceAllString(id, "_"), "_")
	...
	return hash + "_" + semantic
```

with the comment "Some upstream providers (e.g. Kimi/Gemini-compatible backends) emit ids containing ':' or '.', which Anthropic's API rejects with a 400 when such a conversation is replayed through the Anthropic provider." An id that already fits the pattern is returned as is. Used at `core/providers/anthropic/chat.go:1152`: `ID: providerUtils.SanitizeAnthropicToolUseIDPtr(toolCall.ID),`.

The other two gateways on the same path, read while checking claim 16: Portkey `id: toolCall.id,` (`src/providers/anthropic/chatComplete.ts:168`) and Envoy `ID: *toolCall.ID,` (`internal/translator/anthropic_helper.go:472`). So among these four, Bifrost is the one that rewrites. Scope note: this is the Anthropic path. On agentgateway's Gemini path the first wave records made-up ids for calls that arrive with none (J12), which I did not re-read.

## VF15. J27, J31, J38 - Bifrost's `reasoning_details` and its handling of encrypted reasoning

Verdicts: J27 confirmed. J31 confirmed. J38 confirmed (the strip-and-retry part is the first wave's J39).

**J27.** `core/schemas/chatcompletions.go:1962-1980`:

```
const (
	BifrostReasoningDetailsTypeSummary       BifrostReasoningDetailsType = "reasoning.summary"
	BifrostReasoningDetailsTypeEncrypted     BifrostReasoningDetailsType = "reasoning.encrypted"
	BifrostReasoningDetailsTypeText          BifrostReasoningDetailsType = "reasoning.text"
	BifrostReasoningDetailsTypeContentBlocks BifrostReasoningDetailsType = "reasoning.content_blocks"
)

// Not in OpenAI's spec, but needed to support inter provider reasoning capabilities.
type ChatReasoningDetails struct {
	ID        *string                     `json:"id,omitempty"`
	Index     int                         `json:"index"`
	Type      BifrostReasoningDetailsType `json:"type"`
	Summary   *string                     `json:"summary,omitempty"`
	Text      *string                     `json:"text,omitempty"`
	Signature *string                     `json:"signature,omitempty"`
	Data      *string                     `json:"data,omitempty"` // for encrypted data
}
```

The struct's JSON keys are `id`, `index`, `type`, `summary`, `text`, `signature`, `data`. A `format` key sent by a caller is discarded when Go decodes into this struct, so a `reasoning_details` array that passes through Bifrost loses OpenRouter's vendor tag. `rg "BifrostReasoningDetailsTypeContentBlocks|reasoning\.content_blocks" -g "*.go"` over the whole repository returns the declaration line only.

**J31.** `core/providers/gemini/chat.go:187-257`: a thought text part becomes `{Type: BifrostReasoningDetailsTypeText, Text}`; a signature on a regular text part becomes `{Type: BifrostReasoningDetailsTypeEncrypted, Signature}`; for a function call, "Embed thought signature into CallID if present (matches responses.go pattern)" with `callID = fmt.Sprintf("%s%s%s", callID, thoughtSignatureSeparator, encoded)` (base64url), and "Also add to reasoning details for backward compatibility" with `ID: schemas.Ptr(fmt.Sprintf("tool_call_%s", baseCallID))` (standard base64). The separator (`core/providers/utils/utils.go:41-44`): `const ThoughtSignatureSeparator = "_ts_"`. The base id is Gemini's own call id when it sent one and the function name otherwise (`:221-224`).

**J38 with J39.** The binding claim (`core/encryptedreasoning.go:199-205`): "encrypted_content is bound to the identity that minted it: the item id it was issued with, the API key's organization, and the serving endpoint." and "The ciphertext arrives byte-perfect and is still undecryptable, and no amount of retrying the same payload will fix it." The gate (`:177-189`):

```
func shouldStripReasoningAfterClientError(err *schemas.BifrostError) bool {
	if err == nil || err.Error == nil || err.StatusCode == nil || *err.StatusCode != 400 {
		return false
	}
	message := strings.ToLower(err.Error.Message)
	if strings.Contains(message, "cannot be modified") {
		return false
	}
	if containsAnyMarker(message, reasoningConfigParams) && !namesEncryptedReasoningField(message) {
		return false
	}
	return containsAnyMarker(message, reasoningTokenWords)
}
```

The retry (`core/bifrost.go:7933-7961`): "Fail soft on a 400 when the request replays encrypted reasoning. The ciphertext is bound to the identity that minted it (item id, organization, serving endpoint, model family), and a gateway routinely changes that between turns" ... "Runs once per request." The code (`:7948-7956`):

```
		lastWasEncryptedContentStrip = false
		if !shouldRetry && !strippedEncryptedContent && shouldStripReasoningAfterClientError(bifrostError) &&
			stripUnverifiableReasoning(ctx, req) {
			strippedEncryptedContent = true
			lastWasEncryptedContentStrip = true
			extraAttempts++
			shouldRetry = true
			if isEncryptedReasoningRejection(bifrostError) {
				logger.Warn("upstream rejected replayed encrypted reasoning content for %s/%s; retrying once without it: %s", providerKey, model, errMessage)
```

The strip covers chat requests too: `stripChatUnverifiableReasoning` "clears signatures and encrypted payloads" (`core/encryptedreasoning.go:306-316`).

The id-inside-a-signature part that the first wave filed as J38 (`core/providers/utils/utils.go:107-172`): `const ReasoningItemIDMarker = "brid_"`, `const ReasoningItemIDSeparator = ":"`, `return ReasoningItemIDMarker + *id + ReasoningItemIDSeparator + payload`, with the comment "OpenAI cryptographically binds encrypted_content to the id it was issued with and rejects a replay whose id doesn't match." and the quoted refusal "Encrypted content item_id did not match the target item id."

## VF16. J9, J30, J74, J86, J100, J87 - block order in four gateways

Verdicts: J9 confirmed. J30 confirmed. J74 confirmed. J86 confirmed. J87 confirmed. J100 confirmed with a correction about Envoy.

**agentgateway (J9).** `crates/llm/src/conversion/messages.rs:432-503`: "An earlier thinking block is replayed ahead of the turn's text and tool calls, as the provider requires, and only with the signature it was issued with: an unsigned block is rejected." Then `if let Some(signature) = msg.reasoning_signature.as_deref().filter(|s| !s.is_empty()) { out.push(messages::ContentBlock::Thinking { thinking: msg.reasoning_content.clone().unwrap_or_default(), signature: signature.to_string(), }); }`, then content text parts, then `refusal`, then one `ToolUse` per call.

**Bifrost (J30).** `core/providers/anthropic/chat.go:1101-1175`: "First add reasoning details", then the loop over `msg.ChatAssistantMessage.ReasoningDetails` (an encrypted entry with `data` becomes `redacted_thinking`, every other entry becomes `thinking` with `Signature: reasoningDetail.Signature, Thinking: reasoningDetail.Text`), then `msg.Content`, then "Convert tool calls". The loop does not look at where an entry came from.

**Portkey (J74).** `Portkey-AI/gateway` at `669825cb`, `src/providers/anthropic/chatComplete.ts:141-177`:

```
  let inputContent: ContentType[] | string | undefined =
    msg.content_blocks ?? msg.content;
  ...
    inputContent.forEach((item) => {
      if (item.type !== 'tool_use') {
        transformedContent.push(item as AnthropicContentItem);
      }
    });
  }
  if (containsToolCalls) {
    msg.tool_calls.forEach((toolCall: any) => {
      transformedContent.push({
        type: 'tool_use',
```

and on the reply (`:596-601`): `...(!strictOpenAiCompliance && { content_blocks: response.content.filter((item) => item.type !== 'tool_use'), }),`. So with strict mode off the thinking and text blocks keep Anthropic's order among themselves, and tool uses move to the end.

**Envoy AI Gateway (J86, J87, and the correction to J100).** Repository `theagentrouter/agent-router` (formerly `envoyproxy/ai-gateway`) at `9cf85059`, `internal/translator/anthropic_helper.go`.

Reply, non-streaming (`:1447-1488`): a `tool_use` block is appended to `choice.Message.ToolCalls`; a text block is taken only `if choice.Message.Content == nil { choice.Message.Content = &output.Text }`; a thinking block does `choice.Message.ReasoningContent = &openai.ReasoningContentUnion{...ReasoningText: &awsbedrock.ReasoningTextBlock{Text: output.Thinking, Signature: output.Signature}...}` by plain assignment; a redacted block assigns the same field with `RedactedContent: []byte(output.Data)`. So the reply has the first text block, every tool call, and the last thinking or redacted block.

Reply, streaming (`:1232-1240`, `:1288-1299`): a thinking block start sends `Content: emptyStrPtr`; a redacted thinking block start returns nothing under "This is a latency-hiding event, ignore it."; and

```
		case string(constant.ValueOf[constant.TextDelta]()), string(constant.ValueOf[constant.ThinkingDelta]()):
			// Treat thinking_delta just like a text_delta, but read the field
			// that belongs to the variant: RawContentBlockDeltaUnion.Text is
			// only populated for text_delta, and .Thinking only for
			// thinking_delta.
			text := event.Delta.Text
			if event.Delta.Type == string(constant.ValueOf[constant.ThinkingDelta]()) {
				text = event.Delta.Thinking
			}
			delta := openai.ChatCompletionResponseChunkChoiceDelta{Content: &text}
```

The switch has that case and `InputJSONDelta`. How I searched for a signature case: `rg "SignatureDelta|signature_delta" internal/translator -g "*.go" -g "!*_test.go"`. Hits are in `anthropic_awsbedrock.go:745` and `openai_helper.go:686-1046`, which build Anthropic-shaped streams for the opposite direction. This file has zero hits, so on this path a signature delta produces no chunk and thinking text is mixed into `content`.

Request (`:441-488`), which is where the first wave's summary line is off:

```
func openAIMessageToAnthropicMessageRoleAssistant(openAiMessage *openai.ChatCompletionAssistantMessageParam) (anthropicMsg anthropic.MessageParam, err error) {
	contentBlocks := make([]anthropic.ContentBlockParamUnion, 0)
	if v, ok := openAiMessage.Content.Value.(string); ok && len(v) > 0 {
		contentBlocks = append(contentBlocks, anthropic.NewTextBlock(v))
	} else if content, ok := openAiMessage.Content.Value.(openai.ChatCompletionAssistantMessageParamContent); ok {
		contentBlocks, err = processAssistantContent(contentBlocks, content)
		...
	} else if contents, ok := openAiMessage.Content.Value.([]openai.ChatCompletionAssistantMessageParamContent); ok {
		for _, content := range contents {
			contentBlocks, err = processAssistantContent(contentBlocks, content)
```

`processAssistantContent` (`:405-437`) handles parts of type refusal, text, thinking ("Thinking content requires both text and signature"), and redacted thinking, appending each in the order the caller listed them. Tool calls are appended after the loop (`:459-483`). So Envoy has no "reasoning first" rule on the request: the caller's content parts decide where thinking sits, and tool uses go last. The statement in J100 that "agentgateway, Bifrost, and Envoy rebuild thinking first, then text, then tool calls" is right for the first two. For Envoy the accurate version is: the reply splits blocks into three fields and keeps one text block and one reasoning block, and the request follows the caller's part order with tool uses appended. The headline of J100 still holds: none of the four returns Anthropic's block list with tool uses in their original positions.

## VF17. H50 to H54 and H27 - rig's replay rules and Pydantic AI's two checks

Verdicts: all confirmed.

**rig** (`0xPlaygrounds/rig` at `a8673f77`).

- H50, `crates/rig-core/src/completion/message.rs:35-52`: "`content` holds one block per provider output item, in the order the provider produced them. `origin` names the wire, provider and model that produced the turn; a hand-built turn has none and always replays from its canonical fields." `message/native.rs:232-262`: "One provider item, verbatim in its API's JSON shape, beside the canonical block (or message) it was decoded to." with fields `item: serde_json::Value`, `fingerprint: Fingerprint`, `baseline`, `edited`, `blob_slot`. `native.rs:60-81`: the `Origin` struct has the fields `api`, `provider`, `model`, `response_model`, `response_id`, and `context`.
- H51, `native.rs:55-59`: "`model` is the model the request named. Replay compares it, with `api` and `provider`, against the target: only an exact match replays the turn's provider items." The comparison (`native.rs:99-101`) and its caller (`history.rs:1191-1195`):

```
    pub fn same_model(&self, api: &Api, provider: &str, model: &str) -> bool {
        &self.api == api && self.provider == provider && self.model == model
    }
```

```
        origin.same_model(&self.api, self.provider, self.model)
            && self
                .context
                .is_none_or(|context| origin.context == Some(context))
```

  `history.rs:465-475`: "A turn whose origin names `target`'s API, provider and model keeps its provider items." and "Any other turn keeps only canonical fields: provider items are cleared, reasoning becomes plain text, or stays reasoning when the request asks for it and the model reads it".
- H52, `native.rs:3-5`: "Its baseline or [`Fingerprint`] detects edits so an edited block stops replaying its stale item." `native.rs:132-138`: "A 64-bit FNV-1a hash of a value's JSON serialization with every object's keys in sorted order and every whole number written as an integer". `history.rs:398-408`: the `Replay` enum has the variants `Item`, `Identity`, and `Rebuild`, and the comment on `Identity` reads "The block was edited: rebuild it from its canonical fields, keeping these keys of its stale item."
- H53, `native.rs:74-80`: "The fingerprint of the request's tools and system prompt. A wire that binds its items to them ([`ReplayTarget::binds_context`]) replays a turn made under another context as if from another model." `history.rs:290-304`: `fn binds_context(&self, model: &str) -> bool` (default `false`) and `fn drops_unbound_items(...)` ("Anthropic's `drop_block`"). `providers/anthropic/wire.rs:862-865`: `self.provider.dialect.name == ANTHROPIC.name && super::completion::binds_context(&self.facts, model)`, backed by a per-model catalog flag (`providers/anthropic/completion.rs:132-136`, "Whether the Claude `model` binds its thinking blocks to the request's tools and system prompt.").
- H54, blob slots: Anthropic `&["/signature", "/data"]` (`providers/anthropic/wire.rs:887-889`), Gemini `&["/thoughtSignature"]` (`providers/gemini/completion.rs:315-317`), OpenAI Responses `&["/encrypted_content"]` (`providers/openai/responses_api/wire.rs:228-230`), chat wire `&["/thinking_blocks/*/signature", "/thinking_blocks/*/data"]` (`providers/openai/wire/chat.rs:1000-1002`). `needs_next` on the Responses wire: `item.str("type") == Some("reasoning")` (`responses_api/wire.rs:248-250`).

One link to VF2: rig's default for `binds_context` is `false`, the Anthropic wire turns it on per model, and the OpenRouter cassette test's first comment ends "So Chat does not bind context on OpenRouter." The recordings in VF2 are the evidence rig cites for leaving it off there.

**Pydantic AI** (`pydantic/pydantic-ai` at `463a6d51`), H27. `pydantic_ai_slim/pydantic_ai/_canonical_system.py:1-14`:

```
"""Which system can verify the stateless signed or encrypted data in a response.

Stateless round-trip data is data the API verifies on replay by its own key and format, not by looking
up stored state: OpenAI encrypted reasoning and compaction items, Anthropic thinking signatures and
redacted thinking, Gemini thought signatures, xAI encrypted reasoning and tool results, and OpenRouter
or Snowflake reasoning details. The system that verifies it is the producing provider's
[`canonical_system`][pydantic_ai.providers.Provider.canonical_system], ...

Server-side state (response, item, container and conversation ids) is judged by endpoint instead, with
[`Model.accepts_server_state`][pydantic_ai.models.Model.accepts_server_state], and doesn't go through this module.
"""
```

`models/__init__.py:1431-1460`, `accepts_stateless_artifacts`: "It's accepted when that is this model's [`canonical_system`][pydantic_ai.models.Model.canonical_system] or one of its [`compatible_systems`][pydantic_ai.models.Model.compatible_systems]." and "Data that isn't accepted gets the model's fallback for foreign data, such as thinking rendered as text." and "A part without a `provider_name` is never accepted." `:1462-1482`, `accepts_server_state`: "It's accepted when the response has this model's `system` as its `provider_name` and was created at an endpoint this model's provider [`shares_server_state_with`][pydantic_ai.providers.Provider.shares_server_state_with]." with the code `if response.provider_name != self.system: return False` ... `return provider.shares_server_state_with(response.provider_url)`.

## VF18. H30 against H40 - do the Gemini API and Vertex accept each other's signatures

Verdict: both quotes confirmed. The two projects state opposite facts.

Pydantic AI, `pydantic_ai_slim/pydantic_ai/models/google.py:2116-2119`:

```
    # `accepts` is `GoogleModel.accepts_stateless_artifacts`: thought signatures, and the native tool parts that
    # carry them, only go back to the transport that issued them, as the Gemini API and Google Cloud reject each
    # other's even though both providers can be named `google`. It also accepts the pre-v2 names (`google-gla`,
    # `google-vertex`), so history from before the v2 rename still routes them back.
```

and `:2212-2217`:

```
        # Per https://ai.google.dev/gemini-api/docs/thought-signatures#faqs and
        # https://cloud.google.com/vertex-ai/generative-ai/docs/thought-signatures#using-rest-or-manual-handling
        # the documented dummy `skip_thought_signature_validator` works for both Gemini API and Google Cloud.
        # It's also what a signature from the other transport is replaced with: both reject each other's, and
        # both reject a first function call without one (verified live, 2026-10-27).
        part['thought_signature'] = b'skip_thought_signature_validator'
```

LangChain, `langchain-ai/langchain-google` at `227abc8e`, `libs/genai/langchain_google_genai/_compat.py:11-13` and `:21-27`:

```
# Vertex and the Gemini Developer API share content and signature formats.
_NATIVE_MODEL_PROVIDERS = frozenset({"google_genai", "google_vertexai"})
_ModelProviderKind = Literal["native", "foreign", "unknown"]
...
def _classify_model_provider(model_provider: str | None) -> _ModelProviderKind:
    """Classify a source provider for Gemini content replay."""
    if model_provider is None:
        return "unknown"
    if model_provider in _NATIVE_MODEL_PROVIDERS:
        return "native"
    return "foreign"
```

and the use (`:216-217`): `if thought_signature and provider_kind != "foreign":` / `new_block["thought_signature"] = thought_signature`. The same classifier is what `chat_models.py` calls (`:1250`, `:1375`, `:1452`), and that file tags its own replies `{"model_provider": "google_genai"}` (`:1834`, `:2288`). So a signature whose message is tagged `google_genai` or `google_vertexai` is sent on to whichever Google backend the request targets.

A third project read in this pass sides with Pydantic AI. LiteLLM, `litellm/llms/vertex_ai/gemini/rejected_thought_signature_retry.py:23-25`: "A signature only validates on the backend that minted it. AI Studio answers a replayed Vertex signature with ``400 Corrupted thought signature.`` and Vertex answers an AI Studio one with ``400 Invalid thought signature.``" LiteLLM built a retry around those two error strings (VF10). The sentence LangChain wrote is about formats, and "same format" and "accepted by the other backend" are different claims, so both comments can be literally true while the behaviors differ. Two projects that say they saw the rejection, with a date in one and exact error strings in the other, outweigh one comment about formats. None of this was run here.

## Claims that are not plainly confirmed

1. **G37 (confirmed with a correction).** The `endpoint_slug` suffix existed on 2026-09-22 already, on the `data` field of OpenAI `reasoning.encrypted` entries (4 replies in 3 fixtures). The 2026-09-22 to 2026-10-02 window dates when Claude `signature` values began to get it. Fields: `reasoning_details[].signature` and `reasoning_details[].data` on the chat route (message and delta), `output[].signature` on the Responses route. The one recorded Gemini signature has no suffix. The Messages route is not in the fixtures.
2. **F1 (confirmed with a correction).** The SDK says and does what was recorded. Reading it as "OpenRouter gives no usable key" goes too far: rig and LangChain merge by `(type, index)`, and rig's replays of arrays merged that way returned 200.
3. **F8 (overstated).** The SDK removes a Gemini `reasoning.text` entry only when its `signature` is empty. A signed one is replayed, and rig recorded a signed one from `google/gemini-3-flash-preview`.
4. **F40 (confirmed with a correction).** pi's rule is `/^~?anthropic\//` minus ids ending in `:batch`, applied to tool-capable models.
5. **K10 (confirmed with a correction).** The id is `call_<28 hex>__thought__<signature>` only when Gemini sent no id. Otherwise it is Gemini's id plus `__thought__<signature>`. The signature is also in `provider_specific_fields.thought_signature`.
6. **K13 (confirmed with a correction to the summary).** The resend after a 400 is keyed on the error text and covers the Vertex and AI Studio mismatch. A model switch is handled without a 400: placeholder up front for Gemini 3+ history with no signature, signature cut from ids for a non-Gemini target.
7. **K72 (confirmed with a correction).** Settled: `stream_chunk_builder` merges the fragments into whole signed blocks, by signature boundary and without reading `index`. Only a caller that reads raw chunks has to merge.
8. **K35 (overstated).** Shown in code for Anthropic, Bedrock Converse, and Gemini. "Every vendor" comes from a documentation sentence.
9. **J100 (confirmed with a correction).** Envoy has no reasoning-first rebuild. Its reply splits blocks into `content` (first text), `tool_calls`, and `reasoning_content` (last block wins), and its request follows the caller's content-part order with tool uses appended.

Confirmed with a note that changes nothing in the verdict: O77 (the upstream protocol is inferred, and `system_fingerprint` is a second marker), G39 (same-route replays only), K19 (the `chatgpt` provider adds the include itself), K7 (list content is walked in the caller's order), F43 (image-producing models use OpenRouter's own provider), J27 (a fourth type constant is declared and unused), J38 (the retry text is J39).

Parts of the cited items that I did not re-read, so they stand on the first wave alone:

- K6: the list of `provider_specific_fields` keys at `transformation.py:3323-3368`. I read the block loop at `:2961-3033` only.
- K10 and K12: the thinking-block and text-part handling in `vertex_and_google_ai_studio_gemini.py:2374-2402` and `transformation.py:888-931`. I read the tool-call path only.
- K19: whether a bridge call for a `chatgpt/` model reaches the `chatgpt` Responses transform. I read the transform by itself.
- J34: agentgateway's Gemini path (first-wave J12) and the older commit `bdb36de0` that the 2026-11-17 report read.
- G39: what OpenRouter forwards upstream on a replay. The recordings hold OpenRouter's replies only.
- G37: the Anthropic Messages route. rig's fixtures have zero recordings of it.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| VF-s1 | `0xPlaygrounds/rig` at `a8673f775ca78004f011e5ad5b615147fdcd4bff` (2026-11-22): `crates/rig-cassette/fixtures/cassettes/openrouter/**` (63 files), `crates/rig-cassette/fixtures/bank/openrouter.yaml`, `crates/rig-cassette/tests/providers/openrouter/cassette/context_binding.rs` | 2026-11-22 | harness source (recorded OpenRouter traffic) |
| VF-s2 | same commit: `crates/rig-core/src/completion/message.rs`, `message/native.rs`, `history.rs`; `crates/rig-core/src/providers/anthropic/wire.rs`, `anthropic/completion.rs`, `gemini/completion.rs`, `openai/responses_api/wire.rs`, `openai/wire/chat.rs` | 2026-11-22 | harness source |
| VF-s3 | `zed-industries/zed` at `d31d1074a07b7bbaa43a52fef7847a00dd491baf` (2026-11-22): `crates/language_models/src/provider/open_router.rs` | 2026-11-22 | harness source |
| VF-s4 | `https://github.com/zed-industries/zed/commit/7fab4c0527e10a466439b819d4e9e7fe34cd989c.patch` | 2026-11-22 | harness source (commit patch) |
| VF-s5 | `OpenRouterTeam/ai-sdk-provider` at `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20): `src/chat/index.ts`, `src/chat/convert-to-openrouter-chat-messages.ts`, `src/chat/schemas.ts`, `src/chat/signature-roundtrip.test.ts`, `src/schemas/format.ts`, `src/schemas/reasoning-details.ts`, `src/schemas/provider-metadata.ts`, `src/utils/reasoning-details-duplicate-tracker.ts` | 2026-11-22 | OpenRouter SDK source |
| VF-s6 | `https://openrouter.ai/openapi.json` (local mirror), `components.schemas.ReasoningFormat`, `ReasoningDetailUnion`, `ReasoningDetailServerToolCall` | 2026-11-22 | OpenRouter spec |
| VF-s7 | `earendil-works/pi` at `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (2026-11-22): `packages/ai/catalog/sources/openrouter.ts`, `packages/ai/catalog/providers/openrouter.ts`, `packages/ai/src/providers/openrouter.ts`, `packages/ai/CHANGELOG.md` | 2026-11-22 | harness source |
| VF-s8 | `cline/cline` at `8075d2e35cf5cad67c24f180c80a05d311b74a3e` (2026-11-20): `sdk/packages/llms/src/providers/vendors/openai-compatible.ts`, `builtins.ts`, `ai-sdk.ts`, `sdk/packages/llms/src/catalog/catalog-live.ts`, `sdk/packages/shared/src/llms/model-info.ts`, `sdk/packages/llms/package.json` | 2026-11-22 | harness source |
| VF-s9 | `vercel/ai` at `d55f3cfc456d1b7bc7db3a4b70fbf602ddc8647d` (2026-11-20): `packages/openai-compatible` (text search) | 2026-11-22 | harness source |
| VF-s10 | `langchain-ai/langchain` at `8ff3cd30eebfe37712a91f11e3f4b61a545e7a37` (2026-11-20): `libs/partners/openrouter/langchain_openrouter/chat_models.py` | 2026-11-22 | harness source |
| VF-s11 | `BerriAI/litellm` at `90dce4b93bd2b090621a6b9083311294d3358bc6` (2026-11-22): `litellm/litellm_core_utils/prompt_templates/factory.py`, `common_utils.py`, `prompt_cache_breakpoint_utils.py`; `litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py`, `rejected_thought_signature_retry.py`; `litellm/llms/vertex_ai/context_caching/*.py`; `litellm/completion_extras/litellm_responses_transformation/transformation.py`, `history.py`; `litellm/llms/anthropic/chat/transformation.py`, `handler.py`; `litellm/litellm_core_utils/streaming_chunk_builder_utils.py`; `litellm/llms/openai/chat/gpt_transformation.py`; `litellm/llms/chatgpt/responses/transformation.py`; `litellm/llms/bedrock/chat/converse_transformation.py`; `litellm/llms/hosted_vllm/chat/transformation.py`; `litellm/types/llms/openai.py`; `litellm/caching/caching.py`; `litellm/utils.py` | 2026-11-22 | gateway source |
| VF-s12 | `agentgateway/agentgateway` at `888badbbe91f2a57ec8e7bf5a5ef3acda34b1fe5` (2026-11-20): `crates/llm/src/conversion/messages.rs`; text search of `crates/llm/src` and `crates/agentgateway/src/llm` | 2026-11-22 | gateway source |
| VF-s13 | `maximhq/bifrost` at `cae639c53141785bf0f21c32c135941c86e1026a` (2026-11-22): `core/schemas/chatcompletions.go`, `core/providers/utils/utils.go`, `core/providers/anthropic/chat.go`, `core/providers/gemini/chat.go`, `core/encryptedreasoning.go`, `core/bifrost.go` | 2026-11-22 | gateway source |
| VF-s14 | `theagentrouter/agent-router` (formerly `envoyproxy/ai-gateway`) at `9cf85059160fef9afc7a9f5aff75e7ce6cc8f0ce` (2026-11-19): `internal/translator/anthropic_helper.go`; text search of `internal/translator` | 2026-11-22 | gateway source |
| VF-s15 | `Portkey-AI/gateway` at `669825cbe89ee51569918b8f78a9db486fd69dd4` (2026-05-25): `src/providers/anthropic/chatComplete.ts` | 2026-11-22 | gateway source |
| VF-s16 | `pydantic/pydantic-ai` at `463a6d519b7e4c8e038d5a2559b0d26281df6937` (2026-11-22): `pydantic_ai_slim/pydantic_ai/_canonical_system.py`, `models/__init__.py`, `models/google.py` | 2026-11-22 | harness source |
| VF-s17 | `langchain-ai/langchain-google` at `227abc8efd3c357407556002b164f89f893a8c09` (2026-11-19): `libs/genai/langchain_google_genai/_compat.py` | 2026-11-22 | harness source |
| VF-s18 | First-wave evidence files of 2026-11-22 in this series: field evidence TypeScript harnesses (F), field evidence Rust and Python harnesses (G), unified SDK model layers (H), translators open-source gateways (J), translators LiteLLM (K), routing operations errors (O) | 2026-11-22 | workspace document |

