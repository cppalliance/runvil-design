---
produced: 2026-11-22
title: OpenRouter server tools, plugins, and content types - web search, native versus OpenRouter-run tools, tool search and deferred_tools, citations and annotations, PDFs and Files API, multimodal input and output, MCP
---

# OpenRouter server tools, plugins, and content types - web search, native versus OpenRouter-run tools, tool search and deferred_tools, citations and annotations, PDFs and Files API, multimodal input and output, MCP

- What was read: 34 OpenRouter documentation pages (server tools, plugins, Files API, containers, multimodal, changelog), OpenRouter's OpenAPI spec, the public endpoints listing for 27 models, OpenRouter's AI SDK provider source, eight vendor documentation pages (Anthropic, OpenAI, Google), LiteLLM source at a pinned commit as a fill, and four workspace documents.
- How many sources: 51 (listed at the end with keys P-s1 to P-s57; six numbers in that range were not used).
- Read date: 2026-11-22 for every source. No model was called and no credential was used.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
| - | - | - | - | - | - | - | - |
| Web search run for the model | Anthropic, OpenAI, Google, xAI, Perplexity; Exa, Parallel, Firecrawl, Perplexity Search as engines | Each vendor has its own tool: Anthropic `web_search_20250305` and later, OpenAI `web_search`, Gemini `google_search`, xAI `web_search` | One tool, `{"type": "openrouter:web_search", "parameters": {...}}`, in `tools` on all three routes. `parameters.engine` is `auto`, `native`, `exa`, `firecrawl`, `parallel`, or `perplexity`. `auto` runs the vendor's tool where the endpoint lists it in `native_tools`, else Exa. OpenAI's `web_search` spelling is converted to it | With native: `max_results`, `search_context_size`, `max_characters` are ignored; `max_uses` reaches Anthropic only; Google native takes no domain filter. What the model sees differs by engine | OpenRouter runs the loop inside one request. Neutral setting, translated per vendor. All three routes. Behavior depends on which endpoint serves the request | OR-documented | P10, P11, P12, P13, P14, P20 |
| Replay of web search results | Anthropic | `web_search_tool_result` blocks hold `encrypted_content` that the caller "must pass back"; a missing or changed value fails with a 400 | Messages route: request schema accepts `server_tool_use` and `web_search_tool_result`, with `encrypted_content` required per result. Chat completions: not documented | Where the result blocks go in the chat completions shape is undocumented | Caller holds opaque data and returns it unchanged. LiteLLM solves the chat case with an extra message field (`provider_specific_fields.web_search_results`) | OR-documented for Messages; Filled (LiteLLM) for chat | P31, P33, P36 |
| Paused server tool turn | Anthropic | `stop_reason: "pause_turn"`; "send the paused assistant message back unchanged in a new request" | The Messages stop reason enum includes `pause_turn`. No guide text. Chat completions mapping is not documented | Unknown on chat completions | Caller must resend the partial assistant turn unchanged. LiteLLM maps it to `finish_reason: "length"` and keeps the raw value in an extra field | OR-documented (enum only); Filled (LiteLLM) | P32, P34, P85 |
| Web fetch | Anthropic (`web_fetch_20260209`); OpenRouter engines | Anthropic's server tool returns `web_fetch_tool_result` blocks | `openrouter:web_fetch` with `engine` of `auto`, `native`, `exa`, `openrouter`, `firecrawl`, `parallel`. Native on the "Anthropic" and "Claude Platform on AWS" endpoints in the sample | The Messages request schema leaves `web_fetch_tool_result` out of the history block list. The domain block field is `blocked_domains` here and `excluded_domains` on search | Stateless on OpenRouter's engines. Neutral setting. All three routes | OR-documented | P12, P16, P31 |
| Search citations | OpenAI, Anthropic, Google, Perplexity, xAI | OpenAI: `url_citation` annotations that "must" be shown as "prominent, clickable links". Anthropic: citation objects with `encrypted_index`. Google: `url_citation` annotations plus a `search_suggestions` HTML snippet under its terms | Chat: `message.annotations[]` with `type: "url_citation"` and a nested `url_citation` object (`url`, `title`, `content`, `start_index`, `end_index`). Responses: flat `url_citation` plus `uncited: true`. Messages: Anthropic's citation objects | Vendor display duties and Google's suggestions snippet are not mentioned in any guide. `annotations` is missing from the chat schemas in the spec | Output only; nothing to send back for URL citations. The caller reads one shape per route | OR-documented; the gaps are OR-inferred | P38, P39, P40, P41, P45 |
| Citations on supplied documents | Anthropic | `citations: {"enabled": true}` on document blocks; text blocks return character, page, or content block locations | Messages route keeps Anthropic's shapes. Chat completions: not documented | Unknown on chat completions | The request flag sits on a vendor-shaped block. LiteLLM parks the result in an extra message field | OR-documented for Messages; Filled (LiteLLM) for chat | P43, P45 |
| Tool search and deferred loading | Anthropic (`tool_search_tool_regex`, `tool_search_tool_bm25`, `tool_reference`), OpenAI (`tool_search`, namespaces) | Caller sends every definition with `defer_loading: true`; the vendor expands hits on its side; search blocks or items must come back unchanged | `defer_loading` on each route (position differs). `openrouter:tool_search` (regex, run by OpenRouter) on Responses and Messages. Anthropic endpoints get passthrough; elsewhere OpenRouter withholds definitions and appends them when revealed | The BM25 tool spelling returns 400. Chat completions refuses the tool type. Off Anthropic, a `tool_reference` block "reaches the provider as text" | Caller holds the full catalog and sends it every turn. Neutral flag. On chat completions the caller needs `deferred_tools` or its own search tool | OR-documented | P46, P47, P48, P49, P55 |
| Deferred tools protocol state | OpenRouter (any model) | Not a vendor feature | Top-level `deferred_tools` (`protocol`, `profile`, `validation`, `search`) on all three routes. State is hidden inside returned encrypted reasoning: a `reasoning_details` entry on chat, `encrypted_content` on Responses, `redacted_thinking` on Messages | Context compression is off; batch is refused; hosted tools other than web search and code interpreter return 400 | Caller holds state as an opaque reasoning entry and must return every assistant message unchanged, with `tools`, `tool_choice`, and settings fixed for the whole conversation. Four error codes, such as `deferred_tools_state_missing` | OR-documented | P50, P51, P52, P53 |
| Code execution and code interpreter | Anthropic, OpenAI, Google | Vendor-hosted sandbox tools with their own result blocks or items and container ids | Responses: OpenAI's `code_interpreter` type is accepted (a container id or `file_ids` needs BYOK). Messages: the response schema has code execution result blocks; the request union names six Anthropic tool types, and code execution is outside them. Chat: only `function`, `openrouter:*`, and the web search shorthand. Gemini code execution: outside all three unions | Not wrapped as an `openrouter:*` tool. OpenRouter's substitute is its own sandbox (`openrouter:shell`, `openrouter:bash`) | Reachable only on the vendor-shaped route, so the caller must know the vendor. Reachability on Messages is unconfirmed | OR-documented; reachability is OR-inferred | P19, P22, P23 |
| Computer use | Anthropic, OpenAI | Screen control tools that the client executes | Messages: "rejected at validation". Responses: `computer_use_preview` is in the schema | Anthropic computer use is unreachable | Not available on the Messages route | OR-documented | P19, P21 |
| Remote MCP run by the server | Anthropic (`mcp_servers`, `mcp_toolset`, beta header), OpenAI (`mcp` tool) | The vendor connects to the MCP server and returns MCP blocks or items | Responses only: `mcp` and `openrouter:mcp` tool types in the spec since 2026-11-05, with items `mcp_list_tools`, `mcp_call`, `mcp_approval_request`. The guide tells callers to convert MCP tools to function tools themselves | Anthropic's connector: `mcp_servers` is outside the `MessagesRequest` property list. The chat completions and Messages unions leave MCP out. The guides do not describe the server-run tool | Responses route only. An approval item must be answered on the next request. Otherwise the caller runs MCP | OR-documented | P80, P81, P82 |
| Hosted shell and sandbox | OpenAI (`shell`), Anthropic (`bash_20250124`, client-run) | OpenAI runs commands in its container; Anthropic's bash tool hands the call to the client | `openrouter:shell` (Responses, Messages) and `openrouter:bash` (Messages) with `parameters.engine`. Containers are keyed by `session_id`, the `x-session-id` header, or `prompt_cache_key` | Chat completions returns 400 for shell | OpenRouter holds the container. The caller holds a session id or replays `container_id`. Not on chat completions | OR-documented | P4, P17, P29, P69 |
| Apply patch | OpenAI | `apply_patch_call` items with V4A diffs and streamed diff deltas | `openrouter:apply_patch`, Responses only. Native passthrough on OpenAI endpoints, a validator elsewhere. The caller applies the patch and answers with `apply_patch_call_output` | Only OpenAI endpoints stream the diff | Client-executed. `call_id` round trip. Responses route only | OR-documented | P4, P8, P17, P30 |
| Advisor (consult a second model) | Anthropic (`advisor_20260301`) | Server tool with an `advisor_tool_result` block | `openrouter:advisor` on chat and Responses; Anthropic's spelling on Messages; any model can be the advisor | The native spelling ignores `max_uses`, `caching`, `allowed_callers`, `defer_loading` | For memory the caller replays the advisor exchange unchanged, and identity is positional in `tools`. On chat completions it shows as `tool_calls` plus `role: "tool"` messages | OR-documented | P17, P27, P30 |
| Subagent and Fusion (inner model runs) | OpenRouter only | Not a vendor feature | `openrouter:subagent`, `openrouter:fusion` on chat and Responses. Inner runs are billed into the request | Function inheritance with suspend and resume is Responses only | With inherited functions the caller replays opaque `subagent_items` unchanged. Otherwise stateless | OR-documented | P4, P8, P30 |
| Record of a server tool call in history | OpenRouter | Not a vendor feature | Chat: a `reasoning_details` entry of type `reasoning.server_tool_call` (`tool_name`, `arguments`, `result`). Responses: typed `openrouter:*` items plus a `rs_server_tool_round_*` reasoning item. Messages: `server_tool_use` plus a result block | Three shapes for one event. OpenRouter's own AI SDK provider filters out reasoning detail types it does not know | An entry the caller stores with the assistant message. Whether it must be returned is not documented | OR-documented; the consequence is OR-inferred | P26, P28, P29, P37 |
| Server tool usage counters | Anthropic (`usage.server_tool_use`) | Per-tool request counts | Chat and Responses: `usage.server_tool_use_details` (`tool_calls_requested`, `tool_calls_executed`, `web_search_requests`). Messages: `usage.server_tool_use` | One tool-specific counter. Native calls are not in `tool_calls_requested` | Output only | OR-documented | P25 |
| Step budget for server tools | OpenRouter; Anthropic has `max_uses` per tool | Anthropic caps each tool with `max_uses` | `max_tool_calls` (default and maximum 30; in the spec on Responses only) and `stop_server_tools_when` on all three routes | Nothing noted | Neutral request settings | OR-documented | P9 |
| Plugins | OpenRouter | Not a vendor feature | `plugins[]` entries with `id`; fourteen ids in the spec; account defaults and "Prevent overrides" | The web plugin and `:online` are deprecated | Behaviors run inside OpenRouter, chosen per request or per account. The caller keeps nothing except file annotations | OR-documented | P57, P58, P59, P60, P61, P62 |
| PDF input | Anthropic, OpenAI, Google take files natively; other models get parsed text | Vendors take PDFs as document or file parts | Chat `file` part (`file_data`, `filename`); Responses `input_file`. `file-parser` plugin with `pdf.engine` of `native`, `mistral-ocr`, `cloudflare-ai`. Default: native, else Mistral OCR | OCR output is capped at 8 images. Parse cost is billed even with BYOK | Neutral part. OpenRouter picks native or parsed per model | OR-documented | P63, P64, P65 |
| Reuse of a parsed file | OpenRouter | Not a vendor feature | Chat: an `annotations[]` entry of `type: "file"` with `hash` and parsed `content`. Sending it back on the assistant message skips re-parsing. Also in `error.metadata.file_annotations` | Responses returns `file_citation`, which cannot be reused | The caller holds a possibly large blob and returns it unchanged. Optional: it saves cost only | OR-documented | P44, P66 |
| Uploaded files and vendor file ids | Anthropic (`file_id`), OpenAI (file ids, vector stores), Google (File API URIs) | Upload once and reference by id | OpenRouter Files API with `or_file_` ids. `?provider=openai` or `?provider=anthropic` forwards to the vendor's store with the caller's BYOK key. `file_id` fields exist on file and image parts in the spec | Google file URIs are not mentioned anywhere. Whether an id works as model input is unclear | A server holds the file. The id is specific to OpenRouter or to one vendor. Vendor stores need the caller's own vendor key | OR-documented; the gap is OR-inferred | P67, P68, P70 |
| Image input | All vision vendors | URL or inline data; limits differ per vendor | Chat `image_url` part (URL or data URL; `detail` includes the OpenRouter value `original`). Responses `input_image`. Messages `image` | Per-provider image count limits are not listed. Whether OpenRouter downloads URLs for inline-only providers is not stated | Neutral part | OR-documented; download behavior Filled (LiteLLM) | P71, P72, P73, P86 |
| Audio input | OpenAI, Google, others | Inline audio with a format name | Chat `input_audio` (`data`, `format`; any format the provider supports). Responses accepts `wav` and `mp3` only. Base64 only | No URL input | Neutral part. Format support varies by model | OR-documented | P74 |
| Video input | Google mainly | Gemini takes YouTube links on AI Studio or inline data, and has an agentic video mode | Chat `video_url` part (`url`, `processing`). Responses `input_video`. Messages: video input is unsupported. URLs are sent only to providers that support them | `processing` is Gemini only; "Other providers ignore" it | Agentic mode returns encrypted reasoning that the caller must return to keep the video context | OR-documented | P72, P75 |
| Image output from a chat model | Google, OpenAI image models | The vendor returns image parts in the reply | Spec: `modalities: ["text", "image"]`, `message.images[].image_url.url`, `image_config`. The guides now document only `POST /api/v1/images`. Also the `openrouter:image_generation` server tool | The stream delta schema leaves `images` out | An extension field outside OpenAI's message shape. 13 of 476 models | OR-documented | P8, P76, P79 |
| Audio output | OpenAI audio models | `modalities` plus `audio` (`voice`, `format`) | Chat completions only and streaming only, through `delta.audio` (`data`, `transcript`) | The Responses route "does not accept the `audio` parameter" | Voice names are model-specific. 4 of 476 models | OR-documented | P77, P79 |
| Dedicated media routes | Various | Not one vendor's feature | `/api/v1/images`, `/api/v1/videos` (async job with polling), `/api/v1/audio/speech`, `/api/v1/audio/transcriptions` | Outside the chat shape | Separate routes | OR-documented | P78 |
| X search | xAI | A separate `x_search` tool billed per item returned | `parameters.x_search` on `openrouter:web_search` (or on the `web` plugin); legacy top-level `x_search_filter` | Ignored on other vendors | A vendor-specific opt-in object inside a neutral tool | OR-documented | P15 |
| Engine depends on the serving endpoint and region | All | Not one vendor's feature | `native_tools` per endpoint: Anthropic native search on "Anthropic" and "Claude Platform on AWS" only; OpenAI on "OpenAI" and "Azure"; empty on Bedrock. Regional hosts restrict engines and answer `403` | One model can run a different search engine depending on routing | The caller can send the request without knowing the provider. Cost and result shape can differ by provider | OR-documented | P12, P18 |
| Tool call safety check and message check | Anthropic (`safeguards`, Claude Code auto mode) | Anthropic evaluates tool calls through a `safeguards` request field | `plugins: [{"id": "armor"}]` or Anthropic's `safeguards` field; response gains `safeguard_results` and, on a block, an `openrouter:armor_blocked` item. `alignment` plugin adds an `alignment` object | Both are in beta | Runs inside OpenRouter. The response gains extra objects | OR-documented | P61 |

