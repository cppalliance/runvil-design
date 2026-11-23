---
produced: 2026-11-22
title: OpenRouter deep dive - other vendors' hard rules (xAI, DeepSeek, Mistral, Moonshot Kimi, Alibaba Qwen, Zhipu GLM Z.ai, MiniMax, Meta) and local servers (llama.cpp, vLLM, Ollama) - native rule, OpenRouter handling, reasoning_content replay, tool-call id rules, tool_choice limits, thinking switches, cache fields, built-in search
---

# OpenRouter deep dive: the other vendors' hard rules and the local servers behind the Runvil Gateway

- What was read: the vendors' live documentation (xAI, DeepSeek, Mistral, Moonshot Kimi, Alibaba Model Studio, Z.ai, MiniMax, Meta), OpenRouter's documentation mirror, OpenAPI spec, model catalog, and the public per-model endpoint lists for 22 models, plus source code at pinned commits for llama.cpp, vLLM (docs), Ollama, LiteLLM, pi, and opencode, and the two workspace reports named in the task.
- How many sources: 94 (16 OpenRouter, 43 vendor pages, 13 local-server files, 10 translator or harness files, 9 third-party reports, 3 workspace documents).
- Read date: 2026-11-22 for every source. Workspace documents are dated 2026-09-20 and 2026-10-28 and every rule taken from them was re-checked live unless the row says otherwise.

Reading notes for this file:

- OpenRouter's documentation calls xAI "SpaceXSI" in several places. Quotes keep the wording as written.
- Command-line flags of llama.cpp and vLLM are written without their two leading dashes (for example `reasoning-format`).
- "Direct" in the last matrix column means: does the rule also bite when Runvil's Gateway forwards a chat-completions request as written to that vendor's own OpenAI-compatible endpoint, which is what Runvil does today.
- OpenRouter reaches these families two ways. "First-party" means the vendor's own endpoint listed as an OpenRouter provider. "Third-party" means open weights served by another host (Together, Fireworks, DeepInfra, Groq, Novita, Alibaba, and so on). The endpoint lists fetched today show which is which per model (OV8).

## Candidate matrix rows

Basis column: the first label is for the native rule, the second for the OpenRouter column. "Fill" names the project whose handling was read.

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Responses API as the primary route | xAI | Docs: "The Responses API is the recommended way to interact with SpaceXSI models." Chat Completions is labeled "(Deprecated)" and "Function calling only". | Grok is callable on all four OpenRouter routes. `reasoning_details[].format` has the value `xai-responses-v1`, which points to OpenRouter calling xAI's Responses route upstream. | xAI's server-held conversations (`previous_response_id`, 30-day storage) are unavailable: OpenRouter's Responses route is stateless (survey claim LC4, checked by another subagent). | Gateway picks the upstream route. A chat-completions caller can stay on chat completions if the Gateway translates to xAI Responses. Direct: chat completions still works at xAI but returns no ciphertext. | Vendor-documented + OR-inferred | OV13, OV20 |
| Encrypted reasoning items | xAI | Responses only: `include: ["reasoning.encrypted_content"]`. `grok-4.7` returns `encrypted_content` on every response. "Chat Completions is unaffected; it has no field for the ciphertext." | Documented shape: `reasoning_details` entries of type `reasoning.encrypted`, and the `format` enum includes `xai-responses-v1`. Caller must send the array back unmodified and in order. OpenRouter "automatically requests encrypted reasoning content from reasoning-capable providers". | Nothing lost if the array is replayed. A caller that drops `reasoning_details` loses reasoning continuity silently. | Caller holds state. Opaque data must make the round trip unchanged, tagged by format. Direct: xAI's own chat completions route cannot return or accept it, so direct forwarding gets no reasoning continuity. | Vendor-documented + OR-documented | OV14, OV2, OV20 |
| `reasoning.effort` values and models with reasoning always on | xAI | `grok-4.7`, `grok-4.6`, `grok-4.5`: `low`, `medium`, `high` (default), `xhigh` (4.6 and later; treated as `high` on 4.5). "Reasoning cannot be disabled." `stop`, presence and frequency penalties return an error on reasoning models. | `reasoning.effort` unified enum (`max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`). Catalog: `x-ai/grok-4.7` has `"mandatory": true`, `supported_efforts` `xhigh, high, medium, low`. Unsupported efforts map "to the nearest supported level". | The catalog publishes which efforts exist, so little is lost. `none` on a mandatory model is rejected upstream ("the model rejects it"). | Neutral setting (off, low, high) is translatable in the Gateway from a per-model table. A thinking-off request cannot be honored and needs a defined fallback. Direct: yes, same limits. | Vendor-documented + OR-documented | OV15, OV3, OV4 |
| Server-side search tools | xAI | `web_search` and `x_search` are built-in tools on the Responses API. Chat Completions tools: "Currently, only functions are supported as a tool." | `openrouter:web_search` server tool with `engine: "native"`; "On SpaceXSI models, native web search sends only xAI's `web_search` tool." X search needs an `x_search` object. Endpoint field `native_tools` shows `{"openrouter:web_search": {"type": "web_search"}}` on xAI endpoints. | X search filters are a vendor-specific object (`allowed_x_handles` and so on). Per-item X search billing is why it became opt-in. | The feature exists only on xAI's Responses route. Gateway must translate to Responses to reach it. Vendor-specific values for X search. Direct chat completions: not available. | Vendor-documented + OR-documented | OV17, OV10 |
| Cached token reporting and cache routing hint | xAI | Caching is automatic. Hint: `x-grok-conv-id` header on chat completions, `prompt_cache_key` body field on both routes ("Plumbed to `x-grok-conv-id`"). Usage: `prompt_tokens_details.cached_tokens` or `input_tokens_details.cached_tokens`. | "Prompt caching with Grok is automated". OpenRouter keeps a conversation on one provider (sticky routing) keyed by a hash of the opening messages, `session_id`, or `prompt_cache_key`. Reads reported in `prompt_tokens_details.cached_tokens`. | No documented statement that OpenRouter sends `x-grok-conv-id` to xAI (it documents a session affinity key only for Z.AI). | Gateway holds the hint: it can derive a stable conversation id and set the header. No caller change. Direct: yes, the header is plain HTTP and a forwarder can add it. | Vendor-documented + OR-documented | OV18, OV9 |
| Tool count cap | xAI | Responses: "A max of 1000 tools are supported." Chat: "A max of 1000 functions are supported." | No statement found in the mirror or spec. | A request over the cap fails at xAI with xAI's error. | Gateway-only check. Direct: yes. | Vendor-documented + OR-documented (silent) | OV16 |
| Unsupported and ignored Responses fields | xAI | `background` "(Unsupported)", `context_management` "Parsed but not yet executed", `metadata` and `truncation` "Not supported", logprobs "silently ignored" on `grok-4.20` and newer. | OpenRouter's general rule: unsupported parameters are ignored and the rest forwarded. | Caller cannot tell an ignored field from an honored one. | Gateway-only. Direct: yes. | Vendor-documented + OR-documented | OV19, OV5 |
| `reasoning_content` must be passed back when tools are present | DeepSeek | With `tools` in the request "the reasoning_content of all previous turns should be passed back to the API"; otherwise "the API will return a 400 error". Without `tools` it is ignored. Error text seen by clients: "The `reasoning_content` in the thinking mode must be passed back to the API." | Caller may send reasoning back as `reasoning`, `reasoning_details`, or `reasoning_content` ("an alias - it functions identically to `reasoning`"). OpenRouter does not document that it fills the field when the caller dropped it. Third-party reports from April and May 2026 show the DeepSeek 400 reaching callers through OpenRouter, and one maintainer reports a later OpenRouter-side change. | A caller that drops reasoning gets a provider 400 on some hosts and silent success on others, depending on which provider served the turn. | Caller (or Gateway) holds state. Plain-text reasoning must make the round trip on every assistant message of a tool conversation. Vendor-neutral if the caller always returns what it got. Direct: yes, this is native to DeepSeek's own chat completions dialect and Runvil's history layer drops reasoning today. | Vendor-documented + OR-inferred | OV22, OV23, OV1, OV28, OV29 |
| `tool_choice` limits in thinking mode | DeepSeek | "required and named tool choices are not supported in thinking mode; the API returns a 400 error." | Per-endpoint `supports_tool_choice` object in the endpoints API, for example `{"none": true, "auto": true, "required": false, "function": false}` on several `deepseek/deepseek-v4-pro` endpoints. `require_parameters` controls whether unsupporting providers are skipped. | The value differs per provider for one model slug, so the caller sees provider-dependent behavior unless it reads the endpoint list. | Gateway can hold a capability table and reject or downgrade. Caller does not need vendor values. Direct: yes. | Vendor-documented + OR-documented | OV24, OV7 |
| How thinking is switched | DeepSeek | `thinking: {"type": "enabled" or "disabled"}` and `reasoning_effort` (`none`, `low`, `high`, `max`; `minimal` maps to `low`, `medium` and `xhigh` to `high`). Default on at `high`. `temperature` and the penalties are ignored in thinking mode. | `reasoning.effort` and `reasoning.enabled`. Catalog for `deepseek/deepseek-v4-pro`: `supported_efforts` `xhigh, high, none`; for `deepseek-v4.1-flash`: `max, high, low, none`. | The OpenRouter effort lists differ from DeepSeek's own list (`none, low, high, max`). | Neutral on/off plus effort translates in the Gateway. Direct: Runvil's on/off switch needs a per-vendor field (`thinking.type`). | Vendor-documented + OR-documented | OV21, OV4 |
| Cache hit and miss usage fields | DeepSeek | `usage.prompt_cache_hit_tokens`, `usage.prompt_cache_miss_tokens`, and `prompt_tokens_details.cached_tokens` ("Same as prompt_cache_hit_tokens"). Caching on by default. | "Prompt caching with DeepSeek is automated". Reads normalized to `prompt_tokens_details.cached_tokens`. | The miss counter is dropped (it equals prompt tokens minus hits). | Gateway-only normalization. Direct: both spellings arrive, a client reading the OpenAI spelling works. | Vendor-documented + OR-documented | OV25, OV9 |
| Prefix completion | DeepSeek | Beta: last message has role `assistant` with `prefix: true`, base URL `https://api.deepseek.com/beta`. | No statement found in the mirror for DeepSeek's `prefix` flag. | The feature needs a vendor-specific message flag and a different base URL. | Caller would have to send a vendor-specific flag, or the Gateway maps a neutral "continue this assistant text" intent. Direct: yes but on the beta base URL. | Vendor-documented + OR-documented (silent) | OV26 |
| Extra finish reasons | DeepSeek | `finish_reason` values include `insufficient_system_resource` and `aborted`. | `finish_reason` is normalized and `native_finish_reason` holds the vendor value (survey claim LC10, checked by another subagent). | None if the caller reads `native_finish_reason`. | Gateway-only mapping. Direct: the raw value reaches Runvil's client today. | Vendor-documented + OR-documented | OV27 |
| Tool-call ids of exactly nine alphanumerics | Mistral | Docs: ids "are randomly generated strings of exactly 9 characters". API error seen by clients: "Tool call id was call_5019f900a247472bacde0b82 but must be a-z, A-Z, 0-9, with a length of 9." with `"type":"invalid_function_call"` and `"code":"3280"`. | No statement found in the mirror or spec (`ChatToolCall.id` is a plain string). Third-party reports show the 3280 error reaching callers through OpenRouter in December 2025 and March 2026; an OpenRouter employee wrote on 2026-03-31 "we are fixing this on our end". Current behavior is unconfirmed. | If OpenRouter rewrites ids, the mapping is invisible to the caller. If it does not, the caller must know the model family. | Gateway can remint ids per request and map results back; nobody needs to hold state between requests if the mapping is deterministic (hash). Direct: yes, Mistral's own API enforces it. | Vendor-documented + OR-inferred; Fill: opencode, pi, Roo Code, vLLM | OV30, OV37, OV38 |
| Reasoning as content chunk lists (`ThinkChunk`) | Mistral | With reasoning on, `message.content` is a list of `ThinkChunk` (`type: "thinking"`) and `TextChunk`; in streaming `delta.content` is a list during thinking and a plain string during the answer. Replay the full assistant message "including `ThinkChunk`". Schema has an optional `signature` ("Signature to replay some reasoning blocks across turns."). | OpenRouter returns reasoning in `reasoning` and `reasoning_details`, and `content` stays a string. No Mistral value exists in the `format` enum, so entries would be `unknown`. | Whether a `ThinkChunk.signature` survives OpenRouter is undocumented. | Caller holds state and returns it. A translator must turn chunk lists into a reasoning field and back. Direct: yes on Mistral's native `/v1` route, where a client that expects string content breaks. | Vendor-documented + OR-inferred | OV31, OV2, OV37 |
| `reasoning_effort` per model | Mistral | Mistral Large 4: `low`, `medium`, `high` only ("does not accept `none`, `minimal`, `xhigh`, or `max`"). Small 4 and Medium 3.5: `high` or `none`. | Catalog for `mistralai/mistral-large-4-0`: `supported_efforts` `high, none`, `mandatory: false`. | Conflict with Mistral's page for Large 4 (see Conflicts). | Neutral effort translatable from a per-model table. Direct: yes. | Vendor-documented + OR-documented | OV32, OV4 |
| `prompt_mode` and `safe_prompt` | Mistral | `prompt_mode` enum `reasoning` ("Allows toggling between the reasoning mode and no system prompt"). `safe_prompt` boolean, now marked deprecated in favor of custom guardrails. | "OpenRouter will also transmit some provider-specific parameters, such as `safe_prompt` for Mistral". `prompt_mode` is not mentioned. `provider.options.mistral` exists as a passthrough object. | Vendor-specific names stay vendor-specific. | Caller must send vendor-named fields, or the feature stays unused. Not needed for a model-role prompt. Direct: yes. | Vendor-documented + OR-documented | OV33, OV5, OV6 |
| Error envelope | Mistral | `{"object":"error","message","type","param","code","raw_status_code"}`; 401 is `{"detail": "Invalid API Key"}`; 422 is `{"detail": [...]}`; the OpenAI-compatible endpoint "wraps the error in an `error` object". | OpenRouter wraps provider errors in its own envelope; third-party reports show the raw Mistral JSON inside `metadata.raw`. | The four-digit `code` survives only inside the raw string. | Gateway-only normalization. Direct: yes on `/v1`. | Vendor-documented + OR-inferred | OV34 |
| Two chat-completions endpoints | Mistral | `https://api.mistral.ai/v1/chat/completions` "uses the native Mistral format"; `https://api.mistral.ai/openai/v1/chat/completions` is the OpenAI-compatible endpoint, in public preview. | Not applicable (OpenRouter picks the upstream). | Not applicable. | Gateway configuration. Runvil's provider file records `https://api.mistral.ai/v1` as the OpenAI base URL, which Mistral now describes as the native-format route. Direct: yes. | Vendor-documented + Workspace | OV35, OV80 |
| `reasoning_content` replay on K-series | Moonshot Kimi | `kimi-k3.1-pro` and `kimi-k3`: reasoning and Preserved Thinking always on; "pass the complete assistant message returned by the API back to `messages` as-is (including `reasoning_content`)". `kimi-k2.6`: `thinking.keep: "all"` opts in. | Same three replay fields as above. Catalog: `moonshotai/kimi-k2.7-code` `mandatory: true`; `moonshotai/kimi-k3` `mandatory: false` with `none` in `supported_efforts`. | Kimi's hosted API also streams preserved thinking as `summary` and `encrypted` entries in `reasoning_details`, per opencode's source, which says "Kimi does not document this publicly". | Caller holds state and returns it unchanged. Direct: yes. | Vendor-documented + OR-documented; Fill: opencode, LiteLLM | OV40, OV4, OV45 |
| Thinking control and fixed sampling | Moonshot Kimi | K3 series: `thinking` "Not supported", `reasoning_effort` `low`, `high`, `max` (default `max`). `temperature` fixed at `1.0`, `top_p` at `0.95`; "passing any other value returns an error". | `reasoning.effort`. Sampling parameters the endpoint does not list are ignored, per the general rule. | Catalog says `kimi-k3` reasoning can be turned off, vendor says always on (see Conflicts). | Gateway must strip or fix `temperature` for these models, since Runvil prompts can set temperature. Direct: yes, a forwarded `temperature` returns an error. | Vendor-documented + OR-documented | OV39, OV4, OV5 |
| Tool-call ids | Moonshot Kimi | Hosted API today: example id `get_weather_0_4d9e2559`; rule "The `tool_call_id` must match the tool-call `id` in the assistant message". Open-weight Kimi K2 guide: "The format of the tool ID is `functions.{func_name}:{idx}`". | No statement found. | For third-party hosts of open weights, the id format the template expects depends on the host's server. | Gateway should return the ids it received, unchanged. A mid-conversation model switch needs ids reminted in the target format. Direct: return ids unchanged. | Vendor-documented + OR-documented (silent) | OV41 |
| Built-in tools | Moonshot Kimi | Official tools (`web-search`, `date`, `fetch`, and others) are fetched from `GET /v1/formulas/{uri}/tools` and executed by the caller through `POST /v1/formulas/{uri}/fibers`. The Responses API offers "server-side web search". `$web_search` does not appear in today's docs. | Moonshot is absent from OpenRouter's list of native search providers. `openrouter:web_search` falls back to Exa for models without native search. | Kimi's own search engine is not reachable through OpenRouter. | Vendor-specific second route (Formula API) run by the caller or the Gateway. Direct: reachable, with extra HTTP calls. | Vendor-documented + OR-documented | OV42, OV10 |
| Partial mode | Moonshot Kimi | Last message `role: "assistant"` with `partial: true`; with thinking on, the original `reasoning_content` must be included. | No statement found. | Same shape of gap as DeepSeek `prefix`. | Vendor-specific message flag (`partial` here, `prefix` at DeepSeek). Direct: yes. | Vendor-documented + OR-documented (silent) | OV43 |
| Cache options and usage | Moonshot Kimi | `prompt_cache_options: {"mode": "implicit", "ttl": "1h"}`; usage `cached_tokens` and `cache_write_tokens`; a `prompt_cache_breakpoint` in content is "rejected (HTTP 400)". | "Prompt caching with Moonshot AI is automated". `cache_write_tokens` is OpenRouter's name for writes (survey claim LC17). | The TTL choice (5 minutes or 1 hour) is not documented as reachable through OpenRouter. | Gateway-only. Note the 400 on explicit breakpoints: a translator must strip OpenAI-style breakpoints for Kimi. Direct: yes. | Vendor-documented + OR-documented | OV44, OV9 |
| `enable_thinking` and thinking-only models | Alibaba Qwen | `enable_thinking` boolean at the top level of the body ("not an OpenAI standard parameter"); some models are thinking-only. Some open-source versions "only support Stream Output, and non-streaming calling will report an error". | `reasoning.enabled` and `reasoning.effort`. Catalog `reasoning` object per model (for example `qwen/qwen3.8-flash`: `default_enabled: true`, `supports_max_tokens: true`). | The streaming-only limit is not surfaced in the catalog fields read today. | Neutral on/off translatable in the Gateway. Runvil's client always streams, so the streaming-only limit does not bite. Direct: yes, needs the vendor field name. | Vendor-documented + OR-documented | OV46, OV4 |
| `thinking_budget` | Alibaba Qwen | Integer token cap for reasoning. On `qwen4-max` it cannot be combined with `reasoning_effort`, and a budget maps to a level (1 to 4096 `low`, 4097 to 131072 `high`, above that `xhigh`). | `reasoning.max_tokens`: "Some Alibaba Qwen thinking models (mapped to `thinking_budget`)". | "support varies by model", so the caller must check the model description. | Neutral budget translatable. Field name differs per server (`thinking_budget` here, `thinking_token_budget` on vLLM, `thinking_budget_tokens` on llama.cpp per pi). Direct: vendor field name needed. | Vendor-documented + OR-documented; Fill: pi | OV47, OV3, OV78 |
| `preserve_thinking` | Alibaba Qwen | Boolean, default `false`, default `true` on `qwen4-max`, `qwen3.8-max`, `qwen3.8-flash`, where "you must pass back all reasoning_content from the history conversation in full". Folding reasoning into `content` "is not supported". | No statement found for `preserve_thinking`. Generic replay fields apply. | A caller that drops reasoning loses quality silently on models where the default is `true`. | Caller holds state and returns it. A per-model default lives in the Gateway. Direct: yes. | Vendor-documented + OR-documented (silent) | OV48, OV1 |
| Summarized thinking with ciphertext | Alibaba Qwen (`qwen4-max`) | `thinking_display: "summarized"` returns a summary plus `encrypted_content`. "The client can only pass the ciphertext back as-is; it cannot read, truncate, re-encode, modify, or concatenate ciphertext from multiple turns." | No statement found. The `format` enum lists other vendors only, so an Alibaba entry would be tagged `unknown`. | Unknown whether the ciphertext survives OpenRouter. | Caller holds opaque state that must make the round trip unchanged, per turn. This is the same shape as signed or encrypted reasoning elsewhere. Direct: yes, as an extra field on a chat-completions message. | Vendor-documented + OR-documented (silent) | OV49, OV2 |
| `tool_choice` limits | Alibaba Qwen | "Qwen series models do not currently support required"; "Models in thinking mode do not support forcing a specific tool to be called." | `supports_tool_choice` per endpoint: Alibaba endpoints for `qwen/qwen3.6-27b` and `qwen/qwen3.8-flash` show `none` and `auto` only; `qwen/qwen4-max` shows all four. | Same as DeepSeek: provider-dependent for one slug. | Gateway capability table. Direct: yes. | Vendor-documented + OR-documented | OV50, OV7 |
| Explicit cache markers | Alibaba Qwen | `"cache_control": {"type": "ephemeral"}` on content blocks, 5-minute life, 1,024-token minimum, writes at 125 percent and hits at 10 percent; implicit cache at 20 percent; explicit and implicit are "mutually exclusive". Usage `cache_creation_input_tokens` and `cached_tokens` under `prompt_tokens_details`. | "Alibaba prompt caching requires explicit cache breakpoints. Add `cache_control: { "type": "ephemeral" }` to content blocks you want to cache, using the same syntax as Anthropic explicit caching." Listed for six model slugs. | Model coverage is a short list; snapshot endpoints "do not support explicit caching". | Caller or Gateway places markers. A Gateway can place them from a neutral policy. Direct: yes, same marker syntax on the compatible-mode route. | Vendor-documented + OR-documented | OV51, OV9 |
| Built-in search | Alibaba Qwen | Chat completions: `enable_search: true` (non-standard top-level field) with `search_options`. Responses API: a `web_search` tool. | Alibaba is absent from OpenRouter's list of native search providers. | Alibaba's own search is not reachable through OpenRouter. | Vendor-specific request field. Direct: yes. | Vendor-documented + OR-documented | OV52, OV10 |
| The `thinking` object and `clear_thinking` | Zhipu GLM (Z.ai) | `thinking: {"type": "enabled" or "disabled", "clear_thinking": true}`. `clear_thinking` default `true` drops `reasoning_content` from prior turns. With `false`, "you must forward the full, unmodified, and correctly ordered historical `reasoning_content` in `messages`". GLM-5.3 and GLM-5.3-FLASH "use forced thinking and cannot be disabled". | Catalog for `z-ai/glm-5.3`: `mandatory: true`, efforts `max, high, low`. No statement found for `clear_thinking`. | Preserved thinking (the `false` setting) is not documented as reachable. | Caller holds state and returns it; the Gateway decides whether to set `clear_thinking: false`. Direct: yes. | Vendor-documented + OR-documented | OV54, OV55, OV4 |
| `tool_choice` limited to `auto` | Zhipu GLM (Z.ai) | "The default value is auto, and only auto is supported." | `supports_tool_choice` on the Z.AI endpoints for `z-ai/glm-5.3` and `z-ai/glm-4.7`: `auto` only. Third-party hosts of the same weights show more. | Provider-dependent for one slug. | Gateway capability table. Direct: yes. | Vendor-documented + OR-documented | OV56, OV7 |
| Built-in web search | Zhipu GLM (Z.ai) | `tools` entries of type `web_search` (and `retrieval`) beside `function`; results return in a top-level `web_search` array. | Z.AI is absent from OpenRouter's list of native search providers. | Not reachable through OpenRouter. | Vendor-specific tool type and response field. Direct: yes. | Vendor-documented + OR-documented | OV56, OV10 |
| Numeric error codes | Zhipu GLM (Z.ai) | `{"error":{"code":"1214","message":...}}` with business codes (1113 balance, 1261 "Prompt too long", 1302 rate limit, 1305 overload). In streaming, an abnormal end shows in `finish_reason` (`sensitive`, `network_error`, `model_context_window_exceeded`). | OpenRouter's own error envelope and `native_finish_reason`. | The numeric code survives only in raw metadata, if at all. | Gateway-only mapping. Direct: yes. | Vendor-documented + OR-inferred | OV57 |
| Cache and session affinity | Zhipu GLM (Z.ai) | Implicit caching, reads in `usage.prompt_tokens_details.cached_tokens`. | "OpenRouter sends Z.AI a session affinity key with each request, derived from your account and, when provided, your `session_id`". | None documented. | Gateway holds the hint. Direct: Z.ai's own docs read today do not name an affinity field. | Vendor-documented + OR-documented | OV58, OV9 |
| `reasoning_split` and thinking control | MiniMax | `reasoning_split: true` puts thinking in `reasoning_content`; `false` leaves it in `content` inside `<think>` tags (M3 only). `thinking: {"type": "adaptive" or "disabled"}`; M3.1-Flash returns 400 for `disabled` with the text `... is not allowed (2013)`. `temperature` and `top_p` "will be ignored". | The Messages page says a model "such as GPT-5 or MiniMax receives `reasoning_effort: "high"` in its own vocabulary". Catalog: `minimax/minimax-m2.5` `mandatory: true`. None of the five MiniMax endpoint lists fetched today has MiniMax as a provider; they list third-party hosts (and Groq alone for `minimax-m2.7`). | Third-party hosts decide the reasoning format; `reasoning_split` belongs to MiniMax's own API. | Gateway sets `reasoning_split: true` so the client always sees a separate field. Caller returns the full message. Direct: yes. | Vendor-documented + OR-documented | OV60, OV61, OV63 |
| Error envelope with `base_resp` | MiniMax | The OpenAI-compatible response schema includes `base_resp` with `status_code` (1000, 1001, 1002, 1004, 1008, 1013, 1027, 1039, 2013) and `status_msg`. The workspace claim that the native route answers HTTP 200 with a nonzero `status_code` was not re-confirmed today. | Not applicable for third-party hosts. | Not applicable. | Gateway must read `base_resp.status_code` on a 200 before trusting the body. Direct: yes. | Vendor-documented + Workspace | OV62 |
| `tool_choice` limited to `auto` | Meta | "Only `"auto"` (the default) is supported on both Chat Completions and the" Responses API; other values return HTTP 400 with "only `"auto"` is supported for `tool_choice`". | The endpoint list for `meta/muse-spark-1.3` shows the Meta provider with `supports_tool_choice` all four true. This conflicts with Meta's page. | If the catalog is wrong, a `required` request fails at Meta with a 400. | Gateway capability table; a forced-tool request needs a fallback (prompt instruction or reject). Direct: yes. | Vendor-documented + OR-documented (conflict) | OV64, OV66 |
| Reasoning replay only off the chat route | Meta | Reasoning "stays private". Chat Completions: "No (reasoning is not carried between turns)". Responses: `include: ["reasoning.encrypted_content"]` and replay as a `reasoning` input item. `reasoning_effort: "none"` returns HTTP 400. | `reasoning_details` format values `meta-responses-v1` and `google-vertex-meta-responses-v1`. Catalog: `mandatory: true`, efforts `max` to `minimal`. | Nothing if the array is replayed. | Same as xAI: opaque data round trip through the caller; reasoning continuity exists only if the Gateway speaks Responses (or Messages) upstream. Direct chat completions: no continuity. | Vendor-documented + OR-documented | OV65, OV2, OV66 |
| Reasoning field name and format | llama.cpp | Returns `message.reasoning_content`. Server flag `reasoning-format` (`none`, `deepseek`, `deepseek-legacy`), request field `reasoning_format`, `reasoning_effort` ("If `none`, reasoning/thinking is disabled"), `chat_template_kwargs` such as `{"enable_thinking": false}`. Reasoning in history is forwarded to templates that support it. | OpenRouter's field is `reasoning` (with `reasoning_content` accepted as an input alias). | Not applicable. | Gateway normalizes the field name. Caller returns reasoning. Applies to Runvil's local path today. | Filled: llama.cpp | OV67 |
| Capability discovery through `/props` | llama.cpp | `GET /props` returns `chat_template_caps`, including `reasoning_effort_levels` and `reasoning_effort_rejected` ("answered with a 400"). Caps struct fields include `supports_tools`, `supports_tool_calls`, `supports_parallel_tool_calls`, `supports_preserve_reasoning`. | OpenRouter's equivalent is the catalog `reasoning` object and per-endpoint `supported_parameters` and `supports_tool_choice`. | Not applicable. | Gateway reads capabilities per loaded model. No caller knowledge. | Filled: llama.cpp | OV68 |
| Timings, prompt cache, slots | llama.cpp | Response `timings` object (`cache_n`, `prompt_n`, `predicted_n`, per-second rates) beside standard `usage` with `prompt_tokens_details.cached_tokens`. `cache_prompt` (default `true`), `id_slot`, slot save, restore, erase routes. | OpenRouter reports `cached_tokens` and its own cost fields. | Not applicable. | The server holds cache state per slot. Gateway can pin a conversation to a slot. | Filled: llama.cpp | OV69 |
| Grammars | llama.cpp | `grammar`, `json_schema`, and `response_format` (three accepted shapes). With tools: error "Cannot use custom grammar constraints with tools." | OpenRouter exposes `response_format` and `structured_outputs`. | Not applicable. | Gateway must not forward a client grammar together with tools. | Filled: llama.cpp | OV70 |
| `/v1/messages` and `/v1/responses` routes | llama.cpp | Both exist. Messages replies put reasoning in a `thinking` block with `"signature", ""` ("empty signature for local models"). Responses replies put it in a reasoning item with `"encrypted_content", ""`. Both take `extra_params` for llama.cpp-only settings. | Not applicable. | Not applicable. | A local model has plain reasoning text and an empty signature, so the opaque block list degenerates to text. | Filled: llama.cpp | OV71 |
| Silent failures | llama.cpp | A named-function `tool_choice` object is read as a string, fails the type check, logs "Wrong type supplied for parameter", and becomes `auto`. A template that supports tool calls without natively describing tools only logs a warning. Other Appendix C items were not re-read today. | Not applicable. | Not applicable. | Gateway must translate or reject the named form itself, and check template capabilities before sending tools. | Filled: llama.cpp + Workspace | OV72 |
| Reasoning parsers and field name | vLLM | Server flag `reasoning-parser` per family. Field is `reasoning`: "`reasoning` used to be called `reasoning_content`." Controls: `chat_template_kwargs`, `reasoning_effort`, `thinking_token_budget`. | OpenRouter lists `Inferact vLLM` as a provider on one model and `inferact-vllm` in `provider.options`. | Not applicable. | Gateway must read both `reasoning` and `reasoning_content`, and write back the name the server expects. | Filled: vLLM | OV73, OV79 |
| Tool-call parsers and `tool_choice` | vLLM | Needs server flags `enable-auto-tool-choice` and `tool-call-parser`. Named and `required` use structured outputs; "Without a loaded parser, a request with `tool_choice='required'` returns HTTP 400." Mistral templates need "tool call IDs that are exactly 9 digits". | Not applicable. | Not applicable. | Server operator holds the parser choice. Gateway reads errors. | Filled: vLLM | OV74 |
| Structured output controls | vLLM | `structured_outputs` object (`json`, `regex`, `choice`, `grammar`, `structural_tag`); the old `guided_*` fields "were removed in v0.12.0". | Not applicable. | Not applicable. | Vendor-specific field for anything beyond `response_format`. | Filled: vLLM | OV75 |
| Responses and Messages routes | vLLM | `/v1/responses` (with retrieve and cancel) and Anthropic `/v1/messages` with `count_tokens`. `max_tokens=0` on Messages "is rejected". Interleaved thinking replays `reasoning` on the assistant message. | Not applicable. | Not applicable. | Same shape as llama.cpp: plain reasoning text, returned by the caller. | Filled: vLLM | OV76 |
| OpenAI-compatible surface | Ollama | `/v1/chat/completions` supports `reasoning_effort` and `reasoning.effort`; `tool_choice` is unchecked in the supported list. `/v1/responses` is stateless only. Native field is `message.thinking`; the OpenAI layer uses `reasoning`. | `ollama` exists as a key in `provider.options`. | Not applicable. | Third field name for the same thing (`thinking`, `reasoning`, `reasoning_content`). | Filled: Ollama | OV77, OV79 |

