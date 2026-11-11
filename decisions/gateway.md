# The Gateway: design decisions and their reasons

- Scope: the Gateway server in the `runvil-gateway` repository: its HTTP API and auth, its configuration file and config UI, local models, cloud providers and the published model sheet, profiles, routing and tool-call dialects, web search, boot, restart, logging, and progress
- Code: `runvil-gateway` at commit `9369a7d`, `runvil-cloud-providers` at `1817cc4`, `runvil-docs` at `cfd69a4`, and `runvil` at `fa9936f64`
- Paths: code paths start at the `crates/gateway/` folder of the `runvil-gateway` repository

## Summary

The Gateway is a server process, `runvil-gateway`, that accepts OpenAI-shaped requests and forwards each one to a model. One file, `gateway.toml`, lists remote backends as `[[endpoint]]` and `[[model]]` entries and local models as `[[local_model]]` entries. For a remote model the Gateway sends an HTTP request to a provider with the provider's key. A local model is a `llama-server` child process that the Gateway starts at boot from the selected profile. An admin API and a browser UI at `/config/` edit the file through staged copies, called shadow files, and an Apply step. Sections 1 to 9 describe these parts.

A check of this page against the code found fifteen gaps and left ten questions about reasons for the owner. Section 11 lists the gaps, and section 12 lists the questions. The largest gap is the restart feature: the route, the flag, and the exit code exist, and the supervisor that acts on them is still to be built.

Section 10 lists the planned work. Its largest item is the supervisor process that restarts the Gateway after a change that needs a restart.

## From a chat request to a model's reply

A client sends `POST /v1/chat/completions` with a JSON body that names a model. The `AuthedCaller` extractor runs `check_auth`, which accepts the bearer key from `[server] api_key`, a handoff cookie, or a keyless caller on the same machine. `WireJson<ChatRequest>` then parses the body, and `ChatRequest::validate` checks its shape.

`resolve_routed_model` looks the name up in the live `Routing` table. A hit returns a `Model`, and `require_kind` checks that its kind is `chat`. The model's `Endpoint` has a `DominionQueue`, a queue with a concurrency limit, and `admit` returns a `Permit` when a slot is free. On a miss the Gateway answers 404 with the code `model_not_found`, or 503 while the Gateway is still starting that model.

The `Endpoint` also has an `Upstream`. For a remote model it is an `OpenAiUpstream`, built from an `[[endpoint]]` entry: it replaces the model name with the `[[model]]` entry's `upstream` value and posts the body to `{base_url}/chat/completions` with the endpoint's `api_key`. For a local model it is a `LocalUpstream`, which owns one `llama-server` child process and sends the same body to the child's port.

For a plain request, the upstream writes the caller's model name into the `ChatResponse`, and the handler checks the reply's shape and returns it. For `stream: true`, `relay_sse` writes each `ChatChunk` as a `data:` event and ends with `data: [DONE]`. The `Permit` drops when the reply or the stream ends, and the queue admits the next waiting request.

## 1. The process and its API

### The Gateway is the one process with model provider credentials

The Gateway reads each provider's API key from its config, maps a requested model name to a backend, and forwards the request. The Harness and Workshop need the Gateway's URL and, from another machine, its bearer key. The Gateway also has the key for Brave web search. How Workshop finds, starts, and calls the Gateway is on [Workshop: how the app is built](workshop-app.md).

### The API follows OpenAI's shapes, and each side defines its own structs

The chat, embeddings, and model list routes use OpenAI's request and response shapes. The Gateway's README states the aim: "a program written against that standard works against the gateway". `ChatRequest` names three fields, `model`, `messages`, and `stream`, and the Gateway forwards every other field and every message unchanged. Clients send only these shapes: an Anthropic-shaped request is outside the API, because the Gateway's purpose is exactly one way of doing things. Where the Gateway needs something that a standard lacks, the README states the rule: the route follows the standard shape, the addition is the smallest opt-in extension, and a client still works when the extension is absent.

The Gateway and its clients share JSON, and each side defines its own Rust types: JSON is the contract. A shared client crate would be code, and shared code is the coupling that the JSON contract avoids. The Harness's client is on [Model calls and conversations](models-and-conversations.md) and [The Harness and the Host API](harness-and-host.md).

### A caller uses the Gateway's model name, and the backend receives its own

Each `[[model]]` entry pairs `name`, which callers send, with `upstream`, which the backend knows. `Routing::model` looks the name up by exact match. `OpenAiUpstream` puts the `upstream` value into the forwarded request and writes the caller's name into the reply. The Gateway answers an unknown name with 404 and the code `model_not_found`.

An entry also lists `endpoints`, and the Gateway uses the first one. Section 10 lists a choice among several endpoints as planned work.

### Each inference route accepts one model kind

Every model has a `kind`. `POST /v1/chat/completions` accepts `chat`, `POST /v1/embeddings` accepts `embedding`, `POST /v1/rerank` accepts `classifier`, and `POST /v1/audio/speech` accepts `speech`. Each handler calls `require_kind`, which answers a model of another kind with 400 and the code `kind_mismatch` before any backend call.

One `kind` key covers `[[model]]` and `[[local_model]]` entries, so that dominion binding (section 6), the model list, and validation each have one code path. The rerank route copies the request shape that llama-server, vLLM, and Jina use, and the Gateway's README states why: for this route "OpenAI publishes no shape". A standard classifier route from OpenAI would reverse that choice. The speech routes are on [Speech](speech.md).

### The Gateway answers every failure in the OpenAI error envelope

