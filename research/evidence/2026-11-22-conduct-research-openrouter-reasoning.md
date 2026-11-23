---
produced: 2026-11-22
title: "OpenRouter reasoning deep dive - reasoning request object (effort, max_tokens, enabled, exclude, summary, mode, context), reasoning_details array and format enum, replay rules, Anthropic signatures and prefix-locked thinking, OpenAI encrypted reasoning, Gemini thought signatures, DeepSeek and Kimi reasoning_content, streaming accumulation, Responses and Messages routes, usage"
---

# OpenRouter reasoning deep dive - request controls, `reasoning_details`, and replay rules

- What was read: OpenRouter's reasoning guide in full, ten model migration guides, the parameters page, the Responses reasoning page, the changelog, the OpenAPI spec (queried with Python), the model catalog (476 models), and the source of OpenRouter's Vercel AI SDK provider, TypeScript SDK, and TypeScript agent at pinned commits; then the vendors' own pages (Anthropic, Google, OpenAI, DeepSeek, Moonshot, xAI) and one other client (pi) for the gaps.
- How many sources: 43 (24 OpenRouter documentation pages or page groups, the spec, the catalog, 3 OpenRouter repositories, 1 harness repository, 10 vendor pages, 3 workspace documents). 85 evidence items, 31 matrix rows.
- Read date: 2026-11-22. Nothing was run against a live model, and no credential was used.

## Candidate matrix rows

| Feature | Vendor(s) | Native rule (what the vendor requires or offers) | How OpenRouter exposes it (exact field names, behavior) | What OpenRouter loses or flattens | Deciding facts for Runvil (who holds state, round trip needed, neutral or vendor-specific values, route) | Basis | Evidence |
|---|---|---|---|---|---|---|---|
| Reasoning on or off, and effort level | All reasoning vendors | Each vendor has its own switch and level names: OpenAI `reasoning.effort`, Anthropic `output_config.effort`, Google `thinking_level`, DeepSeek `thinking.type` plus `reasoning_effort`, Kimi `reasoning_effort` | `reasoning: {enabled, effort}` with levels `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`; shorthand `reasoning_effort`; per-model `supported_efforts` in the catalog; an unsupported level is mapped to the nearest one | A level means a budget share on old Claude, response effort on new Claude, a thinking level on Gemini; per-model mapping tables are unpublished; the same body is a 400 on a pinned slug and a downgrade on a `~latest` alias | Neutral setting translated in the gateway; stateless; a caller that never reads the catalog can still hit a 400 on mandatory models | OR-documented | R1, R2, R4, R14, R15, R16, R26, R27, R28 |
| Reasoning token budget | Anthropic 4.6 and older, Gemini 2.5, some Qwen | `thinking.budget_tokens` (minimum 1024), `thinkingBudget`, `thinking_budget` | `reasoning.max_tokens`; a formula turns effort into a budget for budget models; sending it with `effort` is rejected | Accepted and dropped on adaptive-only Claude and Gemini 4; imprecise on Gemini 3; budget-to-effort thresholds unpublished; catalog flag `supports_max_tokens` under-reports | Neutral number translated in the gateway; newer models ignore it, so effort is the lasting control | OR-documented | R3, R13, R17, R19, R20, R27 |
| Reasoning that cannot be turned off | Claude Opus 5.5, Sonnet 5.5, Fable; Gemini 4; Grok 4.5 and later; GPT-6 Astra; gpt-oss; GLM 5.3 | Vendor rejects a disable request or offers no off switch | Catalog `reasoning.mandatory: true`; a disable request returns 400 `Reasoning is mandatory for this endpoint and cannot be disabled.` before routing; `~latest` aliases coerce to the lowest effort | A thinking on/off switch is not portable: 151 of 354 reasoning models in the catalog are flagged mandatory, where off is a 400 (or a silent downgrade on the alias ids) | Stateless; a portable "thinking off" switch meets a 400 on these models unless the gateway clamps it, which is what OpenRouter does on its `~latest` aliases | OR-documented | R14, R15, R22 |
| Adaptive thinking and `output_config.effort` | Anthropic 4.6 and later | `thinking: {type: "adaptive"}` plus `output_config.effort` (`low` to `max`); budgets rejected from 4.7 | `reasoning.enabled` or `reasoning.effort`; top-level `verbosity` also sets `output_config.effort` and wins | Two neutral fields feed one vendor field; the meaning of `reasoning.effort` changed on 2026-06-22; `minimal` becomes `low` | Gateway only | OR-documented | R11, R20, R21, R22, R77 |
| `between_tools` thinking | Anthropic Sonnet 5.5 | `thinking: {type: "between_tools"}`, effort `high` or below | Messages route only | Chat Completions cannot select it; the lowest chat setting is adaptive at `low` | Reachable only on the non-chat route, or needs a new neutral value | OR-documented | R23, R77 |
| Thinking display: summarized, omitted, updates | Anthropic | `thinking.display`; default `omitted` on the Claude 5 family | OpenRouter sends `summarized` by default; `omitted` through Messages `thinking.display` or chat `reasoning.exclude`; `updates` through Messages only | `updates` progress notes have no chat control; text between tool calls moves into reasoning output | Gateway default plus one neutral hide switch; `updates` is bound to the Messages route | OR-documented | R5, R24 |
| Pro reasoning mode | OpenAI GPT-5.6 and later | `reasoning.mode: "pro"`, honored by OpenAI and Azure | `reasoning.mode`; request rerouted to the `*-pro` model; field stripped for providers that lack it | A vendor-specific value sits in the neutral object and is silently stripped elsewhere | Caller picks a vendor-specific value or a `-pro` model id; stateless | OR-documented | R7 |
| Reasoning context selection | OpenAI GPT-5.6 and later | `reasoning.context`: `auto`, `all_turns`, `current_turn` | Same field on Chat Completions and Responses | Only has an effect when the caller replays reasoning items | Caller holds the state (replayed items) and picks a vendor-specific value | OR-documented | R8, R79 |
| Reasoning summaries | OpenAI, Anthropic | OpenAI `reasoning.summary`; Anthropic summarized thinking; raw reasoning is never returned | `reasoning.summary` (`auto`, `concise`, `detailed`); summaries arrive as `reasoning.summary` or `reasoning.text` entries | Visible text is shorter than the billed reasoning tokens | Display is gateway only; replay needs the opaque part | OR-documented | R6, R37, R72, R75 |
| Effort change in the middle of a conversation | Anthropic (Fable 5.1, 5.2, Opus 5, 5.5 family), OpenAI GPT-6 | Anthropic per-message `output_config` on a system message; OpenAI `configuration_update` input item | Chat Completions: a content-less system message with `configuration_update: {reasoning: {effort}}`; converted between the three route shapes; 400 if the model lacks it | An OpenRouter extension on chat; fixed placement rules; rejected on batch | Caller holds state: the update message must stay at the same list position on every later request | OR-documented | R30 |
| Signed thinking blocks | Anthropic | `thinking` block with `signature`; must return complete and unmodified within a tool-use turn; 400 if modified | `reasoning.text` entry with `signature` and `format: "anthropic-claude-v1"` | Position among text and tool blocks inside one message; a bare `reasoning` string becomes unsigned thinking that Anthropic rejects | Must make a round trip through the caller unchanged; one opaque array per assistant message | OR-documented | R36, R52, R57, R61, R73 |
| Redacted thinking | Anthropic | `redacted_thinking` block with `data` | `reasoning.encrypted` entry with `data` | Shares one entry type with OpenAI and Gemini blobs; told apart by `format` | Round trip unchanged | OR-documented | R37, R74 |
| Prefix-bound (preserved) thinking | Anthropic Fable 5.1 and later, 5.5 family | A block is valid only while system, tools, and earlier messages are unchanged; 400, or dropped with `drop_block` | Upstream 400 passed through; `thinking.block_binding.prefix_mismatch_behavior` and `input_transformations` exist on the Messages route (and `input_transformations` on Responses) | The chat route offers neither the drop switch nor the report | Caller must keep history append-only; state is held by the caller; the opt-out is on a non-chat route | OR-documented | R62, R76 |
| Account-bound thinking | Anthropic Sonnet 5.5, Haiku 5.5, Fable 5.2 | Blocks readable only by the producing account or a linked one; silently dropped elsewhere | OpenRouter pins routing to a provider that can read the blocks; 400 if provider preferences exclude it | Provider fallback is lost for conversations that replay thinking | The gateway must know which upstream account produced a block, so that tag has to travel with the block | OR-documented | R63, R76 |
| Encrypted reasoning items | OpenAI, Azure, Bedrock | Reasoning item with `encrypted_content`; return items in order with function calls | Chat: `reasoning.encrypted` entry (`data`, `id` like `rs_...`) plus `reasoning.summary`; requested automatically | Item boundaries and order relative to messages and calls are flattened into one array; `phase` has no chat form | Round trip unchanged; the same entry twice is rejected | OR-documented | R35, R37, R58, R65, R67, R79 |
| Thought signatures | Google Gemini | On `generateContent` a signature sits on any part (a `functionCall` part or the last part); resend exactly | Message-level `reasoning.encrypted` entry with `format: "google-gemini-v1"`; the tool call stays plain | Which part the signature belongs to; OpenRouter's own client strips Gemini thought text on replay; finish reason arrives as `stop` with tool calls | Round trip unchanged; the part mapping has to live inside the opaque data | OR-documented; part mapping is an open question | R38, R39, R56, R78 |
| Long-decode continuation token | Google Gemini 4 | A continuation token resumes output past the per-request limit | `reasoning.encrypted` entry with `id: "gemini-continuation"`; resend as the last assistant turn; routing pinned to the issuing provider | Token can reach several MB; only streaming delivers checkpoints | Round trip unchanged; large request bodies; provider pin | OR-documented | R40, R49, R64 |
| Encrypted reasoning | xAI | `encrypted_content` on the Responses API only; the vendor says Chat Completions "has no field for the ciphertext" | Format values `xai-responses-v1`, `bedrock-xai-responses-v1`, `google-vertex-xai-responses-v1` in `reasoning_details` | Entry types per format are undocumented | A gateway has to call the vendor's Responses API upstream to get replayable reasoning | Vendor-documented; OpenRouter side OR-inferred | R34, R82 |
| Required `reasoning_content` replay | DeepSeek | With `tools`, `reasoning_content` must return on every later request or 400 | Returned as `reasoning` plus `reasoning_details` (which can be `[]`); the caller returns them; `reasoning_content` accepted as an input alias | The requirement is undocumented by OpenRouter; an empty array is a meaningful signal | Round trip of plain text; a gateway can fill the vendor field from what the caller returns | OR-inferred | R53, R57, R59, R80 |
| Preserved thinking | Moonshot Kimi K3, K3.1 | Return the whole assistant message with `reasoning_content`; reasoning always on | Same plain-text lane as DeepSeek; catalog lists `none` for K3.1 | Catalog and vendor disagree on whether reasoning can be disabled | Round trip of plain text | Vendor-documented; OpenRouter side undocumented | R28, R81 |
| Plain-text reasoning | Qwen, GLM, gpt-oss, MiniMax, DeepSeek R1 | `reasoning_content` field, `reasoning` field, or think-tags inside content | Normalized to `message.reasoning` and `reasoning.text` entries; replay optional through the `reasoning` string | The upstream placement (three shapes) is hidden | Gateway only; replay is optional plain text | OR-documented | R31, R44, R51 |
| Interleaved thinking with tool calls | Anthropic | Thinking between tool calls; beta header on older models, automatic with adaptive thinking | `x-anthropic-beta` allowlist forwards `interleaved-thinking-2025-05-14`; each tool-loop step returns its own assistant message with its own array | Block order inside one assistant message | Order inside a message needs the vendor's ordered block list | OR-documented; the loss is OR-inferred | R25, R61, R77 |
| Streaming of reasoning | All | Anthropic `thinking_delta` then one `signature_delta`; OpenAI summary deltas; Google signature delta last | `delta.reasoning` and `delta.reasoning_details`; merge by type and order, a signature closes a text entry, encrypted entries are pushed whole | `index` is unreliable; a signature can arrive after answer text starts; the docs give one sentence | Client must accumulate until the stream ends; the rule lives in client code | OR-documented | R45, R46, R47, R49, R75 |
| Reasoning on the Responses route | All | OpenAI's item model | `reasoning` items with `summary`, `content`, `encrypted_content`, plus OpenRouter's `signature` and `format`; stateless; `previous_response_id` is a 400 | Item order is kept; vendor limits are the same as on chat | Caller holds all state and echoes `output` into `input` | OR-documented | R66, R67, R70 |
| Reasoning on the Messages route for non-Anthropic models | OpenAI, Gemini served in Anthropic shape | OpenRouter's own mapping | `thinking` block with `signature: ""` plus a `redacted_thinking` block holding the vendor blob in an OpenRouter envelope | An empty signature looks unsigned; clients that rewrite unsigned blocks break replay | Round trip unchanged; a client must not judge blocks by their signature | OR-documented | R69 |
| OpenRouter's own state inside encrypted reasoning | OpenRouter (deferred tools, server tools, video navigation) | OpenRouter's own feature | Extra `reasoning.encrypted` or `reasoning.server_tool_call` entries; 400 codes such as `deferred_tools_state_missing` when they are missing | A client that filters reasoning breaks tool search and server tools | The opaque field doubles as the gateway's own state channel | OR-documented | R33, R42 |
| Origin tag on opaque reasoning | All | Blocks are valid only for the producing model family, account, or provider | `format` enum (12 values) on each entry; the model snapshot and provider are written into the blob as a suffix | The caller-visible tag is coarse (wire family and cloud); the fine tag is inside the blob | A tag with protocol and model travels with the data and the caller never reads it | OR-documented for `format`; OR-inferred for the suffix | R34, R41, R63 |
| Reasoning usage and billing | All | Vendors bill reasoning as output | `completion_tokens_details.reasoning_tokens` (chat), `output_tokens_details.reasoning_tokens` (Responses), `output_tokens_details.thinking_tokens` (Messages); `pricing.internal_reasoning` | One count; summarized text is shorter than billed tokens | Gateway only | OR-documented | R71, R72 |
| Hiding reasoning from the response | All | Anthropic `display: "omitted"`; others vary | `reasoning.exclude` (chat only); legacy `include_reasoning` | Opaque entries can still come back with exclude set (Gemini continuation) | Neutral switch; opaque replay data must survive it | OR-documented | R5, R10 |
| `:thinking` model variant | OpenRouter (legacy) | OpenRouter's own naming | Deprecated; zero ids in the catalog | A model-id suffix could only switch reasoning on; it could not set a level or budget per request | OpenRouter moved the control from the model id to a per-request field | OR-documented | R12 |
| Model or provider switch in the middle of a conversation | All | Anthropic drops unreadable blocks silently; OpenAI omits incompatible reasoning; Google says "The backend manages compatibility" | Undocumented in general; routing pins and 400s in special cases | Unknown (open question Q1) | One client (pi) drops all thinking when provider, API, or model changes | OR-documented for the special cases; Filled (pi) | R64, R76, R78, R79, R83 |

## Evidence

Every item gives a claim, the exact quote or quotes, the source key, the basis label, and a confidence note where the claim rests on inference. Quotes are word for word. Where a source sentence contains a long dash, the quote is split into two fragments around it. Where a quoted sentence contains a Markdown link, the link target is sometimes cut and the link text is kept in square brackets. Line numbers for repository files are from the pinned commits in the Sources table. Read date for every source is 2026-11-22.

### A. Request side: the `reasoning` object, its relatives, and what happens to requests a model cannot honor

**R1. The Chat Completions `reasoning` request object has seven properties: `context`, `effort`, `enabled`, `exclude`, `max_tokens`, `mode`, `summary`.**

- Quote (spec, `ChatRequest.properties.reasoning`): description "Configuration options for reasoning models", example `{"effort": "medium", "summary": "concise"}`, properties `context`, `effort`, `enabled`, `exclude`, `max_tokens`, `mode`, `summary`.
- Quote (spec, `enabled`): "Enables reasoning with default settings. Defaults to being inferred from `effort` or `max_tokens`."
- Quote (spec, `exclude`): "When true, the model still reasons but the reasoning is omitted from the response. Defaults to false."
- Quote (spec, `max_tokens`): "Maximum number of tokens the model may spend on reasoning, for models that take a reasoning token budget instead of an effort level."
- The Responses route uses the schema `ReasoningConfig`, which is `BaseReasoningConfig` (`context`, `effort`, `mode`, `summary`) plus `enabled` and `max_tokens`. `exclude` is a Chat Completions property only (see R5).
- Source: R-s15. Basis: OR-documented.

**R2. The full `effort` value list is `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`.**

- Quote (spec, `ReasoningEffort` and `ChatRequest.reasoning.effort`): enum `["max", "xhigh", "high", "medium", "low", "minimal", "none", null]`, with `"x-speakeasy-unknown-values": "allow"`.
- Quote (guide): `"effort": "high", // Can be "max", "xhigh", "high", "medium", "low", "minimal" or "none" (OpenAI-style)`
- Quote (guide): `"max_tokens": 2000, // Specific token limit (Anthropic-style)`
- Sources: R-s15, R-s1. Basis: OR-documented.

