# Proprietary inference APIs: the request body converged, the semantics did not

A structural survey of 53 text-generation surfaces and roughly 20 media and speech surfaces, with the self-hosted servers that define what "OpenAI compatible" actually means. Documentation read 2026-10-28.

**Report type:** analytical, market-research variant. The finding leads, then the criteria, then the evidence, then the recommendation. Everything below is written so a reader can stop after this summary and still act correctly.

---

## Executive summary

Fifty-three text-generation API surfaces are covered here, plus about twenty media and speech surfaces and seven self-hosted servers that act as the compatibility baseline. Across them are eleven authentication families, six streaming framings plus a bidirectional socket shape, four incompatible designs that all go by the name "prompt caching," five ways of exposing reasoning, and six naming grammars a client must parse before it knows what it is calling.

**The through-line: the industry converged on the OpenAI request body and on almost nothing else.** Every text surface except Replicate's primary interface accepts an OpenAI-shaped chat request, and half accept an OpenAI Responses request too. But the differences that cost engineering time are not in the request. They are in five things that all return HTTP 200 when they go wrong: how reasoning state is replayed, what counts as a cache hit, how tokens and cost get reported, what an error code means, and whether a schema guarantee is real or best-effort. A gateway that translates request bodies and passes responses through produces structurally valid, semantically wrong traffic, and the wrongness appears as cost overruns and intermittent parse failures rather than as errors.

**The second finding: structural role predicts API shape far better than company size does.** First-party vendors own their protocol and may change it; cloud wrappers invent a unified shape and lose what it cannot express; aggregators grow supersets and become the place where cross-vendor conventions get written down; dedicated platforms inherit their engine's shape and gaps; and the self-hosted baseline, having no commercial reason to converge, diverges in seven directions. A company in two roles at once, like Alibaba or Tencent or Runway, ships both shapes and the seams show.

**What to do with it.** Classify an unfamiliar vendor's role before reading its endpoint page. Then test the five 200-OK axes early, since they are the ones that do not fail loudly: replay a reasoning turn through a tool call, measure a cache hit twice, compare reported usage against your own count, trigger a 429 on purpose, and send a schema the model has not seen. And never treat a successful model list as evidence a provider will accept a request body; Gemini's list only validates that a key exists, and Volcano Ark's returns entries including internal models you cannot call.

Two judgments worth putting in front of a decision-maker, both confidence high because the behaviors are documented with specific error strings rather than inferred. Reasoning replay is the hardest thing to translate, because four vendors define four different correct answers and one of them is "you cannot see the state at all." And the compatibility promise is weaker than it looks: Anthropic's own OpenAI-compatible layer documents that it never returns thinking, never caches, ignores `strict` and `response_format`, and reports empty token-detail objects, so a client routed there gets a 200 with no reasoning, no cache discount, no schema guarantee, and a cost report showing zero.

Evidence quality is uneven. Vendor documentation read today is the strongest material. Local workspace documents range from two to thirteen weeks old and are cited with their age. Chat transcripts contain claims graded as cited, code-grounded or from memory, and remembered claims are marked unverified rather than promoted.

## Contents

- Executive summary
- Scope and method
- The map: five structural roles that predict API shape
- Vendor coverage by role
- The twelve axes where vendors actually diverge
- The OpenAI shape as an industry default
- Media and realtime: five shapes, not one
- What the structure implies for a gateway
- Uncertainties and open questions
- References

---

## Scope and method: what counts, what was read, and how old it is

**What counts as a proprietary inference API.** A hosted HTTP surface that runs a model you did not train and returns a completion, embedding, image, audio artifact or live session, where the operator is not you. Three exclusions follow. Weights you download are out of scope, which is why the self-hosted servers appear only as a baseline. Consumer chat products are out, because the interesting surface is the one with a request body. And resold models count under whoever operates the API, because the wire contract is the operator's; Baidu reselling DeepSeek through Qianfan tells you nothing about DeepSeek's own API and a lot about Qianfan's.

**Why self-hosted servers are in a report about proprietary APIs.** Because they define the phrase. vLLM, SGLang, TGI, llama.cpp, Ollama, LM Studio and LocalAI are what many hosted platforms wrap, so a hosted vendor's OpenAI shape is frequently vLLM's parameter set with a price attached. They are the control group, not a subject.

**How the evidence was gathered.** Three source types, in descending strength.

- **Vendor documentation read on 2026-10-28.** The bulk of the report and the only material treated as current behavior. Roughly 150 vendor pages across API references, guides, pricing and deprecation lists. Every cross-vendor factual claim carries the page and the read date.
- **Local workspace documents dated 2026-07-26 through 2026-09-30.** A Rust agent framework with a driver per vendor, plus the owner's gateway design and the surveys behind it. Four to thirteen weeks old. Valuable for wire detail that vendor docs state once and never repeat, and dangerous for anything a vendor could change. Claims resting on these carry their age.
- **Chat transcripts, graded.** Claims are marked cited (a page was fetched that day), code-grounded (a repository was read with line numbers), or from memory. Several remembered claims proved right and several did not, so none is promoted to a fact.

**What the method cannot do.** Three limits. Documentation describes intent, not deployment, and a vendor can document a field its backend ignores; xAI is unusual in saying so. Vendor pages contradict each other, and where they do the report reports rather than resolves, because resolving needs a live call I did not make. And churn invalidates a survey inside a quarter: in the window these notes were gathered, OpenAI retired its Videos API and deprecated the Speech API, Google retired Imagen 4 and killed the Veo 3.1 preview, Tencent discontinued a platform, and 01.AI closed self-serve with a month of notice.

Two verification notes were commissioned against flagged contradictions, and both arrived and are incorporated.

---

## The map: five structural roles that predict API shape

Sort vendors by what they are structurally rather than by size, and API shapes fall into five buckets. This is the most useful thing in the report, because it places a vendor you have never heard of before you open its docs.

**Table 1. Five structural roles, their characteristic shape, and what each does to you.**

| Role | Characteristic shape | Who | What bites |
|---|---|---|---|
| First-party model vendor | One or two native dialects, self-defined | OpenAI, Anthropic, Google, xAI, Meta, MAI, Mistral, DeepSeek, Moonshot, Alibaba, Zhipu, MiniMax, ByteDance, Baidu, StepFun, iFlytek, Tencent | The compatibility path drops reasoning, caching and schema guarantees |
| Cloud platform wrapper | A unified shape of its own, with vendor-shaped routes bolted on | Bedrock, Azure OpenAI, Vertex AI, watsonx, OCI, Databricks | Translates twice; loses model-specific parameters; auth is cloud IAM |
| Aggregator and router | OpenAI-shaped superset with routing controls | OpenRouter, Hugging Face Inference Providers, Qwen and TokenHub as resellers, Runway | Normalizes and flattens; failover stops once output starts |
| Dedicated inference platform | OpenAI-shaped server over its own engine, or a job system | Together, Fireworks, Groq, Cerebras, SambaNova, Replicate, Baseten, Modal, Predibase, DeepInfra, Nebius, Hyperbolic, Cloudflare | The engine's parameter set leaks through; cache is per replica |
| Self-hosted baseline | OpenAI surface over a local engine | vLLM, SGLang, TGI, llama.cpp, Ollama, LM Studio, LocalAI | Each one breaks "compatible" differently |

Read down the middle column and the pattern is visible. The first-party vendor owns its protocol and can change it, which is how Anthropic deletes `temperature` and returns 400 on every model that sends it. The cloud wrapper does not own the protocol, so it invents a unified one and loses whatever the vendors' protocols could express that its own cannot. The aggregator cannot own it either, so it grows a superset and becomes the place where cross-vendor conventions get written down. The dedicated platform inherits its engine's shape and gaps. The self-hosted baseline has no commercial reason to converge, so each server diverges.

**Four companies occupy two roles at once,** and their docs are confusing for a structural reason rather than a writing failure. Alibaba is a first-party vendor and an aggregator, with about 145 first-party models plus 6 DeepSeek, 3 GLM and 3 Kimi under one key (chat `a74ca56c`, 2026-09-14, graded cited). Tencent replaced its first-party platform with an aggregator: the original Hunyuan API stopped selling 2026-06-30, replaced by TokenHub, which serves DeepSeek, GLM, Kimi and MiniMax alongside its own Hy models (tencentcloud.com/announce/detail/101147, read 2026-10-28). Runway now resells other vendors' video models next to its own. And Microsoft is wrapper and vendor at once, since MAI shares the Azure base. Perplexity is a dedicated platform with a tiny catalogue; Lepton, acquired by NVIDIA in March 2025, is a GPU endpoint deployer whose OpenAI surface comes from whatever engine you put in the container.

---

## Vendor coverage by role

Each entry below covers host and route, auth, request and content model, how tools are carried, streaming family, reasoning, caching, error envelope, versioning, pricing basis, and what an OpenAI-compatible path costs you. Where a vendor is structurally similar to one already described, the entry is shorter and the contrast carries the weight.

### First-party model vendors: the protocol belongs to them

