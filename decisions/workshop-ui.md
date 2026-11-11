# Workshop: what the user sees: design decisions and their reasons

- Scope: what a Workshop user sees and does: the window, its zones and panels, the menus and commands, the agent window, the chat box, and the look
- Code: `runvil` at commit `fa9936f64`, and `runvil-docs` at commit `cfd69a4` for the Workshop guide
- Paths: code paths start at the `crates/workshop/` folder of the `runvil` repository

## Summary

Workshop's UI is one page of plain TypeScript in one desktop window. A title bar shows eight menus, a dock arranges panels in three zones, and a status bar is fixed below the dock. The left zone shows the file tree, the main zone shows editors and Run windows, and the right zone shows agent windows, each a transcript above a chat box. The UI copies the Cursor IDE in look, layout, wording, and behavior. Sections 1 to 7 describe these parts.

A check of this page against the code found eight gaps and left seven questions about reasons for the owner. Section 9 lists the gaps, and section 10 lists the questions. The largest gap is the server's `chat_ready` flag: the UI stores it and only tests read it, so the chat box can offer send and the microphone while the server reports chat as unavailable.

Section 8 lists the planned work. Its largest item is running a prompt from the Run window, whose Run button has an empty click handler.

## From opening Workshop to reading a reply

The desktop app opens one window and loads the UI's page from the Workshop server. `main.ts` preloads the saved UI state, and `setupWindowChrome` shows the title bar. `main.ts` then creates the dock with `createDockview`, and `applyLayoutOrDefault` restores the workspace's saved layout. For a new workspace, the layout policy opens the `tree` panel on the left and an `agent` panel on the right, with an empty editor group between them. Last, `setupWindowMenus` fills the title bar with eight menu buttons.

`AgentPanel` creates an `AgentSocket`, an `AgentSessionService`, and an `AgentSessionView`. The socket connects, the server lists its agents, and the panel launches the agent named `chat`. An agent is a prompt file that the server runs. The view shows a `TranscriptView` above a `ChatBox`.

The chat box's round button is the microphone while the box is empty and the send arrow once the user has typed text. Once the run waits for its first message, Enter emits a `send` event with the exact text. `AgentSessionView` checks that a model is selected, calls `AgentSessionService.respond`, and clears the box.

The server sends the reply as a stream of frames. The service updates its list of transcript items from each frame, `buildTranscript` builds rows from the items, and `TranscriptView` updates each row in place. While the agent works, the round button is Stop and the status bar's activity LED is lit. `renderMarkdown` renders the reply, with DOMPurify as its last step.

## 1. The reference product and the rules of the code

### The UI copies the Cursor IDE in look, layout, wording, and behavior

`ui/AGENTS.md` states the rule and names the files to read: `workbench.desktop.main.css` and `workbench.desktop.main.js` of the installed IDE. The agent window copies the agent pane beside Cursor's file tree and excludes Cursor's separate Glass window, because the Glass window's values differ from the pane's. The owner's goal: "At 100% the UI should be identical to Cursor, the spacing, the font sizes, the font, the layout".

The UI has one theme, a dark one. The repository is public, so its code states facts about Cursor and excludes Cursor's source code and Cursor's own icon glyphs. The icons in `look/icons.ts` are codicon strings, from VS Code's icon set.

### The UI is plain TypeScript on the DOM, with a library each for the dock, the editors, and the chat input

`ui/package.json` lists `dockview` for the dock, `@codemirror` for file editors, `@tiptap` for the chat input, and `marked`, `shiki`, and `dompurify` for replies. The UI leaves out React, which would enlarge the bundle and require a rewrite of the code written against dockview. The chat input calls Tiptap's framework-agnostic API. [Workshop: how the app is built](workshop-app.md) covers the UI's code layers, its registries, and the server's storage of every saved value.

## 2. The window, its zones, and its panels

### Workshop has one operating-system window, and each window the UI names is a dock panel

