# Quick access (QA) - Cursor 3.27.12 findings

Section: Quick access (QA). Date: 2026-10-25. Pass: 1.

- Scope of this file: what the Cursor install does for the section, and how it differs from stock VS Code 1.128.0. The group list itself lives in the sibling file under `vscode\`. Group ids here match that file.
- Short names: C = `C:\Users\Vinnie\AppData\Local\Programs\cursor\resources\app`. Main bundle = `C\out\vs\workbench\workbench.desktop.main.js`. Message table = `C\out\nls.messages.json` (the bundle calls `k(<number>,null)`, and the number is the index into that array, which is how I read the labels). V = stock 1.128.0 source.
- Evidence word used below: `bundle` means I saw it only as strings or minified code in the Cursor bundle. `source` means I read it in V.
- Cursor code is minified, so class names such as `aqe` or `kAe` are the bundle's own short names and can change between builds. The command ids, setting names and label text are stable proof.

## How I checked

- Searched the main bundle for every `registerQuickAccessProvider` call and mapped each to its placeholder text through the message table. There are 19 providers.
- Searched for each command id and setting name found in V for this section, and for the new ones the bundle showed.
- Read the Cursor copy of the quick input controller (`kAe`) and the Go to File provider (`vI`) around their constructors and the parts that differ.
- Checked the automations bundle (`workbench.anysphere-ui-automations.js`) for quick access providers and quick picks. None.
- Checked the four built-in extensions the brief names (`cursor-explorer`, `cursor-commits`, `cursor-checkout`, `cursor-worktree-textmate`) for `showQuickPick` and `createQuickPick`. None.
- Did not read `workbench.glass.main.js` (Agents window, out of scope).

## Summary of how Cursor differs

- The six V groups all exist in Cursor.
- Go to File (QA-2) is a modified copy with its own ranking, an "other results" split and a "Create File" row.
- The quick input widget (QA-6) is Cursor's own copy with a wider default width, a width setting, and a preview box that sits beside a row.
- A few V pieces are not in Cursor: Go to Offset, the "Ask in Chat" row, the Quick Tree mode, and the issue, MCP and agent-session pickers.
- Cursor adds two agent pickers (chat search and recent agents). Those are out of scope and listed under left out.

## Groups, with Cursor evidence

### QA-1 Command Palette

- Tag: both
- Evidence: bundle (V source read as well)
- Boundary: no
- Found in bundle: provider registered with prefix `>`, context key `inCommandsPicker`, placeholder "Type the name of a command to run." (message 8903), help text "Show and Run Commands" (8904). Command id `workbench.action.showCommands` appears 5 times. `workbench.action.clearCommandHistory` appears once.
- Same row furniture as V: sections "recently used", "similar commands", "commonly used", "other commands"; the gear button tooltip "Configure Keybinding" (message 8893); "No matching commands" (8892).
- Settings found: `workbench.commandPalette.history`, `.preserveInput`, `.experimental.suggestCommands`, `.experimental.askChatLocation`, `.experimental.enableNaturalLanguageSearch`. All five match V in type and default.
- Not in Cursor: `workbench.commandPalette.showAskInChat` has zero hits, and the message table has no "Ask in Chat" string. So the bottom "Ask in Chat" row from V is absent. Tag for that one piece: vscode.
- Menu entries: label "Command Palette" and the "Open View..." menu label (8912) are in the message table. I did not trace each menu id in the bundle.
- Cursor commands that show up in the palette and use this widget (their output is Cursor Tab or AI debug data, so boundary):
  - "Developer: Report Cursor Tab Action" (`workbench.action.reportCppAction`; the action's English original is "See Recent Cursor Tab Actions"), opens a quick pick "Select a Cursor Tab request (last 20 requests)"
  - "Copy Recent Cursor Tab Actions" (same family)
  - "Developer: Report AI Action" (`workbench.action.reportAIAction`), opens a quick pick "Select a recent AI action"
- Proof: main bundle, search `workbench.action.showCommands`, `workbench.commandPalette.experimental.askChatLocation`, `reportCppAction`, `reportAIAction`. Message ids 8892, 8893, 8903, 8904.

### QA-2 Go to File and workspace search

- Tag: both (Cursor modified)
- Evidence: bundle
- Boundary: no
- Found in bundle: provider `vI` registered with the empty prefix and `workbench.action.quickOpen`, `commandCenterOrder: 10`. Placeholder (message 9447): "Search files, content, and symbols (append {0} to go to line or {1} to go to symbol)". V says "Search files by name (append ...)". I did not confirm that any content search happens (see unverified).
- Cursor differences that I read in the code:
  - Two search backends. The setting `search.quickOpen.useVscodeSearch` (boolean, default false; text: "When enabled, use the default VSCode quick open search algorithm instead of Cursor's modified version.", message 9637) picks between `doGetVscodePicks` and `doGetCursorPicks`.
  - Cursor's version splits the file results into a first list and an "other results" list (message 9416) when the score is below a threshold. The constants are a high-score threshold of 100 and a cap of 6 high-scoring rows. The first list is labeled "file and symbol results" or "file results" (messages 9424 and 9425, depending on `search.quickOpen.includeSymbols`).
  - A "Create File '{0}'" row with the description "Create a new file at this path" (messages 9431 and 9432, new-file icon) appears when you type something that looks like a path and nothing matches. Accepting it writes an empty file at that path (making the parent folder first if it is missing; a relative path goes under the first workspace folder) and opens it, to the side if Alt or Ctrl is held. On failure it shows "Failed to create file: {0}" (9433). This row is not in V 1.128.0 or in N 1.144.0.
  - The provider takes a `.cursorignore` check (`excludeCursorIgnore`) and `isFromComposer` flag. These are set by chat callers, not by the plain Ctrl+P box, so I count them as chat plumbing and do not catalog them.
- Same as V: "recently opened" separator, "Open to the Side" and "Open to the Bottom" and "Remove from Recently Opened" row buttons (messages 9427-9429), "{0} unsaved changes" label (9430).
- Other providers in the group, found in bundle: workspace symbols `#` (`ome`, `workbench.action.showAllSymbols`, 2 hits, placeholder 9449), Quick Search `%` (`_4a`, command `QuickTextSearchActionId`, placeholder 9451, help "Search for Text" 9452, order 25), messages 9437-9443 for the Quick Search rows (Open in Search View, See More Files, Open File, More, Enter a term to search...).
- Also found: `workbench.action.quickOpenPreviousEditor` (2 hits).
- Settings found: `search.quickOpen.includeSymbols` (default false), `search.quickOpen.includeHistory` (default true), `search.quickOpen.history.filterSortOrder` (default `default`), and the new `search.quickOpen.useVscodeSearch`.
- Proof: main bundle, search `search.quickOpen.useVscodeSearch`, `doGetCursorPicks`, `CREATE_FILE_PICK_ID`, `HIGH_SCORING_THRESHOLD`, `workbench.action.showAllSymbols`, `QuickTextSearch`.

