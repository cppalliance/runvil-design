# The prompt language: design decisions and their reasons

- Scope: the prompt file as its author writes it: front matter, sections, Lua blocks and prose, the order sections run in, `jump`, `call`, and `require`, arguments, substitution, the Lua globals, and the limits and errors an author meets
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4`
- Update: the Paths bullet names the repositories as of `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 11)
- Update: the entries on `return` in sections 3 and 4, and the entry on `call` in section 3, state the rules of the two changes that follow `5bca6f9` in `runvil-harness`: a `return` always ends its chain, and `call` returns nil for a chain that ends with no text (section 11)
- Update: the summary, the walk-through, and entries in sections 1, 2, 3, and 6, among them the entries on the prompt file, on headings, and on the preamble, state the rules of the two changes that follow `8de43c2f` in `runvil-harness`: every heading starts a section, and the code before the first heading is the preamble (section 11)
- Paths: code paths start at the `crates/` folder of `runvil-harness`, the repository of the Engine and the Harness, and a path that begins with `workshop/` starts at the `crates/` folder of `runvil`

## Summary

A prompt is one Markdown file. Its front matter names the prompt and declares its arguments and requirements, and each heading in its body starts a section, which is a list of prose blocks and Lua blocks. Lua before the first heading is the preamble, which is the first section. The Engine runs the top-level sections in file order, each in a fresh Lua VM. A Lua block moves the run to another section with `jump` or `call`, loads shared code with `require`, and sends prose to a model with a call such as `models.infer`. The Engine passes the table `var` from each section to the next. Sections 1 to 7 describe these parts.

A check of this page against the code found one gap and left nine questions about reasons for the owner. Section 9 describes the gap, and section 10 lists the questions. The gap is in the language guide: eight chapters name a failed parse by a parse error kind, and `Prompt::parse` returns one `Error` type whose `code` names the failure.

Section 8 lists the planned work. Its largest item is a call from one prompt file into another: `call` starts a chain only in the sections of the running prompt.

## From a prompt file to its final text

The shipped prompt `greet.md`, in the repository's `prompts/` folder, has four front matter keys, a paragraph that describes the prompt, and one section named `Main`. Its `args:` key declares one required string named `name`. `Main` has a Lua block, one line of prose with the placeholder `{{ var.greeting }}`, and a second Lua block. `Prompt::parse` checks the keys, builds the section tree, and compiles both Lua blocks.

A caller starts a run with one JSON object as its arguments, for example `{"name":"Ada"}`. The run's first step reads `runvil: 0` and accepts the prompt, and then it checks the arguments against the declaration. The paragraph above the `Main` heading has no Lua block, so it is commentary, and the first code that runs is in `Main`. The Engine's scheduler enters `Main` and builds a fresh Lua VM, where `args.name` is `Ada` and `var` is an empty table.

The first block runs `var.greeting = "Hello, " .. args.name .. "!"`. The prose line becomes the pending prose of the second block. When that block reads `prose`, the Engine replaces the placeholder with the value of `var.greeting`, and `prose` is the text `Repeat exactly, with no extra words: Hello, Ada!`.

The second block calls `models.infer(prose)`, which sends the text to a model and returns the reply ([Model calls and conversations](models-and-conversations.md)). The block returns that reply, and a returned string ends the run as its result. If the block omitted its `return`, the walk, the Engine's pass over the sections in file order, would end after `Main`, and the run's result would be the text `done`.

## 1. The prompt file and its front matter

### A prompt is one Markdown file with front matter and a body, and the body needs no heading

The file opens with YAML front matter between two lines of three dashes, and the rest of the file is the body. The parser requires no heading in the body. A body of one Lua block is a whole prompt, and a body of prose alone parses too and ends its run with the text `done` (section 3). A prompt has no title: its name is the front matter `name`, and the Engine reports the start and the end of a run under that name.

The owner's rule: "we don't require any headings at all". An author who wants a heading above the sections writes one, and the parser reads it as a section like any other (section 2). The owner's rule: "If they want to have the H1, they can have it, but they don't need it." The name is in the front matter because every prompt already states it there. The owner's reason: "the real title for the prompt comes from the front matter".

### One kind of prompt covers pipelines, tool loops, and chats

Every prompt is a Markdown file, and the same Engine code runs a fixed pipeline, a model that calls tools in a loop, and a chat with a user. The Lua in each section decides which of these the section does. The choice belongs to the section because one prompt can combine all three.

### Front matter accepts eight keys and refuses any other

The keys are `name`, `description`, `runvil`, `max_tool_iterations`, `output`, `uses`, `args`, and `models`. The parser requires the first two. The struct `Frontmatter` has the serde attribute `deny_unknown_fields`, so a misspelled key fails the parse with the code `frontmatter_invalid`.

Section 5 covers `args:`. Other pages cover four keys: `uses:` ([Tools and Packs](tools-and-packs.md)), `models:` and `max_tool_iterations` ([Model calls and conversations](models-and-conversations.md)), and `output:`, which names one file in the store ([Files and the VFS](files-and-the-vfs.md)).

