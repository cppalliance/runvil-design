<!-- source: source reads of 14 agent harnesses, the Claude Agent SDK, and the Agent Client Protocol at pinned November 2026 commits; links under Sources -->

# How Agent Harnesses Handle User Input, Stop, Queues, and Subagent Steering

This report compares how fourteen agent harnesses handle a message sent while the agent is busy, a stop, queued messages, and messages to subagents, and advises what Runvil's Run Inbox plan should keep and change. The harnesses are Claude Code (with the Claude Agent SDK), OpenAI Codex, opencode, Crush, Pi, pi-subagents, Everruns, the Zed agent (with the Agent Client Protocol, ACP), Goose, Gemini CLI, OpenHands, LangGraph (with Deep Agents), Cline, and Roo Code. Every finding comes from source code read at a pinned commit or a pinned package version, listed under Sources.

## Summary

The field agrees on more than it disagrees, and Runvil's plan already matches most of what it agrees on.

- **A message never kills a running tool by default.** In every harness, a message sent while tools run waits for them to finish and reaches the model at the next model request. Codex and Cline go one step further and cut off the model's reply while it is still streaming, but they too let running tools finish. Tools are killed only when the user explicitly asks for it with a stop or a "send now" action.
- **Every tool call gets a result, and the wording of a made-up result matters.** Everruns once told the model an interrupted call was "cancelled", and the model repeated a send three times. It now says the outcome is unknown.
- **Approvals are a separate channel.** The two harnesses where typed text can approve a pending action, OpenHands and Roo Code, have open bugs about exactly that.
- **Lost messages are the most common bug.** Codex, Pi, Goose, Crush, Gemini CLI, and Everruns all have reports of a message that was accepted and then never reached the model.
- **A finished subagent is resumed from its saved conversation.** Some harnesses keep a finished child loaded for a while, but all of them can rebuild it from its stored messages, and they unload idle children when resources run short.

The plan diverges from the field in one important place: when a message arrives during a tool batch, the plan withdraws the running tool calls. No harness does that for an ordinary message. The recommendations, ranked under Recommendations, are:

1. Let a message cut off only the model's in-flight reply, and let tool calls already issued finish. This also removes the plan's need for made-up "interrupted" tool results. Confidence medium-high.
2. Keep queued messages across a stop, and hand unread messages back to the Host when a run ends, so none is silently lost. Confidence high for the first part, medium for the second.
3. Cap the number of idle tasks, since an idle Runvil task holds a live Lua VM that cannot be rebuilt from its transcript. Confidence medium.
4. Leave room for an approval state in `InputState`, and plan approvals as their own id-keyed channel that a message can never answer. Confidence high.
5. Consider framing a message folded into a running loop with who sent it, and reporting when a chain actually reads a message. Confidence medium to low.

## Background and terms

An **agent harness** is the program around a language model that runs the agent: it sends the conversation to the model, runs the tools the model asks for, and shows the results to a person. A **turn** is everything the agent does in answer to one user message. Within a turn, the harness makes one or more **model requests**, often called model rounds: it sends the conversation and receives a reply. A reply may ask for tools, and the set of tool calls in one reply is a **tool batch**. The harness runs the batch, appends each call's result to the conversation, and makes the next model request. The turn ends when the model replies with text and asks for no tools. The boundary between one model request and the next, right after a tool batch finishes, is where most harnesses let new input in.

Two kinds of message delivery come up everywhere. **Steering** means delivering a message into the turn that is running, so the model reads it at its next request and adjusts. A **follow-up**, also called a queued message, waits until the turn ends and then starts a new turn. Most harnesses offer both.

An **interrupt** or **stop** cuts work short. What it cuts varies: the model's streaming reply, the running tools, shell processes those tools started, or subagents. When a tool call is cut off, many harnesses insert a **synthetic tool result**, a result the harness writes itself, because model APIs reject a conversation in which a tool call has no result.

