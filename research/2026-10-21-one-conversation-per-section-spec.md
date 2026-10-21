---
produced: 2026-10-21
title: "One conversation per section: implementation spec - messages list, no constructor, rendered catalog attached at first render, compaction staging list and swap, validation, migration, acceptance tests"
---

# One conversation per section

- Code: `runvil` at `b1bbe33eb`, `runvil-docs` at `62ae759`.
- Paths: `lua/` is `crates/runvil-internal/lua/src/`, `engine/` is `crates/runvil-internal/engine/src/`, and chapter files are in `runvil-docs/src/language/`.
- Status: settled means Vinnie decided it; proposed means a suggested answer he has not ruled on; open means several answers remain, and the one given is the suggestion; resolved means today's code already behaves as required.
- Drift: later commits remove `tools.allow_tasks`, the model's task tools, and task notices, so the items that mention them apply only at `b1bbe33eb`.

## The rule

Each section has exactly one conversation, the `messages` list. The Engine creates it, empty, when it creates the section's Lua VM. Only the Engine creates lists, so every list stays in its section. Sections clear history by design, and a prompt that wants to keep history stays in its section. The rendered tool catalog attaches to `messages` the first time it renders and stays attached for the rest of the section, including after a model change. Any attempt to change it after that is a hard error. Compaction is the one place where history is replaced. The Engine hands the compactor a second, empty list, and the compactor fills it and returns it. The Engine checks the returned list and swaps its records into `messages`.

## Why

Only `models.loop` sends a list and a tool catalog, because `models.infer` is a single round with one user message and an empty tool list. Tasks and fan-out arms enter their own sections, each in a new VM, so each gets its own list. Only two real prompts create a list today, the Workshop chat agent and the research-person prompt, and each creates one, so the rule matches how prompts already work.

Source: `engine/execute/scheduler/dispatch.rs:242-250`, `engine/execute/scheduler/walk.rs:158-173`, `crates/workshop/agents/agents/chat.md:30`, `prompts/research-person.md:40`.

## The Lua surface

| Concern | Today | After | Status |
|-|-|-|-|
| The list | `messages.new()` returns a new list, any number per section | `messages` is the section's list | settled |
| `messages.new` | The constructor | Raises an error that points to `messages` | settled; wording proposed |
| Builders, `replace`, views, `#`, `pairs`, `ipairs` | Methods on each list | The same methods on `messages` | proposed |
| Scratch records | A second list | Plain Lua tables in record shape | settled |
| Assigning the `messages` global | Allowed | Raises | proposed |
| `models.loop` | `models.loop(list, compactor?)` | Runs over `messages` | settled; signature open |
| Compactor | `compactor(reason)`, which must raise | `compactor(old, fresh, reason)` fills and returns `fresh` | settled shape; arguments proposed |
| `compactors.fail` | The default; raises `context_exhausted` | The same, with the new arguments | proposed |

The suggested `models.loop` signature is `models.loop(messages, compactor?)`, which refuses any other first argument with a message that names `messages`, so existing calls change by one rename. The alternatives are `models.loop(compactor?)` and an options table such as `models.loop{ compactor = f }`. `handle:loop` follows whichever form is chosen.

Source: `lua/messages.rs:37-49`, `lua/globals.rs:3-12`, `lua/models_loop.rs:114-131`, `lua/__impl_coro.lua:236-238`.

## Engine behavior

Today `install_messages` installs a table holding `new`, and injection calls it once per VM, including the H1 section and every task and fan-out arm. After the change, it creates one `MessageList`, keeps a Rust handle to it on `SectionVm`, and installs the list as the `messages` global. The Engine reaches the list only through that handle. The `_G` guard owns the global, as it already owns `argv` and `prose`.

The first `models.loop` round in a section renders the catalog and attaches it to `messages`. Today `prepare_chat` rebuilds the tool scope on every round, from the effective tools plus the task built-ins, and records the advertised names on the chain. After the change it renders once, on the first round, and stores the schemas and the advertised names in the list's state, beside the send state that `commit` keeps. Later rounds send the stored schemas byte for byte, and the dispatch gate checks each model call against the stored names. The model binding is resolved before rendering, and a later model change keeps the stored catalog.

