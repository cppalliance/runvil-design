---
produced: 2026-10-21
title: "One conversation per section: implementation spec - messages list, no constructor, rendered catalog attached at first render, compaction in place with a snapshot, replace only inside a compactor, models.compact and compactors.clear, record metadata never sent to the model, validation, migration, acceptance tests"
---

# One conversation per section

- Code: `runvil` at `b1bbe33eb`, `runvil-docs` at `62ae759`.
- Paths: `lua/` is `crates/runvil-internal/lua/src/`, `engine/` is `crates/runvil-internal/engine/src/`, and chapter files are in `runvil-docs/src/language/`.
- Status: settled means Vinnie decided it; proposed means a suggested answer he has not ruled on; open means several answers remain, and the one given is the suggestion; resolved means today's code already behaves as required.
- Drift: later commits, through `aca75013f`, remove `tools.allow_tasks`, the model's task tools, and task notices, so the items that mention them apply only at `b1bbe33eb`. The task-notice items are marked obsolete where they appear.

## The rule

Each section has exactly one conversation, the `messages` list. The Engine creates it, empty, when it creates the section's Lua VM. Only the Engine creates lists, so every list stays in its section. Sections clear history by design, and a prompt that wants to keep history stays in its section. The rendered tool catalog attaches to `messages` the first time it renders and stays attached for the rest of the section, including after a model change. Any attempt to change it after that is a hard error. Compaction is the one place where history is replaced, and outside a compactor `messages` only grows. A compactor edits `messages` in place with `replace`, which is available only while a compactor runs. The Engine snapshots the records before the call, checks the result after it, and restores the snapshot if the compactor raises or the check fails. A section compacts when `models.loop` overflows, or when it calls `models.compact` itself.

## Why

Only `models.loop` sends a list and a tool catalog, because `models.infer` is a single round with one user message and an empty tool list. Tasks and fan-out arms enter their own sections, each in a new VM, so each gets its own list. Only two real prompts create a list today, the Workshop chat agent and the research-person prompt, and each creates one, so the rule matches how prompts already work.

Source: `engine/execute/scheduler/dispatch.rs:242-250`, `engine/execute/scheduler/walk.rs:158-173`, `crates/workshop/agents/agents/chat.md:30`, `prompts/research-person.md:40`.

## The Lua surface

| Concern | Today | After | Status |
|-|-|-|-|
| The list | `messages.new()` returns a new list, any number per section | `messages` is the section's list | settled |
| `messages.new` | The constructor | Gone; reads nil like any other non-integer key on the list | settled |
| Builders, `append`, views, `#`, `pairs`, `ipairs` | Methods on each list | The same methods on `messages` | proposed |
| `replace` | Edits any list at any time | Available only while a compactor runs; outside one it raises | settled |
| Record metadata | Fields beyond `role`, `content`, `tool_calls`, and `tool_call_id` are dropped | An optional `meta` field holding any JSON value, up to `MAX_RECORD_META_BYTES` (8 KB to start); never sent to the model, readable by the prompt and by tools | settled |
| Setting `meta` | None | Only when a record is added, through a trailing argument on every builder or a field of a record given to `append`, or through `replace` | settled |
| Scratch records | A second list | Plain Lua tables in record shape | settled |
| Assigning the `messages` global | Allowed | Raises | settled |
| `models.loop` | `models.loop(list, compactor?)` | `models.loop(compactor?)` runs over `messages` | settled |
| `handle:loop` | `handle:loop(list, compactor?)` | `handle:loop(compactor?)` runs over `messages` | settled |
| An offer after the section's first `models.loop` round | Reaches the next round | `tools.offer`, `packs.offer`, and `tools.offer_local` all raise, even when identical to an earlier offer | settled |
| A loop started while another loop in the section runs | Allowed, over a second list | Raises kind `lua`, naming `models.infer` as the alternative | settled |
| Compactor | `compactor(reason)`, which must raise | `compactor(reason)` edits `messages` in place; returning means compacted, raising means give up | settled |
| `compactors.fail` | The default; raises `context_exhausted` for `"precheck"` or `"provider"` | The default; accepts any reason string and raises `context_exhausted` carrying it | settled |
| `compactors.clear` | None | Removes every record | settled |
| `models.compact(compactor, reason)` | None | Compacts on demand, outside any loop | settled |

