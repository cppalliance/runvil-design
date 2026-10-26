Bucket: R. Date: 2026-10-25. Pass: 4.

### NEW-R-1 Close Other Windows
- Group: WB-9
- Tag: vscode
- Evidence: source
- Boundary: no
- What: The person runs Close Other Windows from the Command Palette and every other app window is asked to close, leaving the window they ran it in.
- Commands: workbench.action.closeOtherWindows
- Menus: CommandPalette (f1, no category)
- Keys: none found
- Settings: none found
- Cursor: missing - neither the id nor the label "Close Other Windows" is in the main bundle
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts (CloseOtherWindowsAction, lines 69-93); V\src\vs\platform\native\electron-main\nativeHostMainService.ts (getWindows, lines 217-235; closeWindow, lines 1113-1116); C bundle (no hit for the id or the label)
- Pieces:
  1. Typing Close Other Windows in the Command Palette lists the command with no category and no default key. [stock only]
  2. Running it closes every other main window of the app and keeps the window it was run from. [stock only]
  3. Each of those windows is closed with the same call its own close button uses, so a window that has unsaved changes follows its normal close flow and may still ask first (see DL-2). [stock only]
  4. Floating editor windows are not closed one by one, because only main windows are listed, so they go away with their parent window. [stock only]
  5. With a single window open, running the command changes nothing. [stock only]

### NEW-R-2 Choose the size and place of a new window
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: The person picks, through `window.newWindowDimensions`, whether a new window opens centered, like the last active window, offset from it, maximized or full screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.newWindowDimensions
- Cursor: present - the setting, its five values and its default are in the bundle
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 218-231); C bundle (setting `window.newWindowDimensions` with enum default, inherit, offset, maximized, fullscreen and default default)
- Pieces:
  1. Setting `window.newWindowDimensions` chooses how a new window is sized and placed, with values `default`, `inherit`, `offset`, `maximized` and `fullscreen`.
  2. `default`, the default, opens each new window in the center of the screen.
  3. `inherit` opens a new window with the same size as the last active window, and `offset` does the same with an offset position.
  4. `maximized` opens each new window maximized and `fullscreen` opens it in full screen mode.
  5. The setting applies only when at least one window is already open, because the first window always restores the size and place it had when it was closed.
  6. The setting is an application setting, so a workspace cannot override it.

### NEW-R-3 Close the window when its last editor closes
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: With `window.closeWhenEmpty` on, closing the last open editor of a window that shows no folder also closes the window.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.closeWhenEmpty
- Cursor: present - the setting and the close-when-empty code that reads it are in the bundle; the message strings were not compared
- Source: V\src\vs\workbench\electron-browser\window.ts (maybeCloseWindow, lines 625-651; listener, line 395); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 232-236); C bundle (setting `window.closeWhenEmpty` and method maybeCloseWindow)
- Pieces:
  1. Setting `window.closeWhenEmpty` is off by default, and while it is off closing the last editor leaves the empty window open.
  2. With the setting on, closing the last editor in a window that has no folder or workspace closes that window.
  3. A window that has a folder or workspace open stays open when its last editor closes.
  4. The window also stays open while other windows, such as floating editor windows, still have editors open, and while an extension is being developed in it.
  5. With the setting on, a floating editor window closes when its last editor closes, even when a folder is open in the main window.
  6. The check is debounced, so opening another editor right after the last one closes keeps the window open.

### NEW-R-4 Choose what a second launch without files does
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: The person picks, through `window.openWithoutArgumentsInNewWindow`, whether starting the app again with nothing to open creates a new empty window or only brings the last active window forward.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.openWithoutArgumentsInNewWindow
- Cursor: present - setting, values and the macOS-aware default are in the bundle
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 172-182); C bundle (setting `window.openWithoutArgumentsInNewWindow` with enum on and off and default tied to the macOS flag)
- Pieces:
  1. Setting `window.openWithoutArgumentsInNewWindow` chooses what happens when a second instance starts with no folder or file, with values `on` and `off`.
  2. `on` opens a new empty window.
  3. `off` focuses the last active running window instead of opening anything.
  4. The default is `on` on Windows and Linux and `off` on macOS.
  5. The setting is an application setting, and the setting text says it can still be ignored when the new-window or reuse-window command line option is used.

