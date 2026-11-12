# Workshop: how the app is built: design decisions and their reasons

- Scope: the desktop app, the Workshop server and its API, the wire protocol, the Workshop crates, stored state, the workspace document, running prompts through the Harness, calling the Gateway, and the UI's three TypeScript packages
- Code: `runvil` at commit `fa9936f64`, `runvil-gateway` at commit `9369a7d`, and `runvil-docs` at commit `cfd69a4`
- Update: the tiers and the dependency checks of section 3 describe `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`, and the text on Plugins describes `runvil` at `b0693699d` (section 12)
- Paths: code paths start at the `crates/workshop/` folder of `runvil` unless they begin with `crates/`, `.cursor/`, or a repository name

## Summary

Workshop is a desktop app that runs as one process. The desktop app, a Tauri program named `runvil-workshop`, starts the Workshop server on a thread and opens a window on the server's loopback URL. The server sends the UI bundle to the window, answers HTTP routes for files and saved state, and pushes frames over two WebSockets. For each chat the server runs a prompt through the Harness, and it calls the Gateway, a separate process, over HTTP. Sections 1 to 8 describe these parts.

A check of this page against the code found eleven gaps and left nine questions about reasons for the owner. Section 10 lists the gaps, and section 11 lists the questions. The largest gap is the profile switch: the server runs it for a `switch_profile` frame, and the UI lacks a sender for that frame.

Section 9 lists the planned work. Its largest item is a real folder for a run: every run starts with an empty in-memory file system.

## From a typed message to a reply on the screen

The desktop app starts `workshop-server` with `spawn` and opens the `main` window on the server's URL. The UI's `main.ts` opens the workshop socket, `/ws`, for `status`, `models`, and `workbench` frames. Each agent panel creates an `AgentSocket` for `/agents/ws` and an `AgentSessionService` that turns frames into transcript items.

When the agent socket connects, the server's `run_socket` loop sends an `agents` frame, and the UI answers `{"type":"launch","agent":"chat"}`. `AgentSessions::launch` reads the agent with `load_agent`, opens a `Conversation`, and spawns `drive`, and the loop answers with an `agent_session` frame that names the session id. `drive` builds a `RunSetup` and calls `Conversation::run`, which parses the prompt with `Host::parse` and runs a `Harness`. The chat prompt calls `tasks.receive()`, and the server sends the Engine's `InboxWaiting` event as an `agent_event` of kind `awaiting_input`.

The user types a message and presses Enter. `AgentSocket.respond` sends `{"type":"input_response","text":"..."}`. The loop calls `Conversation::send_input`, which calls `RunControl::send`. The Harness appends the text to the inbox of the main chain, which is the run's root task. `tasks.receive()` returns the text, and the prompt calls `models.loop`.

The Harness asks the run's `RunBroker` for the model round. The broker reads the current Gateway binding and the selected model and calls the Gateway. For each streamed piece it calls `Conversation::publish_delta`, and the loop sends an `agent_delta` frame with the round's `reply` id. When the round ends, the Harness passes the `AssistantReply` event to `ConversationRecorder`, which writes it to the run log and then calls `Conversation::observe`. The loop sends the event as an `agent_event` of kind `agent_message` with an `index` and the same `reply` id, and the UI replaces the streamed chunks with the finished reply. The prompt calls `tasks.receive()` again, and the next `awaiting_input` event ends the working state.

## 1. The desktop app and its window

### The desktop app starts the server on a thread and opens a window on its URL

The crate `workshop` in `desktop/` builds the binary `runvil-workshop`. Its `boot` function loads the config, connects the Gateway, calls `spawn` to start `workshop-server` on a thread, and waits for the server's `/health` route. `open_window` then creates the `main` window with `WebviewUrl::External` and the server's URL. The window's page is the UI bundle that the server embeds.

Workshop is a standalone program. The alternative, an extension of a code editor, would make its development depend on that editor.

### The desktop app is a Tauri v2 program, and one module contains its unsafe code

`main.rs` builds a `tauri::Builder` with five plugins, among them a single running instance, dialogs, and updates ([Crates, build, and CI](crates-build-and-ci.md)). `bridge.rs`, compiled on Windows, is the crate's one module with unsafe code: through WebView2's COM interface it reads the paths of dropped files and grants the microphone ([Speech](speech.md)).

The owner's goal: "Workshop has to be a polished, finished product worthy of delivering to users as a packaged desktop application". Workshop's stack is Tauri, Rust, and CodeMirror, where the alternative is a fork of VS Code's open-source editor, because that stack and the agent panel are the product's advantage.

### The OS picks the server's port, and the window loads that one origin and grants it the Tauri API

The desktop app sets the bind address to `127.0.0.1:0`, so the OS picks a free port at each launch. `window_capability` grants the Tauri permissions of the `main` window to that exact origin. `classify_navigation` lets the webview load a URL whose origin equals the server's and opens every other http or https URL in the system browser.

The capability names the exact origin because a wildcard origin would grant the window's Tauri API to every other loopback server on the machine. With the navigation rule, a clicked link to another origin opens in the system browser, and the window still shows the app.

### Closing the window stops the server, and quit also stops a local Gateway

When the window closes, Tauri's exit event shuts down the Gateway supervisor and then the server, and the Gateway process continues to run with its models loaded. The window menu's quit item and the UI's File > Exit row both call `quit_everything`, which stops the supervisor, posts the Gateway's `/shutdown`, and exits. For a Gateway named in `workshop.toml`, quit stops the desktop app, and that Gateway continues to run.