`GatewayError::envelope` builds a body with `message`, `type`, and `code` under `error`, and `classify` maps each error to an HTTP status.

The Gateway reports a failed connection as `upstream_connect` and a later failure as `upstream_transport`, because the two differ for a caller. After a failed connection the request is unsent and unbilled, and a retry is safe. After a read failure or a timeout, the provider may have received the request. The extractors `WireJson`, `WireQuery`, and `WirePath` answer a malformed request in the same envelope. A handler lists them after its auth extractor, so that an unauthenticated caller gets 401 before its malformed input gets 400.

### The Gateway parses each chunk of a streamed reply, and the stream owns its queue slot

For `stream: true`, the Gateway parses each chunk from the backend, writes the caller's model name into it, and sends it to the client as a server-sent event. It skips a chunk that fails the shape check, the same minimal check that a whole response gets. When the backend refuses the request, the failure happens before the SSE response starts, and the client gets a normal JSON error.

The stream owns the request's `Permit` until it ends. The streaming HTTP client sets a 10 second connect timeout and omits the 120 second limit that plain requests have, because that limit covers the whole body read and would end any long stream. When the client disconnects, the response body drops, and the drop closes the backend connection: "Drop is the entire mechanism".

### `GET /v1/models` lists the models that can answer now

The route returns an OpenAI-shaped list with one entry for each model in the live routing table, plus the loaded speech-to-text models ([Speech](speech.md)). Each routed entry adds `kind`, `description`, `context`, `thinking`, and the capability keys from the config, such as `max_output`, `images`, `effort_levels`, and `provider`. The list exists so that the Harness fetches authoritative model metadata from the Gateway.

The capability keys describe a model to clients, and the Gateway forwards each request as the client wrote it. Seven of the nine keys, `default_temperature` included, are facts that a client needs to construct a valid request, so a client applies them. The Harness ignores all seven. A local model that is still starting is absent from the list, and `GET /admin/status` reports it under `loading_models`. The list is limited to models that can answer so that the route remains OpenAI-compatible.

### `POST /v1/tools/web_search` returns search results and leaves page fetches to the caller

The route takes a query and a few optional filters, calls the Brave search API with the key from `[tools.web_search]`, and returns a list of results. The route answers 404 when the config lacks the table and 400 when the body has an unknown key. The Gateway returns at most `max_per_host` results from one host, 2 by default, and strips tracking parameters from each URL.

The call is in the Gateway because the Gateway has the Brave key. The route returns links and snippets, and a caller fetches the pages itself, because comparable search tools also separate search from fetch. The per-host limit exists because raw search rows repeat the same host.

## 2. Auth and the loopback checks

### Auth accepts a bearer key, a handoff cookie, or loopback trust

`check_auth` passes a request when one of three rules matches, tried in this order: the bearer token equals `[server] api_key`, the cookie from the browser handoff verifies, or loopback trust applies. A handler takes an `AuthedCaller` argument, and axum runs the check before the handler's body and before any body parsing.

The handoff is `GET /auth?key=`. The Gateway's tray icon opens that URL, the Gateway sets an HttpOnly cookie, and it redirects to `/config/`, so the browser's history records a URL without the key. The Gateway answers a wrong bearer with 401, even from the same machine: loopback trust applies to the absence of credentials, and a caller that presents wrong ones intended to authenticate. The check fails closed: on any uncertainty it refuses the request.

### The Gateway admits a keyless caller on the same machine by default

With `trust_loopback = true` under `[server]`, the default, the Gateway admits a request that meets three conditions: the peer address is loopback, the request omits the `Authorization` header, and its `Sec-Fetch-Site` header is absent, `same-origin`, or `none`. The setting covers every route, the admin routes included.

The owner's rule: "RUNVIL_GATEWAY_API_KEY should not be required when the incoming connection is on the loopback adaptor". The cost is that on a shared machine another operating-system account can use the Gateway. The header test exists because any web page can send a request to a local port. A browser adds `Sec-Fetch-Site` to every request from a page, so the Gateway answers 401 to a page on another origin.

### A Gateway bound to loopback refuses a request with a foreign `Host` header

When the listener's address is loopback, `build_router` wraps every route in `require_loopback_host` from the `shared-loopback` crate. The layer refuses a request whose `Host` header names anything other than the bound socket. The refusal stops DNS rebinding, an attack that points a web page's hostname at the local machine so that the page can call a local server. Jupyter uses the same defense.

### The walled admin tier refuses callers from other machines

The admin routes have two tiers, and the tier is the directory. `admin/open/` has the routes that any admitted caller may use: profiles, status, progress, and queue cancellation. `admin/walled/` has the routes that read secrets in plaintext, write files, or start processes: config, env, apply, revert, restart, and `POST /shutdown`. `build_router` applies `require_loopback` once to the walled tier, so a caller from another machine gets 403 before auth runs, even with the key.

The check is in a crate that every build compiles, because the UI crate is optional and the wall must exist in every build. A structural check fails when a file outside the tier names a `crate::admin::walled::` path.

### Each route area mounts its own routes and declares them in a registry

A route area is a module such as `relay`, `speech`, `models`, or `health`. Each one defines `routes()`, which mounts its handlers, and `ROUTES`, a list of `RouteInfo` values with a path, the methods, and a `Tier` of `Open` or `Walled`. `build_router` merges the areas, so a new endpoint changes one area module. An area behind a Cargo feature is absent from a build that turns the feature off. The four features, `local`, `web-search`, `config-ui`, and `stt`, are on by default.

