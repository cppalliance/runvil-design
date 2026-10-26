Group: QA-5 Opening quick access and the help list. Section: Quick access. Date: 2026-10-25. Pass: 2.
Features: 13.

### QA-5.1 Open quick access with a prefix argument
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the quick access command with a prefix argument, for example from a keybinding with args, and the box opens with that text already typed and left as is.
- Commands: workbench.action.quickOpen
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (124-152); C bundle (Go to File action with `metadata.args` prefix and `preserveValue` when a string is given, string 3228)

### QA-5.3 See shortcut rows in the empty Quick Open box
- Tag: both
- Evidence: source
- Boundary: no
- What: When Quick Open is started from the command center and the box is empty, the list begins with rows for Go to File, Command Palette, Search for Text, Open Quick Chat (only if quick chat is on), Go to Symbol in Editor, Start Debugging, Run Task and More, followed by the recently opened files; typing hides the rows.
- Commands: workbench.action.quickOpenWithModes; workbench.action.quickOpen; workbench.action.showCommands; workbench.action.quickTextSearch; workbench.action.quickchat.toggle; workbench.action.gotoSymbol; workbench.action.debug.selectandstart
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the rows and their order numbers 10, 20, 25, 40, 50, 60 and 70 are present, but there is no Open Quick Chat row)
- Source: V\src\vs\workbench\contrib\search\browser\anythingQuickAccess.ts (400-412, 824-870); V\src\vs\platform\quickinput\common\quickAccess.ts (152-165); V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts (18-28); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (21-31, 42-48); V\src\vs\workbench\contrib\search\browser\search.contribution.ts (92-98); V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoSymbolQuickAccess.ts (313-325); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (93-98); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (404-410); C bundle (`getHelpPicks` in class `vI` and the provider registrations with `commandCenterOrder`; searched "Open Quick Chat" with zero hits)

### QA-5.5 Show the help list with ?
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing ? at the start of the quick access box, or picking the More row in the command center list, shows a list of every quick access prefix with a short description.
- Commands: workbench.action.quickOpen (argument prefix "?")
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (18-55); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (21-31); C bundle (help provider class `oqe` with prefix "?", strings 8900 to 8902)

### QA-5.6 Read the provider rows in the help list
- Tag: both
- Evidence: source
- Boundary: no
- What: Rows are sorted by prefix and each shows the prefix (an ellipsis for the plain Go to File list), a description, and the key bound to the provider's command if there is one; the "?" row itself is left out.
- Commands: workbench.action.quickOpen; workbench.action.showCommands; workbench.action.gotoSymbol; workbench.action.showAllSymbols; workbench.action.quickTextSearch; workbench.action.gotoLine; workbench.action.gotoOffset; workbench.action.openView; workbench.action.showEditorsInActiveGroup; workbench.action.showAllEditors; workbench.action.showAllEditorsByMostRecentlyUsed; workbench.action.quickOpenTerm; workbench.action.debug.selectandstart; workbench.action.debug.selectDebugConsole
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (no rows for Go to Offset, Report Issue, MCP resources or agent sessions; two agent rows added, "Search chat history" and "Switch between recently used agents"; extension rows still present)
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (56-77); V\src\vs\workbench\contrib\search\browser\searchQuickAccess.contribution.ts (18-36); V\src\vs\workbench\contrib\codeEditor\browser\quickaccess\gotoLineQuickAccess.ts (98-127); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (146-168); V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminal.quickAccess.contribution.ts (21-29); C bundle (provider registrations with `helpEntries`, zero hits for `workbench.action.gotoOffset`, strings 4787 and 4794)

### QA-5.7 Pick a help row to switch to that list
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing Enter or clicking a row in the help list replaces the box text with that prefix and opens that list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (30-40); C bundle (accept handler in `oqe.provide`)

### QA-5.8 Jump to a list by typing its prefix after ?
- Tag: both
- Evidence: source
- Boundary: no
- What: Typing "?" followed by a provider's full prefix, for example "?term " with the trailing space, switches straight to that list without picking a row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\helpQuickAccess.ts (41-48); V\src\vs\platform\quickinput\common\quickAccess.ts (260-265); C bundle (value change handler in `oqe.provide`)

