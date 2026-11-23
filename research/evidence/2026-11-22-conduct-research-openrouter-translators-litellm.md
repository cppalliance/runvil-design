---
produced: 2026-11-22
title: "LiteLLM as an open-source fill for OpenRouter's closed translator - how a chat-completions-shaped translation layer stores, replays, and reports vendor features: thinking_blocks, signatures, Gemini thought signatures in tool call ids, reasoning_items and encrypted_content, provider_specific_fields, reasoning_effort mapping, cache_control and cachedContents, tool call id rewriting, tool_choice downgrades, structured output, web search, finish reasons, drop_params, and LiteLLM's OpenRouter provider module"
---

# LiteLLM as an open-source fill for OpenRouter's closed translator

- What was read: `BerriAI/litellm` at commit `90dce4b93bd2b090621a6b9083311294d3358bc6` (commit date 2026-11-22T02:05:03-08:00, shallow sparse clone of `litellm/`), 40 source files quoted by path and line under 35 source keys. The `docs/my-website/docs` folder is outside the repository tree at this commit, so the documentation was read from the published site `docs.litellm.ai` (2 pages). Also 15 public LiteLLM issue and pull request pages (third-party accounts) and 4 workspace documents.
- How many sources: 56 (35 LiteLLM source entries, 2 documentation pages, 15 issue and pull request pages, 4 workspace documents). Evidence items: 72. Candidate matrix rows: 43.
- Read date: 2026-11-22. Every item here is a fill from LiteLLM (`Filled`), with the six rows on LiteLLM's OpenRouter module labeled `OR-inferred`. A fill shows that a translation exists and what it costs. It never shows what OpenRouter does.

## Candidate matrix rows

