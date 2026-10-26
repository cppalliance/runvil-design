# Quick access (QA) - VS Code 1.128.0 findings

Section: Quick access (QA). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). Cursor findings are in the sibling file under `cursor\`.
- Level: minor groups only. No feature lists, no pieces.
- Short names: V = `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0`. N = `C:\Users\Vinnie\src\vscode` (1.144.0, used only to spot newer things). All proof paths below start under `V\src\vs\` unless they say otherwise.
- Line numbers are from the files as read. They can drift by a line or two, so search the quoted id or label if one does not match.

## What this section is

The floating list that drops from the top of the window. You type, it filters, you press Enter. One widget (the quick input) hosts many providers. Each provider claims a text prefix. The default provider has no prefix and is the "Go to File" list.

Prefixes found in V (all registered through `registerQuickAccessProvider`):

- (none) - Go to File, in `workbench\contrib\search\browser\anythingQuickAccess.ts` line 99 (`PREFIX = ''`), registered in `searchQuickAccess.contribution.ts` lines 18-28
- `>` - Command Palette, `platform\quickinput\browser\commandsQuickAccess.ts` line 51
- `?` - Help list, `platform\quickinput\browser\helpQuickAccess.ts` line 20
- `:` - Go to Line/Column, `editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts` line 25; `::` is the offset form, line 26
- `@` - Go to Symbol in Editor; `@:` groups by category, `editor\contrib\quickAccess\browser\gotoSymbolQuickAccess.ts` lines 50-52
- `#` - Go to Symbol in Workspace, `workbench\contrib\search\browser\symbolsQuickAccess.ts` line 37
- `%` - Quick Search (text), `workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts` line 39
- `view ` - Open View, `workbench\contrib\quickaccess\browser\viewQuickAccess.ts` line 32
- `edt active `, `edt `, `edt mru ` - editor switchers, `workbench\browser\parts\editor\editorQuickAccess.ts` lines 198, 223, 254; registered in `workbench\browser\parts\editor\editor.contribution.ts` lines 146-168
- `term ` - terminals, `workbench\contrib\terminalContrib\quickAccess\browser\terminalQuickAccess.ts` line 25
- `task ` - run task, `workbench\contrib\tasks\browser\tasksQuickAccess.ts` line 25; registered `task.contribution.ts` lines 404-410
- `debug ` and `debug consoles ` - `workbench\contrib\debug\browser\debugCommands.ts` lines 125-126; registered `debug.contribution.ts` lines 88-107
- `ext ` and `ext install ` - `workbench\contrib\extensions\browser\extensionsQuickAccess.ts` lines 18 and 91 (extensions are out of scope, see left out)
- `issue ` - `workbench\contrib\issue\browser\issueQuickAccess.ts` line 22
- `mcpr ` - `workbench\contrib\mcp\browser\mcpResourceQuickAccess.ts` line 586 (MCP, out of scope)
- `agent ` - `workbench\contrib\chat\browser\agentSessions\agentSessionsQuickAccess.ts` line 21 (agent, out of scope)

Registry and routing code: `platform\quickinput\common\quickAccess.ts`, `platform\quickinput\browser\quickAccess.ts`, `platform\quickinput\browser\pickerQuickAccess.ts`.

## Proposed minor groups

Count: 6. Reason: this section has one shared widget plus a set of providers. The providers split cleanly by what the person is trying to do (run a command, find a file, jump inside a file, switch to something already open), plus one group for how you open the box and read the help, and one for the widget. A seventh group for "other pickers" would only hold extension, MCP, issue and agent lists, which are out of scope. Fewer than 6 would merge things people think of separately (the Command Palette and Go to File are different tools to a user). Confidence: high - the split follows how the registrations and keybindings already divide.

### QA-1 Command Palette

- Plain-words name: Command Palette
- One sentence: The `>` list where you type a command name and run it, with recent history and a shortcut to set its keybinding.
- Tag: both (Cursor bundle carries the provider and `workbench.action.showCommands`)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Show All Commands (F1, Ctrl+Shift+P; id `workbench.action.showCommands`)
  - "recently used" and "other commands" sections, plus the "Remove from Recently Used" row button
  - Configure Keybinding gear button on every command row
  - Clear Command History (`workbench.action.clearCommandHistory`, asks for confirmation)
