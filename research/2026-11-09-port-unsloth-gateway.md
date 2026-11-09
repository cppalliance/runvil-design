<!-- sources: github.com/unslothai/unsloth at c4052051f488334b2a59d7488b69b70f02375163 (2026-11-09); runvil-gateway at 9369a7d902a4b083e68f59a0353bcb9227c6c0d3 (2026-10-31, v0.3.0); runvil Workshop; llama.cpp tag b10082; web research 2026-11-09 -->

# Porting Unsloth Studio's model serving to Rust: assessment and plan

## Bottom line

- **A full port of Unsloth Studio to Rust is not a realistic project.** Studio's ML core (training, diffusion, MLX, export) is built on torch, transformers, diffusers, and Unsloth's own kernels, so it stays in Python no matter what. Replacing just the FastAPI server with axum means rewriting roughly 336k lines of non-ML Python, much of it business logic, against a codebase that took 517 commits in `studio/` in one week. Several engineer-years, and a fork would never catch up.
- **The Tauri part already exists.** Unsloth Desktop is a Tauri 2.12 app. On the Runvil side, Workshop is a Tauri 2.11 app that already ships Gateway as a pinned sidecar and embeds Gateway's config page.
- **Scoped to what overlaps with Gateway (serving local and remote models, plus the UI to download, configure, and manage them and their files), this stops being a port.** Gateway is already the Rust + axum version of that slice. What remains is a list of features Studio has and Gateway lacks.
- **The model management UI is the largest gap by volume.** Studio spends about 99k to 117k frontend lines on discovering, downloading, configuring, and managing models; Gateway's config UI is about 12k. Gateway is ahead on config review (diff, apply, revert), pinning, and cloud pricing, but has no download progress, no runtime control, no LoRA, and a shallow local library. Two of Gateway's own design choices (models fixed at boot, downloads deferred to the next boot) block the biggest UI gaps.
- **Of Studio's 218 routes in Gateway's scope, 17 have a direct Gateway equivalent, 61 a partial one, 109 none, and 31 do not apply.** Gateway has 46 routes. Most of the partial matches go through a single whole-document config route.
- **Gateway can already be configured over HTTP, but not in a way an agent can use safely.** `PUT /admin/config` replaces the whole document, has no revision check or dry run, and reports one error string. There is no OpenAPI document or config schema, no harness model can reach the admin routes, and some settings are reachable only through CLI flags, environment variables, or a restart that a standalone Gateway cannot perform. The fix is three layers: a complete resource-oriented admin API that stages every write, docs generated from code, and a small curated tool set for harness models with a separately confirmed apply.
- **The cheapest high-value API gap turned out to be much cheaper than first estimated.** The llama-server build Gateway pins (b10082) already serves Anthropic `/v1/messages`, `count_tokens`, OpenAI `/v1/responses`, legacy `/v1/completions`, and `/tokenize`. Gateway can pass these through to local children instead of translating them: about 1 to 2 weeks, and it lets Claude Code and Codex use local models through Gateway.
- **There is no "Rust PyTorch" that can load arbitrary Hugging Face safetensors models.** A safetensors file holds tensors but no compute graph; the architecture lives in Python modeling code. Every non-Python engine hand-ports architectures. mistral.rs is the only broad Rust engine, with excellent LoRA support but weak Windows packaging and no auth.
- **Recommended path:** add pass-through routes; serve Unsloth fine-tunes as GGUF through llama-server; make the admin API complete and generate its docs from code; ship the agent tool layer; finish Gateway's two existing UI plans (download during Apply, Restart to Apply) and build a real downloads panel on top; then runtime control, tool-call repair via Apache-2.0 parser crates from SMG, a Gateway-built mistral.rs wrapper for direct safetensors serving, and vLLM and SGLang as a Linux-only opt-in.
- **License caveat:** Studio's serving and UI code is AGPLv3 and Gateway is BSL-1.0. Use Studio as a behavior reference, not a source to translate.

## 1. What Unsloth Studio is

Studio is Unsloth's local no-code app for running, training, and exporting open models. It ships two ways: a web UI (`unsloth studio`, FastAPI on 127.0.0.1:8888 serving a React frontend) and Unsloth Desktop, a Tauri 2.12 shell that installs Python with uv on first run and supervises the same backend in `--api-only` mode.

| Area | Files | Lines |
|---|---|---|
| Backend Python (non-test) | 953 | about 735k |
| Backend tests | 2,329 | about 1.18M |
| Frontend TS/TSX | 2,339 | about 678k |
| Tauri shell Rust | 54 | about 53k (29k non-test) |

The backend is not a thin web layer:

- About 700 routes, about 627 Pydantic models, 12 middleware layers. `routes/inference.py` alone is 52.5k lines with protocol translation, stream rewriting, admission control, and GPU memory estimates inline.
- The FastAPI process itself runs the llama-server manager (`llama_cpp.py`, 42k lines), diffusers image and video generation, the RAG embedder, and torch-based hardware detection. Only training, transformers inference, export, and a few others run in spawned workers.
- Under 1% of the test suite is black-box HTTP testing; about 80% imports Python internals.
- The frontend depends on FastAPI behavior: it parses Pydantic 422 `detail[].loc` errors and fetches `/openapi.json` at runtime.
- One part is easier than expected: the main workers already exchange JSON-safe dicts through one shared spawn function, so swapping their IPC is contained.

## 2. Full port: why not

| Scope | What moves to Rust | Rough effort |
|---|---|---|
| Thin front | axum reverse proxy plus auth, settings, chat history, static files | 1 to 2 months, one developer |
| GGUF chat without Python | Above plus llama-server manager, OpenAI and Anthropic compatibility, tool-call parsing, tools, Hugging Face downloads, hardware detection | about a year, small team |
| Full server replacement | All non-ML Python; Python only as workers | several engineer-years |

Estimates are judgment from code size and complexity, not measurement (low-medium confidence). AI-assisted translation would shorten typing but not verification, and verification is the bottleneck because the tests do not transfer. At roughly 500 commits a week, a year-long port would chase about 25,000 upstream commits.

## 3. Scoped to Gateway: serving local and remote models

Excluding Studio's agent features (server-side tool loop, web search, code execution, browser, MCP, RAG), training, export, and image, video, and audio generation, Studio's serving slice is about 180k to 213k lines of Python. Gateway is about 113k lines of Rust. The gap overstates the work: most of Studio's extra volume is GPU fitting heuristics and hardware edge cases.

### What both already have

- OpenAI chat completions with streaming, embeddings, `/v1/models`
- Managed llama-server children with crash respawn, pinned llama.cpp downloads
- Chat template resolution, MTP speculative decoding, vision projectors
- Hugging Face search and resumable downloads, admission control

Gateway also has things Studio does not: rerank, realtime transcription on whisper.cpp, and strict pinning of every artifact.

### What Studio has that Gateway lacks

| Gap | Studio size | Effort guess |
|---|---|---|
| Pass `/v1/messages`, `count_tokens`, `/v1/responses`, `/v1/completions`, `/tokenize` through to llama-server children, plus `x-api-key` auth | n/a (llama-server does the work) | 1 to 2 weeks |
| Translation for clients whose protocol does not match the upstream (for example Claude Code to an OpenAI-only cloud provider) | about 6k | 2 to 4 weeks, optional |
| Tool-call repair for local models, using SMG's parser crates (section 9) | about 10k in Studio | 2 to 4 weeks |
| Reasoning effort controls (`--reasoning-budget`, template kwargs) | about 1k | about 1 week |
| Native Anthropic and Gemini upstream protocols | about 1.6k each | 1 to 2 weeks each |
| LoRA adapters on GGUF models in `[[local_model]]` | n/a | 1 to 2 weeks |
| Load, unload, swap models at runtime; idle unload | about 8k | 3 to 6 weeks |
| Automatic VRAM fitting and offload | about 25k to 30k | 1 to 2 months basic, open-ended for parity |
| AMD, Apple, Intel, multi-GPU detection | about 9k to 12k | 2 to 4 weeks |
| Linux CUDA and ROCm llama.cpp builds, runtime updates | about 15k to 20k | 2 to 4 weeks, mostly release work |

Effort assumes one experienced developer working in Gateway's existing style (low-medium confidence). The UI for downloading, configuring, and managing models is covered separately in section 4.

Notes on the biggest items:

- **Tool-call repair** matters because local models often write tool calls as text. Studio converts them for Hermes/Qwen, Gemma, Qwen3-Coder, Mistral, Llama 3, DeepSeek, GLM, Kimi, MiniMax, and others. Gateway has one emulated dialect (`gemma3_tool_code`) and otherwise relies on llama-server's own parsing. If routes are passed through, repair must handle three response formats or apply only to chat completions.
- **Automatic fitting** is less work than its line count suggests: Studio starts from llama-server's built-in `--fit` and wraps it with retry ladders (flash attention off, smaller KV, drop tensor split, CPU fallback) and MoE expert offload via `--n-cpu-moe`. The bulk of its code is edge cases (unified-memory iGPUs, Metal budgets, Windows commit limits, MIG).

### Design decisions Gateway would have to make

- **Model set.** Gateway fixes the model set at boot; Studio loads, swaps, and auto-downloads at runtime. Gateway already has working `ProvisionModel` and `UnloadModel` handlers in `app/src/commands.rs`, but only tests enqueue them. llama-server's own router mode (`/models/load`) is another option.
- **Pins.** Gateway requires a sha256 pin on every https download; Studio downloads whatever the user picks.
- **Sizing.** Gateway asks the user for `vram_gb` and `gpu_layers`; auto-fit replaces those with computed values.

### What to skip

- Studio's chat conversation UI. Workshop is already the chat client. Studio's model management UI is a different matter and is in scope (section 4).
- Codex subscription OAuth login and multi-account auth.