## Evidence

Every item has one basis label. "Spec" means OpenRouter's OpenAPI document (source P-s35), queried with Python on 2026-11-22. Where an item says "I decoded" or "I tabulated", the step is a mechanical transformation of a cited source and the script sits in the scratch folder for this topic.

### A. The server tool list and where each one is accepted

**P1. Server tools are a beta feature that OpenRouter runs inside the request, and they sit in the `tools` array under an `openrouter:` type prefix.**

- Quote: "Server tools are currently in beta. The API and behavior may change."
- Quote: "Server tools are specialized tools operated by OpenRouter that any model can call during a request. When a model decides to use a server tool, OpenRouter executes it server-side and returns the result to the model, so no client-side implementation is needed."
- Quote (comparison table): "**Who executes it** | OpenRouter | OpenRouter | Your application" and "**Type prefix** | `openrouter:*` | N/A | `function`".
- Source: P-s1. Basis: OR-documented.

**P2. The guide lists thirteen server tools.**

- From the "Available Server Tools" table: `openrouter:web_search`, `openrouter:datetime`, `openrouter:image_generation`, `openrouter:web_fetch`, `openrouter:apply_patch` ("Responses API only"), `openrouter:shell` ("Responses and Messages APIs"), `openrouter:bash` ("Messages API only"), `openrouter:fusion`, `openrouter:advisor`, `openrouter:subagent`, `openrouter:experimental__search_models`, `openrouter:tool_search` ("Responses and Messages APIs"), `openrouter:files`.
- Every tool page opens with the same beta note. The search models page adds: "This tool is experimental. When it graduates, the tool type is likely to be renamed (dropping the `experimental__` prefix), which would be a breaking change for requests using the current name."
- Source: P-s1, P-s12. Basis: OR-documented.

**P3. The spec lists the `tools[].type` spellings each route accepts. The three routes differ.**

- `POST /chat/completions` (`ChatRequest.tools`, schema `ChatFunctionTool`): `function`; `openrouter:advisor`; `openrouter:bash`; `openrouter:datetime`; `openrouter:files`; `openrouter:fusion`; `openrouter:image_generation`; `openrouter:experimental__search_models`; `openrouter:subagent`; `openrouter:web_fetch`; `openrouter:web_search`; the shorthand `web_search`, `web_search_preview`, `web_search_preview_2025_03_11`, `web_search_2025_08_26` (schema `ChatWebSearchShorthand`, description "Web search tool using OpenAI Responses API syntax. Automatically converted to openrouter:web_search."); and a generic envelope `ChatDynamicServerTool` with `type` as a free string, description "Generic OpenRouter server-tool envelope. `type` names a registered server tool (canonical `openrouter:*` form or a registered shorthand); `parameters` carries tool-specific configuration validated against the tool definition."
- `POST /responses` (`ResponsesRequest.tools`): `function`; OpenAI's own types `web_search_preview`, `web_search_preview_2025_03_11`, `web_search`, `web_search_2025_08_26`, `file_search`, `computer_use_preview`, `code_interpreter`, `mcp`, `image_generation`, `local_shell`, `shell`, `apply_patch`, `custom`, `namespace`; and `openrouter:advisor`, `openrouter:subagent`, `openrouter:datetime`, `openrouter:files`, `openrouter:fusion`, `openrouter:image_generation`, `openrouter:experimental__search_models`, `openrouter:web_fetch`, `openrouter:web_search`, `openrouter:apply_patch`, `openrouter:bash`, `openrouter:shell`, `openrouter:tool_search`, `openrouter:mcp`.
- `POST /messages` (`MessagesRequest.tools`): Anthropic's own types `custom`, `bash_20250124`, `text_editor_20250124`, `web_search_20250305`, `web_search_20260209`, `advisor_20260301`, `tool_search_tool_bm25_20251119`, `tool_search_tool_bm25`, `tool_search_tool_regex_20251119`, `tool_search_tool_regex`; and `openrouter:bash`, `openrouter:datetime`, `openrouter:image_generation`, `openrouter:experimental__search_models`, `openrouter:web_fetch`, `openrouter:web_search`, `openrouter:shell`, `openrouter:tool_search`; and one generic inline variant `{"additionalProperties": {}, "properties": {"type": {"type": "string"}}, "required": ["type"], "type": "object"}`.
- Source: P-s35 (I enumerated the unions with a script). Basis: OR-documented.

**P4. The guides state per-route limits that are narrower than the spec unions.**

- Apply patch: "The apply patch server tool is only available through the [Responses API]... It is not supported via the Chat Completions API." (P-s7)
- Shell: "The shell server tool is available through the [Responses API] and the [Messages API]. Requesting it on the Chat Completions API returns a `400` error." (P-s5)
- Bash: "`openrouter:bash` is only available on the Anthropic Messages API. Requesting it on the Chat Completions or Responses API returns a `400` error." (P-s6)
- Tool search: "The `openrouter:tool_search` tool type is accepted by the [Responses API] and the [Messages API]. The Chat Completions API does not accept it in `tools` and returns a `400`; on Chat Completions, use the [`deferred_tools` protocol] instead." (P-s4)
- Fusion: "Fusion in the `tools` array also works on `/chat/completions`, but it's slower there than on the [Responses API]". (P-s10)
- Advisor: "the tool is available on the Chat Completions, Responses, and Anthropic Messages APIs". (P-s8)
- Subagent function inheritance: "Supported on the Responses API (`/api/v1/responses`) only; other APIs reject it with a `400`." (P-s9)
- Basis: OR-documented. See "Conflicts" for the places where the spec union and the guide disagree (`openrouter:bash` appears in the chat and Responses unions).

**P5. The catalog of server tools is an API of its own, and it needs a key.**

- `GET https://openrouter.ai/api/v1/tools` with no credentials answered HTTP 401 with body `{"error":{"message":"No cookie auth credentials found","code":401}}` on 2026-11-22. The spec marks `/tools` and `/tools/{name}` with `security: [{"bearer": []}]`. I stopped there, as the task said.
- The response schema (`ServerTool`) shows what the catalog would give: `id` ("Canonical `openrouter:*` name; the stable `tools[].type` on every API format in `supported_api_formats`"), `aliases` ("Every spelling accepted in `tools[].type` for this tool, including the canonical `id`, and what the call surfaces as under each"), `supported_api_formats` (enum `responses`, `chat-completions`, `anthropic-messages`), `default_engine`, `fallback_engine`, `engines[]` with `executed_by` (enum `provider`, `openrouter`, `client`), `data_regions`, `byok`, `pricing_source` (enum `openrouter`, `provider`, `byok`, `none`), and `native_support` with `endpoint_count` and `model_count`.
- The changelog dates the two endpoints: "`GET /tools`... List server tools" under September 29, 2026.
- Source: P-s36, P-s35, P-s31, P-s34. Basis: OR-documented.

**P6. The Responses output union names five `openrouter:*` item types that appear only on the output side, and the guides leave them out.**

- `OutputItems` includes `openrouter:code_interpreter` (fields `code`, `exitCode`, `language`, `stderr`, `stdout`), `openrouter:file_search` (`queries`), `openrouter:browser_use` (`action`, `screenshotB64`), `openrouter:text_editor` (`command`, `filePath`), and `openrouter:memory` (`action`, `key`, `value`). None of these five types appears in any of the three request `tools` unions, and `https://openrouter.ai/docs/guides/features/server-tools/code-interpreter.md` and `.../memory.md` returned 404 on 2026-11-22.
- Source: P-s35 and two live 404 responses. Basis: OR-documented (for the schema). What produces these items is in "Open questions".

**P7. Pricing differs per tool and per engine.**

- Web search: "**Exa** | Instant/Fast/Auto: \$0.007 per request; Deep Lite/Deep: \$0.012; Deep Reasoning: \$0.015. Includes up to 10 results, then \$0.001 per additional result", "**Parallel** | Turbo or Fast: \$0.001/request; Basic or Advanced: \$0.005/request", "**Perplexity** | \$0.005 per request using OpenRouter credits", "**Firecrawl** | $0.005 per search plus $0.0125 per returned result", "**Native** | Passed through from the provider". (P-s2)
- Web fetch: "**Exa** | \$1 per 1,000 fetches", "**Parallel** | \$1 per 1,000 fetches", "**Firecrawl** | \$2.50 per 1,000 fetches", "**OpenRouter** | Free", "**Native** | Passed through from the provider". (P-s3)
- Shell and bash sandbox: "Sandbox time (commands run with `engine: "openrouter"`) is billed at **\$0.0001 per second**." and "A request that starts a new or sleeping container is billed a minimum of 30 seconds." (P-s5, P-s6)
- Datetime, apply patch, search models, files: "no additional cost beyond standard token usage" or the same in other words. (P-s13, P-s7, P-s12, P-s14)
- Image generation: "Image generation pricing depends on the underlying model used". (P-s11)
- All of it lands in one number: "Server tool usage is included in the request's overall `usage.cost`". (P-s1)
- Basis: OR-documented.

**P8. What each tool hands back to the model.**

- Datetime: `{"datetime": "2025-07-15T14:30:00.000-04:00", "timezone": "America/New_York"}`. (P-s13)
- Web fetch: `url`, `title`, `content`, `status`, `retrieved_at`, or `status: "failed"` with `error`. (P-s3)
- Web search: "The search results (URLs, titles, and content snippets) are returned to the model." (P-s2)
- Image generation: to the model `{"status": "ok", "imageUrl": "data:image/png;base64,..."}`; to the caller an output item of type `openrouter:image_generation` with `result`, `imageUrl`, `prompt`. Default model `openai/gpt-image-2.6-flare`. (P-s11)
- Advisor: `{"status": "ok", "model": ..., "advice": ...}`. Subagent: `{"status": "ok", "model": ..., "task_name": ..., "outcome": ...}`. Fusion: `status`, `analysis` (`consensus`, `contradictions`, `partial_coverage`, `unique_insights`, `blind_spots`), `responses`, and on hard failure a `failure_reason`. (P-s8, P-s9, P-s10)
- Search models: `models[]`, `total_results`, `showing`. (P-s12)
- Shell: `output[]` entries with `stdout`, `stderr`, `outcome` (`{"type": "exit", "exit_code": <int>}` or `{"type": "timeout"}`), plus `container_id` and a `files` array of `container_file_citation` entries. Bash: `command`, `stdout`, `stderr`, `exitCode`. (P-s5, P-s6)
- Files: "The tool returns `{ "result": ... }` on success... or `{ "error": "..." }` when the operation fails". (P-s14)
- Apply patch: nothing is executed. "This is a **human-in-the-loop** tool: OpenRouter validates the diff but never applies it." The caller gets an `apply_patch_call` item and answers with `apply_patch_call_output`. (P-s7)
- Basis: OR-documented.

**P9. Every request with server tools runs an agent loop inside OpenRouter with a step budget.**

- Quote: "Every request that uses server tools runs an agent loop with a step budget. Each tool call the model makes (a web search, an image generation, etc.) consumes one step; when the budget is exhausted, the model is asked to produce its final answer with the context gathered so far."
- Quote: "`max_tool_calls` | `30` | `30` | Total server-tool steps allowed for the request, across all server tools" and "`stop_server_tools_when` | None | None | Array of stop conditions (step count, spend cap, and more). When set, it **overrides** `max_tool_calls`".
- Spec: `stop_server_tools_when` is a property of all three request schemas. `max_tool_calls` is a property of `ResponsesRequest` only. The stop condition types are `step_count_is`, `has_tool_call`, `max_tokens_used`, `max_cost`, `finish_reason_is`. The schema description adds: "When a condition fires while the model is still emitting tool calls, the pending tool calls are executed and one final turn is made with tool calls disabled so the response ends with a natural-language answer instead of an unfinished tool call."
- Inner loops have their own budgets: Fusion `max_tool_calls` default `4`, max `16`; Subagent max `25`.
- Source: P-s1, P-s35. Basis: OR-documented.

### B. Native execution versus OpenRouter's own engines

**P10. "Native" means OpenRouter asks the model's provider to run the provider's own tool.**

- Quote: "Some providers have a built-in version of a server tool. For example, OpenAI and Anthropic (as well as other providers) have built-in web search tools. When the model's endpoint has a "native" version of a tool available, OpenRouter can request the tool to be executed through the provider instead of running its own engine. This is the `native` engine in the server tool configuration."
- Execution paths, quoted: "**Provider**: the model's provider runs the tool natively.", "**OpenRouter**: OpenRouter runs the tool in-process, in an OpenRouter-operated sandbox, in an inner model generation, or by calling a third-party service such as Exa, Parallel, Perplexity, or Firecrawl.", "**Client**: OpenRouter returns the tool call to your application, which runs it."
- Source: P-s1. Basis: OR-documented.

**P11. Each endpoint advertises its native tools in a `native_tools` map, keyed by the OpenRouter tool name.**

- Quote: "Each endpoint object returned by the [endpoints API] has a `native_tools` field. The key is the canonical OpenRouter server tool name... The value is the name of the provider's tool."
- Doc example: `"native_tools": {"openrouter:web_search": { "type": "web_search_20260209" }, "openrouter:web_fetch": { "type": "web_fetch_20260209" }, "openrouter:bash": { "type": "bash_20250124" }}`.
- Spec, `NativeTools.type`: "The provider tool type the request is translated to when this tool runs natively, e.g. `web_search_20260209` on Anthropic or `google_search` on Gemini."
- Changelog: "`PublicEndpoint`: example modified; new property `native_tools`" under September 28, 2026.
- Source: P-s1, P-s35, P-s34. Basis: OR-documented.

**P12. A live read of the public endpoints listing for 27 models shows which provider endpoints run which tool natively.**