**R3. `effort` and `max_tokens` cannot be sent together. The rejection string is documented.**

- Quote (spec, `reasoning.max_tokens`): "Cannot be combined with `reasoning.effort` or the top-level `reasoning_effort`: a request that sets both is rejected with \"Only one of \"reasoning.effort\" and \"reasoning.max_tokens\" can be specified\"."
- Quote (guide): "// One of the following (not both):"
- `enabled` combines with either one. Quote (Opus 5 guide): `"reasoning": { "enabled": false, "effort": "high" }` with the comment "OK: disabled reasoning at high effort".
- Sources: R-s15, R-s1, R-s8. Basis: OR-documented.

**R4. `enabled: true` alone means medium effort.**

- Quote: "`\"enabled\": true` - Enables reasoning at the \"medium\" effort level with no exclusions."
- On newer Claude models the meaning shifts to "adaptive thinking, effort left to the model" (see R21, R22, R29).
- Source: R-s1. Basis: OR-documented.

**R5. `exclude` hides reasoning from the response on Chat Completions only. The Responses route ignores it. Tokens are still billed.**

- Quote: "`\"exclude\": true` - The model will still use reasoning, but it won't be returned in the response. The tokens are still billed and [count against `max_tokens`](#reasoning-tokens-and-max_tokens)."
- Quote: "`exclude` is only honored on Chat Completions (`/api/v1/chat/completions`). The Responses API ignores it and always returns `reasoning` items in `output`; filter them out client-side if you do not want them."
- Exception that matters for replay (Gemini 4 guide): "With `reasoning: { \"exclude\": true }`, reasoning text is hidden but the continuation item is still returned, so long decode keeps working."
- Sources: R-s1, R-s14. Basis: OR-documented.

**R6. `summary` takes `auto`, `concise`, `detailed`.**

- Quote (spec, `ChatReasoningSummaryVerbosityEnum` and `ReasoningSummaryVerbosity`): enum `["auto", "concise", "detailed", null]`.
- Quote (guide example, Responses route, `openai/o3`): `reasoning: { effort: 'high', summary: 'detailed' }` with the comment "Raw reasoning arrives in `content`; OpenAI models return a summary in `summary` instead."
- The guide gives the enum and this example. It gives no per-vendor mapping for `summary` outside OpenAI.
- Sources: R-s15, R-s1. Basis: OR-documented.

**R7. `mode` takes `standard` or `pro`, is OpenAI GPT-5.6 and newer only, and changes routing.**

- Quote (spec, `ReasoningMode`): "Selects the reasoning mode. `standard` is the default; `pro` engages deeper reasoning on models that support it, billed at standard token rates. Only supported by OpenAI GPT-5.6 and newer."
- Quote (guide): "Send `reasoning.mode: \"pro\"` with the standard model"; "OpenRouter reroutes the request to the matching `*-pro` model."
- Quote (guide): "`reasoning.mode` is only supported by OpenAI GPT-5.6 and newer, when served by OpenAI or Azure. Amazon Bedrock's OpenAI-compatible API accepts the field but silently ignores it, so pro reasoning is not available on Bedrock"
- Quote (GPT-5.6 guide): "OpenRouter strips `reasoning.mode`, `prompt_cache_breakpoint`, and `prompt_cache_options` before forwarding requests to providers that don't support them, so requests that fall back to other models keep working."
- Sources: R-s15, R-s1, R-s12. Basis: OR-documented.

**R8. `context` takes `auto`, `all_turns`, `current_turn` and decides which replayed reasoning the model may use. OpenAI GPT-5.6 and newer only.**

- Quote (spec, `ReasoningContext`): "Controls which reasoning is available to the model. `auto` uses the model default (same as omitting); `all_turns` includes reasoning from earlier turns passed in input; `current_turn` limits to the current turn only. Only supported by OpenAI GPT-5.6 and newer."
- Quote (guide): "**`current_turn`**: The model only uses reasoning from the current turn. Prior reasoning items in the input are ignored."
- Quote (guide): "The default value of `context` may vary by model. Set it explicitly if your use case requires a specific behavior."
- The property is in the Chat Completions spec too (changelog, November 11, 2026: "`ChatRequest`: new property `reasoning.context`").
- Sources: R-s15, R-s1, R-s17. Basis: OR-documented.

**R9. Top-level `reasoning_effort` is a shorthand for `reasoning.effort`.**

- Quote (spec): "Shorthand for setting reasoning effort. Equivalent to setting reasoning.effort. Cannot be used simultaneously with reasoning.effort if they differ."
- The spec enum for it includes `max`. The parameters page lists it as "Optional, **enum** (xhigh, high, medium, low, minimal, none)" and describes it as "OpenAI-style reasoning effort setting." (see Conflicts, C3).
- Sources: R-s15, R-s4. Basis: OR-documented.

**R10. The legacy `include_reasoning` boolean still works on Chat Completions only.**

- Quote (guide): "`include_reasoning: true` - Equivalent to `reasoning: {}`" and "`include_reasoning: false` - Equivalent to `reasoning: { exclude: true }`"
- Quote (guide): "For backward compatibility, Chat Completions (`/api/v1/chat/completions`) still accepts the following legacy parameters. The Responses API does not accept them"
- Quote (parameters page): "Deprecated alias for reasoning.exclude. When true, reasoning tokens are returned in the response when supported by the model."
- The `ChatRequest` schema in the spec lists 43 properties, and `include_reasoning` is documented on the guide and parameters pages only, outside that list (see Conflicts, C2). The model catalog still lists `include_reasoning` in `supported_parameters` for 359 of 476 models.
- OpenRouter's own Vercel provider still sends it: `include_reasoning: this.settings.includeReasoning,` (R-s27, `src/chat/index.ts` line 218), marked `@deprecated use \`reasoning\` instead` in `src/types/index.ts` lines 48 to 51.
- Sources: R-s1, R-s4, R-s15, R-s16, R-s27. Basis: OR-documented.

**R11. `verbosity` is a separate top-level parameter. For Claude it sets `output_config.effort`. For OpenAI it is `text.verbosity`.**

- Quote (parameters page): "Key: `verbosity`", "Optional, **enum** (low, medium, high, xhigh, max)", "Default: **medium**".
- Quote (parameters page): "For Anthropic models, this parameter maps to `output_config.effort`. The 'xhigh' level is supported by Anthropic Claude 4.7 Opus and later models. The 'max' level is supported by Anthropic Claude 4.6 Opus and later models."
- Quote (Claude 4.6 guide): "If both are passed, `verbosity` wins."
- Quote (guide, Messages section): "OpenAI models expose two knobs, `reasoning.effort` and `text.verbosity`, and the Messages API request shape has one. `output_config.effort` maps to `reasoning.effort` only, and it is not sent as `text.verbosity`."
- Spec: the Responses route has `text.verbosity` (`TextExtendedConfig`, enum `low`, `medium`, `high`, `xhigh`, `max`). On Chat Completions, `verbosity` is documented by the parameters page and by 114 catalog entries (`supported_parameters` contains `verbosity`), while the spec's `ChatRequest` schema lists 43 other properties (see Conflicts, C2).
- Sources: R-s4, R-s5, R-s1, R-s15, R-s16. Basis: OR-documented.

**R12. The `:thinking` model variant is deprecated. The live catalog has zero ids with that suffix.**

- Quote: "The `:thinking` variant is deprecated and is being removed. Use the [`reasoning` parameter](https://openrouter.ai/docs/guides/best-practices/reasoning-tokens) instead, which works across models and lets you control the reasoning budget per request."
- Quote: "Reasoning is now requested per call through the `reasoning` parameter, so no new models will carry this suffix."
- Catalog check: 0 of 476 model ids contain `:thinking`.
- Sources: R-s3, R-s16. Basis: OR-documented.

**R13. The documented effort-to-budget formula, and the reverse direction.**

- Quote: "The reasoning token allocation is capped at 128,000 tokens maximum and 1024 tokens minimum. The formula for calculating the budget\_tokens is: `budget_tokens = max(min(max_output_tokens * {effort_ratio}, 128000), 1024)`"
- Quote: "effort\_ratio is 0.95 for max and xhigh effort, 0.8 for high effort, 0.5 for medium effort, 0.2 for low effort, and 0.1 for minimal effort."
- Quote: "When using the `reasoning.max_tokens` parameter, that value is used directly with a minimum of 1024 tokens."
- Quote: "**Important**: `max_output_tokens` must be strictly higher than the reasoning budget to ensure there are tokens available for the final response after thinking."
- Reverse direction, budget to effort: "For models that only support `reasoning.effort` (see below), the `reasoning.max_tokens` value will be used to determine the effort level." The guide gives the percentages (95, 95, 80, 50, 20, 10) and stops there. The thresholds that turn a budget into a level are left unstated.
- Forward direction for budget-only models: "For models that only support `reasoning.max_tokens`, the effort level will be set based on the percentages above."
- The formula is written for Anthropic budget models. It stopped applying to Claude 4.6 and newer on June 22, 2026 (see R20).
- Source: R-s1. Basis: OR-documented.

**R14. Each catalog model has a `reasoning` object that tells a client what it may send.**

- Quote (guide): "**`supported_efforts`**: Filter effort selectors to these values, returned in descending effort order (highest first). When `null`, all gateway effort values are accepted. When omitted, the model does not expose effort selection."
- Quote (guide): "**`default_effort`**: Pre-select this effort when enabling reasoning. Maps to `reasoning.effort` in chat requests."
- Quote (guide): "**`supports_max_tokens`**: When present and `true`, show a token budget control and send `reasoning.max_tokens` instead of (or alongside) `reasoning.effort`."
- Quote (guide): "**`mandatory`**: When `true`, hide disable controls and do not send `effort: \"none\"`" and "the model rejects it."
- Quote (guide): "Non-reasoning models and dynamic router models (`openrouter/auto`, `openrouter/free`) omit the `reasoning` field."
- Quote (spec, `ModelReasoning.mandatory`): "When true, reasoning cannot be disabled and effort \"none\" is rejected."
- Catalog counts (476 models): 354 have a `reasoning` object; 151 have `mandatory: true`; 239 have `supported_efforts` and `default_effort`; 188 have `default_enabled` (160 true, 28 false); 9 have `supports_max_tokens: true` (`qwen/qwen3.8-omni-flash`, `qwen/qwen3.8-flash`, `qwen/qwen3.7-flash`, four `nvidia/nemotron-3-*` entries, `anthropic/claude-opus-4.6` and its `:batch` twin). There are 31 distinct `supported_efforts` lists, from `["high"]` to all seven values.
- Sources: R-s1, R-s15, R-s16. Basis: OR-documented.

**R15. Asking a mandatory-reasoning model to stop reasoning is a 400 from OpenRouter itself, before routing, except on the `~...-latest` aliases where it is coerced.**

- Quote (Claude 5.5 guide, Opus 5.5): "**Disabling reasoning fails at OpenRouter, not upstream.** The model is registered as mandatory-reasoning, so `reasoning: {\"enabled\": false}`, `reasoning: {\"effort\": \"none\"}`, and Messages API `thinking: {\"type\": \"disabled\"}` return a 400 (`Reasoning is mandatory for this endpoint and cannot be disabled.`) before the request is routed. The exception is the `~anthropic/claude-opus-latest` alias, where OpenRouter coerces a disable request to the lowest supported effort instead."
- Quote (Sonnet 5.5): "The exception is the `~anthropic/claude-sonnet-latest` alias, where OpenRouter runs adaptive thinking instead of returning the 400 (`effort: \"none\"` becomes `low`)."
- So the same request body is an error on a pinned slug and a silent downgrade on a floating alias.
- Source: R-s10. Basis: OR-documented.

**R16. An effort level the model lacks is mapped to a supported level on request-level settings, and rejected with a 400 on mid-conversation updates.**

- Quote (guide, Gemini section): "If a model doesn't support a specific effort level (for example, if a model only supports `low` and `high`), OpenRouter will map your requested effort to the nearest supported level."
- Quote (guide, Gemini table): `"xhigh"` maps to `"high"` "(mapped down)".
- Quote (Claude 4.7 guide): "`'xhigh'` is only supported on Claude 4.7 Opus. `'max'` is supported on Claude 4.6+. For older models, both automatically fall back to `'high'`."
- Quote (Claude 4.6 guide): "`reasoning.effort: 'minimal'` maps to `'low'` (Anthropic's lowest level); `'none'` disables reasoning entirely, so no `output_config.effort` is sent."
- Quote (guide, mid-conversation): "OpenAI models accept the efforts listed in their [supported reasoning efforts](#discovering-per-model-reasoning-options), and an update with an effort the model does not support is rejected with a 400."
- Quote (GPT-6 guide): "An update whose effort has no supported equivalent is rejected with a 400."
- Whether a request-level `effort: "none"` on a non-mandatory model that lacks `none` in `supported_efforts` is mapped or rejected is not stated.
- Sources: R-s1, R-s5, R-s6, R-s13. Basis: OR-documented.

**R17. A token budget sent to a model that only does adaptive or level-based thinking is accepted and dropped, with no error.**

- Quote (Claude 4.7 guide): "`reasoning.max_tokens` is accepted but ignored"
- Quote (Claude 5.5 guide): "**Budgets are not forwarded.** `reasoning.max_tokens` and `thinking.budget_tokens` are accepted by OpenRouter but the request goes upstream as adaptive thinking with no budget"
- Quote (Gemini 4 guide): "`reasoning.max_tokens` is not supported, and reasoning cannot be turned off." and "**No reasoning budget.** `reasoning.max_tokens` does not set a thinking budget, and reasoning cannot be turned off. Use `reasoning.effort`."
- Quote (guide, Gemini 3): "If you specify `reasoning.max_tokens` explicitly, OpenRouter will pass it through as `thinkingBudget` to Google's API. However, for Gemini 3 models, Google internally maps this budget value to a `thinkingLevel`, so you will not get precise token control."
- Sources: R-s6, R-s10, R-s14, R-s1. Basis: OR-documented.

**R18. Some invalid combinations are left for the upstream vendor to reject, and OpenRouter returns that 400.**

- Quote (Opus 5 guide): "Anthropic only allows disabling thinking at effort levels up to and including `high`. Combining disabled reasoning with `xhigh` or `max` effort returns a 400 from Anthropic"
- Quote (Claude 5.5 guide, Haiku 5.5): "Combining that with effort `xhigh` or `max` returns a 400 from Anthropic."
- Quote (guide, Reasoning Mode): "It cannot be combined with a [mid-conversation effort update](#mid-conversation-effort), which OpenAI does not accept on pro models, so that combination is rejected with a 400."
- Sources: R-s8, R-s10, R-s1. Basis: OR-documented.

### B. How the neutral request maps to each vendor

**R19. Older Claude models (4.5 and earlier): effort becomes a thinking budget by formula, and `reasoning.max_tokens` is the budget itself.**

- Quote: "You can enable reasoning on Anthropic models **only** using the unified `reasoning` parameter with either `effort` or `max_tokens`."
- Quote: "When using the `reasoning.effort` parameter, the budget\_tokens are calculated based on the request's `max_output_tokens` value (`max_tokens` on Chat Completions)."
- Quote (Claude 4.6 guide): "`reasoning.effort` values still convert to `thinking.budget_tokens` for older models." and "Older models (4.5 Opus, 3.7 Sonnet, etc.) behave exactly as before"
- Formula in R13.
- Sources: R-s1, R-s5. Basis: OR-documented.

**R20. Claude 4.6: adaptive thinking by default, budgets still honored, and since June 22, 2026 `reasoning.effort` sets `output_config.effort`.**

- Quote: "For Claude 4.6 Opus and 4.6 Sonnet, OpenRouter now uses adaptive thinking (`thinking.type: 'adaptive'`) by default instead of budget-based thinking (`thinking.type: 'enabled'` with `budget_tokens`)."
- Quote: "If you explicitly set `reasoning.max_tokens`, budget-based thinking is used"
- Quote (update note): "As of June 22, 2026, OpenRouter maps `reasoning.effort` to Anthropic's `output_config.effort` on Claude 4.6 and newer models"; "previously it was ignored. `verbosity` is unchanged: it still sets `output_config.effort`, and wins if both are passed."
- Quote: "Thinking itself stays adaptive either way. On 4.6, `reasoning.effort` no longer influences a thinking token budget."
- So the same neutral field (`reasoning.effort`) meant "ignored" before that date and "response effort" after it, for the same model.
- Source: R-s5. Basis: OR-documented.

**R21. Claude 4.7, Sonnet 5, Opus 5: adaptive only, budgets ignored, and from Sonnet 5 and Opus 5 reasoning is on unless the caller turns it off.**

