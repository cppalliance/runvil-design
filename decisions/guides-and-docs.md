# Guides and docs: design decisions and their reasons

- Scope: the language guide, the Gateway guide, the docs site and its build, crate docs, the wording rules, and where each kind of document is stored
- Code: `runvil` at commit `fa9936f64`, `runvil-docs` at `cfd69a4`, and `runvil-gateway` at `9369a7d`
- Update: the text on the source of the API references, the two copies of the wording test, and the paths of Engine and Harness files describe `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`, after the Engine and the Harness moved (section 8)
- Paths: code paths start at the root of `runvil`, and a path in another repository starts with that repository's name

## Summary

The guides are Markdown chapters in the `runvil-docs` repository, and people edit them by hand. Code in the `runvil` repository builds the docs site. The `build-user-guide` program copies each guide's chapters into a source tree for mdBook, the tool that turns Markdown chapters into a website. The build code calls a guide a book. `cargo xtask site` runs mdBook and rustdoc, writes the landing page, and checks the links. The `site.yml` workflow publishes the result. Inside `runvil`, doc comments and short READMEs are the documentation. Sections 1 to 4 describe these parts.

A check of this page against the code found five gaps and left three questions for the owner. Section 6 lists the gaps, and section 7 lists the questions. The largest gap is the Gateway guide: `runvil-docs` has its 11 chapters, and the site build skips them.

The largest planned item in section 5 is the Workshop guide, which is one chapter titled "Coming Soon".

## From a chapter file to a published page

A chapter is one Markdown file, such as `runvil-docs/src/language/12-tools.md`. The number at the start of its name sets its place in the book, and its first H1 heading is its title.

The `site.yml` workflow starts on each push to `main` or `master` of `runvil` and at 07:17 UTC each day. Its `build` job checks out `runvil` and `cppalliance/runvil-docs`, sets `RUNVIL_DOCS` to the docs folder, and runs `cargo xtask site`.

The site command runs `build-user-guide` in its `stage` mode as a child process. For each book in the list `BOOKS`, `stage` copies `guide/books/<book>/book.toml`, `guide/chrome/back-link.js`, and the book's chapters into `target/site-books/<book>/`, and writes `SUMMARY.md` and `index.md` from the chapter titles.

The site command then runs mdBook on each staged book, builds the rustdoc sites in `RUSTDOC_SITES`, writes the landing page, and checks the links. The chapter is now `target/site/language/12-tools.html`. The `build` job uploads `target/site`, and the `deploy` job publishes it to GitHub Pages.

## 1. Where each kind of document is stored

### Inside `runvil`, doc comments and short READMEs are the documentation

Rustdoc builds the API reference from the doc comments on declarations. Ten crates also have a `README.md`, one paragraph that states the crate's responsibility, and the root `README.md` is the project's front page.

The documentation has this form because long-form docs inside the repository drift from the code after every session of a coding agent. The agents then treat the stale text as truth.

### The guide chapters are files in `runvil-docs`, and `runvil` builds the site

`runvil-docs` contains the chapters under `src/<book>/`. `runvil` contains each book's `book.toml` under `guide/books/`, two files under `guide/chrome/` that add an "All docs" link to every page, and the code and workflow that build the site. The build reads the chapters from the folder that `RUNVIL_DOCS` names and fails when the variable is unset.

Guide prose drifts from the code, and the separate repository puts that prose outside what a coding agent reads while the agent edits the code.

### A crate's doc comment states its structural rules under `## Invariants`

The crate doc comment in `lib.rs` or `main.rs` has a section named `## Invariants` that lists rules such as the crates it may depend on. A structural check requires the section in every `workshop-*`, `harness-*`, and `plugin-*` crate. A crate's `AGENTS.md`, a rule file that coding sessions read, states the product rules that checks leave unenforced ([Crates, build, and CI](crates-build-and-ci.md)).

## 2. The guides

### Each audience has one guide

`runvil-docs` has three sets of chapters: `src/gateway/` for a Gateway operator, `src/language/` for a prompt author, and `src/workshop/` for a Workshop user. A Rust developer reads the API references, because rustdoc covers the Rust API. The owner's rule: "There should not be a user guide per crate. There should be a user guide per product."