**OpenAI owns two protocols and treats the older one as legacy.** Chat Completions at `POST /v1/chat/completions` is the surface the industry copied: Bearer auth, `tool_calls` whose `arguments` is a JSON string, SSE terminated by `[DONE]`. Responses at `POST /v1/responses` is what OpenAI develops: `input` is an array of typed items including `function_call`, `reasoning` and `compaction`, the system prompt is top-level `instructions`, built-in tools share one namespace with your functions, streaming is named events, and completion is `status` plus `incomplete_details.reason`. So the industry standard is OpenAI's legacy path while OpenAI's current path is not the standard.

One behavior changed recently in a way that breaks older assumptions: reasoning items now carry `encrypted_content` **by default**, where it used to require an explicit `include` opt-in (platform.openai.com/docs/api-reference/responses/create, read 2026-10-28). Code that treats an absent field as "no encrypted reasoning" and a present one as "caller asked for it" now reads the same signal in both cases. Read it from `response.output_item.done` rather than `.added`, because the item is not final when the added event fires.

**Anthropic's native shape is unlike OpenAI's, and it is what gateways translate into.** `POST /v1/messages` is the route, and the auth model changed during this survey: `Authorization: Bearer` is now the documented primary scheme, taking an API key or a short-lived token from `POST /v1/oauth/token`, and `x-api-key` is described as a "Legacy fallback for `Authorization`, still supported" (docs.anthropic.com/en/api/getting-started, read 2026-10-28, confirmed by live probe). Most documentation in circulation, including this workspace's notes, still presents `x-api-key` as the model, so this is a correction rather than a detail. The request has only `user` and `assistant` roles with the system prompt hoisted to top-level `system`, plus typed blocks including `thinking` and `redacted_thinking`. Tools are `{name, description, input_schema}` with no `type` wrapper, and `tool_use.input` is a JSON object rather than a string, which is where naive translations break.

Thinking blocks carry a `signature` holding encrypted reasoning, and `display: "omitted"` returns an empty `thinking` field while keeping the signature, now the default on many models. Replay is prefix-bound: on current models a block is accepted only while the system prompt, tools and every preceding message are unchanged, enforced by default for accounts created on or after 2026-08-31. Caching is explicit per-block rather than automatic.

One field needs a hedge because it is easy to overstate. `max_tokens` is mandatory and Anthropic documents no default, so a request omitting it fails. But any fallback shape, whether a profile lookup then a retry at 16,384 or a per-model `default_max_tokens`, is the caller's own policy and not provider behavior. Neither appears in Anthropic's documentation, and presenting either as how the API behaves would be wrong. A gateway has to invent the policy and own it.

**Google Gemini is the most structurally different first-party API, and the difference is a convention.** `POST /v1beta/models/{model}:generateContent` takes `contents` of `{role, parts}` with roles `user` and `model`, so the assistant role is literally named `model`. Parts are typed by which field is set rather than by a type string, and that key-is-the-type convention is the biggest tripwire in this survey. System prompt is a sibling `systemInstruction`, parameters nest in `generationConfig`, and tool parameters use a restricted OpenAPI subset. Auth is `x-goog-api-key` or `?key=`, and Google is migrating to service-account-bound keys.

The structural change a gateway has to plan around is that **Google's default documented surface is now the Interactions API, and Google labels `generateContent` the legacy path** (ai.google.dev/gemini-api/docs/thinking, read 2026-10-28). Two things differ. Thoughts are first-class `thought` steps carrying a mandatory `signature` rather than parts flagged inside a response, and the thinking control is `thinking_summaries` taking `auto` or `none`, not `thinkingConfig.includeThoughts`. Everything this report says about thought signatures holds exactly on `generateContent`, but a reader building against the new surface should not assume the flag-based vocabulary carries over. The media note corroborates the migration independently: Google killed the Gemini API Veo 3.1 previews on 2026-10-22 and routed them to `gemini-omni-1.1-flash` on the Interactions API, while Vertex keeps Veo 3.1 GA on `:predictLongRunning`.

Reasoning is on by default on 3.x, the sampling trio is ignored on 3.6 and later, and thought signatures are mandatory: for parallel calls only the first `functionCall` carries one, and omitting it is a 400 with code `missing_thought_signature`. A blocked prompt returns no candidate at all, so you read `promptFeedback.blockReason`, and `CONTINUATION` means the output cap was hit and you resend with a returned token. The OpenAI route at `/v1beta/openai/` loses thought signatures, `alt=sse` behavior, `continuationToken`, safety settings and cache handles.

**xAI, Meta, Mistral and MAI each imitate OpenAI closely and each break it in one specific place.** xAI serves Responses as the documented primary plus Chat Completions and a legacy Anthropic-compatible route, caps JSON-schema tools at 1,000 per request, and runs `reasoning.effort` from `low` to `xhigh` with no off switch on 4.5 and later. What makes it worth reading is honesty: `background` unsupported, `context_management` parsed but not executed, `output_text.logprobs` accepted and ignored. Meta serves all three dialects at the same price but restricts `tool_choice` to `"auto"` alone, with `"none"`, `"required"` and named functions all returning 400, the strictest restriction here; its `-contributor` suffix encodes data-use consent in the model id. Mistral adds `prompt_mode` and `safe_prompt` and is the most permissive on tool choice, but with `reasoning_effort: "high"` the delta shape mutates mid-stream from a `ThinkChunk` list to a plain string, so a client assuming a string crashes or silently drops the answer. MAI is a model behind the Azure surface rather than a protocol of its own, and `reasoning_display: "encrypted"` adds a Microsoft-only `reasoning.encrypted_content` field.

The pattern across all seven is in Table 2: tool-argument typing and reasoning exposure are the two columns determining whether a translation is mechanical.

**Table 2. First-party vendors on the axes that cost the most integration work.**

| Vendor | Auth | Tool args | Streaming | Reasoning | Caching | OpenAI path |
|---|---|---|---|---|---|---|
| OpenAI | Bearer | JSON string | Chat chunks + Responses events | Opaque, encrypted replay | Auto + breakpoints | Native |
| Anthropic | Bearer, `x-api-key` legacy | JSON object | Named events, block lifecycle | Signed, prefix-bound | Explicit per block | Yes, disclaimed |
| Gemini | Header or `?key=` | Object in typed part | Chunked, `alt=sse` needed | Thought parts, signatures | Implicit + objects | Yes, lossy |
| xAI | Bearer | JSON string | Both | Deltas + encrypted items | Auto, routing hint | Native |
| Meta | Bearer | JSON string | Both | Encrypted items | Auto prefix | Native, three dialects |
| MAI | `api-key` or Entra | JSON string | Chat chunks | Encrypted, off by default | Auto, undocumented | Chat only |
| Mistral | Bearer | JSON string | Data-only, polymorphic | Plaintext ThinkChunks | Auto, 64-token blocks | Native |

### Cloud platform wrappers: a native shape of their own, then compatibility bolted on

**Bedrock translates hardest.** `bedrock-runtime` carries five surfaces, and AWS runs a second host `bedrock-mantle` for compatibility only on the same engine. Auth is SigV4 or a Bedrock API key as bearer, and both still need `bedrock:InvokeModel` on the principal. `modelId` accepts base ids, inference profile ARNs, provisioned throughput ARNs, imported models and prompt ARNs in one field, which is identity-based routing rather than string-based.

`InvokeModel` is a byte pipe whose `body` schema is whatever the family wants. `Converse` replaces that with typed content blocks, `system` as blocks, `toolConfig`, and `inferenceConfig` holding exactly four base parameters, with everything else pushed into `additionalModelRequestFields`. Tool results return as `toolResult` blocks inside a user message rather than a `tool` role. Reasoning arrives as a `reasoningText` block whose signature hashes all messages and must be replayed with all previous messages or the call errors. Streaming is a binary event stream, not SSE, with block indices so text, reasoning and tool-input deltas interleave, and stop reasons are a closed normalized set where the vendor value never survives. The compatibility paths degrade capability rather than detail: Responses rejects `background=true`, drops server-side tools including web search, and returns 404 on `GET /models`, and the two hosts hold separate quotas for the same model.

**Azure OpenAI adds almost no protocol translation, and all its friction is identity, and it has also folded two hosts into one surface.** Classic puts the deployment in the path and requires `api-version`; v1 removes both, using `/openai/v1/` with the deployment name in the body's `model` field. Auth is `api-key`, granting full resource access and not scopeable, or Entra ID bearer. Bodies, tools, streaming and stop reasons are OpenAI's. What differs is naming and errors: the docs state that "Azure OpenAI always requires deployment name, even when using the model parameter," names are arbitrary, so two resources can serve one model under different names, and a filter trip arrives as `inner_error` with a code like `ResponsibleAIPolicyViolation` rather than as a finish reason.

