---
produced: 2026-11-22
title: Mapping vendor-specific model API features onto Runvil's universal inference API - 105 features from Anthropic, OpenAI, Google Gemini, xAI, DeepSeek, Mistral, Kimi, Qwen, GLM, and local servers, with OpenRouter as the first reference - reasoning data sent back to the vendor, prompt caching, tools, structured output, errors, and usage
---

# Mapping vendor features onto Runvil's universal inference API: the Gateway handles most of them and one opaque field handles the rest

- Report type: analytical, with recommendations
- Written for: the Runvil project owner, and for whoever does two work items in Runvil's lesson plan of 2026-11-17: item 2, vendor protocols in the Gateway, and item 3, the standalone gateway client
- Sources: documentation, published specifications, source code, and recorded traffic, all read on 2026-11-22

## Executive summary

**Background.** Runvil runs prompts, which are Markdown files with Lua code. The Engine executes a prompt, the gateway client sends each model request, and the Gateway is the server that forwards the request to a model vendor or to a local model. Today the Gateway forwards every request in one format, OpenAI's chat completions, so it reaches local models and the vendors that accept that format. Runvil plans one universal inference API between the client and the Gateway: chat completions plus a few optional extra fields. The Gateway would translate each request into the vendor's own protocol. The goal is that a prompt never knows which vendor answers it, and that code outside the Gateway knows as little as possible.

**What was examined.** The mapping covers 105 vendor-specific features. For each one it answers: which parts of Runvil would need vendor-specific code to support it? The first reference for each feature was OpenRouter, a commercial service that already puts one OpenAI-style API in front of many vendors. Open-source gateways and libraries came next, and then the vendor's own documentation.

**The answer.** The Gateway alone can handle 65 of the 105 features. One extra field on a message, the opaque field, handles the rest of what Runvil needs. For all 105, a prompt can stay the same whichever vendor answers it.

- **65 features need only the Gateway.** It converts the request and the reply, and the client, the Engine, and prompts contain vendor-neutral code only. This covers reasoning settings, prompt caching, tool definitions, finish reasons, errors, and token counts.
- **25 features need the opaque field.** Some vendors send data in a reply that they expect back unchanged in the next request, such as Anthropic's signed thinking blocks and OpenAI's encrypted reasoning. The opaque field is an optional field on an assistant message for that data. The Gateway fills it, the client and the Engine store it and send it back unread, and a prompt's Lua code never sees it. All 25 features use the same field, so the client and the Engine change once.
- **10 features need a little compatibility code**, which changes or repairs a request so that one vendor accepts it, for example by removing a temperature that a model rejects. All ten pieces can be in the Gateway.
- **5 features would need a separate vendor-specific path.** In four the vendor keeps files, a conversation, or a running job on its own servers, and the fifth is a WebSocket connection in place of HTTP requests. All five are features Runvil does not use today.

**The planned work needs 75 of the 105: 52 of the Gateway-only features, 17 of the opaque-field features, and 6 of the compatibility-code features.** The planned work is a tool loop (the model asks for a tool, the program runs it, and the model continues) with reasoning on Anthropic, OpenAI, and Google models, on vendors with OpenAI-style APIs, and on local servers. The other 30 are features Runvil does not use today, such as code execution on the vendor's servers, batch jobs, and conversations stored by the vendor. Confidence in these counts: medium - the vendors document their rules and open-source translators show that the conversions work, but the opaque field exists only as a design.

**What OpenRouter's design shows.** OpenRouter has its own version of the opaque field, an array named `reasoning_details` on the assistant message.

- **Its rule is the one Runvil wants, and its format loses information.** The rule: the caller stores a reply's reasoning data unread and sends all of it back, in order. The format keeps reasoning entries, text, and tool calls in three separate fields, so it loses where each piece of reasoning was among the text and the tool calls. Anthropic and Google verify a signature on reasoning that is sent back, and that check depends on the order. Every open-source gateway that copies the three-field format loses the order too, and OpenRouter's own client library needed nine dated fixes in 2026 to its handling of this data.
- **It gives a caller one way to ask for things, and that works.** A caller sets one reasoning-effort value, one cache marker, and one tool format, and reads one of five finish reasons and one of 27 error types. OpenRouter converts each to and from the vendor's own form.
- **It cannot hide which provider produced data that has to go back.** A provider is a company that serves a model, and one model can have several. Anthropic, OpenAI, and Google accept signed or encrypted reasoning only from the vendor account (an organization or a cloud project) that produced it. OpenRouter writes the provider into each encrypted value, keeps a conversation on one provider where it can (it calls this sticky routing), and advises callers to name the provider themselves.
- **It has three routes, and ten features work on only one or two of them.** Besides chat completions it has a route in Anthropic's Messages format and one in OpenAI's Responses format. A caller that wants vendor compaction or citations on supplied documents, for example, has to pick the route by vendor. Runvil plans one route, and the opaque field is what lets that one route hold what OpenRouter's other two hold.

**What should change in Runvil's plan.** Runvil's design of 2026-11-17 already has the opaque field: it holds each reply's ordered list of vendor blocks, labeled with the protocol and the model.

1. **The opaque field should record its endpoint, serve chat completions vendors too, hold every kind of block, and arrive once.**
    - The label should also name the endpoint, which is one provider's offering of one model under one account. Anthropic, OpenAI, and Google each tie reasoning data to the account that produced it.
    - Vendors that speak chat completions should use the field too. DeepSeek, Kimi, Qwen, GLM, and MiniMax want their reasoning text back, and by DeepSeek's documented rule a tool loop fails today without it.
    - The field should hold every kind of block a vendor returns, such as the results of tools that the vendor runs, as well as reasoning.
    - The Gateway should send the field whole, once, in the last chunk of a streamed reply, so that the client copies one value.
2. **The Gateway needs a translator for OpenAI's Responses API early.** From GPT-5.4, OpenAI's chat completions API does not support tool calling with reasoning on, and on five GPT-6 models it does not support tool calling at all. Confidence: high - OpenAI's migration pages, fetched twice.
3. **The opaque field should get a name of Runvil's own, and three other fields should use names that are already standard.** OpenRouter's name, `reasoning_details`, would promise OpenRouter's format, which loses block order. The three standard names are `reasoning_effort`, `prompt_tokens_details.cache_write_tokens`, and `annotations` with `url_citation`. All three are in OpenAI's own chat completions reference.
4. **Today's Gateway has four defects that should be fixed before any vendor protocol is added.** For a remote model the Gateway loses the token counts of a streamed reply, replaces the vendor's error text with a fixed phrase, skips an error that arrives inside a stream, and waits on a silent stream indefinitely (section 7).
5. **Settings in a vendor's own words belong in the Gateway's configuration.** OpenRouter's callers write Anthropic's beta header values, Gemini's safety thresholds, and raw vendor settings in an object named `provider.options`. In Runvil the operator, the person who configures the Gateway, writes them in the Gateway's configuration for a model, and a prompt names only a model role such as `strong`.

**How solid the evidence is.** Every statement comes from documentation, a published specification, source code, or traffic that others recorded. For 85 of the 105 features OpenRouter's own published material says how OpenRouter handles the feature. For 7 that was inferred, and for 13 the account comes from an open-source project or from the vendor's documentation, because OpenRouter's material is silent. In a second round, separate checkers fetched the sources again for 128 of the claims the conclusions depend on most: 97 held, 22 held with a correction, 4 did not hold, 3 were conflicts between sources, and 2 were overstated. The feature sections include the corrections (section 11).

**Decisions for the project owner.** Section 8 ends with ten decisions, each with a recommendation. The first four put changes 1 to 3 as questions and add which translator comes after Anthropic's (recommended: OpenAI's Responses API, ahead of Gemini). The other six: what the standalone client does when it talks to OpenRouter directly, what happens when a prompt asks for something a model cannot do, which vendor-neutral names to add now, whether Lua keeps the field that shows a prompt its provider, whether the Gateway also accepts requests in Anthropic's Messages format, and who continues a reply that a vendor stops partway.

## Contents

- Terms used in this report
- 1. Each feature is judged by which parts of Runvil would need vendor-specific code
- 2. Of 105 features, 65 need only the Gateway, 25 need the opaque field, 10 need compatibility code, and 5 need a separate vendor path
- 3. The 105 vendor features: what OpenRouter does with each, and what Runvil needs
- 4. Runvil should copy OpenRouter's rule for sending reasoning data back and three of its field names, and should not copy `reasoning_details` as its opaque field
- 5. Where OpenRouter's callers still have to know the vendor, Runvil's Gateway will face the same problem
- 6. OpenRouter's chat completions route leaves out three kinds of vendor feature: data kept on the vendor's servers, blocks a chat message cannot hold, and settings in a vendor's own words
- 7. What Runvil's Gateway, client, and Engine do today, and six defects that cost remote models their usage numbers, error details, and compaction
- 8. What Runvil should change: store more in the opaque field, build the OpenAI Responses translator early, fix four Gateway defects first, and give the Gateway a table of what each model accepts
- 9. OpenRouter's published material says how it handles 85 of the 105 features, and other projects or vendor documentation cover the rest
- 10. Earlier Runvil documents: what was taken from each, and what is now wrong or out of date in them
- 11. Limits: every statement comes from reading, every source was read on one day, and some of OpenRouter's behavior is still unknown
- 12. Sources and evidence files

## Terms used in this report

Each term has one meaning throughout.

- **Prompt:** a Markdown file with Lua code. Running prompts is what Runvil does.
- **Engine:** the part of Runvil that executes a prompt and keeps the conversation as a list of message records.
- **Lua:** the code inside a prompt. It starts each model request and can read five fields of each record.
- **Host:** the program that embeds the Engine and decides which model fills each model role.
- **Client:** Runvil's gateway client, the HTTP library that sends each model request to the Gateway.
- **Gateway:** Runvil's server. It forwards each model request to a vendor or to a model on the local machine.
- **Protocol:** the request format the Gateway uses toward one vendor. Today it has one, named `openai`, which is chat completions.
- **Model entry:** the Gateway's configuration record for one model, with its endpoint and its settings.
- **Operator:** the person who installs and configures a Gateway.
- **Chat completions:** OpenAI's original request and reply format for chat models, which most vendors and local servers accept in some version.
- **Universal API:** Runvil's planned API between the client and the Gateway: chat completions plus a few optional extra fields.
- **Vendor:** a company that makes models and defines its own API for them, such as Anthropic, OpenAI, or Google.
- **Provider:** a company that serves requests for a model. One model can have several: Claude is served by Anthropic, Amazon Bedrock, Google Vertex AI, and Microsoft Azure.
- **Endpoint:** one provider's offering of one model under one account.
- **Route:** one URL path of an API. OpenRouter has three for model requests: chat completions, Messages (Anthropic's format), and Responses (the format of OpenAI's newer API). The Gateway has one.
- **Upstream:** the server a gateway forwards a request to: a vendor, a provider, or a local model server.
- **Translator:** the code in a gateway that converts a request into one vendor's protocol and converts the reply back.
- **Reasoning:** text a model writes for itself while it works out an answer. Anthropic and Google call it thinking.
- **Block:** one element of a reply: a piece of text, a piece of reasoning, or a tool call.
- **Signature:** a string a vendor attaches to a reasoning block and checks when the block is sent back.
- **Replay:** sending data from an earlier reply back to the vendor, unchanged, in a later request.
- **Opaque field:** an optional extra field on an assistant message in the universal API. The Gateway fills it with data the vendor expects back, such as signed thinking blocks. The client and the Engine store it and send it back unread, and Lua never sees it. Runvil's design for it is in the report of 2026-11-17, "How agent harnesses handle Anthropic's Messages API, and what Runvil should build": the field holds the reply's whole list of blocks, in order, and a label that names the protocol and the model that produced them.
- **Prompt caching:** a vendor stores the processed start of a request, so that a later request that starts the same way is cheaper. A **cache marker** tells the vendor where the cached part ends.
- **Sticky routing:** OpenRouter's habit of sending a conversation's later requests to the provider that served its earlier ones.
- **Tool loop:** the cycle in which the model asks for a tool call, the program runs the tool and sends the result back, and the model continues.
- **Finish reason:** the field in a reply that says why the model stopped.
- **Usage:** the token counts a reply reports.
- **Compaction:** shortening a conversation that has outgrown the model's context window. In Runvil a Lua function called a compactor does it.
- **Lesson plan, planned work:** Runvil's work plan of 2026-11-17 (`runvil-classroom`, `lesson-plans/2026-11-17.md`). Its item 2 adds vendor protocols to the Gateway, and its item 3 makes the gateway client a standalone library.
- **Model role:** one of five names (`default`, `strong`, `small`, `fast`, `creative`) that a prompt uses in place of a model name.
- **Model sheet:** `cloud-provider-models.json`, a weekly file that lists each cloud provider's models and their settings. The Gateway fills its model list from it.
- **OpenRouter's catalog, spec, and SDK:** its public list of models, its published API specification, and its official client libraries.
- **Recorded traffic:** requests and replies to OpenRouter that open-source projects saved in their test suites, dated 2025-05-20 to 2026-11-18.
- **Evidence item:** a code such as R36, in square brackets or on a feature's Evidence line. Each is a numbered entry in one of the evidence files listed in section 12, with the exact quote, its source, and the date it was read.

## 1. Each feature is judged by which parts of Runvil would need vendor-specific code

**What is asked of each feature.** A vendor-specific feature is one thing that a vendor's API does in its own way. Section 3 describes 105 of them in ten groups. Each has its own numbered section, Feature 1 to Feature 105, which gives how OpenRouter exposes the feature, what OpenRouter loses, and what Runvil would need to support it in the universal API.

**Why OpenRouter comes first.** It is the largest existing attempt at one chat completions API over every major vendor, and it has paying callers who report what breaks. Where OpenRouter hides a vendor rule behind a vendor-neutral field, Runvil's Gateway can probably do the same. Where OpenRouter tells its callers that the result depends on the provider, Runvil will probably need code outside the Gateway at the same place.

**The six options.** Runvil's design goal is that the Engine does not have to care which vendor runs a prompt, and that a prompt never does. Table 1 lists the six ways a feature can be supported, from best to worst, with the project owner's words for each.

*Table 1. The six options, best to worst.*

| Option | What it means | The project owner's words |
|---|---|---|
| Gateway only | All vendor-specific code is in the Gateway. The client, the Engine, and Lua stay the same, or change only in a vendor-neutral way | "Ideally we handle it all in the Gateway" |
| Opaque field | Vendor data travels in one optional field. The client and the Engine store it and send it back unread, and Lua never sees it | "adding opaque fields to messages which the engine and the Lua never see, is perfectly fine" |
| Compatibility code | A little vendor-specific code changes or repairs a valid request or reply so that one vendor accepts it: it drops a setting, rewrites an id, cleans a schema | "if we need a little compatibility code to massage things that come from the engine to fit a specific vendor, that is ok too (but note it in the report)" |
| Vendor branch in shared code | The client or the Engine has a branch that runs for one vendor only | "A small vendor patch-up in the universal path is unfortunate but could be ok as long as the vendor is big (Anthropic is big)" |
| Separate vendor path | The feature works only through a second route or request format that goes around the universal API | "Less desirable would be having to have a vendor-specific path that bypasses all of the universal path" |
| Prompt must know the vendor | The prompt's author has to know which vendor runs it | "a complete non-starter would be anything that forces the Lua prompt to care about the vendor" |

**How a feature gets its option.** Supporting one feature can take several pieces of work. The feature is listed under the worst option that any piece needs. The order of the options follows where the vendor-specific code has to be, and whether the request keeps its meaning. The opaque field ranks above compatibility code because the client and the Engine treat the field the same way for every vendor and the request keeps its meaning, while compatibility code changes a request for one vendor.

- **Telling Gateway only from compatibility code.** Both can be entirely in the Gateway. A Gateway-only conversion keeps the meaning of what the Engine sent. Compatibility code changes it for one vendor, usually in a way the prompt's author could notice: an ignored temperature, a rewritten id, a removed schema keyword.
- **Telling compatibility code from a vendor branch.** Compatibility code is at an edge: in the Gateway, or in the one function where the client builds a request. A vendor branch is inside the main code of the client or the Engine. Each feature's section says where every change is, so a reader who draws the line elsewhere can re-sort the features.
- **One opaque field for all.** Every feature that needs the opaque field uses the same one. The client and the Engine are changed once, for Anthropic's signed thinking (feature 1). After that, each further feature takes Gateway code only, and its option reads "Opaque field, reused".

**New vendor-neutral settings.** Some features need a new setting or field in the universal API that means the same for every vendor, such as a reasoning-effort value. The feature's section then names it. A prompt may use such a setting freely. Adding one changes the client, the Engine, and sometimes Lua. That code is the same for every vendor, so the feature's option stays the same.

**Features Runvil does not use today are marked "if adopted".** Examples are code execution on the vendor's servers, batch jobs, and conversations stored by the vendor. Each of their sections says what Runvil would need if it adopted the feature.

**Where each answer comes from.** OpenRouter's translator is closed source. Where its published material is silent, the answer comes from open-source projects or from the vendor's documentation, and the feature's Source line says so:

- **Stated by OpenRouter:** in its documentation, API specification, model catalog, changelog, or official client library source.
- **Inferred about OpenRouter:** worked out from something the Source line names: client code that special-cases OpenRouter, recorded traffic, an issue report, or a field missing from the specification.
- **From other open-source projects:** how a named project handles the same thing. This shows that a solution exists. What OpenRouter does stays unknown.
- **Stated by the vendor:** the vendor's own rule, read on 2026-11-22.
- **Earlier Runvil research:** an existing report or code in the Runvil repositories, named and dated.

What Runvil needs depends on the vendor's rule and on whether a gateway can translate it. So when an open-source gateway translates a feature correctly, that is evidence that Runvil's Gateway can do the same. When every open-source gateway examined gets a feature wrong, the Gateway probably needs more than translation.

**Confidence.** Each feature has a confidence level (high, medium, or low) and a one-phrase reason.

## 2. Of 105 features, 65 need only the Gateway, 25 need the opaque field, 10 need compatibility code, and 5 need a separate vendor path

Table 2 counts the features by option. "Needed" means for the planned work: a tool loop with reasoning on Anthropic, OpenAI, and Google models, on vendors with OpenAI-style APIs, and on local servers. "If adopted" counts the features Runvil does not use today.

*Table 2. Features per option.*

| Option | All | Needed | If adopted | What the features are |
|---|---|---|---|---|
| Gateway only | 65 | 52 | 13 | Translation, defaults, and common forms: reasoning settings, caching, tool formats, finish reasons, errors, usage, streaming, model discovery, routing |
| Opaque field | 25 | 17 | 8 | Signed, redacted, and encrypted reasoning, thought signatures, block order, plain-text reasoning that a vendor wants back, blocks from vendor-run tools |
| Compatibility code | 10 | 6 | 4 | Small fixes, every one of them in the Gateway (Table 3) |
| Vendor branch in shared code | 0 | 0 | 0 | Two cases come close: today's thinking switch, and joining OpenRouter's stream fragments in the standalone client |
| Separate vendor path | 5 | 0 | 5 | Data or jobs kept on the vendor's servers (Table 4) |
| Prompt must know the vendor | 0 | 0 | 0 | Closest: tools that only one vendor has, and settings in a vendor's own words |
| Total | 105 | 75 | 30 | |

Source: counted from the 105 feature sections in section 3.

**New vendor-neutral settings.** Sixteen features need a new setting or field, and for six more one is optional. Two matter now: a reasoning-effort setting (feature 21) and a count of the tokens written to the cache (feature 38). The others come only with a feature Runvil does not use today: a response schema, annotations, a flag that defers a tool, file, audio, and video parts, media in a reply, a pending tool result.

**Confidence.** It is high for 55 features, medium for 43, and low for 7. The medium ones are mostly features where the evidence that the handling works is another project's code, or where the vendor's rule is weeks old.

### All ten pieces of compatibility code can be in the Gateway

The client, the Engine, and Lua stay as they are for all ten.

*Table 3. The ten features that need compatibility code.*

| Feature | What the code does | Vendor | Where it is | Size | What a prompt author could notice |
|---|---|---|---|---|---|
| 12 | Puts Google's placeholder signature on the first function call of a step when history came from another model | Google | Gemini translator | About twenty lines | Google says quality drops for that turn |
| 28 | Drops `temperature`, `top_p`, `top_k` for models that reject or ignore them | Anthropic, Google, Moonshot, OpenAI, DeepSeek; xAI for `stop` and penalties | Request handling, from the per-model table | One table lookup | A temperature set in a prompt is ignored |
| 46 | Rewrites tool-call ids to the target vendor's format, on the call and on its result, and guarantees unique ids | Mistral, Anthropic, Bedrock, Kimi templates | Each translator and the `openai` upstream | One function per id rule | The prompt sees the same result |
| 48 | Cleans a tool schema for the vendor and sets any header the vendor wants for `strict` | Google, Anthropic | Each translator | Small and shrinking | A dropped schema keyword |
| 101 | Small per-vendor adapters on OpenAI-style routes: content chunk lists, an error inside a 200 body, extra finish reasons | Mistral, MiniMax, xAI, DeepSeek | The `openai` upstream, keyed by endpoint | A few lines each | The prompt sees the same result |
| 104 | Checks a local server's limits before forwarding: named `tool_choice`, tools on a template that cannot render them, a grammar sent with tools | llama.cpp | The local upstream | A few checks | The prompt sees the same result |
| 30 (if adopted) | Maps a trailing assistant message to the vendor's prefill flag, or refuses early | DeepSeek, Moonshot, Anthropic | Request handling | A few lines | Prefill fails on newer Claude models |
| 44 (if adopted) | Downgrades or refuses a forced tool choice on a model that rejects it | Anthropic, DeepSeek, Qwen, Zhipu, Meta | Request handling, from the per-model table | One table lookup | "Must call a tool" becomes "may" |
| 51 (if adopted) | Wraps a freeform tool as a function call with one string argument | OpenAI | Responses translator | Small | The prompt sees the same result |
| 56 (if adopted) | Emulates schema output with a forced tool call where the endpoint only writes free text | Claude on Bedrock, others | Each translator | Small | A weaker guarantee |

### The five features that would need a separate vendor path are all ones Runvil does not use today

In four, the vendor keeps data or a job on its servers, so a request has to refer to something outside itself. The fifth is a WebSocket connection.

*Table 4. The five features that would need a separate vendor path, and how each could be avoided.*

| Feature | What it is | Why it needs a separate path | How to avoid it |
|---|---|---|---|
| 68 | Vendor file stores and file search | The file and its id belong to one vendor account, behind the vendor's own upload route | The Gateway offers a vendor-neutral files route and maps ids. The price is stored data in the Gateway |
| 74 | Conversations stored by the vendor | The vendor stores the history and the caller sends an id | Send the whole conversation with the opaque field, which gives the model the same reasoning. LiteLLM's emulation needs a database |
| 75 | Background runs with polling | A job outlives the request and needs a retrieve route | A vendor-neutral job route in the Gateway, if ever wanted |
| 76 | Batch jobs | A separate asynchronous route | A vendor-neutral batch route in the Gateway, if ever wanted |
| 83 | WebSocket transport | A second transport | Not examined |

### The cases that come closest to a vendor branch in shared code, or to a prompt that knows its vendor

**Two cases come close to a vendor branch in the client, and both can be avoided.**

