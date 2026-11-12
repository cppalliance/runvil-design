# Testing and debugging: design decisions and their reasons

- Scope: test harnesses and test support, fixtures, scripted models, the facade suites, the rules for test code, logs and traces, and dev tools
- Code: `runvil` at commit `fa9936f64`, `runvil-gateway` at `9369a7d`, and `runvil-docs` at `cfd69a4`
- Update: the Paths bullet names the repositories as of `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`, after the Engine and the Harness moved (section 10)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, a path that begins with `workshop/` starts at the `crates/` folder of `runvil`, and a `tools/` path starts at the root of its repository

## Summary

A step of a run returns effects: requests for outside work, such as a model reply, that the caller performs ([The Engine](engine.md)). A test steps a run on its own thread and answers every effect itself, so the clock, the seed, and the model's replies are values the test supplies. Each of the Engine's six internal crates has a `test_support` module. The one in `runvil-runtime` defines `Scenario`, a struct with one test's setup, and `ScriptedChat`, a scripted model that answers from a list of replies. The facades `runvil-engine` and `runvil`, two crates of re-exports, each have a suite written against public paths. For debugging, the Harness writes every effect, answer, and event of a run to the Host's recorder. Sections 1 to 6 describe these parts.

A check of this page against the code found four gaps and left five questions about reasons for the owner. Section 8 lists the gaps, and section 9 lists the questions. The largest gap is in the language guide: chapter 16 documents a Host debug mode that adds `Request` and `Response` events to a run, and the Harness leaves those events off in every run.

Section 7 lists the planned work. Its largest item is a replay driver, which steps a recorded run through the Engine again and compares the results.

## From a prompt to an assertion

The test `models_loop_repeats_model_tool_rounds_and_appends_each_exchange`, in `runvil-internal/runtime/src/execute/tests/models_loop.rs`, starts with two inputs. `one_section` writes a prompt with one section. The section's Lua offers `tools/echo`, adds a user message, calls `models.loop()`, and asserts on `messages`. `ScriptedChat::new` takes three replies: two calls to `echo`, built with `calls`, and one built with `text`.

`loop_scenario(source)` creates a `Scenario`. `offer("echo", echo_tool())` adds an `FnTool`, a tool whose call is a closure, to the run's tool catalog, and `chat(&gateway)` sets the scripted model. `run()` parses the source with `Prompt::parse`, builds `RunParams` from `TEST_SEED` and `TEST_STARTED_AT`, creates a `Run`, and steps it on the test's thread.

The scenario answers each effect before the next step. It answers the `Render` effect with the offered tools unchanged, and the first `Chat` effect with the first reply, through `ScriptedChat::complete_now`, which also records the round's messages and tools. That reply calls `echo`, so the Engine issues a `ToolCall` effect, and the scenario calls the closure, which returns `echoed: one`. The second round repeats the pattern. The third reply is the text `both done`, which ends the loop, and the Lua returns `ok`.

The last step returns `Step::Done`, and `run()` returns a `Trace`. The test asserts that `trace.text()` is `ok`, reads the three recorded rounds from `gateway.requests()`, and asserts that the third round's messages contain both tool results.

## 1. The Engine's test support

### Engine tests step a run on the calling thread with the serial driver

`drive(run, perform)`, in the `test_support` module of `runvil-runtime`, steps a `Run` on the calling thread and resumes it with the answer that the `perform` closure returns for each effect. Every Engine test is a plain `#[test]` function.

The owner's rule: "tokio is not allowed anywhere in the engine". Tests need only the serial driver, because the Engine leaves the order and the timing of effects to whoever answers them.

### A `Scenario` is one test's setup, and the test's answers express concurrency, time, and a cancel

`Scenario` is a struct with one run's setup as data. `Scenario::run` steps the run and returns a `Trace` with the result, the events, and each effect paired with its answer. The scenario answers a `Timer` effect at once. A test postpones an answer with `hold`, cancels from a second thread with `cancel_when`, and gets a seeded order of answers from `Rig::shuffled`. `Rig` steps a scenario's run for tests below `Run` and returns the scheduler to the test between steps.

A test uses `Rig` when it inspects something below `Run`.

### `ScriptedChat` answers each `Chat` effect from a list of replies, in the test process

`ScriptedChat::new` takes a list of `ScriptedReply` values: text replies, batches of tool calls, and failures. `complete_now` returns the next reply and records the round's messages, tools, and options. After the last reply the script repeats it.

A scripted model in the test process is enough, because the Engine leaves I/O to its caller, and the code that sends a round over HTTP is outside the Engine's crates.

