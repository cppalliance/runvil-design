<!-- sources: github.com/unslothai/unsloth at c4052051f488334b2a59d7488b69b70f02375163 (2026-11-09); runvil-gateway at 9369a7d902a4b083e68f59a0353bcb9227c6c0d3 (2026-10-31, v0.3.0); runvil Workshop; llama.cpp tag b10082; web research 2026-11-09 -->

# Porting Unsloth Studio's model serving to Rust: assessment and plan

## Bottom line

- **A full port of Unsloth Studio to Rust is not a realistic project.** Studio's ML core (training, diffusion, MLX, export) is built on torch, transformers, diffusers, and Unsloth's own kernels, so it stays in Python no matter what. Replacing just the FastAPI server with axum means rewriting roughly 336k lines of non-ML Python, much of it business logic, against a codebase that took 517 commits in `studio/` in one week. Several engineer-years, and a fork would never catch up.
- **The Tauri part already exists.** Unsloth Desktop is a Tauri 2.12 app. On the Runvil side, Workshop is a Tauri 2.11 app that already ships Gateway as a pinned sidecar and embeds Gateway's config page.
- **Scoped to what overlaps with Gateway (serving local and remote models, plus the UI to download, configure, and manage them and their files), this stops being a port.** Gateway is already the Rust + axum version of that slice. What remains is a list of features Studio has and Gateway lacks.
- **The model management UI is the largest gap by volume.** Studio spends about 99k to 117k frontend lines on discovering, downloading, configuring, and managing models; Gateway's config UI is about 12k. Gateway is ahead on config review (diff, apply, revert), pinning, and cloud pricing, but has no download progress, no runtime control, no LoRA, and a shallow local library. Two of Gateway's own design choices (models fixed at boot, downloads deferred to the next boot) block the biggest UI gaps.
- **The cheapest high-value API gap turned out to be much cheaper than first estimated.** The llama-server build Gateway pins (b10082) already serves Anthropic `/v1/messages`, `count_tokens`, OpenAI `/v1/responses`, legacy `/v1/completions`, and `/tokenize`. Gateway can pass these through to local children instead of translating them: about 1 to 2 weeks, and it lets Claude Code and Codex use local models through Gateway.
- **There is no "Rust PyTorch" that can load arbitrary Hugging Face safetensors models.** A safetensors file holds tensors but no compute graph; the architecture lives in Python modeling code. Every non-Python engine hand-ports architectures. mistral.rs is the only broad Rust engine, with excellent LoRA support but weak Windows packaging and no auth.
- **Recommended path:** add pass-through routes; serve Unsloth fine-tunes as GGUF through llama-server; finish Gateway's two existing UI plans (download during Apply, Restart to Apply) and build a real downloads panel on top; then runtime control, tool-call repair via Apache-2.0 parser crates from SMG, a Gateway-built mistral.rs wrapper for direct safetensors serving, and vLLM and SGLang as a Linux-only opt-in.
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
| Tool-call repair for local models, using SMG's parser crates (section 7) | about 10k in Studio | 2 to 4 weeks |
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

## 5. Child servers already speak the protocols

| Server | Anthropic `/v1/messages` + count_tokens | OpenAI `/v1/responses` | Notes |
|---|---|---|---|
| llama-server b10082 | yes | yes | Verified from the tag's `server.cpp`; also `/v1/completions`, `/tokenize`, router-mode `/models/load` |
| vLLM | yes | yes | Has a Claude Code integration guide |
| mistral.rs 0.10 | yes | yes, with stored responses | Anthropic route ignores `x-api-key` |

This is why pass-through is cheap: Gateway rewrites the model name, enforces its own auth, and relays the stream, the same pattern it already uses for chat completions.

## 6. Safetensors serving

What cannot move to Rust is Studio's own transformers-based worker. That does not rule out safetensors: engines that serve safetensors can run as child processes, exactly like llama-server.

### 6.1 Is there a "Rust PyTorch"?

No. The closest projects:

| Project | What it is | Windows GPU | LLM state | Verdict for Gateway |
|---|---|---|---|---|
| [Candle](https://github.com/huggingface/candle) (Hugging Face) | PyTorch-like tensor library, CUDA and Metal | NVIDIA only | about 40 hand-ported decoder families | Foundation of mistral.rs, TEI, and others; not used directly |
| [Burn](https://github.com/tracel-ai/burn) (Tracel) | Full training framework, CubeCL kernels | Yes, Vulkan and DX12 any vendor, plus CUDA | No released LLM engine; one Apple test decoded 8.5x slower than MLX | Watch; kernels JIT-compile at runtime |
| [tch-rs](https://github.com/LaurentMazare/tch-rs) | Bindings to libtorch (real PyTorch C++) | CUDA | No serving stack | Windows CUDA libtorch is 4.9 GB; not realistic |
| [ort](https://github.com/pykeio/ort) | ONNX Runtime bindings | DirectML (maintenance mode) and vendor EPs | Needs ORT GenAI plus a Python export | Best option for embeddings, rerank, audio |

The building blocks are already Rust: the `safetensors` library itself (now under the PyTorch Foundation), `tokenizers`, `hf-hub`, and `minijinja` for chat templates. The missing piece is always per-architecture model code. Config-driven generic decoders exist for Llama-shaped models but break on new blocks (MLA, hybrid linear attention, new MoE routers).

### 6.2 Rust-native engines

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

### 6.3 vLLM and SGLang as managed children

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

### 6.4 Compiled runtimes (export once with Python, serve without it)

- **[ONNX Runtime GenAI](https://github.com/microsoft/onnxruntime-genai)** is the strongest option on Windows across GPU vendors: continuous batching and paged attention on CUDA and WebGPU, multi-LoRA via converted adapters. Export needs Python (model builder, Olive); no official Rust bindings; DirectML is in maintenance mode.
- **[OpenVINO GenAI](https://openvinotoolkit.github.io/openvino.genai/)** loads raw PEFT LoRA safetensors at runtime and has official Rust crates, but only accelerates Intel GPUs.
- **TensorRT-LLM** dropped Windows. MLC-LLM, AOTInductor, and ExecuTorch are poor fits.

### 6.5 Converting to GGUF instead

- `convert_hf_to_gguf.py` and `convert_lora_to_gguf.py` need Python with CPU-only torch, once, at export time. Unsloth already exports GGUF.
- llama-server loads LoRA adapters at startup only; scales can change globally or per request. Requests with different LoRA settings are not batched together.
- The LoRA converter fails on adapters that include full embedding or output-layer weights, which some Unsloth recipes produce.
- No maintained non-Python converter exists. Ollama v0.34.1 dropped its own: "GGUF model creation now requires using llama.cpp tooling for safetensor conversion and quantization" ([release](https://github.com/ollama/ollama/releases/tag/v0.34.1)).

## 7. Existing practice

- **Desktop apps use llama.cpp everywhere plus MLX on Mac.** No mainstream app runs vLLM natively on Windows. Ollama serves safetensors only through its MLX engine.
- **Unsloth Studio is the only product automating vLLM on Windows** (the private WSL2 distro above). Its follow-up fixes are a ready-made list of edge cases: GPU numbering differs between Windows and WSL, `wsl.exe` re-parses commands, Windows port-range exhaustion, ROCm-on-WSL memory caps.
- **[llama-swap](https://github.com/mostlygeek/llama-swap)** (Go, MIT) is the closest match to Gateway's supervisor: per-model start commands, health checks that extend while loading progresses, idle unload, eviction weights that treat vLLM as expensive to restart, and a wrapper that uses vLLM sleep mode.
- **[llmman](https://github.com/ericcurtin/inferrs)** already does the design under consideration: pinned llama.cpp for GGUF, vLLM, SGLang, or mlx-lm for safetensors, one backend process per model, idle unload after five minutes.
- **Rust gateways only route.** TensorZero was wound down in June 2026; Helicone's gateway is in maintenance mode. None supervise engines.
- **Big engines converged on one split:** a Rust front end handles HTTP, chat templates, tokenization, and tool-call and reasoning parsing, while the Python engine only exchanges token IDs. vLLM's Rust frontend, SGLang's SMG gateway, and NVIDIA Dynamo all work this way. The supported third-party surface into vLLM is the `vllm-proto` gRPC crate, not its internal ZeroMQ protocol.
- **Nobody embeds Python inside a Rust server**, because some CUDA errors can only be cleared by killing the process.
- **Reusable parsers exist.** SMG publishes `tool-parser` and `reasoning-parser` (Apache-2.0, 1.10.1 on crates.io, updated 2026-11-03), and vLLM depends on them. Dynamo publishes `dynamo-parsers`, used by SGLang. These are a license-compatible source for Gateway's tool-call repair instead of Studio's AGPL code.
- **Recurring pain points:** orphaned engine processes (especially on Windows), VRAM conflicts between engines (no project measures live VRAM across engine types), slow vLLM cold starts, and loopback engine traffic accidentally sent through a system proxy. Gateway already calls `.no_proxy()` on its local-child clients.

## 8. Recommendations

In order:

1. **Add pass-through routes** for `/v1/messages`, `/v1/messages/count_tokens`, `/v1/responses`, `/v1/completions`, and `/tokenize` to llama-server children, plus `x-api-key` auth (high confidence: llama-server b10082 already implements them, and this unlocks Claude Code and Codex on local models).
2. **Serve Unsloth fine-tunes as GGUF through the existing llama-server path**, with LoRA settings in `[[local_model]]`, a LoRA section in the Local page, and conversion at export time (high confidence: no new engine, and it works on NVIDIA, AMD, and Intel GPUs on Windows).
3. **Finish Gateway's two existing UI plans:** download during Apply with a live progress panel (bytes, speed, ETA, cancel), as the August plan describes, and the Restart to Apply button from the October plan (high confidence: both are already planned, the `provision_model` handler exists, and it fixes the most visible UI gap without bringing back the bugs the August plan removed).
4. **Use SMG's `tool-parser` and `reasoning-parser` crates for tool-call repair** behind a Gateway-owned trait, pinned to exact versions (medium confidence: vLLM depends on them, but their fit for repairing llama-server output after the fact is untested).
5. **Add runtime control:** expose load and unload, then a loaded-models panel with memory use, eject, and per-model log tails (medium confidence: relaxes the fixed-at-boot design, which is a product decision).
6. **Improve Discover and the local library:** a KV-aware fit estimate from GGUF metadata, automatic mmproj and drafter companions, split GGUF support, partial-file cleanup, and a file picker for local GGUFs (medium confidence: the fit estimate needs more GGUF fields than Gateway reads today).
7. **For direct safetensors serving, build a small Gateway-owned wrapper around mistral.rs** that embeds `mistralrs-server-core`'s router at a pinned git revision and adds Gateway's bearer key and CORS lock. Build it in CI per CUDA target for Windows and Linux plus Metal for macOS, and pin it like llama.cpp (medium confidence: open Windows build issues, a bus factor of one, and no paged attention on native Windows).
8. **Offer vLLM and SGLang as a Linux-only opt-in**, run from a container pinned by digest, with an explicit exception to the runtime-compile rule (medium confidence: depends on whether that exception is acceptable). Defer the WSL2 route until users ask.
9. **Treat automatic fitting at load time as a separate later project**, built on llama-server's `--fit` rather than copying Studio's heuristics (medium confidence: depends on how well `--fit` handles real user hardware).
10. **Watch** Paddock (if it adds LoRA), Tracel's Burn engine (if it ships with Vulkan), and ONNX Runtime GenAI (if safetensors on AMD and Intel Windows GPUs becomes a requirement).

Do not attempt:

- A full port of Studio's backend (high confidence: years of work, the ML core stays Python, and upstream moves too fast).
- Translating Studio's frontend code. Rebuild the model management features it shows inside Gateway's config UI instead, and leave chat to Workshop (high confidence: the code is AGPL, about 100k lines for model management alone, and Gateway's config UI already has the structure).
- Embedding Python or vLLM inside the Gateway process (high confidence: no precedent, and CUDA failures would take down Gateway).

## 9. Effort summary

| Phase | Contents | Effort guess |
|---|---|---|
| 1 | Pass-through routes, GGUF LoRA with its UI, reasoning controls | 3 to 5 weeks |
| 2 | Download during Apply with live progress, Restart to Apply, provider test connection and live model listing, Discover staging fixes | 4 to 6 weeks |
| 3 | Tool-call repair via SMG crates, native Anthropic and Gemini upstreams | 4 to 8 weeks |
| 4 | Runtime load and unload with a loaded-models panel, per-model logs, idle unload | 4 to 8 weeks |
| 5 | Fit estimate, companions, split GGUF, local library and discovery improvements, broader hardware detection and its panel | 2 to 3 months |
| 6 | mistral.rs wrapper child with CI builds for Windows, Linux, macOS | 4 to 6 weeks |
| 7 | vLLM and SGLang Linux opt-in | 3 to 5 weeks |
| Later | Automatic fitting at load | 1 to 2 months for a basic version |
| Optional | WSL2 route for vLLM on Windows | another 3 to 5 weeks |

One developer, low-medium confidence. Phases 1 to 3 (about 3 to 5 months) cover most of what users see in Studio's serving and model management.

## 10. Licensing

- Unsloth Studio (`studio/`, including all serving code, the frontend, and the Tauri shell) is AGPLv3. Only the core `unsloth/` library is Apache 2.0.
- Runvil Gateway is BSL-1.0, copyright The C++ Alliance, Inc.
- Translating Studio's code into Gateway would likely make the result a derivative work under AGPL. Use Studio as a behavior reference (flags, formats, fallbacks) and write from public sources: llama.cpp docs, the Anthropic and OpenAI API specs, the models' chat templates, and Apache-2.0 or MIT crates.
- Every framework and engine recommended here (mistral.rs, Candle, SMG crates, Dynamo crates, ort, minijinja, tokenizers) is MIT or Apache-2.0.
- This is not legal advice; check with whoever handles licensing at The C++ Alliance before anyone works from Studio's source.

## 11. Open questions

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

*2026-11-09 10:50 - Claude Opus 5.5*