### A Gateway bug found along the way

The docs say Gateway maps 181 Hugging Face repo IDs to template families automatically. In `local/src/launch_templates.rs`, `inspect_chat_template` only puts the mapped family in `detected_family` for display; the launch decision ignores it. A mapped model with no embedded template still refuses to launch. Either the docs or the code should change.

## 4. Model management UI

Downloading, configuring, and managing local and remote models and their files is squarely in the Gateway overlap. It is separate from the chat conversation UI, which Workshop already covers.

### Where the UI lives

- **Studio:** about 99k to 117k frontend lines across the model hub, model picker, loaded-models panel, provider dialogs, and the relevant settings tabs, backed by about 92k to 140k backend lines (estimates).
- **Runvil:** Gateway's config UI at `/config/`, about 12.3k lines of TypeScript across seven pages (Settings, Discover, Local, Remote, Cloud, Profiles, Secrets), plus about 1.6k in `shared-ui` and 3.2k lines of admin Rust.
- **Workshop** has no model management UI of its own. It embeds Gateway's config page and adds a Ctrl+/ chat model picker. So this work lands in Gateway's config UI, not in Tauri. Workshop's proxy refuses `/admin/progress` and `/v1/cache` today, so a live downloads panel needs a matching Workshop change.

### Where Gateway is already ahead

- **Config review.** Dirty markers, pending chips, a Review dialog with secrets redacted, Apply with cancel, and Revert All. Studio has no review flow and no settings export.
- **Cloud catalog.** Models from 23 providers via a weekly sheet, with context, max output, capability pills, price per million tokens, and one-click Add. Studio keeps pricing in its backend only.
- **Per-model editing.** Every knob Gateway supports is editable, the `gpu_layers` slider knows the GGUF layer count, and the chat template control explains its source and reason.
- **Integrity.** Every download is hash-verified; Studio checks declared sizes only.

### Side-by-side comparison

| Area | Studio | Gateway today |
|---|---|---|
| Discovery | Hugging Face search with six sorts (Latest, Newest, Trending, Downloads, Recently updated, Likes); filters for format (GGUF, checkpoint, MLX, NPU), nine capabilities, parameter range, context range, uploader, and fits on device; curated rows (Unsloth Trending, Latest Unsloth, Fine-tune ready); recent searches; README rendered with math and diagrams; license and gated chips; ModelScope as a switchable source | Hugging Face only; debounced search by keyword, `user/repo`, or pasted URL; GGUF locked on; workload toggles (Chat, Embedding, Reranker, STT, Image, TTS); three sorts; one page of 30 results, no pagination; model card with publisher, badges, downloads, likes; sanitized README (1 MiB cap); no curated lists |
| Quant selection | Per-quant fit badges (full GPU, over budget, partial, RAM fallback, streamed, does not fit) that mirror the loader's own GPU placement logic; variants sorted by fit then size; Recommended filter; "X left" on partial downloads; update-available flag; mmproj and MTP drafter files pulled in automatically | Groups GGUF files by quant and sums sizes; SHA-256 from Hugging Face LFS for single-file quants; fit is file size times 1.2 against VRAM and RAM, ignoring context and KV cache, first NVIDIA GPU only; Recommended is the largest full fit, else the largest partial fit; multi-part quants disabled; no companions in Discover |
| Downloads | Floating download manager with bytes, speed, ETA, progress bar; cancel keeps partial files; resume on restart; no pause; transport choice (auto, Xet, HTTP), speed limit, per-download target folder; staged queues for multi-file plans; token-rejected dialog; one subprocess per download; jobs re-adopted after restart; integrity by declared size, not hash | No download UI, by design since the August plan; the Download button stages a config entry; local-model changes report `restart_required` and the file downloads at the next boot; `/admin/progress` is a busy flag plus one text line; `/v1/cache` supports byte-level progress but the UI never calls it and Workshop's proxy refuses it; every file is sha256-verified |
| Local library and files | On Device tab with sizes, sort by recent, name, size, or format, partial markers, active state; delete with an impact preview (bytes freed, shared companions kept); reveal in Explorer, Finder, or Files; custom scan folders with health status and a server-side folder browser; adopts the HF cache, LM Studio, oMLX, Ollama (reads manifests, links blobs), and Hermes folders; "Free up space" for orphaned companions; cache inventory with per-cache purge; changeable models folder (keeps history, does not move files); low-disk notice | Lists configured entries with downloaded size and path, or "Not downloaded"; delete by sha256; reveal in file manager (cache paths only); an "Unconfigured files" section with Adopt and Delete; `.part` partials hidden and never cleaned up; size sort falls back to name; `cache_dir` editable with a "does not move files" warning; no file picker; no reuse of HF, Ollama, or LM Studio caches; disk bar warns at 70% and 90% |
| Per-model configuration | Context, KV cache type, GPU layers (auto or manual), MoE layers on CPU, GPU pick and ordering with tensor split, VRAM budget, parallel slots, batch and micro-batch, load mode, context checkpoints, prompt-cache RAM, reasoning budget, disable-vision toggle; speculative decoding (MTP, DFlash, ngram, MLX drafters); chat template editor with validation; extra llama-server args validated against that build's flag list; named run settings with "always load with these"; sampling, system prompt, and presets in the chat settings sheet; stored in browser localStorage plus a server copy | Every `LocalModelConfig` field: `gpu_layers` slider with "Max" and the GGUF layer total, flash attention, K and V cache types, context on a log scale from 512 to 262144, `n_predict`, `parallel` 1 to 16, thinking mode, chat template control showing effective source, detected family, and reason, source and sha256, dominion, `vram_gb` (only with a dominion), draft-mtp speculative decoding, multimodal projector, capability fields; profiles act as presets with a summed VRAM estimate; no per-model presets |
| Runtime control | Draggable loaded-models panel (chat, image, video, speech, embedding) with resident MiB, GPU or CPU placement, and eject; load progress bar (mmap phase, bytes loaded); multi-model mode (off, fit, always); keep-resident (mlock); idle auto-unload; engine install and rollback (vLLM, SGLang, WSL); llama.cpp backend switching and updates; global log viewer linked from load failures | No start, stop, or unload per model; the model set is fixed per boot; Running/Stopped dot and declared VRAM total; per-model stderr tails exist in backend diagnostics but are not shown; `supervised` and `loading_models` from `/admin/status` go unused |
| Applying changes | Changes take effect at runtime, because models load and unload live | Dirty markers, pending chips, a Review dialog with secrets redacted, Apply overlay with Cancel, Revert All; remote changes apply live; local changes need a restart; `/admin/restart` exists but no UI button, and the October Restart to Apply plan has three open to-dos |
| LoRA and fine-tunes | Fine-tuned, exported, and local rows in the picker with QLoRA VRAM estimates and their base model; import by scanning folders; no UI to attach an adapter to an arbitrary base | None |
| Remote providers | 22 presets including Codex subscription OAuth, OpenAI, Anthropic, Gemini, DeepSeek, OpenRouter, Mistral, Ollama, llama.cpp, vLLM, and Custom; base URL override and extra headers; test connection with a model count; load models from the provider plus manual IDs; per-connection enabled models; global and per-provider enable switches; capabilities from a models.dev-style catalog; keys RSA-encrypted in the browser and AES-GCM encrypted in SQLite; pricing tables in the backend only | Remote page to add, edit, and delete `[[model]]` entries; Settings lists endpoints with id, base URL, redacted key, and dominion, protocol locked to `openai`; Cloud page from a weekly sheet of 23 providers with filters by kind, provider tier, and family, columns for context, max output, tools, images, thinking, and price per million tokens, and one-click Add that writes the endpoint and model; keys stored in `gateway.env` via the Secrets page and applied after a restart; test connection only for Hugging Face; no live model listing |
| Model picker | About 18k lines with Downloaded, Recommended by hardware fit, and Connected sections; pinned models with drag reordering; quant sub-pickers reuse the hub's fit logic | Workshop's Ctrl+/ dropdown in the agent toolbar, chat-capable model ids only, remembered per profile; the config UI has a profile switcher instead |
| Hardware | Per-GPU VRAM used and total, utilization, temperature, backend, unusable GPUs with reasons, AMD Windows driver warning, RAM, disk | `/admin/system`: CPU, RAM, the cache drive, and the first NVIDIA GPU via NVML, polled every 5 seconds; no AMD, Intel, or Apple GPU data |
| Config backup | No export; shareable run-config links; server copy of per-model config | No import or export; no raw TOML view |

### Gaps to close

| Gap | Effort guess |
|---|---|
| Downloads: download during Apply with a live progress panel (bytes, speed, ETA, cancel), plus the Workshop proxy change | 2 to 3 weeks |
| Applying changes: Restart to Apply button and supervisor from the October plan | about 1 week |
| Runtime control: loaded-models panel with memory, eject, and per-model log tails | 1 to 2 weeks of UI on top of the runtime load and unload backend (section 3) |
| Quant selection: KV-aware fit estimate from GGUF metadata, all GPUs, automatic companions, split GGUF | 2 to 3 weeks |
| Local library: full cache inventory, partial cleanup, file picker, adopting outside caches, working size sort | 2 to 4 weeks |
| Discovery: capability, size, context, and fits-on-device filters, pagination, curated rows | 1 to 2 weeks |
| Per-model configuration: MoE layers on CPU, GPU pick and tensor split, batch sizes, reasoning budget, chat template editor, validated extra args, named presets | 2 to 3 weeks with the backend knobs |
| LoRA: adapter section on the Local page | about 1 week of UI, with the GGUF LoRA backend item |
| Remote providers: test connection and live model listing per endpoint | 1 to 2 weeks |
| Hardware: per-GPU panel for all vendors | about 1 week of UI, with the hardware detection backend item |
| Config backup: export and import, raw TOML view | a few days |