The tests take their list of routes from the registry, because a route list written by hand in a test can miss a route. The Gateway's private crates are the folders under `crates/gateway/`, and [Crates, build, and CI](crates-build-and-ci.md) covers the structural checks on them.

## 3. The configuration file

### One `gateway.toml` contains the whole configuration

The file has `[server]`, `[local]`, `[tools]`, and `[stt]` tables, and lists of `[[dominion]]`, `[[endpoint]]`, `[[model]]`, `[[local_model]]`, `[[stt_model]]`, and `[[profile]]` entries. Its first key is `config-version = 0`, and the loader refuses any other value. The version is 0 because the format is pre-release. The loader also refuses every unknown key, `include` and `[workshop]` among them, and section 11 lists what the guide says about those two.

### `${VAR}` references resolve from the environment and a sibling `.env` file

A string value in the config may contain `${VAR}`, and `$$` writes a literal dollar sign. At startup the Gateway loads `gateway.env`, the file beside `gateway.toml`, into its environment and then expands each reference. A variable that the shell set overrides the file's value, and an unset variable fails the load with an error that names it.

The file exists so that the secrets are independent of what the process inherits from a shell. The Gateway stores each credential in a `Secret`, a type that prints `redacted` and has one read accessor, `expose()`.

### The Gateway writes a default config on its first run

The Gateway takes its config from `--config` or `RUNVIL_GATEWAY_CONFIG`, then looks beside the executable, then in the working directory, then in `~/.runvil`. When all of these are empty, `generate_default` writes a file into `~/.runvil` with `bind = "127.0.0.1:0"`, a random `api_key`, `trust_loopback = true`, the recommended pair of speech-to-text models, and a profile named `default`. It also writes the state file that selects that profile.

The Gateway generates the file itself, so its first start works the same whether a person or Workshop launches it. The Gateway generates the key once and reads the same key on every later start.

### The Gateway owns its config files and rewrites them whole

For a write through the admin API, the Gateway parses the config into typed structs, changes them, renders the whole file with `toml::to_string_pretty`, and replaces the file through a temp file and a rename. The first save drops the comments and the hand formatting of the file. The owner's rule: "No hand-edited anything. App owns it all."

The same rule covers every operator task: the config UI should do all of them, so that a hand edit of the TOML files is unnecessary.

## 4. Editing the configuration

### A save writes a shadow file, and the real file changes at Apply

`PUT /admin/config` validates the whole config the way a load does and then writes it to `gateway.toml.next`, a staged copy that the code calls a shadow. `PUT /admin/env` writes `gateway.env.next`. The Gateway boots from the real files, so a shadow waits across restarts until the operator presses Apply or Revert All. `POST /admin/config-revert` deletes the shadows, and the real files are unchanged.

The shadow file is the whole record of pending changes, and an operator can inspect it on disk. A save leaves the running config unchanged, and only Apply changes it. The owner's reason: "this prevents an unexpected thud".

### Apply runs as a queued command and promotes files at its commit

`POST /admin/config-apply` reads the shadows under the apply lock, puts an `ApplyConfig` command on the command queue, and waits for its outcome. The apply lock is a mutex that saves, Apply, and Revert share. The command builds the new routing table, writes each captured file over its real file, and swaps the live state in one write. A failed or cancelled command leaves every shadow in place for a retry. A revert cancels an Apply in progress, so a revert that an operator issues during an Apply wins.

The command promotes the files at its commit, after it builds the new routing table, because a promotion before a failure would leave the real file ahead of the running state.

### Apply swaps remote routing at once and reports a restart for boot-read sections

The `ApplyConfig` command rebuilds the routing table from `[[model]]`, `[[endpoint]]`, and `[[dominion]]`, merges the running local models into it, and swaps the table, the config, and the web search service together. Requests in progress finish on the upstream they started with. The reply sets `restart_required` when the env file changed or when a section in `RESTART_SECTIONS` changed: `server`, `profile`, `local_model`, `stt_model`, or `stt`. The UI then shows a banner that asks for a restart.

The owner's rule: "Changes to remote models does not require a gateway restart." The swap is immediate because each request in progress owns an `Arc` of its upstream, and the swap proceeds while those requests finish.

### A pending config resolves `${VAR}` from the pending env file first

When the Gateway validates a pending config, `PendingEnv` reads `gateway.env.next` when it exists and `gateway.env` otherwise, and resolves each `${VAR}` from that file before the process environment. An operator can therefore save a provider key on the Secrets page and add a model that uses it in the same session. Validation runs against the environment that the next boot will have, as well as the one that this process booted with.

The app crate reads the env files. The `gateway-config` crate parses and validates, and it takes the lookup as a function, under its rule that the calling binary loads env files.

### `GET /admin/config` replaces every secret with `***`

The route returns the running config as JSON, and `ser_redacted` writes the marker `***` for each secret field. On `PUT /admin/config`, the Gateway reads a field that still contains the marker as the value on disk, so a secret changes when the UI sends a new literal or a `${VAR}` reference. This round trip lets the UI save the whole config and send the marker in place of each stored secret.

`GET /admin/env` is the exception. It returns the env file's values in plaintext to the Secrets page. The plaintext is deliberate: the route is in the walled tier, behind the loopback wall and the auth check.

### The config UI is plain TypeScript on the Gateway's own port

The Gateway serves the UI at `/config/` on its one listener, to loopback callers. The crate `gateway-config-ui` contains the UI, which is TypeScript against the browser's DOM, bundled with esbuild. It has seven pages: Settings, Discover, Local, Remote, Cloud, Profiles, and Secrets. With `?mode=panel` it runs in an iframe inside Workshop and sends each API call to Workshop with `postMessage`, so the key remains outside the iframe ([Workshop: how the app is built](workshop-app.md)).