### NEW-R-5 Toggle Screen Reader Accessibility Mode
- Group: WB-4
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Toggle Screen Reader Accessibility Mode, or presses Shift+Alt+F1, and the editor switches between screen reader optimized mode and normal mode, with a spoken message saying which one is now on.
- Commands: editor.action.toggleScreenReaderAccessibilityMode
- Menus: CommandPalette (f1, no category)
- Keys: Shift+Alt+F1 (Shift+Alt+F4 on Linux); Ctrl+E (Cmd+E on macOS) while the accessibility help dialog is shown (accessibilityHelpIsShown)
- Settings: editor.accessibilitySupport
- Cursor: present - the id, the label and both key rules are in the bundle; the two messages come through NLS indexes 898 and 899
- Source: V\src\vs\workbench\contrib\codeEditor\browser\accessibility\accessibility.ts (ToggleScreenReaderMode, lines 22-54); V\src\vs\editor\common\config\editorOptions.ts (accessibilitySupport, lines 1452-1465); V\src\vs\editor\common\standaloneStrings.ts (lines 22-23); C bundle (class with id editor.action.toggleScreenReaderAccessibilityMode and the two key rules with weight 210)
- Pieces:
  1. Running Toggle Screen Reader Accessibility Mode from the Command Palette writes `editor.accessibilitySupport` into the User settings, as `on` when screen reader mode was off and as `off` when it was on.
  2. A screen reader announcement says "Screen Reader Optimized Mode enabled." or "Screen Reader Optimized Mode disabled." to match the new state.
  3. Shift+Alt+F1 (Shift+Alt+F4 on Linux) runs the command from anywhere.
  4. While the accessibility help dialog is open, Ctrl+E (Cmd+E on macOS) runs the command, so the mode can be switched without closing the dialog.
  5. When the setting is `auto` and a screen reader is detected, the mode already counts as on, so running the command sets the setting to `off`.
  6. While the mode is on, the status bar shows the Screen Reader Optimized item (see SS-3.26).

### NEW-R-6 Announce Cursor Position
- Group: WB-4
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With the editor focused and screen reader mode on, the person presses Ctrl+Alt+Shift+G and the screen reader says the cursor's line and column.
- Commands: editor.action.announceCursorPosition
- Menus: CommandPalette (f1, no category)
- Keys: Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) when editorTextFocus and accessibilityModeEnabled
- Settings: none found
- Cursor: missing - neither the id nor the label "Announce Cursor Position" is in the main bundle
- Source: V\src\vs\workbench\contrib\codeEditor\browser\accessibility\accessibility.ts (AnnounceCursorPosition, lines 56-92); V\src\vs\platform\accessibility\common\accessibility.ts (accessibilityModeEnabled, line 40); C bundle (no hit for the id or the label)
- Pieces:
  1. Typing Announce Cursor Position in the Command Palette runs the command, and it is listed whether or not screen reader mode is on. [stock only]
  2. Ctrl+Alt+Shift+G (Cmd+Alt+Shift+G on macOS) runs it, but only while an editor has text focus and screen reader mode is on. [stock only]
  3. The screen reader announcement reads "Line {line}, Column {column}" for the primary cursor. [stock only]
  4. The column is the visible column, so a tab counts as its width, which matches the position shown in the status bar. [stock only]
  5. With no focused editor, or an editor with no text model, nothing is announced. [stock only]

### NEW-R-7 Developer: Inspect Key Mappings
- Group: WB-8
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Developer: Inspect Key Mappings, or its JSON form, and a new untitled tab opens with a dump of the keyboard layout and key mappings, used to find out why a key does not do what it should.
- Commands: workbench.action.inspectKeyMappings; workbench.action.inspectKeyMappingsJSON
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Cursor: present - both class ids and titles are in the bundle, with the Developer category and the same untitled editor call
- Source: V\src\vs\workbench\contrib\codeEditor\browser\inspectKeybindings.ts (lines 14-54); V\src\vs\workbench\services\keybinding\browser\keybindingService.ts (_dumpDebugInfo, lines 395-401; _dumpDebugInfoJSON, lines 403-409); C bundle (classes with ids workbench.action.inspectKeyMappings and workbench.action.inspectKeyMappingsJSON, NLS 5029 and 5030)
- Pieces:
  1. The Command Palette lists Developer: Inspect Key Mappings and Developer: Inspect Key Mappings (JSON), and neither has a key or a menu row.
  2. Inspect Key Mappings opens a pinned untitled editor whose text starts with "Layout info:" and the current keyboard layout, then lists the resolved keybindings and the keyboard mapper details, and ends with "Raw mapping:" and the raw key mapping.
  3. Inspect Key Mappings (JSON) opens a pinned untitled editor that holds one JSON object with the two entries `layout` and `rawMapping`.
  4. Both open as untitled files, so nothing is written to disk until the person saves the tab.