### Each internal Engine crate has its own test support, under the same names

Each of the six crates under `runvil-internal` has a `test_support` module. The names repeat: a source builder (`prompt` and `one_section`, in `runvil-prompt`), prepared answers (`text`, `calls`, `fail`, and `FnTool`), and a driver that returns a record. `runvil-lua` builds a section VM with `TestVm`, `runvil-vfs` has the backend `TestFs`, and `runvil-runtime` defines `Observer` and `DebugCapture`, callback traits for run reports and raw model bodies.

The names follow the Engine facade's suite, so that tests read alike across the workspace. Each crate has its own module because one shared test crate would need a dependency of its own on each crate and would cross the `test-support` boundary, which the next entry describes.

### The `test-support` feature of `runvil-runtime` compiles the driver for one bench

`runvil-runtime` declares the empty feature `test-support`, which compiles `drive` and `ScriptedChat` for `benches/models_loop.rs`. The feature exists because `#[cfg(test)]` code is invisible to a bench, and this bench needs the serial driver and `ScriptedChat`. `runvil-lua`, `runvil-prompt`, and `runvil-plugin` declare a feature of the same name for the tests of sibling crates. A structural check fails when a production dependency enables the feature (see [Crates, build, and CI](crates-build-and-ci.md)).

## 2. The facade suites

### The Engine facade's suite uses public paths, and a test that needs more moves into the Engine

The suite in `runvil-engine/tests/suite/` names `runvil_engine` paths for every Engine item, and a test that needs a private item moves into the Engine's unit tests. A re-export made for a test grows a public surface that is hard to shrink later, because removing a public item breaks the crates that depend on it. The owner's rule: "I want ZERO public API surface growth just for tests."

### The Engine facade's suite answers each effect by a fixed rule and fails a test that would wait forever

The suite's own support code is in `runvil-engine/tests/suite/support/scenario.rs`: a `Scenario`, a `Driver` that steps a run, and a `Trace`. The `Driver` performs a `Vfs` effect with `perform_vfs_op` and answers a `Chat` effect with the next scripted reply. It fails the test when the script runs out, when replies are left over at the end, and when the run waits only on postponed effects.

The support code exists to reduce the number of lines of code in the suite.

### The Engine facade's suite contains the tests that add coverage, each a function named with a sentence

The facade re-exports items of the internal crates, so those crates test behavior. The facade's suite contains a test only when it checks something beyond what an internal test or the API listing `public-api.txt` covers. The owner's rule: "If it's not doing us any good, we should remove it, because all that does is create a tax on the vibe coding."

Each test is a separate function with a sentence for its name, so that one failure names one test.

### Harness tests write their own brokers, and the Harness exports `MemoryRecorder`

A Harness test supplies the model through an `InferenceBroker` that the test code defines. The suite of the `runvil` facade, in `runvil/tests/suite/`, uses `Offline`, which lists an empty catalog and refuses every round as `Unavailable`. A public scripted broker would be test API that only the Harness's own tests use, so the Harness leaves it out.

`MemoryRecorder` is public and stores each run's records in a vector. `effect_loop::drive_run` in `harness-runner` is public so that tests have one entry point to the effect loop.

## 3. Rules for test code

### Every clock value, seed, and answer in an Engine-crate test is a value the test supplies

Three `clippy.toml` files ban sleeps, clock reads, timed waits, and `RandomState::new` in `runvil-engine`, `runvil-plugin`, and the crates under `runvil-internal`, tests included (see [Crates, build, and CI](crates-build-and-ci.md)). The test runner, nextest, ends a hung test after three 60-second periods.

The owner's rule: "my instinct tells me that no runvil test should ever sleep or be non-deterministic". The rule covers the Engine crates only, because some Gateway tests are timing-based by nature. The Engine's own determinism rules are on [The Engine](engine.md) and [Tasks and fanout](tasks-and-fanout.md).

### Random test input uses a fixed seed, and `TempRoot` names a temp folder after the test and the process id

`Seeded` is a SplitMix64 generator in the test support of `runvil-types` and `runvil-runtime`. The rule above bans only entropy from the operating system and the clock, so a test may draw input from a fixed seed, and a failure prints its seed.

`TempRoot::new(label)` creates `<temp>/runvil-tests/<crate>/<label>-<pid>-<n>` and removes it on drop. The Engine crates use it because the `tempfile` crate depends on the entropy crates `fastrand` and `getrandom`, and the Engine crates exclude entropy crates even as transitive dependencies.

### Four async tests outside the Engine crates pause tokio's clock, and tests on real sockets wait in real time