### QA-3 Go to Line and Go to Symbol in this file

- Tag: both for Line and Symbol. Go to Offset: vscode only.
- Evidence: bundle
- Boundary: no
- Found in bundle: Go to Line provider (`EOr`, placeholder 5039), `workbench.action.gotoLine` (3 hits), message "Go to Line/Column..." in the table. Go to Symbol in Editor provider (`bce`, placeholder 5044, help 5045, `commandCenterOrder: 40`) and "by Category" help (5046). `workbench.action.gotoSymbol` (2 hits).
- Differences from V:
  - `workbench.action.gotoOffset` has zero hits, so Go to Offset and the "Toggle Zero-Based Offset" button are not in Cursor.
  - The Go to Line placeholder is the older text: "Type the line number and optional column to go to (e.g. 42:5 for line 42 and column 5)." V 1.128.0 adds the `::` offset sentence and a leading colon in the example.
- Proof: main bundle, search `workbench.action.gotoLine`, `workbench.action.gotoSymbol`, `workbench.action.gotoOffset` (absent). Message ids 5039, 5040, 5044-5046.

### QA-4 Switch and open lists

- Tag: both
- Evidence: bundle
- Boundary: no
- Found in bundle (provider, prefix source, placeholder):
  - Open View: `sdt`, placeholder 8910 "Type the name of a view, output channel or terminal to open.", help 8911, menu label 8912 "&&Open View...". `workbench.action.openView` (1 hit), `workbench.action.quickOpenView` (2 hits).
  - Editor switchers: three providers (`rIe`, `tst`, `sIe`) with the placeholder "Type the name of an editor to open it." (3412 and 3414); help text 3413 "Show Editors in Active Group by Most Recently Used" and 3415 "Show All Opened Editors By Appearance". Commands `workbench.action.showEditorsInActiveGroup` and `workbench.action.quickOpenPreviousRecentlyUsedEditor` present.
  - Terminals: `jLt`, command `workbench.action.quickOpenTerm`, placeholder 10862, help 10863.
  - Run Task: `Wln`, placeholder 9941, help 9942, `commandCenterOrder: 60`.
  - Debug: launch configurations (`w5a`, placeholder 5614, help 5615, `commandCenterOrder: 50`) and debug consoles (`h5a`, 5616 and 5617).