The signature is `models.loop(compactor?)`, and `handle:loop(compactor?)` follows it.

Nothing anywhere notices an old form (settled). There are no guiding errors, no compatibility paths, and no migration notes in code, tests, or docs.

Only one loop runs in a section at a time. A `models.loop` or `handle:loop` that starts while another loop in the same section is running raises kind `lua`, whether it starts from a local tool's handler or from a compactor. A loop that starts after an earlier loop returned or raised runs normally. `models.infer` and `tools.call` stay allowed inside handlers and compactors.

`models.compact(compactor, reason)` runs `compactor(reason)` under the same contract as an overflow compaction, and returns nothing. It raises while a loop runs, which covers a call from a local tool's handler, and inside a compactor. `reason` is any string; an overflow passes `"precheck"` or `"provider"`. `models.compact(compactors.clear, "manual compaction")` empties the conversation without leaving the section, and `models.compact(function(reason) messages:replace(#messages, #messages) end, "drop the failed reply")` removes one record.

A record may carry metadata the model never sees (settled). The builders take it as an optional last argument: `messages:system(content, meta)`, `messages:user(content, meta)`, `messages:assistant(content, tool_calls, meta)`, and `messages:tool(content, tool_call_id, meta)`. A record table given to `append` or `replace` takes it as a `meta` field. `meta` must be JSON-representable, and its compact JSON form may be at most `MAX_RECORD_META_BYTES`, a constant set to 8 KB to start. A view reads it as `messages[i].meta`, and a view's JSON form includes it, so a view passed to a tool or to `append` carries it. The projection builds wire messages from the other fields only, so `meta` never reaches the model. Because `commit` compares wire messages, a change to `meta` alone adds nothing to the run log. Records the loop adds carry no `meta`. Open: whether the Engine attaches its own metadata, such as the round id and the provider's token count, to the records the loop adds.

Source: `lua/messages.rs:37-49`, `lua/globals.rs:3-12`, `lua/models_loop.rs:114-131`, `lua/__impl_coro.lua:236-238`, `lua/compactors.rs:295-302`, `lua/protocol/record.rs:85-93`, `lua/projection.rs:22-25`.

## Engine behavior

Today `install_messages` installs a table holding `new`, and injection calls it once per VM, including the H1 section and every task and fan-out arm. After the change, it creates one `MessageList`, keeps a Rust handle to it on `SectionVm`, and installs the list as the `messages` global. The Engine reaches the list only through that handle. The `_G` guard owns the global, as it already owns `argv` and `prose`.

The first `models.loop` round in a section renders the catalog and attaches it to `messages`. Today `prepare_chat` rebuilds the tool scope on every round, from the effective tools plus the task built-ins, and records the advertised names on the chain. After the change it renders once, on the first round, and stores the schemas and the advertised names in the list's state, beside the send state that `commit` keeps. Later rounds send the stored schemas byte for byte, and the dispatch gate checks each model call against the stored names. The model binding is resolved before rendering, and a later model change keeps the stored catalog.

Once the catalog is attached, any call to `tools.offer`, `packs.offer`, or `tools.offer_local` raises at the call, even one identical to an earlier offer (settled). A first round with nothing offered attaches an empty catalog, so a later offer raises the same way (settled). The catalog attaches when the first round builds its tool list, before the projection and the precheck, so a round the precheck refuses still fixes it. The stored catalog holds the schemas, the shape mapping that routes a shaped call back to its canonical tool, the dynamic-set snapshot, and whether the discovery tools are present (settled). Until shaped Packs, the dynamic set, and the discovery tools exist, it holds the schemas and the advertised names, and gains the rest when those features are built.

`models.infer` leaves `messages` and the catalog alone. It sends one user message with an empty tool list and skips the precheck, so it stays the side call a compactor uses to summarize. At `b1bbe33eb`, a model task's notice is pushed onto the owner section's `messages` at the owner's next round. This is obsolete at `aca75013f`, which removed task notices.

