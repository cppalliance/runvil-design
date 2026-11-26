---
produced: 2026-11-26
title: Field survey - where agents store conversation history after compaction and what retrieval tools the model gets (Claude Code, Codex, Gemini CLI, opencode, Amp, Cursor, Aider, Goose, Cline, Roo, OpenHands, Crush, Letta, LangGraph, Deep Agents, ADK, Anthropic and OpenAI APIs)
---

# Post-compaction history storage and retrieval - field survey

Confidence marks: **verified** means a primary doc or source file was read. **secondary** means a third-party writeup or search excerpt. **unverified** means no evidence found.

## Summary table

| Agent or framework | Old history kept? | Where | Model can get it back? |
|---|---|---|---|
| Claude Code and Agent SDK | Yes | `~/.claude/projects/<cwd>/<session>.jsonl` | Yes - summary tells it to Read the transcript path (secondary) |
| Codex CLI | Yes, append-only | `$CODEX_HOME/sessions/YYYY/MM/DD/rollout-*.jsonl` | No (open issue asks for it) |
| Gemini CLI | Unclear | `~/.gemini/tmp/<project>/chats/` | No tool found |
| opencode | Yes | SQLite `opencode.db` | Not built in; community plugin and SDK `history()` |
| Amp | Server-side threads | Amp threads | Yes - `find_thread`, `read_thread` |
| Cursor | Yes | `agent-transcripts/<id>/<id>.jsonl`, plus compaction segment files | Yes in practice - grep and read over files |
| Aider | Yes | `.aider.chat.history.md` | No |
| Goose | Yes | SQLite `sessions.db` | Partial - `chatrecall` extension, off by default |
| Cline | Yes | `tasks/<id>/*.json` | No |
| Roo Code | Yes, tagged | task API history | No (user-side rewind) |
| OpenHands | Yes | `events/event-*.json` | No |
| Crush | Yes | SQLite `crush.db` | No |
| Droid | Yes | `~/.factory/sessions/.../*.jsonl` | Unverified |
| Windsurf Cascade | Unverified | Unverified | Partial - user `@`-mentions a past chat |
| Anthropic API (context editing, compaction) | App holds it | Your app | No built-in; memory tool only if model saved earlier |
| OpenAI Responses compaction | Opaque, provider-side | Encrypted item | No |
| Letta / MemGPT | Yes | Database, `in_context` flag per message | Yes - `conversation_search`, `archival_memory_search` |
| LangChain SummarizationMiddleware | Dropped from latest state | Old checkpoints only | No |
| LangChain Deep Agents | Yes | `/conversation_history/{session_id}.md` on the backend | Yes - `read_file`, `grep`, `glob`, `ls` |
| Google ADK | Yes, events stay | Session store | No tool for raw events |
| Zep / Graphiti | Yes, verbatim episodes | Graph store | App-injected context or `graph.search` |
| Manus | Offloaded to sandbox files | Sandbox file system | Yes - file reads, URL refetch |

## Four storage patterns

1. **Discarded:** CrewAI, smolagents default, AutoGen buffers, Semantic Kernel truncation reducer, Pydantic AI sliding window, LangChain middleware in the latest thread state.
2. **Kept by the app, hidden from the model:** Codex, opencode, Roo, Goose, OpenHands, Crush, Aider, Cline, Google ADK, Anthropic and OpenAI APIs. Dominant design is an append-only log plus a filtered model view.
3. **Kept in a database the agent can query:** Letta, Zep/Graphiti, LlamaIndex memory blocks.
4. **Written to a file the agent can grep and read:** Deep Agents, Claude Code (transcript pointer), Manus, Cursor.

## Per-agent detail - coding agents