Four tests use `#[tokio::test(start_paused = true)]`, in the Harness runner, Workshop's support crate, and the Gateway. A test that uses a real socket, a subprocess, or an operating-system thread waits in real time, because paused time with real sockets fires the timeouts of the code under test early. The heartbeat test of Workshop's server sleeps in real time because its probes are real HTTP requests over loopback on the test runtime, and those requests complete only in real time.

### A test checks behavior, and its pass condition is a true-or-false property

`AGENTS.md` states the preference: "Prefer types and compiler checks, then behavior tests and deterministic fault injection. Gate on yes/no properties, never on a score threshold that ordinary work edits." A test of the shipped research prompt runs the prompt and inspects every `Chat` effect for a tool id. A scan of the prompt files would check only the shape of the source, and the test on `Chat` effects protects the actual contract.

Two build rules apply to tests: a commit that removes a feature also deletes the feature's tests, and an author names a test file beside its module `<stem>-tests.rs` (see [Crates, build, and CI](crates-build-and-ci.md)).

## 4. Fixtures and stand-ins

### Whole prompt files are fixtures in two `tests/prompts` folders

Fourteen prompt files under `runvil-internal/runtime/tests/prompts/`, in the folders `valid`, `invalid`, and `execution`, are fixtures for the Engine's suite. Six more under `runvil-engine/tests/prompts/invalid/` are fixtures for the facade's parser tests. The Engine's suite names each file in an `include_str!`. A fixture is a complete prompt, and inline tests cover the narrow cases.

The fixtures exist because the inline tests leave a gap: a complete document, written the way an author writes one.

### A Plugin crate tests its tools through `TestCall`, and the loader's tests load a fixture library

The `test-support` feature of `runvil-plugin` exports `testing::TestCall`, which owns a tool id, an access to a fresh in-memory file system, and an origin, and lends a `ToolContext`. It exists because the dependency rules refuse a Plugin crate that names the facade or the file-system crate.

The loader's end-to-end tests load `harness-test-plugin`, a dynamic library with a private copy of the C types. The private copy proves that a library Plugin depends on zero Runvil crates.

## 5. Gateway, speech, and Workshop tests

### Every change to speech behavior must pass the speech-sandbox replay

The speech-sandbox replay is the test `every_replay_fixture_matches_golden_snapshots_metrics_and_baseline_thresholds` in the Gateway's `stt/api` crate. It replays fixtures, some scripted and some captured from a real decode, and reads each event's time from the fixture. It fails when the share of shown words that a later update rewrites rises above the committed baseline, and when partial latency or commit lag exceeds 110 percent of the baseline.

The replay tests the speech pipeline's latency and quality, and it follows two reference projects that measure commit behavior by replay with a frozen clock. A fixture captured from a real decode is a fixed input, so a new capture gets a new name.

### A test that needs a GPU, speech models, or a provider key runs on request

The Gateway's CUDA test is `#[ignore]`, and it returns early unless `RUNVIL_LIVE_CUDA=1` is set. The test exists because CUDA linkage, GPU offload, MTP acceptance, projector use, and the reuse of a multi-gigabyte cache can all compile successfully and fail only on real hardware. The long-speech test `realtime_long_speech` is `#[ignore]` too, and it fails when a transcript lacks 5 consecutive words of its clip's text. The script `tools/gateway-tts-live.mjs` in `runvil-gateway` boots a Gateway on a provider-backed profile and asserts on the Gateway's own responses, because drift at the provider shows in those responses, and they are what clients see.

### Workshop tests its UI on Node's built-in runner and its server in process

`npm test` in `workshop/ui` runs `node --test`, the test runner that Node ships, over every `.mjs` file under `test/`, 122 files, 10 of them helpers. Vitest is the fallback. Two conditions call for a second look at it: a need that Node's runner is unable to express, or parallel flakiness that proves unfixable.

The server's integration tests spawn the real server in process against a mock Gateway.

## 6. Logs, traces, and dev tools

### The Harness writes every effect, answer, and event of a run to the Host's recorder

The Harness calls `RunRecorder::append` for each effect, answer, and event, in the order of its effect loop (see [The Harness and the Host API](harness-and-host.md)). Workshop's recorder writes them to `harness/runs.db` in its state directory, a Turso database with the tables `runs` and `records`. A developer who debugs a run reads that file. Workshop runs prompts, and each facade has one example program, `greeter.rs` and `run-prompt.rs`.

Stored JSON reads back as the identical value, so that a replay can compare the results of a second run against the stored ones. The replay is still to be built, and the `workshop-run-log` README says: "nothing reads a record back into the Engine".