The host consolidation matters for classification. `services.ai.azure.com` and `openai.azure.com` now both land on the same `/openai/v1` base and serve third-party Foundry models including MAI, DeepSeek and Grok, so MAI is not a separate surface from Azure OpenAI and treating it as one overcounts the distinct APIs in the landscape. A Foundry project endpoint still changes more than the URL: it routes SDK traffic to Responses, exposes platform tools the plain endpoint lacks, does not route embeddings at all, and gives Anthropic models a separate surface, so one resource presents multiple protocol families with different capability on each.

**Vertex AI wraps the vendor protocol instead of translating it, which makes it the most faithful wrapper here.** Routes are `POST /v1/{model}:generateContent` where `{model}` expands to `projects/{project}/locations/{location}/publishers/{publisher}/models/{model}`. Three path segments do work: `project` is an authorization boundary validated by IAM against the URI, `location` is a region or `global`, and `publisher` selects `google` or a partner. Global improves availability while making data residency impossible.

Anthropic on Vertex is the clearest near-passthrough in the survey: the body is a Messages body, `model` is omitted because the URL names it, and `anthropic_version` moves from a header into the body as `vertex-2023-10-16`, with `tools`, `tool_choice`, `thinking` and `cache_control` all keeping Anthropic's shape. What it loses is a list: input sources, server-side tools, agent infrastructure, the Batches and Models and Admin APIs, Managed Agents and server-side `fallbacks`, with payloads capped at 30 MB, and it rejects first-party beta headers as 400.

**The enterprise hosts invert the wrapper problem by exposing OpenAI shape directly.** Cohere, AI21, NVIDIA, watsonx, Databricks and OCI all present something recognizably OpenAI-shaped, so a gateway must detect which vendor quirks hide behind familiar field names rather than translate a foreign protocol. Cohere is the best case study in what a real version bump changes: `preamble` becomes a `system` role inside one `messages` list, `chat_history` disappears into it, and response text moves from `res.text` to `res.message.content[0].text`. Two structural limits: structured output is explicitly unsupported with `documents` or with `tools`, so the guarantee is mutually exclusive with RAG and with tool use, and `tool_choice` accepts only `REQUIRED` and `NONE`. Stop reasons include `ERROR` and `TIMEOUT`, so a 200 can represent a failed generation.

The rest compress into one sentence each. NVIDIA NIM forces alternating roles ending on `user` and defaults `max_tokens` to 1,024 with a range that silently truncates. watsonx puts the scoping resource in the body rather than the URL and requires a `version` query that does not freeze behavior. Databricks applies a server-side prompt template on pay-per-token endpoints, so the model does not see what you sent and `prompt_tokens` includes the server's additions. And OCI is the only design that admits the shapes are not reconcilable: the native body carries `chatRequest.apiFormat` taking `GENERIC`, `COHERE` or `COHEREV2`, and the response echoes the choice, which is more faithful to the underlying vendor APIs than Bedrock's unification and more work for every client.

### Aggregators and routers: where the cross-vendor standards actually get written

**OpenRouter is the reference implementation of the role.** `https://openrouter.ai/api/v1` serves Chat Completions, a Responses-shaped route and an Anthropic Messages surface, with a model catalogue that returns 200 with no auth header at all. The request is a superset rather than a clone: extra sampling controls, `models` as a fallback list, a structured `provider` block, plugins, `session_id`, and a debug switch returning the transformed body sent upstream. Unsupported parameters are silently ignored rather than rejected, a meaningful philosophical difference from OpenAI's 400-on-unknown stance.

That permissiveness has a hard edge, because it is the opposite of the general rule. The `/responses` shim **rejects** `previous_response_id`: a request with a non-null value gets a 400, and `store` must be absent or literally `false` (openrouter.ai/docs/api/api-reference/responses/create-a-response, read 2026-10-28). So the aggregator does not hold conversation state server-side the way OpenAI's own Responses does, and it says so with an error rather than by ignoring the field. A gateway that models "aggregator ignores unsupported fields" as a blanket rule is wrong on this call, and the difference changes whether a client silently loses context or fails loudly.

Three behaviors define the role. Tool calling has three implementations behind one field name: native pass-through, transformation to provider format, and, for providers with neither, a YAML template injected into the prompt. Reasoning is a structured details array that must replay unmodified and in order. And caching is translated in both directions, where an incoming `cache_control` becomes a `prompt_cache_breakpoint` on the OpenAI route and the reverse toward Anthropic and Google, with TTLs dropped rather than converted. The control surface is the richest anywhere: provider order, fallback toggles, data-collection policy, ZDR, quantization filters, percentile latency preferences, and a max price where the hard ceiling fails the request and performance preferences only deprioritize. Server tools include web search, shell, advisor and subagent, and `deferred_tools` lets a caller hold tools back from the initial context, the aggregator's answer to tool-search deferral. The response carries both a normalized `finish_reason` and `native_finish_reason`, the most honest normalization found anywhere.

**The dedicated platforms are servers rather than reshaper layers, and their differences are their engines.** Together accepts then ignores service tier, store, metadata and prediction, and its reasoning has three patterns across the catalogue that must be symmetric on replay. Fireworks is a superset with a context-overflow flag that matters, because the default truncates silently by lowering max tokens where OpenAI errors, and its caching is the most explicit control surface of any dedicated platform, since a cache only works within one replica and it must ship affinity and isolation headers. Groq is the narrowest subset and fails hard: logprobs, logit bias and any `n` other than 1 all 400. SambaNova runs a per-account base URL with the narrowest caching here.

**Replicate is the structural exception: a job system, not a chat API.** A create call takes a version and an input object whose schema the model declares, polling and cancel are separate routes, and terminal predictions carry compute time and wall clock including queueing. Inputs, outputs and logs are deleted after one hour by default, which matters for any gateway wanting to cache or replay. Every non-official model needs `owner/name` plus a 64-hex version, and the version string is mandatory, a real operational difference from alias-based platforms.

**Hugging Face Inference Providers is the clearest reshaper, and its control surface is deliberately minimal.** Provider selection is a suffix on the model ID rather than a request field, with automatic failover when a provider is flagged unavailable. The compatible route is chat-only, and no per-provider parameter matrix is published, so which parameters work depends on which provider wins. That suffix design is the exact opposite of OpenRouter's structured provider object.

### Dedicated inference platforms: the long tail that behaves like the middle

These platforms are structurally similar, so the useful content is where each breaks. **Baseten** serves Chat Completions, Responses and a beta Anthropic route, executes web search server-side, and caps ingress at 100 MB. **Modal** is not a token API by default: shared and dedicated modes both expose chat and Responses plus an Anthropic route, and the documented pattern runs vLLM, so the surface you get is vLLM's. **Predibase** selects an adapter through the `model` field, overloading a field that elsewhere names a model, and caps `n` at 2 with `max_tokens` defaulting to 20. **Cloudflare Workers AI** offers Responses only for two GPT-OSS models and non-streaming only. **DeepInfra** runs a native route as the full surface with the compatible route as a facade, **Nebius** uses a control-plane-issued routing key as the model value, and **Hyperbolic** retired its serverless API and relaunched on 2026-10-21 as Flux, requiring prepaid credits.

### The self-hosted compatibility baseline: seven servers, seven meanings of "compatible"

**vLLM** has the broadest OpenAI route coverage of any server: completions, chat, batch chat, Responses including retrieve and cancel, embeddings and audio, with auth as an optional flag. **SGLang** has the most configurable reasoning: a parser flag per family, model-specific template keys, a separate-reasoning toggle defaulting to true, and a strict-thinking flag for a grammar-level token budget. **TGI** documents an OpenAI Messages API as fully compatible from 1.4.0 onward and enforces caps server-side so an over-limit request fails rather than being clamped. **llama.cpp** is the most divergent: the native completion route is explicitly not OpenAI-compatible, grammar and JSON schema exist only on the native route, and its reasoning-format flag has a mode that leaves think tags in content while also populating the reasoning field. **Ollama** supports tools, response format, seed, stop and streaming while rejecting tool choice, logit bias, `user`, `n` and image URLs. **LM Studio** implements Responses well enough that Codex works against it, and **LocalAI** extracts structured tool calls from model output regardless of backend, making tool calling emulation by architecture.

### China and APAC vendors: the same shape with different failure modes

These are first-party vendors whose surfaces are mostly OpenAI-shaped, so the interesting content is the failures that look like something else. **DeepSeek** is genuinely OpenAI shape and the primary documented path rather than an adapter, but `tool_choice: "required"` and named choice return 400 in thinking mode. **Moonshot Kimi** isolates accounts, balances and keys by region, so a `.cn` key on the `.ai` host returns 401 with no hint the endpoint is wrong. **Alibaba Qwen** is region-bound the same way, wraps `{model, input, parameters}` one level deeper on its native route, and cannot serve Qwen-Audio over the compatible route at all.