An **approval**, also called a permission or confirmation prompt, is a question the harness asks the person before running a risky tool, such as "allow this shell command?". It is a **rendezvous**: one specific question that must get one specific answer. That makes it different from "the person's next message", which is the topic of this report.

A **transcript**, called a rollout in Codex, is the saved record of a session. **Resume** rebuilds a session from its transcript after a restart. **Replay** re-runs recorded inputs to reproduce a run exactly.

### The Runvil plan under review

Runvil runs a prompt as Lua code inside a deterministic state machine, and the Host program performs each model call and tool call as an effect. Concurrent work inside one run happens on **chains**: the main chain walks the prompt's sections, and each task started with `tasks.spawn` is a chain of its own. The Run Inbox plan gives every chain an inbox. The user's messages, sent through `RunControl::send`, go to the main chain. A task's messages come only from the chain that spawned it, through `tasks.send`. A delivered message has three possible effects:

- If the chain waits in `inbox.next()` or `input.ask()`, the message resumes it.
- If the chain is inside `models.loop` with a model request or a tool batch in flight, the message interrupts that work. The loop appends the message and runs the next request.
- Otherwise, the message waits in the chain's queue.

A task can hand back a result with `inbox.reply` and stay idle, keeping its Lua VM and conversation, until its owner sends it a follow-up. The Host sees an input state of busy, waiting, or ended. Each delivered message is recorded as an `Event::InboxMessage` at the step that applies it, so a replay delivers it at the same point.

## What the harnesses do

### When a message sent mid-turn takes effect

Table 1 shows what each harness does by default with a message the user sends while the agent works.

*Table 1. Default handling of a message sent while the agent is busy.*

| Harness | Default delivery | Cuts the streaming reply | Cuts running tools |
|---|---|---|---|
| Claude Code | folded into the turn at the next tool-result boundary | only for a `now` message | only for a `now` message or Esc |
| Codex | steered into the turn, read at the next model request | yes, a default-on feature | no |
| opencode | steer at the next step; a queue item starts the next turn | no | no |
| Crush | folded in before the next model request | no | no |
| Pi | steer after the whole tool batch; follow-up when the agent would stop | no | no |
| Everruns | at the next reasoning step | no | no |
| Zed | held until the turn ends; a "Steer" mark ends the turn at the next boundary | only "Send Now", which cancels the turn | only "Send Now" |
| Goose | steer between model requests; a second prompt is refused | no | no |
| Gemini CLI | held until idle; with experimental steering, a hint at the next tool-result boundary | no | no |
| OpenHands | read at the next step | no | no |
| Cline | queued by default; a steer cuts the model request | for a steer | no |

Three patterns stand out.

First, nobody kills a running tool because a message arrived. Cline's code says it plainly: a steer will "Interrupt only the current model request; running tools finish normally." Codex's interrupt marker explains why tool interruption is dangerous: "If any tools/commands were aborted, they may have partially executed." A cut-off file write or shell command can leave the world in a state the model cannot see.

Second, the cost of waiting for tools is real, and harnesses answer it with an explicit stronger action rather than by changing the default. A Zed user waited about an hour for a queued message to land behind a long tool call (Zed issue 57761). Claude Code's "send now" moves tools that can run in the background aside and tells the model "[Still running. ... This call was not interrupted. Its result will be delivered to you on its own ...]". opencode's web app moves blocking shells and subagents to the background when the user steers. Claude Code's `now` priority and Zed's "Send Now" cancel the turn outright.

Third, nearly every harness offers both steering and a follow-up queue: Claude Code through its `next` and `later` priorities, Codex through Tab in its terminal UI, opencode through `steer` and `queue` items, Pi through `steer()` and `followUp()`, and Cline through `steer` and `queue`. Users ask for both, and maintainers keep both. Goose desktop shows what not to do: it stops a run when a short message contains words such as "stop", "wait", or "no", and its matcher would likely also fire on "send it", which contains "end".
### What the model sees after an interruption