- **Today's thinking switch.** The client sends `chat_template_kwargs.enable_thinking`, a field that only local servers such as llama.cpp and vLLM understand. That is vendor-specific code in the client today. The fix is for the client to send a standard field and for the Gateway to convert it (feature 22). This work is already planned.
- **Code in the standalone client that joins `reasoning_details` fragments.** It is needed only if the standalone client (item 3 of the lesson plan) has to send reasoning back to OpenRouter or Vercel AI Gateway directly, with the Runvil Gateway left out. Those servers stream the data in fragments that the caller must join (feature 18). The alternatives: reach OpenRouter through the Gateway, where the joining is compatibility code, or accept that the bare client sends reasoning back only to servers that deliver it whole. This is decision 5 in section 8.

**What comes closest to making a prompt know its vendor: single-vendor tools, settings in a vendor's own words, requests that some models refuse, and one field in today's code.**

- **Tools that only one vendor has (feature 73).** No project read has a vendor-neutral form of Maps grounding, X search, or Kimi's formula tools. A prompt can ask for a capability by a vendor-neutral id and get "unavailable" on other models, and Runvil can decline these tools.
- **Settings in a vendor's own words (features 26, 72, 96, 97).** Gemini's safety thresholds, `reasoning.mode: "pro"`, Anthropic's beta header values, and `provider.options` all ask OpenRouter's caller for a vendor's own names. In OpenRouter the caller is a program. In Runvil it would be a prompt. The operator sets each one on a model entry in the Gateway's configuration, and a model role is mapped to that entry.
- **Vendor-neutral requests that some models refuse (features 22, 28, 30, 44).** Thinking off, a temperature, a prefill, and a forced tool choice each fail or are ignored on some models. The prompt names only a model role, yet its author can see the difference. The prompt states a need and the Host picks a model that meets it, or the Gateway reports what it dropped.
- **A field in today's code.** A model handle shows `provider` to Lua (`runvil-harness`, `decisions/models-and-conversations.md`), so a prompt can already branch on the vendor. All 105 features work with that field removed.

## 3. The 105 vendor features: what OpenRouter does with each, and what Runvil needs

Each numbered section below is one vendor-specific feature. The ten groups (3.1 to 3.10) hold related features, and the numbers run across all of them.

**How to read a feature.**

- **On OpenRouter** says how OpenRouter exposes the feature, and **What OpenRouter loses** says what its caller cannot get or cannot tell apart. In both, "the catalog" is OpenRouter's public model list, "the spec" is its published API specification, "the SDK" is one of its official client libraries, and "recordings" are recorded traffic. Features 103 to 105 are about local servers, which OpenRouter does not serve, so they have neither line.
- **Option** is one of the six in Table 1. "Opaque field, reused" means the feature reuses the opaque field that feature 1 builds, so only the Gateway changes. "If adopted" marks a feature Runvil does not use today.
- **Work in Runvil** names the parts that change: the Gateway, the universal API, the client, the Engine, Lua.
- **New setting or field** appears when the universal API needs a new vendor-neutral one for the feature.
- **Source** uses the labels of section 1 and names the projects whose code shows that the handling works.
- **Evidence** lists evidence items (section 12).

**To find a vendor's features:**

- **Anthropic:** features 1 to 5, 9, 22 to 25, 27 to 33, 40, 41, 43 to 48, 52, 54 to 69, 76 to 80, 82, 84, 86 to 88, 90, 91, 95, 96, 98, 100
- **OpenAI:** features 3, 5 to 9, 25 to 28, 33 to 36, 43, 45, 48, 51 to 55, 57, 58, 61, 62, 64 to 69, 71, 74 to 77, 80 to 84, 86 to 88, 91, 98
- **Google:** features 3, 5, 10 to 13, 22, 23, 25, 28, 33, 34, 37, 43, 46 to 48, 54, 55, 58, 60 to 62, 64, 65, 68 to 74, 84 to 86, 88
- **xAI:** features 6, 28, 34, 36, 61, 73, 74, 101
- **DeepSeek:** features 14, 28, 30, 34, 44, 101, 102
- **Mistral:** features 15, 43, 45, 46, 86, 101
- **Moonshot Kimi:** features 14, 22, 28, 30, 33, 34, 46, 73
- **Alibaba Qwen:** features 14, 16, 23, 31, 44, 73, 102
- **Zhipu GLM:** features 14, 22, 34, 36, 44, 73, 86, 88, 102
- **MiniMax:** features 14, 86, 101, 102
- **Meta:** features 6, 44
- **Local servers (llama.cpp, vLLM, Ollama) and providers that serve open-weight models:** features 17, 50, 95, 102 to 105
- **Every vendor:** features 18, 19, 21, 36, 38, 42, 49, 70, 87, 89, 92, 94, 97, 99, 100

### 3.1 Reasoning data that a vendor expects back: every vendor's version fits the one opaque field

OpenRouter's `reasoning_details` array does the job of Runvil's planned opaque field. Every vendor's reasoning data fits the one opaque field. What OpenRouter's array loses is the order of blocks inside a reply, which the opaque field keeps.

#### Feature 1: Signed thinking blocks, exact replay

- **Vendors:** Anthropic
- **What it is:** Each `thinking` block has a `signature`. The whole assistant turn goes back exactly as returned, empty-text blocks included, or the API answers 400.
- **On OpenRouter:** Chat completions route: `message.reasoning_details[]`, one `reasoning.text` entry per block with `text`, `signature`, `format: "anthropic-claude-v1"`. Rule: "the entire sequence of consecutive reasoning blocks must match the outputs generated by the model".
- **What OpenRouter loses:** Where each block was among text and tool calls. A `reasoning` string sent back without the array becomes unsigned thinking, which Anthropic rejects.
- **Option:** Opaque field
- **Work in Runvil:** Gateway: the translator returns and replays the block list. Universal API: the opaque field. Client: copy it from the final chunk and send it back. Engine: a hidden field on the assistant record, which the Engine writes into each request.
- **Source:** Stated by OpenRouter. rig, Hermes Agent, and Agno store the whole block list and send it back, and the 2026-11-17 report designs the same field.
- **Confidence:** High - a documented rule, followed by three projects, not yet run
- **Evidence:** R36, R52, AN7, AN8, AN9, H50

#### Feature 2: Redacted thinking

- **Vendors:** Anthropic
- **What it is:** `redacted_thinking` blocks hold encrypted `data` and go back unchanged.
- **On OpenRouter:** Chat completions route: a `reasoning.encrypted` entry with `data`.
- **What OpenRouter loses:** It shares one entry type with OpenAI and Gemini encrypted values, told apart by `format` only.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway only, once the opaque field exists.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - the same documented replay rule as signed thinking (feature 1)
- **Evidence:** R37, AN2, AN8

#### Feature 3: Order of thinking, text, and tool calls inside one reply

- **Vendors:** Anthropic (interleaved thinking), OpenAI (output items), Google (parts)
- **What it is:** The vendor checks or relies on the original order. Anthropic's prefix check counts "changing text before a thinking block in its own message".
- **On OpenRouter:** Chat completions route: three separate fields, `reasoning_details`, `content`, `tool_calls`, and the position of each block is left undocumented.
- **What OpenRouter loses:** The order. Every open-source gateway read that uses this three-field format loses it: agentgateway and Bifrost rebuild reasoning, then text, then tool calls; Envoy AI Gateway keeps the first text and the last thinking block; Portkey moves tool uses to the end. LiteLLM's rebuild produced Anthropic's "cannot be modified" 400 in real use.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: return the list in the vendor's order and replay it instead of rebuilding it. The client drops text beside tool calls today, which stops mattering once the opaque field is replayed.
- **Source:** Inferred about OpenRouter, from the format in the spec. agentgateway, Bifrost, Envoy AI Gateway, Portkey, and LiteLLM all lose the order. rig keeps the raw blocks, and LLM Gateway adds `block_index`.
- **Confidence:** High - every translator read that uses separate fields loses the order, and the second round confirmed it
- **Evidence:** AN13, J9, J30, J74, J86, J100, K6, K7, K57, Q31, VF16

#### Feature 4: Prefix-bound thinking

- **Vendors:** Anthropic (Fable 5.1 and later, the 5.5 family)
- **What it is:** A thinking block is valid only while `system`, `tools`, and every earlier message are unchanged. A change is a 400 on some models, or the block is dropped when the request sets `thinking.block_binding.prefix_mismatch_behavior: "drop_block"`.
- **On OpenRouter:** Chat completions route: the upstream 400 is passed through and OpenRouter "does not drop thinking blocks for you". The `drop_block` setting and Anthropic's `input_transformations` report exist on the Messages route only.
- **What OpenRouter loses:** The `drop_block` setting and Anthropic's report, for chat callers.
- **Option:** Opaque field
- **Work in Runvil:** Gateway: send `drop_block` and never rewrite earlier messages. Engine: only append to the list (already the rule), and drop the opaque field when a compaction changes earlier messages.
- **Source:** Stated by OpenRouter. Goose, Zed, rig, pi, and opencode all send `drop_block` (2026-11-17 report).
- **Confidence:** Medium - documented, and untested through Runvil, which writes JSON with sorted keys
- **Evidence:** R62, AN3, AN10, AN12

#### Feature 5: Reasoning data tied to the account or endpoint that made it

- **Vendors:** Anthropic (thinking from Sonnet 5.5, Haiku 5.5, Fable 5.2, Mythos 5.2 works only in the organization that produced it), OpenAI (organization and model family), Google (Vertex scopes a signature to "the same conversation history, model, and Google Cloud project")
- **What it is:** An encrypted value is readable only where it was made. Elsewhere it is dropped without an error (Anthropic, OpenAI) or the tool continuation is rejected (Gemini).
- **On OpenRouter:** Routing: a request that sends reasoning back goes only to a provider that can read the blocks, else a 400. Every encrypted value in OpenRouter's examples ends with a base64 suffix that decodes to an `endpoint_slug` naming the model snapshot and the provider.
- **What OpenRouter loses:** A switch to another provider, for conversations that send reasoning back. The stored value is OpenRouter's wrapped form, which differs from the vendor's bytes.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: label the opaque field with the protocol, the model, and the endpoint that produced it. Replay on a match, else rebuild from the vendor-neutral fields. That is one value more than the protocol-and-model label of the 2026-11-17 design.
- **Source:** Stated by OpenRouter for keeping a replay on one provider. Inferred about OpenRouter for the suffix: decoded from 23 documentation examples and from recorded traffic, where the earliest OpenAI value with it is dated 2026-08-04 on the Responses route and 2026-09-22 on the chat completions route, and Claude signatures have it from late September 2026. In 34 recorded replays the reply came from the endpoint the suffix names. rig labels stored data with protocol, provider, and model, Pydantic AI records the endpoint for server ids, and Bifrost strips the data and retries once.
- **Confidence:** Medium - vendors differ on what "account" means
- **Evidence:** R63, AN5, AN11, R41, OA15, OA27, G37, GG22, H27, H51, J39, V21, VF1, VV7, VV18, WT2 to WT7

#### Feature 6: Encrypted reasoning items

- **Vendors:** OpenAI, Azure OpenAI; xAI, whose chat route has "no field for the ciphertext"; Meta, whose chat route does not keep reasoning between requests
- **What it is:** Reasoning items with `encrypted_content` return in order next to the function calls. Only the vendor's Responses route has them.
- **On OpenRouter:** Chat completions route: `reasoning.summary` and `reasoning.encrypted` entries (`data`, an `id` like `rs_...`, `format: "openai-responses-v1"`, also `xai-responses-v1` and `meta-responses-v1`). "OpenRouter automatically requests encrypted reasoning content".
- **What OpenRouter loses:** Item boundaries and each item's position among several output items. The same entry sent twice is rejected.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: a Responses translator that returns the reply's ordered item list in the opaque field. Client and Engine: store the field and send it back, as for signed thinking (feature 1).
- **Source:** Stated by OpenRouter for the format. Inferred that OpenRouter calls the Responses API upstream. LiteLLM's chat-to-Responses bridge, Bifrost, and new-api serve a chat caller from the Responses API, and rig and the Vercel AI SDK keep the items.
- **Confidence:** Medium - a second translator that Runvil has not designed
- **Evidence:** R35, R58, OA13, OA25, OA9, OV14, OV65, K16, K17, J35, H19, H54

#### Feature 7: Tool calling with reasoning on OpenAI's newest models

- **Vendors:** OpenAI (GPT-5.4 and later; the GPT-6 family)
- **What it is:** From GPT-5.4 OpenAI's Chat Completions "does not support tool calling with `reasoning_effort` values other than `none`". On five GPT-6 models (GPT-6 Astra, GPT-6.1 Sol, GPT-6.1 Terra, GPT-6.2 Astra, GPT-6.2 Sol) "tool calling requires Responses". Four of the five also reject effort `none`; GPT-6.1 Terra accepts `none` and still needs Responses for tools. OpenAI states this as a limitation and prints no error text for it.
- **On OpenRouter:** The catalog lists `tools` and the chat route for these models, so OpenRouter translates for the caller.
- **What OpenRouter loses:** Which of the vendor's APIs ran.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: the Responses translator of feature 6 is needed for a tool loop on these models at all. Forwarding chat completions as written, which Runvil does today, cannot run a tool loop with reasoning on them. Runvil's client sends tools and lets the model use its default effort, which is `medium` from GPT-5.5, so today's request already falls inside the limitation (an inference from OpenAI's per-model defaults).
- **Source:** Stated by the vendor, re-fetched in the verification round. Inferred about OpenRouter for OpenRouter's side.
- **Confidence:** High - vendor rule, confirmed on a second fetch
- **Evidence:** OA35, OA36, OA37, OA9, VV1

#### Feature 8: `phase` labels and several output messages in one reply

- **Vendors:** OpenAI (GPT-5.3 Codex, GPT-5.4 and later)
- **What it is:** Each assistant message item has a `phase` (`commentary` or `final_answer`) that the caller sends back.
- **On OpenRouter:** Responses route only. "The Chat Completions format cannot represent multiple output items with distinct phases in a single response".
- **What OpenRouter loses:** The labels and the split, for chat callers.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: the item list in the opaque field keeps each item's `phase`. The vendor-neutral reply gets the final answer's text.
- **Source:** Stated by OpenRouter. LiteLLM keeps an order map in `openai_responses_replay`, and the Vercel AI SDK stores `phase` on each text part.
- **Confidence:** Medium - two open-source projects keep the labels, and nobody has built it for Runvil
- **Evidence:** OA40, OA41, K17, H20

#### Feature 9: Reasoning dropped when history was edited

- **Vendors:** OpenAI (GPT-6.2 and later), Anthropic
- **What it is:** OpenAI drops replayed reasoning after an edit to earlier history, a model-family switch, or a changed `safety_identifier`, and lists each drop in `input_transformations`.
- **On OpenRouter:** The report field exists on the results of the Responses and Messages routes only.
- **What OpenRouter loses:** A chat caller is not told.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: keep the end-user identifier stable for a conversation and log the vendor's report. Engine: it only appends to the list, except in a compactor.
- **Source:** Stated by OpenRouter; stated by the vendor.
- **Confidence:** Medium - the OpenAI rule is five days old
- **Evidence:** OA27, OA21

#### Feature 10: Thought signatures on function calls

- **Vendors:** Google (Gemini 3 and 4)
- **What it is:** The first `functionCall` part of each step in the current turn comes back with its `thoughtSignature`, else HTTP 400.
- **On OpenRouter:** Chat completions route: a `reasoning.encrypted` entry with `format: "google-gemini-v1"` beside the tool calls. The tool call object stays plain.
- **What OpenRouter loses:** Which part the signature belongs to.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: a Gemini translator that keeps the ordered part list, signatures included, in the opaque field.
- **Source:** Stated by OpenRouter (spec and official SDK source). The Vercel AI SDK and opencode keep the signature with each part, and TensorZero keeps a signature block beside the part. Three gateways hide the signature inside the tool-call id instead (agentgateway, Bifrost, LiteLLM): the caller then stores only the id, at the price of ids up to 4,423 characters that break clients which normalize ids.
- **Confidence:** Medium - Runvil has not designed this translator, and open-source projects move the signature in six different ways
- **Evidence:** GG1 to GG5, GG17, GG19, K10, K12, K61, J10, J31, H14

#### Feature 11: Thought signatures on text, final, and image parts

- **Vendors:** Google
- **What it is:** Optional for correctness and needed for quality. When built-in tools are combined with custom functions, every part is signed and all of them must return.
- **On OpenRouter:** Chat completions route: the same array. OpenRouter's own client strips unsigned Gemini reasoning text on replay and keeps the encrypted entries.
- **What OpenRouter loses:** Per-part placement.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway only, in feature 10's translator.
- **Source:** Stated by OpenRouter (SDK source); stated by the vendor.
- **Confidence:** Medium - most open gateways keep the first signature only
- **Evidence:** GG7, GG13, GG20, J31, K12

#### Feature 12: Placeholder signature for history from another model

- **Vendors:** Google
- **What it is:** Google documents the value `skip_thought_signature_validator` for function calls that did not come from Gemini, as a last resort that "will negatively impact model performance".
- **On OpenRouter:** Not documented.
- **What OpenRouter loses:** Unknown whether OpenRouter injects it or lets the 400 through.
- **Option:** Compatibility code
- **Work in Runvil:** Gateway: about twenty lines in the Gemini translator, on the first function call of each step when the opaque field is absent or its label does not match.
- **Source:** From other open-source projects: LiteLLM, the Vercel AI SDK, Gemini CLI, Envoy AI Gateway, and new-api all inject it. Stated by the vendor for the value.
- **Confidence:** High - five projects do the same thing
- **Evidence:** GG8, GG24, GG28, GG30, GG31, K13, H15

#### Feature 13: Long output continued across requests

- **Vendors:** Google (Gemini 4)
- **What it is:** Output past the per-request cap ends with `finishReason: CONTINUATION` and a `continuationToken`. The next request repeats the same contents plus the token.
- **On OpenRouter:** Chat completions route: `finish_reason: "length"`, `native_finish_reason: "CONTINUATION"`, and the token as a `reasoning.encrypted` entry with `id: "gemini-continuation"`. The caller appends the stopped turn and resends; OpenRouter strips it and keeps the request on the same provider.
- **What OpenRouter loses:** The token "can reach several MB", and usage is reported per request.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: make the follow-up requests inside one client request and return one reply, with usage added up and the first request's prompt size kept for the context check. It applies only when a prompt sets `max_tokens` above the per-request cap of 65,536. A reply cut by its own budget stays a plain `length` stop.
- **Source:** Stated by OpenRouter for its loop, which the caller runs. Google's own SDKs, LiteLLM, and the Vercel AI SDK all make the follow-up requests inside the library and hand the application one reply.
- **Confidence:** Medium - three projects do this, and the cost per follow-up request and long silent requests are untested
- **Evidence:** GG41 to GG50, O59, GX46 to GX56, CT7, CT9, CT16, CT26

#### Feature 14: Required replay of plain-text reasoning

- **Vendors:** DeepSeek (when `tools` is present), Moonshot Kimi K3, Alibaba Qwen (`preserve_thinking`), Zhipu GLM (`clear_thinking: false`), MiniMax
- **What it is:** The assistant's `reasoning_content` comes back on later requests. DeepSeek answers a tool conversation without it with a 400.
- **On OpenRouter:** The caller may return `reasoning`, `reasoning_details`, or `reasoning_content` ("an alias"). Replies use `reasoning` and `reasoning.text` entries tagged `unknown`: the key `reasoning_content` is in 0 of 911 recorded responses. Filling the field for a caller that dropped it is not documented, and DeepSeek's 400 has reached callers through OpenRouter.
- **What OpenRouter loses:** The requirement itself is absent from OpenRouter's documentation.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: on the `openai` protocol too, put the reply's reasoning in the opaque field and write the vendor's field back on replay. A space-padded field is the fallback when the opaque field is absent.
- **Source:** Inferred about OpenRouter. pi and opencode send the reasoning back, and LiteLLM pads the field.
- **Confidence:** High - this fails today when Runvil forwards a tool loop to DeepSeek
- **Evidence:** OV22, OV23, OV28, OV40, OV48, OV54, R53, R59, K22, WT20, VV19

#### Feature 15: Reasoning as content chunk lists

- **Vendors:** Mistral (its `/v1` route)
- **What it is:** With reasoning on, `content` is a list of `ThinkChunk` and `TextChunk`, with an optional `signature`, and the delta format changes in the middle of a stream.
- **On OpenRouter:** `reasoning` and `reasoning_details`; `content` stays a string. The `format` value list has other vendors only, so a Mistral entry would be `unknown`.
- **What OpenRouter loses:** Whether a chunk signature survives is undocumented.
- **Option:** Opaque field, reused. The Gateway also converts the content format
- **Work in Runvil:** Gateway: turn chunk lists into text plus reasoning and keep the chunks in the opaque field. Today a reply whose `content` is an array fails in the client as `MalformedResponse`.
- **Source:** Stated by the vendor; inferred about OpenRouter.
- **Confidence:** Medium - Mistral's two endpoints are changing
- **Evidence:** OV31, OV35, L106

#### Feature 16: Summarized thinking with ciphertext

- **Vendors:** Alibaba (`qwen4-max`)
- **What it is:** A summary plus `encrypted_content` that the client "can only pass ... back as-is".
- **On OpenRouter:** Not documented. An Alibaba entry would be tagged `unknown`.
- **What OpenRouter loses:** Unknown whether it survives OpenRouter.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway only.
- **Source:** Stated by the vendor.
- **Confidence:** Medium - one vendor page
- **Evidence:** OV49

#### Feature 17: Plain-text reasoning under three field names

- **Vendors:** llama.cpp (`reasoning_content`), vLLM (`reasoning`, renamed), Ollama (`thinking`), providers that serve open-weight models
- **What it is:** Reasoning text arrives in one of three fields, or inside think tags. Templates for gpt-oss, Kimi, Qwen3, and GLM want it back between tool calls.
- **On OpenRouter:** Normalized to `message.reasoning` and `reasoning.text` entries; `reasoning_content` is accepted on input.
- **What OpenRouter loses:** Which field name the server used.
- **Option:** Opaque field, reused, to send it back. Reading it needs the Gateway only
- **Work in Runvil:** Gateway: normalize the name on the way out, write the server's own name on the way back. The client already reads all three.
- **Source:** Stated by OpenRouter; from other open-source projects: llama.cpp, vLLM, Ollama source.
- **Confidence:** High - read from the servers' source
- **Evidence:** OV67, OV73, OV77, R44, L25

#### Feature 18: How reasoning data arrives in a stream

- **Vendors:** All
- **What it is:** Anthropic sends `thinking_delta` events and then one `signature_delta`. Google sends the signature last, sometimes on an empty part.
- **On OpenRouter:** `delta.reasoning_details` fragments, documented in four bullets with no code. OpenRouter's own SDK merges by type change and says the `index` "cannot be used to reassemble the array"; rig and LangChain merge by `index`. Nine of the SDK's fifteen reasoning special cases are dated 2026 bug fixes, and one loss path is still open.
- **What OpenRouter loses:** Reliability: a signature can arrive after answer text has started, and three clients use two merge rules.
- **Option:** Opaque field, reused, by design choice
- **Work in Runvil:** Gateway: assemble the block list itself and emit it once, whole, on the final chunk. Client: copy one value. The per-vendor code that joins fragments stays out of the client.
- **Source:** Stated by OpenRouter (SDK source); inferred about OpenRouter from its fix history.
- **Confidence:** High - the client-side merge is where OpenRouter's own SDK kept breaking
- **Evidence:** R45, R46, R47, F1, F2, F3, F59, F60, K72, VF6, VT9