- Also seen in this group, not counted as examples: "similar commands" section (natural language, needs an extension that provides it), "commonly used" section (experimental), an "Ask in Chat" row at the bottom (experimental, on by default in V).
- Where it is reachable from: View menu "Command Palette...", Help menu "Show All Commands", Manage gear menu "Command Palette...", editor right-click menu "Command Palette...".
- Proof paths:
  - `workbench\contrib\quickaccess\browser\commandsQuickAccess.ts` lines 273-295 (action and keybinding), 297-328 (clear history), 130-140 (gear button), 170-195 (Ask in Chat row)
  - `platform\quickinput\browser\commandsQuickAccess.ts` lines 51 (prefix), 224-242 (section separators), 286-293 (remove from recent button), 373 (history class)
  - `workbench\contrib\quickaccess\browser\quickAccess.contribution.ts` lines 42-48 (provider), 55-108 (menu items)
  - Settings: `workbench\browser\workbench.contribution.ts` lines 496-535

### QA-2 Go to File and workspace search

- Plain-words name: Go to File and workspace search
- One sentence: The default Ctrl+P list that finds files by name, plus the `#` workspace symbol list and the `%` Quick Search text list.
- Tag: both (Cursor modified the file list, see the Cursor file)
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Go to File with a "recently opened" section and "file results" (Ctrl+P, secondary Ctrl+E on Windows and Linux; id `workbench.action.quickOpen`)
  - Row buttons: "Open to the Side" (or "Open to the Bottom") and "Remove from Recently Opened"
  - Go to Symbol in Workspace (`#`, Ctrl+T; id `workbench.action.showAllSymbols`)
  - Quick Search for text (`%`; id `workbench.action.quickTextSearch`, no default key, has "Open in Search View" and "See More Files")
- Also seen: typing `:` or `@` after a file name jumps to a line or symbol in that file; Quick Open Previous Editor from history (`workbench.action.quickOpenPreviousEditor`, opens with the second item active).
- Proof paths:
  - `workbench\contrib\search\browser\anythingQuickAccess.ts` lines 99 (prefix), 209-219 (config read), 402-440 (separators), 1037-1058 (row buttons), 1111-1115 (open pinned or to the side)
  - `workbench\contrib\search\browser\searchQuickAccess.contribution.ts` lines 18-36 (registration)
  - `workbench\contrib\search\browser\symbolsQuickAccess.ts` line 37; `searchActionsSymbol.ts` lines 17-32 (Ctrl+T)
  - `workbench\contrib\search\browser\quickTextSearch\textSearchQuickAccess.ts` lines 39, 113, 253, 274, 355; `searchActionsTextQuickAccess.ts` lines 23-26; command id in `workbench\contrib\search\common\constants.ts` line 33
  - `workbench\browser\actions\quickAccessActions.ts` lines 125-152 (Go to File action), 197-200 (previous editor)
  - Settings: `workbench\contrib\search\browser\search.contribution.ts` lines 172-196; `workbench\browser\workbench.contribution.ts` lines 319-321, 340

### QA-3 Go to Line and Go to Symbol in this file

- Plain-words name: Go to a spot in this file
- One sentence: Small lists that jump the cursor to a line, a column or a symbol in the open editor, with a live highlight while you move through the list.
- Tag: both for Line and Symbol. Go to Offset is vscode only (not found in the Cursor bundle).
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Go to Line/Column (`:`, Ctrl+G; id `workbench.action.gotoLine`; Go menu item "Go to Line/Column...")
  - Go to Symbol in Editor (`@`, Ctrl+Shift+O; id `workbench.action.gotoSymbol`)
  - Go to Symbol in Editor by Category (`@:`, groups symbols by kind such as methods and functions)
  - Go to Offset (`::`, id `workbench.action.gotoOffset`, plus a "Toggle Zero-Based Offset" button) - vscode only