- Quote (Claude 4.7 guide): "Claude 4.7 Opus supports only adaptive thinking." and "`thinking.budget_tokens` is no longer supported upstream"
- Quote (Sonnet 5 guide): "Reasoning is [on by default](https://platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5#adaptive-thinking-on-by-default) on Sonnet 5: requests that don't configure [`reasoning`](https://openrouter.ai/docs/guides/best-practices/reasoning-tokens) run with adaptive thinking enabled. Disable it explicitly with `reasoning: { \"enabled\": false }` if your integration assumes non-reasoning responses."
- Quote (Opus 5 guide): "Like Sonnet 5, Opus 5 supports only adaptive thinking: token budgets (`reasoning.max_tokens` in Chat Completions, `thinking.budget_tokens` in Messages) are ignored."
- Quote (Sonnet 5 guide): "`'minimal'` maps to `'low'`, and `'none'` disables reasoning entirely so no `output_config.effort` is sent"
- The Sonnet 5 and 4.7 guides also say `temperature`, `top_p`, `top_k` "are no longer supported and will be ignored".
- Sources: R-s6, R-s7, R-s8. Basis: OR-documented.

**R22. Claude 5.5 family and Fable: thinking is always adaptive, cannot be disabled on Opus 5.5 and Sonnet 5.5, and what OpenRouter sends upstream differs per model.**

- Quote (Opus 5.5): "Anthropic returns a 400 `invalid_request_error` for `thinking: {\"type\": \"disabled\"}` and for `thinking: {\"type\": \"enabled\", \"budget_tokens\": ...}`. Omitting `thinking` or setting `{\"type\": \"adaptive\"}` are the only valid forms, and `output_config.effort` (`low`, `medium`, `high`, `xhigh`, `max`) decides how much the model thinks."
- Quote (Opus 5.5): "**Effort maps straight through.** Chat Completions `reasoning.effort` and Messages API `output_config.effort` both become Anthropic's `output_config.effort`. Requests that set no effort are sent without one and run at Anthropic's default of `medium`"
- Quote (Sonnet 5.5): "**No reasoning setting means adaptive thinking at `high`.** OpenRouter sends `thinking: {\"type\": \"adaptive\", \"display\": \"summarized\"}` with no effort, so Anthropic's default of `high` applies."
- Quote (Haiku 5.5): "**No reasoning setting means adaptive thinking at `medium`.** OpenRouter sends no `thinking` field, so Anthropic's default applies. A response can begin with a reasoning block even though the request never asked for one."
- Quote (Haiku 5.5): "**Disabling thinking works at `high` and below.** `reasoning: {\"enabled\": false}`, `reasoning: {\"effort\": \"none\"}`, and Messages API `thinking: {\"type\": \"disabled\"}` send `thinking: {\"type\": \"disabled\"}`."
- Catalog: `anthropic/claude-opus-5.5`, `anthropic/claude-sonnet-5.5`, `anthropic/claude-fable-5`, `-5.1`, `-5.2` all have `"mandatory": true` and `supported_efforts` of `["max", "xhigh", "high", "medium", "low"]`. `anthropic/claude-haiku-5.5` has `"mandatory": false` and `none` in its list.
- Sources: R-s10, R-s16. Basis: OR-documented.

**R23. Sonnet 5.5's `between_tools` setting (no up-front thinking) is reachable only through the Messages route.**

- Quote: "Sonnet 5.5 uses `thinking: {\"type\": \"between_tools\"}` as its lowest setting: the model does not think before responding, and the short notes it writes between tool calls come back as thinking blocks. Send it through OpenRouter's [Messages API](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md). It works at effort `high` and below. At `xhigh` or `max` Anthropic returns a 400."
- Quote: "**Reasoning can't be disabled on Chat Completions.**"
- Quote: "With `between_tools`, effort can't change mid-conversation."
- Spec: the Messages `thinking` field is a `oneOf` of four shapes with `type` equal to `enabled` (requires `budget_tokens`), `disabled`, `adaptive`, `between_tools`. The Chat Completions `reasoning` object's seven properties (R1) select on or off, effort, budget, and OpenAI's mode and context; `between_tools` is a `thinking.type` value that exists on the Messages route.
- Sources: R-s10, R-s15. Basis: OR-documented.

**R24. Thinking display: OpenRouter overrides Anthropic's default and asks for summaries. `omitted` and `updates` are set through the Messages route, or `reasoning.exclude` on Chat Completions.**

- Quote (guide): "For Claude models that support the `thinking.display` field, OpenRouter defaults to **summarized thinking** (`thinking.display: 'summarized'`), so you don't lose the reasoning trace on newer models where Anthropic omits the thinking by default."
- Quote (guide): "If you're using the Anthropic Messages API format directly, you can control this with `thinking.display`"
- Quote (Claude 5.5 guide): "Set `thinking.display` to `\"omitted\"`, or `reasoning.exclude: true` on Chat Completions, if you do not want it."
- Quote (Claude 5.5 guide): "`\"updates\"` is meant for long tool-using turns: it returns a short summary of each progress note in its thinking block and leaves the reasoning blocks empty"
- Spec (`AnthropicThinkingDisplay`): enum `["summarized", "omitted", "updates", null]`.
- Quote (Claude 5.5 guide, Sonnet): "Notes longer than a sentence or two that Sonnet 5.5 writes between tool calls come back as progress-update thinking blocks instead of text. Because OpenRouter requests summarized display, these notes appear in the reasoning output rather than disappearing. If your interface streams text between tool calls, read reasoning too"
- Sources: R-s1, R-s10, R-s15. Basis: OR-documented.

**R25. Anthropic beta features: OpenRouter attaches the beta for features it detects, and forwards a per-provider allowlist of `x-anthropic-beta` values, including interleaved thinking.**

- Quote (Fable 5.1 guide): "You do **not** need to send the Anthropic beta headers for any of them: OpenRouter detects requests that use each feature, attaches the corresponding beta for you, and routes those requests only to providers that support it."
- Quote (provider selection guide): the supported header table lists "Interleaved Thinking | `interleaved-thinking-2025-05-14` | Allows Claude's thinking/reasoning to be interleaved with regular output, rather than appearing as a single block".
- Quote (provider selection guide): "OpenRouter keeps an allowlist of other `x-anthropic-beta` values for each provider and forwards only the values on the list for the provider that serves the request." and "OpenRouter drops every other value, and some values it recognizes are withheld on purpose."
- Sources: R-s9, R-s24. Basis: OR-documented.

**R26. OpenAI: effort levels pass in OpenAI's own vocabulary, with per-model lists. `none` is accepted by some tiers and rejected by others.**

- Quote (guide): "Currently supported by OpenAI reasoning models (o1 series, o3 series, GPT-5 series) and Grok models"
- Quote (guide, Messages section): "a model such as GPT-5 or MiniMax receives `reasoning_effort: \"high\"` in its own vocabulary, translated the same way as request-level `reasoning.effort`."
- Quote (GPT-6 guide): "Sol and Luna accept `none` as a reasoning effort, Astra does not."
- Catalog: `openai/gpt-6-astra` is `{"mandatory": true, "supported_efforts": ["max", "xhigh", "high", "medium", "low"], "default_effort": "medium"}`; `openai/gpt-6-sol` adds `none` and has `"mandatory": false`; `openai/gpt-5` is `{"mandatory": true, "supported_efforts": ["high", "medium", "low", "minimal"], "default_effort": "medium"}`; `openai/o3` is `{"mandatory": true}`.
- Quote (guide): "While most models and providers make reasoning tokens available in the response, some (like the OpenAI o-series) do not."
- Sources: R-s1, R-s13, R-s16. Basis: OR-documented.

**R27. Gemini 3: effort maps one-to-one to `thinkingLevel`; a budget is passed as `thinkingBudget` but Google turns it back into a level. Gemini 2.5 takes budgets. Gemini 4 takes levels only and always reasons.**

- Quote: "Gemini 3 models (such as [google/gemini-3.1-pro-preview](https://openrouter.ai/google/gemini-3.1-pro-preview) and [google/gemini-3-flash-preview](https://openrouter.ai/google/gemini-3-flash-preview)) use Google's `thinkingLevel` API instead of the older `thinkingBudget` API used by Gemini 2.5 models."
- Quote: "OpenRouter maps the `reasoning.effort` parameter directly to Google's `thinkingLevel` values" with the table `minimal` to `minimal`, `low` to `low`, `medium` to `medium`, `high` to `high`, `xhigh` to `"high"` (mapped down).
- Quote: "When using `thinkingLevel`, the actual number of reasoning tokens consumed is determined internally by Google."
- Quote (max tokens section): the supported list starts with "Gemini thinking models".
- Quote (Gemini 4 guide): "Gemini 4 Argon always reasons. `reasoning.effort` accepts `low`, `medium`, and `high`."
- Catalog: `google/gemini-4-argon` is `{"mandatory": true, "supported_efforts": ["high", "medium", "low"], "default_effort": "medium"}`; `google/gemini-4-neon` adds `xhigh`; `google/gemini-2.5-pro` is `{"mandatory": true}`; `google/gemini-2.5-flash` is `{"mandatory": false}`.
- The guide gives no statement about Google's `includeThoughts` switch. It says reasoning is "included in the response by default if the model decides to output them" and offers `exclude` to hide it.
- Sources: R-s1, R-s14, R-s16. Basis: OR-documented.

**R28. xAI, DeepSeek, Kimi, GLM, Qwen, gpt-oss, MiniMax, Mistral: the docs give one line each or none. The catalog is the per-model source of truth.**

- xAI: the guide names "Grok models" under effort support. Catalog: `x-ai/grok-4.7` is `{"mandatory": true, "default_enabled": true, "supported_efforts": ["xhigh", "high", "medium", "low"], "default_effort": "high"}`; `x-ai/grok-4.3` adds `none`, `"mandatory": false`, `"default_effort": "low"`.
- DeepSeek: the guide's exclude example uses `deepseek/deepseek-r1-0528` with `"reasoning": { "effort": "high", "exclude": true }`. Catalog: `deepseek/deepseek-v4.1-flash` is `{"mandatory": false, "default_enabled": true, "supported_efforts": ["max", "high", "low", "none"], "default_effort": "high"}`; `deepseek/deepseek-v4-pro` is `["xhigh", "high", "none"]`; `deepseek/deepseek-r1-0528` is `{"mandatory": true}`.
- Kimi: `moonshotai/kimi-k3.1-pro` is `{"mandatory": false, "default_enabled": true, "supported_efforts": ["max", "high", "low", "none"], "default_effort": "max"}`; `moonshotai/kimi-k2-thinking` is `{"mandatory": true, "default_enabled": true}`.
- GLM: `z-ai/glm-5.3` is `{"mandatory": true, "default_enabled": true, "supported_efforts": ["max", "high", "low"], "default_effort": "max"}`; `z-ai/glm-5.2` is `["xhigh", "high", "none"]`.
- Qwen: quote "Some Alibaba Qwen thinking models (mapped to `thinking_budget`)" and "For Alibaba, support varies by model". Catalog: `qwen/qwen3.8-flash` is `{"mandatory": false, "default_enabled": true, "supports_max_tokens": true}`; `qwen/qwen3.8-max-0902` is mandatory with `["xhigh", "high", "medium", "low", "minimal"]`.
- gpt-oss: `openai/gpt-oss-120b` is `{"mandatory": true, "supported_efforts": ["high", "medium", "low"], "default_effort": "medium"}`.
- MiniMax: `minimax/minimax-m3.1-flash` is mandatory with `["max", "xhigh", "high", "medium", "low"]`, default `max`. Mistral: `mistralai/mistral-large-4-0` is `["high", "none"]`.
- A cross-check on DeepSeek: the vendor's own table maps `medium` to `high` and `xhigh` to `high` (R80), and OpenRouter's catalog lists DeepSeek V4.1 with `max`, `high`, `low`, `none`. The two agree on the distinct levels the model really has.
- Sources: R-s1, R-s16. Basis: OR-documented.

**R29. Requests on the Messages route use Anthropic's native fields, and OpenRouter folds them into the same neutral `reasoning` settings for every model.**

- Quote: "Requests sent through the [Anthropic Messages API](https://openrouter.ai/docs/api/api-reference/anthropic-messages/create-a-message.md) control reasoning with Anthropic's native `thinking` and `output_config.effort` fields. OpenRouter normalizes both into the unified `reasoning` parameter, so they work on every reasoning model, not only Claude."
- Table, quoted: `output_config: { effort }` equals `reasoning: { effort }`; `thinking: { type: "enabled", budget_tokens }` equals `reasoning: { max_tokens: budget_tokens }`; `thinking: { type: "adaptive" }` equals `reasoning: { enabled: true }`; `thinking: { type: "disabled" }` equals `reasoning: { enabled: false }`.
- Quote: "When `thinking: { type: \"enabled\", budget_tokens }` and `output_config.effort` are sent together, Claude receives both unchanged. The unified `reasoning` parameter accepts either `effort` or `max_tokens` but not both, so every other model receives `reasoning.effort` and the budget is not forwarded."
- Quote: "`thinking: { type: \"disabled\" }` disables reasoning even when an `output_config.effort` is also present. Claude still receives the `output_config.effort`, since Anthropic applies it to the whole response, and other models receive no reasoning effort."
- Quote: "Anthropic's `xhigh` and `max` levels stay reasoning efforts and are translated to the model's [supported reasoning efforts](#discovering-per-model-reasoning-options)."
- Source: R-s1. Basis: OR-documented.

**R30. Effort can change in the middle of a conversation without breaking the prompt cache. Each route has its own shape, and OpenRouter converts between them.**

- Quote: "Some models instead accept an effort change as an item in the conversation history. The change applies from that point onward and stays in effect until the next change, the preceding items are untouched, and the cache stays warm. Keep the item at the same position in every later request."
- Chat Completions shape, quoted: `{ "role": "system", "content": "", "configuration_update": { "reasoning": { "effort": "low" } } }`
- Responses shape, quoted: `{ "type": "configuration_update", "reasoning": { "effort": "low" } }`
- Messages shape, quoted: `{ "role": "system", "content": [], "output_config": { "effort": "low" } }`
- Quote: "The Chat Completions form is an OpenRouter extension, since OpenAI's own Chat Completions API has no per-message equivalent."
- Quote: "The three forms are interchangeable: OpenRouter normalizes each into one internal representation and re-emits it in the form the selected provider accepts. A Responses `configuration_update` sent to a Claude model becomes a per-message `output_config`, and a Chat Completions or Messages API update sent to an OpenAI model becomes a Responses `configuration_update` item."
- Quote (GPT-6 guide): "The field is accepted only on `system` and `developer` messages. A `configuration_update` on a user, assistant, or tool message is rejected with a 400."
- Quote: "Claude accepts `low`, `medium`, `high`, `xhigh`, and `max` (`minimal` is sent as `low`, `none` is rejected)."
- Quote: "Do not place two updates next to each other. The first would apply to nothing; OpenAI rejects adjacent `configuration_update` items, and OpenRouter applies the same rule on every API so the three forms stay interchangeable."
- Quote: "Models that accept mid-conversation effort changes include Claude Fable 5.2, Claude Fable 5.1, Claude Opus 5, and the standard OpenAI GPT-6 models (Astra, Sol, Luna) and GPT-6.1 models (Sol, Terra"
- Quote: "Requests that use an update are routed only to endpoints that accept it, and a request for a model that does not support it is rejected with a 400 instead of being sent with the update silently dropped."
- Quote: "Mid-conversation effort updates are not accepted on the [Batch API](https://openrouter.ai/docs/batch-quickstart.md) and are rejected per line."
- Spec: `ConfigurationUpdate` has `"additionalProperties": false` and one required property `reasoning`, whose only property is `effort` with enum `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`. Changelog: `ConfigurationUpdateReasoning` and `ChatSystemMessage.configuration_update` were added September 4, 2026; `ChatDeveloperMessage.configuration_update` September 8, 2026.
- Deciding fact: the update is state that lives in the caller's message list and must stay at the same position on every later request.
- Sources: R-s1, R-s13, R-s9, R-s15, R-s17. Basis: OR-documented.

### C. Response side: `reasoning` and `reasoning_details`

**R31. The plain-text field is `message.reasoning` (and `delta.reasoning` when streaming).**