The UI is framework-free and uses the same tools as Workshop's UI. Each element copies an existing product. The owner's rule: "do not innovate UI elements. always look to unsloth first to find an element to use verbatim."

## 5. Profiles and the local model set

### A profile is a checklist of local and speech-to-text models

A `[[profile]]` entry has a `name` and a `models` list, and each name in the list is a `[[local_model]]` or `[[stt_model]]` entry. The selected profile decides which of those the Gateway loads at boot. Every remote `[[model]]` routes under any profile, and the loader refuses a profile that lists one.

The owner's rule: "profile should only apply to local models ... all remote models will all be available regardless of the profile." If a profile also selected remote models, an operator who configures a cloud model would find it missing from `GET /v1/models` until the selected profile lists it. A remote name fails the load so that a profile's `models` list has one meaning.

### A state file beside the config stores the selected profile

`gateway.state.toml` has one key, `active_profile`. At startup the Gateway takes the profile from `--profile`, then from `RUNVIL_PROFILE`, then from the state file. When all three are empty, the Gateway boots, routes every remote model, and logs `no profile selected; serving remote models only`.

When the state file names an undefined profile, the Gateway logs a warning and boots with only the remote models. An operator who deletes the last profile in the config UI and applies the change therefore still has a Gateway that starts. An undefined name in the flag or the variable stops the boot, because the Gateway would otherwise ignore a wrong explicit instruction from the operator in silence.

### The local model set is fixed for the life of the process

The boot load, one command that runs after the listener binds, starts the selected profile's local models once. After that, the set changes at the next start of the process. The Gateway saves an edit to a profile, a `[[local_model]]`, a `[[stt_model]]`, or `[stt]` and reports `restart_required`.

The owner's rule: "Changes to the local model set require a gateway restart." A switch inside a running process would need code of its own to swap child processes that a restart can start anyway. A finer change, the load or eviction of one model when a request asks for it, would need swap serialization, a drain of requests in progress, an eviction policy, and a check of which models fit in memory together: too much mechanism for too little gain.

### `POST /admin/switch-profile` saves the selection and reports whether a restart is needed

The body has `name`, a profile name or `null`. The route writes the name to `gateway.state.toml`, or deletes that file for `null`, and answers with `profile` and `restart_required`, which is true when the selection differs from the running profile. The running process continues unchanged. Workshop's server restarts its own Gateway after a switch that it requested, and Workshop's UI lacks a control that requests one ([Workshop: how the app is built](workshop-app.md)).

The route leaves the restart to its caller, although the Gateway could restart itself. This form has the smallest effect on a running system, reuses the `restart_required` reply that the config UI renders, and lets a Gateway that runs on its own under the tray continue to run. The route is the one way to select a profile at run time. A staged selection would need Apply, and Apply promotes every pending shadow, so a profile pick from Workshop would also apply unrelated edits that a user staged in the config UI.

### A request for a local model that is still starting gets a 503

`resolve_routed_model` answers a request for a missing name in one of three ways. While the boot load is starting the model's child process, the answer is 503 with the code `model_loading` and `Retry-After: 5`. While a command is active and the config declares the model, for example during its download, the answer is 503 with the code `model_provisioning`. Otherwise the answer is 404. Requests for remote models continue the whole time.

## 6. Queues, tool-call dialects, and outbound TLS

### A dominion is one concurrency limit and one queue that its members share

A `[[dominion]]` entry has an `id`, a `kind` of `remote` or `local`, `max_concurrency` (unlimited when absent), `max_queue` (100 by default), `policy` (`queue` or `reject`), `fair_scheduling` (on by default), and for a local dominion a `vram_gb` budget. An `[[endpoint]]` names a remote dominion and a `[[local_model]]` names a local one, and all members of one dominion admit requests through its one `DominionQueue`. An `[[endpoint]]` that omits the `dominion` key is unlimited, and a `[[local_model]]` that omits it gets a queue of its own, limited to its `parallel` value. When the waiting queue is full the Gateway answers 503 with `queue_full`, and under the `reject` policy it answers 429 with `queue_rejected`. The queue admits waiting callers round-robin by the `X-Runvil-Client` header.

The members share one limit because a concurrency number on each entry would cap only that entry. The limits follow Ray Serve, BentoML, TGI, and Kubernetes APF, which all use a concurrency limit and a bounded queue that refuses new requests when it is full. The round-robin order stops one caller from starving the others. One Gateway process counts a dominion's requests, by design.

### A profile whose models exceed a dominion's `vram_gb` budget fails the load

A local dominion may set `vram_gb`, a budget of GPU memory in gibibytes. A `[[local_model]]` or an `[[stt_model]]` that names the dominion declares its own estimate in `vram_gb`. For every `[[profile]]`, `validate_profile_vram` adds the estimates of the profile's models in each dominion, and a sum above the budget fails the load. So does a selected `[[local_model]]` that omits its estimate. A speech-to-text model counts against the budget, and transcription requests bypass the dominion's queue: the speech-to-text service has an admission gate of its own ([Speech](speech.md)).

The check exists because a profile that overfills the GPU otherwise fails later, as an out-of-memory error in a child process. The owner's rule: "The VRAM is the primary budget, the max concurrency is the secondary budget." A missing estimate fails the load because a budget is meaningful only when it is complete.

### The Gateway emulates tool calls for a model that lacks a tool array