### `runvil: 0` marks a prompt, and a language change leaves the version at 0

The run's first step accepts `runvil: 0`, refuses any other number with the code `unsupported_version`, and refuses a file that lacks the key with the message `not a runvil prompt: no runvil version`. Only the semantics of version 0 exist, and a prompt of another version would otherwise fail in the middle of its run with a confusing error.

A change to the language leaves the number at 0 and deletes the old form, which then fails as a call of nil or as an unknown key. The owner's rule: "as long as we are at 0 you will never suggest to increment it." An alias for the old form would leave two spellings in the language.

## 2. Sections, blocks, and prose

### Every heading starts a section, and the shallowest heading level is the top level

Every heading, from H1 to H6, starts a section. The top level is the shallowest heading level in the file, so in a file whose headings are H2 and H3, each H2 is a top-level section and each H3 is a child of the H2 above it. A heading is exactly one level deeper than its parent. A heading that skips a level, such as an H4 directly under an H2, fails the parse as an orphan heading. So does a first heading that is deeper than a later one: an H2 named `Run` above an H1 fails with ``section `Run` is an orphan H2 heading with no parent H1``. A heading with no text fails at every level, with a message such as `an H1 section heading must not be empty`. A heading inside a block quote or a list item is prose of the section around it.

Only headings mark sections, so XML tags remain free for the author and the model to use for emphasis, nonce delimiters, and injection defense.

The top level follows the file so that a plain series of H2 sections runs in file order by fall-through (section 3). The owner's reason: "Wouldn't it be better to just let the author start with the H2? And now it's implicit, it's just a series of H2." If every prompt had an H1 as its root section, the H2 sections would be its children, and each would run only when the H1's Lua named it. The owner's reason: "every H1 that wants to chain has to have a jump, and that looks ugly".

### Lua names a section by its heading, with the `#` markers

A heading reference is the markers, a space, and the name, as in `'## Research'`. The Engine matches the exact level and the exact name. A bare name raises an error that begins `section heading must include ### markers`. Sibling sections have distinct names, and the parser refuses a duplicate.

The name is the heading's text as the Markdown parser decodes it, so Lua addresses a heading written `## Q&amp;A` as `'## Q&A'`. `collect_headings` in `runvil-internal/prompt/src/sections.rs` builds a section's name from the Markdown parser's text events, and the parser emits a decoded entity as text. With names in raw source form, an author would have to write `jump('## Q&amp;A')`.

### A section is an ordered list of Lua blocks and prose blocks

The parser splits a section's content into `Block::Lua` and `Block::Prose` values in file order. A section may have any number of Lua blocks, and the Engine runs each in order. The parser compiles every Lua block, so a syntax error fails the parse ([The Engine](engine.md)). An error message names a Lua block that opens a section the prologue and one that ends it the epilog.

### A Lua block has one accepted spelling, and a near miss fails the parse

A Lua block opens with a line of exactly three backticks and `lua` and closes with a line of exactly three backticks. The parser checks the whole body before it compiles a block, and the first violation fails the parse with the code `fence_invalid`. A label that starts with `lua` in any letter case and differs from it, such as `Lua` or `lua extra`, is a violation. So is an opening or closing line of another shape, and so is a line outside every fence that looks like a `lua` opening line. If the parser read a near miss as prose, a model would receive the Lua source as prompt text.

A fence with another label, such as `json`, is prose, because prompts show JSON and code examples to models in fences. The parser skips the lines inside another fence, so an author shows a Lua block as an example inside a fence of four backticks.

### The Markdown above a Lua block is that block's `prose`, and a thematic break discards the text above it

The parser collects the Markdown that follows a heading or a Lua block, and the next Lua block reads it as the global `prose`. A thematic break, a line of three dashes, empties the collected text, so the text above a break is commentary for human readers. Markdown after a section's last Lua block is commentary too.

A break needs a blank line above it, or Markdown reads the line above as a heading. Emptying the collected text is a break's only meaning, and this one rule explains commentary at the start of a section, before every Lua block, and after the last Lua block.

### The Engine renders the placeholders in `prose` at the block's first read

Each Lua block has its own `prose`. The Engine substitutes the `{{ }}` placeholders when the block first reads the global, and it stores the result for later reads. The Engine discards prose that a block leaves unread, bad placeholders included. An assignment to `prose` raises an error.

The Engine sends prose to a model when a Lua block passes it to a call such as `models.infer(prose)` ([Model calls and conversations](models-and-conversations.md)).

### A section whose lines are all list markers is a list section

When a section has only prose and every nonblank line starts with a marker (`-`, `*`, `1.`, or `1)`), the parser stores each line's text as an item of the section. A marker with an empty text fails the parse with the code `list_empty_item`. Lua reads the items with `prompt.list`. `tasks.fanout` can run a worker section once for each item, and each such run is a fanout arm ([Tasks and fanout](tasks-and-fanout.md)).