`quit_everything` stops the supervisor first, because a supervisor that still runs when the Gateway stops sees the Gateway missing and launches it again.

### Workshop starts on built-in defaults when `workshop.toml` is absent

`config::load` looks for `workshop.toml` beside the executable, then in the current directory, then in `<home>/.runvil`. `Config::load` replaces each `${VAR}` in a string value with the environment variable, or with an empty string for an unset one. When all three places are empty, `default_config` builds the config in code, with the state directory `<home>/.runvil` and the agents folder `<home>/.runvil/agents`. The file's tables are `[gateway]`, `[server]`, and `[agents]`.

The owner's reason: "can you imagine if you install Photoshop.exe and you have to set some env vars first in order to run it? no way." The Gateway writes its own default config on its first run ([The Gateway](gateway.md)).

## 2. The server process and its HTTP routes

### `spawn` checks the bind address, binds, reopens the last workspace, and then returns

`spawn(config)` is the one function that starts the server. It returns an `InvalidInput` error for a bind address outside loopback before it starts a thread. The thread, named `workshop-server`, has its own tokio runtime: it builds the shared state, binds the listener, reopens the last workspace, and reports the URL. `ServerHandle::shutdown` waits up to 5 seconds for open requests. When some remain, it tears the runtime down and returns `Termination::Forced`.

`spawn` refuses an address outside loopback because the guard of the next entry checks the `Host` header, and a raw client on the local network can forge a loopback `Host` header. The thread binds before it reopens the workspace, because a failed bind after the reopen would leave the `.rvwork` file's `-wal` sidecar beside the file (section 7).

### A guard on the API routes refuses cross-site requests

`cross_site::guard` wraps every API route. It answers 403 to a request that the browser marks `Sec-Fetch-Site: cross-site` and to a request whose `Host` header names an address outside loopback, and 415 to a POST, PUT, or PATCH whose content type differs from `application/json`. The three WebSocket upgrade handlers also check the `Origin` header.

The guard exists because any web page that the user visits can send requests to a loopback address, and these routes write workspace files and open WebSockets. The Gateway has a policy of its own for the same headers. The two policies are separate code, because a merge would change a trust boundary.

### A route that misses its deadline answers a JSON 408

`with_deadline` in `workshop-support` puts a time limit on a group of routes: 10 seconds by default, and 35 seconds for `/v1/models`, which relays a Gateway call whose own timeout is 30 seconds. The WebSocket upgrades and the config panel's proxy run outside any deadline. A late route answers 408 with the body `{"error":{"message":"...","code":"deadline_elapsed"}}`. The message says the operation may still complete, because the server abandons the response while blocking file work continues.

The doc comment on the constant `DEADLINE_ELAPSED_CODE` calls the code `deadline_elapsed` a wire contract and states that both UIs key on the string. Section 10 says how the UIs differ from that comment.

### A `cargo build` bundles the UI, and the server sends the bundle under hashed names

The server crate's `build.rs` calls `build_ui::build_sibling` on `../ui`, which runs esbuild on `ui/src/main.ts` and writes the bundle under `$OUT_DIR`. A build needs Node.js 22 and one `npm ci` in `crates/workshop`. esbuild names each output file with a hash of its content, and the server answers a hashed file with `public, max-age=31536000, immutable`.

The build script runs the bundler so that a plain `cargo build` updates the executable with every change to the TypeScript.

## 3. The Workshop crates and the registry

### Fourteen of Workshop's sixteen Rust crates form four tiers, and each dependency points to a lower tier

`cargo test -p build-xtask` reads the tiers from `crates/build-xtask/src/tidy.rs` and fails when a crate depends on its own tier or a higher one.