#### Feature 19: Model or provider switch in the middle of a conversation

- **Vendors:** All
- **What it is:** Each vendor accepts only its own data: Anthropic drops unreadable blocks silently, OpenAI omits incompatible reasoning, Gemini rejects a missing signature.
- **On OpenRouter:** Undocumented in general. Requests kept on one provider, and 400s, in the special cases of features 5 and 13.
- **What OpenRouter loses:** Unknown.
- **Option:** Opaque field, reused, plus the compatibility code of features 12 and 46
- **Work in Runvil:** Gateway: compare the opaque field's label with the target. On a mismatch rebuild the message from the vendor-neutral fields without reasoning. The check belongs in the Gateway because Runvil's Workshop application can change the model between rounds.
- **Source:** From other open-source projects: pi and Goose drop another model's reasoning on a switch, and rig and opencode pass it on as plain text. Pydantic AI also turns it into text, and an issue report describes Claude then printing its reasoning for more than 90 turns.
- **Confidence:** Medium - every project read withholds one vendor's signed or encrypted data from another model, and they split on whether to drop the readable reasoning or pass it on as text
- **Evidence:** R83, H51, H61, H65, H69, H72, G27, VT12

#### Feature 20: The gateway's own data inside the opaque field

- **Vendors:** OpenRouter
- **What it is:** Not a vendor feature.
- **On OpenRouter:** Deferred-tool state, server tool records, and video navigation are stored in extra `reasoning.encrypted` or `reasoning.server_tool_call` entries. Dropping them returns 400 codes such as `deferred_tools_state_missing`.
- **What OpenRouter loses:** A client that filters reasoning entries breaks tool search.
- **Option:** Opaque field, reused
- **Work in Runvil:** A lesson more than a feature: the opaque field can also hold the Gateway's own data, so each request brings the Gateway everything it needs.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented, with named error codes
- **Evidence:** R42, P50, P51

### 3.2 Reasoning settings and sampling: one effort setting works for every vendor, but some models refuse "thinking off" or a temperature

OpenRouter shows that one vendor-neutral effort setting can stand for every vendor's reasoning control, provided the gateway has a table of what each model accepts. It also shows two requests that some models cannot honor: turning thinking off, and setting a temperature.

#### Feature 21: Reasoning effort level

- **Vendors:** All reasoning vendors
- **What it is:** Each vendor has its own level names: OpenAI `reasoning.effort`, Anthropic `output_config.effort`, Google `thinkingLevel`, DeepSeek and Kimi `reasoning_effort`.
- **On OpenRouter:** `reasoning: {effort}` with `max`, `xhigh`, `high`, `medium`, `low`, `minimal`, `none`. The catalog lists `supported_efforts` per model, and an unsupported level maps to the nearest one.
- **What OpenRouter loses:** The per-model mapping tables are unpublished. One level means a budget share on old Claude, response effort on new Claude, a thinking level on Gemini.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: map from a per-model table (the model sheet already publishes `effort_levels`, and the Gateway does not use them when it handles a request). Universal API: OpenAI's standard `reasoning_effort`. Client: send it. Engine: one more model option. Lua: one more `models.use` option.
- **New setting or field:** Yes: an effort level
- **Source:** Stated by OpenRouter. The mapping tables are taken from LiteLLM, Vercel AI Gateway, and Requesty.
- **Confidence:** High - every gateway and model library read has exactly this vendor-neutral setting
- **Evidence:** R1, R2, R4, R16, K25 to K28, H55, Q40

#### Feature 22: Thinking on or off, and models that cannot turn it off

- **Vendors:** Claude Opus 5.5, Sonnet 5.5, Fable; Gemini 4; Grok 4.5 and later; GPT-6 Astra; GLM 5.3; Kimi K3
- **What it is:** The vendor rejects a disable request, or its only settings are levels of on.
- **On OpenRouter:** The catalog flags `reasoning.mandatory: true` on 151 of its 354 entries that have a reasoning object (recounted for this report). A disable request returns 400 `Reasoning is mandatory for this endpoint and cannot be disabled.` before routing. On `~latest` aliases it is changed to the lowest effort.
- **What OpenRouter loses:** A portable off switch.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: translate the switch per vendor (`thinking.type`, `enable_thinking`, `reasoning_effort: "none"`, template arguments) and publish `thinking: always` for these models, so the run's first step refuses an unmet `thinking: false`. Client: send a standard field in place of `chat_template_kwargs.enable_thinking` (section 7).
- **New setting or field:** No: the switch exists
- **Source:** Stated by OpenRouter.
- **Confidence:** High - the Engine's requirement check already works this way
- **Evidence:** R14, R15, R22, OV21, OV46, OV54

#### Feature 23: Reasoning token budget

- **Vendors:** Anthropic 4.6 and older, Gemini 2.5, some Qwen
- **What it is:** `thinking.budget_tokens`, `thinkingBudget`, `thinking_budget`.
- **On OpenRouter:** `reasoning.max_tokens`. Rejected together with `effort`. "Budgets are not forwarded" on adaptive-only Claude.
- **What OpenRouter loses:** A fading control: newer models ignore or reject it.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: derive a budget from the effort level for the models that still want one.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented on both sides
- **Evidence:** R3, R13, R17, AN20, OV47

#### Feature 24: Thinking modes per model, including `between_tools`

- **Vendors:** Anthropic
- **What it is:** `thinking.type` differs per model: `adaptive`, `enabled` with a budget, `between_tools` (Sonnet 5.5 only), `disabled`.
- **On OpenRouter:** Chosen by OpenRouter per model. `between_tools` is reachable on the Messages route only.
- **What OpenRouter loses:** The middle setting, for chat callers.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: the per-model table in Appendix A of the 2026-11-17 report, with `between_tools` as the answer to "off" on Sonnet 5.5.
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - Anthropic documents the per-model table
- **Evidence:** R20 to R23, AN20 to AN26

#### Feature 25: Thinking display and summaries

- **Vendors:** Anthropic (`thinking.display`), OpenAI (`reasoning.summary`), Google (`includeThoughts`)
- **What it is:** Each vendor returns a summary on request and never the raw reasoning.
- **On OpenRouter:** OpenRouter sends `display: "summarized"` by default. `reasoning.summary` sets the length, `reasoning.exclude` hides it.
- **What OpenRouter loses:** Anthropic's `updates` mode (Messages route only).
- **Option:** Gateway only
- **Work in Runvil:** Gateway default: ask for summaries, so the Host has reasoning text to show.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - a documented default
- **Evidence:** R5, R6, R24, AN18, AN19, GG35

#### Feature 26: Pro reasoning mode and reasoning context

- **Vendors:** OpenAI (GPT-5.6 and later)
- **What it is:** `reasoning.mode: "pro"` and `reasoning.context`.
- **On OpenRouter:** The same names inside the vendor-neutral `reasoning` object. `pro` reroutes to a `*-pro` model listing and is stripped for other providers.
- **What OpenRouter loses:** A vendor-specific value is inside a vendor-neutral object.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: a setting on a model entry, chosen by the operator. A role is mapped to the pro entry and the prompt never sends the value.
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - operator configuration is a design choice
- **Evidence:** R7, R8, OA28, OA30

#### Feature 27: Effort change in the middle of a conversation

- **Vendors:** OpenAI GPT-6 (`configuration_update` item), Anthropic (`output_config` on a content-less system message)
- **What it is:** A way to change effort that keeps the cached prefix: an item or message at a fixed position in history.
- **On OpenRouter:** OpenRouter invented a chat form: a content-less system message with `configuration_update: {reasoning: {effort}}`, converted across its three routes. 400 on a model that lacks it.
- **What OpenRouter loses:** The `text` half of OpenAI's item.
- **Option:** Gateway only, plus an optional new vendor-neutral record
- **Work in Runvil:** Without the feature a changed effort costs one cache miss. With it: a vendor-neutral "settings changed" record in the Engine's list that the Gateway translates.
- **New setting or field:** Optional: a "settings changed" record
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - the cache saving is unmeasured
- **Evidence:** R30, OA44, OA45, AN27

#### Feature 28: Sampling parameters removed, ignored, or fixed

- **Vendors:** Anthropic (Claude 4.7 and later; every model under API version `2026-10-21`), Google (Gemini 3.6 and later, and 4), Moonshot (K3: "passing any other value returns an error"), OpenAI reasoning models, DeepSeek (thinking mode ignores `temperature`), xAI reasoning models (for `stop` and the two penalties only)
- **What it is:** The vendor rejects or ignores `temperature`, `top_p`, `top_k`.
- **On OpenRouter:** The catalog drops them from `supported_parameters`: with batch twins set aside, 10 of 16 Anthropic models and 43 of 62 OpenAI models do not list `temperature`. The same catalog signal is documented as "silently ignored" on Opus 4.7 and as "rejects" on Haiku 5.5.
- **What OpenRouter loses:** A prompt's temperature is dropped or ignored, silently.
- **Option:** Compatibility code
- **Work in Runvil:** Gateway: drop the fields from a per-model table, one lookup per request. The cost is visible to a prompt author: `models.use('strong', {temperature = 0})` is ignored on these models.
- **Source:** Stated by OpenRouter; stated by the vendor.
- **Confidence:** High - documented with error text
- **Evidence:** AN65, AN66, M35, GG92, OV39, OV15, OV21

#### Feature 29: Required output cap

- **Vendors:** Anthropic (`max_tokens` is required)
- **What it is:** The request fails without it.
- **On OpenRouter:** OpenRouter fills `max_tokens: 65536` (in an example printed in its documentation).
- **What OpenRouter loses:** The caller's "no cap" becomes a cap.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: default to the model's `max_output` from the model sheet.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - Anthropic documents the requirement, and OpenRouter's documentation prints the filled value
- **Evidence:** O67, AN90

#### Feature 30: Assistant prefill

- **Vendors:** Anthropic (rejected on Claude 4.6 and later), DeepSeek (`prefix: true` on a beta URL), Moonshot (`partial: true`)
- **What it is:** A final assistant message is rejected, or needs a vendor flag.
- **On OpenRouter:** "OpenRouter returns the provider's 400". The vendor flags are unmapped.
- **What OpenRouter loses:** The provider's error is what the caller gets.
- **Option:** Compatibility code (if adopted)
- **Work in Runvil:** Gateway: map a trailing assistant record to the vendor's flag where one exists and refuse early elsewhere. Runvil's loop does not send a trailing assistant message today.
- **Source:** Stated by OpenRouter; stated by the vendor.
- **Confidence:** Medium - a capability some models lack, so a prompt that relies on it is not portable
- **Evidence:** AN69, OV26, OV43, T41

### 3.3 Prompt caching: the Gateway can handle it without help from the caller, apart from one diagnostic feature

OpenRouter shows that one vendor-neutral cache marker is enough, and that a gateway can go further and place the markers itself. The Gateway alone can handle every caching feature here except Anthropic's cache diagnostics (feature 40), which would also need the opaque field.

#### Feature 31: Explicit cache breakpoints

- **Vendors:** Anthropic (also Claude on Bedrock, Vertex, Azure), Alibaba Qwen
- **What it is:** `cache_control` on up to four blocks, a 20-block lookback, and a per-model minimum, under which a prefix goes uncached.
- **On OpenRouter:** Chat completions route: `cache_control` on text parts and on function tools.
- **What OpenRouter loses:** Placement and the four-marker limit stay the caller's job. Client code picks markers from the model id prefix.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: place the markers itself. Start with the automatic field of feature 32 and add tool and system breakpoints only if measured cache reads show misses.
- **Source:** Stated by OpenRouter. agentgateway places markers by its own policy, and the 2026-11-17 report found that the projects it read mark the same three places.
- **Confidence:** High - Anthropic and OpenRouter both document the marker and its limits, and agentgateway already places markers itself
- **Evidence:** C6, C7, AN30, AN32, F19, J15

#### Feature 32: Automatic caching through one top-level field

- **Vendors:** Anthropic
- **What it is:** A top-level `cache_control` puts the breakpoint on the last cacheable block and moves it forward each turn. Bedrock answers 400 to the field.
- **On OpenRouter:** Top-level `cache_control` on all three routes, rewritten into a trailing breakpoint for Bedrock. The feature survives whole, as far as the documentation says.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: add the field itself.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - one documented field
- **Evidence:** C6, C23, AN31, AN36

#### Feature 33: Cache lifetime

- **Vendors:** Anthropic (`5m`, `1h`), OpenAI (`30m`, `12h`), Google named caches (any `ttl`), Moonshot (`5m`, `1h`)
- **What it is:** Each vendor has its own lifetime values and its own write price per lifetime.
- **On OpenRouter:** `cache_control.ttl` and `prompt_cache_breakpoint.ttl`. "TTLs are not translated": a lifetime is dropped when a marker crosses vendor families, and Gemini is fixed at 5 minutes.
- **What OpenRouter loses:** The lifetime choice across vendors.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: a setting per model entry or endpoint, chosen by the operator.
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - the lifetimes and their prices are documented, and nobody has measured which lifetime costs less on Runvil's conversations
- **Evidence:** C19, C22, C64, C65

#### Feature 34: Automatic prefix caching

- **Vendors:** OpenAI, DeepSeek, xAI, Groq, Moonshot, Z.ai, Gemini implicit caching
- **What it is:** The vendor caches a repeated prefix by itself.
- **On OpenRouter:** The caller sends its request as always. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: send the unchanged part of each request as the same bytes. Engine: the append-only rule already protects the prefix.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - the vendors do the work
- **Evidence:** C1, C2, C10

#### Feature 35: OpenAI explicit breakpoints and cache options

- **Vendors:** OpenAI (GPT-5.6 and later)
- **What it is:** `prompt_cache_breakpoint` on a content block and `prompt_cache_options` at the root.
- **On OpenRouter:** The same names. A block marker is converted to and from `cache_control` according to the serving model, and `prompt_cache_options` "stays OpenAI-only".
- **What OpenRouter loses:** Lifetimes across vendors, `prewarm`, diagnostics.
- **Option:** Gateway only
- **Work in Runvil:** Gateway places them.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented on both sides
- **Evidence:** C3, C4, C19, C21, OA64

#### Feature 36: Cache routing key and session affinity