The parser builds the items when it loads the prompt, so an empty item fails the parse, where it would otherwise fail a run. A plain line among the markers is different: `bullet_items` in `runvil-internal/prompt/src/list.rs` treats the whole section as prose, so that ordinary prose with one incidental bullet in it remains prose. When the prompt runs, `prompt.list` raises for such a section with an error that ends `has no pre-parsed items`.

### `prompt.section` and `prompt.list` read another section, and a miss returns nil

`prompt.section(heading)` returns a section's own raw Markdown, from the line after its heading to the next heading of any level. `prompt.list(heading)` returns the items of a list section as an array of strings. Both return nil for a heading outside the caller's visible set (section 3), and both raise for a malformed heading.

A miss returns nil because a read is optional by nature: a prompt can use headings as an index and test each lookup for nil.

## 3. The order sections run in

### Lua decides every move between sections

A run moves to another section when the walk falls through, when a Lua block calls `jump` or `call`, or when Lua starts a task ([Tasks and fanout](tasks-and-fanout.md)). `jump` and `call` are Lua functions, and the Engine leaves them out of every tool list. A model influences the order through data that Lua reads and branches on.

The rule exists so that a reader of the Lua can reconstruct the whole control graph, and so that every branch is a visible Lua condition.

### The code before the first heading is the preamble, and it runs first as an ordinary section

The preamble is the text between the front matter and the first heading. When it has a Lua block, it is the first top-level section: its name is the empty string, it has no children, and the main walk enters it first, in a fresh VM, as it enters any section. When it has no Lua block, it is commentary, and the walk starts at the first heading's section. The preamble's blocks follow the rules of section 2, so the Markdown above a Lua block is that block's `prose`. A `return` in the preamble ends the run, and a preamble that runs off its end falls through to the next top-level section.

The preamble and the sections after it are entries of one chain, the main walk. The next section therefore starts with the preamble's `var`, on the model that the preamble left current, and under the limit that the preamble set with `tasks.concurrency`, and it can wait on a task that the preamble spawned. From the preamble the top-level sections are siblings, as in `jump('../Scan')`, and a child path misses with an error that ends `` the preamble has no child `Scan` ``. The preamble reports its section start and its section finish under the empty name, and an error message labels its Lua blocks by their place, as `preamble prologue`, `preamble lua 2`, and `preamble epilog`.

An uncaught Lua error in the preamble, such as a failed `assert`, ends the run with the code `lua_runtime`, as in every section, before the next section runs. An author therefore writes there, as an `assert` call, a hard check that the `args:` declaration cannot express, such as the range of a number. `requirements_unmet` and `args_invalid` are the codes of the run's start checks alone ([The Engine](engine.md)).

The Engine has one code path for every section, and the preamble is a section whose name is empty. The owner's rule: "I don't want any special functions for H1. I want everything to be the same." The start of a prompt has no setup that only it can do: tools, model calls, `call`, and `jump` work in every section, and the main walk starts on the `default` model ([Model calls and conversations](models-and-conversations.md)). The owner's reason: "what can H1 really do in terms of setup? I'm thinking nothing."

The run starts at the first top-level section, whichever it is. The owner's reason: "every prompt has a beginning, and we simply start there". A prompt whose body is a Lua block and then a section under the heading `## Research` runs the block and then `Research`. The owner's goal: "I want this to chain without a jump".

### Top-level sections run in file order, and a nested section runs when Lua names it

The main walk enters the first top-level section, which is the preamble when the prompt has one, runs its blocks, and falls through to the next top-level section, until a block executes a `return` or the last top-level section ends. A child section runs when Lua names it with `jump`, `call`, `require`, or a task call. An H1 above a series of H2 headings makes them its children, so the walk runs the H1's blocks, and each H2 section runs only when Lua names it.

A subroutine section is therefore a child of the section that calls it, or a top-level section placed after the section whose `return` ends the run.

### Each section entry starts a fresh Lua VM, and the Engine copies `var` into it

The Engine builds a new Lua VM each time a chain enters a section ([The Engine](engine.md)). A chain is one sequence of section entries: the main walk, a `call`, or a task. A plain Lua global that one section sets reads nil in the next. The Engine copies the previous section's `var` into the new VM, and sections also exchange data through a returned value, the store, and the arguments of a `call`.

With these explicit channels, a section behaves the same whether the run enters it by fall-through, by `jump`, by `call`, or as a task. One VM for a whole walk would create hidden globals that depend on the path, state that is impossible to clone, and inconsistent inputs for a section.

### A `return` ends the chain, and a walk past its last section returns `done`

A `return` at the top level of a Lua block ends the chain the block runs in. A returned string, number, or boolean is the chain's result as text, and on the main walk that result is the run's result. A `return` of nil, or of no value, ends the chain with no text, the way a walk past its last section ends. A returned table fails with `cannot return a table as a result`. After a block that ends with no `return`, the Engine runs the next block. The rule makes Lua control flow compact: one `return` both ends the chain and states its result.

A `return` ends the chain whatever its value, because the author wrote it to end the chain. If a nil value moved the run on to the next block, `return x` would end the chain or continue it depending on what `x` holds, and `return call(...)` would depend on what the called section did. A `return` inside a function is that function's, and it leaves the chain running.