Two columns differ from the shared layout, as the task prompt for this topic asked. "How OpenRouter exposes it" says "see OpenRouter evidence" (another subagent's file answers it). "How LiteLLM translates it" and "What LiteLLM loses or flattens" hold the fill. The "Native rule" cells state the vendor rule as LiteLLM's code comments, error handling, or docs describe it, so their basis is also `Filled (LiteLLM)`, not `Vendor-documented`.

| Feature | Vendor(s) | Native rule (as LiteLLM's code states it) | How OpenRouter exposes it | How LiteLLM translates it | What LiteLLM loses or flattens | Deciding facts for Runvil | Basis | Evidence |
|---|---|---|---|---|---|---|---|---|
| Signed thinking blocks | Anthropic | A `thinking` block needs its `signature`; an unsigned or foreign block is a 400 | see OpenRouter evidence | Reply: blocks go to `message.thinking_blocks` (and again under `provider_specific_fields.thinking_blocks`), text to `reasoning_content`. Request: caller sends `thinking_blocks` back, translator puts them first in the assistant content | Position of each thinking block among text and tool blocks. Text blocks are joined into one string. In a stream the blocks arrive as fragments that the caller must merge by `index` | Caller holds state. Round trip through the caller of a typed, Anthropic-shaped field. Field is vendor-shaped, not neutral. A streaming caller must read the field to rebuild it | Filled (LiteLLM) | K1, K2, K6, K7, K8, K72 |
| Redacted thinking | Anthropic, Bedrock | `redacted_thinking.data` must come back unchanged | see OpenRouter evidence | Same list, `{"type": "redacted_thinking", "data": ...}` | Same as above | Same as above | Filled (LiteLLM) | K2, K6, K21 |
| Interleaved thinking and server tool blocks | Anthropic | Signatures are checked by position, so the original block order must be kept | see OpenRouter evidence | Order is rebuilt by a pattern: thinking[0], server tool group[0], thinking[1], group[1], then text, then client tool calls | The real order. The pattern is a guess that failed in the field before it was added (issue 23047) | A translator that rebuilds a reply has to guess order. A stored ordered block list removes the guess | Filled (LiteLLM) | K7, K57 |
| Thinking required on the open tool-loop message | Anthropic (budgeted thinking) | With `thinking` enabled, an assistant tool-call message must start with a thinking block | see OpenRouter evidence | With `litellm.modify_params` on, the `thinking` param is dropped for that request when the last tool-call message has no `thinking_blocks` and no other message has any | Thinking for that turn | Gateway can do this alone, at the price of a silent quality drop | Filled (LiteLLM) | K9, K59 |
| Compaction blocks | Anthropic | Compaction blocks go at the start of assistant content | see OpenRouter evidence | Reply: `provider_specific_fields.compaction_blocks`. Request: read from the same field and placed first | Blocks survive when the caller returns the field | Caller must return `provider_specific_fields` | Filled (LiteLLM) | K6, K7 |
| Thought signature on function calls | Gemini 3+ | First function call of a turn needs its `thoughtSignature`, else 400 | see OpenRouter evidence | Reply: stored on the tool call in `provider_specific_fields.thought_signature` and appended to the tool call id as `call_...__thought__<base64>`. Request: read from either place, else a placeholder | Ids grow to 843-4423 characters (issue 37849). A client that normalizes ids breaks the next turn | Works with an unaware OpenAI client only because the id is overloaded. Caller state, unread round trip | Filled (LiteLLM) | K10, K12, K61 |
| Thought signature on text and thought parts | Gemini | A signature can sit on any part, also without `thought: true` | see OpenRouter evidence | Reply: all signatures in a flat list `provider_specific_fields.thought_signatures`, thought parts also as `thinking_blocks`. Request: only `thought_signatures[0]` goes on the text part | Which part each signature belonged to. All but the first text signature | Needs `provider_specific_fields` back from the caller | Filled (LiteLLM) | K10, K12, K63 |
| Thought signature from another backend or model | Gemini | A signature only validates on the backend that minted it | see OpenRouter evidence | To a non-Gemini model: the `__thought__` suffix is cut from ids. Between Vertex and AI Studio: on a 400 that names a thought signature, resend with the placeholder `skip_thought_signature_validator`. Missing signature on Gemini 3+: placeholder on the first call | The model's own reasoning state for that history | Gateway can handle model switches alone, with a quality cost Google documents | Filled (LiteLLM) | K12, K13, K64 |
| Server-side tool invocations | Gemini | `toolCall` and `toolResponse` parts must be sent back when context circulation is on | see OpenRouter evidence | Reply: `provider_specific_fields.server_side_tool_invocations`. Request: rebuilt as parts after the function calls | Original part order | Caller must return `provider_specific_fields`. Opt-in param `include_server_side_tool_invocations` | Filled (LiteLLM) | K14 |
| Reasoning items with `encrypted_content` | OpenAI, Azure (Responses API) | Reasoning items pair with the message and function call items that follow them | see OpenRouter evidence | Reply: `message.reasoning_items` (`id`, `encrypted_content`, `summary`) plus an order map with a checksum in `provider_specific_fields.openai_responses_replay`. Request: exact item sequence rebuilt when the checksum matches | Without the map: item order and message phases. From Anthropic-shaped history: item ids | A third field, different from the Anthropic one. Caller state, unread round trip | Filled (LiteLLM) | K2, K16, K17, K18 |
| Responses-only models behind the chat route | OpenAI, Azure, xAI search, ChatGPT | Some models and feature mixes exist only on `/responses` | see OpenRouter evidence | The chat request is bridged: messages become input items, `previous_response_id` is refused | Server-side state (`previous_response_id`), stored responses | The feature lives on another route, and a translator can hide that | Filled (LiteLLM) | K15, K16, K19 |
| Encrypted reasoning bound to one account | OpenAI, Azure | `encrypted_content` decrypts only for the organization that made it | see OpenRouter evidence | Router pins follow-up requests to the deployment that made the content (id prefix `encitem_`, wrapper `litellm_enc:`), else strips the reasoning | Reasoning when the request moves | The gateway must know which upstream account served each turn | Filled (LiteLLM) | K20, K65 |
| Reasoning content with signature | Bedrock Converse | `reasoningContent.reasoningText.signature` and `redactedContent` go back to Claude only | see OpenRouter evidence | Same `thinking_blocks` shape as Anthropic, raw blocks also in `provider_specific_fields.reasoningContentBlocks` | Signed and redacted blocks are dropped for non-Claude Converse targets | One shape serves two vendors because Bedrock hosts Claude | Filled (LiteLLM) | K21 |
| Reasoning text must be passed back | DeepSeek | "The reasoning_content in the thinking mode must be passed back to the API." | see OpenRouter evidence | `reasoning_content` on the assistant message, else a single space is injected | Reasoning chain when the caller dropped it | Plain text field, no signature. Gateway can pad it alone | Filled (LiteLLM) | K22 |
| Reasoning effort, adaptive or budget | Anthropic | Newer models take adaptive thinking plus `output_config.effort`, older ones `budget_tokens` below `max_tokens` | see OpenRouter evidence | `reasoning_effort` maps per model flag in the cost map: adaptive plus effort, or budgets 1024 / 2048 / 4096 / 8192 / 16384 | Exact budgets on adaptive models. `minimal` becomes `low` | A neutral setting can be translated. Needs a per-model table | Filled (LiteLLM) | K25, K26 |
| Reasoning effort, level or budget | Gemini | Gemini 3+ takes `thinkingLevel`, older models `thinkingBudget` | see OpenRouter evidence | By model name: level for Gemini 3+, else budget. Minimal budget differs per 2.5 model (1, 128, 512) | `budget_tokens` is dropped with a warning on Gemini 3+. Thinking cannot be fully off on Gemini 3+ (`none` becomes `minimal` or `low`) | Neutral setting works. Model generation is read from the name | Filled (LiteLLM) | K25, K27 |
| Reasoning effort, other vendors | OpenAI, DeepSeek, xAI, Mistral, Bedrock Nova, OpenRouter | Each has its own level set | see OpenRouter evidence | OpenAI: passed through, or `reasoning: {effort}` on the bridge. DeepSeek: on or off plus effort. Mistral: nearest declared level. Nova 2: `reasoningConfig`. OpenRouter: `max` becomes `xhigh` | Levels a vendor does not list are rounded or dropped | Neutral setting works with a per-vendor level table | Filled (LiteLLM) | K28, K29 |
| Cache breakpoints | Anthropic | At most 4 `cache_control` blocks | see OpenRouter evidence | `cache_control` on a message or part is copied to the block. Optional auto-injection points. Oldest marks trimmed to fit 4 | Marks arrive intact, up to four | Caller places vendor-shaped markers, or the gateway injects them from config | Filled (LiteLLM) | K30, K34 |
| Cache points | Bedrock Converse | A separate `cachePoint` block | see OpenRouter evidence | Each `cache_control` becomes a `cachePoint` block after the content | Marks arrive as `cachePoint` blocks on models that take them | Same marker, translated inside | Filled (LiteLLM) | K31, K68 |
| Explicit context cache object | Gemini | `cachedContents` is a named server object with a TTL, and a request that uses it cannot also send system instruction or tools | see OpenRouter evidence | `cache_control` markers make LiteLLM create a `cachedContents` object, keyed by a hash stored as `displayName`, found again by listing caches | Implicit caching for that prefix. Storage is billed. Only the first continuous marked block is cached | The gateway holds server state per upstream and does extra calls | Filled (LiteLLM) | K32 |
| Cache key and explicit breakpoints | OpenAI | `prompt_cache_key`, `prompt_cache_retention`, and on GPT-5.6+ `prompt_cache_breakpoint` with TTL `30m` or `12h` | see OpenRouter evidence | Keys pass through. `cache_control` and `prompt_cache_breakpoint` are renamed into each other per target | TTLs do not convert: `12h` or `30m` to Anthropic is an error, or a 5m breakpoint under `drop_params` | Two marker dialects, one rename table | Filled (LiteLLM) | K33 |
| Cache usage accounting | Anthropic, Bedrock, Gemini, DeepSeek, OpenAI | Anthropic and Bedrock report input tokens without cached ones | see OpenRouter evidence | `prompt_tokens` is made to include cache reads and writes. Reads in `prompt_tokens_details.cached_tokens`, writes in `cache_creation_input_tokens` and `prompt_tokens_details.cache_write_tokens` | Read, write, and total counts all reach the caller | Gateway only | Filled (LiteLLM) | K35 |
| Tool call id format | Anthropic, Bedrock, Gemini, Mistral | Anthropic `^[a-zA-Z0-9_-]+$`. Bedrock `[a-zA-Z0-9_.:-]+`, 64 max. Older Gemini identifies calls by name only | see OpenRouter evidence | Invalid characters replaced, Bedrock ids cut with a hash, Gemini ids minted as `call_<28 hex>`. Mistral ids are copied unchanged | Original ids after a rewrite | Gateway only. Mistral's nine-character rule stays with the caller in LiteLLM | Filled (LiteLLM) | K11, K36, K71 |
| `tool_choice` and forced tools | Anthropic, Gemini, Bedrock, Mistral | Some models cannot force a tool. Bedrock Converse offers auto, any, and a named tool | see OpenRouter evidence | Mapped per vendor. When a model cannot force: error, or downgrade to `auto` under `drop_params` | The force | Gateway only, policy choice between error and downgrade | Filled (LiteLLM) | K37, K50 |
| `parallel_tool_calls` | Anthropic, Gemini | Anthropic inverts the flag inside `tool_choice`. Gemini cannot turn it off with several tools | see OpenRouter evidence | Anthropic: `disable_parallel_tool_use`. Gemini: `false` with more than one tool is dropped | The setting on Gemini | Gateway only | Filled (LiteLLM) | K37 |
| Tool schema dialect | Gemini, xAI, Mistral | Gemini takes a reduced schema | see OpenRouter evidence | `$defs` unwound, `anyOf` null turned into nullable, unknown keywords filtered, `strict` removed | Schema keywords the vendor does not know | Gateway only | Filled (LiteLLM) | K38 |
| Images in tool results | OpenAI-compatible, Gemini | OpenAI chat tool messages take text only | see OpenRouter evidence | OpenAI-compatible: media moved to a user message after the tool run. Gemini 3+: nested in the function response. Older Gemini: sibling parts | Authority and position of the media | Gateway only | Filled (LiteLLM) | K39 |
| Structured output | Anthropic, Gemini, Bedrock, OpenAI Responses | Native JSON schema output exists on some models only | see OpenRouter evidence | Native format where the model and schema allow, else a forced tool named `json_tool_call` whose arguments become `content` | The force when thinking is on (tool is then unforced) | Gateway only | Filled (LiteLLM) | K41, K42 |
| Refusal | Anthropic | `stop_reason: "refusal"` with `stop_details.explanation` | see OpenRouter evidence | `message.refusal` plus `finish_reason: "content_filter"`, raw reason kept in `provider_specific_fields` | Refusal text and raw stop reason both reach the caller | Gateway only | Filled (LiteLLM) | K43, K46 |
| Web search | Anthropic, Gemini, OpenAI, xAI, Bedrock Nova | Each vendor has its own built-in search tool | see OpenRouter evidence | `web_search_options` becomes `web_search_20250305`, `googleSearch`, a Responses `web_search` tool, or `nova_grounding` | Gemini ignores `user_location` and `search_context_size`. Gemini drops search when mixed with functions | Neutral request works. Replay differs per vendor | Filled (LiteLLM) | K44 |
| Server tool blocks and citations | Anthropic, Gemini, Bedrock | Server tool use and result blocks must be replayed in place | see OpenRouter evidence | Anthropic: calls and results in `provider_specific_fields` (`server_tool_calls`, `web_search_results`, `tool_results`, `citations`). Gemini and Bedrock citations become `annotations` | Anthropic citations are not `annotations`. Block order | Caller must return `provider_specific_fields` | Filled (LiteLLM) | K6, K45 |
| Finish reasons | All | Each vendor has its own list | see OpenRouter evidence | One table to five OpenAI values. Unknown values become `stop`. Raw value in `provider_specific_fields.native_finish_reason` | `pause_turn`, `compaction`, and `model_context_window_exceeded` all read as `length`. `MALFORMED_FUNCTION_CALL` reads as `stop` | Gateway only. A native value field keeps the detail | Filled (LiteLLM) | K4, K46 |
| Error classes | All | Overflow, rate limit, and content policy are coded differently by each vendor | see OpenRouter evidence | Substring matching on error text | Breaks when a vendor rewords an error | Gateway only | Filled (LiteLLM) | K47, K70 |
| Mid-stream errors | All | Errors after HTTP 200 | see OpenRouter evidence | Raised as an exception from the stream. Router gets `MidStreamFallbackError` with the text generated so far | One bridge path turned the failure into an empty success (issue 32086) | Gateway only | Filled (LiteLLM) | K48, K67 |
| Stream usage | All | Usage arrives at different points | see OpenRouter evidence | Sent to the caller only with `stream_options.include_usage` | Usage is kept for cost tracking either way | Gateway only | Filled (LiteLLM) | K48 |
| Unsupported parameters | All | Vendors reject unknown parameters | see OpenRouter evidence | Default is an error. `drop_params` drops, `allowed_openai_params` forces through, `additional_drop_params` drops by name | Under `drop_params` the caller is not told | Policy choice: error by default, silent drop by opt-in | Filled (LiteLLM) | K49, K50 |
| Extension fields at a strict upstream | Any OpenAI-compatible | Unknown message fields are a 400 at strict servers | see OpenRouter evidence | Stripped in six provider transforms. The shared OpenAI-compatible path sends them as given | Hard 400 at strict servers, empty replies at lenient ones (issue 32188) | The gateway must strip its own fields for every other upstream | Filled (LiteLLM) | K24, K60 |
| OpenRouter: reasoning text and effort | OpenRouter | Inferred: streams reasoning text as `delta.reasoning`, takes effort `xhigh` and not `max` | see OpenRouter evidence | `delta.reasoning` copied to `reasoning_content`. `max` rewritten to `xhigh`. `reasoning_details` stays nested under `provider_specific_fields` | `reasoning_details` goes back only if the caller lifts it onto the message itself | Inferred from LiteLLM's code only | OR-inferred | K51 |
| OpenRouter: cache control placement | OpenRouter | Inferred: wants `cache_control` inside content blocks, for Claude, Gemini, MiniMax, GLM, Z-AI models | see OpenRouter evidence | Message-level marker moved to the last content block. Removed for other models | The marker is kept for the five model families | Inferred from LiteLLM's code only | OR-inferred | K52, K56 |
| OpenRouter: usage and cost | OpenRouter | Inferred: returns `usage.cost` when `usage: {"include": true}` is sent, a top-level `provider`, and a usage chunk after the finish chunk | see OpenRouter evidence | Always sends `usage.include`, reads `usage.cost` and `provider` | Cost and provider name are kept in hidden fields | Inferred from LiteLLM's code only | OR-inferred | K53 |
| OpenRouter: errors inside a 200 | OpenRouter | Inferred: a 200 body can hold `finish_reason: "error"` or `choices[].error`, and a stream can hold an `error` chunk | see OpenRouter evidence | Turned into an exception with the embedded code, else 502 | The embedded code and message are kept | Inferred from LiteLLM's code only | OR-inferred | K54 |
| OpenRouter: extra request fields | OpenRouter | Inferred: `transforms`, `models`, `route` are body fields | see OpenRouter evidence | Moved into `extra_body`. Headers `HTTP-Referer` and `X-Title` added | The fields pass through as sent | Inferred from LiteLLM's code only | OR-inferred | K55 |
| OpenRouter: Responses route | OpenRouter | Inferred: `/responses` returns `encrypted_content` for stateless turns, without WebSocket | see OpenRouter evidence | Reuses the OpenAI Responses config with another base URL | Same handling as the OpenAI Responses route | Inferred from LiteLLM's code only | OR-inferred | K55 |

### Where LiteLLM handles each feature

One mark per row. "Inside" means wholly inside the translator. "LiteLLM field back" means the caller must return a LiteLLM-specific field unchanged. "Vendor params" means the caller must pass vendor-shaped parameters. "Left out" means LiteLLM leaves it to the caller or drops it.

| Vendor feature | Inside | LiteLLM field back | Vendor params | Left out | Evidence |
|---|---|---|---|---|---|
| Anthropic signed and redacted thinking | | `thinking_blocks` | | | K6, K7 |
| Anthropic block order with server tools | Rebuilt by pattern | `thinking_blocks`, `provider_specific_fields` | | Real order | K7 |
| Anthropic thinking missing on tool loop | Yes, with `modify_params` | | | | K9 |
| Anthropic compaction blocks | | `provider_specific_fields.compaction_blocks` | `context_management` | | K6, K7 |
| Gemini signature on function calls | Yes for clients that echo ids unchanged | Tool call id suffix, or `provider_specific_fields.thought_signature` | | | K10, K12 |
| Gemini signature on text parts | | `provider_specific_fields.thought_signatures` | | All but the first | K12 |
| Gemini signature from another backend or model | Yes | | | | K13 |
| Gemini server-side tool invocations | | `provider_specific_fields.server_side_tool_invocations` | `include_server_side_tool_invocations` | | K14 |
| OpenAI reasoning items | | `reasoning_items`, `provider_specific_fields.openai_responses_replay` | `include`, `store` (Responses params) | | K17, K19 |
| OpenAI Responses-only models | Yes | | | `previous_response_id` | K15, K16 |
| OpenAI encrypted content account binding | Yes, in the router | | | | K20 |
| Bedrock reasoning with signature | | `thinking_blocks` | | Non-Claude targets | K21 |
| DeepSeek reasoning pass-back | Padded with a space | `reasoning_content` | | | K22 |
| Reasoning effort, all vendors | Yes | | Optional `thinking` in Anthropic shape | | K25-K28 |
| Anthropic cache breakpoints | Yes with injection points | | `cache_control` markers | | K30, K34 |
| Bedrock cache points | Yes | | `cache_control` markers | | K31 |
| Gemini cache object | Yes | | `cache_control` markers | | K32 |
| OpenAI cache key and breakpoints | Yes | | `prompt_cache_key`, `prompt_cache_breakpoint` | | K33 |
| Cache usage accounting | Yes | | | | K35 |
| Tool call id format | Yes | | | Mistral nine characters | K36 |
| `tool_choice`, `parallel_tool_calls` | Yes | | | | K37 |
| Tool schema dialect | Yes | | | | K38 |
| Images in tool results | Yes | | | | K39 |
| Structured output | Yes | | | | K41, K42 |
| Refusal | Yes | | | | K43 |
| Web search request | Yes | | | | K44 |
| Anthropic server tool replay and citations | | `provider_specific_fields` | | Citations as `annotations` | K45 |
| Finish reasons | Yes | | | | K46 |
| Error classes | Yes | | | | K47 |
| Unsupported parameters | Yes | | | | K49 |
| Stripping LiteLLM fields for strict upstreams | Six providers | | | Shared OpenAI-compatible path | K24 |
| OpenRouter `reasoning_details` | | | | Yes | K51 |

## Evidence

All code quotes are from `BerriAI/litellm` at commit `90dce4b93bd2b090621a6b9083311294d3358bc6`, read 2026-11-22. Line numbers are from that commit. Unless an item says otherwise its basis is `Filled (LiteLLM)`.

### Axis 1 - Opaque vendor state on a chat-completions message

**K1. LiteLLM's response message adds four non-OpenAI fields for vendor state, and the stream delta has the same four.**

- Quote, `Message` class (K-s1, lines 1554-1558): `reasoning_content: str | None = None`, `thinking_blocks: list[ChatCompletionThinkingBlock | ChatCompletionRedactedThinkingBlock] | None = None`, `reasoning_items: list[ChatCompletionReasoningItem] | None = None`, `provider_specific_fields: dict[str, Any] | None = Field(default=None)`, `annotations: list[ChatCompletionAnnotation] | None = None`.
- Quote, `Delta` class (K-s1, lines 1681-1684): the same `reasoning_content`, `thinking_blocks`, `reasoning_items`, `provider_specific_fields`.
- Each field is deleted from the object when empty, with the comment `# ensure default response matches OpenAI spec` (K-s1, lines 1630-1643).
- A delta that arrives with `reasoning` is renamed: `# Map 'reasoning' to 'reasoning_content' for providers that return` / `# delta.reasoning (e.g., Cerebras, Groq gpt-oss models).` (K-s1, lines 1700-1705).
- Source: K-s1. Basis: Filled (LiteLLM).

**K2. The three typed fields have three different shapes, two of them copied from vendors.**

- `ChatCompletionThinkingBlock`: `type: Required[Literal["thinking"]]`, `thinking: str`, `signature: str | None`, `cache_control: dict | ChatCompletionCachedContent | None`, `index: ReadOnly[int]` (K-s2, lines 736-741). This is Anthropic's block.
- `ChatCompletionRedactedThinkingBlock`: `type: Required[Literal["redacted_thinking"]]`, `data: str` (K-s2, lines 744-748).
- `ChatCompletionReasoningItem`: docstring `"""Represents an OpenAI Responses API reasoning item for round-tripping in conversation history."""`, fields `type: Required[Literal["reasoning"]]`, `id: str`, `encrypted_content: str | None`, `summary: ReadOnly[list[ChatCompletionReasoningSummaryTextBlock]]` (K-s2, lines 756-762).
- The request-side assistant message type accepts them back: `thinking_blocks: list[ChatCompletionThinkingBlock | ChatCompletionRedactedThinkingBlock] | None` and `reasoning_items: list[ChatCompletionReasoningItem] | None` (K-s2, lines 965-968).
- A tool call's function chunk has its own `provider_specific_fields: dict[str, Any] | None` (K-s2, lines 685-688).
- Source: K-s2. Basis: Filled (LiteLLM).

**K3. Answer to "one neutral field or one field per vendor": one plain-text field shared by all (`reasoning_content`), then one typed field per vendor family, then one untyped dictionary for everything else.**

| Vendor | Where the opaque state sits on the chat message | Evidence |
|---|---|---|
| Anthropic | `thinking_blocks` (signature, redacted data), `provider_specific_fields` keys `compaction_blocks`, `web_search_results`, `tool_results`, `server_tool_calls`, `citations`, `container`, `context_management` | K6 |
| Bedrock Converse | `thinking_blocks` (same shape), `provider_specific_fields.reasoningContentBlocks`, `provider_specific_fields.citationsContent` | K21 |
| Gemini | Tool call id suffix `__thought__<base64>`, `tool_calls[].provider_specific_fields.thought_signature`, `provider_specific_fields.thought_signatures`, `provider_specific_fields.server_side_tool_invocations`, `provider_specific_fields.continuation_token`, and `thinking_blocks` for thought parts | K10, K14 |
| OpenAI and Azure (Responses) | `reasoning_items`, `provider_specific_fields.openai_responses_replay`, `provider_specific_fields.phase`, `provider_specific_fields.tool_search_items` | K17 |
| DeepSeek, Mistral, others | `reasoning_content` text | K22, K23 |

- This table is a synthesis of the items it cites. Confidence: high for the listed keys, each was read in code. Basis: Filled (LiteLLM).

**K4. The vendor's own stop reason survives as `provider_specific_fields.native_finish_reason` on the choice when the mapped value differs.**

- Quote (K-s1, lines 1822-1827): `mapped: Final = map_finish_reason(finish_reason)` / `params["finish_reason"] = mapped` / `if finish_reason != mapped:` / `provider_specific_fields["native_finish_reason"] = finish_reason`.
- Source: K-s1. Basis: Filled (LiteLLM).

**K5. LiteLLM's own documentation states the contract and its weak point: clients built for OpenAI drop the extra field.**

- Quote: "LiteLLM will standardize the `reasoning_content` in the response and `thinking_blocks` in the assistant message."
- Quote: "When using Anthropic models with `thinking` enabled and tool calling, you must include `thinking_blocks` from the previous assistant response when sending tool results back. Failure to do so will result in a `400 Bad Request` error."
- Quote: "OpenAI's Chat Completions spec has no field for `thinking_blocks`" and "OpenAI-compatible clients (LibreChat, Open WebUI, Vercel AI SDK, etc.) ignore the `thinking_blocks` field in responses" and "When these clients reconstruct the assistant message for the next turn, the thinking blocks are lost".
- Quote, spec section: "`thinking_blocks` - Optional[List[Dict[str, str]]]: A list of thinking blocks from the model. Only returned for Anthropic models."
- Source: K-s34. Basis: Filled (LiteLLM documentation).

**K6. Anthropic reply: text blocks are joined into one string, thinking and redacted thinking blocks go into one list, and everything else goes into `provider_specific_fields`. Only these per-type lists survive, and the position of each block is rebuilt later by rule (K7).**

- Quote (K-s3, lines 2969-3000): `if content["type"] == "text":` / `text_content += content["text"]`, then `elif content.get("type") == "thinking":` / `thinking_blocks.append(cast(ChatCompletionThinkingBlock, content))`, then `elif content["type"] == "redacted_thinking":` / `thinking_blocks.append(cast(ChatCompletionRedactedThinkingBlock, content))`.
- Quote, compaction (K-s3, lines 3003-3006): `elif content["type"] == "compaction":` / `compaction_blocks.append(content)`.
- `reasoning_content` is the thinking text of all blocks joined (K-s3, lines 3021-3026).
- `provider_specific_fields` keys set on the message (K-s3, lines 3323-3368): `"citations"`, `"thinking_blocks"`, `"context_management"`, `"input_transformations"`, `"web_search_results"`, `"web_search_calls"`, `"tool_results"`, `"code_interpreter_results"`, `"container"`, `"compaction_blocks"`, `"server_tool_calls"`, plus stop reason fields.
- The message is built with both: `thinking_blocks=thinking_blocks, reasoning_content=reasoning_content` and `_message.provider_specific_fields = provider_specific_fields` (K-s3, lines 3447-3455).
- Source: K-s3. Basis: Filled (LiteLLM).

**K7. Anthropic request: the translator rebuilds the assistant content from the separate fields. Thinking goes first. With server tool calls, the order is rebuilt by a fixed pattern, not from a recorded order.**

- Quote, sequential mode (K-s4, lines 3688-3689): `if not _content_is_list:  # IMPORTANT: ADD THIS FIRST, ELSE ANTHROPIC WILL RAISE AN ERROR` / `assistant_content.extend(replayed_thinking_blocks)`.
- Quote, interleaved mode comment (K-s4, lines 3580-3585): `# INTERLEAVED MODE: When we have both thinking blocks and server` / `# tool calls (e.g. web search), Anthropic's original response` / `# interleaves them: [thinking_1, server_tool_use_1, result_1,` / `# thinking_2, text, server_tool_use_2, result_2, ...].` / `# We must preserve this interleaved order because Anthropic` / `# verifies thinking block signatures based on position.`
- Quote, the rebuild rule (K-s4, lines 3626-3629): `# Interleave: each thinking block precedes its server tool group.` / `# Pattern: thinking[0], group[0], thinking[1], group[1], ...` / `# Any remaining thinking blocks (after all groups) go before text.` / `# Any remaining groups (after all thinking blocks) go after.`
- Server tool calls are recognized by id prefix: `_tc_id.startswith("srvtoolu_")` (K-s4, line 3545). Results come from the message's own extension field: `web_search_results=_provider_specific_fields.get("web_search_results")`, `tool_results=_provider_specific_fields.get("tool_results")` (K-s4, lines 3559-3562).
- Compaction blocks are read from `provider_specific_fields` and placed first: `# Add compaction blocks at the beginning of assistant content` (K-s4, lines 3521-3527).
- Confidence note: the claim that the pattern can differ from the real order is an inference from the code (the comment's own example puts `text` between `thinking_2` and `server_tool_use_2`, while the rebuild puts text after all groups). Issue 23047 (K57) shows the earlier, simpler rebuild failing.
- Source: K-s4. Basis: Filled (LiteLLM).

**K8. Before sending to Anthropic, LiteLLM drops thinking blocks that Anthropic cannot verify: unsigned blocks and blocks that hold another vendor's encrypted reasoning.**

- Quote, docstring (K-s5, lines 197-204): "Anthropic verifies the signature cryptographically, so a block with a null, empty, or missing signature (e.g. from an open-source reasoning model) is rejected with a 400, and so is a block whose signature or data carries another provider's encrypted reasoning. A signed block is kept whatever its text, since ``display: omitted`` returns signed blocks with empty ``thinking``. `redacted_thinking` blocks carry no signature and are always kept."
- The filter is applied in `_replayable_thinking_blocks` (K-s4, lines 3149-3159), which also strips the stream-only `index` key: `_STREAM_ONLY_THINKING_KEYS: Final = frozenset({"index"})` (K-s4, line 3256).
- Source: K-s4, K-s5. Basis: Filled (LiteLLM).

**K9. When the caller lost the thinking blocks, LiteLLM can turn thinking off for that one request, but only under an opt-in global flag.**

- Quote (K-s3, lines 2627-2646): `# Drop thinking param if thinking is enabled but thinking_blocks are missing` / `# This prevents the error: "Expected thinking or redacted_thinking, but found tool_use"` and then `if litellm.modify_params:` / `optional_params.pop("thinking", None)` with the warning `"Dropping 'thinking' param because the last assistant message with tool_calls "` `"has no thinking_blocks. The model won't use extended thinking for this turn."`
- The check skips adaptive thinking: `case None | {"type": "adaptive"}:` / `return False` (K-s6, lines 9828-9833).
- The guard against the opposite error is quoted in K-s6, lines 9783-9784: `"When thinking is disabled, an assistant message cannot contain thinking"`.
- Source: K-s3, K-s6. Basis: Filled (LiteLLM).

**K10. Gemini reply: a thought signature is stored in up to four places at once.**

- Thought parts (`if part.get("thought") is True and "inlineData" not in part:`) become thinking blocks of type `"thinking"` with the signature: `signature = part.get("thoughtSignature")` / `if signature is not None:` / `block["signature"] = signature` (K-s7, lines 2374-2384).
- Every signature on any part goes into a flat list: docstring "can appear on parts even without thought: true (e.g., regular text responses, function calls). This method extracts all thoughtSignature values from parts." (K-s7, lines 2387-2402), stored as `("thought_signatures", thought_signatures)` in `message_provider_fields` (K-s7, lines 3315-3324).
- On the tool call: `_tool_response_chunk["provider_specific_fields"] = {"thought_signature": thought_signature}` (K-s7, lines 2546-2547).
- Inside the tool call id: `if thought_signature and thought_signature_in_tool_call_id:` / `_tool_response_chunk["id"] = encode_tool_call_id_with_signature(` (K-s7, lines 2548-2551). The format is in the docstring: "Embed thought signature into tool call ID for OpenAI client compatibility." and "Format: call_<uuid>__thought__<base64_signature>" (K-s4, lines 1262-1271). The separator constant is `THOUGHT_SIGNATURE_SEPARATOR: Final = "__thought__"` (K-s4, line 103).
- The id embedding is on by default. A deployment parameter `thought_signature_in_tool_call_id` turns it off (`thought_signature_in_tool_call_id: bool = True` at K-s7 line 2507; the parameter is declared in `litellm/types/router.py` line 666 and read in `litellm/litellm_core_utils/model_param_helper.py` lines 49-51, K-s55).
- Source: K-s4, K-s7, K-s55. Basis: Filled (LiteLLM).

**K11. Gemini ids: older Gemini returns function calls without ids, so LiteLLM mints them. Newer Gemini ids are kept and sent back.**

- Quote (K-s7, lines 2535-2545): `"id": f"call_{uuid.uuid4().hex[:28]}",` and the comment `# Gemini 3.5+ returns a stable native `id`; prefer it over` / `# the synthetic call_<uuid> so the same value can be echoed` / `# back on the matching `functionResponse`.`
- Quote (K-s7, lines 1015-1022): "Gemini 3+ accepts (and returns) `id` for strict tool-call matching, on Vertex AI and Google AI Studio alike. Older Gemini models reject the field with HTTP 400."
- For a tool result, Gemini needs the function name, so LiteLLM looks it up from the id in the last assistant message, and fails when it finds none: `f"Missing corresponding tool call for tool response message. Received - message={message}, last_message_with_tool_calls={last_message_with_tool_calls}"` (K-s4, lines 1758-1779).
- Source: K-s4, K-s7. Basis: Filled (LiteLLM).

**K12. Gemini request: the signature is looked up in three places, in order, and a placeholder is used on Gemini 3+ when it is missing.**

- Quote, docstring (K-s4, lines 1308-1314): "Extract thought signature from tool call's provider_specific_fields." / "If not provided try to extract thought signature from tool call id" / "Checks both tool.provider_specific_fields and tool.function.provider_specific_fields." / "Returns None when the tool call carries no signature or its id-embedded signature is not valid base64; callers decide whether a placeholder signature is needed."
- The id-embedded value is checked with strict base64: `return signature if _is_strict_base64(signature) else None` (K-s4, line 1358).
- Quote, placeholder (K-s4, lines 1365-1379): "This is used when transferring conversation history from older models (like gemini-2.5-flash) to gemini-3, which requires thought_signature for strict validation. Google documents it as a last resort that "will negatively impact model performance", so callers must only fall back to it when no real signature is available." The value is `dummy_data: Final = b"skip_thought_signature_validator"`, base64 encoded.
- Only the first call of a message gets the placeholder: `# Gemini signs only the first functionCall part of a parallel batch, so scope the` / `# placeholder fallback to that part instead of fabricating one per sibling call:` (K-s4, lines 1450-1455).
- Text parts: `# Use the first signature for the text part (Gemini expects one signature per part)` with `thoughtSignature=thought_signatures[0]` (K-s8, lines 920-931). The reason one signature must not be sent twice: `# Gemini bills every replayed copy as the previous turn's full` / `# reasoning token count on gemini-3 and newer models` (K-s8, lines 916-917).
- Reasoning text is replayed as a thought part: `assistant_content.append(PartType(thought=True, text=reasoning_content))` (K-s8, lines 888-890).
- `thinking_blocks` are written on the Gemini reply, and the Gemini request code builds its thought parts from `reasoning_content` instead: a search for `thinking_blocks` under `litellm/llms/vertex_ai` matches only reply code in K-s7. Confidence: medium, this rests on a text search.
- Source: K-s4, K-s7, K-s8. Basis: Filled (LiteLLM).

**K13. Gemini history that came from another model or another Google backend is handled inside the translator, in three ways.**

- To a non-Gemini target, the signature is cut from the ids: docstring "Messages with Gemini thought signatures cut from tool call ids, or None when the call's target keeps them." and log line `"Removing thought signatures from tool call IDs for non-Gemini model %s"` (K-s6, lines 1146-1167).
- Between Vertex and AI Studio, a rejected signature triggers one resend with the placeholder. Quote (K-s9, lines 22-26): "A signature only validates on the backend that minted it. AI Studio answers a replayed Vertex signature with ``400 Corrupted thought signature.`` and Vertex answers an AI Studio one with ``400 Invalid thought signature.``" The match is on text: `_REJECTED_SIGNATURE_MARKER: Final = "thought signature"` and `return status_code == 400 and _REJECTED_SIGNATURE_MARKER in message.lower()` (K-s9, lines 17, 26).
- Quote, the resend (K-s9, lines 63-64): "Swaps every thought signature in the history for Google's documented skip-validator placeholder, or returns ``None`` when the body carries nothing the backend could have rejected."
- History with no signature at all, sent to Gemini 3+, gets the placeholder (K12).
- Source: K-s6, K-s9. Basis: Filled (LiteLLM).

**K14. Gemini server-side tool calls (for example Google Search with context circulation) ride in `provider_specific_fields.server_side_tool_invocations` and must come back from the caller.**

- Quote, docstring (K-s7, lines 2408-2416): "These are returned by Gemini when context circulation is enabled (includeServerSideToolInvocations=true). They represent tools executed server-side (e.g. Google Search) and must be circulated back in subsequent turns for multi-turn coherence."
- Each entry holds `tool_type`, `id`, `args`, `response`, `thought_signature`, `response_thought_signature` (K-s7, lines 2425-2456).
- Replay (K-s8, lines 998-1027): `# Re-inject toolCall part` and `# Re-inject toolResponse part if response is present`, appended after the function call parts.
- Source: K-s7, K-s8. Basis: Filled (LiteLLM).

**K15. OpenAI: a chat-completions call is silently rerouted to the Responses API for Responses-only models and for some feature mixes.**

- The decision function is `responses_api_bridge_check` (K-s12, lines 156-286). Triggers read in code: a global flag `litellm.route_all_chat_openai_to_responses` (line 170), a `responses/` model prefix or `mode == "responses"` in the model table (lines 183-189), xAI with `web_search_options` (lines 199-201), and GPT-5 family combinations (lines 272-285).
- Quote, comment (K-s12, lines 203-207): `# Function tools with explicit active reasoning bridge on GPT-5.4+. With effort unset,` / `# OpenAI/Azure and Responses-capable Mantle endpoints bridge only GPT-5.6+ whose map` / `# default is not "none". Custom OpenAI-compatible endpoints keep unset effort on chat.`
- Quote, docstring (K-s12, lines 120-123): "Responses-only backends (``gpt-5.1-codex``, ``o3-pro``, ``gpt-5.4-pro``) list only ``/v1/responses`` and must stay bridged".
- Confidence note: that the OpenAI chat endpoint rejects function tools while reasoning on those models is LiteLLM's claim in code (`chat_rejects_function_tools`), not checked here against OpenAI's documentation.
- Source: K-s12. Basis: Filled (LiteLLM).

**K16. The bridge turns messages into Responses input items and refuses server-side conversation state.**

- System: leading system strings become `instructions`, joined with a space (K-s10, lines 1218-1224). Later ones become input items with `"type": "message",` and `"role": role,`.
- Tool results: a `FunctionCallOutput` with `type="function_call_output",` / `call_id=tool_call_id,` / `output=tool_output,` (K-s10, lines 1268-1274).
- Assistant with tool calls: reasoning items first, then the text as a `message` item, then one item per call with `"type": "function_call",` / `"call_id": tool_call["id"],` plus `name` and `arguments` (K-s10, lines 1275-1307).
- Quote, refusal (K-s10, lines 273-276): "`previous_response_id` is only supported on the Responses API (`/v1/responses`). Continue the stored response there, or send the earlier turns in `messages`." It is raised as `litellm.UnsupportedParamsError` (K-s10, lines 1447-1450).
- Source: K-s10. Basis: Filled (LiteLLM).

**K17. OpenAI reasoning items come back on the chat message in a third field, `reasoning_items`, and an order map with a checksum is stored beside them so the next request can rebuild the exact item sequence.**

- Reply (K-s10, lines 1609-1642): each `ResponseReasoningItem` is built into a dict with the keys `"id": item_id,` / `"type": "reasoning",` / `"encrypted_content": encrypted_content,` / `"summary": summary,` (lines 409-414) and attached to the `Message` through its `reasoning_items` argument, with the summary text as `reasoning_content`.
- The order map: `REPLAY_FIELD: Final = "openai_responses_replay"` (K-s11, line 11), with the model `version: Literal[1]`, `items: tuple[_Entry, ...]`, `checksum: str` (K-s11, lines 72-75).
- The checksum binds the map to the message content: sha256 over `"content"`, `"tool_calls"`, `"reasoning"`, `"tool_search_items"`, `"phase"`, `"items"` (K-s11, lines 228-242).
- On the next request the map is used only when the message is unchanged: `if chat is None or stored is None or stored.checksum != _checksum(chat, stored.items):` / `return None` (K-s11, lines 449-456). Then the generic order of K16 applies.
- It is on for two providers only: `_HISTORY_REPLAY_PROVIDERS: Final = frozenset({"openai", "azure"})` (K-s10, line 271).
- Source: K-s10, K-s11. Basis: Filled (LiteLLM).

**K18. When the history holds Anthropic-shaped `thinking_blocks` and no `reasoning_items`, the bridge rebuilds reasoning items from them. OpenAI's `encrypted_content` can travel inside an Anthropic-shaped block under a LiteLLM prefix.**

- Quote (K-s10, lines 374-378): "Stored reasoning items win because they carry an id the Responses API minted; thinking blocks are the fallback for turns that arrived over another API surface."
- Quote (K-s5, lines 63-73): `ENCRYPTED_REASONING_SIGNATURE_PREFIX: Final = "litellm_encrypted_reasoning:"` and "Anthropic clients echo a thinking block's `signature` and a redacted block's `data` back verbatim, so either field can carry the encrypted reasoning across turns; the prefix tells the two apart from a signature Anthropic minted."
- Quote (K-s5, lines 322-327): "A block carrying encrypted reasoning replays the item it came from byte for byte; a run of plain thinking blocks collapses into one summary-only item. No item carries an `id`: the Responses API 404s on any id it did not mint itself and rejects an empty one, while an item without an id is always accepted."
- Source: K-s5, K-s10. Basis: Filled (LiteLLM).

**K19. On the chat bridge, asking OpenAI to return `encrypted_content` is the caller's job at this commit.**

- A search for `reasoning.encrypted_content` in all of `litellm/` matches three files, all outside the chat bridge folder `litellm/completion_extras/`: `router_utils/pre_call_checks/encrypted_content_affinity_check.py`, `llms/chatgpt/responses/transformation.py`, `llms/anthropic/messages/responses_adapters/transformation.py`.
- Responses parameters named by the caller pass through the bridge: `elif key in ResponsesAPIOptionalRequestParams.__annotations__:` / `responses_api_request[key] = value` (K-s10, lines 1373-1374).
- Confidence: medium. This rests on a text search, and a closed pull request (22448, read only as a search extract, K-s50) once described automatic injection: "`transform_request()` auto-injects `include: ["reasoning.encrypted_content"]` when encrypted reasoning items are detected in the input".
- Source: K-s10, K-s50. Basis: Filled (LiteLLM).

**K20. OpenAI's encrypted reasoning only decrypts for the account that made it, so LiteLLM's router marks each item with the deployment that produced it and pins the next request there.**

- Quote, file docstring (K-s13, lines 4-8): "When Codex or other models use `store: false` with `include: ["reasoning.encrypted_content"]`, the response output items contain encrypted reasoning tokens tied to the originating organization's API key. If a follow-up request containing those items is routed to a different deployment (different org), OpenAI rejects it with an `invalid_encrypted_content` error because the organization_id doesn't match."
- Quote (K-s13, lines 13-15): "**Items with IDs**: Encode model_id into the item ID itself (e.g., ``encitem_...``)" and "**Items without IDs** (Codex): Wrap the encrypted_content with model_id metadata (e.g., ``litellm_enc:{base64_metadata};{original_encrypted_content}``)".
- When no deployment can decrypt, the reasoning is removed and the request goes on: `"encryption boundary is configured; forwarding without its encrypted reasoning"` (K-s13, lines 705-710).
- The error mapper recognizes the failure by text: `elif "invalid_encrypted_content" in error_str or "could not be verified" in error_str:` (K-s27, line 484).
- Source: K-s13, K-s27. Basis: Filled (LiteLLM).

**K21. Bedrock Converse reasoning uses the same `thinking_blocks` field as Anthropic, and what may be replayed depends on which model sits behind Converse.**

- Reply: docstring `"""Return a consistent format for thinking blocks between Anthropic and Bedrock."""`; `reasoningText.text` and `reasoningText.signature` become `thinking` and `signature`, and `redactedContent` becomes a redacted block built with `type="redacted_thinking", data=native_redacted_thinking_data(block["redactedContent"])` (K-s14, lines 2912-2932). The raw blocks are also kept: `provider_specific_fields["reasoningContentBlocks"] = reasoningContentBlocks` (K-s14, lines 3418-3419).
- Request: a `BedrockConverseReasoningTextBlock` with the text and `signature=signature`, or `return BedrockConverseReasoningContentBlock(redactedContent=converse_redacted_content(data))` (K-s4, lines 5957-5976).
- Quote (K-s15, lines 1617-1626): "Whether ``reasoningContent.redactedContent`` may be replayed to this Converse target. Only Claude takes it back. Nova, Llama, DeepSeek and gpt-oss 400 on it (checked live in us-west-2 on 2026-10-14), so every other target keeps dropping the block."
- Signed blocks are dropped for non-Claude targets through `bedrock_converse_rejects_signed_thinking` (K-s15, lines 1629-1632; used at K-s4, lines 5643 and 5819).
- Source: K-s4, K-s14, K-s15. Basis: Filled (LiteLLM).

**K22. DeepSeek needs the reasoning text itself sent back, and LiteLLM pads it when the caller lost it.**

- Quote, docstring (K-s23, lines 95-104): "DeepSeek thinking mode requires `reasoning_content` to be passed back on every assistant message in multi-turn conversations. If it is missing, the API returns: "The reasoning_content in the thinking mode must be passed back to the API."" The docstring then says LiteLLM first looks for the text in `provider_specific_fields["reasoning_content"]` and otherwise injects a single space, which it calls the minimum value the API accepts.
- Quote, warning (K-s23, lines 119-126): `"DeepSeek thinking mode: assistant message is missing "` `"`reasoning_content` and none was saved in "` `"`provider_specific_fields`. A single-space placeholder "` `"is being injected to satisfy API validation, but the "` `"model will receive a blank reasoning chain for this turn, "` `"which may silently degrade multi-turn response quality. "`
- Source: K-s23. Basis: Filled (LiteLLM).

**K23. Mistral reasoning is replayed as a `thinking` content chunk rebuilt from `reasoning_content`, or from the text of `thinking_blocks`.**

- Code (K-s22, lines 621-628 and 737-746): `replayed_reasoning_text` reads `reasoning_content`, else joins `thinking_blocks` text, then builds a `MistralThinkingBlock` with `type="thinking", thinking=[MistralTextBlock(type="text", text=reasoning_text)], closed=True` and puts it before the content parts.
- Source: K-s22. Basis: Filled (LiteLLM).

**K24. LiteLLM's own extension fields have to be removed before a request goes to any other OpenAI-compatible upstream, and that removal is done provider by provider.**

- The field list: `LITELLM_RESPONSE_ONLY_MESSAGE_FIELDS` is the frozen set `{"thinking_blocks", "reasoning_items", "provider_specific_fields"}`, and `LITELLM_INTERNAL_MESSAGE_FIELDS: Final = LITELLM_RESPONSE_ONLY_MESSAGE_FIELDS | {"reasoning_content"}` (K-s20, lines 218-223).
- Quote, docstring (K-s21, lines 1360-1361): "Drop the fields litellm attaches to a returned message (e.g. Anthropic thinking blocks or Responses API items) that no OpenAI-compatible endpoint accepts back, so a reply can be sent to a strict one."
- The two strip functions are called from six provider transforms: `together_ai`, `mistral`, `groq`, `friendliai`, `fireworks_ai`, `databricks` (text search for `strip_litellm_response_only_fields` and `strip_litellm_internal_message_fields` under `litellm/`).
- The shared OpenAI-compatible request builder sends messages with whatever keys they have: `transform_request` returns a body with `"model": model,` / `"messages": transformed_messages,` / `**optional_params,` after `_transform_messages`, which only rewrites tool-message media and user content parts (K-s25, lines 476-488 and 694-730).
- Confidence: medium on "only six". Other providers strip inline (for example `hosted_vllm` pops `thinking_blocks`), so the count is of callers of the shared helpers.
- Source: K-s20, K-s21, K-s25. Basis: Filled (LiteLLM).

### Axis 2 - Reasoning controls

**K25. The effort-to-budget numbers are constants, each overridable by an environment setting of the same name.**

- Quote (K-s17, lines 234, 325-380): `DEFAULT_REASONING_EFFORT_DISABLE_THINKING_BUDGET` 0, `DEFAULT_REASONING_EFFORT_MINIMAL_THINKING_BUDGET_GEMINI_2_5_FLASH` 1, `DEFAULT_REASONING_EFFORT_MINIMAL_THINKING_BUDGET_GEMINI_2_5_PRO` 128, `DEFAULT_REASONING_EFFORT_MINIMAL_THINKING_BUDGET_GEMINI_2_5_FLASH_LITE` 512, `DEFAULT_REASONING_EFFORT_MINIMAL_THINKING_BUDGET` 128, `DEFAULT_REASONING_EFFORT_LOW_THINKING_BUDGET` 1024, `DEFAULT_REASONING_EFFORT_MEDIUM_THINKING_BUDGET` 2048, `DEFAULT_REASONING_EFFORT_HIGH_THINKING_BUDGET` 4096, `DEFAULT_REASONING_EFFORT_XHIGH_THINKING_BUDGET` 8192, `DEFAULT_REASONING_EFFORT_MAX_THINKING_BUDGET` 16384.
- Also `ANTHROPIC_MIN_THINKING_BUDGET_TOKENS: Final = 1024` (K-s17, line 656) and `DEFAULT_ANTHROPIC_CHAT_MAX_TOKENS` with default 128000 (K-s17, line 661).
- Source: K-s17. Basis: Filled (LiteLLM).

**K26. Anthropic: `reasoning_effort` becomes adaptive thinking plus `output_config.effort` on models flagged adaptive, and a token budget on older models. The flags live in LiteLLM's model table, not in code.**

- Quote (K-s3, lines 1867-1873): `if AnthropicConfig._is_adaptive_thinking_model(model, custom_llm_provider):` / `# without display, Anthropic defaults adaptive thinking to` / `# display="omitted" and returns a blank thinking block` / `return AnthropicThinkingParam(` / `type="adaptive",` / `display="summarized",` / `)`.
- Older models: `"low"` gives `budget_tokens=DEFAULT_REASONING_EFFORT_LOW_THINKING_BUDGET`, and so on through `"max"`; `"minimal"` gives the larger of `DEFAULT_REASONING_EFFORT_MINIMAL_THINKING_BUDGET` and `ANTHROPIC_MIN_THINKING_BUDGET_TOKENS`, which is 1024 (K-s3, lines 1874-1906).
- Effort table for adaptive models (K-s3, lines 638-645): `"low": "low"`, `"minimal": "low"`, `"medium": "medium"`, `"high": "high"`, `"xhigh": "xhigh"`, `"max": "max"`, sent as `optional_params["output_config"] = {"effort": mapped_effort}` (K-s3, line 2280).
- `"none"` sends `AnthropicThinkingParam(type="disabled")` only for models that think by default and can be turned off (K-s3, lines 1861-1866).
- Per-model flags read from the cost map: `supports_adaptive_thinking`, `thinking_always_on`, `thinking_on_by_default`, `supports_legacy_thinking`, `supports_output_config`, `supports_{level}_reasoning_effort`. Quote (K-s16, lines 1177-1183): "The model cost map is authoritative: an explicit ``supports_adaptive_thinking`` entry resolved under ``custom_llm_provider``, or a ``fallback_generalizations`` rule for unknown Claude models. The version gate (>= 4.6, including provider-prefixed Bedrock/Vertex ids that map to no exact entry) lives entirely in that declarative rule, not here."
- A caller's adaptive `thinking` sent to an older model is translated down to a medium budget, capped below `max_tokens` (K-s3, lines 2206-2236). A legacy budget sent to a model that rejects it is translated up (K-s16, lines 1285-1290).
- An unknown effort is an error: `f"Unmapped reasoning effort: {reasoning_effort!r}. "` `f"Must be one of: 'minimal', 'low', 'medium', 'high', "` `f"'xhigh', 'max', 'none'."` (K-s3, lines 1909-1913).
- Source: K-s3, K-s16. Basis: Filled (LiteLLM).

**K27. Gemini: Gemini 3 and newer get a thinking level, older models a token budget. The generation is read from the model name.**

- Levels (K-s7, lines 1766-1785): `"minimal"` gives `return {"thinkingLevel": "minimal", "includeThoughts": sends_thoughts}` if the model supports it, else `"low"`; `"low"`, `"medium"`, `"high"` map to themselves; `"xhigh"` only where the model table says so; `("disable", "none")` gives `"thinkingLevel": "minimal" if supports_minimal else "low"` with `"includeThoughts": False`.
- Budgets (K-s7, lines 1686-1726): the constants of K25, with `"none"` giving `"thinkingBudget": 0`.
- The generation test is a regular expression on the name: `if re.match(r"^gemini-(?:[12](?:\.\d+)?|exp|(?:pro|flash)(?!-(?:lite-)?latest$))(?:-|$)", model_name):` / `return False` (K-s7, lines 1002-1005).
- An Anthropic-shaped `thinking` param is accepted too. On Gemini 3+ its `budget_tokens` is dropped with a warning (K-s7, lines 1848-1862). Sending both a budget and a level is an error: `"Cannot specify both `thinking_level` and `thinking_budget` in the same request. "` (K-s7, lines 1824-1826).
- Source: K-s7. Basis: Filled (LiteLLM).

**K28. Other vendors each need their own small rule.**

- OpenAI on the Responses bridge: `responses_api_request["reasoning"] = self.map_reasoning_effort(value, model, custom_llm_provider)`, which builds `Reasoning(effort=reasoning_effort)` or adds a `summary` (K-s10, lines 1375-1376 and 2170-2184).
- DeepSeek (K-s23, lines 24-25 and 47-63): `SUPPORTED_REASONING_EFFORTS: Final = ("minimal", "low", "medium", "high", "xhigh", "max", "ultra")` and `_NO_BUDGET_REASON: Final = "DeepSeek takes thinking only as enabled or disabled, with no budget."` The docstring: "A thinking param wins. Otherwise `reasoning_effort="none"` turns thinking off and any other effort turns it on, and a supported effort is sent only while thinking is on."
- xAI: `reasoning_effort` is in the supported list only `if litellm.supports_reasoning(model=model, custom_llm_provider=self.custom_llm_provider)` (K-s24, lines 190-192).
- Mistral: an Anthropic-shaped `thinking` becomes an effort (`{"type": "disabled"}` gives `"none"`, `{"type": "adaptive"}` gives `"medium"`), and the effort is moved to the nearest level the model declares (K-s22, lines 66-89).
- Bedrock Converse (K-s14, lines 756-853): Nova 2 gets a `"reasoningConfig"` object with `"type": "enabled",` and `"maxReasoningEffort": reasoning_effort,` and only `valid_values: Final = ["low", "medium", "high"]`; OpenAI models on Bedrock get `{"effort": reasoning_effort}` under `reasoning`; Claude gets the Anthropic mapping of K26.
- OpenRouter: see K51.
- Source: K-s10, K-s14, K-s22, K-s23, K-s24. Basis: Filled (LiteLLM).

**K29. LiteLLM's documentation describes a rounding rule for deployments that declare their own effort levels.**

- Quote: "The strength order, from weakest to strongest, is `minimal`, `low`, `medium`, `high`, `xhigh`, `max`. Rounding up is the default, and it sends the weakest declared level at least as strong as the request, or the strongest declared level when the request is above all of them."
- Quote: "`none` and any level name outside the strength order are sent unchanged, so a caller who turned reasoning off is never rounded up. With `drop_params: true`, a level the deployment does not declare is dropped instead".
- Source: K-s34. Basis: Filled (LiteLLM documentation).

### Axis 3 - Prompt caching

**K30. Anthropic: a `cache_control` object on a chat message or content part is copied onto the Anthropic block, and marks beyond four are trimmed.**

- Copy (K-s4, lines 2455-2468): `cache_control_param: Final = original_content_element.get("cache_control")` / `transformed_param: Final = ChatCompletionCachedContent(**cache_control_param)` / `anthropic_content_element["cache_control"] = transformed_param`.
- The marker type is `type: Literal["ephemeral"]` with `ttl: NotRequired[Literal["5m", "1h"]]` (K-s2, lines 711-713).
- Limit (K-s19, lines 75-77): `# Anthropic (and Bedrock Claude) reject requests with more than 4 cache_control` / `# breakpoints: "A maximum of 4 blocks with cache_control may be provided."` / `MAX_CACHE_CONTROL_BLOCKS: Final = 4`. Helper names `_without_oldest_marks` and `newest_marks_within_cache_control_limit` (K-s19, lines 259 and 324) show the newest marks are kept. Confidence: medium on the trimming order, taken from the helper names.
- Source: K-s2, K-s4, K-s19. Basis: Filled (LiteLLM).

**K31. Bedrock Converse: the same marker becomes a separate `cachePoint` block.**

- Code comment (K-s4, line 4827): `# Check for cache_control and add a separate cachePoint block`.
- Docs quote: "LiteLLM automatically translates OpenAI-format `cache_control` markers to Bedrock's native `cachePoint` format, so no changes are needed to your existing code if you're already using `cache_control`."
- Docs quote on which models get them: "OpenAI-family models on Bedrock never get them. Then `supports_bedrock_converse_cache_point` decides when the deployment's `model_info` or the cost map sets it to `true` or `false`".
- Source: K-s4, K-s35. Basis: Filled (LiteLLM).

**K32. Gemini: yes, LiteLLM creates a named `cachedContents` object from `cache_control` markers. It keys the object by a hash it stores in `displayName` and finds it again by listing caches.**

- What is cached (K-s18, `transformation.py` lines 114-117): "Returns the first continuous block of cached messages, plus every system message once any message is cached, since generateContent rejects a system_instruction beside a cachedContent, and the remaining messages."
- The key (K-s18, `vertex_ai_context_caching.py` lines 485-487): `generated_cache_key: Final = local_cache_obj.get_cache_key(` / `messages=cached_messages, tools=tools, tool_choice=tool_choice, model=target.model` / `)`. It is stored on the object: `displayName=cache_key` (K-s18, `transformation.py` lines 190-194).
- Lookup (K-s18, `vertex_ai_context_caching.py` line 228): "Currently, checks cache list, for cache key == displayName, since Google doesn't let us set the name of the cache (their API docs are out of sync with actual implementation)." A process-local map of known names is consulted first (`self._known_caches.get(scope, cache_key)`, line 248).
- TTL: the first `cache_control.ttl` of type `"ephemeral"`, matched by `^([0-9]*\.?[0-9]+)([smh])$` and sent as seconds (K-s18, `transformation.py` lines 55-107).
- Tools move into the cache object and out of the request: `for tool_param in _TOOL_PARAMS:` / `optional_params.pop(tool_param, None)` and `cached_content_request_body["tools"] = tools` (K-s18, `vertex_ai_context_caching.py` lines 505-506 and 521-523).
- Skipped silently when the block is too small or ends on a model turn: `f"Vertex AI context caching: cached content is below minimum token count ({min_tokens})."` and "The cachedContents API rejects contents ending on a model turn, which is how it classifies both assistant messages and tool results, with HTTP 400 "Requests ending with a model turn are not supported"." (K-s18, `vertex_ai_context_caching.py` lines 134-147; `transformation.py` lines 134-139).
- Docs quote on the cost: "Gemini 2.5 and newer already cache repeated prefixes implicitly, for free. Sending `cache_control` (by hand or through `cache_control_injection_points`) opts the request into explicit caching instead: the marked prefix is moved into a `cachedContents` object, so implicit caching no longer applies to it, and Google charges storage for each cache object."
- Source: K-s18, K-s35. Basis: Filled (LiteLLM).

**K33. OpenAI: `prompt_cache_key` and `prompt_cache_retention` pass through, and LiteLLM renames Anthropic's `cache_control` to OpenAI's `prompt_cache_breakpoint` and back. TTLs do not convert.**

- Supported parameter list includes `"prompt_cache_key"`, `"prompt_cache_retention"`, `"store"` (K-s25, lines 292-294).
- Types (K-s2, lines 716-733): `class PromptCacheBreakpoint(TypedDict):` `mode: ReadOnly[Literal["explicit"]]`, `ttl: NotRequired[ReadOnly[Literal["30m", "12h"]]]`; `class PromptCacheOptions(TypedDict, total=False):` `mode: ReadOnly[Literal["implicit", "explicit"]]`, `ttl: ReadOnly[Literal["30m", "12h"]]`.
- Anthropic mark to OpenAI mark (K-s20, lines 93-100): docstring "Anthropic's ``cache_control`` mark rewritten as OpenAI's ``prompt_cache_breakpoint`` on the same block." The new mark is `PromptCacheBreakpoint(mode="explicit")`, so the Anthropic TTL is not copied.
- OpenAI mark to Anthropic mark with a TTL is an error unless `drop_params` is set (K-s20, lines 135-145): `f"This model's prompt cache lasts 5m, or 1h where it offers one, so prompt_cache_breakpoint "` `f"ttl={ttl!r} has no equivalent. Remove the ttl, {PROMPT_CACHE_BREAKPOINT_DROP_PARAMS_HINT} to send "` `"a 5m breakpoint instead, or send cache_control {'type': 'ephemeral', 'ttl': '1h'} on the block "` `"where the model offers a 1h cache."`
- Model gate: `OPENAI_PROMPT_CACHE_BREAKPOINT_MIN_GPT_VERSION: Final = (5, 6)` (K-s19, line 80).
- Docs quote: "GPT-5.6 and newer also accept explicit cache breakpoints: a `prompt_cache_breakpoint` marker on a content block plus a request-level `prompt_cache_options` that picks the mode".
- Source: K-s2, K-s19, K-s20, K-s25, K-s35. Basis: Filled (LiteLLM).

**K34. Automatic injection: a deployment or request can name where LiteLLM should add cache marks, so the caller sends plain messages.**

- Quote, file docstring (K-s19, lines 2-8): "This hook is used to inject cache control directives into messages. Users can define - `cache_control_injection_points` in the completion params and litellm will inject the cache control directives into the messages at the specified injection points."
- Shape (K-s19, `litellm/types/integrations/anthropic_cache_control_hook.py` lines 13-35): a message point has `location: Literal["message"]`, `role: Literal["user", "system", "assistant"] | None`, `index: int | str | None`, `control: ChatCompletionCachedContent | None`; a tool point has `location: Literal["tool_config"]`.
- Source: K-s19. Basis: Filled (LiteLLM).

**K35. Usage: LiteLLM makes `prompt_tokens` include cached tokens for every vendor, and reports reads and writes under fixed names.**

- Anthropic (K-s3, lines 3186-3192): `cache_creation_input_tokens = _usage["cache_creation_input_tokens"]` / `prompt_tokens += cache_creation_input_tokens` and `cache_read_input_tokens = _usage["cache_read_input_tokens"]` / `prompt_tokens += cache_read_input_tokens`. Details (lines 3217-3223): `cached_tokens=cache_read_input_tokens`, `cache_creation_tokens=cache_creation_input_tokens`, `text_tokens=raw_input_tokens`.
- Bedrock (K-s14, lines 3017-3028): `input_tokens: Final = raw_input_tokens + cache_read_input_tokens + cache_creation_input_tokens`.
- Gemini (K-s7, lines 2684-2685 and 2832-2840): `cached_tokens = usage_metadata["cachedContentTokenCount"]`, and `prompt_tokens=usage_metadata.get("promptTokenCount", 0) + billable_tool_use_prompt_tokens + cache_creation_tokens`.
- DeepSeek (K-s1, lines 2137-2142): `prompt_cache_hit_tokens` becomes `cached_tokens`.
- The `Usage` class maps the Anthropic names (K-s1, lines 2144-2157): `_prompt_tokens_details.cached_tokens = params["cache_read_input_tokens"]` and `_prompt_tokens_details.cache_write_tokens = params["cache_creation_input_tokens"]`.
- Write-token naming (K-s1, lines 1962-1966): `cache_write_tokens` is described as "OpenAI naming (prompt_tokens_details.cache_write_tokens); this is the canonical field." and `cache_creation_tokens` as "Anthropic/Bedrock naming; kept in sync with cache_write_tokens (assigning either mirrors to the other)." Per-TTL split: `ephemeral_5m_input_tokens`, `ephemeral_30m_input_tokens`, `ephemeral_1h_input_tokens`, `ephemeral_12h_input_tokens` (K-s1, lines 1905-1909).
- Docs quote: "`prompt_tokens`: These are all prompt tokens including cache-miss and cache-hit input tokens." and "Prompt caching is silently skipped when the input is below the provider's minimum, and no error is returned."
- Source: K-s1, K-s3, K-s7, K-s14, K-s35. Basis: Filled (LiteLLM).

### Axis 4 - Tools

**K36. Tool call ids are rewritten per vendor inside the translator. Mistral ids are copied unchanged.**

- Anthropic (K-s4, lines 1824-1837): `_ANTHROPIC_TOOL_USE_ID_INVALID_CHARS: Final = re.compile(r"[^a-zA-Z0-9_-]")`, docstring `"""Anthropic requires tool_use_id to match ^[a-zA-Z0-9_-]+$."""`, invalid characters become `_`, and an empty result becomes `"tool_use_id"`.
- Bedrock (K-s4, lines 1826-1850): `_BEDROCK_TOOL_USE_ID_MAX_LEN: Final = 64`, docstring "Bedrock Converse requires toolUseId to match [a-zA-Z0-9_.:-]+ and be at most 64 chars. Ids that need rewriting get a short hash of the original appended so two ids that only differ in a replaced char or past the cut still map to distinct values."
- Gemini: minted ids, see K11.
- Mistral (K-s22, lines 801-817): each call is rebuilt as a `MistralToolCallMessage` with `id=_tool.get("id"),`, so the caller's id goes out as it came in. A search for a nine-character rule under `litellm/llms` and `litellm/litellm_core_utils` finds only an unrelated comment. Confidence: medium, this rests on a text search.
- A missing id on any response tool call becomes `f"{uuid.uuid4()}"` (K-s1, lines 1458-1461).
- Source: K-s1, K-s4, K-s22. Basis: Filled (LiteLLM).

**K37. `tool_choice` and `parallel_tool_calls` map through one small table per vendor.**

- Anthropic (K-s3, lines 1213-1247): `"auto"` to `type="auto"`, `"required"` to `type="any"`, `"none"` to `type="none"`, a named function to `type="tool"` with `name`. Parallel calls: `# Anthropic uses 'disable_parallel_tool_use' flag to determine if parallel tool use is allowed` / `# this is the inverse of the openai flag.` and `_tool_choice["disable_parallel_tool_use"] = not parallel_tool_use`.
- Gemini (K-s7, lines 1115-1128): `"none"` to `mode="NONE"`, `"required"` to `mode="ANY"`, `"auto"` to `mode="AUTO"`, a named function to `mode="ANY"` with `allowed_function_names`. `parallel_tool_calls` false with more than one tool is dropped: `if not (value is False and num_tools > 1):` / `optional_params["parallel_tool_calls"] = value` (K-s7, lines 2142-2143).
- Bedrock (K-s14, lines 1030-1037): `"none"` is refused, or dropped under `drop_params`: `f"Bedrock doesn't support tool_choice={tool_choice}. To drop it from the call, set `litellm.drop_params = True."`
- Mistral (K-s22, lines 220-231): `"required" | "any"` gives `"any"`.
- `tool_choice` is removed when the request has no tools: `if not optional_params.get("tools") and not optional_params.get("functions"):` / `optional_params.pop("tool_choice", None)` (K-s25, lines 722-723).
- Source: K-s3, K-s7, K-s14, K-s22, K-s25. Basis: Filled (LiteLLM).

**K38. Tool and output schemas are rewritten for vendors with a narrower schema dialect.**

- Gemini, quote (K-s32, lines 872-875): "Updates the input parameters, removing extraneous fields, adjusting types, unwinding $defs, and adding propertyOrdering if specified, returning the updated parameters." Steps in order (lines 888-918): `unpack_defs`, `_convert_schema_types`, `convert_anyof_null_to_nullable`, `_fix_enum_empty_strings`, `_fix_enum_types`, `process_items`, `add_object_type`, `filter_schema_fields`.
- Gemini structured output: `# remove 'strict' from json schema (not supported by Gemini)` and, for older models, `# For responseSchema, remove 'additionalProperties' (not supported)` (K-s7, lines 1609-1619).
- xAI: `strict` is filtered from every tool (K-s24, line 236).
- Anthropic: tool names that the API would reject are renamed for the request and mapped back on the reply through a per-request reverse map (K-s3, lines 3402-3415).
- Source: K-s3, K-s7, K-s24, K-s32. Basis: Filled (LiteLLM).

**K39. Images in tool results are moved or nested per vendor.**

- OpenAI-compatible targets, quote (K-s21, lines 1674-1684): "Move image, file and audio content out of role:"tool" messages into a user message inserted after the run of consecutive tool messages it belongs to. The OpenAI chat spec only allows text in tool messages, so OpenAI-compatible providers either reject or silently ignore media placed there." and "The inserted user message leads with a text part marking the media as tool output so the model does not read them with user authority."
- Gemini (K-s4, lines 1809-1819): for models without multimodal function responses the media go as sibling parts (`return [_part, *media_parts]`), else nested as `_function_response["parts"]`. The test is `return cls.is_gemini_3_or_newer(model)` (K-s7, lines 1031-1033).
- Responses bridge: a tool result's list content is converted to `input_text` and `input_image` parts inside `function_call_output` (K-s10, lines 1238-1274).
- Source: K-s4, K-s7, K-s10, K-s21. Basis: Filled (LiteLLM).

**K40. Anthropic history that holds tool calls while the request has no tools gets an invented tool.**

- Quote (K-s6, lines 9836-9848): "Prevent Anthropic from raising error when tool_use block exists but no tools are provided." with `name="dummy_tool"`, `description="This is a dummy tool call"`.
- Source: K-s6. Basis: Filled (LiteLLM).

### Axis 5 - Structured output

**K41. Anthropic: a JSON schema `response_format` uses the native output format when the model and the schema allow it, else a tool named `json_tool_call` whose arguments are turned back into message content.**

- Branch (K-s3, lines 2175-2198): the test `if self.uses_native_structured_output_for_schema(` sets `optional_params["output_format"] = _output_format`, and the `else:` branch builds the tool and, when allowed, forces it: `_tool_choice = {` / `"name": RESPONSE_FORMAT_TOOL_NAME,` / `"type": "tool",` / `}`.
- The tool name: `RESPONSE_FORMAT_TOOL_NAME = "json_tool_call"  # default tool name used when converting response format to tool call` (K-s17, line 1695).
- The native path is refused for schemas with unresolvable refs, open objects, lookaround patterns, unsupported string formats, or too many unions, optional properties, or grammar branches (K-s3, lines 1956-1968).
- The tool cannot be forced while the model thinks: `return not AnthropicModelInfo.forced_tool_use_unsupported(model) and not AnthropicModelInfo.will_think(` / `model, custom_llm_provider, request_params` / `)` (K-s16, lines 1225-1230). Then the tool is sent unforced with a description (K-s3, lines 2015-2018).
- On the reply the JSON tool call is removed from `tool_calls` and its arguments become the content (K-s3, lines 2942-2947).
- Source: K-s3, K-s16, K-s17. Basis: Filled (LiteLLM).

**K42. Gemini, Bedrock, and the Responses bridge each have their own structured output mapping.**

- Gemini (K-s7, lines 1612-1643): `json_object` gives `optional_params["response_mime_type"] = "application/json"`; a schema goes to `response_json_schema` on models that support it (`# Use responseJsonSchema (Gemini 2.0+ only, opt-in)`), else to the older `responseSchema`.
- Bedrock Converse (K-s14, lines 1197-1219): docstring "Check if the Bedrock model takes native structured outputs (outputConfig.textFormat) for this request." Otherwise the same `json_tool_call` tool is used (lines 1156-1194).
- Responses bridge (K-s10, lines 2249-2262): `response_format.json_schema` becomes a `"format"` object under `text` with `"type": "json_schema",` plus `"name"`, `"schema"`, and `"strict"` copied from the request.
- Source: K-s7, K-s10, K-s14. Basis: Filled (LiteLLM).

**K43. Refusals: Anthropic's refusal stop reason is turned into OpenAI's `refusal` message field.**

- Quote (K-s16, lines 3062-3066): `if message.get("stop_reason") != "refusal":` / `return {}` / `explanation: Final = _REFUSAL_STOP_DETAILS.validate_python(message.get("stop_details")).get("explanation")` / `return {"refusal": explanation} if isinstance(explanation, str) and explanation else {}`.
- The finish reason becomes `content_filter` (K46), and the raw value is kept: `_STOP_REASONS_KEPT_RAW: Final = frozenset({"refusal", "pause_turn"})` (K-s16, line 3036).
- Source: K-s16. Basis: Filled (LiteLLM).

### Axis 6 - Server tools and search

**K44. `web_search_options` is the neutral request field, mapped to each vendor's built-in search tool.**

- Anthropic (K-s3, lines 2026-2045): an `AnthropicWebSearchTool` with `type="web_search_20250305",` and `name="web_search",`, with `search_context_size` mapped to `max_uses` by the table `ANTHROPIC_WEB_SEARCH_TOOL_MAX_USES` with `"low": 1,` / `"medium": 5,` / `"high": 10,` (K-s17, lines 804-808).
- Gemini (K-s7, lines 1141-1147): docstring "Google doesn't support user_location or search_context_size params", returns `Tools(googleSearch={})`. Search is dropped when function tools are present, with the warning `"Vertex AI does not support mixing function declarations with "` `"search tools (googleSearch, enterpriseWebSearch, urlContext, "` `"googleSearchRetrieval) in the same request. Dropping search "` `"tools and keeping function declarations. To use search tools, "` (K-s7, lines 1202-1208).
- Responses bridge: `web_search_tool: Final[dict[str, object]] = {"type": "web_search"}` then `web_search_tool.update(web_search_options)` (K-s10, lines 2209-2211).
- xAI on chat completions: `"XAI no longer supports web search on /chat/completions (Live Search is deprecated). "` `"Dropping 'web_search_options'. Use the Responses API for XAI web search."` (K-s24, lines 270-273), and the bridge check routes xAI with `web_search_options` to Responses (K-s12, lines 199-201).
- Bedrock Nova: `return BedrockToolBlock(systemTool={"name": "nova_grounding"})` (K-s14, line 753).
- Source: K-s3, K-s7, K-s10, K-s12, K-s14, K-s17, K-s24. Basis: Filled (LiteLLM).

**K45. What comes back and what must be replayed differs per vendor. Only Gemini, Bedrock, and OpenAI citations reach the standard `annotations` field.**

- Anthropic: `tool_use` and `server_tool_use` blocks are both converted to tool calls (K-s3, lines 2973-2978), then server calls are split off by id: `client_tool_calls: Final = [tc for tc in tool_calls if tc["id"] not in server_tool_use_ids]` and `provider_specific_fields["server_tool_calls"] = server_tool_calls` (K-s3, lines 3361-3363 and 3424-3425). Results go to `provider_specific_fields["web_search_results"]` or `["tool_results"]` (lines 2981-2991). Citations go to the `"citations"` key of `provider_specific_fields`, with the cited text added as `"supported_text"` (lines 3009-3020). All of these must come back for the replay of K7.
- Gemini: grounding metadata is converted: `chat_completion_message["annotations"] = annotations` (K-s7, lines 3269-3275).
- Bedrock: `citationsContent` is kept raw and also converted to `annotations` (K-s14, lines 3420-3440).
- Responses bridge: "Annotations are already in compatible format between both APIs, so we just need to convert Pydantic models to dicts." (K-s10, lines 2270-2274).
- The standard annotation type has one kind: `type: Literal["url_citation"]` (K-s2, lines 826-831).
- Source: K-s2, K-s3, K-s7, K-s10, K-s14. Basis: Filled (LiteLLM).

### Axis 7 - Finish reasons, errors, and streaming

**K46. One table maps every vendor's stop reason to the five OpenAI values, and anything unknown becomes `stop`.**

- Quote, Anthropic rows (K-s26, lines 515-523): `"stop_sequence": "stop"`, `"end_turn": "stop"`, `"max_tokens": "length"`, `"tool_use": "tool_calls"`, `"refusal": "content_filter"`, `"compaction": "length"`, `"pause_turn": "length"`, `"model_context_window_exceeded": "length"`.
- Quote, Gemini rows (K-s26, lines 533-555): `"STOP": "stop"`, `"MAX_TOKENS": "length"`, `"CONTINUATION": "length"`, `"SAFETY": "content_filter"`, `"RECITATION": "content_filter"`, `"FINISH_REASON_UNSPECIFIED": "stop"`, `"MALFORMED_FUNCTION_CALL": "stop"`, `"OTHER": "content_filter"`, `"TOO_MANY_TOOL_CALLS": "stop"`, `"UNEXPECTED_TOOL_CALL": "stop"`, `"MISSING_THOUGHT_SIGNATURE": "stop"`.
- Other rows (K-s26, lines 524-571): Cohere `"COMPLETE"`, `"TOOL_CALL"`, `"ERROR_TOXIC"`; Bedrock `"guardrail_intervened": "content_filter"`; Zhipu `"sensitive": "content_filter"`; and `"error": "stop"` with the comment `# Generic error passthrough (OpenRouter and other OpenAI-compatible providers` / `# emit lowercase "error" when a provider fails mid-stream)`.
- Unknown values (K-s26, lines 575-580): `verbose_logger.warning("Unmapped finish_reason '%s', defaulting to 'stop'", finish_reason)` / `return "stop"`.
- Responses status (K-s10, lines 2307-2312): `"completed": "stop"`, `"incomplete": "length"`, `"failed": "stop"`, `"cancelled": "stop"`.
- xAI returns an empty finish reason with tool calls, which LiteLLM repairs: `if choice.finish_reason == "" and choice.message.tool_calls and len(choice.message.tool_calls) > 0:` / `choice.finish_reason = "tool_calls"` (K-s24, lines 288-289).
- The raw value is kept beside the mapped one, see K4 and K43.
- Source: K-s10, K-s24, K-s26. Basis: Filled (LiteLLM).

**K47. Error classes are found by matching substrings of the vendor's error text.**

- Context overflow list (K-s27, lines 106-119): `"exceed context limit"`, `"this model's maximum context length is"`, `"requested token count exceeds the model's maximum context length"`, `"string too long. expected a string with maximum length"`, `"model's maximum context limit"`, `"is longer than the model's context length"`, `"input tokens exceed the configured limit"`, ``"`inputs` tokens + `max_new_tokens` must be"``, `"exceeds the available context size",  # llama.cpp/Lemonade`, `"exceeds the maximum number of tokens allowed",  # Gemini`, `'"code": "context_length_exceeded"'`, `"prompt is too long"`. Two more patterns follow, for Cerebras (`"current length is"` with `"while limit is"`) and for `"maximum input length is"` (lines 124-132).
- Rate limit (K-s27, lines 78-91): `if re.search(r"\b429\b", error_str) and (not isinstance(status_code, int) or status_code == 429):`, then `if re.search(r"rate[\s_\-]*limit", _error_str_lower):`, then `if "service tier capacity exceeded" in _error_str_lower:` for Mistral.
- Content policy (K-s27, lines 142-150): `"content_policy_violation"`, `"responsibleaipolicyviolation"`, `"the response was filtered due to the prompt triggering azure openai's content management"`, `"your task failed as a result of our safety system"`, `"the model produced invalid content"`, `"content_filter_policy"`, `"your request was rejected as a result of our safety system"`.
- The typed exceptions raised are `ContextWindowExceededError`, `RateLimitError`, `ContentPolicyViolationError`, `BadRequestError`, and others (K-s27, for example lines 426-427 and 476).
- Source: K-s27. Basis: Filled (LiteLLM).

**K48. Streaming: an error after the first byte is raised from the stream as an exception, and usage is sent to the caller only on request.**

- A chunk with `finish_reason` `error` raises: `f"{self.custom_llm_provider} raised a streaming error - finish_reason: error, no content string given. Received Chunk={response_obj}"` (K-s28, lines 1924-1927).
- The router can fall back mid-stream. Quote, docstring (K-s28, lines 3060-3066): "Maps the raw exception to an OpenAI-compatible type, then decides whether to raise it directly (non-retriable 4xx) or wrap it in MidStreamFallbackError so the Router can trigger a fallback." and "429 (rate-limit) is explicitly exempted from the 4xx filter because it is transient and the Router should switch to another model group." The error holds `generated_content=self.response_uptil_now` and `is_pre_first_chunk=not self.sent_first_chunk` (lines 3137-3145).
- Usage: `return stream_options is not None and stream_options.get("include_usage", False) is True` (K-s28, lines 788-789). A vendor's usage-only chunk is kept for cost tracking even when the caller did not ask: "They are never returned to the caller; ``stream_options.include_usage`` only controls what the caller sees." (K-s28, lines 2428-2430).
- Source: K-s28. Basis: Filled (LiteLLM).

### Axis 8 - Unsupported parameters

**K49. LiteLLM's answer to "what happens to a parameter the vendor does not support" is: an error by default, and three opt-in switches.**

- Each provider class publishes a list through `get_supported_openai_params`. Examples: OpenAI base list (K-s25, lines 264-295), Gemini (K-s7, lines 1077-1110), xAI (K-s24, lines 153-196), Mistral (K-s22, lines 198-217), DeepSeek adds `"thinking", "reasoning_effort"` (K-s23, line 74), OpenRouter adds the same two (K-s29, lines 76-81).
- The check (K-s6, lines 5985-6013): parameters outside the list are collected, then `if litellm.drop_params is True or (drop_params is not None and drop_params is True):` they are removed, `else:` an error is raised with the text `f"{custom_llm_provider} does not support parameters: {list(unsupported_params.keys())}, for model={model}. To drop these, set `litellm.drop_params=True` or for proxy:\n\n`litellm_settings:\n drop_params: true`\n. \n If you want to use these params dynamically send allowed_openai_params={list(unsupported_params.keys())} in your request."`
- `allowed_openai_params` extends the list for one request: `supported_params.extend(allowed_openai_param_names(allowed_openai_params))` (K-s6, line 6031).
- `additional_drop_params` drops named parameters, including nested paths: `if isinstance(additional_drop_params, (list, tuple)) and k in additional_drop_params:` (K-s6, lines 4765-4766) and `# Apply nested drops from additional_drop_params` (K-s6, lines 6544-6550).
- Parameters that are not OpenAI parameters at all are passed to the vendor: `# if user passed in non-default kwargs for specific providers/models, pass them along` (K-s6, lines 6526-6533).
- A dropped reasoning parameter is logged as a warning (K-s6, lines 5996-6008).
- Source: K-s6, K-s7, K-s22, K-s23, K-s24, K-s25, K-s29. Basis: Filled (LiteLLM).

**K50. The same switch also decides value-level downgrades, where the parameter is supported but one value is not.**

- Sampling on newer Claude models (K-s16, lines 977-987): forwarded when the model supports it `or (param == "temperature" and value == 1)`, else an error `f"{model} does not support {param}={value}. {supported_hint}"` `"To drop unsupported params, set `litellm.drop_params = True`."`
- Forced tool use (K-s16, lines 1005-1023): "True when the model map flags the model with ``supports_forced_tool_use: false`` (Fable 5.1 / Mythos 5.1 400 on ``any``/``tool``) and ``drop_params`` asks for the ``auto`` downgrade; raises a clean client-side 400 for such models without ``drop_params``." The error text: `f"{model} does not support forced tool use (tool_choice='required' or a named tool). "` `"Use tool_choice='auto' and tell the model in the prompt when to call the tool, or set "` `"`litellm.drop_params = True` to downgrade to 'auto' automatically."`
- Gemini sampling (K-s7, lines 1070-1072): `f"{model} does not support {param}: the model map flags it `supports_sampling_params: false` "` `"because Google ignores sampling params on this model. "`
- Gemini named tool that cannot be forced (K-s7, lines 1129-1134): dropped under `drop_params`, else an error whose message begins "VertexAI can't force tool_choice '{name}', since none of that namespace's members can be sent to Gemini as a function."
- Source: K-s7, K-s16. Basis: Filled (LiteLLM).

### Axis 9 - LiteLLM's OpenRouter provider module

Everything in this group that says what OpenRouter does is `OR-inferred`: it is inferred from the special cases LiteLLM's client code has for OpenRouter, at the commit above. It was not checked against OpenRouter's documentation here. What LiteLLM itself does is `Filled (LiteLLM)`.

**K51. Reasoning: LiteLLM's OpenRouter module copies `delta.reasoning` to `reasoning_content`, always accepts `reasoning_effort` and `thinking`, rewrites effort `max` to `xhigh`, and leaves `reasoning_details` nested under `provider_specific_fields`.**

- Quote (K-s29, lines 322-323): `for choice in chunk["choices"]:` / `choice["delta"]["reasoning_content"] = choice["delta"].get("reasoning")`.
- Quote (K-s29, lines 77-81): `"""` / `Include reasoning parameters regardless of model-map capabilities.` / `"""` then `return list(dict.fromkeys([*supported_params, "reasoning_effort", "thinking"]))`.
- Quote (K-s29, lines 90-92): `# OpenRouter expects "xhigh" instead of "max" for reasoning_effort.` / `if non_default_params.get("reasoning_effort") == "max":` / `non_default_params = {**non_default_params, "reasoning_effort": "xhigh"}`.
- `reasoning_details`: a search of all of `litellm/` finds the string once, in a list of keys to redact from logs (`litellm/litellm_core_utils/redact_messages.py` line 339, K-s31). In a non-streaming reply any unknown message key is put under `provider_specific_fields` (`for f in choice["message"].keys() - _MESSAGE_FIELDS:` / `provider_specific_fields[f] = choice["message"][f]`, K-s31 lines 906-907), and the OpenAI-shaped request builder sends message keys as given (K24). So a caller going through LiteLLM to OpenRouter gets `reasoning_details` back nested one level down, and LiteLLM does not lift it back onto the message for the next request.
- Inferred about OpenRouter: it streams reasoning text in `delta.reasoning`, and its `reasoning_effort` accepts `xhigh` and rejects `max`. Confidence: medium, from a client code comment.
- Source: K-s29, K-s31. Basis: OR-inferred (from LiteLLM's OpenRouter client code).

**K52. Cache control: LiteLLM keeps `cache_control` for OpenRouter models whose name contains one of five strings, moves a message-level marker onto the last content block, and removes the marker for all other models.**

- Quote (K-s29, lines 60-67): `class CacheControlSupportedModels(str, Enum):` / `"""Models that support cache_control in content blocks."""` with `CLAUDE = "claude"`, `GEMINI = "gemini"`, `MINIMAX = "minimax"`, `GLM = "glm"`, `ZAI = "z-ai"`.
- Quote, docstring (K-s29, lines 131-136): "Move cache_control from message level to content blocks. OpenRouter requires cache_control to be inside content blocks, not at message level. To avoid exceeding Anthropic's limit of 4 cache breakpoints, cache_control is only added to the LAST content block in each message."
- Inferred about OpenRouter: it reads `cache_control` on content blocks for at least those model families, and a message-level marker is ignored there. Confidence: medium.
- Source: K-s29. Basis: OR-inferred (from LiteLLM's OpenRouter client code).

**K53. Usage and cost: LiteLLM always asks OpenRouter for usage accounting and takes the cost and the serving provider from the body.**

- Quote (K-s29, lines 193-196): `# ALWAYS add usage parameter to get cost data from OpenRouter` / `# This ensures cost tracking works for all OpenRouter models` / `if "usage" not in response:` / `response["usage"] = {"include": True}`.
- Quote (K-s29, lines 237-249): `# OpenRouter returns cost information in the usage object when usage.include=true`, `response_provider: Final = response_json.get("provider")`, `response_cost: Final = _JSON_OBJECT.validate_python(response_json["usage"]).get("cost")`. The provider is stored as `"upstream_provider"` (line 245), and from stream chunks too (lines 336-338).
- Quote (K-s28, lines 2644-2646): `# stream_chunk_builder.  Some providers (e.g. OpenRouter)` / `# send usage in a chunk after finish_reason, which arrives` / `# after _hidden_params["usage"] was initially set.`
- The reply's `model` is taken as the served model for OpenRouter (`_served_openrouter_model`, K-s28 lines 1118-1124).
- Inferred about OpenRouter: `usage: {"include": true}` in the request makes the reply's `usage` hold `cost`; the body has a top-level `provider` string; a usage chunk follows the finish chunk in a stream. Confidence: medium.
- Source: K-s28, K-s29. Basis: OR-inferred (from LiteLLM's OpenRouter client code).

**K54. Errors: LiteLLM treats three OpenRouter shapes as errors even when the HTTP status is 200.**

- Non-streaming (K-s29, lines 37-57): a choice with `choice.get("finish_reason") == "error" or choice.get("error")` raises `OpenRouterException` with `status_code=code if isinstance(code, int) and 400 <= code < 600 else 502` and the default text `"OpenRouter returned finish_reason 'error': the provider failed during generation"`.
- Streaming (K-s29, lines 304-319): a chunk with a top-level `"error"` is read for `error_chunk["message"]`, `error_chunk.get("metadata", {})`, `error_chunk.get("user_id", "")`, `error_chunk["code"]`, and raised with the message format `"Message: {}, Metadata: {}, User ID: {}"`.
- A plain JSON body with an `error` key (K-s31, line 822): `### CHECK IF ERROR IN RESPONSE ### - openrouter returns these in the dictionary`.
- Status mapping (K-s27, lines 2392-2453): 400 and 422 to `BadRequestError`, 401 to `AuthenticationError`, 404 to `NotFoundError`, 408 and 504 to `Timeout`, 429 to `RateLimitError`, 503 to `ServiceUnavailableError`.
- Inferred about OpenRouter: a failed generation can arrive in a 200 body as `finish_reason: "error"` with `choices[].error.code` and `.message`, and stream errors arrive as a chunk with `error.message`, `error.code`, `error.metadata`. This agrees with LC11 in the shared brief. Confidence: medium.
- Source: K-s27, K-s29, K-s31. Basis: OR-inferred (from LiteLLM's OpenRouter client code).

**K55. Pass-through fields, headers, model list, and the Responses route.**

- Quote (K-s29, lines 96-107): `# OpenRouter-only parameters` then `transforms`, `models`, `route` are popped and put in `extra_body`, with `mapped_openai_params["extra_body"] = extra_body  # openai client supports `extra_body` param`.
- Headers (K-s33, lines 3832-3846): `"HTTP-Referer": openrouter_site_url,` and `"X-Title": openrouter_app_name,`, with defaults `"https://litellm.ai"` and `"liteLLM"`.
- Model list (K-s30, lines 14-17 and 88-91): `_DEFAULT_API_BASE: Final = "https://openrouter.ai/api/v1"`, `_CHAT_CATALOG_ROUTE: Final = "models/user"`, and models whose id ends in `_BATCH_ONLY_SUFFIX: Final = ":batch"` are left out of the chat list.
- Responses route (K-s30, `responses/transformation.py` lines 4-6): "OpenRouter supports the Responses API at https://openrouter.ai/api/v1/responses with OpenAI-compatible request/response format, including reasoning with encrypted_content for multi-turn stateless workflows." And line 145: `"""OpenRouter does not support native WebSocket for Responses API"""`.
- The module names three OpenRouter-only fields (`transforms`, `models`, `route`). The `provider` routing block reaches OpenRouter by the general rule that passes non-OpenAI parameters along (K49). Confidence: medium, from reading the module in full.
- Inferred about OpenRouter: `transforms`, `models`, and `route` are top-level body fields; a `:batch` model suffix exists; the Responses route returns `encrypted_content`.
- Source: K-s29, K-s30, K-s33. Basis: OR-inferred (from LiteLLM's OpenRouter client code).

**K56. LiteLLM's cache hook treats OpenRouter as a gateway that may forward to Anthropic, so it keeps the Anthropic-style mark there.**

- Quote (K-s19, lines 515-516): "A gateway or self-hosted server (OpenRouter, a LiteLLM proxy, vLLM) may pass the request on to Anthropic, so there the flag only picks OpenAI's breakpoint for a model the cost map, or the deployment's own ``litellm_provider``, lists under that provider."
- Source: K-s19. Basis: OR-inferred (from LiteLLM's cache hook code).

### Axis 10 - Failures in the field

Every item here is a public issue or pull request page on github.com, a third-party account. The pages were read on 2026-11-22. Where a page was read only through the search tool's page extract, the item says so.

**K57. Issue 23047 (closed 2026-03-10): a reply rebuilt in the wrong block order broke Anthropic's thinking check.**

- Quote, error: "messages.N.content.1: `thinking` or `redacted_thinking` blocks in the latest assistant message cannot be modified. These blocks must remain as they were in the original response."
- Quote, cause: "This happens even when passing `r.choices[0].message` back unmodified" and "the original Anthropic response interleaves thinking blocks *between* tool use/result blocks".
- Quote, the reporter's objection to a proposed fix that stored and replayed the reply's content whole: "that appears to be bypassing stuff like cache points that litellm adds. You're correct that you need to fix the ordering, but I think you should *only* fix that".
- The fix that landed is the pattern rebuild of K7, whose comment cites this issue.
- Source: K-s36. Basis: Filled (LiteLLM), issue report (third-party). Also in workspace: the 2026-11-17 Anthropic Messages report, re-fetched live.

**K58. Issue 29428 (closed 2026-06-24): streamed `thinking_blocks` could not be replayed.**

- Quote: "LiteLLM emits one `ChatCompletionThinkingBlock` per upstream SSE delta" and the three example blocks with `"signature": ""`, `"signature": ""`, `"signature": "sig-final"`.
- Quote, error: "messages.X.content.Y: Invalid 'signature' in 'thinking' block".
- Quote: "the contract differs between `stream=True` and `stream=False` for the same `completion()` call."
- Source: K-s37. Basis: Filled (LiteLLM), issue report (third-party).

**K59. Issue 18926 (closed 2026-01-12): the rule that drops `thinking` when blocks are missing misfired.**

- Quote, error: "messages.1.content.0: When thinking is disabled, an `assistant` message in the final position cannot contain `thinking`. To use thinking blocks, enable `thinking` in your request."
- Quote: "This is, of course, based on the assumption that Opus will return a thinking block associated with a tool calling message if thinking is turned on. Unfortunately, in my testing, this doesn't seem to be *always* the case."
- Source: K-s38. Basis: Filled (LiteLLM), issue report (third-party).

**K60. Issue 32188 (open, last comment 2026-09-24): LiteLLM's own field reached a strict upstream.**

- Quote, error: "litellm.BadRequestError: OpenAIException - Extra inputs are not permitted, field: 'messages[2].thinking_blocks'".
- Quote, comment of 2026-09-24 on lenient servers: "The chat template reads a replayed thinking block as an unfinished assistant turn and keeps reasoning instead of answering: HTTP 200, a complete-looking response containing no text, the whole output budget consumed and billed."
- Quote, same comment: "The root problem is that reasoning replay is provider-specific state being forwarded as if it were generic message history." and "the message-key stripping itself is asymmetric".
- This matches K24 at the commit read today.
- Source: K-s39. Basis: Filled (LiteLLM), issue report (third-party). Also in workspace: the 2026-11-17 Anthropic Messages report, re-fetched live.

**K61. Issue 37849 (open, marked stale 2026-11-20) and issue 52001 (closed 2026-11-17): the Gemini signature inside the tool call id breaks clients that normalize ids.**

- Quote (37849): "Measured against Vertex Gemini 3, the returned tool call id ranges from **843 to 4423 characters** depending on the call."
- Quote (37849), error: "Invalid value at 'contents[1].parts[0].thought_signature' (TYPE_BYTES), Base64 decoding failed for "AY89a1/_57b05e78dc"" and the second form `"message": "Invalid thought signature.", "status": "INVALID_ARGUMENT"`.
- Quote (37849): "Neither is a key a generic OpenAI-compatible client knows about, and the second one is actively harmful: `id` is by convention a short identifier, so clients normalize it." and "once a corrupted signature enters the history every later turn fails."
- State at the commit read today: an id-embedded value that is not strict base64 is now discarded (K12), and a rejected signature triggers a resend with the placeholder (K13). A mangled value that is still valid base64 is sent as is first.
- Issue 52001 was read only as a search extract. Its title names "persistent 400 INVALID_ARGUMENT on Vertex AI". The reporter's own earlier suggestion in 37849, a top-level `tool_calls[].thought_signature` key, cites another gateway: "support that was added for Venice, which returns the signature at exactly that key".
- Source: K-s40, K-s49. Basis: Filled (LiteLLM), issue report (third-party).

**K62. Issue 46369 (open, 2026-10-14): on the Anthropic-shaped route the Gemini signature sits in a field Anthropic clients drop, so every turn uses the placeholder.**

- Quote: "`provider_specific_fields` is not part of the Anthropic Messages schema, so spec-compliant clients (Claude Code, Anthropic SDK users who rebuild history from typed objects) drop it. The real signature therefore never makes it back to Gemini".
- Quote: "For models recognized by `_is_gemini_3_or_newer`, LiteLLM silently injects `skip_thought_signature_validator`. Requests succeed, but every turn runs without the model's own reasoning state, which Google documents as degrading quality."
- Quote, error for models the name test misses: "400 Function call is missing a thought_signature".
- Source: K-s41. Basis: Filled (LiteLLM), issue report (third-party).

**K63. Issue 25322 (closed, opened 2026-04-08): a signature on a separate thought part was never captured.**

- Quote: "But Gemini often places `thoughtSignature` on a **separate** thought part (`thought: true`), not on the `functionCall` part. When the signature lives on the thought part, it's never captured, never embedded in the `tool_call_id`, and never sent back on the following turn."
- Quote, error from a comment: "Function call is missing a thought_signature in functionCall parts. This is required for tools to work correctly, and missing thought_signature may lead to degraded model performance."
- Source: K-s42. Basis: Filled (LiteLLM), issue report (third-party).

**K64. Pull request 49697 (merged 2026-11-03): a router fallback between Google's two backends made every later turn fail.**

- Quote: "A Gemini thought signature validates only on the backend that minted it, so a router fallback between `vertex_ai/` and `gemini/` turns a retryable error into a 400 on every turn after the first tool call".
- Quote: "Google's 400 names no part and reads the same for foreign, truncated or corrupted signatures".
- This is the code of K13.
- Source: K-s43. Basis: Filled (LiteLLM), pull request (third-party).

**K65. Issue 51930 (open, 2026-11-15): after stripping reasoning for a move between Azure regions, the next turn fails because item ids stayed.**

- Quote, error: "Item 'msg_0b354c14…' of type 'message' was provided without its required 'reasoning' item: 'rs_0b354c14…'."
- Quote: "The strip keeps the ids of the items that went with the reasoning."
- Quote, correction by the reporter: "With `store: false` plus `include: ["reasoning.encrypted_content"]` the same sequence returns 200 even without a fix, because Azure has no stored copy to check the replayed ids against."
- Source: K-s44. Basis: Filled (LiteLLM), issue report (third-party).

**K66. Issues 36559 (closed 2026-08-20) and 42104 (open, updated 2026-11-12): moving a mid-conversation system message to the front changed the cached prefix on every turn.**

- Quote (36559): "the hoist itself invalidates the entire cached prompt prefix on every turn a reminder appears". Measured: reminder-turn `cache_creation_input_tokens` 1746 with the hoist against 60 with the message converted in place.
- Quote (42104): "prompt caching for `vertex_ai` Gemini models stops working entirely" with five turns of `cache_read_input_tokens=0`, and "Gemini via `vertex_ai`: ~0% cache hit rate in agent-style multi-turn traffic". The same report measured 77 percent cache reads on turn 2 without the mid-conversation system message.
- Source: K-s45, K-s46. Basis: Filled (LiteLLM), issue report (third-party). Issue 36559 is also in workspace: the 2026-11-17 Anthropic Messages report, re-fetched live.

**K67. Issue 32086 (closed as stale 2026-10-09): a mid-stream upstream failure on a bridged route reached the client as an empty success.**

- Quote: "the failure is converted into a **well-formed, zero-content SSE stream**: `message_start` → `message_delta (stop_reason: end_turn)` → `message_stop`, sub-second, with **no `error` event**."
- Quote: "The client cannot distinguish this from a model that legitimately produced no output".
- Source: K-s47. Basis: Filled (LiteLLM), issue report (third-party).

**K68. Issue 26625 (read only as a search extract): on one route a model-name test dropped `cache_control`, so Bedrock never cached.**

- Quote from the extract: "`cache_control` directives are silently dropped, causing prompt caching to never activate." and the table row `/v1/messages` (Anthropic) with Bedrock `cacheWriteInputTokens` 0 and `cacheReadInputTokens` 0 against 34,004 written on `/v1/chat/completions`.
- Confidence: low until the page is fetched directly.
- Source: K-s48. Basis: Filled (LiteLLM), issue report (third-party).

### Re-checks of workspace facts at today's commit

**K69. The 2026-11-17 Anthropic Messages report read LiteLLM at `bcb1f486`. Four of its LiteLLM facts were re-read at `90dce4b9` and still hold.**

- `parallel_tool_calls` is inverted inside `tool_choice` (K37).
- LiteLLM asks for summarized thinking on adaptive models (K26).
- The default `max_tokens` falls back to a model table and then to 128000 (K25, K-s17 line 661; K-s3 line 809).
- LiteLLM turns thinking off when the tool-loop message lacks thinking blocks, under `modify_params` (K9).
- The report's statement that the typed list forms keep several thinking blocks while "the translator still has to guess where they sat among the text and tool blocks" is confirmed in code by K6 and K7.
- Source: K-s51, K-s3, K-s17. Basis: Filled (LiteLLM). Also in workspace: the 2026-11-17 report.

**K70. The 2026-09-30 exhaustion-signals research read LiteLLM's overflow substring list at commit `264b09ac`. The list at today's commit is longer.**

- Today's list (K47) includes `'"code": "context_length_exceeded"'` and `"prompt is too long"`. The workspace document's finding that "every text matcher breaks" when wording drifts still describes the design.
- Source: K-s52, K-s27. Basis: Filled (LiteLLM). Also in workspace: `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md`.

**K71. The 2026-09-20 model and tool differences research records that Mistral rejects tool call ids that are not exactly nine alphanumerics. LiteLLM's Mistral transform does not rewrite ids to that form.**

- Workspace quote: "Mistral rejects tool-call ids that are not exactly nine alphanumerics".
- LiteLLM copies the caller's id (K36).
- Not re-checked today against Mistral's documentation, so the native rule keeps the `Workspace` label.
- Source: K-s53, K-s22. Basis: Workspace (rule), Filled (LiteLLM) (handling).

**K72. In a stream, Anthropic thinking still arrives as one block fragment per delta, now tagged with an `index`, so a streaming caller has to merge fragments into whole signed blocks before it can send them back.**

- Text fragment (K-s56, lines 883-889): `thinking_blocks = [` / `ChatCompletionThinkingBlock(` / `type="thinking",` / `thinking=thinking_content,` / `index=self.thinking_block_index,`.
- Signature fragment (K-s56, lines 894-898): `ChatCompletionThinkingBlock(` / `type="thinking", thinking="", signature=signature, index=self.thinking_block_index`.
- The fragments are also copied to `provider_specific_fields["thinking_blocks"]` (K-s56, lines 890 and 899), and a streamed compaction arrives as `provider_specific_fields["compaction_delta"]` (lines 902-905).
- On replay the `index` key is removed (K8).
- This is the state after the fix for issue 29428 (K58): the shape of each fragment is the same as the issue showed, with `index` added. Confidence: medium that `index` is what that fix added, since the pull request itself was not read.
- Source: K-s56. Basis: Filled (LiteLLM).

## Conflicts and disagreements

1. **LiteLLM's documentation against its own code, on who returns `thinking_blocks`.** The docs say `thinking_blocks` is "Only returned for Anthropic models" (K5). The code also fills it for Gemini thought parts (K10), for Bedrock Converse (K21), and for Responses reasoning items in a stream (`delta=Delta(thinking_blocks=list(signed_blocks)),` at K-s10 line 737). The code is right for the commit read today. The docs lag.
2. **Issue 37849 against today's code, on validation of the id-embedded Gemini signature.** The issue (still open) says the proxy forwards the value after `__thought__` "without validating it". Today's code returns the value only when it is strict base64 (K12) and resends with a placeholder after a rejection (K13). The code is right for today. The issue describes versions up to 1.96.0 and the main branch of 2026-08-21. A mangled value that is still valid base64 is still sent first.
3. **Closed pull request 22448 against today's code, on `include: ["reasoning.encrypted_content"]`.** The pull request text says the chat bridge adds the include automatically. At today's commit that string appears in three files, all outside the bridge folder (K19). The pull request page shows "State: closed", not merged, in the search extract. Today's code is the better witness, at medium confidence.
4. **Issue 29428 against today's code, on streamed thinking blocks.** The issue was closed by a fix titled "emit replayable streaming thinking blocks". Today's stream handler still emits one fragment per delta, with an `index` (K72). So "replayable" means "mergeable by index", and a streaming caller still has work to do. Whether LiteLLM's own stream rebuilder merges them was not read.
5. **The 2026-11-17 Anthropic Messages report, on how many conventions LiteLLM has.** The report lists three conventions across projects for returning thinking through chat completions and names LiteLLM's as `thinking_blocks`. That is right as far as it goes. Today's code shows LiteLLM alone has three more places for opaque state: `reasoning_items` for OpenAI (K17), the tool call id suffix for Gemini (K10), and the `provider_specific_fields` keys (K3). This adds to the report and does not contradict it.
6. **The shared brief's LC17, on who owns the name `cache_write_tokens`.** LC17 calls `prompt_tokens_details.cache_write_tokens` the extension name OpenRouter uses. LiteLLM's code comment calls the same name "OpenAI naming (prompt_tokens_details.cache_write_tokens); this is the canonical field." (K35), and LiteLLM's docs say OpenAI reports a write split under `usage.input_tokens_details.cache_write_tokens_details` (K-s35). I did not read OpenAI's documentation, so which is right is open. If OpenAI now reports cache writes itself, the name is a standard field and not an OpenRouter extension.
7. **The shared brief's LC3, as a contrast in policy.** LC3 says OpenRouter silently ignores unsupported parameters. LiteLLM's default is the opposite: an error, with silent dropping only under `drop_params` (K49). LiteLLM's OpenRouter module does put `reasoning_effort` and `thinking` in the supported list for every OpenRouter model (K51), which fits a server that ignores what a model cannot use. That is an inference and says nothing certain about OpenRouter.
8. **The 2026-09-20 workspace research against LiteLLM, on Mistral tool call ids.** The workspace says Mistral requires exactly nine alphanumeric characters. LiteLLM sends the caller's id unchanged (K36, K71). Either LiteLLM users with foreign ids hit that error, or Mistral's API accepts other ids today. Not resolved here.
9. **LiteLLM's docs table against its own chat bridge, on OpenAI state.** The docs describe OpenAI reasoning as server-side with `previous_response_id` (K-s34). The chat bridge refuses `previous_response_id` (K16) and uses client-held `reasoning_items` (K17). Both are true: the first describes the native API, the second the stateless translation.

## Open questions blocked by closed source

Each question is about OpenRouter's closed translator. "Tried" is what this topic could try, since OpenRouter's own documents belong to other subagents.

| Question | What I tried | Fill found |
|---|---|---|
| How does a chat-shaped translator keep Anthropic signatures and block order? | Read LiteLLM's reply and request code | Yes, LiteLLM: separate `thinking_blocks` list and a pattern rebuild that guesses order (K6, K7, K57). Shows a typed list loses order |
| How are Gemini thought signatures kept for a client that only echoes standard fields? | Read LiteLLM's Gemini code and four issues | Yes, LiteLLM: signature inside the tool call id, placeholder when missing, resend on rejection (K10, K12, K13, K61) |
| Does a translator create a Gemini `cachedContents` object from `cache_control` marks, and how is it keyed? | Read LiteLLM's context caching code and docs | Yes, LiteLLM: it does, keyed by a hash in `displayName`, found by listing (K32) |
| How is a chat request bridged to a Responses-only OpenAI model, and where does `encrypted_content` go? | Read LiteLLM's bridge | Yes, LiteLLM: `reasoning_items` plus an order map with a checksum (K15-K19) |
| What happens to a parameter the upstream does not support? | Read LiteLLM's parameter check | Yes, LiteLLM: error by default, three switches (K49, K50). LC3 gives OpenRouter's documented answer |
| Does `prompt_tokens` include cached tokens after normalization? | Read LiteLLM's usage code | Yes, LiteLLM: it adds cache reads and writes for Anthropic and Bedrock (K35) |
| How are tool call ids rewritten between vendors? | Read LiteLLM's id code | Yes, LiteLLM: per-vendor character and length rules (K36). No fill for Mistral's rule |
| How are vendor stop reasons mapped, and is the native value kept? | Read LiteLLM's table | Yes, LiteLLM: one table, native value in an extension field (K4, K46) |
| Are extension fields stripped before a strict upstream? | Read LiteLLM's strip helpers and one open issue | Yes, LiteLLM: per provider, with gaps (K24, K60) |
| What happens to encrypted reasoning when the next request goes to another account or region? | Read LiteLLM's router check and one open issue | Yes, LiteLLM: pin to the origin, else strip (K20, K65) |
| How is structured output delivered where the model lacks native support? | Read LiteLLM's Anthropic and Bedrock code | Yes, LiteLLM: a tool named `json_tool_call`, unforced when the model thinks (K41, K42) |
| Does OpenRouter lift `reasoning_details` for clients that do not know it? | Read LiteLLM's OpenRouter module | No fill found. LiteLLM itself leaves `reasoning_details` nested in `provider_specific_fields` (K51) |

## Unverified

- **Vendor rules stated only in LiteLLM's code comments.** These were not re-read in vendor documentation today, so they are `Filled (LiteLLM)` claims about vendors, not `Vendor-documented` facts: Anthropic checks thinking signatures by position (K7); Gemini bills each replayed signature copy (K12); the Gemini placeholder "will negatively impact model performance" (K12); OpenAI chat completions rejects function tools while reasoning on GPT-5.4 and newer (K15); only Claude accepts `redactedContent` back on Bedrock (K21); Gemini 3 and newer accept a function call `id` and older models answer 400 (K11); OpenAI's `prompt_cache_breakpoint` exists from GPT-5.6 (K33); some Claude models reject forced tool use (K50).
- **Text-search results.** Six providers calling the strip helpers (K24), the Gemini request code building thought parts from `reasoning_content` only (K12), Mistral ids going out unchanged (K36), and the caller being the one who adds `include` on the chat bridge (K19) rest on searches of the source tree, not on reading every call path.
- **Pages read only as search extracts.** Issue 26625 (K68), issue 52001 (K61), and pull request 22448 (K19) were read through the search tool's page extract, not fetched directly.
- **Everything labeled `OR-inferred`** (K51-K56) was read from LiteLLM's client code and not checked here against OpenRouter's documentation or spec.
- **The Rust path.** Only Python code under `litellm/` was read. The 2026-11-16 workspace report says chat completions translation is 5 percent migrated to Rust, so the Python code is still the path that serves chat completions, but that share was not re-checked today.
- **LiteLLM's own stream rebuilder.** Whether `stream_chunk_builder` merges the thinking fragments of K72 into whole blocks was not read.
- **Trimming order of extra cache marks** (K30) is taken from helper names.
- Every claim in this file comes from a source read today, and none from memory. Model names in quotes (for example "Fable 5.1", "gpt-5.6", "gemini-3.8-flash") are copied from the sources as written.

## Workspace documents used

- **2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build"** (cabinet output, the Anthropic Messages report; K-s51). Took: its reading of LiteLLM's Anthropic transformation at `bcb1f486`, and issues 23047, 36559, 32188. Re-checked live: yes, the three issues were fetched again today and four code facts were re-read at `90dce4b9` (K57, K60, K66, K69). The report stands as written, with one addition (conflict 5).
- **2026-11-16, "Could LiteLLM substitute for Runvil Gateway?"** (cabinet output, the report on LiteLLM versus the Runvil Gateway; K-s54). Took: that chat completions translation is still Python, and that LiteLLM's stable release then was v1.109.1. Re-checked live: partly. The code read today is Python, and issue pages today name builds `1.111.0.dev4` and `1.111.0-dev.4`. The Rust percentages were not re-checked.
- **`runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md`, dated 2026-09-30** (K-s52). Took: that LiteLLM classifies overflow by substring and that the list misses reworded errors. Re-checked live: yes, the list was re-read at today's commit and is longer (K47, K70).
- **`runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md`, dated 2026-09-20** (K-s53). Took: the Mistral nine-character id rule and the DeepSeek `reasoning_content` rule. Re-checked live: the DeepSeek rule matches LiteLLM's code comment and error string (K22); the Mistral rule was not re-checked at Mistral (K71).

## Sources

All LiteLLM source entries are `BerriAI/litellm` at commit `90dce4b93bd2b090621a6b9083311294d3358bc6` (commit date 2026-11-22T02:05:03-08:00).

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| K-s1 | `BerriAI/litellm` at `90dce4b9`, `litellm/types/utils.py` | 2026-11-22 | gateway source |
| K-s2 | `BerriAI/litellm` at `90dce4b9`, `litellm/types/llms/openai.py` | 2026-11-22 | gateway source |
| K-s3 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/anthropic/chat/transformation.py` | 2026-11-22 | gateway source |
| K-s4 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/prompt_templates/factory.py` | 2026-11-22 | gateway source |
| K-s5 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/prompt_templates/reasoning_utils.py` | 2026-11-22 | gateway source |
| K-s6 | `BerriAI/litellm` at `90dce4b9`, `litellm/utils.py` | 2026-11-22 | gateway source |
| K-s7 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/vertex_ai/gemini/vertex_and_google_ai_studio_gemini.py` | 2026-11-22 | gateway source |
| K-s8 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/vertex_ai/gemini/transformation.py` | 2026-11-22 | gateway source |
| K-s9 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/vertex_ai/gemini/rejected_thought_signature_retry.py` | 2026-11-22 | gateway source |
| K-s10 | `BerriAI/litellm` at `90dce4b9`, `litellm/completion_extras/litellm_responses_transformation/transformation.py` | 2026-11-22 | gateway source |
| K-s11 | `BerriAI/litellm` at `90dce4b9`, `litellm/completion_extras/litellm_responses_transformation/history.py` | 2026-11-22 | gateway source |
| K-s12 | `BerriAI/litellm` at `90dce4b9`, `litellm/responses/bridge_check.py` | 2026-11-22 | gateway source |
| K-s13 | `BerriAI/litellm` at `90dce4b9`, `litellm/router_utils/pre_call_checks/encrypted_content_affinity_check.py` | 2026-11-22 | gateway source |
| K-s14 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/bedrock/chat/converse_transformation.py` | 2026-11-22 | gateway source |
| K-s15 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/bedrock/common_utils.py` | 2026-11-22 | gateway source |
| K-s16 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/anthropic/common_utils.py` | 2026-11-22 | gateway source |
| K-s17 | `BerriAI/litellm` at `90dce4b9`, `litellm/constants.py` | 2026-11-22 | gateway source |
| K-s18 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/vertex_ai/context_caching/transformation.py` and `litellm/llms/vertex_ai/context_caching/vertex_ai_context_caching.py` | 2026-11-22 | gateway source |
| K-s19 | `BerriAI/litellm` at `90dce4b9`, `litellm/integrations/anthropic_cache_control_hook.py` and `litellm/types/integrations/anthropic_cache_control_hook.py` | 2026-11-22 | gateway source |
| K-s20 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/prompt_templates/prompt_cache_breakpoint_utils.py` | 2026-11-22 | gateway source |
| K-s21 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/prompt_templates/common_utils.py` | 2026-11-22 | gateway source |
| K-s22 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/mistral/chat/transformation.py` | 2026-11-22 | gateway source |
| K-s23 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/deepseek/chat/transformation.py` | 2026-11-22 | gateway source |
| K-s24 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/xai/chat/transformation.py` | 2026-11-22 | gateway source |
| K-s25 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/openai/chat/gpt_transformation.py` | 2026-11-22 | gateway source |
| K-s26 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/core_helpers.py` | 2026-11-22 | gateway source |
| K-s27 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/exception_mapping_utils.py` | 2026-11-22 | gateway source |
| K-s28 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/streaming_handler.py` | 2026-11-22 | gateway source |
| K-s29 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/openrouter/chat/transformation.py` | 2026-11-22 | gateway source |
| K-s30 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/openrouter/common_utils.py` and `litellm/llms/openrouter/responses/transformation.py` | 2026-11-22 | gateway source |
| K-s31 | `BerriAI/litellm` at `90dce4b9`, `litellm/litellm_core_utils/llm_response_utils/convert_dict_to_response.py` and `litellm/litellm_core_utils/redact_messages.py` | 2026-11-22 | gateway source |
| K-s32 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/vertex_ai/common_utils.py` | 2026-11-22 | gateway source |
| K-s33 | `BerriAI/litellm` at `90dce4b9`, `litellm/main.py` | 2026-11-22 | gateway source |
| K-s34 | https://docs.litellm.ai/docs/reasoning_content | 2026-11-22 | gateway source (project documentation page) |
| K-s35 | https://docs.litellm.ai/docs/completion/prompt_caching | 2026-11-22 | gateway source (project documentation page) |
| K-s36 | https://github.com/BerriAI/litellm/issues/23047 | 2026-11-22 | issue report (third-party) |
| K-s37 | https://github.com/BerriAI/litellm/issues/29428 | 2026-11-22 | issue report (third-party) |
| K-s38 | https://github.com/BerriAI/litellm/issues/18926 | 2026-11-22 | issue report (third-party) |
| K-s39 | https://github.com/BerriAI/litellm/issues/32188 | 2026-11-22 | issue report (third-party) |
| K-s40 | https://github.com/BerriAI/litellm/issues/37849 | 2026-11-22 | issue report (third-party) |
| K-s41 | https://github.com/BerriAI/litellm/issues/46369 | 2026-11-22 | issue report (third-party) |
| K-s42 | https://github.com/BerriAI/litellm/issues/25322 (full page text as returned by the search tool) | 2026-11-22 | issue report (third-party) |
| K-s43 | https://github.com/BerriAI/litellm/pull/49697 (full page text as returned by the search tool) | 2026-11-22 | issue report (third-party) |
| K-s44 | https://github.com/BerriAI/litellm/issues/51930 | 2026-11-22 | issue report (third-party) |
| K-s45 | https://github.com/BerriAI/litellm/issues/36559 | 2026-11-22 | issue report (third-party) |
| K-s46 | https://github.com/BerriAI/litellm/issues/42104 | 2026-11-22 | issue report (third-party) |
| K-s47 | https://github.com/BerriAI/litellm/issues/32086 | 2026-11-22 | issue report (third-party) |
| K-s48 | https://github.com/BerriAI/litellm/issues/26625 (search extract only) | 2026-11-22 | issue report (third-party) |
| K-s49 | https://github.com/BerriAI/litellm/issues/52001 (search extract only) | 2026-11-22 | issue report (third-party) |
| K-s50 | https://github.com/BerriAI/litellm/pull/22448 (search extract only) | 2026-11-22 | issue report (third-party) |
| K-s51 | Cabinet output dated 2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build" | 2026-11-22 | workspace document |
| K-s52 | `runvil-design`, `research/2026-09-30-compaction-exhaustion-signals.md` (dated 2026-09-30) | 2026-11-22 | workspace document |
| K-s53 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (dated 2026-09-20) | 2026-11-22 | workspace document |
| K-s54 | Cabinet output dated 2026-11-16, "Could LiteLLM substitute for Runvil Gateway?" | 2026-11-22 | workspace document |
| K-s55 | `BerriAI/litellm` at `90dce4b9`, `litellm/types/router.py` and `litellm/litellm_core_utils/model_param_helper.py` | 2026-11-22 | gateway source |
| K-s56 | `BerriAI/litellm` at `90dce4b9`, `litellm/llms/anthropic/chat/handler.py` | 2026-11-22 | gateway source |