An agent window, an editor tab, and a Run window are each one panel in the dock. The features register five panel types: `tree` for the file tree, `editor` for one file, `agent` for one agent window, `run` for one Run window, and `config` for the settings of the Gateway, the separate process that answers model requests ([The Gateway](gateway.md)). The dock's serialized layout lists every panel, and the UI saves it as one value of the workspace file, so one key restores the panels, the tabs, the zone sizes, and the place of each panel.

### The UI draws its own title bar in the desktop app and in a browser

`setupWindowChrome` shows the bar in both modes, because the bar contains the menus. Its center is the window's drag area. In a browser `setupWindowChrome` hides the window buttons. The context key `isWeb`, a named value that menu rows test, is true there, and each command with the precondition `!isWeb` shows disabled.

On Windows the desktop app removes the system frame. On macOS it overlays the native window buttons on the bar, and on Linux it leaves the native frame above the bar. The bar shows on macOS too, because one bar inside the window makes the menus identical in the desktop app and in a browser, and a test can check them in both.

### The dock has three zones, and each panel type has a home zone

`zones.ts` names three zones, each a tabbed dockview group: `left` for the file tree, `main` for editors and Run windows, and `right` for agent windows. `openInZone` places a new panel where the user last dropped that panel, and otherwise in its type's `defaultZone`. The owner's rule: "each zone is a bank of tabbed windows, and you can freely move windows from one place to another, but each zone has a preference".

The `tree` type registers with `closable: false`, so its tab omits the close button. `applyLayoutOrDefault` reopens `tree` or `agent` when a restored layout lacks one, so the UI starts with both panels even from a saved layout that is out of date.

### A zone that loses its last panel remains as an empty group

Dockview deletes a group when its last panel closes, and the other zones then resize. `rebuildZone` in `zones.ts` re-creates the group with `dock.addGroup`, at the zone's remembered size.

Dockview creates, draws, serializes, and restores an empty group itself, so an empty zone needs only that group.

### Every gesture that closes a tab calls the panel's `confirmClose` first

A tab's X, a middle click, Delete on a focused tab, and the Close command all call `closeActiveEditor` in `panel-close.ts`. That function awaits the panel's `confirmClose()`, a method of the base class `WorkshopPart` that resolves true. `EditorPanel` overrides it to show its save dialog, and Cancel stops the close. A close that skipped the call would discard unsaved edits silently. The tab handles Delete and Backspace itself, because dockview's own handler closes an unsaved editor and skips the dialog. Open Workspace, Close Window, and Exit omit the call (section 9).

### The status bar is a fixed footer outside the dock

`StatusBar` appends its view to the page body, below the dock. The bar shows the server's status text, a busy animation named the barberpole, and indicators that features register: dictation's microphone meter ([Speech](speech.md)), a red recording LED, and the activity LED. The bar reports a local error as a toast.

The activity LED is amber while a model turn is processing and green during a burst of output tokens.

## 3. Menus, commands, and keys

### The title bar shows the eight menus of Cursor

`menubar.contribution.ts` declares File, Edit, Selection, View, Go, Run, Terminal, and Help. File opens files, folders, workspaces, and agent windows. Edit and Selection act on the focused text. View toggles the side bars and the status bar, zooms, and opens the command palette. Go opens quick open, Run opens a Run window, and Help opens About. File > Preferences > Settings opens the Gateway's configuration panel.

The menus are Cursor's eight because the goal is a window that behaves like VS Code, and almost every row of Cursor's menus has a VS Code command id.

### Every menu action is a command with the id that VS Code uses

A feature registers each action once, with `registerAction`, in its `*.contribution.ts` file. The menu row, the shortcut, and the command palette all run that one command. Its id is VS Code's id for the same action, such as `workbench.action.files.save`, and Workshop's own rows use the same prefix. The ids follow VS Code's so that an agent tool, which is still to be built, can run commands by id and inherit their preconditions.