Lua returns no value both for a bare `return` and for a block that ends with none, so the Engine marks the difference when it compiles a block. `LuaProgram::compile` in `runvil-internal/lua/src/program.rs` compiles the block's source inside a `do` block, followed by a `return` of the block's one argument. The Engine calls every block with a private marker as that argument, so a block returns the marker only when it runs off its end. The compiler checks the source alone first, so a syntax error reads as it does for the source alone, and every line of the block keeps its number.

When the main walk runs past its last section, or ends at a `return` with no text, the run's result is the text `done`. A prompt that lacks sections, one whose body has no heading and no Lua block, is valid and ends the same way, because the walk has no section to enter.

### `jump` ends the block and continues the walk at another section

`jump(heading)` records its target and ends the current block. The Engine then closes the section, looks the heading up, and enters the target in a fresh VM that starts with the jumper's `var`. A section may jump to an earlier sibling, so two sections can form a loop.

A section of a called chain may jump as well, so what a section can do is independent of how the run entered it. The prompt's author is responsible for coherent control flow, as in any programming language, and that includes infinite loops and wrong targets.

### A jump to a child section starts a walk at the child's level

When a section jumps to one of its direct children, the Engine walks the children from that target and falls through the target's following siblings. When the last child ends, the walk resumes at the section after the jumper.

The rules are identical at every level: the Engine's section semantics are independent of the heading level. A detour that ran only the one child and then returned would apply different fall-through rules at a child level than at the top level.

### `call` runs a contained chain and returns its final text, or nil for a chain with none

`call(heading, args)` starts a new chain at the target and suspends the calling block. The chain follows the rules of the walk: it falls through the target's following siblings, and its sections may jump. It ends when a block executes a `return` or the last section at its level ends. `call` returns the chain's final text as a string, and it returns nil for a chain that ends with no text.

A `call` works like a call of a different prompt whose sections happen to be in the same file. The caller inherits only the chain's answer, because the tokens that the chain spent would otherwise bloat the caller.

`call` returns nil for a chain with no text because the chain gave it nothing, and the empty string would read as a reply. The caller tests the value or gives it a fallback with `or`, and a chain can still return the empty string on purpose. With nil, `return call(...)` also hands on what the called chain did: it ends the caller's chain with no text, where the empty string would end it with a text that no block returned. A caller that joins the value to a string with `..` gets a Lua error for nil, at the line of the join. A task and a fanout arm that end with no text report the empty string, because their text is one field of a result that also says whether they succeeded ([Tasks and fanout](tasks-and-fanout.md)).

### A call's second argument is the called chain's `args`, and the chain starts with a copy of `var`

The second argument of `call` is optional: a table of named fields or a string. Every section of the called chain reads a table as its `args`, and reads a string as `args.text`. With no second argument, the chain reads the caller's `args`. A table that is a list raises `args must be a table of named fields or a string, got a list`. The Engine checks a call's arguments against no declaration and fills no defaults in them. The chain's `var` is a deep copy of the caller's `var`, and the Engine discards the chain's writes to it when the chain ends.

The `args:` declaration describes what a caller passes the prompt, so it has no say in what one section passes another. A string becomes the field `text` so that `args` is a table in every chain, and `text` is also the field of a prompt that declares no arguments (section 5).

### A section names its siblings and its direct children

Every function that takes a heading, except `require`, looks it up in the calling section's visible set: the other sections with the same parent, plus the section's direct children. The parent, a sibling's child, and the section itself are outside the set. A `jump` or a `call` to a heading outside the set fails with an error that lists the set ([The Engine](engine.md)).

With only the children in the set, two sibling sections would be unable to share one worker, and with only the top-level sections in it, a child section would be unaddressable. The error lists only the visible set, so the error text hides the rest of the document's structure.

## 4. Sharing code with `require`

### `require` loads a section as a library into the calling section's VM

`require '## helpers'` runs the Lua blocks of the named section inside the caller's VM. The library reads the caller's globals, `var`, and `args`, and the globals it defines remain for the caller. Runvil's `require` loads only sections of the running prompt, and the Engine leaves Lua's `package` library unloaded.

Shared code exists for reuse, so a library runs only where a section asks for it. With Lua's own `require`, a prompt could load Lua files from disk, and library code would be unable to make a call that waits.

### A library runs once in each section VM, and `require` returns the value of its first `return`

A library's blocks run in order. After a block that ends with no `return`, the next block runs. The first block that executes a `return` ends the load, and `require` returns its value, which may be a table or a function, or `true` when the `return` gives nil or no value. For a library whose blocks all end with no `return`, `require` returns `true`. Later calls in the same VM return the saved value, and a library that requires itself raises an error that begins `require cycle:`.

`require` returns a value of any type because a library returns its module as a table. A `return` ends the load the way a block's `return` ends a chain, and the result `true` for a library with no value matches standard Lua. The cycle error exists because standard Lua 5.5 lacks cycle detection and recurses until the stack overflows.

### `require` can also name the siblings of each ancestor section