### Claude Code (including headless `-p` and Agent SDK)
- **Storage:** Every turn goes to `~/.claude/projects/<sanitized-cwd>/<sessionId>.jsonl`. Multiple compactions share one file. The summary is written as a user message with `isCompactSummary: true` (secondary). A `compact_boundary` entry records trigger and pre-compaction tokens (secondary). `persistSession: false` skips writes. A `SessionStore` adapter can mirror transcripts to a database or object store (verified, Agent SDK docs). Transcripts are deleted after `cleanupPeriodDays` (default 30). `CLAUDE_CODE_TRANSCRIPT_LOCAL_GC=1` drops pre-compaction history from `-p` and SDK transcripts over 5 MB (secondary).
- **Retrieval:** No dedicated history tool. The continuation message tells the model: "If you need specific details from before compaction (like exact code snippets, error messages, or content you generated), read the full transcript at: <path>". The model uses its normal Read tool on the JSONL. Wording comes from an unofficial source mirror, a deobfuscation gist and a traced request (secondary). Open issue 26771 notes the pointer has no line indexing. Open issue 96619: a subagent's summary points at the parent session file, not its own.
- **Hooks (verified):** `PreCompact` (matchers `manual` and `auto`) gets `transcript_path`, `trigger`, `custom_instructions`, and can block. `PostCompact` gets `compact_summary` and cannot block. `SessionStart` with matcher `compact` can inject context. The transcript is written asynchronously and may lag memory.
- **After compaction:** CLAUDE.md, memory, up to 5 recent files and invoked skills are re-injected.
- **Sources:** https://code.claude.com/docs/en/hooks, https://code.claude.com/docs/en/agent-sdk/sessions, https://code.claude.com/docs/en/agent-sdk/session-storage, https://code.claude.com/docs/en/context-window, https://github.com/codeaashu/claude-code/blob/main/src/services/compact/prompt.ts (mirror), https://gist.github.com/sam-saffron-jarvis/9d8e291c4e696ac7948702d6c4884448, https://github.com/anthropics/claude-code/issues/26771, https://github.com/anthropics/claude-code/issues/96619, https://harnessnotes.com/context-compaction

### OpenAI Codex CLI
- **Storage:** Append-only rollout JSONL at `$CODEX_HOME/sessions/YYYY/MM/DD/rollout-*.jsonl`. Compaction appends a `compacted` record with `message` (local compaction only), `replacement_history`, `window_number`. Original turns stay. On resume, `rollout_reconstruction.rs` scans backward for the newest compaction and rebuilds from `replacement_history` plus later items. Remote compaction (OpenAI and Azure Responses) returns one opaque encrypted `compaction` item, so `message` is empty (secondary).
- **Retrieval:** None. Issue 32265 (open) asks for it: "the model can no longer directly retrieve exact details that were omitted". RFC 8573 proposed a pointer to an archive file and was closed for lack of upvotes.
- **Notes:** Local compaction keeps recent user messages up to about 20K tokens. `compact.rs` runs `PreCompact` and `PostCompact` hooks that can stop the turn but not rewrite the summary (read from source). Experimental `token_budget` resets the window with no summary (issue 31822).
- **Sources:** https://raw.githubusercontent.com/openai/codex/main/codex-rs/core/src/compact.rs, https://github.com/openai/codex/blob/main/codex-rs/core/src/session/rollout_reconstruction.rs, https://developers.openai.com/api/docs/guides/compaction, https://github.com/openai/codex/issues/32265, https://github.com/openai/codex/issues/8573, https://github.com/openai/codex/issues/31822, https://rememorio.github.io/blog/codex/en/rollout-recovery/