The Workshop set is one placeholder chapter. Workshop changes fast before its beta, and prose about such a product is out of date sooner than anyone can update it.

### The language guide has sixteen teaching chapters and a quick reference

The files are `01-what-a-prompt-is.md` through `17-quick-reference.md`. Each teaching chapter owns one group of topics, and a feature that fits two chapters belongs to the chapter where an author would look for it first. The quick reference lists every name an author can write, and its `Taught in` column links to the section that teaches the name.

### The language guide teaches what a prompt author can use

The reader is an author of Runvil `.md` prompt files, and the guide omits the Rust API, the internals of the Harness and the Gateway, and every planned form. A chapter teaches an item only when the code or a test shows that an author can use it. The prose is in the present tense and leaves out history, migration, deprecation, and roadmap.

A prompt example is a complete prompt. `runvil-docs/CONTRIBUTING.md` adds three house rules: a single dash, four backticks on every fence, and one line for each paragraph.

### People edit the chapters by hand, and the site workflow publishes them as committed

A chapter changes when a person or a coding session edits its file. The site workflow publishes it as committed: `runvil-docs/CONTRIBUTING.md` says "There is no freshness gate."

### A coding session that finds stale guide text fixes it before its commit

When a coding session finds a doc comment, a code comment, or a guide passage that is wrong, it fixes the text before its next commit, and it commits a fix in `runvil-docs` first. The owner's rule: "fix the docs whenver you find a problem. dont leave it for later. stale docs confuses models". Every `AGENTS.md` omits the rule.

## 3. The docs site

### `build-user-guide` stages one mdBook tree for each book in `BOOKS`

`BOOKS` is the one list of books, and it names two: `workshop` and `language`. For each one, `stage` copies the chapters from the `runvil-docs` checkout, sorts them by file name, and reads each title from the file's first H1, which every chapter needs. It writes `SUMMARY.md` and `index.md` into the staged tree, so authors leave both files out of `runvil-docs`. It also writes each book as one file, `runvil-<book>-guide.md`. Section 6 covers the Gateway set, which `BOOKS` leaves out.

### `cargo xtask site` builds the whole site under `target/` and fails on a broken link

The command writes the site to `target/site/`, and the flag `--books-only` skips rustdoc. One command builds the whole site the same way on a developer's machine, Windows included, and in CI. The build writes its files under `target/` and leaves the checked-in tree unchanged.

Each link ends in a file name, as in `language/index.html`, because a browser that opens the built site from disk, under `file://`, shows a directory listing for a link to a folder. After the build, `check_links` reads every `.html` page outside the rustdoc folders, except each `404.html`, and fails on each link that names a folder or a missing file.

### The site has five API references and two books, each in its own folder

`RUSTDOC_SITES` lists five crates: `runvil-engine`, `runvil`, `harness-gateway-client`, `plugin-web`, and `plugin-mcp`. The five are crates of `runvil-harness`, and the site command documents each at the commit that the root `Cargo.toml` of `runvil` pins ([Crates, build, and CI](crates-build-and-ci.md)). A change to their doc comments therefore reaches the site when `runvil` moves that pin. Rustdoc documents each with its default features, so that each published reference shows its crate as the crate's dependents see it. Each API reference and each book has its own folder under `target/site/`, with its own navigation and search.

The folders are separate so that each product has documentation of its own. Together they are one site, because GitHub Pages publishes one site for each repository.

### The site command generates the landing page as one table

The function `page` in `site-landing.rs` builds `index.html`: a title and a table with one row for each entry in `RUSTDOC_SITES` and each staged book. A book's row shows the `title` and `description` from its `book.toml`, and a `book.toml` that lacks either field fails the build.

The command generates the page from those two lists because a hand-written page can miss an entry. The owner's rule: "a simple table. No images".

### The site workflow publishes on each push to `runvil` and once a night

`site.yml` deploys the full site on a push to `main` or `master`, on a manual run, and once a night. On a pull request that changes the site's build code, it builds the books with `--books-only` and skips the upload. Only the `deploy` job has the Pages permissions.

The nightly run exists for the chapters: they are in `runvil-docs`, and the workflow starts only on events in `runvil`. A chapter edit therefore appears on the site within a day. A rebuild on every push to `runvil-docs` would need a cross-repository token.

