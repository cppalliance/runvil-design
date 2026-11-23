---
produced: 2026-11-22
title: OpenRouter field evidence part 2 - how Rust, Python and Go harnesses (Zed, Goose, rig, genai, Aider, Hermes Agent, Pydantic AI, LangChain, Codex) special-case OpenRouter in source, recorded OpenRouter wire traffic, and issue reports of reasoning_details, thought_signature, cache_control, tool-call id, usage and mid-stream data loss
---

# OpenRouter field evidence part 2 - how Rust, Python and Go harnesses (Zed, Goose, rig, genai, Aider, Hermes Agent, Pydantic AI, LangChain, Codex) special-case OpenRouter in source, recorded OpenRouter wire traffic, and issue reports of reasoning_details, thought_signature, cache_control, tool-call id, usage and mid-stream data loss

- What was read: the OpenRouter code paths of nine projects at pinned commits (Zed, Goose, rig, genai, Aider, Hermes Agent, Pydantic AI, langchain-openrouter, Codex), the 63 fixture files of recorded OpenRouter traffic that rig keeps for its tests, four pages of OpenRouter's documentation, and 26 public issue, pull request, commit and blog pages.
- How many sources: 42 (10 repository sources, with rig counted once for code and once for recorded traffic; 4 OpenRouter documentation pages; 2 workspace documents; 25 issue, pull request and commit pages; 1 blog page). The file holds 129 evidence items and 26 candidate matrix rows.
- Read date: 2026-11-22 for every source. Every repository was cloned without credentials and read at the commit shown in Sources.

A note on the Go part of the topic title: the task named Rust and Python projects only, so this file holds Rust and Python evidence and no Go source was read.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Signed thinking replay | Anthropic (direct, Bedrock, Vertex) | A thinking block's `signature` must return unchanged with its text | Assistant message field `reasoning_details[]`, entries `{"type":"reasoning.text","text","signature","format":"anthropic-claude-v1","index"}`. Streamed as text fragments, then one text-less entry holding the signature. Replayed array is accepted (HTTP 200) on all three Claude routes, even after the tool list changed | One array per message, so the position of a thinking block among text and tool calls is flattened. The plain `reasoning` string duplicates the text | Caller holds state. Round trip unchanged and in order. Chat route. Caller needs the model id to decide whether to replay | OR-inferred (recorded traffic in rig fixtures, client code in rig, genai, Pydantic AI, Hermes, LangChain) | G26, G37, G39, G47, G61, G80, G98 |
| Signature wrapping | Anthropic | Signature is an opaque vendor string | Since a date between 2026-09-22 and 2026-10-02 the `signature` value is `<vendor signature>.<base64url JSON>` where the JSON is `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001\|amazon-bedrock/global"}` | The value is OpenRouter's own envelope, so it is not the vendor's signature byte for byte | The blob is only valid back through OpenRouter. A gateway that hands a caller such a value has to accept it back itself | OR-inferred (recorded traffic in rig fixtures) | G37 |
| Zed's opposite rule for Claude | Anthropic | Same as above | Zed sends `reasoning_details` back for every model except ids starting `anthropic/` (also `~anthropic/`), and drops thinking blocks from the OpenRouter request | Claude through Zed plus OpenRouter continues a tool loop without its earlier thinking | A client-side vendor branch keyed on the model id prefix. Conflicts with rig's recorded acceptance | OR-inferred (Zed code comment) | G1, G122 |
| Encrypted reasoning | OpenAI | Responses API `reasoning` item with `encrypted_content`, replayed when `store` is false | Chat route: `reasoning_details[]` entries `{"type":"reasoning.encrypted","data","id":"rs_...","format":"openai-responses-v1","index"}` plus `reasoning.summary` entries. `native_finish_reason` is `completed`, a Responses status. `reasoning.exclude: true` still returns the encrypted entry | The Responses item structure is flattened into array entries. LangChain strips ids that start `rs_` before replay | Caller holds state. Round trip unchanged. Chat route reaches a Responses-only feature | OR-inferred (recorded traffic in rig fixtures, client code) | G41, G43, G98 |
| Gemini thought signatures | Google (AI Studio, Vertex) | A function call's `thought_signature` must return on the same call | Three shapes seen: `reasoning_details[]` entry `{"type":"reasoning.encrypted","data","id":"<tool call id>","format":"google-gemini-v1"}`; a `reasoning.text` entry with only `signature`; and `tool_calls[].function.thought_signature` or `tool_calls[].extra_content.google.thought_signature` | The signature is tied to a tool call by `id`, so a client that strips ids breaks it. Signatures are scoped to the upstream provider: a turn that moves between `Google AI Studio` and `Google` (Vertex) fails with 400 | Caller holds state and must also pin the upstream provider (`provider.order` with `allow_fallbacks: false`), so the caller has to know which upstream served the request | OR-inferred (client code, issue reports including one answered by OpenRouter's SDK team) | G2, G14, G62, G82, G113, G114, G115 |
| Replay across models | all | Each vendor only accepts its own opaque reasoning | No field says which model may take a blob back. Each entry has a `format` tag | Clients invent the rule: rig replays only to the same wire, provider and model; Pydantic AI to the same system `openrouter`; genai to the same adapter kind | Caller decides when to drop. A gateway that tags each stored block with protocol and model can decide for the caller | Filled (rig, Pydantic AI, genai) | G27, G38, G48, G81 |
| Reasoning controls | all | Effort levels, token budgets, or always-on thinking, per vendor | Request object `reasoning` with `effort`, `max_tokens`, `enabled`, `exclude`. Routes with mandatory reasoning answer a disable with 400 "Reasoning is mandatory for this endpoint and cannot be disabled". For always-thinking Claude models, effort travels in top-level `verbosity` and `reasoning.effort` is ignored | A neutral off switch is not honored everywhere. The catalog's `reasoning.mandatory` and `supported_efforts` are needed to pick a valid value, and one catalog entry was wrong | A neutral on/off or effort setting needs per-model translation from catalog data, plus a retry rule. Gateway-side work | OR-inferred (client code in Goose, Hermes, Pydantic AI, rig, Zed; issue reports) | G13, G17, G29, G63, G64, G88, G110, G111 |
| Forced tool choice while thinking | Anthropic, Alibaba (Qwen) | Anthropic rejects a forced `tool_choice` with thinking on | On Anthropic routes OpenRouter drops `reasoning` and answers without reasoning. Qwen routes answer 400 "The tool_choice parameter does not support being set to required or object in thinking mode" | The caller's thinking request is dropped without an error on one vendor and rejected on another | Vendor-specific outcome for one neutral request | OR-inferred (Pydantic AI code comments with live probe dates) | G89 |
| Cache breakpoints | Anthropic, Google, Moonshot (Kimi) | Anthropic: up to 4 `cache_control` breakpoints with TTL. Gemini: implicit plus explicit | Per-block `cache_control: {"type":"ephemeral","ttl":"1h"}` on content parts and on the last tool's `function`, or one top-level `cache_control`. Clients gate on the `anthropic/` id prefix. On Gemini only the last breakpoint is used. Kimi through OpenRouter also needs the marker | Clients must know the vendor to place markers: 4-marker limit, no TTL on Gemini, no marker at the request end on Gemini (billed twice on Vertex), no top-level marker on `role: tool` | Caller or gateway places markers per vendor. A neutral "cache this prefix" hint needs vendor rules in the gateway | OR-inferred (client code in Zed, Goose, rig, Hermes, Pydantic AI); OR-documented for the field names | G4, G18, G28, G67, G87, G112, G121 |
| Cache and token usage | all | Vendor-specific counters | `usage.prompt_tokens_details.cached_tokens` and `.cache_write_tokens`; `completion_tokens_details.reasoning_tokens` | Anthropic's 5-minute and 1-hour write counts arrive as one number | Gateway normalizes. Field names match what clients already read | OR-inferred (recorded traffic, client code); OR-documented | G6, G20, G44, G71, G86, G100, G119 |
| Cost | OpenRouter | Vendors do not return cost | `usage.cost`, `usage.cost_details.upstream_inference_cost` (chat also `upstream_inference_prompt_cost`, `upstream_inference_completions_cost`; Responses `upstream_inference_input_cost`, `upstream_inference_output_cost`), `usage.is_byok` | Catalog price `-1` is a sentinel for "varies" | Gateway computes. Clients store it beside usage | OR-inferred (recorded traffic, client code) | G20, G32, G44, G72, G86, G100 |
| Usage chunk in a stream | OpenRouter | OpenAI sends usage on a chunk with `choices: []` | The usage chunk has one choice with `delta: {"content":"","role":"assistant"}` that repeats `finish_reason` and `native_finish_reason`. Usage is always sent; `usage: {include: true}` and `stream_options.include_usage` are documented as deprecated | Clients that assume empty `choices` lose or double usage | Client parser must take usage from any chunk | OR-inferred (recorded traffic, issue report); the deprecation is OR-documented | G20, G44, G118 |
| Finish reasons | all | Vendor vocabularies differ | `finish_reason` (stop, length, tool_calls, content_filter, error) and `native_finish_reason` (seen: `tool_use`, `end_turn`, `STOP`, `completed`, `max_output_tokens`, `MALFORMED_FUNCTION_CALL`) | `error` is outside the OpenAI enum and breaks strict SDK parsers | Client must accept `error`. Native reason is optional detail | OR-inferred (recorded traffic, client code, issue reports) | G31, G41, G85, G115, G116, G120 |
| Errors inside HTTP 200 and mid-stream | OpenRouter | n/a | Body or SSE chunk `{"error":{"code","message","metadata":{"raw","provider_name"}}}`; `code` holds the status the error would have had. Also seen: all standard fields null with an `error`, null `choices` with no `error`, and a whole completion nested under `provider` | The upstream's own error text sits in `metadata.raw` as a string, often JSON inside a string | Client must classify a 200 body. Gateway should surface one error shape | OR-inferred (client code in Zed, Pydantic AI, Hermes, rig, Goose, LangChain; issue reports) | G7, G73, G75, G83, G84, G99, G108, G109, G117, G120 |
| Rate limits and credits | OpenRouter and upstreams | n/a | 429 with `error.metadata.retry_after_seconds` or header `X-RateLimit-Reset` (epoch milliseconds, sometimes absent); 402 "You requested up to 65536 tokens, but can only afford 56272"; 403 "Budget limit exceeded (monthly limit)"; upstream 429 wrapped as "Provider returned error" | Whether the key or the upstream is limited has to be read from the wrapper | Client retry rules need the wrapper fields | OR-inferred (client code in Zed, Goose, Hermes, Aider) | G8, G21, G55, G74 |
| Tool-call ids | all | `toolu_...` (Anthropic), `call_...` (OpenAI) | Upstream ids pass through: `toolu_01...` (Anthropic), `toolu_bdrk_01...` (Bedrock), `toolu_vrtx_01...` (Vertex), `call_...` (OpenAI). A `call_...` id replayed to Claude on Bedrock was accepted. Responses route: item `id: "fc_tmp_..."` with the native id in `call_id` | The id reveals the upstream. Id format is not normalized | Caller treats ids as opaque. Cross-vendor replay of ids can fail on strict vendors | OR-inferred (recorded traffic, issue reports) | G40, G107, G124, G125 |
| Tool results sent to Gemini | Google | `function_response` is structured | `role: tool` text becomes a Gemini `function_response`; text containing a JSON Schema `$ref` key is rejected with Google 400 on every later turn | Tool output is not treated as opaque text | Client-side rewrite in Goose (`$ref` to `dollar_ref` plus a note). A gateway translator would own this | OR-inferred (Goose code, issue report) | G16, G123 |
| Tool and output schemas for Gemini | Google | Newer Gemini accepts `$defs` and `$ref` in its own schema field | Through OpenRouter, `$defs`, `$ref` and nullable `anyOf` in tool parameters gave degraded tool calls | Schema features are lost in translation | Client inlines definitions for `google/` models | OR-inferred (Pydantic AI code, issue report) | G94, G126 |
| Provider routing | OpenRouter | n/a | Request `provider` object (`order`, `only`, `ignore`, `allow_fallbacks`, `require_parameters`, `data_collection`, `zdr`, `sort`, `max_price`, `quantizations`, latency and throughput preferences); reply top-level `provider` names the upstream. OpenAI speed tiers are endpoints (`openai/fast`, `openai/flex`) | Behavior differs by upstream for the same model id, so tests pin `provider.order` with `allow_fallbacks: false` | Caller must be able to pin an upstream and read which one served | OR-inferred (client code in rig, Hermes, Zed, Pydantic AI, LangChain) | G12, G33, G65, G78, G95, G101 |
| Session stickiness | OpenRouter, xAI | xAI pins its cache by a conversation header | Top-level `session_id` (Zed sends a SHA-256 of its thread id; Goose also sets `user`); Hermes adds header `x-grok-conv-id` for `x-ai/grok-` models | n/a | Caller supplies a stable opaque id per conversation | OR-inferred (client code) | G5, G19, G66, G68 |
| Model metadata | OpenRouter | n/a | `GET /api/v1/models` and `/models/user`: `supported_parameters`, `context_length`, `top_provider.context_length`, `architecture.input_modalities`, `pricing`, `reasoning.{supported_efforts, default_effort, default_enabled, mandatory, supports_max_tokens}`; id variants `:free`, `:nitro`, `:thinking`; aliases prefixed `~` | Clients still branch on the vendor prefix of the id | Gateway can serve capability data so the client does not branch on vendor | OR-inferred (client code in Zed, Goose, Aider, Hermes, Pydantic AI) | G10, G23, G54, G64, G88 |
| Headers | OpenRouter, Anthropic | Anthropic beta features use `anthropic-beta` | `HTTP-Referer`, `X-Title`, `X-OpenRouter-Categories` for attribution; Hermes sends `x-anthropic-beta` for fine-grained tool streaming on Claude; Aider sends `anthropic-beta`; `X-OpenRouter-Cache: false` and reply header `X-OpenRouter-Cache-Status` for the response cache | A vendor beta header in a universal request | A vendor header set by the client is an L3 pattern in these clients | OR-inferred (client code) | G9, G22, G56, G69, G77, G96, G102 |
| Responses route | OpenAI protocol over any vendor | Stateful by default, stateless with `store: false` and encrypted reasoning | Codex sends `store: false`, `stream: true`, `include: ["reasoning.encrypted_content"]`, `prompt_cache_key`. Recorded replies show `store: false`, `previous_response_id: null`, item ids `rs_tmp_...`, `msg_tmp_...`, `fc_tmp_...`, and for Claude a reasoning item with `signature` and `format: "anthropic-claude-v1"` | Server-side conversation state | Caller holds all state on this route too | OR-inferred (recorded traffic in rig fixtures, Codex code); OR-documented for the Codex setup | G103, G104, G106, G107 |
| System message placement | Anthropic, Google | System text is a top-level field | A `system` message in mid-conversation is kept in place on some routes, moved into the top-level system prompt on others (breaking the cached prefix), or turned into user text | Position of later system messages | Gateway translator decides placement. Client cannot tell without per-model tables | OR-inferred (Pydantic AI code comments with live probe dates) | G91 |
| Several choices | OpenAI | `n > 1` returns n choices | `n = 2` is accepted and only candidate zero is returned | Extra candidates | Not needed by Runvil | OR-inferred (rig test comment over recorded traffic) | G42 |
| Anthropic Messages surface | Anthropic | `message_start` reports input tokens | Through OpenRouter's Messages surface `message_start` reports `input_tokens: 0` and the real count arrives on `message_delta`; Hermes measured the previous turn's cache being rewritten on 14 to 20 percent of consecutive calls | Early token count; cache stability | A second native route has its own differences from the vendor | OR-inferred (rig test comment, Hermes code comment) | G36, G70 |

## Evidence

How to read the labels: an item that states what OpenRouter does is `OR-inferred` and names what it is inferred from. An item that states how a project stores or handles something is `Filled` and names the project. Items taken from OpenRouter's own pages are `OR-documented`. Recorded traffic means the request and response bodies of real OpenRouter calls that rig keeps as replay fixtures under `crates/rig-cassette/fixtures/cassettes/openrouter/` (63 files). The recording date of each fixture is read from the Unix time inside its `gen-<unix>-<random>` response id, because the fixtures scrub `created` to 0. In code quotes, lines that span several source lines are joined with single spaces, and ` / ` separates consecutive comment lines; the words are unchanged.

### Zed (Rust) - `zed-industries/zed` at `d31d107`

**G1. Zed sends `reasoning_details` back for every OpenRouter model except Anthropic ones, and drops thinking blocks from the OpenRouter request.**

- Quote (`crates/language_models/src/provider/open_router.rs:459-468`): "// Anthropic models via OpenRouter don't accept reasoning_details being echoed back" / "// in requests - it's an output-only field for them. However, Gemini models require" / "// the thought signatures to be echoed back for proper reasoning chain continuity." and `let is_anthropic_model = model.id().trim_start_matches('~').starts_with("anthropic/");`
- Quote (`:477-481`): `let reasoning_details_for_message = if is_anthropic_model { None } else { message.reasoning_details.clone() };`
- Quote (`:502-504`): `MessageContent::Thinking { .. } => {}` / `MessageContent::RedactedThinking(_) => {}` / `MessageContent::Compaction(_) => {}`
- Source: G-s1. Basis: OR-inferred, from Zed's code comment. Confidence: low for the claim about OpenRouter itself, because rig's recorded traffic shows the opposite (G37, G39). High for what Zed does.

**G2. Zed reads and writes a Gemini signature on the tool call's `function` object.**

- Quote (`crates/open_router/src/open_router.rs:360-365`): `pub struct FunctionContent { pub name: String, pub arguments: String, #[serde(default, skip_serializing_if = "Option::is_none")] pub thought_signature: Option<String>, }`
- Quote (`crates/language_model_core/src/chat_completion.rs:203-205`): "/// Provider-defined metadata required to replay a reasoning tool call." above `pub thought_signature: Option<String>,`
- Source: G-s1. Basis: OR-inferred, from Zed's wire types (the field sits inside `tool_calls[].function`). Confidence: medium. The recorded Gemini traffic in rig's fixtures shows the signature in `reasoning_details` instead (G46), so OpenRouter may send both or the shape changed.

**G3. Storage shape in Zed: one raw JSON value per assistant message, merged from stream fragments, plus an optional string per tool call.**

- Quote (`crates/language_model_core/src/request.rs:365-366`): `#[serde(default, skip_serializing_if = "Option::is_none")] pub reasoning_details: Option<Arc<serde_json::Value>>,`
- Quote (`crates/language_model_core/src/chat_completion.rs:254-258`): "/// Accumulates structured reasoning metadata from compatible providers." / "/// Array entries are matched by `index` and then `id`. Fragmented `text`," / "/// `summary`, and `data` fields are concatenated while other non-null fields" / "/// replace their previous values."
- Quote (`:1217-1218`): "// OpenRouter sends an empty `reasoning_details` array in the finish chunk;" / "// it must not wipe out details accumulated from earlier chunks."
- Source: G-s1. Basis: Filled (Zed) for the storage shape; the finish-chunk remark is OR-inferred from Zed's test comment, and it matches the recorded stream in Goose's test fixture (G20).