### Gemini CLI
- **Storage:** Sessions record to `~/.gemini/tmp/<project>/chats/`. Format moved to append-only JSONL in PR 23749 (April 2026, verified). Retention 30 days by default. `/chat save <tag>` writes `checkpoint-<tag>.json` with `{history, authType}`. Whether pre-compression messages stay in the session file is **unclear**: issue 21335 said compressed history was not persisted, fix PR 21345 closed unmerged, a third-party review says summaries are now appended (unverified).
- **Retrieval:** None found. In-memory pre-summary messages are dropped.
- **Notes:** `chatCompressionService.ts` fires a `PreCompress` hook, keeps about the last 30% of history, replaces the rest with a `<state_snapshot>` XML summary, then runs a verification pass.
- **Sources:** https://geminicli.com/docs/cli/session-management/, https://github.com/google-gemini/gemini-cli/blob/main/packages/core/src/context/chatCompressionService.ts, https://github.com/google-gemini/gemini-cli/issues/21335, https://github.com/google-gemini/gemini-cli/pull/21345, https://github.com/google-gemini/gemini-cli/pull/23749

### opencode
- **Storage:** SQLite `opencode.db` in WAL mode. Docs: "Earlier messages remain stored even when they are no longer sent to the model". V2 stores compaction as a checkpoint event and queries start at the latest boundary. Pruning marks tool outputs compacted and shows `[Old tool result content cleared]` in context (secondary).
- **Retrieval:** None built in. SDK `history()` returns the full transcript, `messages()` only active context. Community plugin `rmk40/opencode-session-recall` adds 5 tools such as `recall_get`.
- **Notes:** Keeps about `compaction.keep.tokens` (default 15K) of recent conversation. No shell PreCompact hook, plugins have a `compaction` hook.
- **Sources:** https://opencode.ai/docs/compaction/, https://jczhu.com/blog/opencode-memory-internals/, https://developers.cloudflare.com/agents/harnesses/opencode/, https://github.com/rmk40/opencode-session-recall

### Amp (Sourcegraph)
- **Storage:** Threads are server-side objects with IDs (`T-...`). On-disk or server format not found (unverified).
- **Retrieval:** `read_thread` fetches a referenced thread as Markdown and a second model extracts what is relevant to the prompt. `find_thread` searches by keyword or touched files. `@@` finds threads by title in the CLI.
- **Notes:** `/compact` was removed Oct 2025 in favor of Handoff (drafts a goal-focused prompt and file list for a new thread). A later update says handoff was removed as well. Current advice is start a new thread and mention the old one by URL or ID.
- **Sources:** https://ampcode.com/news/handoff, https://ampcode.com/news/read-threads, https://ampcode.com/news/find-threads, https://ampcode.com/news/stick-a-fork-in-it, https://ampcode.com/guides/context-management

### Cursor
- **Storage:** JSONL transcripts at `~/.cursor/projects/<workspace-slug>/agent-transcripts/<id>/<id>.jsonl`, subagents under `.../subagents/`. Hooks receive `transcript_path` (official CLI changelog). A third-party reference says the transcript is a write-only export and the chat itself lives in an internal DB (`store.db` for CLI, `state.vscdb` for IDE), with summary archives in `store.db` (secondary). Summarization runs in Cursor's backend.
- **Local observation on this machine (2026-11-26):** 1,875 chat directories, about 1.86 GB. Each JSONL line has only `role` and `message.content` blocks (`text`, `tool_use`; no `tool_result`, ids or timestamps seen in a 12-line sample). Two chats have a `compaction/` folder holding `INDEX.md` (columns Segment, File, Turns, Approx bytes, Keywords) and `segment_000.md`. Each segment starts with `# HISTORICAL -- DO NOT EDIT`, says it is a record of a compaction segment from the same task, and instructs "Use read_file or grep to look up details". It holds turn statistics, a curated summary section, then a verbose turn-by-turn render (271 turns, 285 KB and 775 turns, 1.54 MB). At least one later chat grepped a segment file on purpose. The compaction summary text itself did not appear in the saved JSONL, so it is probably injected into live context only (unconfirmed).
- **Retrieval:** Official docs: a local search index over past transcripts for the user. Third-party docs claim conversation search is an agent tool (unverified on cursor.com). `@Chats` attaches a prior chat. In practice the agent can Grep and Read the transcript and segment files.
- **Hooks:** `preCompact` is observe-only (gets trigger, context usage, message counts). Latest user message is kept verbatim. If summarization fails or you press Stop, earlier conversation is kept.
- **Sources:** https://cursor.com/docs/hooks, https://cursor.com/docs/cli/changelog, https://cursor.com/help/ai-features/conversation-search, https://github.com/rimio-ai/rimz/blob/main/docs/externals/agent-adapter/cursor-reference.md, https://docs.arantic.com/cursor/context (third party)