## Evidence

Each item gives the claim, the quote or data, the source key, the basis label, and a confidence note where the claim rests on inference.

### OpenRouter mechanisms that apply to all of these vendors

**OV1. On Chat Completions, a caller sends reasoning back in one of three assistant-message fields, and `reasoning_content` is accepted as an alias.**

- Quote: "To preserve reasoning context across multiple turns, you can pass it back to the API in one of two ways: 1. **`message.reasoning`** (string): Pass the plaintext reasoning as a string field on the assistant message 2. **`message.reasoning_details`** (array): Pass the full reasoning\_details block"
- Quote: "For models that only return raw reasoning strings, you can use the simpler `reasoning` field. You can also use `reasoning_content` as an alias - it functions identically to `reasoning`."
- Quote: "When providing reasoning\_details blocks, the entire sequence of consecutive reasoning blocks must match the outputs generated by the model during the original request; you cannot rearrange or modify the sequence of these blocks."
- Source: OV-s1. Basis: OR-documented.
- Note: this is the only place in the 559-page mirror, outside the changelog, where the string `reasoning_content` appears. The page does not say what OpenRouter sends upstream to a vendor whose own field is `reasoning_content`, and it does not say what happens when the caller sends nothing.

**OV2. The `reasoning_details[].format` enum names xAI and Meta formats. Its other values cover OpenAI, Azure, Bedrock, Anthropic, Gemini, an OpenRouter stealth format, and `unknown`, so DeepSeek, Mistral, Moonshot, Alibaba, Z.ai, and MiniMax are outside the named set.**

- Quote (the full list): `"unknown"` "Format is not specified", `"openai-responses-v1"`, `"azure-openai-responses-v1"`, `"bedrock-openai-responses-v1"`, `"bedrock-xai-responses-v1"`, `"xai-responses-v1"` "SpaceXSI responses format version 1", `"meta-responses-v1"` "Meta responses format version 1", `"google-vertex-meta-responses-v1"`, `"google-vertex-xai-responses-v1"`, `"openrouter-stealth-v1"`, `"anthropic-claude-v1"` "Anthropic Claude format version 1 (default)", `"google-gemini-v1"`.
- Changelog quote on the Meta value: "It identifies encrypted reasoning minted by Meta models served on Google Vertex, and the new name says so; the behavior is unchanged."
- Source: OV-s1, OV-s13. Basis: OR-documented.

**OV3. OpenRouter's unified reasoning controls and what it says about these vendors.**

- Quote: `"effort": "high", // Can be "max", "xhigh", "high", "medium", "low", "minimal" or "none" (OpenAI-style)` and `"max_tokens": 2000, // Specific token limit (Anthropic-style)`.
- Quote (effort): "Currently supported by OpenAI reasoning models (o1 series, o3 series, GPT-5 series) and Grok models".
- Quote (budget): "Some Alibaba Qwen thinking models (mapped to `thinking_budget`)" and "For Alibaba, support varies by model".
- Quote: "If a model doesn't support a specific effort level (for example, if a model only supports `low` and `high`), OpenRouter will map your requested effort to the nearest supported level."
- Quote (Messages route): "Claude receives both fields unchanged, while a model such as GPT-5 or MiniMax receives `reasoning_effort: "high"` in its own vocabulary, translated the same way as request-level `reasoning.effort`."
- Source: OV-s1. Basis: OR-documented.

**OV4. The public model catalog publishes a `reasoning` object per model. Values read today for these families:**