**G4. Zed places explicit per-block breakpoints for Anthropic ids only, and avoids the top-level marker.**

- Quote (`open_router.rs` provider, `:577-579`): `if is_anthropic_model && any_message_wants_cache {` / "// OpenRouter's top-level automatic cache_control restricts routing to" / "// Anthropic direct; explicit block breakpoints also work on Bedrock and Vertex."
- Quote (`:590-594`): `set_last_text_cache_control(content, cache_control(Some(open_router::CacheTtl::OneHour)),);` on the system message; the last cached message gets `cache_control(None)`.
- Quote (`crates/open_router/src/open_router.rs:313-323`): `pub enum CacheTtl { #[serde(rename = "5m")] FiveMinutes, ... #[serde(rename = "1h")] OneHour, }`
- Source: G-s1. Basis: OR-inferred, from Zed's code and comment. Confidence: medium. OpenRouter's page read today says the top-level form is supported on four providers (see Conflicts).

**G5. Zed sends a hashed `session_id`.**

- Quote (`:672-679`): `hasher.update(b"zed-openrouter-session-v1\0"); hasher.update(thread_id.as_bytes()); format!("{:x}", hasher.finalize())`
- Source: G-s1. Basis: Filled (Zed).

**G6. Zed asks for usage with `usage: {include: true}` and reads cache counts from `prompt_tokens_details`.**

- Quote (`:608`): `usage: open_router::RequestUsage { include: true },`
- Quote (`chat_completion.rs:219-225`): `pub struct PromptTokensDetails { /// Tokens read from a prompt cache. pub cached_tokens: Option<u64>, /// Tokens written to a prompt cache. pub cache_write_tokens: Option<u64>, }`
- Quote (`:243-246`): `input_tokens: prompt_tokens.saturating_sub(cache_creation_input_tokens).saturating_sub(cache_read_input_tokens),`
- Source: G-s1. Basis: OR-inferred, from Zed's usage mapping: `prompt_tokens` includes cached and cache-written tokens. Confidence: medium.

**G7. Zed keeps the HTTP status apart from the numeric `error.code`, and treats an `error` object inside the stream as a failure.**

- Quote (`crates/open_router/src/open_router.rs:671-673`): "/// The HTTP status remains distinct from the provider's numeric error code" / "/// because OpenRouter can report different values for them, and streaming" / "/// errors do not have their own HTTP response."
- Quote (`:655-661`): `pub struct OpenRouterErrorBody { pub code: u16, pub message: String, #[serde(default, skip_serializing_if = "Option::is_none")] pub metadata: Option<std::collections::HashMap<String, serde_json::Value>>, }`
- Quote (test at `:826-839`): body `{"error":{"code":499,"message":"upstream provider failed"}}` with HTTP 500 yields `api_error.status == Some(500)` and `api_error.code == 499`.
- Source: G-s1. Basis: OR-inferred, from Zed's error types and tests. Confidence: high for the shape, medium for how often status and code differ.

**G8. Zed reads the rate-limit reset from `X-RateLimit-Reset` as epoch milliseconds and waits 60 seconds when the header is absent.**

- Quote (`:643-645`): "/// OpenRouter reports a rate limit's reset time via `X-RateLimit-Reset` when" / "/// present, but omits it on some rate-limited responses; a minute is a" / "/// reasonable default backoff for those."
- Source: G-s1. Basis: OR-inferred, from Zed's code comment. Confidence: medium.

**G9. Zed sends attribution headers and reserves them.**

- Quote (`:464-471`): header names `"http-referer"` with value `"https://zed.dev"` and `"x-title"` with value `OPEN_ROUTER_APP_TITLE`; provider file `:32`: `pub(crate) const RESERVED_HEADER_NAMES: &[&str] = &["HTTP-Referer", "X-Title"];`
- Source: G-s1. Basis: Filled (Zed).

**G10. Zed builds its model list from `GET {api_url}/models/user` and the catalog's capability fields.**

- Quote (`:486`): `let uri = format!("{api_url}/models/user");`
- Quote (`:534`, `:542-549`, `:411-423`): `supports_tools: Some(entry.supported_parameters.contains(&"tools".to_string())),` and `mode: if entry.supported_parameters.contains(&"reasoning".to_string()) { ModelMode::Adaptive }`, and `pub struct ModelReasoning { default_effort, default_enabled, mandatory, supported_efforts, supports_max_tokens }`
- Quote (provider `:433`): `supports_disabling_thinking: !model.mandatory_reasoning,`
- Source: G-s1. Basis: OR-inferred, from Zed's deserialization types (the catalog has a per-model `reasoning` object with these keys). Confidence: high, the same keys are in the local `models.json`.

**G11. Zed refuses custom (non-function) tools on OpenRouter.**

- Quote (provider `:455-457`): `if request.contains_custom_tool_input() { anyhow::bail!("OpenRouter does not support custom tools"); }`
- Source: G-s1. Basis: OR-inferred, from Zed's guard. Confidence: low, it may be Zed's own limit.

**G12. Zed forwards a user-configured `provider` object unchanged.**

- Quote (`crates/open_router/src/open_router.rs:184`): `pub provider: Option<Provider>,` and provider `:668`: `provider: model.provider.clone(),`
- Source: G-s1. Basis: Filled (Zed).

**G13. Zed maps its thinking switch to OpenRouter's `reasoning` object.**

- Quote (`:217-227`): `pub struct Reasoning { pub effort: Option<ReasoningEffort>, pub max_tokens: Option<u32>, pub exclude: Option<bool>, pub enabled: Option<bool>, }`
- Quote (provider `:629-637`): when thinking is off and reasoning is not mandatory, `Some(open_router::Reasoning { enabled: Some(false), effort: None, max_tokens: None, exclude: None, })`; when mandatory, the field is omitted.
- Source: G-s1. Basis: Filled (Zed).

### Goose (Rust) - `aaif-goose/goose` (moved from `block/goose`) at `36b2911`

**G14. Goose adds `reasoning_details` to the request only for `google/gemini` ids, and stores them in the tool request's metadata map.**

- Quote (`crates/oi-kit-providers/src/openrouter.rs:203-205`, `:415-420`): `fn is_gemini_model(model_name: &str) -> bool { model_name.starts_with("google/gemini") }` and `if is_gemini_model(model_name) { escape_gemini_schema_ref_keys_in_tool_responses(payload); openrouter_format::add_reasoning_details_to_request(payload, messages); }`
- Quote (`crates/oi-kit-providers/src/openrouter_format.rs:8`, `:40-47`): `pub const REASONING_DETAILS_KEY: &str = "reasoning_details";` and `if let MessageContent::ToolRequest(req) = content { let mut meta = req.metadata.clone().unwrap_or_default(); meta.insert(REASONING_DETAILS_KEY.to_string(), json!(details)); req.metadata = Some(meta); }`
- Source: G-s2. Basis: Filled (Goose). Storage shape: `ProviderMetadata` (a string-keyed JSON map) on each tool request, key `reasoning_details`, value the raw array.

**G15. In Goose's streaming path the accumulated `reasoning_details` are built and then left unused.**

- Quote (`crates/oi-kit-provider-types/src/formats/openai.rs:1346-1348`): `if let Some(details) = &chunk.choices[0].delta.reasoning_details { accumulated_reasoning.extend(details.iter().cloned()); }`
- Quote (`:1479-1485`): `let _metadata: Option<ProviderMetadata> = if !accumulated_reasoning.is_empty() { let mut map = ProviderMetadata::new(); map.insert("reasoning_details".to_string(), json!(accumulated_reasoning)); Some(map) } else { None };`
- Quote (`:1535-1543`): the tool request's metadata is built from `last_signature` (key `GEMINI_THOUGHT_SIGNATURE_KEY`) and the tool call's own extra fields.
- The provider's only request path is `stream` (`openrouter.rs:601-679`), and `openrouter_format::response_to_message`, the one function that writes the `reasoning_details` key, has zero callers outside its own tests (searched `crates/` for `openrouter_format::` and `extract_reasoning_details`).
- Source: G-s2. Basis: Filled (Goose). Confidence: medium. This is my reading of one commit; I did not run it. If right, Goose at this commit relies on the signature that arrives on the tool call itself (`extra_content.google.thought_signature`, test at `:4207`), and the `reasoning_details` replay in G14 finds nothing to send after a streamed turn.

**G16. Goose rewrites a literal `$ref` token in tool results before they go to Gemini through OpenRouter.**

- Quote (`openrouter.rs:422-426`): "/// OpenRouter translates OpenAI `role: tool` messages into Gemini" / "/// `function_response` parts. Gemini rejects a response containing a literal" / "/// JSON Schema `$ref` key, treating its value as a function-response part name" / "/// instead of arbitrary tool text, and Goose replays persisted history, so one" / "/// such tool result breaks every later turn in the session."
- Quote (`:44-45`, `:409-413`): `const GEMINI_SCHEMA_REF_KEY: &str = "$ref";` / `const GEMINI_SAFE_SCHEMA_REF_KEY_BASE: &str = "dollar_ref";` and the note text "[OpenRouter/Gemini compatibility: interpret `{safe_key}` as the JSON Schema key formed by `$` followed by `ref`.]"
- Source: G-s2. Basis: OR-inferred, from Goose's code comment and the issue behind it (G123). Confidence: high.

**G17. Goose converts `reasoning_effort` into OpenRouter's `reasoning` object and retries once when a route refuses to turn reasoning off.**

- Quote (`openrouter_format.rs:101-103`): "/// Returns true when a reasoning disable request was inserted, which" / "/// mandatory-reasoning endpoints reject; the provider downgrades those to" / "/// the lowest effort on OpenRouter's mandatory-reasoning error."
- Quote (`openrouter.rs:199-201`, `:664-671`): `matches!(error, ProviderError::RequestFailed(message) if message.contains("Reasoning is mandatory"))` and `payload["reasoning"] = json!({ "effort": "low" });`
- Quote (test at `:772`): `"error": { "message": "Reasoning is mandatory for this endpoint and cannot be disabled." }`
- Source: G-s2. Basis: OR-inferred, from Goose's retry rule and test string; the same string is quoted by Pydantic AI (G88) and Hermes (G64). Confidence: high.

**G18. Goose applies Anthropic-style breakpoints to the chat payload when the provider is `openrouter` and the id starts `anthropic/`.**

- Quote (`crates/oi-kit-provider-types/src/cache_semantics.rs:31-33`): `"openrouter" | "litellm" if model_name.starts_with("anthropic/") => { CacheSemantics::ExplicitBreakpoints { max_breakpoints: 4 } }`
- Quote (`:57-58`, `:98-100`): "/// Anthropic-dialect breakpoints for OpenAI-style chat payloads (OpenRouter," / "/// LiteLLM, Databricks)." and `function.insert("cache_control".to_string(), json!({ "type": "ephemeral" }));` on the last tool.
- Quote (`:132-133`): "/// Anthropic rejects `cache_control` on empty text (empty tool results" / "/// serialize to `content: ""`) and on thinking blocks".
- Source: G-s2. Basis: OR-inferred, from Goose's code: OpenRouter passes per-part and per-tool `cache_control` through to Anthropic. Confidence: high (also documented by OpenRouter, G-s11).

**G19. Goose sets `session_id` and `user` to its session id, leaves `transforms` unset, and lets the user pass any extra body keys.**

- Quote (`openrouter.rs:634-638`): `obj.insert("user".to_string(), Value::String(session_id.to_string())); obj.insert("session_id".to_string(), Value::String(session_id.to_string()),);`
- Quote (`:653-656`): "// `transforms` is intentionally not set: OpenRouter defaults to applying" / "// `middle-out` for endpoints with <=8k context, and large-context models" / "// don't benefit from it. Users who want to force a transform can opt in" / "// via `OPENROUTER_PARAMETERS`."
- Source: G-s2. Basis: OR-inferred, from Goose's comment, for the `middle-out` default. Confidence: low, this is a comment and the local survey says OpenRouter's advice moved to a `context-compression` plugin (LC16). Not re-checked by me.

**G20. Goose reads `usage.cost` as a provider-reported cost and reads cache counts from two spellings.**

- Quote (`formats/openai.rs:993-998`, `:1010-1011`): `pub fn get_cost(usage: &Value) -> Option<f64> { usage.get("cost").and_then(|v| v.as_f64()).filter(|c| c.is_finite() && *c >= 0.0) }` and `Some(cost) => usage.with_cost(cost, CostSource::ProviderReported),`
- Quote (`:971-979`): `usage.get("cache_creation_input_tokens") ... .or_else(|| { usage.get("prompt_tokens_details").and_then(|d| d.get("cache_write_tokens")) })`
- Quote (recorded stream in test `test_openrouter_streaming_usage_yielded_once`, `:4026-4032`, response id `gen-1768896871-...`, which dates it 2026-01-20): `"delta":{"role":"assistant","content":"","reasoning":null,"reasoning_details":[]},"finish_reason":"stop","native_finish_reason":"stop"` followed by a chunk with `"choices":[{"index":0,"delta":{"role":"assistant","content":""},"finish_reason":null,"native_finish_reason":null,"logprobs":null}],"usage":{"prompt_tokens":7007,"completion_tokens":49,"total_tokens":7056}`
- Source: G-s2. Basis: OR-inferred, from a recorded stream in Goose's tests. Confidence: high for January 2026; the October 2026 recordings in rig differ in one detail (the usage chunk repeats the finish reason, G44).

**G21. Goose reads a retry delay from `error.metadata.retry_after_seconds` and an overflow signal from `error.metadata.error_type`.**

- Quote (`crates/oi-kit-providers/src/http_status.rs:43-46`): "/// Extract a retry delay from a 429 response. Prefers the body's" / "/// `error.metadata.retry_after_seconds` (OpenRouter shape, more precise than" / "/// the integer header) and falls back to the RFC 7231 `Retry-After` header"
- Quote (test payloads `:963-984`): `("code", json!(400)), ("message", json!("Provider returned error")), ("metadata", json!({ "error_type": "context_length_exceeded", "provider_name": "Relace", "raw": "upstream error" }))` counts as overflow, and `"metadata", json!({ "error_type": "max_tokens_exceeded" })` does not.
- Source: G-s2. Basis: OR-inferred, from Goose's parser and tests. Confidence: medium. Agrees with LC13 on `context_length_exceeded`.

**G22. Goose sends three attribution headers.**

- Quote (`crates/goose/src/providers/openrouter_def.rs:58-60`): `.with_header("HTTP-Referer", "https://goose-docs.ai")?` / `.with_header("X-Title", "goose")?` / `.with_header("X-OpenRouter-Categories", "cli-agent,productivity")?;`
- Source: G-s2. Basis: Filled (Goose).

**G23. Goose reads tool support and context size from `GET api/v1/models`, and strips id variants.**

- Quote (`openrouter.rs:171-174`): `supports_tools: model.get("supported_parameters").and_then(|v| v.as_array()).is_some_and(|params| params.iter().any(|p| p.as_str() == Some("tools"))),`
- Quote (`crates/oi-kit-provider-types/src/canonical/name_builder.rs:68`, `canonical.rs:195-200`): "/// Strip a trailing OpenRouter variant tag such as `:free` or `:nitro`." and "// `:thinking` is the one tag that changes capabilities: it serves the reasoning mode."
- Source: G-s2. Basis: OR-inferred, from Goose's code. Confidence: medium.

**G24. Goose calls a second OpenRouter route, `api/alpha/decisions`, with `provider`, `session_id`, `trace`, and `user` options.**

- Quote (`openrouter.rs:505-515`, `:537`): `pub struct OpenRouterDecisionOptions { pub provider: Option<Value>, pub session_id: Option<String>, pub trace: Option<Value>, pub user: Option<String>, }` and `.request("api/alpha/decisions")`
- Source: G-s2. Basis: Filled (Goose). Not checked against OpenRouter's documentation by me; listed under Unverified.

### rig (Rust) - `0xPlaygrounds/rig` at `a8673f7`: source code

rig no longer has a separate OpenRouter client. `crates/rig-core/src/providers/openrouter/` holds model id constants and typed options, and the wire is the shared OpenAI chat and Responses code with an `OPENROUTER` dialect entry. The workspace's 2026-11-17 survey read rig at `5393668` for its Anthropic path only.

**G25. rig's OpenRouter dialect is seven switches on the shared OpenAI wire.**

- Quote (`crates/rig-core/src/providers/openai/wire/dialects.rs:312-333`): `pub const OPENROUTER: Dialect = Dialect { quirks: Quirks { stream_include_usage: false, verify_path: "/key", // A gateway forwards its upstream's own finish reason. native_finish_reason: true, response_format_with_tools: true, accepts_file_ids: false, rewrite: BodyRewrite::OpenRouter, ... transcription_body: TranscriptionBody::InputAudioJson, ... // The Responses route accepts system instructions as input messages. responses: ResponsesQuirks { system_instructions: SystemInstructionsPlacement::InputSystemMessages, ..ResponsesQuirks::openai() },`
- Quote (`wire.rs:175-177`): "/// OpenRouter: model ids name the upstream vendor (`anthropic/...`)," / "/// which decides what a request carries." above `OpenRouter,`
- Quote (`wire/chat.rs:553`): `BodyRewrite::None | BodyRewrite::OpenRouter | BodyRewrite::HuggingFaceRouter => {}` (the body itself gets no rewrite).
- Source: G-s3. Basis: Filled (rig). What it implies about OpenRouter (OR-inferred from these switches): usage arrives without `stream_options`, `GET /key` checks a credential without spending tokens, file ids are not accepted, and the Responses route takes system text as `input` messages.

**G26. rig keeps `reasoning_details` verbatim on the reasoning block and sends the same array back.**

- Quote (`wire/chat.rs:1102-1105`): "/// The keys an assistant message carries structured reasoning under, each" / "/// kept verbatim with the reasoning text: `reasoning_details`, and the" / "/// signed `thinking_blocks` a LiteLLM proxy returns for Claude." and `const REASONING_DETAIL_KEYS: [&str; 2] = ["reasoning_details", "thinking_blocks"];`
- Quote (`:1395-1407`): `let mut item = Map::new(); if let Some(field) = self.reasoning_field.take() { item.insert(field.to_owned(), text.into()); } item.append(&mut self.reasoning_details); out.edit(at, |slot| *slot = Value::Object(item))?;`
- Quote (`:982-983`): "/// Signed data such as `reasoning_details` or `thinking_blocks` never" / "/// survives an edit."
- Quote (test `wire/chat/hard_case_tests.rs:116-118`): "/// A message's reasoning is one block, as pi keeps it, so a signature that" / "/// arrives after the answer joins the reasoning before it, and the message" / "/// sent back holds the reasoning and the signature as one message did."
- Source: G-s3. Basis: Filled (rig). Storage shape: one reasoning block per assistant message whose provider item is the JSON object `{"reasoning": "<text>", "reasoning_details": [ ...entries as received... ]}`.