### Aider
- **Storage:** `.aider.chat.history.md` is append-only markdown, written by `InputOutput.append_chat_history()` (verified, `io.py`). Summarization (`ChatSummary` in `history.py`) only replaces in-memory `done_messages`, using the weak model first.
- **Retrieval:** None. The model never sees the file. `--restore-chat-history` (default off) re-parses it into `done_messages` at startup and re-summarizes if over `--max-chat-history-tokens`.
- **Sources:** https://raw.githubusercontent.com/Aider-AI/aider/main/aider/history.py, https://raw.githubusercontent.com/Aider-AI/aider/main/aider/io.py, https://aider.chat/docs/config/options.html

### Goose (Block)
- **Storage:** SQLite `sessions.db` (`~/.local/share/goose/sessions/sessions.db` on Unix). Compaction deletes nothing. `compact_messages` marks old messages `agent_visible=false` (still user-visible), adds an agent-only summary message, then an agent-only continuation message (verified, source). Non-manual compaction preserves the latest text-only user message.
- **Retrieval:** `chatrecall` platform extension, off by default. One tool, two modes: `query` (keyword search across stored sessions with `limit`, `after_date`, `before_date`) and `session_id` (loads first and last 3 messages). Excludes the current session. Caveat: search filters on `agentVisible` rows, so whether compacted messages themselves are searchable is **unverified**.
- **Sources:** https://github.com/block/goose/blob/main/crates/goose/src/session/session_manager.rs, https://github.com/block/goose/blob/58f3cc9e/crates/goose/src/context_mgmt/mod.rs, https://github.com/block/goose/blob/e94f3047/crates/goose/src/session/chat_history_search.rs

### Cline and Roo Code
- **Cline storage:** Per-task folders `tasks/<id>/` with `api_conversation_history.json`, `ui_messages.json`, `context_history.json`. Condense sets `conversationHistoryDeletedRange` so older messages are excluded from requests but not deleted (verified via search excerpt). Newer SDK format keeps a `.compaction.json` sidecar (secondary). **Retrieval:** none, user can use checkpoints or edit an earlier message to rewind. A `PreCompact` hook exists.
- **Roo storage:** Non-destructive. `condense` tags every existing message with `condenseParent` and appends a summary with `isSummary: true`. `getEffectiveApiHistory()` returns the summary onward. Sliding-window truncation tags `truncationParent`. **Retrieval:** none, rewind clears tags (PR 9665).
- **Sources:** https://docs.cline.bot/features/auto-compact, https://github.com/cline/cline/blob/8a6441fd/src/core/task/tools/handlers/SummarizeTaskHandler.ts, https://github.com/RooCodeInc/Roo-Code/blob/main/src/core/condense/index.ts, https://github.com/RooCodeInc/Roo-Code/pull/9665

### OpenHands (Software Agent SDK)
- **Storage:** Event stream persisted as `workspace/conversations/<id>/events/event-<index>-<id>.json` plus `base_state.json`. A condenser emits a `Condensation` event with `forgotten_event_ids`, `summary`, `summary_offset`. `View.from_events()` filters forgotten events and inserts the summary. Full log stays intact.
- **Retrieval:** No model-facing tool found. Default `LLMSummarizingCondenser` keeps `keep_first=4` and triggers at `max_size=120` events.
- **Sources:** https://docs.openhands.dev/sdk/arch/condenser, https://docs.openhands.dev/sdk/guides/convo-persistence, https://arxiv.org/pdf/2511.03690

