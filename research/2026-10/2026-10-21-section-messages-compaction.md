---
produced: 2026-10-21
title: "Compaction with one conversation per section: the run log, replace, and views, staging list versus in-place snapshot"
---

# Compaction with one conversation per section: the run log, `replace`, and views

- Question: what `replace` and the record views are for when the spec's compactor starts from an empty staging list, and whether that design brings back the quadratic run log the userdata list was built to prevent.
- Code: `runvil` at `aca75013f`. Spec: "One conversation per section: implementation spec", 2026-10-21. History: the messages userdata plan of 2026-10-09.

**Answer.** The run log stays linear in both designs, because the log's delta is computed when a round is sent, by comparing wire messages by content. It does not track edits. A compactor can rebuild the list from nothing and the next round still logs only what changed. Your instinct about `replace` is right, though. In the spec's staging design, `replace` has no job in compaction, and the views are used only as a copy source. A design where the compactor edits `messages` in place with `replace`, and the Engine keeps a cheap snapshot to roll back to, does the same job with fewer parts. It also keeps today's `compactor(reason)` signature. I recommend the in-place design, with medium confidence: it uses fewer parts and the log behaves the same, but it reverses the staging decision you made at 6:53 PM on 2026-10-20.

## 1. What the userdata does for the run log today

The list is a userdata so that it can remember the last request it sent. It does not watch individual edits. Each `models.loop` round goes through `prepare_chat`, which projects the records into wire messages, runs the precheck, and only then calls `commit`:

```173:187:runvil/crates/runvil-internal/lua/src/messages.rs
    /// Records `request` as the list's send in `round`, and returns the
    /// previous send's round and how many leading messages `request`
    /// shares with that send's request; `(None, 0)` on the first send.
    #[must_use]
    pub fn commit(&self, round: RoundId, request: &[Message]) -> (Option<RoundId>, u64) {
        let previous = self.state().last.replace((round, request.to_vec()));
        previous.map_or((None, 0), |(after, sent)| {
            let keep = sent
                .iter()
                .zip(request)
                .take_while(|(sent, now)| sent == now)
                .count();
            (Some(after), u64::try_from(keep).unwrap_or(u64::MAX))
        })
    }
```