**G27. rig's replay rule: a provider item goes back only to the exact wire, provider and model that produced it.**

- Quote (`crates/rig-core/src/completion/message/native.rs:55-59`): "/// Which wire, provider and model produced an assistant turn." / "/// `model` is the model the request named. Replay compares it, with `api`" / "/// and `provider`, against the target: only an exact match replays the" / "/// turn's provider items."
- Quote (`:99-100`): `pub fn same_model(&self, api: &Api, provider: &str, model: &str) -> bool { &self.api == api && self.provider == provider && self.model == model`
- Quote (`:232-262`): `pub struct Native { /// The provider item. pub item: serde_json::Value, /// The fingerprint of the canonical form the item was decoded to. pub fingerprint: Fingerprint, ... pub baseline: Option<serde_json::Value>, ... pub edited: bool, /// Where the item keeps its reasoning blob, as a JSON pointer such as /// `/signature`, when it holds one. ... pub blob_slot: Option<Cow<'static, str>>, }`
- Source: G-s3. Basis: Filled (rig). Storage shape: per block, the raw provider JSON (`item`), a fingerprint of the decoded form to detect edits, and a JSON pointer to the opaque blob; per turn, an `Origin` of `api`, `provider`, `model`, optional `response_model`, `response_id`, and a `context` fingerprint of tools and system prompt.

**G28. rig maps its neutral cache setting to one top-level `cache_control` for upstreams that take a marker, and sends nothing for upstreams that cache by themselves.**

- Quote (`crates/rig-core/src/providers/openai/options.rs:621-644`): "/// The OpenRouter upstreams that cache automatically, by `vendor/` prefix." `const AUTOMATIC_UPSTREAMS: [&str; 4] = ["openai/", "deepseek/", "x-ai/", "groq/"];` and `(CacheRetention::Short, false) => send("cache_control", json!({"type": "ephemeral"})),` and `(CacheRetention::Long, false) => { send("cache_control", json!({"type": "ephemeral", "ttl": "1h"})) }` and `(CacheRetention::None, true) => { Mapping::unsupported("this upstream caches automatically and cannot be stopped") }`
- Quote (Responses route, `:1355-1362`): "/// OpenRouter's Responses route: Chat's cells, with an explicit-cache" / "/// opt-out for OpenAI models, `text.verbosity` and no `seed` or `stop`." and `send("prompt_cache_options", json!({"mode": "explicit"}))`
- Source: G-s3. Basis: Filled (rig). It relies on OpenRouter's documented top-level `cache_control` (G-s11). The vendor is read from the id prefix in client code.

**G29. rig maps reasoning to OpenRouter's `reasoning` object and refuses values an upstream cannot take.**

- Quote (`options.rs:660-669`): `Reasoning::Off if cannot_disable() => { Mapping::unsupported("this upstream cannot turn reasoning off") } Reasoning::Off => send("reasoning", json!({"effort": "none"})), Reasoning::Effort(effort) => send("reasoning", json!({"effort": effort.as_str()})), Reasoning::Budget { .. } if model.starts_with("openai/") || model.starts_with("x-ai/") => { Mapping::unsupported("this upstream takes an effort level, not a budget") } Reasoning::Budget { tokens } => send("reasoning", json!({"max_tokens": tokens})),`
- Source: G-s3. Basis: Filled (rig).

**G30. rig omits `service_tier` when the caller asks for `auto`.**

- Quote (`options.rs:673-677`): "/// OpenRouter's `service_tier`, which has no `auto` value." and `ServiceTier::Auto => Mapping::Omit("requests use the standard tier unless one is named"),`
- Source: G-s3. Basis: OR-inferred, from rig's mapping. Confidence: medium.

**G31. rig reads `native_finish_reason` only when `finish_reason` is absent or empty.**

- Quote (`wire/chat.rs:1266-1276`): "// A gateway's upstream-native reason is consulted only when the" / "// normalized field is absent or empty (OpenRouter's precedence)." and `.str("native_finish_reason").filter(|reason| self.quirks.native_finish_reason && !reason.is_empty()).map(native_finish_reason),`
- Quote (`wire/chat/finish.rs:53-65`): the native map lowercases and accepts `"stop" | "end_turn" | "stop_sequence" | "complete" | "completed"`, `"length" | "max_tokens" | "max_output_tokens" | "model_length"`, `"tool_calls" | "function_call" | "tool_use"`, `"content_filter" | "safety" | "blocklist" | "prohibited_content" | "spii"`.
- Source: G-s3. Basis: Filled (rig).

**G32. rig exposes OpenRouter's extra reply fields through a typed extras struct.**

- Quote (`crates/rig-core/src/providers/openrouter/extension.rs:658-684`): Chat reads `"/provider"`, `"/choices/0/native_finish_reason"`, `"/service_tier"`, `"/system_fingerprint"`, `"/usage/cost"`, `"/usage/cost_details"`, `"/usage/is_byok"`, `"/usage/prompt_tokens_details"`, `"/usage/server_tool_use_details"`, `"/openrouter_metadata"`, `"/choices/0/message/annotations"`; Responses reads `"/provider"`, `"/service_tier"`, `"/usage/cost"`, `"/usage/cost_details"`, `"/usage/is_byok"`, `"/usage/input_tokens_details"`.
- Quote (`:630-631`): "/// The upstream provider that served the request. Chat; Responses when" / "/// sent."
- Source: G-s3. Basis: Filled (rig).

**G33. rig types the whole `provider` routing object and sends it on both routes.**

- Quote (`extension.rs:499-536`): fields `order`, `only`, `ignore`, `allow_fallbacks`, `require_parameters`, `data_collection`, `zdr`, `sort`, `preferred_min_throughput`, `preferred_max_latency`, `max_price`, `quantizations`; and `:341-342` "/// A throughput floor in tokens per second, alone or per percentile." / "/// Providers below it are tried later, not excluded." and `:442-443` "/// Price ceilings, in USD per million tokens or per item. A request no" / "/// eligible provider can serve under them fails."
- Source: G-s3. Basis: Filled (rig). Agrees with LC8.

**G34. rig decides image support on OpenRouter by the vendor prefix of the model id.**

- Quote (`wire/chat.rs:1067-1076`): "/// list. OpenRouter names a model `vendor/model`: its own catalog entry" / "/// decides, or failing that the vendor's naming rule." and `Some((vendor, upstream)) if dialect.quirks.rewrite == BodyRewrite::OpenRouter => facts.reads_images_or(dialect.name, model, |_| { image_rule(vendor).is_some_and(|(_, rule)| rule(upstream)) }),`
- Source: G-s3. Basis: Filled (rig).

**G35. rig sends an empty `reasoning_content` field for models that require it, looked up under the full OpenRouter id.**

- Quote (`wire/chat.rs:169-176`): "/// The field every assistant message carries its reasoning under," ... "/// OpenRouter under its full id (`moonshotai/kimi-k2.6` behind any" / "/// gateway)."
- Quote (test `wire/chat/history_tests.rs:655-657`): `wire(&OPENROUTER, "moonshotai/kimi-k2.6"),` / "// A gateway's own row for Kimi K3 names no field; Moonshot's does." / `wire(&OPENROUTER, "moonshotai/kimi-k3"),` with the assertion that `body["messages"][1]["reasoning_content"]` equals `json!("")`.
- Source: G-s3. Basis: Filled (rig). Implication (OR-inferred, low confidence): the upstream's "reasoning_content must be present" rule for Kimi still applies through OpenRouter.

**G36. Through OpenRouter's Anthropic Messages surface, `message_start` reports zero input tokens and the count arrives later.**

- Quote (`crates/rig-core/src/providers/anthropic/streaming/tests.rs:661-663`): "// OpenRouter's Anthropic Messages shape: `message_start`" / "// reports a placeholder zero and the real prompt size lands on" / "// the terminal `message_delta`."
- Source: G-s3. Basis: OR-inferred, from rig's test comment. Confidence: medium (a comment over a synthetic test, not a recorded fixture).

### rig: recorded OpenRouter traffic (fixtures at the same commit)

**G37. Recorded Claude replies hold a signed `reasoning.text` entry, and from 2026-10-02 the signature has an OpenRouter suffix that names the upstream endpoint.**

- Quote (`fixtures/cassettes/openrouter/upstream_switch_matrix/switch_unary.yaml:16`, response `gen-1790972129-...`, recorded 2026-10-02 20:15 UTC): `"reasoning_details":[{"format":"anthropic-claude-v1","index":0,"signature":"EsoFCpwBCBIQARgCKkBM...IgGAE=.eyJlbmRwb2ludF9zbHVnIjoiYW50aHJvcGljL2NsYXVkZS00LjUtaGFpa3UtMjAyNTEwMDF8YW1hem9uLWJlZHJvY2svZ2xvYmFsIn0","text":"The user is asking me to:...","type":"reasoning.text"}]` (signature middle elided by me).
- The part after the last `.` decodes (base64url) to `{"endpoint_slug":"anthropic/claude-4.5-haiku-20251001|amazon-bedrock/global"}`. Across fixtures the decoded suffixes are `...|anthropic`, `...|amazon-bedrock`, `...|amazon-bedrock/global`, and `...|google-vertex/global`, each matching the reply's `provider` field (`Anthropic`, `Amazon Bedrock`, `Google`).
- Fixtures recorded 2026-09-22 (`reasoning_tool_order_matrix/blocking_single.yaml`, `streaming_single.yaml`) hold signatures with no `.` suffix (868 and 724 characters, starting `EoQFCroBCBIYAipA`). Fixtures recorded 2026-10-02 and 2026-10-03 all have the suffix.
- The same reply also has the text twice: `"reasoning":"The user is asking me to:..."` beside the entry's `"text"`.
- OpenRouter's documentation mirror has zero occurrences of `endpoint_slug` (searched all 559 pages).
- Source: G-s4 (census script output kept in my scratch folder). Basis: OR-inferred, from recorded traffic in rig's fixtures. Confidence: high that the suffix exists in these recordings; medium that the start date lies between 2026-09-22 and 2026-10-02 (two Claude fixtures recorded before, eight after).

**G38. Recorded switch test: Claude's signed reasoning is not sent to Gemini, and goes back to Claude intact after the Gemini turn.**

- Quote (`crates/rig-cassette/tests/providers/openrouter/cassette/upstream_switch_matrix.rs:1-5`): "//! Reasoning through OpenRouter replays only to the model that produced it." / "//! A conversation that moves to another model, of any family, sends that" / "//! model the reasoning as plain text and no signature or ciphertext, and the" / "//! home model's signed reasoning returns intact when the conversation comes" / "//! back."
- Recorded request 2 (`switch_unary.yaml:27`, to `google/gemini-3-flash-preview`) holds the Claude turn as `{"content":"The user is asking me to:...I'll call the lookup_code function...","role":"assistant","tool_calls":[{"function":{"arguments":"{\"record\":\"alpha\"}","name":"lookup_code"},"id":"toolu_bdrk_01VzaKHoRgZVCVCqesM54vz6","type":"function"}]}` with the thinking text folded into `content` and no `reasoning_details`. It returned 200.
- Recorded request 3 (`:44`, back to `anthropic/claude-haiku-4.5`) holds the first turn with its original `reasoning_details` entry and signature, and the Gemini turn as plain `content`. It returned 200 with `"provider":"Amazon Bedrock"`.
- Source: G-s4. Basis: Filled (rig) for the rule; OR-inferred from recorded traffic for "OpenRouter accepts a history where one assistant turn has signed details and a later one has none". Confidence: high.

**G39. Recorded on three routes: Claude's signed `reasoning_details` are accepted after the tool list changes.**

- Quote (`cassette/context_binding.rs:1-5`): "//! Live fact behind Rig replaying OpenRouter Claude turns verbatim: on each" / "//! route OpenRouter serves Claude Opus 5.5 from (Anthropic, Amazon Bedrock and" / "//! Google Vertex), a turn's signed `reasoning_details` go back after the tool" / "//! list changes, and the request succeeds. So Chat does not bind context on" / "//! OpenRouter."
- Fixtures `context_binding/anthropic.yaml`, `amazon_bedrock.yaml`, `google_vertex.yaml` (recorded 2026-10-03 15:34 UTC): each has two exchanges with status 200, `provider` equal to `Anthropic`, `Amazon Bedrock`, `Google`, model `anthropic/claude-opus-5.5`.
- Also `reasoning_tool_order_matrix/streaming_signed_agent_roundtrip.yaml` (2026-10-02): the follow-up request replays the streamed signature and returns 200, pinned with `"provider":{"order":["Anthropic"],"allow_fallbacks":false}`.
- Source: G-s4. Basis: OR-inferred, from recorded traffic. Confidence: high. This contradicts Zed's comment in G1.

**G40. Recorded tool-call ids are the upstream's own.**

- Chat route ids by upstream: `toolu_01BFY3SG28Vx...` (`provider: Anthropic`), `toolu_bdrk_01Hwh6y...` (`Amazon Bedrock`), `toolu_vrtx_01Xc6A4...` (`Google` serving Claude), `call_izjasbuE1Bsg...` and `call_NcNQgk31ThHo...` (`OpenAI`).
- `portability_matrix/from_openai_responses.yaml` (2026-10-02): a history whose tool call id is `call_izjasbuE1Bsg...` was sent to `anthropic/claude-haiku-4.5`, served by `Amazon Bedrock`, status 200.
- Source: G-s4 (census output). Basis: OR-inferred, from recorded traffic. Confidence: high for pass-through; one data point for the cross-vendor id being accepted.

**G41. Recorded `native_finish_reason` values, and evidence that OpenAI reasoning models are served from the Responses API.**

- Pairs seen in recorded streams (`finish_reason`, `native_finish_reason`): (`tool_calls`, `tool_use`) and (`stop`, `end_turn`) for Claude; (`stop`, `STOP`) for Gemini; (`tool_calls`, `completed`) and (`stop`, `completed`) for `openai/o4-mini` and `openai/gpt-5.2`; (`stop`, `stop`) for `openai/gpt-4o-mini`; `max_output_tokens` on a truncated `openai/o4-mini` reply.
- The `openai/gpt-5.2` and `openai/o4-mini` replies have `reasoning_details` with `"format":"openai-responses-v1"` and types `reasoning.encrypted` and `reasoning.summary`. Recorded entries (`reasoning_roundtrip/nonstreaming.yaml`, 2026-10-02, values trimmed by me): `{"format": "openai-responses-v1", "index": 0, "summary": "**Calculating meeting time**...", "type": "reasoning.summary"}` and `{"data": "gAAAAABqwBArBYP7e3J1TDVWIkg9lsqZABsxphWK...", "format": "openai-responses-v1", "id": "rs_02de745d9b1e8fa7016ac01027c6e887d18f8...", "index": 1, "type": "reasoning.encrypted"}`. The message's plain `reasoning` string equals the summary text.
- rig's next request in that fixture replays both entries unchanged, with the `rs_` id, beside `reasoning`, and gets 200. LangChain removes `rs_` ids before replay (G98), so both forms are in use.
- Source: G-s4 (census and usage scripts). Basis: OR-inferred, from recorded traffic. Confidence: high for the values; the inference "OpenRouter calls OpenAI's Responses API upstream for these chat requests" rests on `completed` and `max_output_tokens` being Responses statuses and on the format tag. Medium.

**G42. OpenRouter accepts `n = 2` and returns one candidate.**

- Quote (`cassette/terminal_metadata_matrix.rs:10-12`): "//! OpenRouter accepts `n = 2` but emits only candidate zero, and those cells" / "//! pin that observed gateway contract instead of assuming direct OpenAI" / "//! behavior."
- Source: G-s4. Basis: OR-inferred, from rig's test comment over its fixtures. Confidence: medium (I did not open those six fixtures).

**G43. `reasoning.exclude: true` hides readable reasoning and still returns the encrypted entry on the OpenAI route.**

- Quote (`cassette/reasoning_usage_matrix.rs:48-53`): "/// `reasoning.exclude: true` asks OpenRouter not to *show* the reasoning, and" / "/// it changes neither the billing nor this mapping. Recorded census, from this" / "/// cell's own fixture: on the OpenAI route `exclude` still returns the" / "/// `reasoning.encrypted` detail (only the human-readable summary is withheld),"
- Fixture `reasoning_usage_matrix/blocking_excluded_reasoning_still_counts_tokens.yaml` (2026-09-22): types present are `reasoning.encrypted` only.
- Source: G-s4. Basis: OR-inferred, from recorded traffic. Confidence: high.

**G44. Recorded shape of the streamed usage chunk and of `usage`.**

- In four recorded streams (2026-10-02) the chunk that holds `usage` has `"choices":[{"delta":{"content":"","role":"assistant"},"finish_reason":"tool_calls","index":0,"native_finish_reason":"tool_use"}]`, so the finish reason appears on two chunks: the finish chunk and the usage chunk.
- `usage` keys: `completion_tokens`, `completion_tokens_details`, `cost`, `cost_details`, `is_byok`, `prompt_tokens`, `prompt_tokens_details`, `total_tokens`.
- Unary example (`switch_unary.yaml:16`): `"usage":{"completion_tokens":190,"completion_tokens_details":{"audio_tokens":0,"image_tokens":0,"reasoning_tokens":116},"cost":0.001558,"cost_details":{"upstream_inference_completions_cost":0.00095,"upstream_inference_cost":0.001558,"upstream_inference_prompt_cost":0.000608},"is_byok":false,"prompt_tokens":608,"prompt_tokens_details":{"audio_tokens":0,"cache_write_tokens":0,"cached_tokens":0,"video_tokens":0},"total_tokens":798}`
- rig's dialect does not send `stream_options` (G25), and the usage still arrives. Its comment: "//! OpenRouter always reports usage, and every reasoning route reports the" / "//! breakdown as `usage.completion_tokens_details.reasoning_tokens`." (`reasoning_usage_matrix.rs:3-4`).
- Source: G-s4. Basis: OR-inferred, from recorded traffic. Confidence: high. Confirms LC12 and LC17.

**G45. Other recorded details: unary replies keep the stream `index` on tool calls, top-level reply fields, and the request-id header.**

- Quote (`crates/rig-core/src/test_utils/raw_parity/tests.rs:389-390`): "// Each answer reasons in its own words, and OpenRouter's unary body" / "// keeps the stream `index` on its calls." Seen in `switch_unary.yaml:16`: `"tool_calls":[{"function":{...},"id":"toolu_bdrk_01VzaKHoRgZVCVCqesM54vz6","index":0,"type":"function"}]`.
- Unary top-level fields: `"model":"anthropic/claude-haiku-4.5","object":"chat.completion","provider":"Amazon Bedrock","service_tier":"default","system_fingerprint":null`.
- Quote (`cassette/raw_capture_matrix.rs:13-14`): "OpenRouter contracts no" / "//! request-id header, so `provider_request_id` is `None` on every turn."
- Source: G-s4. Basis: OR-inferred, from recorded traffic and rig's comments. Confidence: high for the first two, medium for the header (a comment).

