Group: WB-9 Windows and workspaces. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 12.

### WB-9.1 Duplicate the folders as an untitled workspace in a new window
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Workspaces: Duplicate As Workspace in New Window" and a new window opens on an untitled workspace that holds the same folders as the current window.
- Commands: workbench.action.duplicateWorkspaceInNewWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the command, label and multi-folder condition are present; the stock Agents window condition is not)
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts:292-325; C bundle (class with id workbench.action.duplicateWorkspaceInNewWindow, category Workspaces, precondition without an Agents window check)

### WB-9.2 Open the workspace configuration file
- Tag: both
- Evidence: source
- Boundary: no
- What: When a workspace file is open, "Workspaces: Open Workspace Configuration File" opens that file in an editor tab.
- Commands: workbench.action.openWorkspaceConfigFile
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the command, label and workspace-only condition are present; the stock Agents window condition is not)
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts:188-210; C bundle (class with id workbench.action.openWorkspaceConfigFile, category Workspaces, precondition is only the workspace state)

### WB-9.3 Install or remove the cursor and code shell commands
- Tag: both
- Evidence: source
- Boundary: no
- What: Four Shell Command entries put the cursor or code command on the path or take it off, and Cursor also shows a one-time notification offering to do it.
- Commands: workbench.action.installCommandLine; workbench.action.uninstallCommandLine; workbench.action.replaceCommandLine; workbench.action.restoreCommandLine
- Menus: CommandPalette; CommandPalette (Shell Command category)
- Keys: none found
- Settings: none found
- Cursor: changed (stock has only install and uninstall of its own command, macOS only; Cursor keeps those, reworded, and adds the two code commands and the prompt)
- Source: V\src\vs\workbench\electron-browser\actions\installActions.ts:15-80; V\src\vs\workbench\electron-browser\desktop.contribution.ts:65-69; C bundle (four classes in category Shell Command, NLS 12002-12010; registration Li&&(Xe(y4w),Xe(w4w)),Xe(S4w),Xe(k4w) where Li is the macOS flag; prompt code reading cmdLineHookState with buttons Install 'code', Install 'cursor' and Never show again); V\src\vs\workbench\electron-browser\actions\installActions.ts:19-80; V\src\vs\workbench\electron-browser\desktop.contribution.ts:65-69 (macOS only); C bundle (ids workbench.action.installCommandLine and uninstallCommandLine, category "Shell Command" nls 12007, titles nls 12008 to 12010, success text nls 12002 and 12006, registration behind the macOS check)

### WB-9.4 Bring a window to the front by command
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run Switch to Main Window from a secondary editor window, or Focus Window from anywhere, to bring a window to the front and give it focus.
- Commands: workbench.action.switchToMainWindow; workbench.action.focusWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing - 0 hits for both ids in the bundle
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts:393-432; V\src\vs\workbench\electron-browser\desktop.contribution.ts:44-45; C bundle (searched both ids, 0 hits)

### WB-9.5 Offer to open a workspace file found in the folder
- Tag: both
- Evidence: source
- Boundary: no
- What: When you open a folder that has a .code-workspace file at its top level, a notification offers to open that workspace file instead.
- Commands: workspaces.dontPromptToOpen (the Don't Show Again id, not a command); workbench.action.openWorkspaceFromEditor (related, the Open Workspace button on a .code-workspace file, named in ED-5)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - the notification class, id and strings (nls 11961 to 11963) are in the bundle
- Source: V\src\vs\workbench\contrib\workspaces\browser\workspaces.contribution.ts:26-108; C bundle (class with doHandleWorkspaceFiles, neverShowAgain id workspaces.dontPromptToOpen with workspace scope)

### WB-9.6 macOS Touch Bar buttons
- Tag: both
- Evidence: source
- Boundary: no
- What: On a Mac that has a Touch Bar, the strip above the keyboard shows buttons for going back and forward and for running and stepping a debug session, and two settings turn it off or hide single buttons.
- Commands: workbench.action.navigateBack; workbench.action.navigateForward; workbench.action.debug.run; workbench.action.debug.start; workbench.action.debug.continue; workbench.action.debug.pause; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop
- Menus: TouchBarContext (group navigation for Go Back and Go Forward, group 9_debug for the debug buttons)
- Keys: none found
- Settings: keyboard.touchbar.enabled; keyboard.touchbar.ignored
- Cursor: present - TouchBarContext, the back, forward and debug rows and both settings were found in the bundle, and nls 12071 holds the enable description
- Source: V\src\vs\workbench\electron-browser\window.ts (lines 158-170, 377-381 and 905-966); V\src\vs\workbench\electron-browser\desktop.contribution.ts (lines 360-380); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 338-350); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 229-253); V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts (touchBar key); C bundle (TouchBarContext rows with the back, forward and debug entries, both settings)

### WB-9.7 Open Recent picker
- Tag: both
- Evidence: source
- Boundary: no
- What: The person presses Ctrl+R or runs Open Recent from the Command Palette and gets a list of recent folders, workspaces and files to open, with a button on each row to remove it and, in stock only, a marker for folders and workspaces already open in a window.
- Commands: workbench.action.openRecent; workbench.action.quickOpenRecent
- Menus: MenubarRecentMenu (More..., group y_more, order 1); CommandPalette (Open Recent only, category File; Quick Open Recent has f1 false and is not listed)
- Keys: Ctrl+R on workbench.action.openRecent (the Control key on macOS too); none found for workbench.action.quickOpenRecent
- Settings: none found
- Cursor: changed - both command ids, the context key inRecentFilesPicker, the headings "folders & workspaces" and "files", the placeholder "Select to open (hold Ctrl-key to force new window or Alt-key for same window)", the remove button and the two unsaved-files tooltips are in the bundle; the four "Opened in a Window" and "Opened in Active Window" tooltips and the code that adds those buttons are not
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (BaseOpenRecentAction lines 47-278, OpenRecentAction lines 280-309, QuickPickRecentAction lines 311-326); C bundle (class with id workbench.action.openRecent and key R, class with id workbench.action.quickOpenRecent and title "Quick Open Recent...", strings "folders & workspaces", "Remove from Recently Opened", "Folder With Unsaved Files"; no hit for "Folder Opened in a Window")

### WB-9.8 Switch Window picker
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Switch Window... to see every open window in a list, picks one to bring it to the front, or closes a window from its row.
- Commands: workbench.action.switchWindow; workbench.action.quickSwitchWindow
- Menus: CommandPalette (Switch Window... only; Quick Switch Window has f1 false and is not listed); the macOS Window menu row Switch Window... (see TB-1.25)
- Keys: Ctrl+W on macOS (the Control key) on workbench.action.switchWindow; none found on Windows and Linux; none found for workbench.action.quickSwitchWindow
- Settings: none found
- Cursor: changed - both command ids, the context key inWindowsPicker, the placeholder "Select a window to switch to", the "window group" heading, the "Current Window" description, the "{0}, window with unsaved changes" label and a Close Window button were found in the bundle; the separate Close Active Window button for the window you are in was not, and Cursor's current-window row has no button
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts (BaseSwitchWindow lines 233-356, SwitchWindowAction lines 358-376, QuickSwitchWindowAction lines 378-391); C bundle (classes with ids workbench.action.switchWindow and workbench.action.quickSwitchWindow, closeWindowAction and closeDirtyWindowAction only, strings 12029 "Select a window to switch to" and "window group")

### WB-9.9 Close Other Windows
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

### WB-9.10 Choose the size and place of a new window
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

### WB-9.11 Close the window when its last editor closes
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

### WB-9.12 Choose what a second launch without files does
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

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