The Chat effect record then holds `after` (the round this request extends), `keep` (how many leading wire messages it shares with that round's request), and only the messages after `keep`. Following `after` back and taking `keep` messages at each step rebuilds every request a model saw. The 2026-10-09 plan chose this over exact edit deltas on purpose. It rejected "splices against the previous send, with list ids and versions" because they need per-entry marks and three new public types. It also recorded that "a compaction that rewrites the prefix logs the kept suffix again once, on the next send", a cost you accepted then: "we can pay a little extra per compaction".

Three consequences matter here:

- **Content, not identity.** `keep` compares `Message` values. A record copied into a new list projects to an equal wire message, so it counts as kept exactly like the original would.
- **Only sends are compared.** Edits between two sends are invisible. A record added and then removed costs nothing, and a list rebuilt from empty that ends up equal to the last send logs nothing.
- **A refused round is not committed.** `commit` runs after the precheck (`engine/src/execute/scheduler/chat.rs`, lines 147-157). A round refused by the precheck leaves the list's last send as it was, so the retry after a compaction is compared with the last round that actually went out.

## 2. The run log across one compaction

Take a conversation whose wire messages are S (the system message), then user and assistant turns U1, A1, U2, A2, and so on. Each round logs two new messages:

- Round 1 sends S U1: `after` none, `keep` 0, logs S U1.
- Round 2 sends S U1 A1 U2: `after` 1, `keep` 2, logs A1 U2.
- Round 9 sends S U1 ... U9: `after` 8, `keep` 16, logs A8 U9.
- Round 10 would send S U1 ... U10, but the precheck refuses it. Nothing is committed, and the compactor runs.

Say the compactor keeps S, replaces turns 1 to 8 with a summary record Z, and keeps U9 A9 U10. The projection coalesces consecutive user records into one wire message, so Z and U9 go out as one user message W (`lua/src/projection.rs`, lines 16-21).

- The retry sends S W A9 U10: `after` 9, `keep` 1 (only S matches), logs W A9 U10. That is the compacted suffix, logged once.
- Round 11 sends S W A9 U10 A10 U11: `after` 10, `keep` 4, logs A10 U11.

The log grows by the new turns plus one copy of the compacted suffix per compaction. That is linear, and it is the same whether the compacted records were built in a fresh list or edited in place.

The largest single item in a compaction is not in either design's list mechanics. It is the summarizing call itself. `models.infer` sends one user message with no `after`, so its Chat record logs the whole prompt, and a summarizing prompt contains the text of the turns being summarized. That is one copy of the old turns per compaction, again linear. It is the same in both designs.

## 3. How the spec's staging design runs, step by step

The spec's contract (its "The compaction contract", steps 1-9) calls `compactor(old, fresh, reason)`. `old` is a read-only view of `messages`, `fresh` is an empty list no global names, and the compactor fills `fresh` and returns it. A compactor that keeps the system block and the last two user turns and summarizes the rest looks like this:

```lua
local function text(record)
  local content = record.content
  if type(content) == "string" then return content end
  return json.encode(content)
end

local function compact(old, fresh, reason)
  local first = 1
  while old[first] and old[first].role == "system" do
    fresh:append(old[first])
    first = first + 1
  end
  local cut, turns = #old + 1, 0
  for i = #old, first, -1 do
    if old[i].role == "user" then
      turns = turns + 1
      if turns == 2 then cut = i break end
    end
  end
  if cut == first then return nil end
  local lines = {}
  for i = first, cut - 1 do
    lines[#lines + 1] = old[i].role .. ": " .. text(old[i])
  end
  local summary = models.infer("Summarize this conversation:\n" .. table.concat(lines, "\n"))
  fresh:user("Summary of the earlier conversation: " .. summary)
  for i = cut, #old do
    fresh:append(old[i])
  end
  return fresh
end

models.loop(compact)
```

The cut falls on a user record, so it never separates an assistant tool-call batch from its results.

**Where the views come in.** `old[i]` returns a read-only view, which holds the record itself (`lua/src/messages-view.rs`, lines 18-28). The compactor uses views twice: to read `role` and `content` while it decides what to keep, and as the argument to `fresh:append`. `append` accepts a view because a view serializes as its record's JSON form. The test `a_view_appended_or_converted_gives_the_same_record_as_its_table_form` already does exactly `other:append(msgs[i])`.

**Where `replace` comes in.** It doesn't. `old` refuses every edit, and `fresh` is built in order with `append` and the builders, so nothing calls for `replace`. Under the staging design, `replace` stays only as the author's explicit edit outside compaction (the spec's problems table, "Keep `replace` as the author's explicit edit outside compaction").

**What every kept record costs.** `append` sends its argument through `record_json`, which converts it to JSON with `from_value`, then `parse_record`, which validates it again and builds a new record behind a new `Arc` (`lua/src/messages.rs`, lines 244-260 and 376-381). So every record the compactor keeps is serialized, parsed, and validated once more, and `fresh` holds copies, not the same records. That costs CPU and memory once per compaction. It does not cost log space, because the log compares content.

**What the Engine has to build for it.**

1. An empty staging list for each compaction, reachable only as the compactor's argument.
2. A read-only list type for `old`, which reads like a list and refuses every builder, `append`, and `replace`.
3. A lock on `messages` while the compactor runs, because `old` and the `messages` global are the same records and a write through the global would bypass `old`'s refusals.
4. An identity check on the return value, which must be the staging list itself (the spec's validation rule 1).
5. The swap, which must exchange only the record vectors. The list's send state (`last`) has to stay on `messages`, or the next round loses its `after` and logs the whole compacted request with no link to the previous round. The spec says "swaps the record vectors", which is right, but it should name the send state as staying put.
6. Dead handles: after the swap, any access to `old` or `fresh` raises.
7. The new compactor signature, and `compactors.fail` rewritten for it.

Views survive all of this. A view taken from `old` before the swap is its own userdata, holding its own record, so it still reads after `old` goes dead.

## 4. The alternative: the compactor edits `messages` in place

There is exactly one list per section, and the compactor runs inside the section's loop, so it can reach that list as `messages`. It needs no argument for it, for the same reason `models.loop` lost its list argument. The compactor keeps today's signature, `compactor(reason)`. The one change is that returning now means "compacted, retry the round" instead of the deferred-compactor error. Raising still means "give up", which is what `compactors.fail` already does.

The same policy as above, in place:

```lua
local function text(record)
  local content = record.content
  if type(content) == "string" then return content end
  return json.encode(content)
end

local function compact(reason)
  local first = 1
  while messages[first] and messages[first].role == "system" do
    first = first + 1
  end
  local cut, turns = #messages + 1, 0
  for i = #messages, first, -1 do
    if messages[i].role == "user" then
      turns = turns + 1
      if turns == 2 then cut = i break end
    end
  end
  if cut == first then return compactors.fail(reason) end
  local lines = {}
  for i = first, cut - 1 do
    lines[#lines + 1] = messages[i].role .. ": " .. text(messages[i])
  end
  local summary = models.infer("Summarize this conversation:\n" .. table.concat(lines, "\n"))
  messages:replace(first, cut - 1, {
    role = "user",
    content = "Summary of the earlier conversation: " .. summary,
  })
end

models.loop(compact)
```

**How `replace` is called.** Once, as `messages:replace(first, cut - 1, summary_record)`. That swaps the summarized range for one record. `replace` takes an inclusive range and any number of records, and it already keeps every record outside the range as the same `Arc` (`lua/src/messages.rs`, lines 148-171). So only the summary record is parsed and validated, and the system block and the kept turns are shared, not copied.

**How the views are used.** The compactor reads `messages[i].role` and `messages[i].content` through views while it decides the cut and builds the summary prompt. Those are the same reads as the staging design, just on `messages` instead of `old`. The compactor does all its reading before its one `replace`. If a policy needs the original records after editing, it can collect views into a plain Lua table first, because each view keeps its record alive.

**What the Engine does.**

1. On overflow, it snapshots the list's records. The snapshot is a clone of the record vector, which is a vector of `Arc`s, so it costs one pointer copy per record and no record copies. The send state is not touched.
2. It calls `compactor(reason)` under the raw `pcall`, as today (`lua/src/__impl_coro.lua`, lines 236-238).
3. If the compactor raises, the Engine restores the snapshot and raises the failure, as today plus the restore. That covers a compactor that edits part of the list and then fails.
4. If the compactor returns, the Engine validates the list against the snapshot: tool calls answered, system records leading, and the spec's open and proposed rules (system block unchanged, smaller than before, passes the precheck). On failure it restores the snapshot and raises `lua` or `context_exhausted`, as the spec already assigns them.
5. On success it retries the round, uncounted and at most once per round, as the spec proposes. The `Compacting` phase in `lua/src/models_loop/machine.rs` gets a path back to `next_round`, which is the one new path either design needs.

**What this removes** compared with the staging design: the staging list, the read-only list type, the lock on `messages`, the return identity check, the swap, the dead handles, and the new signature. The settled refusal of a loop started while another runs already stops a compactor from calling `models.loop`. In-place compaction also puts `replace` to work as the one mechanism that rewrites history, which matches the 2026-10-09 plan's recorded decision: "Lua keeps control of compaction policy; `replace` is the mechanism."

**What it gives up.** The staging design gives the compactor a clean list to fill, which is simpler for a policy that rebuilds the history in a very different order. In place, a policy like that either reads everything into a table of views first and then rewrites with `replace`, or makes several `replace` calls from the end of the list backward so the earlier positions don't shift. That is a little more work for unusual policies and none for the usual one, which replaces one middle range with a summary.

## 5. Views across a compaction, in either design

A view is `RecordView(Arc<MessageRecord>)`, and every field read builds a fresh Lua value from that record, so writing into a read table never reaches the record. A view taken before compaction keeps reading its original record after it, in both designs. In place, `replace` drops the list's reference to the summarized records, but a view still holding one keeps it alive. With staging, the swap moves the old records into the dead staging list, and views of them keep reading. This is the spec's "views held across a swap" row, marked resolved, and it holds for the in-place design without change.

Views do not tell Rust anything about identity today, because `append` and `replace` copy what they are given. If sharing ever matters, a small change in both designs would let `append` and `replace` recognize a view and reuse its `Arc`, skipping the JSON round trip and the second validation, since the record is valid already. That saves CPU and memory, not log space. It is not needed for either design to work.

## 6. The compaction entry in the run log

The spec proposes a run-log entry at each compaction holding the reason and the records before and after. Neither copy is needed:

- Every request a model saw can already be rebuilt from the Chat records through `after` and `keep`, including the first request after the compaction.
- Replay reproduces the compaction itself, because it runs as Lua plus recorded `models.infer` answers.
- The "after" records are logged anyway, as the next round's delta.
- The "before" records are mostly in earlier Chat records already. Only records appended after the last send, such as a batch's results before a refused round, are missing, and replay rebuilds those.

A full before-and-after entry costs two more copies of a window's worth of history per compaction. That is still linear, but it is the biggest avoidable cost in the design. A marker holding the reason and the record counts before and after gives a debugger the point where history changed, with no record copies. Confidence medium: it depends on what the prompt debugger needs to show, which is not designed yet.

## 7. The two designs side by side

| Concern | Staging list and swap | In place with snapshot |
|-|-|-|
| Compactor signature | `compactor(old, fresh, reason)` | `compactor(reason)`, as today |
| Role of `replace` | None in compaction | The one edit |
| Role of views | Read `old`, copy into `fresh` | Read `messages` |
| Kept records | Copied, re-parsed, re-validated | Shared, same `Arc` |
| New Engine parts | Staging list, read-only list type, lock, identity check, swap, dead handles | Snapshot and restore |
| Failure handling | Leave `messages` untouched | Restore the snapshot |
| Run log | Linear | Linear, identical |
| Decision status | Settled 2026-10-20, 6:53 and 6:55 PM | Proposed here |

## 8. Recommendation

Adopt the in-place design (medium confidence). It does the same job with `replace`, the mechanism the 2026-10-09 plan already named for compaction. It keeps today's compactor signature, and it needs one new Engine part, a snapshot, instead of six. The run log is identical in both designs. The confidence is medium, not high, because it reverses a decision you made, and because the staging design is simpler for a policy that rebuilds the history in a very different order.

Whichever design you pick, two points apply to both:

- The list's send state stays with `messages` through a compaction, so the next round keeps its `after` link. Under staging that means the swap exchanges records only.
- The compaction's run-log entry should be a marker with the reason and counts, not two copies of the history.

If you adopt the in-place design, the spec changes in these places: the compactor rows of "The Lua surface"; steps 2, 3, 7, and 8 of "The compaction contract"; validation rule 1, which goes, while rules 4 to 6 compare with the snapshot; acceptance tests 8 and 13, which test the snapshot and restore in place of the swap identity, the lock, and the dead handles; and the `COMPACTOR_RETURNED` note, since a returning compactor becomes the retry path.

*2026-10-21 08:40 - Claude Opus 5.5 (Cursor agent)*