- Quote (spec, `ChatAssistantMessage.reasoning`): description "Reasoning output", type string or null.
- Quote (spec, `ChatStreamDelta.reasoning`): description "Reasoning content delta", example "I need to".
- Quote (guide): "On Chat Completions they appear in the `reasoning` field of each message, and `reasoning.exclude` can omit them."
- `ChatAssistantMessage` in the spec has ten properties: `audio`, `content`, `images`, `model`, `name`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`. `ChatStreamDelta` has seven: `audio`, `content`, `reasoning`, `reasoning_details`, `refusal`, `role`, `tool_calls`. The name `reasoning_content` appears in the docs as an accepted input alias (R53) and in neither schema.
- Sources: R-s15, R-s1. Basis: OR-documented.

**R32. `reasoning_details` is an array on the assistant message (or on each streaming delta). The spec defines four entry types. The guide describes three.**

- Quote (guide): "**Non-streaming responses**: `reasoning_details` appears in `choices[].message.reasoning_details`" and "**Streaming responses**: `reasoning_details` appears in `choices[].delta.reasoning_details` for each chunk"
- Quote (guide): "Each object in the array represents a specific piece of reasoning information and follows one of three possible types."
- Quote (spec, `ReasoningDetailUnion.discriminator.mapping`): `"reasoning.encrypted"`, `"reasoning.server_tool_call"`, `"reasoning.summary"`, `"reasoning.text"`.
- Quote (spec, `ChatReasoningDetails`): "Reasoning details for extended thinking models".
- Sources: R-s1, R-s15. Basis: OR-documented.

**R33. Fields of each entry type, from the spec.**

- `reasoning.text` (`ReasoningDetailText`): `type` (required), `text` (string or null), `signature` (string or null), `id` (string or null), `format`, `index` (integer). Guide: "Contains raw text reasoning with optional signature verification".
- `reasoning.summary` (`ReasoningDetailSummary`): `type` and `summary` (both required, `summary` is a string), `id`, `format`, `index`. Guide: "Contains a high-level summary of the reasoning process".
- `reasoning.encrypted` (`ReasoningDetailEncrypted`): `type` and `data` (both required, `data` is a string), `id`, `format`, `index`. Guide: "Contains encrypted reasoning data that may be redacted or protected".
- `reasoning.server_tool_call` (`ReasoningDetailServerToolCall`): `type`, `tool_name`, `arguments`, `result` (all required strings), `tool_call_id` (string or null), `id`, `format`, `index`. Spec description: "Record of an OpenRouter server-tool invocation (e.g. openrouter:fusion), carried in reasoning_details so a prior tool call can be rehydrated into a later turn of the same conversation."
- Source: R-s15, R-s1. Basis: OR-documented.

**R34. The complete `format` enum has twelve values.**

- Quote (spec, `ReasoningFormat`): `["unknown", "openai-responses-v1", "azure-openai-responses-v1", "bedrock-openai-responses-v1", "bedrock-xai-responses-v1", "xai-responses-v1", "meta-responses-v1", "google-vertex-meta-responses-v1", "google-vertex-xai-responses-v1", "openrouter-stealth-v1", "anthropic-claude-v1", "google-gemini-v1", null]`, with `"x-speakeasy-unknown-values": "allow"`.
- The guide lists the same twelve with glosses, including "`\"anthropic-claude-v1\"` - Anthropic Claude format version 1 (default)", "`\"unknown\"` - Format is not specified", "`\"openrouter-stealth-v1\"` - OpenRouter stealth model format version 1", and "`\"google-vertex-meta-responses-v1\"` - Meta on Google Vertex responses format version 1 (requests may still send the retired `\"google-vertex-responses-v1\"`, which maps to this value)".
- The value names a wire family plus the serving cloud, not a model: OpenAI reasoning has three values (direct, Azure, Bedrock), xAI has three (direct, Bedrock, Vertex), Meta has two (direct, Vertex).
- The enum grows often. The changelog records "`ReasoningFormat`: enum property\_added" on July 27, August 20, November 3, November 10, and November 18, 2026.
- A value was renamed on November 18, 2026, and old stored values are still accepted: "Requests that replay stored `reasoning_details` with the old value keep working: the API accepts `google-vertex-responses-v1` on input and treats it as `google-vertex-meta-responses-v1`. Update any client code or generated SDK enum that compares against the old string, and accept both values while stored reasoning from before this release is still being replayed."
- Sources: R-s15, R-s1, R-s17. Basis: OR-documented.

**R35. `id` and `index`.**

- Quote (guide): "`id` (string | null): Unique identifier for the reasoning detail" and "`index` (number, optional): Sequential index of the reasoning detail".
- The guide's examples use ids such as `"reasoning-text-1"`. Real OpenAI entries use the upstream reasoning item id: OpenRouter's Vercel provider test fixture has `data: 'encrypted-data-1', id: 'rs_0ad20f1f8629dc53016924443203408193abb5b3d0b4301e26'` (R-s27, `src/chat/convert-to-openrouter-chat-messages.test.ts` lines 1820 to 1821). Gemini 4 continuation entries use the fixed id `"gemini-continuation"` (R40).
- Sources: R-s1, R-s27, R-s14. Basis: OR-documented.

**R36. An Anthropic thinking block arrives as a `reasoning.text` entry, and its signature is the entry's `signature` field.**

- Quote (guide example): `{"type": "reasoning.text", "text": "Let me think through this step by step:\n1. First, I need to understand the user's question...", "signature": "sha256:abc123def456...", "id": "reasoning-text-1", "format": "anthropic-claude-v1", "index": 2}`. The signature string in this example is a placeholder.
- Quote (OpenRouter Vercel provider, `src/chat/convert-to-openrouter-chat-messages.ts` lines 322 to 325): "Anthropic (issue #423): Uses an explicit `signature` field on reasoning.text entries. When the signature is lost during message serialization, custom pruning, or DB storage, sending the entry back causes \"Invalid signature in thinking block\"."
- With summarized display the `text` is Anthropic's summary and the signature still holds the full reasoning (vendor rule in R75).
- Sources: R-s1, R-s27. Basis: OR-documented.

**R37. Anthropic redacted thinking and OpenAI encrypted reasoning both arrive as `reasoning.encrypted` entries, with the opaque blob in `data`.**

- Quote (broadcast guide): "`has_encrypted_reasoning` is `true` when the response included encrypted reasoning (`reasoning_details` entries of type `reasoning.encrypted`, such as OpenAI's encrypted reasoning content or Anthropic's redacted thinking). Encrypted reasoning is only readable by the provider, so the response's encrypted reasoning is never exported."
- Quote (guide example): `{"type": "reasoning.encrypted", "data": "eyJlbmNyeXB0ZWQiOiJ0cnVlIiwiY29udGVudCI6IltSRURBQ1RFRF0ifQ==", "id": "reasoning-encrypted-1", "format": "anthropic-claude-v1", "index": 1}`
- For OpenAI the visible part is a `reasoning.summary` entry and the opaque part is a `reasoning.encrypted` entry with the `rs_` id (R35). The guide says: "Use `reasoning_details` when working with models that return special reasoning types (such as encrypted or summarized) - this preserves the full structure needed for those models."
- Sources: R-s19, R-s1. Basis: OR-documented.

**R38. A Gemini thought signature arrives as a `reasoning.encrypted` entry with `format: "google-gemini-v1"` in the message-level array. The tool call object stays in the plain OpenAI shape.**

- Quote (OpenRouter Vercel provider end-to-end test, `e2e/gemini/reasoning-multiturn.test.ts` lines 4 to 10): "This test verifies that when using Gemini models with thinking enabled, the reasoning_details (thoughtSignature) are properly preserved in tool calls and can be sent back in multi-turn conversations. The providerOptions containing reasoning_details must be preserved when passing response.messages from a first request to a second request, otherwise Gemini will reject the continuation."
- Same test, lines 99 to 101: `expect(firstReasoningDetail?.type).toBe('reasoning.encrypted');`, `expect(firstReasoningDetail?.data).toBeDefined();`, `expect(firstReasoningDetail?.format).toBe('google-gemini-v1');`
- Spec: `ChatToolCall` has exactly three properties, `function` (`name`, `arguments`), `id`, `type`. The signature is not a tool call property on this API.
- Gemini's visible thought text arrives as `reasoning.text` entries with `format: "google-gemini-v1"` and without a `signature` value: "Google Gemini (issue #418): Does NOT use the SDK `signature` field, but Google internally signs thought tokens." (R-s27, `src/chat/convert-to-openrouter-chat-messages.ts` lines 327 to 328).
- On the Responses route the same signature is a `reasoning` output item placed after the `message` item, with the blob in `encrypted_content` and `"format": "google-gemini-v1"` (doc example for `google/gemini-3-flash-preview`, R-s22).
- How OpenRouter ties one encrypted entry back to the specific `functionCall` part it came from is undocumented (see Open questions, Q3).
- Sources: R-s27, R-s15, R-s22. Basis: OR-documented for the shape; the link back to a part is an open question.

**R39. Gemini 3 with tool calls ends with `finish_reason: "stop"`, and OpenRouter's own client rewrites it when it sees encrypted reasoning next to tool calls.**

- Quote (`src/chat/index.ts` lines 549 to 558): "Fix for Gemini 3 thoughtSignature: when there are tool calls with encrypted reasoning (thoughtSignature), the model returns 'stop' but expects continuation. Override to 'tool-calls' so the SDK knows to continue the conversation." followed by `const shouldOverrideFinishReason = hasToolCalls && hasEncryptedReasoning && choice.finish_reason === 'stop';`
- Source: R-s27. Basis: OR-documented (official SDK source). Confidence: medium that this still happens on current models; the code is still present at the pinned commit of 2026-11-20.

**R40. Gemini 4's long-decode continuation token is also a `reasoning.encrypted` entry, with the fixed id `gemini-continuation`.**

- Quote: "On OpenRouter, the continuation token is an encrypted reasoning item. Clients that already pass reasoning back between turns get long decode with no new fields."
- Quote (response shape): `{"type": "reasoning.encrypted", "id": "gemini-continuation", "format": "google-gemini-v1", "data": "<opaque continuation token>", "index": 0}` with `"finish_reason": "length"` and `"native_finish_reason": "CONTINUATION"`.
- Quote: "Treat the token as opaque and pass it back byte for byte."
- Quote: "**Token size grows with the decode** and can reach several MB for a long answer. Your client must accept and resend bodies of that size on later hops."
- Source: R-s14. Basis: OR-documented.

**R41. OpenRouter writes the origin of an encrypted blob into the blob itself: the provider's ciphertext, a dot, then base64url JSON naming the model snapshot and provider.**

- Decoded from OpenRouter's own documentation examples (Responses route `encrypted_content` values): `{"endpoint_slug":"openai/gpt-5.2-20251211|openai"}` (API overview page), `{"endpoint_slug":"openai/gpt-5-mini-2025-08-07|openai"}` (Responses basic usage page), `{"endpoint_slug":"openai/o3-2025-04-16|openai"}` (Responses web search page, five examples), `{"endpoint_slug":"openai/gpt-5.1-codex-mini-20251113|azure"}` (apply patch page), `{"endpoint_slug":"google/gemini-3-flash-preview-20251217|google-vertex/global"}` (image understanding page).
- Each example value has the form `<provider ciphertext>.<base64url JSON>`. The OpenAI ciphertexts start with `gAAAAAB`.
- Quote (Gemini 4 guide, Messages route): "The `data` value wraps the Gemini token in an OpenRouter envelope so its format survives the round trip."
- Server-tool state uses a different prefix: an `encrypted_content` of `openrouter-server-tool.` followed by base64url JSON that decodes to `{"server_tool_round":{"v":1,"members":[{"item_id":"msg_tmp_lk25kz4h0w"},{"item_id":"st_tmp_2foofw1c4sf","call_id":"toolu_01NiP9rEbn67fjNzu8Smb3QG"}]}}` (shell server tool page).
- Sources: R-s22, R-s14. Basis: OR-inferred, from decoding the example values printed in OpenRouter's documentation. The envelope is described in prose only for the Gemini case on the Messages route. Confidence: high that the suffix exists on the Responses route today; medium that the same suffix is inside Chat Completions `reasoning.encrypted.data`, because every decoded example is from the Responses route.

**R42. OpenRouter also stores its own state in encrypted reasoning entries, so a client that drops them breaks OpenRouter features, not only vendor reasoning.**

- Deferred tools, quote: "OpenRouter carries each turn's deferred-tool changes inside the encrypted reasoning it returns: a `reasoning_details` entry on Chat Completions, a reasoning item's `encrypted_content` on Responses, and a `redacted_thinking` block on Messages. Each turn records only what changed, such as the ID of a tool it loaded. There is no separate state field."
- Quote: "**Send assistant messages back exactly as returned**, including the encrypted reasoning described above. Each turn's state is chained to the turns before it, so editing, removing, reordering, trimming, or compacting earlier messages breaks the chain."
- Error codes, quoted: `deferred_tools_state_missing` ("An assistant message with tool calls carries no deferred-tool state, usually because the client did not send the encrypted reasoning back, or carries state altered until it is no longer recognisable."), `deferred_tools_state_invalid` ("The deferred-tool state fails verification: it was modified, or was issued for a different workspace."), `deferred_tools_history_changed`, `deferred_tools_catalog_changed`. All are 400s with the code in `error.metadata.code`.
- Video, quote: "the model's internal video navigation steps are returned as encrypted reasoning rather than as tool calls" and "send the assistant message back with its `reasoning_details` intact, as with other [reasoning token](https://openrouter.ai/docs/guides/best-practices/reasoning-tokens.md) round-tripping."
- Server tools: the `reasoning.server_tool_call` entry type (R33).
- Sources: R-s18, R-s20, R-s15. Basis: OR-documented.

**R43. Encrypted reasoning can show up as the literal text `[REDACTED]`.**

- Quote (guide): "For encrypted reasoning, the content may appear as `[REDACTED]` in streaming responses"
- OpenRouter's Vercel provider had to stop showing it: "Reported error: Gemini reasoning models emit \"[REDACTED]\" as visible reasoning text for encrypted reasoning details (reasoning.encrypted)." (`e2e/issues/issue-438-gemini-reasoning-redacted.test.ts` lines 5 to 6).
- Sources: R-s1, R-s27. Basis: OR-documented.

**R44. OpenRouter normalizes three upstream placements of reasoning text into one field.**

- Quote (spec, `PrivateEndpointReasoningReturnMechanism`): "Where the upstream puts reasoning tokens in its response: `reasoning` and `reasoning-content` are response fields, `content-string` is inline in the message, `none` returns no reasoning. `null` uses the provider default." Enum: `content-string`, `reasoning-content`, `reasoning`, `none`.
- This is configuration for a caller's private endpoint, and it shows the translator's three input shapes: a `reasoning` field, a `reasoning_content` field, and think-tags inside `content`.
- Source: R-s15. Basis: OR-documented.

### D. Streaming

**R45. The documented streaming rule for Chat Completions is four bullets. The guide has no Chat Completions accumulation code today.**

- Quote: "Each reasoning detail chunk is sent as it becomes available"
- Quote: "The `reasoning_details` array in each chunk may contain one or more reasoning objects"
- Quote: "The complete reasoning sequence is built by concatenating all chunks in order"
- Quote (streaming example chunk): `"delta": {"reasoning_details": [{"type": "reasoning.text", "text": "Let me think about this step by step...", "signature": null, "id": "reasoning-text-1", "format": "anthropic-claude-v1", "index": 0}]}`
- The guide's streaming code samples were moved to the Responses route and print `response.reasoning_text.delta` and `response.reasoning_summary_text.delta` events. The only Chat Completions replay code on the page is non-streaming (R52). A search of all 559 mirrored pages for `delta.reasoning` or code that merges `reasoning_details` found nothing else.
- The guide does not say whether to merge by `index`, by `id`, or by order beyond "concatenating all chunks in order".
- Source: R-s1. Basis: OR-documented.

**R46. OpenRouter's own Vercel provider merges stream deltas by entry type and order, and says in a comment that `index` cannot be used.**

- Quote (`src/chat/index.ts` lines 891 to 899): "Accumulate reasoning_details to preserve for multi-turn conversations. Merge consecutive same-type text/summary deltas into a single entry. Providers (e.g. OpenAI `openai-responses-v1`) stream a logical reasoning block as many small deltas that all carry the same `index` (frequently `index: 0`), so `index` cannot be used to reassemble the array"; "reassembly is driven by the `type` transition instead. Without merging summary deltas the array fragments into one entry per delta, which breaks multi-turn round-tripping when re-submitted (see issue #519, langchain #36400)."
- Code, lines 906 to 948:

```ts
if (detail.type === ReasoningDetailType.Text) {
  // A signature closes the text detail it signs: once the last
  // text detail is signed, the next text delta starts a new
  // block (e.g. multiple signed Anthropic thinking blocks).
  if (
    lastDetail?.type === ReasoningDetailType.Text &&
    !lastDetail.signature
  ) {
    // Merge with the previous text detail
    lastDetail.text =
      (lastDetail.text || '') + (detail.text || '');

    // Only copy defined values: assigning `undefined` keys
    // breaks AI SDK prompt validation on replay (#540).
    if (!lastDetail.signature && detail.signature) {
      lastDetail.signature = detail.signature;
    }

    if (!lastDetail.format && detail.format) {
      lastDetail.format = detail.format;
    }
  } else {
    // Start a new text detail
    accumulatedReasoningDetails.push({ ...detail });
  }
} else if (detail.type === ReasoningDetailType.Summary) {
  if (lastDetail?.type === ReasoningDetailType.Summary) {
    // Merge consecutive summary deltas by concatenating text
    lastDetail.summary =
      (lastDetail.summary || '') + (detail.summary || '');

    if (!lastDetail.format && detail.format) {
      lastDetail.format = detail.format;
    }
  } else {
    // Start a new summary detail
    accumulatedReasoningDetails.push({ ...detail });
  }
} else {
  // Encrypted details are opaque blobs (each has its own id);
  // they are discrete and must be pushed as-is, never merged.
  accumulatedReasoningDetails.push(detail);
}
```

- So the working rule is: text deltas append to the last text entry until that entry has a signature; summary deltas append to the last summary entry; encrypted entries are pushed whole and never merged.
- Source: R-s27. Basis: OR-documented (official SDK source at commit `7a235a4c`).

**R47. The Anthropic signature arrives at the end, sometimes as a delta with no text, and sometimes after answer text has started.**