From a section, `require` finds the section's direct children, its siblings, and the siblings of each of its ancestors up to the top level. `call`, `jump`, the prompt reads, and the task calls use the smaller visible set. A `require` that runs while a library loads resolves from the library's own position, so a library can load helper sections nested under it.

The wider set exists for worker sections, which are often nested. With the smaller set, an author would have to copy each library under every parent section. Control flow needs a smaller set than reuse does.

### A library is an ordinary section, and each section requires the libraries it uses

A library section has the same syntax as any section, and the walk enters it like any other. A library that returns a table therefore fails the run there, so the author places it after the section whose `return` ends the run, or nests it under the section that uses it. The placement is the author's responsibility, which the design prefers to a special marker for library sections.

A library that a parent section loaded exists only in the parent's VM, so a called section, a task, and a fanout arm each call `require` themselves. With inheritance, the Engine would replay a parent's libraries implicitly in each child VM.

### A library may call a model while it loads, and `jump` raises during a load

`require` is a Lua function, so a library's blocks run inside the calling block's coroutine ([The Engine](engine.md)) and may call a model, the store, and tools while they load. Library code is ordinary section code that runs in another place. A `jump` during a load raises an error that begins `jump is unavailable while a library loads`, because a transfer of control in the middle of a load lacks a coherent meaning.

The parser leaves `require` calls unchecked, so a bad heading raises an error when the call runs. A check at parse time would need each block's syntax tree, which the Lua compiler leaves unexposed, and a search of the text would misfire on strings and comments.

## 5. Arguments and substitution

### The arguments of a run are one JSON object, and the run's first step checks them

A caller starts a run with the text of one JSON object, and a caller with nothing to pass sends `{}`. The run's first step checks that text directly after the version ([The Engine](engine.md)). Text that is not JSON, JSON that is not an object, and an object that differs from the prompt's declaration each end the run there, before any section runs, with the code `args_invalid`. The error names the first mismatch, as in ``argument `query` must be a string, got integer``. It has no location, because the fault is in the caller's text, and the prompt has no line to name.

The arguments are an object so that a Host can publish any prompt as a tool: the arguments of an MCP tool call are a JSON object. The owner's reason: "we can always put it behind an MCP server". The Engine makes the check, so every section reads arguments of the declared shape, and a prompt has no checks of its own to write for a missing field or a wrong type.

### `args:` declares the object in a subset of JSON Schema, and the default is one optional string named `text`

The value of `args:` is a schema with eight keywords: `type`, `properties`, `required`, `additionalProperties`, `items`, `enum`, `description`, and `default`. A `type` is `string`, `boolean`, `integer`, `number`, `array`, or `object`, and the root is an object whose `properties` name the arguments. The parser refuses any other keyword, a node that lacks a `type`, an array that lacks `items`, and a default that differs from its own node. Each refusal has the code `frontmatter_invalid` and names the YAML path of the node, such as `args.properties.query`. An object is closed: a field outside its `properties` ends the run, unless the object says `additionalProperties: true`. Before the check, the Engine gives each missing field its `default`, at every depth. A `null` for a field outside the `required` list counts as the field left out. A prompt that omits the key takes one optional string named `text`, so a Host with plain text sends `{"text":"hello"}`.

The declaration is JSON Schema because that is the form in which a tool publishes its arguments, and `ArgsSchema::as_json` returns the declaration in that form. It is a subset so that an unknown keyword fails the parse: a misspelled keyword is an error when the prompt loads, where a reader of full JSON Schema would ignore it. An object is closed so that the Engine refuses a misspelled optional field, which an open object would accept and the prompt would never read. The Engine fills a default because a caller that reads the schema expects the prompt to use it. A `null` counts as a field left out because models send `null` for a field they omit, and the checker of the `std` Pack reads a `null` the same way ([Tools and Packs](tools-and-packs.md)).

The owner's rule: "there must be no freeform prompts". The default field is named `text` because that is the word for what a chain returns: a chain ends with its final text.

### `args` is the checked object, a read-only table in every section

Every section VM has `args` behind the guard on the globals table (section 6). It holds the object that passed the check, with its defaults, so it is a table in every section, and a field that the caller left out and that has no default reads nil. Each table inside it is a read-only view. An assignment to the global raises ``args is read-only: assign to `var` or a section global instead``, and a write to a field named `x` raises `args is read-only: cannot set field 'x'`. The preamble, a called chain, and a task follow the same rule. A null inside an object that takes undeclared fields reads as `json.null`. A prompt that needs a different value builds it in Lua and keeps it in `var`.

Apart from writes, a view behaves like the table it stands for. `#`, `pairs`, `next`, `ipairs`, `table.concat`, and `table.unpack` read the data. A view that Lua hands to `tools.call`, to `tasks.fanout`, to a spawn's `item`, or to `var` arrives there as its data.

`args` always holds what the caller passed, so a reader of any section knows where its value came from. The name is `args` because it can only mean what the caller passed, where "input" could also mean what a user sends during the run. One rule for every section also means one way to install `args` in a VM. The owner's reason: "I value uniformity over everything else." A view is an empty stand-in for its table because Lua calls `__newindex` only for a key that a table lacks, so a table that held its own entries would accept a write to each of them.

