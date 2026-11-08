<!-- source: source reads of 20 open-source harnesses and protocols at early November 2026 commits; links inline -->

# How agent harnesses manage tool state

Across twenty harnesses, almost none keeps important state inside tool objects. Tools are thin and cheap to rebuild. The state lives in services scoped to a session, thread, or conversation, a project, or the whole process, and each call gets a small context with ids and handles. Per-run tool state exists everywhere, but usually as session or turn services rather than as per-run tool instances.

On resume, the safe default most harnesses chose is to never run an unfinished call again: they write a synthetic "aborted" or "interrupted" result instead. The harnesses that do run calls again (at least once) leave idempotency to the tool author. Only the OpenAI Agents SDK fails closed, with an explicit "executed" marker written before the first side effect. No harness hands tools an idempotency key, though several tell authors to use the tool call id as one.

## Short answers

- **Is there per-run tool state?** Yes, in every harness. Examples:
  - Codex keeps shells, approvals, and diffs in session and turn services.
  - Zed keeps file read times and edit review state in a per-thread `ActionLog`.
  - OpenHands keeps a shell per conversation.
  - LangGraph creates a run-scoped shell session at run start and tears it down at run end.
  - PydanticAI's `for_run` hook and the OpenAI Agents SDK's per-run computer exist for the same purpose.
  - What is rare is per-run tool objects.
- **Is it per host or per process?** The expensive shared things live above the session:
  - MCP connections are pooled per process (Codex, opencode, Google ADK, PydanticAI) or per project (Zed).
  - Some harnesses keep process-global maps of shells or terminals (Gemini CLI, Continue).
  - Saved approval rules are per user or per project, on disk.
- **What gets stored?** Ten kinds of state recur: live processes, connections, file awareness, edit history, plans, approvals, caches, counters, run identity, and durable memory. The taxonomy below covers each.
- **Do tools remember anything?**
  - Within a run or session, yes: shells, read times, undo history, todos, approvals.
  - Across a restart, only what is explicitly persisted: approval rules, memory files, git checkpoints, todo lists kept in the conversation store, and some caches.
  - Live state (processes, MCP sessions, read tracking, undo history) is rebuilt, never restored.
- **How does this affect idempotency?**
  - Because live state is not restored, re-running a call after a restart runs it against different state. That is why most harnesses refuse to re-run.
  - The ones that do re-run tell tool authors to dedupe with the tool call id or an external key.

## Where tool objects live