### Crush, Droid, Windsurf
- **Crush:** SQLite `crush.db`. Summary is an assistant message with `is_summary_message=true`, its ID stored in `sessions.summary_message_id`, context loads from there. Prior messages stay. No retrieval tool found. A `new_session` tool was proposed in issue 2331 (closed, shipping unconfirmed).
- **Droid (Factory):** Local JSONL at `~/.factory/sessions/<cwd-key>/<session-id>.jsonl` with record types `session_start`, `message`, `compaction_state` (`summaryText`, `summaryTokens`, `removedCount`, `anchorMessage`). Secondary. Retrieval unverified. Factory describes "anchored iterative summarization": a persistent structured summary, only newly dropped spans get summarized and merged in.
- **Windsurf Cascade:** Storage unverified. `@`-mention of a past conversation retrieves summaries, checkpoints and queried parts, not the full chat. User-initiated. Cascade is now a legacy agent in Devin Desktop.
- **Sources:** https://github.com/charmbracelet/crush/blob/d855def3/internal/agent/agent.go, https://github.com/charmbracelet/crush/issues/2331, https://factory.com/news/compressing-context, https://docs.devin.ai/desktop/cascade

## Per-item detail - APIs and frameworks

### Anthropic API
- **Context editing (`clear_tool_uses_20250919`, beta header `context-management-2025-06-27`):** App keeps full history. Server clears old tool results just before the prompt reaches Claude and leaves placeholder text. Options: `keep` (default 3), `clear_at_least`, `exclude_tools`, `clear_tool_inputs`. No retrieval tool. Claude re-runs the tool or reads the memory tool. Docs say Claude gets a warning near the threshold so it can save content to memory first. Clearing breaks the prompt cache.
- **Server-side compaction:** Response holds a `compaction` block with a readable summary and signature. App keeps or drops earlier turns. No retrieval tool. Anthropic recommends pairing it with the memory tool.
- **Memory tool:** Client-side. Claude issues `view`, `create`, `str_replace`, `insert`, `delete`, `rename` against a `/memories` path your handler maps to storage. Model reads files back with `view`, but only content it wrote earlier.
- **Sources:** https://platform.claude.com/docs/en/build-with-claude/context-editing, https://platform.claude.com/docs/en/build-with-claude/compaction, https://platform.claude.com/docs/en/agents-and-tools/tool-use/memory-tool

### OpenAI Responses compaction and Agents SDK
- **Responses:** `context_management` with `compact_threshold`, or `/responses/compact`. Output is an encrypted `compaction` item, "opaque and not intended to be human-interpretable". Stateless, works with Zero Data Retention. No retrieval. Do not prune the compact output.
- **Agents SDK:** `SQLiteSession` stores history. `OpenAIResponsesCompactionSession` wraps a session and calls `responses.compact` as history grows. No retrieval tool. Cookbook `TrimmingSession` drops old turns, `SummarizingSession` replaces them with a summary pair (whether raw turns are kept is unverified).
- **Sources:** https://developers.openai.com/api/docs/guides/compaction, https://openai.github.io/openai-agents-python/sessions/, https://developers.openai.com/cookbook/examples/agents_sdk/session_memory

### Letta / MemGPT
- **Storage:** All messages stay in the database after eviction. `conversation_messages` has an `in_context` flag. Evicted messages are replaced by a recursive summary.
- **Retrieval:** `conversation_search` over stored messages (date and role filters) and `archival_memory_search` (semantic, vector store). Archival is not auto-filled from evicted messages, the agent must call `archival_memory_insert`. Known bug: `conversation_search` for `letta_v1_agent` misses agent replies because that text is in `tool_calls` JSON (issue 3222, user report).
- **Sources:** https://docs.letta.com/guides/agents/memory, https://docs.letta.com/concepts/conversations/index.md, https://github.com/letta-ai/letta/blob/bb52a890/letta/orm/conversation_messages.py, https://github.com/letta-ai/letta/issues/3222