### A placeholder names data in `args`, `item`, `var`, `sys`, or a Lua global

A placeholder is a path between double braces, such as `{{ var.greeting }}`. The first segment selects the source: `args` for the arguments, `item` for the member a task or a fanout arm runs for, `var`, or `sys`. Any other first segment reads the Lua global of that name. The segments after the first are keys, as in `{{ args.query }}`. A string renders as written, a number as text, and a table as compact JSON, so `{{ args }}` renders the whole object.

A missing key, a null value, or a path that ends at a function raises an error at the read of `prose`, and an uncaught one ends the run with the code `substitution`. The owner's rule: "No function calls in {{ }}." Substitution is data only: a placeholder that called a function would let prose invoke tools.

## 6. The Lua environment

### A section VM runs Lua 5.5 with the `string`, `table`, `math`, and `utf8` libraries

The Engine removes fourteen base globals, among them `load`, `dofile`, `rawget`, `rawset`, `collectgarbage`, and `print`. The names `io`, `os`, `package`, `debug`, and `coroutine` read nil.

The libraries include `utf8` because it is pure, and an author needs it to cut a string at a character boundary.

### The Engine installs 20 globals and reserves 62 names

Six Engine globals are the data of the run: `args`, `item`, `messages`, `prose`, `sys`, and `var`. Eight are tables of operations: `compactors`, `json`, `markdown`, `models`, `prompt`, `store`, `tasks`, and `tools`. Six are functions: `call`, `jump`, `log`, `require`, `ui`, and `untrusted`. Two exist only in some VMs: `ui` when the caller supplied a snapshot, and `item` in the first section of a task whose spawn supplied one. The list `RESERVED_NAMES` adds the 19 Lua globals that the sandbox leaves in place and the 23 keywords of Lua 5.5. A Pack prelude, the Lua that the Engine runs for a Pack in each section VM, fails to install when it defines a reserved name.

Other pages cover `models`, `messages`, and `compactors` ([Model calls and conversations](models-and-conversations.md)), `tools` and `untrusted` ([Tools and Packs](tools-and-packs.md)), `tasks` and `item` ([Tasks and fanout](tasks-and-fanout.md)), and `store` ([Files and the VFS](files-and-the-vfs.md)).

`markdown` has five functions over Markdown text from outside the prompt's own sections. `markdown.parse` reads text into a list of blocks, which are plain tables. `markdown.section` returns the one section that a path names, or one block of it, and raises when it is absent, and `markdown.section_or` returns nil for that miss. `markdown.select` collects every block that a selector matches, and `markdown.render` writes the blocks back as one text.

### A handle is a plain value, and its operations are functions of a namespace table

A model handle, a tool object, and a task handle are values with fields. Each operation on one is a function of a namespace table that takes the handle as an argument, as in `models.infer(handle, text)` and `tools.call(tool, arguments)`. The message list is the one exception: its builders are colon methods, as in `messages:user(text)`.

The owner's reason: "Simpler implementation. Simpler mental model for authors. This is best for AI because only one way to do things." [Model calls and conversations](models-and-conversations.md) covers model handles.

### `var` is a guarded table of JSON data

`var` is an empty proxy table over a hidden data table, and each table that an author reads from it is a view of the same kind. The proxy checks each write at the assigning line: a function, a userdata, or a thread raises an error that says the value `must be JSON data`, and a key that JSON data has no place for, such as a boolean or an index past the end of a list, raises an error that names the key. A view measures and iterates like the table it stands for: `#` gives the length of the data, `pairs` and `next` visit its entries, and `var.items[#var.items + 1] = x` appends to a list.

The proxy is empty because Lua calls `__newindex` only for a key that a table lacks, so only an empty table sees every write. The Engine sets `__len` on each view, and its own `pairs` and `next` read through one, so that an author treats a `var` table as a plain table.

### `sys` is a sealed table of facts about the run and the section entry

`sys.when` is the run's start time, `sys.id` is the section entry's id, such as `0.0` for the first section that the main walk enters ([Tasks and fanout](tasks-and-fanout.md)), `sys.taskid` is the enclosing task, `sys.section_name` is the name of the running section, which is the empty string in the preamble, and `sys.section_count` is the number of top-level sections, the preamble included. `sys.index` exists in a task's first section when the spawn supplied an index, and `sys.model` exists after the entry's first tool call. A read of any other field raises `unknown sys field`, and a write raises `sys is read-only`.

The table refuses writes because `sys` is runtime metadata: the Engine writes it, and an author only reads it. The Engine reads the start time from its caller ([The Engine](engine.md)).

### `log` records one line of at most 256 characters, and the Engine removes `print`

`log(message)` takes exactly one string: a single line of at most 256 characters. A newline or a control character in it raises an error. The Engine reports the string as an `Event::Lua` under the current section. A section VM accepts 1024 calls by default, and the next call raises an error with the code `lua_quota`.