- `x-ai/grok-4.7`: `{"mandatory": true, "default_enabled": true, "supported_efforts": ["xhigh", "high", "medium", "low"], "default_effort": "high"}`
- `x-ai/grok-4.5`: `{"mandatory": true, "default_enabled": true, "supported_efforts": ["high", "medium", "low"], "default_effort": "high"}`
- `x-ai/grok-4.20`: `{"mandatory": false, "default_enabled": false}`
- `deepseek/deepseek-v4-pro`: `{"mandatory": false, "supported_efforts": ["xhigh", "high", "none"], "default_effort": "high"}`
- `deepseek/deepseek-v4.1-flash`: `{"mandatory": false, "default_enabled": true, "supported_efforts": ["max", "high", "low", "none"], "default_effort": "high"}`
- `mistralai/mistral-large-4-0`: `{"mandatory": false, "default_enabled": true, "supported_efforts": ["high", "none"], "default_effort": "high"}`
- `moonshotai/kimi-k3`: `{"mandatory": false, "default_enabled": true, "supported_efforts": ["max", "high", "low", "none"], "default_effort": "max"}`
- `moonshotai/kimi-k2.7-code`: `{"mandatory": true, "default_enabled": true}`
- `qwen/qwen4-max`: `{"mandatory": false, "default_enabled": true, "supported_efforts": ["xhigh", "high", "low", "none"], "default_effort": "high"}`
- `qwen/qwen3.8-flash`: `{"mandatory": false, "default_enabled": true, "supports_max_tokens": true}`
- `z-ai/glm-5.3`: `{"mandatory": true, "default_enabled": true, "supported_efforts": ["max", "high", "low"], "default_effort": "max"}`
- `minimax/minimax-m3`: `{"mandatory": false}`; `minimax/minimax-m2.5`: `{"mandatory": true}`
- `meta/muse-spark-1.3`: `{"mandatory": true, "supported_efforts": ["max", "xhigh", "high", "medium", "low", "minimal"], "default_effort": "medium"}`
- `meta-llama/llama-4-maverick`, `mistralai/mistral-nemo`, `deepseek/deepseek-chat`: `null`
- Doc quote on the fields: "**`mandatory`**: When `true`, hide disable controls and do not send `effort: "none"`" and "**`supports_max_tokens`**: When present and `true`, show a token budget control and send `reasoning.max_tokens` instead of (or alongside) `reasoning.effort`."
- Source: OV-s11 (data), OV-s1 (field meanings). Basis: OR-documented.

**OV5. Unsupported parameters are ignored, and Mistral's `safe_prompt` is passed through by name.**

- Quote: "If the chosen model doesn't support a request parameter (such as `logit_bias` in non-OpenAI models, or `top_k` for OpenAI), then the parameter is ignored. The rest are forwarded to the underlying model API."
- Quote: "With the default routing strategy, providers that don't support all the LLM parameters specified in your request can still receive the request, but will ignore unknown parameters. When you set `require_parameters` to `true`, the request won't even be routed to that provider." (link markup removed)
- Quote: "Even when `require_parameters` is `false`, a small set of parameters is used as a soft preference when choosing between providers of the same model: `tools`, the response format (`text.format` on the Responses API, `response_format` on Chat Completions, including structured outputs), and `verbosity`."
- Quote: "OpenRouter will also transmit some provider-specific parameters, such as `safe_prompt` for Mistral or `raw_mode` for Hyperbolic directly to the respective providers if specified."
- Source: OV-s14, OV-s6, OV-s5. Basis: OR-documented.

**OV6. `provider.options` is a per-provider passthrough object, keyed by provider slug, and the spec lists a key for every vendor in this topic.**

- Spec description of `ProviderOptions`: "Provider-specific options keyed by provider slug. The provider segment is case-insensitive; an endpoint tag after `/` must match the endpoint slug exactly. Only options for the matched provider are forwarded; the rest are ignored. Unrecognized keys are silently dropped."
- The schema has 158 keys. Present: `xai`, `deepseek`, `mistral`, `moonshotai`, `alibaba`, `z-ai`, `minimax`, `meta`, `ollama`, `inferact-vllm`, `groq`, `together`, `fireworks`. Each is `{"additionalProperties": {}, "type": "object"}`, so the spec does not say which option names each provider accepts.
- Provider-side doc quote: "OpenRouter merges the parameter names from every scope into the endpoint's passthrough allowlist, which controls the `provider.options` keys forwarded to your API. Descriptors are not enforced yet."
- Source: OV-s10, OV-s3. Basis: OR-documented.
- Note: this is the documented way for a caller to send a vendor-only field such as `enable_search` or `clear_thinking`. Whether a given name is on a given endpoint's allowlist is not published. That is a closed-source gap (see Open questions).

**OV7. The endpoints API publishes `supports_tool_choice` per endpoint, and the values differ between providers of the same model.**

- Reference page example: `supports_tool_choice: auto: true, function: true, none: true, required: true`. Changelog: "`PublicEndpoint`: required `supports_tool_choice` property added".
- Data fetched today (letters: n = none, a = auto, r = required, f = function):
  - `deepseek/deepseek-v4-pro` (13 endpoints): Baidu na, StreamLake a, Relace a, SiliconFlow na, DeepInfra na, DigitalOcean a, Novita na, Alibaba na, Venice na, Azure narf.
  - `deepseek/deepseek-v4.1-flash` (37 endpoints): most narf; Relace a; one Parasail endpoint a and the other naf; Novita with all four false; Baidu naf.
  - `qwen/qwen3.6-27b`: Alibaba na; Chutes arf; SiliconFlow af; Venice narf. `qwen/qwen3.8-flash`: Alibaba na. `qwen/qwen4-max`: Alibaba narf.
  - `z-ai/glm-5.3` (46 endpoints): Z.AI a; Alibaba narf; Mistral naf; Together narf; Fireworks narf; Amazon Bedrock nar. `z-ai/glm-4.7`: Z.AI a; Novita a; Venice a.
  - `moonshotai/kimi-k3` (29 endpoints): Moonshot AI narf; Alibaba narf; Relace na; Amazon Bedrock nar. `moonshotai/kimi-k2.7-code`: Moonshot AI narf; Alibaba na; Novita na.
  - `mistralai/mistral-large-4-0`: Mistral narf (two endpoints). `mistralai/mistral-nemo`: Mistral narf; Io Net arf; DeepInfra na.
  - `minimax/minimax-m3`: CoreWeave narf; DeepInfra a and na; Parasail na. `minimax/minimax-m2.7`: Groq a.
  - `meta/muse-spark-1.3`: Meta narf; Google narf. `x-ai/grok-4.7`: xAI narf; Amazon Bedrock narf.
- Source: OV-s12 (data), OV-s16 (reference page), OV-s13 (changelog). Basis: OR-documented.
- Note: the docs read today do not say whether a request with `tool_choice: "required"` is routed only to endpoints where `required` is true. The soft-preference list in OV5 names `tools`, the response format, and `verbosity`, and leaves `tool_choice` out.

**OV8. Which provider serves each family: first-party endpoints exist for xAI, Mistral, Moonshot, Alibaba, Z.AI, and Meta, and the DeepSeek and MiniMax models fetched today list only other hosts.**

- `x-ai/grok-4.7`: xAI (4 endpoints, tags `xai`, `xai/zdr`, `xai/priority`, `xai/zdr/priority`) and Amazon Bedrock. `x-ai/grok-4.20`: xAI only.
- `deepseek/deepseek-v4-pro`: Alibaba, Azure, Baidu, DeepInfra, DigitalOcean, Novita, Relace, SiliconFlow, StreamLake, Venice. `deepseek/deepseek-v4.1-flash`: 37 endpoints across 28 hosts. `deepseek/deepseek-v3.2`: SiliconFlow, DeepInfra, Baidu, DigitalOcean. `deepseek/deepseek-chat`: DeepInfra, StreamLake. Every provider name in these four lists is one of those hosts, and "DeepSeek" is not among them.
- `mistralai/mistral-large-4-0` and `mistralai/mistral-medium-3-5`: Mistral only. `mistralai/mistral-nemo`: Mistral plus DeepInfra, DekaLLM, Io Net, Parasail.
- `moonshotai/kimi-k3`: Moonshot AI plus 19 other hosts. `moonshotai/kimi-k2.7-code`: Moonshot AI (two endpoints) plus 7 others.
- `qwen/qwen4-max` and `qwen/qwen3.8-flash`: Alibaba only. `qwen/qwen3.6-27b`: Alibaba, Chutes, DeepInfra, Reka, SiliconFlow, Venice.
- `z-ai/glm-5.3`: Z.AI plus 33 other hosts, among them Mistral, Alibaba, Baidu. `z-ai/glm-4.7`: Z.AI, DeepInfra, Venice, Novita, Mancer.
- `minimax/minimax-m3`: CoreWeave, DeepInfra, Featherless, Parasail, Venice. `minimax/minimax-m3.1-flash`: 13 endpoints, among them "Inferact vLLM". `minimax/minimax-m2.7`: Groq. Every provider name in these three lists is a third-party host, and "MiniMax" is not among them.
- `meta/muse-spark-1.3`: Meta and Google (tag `google-vertex/global`). `meta-llama/llama-3.3-70b-instruct`: seven third-party hosts. `meta-llama/llama-4-maverick`: Parasail.
- Source: OV-s12. Basis: OR-documented (public endpoint lists).
- Note: the spec still has `deepseek` and `minimax` keys in `ProviderOptions` (OV6) and the stream-cancellation list names both (OV12), so these providers exist in OpenRouter's system. The lists show which endpoints were published at fetch time.

**OV9. What the prompt-caching guide says per vendor.**

- Grok: "Prompt caching with Grok is automated and does not require any additional configuration." Reads at 0.25x.
- Moonshot AI: "Prompt caching with Moonshot AI is automated and does not require any additional configuration." Reads at 0.25x.
- Groq: "Currently available on Kimi K2 models."
- Alibaba Qwen: "Alibaba prompt caching requires explicit cache breakpoints. Add `cache_control: { "type": "ephemeral" }` to content blocks you want to cache, using the same syntax as Anthropic explicit caching. Cache writes use a 5-minute TTL." and "Alibaba explicit caching is available on `deepseek/deepseek-v3.2`, `qwen/qwen3-max`, `qwen/qwen-plus`, `qwen/qwen3.6-plus`, `qwen/qwen3-coder-plus`, and `qwen/qwen3-coder-flash`. Snapshot endpoints, including `qwen/qwen3.5-plus-02-15` and `qwen/qwen3.5-flash-02-23`, do not support explicit caching." Writes 1.25x, reads 0.1x.
- DeepSeek: "Prompt caching with DeepSeek is automated and does not require any additional configuration." Writes at input price, reads 0.1x.
- Z.AI: "Cache reads are reported in the `cached_tokens` field of `prompt_tokens_details` in the usage response." and "To improve cache hit rates, OpenRouter sends Z.AI a session affinity key with each request, derived from your account and, when provided, your `session_id`."
- Sticky routing: "Sticky routing is tracked at the account level, per model, and per conversation. By default, OpenRouter identifies conversations by hashing the first system (or developer) message and the first non-system message in each request" and "If neither is set, OpenRouter falls back to the OpenAI-style `prompt_cache_key` request field as the sticky routing key."
- Source: OV-s2. Basis: OR-documented.

**OV10. OpenRouter's native search list has xAI and no other vendor from this topic, and X search is opt-in.**

- Quote: "The following providers have native web search:" followed by OpenAI, Anthropic, Google, "**SpaceXSI**: Grok 4 and later (web search; X search requires the `x_search` opt-in)" and Perplexity (link markup removed).
- Quote: "On SpaceXSI models, native web search sends only xAI's `web_search` tool. To also search X/Twitter, add an `x_search` object."
- Quote: "Because a single X search can now return many billable items, OpenRouter no longer adds `x_search` automatically. The legacy top-level `x_search_filter` request field still enables X search for backward compatibility."
- Endpoint data: `native_tools` is `{"openrouter:web_search": {"type": "web_search"}}` on the four xAI endpoints of `x-ai/grok-4.7` and on the Meta endpoint of `meta/muse-spark-1.3`, and `{}` on every Moonshot AI, Alibaba, Z.AI, Mistral, and third-party endpoint checked.
- Quote: "For models without native search, set `engine` to one of the other supported options (Exa, Firecrawl, Parallel, or Perplexity), or leave it as `"auto"` to default to Exa." (link markup removed)
- Source: OV-s7, OV-s8, OV-s12. Basis: OR-documented.
- Note: the Meta endpoint has native web search in the endpoint data although the prose list of native search providers leaves Meta out.

**OV11. OpenRouter's provider-integration pages and spec set no rule for the reasoning field name a provider returns, and no rule for tool-call id format.**

- The current integration page (OV-s3) describes the model-list feed, pricing, capacity, passthrough parameters, uptime, and Auto Exacto. Its only statement on reasoning is the capability descriptor `"reasoning": { "type": "boolean" }` and the advice to "send SSE comments as keep-alives". The legacy page (OV-s4, fetched live) lists "Valid features are: `tools`, `json_mode`, `structured_outputs`, `logprobs`, `web_search`, `reasoning`."
- Capability tests decide tool support: "parameters OpenRouter verifies by test (`tools`, `tool_choice`, `response_format`, `structured_outputs`, `logprobs`, and the rest of the capability suite) are never enabled from the feed, only by the capability tests passing."
- The tool-calling guide's general statement: "OpenRouter standardizes the tool calling interface across models and providers, making it easy to integrate external tools with any supported model." It gives no per-vendor id rule (OV-s15).
- Spec: `ChatAssistantMessage` properties are `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`. The string `reasoning_content` appears in no schema of the 1071. `ChatToolCall.id` is `{"description": "Tool call identifier", "type": "string"}`.
- Source: OV-s3, OV-s4, OV-s10, OV-s15. Basis: OR-documented.
- Confidence note: this is an absence across four documents, searched by string. The task brief expected the integration page to cover field names such as `reasoning_content` versus `reasoning`; the page as mirrored today does not.

**OV12. Stream cancellation support differs per provider.**

- Quote, supported: "Avian, XAI, Cloudflare" and "Friendli, Chutes, DeepSeek". Not currently supported: "Google, Google AI Studio, Minimax", "Mistral, AI21, Featherless", "AionLabs, Alibaba, Nebius".
- Source: OV-s9. Basis: OR-documented.

### xAI

**OV13. xAI documents the Responses API as primary and Chat Completions as deprecated.**