- Quote (`src/chat/index.ts` lines 998 to 1001): "Always include accumulated reasoning_details so the AI SDK can update the reasoning part's providerMetadata with the correct signature.  The signature typically arrives in the last delta, but reasoning-start only carries the first delta's metadata."
- Quote (lines 961 to 963): "Emit even when detail.text is empty/undefined"; "a signature-only delta (no text, just signature) must still be emitted so that the signature propagates to the reasoning part's providerMetadata."
- Quote (lines 951 to 956): "Late-arriving reasoning_details (e.g. a signature-only delta after text has begun) are still accumulated above for multi-turn roundtrip via the finish event's providerMetadata, but must not start a new reasoning block"
- Deciding fact: a client must keep collecting `delta.reasoning_details` until the stream ends, not until the first `delta.content`.
- Source: R-s27. Basis: OR-documented (official SDK source).

**R48. A second client (pi) uses the same merge rule, which supports reading it as the intended contract.**

- Quote (`packages/ai/src/api/openai-completions.ts` lines 619 to 621): "Keep provider replay data in the existing signature slot. OpenRouter streams reasoning_details as deltas: consecutive text/summary deltas are merged into logical entries, while encrypted entries remain opaque and discrete."
- Quote (lines 339 to 340): "`reasoning_details` are replay metadata, not user-visible stream deltas. Keep them in memory during streaming and serialize once when the block is finalized."
- pi's merge differs in one detail: it appends a text delta to the last text entry whether or not that entry already has a signature (`lastDetail.signature ||= detail.signature;`, in `appendOpenAIReasoningDetail`, lines 265 to 279), where OpenRouter's provider starts a new entry after a signature.
- Source: R-s30. Basis: Filled (pi, `badlogic/pi-mono` at commit `9c465268`).

**R49. Gemini 4 streams can deliver several continuation entries, and only streaming delivers the checkpoints.**

- Quote: "When streaming, continuation items arrive in `delta.reasoning_details` chunks during the stream, as Gemini saves checkpoints, and at the end of a hop that stops at a limit. A non-streaming response carries only the hop's final item."
- Quote: "OpenRouter resumes from the last continuation item in the turn, so you can pass the stopped turn back unchanged. Keeping only the latest item makes the request smaller."
- Source: R-s14. Basis: OR-documented.

**R50. Streaming on the other two routes uses each route's native events.**

- Responses route event types in the spec: `response.reasoning_text.delta` (`delta`, `item_id`, `output_index`, `content_index`), `response.reasoning_text.done`, `response.reasoning_summary_part.added`, `response.reasoning_summary_text.delta` (`summary_index`), `response.reasoning_summary_text.done`, `response.reasoning_summary_part.done`. The tool calling guide adds: "Preserve every completed item in order. Reasoning and assistant items are required context for some providers when sending tool results." (on `response.output_item.done`).
- Messages route delta types in the spec (`MessagesContentBlockDeltaEvent`): `text_delta`, `input_json_delta`, `thinking_delta` (`thinking`), `signature_delta` (`signature`), `citations_delta`, `compaction_delta`.
- Sources: R-s15, R-s25. Basis: OR-documented.

### E. Replay rules on Chat Completions

**R51. The caller may send reasoning back in one of two ways: the `reasoning` string or the whole `reasoning_details` array. The array is the one that works for signed, encrypted, or summarized reasoning.**

- Quote: "To preserve reasoning context across multiple turns, you can pass it back to the API in one of two ways:" then "**`message.reasoning`** (string): Pass the plaintext reasoning as a string field on the assistant message" and "**`message.reasoning_details`** (array): Pass the full reasoning\_details block"
- Quote: "Use `reasoning_details` when working with models that return special reasoning types (such as encrypted or summarized) - this preserves the full structure needed for those models."
- Quote: "For models that only return raw reasoning strings, you can use the simpler `reasoning` field."
- Source: R-s1. Basis: OR-documented.

**R52. The array must go back whole, in the original order, unmodified, on the assistant message it came from. The documented way to do that is to append the returned message object as it is.**

- Quote (note titled "Important for Reasoning Models"): "When providing reasoning\_details blocks, the entire sequence of consecutive reasoning blocks must match the outputs generated by the model during the original request; you cannot rearrange or modify the sequence of these blocks."
- Quote (Go example): `messages = append(messages, msg) // includes reasoning_details as-is`
- Quote (cURL example comment): "Send the assistant turn back unmodified (including reasoning_details) plus the tool results, so Claude continues reasoning from where it left off."
- Quote (same example): "Note: You can use '~openai/gpt-sol-latest' instead of '~anthropic/claude-sonnet-latest' - they're completely interchangeable"
- Quote (TypeScript SDK example, `examples/chat-reasoning.example.ts`): `const assistant1 = result1.choices[0].message;` then "// Add assistant response and next user message to history" and `messages.push(assistant1);`
- Tool results follow as ordinary `role: "tool"` messages with `tool_call_id`. The array is a property of the assistant message, next to `content` and `tool_calls`.
- Sources: R-s1, R-s28. Basis: OR-documented.

**R53. `reasoning_content` is accepted as an input alias of `reasoning`. Responses use `reasoning`.**

- Quote: "You can also use `reasoning_content` as an alias - it functions identically to `reasoning`."
- This sentence sits in the section on what the caller passes back, so it documents the request direction. The response schemas (`ChatAssistantMessage`, `ChatStreamDelta`) name the field `reasoning` (R31). OpenRouter's Vercel provider reads `reasoning` and `reasoning_details` and has no code path for `reasoning_content` (R-s27, `src/chat/schemas.ts` lines 54 to 55 and 180 to 181).
- Sources: R-s1, R-s15, R-s27. Basis: OR-documented for input. Confidence: medium that responses never use the alias, because that rests on the schema and SDK reading only `reasoning`.

**R54. The guide frames replay as useful for tool calling and does not state a general "required" rule. Hard failures are documented for specific features.**

- Quote: "Preserving reasoning blocks is useful specifically for tool calling. When models like Claude invoke tools, it is pausing its construction of a response to await external information. When tool results are returned, the model will continue building that existing response. This necessitates preserving reasoning blocks during tool use"
- Quote: "**Reasoning continuity**: The reasoning blocks capture the model's step-by-step reasoning that led to tool requests." and "**Context maintenance**: While tool results appear as user messages in the API structure, they're part of a continuous reasoning flow."
- For vendor specifics the guide points away: "For more detailed information about thinking encryption, redacted blocks, and advanced use cases, see [Anthropic's documentation on extended thinking]" and "see [OpenAI's reasoning documentation]".
- Documented hard failures when the entries are missing or changed: deferred tools (400 `deferred_tools_state_missing`, R42), Gemini 4 continuation (the decode cannot resume, R40), prefix-bound Claude thinking after a history edit (upstream 400, R62), GPT-5.4 family `phase` (quality loss, R65).
- Quote (tool calling guide, Responses route): "Reasoning and assistant items are required context for some providers when sending tool results."
- Sources: R-s1, R-s18, R-s14, R-s25. Basis: OR-documented.

**R55. What OpenRouter's own Vercel provider stores and sends: one message-level array, taken with a fixed priority.**

- On receipt (non-streaming) it attaches the full array to the response metadata and to the first tool call only: "Only attach reasoning_details to the first tool call to avoid duplicating thinking blocks for parallel tool calls (Claude)" (`src/chat/index.ts` lines 467 to 468), and `reasoning_details: choice.message.reasoning_details ?? [],` in `providerMetadata.openrouter` (line 579).
- On send it picks one array per assistant message. Three comment lines (`src/chat/convert-to-openrouter-chat-messages.ts` lines 304 to 306): "Use message-level reasoning_details if available, otherwise find from parts", "Priority: message-level > first tool call > first reasoning part", "This prevents duplicate thinking blocks when Claude makes parallel tool calls".
- The outgoing assistant message (lines 407 to 417): `role: 'assistant'`, `content`, `tool_calls`, `reasoning: effectiveReasoning`, `reasoning_details: finalReasoningDetails`, `annotations`, `cache_control`.
- Source: R-s27. Basis: OR-documented (official SDK source).

**R56. The Vercel provider drops `reasoning.text` entries that lack a signature when the format is Anthropic or Gemini, and keeps everything else.**

- Code (lines 344 to 356):