## 4. Crate docs and wording

### A doc comment contains prose, and examples are tests or example programs

The structural check in `no_doctests.rs`, which the `build-xtask` crate of each workspace has, fails any doc comment with a code block that rustdoc would compile, and any doc attribute that reads its text from a file with `include_str!`. Runnable examples are ordinary tests and two programs in `runvil-harness`, `greeter.rs` in `runvil-engine` and `run-prompt.rs` in `runvil`.

The project's checks leave out doctest steps, which would add about 67 seconds to every local run. The test runs would therefore skip a doctest that an agent writes, and the structural check exists to fail it.

### `AGENTS.md` defines seven terms, and a Node test checks the rulebooks for four of them

The root `AGENTS.md` of each repository defines Engine, Harness, Host, Pack, Plugin, Static Set, and Dynamic Set. Each repository has a copy of the test, `crates/workshop/ui/test/docs-claims.mjs` in `runvil` and `runvil-harness/tools/docs-claims.mjs`. The test reads every rulebook of its repository: each `AGENTS.md`, each `.cursor/rules` file, and each crate doc with an `## Invariants` section. It fails on a lowercase "host", "engine", "harness", or "plugin" outside a short list of allowed phrases. The copy in `runvil-harness` also fails on text in an Engine crate that names the Host.

The test exists so that every rulebook uses the four terms one way, and only the root `AGENTS.md` defines them. The pre-push hook of each repository runs the test.

## 5. Planned work

Each of these items is still to be built.

- **The Workshop guide.** A guide for a person who uses Workshop. The `workshop` book has one placeholder chapter, titled "Coming Soon".
- **A check of guide examples.** A scheduled job in `runvil-docs` that parses every prompt example with the Engine's parser. The site workflow publishes each chapter as committed, with its examples unparsed.
- **A check of names in prose.** A check that fails when a comment or a README names an item that the code lacks. READMEs name code items: `runvil-harness/crates/harness-internal/runner/README.md` names `Host` and `send`.

## 6. Known gaps

Each item names a place where a guide, a README, a contributing file, or a doc comment differs from the code, or a file in `runvil-docs` that the site build skips. The pages of the other subjects, such as [The prompt language](prompt-language.md), list further differences in what a guide teaches.

1. The site omits the Gateway guide. `runvil-docs/src/gateway/` has 11 chapters. `BOOKS` in `crates/build-user-guide/src/main.rs` lists the other two books, so `stage` skips the Gateway chapters. The root `README.md` of `runvil` says the guide is in the Gateway's repository, and `runvil-gateway/README.md` says it is in "the main Runvil documentation". `runvil-gateway` lacks both the chapters and a site workflow.
2. The language guide documents `sys.execution`. Chapter 5 and the quick reference list the field, and the Engine test `sys_holds_no_execution_label` asserts that reading it raises an error.
3. `runvil-docs/CONTRIBUTING.md` says each of three sets "builds as its own book", and `BOOKS` lists two. It omits that `cargo xtask site` belongs to `runvil` and needs `RUNVIL_DOCS`.
4. The crate docs of `build-user-guide` say "The generator owns the chapters". People edit the chapters by hand, and `build-user-guide` copies them unchanged.
5. The site build skips `runvil-docs/src/introduction.md`. The file says Workshop records runs in "an append-only, hash-chained event store", and the crates lack a hash chain.

## 7. Questions for the owner

Each question asks for the reason behind a part of the design.

**The guides**

1. Why does the site build leave out the Gateway guide, and which repository is meant to publish it?
2. Why does the language guide omit every planned form?

**The site**

3. Why mdBook?

## 8. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64`, `runvil-docs` at commit `cfd69a4`, and `runvil-gateway` at commit `9369a7d`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.
- **Update.** On 2026-11-12 the text that the move of the Engine and the Harness changed was brought to `runvil` at commit `48a4dbc08` and `runvil-harness` at `5bca6f9`. Commit `adca9db10` of `runvil` removed the Engine, Harness, and Plugin crates, which are in `runvil-harness` with their history, and the code that builds the site stays in `runvil`. The update covers the entry on the five API references, the two entries of section 4, and the path in the third planned item. The rest of the page still describes the commits above.

*2026-11-10 16:37 - Claude Fable 5.2*