- Quote: "The Responses API is the recommended way to interact with SpaceXSI models. Here's how it compares to the legacy Chat Completions API:"
- Comparison table, column headers "Responses API" and "Chat Completions API (Deprecated)": Stateful Conversations "Built-in support via `previous_response_id`"; Server-side Storage "Responses stored for 30 days"; Reasoning Models "Full support with encrypted reasoning content" against "No reasoning content returned"; Agentic Tools "Native support for tools (search, code execution, MCP)" against "Function calling only".
- Legacy guide quote: "Chat Completions is offered as a legacy endpoint."
- Source: OV-s18, OV-s28. Basis: Vendor-documented. Also in workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`.

**OV14. Encrypted reasoning exists only on xAI's Responses route, and `grok-4.7` always returns it.**

- Quote: "The reasoning content is encrypted by us and can be returned if you pass `include: ["reasoning.encrypted_content"]` to the Responses API. You can send the encrypted content back to provide more context to a previous conversation."
- Quote: "On the Responses API, `grok-4.7` returns `reasoning.encrypted_content` on every response, whether or not `include` lists it, together with the encrypted outputs of any server-side tools. Reasoning items in the output carry an `encrypted_content` field; pass them back unchanged in the next request's `input` so the model keeps its reasoning across turns even when you manage conversation history yourself."
- Quote: "Chat Completions is unaffected; it has no field for the ciphertext."
- Quote (30-day rule): "If you want to continue a conversation after 30 days, please store your responses history and the encrypted thinking content locally, and pass them in a new request body."
- Source: OV-s17, OV-s25. Basis: Vendor-documented.

**OV15. xAI effort values, the default, and reasoning that cannot be switched off.**

- Quote: "`grok-4.7`, `grok-4.6`, and `grok-4.5` support the `reasoning_effort` parameter" and "If not specified, `reasoning_effort` defaults to `"high"`. Reasoning cannot be disabled."
- Quote: "`presencePenalty`, `frequencyPenalty`, and `stop` cannot be used with reasoning models. Requests that include them return an error."
- Quote: "`"xhigh"` is available on `grok-4.6` and later. On models that do not support it, such as `grok-4.5`, requests with `"xhigh"` are treated as `"high"`."
- Summary table rows: `grok-4.7` and `grok-4.6` "`reasoning.effort`: `"low"` / `"medium"` / `"high"` (default) / `"xhigh"`" "Controls reasoning depth (cannot be disabled)"; `grok-4.5` without `xhigh`; `grok-4.20-multi-agent` "Controls agent count (4 or 16)".
- Source: OV-s17. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey ("no off switch on 4.5 and later").

**OV16. xAI caps tools at 1,000 on both routes.**

- Responses reference: "`tools` (array | null)" "A list of tools the model may call in JSON-schema. Currently, only functions and web search are supported as tools. A max of 1000 tools are supported."
- Chat reference: "A list of tools the model may call in JSON-schema. Currently, only functions are supported as a tool. Use this to provide a list of functions the model may generate JSON inputs for. A max of 1000 functions are supported."
- Source: OV-s20, OV-s21. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey.

**OV17. xAI's search tools are server-side tools on the Responses route.**

- Tools overview: "**Built-in Tools**" "Server-side tools managed by SpaceXSI that execute automatically" "Web Search, X Search, Code Interpreter, Image Generation, Collections Search".
- Web search page: tool name `web_search` for "OpenAI Responses API"; filters `allowed_domains` "(max 5)" and `excluded_domains` "(max 5)", which "cannot be set together".
- The chat reference still lists a `search_parameters` object.
- Fill from LiteLLM (pinned commit, OV-s75): the xAI chat transformation logs "XAI no longer supports web search on /chat/completions (Live Search is deprecated). Dropping 'web_search_options'. Use the Responses API for XAI web search."
- Source: OV-s27, OV-s26, OV-s21, OV-s75. Basis: Vendor-documented for the first three bullets; the LiteLLM line is Filled (LiteLLM).

**OV18. xAI caching is automatic, with a per-conversation routing hint and OpenAI-style cached token fields.**

- Quote: "The xAI API performs prompt caching **automatically**. However, we recommend setting the `x-grok-conv-id` HTTP header to maximize your cache hit rate."
- Quote: "The `x-grok-conv-id` HTTP header routes requests with the same conversation ID to the same server. Since cache entries are stored per-server, this maximizes your cache hit rate."
- Chat reference: "`prompt_cache_key` (string | null)" "A stable cache key for best-effort sticky routing / prompt-cache hits across requests sharing a prompt prefix. Plumbed to `x-grok-conv-id`, same as on `/v1/responses`."
- Quote: "Prompt caching is not 100% guaranteed. Cache entries can be evicted due to memory pressure, and requests may be routed to different servers."
- Usage: "Cached tokens appear in `usage.prompt_tokens_details.cached_tokens`" (chat) and "Cached tokens appear in `usage.input_tokens_details.cached_tokens`" (Responses).
- Source: OV-s22, OV-s19, OV-s21, OV-s23, OV-s24. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey ("Auto, routing hint").

**OV19. xAI states which Responses fields it accepts without acting on them.**

- Quotes from the Responses reference: `background` "(Unsupported) Whether to process the response asynchronously in the background."; `context_management` "Optional context-management directives (e.g. compaction). Parsed but not yet executed."; "OpenAI's \`message.output\_text.logprobs\` is accepted for compatibility but silently ignored."; `logprobs` "Not supported by models \`grok-4.20\` and newer; the field will be silently ignored if set."; `metadata` "Not supported. Only maintained for compatibility reasons."; `truncation` "Not supported. Only maintained for compatibility reasons."
- Source: OV-s20. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey.

**OV20. How OpenRouter exposes xAI.**

- Documented: the effort setting is listed as supported by "Grok models" (OV3); the catalog marks reasoning mandatory with the vendor's effort list (OV4); reasoning replay uses `reasoning_details` with `format` `xai-responses-v1` (OV2); search goes through `openrouter:web_search` with `engine: "native"` and an `x_search` opt-in (OV10); caching is "automated" (OV9).
- Quote: "OpenRouter automatically requests encrypted reasoning content from reasoning-capable providers, so replay the returned `encrypted_content` unchanged; no explicit `include` field is required".
- Inference: OpenRouter calls xAI's Responses route upstream even when the caller uses OpenRouter's Chat Completions route. Inferred from two documented facts: the format value is named `xai-responses-v1`, and xAI says its Chat Completions route "has no field for the ciphertext" (OV14), yet OpenRouter returns encrypted entries for Grok.
- Source: OV-s1, OV-s7, OV-s11. Basis: OR-inferred. Confidence: medium. OpenRouter's docs do not state the upstream route.

### DeepSeek

**OV21. DeepSeek switches thinking with a `thinking` object or with `reasoning_effort`, and maps foreign effort names.**

- Table quote: Thinking Mode Toggle `{"thinking": {"type": "enabled/disabled"}}` (OpenAI format), `{"reasoning": {"effort": "none/low/high/max"}}` (Responses API format); Thinking Effort Control `{"reasoning_effort": "low/high/max"}`.
- Quote: "Thinking mode is enabled by default, with the default effort being high".
- Mapping table: `minimal` to `low`, `low` to `low`, `medium` to `high`, `high` to `high`, `xhigh` to `high`, `max` to `max`, `ultra` to `max`.
- API reference quote: "When thinking is omitted or null, the mode follows reasoning_effort; if reasoning_effort is also omitted, thinking is enabled. When this object is provided, type is required and takes precedence over reasoning_effort for selecting the mode."
- Quote: "Thinking mode does not support the temperature, presence_penalty, or frequency_penalty parameters. Please note that, for compatibility with existing software, setting these parameters will not trigger an error but will also have no effect."
- Source: OV-s29, OV-s30. Basis: Vendor-documented.

**OV22. DeepSeek's replay rule depends on whether the request has `tools`.**

- Quote: "If the request carries the tools parameter: the reasoning_content of all previous turns should be passed back to the API and will be concatenated into the context."
- Quote: "If the request does not carry the tools parameter: reasoning_content does not need to be passed back; even if passed to the API, it will be ignored and will not be concatenated into the context."
- Quote: "Please note that for requests carrying the tools parameter, the reasoning_content must be fully passed back to the API in all subsequent requests" and, from the same sentence, "even for turns where the model did not perform a tool call. If your code does not correctly pass back reasoning_content, the API will return a 400 error."
- Quote from the sample: "Additionally, in the Turn 2 request, we still pass the reasoning_content generated in Turn 1 to the API."
- Source: OV-s29. Basis: Vendor-documented. Also in workspace: both reports. The live page confirms the rule as recorded on 2026-09-20 and 2026-10-28.

**OV23. The current error text for a missing `reasoning_content`.**

- DeepSeek's live pages state the status ("the API will return a 400 error") and do not print the message string.
- LiteLLM docstring (pinned commit): "DeepSeek thinking mode requires `reasoning_content` to be passed back on every assistant message in multi-turn conversations. If it is missing, the API returns: "The reasoning_content in the thinking mode must be passed back to the API.""
- Third-party issue report (openclaw issue 71160, created 2026-04-24), body as received through OpenRouter: `400 Provider returned error {"error":{"message":"The `reasoning_content` in the thinking mode must be passed back to the API.","type":"invalid_request_error","param":null,"code":"invalid_request_error"}}`
- Third-party post (Reddit r/openrouter, 2026-05-29) quotes the same JSON when the provider was pinned to `deepseek`.
- Source: OV-s73, OV-s86, OV-s89. Basis: Filled (LiteLLM), supported by two third-party reports. Also in workspace: the 2026-09-20 report quotes the same sentence. Confidence: high for the wording as of April and May 2026; not observed first-hand today.

**OV24. In thinking mode DeepSeek rejects `required` and named tool choices.**

- Quote: "required and named tool choices are not supported in thinking mode; the API returns a 400 error. Disable thinking mode first to use them. With the web tools declared, required is treated as auto."
- Quote: "Setting tool_choice to none disables tool calling even when tools is provided."
- Source: OV-s30. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey.

**OV25. DeepSeek reports cache hits and misses in its own fields and in the OpenAI field.**

- Field descriptions in the API reference: `prompt_tokens` "Number of tokens in the prompt. It equals prompt_cache_hit_tokens + prompt_cache_miss_tokens."; `cached_tokens` "Number of tokens in the prompt that hit the context cache. Same as prompt_cache_hit_tokens."; `prompt_cache_hit_tokens` "Number of tokens in the prompt that hits the context cache."; `prompt_cache_miss_tokens` "Number of tokens in the prompt that misses the context cache."
- Quote: "The DeepSeek API Context Caching on Disk Technology is enabled by default for all users".
- Stream shape quote: "no separate usage-only chunk is emitted: the statistics ride on the last content chunk, whose choices array always contains exactly one element that carries no new content and a non-null finish_reason."
- Quote: "`user_id` can be used for KVCache isolation for privacy management."
- Source: OV-s30, OV-s32. Basis: Vendor-documented.

**OV26. Prefix completion and strict tool mode are beta features on a separate base URL.**

- Quote: "users must ensure that the role of the last message in the messages list is assistant and set the prefix parameter of the last message to True." and "The user needs to set base_url="https://api.deepseek.com/beta" to enable the Beta feature."
- API reference, field `reasoning_content` on an assistant message: "(Beta) Used for the thinking mode in the Chat Prefix Completion feature as the input for the CoT in the last assistant message. When using this feature, the prefix parameter must be set to true."
- Strict mode quote: "Use base_url="https://api.deepseek.com/beta" to enable Beta features" and "all function need to set the strict property to true".
- Source: OV-s33, OV-s30, OV-s31. Basis: Vendor-documented.

**OV27. DeepSeek has extra finish reasons and now documents three protocols.**

- API reference, field `finish_reason`: "Possible values: stop, length, content_filter, tool_calls, insufficient_system_resource, aborted".
- The docs navigation lists "Using the Responses API" and "Using the Anthropic API" under Protocols. The Anthropic page says: "To use the web tools, declare the server tools deepseek_web_search and deepseek_read_url."
- The news page says the names `deepseek-chat` and `deepseek-reasoner` "will be discontinued in three months (2026-07-24)" and point to `deepseek-v4-flash` modes meanwhile. Runvil's 2026-09-14 fixture lists `deepseek-flash` and `deepseek-v4-pro`.
- Source: OV-s30, OV-s34, OV-s35, OV-s92. Basis: Vendor-documented.

**OV28. How OpenRouter handles the DeepSeek replay rule.**

- Documented: the caller may send `reasoning`, `reasoning_details`, or the alias `reasoning_content` (OV1). OpenRouter's pages do not say that it supplies the field when the caller left it out, and do not mention the DeepSeek 400.
- Documented: the DeepSeek models fetched today are served by third-party and reseller hosts (OV8), each with its own `supports_tool_choice` (OV7).
- Third-party evidence that the 400 reached OpenRouter callers: openclaw issue 71160 (2026-04-24) with model `openrouter/deepseek/deepseek-v4-pro`. A maintainer comment in that issue: "Replaying the follow-up without that reasoning still reproduces the provider 400. Replaying with `message.reasoning` or `message.reasoning_content` succeeds with HTTP 200."
- Third-party evidence of an OpenRouter-side change: opencode issue 24569, maintainer comment dated 2026-04-27: "Okay their change rolled out to their api, I tested and was unable to reproduce issue after". opencode issue 24261: "I talked w/ openrouter team and they made this patch specifically due to us raising the issue."
- Third-party evidence after that date: the Reddit post of 2026-05-29 reports the same 400 with `provider.order` set to `["deepseek"]`.
- Source: OV-s1, OV-s12, OV-s86, OV-s87, OV-s88, OV-s89. Basis: OR-inferred. Confidence: medium that OpenRouter accepts any of the three fields and forwards reasoning to DeepSeek-family hosts as `reasoning_content`; low on what it does when the caller sends no reasoning.

**OV29. How three open projects handle the DeepSeek rule.**

- LiteLLM (DeepSeek's own API): `_fill_reasoning_content` promotes a stored value from `provider_specific_fields["reasoning_content"]`, otherwise sets `patched["reasoning_content"] = " "` and logs "A single-space placeholder is being injected to satisfy API validation, but the model will receive a blank reasoning chain for this turn, which may silently degrade multi-turn response quality."
- pi (pinned commit): compat flag `requiresReasoningContentOnAssistantMessages`, described as "Whether all replayed assistant messages must include an empty reasoning_content field when reasoning is enabled. Default: auto-detected from URL." The default is `requiresReasoningContentOnAssistantMessages: isDeepSeek`, where `isDeepSeek = provider === "deepseek" || baseUrl.toLowerCase().includes("deepseek.com")`. For OpenRouter URLs the flag is false and pi replays `reasoning_details` (`assistantMsg.reasoning_details = preservedReasoningDetails`).
- opencode (pinned commit), generic OpenAI chat protocol: `requireReasoning` defaults to true when `request.model.provider === "deepseek"`, the base URL includes `deepseek.com`, or `modelID.includes("deepseek")`. Its OpenRouter protocol then overrides the assistant message with `reasoning_content: undefined, reasoning_text: undefined`, and sends `reasoning` and `reasoning_details`.
- LiteLLM's OpenRouter stream parser copies OpenRouter's field into its own name: `choice["delta"]["reasoning_content"] = choice["delta"].get("reasoning")` (`litellm/llms/openrouter/chat/transformation.py` line 323).
- Source: OV-s73, OV-s76, OV-s77, OV-s78, OV-s80, OV-s81. Basis: Filled (LiteLLM, pi, opencode).
- What the fills show: against DeepSeek's own endpoint, three projects each patch the assistant message (space, empty string, or stored text). Through OpenRouter, pi and opencode both send `reasoning_details` back and rely on OpenRouter for the vendor field.

### Mistral

**OV30. Mistral tool-call ids are exactly nine alphanumeric characters, and the API rejects others with code 3280.**

- Vendor quote (fine-tuning data format section of the docs): "Both `"id"` and `"tool_call_id"` are randomly generated strings of exactly 9 characters."
- The error glossary read today lists common codes (1000, 1100, 1300, 1500, 2300 to 2308, 3051, 3054, 3059, 3505, 3800, 3801) and says "It isn't a complete list". Code 3280 is absent from that table.
- Error text, from third-party reports of real responses: `{"object":"error","message":"Tool call id was call_5019f900a247472bacde0b82 but must be a-z, A-Z, 0-9, with a length of 9.","type":"invalid_function_call","param":null,"code":"3280"}` (Roo Code pull request 10102) and `{"object":"error","message":"Tool call id was exec1774786568428215 but must be a-z, A-Z, 0-9, with a length of 9.","type":"invalid_function_call","param":null,"code":"3280","raw_status_code":400}` (openclaw issue 57672, created 2026-03-30).
- Source: OV-s36, OV-s83, OV-s85. Basis: Vendor-documented for the nine-character rule; the error string rests on third-party reports. Also in workspace: the 2026-09-20 report ("exactly nine alphanumerics", "error code 3280"). Confidence: high.

**OV31. With reasoning on, Mistral returns content as a list of chunks, the stream changes shape mid-response, and the full message must be replayed.**

- Quote: "When `reasoning_effort` is set to `"high"`, the response `message.content` is a **list of chunks** instead of a plain string." Chunk types: "**`ThinkChunk`** (`type: "thinking"`): contains the model's reasoning trace. The `thinking` field is itself a list of `TextChunk` objects." and "**`TextChunk`** (`type: "text"`): contains the final answer."
- Quote: "When streaming, `delta.content` changes shape during the response: 1. **Thinking phase**: `delta.content` is a list containing a `ThinkChunk` 2. **Transition**: a single list containing both a closing `ThinkChunk` and the first `TextChunk` 3. **Answer phase**: `delta.content` is a plain string"
- Quote: "When building multi-turn conversations with reasoning, **always replay the full assistant message (including `ThinkChunk`) back into the message history**. Dropping the reasoning trace across turns degrades model performance." and "Do not strip `ThinkChunk` from assistant messages before replaying them."
- OpenAPI schema `ThinkChunk`: properties `type` (const `thinking`), `thinking` (array of `TextChunk`, `ToolReferenceChunk`, `ReferenceChunk`), `signature` ("Signature to replay some reasoning blocks across turns."), and `closed` ("Whether the thinking chunk is closed or not. Currently only used for prefixing.").
- Quote on a hosted third-party model: "`zai-glm-5-3` supports `low`, `high`, and `max`" and for it "`message.content` is always returned as a list of chunks (thinking + text) regardless of the `reasoning_effort` value; there is no plain-string mode."
- Source: OV-s36, OV-s37. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey (the delta shape change). The `signature` field is new relative to both workspace reports.

**OV32. Mistral's effort values differ per model.**

- For `mistral-small-2603` and `mistral-medium-3-5`, quotes: "`reasoning_effort = "high"`: The response includes a full thinking chunk before the final answer, at the cost of increased token usage." and "`reasoning_effort = "none"`: The model thinks minimally and the thinking chunk is omitted from the response."
- Quote: "Mistral Large 4 (`mistral-large-4` / `mistral-large-4-0`) supports three reasoning levels" (`low`, `medium`, `high`) and "Mistral Large 4 does not accept `none`, `minimal`, `xhigh`, or `max`; these are not aliases for its supported levels. The API-wide enum and SDKs include values used by other models, so a value appearing in the enum does not mean every model accepts it."
- Quote: "If you omit `reasoning_effort`, Mistral Large 4 uses `high`."
- Error glossary: "An argument value the model doesn't support, such as too many images or an unsupported `reasoning_effort` value (code `3051`)".
- Quote: "Native reasoning models (`magistral-small-latest`, `magistral-medium-latest`) are deprecated."
- Source: OV-s36. Basis: Vendor-documented.

**OV33. `prompt_mode`, `safe_prompt`, `prompt_cache_key`, and the tool-choice enum in Mistral's OpenAPI spec.**

- `prompt_mode`: "Allows toggling between the reasoning mode and no system prompt. When set to `reasoning` the system prompt for reasoning models will be used." Enum `MistralPromptMode` has one value, `reasoning`.
- `safe_prompt`: "Whether to inject a safety prompt before all conversations." Default `false`. The guide page says: "`safe_prompt` is deprecated. We recommend using Custom Guardrails instead" (link markup removed).
- `ToolChoiceEnum`: `auto`, `none`, `any`, `required`.
- `ChatCompletionRequest` has `additionalProperties: false`. The error glossary lists as a 422 cause: "A field the endpoint doesn't accept, such as `store` sent by some OpenAI clients (`Extra inputs are not permitted`)".
- Source: OV-s37, OV-s36. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey (`prompt_mode`, `safe_prompt`). The deprecation of `safe_prompt` is new relative to the survey.

**OV34. Mistral's error envelope has four shapes.**

- Quote: "Most errors return a JSON body with this structure:" `{"object": "error", "message": "Rate limit exceeded", "type": "rate_limited", "param": null, "code": "1300", "raw_status_code": 429}`.
- Quote: "**401 errors** return `{"detail": "Invalid API Key"}`."; "**422 validation errors** return `{"detail": [...]}`, with one item per invalid field."; "**Agents and Conversations errors** have `"object": "Error"`, `"type": "invalid_request_error"`, and a numeric `code`."; "**The OpenAI-compatible endpoint** wraps the error in an `error` object, as OpenAI clients expect." (link markup removed)
- Overflow: "A prompt longer than the model's context window (code `3059`)", message `Prompt <N> > <M> maximum context length`, type `invalid_request_prompt_too_long`.
- Through OpenRouter (third-party report, claude-code-router issue 1070): the Mistral JSON appears escaped inside OpenRouter's `error.metadata.raw`.
- Source: OV-s36, OV-s94. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey (the `object: error` shape and the 422 `detail` list).

**OV35. Mistral has two chat-completions endpoints, and `/v1` is the native-format one.**

- Quote: "The OpenAI-compatible endpoint is in public preview. Its behavior, supported parameters, and error responses can change during the preview."
- Quote: "The endpoint is available at `https://api.mistral.ai/openai/v1/chat/completions` and uses the OpenAI Chat Completions request and response format. The standard `https://api.mistral.ai/v1/chat/completions` endpoint uses the native Mistral format and works with the Mistral SDKs."
- Source: OV-s36. Basis: Vendor-documented.
- Note: Runvil's provider file sets `openai_base_url: Some("https://api.mistral.ai/v1")` (OV80). The docs read today do not list what the preview endpoint does with `ThinkChunk` content or nine-character ids.

**OV36. Mistral prompt caching.**

- Quote: "Cached prompt tokens are billed at 10% of the standard input token price" and "`prompt_cache_key` increases the chance of a cache hit, but it **doesn't guarantee one**."
- Quote: "Cache blocks contain 64 tokens. Because of this block size: `cached_tokens` is a multiple of 64. Prompts with fewer than 64 prompt tokens do not have cache hits."
- Source: OV-s36. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey. The `prompt_cache_key` field is new relative to the survey, which listed Mistral under "Automatic, no control".

**OV37. How OpenRouter handles Mistral's rules.**

- Documented: `safe_prompt` is forwarded by name (OV5). The catalog lists `mistralai/mistral-large-4-0` with efforts `["high", "none"]` (OV4). Mistral's first-party endpoints have tags `mistral/zdr` and `mistral/eu` (OV8).
- Tool-call ids: no statement in the mirror, spec, or changelog. Third-party reports show the 3280 error reaching OpenRouter callers: claude-code-router issue 1070 (comment dated 2025-12-19), openclaw issue 52548 (created 2026-03-23), openclaw issue 57672 (created 2026-03-30). In issue 57672 a commenter wrote on 2026-03-31: "Hi I work at OpenRouter - we are fixing this on our end! Will be fixed today."
- `ThinkChunk`: OpenRouter's response shape keeps `content` a string and puts reasoning in `reasoning` and `reasoning_details` (OV1, OV2). No document says how a Mistral chunk list or its `signature` is stored.
- Source: OV-s5, OV-s11, OV-s12, OV-s84, OV-s85, OV-s94. Basis: OR-inferred. Confidence: medium that OpenRouter rewrites ids for Mistral since April 2026 (one employee comment, no documentation); low on signature handling.