**MiniMax** is the clearest case of a surface that looks compatible and is not: the native route returns HTTP 200 with `base_resp.status_code`, so status-only handling reports failures as successes. **ByteDance Doubao** runs two replay regimes and switches between them at `doubao-seed-2-0-lite-260428`, so a model upgrade silently changes the contract. **Baidu ERNIE** carries an extra integer `flag` beside `finish_reason`. **StepFun** returns both `reasoning` and `reasoning_content` identically. **iFlytek Spark** splits auth by transport, so HTTP uses a bearer "APIPassword" while its WebSocket uses HMAC-signed URL auth and cannot do tool calls. **Zhipu GLM** accepts a two-part key as a bearer token or mints an HS256 JWT from it, and its `clear_thinking` defaults to `true` so history reasoning is silently discarded.

The pattern is that these vendors' compatibility is real at the request layer and unreliable at the semantics layer, which is the report's through-line in its sharpest form.

### Exits, migrations and status changes

Four vendors in these notes are not live providers. **01.AI closed self-serve**: a notice dated 2026-08-03 stopped API calls at 24:00 on 2026-09-03 and ran refunds to 2026-12-03, with the company still selling enterprise deployment on DeepSeek (ithome.com/0/987/951.htm, Chinese, read 2026-10-28). **Tencent replaced a platform rather than a model**: the original Hunyuan API was OpenAI-compatible with an admitted divergence where `stop` truncated after the matched string rather than before, and it is gone in favor of TokenHub, which says outright that fields "may be ignored or degraded" (tencentcloud.com/announce/detail/101147, read 2026-10-28). **PlayHT is dead**, with `play.ht` returning no DNS record by 2026-09-22. And **Hume gives sixteen days**, with docs stating "Hume is sunsetting the TTS and EVI APIs. Access ends November 13, 2026 at 12:01 a.m. EST." A vendor can exit this business in a quarter with about a month of notice, which is the practical case for a fallback model list.

---

## The twelve axes where vendors actually diverge

The happy path is nearly universal: a POST with a model name and some messages, a bearer token, and a streamed answer. Everything expensive lives in the twelve axes below, ordered from cheapest to hardest to translate.

### 5.1 Authentication: eleven families, and one that signs the path

Eleven credential shapes appear across the field, and only three are variants of a bearer token.

**Table 3. Authentication families and what a generic OpenAI client cannot produce.**

| Family | Example | Why a plain OpenAI client fails |
|---|---|---|
| Static bearer | OpenAI, xAI, Together, Mistral, most Chinese vendors | Nothing; the base case |
| Vendor-specific key header | Azure `api-key`, Black Forest `x-key`, Deepgram `Token`, Anthropic `x-api-key` as legacy fallback | Header name is not configurable in many SDK builds |
| Required version header | Anthropic `anthropic-version`, Cartesia `Cartesia-Version`, Runway `X-Runway-Version` | No header slot; omitting it fails |
| Query-parameter key | Gemini `?key=`, Gemini Live socket | No query injection point on some paths, and keys land in logs |
| AWS SigV4 | Bedrock, Polly, Transcribe | Canonical request plus a key derived from service, region and day |
| Service-account OAuth2 | Vertex AI | RS256 signing with a private key |
| Microsoft Entra ID | Azure OpenAI, Foundry, MAI | Token fetch with a route-specific scope |
| JWT from an access/secret pair | Baidu Qianfan `bce-v3/ALTAK-...`, Zhipu HS256 mode, iFlytek WebSocket | Local signing step before a header exists |
| OAuth2 client credentials | Databricks, IBM IAM | Token endpoint plus refresh loop |
| Mutual TLS | Google `[service].mtls.googleapis.com` | Client certificate in the TLS handshake |
| HTTP Basic | Some self-hosted gateways | SDK sends bearer |

Auth complexity is uncorrelated with model quality: three of the four most capable first-party labs are plain bearer or a single custom header, while Bedrock, Vertex, OCI and watsonx all need cloud IAM machinery before the first token moves. That matters because cloud IAM shapes cannot be proxied by rewriting headers. SigV4 enters method, path, query, body hash and timestamp into the signature, so anything that rewrites a signed path invalidates the request.

Two credential changes inside stable-looking interfaces break quietly. Google is migrating Gemini to service-account-bound keys, with new AI Studio keys being auth keys from 2026-05-28 and unrestricted standard keys rejected. And several vendors make credentials region-scoped without saying so: a Moonshot `.cn` key on the `.ai` host returns `401 invalid_authentication_error` with no hint the endpoint is wrong, and Tencent's Guangzhou and Singapore keys do not interoperate.

### 5.2 Compatibility layers: three products share one name, and one is documented as unfit for production

The phrase covers a first-class native surface, a migration aid the vendor disclaims, and a third-party normalization proxy, and the three fail differently.

Anthropic's own layer is the most useful document in the ecosystem because it enumerates what it drops: thinking never returned, prompt caching unsupported, `strict` on tools ignored, `response_format` ignored, `logprobs`, `metadata`, `seed`, `service_tier`, `store`, `user`, `reasoning_effort`, `logit_bias`, `audio` and both penalties silently ignored, token-detail objects always empty, error messages explicitly not equivalent, and system and developer messages hoisted into one top-level field so mid-conversation system messages change meaning (docs.anthropic.com/en/api/openai-sdk, read 2026-10-28). Anthropic states the layer "is primarily intended to test and compare model capabilities, and is not considered a long-term or production-ready solution." Note what it does not do: it does not reject a top-level `cache_control`, because unsupported fields are silently ignored, so a client sending cache markers there may get a silently uncached request rather than a 400.

Read that as a generic checklist. The layer preserves the response envelope and the chunk shape; it does not preserve reasoning state, cache state, schema guarantees, or accounting detail. Gemini's compatible route makes the same trade explicitly, since `reasoning_effort` and `thinking_level` overlap and "can't be used at the same time," so a relay must pick one. Bedrock degrades capability rather than detail: Responses there rejects `background=true`, drops server-side tools, and returns 404 on `GET /models`. The judgment to act on, confidence high because the losses are vendor-documented: treat a compatibility route as a separate product with its own capability set, not as a transport switch.

### 5.3 Streaming: six families, and the error that arrives after a 200

SSE is universal for text, but six framings exist and they differ in whether they carry a lifecycle.

**Table 4. Streaming families and what a relay must do to bridge each.**

| Family | Shape | What bridging costs |
|---|---|---|
| OpenAI Chat data-only chunks, `[DONE]` | Flat `choices[0].delta`, positional tool-argument fragments | Synthesize a terminator for non-SSE peers; usage absent unless requested |
| OpenAI Responses named events | `response.created` through `response.completed` | Know which events are terminal; item lifecycle has no chat analogue |
| Anthropic named events | `message_start`, per-index block start/delta/stop, `message_stop` | A state machine per block index |
| Gemini chunked response | Complete response objects, `alt=sse` optional | Text is a part list, not a string |
| Bedrock Converse | Binary event stream with block deltas and a metadata event | Not SSE; block indices interleave text, reasoning and tool input |
| Cohere v2 typed events | `message-start`, `content-delta`, `citation-*`, `tool-*` | Extra event families a naive mapper drops |**Two facts compress the axis.** Usage reporting sits in a different place in every family: OpenAI omits it unless `stream_options.include_usage` is set and then sends one final chunk with empty choices, Anthropic sends input at `message_start` and cumulative output at `message_delta`, Bedrock sends a `metadata` event with a per-TTL cache breakdown, and Gemini has separate thought-token counters. And errors go in band once the 200 is committed. Anthropic states that "an error can occur after the API returns a 200 response," arriving as an `event: error`. OpenRouter states the consequence: it "sends you the HTTP 200 OK status and headers as soon as the provider accepts the request," so every later failure is reported inside the response, and failover "stops once part of the answer has reached you."

### 5.4 Reasoning and thinking: five exposure models and four incompatible replay contracts

This is the hardest axis, because the correct answer depends on a history the client may have edited.

Five exposure models are in production. DeepSeek and Together return a separate field alongside `content`. Anthropic and Google return typed blocks with signatures: Anthropic's `thinking` blocks carry a `signature` holding encrypted reasoning, and Gemini's `thought` parts carry a `thoughtSignature` described as "an encrypted representation of the model's internal reasoning state." OpenAI returns opaque items whose text is unavailable, now with `encrypted_content` populated by default rather than only on request. Both OpenAI and Anthropic offer a summary mode instead: `summary: "auto"` returns `summary_text`, and `display: "omitted"` returns an empty `thinking` field while the signature still holds the full reasoning, now the default on several models. And Anthropic's compatibility layer exposes nothing.

The surface is moving under the field names. Google's default documented surface is now the Interactions API, where thoughts are first-class `thought` steps with a mandatory `signature` and the control is `thinking_summaries` taking `auto` or `none`, not `thinkingConfig.includeThoughts`; `generateContent` is labelled the legacy path (ai.google.dev/gemini-api/docs/thinking, read 2026-10-28). The signature requirement is identical in both, so the replay rule below is stable even though the vocabulary is not.

