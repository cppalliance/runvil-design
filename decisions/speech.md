# Speech: design decisions and their reasons

- Scope: speech to text and speech synthesis in the Gateway, and dictation in Workshop
- Code: `runvil-gateway` at commit `9369a7d`, `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Paths: Gateway code paths start at `crates/gateway/` in `runvil-gateway`, and Workshop code paths at `crates/workshop/` in `runvil`

## Summary

The Gateway turns speech into text with whisper.cpp. A realtime client streams audio over the WebSocket `/v1/realtime` and receives text while the person speaks, and a batch client uploads a file to `POST /v1/audio/transcriptions`. Two Whisper models share each recording: a fast one decodes every half second and shows its words, and an accurate one decodes each finished segment and replaces them. A third model, Silero, decides which audio is speech. `POST /v1/audio/speech` forwards a synthesis request to a remote provider and relays the audio. Workshop's voice feature is dictation: its server relays microphone audio to the Gateway, and the text appears in the chat box. Sections 1 to 8 describe these parts.

A check of this page against the code found seven gaps and left six questions about reasons for the owner. Section 10 lists the gaps, and section 11 lists the questions. All seven gaps are in guide chapter 05, and the largest is speech detection: the chapter describes a loudness threshold, and the code classifies every frame with Silero.

Section 9 lists the planned work. Its largest item is a local engine for speech synthesis: a remote provider produces all audio, and the Gateway refuses a local model of the speech kind.

## From a microphone to text in the chat box

The operator presses the round button of an empty chat box. The agent view calls `press()` on the `SttHandle` that `setupStt` returned, and the handle asks the `SpeechCaptureService` for the microphone. An `AudioWorkletNode` named `pcm16-capture` emits 100 ms chunks of 24 kHz PCM, and the `TakeRegistry` records a take, one recording, at the cursor.

`RealtimeTranscriptionService` sends each chunk as Base64 in an `input_audio_buffer.append` event to `/v1/realtime` on the Workshop server. The server's relay adds the Gateway's bearer key and copies each frame, unparsed, to the Gateway's `/v1/realtime?intent=transcription`.

In the Gateway, the connection's `Session` decodes the Base64 into an `AudioBuffer`, and `Resampler24To16` converts it to 16 kHz. The recording's `Take` appends the samples, and its `Segmenter` passes each 512-sample frame to a `SileroDetector`. Every 500 ms the session sends the open segment's audio to the interim worker, and `WholeWindowState` sorts the words into agreed and tentative text (section 5). When the `Segmenter` closes a segment, `FinalPipeline` sends it to the final worker, and `TakeState` appends the result to the finalized text.

After each change the session sends a `conversation.item.input_audio_transcription.hypothesis` event. The relay copies it back, `decodeRealtimeEvent` checks its shape, the `TakeRegistry` computes the changed words, and `setupStt` writes them into the chat box through `replaceRange`. A second press stops the capture, and `RealtimeTranscriptionService` sends `input_audio_buffer.commit`. The Gateway decodes the audio that is still open and sends `conversation.item.input_audio_transcription.completed` with the whole transcript, which `setupStt` writes as one undo step.

## 1. Where speech runs

### The Gateway transcribes speech, and Workshop is one of its clients

The Gateway loads the speech models, decodes audio, and answers both transcription routes. Workshop's server relays one socket to it (section 8).

The Gateway loads the models because it budgets VRAM (section 2). A model that Workshop loaded itself would allocate VRAM that the Gateway is unaware of.

### Four crates under `stt/` depend on each other in one direction

`gateway-stt`, in the folder `api/`, has the routes, the sessions, and the state of each recording. `gateway-stt-engine` defines the decoder and detector traits and runs the worker threads. `gateway-stt-backend-whisper` implements the traits with Whisper and Silero. `gateway-whisper-ffi` loads the whisper.cpp library and contains the subsystem's unsafe code. `api` depends on `engine` and `backend-whisper`, and `backend-whisper` depends on `engine` and `whisper-ffi`.

The rest of the Gateway calls speech through one struct, `SpeechService`. Only the speech crates select a worker or build a wire event. The FFI and the safe backend are separate crates because one crate for both would enlarge the code that may use `unsafe` and would obscure ownership.

### Each loaded model has one worker thread, and a `Take` owns the state of one recording

`SttEngine` starts one thread for each loaded model, `stt-interim` and `stt-final`, and every client shares them. A decode job is a `DecodeRequest` with its own samples, terms, and history text, and a worker drops all of it when it replies. The code calls one recording a take. A `Take` in `gateway-stt` owns its audio, its finalized text, its agreement state, and its failure.

### The Gateway downloads whisper.cpp as a prebuilt library and loads it at run time

A GitHub workflow builds whisper.cpp at the pinned tag `b4938` as a shared library for each platform. At speech load the Gateway downloads the archive for the machine and checks its digest, and `WhisperLibrary::load` opens the library. A const block in the tests of `gateway-whisper-ffi` asserts the offset of every C struct member that the crate reads or writes, so a layout change fails the test build.

The library is prebuilt because a compile of whisper.cpp inside `cargo build` requires cmake and a C++ toolchain. The Gateway calls the library inside its own process: whisper.cpp processes audio buffers synchronously, and a wire protocol to a whisper server in a child process is more than such a library needs.

## 2. Speech models and loading

### A speech model is an `[[stt_model]]` entry, and the active profile selects it

An `[[stt_model]]` entry has a `name`, a `role` of `interim` or `final`, a `source`, an optional `sha256` pin, a `vram_gb` estimate, and an optional `dominion`. The Gateway loads the speech models that the active profile lists, and speech is off when that list is empty. Validation accepts at most one model of each role in a profile and refuses a final model that lacks an interim partner.

An entry that names a dominion counts against that dominion's `vram_gb` budget, which is the reason that the Gateway loads the models (section 1). A profile has at most one model of each role because the engine runs one worker for each role. [The Gateway](gateway.md) covers profiles and dominions.

### A fast interim model and an accurate final model share each recording

While a person speaks, the interim model decodes the recent audio every `interval_ms`, 500 by default, over at most `window_seconds`, 15 by default. The final model decodes each closed segment again, and its text replaces the interim text. The built-in pair is `whisper-base-en` and `whisper-small-en`.

The owner's goal: "Low latency is important but it is more important to get the transcription right, early on, and so the user can see the words forming and correcting".

### The Gateway loads speech once per process, after it binds its listener

The Gateway binds its listener first. Its queued boot command then calls `SpeechService::load_initial` once, which downloads the models and starts the workers. Each decoder runs one warm-up decode at load and discards the result. Until the load finishes, `/v1/realtime` answers 503. When it fails, speech is unavailable until a restart, and the Gateway continues to answer its other routes. A later profile switch saves the new speech selection, and the next start loads it.

A switch leaves the running engine in place. Its replacement at run time would add five mechanisms to the transcription path: publication of the new engine, quiescence of the running one, rollback, reconstruction, and cancellation.

### Every speech load downloads and opens the pinned Silero model

`SILERO_VAD_MODEL` in the config crate names the Silero model's URL, digest, size, and license. The speech load downloads the file, checks the digest, and opens one detector as a test. The model is a constant in code, outside `[[stt_model]]`, because `SttRole` has only the roles `interim` and `final`.

The speech load fails when any of the three steps fails, because a fallback to loudness detection would be worse in two ways. In a test on noise clips, dictation with loudness detection invented 84 lines of text, and dictation with Silero invented zero. A fallback would also degrade speech silently, and silent degradation can hide a bug.

## 3. The realtime route

### Live transcription uses a strict subset of OpenAI's Realtime protocol

The route is the WebSocket `/v1/realtime?intent=transcription`. A client sends four JSON events: `session.update`, `input_audio_buffer.append`, `input_audio_buffer.commit`, and `input_audio_buffer.clear`. Audio is Base64 text of mono 16-bit PCM at 24 kHz, and the Gateway converts it once to 16 kHz. The converter, `Resampler24To16`, is a low-pass polyphase filter, and the audio runs 1 ms behind its timeline. The Gateway answers an unknown field or an option outside the subset, such as a language or turn detection, with an `error` event whose code names the fault.

The subset exists for standard clients of OpenAI's protocol. One optional extension, the hypothesis snapshot, exists for Runvil's live transcript in three parts (section 5).

### A commit ends the turn, and the Gateway finalizes the item in the background

The Gateway refuses turn detection, so the client ends a turn with `input_audio_buffer.commit`. The first append of a buffer creates the item id and copies the session's prompt, so a later `session.update` applies to the next buffer. At a commit the Gateway answers `input_audio_buffer.committed` and `conversation.item.created` at once and finishes the transcript as a background task.

The buffer copies the prompt so that a prompt change leaves the open buffer and every committed item as they are.

### A client receives hypothesis snapshots on request, and append-only deltas otherwise

A client that puts `item.input_audio_transcription.hypothesis` in the `include` list of `session.update` receives `conversation.item.input_audio_transcription.hypothesis` events. Each one replaces the last: it has a `revision` number, the whole `transcript`, and its `finalized`, `agreed`, and `tentative` parts. A client with an empty list receives standard `conversation.item.input_audio_transcription.delta` events, built from the finalized text plus the agreed text less its last two words.

The extension's two names are part of version one of the public wire format, and canonical fixtures pin them. The two-word margin exists because agreed text only grows, so append-only deltas are safe except near its end, where a rewrite by the final pass can still change words.

### Realtime clients name `realtime-transcribe`, and batch clients name a configured model

`GET /v1/models` lists each loaded speech model under its configured name with the kind `transcription`. When both an interim and a final model are loaded, the list also has the logical name `realtime-transcribe`. The batch route accepts a configured name, and it accepts `realtime-transcribe` and decodes with the final model. With only an interim model, the realtime route answers 503.

The batch route accepts the logical name because the model list advertises it.

### Each queue has a fixed bound, and a take of any length stores at most 30 seconds of audio

The Gateway admits 8 realtime sessions, a session has room for 4 committed items, and each worker queue for 8 jobs. One append decodes to at most 15 MiB, and a commit needs 100 ms of audio. A take stores at most 30 seconds of audio at one time. A recording of any length is one item: the take drops audio once it is finalized and stores the text. An append that still exceeds the 30 seconds after the take has released its waiting ranges (section 5) fails with `too_much_unfinalized_audio`.

Each bound is a constant in the code, and it becomes a setting only when measurements justify one.

### The Gateway answers a client mistake with an `error` event and closes a session that it ends with code 1011

A malformed event, bad audio, or a commit under 100 ms returns one `error` event that repeats the client's `event_id`, and the session continues. A committed item that fails produces one `conversation.item.input_audio_transcription.failed` event.

When the Gateway itself ends a session, `end_session` sends an `error` of type `server_error` that names the cause, logs one line, and closes the socket with code 1011. The event and the close code exist because a log line by itself leaves the client unable to tell why its session ended. The Gateway excludes transcript text, audio, and prompts from every log field.

## 4. Speech detection and segments

### Silero classifies every 32 ms frame of a take as speech or silence

The engine crate defines the trait `SpeechDetector`, which classifies one frame of 512 samples. `SileroDetector` implements it with whisper.cpp's streaming detector: speech starts at a probability of 0.5 and ends below 0.35. Each take opens its own detector and passes it every frame once, in order, because Silero's state depends on the frames before.

Silero detects speech because Whisper invents text over pauses: in one measurement, segmentation by Silero before the decode cut Whisper's hallucinations from 21.3 to 0.2 percent. A loudness rule is a second check before a final decode: the final pass skips a range whose RMS is below 0.001.

### A segment closes after 2 seconds of silence, or 0.2 seconds after a finished sentence

`endpoint` closes a segment after 2 seconds of silence, and the segment becomes a job for the final pass. It closes the segment after 0.2 seconds when the latest interim text ends in `.`, `?`, or `!`, or when the speech lasted under 1 second. The segmenter discards speech shorter than 250 ms as a click.

The owner's rule: "if you pause for two seconds, it crystallizes, and then you can keep talking". The shorter values follow a second rule: each tunable has the lowest-latency value that passes its checks for stability and punctuation.

### The segmenter cuts nonstop speech every 10 seconds, and each forced window repeats 8 seconds

When speech continues for 10 seconds, the segmenter forces a cut at 10.016 seconds, or where speech resumed after the longest pause in the last 1.5 seconds. The next forced window starts 8 seconds earlier, so a final decode covers about 18 seconds. The take aligns the two texts over the overlap, and when they fail to align it estimates the cut from the audio positions. While a forced window is pending, the next final decode receives an empty history.

The sizes follow from the take's limit of 30 seconds: a forced decode is at most 18 seconds long, which leaves at least 12 seconds of capacity for the next 10-second stride.

### An interim window ends 100 ms after the last speech, and a silent range skips the final decode

An interim window starts at the open segment, covers at most `window_seconds`, and ends 100 ms after the last frame that Silero classified as speech. The session skips a tick when that end equals the end of the last decoded window. It also skips a tick while an interim decode is running, and it starts the skipped decode as soon as the running one ends. The final pass skips a range in which Silero classified every frame as silence (section 5).

The window ends at the speech because Whisper invents text over a trailing pause. The window still includes 100 ms of silence. The owner's reason: "we need some silence or else how do we get trailing sentence punctuation."

## 5. How text is committed and corrected

### A word is tentative, then agreed, then finalized

Tentative text is the latest guess of the interim pass, and each tick may change it. Agreed text is interim text that successive passes confirmed. Finalized text is the output of the final pass, and it grows in audio order. The Gateway decides each transition, and a client draws what it receives.

The owner's rule: "once you go past a certain size, we shouldn't be rewriting the beginning".

### A word becomes agreed when two interim passes half a second apart produce it

The take compares words by their letters and digits in lower case. A word is agreed when it appears at the same position in at least 2 interim results whose windows end at least 0.5 seconds apart.

The comparison ignores punctuation, spacing, and case because flips of those made up 45.9 percent of the instability of partial text in one measured recognizer. The gap is 0.5 seconds because a gap of 0.6 seconds needs three ticks, which costs latency.

### The final pass replaces the text of its range, and up to 5 later interim words remain

When a final decode returns for a range of audio, `rewrite_natural` appends its text to the finalized text, and that text replaces what the interim pass showed for the range. Words that the interim pass showed after the final's last word remain on screen, at most 5 of them, when the 2 words before them match the end of the final text.

The final text replaces shown text because the accurate model exists to refine what the user sees. A rule that froze text once shown would discard the final model's corrections of that text.

### Accepted interim text becomes final for audio that the final pass skipped or released

The final pass skips a range that is shorter than half a second or silent, and the interim text accepted for that audio becomes the finalized text. A take queues at most 4 closed segments for the final pass, and later ones wait with their audio. When the audio of the waiting ones would exceed the 30-second limit, the take releases the oldest range and finalizes its interim text.

The first rule exists so that a correct word on screen survives a stop. The second exists so that a take survives a full queue for the final pass.

### The interim and final passes decode with separate profiles

`RoleProfile` in the whisper backend sets the Whisper parameters for a role. The interim profile decodes one segment, sizes the encoder context to the window, and limits output to 4 tokens per second. The final profile uses Whisper's defaults. After an interim decode, `guard_interim` drops a result that is likely silence and low in confidence, and it collapses a phrase repeated 4 or more times.

The settings of the interim profile lower the cost of the decode at each tick.

### The operator's vocabulary and the client's prompt bias both passes on both routes

The operator lists domain terms under `[stt] vocabulary`. A client adds terms, separated by commas, in a session's `transcription.prompt` or in the batch form's `prompt` field. The Gateway joins the operator's terms and then the client's into a prompt that begins `Glossary:`, and both passes receive it. The backend trims the glossary from the end, so the operator's terms survive a client prompt that is too long.

The prompt has the form of a glossary because, in measurements, that form outperforms a raw list of keywords.

## 6. The whisper build and the GPU

### `[stt] whisper_backend` selects the CPU or the CUDA build of whisper.cpp

The setting has three values: `auto`, the default, `cpu`, and `cuda`. The Gateway reads it once, at speech load, on Windows x86-64 and Linux x86-64, the two platforms with both builds. Under every setting the Gateway checks the CPU before it downloads a build, and an x86-64 CPU that lacks one of the builds' six instruction-set extensions fails the speech load with `LocalError::UnsupportedCpu`.

The CUDA build exists for speed: on Linux, a final pass takes 2.14 s with the CPU build and 0.13 s with the same sources built for CUDA. The CPU build exists because a Windows machine that lacks an NVIDIA driver fails to load the CUDA build.

### `auto` selects the CUDA build when the machine meets its requirements, and an explicit `cuda` skips the checks

Under `auto` the Gateway runs one `nvidia-smi` query and reads `CUDA_VISIBLE_DEVICES`. It selects the CUDA build when the query reports an NVIDIA GPU, the variable leaves a GPU visible, and the machine meets the requirements in that build's row of `WHISPER_ASSETS`:

1. Linux: driver 570 or later, and a `libstdc++.so.6` that defines `GLIBCXX_3.4.30`.
2. Windows: driver 580 or later, and every GPU at compute capability 8.6, 8.9, 12.0, or 12.1.

Any other result selects the CPU build, a failed query included. An explicit `cuda` skips these checks, because a refusal would block GPUs that the build supports natively and that run on drivers 525 to 569. Guide chapter 05 lists the cases in which that build ends the Gateway.

The checks exist because the selection is the only guard against a failure that ends the process: whisper runs inside the Gateway's process, and ggml aborts on a CUDA error.

### A graceful stop aborts running decodes and retires speech before the process exits

At a graceful stop, `Gateway::serve` joins the command worker and then calls `SpeechService::shutdown`, both under one 5-second deadline. The shutdown sets a cancellation flag that every decode request shares. whisper.cpp polls the flag through its `abort_callback`, and the aborted request fails. The workers then free whisper's native state while the process is whole.

The order exists for the CUDA build, so that whisper frees its GPU memory while the CUDA runtime is alive. The abort exists because a decode that outlasts the deadline would otherwise leave the retirement unfinished, and the process would exit in the middle of the decode.

## 7. Batch transcription and speech synthesis

### `POST /v1/audio/transcriptions` transcribes one uploaded file in OpenAI's shape

The request is a multipart form with a `model` and a `file` of at most 25 MiB, in 16 kHz mono WAV. The route decodes the whole file with one request to the named model's worker and returns `json` or `verbose_json`. The workers decode English, and the route answers 400 to a `language` other than `en`. It answers 404 with the code `model_not_found` to an unknown model.

The route exists so that OpenAI client tools can transcribe a file.

### A synthesis model is a `[[model]]` entry with `kind = "speech"`, and a remote provider produces the audio

Speech synthesis uses the Gateway's ordinary model catalog: an entry under `[[model]]` with `kind = "speech"`, an endpoint, and an optional `voices` list. The Gateway refuses a `[[local_model]]` of the speech kind before any download. Workshop has yet to call the synthesis route, and section 9 lists the playback of replies as planned work.

Synthesis is a model kind because a speech call is a routable model call that a pipeline binds by name and voice, a remote provider is a backend like any other, and a client discovers the model through `GET /v1/models`. A separate table for synthesis models would lack the passthrough to remote providers, the routing for each model, and the listing on `GET /v1/models`. [The Gateway](gateway.md) covers the catalog and the queues.

### `POST /v1/audio/speech` checks the caller and the voice before the request waits in a queue

The request names a `model`, an `input` of at most 4096 characters, and a `voice`. The handler checks the bearer key before it parses the body. It then checks the voice against the model's `voices` list and answers 400 `invalid_voice` to a voice outside the list. The check precedes the queue so that only a request that passes the check takes a queue slot. An omitted `response_format` becomes `mp3`, since "OpenAI defaults to mp3 while Together defaults to wav". The Gateway forwards the `input` text and every field it leaves unnamed as written.

The cap matches OpenAI's cap and CrispASR's default (section 9 covers CrispASR). The Gateway leaves text in angle brackets in the `input`, because the emotion tags of Orpheus, such as `<laugh>` and `<sigh>`, are inline text. Two requirements shape the route: it is compatible with OpenAI's wire format, and the Gateway is the only process with a provider's key.

### `GET /v1/audio/voices` lists every speech model's voices as objects with `id` and `name`

A speech model's `voices` list appears with the model on `GET /v1/models`, and config validation refuses the key on a model of another kind. `GET /v1/audio/voices` returns the union of the lists, sorted and deduplicated, and each element is an object with `id` first and a `name` that repeats it. A model with an empty list accepts any voice name.

OpenAI's API lacks a voice list. The route exists because off-the-shelf clients such as Open WebUI and Talemate probe for it as the de facto standard. The shape of an element matters as well: a break between Kokoro-FastAPI and Open WebUI shows that clients depend on it.

### The Gateway relays the provider's audio bytes unread, and a failed stream fails the client's read

The Gateway sends the provider's audio to the client in chunks. A spawned task owns the provider's body and the queue permit, and four constants bound it: 60 minutes in total, 1 GiB, 30 seconds of provider silence, and 60 seconds of a client that stops reading. When the stream exceeds a bound or the provider fails after the 200 status, the task ends the body with an error, so the client's read fails.

The task ends the body with an error because a normal end would look like a complete response. The Gateway is unable to validate audio chunk by chunk, and a JSON error is impossible after the 200 status and the first audio bytes. The limit on provider silence exists so that a dead provider releases the queue permit.

## 8. Voice in Workshop

### The Workshop server relays `/v1/realtime` to the Gateway and leaves the frames unparsed

The Workshop server exposes its own WebSocket `/v1/realtime` to the UI. It accepts a request whose `Origin` matches the request's own authority, opens a socket to the Gateway with the Gateway's bearer key, and copies frames in both directions, close codes included.

The UI derives all of its status from capture events and protocol events, which lets the relay copy every payload unread. The relay exists so that the key remains in the server, because a key in the webview would expose a credential to the browser. [Workshop: how the app is built](workshop-app.md) covers how Workshop finds and calls the Gateway.

### One capture service owns the microphone and refuses a second window's press with a reason

The Workshop UI has one `SpeechCaptureService`, and every agent window shares it. `start(owner, presence)` records the caller's token as the owner of the microphone. A start from a second token returns the failure `busy`, and the status bar shows `Dictation is active in another window`. The button in the owning window shows `recording`, and every other window's shows `blocked`. The status bar's `MicMeter` draws the loudness of each chunk from the service's `onLevel` event, and a click on it reveals the owning window.

The capture service decides because there is one microphone, and the capture service is the only code that knows which window has it. A press that took the microphone from the first window would destroy text that the operator is dictating, through a click in another window.

### `setupStt` returns a handle, and the part that embeds the input draws the microphone

`setupStt` connects dictation to one text input and returns an `SttHandle`. `press()` starts a take when idle and stops it when recording, and `state` is `idle`, `recording`, or `blocked`. The agent view copies the state to the chat box and forwards the box's `mic-press` event to `press()`. The button is pressable in every state, and the handle writes the reason for a refused press on the status bar.

The chat box draws the button because the button is part of the chat box, whose contract is events out and props in. [Workshop: what the user sees](workshop-ui.md) covers the button's three roles.

### Dictation inserts at the cursor like typing, and the chat box is read-only during a take

When a take starts, `setupStt` reads the selected range from the input through `insertionContext()`, and the transcript replaces that range, so several takes in a row build one message. The input is read-only from the start of the first take until the last take ends. Dictation writes through the interface `SttInputTarget`, which the chat box implements.

The model is dictation in Windows (Win+H) and in macOS: a take behaves like the person typing at the cursor. The input is read-only during a take because a person who moved the cursor would cause the next update to write at a stale offset.

### Sending a message during a recording discards the take

`submit` in the agent view reads the chat box text and then calls `discardIfRecording()`. The message contains what the operator saw, interim words included. The discard releases the microphone, sends `input_audio_buffer.clear`, and restores the text that the box had before the take.

The send discards the take because a take that continued after the send would write a late transcript into the emptied box. The send also skips the wait for the final text: the operator sent what they saw, so a refinement would be unused, and the wait for it would waste a decode.

### A pure reducer, `reduceTakeRegistry`, computes every change of take state and matches events by connection generation

`reduceTakeRegistry` takes the registry and one input, such as a user action or a decoded server event, and returns the next registry with a list of effects. `setupStt` performs the effects on the editor, the capture service, the status bar, and the socket. Every socket that `RealtimeTranscriptionService` opens has a generation number, and the registry ignores an input stamped with another generation, because an item id is unique only within one connection.

One reducer computes the state because a mesh of callbacks that coordinates take state leads to stale ownership and to rollback defects.

### The registry patches the words that changed, and one dictation is one undo step

For each hypothesis the registry computes a word-level difference with the text on screen and replaces the changed words. It hides the last tentative word until the next update. Interim writes use `addToHistory: false`, and the write of the completed transcript is the one edit in the undo history. When the socket drops during a take, the registry writes the finalized and agreed text as the result and releases the input.

The registry patches only the changed words because a rewrite of the whole take on every update makes the text flicker.

## 9. Planned work

Each of these items is still to be built.

- **A local synthesis engine.** An engine that runs an Orpheus model on the machine. Two candidates await a trial: the prebuilt speech server CrispASR as a child process, and the existing llama-server child with a SNAC audio decoder inside the Gateway. The trial transcribes the audio or compares it with a reference recording, because a wrong decode fails silently, as silent audio. Output encoders, a priority lane for speech in the dominion queue, and sampling defaults wait for the engine. A remote provider produces all audio, and the Gateway refuses a `[[local_model]]` of the speech kind (section 7).
- **Streaming from Together.** A decoder for the server-sent events in which Together streams speech under its `raw` format. The Gateway's closed format set refuses `raw`.
- **Playback in Workshop.** A player for synthesized speech: an `AudioWorklet` that reads a streamed `pcm` response and buffers 400 to 600 ms before it plays. Workshop has yet to call the synthesis route.
- **Interim windows that start at the last agreed word.** An interim window starts at the open segment (section 4). `DecodeOutput` has the word end times, and `next_window_snapshot` ignores them.
- **A Windows CUDA archive with OpenMP off.** An archive that ends the crash of the Gateway at a graceful stop under an explicit `cuda`. It waits for the next whisper.cpp tag, and the workflow builds only the Windows CPU archive with OpenMP off.
- **Decoder experiments.** A streaming interim model, AlignAtt, a rewind guard, look-ahead punctuation at segment joins, and stutter suppression. Both passes decode with whisper.cpp under the profiles of section 5.
- **A fix for two panics.** `process_closed` panics at two calls when the audio of a queued range has left the buffer.
- **Gateway-ended sessions in Workshop.** A change of Workshop's reaction to a session that the Gateway ends. Workshop rolls back the take that is recording, and for a dropped socket it writes the finalized and agreed text (section 8).
- **Tests on real Silero.** A run of the native replay test and the long-speech test with the Silero model. Both tests detect speech by loudness. [Testing and debugging](testing-and-debugging.md) covers the replay test.

## 10. Known gaps

Each item names a place where guide chapter 05, `src/gateway/05-speech.md` in `runvil-docs`, differs from the code.

1. Speech detection. The guide says "Audio quieter than -60 dBFS is treated as silence and never sent to the model" and omits Silero. `Segmenter::classify` passes each frame to a `SileroDetector`.
2. Segment close. The guide says "A segment closes only after 2 seconds of trailing silence". `endpoint` also closes after 0.2 seconds, the constant `SENTENCE_END_SILENCE_SAMPLES`.
3. The batch prompt. The guide says the `prompt` hint is "accepted but ignored by the current English whisper workers". `transcribe` in `batch.rs` adds `prompt_terms(&form.prompt)` to the decode's terms.
4. Interim-only profiles. The guide says "Without a final model, the stop falls back to the interim model." `upgrade` in `realtime/route.rs` answers 503 when `has_final_pass` is false.
5. Load failures. The guide's list of failure stages omits the `Silero` and `SileroDetector` variants of `SpeechError`.
6. Session end. The guide omits the `server_error` event and the close code 1011 that `end_session` sends.
7. Config version. The guide says "Version 2 accepts only the canonical `[stt]` section". The check in `validate.rs` requires `config-version` 0.

## 11. Questions for the owner

Each question asks for the reason behind a part of the design.

**The realtime route**

1. Why is the wire audio 24 kHz when both models decode 16 kHz?
2. Why may four committed items of one session finalize at once?
3. Why does a realtime session use one logical model name while the batch route selects a configured name?
4. Why is one dictation of any length one item and one commit?
5. Why does validation accept a profile with only an interim model, when the realtime route answers 503 under such a profile?

**Workshop**

6. Why does a dropped socket preserve the finalized and agreed text, while a session that the Gateway ends rolls back the take that is recording?

## 12. Method and limits

- **Code.** The page describes `runvil-gateway` at commit `9369a7d`, `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.

*2026-11-10 16:39 - Claude Fable 5.2*