**OV38. How open projects handle the nine-character rule.**

- opencode, generic OpenAI chat protocol (so it also applies to `mistralai/...` model ids through OpenRouter): `const mistral = ["mistral", "devstral", "codestral", "pixtral", "mixtral"].some((family) => modelID.includes(family))` and, when true, `return id.replace(/[^a-zA-Z0-9]/g, "").slice(0, 9).padEnd(9, "0")` (shown on four lines in the source). The same file sets `requireAssistantAfterTool` to `mistral` by default and inserts `{ role: "assistant", content: "Done." }` after a tool message.
- opencode, Mistral's own protocol: `MistralToolID.normalizer` keeps ids matching `/^[A-Za-z0-9]{9}$/` and hashes the others to nine base-36 characters, with a retry on collision.
- pi, Mistral's own API: `const MISTRAL_TOOL_CALL_ID_LENGTH = 9;` and `deriveMistralToolCallId` (strip non-alphanumerics, keep if length is nine, otherwise hash and slice). pi's generic chat-completions path only truncates ids to 40 characters for the `openai` provider.
- Roo Code pull request 10102 (third-party description): "Added `modelId` option to `convertToOpenAiMessages()` that conditionally normalizes IDs only when the model contains 'mistral'" and "OpenRouter now passes modelId to enable normalization for Mistral models".
- vLLM docs: "Mistral's `tokenizer_config.json` chat template requires tool call IDs that are exactly 9 digits, which is much shorter than what vLLM generates." Its example template works "provided `tool_call_id` fields are truncated to the last 9 digits".
- LiteLLM pull request 22318 (third-party description, read as a search excerpt): "Adds `_normalize_tool_call_ids()` to `OpenAIGPTConfig`, called from `transform_request()`" and "Builds a **stable deterministic mapping** (MD5 hash, truncated to 9 chars) for each offending ID". This function was not checked in the pinned LiteLLM clone, which covers the per-vendor folders only.
- Source: OV-s80, OV-s82, OV-s79, OV-s77, OV-s83, OV-s93, OV-s66. Basis: Filled (opencode, pi, Roo Code, vLLM).
- What the fills show: two harnesses rewrite ids on the client side keyed on the model name, even when the request goes through OpenRouter. A deterministic rewrite needs no stored state.

### Moonshot Kimi

**OV39. The K3 series always reasons, takes `reasoning_effort` in place of `thinking`, and fixes its sampling parameters.**

- Quote: "**`kimi-k3.1-pro`**: the flagship thinking model; reasoning and Preserved Thinking are always on, and `reasoning_content` may be returned. Configure its reasoning effort with the top-level `reasoning_effort` request field, which supports `"low"` / `"high"` / `"max"` (default `"max"`)."
- Parameter table: `thinking` is "Not supported" on the K3 series, "Optional. Thinking and Preserved Thinking are always on." on `kimi-k2.7-code`, and on `kimi-k2.6` "Optional. Thinking is on by default; reasoning from previous turns is not preserved by default."
- Parameter table: `tool_choice` is `"auto"` / `"none"` / `"required"` on K3, and on `kimi-k2.7-code` and `kimi-k2.6` "Passing `"required"` returns an error."
- Parameter table: `temperature` "Fixed at `1.0`", `top_p` "Fixed at `0.95`", `n` "Fixed at `1`", penalties "Fixed at `0`". Quote: ""Fixed" means the parameter cannot be modified: passing any other value returns an error, so do not pass it explicitly."
- Quote: "Switching levels invalidates prefix-cache hits. Decide on the `effort` level before the conversation starts and avoid switching it mid-session."
- Source: OV-s38, OV-s40. Basis: Vendor-documented.
- Note: the docs host moved. `https://platform.moonshot.ai/docs/llms.txt` now redirects to `https://platform.kimi.ai/docs/llms.txt`. The API host in examples is still `https://api.moonshot.ai/v1`.

**OV40. Kimi's replay rule.**

- Quote: "In both single-task tool-call loops and multi-turn conversations, pass the complete assistant message back as-is, including `reasoning_content`. `kimi-k3.1-pro` and `kimi-k3` always enable Preserved Thinking."
- Quote: "For K3.1 Pro and K3, multi-turn conversations and tool calls must pass the complete assistant message returned by the API back in `messages` as-is."
- K2.6 and K2.7 table: `thinking.keep` on `kimi-k2.6` "Defaults to `null`, which discards `reasoning_content` from previous turns. Pass `"all"` to enable Preserved Thinking."; on `kimi-k2.7-code` "The model preserves `reasoning_content` from previous turns whether you omit this field, pass `null`, or pass `"all"`. Other values return an error."
- The pages read today do not print an error string for a missing `reasoning_content`.
- Fill from LiteLLM: "Moonshot reasoning models require `reasoning_content` on every assistant message that contains tool_calls (multi-turn tool-calling flows)." It injects `patched["reasoning_content"] = " "` and logs "Injecting a placeholder to satisfy API validation."
- Source: OV-s38, OV-s39, OV-s74. Basis: Vendor-documented for the rule; the validation claim is Filled (LiteLLM). Also in workspace: the 2026-09-20 report ("documented as required; K2.x degrades silently").

**OV41. Kimi tool-call ids: the hosted API shows a different format from the open-weight guide.**

- Hosted API reference example: `"id": "get_weather_0_4d9e2559"` and "The `tool_call_id` must match the tool-call `id` in the assistant message".
- Open-weight guide for Kimi-K2-Instruct on Hugging Face: "The format of the tool ID is `functions.{func_name}:{idx}`, from which we can parse the function name."
- The string `functions.` appears in none of the 14 hosted-API pages fetched today.
- Source: OV-s39, OV-s43. Basis: Vendor-documented. Also in workspace: the 2026-09-20 report gives `functions.name:idx` as the rule for "Kimi K3 / k2.7-code"; see Conflicts.

**OV42. Kimi's official tools run through a separate Formula API that the caller executes.**

- Quote: "Using official tools through the Formula API follows the standard `function` tool flow of the OpenAI protocol, in 4 steps: 1. `GET /v1/formulas/{uri}/tools`" (fetch the declarations), "2. `POST /v1/chat/completions`", "3. `POST /v1/formulas/{uri}/fibers`" (execute what `tool_calls` specifies), "4. `POST /v1/chat/completions`".
- Quote: "When using official tools such as web search with `kimi-k3.1-pro`, use the Formula API official tools channel described on this page (OpenAI protocol, standard `function` tool)".
- Tools listed: `convert`, `web-search`, `rethink`, `random-choice`, `mew`, `memory`, `excel`, `date`, `base64`, `fetch`.
- The docs index describes the Responses API as able to "use server-side web search", and lists separate `/v1/tools/search`, `/v1/tools/search_pro`, and `/v1/tools/fetch` endpoints.
- The strings `$web_search` and `builtin_function` appear in none of the pages fetched today.
- Source: OV-s41, OV-s44. Basis: Vendor-documented. Conflicts with the task's premise of a `$web_search` built-in; see Conflicts.

**OV43. Kimi partial mode.**

- Quote: "To use Partial Mode, append a message with `role=assistant` and `partial=True` at the end of the `messages` list, and place the text you want the model to continue from in the `content` field".
- Sample comment for thinking models: `{"role": "assistant", "content": prefix, "partial": True, "reasoning_content": reasoning_content} # Thinking mode requires reasoning_content`.
- Quote: "Do not mix Partial Mode with `response_format={"type": "json_object"}`, as this may lead to unexpected model responses."
- Source: OV-s42, OV-s39. Basis: Vendor-documented.

**OV44. Kimi cache options and usage fields.**

- Request example: `"prompt_cache_options": { "mode": "implicit", "ttl": "1h" }`.
- Quote: "Explicit cache breakpoints are not supported: the request is rejected (HTTP 400) if `prompt_cache_breakpoint` appears in `content`."
- Quote: "`cached_tokens`, `cache_write_tokens`, and the uncached remainder are mutually exclusive and sum to `prompt_tokens`".
- Quote: "The final usage chunk has `choices: []` and reports totals in the top-level `usage`."
- Source: OV-s39. Basis: Vendor-documented.

**OV45. How OpenRouter handles Kimi.**

- Documented: caching "automated" (OV9); the catalog lists `kimi-k3` with efforts `max, high, low, none` and `mandatory: false`, and `kimi-k2.7-code` with `mandatory: true` (OV4); Moonshot AI is a first-party provider with `supports_tool_choice` all true (OV7, OV8); Moonshot is not a native search provider (OV10).
- Fill from opencode's source comment: "`reasoning_details` carries two dialects. OpenRouter's `reasoning.*` entries must be replayed unmodified (`index` included), so they keep every field they arrived with. Kimi's OpenAI-compatible surface streams preserved thinking as bare `summary` / `encrypted` entries keyed by a stream-only `index`; Kimi does not document this publicly, so the handling follows Kimi Code (Kimi's own client): merge summary deltas by `index`, replay without `index`, and always send `reasoning_content` alongside."
- opencode code: "Kimi always expects `reasoning_content` on replayed assistant messages, even when thinking arrived only through structured details."
- Source: OV-s2, OV-s11, OV-s12, OV-s80. Basis: Filled (opencode) for the Kimi `reasoning_details` dialect; the first bullet is OR-documented and listed here for context. Confidence: medium. The Kimi dialect claim is opencode's observation, and Kimi's pages read today mention only `reasoning_content`.

### Alibaba Qwen (Model Studio, compatible mode)

**OV46. `enable_thinking`, thinking-only models, and streaming-only models.**

- Quote: "Mixed Thinking Mode: Thinking can be enabled or disabled per request, and the control parameters vary by model. The following example uses enable_thinking".
- Quote: "Thinking-only mode: The model always thinks before replying and cannot be disabled."
- Quote: "The commercial Deep Think model also supports non-streaming (synchronous) Output." and "some models (such as qwen3.5-122b-a10b, qwen3.5-27b and other open-source versions) only support Stream Output, and non-streaming calling will report an error."
- API reference: "This parameter is not an OpenAI standard parameter. When calling via the Python SDK, put it in the extra_body object." and "Simply place enable_thinking at the top level of the request body (body) like parameters such as model and messages, for example "enable_thinking": true."
- The same reference page covers resold models: `enable_thinking` applies to "the DeepSeek-V4.1-Flash, DeepSeek-V4-Pro/V4-Flash series (Alibaba Cloud direct)", "Kimi-K2.7-code (thinking-only model)", and "GLM series".
- Page stamp: "Last Updated:Nov 16, 2026" (thinking guide) and "Last Updated:Nov 20, 2026" (API reference).
- Source: OV-s45, OV-s46. Basis: Vendor-documented.

**OV47. `thinking_budget` and `reasoning_effort` on Qwen.**

- Quote: "Use the thinking_budget parameter to set the maximum number of tokens for the reasoning process. Once the limit is reached, the model immediately starts generating the response. kimi-k3.1-pro and kimi-k3 do not support this parameter".
- `qwen4-max`: "Default value: reasoning_effort=high." Mapping: `minimal`, `low` to `low`; `medium`, `high` to `high`; `xhigh`, `max` to `xhigh`; `none` "Disables thinking".
- Quote: "qwen4-max does not support setting both reasoning_effort and thinking_budget. thinking_budget must be a positive integer and cannot be set to 0."
- Budget to level table: 1 to 4096 gives `low`, 4097 to 131072 gives `high`, 131073 to 262144 gives `xhigh`. Quote: "When neither is set, high and 131072 are used."
- Source: OV-s45, OV-s46. Basis: Vendor-documented.

**OV48. `preserve_thinking` and where it defaults to on.**

- Quote: "In Multi-turn, preserve_thinking controls whether the Model reads historical assistant message reasoning_content. When set to true, these thinking contents will be appended to the next-round input."
- API reference: "preserve_thinking boolean (Optional) Default value: false" and "qwen4-max: default value true."
- Quote: "Important: When using qwen4-max, qwen3.8-max/qwen3.8-flash, preserve_thinking defaults to true, and you must pass back all reasoning_content from the history conversation in full. Concatenating reasoning_content into the content field and passing it back is not supported."
- Quote: "if the history messages do not contain reasoning_content, enabling this parameter will not return an error."
- Quote: "After it is enabled, the reasoning_content in the history conversation counts toward the input token count and is billed."
- Source: OV-s45, OV-s46. Basis: Vendor-documented. Also in workspace: the 2026-09-20 report lists `preserve_thinking` as a retain switch with quality loss otherwise; the default-on models are new relative to it.

**OV49. `qwen4-max` can return a reasoning summary plus ciphertext that must be passed back unchanged.**

- Quote: "thinking_display string (Optional) qwen4-max: "summarized". Chain-of-thought summary and encryption. By default, the full chain-of-thought is returned. After this feature is enabled, the original chain-of-thought is no longer returned in plaintext; instead, a summary and the full ciphertext are returned. The summary is for display, and the ciphertext is for restoring historical thinking."
- Field `encrypted_content` on the assistant message: "qwen4-max, thinking_display="summarized": The enabling parameter must be passed in every turn. The client can only pass the ciphertext back as-is; it cannot read, truncate, re-encode, modify, or concatenate ciphertext from multiple turns. The summary cannot replace the historical chain-of-thought."
- Streaming: "The ciphertext is returned in an independent data frame and must not be concatenated with other frames or multi-turn ciphertext."
- Source: OV-s46. Basis: Vendor-documented. New relative to both workspace reports.

**OV50. Qwen `tool_choice` limits.**

- Quote: "Qwen series models do not currently support required: in non-thinking mode, tool calling cannot be guaranteed, and in thinking mode, required is not currently supported."
- Quote: "Models in thinking mode do not support forcing a specific tool to be called."
- On resold models in the same reference: "MiniMax/MiniMax-M3.1-Flash only supports auto." and "deepseek-v4.1-flash-1017 supports setting tool_choice to auto, none, required, or a specified function name."
- Source: OV-s46. Basis: Vendor-documented.

**OV51. Qwen caching: explicit markers in Anthropic syntax, or implicit.**

- Quote: "Explicit cache: A mode that you enable manually. You create a cache for specific content to ensure a deterministic hit within its 5-minute validity period. Tokens used to create the cache are typically billed at 125% of the standard input token price, while subsequent cache hits are typically billed at only 10% of that price."
- Quote: "Implicit cache: This automatic mode requires no extra configuration and cannot be disabled" and "typically billed at 20% of the standard input token price".
- Quote: "Add a"cache_control": {"type": "ephemeral"} marker to the messages array. The system then searches backward from eachcache_control marker and examines up to 20 precedingcontent blocks to find a cache hit." (spacing as on the page)
- Quote: "A cache block contains at least 1,024 tokens." and "Explicit cache and implicit cache are mutually exclusive."
- Usage in the sample: `usage.prompt_tokens_details.cache_creation_input_tokens` and `usage.prompt_tokens_details.cached_tokens`.
- Quote: "Responses API has separate Session cache settings."
- Source: OV-s47. Basis: Vendor-documented.

**OV52. Qwen built-in search differs per route.**

- Chat completions: "Set enable_search: true to enable web search." with the note "Since enable_search is not a standard OpenAI parameter, pass it via extra_body when using the Python SDK."
- Responses API: "Add the web_search tool to the tools parameter to enable web search." and "For web search with models such as qwen3.7-max, use the web search feature of the Responses API."
- Anthropic-compatible route: "Add the Anthropic server-side web search tool (name set to web_search) to the tools parameter to enable web search."
- Source: OV-s48. Basis: Vendor-documented.

**OV53. How OpenRouter handles Qwen.**

- Documented: `reasoning.max_tokens` maps to `thinking_budget` on "Some Alibaba Qwen thinking models" (OV3); the catalog shows `supports_max_tokens: true` for `qwen/qwen3.8-flash` (OV4); explicit caching uses `cache_control` "using the same syntax as Anthropic explicit caching" on a named list of slugs (OV9); Alibaba endpoints publish narrower `supports_tool_choice` values (OV7); Alibaba is not a native search provider (OV10).
- No statement found for `preserve_thinking`, `thinking_display`, `encrypted_content`, `enable_search`, or the streaming-only models.
- Source: OV-s1, OV-s2, OV-s11, OV-s12, OV-s7. Basis: OR-documented.

### Zhipu GLM (Z.ai)

**OV54. The `thinking` object, `clear_thinking`, and effort.**

- Quote: "Thinking is activated by default in GLM-5.3 GLM-5.3-FLASH GLM-5.2 GLM-5.1 GLM-5 GLM-4.7 series" and "(note that `GLM-5.3` and `GLM-5.3-FLASH` use forced thinking and cannot be disabled)".
- Schema `ChatThinking.clear_thinking`: "Default value is True. Controls whether to clear `reasoning_content` from previous conversation turns." With `true`: "For this request, the system ignores/removes `reasoning_content` from prior turns, and only keeps non-reasoning context (e.g., user/assistant visible text, tool calls, and tool results)."
- `reasoning_effort`: "Default is `max`, supported by `GLM-5.2` and above. For the `GLM-5.3` `GLM-5.3-FLASH` model, only the `low` / `high` / `max` levels are supported. For the `GLM-5.2` model, for compatibility with other protocols, passing `none` or `minimal` will cause the model to skip thinking; `low` and `medium` will be mapped to `high`; `xhigh` will be mapped to `max`."
- Source: OV-s49, OV-s50. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey ("`clear_thinking` defaults to `true`").

**OV55. Preserved thinking on Z.ai needs an exact replay.**

- Quote: "This capability is **enabled by default** on the **Coding Plan endpoint** and **disabled by default** on the **standard API endpoint**."
- Schema, `false` setting: "Retains `reasoning_content` from prior turns and includes it in the context sent to the model. To enable Preserved Thinking, you must forward the full, unmodified, and correctly ordered historical `reasoning_content` in `messages`. Missing, truncated, rewritten, or reordered blocks may degrade performance or prevent the feature from taking effect."
- Quote (interleaved thinking): "When using interleaved thinking with tools, **thinking blocks should be explicitly preserved and returned together with the tool results.**"
- Source: OV-s49, OV-s50. Basis: Vendor-documented.

**OV56. Z.ai `tool_choice`, built-in tool types, and finish reasons.**

- `tool_choice`: "Used to control how the model selects which function to call. This is only applicable when the tool type is function. The default value is auto, and only auto is supported."
- Tools: "Tool type, supports web_search, retrieval, function". Schema `WebSearchToolSchema` has `type: web_search` and a `web_search` object. The response has a top-level `web_search` array described as "Search results."
- `finish_reason`: "Reason for model inference termination. Can be `stop`, `tool_calls`, `length`, `sensitive`, `model_context_window_exceeded` or `network_error`."
- Source: OV-s50. Basis: Vendor-documented.