Total: roughly 2 to 4 months of UI work for one developer on top of the backend items, low-medium confidence. The first three rows are the core and take about 4 to 6 weeks. Adopting files from outside caches has to either hash them on adoption (slow for large models) or record them as unpinned local paths, which Gateway already allows.

### The design tension

- Gateway's August plan (download-on-apply) removed the old separate download flow to kill three bugs: orphaned files after a revert, models invisible until refresh, and stuck progress bars. The plan says files download during Apply through `ArtifactStore::ensure_model`.
- The code defers further than the plan: local-model changes report `restart_required` and download at the next boot (`boot_load.rs`: "an apply swaps only the remote table").
- Doing what the plan describes, downloading during Apply with byte-level progress, keeps all three fixes and gives users a real downloads panel. The `provision_model` handler, with cancellation, already exists in `commands.rs`. Starting the new model without a restart is the runtime load and unload item in section 3.

### Smaller issues found in the config UI

- Embedding and reranker repos picked in Discover are staged as `kind: "chat"`; only STT is special-cased.
- `mmproj-*-f16.gguf` files appear to be grouped into the F16 quant by the grouping regex (inferred from the code, not tested).
- Partial `.part` downloads are hidden and never cleaned up.
- Size sort on the Local list falls back to name order.
- `10-config-ui.md` documents six views; the code has seven (Cloud is undocumented), and its Recommended rule is out of date.

## 5. Routes in scope: Studio vs Gateway

This section lists every route in Gateway's scope: serving local and remote models, and managing them (discovery, downloads, files, configuration, runtime, providers, hardware, auth). Studio's chat storage, agent tools, RAG, training, export, data recipes, datasets, and image, video, and audio generation routes are excluded; they account for 480 of its 703 route decorators.

- Studio: 218 routes in scope.
- Gateway: 46 method and path pairs on 43 paths (default build).

How to read the Gateway column in section 5.2:

- **Same:** Gateway has a direct equivalent.
- **Partial:** Gateway covers part of it, usually through the whole-document config API or a narrower route.
- **None:** no Gateway equivalent.
- **n/a:** does not apply to Gateway's design (Python-only, MLX-only, multi-user accounts, Studio-specific notices, training outputs).

| Studio group | Routes | Same | Partial | None | n/a |
|---|---|---|---|---|---|
| Serving (OpenAI, Anthropic, llama-server compatible) | 24 | 3 | 1 | 19 | 1 |
| Model lifecycle and runtime settings | 25 | 1 | 4 | 14 | 6 |
| Discovery and hub proxying | 25 | 0 | 6 | 17 | 2 |
| Downloads | 17 | 0 | 8 | 6 | 3 |
| Local library and files | 35 | 6 | 15 | 14 | 0 |
| Per-model configuration | 10 | 0 | 4 | 4 | 2 |
| LoRA and fine-tunes | 5 | 0 | 0 | 2 | 3 |
| Remote providers | 20 | 2 | 6 | 8 | 4 |
| llama.cpp runtime and managed engines | 20 | 0 | 4 | 16 | 0 |
| Hardware and system | 7 | 3 | 3 | 0 | 1 |
| Auth | 16 | 2 | 4 | 3 | 7 |
| Other model settings | 14 | 0 | 6 | 6 | 2 |
| **Total** | **218** | **17** | **61** | **109** | **31** |

Notes:

- Most "Partial" entries map to Gateway's whole-document `PUT /admin/config`. That is the main reason section 6 proposes per-resource routes.
- Studio mounts its inference router twice, so every Studio path shown under `/v1` also answers under `/api/inference`. That includes Studio's own management routes (`/v1/load`, `/v1/unload`, `/v1/status`, `/v1/estimate-memory`, and others), plus 27 out-of-scope routes that leak under `/v1` the same way.
- Ten `/api/models/*` routes are older twins of `/api/hub/*` routes. Twins share a row below.
- At least 57 of Studio's in-scope routes are owner-only; a few also refuse API keys and need a UI session.

### 5.1 Gateway's routes

Auth tiers: **bearer** admits the key, the config UI's session cookie, or a keyless loopback caller when `trust_loopback` is on (the default). **Walled** does the same but first refuses any non-loopback peer. Feature gates are in parentheses.

| Method | Path | Auth | Purpose |
|---|---|---|---|
| POST | `/v1/chat/completions` | bearer | Chat, routed by model to a remote endpoint or a local llama-server child; SSE when streaming |
| POST | `/v1/embeddings` | bearer | Embeddings for embedding models |
| POST | `/v1/rerank` | bearer | Rerank in the Jina and Cohere shape |
| POST | `/v1/audio/speech` | bearer | Speech synthesis relay to remote speech models |
| GET | `/v1/audio/voices` | bearer | Voice catalog across speech models |
| GET | `/v1/models` | bearer | Model catalog with kind, context, thinking mode, capabilities |
| POST | `/v1/audio/transcriptions` | bearer (stt) | Batch transcription on whisper.cpp |
| GET | `/v1/realtime?intent=transcription` | bearer plus Origin check (stt) | Realtime transcription over WebSocket |
| POST | `/v1/tools/web_search` | bearer (web-search) | Brave web search |
| GET | `/v1/cache` | bearer (local) | List cached blobs with path, sha256, size |
| POST | `/v1/cache` | bearer (local) | Ensure a blob is cached; SSE byte progress on a miss |
| DELETE | `/v1/cache/{sha256}` | bearer (local) | Delete one cached blob |
| GET | `/health` | none | Liveness |
| GET | `/admin/profiles` | bearer | Profile names |
| POST | `/admin/switch-profile` | bearer | Persist the profile selection; takes effect at restart |
| GET | `/admin/status` | bearer | Running profile, models, local children, queue, endpoints, speech |
| GET | `/admin/progress` | bearer | SSE of a busy flag and one status line |
| POST | `/admin/queue/cancel` | bearer | Cancel the active command (boot load, apply) |
| POST | `/admin/queue/cancel-pending` | bearer | Remove a waiting command |
| GET | `/admin/config` | walled | Running config as JSON, secrets as `"***"` |
| PUT | `/admin/config` | walled | Validate and stage the whole config document as `gateway.toml.next` |
| GET | `/admin/config-pending` | walled | Staged config plus the profile selection |
| GET | `/admin/config-dirty` | walled | Pending files, changed sections, restart flag |
| POST | `/admin/config-apply` | walled | Promote staged files and reload remote routing |
| POST | `/admin/config-revert` | walled | Discard staged files |
| GET | `/admin/env` | walled | `gateway.env` with plaintext values |
| PUT | `/admin/env` | walled | Stage the whole env file |
| POST | `/shutdown` | walled | Graceful stop |
| POST | `/admin/restart` | walled | Exit for a supervisor to relaunch; 409 unless launched with `--supervised` |
| GET | `/admin/system` | walled | CPU, RAM, cache disk, first NVIDIA GPU |
| GET | `/admin/cloud-models` | walled | Cached cloud provider model sheet |
| POST | `/admin/cloud-models/refresh` | walled | Re-download the sheet |
| GET | `/admin/hf/search` | walled | Hugging Face search proxy with an allowlisted query |
| GET | `/admin/hf/model/{owner}/{name}` | walled | Hub model detail with file blobs |
| GET | `/admin/hf/model/{owner}/{name}/readme` | walled | Model card markdown, 1 MiB cap |
| POST | `/admin/reveal` | walled | Open the OS file manager at a path inside the cache |
| GET | `/admin/chat-templates` | walled (local) | Template families, repo mappings, each model's template decision |
| GET | `/admin/orphans` | walled (local) | Cache files no config entry references |
| GET | `/admin/model-info?path=` | walled (local) | GGUF header of a cached file |
| GET | `/config` | loopback (config-ui) | Redirect to `/config/` |
| GET | `/auth?key=` | loopback (config-ui) | Exchange the key for a session cookie |
| GET | `/config/`, `/config/app.js`, `/config/app.css`, two icon files | loopback (config-ui) | Settings page assets (5 routes) |

Gateway routes with no in-scope Studio counterpart: `/v1/rerank` (Studio returns 404 on purpose), the four speech routes (Studio has its own, outside this comparison), `/v1/tools/web_search` (an agent tool in Studio), profiles and profile switching, the staged change set (`config-pending`, `config-dirty`, `config-apply`, `config-revert`), queue cancellation, shutdown and restart, GGUF header inspection, and the config UI routes.

### 5.2 Studio's in-scope routes and their Gateway equivalents

#### Serving (24 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `POST /v1/chat/completions` | OpenAI chat to a local model, managed engine, or remote provider | Same: `POST /v1/chat/completions` |
| `POST /v1/completions` | OpenAI text completions | None (llama-server serves it; pass-through planned) |
| `POST /v1/embeddings` | OpenAI embeddings | Same: `POST /v1/embeddings` |
| `POST /v1/responses` | OpenAI Responses API | None (llama-server serves it; pass-through planned) |
| `POST /v1/messages` | Anthropic Messages | None (llama-server serves it; pass-through planned) |
| `POST /v1/messages/count_tokens` | Anthropic token count | None (pass-through planned) |
| `POST /v1/chat/count_tokens` | Token count for a chat payload | None (llama-server has `/v1/chat/completions/input_tokens`) |
| `GET /v1/models` | Loaded plus locally available models | Same: `GET /v1/models` |
| `GET /v1/models/{model_id}` | One model | None |
| `POST /v1/generate/stream` | Studio-native SSE chat | n/a (Studio-specific) |
| `POST /api/inference/cancel` | Cancel in-flight generations by id | None |
| `GET /api/inference/monitor` | Recent API activity | None |
| `DELETE /api/inference/monitor` | Clear API history | None |
| `GET /api/inference/monitor/{entry_id}` | One recorded request and reply | None |
| `GET /api/inference/loaded-models` | Loaded models only | Partial: `GET /admin/status` (models, local children) |
| `GET /props` (also `/v1/props`) | llama-server props | None |
| `GET /version` | Version string | None (the version is in the discovery file) |
| `GET /metrics` | llama-server metrics passthrough | None |
| `GET /slots` | llama-server slots passthrough | None |
| `POST /tokenize` | Tokenize passthrough | None (pass-through planned) |
| `POST /detokenize` | Detokenize passthrough | None |
| `GET /models` | llama.cpp router style model list | None |
| `POST /models/load` | Open WebUI style load | None |
| `POST /models/unload` | Open WebUI style unload | None |

