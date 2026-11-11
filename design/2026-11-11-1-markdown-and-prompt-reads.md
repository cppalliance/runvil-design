# Section Paths, the `markdown` Namespace, and the `prompt` Reads

Status: proposed design, 2026-11-11. It defines three things: section paths, the one way the language names a section; the `markdown` namespace; and the `prompt` reads, `prompt.section` and `prompt.section_or`. `prompt.section` exists today in its one-argument form. Everything else in this document is new and not implemented. The design also removes three things from the language, listed under [No trace of what this design removes](#no-trace-of-what-this-design-removes).

The parsing rules, the path and selector rules, `render`, and the examples that go with them were checked against a throwaway prototype built on `pulldown-cmark` 0.12.2, the version the workspace pins. The prototype also ran the `prompt` examples below over the document's own samples: the visible set, paths from the top, the reads, and what a miss reports. The prototype is not part of this repository.

The `require` loader in `runvil`, `__impl_require.lua`, was run unchanged over stand-ins for the Engine functions it calls. That showed what it does when a section requires itself, when two sections require each other, and when one library is required twice.

The rule for library code, under [Where a path is resolved from](#where-a-path-is-resolved-from), was built in a throwaway copy of `runvil` at commit `b0693699d` and run against the Engine's own tests. [What the library environment changes](#what-the-library-environment-changes) records what was built and what the tests showed. The copy is not part of any repository.

## Contents

1. [Summary](#summary)
2. [Design rules](#design-rules)
3. [Section paths](#section-paths)
4. [No trace of what this design removes](#no-trace-of-what-this-design-removes)
5. [The block model](#the-block-model)
6. [`markdown`](#markdown)
7. [`prompt`](#prompt)
8. [One document, both namespaces](#one-document-both-namespaces)
9. [Scope, identity, and copies](#scope-identity-and-copies)
10. [Recipes](#recipes)
11. [Errors](#errors)
12. [Decisions](#decisions)
13. [Rejected alternatives](#rejected-alternatives)
14. [Open questions](#open-questions)
15. [Implementation notes](#implementation-notes)

## Summary

- **Section paths** are the one way the language names a section. A path is titles separated by `/`. In a prompt, a relative path names a neighbor of the section its code is written in: `Template` is a child and `../Review` is a sibling. A path from the top of the prompt, such as `/Library/Template`, names any section. The `prompt` reads take either kind, and `require` takes only a path from the top.
- **A library names its own sections.** A relative path in a library's code names a child or a sibling of the library, whichever section is using it. So a library can keep its own templates and workers.
- **`markdown`** parses any text into plain Lua tables, the way `json.decode` does for JSON. It names a section by a path from the top of that text, such as `/Guide/Setup`.
- **`prompt`** reads any section of the running prompt.
- Both namespaces pick one block inside a section with the same selector, such as `"fence markdown"` or `"list"`, and an index when the section holds several.
- In both, `section` raises when the section or block is not there, and `section_or` returns `nil` instead.

| | `markdown` | `prompt` |
|---|---|---|
| Reads | any text passed to it | the running prompt |
| A path starts at | the list of blocks passed in | the section the code is written in, or the top of the prompt |
| Path forms | `/Guide/Setup`, with `//` and `[n]` | `Template` for a child, `../Review` for a sibling, `/Library/Template` from the top |
| Can name | every section in the parsed text | every section in the prompt |
| A section comes back as | a table with its subsections inside | a string of its own raw Markdown |
| A block comes back as | the block table inside the parsed text | a block table built for the call |

## Design rules

1. **One way to name a section.** Every function that names a section takes a section path. There is no second form.
2. **Plain data.** Every value is a string, number, boolean, or table. No table has methods, and each table is either a list or a record, never both. A parse result therefore passes `json.encode`, can be stored in `var`, and can be a `tasks.fanout` collection.
3. **Functions, not methods.** Every operation is a namespace function over those values, as with `models` and `tools`. There is one way to do each thing.
4. **Strict by default.** `section` raises when the section or block it names is not there, and the error says what is missing. `section_or` returns `nil` instead, and its name says so. A malformed path, selector, or index raises from both.
5. **Raw text.** Nothing is run or substituted. A `{{ name }}` placeholder comes back as written.
6. **No file access and no effects.** `markdown` takes text, never a file name. A file is read with `store.read` and then parsed. Every function here is pure: the same input gives the same result, and none of them performs an effect or suspends.
7. **Lookups share, they do not copy.** A `markdown` lookup returns the tables that are already inside the list it was given. It creates no strings.

## Section paths

### The syntax

A path is a sequence of steps separated by `/`, and a step is a section's title. These rules hold wherever a path is used:

- A title matches exactly and case-sensitively. The heading level plays no part. It follows from where the section sits.
- Whitespace around the path and around each step is ignored, so `" .. / Review "` is `../Review`.
- A slash inside a title is written `\/`, and a backslash `\\`. In Lua source the long-bracket form avoids doubling: `[[Input\/Output]]`.
- `\/`, `\\`, and `\[` are the only escapes. `\[` is covered under [Paths](#paths). Any other backslash raises.
- An empty step raises: `Guide/`, `../`, and the empty string are all malformed.
- A path is a string. Any other type raises.

Where a path starts, and which forms it may use, depends on what takes it:

| Taken by | Starts at | Forms |
|---|---|---|
| `markdown.section`, `markdown.section_or` | the list of blocks passed in | `/a/b` from the top of the list, with `//title` for any depth and `title[n]` for the nth match |
| `call`, `tasks.spawn`, `tasks.fanout` | the section the code is written in | `name` for a direct child, `../name` for a sibling |
| `jump` | the section that is running | `name` and `../name` as above |
| `prompt.section`, `prompt.section_or` | the section the code is written in, or the top of the prompt | `name` and `../name` as above, or `/a/b` for any section in the prompt |
| `require` | the top of the prompt | `/a/b` for any section in the prompt |

A form used outside its row raises. A `markdown` path with no leading `/`, or with `..`, raises. A prompt path with `//` or an index raises. So does a relative path that is anything other than `name` or `../name`, such as `a/b`, `../../a`, or `..` alone, and a path from the top that contains `..`. A path from the top given to `call`, `jump`, or a `tasks` function raises, and so does a relative path given to `require`.

### Paths in a prompt

A prompt path has three forms. The first two are relative:

- `name` is the direct child titled `name`.
- `../name` is the sibling titled `name`: another section under the same parent. A top-level section's siblings are the other top-level sections.
- `/a/b` starts at the top of the prompt and names one title per level. The H1's title is not a step: `/Templates` is the top-level section titled `Templates`. The `prompt` reads take this form, and it is the only form `require` takes.

A relative path starts at the section its code is written in. For a section's own blocks, that is the section that is running. For a library's code, it is the library, whichever section is using it. [Where a path is resolved from](#where-a-path-is-resolved-from) has the whole rule. A path from the top is the same from everywhere.

From the H1 body, the top-level sections are the children, so the H1 runs `call("Write")`. The H1 has no siblings, so `../name` names nothing there.

### The visible set

The two relative forms name a section's visible set: its siblings, except itself, and its direct children. `call`, `jump`, `tasks.spawn`, and `tasks.fanout` run a section, and they take only these two forms. They have no way to write a parent, a grandchild, a sibling's child, or a cousin, so each of them reaches only a neighbor of the section it starts from.

```text
## Main
### Kid
#### Grand
## Sibling
## Other
### Niece
```

| Path, from `Main` | Result | Why |
|---|---|---|
| `Kid` | found | a direct child |
| `../Sibling`, `../Other` | found | siblings |
| `../Main` | nothing | a section is not its own sibling |
| `Grand`, `Niece` | nothing | `Main` has no child with either title |
| `Kid/Grand`, `../Other/Niece` | raises | a relative path has one name |

A path names at most one section. The prompt parser rejects two siblings with the same title, so a child's name and a sibling's name are each unique, and so is each step of a path from the top.

A section whose title is exactly `..` cannot be named.

### Paths from the top

A path from the top names any section in the prompt, and it names the same section from everywhere. Two kinds of function take one. Neither runs the section as part of the walk, so neither is held to the visible set.

- **The `prompt` reads** return a section's text. They take a path from the top beside the two relative forms.
- **`require`** loads a section's Lua as a library. It takes a path from the top and nothing else.

```text
## Top
## Grand
### Uncle
#### Cousin
### Parent
#### Sibling
##### Niece
#### Self
##### Child
###### Grandchild
## Last
```

| Path, from `Self` | Result | Relationship |
|---|---|---|
| `/Top`, `/Last` | found | top-level sections |
| `/Grand/Uncle` | found | a sibling of the parent |
| `/Grand/Uncle/Cousin` | found | a child of an ancestor's sibling |
| `/Grand/Parent/Sibling/Niece` | found | a sibling's child |
| `/Grand/Parent/Self/Child/Grandchild` | found | a grandchild |
| `/Grand/Parent`, `/Grand` | found | ancestors |
| `/Grand/Parent/Self` | found | `Self` itself |
| `/Grand/Parent/Sibling`, `/Grand/Parent/Self/Child` | found | a sibling and a child, the same sections as `../Sibling` and `Child` |
| `/Grand/Missing` | nothing | `Grand` has no child with that title |

A shared section has one address at every depth. A worker nested anywhere writes `require("/helpers")` for a library and `prompt.section("/Templates", "fence markdown")` for a template.

### A library has one name

`require` takes only a path from the top. `require("helpers")` and `require("../helpers")` raise, and the message says what `require` takes.

- **One spelling.** A library is written the same way in the H1 body, in a top-level section, and in a nested worker.
- **One meaning.** The same `require` names the same library on every line of the prompt, inside a library included. Where it is written and which section is running it play no part.
- **Nothing to fix when code moves.** Nesting a section deeper, or moving its code to another section, changes none of its `require` calls.
- **One name in the output.** The load events and the cycle error print a library by the same path, so one search finds every place that requires it.

A library kept under the section that uses it is named through that section, as in `require("/Main/helpers")`. Renaming or moving `Main` breaks that path, and a copy of `Main` keeps requiring the original's library until its path is edited. The same holds for a library that keeps a helper library under itself: `require("/Library/helper")` is the one line in `Library` that names `Library`.

Nothing is excluded, and nothing guards against a strange `require`. A section may read its own text or an ancestor's. It may also `require` them, and what follows is the author's to handle: the code runs as any library's does. A section that requires itself, and two sections that require each other, meet the cycle error.

### Where a path is resolved from

A relative path is resolved from the section its code is written in.

- **A section's own blocks** resolve from that section, which is the one that is running. A called chain, a task, and a fanout arm each run a section of their own and resolve from it.
- **A library's code** resolves from the library section. That holds while the library loads and when its functions run later, and it does not matter which section is using the library.
- **A function keeps the place it was written in.** A function that a section passes to a library still names the section's neighbors. A tool handler written in a library names the library's.

So a library reads and runs its own sections by relative paths, and each line means the same thing for every user:

```lua
-- written in a library whose children are Template and Worker
local template = prompt.section("Template", "fence markdown").text
local replies = tasks.fanout("Worker", items)
```

A name is resolved where it is used, not where it came from. A section that passes the string `"Worker"` to a function of a library gets the library's `Worker`.

Two things in a library's code still belong to the section that is running:

- **`jump`.** It moves the walk, and the walk is the running section's. So its path names a neighbor of the running section, in a library's code too. A `jump` made while a library loads is refused, as it is today.
- **`_G`.** Inside a library, `_G.prompt`, `_G.call`, and `_G.tasks` are the running section's own. `_G.prompt.section("Topics")` reads a child of whichever section is using the library. A library that does this is relying on its user to have that section, and meeting that is on the user.

In a section's own blocks, a plain name and its `_G` name are the same function.

A path from the top is resolved from the top, so it names the same section from everywhere. `require` takes nothing else.

### When a path names nothing

A well-formed path that names no section is a miss.

- `call`, `jump`, `tasks.spawn`, `tasks.fanout`, `require`, and `prompt.section` raise the not-found error.
- `prompt.section_or` returns `nil`.

The error names the path and says where it looked: the section that was searched, and whether a child or a sibling was wanted there. From `Main` in the tree under [The visible set](#the-visible-set):

```text
section `Grand` not found: `/Main` has no child `Grand`
section `Sibling` not found: `/Main` has no child `Sibling`
section `../Kid` not found: `/Main` has no sibling `Kid`
section `/Other/Neice` not found: `/Other` has no child `Neice`
section `/Othr/Niece` not found: the top level has no section `Othr`
```

The second and third lines are one mistake made in each direction. `Sibling` is a sibling of `Main` and needs `../`, and `Kid` is a child and takes none. A path from the top reports the step where it stopped.

From the H1 body, a child is a top-level section, and there are no siblings:

```text
section `Kid` not found: the top level has no section `Kid`
section `../Main` not found: the H1 has no siblings
```

A relative path is searched from the section its code is written in, so that is the section the error names. For a library's code it is the library. A section that passes the name of its own `Worker` to a function of the library `kit` sees this:

```text
section `Worker` not found: `/kit` has no child `Worker`
```

The error lists no other sections.

### Where the Engine prints a section

- A not-found error prints the path it was given, then the section it searched, by that section's path from the top, and the title it wanted there.
- A library is reported by its path from the top. The three library-load events carry it in their `library` field, such as `/shared_stuff`, and the cycle error reads `require cycle: /a -> /b -> /a`.
- A bare title stays a bare title: `sys.section_name`, the `section` field of `tasks.status`, and the `section` field of events.

### Functions that take a path

| Function | The path names | Forms | On a miss |
|---|---|---|---|
| `call(path, input?)` | the section to run and return from | relative | raises |
| `jump(path)` | the section the walk moves to | relative | raises |
| `tasks.spawn(path, opts?)` | the section the task starts in | relative | raises |
| `tasks.fanout(worker, collection)`, `tasks.fanout(model, worker, collection)` | the worker section | relative | raises |
| `require(path)` | the library section | from the top | raises |
| `prompt.section(path)`, `prompt.section(path, selector, n?)` | the section to read | relative, or from the top | raises |
| `prompt.section_or(path)`, `prompt.section_or(path, selector, n?)` | the section to read | relative, or from the top | `nil` |

A relative path starts at the section the code is written in, except `jump`'s, which starts at the running section. Nothing else about these functions changes. A path is looked up when the call runs, not when the prompt is parsed.

## No trace of what this design removes

This design removes three things from the language, and it removes each one completely. Nothing that remains may show that any of them existed.

1. **The address form with `#` markers.** A section reference no longer writes the section's heading markers, as in `"## Rules"`. Section paths are the only way the language names a section.
2. **`prompt.list`.** The read is `prompt.section(path, "list").items`.
3. **List sections.** The parser no longer treats a section that holds only a list as a kind of its own. The parse error for an empty bullet in such a section goes with it, and so does the refusal to use such a section as a worker. A list is a `list` block and nothing else.

The same rules apply to all three:

- **No code recognizes them.** No parser branch, check, warning, hint, or error message looks for `#` markers in a section reference, for a call of `prompt.list`, or for a section made only of a list. A string such as `"## Rules"` is an ordinary relative path with one name, `## Rules`. It names a child with that title. No section normally has one, so the lookup misses exactly as it does for any other name that matches nothing: the ordinary not-found error, or `nil` from `prompt.section_or`. Given to `require`, it raises the way every relative path does there. `prompt.list` is not a field of `prompt`, so calling it fails the way calling any missing field fails, with no message of its own. A section that holds only a list is an ordinary section.
- **No documentation mentions them.** The guide, the language spec, crate docs, code comments, error text, and test names describe the language as it is. None of them says how something used to be written, compares an old form with a new one, or notes that anything changed.
- **No name carries them.** An identifier, parameter, field, or module that calls a section address a heading is renamed for what it holds: a path. Names that exist only for list sections are deleted with them.
- **No output shows them.** Wherever the Engine prints or records a section address, it writes a path. No message names `prompt.list`.
- **No prompt uses them.** Every prompt in the repositories, shipped, example, and test, is rewritten in the same change.

Done means a search of the `runvil` and `runvil-docs` repositories, and of the language spec, finds no section reference written with `#` markers, no `prompt.list`, and no sentence about any of the three. The dated records in `runvil-design`, this document among them, are the only places that say they existed.

The same holds for what this design changes without removing: which sections `require` and the reads can name, which form `require` takes, which section a library's relative paths start at, what a read does on a miss, and what the not-found error prints. The guide describes each as it is, with no note that it was ever otherwise.

## The block model

### Blocks

A parsed document is a list of blocks. A block is a record with a `kind`, the `line` it starts on, and the fields of its kind.

| `kind` | Fields | What it is |
|---|---|---|
| `frontmatter` | `data` | The YAML mapping that opens the text, as JSON-shaped tables. It is only ever the first block. |
| `section` | `level`, `title`, `blocks` | A heading and everything under it. `level` is 1 to 6. `blocks` is a list of blocks, subsections included. |
| `prose` | `text` | A run of Markdown that is no other kind: paragraphs, block quotes, indented code, HTML, link definitions. |
| `fence` | `lang`, `info`, `text` | A fenced code block. `info` is the whole info string, `lang` its first word, `text` the body. |
| `list` | `ordered`, `items` | A bullet or numbered list. `items` is a list of strings with the markers removed. |
| `rule` | none | A thematic break such as `---`. |

### Shape rules

- A table is a list (keys 1 to n) or a record (string keys). `json.encode` refuses a table that "mixes list items and named keys", so a section keeps its members in `blocks` instead of being an array itself.
- The document, every `blocks`, and every `items` are marked lists, the kind `json.array` makes, so an empty one encodes as `[]`.
- No block points at its parent or its neighbors, so there are no cycles.
- A field that a kind does not have reads as `nil`. It is never an error.
- `title`, a fence's `text`, and an item can be the empty string. A prose `text` never is.

### Text and line numbers

- The text must be UTF-8. A byte order mark at the very start is dropped.
- `\r\n` and a lone `\r` are read as `\n`. Every string in the result uses `\n` only.
- `line` is 1-based. It counts lines of the text given to `markdown.parse`, frontmatter lines included.
- `line` is the line a block starts on: the heading line for a section, the opening fence line for a fence, the first marker line for a list, and the first non-blank line for prose.
- `line` is stored when the text is parsed. No lookup changes it. See [Scope, identity, and copies](#scope-identity-and-copies).

### Parsing rules

1. **CommonMark, no extensions.** The parser and its options are the ones the prompt parser uses, so a heading or a fence means the same thing in a prompt and in parsed text. Tables, task lists, and footnotes are not parsed as such. They stay in prose or in an item's string.
2. **Only top-level constructs become blocks.** A heading, fence, list, or thematic break inside a block quote stays in the surrounding prose. One inside a list item stays in that item's string.
3. **Nothing is dropped.** Every non-blank line of the text belongs to exactly one block, counting a heading line as part of its section. Whatever is not frontmatter, a heading, a fence, a list, or a rule is prose.
4. **Frontmatter.** It is recognized only at the very start: the first line is `---`, a later line is `---`, and the lines between parse as a YAML mapping. The delimiter rule is the prompt parser's, which ignores trailing spaces. `data` holds the mapping as JSON-shaped tables: a mapping is a record, a sequence is a marked list, and a YAML null is `json.null`. When any condition fails there is no frontmatter, and those lines are ordinary Markdown.
5. **Section.** A heading opens a section, which ends at the next heading of the same or a smaller level. A heading of a larger level opens a subsection inside it, even when it skips levels: an H4 directly under an H2 is a member of the H2's `blocks` with `level = 4`. A section's own content always comes before its subsections. `title` is the heading's text with inline markup removed, a line break inside the heading read as a space, and the result trimmed.
6. **Prose.** Prose is a maximal run, so two prose blocks are never adjacent. A second prose block exists only when a fence, list, or rule sits between it and the first. Two paragraphs separated by a blank line are one block. `text` is the source as written: leading blank lines and trailing whitespace are removed, and the first line keeps its indentation.
7. **Fence.** Backtick and tilde fences both count. `info` is the info string, trimmed. `lang` is its first word, or `""` when the fence has none. `text` is the lines between the two fence lines, with the fence's own indentation removed and without the final line ending. A fence is opaque: a `#` line inside it is not a heading. A fence with no closing line runs to the end of the text.
8. **List.** One block per list, and a change of marker, such as `-` then `*`, starts a new list. `ordered` is true for a numbered list. An item is its source with the marker removed, each later line losing the indentation the marker gave it, and trailing whitespace trimmed. An item that spans lines or holds nested content keeps it as raw Markdown, so `markdown.parse(item)` recovers its structure.
9. **Rule.** The name is `rule` because `break` is a Lua keyword and could not be a bare key in a dispatch table.
10. **Parsing does not fail on content.** Any UTF-8 text within the size limit is a document. Skipped levels, repeated titles, and an unclosed fence are all accepted. This is looser than the prompt parser on purpose, because `markdown` reads text the author does not control, such as a model reply.

### Worked example

````markdown
---
title: Notes
tags: [a, b]
---

Preface paragraph.

# Guide

Intro text
on two lines.

- alpha
- beta

## Templates

Use these.

```lua
return 1
```

```markdown
# Hello
```

Then the fallback:

```json strict
{ "ok": true }
```

### Extras

---

```lua
return 2
```

## Usage

1. first
2. second
````

`local md = markdown.parse(text)` gives:

```lua
md = {
  { kind = "frontmatter", line = 1, data = { title = "Notes", tags = { "a", "b" } } },
  { kind = "prose", line = 6, text = "Preface paragraph." },
  { kind = "section", line = 8, level = 1, title = "Guide", blocks = {
    { kind = "prose", line = 10, text = "Intro text\non two lines." },
    { kind = "list", line = 13, ordered = false, items = { "alpha", "beta" } },
    { kind = "section", line = 16, level = 2, title = "Templates", blocks = {
      { kind = "prose", line = 18, text = "Use these." },
      { kind = "fence", line = 20, lang = "lua", info = "lua", text = "return 1" },
      { kind = "fence", line = 24, lang = "markdown", info = "markdown", text = "# Hello" },
      { kind = "prose", line = 28, text = "Then the fallback:" },
      { kind = "fence", line = 30, lang = "json", info = "json strict", text = '{ "ok": true }' },
      { kind = "section", line = 34, level = 3, title = "Extras", blocks = {
        { kind = "rule", line = 36 },
        { kind = "fence", line = 38, lang = "lua", info = "lua", text = "return 2" },
      } },
    } },
    { kind = "section", line = 42, level = 2, title = "Usage", blocks = {
      { kind = "list", line = 44, ordered = true, items = { "first", "second" } },
    } },
  } },
}
```

Because the result is plain tables, ordinary indexing reads all of it:

```lua
#md                              --> 3
md[1].kind                       --> "frontmatter"
md[1].data.title                 --> "Notes"
md[1].data.tags[2]               --> "b"
md[2].text                       --> "Preface paragraph."
md[2].line                       --> 6
md[4]                            --> nil
md.kind                          --> nil     (md is the list, not a block)

local guide = md[3]
guide.kind                       --> "section"
guide.level                      --> 1
guide.title                      --> "Guide"
#guide.blocks                    --> 4
guide.blocks[1].text             --> "Intro text\non two lines."
guide.blocks[2].ordered          --> false
guide.blocks[2].items[1]         --> "alpha"
#guide.blocks[2].items           --> 2

local templates = guide.blocks[3]
templates.title                  --> "Templates"
templates.blocks[2].lang         --> "lua"
templates.blocks[5].lang         --> "json"
templates.blocks[5].info         --> "json strict"
templates.blocks[5].text         --> '{ "ok": true }'
templates.blocks[6].title        --> "Extras"
templates.blocks[6].blocks[1]    --> { kind = "rule", line = 36 }
templates.blocks[2].title        --> nil     (a field another kind owns)

for i, block in ipairs(md) do end   -- top-level blocks in file order
```

### Edge cases

Each row was run through the prototype.

| Text | Result |
|---|---|
| Empty, or only blank lines | An empty list. |
| Two paragraphs separated by a blank line | One `prose`. |
| A paragraph, then a list or a fence with no blank line between | A `prose`, then the `list` or `fence`. |
| A heading inside a block quote | It stays in the `prose`, as `> # Quoted`. |
| A heading, fence, or nested list inside a list item | It stays in the item's string. |
| A list or fence inside a block quote | It stays in the `prose`. |
| A `#` line inside a fence or an HTML comment | Not a heading. |
| `Title` over `=====`, or `Sub` over `---` | A level 1 or a level 2 section. |
| A paragraph line directly above `---` | A level 2 section, as CommonMark reads it, not a rule. |
| ``## The *big* `foo` API ##`` | A section with `title = "The big foo API"`. |
| `##` with no text | A section with `title = ""`. No path can name it; indexing and `markdown.select` reach it. |
| `#### B` directly under `## A` | `B` is in `A`'s `blocks`, with `level = 4`. |
| `### A`, then `# B` | Two top-level sections. |
| Two lists with different markers, back to back | Two `list` blocks. |
| An item spanning lines, or holding a nested list or a fence | One string, such as `"a\n- a1\n- a2"`. |
| An item with no content | `""`. |
| `- [ ] todo` | The item is `"[ ] todo"`. |
| A numbered list that starts at 5 | `ordered = true`. The start number is not kept. |
| A fence with no closing line | It runs to the end of the text, and later `#` lines are part of its `text`. |
| A fence with no lines inside | `text = ""`. |
| A fence with no info string | `lang = ""` and `info = ""`. |
| A `~~~` fence, or a fence indented up to three spaces | A `fence`. The indentation is removed from `text`. |
| Indented code, raw HTML, a pipe table, a link definition | `prose`. |
| `---` on the first line with no closing `---`, or with lines between that are not a YAML mapping | No frontmatter. The first line is a `rule`, and the rest is ordinary Markdown. |
| Frontmatter with a key such as `1:` or `true:` | The key reads as its text, `"1"` or `"true"`. |
| Frontmatter holding `.inf`, `.nan`, or a date | `json.null` for a number JSON cannot hold, and a string for a date. |
| Frontmatter that repeats a key | The key keeps its last value. |

## `markdown`

`markdown` is an Engine global in every prompt, like `json`. It needs no declaration.

```lua
markdown.parse(text)                              --> list of blocks
markdown.section(blocks, path)                    --> section; raises when there is none
markdown.section(blocks, path, selector, n?)      --> block; raises when there is none
markdown.section_or(blocks, path)                 --> section, or nil
markdown.section_or(blocks, path, selector, n?)   --> block, or nil
markdown.select(blocks, selector)                 --> list of blocks
markdown.render(blocks)                           --> string
```

### The `blocks` argument

In every signature, `blocks` is a list of blocks: the value `markdown.parse` returned, any section's `blocks`, or a list the author built.

- **A lookup is scoped to the list it is given.** Pass a section's `blocks` to search inside that section and nowhere else.
- **A section is not a list.** Passing a section record raises, and the message points at its `blocks` field. One call never has two meanings.
- **It is read as a sequence:** items 1, 2, and so on, up to the first `nil`. An item that is not a table raises.
- **Views are read through.** A table read back from `var`, or reached through any other guarded value, is a view over the real table. The functions read through it by the rule `json.encode` uses, so `markdown.section(var.doc, "/Guide")` works.
- **Only the needed fields are read.** A lookup reads `kind`, `title`, `blocks`, and `lang`. A block whose `kind` it does not know matches nothing. `render` checks every field of every block it writes.
- **Depth is bounded.** A list nested more than 127 levels deep, which is the `json.encode` limit, or a list that contains itself, raises. Parsed text never nests past six sections.

### Syntax

```
path      = step , { step } ;
step      = ( "/" | "//" ) , title , [ "[" , index , "]" ] ;
title     = one or more characters, with "\/" for a slash, "\\" for a backslash,
            and "\[" for a bracket that would otherwise read as an index ;

selector  = kind | "fence" , space , lang | "fence" , space , '""' ;
kind      = "frontmatter" | "section" | "prose" | "fence" | "list" | "rule" ;
lang      = one or more characters with no space ;

index     = a nonzero integer, with "-" in front to count from the end ;
```

A path and a selector are strings. Any other type raises.

### Paths

A `markdown` path is a [section path](#section-paths) from the top of the list passed in. It always starts with `/`. It adds two forms a prompt path does not have, and it borrows both from XPath.

- `/title` steps to the sections with that title among the direct members of the current list.
- `//title` steps to the sections with that title at any depth in the current list, direct members included.
- `[n]` after a title keeps the `n`th section that step matched. `[1]` is the first and `[-1]` is the last.
- Only a bracketed integer at the very end of a step is an index. `/[Draft] Notes` and `/Notes [old]` are plain titles. A title that itself ends in a bracketed integer escapes the bracket: `/Appendix \[2]`.
- A path with no leading `/`, or with an empty step such as `/Guide/` or `///a`, raises.
- There is no `..`. Plain data holds no parent, so a path that uses it raises.

### How a path is matched

Each step is applied to the whole tree at once.

1. The first step starts from `blocks`. A later step starts from the `blocks` of every section the step before it kept.
2. `/title` keeps the sections with that title that are direct members of a starting list. `//title` keeps those at any depth under a starting list.
3. The kept sections are in file order, the order their headings appear in the text. A section counts once, even when two starting lists both reach it.
4. `[n]` then keeps only the `n`th of them, or none when there are fewer.
5. After the last step, one section is the result. None is a miss. More than one raises.

Only the last step has to end with one section. A step in the middle may keep several.

```markdown
# A
## A
### B
## B
```

```lua
markdown.section(md, "/A/A/B").line     --> 3
markdown.section(md, "/A/B").line       --> 4
markdown.section(md, "//A/B")           --> raises   (both A sections have a B: lines 3 and 4)
markdown.section(md, "//A/B[1]").line   --> 3        (file order, not the order the A sections were found)
markdown.section(md, "//A/B[2]").line   --> 4
markdown.section(md, "//A//B[2]").line  --> 4        (the inner B counts once, though both A sections reach it)
```

The index differs from XPath in one way. It counts everything its step matched, in file order, so `//Items[2]` is the second `Items` in the text. XPath would count per parent.

### Selectors

A selector names a kind of block. `"fence markdown"` narrows `fence` to one language, compared with `lang` exactly and case-sensitively. A space separates the two because `lang` is the first word of the info string and so never contains one. Only `fence` takes a language. Whitespace around the selector and extra spaces between its two words are ignored.

```lua
"fence"            -- any fence, named or not
"fence markdown"   -- a fence whose lang is "markdown"
'fence ""'         -- a fence with no language, the one whose lang is ""
```

A bare kind matches every block of that kind, so `fence` alone cannot mean "unnamed". The unnamed case is written as the value `lang` holds for it. The cost is that a fence whose info string is literally two quote characters cannot be picked by language.

### Indexes

`1` is the first match and `-1` is the last, both for a path step's `[n]` and for the `n` that follows a selector. The selector's index is a number argument, not part of the selector string, so a loop variable passes straight in. It must be a number with an integer value, and not zero.

The selector's index is optional. Without one, the selector must match exactly one of the section's own blocks: no match is a miss, and more than one raises and says how many there are and on which lines. A path step without `[n]` follows the same rule. With an index, the selector may match any number of blocks, and the index picks among them.

An index with no selector raises.

### `section` and `section_or`

The two functions take the same arguments and find the same thing. They differ only when it is not there.

| Outcome | `section` | `section_or` |
|---|---|---|
| The path names no section | raises | `nil` |
| The selector matches no block, or fewer blocks than the index asks for | raises | `nil` |
| A `markdown` path matches several sections | raises | raises |
| The selector matches several blocks and no index is given | raises | raises |
| The path, selector, or index is malformed, or `blocks` is not a list of blocks | raises | raises |

The first two rows are things that are absent. The rest are mistakes in the call, or a choice that only an index can make, and both functions raise for them. The same table holds for `prompt.section` and `prompt.section_or`.

An index that runs past the last match is an absence, so a loop can count upward with `section_or` until it returns `nil`.

### `markdown.parse(text)`

**Effect:** Parses Markdown text into blocks by the parsing rules above. **Return:** A list of blocks, empty for empty text. **Failure:** A non-string argument, text that is not UTF-8, or text longer than 8,388,608 bytes raises. That is the limit `json.decode` has, for the same reason: the parse runs outside the Lua memory limit. No UTF-8 text within the limit raises. **Example:** `local md = markdown.parse(store.read("notes.md"))`

### `markdown.section(blocks, path)`

**Effect:** Finds the one section that `path` names, starting from `blocks`. **Return:** The section table itself, not a copy. **Failure:** A `blocks` that is not a list of blocks, a malformed path, a step index of zero, a path that names no section, or a path that matches more than one section raises. **Example:** `local setup = markdown.section(md, "/Guide/Setup")`

```lua
markdown.section(md, "/Guide").title                    --> "Guide"
markdown.section(md, "/Guide/Templates").line           --> 16
markdown.section(md, "/Guide/Templates/Extras").level   --> 3
markdown.section(md, "//Extras").line                   --> 34
markdown.section(md, "//Templates/Extras").line         --> 34
markdown.section(md, "/Guide//Extras").line             --> 34
markdown.section(guide.blocks, "/Usage").line           --> 42       (a path starts at the list you pass)
markdown.section(md, "/Templates")                      --> raises   (Templates is not at the top level)
markdown.section(md, "/guide")                          --> raises   (titles are case-sensitive)
markdown.section_or(md, "/Templates")                   --> nil
markdown.section_or(md, "/Guide/Nope")                  --> nil
markdown.section_or(md, "/Guide").title                 --> "Guide"
```

Parsed text may repeat a title among siblings, which a prompt may not. A step index picks one:

```markdown
# Log

## Entry

first

## Entry

second
```

```lua
markdown.section(md, "/Log/Entry")                       --> raises   (matches 2 sections, at lines 3 and 7)
markdown.section(md, "/Log/Entry[1]").line               --> 3
markdown.section(md, "/Log/Entry[2]").line               --> 7
markdown.section(md, "/Log/Entry[-1]").line              --> 7
markdown.section(md, "/Log/Entry[3]")                    --> raises   (there are two)
markdown.section_or(md, "/Log/Entry[3]")                 --> nil
markdown.section(md, "/Log/Entry[2]", "prose").text      --> "second"
```

### `markdown.section(blocks, path, selector, n?)`

**Effect:** Finds the section as above, then picks one block matching `selector` among that section's own `blocks`: the only match when there is no `n`, and the `n`th match when there is. Blocks inside its subsections are not counted. **Return:** The block table itself. **Failure:** Everything the two-argument form raises for, plus an unknown kind, a language on a kind other than `fence`, no matching block, fewer than `n` matches, more than one match when no `n` is given, an index with no selector, and an index that is zero or not an integer. **Example:** `local body = markdown.section(md, "/Guide/Templates", "fence markdown").text`

Without an index, the selector names the one block of its kind:

```lua
markdown.section(md, "/Guide/Templates", "fence markdown").text   --> "# Hello"
markdown.section(md, "/Guide/Templates", "fence lua").text        --> "return 1"   (the other lua fence is in Extras)
markdown.section(md, "/Guide/Templates", "section").title         --> "Extras"
markdown.section(md, "/Guide", "list").items[2]                   --> "beta"
markdown.section(md, "//Extras", "fence").text                    --> "return 2"
markdown.section(md, "/Guide/Templates", "list")                  --> raises   (it holds no list)
markdown.section(md, "/Guide/Templates", "fence")                 --> raises   (3 fences, at lines 20, 24, and 30)
markdown.section(md, "/Guide/Templates", "prose")                 --> raises   (2 prose blocks, at lines 18 and 28)
```

With an index, it picks among several:

```lua
markdown.section(md, "/Guide/Templates", "fence", 3).text           --> '{ "ok": true }'
markdown.section(md, "/Guide/Templates", "fence", -1).lang          --> "json"
markdown.section(md, "/Guide/Templates", "prose", 2).text           --> "Then the fallback:"
markdown.section(md, "/Guide/Templates", "fence markdown", 1).text  --> "# Hello"
markdown.section(md, "/Guide/Templates", "fence", 4)                --> raises   (there are three)
```

To take every direct member of one kind, loop over the section's `blocks` with `ipairs` and test `kind`. The data is plain, so no function is needed for that.

### `markdown.section_or(blocks, path)` and `markdown.section_or(blocks, path, selector, n?)`

**Effect:** The same lookup as `markdown.section`, in both forms. **Return:** What `markdown.section` returns, or nil when the section or the block is not there. **Failure:** Everything else `markdown.section` raises for. **Example:** `local abstract = markdown.section_or(paper, "//Abstract")`

```lua
markdown.section_or(md, "/Guide/Templates", "fence markdown").text   --> "# Hello"
markdown.section_or(md, "/Guide/Templates", "list")                  --> nil      (it holds no list)
markdown.section_or(md, "/Guide/Templates", 'fence ""')              --> nil      (every fence there has a language)
markdown.section_or(md, "/Guide/Templates", "fence lua", 2)          --> nil      (the second lua fence is in Extras)
markdown.section_or(md, "/Guide/Nope", "fence")                      --> nil      (no such section)
markdown.section_or(md, "/Guide/Templates", "fence")                 --> raises   (three fences is a choice, not an absence)
```

### `markdown.select(blocks, selector)`

**Effect:** Collects every block matching `selector` at any depth, in file order, with a section before its members. **Return:** A new list that holds the matching block tables themselves, empty when nothing matches. **Failure:** A `blocks` that is not a list of blocks, or a malformed selector, raises. **Example:** `local fences = markdown.select(md, "fence lua")`

```lua
#markdown.select(md, "section")                      --> 4   (Guide, Templates, Extras, Usage)
#markdown.select(md, "fence")                        --> 4
#markdown.select(md, "fence lua")                    --> 2
markdown.select(md, "fence lua")[2].text             --> "return 2"
markdown.select(md, "list")[2].ordered               --> true
#markdown.select(md, "fence python")                 --> 0
#markdown.select(md, "frontmatter")                  --> 1
markdown.select(templates.blocks, "prose")[1].text   --> "Use these."
```

### `markdown.render(blocks)`

**Effect:** Writes a list of blocks back as Markdown, each block in one fixed style. Blocks are joined by one blank line, and the result has no trailing newline. **Return:** A string, empty for an empty list. **Failure:** A value that is not a list of blocks raises, and so does a block that cannot be written: an unknown `kind`; a field that is missing or of the wrong type; a `level` that is not an integer from 1 to 6; a `title`, `info`, or `lang` that holds a line break; a `list` with no items; and a `frontmatter` that is not the first block or whose `data` is not a record. The message names the block, such as `blocks[2].text`. **Example:** `local text = markdown.render(setup.blocks)`

| Block | Written as |
|---|---|
| `frontmatter` | `---`, the data as YAML with its keys sorted, `---`. |
| `section` | `#` repeated `level` times, then a space and the title when the title is not empty, then its `blocks`. |
| `prose` | `text`, unchanged. |
| `fence` | A backtick fence one longer than the longest run of backticks in `text`, and at least three; then `info`, or `lang` when there is no `info`; then `text`; then the closing fence. An empty `text` puts no line between the two fences. A tilde fence is used instead when the info string holds a backtick. |
| `list` | `- item` for each item, or `1. item` numbered from 1. The later lines of an item are indented to sit under its first, and an empty item is its marker alone. A list right after a list of the same type is written with `*`, or with `1)`, so the two stay separate lists. |
| `rule` | `---`. A rule that is the first block is written `***`, so the output never opens with a line that could start frontmatter. |

```lua
markdown.render(guide.blocks[4].blocks)   --> "1. first\n2. second"
markdown.render({ guide.blocks[4] })      --> "## Usage\n\n1. first\n2. second"
```

A section's `blocks` render without its heading. Wrapping the section in a list renders the heading too.

`render` does not escape. Text that came from a parse is written back as it was read, so for blocks that `markdown.parse` produced, parsing the output gives the same blocks apart from `line`. The one known exception is a title whose text Markdown reads as markup: a heading written `## \*x\*` has the title `*x*`, which is written as `## *x*` and read back as `x`.

## `prompt`

`prompt` reads the running prompt. A section is named by a [section path](#section-paths): a relative path for a neighbor, or a path from the top for any section.

```lua
prompt.section(path)                    --> string; raises when there is no such section
prompt.section(path, selector, n?)      --> block; raises when there is none   (new)
prompt.section_or(path)                 --> string, or nil                     (new)
prompt.section_or(path, selector, n?)   --> block, or nil                      (new)
```

The selector forms mirror `markdown.section(blocks, path, selector, n?)` argument for argument, and return the same block tables. They are how a prompt reads a template kept in a fence, the items of a list, or any other single block.

### What a read returns

A read returns a section's own content: the text after its heading line up to the next heading of any level. A subsection is a section of its own, and it is read by its own path. So `prompt` returns a section as text, and never as a table with its subsections inside.

Any section can be read, the running section and its ancestors included. `prompt` has no read for the H1's text or the frontmatter.

### `prompt.section(path)`

**Effect:** Reads a section's own raw Markdown: the text after its heading line up to the next heading of any level, with leading blank lines and trailing whitespace trimmed. It is never run or substituted. **Return:** A string, empty for an empty section. **Failure:** A non-string or malformed path, or a path that names no section, raises. **Example:** `local rules = prompt.section("/Rules")`

### `prompt.section(path, selector, n?)`

**Effect:** Resolves `path` as above, parses that section's own raw Markdown by the block rules, and picks one block matching `selector`: the only match when there is no `n`, and the `n`th match when there is. **Return:** A block table built for this call. Its `line` counts lines of the prompt file. **Failure:** Everything `prompt.section(path)` raises for, plus a malformed selector, no matching block, fewer than `n` matches, more than one match when no `n` is given, an index with no selector, and an index that is zero or not an integer. The kinds `section` and `frontmatter` also raise, because a section's own content never holds either. A subsection is named by its own path. **Example:** `local template = prompt.section("/Templates", "fence markdown").text`

The result is the block that the same pick over `markdown.parse(prompt.section(path))` gives, with two differences. `line` counts lines of the prompt file instead of lines of the section. And frontmatter is never recognized, because a prompt's frontmatter is at the top of the file and never in a section.

A Lua fence is a `fence` like any other: its `lang` is `lua`, and its `text` is the source, which is not run. The sandbox has no way to load code from a string, so reading a section's Lua never lets a caller run it.

### `prompt.section_or(path)` and `prompt.section_or(path, selector, n?)`

**Effect:** The same read as `prompt.section`, in both forms. **Return:** What `prompt.section` returns, or nil when the section or the block is not there. **Failure:** Everything else `prompt.section` raises for. **Example:** `local extras = prompt.section_or("Extras") or ""`

A name built at run time can act as a key: `prompt.section_or("/Vendors/" .. vendor)` is `nil` for a vendor with no section. The name is a path step like any other. A `/` or a `\` in it must be escaped, and an empty name makes the path malformed, which raises.

### Lists

A list is a `list` block, wherever it sits in a section. The language has no other notion of a list.

```lua
local topics = prompt.section("Topics", "list").items
```

- The items follow the block rules: the markers are removed, a nested bullet stays in its parent's item, and a bullet with no content is the item `""`.
- A list beside prose is found, because the selector picks the list and not the section.
- A section with two lists needs an index, such as `prompt.section("Topics", "list", 2)`.
- A section that holds only a list is an ordinary section. Named by `call`, or given to `tasks.fanout` as the worker, it is entered like any section with no Lua, and it runs nothing of its own.

### Worked example

````markdown
---
name: report
description: Write one report per topic
runvil: 0
---

# Report

## Write

```lua
local style = prompt.section("house_style")
local template = prompt.section("Template", "fence markdown").text
local topics = prompt.section("Topics", "list").items
```

### house_style

Short sentences. No hype.

### Template

Fill in this outline.

```markdown
# Title

## Summary
```

### Topics

- allocators
- coroutines

## Review

Check every claim.
````

Inside `Write`, whose children are `house_style`, `Template`, and `Topics`, and whose sibling is `Review`:

```lua
prompt.section("house_style")                          --> "Short sentences. No hype."
prompt.section("Template")                             --> "Fill in this outline.\n\n```markdown\n# Title\n\n## Summary\n```"
prompt.section("Template", "fence markdown").text      --> "# Title\n\n## Summary"
prompt.section("Template", "fence markdown").line      --> 25    (a line of the prompt file)
prompt.section("Template", "fence").lang               --> "markdown"
prompt.section("Template", "prose").text               --> "Fill in this outline."
prompt.section("Template", "fence", 1).lang            --> "markdown"   (an index is allowed when there is one match)
prompt.section("Topics", "list").items                 --> { "allocators", "coroutines" }
prompt.section("../Review")                            --> "Check every claim."
prompt.section("/Review")                              --> "Check every claim."   (the same sibling, from the top)
prompt.section("/Write/Template", "fence").lang        --> "markdown"   (the same child, from the top)

prompt.section("Missing")                              --> raises   (Write has no child with that title)
prompt.section("Review")                               --> raises   (Review is a sibling, not a child)
prompt.section("../Write")                             --> raises   (a section is not its own sibling)
prompt.section("Template", "list")                     --> raises   (Template holds no list)
prompt.section("Template", "fence", 2)                 --> raises   (Template has one fence)
prompt.section("Template/Summary")                     --> raises   (a relative path has one name)

prompt.section_or("Missing")                           --> nil
prompt.section_or("Missing", "fence")                  --> nil
prompt.section_or("Template", "list")                  --> nil
prompt.section_or("Template", 'fence ""')              --> nil      (the only fence has a language)
prompt.section_or("Template/Summary")                  --> raises   (a malformed path is not a miss)
```

The `## Summary` line inside the template fence is not a section of the prompt, because a fence is opaque.

One level down, inside `Template`:

```lua
prompt.section("../Topics", "list").items              --> { "allocators", "coroutines" }   (a sibling)
prompt.section("/Review")                              --> "Check every claim."   (the parent's sibling)
prompt.section("/Write", "fence lua").lang             --> "lua"   (its parent, by a path from the top)
prompt.section_or("../Review")                         --> nil     (Review is not a sibling of Template)
```

### Reading a shared section from any depth

A template that several sections use is read by its path from the top, which is the same string in every one of them.

````markdown
---
name: shared-template
description: Two workers read one template
runvil: 0
---

# Shared template

## Research

### Worker

```lua
local template = prompt.section("/Templates", "fence markdown").text
```

## Review

### Worker

```lua
local template = prompt.section("/Templates", "fence markdown").text
```

## Templates

```markdown
# Title
```
````

Inside the `Worker` under `Research`:

```lua
prompt.section("/Templates", "fence markdown").text    --> "# Title"   (a top-level section)
prompt.section("/Review/Worker", "fence lua").lang     --> "lua"       (any section, by its path from the top)
prompt.section_or("../Templates")                      --> nil         (Worker has no sibling with that title)
call("/Templates")                                     --> raises      (call takes no path from the top)
```

Inside `Research`, the same template is also a sibling:

```lua
prompt.section("../Templates", "fence markdown").text  --> "# Title"
prompt.section("/Templates", "fence markdown").text    --> "# Title"
```

### A library with its own sections

A library keeps its templates and workers under itself and names them by relative paths. Each line means the same thing whichever section is using the library.

```text
## Research
### Topics
## Review
## Library
### helper
### Template
### Worker
```

```lua
-- written in Library
local helper   = require("/Library/helper")
local template = prompt.section("Template", "fence markdown").text   -- Library's own child
local replies  = tasks.fanout("Worker", items)                       -- each arm runs Library's Worker
local topics   = _G.prompt.section_or("Topics", "list")              -- the user's child Topics, if it has one
```

The last line reads a section of whichever section is using the library, and the `_G` in front of it says so. Used from `Research` it gives the list under `Research`. Used from `Review`, which has no `Topics`, it gives `nil`, and `_G.prompt.section` in its place raises and names the user:

```text
section `Topics` not found: `/Review` has no child `Topics`
```

A mistake in one of the library's own paths names the library:

```text
section `Templat` not found: `/Library` has no child `Templat`
```

### How reading meets the rest of the language

- **The H1 body.** A relative path names a top-level section, since those are the H1's children. A path from the top names any section, nested ones included.
- **`call`, `jump`, and child walks.** Each section a chain enters reads from its own position. A read moves nothing, and the walk continues where it was.
- **Tasks and fanout arms.** An arm reads from the worker section it runs in. A worker under any parent reads a shared section by the same path from the top, and it reads its own parent the same way.
- **Libraries.** A library's code reads the library's own children and siblings by relative paths, and the running section's through `_G.prompt` (see [Where a path is resolved from](#where-a-path-is-resolved-from)).
- **`var`.** Handing text down in `var` still works. It copies the text into every arm and every later section entry, which a read does not.
- **`prose` and substitution.** A read returns the section's text as written. It renders no placeholders, and it does not consume the pending prose of the section it reads.
- **Tool handlers and the model.** A handler reads from the section it is written in, and a handler written in a library reads from the library. The model has no way to call `prompt.section`, so it sees nothing that an author did not pass on.
- **Pack preludes.** The environment a prelude runs in holds none of the functions that take a path, so a prelude's code names no section.
- **Running what was read.** Reading a section's Lua fence gives its source as a string. The sandbox cannot load code from a string, so a read never runs anything. `require` is the function that runs a section's Lua from anywhere, and it runs it as a library.
- **Events and replay.** A read performs no effect and emits no event. The prompt is fixed for the run, so the same read gives the same text on replay.
- **Errors.** A strict read raises the same not-found error that `call` raises, and `pcall` catches it the same way. Left uncaught, it ends the run as any Lua error does: with `lua_runtime` in a section, and as a failed gate in the H1 body. A read in the H1 body is therefore a check that the prompt has the sections it needs, made before any section runs.

### Using `markdown` on prompt text

`prompt.section(path)` returns text, and `markdown.parse` takes text, so the two compose.

```lua
local blocks = markdown.parse(prompt.section("Template"))
#blocks                                              --> 2
blocks[1].kind                                       --> "prose"
blocks[2].lang                                       --> "markdown"

-- the template is itself Markdown, so it parses too
local outline = markdown.parse(blocks[2].text)
markdown.section(outline, "/Title/Summary").level    --> 2
```

## One document, both namespaces

A prompt path and a `markdown` path are written differently over the same headings, on purpose.

```markdown
# H1

## H2

### Items

- a
- b

## H3

### Items

- c
```

As the running prompt (frontmatter and Lua left out), read through `prompt`:

| Running section | Call | Result | Why |
|---|---|---|---|
| `H2` | `prompt.section("Items", "list").items` | `{ "a", "b" }` | Its own child. |
| `H3` | `prompt.section("Items", "list").items` | `{ "c" }` | Its own child. |
| `H2` | `prompt.section("/H3/Items", "list").items` | `{ "c" }` | A path from the top names any section. |
| `H2` | `prompt.section("../H3/Items")` | raises | A relative path has one name. |
| the H1 | `prompt.section("/H2/Items", "list").items` | `{ "a", "b" }` | The same path means the same section from everywhere. |
| the H1 | `prompt.section("Items")` | raises | The H1's children are `H2` and `H3`. |
| the H1 | `prompt.section_or("Items")` | `nil` | The same miss, asked for as `nil`. |
| `Items` under `H2` | `prompt.section("../Items")` | raises | A section is not its own sibling. |
| `Items` under `H2` | `prompt.section("/H3")` | `""` | `H3` has no text of its own. |

The relative path `Items` names a different section from each caller. The path `/H2/Items` names one section from every caller.

As text given to `markdown.parse`:

```lua
markdown.section(md, "/H1/H2/Items").line               --> 5
markdown.section(md, "/H1/H3/Items").line               --> 12
markdown.section(md, "//H3/Items").line                 --> 12
markdown.section(md, "/H1/H2/Items", "list").items      --> { "a", "b" }
markdown.section(md, "/Items")                          --> raises   (no top-level section has that title)
markdown.section_or(md, "/Items")                       --> nil
markdown.section(md, "//Items")                         --> raises   (matches 2 sections, at lines 5 and 12)
markdown.section_or(md, "//Items")                      --> raises   (the same two matches)
markdown.section(md, "//Items[1]").line                 --> 5
markdown.section(md, "//Items[2]").line                 --> 12
```

Both can name every section. They differ in how:

- The H1's title is a step in a `markdown` path over the prompt's text, and is not a step in a prompt path. `/H1/H2/Items` there is `/H2/Items` here.
- Code in a prompt is written in some section, so a prompt path can be relative to it. Parsed text has no such place, so a `markdown` path with no leading `/`, such as `"H2/Items"`, raises.
- Sibling titles are unique in a prompt, so a prompt path always names at most one section and needs neither `//` nor `[n]`. Parsed text makes no such promise.
- `markdown.section` returns the section's table, subsections included. `prompt.section` returns the section's own text.

## Scope, identity, and copies

### Scope

A lookup sees only the list it is given. Passing a section's `blocks` narrows the search to that section, at any depth of nesting.

```lua
local guide = markdown.section(md, "/Guide")

markdown.section(guide.blocks, "/Usage").line       --> 42       (the path starts inside Guide)
markdown.section(guide.blocks, "/Guide")            --> raises   (a section is not inside its own blocks)
markdown.section_or(guide.blocks, "/Guide/Usage")   --> nil
#markdown.select(guide.blocks, "fence lua")         --> 2        (searched inside Guide only)
markdown.section(guide, "/Usage")                   --> raises   (pass guide.blocks)
markdown.section_or(guide, "/Usage")                --> raises   (the same mistake)
```

### Line numbers

`line` always counts lines of the text that was given to `markdown.parse`, never lines of a subset. It is a field stored on each block at parse time, and narrowing a lookup does not touch it. The number changes base only when a different string is parsed.

```lua
markdown.section(guide.blocks, "/Templates", "fence", 3).line   --> 30   (still a line of the whole text)
markdown.parse(templates.blocks[3].text)[1].line                --> 1    (a new parse counts from 1)
```

- `markdown.parse(block.text)` and `markdown.parse(item)` count from line 1 of that string.
- `markdown.parse(prompt.section(path))` counts from the first line of that section's text.
- The selector forms of `prompt.section` and `prompt.section_or` count lines of the prompt file.

### Identity

`markdown.section`, `markdown.section_or`, and `markdown.select` return the tables that are inside the list they were given.

```lua
markdown.section(md, "/Guide") == md[3]                  --> true   (the same table, not a copy)
markdown.select(md, "fence")[1] == templates.blocks[2]   --> true
```

So an edit made through a lookup's result is an edit of `md`, and a later `markdown.render(md)` writes it. The selector forms of `prompt.section` and `prompt.section_or` are the opposite: they build a new table on every call, and editing that table changes nothing else.

### Where bytes are copied

Lua strings are immutable and are passed by reference. Putting a block's `text` in another variable or table never copies it. Bytes are copied only when a string is created, which happens when text crosses from the Engine into Lua.

| Call | Bytes copied |
|---|---|
| `markdown.parse(text)` | Each block string once: every `text`, `title`, `lang`, `info`, item, and frontmatter value. A section holds no text of its own, so nesting adds nothing. The total is at most the size of `text`. |
| `markdown.section`, `markdown.section_or`, `markdown.select` | None. They return tables that already exist. `select` creates one new list to hold them. |
| `markdown.render(blocks)` | The one string it returns. |
| `prompt.section(path)`, `prompt.section_or(path)` | The section's Markdown, once per call. |
| `prompt.section(path, selector, n?)`, `prompt.section_or(path, selector, n?)` | The chosen block's strings, once per call: a fence's text, or each item of a list. Nothing else in the section becomes a Lua value. |

Lua also keeps a single copy of every string of 40 bytes or fewer. `kind` values, languages, and most titles are therefore shared across all blocks and all calls.

The `prompt` reads are not memoized. The prompt's text lives outside the Lua heap, so each call creates its strings again. Keep the result in a local when it is needed more than once. To read several blocks of one section, parse it once with `markdown.parse(prompt.section(path))` and index the result.

### Across sections

Every section entry gets a fresh VM, so a parse result does not survive a fall-through or a `jump`. Two things can carry it.

- **`var`.** `var.doc = md` stores a deep copy as JSON. Reading it back gives guarded views. Indexing and `ipairs` read through them, and so do the `markdown` functions. Lua's `#` and `pairs` do not, which is true of every table kept in `var`. Every later section entry rebuilds the whole copy, used or not.
- **A fanout collection.** `tasks.fanout("Worker", markdown.select(md, "section"))` gives each arm one section as its `item`.

Carrying the text, or the name of a store file, and parsing it where it is needed is usually cheaper than carrying the tables.

## Recipes

```lua
-- A document template kept in a top-level section, read from any depth and written to the store once
local template = prompt.section("/Schema", "fence markdown").text
if not store.exists("game.md") then
  store.write("game.md", template)
end

-- One task per item of a list kept in a child section
local topics = prompt.section("Topics", "list").items
local replies = tasks.fanout("Worker", topics)

-- An optional block: nil when the section or the fence is missing
local fence = prompt.section_or("Extras", "fence json")
local extras = fence and json.decode(fence.text)

-- Optional text with a default
local style = prompt.section_or("/house_style") or "Plain sentences."

-- A library shared by every section, loaded from any depth
local helpers = require("/helpers")

-- In a library: a function that fans out over the library's own Reviewer, for whichever section calls it
local function review(drafts)
  return tasks.fanout("Reviewer", drafts)
end

-- The JSON a model replied with
local reply = models.infer(prose)
local found = markdown.select(markdown.parse(reply), "fence json")[1]
local data = found and json.decode(found.text)

-- A section a model's reply may have left out, or written twice
local findings = markdown.section_or(markdown.parse(reply), "//Findings[1]")
local summary = findings and markdown.render(findings.blocks) or ""

-- One section of a stored document, as text for a model; it raises when the document has no Abstract
local paper = markdown.parse(store.read("paper.md"))
local abstract = markdown.render(markdown.section(paper, "//Abstract").blocks)

-- Each top-level section of that document
for _, block in ipairs(paper) do
  if block.kind == "section" then
    log(block.title)
  end
end
```

## Errors

Messages are written for a model to read: what was wrong, and what was required. The wording below is proposed.

In a prompt. `Write` is the section of the report prompt shown under `prompt`: its children are `house_style`, `Template`, and `Topics`, and its sibling is `Review`.

| Call | Message |
|---|---|
| `call(42)` | `section target must be a string, got integer` |
| `require(42)` | `require takes a section path such as '/helpers', got integer` |
| `call("Missing")`, `prompt.section("Missing")`, in `Write` | ``section `Missing` not found: `/Write` has no child `Missing` `` |
| `call("Review")`, in `Write` | ``section `Review` not found: `/Write` has no child `Review` `` |
| `jump("../Topics")`, in `Write` | ``section `../Topics` not found: `/Write` has no sibling `Topics` `` |
| `call("Template")`, in the H1 body | ``section `Template` not found: the top level has no section `Template` `` |
| `jump("../Write")`, in the H1 body | ``section `../Write` not found: the H1 has no siblings`` |
| `prompt.section("Templat")`, in the library `Library` | ``section `Templat` not found: `/Library` has no child `Templat` `` |
| `call(name)`, in the library `kit`, given the name `"Worker"` by its user | ``section `Worker` not found: `/kit` has no child `Worker` `` |
| `_G.prompt.section("Topics")`, in a library that `Review` is using | ``section `Topics` not found: `/Review` has no child `Topics` `` |
| `require("/Library/Templat")` | ``section `/Library/Templat` not found: `/Library` has no child `Templat` `` |
| `require("helpers")`, `require("../helpers")` | `section path "helpers" does not start with "/"; require takes a path from the top, such as "/helpers"` |
| `prompt.section("/Librar/Template")` | ``section `/Librar/Template` not found: the top level has no section `Librar` `` |
| `prompt.section("Template/Summary")` | `section path "Template/Summary" has more than one name; a relative path is a child's name, or "../" and a sibling's name` |
| `call("/Write")` | `section path "/Write" starts with "/"; only require, prompt.section, and prompt.section_or take a path from the top` |
| `jump("..")` | `section path ".." needs a name after ".."` |
| `prompt.section("Topics[1]")` | `section path "Topics[1]" has an index; a path in a prompt takes none` |
| `call("Template/")` | `section path "Template/" has an empty step` |
| `require("/a")`, where `a` requires `b` and `b` requires `a` | `require cycle: /a -> /b -> /a` |
| `require("/Main")`, in `Main` | `require cycle: /Main -> /Main` |
| `tasks.fanout("Worker", nil)` | `tasks.fanout's second parameter is a collection, got nil; for the items of a list in a section, pass prompt.section(path, "list").items` |

Every function that takes a prompt path raises the same not-found error, with the same text for the same path searched from the same section. `prompt.section_or` returns `nil` in its place and raises all the others.

`markdown` and the selector:

| Call | Message |
|---|---|
| `markdown.parse(42)` | `markdown.parse expects a string, got number` |
| `markdown.parse(text)`, text too long | `markdown.parse: input exceeds the 8388608-byte limit` |
| `markdown.parse(text)`, text not UTF-8 | `markdown.parse: text is not valid UTF-8` |
| `markdown.section(guide, "/Usage")` | `markdown.section: blocks must be a list of blocks, got a section; pass its blocks field` |
| `markdown.section("text", "/Usage")` | `markdown.section: blocks must be a list of blocks, got string` |
| `markdown.section(md, "Guide/Setup")` | `markdown.section: a path starts with "/", got "Guide/Setup"` |
| `markdown.section(md, "/Guide/../Usage")` | `markdown.section: path "/Guide/../Usage" uses ".."; parsed text has no parent to step to` |
| `markdown.section(md, "/Guide/")` | `markdown.section: path "/Guide/" has an empty step` |
| `markdown.section(md, [[/a\b]])` | `markdown.section: path "/a\b" has an unknown escape "\b"; the escapes are "\/", "\\", and "\["` |
| `markdown.section(md, "/Guide/Nope")` | `markdown.section: path "/Guide/Nope" matches no section; step 2, "Nope", matches nothing` |
| `markdown.section(md, "/Log/Entry[3]")` | `markdown.section: path "/Log/Entry[3]" matches no section; step 2, "Entry[3]", matches nothing` |
| `markdown.section(md, "//Nope")` | `markdown.section: path "//Nope" matches no section; step 1, "//Nope", matches nothing` |
| `markdown.section(md, "//Items")` | `markdown.section: path "//Items" is ambiguous; it matches 2 sections, at lines 5 and 12; add an index such as "//Items[1]"` |
| `markdown.section(md, "/Log/Entry[0]")` | `markdown.section: path "/Log/Entry[0]" has index 0; an index is a nonzero integer` |
| `markdown.section(md, "/Guide", "fences", 1)` | `markdown.section: unknown kind "fences"; kinds are frontmatter, section, prose, fence, list, rule` |
| `markdown.section(md, "/Guide", "list lua", 1)` | `markdown.section: only a fence selector takes a language, got "list lua"` |
| `markdown.section(md, "/Guide/Templates", "fence")` | `markdown.section: selector "fence" matches 3 blocks, at lines 20, 24, and 30; add an index, such as 1 for the first` |
| `prompt.section("Topics", "list")`, in a section with two lists | `prompt.section: selector "list" matches 2 blocks, at lines {lines}; add an index, such as 1 for the first` |
| `markdown.section(md, "/Guide/Templates", "list")` | `markdown.section: no block in "/Guide/Templates" matches selector "list"` |
| `markdown.section(md, "/Guide/Templates", "fence", 4)` | `markdown.section: selector "fence" matches 3 blocks in "/Guide/Templates"; there is no block 4` |
| `prompt.section("Template", "list")` | ``prompt.section: no block in section `Template` matches selector "list"`` |
| `prompt.section("Template", "fence", 2)` | ``prompt.section: selector "fence" matches 1 block in section `Template`; there is no block 2`` |
| `markdown.section(md, "/Guide", nil, 1)` | `markdown.section: an index needs a selector before it` |
| `markdown.section(md, "/Guide", "fence", 0)` | `markdown.section: index must be a nonzero integer, got 0` |
| `markdown.render({ { kind = "prose" } })` | `markdown.render: blocks[1].text must be a string, got nil` |
| `prompt.section("Template", "section")` | `prompt.section: a section's own content has no subsections; name a subsection by its own path` |
| `prompt.section("Template", "frontmatter")` | `prompt.section: a section's own content has no frontmatter` |

The rows about a section or a block that is not there are the misses: "matches no section", "no block", and "there is no block". `section_or` returns `nil` for each. Every other row raises from `section_or` too, and the message then starts with `markdown.section_or` or `prompt.section_or`.

A failure of the Lua memory limit while a result is built passes through unchanged, as it does for `json.decode`.

## Decisions

- **One path syntax for the whole language.** Titles separated by `/`, with the same escapes and the same title match everywhere. Vinnie: "there is a benefit to having just one path syntax". A `markdown` path starts at the top of the text, and a relative prompt path starts at the section its code is written in.
- **A neighbor is named by a relative path, with two forms.** `name` is a child and `../name` is a sibling. The level is not in the address, so moving a section to another depth changes no path. Vinnie: "I'm not entirely thrilled with "### Template" in general, because if you ever want to cut and paste code to a different nesting level, now all the labels are invalid", and "I was reluctant to invent a syntax that distinguishes siblings from children". `../` is that syntax, borrowed from file paths, and the two forms are exactly the visible set.
- **It is a language change.** Every function that names a section takes a path: `call`, `jump`, `tasks.spawn`, `tasks.fanout`, `require`, and the `prompt` reads. They resolve over the same section tree, so one of them keeping another form would leave two address styles in one language. Vinnie: "Yes this would be a comprehensive language change."
- **`require` and the reads take a path from the top.** They load code or text and leave the walk where it is. A shared library or template keeps one address at any depth. Vinnie: "You are right about `require` being absolute".
- **`require` takes only a path from the top.** A library then has one spelling in every section, a `require` means the same on every line, and nesting the caller deeper breaks nothing. With relative forms, one top-level library would be `lib` from the H1 body, `../lib` from a top-level section, and `/lib` from a nested one. A count of the `require` calls in the guide's example prompts and in the prompt files of the workspace found 45 that resolve. 40 of them name a top-level section, and the four in real prompts name one library from three depths. Vinnie: "should we simply have `require` only accept absolute paths?", and after the costs were listed, "yes this sounds right". The reads keep the relative forms, because a section reads its own children and siblings by them, and so does a library.
- **A path from the top names any section.** Jason: "wouldn't this require you to put your templates in the same section where they are used? so this example would fail". His example read a top-level `Templates` from two nested workers. Vinnie: "yeah lets make that change. `section` can use the expanded path syntax", and then, "but I thought we just said we are going to loosen `prompt` so it can access any section?" `require` follows the same rule, so it names the same sections a read does. A rule for which sections are in and which are out would need a check on every path, and naming any section needs none. This sets aside an earlier position, that a read is held to the visible set. Vinnie had written, "the API for examining a prompt's markdown needs to follow this rule", and later, "I dont mind reversing the earlier "follow this rule" decision."
- **Functions that run a section take only the relative forms.** `call`, `jump`, `tasks.spawn`, and `tasks.fanout` move the walk or start one, and the visible set keeps that local to the section the path starts at. Vinnie: "A section can only see the names of its siblings and child sections", and "The rules are the same for call and jump".
- **A relative path resolves from the section its code is written in.** For a section's own blocks that is the running section, and for a library's code it is the library, at load time and afterward. A line of library code then means the same thing whichever section requires it. Vinnie had said of a library whose lookups depend on its user: "Context sensitivity, depending on where it's required from, that's almost like non-deterministic behavior." A library can keep its own templates and workers and name them the natural way. Under the first rule this design had, every relative path resolved from the running section, and three things followed: the natural line in a library named a section of its user, the mistake was silent when the user had a section of that name, and a library could not run a section of its own at all. Vinnie asked, "what about prompt.section from inside a library? is it relative to the library's sections at the point of `require` ?", and then, "we have the same problem if the library wants to use call or jump, or really anything that takes a section name". The rule was built and tested in a copy of the Engine before it was adopted, and Vinnie, on the advice to adopt it: "I agree".
- **One resolver, told where the code is written.** Vinnie: "every function which takes a section name has to call it". Every lookup goes through one resolver, which takes the section to start from. `require` gives a library's code its own `call`, `tasks`, and `prompt`, bound to the library, through the environment its blocks load with. Vinnie: "When `require` is issued, we inject a special prompts. table which wraps the real prompts table". Lua ties that environment to the code, so a function the library defines keeps it when it runs later, in a tail call too. Pack preludes are already loaded this way.
- **`jump` resolves from the running section.** A library has no walk of its own. `jump` moves the walk of the section that is running, so its target can only be a neighbor of that section. Refusing `jump` in a library's code would be a guard for a mistake the not-found error already reports.
- **Inside a library, `_G.prompt`, `_G.call`, and `_G.tasks` are the running section's.** This is how the environment works with no code added, and it keeps an earlier decision. Vinnie: "A library should be able to read the user's sections. And then that's on the user". The `_G` at the call shows a reader that the line depends on the user.
- **Nothing guards against a strange `require`.** A section may require itself or an ancestor. The cycle error already reports the loops. Vinnie: "I feel like its on the author. They will eventually get an error. The principle of "do more with less" applies to the errors. The recursion error catches the user's misuse. This is better than explicit prevention."
- **The not-found error lists no sections.** Vinnie: "I dont see a point to giving the "everything reachable" list. Never did."
- **The not-found error says where it looked.** It names the section it searched, by that section's path from the top, and whether it wanted a child or a sibling there. Two common misses would otherwise read as a bare "not found" for a section that is plainly in the file: a sibling written without `../`, and a name that a section passes to a library, which is searched in the library. The lookup already holds everything the message prints. Vinnie adopted it after a review of how hard each kind of failure is to diagnose.
- **`section` raises on a miss, and `section_or` returns nil.** A read that names nothing fails at the line of the read, with an error that says what is missing, instead of later as an index of `nil`. Every function that takes a prompt path then does the same thing on a miss, and the one that does not says so in its name. Vinnie: "What if we make `section` raise if it can't find it? and we provide a section_or which returns nil if not found?"
- **A missing block is a miss too.** In the selector form, `section` raises when the section is there and the block is not, and `section_or` returns nil for either. Several matches with no index raise from both, because that is a choice the call did not make, and not an absence.
- **`markdown` has the same pair.** `markdown.section` raises and `markdown.section_or` returns nil, so the two namespaces stay mirrored under one rule. Code that reads a model's reply, where a missing section is normal, calls `section_or`. Vinnie, on the pair and on the three points that came with it, which are its name, the missing block, and this one: "Lets do the section_or and I adopt your 3 things to settle".
- **No trace of anything removed.** Vinnie: "No, old-style strings must not raise. In fact, we should be specific in the design document: No trace of the old behavior should be left behind. Not in documentation ("it used to work like this") and not in code ("raise if...")." The rule was stated for the form with `#` markers, and it covers `prompt.list` and list sections the same way.
- **`prompt.list` is removed.** The selector form of `prompt.section` reads a list, a fence, or any other block with one syntax, and it is the syntax `markdown.section` has. Vinnie: "if you keep prompt.list then you also need prompt.markdown so that a caller can get a markdown "template"", and "But this seems too specific. Is there a more general syntax we can use, to name the list, and a fence, which mirrors what the markdown table interface says?"
- **List sections are removed with it.** The parser's list section existed to feed `prompt.list`. Vinnie chose to remove it entirely, with its parse error and its worker refusal, over keeping the two guards. A list is a `list` block and nothing else.
- **The selector's index is optional.** Without one, the selector must match exactly one block, which is the rule a path step without `[n]` follows. The common call gets shorter, and it checks the author's belief that the section holds one such block. Vinnie, on both removals and on this: "yes that sounds good".
- **`markdown.parse` is the `json.decode` of Markdown.** Text in, plain tables out, and the same input limit. Jason: "markdown.parse…nice, sounds like json.decode". Vinnie: "it does".
- **`markdown.parse` takes text, not a file name.** One string cannot mean both, and the VM has no file access. Vinnie: "Runvil creates the Lua VM in a restricted sandbox which disallows all file access", and on this point, "I agree taking text".
- **Plain data.** Tables with no methods can be encoded, stored in `var`, and fanned out. Vinnie: "I dont want to give up plain data." The cost is `md[3].blocks[1]` instead of `md[3][1]`, and no indexing sugar on `md`.
- **A list is a block.** Lists are the data authors most often want out of Markdown, and a kind added later would be carved out of `prose` and shift existing indexes. Vinnie: "right, a list is a block".
- **A `markdown` path that matches more than one section raises.** An ambiguous address never resolves to an arbitrary first hit, in `section` or in `section_or`.
- **A `markdown` path step takes an index, `[n]`.** A prompt cannot repeat a sibling title, because the prompt parser rejects it, so a prompt path needs no index. Parsed text can. Vinnie: "however markdown.parse has to work generally", and "Yes the [] looks needed". The index is part of the string because it can sit on any step, including one in the middle of a path. It counts in file order and accepts a negative number, like the selector's index.
- **The selector and its index are arguments, not path steps.** Every path step is then a title, so a section named `list` or `rule` cannot collide with a kind. The selector's index stays a number. The same selector and index follow a path in `markdown` and in `prompt`. Jason, on that form: "It reads well, give me this prompt section, with all the markdown fences, then give me the one at index 1".
- **`fence ""` selects a fence with no language.** `fence` alone has to keep matching every fence, as a bare kind does for every kind, so the unnamed case needs its own spelling. Any word in the language position could be a real language, so the spelling is the empty string that `lang` holds.
- **Selectors count a section's own blocks only.** This matches what `prompt.section(path)` already treats as a section's content. A subsection's blocks are read by naming the subsection.
- **`//` is included in `markdown`.** A model reply may or may not wrap its sections in an H1, and a stored document's H1 title is usually not known in advance. `//Findings` covers both.
- **A lookup is scoped to the list it is given, and `line` never changes.** Scope comes from the argument, so there is no second way to narrow a search. `line` stays tied to the parsed text, so a block reports the same line however it was reached.
- **Lookups return the tables that are there.** Vinnie: "i am assuming that the strings in Lua will be shared (i.e.. not duplicated needlessly, when calling section())". They are: `markdown.section`, `markdown.section_or`, and `markdown.select` create no strings and copy no tables.
- **The `prompt` reads are not memoized.** A section entry's VM is short-lived and a local variable already holds a result. A cache would keep every string it ever returned alive until the VM ends.
- **Line endings are normalized and nothing else is.** A `\r\n` file and a `\n` file give the same strings, which keeps comparisons and line counts predictable. All other text is kept as written.
- **Parsing does not fail on content.** The prompt parser rejects skipped levels and repeated sibling names because a prompt is a program. `markdown` reads data.

## Rejected alternatives

- **A reference that carries the heading level**, in a prompt or in `markdown`. Jason's gist proposed it for `markdown`, alone or in a list: `markdown.section(md, { "# Guide", "## Templates" })`. Vinnie: "the syntax in your gist creates ambiguities". The level only restates where a section sits, so moving the section breaks every reference to it. In parsed text the level must be known in advance, so a reply that writes `# Abstract` where `## Abstract` was expected is not found.
- **Detecting such a string and raising.** A check that names the form keeps it alive in the code and in an error message. Vinnie: "No, old-style strings must not raise."
- **A path from the top for the functions that run a section.** Loading text or code leaves the walk where it is. Running a section far from the caller does not, and the walk would stop being readable from the section tree.
- **Reads held to the visible set.** A template shared by sections under different parents would have to be copied under each parent, or read by a parent and handed down in `var`, which copies it into every arm and every later section entry.
- **A middle set for `require` and the reads: children, siblings, and the siblings of each ancestor.** It covers the shared library and the shared template, but it needs a rule for what is in and what is out, a check on every path from the top, and a list of exceptions to explain.
- **Resolving every relative path from the running section**, in a library's code too. It was this design's first rule, chosen because it needs the least Engine code. Vinnie: "I want to eliminate special cases, because I want the engine to be the smallest amount of code possible. I want a small kernel of code." Under it the natural line in a library, a relative path to its own child, named a section of the library's user, and the mistake was silent when the user had a section of that name. A library could read its own sections only by paths from the top, which break when the library moves, and it could not run one of them at all. The adopted rule costs the Engine about 250 lines.
- **Resolving from the library only while it loads.** `require` works this way today. The same line would name the library's child during the load and the user's child when a library function runs later.
- **Finding the code's section from the call stack.** `return tasks.fanout("Worker", items)` is a tail call, so the library function's frame is gone before the lookup runs.
- **Keeping the resolver in `_G`.** Vinnie: "in the global table _G we put an __impl function called the "section path resolver"". A library's code and its user's code share one `_G`, so a resolver found there cannot tell which of them is asking. The one resolver stayed, and `require` binds the section it starts from.
- **Rewriting a library's relative paths into paths from the top.** `call` and the `tasks` functions take no path from the top, and the not-found error would lose the path as the author wrote it. The request carries the section to start from instead.
- **Refusing `jump` in a library's code.** It would guard against a mistake the not-found error already reports, and it would stop a library function from moving its user's walk, which works today.
- **Closing `_G` inside a library**, so that `_G.prompt` is the library's too. It takes more code, and it removes the one way a library reads a section of its user.
- **A guard against requiring yourself or an ancestor.** It is code for a mistake the cycle error already reports, and it forbids uses nobody has thought of yet. Vinnie: "Someone will at some point find a valid use for requiring the parent."
- **Listing the sections a caller could have named.** With a path from the top that list is every section in the prompt. A list of the visible set alone would leave out most of what `require` and the reads can name.
- **A not-found error that only repeats the path.** ``section `Review` not found`` reads the same for a misspelled title, for a sibling written as a child, and for a name a library looks up among its own sections.
- **Returning nil for every miss.** A misspelled path then fails later and somewhere else, as an index of `nil` or as an empty string in a prompt. The author who wants an optional read and the author who made a mistake write the same call.
- **A second argument that is a default**, as in `section_or(path, "")`. The selector and its index already follow the path, so a default has no place to go. Lua's `or` does the same job after the call.
- **A test such as `prompt.exists(path)` beside a strict read.** An optional read would resolve the path twice, once to test and once to read.
- **`find` as the name of the lenient read.** Lua's `string.find` returns nil when nothing matches, so the name fits. `section_or` was kept because its pairing with `section` is plain in a small API.
- **Relative paths for `require`.** `require("lib")` for a child and `require("../lib")` for a sibling read well for a library kept beside its user. They give one library several spellings, and `../lib` breaks when its caller moves deeper. Leaving them out costs a longer name for a library kept under its user, as in `require("/Main/lib")`, and a copy of `Main` that requires the original's library.
- **A path from the top as the only form for the reads too.** The rule would be shorter: run with a relative path, load with a path from the top. Text kept under the section that uses it would then be named through that section, and so would a library's own templates, so renaming or copying a section would break its reads.
- **More than one `..`**, such as `prompt.section("../../Templates")`. It names an ancestor's sibling, but it changes whenever the caller moves deeper. The path from the top does not.
- **A uniqueness rule in place of `../`.** If a child could never share a title with its parent's sibling, a bare name could mean either. That rejects prompts that parse today, and it hides whether a call goes down or sideways.
- **`./name` for a child.** It is a second spelling of `name`.
- **A `prompt.section` that picks its return by what the section holds.** Jason: "if a section only has one code block it's smart enough to know how to handle it". The return type would depend on the section's content, so adding a sentence above a fence would silently change what the call returns. The selector states the same wish explicitly.
- **`call` returning a markdown block as a string.** Jason: "you could call a section with a markdown codeblock, and it just returns the string". Vinnie: "call expects to run code". The read is `prompt.section(path, "fence markdown").text`.
- **`prompt.markdown` beside `prompt.list`.** It would be one function per kind of block, and the next kind would need another. The selector covers every kind with one function.
- **Keeping list sections and their two guards.** With `prompt.list` gone they would be a second notion of a list beside the `list` block, with a parse error and a refusal that nothing else in the language explains.
- **An index on every selector.** Most sections hold one block of the wanted kind, so `"fence markdown", 1` adds a number that says nothing, and it hides the case where a second block has appeared.
- **A selector without an index meaning the first match.** Which block comes back would depend on file order, and nothing would say there were two.
- **A section that is both an array and a record.** `md[3][1]` beside `md[3].title` reads well, but `json.encode` refuses a table that mixes list items and named keys.
- **Accepting a section where a list of blocks is expected.** `markdown.section(guide, "/Usage")` could mean "inside Guide" or "Guide itself". Raising keeps one meaning per call.
- **Selectors inside the path, such as `/Templates/fence[3]`.** A title and a kind would share one namespace, so a section titled `fence` could not be told from the kind. A loop over a section's fences would also need string building.
- **XPath's per-parent index.** In XPath, `//Items[2]` means every `Items` that is the second one under its own parent. Counting in file order is what an author expects from "the second `Items`".
- **First match wins.** Which section comes back would depend on file order, and nothing would say there were two.
- **A `text` field on every section.** It would make a section's source one field away, but each nesting level would hold another copy of the same bytes. `markdown.render(section.blocks)` gives the text when it is wanted.
- **Reading a file by name**, such as `markdown.parse("file.md")` or `load_text("other.md")`. The sandbox has no file access. Text comes from `store.read` or from `prompt.section`.

## Open questions

1. **A relative path longer than one name, for the `prompt` reads.** `prompt.section("Template/Variants")` would name a child's child, and it would keep working when the caller moves, which the path from the top does not. It is left out: a relative path has one name everywhere.
2. **A relative `require` inside a library.** A library names a helper library under itself as `require("/Library/helper")`, the one line in a library that breaks when the library is renamed. A relative form bound to the library, as `call` is, would close that. It is left out: a library has one spelling.
3. **Other XPath forms in `markdown`.** `*` for any one section is left out.
4. **Whether `markdown.render` ships in the first version.** Nothing else here depends on it.

## Implementation notes

### What exists today

- `prompt.section` and `prompt.list` are installed by `install_prompt_table` in `crates/runvil-internal/lua/src/vm/install.rs`, over one read callback. The callback is `read_visible` in `crates/runvil-internal/runtime/src/execute/walk_target.rs`, which returns a `SectionText` holding the section's Markdown and its list items.
- Every section lookup goes through `crates/runvil-internal/runtime/src/heading_address.rs`: one address parser, `find_visible` for the visible set, and `find_reachable` for `require`. `walk_target.rs` calls them for `jump`, `call`, and spawn targets (`resolve_jump_target`), for the reads (`read_visible`), and for libraries (`resolve_library`). `scheduler/walk.rs` calls them for the H1.
- A read resolves over the visible set and returns nil on a miss. `require` resolves over a wider set, the `Reachable` view: a section's children, its siblings, and the siblings of each ancestor. A `require` made while a library loads resolves from that library's position, which `RequireOrigin::from_key` rebuilds from the library's key.
- A miss from `call`, `jump`, a `tasks` function, or `require` raises through `make_not_found_error`, which lists every section the caller could have named.
- `call`, `tasks.spawn`, and `tasks.fanout` are Lua shims in `crates/runvil-internal/lua/src/` (`__impl_coro.lua`, `__impl_tasks.lua`, `__impl_fanout.lua`). Each yields a request table, and the scheduler resolves the request's target from the chain's current section, in `resolve_chain_target` in `runtime/src/execute/scheduler/walk.rs`. A read is bound to the entered section's position when its VM is built.
- A library's blocks load into the VM's one environment (`lua/src/require.rs`), so a library's code and the code of the section using it share every global, the functions that take a section among them. Pack preludes load with an environment of their own (`lua/src/prelude.rs`), and the functions a prelude defines keep it when they run later.
- Sibling titles are checked by `check_unique_sibling` in `crates/runvil-internal/prompt/src/sections.rs`. It is the only uniqueness check, and it is what lets a path name at most one section.
- The list-or-record rule is `shape` in `crates/runvil-internal/lua/src/json/keys.rs`. Reading through guarded views is `Encoder::resolve` in `json/encode.rs`, where the 127-level depth limit also lives. The 8,388,608-byte input limit is `INPUT_LIMIT` in `json/decode.rs`.
- Engine global names are listed in `RESERVED_NAMES` in `crates/runvil-internal/lua/src/globals.rs`. `markdown` becomes one more entry.
- The VM is Lua 5.5 through `mlua` 0.12. `pulldown-cmark` 0.12 and `serde_yaml_ng` 0.10 are workspace dependencies that `runvil-prompt` uses. `runvil-lua` depends on neither.

### What section paths change

The lookup is already in one place, so the change is small in code and large in text.

- **The lookup.** The address parser in `heading_address.rs` becomes the path parser. A child is looked up among the caller's children and a sibling among the caller's siblings, each by title alone. A path from the top is walked from the top-level sections, one title per step. The reads accept one, and `require` accepts nothing else. `resolve_jump_target` already tells a child target from a sibling one. The module, its functions, and the `heading` parameters along the way are renamed.
- **What the Engine prints.** `Library::heading` (the cycle error and the load reports read it) and the `library` field of the three library-load events in `crates/runvil-internal/types/src/event.rs` carry a path from the top. The text of `require`'s type error in `lua/src/require.rs` names a path. `make_not_found_error` no longer takes the sections to list. It takes the section that was searched and the title that was wanted there, which the lookup already holds: the section a relative path started at, and the last section found for a path from the top.
- **Prompts.** `runvil/prompts/`, `crates/workshop/agents/agents/chat.md`, the test prompts, and the prompts in other repositories.
- **Tests.** A single-line search finds about 420 references in 78 files under `runvil/crates`.
- **The guide.** `runvil-docs/src/language/` holds about 440 references across 13 of its 17 chapters. Chapter 8 teaches the lookup and the visible set, and chapter 3 teaches `require`'s reach.
- **The language spec** in this repository.

Each rewrite follows from the section the reference names today. A `require` becomes that section's path from the top, wherever the `require` is written. One made while a library loads names the section it resolves to from the library. Any other reference becomes `name` when it names a section one level below the caller, and `../name` when it names one at the caller's own level. A reference in a library's code that names a section of the library's user is written through `_G`.

### What removing `prompt.list` and list sections changes

- **The read.** `install_prompt_table` no longer installs `prompt.list`. `SectionText` in `lua/src/section.rs` loses its `items`, and `read_visible` stops building them.
- **The parser.** `Section::items`, `Section::is_list`, and `list_items` in `crates/runvil-internal/prompt/src/` go, and so does all of `list.rs`. A section's blocks are unchanged.
- **The parse error.** `ErrorCode::ListEmptyItem`, number 104, is removed from `crates/runvil-internal/types/src/error.rs`, and the number is not used again. Its `parse_list` mapping in `crates/workshop/server/src/error.rs` and its arm in `lua/src/error_value.rs` go with it. Both facades name the variant in their `public-api.txt` listings, so both listings change.
- **The worker refusal.** The `is_list` check in `runtime/src/execute/scheduler/tasks.rs` is deleted.
- **The collection error.** Its text in `lua/src/collection.rs` names the read that gives a list's items.
- **Prompts and tests.** A search finds `prompt.list` 37 times in 13 files under `runvil/crates`, four fanout test prompts among them. `crates/runvil-engine/tests/prompts/invalid/list-h3-non-list-content.md` exists only to trigger the parse error, and it is deleted.
- **The guide and the spec.** `prompt.list` appears 63 times in 7 chapters of the guide and 5 times in the language spec. The guide also teaches list sections in chapter 3 and lists their parse error in chapters 16 and 17.

Two results differ for prompts that exist today, because items now follow the block rules. A nested bullet is part of its parent's item, where it was an item of its own. And a bullet with no content is the item `""`, where it failed the parse.

### What `require` and the reads change

This part removes more code than it adds.

- **Two lookups, both small.** A read resolves a relative path over the visible set, as `call` does, and walks a path from the top from the top-level sections. `require` only walks from the top. `Reachable`, `find_reachable`, and `resolve_reachable` in `heading_address.rs` are deleted, with the scope-by-scope iteration behind them.
- **`require` has no starting point.** `resolve_library` in `walk_target.rs` needs only the prompt and the path. `RequireOrigin` is deleted, and `library_resolver` in `crates/runvil-internal/runtime/src/execute/section_context.rs` is built the same way for a walked section and for the H1. The body of `RequireOrigin::from_key`, which turns a library's key into its position, stays as a function of its own, because a library's lookups start there. The resolver that `lua/src/require.rs` installs takes the path alone, where it takes `(String, Option<LibraryKey>)` today, and its `Loading` stack keeps each library's path for the load reports and nothing else.
- **The loader is unchanged.** `__impl_require.lua` still caches a library under its identity, which is the section's position, so a library required twice in one section loads once, however its path is spaced. The cycle check works as it does today, and it is what stops a section that requires itself.
- **The two reads.** `install_prompt_table` installs `prompt.section` and `prompt.section_or` over the one read callback. On a miss the first raises the not-found error and the second returns nil. The selector forms add the block lookup, where the same split applies.
- **Tests.** Under `runtime/src/execute/tests/suite/`, `require/reach.rs` asserts that a read cannot see what `require` reaches and that a load resolves from the library, and `require/failures.rs` and `exec_flow/prompt_reads.rs` assert the list. The tests in `heading_address.rs` assert which sections are out of reach. They are replaced by tests of the two kinds of path.
- **The guide.** Chapter 3 teaches a missed read as `nil` and how that differs from `call`. It has sections on which sections `require` can reach, on requires made while a library loads, and on what a miss lists. One statement replaces the first two: `require` takes a path from the top. Its `require_position` and `own_helpers` examples are rewritten with such paths. Chapter 8 repeats the read rules beside `call` and `jump`. The not-found error is quoted with its list in six chapters. All of it is rewritten to describe the language as it is.

Three results differ for prompts that exist today:

- A read that names no section raises, where it returned nil. A prompt that relied on nil calls `prompt.section_or`.
- A library function can no longer `require` a section of whichever section is using it. Today a `require` made after the library has loaded resolves from the running section. Code that relied on that has its user require the section and pass the result in.
- A path from the top names sections that `require` and the reads could not name: the caller, its ancestors, and sections nested under other parents.

### What the library environment changes

A library's blocks load with an environment of their own, the facility Pack preludes use. This was built in a copy of `runvil` at commit `b0693699d`. The copy keeps today's section references, which the mechanism does not depend on.

- **The environment.** `require` builds one table for a library and loads each of the library's blocks with it (`lua/src/require.rs`, and a `load_in` beside `LuaProgram::load` in `lua/src/program.rs`). The table holds three names bound to the library: `call`, `prompt`, and `tasks`, which is the Engine's table copied with `spawn` and `fanout` bound. Its metatable sends every other global read and every global write on to the VM's globals table, and so through the `_G` guard. The metatable is protected, so a library's code cannot read or replace it.
- **The shims.** `call`, `tasks.spawn`, and `tasks.fanout` become factories over the library they are bound to, with none for a section's own code. The `call` and `spawn` requests carry the library's key (`lua/src/protocol/request.rs` and `protocol/parse.rs`).
- **The lookups.** The scheduler resolves a target from the position the request names, or from the chain's current section when it names none (`scheduler/walk.rs`). The read callback takes the same optional key (`section_context.rs`). The path stays as the author wrote it, so the not-found error can print it beside the section that was searched.
- **What is not bound.** `jump` and `require`. Everything else in a library's code is the VM's own: `sys`, `var`, `prose`, `messages`, `tools`, `models`, `store`, and `log` describe the section that is running.

What the Engine's tests showed:

- **Nothing else moved.** All 669 tests of `runvil-lua` pass, and 1110 of the 1112 tests of `runvil-runtime`. The two that fail assert the rule this one replaces: `prompt_reads_work_in_h1_and_a_library` and `a_load_resolves_from_the_library_and_a_later_call_from_the_caller`.
- **The rule holds.** 34 new tests pass. 23 drive a section VM, and 11 run whole prompts through the scheduler. In the whole prompts, the library and the section using it both have a child under the same heading, so a lookup from the wrong place returns the wrong text. With the one line that sets the environment taken out, the tests that assert the binding fail and return the caller's answers.
- **The `_G` guard still governs a library's code.** A library reads the caller's globals as they are at that moment, and its own globals land in the caller's. `argv` reads frozen and refuses assignment, and the H1 pass's repair of `argv` lands in the VM's globals. `messages` and `prose` refuse assignment. A metatable the author set on `_G` runs for a library's code, and its error names the library's line.
- **A request names the library however the call is made:** while the library loads, from a function called later, in a tail call, through `pcall`, from a tool handler, and through a bound function the library hands to its user. A function that a section passes to a library names no library.
- **A fanout from a library runs the library's worker** when the H1 body, a top-level section, or a nested section is using the library, and each arm runs in the library's subtree.
- **The size.** 17 files outside the tests changed, with 363 lines added and 109 removed, and the result passes clippy under the workspace's lints.
- **The cost.** A global read from a library's code takes about twice as long as from a section's own block, about 10 nanoseconds more on the machine that ran it. A loop that calls three functions through globals ran 14% to 18% slower, a loop that builds strings 4% slower, and a loop that keeps its functions in locals ran the same.

Three things an implementation should know:

- A library's code does not see a section's own replacement of `call`, `tasks`, or `prompt`, and assigning one of the three inside a library changes it for the library alone.
- `_ENV` in a library's code is the environment table, and `_G` is still the VM's globals.
- An uncaught miss raised by a tail call in a library function is placed on the line of the section's code that called the function, because the tail call drops the library function's frame. The lookup itself does not depend on the stack.

One result differs for prompts that exist today. A read, a `call`, or a `tasks` call in a library's code names the library's neighbors, where it named those of the running section. No prompt in the workspace depends on the old meaning: the one library in a real prompt, `## lib` in `agora-author-advocacy.md`, names no section. In the guide, chapter 3's `kid_reader` example, a library that reads its user's list, is rewritten with `_G.prompt`, and its `require_position` example is rewritten for the rule.

### Where the `markdown` code goes

- `runvil-prompt` depends on `runvil-lua`, so the block parser cannot live in `runvil-prompt`. It belongs in `runvil-lua`, or in a new crate below it. A new crate fits the repository's preference for more, smaller crates: it would hold the text-to-blocks parser, the path and selector grammars, and the writer behind `render`, with no `mlua` dependency. `runvil-lua` would hold the conversion to Lua tables, the walks over Lua tables, the `markdown` global (installed the way `json::install` installs `json`), and the selector forms of the `prompt` reads.
- `SectionText` needs the prompt-file line that the section's Markdown starts on, so that the selector forms can report `line`. `runvil-prompt` can record it where it already computes block lines.
- `read_visible` copies the section's Markdown into an owned string on each read. Carrying the shared prompt text with a range instead leaves each read copying only the bytes it returns: the whole text for the one-argument form, and the chosen block's strings for the selector form.

### How the parser works

This is the approach the prototype took, in about 250 lines for the parser.

1. Drop a byte order mark and turn `\r\n` and `\r` into `\n`.
2. Split frontmatter by the rule `split_frontmatter` in `runvil-prompt` uses, and keep it only when the YAML between the delimiters deserializes to a JSON object.
3. Make one pass over `Parser::new_ext(body, Options::empty()).into_offset_iter()`, counting depth. At depth 0, record each heading, fenced code block, list, and rule with its source range. Collect a heading's title from its text and code events, a fence's `text` from its text events, and a list's items from the item ranges.
4. Treat the text between two recorded ranges as prose. This is what keeps link definitions, which produce no events, and it is what makes rule 3 hold by construction.
5. Fold the flat sequence into sections with a stack: a heading closes every open section whose level is the same or larger.
6. Build each item string from the source. It starts at the item's first child event, and each later line loses that many columns of leading whitespace.

A `markdown` path is matched one step at a time, each step as a single walk of the tree in file order, so the kept sections come out ordered and counted once.

### Tests to carry over

- Every example in this document. The prototype ran the parse, path, selector, and render examples, and the `prompt` examples, with further cases.
- Rule 3: every non-blank line of the input lies in exactly one block.
- `markdown.parse(markdown.render(md))` equals `md` apart from `line`, for parse output. The prototype checked this on 56 inputs, and only the escaped-title case differed.
- The block parser and the prompt parser agree on where top-level headings are, for the same text.
- A result read back from `var` works with every `markdown` function.
- A path with `#` markers in it behaves as any other relative path that names nothing: `prompt.section_or("## Rules")` is `nil`, `prompt.section("## Rules")` and `call("## Rules")` raise the ordinary not-found error, and `require("## Rules")` raises the error every relative path raises there.
- From every section and from the H1, a path from the top reads and requires every section of the prompt: the caller itself, its ancestors, and the sections nested under other parents.
- `call`, `jump`, `tasks.spawn`, and `tasks.fanout` raise for a path from the top, and resolve the two relative forms over the visible set.
- `require` raises for every path that does not start with `/`, and names the same library from the H1, from every section, and from a library's code.
- A read, a `call`, a `tasks.spawn`, and a `tasks.fanout` in a library's code resolve a relative path from the library, while the library loads and when its functions run later, whichever section is using the library. The Engine prototype's 34 tests cover this bullet and the four after it.
- A function that a section passes to a library resolves from the section, and a tool handler written in a library resolves from the library.
- `jump` in a library's code resolves from the running section.
- Inside a library, `_G.prompt`, `_G.call`, and `_G.tasks` resolve from the running section.
- A library's code reads and writes every other global as a section's own block does: plain globals both ways, `argv`, `messages`, `prose`, and a metatable the author set on `_G`.
- A section that requires itself, and two sections that require each other, raise the cycle error.
- The not-found error has the same text from every function that takes a prompt path. It names the path, the section it searched, and whether it wanted a child or a sibling there, and it lists nothing.
- A miss on a relative path in a library's code names the library, and a miss through `_G` names the running section.
- In all four functions, a selector with no index returns the only match and raises when several match.
- For a section or a block that is not there, `section` raises and `section_or` returns nil. For a malformed path, selector, or index, and for several matches, both raise.
- A section that holds only a list parses and runs as an ordinary section, and a bullet with no content in it is the item `""`.