### LangChain, LangGraph, langmem
- **SummarizationMiddleware:** `before_model` returns `RemoveMessage(REMOVE_ALL_MESSAGES)` plus summary and kept messages (default last 20). Old messages leave the latest thread state. Earlier immutable checkpoints still contain them (a third-party measurement found 17 of 43 checkpoint rows still held a marker after compaction). No retrieval path. LangChain's guidance is a separate append-only table for the full transcript.
- **langmem:** `summarize_messages` and `SummarizationNode` replace old messages. Guide recommends a separate state key for full history. `create_search_memory_tool` and `create_manage_memory_tool` serve deliberately stored memories, not compacted messages.
- **Sources:** https://github.com/langchain-ai/langchain/blob/master/libs/langchain_v1/langchain/agents/middleware/summarization.py, https://ranjankumar.in/langgraph-context-memory-retention-inversion, https://langchain-ai.github.io/langmem/guides/summarization/

### LangChain Deep Agents (filesystem offload)
- **Storage:** Summarization appends evicted messages as markdown to `/conversation_history/{session_id}.md` on the configured backend. Large tool results and write/edit arguments over 20,000 tokens are offloaded to files too (result leaves a path and a 10-line preview). `state["messages"]` is not rewritten, the summary is tracked in `_summarization_event`.
- **Retrieval:** `read_file`, `grep` (literal match), `glob`, `ls` from `FilesystemMiddleware`. Summary message says: "The full conversation history has been saved to {file_path} should you need to refer back to it for details." A `compact_conversation` tool lets the agent trigger compaction itself.
- **Notes:** Triggers at 85% of window, keeps newest 10%. If offload fails, summarization still runs and the summary has no path.
- **Sources:** https://docs.langchain.com/oss/python/deepagents/context-engineering, https://reference.langchain.com/python/deepagents/middleware/summarization, https://raw.githubusercontent.com/langchain-ai/deepagents/main/libs/deepagents/deepagents/middleware/summarization.py

### Google ADK
- **Storage:** Compaction never deletes events. A compaction event carries the summary in `actions.compaction` (`compacted_content`). Prompt building skips events whose timestamps fall in the summarized range. Triggers: sliding window (`compaction_interval`, `overlap_size`) or token threshold. Ranges never split a tool call from its response.
- **Retrieval:** No tool reads the raw events of a compacted range. A configured `MemoryService` gives `load_memory` (model-driven) and `preload_memory`. Memory must be fed via `add_session_to_memory`. Automatic feed from compaction is unverified.
- **Sources:** https://github.com/google/adk-python/blob/main/docs/guides/apps/events_compaction/index.md, https://github.com/google/adk-python/blob/main/src/google/adk/apps/compaction.py, https://adk.dev/sessions/memory/

### Microsoft frameworks, AutoGen, others
- **Semantic Kernel:** `ChatHistoryTruncationReducer` discards removed messages. `ChatHistorySummarizationReducer` summarizes them into one message. No retrieval.
- **Microsoft Agent Framework:** Strategies (Truncation, SlidingWindow, ToolResultCompaction, Summarization) mark `MessageGroup`s excluded before the model call. Applies only to agents with their own in-memory history. With `CompactionProvider` via `UseAIContextProviders`, stored history is unchanged. No retrieval tool documented.
- **AutoGen:** Model context is a view (`BufferedChatCompletionContext`, `HeadAndTailChatCompletionContext` inserts "Skipped N messages."). Discarded messages may remain in the backing list (code snippet only). No retrieval.
- **Pydantic AI:** History processor output replaces the run history. Provider-native `OpenAICompaction` (encrypted) and `AnthropicCompaction` (readable) put a `CompactionPart` in history. No retrieval.
- **smolagents:** `AgentMemory.steps` keeps everything until `reset()`. No built-in compaction (issue 901).
- **CrewAI:** `summarize_messages` rewrites the list in place with one summary. Old messages not kept. Docs warn summaries can lose exact quotes. `Memory().recall` is a separate store.
- **Mem0:** Stores extracted facts by default (`infer=True`), not a transcript. `search_memories` is a separate layer, not compacted turns.
- **Zep / Graphiti:** Each message becomes a verbatim episode (`store_raw_episode_content` default True) linked to derived facts. `graph.search(scope="episodes")` and `thread.get_user_context()` retrieve. The context block is injected by the app unless wrapped as a tool.
- **LlamaIndex:** FIFO queue in a SQL store. Overflow is archived and passed to memory blocks. `VectorMemoryBlock` retrieves top-k and injects it automatically. No model-called tool.
- **Sources:** https://learn.microsoft.com/en-us/agent-framework/concepts/agents/conversations/compaction, https://microsoft.github.io/autogen/stable/reference/python/autogen_core.model_context.html, https://pydantic.dev/docs/ai/capabilities/compaction/, https://github.com/huggingface/smolagents/issues/901, https://docs.crewai.com/edge/en/concepts/memory, https://docs.mem0.ai/core-concepts/how-it-works, https://help.getzep.com/episodes, https://developers.llamaindex.ai/python/framework/module_guides/deploying/agents/memory/