- Also seen: "Open to the Side" button on symbol rows; outline-based picks when the active pane is not a plain text editor (it asks the outline service, line 177 of the workbench file below); status text "Line N, column M" for screen readers.
- Proof paths:
  - `workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts` lines 73-126 (actions, keys, placeholders), 85 (zero-based button)
  - `editor\contrib\quickAccess\browser\gotoLineQuickAccess.ts` lines 25-26 (prefixes); `gotoSymbolQuickAccess.ts` lines 50-52, 247, 406, 459-470
  - `workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts` lines 267, 278-325 (action, key, help entries)
  - `editor\contrib\quickAccess\browser\editorNavigationQuickAccess.ts` lines 155, 202-216 (status text, range highlight)
  - `workbench\contrib\quickaccess\browser\quickAccess.contribution.ts` lines 82-90 (Go menu item)

### QA-4 Switch and open lists

- Plain-words name: Switch and open lists
- One sentence: Prefix lists that jump to something that is already open or already defined - a view, an editor, a terminal, a task, a launch configuration.
- Tag: both
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Open View (`view `; menu View > Open View...; Quick Open View is Ctrl+Q on Windows and Mac (the Control key on Mac) and has no default key on Linux; lists Side Bar, Panel, Secondary Side Bar, Terminal, Debug Console and Output channels)
  - Editor switchers (`edt active `, `edt `, `edt mru `; each row has a "Close Editor" button; Ctrl+Tab opens the in-group most-recently-used one)
  - Switch Active Terminal (`term `; rows for panel and editor terminals, "Create New Terminal", rename and kill buttons)
  - Run Task (`task `) and Start Debugging (`debug `, with "Configure Launch Configuration" and "Remove Launch Configuration" row buttons; `debug consoles ` for debug consoles)
- Overlap to watch: the Ctrl+Tab editor switcher and "Show All Editors" commands are declared by the Editor part, so the Editor section may list them as commands. The list itself is drawn here.
- Proof paths:
  - `workbench\contrib\quickaccess\browser\viewQuickAccess.ts` lines 32 (prefix), 152-208 (container groups), 231-262 (actions and Ctrl+Q)
  - `workbench\contrib\quickaccess\browser\quickAccess.contribution.ts` lines 121-145 (view picker cycle keys)
  - `workbench\browser\parts\editor\editorQuickAccess.ts` lines 198, 223, 254, 169; `editor.contribution.ts` lines 146-168; `workbench\browser\parts\editor\editorActions.ts` lines 1712-1764 (show editors commands), 1804-1835 (Ctrl+Tab)
  - `workbench\contrib\terminalContrib\quickAccess\browser\terminalQuickAccess.ts` lines 25, 41-80, 113-117; `terminal.quickAccess.contribution.ts` lines 18-37
  - `workbench\contrib\tasks\browser\tasksQuickAccess.ts` lines 25, 39, 67-85
  - `workbench\contrib\debug\browser\debugQuickAccess.ts` lines 28-158; `debugConsoleQuickAccess.ts` lines 38-41

### QA-5 Opening quick access and the help list

- Plain-words name: Opening quick access and the help list
- One sentence: The ways in to the box (keys, menus, the title-bar command center) and the `?` list that names every prefix, plus the keys that cycle the list while you hold a shortcut.
- Tag: both
- Evidence: source
- Boundary: no (the command center box itself is drawn by the title bar, so the Title Bar section overlaps)
- Examples (up to 4):
  - Help list (`?`; "Show all Quick Access Providers"; typing a prefix inside it switches to that provider)
  - Quick Open from the command center (`workbench.action.quickOpenWithModes`; with an empty box it adds shortcut rows in this order: Go to File, Command Palette, Search for Text, Open Quick Chat when quick chat is on, Go to Symbol in Editor, Start Debugging, Run Task and "More")
  - Navigate Next and Previous in Quick Open (Ctrl+P again, or Ctrl+Shift+P, cycles the list while the shortcut is held; `workbench.action.quickOpenNavigateNext` and `...Previous`)
  - Close and accept keys that work in every provider (Escape, `workbench.action.closeQuickOpen`; Mac Ctrl+N and Ctrl+P select next and previous)