### A prompt writes a trace line with `log`, and raw model bodies are opt-in events

`log(message)` accepts one single-line string of at most 256 characters, and the Engine reports it as an `Event::Lua` under the current section (see [The prompt language](prompt-language.md)). The execution fixtures assert the exact list of these checkpoints.

`RunParams::with_report_debug(true)` adds two events to each model round: `Request`, the JSON body sent, and `Response`, the body returned. The events exist so that a developer can diagnose an empty reply, and they are optional so that a production run pays zero for them. A Harness run leaves them off, and section 8 lists that as a gap.

### Workshop discards `tracing` lines, and the Gateway writes a log file

The Harness and Workshop report operational problems with `tracing` macros, such as the error `Pack factory panicked; the Pack is unavailable to this run`. Workshop lacks a `tracing` subscriber, so its process discards these lines. Section 7 lists the subscriber as planned work.

The Gateway installs a subscriber that writes a rotated `gateway.log`, and `runvil-gateway diagnostics` prints a JSON report (see [The Gateway](gateway.md)).

## 7. Planned work

Each of these items is still to be built.

- **A replay driver for run logs.** A driver that steps a recorded run through the Engine again and compares the results. Workshop's recorder stores every effect, answer, and event of a run in `harness/runs.db`, and a developer reads them there.
- **A `tracing` subscriber for Workshop.** Code in Workshop's process that writes each `tracing` line to a log. The process discards these lines, as section 6 describes.
- **Desktop tests on a Tauri runtime.** Tests of `menu.rs`, `linux_media.rs`, and `window_state::apply`, which need Tauri runtime fixtures. The desktop crate lacks those fixtures.
- **A CI job with the Windows symlink privilege.** A job on a Windows runner that has the privilege to create a file link. Windows runs skip the three file-link tests of `runvil-vfs`.
- **Retiring `tools.call_as_model`.** `tools.call_as_model` is a test hook through which a fixture section calls a tool as a model would. Test support installs it, and the Lua shim has a function that exists only for it.
- **The determinism rule for Harness, Gateway, and Workshop tests.** An extension to these tests of the rule that a test supplies every clock value, seed, and answer, which depends on deterministic drivers for those crates. The rule covers the Engine crates only (section 3).

## 8. Known gaps

Each item names a place where a guide differs from the code, or where two parts of the code disagree.

1. The run record. Guide chapter 4, under "What a run's record holds", lists a `name`, a `prompt_hash`, and an `execution` field on every event. The code lacks all three, and the tests `a_new_log_has_neither_a_name_nor_a_prompt_hash_column` and `no_serialized_event_has_an_execution_key` assert that.
2. Raw model bodies. Guide chapter 16 says: "In Host debug mode, the run adds `Request` and `Response` events that hold the raw model request and response bodies." `prepare.rs` in `harness-runner` builds the `RunParams` of every Harness run and leaves `with_report_debug` off, and `RunSetup` lacks a field for it. `StreamAccumulator::finish` in `harness-gateway-client` attaches both bodies to every `Completion`, and a Harness run leaves them unread.
3. The Harness's log. Guide chapter 4 says that for an unavailable Pack "the Harness writes a line naming the Pack to the Harness's own log". `packs.rs` in `harness-runner` emits that line through `tracing`, and Workshop discards it (section 6).
4. The speech probe. `tools/gateway-tts-live.mjs` has identical bytes in `runvil` and `runvil-gateway`, and its `buildGateway` function runs `cargo build -p gateway`. The `gateway` package is in `runvil-gateway` only, so the copy in `runvil` fails at its build step when a provider key is set.

## 9. Questions for the owner

Each question asks for the reason behind a part of the design.

**Engine test support**

1. Why does `ScriptedChat` repeat its last reply when the script runs out, while the Engine facade's suite fails the test?
2. Why does the Engine's suite name each prompt fixture by hand in an `include_str!`?

**Rules for test code**

3. Why do tests such as `no_serialized_event_has_an_execution_key` assert that a field is absent, when a commit that removes a feature also deletes every trace of it?
4. What is the reason for paused time in in-process async tests?

**Logs and traces**

5. Why does `RunSetup` omit a switch for the `Request` and `Response` events?

## 10. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64`, `runvil-gateway` at `9369a7d`, and `runvil-docs` at `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. Their tests, test support, and benches moved with them, and the CI of `runvil-harness` runs them on Linux and on Windows. The Workshop crates and their tests stay in `runvil`. The text of the page still describes the code at the commits above, when all of these crates were in `runvil`.

*2026-11-10 16:47 - Claude Fable 5.2*