### Design writeups
- **Manus:** File system in the sandbox is "the ultimate context". Compression is restorable: drop page content but keep the URL, drop document content but keep the path. Agent reads files and refetches on demand. https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
- **Anthropic, effective context engineering:** Compaction, structured note-taking (NOTES.md or memory tool) and sub-agents. Just-in-time retrieval: agent keeps lightweight identifiers and loads with tools (`glob`, `grep`, `head`, `tail`). Tool result clearing is "one of the safest lightest touch forms of compaction". https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- **Factory, evaluating compression:** Anchored iterative summarization scored higher than OpenAI and Anthropic approaches on the vendor's own probe eval. Artifact tracking was weak for all methods. Lost details force re-reads of files. No raw history store described. https://factory.ai/news/evaluating-compression
- **Cognition, Don't Build Multi-Agents:** Argues for sharing full traces and a dedicated compression model. No storage or retrieval detail. https://cognition.ai/blog/dont-build-multi-agents

## Cross-cutting findings

1. **Three retrieval designs exist.** (a) Pointer plus existing file tools: Claude Code, Deep Agents, Cursor segment files, Manus. No new tool needed, just a path in the summary and `read_file` or `grep`. (b) Dedicated search tools over a store: Letta `conversation_search`, Amp `find_thread` and `read_thread`, Goose `chatrecall`, opencode plugin. (c) Nothing: Codex, Gemini CLI, Aider, Cline, Roo, OpenHands, Crush, provider APIs, most frameworks.
2. **Provider-side compaction gives no way back.** Anthropic and OpenAI compaction are stateless. OpenAI's output is encrypted. The app holds the history and the model sees only the summary.
3. **Hooks differ.** Claude Code `PreCompact` can block and add instructions. Codex and Crush hooks can stop but not rewrite. Cursor's is observe-only. Gemini has `PreCompress`. Cline's can add context.
4. **Retention can erase the log.** Claude Code deletes transcripts after 30 days by default, Gemini CLI also 30 days.
5. **Known weak points of the pointer design:** no line index (Claude Code issue 26771), subagent summaries pointing at the parent file (issue 96619), async transcript writes lagging memory, and offload failure leaving a summary with no path (Deep Agents).

## Unverified or not found
- Cursor agent-side search tool and whether its summary text references a transcript path (local files suggest segment files exist, injected text not visible).
- Whether Gemini's session file keeps pre-compression messages.
- Droid, Windsurf and OpenHands retrieval tools.
- Whether Goose `chatrecall` surfaces compacted messages.
- Amp storage format.
- Whether ADK compaction auto-feeds `MemoryService`.
- The Claude Code "read the full transcript at" wording is from a mirror, not Anthropic docs.
- Harness Notes comparison page was used as a cross-check only and was spot-checked, not fully verified.