Once the catalog is attached, any offer that would change it raises at the offer call. Three details are proposed. An identical re-offer does nothing. A first round with nothing offered attaches an empty catalog, so a later offer raises like any other change. Calling `tools.offer_local` or setting `tools.allow_tasks` after first render counts as a change. Where the error fires, and what the stored catalog holds, are open questions in the tool offering design.

`models.infer` leaves `messages` and the catalog alone. It sends one user message with an empty tool list and skips the precheck, so it stays the side call a compactor uses to summarize. A model task's notice is pushed onto the owner section's `messages` at the owner's next round.

The run log gains an entry at first render and one at each compaction (proposed). Each logged chat round keeps only the advertised tool names, so the first-render entry holds the schemas and lets a replay check the bytes sent. Each compaction entry holds the reason and the records before and after. Compaction runs as Lua plus recorded `models.infer` results, so a replay reproduces it with the Plugins unloaded.

Source: `lua/messages.rs:37-49, 174-188`; `lua/vm/inject.rs:150`; `lua/globals.rs:3-12`; `engine/execute/scheduler/chat.rs:119-124, 133-168, 193`; `lua/vm/state.rs:128-135`; `engine/execute/scheduler/dispatch.rs:242-250`; `engine/execute/run/effect.rs:236-268`.

## The compaction contract

Today the only shipped compactor is `compactors.fail`, and neither real prompt passes its own. A compactor receives only the overflow reason, and one that returns raises "replacement compactors are deferred".

1. The loop detects overflow, either from the precheck before dispatch or from the provider's context-overflow reply. Both arrive at a round boundary, after the last batch's tool results are pushed, so every tool call in the list has its result.
2. The Engine builds an empty staging list.
3. The Engine calls the compactor with the staging list. Proposed: the call is `compactor(old, fresh, reason)`, where `old` is a read-only view of `messages`, `fresh` is the staging list, `reason` is `"precheck"` or `"provider"`, and writes through the `messages` global raise until the compactor returns.
4. The compactor fills `fresh`. It may call `models.infer`, for example to summarize old turns. Proposed: `models.loop` raises inside a compactor. Open: whether it may call `tools.call`; the suggestion is yes, because tool calls are recorded and replay still works.
5. The compactor returns `fresh`. Proposed: returning nil gives up and raises `context_exhausted`.
6. The Engine checks the returned list against the validation rules. Any failure leaves `messages` as it was.
7. The Engine swaps the record vectors. `messages` now holds the compacted records and keeps its identity, so Lua references stay valid and the catalog stays attached. The staging list now holds the old records.
8. The Engine writes the old and new records to the run log. Proposed: `old` and `fresh` become dead handles, and any access raises.
9. The loop retries the round that overflowed, with the attached catalog unchanged.

The existing list supports the swap. `replace` already builds a new record vector inside the same list, and `commit` compares each request with the last by content, so the send state stays correct. After a swap the precheck falls back to a full token estimate, because its anchor matches only the messages it measured. A view held from before the swap keeps reading its original record, because a view holds the record itself.

A compactor can already call `models.infer` and suspend, as an Engine test shows, so returning `fresh` is the new part. The Workshop chat agent wraps `models.loop` in `pcall`, so a failed compaction ends the turn and the conversation continues.

Source: `lua/compactors.rs:117-126, 171-219`; `prompts/research-person.md:42`; `lua/models_loop/machine.rs:33-36, 196-200, 249-257, 320-341`; `lua/__impl_coro.lua:236-238`; `engine/execute/scheduler/chat.rs:176, 294-303`; `lua/messages.rs:163-188`; `lua/messages-view.rs:18-28`; `engine/execute/tests/models_loop_contract.rs:276-299`; `crates/workshop/agents/agents/chat.md:34`.

## Validation rules