| Harness | Tool objects built | Per-call context | MCP connections | Durable tool state |
|---|---|---|---|---|
| [Codex CLI](https://github.com/openai/codex) | fresh registry per model request | session, turn, step, cancel token, call id | process pool, lease per chat | rollout JSONL, exec-policy rules file |
| [Gemini CLI](https://github.com/google-gemini/gemini-cli) | once per `Config`; shallow clone per subagent | abort signal, output updates | per `Config` | chat JSONL, git checkpoints, policy TOML, memory markdown |
| [opencode](https://github.com/anomalyco/opencode) v2 | per project directory; snapshot per request | session, agent, message, and call ids | process pool across projects | SQLite sessions and saved permissions, git snapshots |
| [Cline](https://github.com/cline/cline) | per session runtime; rebuilt on mode switch | serializable: run id, call id, signal | per session (SDK) or extension-wide (VS Code) | session files and SQLite, checkpoint refs |
| [Roo Code](https://github.com/RooCodeInc/Roo-Code) | module-level singletons | the whole `Task` | process singleton | task history, shadow git |
| [Continue](https://github.com/continuedev/continue) | none (plain functions) | IDE, LLM, config, call id | process singleton | session JSON |
| [Zed](https://github.com/zed-industries/zed) | per thread; snapshot per turn | ids, fs, cancel, grants | per project | SQLite threads with results and grants |
| [OpenHands](https://github.com/OpenHands/software-agent-sdk) | per conversation, from a registry of factories | the whole conversation | per conversation | `base_state.json`, events, `TASKS.json` |
| [SWE-agent](https://github.com/SWE-agent/SWE-agent) | executables in a container per attempt | env and state files | none | `.traj` file |
| [Aider](https://github.com/Aider-AI/aider) | none (methods on `Coder`) | none | none | git, history files, tags cache |
| [OpenAI Agents SDK](https://github.com/openai/openai-agents-python) | per agent; two narrow per-run factories | user context, call id | app-owned objects | sessions, `RunState` JSON |
| [PydanticAI](https://github.com/pydantic/pydantic-ai) | per agent; opt-in per-run copy (`for_run`) | deps, call id, retry count | shared and counted; per-run copy for per-user auth | none in core; durable engines |
| [Google ADK](https://github.com/google/adk-python) | per agent; resolved per invocation | session state, call id, services | pooled by a hash of request headers | session events, artifacts, memory |
| [LangGraph](https://github.com/langchain-ai/langgraph) | per compiled graph | state, run context, store, call id | new session per call by default | thread checkpoints, Store |
| [CrewAI](https://github.com/crewAIInc/crewAI) | per agent | none (arguments only) | fresh client per call | LanceDB memory, task outputs, checkpoints |
| [AutoGen](https://github.com/microsoft/autogen) | per agent or workbench | cancel token, call id | one session per workbench | LLM context only |
| [Agent Framework](https://github.com/microsoft/agent-framework) | per agent, merged per run | session, arguments, run kwargs | per agent until close | session dicts, history, workflow checkpoints |
| [Goose](https://github.com/aaif-goose/goose) | per session slot, started lazily; lease per turn | session, call id, lease | per session slot | SQLite session `extension_data`, memory files, `permission.yaml` |
| [MCP servers](https://modelcontextprotocol.io/specification) | per process, per session, or per request (2026) | request context | not applicable | none in the protocol |
| [smolagents](https://github.com/huggingface/smolagents) | per agent, across runs | none | user-owned | none |

The spectrum has three bands:

- **Built once per process or agent and shared:** Gemini CLI, Roo Code, LangGraph, CrewAI, AutoGen, Agent Framework, smolagents, OpenAI Agents SDK, PydanticAI (by default), and Google ADK.
- **Built per session, thread, conversation, or project:** Zed, Cline, OpenHands, Goose, opencode, SWE-agent (one container per attempt), and Aider (one `Coder`).
- **Built per model request:** Codex, opencode's snapshot, and MCP servers under the 2026 revision.

The newest designs, and the ones that were rewritten, all moved toward the second and third bands:

- **Codex:** an extension API where `tools_for_step` returns fresh executors on every step.
- **opencode v2:** "acquire Permission.Service and every other required Location service while the layer is constructed", and only ids per call.
- **Cline:** an SDK that builds per-session plugin copies and passes a serializable per-call context.
- **Goose:** a rewrite that moved undo and process maps out of tools and put durable state in the session row.
- **OpenHands V1:** a registry of factories with one set of instances per conversation.
- **MCP 2026-07-28:** a fresh server per request, with cross-call state as explicit handles.

## What tools store

| Kind of state | Usual scope | Examples |
|---|---|---|
| Live processes: shells, PTYs, kernels, containers | session, sometimes process-global maps | Codex PTY store (cap 64, least recently used pruning); OpenHands tmux pool; Gemini CLI and Continue process-global maps; LangGraph run-scoped shell; Agent Framework `ShellSession` with a single owner; SWE-agent container |
| Connections: MCP, LSP, HTTP pools | process or project pool, or per session, or per call | Codex and opencode pools with leases; Zed per project; OpenHands per conversation; LangChain MCP adapters and CrewAI per call |
| File awareness: read times, staleness checks | thread or session, in memory | Zed `ActionLog.file_read_times`; opencode v1.3 `FileTime`, later removed; Continue CLI process-global read set, never reset; most others have none |
| Edit history: undo, diffs, checkpoints | turn, session, or git on disk | Codex turn diff; OpenHands 10 versions per file; Gemini CLI shadow git; Cline stash refs; Aider commits; Goose removed its undo |
| Plans and todos | session, usually in the conversation store | Goose `extension_data`; LangGraph state; OpenHands `TASKS.json`; Gemini CLI and opencode keep todos stateless |
| Approvals and permissions | call, session, and saved rules | Codex session cache plus a rules file; Gemini CLI policy TOML; opencode SQLite; Goose `permission.yaml`; Zed per-thread grants; Aider in memory; Agent Framework session state |
| Caches: results, tool lists, repo maps | run, session, or process | Cline oversized-result cache; CrewAI opt-in result cache; PydanticAI per-run MCP definitions; ADK tool-list cache; Aider tags cache on disk |
| Counters and limits | run, or wrongly on the tool instance | LangGraph separates run and thread counters; CrewAI and Agent Framework keep counts on shared instances |
| Run identity and credentials | run | PydanticAI per-run MCP copy for per-user auth; Agent Framework per-run headers; ADK pool keyed by headers; Goose `AGENT_SESSION_ID` |
| Durable memory and artifacts | across sessions | Gemini CLI `GEMINI.md`; Goose memory files; CrewAI LanceDB; ADK memory and artifacts; LangGraph Store |

## What survives a restart

- **Survives:** conversations with their tool calls and results (all of them), saved approval rules (Codex, Gemini CLI, opencode v2, Goose, Zed), memory files and stores, git checkpoints, todo lists kept in the conversation store (Goose, LangGraph, OpenHands), and Aider's tags cache.
- **Lost:** live processes, MCP sessions, read tracking (Zed does not persist its `ActionLog`), undo history (OpenHands starts it fresh on resume), and in-memory approval caches (the Codex session cache, Aider's "never prompt" set, opencode v1's rules).
- **The pattern:** durable tool state moves out of tools and into the conversation store, keyed by session or call id. Goose, LangGraph, Google ADK, and OpenHands all do this. Live state is rebuilt on resume, never restored, and nobody tries to serialize a shell.

## Idempotency and replay

What each harness does with a call that started but has no recorded result:

| Behavior | Harnesses |
|---|---|
| Write a synthetic result and never run it again | Codex ("aborted"), Cline ("interrupted"), Zed ("canceled", plus a `replay` hook that "never re-runs the command"), Continue ("canceled"), opencode (aborted, with an `executed` flag), Goose (an expired-lease error) |
| Refuse to continue (fail closed) | OpenAI Agents SDK: `executed` is set "before the first user-code side effect", and resume refuses a call that executed without committed output |
| Run it again (at least once) | Google ADK ("may run more than once"), LangGraph (the node re-executes from its start), OpenHands (unmatched actions run on the next step; after a crash only the first is backfilled), CrewAI (task granularity), PydanticAI under Temporal or Prefect (engine retries), SWE-agent and smolagents (full rerun) |

The supporting mechanisms:

- **Dedupe by call id** exists in a few places:
  - Goose recomputes call status from the transcript and drops duplicate ids.
  - Continue runs a call only from its `generated` status.
  - LangGraph's `create_agent` dispatches only calls that have no `ToolMessage` yet.
  - The OpenAI Agents SDK rejects a reused call id whose arguments differ.
- **Caching by arguments** appears twice: CrewAI's opt-in cache, and Prefect's cache key, which includes `tool_call_id`.
- **No harness generates idempotency keys for tools.** The docs push the job to tool authors:
  - OpenAI: "use that call ID as an idempotency key".
  - PydanticAI with Prefect: "side-effecting tools should be idempotent (for example, by passing an idempotency key)".
  - ADK: "modify the Tool to check for and prevent duplicate runs".
  - LangGraph: derive keys from durable state, or wrap side effects in `@task` so finished work is restored.
- **Approvals are guarded against replay** in four harnesses:
  - Agent Framework honors only approvals that match pending requests it recorded itself, and drops replayed or fabricated ones.
  - Goose expires approvals along with the extension lease.
  - OpenAI reattaches approvals only to the original live agent object.
  - LangGraph stores decisions as resume values per task.
- **The protocol layer gives no guarantee:**
  - MCP's `idempotentHint` defaults to false, and every annotation is an untrusted hint.
  - Under the 2026 revision, a broken stream loses the in-flight request, and the client must re-issue it with a new request id. A non-idempotent tool can therefore run twice unless the server dedupes.

The conclusion: the dominant and safest default is "never run an unfinished call again; write a synthetic result". Every harness that does re-run calls pushes idempotency onto the tool author.

The minimum machinery for a safe resume is a durable record that a call started, written before its side effect. That turns "did it run?" from unknowable into an explicit decision. OpenAI writes that record on purpose. Codex, opencode, and Zed get the same effect by persisting the call before its result.

## Pitfalls seen in the wild

- **Mutable state on shared tool objects leaks across runs:**
  - Roo Code's singleton tools keep `lastSeenPartialPath` and a module-level `approvedTodoList`.
  - CrewAI keeps usage counters on the tool instance, reset only by hand.
  - Agent Framework's `invocation_count` is "never automatically reset", with a docs warning about singleton tools.
  - smolagents keeps `agent.state` and the executor's variables through `run(reset=True)`.
  - The Continue CLI's read-before-edit set is never reset per session.
- **Writing a per-run instance back onto a shared object races.** The OpenAI Agents SDK's `resolve_computer` writes the run's computer onto the shared `tool.computer`, which breaks when two runs share one tool.
- **Shallow clones share nested state.** Gemini CLI's subagent `clone()` is an `Object.assign`, so subagents get their own message bus but share everything underneath.
- **Process-global maps collide when several sessions share a process.** This affects Gemini CLI's shell maps (inferred) and Continue's terminal maps.
- **Stateless tools plus external state files need tests.** SWE-agent's editor undo appends to a temporary copy and appears never to persist.
- **Per-call MCP sessions break stateful servers such as a browser.** CrewAI does this on purpose and LangChain's adapters do it by default. OpenHands pins the legacy session mode because servers keep per-session state, and Codex and opencode never share connections to older-protocol servers.
- **Threads cannot be canceled.** LangGraph, CrewAI, AutoGen, Agent Framework, and smolagents all have sync tools that keep running, and finish their side effects, after a cancel.

## What this means for runvil's per-run Packs

1. **The direction matches where the field converged.** The closest designs are:
   - Codex's extension API: install-time contributors hold plugin-level dependencies, `tools_for_step` makes fresh executors, and there are typed stores per scope.
   - opencode v2: services captured at construction, ids only per call.
   - Cline's SDK: a process registry, a copy of each plugin per session, and a serializable per-call context.
   - Goose: a process manager, lazy per-session instances, a per-turn lease, and a per-call context.
   - OpenHands V1: a registry of factories with per-conversation instances.
   - PydanticAI's `for_run`.

   The Plugin's shared object, a Pack instance per run, and per-call facts map onto these one for one. Confidence: high, since six independent designs reached the same shape.

2. **MCP connections in the Plugin's shared object are well supported, with two caveats.** Codex and opencode pool connections with a lease per chat or project, PydanticAI counts references, ADK pools by identity, and Zed shares per project.
   - Legacy stateful MCP servers keep per-session state, which is why OpenHands chose per-conversation sessions and why Codex and opencode never share older-protocol servers. Let the MCP Pack hold a per-run session for servers that need one. Confidence: medium.
   - Do not store run identity or credentials on the shared connection. Pass them per call, or give each run its own lease the way Codex does. Agent Framework shows how much cleanup per-run credentials on a shared connection require. Confidence: medium.

3. **Per-run state belongs on the Pack instance, never on the Plugin's shared object.** That covers read and staleness tracking (Zed's scope works, and Continue's global set leaks), per-run caches (PydanticAI keeps its MCP definitions cache on the run so replay stays deterministic), and counters (CrewAI and Agent Framework leak them). A factory for a stateless Pack can hand every run the same `Arc`, as PydanticAI's `for_run` returns `self` by default. That is safe only while the Pack holds nothing per run. Never write a run's instance back onto shared state. Confidence: high.

4. **State that a resumed run must see belongs in the run record, not the Pack instance.** Goose, LangGraph, ADK, and OpenHands keep todos and approvals in the conversation store. A Pack instance dies with its run, which is right for live handles such as shells and sessions, and wrong for anything a resumed run needs. Confidence: high.

5. **runvil's recorder already has the marker that makes resume safe.** The Harness writes each effect before its performer starts and each answer before the run resumes. That is the "executed before the side effect" record the OpenAI Agents SDK uses. When resume lands, a tool call with an effect record and no answer should get a synthetic result or a refusal, never a second run. Confidence: high.

6. **Give Packs a stable call id to use as an idempotency key.** Every set of docs above tells tool authors to use the call id. runvil's `ToolContext` lends the tool id, the access, and the origin, and `ToolCallOrigin` names the section, entry, caller, and model but no call id. Lending the run id plus the effect id would give Pack authors the key those harnesses ask for. Confidence: medium.

7. **Plan for a teardown step beyond `Drop`.** OpenHands has `interrupt` and `close`, Codex has `on_thread_stop`, Cline has `shutdown`, and opencode has `dispose`, because background processes and sessions outlive single calls. The per-run Pack plan relies on `Drop` at run end. That covers synchronous cleanup, which is also what the C contract's reserved `RV_PACK_DELETE` maps to. Shutting down an MCP session or a shell gracefully would need an async close hook. Confidence: medium.

8. **A turn-level scope may earn its place later.** Codex (turn state and diff tracker), Zed (a tool snapshot per turn), and Goose (a lease per turn) all keep a scope between the run and the call, mostly for approval waiters and diffs. runvil's nearest equivalent is the section entry. Confidence: low.

## Method and limits

- Six readers each took three harnesses and answered the same seven questions from source. They read code at the commits named in the evidence, all from early November 2026 except Continue (July 2026) and SWE-agent (July 2026).
- Several repositories have moved: opencode to `anomalyco/opencode`, Goose to `aaif-goose/goose`, and the OpenHands backend to `OpenHands/software-agent-sdk`. AutoGen is archived in favor of Agent Framework, and Continue may be winding down.
- Claims the readers marked as inferred are flagged as inferred here. I did not reopen every link myself.
- The MCP 2026-07-28 statements come from the spec pages the reader cited, and adoption of that revision is still early.
- Claude Code is not covered, because its source is not public.

*2026-11-07 21:10 - claude-opus-5.5*