Run-log entries for first render and for each compaction are deferred until a debugger needs them (settled). Replay does not need them. The Chat records rebuild every request a model saw through `after` and `keep`, and compaction runs as Lua plus recorded `models.infer` and tool results, so a replay reproduces it with the Plugins unloaded. Each logged chat round keeps only the advertised tool names, so a first-render entry would add the schemas and let a replay check the tool bytes sent. A compaction entry would hold the reason; whether it also holds records or only record counts is decided when the entry is built.

Source: `lua/messages.rs:37-49, 174-188`; `lua/vm/inject.rs:150`; `lua/globals.rs:3-12`; `engine/execute/scheduler/chat.rs:119-124, 133-168, 193`; `lua/vm/state.rs:128-135`; `engine/execute/scheduler/dispatch.rs:242-250`; `engine/execute/run/effect.rs:236-268`.

## The compaction contract

Today the only shipped compactor is `compactors.fail`, and neither real prompt passes its own. A compactor receives only the overflow reason, and one that returns raises "replacement compactors are deferred".

A compaction starts in one of two ways. The loop detects overflow, either from the precheck before dispatch or from the provider's context-overflow reply; both arrive at a round boundary, after the last batch's tool results are pushed, so every tool call in the list has its result. Or the section calls `models.compact(compactor, reason)` outside any loop. Both run the same steps (settled).

1. The Engine snapshots `messages`. The snapshot is a copy of the record vector, one `Arc` per record, so no record is copied. The list's send state is not touched.
2. The Engine makes `replace` available and calls `compactor(reason)`. `reason` is `"precheck"` or `"provider"` for an overflow, and the caller's string for `models.compact`. `replace` stays available until the compactor returns or raises.
3. The compactor edits `messages` in place. It reads records through views and rewrites them with `replace`, and the builders and `append` work as usual. It may call `models.infer`, for example to summarize old turns. `models.loop` and `models.compact` raise inside a compactor. It may call `tools.call` (settled). Tool calls are recorded effects, so replay still works. Restoring the snapshot puts back only `messages`, not a tool's side effects, as with any Lua that calls a tool and then raises.
4. If the compactor raises, the Engine restores the snapshot and raises the compactor's failure. `compactors.fail` gives up this way, with `context_exhausted`. Every write the compactor made through `messages`, by `replace`, a builder, or `append`, is undone by the restore (settled).
5. If the compactor returns, the Engine checks `messages` against the validation rules. Any failure restores the snapshot and raises.
6. On success `messages` keeps the compacted records. It is the same object throughout, so Lua references stay valid and the catalog stays attached.
7. After an overflow, the loop retries the round that overflowed, with the attached catalog unchanged. The retry does not count against `max_tool_iterations`, and a round compacts at most once, so a retry that overflows again raises `context_exhausted` (settled). `models.compact` returns nothing.

The existing list supports this. `replace` already builds the edited record vector inside the same list and keeps every record outside the edited range as the same `Arc`. `commit` compares each request with the last by content, so the next round logs only the messages from the first changed one on. After a compaction the precheck falls back to a full token estimate, because its anchor matches only the messages it measured. A view held from before the compaction keeps reading its original record, because a view holds the record itself.

A compactor can already call `models.infer` and suspend, as an Engine test shows, so returning normally is the new part. Because a compactor may suspend, `models.compact` runs through the same Lua trampoline as `models.loop`. The Workshop chat agent wraps `models.loop` in `pcall`, so a failed compaction ends the turn and the conversation continues.

Source: `lua/compactors.rs:117-126, 171-219`; `prompts/research-person.md:42`; `lua/models_loop/machine.rs:33-36, 196-200, 249-257, 320-341`; `lua/__impl_coro.lua:236-238`; `engine/execute/scheduler/chat.rs:176, 294-303`; `lua/messages.rs:148-188`; `lua/messages-view.rs:18-28`; `engine/execute/tests/models_loop_contract.rs:276-299`; `crates/workshop/agents/agents/chat.md:34`.

## Validation rules

The structural rules apply to every compaction, and the size rules only to one an overflow started (settled).

