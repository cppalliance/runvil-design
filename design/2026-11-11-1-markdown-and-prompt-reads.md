# The `markdown` Namespace and the `prompt` Reads

Status: proposed design, 2026-11-11. `prompt.section(heading)` and `prompt.list(heading)` exist today. Everything else in this document is new and not implemented. It fills in the "Deferred `markdown` namespace" paragraph of [The Runvil Prompting Language](architect-runvil-prompting-language.md).

The parsing rules, the path and selector rules, `render`, and the examples that go with them were checked against a throwaway prototype built on `pulldown-cmark` 0.12.2, the version the workspace pins. The `prompt` examples were checked as the same parse run over the sample prompt, and their visibility outcomes follow the existing lookup in `heading_address.rs`. The prototype is not part of this repository.

## Contents

1. [Summary](#summary)
2. [Design rules](#design-rules)
3. [The block model](#the-block-model)
4. [`markdown`](#markdown)
5. [`prompt`](#prompt)
6. [One document, both namespaces](#one-document-both-namespaces)
7. [Scope, identity, and copies](#scope-identity-and-copies)
8. [Recipes](#recipes)
9. [Errors](#errors)
10. [Decisions](#decisions)
11. [Rejected alternatives](#rejected-alternatives)
12. [Open questions](#open-questions)
13. [Implementation notes](#implementation-notes)

## Summary

Lua gets two ways to read Markdown, and they share one block model.

- `markdown` parses any text into plain Lua tables, the way `json.decode` does for JSON. It names a section by path, such as `"/Guide/Setup"`.
- `prompt` reads the running prompt. It names a section by heading reference, such as `"## Setup"`, and only among the sections the caller can see.
- Both pick one block inside a section with the same selector and index, such as `"fence markdown", 1`.

| | `markdown` | `prompt` |
|---|---|---|
| Reads | any text passed to it | the running prompt |
| Names a section by | path: `"/Guide/Setup"` | heading reference: `"## Setup"` |
| Can name | every section in the parsed text | the caller's siblings and direct children |
| A section comes back as | a table with its subsections inside | a string of its own raw Markdown |
| A block comes back as | the block table inside the parsed text | a block table built for the call |

## Design rules

1. **Plain data.** Every value is a string, number, boolean, or table. No table has methods, and each table is either a list or a record, never both. A parse result therefore passes `json.encode`, can be stored in `var`, and can be a `tasks.fanout` collection.
2. **Functions, not methods.** Every operation is a namespace function over those values, as with `models` and `tools`. There is one way to do each thing.
3. **Nil for absent, raise for mistakes in the call.** A section or block that is not there is `nil`. A malformed path, heading, selector, or index raises.
4. **Raw text.** Nothing is run or substituted. A `{{ name }}` placeholder comes back as written.
5. **No file access and no effects.** `markdown` takes text, never a file name. A file is read with `store.read` and then parsed. Every function here is pure: the same input gives the same result, and none of them performs an effect or suspends.
6. **Lookups share, they do not copy.** A `markdown` lookup returns the tables that are already inside the list it was given. It creates no strings.

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
markdown.parse(text)                          --> list of blocks
markdown.section(blocks, path)                --> section, or nil
markdown.section(blocks, path, selector, n)   --> block, or nil
markdown.select(blocks, selector)             --> list of blocks
markdown.render(blocks)                       --> string
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

A path is a subset of XPath over the section tree.

- `/title` steps to the sections with that title among the direct members of the current list.
- `//title` steps to the sections with that title at any depth in the current list, direct members included.
- `[n]` after a title keeps the `n`th section that step matched. `[1]` is the first and `[-1]` is the last.
- A title matches exactly and case-sensitively. The heading level plays no part: `/Guide` matches a top-level section titled `Guide` whether it is written `# Guide` or `## Guide`.
- Whitespace around the path and around each title is ignored, so `" / Guide / Templates "` is `/Guide/Templates`.
- A slash inside a title is written `\/`, and a backslash `\\`. In Lua source the long-bracket form avoids doubling: `[[/Guide/Input\/Output]]`.
- Only a bracketed integer at the very end of a step is an index. `/[Draft] Notes` and `/Notes [old]` are plain titles. A title that itself ends in a bracketed integer escapes the bracket: `/Appendix \[2]`.
- `\/`, `\\`, and `\[` are the only escapes. Any other backslash raises.
- A path with no leading `/`, or with an empty step such as `/Guide/` or `///a`, raises.

### How a path is matched

Each step is applied to the whole tree at once.

1. The first step starts from `blocks`. A later step starts from the `blocks` of every section the step before it kept.
2. `/title` keeps the sections with that title that are direct members of a starting list. `//title` keeps those at any depth under a starting list.
3. The kept sections are in file order, the order their headings appear in the text. A section counts once, even when two starting lists both reach it.
4. `[n]` then keeps only the `n`th of them, or none when there are fewer.
5. After the last step, one section is returned, none gives `nil`, and more than one raises.

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

A selector and its index come together. Giving one without the other raises.

### `markdown.parse(text)`

**Effect:** Parses Markdown text into blocks by the parsing rules above. **Return:** A list of blocks, empty for empty text. **Failure:** A non-string argument, text that is not UTF-8, or text longer than 8,388,608 bytes raises. That is the limit `json.decode` has, for the same reason: the parse runs outside the Lua memory limit. No UTF-8 text within the limit raises. **Example:** `local md = markdown.parse(store.read("notes.md"))`

### `markdown.section(blocks, path)`

**Effect:** Finds the one section that `path` names, starting from `blocks`. **Return:** The section table itself, not a copy, or nil when no section matches. **Failure:** A `blocks` that is not a list of blocks, a malformed path, a step index of zero, or a path that matches more than one section raises. **Example:** `local setup = markdown.section(md, "/Guide/Setup")`

```lua
markdown.section(md, "/Guide").title                    --> "Guide"
markdown.section(md, "/Guide/Templates").line           --> 16
markdown.section(md, "/Guide/Templates/Extras").level   --> 3
markdown.section(md, "//Extras").line                   --> 34
markdown.section(md, "//Templates/Extras").line         --> 34
markdown.section(md, "/Guide//Extras").line             --> 34
markdown.section(md, "/Templates")                      --> nil   (Templates is not at the top level)
markdown.section(md, "/Guide/Nope")                     --> nil
markdown.section(md, "/guide")                          --> nil   (titles are case-sensitive)
markdown.section(guide.blocks, "/Usage").line           --> 42    (a path starts at the list you pass)
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
markdown.section(md, "/Log/Entry[3]")                    --> nil
markdown.section(md, "/Log/Entry[2]", "prose", 1).text   --> "second"
```

### `markdown.section(blocks, path, selector, n)`

**Effect:** Finds the section as above, then picks the `n`th block matching `selector` among that section's own `blocks`. Blocks inside its subsections are not counted. **Return:** The block table itself, or nil when the section is missing or has fewer matches. **Failure:** Everything the two-argument form raises for, plus an unknown kind, a language on a kind other than `fence`, a selector with no index, an index with no selector, and an index that is zero or not an integer. **Example:** `local body = markdown.section(md, "/Guide/Templates", "fence markdown", 1).text`

```lua
markdown.section(md, "/Guide/Templates", "fence", 3).text           --> '{ "ok": true }'
markdown.section(md, "/Guide/Templates", "fence markdown", 1).text  --> "# Hello"
markdown.section(md, "/Guide/Templates", "prose", 2).text           --> "Then the fallback:"
markdown.section(md, "/Guide/Templates", "fence", -1).lang          --> "json"
markdown.section(md, "/Guide/Templates", "fence lua", 2)            --> nil   (the second lua fence is in Extras)
markdown.section(md, "/Guide/Templates", 'fence ""', 1)             --> nil   (every fence there has a language)
markdown.section(md, "/Guide/Templates", "section", 1).title        --> "Extras"
markdown.section(md, "//Extras", "fence", 1).text                   --> "return 2"
markdown.section(md, "/Guide", "list", 1).items[2]                  --> "beta"
markdown.section(md, "/Guide/Nope", "fence", 1)                     --> nil
```

To take every direct member of one kind, loop over the section's `blocks` with `ipairs` and test `kind`. The data is plain, so no function is needed for that.

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

`prompt` reads the running prompt. It keeps the Runvil style: a section is named by heading reference and resolved over the caller's visible set, the same lookup `call` and `jump` use. It never takes a path.

```lua
prompt.section(heading)                --> string, or nil            (exists today)
prompt.section(heading, selector, n)   --> block, or nil             (new)
prompt.list(heading)                   --> list of strings, or nil   (exists today)
```

### Heading references and the visible set

A heading reference is one or more `#` characters, whitespace, and the exact heading text: `"## Setup"`. The level is part of the address.

The visible set is the caller's siblings, except the caller itself, plus the caller's direct children. From the H1 it is every top-level section. Resolution never searches farther, so a section cannot name itself, its parent, a niece, a cousin, or a grandchild. A heading that is outside the visible set and a heading that exists nowhere give the same `nil`.

A heading reference needs no index. The prompt parser rejects a prompt in which two siblings share a title ("sibling section names must be unique"). A sibling and a child may share a title, but they sit at different levels and the level is part of the reference. So a reference names at most one visible section.

This is why `prompt` returns a section as text and not as a table. A section table carries its subsections in `blocks`, and the subsections of a visible sibling are nieces, which the caller may not see. A section's own raw Markdown stops at its first subsection heading, so it cannot leak them.

`prompt` has no read for the caller's own section, the H1's text, or the frontmatter.

### `prompt.section(heading)`

**Effect:** Reads a visible section's own raw Markdown: the text after its heading line up to the next heading of any level, with leading blank lines and trailing whitespace trimmed. It is never run or substituted. **Return:** A string, empty for an empty section, or nil when no visible section has that heading. **Failure:** A non-string target, a malformed reference, or an ambiguous reference raises. **Example:** `local rules = prompt.section("## Rules")`

### `prompt.section(heading, selector, n)`

**Effect:** Resolves `heading` as above, parses that section's own raw Markdown by the block rules, and picks the `n`th block matching `selector`. **Return:** A block table built for this call, or nil when no visible section has that heading or the section has fewer matches. Its `line` counts lines of the prompt file. **Failure:** Everything `prompt.section(heading)` raises for, plus a malformed selector, a selector with no index, an index with no selector, and an index that is zero or not an integer. The kinds `section` and `frontmatter` also raise, because a section's own content never holds either. A subsection is named by its own heading, and only when the caller can see it. **Example:** `local template = prompt.section("### Template", "fence markdown", 1).text`

The result is the block that the same pick over `markdown.parse(prompt.section(heading))` gives, with two differences. `line` counts lines of the prompt file instead of lines of the section. And frontmatter is never recognized, because a prompt's frontmatter is at the top of the file and never in a section.

A Lua fence of a visible section is a `fence` like any other: its `lang` is `lua`, and its `text` is the source, which is not run.

### `prompt.list(heading)`

**Effect:** Reads a visible list section and strips the item markers. **Return:** A list of strings, or nil when no visible section has that heading. **Failure:** A non-string target, a malformed or ambiguous reference, or a section that is not a list raises. **Example:** `local topics = prompt.list("### Topics")`

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
local style = prompt.section("### house_style")
local template = prompt.section("### Template", "fence markdown", 1).text
local topics = prompt.list("### Topics")
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
````

Inside `## Write`, whose direct children are the three `###` sections:

```lua
prompt.section("### house_style")                          --> "Short sentences. No hype."
prompt.section("### Template")                             --> "Fill in this outline.\n\n```markdown\n# Title\n\n## Summary\n```"
prompt.section("### Template", "fence markdown", 1).text   --> "# Title\n\n## Summary"
prompt.section("### Template", "fence markdown", 1).line   --> 25    (a line of the prompt file)
prompt.section("### Template", "fence", 1).lang            --> "markdown"
prompt.section("### Template", "prose", 1).text            --> "Fill in this outline."
prompt.section("### Template", "fence", 2)                 --> nil
prompt.section("### Template", 'fence ""', 1)              --> nil   (the only fence has a language)
prompt.section("### Topics", "list", 1).items              --> { "allocators", "coroutines" }
prompt.list("### Topics")                                  --> { "allocators", "coroutines" }
prompt.section("### Missing")                              --> nil
prompt.section("### Missing", "fence", 1)                  --> nil
prompt.section("## Write")                                 --> nil   (a section does not see itself)
```

The `## Summary` line inside the template fence is not a section of the prompt, because a fence is opaque.

### Using `markdown` on prompt text

`prompt.section(heading)` returns text, and `markdown.parse` takes text, so the two compose. Visibility is enforced in one place: `markdown` only ever sees what `prompt.section` handed over.

```lua
local blocks = markdown.parse(prompt.section("### Template"))
#blocks                                              --> 2
blocks[1].kind                                       --> "prose"
blocks[2].lang                                       --> "markdown"

-- the template is itself Markdown, so it parses too
local outline = markdown.parse(blocks[2].text)
markdown.section(outline, "/Title/Summary").level    --> 2
```

## One document, both namespaces

The two addressing styles give different answers for the same headings, on purpose.

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

| Caller | `prompt.list("### Items")` | Why |
|---|---|---|
| `## H2` | `{ "a", "b" }` | Its own child. The other `Items` is a niece. |
| `## H3` | `{ "c" }` | Its own child. |
| `# H1` | `nil` | The H1 sees only `## H2` and `## H3`. |
| `### Items` under `## H2` | `nil` | A section does not see itself, and it has no siblings or children. |

No caller can reach the other parent's `Items`, and no reference is ambiguous.

As text given to `markdown.parse`:

```lua
markdown.section(md, "/H1/H2/Items").line               --> 5
markdown.section(md, "/H1/H3/Items").line               --> 12
markdown.section(md, "//H3/Items").line                 --> 12
markdown.section(md, "/H1/H2/Items", "list", 1).items   --> { "a", "b" }
markdown.section(md, "/Items")                          --> nil      (no top-level section has that title)
markdown.section(md, "//Items")                         --> raises   (matches 2 sections, at lines 5 and 12)
markdown.section(md, "//Items[1]").line                 --> 5
markdown.section(md, "//Items[2]").line                 --> 12
```

Neither style can be mistaken for the other. `prompt.section("/H1/H2/Items")` raises because a heading reference needs `#` markers, and `markdown.section(md, "### Items")` raises because a path starts with `/`.

## Scope, identity, and copies

### Scope

A lookup sees only the list it is given. Passing a section's `blocks` narrows the search to that section, at any depth of nesting.

```lua
local guide = markdown.section(md, "/Guide")

markdown.section(guide.blocks, "/Usage").line    --> 42    (the path starts inside Guide)
markdown.section(guide.blocks, "/Guide")         --> nil   (a section is not inside its own blocks)
markdown.section(guide.blocks, "/Guide/Usage")   --> nil
#markdown.select(guide.blocks, "fence lua")      --> 2     (searched inside Guide only)
markdown.section(guide, "/Usage")                --> raises   (pass guide.blocks)
```

### Line numbers

`line` always counts lines of the text that was given to `markdown.parse`, never lines of a subset. It is a field stored on each block at parse time, and narrowing a lookup does not touch it. The number changes base only when a different string is parsed.

```lua
markdown.section(guide.blocks, "/Templates", "fence", 3).line   --> 30   (still a line of the whole text)
markdown.parse(templates.blocks[3].text)[1].line                --> 1    (a new parse counts from 1)
```

- `markdown.parse(block.text)` and `markdown.parse(item)` count from line 1 of that string.
- `markdown.parse(prompt.section(heading))` counts from the first line of that section's text.
- `prompt.section(heading, selector, n)` counts lines of the prompt file.

### Identity

`markdown.section` and `markdown.select` return the tables that are inside the list they were given.

```lua
markdown.section(md, "/Guide") == md[3]                  --> true   (the same table, not a copy)
markdown.select(md, "fence")[1] == templates.blocks[2]   --> true
```

So an edit made through a lookup's result is an edit of `md`, and a later `markdown.render(md)` writes it. `prompt.section(heading, selector, n)` is the opposite: it builds a new table on every call, and editing that table changes nothing else.

### Where bytes are copied

Lua strings are immutable and are passed by reference. Putting a block's `text` in another variable or table never copies it. Bytes are copied only when a string is created, which happens when text crosses from the Engine into Lua.

| Call | Bytes copied |
|---|---|
| `markdown.parse(text)` | Each block string once: every `text`, `title`, `lang`, `info`, item, and frontmatter value. A section holds no text of its own, so nesting adds nothing. The total is at most the size of `text`. |
| `markdown.section`, `markdown.select` | None. They return tables that already exist. `select` creates one new list to hold them. |
| `markdown.render(blocks)` | The one string it returns. |
| `prompt.section(heading)` | The section's Markdown, once per call. |
| `prompt.section(heading, selector, n)` | The chosen block's strings, once per call. Nothing else in the section becomes a Lua value. |
| `prompt.list(heading)` | Each item, once per call. |

Lua also keeps a single copy of every string of 40 bytes or fewer. `kind` values, languages, and most titles are therefore shared across all blocks and all calls.

The `prompt` reads are not memoized. The prompt's text lives outside the Lua heap, so each call creates its strings again. Keep the result in a local when it is needed more than once. To read several blocks of one section, parse it once with `markdown.parse(prompt.section(heading))` and index the result.

### Across sections

Every section entry gets a fresh VM, so a parse result does not survive a fall-through or a `jump`. Two things can carry it.

- **`var`.** `var.doc = md` stores a deep copy as JSON. Reading it back gives guarded views. Indexing and `ipairs` read through them, and so do the `markdown` functions. Lua's `#` and `pairs` do not, which is true of every table kept in `var`. Every later section entry rebuilds the whole copy, used or not.
- **A fanout collection.** `tasks.fanout("### Worker", markdown.select(md, "section"))` gives each arm one section as its `item`.

Carrying the text, or the name of a store file, and parsing it where it is needed is usually cheaper than carrying the tables.

## Recipes

```lua
-- A template kept in the prompt
local fence = prompt.section("### Template", "fence markdown", 1)
local template = fence and fence.text

-- The JSON a model replied with
local reply = models.infer(prose)
local found = markdown.select(markdown.parse(reply), "fence json")[1]
local data = found and json.decode(found.text)

-- One section of a stored document, as text for a model
local paper = markdown.parse(store.read("paper.md"))
local abstract = markdown.section(paper, "//Abstract")
local text = abstract and markdown.render(abstract.blocks)

-- Each top-level section of that document
for _, block in ipairs(paper) do
  if block.kind == "section" then
    log(block.title)
  end
end
```

## Errors

Messages are written for a model to read: what was wrong, and what was required. The wording below is proposed for the new failures.

| Call | Message |
|---|---|
| `markdown.parse(42)` | `markdown.parse expects a string, got number` |
| `markdown.parse(text)`, text too long | `markdown.parse: input exceeds the 8388608-byte limit` |
| `markdown.parse(text)`, text not UTF-8 | `markdown.parse: text is not valid UTF-8` |
| `markdown.section(guide, "/Usage")` | `markdown.section: blocks must be a list of blocks, got a section; pass its blocks field` |
| `markdown.section("text", "/Usage")` | `markdown.section: blocks must be a list of blocks, got string` |
| `markdown.section(md, "## Setup")` | `markdown.section: a path starts with "/", got "## Setup"` |
| `markdown.section(md, "/Guide/")` | `markdown.section: path "/Guide/" has an empty step` |
| `markdown.section(md, [[/a\b]])` | `markdown.section: path "/a\b" has an unknown escape "\b"; the escapes are "\/", "\\", and "\["` |
| `markdown.section(md, "//Items")` | `markdown.section: path "//Items" is ambiguous; it matches 2 sections, at lines 5 and 12; add an index such as "//Items[1]"` |
| `markdown.section(md, "/Log/Entry[0]")` | `markdown.section: path "/Log/Entry[0]" has index 0; an index is a nonzero integer` |
| `markdown.section(md, "/Guide", "fences", 1)` | `markdown.section: unknown kind "fences"; kinds are frontmatter, section, prose, fence, list, rule` |
| `markdown.section(md, "/Guide", "list lua", 1)` | `markdown.section: only a fence selector takes a language, got "list lua"` |
| `markdown.section(md, "/Guide", "fence")` | `markdown.section: a selector needs an index; 1 is the first match and -1 the last` |
| `markdown.section(md, "/Guide", "fence", 0)` | `markdown.section: index must be a nonzero integer, got 0` |
| `markdown.render({ { kind = "prose" } })` | `markdown.render: blocks[1].text must be a string, got nil` |
| `prompt.section("### Template", "section", 1)` | `prompt.section: a section's own content has no subsections; name a subsection by its heading` |

`prompt.section` keeps its existing messages for a non-string target, a reference without `#` markers, and an ambiguous reference. A failure of the Lua memory limit while a result is built passes through unchanged, as it does for `json.decode`.

## Decisions

- **`markdown.parse` is the `json.decode` of Markdown.** Text in, plain tables out, and the same input limit. Jason: "markdown.parse…nice, sounds like json.decode". Vinnie: "it does".
- **`markdown.parse` takes text, not a file name.** One string cannot mean both, and the VM has no file access. Vinnie: "Runvil creates the Lua VM in a restricted sandbox which disallows all file access", and on this point, "I agree taking text".
- **Plain data.** Tables with no methods can be encoded, stored in `var`, and fanned out. Vinnie: "I dont want to give up plain data." The cost is `md[3].blocks[1]` instead of `md[3][1]`, and no sugar such as `md["## Setup"]`.
- **A list is a block.** Lists are the data authors most often want out of Markdown, and a kind added later would be carved out of `prose` and shift existing indexes. Vinnie: "right, a list is a block".
- **`markdown` uses paths and `prompt` uses heading references.** Vinnie: "I want markdown to have the XPATH syntax ("/path/to/section") and `prompt` to have the Runvil style." Parsed text has no caller position, so the whole tree is addressable. The prompt has one, so the visible set applies.
- **`prompt` follows the visibility rule.** Vinnie: "A section can only see the names of its siblings and child sections", "the API for examining a prompt's markdown needs to follow this rule", and "The rules are the same for call and jump".
- **A path that matches more than one section raises.** An ambiguous address never resolves to an arbitrary first hit, which is what heading references already do.
- **A path step takes an index, `[n]`.** A prompt cannot repeat a sibling title, because the prompt parser rejects it, but parsed text can. Vinnie: "however markdown.parse has to work generally", and "Yes the [] looks needed". The index is part of the string because it can sit on any step, including one in the middle of a path. It counts in file order and accepts a negative number, like the selector's index.
- **The selector and its index are arguments, not path steps.** Every path step is then a title, so a section named `list` or `rule` cannot collide with a kind. The selector's index stays a number. The same `selector, n` pair also works after a heading reference in `prompt.section`. Jason, on that form: "It reads well, give me this prompt section, with all the markdown fences, then give me the one at index 1".
- **`fence ""` selects a fence with no language.** `fence` alone has to keep matching every fence, as a bare kind does for every kind, so the unnamed case needs its own spelling. Any word in the language position could be a real language, so the spelling is the empty string that `lang` holds.
- **Selectors count a section's own blocks only.** This matches what `prompt.section(heading)` already treats as a section's content, and it is what keeps a selector from reaching a niece.
- **`//` is included.** A model reply may or may not wrap its sections in an H1, and a stored document's H1 title is usually not known in advance. `//Findings` covers both.
- **A lookup is scoped to the list it is given, and `line` never changes.** Scope comes from the argument, so there is no second way to narrow a search. `line` stays tied to the parsed text, so a block reports the same line however it was reached.
- **Lookups return the tables that are there.** Vinnie: "i am assuming that the strings in Lua will be shared (i.e.. not duplicated needlessly, when calling section())". They are: `markdown.section` and `markdown.select` create no strings and copy no tables.
- **The `prompt` reads are not memoized.** A section entry's VM is short-lived and a local variable already holds a result. A cache would keep every string it ever returned alive until the VM ends.
- **Line endings are normalized and nothing else is.** A `\r\n` file and a `\n` file give the same strings, which keeps comparisons and line counts predictable. All other text is kept as written.
- **Parsing does not fail on content.** The prompt parser rejects skipped levels and repeated sibling names because a prompt is a program. `markdown` reads data.

## Rejected alternatives

- **Heading references in `markdown`, alone or in a list.** Jason's gist: `markdown.section(md, "### Extras")` and `markdown.section(md, { "# Guide", "## Templates" })`. Jason: "it'd be nice if the slashed path weren't required". Vinnie: "the syntax in your gist creates ambiguities". The level must be known in advance, so a reply that writes `# Abstract` where `## Abstract` was expected gives nil. How deep each step searches and where an index goes are undefined. And the same string would reach the whole text in `markdown` but only the visible set in `prompt`.
- **A `prompt.section` that picks its return by what the section holds.** Jason: "if a section only has one code block it's smart enough to know how to handle it". The return type would depend on the section's content, so adding a sentence above a fence would silently change what the call returns. The selector states the same wish explicitly.
- **`call` returning a markdown block as a string.** Jason: "you could call a section with a markdown codeblock, and it just returns the string". Vinnie: "call expects to run code". The read is `prompt.section(heading, "fence markdown", 1).text`.
- **Paths in `prompt`.** A path from the root would let `## H2` read the child of `## H3`, which the visibility rule forbids.
- **A section that is both an array and a record.** `md[3][1]` beside `md[3].title` reads well, but `json.encode` refuses a table that mixes list items and named keys.
- **Accepting a section where a list of blocks is expected.** `markdown.section(guide, "/Usage")` could mean "inside Guide" or "Guide itself". Raising keeps one meaning per call.
- **Selectors inside the path, such as `/Templates/fence[3]`.** A title and a kind would share one namespace, so a section titled `fence` could not be told from the kind. A loop over a section's fences would also need string building.
- **XPath's per-parent index.** In XPath, `//Items[2]` means every `Items` that is the second one under its own parent. Counting in file order is what an author expects from "the second `Items`".
- **First match wins.** Which section comes back would depend on file order, and nothing would say there were two.
- **A `text` field on every section.** It would make a section's source one field away, but each nesting level would hold another copy of the same bytes. `markdown.render(section.blocks)` gives the text when it is wanted.
- **Reading a file by name**, such as `markdown.parse("file.md")` or `load_text("other.md")`. The sandbox has no file access. Text comes from `store.read` or from `prompt.section`.

## Open questions

1. **Addresses in `prompt` that survive re-nesting.** Vinnie: "I'm not entirely thrilled with "### Template" in general, because if you ever want to cut and paste code to a different nesting level, now all the labels are invalid", "but I was reluctant to invent a syntax that distinguishes siblings from children", and "there is a benefit to having just one path syntax". The `#` count is what tells a sibling from a child today. Vinnie: "in Runvil, sibling and child section names must be unique". The parser checks siblings only, so a sibling `## Notes` beside the caller's own child `### Notes` parses, and only the level separates them. Dropping the level needs something else to do that. One candidate is to make the quoted rule true in the parser: a section may not share a title with a sibling of its parent, which rejects prompts that parse today. The other is a relative path: a bare name is a child and `../name` is a sibling, which gives one path syntax for both namespaces but changes every heading reference in the language, `call` and `jump` included. The `markdown` half of this document does not depend on the answer, and the `selector, n` tail of `prompt.section` carries over either way.
2. **Keep or drop `prompt.list`.** `prompt.section(heading, "list", 1).items` covers it, and Jason suggested dropping it. The two are not identical: `prompt.list` needs every nonblank line of the section to be a list item and raises for a section that is not a list, while the selector accepts a list beside prose and returns nil when there is none.
3. **Other XPath forms.** `*` for any one section, and `..` for a parent, are left out. `..` would need parent references, which plain data does not have.
4. **Whether `markdown.render` ships in the first version.** Nothing else here depends on it.

## Implementation notes

### What exists today

- `prompt.section` and `prompt.list` are installed by `install_prompt_table` in `crates/runvil-internal/lua/src/vm/install.rs`, over one read callback. The callback is `read_visible` in `crates/runvil-internal/runtime/src/execute/walk_target.rs`, which returns a `SectionText` holding the section's Markdown and its list items.
- The visible-set lookup is `find_visible` in `crates/runvil-internal/runtime/src/heading_address.rs`, shared by `call`, `jump`, `prompt.section`, `prompt.list`, `tasks.spawn`, and the `tasks.fanout` worker template. The selector form needs no new visibility code.
- Sibling titles are checked by `check_unique_sibling` in `crates/runvil-internal/prompt/src/sections.rs`. It is the only uniqueness check.
- The list-or-record rule is `shape` in `crates/runvil-internal/lua/src/json/keys.rs`. Reading through guarded views is `Encoder::resolve` in `json/encode.rs`, where the 127-level depth limit also lives. The 8,388,608-byte input limit is `INPUT_LIMIT` in `json/decode.rs`.
- Engine global names are listed in `RESERVED_NAMES` in `crates/runvil-internal/lua/src/globals.rs`. `markdown` becomes one more entry.
- The VM is Lua 5.5 through `mlua` 0.12. `pulldown-cmark` 0.12 and `serde_yaml_ng` 0.10 are workspace dependencies that `runvil-prompt` uses. `runvil-lua` depends on neither.

### Where the code goes

- `runvil-prompt` depends on `runvil-lua`, so the block parser cannot live in `runvil-prompt`. It belongs in `runvil-lua`, or in a new crate below it. A new crate fits the repository's preference for more, smaller crates: it would hold the text-to-blocks parser, the path and selector grammars, and the writer behind `render`, with no `mlua` dependency. `runvil-lua` would hold the conversion to Lua tables, the walks over Lua tables, the `markdown` global (installed the way `json::install` installs `json`), and the selector form of `prompt.section`.
- `SectionText` needs the prompt-file line that the section's Markdown starts on, so that the selector form can report `line`. `runvil-prompt` can record it where it already computes block lines.
- `read_visible` copies the section's Markdown and every list item on each read, whichever of the two the caller wants. Splitting the read, or carrying the shared prompt text with a range instead of an owned string, leaves each read copying only the bytes it returns.
- When this lands, the language spec's "Deferred `markdown` namespace" paragraph and its `prompt.section` entry take their text from this document.

### How the parser works

This is the approach the prototype took, in about 250 lines for the parser.

1. Drop a byte order mark and turn `\r\n` and `\r` into `\n`.
2. Split frontmatter by the rule `split_frontmatter` in `runvil-prompt` uses, and keep it only when the YAML between the delimiters deserializes to a JSON object.
3. Make one pass over `Parser::new_ext(body, Options::empty()).into_offset_iter()`, counting depth. At depth 0, record each heading, fenced code block, list, and rule with its source range. Collect a heading's title from its text and code events, a fence's `text` from its text events, and a list's items from the item ranges.
4. Treat the text between two recorded ranges as prose. This is what keeps link definitions, which produce no events, and it is what makes rule 3 hold by construction.
5. Fold the flat sequence into sections with a stack: a heading closes every open section whose level is the same or larger.
6. Build each item string from the source. It starts at the item's first child event, and each later line loses that many columns of leading whitespace.

A path is matched one step at a time, each step as a single walk of the tree in file order, so the kept sections come out ordered and counted once.

### Tests to carry over

- Every example in this document. The prototype ran the parse, path, selector, and render examples, with further cases, as 130 checks.
- Rule 3: every non-blank line of the input lies in exactly one block.
- `markdown.parse(markdown.render(md))` equals `md` apart from `line`, for parse output. The prototype checked this on 55 inputs, and only the escaped-title case differed.
- The block parser and the prompt parser agree on where top-level headings are, for the same text.
- A result read back from `var` works with every `markdown` function.