The conversation left behind after an interruption decides whether the model recovers well. Harnesses differ in three places: partial text, tool calls without results, and markers.

**Partial reply text.** Claude Code, opencode, Crush, Zed, and Goose keep the text the model had streamed before the cut. Codex, Pi, Everruns, and OpenHands drop it, because they write a reply into history only once it is complete. Gemini CLI rolls back the whole exchange, including the user's message. Cline keeps the visible text but drops the reply's tool calls and reasoning: "A cancelled stream may contain incomplete tool JSON or unsigned reasoning. Keep only replayable visible content from that response." Keeping partial text has a known cost. In opencode issue 45859, the half-finished reply left in context made the model blame itself for the interruption.

**Tool calls without results.** Every mature harness now ensures each tool call has a result before the next model request, and most learned it from bugs. Claude Code had interrupted tool calls without results that broke resume (Claude Code issues 3003 and 1894). Codex fills a missing result with "aborted" each time it builds a request, Pi with "No result provided", and Pi's durable harness with "Tool result unavailable: history ends before this call completed." OpenHands shows the opposite failure: it writes "Tool call interrupted before completion. The conversation was paused." over every call in an interrupted batch, including calls that had already finished (OpenHands SDK issue 6571).

**What the synthetic result says.** This is where the field learned the most. Everruns once told the model an interrupted call was "cancelled - another message came in before it could be completed", and the model repeated a send three times (Everruns PR 4562). Its text now reads: "outcome unknown - the turn was interrupted before this call reported back, so it may have completed; check its effect before repeating it, and do not repeat an action that changes something without checking or asking." Claude Code separates two cases and tells the model nobody refused the call:

- A call cut off to deliver a message: "[Tool call did not complete: the turn was ended to deliver the message that follows. Nothing refused it; re-run it if still needed.]"
- A call that never started: "[Tool call skipped: the turn ended to deliver the message that follows before this call ran. Nothing refused it; re-run it if still needed.]"

Claude Code added the "Nothing refused it" wording after a bug in which a closed approval reached the model "as rejected by the user" (changelog, version 2.1.301). The lesson is that a made-up result should say whether the call ran, whether it may have had effects, and that the user did not refuse it.

**Markers.** Some harnesses add a message to tell the model a stop happened. Claude Code appends "[Request interrupted by user]" or "[Request interrupted by user for tool use]". Codex wraps its marker in `<turn_aborted>` tags ("The user interrupted the previous turn on purpose. ..."), and Everruns appends "User requested to cancel the work." The others add none. A marker can lie about who acted: the OpenClaw host ended Codex turns through the interrupt call, so every turn was recorded with "The user interrupted the previous turn on purpose" when no user had done anything (OpenClaw issue 111019).

When a message is folded into a running turn, Claude Code tells the model what it is: "The user sent a new message while you were working: <message> ... Address the message above as you continue this turn." An earlier version said "IMPORTANT: After completing your current task, you MUST address the user's message above.", and the model answered in text the UI never showed (Claude Code issue 67708). In Crush, Gemini models ignored messages folded in mid-turn (Crush issue 2572). How a folded message is framed changes whether the model notices it.

### Approvals and typed-ahead text

Whether a message the user typed in advance can answer an approval prompt splits the field into two groups.

**Unsafe.** In OpenHands, approving a pending action just means calling `run()` again, and the app sends every user message with `run: true`. So any message the user sends while an approval is pending runs the pending action. In Roo Code, a queued message is sent as "yesButtonClicked" when the agent stops at a tool approval, which approves the tool, and three issues report commands approved this way (Roo Code issues 9720, 12302, and 11389).

**Safe.** Every other harness keeps approvals on a separate channel, matched by an id, so a typed message cannot reach it. Within this group there are two behaviors.

