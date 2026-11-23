---
produced: 2026-11-22
title: Runvil today code map for the OpenRouter deep dive - what the Gateway, the gateway client, and the Engine do on each axis of the universal inference API (request fields, message type, reasoning, tool calls, streaming, usage, finish reasons, errors, model discovery, caching, multimodal, gemma3_tool_code dialect, Upstream and Protocol seams)
---

# Runvil today: a code map of the Gateway, the gateway client, and the Engine on each axis of the universal inference API

- What was read: the Rust source of `runvil-harness` (the client crate `harness-gateway-client`, the Engine crates `runvil-model-client`, `runvil-lua`, `runvil-types`, `runvil-runtime`, and the runner's broker trait), `runvil-gateway` (the `protocol`, `app`, `config`, `routing`, and `gateway-api-types` crates, `openapi.json`, the README), and `runvil-cloud-providers` (the sheet types and six provider files), plus three decision pages, the Gateway's README, three `AGENTS.md` files, and the 2026-11-17 workspace report on Anthropic's Messages API. Local code reading only. Nothing was built, run, or edited.
- How many sources: 53 source entries (48 code files or file groups at pinned commits, 3 groups of decision pages and README or AGENTS files, 1 workspace report, 1 dependency source file).
- Read date: 2026-11-22. Commits: `runvil-harness` `22a0992f`, `runvil-gateway` `52fd92d`, `runvil-cloud-providers` `50961b2`, `runvil-design` `217c27d` (see L1).

## Candidate matrix rows (one row per axis, in place of the brief's feature table)

Every cell states what the code does at the commits above. All rows have the basis `Workspace`. "Upstream" means the vendor or server the Gateway forwards to. "Passes through" means the Gateway parses the JSON into an untyped value and writes it back out without reading it.

| Axis | Gateway today | Client today | Engine today | What Lua can see or set today | Evidence |
|---|---|---|---|---|---|
| 1. Request fields | `ChatRequest` names `model`, `messages`, `stream`. Every other top-level field sits in `rest` and passes through. The upstream gets the body with `model` swapped for the `[[model]].upstream` value and `stream` set true on the stream path, plus a bearer header. The handler reads `tool_dialect`, `kind`, `upstream_name`, and the endpoint from the model entry. The capability keys go to the catalog route only. Message roles are checked against `system`, `user`, `assistant`, `tool`, `function`, `developer`. The JSON body limit is axum's default of 2 MB. | The body's key set is exactly `model`, `messages`, `stream: true`, `stream_options: {include_usage: true}`, then `tools` and `tool_choice: "auto"` when the tool list is non-empty, then `temperature`, `max_tokens`, and `chat_template_kwargs: {enable_thinking}` when set. One function builds it. The request has a bearer header when a key is set and JSON content, nothing else. | `ModelInvocation` has `temperature` (0.0 to 2.0), `max_tokens` (at least 1), `thinking` (optional bool). `ModelBinding` adds `alias`, `description`, `id`, `context`, `provider`, and the catalog `ModelInfo`. `CompletionOptions` is the model name plus one `ModelInvocation`. `Effect::Chat` holds `binding`, `messages`, `after`, `tools`, `round`. | Sets `temperature` and `max_tokens` through `models.use(role or handle, {...})`. Any other option key raises. The thinking switch comes from the front matter key `models: <role>: thinking`, and Lua reads it. A handle exposes `name`, `label`, `model_id`, `description`, `context`, `provider`, `thinking`, `temperature`, `max_tokens`, all read-only. | L3-L14 |
| 2. The message type | Messages are `Vec<Value>`, response choices are `Vec<Value>`, and a chunk's `delta` is a `Value`. Unknown message fields pass through in both directions. The check on a message is: an object, a supported string `role`, and either a `content` key or a call key. | Serializes each Engine `Message` as given (four keys at most). Builds the reply from the fields it accumulates, so a reply field outside that set ends at the client. | Wire `Message`: `role`, `content` (a JSON value: string or parts array), `tool_call_id`, `tool_calls`. List record `MessageRecord`: `role`, `content` (text or parts), `tool_calls`, `meta`. All four record fields are Lua-visible. The state Lua cannot read sits on the list: `last_send` and the frozen tool sets. The projection sends `role`, `content`, `tool_call_id`, `tool_calls`. It joins the leading system records into one message, joins consecutive user records, joins consecutive text-only assistant records, and folds a text-only assistant record into the tool-call record after it. | Reads five fields per record: `role`, `content`, `tool_calls`, `tool_call_id`, `meta`. Appends with `system`, `user`, `assistant`, `tool`, `append`. `meta` is any JSON up to 8192 bytes. Any other field on an appended table is dropped at the builder. Records the loop appends have plain text content and `meta` absent. | L15-L24 |
| 3. Reasoning | `delta` and `message` pass through, so `reasoning_content`, `reasoning`, and `reasoning_details` reach the caller as the upstream wrote them, and a caller-written reasoning field on a request message reaches the upstream. `chat_template_kwargs` passes through as written. `thinking` (`never`, `always`, `switchable`) is catalog metadata from the config. `effort_levels`, `default_effort`, `adaptive_thinking` are validated and published. | Reads reasoning text from the first non-blank of `reasoning_content`, `reasoning`, `thinking`, as a string. Streams it to the Host's callback as `StreamDelta::Reasoning` and stores it on `Completion.reasoning_content`. A reply with reasoning and blank text is `EmptyReply` with "reasoning content was present but ignored". Requests are built from `Message`, whose four keys leave reasoning out. | Emits one `Thinking` event per round (turn, round id, model, text) and the debug `Response` event, then takes the result out of the completion. The record appended to the list is the reply text or the tool calls. The precheck's usage anchor subtracts `reasoning_tokens`. The switch is `Option<bool>` on the binding, checked against the model's `ThinkingMode` at the run's first step. | Reads `handle.thinking` (nil, true, or false). The front matter sets it per role. Reasoning text reaches the Host as an event, and the `ChatResult` the loop passes along is an opaque userdata. | L25-L32 |
| 4. Text beside tool calls, block order, empty replies | Passes through whatever the upstream sent. | The normalizer returns the tool calls as soon as `tool_calls` is non-empty and never reads `content` on that branch, so text beside tool calls ends there. The accumulator holds one `content` string, one reasoning string, and calls keyed by `index`, so the order between text, reasoning, and calls is gone. Only choice index 0 is read. Blank text with zero calls fails as `EmptyReply` with the finish reason as a fact. | For a tool round the loop appends an assistant record with empty content and the calls, then one tool record per call. An empty reply that finished with `stop` after an answered call ends the loop with an empty assistant record. Any other empty reply raises `empty_model_reply`. A `models.infer` reply with tool calls fails as `MalformedResponse`. | Sees the assistant record with `content == ""` and `tool_calls`, and the final reply as `messages[#messages].content`. | L33-L39 |
| 5. Tools | `tools` and `tool_choice` pass through, except on the `gemma3_tool_code` dialect, which removes both. `parallel_tool_calls` is a published capability key. | Sends each tool as `{"type":"function","function":{name, description, parameters}}` and `tool_choice: "auto"`. Merges stream fragments by `index` and concatenates `id`, `name`, `arguments`. Requires `type` equal to `"function"`, a string `id`, a string `name`, and `arguments` as a JSON string that parses. A batch cut off by `length` or `content_filter` fails as `MalformedResponse`. | `ToolCall::from_parts` refuses a blank id, a blank name, and non-object arguments. `Completion::from_result` refuses an empty batch and two calls with one id. Tool names match `[A-Za-z][A-Za-z0-9_-]{0,63}`. The projection requires every call id to be unique across the whole list and each batch to be answered directly, and it writes `arguments` back as the re-serialized object. A batch of several calls runs as one `tool_calls` request and its records append in the model's order. | Reads `tool_calls` as `{id, name, arguments}` records. Offers tools through the tools API (out of this map's scope). The `tool_choice` value and the tool wire shape are fixed in the client. | L40-L49 |
| 6. Streaming | Parses each `data:` line into `ChatChunk {model, choices: [{index, delta, ...}], ...}` and writes it back out. Skips blank lines, `:` comments, and `event:`, `id:`, `retry:` lines. A line that fails to parse (no string `model`, no `choices`, a choice missing `index` or `delta`) or has empty `choices` is logged and skipped. Unknown top-level, choice-level, and delta fields survive on a chunk that parses. The upstream's `[DONE]` is consumed and the relay writes its own, also after an error event and after an upstream body that ended without one. Forwards the upstream `Content-Type` and `Cache-Control`. The stream client's only limit is a 10 second connect timeout. | Reads `data:` lines and skips blank lines, comments, and other fields. Requires `[DONE]`. A chunk that is invalid JSON fails the round. Top-level `error` fails the round through the stream classifier. Keeps top-level `usage`, `timings`, `metrics` from the latest chunk that has them, including a chunk with empty or absent `choices`. Waits at most 120 seconds for headers and for each next chunk, and accepts at most 16 MiB. | Receives one finished `Completion` per `Chat` effect. `InferenceBroker::chat` returns the whole reply. A Host that shows live text calls `GatewayBroker::chat_streaming`, which hands out `StreamDelta::Text` and `StreamDelta::Reasoning`. | Lua receives the round's outcome when the round ends. | L50-L64 |
| 7. Usage and cost | On the pass-through path a `usage` object survives only on a chunk that has at least one choice. The OpenAI-shaped usage-only chunk (`choices: []`) is skipped. The emulated dialect writes its own empty-`choices` summary chunk with `usage`, `timings`, `metrics`. Every number in a reply comes from the upstream. | Parses `usage` into `prompt_tokens`, `completion_tokens`, `total_tokens` (all three required), `prompt_tokens_details.cached_tokens`, `completion_tokens_details.reasoning_tokens`. Other usage keys are ignored. A malformed section becomes absent with a diagnostic line. | `Usage` has `prompt_tokens`, `completion_tokens`, `total_tokens`, `cached_tokens`, `reasoning_tokens`. `CallMetrics` groups `usage`, `llama`, `vllm`, `client`. Usage feeds the precheck's anchor and the `AssistantReply` event. Prices live in the provider sheet (`prompt_per_mtok`, `completion_per_mtok`), which the config UI reads. | The Host reads usage from the reply event. Inside the Engine it feeds the precheck, which Lua meets as a compactor call. | L65-L70 |
| 8. Finish reasons | `finish_reason` is an unnamed choice field and passes through. The emulated dialect writes `"tool_calls"` when a fence parses. | Stores the string as given. Compares it in one place: `length` or `content_filter` with tool calls present fails the round. | Compares `"stop"` (the loop's clean exit after an empty reply) and `"length"` (a truncation event on a text reply). The loop branches on the outcome variant. The run log and the reply event record the string. | Reads `finish_reason` only as a field of a caught `empty_model_reply` error. | L71-L73 |
| 9. Errors | Every failure is `{"error": {"message", "type", "code"}}`. An upstream 4xx keeps its status with type `invalid_request_error` and code `upstream_client_error`. Every upstream 5xx becomes 502 with `server_error` and `upstream_error`. The message is `upstream returned {status}`. The upstream body goes to the Gateway's log as bounded `code`, `type`, `message` fields. `Retry-After: 5` is set for `model_loading` only. A transport failure mid-stream becomes an envelope `data:` event followed by `[DONE]`. A bare upstream `{"error": ...}` chunk is skipped by the chunk parser. | Classifies by status and body phrases: 400 or 413 with an overflow phrase is `ContextOverflow`, 429 is `QuotaExhausted` or `RateLimited`, 503 or 529 is `Overloaded`, other 5xx is `ServerError` (or `Overloaded` when the body says so), 401 or 403 is `Unavailable`, 400 with a refusal word is `Refused`, the rest `Rejected`. An in-stream envelope is classified by the phrases in its `message`, with `Transport` as the default. Each round is one HTTP attempt. | `ContextOverflow` becomes the provider overflow and calls the compactor. `EmptyReply` goes to the loop's exit rule. Every other code fails the call. | A caught failure is a table with `kind` (`model_failure`, `empty_model_reply`, `context_exhausted`, `model_required`), `code` (such as `rate_limited`), and `message` (the fixed phrase). | L74-L85 |
| 10. Model discovery | `GET /v1/models` returns `id`, `object`, `kind`, `description`, `context`, `thinking`, and nine flattened capability keys: `max_output`, `default_temperature`, `images`, `parallel_tool_calls`, `effort_levels`, `default_effort`, `adaptive_thinking`, `voices`, `provider`. All values come from `gateway.toml`. The config UI fills a new `[[model]]` from the provider sheet. | Reads `id`, `description`, `context`, `thinking`, `provider`. Skips an entry without `context`. Fails the catalog when an entry has `context` and lacks `thinking`. | `ModelInfo` holds `id`, `description`, `context`, `thinking`, `provider`. The run's first step checks each declared `min_context` and `thinking` requirement. | Declares `min_context` and `thinking` per role in front matter. Reads the handle fields listed in row 1. | L86-L96 |
| 11. Caching | The body passes through, so a caller-written `cache_control` would reach the upstream. The client's body is the nine keys in row 1. | Reads `prompt_tokens_details.cached_tokens` into `Usage.cached_tokens`. | The stable-prefix rules: `replace` works only inside a compactor, system records lead the list, the projection is recomputed per dispatch and "is the same for every model", and the tool sets freeze at the section entry's first loop round. `commit` records how many leading messages a request shares with the previous send. | Edits the prefix only through a compactor. | L97-L101 |
| 12. Multimodal content | A message's `content` passes through in any shape. `images` is a published capability key. The 2 MB body limit applies to inline image data. | Sends `content` as the Engine built it (string or parts array). A reply whose `content` is anything other than a string or null fails as `MalformedResponse`. | Content parts are `text` and `image_url` (a URL string of any scheme). Parts are legal on any role. A system block of several records refuses parts. The precheck counts text parts only. | Appends a parts array, for example `messages:user({{type="text", text=...}, {type="image_url", image_url={url=...}}})`. | L102-L107 |
| 13. The `gemma3_tool_code` dialect | A module in the app crate with pure functions around the upstream call: `prepare_request` removes `tools` and `tool_choice` and inserts a system guide, `apply_response` turns `tool_code` fences into `tool_calls`, `response_chunks` re-emits a buffered reply as chunks. Selected by one string comparison on the model's `tool_dialect`. Adds the response field `gateway_warning`. | Sees ordinary OpenAI tool calls with ids `call_tool_code_N`. | Sees ordinary tool calls. | Sees the same records as on any model. | L108-L113 |
| 14. Seams | `Upstream` has `send(req, upstream_model)`, `stream(req, upstream_model)`, three more workloads, and `shutdown`. One upstream is built per `[[endpoint]]` from `base_url` and `api_key`. `Protocol` has one variant, matched in `Routing::from_config`. The model's capabilities, thinking mode, and context stay with the handler, which holds the `Model`. | Public: `GatewayBroker`, `GatewayChat`, `fetch_model_catalog`, `build_request_body`, `read_completion_stream`, `read_body_capped`, `ChunkSource`, `StreamDelta`, `escape_controls`, the two classifiers, `GatewayEndpoint`, `SecretString`. | `InferenceBroker` has `models()` and `chat(binding, messages, tools, options, round)`. A broker builds a `Completion` through `from_result` and the `with_*` setters. | Not applicable. | L114-L121 |

## Evidence

Every item has the basis `Workspace`, read on 2026-11-22 at the commits in L1. Citations give the repository, the file path, and line numbers at that commit. An item marked "Inference" is a conclusion from reading types and control flow. Nothing was run.

### Provenance

**L1. The four repositories were read at these HEAD commits, all on branch `master`.**

- `runvil-harness`: `22a0992f` (commit date 2026-11-22T04:27:05-08:00), clean tree.
- `runvil-gateway`: `52fd92d` (2026-11-19T17:22:12-08:00), clean tree.
- `runvil-cloud-providers`: `50961b2` (2026-11-13T11:55:08-08:00), clean tree.
- `runvil-design`: `217c27d` (2026-11-21T06:32:19-08:00). Ten deleted image files are uncommitted. `decisions/gateway.md` is unmodified, last committed in `639c083` on 2026-11-10.
- `runvil-workshop`: `f40a4fce` (2026-11-21T18:13:32-08:00), read for one fact (L2).
- Source: git's short HEAD id, the last commit's committer date, and the working tree status, read for each repository. Basis: Workspace.

**L2. The client's wire code and the Engine's message, projection, and loop code are unchanged since the commit the 2026-11-17 report mapped.**

- A tree comparison (a git diff summary from `aa7a2a63` to HEAD) over the files the report cites lists changes only in: the client's `Cargo.toml`, `README.md`, `config.rs`, `lib.rs`, new `resources.rs` and `timer.rs`, deleted `search.rs`, then `performers.rs`, `compactors.rs` (a doc comment, 7 lines), `lua/src/models.rs`, `protocol/answer.rs` (`VfsOutcome` moved out), a test line in `types/src/models.rs`, and the two decision pages.
- `wire/request.rs`, `wire/stream.rs`, `wire/parse.rs`, `wire/read.rs`, `wire/classify.rs`, `transport.rs`, `broker.rs`, `catalog.rs`, `failure.rs`, `client/wire.rs`, `model/options.rs`, `messages.rs`, `projection.rs`, `protocol/record.rs`, `models_loop/machine.rs`, `types/src/metrics.rs`, and `execute/turn.rs` show no difference. The last commit that touched `crates/harness-gateway-client/src/wire` is `3d87c936` (2026-11-13).
- `aa7a2a63` is present in the local object store and is not an ancestor of today's HEAD (git's ancestor test for the pair exits 1), so the history was rewritten after the report. The comparison above is tree to tree.
- In `runvil-gateway`, the last commit that touched `crates/gateway/protocol/src`, `app/src/relay.rs`, and `crates/gateway/README.md` is `fb60a7f` (2026-11-15, "Describe the Gateway API with utoipa").
- `runvil-workshop` pins the Harness at `ee89d0882ec8cecfcd0159c7f61c6741fceef612` (`Cargo.toml:28-30`, a commit of 2026-11-21). A git diff summary from that commit to HEAD over the client's `wire` folder, `transport.rs`, `broker.rs`, and `catalog.rs` is empty, so Workshop's client has the same wire code as HEAD.
- Source: git history of the three repositories, L-s62. Basis: Workspace.