| Rule | Reason | Applies to | Status |
|-|-|-|-|
| Every assistant tool call has its result right after it | Providers reject an unanswered call (`lua/projection.rs:14-16`) | Every compaction | settled |
| System records appear only in the leading block | The provider's message shape (`lua/messages.rs:61-73`) | Every compaction | settled |
| The list's token estimate is below that of the snapshot | A larger history can only overflow again; Cline applies the same rule | Overflow only | settled |
| The list passes the precheck | Otherwise the retry overflows again | Overflow only | settled |

Breaking either structural rule is an author bug, so it raises kind `lua` at the `models.loop` or `models.compact` call, where `pcall` can catch it. Breaking either size rule means the conversation cannot fit, so it raises the existing `context_exhausted` kind with a message naming the window, the count, and the reserve.

Three earlier rules are gone (settled). The rule that the compactor returns the staging list went with the staging list. The rules that the leading system records equal those before compaction, and that the list holds more than its system records, would reject `compactors.clear`, which removes every record on purpose. An empty list still fails at the next round with `messages must not be empty`.

## Problems in today's code

| Problem | Fix | Status |
|-|-|-|
| The retry counts against `max_tool_iterations`, because `next_round` counts rounds (`lua/models_loop/machine.rs:233-237`) | Leave the retry uncounted, and compact at most once per round | settled |
| A compactor leaves only system records, or nothing | Allowed: `compactors.clear` empties the list on purpose, and an empty list fails at the next round with `messages must not be empty` unless records are added first | settled |
| `compactors.fail` parses its argument as `"precheck"` or `"provider"` and raises on any other (`lua/compactors.rs:295-302`) | Accept any reason string; `context_exhausted` carries it as its `reason` field | settled |
| The precheck leaves out tool schemas (`lua/compactors.rs:233-237`), so a list that passes can still overflow at the provider | A provider overflow on the retry raises `context_exhausted`, because a round compacts at most once; counting the schemas, which stay constant per section, comes later | settled |
| The H1 section runs `models.loop` in the documented decision-tool idiom (`12-tools.md:1496, 1533`) | Give H1 a usable `messages`, like any section; the H1 freeze drops it with the VM (`engine/execute/scheduler/h1.rs:78-82`) | settled |
| A local tool's handler or a compactor starts a loop while the section's loop runs, so its records land before the outer batch's records | Refuse the second loop with kind `lua` | settled |
| Authors edit history with `replace` anywhere, system records included | `replace` works only inside a compactor; an edit outside a loop goes through `models.compact` | settled |
| A local tool's handler appends to `messages` while the loop is mid-batch, so its records land before the batch's records | Refuse Lua edits to `messages` while a loop runs, except inside the compactor | open |
| Views held across a compaction | Views hold the record itself (`lua/messages-view.rs:18-28`); add a test | resolved |

## Migration

### Prompts

In the Workshop chat agent, delete line 30 (`local history = messages.new()`) and use `messages` for `history` on lines 33 and 34. In the research-person prompt, delete line 40 and use `messages` for `msgs` on lines 41 to 43. The vendored copies in `wg21-paperflow` follow at the next submodule bump. Paperweight calls only `models.infer` and stays as it is.

### Docs

The docs are updated as the last step, after all the code, so they match the finished code (settled). That covers the language guide in `runvil-docs` and the `runvil` repository's prose docs: crate and module docs, plugin crate docs, test and bench module docs, READMEs, and the shipped prompts' prose. Each code step still keeps the doc comments on the items it changes accurate.

Chapter 11 has 70 lines naming `messages.new()`. Rewrite it around `messages` as the section's conversation, starting with its section on the `messages` global, and restate its overflow rule for compaction. The guide describes the language as it is, with no mention of the old constructor, the list argument, or a migration. Chapter 12 has 23 such lines and chapter 15 has 15. Chapters 02, 05, 07, 13, and 16 have one to three each, and chapters 14 and 17 name `messages.new` without parentheses. Four examples use two lists in one section:

- Two lists shown to be independent: drop the example.
- Records copied into a second list: copy fields into plain tables.
- Two loops with an offer between them: show the hard error instead, or split the work into two sections.
- A fresh list built to receive a task notice: the notice arrives on `messages`. Obsolete at `aca75013f`, which removed task notices; the example goes with the docs pass for that removal.