### A row that Workshop has yet to implement shows disabled under its final command id

`stubs.contribution.ts` is a table of 136 rows. Each row registers a command under its VS Code id with the precondition `false` and an empty `run` function, so the menu shows it disabled with its final name and shortcut. To implement a row, a developer deletes it from the table and calls `registerAction` in the owning feature. Every Terminal row is a stub, and so are the Run menu's debug rows.

The stubs exist so that the menus show their final shape and tell the truth about which rows work. A stub has its final id, so implementing the row leaves every menu, test, and keybinding unchanged.

### Shortcuts copy Cursor's, and the dispatcher stops every key combination that has a rule

A keybinding rule is a string of one or two key combinations, such as `ctrlcmd+m ctrlcmd+o`, where `ctrlcmd` means Cmd on macOS and Ctrl elsewhere. One `KeybindingDispatcher` listens on the document in the capture phase, because a bubbling listener would run after CodeMirror's keymap on the editor's content element, and a combination that both define would fire twice. For a combination that any rule names, it calls `preventDefault` and `stopPropagation`, even where the rule's `when` condition fails. The registry therefore omits Tab and Shift+Tab, since a rule for either would stop the key in every other control. The agent window's shortcuts register one weight tier above the stubs that share them, and the higher tier wins.

### A tab's right-click menu is built from the menu registry

A right-click on a closable tab opens the registry's `MenuId.EditorTitleContext` menu at the pointer. For that menu, `panel-tab.ts` sets the context key `activeEditor` to the clicked tab's panel type, and the command receives `{ panelId }` and acts on that tab. An explicit panel id is simpler than the two alternatives, Theia's event adapter and JupyterLab's hit test on the DOM. Close and Close Others work, and most other rows are stubs.

### Quick open picks its mode from the typed prefix and lists the files the UI already has

The title bar's folder-name button and Ctrl+P open the quick open box. An empty prefix lists files, `>` lists commands, `:` asks for a line number, and `?` lists the modes. The prefixes `@`, `%`, `debug `, and `task ` each show one row that says the mode is unavailable. The file list is the folder listings that the tree has fetched, plus recent files. The owner's rule: "Only use what is there."

## 4. The agent window

### Each agent window is one panel, one socket, and one agent session

`AgentPanel` creates an `AgentSocket`, an `AgentSessionService`, and an `AgentSessionView`. File > New Agents Window opens another `agent` panel, and the New Chat commands reuse an empty one first. When the server sends its list of agents, the panel launches the one named `chat`, and the server starts one run of that prompt as the window's session. A panel restored from a saved layout starts a fresh session.

The owner's rule: "The Agent window should come up with the chat control already there".

### The transcript has five kinds of row

`buildTranscript` builds turns of rows from the session's items. A user message becomes a `human` row and starts a turn, and a reply becomes a `markdown` row. The function collects reasoning and tool calls into a group until a reply, a user message, or an error closes it. A group of one tool call becomes a `tool` row, a group of only reasoning becomes a `thought` row, and any other mix becomes a collapsible `group` row. `TranscriptView` updates rows by key, so a row that the user opened remains open. A thought row is closed until the user opens it.

### A tool call shows as one line, worded by its vendor-qualified name

`tool-labels.ts` joins a call's `vendor` ([Tools and Packs](tools-and-packs.md)) and tool id, and `toolKind` compares the result with three fixed names. A call of `runvil/web/search` or `runvil/web/fetch` shows a line that opens a hover card of results, and a call of any other tool shows a generic line that expands to its arguments and result. The third name, `runvil/user-input/ask`, matches a tool that the shipped Packs lack.

The match includes the vendor because another vendor's Pack can use the same tool id, and a match on the last segment of the id would also catch a tool named `ask` that a prompt defines in Lua.

### The UI treats text from a model, a tool, or the user as untrusted

