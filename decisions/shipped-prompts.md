# Prompts that ship: design decisions and their reasons

- Scope: the sample prompts in `prompts` and Workshop's built-in chat prompt
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the place of the sample prompts and the Paths bullet describe `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`, after the Engine and the Harness moved (section 7)
- Paths: code paths start at the root of the `runvil` repository, except the `prompts` folder and the Engine and Harness crates, which are in the `runvil-harness` repository at the same paths

## Summary

Runvil ships six prompts. Five are sample files in the `prompts` folder of `runvil-harness`, the repository of the Engine and the Harness: four small examples and `research-person.md`, which researches a person on the web. The sixth is `chat.md`, the built-in chat agent, which Workshop compiles into its binary and launches when an agent panel opens. Its one section is a loop: it reads the operator's next message, runs model rounds and tool calls until the model answers, and summarizes older turns when a turn overflows the model's context window. Section 1 describes the samples, and sections 2 and 3 describe the chat prompt.

A check of this page against the code found two gaps and left four questions about reasons for the owner. Section 5 lists the gaps, and section 6 lists the questions. In the larger gap, the prose of `chat.md` describes a system message about a mounted language guide, while the Lua line that adds that message is a comment and the mount is switched off.

Section 4 lists the planned work. Its largest item is a Pack ([Tools and Packs](tools-and-packs.md)) that turns a folder of prompts into tools.

## From `chat.md` to a running agent

The built-in chat prompt is the file `crates/workshop/agents/agents/chat.md`. The `workshop-agents` crate compiles the file's text into Workshop as the constant `BUILTIN_CHAT_SOURCE`, and `discover_agents` lists the `.md` files of the agents folder plus the name `chat`.

When an agent panel opens, the UI launches `chat`. The server opens a `Conversation` and builds a `RunSetup` with `has_user` set to true. `Conversation::run` calls `Host::parse`, which reads the text through `load_agent` and checks the three Packs under `uses:`. `Conversation::run` then creates a `Harness` and calls its `run` method ([The Harness and the Host API](harness-and-host.md)).

The run enters the prompt's one section, `## Conversation`. Its Lua block offers tools, defines the function `compact`, and waits in `tasks.receive()`. When the operator sends a message, `Conversation::send_input` calls `RunControl::send`, and `tasks.receive()` returns the text. The block appends the text with `messages:user`, and `models.loop` runs model rounds and tool calls until the model answers in text. The block then waits again. `Conversation::close` cancels the run.

## 1. The sample prompts

### Five sample prompts ship under `prompts`, and a test parses each one

The samples are `hello.md`, `echo.md`, `greet.md`, `analyst-example.md`, and `research-person.md`, and the language guide's first chapter says what each one shows.

The test `every_shipped_prompt_parses_offline` in `runvil-runtime` parses each `.md` file under `prompts`, so a language change that breaks a sample fails the test run. The folder is in `runvil-harness`, with the test that reads it. [The prompt language](prompt-language.md) covers the forms the samples use.

### `research-person.md` runs at most 20 rounds, and the model decides when to stop

The prompt declares `uses: [web]` and `max_tool_iterations: 20`. Its section offers `web/search` and `web/fetch`, sends its prose as the user message, calls `models.loop()`, and returns the model's last reply. The prose asks for a summary of roughly 500 to 600 tokens. The prose describes each tool by its job, and a test in `shipped.rs` fails when a user message contains `web/search` or `web/fetch`. [Tools and Packs](tools-and-packs.md) states the rule.

The round limit is in the front matter because the budget belongs with the prompt that needs it. The length target is loose, so the prompt leaves the length to the model's judgment.

## 2. The built-in chat prompt

### The chat agent is a Markdown prompt that Workshop compiles into its binary

`discovery.rs` embeds the text of `chat.md` with `include_str!`, and a `chat.md` in the configured agents folder replaces the embedded text. Workshop embeds the prompt so that a fresh install always has a working chat.

The chat gate tests in `workshop-server` launch the embedded text, and the Engine test file `models_loop-author-shapes.rs` contains a copy of the prompt's `compact` function (section 3). A change to one copy requires the same change to the other, and the two copies are in two repositories: the prompt in `runvil`, and the test in `runvil-harness`. [Workshop: how the app is built](workshop-app.md) covers discovery and launch.

### The chat prompt offers `std`, `web`, and the extras in its one section

The front matter declares `user`, `std`, and `web`. The Lua block in `## Conversation` offers `std`, then `web`, then `tools.packs.extras()`, the run's Packs that the prompt leaves undeclared. In Workshop those are `mcp` and `subagents`. The tool list of `subagents` is empty, so the third offer adds the two tools of the `mcp` Pack to the chat model's tool list.