- I fetched `GET https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints` with no credentials for 27 recent models on 2026-11-22 (all HTTP 200) and tabulated `native_tools`:
  - Anthropic models (`anthropic/claude-opus-5.5`, `claude-sonnet-5.5`, `claude-haiku-5.5`): `openrouter:web_search -> web_search_20260209`, `openrouter:web_fetch -> web_fetch_20260209`, `openrouter:advisor -> advisor_20260301` on the endpoints named "Anthropic" and "Claude Platform on AWS" only. `openrouter:bash -> bash_20250124` on "Amazon Bedrock", "Anthropic", "Azure", "Claude Platform on AWS", "Google". The Bedrock, Azure, and Google endpoints of the same models list no native web search, web fetch, or advisor.
  - OpenAI models (`openai/gpt-6.2-astra`, `gpt-6.2-sol`, and others): `openrouter:web_search -> web_search` and `openrouter:apply_patch -> apply_patch` on "OpenAI" and "Azure". The "Amazon Bedrock" endpoint of the same model has an empty map.
  - Google models (`google/gemini-4-carbon`, `gemini-4-hydrogen`, `gemini-4-neon`): `openrouter:web_search -> google_search` on "Google" and "Google AI Studio".
  - xAI models (`x-ai/grok-4.5`, `4.6`, `4.7`): `openrouter:web_search -> web_search` on "xAI"; the "Amazon Bedrock" endpoint of `grok-4.7` has an empty map.
  - Perplexity (`perplexity/sonar-pro-search`, `sonar-reasoning-pro`), Mistral, DeepSeek, Moonshot, Qwen, MiniMax, Cohere, Z.AI, Amazon, Meta samples: empty map on every endpoint.