`renderMarkdown` renders a reply or a thought: it parses with `marked`, highlights code with Shiki, and passes the result through DOMPurify as its last step. The row components insert user text, tool arguments, and tool output through `textContent`. A URL in a hover card becomes a link when its scheme is `http:` or `https:`.

DOMPurify runs inside `renderMarkdown` because `marked` leaves its output unsanitized by design, and a sanitizer at that one place covers every caller.

### An agent error opens a popup above the chat box, and Try again resends the last message

`AgentSessionView` shows each error through the chat box's `error` prop, and the transcript omits errors, as Cursor's transcript does. Try again sends the last user message again through `AgentSessionService.respond`, which appends a second user message to the agent's history. Workshop resends because the chat agent's loop returns to its `tasks.receive()` call after a failed round, and that call waits for the user's next message. The button is enabled while the run accepts messages, a state the service exposes as `started`.

### Stop is the round button or Ctrl+Shift+Backspace, and Esc is unbound

The agent feature binds the command `composer.cancelComposerStep` to `ctrlcmd+shift+backspace` while an agent window is active. The command and the chat box's Stop button call `cancelTurn`, which sends the server a cancel frame. The server calls `stop_round`, the Harness drops the model round and the tool calls in flight, and the run continues ([The Harness and the Host API](harness-and-host.md)). Esc is unbound, as in Cursor.

### The toolbar's model picker works, and its mode chip and token ring lack a backend

In `AgentToolbar`, `ModelPickerTrigger` lists the catalog from `ModelService` and sends the server a select command. `ModeChip` offers five modes, fires an `agent-mode-changed` event, and changes the round button's color. `TokenRing` reads a stub that returns `null`, which hides the ring. The toolbar copies the controls that Cursor shows, and the UI connects a control only where a backend exists.

### The chat box accepts a message while the agent works

The chat box is editable at all times. `renderInputState` in `AgentSessionView` sets the box's `action` prop: `idle` until the run first waits or works and again after the run ends, `send` while the agent waits, and `stop` while it works. The server adds a message sent during work to the run's inbox, and the UI drops a reply that the message cut off. Send is idle before the run starts, so that the UI sends a message only once the run has its Harness.

## 5. The chat box

### The chat box is one isolated component with props, events, and a handle

`ChatBox`, in `ui/src/parts/chatbox/`, owns the text editor, the mention pills, the `@` typeahead, and the round button. The part that embeds it, `AgentSessionView`, sets state with props, receives each `ChatBoxEvent`, and calls the methods of `ChatBoxHandle`. `ui/test/chatbox-boundary.mjs` fails on an import from the agent, speech, or services code, and on the word `grant`. The owner's rule: "make the chat box totally self-contained isolated source code component".

### The chat box declares each planned feature in its types

Slash commands, paste, attachments, and editing an older turn are still to be built. `types.ts` declares `commandSource` and `onPasteFiles`, and the box leaves both unused. Each of these features has an optional prop, an event variant, or a handle method in the box's types.

`serialize()` returns a `SerializedDraft` with the version field `v: 1`, and `restore()` leaves the box unchanged for any other version, because a saved draft that lacks a version becomes unreadable at the first change of its schema. Tests are the two methods' only callers.

### One round button is the microphone, the send arrow, or Stop

`actionState` in `chat-box.ts` picks the button's role: the microphone while the box is empty, the send arrow once the box has text, and Stop while the agent works and the box is empty. When the model selection is empty, the session view sets `action` to `send-blocked`: the button looks disabled and still emits `send`, so the view can name the blocker on the status bar. The whole chat box, this button included, copies Cursor's older input box and ignores Cursor's React prompt input. Dictation writes into the same box ([Speech](speech.md)).

### A chip has a kind and an opaque payload

A `ChipRef` describes a mention pill, a typeahead row, or an attachment. It has an `id`, a `label`, an optional `kind` such as file or folder, and a `data` field of any JSON value. The box styles a pill by `kind`, returns `data` unchanged in events and drafts, and leaves its content unread. The box only passes the payload to the part that embeds it, and that part interprets it.