Anthropic's control is a three-way split rather than the two-way choice older material describes. `thinking.type.enabled` with `budget_tokens` is rejected with a 400 on Claude 4.7 and later, and the error names adaptive thinking plus `output_config.effort` as the replacement; 4.6 only deprecates it and those requests still succeed. `thinking.type.between_tools` exists on Sonnet 5.5 alone, accepted only at effort `high` or below and taking no other field. And `disabled` is accepted on Haiku 5.5 but not on Sonnet 5.5 (docs.anthropic.com/en/api/errors, read 2026-10-28). A gateway with two branches for "adaptive versus budget" has no branch for the model that only supports the middle setting.

The replay contracts split four ways, and this is where a gateway stops being a proxy.

- **Hard 400 on missing replay.** DeepSeek returns 400 if `reasoning_content` is not fully passed back "even for turns where the model did not perform a tool call," but only when `tools` is present; without `tools` replayed reasoning is ignored. Gemini requires all thought blocks resent exactly, with the code `missing_thought_signature`.
- **Hard 400 on modification, with prefix binding.** Anthropic documents four separate 400s for edited, reordered, filtered or reconstructed blocks, naming the position, plus a rule where a replayed block is accepted only while the system prompt, tools and every preceding message are unchanged.
- **Soft degradation.** Kimi K2.x, Qwen and GLM in default modes drop historical reasoning silently and succeed with quality loss. Mistral warns that stripping `ThinkChunk` degrades quality sharply but does not error.
- **Optional, but now shipped by default.** OpenAI only recommends replay and "will smartly ignore any reasoning items that aren't relevant," yet `encrypted_content` arrives populated on reasoning items, so the payload to replay is already there whether or not the caller asked. Read it from `response.output_item.done` rather than `.added`, because the item is not final when the added event fires.

The consequence is uniform even though the contracts are not: reasoning is a per-turn artifact whose validity depends on history immutability, so a gateway doing context editing, pruning, system-prompt rewriting or compaction must strip reasoning blocks rather than edit around them. Confidence high, since every behavior above is documented with a specific error string.

### 5.5 Prompt caching: four designs, one name, and a discount that can cost more than it saves

**Table 5. Caching designs, with a representative and the trap in each.**

| Design | Representative | Read price | Write price | What breaks it |
|---|---|---|---|---|
| Automatic, no control | Groq, Meta, DeepSeek, Mistral | 50% off at Groq, 10% of input at Mistral | None | Any prefix change; Mistral's 64-token blocks mean short prompts never hit |
| Explicit breakpoints | Anthropic, OpenAI GPT-5.6+ | 0.1x, down to 0.025x | 1.25x at 5m, 2x at 1h; OpenAI 1.25x at 30m, 2.5x at 12h | Tool definitions, `tool_choice`, thinking budget, `speed`, an image anywhere |
| Named cache resources | Gemini `cachedContents` | Per-token cached rate | Hourly storage per million tokens | `tools`, `toolConfig` and `systemInstruction` are immutable; absent on the Interactions API and on Gemini 4 Argon+ |
| Translated pass-through | OpenRouter | Provider's cached rate | Provider's write rate | TTLs drop across provider families; the discount field goes negative on Anthropic writes |

The invalidation rules generalize better than the prices. Anthropic's table is the most explicit: the cache follows `tools` then `system` then `messages`, and changes at each level invalidate that level and everything after, so a tool-definition change invalidates everything while a `tool_choice` change invalidates only message blocks. Changing `budget_tokens` invalidates message breakpoints "because the budget value is rendered into the prompt," which makes reasoning configuration part of the cache key.

Two Gemini specifics are commonly stated wrong. The explicit-cache minimum is **per model, not universal**: 4,096 tokens for the Gemini 3 family and 2,048 for Gemini 2 and 2.5, where Google's page says only that the minimum "varies by model" and its tables carry the figures (ai.google.dev/api/caching, read 2026-10-28). And explicit caching does not exist at all on the Interactions API, so the migration to Google's default surface removes the cache-management layer rather than porting it.

Two operational facts fall out. Cached tokens mostly do not count against token rate limits, which makes a high hit rate a capacity multiplier as well as a cost saving. And caching is frequently per replica, so Fireworks ships affinity headers and warns that sampling during rollouts degrades hit rates. The reported fields disagree about what they include: Anthropic's `input_tokens` "only represents tokens that appear after your last cache breakpoint," so mapping it to `prompt_tokens` understates by the entire cached prefix. Bedrock sits with Anthropic rather than OpenAI here, reporting a disjoint set with a total: `inputTokens`, `outputTokens`, `totalTokens`, `cacheReadInputTokens`, `cacheWriteInputTokens` and `cacheDetails` of `{ inputTokens, ttl }` (docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_ConverseStream.html, read 2026-10-28). That closes a gap the local evidence never filled, and a gateway assuming OpenAI's inclusive `prompt_tokens` shape will double-count every cached token on Bedrock. Anthropic's cache diagnostics no longer need a beta header either; the `diagnostics` object itself is the opt-in.

### 5.6 Structured output: the gap between valid JSON and a guarantee

Three levels exist and they are indistinguishable in a successful response. JSON mode returns parseable JSON with no schema promise: OpenAI states `json_object` "will not guarantee the output matches any specific schema, only that it is valid and parses without errors," and Mistral requires you to instruct JSON yourself on top of it. Schema mode with constrained decoding is a real guarantee: OpenAI's Structured Outputs "ensures the model will always generate responses that adhere to your supplied JSON Schema," and Anthropic names the mechanism, "Structured outputs guarantee schema-compliant responses through constrained decoding." Groq is the only vendor publishing the split per model.

Two asymmetries matter more than the levels. Refusal preempts the schema: OpenAI documents that "a refusal does not necessarily follow the schema you have supplied," which is why a separate `refusal` field exists. And capability is often conditional exactly when you need it: Cohere's structured output is unsupported with `documents` or `tools`, so it is mutually exclusive with RAG and with tool use, and ERNIE X1 Turbo has no `json_schema`. Add Anthropic's layer ignoring `strict`, and a gateway accepting that flag has promised something it may not be able to deliver. Confidence medium-high, because the levels are documented but failure rates are not.

### 5.7 Error envelopes: the status code is the least informative field

**Table 6. One condition, six encodings.**

| Condition | Vendor | Status | Discriminator |
|---|---|---|---|
| Overloaded | Anthropic | 529 | `overloaded_error` |
| Overloaded | OpenAI | 503 | `server_is_overloaded` |
| Overloaded | Bedrock | 503 or 424 | `serviceUnavailableException` / `modelStreamErrorException` |
| Rate rising too fast | OpenAI | 429 | `slow_down`, which "can occur even when your traffic is within" normal limits |
| Spend cap exhausted | Anthropic | 429 | No `retry-after`, fails until access resumes |
| Spend limits | OpenAI | 429 | Four codes splitting one condition: `credit_balance_exhausted`, `organization_spend_limit_exceeded`, `project_spend_limit_exceeded`, `organization_usage_limit_exceeded` |
| Daily quota versus rate | Gemini | 429 | `quota_exceeded` versus `rate_limit_exceeded` |
| Balance | DeepSeek | 402 | Balance condition |

The OpenAI row is the one most often mishandled, because it used to be a single branch. `insufficient_quota` survives as the umbrella `error.type`, but billing failures now split into four distinct `code` values: `credit_balance_exhausted`, `organization_spend_limit_exceeded`, `project_spend_limit_exceeded` and `organization_usage_limit_exceeded` (platform.openai.com/docs/guides/error-codes, read 2026-10-28). Those need four different responses, since topping up a balance, raising an org cap, raising a project cap, and waiting out a usage window are not the same remedy. A gateway branching on `insufficient_quota` alone now under-reports and will tell an operator to add credit when the real fix is a project limit.

OpenRouter shows the same fragmentation from the aggregator side, and its strings changed. The 402 `error_type` is `payment_required`, the in-flight case appears as `error.metadata.limit_source` equal to `openrouter_in_flight_budget`, and budget rejections are 403. The older `in_flight_budget_exhausted` and `insufficient_credits` reason strings no longer appear in the docs, so code keyed on them is keyed on nothing.

The envelope shape diverges as much as the status. Mistral uses `{"object":"error",...}` so a parser looking for `.error.message` finds nothing, and adds a second FastAPI-style body with `detail[]` on 422. IBM uses `{trace, errors:[{code,message,moreinfo}]}` with `moreinfo` a documentation link. Zhipu nests a quoted integer as `{"error":{"code":"1302"}}`. MiniMax's native route returns HTTP 200 with `base_resp.status_code`, so status-only handling reports failures as successes. Meta returns an empty body on an unknown path and `text/plain` on a Files validation error, so not every 4xx is JSON. AWS uses named exceptions including two distinct 424s that a naive mapper flattens to 500.

Google's split is the most commonly misread. The canonical `google.rpc.Status` carries a numeric enum `code`, but in JSON form "the most important difference is that the `code` field in the JSON is an HTTP status code, not the direct value of `google.rpc.Status.code`," with the enum name in `status` (cloud.google.com/apis/design/errors, read 2026-10-28). Gemini layers a snake_case vocabulary on top and commits to total coverage: any unlisted code "falls back to the `snake_case` version of the HTTP status." A gateway that maps status to status and drops the code preserves the shape and destroys the meaning, and its retry logic will retry spend-cap failures forever. Confidence high.

