Group: TB-2 Command center and window title. Section: Title bar (TB). Date: 2026-10-25. Pass: 2.
Features: 16.

### TB-2.1 Open Quick Open from the command center box
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the search-style box in the middle of the title bar to open Quick Open with the help entry included. In stock, when the chat setting `chat.unifiedAgentsBar.enabled` is on, the click opens the unified quick access instead (chat feature, not cataloged). The box is on by default in stock and off by default in Cursor.
- Commands: workbench.action.quickOpenWithModes
- Menus: CommandCenterCenter (order 100); CommandCenter (submenu titled "Command Center", order 101); CommandCenterCenter (order 100, search icon); CommandCenter (submenu "Command Center", order 101)
- Keys: none found
- Settings: window.commandCenter
- Cursor: changed - present, but `window.commandCenter` defaults to false in Cursor (true in stock), and the click always opens classic Quick Open
- Source: V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; V\src\vs\workbench\browser\actions\quickAccessActions.ts; C bundle (modules `commandCenterControl` and `quickAccessActions`, registration of `window.commandCenter`); V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts (93-110, 252-270); V\src\vs\workbench\browser\actions\quickAccessActions.ts (154-196); V\src\vs\workbench\browser\workbench.contribution.ts (860-867); C bundle (action registered on `CommandCenterCenter` with title string 3227, setting block with default false, hover strings 3993 to 3995)

### TB-2.2 Show or hide the command center
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn the center box on or off. Off collapses the middle of the bar to plain window title text and lowers the bar from 35 px to 30 px. In stock, turning on Navigation Controls or the experimental Share setting turns the box on if it was off. Compact floating editor windows never draw the box.
- Commands: toggle.window.commandCenter
- Menus: TitleBarContext; TitleBarTitleContext (group 2_config, order 1, shown when the title bar is not compact)
- Keys: none found
- Settings: window.commandCenter
- Cursor: changed - setting present with default false; the right-click row and the id `toggle.window.commandCenter` are not in the bundle (the `titlebarActions` module has no Command Center string)
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\browser\layout.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; C bundle (settings block for `window.commandCenter`, module `vs/workbench/browser/parts/titlebar/titlebarActions`)

### TB-2.3 Command center label and hover text
- Tag: both
- Evidence: source
- Boundary: no
- What: The box shows the workspace name. With editor tabs hidden it shows the active file name instead. With a customized `window.title` or a screen reader on, it shows the whole formatted title. If there is nothing to show it says "Search". The extension-development and admin markers are added before and after the label, and line breaks show as a return symbol. Hover reads "Search {workspace} ({key})" then an em dash and the title; the key part only appears when Quick Open has a binding. Colors come from the `commandCenter.*` theme colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.title; window.titleSeparator; workbench.editor.showTabs
- Cursor: present - same label logic and strings (`label.dfl`, `title`, `title2`); stock's compact look (icon hidden, label left) depends on `chat.agentsControl.enabled`, which is not in the bundle, so Cursor always shows the search icon
- Source: V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (module `commandCenterControl`, strings 3990-3994, label function in the Quick Open view item class)

### TB-2.4 Go back and forward arrows
- Tag: both
- Evidence: source
- Boundary: no
- What: Two arrow buttons to the left of the box step back and forward through editor navigation history. Each is disabled when there is nowhere to go (context keys `canNavigateBack` and `canNavigateForward`). In stock they live on the command center menu, so they show only while the box is on.
- Commands: workbench.action.navigateBack; workbench.action.navigateForward
- Menus: CommandCenter (Back order 1, Forward order 2, when `config.workbench.navigationControl.enabled`); MenubarGoMenu (group 1_history_nav)
- Keys: Back - Alt+Left (Windows), Ctrl+- (macOS), Ctrl+Alt+- (Linux), plus the browser Back key; Forward - Alt+Right (Windows), Ctrl+Shift+- (macOS and Linux), plus the browser Forward key
- Settings: workbench.navigationControl.enabled
- Cursor: changed - arrows exist on the command center menu; when the box is off, Cursor draws the same two commands as buttons in its own left title bar toolbar (strings 4040-4041, enabled by the same context keys)
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts; C bundle (editor actions module with the `CommandCenter` menu rows for `workbench.action.navigateBack`, title bar part `createActionForId`)

### TB-2.5 Show or hide navigation controls
- Tag: both
- Evidence: source
- Boundary: no
- What: Hide or show the back and forward arrows from the title bar right-click menu or the setting. The row appears only when the title bar is not compact and the command center is on. In Cursor the same setting also gates the arrows that Cursor puts in its left toolbar when the box is off.
- Commands: toggle.workbench.navigationControl.enabled
- Menus: TitleBarContext; TitleBarTitleContext (group 2_config, order 2)
- Keys: none found
- Settings: workbench.navigationControl.enabled
- Cursor: present - generic toggle class builds the id as `toggle.` plus the setting name; row text in strings 4014-4015; row shows when `config.window.commandCenter` is on; default of the setting is true in both
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (module `vs/workbench/browser/parts/titlebar/titlebarActions`, setting registration)