**G46. Recorded Gemini 3 stream: the signature arrives alone, after the content, as a `reasoning.text` entry with no text.**

- Quote (`upstream_switch_matrix/switch_streamed.yaml`, response `gen-1790972115-...`): `{"choices":[{"delta":{"content":"","reasoning_details":[{"format":"google-gemini-v1","index":0,"signature":"AY89a19QiA2xoKvfVf9xoeiRvpyOPIzSO188TDrn/lJkGATbpvhpjLr/62Z2pxCwkOegYrTKtpgQmdYpeyJaswt4r0VRJCqYtD2JpNNmvsjx/G12bJ3BhA7v","type":"reasoning.text"}],"role":"assistant"},"finish_reason":null,"index":0,"native_finish_reason":null}],"created":0,"id":"gen-1790972115-wiOgqpatnKpbX90WIgFj","model":"google/gemini-3-flash-preview","object":"chat.completion.chunk","provider":"Google"}`
- Quote (`wire/chat/history_tests.rs:723-732`): "/// Round 5 generated-history finding 2: `reasoning_details` join the" / "/// message's one reasoning block, the turn's first, wherever they arrive," ... "/// the fifth signs the first of two parallel calls, as Gemini 3 does."; the test's call-signing entry is `{"type": "reasoning.encrypted", "data": "gsig", "format": "google-gemini-v1", "index": 0, "id": "call_1"}`.
- The Gemini signature has no OpenRouter suffix (120 characters, no `.`). One recorded stream also has the SSE comment line `: OPENROUTER PROCESSING`.
- Source: G-s4, G-s3. Basis: OR-inferred, from recorded traffic. Confidence: high.

### genai (Rust) - `jeremychone/rust-genai` at `2a8a5b3`

**G47. genai stores each `reasoning_details` entry as a verbatim `Custom` content part tagged with the model, ahead of text and tool calls.**

- Quote (`src/adapter/adapters/openai/adapter_impl.rs:105-118`): "Capture reasoning_details as Custom parts, verbatim (OpenRouter)" ... "// array beside the plaintext `reasoning`, and requires that array back" / "// verbatim and in order for a multi-turn tool round trip to keep the" / "// model's continuity token." and `content.push(ContentPart::from_custom(detail, Some(model_iden.clone())));`
- Source: G-s5. Basis: Filled (genai). Storage shape: `ContentPart::Custom(CustomPart { model_iden: Option<ModelIden>, data: serde_json::Value })`, one per entry, where `ModelIden` is adapter kind plus model name.

**G48. genai sends the entries back only when the part came from the same adapter kind, because strict upstreams reject the field.**

- Quote (`src/adapter/adapters/openai/adapter_shared.rs:500-509`): "// OpenRouter's reasoning blocks, captured verbatim by the response path," / "// go back verbatim: a signed or encrypted block cannot be rebuilt from" / "// the plaintext, and the gateway requires the original sequence. Blocks" / "// from another adapter stay behind: strict providers refuse the field" / "// (Groq with a 400, Mistral with a 422)." and `if custom.adapter_kind().is_none_or(|kind| kind == model_iden.adapter_kind) { reasoning_details.push(custom.data) }`
- Quote (`:541-546`): "// Echo reasoning_details back for OpenRouter, which needs the provider's" ... `message.x_insert("reasoning_details", reasoning_details)?;`
- Source: G-s5. Basis: Filled (genai). Note the check is on adapter kind (OpenRouter), not on model, so a Claude entry would be sent to a Gemini model through OpenRouter. rig and Pydantic AI differ here (G27, G81).

**G49. genai's merge rules for streamed `reasoning_details` describe how OpenRouter fragments them.**

- Quote (`src/adapter/adapters/openai/streamer.rs:455-460`): "/// OpenRouter streams a `reasoning.text` or `reasoning.summary` block in fragments, with a text" / "/// block's `signature` on a last, text-less piece. A fragment extends the last block when it has" / "/// the same type, unless that block is signed (the signature closes the block it signs, so two" / "/// signed thinking blocks stay apart) or the fragment's content has a different `index`. The same" / "/// `index` says nothing, since some providers send `index: 0` on every piece. Any other piece" / "/// (`reasoning.encrypted`) is a whole block."
- Quote (test `:544-549`): "/// OpenRouter streams unsigned reasoning one token per piece, all at `index: 0`" / "/// (frames trimmed from a live `nvidia/nemotron-3.5-lightning:free` stream)." with frame `[{"type": "reasoning.text", "text": "The", "format": "unknown", "index": 0}]`
- Source: G-s5. Basis: OR-inferred, from genai's merge code and test frames. Confidence: high; the Claude and Gemini frames in G37 and G46 fit the same rule.

**G50. genai's Anthropic adapter drops OpenRouter's reasoning entries when a conversation moves to the native Anthropic API.**

- Quote (`src/adapter/adapters/anthropic/adapter_shared.rs:360-362`): "// OpenRouter's reasoning blocks aren't Anthropic content: it refuses" / "// their `type` with a 400 when a conversation moves here." and `ContentPart::Custom(custom_part) if is_reasoning_detail(&custom_part) => {}`
- Source: G-s5. Basis: Filled (genai). With G37 this means a signature obtained through OpenRouter stays an OpenRouter-only value in this client.

**G51. genai marks cache points on the shared OpenAI wire with `prompt_cache_breakpoint`, for OpenRouter too.**

- Quote (`adapter_shared.rs:688`): `content_block.x_insert("prompt_cache_breakpoint", json!({"mode": "explicit"}))?;`
- Source: G-s5. Basis: Filled (genai). It relies on OpenRouter's documented conversion of `prompt_cache_breakpoint` to a `cache_control` breakpoint toward Anthropic and Google (G-s11, LC7).

**G52. genai's OpenRouter adapter is the OpenAI adapter with another endpoint and key.**

- Quote (`src/adapter/adapters/all_adapters.rs:193-199`): `pub struct OpenRouterAdapter;` ... `kind: AdapterKind::OpenRouter, key_env: Some("OPEN_ROUTER_API_KEY"), endpoint: "https://openrouter.ai/api/v1/",`
- Source: G-s5. Basis: Filled (genai).

### Aider (Python) - `Aider-AI/aider` at `5dc9490` (same commit the 2026-11-17 survey read; last commit 2026-05-22)

Aider reaches OpenRouter through LiteLLM (`openrouter/<vendor>/<model>` names), so its own OpenRouter code is settings and metadata.

**G53. Aider writes OpenRouter's `reasoning` object where other providers get `reasoning_effort` or Anthropic's `thinking`.**

- Quote (`aider/models.py:793-799`): `if self.name.startswith("openrouter/"): ... self.extra_params["extra_body"]["reasoning"] = {"effort": effort}`
- Quote (`:850-857`): "# OpenRouter models use 'reasoning' instead of 'thinking'" and `self.extra_params["extra_body"]["reasoning"] = {"max_tokens": num_tokens}`
- Quote (`:424-436`): "# Ensure OpenRouter models accept thinking_tokens and reasoning_effort" with an exclusion for names holding `claude-opus-4.7` or `claude-opus-4-7`.
- Source: G-s6. Basis: Filled (Aider).

**G54. Aider reads context size and price from the public catalog, with a 24 hour cache and a page-scrape fallback.**

- Quote (`aider/openrouter.py:30-31`, `:69-73`, `:75-83`): `MODELS_URL = "https://openrouter.ai/api/v1/models"` / `CACHE_TTL = 60 * 60 * 24  # 24 h`; `record.get("top_provider", {}).get("context_length") or record.get("context_length")`; `"input_cost_per_token": _cost_per_token(pricing.get("prompt")), "output_cost_per_token": _cost_per_token(pricing.get("completion")),`
- Quote (`:60-63`): "# Consider both the exact id and id without any “:suffix”." and `candidates.add(route.split(":", 1)[0])`
- Source: G-s6. Basis: Filled (Aider). The catalog request is made without an auth header, which agrees with LC1.

**G55. Aider special-cases two OpenRouter failures by matching text.**

- Quote (`aider/exceptions.py:92-99`): `if "OpenrouterException" in str(ex) and "'choices'" in str(ex): return ExInfo("APIConnectionError", True, ("OpenRouter or the upstream API provider is down, overloaded or rate" " limiting your requests."),)`
- Quote (`:103-109`): `if "insufficient credits" in err_str and '"code":402' in err_str: return ExInfo("APIError", False, "Insufficient credits with the API provider. Please add credits.",)`
- Source: G-s6. Basis: OR-inferred, from Aider's exception matcher: a reply body without `choices` is a known transient outcome (retryable), and the credit error body holds `"code":402`. Confidence: medium. Fits G84 and G108.

**G56. Aider's settings for `openrouter/anthropic/*` models turn on `cache_control` and send an `anthropic-beta` header.**

- Quote (`aider/resources/model-settings.yml:330-343`): `- name: openrouter/anthropic/claude-3.7-sonnet` ... `extra_params: extra_headers: anthropic-beta: prompt-caching-2024-07-31,pdfs-2024-09-25,output-128k-2025-02-19 max_tokens: 64000 cache_control: true`
- Count: 42 `anthropic-beta` entries in the file, 11 of them on `openrouter/` model names (counted by me with a 12-line lookback, so approximate).
- Source: G-s6. Basis: Filled (Aider). Whether OpenRouter forwards `anthropic-beta` to Anthropic is not shown by this file; see Hermes G77 for a second client that depends on it.

**G57. Aider asks for reasoning text on DeepSeek R1 with `include_reasoning: true`.**

- Quote (`model-settings.yml:548-556`): `- name: openrouter/deepseek/deepseek-r1` ... `extra_params: max_tokens: 8192 include_reasoning: true`
- Source: G-s6. Basis: Filled (Aider). rig's fixture test for `openai/o4-mini` also sends `include_reasoning: true` (`cassette/streaming_tools.rs:4-5`).

**G58. Aider has an OpenRouter sign-in flow and a free-tier check.**

- Quote (`aider/onboarding.py:31`, `:37-38`, `:178`): `"https://openrouter.ai/api/v1/auth/key",` / "# According to the documentation, 'is_free_tier' will be true if the user has never paid" / `"https://openrouter.ai/api/v1/auth/keys",` with `"code_challenge_method": "S256"`.
- Source: G-s6. Basis: Filled (Aider). Not read by me beyond these lines, since auth is outside this topic.

**G59. Aider does not replay reasoning through OpenRouter.**

- The workspace's 2026-11-17 survey lists Aider's thinking replay as "None" (Table E1), and at the same commit `aider/` has zero occurrences of `reasoning_details` or `thinking_blocks` in its Python files (searched).
- Source: G-s6, G-s15. Basis: Workspace, re-checked against the same pinned commit today.

### Hermes Agent (Python) - `NousResearch/hermes-agent` at `0f602c5`

Hermes has an OpenRouter provider profile (`plugins/model-providers/openrouter/__init__.py`) and many OpenRouter branches in its shared chat-completions code. The folder `agent/` alone has 53 Python files that name OpenRouter (211 non-test Python files in the whole repository). Many comments cite Hermes issue numbers and live measurements.

**G60. Hermes replays three reasoning fields to OpenRouter-format gateways and strips them for strict upstreams.**

- Quote (`agent/message_sanitization.py:697-699`): "# OpenRouter-format gateways read the unified array, the string and the alias." `(_ALL_CARRIERS, frozenset({"openrouter", "kilocode", "ai-gateway", "nous"}), ("openrouter.ai", "kilo.ai", "ai-gateway.vercel.sh", "nousresearch.com")),`
- Quote (`:684-686`): `RC, R, RD = "reasoning_content", "reasoning", "reasoning_details"`
- Quote (`agent/transports/chat_completions.py:306-309`): "``reasoning_details`` to OpenRouter-format gateways (OpenRouter, Kilo, Vercel AI Gateway," / "Nous Portal) and to vendors documenting it (Novita, MiniMax /v1, TokenHub), never to strict" / "schemas (Groq, Mistral, Cerebras, opencode relays: ``property 'reasoning_details' is" / "unsupported`` / ``Extra inputs are not permitted`` / ``no such field``, #70233)."
- Source: G-s7. Basis: Filled (Hermes).

**G61. Hermes stores `reasoning_details` entries byte for byte on the assistant message and learned that dropping the streamed ones lost continuity.**

- Quote (`agent/chat_completion_helpers.py:3065-3068`): "# Structured reasoning_details deltas carry the provider's replay data; the" / "# non-streaming path already keeps them, so dropping them here lost" / "# reasoning continuity on nearly every turn. Pydantic parks unknown fields" / "# in ``model_extra``."
- Quote (`:1663-1674`): "# Signed/opaque entries are replayed byte-exact; only unsigned text is redacted below." and `msg["reasoning_details"] = preserved`
- Quote (`:1677-1680`): "# Provider-native carriers replayed verbatim on later turns:" / "# anthropic_content_blocks keeps interleaved thinking + tool_use order" / "# (reconstruction reorders signed blocks -> HTTP 400); codex_* items are" / "# the encrypted reasoning / exact message items Responses prefix caching"
- Source: G-s7. Basis: Filled (Hermes). Storage shape: the stored assistant message dict keeps `reasoning_details` (list of raw dicts), `reasoning_content` (string), and, for native routes, `anthropic_content_blocks` or `codex_*` items. The OpenAI SDK's typed objects drop the unknown field unless read from `model_extra`.

**G62. Hermes checks a Gemini signature sidecar before replaying it, because an empty one fails the request.**

- Quote (`agent/transports/chat_completions.py:321-335`): "Whether OpenRouter's Gemini sidecar contains a usable thought signature." ... "Replaying an empty or non-string value makes a multimodal" / "request fail with ``Corrupted thought signature``; omit that sidecar while" / "leaving the stored history untouched." and `candidate = extra_content.get("thought_signature")` / `candidate = google.get("thought_signature")`
- Source: G-s7. Basis: OR-inferred, from Hermes' guard: OpenRouter's Gemini tool calls can arrive with `extra_content` holding `thought_signature` directly or under `google`. Confidence: medium.

**G63. For Claude models that always think, Hermes omits `reasoning` and sends the effort as top-level `verbosity`.**

- Quote (`plugins/model-providers/openrouter/__init__.py:162-185`): "# Reasoning-mandatory Anthropic models use adaptive thinking: any" / "# ``reasoning`` field (disable, or an enabled form on a tool-continuation" / "# turn without a replayed thinking block) makes OpenRouter emit" / "# ``thinking: {type: \"disabled\"}`` -> 400. Omit it; the user's effort" / "# still reaches Anthropic's output_config.effort via top-level ``verbosity``." and later "OpenRouter honors the requested effort on the top-level ``verbosity`` field instead (it maps" / "# to Anthropic's ``output_config.effort``; ``reasoning.effort`` is accepted but ignored" and `top_level["verbosity"] = effort`
- Quote (same block): "(chat_completions never replays signed thinking" / "# blocks)".
- Source: G-s7. Basis: OR-inferred, from Hermes' code comments, which cite issues 42991 and 43432 (G110, G111). Confidence: medium to high; the token-spend table in G111 backs the `verbosity` claim. The last quote says Hermes' chat-completions path does not replay signed thinking for these models, which sits oddly beside G60 and G61; I did not resolve which applies when.

**G64. Hermes clamps reasoning effort to the catalog's `reasoning.supported_efforts`, and found one catalog entry wrong.**

- Quote (`__init__.py:70-73`): "OpenRouter's /v1/models entries publish ``reasoning.supported_efforts``" / "per model (ported from PrimeIntellect-ai/prime-agent#1258). Sending an" / "unsupported effort (e.g. ``ultra`` to a route that stops at ``high``)" / "yields provider 4xx errors; clamp to the nearest LOWER supported level"
- Quote (`:88-92`): "# A reasoning-mandatory route 400s on a disable (\"Reasoning is" / "# mandatory for this endpoint and cannot be disabled\")" ... "# OpenRouter's catalog lists ``none`` for openai/gpt-6.1-sol, but upstream 400s on it" / "# (live 2026-09-29), so the OpenAI ladder in agent.reasoning_effort wins over the catalog."
- Quote (`agent/reasoning_params.py:93-94`): "True when reasoning extra_body is safe to send: OpenRouter forwards unknown extra_body upstream and" / "some routes 400 on ``reasoning``, so gate to known reasoning-capable families and direct Nous Portal."
- Source: G-s7. Basis: OR-inferred, from Hermes' code comments with a live date. Confidence: medium. The second quote disagrees with LC3 ("unsupported parameters are silently ignored") for at least some routes.

**G65. Hermes treats OpenAI speed tiers as OpenRouter endpoints and pins them with `provider.only`.**

- Quote (`__init__.py:47-52`): "# OpenAI speed tiers. Nous Portal serves them as distinct slugs (``-fast``/``-flex``); OpenRouter" / "# serves them as ENDPOINTS of the base model (tags ``openai/fast``, ``openai/flex``) and silently" / "# routes an unknown suffix to the standard tier at standard price." and `_SPEED_TIER_ENDPOINTS = {"": ("openai", "azure", "azure/us"), "-fast": ("openai/fast",), "-flex": ("openai/flex",)}`
- Source: G-s7. Basis: OR-inferred, from Hermes' code comment. Confidence: medium.

**G66. Hermes sends `session_id` as the sticky routing key, validated `provider` preferences, and a router plugin.**

- Quote (`__init__.py:128-129`): "# Top-level session_id is OpenRouter's sticky routing key (used directly," / "# not hashed from the opening messages; active from the first request)."
- Quote (`:149`): `body["plugins"] = [{"id": "pareto-router", "min_coding_score": score_f}]` for model `openrouter/pareto-code`.
- Quote (`agent/chat_completion_helpers.py:60`, `:483`): `_OPENROUTER_PROVIDER_SORT_VALUES = {"throughput", "latency", "price"}` and `merged["require_parameters"] = True if merged["require_parameters"] else None`
- Source: G-s7. Basis: Filled (Hermes); the sticky-key remark is OR-inferred from the comment, medium confidence.

**G67. Hermes puts `cache_control` on the message envelope for Claude and Kimi through OpenRouter, with rules for tool messages.**