- Vocabulary: `workshop-protocol` (wire frames), `workshop-support` (atomic writes, backoff, deadlines, config, `RetainedBus`), `workshop-registry`, and `workshop-gateway-discovery` (the reader of the Gateway's discovery file).
- Library Plugins, a second table of the lowest tier: `workshop-plugin-std` (the dynamic library of the `std` Plugin) and `workshop-plugin-bashkit` (the dynamic library of the bashkit Plugin). Each of the two crates depends only on crates outside the Workshop family, and `workshop-server` alone may depend on them.
- Services: `workshop-gateway` (Gateway client, binding, heartbeat, progress), `workshop-menu` (the Model menu and model catalog), and `workshop-status` (the status bus).
- Features: `workshop-agents` (conversations), `workshop-run-log` (the run log), `workshop-user-state` (`ui-state.json`), and `workshop-workspace` (granted folders and the `.rvwork` file).
- Server: `workshop-server` (composition root, routes, and the two sockets).

`workshop-server-api` and the desktop app `workshop` are outside the tiers, and `workshop-registry` may depend on `workshop-protocol`.

The rule behind the tiers: the dependency graph has to point one way only, it has to be a tree, and cargo has to enforce it. [Crates, build, and CI](crates-build-and-ci.md) covers the naming rules and the checks.

### Each subsystem registers its routes, state, sinks, and tasks in `workshop-registry`

`Registry` has four collections: routes and background tasks as ordered lists, and state handles and push sinks as maps keyed by Rust type. A subsystem's `register` function adds its entries and returns a struct of guards, and dropping a guard removes the entry. A consumer reads a handle by type with `state::<T>()` and treats `None` as an absent feature. In `server/src/app/compose.rs`, `compose` calls `require::<T>()` for seven handle sets, and a missing one stops startup with `MissingContribution`. The registry also defines the traits that connect subsystems, such as `WorkspaceRoots` and `MenuSink`.

The owner's goal: "I want a central server that has an API that lets the sub-crates install themselves, so we don't have a cyclic dependency." The registry defines these traits because a trait in a subsystem's own crate would make one service crate depend on another, which the tier check forbids.

### Each public item has one path, through its crate root

In `workshop-gateway`, `workshop-workspace`, and `workshop-server`, every production module is private, and the crate root re-exports the items that other crates use. Items for tests compile under the `test-fixtures` feature. The goal: each public item has exactly one public path and at least one user outside its crate, and the test items behind `test-fixtures` are the exception. Section 10 names two crates that differ.

### `workshop-server-api` is the desktop app's one Workshop dependency, and Workshop crates import the Engine through `runvil`

`workshop-server-api` is a list of re-exports: the config types, `spawn`, `ServerHandle`, `GatewayUpdater`, and the `discovery` module. The check in `crates/build-xtask/src/product.rs` fails when the desktop app names another Workshop crate. The Engine and the Harness are in the `runvil-harness` repository, and the root `Cargo.toml` pins four of its crates by commit: `runvil`, `harness-gateway-client`, `plugin-web`, and `plugin-mcp`. The check in `crates/build-xtask/src/harness_pin.rs` fails when a crate here takes any other crate of that repository, so an Engine crate stays out of every Workshop manifest ([Crates, build, and CI](crates-build-and-ci.md)). `workshop-agents`, `workshop-run-log`, and `workshop-server` import the Engine, the Harness, and the Plugin contract from `runvil` ([The Harness and the Host API](harness-and-host.md)).

## 4. The two sockets and the wire protocol

### The UI opens `/ws` for app state and one `/agents/ws` socket for each agent panel

`/ws` is the workshop socket: the server pushes `status`, `models`, and `workbench` frames on it, and it accepts `select_model` and `switch_profile` frames. `/agents/ws` is the agent socket: each agent panel opens one, and one socket has one conversation. The server modules `workshop_socket` and `agents` each define their own router and state type. The UI's `WorkshopSocket` and `AgentSocket` reconnect by themselves, with a delay that doubles from 1 second up to 30.

### One task owns each socket

Each connection is one task with one `select!` loop that reads the client's frames and writes every outgoing frame: `run_connection` for `/ws` and `run_socket` for `/agents/ws`. The loop receives status, catalog, menu, event, and delta updates from broadcast channels.

### Every pushed frame is durable or ephemeral

The crate docs of `workshop-protocol` put each frame the server pushes into one of two classes. The server always delivers a durable frame. `agent_event` frames are durable: the socket's loop sends, in order, every transcript entry past its cursor that has a frame type, and it numbers the frames from zero. The server may drop an ephemeral frame for a slow client. `status`, `models`, and `workbench` frames are ephemeral complete snapshots: `RetainedBus<T>` in `workshop-support` stores the newest value, and the loop sends it to each new connection. `agent_delta` frames are ephemeral too, and the transcript omits them.

The contract has these two classes because a promise of exact delivery is too strong for a transport that disconnects.

### Two hand-written files define the frames, and tests on both sides assert the same fixtures

The Rust types for the `/ws` frames and for the user's `input_response` frame are in `workshop-protocol`. The agent socket's other frames are in `server/src/agents/wire.rs`. The TypeScript types for both sockets are in `ui/src/services/protocol.ts`. Rust tests and UI tests assert the same two JSON fixtures, `workshop-frames.json` and `agent-frames.json`, so a change of shape on one side fails a test. In the UI, `narrowFrame` checks each inbound frame with a guard and drops a frame that fails.

The files are hand-written, and the first bug from a difference between them would justify a code generator. The agent socket's other frames name Engine types, so they are in the server: in `workshop-protocol` they would make `workshop-status` and `workshop-user-state` compile the Engine and Lua.

### The server owns the Model menu state, and the UI applies each snapshot

`workshop-menu` stores what the code calls the Model menu state: the Gateway's profile list, the active profile, a switch in progress, and the selected model. On each change it publishes a complete `WorkbenchSnapshot`, which `/ws` sends as a `workbench` frame with the fields `profiles`, `active`, `switching`, `switch_in_flight`, `selected`, and `chat_ready`. The agent toolbar's model picker sends a click as a `select_model` frame and shows the selection of the next snapshot. The UI lacks a sender for the socket's other inbound frame, `switch_profile` (section 10). The server computes `chat_ready`: the catalog has a chat-capable model, a model is selected, the switch flag is clear, and the heartbeat says the Gateway answers.

The owner's rule: "The menu is driven entirely from state signals from the server." The UI leaves `chat_ready` to the server because a boolean is trustworthy only when one party computes it.

### The status bar has one frame type, and the server decides what it shows

A `status` frame is a `StatusBarUpdate`: a `label`, a `description`, a `busy` flag, a `severity`, and an `activity` of `general`, `thinking`, or `generating`. Server code sends one through the methods of `Push`, such as `push_busy` and `push_idle`. For each conversation, `server/src/agents/status.rs` spawns a reporter that reads the conversation's events, deltas, and failure reports: a delta pushes an activity pulse, a failure pushes an error, and the main chain's wait on its inbox pushes idle. In the UI, `ActivityIndicator` sets the activity light from these frames. For the Gateway's progress stream, `workshop-gateway` sets `busy` after 1 second of work and for at least 500 ms.

## 5. Running prompts through the Harness

### Every chat is a prompt that the Harness runs

A chat in Workshop is one run of a prompt. The loop that reads the user's message, calls the model, and runs tools is Lua in the prompt, and the Harness performs the model calls and the tool calls ([The Harness and the Host API](harness-and-host.md)).

### An agent is a `.md` file in the agents folder, and `chat` is compiled in

`discover_agents` lists the stems of the `.md` files in the folder that `[agents] path` names, plus `chat`. `BUILTIN_CHAT_SOURCE` embeds `agents/chat.md` at compile time ([Prompts that ship](shipped-prompts.md)), and a `chat.md` in the folder replaces it. `load_agent` accepts a name that `discover_agents` lists. The agents folder is a Workshop feature: the Harness receives the prompt text from the Host's prompt finder.

The built-in exists so that a fresh install always has a working chat.

### `workshop-agents` owns conversations, and a conversation outlasts its socket

A `Conversation` is a cloneable handle to one launched agent and its one run. It has a random 128-bit id, a transcript in memory, three broadcast channels for events, deltas, and failure reports, and the run's `RunControl`. `Conversations` is the table of the running ones. A socket that reconnects sends `attach` with the id, and the server replays the transcript from index zero. When the run ends, the conversation removes itself from the table. A server restart loses the transcript.

The conversation code is in Workshop because Workshop is the only chat Host, and the Harness only prepares and runs each run.

### The server builds one `Host` at startup and one `RunSetup` for each run

`AgentSessions::new` builds the `Host` that every run shares: its Packs, a prompt finder over the agents folder, and a tokio timer. `with_plugins` calls the `install` function of two Plugins: `plugin_web`, and `plugin_mcp` with the servers from the file that `[agents] mcp` names. It then loads each dynamic library in the directory that `[agents] plugins` names, through `insert_plugin_dir`, and the `std` Plugin is one of those libraries ([Tools and Packs](tools-and-packs.md)). The desktop app fills an unset `[agents] plugins` with the `plugins` directory of its own installation, and `runvil-workshop --plugins` lists what that directory loads. When the registry lacks `runvil/std`, the server emits a `tracing` warning, and `Host::parse` refuses a prompt that requires `std`. For each run, `setup_for` builds a `RunSetup` with the conversation's recorder, a `RunBroker`, `has_user: true`, and an empty in-memory file system from `run_vfs`. `with_host_state` then copies the Model menu's selected model and the workspace's granted folders into the setup, and the prompt reads the model and the first folder through `ui()`. A grant or a revoke after that applies to the next conversation.

To a run, the granted folders are `ui()` data only, so a grant or a revoke leaves the run's file access unchanged. A `ui()` that changes during a run would need a new channel from the Host into the run.

### Workshop's broker reads the Gateway binding and the selected model for every round

The Harness asks the Host's inference broker for each model round. `WorkshopBroker` reads the current Gateway binding from the registry on every call and caches one `GatewayBroker` for each binding generation, so a relaunched Gateway answers the next round. Its `routed` function sends the round to the model that the Model menu has selected at that moment ([Model calls and conversations](models-and-conversations.md)). `RunBroker`, one for each run, streams the pieces of a section's own round into the conversation. When the binding is unusable, the broker fails the round with `ErrorCode::Unavailable`, and `launch` refuses a new conversation.

The broker reads both for each round so that a change of either applies while the agent runs, and the run continues.

### The recorder tee writes each event to the run log and then to the transcript

The Harness reports each effect, answer, and event of a run to the `RunRecorder` in its `RunSetup`. Workshop passes `ConversationRecorder`, a tee: it forwards each record to `TursoRecorder`, and after that recorder accepts an event, it calls `Conversation::observe`, which appends the event to the transcript and broadcasts it. `workshop-run-log` stores the log in `harness/runs.db`, a database of Turso, an embedded SQL library, with a `runs` table and an append-only `records` table.

The run log's schema is a list of `CREATE TABLE IF NOT EXISTS` statements, which leave the tables of an existing file as they are, and the database omits a version number. A person must therefore delete a file from an older build before a new build writes to it.

### The server sends each message at once, Stop drops the round, and close cancels the run

The socket's loop passes the text of an `input_response` frame to `Conversation::send_input`, which calls `RunControl::send`. While the prompt waits in `tasks.receive()`, the text answers it, and while a model round runs, `models.loop` reads the text before its next model request ([Tasks and fanout](tasks-and-fanout.md)). `send_input` returns false before the run has its Harness and after the run has ended, and the socket then answers with an `error` frame. A `cancel` frame calls `stop_round`: the run drops its effects in flight and continues. `Conversation::close` cancels the run. The socket's other requests are `launch` and `attach`, so a conversation ends when its run ends or the process exits.

`Conversation` refuses a message typed before the run has its Harness, because the place for it is a follow-up queue in Workshop, which section 9 lists as planned work.

### The agent socket sends indexed transcript events and live chunks, with the Agent Client Protocol's names

An `agent_event` frame has an `index`, an `event`, and for a model round a `reply` id. The `index` counts, from zero, the transcript entries that have a frame type, and the transcript's own index also counts the events that the wire omits. The event kinds are `user_message`, `agent_message`, `agent_thought`, `tool_call`, `tool_call_update`, `awaiting_input`, and `run_ended`. `AgentEvent::from_event` builds them from the Engine's events: a message in the main chain's inbox becomes `user_message`, the main chain's wait on its empty inbox becomes `awaiting_input`, and the run's end becomes `run_ended`. An `agent_delta` frame is one streamed chunk with its round's `reply` id, and the UI replaces the chunks when it receives the event with the same id.

The names are the Agent Client Protocol's because the draft of that protocol has the same architecture.

### `POST /prompts/contract` parses prompt text and returns what its front matter declares

The Run window posts a prompt's name and text. The route calls `Prompt::parse` and answers a `ContractResponse` with `name`, `description`, `runvil`, `max_tool_iterations`, `input`, `output`, `uses`, `args`, and `models`. A prompt that fails to parse answers 422 with a code such as `parse_frontmatter` or `parse_lua`.

The route is in the server crate because it only parses a prompt: its work is independent of the granted-folder check, and in `workshop-workspace` it would be that crate's only reason to depend on the Engine.

## 6. Finding and calling the Gateway

### The Gateway is a separate process, and Workshop calls it over HTTP

`workshop-gateway` contains `GatewayClient`, Workshop's HTTP client for the Gateway. The server uses it for the model list, the profile routes, the progress stream, and the config panel's proxy. A run's model rounds use `GatewayBroker` from `harness-gateway-client`. The UI calls the Workshop server, and the server adds the key: it relays `/v1/models`, the config panel's calls, and the speech socket `/v1/realtime` ([Speech](speech.md)). [The Gateway](gateway.md) covers the Gateway itself.

Every Gateway is a process of its own, so one code path calls a local Gateway and a Gateway on the network.

### At boot the desktop app attaches to a running Gateway or launches the one beside it

`ensure_gateway` calls `plan_gateway`, which returns one of four `GatewayPlan` values. `Attach`: the discovery file `gateway.json` in `<home>/.runvil/run` names a live Gateway. `Launch`: the file is absent or stale, and `runvil-gateway` exists beside the executable. `ConfigOnly`: both are absent, and `workshop.toml` has a `[gateway]` URL. `Fail`: boot ends with an error dialog. The launch is a detached `std::process::Command`. The Gateway is the one writer of the discovery file and of `gateway.toml`. The server then resolves the same endpoint in `resolve`: the discovery file first, the `[gateway]` table second.

The launch is detached because the Gateway owns its lifecycle. The sidecar API of Tauri's shell plugin kills its children at exit, which is the opposite of this design.

### The desktop app supervises a Gateway that it found through the discovery file

`supervise` starts a `GatewaySupervisor` thread when boot attached through the discovery file, and skips it for a Gateway from the `[gateway]` table. Every 5 seconds the thread resolves the discovery file again. When the Gateway is missing, the thread launches the sibling binary, validates the new process, and publishes the new port and key to the server. The supervisor's one way to stop a process is an authenticated shutdown request, which it sends to a process it launched and failed to publish.

The supervisor exists because a local Gateway listens on a random port. After that Gateway exits, the server's endpoint is dead until the supervisor publishes the port of a new one. The supervisor stops a process by request only, because the identity of the process behind a PID can change or remain uncertain.

### A replacement of the Gateway binding requires a `ValidatedConnection`

`GatewayBinding` stores one immutable `GatewaySnapshot`: the HTTP client with its base URL and key, a generation number, and the validated identity of a local Gateway. Every Gateway call in the server reads the current snapshot. `GatewayUpdater::replace_sidecar_cancellable` installs the next generation, and its argument is a `ValidatedConnection`. That type's constructor is private to `workshop-gateway-discovery`: `validate` checks the process image name, the boot identity, the health route, and that the Gateway accepts the key.

The updater takes this type because an updater that accepted a raw discovery file would leave validation to its callers, and a caller could bypass the trust boundary. The identity is part of the snapshot because, with the identity in a second store, quit could target a retired process and leave its replacement alive.

### A heartbeat probes the Gateway every 5 seconds, and Workshop runs while the Gateway is down

The heartbeat task in `workshop-gateway` calls the Gateway's `GET /health` every 5 seconds while the Gateway answers. After a failed probe, the next probes wait for growing delays from `ReconnectBackoff`, which resets after useful work such as a completed model reply. Each change of state sets the Model menu's `gateway_reachable` flag, and when the Gateway answers again, the task refetches the profile state and the model catalog.

The requirement behind the heartbeat: Workshop tolerates the loss of its connection to the Gateway and reconnects.

### A profile switch restarts a Gateway that Workshop supervises

For a `switch_profile` frame, which the UI lacks a sender for (section 10), the server publishes a snapshot with `switching` set and starts one task for the switch, and a second request during the switch gets an `error` frame. `drive_switch` posts the selection to the Gateway. When the answer says `restart_required` and the binding is a local Gateway from the discovery file, the server posts `/shutdown` and the supervisor launches the Gateway again. The server waits up to 90 seconds for the new generation. For a Gateway from the `[gateway]` table, the status bar asks the user to restart it by hand.

The switch is a task of the server because a profile switch is global server state: it completes even when the client that sent the frame disconnects.

### The server proxies the Gateway's config UI and forwards admin routes by rule

The Gateway's config UI runs in an iframe in a Workshop panel. The server sends the UI's files at `/gateway/config/` from Workshop's own origin, because a cross-origin iframe makes a console window flicker. `/gateway/api/{*path}` forwards the panel's calls to the Gateway with the key attached. `forward_allowed` forwards any method on a path under `/admin/`, except `GET /admin/progress`, plus two cache routes under `/v1/cache`, and it refuses every other path.

The proxy forwards by rule because a rule is less code than an exact list of routes, and an exact list has to change each time the Gateway's admin routes change.

## 7. Stored state and the workspace document

### A person writes the config, and each crate writes its own state file

- `workshop.toml`: the config. A person writes it, and the program reads it.
- `workshop-state.json`: the last model selected for each Gateway profile, written by `workshop-menu`.
- `ui-state.json`: the user's UI state, written by `workshop-user-state`.
- `last-workspace`: the path of the last workspace file, written by `workshop-workspace`.
- `harness/runs.db`: the run log, written by `workshop-run-log`.
- A `.rvwork` file: the workspace, at a path of the user's choice.

The four files in the middle are in the state directory, `[server] state_dir`, which defaults to the config file's folder or to `<home>/.runvil`.

The program leaves `workshop.toml` to the person, because a program that rewrites a hand-edited file destroys its comments, turns every click into a git diff, and races the user's open editor.

### The UI saves its state through the server, as opaque JSON in two buckets

`ui/src/services/ui-storage.ts` is the UI's one storage adapter. The user bucket is `GET /user/state` and `PUT /user/state/{key}`, stored in `ui-state.json`, with the keys `editor_settings`, `zoom`, `recent_files`, and `commands_history`. The workspace bucket is `GET /workspace/file/state` and `PUT /workspace/file/state/{key}`, stored in the open `.rvwork` file, with the keys `layout`, `tree`, and `closed_editors`. `StateBucketValue::new` checks each put: the key is on the bucket's list, and the body is JSON of at most 1 MiB. The server stores the text, and the UI owns each value's shape and version. A UI test fails when a source file names `localStorage`.

The UI stores its state on the server because the server's port changes at each launch: the page's origin changes with it, and browser storage starts empty for a new origin. State about the workspace is in the workspace bucket. The owner's reason: "expanded folder state is a property of the workspace not the user account, because the workspace has folders."

### A workspace is one `.rvwork` file

The user opens a workspace, saves it under a new name, and duplicates it from the File menu, through the routes `/workspace/file/open`, `/workspace/file/save-as`, and `/workspace/file/duplicate`. The file stores small data: the granted folders, the window's geometry, and the workspace bucket. The code reserves named siblings beside the file for larger data: `agents`, `runs`, `terminals`, `plugins`, and `index.db`. Two workspace files in one folder share their siblings.

The owner's rule: "Save As should just create the file. no directory ceremony." The design accepts the risk of shared siblings, and the code omits a check for it.

### The `.rvwork` file is a Turso database with three tables, and one task owns its connection

`SCHEMA_V1` creates three tables. `meta` has the format name and the version, `grants` has each folder's path, its position, and the time of its grant, and `kv` has JSON values by key: `window` and the three keys of the workspace bucket. Open accepts a database whose `meta` format is `runvil-workspace` at version 1 and leaves any other file unchanged. One tokio task in `workspace/src/workspace_file/actor.rs` owns the one connection, and callers send it commands over a channel, so the order of commands is the order of writes. The root `Cargo.toml` pins `turso` to `=0.7.2`.

One task owns the connection so that Turso's async API is behind a single writer, and the code that confines file access to the granted folders is synchronous.

### Launch reopens the last workspace, and an unsaved workspace exists in memory until its first Save As

`last-workspace` is a text file with the path of the last workspace file, and `spawn` reopens that file before it reports ready. For a missing pointer the server starts with a workspace in memory, and for a damaged pointer or a file that open refuses it also logs a line. Workshop loses an in-memory workspace at quit, window geometry included. After the first Save As, the server writes each change to the file as it happens.

The rule for a bad pointer is that boot always continues.

### The in-memory set of granted folders decides file access, and the server copies each change to the file

The user grants a folder by a drop on the window, the folder picker, or a workspace file, and `POST /workspace/grant` and `POST /workspace/revoke` change the set. Before each file operation of `GET /workspace/tree`, `GET /workspace/file`, and `PUT /workspace/file`, `workshop-workspace` checks the request path as text (it refuses `..`), then canonicalizes the path and matches it against the granted folders. When a write to the workspace file fails, the server logs a warning and the in-memory change remains.

A failed write is a warning only: the server continues to run, and boot continues. One gap is open: a path can change between the check and the use. An exploit of it needs local write access inside a granted folder, and section 9 lists the work that closes it.

### A directory listing reads a link's own attributes and leaves the target unopened

`directory_listing` classifies a symbolic link or a junction from the link itself: on Windows with `is_symlink_dir`, and on Unix with one `fs::metadata` call that reads the target's type. A link's size is 0, and its time is the link's own.

A listing that read each target's metadata would have two faults. A stale link to an offline share could make the listing miss its 10-second deadline, and on Windows a link to a UNC path would send the user's NTLM authentication to that host. The Windows code leaves the target untouched, for a chain of links as well.

### A `.rvwork` file is complete after a quit

While a workspace file is open, a sidecar named `Name.rvwork-wal` may exist beside it with recent writes. At a graceful shutdown the workspace's registered task calls `close_backing`, which runs `PRAGMA wal_checkpoint(TRUNCATE)` and removes the sidecar. After a crash, the database replays the sidecar at the next open. A user who wants a copy of the file should quit Workshop first. The server skips a checkpoint after each write, because a write-ahead log between quits is ordinary database behavior.

## 8. The UI code: packages, layers, and registries

### The UI is three npm packages: `ui`, `@workshop/platform`, and `@workshop/look`

`crates/workshop` is one npm workspace with three members. `ui` is the app. `@workshop/platform`, in `platform/`, contains the mechanics that every feature uses: events, the `Disposable` lifecycle, the service, command, menu, keybinding, and panel registries, the context-key service, and `WorkshopPart`, the base class of panels. `@workshop/look`, in `look/`, contains visual code: tokens, icons, and views such as the dropdown and the modal ([Workshop: what the user sees](workshop-ui.md)). `ui` imports both, `look` imports its own files, and `platform` imports its own files plus types from `dockview`, the docking library. A `test/boundary.mjs` in each of the two enforces that.

`look` is Workshop's own package, and the Gateway's config UI uses a package of its own, `shared-ui`. The two are separate so that they evolve independently and a change in one leaves the other unbroken.

### The app package has three layers, and `main.ts` connects the parts

`ui/src` has `base/` for path helpers, `services/` for the app's services and their tokens, and `parts/` with one folder for each feature. Parts import services, and services import `base`. `main.ts` is the composition root: it creates the workshop socket and the services, registers most services under a token with `registerService`, and connects socket events to the status bar. A part looks a service up by its token with `getService`. `main.ts` adds every top-level object to one `DisposableStore` and disposes it on `pagehide`.

With the one store, teardown is one `dispose()` call up the tree. The composition root exists so that one place makes the connections between components. The owner's goal: "I want a clean architecture, like neatly routed cables in a datacenter."

### The entry bundle registers every action at startup, and a panel's code loads on first use

The entry bundle is `main.ts` plus the `*.contribution.ts` modules it imports, which `ui/src/parts/workbench.contributions.ts` lists. A contribution module registers its feature's actions, menu rows, shortcuts, and panel type. A panel's implementation is a separate chunk: the panel registry imports the part's `index.ts` when the panel first opens, and that module's `register()` installs the panel factory. A lazy panel's imports are shared code, such as `services/` and `@workshop/platform`.

Actions register in the entry bundle because a feature's `register()` runs when its chunk loads, so an action that registered there would show as a disabled raw id at boot.

### One `registerAction` call declares a command, its menu rows, and its shortcut

`registerAction` in `platform/action-registry.ts` registers the command, one row for each menu the action lists, a command palette row when `f1` is set, and the keybinding rule. A row's `when` expression hides the row, its `precondition` disables it, and `toggled` shows a check mark. The expressions test context keys, named values such as `editorTextFocus` or `activeEditor` that the owning feature sets in the one `ContextKeyService`. `activeEditor` is the type id of the active panel, so a condition for the text editor reads `activeEditor == 'editor'`. `KeybindingDispatcher` listens for `keydown` on the document in the capture phase and runs the command of the matching rule.

The type names, command ids, menu ids, and context-key names are VS Code's, because two editors that reimplement these mechanics from scratch use the same names and shapes. The dispatcher uses the capture phase because a bubbling listener runs after CodeMirror's keymap on the content element, so a chord that both define would fire twice.

### Each feature registers its panel type, and a rule derives each panel's id

`registerPanelType` takes a type id, a title, a default zone of left, main, or right, a `closable` flag, an optional `panelId` function, and the lazy import. The types are `tree`, `editor`, `agent`, `run`, and `config`, and a second registration of one id throws. `panelIdFor` computes a panel's id: the type's `panelId(params)` when the type defines it, else `type:instance` when the open has a string `instance`, else the bare type, which makes the type a singleton. A saved layout whose `version` differs from `LAYOUT_SCHEMA_VERSION`, which is 5, falls back to the default layout.

Each feature registers its own type because a central list of panel types would be a place where `services/` imports feature code. Only the id rule decides whether a type has one panel or many, because a separate flag for multi-instance types could contradict it.

## 9. Planned work

Each of these items is still to be built.

- **A real folder for a run.** A mount of a real directory in a run's file system. Every run starts with an empty in-memory file system, so the `std` file tools see an empty root. Which directory to mount is a product decision ([Files and the VFS](files-and-the-vfs.md)).
- **Runs from the Run window.** A Run button that starts a run, with run storage, run ids, and saved drafts. The Run window posts a prompt's text to `/prompts/contract` (section 5), and its Run button is inert.
- **Saved conversations.** A database-driven system that saves agent memory. A transcript is in memory, and it ends with the server process.
- **A close request and a message queue.** A close request on the agent socket, a shutdown path for conversations, and a follow-up queue with edit and unsend. A conversation ends when its run ends or the process exits, and the server sends each message to the run at once.
- **Remote access to the server.** A server that accepts clients from other machines. `spawn` refuses a bind address outside loopback. The work also closes the gap between the path check and the file operation (section 7).
- **A lock for file writes.** A lock that makes the writes to one file run one at a time. A file write that outlasts its route deadline continues after the 408 and can replace a newer save.
- **`POST /admin/restart` for a profile switch.** A profile switch that restarts the Gateway through that route. The Gateway has the route, and Workshop's switch posts `/shutdown`.
- **Plugins from a workspace.** Library Plugins that the server loads from the `plugins` folder beside a workspace file. The server loads the library Plugins of one directory, which `[agents] plugins` names and the desktop app sets to the `plugins` directory of its installation (section 5).
- **A log subscriber.** A `tracing` subscriber that writes the app's log lines. The Workshop crates call `tracing::warn!`, and the app lacks a subscriber, so the process discards those lines.

## 10. Known gaps

Each item names code that the rest of the code leaves unused, two parts of the code that differ, a path on one developer's machine, or a text that differs from the code.

**The Model menu state**

1. `chat_ready` and `switch_in_flight`. The server computes both flags for each `workbench` frame, and the UI stores them unused. `main.ts` creates a `WorkbenchService` and passes each `workbench` frame to its `applySnapshot`, which copies the flags to `chatReady` and `switchInFlight`. A search of `ui/src` finds the service and the two names in three files: `main.ts`, `services/protocol.ts`, and the service's own file. The chat box offers Send from the session's own `started` flag and blocks it while the model selection is empty (`ui/src/parts/agent/agent-session-view.ts`).
2. `switch_profile`. The server accepts the frame and runs the profile switch of section 6 for it, and the UI lacks a sender. Tests are the only callers of `WorkshopSocket.switchProfile`, and `main.ts` reads one of the six `workbench` fields, `selected`. The Gateway's config panel posts `/admin/switch-profile` through the proxy, a path that bypasses the restart of section 6.

**The server crates**

3. A demo mount. `server/src/agents/language_guide.rs` mounts a directory read-only at `/_runvil/language-guide` in every run when its constant `ENABLED` is true. `ENABLED` is `false`, and `SOURCE` is an absolute path under one user's home folder. The module's own comment calls it a hack.
4. Two public paths. The goal of section 3 is one public path for each item. `workshop-menu` declares `pub mod catalog` and `pub mod menu` and also re-exports `CatalogBus` and `MenuBus` at its root, and `workshop-status` does the same with `pub mod status` and `StatusBus`. The server uses both forms: `server/src/app.rs` imports `workshop_menu::catalog::CatalogBus`, and `server/src/workshop_socket.rs` imports `workshop_menu::CatalogBus`.
5. A method that only tests call. Tests are the only callers of `WorkspaceRoots::subscribe` in `registry/src/traits.rs`.

**Guides, a rule file, a prompt's prose, and a doc comment**

6. Registry wording. `.cursor/rules/workshop-architecture.mdc` says subsystems register into "proxy slots". `Registry` has four collections (section 3).
7. The Workshop guide. `runvil-docs/src/workshop/01-stub.md`, a stub, is the guide's one Workshop page. The guide therefore omits the advice of section 7 to quit Workshop before copying a `.rvwork` file.
8. The Gateway guide. Four of its pages describe a `[workshop]` section of `gateway.toml`. `RawConfig` in `runvil-gateway/crates/gateway/config/src/config.rs` refuses unknown tables, and `workshop` is unknown to it ([The Gateway](gateway.md)).
9. The chat prompt's prose. `agents/chat.md` says a model selection "takes effect on the next run" and describes "one system message". `routed` in `server/src/agents/broker.rs` sends each round to the selected model, and the prompt's `messages:system` line is commented out ([Prompts that ship](shipped-prompts.md)).
10. A doc comment on the deadline code. The comment on `DEADLINE_ELAPSED_CODE` in `support/src/deadline.rs` says "Both UIs" key on the string. The Gateway's config UI lacks the string. Workshop's UI names it as `ErrorCatalog.DeadlineElapsed` in `ui/src/services/error-catalog.ts`, and `httpFailure` in `ui/src/services/workspace-api.ts` sets that code when an answer's status is 408, whatever its body.

**The UI**

11. A label for a missing tool. `ui/src/parts/agent/transcript/tool-labels.ts` has a case for `runvil/user-input/ask`, and the Packs lack a tool with that id.

## 11. Questions for the owner

Each question asks for the reason behind a part of the design.

**The desktop app and the server**

1. Why does the window load the UI from the server, when Tauri can bundle the files into the app?
2. Why does the server send the bundle under hashed names with immutable caching?

**Conversations and runs**

3. Why does a conversation outlast its socket?
4. Why do all runs share one `Host`?
5. Why does `RunBroker` stream a task's `models.loop` rounds into the conversation? Every chain's round has the origin `Chat`, and the wire's `agent_event` omits the task.
6. Why does the server compute `chat_ready` and `switch_in_flight` and accept `switch_profile`, three things that the UI leaves unused?

**State and the workspace**

7. Why does a workspace file have exactly the tables `meta`, `grants`, and `kv`?
8. Which threat does the granted-folder check answer: a page in the webview, a model's tool call, or both?

**The UI code**

9. Why are the app's layers `base`, `services`, and `parts`?

## 12. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64`, `runvil-gateway` at commit `9369a7d`, and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-11 the tiers of section 3, the `with_plugins` text of section 5, and the planned item on Plugins were brought to `runvil` at commit `b0693699d`. Between the two commits, `std` became a library Plugin that the server loads from a Plugin directory, and Workshop gained the crate `workshop-plugin-std`. The rest of the page still describes the commits above.
- **Update.** On 2026-11-12 the two entries of section 3 on tiers and on dependencies were brought to `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness`, and pinned four of them by commit, and `df857f4b1` added the check `harness_pin`. Earlier commits added the crate `workshop-plugin-bashkit`. The text of section 5 on Plugins still describes `b0693699d`: at `df857f4b1` the Plugin directory also has the library of the bashkit Plugin, and the server has a switch, committed as off, that mounts the `C:` drive read-only in every run.

*2026-11-10 16:42 - Claude Fable 5.2*
