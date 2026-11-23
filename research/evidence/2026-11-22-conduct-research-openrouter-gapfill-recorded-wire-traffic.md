---
produced: 2026-11-22
title: OpenRouter gap-fill from recorded wire traffic and debug echoes - endpoint_slug suffix on signatures and encrypted reasoning, model switch replays, Gemini thought signature shapes, OpenAI Responses upstream evidence, native_finish_reason, cache token arithmetic, tool-call id formats, truncated tool calls, server tool round records, mid-stream errors, keep-alive comments, echo_upstream_body
---

# OpenRouter gap-fill from recorded wire traffic and debug echoes

- What was read: real OpenRouter requests and responses that seven open-source projects keep as test fixtures (rig, Pydantic AI, opencode, Kilo Code, req_llm, RubyLLM, llm-openrouter; eight sets counting rig's deleted cassettes recovered from git history), about 1,600 recorded exchanges dated 2025-05-20 to 2026-11-18, plus OpenRouter's own debug-echo sample and one third-party report that used the debug option.
- Sources: 17 keys (8 recording sets, 1 gateway test table, 4 OpenRouter documentation pages and the catalog, 1 third-party pull request, 4 first-wave evidence files, and 2 repositories that were checked and set aside).
- Read date: 2026-11-22. Every recording shows what OpenRouter answered on its recording date. None of them shows what OpenRouter does today.

How the labels are used here: a statement about OpenRouter drawn from a recording is `OR-inferred` and names the project, commit, and recording date. A statement about how a project copes is `Filled` and names the project. Quotes of JSON are exact, with long base64 trimmed and marked `...`. The scripts that parsed the fixtures and every intermediate listing are in the research scratch folder for this topic (prefix WT).

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Endpoint tag on signed and encrypted reasoning | Anthropic, OpenAI, Azure OpenAI, xAI (Google only in a docs sample) | The vendor blob is opaque and only the backend that issued it can read it | Chat: `reasoning_details[].signature` (Claude) and `.data` (OpenAI, Azure, xAI). Responses: `signature` (Claude) and `encrypted_content`. Value is `<vendor value>.<base64url of {"endpoint_slug":"<canonical model slug>\|<endpoint slug>"}>` | The caller holds OpenRouter's envelope, not the vendor's bytes. Start date differs per vendor (OpenAI by 2026-08-04, Anthropic between 2026-09-22 and 2026-09-29, xAI after 2026-05-30) | Caller round-trips the value unchanged. The gateway reads its own tag. The caller needs no vendor knowledge. Same on chat and Responses | OR-inferred (recordings) | WT2, WT3, WT4, WT5, WT6 |
| Replay lands on the issuing endpoint | Anthropic, OpenAI | A signature validates only where it was issued | In 34 recorded replays that had a tagged value in history, the reply came from the endpoint named in the tag (6 pinned by the caller, 28 not) | What happens on a mismatch is not in any recording | State lives inside the blob the caller returns. Gateway only | OR-inferred (correlation, weak) | WT7 |
| Reasoning after a model or provider switch | all | Vendors refuse foreign signatures with 400 | Not exercised by any recording: rig and Pydantic AI remove or rewrite reasoning before the switch, and those requests returned 200 | Unknown for OpenRouter | A client-side or gateway-side strip is what the recorded projects rely on | Filled (rig, Pydantic AI, Bifrost) | WT9, WT10, WT11, WT12 |
| Gemini thought signature shapes on chat | Google | `thoughtSignature` on a part; required on function-call parts | Five recorded shapes: `reasoning.encrypted` with `id` equal to the tool-call id; `reasoning.encrypted` with `data` only; `reasoning.text` with `signature` only; `reasoning.text` with `text` only; Responses item with `signature`. `format` is `google-gemini-v1` or, in 2025, `unknown` | Which part a signature belonged to is reduced to an optional `id`. The shape changed several times in twelve months | Caller round-trips each entry unchanged, `id` included. Gateway does the tying | OR-inferred (recordings) | WT14, WT15, WT27 |
| Upstream protocol per OpenAI model | OpenAI, Azure OpenAI, xAI | Chat Completions or Responses | `native_finish_reason` is `stop`, `tool_calls`, `length` for `gpt-4o` and `gpt-4o-mini`, and `completed`, `max_output_tokens` for `gpt-4.1-mini`, `gpt-5.x`, `o3`, `o4-mini`, and xAI Grok | The protocol choice shows only through the native value. `max_tokens: 1` produced 16 tokens on a Responses-served model | Gateway only. No caller state | OR-inferred (recordings) | WT16, WT17, WT18 |
| Reasoning text for DeepSeek, Kimi, Qwen, GLM and other open-weight models | DeepSeek, Moonshot, Alibaba, Z.AI, many hosts | Vendor field `reasoning_content` | Always `message.reasoning` plus `reasoning_details[{type: "reasoning.text", format: "unknown"}]` (`format: null` in 2025). The key `reasoning_content` appears in 0 of 911 recorded responses | Vendor field name and any vendor replay rule are hidden | Plain text, unsigned. What OpenRouter writes upstream on replay is unrecorded | OR-inferred (recordings) | WT20, WT21 |
| Cache token accounting | Anthropic, OpenAI, Google | Anthropic reports cache tokens outside `input_tokens`; OpenAI inside; Gemini has explicit caches | `usage.prompt_tokens` is the total. For Anthropic and OpenAI, `cached_tokens` and `cache_write_tokens` are disjoint parts of it. For Gemini both fields name the same tokens on the request that creates the cache | One subtraction rule does not fit every vendor: `prompt - cached - write` is negative for Gemini | Gateway normalization. A caller that prices tokens needs the vendor family | OR-inferred (recordings, checked against catalog prices) | WT23, WT24, WT25 |
| Tool-call id formats | all | Vendor-specific | Passed through from each upstream: `toolu_`, `toolu_bdrk_`, `toolu_vrtx_`, `call_` plus 24, `chatcmpl-tool-`, nine characters (Mistral), `name:0` (Moonshot), `functions.name:0`, `tool_0_name_` plus 20 then `call_` plus digits (Gemini) | No normalization | Ids issued by one provider were accepted by another provider of the same model and by another vendor (recorded 200s) | OR-inferred (recordings) | WT27, WT28 |
| Tool call cut by the output limit | OpenAI (recorded) | `length` or `max_output_tokens` | `finish_reason: "tool_calls"` beside `native_finish_reason: "length"` or `"max_output_tokens"`; arguments stop mid-string | The neutral field hides the truncation | Caller validates arguments as JSON or reads the native value | OR-inferred (recordings) | WT29, WT30 |
| Server tool round record | OpenRouter server tools on Responses | Not applicable | Output item `{"id": "rs_server_tool_round_...", "type": "reasoning", "encrypted_content": "openrouter-server-tool.<base64 JSON>"}` listing the round's member item ids | The content is plain base64 JSON, not encrypted | Caller returns it unchanged in a stateless replay. Gateway owns the meaning | OR-inferred (recording) | WT31 |
| Server tools on the chat route | OpenRouter server tools, OpenAI web search | Not applicable | Keep-alive comments while the tool runs, then only the final `content`; counts in `usage.server_tool_use_details`; citations in `message.annotations` | The call and its result leave no entry in `message` in the recordings | Gateway holds the whole loop | OR-inferred (recordings) | WT32, WT33 |
| Error inside a stream | all | Vendor error events | A `chat.completion.chunk` with top-level `error` and a `choices` array. `data: [DONE]` followed in one recording; another capture ends at the usage chunk | In both 2025-11 recordings `finish_reason` on the error chunk was `null` or `"stop"`, not `"error"` | Caller checks every chunk for `error` | OR-inferred (recordings) | WT34 |
| Keep-alive comments | all | Not applicable | Chat: `: OPENROUTER PROCESSING` plus a blank line. Responses: `: ` plus a blank line. Before the first chunk and between chunks | Interval is not recoverable from fixtures | SSE reader skips comment lines | OR-inferred (recordings) | WT35, WT36 |
| Responses route stream framing | OpenAI-shaped | `event:` and `data:` lines, no `[DONE]` | `data:` lines only, event name inside the JSON `type`, and a closing `data: [DONE]` | Differs from OpenAI's own framing | Client parses by `type` | OR-inferred (three recorders) | WT37 |
| Messages route usage and tail | Anthropic-shaped | `input_tokens` in `message_start` | `message_start.usage.input_tokens: 0`; real count, `cost`, `cost_details`, `is_byok` in `message_delta.usage`; `provider` inside `message`; a trailing `event: data` / `data: [DONE]` | Differs from Anthropic's own stream | Client reads `message_delta` | OR-inferred (recording) | WT26, WT38 |
| Context overflow rejection | all | Vendor-specific text | HTTP 400, `error.message` text, `error.metadata.provider_name: null`, header `X-Openrouter-Error-Origin: client`. The body had three keys and `error_type` was not one of them (2026-11-18) | The typed code documented for provider errors was missing on OpenRouter's own pre-send rejection | Caller matches text or reads the header | OR-inferred (recording) | WT39, WT40, WT41 |
| Out-of-spec chat fields | OpenAI | `n` | `n: 2` returned one choice (index 0). `transforms: ["middle-out"]` and `usage: {include: true}` were accepted with 200 | Silent reduction | Nothing for the caller to hold | OR-inferred (recordings) | WT42, WT43 |
| Upstream body shown by the debug echo | Anthropic | Messages API body | Docs sample: `system` as a text-block array, `model: "claude-haiku-5-5"`, `max_tokens: 65536`, `metadata.user_id`, `stop_sequences: []`. Third-party: `thinking: { type: "adaptive", display: "summarized" }` when `reasoning.enabled: true` | Only two published echoes were found | Gateway fills vendor-required defaults | OR-documented (docs sample), OR-inferred (third-party report) | WT19 |
| `native_finish_reason` vocabulary over time | Anthropic | `tool_use`, `end_turn`, `max_tokens` | 2025-11: `tool_calls`, `stop`, `length` for Claude on Anthropic and Vertex. 2026-05 onward: the vendor's own words | The native field changed meaning without a version marker | Do not key logic on it alone | OR-inferred (recordings) | WT44, WT45 |

## Evidence

### The recordings and what each recorder changes

**WT1. Eight sets of recorded OpenRouter traffic were read. Each recorder alters the bytes in a known way, which limits what each set can show.**

| Set | Location | Exchanges with OpenRouter | Recording dates | How the date is known | What the recorder changes |
|---|---|---|---|---|---|
| rig, current tree | `crates/rig-cassette/fixtures/cassettes/openrouter/` (63 files) and `fixtures/bank/openrouter.yaml` (111 replies) | 107 | 2026-09-22 to 2026-10-03 | Unix time inside `gen-<unix>-<random>` | `created` set to 0, JSON keys sorted, response headers reduced to `content-type`, credentials scrubbed. SSE comment lines and `event:` lines are kept (its Anthropic and OpenAI fixtures have `event:` lines) |
| rig, history | Same folder at commit `a023bd5b` (251 files; 188 of them were deleted on 2026-10-05 by commit `daff77b9`) | 335 | 2026-09-22 to 2026-10-03 | Same | Same |
| Pydantic AI | `tests/models/cassettes/test_openrouter/`, `tests/models/openrouter/cassettes/`, and other cassette folders under `tests/` | 129 | 2025-05-20 to 2026-11-17 | `recorded_at` on newer cassettes, the `gen-` id on older ones | Bodies stored as parsed YAML (key order kept), streams as text, some response headers kept |
| opencode | `packages/ai/test/fixtures/recordings/openrouter-*.json` | 17 | 2026-07-18 to 2026-11-18 | `metadata.recordedAt` | Headers reduced to `content-type`; bodies verbatim |
| Kilo Code | `packages/llm/test/fixtures/recordings/openai-compatible-chat/openrouter-*.json` | 8 | 2026-05-06 | `metadata.recordedAt` | Same as opencode |
| req_llm | `test/support/fixtures/openrouter/` (911 JSON files in 276 model folders) | 911 | 2025-10-03 to 2026-10-09 (609 in 2025-11, 209 in 2026-05, 2 in 2026-10) | `gen-` id and the `date` response header | Streams stored as base64 of the raw network chunks, so bytes and chunk boundaries are exact. The recorder can stop a capture when its decoder marks a chunk terminal (`lib/req_llm/stream_server.ex` lines 933-934) |
| RubyLLM | `spec/fixtures/vcr_cassettes/*openrouter*` (78 cassettes) | 202 | 2026-11-18 | `recorded_at` | Bodies stored as base64 (`!binary`), so a file line number cannot point at a JSON field. Most response headers kept |
| llm-openrouter | `tests/cassettes/test_llm_openrouter/` | 10 | 2026-08-04 | `gen-` id | Streams as text |

- Also checked and found to hold no recorded OpenRouter responses: OpenRouter's AI SDK provider (inline mock data in tests), `langchain-openrouter` (unit and integration tests, no cassettes), LiteLLM's OpenRouter tests, rust-genai, the Vercel AI SDK, OpenRouter's TypeScript SDK and agent. Hermes Agent's `tests/fixtures/provider_wire_snapshot.json` holds the requests Hermes builds, not OpenRouter's answers. Prism has two small fixtures with real ids dated 2025-10-15 that were not analyzed.
- Sources: WT-s1 to WT-s8. Basis: this item describes the sources.

### Endpoint tag on signatures and encrypted reasoning (OQ4, OQ9)

**WT2. The part after the last dot decodes to one JSON object with one key, `endpoint_slug`. Thirteen distinct values were decoded from recordings.**

| Decoded suffix | Field it was on | Route | Reply `provider` | Fixture and line | Recorded |
|---|---|---|---|---|---|
| `{"endpoint_slug":"openai/gpt-5-nano-2025-08-07\|openai"}` | reasoning item `encrypted_content` | Responses | (not in body) | llm-openrouter `tests/cassettes/test_llm_openrouter/test_reasoning.yaml:224` | 2026-08-04 |
| `{"endpoint_slug":"openai/gpt-5.2-20251211\|openai"}` | `reasoning_details[].data` | chat | `OpenAI` | rig `reasoning_delta_hook/streaming.yaml:187` | 2026-09-22 |
| `{"endpoint_slug":"openai/o4-mini-2025-04-16\|openai"}` | `reasoning_details[].data` | chat | `OpenAI` | rig `reasoning_usage_matrix/blocking_excluded_reasoning_still_counts_tokens.yaml:16` | 2026-09-22 |
| `{"endpoint_slug":"anthropic/claude-4.6-sonnet-20260217\|claude-on-aws"}` | `reasoning_details[].signature` | chat | `Claude Platform on AWS` | opencode `openrouter-reasoning.json:25` | 2026-09-29 |
| `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` | `reasoning_details[].signature`; reasoning item `signature` | chat; Responses | `Amazon Bedrock` | rig `upstream_switch_matrix/switch_unary.yaml:16`; `upstream_switch_matrix/responses_switch_unary.yaml:16` | 2026-10-02 |
| `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|anthropic"}` | `reasoning_details[].signature` | chat | `Anthropic` | rig `reasoning_tool_order_matrix/streaming_signed_agent_roundtrip.yaml:51` | 2026-10-02 |
| `{"endpoint_slug":"anthropic/claude-opus-5.5-20260921\|amazon-bedrock"}`, `...\|anthropic"}`, `...\|google-vertex/global"}` | `reasoning_details[].signature` | chat | `Amazon Bedrock`, `Anthropic`, `Google` | rig `context_binding/amazon_bedrock.yaml:16`, `anthropic.yaml:16`, `google_vertex.yaml:16` | 2026-10-03 |
| `{"endpoint_slug":"anthropic/claude-sonnet-5-20260630\|claude-on-aws"}` | `reasoning_details[].signature` | chat | `Claude Platform on AWS` | Pydantic AI `test_openrouter_map_messages_reasoning.yaml` | 2026-10-25 |
| `{"endpoint_slug":"openai/gpt-5.2-codex-20260114\|azure"}` | `reasoning_details[].data`, `format: azure-openai-responses-v1` | chat | `Azure` | Pydantic AI `test_openrouter_thinking_false_on_mandatory_reasoning_route[openai-gpt-5.2-codex].yaml:74` | 2026-11-06 |
| `{"endpoint_slug":"x-ai/grok-build-0.1-20260520\|xai"}` | `reasoning_details[].data`, `format: xai-responses-v1` | chat | `xAI` | Pydantic AI `test_openrouter_thinking_false_on_mandatory_reasoning_route[x-ai-grok-build-0.1].yaml:78` | 2026-11-06 |
| `{"endpoint_slug":"anthropic/claude-fable-5.2-20261109\|google-vertex/global"}` | `reasoning_details[].signature` | chat | `Google` | Pydantic AI `test_openrouter_effort_change_keeps_the_cache[anthropic-fable-5-2].yaml` | 2026-11-10 |
| `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` | `reasoning_details[].signature` | chat | `Amazon Bedrock` | RubyLLM `chat_with_extended_thinking_openrouter_claude-haiku-4-5_returns_thinking_when_available.yml`, exchange 0 | 2026-11-18 |

- Quote (rig `upstream_switch_matrix/switch_unary.yaml:16`, 2026-10-02 20:15 UTC): `"reasoning_details":[{"format":"anthropic-claude-v1","index":0,"signature":"EsoFCpwBCBIQARgCKkBMuBvY...VIgGAE=.eyJlbmRwb2ludF9zbHVnIjoiYW50aHJvcGljL2NsYXVkZS00LjUtaGFpa3UtMjAyNTEwMDF8YW1hem9uLWJlZHJvY2svZ2xvYmFsIn0","text":"The user is asking me to:...","type":"reasoning.text"}]` (signature middle and text trimmed by me).
- Quote (Pydantic AI, xAI cassette lines 74-79, 2026-11-06): `format: xai-responses-v1` / `id: rs_cab17b2a-439b-9d13-95ed-ea1ae3d166ed` / data ending `...2v07RS1z5c9YRVH2L85kMSKtGADcw.eyJlbmRwb2ludF9zbHVnIjoieC1haS9ncm9rLWJ1aWxkLTAuMS0yMDI2MDUyMHx4YWkifQ`.
- The suffix is base64url without padding. The vendor part before the dot keeps its own alphabet (Anthropic signatures use standard base64 with `/`, `+`, `=`; OpenAI blobs start `gAAAAAB`).
- Sources: WT-s1, WT-s3, WT-s4, WT-s7, WT-s8. Basis: OR-inferred (inferred from recorded traffic in the named projects on the dates shown).

**WT3. The tag arrived at different times for different vendors. The recordings bracket each start date.**

| Vendor value | Latest recording where the value is the vendor's string alone | Earliest recording with the tag |
|---|---|---|
| OpenAI `reasoning.encrypted` `data` (chat) | 2026-05-26: Pydantic AI `test_eager_input_streaming_sent_to_openrouter[non-anthropic-enabled].yaml:74-81`, `gpt-5-mini`, data ends `...4AaXiV-pTv6BTQ=`. Also 68 values in req_llm fixtures dated 2025-10-27 to 2025-11-02 | 2026-08-04 on the Responses route (llm-openrouter); 2026-09-22 on the chat route (rig) |
| Anthropic `signature` (chat) | 2026-07-18: opencode `openrouter-reasoning-tool-loop.json:33`, provider `Anthropic`, signature ends `...7I0ScF53U9JGhgB`. 2026-09-22 16:48 UTC: rig `reasoning_tool_order_matrix/blocking_single.yaml:16`, provider `Anthropic`, signature ends `...IITRuBIrLEYAQ==` | 2026-09-29 16:18 UTC: opencode `openrouter-reasoning.json:25`, provider `Claude Platform on AWS` |
| xAI `reasoning.encrypted` `data` | 2026-05-30: req_llm `x_ai_grok_build_0_1/basic.json`. 2025-11-02: Pydantic AI `test_openrouter_stream_with_native_options.yaml:54`, id `rs_619f3702-e30e-6ba5-7b57-466db8036365_us-east-1` | 2026-11-06 (Pydantic AI) |
| Google `reasoning.encrypted` `data` and `reasoning.text` `signature` (chat) | Every chat recording through 2026-11-06, for example Pydantic AI `[google-gemini-3.5-flash].yaml:72-75`, data starts `AY89a1/Gmh64qKx` and ends `...IpFC0hxm1BoFNgF` | None on the chat route (see WT6) |

- So on 2026-09-22 the same rig run recorded an OpenAI value with the tag (16:47:12 UTC) and an Anthropic value in the vendor's own form (16:48:17 UTC), 65 seconds apart.
- The tag was still present four days before this research (RubyLLM, 2026-11-18).
- OpenRouter's changelog and documentation mirror have zero occurrences of the key name `endpoint_slug` (first-wave finding, re-run today with the same result).
- Sources: WT-s1, WT-s3, WT-s4, WT-s6, WT-s7, WT-s8, WT-s15. Basis: OR-inferred (recordings).

**WT4. The tag names a model snapshot and a serving endpoint: `<canonical model slug>|<endpoint slug>`.**

- The model half equals the catalog's `canonical_slug` for the requested model, not the id the caller sent. Catalog read 2026-11-22: `anthropic/claude-haiku-4.5` has `canonical_slug` `anthropic/claude-4.5-haiku-20251001`; `anthropic/claude-sonnet-4.6` has `anthropic/claude-4.6-sonnet-20260217`; `openai/gpt-5.2` has `openai/gpt-5.2-20251211`; `anthropic/claude-opus-5.5` has `anthropic/claude-opus-5.5-20260921`; `openai/gpt-5.2-codex` has `openai/gpt-5.2-codex-20260114`. Each matches the decoded tags in WT2.
- The endpoint half has the shape OpenRouter documents for endpoint slugs. Quote (provider selection guide): "Every endpoint slug has the shape `provider[/variant][/region][/tier]`, with each segment present only when the endpoint needs it" and "(`google-vertex/global` and `google-vertex/global/priority`, `baseten/fp4/us` and `baseten/fp4/us/priority`)".
- Recorded endpoint halves: `openai`, `azure`, `xai`, `anthropic`, `claude-on-aws`, `amazon-bedrock`, `amazon-bedrock/global`, `google-vertex/global`. Each agrees with the reply's `provider` display name (`OpenAI`, `Azure`, `xAI`, `Anthropic`, `Claude Platform on AWS`, `Amazon Bedrock`, `Google`).
- So the tag identifies model snapshot, provider, and region or variant. It does not name an account: the same `anthropic/claude-4.5-haiku-20251001|amazon-bedrock/global` value appears in rig's recordings and in RubyLLM's recordings, which two unrelated projects made six weeks apart.
- Sources: WT-s1, WT-s7, WT-s11, WT-s13. Basis: OR-inferred for the reading of the tag (recordings compared with the catalog), OR-documented for the slug shape.

**WT5. The tag is present on all three routes, as far as recordings and OpenRouter's samples reach.**

- Chat route: on `reasoning_details[].signature` for Claude and on `reasoning_details[].data` for OpenAI, Azure OpenAI, and xAI (WT2).
- Responses route, recorded: on a Claude reasoning item's `signature`, beside `"format":"anthropic-claude-v1"`. Quote (rig `upstream_switch_matrix/responses_switch_unary.yaml:16`, 2026-10-02): `{"content":[...],"format":"anthropic-claude-v1","id":"rs_tmp_tlkmunm8kfr","signature":"EuQFCpwBCBIQARg...Y2svZ2xvYmFsIn0","status":"completed","summary":[],"type":"reasoning"}`. And on an OpenAI reasoning item's `encrypted_content` beside `"format":"openai-responses-v1"` (llm-openrouter 2026-08-04; RubyLLM 2026-11-18).
- Messages route: no third-party recording has reasoning on that route. OpenRouter's service tiers guide prints a Messages output for `openai/gpt-5-mini` whose blob ends in the tag (first-wave item S31 decoded it to `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|openai"}`); the sample in the mirror is clipped in the middle, so the block type around it was not re-read.
- Sources: WT-s1, WT-s7, WT-s8, WT-s12, WT-s15. Basis: OR-inferred (recordings); the Messages sample is OR-documented by way of the first-wave file.

**WT6. Gemini signatures on the chat route are Google's string alone in every recording, while one OpenRouter documentation sample on the Responses route has the tag.**

- Chat route, Vertex endpoint, 2026-11-04 (Pydantic AI `test_openrouter_cache_gemini_does_not_double_prompt_tokens.yaml`, request pinned with `"provider": {"order": ["google-vertex"], "allow_fallbacks": false}`): `{"type": "reasoning.encrypted", "data": "AY89a19s3lEmKyT...hj45WU/MKklUew=", "format": "google-gemini-v1", "index": 0}`. The value ends in base64 padding.
- Documentation sample (image understanding guide, Responses output, lines 230-235 of the mirror): `"id": "rs_tmp_kp722gun36"`, `"type": "reasoning"`, `"encrypted_content": "AY89a1+LwO7pEbrfZ/kZ+Qdrc8r/3ztvp8blrnIqc6apf5dNmyC1...b2ludF9zbHVnIjoiZ29vZ2xlL2dlbWluaS0zLWZsYXNoLXByZXZpZXctMjAyNTEyMTd8Z29vZ2xlLXZlcnRleC9nbG9iYWwifQ"`, `"format": "google-gemini-v1"`. First-wave decoding: `{"endpoint_slug":"google/gemini-3-flash-preview-20251217|google-vertex/global"}`.
- What this cannot settle: whether the Gemini tag is specific to the Responses route, or began after 2026-11-06. The sample has no date.
- Sources: WT-s3, WT-s12, WT-s15. Basis: OR-inferred (recording) and OR-documented (sample). Confidence: high for each observation, low for any rule joining them.

**WT7. Every recorded replay of a tagged value was answered with 200 by the endpoint the tag names. No recording sends a tagged value to a different endpoint.**

- Count: 34 requests whose history held a tagged signature or tagged encrypted blob (29 in rig at `a023bd5b`, 1 in Pydantic AI, 4 in RubyLLM). All returned 200.
- 6 of the 34 were pinned by the caller (`provider.only` or `provider.order` with `allow_fallbacks: false`). The other 28 had no provider preference, and each was served by the tagged endpoint: for example rig `history_survival_matrix/unary.yaml` exchanges 1 to 6 (2026-10-02), tag `...|amazon-bedrock/global`, reply `"provider":"Amazon Bedrock"`.
- rig tested the three Claude Opus 5.5 routes one by one with a tool list that changed between turns. Quote (rig `tests/providers/openrouter/cassette/context_binding.rs:1-5`): "//! Live fact behind Rig replaying OpenRouter Claude turns verbatim: on each" / "//! route OpenRouter serves Claude Opus 5.5 from (Anthropic, Amazon Bedrock and" / "//! Google Vertex), a turn's signed `reasoning_details` go back after the tool" / "//! list changes, and the request succeeds. So Chat does not bind context on" / "//! OpenRouter."
- What it shows: the tagged value is accepted back on the same endpoint, on chat and on Responses, streamed and unary.
- What it cannot show: whether the tag steers routing. In the same sessions, first turns without any history also went to the same provider (all unpinned `anthropic/claude-haiku-4.5` requests in rig on 2026-10-02 and in RubyLLM on 2026-11-18 were served by Amazon Bedrock). A year earlier the same model was spread over three providers inside one minute (req_llm, 2025-11-02: `Anthropic`, `Amazon Bedrock`, `Google`).
- Sources: WT-s1, WT-s2, WT-s3, WT-s7. Basis: OR-inferred (recordings). Confidence: high for "accepted on the same endpoint", low for "the tag causes the routing".

**WT8. `format` values seen in recordings.**

- `anthropic-claude-v1` (Claude on every provider), `openai-responses-v1` (OpenAI), `azure-openai-responses-v1` (Azure, 2026-11-06), `xai-responses-v1` (xAI, from 2025-11-02), `google-gemini-v1` (Gemini, from 2025-12-23), `unknown` (open-weight models and, in 2025, Gemini and one Bedrock Claude reply dated 2025-10-03), and JSON `null` (several hosts on 2025-11-02 and Minimax on 2025-11-03).
- Quote (req_llm `anthropic_claude_sonnet_4_5/reasoning_basic.json`, 2025-10-03, provider `Amazon Bedrock`): a `reasoning.text` entry with a `signature` and `"format": "unknown"`.
- Sources: WT-s3, WT-s6. Basis: OR-inferred (recordings).

### Replay after a model or provider switch (OQ1) and unsigned reasoning (OQ5)

**WT9. rig's switch matrix does not test what OpenRouter does with foreign reasoning, because rig removes it before sending. What the fixtures show is that the stripped history is accepted, and that a model accepts its own tagged reasoning beside another model's plain-text turn.**

- Design, in rig's words (`crates/rig-cassette/tests/providers/openrouter/cassette/upstream_switch_matrix.rs:1-5`): "//! Reasoning through OpenRouter replays only to the model that produced it." / "//! A conversation that moves to another model, of any family, sends that" / "//! model the reasoning as plain text and no signature or ciphertext, and the" / "//! home model's signed reasoning returns intact when the conversation comes" / "//! back."
- Claude to Gemini (`upstream_switch_matrix/switch_unary.yaml:27`, 2026-10-02 20:15 UTC). The assistant turn sent to `google/gemini-3-flash-preview` has three keys, `content`, `role`, `tool_calls`, and Claude's thinking text is joined to the front of `content`: `{"content":"The user is asking me to:\n1. Think it through\n2. Call lookup_code for record \"alpha\"...I'll call the lookup_code function to retrieve the code stored for record \"alpha\".","role":"assistant","tool_calls":[{"function":{"arguments":"{\"record\":\"alpha\"}","name":"lookup_code"},"id":"toolu_bdrk_01VzaKHoRgZVCVCqesM54vz6","type":"function"}]}`. Reply (line 33): status 200, `"provider":"Google"`, `"native_finish_reason":"STOP"`.
- Back to Claude (same file, line 44). The first assistant turn has `content`, `reasoning`, `reasoning_details` (the signed entry with the `...|amazon-bedrock/global` tag), `role`, `tool_calls`; Gemini's turn is `{"content":"**Repeating Code Block**...amber-5521","role":"assistant"}`. Reply (line 50): status 200, `"provider":"Amazon Bedrock"`.
- Claude to another Claude (rig at `a023bd5b`, `upstream_switch_matrix/same_family_unary.yaml:27`, 2026-10-02 05:29 UTC): the turn sent to `anthropic/claude-sonnet-4.6` has `content`, `role`, `tool_calls` only. Reply: status 200, `"provider":"Claude Platform on AWS"`, with a new signature tagged `anthropic/claude-4.6-sonnet-20260217|claude-on-aws`.
- Responses route (`upstream_switch_matrix/responses_switch_unary.yaml:27`): the request to Gemini has the `function_call` and `function_call_output` items and no reasoning item. Reply: 200.
- All 20 requests in the switch cells and the same-family cells (chat and Responses, streamed and unary) returned 200.
- What it cannot show: OpenRouter's answer to a `reasoning_details` entry in a foreign `format`, or to a tagged value sent to another endpoint. No recording in any of the eight sets contains such a request.
- Sources: WT-s1, WT-s2. Basis: Filled (rig) for the stripping rule; OR-inferred (recorded traffic in rig at `a8673f7` and `a023bd5b`, 2026-10-02) for the 200s.

**WT10. Bifrost keeps a table of the refusal texts it saw after a provider or model switch, including the form they take through OpenRouter.**

- Quote (Bifrost `core/encryptedreasoning_test.go:1103-1108`): "TestShouldStripReasoningAfterClientError pins the fail-soft gate: a 400 that names a reasoning token by family word earns the one stripped retry, whatever verdict the upstream attaches, and nothing else does." and "The TRUE rows are the refusals each surface was observed to return after a provider or model switch; only Anthropic's is documented."
- Rows (lines 1115-1136), all status 400: `"invalid encrypted reasoning"` (Bedrock Mantle); `"encrypted reasoning was created for a different account or model"`; `"Encrypted content cannot be used in a different region from the one that created it."`; `"The encrypted content for item rs_1 could not be verified. Reason: Encrypted content organization_id did not match the target organization."` (OpenAI); `"Could not decrypt the provided encrypted_content. Ensure the value is the unmodified encrypted_content from a previous response."` (xAI); ``"messages.1.content.0: Invalid `signature` in `thinking` block"`` (Anthropic); `"Invalid thought signature."`, `"Corrupted thought signature."`, `"Unable to submit request because Thought signature is not valid."` (Gemini).
- OpenRouter rows: `{"openrouter outer envelope alone", "Provider returned error", 400, false}` and ``{"openrouter after the provider lifts metadata.raw", "Provider returned error (Anthropic): messages.1.content.0: Invalid `signature` in `thinking` block", 400, true}``.
- Test data uses the OpenRouter envelope form of a blob: `"gAAAAABqc9R3ciphertext.eyJlbmRwb2ludF9zbHVnIjoieCJ9"` (line 100), whose suffix decodes to `{"endpoint_slug":"x"}`. So Bifrost's authors have met the tag.
- Source: WT-s9. Basis: Filled (Bifrost). It shows which texts a gateway must recognize and that through OpenRouter the vendor text sits under `error.metadata.raw`. It does not show when OpenRouter forwards a foreign blob.

**WT11. Pydantic AI moves thinking that it cannot replay into a tagged user message, and a recording shows Claude on OpenRouter's Responses route accepting that.**

- Quote (Pydantic AI `tests/models/openrouter/cassettes/test_carried_thinking/test_openrouter_responses_claude_accepts_carried_thinking.yaml:34-38`, recorded 2026-10-26): `- role: user` / `content: |-` / `<assistant_thinking>` / `The lookup came back with 8.1 for the 10-year, so a 1% move in yields is worth roughly 8% of price.` / `</assistant_thinking>`. The same request has a `function_call` with the invented id `call_treasury_duration_1`. Reply: status 200, model `anthropic/claude-sonnet-4.5`.
- Source: WT-s3. Basis: Filled (Pydantic AI) for the technique; OR-inferred (recording) for the 200.

**WT12. No recording sends a Claude `reasoning.text` entry without its signature, so the recordings add nothing to the two conflicting accounts on OQ5.**

- Searched: every request in all eight sets for an assistant turn with `reasoning_details` whose `anthropic-claude-v1` entry lacks `signature`, or with a bare `reasoning` string. Result: zero requests. Every client either sends the signed entry back whole or removes reasoning (WT9, WT11).
- Related recorded fact: Claude accepted a tool continuation whose assistant turn had only `content` and `tool_calls` (req_llm `anthropic_claude_haiku_4_5/tool_round_trip_2.json`, 2025-11-02; RubyLLM tool cassettes, 2026-11-18). Those first turns had no thinking, so they say nothing about a thinking turn replayed bare.
- Basis: no evidence found. See Open questions.

### Block order (OQ2)

**WT13. Recordings show the order in which OpenRouter emits a Claude turn and the request shape it accepts back. No published debug echo shows the upstream body for a turn with history.**

- Stream order, Claude on Bedrock (rig `upstream_switch_matrix/switch_streamed.yaml`, 2026-10-02): lines 17-63 are `reasoning` deltas, each with a `reasoning_details` entry `{"format":"anthropic-claude-v1","index":0,"text":"...","type":"reasoning.text"}`; line 65 is one delta whose entry has `signature` and no `text`; lines 67-77 are `content` deltas; lines 79-85 are `tool_calls` deltas; line 87 is the finish chunk; line 89 the usage chunk; line 91 `data: [DONE]`. The same order appears in opencode (2026-07-18), Pydantic AI (2026-10-25), and RubyLLM (2026-11-18). That is the vendor's block order: thinking, text, tool use.
- Accepted request shape: one assistant message with `content` (the text), `reasoning`, `reasoning_details` (one signed entry), and `tool_calls` (rig `switch_unary.yaml:44`; RubyLLM `chat_openrouter_reasoning_details_round-trip_replays_reasoning_details_across_tool_calls_and_turns.yml`, exchange 1). All such replays returned 200.
- All 67 recorded Claude replies that have reasoning (rig, Pydantic AI, RubyLLM, req_llm) have one signature, on `reasoning.text` entries at `index` 0. No recording has two signed entries, or thinking between two tool calls, so the recordings cannot show how position is rebuilt for interleaved thinking.
- OpenAI comparison (RubyLLM `chat_web_search_with_openrouter_openai_gpt-5_2_searches_transparently_returning_citations_and_usage_counters.yml`, 2026-11-18): one reply holds four upstream reasoning items as eight entries, `reasoning.summary` at `index` 0, 2, 4, 6 and `reasoning.encrypted` at `index` 1, 3, 5, 7 with ids `rs_04878cd67330e9ca016afd47a3523487d1925ce7a2934e1f14` and three more, beside one `content` string. The web searches that ran between those reasoning items have no entry in the message.
- Source: WT-s1, WT-s3, WT-s4, WT-s7. Basis: OR-inferred (recordings).

### Gemini thought signatures on the chat route (OQ3)

**WT14. Five shapes were recorded. In each, the tool call object itself has four keys (`function`, `id`, `index`, `type`); the signature never sits on the tool call.**

| Shape | Example | Fixture | Recorded |
|---|---|---|---|
| A. `reasoning.encrypted` with `id` equal to the tool-call id | `{"data": "EjQKMgFyyNp8hPllMrP4ALd8U0VgyeKNwyn7EqtDsNWBVna1GcyLgBzBjfqxqs7OqTTJGtGJ", "format": "google-gemini-v1", "id": "tool_add_QpVaa8pTdTtaG8iAAvhe", "index": 0, "type": "reasoning.encrypted"}` beside `"tool_calls": [{"function": {"arguments": "{\"b\":3,\"a\":2}", "name": "add"}, "id": "tool_add_QpVaa8pTdTtaG8iAAvhe", "index": 0, "type": "function"}]` | req_llm `google_gemini_3_flash_preview/tool_round_trip_1.json`, provider `Google AI Studio` | 2025-12-23 |
| A, later | `type: reasoning.text` with `text` at `index: 0`, then `type: reasoning.encrypted`, `data: AY89a1/ZpTRl7gt...`, `format: google-gemini-v1`, `id: call_172647`, `index: 1`; tool calls `call_172647` and `call_172648`. Only the first of the two parallel calls has an entry | Pydantic AI `test_openrouter_google_inlines_property_names_and_contains_refs.yaml:128-151`, `google/gemini-3.6-flash`, provider `Google` | 2026-10-27 |
| B. `reasoning.encrypted` with `data` and no `id` (text reply) | `type: reasoning.encrypted` / `data: AY89a1/Gmh64qKx...IpFC0hxm1BoFNgF` / `format: google-gemini-v1` / `index: 0` | Pydantic AI `...mandatory_reasoning_route[google-gemini-3.5-flash].yaml:72-75`; also req_llm `google_gemini_3_1_flash_lite/basic.json` (2026-05-30) | 2026-11-06 |
| C. `reasoning.text` with `signature` and no `text` | `- format: google-gemini-v1` / `index: 0` / `signature: AY89a1/KcFoa2eBtwTKrKGuv9agcbDFeHKJsXT/9P/yBWK5l...` / `type: reasoning.text` | Pydantic AI `test_openrouter_web_search_tool_full_params.yaml:67-70`, `google/gemini-3.6-flash` | 2026-08-12 |
| C, streamed | `{"format":"google-gemini-v1","index":0,"signature":"AY89a19QiA2xoKvfVf9xoeiRvpyOPIzSO188TDrn/lJkGATbpvhpjLr/62Z2pxCwkOegYrTKtpgQmdYpeyJaswt4r0VRJCqYtD2JpNNmvsjx/G12bJ3BhA7v","type":"reasoning.text"}`, sent in a chunk after the answer text | rig `upstream_switch_matrix/switch_streamed.yaml:110-112`, `google/gemini-3-flash-preview` | 2026-10-02 |
| D. `reasoning.text` with `text` only | `{"format":"google-gemini-v1","index":0,"text":"**Repeating Code Block**...","type":"reasoning.text"}`; in 2025 the same shape had `"format":"unknown"` | rig `upstream_switch_matrix/switch_unary.yaml:33`; req_llm `google_gemini_2_5_flash/multi_tool.json` (2025-11-02) | 2026-10-02 |
| E. Responses route item | `{"id":"rs_tmp_v1eaez118d","signature":"AY89a1/sn1v0J6O...HpCBiLKnVgiYZw=","status":"completed","summary":[],"type":"reasoning"}`, placed after the message item | rig `openai_responses_compat/openai_responses_raw_response_accepts_service_tier_metadata.yaml:16` | 2026-09-22 |

- Note on D: the unary reply of 2026-10-02 had thinking text and no signature anywhere, while the streamed twin recorded sixteen seconds earlier had a signature and no thinking text.
- Sources: WT-s1, WT-s3, WT-s6. Basis: OR-inferred (recordings). Confidence: high for each shape on its date; the rule that picks a shape is not visible.

**WT15. Each shape that was sent back unchanged was accepted.**

- Shape A with the tool call and its result, `tools` omitted from the request: 200 (req_llm `google_gemini_3_flash_preview/tool_round_trip_2.json`, 2025-12-23). Shape A with a second, unsigned parallel call: 200 (Pydantic AI, 2026-10-27, exchange 1).
- Shape B on a Vertex-pinned request: 200 (Pydantic AI `test_openrouter_cache_gemini_does_not_double_prompt_tokens.yaml`, exchange 1, 2026-11-04).
- No recording replays a Gemini signature on a different Google endpoint (AI Studio versus Vertex).
- Sources: WT-s3, WT-s6. Basis: OR-inferred (recordings).

### OpenAI and xAI on the chat route (OQ11)

**WT16. `native_finish_reason` splits OpenAI models into two groups, and the split matches the two OpenAI protocols.**

| Model (provider) | `finish_reason` and `native_finish_reason` pairs recorded | Fixtures and dates |
|---|---|---|
| `openai/gpt-4o`, `openai/gpt-4o-mini` (`OpenAI`, `Azure`) | `stop`/`stop`, `tool_calls`/`tool_calls`, `length`/`length`, and `tool_calls`/`stop` when `tool_choice` named a function | Kilo Code 2026-05-06; Pydantic AI 2026-07-24; rig 2026-09-23 and 2026-10-02; req_llm 2025-11-02 |
| `openai/gpt-4.1-mini` (`OpenAI`) | `stop`/`completed`, `tool_calls`/`completed`, `length`/`max_output_tokens`, `tool_calls`/`max_output_tokens` | Pydantic AI 2026-01-13 and 2026-08-11; rig 2026-09-23 and 2026-10-03; req_llm 2026-10-09 |
| `openai/gpt-5-mini`, `gpt-5.1-codex-mini`, `gpt-5.2`, `gpt-5.5`, `gpt-6-luna`, `gpt-6.1-sol`, `o3`, `o4-mini` (`OpenAI`), `gpt-5.2-codex` (`Azure`) | `stop`/`completed`, `tool_calls`/`completed`, `length`/`max_output_tokens` | Pydantic AI 2025-10-29 to 2026-11-06; rig 2026-09-22 to 2026-10-02; RubyLLM 2026-11-18 |
| `x-ai/grok-4`, `grok-3-mini`, `grok-build-0.1` (`xAI`) | `stop`/`completed`, `tool_calls`/`completed`; also `stop`/`stop` (2025-10-03 and five req_llm replies) and `tool_calls`/`tool_calls` (four req_llm replies) | Pydantic AI 2025-10-03, 2025-11-02, 2026-11-06; req_llm 2025-11-02 |

- `completed` and `max_output_tokens` are words of the Responses API (`status` and `incomplete_details.reason`); `stop`, `tool_calls`, `length` are Chat Completions words. So in these recordings a non-reasoning model, `gpt-4.1-mini`, was reached through the Responses API, and `gpt-4o` and `gpt-4o-mini` through Chat Completions.
- One exception: `openai/gpt-4.1-mini` answered `stop`/`stop` on 2026-08-27 in a request that used the `openrouter:web_search` server tool with engine settings (Pydantic AI `test_openrouter_web_search_engine_settings.yaml`). The split is therefore not purely by model.
- xAI: the same `completed` word plus `"format":"xai-responses-v1"` and ids like `rs_cab17b2a-439b-9d13-95ed-ea1ae3d166ed` show a Responses-shaped upstream for xAI from 2025-11-02 on.
- Sources: WT-s1, WT-s2, WT-s3, WT-s5, WT-s6, WT-s7. Basis: OR-inferred (recordings). Confidence: high that the native words are as recorded, medium that they identify the upstream protocol.

**WT17. On a Responses-served model, `max_tokens: 1` produced sixteen tokens.**

- rig `terminal_metadata_matrix/blocking_gpt_4_1_mini_tiny_plain_two.yaml:10` (2026-09-23): request `{"max_tokens":1,...,"model":"openai/gpt-4.1-mini","n":2,...}`. Reply (line 16): `"content":"cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt cobalt"`, `"native_finish_reason":"max_output_tokens"`, `"completion_tokens":16`.
- Same request size for `openai/gpt-4o-mini` (`terminal_metadata_matrix/blocking_gpt_4o_mini_tiny_plain_one.yaml`): `"completion_tokens":1`, `"native_finish_reason":"length"`.
- rig notes the vendor limit in its direct OpenAI tests (`crates/rig-cassette/tests/providers/openai/cassette/corpus_matrix_responses.rs:54`): "the Responses wire's `max_output_tokens` floor is 16". So the caller's 1 was raised to the floor on the way upstream. The reply does not say so.
- Source: WT-s1. Basis: OR-inferred (recorded traffic in rig at `a8673f7`, 2026-09-23).

**WT18. OpenAI reasoning on the chat route: entry shapes, ids, and what a replay needs.**

- Non-streaming reply (rig `reasoning_roundtrip/nonstreaming.yaml:16`, 2026-10-02, `openai/gpt-5.2`): two entries, `{"format":"openai-responses-v1","index":0,"summary":"...","type":"reasoning.summary"}` and `{"data":"gAAAAABqwBArBYP...MTF8b3BlbmFpIn0","format":"openai-responses-v1","id":"rs_02de745d9b1e8fa7016ac01027c6e887d18f8d7f5699976b67","index":1,"type":"reasoning.encrypted"}`. The `id` is the upstream reasoning item id (`rs_` plus 50 hex digits) and sits only on the encrypted entry.
- When the model returns no summary, there is one `reasoning.encrypted` entry at `index` 0 (rig `streaming_tools/stream_encrypted_reasoning_survives_into_the_next_turn.yaml:25`, `openai/o4-mini`).
- In 2025 both entries had `index` 0 (Pydantic AI `test_openrouter_preserve_reasoning_block.yaml`, 2025-10-29).
- Replay with both entries unchanged: 200 (rig `reasoning_roundtrip/nonstreaming.yaml:27`). Tool continuation that sent back only `role` and `tool_calls`, with the reasoning entry left out: also 200 (rig `stream_encrypted_reasoning_survives_into_the_next_turn.yaml:61`, `openai/o4-mini`, 2026-10-02).
- Tool-call ids from Responses-served models are the upstream `call_id` (`call_7K8RytYgHp3bB8lU7QIppN3v`), never an `fc_` item id.
- On OpenRouter's own Responses route, message and function-call items get OpenRouter ids (`msg_tmp_ym1zsqcli1m`, `fc_tmp_iflee3r81p`, `ctc_tmp_aasptvj0fik`) even for OpenAI models, while reasoning items keep the upstream `rs_` id and function calls keep the upstream `call_id` (opencode `openrouter-responses/continues-a-tool-call.json`, 2026-11-18; RubyLLM 2026-11-18).
- Sources: WT-s1, WT-s3, WT-s4, WT-s7. Basis: OR-inferred (recordings).

**WT19. Only two published debug echoes were found, both for Claude. None was found for an OpenAI, Gemini, xAI, or DeepSeek model.**

- OpenRouter's own sample (errors and debugging page). Request: `model: 'anthropic/claude-haiku-5.5'`, a system and a user message, `debug: { echo_upstream_body: true }`. Printed upstream body: `"system": [ { "type": "text", "text": "You are a helpful assistant." } ]`, `"messages": [ { "role": "user", "content": "Hello!" } ]`, `"model": "claude-haiku-5-5"`, `"stream": true`, `"metadata": { "user_id": "<redacted>" }`, `"max_tokens": 65536`, `"stop_sequences": []`. So the system message becomes a `system` block array, the vendor's model id is used, a `max_tokens` the caller did not send is filled in, and `metadata.user_id` is set.
- The same page shows the Responses form: `"type": "response.debug"` with `"debug": { "echo_upstream_body": { "model": "claude-haiku-5-5", "messages": [...], "stream": true, "max_tokens": 65536 } }` and `"sequence_number": 0`.
- Third-party report (SillyTavern pull request 5617, opened 2026-05-09): "Verified with `curl` against OpenRouter using `debug.echo_upstream_body`:". For a request with only `reasoning: { exclude: false }` the author reports "upstream body has no `thinking` block, `native_tokens_reasoning: 0`." For `reasoning: { enabled: true }` the author reports "upstream body contains `thinking: { type: \"adaptive\", display: \"summarized\" }`, `native_tokens_reasoning > 0`." The pull request does not paste the whole body.
- Searched without result: the documentation mirror (the option is described on 8 pages, with upstream bodies only on the one above), OpenRouter's AI SDK provider tests (the option is passed, no echoed body is stored), four web searches for pasted echoes with `instructions`, `store`, `prompt_cache_key`, `thoughtSignature`, `reasoning_content`, `cache_control`. No recording in the eight sets sets `debug`.
- Sources: WT-s10, WT-s14. Basis: OR-documented (OpenRouter's sample); OR-inferred from an issue report (the SillyTavern text, third-party).

### DeepSeek, Kimi, and other open-weight reasoning (OQ7)

**WT20. Reasoning from DeepSeek, Kimi, Qwen, GLM, MiniMax, and gpt-oss arrives as `reasoning` plus one `reasoning.text` entry. The vendor's field name `reasoning_content` is in none of the 911 req_llm responses.**

- Quote (req_llm `moonshotai_kimi_k2_6/basic.json`, 2026-05-29, provider `DeepInfra`): `"reasoning": "The user said \"Hello world!\"...",` / `"reasoning_details": [ { "format": "unknown", "index": 0, "text": "The user said \"Hello world!\"...", "type": "reasoning.text" } ]`.
- Quote (Pydantic AI `...mandatory_reasoning_route[moonshotai-kimi-k2.7-code].yaml:70-74`, 2026-11-06, provider `StreamLake`): `reasoning:` text, then `reasoning_details:` with `type: reasoning.text`, `format: unknown`, `index: 0`.
- In 2025-11 several hosts returned `"format": null` on the same shape (req_llm: Chutes, DeepInfra, Novita, Together, Minimax).
- Replay: no recording sends reasoning back to a DeepSeek or Kimi model. req_llm's tool round trips send the assistant turn with `content`, `role`, `tool_calls` only, and all returned 200 (for example `moonshotai_kimi_k2_0905/tool_round_trip_2.json`, 2025-11-02, provider `SiliconFlow`; `deepseek_deepseek_chat_v3_1/tool_round_trip_2.json`, provider `Chutes`). Those are 2025 models. The DeepSeek V4 rule (reasoning must come back) has no recording.
- Sources: WT-s3, WT-s6. Basis: OR-inferred (recordings). What OpenRouter writes upstream on replay stays open.

**WT21. req_llm parses DeepSeek tool-call tokens out of the `reasoning` string, which implies some OpenRouter hosts returned them there.**

- Quote (req_llm `lib/req_llm/providers/openrouter.ex:1028-1030`): `defp parse_deepseek_tool_calls(reasoning) do` / `~r/<｜tool▁call▁begin｜>([^<]+)<｜tool▁sep｜>({[^}]+})<｜tool▁call▁end｜>/` / `|> Regex.scan(reasoning, capture: :all_but_first)`. The recovered calls get ids `"call_#{index}"`.
- No fixture in the current tree contains those tokens (searched all 911), so the leak itself is not recorded.
- Source: WT-s6. Basis: Filled (req_llm), and OR-inferred from client code that special-cases OpenRouter. Confidence: low for today.

### Cache keys and markers (OQ15, OQ16)

**WT22. No debug echo with a cache key or a cache marker after translation was found. The recordings show only the caller's side and the usage that came back.**

- OpenAI-native cache fields pass through the Responses route. Pydantic AI `test_openrouter_responses_prompt_cache_e2e.yaml` (2026-07-17, `openai/gpt-5.6-sol`, `provider: {only: [openai]}`): request has `prompt_cache_options:` with `mode: explicit`, `ttl: 30m` (lines 191, 472) and a content part with `prompt_cache_breakpoint:` (lines 22, 303). First reply: `cache_write_tokens: 4012`, `cached_tokens: 0`. Second reply: `cached_tokens: 4012`. So the explicit breakpoint took effect upstream.
- `cache_control` on a function tool for Claude is accepted: Pydantic AI `test_openrouter_cache_tool_definitions_e2e.yaml` (2026-11-04) sends one marker, on `tools[0]`, and gets 200. The prompt was 377 tokens, under the vendor minimum, so the usage shows zero cache activity and the recording cannot show what the marker became.
- Four markers in one request are accepted: `test_openrouter_limit_cache_points_e2e.yaml` (2026-11-04) has 4 `cache_control` markers and returned 200 with `cache_write_tokens: 1315`. Pydantic AI trims to four on its side, so a request with more than four is not recorded.
- For Gemini and for `openai/gpt-5-mini`, Pydantic AI removes `cache_control` from tools and unsupported parts before sending, so those cassettes hold no marker (`test_openrouter_cache_tool_definitions_gemini_e2e.yaml`, `test_openrouter_cache_point_unsupported_provider_e2e.yaml`).
- Not found anywhere: a request with the caller's `prompt_cache_key` on the chat route together with evidence that it reached OpenAI; any sign of `x-grok-conv-id`. req_llm documents its own option as "Prompt cache key, sent as the top-level `prompt_cache_key` body field. Requests sharing a key are routed to keep their cached prefix warm" (`lib/req_llm/providers/openrouter.ex:129`), which is a client's reading of OpenRouter's routing, not a recording.
- Sources: WT-s3, WT-s6. Basis: OR-inferred (recordings) for the three accepted cases; the rest is "no evidence found".

### Usage arithmetic with cache activity (OQ17)

The check used today's catalog prices (`pricing.prompt`, `pricing.input_cache_read`, `pricing.input_cache_write`, read 2026-11-22) and asked which split of `prompt_tokens` reproduces the recorded `cost_details.upstream_inference_prompt_cost`. P is `prompt_tokens`, C is `prompt_tokens_details.cached_tokens`, W is `prompt_tokens_details.cache_write_tokens`.

**WT23. Anthropic-served replies: `prompt_tokens` is the total, and cached and written tokens are disjoint parts inside it. Nine of nine recordings match `(P - C - W) x input + C x read + W x write` to the last digit.**

| Recording (Pydantic AI) | Provider | P | C | W | Recorded prompt cost | Formula result |
|---|---|---|---|---|---|---|
| `test_openrouter_effort_change_keeps_the_cache[anthropic].yaml:485-497`, 2026-10-31, `claude-opus-5.5` | `Claude Platform on AWS` | 10835 | 0 | 10831 | 0.054171 | 4 x 0.000004 + 10831 x 0.000005 = 0.054171 |
| same file, lines 1006-1018 | same | 10873 | 10831 | 38 | 0.0023722 | 4 x 0.000004 + 10831 x 0.0000002 + 38 x 0.000005 = 0.0023722 |
| `test_openrouter_cache_all_settings_real_api.yaml:246-258`, 2026-11-04, `claude-sonnet-5.5` | `Anthropic` | 3597 | 0 | 3593 | 0.0089905 | 4 x 0.000002 + 3593 x 0.0000025 = 0.0089905 |
| same file, lines 537-549 | `Anthropic` | 3655 | 3593 | 60 | 0.0005133 | 2 x 0.000002 + 3593 x 0.0000001 + 60 x 0.0000025 = 0.0005133 |
| same file, lines 799-811 | `Anthropic` | 3597 | 3512 | 81 | 0.0005617 | matches |
| `test_openrouter_effort_change_keeps_the_cache[anthropic-fable-5-2].yaml:486-498`, 2026-11-10, `claude-fable-5.2` | `Google` (Vertex) | 10835 | 0 | 10831 | 0.108342 | matches |
| same file, lines 1008-1020 | `Google` (Vertex) | 10873 | 10831 | 38 | 0.0025782 | matches |
| `test_openrouter_cache_messages_breakpoint_on_trailing_system_message_real_api.yaml`, 2026-11-04, `claude-haiku-5.5` | `Anthropic` | 4453 | 0 | 4451 | 0.000556575 | matches |
| same file, second exchange | `Anthropic` | 4473 | 4438 | 31 | 0.000048655 | matches |

- Three other splits were tested (cache tokens outside `prompt_tokens`; cached inside and write outside; write ignored). None of them matches any of the nine.
- So the three vendor counters (`input_tokens`, `cache_read_input_tokens`, `cache_creation_input_tokens`) are summed into `prompt_tokens`, on Anthropic direct, on Claude Platform on AWS, and on Vertex.
- Sources: WT-s3, WT-s13. Basis: OR-inferred (recorded traffic in Pydantic AI at `463a6d5`, 2026-10-31 to 2026-11-10, checked against the catalog). Confidence: high.

**WT24. OpenAI-served replies follow the same rule.**

- rig `prompt_caching/blocking_probe.yaml:16` (2026-10-02, `openai/gpt-4o-mini`): `"prompt_tokens":4597`, `"cached_tokens":3968`, `"cache_write_tokens":0`, `"upstream_inference_prompt_cost":0.00039195`. Check: 629 x 0.00000015 + 3968 x 0.000000075 = 0.00039195. Line 33: 4597, 4480, cost 0.00035355. Check: 117 x 0.00000015 + 4480 x 0.000000075 = 0.00035355.
- Pydantic AI `test_openrouter_effort_change_keeps_the_cache[openai].yaml:475-487` (2026-10-31, `openai/gpt-6.1-sol`, no cache marker in the request): P 8028, C 0, W 8025, cost 0.0200685 = 3 x 0.000002 + 8025 x 0.0000025. Lines 975-987: P 8058, C 8025, W 30, cost 0.0008835 = 3 x 0.000002 + 8025 x 0.0000001 + 30 x 0.0000025. So a newer OpenAI model reports cache writes without being asked, and the parts are disjoint.
- Sources: WT-s1, WT-s3, WT-s13. Basis: OR-inferred (recordings).

**WT25. Gemini replies break the rule: on the request that creates an explicit cache, `cached_tokens` and `cache_write_tokens` are the same tokens counted twice, and both are charged.**

- Pydantic AI `test_openrouter_cache_gemini_does_not_double_prompt_tokens.yaml:817-829` (2026-11-04, `google/gemini-3.5-flash-lite`, Vertex): `prompt_tokens: 11085`, `cached_tokens: 6901`, `cache_write_tokens: 6901`, `upstream_inference_prompt_cost: 0.0020373133333333334`. The disjoint formula gives a negative uncached count (11085 - 6901 - 6901 = -2717). The split that matches is `(P - C) x input + C x read + W x write`: 4184 x 0.0000003 + 6901 x 0.00000003 + 6901 x 0.0000000833 = 0.0020373133.
- Second exchange (lines 1665-1677): 11098, 11087, 11087, cost 0.0012598267, same split.
- AI Studio (`test_openrouter_cache_instructions_gemini_real_api.yaml:267-279`, `google/gemini-2.5-flash`): 2318, 2311, 2311, cost 0.00026401333, same split. On the next request (lines 547-559) the cache is only read: 2317, 2311, 0, cost 0.00007113.
- Pydantic AI's test explains a related quirk (`tests/models/openrouter/test_cache.py:955-956`): "With a `cache_control` on the final content part of the request, OpenRouter's Google Vertex endpoint bills the cached prefix and then the same prefix again, doubling `prompt_tokens`."
- Sources: WT-s3, WT-s13. Basis: OR-inferred (recordings checked against the catalog). Confidence: high for the arithmetic on these dates.

**WT26. On the Messages route the stream reported `input_tokens: 0` first and the real count later, with OpenRouter's cost fields inside Anthropic's `usage` object.**

- rig `crates/rig-cassette/fixtures/cassettes/anthropic/streaming/gateway_message_delta_metadata.yaml` (request path `/api/v1/messages`, header `anthropic-version: 2023-06-01`; added to the repository on 2026-09-19, the body has no dated id). Line 20, `message_start`: `{"message":{"container":null,"content":[],"id":"msg_REDACTED_1","model":"anthropic/claude-haiku-4.5","provider":"Amazon Bedrock","role":"assistant",...,"usage":{"cache_creation":null,"cache_creation_input_tokens":null,"cache_read_input_tokens":null,"inference_geo":null,"input_tokens":0,"output_tokens":0,"server_tool_use":null,"service_tier":null,"speed":"standard"}},"type":"message_start"}`.
- Line 41, `message_delta`: `"usage":{"cache_creation":{"ephemeral_1h_input_tokens":0,"ephemeral_5m_input_tokens":0},"cache_creation_input_tokens":0,"cache_read_input_tokens":0,"cost":0.000112,"cost_details":{"upstream_inference_completions_cost":0.00008,"upstream_inference_cost":0.000112,"upstream_inference_prompt_cost":0.000032},"input_tokens":32,"is_byok":false,"output_tokens":16,"output_tokens_details":{"thinking_tokens":0},...}`.
- rig's changelog states the same from its own traffic (`crates/rig-core/CHANGELOG.md:516`): "OpenRouter's Messages endpoint can send `input_tokens: 0` on `message_start` with the real count on `message_delta` (observed when it routes to an Amazon Bedrock upstream)".
- Source: WT-s1. Basis: OR-inferred (recorded traffic in rig at `a8673f7`, recorded on or before 2026-09-19).

### Tool-call ids (OQ20)

**WT27. Tool-call ids are whatever the upstream issued. Formats recorded per upstream:**

| Upstream (`provider`) | Format | Example | Recorded |
|---|---|---|---|
| `Anthropic` | `toolu_` + 24 | `toolu_019wLneJ49MzifRCUjwyowgM` | 2025-11-02 to 2026-11-04 |
| `Amazon Bedrock` (Claude) | `toolu_bdrk_` + 24 | `toolu_bdrk_01VzaKHoRgZVCVCqesM54vz6` | 2025-11-02 to 2026-11-18 |
| `Google` (Claude on Vertex) | `toolu_vrtx_` + 24 | `toolu_vrtx_01Xc6A4DN2XARWHp1sTjECbt` | 2025-11-02 to 2026-10-03 |
| `OpenAI`, `Azure` (both protocols) | `call_` + 24 letters and digits | `call_R9TubcSGVuRlo8NfFF9le3wY` | 2025-11-02 to 2026-11-18 |
| `Google`, `Google AI Studio` (Gemini) | `tool_0_<name>_` + 20 on 2025-11-02; `tool_<name>_` + 20 on 2025-12-03 and 2025-12-23; `call_` + six digits, consecutive within a reply, on 2026-10-27 | `tool_0_get_weather_vT4yDZZzHnBRYetdsGq5`; `tool_add_QpVaa8pTdTtaG8iAAvhe`; `call_172647`, `call_172648` | as shown |
| `Mistral` | 9 letters and digits | `3sniiMddS`, `cQUVeBqlm` | 2025-10-15 to 2025-11-02 |
| `Moonshot AI` (Kimi), also Novita and Fireworks for Kimi | `<name>:<n>` and `functions.<name>:<n>` | `get_weather:0`, `functions.get_weather:0` | 2025-11-02 |
| `xAI` | `call_` + 8 digits | `call_36723511` | 2025-11-02 |
| `DeepInfra` | `call_` + 4 hex, or `functions.<name>_<4 hex>` for gpt-oss | `call_2ff1`, `functions.add_21f2` | 2025-11-02 |
| `Nebius`, `BaseTen`, `Phala`, `WandB`, `Novita` | `chatcmpl-tool-` + 32 hex | `chatcmpl-tool-81b24eff03ca4fe381ff025a214c6037` | 2025-11-02 |
| `Alibaba`, `Chutes`, `Parasail`, `Z.AI`, `StreamLake` | `call_` + 24 hex | `call_fc83dccb37d94599a5ea19c5` | 2025-10-15 to 2026-11-17 |
| `SiliconFlow` | 32 hex, no prefix | `019a45a5c5913cfc3ae2ca3dbeaab0cf` | 2025-10-07 to 2025-11-02 |
| `Minimax`, `Novita` (MiniMax) | `call_function_<digits>_<n>` | `call_function_7055506536_1` | 2025-11-02 |
| `Groq` | `fc_` + UUID | `fc_66816916-59e8-4a91-98c9-b86733d5d0e6` | 2025-11-02 |
| `Mancer 2` | `toolcall-` + 16 | `toolcall-61d104dcNe2OVP2l` | 2025-11-02 |

- Sources: WT-s1, WT-s3, WT-s6, WT-s7. Basis: OR-inferred (recordings). The same model served by different providers returns different id formats in the same minute.

**WT28. Ids from one upstream were replayed to another upstream and accepted. No recording shows an id being rewritten on the way back, and no recording sends a foreign id to Mistral.**

- Across vendors: a Bedrock Claude id sent to Gemini, `"id":"toolu_bdrk_01VzaKHoRgZVCVCqesM54vz6"` with model `google/gemini-3-flash-preview`: 200 (rig `switch_unary.yaml:27`, 2026-10-02). An OpenAI id sent to Claude on Bedrock, `call_izjasbuE1BsgJbxnA7svUiMH`: 200 (rig `portability_matrix/from_openai_responses.yaml`, 2026-10-02).
- Across providers of one model (req_llm, 2025-11-02): `toolu_bdrk_01X2o24kU5mMoX5AtUQFH3pp` issued by Amazon Bedrock, next turn served by `Anthropic`: 200. `chatcmpl-tool-4f85a55bd81842cbaa745bb2788a8ea8` (Nebius) to `DeepInfra`: 200. `add:0` (Novita) to `SiliconFlow`, and `add:0` (Moonshot AI) to `Groq`: 200. `chatcmpl-tool-1b39f75aa6dc40b796299253dfe59f2b` (Novita) to `Chutes`: 200.
- Invented ids: `call_history_single` and `call_history_alpha` to `openai/gpt-4.1-mini` on the Responses upstream (rig, 2026-10-03); `call_1`, `call_2`, `toolu_01` to Claude (Pydantic AI, 2026-10-31 and 2026-11-04); `call_treasury_duration_1` on the Responses route to Claude (2026-10-26). All 200.
- What it cannot show: whether OpenRouter rewrote any of these ids before the upstream saw them (that needs a debug echo), and what happens with an id that breaks a vendor's own rule, such as Mistral's nine characters.
- Sources: WT-s1, WT-s3, WT-s6. Basis: OR-inferred (recordings).

### Tool call cut by the output limit (OQ23)

**WT29. Independent re-read of the truncation fixtures: the neutral field says `tool_calls`, the native field says the output ran out, and the arguments are incomplete JSON.**

- Stream, Chat Completions upstream (rig at `a023bd5b`, `tool_truncation_contract_matrix/streaming_gpt4o_low_agent.yaml`, 2026-09-23 08:37 UTC, `openai/gpt-4o-mini`, request line 10 has `"max_tokens":16`, `"tool_choice":"required"`). Lines 17-33: argument fragments `""`, `""`, `"{\""`, `"summary"`, `"\":\""`, `"The"`, `" cache"`, `" warmer"`, `" raced"`. Line 35: `{"choices":[{"delta":{"content":"","role":"assistant"},"finish_reason":"tool_calls","index":0,"native_finish_reason":"length"}],...}`. Line 37: the usage chunk with the same two values and `"completion_tokens":16`. Line 39: `data: [DONE]`. The joined arguments are `{"summary":"The cache warmer raced`.
- Non-streaming, Responses upstream (`tool_truncation_contract_matrix/blocking_gpt41_low_agent.yaml:16`, `openai/gpt-4.1-mini`): `"finish_reason":"tool_calls"`, `"native_finish_reason":"max_output_tokens"`, `"arguments":"{\"summary\":\"The cache warmer raced the artifact uploader, the retry storm saturated the"`.
- Of the 24 fixtures in that folder, the 16 "low" and "mid" budget ones show the pattern (8 with native `length`, 8 with native `max_output_tokens`), and the 8 "complete" ones show `tool_calls` with native `tool_calls` or `completed`.
- The folder was deleted from rig's tree on 2026-10-05; the replies survive in `crates/rig-cassette/fixtures/bank/openrouter.yaml` at the current commit (entries sourced from `openrouter/tool_truncation_contract_matrix/...`, for example at line 1384), which is what the first-wave item O78 cited. This re-read agrees with O78.
- Sources: WT-s2, WT-s1, WT-s15. Basis: OR-inferred (recorded traffic in rig, 2026-09-23). Recorded for OpenAI only.

**WT30. More cases where the neutral `finish_reason` is derived from the content, not from the native value.**

- Forced tool choice: `"finish_reason":"tool_calls","native_finish_reason":"stop"` (Kilo Code `openrouter-streams-tool-call.json:24`, 2026-05-06, `openai/gpt-4o-mini`, request `tool_choice` names the function). The native `stop` is the upstream's own word for that case, and OpenRouter's neutral field replaces it with `tool_calls`.
- Gemini tool calls: `tool_calls` beside native `STOP` in every Gemini tool reply from 2025-12 on (for example Pydantic AI `test_openrouter_google_inlines_property_names_and_contains_refs.yaml:115`).
- Gemini malformed call (req_llm `google_gemini_2_5_flash_lite/object_streaming.json`, 2025-11-02, decoded stream lines 7-37): fifteen chunks each with a whole tool call, each with `"finish_reason":"tool_calls","native_finish_reason":null` and each with `"index":0` and a different id, then one chunk with `"finish_reason":"stop","native_finish_reason":"MALFORMED_FUNCTION_CALL"` and a top-level `error` (quoted in WT34).
- Sources: WT-s3, WT-s5, WT-s6. Basis: OR-inferred (recordings).

### Server tool records (OQ25)

**WT31. The `rs_server_tool_round_*` item is a plain-text membership record of one server-tool round.**

- RubyLLM `protocols_openrouter_responses_executes_a_hosted_shell_and_returns_its_real_output_and_billed_usage.yml`, exchange 0 (recorded 2026-11-18 06:34 UTC, `POST /api/v1/responses`, `openai/gpt-5.2`, `tools: [{"type": "openrouter:shell", "parameters": {"engine": "openrouter"}}]`, `store: false`, `include: ["reasoning.encrypted_content"]`). The `output` array has four items in this order:
  1. `{"id": "rs_0c095071aba48b48016afd46fc1d4887d1aa3b1833f88e53f1", "type": "reasoning", "status": "completed", "summary": [], "encrypted_content": "gAAAAABq_Ub8jTK...=.eyJlbmRwb2ludF9zbHVnIjoib3BlbmFpL2dwdC01LjItMjAyNTEyMTF8b3BlbmFpIn0", "format": "openai-responses-v1"}`
  2. `{"type": "openrouter:shell", "id": "st_tmp_ch0b7276x1w", "status": "completed", "action": {"commands": ["python3 -c \"print(17*23)\""], "max_output_length": 2000, "timeout_ms": 100000}, "output": [{"stdout": "391", "stderr": "", "outcome": {"type": "exit", "exit_code": 0}}], "container_id": "gen_a96ecbb6d077", "call_id": "call_frblB9ms1HEasdWb6iahoO6Q", "arguments": "{\"commands\":[\"python3 -c \\\"print(17*23)\\\"\"],\"timeout_ms\":100000,\"max_output_length\":2000}"}`
  3. `{"id": "rs_server_tool_round_inz2g5vgo8p", "type": "reasoning", "status": "completed", "summary": [], "encrypted_content": "openrouter-server-tool.eyJzZXJ2ZXJfdG9vbF9yb3VuZCI6eyJ2IjoxLCJtZW1iZXJzIjpbeyJpdGVtX2lkIjoicnNfMGMwOTUwNzFhYmE0OGI0ODAxNmFmZDQ2ZmMxZDQ4ODdkMWFhM2IxODMzZjg4ZTUzZjEifSx7Iml0ZW1faWQiOiJzdF90bXBfY2gwYjcyNzZ4MXciLCJjYWxsX2lkIjoiY2FsbF9mcmJsQjltczFIRWFzZFdiNmlhaG9PNlEifV19fQ"}`
  4. the final message item, `"text": "391"`.
- The part after `openrouter-server-tool.` decodes (base64url) to `{"server_tool_round":{"v":1,"members":[{"item_id":"rs_0c095071aba48b48016afd46fc1d4887d1aa3b1833f88e53f1"},{"item_id":"st_tmp_ch0b7276x1w","call_id":"call_frblB9ms1HEasdWb6iahoO6Q"}]}}`.
- Reading: the record lists the items that formed one round (the upstream reasoning item and the server tool call with the upstream `call_id`). Despite the field name, the content is readable base64 JSON, and the item has no `format` key. With `store: false` the caller owns the history, so this item is what lets OpenRouter regroup the round when the items come back.
- Usage of the same reply: `"server_tool_cost": 0.0030000000000000005` inside `cost_details` and `"server_tool_use_details": {"tool_calls_requested": 1, "tool_calls_executed": 1}`.
- What it cannot show: whether a replay without the record fails. The cassette has no follow-up request.
- Source: WT-s7. Basis: OR-inferred (recorded traffic in RubyLLM at `2af24ff`, 2026-11-18).

**WT32. On the chat route, recorded server-tool runs leave no entry in the message. Zero `reasoning.server_tool_call` entries exist in any recording.**

- `openrouter:advisor` (Pydantic AI `test_openrouter_advisor_tool.yaml`, 2026-07-24, `openai/gpt-4o-mini`): the reply message has four keys, `content`, `reasoning: null`, `refusal: null`, `role`. The run shows only in `"server_tool_use_details": {"tool_calls_executed": 1, "tool_calls_requested": 1}` and in `cost` (0.0160614 against `upstream_inference_cost` 0.0001764).
- Streamed twin (`test_openrouter_advisor_tool_stream.yaml`): thirty `: OPENROUTER PROCESSING` comments, then content deltas only.
- `openrouter:web_search` (Pydantic AI `test_openrouter_web_search_annotations.yaml`, 2026-08-14, `deepseek/deepseek-chat`): `message.annotations` holds five `url_citation` objects with `content`, `title`, `url`, `start_index: 0`, `end_index: 0`; usage has `"web_search_requests": 1`. The reply's `provider` field reads `"OpenAI"` although the model is DeepSeek's.
- Searched for the literal `server_tool_call` in all eight sets: zero hits in OpenRouter responses.
- Sources: WT-s3. Basis: OR-inferred (recordings from 2026-07 and 2026-08). The documented `reasoning.server_tool_call` entry may postdate them or may apply to cases not recorded.

**WT33. Vendor-run tools pass through with the vendor's own item types on the Responses route, and flatten on the chat route.**

- Responses route, `openai/gpt-5.2`, 2026-11-18 (RubyLLM): OpenAI web search as `{"type": "web_search_call", "id": "ws_001fdad0d954dfa0016afd4706238887d195e6e2bde8b713a0", "status": "completed", "action": {"type": "search", "query": "latest stable Ruby version", ...}}` between two reasoning items; remote MCP as `mcp_list_tools` (`mcpl_...`), `mcp_approval_request` (`mcpr_...`), and `mcp_call` (`mcp_...`, with `output`, `error: null`, `approval_request_id`). A stateless continuation that sent the approval back returned the completed `mcp_call` (cassette `chat_remote_mcp_tools_openrouter_executes_a_remote_read-only_tool_after_approval_using_stateless_continuation.yml`).
- Chat route, same model and day: the reply has the alternating summary and encrypted entries of WT13 and `"server_tool_use_details": {"web_search_requests": 3}`. The searches themselves are not in the message.
- Source: WT-s7. Basis: OR-inferred (recordings).

### Streams: errors, keep-alives, chunk fields (OQ30)

**WT34. Two recordings hold a chat stream with an error chunk. In one, `data: [DONE]` follows. The other capture ends at the usage chunk. In neither is `finish_reason` equal to `"error"`.**

- Pydantic AI `tests/models/cassettes/test_openrouter/test_openrouter_stream_error.yaml` (id `gen-1762179802-...`, 2025-11-03 14:23 UTC, `minimax/minimax-m2:free`, request `max_completion_tokens: 10`). Lines 30-62: seventeen `: OPENROUTER PROCESSING` comments. Lines 64-68: three chunks, the last two with `"finish_reason":"length","native_finish_reason":"length"`. Line 70: `data: {"id":"gen-1762179802-UN8pkJI4AGZvryk0kFnb","provider":"Minimax","error":{"code":400,"message":"Token limit reached"},"model":"minimax/minimax-m2:free","object":"chat.completion.chunk","created":1762179802,"choices":[{"index":0,"delta":{"role":"assistant","content":""},"finish_reason":null,"native_finish_reason":null,"logprobs":null}],"usage":{"prompt_tokens":43,"completion_tokens":10,"total_tokens":53,"cost":0,...}}`. Line 72: `data: [DONE]`. The recorder stores the whole HTTP body, so the `[DONE]` is OpenRouter's.
- req_llm `google_gemini_2_5_flash_lite/object_streaming.json` (id `gen-1762104948-...`, 2025-11-02 17:35 UTC, byte-exact chunks). Decoded line 37: `data: {"id":"gen-1762104948-zKpcZ11QzTPiw9B9xeH0","provider":"Google","model":"google/gemini-2.5-flash-lite","object":"chat.completion.chunk","created":1762104948,"choices":[{"index":0,"delta":{"role":"assistant","content":"","reasoning":null,"reasoning_details":[]},"finish_reason":"stop","native_finish_reason":"MALFORMED_FUNCTION_CALL","logprobs":null}],"error":{"message":"Provider returned error","code":502,"metadata":{"raw":"Malformed function call: call:structured_output{age:39,name:"}}}`. Decoded line 39: a usage chunk with `"finish_reason":null,"native_finish_reason":null` and `"completion_tokens":243`. The capture ends there, in a separate network chunk from the error. It is the only one of 161 req_llm streams whose capture ends without `data: [DONE]`.
- Caveat on the second: req_llm's stream server saves the fixture when its decoder marks a chunk terminal (`lib/req_llm/stream_server.ex:933-934`: `if Decoder.terminal_chunk?(chunk) do` / `finalize_stream_with_fixture(%{new_state | terminated?: true})`), so the capture may have stopped before the HTTP body did.
- Both error chunks keep a non-empty `choices` array and the top-level `error` object; one also has `usage` in the same chunk. The error codes inside are 400 and 502 while the HTTP status was 200.
- No recording from 2026 holds a mid-stream error.
- Sources: WT-s3, WT-s6. Basis: OR-inferred (recorded traffic in Pydantic AI at `463a6d5` and req_llm at `0e49ee9`, 2025-11-02 and 2025-11-03). Confidence: high that `[DONE]` followed on 2025-11-03; the second case is inconclusive.

**WT35. Keep-alive comments: exact text, spacing, and position.**

- Chat route: the line `: OPENROUTER PROCESSING` followed by one empty line. Byte-exact (req_llm chunk decode): `: OPENROUTER PROCESSING\n\n`. Counted: 846 in req_llm (2025-10 to 2026-10), 121 in Pydantic AI, 57 in rig, 12 in RubyLLM (2026-11-18).
- Responses route: the line `: ` (colon, space) followed by one empty line. rig `upstream_switch_matrix/responses_switch_streamed.yaml:137` (2026-10-02) has `\n\n: \n\n: \n\n: \n\n` between two reasoning deltas; llm-openrouter `test_reasoning.yaml` (2026-08-04) starts with `: \n\ndata: {"type":"response.created",...`; RubyLLM has 4 (2026-11-18). One older Responses recording has the chat text (Pydantic AI `test_openai_responses_raw_cot_stream_openrouter.yaml`, 2025-11-27).
- Position: before the first data line (up to 30 in a row while a server tool ran, WT32) and between data lines (rig `reasoning_roundtrip/streaming.yaml`, second exchange: 9 comments, 90 chunks, 8 comments, 110 chunks, 7 comments, 972 chunks).
- The interval between comments cannot be read from any fixture: none stores arrival times.
- Sources: WT-s1, WT-s3, WT-s6, WT-s7, WT-s8. Basis: OR-inferred (recordings).

**WT36. Chunk fields on the chat route.**

- `provider` is on every chunk: 18,528 of 18,528 chunks in req_llm, 1,172 of 1,172 in rig's longest stream. `id`, `model`, `object`, `created` likewise.
- `native_finish_reason` is a key on every choice of every chunk, `null` until the end (rig: 1,172 of 1,172).
- The end of a stream is two chunks and `[DONE]`. Quote (Pydantic AI `test_openrouter_cache_streaming_e2e.yaml:88-92`, 2026-11-04, original key order): `data: {"id":"gen-1793763646-kd933MPfBckz0HNxjg7D","object":"chat.completion.chunk","created":1793763646,"model":"anthropic/claude-sonnet-5.5","provider":"Claude Platform on AWS","choices":[{"index":0,"delta":{"content":"","role":"assistant"},"finish_reason":"stop","native_finish_reason":"end_turn"}]}` then `data: {"id":"gen-1793763646-kd933MPfBckz0HNxjg7D","object":"chat.completion.chunk","created":1793763646,"model":"anthropic/claude-sonnet-5.5","provider":"Claude Platform on AWS","service_tier":"default","choices":[{"index":0,"delta":{"content":"","role":"assistant"},"finish_reason":"stop","native_finish_reason":"end_turn"}],"usage":{"prompt_tokens":667,"completion_tokens":6,"total_tokens":673,"cost":0.0001583,"is_byok":false,"prompt_tokens_details":{"cached_tokens":653,"cache_write_tokens":10,"audio_tokens":0,"video_tokens":0},"cost_details":{...},"completion_tokens_details":{"reasoning_tokens":0,"image_tokens":0,"audio_tokens":0}}}` then `data: [DONE]`.
- So the usage chunk has a non-empty `choices` array that repeats both finish values, and `service_tier` appears on that chunk only. This confirms survey claim LC12 on recorded traffic.
- The usage chunk arrives even when the request has no `stream_options`: rig's streaming requests send `"stream":true` alone and every one ends with a usage chunk (for example `upstream_switch_matrix/switch_streamed.yaml:10` and `:89`).
- When reasoning was streamed, the first finish chunk has `"reasoning":null` in its delta: `{"delta":{"content":"","reasoning":null,"role":"assistant"},"finish_reason":"tool_calls",...}` (rig `switch_streamed.yaml:87`).
- JSON key order changed between 2025 (`id`, `provider`, `model`, `object`, `created`, `choices`) and 2026 (`id`, `object`, `created`, `model`, `provider`, `choices`).
- Sources: WT-s1, WT-s3, WT-s6. Basis: OR-inferred (recordings).

**WT37. The Responses route streams `data:` lines only and ends with `data: [DONE]`.**

- llm-openrouter `test_reasoning.yaml` (2026-08-04, `openai/gpt-5-nano`): the body has 12 `data:` lines and zero `event:` lines; it begins `: \n\ndata: {"type":"response.created","response":{"id":"gen-1785882693-G0Ba9AGVEVeNhHohmw0g","object":"response",...` and ends `..."sequence_number":10}\n\ndata: [DONE]\n\n`.
- Same framing in rig (2026-09-22 and 2026-10-02, Gemini and Claude models) and RubyLLM (2026-11-18, `openai/gpt-5.2`). rig's recorder keeps `event:` lines where they exist (its direct OpenAI fixtures have them), so their absence here is OpenRouter's.
- For comparison, rig's recording of OpenAI's own Responses stream (`fixtures/cassettes/openai/checkpoint_matrix_responses/multi_turn_streamed.yaml`) has `event: response.created` style lines, ends at `event: response.completed`, and has zero `[DONE]` lines. So a client written for OpenAI's framing must take the event name from the JSON `type` field here and must accept the extra `[DONE]`.
- Sources: WT-s1, WT-s7, WT-s8. Basis: OR-inferred (three independent recorders).

**WT38. The Messages route stream ends with an extra frame.**

- rig `fixtures/cassettes/anthropic/streaming/gateway_message_delta_metadata.yaml:43-47`: `event: message_stop` / `data: {"type":"message_stop"}` / blank / `event: data` / `data: [DONE]`. rig's recording of Anthropic's own stream (`fixtures/cassettes/anthropic/corpus_endings/turn_finished_stop_streamed.yaml`) ends at `event: message_stop` and has zero `[DONE]` lines.
- The same fixture shows `event:` lines on this route (`event: message_start`, `event: content_block_delta`, and so on), and `"stop_reason":"max_tokens"` in `message_delta` for a request with `"max_tokens":16`.
- Source: WT-s1. Basis: OR-inferred (recording, on or before 2026-09-19).

### Error bodies and response headers

**WT39. A context overflow rejected by OpenRouter itself came back without a typed error code, four days before this research.**

- RubyLLM `chat_real_error_scenarios_openrouter_claude-haiku-4-5_handles_context_length_exceeded_errors.yml` (recorded 2026-11-18 06:40:22 GMT, `POST /api/v1/chat/completions`, `stream: false`). Status 400. Whole body: `{"error": {"message": "This endpoint's maximum context length is 200000 tokens. However, you requested about 1500007 tokens (1500007 of text input). Please reduce the length of the input, or use the context-compression plugin to compress your prompt automatically.", "code": 400, "metadata": {"provider_name": null}}}`.
- Response headers on that reply: `X-Openrouter-Error-Origin: client`, `X-Generation-Id: gen-1794984022-haKIDC65tel0J1hc3mWv`, and `Access-Control-Expose-Headers: WWW-Authenticate,PAYMENT-REQUIRED,Payment-Receipt,PAYMENT-RESPONSE,Location,X-Generation-Id,x-openrouter-error-origin,x-openrouter-error-tool,X-Provider-Name,request-id,cf-ray`.
- The `error` object has three keys (`message`, `code`, `metadata`) and `metadata` has one (`provider_name`). The string `error_type` is not in the body.
- OpenRouter's errors page (read 2026-11-22) says where the typed code sits on Chat Completions: `error.metadata.error_type`, "on the mid-stream error chunk (see [Mid-Stream Errors](#mid-stream-errors)) and on the non-streaming response when a provider error interrupts generation." The recorded rejection happened before any provider was called (`provider_name: null`, origin `client`), which is outside that sentence.
- The header names `x-openrouter-error-origin` and `x-openrouter-error-tool` have zero occurrences in the documentation mirror.
- Source: WT-s7, WT-s10. Basis: OR-inferred (recorded traffic in RubyLLM at `2af24ff`, 2026-11-18) against OR-documented text. Confidence: high for the recording.

**WT40. Other recorded error bodies.**

- Provider 400 with a retry trail (Pydantic AI `test_openrouter_forced_tool_choice_while_reasoning_provider_400_without_flag.yaml:46-82`, 2026-11-17, `qwen/qwen3.8-flash`): `message: Provider returned error`, `code: 400`, `metadata.raw` holding the vendor's SSE line `data: {"error":{"code":"invalid_parameter_error","param":null,"message":"The tool_choice parameter does not support being set to required or object in thinking mode","type":"invalid_request_error"},"id":"chatcmpl-90b95656-eb93-9431-a37f-86e8714852de"}`, `provider_name: Alibaba`, `is_byok: false`, and `previous_errors:` listing the same error from the earlier attempt. This body too has `metadata.raw` and `provider_name` and lacks `error_type`.
- Provider 429 (Pydantic AI `test_openrouter_errors_raised.yaml:45-48`, undated, model `google/gemini-2.0-flash-exp:free`): `code: 429`, `message: Provider returned error`, `metadata.provider_name: Google`, `metadata.raw: "google/gemini-2.0-flash-exp:free is temporarily rate-limited upstream. Please retry shortly, or add your own key to accumulate your rate limits: https://openrouter.ai/settings/integrations"`.
- No tool-capable endpoint (req_llm `minimax_minimax_01/tool_round_trip_1.json`, 2025-11-02): HTTP 404, `{"code": 404, "message": "No endpoints found that support tool use. To learn more about provider routing, visit: https://openrouter.ai/docs/provider-routing"}`.
- Unknown model (rig at `a023bd5b`, `response_identity_edge/routed_failure_error_shape.yaml:16`; the body has no dated id, and the file was in the tree on 2026-10-04): HTTP 400, `{"error":{"code":400,"message":"openai/gpt-nonexistent-routed-model is not a valid model ID"},"user_id":"user_REDACTED"}`. A top-level `user_id` sits beside `error`.
- Missing key (RubyLLM, 2026-11-18): HTTP 401, `{"error": {"message": "Missing Authentication header", "code": 401}}`.
- Across all eight sets, the string `error_type` is in none of the recorded OpenRouter error bodies or error chunks (dates 2025-11-02 to 2026-11-18). The 2026-11-17 body above also has a top-level `user_id` beside `error`.
- Sources: WT-s2, WT-s3, WT-s6, WT-s7. Basis: OR-inferred (recordings).

**WT41. Response headers on 202 RubyLLM exchanges (2026-11-18): OpenRouter's own headers are present and vendor headers are not.**

- Present: `X-Generation-Id` (98 replies), `X-Openrouter-Release` (87), `X-Openrouter-Error-Origin: client` (2), plus Cloudflare and CORS headers. `X-Provider-Name` is on 6 replies, all from the image and decisions routes (values `Google`, `Cloudflare`, `TypeSafe`), and on none of the chat or Responses replies.
- The set of header names across all 202 replies is 21 names. None of them is a vendor header such as `anthropic-*`, `openai-*`, `x-ratelimit-*`, `retry-after`, or a vendor request id. Three batch-creation replies (`POST /api/v1/batches`) have `traceparent` and `x-cloud-trace-context`, which are infrastructure trace headers.
- Earlier recordings show the exposed-header list growing: `X-Generation-Id,cf-ray` (req_llm, 2026-05-29), `X-Generation-Id,X-Provider-Name,request-id,cf-ray` (Pydantic AI, 2026-10-26), `X-Generation-Id,x-openrouter-error-origin,X-Provider-Name,request-id,cf-ray` (2026-11-04), and the eleven-name list of WT39 (2026-11-18).
- Sources: WT-s3, WT-s6, WT-s7. Basis: OR-inferred (recordings). It answers "are vendor response headers forwarded" with no for these replies; the recorder may drop headers it does not know, which was not audited.

### Out-of-spec fields and tool-call emulation (OQ35, OQ19)

**WT42. Out-of-spec request fields that were recorded.**

- `n: 2` returns one choice. rig `terminal_metadata_matrix/blocking_gpt_4_1_mini_tiny_plain_two.yaml` (2026-09-23): request has `"n":2`, the reply's `choices` array has one element with `"index":0`. Streaming twin: all 18 chunks have choice index 0. rig's test says (`tests/providers/openrouter/cassette/terminal_metadata_matrix.rs:10-12`): "//! OpenRouter accepts `n = 2` but emits only candidate zero, and those cells" / "//! pin that observed gateway contract instead of assuming direct OpenAI" / "//! behavior."
- `transforms: ["middle-out"]` was accepted with 200 on 2025-10-03 and 2025-11-02 (Pydantic AI `test_openrouter_with_native_options.yaml`, `test_openrouter_stream_with_native_options.yaml`). No later recording sends it.
- `usage: {"include": true}` was accepted with 200 on 2025-11-10 (Pydantic AI `test_openrouter_usage.yaml`, exchange 1; the reply gained `cost` and detail objects that exchange 0 lacks) and on 2026-07-18 (opencode). From 2026 on, replies have `cost` without it.
- `preset: "@preset/comedian"` accepted on 2025-10-03. `seed`, `n: 1`, `reasoning_effort`, `max_completion_tokens`, `stream_options` accepted throughout.
- No recording sends `store`, `audio`, or `prediction` on the chat route.
- Sources: WT-s1, WT-s3, WT-s4, WT-s6. Basis: OR-inferred (recordings).

**WT43. Tool-call emulation: no recording shows OpenRouter's YAML template at work.**

- When no endpoint of a model supported tools, the request failed with 404 instead of being emulated (WT40, `minimax/minimax-01`, 2025-11-02).
- One reply has template residue beside a parsed call (req_llm `nousresearch_hermes_4_70b/tool_round_trip_1.json`, 2025-11-02, provider `Chutes`): `"content": "<tool_call>\n</tool_call>"` together with `"tool_calls": [{"function": {"arguments": "{\"a\": 2, \"b\": 3}", "name": "add"}, "id": "call_2dc1ed497d5b47dea3edb141", ...}]`. Whether OpenRouter or the host parsed the call cannot be told from the reply.
- No request or response in the eight sets contains YAML tool instructions. Requests are the callers' own, and the upstream body is not visible without the debug echo.
- Sources: WT-s6. Basis: OR-inferred (recordings); the question itself stays open.

### Drift worth knowing about

**WT44. `native_finish_reason` for Claude changed vocabulary between late 2025 and 2026.**

- 2025-11-02 (req_llm): provider `Anthropic` returned native `stop` (6), `tool_calls` (5), `length` (2); provider `Google` (Claude on Vertex) returned `tool_calls` and `length`; provider `Amazon Bedrock` returned a mix of `max_tokens`, `end_turn`, `stop`, `tool_calls`. 2025-11-28 (Pydantic AI `test_openrouter_tool_optional_parameters.yaml:70-78`): `claude-sonnet-4.5` on `Google`, `native_finish_reason: tool_calls`.
- 2026-05-06 onward (Kilo Code, opencode, rig, Pydantic AI, RubyLLM): `tool_use`, `end_turn`, `max_tokens` on every Claude provider.
- So the native field held OpenAI-style words for Claude on two of three providers in 2025.
- Sources: WT-s3, WT-s5, WT-s6. Basis: OR-inferred (recordings).

**WT45. Smaller observations.**

- The reply's `model` field is sometimes the dated slug: `"model": "moonshotai/kimi-k2.6-20260420"` for a request to `moonshotai/kimi-k2.6` (req_llm, 2026-05-29). Most replies echo the requested id.
- A tool continuation for one model can be served by a different provider than the turn that issued the call, when no thinking is involved (req_llm `anthropic_claude_haiku_4_5`: `tool_round_trip_1.json` by `Amazon Bedrock`, `tool_round_trip_2.json` by `Anthropic`, 2025-11-02).
- `usage.cache_creation: {"ephemeral_5m_input_tokens": 0, "ephemeral_1h_input_tokens": 0}` appeared on a chat reply from provider `Anthropic` on 2026-07-18 (opencode) and is not in later chat recordings.
- `service_tier` in chat replies took the values `default`, `provisioned` (Gemini on Vertex, rig `switch_streamed.yaml:116`), and `null`.
- Sources: WT-s1, WT-s4, WT-s6. Basis: OR-inferred (recordings).

## Conflicts and disagreements

1. **`[DONE]` after a mid-stream error: two recordings one day apart differ.** Pydantic AI's cassette of 2025-11-03 has `data: [DONE]` right after the error chunk. req_llm's capture of 2025-11-02 ends at the usage chunk that follows the error chunk. The first recorder stores the whole body, the second can stop early by its own rule (WT34). The Pydantic AI recording is the one to trust for "the sentinel followed on that day". Neither is evidence for 2026.
2. **`finish_reason` on the error chunk.** OpenRouter's documentation today shows `"finish_reason":"error"` on the mid-stream error chunk (first-wave item O42, survey claim LC11). The two recorded error chunks from 2025-11 have `"finish_reason":null` and `"finish_reason":"stop"` (WT34). The likely reading is that the documented value came later; no 2026 recording of a mid-stream error exists to confirm it. A reader that only looks for `finish_reason: "error"` would have missed both recorded errors.
3. **Survey claim LC13, context overflow is `error_type: context_length_exceeded`.** A recording from 2026-11-18 shows OpenRouter's own length check answering 400 with a body in which `error_type` is missing (WT39). The documentation sentence that places `error.metadata.error_type` on chat responses is about provider errors, so the two do not contradict each other word for word, but the survey claim as written is too broad: on that day the only signals for a pre-send overflow were the message text and the `X-Openrouter-Error-Origin: client` header. A provider-side 400 recorded on 2026-11-17 also lacks `error_type` (WT40), which does disagree with the documented placement and deserves a re-check.
4. **Gemini and the endpoint tag.** OpenRouter's image understanding guide shows a Gemini reasoning item on the Responses route with the tag (WT6). Every third-party chat-route recording of a Gemini signature through 2026-11-06, including one pinned to Vertex, has Google's string alone. Both are true as observed. Which rule joins them (route, date, or endpoint) is unknown.
5. **Start date of the Anthropic tag.** The first-wave Rust and Python file (item G37) put it "between 2026-09-22 and 2026-10-02". opencode's recording of 2026-09-29 narrows the window to 2026-09-22 16:48 UTC to 2026-09-29 16:18 UTC. The same first-wave item did not say that OpenAI values already had the tag on 2026-09-22, or that it is older there (2026-08-04 at the latest) (WT3).
6. **Which OpenAI models use the Responses API upstream.** The first-wave OpenAI file (item OA10) wrote "Not shown for non-reasoning models such as `openai/gpt-4o`, which could be served through either upstream API." The recordings show it: `gpt-4o` and `gpt-4o-mini` return Chat Completions words in `native_finish_reason` on every date from 2025-11 to 2026-10, and `gpt-4.1-mini` returns Responses words (WT16). One `gpt-4.1-mini` reply with a server tool returned `stop`, so the rule has at least one more input than the model.
7. **Summing the three Anthropic counters.** The open question list says LiteLLM and agentgateway sum the three counters. The recordings show OpenRouter reports that same sum as `prompt_tokens` for Claude (WT23), so the fills and OpenRouter agree there. The Gemini recordings disagree with any single formula: applying the Anthropic subtraction to Gemini's fields gives a negative count (WT25).
8. **Pydantic AI's description of Gemini double billing.** Its test docstring says a marker on the final content part makes the Vertex endpoint bill "the cached prefix and then the same prefix again, doubling `prompt_tokens`". The cassette recorded with the corrected placement shows `prompt_tokens` not doubled but `cached_tokens` equal to `cache_write_tokens`. These are two observations of the same area, and only the second is in a recording.
9. **Number of rig fixtures.** The first wave counted 63 OpenRouter cassettes. That is the current tree. The folder held 251 files until 2026-10-05 (commit `daff77b9` deleted 188), and the deleted ones include the same-family switch cells, the truncation matrix, and the session matrix used here (WT1).
10. **Survey claims that hold on recorded traffic:** LC10 (both finish fields present, with the drift in WT44), LC12 (usage chunk with a non-empty `choices` array, WT36), and the first-wave item O78 on truncated tool calls (WT29). LC3 fits the `n: 2` case, where the extra candidate is dropped without an error (WT42).

## Open questions blocked by closed source

Each question I own, with its state after this pass.

| Question | State | What the recordings give | What was tried, and what stays open | Fill |
|---|---|---|---|---|
| OQ1 foreign `reasoning_details` after a switch | Still open | Clients that switch models strip reasoning first and get 200 (WT9, WT11). 34 same-endpoint replays of tagged values got 200 (WT7) | Scanned every request in eight sets for a foreign `format` or a tagged value sent to another endpoint: zero. OpenRouter's answer to that case is unrecorded | rig (reasoning to plain text, signature removed), Pydantic AI (thinking as tagged user text), Bifrost (refusal text table and one stripped retry, WT10) |
| OQ2 block order for interleaved thinking | Still open | Emit order and the accepted replay shape for a single thinking block (WT13) | All 67 recorded Claude thinking replies have one block. No debug echo with history was found | First-wave fills stand |
| OQ3 tying a Gemini signature to its part | Partly answered | A function-call signature is tied by `id` equal to the tool-call id, only the first of parallel calls has one, and text-part signatures have no tie field. Five shapes with dates (WT14). Unchanged replay gives 200 (WT15) | How OpenRouter places a no-`id` signature back upstream is not visible | First-wave fills stand |
| OQ4 purpose and scope of the `endpoint_slug` suffix | Partly answered | It names canonical model slug and endpoint slug (WT4); present on chat `signature` and `data`, on Responses `signature` and `encrypted_content` (WT5); start dates per vendor (WT3) | What OpenRouter does when the tagged endpoint differs from the routed one is unrecorded. The tag being the cause of same-endpoint routing is unproven (WT7) | none |
| OQ5 unsigned Claude reasoning: dropped or forwarded | Still open | Nothing: no client sends it (WT12) | Scanned all requests for a Claude entry without `signature` or a bare `reasoning` string | First-wave accounts stand in conflict |
| OQ6 server-side de-duplication of reasoning entries | Still open | Nothing: no request repeats an entry. Two different signed entries in one history were accepted (rig `history_survival_matrix`, WT7) | Scanned requests for repeated ids or signatures: zero | none |
| OQ7 DeepSeek and Kimi reasoning field and replay | Partly answered | The reply field is always `reasoning` plus a `reasoning.text` entry; `reasoning_content` never appears (WT20). 2025 models accepted tool continuations with reasoning left out | No request sends reasoning back to these models, and no recording is of DeepSeek V4 in a tool loop. The upstream field OpenRouter writes is unknown | req_llm parses leaked DeepSeek tool tokens (WT21); first-wave fills stand |
| OQ9 how OpenRouter knows which account produced a Claude block | Partly answered | The visible tag has model snapshot and endpoint and no account part: two unrelated recorders hold the identical tag value (WT4) | Whether account identity sits elsewhere (server-side or inside Anthropic's own bytes) is not visible | none |
| OQ11 OpenAI upstream protocol, `instructions`, `store`, `include` | Partly answered | Protocol per model from native words (WT16); `max_tokens` floor of 16 on Responses-served models (WT17); entry shapes and ids (WT18) | No debug echo for an OpenAI model was found in docs, issues, or web search (WT19). System-message placement, `store`, and `include` stay open | First-wave fills (LiteLLM bridge, Codex) stand |
| OQ15 `prompt_cache_key` to OpenAI, `x-grok-conv-id` for xAI | Still open | OpenAI-native `prompt_cache_options` and `prompt_cache_breakpoint` work through the Responses route (WT22) | No recording sets `prompt_cache_key` on chat; no echo found | req_llm sends `prompt_cache_key` as a top-level field (client side only) |
| OQ16 `cache_control` on a function tool or top level; more than four markers | Still open | A marker on a function tool for Claude is accepted; four markers are accepted (WT22) | Clients remove markers for non-Anthropic models and cap at four before sending, so the cases are not on the wire | Pydantic AI caps and strips client-side |
| OQ17 how `prompt_tokens` is built | Answered by recordings | Anthropic: sum of the three counters (WT23). OpenAI: cached and written inside the total (WT24). Gemini: cached and written overlap (WT25). Messages route: zero then real count (WT26) | `cache_discount` is in none of the recordings | none needed |
| OQ19 YAML tool emulation | Still open | A model without tool endpoints got 404, not emulation (2025-11-02). One reply has `<tool_call>` residue beside a parsed call (WT43) | No upstream body is visible without the debug echo | First-wave fills stand |
| OQ20 tool-call id rewriting across vendors | Partly answered | Formats per upstream (WT27). Ids crossed vendors and providers and were accepted (WT28) | Rewriting is invisible in a reply. No foreign id was sent to Mistral in any recording | First-wave fills stand |
| OQ23 tool call cut by the output limit | Answered by recordings for OpenAI | `tool_calls` beside native `length` or `max_output_tokens`, arguments cut mid-string, on both OpenAI protocols (WT29) | Not recorded for Anthropic or Gemini | none needed |
| OQ25 `reasoning.server_tool_call` and `rs_server_tool_round_*` | Partly answered | The round item decodes to a member list (WT31). On chat, server tools leave comments, final content, annotations, and usage counts (WT32, WT33) | Zero `reasoning.server_tool_call` entries in any recording, so its shape and whether it must be returned stay open. No replay of the round item is recorded | none |
| OQ30 `[DONE]` after an error, keep-alive interval, timeouts | Partly answered | `[DONE]` followed on 2025-11-03 (WT34). Comment text per route and positions (WT35). Chunk fields and usage chunk (WT36) | Fixtures store no arrival times, so the interval and any timeout stay open | none |
| OQ35 out-of-spec chat fields | Partly answered | `n: 2` gives one choice. `transforms`, `usage.include`, `preset` accepted (WT42) | `store`, `audio`, `prediction` are in no recording | none |

Not mine, but touched by evidence here: OQ29 (stop-reason mapping: derived `tool_calls` over native `stop`, `STOP`, `length`, and `MALFORMED_FUNCTION_CALL` with `finish_reason: "stop"`, WT30, WT44), OQ31 (overflow error body, WT39), OQ32 (response headers, WT41), OQ12 (several reasoning items on chat, WT13), OQ13 and OQ14 (OpenRouter item ids and vendor tool items on Responses, WT18, WT33).

## Unverified

- That the tag steers routing. The recordings show agreement between tag and serving endpoint, never a case that would tell cause from coincidence (WT7).
- That `completed` and `max_output_tokens` can only come from a Responses upstream. I read these as the Responses API's status and incomplete reason from the first-wave OpenAI file and from rig's direct OpenAI fixtures, and did not re-fetch OpenAI's documentation today.
- The interval of keep-alive comments and any stream timeout. No fixture stores timing.
- Whether req_llm's capture of the Gemini error stream ended before OpenRouter's body did (WT34). I read the stop rule in the stream server but did not trace the OpenRouter decoder that sets the terminal mark.
- Whether RubyLLM's recorder filters response headers. The census in WT41 assumes it stores what it received.
- Prism's two OpenRouter fixtures (ids dated 2025-10-15) and RubyLLM's batch, image, and decisions cassettes were listed but not analyzed.
- The documentation sample dates. The Gemini sample with the tag (WT6) and the debug echo sample (WT19) have no dates, so "today's documentation shows" is all that can be said.
- From memory, not sourced: that OpenAI's Chat Completions returns `stop` as the finish reason for a call forced by `tool_choice`. The recording in WT30 shows the native value `stop`, which fits, but I did not read a vendor page for it.

## Workspace documents used

- 2026-11-22, "OpenRouter field evidence - Rust and Python harnesses" (first-wave evidence file in the cabinet research folder). Took: the rig cassette inventory, items G37 (signature wrapping) and the rig test comments, and the first decoding of the suffix. Re-checked against the fixtures themselves; this file narrows its date window and adds the deleted cassettes.
- 2026-11-22, "OpenRouter routing and operations evidence" (first-wave). Took: item O78 on the truncated tool call and items O65 to O68 on the debug option. Re-read O78 independently from the deleted cassettes (agrees). Re-read the debug page in the mirror (agrees).
- 2026-11-22, "OpenRouter API surfaces and request superset" (first-wave). Took: item S31, the decoded suffixes in OpenRouter's documentation samples, including the Messages sample. Re-checked the provider selection and image understanding samples in the mirror; the Messages sample is clipped in the mirror and was not re-read.
- 2026-11-22, "OpenRouter vendor evidence - OpenAI and Responses" (first-wave). Took: item OA10 and its open question about non-reasoning models. This file supplies recordings for it.
- 2026-11-22, the lead agent's list of open questions for the second wave (scratch). Took: the question numbers and the known fills.
- The 2026-10-28 survey of proprietary inference APIs in `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, by way of the claims LC3, LC10, LC11, LC12, LC13 listed in the shared brief. Not re-read directly.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| WT-s1 | https://github.com/0xPlaygrounds/rig at `a8673f775ca78004f011e5ad5b615147fdcd4bff` (2026-11-22): `crates/rig-cassette/fixtures/cassettes/openrouter/` (63 files), `crates/rig-cassette/fixtures/bank/openrouter.yaml`, `crates/rig-cassette/fixtures/cassettes/anthropic/streaming/gateway_message_delta_metadata.yaml`, direct OpenAI and Anthropic fixtures for comparison, `crates/rig-cassette/src/scrub.rs`, `crates/rig-cassette/tests/providers/openrouter/cassette/*.rs`, `crates/rig-core/CHANGELOG.md` | 2026-11-22 | harness source (third-party recordings of OpenRouter traffic) |
| WT-s2 | Same repository at `a023bd5b40f7cf6371d01ad18454b1bb01be75e5` (2026-10-04): `crates/rig-cassette/fixtures/cassettes/openrouter/` (251 files, 188 of them deleted by `daff77b9` on 2026-10-05) | 2026-11-22 | harness source (third-party recordings) |
| WT-s3 | https://github.com/pydantic/pydantic-ai at `463a6d519b7e4c8e038d5a2559b0d26281df6937` (2026-11-22): `tests/models/cassettes/test_openrouter/`, `tests/models/openrouter/cassettes/`, `tests/models/cassettes/test_model_settings_delta/`, `tests/models/cassettes/test_openai_prompt_cache/`, `tests/models/openrouter/test_cache.py` | 2026-11-22 | harness source (third-party recordings) |
| WT-s4 | https://github.com/anomalyco/opencode at `71cd8c064c8b93c31604c4c4074adb8b45790fc1` (2026-11-22): `packages/ai/test/fixtures/recordings/openrouter-*.json` and `openrouter-responses/` | 2026-11-22 | harness source (third-party recordings) |
| WT-s5 | https://github.com/Kilo-Org/kilocode at `8045e72da2cf057e42ed70820891d5ca11aa3f4b` (2026-11-22): `packages/llm/test/fixtures/recordings/openai-compatible-chat/openrouter-*.json` | 2026-11-22 | harness source (third-party recordings) |
| WT-s6 | https://github.com/agentjido/req_llm at `0e49ee99737180444f6fc779f3cccd799f8344a5` (2026-11-21): `test/support/fixtures/openrouter/` (911 files), `lib/req_llm/providers/openrouter.ex`, `lib/req_llm/stream_server.ex`, `lib/req_llm/streaming/decoder.ex` | 2026-11-22 | harness source (third-party recordings) |
| WT-s7 | https://github.com/crmne/ruby_llm at `2af24ff4b7e1e41d495e74cd3644bdf2c2b0d602` (2026-11-18): `spec/fixtures/vcr_cassettes/*openrouter*` (78 cassettes) | 2026-11-22 | harness source (third-party recordings) |
| WT-s8 | https://github.com/simonw/llm-openrouter at `6acf8eeab370a3b196f61d27d1382b3f73ae4507` (2026-11-04): `tests/cassettes/test_llm_openrouter/` | 2026-11-22 | harness source (third-party recordings) |
| WT-s9 | https://github.com/maximhq/bifrost at `cae639c53141785bf0f21c32c135941c86e1026a` (2026-11-22): `core/encryptedreasoning_test.go` lines 100, 1103-1143 | 2026-11-22 | gateway source |
| WT-s10 | https://openrouter.ai/docs/api_reference/errors-and-debugging | 2026-11-22 | OpenRouter doc |
| WT-s11 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| WT-s12 | https://openrouter.ai/docs/guides/overview/multimodal/image-understanding and https://openrouter.ai/docs/guides/features/service-tiers | 2026-11-22 | OpenRouter doc |
| WT-s13 | https://openrouter.ai/api/v1/models (catalog: `canonical_slug`, `pricing`) | 2026-11-22 | OpenRouter spec (catalog) |
| WT-s14 | https://github.com/SillyTavern/SillyTavern/pull/5617 (opened 2026-05-09, last update 2026-09-28) | 2026-11-22 | issue report (third-party) |
| WT-s15 | First-wave evidence files of 2026-11-22: Rust and Python harnesses (G37), routing and operations (O42, O65 to O68, O78), API surfaces (S31), OpenAI and Responses (OA10) | 2026-11-22 | workspace document |
| WT-s16 | https://github.com/prism-php/prism at `385022b5e0fec9be49a5b8e4a468fdafaa8f1267`: `tests/Fixtures/openrouter/` (listed, not analyzed) | 2026-11-22 | harness source |
| WT-s17 | https://github.com/langchain-ai/langchain at `8ff3cd30eebfe37712a91f11e3f4b61a545e7a37`: `libs/partners/openrouter/` (checked: tests only, zero cassettes) | 2026-11-22 | harness source |