### QA-5.9 Switch lists by typing or deleting a prefix
- Tag: both
- Evidence: source
- Boundary: no
- What: Any prefix typed at the start of the box (@, :, #, %, "view ", "task " and so on) switches to that list and keeps the rest of the text, the longest matching prefix wins, and deleting the prefix goes back to Go to File. The > case is QA-1.3.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\quickAccess.ts (193-225, 237-250); V\src\vs\platform\quickinput\common\quickAccess.ts (225-265); C bundle (controller methods `registerPickerListeners` and `getOrInstantiateProvider`)

### QA-5.10 Keep or reselect typed text when opening another list
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening a different list while one is open moves your typed text under the new prefix (type "foo" in Go to File, press F1, and the box shows ">foo"), and opening the same list again only selects the typed text so the next key replaces it.
- Commands: workbench.action.showCommands; workbench.action.quickOpen
- Menus: none found
- Keys: F1; Ctrl+Shift+P (opens the palette from any list except Go to File, where the same keys move up the list, see QA-5.11)
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\quickAccess.ts (55-100, 172-191); C bundle (`doShowOrPick` and `adjustValueSelection` on the quick access controller)

### QA-5.11 Press a list's open key again to move through it
- Tag: both
- Evidence: source
- Boundary: no
- What: With a list open, pressing its open key again moves down and adding Shift moves up, for Go to File, Open View, the editor switcher, Open Recent and terminals.
- Commands: workbench.action.quickOpenNavigateNextInFilePicker; workbench.action.quickOpenNavigatePreviousInFilePicker; workbench.action.quickOpenNavigateNextInViewPicker; workbench.action.quickOpenNavigatePreviousInViewPicker; workbench.action.quickOpenNavigateNextInEditorPicker; workbench.action.quickOpenNavigatePreviousInEditorPicker; workbench.action.quickOpenNavigateNextInRecentFilesPicker; workbench.action.quickOpenNavigatePreviousInRecentFilesPicker; workbench.action.quickOpenNavigateNextInTerminalPicker; workbench.action.quickOpenNavigatePreviousInTerminalPicker
- Menus: none found
- Keys: Ctrl+P and Ctrl+E for next, Ctrl+Shift+P and Ctrl+Shift+E for previous (Go to File, when inQuickOpen && inFilesPicker; Mac uses Cmd+P and Cmd+Shift+P only); Ctrl+Q and Ctrl+Shift+Q (Open View, when inQuickOpen && inViewsPicker; Mac uses Control+Q, none on Linux); Ctrl+Tab and Ctrl+Shift+Tab (editor switcher, when inQuickOpen && inEditorsPicker; Mac uses Control+Tab); Ctrl+R and Ctrl+Shift+R (Open Recent, when inQuickOpen && inRecentFilesPicker; Mac uses Control+R); terminal list has no default key
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (74-98); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (120-146); V\src\vs\workbench\contrib\quickaccess\browser\viewQuickAccess.ts (228-262); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (143-144, 312-331); V\src\vs\workbench\browser\actions\windowActions.ts (470-492); V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminal.quickAccess.contribution.ts (30-33); C bundle (the same ids and keys for file, view, editor, recent files and terminal pickers; the file picker key table `Eje` is Ctrl+P with secondary Ctrl+E)

### QA-5.12 Navigate Next and Previous in Quick Open commands
- Tag: both
- Evidence: source
- Boundary: no
- What: Two palette commands move the highlight down or up the open list; they have no default key, so you bind your own, and they also arm release-to-open (QA-5.13).
- Commands: workbench.action.quickOpenNavigateNext; workbench.action.quickOpenNavigatePrevious
- Menus: CommandPalette (implicit, f1)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (205-240, 280-283); C bundle (base class `SEr` and the two subclasses, strings 3229 and 3230)

### QA-5.13 Release the held key to open the highlighted row
- Tag: both
- Evidence: source
- Boundary: no
- What: After moving through a list with a held modifier (Ctrl, Alt, Shift or Cmd, whichever the shortcut uses), letting go of that key opens the highlighted row; lists started by a cycle key such as Ctrl+Tab open with the input line hidden.
- Commands: workbench.action.quickOpenNavigateNext; workbench.action.quickOpenNavigatePrevious
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\platform\quickinput\browser\quickInput.ts (974-1025); V\src\vs\platform\quickinput\browser\quickAccess.ts (112-123); V\src\vs\platform\quickinput\browser\quickInputController.ts (860-867); C bundle (`registerQuickNavigation` key up handler and `hideInput` set in `doShowOrPick`)

### QA-5.14 Select Next and Previous in Quick Open
- Tag: both
- Evidence: source
- Boundary: no
- What: On Mac, Control+N and Control+P move the highlight down and up in any open list; on Windows and Linux the commands exist with no default key.
- Commands: workbench.action.quickOpenSelectNext; workbench.action.quickOpenSelectPrevious
- Menus: CommandPalette (implicit, f1)
- Keys: Ctrl+N (Mac, Control key, when inQuickOpen); Ctrl+P (Mac, Control key, when inQuickOpen)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (243-280); V\src\vs\workbench\browser\quickaccess.ts (20-25); C bundle (the two actions with Mac `WinCtrl` N and P keys, strings 3231 and 3232)

### QA-5.15 Close the open list with Escape
- Tag: both
- Evidence: source
- Boundary: no
- What: Escape or Shift+Escape closes any quick access list without picking a row.
- Commands: workbench.action.closeQuickOpen
- Menus: none found
- Keys: Escape; Shift+Escape (when inQuickOpen)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\quickAccessActions.ts (30-40); V\src\vs\workbench\browser\quickaccess.ts (20-22); C bundle (key rule for `workbench.action.closeQuickOpen` with Escape and Shift+Escape)

## Handed off

- Menu routes to Go to File (Go menu "Go to File...", "Open File..." in the empty editor and tab strip right-click menus): QA-2.1 already lists them as entry points.
- Opening Go to File with the previous file ready (`workbench.action.quickOpenPreviousEditor`), restoring the last typed text (`workbench.quickOpen.preserveInput`) and closing on focus loss (`workbench.quickOpen.closeOnFocusLost`): QA-2 (QA-2.18, QA-2.17 and QA-2.19).
- The commands with no default key that drive the open list from your own keys (`workbench.action.acceptSelectedQuickOpenItem`, `workbench.action.alternativeAcceptSelectedQuickOpenItem`, `workbench.action.focusQuickOpen`, `workbench.action.quickPickManyToggle`) and the multi-step Back key (`workbench.action.quickInputBack`): QA-6.
- Drawing of the command center box, its label, hover and the window title text: TB-2.
- The Open Recent list that Ctrl+R cycles: TB-1 (File menu, Open Recent).
- Editor watermark tips "Show All Commands" and "Go to File": ED-9 (ED-9.18).
- Terminal skip-shell list that keeps Ctrl+P and Ctrl+Shift+P working inside the terminal (`terminal.integrated.commandsToSkipShell`): PN-3.

## Unclear

- QA-5.2 and QA-5.4 are also listed in the Title Bar file as TB-2.1 and TB-2.9, because the catalog text and the request for this group both name the command center and the workspace button. The parent should keep one copy of each.
- The cycle keys for the view, editor and terminal lists are listed once here in QA-5.11, but pass 1 also named the view cycle keys under QA-4. QA-4 should point here instead of repeating them.
- Stock `quickOpenWithModes` can switch to a unified agents quick access when the chat setting `chat.unifiedAgentsBar.enabled` is on. That branch is chat and was left out. Cursor's bundle has zero hits for `workbench.action.unifiedQuickAccess`.
- The "Open Quick Chat" row (QA-5.3) belongs to chat. I listed it only inside the row order. Cursor has the quick chat command id but no such row.
- In V the Report Issue row only appears when `extensions.experimental.issueQuickAccess` is on (V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts, 102-114), so the help list in stock can differ by that setting. I did not check the Cursor equivalent beyond zero hits for the `inReportIssuePicker` key in pass 1.
- Cursor registers a Ctrl+Tab cycle pair for its agents picker (`workbench.action.quickOpenNavigateNextInAgentsPicker`) next to the editor switcher pair. That picker is out of scope, so it is not listed.
- Cursor's help provider reads the provider list without a context key service, while V filters providers by a `when` expression. I found no V provider that sets `when`, so I saw no visible difference.
- Nothing newer than 1.128 was found for this group. A diff of the quick access source files between V and N shows only API options (`providerPrefix`, `anchor`) and a setting id rename, no new on-screen feature.
- Nothing was run on screen. Everything here comes from reading source and bundle fragments.