```ts
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

- Quote (lines 327 to 337): "Google Gemini (issue #418): Does NOT use the SDK `signature` field, but Google internally signs thought tokens. When reasoning text is modified during roundtripping (serialization, encoding changes, field reordering), Google rejects with \"Corrupted thought signature\". Since the SDK cannot verify whether the text is intact, Gemini reasoning.text entries are always stripped on roundtrip"; "the encrypted reasoning blob (reasoning.encrypted) handles multi-turn continuity instead. Other formats (OpenAI, xAI, Azure) and non-text detail types pass through unchanged."
- A missing `format` is treated as Anthropic: "Anthropic Claude was the first reasoning that we're passing back and forth" and `export const DEFAULT_REASONING_FORMAT = ReasoningFormat.AnthropicClaudeV1;` (`src/schemas/format.ts` lines 10 to 12).
- So "send the array back unmodified" is not what OpenRouter's own client does for Gemini: it removes the visible thought text and sends back only the encrypted entries.
- Source: R-s27. Basis: OR-documented (official SDK source).

**R57. A plain `reasoning` string sent to Claude without matching `reasoning_details` makes OpenRouter build thinking blocks that Anthropic rejects.**

- Quote (lines 389 to 397): "Only include reasoning text if we have valid, non-empty reasoning_details. When providerMetadata is lost during message serialization or custom pruning (e.g., stripping providerOptions from reasoning parts), or when switching between models mid-conversation, reasoning text may exist without corresponding reasoning_details. Sending reasoning without reasoning_details causes the API to construct thinking blocks without valid signatures, which Anthropic rejects with \"Invalid signature in thinking block\" (issue #423)."
- Code (lines 402 to 405): `const effectiveReasoning = reasoning && finalReasoningDetails && finalReasoningDetails.length > 0 ? reasoning : undefined;`
- This is the one place where OpenRouter's own source states what the closed translator does with the `reasoning` string: it turns it into provider thinking content. The prompt injection guide agrees: "The `prediction` field and the message-level `reasoning` and `reasoning_details` fields are forwarded unscanned, including reasoning text that the router replays to the provider as thinking or reasoning content".
- Sources: R-s27, R-s23. Basis: OR-documented (official SDK source comment and guardrail guide). Confidence: medium on the exact upstream shape, since the comment reports an observed error and does not show the upstream request.

**R58. The API rejects the same reasoning entry appearing twice in one request, and the Vercel provider de-duplicates with a key rule it says is copied from the server.**

- Quote (`src/chat/convert-to-openrouter-chat-messages.ts` lines 69 to 72): "Track reasoning_details across all messages in this conversion to prevent duplicates. This fixes issue #254 where the same reasoning ID appears in multiple assistant messages during multi-turn conversations, causing the API to reject the request with \"Duplicate item found with id\" error."
- Quote (`src/utils/reasoning-details-duplicate-tracker.ts` lines 13 to 17): "The canonical key logic matches the OpenRouter API's deduplication exactly (see openrouter-web/packages/llm-interfaces/reasonings/duplicate-tracker.ts): - Summary: key = summary field - Encrypted: key = id field (if truthy) or data field - Text: key = text field (if truthy) or signature field (if truthy)"
- Sources: R-s27. Basis: OR-inferred, from official client code that names a file in OpenRouter's closed server repository. Confidence: medium. It shows the server has a duplicate tracker with these keys; what the server does on a duplicate today (drop or reject) is known only from the quoted error string.

**R59. For DeepSeek V4 in thinking mode, even an empty `reasoning_details: []` must go back, or the next turn fails.**

- Quote (provider changelog, 2.8.1 patch notes, `CHANGELOG.md` lines 99 to 102): "Some providers (notably DeepSeek V4 in thinking mode) return `reasoning_details: []` on turns where they produced no visible reasoning tokens. They require this empty array to be sent back in subsequent requests to maintain conversation state; omitting it causes 4xx errors on follow-up turns."
- Quote (`src/chat/convert-to-openrouter-chat-messages.ts` lines 399 to 401): "an empty finalReasoningDetails ([]) means the provider explicitly returned no reasoning tokens this turn"; "do not send reasoning text in that case either."
- DeepSeek's native rule is that `reasoning_content` must be passed back once `tools` is present (R80). The inference is that OpenRouter builds the upstream `reasoning_content` from the caller's `reasoning_details` or `reasoning`, and treats a missing array as a missing field.
- Sources: R-s27. Basis: OR-inferred from official SDK source. Confidence: medium. OpenRouter's documentation pages do not mention DeepSeek's or Kimi's replay requirement anywhere.

**R60. OpenRouter's own Vercel provider knows six of the twelve formats and three of the four entry types, and silently drops entries it cannot parse.**

- Quote (`src/schemas/format.ts` lines 1 to 8): `Unknown = 'unknown'`, `OpenAIResponsesV1 = 'openai-responses-v1'`, `AzureOpenAIResponsesV1 = 'azure-openai-responses-v1'`, `XAIResponsesV1 = 'xai-responses-v1'`, `AnthropicClaudeV1 = 'anthropic-claude-v1'`, `GoogleGeminiV1 = 'google-gemini-v1'`.
- Quote (`src/schemas/reasoning-details.ts` lines 5 to 9): the type enum has `Summary`, `Encrypted`, `Text`. Lines 58 to 65: `z.unknown().transform(() => null)` followed by `.transform((d) => d.filter((d): d is ReasoningDetailUnion => !!d))`.
- Quote (`src/schemas/provider-metadata.ts` lines 83 to 86): "Use ReasoningDetailArraySchema (with unknown fallback) instead of z.array(ReasoningDetailUnionSchema) so that a single malformed entry (e.g., a future format not yet in the enum) is individually dropped rather than causing the entire array to fail parsing."
- Consequence at this commit: an entry with `format` of `bedrock-openai-responses-v1`, `bedrock-xai-responses-v1`, `meta-responses-v1`, `google-vertex-meta-responses-v1`, `google-vertex-xai-responses-v1`, or `openrouter-stealth-v1`, or with `type` of `reasoning.server_tool_call`, fails the schema and is filtered out before the next request. The generated TypeScript SDK (`@openrouter/sdk` 1.5.128) has all twelve values (`src/models/reasoningformat.ts`).
- Sources: R-s27, R-s28. Basis: OR-documented (official SDK source). Confidence: high on the code path; the effect on a live conversation was not run.

**R61. Interleaved thinking: the chat shape gives one `reasoning_details` array, one `content`, and one `tool_calls` list per assistant message, so block order inside a single assistant message is not expressed.**

- Quote (tool calling guide): "Interleaved thinking allows models to reason between tool calls, enabling more sophisticated decision-making after receiving tool results."
- Quote (provider selection guide): "Interleaved Thinking | `interleaved-thinking-2025-05-14` | Allows Claude's thinking/reasoning to be interleaved with regular output, rather than appearing as a single block"
- Thinking that happens between tool calls lands on separate assistant messages (one per model call in the tool loop), each with its own array, so that order is kept by message order.
- Inside one assistant message, Anthropic can return `thinking`, `text`, `thinking`, `tool_use` in that order, and from Fable 5.1 and Sonnet 5.5 a separate signed progress-update thinking block sits "immediately before the `tool_use` or `server_tool_use` block it introduces" (R-s31). The Chat Completions message has three separate properties for these, and the `index` field is the only positional hint. How OpenRouter rebuilds the native block order from them is undocumented (Open questions, Q4).
- Sources: R-s25, R-s24, R-s15, R-s31. Basis: OR-documented for the shapes; OR-inferred for the loss of order (inferred from the response schema having three separate properties and from R55).

**R62. Prefix-locked (preserved) thinking on newer Claude models: OpenRouter forwards the upstream 400, does not drop blocks for the caller, and the opt-in to drop blocks exists on the Messages route.**

- Quote (Fable 5.1 guide): "Upstream, Fable 5.1 can bind each thinking block to the transcript prefix that produced it: replaying a thinking block after editing earlier history (injected/removed messages, in-place summarization, a changed system prompt) returns a 400 `invalid_request_error` on enforced accounts."
- Quote (both guides): "Whether it is enforced through OpenRouter depends on the provider that serves the request. Some providers already return this 400 for requests through OpenRouter, and the rest can start enforcing it, so do not rely on history edits succeeding. OpenRouter passes the upstream error through and does not drop thinking blocks for you; set `drop_block` yourself if you want that."
- Quote (Claude 5.5 guide): "Preserved thinking ties each thinking block to the conversation that produced it, meaning the system prompt, the tool list, and every message before it. This applies to all three models." and "enforced accounts (Anthropic API accounts created on or after August 31, 2026)".
- Quote (Claude 5.5 guide): "If your harness replays history exactly as received and only appends, nothing changes for you."
- The three edits that break it, quoted: "**Per-turn reminders or mid-session system prompt changes**", "**Adding or removing tools**", "**Compaction that summarizes older turns while replaying newer ones with their thinking blocks**: replace the whole history with one summary message and replay no earlier thinking blocks, or set `drop_block` below so mismatched blocks are dropped instead of erroring."
- The control, quoted (Messages route): `"thinking": {"type": "adaptive", "block_binding": { "prefix_mismatch_behavior": "drop_block" }}`. Spec (`AnthropicThinkingBlockBinding`): `prefix_mismatch_behavior` enum `["error", "drop_block", null]`, plus a deprecated alias `mismatch_behavior`.
- The report, quoted: "With `drop_block`, each removal is reported in the response's `input_transformations`" and "(`model_binding_mismatch` entries after a model switch are expected.)". Spec example (`AnthropicInputTransformation`): `{"path": "messages.1.content.0", "reason": "prefix_binding_mismatch", "type": "thinking_dropped"}`.
- Route coverage in the spec: `input_transformations` is a property of `BaseMessagesResult` and `MessagesStartEvent` (Messages route) and of `OpenResponsesResult` (Responses route, described as "Input items the upstream provider changed before inference (OpenAI `input_transformations`). Present only when the provider reported the field"). The Chat Completions result (`ChatResult`) has eleven properties: `alignment`, `choices`, `created`, `id`, `model`, `object`, `openrouter_metadata`, `safeguard_results`, `service_tier`, `system_fingerprint`, `usage`. The Chat Completions request's `reasoning` object has the seven properties in R1.
- So on Chat Completions a caller gets the upstream 400 on a prefix mismatch, and the `drop_block` switch and the `input_transformations` report are Messages and Responses features.
- Changelog: `AnthropicThinkingBlockBinding` and the `input_transformations` property on `BaseMessagesResult` and `MessagesStartEvent` are listed on September 1, 2026; `mismatch_behavior` on September 9, 2026.
- Sources: R-s9, R-s10, R-s15, R-s17. Basis: OR-documented. The last conclusion about Chat Completions is read from the spec's property lists; confidence high for the spec, medium that no undocumented pass-through exists.

**R63. Account-bound thinking (Sonnet 5.5 and Haiku 5.5): OpenRouter pins routing to a provider that can read the blocks and returns a 400 when the caller's provider preferences exclude it.**

- Quote (Haiku 5.5): "Thinking blocks from Haiku 5.5 are only readable by the provider account that produced them. Replaying one through a different account is not an error upstream: Anthropic drops the block and the model loses that reasoning."
- Quote (Haiku 5.5): "When a request replays Haiku 5.5 thinking blocks, OpenRouter keeps it on the provider that produced them instead of falling back to a different provider. If you pass thinking blocks back unchanged, nothing changes for you. If `provider.only` or `provider.ignore` excludes the provider that produced the blocks, OpenRouter returns a 400 asking you to allow that provider or start a new conversation without the thinking blocks."
- Quote (Sonnet 5.5): "Anthropic, Claude Platform on AWS, and Azure share an account group, so OpenRouter can fall back among them when you replay Sonnet 5.5 thinking blocks. It does not send those replays to Amazon Bedrock or Google Vertex."
- Deciding fact: OpenRouter must know which provider account produced a block. The docs do not say how. The provider tag inside the encrypted blob (R41) is one way it could know (OR-inferred, low confidence that it is the mechanism for Claude).
- Source: R-s10. Basis: OR-documented.

**R64. What the docs say about changing model or provider between turns: specific cases are documented, the general case is left unstated.**

- Same wire family across vendors, code structure only: "The `reasoning_details` functionality works identically across all supported reasoning models. You can easily switch between OpenAI reasoning models (like `~openai/gpt-sol-latest`) and Anthropic reasoning models (like `~anthropic/claude-sonnet-latest`) without changing your code structure." This sentence is about using the same client code for either model. It makes no promise about one conversation that moves from one model to the other.
- Claude account binding: routing is pinned, or a 400 (R63).
- Claude model switch with `drop_block`: "`model_binding_mismatch` entries after a model switch are expected." (R62).
- Gemini 4 continuation: "**Continuations stay on one provider.** A continuation token only works on the provider that issued it. OpenRouter routes a continuation request back to that provider and does not fall back to another one. If that provider is unavailable, the request fails instead of restarting the answer elsewhere."
- Deferred tools: "Keep the same provider, API, and model for the whole conversation: switching models makes pi strip reasoning signatures, and the next turn returns a `400`."
- OpenAI on the Responses route: the result can report "a replayed reasoning item removed because the conversation before it was edited" (`ResponsesInputTransformation`, example `{"action": "removed", "id": "rs_0ca73592c5f68037016afcc009779887", "reason": "prefix_mismatch", "type": "reasoning"}`).
- A renamed format value is still accepted on input (R34).
- No page states what OpenRouter does with an `anthropic-claude-v1` entry when the next turn goes to an OpenAI or Gemini model, or the reverse (Open questions, Q1).
- Sources: R-s1, R-s10, R-s14, R-s18, R-s15. Basis: OR-documented.

**R65. Some OpenAI replay state has no Chat Completions form at all: `phase` on assistant messages.**

- Quote: "`phase` is **not available** in the Chat Completions API. The Chat Completions format cannot represent multiple output items with distinct phases in a single response. Use the Responses API for full `phase` support."
- Quote: "If `phase` metadata is dropped when reconstructing conversation history, significant performance degradation can occur" and "including early stopping on longer-running tasks."
- Models named: `openai/gpt-5.5`, `openai/gpt-5.5-pro`, `openai/gpt-5.4`, `openai/gpt-5.4-pro`, `openai/gpt-5.3-codex`. A GPT-6 Astra response in the GPT-6 guide also shows `"phase": "final_answer"`.
- Source: R-s11, R-s13. Basis: OR-documented.

### F. The same on the other two routes

**R66. Responses route: reasoning is a `reasoning` item in `output`. OpenRouter adds `signature` and `format` to OpenAI's item shape.**

- Spec (`OutputItemReasoning`): `type` (`reasoning`), `id`, `summary` (array of `summary_text`), `content` (array of `reasoning_text`), `encrypted_content` (string or null), `status`. `OutputReasoningItem` and `ReasoningItem` add `signature` ("A signature for the reasoning content, used for verification") and `format` (`ReasoningFormat`). `ReasoningItem` is described as "Reasoning output item with signature and format extensions".
- Spec example: `"format": "anthropic-claude-v1", "id": "reasoning-123", "signature": "EvcBCkgIChABGAIqQKkSDbRuVEQUk9qN1odC098l9SEj..."`.
- Vendor mapping seen in documented examples: Claude gives `content` (`reasoning_text`) plus `signature`; OpenAI gives `summary` plus `encrypted_content` with `"format": "openai-responses-v1"` (or `"azure-openai-responses-v1"` when Azure served it); Gemini gives `encrypted_content` with `"format": "google-gemini-v1"` on an item placed after the message item.
- Sources: R-s15, R-s1, R-s2, R-s22, R-s24. Basis: OR-documented.

**R67. Responses route replay: send every output item back in order, unchanged. The route is stateless, so the caller holds all state.**

- Quote: "To preserve reasoning across turns, send the previous response's `output` items back in the next request's `input`, in the order they were returned, and append your `function_call_output` items after them. Keep each `reasoning` item's `content`, `summary`, `signature`, and `encrypted_content` unchanged"
- Quote: "OpenRouter automatically requests encrypted reasoning content from reasoning-capable providers, so replay the returned `encrypted_content` unchanged; no explicit `include` field is required"
- Spec: `ResponseIncludesEnum` still lists `reasoning.encrypted_content`. `store` is `{"const": false, "default": false}`. `previous_response_id`: "Not supported on this proxy. Each response request is independent: no responses are stored, so a previous response cannot be referenced. Requests with a non-null value are rejected with a 400 error. Send the full conversation history in `input` instead."
- Quote (Gemini 4 guide): "This is the stateless pattern the Responses API already uses for encrypted reasoning. OpenRouter does not store the response, so `previous_response_id` cannot be used to continue."
- Sources: R-s1, R-s15, R-s14. Basis: OR-documented.

**R68. Messages route with Claude: native `thinking` and `redacted_thinking` blocks, with the native request controls.**

- Spec: `AnthropicThinkingBlock` requires `type`, `thinking`, `signature`. `AnthropicRedactedThinkingBlock` requires `type`, `data`. Streaming uses `thinking_delta` and `signature_delta` (R50).
- Spec, request: `thinking` is one of `{type: "enabled", budget_tokens, display, block_binding}`, `{type: "disabled"}`, `{type: "adaptive", display, block_binding}`, `{type: "between_tools"}`. `output_config.effort` enum is `low`, `medium`, `high`, `xhigh`, `max`. `context_management` accepts the edit type `clear_thinking_20251015` with a `keep` property that takes `AnthropicThinkingTurns` (`{"type": "thinking_turns", "value": 3}`) or `all`.
- Usage on this route reports `output_tokens_details.thinking_tokens` (`AnthropicOutputTokensDetails`).
- Quote (Claude Code cookbook): "OpenRouter's \"Anthropic Skin\" behaves exactly like the Anthropic API. It handles model mapping and passes through advanced features like \"Thinking\" blocks and native tool use." and "Claude Code with OpenRouter is only guaranteed to work with the Anthropic first-party provider."
- Sources: R-s15, R-s26. Basis: OR-documented.

**R69. Messages route with a non-Anthropic model: reasoning text becomes a `thinking` block with an empty signature, and encrypted reasoning becomes a `redacted_thinking` block.**

- Documented live output for `openai/gpt-5-mini` on `POST /api/v1/messages` (service tiers guide): the `content` array starts with a block of `"type":"thinking"` whose `thinking` text begins `**Exploring the meaning of life**\n\nThe user asks a classic philosophical question` and which ends with `"signature":""}`, followed by `{"type":"redacted_thinking","data":"gAAAAABq_MTy1Mv4EajDcTycXAwagO8z_cn4pKuwpXQUBZeUyZWS9TVPiKHDzAe4kiApAbnLp8OBGgo-SGh-..."}`. The `data` value starts with the same `gAAAAAB` prefix as OpenAI `encrypted_content` values on the Responses route.
- Quote (Gemini 4 guide): "A hop that stops at a limit returns `stop_reason: \"max_tokens\"` and a `redacted_thinking` content block that carries the token."
- Quote (Gemini 4 guide): "pass the assistant content back unchanged: continuation blocks look like other `redacted_thinking` blocks, such as thought signatures, and OpenRouter resumes from the last continuation block." This sentence also says Gemini thought signatures are `redacted_thinking` blocks on this route.
- Quote (tool search guide): "**pi on the Messages API with a non-Anthropic model** is not supported: pi rewrites that model's unsigned thinking blocks, which removes the carried state."
- Deciding fact: a client that treats an empty `signature` as "unsigned, safe to rewrite or drop" breaks replay for non-Anthropic models on this route.
- Sources: R-s21, R-s14, R-s18. Basis: OR-documented.

**R70. OpenRouter's agent SDKs hide the replay: they echo output items back unchanged on the Responses route.**

- Quote (guide): "The Python and TypeScript tabs use the agent helper `call_model` / `callModel`, which runs the tool loop for you and replays the assistant turn with its reasoning blocks, so you never handle `message.reasoning_details` yourself."
- Code (`packages/agent/src/lib/turn-context.ts` lines 102 to 112): "Convert a response's output into input items for the next request." then `export function outputToInputItems(output: unknown): Array<models.InputsUnion1> {` ... `return items.filter(isInputItem);`
- Sources: R-s1, R-s29. Basis: OR-documented (official SDK source at commit `8a2d96e5`).

### G. Usage and billing

**R71. Reasoning tokens are reported in a per-route usage field.**

- Chat Completions: `usage.completion_tokens_details.reasoning_tokens`, spec description "Tokens used for reasoning", integer or null.
- Responses: `usage.output_tokens_details.reasoning_tokens` (required integer in `OpenAIResponsesUsage`).
- Messages: `usage.output_tokens_details.thinking_tokens` (`AnthropicOutputTokensDetails`).
- Generation record: `native_tokens_reasoning`, "Native reasoning tokens as reported by provider".
- Source: R-s15. Basis: OR-documented.

**R72. Reasoning is billed as output, counts against the output limit on most providers, and is billed even when hidden or when it used the whole budget.**

- Quote: "Reasoning tokens are considered output tokens and charged accordingly. On most providers they also count against the request's `max_output_tokens`"
- Quote: "If the model spends all of it reasoning, the response returns `finish_reason: \"length\"` with an empty `content`, and the reasoning tokens are still billed. With `reasoning.exclude: true` the `reasoning` field is omitted as well, so the response carries no text at all and the only signal left is `finish_reason` plus `usage`."
- Quote: "To detect this case, subtract `usage.completion_tokens_details.reasoning_tokens` from `usage.completion_tokens`. For example, a request with `max_tokens: 300` that returns `completion_tokens: 302` and `reasoning_tokens: 301` produced a single visible token."
- Quote (summarized thinking): "The model consumes the same number of tokens either way, and usage is billed based on the tokens the model actually generates. Because the visible summary is condensed, it may contain fewer tokens than the reasoning token count reported in `usage`."
- Quote (pro mode): "Pro mode bills at the same per-token rates as standard mode, but typically consumes more tokens."
- Quote (Gemini 4): "Every hop is its own generation with its own usage and cost. A continuation hop's prompt tokens include all the reasoning from earlier hops, and each hop bills its full prompt, so input cost grows with every hop."
- Catalog: 36 models have a `pricing.internal_reasoning` value ("Price in USD per internal reasoning token"). For 35 it equals the `completion` price. `perplexity/sonar-deep-research` is the one model where it differs (`completion` 0.000008, `internal_reasoning` 0.000003).
- Sources: R-s1, R-s14, R-s15, R-s16. Basis: OR-documented.

### H. The vendors' own native rules (read live, for comparison with what OpenRouter exposes)

**R73. Anthropic: within a tool-use turn thinking blocks must go back complete and unmodified; across turns it is recommended; a modified block is a 400.**

- Quote: "Pass every `thinking` block back to the API complete and unmodified, alongside the `tool_use` block it accompanied."
- Quote: "**Required:** within a tool-use turn, pass thinking blocks back." / "**Recommended:** across turns, pass everything back." / "**Allowed:** outside tool use, omit prior turns’ thinking, but not a block between two that you keep."
- Quote: "Within the latest assistant message, the sequence of consecutive `thinking` blocks must match what the model generated in the original request: you can’t rearrange, edit, or partially drop them. This includes [`redacted_thinking` blocks]"
- Quote: "Modified thinking blocks are rejected with a 400 error."
- Quote: "You don’t need to prune old thinking yourself. Pass all thinking blocks back in multi-turn conversations, and the API automatically filters them, keeps the blocks needed to preserve the model’s reasoning, and bills input tokens only for the blocks actually shown to Claude."
- Quote: "**Mid-turn conflicts degrade gracefully.** If you toggle thinking mid-turn (for example, between sending a tool call and returning its result), the API doesn’t error. Instead, it silently disables thinking for that request."
- OpenRouter's note in R52 is a near copy of the third quote, applied to `reasoning_details`.
- Source: R-s31. Basis: Vendor-documented (Anthropic).

**R74. Anthropic: redacted thinking is a separate block type with an opaque `data` field and the same replay rule.**

- Quote: "the API might return `redacted_thinking` blocks when portions of Claude’s reasoning are safety-redacted. A `redacted_thinking` block contains encrypted thinking content in a `data` field, with no readable text"
- Quote: "Filtering on `block.type == \"thinking\"` alone silently drops `redacted_thinking` blocks and breaks the multi-turn protocol"
- Source: R-s31. Basis: Vendor-documented (Anthropic).

**R75. Anthropic: the `signature` is an encrypted copy of the full reasoning, the visible text is only a summary, and when streaming the signature is one delta at the end of the block.**

- Quote: "Each thinking block also carries a `signature` field, an encrypted copy of the full reasoning that you pass back unchanged in multi-turn and tool-use conversations"
- Quote: "`\"omitted\"`: thinking blocks are returned with an empty `thinking` field. The `signature` field still carries the encrypted full thinking for multi-turn continuity"; this is "the default on Claude Fable 5.2, Claude Mythos 5.2, Claude Fable 5.1, Claude Mythos 5.1, Claude Fable 5, Claude Mythos 5, Claude Opus 5.5, Claude Opus 5, Claude Sonnet 5.5, Claude Sonnet 5, Claude Opus 4.8, Claude Opus 4.7"
- Quote: "The server decrypts the `signature` to reconstruct the original thinking for prompt construction". "Any text you place in the `thinking` field of a round-tripped omitted block is ignored."
- Quote: "Thinking blocks stream as `thinking_delta` events inside `content_block_delta` events, followed by a single `signature_delta` event just before the block’s `content_block_stop`."
- Quote: "To reassemble complete thinking blocks with their signatures after streaming, use your SDK’s message-accumulation helper"
- Source: R-s31. Basis: Vendor-documented (Anthropic).

**R76. Anthropic preserved thinking: a block is tied to its prefix, to a model family, and for some models to an account. A block the new model or account cannot read is dropped without error, and the caller should keep sending it.**