### NEW-R-8 Developer: Open URL
- Group: WB-8
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Developer: Open URL, types an address in the input box, and the app hands it to its URL service as if the link had been opened from outside.
- Commands: workbench.action.url.openUrl
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Cursor: changed - the id, the title, the Developer category and the "URL to open" prompt are in the bundle, but Cursor's box starts empty and keeps no last input
- Source: V\src\vs\workbench\contrib\url\browser\url.contribution.ts (OpenUrlAction, lines 27-57); C bundle (class with id workbench.action.url.openUrl, title "Open URL", category Developer, prompt message 11526, no storage read)
- Pieces:
  1. The Command Palette lists Developer: Open URL, with no key and no menu row.
  2. Running it shows an input box with the prompt "URL to open".
  3. Pressing Enter with text in the box reads the text as an address and passes it to the URL service, which gives it to the handler registered for that address, and the command shows nothing of its own.
  4. Pressing Escape, or accepting an empty box, does nothing.
  5. In stock the box starts filled with the last address entered in this workspace, and each accepted address is remembered for the next run. [stock only]
  6. In Cursor the box always starts empty and nothing is remembered. [cursor only]

### NEW-R-9 Rerun All Running Tasks
- Group: WB-2
- Tag: vscode
- Evidence: source
- Boundary: no
- What: The person runs Tasks: Rerun All Running Tasks from the Command Palette and every task that is running is ended and started again at once.
- Commands: workbench.action.tasks.rerunAllRunningTasks
- Menus: CommandPalette (Tasks category, when tasks can run)
- Keys: none found
- Settings: none found
- Cursor: missing - neither the id nor the label "Rerun All Running Tasks" is in the main bundle
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (palette entry, lines 289-296); V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (registration, lines 634-638; _runRerunAllRunningTasksCommand, lines 3535-3546; _restart, lines 2158-2185); V\src\vs\workbench\contrib\tasks\common\tasks.ts (line 1442); C bundle (no hit for the id or the label)
- Pieces:
  1. The Command Palette lists Tasks: Rerun All Running Tasks whenever the app can run tasks, and the command has no default key. [stock only]
  2. Running it restarts every running task in one go, ending each one and then starting it again. [stock only]
  3. Before each restart the task is looked up again in the task configuration, so edits made to its definition since it started are used. [stock only]
  4. A task that cannot be ended gets the warning "Failed to terminate and restart task {name}" and is not started again, while the other tasks are still restarted. [stock only]
  5. With no task running, an information notification says "No running tasks to restart". [stock only]
  6. The command first checks Workspace Trust, as the other task commands do. [stock only]

### NEW-R-10 Find all references to a Markdown file
- Group: SB-7
- Tag: both
- Evidence: source
- Boundary: no
- What: The person right-clicks a Markdown file in the Explorer, or its tab, and picks Find File References to list every link in the workspace that points to that file in the References view; scope note: this works on Markdown sources but only navigates, so it is kept apart from the Markdown source editing that was ruled out.
- Commands: markdown.findAllFileReferences
- Menus: explorer/context (group 4_search); editor/title/context; commandPalette (when editorLangId is Markdown)
- Keys: none found
- Settings: references.preferredLocation (read, set to view for the call and put back)
- Cursor: changed - the id, the label and all three menu entries are in the Cursor manifest, but the when clauses name only the markdown language, where stock also names prompt, instructions, chat agent and skill files
- Source: V\extensions\markdown-language-features\package.json (command, lines 188-191; explorer/context, lines 293-304; editor/title/context, lines 305-315; commandPalette, lines 365-368); V\extensions\markdown-language-features\src\languageFeatures\fileReferences.ts (lines 14-60); C extensions (markdown-language-features package.json and package.nls.json with title "Find File References")
- Pieces:
  1. Right-clicking a Markdown file in the Explorer shows Find File References in the search group of the menu.
  2. Right-clicking the tab of a Markdown file shows the same entry.
  3. The Command Palette lists Markdown: Find File References while the active editor is a Markdown file, and the command then uses that editor's file.
  4. While it searches, the window shows progress with the text "Finding file references".
  5. The results open in the Reference Search Results view (see SB-7), even when `references.preferredLocation` is set to peek, because the command sets the setting to view for the call and then puts the old value back.
  6. Each result is a link in a Markdown file that points at the chosen file, listed under the file that holds the link.
  7. When there is no file to search for, an error message says "Find file references failed. No resource provided."
  8. In stock the entry also shows for prompt, instructions, chat agent and skill files, and Cursor's menu entries list only Markdown files. [stock only]