- Quote (`agent/agent_runtime_helpers.py:1351-1353`, `:1405-1406`): "# Kimi/Moonshot via OpenRouter takes Claude's envelope cache_control; without it moonshotai/kimi-k2.6" / "# served ~1% hits on 64K prompts (1% → 67% → 84% → 97% with it, #25970)." and `if (is_openrouter or is_nous_portal) and (is_claude or is_kimi) and not is_anthropic_wire: return True, False`
- Quote (`agent/prompt_caching.py:28-30`): "OpenRouter/Nous Portal relocate a part-level ``cache_control`` onto the ``tool_result``" / "block; LiteLLM-style proxies copy parts verbatim, so it lands at ``tool_result.content[0]``" / "(non-retryable 400)."
- Quote (`:70-71`): "# content: no part can carry it, and OpenRouter rejects a top-level marker on role:tool" / "# (silent hang) and ignores it on empty assistant turns".
- Quote (`:3-5`): "Default layout: 4 cache_control breakpoints".
- Source: G-s7. Basis: OR-inferred, from Hermes' code comments and issue 25970 (G112). Confidence: medium. These describe translation behavior that OpenRouter's pages do not state.

**G68. Hermes pins xAI's cache through OpenRouter with a header.**

- Quote (`__init__.py:196-203`): "# xAI's prompt cache is pinned per backend server via this header." and `if grok_conv_id and model and model.startswith(GROK_AGGREGATOR_MODEL_PREFIXES): top_level["extra_headers"] = {"x-grok-conv-id": grok_conv_id}`
- Source: G-s7. Basis: Filled (Hermes). It assumes OpenRouter forwards `x-grok-conv-id` to xAI; not verified by me.

**G69. Hermes handles OpenRouter's response cache with a request header and a reply header.**

- Quote (`agent/turn_api_request.py:157-160`): "# OpenRouter caching replays identical responses, even empty ones; an empty-response" / "# retry must bypass the cache." and `_set_extra_header(api_kwargs, "X-OpenRouter-Cache", "false")`
- Quote (`agent/rate_limit_credits.py:235`, `:240`): "Log X-OpenRouter-Cache-Status; HITs count in ``_or_cache_hits``." and `status = headers.get("x-openrouter-cache-status")`
- Source: G-s7. Basis: OR-inferred, from Hermes' code. Confidence: medium.

**G70. Hermes measured cache rewrites on OpenRouter's Anthropic Messages surface and keeps Claude on chat completions there.**

- Quote (`agent/nous_wire.py:3-8`): "Portal serves Claude two ways and Hermes cannot tell which from the request: an OpenRouter" / "passthrough (today, for every ``anthropic/*`` id) or GMI/Vertex (planned once GMI is back). The" / "native Messages wire is the better transport, but on the OpenRouter path it re-writes the previous" / "turn's prompt cache on 14-20% of consecutive calls in concurrent tool loops (measured 2026-09-06;" / "NousResearch/api#227), so the session must ride chat/completions there."
- Quote (`:10-11`, `:31`): "The upstream IS visible in the first RESPONSE: OpenRouter stamps ``provider`` (chat wire) and" / "mints ``gen-<unix>-<rand>`` ids" and `_OPENROUTER_ID = re.compile(r"^gen-\d{9,}-[A-Za-z0-9_-]{8,}$")`
- Source: G-s7. Basis: OR-inferred, from Hermes' module docstring. Confidence: low to medium: the measurement is through Nous Portal, which passes through to OpenRouter, and the cited issue is in a repository I could not read.

**G71. OpenRouter's Claude usage gives one cache-write number, so Hermes prices it by the TTL it asked for.**

- Quote (`agent/turn_usage.py:127-128`): "A route that honours the tier it was sent but reports no split (OpenRouter Claude) bills its" / "writes at the requested tier: name it so pricing charges a 1h write at the 1h rate."
- Source: G-s7. Basis: OR-inferred, from Hermes' code comment. Confidence: medium. Fits the single `cache_write_tokens` field in G44.

**G72. Hermes estimates OpenRouter cost from the catalog and guards against a negative price.**

- Quote (`agent/usage_pricing.py:590-591`): "# A negative rate is a \"pricing varies\" sentinel (OpenRouter publishes \"-1\" for" / "# openrouter/auto), not a price: priced through, it books huge negative session costs."
- Quote (`:863-864`): `notes.append("OpenRouter cost is estimated from the models API until reconciled.")`
- Source: G-s7. Basis: OR-inferred, from Hermes' code comment. Confidence: medium.

**G73. Hermes unwraps OpenRouter's "Provider returned error" body to reach the upstream's message.**

- Quote (`agent/error_classifier.py:1591-1595`): "Lowercased inner message from OpenRouter's ``error.metadata.raw`` JSON wrapper." and `inner = _json_dict(metadata.get("raw")) if isinstance(metadata, dict) else None`
- Quote (`:1742-1752`): "OpenRouter's \"Provider returned error\" wrapper: the key is healthy, the" / "upstream failed, so credential rotation is the wrong recovery." and `return isinstance(metadata, dict) and ("raw" in metadata or "provider_name" in metadata)`
- Quote (`:1012`): "# the upstream cannot open). Not gated on provider" followed in the same comment by "OpenRouter proxies Anthropic errors."
- Source: G-s7. Basis: OR-inferred, from Hermes' classifier. Confidence: high; the same shape is in G21, G108, G114.

**G74. Hermes maps specific OpenRouter status and text pairs to recovery actions.**

- Quote (`error_classifier.py:121-123`): "# OpenRouter org-level monthly cap arrives as 403 \"Budget limit exceeded (monthly limit)\" (#107166):" / "# account exhaustion, not a credential problem." / `"budget limit exceeded",`
- Quote (`:333-334`, `:352`): "# Last entry: OpenRouter 404 when no endpoint supports tool calling" and `"no endpoints found that support tool use",`
- Quote (`:429-433`): "# OpenRouter 404 when the account data policy excludes the only endpoint." with patterns `"no endpoints available matching your guardrail", "no endpoints available matching your data policy", "no endpoints found matching your data policy",`
- Quote (`:1273-1277`): "# OpenRouter-wrapped upstream 429: the key is healthy" and `return _v(_R.upstream_rate_limit, should_fallback=True, error_context=ctx)`
- Quote (`agent/turn_recovery.py:604-606`): "A credit-limited 402 (OpenRouter: \"...requested up to 65536 tokens, but can only afford" / "56272\") means the account has credit and the output cap was too large: retry ONCE with the" / "cap lowered to the affordable budget".
- Quote (`error_classifier.py:455-461`): "# Aggregator prompt-injection guardrails (OpenRouter Guardrails' regex detector)" with pattern `"prompt injection patterns detected"`.
- Source: G-s7. Basis: OR-inferred, from Hermes' classifier strings. Confidence: medium to high (each cites an issue). LC13 says budget rejections are 403, which this agrees with.

**G75. Hermes treats an `error` chunk in the stream and the finish reasons `error` and `error_finish` as stream failures.**

- Quote (`agent/chat_completion_helpers.py:2346-2349`): "# SSE error events from proxies (OpenRouter's {\"error\":{\"message\":\"Network" / "# connection lost.\"}}) surface as SDK APIError without a status_code (unlike" / "# APIStatusError). They mean the upstream stream died: retry with a fresh" / "# connection like an httpx drop."
- Quote (`:61`): `_PROVIDER_STREAM_ERROR_FINISH_REASONS = {"error", "error_finish"}`
- Quote (`error_classifier.py:472-476`): "# Empty-response advisories (OpenRouter / nano-gpt). Checked before overflow" with patterns `"returned an empty response", "empty response despite retries", "provider returned an empty response",`
- Source: G-s7. Basis: OR-inferred, from Hermes' code. Confidence: high for `error` (LC11, G85, G116); `error_finish` is from this file only.

**G76. Hermes skips its own client rebuild-and-retry step for OpenRouter.**

- Quote (`agent/agent_runtime_helpers.py:739-741`): "transport error. Skipped for aggregators (OpenRouter, Nous) that manage retries server-side." and `if agent._fallback_activated or error_type not in _TRANSIENT_TRANSPORT_ERRORS or agent._is_openrouter_url(): return False`
- Source: G-s7. Basis: Filled (Hermes).

**G77. Hermes sends an Anthropic beta header to OpenRouter for Claude so tool arguments stream.**

- Quote (`agent/agent_init.py:860-869`): "# Fine-grained tool streaming for Claude on OpenRouter: without the beta header" / "# Anthropic buffers the whole tool call and OpenRouter's proxy times out." and `headers["x-anthropic-beta"] = ",".join(filter(None, (existing_beta, _TOOL_STREAMING_BETA)))`
- Source: G-s7. Basis: OR-inferred, from Hermes' code comment: OpenRouter forwards `x-anthropic-beta` to Anthropic. Confidence: low to medium; not confirmed in OpenRouter's pages by me.

**G78. The serving upstream is in the body of each chunk; Hermes also logs `x-openrouter-*` reply headers when present.**

- Quote (`agent/chat_completion_helpers.py:2818-2820`): `upstream_provider = chunk.provider  # OpenRouter stamps who served`
- Quote (`agent/stream_diag.py:20`): `"cf-ray", "cf-cache-status", "x-openrouter-provider", "x-openrouter-model", "x-openrouter-id",`
- Source: G-s7. Basis: OR-inferred, from Hermes' code and issue 90216 (G129). Confidence: high for the body field (matches G37 frames).

**G79. Hermes has 53 Python files under `agent/` that name OpenRouter; its error and retry copy points users at OpenRouter pages.**

- Example (`agent/turn_recovery.py:1390-1399`): `if agent._is_openrouter_url() and "support tool use" in error_msg:` followed by the hint "No OpenRouter providers for {_model} support tool calling with your current settings." and a link to `https://openrouter.ai/models/{_model}`.
- Source: G-s7. Basis: Filled (Hermes). Listed to show the scale of client-side special-casing in one harness.

### Pydantic AI (Python) - `pydantic/pydantic-ai` at `463a6d5`

**G80. Pydantic AI turns each `reasoning_details` entry into a thinking part and back.**

- Quote (`pydantic_ai_slim/pydantic_ai/models/_reasoning_details.py:19-29`): `class BaseReasoningDetail(BaseModel, frozen=True): id: str | None = None format: (Literal['unknown', 'openai-responses-v1', 'anthropic-claude-v1', 'xai-responses-v1', 'google-gemini-v1'] | str | None) = None index: int | None = None type: Literal['reasoning.text', 'reasoning.summary', 'reasoning.encrypted']`
- Quote (`:57-80`): for `ReasoningText`, `ThinkingPart(id=reasoning.id, content=reasoning.text, signature=reasoning.signature, provider_name=provider_name, provider_details=provider_details,)`; for `ReasoningEncrypted`, `ThinkingPart(id=reasoning.id, content='', signature=reasoning.data, provider_name=provider_name, provider_details=provider_details,)`, where `provider_details = reasoning.model_dump(include={'format', 'index', 'type'})`.
- Quote (`models/openrouter.py:1526-1537`): `message_param['reasoning_details'] = self.reasoning_details` and `if self.accepts_stateless_artifacts(item): if reasoning_detail := into_reasoning_detail(item): self.reasoning_details.append(reasoning_detail.model_dump())`
- Source: G-s8. Basis: Filled (Pydantic AI). Storage shape: `ThinkingPart { id, content, signature, provider_name: "openrouter", provider_details: {"format", "index", "type"} }`. Encrypted `data` is stored in the `signature` slot. The entry is rebuilt from typed fields on the way back, so a key outside these fields would be dropped (the model lists `id`, `format`, `index`, `type`, `text`, `signature`, `summary`, `data`).

**G81. Pydantic AI replays those parts only to OpenRouter, and declares no other system compatible.**

- Quote (`pydantic_ai_slim/pydantic_ai/providers/openrouter.py:275-278`): `def canonical_system(self) -> str: # Fixed, so a subclass that only renames itself keeps sharing reasoning details. Which upstream verifies # them depends on where OpenRouter routed the request, so no other provider is declared compatible. return 'openrouter'`
- Quote (`pydantic_ai_slim/pydantic_ai/_canonical_system.py:3-6`): "Stateless round-trip data is data the API verifies on replay by its own key and format, not by looking" / "up stored state: OpenAI encrypted reasoning and compaction items, Anthropic thinking signatures and" / "redacted thinking, Gemini thought signatures, xAI encrypted reasoning and tool results, and OpenRouter" / "or Snowflake reasoning details."
- Source: G-s8. Basis: Filled (Pydantic AI). The check is on the system name, not the model, so like genai it would send a Claude entry to another model through OpenRouter.

**G82. In streams, Pydantic AI keys thinking parts by entry type plus `index`, because Gemini sends two entry types with one index.**

- Quote (`models/openrouter.py:1696-1701`): "# OpenRouter's index is stable across chunks, unlike the position in the current" / "# chunk. It distinguishes separate details while merging deltas for one detail." / "# The type remains part of the identifier because Gemini 3 Pro can emit text and" / "# encrypted details with the same index; those must remain separate ThinkingParts" / "# for thought-signature handling."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comment. Confidence: medium. genai's comment (G49) says the opposite about `index` for some upstreams ("some providers send `index: 0` on every piece"), so the meaning of `index` varies by upstream.

**G83. Pydantic AI maps an error that arrives inside a 200 response to the status its `code` names.**

- Quote (`models/openrouter.py:466-471`): "Map an error OpenRouter sent inside a 200 response or stream." / "Its `code` is the HTTP status OpenRouter uses for the same error, so it's reported as the status code although" / "the response's own status was 200."
- Quote (`:473-475`): `if error.code == 403 and flags_input(error.metadata): # A 403 for flagged input is a rejected request, not a credentials problem.`
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's mapping. Confidence: high.

**G84. Pydantic AI recognizes three malformed reply shapes that OpenRouter has returned with HTTP 200.**

- Quote (`models/openrouter.py:622-623`): "OpenRouter error response with null standard fields"
- Quote (`:629-630`): "OpenRouter body carrying no completion: null `choices` and no error envelope"
- Quote (`:665-666`): "OpenRouter response where the real completion is nested in `provider`"
- Quote (`:1462`): "# OpenRouter intermittently returns responses with null standard fields (https://github.com/pydantic/pydantic-ai/issues/3994)."
- Quote (`:1661-1663`): "# Parity with `OpenRouterModel._validate_completion`: the same no-completion body can" / "# arrive mid-stream."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's validators and the issues behind them (G108, G109). Confidence: high for the first two shapes; the nested shape was reported once and the reporter later posted a different raw body (G108).

**G85. Pydantic AI types `finish_reason` with an added `error` value and treats `native_finish_reason` as optional.**

- Quote (`models/openrouter.py:1617-1624`): `native_finish_reason: str | None = None` and `finish_reason: Literal['stop', 'length', 'tool_calls', 'content_filter', 'error'] | None` with the docstring "Notably, removes 'function_call' and adds 'error' finish reasons."
- Quote (`:1633-1638`): `provider: str | None = None` with "May be absent in early streaming chunks; only the final chunk typically carries" / "the provider name."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's wire types. Confidence: high for `error`. The remark about `provider` disagrees with rig's recorded frames, where every chunk has `provider` (G37, G46); the behavior may differ by date or upstream.

**G86. Pydantic AI reads OpenRouter's extra usage fields.**

- Quote (`models/openrouter.py:554-603`): `class _OpenRouterCostDetails: upstream_inference_cost ... upstream_inference_prompt_cost ... upstream_inference_completions_cost`, `class _OpenRouterServerToolUseDetails: tool_calls_requested ... tool_calls_executed ... web_search_requests`, `class _OpenRouterPromptTokenDetails(...): cache_write_tokens: int | None = None video_tokens: int | None = None`, `class _OpenRouterUsage(...): cost: float | None = None cost_details ... is_byok: bool | None = None server_tool_use_details ...`
- Source: G-s8. Basis: Filled (Pydantic AI). Field names agree with the recorded usage in G44.

**G87. Pydantic AI's cache rules per upstream family.**

- Quote (`providers/openrouter.py:357-358`): `supports_cache_control = provider in ('anthropic', 'google')` / `supports_anthropic_cache = provider == 'anthropic'`
- Quote (`:403-404`): "# On Gemini routes, a breakpoint at the end of the request doubles the billed prompt on OpenRouter's" / "# Google Vertex endpoint, so the message breakpoint goes on an earlier message."
- Quote (`models/openrouter.py:251-255`): "Runs for profiles without `openrouter_supports_trailing_cache_control` (Gemini routes), where OpenRouter uses only the" / "last breakpoint across message content. A breakpoint on the final content part of the request makes OpenRouter's" / "Google Vertex endpoint bill the prompt twice, and one on a tool message isn't used"
- Quote (`providers/openrouter.py:409`, `:400`): `openrouter_max_cache_points=4 if supports_anthropic_cache else None,` and "# Only Anthropic routes take a TTL; Gemini caches at its 5-minute default."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comments. Confidence: medium; "only the last breakpoint" for Gemini is also on OpenRouter's caching page (G-s11), the double billing is from this code only.

**G88. Pydantic AI keeps a per-vendor list of OpenRouter routes that refuse to turn reasoning off.**

- Quote (`providers/openrouter.py:195-199`): "OpenRouter rejects `reasoning={'effort': 'none'}` on these with \"Reasoning is mandatory for this endpoint and cannot" / "be disabled\", including for models that can turn reasoning off on their own API, so their OpenRouter profile marks" / "them always-on and `thinking=False` asks for the lowest effort instead: `minimal`, which every one of them accepted" / "live (2026-11-05), except on the routes in `_LOW_REASONING_FLOOR_MODEL_PREFIXES`. Taken from `reasoning.mandatory` in" / "[OpenRouter's model list](https://openrouter.ai/api/v1/models); `~vendor/…-latest` aliases match without the `~`."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comment with a live probe date. Confidence: high.

**G89. A forced tool choice while thinking has two different outcomes through OpenRouter.**

- Quote (`providers/openrouter.py:410-413`): "# Anthropic errors on a forced `tool_choice` with thinking enabled; OpenRouter instead" / "# drops `reasoning` from the request and returns a response with no reasoning at all."
- Quote (`:219-225`): "Live probes (2026-10-31, 2026-11-17) got a 400 for a forced `tool_choice` while Qwen3.8 Flash reasons: \"The" / "tool_choice parameter does not support being set to required or object in thinking mode\". Alibaba, its only host on" / "OpenRouter, returns it. OpenRouter's model list advertises `tool_choice` for the route, so it can't be used to find" / "these routes." ... "Qwen3.8 2.4T is rejected on Alibaba but accepted on Venice, and Muse Spark 1.3 is rejected" / "on Meta but accepted on Google Vertex."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comments with live probe dates. Confidence: medium to high. The last sentence is direct evidence of behavior that differs by upstream for one model id.

**G90. OpenRouter has its own chat extension for changing effort in mid-conversation, accepted only on some routes.**

- Quote (`providers/openrouter.py:234-235`): "OpenRouter routes a request carrying a `configuration_update` only to endpoints that accept it, and rejects it with a" / "400 (\"Mid-conversation reasoning effort (configuration_update) is not supported\") on every other model"
- Quote (`:390-391`): "# OpenRouter's own Chat Completions extension: a content-less `system` message carrying a" / "# `configuration_update`."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comment; it cites OpenRouter's page `reasoning-tokens#changing-effort-mid-conversation`, which another subagent covers. Confidence: medium.