- **Leave the approval open and queue the message.** Cline logs "Leaving pending tool approval open and routing user message as queued follow-up". Codex holds the message until the tool call it is waiting behind finishes.
- **Close the approval and deliver the message, saying the action did not run.** Claude Code answers the model "[Tool call not completed: an approval request was still unanswered when the message that follows arrived and was closed, so the action awaiting approval did not run. Nobody refused it, so this is not the user's decision; ask again if it is still needed.]" Everruns writes "Cancelled because the user sent a message instead". Zed writes "Permission denied: user sent a follow-up message instead of approving the tool call."

LangGraph shows how hard a general rendezvous is. Its `interrupt()` pauses a graph node to ask a question, and the answer comes back with `Command(resume=...)`. Answers are matched by position: an interrupt's id is a hash of the task's namespace and how many answers that task already has. On resume the node runs again from its first line, so everything before each `interrupt()` repeats, side effects included, and a validation loop reprints every earlier answer on each resume (LangGraph issue 7780).

### Telling the Host the agent is waiting

Every harness exposes some state the UI can use to show whether the agent works or waits. Several converge on the same three states:

- Claude Code: `idle`, `running`, `requires_action`.
- ACP version 2: `running`, `requires_action`, `idle`.
- Codex's app-server: `idle`, or `active` with the flags `waitingOnApproval` and `waitingOnUserInput`.
- Gemini CLI: `Idle`, `Responding`, `WaitingForConfirmation`.

Crush and Pi report only whether the agent is busy. Two details matter. Claude Code reports `idle` only after background work has also ended, and calls it the "authoritative turn-over signal". And users want to know when a queued message was actually read, not only when it was accepted. That is an open request against the Claude Agent SDK (TypeScript SDK issue 423). pi-subagents tracks it per message: "queued" means the child accepted the message, and "delivered" means a matching user message appeared in the child. Everruns answers "delivered" as soon as a send is accepted, which its own code shows means "accepted, not read".
### Queues, and how messages get lost

A message that the harness accepted and then dropped is the most frequent defect in this survey. It shows up in four situations.

- **On interrupt.** Codex drops steers it has accepted but not yet sent to the model when a turn is interrupted (Codex issue 45967, open). Its terminal UI copes by putting them back in the input box: "The server has already discarded pending input by the time the interrupted turn reaches the UI, so any unacknowledged steers still tracked here must be restored locally." Crush clears its whole queue on cancel. Pi strands a follow-up queued during a run that then aborts (Pi issue 10017, open). Gemini CLI does the reverse and resubmits a pending hint as a new turn after the user cancels (Gemini CLI issue 26133).
- **At the end of a run.** Goose discards unread steers when a run ends, so a steer that arrives after the last drain is lost. Everruns fixed a message sent just as the session went idle that was never answered (Everruns PR 4290).
- **On restart.** Crush, Pi's core agent, Zed, Goose, and Gemini CLI keep queued messages only in memory. Codex added a durable queue after users reported losing theirs (Codex issues 32715 and 25268).
- **By merging.** Claude Code and Gemini CLI join several queued messages into one prompt at the end of a turn, and users complain they lose the separation (Claude Code issue 73661).

Two designs avoid most of this. Everruns closes a turn's inbox in one atomic step with the check for pending input, so "A send is therefore always either steering for this turn or the start of the next one, never lost between the two." opencode stores its inbox in SQLite and delivers an item by deleting its row and writing the message in the same database transaction, so a restart loses nothing.

Harnesses also differ on where a delivered message is recorded. opencode and Crush record it where the model read it, not where it was typed, and Codex records only steers that reached the model. Everruns, which like Runvil is built as a pure state machine with effects at the edge, records every input as an `input.message` event and rebuilds history from the event log.

### Steering and resuming subagents