| Rule | Reason | Status |
|-|-|-|
| The compactor returns the staging list itself | Returning `old` or another value is an author bug | settled |
| Every assistant tool call has its result right after it | Providers reject an unanswered call (`lua/projection.rs:14-16`) | proposed |
| System records appear only in the leading block | The provider's message shape (`lua/messages.rs:61-73`) | proposed |
| The leading system records equal those of `old` | The system prompt holds the model's instructions | open |
| The list passes the precheck | Otherwise the retry overflows again | open |
| The list's token estimate is below that of `old` | A larger history can only overflow again; Cline applies the same rule | proposed |

Breaking any of the first four rules is an author bug, so it raises kind `lua` at the `models.loop` call, where `pcall` can catch it. Breaking either of the last two means the conversation cannot fit, so it raises the existing `context_exhausted` kind with a message naming the window, the count, and the reserve.

## Problems in today's code

| Problem | Fix | Status |
|-|-|-|
| Once `messages` is the list, `messages.new` reads nil through `__index`, so an old call fails with "attempt to call a nil value" (`lua/messages.rs:409-415`) | `__index` answers the key `new` with the guiding error | settled; wording proposed |
| The retry counts against `max_tool_iterations`, because `next_round` counts rounds (`lua/models_loop/machine.rs:233-237`) | Leave the retry uncounted, and compact at most once per round | proposed |
| A compactor returns only system records, or nothing | Reject a list with nothing beyond its system records, unless a task notice is waiting | open |
| The precheck leaves out tool schemas (`lua/compactors.rs:233-237`), so a list that passes can still overflow at the provider | A provider overflow on the retry raises `context_exhausted`; counting the schemas, which stay constant per section, comes later | proposed |
| The H1 section runs `models.loop` in the documented decision-tool idiom (`12-tools.md:1496, 1533`) | Give H1 a usable `messages`, like any section; the H1 freeze drops it with the VM (`engine/execute/scheduler/h1.rs:78-82`) | open |
| Authors edit history with `replace`, system records included | Keep `replace` as the author's explicit edit outside compaction | proposed |
| Views held across a swap | Views hold the record itself (`lua/messages-view.rs:18-28`); add a test | resolved |

## Migration

### Prompts

In the Workshop chat agent, delete line 30 (`local history = messages.new()`) and use `messages` for `history` on lines 33 and 34. In the research-person prompt, delete line 40 and use `messages` for `msgs` on lines 41 to 43. The vendored copies in `wg21-paperflow` follow at the next submodule bump. Paperweight calls only `models.infer` and stays as it is.

### Docs

Chapter 11 has 70 lines naming `messages.new()`. Rewrite it around `messages` as the section's conversation, starting with its section on the `messages` global, and restate its overflow rule for compaction. Chapter 12 has 23 such lines and chapter 15 has 15. Chapters 02, 05, 07, 13, and 16 have one to three each, and chapters 14 and 17 name `messages.new` without parentheses. Four examples use two lists in one section:

- Two lists shown to be independent: drop the example.
- Records copied into a second list: copy fields into plain tables.
- Two loops with an offer between them: show the hard error instead, or split the work into two sections.
- A fresh list built to receive a task notice: the notice arrives on `messages`.

### Tests

At `b1bbe33eb`, 64 Rust files under `crates/` contain `messages.new(`. Of those, 58 are tests, test helpers, or benches, and six are source files whose doc comments or strings name it. The Lua crate's test helper installs a list as a global named `msgs`; change it to install the section's `messages`. Rust tests that build `MessageList::default()` directly keep working, because the type stays. The Engine test for a returning compactor becomes a test of the new contract. The Workshop chat gate test already asserts one accumulated history byte for byte and stays as it is, and the Workshop server test prompts change like the chat agent.

### Error messages

Three refusals name the constructor and get new wording: the list assignment refusal, the view's read-only refusal, and `NOT_A_LIST`. The `COMPACTOR_RETURNED` message goes away. The message `messages must not be empty` stays.

Source: `crates/workshop/agents/agents/chat.md:30-34`; `prompts/research-person.md:40-43`; `11-conversations.md:44-52, 64-66, 882-886, 1425`; `12-tools.md:944-951`; `15-tasks.md:2493-2501`; `14-fanout.md:682`; `17-quick-reference.md:115`; `lua/test_vm.rs:254-259, 289-300`; `engine/execute/tests/models_loop_compactors.rs:134`; `crates/workshop/server/tests/it/chat_gate/canonical_sequence.rs:65-110`; `crates/workshop/server/tests/it/agents.rs:47-54`; `lua/messages.rs:431-432`; `lua/messages-view.rs:98-99`; `lua/protocol/parse/chat.rs:18-19`; `lua/models_loop/machine.rs:34-36`; `engine/execute/scheduler/chat.rs:127`.