**G91. Where a mid-conversation `system` message ends up depends on the route.**

- Quote (`providers/openrouter.py:253-261`): "- OpenAI routes keep it where it was sent" ... "- Anthropic routes keep it as a native entry only where Anthropic's own API accepts one" ... "On Claude Sonnet 4.6, Opus 4.7 and Haiku 4.5," / "OpenRouter moves it into the top-level system prompt, which busts the cached prefix after it." ... "- Gemini routes move it into the system instruction, which busts the cached prefix after it."
- Quote (`:72-73`, profile field doc): "OpenRouter turns one anywhere else into user text on Anthropic routes, so `OpenRouterModel` moves each one past" / "the user and tool messages that follow it, as `AnthropicModel` does."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comments ("Measured live on Chat Completions (2026-10-30)"). Confidence: medium.

**G92. Two smaller route facts from Pydantic AI's profile.**

- Quote (`providers/openrouter.py:368-369`): "OpenRouter only" / "#    accepts the older `max_tokens` field, so `openai_chat_supports_max_completion_tokens=False`."
- Quote (`:375-377`): "# OpenRouter answers 404 \"No endpoints found that support input video\" for a model without video" / "# input (checked live with Claude, GPT and DeepSeek routes, 2026-11-12)"
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code comments. Confidence: medium. Note langchain-openrouter does send `max_completion_tokens` when set (G101), so "only accepts" may be too strong.

**G93. Pydantic AI treats web search and the advisor as gateway tools that work with any model.**

- Quote (`providers/openrouter.py:393-395`): "# OpenRouter's native tools (web search plugin, advisor) are gateway features that" / "# work with any underlying model, so the upstream profile's vendor-specific tool" / "# gating (e.g. Anthropic's valid-executor list) doesn't apply."
- Source: G-s8. Basis: Filled (Pydantic AI). Agrees with LC9 on the existence of server tools.

**G94. For `google/` models Pydantic AI inlines schema definitions because OpenRouter's translation does not handle `$defs` and `$ref` well.**

- Quote (`providers/openrouter.py:86-94`): "Legacy Google JSON schema transformer for OpenRouter compatibility." / "OpenRouter's compatibility layer doesn't fully support modern JSON Schema features" / "like $defs/$ref and anyOf for nullable types. This transformer restores v1.19.0" / "behavior by inlining definitions and simplifying nullable unions."
- Source: G-s8. Basis: OR-inferred, from Pydantic AI's code and issue 3617 (G126). Confidence: medium (the issue is from December 2025).

**G95. Pydantic AI types the `provider` object and other OpenRouter request fields under an `openrouter_` prefix.**

- Quote (`models/openrouter.py:306-339`, `:369-400`): `class OpenRouterProviderConfig(TypedDict, total=False): order ... allow_fallbacks ... require_parameters ... data_collection: Literal['allow', 'deny'] ... zdr ... only ... ignore ... quantizations ... sort: Literal['price', 'throughput', 'latency', 'exacto'] ... max_price` and settings `openrouter_models`, `openrouter_provider`, `openrouter_preset`, `openrouter_transforms`, `openrouter_reasoning`, `openrouter_usage`.
- Source: G-s8. Basis: Filled (Pydantic AI).

**G96. Pydantic AI takes `app_url` and `app_title` for the attribution headers.**

- Quote (`providers/openrouter.py:456-459`, constructor doc): "app_url: Optional url for app attribution. Falls back to" / "`OPENROUTER_APP_URL` when omitted." and "app_title: Optional title for app attribution."
- Source: G-s8. Basis: Filled (Pydantic AI).

### LangChain (Python) - `langchain-ai/langchain` at `8ff3cd3`, package `libs/partners/openrouter` (`langchain-openrouter` 0.3.0)

**G97. `langchain-openrouter` is a separate partner package built on OpenRouter's official Python SDK.**

- Quote (`libs/partners/openrouter/langchain_openrouter/chat_models.py:408-414`, `pyproject.toml:26`): "Build and return an `openrouter.OpenRouter` SDK client." / `import openrouter` and `"openrouter>=0.9.2",`
- Quote (`:607`): `for chunk in self.client.chat.send(messages=message_dicts, **params):`
- Source: G-s9. Basis: Filled (LangChain).

**G98. LangChain keeps `reasoning_details` in `additional_kwargs`, merges stream fragments before replay, and removes OpenAI reasoning item ids.**

- Quote (`chat_models.py:1321-1330`): "# Preserve reasoning content for multi-turn conversations (e.g." / "# tool-calling loops). OpenRouter stores reasoning text in `reasoning`" / "# and structured fragment details in `reasoning_details`; the latter" / "# is merged before serialization to undo streaming fragmentation." and `message_dict["reasoning_details"] = _strip_ephemeral_reasoning_ids(_merge_reasoning_details(message.additional_kwargs["reasoning_details"]))`
- Quote (`:1214-1219`): "During streaming, `AIMessageChunk.__add__` list-concatenates" / "`reasoning_details` in `additional_kwargs`, fragmenting a single entry" / "into many. When serialized back to the API via" / "`_convert_message_to_dict`, these fragments cause" / "`BadRequestResponseError` on multi-turn conversations (the provider" / "rejects the malformed thinking block with `Invalid signature`)."
- Quote (`:1196-1205`): "Remove OpenAI Responses reasoning item IDs from outbound payloads." and `if not (key == "id" and isinstance(item, str) and item.startswith("rs_"))`
- Source: G-s9. Basis: Filled (LangChain). Storage shape: `AIMessage.additional_kwargs["reasoning_details"]` (list of raw dicts) and `additional_kwargs["reasoning_content"]` (string). Implication (OR-inferred, medium): fragments sent back unmerged are rejected upstream with "Invalid signature", and `rs_` ids are not needed (or not wanted) on replay.

**G99. LangChain raises on an `error` object in a reply or in a stream chunk without `choices`.**

- Quote (`chat_models.py:609-616`): `if not chunk_dict.get("choices"): if error := chunk_dict.get("error"): msg = (f"OpenRouter API returned an error during streaming: " f"{error.get('message', str(error))} " f"(code: {error.get('code', 'unknown')})") raise ValueError(msg)`
- Source: G-s9. Basis: Filled (LangChain).

**G100. LangChain surfaces `cost` and `cost_details`, and casts token counts to int.**

- Quote (`chat_models.py:1528-1529`): "OpenRouter may return token counts as floats rather than ints, so all" / "values are explicitly cast to int."
- Quote (`:1559-1560`): `cache_read = input_details_dict.get("cached_tokens")` / `cache_creation = input_details_dict.get("cache_write_tokens")`
- Quote (`:860-865`): "# Surface OpenRouter cost data in response_metadata" with `message.response_metadata["cost"] = token_usage["cost"]`
- Source: G-s9. Basis: Filled (LangChain); the float remark is OR-inferred from the comment, low confidence.

**G101. LangChain's OpenRouter-only request fields.**

- Quote (`chat_models.py:803-816`): "# OpenRouter-specific params" with `params["reasoning"]`, `params["provider"] = self.openrouter_provider`, `params["route"]`, `params["plugins"]`, `params["session_id"]`, `params["trace"]`; and `:789-790` `params["max_completion_tokens"] = self.max_completion_tokens`.
- Source: G-s9. Basis: Filled (LangChain).

**G102. LangChain sends the three attribution headers when configured.**

- Quote (`chat_models.py:425-431`): `extra_headers["HTTP-Referer"] = self.app_url` / `extra_headers["X-Title"] = self.app_title` / `extra_headers["X-OpenRouter-Categories"] = ",".join(self.app_categories)`
- Source: G-s9. Basis: Filled (LangChain).

### Codex (Rust) - `openai/codex` at `227fa4e`, and the Responses route

**G103. Codex speaks only the Responses API; the chat wire was removed.**