**Delivery to a running child** follows the same boundary rule as user input in every harness that supports it. Claude Code's `SendMessage` appends to the child's own pending list, drained at the child's next tool round. Codex, opencode, Crush (through its new `agent_send` tool), pi-subagents, and Everruns all deliver at the child's next model request. Claude Code users complained that a child saw steering only after its whole tool batch (Claude Code issue 43124), and since version 2.1.317 Claude Code moves the child's running command to the background so the message arrives sooner. Codex's newer multi-agent tools split two intents: `send_message` "Does not trigger a new turn", while `followup_task` starts a turn on an idle child.

**Labels.** Harnesses that steer children say who is speaking:

- Claude Code: "The coordinator sent a message while you were working: ... Address this before completing your current task."
- pi-subagents: "Mid-run steering from the parent orchestrator: ... Incorporate this guidance at the next safe point. Do not restart the task unless the guidance explicitly asks you to."
- Deep Agents: "This is not a message from the user: don't treat it as their answer to any question."
- Claude Code's child prompt says parent messages "never count as the user's approval".

**Resuming a finished child.** Every harness that resumes a finished child can rebuild it from its stored conversation, because for a model-driven loop the conversation is the state. Claude Code keeps a finished child in memory for a short time, then evicts it and rebuilds it from its saved transcript on the next message. Codex's newer tools keep finished children loaded until it needs their slot, then unload one and rebuild it from its rollout when a follow-up arrives; with nothing to unload, a new spawn fails with `AgentLimitReached`. opencode, Crush, Goose, OpenHands, and pi-subagents always rebuild from stored messages. Zed continues a child only while its session is still in memory, and otherwise fails with "Subagent session {session_id} was released".

**Every continuation should notify the parent.** opencode keeps "One observer per job generation, including continuations of the same child". Everruns has the opposite bug: a finished child stays marked finished when it is continued, so the parent gets no new notice. The October Runvil subagent study found the same flaw in an older Codex version.

**Interrupts and children.** Codex interrupts do not reach children; in one report a recursive tree grew to 469 agents that interrupts "did not cascade" to (Codex issue 42032). Crush leaves detached subagents running when the parent's turn is cancelled. Zed asks every running subagent to cancel when the parent cancels.

### Recording and replay

Everruns is the closest match to Runvil's architecture. A turn is an event log plus a serializable state, moved by pure transitions, and resume folds the log, so an answer already logged is completed without running the model or tools again. LangGraph shows the cost of the other approach, which rebuilds state by running code again. Node code must call `interrupt()` in the same order every time and repeats every side effect before each interrupt. Runvil's plan to record each delivery as an event and re-deliver it at the same step on replay follows the Everruns approach.

## Where the Run Inbox plan stands

The plan already matches the field, or improves on it, in these places:

- **Delivery timing.** Per-chain inboxes and delivery at the next model request match Claude Code, Codex, opencode, Crush, Pi, and Everruns.
- **Partial replies.** An interrupted model request leaves nothing in `messages`. That matches Codex, Pi, and Everruns, and avoids opencode's self-blame bug.
- **Finished results.** Finished tool calls keep their real results, which avoids OpenHands issue 6571.
- **No stranding at loop end.** `models.loop` checks the queue once more before it returns, which closes the same race Everruns closes atomically.
- **No merging.** Each queued message is appended as its own user record.
- **Approvals.** `input.ask()` means "the next message", and approvals are out of scope, so a message cannot answer an approval today.
- **Recording.** Deliveries are recorded as events at the step that applies them, as Everruns does, and nothing is rebuilt by running code again, which avoids LangGraph's replay problem.
- **Input state.** `InputState` is a current state the Host can always read, not a stream of notices it might miss, which matches Codex and ACP.
- **Owner checks.** `tasks.send` reuses the existing owner check, so a chain can steer only tasks it spawned.

It differs in these places:

- **Tool withdrawal.** A message that arrives during a tool batch withdraws the running tool calls. No harness does that for an ordinary message.
- **Made-up results.** Cut-off calls get the result text "interrupted by a new message", which does not say whether the call may have run or that nobody refused it.
- **Unread messages.** The plan does not say what happens to queued messages on `stop_round`, or to messages the Host sent that the run never read before it ended.
- **Idle tasks.** An idle task keeps its Lua VM with no limit on how many may pile up. Unlike the other harnesses, Runvil cannot rebuild an evicted task from its transcript.
- **Read receipts.** Neither the Host nor a parent learns when a message was actually read.
## Recommendations

The recommendations are ranked by how much they change the outcome for a user, then by how strongly the field agrees.

### 1. Let a message cut off only the model's reply, never a tool call

When a message reaches a chain inside `models.loop`:

- **While the model's reply is still streaming,** withdraw that model request, append the message, and make the request again. This is what the plan already says for a model request, and what Codex and Cline do.
- **While a tool batch runs,** including calls still waiting for a free slot, let every call finish and record its real result. Then append the message before the next model request. This is what Claude Code, opencode, Crush, Pi, Everruns, and Goose do.

To stop a long tool, the user presses stop, which is `RunControl::stop_round` today, and then sends. That pair is the equivalent of Claude Code's `now` and Zed's "Send Now". A single "stop and send" call can come later if users want one.

This makes the plan simpler. A message never leaves a tool call without a result, so step 3 no longer needs its rule for recording an interrupted tool batch, the "interrupted by a new message" text, or the exception for tools that survive stops. The cost is that a message can wait behind a slow tool. Zed issue 57761 shows how long that can be, and the stop button is the answer the field settled on.

Confidence: medium-high. Every harness behaves this way for an ordinary message. The risk is user frustration with slow tools, which only use will show.

If the plan keeps tool withdrawal instead, reword the made-up results along the lines of Everruns and Claude Code:

- For a call cut off while running: "Stopped before it finished, because a new message arrived. It may have partly run; check its effect before repeating it. Nobody refused it."
- For a call that never started: "Not run, because a new message arrived first. Nobody refused it; run it again if it is still needed."

### 2. Never silently drop a message the Host sent

- **Keep queued messages across a stop.** `stop_round` should drop work in flight and leave every inbox untouched. The next read then gets the queued messages, as in Claude Code, Cline, and opencode with `resume=true`. Pin this with a test. Codex issue 45967 shows what happens without that rule.
- **Hand unread messages back when the run ends.** A run can end, by finishing, failing, or being cancelled, while it still holds messages the Host sent that the main chain never read. Some are still in `RunControl`'s buffer, and some are in the main chain's inbox. Return them on the run's report, for example as a `RunReport.unread: Vec<String>` field. The Workshop can then put them back in the chat box, as Codex's terminal UI does. `RunControl::send` returning `Ok` should be documented as "accepted", not "read".

Confidence: high for keeping messages across a stop, since every lost-message bug in the survey follows from not doing it. Medium for the report field, since the exact shape is a design choice.

### 3. Cap idle tasks

An idle task in Runvil keeps a live Lua VM. Other harnesses can unload an idle child and rebuild it later from its conversation; Codex does exactly that when it runs out of slots. Runvil cannot, because a Lua VM's state is more than its conversation. Without a cap, a model that spawns and abandons many resumable tasks grows memory with no bound.

Add a limit such as `RunLimits.max_idle_tasks`. When a reply would exceed it, end the task that has been idle longest, as abandoned with the reason "too many idle tasks". Its owner's next `tasks.send` then fails with `task_ended`. The simpler alternative is to make `inbox.reply` raise when the cap is reached, but that puts the error in the child, which did nothing wrong.

Confidence: medium. The need is clear; the right default and the choice of which task to end need a decision.

### 4. Leave room for approvals now