Chapter 11 documents `meta`: the builders' last argument, the record field, the view field, the size limit, and that the model never sees it. Chapter 17 adds it to the builder and record rows.

Chapter 11's section on editing a list with `replace` becomes part of compaction: `replace` inside a compactor, and `models.compact` for a rewrite outside a loop, with `compactors.clear` as the way to start the conversation over.

### Tests

At `b1bbe33eb`, 64 Rust files under `crates/` contain `messages.new(`. Of those, 58 are tests, test helpers, or benches, and six are source files whose doc comments or strings name it. The Lua crate's test helper installs a list as a global named `msgs`; change it to install the section's `messages`. Rust tests that build `MessageList::default()` directly keep working, because the type stays. The Engine test for a returning compactor becomes a test of the new contract. At `aca75013f`, seven test files make about 33 Lua calls to `replace`; each moves inside a compactor run by `models.compact`, or to the Rust-side `MessageList::replace`. The Workshop chat gate test already asserts one accumulated history byte for byte and stays as it is, and the Workshop server test prompts change like the chat agent.

### Error messages

Two refusals name the constructor and get new wording: the list assignment refusal and the view's read-only refusal. `NOT_A_LIST` goes away. With no list argument, only the Engine builds the chat request's list field, so a request without a list is a malformed yield. The four loop argument refusals in `lua/models_loop.rs` change from `(messages, compactor?)` to `(compactor?)`, and a new refusal covers a loop started while another runs. New refusals cover `replace` outside a compactor, and `models.compact` while a loop runs or inside a compactor. The refusals that tell authors to change records with `replace` point to `models.compact` instead. New refusals cover a `meta` that is not JSON-representable or is larger than `MAX_RECORD_META_BYTES`, naming the record's position, the size, and the limit. The `COMPACTOR_RETURNED` message goes away, because a compactor that returns has compacted. The message `messages must not be empty` stays.

Source: `crates/workshop/agents/agents/chat.md:30-34`; `prompts/research-person.md:40-43`; `11-conversations.md:44-52, 64-66, 882-886, 1425`; `12-tools.md:944-951`; `15-tasks.md:2493-2501`; `14-fanout.md:682`; `17-quick-reference.md:115`; `lua/test_vm.rs:254-259, 289-300`; `engine/execute/tests/models_loop_compactors.rs:134`; `crates/workshop/server/tests/it/chat_gate/canonical_sequence.rs:65-110`; `crates/workshop/server/tests/it/agents.rs:47-54`; `lua/messages.rs:431-432`; `lua/messages-view.rs:98-99`; `lua/protocol/parse/chat.rs:18-19`; `lua/models_loop/machine.rs:34-36`; `engine/execute/scheduler/chat.rs:127`.

## Implementation steps

Only step 4 depends on the tool offering redesign. It can ship first by attaching today's effective tool list, rendered once, and switch to the redesigned offers later. Each acceptance test goes in with the step it checks.