## Implementation steps

Only step 4 depends on the tool offering redesign. It can ship first by attaching today's effective tool list, rendered once, and switch to the redesigned offers later. Each acceptance test goes in with the step it checks.

1. Install `messages` per VM. Files: `lua/messages.rs` (`install_messages` installs a list), `lua/vm.rs` (`SectionVm` holds the handle), `lua/vm/inject.rs:150`, and `lua/globals.rs` (the guard owns `messages`). Blocked by the H1 choice, which decides only whether H1 gets a list.
2. Remove the constructor. Files: `lua/messages.rs` (`__index` answers `new`, and the refusal at lines 431-432), `lua/messages-view.rs:98-99`, and `lua/protocol/parse/chat.rs:18-19`. Ready now.
3. Run `models.loop` over `messages`. Files: `lua/models_loop.rs:114-131` and `lua/__impl_coro.lua:247-258`. Blocked by the signature choice.
4. Render the catalog once, attach it, and refuse changes. Files: `engine/execute/scheduler/chat.rs:141-168` (render on the first round, reuse after), `lua/vm/state.rs:128-168`, the offer functions in `lua/tools.rs` and `lua/packs.rs`, and the list's state in `lua/messages.rs`. Blocked by two tool offering questions: where the hard error fires, and what the stored catalog holds.
5. Build compaction and its validation. Files: `lua/models_loop/machine.rs` (a path from `Compacting` back to a round, and the retry rule), `lua/__impl_coro.lua:236-238` (pass `old`, `fresh`, and `reason`), `lua/compactors.rs` (the `Compactor` enum and the new `fail`), `lua/messages.rs` (the read-only view, the lock, the swap, and dead handles), and `lua/projection.rs` (the cross-record checks to reuse). Blocked by three open choices: `tools.call` in a compactor, a list with nothing beyond its system records, and the two open validation rules.
6. Add the run-log entries for first render and each compaction. File: `engine/execute/run/effect.rs`. Ready once steps 4 and 5 are done.
7. Migrate prompts, docs, and tests. Each piece is ready once the step it depends on is done.

## Acceptance tests

1. Each section has one `messages`, which only the Engine creates, along with the VM. Two reads give the same object, and every list Lua can reach is that object. (invariant)
2. After a `jump`, and in each task and fan-out arm, `messages` is a different, empty list. (invariant)
3. Reading `messages.new` raises the guiding error at the author's line.
4. Assigning to the `messages` global raises.
5. The catalog renders at the section's first `models.loop` round, and every later request sends the same `tools` bytes, across a multi-round conversation that compacts. (invariant)
6. Once the catalog is attached, an offer that would change it raises at the offer call with the expected text, and an identical re-offer does nothing. (invariant)
7. With nothing offered at the first round, a later offer raises.
8. After a model change between rounds, the catalog stays the same. (invariant)
9. A reference to `messages` taken before compaction is the same object afterward and reads the compacted records, and the run log holds the records before and after. (invariant)
10. A view held from before compaction still reads its original record.
11. Breaking each validation rule raises that rule's error kind and leaves `messages` unchanged.
12. A compactor that returns nil raises `context_exhausted`, and `messages` is unchanged.
13. A compactor that calls `models.infer` and returns `fresh` lets the retried round succeed.
14. A write through `messages` while the compactor runs raises, and any access to `old` or `fresh` after it returns raises.
15. `models.infer` sends an empty tool list and leaves `messages` untouched, before and after the catalog attaches. (invariant)
16. A model task's notice is pushed onto the owner section's `messages`.
17. Replaying a run that compacted, with the Plugins unloaded, reproduces the catalog bytes and the records before and after each compaction. (invariant)

*2026-10-21 07:05 - Claude Opus 5.5 (Cursor agent)*