### 5.8 Versioning and deprecation: the axis that changes behavior with no code change

Three channels are live: the universal `/v1` path, which is stable and nearly meaningless; date headers, which are strict contracts; and date query parameters, the legacy pattern Azure is leaving.

Anthropic's date header is the strictest channel here. Version `2026-10-21` did not merely reject non-default sampling values on new models, which was the older behavior; it removed `temperature`, `top_p` and `top_k` for every model, so a request sending one gets a 400 even when the value equals the default, and anyone needing non-default sampling on an older model must stay pinned to `2023-06-01`. That is a vendor choosing to break clients loudly rather than ignore parameters, the opposite of Azure's approach, where an `api-version` does not freeze behavior because the service uses your date or the most recent one before it.

Floating aliases are more dangerous because nothing in the client changes when the model does. Gemini defines four tiers where "Latest" models "will get hot-swapped with every new release" and Preview models are deprecated "with at least 2 weeks notice." OpenAI publishes numeric notice floors, six months for GA, three for specialized variants, two weeks for previews, and exposes `shutdown_date` on the models list, the most useful deprecation affordance found anywhere. Bedrock attaches four dated transitions per model.

Judging who breaks clients most, confidence medium because this is inference from published policy rather than measured outage rates: Anthropic breaks most per bump; Gemini has the most churn in model identity, now compounded by a surface migration; OpenAI has the most predictable process across the widest surface; and Bedrock and Azure break at the platform layer, since a region-prefixed id cannot be used from another region and an Azure deployment name is not a model id at all.

### 5.9 Rate limiting and quota: five scopes and four reset formats

Scope is the first surprise. OpenAI defines limits at organization and project level, not per user. Gemini applies them per project rather than per key. Bedrock limits per region and per model, and its two hosts hold separate quotas "even when calling the same underlying model." Azure limits per deployment type per region, and "Quota in East US doesn't carry over to West Europe." OpenRouter does not limit per key at all. DeepSeek limits concurrency rather than requests or tokens.

Header conventions are close enough to be dangerous. Groq uses OpenAI's names, but its request limit "always refers to Requests Per Day," so reading it with a per-minute assumption mis-plans by a factor of 1,440. Anthropic's token headers "display the values for the most restrictive limit currently in effect." Gemini documents no limit headers at all. Reset formats split four ways: durations at OpenAI, RFC 3339 timestamps at Anthropic, `retry-after-ms` at Azure, and `RetryInfo.retryDelay` in the body at Google.

Provisioned throughput is a second product everywhere it exists. Azure bills per PTU-hour "regardless of the number of tokens consumed," warns that "Having PTU quota doesn't guarantee that capacity is available," and routes spillover to a standard deployment via `x-ms-spillover-deployment`, which turns the limit boundary into a routing decision. Databricks reserves output tokens from `max_tokens` up front, so a request that would exceed the output limit gets an immediate 429 rather than a truncation.

### 5.10 Model discovery and naming: six grammars and several untrustworthy lists

**The best lists carry lifecycle and capability data:** Anthropic returns `lifecycle`, `retires_at` and a capability tree; Bedrock returns four dated transitions plus modalities; Mistral returns `capabilities`, `aliases` and `deprecation_replacement_model`; Baidu Qianfan returns context and output limits, modalities, pricing, `status`, `shutdown_time`, `tpm` and `rpm`. The worst validate only the credential: Gemini's list "only validate[s] that your API key exists," so a 200 does not imply inference works, and Azure discovery is a control-plane operation because deployment names are operator-chosen.

Two lists actively mislead. Volcano Ark's `GET /api/v3/models` is uncurated, with roughly 523 entries including internal test models, so the documented path is control-plane `ListFoundationModels`. And OpenRouter's catalogue is unauthenticated, so a 200 says nothing about your key.

Six naming grammars coexist: dated snapshots where the date is identity, floating aliases where a swap breaks with two weeks notice, namespaced ids, deployment names where the field is not a model id at all, region-qualified ids, and variant suffixes where a router variant and a catalogue variant behave differently. Two decisions stand out. Meta's `-contributor` suffix encodes data-retention consent in the model id, and ByteDance switches from plaintext to encrypted reasoning replay between `doubao-seed-2-0-lite-260428` and later models, so upgrading the model changes the replay contract with no other signal.

### 5.11 Pricing: the multipliers matter more than the base rate

Base per-token prices are the least interesting number because the multipliers stack. Cache reads range from 50 percent off at Groq and Fireworks to 0.025x at Anthropic's newest tier, with 0.1x the common case and 0.05x on Opus 5.5, Sonnet 5.5 and Fable 5.1. Cache writes are the only structure where a request can cost more than an uncached one, at 1.25x for a five-minute Anthropic TTL up to 2.5x for a twelve-hour OpenAI TTL, and write charges are the item most often omitted from a relay's cost model. Batch discounts cluster at 50 percent but diverge on stacking: Anthropic's "can stack" with caching while Groq's is explicitly non-stacking. Long-context premiums are structured differently again: OpenAI splits its table into short and long context columns, xAI doubles the whole request above 200,000 prompt tokens, and Anthropic bands by prompt size with a 5x step at 100,000 tokens on Haiku 5.5.

Media pricing adds at least three more meters, since image is per megapixel with rounding up, video is per output second, synthesis is per character, and transcription is per minute or hour. Aggregator economics are separate again: OpenRouter charges no inference markup and earns on credits at 5.5 percent with a minimum plus 5 percent of list-price cost on BYOK above a monthly allowance, while cloud wrappers keep per-token prices identical across endpoints and attribute cost through IAM principal and application inference profiles.

### 5.12 Media modalities: a preview of why they do not unify

Every media surface breaks at least one chat assumption, and section 7 treats the five families as distinct shapes.

---

## The OpenAI shape as an industry default

**How far it reaches.** Every text surface in this survey except Replicate's primary interface accepts an OpenAI-shaped chat request: sixteen hosted surfaces and all seven self-hosted servers, plus the first-party vendors shipping it natively or as a documented side door, plus Google's and Anthropic's compatibility routes. What it does not cover is Replicate, whose product is a versioned job system, and Lepton, whose product is a GPU endpoint deployer.

Reach is not fidelity. Groq rejects several standard fields outright and Ollama rejects four, Predibase caps multiple completions at 2 and stop sequences at 4 while defaulting max tokens to 20, and Cloudflare offers Responses only for two models on a non-streaming path.

**The extensions that became de facto standard, and who made them standard.** Aggregators built these because a router has to reconcile vendors, and reconciling produces a specification.

- **A structured provider-preference object.** OpenRouter's block, with ordering, fallback toggles, allow and deny lists and percentile latency preferences, has no OpenAI equivalent and is now the reference design for routing. Hugging Face's model-ID suffix is a weaker variant.
- **A fallback model list in the body,** so a request survives one model being unavailable rather than one provider.
- **Reasoning as a structured object rather than a string.** OpenRouter's details array, Cerebras' format controls and Groq's flags all express what OpenAI's single nested object cannot. It is a replay contract, not a conversion one: each entry carries a vendor `format` tag such as `anthropic-claude-v1`, so reasoning is replayed verbatim or dropped.
- **`reasoning_content` as a field name.** Not OpenAI at all. Popularized by DeepSeek and now the de facto output field on vLLM, SGLang, llama.cpp, Fireworks and one of Together's two variants.
- **Cache control fields and usage cost fields.** Anthropic's breakpoint blocks and OpenAI's breakpoint field are now translated into each other by OpenRouter, and cost, a BYOK flag and a nested breakdown exist nowhere in OpenAI's spec and are purely an aggregator invention.
- **Server-executed tools under a vendor namespace, and a per-request zero-data-retention flag,** including the BYOK interaction where a key does not exempt a provider from retention policy unless ZDR is declared on the key.

**Where the shape breaks down.** Streaming envelopes differ before anything semantic does: OpenAI's final usage chunk has empty choices, OpenRouter's has a non-empty array holding a content-free delta that repeats the finish reason, and Fireworks puts usage in the finish chunk inside a field the SDK type does not declare. Error semantics differ in direction and retryability: OpenAI 400s on unknown parameters, OpenRouter ignores them, Fireworks silently truncates max tokens, and OpenRouter's in-flight-budget case is a 402 while its budget rejections are 403.

Tool calling has three incompatible implementations behind one field name: native pass-through, transform to provider format, and prompt-injection emulation, so tool reliability cannot be inferred from `tools` appearing in a supported-parameters list. Model identity has four conventions with no common ground, and on at least five platforms a gateway cannot resolve which model a request names from the request field. Finish reasons leak across shapes: xAI returns `end_turn` inside an OpenAI-shaped field, Mistral returns `error` as a finish reason, and Bedrock normalizes to a closed set where the vendor value never survives.