- Proof paths:
  - `platform\quickinput\browser\helpQuickAccess.ts` lines 18-75; `workbench\contrib\quickaccess\browser\quickAccess.contribution.ts` lines 21-31
  - `workbench\browser\actions\quickAccessActions.ts` lines 154-195 (command center action), 205-283 (navigate and select actions)
  - `workbench\contrib\search\browser\anythingQuickAccess.ts` lines 824-870 (command center help picks; the order numbers 10, 20, 25, 30, 40, 50, 60, 70 come from the `commandCenterOrder` fields in the provider registrations)
  - `platform\quickinput\common\quickAccess.ts` line 159; `platform\quickinput\browser\quickAccess.ts` (prefix routing)
  - Context keys: `workbench\browser\quickaccess.ts` lines 20-25 (`inQuickOpen`, `inFilesPicker`)

### QA-6 Quick pick and input box widget

- Plain-words name: Quick pick and input box widget
- One sentence: The shared floating panel (list, input box, title bar, buttons, checkboxes, step counter) that every picker and prompt in the app reuses.
- Tag: both
- Evidence: source
- Boundary: no
- Examples (up to 4):
  - Single and multi-select quick pick (check-all box, "{0} Selected" count badge, OK button, optional Custom button, "Toggle all checkboxes")
  - Input box mode (placeholder, prompt text, password dots, validation message with an info, warning or error icon)
  - Title bar with Back button, title text, step counter ("step N of M"), and title buttons
  - Position and drag: top (default), center, or dragged; saved between runs; commands "Align Quick Input Top" and "Align Quick Input Center", also shown as "Quick Input Position" in Customize Layout
- Also seen: list rows with icon, label, description, detail, keybinding label and match highlights; row buttons that show on hover; section separators that can carry buttons; progress bar when busy; hover on a row (Ctrl+Space); pin buttons used by the task picker and the terminal "run recent command" picker (`quickPickPin.ts`, used by `workbench\contrib\tasks\browser\taskQuickPick.ts` and `workbench\contrib\terminalContrib\history\browser\terminalRunRecentQuickPick.ts`); two more modes, Quick Widget (custom HTML) and Quick Tree (checkbox tree). In V the tree and widget modes are only used by chat code, so they are boundary-adjacent and left out of the examples.
- Keys inside the widget: Up, Down, PageUp, PageDown, Ctrl+Home, Ctrl+End, Enter, Escape, Right arrow (quick pick only: accept without closing, when focus is in the list or the cursor is at the end of the box), Space (toggle the checkbox when focus is not in the input box), Ctrl+Space (toggle hover), Alt+Left on Windows (Back), Ctrl+- on Mac, Ctrl+Alt+- on Linux, Ctrl+Up and Ctrl+Down (separators, with a quick access fallback).
- Proof paths:
  - `platform\quickinput\common\quickInput.ts` lines 20-110 (item shape), 267-420 (base input), 509-740 (quick pick), 741-800 (input box), 800-870 (buttons and button locations), 948-1065 (service), 1073-1280 (tree)
  - `platform\quickinput\browser\quickInputController.ts` lines 154-230 (title bar, check-all, count badges, OK and Custom buttons, progress), 732 (back tooltip), 889-960 (layout), 1032-1050 (saved position, storage key `workbench.quickInput.viewState`), 1057-1170 (drag and snap)
  - `platform\quickinput\browser\quickInputActions.ts` lines 103-205 (movement keys), 212-300 (accept, hide, toggle)
  - `platform\quickinput\browser\quickInputList.ts`, `quickInputBox.ts`, `quickPickPin.ts`, `media\quickInput.css`
  - `workbench\browser\actions\layoutActions.ts` lines 1280-1310 (align actions), 1385-1388 (Customize Layout items)
  - `workbench\browser\actions\quickAccessActions.ts` lines 31-124 (close, accept, focus, toggle, back)
  - `workbench\services\quickinput\browser\quickInputService.ts` line 42 (close on focus lost)

## Settings the widget and providers expose