**OV57. Z.ai error shape and codes.**

- Quote: "Errors are always returned as JSON, with a top-level error object that includes a `code` and `message` value." Example: `{"error": {"code": "1214", "message": "Parameter `${field}` is invalid. Please check the documentation."}}`.
- Codes from the table: 1113 (429) "Insufficient balance or no resource package. Please recharge."; 1210 (400) "Invalid API parameter, please check the documentation."; 1261 (400) "Prompt too long"; 1301 (400) content safety; 1302 (429) "Rate limit reached for requests"; 1305 (429) "The service may be temporarily overloaded, please try again later"; 1308 to 1321 (429) plan and usage limits.
- Quote: "When using streaming (SSE) calls, if the API terminates abnormally during inference, the above error codes will not be returned. Instead, the reason for the exception will be provided in the `finish_reason` parameter of the response body."
- Source: OV-s51. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey (`{"error":{"code":"1302"}}`).

**OV58. Z.ai caching.**

- Quote: "**Automatic Cache Recognition**: Implicit caching that intelligently identifies repeated context content without manual configuration" and "Detailed display of cached token counts in response field `usage.prompt_tokens_details.cached_tokens`".
- Source: OV-s52. Basis: Vendor-documented.

**OV59. How OpenRouter handles Z.ai.**

- Documented: catalog `z-ai/glm-5.3` has `mandatory: true` and efforts `max, high, low`, matching Z.ai's page (OV4, OV54); the Z.AI endpoint shows `supports_tool_choice` auto only, matching Z.ai's page (OV7, OV56); OpenRouter sends Z.AI "a session affinity key with each request" (OV9); Z.AI is not a native search provider (OV10).
- No statement found for `clear_thinking`. `provider.options` has a `z-ai` key (OV6).
- Fill from pi: `thinkingFormat` value `"zai"`, described as "uses thinking.type".
- Source: OV-s2, OV-s11, OV-s12, OV-s10, OV-s78. Basis: OR-documented.

### MiniMax

**OV60. `reasoning_split`, the `thinking` object, effort, and ignored parameters.**

- Quote: "`reasoning_split`" "Output-format switch. When `true`, thinking content is separated into the `reasoning_content` field; when `false`, thinking stays inside `content` wrapped in `<think>` tags. `MiniMax-M3.1-Flash` does not support setting this parameter to `false`."
- Quote: "`thinking`" "Controls thinking. `type` can be `disabled` or `adaptive`; when omitted, thinking is on by default. `MiniMax-M3.1-Flash` always thinks and returns `400` if `disabled` is sent."
- Error text: `model "MiniMax-M3.1-Flash" requires adaptive thinking; thinking.type="disabled" (including reasoning.effort=none) is not allowed (2013)`.
- Quote: "`reasoning_effort`" "Thinking depth, effective for `MiniMax-M3.1-Flash` only. Accepts `low`, `medium`, `high`, `xhigh`, and `max`; higher levels think more thoroughly. Defaults to `max` when omitted. `none` is not supported."
- Quote: "Some OpenAI parameters (such as `temperature`, `top_p`, `presence_penalty`, `frequency_penalty`, `logit_bias`, etc.) will be ignored" and "The `n` parameter only supports value 1".
- Source: OV-s53. Basis: Vendor-documented. Also in workspace: the task brief names `reasoning_split`; see OV61 for what changed.

**OV61. MiniMax replay rule, and the field is `reasoning_content` in today's docs.**

- Quote: "In multi-turn function call conversations, append the complete model response (the `response_message` object) to the conversation history, including the `tool_calls` field and the thinking content that `MiniMax-M3.1-Flash` returns separately in the `reasoning_content` field, to keep the reasoning chain continuous."
- Quote: "The model's thinking is returned in the `reasoning_content` field and must be kept when passing the message back".
- Quote: "MiniMax-M3.1-Flash thinks on demand: for simple tool calls, the model may call the tool directly without producing thinking content, in which case that turn has no `thinking` block or `reasoning_content` field."
- The string `reasoning_details` appears in none of the five MiniMax pages fetched today (OpenAI SDK page, chat completion reference, tool use guide, text generation guide, Responses reference).
- Source: OV-s53, OV-s55, OV-s54. Basis: Vendor-documented. Conflicts with the task's premise that MiniMax has "its own `reasoning_details`"; see Conflicts.

**OV62. MiniMax's response schema has a `base_resp` status object, even on the OpenAI-compatible route.**

- Schema: `base_resp` "Error status code and details", with `status_code` values "`1000`: Unknown error", "`1001`: Request timeout", "`1002`: Rate limit triggered", "`1004`: Authentication failed", "`1008`: Insufficient balance", "`1013`: Internal server error", "`1027`: Output content error", "`1039`: Token limit exceeded", "`2013`: Parameter error", and `status_msg` "Error details". Examples show `base_resp: status_code: 0, status_msg: ''` on successful responses.
- The same docs say M3.1-Flash "returns `400`" for a disallowed `thinking` value, so parameter errors on this route use an HTTP error status.
- Source: OV-s54, OV-s56, OV-s53. Basis: Vendor-documented.
- Not re-confirmed: the workspace statement that "the native route returns HTTP 200 with `base_resp.status_code`". The native text route's page is absent from today's docs index, which lists OpenAI, Anthropic, and Responses compatible routes. See Unverified.

**OV63. How OpenRouter reaches MiniMax models.**

- The three MiniMax model endpoint lists fetched today name third-party hosts only: `minimax/minimax-m3` (CoreWeave, DeepInfra, Featherless, Parasail, Venice), `minimax/minimax-m3.1-flash` (12 hosts, among them Fireworks, Together, SambaNova, and "Inferact vLLM"), `minimax/minimax-m2.7` (Groq). OV8.
- Catalog: `minimax/minimax-m3` `{"mandatory": false}`, `minimax/minimax-m2.5` `{"mandatory": true}`, with no `supported_efforts` list (OV4).
- OpenRouter's Messages page names MiniMax as receiving `reasoning_effort` "in its own vocabulary" (OV3).
- Source: OV-s12, OV-s11, OV-s1. Basis: OR-documented.
- Note: on third-party hosts the reasoning field and thinking switches are whatever that host's server offers (vLLM, SGLang, or the host's own stack), so MiniMax's own `reasoning_split` and `thinking` fields do not apply there.

### Meta

**OV64. Meta accepts only `tool_choice: "auto"`.**

- Quote: "**`tool_choice` must be `"auto"`**: Only `"auto"` (the default) is supported on both Chat Completions and the" Responses API; "`"none"`, `"required"`, and named function choices return `HTTP 400` ("only `"auto"` is supported for `tool_choice`"). With `"auto"`, the model decides whether to call a tool, so your app should handle both a tool call and a plain text response."
- Quote: "**Use `tools`, not deprecated fields**: `functions` and `function_call` are deprecated."
- Source: OV-s57. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey. Confirmed live.

**OV65. Meta's reasoning is private, cannot be switched off, and crosses turns only on the Responses and Messages routes.**

- Quote: "That reasoning stays private and does not appear in the response body."
- Effort table: `"none"` "Disables reasoning. **Not supported by Muse Wonder, Muse Spark, or Muse Ember**: returns `HTTP 400`."; `"max"` "Supported by `muse-wonder-1.0`, `muse-spark-1.3`, and `muse-ember-1.0`; not available with `muse-spark-1.3-contributor`."
- Quote: "On Chat Completions, use the top-level `reasoning_effort` parameter. On the Responses API, nest it as `reasoning.effort`." (link markup removed)
- Quote: "For stateless replay, where you manage history yourself instead of chaining response IDs, request the model's encrypted reasoning with `include: ["reasoning.encrypted_content"]` and replay it as a `reasoning` input item on the next turn."
- Protocols table, column "Reasoning across turns": Chat Completions "No (reasoning is not carried between turns)"; Messages: reasoning replay through Anthropic `thinking`.
- Quote: "It does not carry reasoning across turns for external keys, and search grounding and tool search run only on the Responses and Messages APIs." (link markup removed)
- Quote: "Muse Spark is a reasoning model, so `logprobs` is not supported. `logprobs: true` (Chat Completions) and `include: ["message.output_text.logprobs"]` (Responses API) both return `HTTP 400`."
- Source: OV-s58, OV-s59. Basis: Vendor-documented. Also in workspace: the 2026-10-28 survey ("Encrypted items", "three dialects").

**OV66. How OpenRouter exposes Meta.**

- Documented: `reasoning_details` format values `meta-responses-v1` and `google-vertex-meta-responses-v1` (OV2); catalog `meta/muse-spark-1.3` has `mandatory: true` and six efforts, default `medium` (OV4), while Meta's page says the default on Muse Spark is `"high"`; the Meta endpoint has `native_tools` for web search (OV10).
- The endpoint list for `meta/muse-spark-1.3` shows the Meta provider (tag `meta`) with `supports_tool_choice` `{"none": true, "auto": true, "required": true, "function": true}`, and `tool_choice` in its `supported_parameters`.
- Inference: as with xAI, the format name and the encrypted reasoning point to OpenRouter calling Meta's Responses route upstream.
- Source: OV-s1, OV-s11, OV-s12. Basis: OR-documented for the data; the upstream route is OR-inferred (confidence medium). The `supports_tool_choice` value conflicts with OV64; see Conflicts.

### Local servers behind the Runvil Gateway

All llama.cpp items are from `ggml-org/llama.cpp` at commit `8d60873dfb8619f2b5ed31adfbab07cccb40db5c` (committed 2026-11-22). vLLM items are from `vllm-project/vllm` at commit `e2052f5c7ad64564f00dd4592e77c27785c9333f` (committed 2026-11-23 +08:00). Ollama source is from `ollama/ollama` at commit `318ee284aa3b5558de57efefff730ea33c4a9bef`. Each is "Filled": it shows how that server defines the thing, and it says nothing about OpenRouter.

**OV67. llama.cpp returns reasoning in `reasoning_content` and has four ways to control it.**

- README, server flag `reasoning-format`: "controls whether thought tags are allowed and/or extracted from the response, and in which format they're returned; one of: - none: leaves thoughts unparsed in `message.content` - deepseek: puts thoughts in `message.reasoning_content` - deepseek-legacy: keeps `<think>` tags in `message.content` while also populating `message.reasoning_content` (default: auto)" (line 240, `<br/>` tags removed).
- Server flags also listed: `reasoning` "Use reasoning/thinking in the chat ('on', 'off', or 'auto', default: 'auto' (detect from template))"; `reasoning-effort` "reasoning effort level given to the chat template"; `reasoning-budget` "token budget for thinking: -1 for unrestricted, 0 for immediate end, N>0 for token budget (default: -1)" (lines 241 to 243).
- Request fields (lines 1468 to 1472): "`chat_template_kwargs`: Allows sending additional parameters to the json templating system. For example: `{"enable_thinking": false}`"; "`reasoning_effort`: If `none`, reasoning/thinking is disabled. Otherwise, the value is made available to the jinja template."; "`reasoning_format`: The reasoning format to be parsed. If set to `none`, it will output the raw generated text."
- Lines 1588 to 1590: "The server supports parsing and returning reasoning via the `reasoning_content` field, similar to Deepseek API." and "Reasoning input (preserve reasoning in history) is also supported by some specific templates. For more details, please refer to PR#18994." (link markup removed)
- Source: OV-s60. Basis: Filled (llama.cpp). Also in workspace: the 2026-09-20 report, Appendix C and section 2.4.

**OV68. llama.cpp publishes template capabilities through `/props`.**

- README line 1043: "`chat_template_caps` - capabilities of the chat template (see `common/jinja/caps.h` for more info), `reasoning_effort_levels` lists the `reasoning_effort` values the template renders distinctly, from lowest to highest, and `reasoning_effort_rejected` the ones it raises on, which are answered with a 400".
- `common/jinja/caps.h` fields: `supports_tools` (line 12), `supports_tool_calls` (line 13), `supports_parallel_tool_calls` (line 15), `supports_preserve_reasoning` (line 18), `supports_reasoning_effort` (line 21), `reasoning_effort_levels` (line 24), `reasoning_effort_rejected` (line 27).
- Source: OV-s60, OV-s61. Basis: Filled (llama.cpp). Also in workspace: Appendix C names `chat_template_caps.supports_tool_calls`. The effort-level fields are new relative to it.

**OV69. llama.cpp timings, prompt cache, and slots.**

- README lines 1544 to 1568: "The response contains a `timings` object", with `"cache_n": 236, // number of prompt tokens reused from cache`, `"prompt_n": 1, // number of prompt tokens being processed`, `"predicted_n": 35, // number of predicted tokens`, and "The total number of tokens in context is equal to `prompt_n + cache_n + predicted_n`".
- Lines 1570 to 1582: "The response also includes a standard `usage` object" with `"prompt_tokens_details": {"cached_tokens": 0}`.
- Line 603: "`cache_prompt`: Re-use KV cache from a previous request if possible. This way the common prefix does not have to be re-processed, only the suffix that differs between the requests." "Default: `true`".
- Line 601: "`id_slot`: Assign the completion task to an specific slot. If is -1 the task will be assigned to a Idle slot."
- Slot routes: "POST `/slots/{id_slot}?action=save`: Save the prompt cache of the specified slot to a file." and matching `restore` and `erase` (lines 1269 to 1309). Server flags `cache-reuse` and `cache-ram` (lines 169, 227).
- Source: OV-s60. Basis: Filled (llama.cpp).

**OV70. llama.cpp grammars and the tools conflict.**

- README: "`grammar`: Set grammar for grammar-based sampling." and "`json_schema`: Set a JSON schema for grammar-based sampling" (lines 585 to 587).
- README line 1466: "The `response_format` parameter supports both plain JSON output (e.g. `{"type": "json_object"}`) and schema-constrained JSON", and "The OpenAI wrapper form `{"type": "json_schema", "json_schema": {"name": ..., "strict": ..., "schema": {...}}}` is also accepted."
- `tools/server/server-common.cpp` line 1620: `throw std::invalid_argument("Cannot use custom grammar constraints with tools.");`. `common/chat.cpp` line 1601: `throw std::runtime_error("Cannot specify grammar with tools");`.
- Source: OV-s60, OV-s62, OV-s64. Basis: Filled (llama.cpp). Also in workspace: Appendix C quotes the first string. The survey's line that "grammar and JSON schema exist only on the native route" does not match the README read today; see Conflicts.

**OV71. llama.cpp serves Anthropic Messages and OpenAI Responses routes, with empty signatures.**

- README headings: "POST `/v1/responses`: OpenAI-compatible Responses API" (line 1703), "POST `/v1/responses/input_tokens`: Token Counting" (line 1812), "POST `/v1/messages`: Anthropic-compatible Messages API" (line 1842).
- README line 1709: "`extra_params`: llama.cpp parameters that the Responses API has no field for, such as `seed`, `min_p` or `repeat_penalty`, as an object".
- `tools/server/server-task.cpp` lines 752 to 756: `{"type", "thinking"}, {"thinking", msg.reasoning_content}, {"signature", ""}  // empty signature for local models (no cryptographic verification)`.
- `tools/server/server-task.cpp` lines 921 to 922: "// Anthropic API requires a signature_delta before closing thinking blocks" and "// We use an empty signature since we can't generate a cryptographic signature for local models".
- `tools/server/server-task.cpp` line 540: `{"encrypted_content", ""},` in the Responses reasoning item.
- `tools/server/server-chat.cpp` lines 460 and 585: incoming Messages `thinking` blocks are concatenated into `reasoning_content` on the converted message.
- Source: OV-s60, OV-s63. Basis: Filled (llama.cpp).

**OV72. Silent failures in llama.cpp, re-checked for two items of Appendix C.**

- Named-function `tool_choice`: `tools/server/server-common.cpp` line 1500 reads `auto tool_choice = json_value(body, "tool_choice", std::string("auto"));`. `json_value` in `server-common.h` lines 46 to 58 catches a type error, logs `"Wrong type supplied for parameter '%s', using default value: %s\n"`, and returns the default. So an object-valued `tool_choice` becomes `"auto"` with a log line and no error to the caller. The string parser accepts `auto`, `none`, `required` and throws "Invalid tool_choice: " otherwise (`common/chat.cpp` lines 324 to 334).
- Tools sent to a template that does not describe tools natively: `common/chat.cpp` lines 1603 to 1607 only log a warning that begins "Template supports tool calls but does not natively describe tools. The fallback behaviour used may produce bad results".
- Without the Jinja engine: the server throws a runtime error saying the `tools` param requires the `jinja` flag, and a second one for a `tool_choice` other than `auto` (`server-common.cpp` lines 1504 and 1507, paraphrased).
- Not re-read today: the Mistral Nemo nine-character raise, the Llama 3.x single-call raise, `required` running to `max_tokens` (issue 27217), leaked tool markup, and bad GGUF templates. These stay on the workspace report's evidence.
- Source: OV-s62, OV-s64. Basis: Filled (llama.cpp). Also in workspace: the 2026-09-20 report, Appendix C.

**OV73. vLLM names the field `reasoning`, selects a parser per family, and maps efforts on the server.**

- Quote: "Reasoning models return an additional `reasoning` field in their outputs" and "`reasoning` used to be called `reasoning_content`. To migrate, directly replace `reasoning_content` with `reasoning`. It is important that you also update your client code. Otherwise, your client code could silently read an empty `reasoning_content`, even when `reasoning` is populated."
- Parser table, examples: DeepSeek R1 `deepseek_r1`, DeepSeek-V3.1 `deepseek_v3`, GLM-4.5 `glm45`, MiniMax-M2 `minimax_m2_append_think`, Qwen3 `qwen3`, Gemma 4 `gemma4`.
- Quote: "The reasoning feature for the Qwen3 series is enabled by default. To disable it, you must pass `enable_thinking=False` in your `chat_template_kwargs`." and "IBM Granite 3.2 and DeepSeek-V3.1 reasoning is disabled by default; to enable it, you must also pass `thinking=True` in your `chat_template_kwargs`."
- Quote: "The map applies to the effort set through the Chat Completions (`reasoning_effort`), Responses (`reasoning.effort`) and Anthropic Messages (`output_config.effort`) APIs."
- Quote: "Once the reasoning token count reaches the configured `thinking_token_budget`, vLLM forces the model to produce `reasoning_end_str`, effectively terminating the reasoning block."
- Quote: "tool calling only parses functions from the `content` field, not from the `reasoning`."
- Source: OV-s65. Basis: Filled (vLLM).

**OV74. vLLM tool calling needs server flags, and forced choices use constrained decoding.**