- **Vendors:** OpenAI (`prompt_cache_key`), xAI (`x-grok-conv-id`), Z.ai, any model with several providers
- **What it is:** A stable key steers requests to the machine that holds the cache.
- **On OpenRouter:** Sticky routing keyed on account, model, and a hash of the opening messages, or on `session_id`, `x-session-id`, or `prompt_cache_key`. Whether the caller's key reaches OpenAI or xAI is not written down.
- **What OpenRouter loses:** Sticky routing is off when `provider.order` is set, and two pages give two lifetimes (60 minutes and 10 minutes).
- **Option:** Gateway only
- **Work in Runvil:** Gateway: derive a key from a hash of the opening messages and set the vendor's field or header. A conversation id from the client makes it exact.
- **New setting or field:** Optional: a conversation id (OpenAI's `prompt_cache_key` is a standard name for it)
- **Source:** Stated by OpenRouter; whether the key is forwarded is unpublished. pi sends the key itself.
- **Confidence:** Medium - the cache benefit is unmeasured
- **Evidence:** C25 to C29, C16, OV18, OV58

#### Feature 37: Named cache objects

- **Vendors:** Google (Gemini 2.5 and 3; unavailable on Gemini 4 and on the Interactions API)
- **What it is:** The caller creates a `cachedContents` resource and refers to it by name.
- **On OpenRouter:** Hidden behind the same `cache_control` marker. OpenRouter creates and reuses the object, counts only the last breakpoint, and fixes the lifetime at 5 minutes.
- **What OpenRouter loses:** Cache names, lifetime, update, delete.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Gateway: hold the handle, or find it again by listing as LiteLLM does (a hash of the marked prefix stored as the cache's `displayName`). Implicit caching covers the common case.
- **Source:** Stated by OpenRouter. LiteLLM finds the cache again by listing.
- **Confidence:** Medium - Google is retiring the feature on its newest models
- **Evidence:** C11, C15, GG52 to GG57, K32

#### Feature 38: Cache read and write counts

- **Vendors:** All
- **What it is:** Anthropic reports reads and writes apart from `input_tokens`. OpenAI and DeepSeek report inclusive counts.
- **On OpenRouter:** `prompt_tokens` is the whole prompt, with `prompt_tokens_details.cached_tokens` and `.cache_write_tokens`.
- **What OpenRouter loses:** The split of writes by lifetime. In recordings the two counts are separate parts of the prompt for Anthropic and OpenAI, and name the same tokens on the Gemini request that creates a cache, so one subtraction rule does not fit every vendor.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: add Anthropic's three counters together, and keep reads and writes disjoint for every vendor. Client and Engine: one more vendor-neutral field on `Usage` for cache writes.
- **New setting or field:** Yes: a cache-write count (now OpenAI's own field name)
- **Source:** Stated by OpenRouter; inferred from recorded traffic. The sum rule is taken from LiteLLM and agentgateway.
- **Confidence:** High - checked against recorded usage numbers
- **Evidence:** C34, C41, C42, C73, C74, Q30, WT23 to WT25

#### Feature 39: Cost in the usage object

- **Vendors:** An OpenRouter addition
- **What it is:** Vendors report tokens only.
- **On OpenRouter:** `usage.cost`, `usage.cost_details`, `usage.is_byok`. These fields are additions, so the vendor's data survives whole.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Gateway: needs cache read and write prices per model from the model sheet.
- **New setting or field:** Optional: a cost value in usage
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - six gateways copied the field, with different units
- **Evidence:** C43, Q44

#### Feature 40: Cache diagnostics

- **Vendors:** Anthropic
- **What it is:** `diagnostics.previous_message_id` in the request, `cache_miss_reason` in the reply.
- **On OpenRouter:** Outside the schemas of both routes.
- **What OpenRouter loses:** The feature.
- **Option:** Opaque field (if adopted)
- **Work in Runvil:** Gateway: the previous reply's message id would be stored in the opaque field.
- **Source:** Stated by the vendor; inferred about OpenRouter from the spec.
- **Confidence:** Low - no surveyed project uses it through a gateway
- **Evidence:** AN33, AN34

#### Feature 41: Tool changes that keep the cache

- **Vendors:** Anthropic (`tool_addition` and `tool_removal` blocks, and the `clear_at` field)
- **What it is:** Blocks inside a system message that change the tool set and leave the cached prefix alone. `clear_at` is a field on such a message, with the values `next_user_message` and `never`.
- **On OpenRouter:** Messages route only: "the Chat Completions API has no representation for these blocks".
- **What OpenRouter loses:** The feature, for chat callers.
- **Option:** Gateway only, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** The Engine fixes the tool list at the first round of a tool loop, so the need does not arise today. If wanted: a vendor-neutral "tools changed" record that the Gateway translates.
- **New setting or field:** Yes, if the feature is adopted
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - one vendor has the feature
- **Evidence:** AN64, AN79, AN80, C56, T42

### 3.4 Tools: definitions translate cleanly, while ids, schemas, and forced tool choice need small fixes

Tool definitions translate cleanly. Three details cause trouble: tool-call ids, schemas, and a forced tool choice. OpenRouter passes the vendor's error through for all three, and it publishes per-endpoint flags for the forced choice only (`supports_tool_choice`, since 2026-08-28). Each is a small piece of compatibility code in a gateway.

#### Feature 42: Tool definition format

- **Vendors:** All
- **What it is:** OpenAI `tools[].function`, Anthropic `tools[]` with `input_schema`, Gemini function declarations.
- **On OpenRouter:** The OpenAI format, translated per provider. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - every translator read does this
- **Evidence:** T1, T2

#### Feature 43: `tool_choice` words

- **Vendors:** OpenAI, Anthropic, Google, Mistral
- **What it is:** OpenAI `auto`, `none`, `required`, a named function. Anthropic `any`, `tool`. Gemini `ANY`, `VALIDATED`.
- **On OpenRouter:** The four OpenAI forms, with a per-endpoint `supports_tool_choice` object since 2026-08-28.
- **What OpenRouter loses:** Gemini's `VALIDATED`, and on the chat completions route OpenAI's `allowed_tools`, which the Responses route accepts.
- **Option:** Gateway only
- **Work in Runvil:** Gateway. The client sends `auto` only today.
- **New setting or field:** Optional: a forced choice
- **Source:** Stated by OpenRouter.
- **Confidence:** High - a one-to-one mapping
- **Evidence:** T7, T8, GG83

#### Feature 44: Forced tool choice refused

- **Vendors:** Anthropic (Fable 5.2, Mythos 5.2, Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1), DeepSeek and Qwen in thinking mode, Zhipu GLM and Meta (`auto` only)
- **What it is:** A 400, for example `tool_choice: type "tool" and "any" are not supported for this model.`
- **On OpenRouter:** Forwarded, and "the provider's 400" comes back. The flags show `required: false, function: false` on every endpoint of Claude Fable 5.2, Opus 5.5, and Sonnet 5.5 and on 12 of the 13 endpoints of `deepseek/deepseek-v4-pro`. On two of the three Meta models read they show all four values as supported.
- **What OpenRouter loses:** A vendor-neutral "must call a tool" that works on every model.
- **Option:** Compatibility code (if adopted)
- **Work in Runvil:** Gateway: from the model table, downgrade to `auto` plus an instruction, or refuse early. Runvil never forces a tool today.
- **Source:** Stated by OpenRouter. LiteLLM and agentgateway pass the value through without a downgrade.
- **Confidence:** Medium - only matters once a forced choice is offered
- **Evidence:** T10, T9, T48, T55, T60, OV24, OV50, OV56, OV64, VV22

#### Feature 45: Parallel tool calls

- **Vendors:** OpenAI and Mistral (`parallel_tool_calls`), Anthropic (`disable_parallel_tool_use`, inverted), single-call templates such as Llama 3.x
- **What it is:** One switch, spelled both ways.
- **On OpenRouter:** `parallel_tool_calls`. The Anthropic mapping is undocumented, and the catalog lists the flag on 18 of 476 models.
- **What OpenRouter loses:** Too few models list the flag for it to guide routing.
- **Option:** Gateway only
- **Work in Runvil:** Gateway inverts it for Anthropic.
- **Source:** Stated by OpenRouter; from other open-source projects: LiteLLM, agentgateway.
- **Confidence:** High - two translators invert it the same way
- **Evidence:** T13, T14, T60

#### Feature 46: Tool-call id formats

- **Vendors:** Anthropic (`^[a-zA-Z0-9_-]+$`), Mistral (its validator library `mistral-common` requires exactly nine alphanumerics on tokenizer versions 3 to 11, error 3280), Bedrock (64 characters), Kimi K2 templates (`functions.name:idx`), Gemini (optional before Gemini 3)
- **What it is:** Each vendor checks ids its own way.
- **On OpenRouter:** The vendor's own id reaches the caller: recordings show `toolu_`, `toolu_bdrk_`, `call_`, `chatcmpl-tool-`, nine characters from Mistral, and `functions.name:0`. A rewrite when history crosses vendors is not documented. Mistral's error has reached callers through OpenRouter, and other recordings show one vendor's ids accepted by another with a 200.
- **What OpenRouter loses:** Ids reach the caller as the vendor wrote them. What OpenRouter does to ids in a request is unsettled.
- **Option:** Compatibility code
- **Work in Runvil:** Gateway: rewrite ids deterministically on the call and on its result for the target vendor (a hash, so nothing is stored), and guarantee unique, non-empty ids, because the Engine requires uniqueness across the whole conversation. Small: one function per id rule.
- **Source:** Inferred about OpenRouter. Bifrost, opencode, pi, and Roo Code rewrite ids.
- **Confidence:** High - four projects rewrite ids, and each vendor's rule is known
- **Evidence:** T15, T16, T18, T59, T62, T64, OV30, OV37, OV41, J34, L46, WT27, WT28, VT15, VV20

#### Feature 47: Tool result pairing and order

- **Vendors:** Anthropic (all results in the next user message, first), Google (all calls, then all responses)
- **What it is:** Ordering rules enforced with 400s.
- **On OpenRouter:** Standard `tool` messages. The regrouping is undocumented. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway regroups. The Engine already emits one tool message per call directly after the assistant record.
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - the ordering rules are documented
- **Evidence:** T19, T49, GG85

#### Feature 48: Tool schema differences and `strict`

- **Vendors:** Google (a subset of OpenAPI schema), Anthropic (root object, strict subset), OpenAI (strict rules)
- **What it is:** A rich schema can be rejected.
- **On OpenRouter:** Passed as written. `strict` is stripped for Anthropic unless the caller sends `x-anthropic-beta: structured-outputs-2025-11-13`.
- **What OpenRouter loses:** A strict request silently becomes non-strict.
- **Option:** Compatibility code
- **Work in Runvil:** Gateway: clean the schema per vendor and set any header itself. Small and shrinking: Gemini's newer `parametersJsonSchema` and `responseJsonSchema` take plain JSON Schema, and what remains is inlining references and dropping a few keywords.
- **Source:** Stated by OpenRouter. agentgateway, LiteLLM, Portkey, and LangChain clean schemas for Gemini, and the Vercel AI SDK, Bifrost, and Gemini CLI now send plain JSON Schema.
- **Confidence:** Medium - what each vendor rejects keeps moving
- **Evidence:** T20, T21, T22, T62, GG87, GX36 to GX45

#### Feature 49: Streaming of tool arguments, and calls cut by the output cap

- **Vendors:** All
- **What it is:** OpenAI streams fragments, Gemini sends whole calls, and Anthropic's eager streaming can yield invalid JSON. A cut call ends with the vendor's length reason.
- **On OpenRouter:** `delta.tool_calls[]`. OpenRouter sets `eager_input_streaming: true` on every streamed request with tools. A recording shows a cut call as `finish_reason: "tool_calls"` beside `native_finish_reason: "length"`.
- **What OpenRouter loses:** That the call was cut, in the normalized finish reason.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: emit deltas in OpenAI's format, check the JSON when a call closes, and report `length` for a cut call. The client already fails a batch that ends in `length`.
- **Source:** Stated by OpenRouter; inferred about OpenRouter from rig's recordings.
- **Confidence:** High - 16 of the 24 replies that rig recorded for this case show the cut call, all from OpenAI models
- **Evidence:** T24 to T28, O78, AN78, F51, WT29, VF4

#### Feature 50: Tool calling for models that lack it

- **Vendors:** Providers that serve open-weight models
- **What it is:** The model's template does not render tools.
- **On OpenRouter:** Three paths behind one field: pass as written, transform, or a prompt template. The third is known from one code comment and conflicts with "the parameter is ignored".
- **What OpenRouter loses:** Which of the three was used.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: it already emulates tool calls for one model family, in its `gemma3_tool_code` format.
- **Source:** Stated by OpenRouter, in one code comment; earlier Runvil research.
- **Confidence:** Medium - OpenRouter's own account is one code comment
- **Evidence:** T1, T5, L108

#### Feature 51: Freeform tools and grammars

- **Vendors:** OpenAI (custom tools with `lark` or `regex` grammars, `apply_patch`)
- **What it is:** The tool's input is plain text, not a JSON object.
- **On OpenRouter:** Responses route only. The chat route accepts `function` tools and OpenRouter's server tools only.
- **What OpenRouter loses:** The feature, for chat callers.
- **Option:** Compatibility code (if adopted)
- **Work in Runvil:** Gateway: offer the custom tool to the model and hand the call to the client as a function call with one string argument. Without that wrapper the universal API, the client, and the Engine each need a second tool-call format.
- **New setting or field:** No, with the wrapper
- **Source:** Stated by OpenRouter. The wrapper is an untested proposal: no project read does it.
- **Confidence:** Low - no project read wraps a freeform tool this way
- **Evidence:** OA54, T37

#### Feature 52: Tool search and deferred tool loading

- **Vendors:** Anthropic (`defer_loading`, `tool_reference` blocks), OpenAI (`tool_search`, namespaces)
- **What it is:** Deferred definitions stay out of the prefix, and the search blocks come back unchanged.
- **On OpenRouter:** `defer_loading` per tool, plus OpenRouter's own `deferred_tools` protocol whose state hides "inside the encrypted reasoning it returns". Regex search run by OpenRouter; passed through where the endpoint defers tools itself.
- **What OpenRouter loses:** On other providers a `tool_reference` block "reaches the provider as text". The BM25 variant returns 400.
- **Option:** Opaque field, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** Gateway: use the vendor's feature where it exists and emulate it elsewhere, with the emulation's data in the opaque field. Engine: a vendor-neutral deferral flag on a tool offer.
- **New setting or field:** Yes: a deferral flag
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** Medium - the 2026-10-14 report holds the native rules
- **Evidence:** P46 to P53, AN62, AN63

#### Feature 53: Asynchronous tool calls

- **Vendors:** OpenAI (GPT-6: `async: true` on a tool)
- **What it is:** The model keeps working, and the tool's result arrives in a later request.
- **On OpenRouter:** Responses route only. On chat "the flag is removed before the request is routed and the call behaves like any other tool call".
- **What OpenRouter loses:** The feature, for chat callers.
- **Option:** Gateway only, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** Without it the flag is never sent and every call runs synchronously, as today. With it: the Engine's loop gains a vendor-neutral pending tool result, and the Gateway uses the vendor's own form on GPT-6 and a placeholder result elsewhere.
- **New setting or field:** Yes: a pending tool result
- **Source:** Stated by OpenRouter. OpenRouter's own agent SDK does asynchronous tools in the client for any model, with placeholder results, and the Vercel AI SDK and Bifrost keep the flag as per-call data.
- **Confidence:** Low - no project read does it behind a chat-completions gateway
- **Evidence:** OA42, OA43, CT39 to CT41

#### Feature 54: Images in tool results

- **Vendors:** Anthropic, Google, OpenAI
- **What it is:** A tool result may hold images.
- **On OpenRouter:** Content parts on `tool` messages, "only forwarded to supported multimodal models".
- **What OpenRouter loses:** What a text-only model receives.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Gateway decides per model. The Engine's tool results are text today.
- **New setting or field:** Optional: image parts in a tool result
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - what a text-only model receives is unpublished
- **Evidence:** T43

### 3.5 Structured output and message layout: the Gateway can translate both

#### Feature 55: JSON mode and JSON Schema output

- **Vendors:** OpenAI (`response_format`), Anthropic (`output_config.format`), Google (`responseJsonSchema`)
- **What it is:** Three spellings, with different schema subsets.
- **On OpenRouter:** `response_format`, with a per-endpoint `structured_outputs` flag. Enforcement "varies by provider".
- **What OpenRouter loses:** The guarantee.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Gateway translates. Client, Engine, and Lua: a vendor-neutral way to ask for a schema, which the client's nine-key body lacks today.
- **New setting or field:** Yes: a response schema
- **Source:** Stated by OpenRouter.
- **Confidence:** High on the mapping, medium on the guarantee - vendors differ on enforcement
- **Evidence:** T30 to T34, AN76, GG90, L49

#### Feature 56: Schema output on a model without a native mode

- **Vendors:** Claude on Bedrock, others
- **What it is:** The endpoint only writes free text.
- **On OpenRouter:** One page says the "request will fail", another that the "parameter is ignored". Response healing repairs JSON syntax on non-streaming requests only.
- **What OpenRouter loses:** A consistent answer.
- **Option:** Compatibility code (if adopted)
- **Work in Runvil:** Gateway: a forced tool call as the emulation (Bifrost does this on Vertex Anthropic), or check the reply at the boundary and report the downgrade.
- **Source:** From other open-source projects: Bifrost.
- **Confidence:** Medium - one gateway shows the emulation
- **Evidence:** T34, T35, J-table 2

#### Feature 57: Refusals

- **Vendors:** OpenAI (`refusal`), Anthropic (`stop_reason: "refusal"` with `stop_details`)
- **What it is:** The model declines inside an HTTP 200.
- **On OpenRouter:** `message.refusal` with `finish_reason: "content_filter"`. A policy block is HTTP 403 with `error_type: "refusal"`.
- **What OpenRouter loses:** Anthropic's category detail, on the chat completions route.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: map to the standard `refusal` field and `content_filter`. Client: read `refusal`.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented on all three routes
- **Evidence:** T36, O17, AN72, AN73

#### Feature 58: System prompt placement

- **Vendors:** Anthropic (top-level `system`), Google (`systemInstruction`), OpenAI (from GPT-6.2, top-level `instructions` are appended after `input` and billed uncached)
- **What it is:** Each vendor puts system text somewhere else.
- **On OpenRouter:** A `system` message, moved into Anthropic's `system` blocks (shown in an example printed in OpenRouter's documentation).
- **What OpenRouter loses:** How a chat system message is sent to OpenAI is undocumented.
- **Option:** Gateway only
- **Work in Runvil:** Gateway. For OpenAI, send a first `developer` message and leave `instructions` empty, to keep the cache.
- **Source:** Stated by OpenRouter; stated by the vendor.
- **Confidence:** Medium - the OpenAI rule is five days old
- **Evidence:** O67, OA62, T38

#### Feature 59: System messages in the middle of a conversation

- **Vendors:** Anthropic (placement rules on 11 models), others
- **What it is:** A system message between turns.
- **On OpenRouter:** For Claude models OpenRouter says ordinary system text "is still sent as the top-level `system` prompt". Whether a system message later in the conversation stays in place is undocumented.
- **What OpenRouter loses:** Unknown.
- **Option:** Gateway only
- **Work in Runvil:** The Engine keeps system records as the leading block of the list, so the case does not arise.
- **Source:** Inferred about OpenRouter.
- **Confidence:** Medium - inferred, and the Engine avoids the case
- **Evidence:** AN79, AN80

#### Feature 60: Message cleanup rules

- **Vendors:** Anthropic (non-empty text blocks, merged same-role turns), Google (roles `user` and `model`)
- **What it is:** Small validity rules per vendor, each with its own 400.
- **On OpenRouter:** Merged and renamed inside OpenRouter. The rules are unpublished. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: drop empty text blocks and merge turns. The Engine appends an assistant record with empty text for each tool round.
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - each vendor rule is documented with its 400
- **Evidence:** T38 to T40

### 3.6 Vendor-run tools, citations, files, and media: mostly features Runvil has not adopted

Runvil has adopted few of these features. It runs tools in its own Plugins and searches the web through the Gateway's own route, so a tool that the vendor runs is a choice. Each feature below says what that choice would cost.

#### Feature 61: Web search run by the vendor

- **Vendors:** Anthropic, OpenAI, Google, xAI
- **What it is:** Each vendor has its own search tool.
- **On OpenRouter:** One vendor-neutral tool, `{"type": "openrouter:web_search"}`, with `parameters.engine` of `auto`, `native`, or a named engine. `auto` uses the vendor's tool where the endpoint lists it in `native_tools`, else Exa.
- **What OpenRouter loses:** With the vendor's own search, most vendor-neutral settings are ignored, and one model has that search on some provider endpoints only.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Runvil searches through the Gateway's `/v1/tools/web_search` today. A vendor's search can be had with the universal API unchanged in two ways: wrap it as an ordinary tool whose handler makes its own model request, or let the Gateway run the tool loop inside one request. Keeping the vendor's own blocks in the conversation is feature 62.
- **Source:** Stated by OpenRouter. Gemini CLI runs Google Search as a function tool that makes a second request, and OpenRouter, LiteLLM, and Bifrost run the tool loop inside the gateway.
- **Confidence:** Medium - shown by Gemini CLI and three gateways, and nobody has built it for Runvil
- **Evidence:** P10 to P14, AN37 to AN41, GG68, GX13 to GX22, CT35

#### Feature 62: Server tool blocks in history

- **Vendors:** Anthropic (`server_tool_use`, `web_search_tool_result` with `encrypted_content`), OpenAI (`web_search_call` and other call items), Google (`toolCall` and `toolResponse` parts, every one signed)
- **What it is:** The blocks come back unchanged on the next request.
- **On OpenRouter:** Three forms of one event: a `reasoning.server_tool_call` entry on the chat completions route (in the spec only), typed items on the Responses route, Anthropic's blocks on the Messages route.
- **What OpenRouter loses:** On the chat completions route the vendor's own blocks are replaced by a record, counts, and annotations.
- **Option:** Opaque field, reused (if adopted)
- **Work in Runvil:** Gateway only, once the opaque field exists.
- **Source:** Stated by OpenRouter (spec). opencode, TensorZero, and rig send the raw payload back, and LiteLLM keeps the blocks in `provider_specific_fields`.
- **Confidence:** Medium - four open-source projects send the blocks back, and OpenRouter's chat completions route replaces them
- **Evidence:** P26 to P36, GG13, H8, H45, H66, K45, CT28 to CT33, WT31 to WT33, L40

#### Feature 63: A turn the vendor paused

- **Vendors:** Anthropic (`pause_turn`)
- **What it is:** A long server-tool loop stops, and the caller sends the paused reply back unchanged, with the same tools, to continue.
- **On OpenRouter:** In the stop-reason list of the Messages route. The mapping on the chat completions route is undocumented.
- **What OpenRouter loses:** Unknown on the chat completions route.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** It can occur only when a vendor-run tool is offered. The Gateway then resends the paused content itself inside the same client request, and the client sees one reply.
- **Source:** Stated by the vendor; stated by OpenRouter for the stop-reason list only. Pydantic AI continues inside the library (up to 10 segments), and the tool loops that OpenRouter, LiteLLM, and Bifrost run inside the gateway never show a pause to the caller. LiteLLM, Bifrost, and LangChain hand a pause from the vendor to the caller instead.
- **Confidence:** Medium - one library continues it itself, and six projects give five different finish values for it
- **Evidence:** P32, P34, P85, VV8, CT1, CT8, CT12, CT15, CT18, CT20, CT27

#### Feature 64: Citations

- **Vendors:** OpenAI (`url_citation` annotations), Anthropic (citation objects, and citations on supplied documents), Google (grounding metadata, with display duties)
- **What it is:** Each vendor returns its sources its own way.
- **On OpenRouter:** Chat completions route: `message.annotations[]` with a nested `url_citation`. Citations on supplied documents stay on the Messages route.
- **What OpenRouter loses:** Google's search suggestions snippet and the vendors' display duties are unmentioned. Google's terms require showing that snippet "to the end user who submitted the prompt".
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Gateway: map to `annotations` with `url_citation`, which is in OpenAI's own chat reference. Client, Engine, Lua: a way to read annotations on a reply. Google's snippet has a Gemini-only form in every project read, and showing it is the application's duty.
- **New setting or field:** Yes: annotations on a reply
- **Source:** Stated by OpenRouter; stated by the vendor for Google's terms; from other open-source projects: nine open projects, where the snippet reaches the application only inside a raw Gemini object.
- **Confidence:** Medium - the display duties stay with the application
- **Evidence:** P38 to P45, Q47, GX13, GX23

#### Feature 65: Vendor-run code execution and containers

- **Vendors:** Anthropic, OpenAI, Google
- **What it is:** A hosted sandbox. Anthropic and OpenAI return a container id that the caller sends back to keep state.
- **On OpenRouter:** Not wrapped. OpenRouter substitutes its own sandbox tools (`openrouter:shell`, `openrouter:bash`), off the chat route.
- **What OpenRouter loses:** The vendor's feature.
- **Option:** Opaque field, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** Runvil runs code in its own Plugins. If adopted: the container id and result blocks are stored in the opaque field.
- **New setting or field:** Yes, if the feature is adopted
- **Source:** Stated by OpenRouter.
- **Confidence:** Low - no plan to adopt
- **Evidence:** P19, P22, P23, AN48, GX25 to GX28

#### Feature 66: Computer use and other client tools with vendor-fixed schemas

- **Vendors:** Anthropic (bash, text editor, memory, computer), OpenAI (computer)
- **What it is:** The client runs the tool, and the vendor fixes its schema.
- **On OpenRouter:** The Messages route accepts Anthropic's bash and text editor types. Its computer tools are "rejected at validation".
- **What OpenRouter loses:** Computer use.
- **Option:** Gateway only
- **Work in Runvil:** A Runvil tool with its own schema works on every vendor.
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - a design choice
- **Evidence:** P21, AN42, AN43

#### Feature 67: Remote MCP run by the vendor

- **Vendors:** Anthropic (`mcp_servers`, `mcp_toolset`), OpenAI (`mcp` tool with approval items)
- **What it is:** The vendor connects to the MCP server.
- **On OpenRouter:** Responses route only (`mcp`, and `openrouter:mcp`, which was added to the spec on 2026-11-05). The guide tells callers to turn MCP tools into function tools themselves.
- **What OpenRouter loses:** Anthropic's connector.
- **Option:** Gateway only
- **Work in Runvil:** Runvil's MCP Pack runs MCP in the Host.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - Runvil already runs MCP in the Host
- **Evidence:** P80 to P82, AN47

#### Feature 68: Vendor file stores and file search

- **Vendors:** Anthropic (Files API), OpenAI (files, vector stores, `file_search`), Google (File API URIs)
- **What it is:** Upload once and refer by id. The store belongs to one vendor account.
- **On OpenRouter:** OpenRouter's own Files API with `or_file_` ids. `?provider=openai` or `?provider=anthropic` forwards to the vendor's store with the caller's own vendor key. `file_search` "requires BYOK" (the caller's own vendor key).
- **What OpenRouter loses:** A vendor's id works only with that vendor.
- **Option:** Separate vendor path (if adopted)
- **Work in Runvil:** A vendor-specific upload route and a vendor's id inside message content. The Gateway alone could handle it if it owned a vendor-neutral files route and mapped ids, at the price of stored data in the Gateway.
- **New setting or field:** Yes, if the Gateway handles it
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - handling it in the Gateway needs stored data in the Gateway
- **Evidence:** P67 to P70, AN56, AN57, OA51, GG73

#### Feature 69: PDF and document input

- **Vendors:** Anthropic, OpenAI, Google natively; other models need text
- **What it is:** A `file` or document part.
- **On OpenRouter:** A `file` part on the chat completions route. The `file-parser` plugin picks `native`, `mistral-ocr`, or `cloudflare-ai`. The feature survives whole.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Gateway: pass natively or parse. Engine: a `file` content part beside `text` and `image_url`.
- **New setting or field:** Yes: a file part
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - the quality of parsed text is untested
- **Evidence:** P63 to P66

#### Feature 70: Image, audio, and video input

- **Vendors:** All vision vendors; Google for video
- **What it is:** A URL or inline data, with per-provider limits. For video, YouTube links work on Google AI Studio only, and Vertex takes inline data only.
- **On OpenRouter:** `image_url`, `input_audio`, `video_url` parts. Video URLs go only to providers that support them.
- **What OpenRouter loses:** Whether OpenRouter downloads a URL for an inline-only provider.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: turn a URL into inline data where the vendor needs it (LiteLLM forces base64 on Bedrock and Vertex). Engine: part types for audio and video.
- **New setting or field:** Yes, for audio and video
- **Source:** Stated by OpenRouter; from other open-source projects: LiteLLM.
- **Confidence:** Medium - per-provider limits are unpublished
- **Evidence:** P71 to P75, P86, GG74, GG76

#### Feature 71: Image and audio output from a chat model

- **Vendors:** Google, OpenAI
- **What it is:** Image or audio parts inside the reply.
- **On OpenRouter:** `modalities`, `message.images[]`, `delta.audio`: extension fields outside OpenAI's message format.
- **What OpenRouter loses:** How per-image signatures are kept is unpublished.
- **Option:** Opaque field, reused, for the signatures. The rest needs the Gateway only, plus a new vendor-neutral field (if adopted)
- **Work in Runvil:** Client: a reply whose `content` is not a string fails today. Engine and Lua have to receive media.
- **New setting or field:** Yes: media in a reply
- **Source:** Stated by OpenRouter.
- **Confidence:** Low - 13 and 4 of 476 catalog entries
- **Evidence:** P76, P77, GG96

#### Feature 72: Safety settings and blocked prompts

- **Vendors:** Google
- **What it is:** `safetySettings` in Google's own categories and thresholds. A blocked prompt is an HTTP 200 with an empty candidate list and `promptFeedback.blockReason`.
- **On OpenRouter:** The chat route leaves the settings out. A block becomes an error with `error_type: "content_policy_violation"`, status 403.
- **What OpenRouter loses:** The control.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: a default on a model entry, set by the operator, and a choice that the Gateway builds, with `finish_reason: "content_filter"` where Google sent an empty candidate list. A prompt that set Google's thresholds itself would have to know the vendor.
- **Source:** Inferred about OpenRouter: the spec has `provider.options` and no safety field. new-api and one-api send a gateway default, and LiteLLM, agentgateway, Bifrost, and Pydantic AI turn a blocked prompt into a `content_filter` choice. No project read offers a vendor-neutral setting.
- **Confidence:** Medium - nine projects, all with Google's own setting names
- **Evidence:** GG59, GG60, GG64, GG65, GX1 to GX12

#### Feature 73: Tools that exist at one vendor only

- **Vendors:** Google (URL context, Maps grounding, code execution beside functions), xAI (X search), Moonshot (formula tools), Alibaba (`enable_search`), Zhipu (`web_search` tool type)
- **What it is:** A tool or switch that only this vendor has, some with display duties on the application.
- **On OpenRouter:** X search is a vendor object inside the vendor-neutral search tool. The others are undocumented or unreachable.
- **What OpenRouter loses:** The features.
- **Option:** Opaque field, plus a new vendor-neutral setting or field (if adopted). This feature comes closest to making a prompt know its vendor
- **Work in Runvil:** A prompt may ask for a capability by a vendor-neutral id. The Gateway uses the vendor's tool where the vendor has one and answers "unavailable" elsewhere. A prompt that names the vendor's tool would have to know the vendor.
- **New setting or field:** Yes: a vendor-neutral id for a capability
- **Source:** Stated by OpenRouter for X search; inferred about OpenRouter for the gaps. Pydantic AI has vendor-neutral tool classes for web search, web fetch, and code execution that map to each vendor's tool. Maps grounding has a vendor-neutral form in no project read.
- **Confidence:** Low - no plan to adopt
- **Evidence:** P15, GG69, GG70, OV17, OV42, OV52, OV56, GX13 to GX28

### 3.7 Data and jobs kept on the vendor's servers: a tool loop with reasoning works without any of them

This group has four of the five features that would need a separate vendor path, and the fifth is feature 68. A tool loop with reasoning works without any of them, and OpenRouter declines most of them too.

#### Feature 74: Conversations stored by the vendor

- **Vendors:** OpenAI (`previous_response_id`, `store`, Conversations), xAI, Google (Interactions API)
- **What it is:** The vendor stores the conversation and the caller sends an id.
- **On OpenRouter:** Rejected: "Requests that set `store: true` or a non-null `previous_response_id` are rejected with a `400` error". Every route takes the whole conversation in each request, and the Interactions API is not offered.
- **What OpenRouter loses:** Stored conversations as a whole.
- **Option:** Separate vendor path (if adopted)
- **Work in Runvil:** A store, and routes that read from it. Sending the whole conversation each time, with the opaque field, gives the model the same reasoning. The run log records each request's messages, and a request that named a stored conversation would leave them out.
- **Source:** Stated by OpenRouter. Bifrost and agentgateway forward or drop the id. LiteLLM alone emulates it, by rebuilding history from its own request log database. The three projects found to support Google's Interactions API each give it an entry point separate from ordinary chat requests.
- **Confidence:** High - OpenRouter declines, and the one emulation needs a database
- **Evidence:** OA1, OA2, OA24, GG79, J-table 2, CT44 to CT46, GX64 to GX66

#### Feature 75: Background runs, polling, cancel, webhooks

- **Vendors:** OpenAI
- **What it is:** A long-running job with a retrieve route.
- **On OpenRouter:** `background` is a boolean documented by its type only, and the spec has one Responses path.
- **What OpenRouter loses:** The job mode.
- **Option:** Separate vendor path (if adopted)
- **Work in Runvil:** A job id and a second route.
- **Source:** Stated by OpenRouter (spec); inferred about OpenRouter.
- **Confidence:** High - the spec has one Responses path
- **Evidence:** OA57, OA58

#### Feature 76: Batch jobs

- **Vendors:** Anthropic, OpenAI
- **What it is:** Asynchronous batches at half price.
- **On OpenRouter:** OpenRouter's own `/batches` route with `:batch` model variants.
- **What OpenRouter loses:** Vendor beta features inside a batch.
- **Option:** Separate vendor path (if adopted)
- **Work in Runvil:** A separate route. The Engine sends every round as a live request and waits for the reply.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - a separate route at every vendor
- **Evidence:** AN58, AN59, OA59

#### Feature 77: Compaction done by the vendor

- **Vendors:** Anthropic (`compact_20260112`, `compaction` blocks, stop reason `compaction`), OpenAI (compaction items)
- **What it is:** The vendor summarizes and returns an opaque block that the caller sends back.
- **On OpenRouter:** Messages and Responses routes only. On the chat completions route the nearest thing is OpenRouter's own `context-compression` plugin.
- **What OpenRouter loses:** The feature, for chat callers. Anthropic's second, on-demand form (beta `compact-2026-09-04`, one signed block) is outside OpenRouter's Messages schema.
- **Option:** Opaque field, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** In Runvil only a compactor shortens the conversation, and the Engine's rule is that earlier messages change only at an explicit compaction. If adopted as one more compactor: the block is stored in the opaque field.
- **New setting or field:** Yes: a way for a compactor to ask for the vendor's summary, not yet designed
- **Source:** Stated by OpenRouter; from other open-source projects: LiteLLM (`provider_specific_fields.compaction_blocks`, and it builds the same block itself for other vendors).
- **Confidence:** Medium - only LiteLLM stores the block, and vendor compaction conflicts with the Engine's rule that only a compactor changes earlier messages
- **Evidence:** AN50, AN52, OA46, K6, CT4, CT10

#### Feature 78: Context editing done by the vendor

- **Vendors:** Anthropic (`clear_tool_uses_20250919`, `clear_thinking_20251015`)
- **What it is:** Applied before the prompt reaches the model. The client keeps the full history.
- **On OpenRouter:** Messages route only.
- **What OpenRouter loses:** The feature, for chat callers.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** A Gateway setting on a model entry.
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - one vendor, one request field
- **Evidence:** AN51

#### Feature 79: Token counting

- **Vendors:** Anthropic (`count_tokens`)
- **What it is:** A route that estimates input tokens.
- **On OpenRouter:** Left out: the spec's 123 paths stop short of a token-counting route.
- **What OpenRouter loses:** The route.
- **Option:** Gateway only, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** A vendor-neutral Gateway route. The Engine estimates, then corrects from reported usage.
- **New setting or field:** Yes, if the feature is adopted
- **Source:** Inferred about OpenRouter from the spec.
- **Confidence:** Medium - inferred from a missing route
- **Evidence:** AN86

#### Feature 80: Service tiers and fast mode

- **Vendors:** OpenAI (`service_tier`), Anthropic (`speed: "fast"`)
- **What it is:** Price and latency tiers.
- **On OpenRouter:** Vendor-neutral `service_tier` values, each tier a separate endpoint, "fully interchangeable on all APIs". The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** A Gateway model entry. The `fast` role can point at a fast-tier entry.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented as interchangeable values
- **Evidence:** OA66, OA67, AN84, AN85

#### Feature 81: Predicted outputs, and automatic truncation

- **Vendors:** OpenAI
- **What it is:** `prediction` on a few older models. `truncation: "auto"` drops old items.
- **On OpenRouter:** `prediction` on the chat completions route, `truncation` on the Responses route, and OpenRouter's own middle-out trimming. The feature survives whole.
- **Option:** Gateway only (if adopted)
- **Work in Runvil:** Decline both. Silent trimming conflicts with the Engine's own size check before each request.
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - both are fading vendor features
- **Evidence:** OA48, OA68, O60

#### Feature 82: Skills, and code the model writes to call tools

- **Vendors:** Anthropic (`container.skills`), OpenAI (programmatic tool calling: `program` items with a fingerprint "that must be round-tripped")
- **What it is:** Vendor-held code and containers, with ids or fingerprints the caller returns.
- **On OpenRouter:** `container.skills` is in the response schema of the Messages route only. Programmatic tool calling is outside the tool list of the Responses route.
- **What OpenRouter loses:** The features.
- **Option:** Opaque field, plus a new vendor-neutral setting or field (if adopted)
- **Work in Runvil:** The id or fingerprint would be stored in the opaque field.
- **New setting or field:** Yes: a vendor-neutral id that the Gateway maps to the vendor's own, not yet designed
- **Source:** Stated by the vendor; inferred about OpenRouter from the spec.
- **Confidence:** Low - no plan to adopt
- **Evidence:** AN88, OA55

#### Feature 83: WebSocket transport

- **Vendors:** OpenAI
- **What it is:** A second transport with its own events.
- **On OpenRouter:** Not offered for inference.
- **What OpenRouter loses:** The transport.
- **Option:** Separate vendor path (if adopted)
- **Work in Runvil:** Not examined.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - a different transport
- **Evidence:** OA6, OA72

### 3.8 Finish reasons, errors, usage, and streaming: all Gateway work, and where today's Gateway loses the most

The Gateway alone can handle every feature in this group. This is also where Runvil's Gateway loses the most today, because it forwards the chunks it understands and drops the rest (section 7).

#### Feature 84: Finish reason vocabulary

- **Vendors:** Anthropic (eight values), Google (23), OpenAI Responses (`status` plus `incomplete_details.reason`)
- **What it is:** Each vendor names its stops its own way.
- **On OpenRouter:** Chat completions route: five normalized values (`stop`, `length`, `tool_calls`, `content_filter`, `error`) plus `native_finish_reason`. A full mapping table is unpublished.
- **What OpenRouter loses:** The native value is outside the spec's chat schemas and outside the Messages and Responses routes. In recordings it varies with the upstream protocol for OpenAI models (`stop` on some, `completed` on others), and for Claude it changed from normalized words in 2025 to Anthropic's own words in 2026.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: map to the standard words. An output-only `native_finish_reason` beside them is optional.
- **Source:** Stated by OpenRouter; inferred from recorded traffic. The mapping is taken from LiteLLM and the Vercel AI SDK, which disagree on `pause_turn`, `compaction`, and `MALFORMED_FUNCTION_CALL`.
- **Confidence:** High - documented, with the gaps shown by recordings
- **Evidence:** O52 to O57, O77, AN71, GG61, WT16, WT44, VF4

#### Feature 85: Tool calls reported under the wrong finish reason

- **Vendors:** Google (`STOP` with function calls)
- **What it is:** Gemini reports `STOP` for a reply that ends in function calls.
- **On OpenRouter:** Not documented. Two observations disagree: a chunk that a user posted on 2025-12-18 shows `tool_calls` beside a native `STOP`, and OpenRouter's own SDK changes `stop` to tool calls in the client.
- **What OpenRouter loses:** Reliability.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: report `tool_calls` whenever the reply holds a call. The Engine already starts a tool round whatever the finish reason.
- **Source:** Inferred about OpenRouter from the official SDK's source.
- **Confidence:** High - the Gateway's rule does not depend on which observation is typical
- **Evidence:** O58, T17

#### Feature 86: Error envelope and error classes

- **Vendors:** Anthropic (`error.type`, status 529), OpenAI (`error.code`), Google (status names), Zhipu (numeric codes), Mistral (a different format by error and by route), MiniMax (an earlier Runvil survey says `base_resp` inside an HTTP 200, which could not be confirmed again on 2026-11-22)
- **What it is:** Every vendor encodes the same conditions differently.
- **On OpenRouter:** `error_type` with 27 values, at a different path on each route. The vendor's code is in `metadata.provider_code` and its payload in `metadata.raw` "when available".
- **What OpenRouter loses:** The vendor's status. A 500 masks the message.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: forward the status, a normalized type, the vendor's code and message, and `retry-after`. Today it sends `upstream returned {status}`. Client: classify by the typed code first and keep the phrase rules as a fallback.
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - documented, and Runvil's side was read in code
- **Evidence:** O33 to O41, AN91, OV34, OV57, OV62, L74

#### Feature 87: Errors after HTTP 200

- **Vendors:** Anthropic (`event: error`), OpenAI (`response.failed`), any
- **What it is:** A failure inside a stream that already started.
- **On OpenRouter:** Chat completions route: a chunk with a top-level `error` and `finish_reason: "error"`. Non-streaming requests can also return a 200 body with `error`.
- **What OpenRouter loses:** Whether `[DONE]` follows the error chunk is not stated. In two recordings from 2025-11 the error chunk's `finish_reason` was `null` or `"stop"`, and `[DONE]` followed in one of them.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: turn the vendor's event into a typed envelope event. Client: treat any chunk with an `error` as a failure and classify it by its type, where today it defaults to a transport failure.
- **Source:** Stated by OpenRouter; inferred about OpenRouter from recorded traffic.
- **Confidence:** High - documented and seen in recordings
- **Evidence:** O42 to O45, L85, WT34

#### Feature 88: Context overflow signal

- **Vendors:** Anthropic (message text), OpenAI (`context_length_exceeded`), Google, Zhipu (code 1261)
- **What it is:** Each vendor reports an over-long prompt differently.
- **On OpenRouter:** `error_type: "context_length_exceeded"`, status 400.
- **What OpenRouter loses:** On output, "context full" and "own cap reached" both show as `length`. A recording of 2026-11-18 shows OpenRouter's own pre-send overflow rejection as a 400 whose body lacks `error_type`.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: one typed code. This is what lets the Engine's compactor run for a remote model.
- **Source:** Stated by OpenRouter; inferred about OpenRouter from recorded traffic; earlier Runvil research.
- **Confidence:** High - documented, with one recorded exception
- **Evidence:** O63, O64, O76, WT39

#### Feature 89: Rate limits and retry hints

- **Vendors:** All
- **What it is:** A 429 with vendor headers and four reset formats.
- **On OpenRouter:** 429 with `Retry-After` when every attempted provider gave a hint.
- **What OpenRouter loses:** The vendors' rate-limit headers, which is inferred.
- **Option:** Gateway only
- **Work in Runvil:** Gateway forwards `retry-after`. The retry itself belongs in the client.
- **Source:** Stated by OpenRouter for the 429. Inferred about OpenRouter for the lost headers, from its documentation's silence and from 202 replies recorded on 2026-11-18.
- **Confidence:** High - documented, and recordings hold OpenRouter's own headers only
- **Evidence:** O30 to O32, WT41

#### Feature 90: Keep-alive and idle limits

- **Vendors:** Anthropic (`ping` events), others silent
- **What it is:** A slow stream needs a heartbeat.
- **On OpenRouter:** An SSE comment, `: OPENROUTER PROCESSING`. The interval and any timeout number are unpublished. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: forward a heartbeat and add an idle deadline.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented, and Runvil's side was read in code
- **Evidence:** O49, O50, L60, WT35

#### Feature 91: Usage delivery in a stream

- **Vendors:** OpenAI (a last chunk with empty `choices`, on request), Anthropic (split across two events)
- **What it is:** Each vendor puts usage in a different place.
- **On OpenRouter:** Always sent. The last chunk has one choice with a content-free delta that repeats the finish reason.
- **What OpenRouter loses:** OpenAI's own chunk layout.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: stop skipping the usage-only chunk (section 7) and emit usage in one fixed place.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented and seen in recordings
- **Evidence:** C33, C38, L57, WT36

#### Feature 92: Reasoning token counts

- **Vendors:** All
- **What it is:** Each vendor counts thinking tokens under its own name, and Gemini leaves them out of its candidate count.
- **On OpenRouter:** `completion_tokens_details.reasoning_tokens`. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway arithmetic. Engine: stop subtracting reasoning tokens from the context estimate once the opaque field is replayed.
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - plain arithmetic
- **Evidence:** R71, GG94

#### Feature 93: Cancellation and billing

- **Vendors:** 24 named providers stop billing when the caller aborts, and 24 others do not
- **What it is:** The outcome depends on the provider.
- **On OpenRouter:** Abort the connection.
- **What OpenRouter loses:** One model bills differently on Anthropic (stops) and on Bedrock (does not).
- **Option:** Gateway only
- **Work in Runvil:** Gateway: closing the upstream connection is already the mechanism.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - the provider lists are documented
- **Evidence:** O51, V16

### 3.9 Model discovery, routing, and pass-throughs for vendor settings: operator settings in the Gateway

OpenRouter's routing controls have little to teach a gateway with one operator. Its two pass-throughs for vendor settings, beta headers and `provider.options`, are the part to avoid: both make the caller write a vendor's own setting names.

#### Feature 94: Capability metadata per model

- **Vendors:** All
- **What it is:** Vendors publish limits unevenly. Gemini's model list gives one boolean for thinking.
- **On OpenRouter:** `supported_parameters` (the union across a model's endpoints), a `reasoning` object, and per-endpoint `supports_tool_choice`, set by OpenRouter's own capability tests.
- **What OpenRouter loses:** One model id supports a parameter on one provider and not on another: `deepseek/deepseek-v3.2` lists `tools` on 2 of 4 endpoints.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: apply the model sheet's values when it handles a request (today it only publishes them), and read OpenRouter's `supported_parameters` and `reasoning` object into the model sheet.
- **New setting or field:** Optional: more role requirements than `min_context` and `thinking`
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - the union rule held on 25 of 25 models
- **Evidence:** M12, M24, M50, T4, L91, L94

#### Feature 95: Several providers for one model

- **Vendors:** Anthropic direct, Bedrock, Vertex, Azure; providers that serve open-weight models
- **What it is:** Each provider has its own features, price, and limits.
- **On OpenRouter:** A `provider` object with thirteen documented fields. The default is a weighted random pick.
- **What OpenRouter loses:** Under one model id the providers differ in native web search, fast mode, cache behavior, structured output, and billing after a cancel.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: an endpoint list per model entry, chosen by the operator. Features 5 and 36 need the choice to stay the same for a conversation.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - public endpoint data
- **Evidence:** O1 to O12, AN94 to AN97, P12

#### Feature 96: Vendor beta headers and API versions

- **Vendors:** Anthropic (`anthropic-beta`, `anthropic-version`)
- **What it is:** A header switches a feature on.
- **On OpenRouter:** `x-anthropic-beta`, passed through an unpublished per-provider allowlist that "drops every other value".
- **What OpenRouter loses:** The caller has to know Anthropic's header values.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: attach the header for each feature it uses. A header set by a prompt would make the prompt know the vendor.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - the allowlist is documented
- **Evidence:** AN81 to AN83, O13

#### Feature 97: Passing a vendor's own parameters through

- **Vendors:** All
- **What it is:** Parameters that only one vendor names.
- **On OpenRouter:** `provider.options[<provider slug>]`, filtered by an unpublished per-endpoint allowlist; unknown keys are dropped.
- **What OpenRouter loses:** The caller writes vendor parameter names and the provider's id.
- **Option:** Gateway only, as long as the operator sets it in the Gateway's configuration
- **Work in Runvil:** Gateway: an extras table on a model entry, set by the operator. Offering the same thing to a prompt would make the prompt know the vendor.
- **Source:** Stated by OpenRouter.
- **Confidence:** Medium - a design choice
- **Evidence:** O3, S46

#### Feature 98: End-user identity

- **Vendors:** Anthropic (`metadata.user_id`), OpenAI (`safety_identifier`)
- **What it is:** An opaque id for abuse handling. On GPT-6.2 a changed id makes the API drop replayed reasoning.
- **On OpenRouter:** `user`, folded "into the hashed identity sent upstream" and never forwarded raw.
- **What OpenRouter loses:** The raw value.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: one stable hashed id per client.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented
- **Evidence:** O69, AN87, OA77

#### Feature 99: Credentials, regions, and retention

- **Vendors:** All
- **What it is:** Eleven kinds of credential, regional providers, and retention terms.
- **On OpenRouter:** Provider keys stored per workspace, `provider.data_collection`, `provider.zdr`, regional hostnames.
- **Option:** Gateway only
- **Work in Runvil:** Gateway configuration.
- **Source:** Stated by OpenRouter; earlier Runvil research.
- **Confidence:** High - configuration only
- **Evidence:** O25 to O29

#### Feature 100: Fallback to another model or provider

- **Vendors:** Anthropic (server-side `fallbacks`), any
- **What it is:** A second model or provider when the first fails or refuses.
- **On OpenRouter:** `models` array on the chat completions and Responses routes; `fallbacks` on the Messages route, handled by OpenRouter and not by Anthropic. Failover continues after the 200 until the first token reaches the caller.
- **What OpenRouter loses:** Anthropic's `fallback` content block, and `fallback_credit_token`, a beta request field that the schema of OpenRouter's Messages route leaves out.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: the planned choice among a model's endpoints. A fallback model is the Host's choice, through roles.
- **Source:** Stated by OpenRouter.
- **Confidence:** High - documented
- **Evidence:** O15 to O18, O47, AN74, S33

### 3.10 Vendors with OpenAI-style APIs and local servers: small fixes that matter today

These features matter today, before any vendor protocol is added, because Runvil's Gateway already forwards chat completions requests unchanged to these servers.

#### Feature 101: OpenAI-style routes that differ from OpenAI

- **Vendors:** Mistral (`/v1` "uses the native Mistral format"), MiniMax (`base_resp.status_code` inside the body), xAI (chat completions is labeled "(Deprecated)" and "Function calling only"), DeepSeek (extra finish reasons), and OpenRouter itself when it is the Gateway's upstream (reasoning state in stream fragments, an error chunk inside a 200)
- **What it is:** The format matches and the meaning differs.
- **On OpenRouter:** Hidden inside OpenRouter's translator.
- **Option:** Compatibility code
- **Work in Runvil:** Gateway: small per-vendor adapters inside the `openai` upstream, keyed by endpoint. Each is a few lines. The OpenRouter adapter is the largest: it merges `reasoning_details` fragments into the opaque field. Runvil's provider file records Mistral's `/v1` as its OpenAI base URL.
- **Source:** Stated by the vendor; earlier Runvil research.
- **Confidence:** Medium - each quirk is documented, and the adapters are unwritten
- **Evidence:** OV13, OV27, OV35, OV62, OV80

#### Feature 102: Thinking switch spellings on OpenAI-style routes

- **Vendors:** DeepSeek, Qwen, GLM, MiniMax (`thinking.type`, `enable_thinking`), llama.cpp and vLLM (`chat_template_kwargs`, `reasoning_format`)
- **What it is:** Each server spells the switch and the budget its own way. pi keeps eleven named thinking formats for "OpenAI-compatible" endpoints.
- **On OpenRouter:** One `reasoning` object. The feature survives whole.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: a table per endpoint.
- **Source:** Stated by OpenRouter; from other open-source projects: pi.
- **Confidence:** High - pi's table shows the spread
- **Evidence:** OV78, OV21, OV46, OV47

#### Feature 103: Capability discovery on a local server

- **Vendors:** llama.cpp (`/props` with `chat_template_caps`)
- **What it is:** The loaded template decides whether tools, parallel calls, and effort levels work.
- **Option:** Gateway only
- **Work in Runvil:** Gateway: read `/props` when the model starts, as it already does to detect the model's tool format.
- **Source:** From other open-source projects: llama.cpp.
- **Confidence:** High - read in llama.cpp's source
- **Evidence:** OV68

#### Feature 104: Silent failures on a local server

- **Vendors:** llama.cpp
- **What it is:** A named `tool_choice` becomes `auto`, tools are ignored by a template that cannot render them, and a grammar sent with tools is a 400.
- **Option:** Compatibility code
- **Work in Runvil:** Gateway: check before forwarding. Appendix C of the 2026-09-20 report lists the cases.
- **Source:** From other open-source projects: llama.cpp; earlier Runvil research.
- **Confidence:** Medium - two of the cases were re-read today
- **Evidence:** OV72

#### Feature 105: Local Messages and Responses routes

- **Vendors:** llama.cpp, vLLM
- **What it is:** Both serve `/v1/messages` and `/v1/responses`, and llama.cpp returns reasoning with an empty signature. Earlier Runvil reports say vLLM sends a random one, which was not checked again.
- **Option:** Opaque field, reused
- **Work in Runvil:** Gateway: the opaque field's label stops a block from a local server from being sent to Anthropic.
- **Source:** From other open-source projects: llama.cpp, vLLM.
- **Confidence:** High - the label check compares the model and never reads the signature
- **Evidence:** OV71, OV76, W19

## 4. Runvil should copy OpenRouter's rule for sending reasoning data back and three of its field names, and should not copy `reasoning_details` as its opaque field

The Gateway's written rule is that an extension uses "a name inside the standard's own namespace", so a name that others already use has value. Table 5 lists OpenRouter's extension names, who else uses each, and what Runvil should do. The counts come from other gateways' documentation, read on 2026-11-22 [Q, VT]. The notes under the table give the detail for each name.

*Table 5. OpenRouter's extension names: who else uses them, and what Runvil should do.*

| OpenRouter's name | Others that use it | Advice for Runvil | Confidence |
|---|---|---|---|
| `prompt_tokens_details.cache_write_tokens` | OpenAI itself, five gateways | Use it | High |
| `annotations` with `url_citation` | OpenAI itself, Open Responses | Use it when citations are wanted | High |
| `reasoning_effort` | OpenAI's standard chat field | Use it | High |
| `reasoning` string on a message | Nine parties | Read it and `reasoning_content` | Medium |
| `reasoning_details` array | Two gateways in full, two in part | Accept on input only | Medium |
| `native_finish_reason` | Two copies of OpenRouter's API | Optional, output only | Medium |
| `usage.cost` | Six gateways | Optional, with the unit stated | Medium |
| `cache_control` on content parts | Four parties | Let it pass through | High |
| `provider` object, `models` array | Resellers | Leave out | High |
| `plugins`, `transforms`, model id suffixes | Copies only | Leave out | High |

Source: evidence items Q30 to Q48, and their second reading, items VT1 to VT7.

- **`prompt_tokens_details.cache_write_tokens`.** It is in OpenAI's own Chat Completions reference today, so it is part of the standard. LLM Gateway, Bedrock for OpenAI models, agentgateway, Bifrost, and Envoy AI Gateway use it too. Confidence: high - read in OpenAI's live reference.
- **`annotations` with `url_citation`.** It is a standard field in OpenAI's reference and in the Open Responses schema.
- **`reasoning_effort`.** The top-level field is OpenAI's standard. OpenRouter's object form, `reasoning.effort`, is also used by Vercel AI Gateway, LLM Gateway, Infron, and Ollama, and the Gateway should accept it on input.
- **`reasoning` string.** Nine parties use `reasoning`, among them vLLM (which renamed it from `reasoning_content`), Vercel AI Gateway, Groq, Cerebras, and Together. Eleven use `reasoning_content`, among them DeepSeek and llama.cpp. The client reads both today. The Gateway should let the upstream's name pass through and settle on one name only when a third-party client needs it. Confidence: medium - the two names are still used about equally.
- **`reasoning_details`.** Vercel AI Gateway copies it field for field, and LLM Gateway adds two fields. Bifrost uses the name and the type strings and leaves out the `format` value, and Infron documents the name only. MiniMax's own pages name `reasoning_content` today, so the earlier count of five adopters is four. The Gateway should accept the array on input as a second-best source when the opaque field is absent. Its rule is right: the data goes back whole, unchanged, and in order. Its format is weak in the ways listed under "What to avoid". Confidence: medium - it is the most widely used structured format for reasoning data.
- **`native_finish_reason`.** The idea is a raw value beside the normalized one. Two copies of OpenRouter's schema have it, and one reseller strips it. The Engine should never branch on it. Confidence: medium - it is cheap, and it is the only name in use for this.
- **`usage.cost`.** Six small and mid-size gateways have it, with different units.
- **`cache_control` on content parts.** Vercel AI Gateway, Infron, LiteLLM, and Alibaba accept it. Runvil's own client should not need it, because the Gateway places the markers. Confidence: high - the harnesses surveyed place markers in a gateway or a library.
- **`provider` preferences and `models`.** Resellers that copy OpenRouter's API use them, and Vercel in part. One operator configures the Gateway, so the universal API leaves them out.
- **`plugins`, `transforms`, and model id suffixes such as `:thinking` and `:online`.** OpenRouter itself has deprecated `transforms`, `:thinking`, and `:online`.

### What to take from OpenRouter: its rule for sending data back, Gateway data in the same field, tested capability flags, and an option that shows the translated request

**The rule for sending data back.** The caller stores a reply's reasoning data unread and sends it back whole, in order, on the same assistant message. Runvil's design already says this. Confidence: high - Anthropic, OpenAI, and Google each require it.

**Gateway data in the same field.** OpenRouter stores its own data for deferred tools, vendor-run tools, and Gemini 4 continuation tokens in the entries the caller already returns [R42, P50, GG41]. A gateway that does this keeps its data in the conversation itself. For Runvil it means the Gateway can emulate tool search or keep a cache handle in the opaque field. Confidence: high - documented, with named error codes for when the data is missing.

**Capability flags that come from tests.** OpenRouter's catalog has a `reasoning` object per model (`mandatory`, `supported_efforts`, `default_effort`) and a per-endpoint `supports_tool_choice` object, and it says tool and schema support is "never enabled from the feed, only by the capability tests passing" [T4, M24]. Runvil's `/v1/models` already publishes `thinking`, `effort_levels`, and `default_effort`. The Gateway should apply them when it handles a request, and trust tests over vendor lists. Confidence: high - the flags are public data, checked on 25 models.

**An option that returns the translated request.** `debug: {"echo_upstream_body": true}` returns the body that OpenRouter sent to the provider, in the first chunk of a stream [O65 to O67]. It is the one view into OpenRouter's translator, and a translating Gateway needs the same view for its own tests. It belongs in development builds only. Confidence: medium - it is useful, and it can leak request contents if left on.

### What to avoid in OpenRouter's design: it loses the order of blocks, leaves joining and vendor settings to the caller, splits features across three routes, and changes requests silently

**A list of reasoning entries beside separate text and tool-call fields.** This is the main flaw of `reasoning_details` as an opaque field. It loses the order of blocks inside a reply, which Anthropic's prefix check and Gemini's part signatures both depend on. LLM Gateway had to add `block_index` and `google_part` to repair it [Q31, VT2]. Runvil's ordered block list keeps that order. Confidence: high - every open-source gateway read that uses the three-field format loses the order [J-table 2, VF16].

**Data that the client joins from stream fragments.** OpenRouter streams `reasoning_details` in pieces and leaves the joining rule to the client. Its own SDK needed nine dated fixes in 2026 to this code, and one way to lose data is still open. That SDK joins by type change and says the `index` "cannot be used to reassemble the array", while rig and LangChain join by `index` [F1 to F3, F59, F60, VF6, VT9]. Runvil's Gateway should build the list and send it once, on the final chunk. Confidence: high - the fix history is in the SDK's own source.

**Readable entries that clients edit.** The entries have named types and readable fields, so clients change them. OpenRouter's own SDK strips unsigned Anthropic and Gemini text entries, removes duplicates across messages, and drops any entry whose `format` is outside the six values it knows, while the spec has twelve. LangChain's OpenRouter client removes the `id` of entries whose id starts with `rs_` [F6 to F8, F13, H42, VF6, VF9]. An opaque field that is one labeled value gives a client one thing to copy. Confidence: high - read in code and confirmed by running the SDK's schema.

**The provider written into the vendor's data.** OpenRouter appends its endpoint name to the signature or the encrypted data, so the stored value is OpenRouter's and differs from the vendor's [R41, G37]. Runvil's label belongs in the opaque field beside the blocks. Confidence: medium - the suffix is inferred from decoded samples.

**Vendor settings that the caller writes.** `x-anthropic-beta`, `provider.options[<slug>]`, `reasoning.mode: "pro"`, the `x_search` object, and the video part's `processing: "agentic"` all ask the caller for a vendor's own names [AN82, O3, R7, P15]. In Runvil the caller would be a Lua prompt, which would then have to know the vendor. These settings belong on the operator's model entry in the Gateway's configuration. Confidence: high - OpenRouter documents each one as a caller-side setting.

**Different features on each of three routes.** OpenRouter serves every chat model on its chat completions, Responses, and Messages routes. Ten features work on only one or two of them: `between_tools`, `drop_block`, vendor compaction, context editing, document citations, `phase`, asynchronous tools, custom tools, shell, and tool changes [R62, AN52, OA41, OA43, T37]. Shell works on the Messages and Responses routes, and a request for it on the chat completions route gets an HTTP 400. A caller that wants one of the ten picks the route by vendor. One harness, pi, already moved every `anthropic/*` model to the Messages route to keep signed thinking [F40]. Runvil's one route avoids this, provided the opaque field can hold what those routes hold. Confidence: high - OpenRouter's own guides say which route each feature needs.

**Silent changes to a request.** OpenRouter accepts a thinking budget and drops it, strips `strict` when a header is missing, drops beta header values outside its allowlist, turns "thinking off" into the lowest effort on alias ids, and trims the middle of an over-long prompt by default on small-context endpoints [S49, AN82, R15, O60]. The Gateway already has a `gateway_warning` field, and it should report a dropped or changed setting there. Confidence: medium - reporting costs little, and how often a prompt author would read it is unknown.

## 5. Where OpenRouter's callers still have to know the vendor, Runvil's Gateway will face the same problem

Each case below is a place where the largest chat completions API over many vendors could not hide the vendor. There are six kinds.

**1. A vendor-neutral request that some models refuse.** OpenRouter's stated rule is that an unsupported parameter is ignored. Its documentation also gives 22 cases in which OpenRouter rejects the request itself (18 with an HTTP 400, one with a 403, and three with the status left unstated), and more in which it passes the provider's 400 through [S41 to S48]. The ones that matter for a tool loop:

- Turning reasoning off on a model flagged `mandatory`: `Reasoning is mandatory for this endpoint and cannot be disabled.` [R14].
- A forced `tool_choice` on Fable 5.2, Mythos 5.2, Opus 5.5, Sonnet 5.5, Fable 5.1, Mythos 5.1: "the provider's 400" [T10].
- A final assistant message on Claude 4.6 and later: the provider's 400 [AN69].
- A changed prefix under Claude thinking that is sent back: "OpenRouter passes the upstream error through and does not drop thinking blocks for you" [R62].
- `effort` together with `max_tokens`, `store: true`, a non-null `previous_response_id`, `tool_choice` with deferred tools, more than three `fallbacks` [R3, OA1, T12, AN74].

*For Runvil:* the same requests can fail: thinking off, a temperature, a prefill, and a forced tool choice (features 22, 28, 30, 44). The Gateway needs a table of what each model accepts and a stated policy for each case: translate, downgrade and report, or refuse at the run's first step. Confidence: high - each case has a quoted error.

**2. Behavior that depends on which provider served the request.** One model id covers several providers, and the documentation says "depends on the provider" in many places:

- Web search and web fetch run by the vendor exist on the Anthropic and "Claude Platform on AWS" endpoints of a Claude model, while its Bedrock, Azure, and Vertex endpoints use a search engine that OpenRouter runs itself [P12, AN40].
- `supports_tool_choice`, `structured_outputs`, context length, and output limit differ per endpoint of one model. `z-ai/glm-5.3` has 46 endpoints with context from 262,144 to 1,048,576 tokens [M49, M50, T9].
- Whether the prefix check is enforced "depends on the provider that serves the request" [AN10].
- Aborting a stream stops billing on 24 named providers and continues on 24 others [O51, V16].
- YouTube links work on Google AI Studio and fail on Vertex [GG76].
- Fast mode "is only served by the Anthropic first-party provider" [AN85].

*For Runvil:* the Gateway has one endpoint per model entry today, so most of this cannot happen yet (feature 95). The problem returns on the day a model entry gets a second endpoint, and the entry will then need capability data per endpoint. Confidence: high - the differences are in OpenRouter's public endpoint data.

**3. Data that is valid only where it was made.** Here OpenRouter's caller ends up needing to know the provider. OpenRouter's SDK team confirmed on 2026-06-17 that a Gemini tool continuation that moves between Google AI Studio and Vertex returns a 400, and advised callers to name the provider with `provider: { order, allow_fallbacks: false }`. That comment is still the latest on the issue [F45, VT8]. Google's Vertex page scopes a signature to "the same conversation history, model, and Google Cloud project" [VV18]. Anthropic's newest thinking blocks work only in the organization that produced them [VV7]. OpenAI drops reasoning made under another organization [OA27]. OpenRouter writes the provider into each encrypted value and keeps the conversation on that provider, and it answers 400 when the caller's provider preferences exclude it [R63].

*For Runvil:* the opaque field's label has to name the endpoint, and the Gateway has to keep a conversation on that endpoint (feature 5). Confidence: medium - the vendors define "account" differently, and nobody has measured how often a mismatch occurs.

**4. Features that work only on a route in a vendor's format.** Ten features work on only one or two of OpenRouter's three routes, among them vendor compaction, document citations, and asynchronous tools (section 4). A caller that wants one picks the route by vendor.

*For Runvil:* an opaque field that held only reasoning would force the same choice. The ten are block types, item types, and request settings, so the field has to hold every kind of block. Confidence: high - OpenRouter's own guides say which route each needs.

**5. Vendor settings that the caller writes.** OpenRouter's callers write cache markers chosen from the model id prefix [F19], `x-anthropic-beta` for strict tools and interleaved thinking [T21, R25], `provider.options` [O3], vendor-specific cache lifetimes [C22], and the `x_search` object [P15].

*For Runvil:* the Gateway can handle each of these itself: cache markers and lifetimes, schema cleaning, beta headers, and vendor parameters (features 31, 33, 48, 96, 97). Confidence: high - each answer is in use in at least one open-source project.

**6. Repairs that moved into client code.** OpenRouter's own SDK for the Vercel AI SDK creates an id when a provider sends an empty or repeated one, rewrites `stop` to tool calls for Gemini, and has fifteen special cases for reasoning [T18, O58, F67]. When a reply ends for a reason other than the model finishing, it drops the tool calls whose arguments are incomplete JSON [T28]. Across six TypeScript projects the first round of evidence gathering counted 187 special cases for OpenRouter, and across nine Rust, Python, and Go projects 90, with reasoning sent back as the largest group in both [F67, and the tally in the G evidence file]. Harnesses also keep rules for single model ids, for what the catalog leaves out [F35, F39], and match the phrase "Provider returned error" to decide on a retry [F64, F41].

*For Runvil:* the Gateway has to put these into one common form, so that Runvil's client never grows such a list: unique ids (feature 46), the finish reason for tool calls (feature 85), argument checks (feature 49), typed errors (feature 86). Confidence: high - counted from source at pinned commits.

## 6. OpenRouter's chat completions route leaves out three kinds of vendor feature: data kept on the vendor's servers, blocks a chat message cannot hold, and settings in a vendor's own words

**Kind 1: data kept on the vendor's servers.** OpenRouter rejects it. `store: true` and a non-null `previous_response_id` get a 400, the spec's only Responses path is the create call, `conversation` is outside the schema, and Google's Interactions API is not offered [OA1, OA2, OA24, GG79]. Background runs, polling, and webhooks are left out with it [OA57]. The six open-source gateways compared in the first round forward or drop a `previous_response_id` for a vendor that does not hold the conversation [J-table 2]. One project, LiteLLM, emulates it by rebuilding history from its own request log database, and emulates background runs with a Redis cache [CT43, CT44]. The three projects found to support Google's Interactions API each give it an entry point separate from ordinary chat requests, in the Vercel AI SDK through a second model object [GX64 to GX66].

*What it shows:* a format in which each request holds the whole conversation cannot refer to data kept on a server. LiteLLM, the one project that offers stored conversations anyway, had to add a database of its own. Every other project sends the vendor's data back with each request, as in features 1 to 20. That suits Runvil, because the Engine keeps the whole conversation in its own list. Confidence: high - OpenRouter's spec and seven projects agree.

**Kind 2: blocks and settings that a chat message cannot hold.** These work only on OpenRouter's Messages or Responses route:

- Anthropic: `server_tool_use` and result blocks, `pause_turn`, compaction blocks, context editing, document citations, `tool_addition` and `tool_removal` blocks, the `clear_at` field, `between_tools`, the `drop_block` setting, `stop_details` [AN45, AN52, AN55, AN64, R62].
- OpenAI: `phase` and several assistant messages per reply, asynchronous tool calls, custom tools, shell, apply patch, compaction items [OA41, OA43, OA54].
- Google: `executableCode` and `codeExecutionResult` parts, and the signed `toolCall` and `toolResponse` parts that appear when built-in tools are combined with functions. The documentation of all three routes leaves these out [GG13, GG70].

*What it shows:* a chat completions API can hold these only if it has an ordered list of the vendor's own blocks that the caller stores unread. OpenRouter's `reasoning_details` cannot hold them, which is why OpenRouter has two more routes. The 2026-11-17 report judged that vendor-run tools and vendor compaction "produce blocks and stop reasons with no home in chat completions" and might force a route in the vendor's format. The evidence here supports a narrower reading: the opaque field can hold the blocks, and a gateway can handle the stop reasons that ask for a continuation (features 13 and 63). Confidence: medium - LiteLLM and OpenRouter each store these blocks in a non-standard place and both lose their order, while Pydantic AI, LiteLLM, the Vercel AI SDK, and Google's own SDKs continue such a turn themselves.

**Kind 3: settings in a vendor's own words.** Gemini's `safetySettings`, media resolution, URL context, and Maps grounding, Anthropic's `container.skills` and cache diagnostics, Kimi's formula tools, Alibaba's `enable_search`, Zhipu's `web_search` tool type, DeepSeek's `prefix`, and Kimi's `partial` are left out of the documented fields of every OpenRouter route [GG65, GG69, GG75, AN88, OV42, OV52, OV56]. The only way to send them is `provider.options`, whose allowlist is unpublished.

*What it shows:* an API could pass these through easily, and the price is a caller that knows the vendor. OpenRouter mostly declined. Runvil should decline at the prompt and give the operator a setting per model. Confidence: high - none of the projects read has a vendor-neutral form for these.

**Token counting is also left out.** None of the 123 paths in OpenRouter's spec counts tokens [AN86]. A caller estimates locally or reads `usage` afterward, which is what the Engine's precheck already does.

## 7. What Runvil's Gateway, client, and Engine do today, and six defects that cost remote models their usage numbers, error details, and compaction

The list below says what each part of Runvil does today, area by area, with the files. The Gateway is the repository `runvil-gateway`. The client is `crates/harness-gateway-client` and the Engine is `crates/runvil-internal`, both in `runvil-harness`. The code was read on 2026-11-22 at `runvil-harness` `22a0992f`, `runvil-gateway` `52fd92d`, and `runvil-cloud-providers` `50961b2`. It was only read: the build and the tests were not run, so each statement still needs a test to confirm it. The 2026-11-17 report on Anthropic's Messages API described the same code at an older commit: every claim in its Table 3 and Appendix D still holds, and four line ranges moved.

**Request fields**

- **Gateway:** Names `model`, `messages`, `stream`. Every other top-level field passes through. Rewrites `model` only (`protocol/src/wire.rs`, `protocol/src/upstream/openai.rs`).
- **Client:** The body has exactly nine keys: `model`, `messages`, `stream`, `stream_options`, `tools`, `tool_choice: "auto"`, `temperature`, `max_tokens`, `chat_template_kwargs.enable_thinking` (`wire/request.rs`).
- **Engine:** `ModelInvocation` has `temperature`, `max_tokens`, `thinking` (`model-client/src/model/options.rs`).
- **What Lua sees or sets:** `models.use(role, {temperature, max_tokens})`. The thinking switch comes from front matter. A handle exposes `provider`, read-only.

**Message record**

- **Gateway:** Messages are untyped JSON. Unknown message fields pass through in both directions.
- **Client:** Builds the reply from the fields it accumulates, so any reply field outside that set ends at the client (`wire/parse.rs`, `wire/stream.rs`).
- **Engine:** Wire `Message` has `role`, `content`, `tool_call_id`, `tool_calls`. The list record has `role`, `content`, `tool_calls`, `meta`, all Lua-visible. A hidden per-reply field is stated as intended in `decisions/engine.md` and is not in the record type yet.
- **What Lua sees or sets:** Five fields per record: `role`, `content`, `tool_calls`, `tool_call_id`, `meta`.

**Reasoning**

- **Gateway:** Passes `reasoning_content`, `reasoning`, `reasoning_details` through as the upstream wrote them.
- **Client:** Reads a string from the first of `reasoning_content`, `reasoning`, `thinking`. Streams it to the Host. Never sends it back.
- **Engine:** One `Thinking` event per round, then discarded. The token estimate subtracts reasoning tokens (`lua/src/compactors.rs`).
- **What Lua sees or sets:** `handle.thinking` only.

**Text beside tool calls, block order**

- **Gateway:** Passes through.
- **Client:** Returns the tool calls and never reads the text beside them. Order between text, reasoning, and calls is gone.
- **Engine:** Appends an assistant record with empty content and the calls.
- **What Lua sees or sets:** The record with empty content.

**Tools**

- **Gateway:** `tools` and `tool_choice` pass through, except for the one tool format that the Gateway emulates.
- **Client:** Function tools only, `tool_choice` fixed at `auto`. Fails a batch cut off by `length` or `content_filter`.
- **Engine:** Requires each call id to be unique across the whole list, and each batch to be answered directly (`lua/src/projection.rs`).
- **What Lua sees or sets:** `tool_calls` records.

**Streaming**

- **Gateway:** Parses and re-writes each chunk. Unknown top-level, choice, and delta fields survive on a chunk that parses. A chunk with empty `choices` is skipped. Writes its own `[DONE]`. Stream limit: a 10 second connect timeout only (`protocol/src/upstream.rs`, `http_util.rs`, `app/src/relay.rs`).
- **Client:** Requires `[DONE]`. Skips comments. 120 seconds per receive, 16 MiB per reply.
- **Engine:** Gets one finished `Completion` per round.
- **What Lua sees or sets:** The round's outcome.

**Usage**

- **Gateway:** A `usage` object survives only on a chunk that has a choice.
- **Client:** Reads `prompt_tokens`, `completion_tokens`, `total_tokens`, `cached_tokens`, `reasoning_tokens`.
- **Engine:** `Usage` has those five (`types/src/metrics.rs`).
- **What Lua sees or sets:** Through the Host's events.

**Finish reasons**

- **Gateway:** Pass through.
- **Client:** Compares `length` and `content_filter` in one place.
- **Engine:** Compares `stop` and `length`.
- **What Lua sees or sets:** Inside a caught `empty_model_reply` error.

**Errors**

- **Gateway:** Replaces the upstream body with `upstream returned {status}`. 4xx keeps its status, every 5xx becomes 502. `Retry-After` is set for `model_loading` only (`protocol/src/error.rs`).
- **Client:** Classifies by status and by phrases in the body (`wire/classify.rs`). One HTTP attempt per round.
- **Engine:** `ContextOverflow` calls the compactor. Other codes fail the call.
- **What Lua sees or sets:** `kind`, `code`, fixed `message`.

**Model metadata**

- **Gateway:** `GET /v1/models` publishes `thinking`, `max_output`, `effort_levels`, `default_effort`, `adaptive_thinking`, and more from `gateway.toml`. The code that handles a request does not read them (`app/src/models.rs`).
- **Client:** Reads `id`, `description`, `context`, `thinking`, `provider`.
- **Engine:** Checks `min_context` and `thinking` at the run's first step.
- **What Lua sees or sets:** `min_context` and `thinking` per role.

**Where a translator would attach**

- **Gateway:** `Upstream::send` and `stream` get the request and the upstream model name. `Protocol` has one variant, matched in `Routing::from_config`. The one emulated tool format, `gemma3_tool_code`, is three pure functions around the upstream call (`app/src/dialect.rs`).
- **Client:** `build_request_body` and `read_completion_stream` are public.
- **Engine:** `InferenceBroker::chat`.

Source: evidence items L1 to L121.

**Reading the code turned up six defects.** Each was read from code and has yet to be reproduced, so confidence in each is medium until someone runs it.

- **Usage is lost for a streamed reply from an OpenAI-style cloud provider.** The client asks for the usage chunk with `stream_options.include_usage`. OpenAI sends it as a chunk with empty `choices`, and the Gateway's chunk parser skips a chunk with empty `choices` (`protocol/src/wire.rs`, `protocol/src/upstream.rs`) [L57, L58, L67].
- **A failed stream can look complete.** A bare `{"error": ...}` object from the upstream fails the Gateway's chunk check and is skipped, and the Gateway then writes a clean `[DONE]` [L85].
- **A vendor's "prompt too long" error does not start compaction.** The client recognizes an overflow by phrases in the error body, and the Gateway replaces that body with `upstream returned {status}`. So the Engine is not told that the request was too long and does not call the compactor in response. The Engine's own estimate, made before a request is sent, can still call it. Spent quota and a content refusal are misclassified the same way [L74 to L81]. The 2026-11-17 report found this for Anthropic, and it holds for every remote upstream.
- **A slow stream has two time limits that disagree.** The Gateway swallows the upstream's keep-alive comments and waits on a silent stream indefinitely. The client gives up after 120 seconds of silence [L54, L60, L61].
- **Tool-call ids must be unique across the whole conversation.** The Engine enforces it when it builds a request from its list, while the metrics type notes that providers reuse ids such as `call_1`, and the Gateway's own emulated tool format restarts its ids at 0 on every reply [L46].
- **Every OpenRouter model in the model sheet is marked as lacking tools and thinking.** The program that builds the sheet reads ten values from each catalog entry and leaves out `supported_parameters` and the `reasoning` object, so a model added from the Gateway's Cloud page gets `thinking = "never"` [L94, M58, M59].

**The client already has one piece of vendor-specific code.** It sends the thinking switch as `chat_template_kwargs.enable_thinking`, a field that only llama.cpp and vLLM understand. The fix is for the client to send a standard field and for the Gateway to convert it for each upstream. The Gateway's decision page (`runvil-design`, `decisions/gateway.md`) already lists that work as "The thinking switch for remote providers".

## 8. What Runvil should change: store more in the opaque field, build the OpenAI Responses translator early, fix four Gateway defects first, and give the Gateway a table of what each model accepts

The current plan: the client keeps speaking chat completions, the Gateway gains a translator per vendor protocol behind the same route, and the opaque field holds each reply's ordered list of vendor blocks, labeled with protocol and model. The evidence supports that plan and changes it in seven places.

**1. The opaque field should record its endpoint, be used on the `openai` protocol, hold every kind of block, and arrive once.** Confidence: high that the opaque field is the right mechanism, because every vendor's data fits it and every less detailed format read loses something. Medium on the details, because nobody has built it.

- **The label should name the endpoint.** Protocol and model are too little. Anthropic's newest thinking is bound to an organization, Google scopes a signature to a Cloud project, OpenAI drops reasoning from another organization, and OpenRouter writes the endpoint into every encrypted value for the same reason (feature 5).
- **The `openai` protocol should use it too.** DeepSeek, Kimi, Qwen, GLM, and MiniMax need their reasoning text back, and so do local chat templates for gpt-oss, Kimi, Qwen3, and GLM. The Gateway can put that text in the opaque field and write the vendor's field when it sends the conversation back. That fixes a failure that DeepSeek's documented rule causes today on a tool loop (features 14 and 17).
- **It should hold every kind of block.** Blocks from vendor-run tools, compaction blocks, `phase` labels, container ids, and the Gateway's own data all fit the same ordered list (features 8, 20, 62, 77).
- **The Gateway should send it whole, once, on the final chunk.** The client then copies one value, and the per-vendor joining stays in the Gateway (feature 18).

**2. OpenAI needs a Responses translator early.** From GPT-5.4, OpenAI's chat completions API does not support tool calling with reasoning on, and on five GPT-6 models it does not support tool calling at all (feature 7). Runvil's Gateway forwards chat completions requests unchanged, so a tool loop on those models needs the translator. Runvil's client lets the model use its default effort, which is `medium` from GPT-5.5, so today's requests already hit the limit (inferred from OpenAI's per-model defaults). Sending `reasoning_effort: "none"` is a stopgap that turns reasoning off, and it restores tool calling on GPT-5.4 to 5.6, GPT-6 Sol, and GPT-6 Luna only. The same translator serves xAI and Meta, whose encrypted reasoning also exists only on a Responses route (feature 6). Confidence: high - OpenAI's own migration pages, fetched twice.

**3. Gemini needs a third translator of the same kind, with two extra rules.** Its part list goes in the opaque field. The extras are the placeholder signature for history from another model (feature 12) and the continuation of long output (feature 13). Confidence: medium - five open-source projects use the placeholder, and the continuation is new.

**4. Four Gateway defects should be fixed before any vendor protocol.** They cost usage numbers, error details, and a compaction trigger on every remote model today (section 7): the skipped usage chunk, the replaced error body, the skipped error inside a stream, and the missing heartbeat and idle deadline. Confidence: medium - read from code and not yet reproduced.

**5. The Gateway needs a table of what each model accepts, and it has to consult that table on every request.** Dropping sampling parameters, refusing "thinking off", downgrading a forced tool choice, rewriting ids, cleaning schemas, and filling `max_tokens` all depend on the model. The model sheet already publishes part of the table (`thinking`, `effort_levels`, `max_output`), and the Gateway leaves it unread when it handles a request. OpenRouter's public catalog has the rest: `reasoning.mandatory`, `supported_efforts`, `supported_parameters`, and the per-endpoint `supports_tool_choice`. Confidence: high - each of these fixes differs from model to model.

**6. The universal API should gain a short list of vendor-neutral names, all standard or shared.** Confidence: high for the first two, medium for the rest.

- A reasoning-effort setting, as OpenAI's standard `reasoning_effort` (feature 21).
- A cache-write count in usage, as `prompt_tokens_details.cache_write_tokens` (feature 38).
- The thinking switch as a standard field, in place of `chat_template_kwargs.enable_thinking` (feature 22 and section 7).
- Optional, each when its feature is wanted: `refusal`, a response schema, `annotations`, a conversation id, a deferral flag on tools, file and audio parts.

**7. Runvil needs one policy for a vendor-neutral request that a model cannot honor.** OpenRouter mixes three answers: ignore silently, pass the provider's 400 through, and reject with its own 400. Its callers ended up with per-model rules in client code. Runvil has two better tools: the requirement check at a run's first step, and the `gateway_warning` field. Suggested policy: refuse at the first step when a declared requirement cannot be met (`thinking: false` on a model that always thinks), and drop the setting with a warning when the model ignores it (a temperature on Claude). Confidence: medium - the right answer depends on how much a prompt author should be told.

### What should stay as planned: one route, Anthropic first, and prompts that name only a model role

- **One route for model requests.** Every feature that OpenRouter serves only on its Messages or Responses route fits the opaque field. A second inbound route is what made OpenRouter's callers choose a route by vendor. Confidence: medium - nobody has built the opaque field, and the evidence that it can hold these features is other projects' code.
- **Anthropic first.** Its rules are still the strictest documented, and the design that satisfies them is the one the other two vendors need. Confidence: medium - the OpenAI finding is a reason to start the second translator early, and the order stays.
- **Prompts that name only a model role.** None of the 105 features requires a prompt to know its vendor (section 2). Confidence: high - each feature that came close can be handled in the Gateway's configuration.

### Decisions for the project owner, starting with what the opaque field records and where it is used

1. **Whether the opaque field's label names the endpoint** as well as the protocol and model. Recommended: yes. Confidence: medium - three vendors bind data to an account.
2. **Whether the `openai` protocol uses the opaque field** for plain-text reasoning that a vendor wants back. Recommended: yes. Confidence: high - DeepSeek's rule is documented and fails today.
3. **Whether the opaque field gets a name of Runvil's own or OpenRouter's `reasoning_details`.** Recommended: Runvil's own, with `reasoning_details` accepted on input. Confidence: medium - OpenRouter's format loses block order (section 4).
4. **Whether the second translator is for OpenAI's Responses API or for Gemini.** Recommended: OpenAI's, because tool calling on GPT-6 depends on it. Confidence: medium - it depends on which models the students who do the lesson plan want to run.
5. **What the standalone client does when it talks to OpenRouter directly.** To send reasoning back there, the client needs code that joins `reasoning_details` stream fragments, which is a vendor branch in shared code. The alternative is one general rule: the client keeps every non-standard field of the final assistant message and of each tool call, unread and labeled with endpoint and model, and sends it back to the same endpoint and model. That rule covers DeepSeek's `reasoning_content`, Google's `extra_content`, and Runvil's own opaque field. Recommended: the general rule only. Confidence: low - it is an untested proposal.
6. **What happens when a prompt asks for something the model cannot do** (change 7). Recommended: refuse at the first step for a declared requirement, and drop with a warning otherwise. Confidence: medium - it depends on how much a prompt author should be told.
7. **Which vendor-neutral names to add now** (change 6). Recommended: the reasoning-effort setting and the cache-write count. Confidence: high - both are OpenAI's own field names.
8. **Whether Lua keeps `handle.provider`.** A model handle shows the provider id to a prompt today, so a prompt can branch on the vendor. Confidence: medium - it is the one place where a prompt can depend on its vendor today.
9. **Whether the Gateway also accepts requests in Anthropic's Messages format.** The decision page rules it out, and two reports recommend it (section 10). Recommended: keep the one route. Confidence: medium - nobody has built the opaque field.
10. **Who continues a reply that the vendor stops partway.** Long output on Gemini 4 and a paused turn on Claude can each be continued by the Gateway inside one client request (features 13 and 63), or handed to the Engine as a new step in its loop. Recommended: the Gateway, and only when the feature is adopted. Confidence: medium - four libraries continue the first themselves and one the second, and the cost of a long silent request is untested.

## 9. OpenRouter's published material says how it handles 85 of the 105 features, and other projects or vendor documentation cover the rest

Table 6 counts the features by where the account of OpenRouter's handling comes from.

*Table 6. Where the account of each feature comes from.*

| Source | Features | What it covers |
|---|---|---|
| Stated by OpenRouter | 85 | Field names and formats on all three routes, the rule for sending reasoning back, reasoning settings, cache markers and their conversion, usage fields, the list of vendor-run tools, error types, routing, the catalog's flags, the documented rejections |
| Inferred about OpenRouter | 7 | The loss of block order (from the spec's format), tool-call ids passed through (from recorded traffic), DeepSeek's rule left to the caller (from issue reports and recorded traffic), the finish reason for tool calls (from the official SDK's fixes), a system message in the middle of a conversation, Gemini safety settings (a field missing from the spec), token counting (a route missing from the spec). Several features in the first group also have one inferred part, named on the feature's Source line, such as the provider written into encrypted values (feature 5) |
| From other open-source projects | 6 | Features where OpenRouter's material is silent or does not apply: the Gemini placeholder signature, the model switch, schema output on a model that lacks a schema mode, and three local-server features |
| Stated by the vendor | 7 | Features where OpenRouter is silent and the vendor's rule decides: tool calling on OpenAI's newest models, Qwen's encrypted reasoning, Mistral's chunk lists, cache diagnostics, skills and programmatic tool calling, the paused turn, the OpenAI-style routes that differ from OpenAI |
| Earlier Runvil research | 0 | Fifteen features name an earlier Runvil document as a second source. Twelve statements taken from such documents were not read again on 2026-11-22 (section 10) |
| Total | 105 | |

Source: counted from the Source lines of the 105 feature sections.

**For 19 features, an open-source project's code is the evidence that the proposed handling works.** In most of them OpenRouter's own handling is documented too: OpenRouter's material says what the field is, and the project shows whether a translation behind it works. The projects named most often:

- **rig** stores the whole block list beside each vendor-neutral block, labels it with protocol, provider, and model, and sends it back only on an exact match (features 1, 3, 5, 6, 19).
- **LiteLLM** has the chat-to-Responses bridge, Gemini cache objects, the usage sum rule, the placeholder signature, padding for DeepSeek, and the long-output loop (features 6, 8, 12, 13, 14, 37, 38).
- **The Vercel AI SDK** keeps data per part and `phase` per text part, runs the long-output loop, and sends plain JSON Schema to Gemini (features 8, 10, 13, 48).
- **Bifrost** rewrites ids, emulates schema output with a forced tool call, strips account-bound reasoning and retries, and runs a tool loop in the gateway (features 5, 46, 56, 61).
- **Pydantic AI** records the endpoint for server ids, continues a paused turn inside the library, and has vendor-neutral tool classes (features 5, 63, 73).
- **Gemini CLI** runs search as an ordinary tool that makes its own request (feature 61).
- **pi and opencode** send plain-text reasoning back and rewrite ids (features 14 and 46). On a model switch pi drops the other model's reasoning and opencode passes it on as text (feature 19).
- **llama.cpp and vLLM** show the local-server behavior in their own source (features 17, 103, 104, 105).

**What every open-source translator gets wrong or leaves out.** Each case is evidence that translation alone is too little, or that a different mechanism is needed:

- **Block order inside an Anthropic reply.** Every gateway with the three-field chat format loses it (feature 3). This is the strongest single argument for an opaque field that is an ordered block list.
- **A label on stored vendor data.** Of six open-source gateways only one labels the data with its producer, and the model libraries disagree on whether the label includes the model (features 5 and 19).
- **A native finish reason beside the normalized one.** None of the six gateways returns it (feature 84).
- **Conversations stored by another vendor.** One project emulates them, with a database (feature 74).
- **Safety settings and Maps grounding.** None of the projects read offers a vendor-neutral form of either (features 72 and 73).

**The open questions about OpenRouter leave what Runvil has to build unchanged.** For each question, an open-source project shows a translation that works, or the feature is one Runvil does not use today. Section 11 lists the questions.

## 10. Earlier Runvil documents: what was taken from each, and what is now wrong or out of date in them

Runvil's repositories already hold research on vendor APIs. Evidence file W groups it into 43 sources, which cover about 60 documents, and has 250 evidence items from them [W]. Table 7 lists the documents used here, with what was taken from each and whether it was read again in a live source on 2026-11-22.

*Table 7. Earlier Runvil documents used.*

| Document | Date | What was taken from it | Read again on 2026-11-22 |
|---|---|---|---|
| `runvil-design`, `research/2026-10/2026-10-28-survey-proprietary-inference-apis.md` | 2026-10-28 | The first account of OpenRouter as "the reference implementation" of the aggregator role, nineteen claims about it (sixteen of them from this survey), and its twelve points of comparison | Yes. All nineteen were checked, and five corrections to the survey are listed below |
| Finished report "How agent harnesses handle Anthropic's Messages API, and what Runvil should build" | 2026-11-17 | The opaque block-list design, the Anthropic contract, the per-model thinking table, the map of Runvil's code, and the survey of 21 harnesses and 5 gateways | Yes for the code map and the points used here. Five corrections below |
| `runvil-design`, `research/2026-09/2026-09-20-model-tool-differences.md` | 2026-09-20 | The replay and tool-call id rules per vendor (Table 2, Appendix B), and the llama-server duties (Appendix C) | Yes for DeepSeek, Mistral, Kimi, Gemini, OpenAI. Three corrections below |
| `runvil-design`, `research/2026-09/2026-09-30-compaction-exhaustion-signals.md` | 2026-09-30 | OpenRouter's overflow and in-stream error signals, and the finding that Runvil's stream reader classes an in-stream error as a transport failure | Yes. One correction below |
| `runvil-design`, `research/2026-10/2026-10-14-harness-mcp-dynamic-tools.md` | 2026-10-14 | The native tool search rules at Anthropic and OpenAI | Yes. One note below |
| `runvil-design`, `decisions/gateway.md` and `runvil-gateway`, `crates/gateway/README.md` | 2026-11-10 and 2026-11-15 | The Gateway's rule on standards and extensions, "zero dialects", the planned work list, and the statement that "an Anthropic-shaped request is outside the API" | Code facts re-read at today's head |
| `runvil-harness`, `decisions/models-and-conversations.md` and `decisions/engine.md` | 2026-11-20 and 2026-11-22 | The five record fields, the model roles, what a prompt may set, and the project owner's words on data in the list that "Lua can't see" | Code facts re-read at today's head |
| `runvil-classroom`, `lesson-plans/2026-11-17.md` and `lesson-plans/2026-11-11.md` | 2026-11-17 and 2026-11-11 | Items 2 and 3, their open questions, and the eight decisions for the standalone client | Not applicable |
| Finished report "Could LiteLLM substitute for Runvil Gateway?" | 2026-11-16 | That LiteLLM's chat translation is still Python, and its release cadence | No. LiteLLM's source was read again at today's commit |
| `runvil-design`, `research/2026-11-09-port-unsloth-gateway.md` | 2026-11-09 | That llama-server already serves `/v1/messages` and `/v1/responses`, and a recommendation to pass those routes through | Yes for llama.cpp's routes |
| `runvil-design`, `research/2026-10/2026-10-21-compaction-in-agent-harnesses.md` | 2026-10-21 | Vendor-side compaction (`compact_20260112`, the Responses compact endpoint) | Yes |
| `runvil` and `runvil-gateway`, `vibe/2026-09/2026-09-14-2-provider-model-sheets.md`, and `runvil-cloud-providers`, `crates/gateway-cloud-providers/src/providers/` | 2026-09-14, code at `50961b2` | What the model sheet reads from each vendor's model list, the seven providers that lack an OpenAI-compatible base URL, and the rule "normalize the knob, never the settings" | The OpenRouter catalog was fetched again. One correction below |
| `runvil-gateway`, `vibe/2026-08/2026-08-23-2-gateway-phases-4-5.md` | 2026-08-23 | "Anthropic inbound shim (rejected)" | Not applicable |
| `runvil-classroom`, `transcripts/2026-11-06-class-1.md` | 2026-11-06 | The instructor's words on a generic protocol with "just enough extensions" (a machine transcript) | Not applicable |

Source: evidence file W, items W1 to W250.

**Corrections to the 2026-10-28 survey of vendor APIs.** Confidence in each is high: each was read in a live source on 2026-11-22, and most were fetched twice.

- **"Unsupported parameters are silently ignored".** That is the default only. OpenRouter documents 22 rejections of its own and passes several provider 400s through (section 5). The survey named `previous_response_id` as the one exception [S41 to S48, V22].
- **"Reasoning is replayed verbatim or dropped".** The documentation supports "replayed verbatim". On the chat completions route the documented results of a mismatch are requests kept on one provider and 400s, and the general case is undocumented. The survey said each entry has "a vendor `format` tag such as `anthropic-claude-v1`". The array has four entry types and twelve `format` values, and OpenRouter's reasoning guide still says three types [R32 to R34, V2].
- **Cache marker translation "on the OpenAI route".** The conversion follows the model that serves the request, whichever route was called. Lifetimes are dropped in both directions [C19 to C22, V9].
- **Google's OpenAI-compatible route "loses thought signatures" and cache handles.** It keeps both: signatures in `tool_calls[].extra_content.google.thought_signature`, and `cached_content` through `extra_body`. `missing_thought_signature` is a finish reason and an Interactions API error code, and the HTTP 400 for a missing signature has `status: "INVALID_ARGUMENT"` [GG6, GG25, VV13, VV16].
- **Smaller corrections.** Hugging Face Inference Providers has routed language models through OpenRouter since 2026-11-16, so it has stopped being an independent example. vLLM renamed `reasoning_content` to `reasoning`. The survey's remark that the `in_flight_budget_exhausted` string is gone has itself gone out of date. From GPT-6.2, a system prompt sent as top-level `instructions` loses the cache. Mistral's `/v1` route is now described as its native format [VT4, VV23, O37, OA62, VV20].

**Corrections to the 2026-11-17 report on Anthropic's Messages API.**

- **The opaque field's label.** That report labels the block list with protocol and model. Anthropic's newest thinking is bound to the organization that produced it, Google scopes a signature to one Cloud project, and OpenAI drops reasoning from another organization. The label needs the endpoint too (feature 5) [VV7, VV18, OA27].
- **A route in Anthropic's format is less likely to be needed than it judged.** It put a roughly even chance on Runvil wanting `/v1/messages` later, because blocks from vendor-run tools, `pause_turn`, and compaction blocks had "no home in chat completions". The opaque field can hold them (features 62 and 77), and the Gateway can continue a stopped turn itself (features 13 and 63), as Pydantic AI, LiteLLM, the Vercel AI SDK, and Google's own SDKs already do.
- **Which project rewrites ids, and which clients work against Bifrost.** agentgateway copies tool-call ids unchanged today, and the project that rewrites them is Bifrost. A client that reads `reasoning_details` still fails against Bifrost, because Bifrost's entries lack OpenRouter's `format` value [J34, J27].
- **`prompt_tokens_details.cache_write_tokens`.** That report's Table A3 calls it "an extension name that agentgateway and OpenRouter both use". It is in OpenAI's own Chat Completions reference today [Q30, VV4, W198].
- **An eighth stop reason.** Anthropic's beta reference lists `compaction` [VV8].

Confidence: high for the label, the attributions, the field name, and the stop reason, which were read live or in code at pinned commits. Medium for the judgment about a second route, because nobody has built the opaque field.

**Corrections to the 2026-09-20 report on tool differences.**

- Kimi K3 "drops the ``func:index`` format requirement" for tool-call ids. The older K2 templates were not checked again [T57, VV21].
- For an encrypted value from another organization, OpenAI's pages today describe the item being removed and reported with the reason `organization_mismatch`. The `invalid_encrypted_content` error that the report quoted was searched for on 30 OpenAI pages and not found [VV5].
- Gemini 3 and later always return an `id` on a function call, and Google tells the caller to send it back with the tool's result, so "ids optional" is out of date [GG85].

Confidence: high - each was read in the vendor's live documentation or code on 2026-11-22.

**Corrections to other documents and to one code comment.**

- The 2026-09-30 report's table of exhaustion signals groups OpenRouter with OpenAI's 429 quota codes. OpenRouter's own signals are a 402 with `error_type: "payment_required"` and a 403 for budget rejections [O76].
- The 2026-10-14 report says most proxies do not forward `tool_reference` blocks. OpenRouter now documents its own handling: it passes the block through where the endpoint defers tools itself, and elsewhere the block "reaches the provider as text" [P46].
- A comment in `runvil-cloud-providers`, `openrouter.rs` says the default list is the full catalog. Today it is the text-output list, 476 of 739 models [M].
- xAI's function calling guide gives a tool limit of 350 and its two API reference pages give 1,000, which explains why two Runvil documents disagree [VV24].

**Some statements from these documents were not read again on 2026-11-22.** There are twelve, counting those read only in part. Among them are MiniMax's HTTP 200 for a failed request [OV62], Anthropic's tool-call id pattern [T50], the wording "prompt is too long" [O73], and vLLM's random signature [W19].

**Runvil's own documents disagree about a second route, and the project owner has to settle it.** The decision page rules a request in Anthropic's format out of the Gateway's API, and the August plan lists "Anthropic inbound shim (rejected)". The 2026-11-09 port study and a student report dated 2026-11-17 (`runvil-classroom`, `students/robles/2026-11-17-anthropic-messages-runvil-report.pdf`) both recommend serving `/v1/messages` from the Gateway. The 2026-11-17 report on Anthropic's Messages API recommends translating behind the existing route. The evidence here supports the decision page: every feature that OpenRouter serves only on its Messages route fits the opaque field, and a second route is what forced OpenRouter's callers to choose a route by vendor (section 4). Confidence: medium - nobody has built the opaque field.

## 11. Limits: every statement comes from reading, every source was read on one day, and some of OpenRouter's behavior is still unknown

**Every statement comes from reading.** The sources are documentation, specifications, a public catalog, source code, and traffic that others recorded. No request went to a model, to OpenRouter's model routes, or to a vendor, and no Runvil code was built or tested. The one exception is a local run of OpenRouter's own schema files against hand-made entries, to see what its parser does with an unknown `format` [VF6].

**Only public pages were read.** Two OpenRouter routes need a key and were left unread: `GET /api/v1/tools`, which has the per-tool names and prices, and `GET /api/v1/models/user`.

**OpenRouter's translator is closed source.** For 20 of the 105 features, the account of OpenRouter's handling is inferred or comes from another project or the vendor (section 9). Another project's code shows how that project solves a problem. It is evidence for what Runvil needs, and what OpenRouter does internally stays unknown.

**Every source was read on one day, 2026-11-22.** OpenRouter's changelog shows changes on most days of November, its `format` list grew five times since July, and three of the vendor rules used here are less than a month old: Anthropic's API version of 2026-10-21, OpenAI's GPT-6.2 rules of 2026-11-17, and Hugging Face's move to OpenRouter on 2026-11-16. Some of this will be out of date within weeks.

**Documentation and deployed behavior can differ.** OpenRouter's own sources disagree with each other in at least seven places: the sticky routing lifetime (60 minutes against 10), forced tool choice on Haiku 5.5 (the guide says it works, the catalog says it fails), the `openrouter:bash` tool (in the spec's chat list, refused per the guide), "three possible types" of reasoning entry in the guide against four in the spec, an unsupported structured output that "will fail" on one page and "is ignored" on another, 503 against 529 for an overloaded provider, and the default effort of Opus 5.5 [V10, V17, V19, V2, T34, O, AN25]. Where they disagree, the feature's section says so and takes neither side.

**Recorded traffic is dated.** The recordings that show OpenRouter's real output are from 2025-05-20 to 2026-11-18, from seven open-source projects. They show what happened on those days.

**The classification is a judgment.** Each feature's option was assigned from the evidence, under the rules in section 1. Every feature listed under the opaque field depends on a field that Runvil has designed and has yet to build. A reader who separates Gateway-only work from compatibility code differently can re-sort the features from each one's "Work in Runvil" line.

**The second round covered a sample.** It re-read 128 claims, the ones the conclusions depend on most and the most surprising ones, out of about 1,950 evidence items. Table 8 gives the result. Its corrections are in the feature sections and in section 10.

*Table 8. The second round: 128 verdicts.*

| What was re-checked | Confirmed | Confirmed with a correction | Not confirmed | Conflict between sources | Overstated |
|---|---|---|---|---|---|
| OpenRouter's documentation, spec, and catalog (27 claims) | 21 | 4 | 0 | 2 | 0 |
| Vendor documentation (27 verdicts) | 21 | 2 | 3 | 1 | 0 |
| Open-source code at pinned commits (48 items) | 39 | 7 | 0 | 0 | 2 |
| Third-party pages and issue reports (26 verdicts) | 16 | 9 | 1 | 0 | 0 |
| Total | 97 | 22 | 4 | 3 | 2 |

Source: the four verification evidence files, keys V, VV, VF, VT. Three of the four "not confirmed" verdicts are claims from earlier Runvil documents that the first round had already questioned. The fourth is the claim that MiniMax produces `reasoning_details` itself, which MiniMax's own pages do not support today.

**Some sources were first read as excerpts.** Thirty-one pages in the evidence file on other gateways and about ten issue pages were read as search-result excerpts in the first round. The second round opened in full the ones the conclusions depend on. The rest are marked in their evidence files.

**Runvil's own code was read and has yet to be run.** The six defects in section 7 are readings of code.

### What is still unknown about OpenRouter, and why what Runvil has to build stays the same

A third round took the 36 questions that OpenRouter's closed source left open and looked for answers in recorded traffic, open-source translators and client libraries, vendor documentation, and other gateways' documentation. Most now have an answer, or another project's code that shows a translation that works. The few that remain concern things only OpenRouter has, such as its records of vendor-run tools. Table 9 lists what is still unknown about OpenRouter's own behavior, with what was tried.

*Table 9. What is still unknown about OpenRouter.*

| Still unknown about OpenRouter | What was tried | What other projects do |
|---|---|---|
| What it does with reasoning entries in a foreign format after a model or provider switch | Its docs, spec, and SDKs, and about 1,600 recorded exchanges from seven projects. None of the recordings exercises it, because the recording clients strip reasoning before a switch | pi and Goose drop another model's reasoning, and rig and opencode pass it on as text; Bifrost strips and retries once |
| How it rebuilds the order of thinking, text, and tool calls for Anthropic | Docs, and the two published examples of a translated request. All 67 recorded Claude thinking replies have one block | Every open gateway with the same shape loses the order; rig, Hermes Agent, and Agno keep the block list |
| Its rule for tying a Gemini signature entry back to a part | Five recorded shapes. A function-call signature has an `id` equal to the tool-call id, and a text-part signature has only its place in the array | Six transports in open projects (feature 10) |
| What happens when a value that names one endpoint reaches a different endpoint, and whether it now keeps a Gemini tool continuation on one upstream | 34 recorded replays all came back from the endpoint the value names, and none shows a mismatch. The latest advice from OpenRouter staff, on 2026-06-17, is to name the provider and forbid any other | LiteLLM keeps the request on one provider or strips the data; Bifrost strips and retries; three projects report that Google AI Studio and Vertex reject each other's signatures |
| Whether it drops unsigned Claude reasoning or lets Anthropic reject it | Two accounts that disagree: a code comment in OpenRouter's own client library and a user's comment on an issue report. None of the recordings sends it | Not needed: Runvil's opaque field holds signed thinking only |
| Its server-side rule for removing repeated reasoning entries | Only the SDK's client-side copy of a key rule | Not needed |
| Whether it fills a vendor's required `reasoning_content` for a caller that dropped it, and what it writes upstream | Issue reports from April and May 2026 show DeepSeek's 400 reaching callers. None of the recordings sends reasoning back to these models | LiteLLM pads the field; pi and opencode return it |
| How it sends a chat system message to OpenAI, and which `store` and `include` values it sends | A search of docs, issues, and the web found such examples for Claude only. Recordings do show which models are served through the Responses API | LiteLLM's bridge and Codex |
| Whether a caller's `prompt_cache_key` reaches OpenAI, whether `x-grok-conv-id` is set, what a cache marker does on Gemini 4, and which sticky lifetime is real | Docs (two pages give two lifetimes) and recordings (clients strip markers before sending) | pi sends the key itself; LiteLLM's cache code treats Gemini 4 like any other model |
| Whether the prompt-template tool emulation still runs | One code comment in the docs. A recorded request to a model without tool endpoints got a 404 | Runvil's own `gemma3_tool_code` tool format |
| Whether it rewrites tool-call ids on the way in | Recordings show one vendor's ids accepted by another, and third-party captures show Mistral's rejection relayed. An employee wrote on 2026-03-31 that a fix was coming, and no page reports that it shipped | Bifrost, opencode, pi, and Roo Code rewrite |
| Which server tools leave a `reasoning.server_tool_call` entry on the chat route, and whether it must be returned | The spec (the entry appears there and in none of the guides), every tool guide, both SDKs, and the recordings (none has such an entry). The per-tool table is behind `GET /api/v1/tools`, which needs a key | This field exists in OpenRouter alone |
| Whether Anthropic's code execution, memory, and MCP connector work through its `/messages` route, and what `background` and `item_reference` do on `/responses` | The spec and guides. It needs a live request | LiteLLM emulates background runs and stored history with a database |
| Whether it resends a turn that Anthropic paused (`pause_turn`) or hands the pause to the caller, and which finish reason the chat route shows | Guides, spec, changelog, and both SDKs. The value is in the stop-reason list of the Messages route and in none of the guides | Pydantic AI and Anthropic's own SDKs resend automatically; LiteLLM hands it to the caller |
| Whether `[DONE]` always follows an error chunk, the keep-alive interval, and any timeout | Two recordings from 2025-11, one with `[DONE]`. Fixtures store only the bytes, not their arrival times | Not needed: Runvil's client should treat any chunk with `error` as a failure |
| The allowlists behind `x-anthropic-beta` and `provider.options`, and the safety thresholds it sends to Gemini by default | Docs and spec. Both lists are unpublished | Anthropic tells gateways to forward beta values as written; new-api and one-api send an operator default for safety |

**Outside OpenRouter, three questions are also unresolved.**

- **Whether Gemini thought signatures work across Google AI Studio and Vertex.** Google's pages are silent on it. Three projects report rejection in both directions, two of them from live requests, and one project's code says the formats are shared [GX30 to GX35].
- **Pages of one publisher that conflict:** xAI's tool limit (350 against 1,000), Meta's `tool_choice` (Meta says `auto` only, and OpenRouter's catalog says all four on two of three Meta models), and forced tool choice on Haiku 5.5 [VV24, VV22, V17].
- **The six readings of code in section 7.** Each needs one test run to confirm.

## 12. Sources and evidence files

**How to read an evidence item.** A code such as `R36` is item 36 in the evidence file whose items start with `R`. Each item has the exact quote, the URL or the repository, commit, file, and line, and the date it was read. Table 10 lists the 26 evidence files, which are in the `evidence/` folder beside this report. All were produced on 2026-11-22.

*Table 10. The evidence files, by the letters their items start with.*

| Keys | Evidence file | Items | Sources |
|---|---|---|---|
| S | ["OpenRouter API surfaces and request superset"](evidence/2026-11-22-conduct-research-openrouter-api-surfaces-request-superset.md) | 69 | 66 |
| R | ["OpenRouter reasoning deep dive - request controls, `reasoning_details`, and replay rules"](evidence/2026-11-22-conduct-research-openrouter-reasoning.md) | 85 | 43 |
| C | ["OpenRouter prompt caching and usage accounting"](evidence/2026-11-22-conduct-research-openrouter-prompt-caching-usage-accounting.md) | 85 | 51 |
| T | ["OpenRouter client tools (function calling) and structured output"](evidence/2026-11-22-conduct-research-openrouter-tools-structured-output.md) | 67 | 52 |
| P | ["OpenRouter server tools, plugins, and content types"](evidence/2026-11-22-conduct-research-openrouter-server-tools-plugins-content-types.md) | 86 | 51 |
| O | ["OpenRouter routing and operations evidence"](evidence/2026-11-22-conduct-research-openrouter-routing-operations-errors.md) | 82 | 58 |
| M | ["OpenRouter model discovery"](evidence/2026-11-22-conduct-research-openrouter-model-discovery-catalog.md) | 64 | 42 |
| F | ["OpenRouter field evidence part 1 - how TypeScript harnesses and SDKs special-case OpenRouter in source"](evidence/2026-11-22-conduct-research-openrouter-field-evidence-typescript-harnesses.md) | 68 | 50 |
| G | ["OpenRouter field evidence part 2 - how Rust, Python and Go harnesses special-case OpenRouter in source"](evidence/2026-11-22-conduct-research-openrouter-field-evidence-rust-python-harnesses.md) | 129 | 42 |
| AN | ["OpenRouter and Anthropic vendor-native features"](evidence/2026-11-22-conduct-research-openrouter-vendor-anthropic.md) | 99 | 91 |
| OA | ["OpenRouter deep dive - OpenAI vendor-native features and the Responses API"](evidence/2026-11-22-conduct-research-openrouter-vendor-openai-responses.md) | 85 | 68 |
| GG | ["OpenRouter deep dive - Google Gemini vendor-native features"](evidence/2026-11-22-conduct-research-openrouter-vendor-google-gemini.md) | 108 | 57 |
| OV | ["OpenRouter deep dive: the other vendors' hard rules and the local servers behind the Runvil Gateway"](evidence/2026-11-22-conduct-research-openrouter-vendor-others-and-local-servers.md) | 81 | 94 |
| K | ["LiteLLM as an open-source fill for OpenRouter's closed translator"](evidence/2026-11-22-conduct-research-openrouter-translators-litellm.md) | 72 | 56 |
| H | ["Unified model layers in open-source SDKs"](evidence/2026-11-22-conduct-research-openrouter-unified-sdk-model-layers.md) | 73 | 23 |
| J | ["Open-source gateways and translators: vendor state through the chat-completions shape"](evidence/2026-11-22-conduct-research-openrouter-translators-open-source-gateways.md) | 100 | 60 |
| Q | ["OpenRouter deep dive: other aggregators and cloud gateways"](evidence/2026-11-22-conduct-research-openrouter-other-aggregators-cloud-gateways.md) | 63 | 94 |
| L | ["Runvil today: a code map of the Gateway, the gateway client, and the Engine"](evidence/2026-11-22-conduct-research-openrouter-runvil-today-code-map.md) | 121 | 53 |
| W | ["Workspace research sweep"](evidence/2026-11-22-conduct-research-openrouter-workspace-research-sweep.md) | 250 | 43 |
| WT | ["OpenRouter gap-fill from recorded wire traffic and debug echoes"](evidence/2026-11-22-conduct-research-openrouter-gapfill-recorded-wire-traffic.md) | 45 | 17 |
| GX | ["Gap fill for Google Gemini: translations that exist in open-source projects for the Gemini features OpenRouter documents no chat-completions field for"](evidence/2026-11-22-conduct-research-openrouter-gapfill-gemini-translation.md) | 68 | 38 |
| CT | ["Gap-fill - turns the vendor pauses or cuts, vendor-run server tool blocks, and asynchronous or stateful extras"](evidence/2026-11-22-conduct-research-openrouter-gapfill-continuation-server-tools.md) | 49 | 50 |
| V | ["OpenRouter verification pass: independent live re-fetch of 27 first-wave claims about OpenRouter's docs, spec, and public catalog"](evidence/2026-11-22-conduct-research-openrouter-verification-openrouter-claims.md) | 27 claims | 84 |
| VV | ["Independent verification of first-wave claims about vendor native rules"](evidence/2026-11-22-conduct-research-openrouter-verification-vendor-claims.md) | 27 verdicts | 60 |
| VF | ["Verification of first-wave claims about open-source code at pinned commits"](evidence/2026-11-22-conduct-research-openrouter-verification-field-evidence-code.md) | 48 items | 14 repositories |
| VT | ["Verification of third-party pages"](evidence/2026-11-22-conduct-research-openrouter-verification-third-party-pages.md) | 26 verdicts | 62 |

**How much each kind of source can show.** OpenRouter's documentation, spec, catalog, and SDK source are the authority on what OpenRouter documents, and what its closed translator does stays uncertain. Vendor documentation is the authority on each vendor's own rule. Source code at a pinned commit is direct evidence of what a project does. Recorded traffic shows what OpenRouter sent on the recording date. Issue reports are third-party accounts and are labeled so where they are used.

### OpenRouter's own material (all public, read without credentials)

- **Spec:** `https://openrouter.ai/openapi.json` (OpenAPI 3.1, 123 paths, 1,071 schemas; byte-identical on two fetches about an hour apart).
- **Catalog:** `https://openrouter.ai/api/v1/models` (476 entries) and `https://openrouter.ai/api/v1/models/{author}/{slug}/endpoints`.
- **Documentation**, each page at `https://openrouter.ai/docs/` plus the path, all 559 pages mirrored on 2026-11-22. The pages used most:
  - `guides/best-practices/reasoning-tokens`, `api_reference/responses/reasoning`
  - `guides/best-practices/prompt-caching`, `cookbook/administration/usage-accounting`
  - `guides/features/tool-calling`, `guides/features/structured-outputs`, `guides/features/plugins/response-healing`
  - `guides/features/server-tools` and its twelve tool pages, `guides/features/plugins`, `guides/overview/multimodal/*`
  - `guides/routing/provider-selection`, `guides/routing/model-fallbacks`, `guides/providers/traffic-routing`, `guides/providers/integration`
  - `api_reference/overview`, `api_reference/parameters`, `api_reference/streaming`, `api_reference/errors-and-debugging`, `api_reference/limits`, `api_reference/versioning`, `api_reference/models`
  - `api/api-reference/chat/create-a-chat-completion`, `api/api-reference/responses/create-a-response`, `api/api-reference/anthropic-messages/create-a-message`
  - the model migration guides under `cookbook/evaluate-and-optimize/model-migrations/`: `claude-4-6`, `claude-4-7`, `sonnet-5`, `opus-5`, `fable-5-1`, `claude-5-5`, `gpt-5-4`, `gpt-5-6`, `gpt-6`, `gemini-4`
  - `changelog`
- **Official SDK source:** `OpenRouterTeam/ai-sdk-provider` at `7a235a4`, `OpenRouterTeam/typescript-sdk` at `7ad8458`, `OpenRouterTeam/python-sdk` at `b48773e`, `OpenRouterTeam/go-sdk` at `654d9d4`, `OpenRouterTeam/typescript-agent` at `8a2d96e`.

### Vendor documentation (read live on 2026-11-22)

- **Anthropic:** `https://platform.claude.com/docs/en/` plus `build-with-claude/preserved-thinking` (and its prefix checks page), `build-with-claude/thinking`, `build-with-claude/effort`, `build-with-claude/prompt-caching`, `build-with-claude/handling-stop-reasons`, `build-with-claude/structured-outputs`, `build-with-claude/files`, `api/errors`, `api/versioning`, `agents-and-tools/tool-use/server-tools`, `agents-and-tools/tool-use/web-search-tool`, `agents-and-tools/tool-use/tool-search-tool`, `agents-and-tools/mcp-connector`. Gateway guidance: `https://code.claude.com/docs/en/llm-gateway-protocol`.
- **OpenAI:** `https://developers.openai.com/api/docs/guides/` plus `reasoning`, `migrate-to-responses`, `conversation-state`, `function-calling`, `prompt-caching`, `structured-outputs`, `tools-web-search`, `tools-tool-search`, `latest-model/gpt-6-astra`. References: `https://developers.openai.com/api/reference/resources/responses/methods/create` and `.../resources/chat/subresources/completions/methods/create`.
- **Google:** `https://ai.google.dev/gemini-api/docs/` plus `thought-signatures`, `thinking`, `caching`, `function-calling`, `structured-output`, `openai`, `google-search`; the same pages at `https://aistudio.google.com/docs/`, with `long-decode` and `safety-settings`; `https://ai.google.dev/api/generate-content`; Vertex AI: `https://docs.cloud.google.com/vertex-ai/generative-ai/docs/thought-signatures`.
- **xAI:** `https://docs.x.ai/developers/` plus `model-capabilities/text/reasoning`, `model-capabilities/text/comparison`, `rest-api-reference/inference/responses`, `rest-api-reference/inference/chat-completions`, `advanced-api-usage/prompt-caching`.
- **DeepSeek:** `https://api-docs.deepseek.com/guides/thinking_mode`, `/guides/tool_calls`, `/guides/kv_cache`, `/api/create-chat-completion`.
- **Mistral:** `https://docs.mistral.ai/capabilities/function_calling/`, `https://docs.mistral.ai/openapi.yaml`, and the `mistral-common` validator.
- **Moonshot:** `https://platform.kimi.ai/docs/guide/use-thinking-models`, `/docs/api/chat`, `/docs/guide/use-official-tools`, and `https://huggingface.co/moonshotai/Kimi-K3/blob/main/encoding_k3.py`.
- **Alibaba:** `https://www.alibabacloud.com/help/en/model-studio/deep-thinking`, `/context-cache`, `/qwen-api-via-openai-chat-completions`.
- **Zhipu:** `https://docs.z.ai/guides/capabilities/thinking-mode`, `https://docs.z.ai/api-reference/llm/chat-completion`.
- **MiniMax:** `https://platform.minimax.io/docs/api-reference/text-openai-api`. **Meta:** `https://dev.meta.ai/docs/tool-calling`, `https://dev.meta.ai/docs/reasoning`.

### Open-source code at pinned commits

- **Gateways and translators:** `BerriAI/litellm` at `90dce4b`, `agentgateway/agentgateway` at `888badb`, `maximhq/bifrost` at `cae639c`, `Portkey-AI/gateway` at `669825c`, `theagentrouter/agent-router` (formerly `envoyproxy/ai-gateway`) at `9cf8505`, `musistudio/claude-code-router` at `6b6ca0c` and its engine package `@the-next-ai/ai-gateway` 1.0.21, `QuantumNous/new-api` at `185e1e5`, `songquanpeng/one-api` at `8df4a26`, `Helicone/ai-gateway` at `9649b27`.
- **Model libraries:** `vercel/ai` at `d55f3cf`, `pydantic/pydantic-ai` at `463a6d5`, `langchain-ai/langchain` at `8ff3cd3`, `langchain-ai/langchain-google` at `227abc8`, `tensorzero/tensorzero` at `62eb8f6`, `0xPlaygrounds/rig` at `a8673f7`, `jeremychone/rust-genai` at `2a8a5b3`.
- **Harnesses:** `earendil-works/pi` at `9c46526`, `anomalyco/opencode` at `71cd8c0`, `cline/cline` at `8075d2e`, `RooCodeInc/Roo-Code` at `b867ec9`, `Kilo-Org/kilocode` at `8045e72`, `zed-industries/zed` at `d31d107`, `block/goose` at `36b2911`, `Aider-AI/aider` at `5dc9490`, `NousResearch/hermes-agent` at `0f602c5f`, `openai/codex` at `227fa4e`, `google-gemini/gemini-cli` at `eecccfc`.
- **Local servers:** `ggml-org/llama.cpp` at `8d60873`, `vllm-project/vllm` at `e2052f5`, `ollama/ollama` at `318ee28`.
- **Recorded OpenRouter traffic:** rig's 63 recorded exchanges (cassettes) under `crates/rig-cassette/fixtures/cassettes/openrouter/` and its reply bank, recorded 2026-09-22 to 2026-10-03.

### Other gateways' documentation

- Vercel AI Gateway: `https://vercel.com/docs/ai-gateway/sdks-and-apis/openai-chat-completions/reasoning` (updated 2026-09-08). LLM Gateway: `https://docs.llmgateway.io/features/reasoning`. Requesty: `https://docs.requesty.ai/features/reasoning`. Hugging Face: `https://huggingface.co/blog/openrouter` (2026-11-16). Open Responses: `https://www.openresponses.org/specification` and `/governance`. Amazon Bedrock Converse: `https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_Converse.html`. Cloudflare AI Gateway: `https://developers.cloudflare.com/ai-gateway/usage/chat-completion/`.

### Issue reports and pull requests (third-party accounts)

- `OpenRouterTeam/ai-sdk-provider` issue 491 (Gemini signatures across AI Studio and Vertex; staff comment 2026-06-17), issue 574 (forced eager tool streaming), pull request 608 (late reasoning details lost; open).
- `earendil-works/pi` issue 9534 (signatures lost on the chat route). `pydantic/pydantic-ai` issues 5869 and 4370. `BerriAI/litellm` issues 23047, 32188, 37849, 52001. `maximhq/bifrost` issue 7729. `anomalyco/opencode` issue 41469.

### Runvil, by repository

- `runvil-gateway` at `52fd92d`: `crates/gateway/README.md`, `crates/gateway/protocol/src/` (`wire.rs`, `upstream.rs`, `upstream/openai.rs`, `error.rs`, `http_util.rs`), `crates/gateway/app/src/` (`relay.rs`, `routing.rs`, `models.rs`, `dialect.rs`), `crates/gateway/config/src/config/vocab.rs`.
- `runvil-harness` at `22a0992f`: `decisions/models-and-conversations.md`, `decisions/engine.md`, `crates/harness-gateway-client/src/` (`wire/request.rs`, `wire/parse.rs`, `wire/stream.rs`, `wire/read.rs`, `wire/classify.rs`, `transport.rs`), `crates/runvil-internal/model-client/src/`, `crates/runvil-internal/lua/src/` (`messages.rs`, `projection.rs`, `compactors.rs`, `protocol/record.rs`), `crates/runvil-internal/types/src/metrics.rs`.
- `runvil-cloud-providers` at `50961b2`: `crates/gateway-cloud-providers/src/providers/`.
- `runvil-design`: `decisions/gateway.md` and the research reports named in section 10. `runvil-classroom`: `lesson-plans/2026-11-17.md`, `lesson-plans/2026-11-11.md`.

*2026-11-22 19:17 - Claude Fable 5.2*