The Gateway rewrites tool calls for a model whose `tool_dialect` is `gemma3_tool_code`. `prepare_request` removes `tools` and `tool_choice` and inserts a system message that describes the tools. `apply_response` turns each `tool_code` fence in the reply into an OpenAI `tool_calls` entry. For `stream: true` the Gateway buffers one whole reply and sends it on as chunks. For a fence that fails to parse, the Gateway returns empty content and a `gateway_warning` field that states the cause. The Gateway is the last component on the request's path and lacks a fallback, so it warns and continues, and the field shows the warning to the caller.

The Gateway does this work so that every client uses one OpenAI-shaped interface. The owner's goal: "The goal is zero dialects." A tool loop therefore uses each model's format, and authors write their prompts without it.

### The Gateway detects a local model's dialect, and a tie or a miss fails the model

When a local chat model starts, `resolve_local_dialect` reads the child's `/props` and `/v1/models` replies and scores two dialects, `openai` and `gemma3_tool_code`, in plain Rust. A metadata file that the Gateway wrote beside the model file supplies the chat template when `/props` lacks one. The function selects the dialect with the highest score. It returns `DialectResolveError::Tie` for two equal top scores and `NoMatch` for an empty score list. Either one fails the model's start, because the alternative is a silent fallback to `openai`.

A remote `[[model]]` takes a `tool_dialect` key that defaults to `openai`. A `[[local_model]]` lacks the key, and an operator leaves a local model's dialect to the detection.

### The Gateway adds a dialect or a protocol when a configured model needs one

The chat handler picks the dialect with one string comparison, and `Protocol` has one variant, `Openai`. An operator who wants Anthropic configures Anthropic's OpenAI-compatible endpoint.

A general translation layer, with a translator trait, is still to be built, and it waits for a second backend that needs it. The owner's rule: "I was thinking we just add them as needed, we don't try to cover every possible model at once."

### Outbound TLS trusts the operating system's certificate store

The Gateway checks the certificates of outbound connections against the operating system's trust store, through `rustls-platform-verifier`, so it trusts a corporate proxy's root certificate that an administrator installed there. Desktop users behind such a proxy therefore get working TLS. The platform verifier is also reqwest's default.

## 7. Local models

### A local model runs as a `llama-server` child process

`LocalRuntime` starts one `llama-server` child for each `[[local_model]]` in the selected profile and registers it in the routing table beside the remote models. The child's `--parallel` value is the entry's `parallel` key, 1 by default. A model of kind `embedding` starts with `--embeddings`, and a `classifier` starts with `--reranking`. A chat model may also declare a draft model and a multimodal projector in sub-tables, each with its own `source` and digest.

The `parallel` key exists because, with one slot, the requests of a concurrent fan-out queue behind each other in the child. A child on Windows starts at below-normal priority, which yields CPU and I/O scheduling to interactive processes while the child loads the model's weights.

### The Gateway downloads model files into its cache and resumes a broken transfer

A model's `source` is a URL or a file path. The Gateway downloads a URL into `<cache_dir>/models`, where `cache_dir` defaults to `~/.runvil`, and checks the `sha256` pin when the entry has one. After one hash that matches the pin, a marker file records the digest, the file's size, and its modification time, and a later start skips the hash while all three match. The cache lists a file when its `<file>.meta.json` sidecar exists, and both download paths write the sidecar before they publish the file. Three routes manage the same cache for other programs: `POST /v1/cache`, `GET /v1/cache`, and `DELETE /v1/cache/{sha256}`. Through them another program can tell the Gateway which pinned Hugging Face file it needs.

A failed download leaves its partial file, and the next attempt asks the server for the remaining bytes, because a restart from the first byte is fatal for a multi-gigabyte download on a connection that resets. A transfer that receives zero bytes for 30 seconds fails, so that Cancel and Quit take effect. The marker is a weaker check than the hash, and the design accepts that: anyone who can write the cache can fake a size and a modification time, and the cache folder has the operator's trust.

### The Gateway respawns a dead `llama-server` when a request fails

`LocalUpstream` owns a model's child process and the options that started it. When a request fails with a transport error and the child has exited, the upstream starts the child again with the same port, alias, and key, and retries the request once. A second respawn within 3 seconds fails with `RespawnCooldown`.

A dead child would otherwise leave the Gateway healthy while every chat request for that model fails with 502 and the code `upstream_transport`. The upstream checks the child only when a request fails, which makes a background watchdog for the children unnecessary. The port, alias, and key are fixed so that the routing table and each `Arc<Model>` remain valid across a respawn. The cooldown prevents a respawn loop on a hard GPU fault.

### The Gateway downloads `llama-server` and picks the build from the GPU

The Gateway finds the `llama-server` program in this order: `[local] llama_server_path`, the `RUNVIL_LLAMA_SERVER` variable, then a managed download with a pinned checksum. On Windows x64 the key `[local] llama_backend` chooses the download. Its default, `auto`, asks `nvidia-smi` for each GPU's compute capability: 12.x selects the project's own `cuda-blackwell` build, another NVIDIA GPU selects `cuda`, and anything else selects `vulkan`.

The server is a download so that the CUDA toolkit, CMake, and a GPU are all unnecessary on a computer that builds Runvil. The standing rule in the `AGENTS.md` of the `runvil` repository is that run-time code downloads native programs and leaves compiling to release workflows.

### The Gateway picks a chat template in a fixed order and refuses a model that lacks one

For a local chat model, `resolve_chat_template_file` picks the template in this order: a path in `chat_template_file`, a bundled family written as `builtin:<family>`, a known replacement for a broken template, and last the template inside the model file. The Gateway bundles templates for the twelve families of the `Family` enum. When all four are empty, the start fails with `MissingChatTemplate`, and the message names the fix.