- Quote: "vLLM currently supports named function calling, as well as the `auto`, `required` (as of `vllm>=0.8.3`), and `none` options for the `tool_choice` field in the chat completion API."
- Paraphrase of line 88: vLLM loads the parser named by the `tool-call-parser` flag only when the `enable-auto-tool-choice` flag is also set.
- Quote: "Without a loaded parser, a request with `tool_choice='required'` returns HTTP 400."
- Quote: "Most OpenAI-compatible clients and agent frameworks never set `strict` on their tools, so with `tool_choice="auto"` the model generates tool calls without any grammar and malformed markup can leak into the response."
- Mistral: "Mistral's `tokenizer_config.json` chat template requires tool call IDs that are exactly 9 digits, which is much shorter than what vLLM generates. Since an exception is thrown when this condition is not met, the following additional chat templates are provided".
- Source: OV-s66. Basis: Filled (vLLM).

**OV75. vLLM structured output controls.**

- Quote: "If you are still using the following deprecated API fields which were removed in v0.12.0, please update your code to use `structured_outputs`", with the mapping "`guided_json` -> `{"structured_outputs": {"json": ...}}`" and the same for `regex`, `choice`, `grammar`, `whitespace_pattern`, `structural_tag`.
- Source: OV-s67. Basis: Filled (vLLM).

**OV76. vLLM serves Responses and Anthropic Messages routes, and interleaved thinking replays `reasoning`.**

- Quote: "Responses API (`/v1/responses`, `/v1/responses/{response_id}`, `/v1/responses/{response_id}/cancel`)" and "Anthropic messages API (`/v1/messages`, `/v1/messages/count_tokens`)" with the sub-point "`max_tokens=0` (Anthropic's prompt-cache pre-warming request) is rejected".
- Usage, with the `enable-prompt-tokens-details` flag: "`/v1/chat/completions/batch` sums `cached_tokens`, `created_cache_tokens` and per-modality `multimodal_tokens` across conversations in `usage.prompt_tokens_details`".
- Interleaved thinking page: models `moonshotai/Kimi-K2-Thinking` (parser `kimi_k2`) and `MiniMaxAI/MiniMax-M2` (parser `minimax_m2`); the sample appends `"reasoning": response.choices[0].message.reasoning, # append reasoning` to the assistant message.
- Source: OV-s68, OV-s69. Basis: Filled (vLLM). Also in workspace: the 2026-10-28 survey ("Responses including retrieve and cancel").

**OV77. Ollama's OpenAI-compatible surface.**

- `/v1/chat/completions` supported request fields include "[x] `reasoning_effort` (model-defined names and compatibility aliases)" and "[x] `reasoning`", and unchecked "[ ] `tool_choice`", "[ ] `logit_bias`", "[ ] `user`", "[ ] `n`".
- Quote: "Boolean-only models that support enabling thinking map recognized compatibility efforts to `true`, while `"none"` requests `false`."
- Quote: "Ollama supports the OpenAI Responses API. Only the non-stateful flavor is supported (i.e., there is no `previous_response_id` or `conversation` support)." (link markup removed)
- Native API: "Chat returns `message.thinking` and `message.content`." and "Use `/api/show` to discover the values a model supports and the value Ollama uses by default".
- Source code, `openai/openai.go` lines 36 and 47: the message and delta structs have a `Reasoning` string field with the JSON tag `json:"reasoning,omitempty"`.
- Source: OV-s70, OV-s71, OV-s72. Basis: Filled (Ollama). Also in workspace: the 2026-10-28 survey ("rejecting tool choice, logit bias, `user`, `n` and image URLs").

**OV78. pi's compatibility schema lists the per-server differences a client has to bridge.**

- `thinkingFormat` description: "Format for reasoning or thinking parameters. When omitted, Pi auto-detects the format from the provider URL. "openai" uses reasoning_effort, "openrouter" uses reasoning.effort, "deepseek" uses thinking.type plus reasoning_effort when supported, "together" uses reasoning.enabled plus reasoning_effort when supported, "baseten" uses configurable chat_template_args plus reasoning_effort when supported, "zai" uses thinking.type, "qwen" uses top-level enable_thinking, "qwen-chat-template" uses chat_template_kwargs.enable_thinking and preserve_thinking, "chat-template" uses configurable chat_template_kwargs, "string-thinking" uses top-level thinking, and "ant-ling" uses reasoning.effort only when the mapped effort is non-null."
- Budget field names: "Top-level request field name used by OpenAI-compatible endpoints to cap reasoning tokens. "thinking_token_budget" is used by vLLM, "thinking_budget" by Qwen, DashScope, or SGLang, and "thinking_budget_tokens" by llama.cpp."
- Reading reasoning (`openai-completions.ts` line 584): `const reasoningFields = ["reasoning_content", "reasoning", "reasoning_text"];` with the comment "Some endpoints return reasoning in reasoning_content (llama.cpp)".
- Reading cache usage (lines 1529 to 1536): `rawUsage.prompt_tokens_details?.cached_tokens ?? rawUsage.prompt_cache_hit_tokens ?? rawUsage.cached_tokens ?? 0` with the comment "Providers disagree on placement: OpenAI/OpenRouter use prompt_tokens_details.cached_tokens, DeepSeek uses prompt_cache_hit_tokens, and Kimi documents top-level usage.cached_tokens on the final usage chunk."
- Default flags by URL (lines 1646 to 1670): `supportsReasoningEffort: !isGrok && !isZai && !isMoonshot && !isTogether && !isCloudflareAiGateway && !isNvidia && !isAntLing`, `maxTokensField: useMaxTokens ? "max_tokens" : "max_completion_tokens"`, `supportsDeveloperRole: isOpenAI`.
- Source: OV-s78, OV-s77. Basis: Filled (pi), pinned at commit `9c4652688590abb836b40ebd1e4725ecf9a7c7d1`.
- What the fill shows: one client keeps eleven named thinking formats and three budget field names for endpoints that all call themselves OpenAI-compatible, and selects them by provider URL.

**OV79. OpenRouter lists local-server stacks as providers.**

- The spec's `ProviderOptions` has the keys `ollama` and `inferact-vllm` (OV6). The changelog records "`ProviderOptions`: new property `ollama`".
- The endpoint list for `minimax/minimax-m3.1-flash` includes provider "Inferact vLLM" with tag `inferact-vllm/nvfp4`.
- Source: OV-s10, OV-s13, OV-s12. Basis: OR-documented.

### Runvil's own records

**OV80. What Runvil's provider files record for each vendor.**

- `xai.rs`: base `https://api.x.ai`, OpenAI base `https://api.x.ai/v1`; reads `context_length` and prices from `GET /v1/models`. Fixture ids (2026-09-14): `grok-4.20-0309-reasoning`, `grok-4.3`, `grok-4.5`, `grok-4.6`, `grok-build-0.1`.
- `deepseek.rs`: OpenAI base `https://api.deepseek.com/v1`, list at `/models`, "IDs-only" shape. Fixture ids: `deepseek-flash`, `deepseek-v4-pro`.
- `mistral.rs`: OpenAI base `https://api.mistral.ai/v1`; reads `capabilities.function_calling`, `capabilities.vision`, `max_context_length`, deprecation fields.
- `moonshot.rs`: `https://api.moonshot.ai/v1`; reads `context_length`, `supports_image_input`, `supports_video_input`, `supports_reasoning`. Fixture ids: `kimi-k2.6`, `kimi-k2.7-code`.
- `qwen.rs`: `https://dashscope-intl.aliyuncs.com/compatible-mode/v1`, key `DASHSCOPE_API_KEY`, "IDs-only" list.
- `minimax.rs`: `https://api.minimax.io/v1`, "IDs-only" list, test ids `MiniMax-M3`, `MiniMax-M2.7`.
- `meta.rs`: `https://api.meta.ai/v1`, "IDs-only" list. Fixture ids: `muse-spark-1.1` to `muse-spark-1.3`, `muse-spark-1.3-contributor`.
- The seven files record model-list facts only: ids, context windows, capability booleans, prices, deprecation dates. Replay rules, tool-call id rules, `tool_choice` limits, and thinking field names live in the research reports, not in this code. `moonshot.rs` is the only one that reads a reasoning capability flag (`supports_reasoning`).
- Source: OV-s92. Basis: Workspace (`runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/`, read 2026-11-22).

**OV81. Status of the workspace reports' rows for these vendors after the live re-check.**