### TB-2.6 Title bar right-click menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click anywhere on the title bar, including on the box and on toolbar buttons, to open a menu of title bar toggles. On Windows with a custom title bar, the OS system-menu request is redirected to this same menu. Each row is its own feature in the group that owns the thing toggled (see Handed off); this feature is the menu itself.
- Commands: none found
- Menus: TitleBarContext
- Keys: none found
- Settings: none found
- Cursor: changed - present, with different rows: it adds Agents Window Button (agent, not cataloged) and, on Linux only, Restore Native Title Bar; it has no Command Center, Notifications, Integrated Browser, Hide Custom Title Bar or Hide Custom Title Bar In Full Screen row
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; C bundle (`TitleBarContext` 13 hits, `onDidTriggerWindowSystemContextMenu`, module `titlebarActions`)

### TB-2.7 Title right-click menu with folder path (macOS)
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS, right-click on the title text, or Cmd+click on it, opens a second menu with the same toggles plus one row per folder in the active file's path and one for the file. Picking a row reveals it in Finder. The path rows appear only with a custom title bar and an active file. Cmd+click is caught first so it does not open Quick Open.
- Commands: workbench.action.revealPathInFinder<n> (one id generated per path segment, n is the segment index)
- Menus: TitleBarTitleContext (path rows in group 1_file, negative orders)
- Keys: none found (mouse: Cmd+click on the title text)
- Settings: none found
- Cursor: present - same menu id, same generated command ids and Cmd+click handler found in the bundle
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\window.ts; C bundle (`TitleBarTitleContext` 9 hits, `workbench.action.revealPathInFinder` generator)

### TB-2.8 Window title as plain text when the command center is off
- Tag: both
- Evidence: source
- Boundary: no
- What: With the box off, the middle of the bar shows the window title string as centered plain text, clipped with an ellipsis, and it updates as you switch editors. With a native title bar the custom bar draws no title text because the OS draws it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.commandCenter; window.title
- Cursor: changed - when the box is off and a workspace is open, Cursor shows the workspace name as a button (see TB-2.9); with no workspace it shows the window title text or the product name
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part `createTitle` override)

### TB-2.9 Click the workspace name to open Quick Open
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: In Cursor with the command center off and a workspace open, the middle of the bar is a text button showing the workspace name (or the full title when `window.title` is customized). A left-button press, Enter or Space runs Quick Open. The button is excluded from window dragging. Its class name `agent-workspace-quickopen` hints at the agent layout, but the action is stock Quick Open, so it is not marked boundary.
- Commands: workbench.action.quickOpenWithModes
- Menus: none found
- Keys: Enter or Space while the button has focus; Enter or Space (while the button has focus)
- Settings: window.commandCenter; window.title
- Cursor: cursor only
- Source: C bundle (title bar part `createTitle`, button class `window-title-text agent-workspace-quickopen`); C bundle (title bar part `createTitle`, button class `window-title-text agent-workspace-quickopen`, mouse down and key down handlers); V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts (591-603, stock sets plain title text here)

### TB-2.10 Change the window title template
- Tag: both
- Evidence: source
- Boundary: no
- What: The `window.title` template sets the text in the OS window title (taskbar, Alt-Tab, Mission Control) and the title text in the bar. Variables: activeEditorShort, activeEditorMedium, activeEditorLong, activeEditorLanguageId, activeFolderShort, activeFolderMedium, activeFolderLong, folderName, folderPath, rootName, rootNameShort, rootPath, profileName, appName, remoteName, dirty, focusedView, activeEditorState, separator. The default on Windows and Linux is dirty, active file, root name, profile name, app name joined by separators; on macOS desktop it leaves out dirty and app name. An empty result falls back to the product name.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.title
- Cursor: present - same variable set and same default template code
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (module `vs/workbench/browser/parts/titlebar/windowTitle`, `getWindowTitle` and the default template)

### TB-2.11 Change the window title separator
- Tag: both
- Evidence: source
- Boundary: no
- What: `window.titleSeparator` sets the text used by the `${separator}` variable. The default is a hyphen with spaces, and an em dash on macOS. The separator only shows between values that are not empty.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.titleSeparator
- Cursor: present - setting registered with the same default (4 hits)
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (settings block, string 4276)

### TB-2.12 Show the active repository and branch in the window title
- Tag: both
- Evidence: source
- Boundary: no
- What: The variables `${activeRepositoryName}` and `${activeRepositoryBranchName}` fill from the active Source Control repository, so the title can show the repo and branch. The SCM contribution registers them with the title service through the same internal command extensions can call (`registerWindowTitleVariable`, extension system, not cataloged).
- Commands: registerWindowTitleVariable
- Menus: none found
- Keys: none found
- Settings: window.title
- Cursor: present - the SCM activity module registers both variables with the title service
- Source: V\src\vs\workbench\contrib\scm\browser\activity.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; C bundle (SCM activity module, `registerVariables` call)