### Axis 1: request fields

**L3. Every request body starts with four fixed keys.** `runvil-harness`, `crates/harness-gateway-client/src/wire/request.rs:38-43`:

```rust
    let mut body = serde_json::json!({
        "model": options.model(),
        "messages": messages,
        "stream": true,
        "stream_options": { "include_usage": true },
    });
```

Source: L-s1. Basis: Workspace.

**L4. Five more keys are conditional, and that is the whole body.** Same file, `:44-71`: `body["tools"] = Value::Array(wrapped);` and `body["tool_choice"] = Value::String("auto".into());` run only when `tools.filter(|tools| !tools.is_empty())` is `Some`. Then `body["temperature"]`, `body["max_tokens"]`, and `body["chat_template_kwargs"] = serde_json::json!({ "enable_thinking": thinking, });` are each written only inside `if let Some(...)`. The function returns `body` at `:72`. Source: L-s1. Basis: Workspace.

**L5. One function shapes the body, by rule.** `runvil-harness`, `crates/harness-gateway-client/AGENTS.md:3`: "`build_request_body` is the only place the chat-completions body is shaped, so every transport sends the same JSON for one `Chat` effect. Never build or patch a body elsewhere." A search of `runvil-harness/crates/**/*.rs` for `reasoning_effort`, `effort_levels`, `default_effort`, `cache_control`, `response_format`, `parallel_tool_calls`, `prompt_cache`, `max_output`, `default_temperature`, `native_finish_reason`, `reasoning_details`, `cache_write`, `X-Runvil-Client`, and `retry-after` (case-insensitive) matched four lines: three name the Engine's own test module `parallel_tool_calls`, and one is a comment in a `require` test that has the words "retry after". Source: L-s11, search run 2026-11-22. Basis: Workspace.

**L6. The HTTP request is a POST of that JSON with an optional bearer header.** `runvil-harness`, `crates/harness-gateway-client/src/transport.rs:271-276`:

```rust
        let mut request = http
            .post(format!("{}/chat/completions", self.base_url))
            .json(&request_body);
        if let Some(key) = &self.key {
            request = request.bearer_auth(key.expose());
        }
```

The Gateway's fair-queue header is `X-Runvil-Client` (`runvil-gateway`, `crates/gateway/app/src/relay.rs:25-26`, whose comment says an absent header means the `default` bucket). The client sets the two request parts above and no others, so every Harness request is scheduled in the `default` bucket. Source: L-s7, L-s35. Basis: Workspace.

**L7. The Engine's per-request settings are three optional values.** `runvil-harness`, `crates/runvil-internal/model-client/src/model/options.rs:68-87`: `pub struct ModelInvocation` with `pub temperature: Option<Temperature>`, `pub max_tokens: Option<NonZeroU32>`, and `pub thinking: Option<bool>`, documented as "The thinking switch, sent as `chat_template_kwargs.enable_thinking`, when set." `Temperature::new` (`:30-44`) refuses a value outside `[0.0, 2.0]` with "temperature {value} is outside the supported range [0.0, 2.0]". Source: L-s13. Basis: Workspace.

**L8. A binding is one model plus those settings, and the options sent to a broker are the model name plus the same settings.** Same file, `:120-132`: `ModelBinding` has `alias`, `description`, `id: ModelId`, `invocation: ModelInvocation`, `context: NonZeroU32`, `provider: Option<String>`, `catalog: Option<ModelInfo>`. `:255-265`: `CompletionOptions` has `model: String` and `invocation: ModelInvocation`. `:249-251`: `CompletionOptions::new(self.id.name()).with_invocation(self.invocation.clone())`. `:319-328`: `with_model` "Replaces the model name sent in the request and keeps every other setting." The effect loop builds the options from the binding: `runvil-harness`, `crates/harness-internal/runner/src/effect_loop.rs:583-585`, `let options = binding.completion_options();` then `broker.chat(binding, messages, tools, options, round).await`. Source: L-s13, L-s26. Basis: Workspace.

**L9. Lua sets two options, and any other key is refused.** `runvil-harness`, `crates/runvil-internal/lua/src/models.rs:181-187` (`struct UseOptions { temperature: Option<Temperature>, max_tokens: Option<NonZeroU32> }`) and `:299-306`:

```rust
        "temperature" => parsed.temperature = Some(decode_temperature(value)?),
        "max_tokens" => parsed.max_tokens = Some(decode_max_tokens(value)?),
        other => {
            return Err(mlua::Error::external(format!(
                "models.use option {other:?} is unknown: expected temperature or max_tokens"
            )));
```

The handle's readable fields are registered at `:108-118`: `name`, `label`, `model_id`, `description`, `context`, `provider`, `thinking`, `temperature`, `max_tokens`. Source: L-s23. Basis: Workspace.

**L10. The thinking switch is set from the prompt's front matter when the run is built.** `runvil-harness`, `crates/runvil-internal/runtime/src/execute/fill.rs:39-41` reads `let thinking = declared.and_then(RoleRequirements::thinking);` and `:62-69` builds the binding with `ModelInvocation::new().with_thinking(thinking)`. The check at `:79-90` fails only two pairs: `Some(true)` on `ThinkingMode::Never` and `Some(false)` on `ThinkingMode::Always`. Inference: a declared `thinking: true` on an `Always` model, or `thinking: false` on a `Never` model, passes the check and the client sends the switch for it too, since the binding's value is `Some`. Confidence: high (two short functions). Source: L-s25. Basis: Workspace.

**L11. The Gateway names three request fields and keeps the rest verbatim.** `runvil-gateway`, `crates/gateway/protocol/src/wire.rs:34-47`:

```rust
pub struct ChatRequest {
    /// The model name, resolved against the routing table.
    pub model: String,
    /// The conversation messages, passed through to the backend verbatim.
    pub messages: Vec<Value>,
    /// Whether the caller asked for a streaming (SSE) completion. Absent
    /// means non-streaming; an absent `stream` is never forwarded.
    #[serde(default, skip_serializing_if = "is_false")]
    pub stream: bool,
    /// Every field the gateway does not name, preserved verbatim.
    #[serde(flatten)]
    pub rest: Map<String, Value>,
}
```

The module doc at `:6-7`: "The message and choice payloads are kept as opaque JSON so everything the gateway does not route passes through untouched." The message check (`:110-127`) requires an object, a string `role` in `SUPPORTED_ROLES` (`:59-66`: `"system"`, `"user"`, `"assistant"`, `"tool"`, `"function"`, `"developer"`), and `content` or `tool_calls` or `function_call`, and says "Unknown fields are left untouched for verbatim passthrough." Also in `runvil-design`, `decisions/gateway.md` (2026-11-10): "`ChatRequest` names three fields, `model`, `messages`, and `stream`, and the Gateway forwards every other field and every message unchanged." Source: L-s30, L-s60. Basis: Workspace.

**L12. The OpenAI upstream changes two things in the body and adds a bearer header.** `runvil-gateway`, `crates/gateway/protocol/src/upstream/openai.rs:268-279`:

```rust
    async fn stream(
        &self,
        mut req: ChatRequest,
        upstream_model: &str,
    ) -> Result<StreamedChunks, ProtocolError> {
        let requested = std::mem::replace(&mut req.model, upstream_model.to_string());
        req.stream = true;
        let response = self
            .post(&self.http_stream, "chat/completions", &req)
            .await?;
        Ok(sse_chunks(response, requested))
    }
```

`post` (`:59-68`) is `client.post(format!("{}/{path}", self.base_url)).json(body)` plus `builder.bearer_auth(self.api_key.expose())` when the key is non-empty. Those are the only request parts it writes. A vendor header (a version header, an attribution header) would be new code in this function. Source: L-s32. Basis: Workspace.

**L13. The chat handler reads four things from the model entry: its kind, its endpoint, its tool dialect, and its upstream name.** `runvil-gateway`, `crates/gateway/app/src/relay.rs:121-193`: the handler calls `require_kind(&model, ModelKind::Chat)`, `model.endpoint.queue.admit(...)`, compares `model.tool_dialect == crate::dialect::GEMMA3_TOOL_CODE`, and calls `.stream(request, &model.upstream_name)` or `.send(request, &model.upstream_name)`. `runvil-design`, `decisions/gateway.md` section 11 item 3: "The Gateway parses `default_max_tokens` and leaves it unused." and section 1: "The capability keys describe a model to clients, and the Gateway forwards each request as the client wrote it." A search of the Gateway's non-test Rust for `default_max_tokens`, `default_temperature`, `max_output()`, `parallel_tool_calls`, and `adaptive_thinking` finds them only in `gateway-api-types` and in the config crate's struct, accessor, and validator. `app/src/relay.rs` and the `protocol` crate are outside that list. Source: L-s35, L-s60, search run 2026-11-22. Basis: Workspace.

**L14. The chat route reads its JSON body under axum's default size limit.** `runvil-gateway`, `crates/gateway/app/src/error/wire.rs:27-35`: `WireJson` calls `Json::<T>::from_request(request, state)`. A search of the repository's Rust for `DefaultBodyLimit` and `RequestBodyLimitLayer` matches one line, in the speech batch route (`crates/gateway/stt/api/src/batch.rs:149`). `Cargo.lock` pins `axum` 0.8.9 and `axum-core` 0.5.6, whose source says (`axum-core-0.5.6/src/extract/default_body_limit.rs:128`) the default limit for `Bytes` and "extractors built on top of it such as `String`, [`Json`], and [`Form`]) is 2MB", with `const DEFAULT_LIMIT: usize = 2_097_152; // 2 mb` at `src/ext_traits/request.rs:319`. Inference: a chat request over 2 MB, for example one with inline base64 images, is refused before the handler runs. Confidence: medium (read from the dependency, not run). Source: L-s39, L-s47. Basis: Workspace.

### Axis 2: the message type

**L15. The wire message has four fields, and its content is a JSON value.** `runvil-harness`, `crates/runvil-internal/model-client/src/client/wire.rs:27-47`:

```rust
#[derive(Debug, Clone, PartialEq, Eq, serde::Serialize)]
#[non_exhaustive]
pub struct Message {
    /// Who speaks the message.
    pub(crate) role: Role,
    /// The message content, serialized into the request verbatim: a JSON
    /// string for a plain text message (every inherent constructor), or an
    /// OpenAI content-parts array for a multimodal message built through
    /// [`crate::detail::message_from_validated_parts`].
    pub(crate) content: Value,
    /// For a `tool` message, the id of the tool call this result answers.
    #[serde(skip_serializing_if = "Option::is_none")]
    pub(crate) tool_call_id: Option<String>,
```

The fourth field follows at `:45-46`: `#[serde(skip_serializing_if = "Option::is_none")]` and `pub(crate) tool_calls: Option<Vec<Value>>,`. Its doc comment (`:40-44`) says the projection "re-renders each call from its neutral `ToolCallRecord` into the OpenAI function-call shape, so key order and whitespace can differ from the provider's original." `Role` is `System`, `User`, `Assistant`, `Tool` (`client/role.rs:13-25`). Source: L-s12. Basis: Workspace.

**L16. The list's record has four fields, and content is text or parts.** `runvil-harness`, `crates/runvil-internal/lua/src/protocol/record.rs:86-99`:

```rust
pub struct MessageRecord {
    /// The message role, with a tool result's call ID.
    pub role: RecordRole,
    /// The visible content.
    pub content: MessageContent,
    /// The normalized tool calls the record holds; the record parse refuses
    /// them on every role but assistant.
    pub tool_calls: Vec<ToolCallRecord>,
    /// Metadata the projection never sends; `None` when absent or JSON null.
    pub meta: Option<serde_json::Value>,
}
```

`MessageContent` is `Text(String)` or `Parts(NonEmpty<ContentPart>)` (`:22-30`). `ContentPart` is `Text(String)` or `ImageUrl(String)` (`:13-20`). `ToolCallRecord` is `id`, `name`, `arguments` (`:32-45`). `RecordRole::Tool` holds `tool_call_id` (`:47-62`). Source: L-s19. Basis: Workspace.

**L17. The state that Lua cannot read is on the list, beside the records.** `runvil-harness`, `crates/runvil-internal/lua/src/messages.rs:83-94`:

```rust
struct ListState {
    /// The records, in order, shared with the views read from them.
    records: Vec<Arc<MessageRecord>>,
    /// The last issued send: its round, model name, and wire request.
    last_send: Option<(RoundId, String, Vec<Message>)>,
    /// Attached at the first successful build and never detached.
    rendered: Option<Arc<RenderedTools>>,
    /// Set while a compactor runs over the list.
    compacting: bool,
}
```

The design intent for hidden per-reply data is recorded in `runvil-harness`, `decisions/engine.md` (section 5, "The rules of `models.loop` run in Rust"): "A model's reply reaches the step function as a value that is opaque to the trampoline, which passes it along, and the step appends the records itself, so the Engine can keep data in the list that is hidden from Lua." and "The owner expects to need this for requests to OpenAI endpoints and to Anthropic endpoints, which is planned work, and says of what the Engine then does with the list: \"Lua can't see it.\"" At HEAD every field of `MessageRecord` (L16) is one of the five Lua reads (L18). Source: L-s17, L-s27. Basis: Workspace.

**L18. Lua reads five fields through a read-only view.** `runvil-harness`, `crates/runvil-internal/lua/src/messages.rs:263-264`: `const FIELDS: [&str; 5] = ["role", "content", "tool_calls", "tool_call_id", "meta"];`. A write to a view raises "records read from messages are read-only; rewrite history with models.compact(compactor, reason)" (`:606-615`). The builders are `system`, `user`, `assistant`, `tool`, `append`, and `replace` (`:496-564`). `assistant` takes `(content, tool_calls, meta)` and `tool` takes `(content, tool_call_id, meta)` (`:506-521`). Source: L-s17. Basis: Workspace.

**L19. The builder drops any other field and caps `meta` at 8192 bytes.** `runvil-harness`, `crates/runvil-internal/lua/src/protocol/record.rs:10-11`: `const MAX_RECORD_META_BYTES: usize = 8 * 1024;`. The doc on `parse_record` (`:180-183`): "a `meta` of any JSON value, none when null, at most [`MAX_RECORD_META_BYTES`] as compact JSON. Fields beyond the five a record holds (`role`, `content`, `tool_call_id`, `tool_calls`, `meta`) are dropped." The check is `let size = meta.to_string().len(); if size > MAX_RECORD_META_BYTES` (`:270-271`). Source: L-s19. Basis: Workspace.

**L20. The records the loop appends hold plain text and no `meta`.** `runvil-harness`, `crates/runvil-internal/lua/src/models_loop/machine.rs:507-515`:

```rust
fn text_record(role: RecordRole, content: String) -> MessageRecord {
    MessageRecord {
        role,
        content: MessageContent::Text(content),
        tool_calls: Vec::new(),
        meta: None,
    }
}
```

A text reply is `text_record(RecordRole::Assistant, reply)` (`:316`). A tool round is `text_record(RecordRole::Assistant, String::new())` with `tool_calls` filled from each call's `id`, `name`, `arguments`, followed by one `RecordRole::Tool { tool_call_id: call.id }` record per result (`:409-430`). Source: L-s21. Basis: Workspace.

**L21. The projection sends exactly four fields.** `runvil-harness`, `crates/runvil-internal/lua/src/projection.rs:23-25`: "Metadata is stripped: the wire message holds exactly `role`, `content`, `tool_call_id`, and `tool_calls`, so a record's `meta` (a copied credential, say) can never reach the provider." `wire_message` (`:268-316`) writes each tool call as `{"id": call.id, "type": "function", "function": {"name": call.name, "arguments": call.arguments.to_string()}}` and ends in `message_from_validated_parts(record.role.kind(), content, record.role.tool_call_id().map(str::to_owned), tool_calls)`. The doc at `:30-32`: "The projection is recomputed per dispatch and is the same for every model." Source: L-s18. Basis: Workspace.

**L22. The projection merges four kinds of neighbors.** Same file, `:157-205`: several leading system records become one system record whose text is the records' texts joined with `"\n\n"` (`:163-170`). A record with the same role as the one before it, both without tool calls, role user or assistant, merges into it with the separator `"\n\n"` for user and `""` for assistant (`:180-190`). A text-only assistant record followed by an assistant record merges with it and takes its tool calls: `last.content = merge_content(&last.content, &record.content, ""); last.tool_calls.clone_from(&record.tool_calls);` (`:194-200`). Source: L-s18. Basis: Workspace.

**L23. The projection refuses four list shapes before each request.** Same file, `:59-128`, with these messages: "messages[{index}] is a system message outside the leading system block", "messages[{index}] tool call id {:?} duplicates an earlier tool call", "messages[{index}] is an orphan tool record: no pending assistant tool call with id {id:?}", and "messages[{index}] tool call {id:?} has no tool result". Source: L-s18. Basis: Workspace.

**L24. The Gateway keeps message, choice, and delta payloads as untyped JSON in both directions.** `runvil-gateway`, `crates/gateway/protocol/src/wire.rs:38-39` (`pub messages: Vec<Value>`), `:134-135` (`pub choices: Vec<Value>` with "passed through from the backend verbatim"), and `:237-239`: "The incremental payload, kept as opaque JSON so every field the gateway does not route passes through untouched." `pub delta: Value,`. Source: L-s30. Basis: Workspace.

### Axis 3: reasoning

**L25. The client reads streamed reasoning from three delta keys into one buffer and hands each fragment to the caller's callback.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:247-254`:

```rust
        for key in ["reasoning_content", "reasoning", "thinking"] {
            if let Some(text) = append_string_fragment(delta, key, &mut self.reasoning, key)?
                && !text.is_empty()
            {
                held = true;
                on_delta(StreamDelta::Reasoning(text));
            }
        }
```

`append_string_fragment` (`:438-454`) accepts a string or null and fails any other type as "stream delta `{label}` was present but not a string". `StreamDelta` has two variants, `Text(String)` and `Reasoning(String)` (`wire/delta.rs:12-20`). An array or object under one of the three keys therefore fails the round, and a field with another name (for example `reasoning_details`) is not read. Source: L-s2, L-s6. Basis: Workspace.

**L26. The finished reply keeps the first non-blank of the same three keys.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:233-258`: "First nonblank string among the known reasoning field synonyms." with `for key in ["reasoning_content", "reasoning", "thinking"]`. The stream's reassembled body writes the buffer back under one name: `message.insert("reasoning_content".to_owned(), Value::String(reasoning));` (`wire/stream.rs:395-397`). Source: L-s3, L-s2. Basis: Workspace.

**L27. Reasoning never becomes the answer, and a reply with reasoning alone is an empty reply.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:10-11`: "Reasoning fields are a side channel only and are never promoted into the answer." At `:158`: `let mut error = make_empty_reply_error(reasoning_content.is_some());`. `crates/runvil-internal/model-client/src/model/error.rs:16`: `const REASONING_IGNORED: &str = "reasoning content was present but ignored";`. The client's test `empty_content_with_reasoning_is_error` (`parse.rs:694-717`) pins the text "the model replied with no text and no tool calls: reasoning content was present but ignored". Source: L-s3, L-s14. Basis: Workspace.

**L28. The Engine reports reasoning as one event and then lets the completion go.** `runvil-harness`, `crates/runvil-internal/runtime/src/execute/turn.rs:60-63` and `:96-98`:

```rust
    let thinking = completion
        .reasoning_content()
        .filter(|text| !text.is_empty())
        .map(str::to_owned);
```

```rust
    if let Some(thinking) = &thinking {
        emitter.thinking(section, turn, round, &model, thinking);
    }
```

At `:104`: `let outcome = detail::completion_into_result(completion);`. The event is `Thinking { turn: u32, round: RoundId, model: String, text: String }` (`crates/runvil-internal/types/src/event.rs:312-322`, "One completed block of model thinking."). With debug capture on, the `Response` event also holds `reasoning_content: Option<String>` (`event.rs:402-412`). `Completion.reasoning_content` is `Option<String>` (`model-client/src/client/wire.rs:347-348`). Source: L-s25, L-s24, L-s12. Basis: Workspace.

**L29. Nothing from a reply's reasoning goes into a later request.** The request is built from `Message` values (L15), whose keys are `role`, `content`, `tool_call_id`, `tool_calls`, and the loop builds each assistant record from the reply text or from call ids, names, and arguments (L20). The precheck states the same rule: `runvil-harness`, `crates/runvil-internal/lua/src/compactors.rs:183-190`: "The count is the prompt plus the completion tokens, less any reported reasoning tokens: the history never resends reasoning, so counting it would overflow early." and `let tokens = total.saturating_sub(u64::from(usage.reasoning_tokens.unwrap_or(0)));`. Source: L-s22. Basis: Workspace.

**L30. The thinking switch has three modes on a model and one optional bool on a request.** Engine: `runvil-harness`, `crates/runvil-internal/types/src/models.rs:96-107`: `pub enum ThinkingMode { Never, Always, Switchable }` with "The client may turn thinking on or off per request." Gateway: `runvil-gateway`, `crates/gateway-api-types/src/metadata.rs:14-31`: "`never` and `always` mean the backend ignores a per-call switch; `switchable` means the client may emit `chat_template_kwargs.enable_thinking`." `runvil-harness`, `decisions/models-and-conversations.md` section 1: "`thinking` is a requirement on a role's model. The owner's reason: thinking \"is a property of a model not a model role\"." Source: L-s24, L-s41, L-s27. Basis: Workspace.

**L31. The effort keys have four readers: the config validator, the catalog route, the OpenAPI marker list, and the config UI.** `runvil-gateway`, `crates/gateway-api-types/src/metadata.rs:112-126` defines `effort_levels: Vec<String>` ("The reasoning-effort levels the model accepts. Empty means the model has no effort knob."), `default_effort: Option<String>`, and `adaptive_thinking: bool`. A search of the Gateway's non-test Rust for `effort_levels` and `default_effort` finds: the type and its accessors (`metadata.rs:116-177`), the sheet type (`gateway-api-types/src/lib.rs:238-240`), the config validator (`crates/gateway/config/src/config/validate.rs:208-232`, with "default_effort requires a non-empty effort_levels" and "is not in effort_levels", and `:280-309`, chat-only), and an OpenAPI marker path (`crates/gateway/app/src/openapi/markers.rs:136`). The config UI reads and writes them (`config-ui/ui/src/components/settings-registry.ts:131-150`, `services/cloud-merge.ts:89-96`). The catalog route publishes them (L86). In the Harness, the catalog reader's struct lists `id`, `description`, `context`, `thinking`, `provider` (L89), so the effort keys are ignored on read, and the search in L5 found no effort name anywhere in the Harness. Source: L-s41, L-s40, L-s45, searches run 2026-11-22. Basis: Workspace.

**L32. The Gateway forwards the thinking switch as written, and a translation of it is planned work.** `runvil-design`, `decisions/gateway.md` section 10 (2026-11-10): "**The thinking switch for remote providers.** A per-model setting that translates `chat_template_kwargs.enable_thinking` for a provider that ignores the field. The Gateway forwards the field as the client wrote it." The code agrees: `chat_template_kwargs` is an unnamed field in `ChatRequest.rest` (L11). Source: L-s60, L-s30. Basis: Workspace.

### Axis 4: text beside tool calls, block order, and empty replies

**L33. The normalizer returns tool calls before it looks at the text.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:128-135`:

```rust
    if let Some(raw_calls) = tool_calls.filter(|calls| !calls.is_empty()) {
        let calls = parse_openai_tool_calls(raw_calls)?;
        return Ok(NormalizedTurn {
            outcome: CompletionResult::ToolCalls(calls),
            finish_reason,
            reasoning_content,
        });
    }
```

`content` is first read at `:139`. The result type has two variants, `Text(String)` and `ToolCalls(Vec<ToolCall>)` (`model-client/src/client/wire.rs:281-288`), so one reply is text or calls. Source: L-s3, L-s12. Basis: Workspace.

**L34. The accumulator holds both the text and the calls up to that point.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:385-411`: `into_body` writes `content` (the accumulated text, or null) and `tool_calls` into one reassembled `message`. That reassembled body is what the debug capture stores: `Ok(completion.with_raw(RawExchange::new(request_body, response_body)))` (`:379`). Live text fragments also reach the Host's callback as `StreamDelta::Text` before the turn is judged (`:241-246`). Source: L-s2. Basis: Workspace.

**L35. Only the first choice is read.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:116-117`: "Only the first choice (`index == 0`) is accumulated, mirroring the buffered normalizer, which reads `choices[0]` alone." and `:217-220`: `if index != 0 { return Ok(false); }`. Source: L-s2. Basis: Workspace.

**L36. The accumulator's shape has one slot per kind, so block order is not represented.** Same file, `:126-136`:

```rust
pub(super) struct StreamAccumulator {
    /// Answer text; `None` until the first `content` fragment arrives.
    content: Option<String>,
    /// Reasoning side-channel text; `None` until the first fragment.
    reasoning: Option<String>,
    tool_calls: BTreeMap<u64, ToolCallParts>,
    finish_reason: Option<String>,
    model: Option<String>,
    /// Raw top-level metadata sections, latest occurrence wins.
    sections: Map<String, Value>,
}
```

Inference: a reply that interleaves thinking, text, and tool calls arrives at the Engine as one text string, one reasoning string, and calls sorted by `index`. Confidence: high (the struct is the whole state). Source: L-s2. Basis: Workspace.

**L37. An empty reply is blank text plus zero calls, and the loop decides what it means.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:148-164`: "Whitespace-only content is not a product; classify with `trim`, but preserve the original nonblank payload verbatim." then `make_empty_reply_error(...)` with the `ErrorFact::FinishReason` fact. The loop's exit rule, `crates/runvil-internal/lua/src/models_loop/machine.rs:322-325`:

```rust
        if round.finish_reason.as_deref() == Some("stop") && self.answered > 0 {
            let record = text_record(RecordRole::Assistant, String::new());
            return self.push(vec![record], Self::check);
        }
```

`decisions/models-and-conversations.md` section 4: "A reply with only reasoning text is empty because the client refuses to promote reasoning into the answer." Source: L-s3, L-s21, L-s27. Basis: Workspace.

**L38. The loop branches on the outcome variant, not on the finish reason.** `runvil-harness`, `crates/runvil-internal/lua/src/protocol/answer.rs:64-68`: "Callers branch on the variant, never on the round's finish reason alone - backends routinely finish tool-call rounds with `stop`." The variants are `Overflow(OverflowReason)`, `Reply(String)`, `ToolCalls(Vec<ToolCallEvent>)`, `Empty(Option<String>)` (`:69-87`). Source: L-s20. Basis: Workspace.

**L39. A `models.infer` round that answers with tool calls fails.** `runvil-harness`, `crates/runvil-internal/runtime/src/execute/infer.rs:47-53`: `CompletionResult::ToolCalls(_) => Err(make_malformed_reply_error("the round sent tool calls, but no tools were advertised"))`. Source: L-s25. Basis: Workspace.

### Axis 5: tools

**L40. Tools go out in the OpenAI function shape with `tool_choice` fixed to `"auto"`.** `runvil-harness`, `crates/harness-gateway-client/src/wire/request.rs:44-60`: each tool is `{"type": "function", "function": {"name": tool.name(), "description": tool.description(), "parameters": tool.parameters()}}`, then `body["tool_choice"] = Value::String("auto".into());`. `decisions/models-and-conversations.md` section 7: "A model with another tool format is the Gateway's concern." Source: L-s1, L-s27. Basis: Workspace.

**L41. A tool's wire name and schema are checked when the schema is built.** `runvil-harness`, `crates/runvil-internal/model-client/src/detail.rs:59-64`: "Returns a [`ToolSchemaInvalid`](ErrorCode::ToolSchemaInvalid) error with the reason `invalid_name` when `name` does not match `[A-Za-z][A-Za-z0-9_-]{0,63}`, and `non_object_schema` when `parameters` is not a JSON object". Source: L-s16. Basis: Workspace.

**L42. A reply's tool call must be exact OpenAI shape.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:167-174`:

```rust
/// Parses the OpenAI `message.tool_calls` array into [`ToolCall`]s.
///
/// Each call must be an object with `"type": "function"`, a string `id`,
/// and an object `function` containing a string `name` and an `arguments`
/// field that is present, a JSON-encoded string, and valid JSON. Missing or
/// null arguments are rejected rather than coerced. [`ToolCall::from_parts`]
/// then refuses a blank id, a blank name, and arguments that do not decode
/// to an object.
```

Failure texts include `tool call had no string id` (`:194`), `tool call arguments were missing` (`:220`), and `tool call arguments were not a JSON-encoded string` (`:224`). A `type` other than the string `function` fails at `:184-190`. Source: L-s3. Basis: Workspace.

**L43. Streamed fragments merge by `index`, and `id`, `name`, and `arguments` each grow by concatenation.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:104-112`: "One tool call assembled from streamed fragments, keyed by the fragment `index`. `id`, `name`, and `arguments` each grow by string concatenation as fragments arrive, per the `OpenAI` streaming contract." A fragment without an integer `index` fails as "stream tool-call fragment had no integer index" (`:276-279`). Inference: an upstream that repeats the whole `id` or `name` on every fragment would produce a doubled id or name. Confidence: medium (follows from `parts.id.push_str(id)` at `:284`, not run against such an upstream). Source: L-s2. Basis: Workspace.

**L44. The Engine's constructors run the checks that are independent of the wire.** `runvil-harness`, `crates/runvil-internal/model-client/src/normalize.rs:23-62`: "tool call id was blank", "tool call name was blank", "tool call arguments were not a JSON object", and "duplicate tool call id {id:?} within one turn". `Completion::from_result` (`client/wire.rs:380-392`) returns `EmptyReply` for an empty batch and runs `check_unique_call_id` over the batch. Source: L-s15, L-s12. Basis: Workspace.

**L45. A batch cut off by the output limit or a content filter fails the round.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:341-355`: "The truncation rule runs before normalization: a tool-call batch cut short by `length` or `content_filter` may hold partial JSON arguments, and partial arguments must not execute." with `matches!(self.finish_reason.as_deref(), Some("length" | "content_filter"))` and the message "tool-call batch truncated by finish_reason {reason:?}: partial arguments must not execute". Source: L-s2. Basis: Workspace.

**L46. Call ids are stored as the model gave them and must be unique across the whole conversation.** `runvil-harness`, `crates/runvil-internal/lua/src/projection.rs:72-74`: "Every call ID ever seen: uniqueness holds across the whole list, not only within one batch." The projection's test (`projection-tests.rs:408-409`) asserts "messages[3] tool call id \"call_1\" duplicates an earlier tool call" with the note "ids stay unique across the whole list, not only within one batch". The event type says the opposite about providers: `crates/runvil-internal/types/src/metrics.rs:24-26`: "The provider-issued tool-call id. Providers recycle ids like `call_1` across rounds, so consumers scope the id by turn." The id is copied without change from the completion into the record (`machine.rs:414-419`). Inference: on an upstream that reuses an id in a later round, the next request's projection fails with the duplicate message. The Gateway's own emulated dialect is such a source: it mints `id: format!("call_tool_code_{index}")` (`runvil-gateway`, `crates/gateway/app/src/dialect.rs:253`) from a counter that starts at 0 for each reply (`dialect/content.rs:18-20`: `let mut next_id = 0usize;` inside `parse_content_tool_dialect`). Confidence: medium. The two rules were read side by side and no test was run. Source: L-s18, L-s24, L-s21, L-s38. Basis: Workspace.