1. Install `messages` per VM, H1 included. Files: `lua/messages.rs` (`install_messages` installs a list), `lua/vm.rs` (`SectionVm` holds the handle), `lua/vm/inject.rs:150`, and `lua/globals.rs` (the guard owns `messages`). Ready now.
2. Remove the constructor. Files: `lua/messages.rs` (the constructor goes, and the refusal at lines 431-432 is reworded), `lua/messages-view.rs:98-99`, and `lua/protocol/parse/chat.rs:18-19` (`NOT_A_LIST` goes). Ready now.
3. Run `models.loop(compactor?)` over `messages`, and refuse a loop started while another runs. Files: `lua/models_loop.rs:114-131` and `lua/__impl_coro.lua:247-258`. Ready now.
4. Render the catalog once, attach it, and refuse changes. Files: `engine/execute/scheduler/chat.rs:141-168` (render on the first round, reuse after), `lua/vm/state.rs:128-168`, the offer functions in `lua/tools.rs` and `lua/packs.rs`, and the list's state in `lua/messages.rs`. Ready now, over today's effective tool list; the stored catalog gains the shape mapping, the dynamic-set snapshot, and the discovery flag when those features are built.
5. Build compaction in place and its validation. Files: `lua/models_loop/machine.rs` (a path from `Compacting` back to a round, and the retry rule), `lua/__impl_coro.lua:236-258` (the compactor call stays `compactor(reason)`, and `models.compact` gets an entry through the same trampoline), `lua/compactors.rs` (the `Compactor` enum, `fail` taking any reason, and `clear`), `lua/messages.rs` (the snapshot and restore, and `replace` available only while a compactor runs), `lua/models.rs` (`models.compact`), and `lua/projection.rs` (the cross-record checks to reuse). Ready now.
6. Add the run-log entries for first render and each compaction. File: `engine/execute/run/effect.rs`. Deferred until a debugger needs them (settled).
7. Migrate prompts, docs, and tests. Each piece is ready once the step it depends on is done.
8. Add record metadata. Files: `lua/protocol/record.rs` (the `meta` field, `MAX_RECORD_META_BYTES`, and the record's JSON form), `lua/protocol/parse/chat.rs` (`parse_record` reads and bounds `meta`), `lua/messages.rs` (the builders' last argument), and `lua/messages-view.rs` (the view's `meta` field). Ready now, and independent of steps 4 to 6.

## Acceptance tests

1. Each section has one `messages`, which only the Engine creates, along with the VM. Two reads give the same object, and every list Lua can reach is that object. (invariant)
2. After a `jump`, and in each task and fan-out arm, `messages` is a different, empty list. (invariant)
3. Assigning to the `messages` global raises.
4. The catalog renders at the section's first `models.loop` round, and every later request sends the same `tools` bytes, across a multi-round conversation that compacts. (invariant)
5. Once the catalog is attached, any call to `tools.offer`, `packs.offer`, or `tools.offer_local` raises at the call with the expected text, including one identical to an earlier offer. (invariant)
6. With nothing offered at the first round, a later offer raises.
7. After a model change between rounds, the catalog stays the same. (invariant)
8. A reference to `messages` taken before compaction is the same object afterward and reads the compacted records. (invariant)
9. A view held from before compaction still reads its original record.
10. Breaking each validation rule raises that rule's error kind and restores `messages` to its records before the compaction.
11. A compactor that raises, including one that edited `messages` first, restores `messages` to its records before the compaction and raises its failure; `compactors.fail` raises `context_exhausted` carrying the reason.
12. A compactor that calls `models.infer`, edits `messages` with `replace`, and returns lets the retried round succeed.
13. `replace` raises outside a compactor and works inside one, whether an overflow or `models.compact` started it.
14. `models.infer` sends an empty tool list and leaves `messages` untouched, before and after the catalog attaches. (invariant)
15. A model task's notice is pushed onto the owner section's `messages`. (obsolete at `aca75013f`, which removed task notices)
16. Replaying a run that compacted, with the Plugins unloaded, reproduces every request the model saw, before and after each compaction. (invariant)
17. A loop started while another loop in the section runs, from a local tool's handler or from a compactor, raises kind `lua`, and a loop started after an earlier loop returned or raised runs. (invariant)
18. `models.compact(compactors.clear, "manual compaction")` leaves `messages` empty, and a later `messages:user(text)` and `models.loop()` run normally.
19. `models.compact` raises while a loop runs, including from a local tool's handler, and inside a compactor.
20. The size rules do not apply to `models.compact`: a manual compaction that keeps the list's size succeeds, while the structural rules still apply.
21. A compactor that calls `tools.call` gets the tool's result, and a replay of that run reproduces the compaction with the Plugins unloaded. (invariant)
22. A record's `meta` never reaches the model: every wire message holds only `role`, `content`, `tool_calls`, and `tool_call_id`. (invariant)
23. Each builder's last argument and a record table's `meta` field set `meta`, `messages[i].meta` reads it, and a view passed to a tool or to `append` carries it.
24. A `meta` whose compact JSON form is exactly `MAX_RECORD_META_BYTES` is accepted, one byte more is refused at the adding call, and a value that is not JSON-representable is refused there too; a refused record leaves the list unchanged.
25. Replacing a record with one that differs only in `meta` adds no messages to the next Chat record.

*2026-10-21 07:05 - Claude Opus 5.5 (Cursor agent); decisions synced 2026-10-21 10:40*