### The `send` event has the exact text and the chips

The box emits one `send` event with `text`, `mentions`, and `attachments`. The text is exactly what the user typed, untrimmed. The box shows its content until the embedding part calls `clear()`, so the text survives a failed send. The payload is text and chips, because the editor's raw document JSON would couple the message format on the socket to the editor's schema. `AgentSessionView` sends the text and drops the two chip lists.

### The `@` typeahead copies Cursor's flat menu and lists a three-item stub

Typing `@` at the start of a word opens one flat list, and the `mentionSource` prop supplies its items. `AgentSessionView` omits that prop, so the default `stubMentionSource` answers with three canned file names: `README.md`, `src/main.ts`, and `Cargo.toml`. The stub has items because a test of the popup needs rows to render.

## 6. The editor and the Run window

### The editor is CodeMirror 6 behind the `EditorSurface` interface

`editor-surface.ts` defines `EditorSurface` and implements it with CodeMirror 6. Panels, zones, and the save flow call the interface, and the four files that import CodeMirror are all in `ui/src/parts/editor/`. The editor uses the defaults of Cursor's editor.

CodeMirror is a ready-made editor, and the interface isolates it from the rest of the UI. The owner's rule: "I want each piece of code to be very well isolated... I don't want AI code bloat."

### The Run window shows what a prompt declares, and its Run button has an empty click handler

Run > New Run Window runs `workbench.action.newRunWindow`, which opens a `run` panel in the main zone and loads the focused editor's file. The panel posts the prompt's text to the server route `POST /prompts/contract` and draws one row for each front-matter key except `uses`. The owner's rule: "let's keep the UI simple, one row for each thing."

The Run button exists so that the window has its final shape. Its click handler is empty because running a prompt from this window is still to be built (section 8).

## 7. The look

### The `look` package defines the look of the whole Workshop family

`@workshop/look`, in `look/`, exports the color palette, the size and semantic tokens, the icons, and five visual components: modal, dropdown, toast stack, status bar view, and progress bar. The owner's reason: "we have a product family of executable processes which share a look and feel". The Gateway's config UI has its own copy, `shared-ui`, and `ui/test/no-shared-ui.mjs` fails on any mention of that name.

### Component CSS takes each value from a token, and each class starts with `ws-`

The repository's root `AGENTS.md` requires Workshop's component CSS to use custom properties for every color, size, and spacing value. The tokens are in three tiers in `look/`: the palette in `tokens.css`, the size scale in `sizes.css`, and intent-named aliases in `semantic.css`. Each feature's CSS file is beside its TypeScript file.

The class prefix separates Workshop's CSS from other CSS, a job that Shadow DOM could also do. Shadow DOM duplicates CSS across shadow roots, and the prefix provides most of its benefit at a small part of its cost.

## 8. Planned work

Each of these items is still to be built.

- **Running a prompt from the Run window.** A click on the Run button that starts a run of the prompt, with storage for runs and remembered inputs. The Run menu's debug rows need a debug session that the server owns. The button's click handler is empty, and the debug rows are stubs.
- **A real `@` source.** A source for the `@` typeahead that lists open and recent files and, once a workspace file index exists, searches the workspace. The typeahead lists the three file names of `stubMentionSource`.
- **Slash commands.** A command chip at the start of the text, which the box reports through its `command` event. `types.ts` declares the event and `commandSource`, and both are unused.
- **Attachments and paste.** Pasted files and images, shown as chips in a strip above the text. `types.ts` declares `onPasteFiles`, which the box leaves unused, and `AgentSessionView` drops the `attachments` list of each `send` event.
- **Editing an older turn.** A way to edit an earlier message. A send from the edited turn opens a new agent window, because cutting the first window's history at that turn would discard a whole branch of it. The transcript shows each user message as a `human` row.
- **Saved history.** Transcripts, drafts, and sessions that resume after a relaunch. A panel restored from a saved layout starts a fresh session.
- **A message queue.** Messages that the user can edit or unsend until the agent takes them. The server adds each sent message to the run's inbox.
- **Modes and context usage.** Modes that change the agent's behavior, and usage data for the token ring. `ModeChip` fires an event and changes the round button's color, and `TokenRing` reads a stub that returns `null`.
- **Choosing Packs.** A screen that lists the available Packs with checkboxes, and a view of each MCP server's state. The Run window draws a row for each front-matter key except `uses`.
- **Rows for more tools.** Terminal cards, file-edit groups, and approval rows. A call of any tool other than `runvil/web/search` and `runvil/web/fetch` shows a generic line.