**What is provably lost when a native surface is flattened.** Anthropic's layer never returns thinking, never caches, ignores `strict` and `response_format`, reports empty token-detail objects, and hoists mid-conversation system messages into a single field so their meaning changes. Gemini's route loses thought signatures, `alt=sse` behavior, `continuationToken`, safety settings and cache handles. Bedrock's Converse loses any parameter it did not model unless you name it in `additionalModelRequestFields`. And Google's own migration shows the ceiling: moving to the Interactions API drops explicit caching entirely rather than porting it.

The honest summary, confidence high: the OpenAI request body is a usable lowest common denominator for text in, text out, with streaming. Everything above that line, meaning tools, reasoning, caching, structured output, usage accounting and errors, is where vendors diverge.

---

## Media and realtime: five shapes, not one

A unified chat surface was achievable because every vendor computes the same thing from the same inputs and returns the same kind of object. Media does not have that property: the five families differ in where the time goes and who holds state.

**Image is a blocking call, so the response is the result.** OpenAI's `/v1/images/generations` and `/edits` are synchronous, output is always base64 because `response_format: url` is unsupported on GPT Image models, and optional SSE partial-image streaming is a latency trick rather than a job model. That canonical shape is shallowly imitated: many gateways copy DALL-E's `response_format: url` and then break on current models. Black Forest Labs runs the opposite design, a clean async job returning `{id, polling_url}`, where the returned URL must be used because the global endpoint routes across clusters. Midjourney proves the shape is not universal: no public API, and its guidelines forbid automating interactions, so every wrapper automates a subscriber account and is bannable.

**Video is a job essentially everywhere, and only the handle and completion signal vary.** Sora, the shape many gateways coded against, is gone: shut down 2026-09-24 with no replacement. Google Veo has diverged between the Gemini API and Vertex: the Gemini API killed Veo 3.1 previews on 2026-10-22, while Vertex still serves `veo-3.1-generate-001` as GA, and Vertex's polling is a `:fetchPredictOperation` POST with the operation name in the body, which breaks naive pollers. Results carry `raiMediaFilteredCount`, so a request can succeed with zero videos. Runway pins `X-Runway-Version: 2024-11-06` across two years of model churn, versioning the envelope rather than the models.

**Synthesis has no result object, so its contract is content type and framing.** Almost every vendor is a POST returning a byte stream, and what separates them is whether limits count characters or bytes, whether input is text or SSML, and whether billing starts at submit or success. Azure needs four headers including a required `User-Agent`, takes SSML rather than a prompt, and splits into two lifecycles because real-time caps at 10 minutes while long content needs async batch. Google returns base64 inside JSON rather than bytes, and Chirp 3 caps a request at 5,000 total bytes. Deepgram fuses voice and language into one string, so `aura-2-thalia-en` is Thalia in English and voice lookup is model lookup. Cartesia accepts partial transcripts on a `context_id` with `continue: true` to hold prosody across parts, the primitive for piping LLM tokens to speech without waiting for a sentence boundary. And OpenAI's speech surface is migrating shape: `tts-1`, `tts-1-hd` and `gpt-4o-mini-tts` shut down 2027-01-06 into a realtime model the docs say "isn't a supported model for `/v1/audio/speech`," so batch synthesis closes in favor of a socket inside ten weeks.

**Transcription is bimodal: a job returning a document, or a socket returning partials.** OpenAI has the weakest batch shape: multipart `file` capped at 25 MB, no URL input, no job handle and no idempotency key. Amazon Transcribe is strictest, requiring audio already in S3, with a caller-chosen account-unique job name so resubmission conflicts rather than starting a second job. Deepgram bundles timestamps, segmentation and diarization into one call where Azure prices diarization as an add-on. Gladia publishes the only full retry decision table, and its advice is the right model for everyone: re-POSTing "does not make the transcript arrive faster: it creates a new job that competes for the same capacity and can worsen the backlog."

**Realtime inverts everything, and there are three incompatible session shapes.** There is no request and no response, only a stateful socket where output arrives interleaved with input and the vendor decides when you finished speaking. OpenAI runs WebRTC in browsers, WebSocket server-to-server, and SIP for telephony, with turn detection as the core surface and PCM at 24 kHz alongside G.711 µ-law and A-law so a SIP integration can skip transcoding. Gemini Live takes its key as a `?key=` query parameter and forbids configuration changes while the connection is open. ElevenLabs Agents is architecturally different again: you connect to a configured agent whose prompt, voice, tools and turn-taking live in vendor-side config, so the request is closer to "connect me to agent X" than "model plus prompt plus audio."

**Why a unified media surface is harder than a unified chat surface.** Three structural reasons. The state owner differs: chat is stateless or optionally server-held, so a gateway can force statelessness and stay a proxy, but video and batch transcription hold job state that outlives the connection, and realtime holds state both sides mutate concurrently. The success criterion is not one thing: image success is bytes, synthesis success is a complete stream whose failure mode is truncation that looks fine at the transport layer, job success is a terminal status that can report zero outputs, and realtime has no success event at all. And the cost meter does not convert. A gateway can unify the first four families behind submit-then-fetch, because a job handle plus a terminal document covers image, video, batch transcription and synthesis with a synchronous fake. Realtime cannot, and the conclusion, confidence high because the three session shapes share no vocabulary, is that a media gateway needs two abstractions: a job model and a session model.

---

## What the structure implies for a gateway

Each judgment carries a confidence level and its reason.

**Build the translation layer around five semantic objects, not five protocols.** Confidence high, because the twelve axes reduce to five things that must survive a hop: reasoning artifacts, cache identity, usage accounting, error classification, and tool-call correlation identity. A gateway modelling those five can add a dialect in a day; one modelling protocols rewrites its core per vendor, which is what happened in this workspace, where a single `Openai` protocol variant left Anthropic support unbuilt for two months.

**Treat reasoning as opaque provider-owned state, replayed by position and stored outside the message record.** Confidence high, because Anthropic rejects reconstructed blocks with a positioned 400, Google requires exact resend with `missing_thought_signature`, and ByteDance requires byte-identical `encrypted_content`. Store the assistant's whole native block list as engine-owned data keyed by provider and model, replay verbatim when those match, and strip after any edited position rather than editing inside it. The rejected alternative, a stateful gateway keyed by tool-call id, breaks statelessness, fails across restarts, and blocks replaying runs from logs. And note what does not solve it: OpenRouter's `reasoning_details` array looks like portable currency but is a replay contract.

**Assume any history rewriting destroys cache hits and possibly reasoning validity.** Confidence high on mechanism, medium on magnitude, because magnitude is workload-dependent and unpublished. Tool definitions, `tool_choice`, thinking budget, `speed`, and even an image appearing anywhere in a prompt all change Anthropic's cache key, and the same edits that miss cache also invalidate thinking blocks.

**Never map an upstream status to a downstream status without carrying the code.** Confidence high, because every discriminator is a different field: OpenAI's four 429 spend codes plus `slow_down`, Anthropic's 529 and its `retry-after`-less spend 429, Gemini's `quota_exceeded` versus `rate_limit_exceeded`, and Zhipu's `1302` and `1305` both arriving as 429 but needing opposite responses. Emit a normalized class and the native code together, the way OpenRouter emits both `finish_reason` and `native_finish_reason`. The four-way OpenAI split sharpens this: a single `insufficient_quota` branch conflates topping up a balance with raising a project cap.

**Do not promise a schema guarantee you cannot enforce at every backend.** Confidence medium-high, because guarantee levels are documented but failure rates are not. Anthropic's layer ignores `strict`, Cohere's structured output is incompatible with tools and documents, and ERNIE X1 Turbo has no `json_schema`. Validate at the boundary and either reject or report the downgrade rather than returning 200.

**Model the cost meters separately.** Confidence high, because the units do not convert and a blended number is misleading. Anthropic's `input_tokens` excludes everything before the last cache breakpoint, so mapping it to `prompt_tokens` understates by the cached prefix, while Bedrock's disjoint buckets mean the same mapping double-counts.

**Store region and key tier as configuration, not as a suffix on the key.** Confidence high, because Moonshot, Alibaba and Tencent all return an authentication error for a region mismatch with no hint that region is the problem. Add Meta's `-contributor` suffix and Alibaba's `sk-sp-` keys needing a dedicated base URL, and credential identity is a tuple.

**Construct base URLs from documentation rather than appending paths to a versioned base.** Confidence high, and this is a live bug rather than a caution, since a client configured with the versioned Gemini base and then appending the documented path requests `/v1beta/v1beta/models`, which a live probe confirmed returns 404.

**Plan for vendor exit as a normal event, and do not make the gateway the load-bearing abstraction for media.** Confidence high on the first, because 01.AI closed self-serve with a month of notice, Tencent discontinued a platform, OpenAI removed a modality, and PlayHT stopped resolving. Confidence medium on the second, because it is a judgment about effort allocation: the five media families share no success criterion, completion signal or cost unit, so two abstractions is the realistic ceiling.

---

---

## Uncertainties and open questions