- No endpoint in the sample lists a native code execution, code interpreter, file search, computer use, MCP, memory, URL context, or tool search entry.
- Source: P-s37 (raw JSON saved in scratch). Basis: OR-documented (OpenRouter's public API output). Confidence: the sample is 27 of 476 models, picked as the newest per vendor.

**P13. Web search picks its engine from one field, `parameters.engine`, with `auto` as the default.**

- Quote: "`engine` | string | `auto` | Search engine to use: `auto`, `native`, `exa`, `firecrawl`, `parallel`, or `perplexity`".
- Quote: "**`auto`** (default): Uses native search if the provider supports it, otherwise falls back to Exa" and "**`native`**: Prefers the provider's built-in web search; falls back to Exa if the model doesn't support native search."
- Native providers, quoted with link targets removed: "**[OpenAI]**: GPT-4.1, GPT-4.1 Mini, GPT-4.1 Nano, GPT-5 and later, o3, o3 Pro, o4-mini", "**[Anthropic]**: Claude 3.5 Haiku, Claude 3.7 Sonnet, Claude 4 and later (all Opus/Sonnet variants)", "**[Google]**: Gemini 3 Flash, Gemini 3 Pro, Gemini 3.1 Flash/Lite, Gemini 3.5 Flash", "**[SpaceXSI]**: Grok 4 and later (web search; X search requires the [`x_search` opt-in])", "**[Perplexity]**: all Perplexity models (search is core to their API)". The docs call xAI "SpaceXSI" throughout.
- Quote: "Older OpenAI models (including GPT-4o, GPT-4o Mini, and GPT-4 Turbo) do **not** support native web search. If you set `engine: "native"` with these models, the server tool will fall back to Exa search."
- Source: P-s2. Basis: OR-documented.

**P14. With the native engine, most neutral web search settings are dropped or behave differently per vendor.**

- `max_results`: "Applies to Exa, Firecrawl, Parallel, and Perplexity engines; ignored with native provider search".
- `max_uses`: "With native provider search, forwarded only to Anthropic (as `max_uses`); other native search providers ignore it".
- `search_context_size` and `max_characters`: "Ignored with native provider search and Firecrawl."
- `user_location`: "Currently only supported by native provider search; ignored with Exa, Firecrawl, Parallel, and Perplexity".
- Domain filters, from the table: "**Native (Anthropic)** | Yes | Yes | Mutually exclusive", "**Native (OpenAI)** | Yes | No | `excluded_domains` ignored (no error)", "**Native (Google)** | No | No | Not supported. With `engine: "auto"`, falls back to Exa when filters are set. With `engine: "native"`, returns a 400 error", "**Native (SpaceXSI)** | Yes | Yes | Mutually exclusive".
- Source: P-s2. Basis: OR-documented.

**P15. X search is an xAI-only switch that rides on the web search tool.**

- Quote: "On SpaceXSI models, native web search sends only xAI's `web_search` tool. To also search X/Twitter, add an `x_search` object. An empty object enables X search with no filters".
- Quote: "`x_search` | object | N/A | Opt in to X/Twitter search on SpaceXSI models, with optional filters. Only used with native provider search on SpaceXSI; ignored elsewhere".
- Quote: "The legacy top-level `x_search_filter` request field still enables X search for backward compatibility."
- Source: P-s2. Basis: OR-documented.

**P16. Web fetch has the same engine field with a different fallback.**

- Quote: "`engine` | string | `auto` | Fetch engine to use: `auto`, `native`, `exa`, `openrouter`, `firecrawl`, or `parallel`".
- Quote: "**`auto`** (default): Uses native fetch if the provider supports it, otherwise falls back to Exa" and "**`native`**: Prefers the provider's built-in web fetch; falls back to the `openrouter` engine if the model doesn't support native fetch."
- Quote: "**OpenRouter/native engines**: Hard limit of 50 fetches per request".
- The web fetch tool uses `blocked_domains` where web search uses `excluded_domains`.
- Source: P-s3. Basis: OR-documented.

**P17. Shell, bash, apply patch, and advisor each have their own meaning of "native".**

- Shell: "`engine` | string | `auto` | Which shell engine to use: `openrouter` runs commands server-side in the OpenRouter sandbox; `auto` keeps the provider's native hosted shell when available (OpenAI) and routes to the OpenRouter sandbox on other providers". Also: "On the Responses API you can also send OpenAI's native tool shape (`{ "type": "shell" }`, or the legacy Codex `local_shell`) instead of `openrouter:shell`. On OpenAI models this uses OpenAI's own hosted shell; on any other model, OpenRouter routes the call to its sandbox transparently." (P-s5)
- Bash: "`auto` (default) / `native`: local, human-in-the-loop execution. The tool call is returned to your application, which runs the commands itself and sends the results back on the next request". And: "The native `bash_20250124` tool runs **client-side** by default: OpenRouter returns the `tool_use` to your application to execute locally, exactly as a direct call to the provider would." (P-s6)
- Apply patch: "`auto`: uses native passthrough when the endpoint supports incremental diff streaming, otherwise falls back to OpenRouter's HITL validator." and "Currently supported on OpenAI endpoints." (P-s7)
- Advisor: the matrix says "Provider (native passthrough) or OpenRouter inner generation". On the Messages route "request the advisor with the native Anthropic tool shape, and it works with **any executor model**, not just Anthropic ones", with type `advisor_20260301`. (P-s1, P-s8)
- Basis: OR-documented.

**P18. Region, zero-data-retention, and workspace settings can change or forbid the engine.**

- Quote: "With `engine: "auto"` or no engine, OpenRouter selects only engines resident in that region. A pinned engine that is not resident in the region is rejected." and "When no resident engine exists, the request fails. A server tool returns a `403` error, and the Web Search plugin returns a `400` error. OpenRouter does not fall back to global infrastructure."
- Quote: "on `us.openrouter.ai`, `exa`, `auto`, and an omitted engine run on Exa. Every other engine, including `native`, is rejected. No engine is available on `eu.openrouter.ai`."
- Quote: "when ZDR is enforced... Web Search, Web Fetch, and the Web Search plugin reject the Firecrawl engine. No other tool backend is filtered by ZDR."
- Workspace lock: "A request that names an engine outside the list, or whose domain list has no overlap with the workspace's, is rejected with a `403`."
- Source: P-s1, P-s2. Basis: OR-documented.

**P19. On the Responses route the request schema accepts OpenAI's hosted tool types by their OpenAI names, and two of them need the caller's own OpenAI key.**

- `file_search`: "File search tool configuration. `vector_store_ids` names OpenAI vector stores, so it requires BYOK (your own OpenAI key); requests on OpenRouter credits get a 400."
- `code_interpreter`: "Code interpreter tool configuration. An existing container id or `container.file_ids` requires BYOK (your own OpenAI key); requests on OpenRouter credits must use `auto` with no `file_ids`."
- `computer_use_preview`: "Computer use preview tool configuration" (fields `display_height`, `display_width`, `environment`).
- `mcp`: "MCP (Model Context Protocol) tool configuration" (fields `server_label`, `server_url`, `connector_id`, `authorization`, `headers`, `allowed_tools`, `require_approval`, `server_description`).
- `image_generation`, `shell`, `local_shell`, `apply_patch`, `custom`, `namespace` are also in the union.
- Source: P-s35. Basis: OR-documented. The docs do not say what happens when one of these is sent to a non-OpenAI model, except for `shell` (P17) and `web_search` (P20).

**P20. OpenAI's `web_search` tool type is hoisted to the OpenRouter server tool on any model.**

- Quote: "If your application already provides the `web_search` tool (e.g. OpenAI's built-in web search tool type), OpenRouter automatically recognizes it and hoists it to the `openrouter:web_search` server tool... As long as your application exposes the `web_search` tool, web search still works as a server tool with any model on OpenRouter."
- Source: P-s32. Basis: OR-documented.

**P21. On the Messages route, Anthropic's computer use tool is refused, and the schema names only six Anthropic tool types.**

- Quote: "OpenRouter's Messages API does not accept either computer tool form, so computer-use requests through OpenRouter are rejected at validation. If you call Anthropic directly, Opus 5.5 accepts only the `computer_toolset_20260801` toolset and rejects `computer_20251124`."
- The named Anthropic types in `MessagesRequest.tools` are `custom`, `bash_20250124`, `text_editor_20250124`, `web_search_20250305`, `web_search_20260209`, `advisor_20260301`, plus the four tool search spellings (P3). The names `code_execution_*`, `web_fetch_*`, `memory_*`, `computer_*`, and `mcp_toolset` are outside that list.
- `MessagesRequest` has these top-level properties and no others: `cache_control`, `context_management`, `deferred_tools`, `fallbacks`, `max_tokens`, `messages`, `metadata`, `model`, `models`, `output_config`, `plugins`, `provider`, `route`, `safeguards`, `service_tier`, `session_id`, `speed`, `stop_sequences`, `stop_server_tools_when`, `stream`, `system`, `temperature`, `thinking`, `tool_choice`, `tools`, `top_k`, `top_p`, `trace`, `user`. `mcp_servers` and `container` are outside that list. The only header parameter the spec lists for `POST /messages` is `X-OpenRouter-Metadata`.
- Source: P-s38 (quote), P-s35 (schema). Basis: OR-documented.

**P22. The Messages response schema does model Anthropic's code execution and web fetch blocks, so those blocks can come back even though the request union does not name the tools.**

- `ORAnthropicContentBlock` members: `text`, `tool_use`, `thinking`, `redacted_thinking`, `server_tool_use`, `web_search_tool_result`, `web_fetch_tool_result`, `code_execution_tool_result`, `bash_code_execution_tool_result`, `text_editor_code_execution_tool_result`, `tool_search_tool_result`, `container_upload`, `compaction`, `advisor_tool_result`, `openrouter_shell_tool_result`, `openrouter_bash_tool_result`, `openrouter_tool_search_result`. `BaseMessagesResult` has a `container` property.
- The Armor plugin schema says native code execution exists somewhere: `evaluate_native_tools`: "Run provider-native server tools (web search, code execution, ...) through the agent loop so Armor evaluates them too. Defaults to false: native passthrough stands and such calls are reported `skipped`."
- The deferred tools section names what it passes to a provider untouched: "Hosted tools the provider executes itself are also sent unchanged when their executed calls can be replayed with the conversation: `web_search` (every variant) and `code_interpreter`."
- Source: P-s35, P-s4. Basis: OR-documented. Confidence note: these three facts together suggest that Anthropic `code_execution_*` can be sent through the generic `{type: string}` variant on the Messages route and passed to an Anthropic endpoint. No OpenRouter page says so. That reading is an inference and is listed in "Open questions".

**P23. The three tool unions leave out Gemini code execution, URL context, and Maps grounding, and the only vendor tool the chat completions union names is web search.**

- From P3: the chat union holds `function`, the `openrouter:*` types, the four `web_search*` shorthands, and the generic envelope whose `type` "names a registered server tool". The Responses union holds OpenAI's types. The Messages union holds Anthropic's types. A search of the spec for `code_execution` finds only Anthropic schemas, and the only Gemini tool name in the spec is `google_search` inside the `NativeTools` description.
- Source: P-s35. Basis: OR-inferred (from fields missing in the spec and the docs; no page states that Gemini code execution or URL context is unreachable).

**P24. Anthropic's newest web search version is ahead of the one OpenRouter translates to.**

- Anthropic lists three versions: "`web_search_20250305`: basic web search", "`web_search_20260209`: adds [dynamic filtering]", "`web_search_20260318`: adds [response inclusion] control for agentic workflows". (P-s43)
- OpenRouter's Messages schema names `web_search_20250305` and `web_search_20260209`, and the live `native_tools` maps show `web_search_20260209` (P12). (P-s35, P-s37)
- Basis: Vendor-documented (Anthropic) for the first line, OR-documented for the second. The gap between them is an observation, with no OpenRouter statement about it.

### C. How server tool activity shows in the response and in the history

**P25. Usage has one aggregate object for server tools, and its name differs by route.**

- Quote: "Server tool usage is tracked in the response `usage.server_tool_use_details` object on the Chat Completions and Responses APIs".
- Quote: "The object is omitted when no server-side tool usage is recorded. The Messages API reports the same counters in Anthropic's `usage.server_tool_use` object, which is `null` instead of omitted when no server tool runs. Provider-native web search may still report `web_search_requests` without an OpenRouter server-tool invocation. The object reports aggregate activity for the request rather than a separate entry for every tool; `web_search_requests` is currently the only tool-specific counter."
- Fields: `tool_calls_requested` ("Provider-native tools are not included."), `tool_calls_executed`, `web_search_requests` ("do not add the two together").
- Spec: `ORAnthropicServerToolUsage` has `web_search_requests`, `web_fetch_requests`, `tool_calls_requested`, `tool_calls_executed`. Anthropic's own shape in the same spec (`AnthropicServerToolUsage`) has only the first two.
- Source: P-s1, P-s35. Basis: OR-documented.

**P26. On chat completions, an OpenRouter server tool call is recorded as an entry in `reasoning_details`, with its own type.**

- Spec, `ReasoningDetailServerToolCall`: `type` is `reasoning.server_tool_call`, required fields `tool_name`, `arguments`, `result`, optional `id`, `index`, `tool_call_id`, `format`. Description: "Record of an OpenRouter server-tool invocation (e.g. openrouter:fusion), carried in reasoning_details so a prior tool call can be rehydrated into a later turn of the same conversation."
- `ReasoningDetailUnion` maps four types: `reasoning.encrypted`, `reasoning.server_tool_call`, `reasoning.summary`, `reasoning.text`.
- The Tools API schema says the same from the other side. `ServerToolOutputName.type`: "Output item `type` on Responses, content block `type` on Anthropic Messages, `reasoning_details[].type` on Chat Completions". And `ServerToolInputName.output_names`: "What the call surfaces as on each accepting API format when requested under this spelling; a format is absent when the call is not visible to the caller".
- Source: P-s35. Basis: OR-documented. Confidence note: the guide examples show server tool output on the Responses and Messages routes only, so a chat completions response that holds such an entry is unseen, and the per-tool `output_names` values sit behind the authenticated Tools API, so which tools produce the entry (and which are "not visible to the caller") is unconfirmed.

**P27. The advisor is described differently: on chat completions it shows as `tool_calls` plus `role: "tool"` messages.**

- Three quotes, one per route: "**Responses API**: include the `openrouter:advisor` output items from prior responses in `input`, unchanged." Then "**Chat Completions**: include the assistant message's advisor `tool_calls` and the paired `role: "tool"` result messages from prior turns." Then "**Anthropic Messages API**: include the assistant message's advisor `server_tool_use` and `advisor_tool_result` content blocks from prior turns."
- Spec: `ChatToolCall.type` has one enum value, `function`. So an advisor call in `tool_calls` looks like a function call the caller never defined.
- Source: P-s8, P-s35. Basis: OR-documented.

**P28. On the Responses route each server tool call is a typed output item, and OpenRouter adds a synthetic reasoning item that binds the items of one server tool round.**

- The doc example for shell shows the `output` array as: a `message`, then `{type: "openrouter:shell", id: "st_tmp_2foofw1c4sf", status: "completed", action, output, container_id: "gen_453fe7911d9a", call_id: "toolu_01NiP9rEbn67fjNzu8Smb3QG", arguments: ...}`, then `{id: "rs_server_tool_round_71vxtx453q7", type: "reasoning", status: "completed", summary: [], encrypted_content: "openrouter-server-tool.eyJzZXJ2ZXJfdG9vbF9yb3VuZCI6..."}`, then a second `message`.
- I base64-decoded the part after `openrouter-server-tool.`. It reads: `{"server_tool_round":{"v":1,"members":[{"item_id":"msg_tmp_lk25kz4h0w"},{"item_id":"st_tmp_2foofw1c4sf","call_id":"toolu_01NiP9rEbn67fjNzu8Smb3QG"}]}}`. So this "encrypted" content is plain base64 JSON that lists the ids of the items in the round.
- Typed item names in `OutputItems`: `openrouter:datetime`, `openrouter:web_search`, `openrouter:image_generation`, `openrouter:bash`, `openrouter:apply_patch`, `openrouter:shell`, `openrouter:web_fetch`, `openrouter:tool_search`, `openrouter:experimental__search_models`, `openrouter:fusion`, `openrouter:advisor`, `openrouter:subagent`, `openrouter:files`, `openrouter:armor_blocked`, alongside OpenAI's own `web_search_call`, `file_search_call`, `image_generation_call`, `code_interpreter_call`, `computer_call`, `apply_patch_call`, `shell_call`, `shell_call_output`, `mcp_list_tools`, `mcp_call`, `mcp_approval_request`, `tool_search_call`.
- The item name depends on the spelling the caller used: "On the Responses API the call becomes an `openrouter:shell` output item (or a native `shell_call` when you send OpenAI's tool shape)."
- Source: P-s5, P-s35. Basis: OR-documented (the decode is mine). No page explains the `rs_server_tool_round_` item or says whether it must be sent back.

**P29. On the Messages route a server tool call is a `server_tool_use` block followed by a result block, and OpenRouter adds its own result block types for tools that Anthropic does not define.**

- Quote: "On the Messages API it becomes a `server_tool_use` content block named `openrouter:shell`, paired with an `openrouter_shell_tool_result` block carrying each command's output. Anthropic defines no native shell result block, so this OpenRouter-namespaced one is where the output arrives."
- The bash example shows `{type: "server_tool_use", id: "toolu_01L2udfHyfvhsNCgKnQnh1CK", name: "openrouter:bash", input}` then `{type: "openrouter_bash_tool_result", tool_use_id: ..., content, container_id: "gen_f297554427ca"}`. The id has the `toolu_` prefix, where Anthropic's own server tool ids start with `srvtoolu_` (P34).
- The advisor uses Anthropic's shapes: "a `server_tool_use` block with `name: "advisor"` for the call, followed by an `advisor_tool_result` block with the advice".
- Tool search answers "in kind": "Each API's native spelling is accepted as an alias and answered in kind". The spec has both `tool_search_tool_result` and `openrouter_tool_search_result` blocks.
- Source: P-s5, P-s6, P-s8, P-s4, P-s35. Basis: OR-documented.

**P30. Several server tools need the caller to send their items back unchanged on the next request.**

- Advisor memory: "the only requirement is that you **replay the advisor exchanges you received**" and "Advisor identity is positional, derived from the entry's index in the request `tools` array. Keep the order of advisor entries stable across the requests of a conversation (and echo the `instance_name` field on replayed Responses items unchanged)." (P-s8)
- Subagent with inherited client functions (Responses only): the caller must send back "The `openrouter:subagent` item, verbatim", "Every projected `function_call` item exactly as returned, with `subagent_id` and `subagent_items` intact, in the order you received them", "One `function_call_output` item per projected call", and "The same `tools` array". About `subagent_items`: "Treat it as opaque and replay it unchanged so the worker resumes with its context intact." A bad `subagent_id` "is rejected with a `400`". (P-s9)
- Apply patch: the caller answers with an `apply_patch_call_output` item whose `call_id` "Must match the `call_id` from the `apply_patch_call`". (P-s7)
- Sandbox continuity without a session id: "the container id is the most recently returned `container_id` in the replayed conversation, if there is one". (P-s20)
- Deferred tools: see P46.
- Gemini agentic video (a vendor-run step that is not a tool on the wire): "the model's internal video navigation steps are returned as encrypted reasoning rather than as tool calls" and on chat completions "send the assistant message back with its `reasoning_details` intact". (P-s25)
- Basis: OR-documented.

**P31. The Messages request schema accepts Anthropic's `server_tool_use` and `web_search_tool_result` blocks in history, with `encrypted_content` required on each result.**

- `MessagesMessageParam` content variants: `text`, `image`, `document`, `tool_use`, `tool_result`, `thinking`, `redacted_thinking`, `server_tool_use`, `web_search_tool_result`, `search_result`, `compaction`, `advisor_tool_result`, `tool_addition`, `tool_removal`, `openrouter_shell_tool_result`, `openrouter_bash_tool_result`, `openrouter_tool_search_result`.
- `AnthropicWebSearchResultBlockParam` requires `type`, `encrypted_content`, `title`, `url`.
- Six block types are in the response-side union (P22) and outside the request-side list: `web_fetch_tool_result`, `code_execution_tool_result`, `bash_code_execution_tool_result`, `text_editor_code_execution_tool_result`, `tool_search_tool_result`, and `container_upload`.
- `tool_result.content` may hold `{"type": "tool_reference", "tool_name": ...}` parts.
- Source: P-s35. Basis: OR-documented. Whether the server rejects a replayed `web_fetch_tool_result` block, or the schema is just behind, is unknown (see "Open questions").

**P32. The Messages stop reason enum includes `pause_turn`, and no OpenRouter page describes it.**

- `ORAnthropicStopReason`: `end_turn`, `max_tokens`, `model_context_window_exceeded`, `stop_sequence`, `tool_use`, `pause_turn`, `refusal`, `compaction`, `null`.
- A search of the mirrored guides and cookbook for `pause_turn` finds it only inside generated API reference schemas.
- Source: P-s35 and a content search of the 559 mirrored pages. Basis: OR-documented (the enum). How `pause_turn` maps to `finish_reason` on chat completions is in "Open questions".

**P33. Anthropic's native rule for web search results: the result blocks hold encrypted content that must come back exactly.**

- Quote: "`encrypted_content`: Encrypted content that you must pass back in multi-turn conversations".
- Quote: "To continue a conversation that contains search results, send the assistant’s content blocks back exactly as you received them, including each result’s `encrypted_content`. The API decrypts that content in later requests to restore the search results in Claude’s context. If `encrypted_content` is missing or modified, the request fails with a 400 validation error."
- Quote: "`encrypted_index`: A reference that must be passed back for multi-turn conversations" (on citations).
- Source: P-s43. Basis: Vendor-documented (Anthropic, read 2026-11-22).

**P34. Anthropic's native rule for server tool blocks and `pause_turn`.**

- Quote: "The `server_tool_use` block appears in Claude’s response when a server-executed tool runs. Its `id` field uses the `srvtoolu_` prefix to distinguish it from client tool calls".
- Quote: "Unlike client `tool_use` blocks, you don’t need to respond with a `tool_result`. The tool’s result block (for example, `web_search_tool_result` for web search) follows the `server_tool_use` block in the same assistant turn, paired by `tool_use_id`. If Claude calls one of your client tools at the same time, the `server_tool_use` block appears without its result, and the response ends with `stop_reason: "tool_use"`."
- Quote: "On a long-running turn, the API might pause that loop and return a `pause_turn` stop reason." and (web search page) "The API can pause a long-running search turn and return `stop_reason: "pause_turn"`. To continue, send the paused assistant message back unchanged in a new request."
- Source: P-s44, P-s43. Basis: Vendor-documented (Anthropic, read 2026-11-22). Also in workspace: the 2026-11-17 report on the Anthropic Messages API lists `pause_turn` as "A server tool loop paused".

**P35. OpenAI's native rule for web search output.**

- Quote: "Model responses that use the web search tool will include two parts". The first is "A `web_search_call` output item with the ID of the search call, along with the action taken in `web_search_call.action`." The second is a `message` output item with "The text result in `message.content[0].text`" and "Annotations `message.content[0].annotations` for the cited URLs".
- Quote: "When your interface displays web results or information from them, you must show inline citations as prominent, clickable links."
- Quote: "Chat Completions search models always search before responding; Responses search is a tool".
- Source: P-s50. Basis: Vendor-documented (OpenAI, read 2026-11-22).

**P36. Fill for the undocumented chat completions case: LiteLLM turns Anthropic server tool blocks into ordinary `tool_calls` and parks the result blocks in an extra message field.**

- At commit `90dce4b`, `litellm/llms/anthropic/chat/transformation.py` lines 2973 to 2991: `elif content["type"] == "tool_use" or content["type"] == "server_tool_use":` builds a normal OpenAI tool call; `elif content["type"].endswith("_tool_result"):` appends `web_search_tool_result` and `web_fetch_tool_result` blocks to `web_search_results` and every other result block to `tool_results`.
- Lines 3324 to 3353 put them in `provider_specific_fields`. The keys set there include `citations`, `thinking_blocks`, `web_search_results`, `web_search_calls`, `tool_results`, and `code_interpreter_results`.
- On the way back, `litellm/litellm_core_utils/prompt_templates/factory.py` lines 2377 to 2379 say: "For server-side tools (web_search), we need to reconstruct: - server_tool_use blocks (id starts with "srvtoolu_") - web_search_tool_result blocks (from provider_specific_fields)". Lines 3545 and 3561 to 3562 detect the `srvtoolu_` prefix and read `_provider_specific_fields.get("web_search_results")` and `.get("tool_results")`.
- Source: P-s42. Basis: Filled (LiteLLM). This shows one working design. It says nothing about what OpenRouter does. Also in workspace: the 2026-11-17 Anthropic report records a LiteLLM failure where rebuilt blocks came back in a different order and the API answered that the blocks "cannot be modified".

**P37. OpenRouter's own AI SDK provider keeps three reasoning detail types and discards any other type.**

- At commit `7a235a4`, `src/schemas/reasoning-details.ts` lines 5 to 8 define `Summary = 'reasoning.summary'`, `Encrypted = 'reasoning.encrypted'`, `Text = 'reasoning.text'`. Lines 58 to 65: `const ReasoningDetailsWithUnknownSchema = z.union([ReasoningDetailUnionSchema, z.unknown().transform(() => null)]);` and `ReasoningDetailArraySchema = z.array(ReasoningDetailsWithUnknownSchema).transform((d) => d.filter((d): d is ReasoningDetailUnion => !!d));`.
- A search of `src` (tests excluded) for `server_tool_call` finds nothing.
- The same file set maps AI SDK provider tools to server tools: `src/chat/index.ts` lines 1450 to 1462, "Provider tool IDs follow the format `openrouter.<tool_name>`, which maps to `openrouter:<tool_name>` in the API request tools array."
- Source: P-s41. Basis: OR-documented (official SDK source). Confidence note: that a `reasoning.server_tool_call` entry is therefore lost when this SDK replays a conversation is my inference from the filter; I did not trace every code path.

### D. Citations

**P38. On chat completions, web search citations are `annotations` on the assistant message, each a `url_citation` with a nested object.**

- Quote: "Web search results for all models (including native-only models like Perplexity and OpenAI Online) are available in the API and standardized by OpenRouter to follow the same annotation schema in the [OpenAI Chat Completion Message type]".
- Shape from the same page: `"annotations": [{"type": "url_citation", "url_citation": {"url": ..., "title": ..., "content": ..., "start_index": 100, "end_index": 200}}]`, with the comment on `content`: "Added by OpenRouter if available".
- Source: P-s16. Basis: OR-documented.

**P39. On the Responses route the same citation is flat, and an extra flag marks pages that were read and not quoted.**

- Quote: "Web search responses include citation annotations. A `url_citation` with `uncited: true` means the page was consulted rather than quoted in the answer."
- Shape: `{"type": "url_citation", "url": "https://example.com/article", "start_index": 0, "end_index": 50, "content": "Excerpt from the web page..."}` inside `output[].content[].annotations`.
- Spec `URLCitation`: required `type`, `url`, `title`, `start_index`, `end_index`; optional `content` and `uncited` (`const: true`). Changelog: "`URLCitation`: new property `uncited`" under November 13, 2026.
- `OpenAIResponsesAnnotation` is a union of `FileCitation` (`file_citation`: `file_id`, `filename`, `index`), `URLCitation`, and `FilePath` (`file_path`: `file_id`, `index`).
- Source: P-s30, P-s35, P-s34. Basis: OR-documented.

**P40. With the Exa engine, the `content` field of a citation holds the excerpts the model saw.**

- Quote: "The selected excerpts are returned to the model on each result and surfaced to API callers via `url_citation` annotations. Within a single result, excerpts that come from different parts of the page are separated by Exa's `[...]` markers".
- Source: P-s2. Basis: OR-documented.

**P41. The chat completions schemas in the spec leave `annotations` out, while OpenRouter's own SDK reads it from both the message and the stream delta.**

- `ChatAssistantMessage` properties: `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`. `ChatStreamDelta` properties: `audio`, `content`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`. Neither lists `annotations`. The only chat-shaped schema with `message.annotations` is the inline body inside `BatchObject`, with two variants, `file` and `url_citation`.
- OpenRouter's AI SDK provider at commit `7a235a4`, `src/chat/schemas.ts` lines 75 to 127, parses `annotations` as a union of `url_citation` (comment: "title, start_index, end_index are optional as some upstream providers may omit them"), `file_annotation` with `file_id` and `quote` (comment: "File annotation from FileParserPlugin (old format)"), and `file` with `hash`, `name`, `content` (comment: "File annotation from FileParserPlugin (new format)"). `src/chat/index.ts` line 1028 reads `delta.annotations` while streaming.
- Source: P-s35, P-s41. Basis: OR-documented. The mismatch is recorded under "Conflicts".

**P42. The deprecated web plugin tells the model to cite with Markdown links, so citations also sit in the text itself.**

- Default search prompt, quoted: "A web search was conducted on `date`. Incorporate the following web search results into your response. IMPORTANT: Cite them using markdown links named using the domain of the source. Example: [nytimes.com](https://nytimes.com/some-page)."
- Source: P-s16. Basis: OR-documented.

**P43. On the Messages route the schema keeps Anthropic's own citation objects.**

- `AnthropicTextBlock` has a `citations` property. The spec defines `AnthropicCitationCharLocation`, `AnthropicCitationPageLocation`, `AnthropicCitationContentBlockLocation`, `AnthropicCitationWebSearchResultLocation`, and `AnthropicCitationSearchResultLocation`. On the request side, `document` blocks and `search_result` blocks accept `citations` (`{"enabled": true}`).
- Source: P-s35. Basis: OR-documented.

**P44. File citations on the Responses route cannot be reused, and file annotations on chat completions can.**

- Quote: "The Responses API returns `file_citation` annotations instead, which can't be sent back to skip parsing".
- Quote: "The Responses API may include a `file_citation` annotation when the provider cites a parsed file in its output. Parsing a file alone does not guarantee a citation".
- Source: P-s22. Basis: OR-documented. The chat completions `file` annotation is covered in P66.

**P45. No OpenRouter page says how Anthropic document citations or Gemini search suggestions reach a chat completions caller. The vendors attach display duties to both.**

- A content search of the mirrored guides for `searchEntryPoint`, `groundingMetadata`, `search_suggestions`, `cited_text`, and `char_location` finds no guide text (hits are only inside generated schema dumps for the Messages route).
- Gemini's rule: "`google_search_result` : Contains `search_suggestions`, an HTML snippet for rendering search suggestions in your UI. Full usage requirements are detailed in the [Terms of Service]". The same page describes the answer as "`text` with `annotations`" where "Each `url_citation` annotation links a text segment (defined by `start_index` and `end_index`) to a source URL." (P-s52)
- OpenAI's rule is quoted in P35.
- Fill: LiteLLM at `90dce4b` stores Anthropic `citations` in `provider_specific_fields["citations"]` on the chat message (`transformation.py` lines 3009 to 3018 and 3324), outside the OpenAI `annotations` field. (P-s42)
- Basis: OR-inferred (from absence in the docs) for the first line, Vendor-documented (Google) for the second, Filled (LiteLLM) for the fourth.

### E. Deferred tool loading and tool search

**P46. A tool is marked deferred with `defer_loading`, and the flag's position differs by route.**

- Table, quoted: "Chat Completions | `tools[].function.defer_loading: true`", "Responses | `tools[].defer_loading: true` on a function tool", "Messages | `tools[].defer_loading: true`".
- What happens on an endpoint that defers natively: "Passed through: the provider withholds and expands the tool itself." Elsewhere on chat completions: "Withheld until you re-send the tool without `defer_loading`." On Responses: "or load it through a client `tool_search_output` input item". On Messages: "Withheld until a `tool_reference` block in a tool result names it, or you re-send it without `defer_loading`. The `tool_reference` block reaches the provider as text."
- Quote: "Endpoints that defer natively are Anthropic models on Anthropic, Vertex, Azure Foundry and Claude Platform on AWS, and Anthropic-compatible gateways that implement deferral. Everywhere else, revealed definitions are appended after your loaded tools in the order they were revealed, so the prompt prefix the provider caches does not move between turns."
- Quote: "Client-side deferral on endpoints without native deferral is rolling out per account. Until an account is admitted, Chat Completions and Responses send deferred tools in full with the flag removed, and Messages routes a deferred request only to endpoints that defer natively, returning a `400` when the model has none."
- Spec: `ChatFunctionTool` has `defer_loading` at both `tools[].defer_loading` and `tools[].function.defer_loading`.
- Source: P-s4, P-s35. Basis: OR-documented.

**P47. The `openrouter:tool_search` server tool is a regex search that OpenRouter runs, on Responses and Messages only.**

- Quote: "The model supplies a regular expression, matched case-insensitively against each deferred tool's name, description, argument names, and argument descriptions." and "Patterns are capped at 200 characters."
- Quote: "Tool search works on any model and any provider, not only those with native support for it."
- Accepted spellings: "Responses | `openrouter:tool_search`, `tool_search`" and "Messages | `openrouter:tool_search`, `tool_search_tool_regex`, `tool_search_tool_regex_20251119`".
- Quote: "Only the regex variant is implemented." The BM25 spellings return a `400` whose message begins "`tool_search_tool_bm25_20251119` is not supported yet. OpenRouter implements the regex tool-search variant only".
- Parameter: "`max_results` | `5` | Maximum tools returned by a single search. The model may request fewer. Capped at 50."
- Rule: "**`openrouter:tool_search` itself can never be deferred.**" Breaking it "fails the request with a `400` rather than quietly ignoring the deferral".
- Source: P-s4. Basis: OR-documented.

**P48. Where no search tool is declared, OpenRouter looks for one by name, may add its own, or refuses.**

- Quote: "On an endpoint without native deferral, at least one of your loaded tools must look like a search tool: a name containing `tool_search`, `tool-search`, `toolsearch`, `search_tools` or `find_tools`, or ending in `_search` or `-search`; an Anthropic `tool_search_tool_regex` or `tool_search_tool_bm25` tool; or `openrouter:tool_search`. This is a name check, not a check that the tool searches. Without such a tool, OpenRouter adds `openrouter:tool_search` to a Chat Completions or Responses request where managed search is enabled for it; otherwise the request fails with a `400` that names what was looked for".
- Error text: "At least one tool must have defer\_loading=false and be able to reveal the others. OpenRouter looked for a non-deferred tool named like a search tool …"
- Source: P-s4. Basis: OR-documented.

**P49. Deferral takes over `tool_choice`.**

- Quote: "If you set anything else, it must be `{"type": "allowed_tools", ...}` naming the tools that should be callable immediately. Deferred tools are added to that set as the model finds them."
- Error text: "`tool_choice` conflicts with `openrouter:tool_search`. Deferred tools are revealed through `tool_choice`, so it must be omitted or set to `{"type": "allowed_tools", ...}`. Remove `tool_choice`, or drop `defer_loading` from your tools to use it as-is."
- Source: P-s4. Basis: OR-documented.

**P50. `deferred_tools` is a separate, versioned request field on all three routes.**

- Quote: "The `deferred_tools` request field opts a conversation into OpenRouter's versioned deferred-tool protocol. It works on the Chat Completions, Responses, and Messages APIs. Every field is required".
- Doc example: `{"protocol": "v1", "profile": "portable", "validation": "runtime", "search": {"type": "regex", "max_results": 5}}`.
- Doc values: `profile`: "`"portable"` is the only profile in this release". `validation`: "`"runtime"` is the only mode in this release: OpenRouter validates each call against the original tool schema before it reaches you." `search`: "`{ "type": "regex" }` or `{ "type": "bm25" }` searches your deferred tools inside OpenRouter. `{ "type": "custom", "tool_id": "...", "result_key": "..." }` uses one of your non-deferred function tools as the search: your client runs it and returns matching tool names under `result_key` in its result. `max_results` defaults to `5` and accepts `1` to `100`."
- Spec `DeferredToolsControl` has more than the guide: `profile` enum `portable`, `universal` ("`universal` defers every wrappable tool behind the same wrappers so the provider-facing tool list never changes... and rebuilds the catalog from each request so tools can be added or removed mid-conversation."), `interface` enum `wrappers`, `code_mode` ("`wrappers`: the `tool_search` and `tool_exec` tools. `code_mode`: one `tool_code` tool that runs a short JavaScript program in which `search()` and `call()` reach the catalog... Experimental."), `scope` enum `all`, `mcp`, `cascade_mode` enum `keywords`, `native`, `hint`, `min_deferred_tools`, `announce_catalog_changes`.
- Usage: `ChatUsage.deferred_tools_details` with `turns`, `prompt_tokens`, `completion_tokens`, `total_tokens`; description "Totals across every upstream model call of a deferred-tools run. The native usage fields report only the final call; cost covers every call."
- Source: P-s4, P-s35. Basis: OR-documented. The guide and the spec disagree on the profile list; see "Conflicts".

**P51. The protocol's state travels inside the encrypted reasoning that the caller must return.**

- Quote: "OpenRouter carries each turn's deferred-tool changes inside the encrypted reasoning it returns: a `reasoning_details` entry on Chat Completions, a reasoning item's `encrypted_content` on Responses, and a `redacted_thinking` block on Messages. Each turn records only what changed, such as the ID of a tool it loaded. There is no separate state field."
- Quote: "**Send assistant messages back exactly as returned**, including the encrypted reasoning described above. Each turn's state is chained to the turns before it, so editing, removing, reordering, trimming, or compacting earlier messages breaks the chain."
- Quote: "**Keep the tool catalog fixed.** Send the same `tools`, `tool_choice`, and `deferred_tools` settings on every turn. Tool order, descriptions, and `cache_control` are part of the catalog, so changing any of them requires a new conversation."
- Quote: "**Context compression is off.** OpenRouter does not apply `middle-out` or other context compression to deferred-tool requests, so a conversation that outgrows the model's context window fails instead of being trimmed." and "**The Batch API is not supported.**"
- Quote: "an assistant message that contains only text and carries no state is accepted as ordinary conversation content... An assistant message with tool calls but no state is rejected."
- Error codes (in `error.metadata.code` on chat completions and Responses, `metadata.code` on Messages): `deferred_tools_state_missing`, `deferred_tools_state_invalid` ("it was modified, or was issued for a different workspace"), `deferred_tools_history_changed`, `deferred_tools_catalog_changed`.
- Source: P-s4. Basis: OR-documented.

**P52. The guide names three clients and what breaks in each.**

- Quote: "**Claude Code** replays encrypted reasoning as-is within a session. Resuming with `--continue` after a tool-using turn is not supported: the resumed transcript drops the `redacted_thinking` blocks, so the next turn returns `deferred_tools_state_missing`".
- Quote: "**pi** needs pi-ai `0.84.3` or later on Chat Completions. Earlier versions drop the encrypted `reasoning_details` entry. Keep the same provider, API, and model for the whole conversation: switching models makes pi strip reasoning signatures, and the next turn returns a `400`."
- Source: P-s4. Basis: OR-documented.

**P53. With `deferred_tools`, vendor-hosted tools are sorted into three groups.**

- Quote: "Hosted tools the provider executes itself are also sent unchanged when their executed calls can be replayed with the conversation: `web_search` (every variant) and `code_interpreter`. They are never deferred or loaded... Other hosted tool types (`file_search`, `mcp`, `image_generation`, `computer_use_preview`, shells) return a `400` with `deferred_tools` until their replayed items are supported."
- Quote: "Freeform (custom) tools that are not deferred, such as Codex `apply_patch`, are sent to the model unchanged and their calls are passed back without validation."
- Source: P-s4. Basis: OR-documented.

**P54. The Messages schema also accepts Anthropic's blocks for adding and removing tools mid-conversation.**

- `MessagesToolAdditionBlock`, type `tool_addition`: "Adds a tool mid-conversation without invalidating the prompt cache: either a previously deferred tool (declared in `tools` with `defer_loading: true`, referenced by name) or a full inline `tool_definition`. Only valid in `role: "system"` messages. Not supported on Claude Sonnet 5 or models older than Claude Opus 4.8." Its `tool` is one of `tool_reference`, `mcp_tool_reference`, `mcp_toolset_reference`, `tool_definition`. `MessagesToolRemovalBlock` has type `tool_removal`.
- Source: P-s35. Basis: OR-documented.

**P55. The vendors' native rules for tool search.**

- Anthropic: "You provide every tool definition in the `tools` array and set `defer_loading: true` on the tools that shouldn’t load up front. At least one tool, normally the tool search tool itself, must stay non-deferred." And: "On the next request, pass the assistant’s content back unchanged, including the `server_tool_use` and `tool_search_tool_result` blocks... Don’t return a `tool_result` for the `srvtoolu_...` ID: the API rejects the request." (P-s45)
- OpenAI: "When new tools are discovered by the model, they are injected at the end of the context window." And: "mark the ones you want to defer with `defer_loading: true`. If you are using [MCP servers], set `defer_loading: true` on the MCP server tool definition." (P-s51)
- Basis: Vendor-documented (Anthropic and OpenAI, read 2026-11-22). Also in workspace: `runvil-design`, `research/2026-10-14-harness-mcp-dynamic-tools.md` (2026-10-14) gives the same rules; the live pages confirm them.

**P56. Dates from the changelog.**

- October 7, 2026: new schemas `ToolSearchCallItem`, `ToolSearchOutputItem`, `OutputToolSearchCallItem`.
- October 10, 2026: new schemas `DeferredToolsControl`, `DeferredToolsSearchStrategy`; "`ChatRequest`: new property `deferred_tools`", the same for `MessagesRequest` and `ResponsesRequest`; "`CustomTool`: new property `defer_loading`".
- November 5, 2026: "`ChatUsage`: new property `deferred_tools_details`"; `DeferredToolsControl` "enum property\_added".
- November 6 to 8, 2026: `DeferredToolsControl` gains `interface`, `scope`, `cascade_mode`, `hint`, `min_deferred_tools`, `announce_catalog_changes`.
- November 9, 2026 (breaking): "`deferred_tools.scope` now accepts only `"all"` (the default when omitted) or `"mcp"`; requests or saved presets sending `"scope": "client"` now return 400."
- Source: P-s34. Basis: OR-documented.

### F. Plugins

**P57. A plugin always runs once per request and changes the request or the response. The guide lists seven.**

- Quote: "OpenRouter plugins extend the capabilities of any model by injecting or mutating a request or response to add functionality like PDF processing, automatic JSON repair, and context compression. Unlike [server tools] (which the model can call 0-N times), plugins always run once when enabled."
- Guide table: "**Web Search** (deprecated)", "**PDF Inputs**", "**Response Healing**", "**Alignment** (beta)", "**Armor** (beta)", "**Pareto Router**", "**Context Compression**".
- Quote: "Each plugin is identified by its `id` and can include optional configuration parameters." The `plugins` array is a property of all three request schemas.
- Source: P-s15, P-s35. Basis: OR-documented.

**P58. The spec's `plugins` union has fourteen ids, twice what the guide lists.**

- Discriminator values for `id`: `alignment`, `armor`, `auto-beta-router`, `auto-router`, `context-compression`, `file-parser`, `fusion`, `jev-router`, `moderation`, `pareto-router`, `response-healing`, `switchyard-router`, `web`, `web-fetch`.
- Settings per plugin, from the schemas:
  - `web`: `enabled`, `engine` (enum `native`, `exa`, `firecrawl`, `parallel`, `perplexity`), `max_results`, `max_uses`, `mode`, `search_prompt`, `include_domains`, `exclude_domains`, `user_location`, `x_search`.
  - `web-fetch`: `allowed_domains`, `blocked_domains`, `max_content_tokens`, `max_uses`.
  - `file-parser`: `enabled`, `pdf.engine` (`mistral-ocr`, `native`, `cloudflare-ai`, and the deprecated `pdf-text`).
  - `response-healing`: `enabled`.
  - `context-compression`: `enabled`, `engine` (enum with one value, `middle-out`).
  - `moderation`: `id` only.
  - `fusion`: `enabled`, `preset` (enum `general-high`, `general-budget`, `general-fast`), `analysis_models`, `model`, `max_tool_calls`, `tools`.
  - `auto-router` and `auto-beta-router`: `allowed_models`, `excluded_models`, `cost_tier`, `cost_quality_tradeoff` (deprecated), `enabled`, and on `auto-router` `pin_model`.
  - `pareto-router`: `min_coding_score`, `max_price`, `price_source`, `enabled`. `jev-router`: `models`, `allowed_models`, `excluded_models`, `cost_tier`. `switchyard-router`: `algorithm`, `judge_model`.
  - `armor`: `classifier_context`, `enabled`, `evaluate_native_tools`, `mode` (enum `enforce`, `audit`), `on_error`, `use_cc_style_default_rules`.
  - `alignment`: `rules` (required), `mode` (enum `enforce`, `audit`), `threshold` ("Default 0.7"), `max_retries` ("Default 1"), `instruct` ("Whether the rules are stated to the model in a system message on every provider call. Default true.").
- Source: P-s35. Basis: OR-documented.

**P59. Plugins can be switched on for a whole account, and an admin can stop a request from changing them.**

- Quote: "Plugin settings are applied in the following order of precedence". First: "**Request-level settings**: Plugin configurations in the `plugins` array of individual requests". Second: "**Account defaults**: Settings configured in the Plugins settings page".
- Quote: "When "Prevent overrides" is enabled for a plugin, individual API requests cannot disable or modify that plugin's configuration."
- Quote: "If a plugin is enabled by default in your account settings, you can disable it for a specific request by passing `"enabled": false` in the plugins array".
- Quote: "Settings on the workspace's Plugins tab do not apply to server tools." (from the web search server tool page)
- Source: P-s15, P-s2. Basis: OR-documented.

**P60. The web plugin and the `:online` model suffix are deprecated in favour of the server tool, and the plugin's `native` engine behaves differently from the tool's.**

- Quote: "The Web Search plugin (`plugins: [{ "id": "web" }]`) is deprecated. Use the `openrouter:web_search` server tool instead." (P-s1)
- Quote: "`:online` is a shortcut for using the `web` plugin, and is exactly equivalent to:" followed by a block that reads `"model": "openrouter/auto"` and `"plugins": [{ "id": "web" }]`. Two pages print `openrouter/auto` as the model in the equivalent form; the plugins overview page prints the original model slug (`openai/gpt-5.2`) in the same place. (P-s16, P-s32, P-s15)
- Difference table: plugin "Always searches once", "Once per request"; server tool "Model decides when/whether to search", "0 to N times per request". (P-s2)
- Plugin engine `native`: "Uses the model provider's built-in web search. This path substitutes no other engine, so a model without native search runs its request without a search rather than falling back to Exa". (P-s16) The server tool's `native` falls back to Exa (P13).
- Context size on the plugin path: "You can specify the search context size in your API request using the `web_search_options` parameter", example `"web_search_options": {"search_context_size": "high"}`. (P-s16)
- When both are sent: "the server tool's parameters override the matching plugin settings (`allowed_domains`, `excluded_domains`, `max_results`, `user_location`, `max_uses`, `x_search`), and plugin settings the tool does not set, such as `search_prompt`, are kept." (P-s2)
- Basis: OR-documented.

**P61. Alignment checks assistant messages, and Armor checks tool calls. Both add their own objects to the response.**

- Alignment: "The Alignment plugin checks every assistant message of a request against rules you send with the request." and "In `audit` mode, OpenRouter delivers each message with its outcome." The page header says "Beta since October 2026". The response gains an `alignment` object with `calls[].outcome` (`allowed` or `blocked`) and per-rule `probability`. (P-s17)
- Armor: "Armor is a classifier that runs between the model and your application and evaluates every tool call the model makes." On a block: "The response has no `function_call`; in its place is an `openrouter:armor_blocked` item carrying the call and the rule that fired, followed by the model's own text, and `safeguard_results` records the verdict under the call's id". On a pass: "The only visible difference is a `safeguard_results` field in the response that records the verdict." (P-s18)
- Armor also answers Anthropic's `safeguards` request field: "Armor reads Claude Code's `safeguards` entry and answers in the format Claude Code expects from Anthropic's own implementation, so auto mode works on any model OpenRouter serves". (P-s18)
- Basis: OR-documented.

**P62. Context compression is middle-out truncation, on by default for small-context endpoints.**

- Quote: "The plugin works by removing or truncating messages from the middle of the prompt, until the prompt fits within the model's context window."
- Quote: "[All OpenRouter endpoints] with 8k (8,192 tokens) or less context length will default to using context compression. To disable this, pass `plugins: [{"id": "context-compression", "enabled": false}]` in the request body."
- Quote: "Context compression is automatically skipped for [image-generation models] that only output images (no text output)."
- It is switched off for `deferred_tools` requests (P51). The advisor uses it for its own memory: "if the history exceeds the advisor model's context window, it is compressed with the [middle-out transform]". (P-s8)
- Source: P-s39, P-s15. Basis: OR-documented.

### G. Files and PDFs

**P63. A file goes in a `file` part on chat completions and an `input_file` part on Responses.**

- Quote: "On the Responses API, PDFs can be sent as **direct URLs** (`file_url`) or **base64-encoded data URLs** (`file_data`) in the `input` array, via the `input_file` content type; on Chat Completions they go in a `file` content part. This feature works on **any** model on OpenRouter."
- Spec `ChatContentFile.file`: `file_data` ("File content as base64 data URL or URL"), `file_id` ("File ID for previously uploaded files"), `filename` ("Original filename"). Spec `InputFile`: `file_data`, `file_id`, `file_url`, `filename`.
- Doc example for chat completions: `{type: "file", file: {filename: "document.pdf", file_data: ("data:application/pdf;base64," + $b64)}}`.
- Source: P-s22, P-s35. Basis: OR-documented.

**P64. PDF handling has three engines, chosen through the `file-parser` plugin.**

- Quote: "`engine` is one of `cloudflare-ai`, `mistral-ocr`, or `native`".
- Three quotes, one per engine: "`"mistral-ocr"`: Best for scanned documents or PDFs with images (\$2 per 1,000 pages)." Then "`"cloudflare-ai"`: Converts PDFs to markdown using Cloudflare Workers AI (Free)." Then "`"native"`: Only available for models that support file input natively (charged as input tokens)."
- Quote: "The `"pdf-text"` engine is deprecated and automatically redirected to `"cloudflare-ai"`."
- Default: "If you don't explicitly specify an engine, OpenRouter will default first to the model's native file processing capabilities, and if that's not available, we will use the `"mistral-ocr"` engine."
- Quote: "OCR costs apply to all requests, including BYOK. OpenRouter uses its own Mistral key for OCR (not your BYOK key)".
- Quote: "OpenRouter requests at most **8 images per PDF** from Mistral... and forwards no more than 8 images per request to the downstream model." and "If your downstream model does not accept image input at all, OCR-extracted images are stripped entirely and only the parsed text is forwarded."
- Source: P-s22. Basis: OR-documented.

**P65. Whether a PDF reaches the model as a file or as parsed text depends on the model, and OpenRouter downloads PDF URLs itself for two of the three engines.**

- Quote: "When a model supports file input natively, the PDF is passed directly to the model. When the model does not support file input natively, OpenRouter will parse the file and pass the parsed results to the requested model."
- Quote: "PDF URLs work with all processing engines. For Mistral OCR, the URL is passed directly to the service. For other engines, OpenRouter fetches the PDF and processes it internally."
- Source: P-s22. Basis: OR-documented.

**P66. On chat completions the parsed PDF comes back as a `file` annotation that the caller can send back to skip parsing.**

- Quote: "When you send a PDF to the Chat Completions API (`/api/v1/chat/completions`), the response may include file annotations in the assistant's message... By sending these annotations back in subsequent requests, you can avoid re-parsing the same PDF document multiple times, which saves both processing time and costs."
- Type from the page: `type FileAnnotation = { type: 'file'; file: { hash: string; name?: string; content: ContentPart[]; }; };` with `ContentPart` being `{ type: 'text'; text: string }` or `{ type: 'image_url'; image_url: { url: string } }`.
- Quote: "The `hash` field uniquely identifies the parsed file content and is used to skip re-parsing when you include the annotation in subsequent requests."
- Where it goes on the next request, from the cURL example: `{role: "assistant", content: ($a[0].content // ""), annotations: $a[0].annotations}`, sent along with the original user message that still holds the file.
- Error path: "the error response still includes the parsed annotations under `error.metadata.file_annotations`" and "The `"native"` engine doesn't produce annotations because the file is forwarded directly to the model."
- The annotation can be large. A comment in the doc's script says: "annotations can hold the parsed text of the whole PDF".
- Source: P-s22. Basis: OR-documented.

**P67. OpenRouter has its own Files API with its own ids.**

- Quote: "The Files API stores files in your workspace. You can upload a file once and use it in many requests." and "File ids start with `or_file_`."
- Limits: "The maximum file size is 100 MiB", "Each workspace can store up to 10 GiB in total", "Workspace files do not expire."
- Types: "PDF documents", "PNG, JPEG, GIF, and WebP images", "DOCX, XLSX, and PPTX documents", "MP3, WAV, FLAC, and OGG audio", "UTF-8 text".
- Quote: "Files that you upload cannot be downloaded directly, they can only be used within OpenRouter sandboxes created by the shell or bash tools."
- Shape switching: "Requests that look like OpenAI SDK requests get the OpenAI shape. Requests with an `anthropic-version` header get the Anthropic shape."
- Regional limit: "The Files API works on the global endpoint (`openrouter.ai`) only."
- Source: P-s19. Basis: OR-documented.

**P68. Vendor file stores are reachable only with the caller's own vendor key, and the docs do not say how a vendor file id is then used in a prompt.**

- Quote: "You can pass `provider=openai` or `provider=anthropic` as a query parameter. The request then goes to that provider's own Files API. It uses the [BYOK key] you configured for that provider. Without a configured key, the request returns a `400` error." Changelog: "new parameter `provider`" on the file endpoints under July 29, 2026.
- Spec fields that take a file id in a prompt: `ChatContentFile.file.file_id`; `InputFile.file_id`; `InputImage.file_id` ("Provide either an image_url (a URL or a base64 data URL) or the file_id of an uploaded image.", added September 17, 2026); and on the Messages route the `file` source of `document` and `image` blocks, whose spec example is `{"file_id": "or_file_011CNha8iCJcU1wXNR6q4V8w", "type": "file"}`.
- OpenAI `code_interpreter` with `container.file_ids` "requires BYOK" (P19).
- A content search for `file_uri`, `fileUri`, and "Gemini File API" finds nothing in the mirrored guides and cookbook, and nothing in the spec.
- Source: P-s19, P-s35, P-s34. Basis: OR-documented for the quoted lines. The use of `or_file_` ids as model input is in "Conflicts", and Gemini file URIs are in "Open questions".

**P69. Sandbox containers keep files between requests, keyed by session.**

- Quote: "When the request has a session id, the container id is `sess_` plus the session id. All requests with the same session id share one container."
- Session id sources, in order: "The `session_id` field in the request body.", "The `x-session-id` request header.", "The `prompt_cache_key` field in the request body."
- Quote: "A container sleeps after it has been idle for 5 minutes." and "Saved container files are kept for 30 days after the container was last used, then deleted."
- Attaching uploads: `"environment": {"type": "container_auto", "file_ids": ["or_file_011CNha8iCJcU1wXNR6q4V8w"]}`, "A request can attach up to 20 files."
- Output files appear in tool results as `container_file_citation` entries with `container_id`, `file_id` (prefix `cfile_`), `filename`. "The final assistant message does not carry these files as annotations." They are fetched from `GET /api/v1/containers/{container_id}/files/{file_id}/content` or promoted with `POST .../promote`.
- Quote: "The shell tool and the bash tool use separate compute environments, even for the same container id. They share the same saved files."
- Source: P-s20, P-s5. Basis: OR-documented.

**P70. Anthropic's native rule for uploaded files.**

- Quote: "**Messages API**: Refer to an uploaded file by its `file_id` in a `document`, `image` or `container_upload` block."
- Source: P-s48. Basis: Vendor-documented (Anthropic, read 2026-11-22).

### H. Multimodal input and output

**P71. Each route has its own content part names for the same media.**

- Chat completions (`ChatContentItems`): `text`, `image_url`, `input_audio`, `video_url`, `file`, and a legacy `input_video` (`Legacy_ChatContentVideo`).
- Responses: "**Images**: Use `input_image` content type", "**PDFs**: Use `input_file` content type with PDF data", "**Audio**: Use `input_audio` content type", "**Video**: Use `input_video` content type".
- Messages: Anthropic's `text`, `image`, `document` blocks. "The Messages API (`/api/v1/messages`) does not support video inputs."
- Source: P-s35, P-s21, P-s25. Basis: OR-documented.

**P72. URL or base64 depends on the modality and, for video, on the provider.**

- Quote: "**Images**: `https://example.com/image.jpg`", "**PDFs**: `https://example.com/document.pdf`", "**Audio**: Not supported via URL (base64 only)", "**Video**: Provider-specific (e.g., YouTube links for Gemini on AI Studio)".
- Quote: "Video URL support varies by provider. OpenRouter only sends video URLs to providers that explicitly support them. For example, Google Gemini on AI Studio only supports YouTube links (not Vertex AI)." and "**Google Gemini (Vertex AI)**: Does not support video URLs. Use base64-encoded data URLs instead".
- Images: "Each image is an `input_image` content part whose `image_url` can either be a URL or a base64-encoded data URL." and "The number of images you can send in a single request varies per provider and per model." and "we recommend sending the text prompt first, then the images... system messages should contain text only." Supported image types: `image/png`, `image/jpeg`, `image/webp`, `image/gif`.
- The image page does not say whether OpenRouter downloads an image URL for a provider that takes only inline data. For PDFs it does say so (P65).
- Source: P-s21, P-s25, P-s23. Basis: OR-documented.

**P73. `image_url.detail` has one OpenRouter-only value.**

- Spec: enum `auto`, `low`, `high`, `original`. "`original` is an OpenRouter extension (not in the OpenAI Chat Completions spec) requesting true original-resolution media; it is downgraded to `high` for providers that lack an original-resolution tier."
- Source: P-s35. Basis: OR-documented.

**P74. Audio input is base64 only, and the accepted formats differ by route.**

- Quote: "Audio files must be **base64-encoded** - direct URLs are not supported for audio content."
- Quote: "The Responses API (`/api/v1/responses`) accepts two `format` values for `input_audio`", which the page lists as `wav` and `mp3`. And: "Chat Completions (`/api/v1/chat/completions`) accepts any format string the provider supports, such as `aiff`, `aac`, `ogg`, `flac`, `m4a`, `pcm16`, or `pcm24`".
- Source: P-s24. Basis: OR-documented.

**P75. Video input has a Gemini-only processing switch, and its results come back as encrypted reasoning.**

- Quote: "Video processing modes are currently a **Google Gemini** feature. When a processing mode is set, OpenRouter routes the request to an endpoint that supports it. Other providers ignore the `processing` field."
- Values: `agentic` and `static`. "If `processing` is omitted, OpenRouter does not send a value upstream and Gemini's default applies."
- Quote: "**Chat Completions API**: The navigation steps appear as encrypted `reasoning_details` entries on the assistant message, and the response finishes with `finish_reason: "stop"`."
- Supported video types: `video/mp4`, `video/mpeg`, `video/mov`, `video/webm`.
- Source: P-s25. Basis: OR-documented.

**P76. Image output from a chat model is in the spec (`modalities`, `message.images`, `image_config`) and absent from today's guides, which document only the dedicated Image API.**

- Spec `ChatRequest.modalities`: "Output modalities for the response. Supported values are "text", "image", and "audio"." `ResponsesRequest.modalities`: "Supported values are "text" and "image"."
- Spec `ChatAssistantMessage.images` (`ChatAssistantImages`): "Generated images from image generation models", items `{"image_url": {"url": ...}}` with `url` described as "URL or base64-encoded data of the generated image".
- Spec `ImageConfig`: "Provider-specific image configuration options. Keys and values vary by model/provider."
- In the spec `images` is a property of the full message only. `ChatStreamDelta` lists `audio`, `content`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`.
- The image generation guide opens: "OpenRouter provides a dedicated Image API for generating images from text prompts (and optional reference images)." A content search of the mirrored `/docs/guides/` pages for `message.images`, `image_config`, and `"modalities"` finds only the audio page (for `modalities`). Outside the guides, the batch quickstart has the line "Image or audio output (`modalities`, `audio`, `image_config`) | Use the sync API." and one cookbook page says the image generation server tool's `parameters` object "accepts `model` plus all `image_config` fields".
- Source: P-s35, P-s26, and a content search of the mirror. Basis: OR-documented.

**P77. Audio output works on chat completions only and only when streaming.**

- Quote: "Audio output uses the Chat Completions API (`/api/v1/chat/completions`), because the Responses API does not accept the `audio` parameter."
- Quote: "To receive audio output, set `modalities` to `["text", "audio"]` and provide the `audio` configuration with your desired voice and format".
- Quote: "Audio output requires streaming (`stream: true`). Audio data and transcript are delivered incrementally via the `delta.audio` field in each chunk".
- `audio` options: `voice` ("Available voices vary by model.") and `format` ("Available formats vary by model.").
- Spec: `ChatAudioOutput` has `data`, `expires_at`, `id`, `transcript`. The spec's `ChatRequest` property list leaves `audio` out (see "Conflicts").
- Source: P-s24, P-s35. Basis: OR-documented.

**P78. Image, video, speech, and transcription each have a dedicated route outside the chat shape.**

- Images: "Send a `POST` request to `/api/v1/images` with the model and prompt". Streaming there uses its own events, `image_generation.partial_image` and `image_generation.completed`. Discovery at `/api/v1/images/models`.
- Video: "Video generation uses an asynchronous API at `/api/v1/videos`. You submit a prompt, receive a job ID, then poll until the video is ready to download."
- Speech: "Text-to-speech uses a dedicated endpoint at `/api/v1/audio/speech`... The endpoint is compatible with the OpenAI Audio Speech API".
- Transcription: "Speech-to-text uses a dedicated endpoint at `/api/v1/audio/transcriptions`."
- Spec paths: `/images`, `/images/models`, `/videos`, `/videos/{jobId}`, `/videos/{jobId}/content`, `/videos/models`, `/audio/speech`, `/audio/speech/models`, `/audio/transcriptions`.
- Source: P-s21, P-s26, P-s35. Basis: OR-documented.

**P79. Media support is uneven across the catalog.**

- Of 476 models in the public catalog on 2026-11-22, `architecture.input_modalities` includes `image` for 336, `file` for 204, `video` for 103, `audio` for 52. `output_modalities` includes `image` for 13, `audio` for 4, `video` for 2.
- Source: P-s40 (I counted with a script). Basis: OR-documented.

### I. MCP

**P80. The MCP guide treats MCP as something the caller runs.**

- Quote: "By converting MCP (Anthropic) tool definitions to OpenAI-compatible tool definitions, you can use MCP servers with OpenRouter."
- Quote: "The [`@openrouter/mcp`] package handles all of this for you". The sentence goes on to say the package connects to a remote MCP server and returns tools for `callModel`. This is the Agent SDK, which runs in the caller's process.
- Source: P-s33. Basis: OR-documented.

**P81. The spec nonetheless has a server-run MCP tool on the Responses route, added on November 5, 2026, which the guides do not describe.**

- `McpServerTool_OpenRouter`, type `openrouter:mcp`: "OpenRouter built-in server tool: connects the model to a remote MCP server and runs its tools during the response". `parameters` is required.
- `McpServerToolParameters`: `server_label` (required, "Label for this MCP server, used in mcp_list_tools and mcp_call items."), `server_url` ("URL of the remote MCP server (streamable HTTP)."), `authorization` ("OAuth access token sent as the Authorization bearer to the server."), `headers`, `allowed_tools`, `server_description`, `connector_id` ("Provider-defined connector identifier, forwarded as written to providers that run MCP themselves."), `require_approval` ("Approval policy for tool calls. Defaults to always: a call that needs approval ends the response with an mcp_approval_request item, answered with an mcp_approval_response on the next request.").
- OpenAI's own `mcp` tool type is in the same union (P19). Output items `mcp_list_tools`, `mcp_call`, `mcp_approval_request` are in `OutputItems`.
- Changelog, November 5, 2026: new schemas "`McpServerToolParameters`" and "`McpServerTool_OpenRouter`"; "`OutputItems`: openrouter:mcp `#/components/schemas/OutputMcpServerToolItem` object removed".
- `openrouter:mcp` is in the Responses union only. It is absent from the chat completions and Messages unions (P3), from the guide's table of server tools (P2), and `https://openrouter.ai/docs/guides/features/server-tools/mcp.md` returned 404 on 2026-11-22.
- `deferred_tools` knows about MCP tools: `scope: "mcp"` defers "only MCP server tools (`mcp__*` names and Responses namespaces outside `functions`)".
- Source: P-s35, P-s34, one live 404. Basis: OR-documented.

**P82. Anthropic's native MCP connector needs a top-level field and a beta header that the OpenRouter Messages schema does not list.**

- Quote: "**Messages API**: Pass `mcp_servers` and a matching `mcp_toolset` in `tools` on each request." The page's metadata lists "betaHeader: mcp-client-2026-09-15, mcp-client-2025-11-20", and its cURL example sends `-H "anthropic-beta: mcp-client-2025-11-20"`.
- `mcp_servers` is outside the `MessagesRequest` property list (P21).
- Source: P-s46, P-s35. Basis: Vendor-documented (Anthropic, read 2026-11-22). That the connector is therefore unreachable through OpenRouter is an inference from the missing field.

### J. The two local survey claims

**P83. LC9 holds, with one precision fix and a longer list.**

- The claim: "Server tools include web search, shell, advisor and subagent, and `deferred_tools` lets a caller hold tools back from the initial context" (`runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, line 138).
- Live: all four tools exist (P2). The list is now thirteen in the guide, plus `openrouter:mcp` in the spec (P81).
- Precision: the thing that holds a tool back is the per-tool flag `defer_loading` (P46). `deferred_tools` is the request field that opts into the versioned protocol with router-built search and state (P50, P51). A caller can use `defer_loading` with no `deferred_tools` field.
- Basis: OR-documented (live pages, 2026-11-22); also in workspace: the 2026-10-28 survey.

**P84. LC19 holds, and the feature has grown since the claim was written.**

- The claim, as worded in the research brief: "OpenRouter offers gateway-managed regex tool search in beta". The workspace traces it to the MCP Progressive Disclosure working group's limits document (`runvil-design`, `research/2026-10-14-harness-mcp-dynamic-tools.md`, line 458), which is a third-party statement about OpenRouter.
- Live: `openrouter:tool_search` is in beta, regex only, run by OpenRouter (P47). New since then: the `deferred_tools` protocol also offers `bm25` and caller-run `custom` search (P50); the server tool type is refused on chat completions (P4); and the spec has a second profile, `universal` (P50).
- Basis: OR-documented (live pages, 2026-11-22).

### K. Two more fills from LiteLLM

**P85. LiteLLM maps Anthropic's `pause_turn` to the chat completions finish reason `length` and keeps the raw value in an extra field.**

- At commit `90dce4b`, `litellm/litellm_core_utils/core_helpers.py` lines 514 to 523, inside `_FINISH_REASON_MAP`: `"tool_use": "tool_calls"`, `"refusal": "content_filter"`, `"compaction": "length"`, `"pause_turn": "length"`, `"model_context_window_exceeded": "length"`.
- `litellm/llms/anthropic/common_utils.py` lines 3036 to 3045: `_STOP_REASONS_KEPT_RAW: Final = frozenset({"refusal", "pause_turn"})`, and `stop_reason_provider_specific_fields` returns `{"stop_reason": stop_reason}` for those two.
- `factory.py` lines 2181 and 3570 to 3571 hold `_paused_server_tool_use_ids` and `_paused_turn_replay`, which rebuild a paused turn on the next request.
- Source: P-s42. Basis: Filled (LiteLLM). It shows one way to give `pause_turn` a place in the chat completions shape. It says nothing about OpenRouter.

**P86. LiteLLM converts image URLs to inline base64 for the Anthropic endpoints that refuse URL sources.**

- At commit `90dce4b`, `litellm/litellm_core_utils/prompt_templates/factory.py` lines 3381 to 3385: the comment "Vertex AI Anthropic also doesn't support URL sources for images", then `force_base64 = is_bedrock_invoke or is_vertex_ai`. The helper `convert_url_to_base64` is defined in `prompt_templates/image_handling.py` line 160.
- Source: P-s42. Basis: Filled (LiteLLM). OpenRouter's image page is silent on the same point (P72).

## Conflicts and disagreements

Conflicts inside OpenRouter's own material (guide against spec, or guide against live API):

1. **`openrouter:bash` routes.** The bash guide says "Requesting it on the Chat Completions or Responses API returns a `400` error." The spec lists `BashServerTool` in the `tools` union of both `ChatRequest` and `ResponsesRequest`, and `OutputItems` has an `openrouter:bash` item. Not resolved here. The guide is the more specific statement, and the union may only reflect that the parser accepts the shape before the route check runs (P3, P4).
2. **`deferred_tools.profile`.** The tool search guide says "`"portable"` is the only profile in this release". The spec enum is `portable`, `universal`, with six more universal-only fields added between November 5 and 9, 2026. The spec is the newer source by the changelog dates; the guide text looks behind (P50, P56).
3. **`max_tool_calls` placement.** The server tools guide calls `max_tool_calls` and `stop_server_tools_when` "top-level request fields" for "Every request that uses server tools". In the spec `max_tool_calls` is a property of `ResponsesRequest` only; `ChatRequest` and `MessagesRequest` have `stop_server_tools_when` alone (P9).
4. **Chat `annotations`.** Two guide pages show `message.annotations` on chat completions (`url_citation` and `file`). The spec's `ChatAssistantMessage` and `ChatStreamDelta` schemas do not list `annotations`. OpenRouter's own AI SDK provider reads `message.annotations` and `delta.annotations`. The guides and the SDK agree, so the spec schema is the incomplete one (P38, P41, P66).
5. **Chat request fields missing from the spec.** The guides document `audio`, `web_search_options`, and the legacy `x_search_filter` as chat request fields. The spec's `ChatRequest` property list leaves all three out (P60, P77, P15).
6. **Two shapes for a server tool call on chat completions.** The spec says such a call is "carried in reasoning_details" as `reasoning.server_tool_call`. The advisor guide says to replay "the assistant message's advisor `tool_calls` and the paired `role: "tool"` result messages". Both may be true for different tools; no page says which tool uses which (P26, P27).
7. **Perplexity native search.** The web search guide lists "**Perplexity**: all Perplexity models" as having native web search. The live endpoints listing for `perplexity/sonar-pro-search` and `perplexity/sonar-reasoning-pro` shows an empty `native_tools` map. A likely reading is that Perplexity models search on their own with no tool to translate to, but that is my reading and no page states it (P12, P13).
8. **`:online` equivalence.** The web plugin page and the `:online` page say the suffix is "exactly equivalent to" a request with `"model": "openrouter/auto"`. The plugins overview page shows the equivalent with the original model slug. The second is the plausible one; the first reads like a documentation slip (P60).
9. **What an uploaded file is for.** The Files API page says "You can upload a file once and use it in many requests" and also "Files that you upload cannot be downloaded directly, they can only be used within OpenRouter sandboxes created by the shell or bash tools." The spec has `file_id` fields on chat `file` parts, Responses `input_file` and `input_image` parts, and Messages `document` and `image` sources, with an `or_file_` id as the example for the last. Not resolved (P67, P68).
10. **Perplexity engine with both domain lists.** The plugin page says "when both provided, `include_domains` wins". The server tool page says "when both are provided, the request is rejected with a 400". These describe two different surfaces, so both can hold (P14, P-s16).
11. **The guide's tool table against the spec.** The guide lists thirteen server tools. The spec adds `openrouter:mcp` as a request type and five output-only item types that the guides leave out (P6, P81).

Disagreements with the local Runvil documents:

12. **LC9** holds with a precision fix: the hold-back mechanism is the per-tool `defer_loading` flag, and `deferred_tools` is the protocol opt-in (P83).
13. **LC19** holds and is now incomplete: BM25 and caller-run search exist under `deferred_tools.search`, and a `universal` profile exists in the spec (P84).
14. **The 2026-11-17 report on the Anthropic Messages API** says server tools "produce blocks and stop reasons with no home in chat completions: `server_tool_use`, `pause_turn`, and `compaction`". That is right for the OpenAI standard. It is worth knowing that two projects have since given them a place: OpenRouter's spec defines a `reasoning.server_tool_call` entry in `reasoning_details` for its own server tools (P26), and LiteLLM maps `server_tool_use` to `tool_calls`, parks result blocks in `provider_specific_fields`, and maps `pause_turn` to `length` (P36, P85). Neither is part of the OpenAI standard.
15. **The task framing** assumed PDF engines `pdf-text`, `mistral-ocr`, `native`. Live: `pdf-text` is deprecated and redirected to `cloudflare-ai` (P64).

## Open questions blocked by closed source

1. **When web search runs natively on an Anthropic endpoint and the caller is on chat completions, where do the `web_search_tool_result` blocks and their `encrypted_content` go, and what must come back?** Tried: all server tool guides, the spec (`ChatAssistantMessage` has `tool_calls`, `reasoning_details`, no block list), the official AI SDK provider. The guides say only that citations are normalized to `annotations` and that usage reports `web_search_requests`. Fill found: LiteLLM (P36) keeps the raw result blocks in `provider_specific_fields.web_search_results` and rebuilds the pairs on replay.
2. **Does OpenRouter ever end a response with `pause_turn`, and what `finish_reason` does a chat completions caller see then?** Tried: spec (the Messages enum has `pause_turn`), guides (no mention), changelog (no hit). Fill found: Anthropic's rule (P34) and LiteLLM's mapping to `length` with the raw value kept beside it (P85).
3. **Can a caller reach Anthropic code execution, memory, or native-spelled web fetch on the Messages route through the generic `{type: string}` tool variant?** Tried: spec, guides, cookbook, the live `native_tools` sample. Evidence points both ways (P21, P22). No fill found; this is OpenRouter policy, which another project cannot show.
4. **Can a caller reach Gemini code execution, URL context, or Maps grounding on any route?** Tried: the three tool unions, a spec-wide search, the guides. No spelling exists (P23). No fill found; the LiteLLM checkout I read did not include its Gemini translator.
5. **Are Anthropic document citations and Gemini search suggestions mapped into chat `annotations`, dropped, or passed elsewhere?** Tried: guides and spec (P45). Fill found for Anthropic only: LiteLLM stores them in `provider_specific_fields["citations"]`. No fill found for Gemini.
6. **Which server tools produce a `reasoning.server_tool_call` entry on chat completions, which are "not visible to the caller", and must the entry be sent back?** The Tools API field `output_names` answers the first two parts and needs a key (P5, P26). No fill found.
7. **What is the `rs_server_tool_round_*` reasoning item on the Responses route for, and must the caller return it?** Its content decodes to a list of item ids (P28). No page explains it. No fill found.
8. **Does OpenRouter download an image URL and inline it for a provider that takes only base64?** The image page is silent, the PDF page says OpenRouter fetches PDFs, and the video page says URLs are sent only to providers that support them (P65, P72). Fill found: LiteLLM forces base64 for Anthropic on Bedrock invoke and Vertex (P86).
9. **If routing fails over from an endpoint with native web search to one without it (for example Anthropic first-party to Bedrock), does the engine switch to Exa inside the same request, and does the response shape change?** The `native_tools` data shows the capability differs per endpoint of one model (P12). No page covers failover. No fill found.
10. **Is an `or_file_` id or a vendor file id usable in a chat `file` part as model input?** The docs conflict (see conflict 9). No fill found.
11. **What produces the `openrouter:code_interpreter`, `openrouter:memory`, `openrouter:browser_use`, `openrouter:text_editor`, and `openrouter:file_search` output items?** Spec only (P6). No fill found.
12. **Is `openrouter:mcp` live?** It is in the spec since November 5, 2026, and the guides do not describe it (P81). Testing it needs an inference call, which this research does not make.

## Unverified

- No request was sent to any model. Every response shape in this file comes from documentation, the spec, or SDK source, and none was observed on the wire.
- The per-tool alias table (`aliases[].type`, `api_formats`, `output_names`) and the engine price rows from `GET /api/v1/tools` were not read, because the endpoint needs a key.
- The `native_tools` tabulation covers 27 of 476 models.
- The changelog page starts on July 3, 2026. Anything that changed before that date was not checked.
- From memory, not verified today: Google's older `generateContent` grounding response uses a `groundingMetadata` object with a `searchEntryPoint` whose rendered HTML must be displayed. The live Google page I read (P-s52) describes `google_search_call` and `google_search_result` steps with a `search_suggestions` HTML snippet, and I did not read a page for the older field names.
- xAI's and Perplexity's own search documentation was not read. One xAI documentation URL I tried returned 404.
- OpenAI's MCP and connectors guide was downloaded and not read; OpenAI's file search and code interpreter pages were not fetched. The OpenAI rows in the matrix rest on OpenRouter's spec descriptions of those tool types.
- Anthropic's memory tool, text editor tool, and computer use pages were not read.
- P37's consequence (that OpenRouter's AI SDK provider loses a `reasoning.server_tool_call` entry on replay) is inferred from one filter and was not traced through the replay path.
- P22's reading (that Anthropic code execution passes through the generic Messages tool variant) is an inference.
- The LiteLLM checkout was a partial clone made by another research agent at commit `90dce4b`; I read four files in it and did not confirm that the Gemini translator is absent from the full repository.

## Workspace documents used

- (P-s54) `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md`, dated 2026-10-28. Took the wording of LC9 and LC19 (line 138). Re-checked live: yes, both against OpenRouter's pages on 2026-11-22 (P83, P84).
- (P-s55) `runvil-design`, `research/2026-10-14-harness-mcp-dynamic-tools.md`, dated 2026-10-14. Took the origin of the LC19 claim (line 458, the MCP working group's limits document), and the Anthropic and OpenAI tool search rules (lines 452 to 470). Re-checked live: the vendor rules yes (P55). Its note that Claude Code turns tool search off for a non-first-party base URL "because most proxies do not forward `tool_reference` blocks" was not re-checked; OpenRouter now documents its own handling of `tool_reference` (P46).
- (P-s56) Report on the Anthropic Messages API and harness survey, dated 2026-11-17 (cabinet output). Took the meaning of `pause_turn`, the statement that server tool blocks have "no home in chat completions", and the LiteLLM block reorder failure. Re-checked live: the Anthropic rule yes (P34); the LiteLLM issue was not re-fetched.
- (P-s57) `runvil-plugin-web`, `research/2026-11-19-web-fetch-field-survey.md`, dated 2026-11-19. Searched for OpenRouter and found no mention, so nothing was taken from it. OpenRouter's web fetch server tool (P16) is new material relative to that survey.

## Sources

The keys are not contiguous: numbers 27 to 29, 47, 49, and 53 were not used. There are 51 sources.

| Key | URL, or repository at commit with file path | Read date | Kind |
| - | - | - | - |
| P-s1 | https://openrouter.ai/docs/guides/features/server-tools | 2026-11-22 | OpenRouter doc |
| P-s2 | https://openrouter.ai/docs/guides/features/server-tools/web-search | 2026-11-22 | OpenRouter doc |
| P-s3 | https://openrouter.ai/docs/guides/features/server-tools/web-fetch | 2026-11-22 | OpenRouter doc |
| P-s4 | https://openrouter.ai/docs/guides/features/server-tools/tool-search | 2026-11-22 | OpenRouter doc |
| P-s5 | https://openrouter.ai/docs/guides/features/server-tools/shell | 2026-11-22 | OpenRouter doc |
| P-s6 | https://openrouter.ai/docs/guides/features/server-tools/bash | 2026-11-22 | OpenRouter doc |
| P-s7 | https://openrouter.ai/docs/guides/features/server-tools/apply-patch | 2026-11-22 | OpenRouter doc |
| P-s8 | https://openrouter.ai/docs/guides/features/server-tools/advisor | 2026-11-22 | OpenRouter doc |
| P-s9 | https://openrouter.ai/docs/guides/features/server-tools/subagent | 2026-11-22 | OpenRouter doc |
| P-s10 | https://openrouter.ai/docs/guides/features/server-tools/fusion | 2026-11-22 | OpenRouter doc |
| P-s11 | https://openrouter.ai/docs/guides/features/server-tools/image-generation | 2026-11-22 | OpenRouter doc |
| P-s12 | https://openrouter.ai/docs/guides/features/server-tools/search-models | 2026-11-22 | OpenRouter doc |
| P-s13 | https://openrouter.ai/docs/guides/features/server-tools/datetime | 2026-11-22 | OpenRouter doc |
| P-s14 | https://openrouter.ai/docs/guides/features/server-tools/files (fetched live; the page was missing from the local mirror) | 2026-11-22 | OpenRouter doc |
| P-s15 | https://openrouter.ai/docs/guides/features/plugins | 2026-11-22 | OpenRouter doc |
| P-s16 | https://openrouter.ai/docs/guides/features/plugins/web-search | 2026-11-22 | OpenRouter doc |
| P-s17 | https://openrouter.ai/docs/guides/features/plugins/alignment | 2026-11-22 | OpenRouter doc |
| P-s18 | https://openrouter.ai/docs/guides/features/plugins/armor | 2026-11-22 | OpenRouter doc |
| P-s19 | https://openrouter.ai/docs/guides/features/files-api | 2026-11-22 | OpenRouter doc |
| P-s20 | https://openrouter.ai/docs/guides/features/containers | 2026-11-22 | OpenRouter doc |
| P-s21 | https://openrouter.ai/docs/guides/overview/multimodal/overview | 2026-11-22 | OpenRouter doc |
| P-s22 | https://openrouter.ai/docs/guides/overview/multimodal/pdfs | 2026-11-22 | OpenRouter doc |
| P-s23 | https://openrouter.ai/docs/guides/overview/multimodal/image-understanding | 2026-11-22 | OpenRouter doc |
| P-s24 | https://openrouter.ai/docs/guides/overview/multimodal/audio | 2026-11-22 | OpenRouter doc |
| P-s25 | https://openrouter.ai/docs/guides/overview/multimodal/videos | 2026-11-22 | OpenRouter doc |
| P-s26 | https://openrouter.ai/docs/guides/overview/multimodal/image-generation (with the headings of the video-generation, tts, and stt pages in the same folder) | 2026-11-22 | OpenRouter doc |
| P-s30 | https://openrouter.ai/docs/api_reference/responses/web-search | 2026-11-22 | OpenRouter doc |
| P-s31 | https://openrouter.ai/docs/api/api-reference/tools/list-server-tools and https://openrouter.ai/docs/api/api-reference/tools/get-a-server-tool | 2026-11-22 | OpenRouter doc |
| P-s32 | https://openrouter.ai/docs/guides/routing/model-variants/online | 2026-11-22 | OpenRouter doc |
| P-s33 | https://openrouter.ai/docs/cookbook/coding-agents/mcp-servers | 2026-11-22 | OpenRouter doc |
| P-s34 | https://openrouter.ai/docs/changelog (entries from July 3, 2026 to November 21, 2026) | 2026-11-22 | OpenRouter doc |
| P-s35 | https://openrouter.ai/openapi.json (OpenAPI 3.1, 123 paths, 1071 schemas) | 2026-11-22 | OpenRouter spec |
| P-s36 | `GET https://openrouter.ai/api/v1/tools` with no credentials: HTTP 401 | 2026-11-22 | OpenRouter spec (live public API response) |
| P-s37 | `GET https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints` for 27 models, no credentials, all HTTP 200 | 2026-11-22 | OpenRouter spec (live public API response) |
| P-s38 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5 | 2026-11-22 | OpenRouter doc |
| P-s39 | https://openrouter.ai/docs/guides/features/message-transforms | 2026-11-22 | OpenRouter doc |
| P-s40 | https://openrouter.ai/api/v1/models (476 models) | 2026-11-22 | OpenRouter spec (live public API response) |
| P-s41 | https://github.com/OpenRouterTeam/ai-sdk-provider at `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (commit date 2026-11-20): `src/schemas/reasoning-details.ts`, `src/chat/schemas.ts`, `src/chat/index.ts`, `src/chat/convert-to-openrouter-chat-messages.ts` | 2026-11-22 | OpenRouter SDK source |
| P-s42 | https://github.com/BerriAI/litellm at `90dce4b93bd2b090621a6b9083311294d3358bc6` (commit date 2026-11-22): `litellm/llms/anthropic/chat/transformation.py`, `litellm/litellm_core_utils/prompt_templates/factory.py`, `litellm/litellm_core_utils/core_helpers.py`, `litellm/llms/anthropic/common_utils.py` | 2026-11-22 | gateway source |
| P-s43 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/web-search-tool | 2026-11-22 | vendor doc |
| P-s44 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/server-tools | 2026-11-22 | vendor doc |
| P-s45 | https://platform.claude.com/docs/en/agents-and-tools/tool-use/tool-search-tool | 2026-11-22 | vendor doc |
| P-s46 | https://platform.claude.com/docs/en/agents-and-tools/mcp-connector | 2026-11-22 | vendor doc |
| P-s48 | https://platform.claude.com/docs/en/build-with-claude/files | 2026-11-22 | vendor doc |
| P-s50 | https://developers.openai.com/api/docs/guides/tools-web-search | 2026-11-22 | vendor doc |
| P-s51 | https://developers.openai.com/api/docs/guides/tools-tool-search | 2026-11-22 | vendor doc |
| P-s52 | https://ai.google.dev/gemini-api/docs/google-search | 2026-11-22 | vendor doc |
| P-s54 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (dated 2026-10-28) | 2026-11-22 | workspace document |
| P-s55 | `runvil-design`, `research/2026-10-14-harness-mcp-dynamic-tools.md` (dated 2026-10-14) | 2026-11-22 | workspace document |
| P-s56 | Report on the Anthropic Messages API and harness survey (dated 2026-11-17, cabinet output) | 2026-11-22 | workspace document |
| P-s57 | `runvil-plugin-web`, `research/2026-11-19-web-fetch-field-survey.md` (dated 2026-11-19) | 2026-11-22 | workspace document |