The three sources ahead of the model file exist to correct bugs in GGUF files after they ship. A missing template fails the start, because the alternative is a silent fallback.

## 8. Cloud providers and the model sheet

### A weekly workflow publishes one sheet of every cloud provider's models

The repository `runvil-cloud-providers` has a GitHub workflow that runs each Monday at 12:00 UTC and on manual dispatch. It runs a binary that calls each provider's model-list endpoint with keys from GitHub secrets, normalizes the answers, and writes one JSON file, `cloud-provider-models.json`. Each provider's part of the file is a slice. The workflow uploads the file as an asset of a release tagged `models`, at one fixed URL, when the model lists differ from the published ones.

The sheet exists because metadata for the models of closed-weight providers, written by hand into a config, becomes stale. The sheet omits every secret, so any Gateway can download it for free, and a provider key of its own is unnecessary. The repository is public because a release asset of a private repository requires authentication, which would stop end users from downloading the sheet.

### Each provider has one Rust file with a public descriptor and private details

Each of the 23 providers has a file such as `anthropic.rs` that exports a `Provider` value: a name, a display name, a `Tier`, the name of its key variable, and its base URLs. The auth header, pagination, field names, and model grouping are private to the file.

A tier is `prime`, `subprime`, `niche`, or `aggregator`. It is an opinion about a provider that the project curates. The sheet records that a model has a setting and copies the provider's own values for it unchanged. A `Provider` describes a vendor in code, and an `[[endpoint]]` is a backend that an operator configured.

### After a failed provider fetch, the binary copies that provider's slice from the previous sheet

The binary downloads the published sheet before it builds a new one. Each slice then gets a status: `ok` for a fresh fetch, `stale` for a failed fetch with a previous slice, `unavailable` for a failed fetch that lacks one, and `static` for a provider with a compiled-in list. A `stale` slice is a copy of the previous one with its original `fetched_at`. The owner's rule: "it should propagate the previous file's data."

A failed download of the previous sheet, other than HTTP 404, stops the binary with a nonzero exit code before it writes a file. An outage would otherwise end in a published sheet that swaps the last good data for empty slices.

### New sheet fields are additive, and the Gateway refuses an unknown schema version

A developer who adds a field to the sheet types marks it `#[serde(default)]`, so a Gateway that is one version behind can read a newer sheet. `schema_version` is 1 and rises for a removal or a rename. The publisher rebuilds from its main branch each week, so until the publisher versions its output, an addition with a default is the only safe kind of change.

The Gateway compares each parsed sheet's version with `ACCEPTED_SHEET_SCHEMA_VERSION`. A cache file with another version counts as absent, because a Gateway has to boot even when its sheet is missing, and the next good download repairs the cache. The sheet types exist in both repositories, and each repository defines its own constant for the sheet's schema version. The script `schema/cross-check.sh` in `runvil-cloud-providers` compares the JSON schemas derived from the two copies.

### The Gateway caches the sheet on disk and checks its age at each launch

After boot, one background task reads `cloud-provider-models.json` from the state directory, `~/.runvil`. It downloads a fresh copy when the file is missing, fails to parse, or has a `generated_at` older than seven days. `GET /admin/cloud-models` returns the sheet to the config UI, and `POST /admin/cloud-models/refresh` forces a download and answers with the result.

The URL is a constant, and `RUNVIL_MODELS_SHEET_URL` overrides it. The sheet is the single source of the list. A live fetch from each provider would use the operator's keys, which the owner's rule in the next entry excludes.

### The Cloud page adds a model through the existing config path and leaves keys to the Secrets page

The Cloud page lists the sheet's models under a row of kind buttons, a provider dropdown grouped by tier, and a family dropdown. The Add button opens a dialog filled from the sheet entry. On confirm, the UI merges a `[[model]]` entry, and an `[[endpoint]]` when the config lacks one for that provider, into the config document and sends the whole document to `PUT /admin/config`. That path validates and reverts correctly, and a second write path would double the places where a write can fail.

The new endpoint's `api_key` is a `${VAR}` reference, and the operator enters the value on the Secrets page. The owner's rule: "no keys. just download cloud-provider-models.json". The page disables Add for a provider whose slice lacks `openai_base_url`, the base URL of an OpenAI-compatible API.

## 9. Boot, the command queue, restart, logs, and progress

### The Gateway binds its listener before it provisions any model

On start the Gateway reads its config, binds the listener, builds the routing table from every `[[model]]`, writes its discovery file, `gateway.json`, and reports ready. When a profile is selected, it then puts one `LoadProfile` command on the queue. That command downloads the selected profile's model files, starts the child processes, adds each ready model to the routing table, and loads the speech-to-text engine once.

The order exists so that the Gateway is live the moment it starts. The bound port is its readiness signal.

### One queue runs slow work one command at a time

`CommandQueue` has one worker that takes commands in the order they were queued. Two commands run in production: `LoadProfile`, the boot load, and `ApplyConfig`. Each has a `CancellationToken` that the worker checks at download chunks and between phases. `POST /admin/queue/cancel` cancels the active command, and `GET /admin/status` reports the active and waiting commands. Quit and `POST /shutdown` cancel the active command first, and the shutdown waits at most 5 seconds for the worker.

The rule is that every piece of work that takes time is a command in this queue, and that any command can be cancelled. With one command at a time, each download has the bandwidth to itself.

### A Gateway process takes a lease and then writes a discovery file