- Not in Cursor: `issue ` provider (no `inReportIssuePicker`, though `workbench.action.openIssueReporter` exists as a command), `mcpr ` provider, and the V `agent ` sessions provider.
- Proof: main bundle, search `registerQuickAccessProvider` (19 providers matched my pattern) and the message ids above.

### QA-5 Opening quick access and the help list

- Tag: both
- Evidence: bundle
- Boundary: no (the command center and the title-bar workspace button belong to the Title Bar section)
- Found in bundle:
  - Help provider `oqe` with prefix `?`, placeholder 8900 "Type '{0}' to get help on the actions you can take from here.", help text 8901 "Show all Quick Access Providers", `commandCenterLabel` "More" (8902), order 70. Same code shape as V: typing a prefix inside the help list switches provider.
  - `workbench.action.quickOpenWithModes` (title "Quick Open", message 3227, icon search, menu `CommandCenterCenter`). The run body is the classic path only (`includeHelp: true, from: 'commandCenter'`). The V unified-quick-access branch is not there.
  - `workbench.action.quickOpenNavigateNextInFilePicker` present. `inQuickOpen` context key present.
  - Cursor-specific entry: a title-bar button with the class `agent-workspace-quickopen` shows the workspace name and runs `workbench.action.quickOpenWithModes` on mouse down or on Enter or Space. This is a Title Bar section item that opens this section's list. It appears only on the branch where a workspace name is set.
- Command center rows (the empty-box shortcut list) use the same `commandCenterOrder` numbers as V: 10 Go to File, 20 Command Palette, 25 Search for Text, 40 Go to Symbol in Editor, 50 Start Debugging, 60 Run Task, 70 More. I did not check the Open Quick Chat row (order 30) in the bundle.
- Settings found: `workbench.quickOpen.closeOnFocusLost` (default true, message 4207) and `workbench.quickOpen.preserveInput` (default false, 4208), same as V.
- Proof: main bundle, search `workbench.action.quickOpenWithModes`, `agent-workspace-quickopen`, `commandCenterOrder`.

### QA-6 Quick pick and input box widget

- Tag: both (Cursor has its own copy of the controller)
- Evidence: bundle
- Boundary: no for the widget as a whole. One feature inside it, the row preview box, shows Cursor Tab debug data, so that feature is boundary (see Cursor additions).
- Found in bundle (same as V): the controller class `kAe` builds the same parts with the same DOM class names - `.quick-input-widget`, `.quick-input-titlebar`, `.quick-input-left-action-bar`, `.quick-input-title`, `.quick-input-right-action-bar`, `.quick-input-header`, `.quick-input-check-all`, `.quick-input-description`, `.quick-input-filter`, `.quick-input-visible-count`, `.quick-input-count`, `.quick-input-action` (OK and Custom buttons), `.quick-input-message`, `.quick-input-progress`. Labels in the table: "Toggle all checkboxes" (2248), "{0} Results" (2249), "{0} Selected" (2250), "OK" (2251), "Custom" (2252).
- Same keys and commands as V: `quickInput.nextSeparatorWithQuickAccessFallback` and `quickInput.nextSeparator` (with the Mac and non-Mac variants), `quickInput.acceptInBackground` (Right arrow), `quickInput.toggleHover`, `workbench.action.quickInputBack` (2 hits), `workbench.action.alignQuickInputTop`, `workbench.action.alignQuickInputCenter`, context keys `inQuickInput`, `quickInputAlignment`, `quickInputType`, `cursorAtEndOfQuickInputBox`.
- Same drag and snap code as V (snap threshold 20, horizontal snap ratio 0.25) and the same saved-position storage key `workbench.quickInput.viewState`.
- Different from V:
  - Width. V fixes the widget at 600 px (CSS `width: 600px`, controller `MAX_WIDTH = 600`). Cursor adds the setting `workbench.quickOpen.width` (number, default 820, minimum 580, maximum 1000, message 4209 "Controls the width in pixels of Quick Open and other quick pickers."). The controller re-lays out when it changes.
  - Preview box. Cursor adds a floating box, positioned with fixed coordinates, beside the active row (the code also has a second layout that puts it under the row; I did not work out when that one is used). It is drawn with Solid components (not the V DOM code) through a second service named `quickInputService2` (`renderPreviewBox`). It shows when the active row, or a hovered row, carries `cppReportEvent` or `customPreview`. Only `cppReportEvent` rows were found being created (the "Report Cursor Tab Action" picker). I did not find any code that sets `customPreview`.
  - No Quick Tree mode. `createQuickTree` has zero hits. `createQuickWidget` is present (2 hits).