The model loop runs in that section, so the block makes the offers there: an offer in the Lua under the prompt's title, the H1, would leave the loop's tool list unchanged. The third offer exists so that the tools of a new Pack among the extras appear in the chat model's tool list without a change to the prompt.

### The chat prompt requires a person and a context window of 32,768 tokens

The front matter declares the `user` Pack. When `RunSetup.has_user` is false, the Harness marks that Pack unavailable with the reason "this run has no user", and the run's first step ends the run. A chat loop needs a person to talk to, so refusing the run up front is the honest behavior.

The front matter also sets `min_context: 32768` for the `default` role. When the model's window is smaller, the first step ends the run with a notice that the role "requires a context of at least" that many tokens.

## 3. The chat prompt's turn loop

### The chat loop reads one message, then runs `models.loop` under `pcall`

Each pass of the `while true` loop appends the operator's next message with `messages:user(tasks.receive())` and calls `models.loop(compact)` inside `pcall`. A failed model round raises an error, the `pcall` catches it, and the next pass calls `tasks.receive()` again. The `pcall` is there so that the chat survives a transport error. A stop raises the canceled error inside the `pcall`, which catches it the same way.

### The chat prompt summarizes older turns when a turn overflows

`chat.md` passes `models.loop` a compactor, the Lua function `compact`, which the loop calls when a round overflows the model's context window ([Model calls and conversations](models-and-conversations.md)). With the default compactor, an overflow would fail the turn, and every later turn would overflow again.

`compact` asks the model through `models.infer` to summarize every record between the system block and the second-to-last user message. It replaces those records with one user record marked `meta = { summary = true }`, and the loop tries the round again.

While the conversation has one or two user messages, the range to summarize is empty, so `compact` calls `compactors.fail` and the turn fails. The compactor example in guide chapter 11 summarizes a first turn that overflows. A comment in the Engine test `the_chat_agent_shape_fails_an_overflowing_first_turn` states why `compact` fails it: "With one user turn there is nothing older to summarize", so the turn "fails rather than summarizing the question it is answering".

The summary request contains at most 1,500 bytes of each record, cut on a character boundary, and 48,000 bytes in all, filled from the newest record backward. When the range exceeds that budget, the request omits the oldest records, an earlier summary included. The bound exists because an unbounded summary request would be close to the size of the list that overflowed, and it would overflow as well. The prompt sizes that budget for its 32,768-token minimum (section 2).

## 4. Planned work

Each of these items is still to be built.

- **Prompts as tools.** A Pack that turns a folder of prompts into tools, and with it the publishing of prompts over the Model Context Protocol (MCP). Workshop runs prompts, and the example `crates/runvil/examples/run-prompt.rs` of `runvil-harness` shows how a Host runs one prompt through a `Harness`. The `mcp` Pack is an MCP client ([Tools and Packs](tools-and-packs.md)).
- **A quiet recovery from overflow.** A change to what the operator sees when the compactor recovers a turn. The operator sees a `Model turn failed` frame before the reply of each such turn.

## 5. Known gaps

The prose of `chat.md` differs from the code in two places.

1. The system message. The paragraph under the prompt's title, prose that the run ignores, says the agent's "one system message says where the Runvil prompting language documentation is mounted". The line that adds the message is a Lua comment, and `ENABLED` in `language_guide.rs`, the switch that mounts the guide in each run, is `false`. The mount's `SOURCE` is a folder on one developer's machine.
2. The model switch. The same paragraph says "a selection change takes effect on the next run". `WorkshopBroker::routed` in `broker.rs` sends each round to the model that the menu names, and the gate test `gate_a_model_picked_between_turns_serves_the_next_round` checks it.

## 6. Questions for the owner

Each question asks for the reason behind a part of the design.

**The sample prompts**

1. Why does Runvil ship `research-person.md`, and what are the four small samples for?

**The chat prompt**

2. Why is the chat prompt's `min_context` 32,768 tokens?
3. Why does `chat.md` declare `std` while the language guide mount is switched off?
4. Why does the chat loop run until a cancel?

## 7. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-12 the Paths bullet and the text on where the sample prompts are were brought to `runvil` at commit `df857f4b1` and `runvil-harness` at `cc51d45`. Commit `adca9db10` of `runvil` removed the `prompts` folder and the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths. The update covers one sentence each in the summary, the entry on the five samples, the entry on the embedded chat prompt, and the first planned item. The rest of the page still describes the commits above. At `runvil` commit `5bc561c14`, the chat prompt also requires and offers the `bash` Pack, and the text on the chat prompt's Packs is still to be brought to that commit.

*2026-11-10 16:41 - Claude Fable 5.2*