All paths under `V\src\vs\`. Defaults are from the declarations.

- `workbench.commandPalette.history` - number, default 50, min 0. Recently used commands kept; 0 turns history off. `workbench\browser\workbench.contribution.ts` line 496.
- `workbench.commandPalette.preserveInput` - boolean, default false. Restore the last typed text on next open. Same file line 502.
- `workbench.commandPalette.experimental.suggestCommands` - boolean, default false, tag experimental. Shows a "commonly used" list (uses `commandPaletteSuggestedCommandIds` from the product). Line 507.
- `workbench.commandPalette.experimental.askChatLocation` - `chatView` or `quickChat`, default `chatView`, experimental. Line 513.
- `workbench.commandPalette.showAskInChat` - boolean, default true, experimental. The "Ask in Chat" row. Line 524.
- `workbench.commandPalette.experimental.enableNaturalLanguageSearch` - boolean, default true, experimental. The "similar commands" section; needs an extension. Line 530.
- `workbench.quickOpen.closeOnFocusLost` - boolean, default true. Applies to all quick inputs, read in `workbench\services\quickinput\browser\quickInputService.ts` line 42. Declared line 536.
- `workbench.quickOpen.preserveInput` - boolean, default false. Restore last typed text in Quick Open. Line 541.
- `search.quickOpen.includeSymbols` - boolean, default false. Add workspace symbols to Go to File results. `workbench\contrib\search\browser\search.contribution.ts` line 172.
- `search.quickOpen.includeHistory` - boolean, default true. Add recently opened files to results. Line 182.
- `search.quickOpen.history.filterSortOrder` - `default` (by relevance) or `recency`, default `default`. Line 188.
- `workbench.editor.enablePreviewFromQuickOpen` - editors from Quick Open open as preview; hold Ctrl to pin. `workbench\browser\workbench.contribution.ts` line 319.
- `workbench.editor.openSideBySideDirection` - `right` or `down`; sets the wording of the "Open to the Side" or "Open to the Bottom" row button. Line 340.
- Related but owned by the Title Bar section: `window.commandCenter` (line 865). It shows the launcher that opens `workbench.action.quickOpenWithModes`.
- Related file search settings that change Go to File results (owned by Search): `search.useIgnoreFiles` (line 154) and `search.followSymlinks` (line 198) in `search.contribution.ts`.
- Not a setting but stored state: `workbench.quickInput.viewState` (saved drag position, application scope), `platform\quickinput\browser\quickInputController.ts` line 45.
- Interface-only name: `workbench.quickOpen.enableExperimentalNewVersion` appears in `workbench\browser\quickaccess.ts` line 40 but is not declared as a setting anywhere in V. Treat as dead.
- Colors the widget reads (theme section, listed so nobody hunts for them later): `quickInput.background`, `quickInput.foreground`, `quickInputTitle.background`, `pickerGroup.foreground`, `pickerGroup.border`, `quickInputList.focusBackground`, `quickInputList.focusForeground`, `quickInputList.focusIconForeground`, `quickInputList.focusHighlightForeground`, in `platform\theme\common\colors\quickpickColors.ts`.

## Default keybindings seen (V)

- Go to File: Ctrl+P, plus Ctrl+E on Windows and Linux. `workbench\browser\actions\quickAccessActions.ts` lines 24-28.
- Command Palette: Ctrl+Shift+P, plus F1. `workbench\contrib\quickaccess\browser\commandsQuickAccess.ts` lines 281-287. (Not bound on Firefox web.)
- Go to Line: Ctrl+G (Mac: Control+G). Go to Symbol in Editor: Ctrl+Shift+O. Go to Symbol in Workspace: Ctrl+T.
- Quick Open View: Ctrl+Q (not on Linux). Editor switcher in group: Ctrl+Tab (Mac: Control+Tab).
- Back inside a multi-step picker: Alt+Left (Windows), Control+- (Mac), Ctrl+Alt+- (Linux).

## Menu entries that open this section

- View menu: "Command Palette..." and "Open View..." (`quickAccess.contribution.ts` lines 55-62 and 73-80)
- Help menu: "Show All Commands" (lines 64-71)
- Go menu: "Go to Line/Column..." (lines 82-90); "Go to Symbol in Editor..." and "Go to Symbol in Workspace..." have mnemonic titles declared on their actions
- Manage (gear) menu: "Command Palette..." (lines 91-99)
- Editor right-click menu: "Command Palette..." (lines 100-108), hidden in simple inputs
- Title bar command center: "Quick Open" (`workbench.action.quickOpenWithModes`, menu `CommandCenterCenter`), `quickAccessActions.ts` lines 154-163

## Newer-than-1.128 (seen only in N, 1.144.0)

Tag: newer-than-1.128. Evidence: N source. None of this is in V, so none of it is cataloged as a feature.

- Quick Open accepts a link: typing `scheme://...` shows a link row and an "Invalid Link" row with the text "Enter a valid link to open it from Quick Open." `N\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts`.
- New run options for the Go to File provider: `filesOnly`, `fileSearchDelegate`, and a `providerPrefix` option on the quick access controller. Same file, and `N\src\vs\platform\quickinput\common\quickAccess.ts`.
- Quick input can be anchored below an element (`anchorPosition: 'below'`), can host a custom widget above the list, and has `withQuickInputAnchor`. `N\src\vs\platform\quickinput\common\quickInput.ts`, `quickInputController.ts`.
- Quick items can carry a theme color for the icon (`iconColor`). `N\src\vs\platform\quickinput\common\quickInput.ts`.
- Open and close motion classes (`quick-input-widget-closing`, `quick-input-widget-overlay`) and a new `modern-ui` style hook. `N\src\vs\platform\quickinput\browser\media\quickInput.css`.
- Go to Symbol also works on a focused chat's outline. `N\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts` (chat, so out of scope).
- Setting line numbers moved (`workbench.commandPalette.*` at lines 515-549 in N) but the same six command palette settings and two quick open settings exist, so no new settings in this section.