#### Model lifecycle and runtime settings (25 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `POST /v1/load` | Load a model | None (handler exists in `commands.rs`, not exposed) |
| `POST /v1/unload` | Unload a model | None (same) |
| `POST /v1/validate` | Check a model resolves, fits, and can coexist | Partial: `PUT /admin/config` validates the whole document while staging |
| `GET /v1/status` | Active model and backend | Same: `GET /admin/status` |
| `GET /v1/load-progress` | Byte-level load progress | Partial: `GET /admin/progress` (busy flag and one text line) |
| `POST /v1/estimate-memory` | Price a load from model metadata | None |
| `GET /v1/llama-flags` | Flags the installed llama-server accepts | None |
| `POST /v1/mlx-drafters` | MLX drafters for a model | n/a (MLX) |
| `POST /v1/int8-prefill-availability` | MLX int8 prefill check | n/a (MLX) |
| `POST /v1/ane-prefill-availability` | MLX Neural Engine prefill check | n/a (MLX) |
| `POST /v1/fast-verify-availability` | MLX Fast Verify check | n/a (MLX) |
| `GET /api/inference/sampling-defaults` | Effective sampling for a model | None |
| `GET /api/inference/active-generations` | In-flight generations and decode capacity | None (status lists queue commands, not generations) |
| `POST /api/inference/transformers-upgrade-check` | Whether a model needs newer transformers | n/a (Python) |
| `POST /api/inference/install-latest-transformers` | Install a transformers overlay | n/a (Python) |
| `GET`, `PUT /api/settings/multi-model` | Multi-model mode (off, fit, always) | None (every profile model stays resident) |
| `GET`, `PUT /api/settings/model-memory` | Keep resident and mlock switches | None |
| `GET`, `PUT /api/settings/openai-auto-switch` | Auto-switch, idle unload, keep KV, auto-download | None |
| `GET`, `PUT /api/settings/vram-budget` | Fraction of VRAM loads may use | Partial: local `[[dominion]]` `vram_gb` via `PUT /admin/config` |
| `GET`, `PUT /api/settings/first-token-timeout` | First-token timeout | None (timeouts are compile-time constants) |

#### Discovery and hub proxying (25 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/models/list` | Suggested plus loaded models | None |
| `GET /api/models/config/{model}` | A model's config and capabilities from cache or the Hub | Partial: `GET /admin/hf/model/{owner}/{name}` |
| `POST /api/models/remote-code-scan` | Scan a repo for `trust_remote_code` code | n/a (transformers) |
| `POST /api/models/discard-remote-code` | Purge a declined scan's download | n/a (transformers) |
| `GET /api/models/check-vision/{model}` | Whether a model is vision capable | None |
| `GET /api/models/check-embedding/{model}` | Whether a model is an embedding model | None |
| `GET /api/models/kv-cache-estimate` | KV cache size for a quant and context | None |
| `GET /api/hub/gguf-variants` (twin `/api/models/gguf-variants`) | GGUF quants with sizes | Partial: the config UI derives quants from `GET /admin/hf/model/{owner}/{name}` |
| `POST /api/hub/token/validate` | Validate a Hugging Face token | Partial: the config UI tests the token through `GET /admin/hf/search` |
| `GET`, `HEAD /api/hub/proxy/{path}` | General same-origin relay to the Hugging Face endpoint | Partial: `GET /admin/hf/search`, `/admin/hf/model/{owner}/{name}`, and `.../readme` (allowlisted) |
| `GET /api/hub/modelscope/api/{kinds}` | ModelScope search | None |
| `GET /api/hub/modelscope/api/{kinds}/{owner}/{name}` | ModelScope repo info | None |
| `GET /api/hub/modelscope/api/{kinds}/{owner}/{name}/revision/{revision}` | Repo info at a revision | None |
| `GET /api/hub/modelscope/api/{kinds}/{owner}/{name}/auth-check` | Gated access check | None |
| `GET /api/hub/modelscope/api/{kinds}/{owner}/{name}/tree/{revision}` | File tree | None |
| `POST /api/hub/modelscope/api/{kinds}/{owner}/{name}/paths-info/{revision}` | Paths info | None |
| `GET`, `HEAD /api/hub/modelscope/{owner}/{name}/resolve/{revision}/{path}` | Resolve a model file | None |
| `GET`, `HEAD /api/hub/modelscope/datasets/{owner}/{name}/resolve/{revision}/{path}` | Resolve a dataset file | None |
| `GET`, `HEAD /api/hub/modelscope/` | Reachability probe | None |
| `GET /api/hub/modelscope/datasets/{owner}/{name}` | Redirect to the dataset page | None |
| `GET /api/hub/modelscope/{owner}/{name}` | Redirect to the model page | None |
| `POST`, `PUT`, `PATCH`, `DELETE /api/hub/modelscope/{rest}` | Refuse writes with 403 | None |
| `GET /api/picker/chat-template/{model}` | Default chat template for a model | Partial: `GET /admin/chat-templates` |
| `POST /api/picker/validate-chat-template` | Validate a custom chat template | None |

#### Downloads (17 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `POST /api/hub/download` | Start a download job | Partial: `POST /v1/cache` (unused by the UI); otherwise stage a config entry and download at the next boot |
| `POST /api/hub/download/cancel` | Cancel a download | Partial: `POST /admin/queue/cancel` (cancels boot provisioning) |
| `GET /api/hub/download-status` | One job's status | Partial: `GET /admin/status` (queue and provisioning flags) |
| `GET /api/hub/active-downloads` | Running jobs | Partial: `GET /admin/status` queue |
| `GET /api/hub/transport-status` | Which transport a download would use and why | None (HTTP only) |
| `GET /api/hub/gguf-download-progress` (twin under `/api/models`) | GGUF byte progress | Partial: `GET /admin/progress` text; `POST /v1/cache` streams byte progress |
| `GET /api/hub/download-progress` (twin under `/api/models`) | Repo byte progress | Partial: same |
| `GET /api/studio/download-transport-capabilities` | Xet and HTTP capabilities | None |
| `GET`, `PUT /api/settings/download-transport` | Transport mode | None |
| `GET`, `PUT /api/settings/download-speed-limit` | Download speed cap | None |
| `POST /api/settings/xet-notice/reserve` | Consume an Xet notice | n/a |
| `GET`, `PUT /api/settings/helper-precache` | Pre-download Studio's helper model | n/a (Studio-specific) |

#### Local library and files (35 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/hub/local` (twin `/api/models/local`) | On-device models from every known store | Partial: `GET /v1/cache` (tracked blobs) and configured entries in `GET /admin/config` |
| `GET`, `POST /api/hub/scan-folders` and `DELETE /api/hub/scan-folders/{folder_id}` (three twins under `/api/models`) | Custom scan folders | None |
| `GET /api/models/recommended-folders` | Known model folders that hold weights (LM Studio, Ollama) | None |
| `GET /api/models/browse-folders` | Server-side folder browser | None |
| `GET /api/hub/models-folder` | Models folder location | Partial: `[local].cache_dir` in `GET /admin/config` |
| `GET /api/hub/cached-gguf` and `GET /api/hub/cached-models` (twins under `/api/models`) | Cached repos in Hugging Face and other caches | Partial: `GET /v1/cache` |
| `GET /api/hub/hidden-models` | Hidden-model matchers | None |
| `POST /api/hub/delete-impact` | Preview what a delete removes | None |
| `GET /api/hub/orphan-companions` | Unused companion files | Partial: `GET /admin/orphans` (lists them; no route deletes them) |
| `POST /api/hub/local/reveal` | Reveal a model outside the Hugging Face cache | Partial: `POST /admin/reveal` (cache paths only) |
| `DELETE /api/hub/local/delete` | Delete a model outside the Hugging Face cache | None |
| `DELETE /api/hub/delete-cached` (twin under `/api/models`) | Delete a cached repo or one variant | Same: `DELETE /v1/cache/{sha256}` |
| `GET /api/models/cached-model-path` | Path of a cached model | Partial: `GET /v1/cache` returns paths |
| `POST /api/models/reveal-cached-model` | Reveal a cached model | Same: `POST /admin/reveal` |
| `GET`, `PUT /api/settings/hugging-face-cache` | Hugging Face cache location | Partial: `[local].cache_dir` via `GET` and `PUT /admin/config` (Gateway keeps its own cache) |
| `GET /api/settings/caches` | Cache inventory with sizes | Partial: `GET /v1/cache` sizes and `GET /admin/system` disk |
| `GET /api/settings/caches/{key}/breakdown` | One cache's size breakdown | None |
| `POST /api/settings/caches/purge` | Purge selected caches | None |
| `GET /api/library` | Unified library with disk usage | Partial: `GET /v1/cache` |
| `POST /api/library/items/delete` | Delete a library item | Same: `DELETE /v1/cache/{sha256}` |
| `POST /api/library/items/reveal` | Reveal a library item | Same: `POST /admin/reveal` |
| `GET /api/library/locations` | Storage roots | Partial: `[local].cache_dir` in `GET /admin/config` |
| `POST /api/library/locations/reveal` | Reveal a storage root | Same: `POST /admin/reveal` |
| `POST /api/library/locations/move` | Move a storage root and its files | None (changing `cache_dir` does not move files) |

