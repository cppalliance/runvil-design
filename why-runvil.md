# Why Runvil

*The Markdown is the program, and the model is the CPU.*

## Executive summary

Runvil is an open-source prompting language. A prompt is one Markdown file: the prose is the model's instructions, and the Lua beside it is the logic. Elsewhere, a prompt is a string in a Python program or a long instruction file the model must remember. In Runvil the model does the work that needs judgment and code does the rest, so an agent follows its rules on step one hundred as it did on step one. Runvil is for anyone who can write clearly and think in steps. It is the execution layer for applications that need programmable, inspectable, model-independent AI behavior, and it powers the Knowledge Capture Corporation's pipeline, from interview to indexed knowledge.

## Why it is better

An agent written as an instruction file depends on the model's memory. A model follows a rule that is in front of it and forgets one it must remember for hours. In 104 recorded sessions, our coding agent launched helper agents 3,559 times. A rule it had to remember failed in 1,905 launches, or 53.5 percent; a rule whose template was in view each time failed in 20. The fix is to stop asking the model to do what code can do.

Markdown is the file format of AI. Models answer in it by default, and people expect to read it. Headings divide a Runvil prompt into sections, and within a section the model receives the prose as written: bold and italic text, bulleted and numbered lists, block quotes, and HTML or XML tags. Illustrations and notes for people stay in the file, out of the model's context. On GitHub the prompt renders as a formatted document, and the document a person reviews is the program that runs.

Lua is the deterministic layer. Loops, branches, retries, and round limits are code, and a model runs only when a line of Lua calls it, so a reviewer finds every model call and every branch in the file. Lua is tiny, and it was designed from the ground up to be embedded. In Runvil it runs in a sandbox, and the Harness, Runvil's runtime, performs and records every model call, tool call, and file operation.

Each section starts with an empty tool list, and the model is offered only the tools its Lua names, which keeps it on task. For structured data, other systems ask the model for one large JSON object. They constrain every token to a schema, or validate the result with Pydantic and retry when it fails, which happens more often as the object grows. In Runvil, a model with 100 elements to return calls a tool 100 times, and each call appends one element to a Lua table for ordinary code to loop over. A small call is easy even for the small models that struggle with large JSON. This section of a prompt turns a meeting transcript into JSON:

````markdown
## Extract

Record **every** action item in the transcript below, one at a time.

{{ args }}

```lua
local items = {}
tools.offer(tools.adhoc({
  name = 'record_item', description = 'Record one action item',
  parameters = { owner = 'string', task = 'string' },
}, function(item)
  items[#items + 1] = item
  return 'recorded'
end))
messages:user(prose)
models.loop()
return json.encode(items)
```
````

Runvil was designed to turn prompts that a model orchestrates into programs that Lua orchestrates. It grew out of [tools-public](https://github.com/cppalliance/tools-public), a published library of 66 prompt tools, and it expresses that library's interactive agents more efficiently and its pipelines more tightly. [The Mentographist](https://github.com/cppalliance/tools-public/blob/master/agents/mentograph.md), an interviewer, tracks 15 coverage zones in its reasoning; as Lua tables they survive every compaction. [Entropotros](https://github.com/cppalliance/tools-public/blob/master/prototypes/entropotros.md), a debate host, asks the model to count words and turns, which Lua does instead. [The Nash Equilibrium](https://github.com/cppalliance/tools-public/blob/master/agents/nash.md) calls its document schema inviolable, and with Lua writing the document, it is. [The Staker](https://github.com/cppalliance/tools-public/blob/master/analysis/staker.md) has rules for ten parallel researchers that become a few lines of Lua.

## Top ten features

1. **One Markdown file.** Front matter declares the tools and models a prompt needs, and the whole file is checked before anything runs.
2. **Explicit model calls.** `models.infer` runs one round, `models.loop` runs a bounded tool loop, and failures are typed errors that `pcall` catches.
3. **A tool catalog shaped by the author.** A section can rename, re-describe, or wrap any tool. Large catalogs stay out of the context until the model searches them.
4. **[Ad hoc tools](https://gh.cppalliance.org/runvil/language/12-tools.html#adhoc-tools-with-toolsadhoc).** `tools.adhoc` makes a Lua function a tool in one call, to collect results, record a decision, or guard another tool.
5. **A context the author owns.** A called section returns only its answer. When the window fills, a compactor written in Lua chooses what to keep.
6. **Fan-out and sub-agents.** `tasks.fanout` runs a section once per item, concurrently, and returns the results in order. A model can start sub-agents of its own.
7. **Plugins.** A Plugin is a dynamic library built on a versioned C contract with a safe Rust SDK. Copy it into a folder and every prompt can use its tools. MCP servers connect through a built-in client.
8. **A virtual file system with a bash shell.** Lua, tools, and Plugins share one file system, mounted in memory or over the project's real folder. The bashkit Plugin adds 167 bash commands such as `cat` and `grep`, reimplemented in Rust for language models.
9. **Workshop, the IDE where the agent is a prompt.** The chat agent is a 58-line Runvil file. Edit its system prompt, choose its tools, or write a specialized agent. Run a prompt to a breakpoint, replay to it from the recording, and have the agent help debug it.
10. **Any model, every run recorded.** A prompt asks for a role such as `fast` or `strong`. The Gateway serves 16 cloud providers and local models through one API and alone holds the keys. Every run is written to an append-only log.

## Open source

Runvil is open source under the Boost Software License 1.0 and will stay open source; the Knowledge Capture Corporation builds its commercial products on it. Runvil and its Gateway are 356,000 lines of Rust with more than 6,400 tests, written in fifteen weeks with the prompt tools in tools-public. Find it at [runvil.org](https://runvil.org) and [github.com/cppalliance/runvil](https://github.com/cppalliance/runvil).

---

*2026-11-11 18:47 - Claude Fable 5.2 (Cursor agent).*