The limits loosen only after examples from real prompt debugging establish a need for multiline or structured values.

### `ui()` returns the Host's snapshot from the start of the run

`ui()` returns a new table on each call with the data the caller supplied when the run started. The Harness fills two fields, `selected_model` and `workspace_root`.

### `pairs` and `next` visit keys in a fixed order

The Engine replaces both functions. Each visits the array part of a table in index order, then the other keys sorted: booleans, then numbers, then strings. Two runs therefore walk a table the same way. Both read through a view, so a walk of a `var` table or of a table inside `args` visits the entries of the data in that order, and the value it yields for a nested table is a view too. A `__pairs` metamethod still applies, on a table and on a userdata.

Lua leaves the order of hash keys open, and the Lua binding sets the hash seed itself. The sort makes the order independent of that internal hash state.

### One guard on `_G` answers reads of `args`, `prose`, and `messages`, and `setmetatable` refuses finalizers

The Engine puts one metatable on the globals table. It answers reads of `args`, `prose`, and `messages` and raises on an assignment to any of the three. An author's `setmetatable(_G, mt)` records `mt` behind the guard, so the documented pattern of default values for globals works. Author code can overwrite every other Engine global for the rest of that VM.

`setmetatable` raises for a metatable with a `__gc` field. Lua runs finalizers with hooks disabled, so a finalizer that loops would continue to run after a cancel.

### `json` is a global table with `json.null` for null and `json.array` for lists

`json` has six members: `encode`, `encode_lines`, `decode`, `decode_lines`, `null`, and `array`. `json.decode` returns fresh writable tables and returns `json.null` for each JSON null. `json.null` is truthy, and code tests for it with `v == json.null`. An unmarked empty table encodes as `{}`, and `json.array()` returns an empty table that encodes as `[]`. The two `_lines` functions read and write JSON Lines.

The table is always present because `require` loads only sections of the running prompt. Null is a sentinel value so that a key whose value is null remains in its table. A sequence encodes as an array, so only the empty array needs the `json.array()` marker.

### `json.encode` raises for a value outside JSON and names its path

`json.encode` raises for a function, a thread, a userdata other than `json.null`, NaN, an infinity, a table that mixes list items and named keys, and a list with a gap. The message names the place, as in `value.items[3]`. The encoder writes keys sorted, so the same value produces the same text and the same error on every run. The option `{ debug = true }` writes a placeholder string such as `"[function]"` for each refused value.

The Engine walks the value itself, because the conversion of mlua, the Lua binding, silently drops keys from mixed tables and leaves the path out of its errors. The walk reads through sealed tables such as `var`, because `json.encode(var.items)` would otherwise produce `{}` silently.

## 7. Errors and limits

### Lua catches every Engine failure as a table with `kind`, `code`, and `message`

`pcall` returns a failure as a table whose `tostring` is its message. `kind` is one of sixteen strings that an author branches on, such as `lua`, `store`, `tool`, `model_failure`, and `canceled`. `code` is the tag of the exact failure, such as `lua_quota`, and an uncaught error ends the run with it ([The Engine](engine.md)). `message` starts with the prompt's file and line when the failure has a location.

The table prints as its message, so code that prints a caught failure shows the text, and code that branches reads `kind`.

### A cancel passes through `pcall`, and a stop raises an error that `pcall` catches

When the Host cancels a run, the Engine's `pcall` and `xpcall` raise a caught failure again, and `xpcall` skips the author's message handler. A Lua block may run any number of instructions, and a hook reads the cancel flag every 10,000 instructions ([The Engine](engine.md)). A cancel passes through `pcall` because cancellation is the only way to stop a runaway loop.

A stop drops the model round or the tool call in flight and leaves the cancel flag clear. The waiting Lua call raises an error of kind `canceled`, and a `pcall` around it catches the error. The built-in chat prompt wraps `models.loop` in `pcall`, so a stop ends one reply and the chat continues ([The Harness and the Host API](harness-and-host.md)).

### A call that waits raises an error inside a comparator of `table.sort`

A call that waits, such as `tools.call`, yields the block's coroutine, and the module comment of `runvil-internal/lua/src/coro.rs` says "Yield cannot cross the C boundary". Lua's `table.sort` is a C function, so a waiting call inside its comparator raises Lua's own error, `attempt to yield across a C-call boundary`.

### `call`, tasks, and fanout arms nest at most 8 levels

A `call`, a task, and a fanout arm each run one level deeper than the chain that started them, and a `jump` runs at the jumper's level. A start at level 9 fails with the code `call_depth_exceeded` ([Tasks and fanout](tasks-and-fanout.md)).

The nesting cap and the round cap of `models.loop` are backstops: a prompt ends through its explicit Lua conditions. `jump` calls are unlimited in number, so a loop of `jump` calls ends by a Lua condition or by a cancel.

### One `json` or `var` conversion has a nesting limit and a size budget

The limits are fixed numbers:

- `json.encode` and `json.decode` accept 127 levels of nesting.
- `json.decode` accepts 8,388,608 bytes of input.
- One `json.encode` call or one write to `var` may visit as many values as one eighth of the VM's memory limit in bytes, which is 8,388,608 values at the default of 64 MiB.
- One path of an encoded value may have 100,000 keys that are strings or tables.

The nesting limit is 127 because `serde_json`, which parses the input of `json.decode`, refuses the 128th nested array or object, and the encoder writes only a depth that the decoder accepts. The size budget exists for tables that are small in Lua and huge once converted: a conversion runs inside one Rust call, outside the instruction hook and outside the limit on the Lua heap.

## 8. Planned work

Each of these items is still to be built.

- **A call into another prompt file.** A Lua function, planned as `execute`, that calls another prompt file, so that a large program can consist of several prompt files. `call` starts a chain only in the sections of the running prompt. `Prompt::parse` finds the run's prompt by its name through a `PromptFinder`, and a called prompt is to resolve the same way.
- **Lua module files and Markdown includes.** A `require` of a Lua file by its path, as in `require("./path.lua")`, and an `include` for partial Markdown. `require` loads only sections of the running prompt.
- **Front matter in Lua.** A sealed table with the prompt's own `name`, `args`, and `models` declarations. The `prompt` table has two functions, `section` and `list`, and both read sections.
- **A live clock.** A way for Lua to read the current time during a run. `sys.when` is the run's start time.
- **More of `json`.** Encoding of message records, pretty printing, and a decode that builds its values inside the VM's memory limit. `json.encode` raises for a message record, as for every userdata other than `json.null`, and `json.decode` accepts 8,388,608 bytes of input.
- **An acyclic section order.** A rule that `jump` names only a child or a later sibling. Whether the rule is possible, and whether it would limit authors too much, are open questions. A section may jump to an earlier sibling, so two sections can form a loop.

## 9. Known gaps

The item names a place where the language guide differs from the code.

1. Parse error kinds. Eight chapters of the guide name a failed parse by a parse error kind, such as `Fence` or `Structure`, and chapter 16 also lists the code for each kind. `Prompt::parse` returns the one `Error` type ([The Engine](engine.md)), whose `code` is `frontmatter_invalid`, `structure_invalid`, `fence_invalid`, `list_empty_item`, `lua_compile`, or `internal`.

## 10. Questions for the owner

Each question asks for the reason behind a part of the design.

**Sections and the walk**

1. Why does Lua name a section with its `#` markers?
2. Why does the walk skip child sections?
3. Why does a called chain, a task, or a fanout arm work on a copy of `var`, and why does the Engine discard its writes?
4. Why does a main walk that ends with no text return the text `done`, while a called chain returns nil and a task returns the empty string?
5. Why may a section alternate any number of Lua blocks and prose blocks?
6. Why is the number of `jump` calls unlimited, while nesting and the rounds of `models.loop` each have a cap?

**Parsing**

7. Why is a heading inside a block quote or a list item prose?
8. Why is the `runvil:` key optional to the parser and required by the run's first step?

**The Lua environment**

9. Why does a read of an unknown `sys` field raise, while a missing key of `var` or of `args` reads nil?

## 11. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Move.** On 2026-11-12 the Paths bullet was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history and at the same paths under `crates/`. The `prompts/` folder of the walk-through moved with them. The text of the page still describes the code at the commits above, when those crates were in `runvil`.
- **Return rule.** On 2026-11-13 the entries on `return` in sections 3 and 4 were brought to the change that follows `5bca6f9` in `runvil-harness`, whose plan is `plan/2026-11-13-1-return-ends-the-chain.plan.md` there. Before it, a block that returned nil or no value let the next block run, and `require` took the first block value other than nil. That session ran the build and its tests. The rest of the page still describes the code at the commits above.
- **`call` with no text.** On 2026-11-13 the entry on `call` in section 3 and question 4 of section 10 were brought to the second change that follows `5bca6f9` in `runvil-harness`: `call` returns nil for a chain that ends with no text, where it returned the empty string. That session ran the build and its tests too.
- **Every heading is a section.** On 2026-11-13 the text on headings, the preamble, and `argv` was brought to the two changes that follow `8de43c2f` in `runvil-harness`, whose plan is `plan/2026-11-13-2-every-heading-is-a-section.plan.md` there. The update covers the summary, the walk-through, the entry on the prompt file in section 1, the entry on headings in section 2, the entries on the preamble, the walk, chains, `return`, and a jump to a child in section 3, the entries on `args` and `argv` in section 5, the entries on `sys` and the `_G` guard in section 6, and the questions of section 10. Before them, a prompt had exactly one H1, whose text was its title, and the Lua blocks under it ran once as the H1 pass, which handed its `var`, its `argv`, its model, its concurrency limit, and its tasks to the walk over the H2 sections. A Lua error in the pass ended the run with the code `requirements_unmet`, and the pass could assign `argv`, which every later section read frozen. Section 10 had a question on a prompt with an H1 and no sections, and the questions after it each moved up one number. That session ran the build and its tests.

*2026-11-10 16:47 - Claude Fable 5.2*