- Proof: main bundle, search `quick-input-preview-box-content`, `quickInputService2`, `workbench.quickOpen.width`, `cppReportEvent`, `createQuickPick(e={useSeparators:!1})`, `quickInput.acceptInBackground`.

## Settings, Cursor view

Found in the main bundle in the quick access area (name, default, status against V):

- `workbench.commandPalette.history` 50, same as V
- `workbench.commandPalette.preserveInput` false, same
- `workbench.commandPalette.experimental.suggestCommands` false, same
- `workbench.commandPalette.experimental.askChatLocation` `chatView`, same
- `workbench.commandPalette.experimental.enableNaturalLanguageSearch` true, same
- `workbench.commandPalette.showAskInChat`: absent in Cursor (V only)
- `workbench.quickOpen.closeOnFocusLost` true, same
- `workbench.quickOpen.preserveInput` false, same
- `workbench.quickOpen.width` 820 (580 to 1000): Cursor only
- `search.quickOpen.includeSymbols` false, `search.quickOpen.includeHistory` true, `search.quickOpen.history.filterSortOrder` `default`: same as V
- `search.quickOpen.useVscodeSearch` false: Cursor only
- `workbench.editor.enablePreviewFromQuickOpen`: present (message in table); same text as V
- `workbench.quickInput.viewState`: stored state key, present

## Global Command Preview (boundary report)

- Question asked: does it belong in Quick access or in Editor?
- Answer: Editor (ED). It is an editor pane, not a quick access provider. Flag: boundary. No group was made for it.
- What it is: a read-only markdown page that previews one published "global" command (a slash command whose source is `global`). The tab name is `/<command name>`, the description line is "Global Command Preview", and the body lists the command text with "Source: Global command", an "Availability" line (All agents, Cloud only or Local only) and an "Arguments" line when the command has an argument hint.
- Why boundary: the content comes from the agent command store, not from the editor itself.
- Evidence (bundle, strings and ids):
  - Editor pane id `globalCommandPreviewEditor`, registered with the label "Global Command Preview"
  - Input type id `workbench.input.globalCommandPreview`, editor id `workbench.editor.globalCommandPreview`
  - URI scheme `cursor-command-preview`, authority `global`, path `/<name>.md`
  - Not saved or restored (the serializer says it cannot serialize), save and revert do nothing, icon is the markdown icon
  - A service method `openGlobalCommandPreview` opens it pinned, when a command's source is `global` and the window is not the Agents window (`isGlass` is false)
  - The telemetry map also lists `workbench.input.globalCommandPreview` as `global_command_preview`
- Connection to this section: none found. I searched for the id and label near the Command Palette code and near every quick access provider and found no link. It is not opened from a quick access list in the code I read.
- Also in the bundle: `workbench.glass.main.js` mentions it, but I did not read that file.

## Cursor additions (not in stock 1.128.0)

Tag: cursor. Evidence: bundle unless noted.