#### Per-model configuration (10 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET`, `PUT /api/settings/openai-auto-switch/overrides` | Per-model load overrides | Partial: per-model knobs in `[[local_model]]` via `GET` and `PUT /admin/config` |
| `GET`, `PUT /api/settings/model-sampling-defaults` | Per-model sampling defaults | None (sampling presets follow the thinking mode) |
| `GET`, `PUT /api/settings/model-aliases` | Public model aliases | Partial: `[[model]]` `name` and `upstream` via `PUT /admin/config` |
| `GET`, `PUT /api/settings/pinned-models` | Pinned and connected models for the picker | None |
| `GET`, `PUT /api/settings/last-local-model` | Last loaded model, for restore | n/a (models are fixed per profile) |

#### LoRA and fine-tunes (5 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/models/loras` | LoRA adapters and exported models | None |
| `GET /api/models/loras/{lora_path}/base-model` | Base model of an adapter | None |
| `GET /api/models/checkpoints` | Training checkpoints as model sources | n/a (training outputs) |
| `GET /api/hub/unsloth-models` | Unsloth fine-tunes and exports | n/a (training outputs) |
| `DELETE /api/models/delete-finetuned` | Delete a fine-tune or one exported quant | n/a (training outputs) |

#### Remote providers (20 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/providers/public-key` | RSA key for browser-side key encryption | n/a (keys stay on the machine; see `PUT /admin/env`) |
| `GET /api/providers/registry` | Supported provider types and defaults | Partial: `GET /admin/cloud-models` (23 providers) |
| `GET /api/providers/pricing` | Price table | Same: `GET /admin/cloud-models` (prices per model) |
| `GET /api/providers/` | Saved providers | Partial: `[[endpoint]]` entries in `GET /admin/config` |
| `POST /api/providers/` | Create a provider | Partial: `PUT /admin/config` (whole document) plus `PUT /admin/env` for the key |
| `PUT /api/providers/{provider_id}` | Update a provider | Partial: same |
| `PUT /api/providers/{provider_id}/api-key/migrate` | Re-wrap a stored key | n/a |
| `DELETE /api/providers/{provider_id}` | Delete a provider | Partial: `PUT /admin/config` without the entry |
| `POST /api/providers/test` | Test a connection | None |
| `GET /api/providers/model-catalog` | Cached remote model catalog | Same: `GET /admin/cloud-models` and `POST /admin/cloud-models/refresh` |
| `POST /api/providers/model-capabilities` | Capabilities of a provider's models | Partial: `GET /admin/cloud-models` |
| `POST /api/providers/models` | Live model list from a provider | None |
| `POST /api/providers/{id}/oauth/start`, `GET /api/providers/{id}/oauth/flows/{flow_id}`, `POST /api/providers/{id}/oauth/flows/{flow_id}/complete`, `DELETE /api/providers/{id}/oauth/flows/{flow_id}`, `GET /api/providers/{id}/codex/models`, `DELETE /api/providers/{id}/oauth` | Codex subscription OAuth login (6 routes) | None |
| `GET`, `PUT /api/settings/managed-provider-urls` | Private or LAN provider URLs for managed accounts | n/a (single user) |

#### llama.cpp runtime and managed engines (20 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/llama/update-status` | Installed vs latest llama.cpp | None (llama.cpp is pinned per Gateway release) |
| `POST /api/llama/update` | Install or update llama.cpp | None |
| `GET /api/llama/update-changelog` | Release notes between versions | None |
| `GET`, `POST /api/llama/backend` | Current and available llama.cpp builds; switch build | Partial: `[local].llama_backend` via `PUT /admin/config` plus restart |
| `GET`, `PUT /api/settings/llama-cpp-path` | Custom llama.cpp path | Partial: `[local].llama_server_path` via `PUT /admin/config` plus restart |
| `GET /api/engines`, `POST /api/engines/{engine}/install`, `POST /api/engines/{engine}/cancel`, `DELETE /api/engines/{engine}`, `POST /api/engines/{engine}/rollback`, `DELETE /api/engines/wsl/environment` | Managed vLLM and SGLang engines and the Windows WSL distro (6 routes) | None |
| `GET /api/npu/status`, `POST /api/npu/enable`, `GET /api/npu/models`, `GET /api/npu/downloads`, `POST` and `GET /api/npu/models/{model_id}/download`, `DELETE /api/npu/models/{model_id}` | AMD NPU runtime and models (7 routes) | None |

#### Hardware and system (7 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/system` | Platform, GPU, memory | Same: `GET /admin/system` |
| `GET /api/system/disk` | Free space where downloads land | Same: `GET /admin/system` (cache drive) |
| `GET /api/system/gpu-visibility` | GPUs visible to the backend | Partial: `GET /admin/system` (first NVIDIA GPU only) |
| `GET /api/system/hardware` | GPU name and total VRAM | Same: `GET /admin/system` |
| `GET /api/train/hardware` | Live GPU utilization | Partial: `GET /admin/system` (VRAM used, no GPU utilization) |
| `GET /api/train/hardware/visible` | Live utilization of visible GPUs | Partial: same |
| `POST /api/settings/igpu-carveout-notice/dismiss` | Dismiss integrated-GPU memory advice | n/a |

#### Auth (16 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET /api/auth/identity` | Prove this is the real local server | Partial: the discovery file plus a `/health` and bearer probe (`gateway-api-discovery`) |
| `GET /api/auth/status` | First-boot auth state | n/a (one bearer key) |
| `POST /api/auth/login`, `POST /api/auth/logout`, `POST /api/auth/refresh`, `POST /api/auth/desktop-initial-password`, `POST /api/auth/change-password` | Password sessions (5 routes) | n/a (one bearer key) |
| `POST /api/auth/desktop-login` | Exchange the desktop secret for tokens | Partial: `GET /auth?key=` cookie handoff |
| `POST`, `GET /api/auth/api-keys` and `DELETE /api/auth/api-keys/{key_id}` | Create, list, and revoke API keys | None (one shared key in `[server].api_key`) |
| `GET /api/liveness` | Liveness | Same: `GET /health` |
| `GET /api/health` | Liveness plus capability bits | Same: `GET /health` (no capability bits) |
| `GET`, `PUT /api/settings/keyless-api-access` | Keyless access scope | Partial: `[server].trust_loopback` via `PUT /admin/config` plus restart |
| `PUT /api/accounts/{account_id}/model-access` | Per-account model allowlist | n/a (single user) |

#### Other model settings (14 routes)

| Studio route | Purpose | Gateway |
|---|---|---|
| `GET`, `PUT`, `DELETE /api/settings/hugging-face-token` | Hugging Face token | Partial: `HF_TOKEN` in `gateway.env` via `GET` and `PUT /admin/env` plus restart |
| `PUT /api/settings/hugging-face-token/migrate` | Re-wrap the stored token | n/a |
| `GET`, `PUT /api/settings/hub` | Custom Hugging Face endpoint | None |
| `PUT /api/settings/hub/source` | Switch the hub to ModelScope | None |
| `POST /api/settings/hub/source-notice` | Claim a one-time notice | n/a |
| `GET`, `PUT`, `DELETE /api/settings/embedding-model` | Default embedding model | Partial: an embedding `[[local_model]]` via `PUT /admin/config` |
| `GET /api/settings/embedding-model/resolve` | Embedding backend and download plan | None |
| `POST /api/settings/embedding-model/unload` | Unload the embedding model | None |
| `PUT /api/settings/embedding-model/device` | Embedding model device | None |

## 6. Configuring Gateway through routes, and API docs for harness models

The goal: every part of Gateway can be read and changed through HTTP routes, and those routes are documented well enough that a model in the Runvil harness can carry out instructions like "add Qwen3 8B at Q4 and make it the default chat model", "add my OpenAI key", or "delete the models I have not used".

### 6.1 What works today

- **The bones are right.** Configuration already goes through routes with a staged change set:
  - `PUT /admin/config` validates the document and stages it as `gateway.toml.next`.
  - `GET /admin/config-dirty` and `GET /admin/config-pending` show what is staged.
  - `POST /admin/config-apply` promotes it; `POST /admin/config-revert` discards it.
  - Secrets come back as `"***"`, and sending `"***"` keeps the stored value.

  This stage, review, apply flow is the plan-then-apply pattern that vendor guidance recommends for agents.
- **Every config file section is reachable** through that one route: `[server]`, `[local]`, `[[dominion]]`, `[[endpoint]]`, `[[model]]`, `[[local_model]]`, `[[stt_model]]`, `[[profile]]`, `[tools.web_search]`, and `[stt]`. The profile selection has its own route, and secrets go through `PUT /admin/env`.

### 6.2 What stands in the way

**API shape**

- **Whole-document writes only.** An agent must read the full config, edit it, and send it all back; anything it omits is deleted. There is no per-resource route and no PATCH.
- **No revision check.** The UI and an agent can overwrite each other's edits; the last writer wins.
- **No validate-only mode and no structured diff.** `config-dirty` names changed sections, not fields.
- **Unstructured errors.** Validation failures come back as one flattened string, and only the first failure is reported.
- **Lossy round trip.** A `${VAR}` in a non-secret field comes back resolved and is written back as a literal, and the first save drops every comment and the hand ordering.
- **Plaintext secrets.** `GET /admin/env` returns every value in plaintext.

**Coverage gaps**

- **Not reachable by any route:**
  - CLI flags: config path, `--profile`, tray, supervision, preferred port.
  - `RUST_LOG`, the log location, and log retention.
  - The cloud sheet URL, which is set only through the environment.
  - Compile-time limits such as timeouts and size caps.