- Quote (`codex-rs/model-provider-info/src/lib.rs:112`): `const CHAT_WIRE_API_REMOVED_ERROR: &str = "`wire_api = \"chat\"` is no longer supported.\nHow to fix: set `wire_api = \"responses\"` in your provider config.\nMore info: https://github.com/openai/codex/discussions/7782";`
- Quote (`:119-122`): `pub enum WireApi { /// The Responses API exposed by OpenAI at `/v1/responses`. #[default] Responses, }`
- Source: G-s10. Basis: Filled (Codex). Also in workspace: the 2026-11-17 survey (Codex `c08843a`, "Responses API only").

**G104. What Codex sends, and so what it needs from OpenRouter's `/responses` route.**

- Quote (`codex-rs/core/src/client.rs:1326`, `:1367-1381`): `let mut include = vec!["reasoning.encrypted_content".to_string()];` and `tool_choice: "auto".to_string(), ... reasoning: Some(reasoning), store: false, stream: true, stream_options, include, service_tier, prompt_cache_key: self.prompt_cache_key(responses_metadata),`
- Quote (`codex-rs/codex-api/src/common.rs:408-436`): `pub struct ResponsesApiRequest { ... pub model: String, pub stream: bool, ... pub input: Vec<ResponseItem>, ... pub tools ..., pub tool_choice: String, pub parallel_tool_calls: bool, pub reasoning: Option<Reasoning>, pub store: bool, ... pub include: Vec<String>, ... pub prompt_cache_key: Option<String>, ... pub prompt_cache_options ..., pub text ..., pub client_metadata ..., }`
- Quote (`lib.rs:73-75`): `const DEFAULT_STREAM_IDLE_TIMEOUT_MS: u64 = 300_000;` / `const DEFAULT_STREAM_MAX_RETRIES: u64 = 5;` / `const DEFAULT_REQUEST_MAX_RETRIES: u64 = 4;`
- Source: G-s10. Basis: Filled (Codex). Codex over HTTP is stateless: `store: false`, the whole `input` list each turn, and encrypted reasoning items echoed back. `previous_response_id` appears only on its WebSocket transport (`common.rs:475`), which a custom provider gets only with `supports_websockets = true`. This fits LC4 (OpenRouter rejects a non-null `previous_response_id` and wants `store` false or absent).

**G105. Codex treats OpenRouter as one more user-configured Responses provider.**

- Searched `codex-rs/model-provider-info`, `codex-rs/core/src`, `codex-rs/codex-api/src` and `docs` at this commit for "openrouter" (any case): zero matches, and zero file names in the whole tree hold the word.
- Quote (`codex-rs/model-provider-info/src/capabilities.rs:13-22`): `pub struct ModelProviderCapabilities { ... pub cyber_access_programs: Option<bool>, ... pub external_web_access: Option<bool>, /// Remote context-compaction protocol; omission preserves provider defaults. pub remote_compaction: Option<RemoteCompactionSupport>, }`
- Source: G-s10. Basis: Filled (Codex). Confidence: medium, because the sparse checkout covered four folders, not the whole repository.

**G106. OpenRouter's own Codex page configures a custom provider with command-based auth.**

- Quote (page "Codex CLI"): `model_provider = "openrouter"` / `model_reasoning_effort = "high"` / `model="~openai/gpt-sol-latest"` / `[model_providers.openrouter]` / `name = "openrouter"` / `base_url="https://openrouter.ai/api/v1"` / `[model_providers.openrouter.auth]` / `command = "sh"` / `args = ["-c", "echo $OPENROUTER_API_KEY"]`
- Quote: "A plain `env_key = \"OPENROUTER_API_KEY\"` also works for authentication, but Codex won't fetch the OpenRouter model catalog in that mode. Non-OpenAI models will show the \"Unknown model\" fallback-metadata warning."
- Quote: "Point `model` at any OpenRouter slug (e.g. `~openai/gpt-sol-latest`, `~anthropic/claude-sonnet-latest`)".
- Source: G-s12. Basis: OR-documented. The page sets no `wire_api` (Responses is Codex's default) and says nothing about `store`, `include`, or reasoning replay.

**G107. Recorded Responses-route traffic through OpenRouter (from rig's fixtures) shows the stateless shape Codex needs, for Claude and Gemini too.**

- Fixture `upstream_switch_matrix/responses_switch_unary.yaml` (recorded 2026-10-02 20:14 UTC). Request keys: `include`, `input`, `max_output_tokens`, `model`, `reasoning`, `tools`.
- Reply for `anthropic/claude-haiku-4.5`: `"store": false`, `"previous_response_id": null`, `"status": "completed"`, and output items `{"type":"reasoning","id":"rs_tmp_tlkmunm8kfr","format":"anthropic-claude-v1","signature":"EuQF...","content":[{"type":"reasoning_text","text":"..."}],"summary":[],"status":"completed"}`, `{"type":"message","id":"msg_tmp_5aggo72z53h","role":"assistant",...}`, `{"type":"function_call","id":"fc_tmp_28uhmip58ry","call_id":"toolu_bdrk_01BxfvW8szzgbEbBUht6oynv","name":"lookup_code","arguments":"{\"record\": \"alpha\"}","status":"completed"}`.
- Reply for `google/gemini-3-flash-preview`: a `reasoning` item with `"format":"google-gemini-v1"` and `content` text, with no `signature` key in this unary cell.
- Usage: `{"cost":0.001608,"cost_details":{"upstream_inference_cost":0.001608,"upstream_inference_input_cost":0.000608,"upstream_inference_output_cost":0.001},"input_tokens":608,"input_tokens_details":{"cache_write_tokens":0,"cached_tokens":0},"is_byok":false,"output_tokens":200,"output_tokens_details":{"reasoning_tokens":122},"total_tokens":808}`
- The reply's top-level keys do not include `provider` in this fixture.
- Quote (`cassette/upstream_switch_matrix.rs:9-11`): "`responses_` cells repeat them on OpenRouter's Responses route, where" / "//! Claude's signature rides beside the reasoning item."
- Source: G-s4 (my `responses_shape.py` output). Basis: OR-inferred, from recorded traffic. Confidence: high for the shapes on that date.

### Issue reports (third-party accounts)

Every item here is an issue report or pull request discussion, so it is a third-party account, not OpenRouter's statement, except where a comment is signed by OpenRouter staff. "Fetched" means I opened the page itself today; "search text" means I read the page text that the search tool returned today and did not open the page separately.

Source keys for this section: G108 is G-s17, G109 is G-s18, G110 is G-s19, G111 is G-s20, G112 is G-s21, G113 is G-s22, G114 is G-s23 and G-s24, G115 is G-s25, G116 is G-s26, G117 is G-s27, G118 is G-s28, G119 is G-s29 and G-s30, G120 is G-s31, G121 is G-s32 and G-s34, G122 is G-s33, G123 is G-s35, G124 is G-s36, G125 is G-s37, G126 is G-s38, G127 is G-s39 and G-s40, G128 is G-s41, G129 is G-s42.

**G108. Pydantic AI issue 3994: OpenRouter replies with HTTP 200 where every standard field is null.**

- URL: https://github.com/pydantic/pydantic-ai/issues/3994 - created 2026-01-13, closed. Fetched.
- Quote (reporter's raw body, 2026-01-14 and 2026-01-31): `{'id': None, 'choices': None, 'created': 1768361801, 'model': None, 'object': None, 'service_tier': None, 'system_fingerprint': None, 'usage': None, 'error': {'message': 'Provider returned error', 'code': 524, 'metadata': {'raw': 'error code: 524', 'provider_name': 'Google'}}, 'user_id': '...'}`
- Quote (error): "pydantic_ai.exceptions.UnexpectedModelBehavior: Invalid response from openrouter chat completions endpoint: 5 validation errors for _OpenRouterChatCompletion"
- The first post also describes a completion nested under `provider`; the reporter's later comment says "Update, the responses are invalid" and shows only the error shape above. "Affected models: `google/gemini-3-flash-preview`, `anthropic/claude-sonnet-4.5`".
- Basis: OR-inferred, from an issue report (third-party). Confidence: high for the error-in-200 shape, low for the nested shape.

**G109. Pydantic AI issue 6900: null `choices` with no `error` object.**

- URL: https://github.com/pydantic/pydantic-ai/issues/6900 - created 2026-07-30, closed. Fetched.
- Quote: "OpenRouter intermittently returns a response body with **null `choices` and no error envelope at all**" and "We hit this in production regularly enough that it's the last surviving guard of what used to be a ~574-line local OpenRouter patch module".
- A bot comment on the same page shows a credit error body: `{'code': 402, 'message': "This request requires more credits, or fewer max_tokens. You requested up to 65536 tokens, but can only afford 38537. To increase, visit https://openrouter.ai/workspaces/.../keys/... and adjust the key's weekly limit"}` (workspace and key ids elided by me).
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium.

**G110. Hermes issue 42991: a reasoning disable sent through OpenRouter becomes Anthropic's `thinking: {type: "disabled"}` and fails on always-thinking Claude models.**

- URL: https://github.com/NousResearch/hermes-agent/issues/42991 - created 2026-06-09, closed (merged). Fetched.
- Quote: "Over OpenRouter that path forwards `reasoning: {enabled: false}` when reasoning is off, which OpenRouter translates into Anthropic's `thinking: {type: \"disabled\"}`" and the upstream error `"thinking.type.disabled" is not supported for this model`.
- Quote (validation table): "reasoning disabled (`{enabled: false}`) | HTTP 400 `thinking.type.disabled` | HTTP 200 (adaptive)".
- Basis: OR-inferred, from an issue report with live calls (third-party). Confidence: high.

**G111. Hermes issue 43432: `reasoning.effort` does nothing for always-thinking Claude through OpenRouter; `verbosity` does.**

- URL: https://github.com/NousResearch/hermes-agent/issues/43432 - created 2026-06-10, closed. Fetched.
- Quote: "`verbosity` produces a clear, monotonic, repeatable change in both completion and reasoning token spend. `reasoning.effort` does not move the needle (matches OpenRouter's \"ignored\" doc)."
- Quote (the reporter citing OpenRouter's Claude 4.7 migration guide): "To influence overall response effort, use `verbosity`. It maps to Anthropic's `output_config.effort` ... `reasoning.effort` is accepted but ignored."
- Quote (measured): `verbosity:low` gave reasoning tokens "38 / 31", `verbosity:high` "110 / 113", and `{"reasoning":{"enabled":true,"effort":"low"}}` gave 73 against a baseline of 87.
- Basis: OR-inferred, from an issue report with measurements (third-party). Confidence: medium to high. A neutral effort setting maps to a different top-level field for one vendor family.

**G112. Hermes issue 25970: Kimi K2.6 through OpenRouter got about 1 percent cache hits until `cache_control` markers were sent.**

- URL: https://github.com/NousResearch/hermes-agent/issues/25970 - created 2026-05-14, closed. Fetched.
- Quote: "Typical chat turn: ` cache=368/64403 (1%)`" and "within-turn tool-heavy turns now show cache buildup 1% → 67% → 84% → 97%."
- Quote (maintainer): "applies to every provider that respects `cache_control` (including Kimi/Moonshot via OpenRouter, which uses the same wire format as Claude via OpenRouter)."
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium.

**G113. OpenRouter AI SDK provider issue 491: a Gemini signature replayed after the upstream switched between AI Studio and Vertex returns 400; OpenRouter's SDK team confirmed the cause is in routing.**

- URL: https://github.com/OpenRouterTeam/ai-sdk-provider/issues/491 - created 2026-04-30, closed 2026-06-17. Fetched.
- Quote (reporter): "google-vertex -> google-vertex: success / google-ai-studio -> google-ai-studio: success / google-vertex -> google-ai-studio: fails / google-ai-studio -> google-vertex: fails"
- Quote (closing comment signed "the OpenRouter SDK team"): "The failure is gated entirely by **upstream provider switching**: continuing a tool call on the *same* Google endpoint succeeds (200), while switching between `Google AI Studio` and `Google` (Vertex) mid-tool-call returns 400 (`Unable to submit request because Thought signature is not valid.` / `Corrupted thought signature.`). Gemini thought signatures are provider-scoped, so they can't be validated by a different upstream provider."
- Quote (same comment): "A practical workaround in the meantime: pin the provider for a multi-turn tool-call conversation (e.g. `provider: { order: [...], allow_fallbacks: false }`) so all turns route to the same Google endpoint."
- Quote (entry shape in the report): `{"type": "reasoning.encrypted", "format": "google-gemini-v1", "data": "<large opaque blob>", "id": "tool_get-available-images_123", "index": 0}`
- Basis: OR-inferred, from an issue report; the cause is stated by OpenRouter staff in a public comment, which is close to OR-documented. Confidence: high as of June 2026. Whether routing was fixed since is unknown. Compare G37: for Claude, OpenRouter now tags the signature with the endpoint.

**G114. Two reports of Gemini 3 tool calls failing through OpenRouter when `reasoning_details` were not kept whole.**

- URL: https://github.com/OpenRouterTeam/ai-sdk-provider/issues/239 - created 2025-11-19, closed 2026-04-08. Search text.
- URL: https://github.com/RooCodeInc/Roo-Code/issues/10307 - created 2025-12-23, closed. Search text.
- Quote (both reports show this text inside `error.metadata.raw`): "Gemini models require OpenRouter reasoning details to be preserved in each request. Please refer to our docs: https://openrouter.ai/docs/guides/best-practices/reasoning-tokens#preserving-reasoning-blocks. Upstream error:" followed by Google's "Function call is missing a thought_signature in functionCall parts." The first report's body also has `"status": "INVALID_ARGUMENT"` and `"provider_name": "Google AI Studio"`; a later comment there shows the Vertex wording "Unable to submit request because function call `default_api:readFile` in the 2. content block is missing a `thought_signature`." with `"provider_name": "Google"`.
- Quote (Roo Code analysis): "For `reasoning.encrypted` types, the `id` is the **tool call ID** that links the thought signature to the function call and **must be preserved**".
- Basis: OR-inferred, from issue reports (third-party). Confidence: high. OpenRouter prepends its own sentence to the upstream error, inside `metadata.raw`.

**G115. OpenRouter examples issue 67: a claim that OpenRouter strips Gemini's signature.**

- URL: https://github.com/OpenRouterTeam/openrouter-examples/issues/67 - created 2026-03-10, open. Search text.
- Quote: "OpenRouter's OpenAI-compatible format translation **strips `thought_signature` and `reasoning_details`** from Gemini responses." and the failing reply `{"finish_reason": "error", "native_finish_reason": "MALFORMED_FUNCTION_CALL", "message": {"role": "assistant", "content": null}}`
- Basis: OR-inferred, from an issue report (third-party). Confidence: low for the "strips" claim: G46 and G113 show the signature delivered in `reasoning_details`, so the loss is more likely in clients that do not read that field. High for the reply shape with `finish_reason: "error"`.

**G116. Pydantic AI issue 4370: `native_finish_reason` went missing from stream chunks for a day, and encrypted entries arrived with only two keys.**

- URL: https://github.com/pydantic/pydantic-ai/issues/4370 - created 2026-02-19, closed. Fetched.
- Quote: "OpenRouter seems to have stopped including `native_finish_reason` in every streaming chunk just recently" and a commenter: "looks like there was an issue with OpenRouter earlier today which has been resolved".
- Quote: "OpenRouter sent reasoning.encrypted chunks with only type and data: {\"type\": \"reasoning.encrypted\", \"data\": \"EjQKMgG+Pvb7kpZI...\"}"
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium. Fields beyond `type` are not guaranteed on every entry.

**G117. opencode issue 41469: empty replies and upstream errors that vanish.**

- URL: https://github.com/anomalyco/opencode/issues/41469 - created 2026-08-10, closed. Search text.
- Quote (comment, 2026-08-23, direct call to OpenRouter): `{"error": {"message": "Provider returned error", "code": 429, "metadata": {"raw": "stealth/ox-alpha is temporarily rate-limited upstream. Please retry shortly.", "provider_name": "Stealth", "limit_source": "upstream_provider_shared_pool"}}}`
- Quote (issue body): "the session loop exits silently" and "no error is surfaced to the user, no retry happens".
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium. Adds a second `limit_source` value beside the one in LC13.

**G118. LibreChat agents issue 662: usage counted twice because OpenRouter puts usage on a chunk that still has a choice.**

- URL: https://github.com/LibreChat-AI/agents/issues/662 - date and state not captured. Search text.
- Quote: "OpenRouter sends `usage` on the chunk that carries `finish_reason`, and that chunk still has a choice. OpenAI sends usage on a separate chunk with `choices: []`."
- Quote: "The top-level `provider` field OpenRouter sends (the provider that served the call) is also dropped."
- Basis: OR-inferred, from an issue report (third-party). Confidence: high, it matches the recorded frames in G44. Supports LC12.

**G119. Two reports of cache-write tokens dropped by clients that only read OpenAI's standard usage fields.**

- URL: https://github.com/anomalyco/opencode/issues/18440 - created 2026-03-20, state not captured. Search text. Quote: "`cacheWriteInputTokens` falls back to 0 for OpenRouter because the code only looks for cache write tokens in `anthropic` and `bedrock` metadata."
- URL: https://github.com/pydantic/genai-prices/issues/345 - created 2026-04-27, closed. Search text. Quote: "OpenRouter activity log: `cost_total=$0.199188`, `cost_cache=$0.039462`" against a client-computed "$0.159726", "priced as if there were zero cache-write tokens."
- Basis: OR-inferred, from issue reports (third-party). Confidence: high that `prompt_tokens_details.cache_write_tokens` is the field and that clients miss it.

**G120. MeshWeaver write-up: a lone `error` chunk on an open 200 stream, and `finish_reason: "error"` breaking a strict SDK.**

- URL: https://www.meshweaver.cloud/AI/ProviderStreamFaults - undated page, incident dated 2026-09-30 in the text. Search text. Kind: blog (third-party).
- Quote: "When OpenRouter has ACCEPTED a request (HTTP 200, the SSE stream open) but cannot serve it, it streams a lone `{\"error\":{\"code\":…,\"message\":\"…\"}}` chunk with no `choices`, no `model` and no `finish_reason`."
- Quote: "That value is not in the OpenAI specification, so `OpenAI.Chat.ChatFinishReasonExtensions.ToChatFinishReason` throws `ArgumentOutOfRangeException: Unknown ChatFinishReason value. … Actual value was error.`"
- Basis: OR-inferred, from a blog (third-party). Confidence: medium. Supports LC11.

**G121. Zed issue 52576: no prompt caching for Claude through OpenRouter until the client added `cache_control`.**

- URL: https://github.com/zed-industries/zed/issues/52576 - created 2026-03-27, closed. Fetched.
- Quote: "**native_tokens_cached is always 0**" and a commenter: "In a session with just 4 prompts, the agent performed 130 tool calls. Because prompt caching wasn't working (same result of 0 tokens cached according to openrouter activity log), each request resent the full context."
- Quote (later comment, 2026-09-07): "Is this fixed? As caching still doesn't work."
- The fix (pull request 57498, search text of a mirror of commit `f844c93`): "The implementation intentionally avoids OpenRouter's top-level automatic `cache_control` field so routing remains available across Anthropic-compatible providers including Anthropic, Bedrock, and Vertex AI."
- Basis: OR-inferred, from an issue report (third-party). Confidence: high: OpenRouter does not cache Claude prompts unless the caller asks.

**G122. Zed commit 7fab4c0 (pull request 66857): echoing `reasoning_details` for `~anthropic/` aliases was fixed as a bug, untested against the API.**

- URL: https://github.com/zed-industries/zed/commit/7fab4c0527e10a466439b819d4e9e7fe34cd989c - dated 2026-11-12. Search text.
- Quote: "`into_open_router` only checked for the `anthropic/` prefix, so aliases were not treated as Anthropic models: no `cache_control` was sent (no prompt caching), and `reasoning_details` was echoed back in requests." and "I did not test this manually against the OpenRouter API."
- Basis: Filled (Zed). It shows Zed's rule in G1 is still current ten days before today, and that it was not re-tested.

**G123. Goose issue 11260: a `$ref` key in tool output breaks Gemini sessions through OpenRouter.**

- URL: https://github.com/aaif-goose/goose/issues/11260 (redirected from `block/goose`) - created 2026-08-14, closed. Fetched.
- Quote (Google's error through OpenRouter): "400 INVALID_ARGUMENT: The referenced name `#/components/schemas/Example` in function_response.response does not match to a display_name in the function_response.parts."
- Quote: "I also reproduced the failure by posting the exact Goose-generated Chat Completions body directly to OpenRouter" and "Key rewritten to `dollar_ref`, original `#/components/schemas/Example` value preserved verbatim: HTTP success with a normal completion."
- Basis: OR-inferred, from an issue report with A/B controls (third-party). Confidence: high as of August 2026.

**G124. OpenClaw issue 24900: Anthropic tool-call ids replayed to an OpenAI model through OpenRouter fail forever.**

- URL: https://github.com/openclaw/openclaw/issues/24900 - created 2026-02-24, closed as duplicate. Search text.
- Quote: "`400 Provider returned error: No tool call found for function call output with call_id toolu_...`"
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium. Note the wording "function call output with call_id", which is the Responses API's vocabulary, so the OpenAI model was being served from Responses upstream (fits G41). rig's fixture shows the reverse direction (a `call_` id sent to Claude) accepted (G40).

**G125. gptel issue 747: Claude through OpenRouter returns `toolu_` ids on an OpenAI-shaped API.**

- URL: https://github.com/karthink/gptel/issues/747 - created 2025-03-26, closed. Search text.
- Quote: "when Claude 3.7 Sonnet through OpenRouter makes a tool call, it uses the `toolu_...` format instead of the `call_...` format." and the client error "`Unexpected tool_call_id format: ...`".
- Basis: OR-inferred, from an issue report (third-party). Confidence: high, matches G40. Old (March 2025) but the behavior is the same in October 2026 recordings.

**G126. Pydantic AI issue 3617: Gemini through OpenRouter gave degraded tool arguments for schemas with `$defs`, `$ref` and nullable `anyOf`.**

- URL: https://github.com/pydantic/pydantic-ai/issues/3617 - created 2025-12-02, closed. Fetched.
- Quote: "The new format with `$defs`, `$ref`, and `anyOf` causes Gemini via OpenRouter to misunderstand the schema structure, even though it works when using `GoogleModel` directly."
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium; it is a year old and OpenRouter's translation may have changed.

**G127. OpenRouter AI SDK provider issue 423 and pull request 442: a signature lost in the client causes Anthropic's "Invalid signature" through OpenRouter, and so does plain `reasoning` text sent without its details.**

- URL: https://github.com/OpenRouterTeam/ai-sdk-provider/issues/423 - created 2026-02-19, closed. Search text.
- Quote: "invalid_request_error: messages.1.content.0: Invalid `signature` in `thinking` block" and "A final delta with only the signature: `{ type: \"reasoning.text\", signature: \"eyJ...\", format: \"anthropic-claude-v1\" }`"
- URL: https://github.com/OpenRouterTeam/ai-sdk-provider/pull/442 - merged 2026-03-13. Search text. Quote (written by an OpenRouter SDK maintainer): "the `reasoning` text field was still accumulated and sent to the API, but without corresponding `reasoning_details` (which carry the signature). The API would construct a thinking block from the orphaned `reasoning` text, and Anthropic would reject it due to the missing/invalid signature."
- Basis: OR-inferred, from an issue report and a pull request in OpenRouter's own SDK repository. Confidence: medium to high. Two deciding facts: a signature-only stream delta exists (matches G37, G49), and OpenRouter builds an Anthropic thinking block from a replayed plain `reasoning` string. The second conflicts with rig's recorded request in G38, where Claude's thinking text went to Gemini as `content`, and with G128.

**G128. genai pull request 301: dropping `reasoning_details` through OpenRouter fails silently.**

- URL: https://github.com/jeremychone/rust-genai/issues/301 - created 2026-09-07, merged. Fetched.
- Quote: "through OpenRouter a signed or encrypted block was dropped on decode and the next request went out without it. The gateway accepts that request and the model continues without its prior reasoning, so nothing errors."
- Quote: "**Verified live** against `anthropic/claude-sonnet-4-6` and `moonshotai/kimi-k3` through OpenRouter on 2026-09-05: sending both fields together on a replayed assistant turn is accepted, and with `reasoning_details` present the signature is carried; with plaintext only it is not."
- Basis: OR-inferred, from a pull request with a live check (third-party). Confidence: medium to high. A second live confirmation, besides rig's, that Claude accepts replayed `reasoning_details` through OpenRouter.

**G129. Hermes issue 90216: the serving upstream changes per request and is named only in chunk bodies.**

- URL: https://github.com/NousResearch/hermes-agent/issues/90216 - created 2026-08-19, closed. Fetched.
- Quote: "three consecutive requests for the same model were served by `\"provider\":\"Alibaba\"`, `\"Novita\"`, and `\"StreamLake\"` respectively" and "Mid-stream `RemoteProtocolError` drops (\"peer closed connection without sending complete message body\") cluster on specific downstream providers".
- Basis: OR-inferred, from an issue report (third-party). Confidence: medium (observed through Nous's relay to OpenRouter).

### Counts and storage shapes

**Special cases per project and per axis.** A special case here is one evidence item in a project's source section above (G1 to G36 and G47 to G105, without the two summary items G59 and G79 and without rig's recorded-traffic items). Each item is counted once under its main axis. "Reasoning controls" and "Other" are two axes I added to the task's list: the first for request-side thinking settings, the second for route, auth, and message-placement items.

| Project | Reasoning replay | Reasoning controls | Caching | Usage and cost | Tools | Structured output | Errors and stream failures | Routing and provider options | Headers | Model metadata | Other | Total |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Zed | 3 | 1 | 1 | 1 | 1 | 0 | 2 | 2 | 1 | 1 | 0 | 13 |
| Goose | 2 | 1 | 1 | 1 | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 11 |
| rig | 3 | 1 | 1 | 2 | 0 | 0 | 1 | 2 | 0 | 1 | 1 | 12 |
| genai | 4 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 | 6 |
| Aider | 0 | 2 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 1 | 1 | 6 |
| Hermes Agent | 3 | 2 | 4 | 2 | 0 | 0 | 4 | 3 | 1 | 0 | 0 | 19 |
| Pydantic AI | 3 | 2 | 1 | 1 | 2 | 1 | 3 | 1 | 1 | 0 | 2 | 17 |
| langchain-openrouter | 1 | 0 | 0 | 1 | 0 | 0 | 1 | 1 | 1 | 0 | 1 | 6 |
| Codex | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Total | 19 | 9 | 10 | 8 | 4 | 1 | 13 | 10 | 5 | 4 | 7 | 90 |

These are counts of what I recorded, not a census of every branch. Hermes in particular has far more OpenRouter branches than the 19 items here (G79). Codex's three items (G103 to G105) describe a generic Responses client, so its count of OpenRouter-specific branches is zero.

**How each client stores the vendor's opaque reasoning state from OpenRouter.**

| Project | Where it is stored | Exact shape | Replay rule | Evidence |
|---|---|---|---|---|
| Zed | On each request message | `reasoning_details: Option<Arc<serde_json::Value>>` (the merged raw array); on each tool call `thought_signature: Option<String>` | Sent back for all models except ids starting `anthropic/` or `~anthropic/` | G1, G2, G3 |
| Goose | On each tool request's metadata map | key `reasoning_details` with the raw array (non-streaming code only); key for the Gemini signature plus the tool call's extra fields such as `extra_content` | `reasoning_details` added to the request only for ids starting `google/gemini` | G14, G15 |
| rig | On the reasoning block of the turn | `Native { item: {"reasoning": text, "reasoning_details": [...]}, fingerprint, baseline, edited, blob_slot }` and per turn `Origin { api, provider, model, response_model, response_id, context }` | Only when wire, provider and model all match; an edited block loses its signed data | G26, G27 |
| genai | One content part per entry | `ContentPart::Custom(CustomPart { model_iden, data })` with `data` the raw entry | Only to the same adapter kind (OpenRouter); dropped by the native Anthropic adapter | G47, G48, G50 |
| Hermes Agent | On the stored assistant message dict | `reasoning_details`: list of raw dicts, beside `reasoning_content` | To OpenRouter-format gateways only; rejected fields are learned per provider, host and model | G60, G61 |
| Pydantic AI | One thinking part per entry | `ThinkingPart { id, content, signature, provider_name: "openrouter", provider_details: {format, index, type} }`; encrypted `data` goes in `signature` | Only to a model whose system is `openrouter` | G80, G81 |
| langchain-openrouter | On the message | `additional_kwargs["reasoning_details"]`: list of raw dicts; `additional_kwargs["reasoning_content"]`: string | Always sent back, after merging fragments and removing ids that start `rs_` | G98 |
| Aider | Not stored | Reasoning is shown and discarded | n/a | G59 |
| Codex | In its `input` item list (Responses) | Whole `reasoning` items with `encrypted_content`, requested with `include: ["reasoning.encrypted_content"]` | Every turn, `store: false` | G104 |

Of the seven chat-route clients that store the entries, six keep them as raw JSON (Zed, Goose, rig, genai, Hermes, LangChain) and one (Pydantic AI) re-types them into its own fields. Only rig ties replay to the exact model; Zed and Goose tie it to a vendor prefix of the model id; genai and Pydantic AI tie it to "came from OpenRouter"; LangChain and Hermes send back whatever is stored.

## Conflicts and disagreements

1. **Does Claude through OpenRouter accept replayed `reasoning_details`?** Zed says no (G1: "it's an output-only field for them") and re-affirmed the rule in a commit dated 2026-11-12 that was not tested against the API (G122). rig's recorded traffic says yes on three routes (Anthropic, Bedrock, Vertex) on 2026-10-02 and 2026-10-03 (G37, G39), genai's pull request reports a live check on 2026-09-05 (G128), and OpenRouter's reasoning page has a Claude example with the line "reasoning blocks preserved unmodified, so Claude continues reasoning from" (G-s13). The recorded traffic and the documentation agree, so Zed's comment is the outlier. I could not find when or whether it was ever true. The practical effect is that Zed plus OpenRouter plus Claude runs tool loops without earlier thinking.
2. **Does the top-level `cache_control` limit routing to Anthropic direct?** Zed's comment says it "restricts routing to Anthropic direct" (G4, repeated in the pull request text in G121). OpenRouter's caching page read today says: "**Automatic caching** (top-level `cache_control`) is supported on the **Anthropic**, **Google Vertex AI**, **Azure**, and **Amazon Bedrock** providers, as well as Claude Platform on AWS." (G-s11). The page is the primary source and is newer than Zed's change, so the page wins for today. rig sends the top-level form (G28).
3. **Is `usage: {include: true}` needed?** Zed and Goose still send it (G6; Goose at `openrouter.rs:658`). OpenRouter's usage page read today says: "On Chat Completions, the `usage: { include: true }` and `stream_options: { include_usage: true }` parameters are deprecated and have no effect." (G-s14). rig sends neither and its recorded streams have usage (G44). The page wins.
4. **LC3, "unsupported parameters are silently ignored, not rejected".** Several clients record 400 answers for a parameter or value the route does not take: a reasoning disable on a mandatory-reasoning route (G17, G64, G88), an effort level outside the route's list (G64), `configuration_update` on a model that lacks it (G90), a forced `tool_choice` while Qwen reasons (G89), and "some routes 400 on ``reasoning``" (G64). One case goes the other way: OpenRouter drops `reasoning` without an error when Claude gets a forced tool choice (G89). So LC3 holds for some parameters and fails for others. I did not re-read the documentation sentence behind LC3.
5. **"OpenRouter strips Gemini's `thought_signature`"** (G115, an open issue in OpenRouter's examples repository) against the recorded Gemini frame with a signature in `reasoning_details` (G46) and OpenRouter's SDK team describing the signature as round-tripped (G113). The recorded frame and the staff comment win. The issue describes clients that do not read `reasoning_details`.
6. **What happens to a replayed plain `reasoning` string for Claude?** OpenRouter's SDK pull request says "The API would construct a thinking block from the orphaned `reasoning` text, and Anthropic would reject it" (G127). genai's pull request says a replay with plaintext only is accepted and the model continues without its prior reasoning (G128). Hermes says an enabled `reasoning` field on a tool-continuation turn without a replayed thinking block makes OpenRouter emit `thinking: {type: "disabled"}` for always-thinking models (G63). These describe three different outcomes on different dates and models. Unresolved.
7. **Is `provider` on every stream chunk?** Pydantic AI's type comment says it "May be absent in early streaming chunks; only the final chunk typically carries the provider name" (G85). rig's recorded frames from October 2026 have `provider` on every frame I printed (G37, G46), and Hermes reads it from the first chunk that has it (G78). Likely changed over time. Unresolved.
8. **`max_completion_tokens`.** Pydantic AI: "OpenRouter only accepts the older `max_tokens` field" (G92). langchain-openrouter sends `max_completion_tokens` when the user sets it (G101). Not tested by either source in what I read. Unresolved.
9. **LC6, "each entry has a vendor `format` tag".** Usually true (G37, G41, G46). Exceptions: entries with only `type` and `data` during an incident (G116), and `"format": "unknown"` for an open-weight model (G49). A client cannot rely on the tag being present.
10. **LC12, the shape of the final usage chunk.** Confirmed for October 2026 (G44, G118). A stream recorded in January 2026 (G20) has the usage chunk with `finish_reason: null`, so the repeated finish reason is newer than January 2026.
11. **Within Hermes.** One comment says its chat-completions path "never replays signed thinking blocks" (G63) while its carrier table replays `reasoning_details` to OpenRouter (G60, G61). I did not trace which wins for Claude.
12. **Against the workspace's 2026-11-17 survey.** No factual conflict, two updates. First, rig moved on from `5393668`: its OpenRouter provider is now a dialect entry on the shared OpenAI wire, and the folder `providers/openrouter` holds only ids and typed options. Second, Goose's repository now lives at `aaif-goose/goose` (GitHub answers `block/goose` with a 301) and its provider code is in crates named `oi-kit-providers` and `oi-kit-provider-types`, where the survey cites `crates/goose-providers` and `crates/goose-provider-types`.

## Open questions blocked by closed source

1. **What does OpenRouter do with the endpoint suffix on a Claude signature when the next request routes to a different upstream (for example the signature came from Bedrock and the request goes to Anthropic direct)?** Tried: the documentation mirror (zero hits for `endpoint_slug`), rig's fixtures (every replay stayed on one route or was pinned), the client sources (none parses the suffix). Fill: none. The suffix exists so OpenRouter can tell which endpoint signed the block (G37); what it does with that is unknown. For Gemini the unhandled version of this problem produced 400s in mid 2026 (G113).
2. **Does Claude accept replayed `reasoning_details` through OpenRouter?** Tried: Zed, rig, genai, OpenRouter's page. Fill found: yes, from rig's recorded traffic and genai's live check (G39, G128). Zed's contrary comment is unexplained.
3. **How does OpenRouter rebuild the Anthropic content order from one `reasoning_details` array when a turn has thinking, text, more thinking, and several tool calls?** Tried: rig fixtures (single thinking block per turn only), client sources. Fill: partial, from other projects. Hermes keeps a separate ordered block list for the native Anthropic route because "reconstruction reorders signed blocks -> HTTP 400" (G61); genai closes a block at each signature (G49). Neither shows what OpenRouter does.
4. **Which of the three Gemini signature shapes does OpenRouter send when?** (`reasoning.encrypted` entry with `id` equal to the tool call id, `reasoning.text` entry with only `signature`, or a field on the tool call.) Tried: Zed, Goose, Hermes, Pydantic AI, rig fixtures, issue reports. Fill: none as a rule. Clients read all three (G2, G14, G46, G62, G82, G113).
5. **Does OpenRouter forward `anthropic-beta`, `x-anthropic-beta`, and `x-grok-conv-id` to the upstream?** Tried: Aider, Hermes. Fill: Hermes and Aider depend on it (G56, G68, G77), which suggests yes, with no documentation read by me to confirm.
6. **How is `cache_control` on tool messages and at the end of a Gemini request translated?** Tried: Hermes, Pydantic AI. Fill: client observations only: a part-level marker on `role: tool` is moved onto the `tool_result` block, a top-level marker on `role: tool` hangs, and a trailing marker on Gemini's Vertex route bills the prompt twice (G67, G87).
7. **Does OpenRouter now keep a tool-call continuation on the upstream that produced a provider-scoped signature?** Tried: the SDK issue (G113). Fill: as of 2026-06-17 no, and the stated workaround was pinning `provider.order` with `allow_fallbacks: false`. Later state unknown.
8. **Does a replayed plain `reasoning` string become an upstream thinking block?** See Conflict 6. Fill: conflicting client reports, none decisive.
9. **Does the Responses route return a top-level `provider`?** rig's extras say "Responses when sent" (G32) and the recorded unary Responses reply lacks the key (G107). Fill: none.

## Unverified

- G15 (Goose's streamed `reasoning_details` being unused) is a code reading at one commit. I did not build or run Goose.
- Goose's `api/alpha/decisions` route and its default model `typesafe/jev-1.13` (G24) were not checked against OpenRouter's documentation.
- Goose's comment that OpenRouter applies `middle-out` by default for endpoints with 8k context or less (G19) was not checked. LC16 suggests it may be stale.
- Hermes' claim that `session_id` is used directly as the sticky routing key (G66), and the 14 to 20 percent cache rewrite figure (G70), which cites `NousResearch/api#227`, a repository I could not read.
- That OpenRouter serves OpenAI reasoning models from the Responses API upstream (G41) is my inference from `native_finish_reason: "completed"`, the format tag `openai-responses-v1`, and the error wording in G124.
- The start date of the signature suffix (G37) is bounded by fixture dates only.
- rig's `n = 2` remark (G42): I did not open the six `terminal_metadata_matrix` fixtures.
- Codex (G105): the sparse checkout covered `codex-rs/model-provider-info`, `codex-rs/core/src`, `codex-rs/codex-api/src`, and `docs`. Other folders were only searched by file name.
- Issue pages marked "search text" (G114, G115, G117, G118, G119, G120, G122, G124, G125, G127 and the pull request text in G121) were read as page text returned by the search tool, not opened one by one. For G118 and one of the two reports in G119 the state and date were not captured.
- Aider's count of `anthropic-beta` entries on `openrouter/` names (G56) is approximate.
- Nothing in this file is from memory. Model names such as `claude-opus-5.5`, `gpt-6-astra`, and `gemini-3-flash-preview` are quoted from the sources as found.

## Workspace documents used

- **2026-11-17, "Anthropic Messages API harness survey" (finished report in the cabinet output folder).** Used Table E1 and its source list as the map of the nine projects and their earlier pinned commits, and took two facts: Aider's thinking replay is "None" and Codex is "Responses API only". Re-checked both today at pinned commits (G59, G103). Aider is at the same commit as the survey; every other project has moved. Corrections to names and locations are in Conflict 12. The survey's own OpenRouter content is limited to three mentions drawn from OpenRouter's documentation, and my findings do not contradict them: it lists `reasoning_details` as one of three non-standard conventions and `prompt_tokens_details.cache_write_tokens` as the cache-write field, which G44 confirms from recorded traffic.
- **`runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, dated 2026-10-28.** I used its OpenRouter claims as listed in section 8 of the shared brief (LC1 to LC19) and did not open the document beyond a search for `reasoning_details` (two mentions). Checked against today's evidence: LC4, LC8, LC10, LC11, LC12, LC17 agree with client code or recorded traffic; LC3 is contradicted in part (Conflict 4); LC6 holds with exceptions (Conflict 9); LC13 gains a second `limit_source` value, `upstream_provider_shared_pool` (G117).
- A content search of the workspace's Markdown files for `reasoning_details` (outside the scratch folder and this research series) found only these two documents. `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` has zero mentions of OpenRouter.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| G-s1 | `zed-industries/zed` at `d31d1074a07b7bbaa43a52fef7847a00dd491baf` (2026-11-22): `crates/open_router/src/open_router.rs`, `crates/language_models/src/provider/open_router.rs`, `crates/language_model_core/src/chat_completion.rs`, `crates/language_model_core/src/request.rs` | 2026-11-22 | harness source |
| G-s2 | `aaif-goose/goose` (cloned as `block/goose`, which redirects) at `36b29113aff691e1a46a21a5c25900824caf9949` (2026-11-20): `crates/oi-kit-providers/src/openrouter.rs`, `openrouter_format.rs`, `http_status.rs`; `crates/oi-kit-provider-types/src/formats/openai.rs`, `cache_semantics.rs`, `canonical.rs`, `canonical/name_builder.rs`; `crates/goose/src/providers/openrouter_def.rs` | 2026-11-22 | harness source |
| G-s3 | `0xPlaygrounds/rig` at `a8673f775ca78004f011e5ad5b615147fdcd4bff` (2026-11-22): `crates/rig-core/src/providers/openrouter/`, `crates/rig-core/src/providers/openai/wire.rs`, `wire/dialects.rs`, `wire/chat.rs`, `wire/chat/finish.rs`, `wire/chat/history_tests.rs`, `wire/chat/hard_case_tests.rs`, `options.rs`; `crates/rig-core/src/completion/history.rs`, `completion/message/native.rs`; `crates/rig-core/src/providers/anthropic/streaming/tests.rs`; `crates/rig-core/src/test_utils/raw_parity/tests.rs` | 2026-11-22 | harness source |
| G-s4 | Same rig commit: recorded fixtures `crates/rig-cassette/fixtures/cassettes/openrouter/**` (63 files, recorded 2026-09-22 to 2026-10-03) and their tests `crates/rig-cassette/tests/providers/openrouter/cassette/*.rs` | 2026-11-22 | harness source (recorded OpenRouter traffic) |
| G-s5 | `jeremychone/rust-genai` at `2a8a5b39c96028d2143d8754ee2bba42a79e32c5` (2026-11-18): `src/adapter/adapters/openai/adapter_impl.rs`, `adapter_shared.rs`, `streamer.rs`; `src/adapter/adapters/anthropic/adapter_shared.rs`; `src/adapter/adapters/support.rs`, `all_adapters.rs`; `tests/tests_yakbak_parser_regressions.rs` | 2026-11-22 | harness source |
| G-s6 | `Aider-AI/aider` at `5dc9490bb35f9729ef2c95d00a19ccd30c26339c` (2026-05-22): `aider/openrouter.py`, `aider/models.py`, `aider/exceptions.py`, `aider/onboarding.py`, `aider/resources/model-settings.yml` | 2026-11-22 | harness source |
| G-s7 | `NousResearch/hermes-agent` at `0f602c5f6f18615632535251d62a2db74ce93b69` (2026-11-22): `plugins/model-providers/openrouter/__init__.py`; `agent/transports/chat_completions.py`, `agent/message_sanitization.py`, `agent/chat_completion_helpers.py`, `agent/error_classifier.py`, `agent/prompt_caching.py`, `agent/agent_runtime_helpers.py`, `agent/nous_wire.py`, `agent/turn_usage.py`, `agent/usage_pricing.py`, `agent/turn_recovery.py`, `agent/turn_api_request.py`, `agent/rate_limit_credits.py`, `agent/agent_init.py`, `agent/reasoning_params.py`, `agent/stream_diag.py` | 2026-11-22 | harness source |
| G-s8 | `pydantic/pydantic-ai` at `463a6d519b7e4c8e038d5a2559b0d26281df6937` (2026-11-22): `pydantic_ai_slim/pydantic_ai/models/openrouter.py`, `models/_reasoning_details.py`, `models/__init__.py`, `providers/openrouter.py`, `_canonical_system.py` | 2026-11-22 | harness source |
| G-s9 | `langchain-ai/langchain` at `8ff3cd30eebfe37712a91f11e3f4b61a545e7a37` (2026-11-20): `libs/partners/openrouter/langchain_openrouter/chat_models.py`, `libs/partners/openrouter/pyproject.toml` | 2026-11-22 | harness source |
| G-s10 | `openai/codex` at `227fa4e805d44988b0888390f41afe1358e9d24d` (2026-11-22): `codex-rs/model-provider-info/src/lib.rs`, `capabilities.rs`; `codex-rs/core/src/client.rs`; `codex-rs/codex-api/src/common.rs` | 2026-11-22 | harness source |
| G-s11 | https://openrouter.ai/docs/guides/best-practices/prompt-caching | 2026-11-22 | OpenRouter doc |
| G-s12 | https://openrouter.ai/docs/cookbook/coding-agents/codex-cli | 2026-11-22 | OpenRouter doc |
| G-s13 | https://openrouter.ai/docs/guides/best-practices/reasoning-tokens | 2026-11-22 | OpenRouter doc |
| G-s14 | https://openrouter.ai/docs/cookbook/administration/usage-accounting | 2026-11-22 | OpenRouter doc |
| G-s15 | Workspace: 2026-11-17, "Anthropic Messages API harness survey" (finished report), Table E1 and source list | 2026-11-22 | workspace document |
| G-s16 | Workspace: `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (claims LC1 to LC19 as listed in the shared brief) | 2026-11-22 | workspace document |
| G-s17 | https://github.com/pydantic/pydantic-ai/issues/3994 | 2026-11-22 | issue report (third-party) |
| G-s18 | https://github.com/pydantic/pydantic-ai/issues/6900 | 2026-11-22 | issue report (third-party) |
| G-s19 | https://github.com/NousResearch/hermes-agent/issues/42991 | 2026-11-22 | issue report (third-party) |
| G-s20 | https://github.com/NousResearch/hermes-agent/issues/43432 | 2026-11-22 | issue report (third-party) |
| G-s21 | https://github.com/NousResearch/hermes-agent/issues/25970 | 2026-11-22 | issue report (third-party) |
| G-s22 | https://github.com/OpenRouterTeam/ai-sdk-provider/issues/491 | 2026-11-22 | issue report (third-party, with an OpenRouter staff reply) |
| G-s23 | https://github.com/OpenRouterTeam/ai-sdk-provider/issues/239 | 2026-11-22 | issue report (third-party) |
| G-s24 | https://github.com/RooCodeInc/Roo-Code/issues/10307 | 2026-11-22 | issue report (third-party) |
| G-s25 | https://github.com/OpenRouterTeam/openrouter-examples/issues/67 | 2026-11-22 | issue report (third-party) |
| G-s26 | https://github.com/pydantic/pydantic-ai/issues/4370 | 2026-11-22 | issue report (third-party) |
| G-s27 | https://github.com/anomalyco/opencode/issues/41469 | 2026-11-22 | issue report (third-party) |
| G-s28 | https://github.com/LibreChat-AI/agents/issues/662 | 2026-11-22 | issue report (third-party) |
| G-s29 | https://github.com/anomalyco/opencode/issues/18440 | 2026-11-22 | issue report (third-party) |
| G-s30 | https://github.com/pydantic/genai-prices/issues/345 | 2026-11-22 | issue report (third-party) |
| G-s31 | https://www.meshweaver.cloud/AI/ProviderStreamFaults | 2026-11-22 | blog (third-party) |
| G-s32 | https://github.com/zed-industries/zed/issues/52576 | 2026-11-22 | issue report (third-party) |
| G-s33 | https://github.com/zed-industries/zed/commit/7fab4c0527e10a466439b819d4e9e7fe34cd989c | 2026-11-22 | harness source (commit message) |
| G-s34 | https://git.smallrat.net/mitch/zed/commit/f844c93c948dfbe68d99ec9fb7cd2482a8622a09 (a mirror of Zed commit `f844c93`, pull request 57498) | 2026-11-22 | harness source (commit message, mirror) |
| G-s35 | https://github.com/aaif-goose/goose/issues/11260 | 2026-11-22 | issue report (third-party) |
| G-s36 | https://github.com/openclaw/openclaw/issues/24900 | 2026-11-22 | issue report (third-party) |
| G-s37 | https://github.com/karthink/gptel/issues/747 | 2026-11-22 | issue report (third-party) |
| G-s38 | https://github.com/pydantic/pydantic-ai/issues/3617 | 2026-11-22 | issue report (third-party) |
| G-s39 | https://github.com/OpenRouterTeam/ai-sdk-provider/issues/423 | 2026-11-22 | issue report (third-party) |
| G-s40 | https://github.com/OpenRouterTeam/ai-sdk-provider/pull/442 | 2026-11-22 | issue report (third-party, written by an OpenRouter SDK maintainer) |
| G-s41 | https://github.com/jeremychone/rust-genai/issues/301 | 2026-11-22 | issue report (third-party) |
| G-s42 | https://github.com/NousResearch/hermes-agent/issues/90216 | 2026-11-22 | issue report (third-party) |