## Left out and why

- Extensions list and install pickers (`ext `, `ext install `): the extension system is out of scope. `workbench\contrib\extensions\browser\extensionsQuickAccess.ts`.
- MCP resource picker (`mcpr `): MCP is out of scope. `workbench\contrib\mcp\browser\mcpResourceQuickAccess.ts`.
- Agent sessions picker (`agent `) and the experimental unified quick access (`workbench.action.unifiedQuickAccess`, gated by `chat.unifiedAgentsBar.enabled`): agent and chat surfaces. `workbench\contrib\chat\browser\agentSessions\agentSessionsQuickAccess.ts`, `experiments\unifiedQuickAccess.ts`.
- "Ask in Chat" row in the Command Palette: it is listed inside QA-1 only as a seen item, because its output goes to chat. The chat it opens is out of scope.
- Chat mode picker, model picker, agent picker, tool picker (`chatToolPicker.ts`): out of scope by the brief.
- Quick Tree and Quick Widget consumers (`chatUrlFetchingConfirmation.ts`, `languageModelToolsConfirmationService.ts`, `createPluginAction.ts`, `chatQuick.ts`): chat code. The widget modes are noted in QA-6 only.
- Open Recent, Switch Window, and other quick picks that other areas build with `createQuickPick` (for example `workbench\browser\actions\windowActions.ts`): they belong to the area that owns the command. The widget they use is QA-6.
- Issue reporter picker (`issue `): reachable only through Help > Report Issue, which the Dialogs or Title Bar sections can claim. `workbench\contrib\issue\browser\issueQuickAccess.ts`.
- Standalone editor quick access (`editor\standalone\browser\quickAccess`): web-component editor only, not the workbench. The brief says it is not needed.
- Built-in extensions (git, markdown, simple-browser, references-view and others) only touch this section through `contributes.menus.commandPalette` entries that hide or show their commands in the palette. They add no quick access providers.

## Could not verify

- I did not run V, so every statement about on-screen behavior (hover timing, snap distances, how the highlight looks) comes from reading code, not from watching it.
- The `commandPaletteSuggestedCommandIds` list lives in the product file, which is not in V `src`. I did not open `product.json`, so I do not know which commands "commonly used" would show.
- Whether the "similar commands" section ever appears in stock V: it needs an extension that provides related-information, and I did not check which built-in extension, if any, does that.
- Exact Go to File ranking rules (scorer thresholds) were not traced beyond reading the separators and button wiring.