- **Restart.** Boot-owned settings need a restart: `[server]`, `[[profile]]`, `[[local_model]]`, `[[stt_model]]`, `[stt]`, `gateway.env`, and the profile selection. But `POST /admin/restart` only works under a hidden `--supervised` launch that nothing performs. Workshop restarts its sidecar by shutting it down and relaunching it.
- **Misreported live changes.** `[local]` and local `[[dominion]]` edits are reported as applied live but only take effect after a restart.
- **Orphaned cache files** can be listed but not deleted.
- **Runtime actions** from sections 3 and 4 (load, unload, download now, per-model logs) have no routes.

**Auth**

- **Keyless admin access.** `trust_loopback`, on by default, lets any keyless loopback caller use the whole admin surface, including plaintext secrets and process control.
- **One shared key.** There are no read-only or scoped tokens, and no record of who changed what.
- **LAN reach.** `switch-profile` and the queue routes accept LAN callers who present the bearer key.

**Docs**

- **No machine-readable description.** There is no OpenAPI document and no JSON Schema for the config. Admin request and response types are private to each handler, config bodies are untyped JSON, and the route registry is internal and not served.
- **Best field reference today:** `gateway.local.example.toml`, which a test keeps valid. `crates/gateway/README.md` already states the intent: a spec generated from source, with CI comparing the two.
- **The user docs have drifted** in several places: how apply behaves for boot-owned sections, a `[workshop]` section the loader now rejects, status fields that do not exist, and the auth tier of `/admin/chat-templates`.

**Harness side**

- **Harness models get tools only from Packs.** Workshop installs three:
  - `std`: only the read-only file tools work.
  - `web`: fetch refuses loopback, plain http, and ports other than 80 and 443, and only sends GET.
  - `mcp`: lists and calls tools on configured MCP servers.

  There is no shell tool yet.
- **So no harness model can reach Gateway's admin routes today.** The Workshop server can, but its config-panel proxy forwards all of `/admin/` except `GET /admin/progress`. That includes `GET /admin/env` and `POST /admin/reveal`, which is too broad to hand a model.
- **No approval step for tool calls.** Workshop's Ask mode covers file writes only.

### 6.3 Prior art

- **Studio's MCP server** has 23 hand-written FastMCP tools at `/mcp`, off until the owner enables them, authenticated with one API key.
  - Eight are about models: `studio_status`, `list_models`, `load_model`, `unload_model`, `plan_load`, `download_model`, `get_job`, `cancel`.
  - Safety comes from read-only and destructive hints, per-tool switches, caller profiles that hide tools, and preview or validate-only modes such as `plan_load`.
  - Descriptions run 900 to 1,400 characters, with examples, side effects, and timeouts. Errors start with a tag such as `[gpu_busy retry_after=12]`, followed by a hint.
  - No tool writes persistent settings or configures providers. It is AGPL, so it is a pattern to learn from, not code to reuse.