### TB-2.13 See unsaved changes and run mode in the window title
- Tag: both
- Evidence: source
- Boundary: no
- What: The default title starts with a filled dot when the active editor has unsaved changes (not on macOS desktop, which shows its own indicator). The title is prefixed with "[Extension Development Host]" in an extension development window and suffixed with "[Administrator]" on Windows or "[Superuser]" elsewhere when running elevated. Non-space whitespace in the title is replaced with spaces.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.title
- Cursor: present - the three marker strings are in module `windowTitle` (4043-4045)
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; C bundle (module `vs/workbench/browser/parts/titlebar/windowTitle`)

### TB-2.14 Screen reader friendly window title
- Tag: both
- Evidence: source
- Boundary: no
- What: When a screen reader is detected and `accessibility.windowTitleOptimized` is on, the title gets a separator and the active editor state (for example "modified") appended, unless the template already has it. The command center then shows the full title instead of the workspace name.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.windowTitleOptimized; window.title
- Cursor: present - setting found (4 hits) and the append logic is in the same `getWindowTitle` code
- Source: V\src\vs\workbench\browser\parts\titlebar\windowTitle.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (module `windowTitle`)

### TB-2.15 Command center hides while Quick Open is at the top
- Tag: both
- Evidence: source
- Boundary: no
- What: When a quick input opens at the top of the window, the box is hidden (it keeps its space, so nothing shifts) so it does not sit behind the input. It returns when the input closes. In stock it stays visible when the quick input is centered.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the box hides on every quick input show in the active window, with no alignment check, and returns on hide
- Source: V\src\vs\workbench\browser\parts\titlebar\commandCenterControl.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (command center control class with `onShow` and `onHide` listeners)

### TB-2.16 Share button in the command center (experimental)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With `workbench.experimental.share.enabled` on, a share provider available and a folder open, a Share icon appears in the command center. It builds a link to the active file or selection, copies it, and shows it in a dialog.
- Commands: workbench.action.share
- Menus: CommandCenter (order 3)
- Keys: Alt+Ctrl+S (Alt+Cmd+S on macOS)
- Settings: workbench.experimental.share.enabled
- Cursor: changed - the command, the key and the setting (default false) exist, but the action has no `CommandCenter` menu row, so Cursor has no button
- Source: V\src\vs\workbench\contrib\share\browser\share.contribution.ts; C bundle (share action constructor with `f1` and keybinding but no `menu`, setting registration)

## Handed off
- Show or hide layout controls (right-click row `toggle.workbench.layoutControl.enabled`, group 2_config order 4, and setting `workbench.layoutControl.enabled`): TB-4.
- Show or hide the notification bell in the title bar (row `toggle.workbench.notifications.showInTitleBar`, order 5, only when `workbench.notifications.position` is top-right; Cursor has neither the row nor the bell): TB-5.
- Show or hide the Integrated Browser button (row `toggle.workbench.browser.showInTitleBar`, order 8; Cursor has neither): TB-5.
- Show or hide editor actions in the title bar (row `toggle.workbench.editor.editorActionsLocation`, order 3, setting `workbench.editor.editorActionsLocation`): ED-1.
- Menu Bar row on the right-click menu (`workbench.action.toggleMenuBar`, Windows, Linux and web only) and Focus Title Bar (`workbench.action.focusTitleBar`): TB-1.
- Hide Custom Title Bar and Hide Custom Title Bar In Full Screen rows: TB-3 (listed there as TB-3.10 and TB-3.11).
- Restore Native Title Bar row (Cursor): TB-3 (listed there as TB-3.12).
- Debug toolbar shown in the command center (`debug.toolBarLocation` set to `commandCenter`, submenu `DebugToolBar` on `CommandCenterCenter`, order 1; present in Cursor): ED-11.
- Left out as chat or agent: the Agents control on the command center (`AgentsTitleBarControlMenu`), the Agents Window Button row and its setting `workbench.agentsWindowButton.enabled`, the unified quick access branch and the compact box look driven by `chat.agentsControl.enabled`.

## Unclear
- QA-5 (opening quick access, including the command center) overlaps with TB-2.1. TB-2.1 is the title bar side only.
- I did not read whether Cursor turns the command center on when Navigation Controls are turned on, as stock does. Cursor's layout module has a list containing `workbench.navigationControl.enabled` next to `window.commandCenter`, but I did not read its handler.
- Cursor's left toolbar arrows (TB-2.4) are drawn by the title bar part next to agent buttons. TB-4 or TB-5 may also list that toolbar. I kept only the two arrows here.
- The exact `when` clauses and orders of Cursor's right-click rows were not decoded beyond the ones named above.
- Cursor was not run. Everything on the Cursor side is from strings and minified code in the bundle.