Before it logs, binds, or writes shared state, a Gateway process takes an operating-system file lock, `GatewayInstanceLease`, in the run directory, `~/.runvil/run`. A process that loses the lock waits up to 10 seconds for the winner's discovery file and then exits. The lock is the first step because a duplicate launch would otherwise cause side effects before one process owns the shared state.

After the bind, the winner writes `gateway.json` into the run directory with its port, key, process id, boot time, and version, and removes the file at a graceful stop. The default config binds port 0, so the operating system assigns a free port and a port conflict is impossible. How Workshop reads the file and launches a Gateway is on [Workshop: how the app is built](workshop-app.md).

### `runvil-gateway` starts the server by default and shows a tray icon on a desktop

The binary has one subcommand, `diagnostics`, and every other launch starts the server. `--no-tray` runs headless for servers and CI, `--browser` opens the Settings page after the bind, and `--login` marks a start from the login entry and overrides `--browser`. A second launch opens the running Gateway's Settings page.

`diagnostics` prints a JSON report of the state directory, the config path, the log files, and the discovery file, so that a person who is new to the config can still find the files of a failed run. On a desktop the Gateway shows a tray icon with a status line, Workshop, Settings, Launch at Login, and Quit. The Gateway is a background process, so on an installed system the tray icon is the part of it that a person sees.

### The Gateway asks for a restart with exit code 75, and the supervisor that acts on it is still to be built

`POST /admin/restart` cancels the active command and stops the Gateway with the exit code `RESTART_REQUESTED`, 75. The route answers 409 with the code `restart_unavailable` unless the process started with the hidden flag `--supervised`. A supervised Gateway also watches its stdin and stops when the pipe closes. The first Apply after a start copies each real file to `<file>.prev` before it replaces it, and the next successful start deletes the copies.

These parts are the Gateway's half of the restart design. The other half is a supervisor process that starts the Gateway with `--supervised`, starts a new one after exit code 75, and restores the `.prev` files when the new one fails to boot. The supervisor is still to be built (section 11). The Gateway reports the request through its exit code and leaves the launch of its successor to the supervisor, because a Gateway that started its own successor would race it for the lease and the port.

### The log has fixed limits, a fixed location, and a redaction pass

The `gateway-logging` crate formats each record on the thread that logs it and puts it on a queue of 8,192 records and 32 MiB. One worker thread writes the queue to `<state dir>/logs/gateway.log`. A new record that finds the queue full evicts the oldest Debug, Trace, or Info record at or below its own level. When the queue lacks one, the producer waits at most 25 ms, and then the logger drops the new record and counts the loss. A segment rotates at 16 MiB, and five numbered segments remain. A redaction pass masks classified fields, such as credentials and cookies, before the formatter runs.

The queue has a bound so that a stalled sink consumes a bounded amount of memory. The cost is deliberate: the logger loses records while a sink is stalled for good, and summaries and emergency diagnostics are part of its contract. The location is fixed in code, because a failure to find or parse the config needs a destination for its log records.

### Progress is a busy flag and one line of text

Code that starts slow work calls `ProgressHub::begin(text)` and gets an `Activity` guard. It updates the text with `set_text`, and the activity ends when the guard drops. The hub publishes one `Progress { busy, text }` snapshot per change. `GET /admin/progress` streams the snapshots as server-sent events, `GET /admin/status` includes the current one, and the tray shows the text.

The terminal shows only log lines: the terminal is for logs, and the UI is for progress.

## 10. Planned work

Each of these items is still to be built.

- **The supervisor and Restart to Apply.** A parent process for the Gateway and a Restart to Apply button in the config UI. The process starts the Gateway with `--supervised`, starts a new one after exit code 75, and restores the `.prev` config files when the new one fails to boot. `POST /admin/restart` answers 409 unless the Gateway started with `--supervised`, and the UI shows a banner that asks the operator for a restart.
- **An operating-system service.** A headless service with a thin tray client for each user. The Gateway is a process that a person, Workshop, or the login entry starts, and the Linux release archive has a sample systemd unit.
- **A choice among endpoints.** A pick among a model's `endpoints` by load, with fallback chains and priority weights. The first priority is a lane for speech requests in a dominion's queue. The Gateway uses the first endpoint in a model's list, and a queue admits waiting callers round-robin.
- **A second protocol.** A translator for providers that lack an OpenAI-compatible API. `Protocol` has one variant, `Openai`, and the Cloud page disables Add for seven of the 23 providers.
- **The thinking switch for remote providers.** A per-model setting that translates `chat_template_kwargs.enable_thinking` for a provider that ignores the field. The Gateway forwards the field as the client wrote it.
- **Normalized metrics.** One format of timing record for every backend. The Harness parses the llama.cpp and vLLM timing fields itself.
- **Timeouts.** A configurable limit for plain requests and an idle deadline for streams. A plain request has a fixed limit of 120 seconds, and a stream has only the 10 second connect timeout.
- **Image generation.** An OpenAI-shaped image route with a local `sd-server` child process. The Gateway's inference routes cover chat, embeddings, rerank, and speech.
- **Sheet upkeep.** An age limit for `stale` slices, versioned asset names at the publisher, and workflow secrets for eight providers. The binary copies a `stale` slice forward for as long as the provider's fetch fails, the workflow overwrites one asset at one URL, and a provider whose key the workflow lacks counts as a failed fetch.
- **Routing reload for an env change.** A reload of the routing table when only the env file changes, so that a new key works before the restart. An Apply that changes the env file reports `restart_required`, and the new value applies at the next start.

## 11. Known gaps

Each item names a feature that is half built, a config key that the code leaves unused, two parts of the code that disagree, or a place where a doc comment or a guide chapter differs from the code.