- **[LiteLLM's LiteAdmin MCP](https://github.com/BerriAI/liteadmin-mcp/tree/main)** administers a gateway through 65 reviewed operations taken from the gateway's own OpenAPI, with a read-only switch, a tool allowlist, and a rule to keep provider keys out of chat.
- **[Cloudflare's Code Mode](https://blog.cloudflare.com/code-mode-mcp/)** covers about 2,500 endpoints with two tools: search the spec, and run code.
- **Vendor guidance on tool sets:**
  - Keep the active set small: under 20 tools ([OpenAI](https://developers.openai.com/api/docs/guides/function-calling)), 10 to 20 (Google), and Anthropic says selection degrades past 30 to 50.
  - Write descriptions of at least 3 to 4 sentences, including when not to use the tool. In Anthropic's testing, example inputs raised accuracy on complex parameters from 72% to 90% ([advanced tool use](https://www.anthropic.com/research/advanced-tool-use)).
  - Auto-generating MCP tools from OpenAPI is discouraged for production, including by the FastMCP author.
- **MCP itself:**
  - [Spec 2026-07-28](https://modelcontextprotocol.io/specification/2026-07-28/changelog) is stateless and lets a server ask for confirmation by returning an input-required result.
  - Tool annotations (`readOnlyHint`, `destructiveHint`) are untrusted hints, but real clients act on them.
  - The official Rust SDK, [rmcp](https://github.com/modelcontextprotocol/rust-sdk/) 3.9.0, mounts on axum with `nest_service`. It checks the Host header by default and leaves Origin checks off until configured.
- **Patterns for safe changes:**
  - Validate-only on every write ([AIP-163](https://google.aip.dev/163), Kubernetes server-side dry run).
  - Apply guarded by a revision (ETag and `If-Match`).
  - RFC 9457 problem details with a pointer to each failing field.
  - Long-running operations the client can poll ([AIP-151](https://google.aip.dev/151)).
  - Write-only secrets.

  The Idempotency-Key header is still an IETF draft.
- **Rust tooling:**
  - utoipa 6 with utoipa-axum 0.3 supports axum 0.8 and OpenAPI 3.1. It uses doc comments as descriptions and supports write-only fields and `x-` extensions.
  - aide's stable release pins an older schemars than rmcp uses.
  - A committed spec snapshot plus [oasdiff](https://www.oasdiff.com/docs/github-action) in CI catches breaking changes.
  - For offline human docs, only `utoipa-swagger-ui` vendors its assets; Scalar and Redoc load from CDNs unless self-hosted.
- **llms.txt** is rarely fetched unless an agent is pointed at it: 97% of files got no requests in May 2026 ([Ahrefs](https://ahrefs.com/blog/llmstxt-study/)). Task recipes reach models more reliably through MCP resources and the tool descriptions themselves.

### 6.4 Proposed design

Three layers, built in this order.

**Layer 1: a complete, resource-oriented admin API.** This becomes the source of truth, and the config UI and agents both use it.

- **Per-resource routes.** Every collection (`endpoints`, `models`, `local-models`, `stt-models`, `profiles`, `dominions`) and every singleton section (`server`, `local`, `stt`, `tools/web-search`) gets list, get, create or replace by id, JSON Merge Patch, and delete. Keep the whole-document `GET` and `PUT /admin/config` for backup, import, and export.
- **Writes only stage.** Every write stages into the existing change set instead of applying, so stage, review, apply stays the safety net. `?dry_run=true` on every write validates and returns the resulting diff without staging anything.
- **A change-set resource.**
  - `GET /admin/changes` returns a field-level diff: before and after values, secrets redacted, and a restart flag per change.
  - Every read returns the config revision as an ETag, and every write and apply requires `If-Match`, so the UI and an agent cannot silently overwrite each other.
  - Revert stays as it is.
- **Write-only secrets.** Secrets can be set and cleared but never read back. Plaintext `GET /admin/env` is either dropped or kept for the local UI only.
- **Structured errors.** RFC 9457 problem details that list every failure, each with a JSON pointer to the field and a hint on how to fix it.
- **Close the coverage gaps:**
  - A restart that works without `--supervised`, or Workshop launching Gateway supervised.
  - Orphan deletion.
  - Download and provision jobs with ids and progress.
  - Runtime load and unload, and per-model log tails.
  - The cloud sheet URL and log level as config.
  - A capability route that reports build features and versions.
- **Fix what is broken:** the restart misreport for `[local]` and local dominions, and the `${VAR}` round-trip loss. Keep comments by writing with a format-preserving TOML editor.
- **Enforce coverage with tests.** Every config field in the schema must be writable through some route, and the config UI may call only routes that appear in the published spec.

**Layer 2: machine-readable docs generated from code.**

- **Config JSON Schema.** Derive `JsonSchema` on the `gateway-config` types (schemars is already a workspace dependency). Publish the schema at a route and as a committed file, using the same bless-and-CI pattern as `sheet.schema.json`.
- **OpenAPI 3.1 document.** Move the admin request and response types into `gateway-api-types` and generate the document from the handlers. Serve it at `/openapi.json`, commit it, and diff it in CI with oasdiff. `crates/gateway/README.md` already says it wants exactly this.
- **Operations written for model readers:**
  - a snake_case `operationId`;
  - three to five sentences, including when not to use the operation;
  - an example request;
  - extensions that mark restart-required and destructive operations.
- **Human docs.** Serve a docs page from the config UI with vendored assets, and publish the same reference in runvil-docs.

**Layer 3: an agent layer for harness models.**

- **A small, task-shaped tool set** of about 10 to 15 tools, hand-written on top of Layer 1 rather than one tool per route. For example:
  - `gateway_status`, `list_models`, `search_hub_models`
  - `plan_add_local_model`, which returns quant options with fit plus the staged diff
  - `add_remote_model`, which uses the cloud catalog
  - `set_secret`, `update_model`, `remove_model`, `set_profile`
  - `review_changes`, `apply_changes`, `revert_changes`, `job_status`
- **Stage-only writes.** Every mutating tool only stages. `apply_changes` is separate, marked destructive, and requires the revision the user reviewed. It asks the user to confirm, either through an MCP input-required result or a harness approval step.
- **Two ways in:**
  - A Gateway `/mcp` endpoint (rmcp on axum, loopback only, with Host and Origin checks) for any MCP client, including the harness's existing `mcp` Pack.
  - A Workshop `gateway` Pack that keeps the key inside the Workshop server and uses a tight route allowlist. It also avoids the port-change and reconnect problems the `mcp` Pack has when Gateway restarts.
- **Task recipes** such as "add a local model", "add a cloud provider", "free disk space", and "switch profile", shipped as MCP resources and inside the tool descriptions, plus an `llms.txt` for coding agents.
- **Scoped access.** Add scoped, revocable tokens (read-only, configure, admin), limit keyless loopback trust to read-only routes, and keep an audit log that records whether the UI or an agent made each change.

### 6.5 Effort

| Piece | Effort guess |
|---|---|
| Layer 1: per-resource routes with merge patch, dry run, change-set diff, revisions, problem details, write-only secrets | 3 to 5 weeks |
| Layer 1: coverage gaps (restart, orphan deletion, jobs, capabilities, misreport and round-trip fixes) | 2 to 4 weeks, partly shared with section 4 |
| Layer 2: config JSON Schema, OpenAPI generation, CI snapshot, docs page | 2 to 3 weeks |
| Layer 3: tool set, MCP endpoint, Workshop Pack, recipes, scoped tokens, confirmed apply | 3 to 5 weeks |

Total: about 2.5 to 4 months for one developer (low-medium confidence). Layer 1 also makes the section 4 UI work cheaper, because the downloads panel, runtime control, and provider tests become thin clients of documented routes.

## 7. Child servers already speak the protocols

| Server | Anthropic `/v1/messages` + count_tokens | OpenAI `/v1/responses` | Notes |
|---|---|---|---|
| llama-server b10082 | yes | yes | Verified from the tag's `server.cpp`; also `/v1/completions`, `/tokenize`, router-mode `/models/load` |
| vLLM | yes | yes | Has a Claude Code integration guide |
| mistral.rs 0.10 | yes | yes, with stored responses | Anthropic route ignores `x-api-key` |

This is why pass-through is cheap: Gateway rewrites the model name, enforces its own auth, and relays the stream, the same pattern it already uses for chat completions.

## 8. Safetensors serving

What cannot move to Rust is Studio's own transformers-based worker. That does not rule out safetensors: engines that serve safetensors can run as child processes, exactly like llama-server.

### 8.1 Is there a "Rust PyTorch"?

No. The closest projects:

| Project | What it is | Windows GPU | LLM state | Verdict for Gateway |
|---|---|---|---|---|
| [Candle](https://github.com/huggingface/candle) (Hugging Face) | PyTorch-like tensor library, CUDA and Metal | NVIDIA only | about 40 hand-ported decoder families | Foundation of mistral.rs, TEI, and others; not used directly |
| [Burn](https://github.com/tracel-ai/burn) (Tracel) | Full training framework, CubeCL kernels | Yes, Vulkan and DX12 any vendor, plus CUDA | No released LLM engine; one Apple test decoded 8.5x slower than MLX | Watch; kernels JIT-compile at runtime |
| [tch-rs](https://github.com/LaurentMazare/tch-rs) | Bindings to libtorch (real PyTorch C++) | CUDA | No serving stack | Windows CUDA libtorch is 4.9 GB; not realistic |
| [ort](https://github.com/pykeio/ort) | ONNX Runtime bindings | DirectML (maintenance mode) and vendor EPs | Needs ORT GenAI plus a Python export | Best option for embeddings, rerank, audio |

The building blocks are already Rust: the `safetensors` library itself (now under the PyTorch Foundation), `tokenizers`, `hf-hub`, and `minijinja` for chat templates. The missing piece is always per-architecture model code. Config-driven generic decoders exist for Llama-shaped models but break on new blocks (MLA, hybrid linear attention, new MoE routers).

### 8.2 Rust-native engines

**mistral.rs** ([repo](https://github.com/EricLBuehler/mistral.rs), MIT) is the only broad option:

- About 60 architecture entries; safetensors, GGUF, GPTQ, AWQ, FP8, NVFP4, in-situ quantization.
- Best LoRA support found anywhere: multiple PEFT adapters per base model, per-request selection, hot load and unload through vLLM-compatible routes, content-hashed adapter versions. Rejects adapters with `modules_to_save`, added tokens, or changed embeddings or output head.
- OpenAI chat, completions, embeddings, Responses, Anthropic Messages.
- Problems, verified directly:
  - No prebuilt Windows CUDA binary; v0.9.5's only Windows download is a CPU zip. The Windows installer does a source build that needs nvcc and Visual Studio.
  - The paged-attention CUDA kernels are compiled only under `target_family = "unix"` in `mistralrs-paged-attn/build.rs`, so native Windows loses the fastest decode path.
  - No authentication at all, and CORS allows any origin.
- It can be embedded as an axum 0.8 `Router` (`mistralrs-server-core`), so a host can add its own auth. The crates.io release lags GitHub by about seven months, so this means a pinned git dependency.
- One developer holds about 88% of commits.

Others:

- **[Paddock](https://github.com/truespar/paddock)** is the only Rust engine that ships Windows x64 NVIDIA binaries with `.sha256` files (verified for v0.1.20). Two months old, GGUF-centric, single GPU, rejects LoRA ("merge the adapter into the GGUF for now"). Loopback callers skip its API key.
- **[TEI](https://github.com/huggingface/text-embeddings-inference)** covers embeddings and rerank with a real API key on every inference route; no Windows binaries.
- **TGI** was archived on 2026-03-21. vllm.rs was renamed **xinfer** (no Windows build, no LoRA). candle-vllm, Crane, Kalosm, Ratchet, shimmy, lm.rs, and rustformers/llm are Linux-only, narrow, stale, or not servers.

### 8.3 vLLM and SGLang as managed children

Studio already runs both as optional child servers (about 5.4k lines). What it takes:

- **Platform.** Both are Linux-only natively. vLLM on Apple Silicon runs through the `vllm-metal` plugin on MLX. On Windows, Studio imports its own private WSL2 distro from a sha256-checked Ubuntu image, installs a hash-pinned uv environment inside, and reaches the engine over localhost forwarding. That landed in October 2026, is labeled experimental, and needs one admin prompt.
- **Size.** vLLM 0.33 is a 336 MB wheel plus about 3 GB of PyTorch; the Docker image is 9.7 GB.
- **Runtime compilation.** torch.compile with Triton needs a C compiler at runtime and FlashInfer JIT needs nvcc. `--enforce-eager` reduces this but does not remove it and slows decode. This conflicts with Gateway's "nothing native compiled at runtime" rule.
- **GPU memory.** `--gpu-memory-utilization` is a fraction of total memory reserved up front, so a vLLM child cannot share a GPU with llama-server children without explicit budgets.
- **Startup.** About 2 minutes cold is normal (134 s for Qwen3-8B on an L40S in one report); a first compile can go silent for tens of minutes. Studio times out on silence, not a fixed limit.
- **Per-family flags.** `--tool-call-parser` and `--reasoning-parser` must be chosen per model family, and names differ between vLLM and SGLang.
- **Auth gap.** vLLM's `--api-key` covers only some route prefixes; `/invocations` serves inference with no key. A PR to cover all routes ([#59892](https://github.com/vllm-project/vllm/pull/59892)) was closed unmerged. Bind to loopback and proxy only an allow-list.
- **SGLang security.** Six CERT-listed CVEs including an unauthenticated remote code execution in LoRA loading ([VU#281278](https://www.kb.cert.org/vuls/id/281278)); patch status unverified.
- **Upside.** vLLM loads LoRA adapters at runtime, the natural way to serve many fine-tunes.

### 8.4 Compiled runtimes (export once with Python, serve without it)

- **[ONNX Runtime GenAI](https://github.com/microsoft/onnxruntime-genai)** is the strongest option on Windows across GPU vendors: continuous batching and paged attention on CUDA and WebGPU, multi-LoRA via converted adapters. Export needs Python (model builder, Olive); no official Rust bindings; DirectML is in maintenance mode.
- **[OpenVINO GenAI](https://openvinotoolkit.github.io/openvino.genai/)** loads raw PEFT LoRA safetensors at runtime and has official Rust crates, but only accelerates Intel GPUs.
- **TensorRT-LLM** dropped Windows. MLC-LLM, AOTInductor, and ExecuTorch are poor fits.

### 8.5 Converting to GGUF instead

- `convert_hf_to_gguf.py` and `convert_lora_to_gguf.py` need Python with CPU-only torch, once, at export time. Unsloth already exports GGUF.
- llama-server loads LoRA adapters at startup only; scales can change globally or per request. Requests with different LoRA settings are not batched together.
- The LoRA converter fails on adapters that include full embedding or output-layer weights, which some Unsloth recipes produce.
- No maintained non-Python converter exists. Ollama v0.34.1 dropped its own: "GGUF model creation now requires using llama.cpp tooling for safetensor conversion and quantization" ([release](https://github.com/ollama/ollama/releases/tag/v0.34.1)).

## 9. Existing practice

- **Desktop apps use llama.cpp everywhere plus MLX on Mac.** No mainstream app runs vLLM natively on Windows. Ollama serves safetensors only through its MLX engine.
- **Unsloth Studio is the only product automating vLLM on Windows** (the private WSL2 distro above). Its follow-up fixes are a ready-made list of edge cases: GPU numbering differs between Windows and WSL, `wsl.exe` re-parses commands, Windows port-range exhaustion, ROCm-on-WSL memory caps.
- **[llama-swap](https://github.com/mostlygeek/llama-swap)** (Go, MIT) is the closest match to Gateway's supervisor: per-model start commands, health checks that extend while loading progresses, idle unload, eviction weights that treat vLLM as expensive to restart, and a wrapper that uses vLLM sleep mode.
- **[llmman](https://github.com/ericcurtin/inferrs)** already does the design under consideration: pinned llama.cpp for GGUF, vLLM, SGLang, or mlx-lm for safetensors, one backend process per model, idle unload after five minutes.
- **Rust gateways only route.** TensorZero was wound down in June 2026; Helicone's gateway is in maintenance mode. None supervise engines.
- **Big engines converged on one split:** a Rust front end handles HTTP, chat templates, tokenization, and tool-call and reasoning parsing, while the Python engine only exchanges token IDs. vLLM's Rust frontend, SGLang's SMG gateway, and NVIDIA Dynamo all work this way. The supported third-party surface into vLLM is the `vllm-proto` gRPC crate, not its internal ZeroMQ protocol.
- **Nobody embeds Python inside a Rust server**, because some CUDA errors can only be cleared by killing the process.
- **Reusable parsers exist.** SMG publishes `tool-parser` and `reasoning-parser` (Apache-2.0, 1.10.1 on crates.io, updated 2026-11-03), and vLLM depends on them. Dynamo publishes `dynamo-parsers`, used by SGLang. These are a license-compatible source for Gateway's tool-call repair instead of Studio's AGPL code.
- **Recurring pain points:** orphaned engine processes (especially on Windows), VRAM conflicts between engines (no project measures live VRAM across engine types), slow vLLM cold starts, and loopback engine traffic accidentally sent through a system proxy. Gateway already calls `.no_proxy()` on its local-child clients.

## 10. Recommendations

In order:

1. **Add pass-through routes** for `/v1/messages`, `/v1/messages/count_tokens`, `/v1/responses`, `/v1/completions`, and `/tokenize` to llama-server children, plus `x-api-key` auth (high confidence: llama-server b10082 already implements them, and this unlocks Claude Code and Codex on local models).
2. **Serve Unsloth fine-tunes as GGUF through the existing llama-server path**, with LoRA settings in `[[local_model]]`, a LoRA section in the Local page, and conversion at export time (high confidence: no new engine, and it works on NVIDIA, AMD, and Intel GPUs on Windows).
3. **Make the admin API complete and resource-oriented** (section 6, layer 1): per-resource routes with merge patch, dry run on every write, a field-level change-set diff, revisions with `If-Match`, write-only secrets, RFC 9457 errors, and routes for everything that is CLI-only or missing today, with a test that every config field is writable by route (high confidence: this is the prerequisite for agent configuration, and the config UI's new features get cheaper on top of it).
4. **Generate the config JSON Schema and an OpenAPI document from code** and check both in CI (section 6, layer 2) (high confidence: schemars is already in the workspace, and `crates/gateway/README.md` already states this intent).
5. **Finish Gateway's two existing UI plans** as thin clients of the new routes: download during Apply with a live progress panel (bytes, speed, ETA, cancel), as the August plan describes, and the Restart to Apply button from the October plan (high confidence: both are already planned, the `provision_model` handler exists, and it fixes the most visible UI gap without bringing back the bugs the August plan removed).
6. **Ship the agent layer** (section 6, layer 3): about 10 to 15 task-shaped tools over the new routes, stage-only writes with a separately confirmed apply, exposed as a Gateway `/mcp` endpoint and a Workshop `gateway` Pack, plus scoped tokens and an audit log (medium confidence: the harness has no tool-call approval step yet, and the confirmation design is open).
7. **Use SMG's `tool-parser` and `reasoning-parser` crates for tool-call repair** behind a Gateway-owned trait, pinned to exact versions (medium confidence: vLLM depends on them, but their fit for repairing llama-server output after the fact is untested).
8. **Add runtime control:** expose load and unload, then a loaded-models panel with memory use, eject, and per-model log tails (medium confidence: relaxes the fixed-at-boot design, which is a product decision).
9. **Improve Discover and the local library:** a KV-aware fit estimate from GGUF metadata, automatic mmproj and drafter companions, split GGUF support, partial-file cleanup, and a file picker for local GGUFs (medium confidence: the fit estimate needs more GGUF fields than Gateway reads today).
10. **For direct safetensors serving, build a small Gateway-owned wrapper around mistral.rs** that embeds `mistralrs-server-core`'s router at a pinned git revision and adds Gateway's bearer key and CORS lock. Build it in CI per CUDA target for Windows and Linux plus Metal for macOS, and pin it like llama.cpp (medium confidence: open Windows build issues, a bus factor of one, and no paged attention on native Windows).
11. **Offer vLLM and SGLang as a Linux-only opt-in**, run from a container pinned by digest, with an explicit exception to the runtime-compile rule (medium confidence: depends on whether that exception is acceptable). Defer the WSL2 route until users ask.
12. **Treat automatic fitting at load time as a separate later project**, built on llama-server's `--fit` rather than copying Studio's heuristics (medium confidence: depends on how well `--fit` handles real user hardware).
13. **Watch** Paddock (if it adds LoRA), Tracel's Burn engine (if it ships with Vulkan), and ONNX Runtime GenAI (if safetensors on AMD and Intel Windows GPUs becomes a requirement).

Do not attempt:

- A full port of Studio's backend (high confidence: years of work, the ML core stays Python, and upstream moves too fast).
- Translating Studio's frontend code. Rebuild the model management features it shows inside Gateway's config UI instead, and leave chat to Workshop (high confidence: the code is AGPL, about 100k lines for model management alone, and Gateway's config UI already has the structure).
- Embedding Python or vLLM inside the Gateway process (high confidence: no precedent, and CUDA failures would take down Gateway).

## 11. Effort summary

| Phase | Contents | Effort guess |
|---|---|---|
| 1 | Pass-through routes, GGUF LoRA with its UI, reasoning controls | 3 to 5 weeks |
| 2 | Complete admin API (per-resource routes, merge patch, dry run, change-set diff, revisions, problem details, write-only secrets, coverage gaps), config JSON Schema and OpenAPI with CI snapshots | 7 to 12 weeks |
| 3 | UI on the new routes: download during Apply with live progress, Restart to Apply, provider test connection and live model listing, Discover staging fixes | 3 to 5 weeks |
| 4 | Agent layer: 10 to 15 tools, Gateway `/mcp` endpoint, Workshop `gateway` Pack, task recipes, scoped tokens, confirmed apply | 3 to 5 weeks |
| 5 | Tool-call repair via SMG crates, native Anthropic and Gemini upstreams | 4 to 8 weeks |
| 6 | Runtime load and unload with a loaded-models panel, per-model logs, idle unload | 4 to 8 weeks |
| 7 | Fit estimate, companions, split GGUF, local library and discovery improvements, broader hardware detection and its panel | 2 to 3 months |
| 8 | mistral.rs wrapper child with CI builds for Windows, Linux, macOS | 4 to 6 weeks |
| 9 | vLLM and SGLang Linux opt-in | 3 to 5 weeks |
| Later | Automatic fitting at load | 1 to 2 months for a basic version |
| Optional | WSL2 route for vLLM on Windows | another 3 to 5 weeks |

One developer, low-medium confidence. Phases 1 to 4 (about 4 to 6 months) give a Gateway that serves Claude Code and Codex on local models, has a real downloads panel, and can be configured end to end by a harness model from user instructions.

## 12. Licensing

- Unsloth Studio (`studio/`, including all serving code, the frontend, and the Tauri shell) is AGPLv3. Only the core `unsloth/` library is Apache 2.0.
- Runvil Gateway is BSL-1.0, copyright The C++ Alliance, Inc.
- Translating Studio's code into Gateway would likely make the result a derivative work under AGPL. Use Studio as a behavior reference (flags, formats, fallbacks) and write from public sources: llama.cpp docs, the Anthropic and OpenAI API specs, the models' chat templates, and Apache-2.0 or MIT crates.
- Every framework and engine recommended here (mistral.rs, Candle, SMG crates, Dynamo crates, ort, minijinja, tokenizers) is MIT or Apache-2.0.
- This is not legal advice; check with whoever handles licensing at The C++ Alliance before anyone works from Studio's source.

## 13. Open questions

- Whether per-resource writes should always stage into the change set, or apply immediately for sections that are already live-safe (remote models and endpoints).
- Whether to generate OpenAPI with utoipa (its own schema model) or aide (schemars-based, but its stable release pins an older schemars than rmcp), given that the config JSON Schema and MCP tool schemas would come from schemars.
- How confirmation for `apply_changes` should work in the harness, which has no tool-call approval step today.
- Whether keyless loopback trust should remain for read-only routes once scoped tokens exist.
- Whether downloading during Apply can stage files atomically enough to keep the August plan's no-orphans-on-revert guarantee when a user reverts mid-download.
- Whether adopting models from the HF cache, Ollama, or LM Studio folders should hash each file on adoption (slow for large models) or record them as unpinned local paths.
- Throughput and stability of a native Windows CUDA mistral.rs build without paged attention, compared with llama-server on the same GPU.
- Whether mistral.rs accepts Unsloth adapter folders that include tokenizer files.
- Whether SMG's parser crates can be driven on completed or streamed llama-server output.
- Whether the SGLang CVEs are patched in current releases.
- Release date and Windows backends of Tracel's Burn engine.
- Whether ONNX Runtime GenAI's batching engine supports per-request LoRA.

## Supporting research

All produced 2026-11-09:

- Unsloth Studio codebase survey for a Rust axum Tauri port
- Unsloth Studio model serving vs Runvil Gateway feature gap
- Rust ML and tensor frameworks 2026 (is there a Rust PyTorch)
- Rust-native LLM inference servers for safetensors, GGUF and LoRA
- Engine manager practice (apps, model managers, and gateways that supervise engines)
- Packaging Python inference engines for desktop apps, and the convert-to-GGUF alternative
- Rust front ends and Python engine cores (vLLM Rust frontend, SMG, Dynamo, TGI, PyO3)
- Compiled LLM runtimes without Python at serve time (ORT GenAI, OpenVINO GenAI, TensorRT, MLC, AOTInductor, ExecuTorch)
- Unsloth Studio backend HTTP route inventory for Runvil Gateway scope
- Runvil Gateway HTTP route inventory, auth tiers, and configuration coverage via admin routes
- Runvil harness tool surface and Unsloth Studio MCP prior art for configuring Gateway
- Agent-configurable APIs and LLM-consumable API documentation practice (OpenAPI, MCP, safe-mutation patterns, Rust tooling)

*2026-11-09 11:20 - Claude Opus 5.5*