- Quote: "**What came before it:** `system`, `tools`, and every earlier message. If any of them changes, the block fails the [prefix check], and the request can get a 400 error. So only add to a conversation"
- Quote: "Change `model` and keep sending the whole conversation. A thinking block that the new model or account can’t use never causes an error. The API drops it for that request and doesn’t bill it, and Claude answers without that reasoning."
- Quote: "Back on Claude Fable 5.1, they’re read again, because the API never changes your `messages`. On a switch, reasoning is lost permanently only if your code removes it. So don’t remove thinking, and don’t keep a separate history for each model."
- Quote: "A turn from another provider’s model goes at the end, as `text` and `tool_use` content. Claude’s earlier thinking stays valid."
- Quote: "Thinking blocks from Claude Fable 5.2, Claude Mythos 5.2, Claude Sonnet 5.5, and Claude Haiku 5.5 work only in the account that produced them or a linked one. In any other account, the API drops them, with no error."
- Quote: "[`prefix_mismatch_behavior`] chooses whether a request that fails the prefix check gets a 400 error or the API drops the failing blocks. The response’s [`input_transformations`] lists thinking blocks the API dropped or let through."
- Quote: "When you [remove secrets, personal data, or injected text] from your history, also remove every thinking block after the first place they appear."
- Listed models with preserved thinking: `claude-fable-5-2`, `claude-mythos-5-2`, `claude-fable-5-1`, `claude-mythos-5-1`, `claude-opus-5-5`, `claude-sonnet-5-5`, `claude-haiku-5-5`.
- The model-switch table shows reading is one-directional in places, for example blocks from Claude Sonnet 5.5 are readable by Claude Fable 5.2, Claude Opus 5.5 and Claude Sonnet 5.5, and not by Claude Fable 5.1 or Claude Haiku 5.5.
- Source: R-s32. Basis: Vendor-documented (Anthropic).

**R77. Anthropic effort: per-model defaults and the `between_tools` and `disabled` limits match what OpenRouter's migration guides say.**

- Quote: "Most Claude models default to high effort, spending as many tokens as needed for excellent results; Claude Opus 5.5 and Claude Haiku 5.5 default to medium."
- Quote: "To have the model respond without thinking first, send `thinking: {\"type\": \"between_tools\"}`, not `\"disabled\"`. It works at `low`, `medium`, and `high` effort. At `xhigh` or `max` it returns a 400 error"
- Quote: "Not every model that supports `max` supports `xhigh`."
- Quote (manual mode page): "On Claude Opus 4.5, Claude Sonnet 4.5 (deprecated), and earlier Claude 4 models, add the `interleaved-thinking-2025-05-14` [beta header] to your API request." and "adaptive thinking interleaves automatically, and the Claude API ignores the header on these models."
- Sources: R-s33, R-s43. Basis: Vendor-documented (Anthropic).

**R78. Google: on `generateContent` a signature can sit on any part, including a function call part or the last part of a response. On the Interactions API it sits only on `thought` steps and built-in tool steps. In stateless mode every thought block must be resent exactly.**

- Quote: "In the `generateContent` API, there are no dedicated thought blocks. Because of this, signatures are metadata that can be attached to any part, such as living inside `functionCall` parts or the final part of a response."
- Quote: "In the Interactions API, thoughts are a first-class representation as dedicated `thought` steps. Because of this signatures are limited exclusively to two known locations, `thought` steps, or built-in tool steps (like `google_search_call`/ `google_search_result`). They never appear on user inputs, model outputs, or standard function calls."
- Quote (stateless mode): "You **MUST** always resend all `thought` blocks exactly as they were received from the model." and "You should **NOT** remove or modify thought blocks from the history, as they contain the signatures required for the model to continue its reasoning."
- Quote: "When switching models within a session, you should still resend the previous model's thought blocks. The backend manages compatibility."
- Quote: "A thought block may contain **only a signature with no summary**"
- Quote (streaming): the `thought_signature` delta is "the last delta before `step.stop`".
- Levels per model (table): `gemini-4-argon` "On (medium) | low, medium, high"; `gemini-4-neon` "On (high) | low, medium, high, xhigh"; `gemini-3-flash-preview` "On (high) | minimal, low, medium, high". These match the OpenRouter catalog lists in R27 except that the catalog's `default_effort` for `google/gemini-4-neon` is `medium` where Google says its default is `high`.
- Google's thought-signatures page now covers the Interactions API only ("The Interactions API makes it easier to work with thought signatures because it manages reasoning at the API level."). The exact `generateContent` error string for a missing signature was not on the pages fetched today; the workspace survey of 2026-10-28 records it as `missing_thought_signature` (not re-checked).
- Sources: R-s34, R-s35. Basis: Vendor-documented (Google).

**R79. OpenAI: encrypted reasoning comes back by default in stateless mode, reasoning items should go back with function calls in the original order, and reasoning from another model family is omitted server-side.**

- Quote: "When you create a response in stateless mode, reasoning items in the response's `output` array include an `encrypted_content` property by default. Stateless mode applies when `store` is `false` or when your organization uses Zero Data Retention (ZDR). The API still accepts the legacy `reasoning.encrypted_content` value in `include` for compatibility, but doesn't require it."
- Quote: "When doing function calling with a reasoning model in the Responses API, pass back the reasoning items returned with the function calls along with the tool outputs."
- Quote: "If you manage conversation history yourself, include the original input and all output items from each response in the input of the next request. Keep those items in their original order and append new items."
- Quote: "When you switch model families, the API omits incompatible reasoning from the model's context, even when `reasoning.context` is `all_turns`."
- Quote: "The reasoning items remain opaque, and the API does not return their reasoning text."
- Quote: "Supported values are model-dependent and can include `none`, `minimal`, `low`, `medium`, `high`, `xhigh`, and `max`." and "GPT-6.2 Astra and GPT-6.2 Sol reject requests that set `reasoning.effort` (Responses) or `reasoning_effort` (Chat Completions) to `none`."
- OpenAI's guide documents `configuration_update` only as a Responses input item; that matches OpenRouter calling its Chat Completions form "an OpenRouter extension".
- Source: R-s36. Basis: Vendor-documented (OpenAI).

**R80. DeepSeek: `reasoning_content` must be passed back on every later request once `tools` is present, or the API returns a 400. Without `tools` it is ignored.**

- Quote: "Please note that for requests carrying the `tools` parameter, the `reasoning_content` must be fully passed back to the API in all subsequent requests" and, continuing the same sentence, "even for turns where the model did not perform a tool call. If your code does not correctly pass back `reasoning_content`, the API will return a 400 error."
- Quote: "If the request does not carry the `tools` parameter: `reasoning_content` does not need to be passed back; even if passed to the API, it will be ignored and will not be concatenated into the context."
- Effort mapping table, quoted rows: `minimal` to `low`, `low` to `low`, `medium` to `high`, `high` to `high`, `xhigh` to `high`, `max` to `max`, `ultra` to `max`. Toggle: `{"thinking": {"type": "enabled/disabled"}}` on the OpenAI format; "Thinking mode is enabled by default, with the default effort being `high`".
- Source: R-s37. Basis: Vendor-documented (DeepSeek).

**R81. Kimi K3 and K3.1 Pro: reasoning is always on, the control is top-level `reasoning_effort`, and the whole assistant message including `reasoning_content` must go back.**

- Quote: "`kimi-k3.1-pro`: the flagship thinking model; reasoning and Preserved Thinking are always on, and `reasoning_content` may be returned. Configure its reasoning effort with the top-level `reasoning_effort` request field, which supports `\"low\"` / `\"high\"` / `\"max\"` (default `\"max\"`)."
- Quote: "For K3.1 Pro and K3, multi-turn conversations and tool calls must pass the complete assistant message returned by the API back in `messages` as-is."
- Quote: "`kimi-k3.1-pro` always reasons and does not support the `thinking` parameter."
- Compare the OpenRouter catalog entry for `moonshotai/kimi-k3.1-pro`: `"mandatory": false` with `none` in `supported_efforts` (R28). Moonshot says the model always reasons (see Conflicts, C9).
- Source: R-s38. Basis: Vendor-documented (Moonshot AI).

**R82. xAI: encrypted reasoning exists on the Responses API only, and the vendor says Chat Completions has nowhere to put it.**

- Quote: "On the Responses API, `grok-4.7` returns `reasoning.encrypted_content` on every response, whether or not `include` lists it, together with the encrypted outputs of any server-side tools. Reasoning items in the output carry an `encrypted_content` field; pass them back unchanged in the next request's `input` so the model keeps its reasoning across turns even when you manage conversation history yourself."
- Quote: "Chat Completions is unaffected; it has no field for the ciphertext."
- Quote: "If not specified, `reasoning_effort` defaults to `\"high\"`. Reasoning cannot be disabled." and "`\"xhigh\"` is available on `grok-4.6` and later. On models that do not support it, such as `grok-4.5`, requests with `\"xhigh\"` are treated as `\"high\"`."
- OpenRouter's `xai-responses-v1` format (R34) implies OpenRouter calls xAI's Responses API upstream and returns that ciphertext in `reasoning_details` on its own Chat Completions route.
- Source: R-s39. Basis: Vendor-documented (xAI). The last sentence is OR-inferred from the format name.

### I. One fill from another client, and the local survey claim

**R83. Fill (pi): store the whole `reasoning_details` array as an opaque string next to the turn, replay it only to the same provider, API, and model, and drop thinking when any of the three changes.**

- Quote (`packages/ai/src/api/transform-messages.ts` lines 122 to 132): `const isSameModel = assistantMsg.provider === model.provider && assistantMsg.api === model.api && assistantMsg.model === model.id;` and "Thinking from another model is dropped. Signatures and encrypted reasoning are only valid for the model that produced them, and sending readable thinking as plain text makes the new model treat it as part of the visible reply." and `if (!isSameModel) return [];`
- Quote (`packages/ai/src/api/openai-completions.ts` lines 342 to 345): `block.thinkingSignature = JSON.stringify(streamedReasoningDetails);` and (lines 1378 to 1380) `assistantMsg.reasoning_details = preservedReasoningDetails;`
- pi also fills the DeepSeek-style requirement itself (lines 1381 to 1386): `if (compat.requiresReasoningContentOnAssistantMessages && model.reasoning && assistantMsg.reasoning_content === undefined) { assistantMsg.reasoning_content = ""; }`
- OpenRouter's tool search guide confirms the behavior from the outside: "switching models makes pi strip reasoning signatures".
- Source: R-s30, R-s18. Basis: Filled (pi, `badlogic/pi-mono` at commit `9c465268`). It shows one client's answer to the model-switch question. It does not show what OpenRouter's translator does with foreign entries.

**R84. Local survey claim LC6 holds, with four corrections of detail.**

- LC6: "Reasoning is a structured details array (`reasoning_details`) that must replay unmodified and in order. Each entry has a vendor `format` tag such as `anthropic-claude-v1`. It is a replay contract, not a conversion contract."
- Confirmed live: the array (R32), the order and no-modification rule (R52), the `format` tag (R34). Nothing in OpenRouter's documentation describes converting an entry from one format to another.
- Correction 1: the `format` tag names a wire family plus the serving cloud (twelve values), so "vendor tag" understates it (R34).
- Correction 2: the caller may also replay only the `reasoning` string, or `reasoning_content` as an alias, for models that return plain text (R51, R53). So the contract has a plain-text lane beside the opaque lane.
- Correction 3: "unmodified" is the documented rule, and OpenRouter's own client edits the array before sending it: it drops unsigned Anthropic and Gemini text entries, de-duplicates across messages, and drops entries with formats it does not know (R56, R58, R60).
- Correction 4: the survey's gloss "so reasoning is replayed verbatim or dropped" (survey line 332) has no documentary support for the "dropped" half on the Chat Completions route. Documented outcomes when the serving model or provider changes are a routing pin, a 400, or an upstream drop reported through `input_transformations` on the other two routes (R62, R63, R64).
- Sources: R-s40 and the items cited. Basis: OR-documented for the confirmation; the survey itself is Workspace (2026-10-28), re-checked live today.

**R85. A `reasoning` setting sent to a model that does not reason is accepted and has no effect (one documented example).**

- The Responses reasoning page sends `reasoning: { effort: 'high' }` to `openai/gpt-4o-mini` and prints the result: `"status": "completed"` with a single `message` output item and no `reasoning` item. The catalog entry for `openai/gpt-4o-mini` has `"reasoning": null`.
- This is the request-side counterpart of R17: OpenRouter drops reasoning settings the target cannot use. The exception is a disable request on a mandatory-reasoning model, which is a 400 (R15).
- Sources: R-s2, R-s16. Basis: OR-documented (a printed example, not a stated rule). Confidence: medium that this is general behavior.

## Conflicts and disagreements

### Between OpenRouter's own sources

- **C1. Three entry types or four.** The reasoning guide says each `reasoning_details` object "follows one of three possible types". The spec's `ReasoningDetailUnion` has four, adding `reasoning.server_tool_call`. The spec is the newer and more exact source. A client written from the guide will meet a type it does not know (R32, R33).
- **C2. `verbosity` and `include_reasoning` on Chat Completions.** The parameters page and the catalog's `supported_parameters` (114 and 359 models) say Chat Completions accepts them. The spec's `ChatRequest` schema lists 43 properties and these two are outside it. The Claude 5.5 guide says "Chat Completions `reasoning.effort` (or `verbosity`) and Messages API `output_config.effort` become Anthropic's `output_config.effort`", so the parameters page is right about behavior and the spec is incomplete (R10, R11).
- **C3. `reasoning_effort` values.** The parameters page lists six values (`xhigh`, `high`, `medium`, `low`, `minimal`, `none`). The spec lists seven, adding `max`. The reasoning guide also lists seven for `reasoning.effort`. The spec and guide agree, so the parameters page is behind (R9).
- **C4. Default effort of Claude Opus 5.5.** OpenRouter's Claude 5.5 migration guide says the default is `medium` ("Requests that set no effort are sent without one and run at Anthropic's default of `medium`"). Anthropic's effort page, read today, agrees ("Claude Opus 5.5 and Claude Haiku 5.5 default to medium"). The OpenRouter catalog entry for `anthropic/claude-opus-5.5` says `"default_effort": "high"`. Two sources against one: the catalog value looks wrong. The same kind of mismatch shows for `google/gemini-4-neon` (catalog `medium`, Google's table "On (high)") (R22, R77, R78).
- **C5. Is `include: ["reasoning.encrypted_content"]` needed on the Responses route.** The reasoning guide says "no explicit `include` field is required". The GPT-5.6 migration guide's example still sends `"include": ["reasoning.encrypted_content"]`. OpenAI's own guide says the value is legacy and not required. The reasoning guide is right; the migration example is leftover (R67, R79).
- **C6. What `index` means.** The guide calls it "Sequential index of the reasoning detail". OpenRouter's Vercel provider says providers "stream a logical reasoning block as many small deltas that all carry the same `index` (frequently `index: 0`), so `index` cannot be used to reassemble the array". The client code reflects observed behavior, so treat `index` as unreliable for merging (R35, R46).
- **C7. Effort levels on the Responses reasoning page.** That page's table lists four levels (`minimal`, `low`, `medium`, `high`). The spec and the reasoning guide list seven. The page is older.
- **C8. Which models take a token budget.** The reasoning guide says budgets are supported by "Anthropic reasoning models" and Gemini thinking models. The catalog flags `supports_max_tokens: true` on nine entries only, and among Claude models only on `anthropic/claude-opus-4.6`. Older budget-based Claude entries such as `anthropic/claude-sonnet-4.5` show `{"mandatory": false}` with no effort list and no budget flag. The catalog under-reports budget support, so a client cannot rely on the flag alone (R14, R19).
- **C9. Kimi K3.1 Pro can or cannot stop reasoning.** The catalog entry for `moonshotai/kimi-k3.1-pro` has `"mandatory": false` and `none` in `supported_efforts`. Moonshot's page says "`kimi-k3.1-pro` always reasons". Either OpenRouter maps `none` to something else for this model or the catalog is wrong. Not resolved (R28, R81).
- **C10. The guide's "works identically" claim versus the client code.** The guide says "The `reasoning_details` functionality works identically across all supported reasoning models." OpenRouter's Vercel provider has three vendor-specific branches for it: strip unsigned text for Anthropic and Gemini formats, keep an empty array for DeepSeek, and rewrite the finish reason for Gemini tool calls (R39, R56, R59). The guide statement holds for the shape of the code a caller writes, and fails for the details.
- **C11. "Send it back unmodified" versus the client code.** The documented rule is no modification. OpenRouter's Vercel provider removes entries before sending (R56, R58, R60). Both are true: the server accepts a pruned array in these cases, and the rule is the safe default.
- **C12. Example signature.** The guide's example signature is `"sha256:abc123def456..."`. Real Anthropic signatures are long base64 blobs (the spec example is `"EvcBCkgIChABGAIqQKkSDbRuVEQUk9qN1odC098l9SEj..."`). The guide example is a placeholder and should not be read as a format.
- **C13. Which Claude models accept mid-conversation effort.** The reasoning guide's list is "Claude Fable 5.2, Claude Fable 5.1, Claude Opus 5". The Claude 5.5 guide says per-turn effort changes "work on all three models" (Opus 5.5, Sonnet 5.5, Haiku 5.5) through the Messages route. The guide's list starts with "include", so it is a partial list (R30).

### With the local Runvil documents