Approvals are out of scope for this plan, but `InputState` is a public type the Workshop will match on. Mark it `#[non_exhaustive]` so a `RequiresAction` state, as in Claude Code and ACP, can be added without breaking hosts.

When approvals are designed, make each one its own effect with an id, answered only through its own call. A message arriving while an approval is pending should close the approval and deliver the message. The pending tool never ran, so the made-up result can say so: "Not run: a new message arrived while this action waited for approval. Nobody refused it; ask again if it is still needed." That is the Claude Code, Everruns, and Zed behavior. It fits recommendation 1, since the tool had not started, and it lets the message land at once instead of waiting behind an unanswered prompt. Never let a message count as approval, which is the OpenHands and Roo Code bug.

Confidence: high for `#[non_exhaustive]` and for keeping approvals separate. Medium for closing rather than leaving the approval open, where Cline and Codex choose the other way.

### 5. Consider framing folded messages and reporting reads

- **Framing.** When `models.loop` appends a message because it arrived during the loop, wrap it in one short line that says it arrived mid-work and who sent it. On the main chain: "The user sent this message while you were working:". On a task chain: "The task that started you sent this message while you were working:". Messages that a prompt reads with `inbox.next()` or `input.ask()` stay unwrapped, since the author places them. This reverses the plan's decision that `tasks.send` text is appended exactly as given, but only for messages folded in mid-loop. Claude Code, pi-subagents, Codex, and Deep Agents all frame such messages, and two bug reports show models missing or misreading unframed ones. Confidence: medium-low, because the evidence is about framing in general, and Runvil authors may prefer full control.
- **Read reports.** Emit an event when a chain actually takes a message, whether a reader returns it or `models.loop` appends it. The Workshop can then show a sent message as queued until it is read, and a parent can tell a steer landed. The plan's `Event::InboxMessage` already records delivery, so this is a second, small event. Confidence: medium-low. It is cheap, and the evidence is one open SDK request plus pi-subagents' receipts.

### 6. Not now: a separate follow-up queue

Nearly every harness offers both steering and a follow-up queue that waits for the turn to end. In Runvil terms, a follow-up would be a message that `models.loop` does not fold in and that only the next `inbox.next()` or `input.ask()` returns. The plan deliberately has one `send`, and nothing in the evidence forces a second mode now. Revisit it if users ask to type ahead without redirecting the current work. Confidence: low that it is needed soon.

## What not to copy

- **Typed text that approves an action.** OpenHands and Roo Code.
- **Keyword guessing of a stop.** Goose desktop stops a run when a short message contains words such as "stop" or "wait".
- **Merging queued messages** into one prompt. Claude Code and Gemini CLI.
- **Dropping accepted steers on interrupt.** Codex, Crush, and Pi.
- **Overwriting finished tool results** with an "interrupted" error. OpenHands.
- **Labeling a host's interrupt as the user's.** Codex as driven by OpenClaw.
- **Reporting "delivered" when the message was only accepted.** Everruns.
- **Broadcasting every user hint to every running subagent.** Gemini CLI, which also delivers the hint to the parent again.
- **Rebuilding state by running code again.** LangGraph, where every side effect before an `interrupt()` repeats on resume.

## Method and limits

Six researchers read the harnesses in parallel on 2026-11-08, each covering two to four harnesses against the same six questions: input while busy, interrupts and their aftermath, waiting for input and approvals, queues, subagent steering and resumption, and recording. They read source code at pinned commits and cited file and line, quoted the strings the model and user see, and marked each claim with a confidence level. Claude Code is closed source, so its evidence is the published npm bundle of version 2.1.319, its public changelog, a third-party tracker of its prompts, and its issues. This report builds on the October Runvil study "How agent harnesses implement the subagent task tool" (2026-10-10), which covered spawning, status, and cancel, and does not repeat it.

The limits:

- No harness was run. Behavior comes from code and documentation, and some claims are marked as inferences.
- LangGraph's server code for its four strategies for handling a new message while a run is active (reject, enqueue, interrupt, rollback) is not public, so that part rests on its documentation.
- A few absence claims, such as "no harness code sets this state", rest partly on GitHub code search, which can miss files.
- Harness code changes fast. Two findings of the October study no longer hold: opencode replaced its `task` tool, and Crush can now message a running child. Every statement here holds as of its pinned commit.

## Sources

Repositories, each pinned at the commit read on 2026-11-08 unless noted:

- Claude Code: npm `@anthropic-ai/claude-code-linux-x64@2.1.319`; https://github.com/anthropics/claude-code at `0ed94698aa0aeee471465fc9c867c2e8434c4d68` for the changelog; prompt tracker https://github.com/Piebald-AI/claude-code-system-prompts at `d2fd553cdcf9ce7fb525604b5bb6d749d8e71daf`.
- Claude Agent SDK: https://github.com/anthropics/claude-agent-sdk-python at `2209f2c134fee914c0f75733b631a5fc238ca846`; TypeScript typings from https://unpkg.com/@anthropic-ai/claude-agent-sdk@0.3.319/sdk.d.ts.
- Codex: https://github.com/openai/codex at `748ab55ca7de0b647d9a7a9d9a8e70e00f28666d`.
- opencode: https://github.com/anomalyco/opencode, `dev` at `caf8d6a554ecd988767c08f1f15508849eff8728`.
- Crush: https://github.com/charmbracelet/crush at `8f1f02951cabc30b82e497fd916ca434bad1e6f4`; fantasy https://github.com/charmbracelet/fantasy at `ef61b440e05fcd7c76e3e76b12f1b4a3b15e88d6`.
- Pi: https://github.com/earendil-works/pi at `ccbb224c28a17abff4a8ef4f1828dc22cfa5bd04`.
- pi-subagents: https://github.com/nicobailon/pi-subagents at `3ff4f75545edf2695d5ea82bf46a1ea9ef2008a8`.
- Everruns: https://github.com/everruns/everruns at `d65bae87b536be6cac7293a7f92aa0ba7f5a1c3e`.
- Zed: https://github.com/zed-industries/zed at `b5cd310b3b54dfc659fcc06453e6358843dfe216`; Agent Client Protocol https://github.com/agentclientprotocol/agent-client-protocol at `7441d106a853682cce61d2dd7db54123e4b9a477`.
- Goose: https://github.com/aaif-goose/goose at `5bad61a87c27c313aec5187bfb72e69b594086b5`.
- Gemini CLI: https://github.com/google-gemini/gemini-cli at `62bab5b232c15b925d31e655ce844ab939c882b7`.
- OpenHands: https://github.com/OpenHands/software-agent-sdk at `e6569442a23951f852ead8514ae51dff1b377f37`; app https://github.com/OpenHands/OpenHands at `cc99322a1efd0f4be8d6934ae93a4d1d4bc33e05`.
- LangGraph: https://github.com/langchain-ai/langgraph at `e5e37b5d70dbfd93c7006bd90a658c6179a95946`; Deep Agents https://github.com/langchain-ai/deepagents at `4182b6f89ddc8236159e0b51e76eeb3c8ca01eb8`; double-texting documentation at https://docs.langchain.com/langsmith/double-texting.
- Cline: https://github.com/cline/cline at `94eecbbe1a2f899c66b118d13656651b0a884e1f`.
- Roo Code: https://github.com/RooCodeInc/Roo-Code at `b867ec9145750d0ae1ff7f02d35406e9bf2a0b16`.

Issues and pull requests are cited by number in the text, against the repository of the harness named beside them. The per-harness evidence, with file-and-line citations for every claim, is in six research notes produced 2026-11-08 and titled by harness group, for example "OpenAI Codex codex-rs - user input during a turn, ... - source evidence".

*2026-11-08 06:05 - Claude Opus 5.5 (Cursor agent)*