**L47. Tool-call arguments are replayed as a re-serialized object, not as the model's original string.** The client parses `function.arguments` into a JSON value (`parse.rs:211-227`), the record stores the object (`record.rs:41-44`: "The call arguments; always an object, normalized to `{}` when the record had none."), and the projection writes `"arguments": call.arguments.to_string()` (`projection.rs:303`). The `Message` doc (`client/wire.rs:40-44`) says: "so key order and whitespace can differ from the provider's original." A test comment in `crates/runvil-internal/types/src/metrics.rs:493-496` states the key order: "`Value::Object` is a `BTreeMap`, so serialization emits keys in sorted order no matter the order they were inserted. No crate enables `serde_json`'s `preserve_order`". Source: L-s3, L-s19, L-s18, L-s12, L-s24. Basis: Workspace.

**L48. Several calls in one reply run as one batch, and whether a model may call in parallel is left to the upstream's default.** `runvil-harness`, `crates/runvil-internal/lua/src/models_loop/machine.rs:8-13`: "The batch rule: every call a round left unanswered goes out as one `tool_calls` request, even a lone one, and a call goes out alone as a `tool_call` request only when the scheduler's answer stops short of it." and "The round's records append only once every call has its result, in the model's order." The request body's key set (L3, L4) is what reaches the upstream. On the Gateway, `parallel_tool_calls: bool` is a capability key ("Whether the model can emit parallel tool calls. Defaults to false.", `gateway-api-types/src/metadata.rs:108-111`) that the catalog publishes (L86). Source: L-s21, L-s41. Basis: Workspace.

**L49. Structured output exists today as one flag in the provider sheet.** The search in L5 returned zero matches for `response_format` in the Harness. In the Gateway's Rust, `response_format` appears only in the speech route's types (`crates/gateway/protocol/src/wire/speech.rs:85`). The provider sheet has `structured_outputs: bool` ("Honors response schemas.", `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/sheet-types.rs:225-226`), and the flag stays there: the config's `Capabilities` struct is the nine keys in L87, and the Cloud page's merge copies the fields listed in L93. A caller-written `response_format` would pass through the Gateway inside `rest` (L11). Source: L-s50, L-s41, searches run 2026-11-22. Basis: Workspace.

### Axis 6: streaming

**L50. The client's scanner keeps `data:` lines and skips everything else.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:34-37`: "Splits a raw SSE byte stream into `data:` payloads. Blank lines, `:` comments, and non-`data:` fields (`event:`, `id:`, `retry:`) are skipped; the caller sees only payload text." The code at `:79-85`: `if line.is_empty() || line.starts_with(':') { continue; }`, then `let Some(data) = line.strip_prefix("data:") else { continue; };`, then `return Some(data.trim_start().to_owned());`. Each `data:` line is one payload, so a JSON payload split over several `data:` lines is read as separate payloads. Source: L-s2. Basis: Workspace.

**L51. The client requires the `[DONE]` sentinel and caps the stream's size.** `runvil-harness`, `crates/harness-gateway-client/src/wire/read.rs:166-172`: "A stream that ends without the sentinel was cut off; its accumulation may be missing the tail, so it must never pass for a complete turn." with the error "completion stream ended without the [DONE] sentinel". At `:143-148`: `if received > max_bytes` fails with "response stream exceeds the {max_bytes}-byte limit". Reading stops at the sentinel (`break 'read;`, `:152-155`). Source: L-s4. Basis: Workspace.

**L52. Per chunk, the client hard-fails on bad JSON, fails on a top-level `error`, and treats absent or empty `choices` as a summary chunk.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:162-198`: invalid JSON is "stream chunk was not valid JSON" (`:162-164`). Then:

```rust
        if let Some(envelope) = chunk.get("error").filter(|error| !error.is_null()) {
            let message = envelope
                .get("message")
                .and_then(Value::as_str)
                .unwrap_or("stream error envelope omitted its message");
            return Err(make_classify_stream_error(&escape_controls(message, 2000)));
        }
```

Then `for key in ["usage", "timings", "metrics"]` stores each non-null section (`:181-187`), and "Absent or empty `choices` is the summary-chunk shape (`stream_options.include_usage`): metadata only, nothing to index." (`:188-198`). The module doc (`:18-20`) says the loop "hard-fails on the first malformed chunk, because a completion's product must be whole." Source: L-s2. Basis: Workspace.

**L53. A chunk field the accumulator does not name ends at the client, including in the debug capture.** The accumulator's whole state is the struct in L36. `into_body` (`stream.rs:385-430`) builds a new body with `role`, `content`, `reasoning_content`, `tool_calls`, `index`, `finish_reason`, `model`, and the three sections. `RawExchange.response` is documented as "The response as the backend returned it, reassembled into the buffered chat-completion shape when it streamed." (`model-client/src/client/wire.rs:304-306`). Inference: fields such as a chunk `id`, a provider name, a second finish-reason field, a structured reasoning array, or message annotations are parsed as JSON and then not kept anywhere. Confidence: high. Source: L-s2, L-s12. Basis: Workspace.

**L54. The client's timer covers each receive, not the whole reply.** `runvil-harness`, `crates/harness-gateway-client/src/transport.rs:47-50`: `const DEFAULT_RECEIVE_TIMEOUT: Duration = Duration::from_secs(120);` and `const DEFAULT_MAX_RESPONSE_BYTES: u64 = 16 * 1024 * 1024;`. At `:267-270`: "No reqwest `.timeout`: it caps the whole request including the body, which would cut off a long stream that is still arriving. The timeout bounds the headers here and each receive in `ResponseChunks`." Each chunk read is `tokio::time::timeout(self.timeout, self.response.chunk())` (`:68-73`). The timer restarts on any bytes received, so a line the scanner later skips (a comment) also restarts it. `with_request_limits` (`:164-172`) changes both values, and "the standard broker uses the defaults" (`decisions/models-and-conversations.md` section 7). Source: L-s7, L-s27. Basis: Workspace.

**L55. The Gateway's chunk type requires `model` and `choices`, and each choice requires `index` and `delta`.** `runvil-gateway`, `crates/gateway/protocol/src/wire.rs:198-208` and `:233-244`:

```rust
pub struct ChatChunk {
    /// The model name, rewritten to the caller's requested name.
    pub model: String,
    /// The partial choices for this chunk.
    pub choices: Vec<ChatChunkChoice>,
    /// Every field the gateway does not name (for example `usage` on a
    /// final chunk), preserved verbatim.
    #[serde(flatten)]
    pub rest: Map<String, Value>,
}
```

```rust
pub struct ChatChunkChoice {
    /// The completion choice this delta belongs to.
    pub index: u32,
    /// The incremental payload, kept as opaque JSON so every field the
    /// gateway does not route passes through untouched.
    pub delta: Value,
    /// Every field the gateway does not name (for example `finish_reason`
    /// on the terminal chunk), preserved verbatim.
    #[serde(flatten)]
    pub rest: Map<String, Value>,
}
```

`openapi.json` lists the same required sets: `ChatChunk` requires `model` and `choices`, `ChatChunkChoice` requires `index` and `delta`, and both mark `additionalProperties` with `x-runvil-pass-through: true`. Source: L-s30, L-s42. Basis: Workspace.

**L56. The Gateway's stream parser skips a chunk it cannot use and treats the end of the upstream body as the end of the stream.** `runvil-gateway`, `crates/gateway/protocol/src/upstream.rs:215-228` (doc): "Each `data:` line holds one JSON chunk; blank lines, comments, and the `event:`/`id:`/`retry:` fields are skipped, and the terminal `[DONE]` sentinel - which is not JSON - ends the stream without being yielded and without ever reaching the malformed-chunk log. Every chunk's model is rewritten to `requested` (the caller's model name, never the backend's). A chunk that is undecodable or fails the minimal shape check is logged and skipped, so one bad chunk never ends an otherwise healthy stream." The code at `:263-278`:

```rust
                    if data == "[DONE]" {
                        return None;
                    }
                    let mut chunk = match serde_json::from_str::<ChatChunk>(data) {
                        Ok(chunk) => chunk,
                        Err(error) => {
                            tracing::warn!(%error, "skipping undecodable upstream chunk");
                            continue;
                        }
                    };
                    if let Err(reason) = chunk.validate() {
                        tracing::warn!(%reason, "skipping malformed upstream chunk");
                        continue;
                    }
                    chunk.model.clone_from(&requested);
                    return Some((Ok(chunk), (bytes, buffer, requested, false)));
```

When the byte stream ends, `None => return None,` (`:288`) ends the chunk stream the same way `[DONE]` does. A read error becomes one `Err(ProtocolError::upstream_transport(error))` item (`:282-287`). Source: L-s31. Basis: Workspace.

**L57. A chunk with empty `choices` is skipped by design, and the doc names the usage chunk as the example.** `runvil-gateway`, `crates/gateway/protocol/src/wire.rs:213-218`: "A chunk must have at least one choice; each choice's `index` and `delta` are required typed fields, so deserialization has already proven them present. A chunk that fails this check (for example a usage-only summary object a backend appends mid-stream) is malformed: the parser logs and skips it rather than relaying it or ending the stream." The check is `if self.choices.is_empty() { return Err("upstream chunk has no choices"); }` (`:222-227`). The test `stream_malformed_chunks_are_logged_and_skipped` (`crates/gateway/protocol/src/upstream/tests/stream.rs:99-131`) feeds `data: not json` and `data: {"model":"m","choices":[]}` and asserts that only the two good chunks arrive and that "each malformed chunk is logged once". Source: L-s30, L-s44. Basis: Workspace.

**L58. Inference: which upstream chunk fields survive the Gateway's parse and re-serialize step.** From L55 to L57:

- A field inside `delta` (for example `reasoning_details`, `reasoning`, `reasoning_content`, `annotations`) survives, because `delta` is an untyped value.
- A field on the choice beside `index` and `delta` (for example `finish_reason`, `native_finish_reason`, `logprobs`) survives in the choice's `rest`.
- A top-level field beside `model` and `choices` (for example `id`, `created`, `usage`, `provider`, `system_fingerprint`, or a top-level `error` on a chunk that also has choices) survives in the chunk's `rest`.
- All of this holds only for a chunk that has a string `model` and at least one choice with `index` and `delta`. A chunk with `choices: []` (the OpenAI-shaped usage chunk), a chunk without `model`, a choice without `delta`, and a bare `{"error": {...}}` object are each logged and dropped.
- `model` is always replaced by the caller's model name.

Confidence: high on the type reasoning (a `String` or `Vec` field without a `#[serde(default)]` attribute is required, and these two fields are plain). The repository's tests cover plain content chunks, `not json`, and empty `choices`. No test feeds a chunk with `reasoning_details`, a usage chunk with a choice, or an error object. Source: L-s30, L-s31, L-s44. Basis: Workspace.

**L59. The relay writes each chunk back as JSON, turns a mid-stream failure into an envelope event, and always ends with `[DONE]`.** `runvil-gateway`, `crates/gateway/app/src/relay.rs:199-204`: "The relay is typed: each upstream chunk is validated and re-serialized per chunk rather than splicing upstream bytes through. A mid-stream failure is emitted as an error-envelope `data:` event before the stream ends, and a clean end is marked with the `data: [DONE]` sentinel. The response forwards the upstream `Content-Type`/`Cache-Control` when present, defaulting to `text/event-stream`/`no-cache`." The code: `Ok(chunk) => match serde_json::to_string(&chunk) { Ok(json) => (format!("data: {json}\n\n"), false), ...` (`:220-221`), `Err(error) => (format!("data: {}\n\n", error.envelope()), true),` (`:230`), and `let done = futures_util::stream::once(async { Ok("data: [DONE]\n\n".to_owned()) });` chained after the relayed stream (`:238-239`). So `[DONE]` follows an error event, and it follows an upstream body that ended without its own sentinel. Source: L-s35. Basis: Workspace.

**L60. The Gateway's stream has a connect timeout and no other limit.** `runvil-gateway`, `crates/gateway/protocol/src/http_util.rs:16-20`: `const CONNECT_TIMEOUT: Duration = Duration::from_secs(10);` and `const REQUEST_TIMEOUT: Duration = Duration::from_secs(120);`. At `:32-37`: "The streaming path therefore bounds only the connect; once the stream is open, chunk flow is the liveness signal." `runvil-design`, `decisions/gateway.md` section 10 lists the planned item: "**Timeouts.** A configurable limit for plain requests and an idle deadline for streams. A plain request has a fixed limit of 120 seconds, and a stream has only the 10 second connect timeout." Source: L-s34, L-s60. Basis: Workspace.

**L61. Inference: upstream keep-alive lines end at the Gateway, and the caller's stream is made of chunk events alone.** The parser skips comment lines (`upstream.rs:256-258`), and the relay's output is one event per parsed chunk, one error event, and the sentinel (L59). So while an upstream sends only comments, or sends chunks the Gateway drops, the client receives no bytes, and its 120 second receive timer (L54) runs out as `Timeout`. Confidence: high on the control flow, not run. Source: L-s31, L-s35, L-s7. Basis: Workspace.

**L62. Two upstream response headers reach the caller on a stream, and a plain reply is rebuilt as JSON.** `runvil-gateway`, `crates/gateway/protocol/src/upstream.rs:22-34`: `StreamedChunks` holds `content_type: Option<String>` ("The upstream `Content-Type`, forwarded when present; the relay defaults to `text/event-stream` otherwise."), `cache_control: Option<String>`, and `chunks`. The non-stream path ends in `Ok(Json(response).into_response())` (`relay.rs:193`). Rate-limit headers, request ids, and `retry-after` from the upstream stay at the Gateway. Source: L-s31, L-s35. Basis: Workspace.

**L63. Local models go through the same stream parser.** `runvil-gateway`, `crates/gateway/local/src/upstream.rs:323-334`: `LocalUpstream::forward_stream` ends in `Ok(gateway_protocol::upstream::sse_chunks(response, requested))`. Source: L-s46. Basis: Workspace.

**L64. Only the Host's broker sees live text.** `runvil-harness`, `crates/harness-internal/runner/src/performers.rs:60-68`: "Runs one model round and returns the finished reply." and "The Harness takes only the finished reply. A broker that shows the reply as it forms streams it on its own." `crates/harness-gateway-client/src/broker.rs:87-96`: `chat_streaming(&self, _binding: ModelBinding, messages, tools, options, on_piece: Arc<dyn Fn(StreamDelta) + Send + Sync>)`. Tool-call fragments are held back: "The client buffers tool-call fragments until the whole batch of tool calls is complete and validated. They appear only in the finished completion." (`wire/delta.rs:9-11`). Source: L-s26, L-s8, L-s6. Basis: Workspace.

### Axis 7: usage and cost

**L65. The usage type has three required counts and two optional details.** `runvil-harness`, `crates/runvil-internal/types/src/metrics.rs:139-153`:

```rust
pub struct Usage {
    /// Tokens in the prompt.
    pub prompt_tokens: u32,
    /// Tokens generated in the completion.
    pub completion_tokens: u32,
    /// Prompt plus completion tokens.
    pub total_tokens: u32,
    /// Prompt tokens served from a prefix cache, when the backend reports
    /// the detail.
    #[serde(skip_serializing_if = "Option::is_none")]
    pub cached_tokens: Option<u32>,
    /// Tokens spent on reasoning, when the backend reports the detail.
    #[serde(skip_serializing_if = "Option::is_none")]
    pub reasoning_tokens: Option<u32>,
}
```