- **LC6 (2026-10-28 survey, R-s40).** Holds. Four corrections of detail are in R84. The one that matters most: the survey says reasoning "is replayed verbatim or dropped". OpenRouter's documentation supports "replayed verbatim" and gives no general statement that foreign or stale entries are dropped. Documented outcomes are routing pins and 400s. The survey's wording is an inference, and should be marked as one.
- **Survey section 5.4, OpenAI line.** The survey says OpenAI's `encrypted_content` "arrives populated on reasoning items" by default. OpenAI's guide read today says the same for stateless mode, and OpenRouter's guide says it requests it automatically. Agreement (R67, R79).
- **Survey section 5.4, Anthropic three-way thinking control.** Agreement with OpenRouter's Claude 5.5 guide and Anthropic's effort page (R22, R23, R77).
- **2026-11-17 report on the Anthropic Messages API (cabinet output, R-s41).** It says of `reasoning_details` and LiteLLM's `thinking_blocks`: "the translator still has to guess where they sat among the text and tool blocks". Today's reading agrees: the Chat Completions message has three separate properties and no documented ordering between them (R61). The report's remark that "a client that reads `reasoning_details` also works against OpenRouter" needs one qualifier from today's evidence: a client must also keep entry types and formats it does not recognize, because OpenRouter stores its own state there (R42, R60).
- **2026-09-20 model and tool differences (R-s42).** It says Gemini's OpenAI-compatible layer "exposes signatures through `extra_content`". That is Google's own compatibility endpoint. Through OpenRouter the signature is a `reasoning.encrypted` entry (R38). The two statements describe different servers and both stand. Its DeepSeek and Kimi replay rules match the vendor pages read today (R80, R81).

## Open questions blocked by closed source

- **Q1. What does OpenRouter do with a `reasoning_details` entry whose `format` or origin does not match the model or provider serving the next turn (for example an `anthropic-claude-v1` signed entry sent to a GPT or Gemini model)?** Tried: every page of the documentation mirror (search for format, switch, mismatch, drop), the spec, the changelog, both official SDKs. Found only special cases: account-bound Claude blocks (pin or 400, R63), Gemini continuation (pin, R64), deferred tools (400), and a renamed format accepted on input (R34). The embedded `endpoint_slug` (R41) shows OpenRouter has the information to decide. Fill found: Anthropic's own rule is to drop unreadable blocks without error and to place another provider's turn "as `text` and `tool_use` content" (R76); OpenAI's rule is that it "omits incompatible reasoning" on a family switch (R79); Google says "The backend manages compatibility" (R78); pi drops all thinking when provider, API, or model differs (R83). None of these shows OpenRouter's behavior.
- **Q2. Does OpenRouter build DeepSeek's and Kimi's required `reasoning_content` from what the caller sends back, and from which field?** Tried: reasoning guide, parameters page, spec, changelog, model pages in the catalog. The documentation has no sentence on it. Partial answer from OpenRouter sources: the guardrail guide speaks of "reasoning text that the router replays to the provider as thinking or reasoning content" (R57), and the Vercel provider says DeepSeek V4 needs even an empty `reasoning_details` array returned or the next turn gets a 4xx (R59). So the array (or the `reasoning` string) is the input, and a missing array is a failure. Whether `reasoning` alone is enough for DeepSeek with tools is not shown. Fill found: pi sets `reasoning_content = ""` itself when the target needs the field (R83); DeepSeek's and Moonshot's native rules are in R80 and R81.
- **Q3. How does OpenRouter attach a Gemini signature back to the right part (a specific `functionCall` part, or the final text part) when the caller returns a flat list of `reasoning.encrypted` entries?** Tried: reasoning guide, Gemini 4 guide, spec (`ChatToolCall` has `function`, `id`, `type`), the Vercel provider and its tests. The tests show one encrypted entry per turn with `format: "google-gemini-v1"` and no part reference in the fields the client reads. The Responses route example places the reasoning item after the message item, so order may be the link there. Fill found: Google's native rule that on `generateContent` signatures are "attached to any part, such as living inside `functionCall` parts or the final part of a response" (R78). No fill shows OpenRouter's own mapping; a guess is that the entry's `id` or the envelope holds the position.
- **Q4. How does OpenRouter rebuild Anthropic's block order (thinking, text, thinking, tool_use, plus progress-update thinking blocks) from a Chat Completions assistant message that has separate `reasoning_details`, `content`, and `tool_calls` properties?** Tried: guide, tool calling guide, spec, Vercel provider. The provider sends one array per message and puts all reasoning before text when it builds its own content list. No documentation on the server side. Fill found: the workspace report of 2026-11-17 notes the same gap for LiteLLM's `thinking_blocks`; Anthropic's rule that only the sequence of consecutive thinking blocks in the latest assistant message must match (R73) suggests "all thinking first, in order" is enough in the common case. Not confirmed for interleaved progress-update blocks.
- **Q5. Does the server reject or silently de-duplicate repeated reasoning entries?** Tried: docs and spec (no mention). The Vercel provider quotes the error "Duplicate item found with id" and says its key rule "matches the OpenRouter API's deduplication exactly" (R58). So both a server-side tracker and an error exist; which case gets which is unknown. No fill needed; this is OpenRouter-specific.
- **Q6. Does OpenRouter itself verify reasoning entries, or only forward them?** Documented: deferred-tool state "fails verification: it was modified, or was issued for a different workspace" (R42), so some entries are signed or bound by OpenRouter. For vendor reasoning the documentation says errors are passed through (R62). Whether OpenRouter checks the envelope suffix before routing is unknown. No fill found.
- **Q7. The budget-to-effort conversion thresholds.** The guide says a `reasoning.max_tokens` value "will be used to determine the effort level" for effort-only models and gives only the forward percentages (R13). No fill found; other gateways' tables would not show OpenRouter's.
- **Q8. What `reasoning.summary` does for non-OpenAI models, and which entry type Claude's summarized thinking uses (`reasoning.text` with a signature, or `reasoning.summary`).** The guide's Anthropic-format examples show both types. The Vercel provider's Anthropic handling is all on `reasoning.text`. Not settled by documentation. No live call was made.
- **Q9. Does the provenance suffix seen on Responses `encrypted_content` also appear inside Chat Completions `reasoning.encrypted.data`, and for Anthropic signatures?** All decoded examples are from the Responses route (R41). Would need one live response to settle. Not run, by the rules of this research.

## Unverified

- No request was sent to any model. Every behavior here is from documentation, the spec, the catalog, or source code. Error strings are quoted from those, not observed.
- From memory, not verified today: older versions of pi matched a `reasoning.encrypted` entry to a tool call by comparing the entry's `id` to the tool call id. The current pi source only shows a "legacy" parser that requires a non-empty `id` on entries stored on tool calls (`parseLegacyEncryptedReasoningDetail`), which is consistent with that memory and does not prove it.
- From memory, not verified today: Google's `generateContent` error for a missing signature on Gemini 3 names the function call and content block. The workspace document of 2026-09-20 quotes such a string; Google's pages fetched today describe the Interactions API and did not show it.
- The claim that the envelope suffix is how OpenRouter pins account-bound Claude replays (R63) is a guess.
- The `xai-responses-v1`, `meta-responses-v1`, `bedrock-*`, `google-vertex-*`, and `openrouter-stealth-v1` formats: the documentation gives the names and one-line glosses only. Which entry types each produces was not found.
- Whether `reasoning_content` ever appears in an OpenRouter response (R53) was judged from schemas and SDK code only.
- The Vercel provider behaviors (R55 to R60) were read at commit `7a235a4c` (package version 3.1.0). They were not executed.
- The DeepSeek empty-array requirement (R59) comes from the provider's changelog and code comments, not from OpenRouter's documentation pages.

## Workspace documents used

- `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (2026-10-28). Took: claim LC6 (lines 138 and 332), the four-way split of vendor replay contracts in section 5.4, and the `missing_thought_signature` code. Re-checked live: LC6 against OpenRouter's guide and spec (holds, R84); Anthropic, OpenAI, DeepSeek rules against vendor pages (hold, R73 to R80). Not re-checked: the exact Google `generateContent` error code.
- Cabinet output, "Anthropic Messages API harness survey" report (2026-11-17). Took: the observation that list-form reasoning fields lose block position, and that `reasoning_details` is one of three chat-completions conventions. Re-checked live against OpenRouter's spec (agrees, R61).
- `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (2026-09-20). Took: per-vendor replay rules table (DeepSeek, Kimi, Gemini) as a cross-check. Re-checked live for DeepSeek and Kimi (agree, R80, R81). Its Gemini `extra_content` statement is about Google's own compatibility endpoint and was not re-checked.
- Shared brief for this deep dive (`BRIEF.md`, 2026-11-22). Took: the claim list and the ladder of accommodation levels.

## Sources

| Key | URL, or repository at commit with file path | Read date | Kind |
|---|---|---|---|
| R-s1 | https://openrouter.ai/docs/guides/best-practices/reasoning-tokens | 2026-11-22 | OpenRouter doc |
| R-s2 | https://openrouter.ai/docs/api_reference/responses/reasoning | 2026-11-22 | OpenRouter doc |
| R-s3 | https://openrouter.ai/docs/guides/routing/model-variants/thinking | 2026-11-22 | OpenRouter doc |
| R-s4 | https://openrouter.ai/docs/api_reference/parameters | 2026-11-22 | OpenRouter doc |
| R-s5 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-4-6 | 2026-11-22 | OpenRouter doc |
| R-s6 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-4-7 | 2026-11-22 | OpenRouter doc |
| R-s7 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/sonnet-5 | 2026-11-22 | OpenRouter doc |
| R-s8 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/opus-5 | 2026-11-22 | OpenRouter doc |
| R-s9 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/fable-5-1 | 2026-11-22 | OpenRouter doc |
| R-s10 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/claude-5-5 | 2026-11-22 | OpenRouter doc |
| R-s11 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-5-4 | 2026-11-22 | OpenRouter doc |
| R-s12 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-5-6 | 2026-11-22 | OpenRouter doc |
| R-s13 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gpt-6 | 2026-11-22 | OpenRouter doc |
| R-s14 | https://openrouter.ai/docs/cookbook/evaluate-and-optimize/model-migrations/gemini-4 | 2026-11-22 | OpenRouter doc |
| R-s15 | https://openrouter.ai/openapi.json (schemas `ChatRequest`, `ChatAssistantMessage`, `ChatStreamDelta`, `ChatToolCall`, `ChatResult`, `ChatUsage`, `ReasoningDetailUnion` and its four members, `ReasoningFormat`, `ReasoningEffort`, `ReasoningMode`, `ReasoningContext`, `ReasoningConfig`, `ModelReasoning`, `ConfigurationUpdate`, `OutputItemReasoning`, `OutputReasoningItem`, `ReasoningItem`, `ResponsesRequest`, `ResponseIncludesEnum`, `ResponsesInputTransformation`, `OpenResponsesResult`, `MessagesRequest`, `MessagesOutputConfig`, `AnthropicThinkingBlock`, `AnthropicRedactedThinkingBlock`, `AnthropicThinkingBlockBinding`, `AnthropicThinkingDisplay`, `AnthropicInputTransformation`, `AnthropicOutputTokensDetails`, `MessagesContentBlockDeltaEvent`, `PrivateEndpointReasoningReturnMechanism`, `Pricing`) | 2026-11-22 | OpenRouter spec |
| R-s16 | https://openrouter.ai/api/v1/models (476 models, `reasoning`, `supported_parameters`, `pricing.internal_reasoning`) | 2026-11-22 | OpenRouter spec (model catalog) |
| R-s17 | https://openrouter.ai/docs/changelog (entries dated July 27, August 20, September 1, 4, 8, 9, October 28, November 3, 5, 10, 11, 18, 2026) | 2026-11-22 | OpenRouter doc |
| R-s18 | https://openrouter.ai/docs/guides/features/server-tools/tool-search | 2026-11-22 | OpenRouter doc |
| R-s19 | https://openrouter.ai/docs/guides/features/broadcast | 2026-11-22 | OpenRouter doc |
| R-s20 | https://openrouter.ai/docs/guides/overview/multimodal/videos | 2026-11-22 | OpenRouter doc |
| R-s21 | https://openrouter.ai/docs/guides/features/service-tiers (printed Messages response for `openai/gpt-5-mini`) | 2026-11-22 | OpenRouter doc |
| R-s22 | Printed response examples with `encrypted_content`: https://openrouter.ai/docs/guides/overview/multimodal/image-understanding , https://openrouter.ai/docs/api_reference/overview , https://openrouter.ai/docs/api_reference/responses/basic-usage , https://openrouter.ai/docs/api_reference/responses/web-search , https://openrouter.ai/docs/guides/features/server-tools/apply-patch , https://openrouter.ai/docs/guides/features/server-tools/shell | 2026-11-22 | OpenRouter doc |
| R-s23 | https://openrouter.ai/docs/guides/features/guardrails/prompt-injection | 2026-11-22 | OpenRouter doc |
| R-s24 | https://openrouter.ai/docs/guides/routing/provider-selection (Provider-Specific Headers, printed Azure examples) | 2026-11-22 | OpenRouter doc |
| R-s25 | https://openrouter.ai/docs/guides/features/tool-calling (Interleaved Thinking, streaming tool loop) | 2026-11-22 | OpenRouter doc |
| R-s26 | https://openrouter.ai/docs/cookbook/coding-agents/claude-code-integration | 2026-11-22 | OpenRouter doc |
| R-s27 | `OpenRouterTeam/ai-sdk-provider` at commit `7a235a4cad7624ed60db913b96f5f0f6bd8c8d33` (2026-11-20, package `@openrouter/ai-sdk-provider` 3.1.0): `src/chat/index.ts`, `src/chat/convert-to-openrouter-chat-messages.ts`, `src/chat/schemas.ts`, `src/schemas/reasoning-details.ts`, `src/schemas/format.ts`, `src/schemas/provider-metadata.ts`, `src/utils/reasoning-details-duplicate-tracker.ts`, `src/utils/resolve-reasoning.ts`, `src/types/index.ts`, `CHANGELOG.md`, `e2e/gemini/reasoning-multiturn.test.ts`, `e2e/issues/issue-418-gemini-thought-signature.test.ts`, `e2e/issues/issue-438-gemini-reasoning-redacted.test.ts`, `src/chat/convert-to-openrouter-chat-messages.test.ts` | 2026-11-22 | OpenRouter SDK source |
| R-s28 | `OpenRouterTeam/typescript-sdk` at commit `7ad8458755a042a1ccf1fc96046dee3dd9055849` (2026-11-21, package `@openrouter/sdk` 1.5.128): `examples/chat-reasoning.example.ts`, `src/models/reasoningformat.ts` | 2026-11-22 | OpenRouter SDK source |
| R-s29 | `OpenRouterTeam/typescript-agent` at commit `8a2d96e541cb49ecc1869d490ce54b4bcf3b5a55` (2026-11-17): `packages/agent/src/lib/turn-context.ts` | 2026-11-22 | OpenRouter SDK source |
| R-s30 | `badlogic/pi-mono` at commit `9c4652688590abb836b40ebd1e4725ecf9a7c7d1` (2026-11-22): `packages/ai/src/api/openai-completions.ts`, `packages/ai/src/api/transform-messages.ts` | 2026-11-22 | harness source |
| R-s31 | https://platform.claude.com/docs/en/build-with-claude/thinking | 2026-11-22 | vendor doc (Anthropic) |
| R-s32 | https://platform.claude.com/docs/en/build-with-claude/preserved-thinking | 2026-11-22 | vendor doc (Anthropic) |
| R-s33 | https://platform.claude.com/docs/en/build-with-claude/effort | 2026-11-22 | vendor doc (Anthropic) |
| R-s34 | https://ai.google.dev/gemini-api/docs/thinking (fetched as `thinking.md.txt`) | 2026-11-22 | vendor doc (Google) |
| R-s35 | https://ai.google.dev/gemini-api/docs/thought-signatures (fetched as `thought-signatures.md.txt`) | 2026-11-22 | vendor doc (Google) |
| R-s36 | https://developers.openai.com/api/docs/guides/reasoning | 2026-11-22 | vendor doc (OpenAI) |
| R-s37 | https://api-docs.deepseek.com/guides/thinking_mode | 2026-11-22 | vendor doc (DeepSeek) |
| R-s38 | https://platform.moonshot.ai/docs/guide/use-kimi-k2-thinking-model (page title "Thinking Models") | 2026-11-22 | vendor doc (Moonshot AI) |
| R-s39 | https://docs.x.ai/docs/guides/reasoning | 2026-11-22 | vendor doc (xAI) |
| R-s40 | `runvil-design`, `research/2026-10-28-survey-proprietary-inference-apis.md` (dated 2026-10-28) | 2026-11-22 | workspace document |
| R-s41 | Cabinet output, "Anthropic Messages API harness survey" report (dated 2026-11-17) | 2026-11-22 | workspace document |
| R-s42 | `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` (dated 2026-09-20) | 2026-11-22 | workspace document |
| R-s43 | https://platform.claude.com/docs/en/build-with-claude/extended-thinking | 2026-11-22 | vendor doc (Anthropic) |