- Go to File changes (QA-2): `search.quickOpen.useVscodeSearch`; split into first results and "other results"; "Create File '{0}'" row; placeholder wording "Search files, content, and symbols". Messages 9416, 9424-9425, 9431-9433, 9447, 9637.
- Quick input width setting (QA-6): `workbench.quickOpen.width`, default 820 instead of V's fixed 600. Message 4209.
- Row preview box (QA-6): floating box next to a row, used by "Developer: Report Cursor Tab Action". Boundary, because the content is Cursor Tab debug data (request id, time, model input and output). Ids: `quickInputService2`, `renderPreviewBox`, `quick-input-preview-box-content`, `cppReportEvent`.
- Palette commands that open quick picks: `workbench.action.reportCppAction` (also a "Copy Recent Cursor Tab Actions" sibling) and `workbench.action.reportAIAction`. Boundary (Tab and AI debug output).
- Title-bar workspace button `agent-workspace-quickopen` that opens the Quick Open list. Overlap with Title Bar; the quick access side is the existing `workbench.action.quickOpenWithModes`.

## In V but not in Cursor (tag: vscode)

- Go to Offset (`workbench.action.gotoOffset`, `::` prefix, zero-based toggle button) and the newer Go to Line placeholder text. Searched: zero hits.
- "Ask in Chat" row and `workbench.commandPalette.showAskInChat`. Searched: zero hits.
- Quick Tree mode (`createQuickTree`). Zero hits. Used only by chat code in V.
- `unifiedQuickAccess` command and `chat.unifiedAgentsBar.enabled` branch of Quick Open. Zero hits for `workbench.action.unifiedQuickAccess`.
- Providers `issue `, `mcpr ` and `agent ` (V agent sessions). No registrations found. `workbench.action.chat.history` has zero hits.
- The Cursor bundle seems to carry slightly older copies of some quick access files than V 1.128.0 (Go to Line text, no Ask in Chat). I list this as an observation only. It means a V-based design of this section should be checked against Cursor for each piece before copying.

## Newer-than-1.128

- Nothing newer than 1.128.0 was seen in Cursor. Newer items from VS Code 1.144.0 are in the sibling file: link opening in Quick Open (and its "Invalid Link" row), below-anchored quick inputs, `iconColor`, motion classes. The message table has no "Invalid Link" string, so link opening is confirmed absent. I did not search for the other newer items one by one.

## Left out and why

- Agent and chat pickers. Cursor registers two: a chat and cloud agent search (prefix `chat:`, placeholder "Search chats and cloud agents", message 4786, help "Search chat history", 4787) and a recent agents switcher (prefix `agents mru `, "Switch between recently used agents", 4793). Out of scope by the brief (agent picker). The related "All Agents" command also appears next to them.
- "Select a recent AI action" quick pick (`workbench.action.reportAIAction`). The picker is out of scope; only the palette entry and the widget use are noted above.
- The `isFromComposer`, `additionPicks`, `enableEditorSymbolSearch` and `excludeCursorIgnore` options on the Go to File provider. These let the chat input reuse the file list. Chat plumbing.
- Extensions pickers (`ext `, `ext install `). Both providers are present in the Cursor bundle (placeholders 6355 and 6359) but the extension system is out of scope.
- Model picker and chat mode picker. Not searched for, since the brief puts them out of scope.
- The four Cursor built-in extensions the brief names, plus the automations bundle: no quick access content found.

## Could not verify

- Whether the Go to File box in Cursor actually matches file content. The placeholder says "files, content, and symbols" but the picks code I read only builds file picks, history picks and symbol picks. No content search call was seen.
- What sets `customPreview` on a row. The preview box checks for it, but only `cppReportEvent` was seen being set.
- When the preview box uses its under-the-row layout instead of the beside-the-row layout.
- Every menu id for the Command Palette entries (View, Help, gear, editor right-click). I confirmed the command ids and the "Command Palette" and "Open View..." labels, not each menu placement.
- Whether Cursor's Open Quick Chat shortcut row (order 30) shows. It depends on a quick-chat service I did not trace.
- The key bindings. I read them from V and confirmed that the command ids exist in the Cursor bundle, but I did not pull the Cursor key tables, so a changed default would not show up here.
- Behavior on screen. Nothing here was seen running. All of it is from code and strings.