## 9. Known gaps

Each item names a value that only tests read, a path that closes editors and skips `confirmClose`, a comment or a rule file that differs from the code, or a stub.

**The code**

1. `chat_ready`. The server computes the flag `chat_ready` and sends it in every snapshot of its state. The flag is true when the catalog has a chat-capable model, a model is selected, the Gateway answers, and every switch of the Gateway's model profile has finished. `main.ts` passes each snapshot to a `WorkbenchService`, whose sole readers are tests. `AgentSessionView` blocks a send when the model selection is empty, and it passes dictation a blocker that returns `null`. A user can therefore send while a profile switch is in progress or the Gateway is unreachable. A recording still requires dictation's own socket, which the server relays to the Gateway, to be ready.
2. Open Workspace. `openWorkspaceFromFile` replaces the dock's panels through `applyLayoutOrDefault` and omits `confirmClose`.
3. Close Window and Exit. File > Close Window calls `closeWindow` in `window-chrome.ts`, and File > Exit invokes `quit` in `desktop/src/quit.rs`. Both end the page and skip `confirmClose`.

**Comments, rule files, and the guide**

4. Theming. The header of `look/semantic.css` says that theming the app is an edit of that file. The rule file `workshop-spa.mdc` says that colors are themed in `look/tokens.css`.
5. CodeMirror imports. The header of `editor-surface.ts` says that it is the only file that imports `@codemirror`. Three more files in its folder import it.
6. Two more comments. `main.ts` says the title bar is hidden in a browser, where `setupWindowChrome` shows it. The header of `workbench-service.ts` names a Model menu and a chat-gating hook as its readers, and the UI lacks both.
7. Ctrl+Q. A doc comment in `desktop/src/menu.rs` says that Ctrl+Q opens View on Windows and Linux. Open View is a row of the stub table, so the dispatcher stops the key and the window ignores it. File > Exit quits.
8. The Workshop guide. The Workshop guide in `runvil-docs` is one stub file ([Guides and docs](guides-and-docs.md)).

## 10. Questions for the owner

Each question asks for the reason behind a part of the design.

**The reference and the look**

1. Why does the UI have one theme, the dark one?
2. Why are the palette, size, and semantic tokens in `look`, while the UI's component tokens are in `ui/src/tokens/component.css`?
3. Does the rule in the root `AGENTS.md` against raising schema version numbers cover `LAYOUT_SCHEMA_VERSION` and the draft field `v`?

**The agent window**

4. Why is Esc unbound from Stop, beyond the match with Cursor?
5. Why does `toolKind` have the `ask` kind, when the shipped Packs lack the tool that it matches?

**The chat box and the Run window**

6. Why does the boundary test ban the word `grant` from the chat box's directory?
7. Why does the Run window omit a row for `uses`, when it shows read-only rows for `name` and `description`?

## 11. Method and limits

- **Code.** The page describes `runvil` at commit `fa9936f64` and `runvil-docs` at commit `cfd69a4`.
- **Checks.** The facts on this page were read in the code at those commits. A second session checked the gaps, the values, and the walk-through against the code.
- **Limits.** The sessions read the code and its tests, and left the build and the tests unrun.

*2026-11-10 16:50 - Claude Fable 5.2*