### NEW-R-11 Quick Open Browser Tab
- Group: ED-10
- Tag: vscode
- Evidence: source
- Boundary: yes - the stock Browser tab is shared with agents.
- What: While a Browser tab is active the person presses Ctrl+Shift+A, or runs Browser: Quick Open Browser Tab..., and gets a list of every open Browser tab to jump to or close.
- Commands: workbench.action.browser.quickOpen
- Menus: CommandPalette (category Browser)
- Keys: Ctrl+Shift+A (Cmd+Shift+A on macOS) when a Browser editor is active
- Settings: none found
- Cursor: missing - the id and the label "Quick Open Browser Tab" are not in the main bundle
- Source: V\src\vs\workbench\contrib\browserView\electron-browser\features\browserTabManagementFeatures.ts (BrowserTabQuickPick, lines 74-235; QuickOpenBrowserAction, lines 237-259; registration, line 545); V\src\vs\platform\browserView\common\browserView.ts (lines 12-19); C bundle (searched workbench.action.browser and the label, no hit)
- Pieces:
  1. Ctrl+Shift+A (Cmd+Shift+A on macOS) while a Browser tab is active, or Browser: Quick Open Browser Tab... in the Command Palette, opens a list with the placeholder "Select a browser tab". [stock only]
  2. Every open Browser tab is a row with its icon, page name and description, and typing narrows the rows by name or description. [stock only]
  3. When Browser tabs sit in more than one editor group, each group gets a heading with the group's name, and tabs that are open in no group sit under a "Background" heading. [stock only]
  4. The row of the active tab starts highlighted, and tabs that are not pinned show in italics. [stock only]
  5. Choosing a row opens that tab, in its own editor group when it has one. [stock only]
  6. Each row has a Close button that closes that tab and leaves the list open, and a Close All button next to the input closes every Browser tab. [stock only]
  7. The list updates live while it is open, as tabs open, close or change their name. [stock only]
  8. A row "New Integrated Browser Tab" with a plus icon is always shown below a separator, whatever is typed, and opens a blank tab. [stock only]
  9. The globe button in the title bar opens this same list when a Browser tab is already open (see TB-5.9). [stock only]

### NEW-R-12 Confirm before closing the window with a debug session running
- Group: DL-2
- Tag: both
- Evidence: source
- Boundary: no
- What: With `debug.confirmOnExit` set to always, closing, quitting or reloading the window while a debug session is live shows a warning box that asks whether to stop it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.confirmOnExit
- Cursor: present - the setting with values never and always and the default never, and the shutdown check with the veto reason veto.debug, are in the bundle; the box strings come through NLS indexes
- Source: V\src\vs\workbench\contrib\debug\common\debugLifecycle.ts (lines 14-58); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 663-672); C bundle (setting `debug.confirmOnExit`; class with onBeforeShutdown veto "veto.debug")
- Pieces:
  1. Setting `debug.confirmOnExit` chooses whether the window asks before closing with live debug sessions, with values `never` (the default) and `always`.
  2. With `always` and one debug session running, closing the window shows a warning box "There is an active debug session, are you sure you want to stop it?" with a Stop Debugging button and a Cancel button.
  3. With several sessions running the box reads "There are active debug sessions, are you sure you want to stop them?"
  4. Only top-level sessions are counted, so sessions started by another session do not add to the number.
  5. Cancel keeps the window open and the sessions running, and Stop Debugging lets the window close.
  6. The check does not look at why the window is shutting down, so quitting and reloading ask the same question.
  7. With `never`, or with no session running, no box shows.

### NEW-R-13 Choose the browser that opens external links
- Group: DL-5
- Tag: both
- Evidence: source
- Boundary: no
- What: The person sets `workbench.externalBrowser` to a browser name or the path of its program, and web links that leave the app open in that browser instead of the system default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.externalBrowser
- Cursor: present - the setting with its restricted flag and the opener code that reads it are in the bundle
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (lines 34-39); V\src\vs\platform\native\electron-main\nativeHostMainService.ts (openExternal, lines 658-671; openExternalBrowser, lines 673-712); V\src\vs\workbench\electron-browser\window.ts (setupOpenHandlers, lines 872-886); C bundle (setting `workbench.externalBrowser` with restricted true; setupOpenHandlers reading it)
- Pieces:
  1. Setting `workbench.externalBrowser` is empty by default, and then http and https links that leave the app open in the system default browser.
  2. Setting it to a browser name such as `edge`, `chrome` or `firefox` opens those links in that browser.
  3. Setting it to the absolute path of a browser program opens those links with that program.
  4. If a path value points at nothing, the problem is only written to the log and the system default browser opens the link.
  5. If the chosen browser fails to start, the link opens in the system default browser instead.
  6. Only http and https links use the setting, and other kinds of links go to the system's own handler for them.
  7. The setting is marked restricted, so a value set by an untrusted workspace is not applied, and it is offered only in the desktop app.