- Confirmed live: DeepSeek replay with tools and the 400 (OV22, OV23); DeepSeek `tool_choice` in thinking mode (OV24); Mistral nine-character ids and code 3280 (OV30); Mistral delta shape change (OV31); Mistral 64-token cache blocks (OV36); Mistral error shapes (OV34); xAI Responses primary, effort ladder, 1,000-tool cap, ignored fields, routing hint (OV13 to OV19); Meta `tool_choice` auto only (OV64); Z.ai `clear_thinking` default (OV54); Z.ai quoted integer code (OV57); Ollama lacking `tool_choice` (OV77); vLLM Responses route (OV76).
- Changed or sharpened: see Conflicts (Kimi id format on the hosted API, Kimi built-in tools, MiniMax field name, `safe_prompt` deprecation, Mistral `prompt_cache_key`, Mistral's second endpoint, Qwen default-on `preserve_thinking`, llama.cpp grammar on the chat route).
- Source: OV-s90, OV-s91. Basis: Workspace, re-checked as stated.

## Conflicts and disagreements

Conflicts between OpenRouter's published data and the vendors' pages (both read today, not resolved by a live call):

1. **Meta `tool_choice`.** Meta's page says only `"auto"` is supported and other values "return `HTTP 400`" (OV64). OpenRouter's endpoint list shows the Meta provider with `none`, `auto`, `required`, and `function` all true (OV66). Either OpenRouter converts the other values before calling Meta, or the catalog flag is wrong. No document settles it.
2. **Mistral Large 4 effort values.** Mistral: `low`, `medium`, `high`, and it "does not accept `none`" (OV32). OpenRouter catalog: `supported_efforts` `["high", "none"]`, `mandatory: false` (OV4).
3. **Kimi K3 off switch.** Kimi: reasoning is "always on" and `thinking` is "Not supported" (OV39). OpenRouter catalog for `moonshotai/kimi-k3`: `mandatory: false` with `none` among the efforts (OV4). A possible reason, from Alibaba's page: on Alibaba's deployment of `kimi-k3.1-pro` "You can set it to false to disable thinking" (OV-s46). The slug has 29 endpoints across 20 hosts, so the catalog value may describe some hosts and not Moonshot's own.
4. **DeepSeek effort names.** DeepSeek: `none`, `low`, `high`, `max` (OV21). OpenRouter catalog: `xhigh, high, none` for `deepseek-v4-pro` and `max, high, low, none` for `deepseek-v4.1-flash` (OV4).
5. **Meta default effort.** Meta: the default is `"high"` "on every Muse Spark version" (OV-s58). OpenRouter catalog: `default_effort: "medium"` for `meta/muse-spark-1.3` (OV4).
6. **OpenRouter's native search list and its endpoint data.** The prose list of native search providers leaves Meta out; the Meta endpoint has `native_tools` for web search (OV10).
7. **DeepSeek as a provider.** The spec has a `deepseek` key in `ProviderOptions`, the stream-cancellation list names DeepSeek, and a third-party post from 2026-05-29 pins `provider.order` to `["deepseek"]`. The four DeepSeek model endpoint lists fetched today name only other hosts as providers (OV8).

Conflicts inside one vendor's own documentation:

8. **xAI reasoning on Chat Completions.** The comparison page says "No reasoning content returned" for Chat Completions (OV13). The Chat Completions reference documents a response field `reasoning_content` described as "The reasoning trace generated by the model." (OV-s21).
9. **Z.ai error code type.** The Errors page example has `"code": "1214"`, a string. The chat-completion schema defines `Error.code` as `type: integer`, `format: int32` (OV57, OV-s50).

Conflicts between third-party reports:

10. **Did OpenRouter fix DeepSeek replay?** An opencode maintainer wrote on 2026-04-27 that "their change rolled out to their api" and the error stopped. A Reddit post on 2026-05-29 reports the same 400 with the provider pinned to `deepseek` (OV28).

Where this file's findings differ from the local Runvil documents and the task brief:

11. **Kimi tool-call id format (corrects `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, Table 2 and Table B1).** The report says ids for "Kimi K3 / k2.7-code" "must be `functions.name:idx`". Today the hosted API reference shows `get_weather_0_4d9e2559` and requires only that `tool_call_id` match (OV41). The `functions.{func_name}:{idx}` format is documented for the open-weight Kimi-K2-Instruct template. The live pages are right for the hosted API; the workspace rule still describes the open-weight template, which matters for local servers and third-party hosts (Appendix D of the same report says so correctly).
12. **Kimi `$web_search` (corrects the task brief's premise).** Today's docs describe official tools through the Formula API and server-side web search on the Responses route. `$web_search` and `builtin_function` do not appear (OV42). Whether the API still accepts the old built-in is unverified.
13. **MiniMax `reasoning_details` (corrects the task brief's premise).** Today's docs name `reasoning_content` as the separate field under `reasoning_split: true`. `reasoning_details` is absent from the five pages read (OV61).
14. **Mistral caching (updates the 2026-10-28 survey, Table 5).** The survey lists Mistral under "Automatic, no control". Today's docs add `prompt_cache_key` (OV36).
15. **Mistral's OpenAI path (updates the survey, Table 2, "OpenAI path: Native").** Mistral now describes `/v1/chat/completions` as "the native Mistral format" and offers a separate OpenAI-compatible endpoint in public preview (OV35). Runvil's `mistral.rs` records `/v1` as the OpenAI base URL (OV80).
16. **`safe_prompt` (updates the survey).** Mistral marks it deprecated (OV33). OpenRouter's parameters page still names it as a pass-through example (OV5).
17. **llama.cpp grammar and schema (corrects the survey's baseline paragraph).** The survey says "grammar and JSON schema exist only on the native route". The README read today documents schema-constrained `response_format` on `/v1/chat/completions`, and the chat route's code handles a client `grammar` (it errors only when tools are also present) (OV70).
18. **Qwen `preserve_thinking` (sharpens the 2026-09-20 report).** The report treats it as an opt-in retain switch. On `qwen4-max`, `qwen3.8-max`, and `qwen3.8-flash` it now defaults to `true` and full replay is required (OV48). `qwen4-max` adds ciphertext replay (OV49).
19. **Mistral reasoning replay (sharpens the 2026-09-20 report, Table 2: "Mistral: none documented").** Mistral now documents that the full assistant message including `ThinkChunk` should be replayed, with quality loss otherwise, and the schema has a `signature` field (OV31).
20. **OpenRouter provider-integration page (differs from the task brief's expectation).** The brief expected the page to state what OpenRouter requires of providers on field names such as `reasoning_content` versus `reasoning`, tool-call format, usage, and finish reasons. The page as mirrored today, and the legacy page fetched live, state none of these (OV11).

Survey claims from section 8 of the brief that touch this topic: LC3 (unsupported parameters ignored) holds, with the nuance that `tools`, the response format, and `verbosity` act as a soft routing preference (OV5). LC6 (`reasoning_details` with a vendor `format` tag, replay contract) holds; this file adds that only xAI and Meta among these vendors have their own tag (OV2). LC1 (four routes per model) is consistent with the catalog's `endpoints` arrays for every model checked.

## Open questions blocked by closed source

1. **When the caller sends no reasoning back, does OpenRouter fill `reasoning_content` for DeepSeek-family and Kimi upstreams, or pass the request and let the provider return 400?** Tried: mirror, spec, changelog, catalog, endpoint lists. Fill found: LiteLLM injects a single space for DeepSeek and for Moonshot tool-call messages; pi injects an empty string for DeepSeek's own URL; opencode requires the field for any model id containing "deepseek" on generic endpoints. Third-party reports show the 400 reaching OpenRouter callers in April and May 2026 (OV28, OV29, OV40).
2. **Which field does OpenRouter write reasoning into for each upstream (`reasoning_content`, `reasoning`, a Mistral chunk list)?** Tried: integration pages and spec (OV11). Fill found: opencode keeps a `reasoningField` per provider (`"reasoning_content"` for Moonshot, Alibaba, DeepInfra, Cohere); pi reads three names and writes back the one it saw; vLLM renamed its field (OV73, OV78).
3. **Does OpenRouter rewrite tool-call ids for Mistral-family endpoints today?** Tried: mirror, spec, changelog. Found one employee comment dated 2026-03-31 in a third-party issue. Fill found: opencode, pi, Roo Code, and vLLM each normalize to nine characters; a hash makes the rewrite stateless (OV37, OV38).
4. **Does OpenRouter route on `supports_tool_choice`, or forward `tool_choice: "required"` to an endpoint that lacks it?** Tried: provider-selection page (the soft-preference list leaves `tool_choice` out; `require_parameters` is the caller's switch). Partial fill: LiteLLM's Moonshot transformation turns `tool_choice: "required"` into an added message (`_add_tool_choice_required_message`, lines 221 to 245 at the pinned commit). No source shows OpenRouter's policy (OV7).
5. **Does OpenRouter send xAI's `x-grok-conv-id` or `prompt_cache_key`, and a comparable key to Kimi, Mistral, or DeepSeek?** Tried: prompt-caching page, which documents a session affinity key for Z.AI only. Fill found: xAI documents the header; pi sends `x-session-id` to OpenRouter and `x-session-affinity` to other hosts (OV18, OV78).
6. **Which vendor-only fields are on each endpoint's `provider.options` allowlist (`enable_search`, `clear_thinking`, `preserve_thinking`, `thinking_display`, `reasoning_split`, `prompt_mode`, `partial`, `prefix`)?** Tried: spec (open objects), integration page (allowlists exist, contents unpublished). No fill found, since this is OpenRouter configuration. OpenRouter's debug switch that returns the upstream body would show it, but that needs an authenticated call, which this research does not make (OV6).
7. **How are Mistral's `ThinkChunk.signature`, Kimi's `summary` and `encrypted` details, and Qwen's `encrypted_content` stored in `reasoning_details` and replayed?** Tried: reasoning-tokens page (the format enum names other vendors only). Partial fill: opencode's handling of the Kimi dialect. No fill found for Mistral signatures or Qwen ciphertext (OV2, OV45, OV49).
8. **Which upstream route does OpenRouter call for xAI and Meta?** Tried: docs. Inferred as Responses from the format names (OV20, OV66). Fill: LiteLLM states that xAI web search works only on the Responses route (OV17).
9. **What do third-party hosts do with replayed reasoning for open weights (Kimi, GLM, MiniMax, Qwen, DeepSeek)?** It depends on each host's server. Fill found: llama.cpp forwards `reasoning_content` to templates that support it, vLLM takes `reasoning` and documents interleaved thinking for Kimi K2 and MiniMax M2 (OV67, OV76). No per-host statement was found for OpenRouter's providers.

## Unverified

- MiniMax native route answering HTTP 200 with a nonzero `base_resp.status_code`: taken from the 2026-10-28 survey, not re-confirmed. Today's docs index lists OpenAI, Anthropic, and Responses compatible routes and the OpenAI-compatible schema includes `base_resp` (OV62).
- That MiniMax's earlier M2 documentation used a `reasoning_details` field with `reasoning_split`: from memory, no source read today.
- That Kimi's API still accepts a `$web_search` built-in function: from memory and the task brief; absent from today's docs (OV42).
- The DeepSeek 400 message string and the Mistral 3280 message string: read in LiteLLM source and third-party reports dated December 2025 to May 2026, not printed in the vendors' pages read today, and not observed first-hand because no inference call was made (OV23, OV30).
- Third-party issue reports (keys OV-s83 to OV-s89, OV-s93, OV-s94) were read as web-search result excerpts on 2026-11-22, not as full pages.
- The OpenRouter employee comment about fixing Mistral ids (2026-03-31) is one comment in a third-party issue; no OpenRouter document confirms the fix or describes it.
- Whether Mistral's preview OpenAI-compatible endpoint returns string content with reasoning in a separate field, and whether it relaxes the nine-character id rule: not stated on the pages read.
- Whether the Qwen streaming-only limit, `preserve_thinking`, or ciphertext replay behave the same through OpenRouter: no statement found.
- Appendix C items for llama.cpp other than the two re-read (OV72) rest on the 2026-09-20 report.
- Kimi's API returning a validation error (rather than degrading) when `reasoning_content` is missing on tool-call messages: stated by LiteLLM's source comment, not by Kimi's pages (OV40).
- Hosts such as Together, Fireworks, DeepInfra, Groq, and Novita were checked only through OpenRouter's endpoint lists (provider names and `supports_tool_choice`), not through their own documentation.

## Workspace documents used

- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20). Took: Table 2 (hard replay rules), Appendix B (history replay matrix), Appendix C (llama-server duties), Appendix D (open-weight tool formats), section 2.4. Re-checked live: DeepSeek, Mistral, Kimi, Qwen, GLM rows of Table 2 against vendor docs; two Appendix C duties against llama.cpp source. Not re-checked: the rest of Appendix C and Appendix D. Corrections: items 11, 18, 19 in Conflicts.
- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took: the first-party vendor paragraph and Table 2 (xAI, Meta, Mistral), the China and APAC paragraph (DeepSeek, Moonshot, Alibaba, MiniMax, Zhipu), section 5.4 on replay contracts, Table 5 on caching, section 5.7 on error envelopes, the self-hosted baseline paragraph. Re-checked live: every row named in OV81. Not re-confirmed: MiniMax HTTP 200 envelope. Corrections: items 14 to 17 in Conflicts.
- `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/` (`xai.rs`, `deepseek.rs`, `mistral.rs`, `moonshot.rs`, `qwen.rs`, `minimax.rs`, `meta.rs`) and `tests/fixtures/2026-09-14-*.json` (read 2026-11-22; fixtures dated 2026-09-14). Took: base URLs, model-list fields, and model ids per vendor (OV80). Re-checked: Mistral's base URL against Mistral's docs (item 15 in Conflicts).

## Sources

All read on 2026-11-22. OpenRouter pages were read from the local mirror fetched that day unless marked "live".

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| OV-s1 | https://openrouter.ai/docs/guides/best-practices/reasoning-tokens | 2026-11-22 | OpenRouter doc |
| OV-s2 | https://openrouter.ai/docs/guides/best-practices/prompt-caching | 2026-11-22 | OpenRouter doc |
| OV-s3 | https://openrouter.ai/docs/guides/providers/integration | 2026-11-22 | OpenRouter doc |
| OV-s4 | https://openrouter.ai/docs/guides/providers/integration-legacy.md (live, HTTP 200) | 2026-11-22 | OpenRouter doc |
| OV-s5 | https://openrouter.ai/docs/api_reference/parameters | 2026-11-22 | OpenRouter doc |
| OV-s6 | https://openrouter.ai/docs/guides/routing/provider-selection | 2026-11-22 | OpenRouter doc |
| OV-s7 | https://openrouter.ai/docs/guides/features/server-tools/web-search | 2026-11-22 | OpenRouter doc |
| OV-s8 | https://openrouter.ai/docs/guides/features/server-tools | 2026-11-22 | OpenRouter doc |
| OV-s9 | https://openrouter.ai/docs/api_reference/streaming | 2026-11-22 | OpenRouter doc |
| OV-s10 | https://openrouter.ai/openapi.json (schemas `ProviderOptions`, `ChatAssistantMessage`, `ChatToolCall`, `ReasoningEffort`) | 2026-11-22 | OpenRouter spec |
| OV-s11 | https://openrouter.ai/api/v1/models (476 models; `supported_parameters`, `reasoning`, `endpoints`) | 2026-11-22 | OpenRouter doc (public API data) |
| OV-s12 | https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints for 22 models (live, no auth header): `x-ai/grok-4.7`, `x-ai/grok-4.20`, `deepseek/deepseek-v4-pro`, `deepseek/deepseek-v4.1-flash`, `deepseek/deepseek-v3.2`, `deepseek/deepseek-chat`, `mistralai/mistral-large-4-0`, `mistralai/mistral-medium-3-5`, `mistralai/mistral-nemo`, `moonshotai/kimi-k3`, `moonshotai/kimi-k2.7-code`, `qwen/qwen4-max`, `qwen/qwen3.8-flash`, `qwen/qwen3.6-27b`, `z-ai/glm-5.3`, `z-ai/glm-4.7`, `minimax/minimax-m3`, `minimax/minimax-m3.1-flash`, `minimax/minimax-m2.7`, `meta/muse-spark-1.3`, `meta-llama/llama-3.3-70b-instruct`, `meta-llama/llama-4-maverick` | 2026-11-22 | OpenRouter doc (public API data) |
| OV-s13 | https://openrouter.ai/docs/changelog | 2026-11-22 | OpenRouter doc |
| OV-s14 | https://openrouter.ai/docs/api_reference/overview | 2026-11-22 | OpenRouter doc |
| OV-s15 | https://openrouter.ai/docs/guides/features/tool-calling | 2026-11-22 | OpenRouter doc |
| OV-s16 | https://openrouter.ai/docs/api/api-reference/endpoints/list-all-endpoints-for-a-model | 2026-11-22 | OpenRouter doc |
| OV-s17 | https://docs.x.ai/developers/model-capabilities/text/reasoning.md | 2026-11-22 | vendor doc |
| OV-s18 | https://docs.x.ai/developers/model-capabilities/text/comparison.md | 2026-11-22 | vendor doc |
| OV-s19 | https://docs.x.ai/developers/advanced-api-usage/prompt-caching/maximizing-cache-hits.md | 2026-11-22 | vendor doc |
| OV-s20 | https://docs.x.ai/developers/rest-api-reference/inference/responses.md | 2026-11-22 | vendor doc |
| OV-s21 | https://docs.x.ai/developers/rest-api-reference/inference/chat-completions.md | 2026-11-22 | vendor doc |
| OV-s22 | https://docs.x.ai/developers/advanced-api-usage/prompt-caching.md | 2026-11-22 | vendor doc |
| OV-s23 | https://docs.x.ai/developers/advanced-api-usage/prompt-caching/how-it-works.md | 2026-11-22 | vendor doc |
| OV-s24 | https://docs.x.ai/developers/advanced-api-usage/prompt-caching/usage-and-pricing.md | 2026-11-22 | vendor doc |
| OV-s25 | https://docs.x.ai/developers/model-capabilities/text/generate-text.md | 2026-11-22 | vendor doc |
| OV-s26 | https://docs.x.ai/developers/tools/web-search.md | 2026-11-22 | vendor doc |
| OV-s27 | https://docs.x.ai/developers/tools/overview.md | 2026-11-22 | vendor doc |
| OV-s28 | https://docs.x.ai/developers/model-capabilities/legacy/chat-completions.md | 2026-11-22 | vendor doc |
| OV-s29 | https://api-docs.deepseek.com/guides/thinking_mode | 2026-11-22 | vendor doc |
| OV-s30 | https://api-docs.deepseek.com/api/create-chat-completion | 2026-11-22 | vendor doc |
| OV-s31 | https://api-docs.deepseek.com/guides/tool_calls | 2026-11-22 | vendor doc |
| OV-s32 | https://api-docs.deepseek.com/guides/kv_cache | 2026-11-22 | vendor doc |
| OV-s33 | https://api-docs.deepseek.com/guides/chat_prefix_completion | 2026-11-22 | vendor doc |
| OV-s34 | https://api-docs.deepseek.com/guides/anthropic_api | 2026-11-22 | vendor doc |
| OV-s35 | https://api-docs.deepseek.com/updates | 2026-11-22 | vendor doc |
| OV-s36 | https://docs.mistral.ai/llms-full.txt (sections Reasoning, Prompt caching, Chat completions, Function Calling, Error glossary, Safe Prompt, Native reasoning (deprecated), fine-tuning data format) | 2026-11-22 | vendor doc |
| OV-s37 | https://docs.mistral.ai/openapi.yaml (schemas `ChatCompletionRequest`, `ThinkChunk`, `MistralPromptMode`, `ToolChoiceEnum`, `ReasoningEffort`) | 2026-11-22 | vendor doc |
| OV-s38 | https://platform.kimi.ai/docs/guide/use-thinking-models.md | 2026-11-22 | vendor doc |
| OV-s39 | https://platform.kimi.ai/docs/api/chat.md | 2026-11-22 | vendor doc |
| OV-s40 | https://platform.kimi.ai/docs/api/models-overview.md | 2026-11-22 | vendor doc |
| OV-s41 | https://platform.kimi.ai/docs/guide/use-official-tools.md | 2026-11-22 | vendor doc |
| OV-s42 | https://platform.kimi.ai/docs/guide/use-partial-mode-feature-of-kimi-api.md | 2026-11-22 | vendor doc |
| OV-s43 | https://huggingface.co/moonshotai/Kimi-K2-Instruct/raw/main/docs/tool_call_guidance.md | 2026-11-22 | vendor doc |
| OV-s44 | https://platform.kimi.ai/docs/llms.txt (reached by redirect from https://platform.moonshot.ai/docs/llms.txt) | 2026-11-22 | vendor doc |
| OV-s45 | https://www.alibabacloud.com/help/en/model-studio/deep-thinking (page stamp Nov 16, 2026) | 2026-11-22 | vendor doc |
| OV-s46 | https://www.alibabacloud.com/help/en/model-studio/qwen-api-via-openai-chat-completions (page stamp Nov 20, 2026) | 2026-11-22 | vendor doc |
| OV-s47 | https://www.alibabacloud.com/help/en/model-studio/context-cache (page stamp Nov 21, 2026) | 2026-11-22 | vendor doc |
| OV-s48 | https://www.alibabacloud.com/help/en/model-studio/web-search (page stamp Nov 21, 2026) | 2026-11-22 | vendor doc |
| OV-s49 | https://docs.z.ai/guides/capabilities/thinking-mode.md | 2026-11-22 | vendor doc |
| OV-s50 | https://docs.z.ai/api-reference/llm/chat-completion.md | 2026-11-22 | vendor doc |
| OV-s51 | https://docs.z.ai/api-reference/api-code.md | 2026-11-22 | vendor doc |
| OV-s52 | https://docs.z.ai/guides/capabilities/cache.md | 2026-11-22 | vendor doc |
| OV-s53 | https://platform.minimax.io/docs/api-reference/text-openai-api.md | 2026-11-22 | vendor doc |
| OV-s54 | https://platform.minimax.io/docs/api-reference/text-chat-openai.md | 2026-11-22 | vendor doc |
| OV-s55 | https://platform.minimax.io/docs/guides/text-m3-function-call.md | 2026-11-22 | vendor doc |
| OV-s56 | https://platform.minimax.io/docs/api-reference/errorcode.md | 2026-11-22 | vendor doc |
| OV-s57 | https://dev.meta.ai/docs/tool-calling.md | 2026-11-22 | vendor doc |
| OV-s58 | https://dev.meta.ai/docs/reasoning.md | 2026-11-22 | vendor doc |
| OV-s59 | https://dev.meta.ai/docs/protocols.md | 2026-11-22 | vendor doc |
| OV-s60 | `ggml-org/llama.cpp` at `8d60873dfb8619f2b5ed31adfbab07cccb40db5c`, `tools/server/README.md` | 2026-11-22 | vendor doc (local server) |
| OV-s61 | `ggml-org/llama.cpp` at `8d60873d`, `common/jinja/caps.h` lines 12 to 27 | 2026-11-22 | gateway source (local server) |
| OV-s62 | `ggml-org/llama.cpp` at `8d60873d`, `tools/server/server-common.cpp` lines 1496 to 1508 and 1618 to 1620, `tools/server/server-common.h` lines 46 to 58 | 2026-11-22 | gateway source (local server) |
| OV-s63 | `ggml-org/llama.cpp` at `8d60873d`, `tools/server/server-task.cpp` lines 531 to 540, 750 to 756, 921 to 930, and `tools/server/server-chat.cpp` lines 451 to 585 | 2026-11-22 | gateway source (local server) |
| OV-s64 | `ggml-org/llama.cpp` at `8d60873d`, `common/chat.cpp` lines 324 to 334 and 1599 to 1607 | 2026-11-22 | gateway source (local server) |
| OV-s65 | `vllm-project/vllm` at `e2052f5c7ad64564f00dd4592e77c27785c9333f`, `docs/features/reasoning_outputs.md` | 2026-11-22 | vendor doc (local server) |
| OV-s66 | `vllm-project/vllm` at `e2052f5c`, `docs/features/tool_calling.md` | 2026-11-22 | vendor doc (local server) |
| OV-s67 | `vllm-project/vllm` at `e2052f5c`, `docs/features/structured_outputs.md` | 2026-11-22 | vendor doc (local server) |
| OV-s68 | `vllm-project/vllm` at `e2052f5c`, `docs/serving/online_serving/README.md` and `openai_compatible_server.md` | 2026-11-22 | vendor doc (local server) |
| OV-s69 | `vllm-project/vllm` at `e2052f5c`, `docs/features/interleaved_thinking.md` | 2026-11-22 | vendor doc (local server) |
| OV-s70 | https://docs.ollama.com/api/openai-compatibility.md | 2026-11-22 | vendor doc (local server) |
| OV-s71 | https://docs.ollama.com/capabilities/thinking.md | 2026-11-22 | vendor doc (local server) |
| OV-s72 | `ollama/ollama` at `318ee284aa3b5558de57efefff730ea33c4a9bef`, `openai/openai.go` lines 36, 47, 125, 126 | 2026-11-22 | gateway source (local server) |
| OV-s73 | `BerriAI/litellm` at `90dce4b93bd2b090621a6b9083311294d3358bc6` (committed 2026-11-22), `litellm/llms/deepseek/chat/transformation.py` lines 94 to 132 | 2026-11-22 | gateway source |
| OV-s74 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/moonshot/chat/transformation.py` lines 100 to 245 | 2026-11-22 | gateway source |
| OV-s75 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/xai/chat/transformation.py` lines 268 to 275 | 2026-11-22 | gateway source |
| OV-s76 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/openrouter/chat/transformation.py` line 323 | 2026-11-22 | gateway source |
| OV-s77 | `badlogic/pi-mono` at `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (committed 2026-11-22), `packages/ai/src/api/openai-completions.ts` lines 580 to 600, 1193 to 1219, 1318 to 1387, 1528 to 1560, 1600 to 1680 | 2026-11-22 | harness source |
| OV-s78 | `badlogic/pi-mono` at `9c465268`, `packages/ai/src/providers/compat-schema.ts` lines 39, 251 to 283, 307 | 2026-11-22 | harness source |
| OV-s79 | `badlogic/pi-mono` at `9c465268`, `packages/ai/src/api/mistral-conversations.ts` lines 30, 143 to 145, 235 to 264 | 2026-11-22 | harness source |
| OV-s80 | `sst/opencode` at `10144b53c34c6d1b9c15b1e62fad9e87e1efb673` (committed 2026-11-23 +08:00), `packages/ai/src/protocols/openai-chat.ts` lines 114 to 126, 560 to 596, 660 to 690 | 2026-11-22 | harness source |
| OV-s81 | `sst/opencode` at `10144b53`, `packages/ai/src/providers/openrouter.ts` lines 146 to 177 | 2026-11-22 | harness source |
| OV-s82 | `sst/opencode` at `10144b53`, `packages/ai/src/protocols/utils/mistral-tool-id.ts` | 2026-11-22 | harness source |
| OV-s83 | https://github.com/RooCodeInc/Roo-Code/pull/10102 (search excerpt) | 2026-11-22 | issue report (third-party) |
| OV-s84 | https://github.com/openclaw/openclaw/issues/52548 (search excerpt; created 2026-03-23) | 2026-11-22 | issue report (third-party) |
| OV-s85 | https://github.com/openclaw/openclaw/issues/57672 (search excerpt; created 2026-03-30) | 2026-11-22 | issue report (third-party) |
| OV-s86 | https://github.com/openclaw/openclaw/issues/71160 (search excerpt; created 2026-04-24) | 2026-11-22 | issue report (third-party) |
| OV-s87 | https://github.com/anomalyco/opencode/issues/24261 (search excerpt; created 2026-04-25) | 2026-11-22 | issue report (third-party) |
| OV-s88 | https://github.com/anomalyco/opencode/issues/24569 (search excerpt; comment dated 2026-04-27) | 2026-11-22 | issue report (third-party) |
| OV-s89 | https://www.reddit.com/r/openrouter/comments/1tqvr53/tool_calling_broken_with_deepseek_v4_pro_when/ (search excerpt; posted 2026-05-29) | 2026-11-22 | blog (third-party) |
| OV-s90 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (dated 2026-09-20) | 2026-11-22 | workspace document |
| OV-s91 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (dated 2026-10-28) | 2026-11-22 | workspace document |
| OV-s92 | `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/` (`xai.rs`, `deepseek.rs`, `mistral.rs`, `moonshot.rs`, `qwen.rs`, `minimax.rs`, `meta.rs`) and `tests/fixtures/2026-09-14-*.json` | 2026-11-22 | workspace document |
| OV-s93 | https://github.com/BerriAI/litellm/pull/22318 (search excerpt) | 2026-11-22 | issue report (third-party) |
| OV-s94 | https://github.com/musistudio/claude-code-router/issues/1070 (search excerpt; comment dated 2025-12-19) | 2026-11-22 | issue report (third-party) |