Every field is a token count. `CallMetrics` (`:85-98`) groups `usage`, `llama` (llama.cpp `timings`), `vllm` (vLLM `metrics`), and `client` (the client's own clock). Source: L-s24. Basis: Workspace.

**L66. The client reads five usage values from the wire and ignores the other keys.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:353-379`: `struct WireUsage { prompt_tokens: u32, completion_tokens: u32, total_tokens: u32, prompt_tokens_details: Option<WirePromptTokensDetails>, completion_tokens_details: Option<WireCompletionTokensDetails> }`, where the details hold `cached_tokens: Option<u32>` and `reasoning_tokens: Option<u32>`. The structs have plain `#[derive(Deserialize)]`, so unknown keys (a cost, a cache-write count, audio tokens) are skipped. A `usage` object that lacks one of the three counts, or has a non-integer there, fails to parse and becomes absent with the diagnostic "malformed `{key}` in completion response ignored: {error}" (`:333-351`), which the Engine reports as a `model_metadata_degraded` event. Source: L-s3. Basis: Workspace.

**L67. Usage is read from the top level of any chunk, and the latest one wins.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:116-121`: "Metadata sections (`usage`, llama.cpp `timings`, vLLM `metrics`) are kept verbatim from whichever chunk held them last, including the empty-choices summary chunk `stream_options.include_usage` appends, and are handed to the lenient metadata parser unjudged." The request asks for that chunk (`wire/request.rs:10-13`): "That asks the backend to send token usage in a final chunk with an empty `choices` list". Source: L-s2, L-s1. Basis: Workspace.

**L68. Inference: on the pass-through path, usage reaches the client only when the upstream puts it on a chunk that has a choice.** The client asks for a final chunk "with an empty `choices` list" (L67), and the Gateway skips a chunk with empty `choices` (L57). A usage object on a chunk with a choice passes (L58). The emulated dialect is the exception inside the Gateway: `response_chunks` builds its own summary chunk with `choices: Vec::new()` and the `usage`, `timings`, `metrics` of the buffered reply, and hands it straight to the relay (`runvil-gateway`, `crates/gateway/app/src/dialect.rs:159-171`), without the upstream parser's check. Confidence: high on the reading, not run. The 2026-11-17 report states the same drop in its Table 3. Source: L-s30, L-s38, L-s61. Basis: Workspace.

**L69. The Engine uses usage for the precheck and for the reply event.** `runvil-harness`, `crates/runvil-internal/runtime/src/execute/scheduler/chat.rs:286-290`: `let usage = served.metrics.as_ref().and_then(|metrics| metrics.usage.as_ref()); self.chain.anchor.settle(usage);`. `UsageAnchor` (`lua/src/compactors.rs:164-192`) is "the projected messages the round sent, and the tokens the provider counted for them plus the reply." The `AssistantReply` event has `metrics: Option<CallMetrics>` (`types/src/event.rs:336-337`). A round without usage falls back to the estimate of four characters per token plus four tokens per message (`compactors.rs:139-143`, `:254-257`). Source: L-s25, L-s22, L-s24. Basis: Workspace.

**L70. Prices exist in the provider sheet only.** `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/sheet-types.rs:298-307`: `pub struct Pricing { currency: String, prompt_per_mtok: f64, completion_per_mtok: f64 }` ("Token pricing, normalized to per-million-token units."). The OpenRouter file reads two prices and says of the rest (`providers/openrouter.rs:81-88`): "The cache, image, audio, and request price variants are ignored." The config's nine capability keys (L87) and the Engine's five token counts (L65) are the whole of those two types, so a price lives in the sheet and a cost is computed by nobody. Source: L-s50, L-s51. Basis: Workspace.

### Axis 8: finish reasons

**L71. The client stores the finish reason as a free string and compares it once.** `runvil-harness`, `crates/harness-gateway-client/src/wire/stream.rs:221-229`: `Some(Value::String(reason)) => self.finish_reason = Some(reason.clone()),` and any other non-null type is "stream choice `finish_reason` was present but not a string". The one comparison is `Some("length" | "content_filter")` with tool calls present (`:344-348`, L45). Source: L-s2. Basis: Workspace.

**L72. The Engine compares two strings.** `"stop"`: `runvil-harness`, `crates/runvil-internal/lua/src/models_loop/machine.rs:322` (L37). `"length"`: `crates/runvil-internal/runtime/src/execute/turn.rs:105-108`: `if served.finish_reason.as_deref() == Some("length") { emitter.report(section, event_lifecycle::MODEL_TURN_TRUNCATED); }`, on a text reply only. The string is recorded in `ChatAnswerRecord.finish_reason` (`execute/run/effect.rs:698-708`) and `AssistantReply.finish_reason` (`types/src/event.rs:332-333`). Source: L-s21, L-s25, L-s24. Basis: Workspace.

**L73. The Gateway passes the finish reason through and writes one itself in the emulated dialect.** `runvil-gateway`, `crates/gateway/protocol/src/wire.rs:240-241`: "Every field the gateway does not name (for example `finish_reason` on the terminal chunk), preserved verbatim." `crates/gateway/app/src/dialect.rs:99-102`: `choice_object.insert("finish_reason".to_owned(), Value::String("tool_calls".to_owned()));`. Source: L-s30, L-s38. Basis: Workspace.

### Axis 9: errors

**L74. Every Gateway failure is one envelope shape.** `runvil-gateway`, `crates/gateway/protocol/src/error.rs:150-155`:

```rust
    pub fn envelope(&self) -> serde_json::Value {
        let (_, kind, code) = self.classify();
        serde_json::json!({
            "error": { "message": self.to_string(), "type": kind, "code": code }
        })
    }
```

`crates/gateway/protocol/src/wire/envelope.rs:5-6` names it "The body of every JSON error response, in the OpenAI shape", with the three keys `message`, `type`, `code` under `error`. `openapi.json` marks `ErrorEnvelope` and `ErrorDetail` with `additionalProperties: false` and requires `message`, `type`, `code`. Source: L-s33, L-s30, L-s42. Basis: Workspace.

**L75. An upstream 4xx keeps its status with one type and one code, and every upstream 5xx becomes 502.** `runvil-gateway`, `crates/gateway/protocol/src/error.rs:126-138`:

```rust
            ProtocolError::UpstreamStatus { status, .. } => {
                let code = reqwest::StatusCode::from_u16(*status)
                    .unwrap_or(reqwest::StatusCode::BAD_GATEWAY);
                if code.is_client_error() {
                    (code, "invalid_request_error", "upstream_client_error")
                } else {
                    (
                        reqwest::StatusCode::BAD_GATEWAY,
                        "server_error",
                        "upstream_error",
                    )
                }
            }
```

The table test pins a 429 to `(TOO_MANY_REQUESTS, "invalid_request_error", "upstream_client_error")` (`:220-226`) and a 503 envelope to the message "upstream returned 503" (`:259-265`). Source: L-s33. Basis: Workspace.

**L76. The upstream's error body stays in the Gateway.** `runvil-gateway`, `crates/gateway/protocol/src/error.rs:41-48`: `#[error("upstream returned {status}")] UpstreamStatus { status: u16, /// The (truncated) upstream body, for diagnostics. body: String, }`. The envelope's message is `self.to_string()` (L74), so the body is not in it. `crates/gateway/protocol/src/upstream/openai.rs:80-82`: "F5: the raw body may echo prompt content or credentials, so only its OpenAI error envelope reaches the log, bounded and escaped." The log fields are the upstream's `error.code`, `error.type`, and `error.message`, each cut to 512 characters (`:136-172`). Source: L-s33, L-s32. Basis: Workspace.

**L77. The Gateway's own failures have their own codes.** `runvil-gateway`, `crates/gateway/app/src/error/classify.rs:53-112`: `Unauthorized` is 401 `authentication_error` `unauthorized`. `UnknownModel` is 404 `model_not_found`. `KindMismatch` and `MalformedRequest` are 400. `QueueFull` is 503 `server_error` `queue_full`. `QueueRejected` is 429 `rate_limit_error` `queue_rejected`. `ModelProvisioning` and `ModelLoading` are 503. `Protocol(error) => error.classify()` (`:82`). Transport failures are 502 with `upstream_transport`, `upstream_connect`, or `upstream_protocol` (`protocol/src/error.rs:111-125`). `Retry-After` is set for one variant: `matches!(self, GatewayError::ModelLoading(_)).then_some(MODEL_LOADING_RETRY_AFTER_SECONDS)` with the value 5 (`app/src/error.rs:322`, `:336-338`). Source: L-s39, L-s33. Basis: Workspace.

**L78. A finer upstream mapping exists for the speech route alone.** `runvil-gateway`, `crates/gateway/app/src/error.rs:79-90`: `UpstreamRateLimited` ("Maps to 429 so an OpenAI client surfaces a retryable rate-limit error rather than a server failure. Speech-only: every other route keeps the shared [`ProtocolError`] mapping, so their envelopes stay bit-identical.") and `UpstreamUnavailable` ("Maps to 503 so a client can retry, rather than the shared mapping's 502."). Source: L-s39. Basis: Workspace.

**L79. The client classifies an HTTP failure by status and by phrases in the body.** `runvil-harness`, `crates/harness-gateway-client/src/wire/classify.rs:10-19`:

```text
//! 1. A 400 or 413 whose body names a context limit is `ContextOverflow`.
//! 2. A 429 whose body names quota or billing is `QuotaExhausted`; every
//!    other 429 is `RateLimited`.
//! 3. A 503 or 529, or any 5xx whose body names `overloaded`, is
//!    `Overloaded`; every other 5xx is `ServerError`.
//! 4. A 401 or 403 is `Unavailable`.
//! 5. A 400 whose body names a content policy term is `Refused`.
//! 6. Every other status is `Rejected`.
```

The phrase lists (`:26-48`), matched case-insensitively against the whole body text: overflow is "context length", "context window", "context size", "context_length_exceeded", "maximum context length", "prompt is too long", "too many tokens", "exceeds the available context size", "exceed_context_size", "input is too long", "exceeds the maximum number of tokens", "too large for model". Quota is "quota", "billing", "insufficient_quota", "credit". Refusal is "content_filter", "content policy", "safety", "refus". The body is cut to 2000 characters and control-escaped first (`transport.rs:288-296`), and `with_detail` appends it to the error's message and leaves the prose as it was (`failure.rs:59-66`: "The prose stays as it was, so provider text never reaches the model."). Source: L-s5, L-s7, L-s10. Basis: Workspace.

**L80. An in-stream error has only its text to go on.** `runvil-harness`, `crates/harness-gateway-client/src/wire/classify.rs:96-104`: the doc says the envelope has body text and no status, so only the text rules apply, in this order: "Text that names a context limit is `ContextOverflow`. Text that names quota or billing is `QuotaExhausted`. Text that contains `overloaded` is `Overloaded`. Text that names a content policy term is `Refused`. Any other text is `Transport`, because the stream died in flight." The text passed in is `error.message` (L52). The envelope's `type` and `code` are not read there. Source: L-s5, L-s2. Basis: Workspace.

**L81. Inference: for a remote upstream behind the Gateway, the client's phrase rules see only the Gateway's fixed text.** The Gateway's body for an upstream 400 is `{"error":{"message":"upstream returned 400","type":"invalid_request_error","code":"upstream_client_error"}}` (L74 to L76). That text matches zero of the twelve overflow phrases, four quota words, and four refusal words. So through the Gateway:

- an upstream context overflow (400 or 413) is classified `Rejected`, and the compactor's provider path (L82) is not reached,
- an upstream 429 for spent quota is `RateLimited`,
- an upstream 400 content refusal is `Rejected`,
- an upstream 500, 503, or 529 arrives as 502 and is `ServerError`, so `Overloaded` is reached only by the Gateway's own 503s (`queue_full`, `model_loading`, `model_provisioning`),
- an upstream 401 or 403 keeps its status and is `Unavailable` with "the model backend did not accept the credentials".

Confidence: high on the string reasoning, not run. The 2026-11-17 report reached the overflow part of this with medium confidence. Source: L-s33, L-s5, L-s61. Basis: Workspace.

**L82. The Engine treats two codes specially.** `runvil-harness`, `crates/runvil-internal/runtime/src/execute/scheduler/chat.rs:334-366`: `error.code == ErrorCode::ContextOverflow` answers the round with `overflow_result(OverflowReason::Provider, ...)`, `error.code == ErrorCode::EmptyReply` answers with `ChatOutcome::Empty(Some(error.prose.clone()))` and the `FinishReason` fact, and any other error is "a failed turn and the call's error". The model codes are `ContextOverflow = 401` through `InvalidParts = 413` (`types/src/error.rs:327-351`). Source: L-s25, L-s24. Basis: Workspace.

**L83. Each round is one HTTP attempt, and a failure goes to Lua.** `runvil-harness`, `crates/harness-gateway-client/src/transport.rs:254-312`: `complete` sends once and returns the classifier's error or the stream's result. `crates/harness-internal/runner/src/effect_loop.rs:583-587`: the effect's answer is `EffectAnswer::Chat(completion)` with whatever `broker.chat(...)` returned. A search of the client, the runner, and the scheduler's chat arm for "retry", "retries", "retryable", and "backoff" finds only doc text. The fixed prose tells the reader what to do, for example "The model backend is overloaded. Wait, then try again." (`types/src/error.rs:676`). Inside the Gateway, one retry exists for a local model whose child process died (`runvil-gateway`, `crates/gateway/local/src/upstream.rs:368-376`, `RecoveryOutcome::Retry => self.forward(req, upstream_model).await`). Source: L-s7, L-s26, L-s24, L-s46. Basis: Workspace.

**L84. A caught failure in Lua is a table with `kind`, `code`, and `message`.** `runvil-harness`, `crates/runvil-internal/lua/src/error_value.rs:619-620`: `fields.raw_set("kind", kind.tag())?;` and `fields.raw_set("code", code.tag())?;`. At `:362-368`: "Returns the fields a universal error's Lua error table holds beside `kind`, `code`, and `message`, from its facts: `reason` for context exhaustion, `finish_reason` for an empty reply when the backend gave one". The kinds for model rounds are `EmptyModelReply`, `ModelFailure` ("A model round or a catalog fetch failed for any reason other than an empty reply: transport, backend, decode, or configuration."), and `ModelRequired` (`:66-73`). `decisions/models-and-conversations.md` section 7: "The client appends the provider's escaped text to the error's message, and Lua receives the prose." Source: L-s23, L-s27. Basis: Workspace.

**L85. Inference: an upstream's own in-stream error object is dropped by the Gateway, so the client sees a stream that ends normally.** A bare `data: {"error": {...}}` line holds the one key `error`, so `serde_json::from_str::<ChatChunk>` fails on the missing `model` and the line is skipped (L56). If the upstream then closes the body, the Gateway writes `[DONE]` (L59). The client then finishes the turn from what it has: partial text becomes a text reply whose finish reason is unset, and an empty accumulation becomes `EmptyReply`. The client's stream classifier (L80) is reached by the Gateway's own envelope after a transport failure, or by an upstream error chunk that also has `model` and a valid choice. The client's comment names the case it expects: "A mid-stream `error` envelope is how the gateway (and llama.cpp) report a failure after the 200 has already been sent" (`runvil-harness`, `wire/stream.rs:165-168`). Confidence: high on the type reasoning, not run. Source: L-s31, L-s35, L-s2. Basis: Workspace.

### Axis 10: model discovery

**L86. `GET /v1/models` builds each entry from the routing table.** `runvil-gateway`, `crates/gateway/app/src/models.rs:97-107`:

```rust
        .map(|model| {
            CatalogModelInfo::inference(ModelInfo {
                id: model.name.clone(),
                object: "model",
                kind: model.kind,
                description: model.description.clone(),
                context: model.context,
                thinking: model.thinking,
                capabilities: model.capabilities.clone(),
            })
        })
```

`ModelInfo` (`crates/gateway-api-types/src/metadata.rs:195-218`) is "One catalogued model, with Runvil extensions beside the OpenAI `id`." and flattens `capabilities` into the entry. Loaded speech-to-text models are appended as `{id, object: "model", kind: "transcription"}` (`models.rs:52-59`, `:110-116`). The routing table is built from `[[model]]` entries in `gateway.toml` plus running local models (`routing.rs:127-148`). Source: L-s37, L-s41, L-s36. Basis: Workspace.

**L87. Nine capability keys are flattened into each entry.** `runvil-gateway`, `crates/gateway-api-types/src/metadata.rs:91-138`, with these docs:

- `max_output: Option<u32>`: "Max output tokens the model can emit per completion. Must not exceed `context` when set."
- `default_temperature: Option<f32>`: "Sampling temperature applied when the caller omits one."
- `images: bool`: "Whether the model accepts image inputs."
- `parallel_tool_calls: bool`: "Whether the model can emit parallel tool calls. Defaults to false."
- `effort_levels: Vec<String>`: "The reasoning-effort levels the model accepts. Empty means the model has no effort knob."
- `default_effort: Option<String>`: "The effort level applied when the caller omits one; requires a non-empty `effort_levels` and must name a listed level."
- `adaptive_thinking: bool`: "Whether the model adaptively chooses how much to think per request; chat kind only."
- `voices: Vec<String>`: speech only.
- `provider: Option<String>`: "The model's provider id, such as `xai`. It describes the model, not the endpoint the gateway reaches it through".

The struct doc (`:80-87`): "validated at load, and surfaced verbatim on `GET /v1/models` so clients can shape requests before sending them." Source: L-s41. Basis: Workspace.

**L88. The spec marks the catalog's extra keys as extensions that every caller receives.** In the Rust source each extra key has `#[schema(extensions(("x-runvil-extension" = json!(true)), ("x-runvil-opt-in" = json!(false))))]` (`metadata.rs:97-137`, `:207-211`). `openapi.json` (OpenAPI 3.1.0, 12 paths, 38 schemas, title "Runvil Gateway", version 0.3.1) lists `Capabilities` with required `images`, `parallel_tool_calls`, `effort_levels`, `adaptive_thinking`, `voices`. The chat route's one extension field is `choices[].message.gateway_warning`, marked `"x-runvil-opt-in": false` (`openapi.json:1787-1793`, source at `relay.rs:113-118`). The README's rule for such additions (`crates/gateway/README.md:19-21`): "It is named as an extension, with a name inside the standard's own namespace", "It is documented beside the standard shape it extends", and "It is opt-in wherever the protocol offers a way to ask. A client that does not ask never receives the extra field or event." The README also says of a capability endpoint: "no such endpoint exists yet" (`README.md:29`). Source: L-s41, L-s42, L-s35, L-s43. Basis: Workspace.

**L89. The client's catalog reader takes five keys.** `runvil-harness`, `crates/harness-gateway-client/src/catalog.rs:20-32`:

```rust
#[derive(Debug, Deserialize)]
struct ModelsListEntry {
    id: String,
    #[serde(default)]
    description: String,
    #[serde(default)]
    context: Option<u32>,
    #[serde(default)]
    thinking: Option<ThinkingMode>,
    /// The model's provider id, when the gateway names one.
    #[serde(default)]
    provider: Option<String>,
}
```

An entry without `context` is skipped (`:190-195`, "An entry with no context window is not an inference model"). An entry with `context` and without `thinking` fails the whole catalog with "a model declares a context window but no thinking mode" (`:205-210`), and a zero `context` fails with "a model declares a zero-token context window". `thinking` must be one of `never`, `always`, `switchable`. Inference: pointed at a plain OpenAI-shaped `/models` list (entries with `id`, `object`, `created`, `owned_by`), this reader returns an empty catalog, since no entry has `context`. Confidence: high. Source: L-s9. Basis: Workspace.

**L90. The Engine's model info holds the same five facts.** `runvil-harness`, `crates/runvil-internal/types/src/models.rs:162-170`: `pub struct ModelInfo { id: ModelId, description: String, context: NonZeroU32, thinking: ThinkingMode, provider: Option<String> }`. The run's first step compares a role's declared `min_context` and `thinking` with it (`runtime/src/execute/fill.rs:37-61`). The roles are `Default`, `Strong`, `Small`, `Fast`, `Creative` (`types/src/models.rs:117-128`). Source: L-s24, L-s25. Basis: Workspace.

**L91. Seven capability keys are published and applied by nobody.** `runvil-design`, `decisions/gateway.md` section 1 (2026-11-10): "Seven of the nine keys, `default_temperature` included, are facts that a client needs to construct a valid request, so a client applies them. The Harness ignores all seven." At HEAD the Harness reads `provider` (L89) and exposes it to Lua (L9), and the Gateway's request path forwards the body as the client wrote it (L13). The docs on `default_temperature` and `default_effort` say "applied when the caller omits one" (L87), and no code applies them. Source: L-s60, L-s41. Basis: Workspace.

**L92. The provider sheet has a richer entry than the config.** `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/sheet-types.rs:179-240`: `ModelEntry` has `id`, `display_name`, `family`, `variant_of`, `variant`, `languages`, `kind`, `released_at`, `context_window`, `max_output`, `images`, `pdf_input`, `video_input`, `audio_input`, `batch`, `citations`, `code_execution`, `structured_outputs`, `tool_calling`, `thinking: Thinking { supported, enabled, adaptive }`, `effort_levels`, `default_effort`, `pricing`, `deprecation`. The doc on `effort_levels` (`:231-233`) says the names are the provider's own and are "never mapped to a cross-provider scale." The Gateway has its own copy of these types (`runvil-gateway`, `crates/gateway-api-types/src/lib.rs:185-245`). Source: L-s50, L-s41. Basis: Workspace.

**L93. The config UI copies eight sheet facts into a new `[[model]]` and maps the thinking triple to a mode.** `runvil-gateway`, `crates/gateway/config-ui/ui/src/services/cloud-merge.ts:80-97`: the model gets `name`, `kind`, `description`, `context`, `thinking: thinkingMode(entry.thinking)`, `upstream: entry.id`, `endpoints: [provider]`, `images`, `adaptive_thinking: entry.thinking.adaptive`, `effort_levels`, and `max_output` and `default_effort` when present. The mode mapping (`:32-40`):

```ts
function thinkingMode(thinking: CloudModelEntry["thinking"]): string {
  if (!thinking.supported) {
    return "never";
  }
  // Manual budget mode is a per-call switch; a bare reasoning flag
  // means the backend reasons without one.
  return thinking.enabled ? "switchable" : "always";
}
```

The new endpoint is always `protocol: "openai"` with `base_url: slice.openai_base_url` (`:72-77`). The sheet's `tool_calling`, `structured_outputs`, `pdf_input`, `video_input`, `audio_input`, `batch`, `citations`, `code_execution`, and `pricing` stay in the sheet. Source: L-s45. Basis: Workspace.

**L94. For OpenRouter the sheet builder reads eight fields and leaves every capability flag at its base value.** `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/openrouter.rs:57-70`: "One model as the wire reports it. `canonical_slug`, `description`, `per_request_limits`, `supported_parameters`, `default_parameters`, `supported_voices`, and `links` are ignored." The struct reads `id`, `name`, `created`, `context_length`, `architecture` (input and output modalities), `pricing` (prompt and completion), `top_provider.max_completion_tokens`, `expiration_date`. `normalize_model` (`:122-172`) starts from `base_entry`, which is "chat kind, every capability false, every optional field empty" with `tool_calling: false` and `thinking: Thinking::default()` (`providers/openai_shape.rs:39-72`), and sets `kind`, `context_window`, `max_output`, the four input modality flags, `pricing`, and `deprecation`. Inference: every OpenRouter entry in the sheet has `tool_calling: false`, `structured_outputs: false`, `thinking.supported: false`, and empty `effort_levels`, so a model added from the Cloud page gets `thinking = "never"` (L93). Confidence: high. Source: L-s51, L-s53. Basis: Workspace.

**L95. The other five provider files read what each vendor's list endpoint reports.**

- Anthropic (`providers/anthropic.rs`): `x-api-key` and `anthropic-version: 2023-06-01` on the list call (`:39`, `:64-65`), `max_input_tokens`, `max_tokens`, and a `capabilities` object with `batch`, `citations`, `code_execution`, `image_input`, `pdf_input`, `structured_outputs`, `thinking` (with `types.adaptive` and `types.enabled`), and `effort` (levels `low`, `medium`, `high`, `xhigh`, `max`) (`:101-142`). `tool_calling: true` is curated: "tool use is core to every Anthropic chat model; the list endpoint does not report it." (`:214-216`). `openai_base_url: Some("https://api.anthropic.com/v1")` (`:30`).
- OpenAI (`providers/openai.rs:1-4`): "the IDs-only response shape (`id`, `created`, `owned_by`): no pagination, no token limits, and no capability reporting, so every entry is the conservative base entry." `openai_base_url: Some("https://api.openai.com/v1")`.
- Gemini (`providers/gemini.rs`): `x-goog-api-key`, `inputTokenLimit`, `outputTokenLimit`, `supportedGenerationMethods`, and a `thinking` flag. `tool_calling: generates` and `thinking.supported: model.thinking` with `enabled: false` ("The bare reasoning flag says nothing about budget modes") (`:190-202`). `openai_base_url: Some("https://generativelanguage.googleapis.com/v1beta/openai")`.
- xAI (`providers/xai.rs:60-78`): `id`, `created`, `context_length`, and two prices. `openai_base_url: Some("https://api.x.ai/v1")`.
- DeepSeek (`providers/deepseek.rs:1-4`): "the IDs-only OpenAI response shape". `openai_base_url: Some("https://api.deepseek.com/v1")`.

Of the 23 descriptors, 16 have an `openai_base_url` and 7 have `None` (`azure_speech`, `bedrock`, `deepgram`, `elevenlabs`, `foundry`, `leonardo`, `soniox`), which matches `decisions/gateway.md` section 10: "the Cloud page disables Add for seven of the 23 providers." Source: L-s52, L-s53, L-s54, L-s60. Basis: Workspace.

**L96. Inference: the OpenRouter descriptor is keyless, and the Cloud page's merge writes its endpoint without an `api_key`.** `runvil-cloud-providers`, `providers/openrouter.rs:25-33`: `key_env: None`, `openai_base_url: Some("https://openrouter.ai/api/v1")`, `env_vars: &[]`. The merge adds an `api_key` entry (a `${VAR}` reference to the key variable) only when the slice has a key-role variable, and omits the key otherwise (`cloud-merge.ts:71-77`, and its doc at `:44-47`: "omitted for keyless providers"). `EndpointConfig.api_key` is a required field in the config type (`runvil-gateway`, `crates/gateway/config/src/config.rs:342-344`, no `#[serde(default)]`), and `OpenAiUpstream` sends a bearer header only when the key is non-empty (L12). So a working OpenRouter endpoint needs a key that the operator adds by hand. Whether the keyless entry passes `PUT /admin/config` was not run. Confidence: medium. Source: L-s51, L-s45, L-s40. Basis: Workspace.

### Axis 11: caching

**L97. The append-only rule is the Engine's caching behavior.** `runvil-harness`, `crates/runvil-internal/lua/src/messages.rs:531-538`: outside a compactor `replace` raises "replace works only inside a compactor; rewrite history with models.compact(compactor, reason)", and index assignment raises "messages cannot be assigned to by index" (`:584-592`). `decisions/models-and-conversations.md` section 5: "`replace` works only inside a compactor so that the request's prefix changes only at an explicit compaction. A request that repeats the first messages of the previous request can use the provider's prompt cache." Source: L-s17, L-s27. Basis: Workspace.

**L98. System records lead the list for the same reason.** `runvil-harness`, `crates/runvil-internal/lua/src/messages.rs:73-81`: "system messages lead the list, and only a compactor can change the leading block". `decisions/models-and-conversations.md` section 5: "The owner's reason: \"Because of prefix cache invalidation, changes to the prefix of the messages should be visible and explicit\"." Source: L-s17, L-s27. Basis: Workspace.

**L99. The tool list is frozen at a section entry's first loop round.** `runvil-harness`, `crates/runvil-internal/runtime/src/execute/scheduler/chat.rs:116-117`: "The section entry's first round freezes its sets, and every offer raises after that, so each later round sends the same tools." `lua/src/messages.rs:27-30`: "The list also holds the section's frozen sets, which the Engine attaches at the section's first loop round, so they last exactly as long as the section." Source: L-s25, L-s17. Basis: Workspace.

**L100. The Engine measures the shared prefix of consecutive requests for its run log.** `runvil-harness`, `crates/runvil-internal/lua/src/messages.rs:164-178`: `commit` "returns the previous send's round and how many leading messages `sent` shares with that send". `Effect::Chat.after` is `Option<(RoundId, u64)>` (`runtime/src/execute/run/effect.rs:219-224`). The number goes to the effect's record. It is not sent to the model. Source: L-s17, L-s25. Basis: Workspace.

**L101. Prompt caching today rests on the stable prefix alone, and the one cache number read is `cached_tokens`.** The search in L5 returned zero matches for `cache_control` and `prompt_cache` in the Harness. In the Gateway's Rust the only `cache_control` is the HTTP response header (`crates/gateway/protocol/src/upstream.rs:28-29`, `:235-239`), which concerns HTTP caching of the response and is unrelated to prompt caching. A caller-written cache marker on a message or at the top level would pass through the Gateway (L11, L24). The client reads `cached_tokens` (L66), and `Usage` counts cache reads only (L65). Source: L-s31, searches run 2026-11-22. Basis: Workspace.

### Axis 12: multimodal content

**L102. A record's content can be a parts array of two kinds.** `runvil-harness`, `crates/runvil-internal/lua/src/protocol/record.rs:13-20`: `pub enum ContentPart { Text(String), ImageUrl(String) }` with "An image URL; the record parse does not restrict its scheme." At `:167-169`: "The content-part types the chat protocol accepts (the Multimodal contract: text parts and image-URL parts)." `const CHAT_PART_TYPES: [&str; 2] = ["text", "image_url"];`. An `image_url` part must have "an image_url table with a string url field" (`:353-359`), and only `url` is kept. Another type fails with "has unknown type {unknown:?}; known types: {}" (`:345-349`). The content parse runs before the role is resolved (`:211-220`), so parts are accepted on any role. Source: L-s19. Basis: Workspace.

**L103. The projection writes parts as OpenAI content parts.** `runvil-harness`, `crates/runvil-internal/lua/src/projection.rs:269-285`: `{ "type": "text", "text": text }` and `{ "type": "image_url", "image_url": { "url": url } }`. Merged neighbors concatenate their parts, with the separator as a text part (`:208-236`). Source: L-s18. Basis: Workspace.

**L104. A composed system block refuses parts.** Same file, `:60-70`: "messages[{}] is a system message with content parts; only plain text system messages can be composed". Source: L-s18. Basis: Workspace.

**L105. The precheck counts text parts only.** `runvil-harness`, `crates/runvil-internal/lua/src/compactors.rs:227-231`: "The estimate counts message text and tool-call argument characters; an image part contributes nothing (its token cost bears no relation to the URL's length), and tool schemas and the system template are not counted, so an image-heavy conversation can still overflow at the provider - the provider-overflow path covers it." L81 shows that the provider-overflow path is not reached through the Gateway for a remote upstream. Source: L-s22. Basis: Workspace.

**L106. A reply's content must be a string or null.** `runvil-harness`, `crates/harness-gateway-client/src/wire/parse.rs:137-147`: "`content`, when present, must be a string or JSON null; a present value of any other type is a malformed shape." with the error "`content` was present but not a string". The stream side fails a non-string `delta.content` the same way (`wire/stream.rs:241`, `:438-454`). The test `wrong_typed_top_level_fields_are_malformed` covers `"content": []` (`parse.rs:673-676`). Inference: a reply that returns content as a parts array (text parts, image parts, audio) fails the round as `MalformedResponse`. Confidence: high. Source: L-s3, L-s2. Basis: Workspace.

**L107. The Gateway accepts any content shape and publishes `images` as a flag.** `runvil-gateway`, `crates/gateway/protocol/src/wire.rs:106-108`: "A message must be a JSON object with a supported string `role` and must have either `content` (any shape: string, array, or null) or a tool/function call." The `images` capability (L87) is published and the Engine does not read it (L89). The body limit in L14 applies. Source: L-s30. Basis: Workspace.

### Axis 13: the Gateway's one emulated dialect

**L108. The dialect is picked by one string comparison in the chat handler.** `runvil-gateway`, `crates/gateway/app/src/relay.rs:137-143`: "Emulated dialects rewrite the request (guide injection, tool stripping) and parse the reply's content fences. The fence parse needs the whole reply, so the emulated streaming path buffers one non-streaming upstream round trip and re-emits the rewritten response as synthetic chunks; without it an always-streaming caller would silently lose tool calling on this dialect." then `let emulated = model.tool_dialect == crate::dialect::GEMMA3_TOOL_CODE;`. The config value is the enum `ToolDialect { Openai, Gemma3ToolCode }`, default `openai` (`crates/gateway/config/src/config/vocab.rs:68-84`), stored on the routed model as a string (`crates/gateway/routing/src/model.rs:12-18`, `:58-59`). `runvil-design`, `decisions/gateway.md` section 6: "The owner's goal: \"The goal is zero dialects.\"" Source: L-s35, L-s40, L-s36, L-s60. Basis: Workspace.

**L109. The request side removes the tool fields and puts a guide in front.** `runvil-gateway`, `crates/gateway/app/src/dialect.rs:44-66`: `request.rest.remove("tools"); request.rest.remove("tool_choice");` and, when a guide was rendered, `request.messages.insert(0, serde_json::json!({ "role": "system", "content": guide }))`. "Mutation is atomic: the guide is fully rendered before anything is removed, so a preparation failure leaves the request unmodified." A `tools` value that is not an array is refused with `MalformedRequest`. The guide (`dialect/guide.rs:45-84`) tells the model to call tools by writing a sole `tool_code` fence with one call per line, then lists each tool as a signature with its description, and ends with one example. Source: L-s38. Basis: Workspace.

**L110. The reply side has three outcomes.** `runvil-gateway`, `crates/gateway/app/src/dialect.rs:68-115` and `:202-214`: `ContentParse::NotProtocol` leaves prose alone. `ContentParse::Calls` sets `content` to null, writes `tool_calls`, and sets `finish_reason` to `"tool_calls"`. `ContentParse::Malformed(reason)` empties `content` and adds `gateway_warning`. The module doc (`:11-17`): "the gateway is terminal with no fallback, so a recognized-but-malformed fence never fails the turn and never masquerades as final text". The fence parser is in `dialect/content.rs:9-59` and also accepts a JSON fence that holds OpenAI `tool_calls` (`:102-128`). Source: L-s38. Basis: Workspace.

**L111. For a streaming caller the dialect buffers one plain round trip and re-emits it as chunks.** `runvil-gateway`, `crates/gateway/app/src/relay.rs:151-170`: `buffered.stream = false;`, `buffered.rest.remove("stream_options");`, `.send(buffered, &model.upstream_name)`, `crate::dialect::apply_response(&mut response, &model.name);`, `return Ok(relay_sse(crate::dialect::response_chunks(response), permit));`. `response_chunks` (`dialect.rs:117-178`) emits one chunk whose delta is the whole message, adds an `index` to each tool call, and adds the summary chunk (L68). The caller therefore receives no partial text on this dialect, and the non-stream call runs under the 120 second whole-request limit (L60). Source: L-s35, L-s38. Basis: Workspace.

**L112. The dialect has model-specific and tool-specific knowledge inside it.** `runvil-gateway`, `crates/gateway/app/src/dialect.rs:386-398`:

```rust
/// Maps positional `tool_code` args onto schema-ish parameter names.
///
/// Gemma IT frequently emits `search("...")` / `fetch("https://...")` instead
/// of keyword form. Keep this table aligned with shipped tool aliases.
```

The table has entries for `search`, `web_search`, `fetch`, `web_fetch`, and `echo`. Call ids are synthetic (`call_tool_code_{index}`, L46). Source: L-s38. Basis: Workspace.

**L113. As a precedent, the dialect is a set of functions around the upstream call, in the app crate.** The three entry points take and return the Gateway's own wire types: `prepare_request(&mut ChatRequest) -> Result<(), GatewayError>`, `apply_response(&mut ChatResponse, model: &str)`, `response_chunks(ChatResponse) -> StreamedChunks` (`dialect.rs:44`, `:72`, `:127`). They run in the handler before and after `Upstream::send`, so they work with any upstream, remote or local. They do not implement `Upstream`. The integration tests are in `crates/gateway/app/tests/it/chat/gemma.rs` (four tests, including `gemma_stream_true_emulates_tool_calls_over_sse`). `decisions/gateway.md` section 6 adds that a local model's dialect is detected at start (`resolve_local_dialect`), which this map did not open. Source: L-s38, L-s60. Basis: Workspace.

### Axis 14: seams

**L114. The `Upstream` trait has two chat methods that take the request and the upstream model name.** `runvil-gateway`, `crates/gateway/protocol/src/upstream.rs:67-82` and `:148-154`:

```rust
#[async_trait]
pub trait Upstream: Send + Sync {
    async fn send(
        &self,
        req: ChatRequest,
        upstream_model: &str,
    ) -> Result<ChatResponse, ProtocolError>;
```

```rust
    async fn stream(
        &self,
        req: ChatRequest,
        _upstream_model: &str,
    ) -> Result<StreamedChunks, ProtocolError> {
        Err(ProtocolError::ModelUnavailable(req.model))
    }
```

The other methods are `send_embeddings`, `send_rerank`, `send_speech` (each defaulting to `ModelUnavailable`), and `shutdown`. The module doc (`:1-4`): "This crate implements the trait once, as [`OpenAiUpstream`], which forwards the OpenAI shape unchanged." A second implementation, `LocalUpstream`, is in the local crate (L63). Source: L-s31, L-s46. Basis: Workspace.

**L115. A stream is returned as two header values and a boxed stream of typed chunks.** Same file, `:22-34`: `pub struct StreamedChunks { pub content_type: Option<String>, pub cache_control: Option<String>, pub chunks: BoxStream<'static, Result<ChatChunk, ProtocolError>> }`. "The upstream's terminal `[DONE]` sentinel is consumed here, never yielded; the relay emits its own." So an upstream implementation hands the relay `ChatChunk` values (L55), and whatever a translator wants the caller to see has to fit in `model`, `choices[].index`, `choices[].delta`, or the two `rest` maps. Source: L-s31. Basis: Workspace.

**L116. `Protocol` has one variant, and one `match` wires it.** `runvil-gateway`, `crates/gateway/config/src/config/vocab.rs:7-14`:

```rust
/// The wire protocol an endpoint speaks. The OpenAI shape is the only one.
#[derive(Debug, Clone, Copy, PartialEq, Eq, Serialize, Deserialize)]
#[serde(rename_all = "lowercase")]
#[non_exhaustive]
pub enum Protocol {
    /// The OpenAI `/chat/completions` shape.
    Openai,
}
```

`crates/gateway/app/src/routing.rs:98-104`:

```rust
            let upstream: Arc<dyn Upstream> = match endpoint.protocol() {
                Protocol::Openai => Arc::new(OpenAiUpstream::new(
                    endpoint.base_url(),
                    endpoint.api_key().clone(),
                )),
                _ => unreachable!("Protocol is non_exhaustive; wire up new protocols here"),
            };
```

`protocol` is a required key of `[[endpoint]]`, whose other keys are `id`, `base_url`, `api_key`, `dominion` (`config/src/config.rs:330-349`, with `deny_unknown_fields`). The config UI shows the protocol as a locked dropdown with the one option `openai` (`config-ui/ui/src/pages/settings-page.ts:1379-1387`, asserted in `settings-sections.test.mjs:437`), and the Cloud merge writes `protocol: "openai"` (L93). Source: L-s40, L-s36, L-s45. Basis: Workspace.

**L117. An upstream is built per endpoint and sees one per-model value at request time: the upstream model name.** `runvil-gateway`, `crates/gateway/routing/src/model.rs:20-31`: `pub struct Endpoint { pub id: String, pub upstream: Arc<dyn Upstream>, pub queue: DominionQueue }`. `:43-64`: `pub struct Model { name, kind, description, context, thinking, capabilities, tool_dialect, upstream_name, endpoint }`. The handler holds the `Model` and passes `&model.upstream_name` (L13). The constructor receives `endpoint.base_url()` and `endpoint.api_key()` (L116). So the model's `context`, `thinking`, `max_output`, `effort_levels`, and `provider` are in the handler's hands and are not arguments of the trait. Several models can share one endpoint, and a model uses the first endpoint in its list (`routing.rs:129`, `model.endpoints().first()`). Source: L-s36, L-s35. Basis: Workspace.

**L118. The client's public surface is fifteen items, thirteen of them on the model path.** `runvil-harness`, `crates/harness-gateway-client/src/lib.rs:88-102`: `GatewayBroker`, `fetch_model_catalog`, `GatewayEndpoint`, `SecretString`, `provide_gateway_resources`, `TokioTimer`, `GatewayChat`, `make_classify_http_failure_error`, `make_classify_stream_error`, `StreamDelta`, `ChunkSource`, `read_body_capped`, `read_completion_stream`, `build_request_body`, `escape_controls`. Two of them, `provide_gateway_resources` and `TokioTimer`, serve web search and timers. The crate doc (`:57-58`): "The client, or another broker, owns the connection and supplies the wire code's chunks and clock." Source: L-s11. Basis: Workspace.

**L119. Another transport plugs in through `ChunkSource`.** `runvil-harness`, `crates/harness-gateway-client/src/wire/read.rs:23-56`: the trait has one method, `fn next_chunk(&mut self) -> impl Future<Output = Result<Option<Self::Chunk>, Error>> + Send;`, and its doc lists the four steps a transport follows: build the body with `build_request_body`, wrap the response body in a `ChunkSource`, on a failure status read the body with `read_body_capped` and `escape_controls` and fail with `make_classify_http_failure_error`, otherwise call `read_completion_stream`. The scanner, the accumulator, and the normalizer stay private (`wire.rs:5-9`): "so every completion this crate returns passed the one byte cap, the one `[DONE]` rule, and the one turn rule set." `decisions/models-and-conversations.md` section 7: "The wire functions are public so that another broker can reuse them." Source: L-s4, L-s11, L-s27. Basis: Workspace.

**L120. The broker trait has two methods, and the standard broker ignores the binding and the round.** `runvil-harness`, `crates/harness-internal/runner/src/performers.rs:56-76`:

```rust
pub trait InferenceBroker: Send + Sync {
    /// Lists the models the broker serves.
    fn models(&self) -> BoxFuture<'static, Result<ModelCatalog, Error>>;
```

and `fn chat(&self, binding: ModelBinding, messages: Vec<Message>, tools: Vec<ToolSchema>, options: CompletionOptions, round: Round) -> BoxFuture<'static, Result<Box<Completion>, Error>>;`. `GatewayBroker::chat` names its parameters `_binding` and `_round` and forwards `messages`, `tools`, `options` (`harness-gateway-client/src/broker.rs:126-135`). So the model's catalog info (context, thinking mode, provider) reaches the broker inside `binding` and is unused when the request is built. Source: L-s26, L-s8. Basis: Workspace.

**L121. A broker returns a `Completion` built through validating constructors, and its fields are the Engine's whole view of a reply.** `runvil-harness`, `crates/runvil-internal/model-client/src/client/wire.rs:340-360`: `result: CompletionResult`, `finish_reason: Option<String>`, `reasoning_content: Option<String>`, `model: String`, `metrics: Option<CallMetrics>`, `metadata_diagnostics: Vec<String>`, `raw: Option<RawExchange>`. The setters are `from_result`, `with_metrics`, `with_raw`, `with_finish_reason`, `with_reasoning_content`, `with_metadata_diagnostics`, `with_model` (`:362-463`). `raw` is "kept for debug capture" and "The Engine treats both values as opaque and passes them only to the debug capture." (`:290-296`). Source: L-s12. Basis: Workspace.

## Check of the 2026-11-17 report's Table 3 and Appendix D against today's HEAD

The report is "How agent harnesses handle Anthropic's Messages API, and what Runvil should build" (2026-11-17). It mapped `runvil-harness` at `aa7a2a63`.

**Result: every claim in Table 3, in the two gaps below it, and in Appendix D still holds at today's HEAD. Four line ranges moved, and no behavior changed.** The files behind the claims are unchanged since the report's commit (L2), except for a doc comment in `compactors.rs`.

*Table 3 and the two gaps under it.*

| Report claim | Report's location | Status at HEAD | Evidence |
|---|---|---|---|
| Reasoning text becomes one `Thinking` event and is then discarded. No type has a slot for provider data | `runvil-harness`, `execute/turn.rs:60-63`, `:96-98` | Holds, same lines. Precise wording: the text goes to the Host as a `Thinking` event (and into the debug `Response` event), and it is absent from the conversation | L28, L29, L16, L121 |
| The normalizer returns the tool calls and never reads the text | `runvil-harness`, `wire/parse.rs:128-135` | Holds, same lines | L33 |
| Both parsers keep only `data:` lines. The client requires `[DONE]`. The Gateway treats the end of the body as a clean end | `runvil-gateway`, `protocol/src/upstream.rs:229-299` | Holds, same lines | L50, L51, L56, L59 |
| The Gateway's stream parser drops the usage-only chunk. `Usage` has no field for cache writes | `runvil-gateway`, `protocol/src/wire.rs:219-224`; `runvil-harness`, `types/src/metrics.rs:139-153` | Holds. The Gateway check is at `wire.rs:222-227` at HEAD. The report's lines match the Gateway at `fe136b4`, the commit before `fb60a7f` | L57, L65, L68 |
| The Gateway's message is `upstream returned {status}`. A 429 is typed `invalid_request_error`, every 5xx becomes 502, and no upstream header is forwarded | `runvil-gateway`, `protocol/src/error.rs:109-155` | Holds, same lines. On an error response the only header the Gateway adds is `Retry-After` for `model_loading`. On a successful stream it forwards the upstream `Content-Type` and `Cache-Control` | L74 to L77, L62 |
| Nothing retries a failed round, in the client, the Harness, or the Engine | `runvil-harness`, `transport.rs:254-312` | Holds, same lines | L83 |
| The Gateway's stream has no idle timeout. The client allows 120 seconds between chunks | `runvil-gateway`, `protocol/src/http_util.rs:32-44` | Holds, same lines | L54, L60 |
| Catalog fields are published and never applied | `runvil-gateway`, `app/src/models.rs:80-89` | Holds. The code is at `models.rs:97-107` at HEAD (the report's lines match `fe136b4`) | L86, L91, L13 |
| One boolean, sent as `chat_template_kwargs.enable_thinking` | `runvil-harness`, `wire/request.rs:67-71` | Holds, same lines | L4, L7 |
| Free strings, compared in three places with `"stop"`, `"length"`, and `"content_filter"` | `runvil-harness`, `models_loop/machine.rs:322` | Holds. The three places are `wire/stream.rs:347`, `models_loop/machine.rs:322`, `execute/turn.rs:106` | L45, L71, L72 |
| Context overflow cannot be detected through the Gateway for a cloud model | `runvil-harness`, `wire/classify.rs:70-81`; `runvil-gateway`, `protocol/src/error.rs:150-155` | Holds, same lines. L81 extends it to quota, refusal, and overload | L79, L81 |
| The estimate subtracts reasoning tokens | `runvil-harness`, `lua/src/compactors.rs:185-186` | Holds. The comment is at `:183-186` and the subtraction at `:190` at HEAD (one doc line above it was shortened) | L29 |

*Appendix D, Table D1.*

| Stage | Report's file reference | Status at HEAD |
|---|---|---|
| 1 | `runvil-gateway`, `config/src/config/vocab.rs:7-14`, doc line "The OpenAI shape is the only one" | Holds, same lines (L116) |
| 1 | `runvil-gateway`, `app/src/routing.rs:98-104` | Holds, same lines (L116) |
| 1 | `runvil-gateway`, `routing/src/model.rs` | Holds. `Model` has `capabilities`, `thinking`, `context`, and the upstream receives only `upstream_name` (L117) |
| 1 | `runvil-gateway`, `protocol/src/upstream.rs`, new `upstream/anthropic.rs`, 500-line cap | Holds. The cap is `const LIMIT: usize = 500;` in `crates/build-ceiling/src/lib.rs:14`. `upstream.rs` is 303 lines and `upstream/openai.rs` is 326 |
| 1 | `runvil-gateway`, `config-ui/ui/src/pages/settings-page.ts:1379-1387`, a test asserts the lock | Holds, same lines. The test is `settings-sections.test.mjs:437` |
| 2 | `runvil-gateway`, `protocol/src/error.rs:109-155`, `classify` and its table test | Holds, same lines. The table test is at `:184-239` |
| 2 | `runvil-gateway`, `protocol/src/http_util.rs:32-44` | Holds, same lines |
| 2 | `runvil-gateway`, `app/src/relay.rs:198-199` (`[DONE]` after an error event) | Holds. The two lines are at `relay.rs:238-239` at HEAD (the report's lines match `fe136b4`) |
| 3 | `runvil-gateway`, `protocol/src/upstream/openai.rs` | Holds. The file forwards the whole `ChatRequest`, `rest` included (L12) |
| 3 | `runvil-gateway`, `crates/gateway/README.md` and `gateway.local.example.toml` | Both files exist |
| 1 to 3 | `runvil-gateway`, `app/tests/it/support/backends.rs`, new `tests/it/chat/anthropic.rs` | `support/backends.rs` exists. `tests/it/chat/` has `catalog.rs`, `gemma.rs`, `streaming.rs` |
| 4 | `runvil-harness`, `wire/stream.rs:169-175` | Holds, same lines (L52) |
| 4 | `runvil-harness`, `transport.rs` | Holds (L6, L83) |
| 4 | `runvil-harness`, `wire/classify.rs:70-128` | Holds, same lines (L79, L80) |
| 5 | `runvil-harness`, `model-client/src/client/wire.rs:29-47`, `:342-360` | Holds, same lines (L15, L121) |
| 5 | `runvil-harness`, `lua/src/protocol/record.rs` and `protocol/answer.rs` | Holds. `answer.rs` lost `VfsOutcome` since the report, and `ChatResult` is at `:93-104` |
| 5 | `runvil-harness`, `lua/src/models_loop/machine.rs:306-334`, `:409-430`, `:458-476` | Holds, same lines (L20, L37) |
| 5 | `runvil-harness`, `lua/src/projection.rs:268-316` | Holds, same lines (L21) |
| 5 | `runvil-harness`, `lua/src/compactors.rs:185-193` | Holds. `UsageAnchor::new` is at `:188-192` |
| 5 | `runvil-harness`, `types/src/metrics.rs:139-153` | Holds, same lines (L65) |
| 5 | `runvil-harness`, `decisions/models-and-conversations.md` and `decisions/engine.md` | Both exist and both changed since the report (last commits `d5c2ebe9` on 2026-11-20 and `9ba47f6e` on 2026-11-22). The sections on the client, messages, and the loop say the same things as before |
| 5 | `runvil-workshop`, `Cargo.toml:28-31` | Holds. The pin is on lines 28 to 30 and names `ee89d0882ec8cecfcd0159c7f61c6741fceef612` |

**What this map adds beyond the report's Table 3**, each a finding from today's reading and not a correction: the Gateway drops a bare upstream in-stream error object (L85), usage survives on a chunk that has a choice (L58, L68), upstream keep-alive comments end at the Gateway (L61), tool-call arguments are replayed re-serialized (L47), call ids must be unique across the whole conversation while the emulated dialect restarts its ids per reply (L46), the chat route has a 2 MB body limit (L14), the thinking switch is sent whenever a role declares it (L10), the standard broker ignores the binding (L120), a reply with array content fails (L106), and every OpenRouter entry in the sheet has its capability flags at the base value (L94).

## Conflicts and disagreements

1. **The Harness asks for a chunk shape that the Gateway calls malformed.** The client's request doc says `stream_options.include_usage` yields "a final chunk with an empty `choices` list" (L67). The Gateway's chunk check says a chunk with empty `choices`, "for example a usage-only summary object a backend appends mid-stream", is malformed and skips it (L57). Both are code facts. The Gateway's rule wins on the wire, so usage from an OpenAI-shaped upstream does not reach the client (L68).
2. **Two doc comments promise a default that nothing applies.** `default_temperature` is "Sampling temperature applied when the caller omits one" and `default_effort` is "The effort level applied when the caller omits one" (L87). The request path reads no capability key (L13), and `runvil-design`, `decisions/gateway.md` says the Harness ignores them and lists the unused `default_max_tokens` as a known gap. The code and the decision page are right. The doc comments describe intent.
3. **The Gateway's stream doc is wider than its code.** The doc on `sse_chunks` says a mid-stream transport failure surfaces as an error "so a caller never mistakes a truncated stream for a complete one" (L56). That holds for a read error. An upstream body that ends cleanly without `[DONE]` ends the chunk stream normally, and the relay then writes `[DONE]` (L59), so that kind of truncation looks complete to the caller.
4. **Tool-call id rules disagree inside the workspace.** The projection requires ids unique across the whole list. The metrics type says "Providers recycle ids like `call_1` across rounds". The Gateway's emulated dialect starts its id counter at 0 for every reply (L46). Reading alone cannot say which wins in a two-round tool loop on such a model. A run would.
5. **A decision page states the thinking switch more narrowly than the code.** `runvil-harness`, `decisions/models-and-conversations.md` section 1: "A `Switchable` model passes both, and each round on the role sends the switch as the request field `chat_template_kwargs.enable_thinking`." The code sets the switch whenever the role declares `thinking` and the check passes, which includes `thinking: true` on an `Always` model and `thinking: false` on a `Never` model (L10). The code is the fact.
6. **The brief's summary of the Engine is ahead of the record type.** Section 2 of the shared brief says "The Engine can keep data in the list that Lua cannot see." At HEAD the list keeps hidden state beside the records (`last_send`, the frozen tool sets), and every field of a record is Lua-visible (L16, L17, L18). `decisions/engine.md` gives the hidden per-reply data as the reason the loop is in Rust and calls its use "planned work".
7. **`decisions/gateway.md` describes older commits.** The page is dated 2026-11-10 and names `runvil-gateway` `9369a7d` and `runvil-cloud-providers` `1817cc4`. Every statement taken from it here was re-read in the code at `52fd92d` and `50961b2` and holds, including the count of 23 providers with seven lacking an OpenAI-compatible base URL (L95).
8. **The 2026-11-17 report and today's HEAD agree.** Four line ranges moved (three in the Gateway, where the report's numbers match the commit before `fb60a7f`, and one in `compactors.rs`). See the section above.

## Open questions blocked by closed source

This topic is local code, and every question it raised is answerable from the workspace. One dependency on closed-source behavior belongs to other topics and is recorded here so the lead agent can join the two halves:

- **Which OpenRouter stream chunks survive the Gateway depends on their exact shape.** L58 gives the rule: a chunk survives when it has a string `model` and at least one choice with `index` and `delta`. The brief's claim LC12 says OpenRouter's final usage chunk has a non-empty `choices` array, and LC11 says a mid-stream error arrives as a chunk with `finish_reason: "error"`. If both chunks also have `model` and a `delta`, then OpenRouter's usage and its mid-stream error would pass the Gateway, where OpenAI's usage chunk and a bare error object do not. This file did not check LC11 or LC12. What was tried: nothing on the web, by the task's instruction. Fill: none sought here. The streaming topic's evidence file should state the exact chunk shapes.

## Unverified

- **Nothing was built or run.** Every item marked "Inference" (L10, L14, L36, L43, L46, L53, L58, L61, L68, L81, L85, L89, L94, L96, L106) is a reading of types and control flow.
- **The 2 MB body limit (L14)** rests on the dependency's source in the local cargo registry (`axum-core` 0.5.6) and on a repository-wide search for a limit override. No oversized request was sent.
- **Whether a keyless OpenRouter endpoint passes config validation (L96)** was not run.
- **Whether a recycled tool-call id breaks a second tool round (L46)** was not run. The Engine's loop tests were not searched for a case that reuses an id across rounds.
- **What a Host's recorder stores of a `Thinking` event** was not traced past `emitter.thinking` (L28).
- **The front matter parser for `models:`** was not opened. The `min_context` and `thinking` keys are taken from `decisions/models-and-conversations.md` and from their use in `execute/fill.rs`.
- **`resolve_local_dialect`** (local dialect detection, L113) was not opened. The statement is from `decisions/gateway.md`.
- **The Lua error table's `message` value** is taken from the decision page ("Lua receives the prose"). The helper that builds it was not opened.
- **The shapes of llama.cpp's and OpenRouter's usage and error chunks** were not checked. L58, L68, and L85 state the Gateway's rule, and the outcome for a given upstream depends on that upstream's chunk shape.
- **The Gateway's integration tests** were listed by name (`tests/it/chat/streaming.rs`, `gemma.rs`) and not read in full.

## Workspace documents used

- **Cabinet output, 2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build".** Took sections 5 and 6 and Appendix D as the map to verify. Re-checked: yes, every Table 3 and Appendix D reference was opened at today's HEAD (see the check section).
- **`runvil-harness`, `decisions/models-and-conversations.md`** (last commit `d5c2ebe9`, 2026-11-20). Took the stated rules for roles, `models.use`, the loop, messages, compaction, and the client. Re-checked against code: yes, for every statement cited in an evidence item.
- **`runvil-harness`, `decisions/engine.md`** (last commit `9ba47f6e`, 2026-11-22). Took section 4 (`Effect::Chat`) and section 5 (the loop's rules run in Rust, the "Lua can't see it" intent). Re-checked against code: yes.
- **`runvil-design`, `decisions/gateway.md`** (2026-11-10, describes `runvil-gateway` `9369a7d`). Took the request walk, the capability-key statement, the dialect entry, the planned-work list, and known gap 3. Re-checked against code at `52fd92d`: yes for every statement cited.
- **`runvil-gateway`, `crates/gateway/README.md`** (last commit `fb60a7f`, 2026-11-15). Took the extension rule and the capability-endpoint statement. It is the current file.
- **`runvil-harness`, `AGENTS.md`; `runvil-harness`, `crates/harness-gateway-client/AGENTS.md`; `runvil-gateway`, `AGENTS.md`** (all at HEAD). Took the definitions of Engine, Harness, Host, and the one-builder rule for the request body.
- **The shared brief for this deep dive** (2026-11-22, scratch). Took the rules, the ladder, and claims LC11 and LC12 as pointers only.

## Sources

All read on 2026-11-22. Paths are relative to the repository root. Commits: `runvil-harness` `22a0992f`, `runvil-gateway` `52fd92d`, `runvil-cloud-providers` `50961b2`, `runvil-design` `217c27d`, `runvil-workshop` `f40a4fce`.

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| L-s1 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/wire/request.rs` | 2026-11-22 | harness source |
| L-s2 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/wire/stream.rs` | 2026-11-22 | harness source |
| L-s3 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/wire/parse.rs` | 2026-11-22 | harness source |
| L-s4 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/wire/read.rs` | 2026-11-22 | harness source |
| L-s5 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/wire/classify.rs` | 2026-11-22 | harness source |
| L-s6 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/wire/delta.rs` | 2026-11-22 | harness source |
| L-s7 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/transport.rs` | 2026-11-22 | harness source |
| L-s8 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/broker.rs` | 2026-11-22 | harness source |
| L-s9 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/catalog.rs` | 2026-11-22 | harness source |
| L-s10 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/failure.rs` | 2026-11-22 | harness source |
| L-s11 | `runvil-harness` at `22a0992f`, `crates/harness-gateway-client/src/lib.rs`, `src/wire.rs`, `src/config.rs`, `AGENTS.md` | 2026-11-22 | harness source |
| L-s12 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/model-client/src/client/wire.rs`, `client/role.rs` | 2026-11-22 | harness source |
| L-s13 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/model-client/src/model/options.rs` | 2026-11-22 | harness source |
| L-s14 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/model-client/src/model/error.rs` | 2026-11-22 | harness source |
| L-s15 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/model-client/src/normalize.rs` | 2026-11-22 | harness source |
| L-s16 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/model-client/src/detail.rs` | 2026-11-22 | harness source |
| L-s17 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/messages.rs` | 2026-11-22 | harness source |
| L-s18 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/projection.rs`, `projection-tests.rs` | 2026-11-22 | harness source |
| L-s19 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/protocol/record.rs` | 2026-11-22 | harness source |
| L-s20 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/protocol/answer.rs`, `protocol/request.rs`, `protocol/parse/chat.rs` | 2026-11-22 | harness source |
| L-s21 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/models_loop/machine.rs` | 2026-11-22 | harness source |
| L-s22 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/compactors.rs` | 2026-11-22 | harness source |
| L-s23 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/lua/src/models.rs`, `error_value.rs` | 2026-11-22 | harness source |
| L-s24 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/types/src/metrics.rs`, `models.rs`, `event.rs`, `error.rs` | 2026-11-22 | harness source |
| L-s25 | `runvil-harness` at `22a0992f`, `crates/runvil-internal/runtime/src/execute/turn.rs`, `scheduler/chat.rs`, `infer.rs`, `fill.rs`, `requirements.rs`, `run/effect.rs` | 2026-11-22 | harness source |
| L-s26 | `runvil-harness` at `22a0992f`, `crates/harness-internal/runner/src/performers.rs`, `effect_loop.rs` | 2026-11-22 | harness source |
| L-s27 | `runvil-harness` at `22a0992f`, `decisions/models-and-conversations.md`, `decisions/engine.md`, `AGENTS.md` | 2026-11-22 | workspace document |
| L-s30 | `runvil-gateway` at `52fd92d`, `crates/gateway/protocol/src/wire.rs`, `wire/envelope.rs` | 2026-11-22 | gateway source |
| L-s31 | `runvil-gateway` at `52fd92d`, `crates/gateway/protocol/src/upstream.rs` | 2026-11-22 | gateway source |
| L-s32 | `runvil-gateway` at `52fd92d`, `crates/gateway/protocol/src/upstream/openai.rs` | 2026-11-22 | gateway source |
| L-s33 | `runvil-gateway` at `52fd92d`, `crates/gateway/protocol/src/error.rs` | 2026-11-22 | gateway source |
| L-s34 | `runvil-gateway` at `52fd92d`, `crates/gateway/protocol/src/http_util.rs` | 2026-11-22 | gateway source |
| L-s35 | `runvil-gateway` at `52fd92d`, `crates/gateway/app/src/relay.rs` | 2026-11-22 | gateway source |
| L-s36 | `runvil-gateway` at `52fd92d`, `crates/gateway/app/src/routing.rs`, `crates/gateway/routing/src/model.rs` | 2026-11-22 | gateway source |
| L-s37 | `runvil-gateway` at `52fd92d`, `crates/gateway/app/src/models.rs` | 2026-11-22 | gateway source |
| L-s38 | `runvil-gateway` at `52fd92d`, `crates/gateway/app/src/dialect.rs`, `dialect/content.rs`, `dialect/guide.rs`, `crates/gateway/app/tests/it/chat/gemma.rs` (test names) | 2026-11-22 | gateway source |
| L-s39 | `runvil-gateway` at `52fd92d`, `crates/gateway/app/src/error.rs`, `error/classify.rs`, `error/wire.rs` | 2026-11-22 | gateway source |
| L-s40 | `runvil-gateway` at `52fd92d`, `crates/gateway/config/src/config.rs`, `config/vocab.rs`, `config/validate.rs`, `config/accessors-models.rs` | 2026-11-22 | gateway source |
| L-s41 | `runvil-gateway` at `52fd92d`, `crates/gateway-api-types/src/metadata.rs`, `src/lib.rs` | 2026-11-22 | gateway source |
| L-s42 | `runvil-gateway` at `52fd92d`, `crates/gateway/app/openapi.json` | 2026-11-22 | gateway source |
| L-s43 | `runvil-gateway` at `52fd92d`, `crates/gateway/README.md`, `AGENTS.md` | 2026-11-22 | workspace document |
| L-s44 | `runvil-gateway` at `52fd92d`, `crates/gateway/protocol/src/upstream/tests/stream.rs` | 2026-11-22 | gateway source |
| L-s45 | `runvil-gateway` at `52fd92d`, `crates/gateway/config-ui/ui/src/services/cloud-merge.ts`, `pages/settings-page.ts`, `pages/settings-sections.test.mjs`, `components/settings-registry.ts` | 2026-11-22 | gateway source |
| L-s46 | `runvil-gateway` at `52fd92d`, `crates/gateway/local/src/upstream.rs` | 2026-11-22 | gateway source |
| L-s47 | `axum-core` 0.5.6 (the version `runvil-gateway`'s `Cargo.lock` pins), `src/extract/default_body_limit.rs`, `src/ext_traits/request.rs`, read in the local cargo registry | 2026-11-22 | gateway source (dependency) |
| L-s50 | `runvil-cloud-providers` at `50961b2`, `crates/gateway-cloud-providers/src/sheet-types.rs`, `src/lib.rs` | 2026-11-22 | gateway source |
| L-s51 | `runvil-cloud-providers` at `50961b2`, `crates/gateway-cloud-providers/src/providers/openrouter.rs`, `providers/openrouter-taxonomy.rs` | 2026-11-22 | gateway source |
| L-s52 | `runvil-cloud-providers` at `50961b2`, `crates/gateway-cloud-providers/src/providers/anthropic.rs` | 2026-11-22 | gateway source |
| L-s53 | `runvil-cloud-providers` at `50961b2`, `crates/gateway-cloud-providers/src/providers/openai.rs`, `providers/openai_shape.rs`, `providers/xai.rs`, `providers/deepseek.rs` | 2026-11-22 | gateway source |
| L-s54 | `runvil-cloud-providers` at `50961b2`, `crates/gateway-cloud-providers/src/providers/gemini.rs` | 2026-11-22 | gateway source |
| L-s60 | `runvil-design` at `217c27d`, `decisions/gateway.md` (dated 2026-11-10) | 2026-11-22 | workspace document |
| L-s61 | Cabinet output, 2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build" (sections 5, 6, Appendix D) | 2026-11-22 | workspace document |
| L-s62 | `runvil-workshop` at `f40a4fce`, `Cargo.toml` | 2026-11-22 | harness source |