**Two verification notes landed during drafting, and they changed real claims.** The first resolved four contradictions with live probes where a probe was possible: it corrected Anthropic's auth model to Bearer with `x-api-key` as a legacy fallback, confirmed both Gemini key transports work while noting key *type* is now independent of transport, resolved Gemini's base URL as a single `/v1beta` segment with a 404 probe, and struck the `max_tokens` default behavior as a client policy rather than a vendor fact.

The second graded 20 transcript claims, and its corrections are integrated above rather than listed here. In summary: Anthropic's compatibility layer silently ignores a top-level `cache_control` rather than rejecting it, so a client sending cache markers there gets a silently uncached request. `prefix_mismatch_behavior` is a field at `thinking.block_binding`, not a header. OpenRouter's `reasoning_details` is a replay contract, not a conversion contract. OpenAI's `encrypted_content` now defaults on. Google's default surface is the Interactions API. Bedrock's usage field names are now known. And Anthropic's thinking control is a three-way split.

**Still unverified, so marked rather than stated as fact.** Vertex AI Express Mode auth (the page timed out), AI21's stop reasons and pricing, NVIDIA's platform-level `tool_choice`, watsonx rate limits and pricing, OCI's `finishReason` enumeration, Mistral's rate-limit header names, Cerebras' cache mechanics, SambaNova's separate reasoning field, whether `gpt-transcribe` returns word timestamps, Kling's billing on failed tasks, and Replicate's OpenAI shim base URL.

**Vendor contradictions left standing.** Google publishes both a 64 and a 128 character tool-name limit on live pages, and two live pages disagree on whether Chirp 3 supports `Recognize`. IBM's Node SDK says `tool_choice: "auto"` is unsupported while its capability page demonstrates it. Alibaba's compatibility table lists Beijing under the Singapore host against its own migration note.

**The alternative I set aside.** The obvious competing explanation is that the fragmentation is transitional and a de facto standard will emerge the way SQL did. I set it aside because the compatibility convention has been stable for roughly three years while the divergences have grown, and because the party best positioned to standardize, OpenAI, is the one whose current protocol is not the standard.

**What would change the main judgment.** The through-line claim is that request bodies converged and semantics did not. That would be falsified by evidence that most high-volume production traffic is text-in, text-out with streaming, which would make the semantic axis real but commercially marginal. I have no traffic data, so read this as a statement about surface complexity rather than about where the volume sits.

---

## References

Vendor documentation read 2026-10-28. Primary sources.

**First-party labs.** platform.openai.com/docs/api-reference/{chat,responses,authentication}, /guides/{prompt-caching,conversation-state,structured-outputs,rate-limits,error-codes,moderation,batch,realtime,voice-websockets,text-to-speech,speech-to-text,tools-tool-search}, /deprecations, /pricing. docs.anthropic.com/en/api/{messages,getting-started,versioning,errors,handling-stop-reasons,openai-sdk} and /docs/build-with-claude/{thinking,prompt-caching,batch-processing,structured-outputs}; platform.claude.com/docs/en/{about-claude/pricing,build-with-claude/prompt-caching,build-with-claude/cache-diagnostics,build-with-claude/preserved-thinking,api/migration-2026-10-21,api/rate-limits,agents-and-tools/tool-use/fine-grained-tool-streaming}. ai.google.dev/api/{generate-content,models,caching} and /gemini-api/docs/{api-key,openai,caching,thinking,thought-signatures,models,pricing,safety-settings,api-errors,troubleshooting,live-api/session-management,veo,omni}. docs.x.ai/docs, dev.meta.ai/docs, learn.microsoft.com/azure/foundry, docs.mistral.ai.

**Cloud platforms.** docs.aws.amazon.com/bedrock/latest (Converse and ConverseStream API references, endpoints, API keys, conversation inference, structured output, prompt caching, guardrails, provisioned throughput, ListFoundationModels). learn.microsoft.com/azure/ai-foundry/openai/{reference,api-version-lifecycle,quotas-limits} and /azure/ai-foundry/how-to/develop/sdk-overview. cloud.google.com/vertex-ai/docs/reference, docs.cloud.google.com/vertex-ai/generative-ai/docs/{learn/locations,partner-models}, platform.claude.com/docs/en/build-with-claude/claude-on-vertex-ai, cloud.google.com/apis/design/errors. docs.cohere.com, docs.ai21.com, docs.api.nvidia.com/nim/reference, cloud.ibm.com/apidocs/watsonx-ai, docs.databricks.com foundation-model-apis, docs.oracle.com OCI Generative AI Inference.

**China and APAC.** api-docs.deepseek.com, platform.moonshot.ai/docs, kimi.ai/help/kimi-api/api-troubleshooting, alibabacloud.com/help/model-studio/compatibility-of-openai-with-dashscope, docs.bigmodel.cn, platform.minimax.io/docs, ark.volcengine.com, cloud.baidu.com/doc/qianfan-api, platform.stepfun.com/docs, xfyun.cn Spark HTTP documents, tencentcloud.com/announce/detail/101147, cloud.tencent.com/document/product/1729/111007, ithome.com/0/987/951.htm, 01.ai.

**Aggregators and platforms.** openrouter.ai/docs (api-reference, api/api-reference/responses/create-a-response, errors, provider-selection, limits, faq, prompt-caching, model-routing, byok, reasoning-tokens, features/prompt-caching, server-tools), docs.together.ai, docs.fireworks.ai, console.groq.com/docs, inference-docs.cerebras.ai, docs.sambanova.ai, replicate.com/docs, huggingface.co/docs (inference-providers, inference-endpoints, text-generation-inference), developers.cloudflare.com/workers-ai, docs.baseten.co, modal.com/docs, docs.predibase.com, docs.novita.ai, docs.deepinfra.com, docs.tokenfactory.nebius.com, nvidia.com/dgx-cloud/lepton, docs.perplexity.ai.

**Self-hosted baseline.** docs.vllm.ai (openai_compatible_server, serve CLI reference, chat protocol source), docs.sglang.io, huggingface.co/docs/text-generation-inference, llama.cpp tools/server README, docs.ollama.com/openai-compatibility, lmstudio.ai docs, localai.io docs.

**Media and realtime.** developers.openai.com/api/reference/resources/images and /api/docs/{guides,deprecations}, ai.google.dev/gemini-api/docs/{pricing,veo,omni}, docs.stability.ai, docs.midjourney.com Community Guidelines and Terms, docs.bfl.ai, developer.ideogram.ai, recraft.ai/docs, docs.leonardo.ai, docs.dev.runwayml.com, docs.agents.lumalabs.ai, kling.ai/document-api/pricing, elevenlabs.io/pricing, docs.cartesia.ai, developers.deepgram.com and deepgram.com/pricing, learn.microsoft.com/azure/ai-services/speech-service, aws.amazon.com/{polly,transcribe}/pricing, cloud.google.com/{text-to-speech,speech-to-text}/pricing, assemblyai.com, gladia.io, rev.ai, speechmatics.com, soniox.com, docs.audiostack.ai PlayHT deprecation notice, dev.hume.ai.

**Local workspace evidence, with age.** `everruns/dokuman/04-models-and-providers.md` (2026-09-17, 5.9 weeks). `everruns/knowledge/foundations/{llm-drivers,providers}.md` and `security/usage-tracking.md` (2026-09-16, 6 weeks). `everruns/docs/providers/*.md` (2026-09-11, 6.7 weeks). `runvil/vibe/2026-09/2026-09-14-2-provider-model-sheets.md` (6.3 weeks). `runvil-design/research/2026-09/2026-09-20-model-tool-differences.md` (5.4 weeks). `runvil-design/research/2026-09-30-compaction-exhaustion-signals.md` (4 weeks). `runvil-design/research/2026-10-14-harness-mcp-dynamic-tools.md` (2 weeks). `runvil-design/design-prev/design-gateway-residue.md` (2026-07-26, 13 weeks, likeliest to have drifted).

**Chat transcripts, with grading.** `f3f6b583` (2026-10-24) for Anthropic Messages in runvil, cited to that day's fetches and code-grounded for the runvil side. `091b79d2` (2026-10-16) for the native-versus-compat distinction. `9947d040` (2026-10-14) for OpenAI detail, partly from memory and self-flagged. `a74ca56c` and `b44ccb1c` (2026-09-14) for Qwen's reseller catalogue and the one raw Anthropic model-list payload. `ca8d6235` (2026-09-20) for OpenRouter's translation rules.

**Verification.** `2026-10-28-inference-api-survey-verification.md`, which resolved four contradictions with live probes where possible. Recorded probes: `GET /v1beta/models` returned 400 `API_KEY_INVALID`, `GET /v1beta/v1beta/models` returned 404, Gemini with neither credential returned 403 `PERMISSION_DENIED`, Anthropic with `x-api-key` and with `Authorization: Bearer` returned the identical `invalid x-api-key` error, and Anthropic with neither returned "x-api-key header is required." And `2026-10-28-inference-api-survey-verification-b.md`, which graded 20 transcript claims against live documentation.

*2026-10-28 12:40 - DeepSeek V4.1 Flash 1017*