**Restart**

1. The restart feature is half built. The Gateway's half is in the code: `POST /admin/restart` in `app/src/admin/walled/restart.rs`, the hidden options `--supervised` and `--preferred-port` in `app/src/cli.rs`, `RESTART_REQUESTED` and `stdin_watchdog` in `app/src/shutdown.rs`, and the `.prev` copies in `app/src/commands/apply.rs`. The supervisor's half is still to be built: the app lacks a supervisor module, the Gateway's own tests are the one caller in `runvil-gateway` or `runvil` that passes `--supervised`, and the UI lacks the Restart to Apply button. Workshop's `GatewaySupervisor` thread launches a missing Gateway with an empty argument list ([Workshop: how the app is built](workshop-app.md)). `POST /admin/restart` therefore answers 409 on every Gateway that a shipped launcher starts. Apply writes the `.prev` copies, and the next start deletes them unused.
2. The doc comment on `supervised` in `app/src/admin/open/status.rs` says the config UI offers Restart to Apply. `main.ts` shows the banner `Restart the gateway to apply these changes.`, and the button is still to be built.

**The config crate**

3. The Gateway parses `default_max_tokens` and leaves it unused. `ModelConfig` in `config/src/config.rs` documents the key as "A `max_tokens` default supplied when the caller omits one." The config crate validates it, the UI edits it, and guide chapter 3 lists it. The config crate's own tests are the one caller of its accessor, `default_max_tokens()`, in `runvil-gateway`, and the chat handler forwards `max_tokens` as the caller sent it.
4. One error text names a key that the config lacks. `Config::validate` in `config/src/config/validate.rs` reports an empty bearer key as `server.key must not be empty`, and the key is `[server] api_key`.
5. One doc comment calls a local model outside every dominion unlimited. The comment on `DominionConfig` in `config/src/config.rs` says that an endpoint or a local model "without a `dominion` is unlimited". `resolve_admission` in `local/src/runtime/launch.rs` limits such a local model to its `parallel` value, and the code queues up to 100 waiting requests. Item 15 lists the same statement in guide chapter 8.

**The Gateway guide, in `runvil-docs`**

6. A model that is still downloading. Chapters 1 and 7 say the Gateway answers 503 with `model_loading`, and chapter 7 adds `Retry-After: 5`. `resolve_routed_model` answers `model_provisioning` during a download, and `retry_after_seconds` in `app/src/error.rs` sets the header only for `ModelLoading`.
7. The config version. Chapter 2 calls the config "a single version-2 `gateway.toml` file" above an example with `config-version = 0`. `Config::validate` requires 0.
8. The `[workshop]` table. Chapters 2, 9, 10, and 11 describe it as a section that loads. `RawConfig` refuses unknown keys and lacks a `workshop` field, and `renderWorkshop` in `settings-page.ts` edits `stt`.
9. The diagnostic for `include`. Chapter 2 says an `include` chain fails with its own diagnostic and lists a "removed layout feature" kind. `ConfigErrorKind` in `config/src/api_error.rs` has six kinds, and that kind is absent.
10. Capability keys. Chapter 3 shows a `[model.capabilities]` table with seven keys. `Capabilities` is flattened into the model entry, the test `parses_model_capabilities` writes the keys directly in `[[model]]`, and the struct has two more keys, `voices` and `provider`.
11. UI pages. Chapter 10 names six views. `router.ts` defines seven `PageId` values, and the seventh is `cloud`.
12. Progress. Chapter 11 says `GET /admin/progress` replays live operations and that status reports a progress fraction. `progress_sse_response` sends one `Progress` snapshot per change, and `ActiveCommandReply` has `name` and `started_at`.
13. Inline Apply. Chapter 9 says an Apply that touches only boot-owned sections runs inline. `capture_apply` returns `ApplyPlan::Inline` when `gateway.toml.next` is absent, so every config shadow queues a command.
14. Log files. Chapter 1 says five previous runs remain on disk. `RETAINED_SEGMENTS` in `logging/src/config.rs` is five segments of at most 16 MiB each, and a run that writes more than 16 MiB fills several of them.
15. A local model outside every dominion. Chapter 8 calls an endpoint or a local model that omits the `dominion` key unlimited. Chapter 4 and `resolve_admission` in `local/src/runtime/launch.rs` limit such a model to its `parallel` value, and the code queues up to 100 waiting requests.

## 12. Questions for the owner

Each question asks for the reason behind a part of the design.

**The API**

1. Why does each `[[model]]` entry have two names, `name` for callers and `upstream` for the backend?
2. Why does each route check a model's kind before it calls the backend?
3. Is the Gateway meant to apply `default_max_tokens`? Section 11 shows that it leaves the key unused.

**Configuration and profiles**

4. Why does a config value name a secret through a `${VAR}` reference?
5. Why is the selected profile stored in a state file beside the config?

**Local models**

6. Why does the Gateway run `llama-server` as a child process, when it could link llama.cpp into its own process?
7. Why is run-time code forbidden to compile a native dependency?
8. Why does a tie or a miss in dialect detection fail the model's start?
9. Why does `auto` choose Vulkan after a failed GPU probe, when it could report the failure?

**Restart**

10. Why is `POST /admin/restart` a separate request from Apply?

## 13. Method and limits

- **Code.** The page describes `runvil-gateway` at commit `9369a7d`, `runvil-cloud-providers` at `1817cc4`, `runvil-docs` at `cfd69a4`, and `runvil` at `fa9936f64`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.

*2026-11-10 16:44 - Claude Fable 5.2*
