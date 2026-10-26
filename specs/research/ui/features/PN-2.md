Group: PN-2 Terminal sessions. Section: Panel. Date: 2026-10-25. Pass: 2.
Features: 50.

Reading note: V paths below are under `V\src\vs\workbench\contrib\terminal\browser\` unless the Source line gives a longer path. Key labels are the Windows and Linux defaults, with the Mac label in brackets when it differs. The Cursor field was checked three ways: command ids and menu ids in the main bundle, the localize keys in `out\nls.keys.json` and `out\nls.messages.json` (each V key was compared with the Cursor key list for the same module), and setting ids in the bundle. Cursor's terminal code is older than V in places, which is why a few settings and strings are missing.

### PN-2.1 Show or focus the Terminal view
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+` or pick View > Terminal to open the panel on the Terminal tab (a first terminal is created if none exists yet), and run Focus Terminal to jump straight into the active terminal, creating one when there is none.
- Commands: workbench.action.terminal.toggleTerminal; workbench.action.terminal.focus
- Menus: MenubarViewMenu
- Keys: Ctrl+` on workbench.action.terminal.toggleTerminal (Mac Ctrl+`); Ctrl+Down on workbench.action.terminal.focus (only when accessibility mode is on and the accessible view for the terminal is open on its last line)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminal.contribution.ts (view and open command, lines 106-135); terminalActions.ts (Focus, lines 497-514); terminalView.ts (_initializeTerminal, lines 144-186); V\src\vs\workbench\services\views\browser\viewsService.ts (open command descriptor adds the View menu row); C bundle (ids toggleTerminal and focus; NLS key miToggleIntegratedTerminal in module terminal.view.contribution)

### PN-2.2 Create a new terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the plus button in the Terminal view, press Ctrl+Shift+` or pick Terminal > New Terminal to open a terminal with the default profile in the default location (the panel, or the editor area when the location setting says editor); Alt or Ctrl click on the plus button splits the active terminal instead, a multi-root window first asks which folder to start in (see PN-2.8), a secondary window opens the terminal in its own editor area, and two palette-only variants create one in the active workspace or a local one while in a remote window.
- Commands: workbench.action.terminal.new; workbench.action.terminal.newInActiveWorkspace; workbench.action.terminal.newLocal
- Menus: MenubarTerminalMenu; ViewTitle; TerminalEditorInstanceContext; TerminalTabEmptyAreaContext
- Keys: Ctrl+Shift+` on workbench.action.terminal.new (Mac Ctrl+Shift+`); Ctrl+Shift+C on workbench.action.terminal.new (web build only, when terminal is not focused)
- Settings: terminal.integrated.defaultLocation
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (New, lines 1213-1263; NewInActiveWorkspace, lines 297-309); V\src\vs\workbench\contrib\terminal\electron-browser\terminalRemote.ts (NewLocal); terminalMenus.ts (Terminal menu lines 41-105, plus button lines 464-483, tab empty area lines 307-330, editor instance menu lines 209-305); terminal.web.contribution.ts; terminalConfiguration.ts (defaultLocation, lines 118-127); C bundle (ids new, newInActiveWorkspace, newLocal; NLS keys miNewTerminal and workbench.action.terminal.new)

### PN-2.3 Create a terminal with a chosen profile
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Create New Terminal (With Profile), or right-click the empty area of the tabs list and pick New Terminal With Profile, to get a pick list headed "Select the terminal profile to create" with sections for configured profiles, contributed profiles and detected profiles (the default one first); Alt held on accept splits the active terminal instead, a gear button on each profile row saves it to settings under a name you type, and a profile whose path another user can edit asks for a Yes or Cancel first.
- Commands: workbench.action.terminal.newWithProfile
- Menus: TerminalTabEmptyAreaContext
- Keys: none found
- Settings: terminal.integrated.profiles.windows; terminal.integrated.profiles.osx; terminal.integrated.profiles.linux; terminal.integrated.useWslProfiles
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (NewWithProfile, lines 1530-1642); terminalProfileQuickpick.ts (lines 42-175 and 232-310); terminalService.ts (showProfileQuickPick, lines 236-275); terminalMenus.ts (lines 307-330); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (profiles, lines 228-395); C bundle (id newWithProfile; NLS keys terminal.integrated.selectProfileToCreate, terminalProfiles, terminalProfiles.detected, enterTerminalProfileName, unsafePathWarning, createQuickLaunchProfile)

### PN-2.4 Launch Profile drop-down beside the plus button
- Tag: both
- Evidence: source
- Boundary: no
- What: The small chevron next to the plus button (tooltip "Launch Profile...") opens a list of profiles to create a terminal from, with the default profile marked "(Default)", a submenu to split with a chosen profile, then Select Default Profile, Configure Terminal Settings, Run Task... and Configure Tasks...; the same drop-down sits on the editor-area toolbar for terminal editors; stock also puts New Terminal, New Terminal Window and Split Terminal at its top.
- Commands: workbench.action.terminal.newWithProfile; workbench.action.terminal.split; workbench.action.terminal.selectDefaultShell; workbench.action.terminal.openSettings; workbench.action.tasks.runTask; workbench.action.tasks.configureTaskRunner
- Menus: TerminalNewDropdownContext; ViewTitle
- Keys: none found
- Settings: terminal.integrated.profiles.windows; terminal.integrated.profiles.osx; terminal.integrated.profiles.linux; terminal.integrated.defaultProfile.windows; terminal.integrated.defaultProfile.osx; terminal.integrated.defaultProfile.linux
- Cursor: changed (no New Terminal, New Terminal Window or Split Terminal rows at the top; the split submenu is titled "Split Terminal" and is not named "Split Terminal with Profile"; no grouping of AI profiles; default profile sorted first)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (getTerminalActionBarArgs, lines 783-930; TerminalNewDropdownContext, lines 332-379); terminalView.ts (createActionViewItem, lines 250-302); terminalEditor.ts (lines 155-190); C bundle (function that builds the drop-down starts straight at the profile loop, NLS keys launchProfile, defaultTerminalProfile and splitTerminal, no key split.profile; the js-debug extension adds a "JavaScript Debug Terminal" profile at C\extensions\ms-vscode.js-debug\package.json)

### PN-2.5 Choose the default terminal profile
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Terminal: Select Default Profile (or pick it at the bottom of the Launch Profile list) to get a pick list headed "Select your default terminal profile"; the choice is written to the per-platform default profile setting (and a detected profile is also added to the profiles setting), and from then on the plus button, Ctrl+Shift+` and splits use it.
- Commands: workbench.action.terminal.selectDefaultShell
- Menus: TerminalNewDropdownContext
- Keys: none found
- Settings: terminal.integrated.defaultProfile.windows; terminal.integrated.defaultProfile.osx; terminal.integrated.defaultProfile.linux; terminal.integrated.profiles.windows; terminal.integrated.profiles.osx; terminal.integrated.profiles.linux; terminal.integrated.useWslProfiles
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (SelectDefaultProfile, lines 1371-1375); terminalProfileQuickpick.ts (setDefault branch, lines 42-80); terminalService.ts (lines 236-246); terminalMenus.ts (lines 332-343); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (defaultProfile, lines 440-480); C bundle (id selectDefaultShell; NLS key terminal.integrated.chooseDefaultProfile; the three defaultProfile settings)

### PN-2.6 Open a new terminal in its own window
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Pick Terminal > New Terminal Window or press Ctrl+Shift+Alt+` to open a terminal in a new compact window (an auxiliary window that holds only that terminal), and the same row shows in the Launch Profile list.
- Commands: workbench.action.terminal.newInNewWindow
- Menus: MenubarTerminalMenu; TerminalNewDropdownContext
- Keys: Ctrl+Shift+Alt+` on workbench.action.terminal.newInNewWindow (Mac Ctrl+Shift+Alt+`)
- Settings: none found
- Cursor: missing (searched the bundle for the id and the NLS keys newInNewWindow and miNewInNewWindow: none)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (NewInNewWindow, lines 356-374); terminalMenus.ts (menu row lines 56-67; drop-down lines 802-807); V\src\vs\workbench\contrib\terminal\common\terminalStrings.ts (newInNewWindow); C bundle (no hit for newInNewWindow)

### PN-2.7 Create a terminal in the editor area
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Create New Terminal in Editor Area (or the version "to the Side") to open a terminal as an editor tab; the plus button on a terminal editor's toolbar adds another one in the same group, the tab bar and empty-group right-click menus carry a New Terminal row, and the default-location setting makes every new terminal open this way.
- Commands: workbench.action.createTerminalEditor; workbench.action.createTerminalEditorSide; workbench.action.createTerminalEditorSameGroup
- Menus: EditorTabsBarContext; EmptyEditorGroupContext; EditorTitle; CompactWindowEditorTitle
- Keys: none found
- Settings: terminal.integrated.defaultLocation
- Cursor: changed (the three commands and the toolbar plus button exist; no New Terminal row in the tab bar or empty-group menus, and no registration on the compact window title menu, searched the bundle and found none)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 315-354); terminalMenus.ts (tab bar and empty group, lines 187-207; toolbar, lines 710-722); terminalEditor.ts (lines 155-190); terminalEditorService.ts (openEditor, lines 140-165); C bundle (the three ids; the only registration of createTerminalEditorSameGroup besides the command is the Pe.EditorTitle toolbar item; `Pe.CompactWindowEditorTitle` has 0 hits)

### PN-2.8 Starting folder for new and split terminals
- Tag: both
- Evidence: source
- Boundary: no
- What: A new terminal in a multi-root window first shows a pick list "Select current working directory for new terminal" (a folder row says "(Overridden)" when the folder sets its own terminal start path), a single folder starts there, and a split follows the split setting (workspace root with a picker when there are several roots, the parent's initial folder, or the parent's current folder); a command also starts a terminal in a folder passed as an argument.
- Commands: workbench.action.terminal.newWithCwd
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.cwd; terminal.integrated.splitCwd
- Cursor: changed (present, but the picker text reads "(Overriden)" with a typo that V has fixed)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (getCwdForSplit, lines 92-122; NewWithCwd, lines 984-1011; pickTerminalCwd, lines 1649-1692; New, lines 1240-1253); terminalService.ts (_resolveCwd, lines 1186-1200); terminalConfiguration.ts (cwd, lines 383-389; splitCwd, lines 490-500); C bundle (NLS keys workbench.action.terminal.newWorkspacePlaceholder and workbench.action.terminal.overriddenCwdDescription with the text "(Overriden) {0}"; id newWithCwd; both settings)

### PN-2.9 Workspace trust gate for new terminals
- Tag: both
- Evidence: source
- Boundary: no
- What: In an untrusted workspace, starting a terminal asks for trust with the message "Creating a terminal process requires executing code", and if trust is refused the tab shows "Cannot launch a terminal process in an untrusted workspace"; a restricted setting can bypass the check.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.allowInUntrustedWorkspace
- Cursor: changed (the two messages exist; the bypass setting `terminal.integrated.allowInUntrustedWorkspace` is not in the bundle or the key list)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (_createProcess, lines 1591-1606; _trust, lines 1948-1957); terminalConfiguration.ts (lines 674-680); C bundle (NLS keys workspaceNotTrustedCreateTerminal and terminal.requestTrust present; setting string absent)

### PN-2.10 Run selected text or the active file in a terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Terminal > Run Selected Text (or Run Active File) to send the selection, the current line or the file path to the active terminal and bring it forward, creating a terminal when none is open; a warning says "Only files on disk can be run in the terminal" for other file kinds, and the terminal and editor toolbars carry both as hidden-by-default buttons.
- Commands: workbench.action.terminal.runSelectedText; workbench.action.terminal.runActiveFile
- Menus: MenubarTerminalMenu; ViewTitle; EditorTitle
- Keys: none found
- Settings: terminal.integrated.focusAfterRun
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (RunSelectedText, lines 564-585; RunActiveFile, lines 587-612); terminalMenus.ts (menu rows lines 82-104; view toolbar lines 497-523; editor toolbar lines 736-757); terminalConfiguration.ts (focusAfterRun, lines 664-673); C bundle (ids runSelectedText and runActiveFile; NLS keys miRunSelectedText, miRunActiveFile and workbench.action.terminal.runActiveFile.noFile; setting focusAfterRun)

### PN-2.11 Name a terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Press F2 (Enter on Mac) on a tab, or pick Rename... from its right-click menu, to type a new name right in the tabs list (empty resets to the default title, Enter saves, Esc cancels), while Rename from the palette or from a terminal editor tab asks in an "Enter terminal name" box; the default title and the grey description next to it come from template settings such as process, task, working folder and separator.
- Commands: workbench.action.terminal.rename; workbench.action.terminal.renameActiveTab; workbench.action.terminal.renameWithArg
- Menus: TerminalTabContext; EditorTitleContext
- Keys: F2 on workbench.action.terminal.renameActiveTab (when terminalTabsFocus; Mac Enter)
- Settings: terminal.integrated.tabs.title; terminal.integrated.tabs.description; terminal.integrated.tabs.separator; terminal.integrated.tabs.allowAgentCliTitle
- Cursor: changed (all commands and the first three settings present; `terminal.integrated.tabs.allowAgentCliTitle` is not in the bundle or the key list)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (Rename and RenameActiveTab, lines 762-818; RenameWithArgs, lines 1013-1045; renameWithQuickPick, lines 1738-1754; validateTerminalName, lines 1504-1514); terminalTabsList.ts (inline input box, lines 438-512); terminalInstance.ts (rename, lines 2398-2420); terminalConfiguration.ts (title templates, lines 19-42; settings, lines 345-358; allowAgentCliTitle, lines 139-143); terminalMenus.ts (lines 591-600 and 674-681); C bundle (the ids; NLS keys emptyTerminalNameInfo, workbench.action.terminal.rename.prompt and terminalInputAriaLabel; the three template settings)

### PN-2.12 Change a terminal's icon or color
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Change Icon... to open a floating grid of every icon with a filter box, and Change Color... to pick one of the terminal ANSI colors (or Reset to default) in a list; with several tabs selected the same icon or color is applied to all of them, and two settings set the default icon and color for every tab.
- Commands: workbench.action.terminal.changeIcon; workbench.action.terminal.changeIconActiveTab; workbench.action.terminal.changeColor; workbench.action.terminal.changeColorActiveTab
- Menus: TerminalTabContext; EditorTitleContext
- Keys: none found
- Settings: terminal.integrated.tabs.defaultIcon; terminal.integrated.tabs.defaultColor
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 710-760); terminalInstance.ts (changeIcon, lines 2422-2437; changeColor, lines 2439-2488); terminalIconPicker.ts; terminalIcon.ts; terminalMenus.ts (lines 601-620 and 683-700); terminalConfiguration.ts (lines 52-61); C bundle (the four ids; NLS keys workbench.action.terminal.changeIcon and workbench.action.terminal.changeColor; both settings; "Reset to default" string)

### PN-2.13 Status icons on a terminal tab
- Tag: both
- Evidence: source
- Boundary: no
- What: A small icon with a color shows on the tab (as the icon when the list is narrow, as a badge letter when it is wide) for a bell, a lost connection to the shell ("Lost connection to process"), extensions that want the terminal relaunched (with a Relaunch Terminal action), extensions that changed its environment (with Show Environment Contributions), the shell integration level (Rich, Basic, No, Injection failed to activate) and a shell integration launch failure (with Learn more about shell integration and Open user settings); animated icons can be turned off.
- Commands: workbench.action.terminal.relaunch; workbench.action.terminal.showEnvironmentContributions; workbench.action.terminal.learnMore
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.tabs.enableAnimation; terminal.integrated.enableVisualBell; terminal.integrated.environmentChangesRelaunch
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalStatusList.ts; terminalInstance.ts (statuses at lines 862, 988, 1575 and 1788); environmentVariableInfo.ts (lines 28-95); terminalTabsList.ts (TabDecorationsProvider, lines 810-843; renderElement, lines 346-436); terminalConfiguration.ts (lines 66-71, 413-421 and 474-478); C bundle (status ids bell, disconnected, relaunch-needed, env-var-info-changes-active, shell-integration-info and shell-integration-attention-needed; NLS keys bellStatus, disconnectStatus, shellIntegration.learnMore, shellIntegration.openSettings, relaunchTerminalLabel; the shell integration level strings live in module terminalTooltip in Cursor and in terminalInstance in V)

### PN-2.14 Hover card on a terminal tab
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering a tab (or the single tab in the view title) shows a card with the title, the process ID, the command line, the text of every status, and action links from those statuses, ending in a Show Details or Hide Details link that adds longer text and is remembered; Ctrl+K Ctrl+I puts focus into the card.
- Commands: workbench.action.terminal.focusHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I on workbench.action.terminal.focusHover (when terminalTabsFocus or terminalFocus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTooltip.ts (lines 1-76); terminalActions.ts (FocusHover, lines 1342-1352); terminalTabsList.ts (hoverInfo, lines 370-373); terminalTabbedView.ts (focusHover, lines 604-616); V\src\vs\workbench\contrib\terminal\common\terminalStorageKeys.ts (TabsShowDetailed); C bundle (id focusHover; NLS keys showDetails, hideDetails, shellProcessTooltip.processId, shellProcessTooltip.commandLine; storage key terminal.integrated.tabs.showDetailed)

### PN-2.15 Side tabs list and when it hides
- Tag: both
- Evidence: source
- Boundary: no
- What: Beside the terminal, a list shows one row per terminal (icon, title, description, a split mark of ┌ ├ └ for panes in one group, and inline Split and Kill buttons when the list is wide), and by default it appears only once there is more than one terminal and stays open while the mouse is over it; the empty area of the list has a right-click menu with New Terminal With Profile, New Terminal and Hide Tabs, and turning tabs off replaces the list with a drop-down of numbered groups plus a Show Tabs entry.
- Commands: workbench.action.terminal.switchTerminal; workbench.action.terminal.splitActiveTab; workbench.action.terminal.killActiveTab
- Menus: TerminalTabEmptyAreaContext; ViewTitle
- Keys: none found
- Settings: terminal.integrated.tabs.enabled; terminal.integrated.tabs.hideCondition
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabbedView.ts (shouldShowTabs, lines 167-210; empty area menu, lines 432-455; getTabActions, lines 550-562); terminalTabsList.ts (renderElement, lines 346-436; fillActionBar, lines 525-548); terminalView.ts (SwitchTerminalActionViewItem, lines 359-402); terminalActions.ts (SwitchTerminal, lines 1403-1445; switchTerminalShowTabsTitle, line 68); terminalGroupService.ts (getGroupLabels, lines 500-504); terminalConfiguration.ts (lines 63-83); C bundle (ids switchTerminal, splitActiveTab, killActiveTab; NLS keys showTerminalTabs, hideTabs, terminals, terminal.tabs; both settings)

### PN-2.16 Place and size the tabs list
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click the empty area of the tabs list and pick Move Tabs Left or Move Tabs Right to put it on the other side of the terminal, and drag the divider to resize it, where the list snaps to a narrow icon-only strip (46 px) or a wide text view (at least 80 px, at most 500 px) and a double click on the divider fits the width to the longest title or collapses it again; the width is remembered separately for a bottom or top panel and a left or right panel.
- Commands: none found
- Menus: TerminalTabEmptyAreaContext
- Keys: none found
- Settings: terminal.integrated.tabs.location
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabbedView.ts (lines 118-135, 217-338 and 550-562); terminalTabsList.ts (TerminalTabsListSizes, lines 62-70); V\src\vs\workbench\contrib\terminal\common\terminalStorageKeys.ts (TabsListWidthHorizontal, TabsListWidthVertical); terminalConfiguration.ts (lines 108-117); C bundle (storage keys tabs-list-width-vertical and tabs-list-width-horizontal; NLS keys moveTabsLeft and moveTabsRight; setting tabs.location)

### PN-2.17 Compact toolbar when the tabs list is hidden
- Tag: both
- Evidence: source
- Boundary: no
- What: With the tabs list hidden or in its narrow strip, the view title shows the active terminal as one button (icon, title and description in its color; click to focus, drag to move it, hover for the card) and, next to the plus button, Split and Kill buttons, with settings choosing when each shows (always, single terminal, single terminal or narrow, never); three more buttons (Clear Terminal, Run Active File, Run Selected Text) can be switched on from the toolbar menu.
- Commands: workbench.action.terminal.focus; workbench.action.terminal.split; workbench.action.terminal.kill; workbench.action.terminal.clear; workbench.action.terminal.switchTerminal
- Menus: ViewTitle
- Keys: none found
- Settings: terminal.integrated.tabs.showActiveTerminal; terminal.integrated.tabs.showActions; terminal.integrated.tabs.enabled; terminal.integrated.tabs.hideCondition
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalMenus.ts (ViewTitle, lines 381-551); terminalView.ts (SingleTerminalTabActionViewItem, lines 405-565); V\src\vs\workbench\contrib\terminal\common\terminalContextKey.ts (shouldShowViewInlineActions, lines 147-170); terminalConfiguration.ts (lines 84-107); C bundle (ids focus, split, kill, clear; the two settings; the ViewTitle menu code that tests config.terminal.integrated.tabs.showActiveTerminal and terminalGroupCount; isHiddenByDefault on the toolbar items)

### PN-2.18 Select and focus terminals in the tabs list
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking a row selects it and, by default, a double click moves focus into that terminal (a setting makes a single click do it), Ctrl or Shift click selects several for one menu action, Alt click splits that terminal, a middle click kills it, a double click on empty space creates a terminal, and a keyboard shortcut jumps focus into the list.
- Commands: workbench.action.terminal.focusTabs
- Menus: TerminalTabContext
- Keys: Ctrl+Shift+\ on workbench.action.terminal.focusTabs (when terminalTabsFocus or terminalFocus)
- Settings: terminal.integrated.tabs.focusMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabsList.ts (mouse handlers, lines 158-221; middle click, lines 395-403); terminalTabbedView.ts (focusTabs, lines 573-582); terminalActions.ts (FocusTabs, lines 516-526; getSelectedViewInstances, lines 1471-1503); terminalConfiguration.ts (lines 129-138); C bundle (id focusTabs; NLS key focus.tabsView; setting tabs.focusMode)

### PN-2.19 Reorder tabs and drop things onto them
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag a row to reorder terminals and groups (dropping on another row moves the group there, dropping on empty space sends it to the end), hold a drag over a row for half a second to switch to it, drop a file or folder from the Explorer or the desktop onto a row to paste its path into that terminal, and drop a terminal editor tab or a terminal from another window onto the list to bring it into the panel.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalTabsList.ts (drag and drop, lines 640-806); terminalTabbedView.ts (empty area drop, lines 370-410 and 477-547); V\src\vs\workbench\contrib\terminal\browser\terminal.contribution.ts (drag and drop contribution, lines 83-102); terminalUri.ts; C bundle (code for sendPath, moveGroupToEnd and requestDetachInstance present)

### PN-2.20 Switch between terminal groups
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+PageDown or Ctrl+PageUp (Cmd+Shift+] or Cmd+Shift+[ on Mac) in a terminal to make the next or previous group active and bring the panel forward, and nine unbound commands jump to the first through ninth group so you can assign your own keys.
- Commands: workbench.action.terminal.focusNext; workbench.action.terminal.focusPrevious; workbench.action.terminal.focusAtIndex1; workbench.action.terminal.focusAtIndex2; workbench.action.terminal.focusAtIndex3; workbench.action.terminal.focusAtIndex4; workbench.action.terminal.focusAtIndex5; workbench.action.terminal.focusAtIndex6; workbench.action.terminal.focusAtIndex7; workbench.action.terminal.focusAtIndex8; workbench.action.terminal.focusAtIndex9
- Menus: none found
- Keys: Ctrl+PageDown on workbench.action.terminal.focusNext and Ctrl+PageUp on workbench.action.terminal.focusPrevious (when terminalFocus and not terminalEditorFocus; Mac Cmd+Shift+] and Cmd+Shift+[)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (FocusNext and FocusPrevious, lines 528-562); terminalCommands.ts (lines 1-28); C bundle (ids focusNext, focusPrevious and focusAtIndex1 to focusAtIndex9; the focusAtIndex registration loop is present)

### PN-2.21 Split a terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+5 in a terminal, click the Split button, pick Terminal > Split Terminal, use Split on a tab's right-click menu, or Alt click the plus button to add a second terminal next to the active one inside the same group (side by side when the panel is at the bottom or top, stacked when it is on a side); in the editor area a split opens another terminal editor to the side.
- Commands: workbench.action.terminal.split; workbench.action.terminal.splitActiveTab; workbench.action.terminal.splitInActiveWorkspace
- Menus: MenubarTerminalMenu; ViewTitle; TerminalTabContext; TerminalEditorInstanceContext; EditorTitle
- Keys: Ctrl+Shift+5 on workbench.action.terminal.split (when terminalFocus; Mac Cmd+\ with Ctrl+Shift+5 as the second key); Ctrl+Shift+5 on workbench.action.terminal.splitActiveTab (when terminalTabsFocus; Mac Cmd+\)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (Split, lines 1053-1083; SplitActiveTab, lines 1085-1111; SplitInActiveWorkspace, lines 1184-1192); terminalMenus.ts (lines 68-80, 440-450, 560-568, 210-219 and 715-722); terminalGroup.ts (orientation and split, lines 483-552); terminalService.ts (_splitTerminal, lines 1203-1229); terminalEditorService.ts (splitInstance, lines 215-245); terminalView.ts (panel-only split, lines 252-272); C bundle (ids split, splitActiveTab, splitInActiveWorkspace; NLS keys miSplitTerminal and splitTerminal)

### PN-2.22 Move focus between split panes
- Tag: both
- Evidence: source
- Boundary: no
- What: With a split group active, Alt plus the Left or Up arrow moves focus to the previous terminal in the group and Alt plus the Right or Down arrow to the next (Alt+Cmd plus arrows on Mac), and the panel is shown if it was hidden.
- Commands: workbench.action.terminal.focusPreviousPane; workbench.action.terminal.focusNextPane
- Menus: none found
- Keys: Alt+Left and Alt+Up on workbench.action.terminal.focusPreviousPane; Alt+Right and Alt+Down on workbench.action.terminal.focusNextPane (when terminalFocus and a split terminal is active; Mac Alt+Cmd with the arrow)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 405-445); C bundle (ids focusPreviousPane and focusNextPane; NLS keys workbench.action.terminal.focusPreviousPane and workbench.action.terminal.focusNextPane)

### PN-2.23 Resize split panes
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the divider between panes, or press the resize shortcuts with a terminal focused, to make the active pane larger toward the chosen side (Linux has Ctrl+Shift with Left or Right, the Mac has Ctrl+Cmd with all four arrows, Windows has no default key).
- Commands: workbench.action.terminal.resizePaneLeft; workbench.action.terminal.resizePaneRight; workbench.action.terminal.resizePaneUp; workbench.action.terminal.resizePaneDown
- Menus: none found
- Keys: Ctrl+Shift+Left on workbench.action.terminal.resizePaneLeft and Ctrl+Shift+Right on workbench.action.terminal.resizePaneRight (Linux only, when terminalFocus); Ctrl+Cmd+Left, Right, Up and Down on the four resize commands (Mac only, when terminalFocus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 447-495); terminalGroup.ts (SplitPaneContainer.resizePane, lines 66-110; resizePane, lines 585-632); C bundle (the four ids; NLS keys workbench.action.terminal.resizePaneLeft to resizePaneDown)

### PN-2.24 Unsplit a pane or join terminals into a group
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a pane's tab and pick Unsplit Terminal to move it into a group of its own, or select two or more tabs and pick Join Terminals (or run Join Terminals... from the palette and choose one) to merge separate groups into one split group, with messages when there are too few terminals or all are already joined.
- Commands: workbench.action.terminal.unsplit; workbench.action.terminal.join; workbench.action.terminal.joinActiveTab
- Menus: TerminalTabContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (Unsplit, lines 1113-1123; JoinActiveTab, lines 1125-1135; Join, lines 1137-1182); terminalMenus.ts (lines 632-651); terminalGroupService.ts (unsplitInstance, joinInstances); C bundle (ids unsplit, join, joinActiveTab; NLS keys unsplitTerminal, workbench.action.terminal.joinInstance, workbench.action.terminal.join.insufficientTerminals, workbench.action.terminal.join.onlySplits)

### PN-2.25 Move a terminal into the editor area or a new window
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Move Terminal into Editor Area on a tab's right-click menu (or from the palette) to turn the selected terminals into editor tabs, pick Move Terminal into New Window to put them in a window of their own, or drag a tab (or the single tab in the view title) into the editor area to do the same.
- Commands: workbench.action.terminal.moveToEditor; workbench.action.terminal.moveIntoNewWindow
- Menus: TerminalTabContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 376-391); terminalMenus.ts (lines 568-590); terminalService.ts (moveToEditor and moveIntoNewEditor, lines 790-815); terminal.contribution.ts (drag and drop data, lines 83-102); V\src\vs\workbench\contrib\terminal\common\terminalStrings.ts; C bundle (ids moveToEditor and moveIntoNewWindow; NLS keys moveToEditor and moveIntoNewWindow)

### PN-2.26 Move a terminal back into the panel
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a terminal editor tab and pick Move Terminal into Panel (or run it from the palette) to return the terminal to the Terminal view, or drag the editor tab onto the tabs list.
- Commands: workbench.action.terminal.moveToTerminalPanel
- Menus: EditorTitleContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 393-403); terminalMenus.ts (lines 665-673); terminalService.ts (moveToTerminalView, lines 817-850); C bundle (id moveToTerminalPanel; NLS key workbench.action.terminal.moveToTerminalPanel; the EditorTitleContext registration with the vscode-terminal scheme check)

### PN-2.27 Terminal as an editor tab
- Tag: both
- Evidence: source
- Boundary: no
- What: A terminal in the editor area is an editor tab with the terminal's icon, color, status decorations, title and description, a toolbar with plus (Alt click splits to the side) and the profile drop-down, and a right-click menu with Split, New Terminal, Kill Terminal, Copy, Copy as HTML, Paste, Clear, Select All and Toggle Size to Content Width; Ctrl+W closes it, and terminal editors are saved with the window layout and come back after a reload.
- Commands: workbench.action.terminal.killEditor
- Menus: TerminalEditorInstanceContext; EditorTitle; EditorTitleContext
- Keys: Ctrl+W on workbench.action.terminal.killEditor (when terminalFocus and terminalEditorFocus; on Windows Ctrl+F4 with Ctrl+W as the second key)
- Settings: none found
- Cursor: changed (present, but the toolbar buttons are not registered on the compact window title menu)
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalEditor.ts; terminalEditorInput.ts (lines 30-170); terminalEditorService.ts; terminalEditorSerializer.ts; terminalActions.ts (KillEditor, lines 1305-1316); terminalMenus.ts (lines 209-305, 665-709 and 710-781); terminal.contribution.ts (editor pane and serializer, lines 71-82); C bundle (id killEditor; editor type id workbench.editors.terminal; menu id Pe.TerminalEditorInstanceContext; no Pe.CompactWindowEditorTitle)

### PN-2.28 Kill terminals
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the trash button in the view title or on a tab row, press Delete on a focused tab, middle-click a tab, or pick Kill Terminal from a terminal's or tab's right-click menu to end a terminal and its shell (all selected tabs go together), and run Kill All Terminals from the palette to end every one.
- Commands: workbench.action.terminal.kill; workbench.action.terminal.killViewOrEditor; workbench.action.terminal.killActiveTab; workbench.action.terminal.killAll
- Menus: ViewTitle; TerminalInstanceContext; TerminalTabContext
- Keys: Delete on workbench.action.terminal.killActiveTab (when terminalTabsFocus; Mac Cmd+Backspace or Delete)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 1265-1340); terminalMenus.ts (lines 110-119, 451-463 and 652-662); terminalTabsList.ts (inline Kill, lines 541-544; middle click, lines 395-403); terminalService.ts (safeDisposeTerminal, lines 430-444); C bundle (ids kill, killViewOrEditor, killActiveTab, killAll; NLS keys workbench.action.terminal.kill, workbench.action.terminal.killAll, killTerminal and killTerminal.short)

### PN-2.29 Confirm before killing a busy terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: When a terminal has a running child process, killing a panel terminal asks "Do you want to terminate the active terminal session?" with a Terminate button, while a terminal editor tab is marked as changed and asks "Do you want to terminate running processes?" when closed; the setting chooses where it asks (never, editor, panel, always; editor by default) and a second setting lists process names that do not count as running work.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.confirmOnKill; terminal.integrated.ignoreProcessNames
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (safeDisposeTerminal, lines 430-444; _showTerminalCloseConfirmation, lines 935-948); terminalEditorInput.ts (showConfirm and confirm, lines 96-118); terminalConfiguration.ts (confirmOnKill, lines 401-411); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (ignoreProcessNames, lines 413-428); C bundle (NLS keys terminalService.terminalCloseConfirmationSingular, confirmDirtyTerminal.message and confirmDirtyTerminal.detail; both settings)

### PN-2.30 Confirm before closing the window with terminals open
- Tag: both
- Evidence: source
- Boundary: no
- What: With the setting on, closing the window or quitting while terminals are running asks "Do you want to terminate the N active terminal sessions?" (or the single-session wording) with a Terminate button, either always or only when a terminal has child processes, and it is skipped when the sessions will be kept for a reload; off by default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.confirmOnExit
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (shutdown handling, lines 630-668; _showTerminalCloseConfirmation, lines 935-948); terminalConfiguration.ts (confirmOnExit, lines 390-400); C bundle (NLS keys terminalService.terminalCloseConfirmationPlural and terminate; setting confirmOnExit)

### PN-2.31 Hide the Terminal view when it is empty
- Tag: both
- Evidence: source
- Boundary: no
- What: Closing the last terminal hides the Terminal view when it is the only view in its panel tab (a setting can keep it open), and at startup the view can be hidden always, hidden only when no sessions were restored, or never hidden, in which case a first terminal is created.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.hideOnLastClosed; terminal.integrated.hideOnStartup
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (lines 205-215); terminalView.ts (_initializeTerminal, lines 144-186); terminalConfiguration.ts (lines 559-574); C bundle (both settings)

### PN-2.32 Process exit and launch failure notices
- Tag: both
- Evidence: source
- Boundary: no
- What: When a shell exits with an error, an error notification says "The terminal process ... terminated with exit code: N" (skipped after Ctrl+D and when the alert setting is off), a shell that cannot start says "The terminal process failed to launch" with an Open Help action, a terminal set to wait on exit keeps its tab until you press a key, and Relaunch Active Terminal restarts the shell in the same tab.
- Commands: workbench.action.terminal.launchHelp; workbench.action.terminal.relaunch
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.showExitAlert
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalInstance.ts (exit handling, lines 1735-1783; relaunch, lines 1913-1925; messages, lines 2845-2890); terminalActions.ts (TerminalLaunchHelpAction, lines 124-135; Relaunch, lines 1047-1051); terminalConfiguration.ts (lines 479-483); C bundle (ids launchHelp and relaunch; NLS keys terminalLaunchHelp, launchFailed.exitCodeOnly, terminated.exitCodeOnly and workbench.action.terminal.relaunch; setting showExitAlert)

### PN-2.33 Restore terminals after a reload
- Tag: both
- Evidence: source
- Boundary: no
- What: After a window reload the tabs, groups, split layout, titles, icons, colors and a number of scrollback lines come back and the shells reattach to their running processes, after quitting or closing the window a setting decides whether content and processes are recreated on next open, a lost shell connection restarts the terminal with "Restarting the terminal because the connection to the shell process was lost...", and background tasks with problem matchers can reconnect too.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.enablePersistentSessions; terminal.integrated.persistentSessionReviveProcess; terminal.integrated.persistentSessionScrollback; task.reconnection
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalService.ts (backend start, lines 277-330; reconnect and recreate groups, lines 452-560; shutdown and revive, lines 620-700; saved title and icon, lines 738-760); terminalProcessManager.ts (line 570); terminalEditorSerializer.ts; terminalMainContribution.ts; terminalConfiguration.ts (lines 542-558); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (lines 401-406); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (task.reconnection, line 551); C bundle (the three terminal settings and task.reconnection; NLS key ptyHostRelaunch; restoredGroupCount code)

### PN-2.34 Detach and attach terminal sessions
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Detach Session to close the active terminal's tab while its shell keeps running in the background terminal host, and Attach to Session to pick one of those unattached shells (shown with its title, folder and process ID) and open it in a new tab, with a message when there is none to attach.
- Commands: workbench.action.terminal.detachSession; workbench.action.terminal.attachToSession
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 820-869); terminalInstance.ts (detachProcessAndDispose, line 1353); terminalService.ts (isAttachedToTerminal); C bundle (ids detachSession and attachToSession; NLS keys workbench.action.terminal.detachSession, workbench.action.terminal.attachToSession and noUnattachedTerminals; code isAttachedToTerminal present)

### PN-2.35 Task state on a terminal tab
- Tag: both
- Evidence: source
- Boundary: no
- What: The tab of a task's terminal shows the task state as an icon (a spinner while it runs, a check on success, and error, warning or info icons when the problem matcher finds problems, with "and waiting..." text for idle background tasks), with hover text such as "Task is running" or "Task has errors", and a marker line notes where the detected errors for the run begin.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.tabs.enableAnimation
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\taskTerminalStatus.ts (lines 29-66); terminalTaskSystem.ts (status manager, line 238); terminalStatusList.ts; terminalConfiguration.ts (lines 68-71); C bundle (all ten NLS keys of module taskTerminalStatus, among them taskTerminalStatus.active, taskTerminalStatus.errors and task.watchFirstError)

### PN-2.36 Task terminal reuse
- Tag: both
- Evidence: source
- Boundary: no
- What: Running a task reuses an idle terminal according to the task's presentation options in tasks.json (panel shared, dedicated or new, plus group, clear, close and showReuseMessage), and a finished task terminal stays open with "Terminal will be reused by tasks, press any key to close it." or "Press any key to close the terminal." instead of vanishing; a reused terminal scrolls to the bottom and is cleared when the task asks for it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (reuse, lines 1554-1606; wait on exit text, lines 2008-2025); V\src\vs\workbench\contrib\tasks\common\tasks.ts (PanelKind, presentation options); C bundle (NLS keys reuseTerminal and closeTerminal in module terminalTaskSystem; the strings showReuseMessage and "dedicated" in the task schema)

### PN-2.37 Rerun a task from its terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: In a terminal that belongs to a task, pick Rerun Task from its right-click menu, or press Ctrl+Shift+R, to run that task again.
- Commands: workbench.action.tasks.rerunForActiveTerminal
- Menus: TerminalInstanceContext
- Keys: Ctrl+Shift+R on workbench.action.tasks.rerunForActiveTerminal (when terminalFocus; Mac Ctrl+Shift+R)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (lines 584-603); V\src\vs\workbench\contrib\tasks\common\tasks.ts (RerunForActiveTerminalCommandId, line 1441); C bundle (id rerunForActiveTerminal; NLS key rerunTask in module terminalTaskSystem; context key for an active task terminal)

### PN-2.38 Shell integration for new terminals
- Tag: both
- Evidence: source
- Boundary: no
- What: By default each new terminal gets a startup script injected for bash, zsh, fish and PowerShell (and Git Bash on Windows) so the terminal learns about commands and folders; the setting only affects terminals created afterwards, a wait-time setting decides how long to wait before calling it absent, and a shell that fails to start with the script is relaunched without it with a warning on the tab and links to learn more or open the setting.
- Commands: workbench.action.terminal.learnMore
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.shellIntegration.enabled; terminal.integrated.shellIntegration.timeout
- Cursor: changed (the enabled setting is present; the timeout setting is not in the bundle or the key list)
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 610-636); terminalInstance.ts (level text, lines 950-991; relaunch without shell integration, lines 1783-1806); terminalTooltip.ts (lines 44-48); C bundle (id learnMore; NLS keys shellIntegration.learnMore and shellIntegration.openSettings; setting enabled; timeout absent)

### PN-2.39 Scrollback size
- Tag: both
- Evidence: source
- Boundary: no
- What: One setting sets how many lines each terminal keeps in its buffer (1000 by default, more lines use more memory) and a second sets how many of those lines come back when a session is restored (100 by default).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.scrollback; terminal.integrated.persistentSessionScrollback
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (lines 318-321); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (lines 401-406); C bundle (both settings; NLS key terminal.integrated.persistentSessionScrollback)

### PN-2.40 Set the terminal width and height
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Set Fixed Dimensions to type a column count and then a row count (empty means automatic) that give the terminal a fixed size with scrollbars, or press Alt+Z (Toggle Size to Content Width) to widen the terminal to its longest wrapped line and press it again to go back to fitting the panel.
- Commands: workbench.action.terminal.setDimensions; workbench.action.terminal.sizeToContentWidth
- Menus: TerminalTabContext; TerminalInstanceContext; TerminalEditorInstanceContext; EditorTitleContext
- Keys: Alt+Z on workbench.action.terminal.sizeToContentWidth (when terminalFocus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\terminalActions.ts (lines 1384-1401); terminalInstance.ts (setFixedDimensions, lines 2173-2209; toggleSizeToContentWidth, lines 2211-2236); terminalMenus.ts (lines 164-172, 292-301, 622-630 and 702-708); C bundle (ids setDimensions and sizeToContentWidth; NLS keys workbench.action.terminal.setFixedDimensions and workbench.action.terminal.sizeToContentWidthInstance; code setFixedDimensions and toggleSizeToContentWidth present)

### PN-2.41 Open an external terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+C while no terminal has focus, or run Open New External Terminal from the Command Palette, to start the system terminal program (not the built-in one) in the workspace folder; with several folders open a list titled "Select workspace folder" asks which one, a remote window uses the folder's local path when it has one, and otherwise the folder of the last active file or the home folder is used; three per-platform settings choose the program, and debug launch requests for an external console use the same program setting. This is the desktop use of Ctrl+Shift+C, so PN-2.2 lists the same key for New Terminal in the web build only.
- Commands: workbench.action.terminal.openNativeConsole
- Menus: CommandPalette
- Keys: Ctrl+Shift+C on workbench.action.terminal.openNativeConsole (when terminal is not focused; Mac Cmd+Shift+C)
- Settings: terminal.external.windowsExec; terminal.external.osxExec; terminal.external.linuxExec
- Cursor: present (key rule in Cursor has only the not-terminal-focused condition, V also excludes the sessions window)
- Source: V\src\vs\workbench\contrib\externalTerminal\electron-browser\externalTerminal.contribution.ts (command and key, lines 29-101; palette entry, lines 104-108; settings, lines 137-165); V\src\vs\workbench\api\node\extHostDebugService.ts (external branch, lines 209-232); C bundle (id openNativeConsole with primary key CtrlCmd+Shift+C and the notFocus condition; NLS keys globalConsoleAction and selectWorkspace; the three terminal.external settings)

### PN-2.42 Environment variables for new terminals
- Tag: both
- Evidence: source
- Boundary: no
- What: Edit one settings object per platform to add environment variables to every new terminal (a null value deletes a variable), choose whether new shells inherit the editor's own environment (on macOS and Linux this may source a login shell so PATH is set up; no effect on Windows), and choose whether the LANG variable is forced to a UTF-8 value (auto, off or on); a terminal profile can also carry its own env object.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.env.windows; terminal.integrated.env.osx; terminal.integrated.env.linux; terminal.integrated.inheritEnv; terminal.integrated.detectLocale
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (detectLocale, lines 323-333; env settings, lines 447-473); V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (inheritEnv, lines 395-400; profile env property, lines 52-59); V\src\vs\workbench\contrib\terminal\browser\terminalProfileResolverService.ts (line 152); V\src\vs\workbench\contrib\terminal\electron-browser\localTerminalBackend.ts (line 375); V\src\vs\workbench\contrib\terminal\browser\remoteTerminalBackend.ts (line 177); C bundle (the five setting ids; NLS messages for env.osx, inheritEnv and detectLocale)

### PN-2.43 Terminal profile for tasks and debug
- Tag: both
- Evidence: source
- Boundary: no
- What: Set a profile object (shell path, arguments, icon, color and so on) per platform that tasks and debug sessions use for their terminals instead of the default profile; the setting is empty by default (so the default profile is used), it is a restricted setting that an untrusted workspace ignores, and on Windows it is ignored while the old automation shell setting is set.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.automationProfile.windows; terminal.integrated.automationProfile.osx; terminal.integrated.automationProfile.linux
- Cursor: present
- Source: V\src\vs\platform\terminal\common\terminalPlatformConfiguration.ts (schema, lines 81-94; the three settings, lines 117-170); V\src\vs\workbench\contrib\terminal\browser\terminalProfileService.ts (lines 158-165); V\src\vs\workbench\contrib\terminal\browser\terminalProfileResolverService.ts (allowAutomationShell, lines 205-250); V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (line 1227); V\src\vs\workbench\api\browser\mainThreadTerminalService.ts (lines 128-129); C bundle (the three setting ids with restricted true; NLS key terminal.integrated.automationProfile.linux; allowAutomationShell in the task system and in the terminal service bridge)

### PN-2.44 Debug session in an integrated terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: When a launch configuration asks for the integrated terminal, the debug session's program runs in a terminal tab with the debug icon, named after the session title (or "Debug Process"), started with the automation profile and with shell integration forced on; the panel shows the tab without taking focus, a finished session's tab is reused by the next session with the same shell, folder and title once its program has ended, and a setting clears the screen before the reuse.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.terminal.clearBeforeReusing; terminal.integrated.automationProfile.windows; terminal.integrated.automationProfile.osx; terminal.integrated.automationProfile.linux
- Cursor: present (the ext host bundle has the clear-before-reuse code and the forced shell integration option)
- Source: V\src\vs\workbench\api\node\extHostDebugService.ts (integrated branch, lines 79-207; DebugTerminalCollection, lines 234-289); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (clearBeforeReusing, lines 558-562); C bundle (setting debug.terminal.clearBeforeReusing in the main bundle; NLS key debug.terminal.title in module extHostDebugService; the reuse code in C\out\vs\workbench\api\node\extensionHostProcess.js)

### PN-2.45 Task terminal visibility and echo
- Tag: both
- Evidence: source
- Boundary: no
- What: The presentation options in a task's tasks.json entry decide how its terminal shows up: reveal (always brings the terminal forward, silent only when the task exits non-zero or its problem matcher finds an error, never keeps it in the background), focus (the terminal also takes keyboard focus), revealProblems (always, onProblem or never opens the Problems view, and a choice of always opens Problems instead of the terminal) and echo (whether the "Executing task: ..." line is printed first); defaults are echo on, reveal always, focus off, revealProblems never.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (background tasks, lines 904-916 and 985-996; other tasks, lines 1106-1118; immediate reveal and focus, lines 1155-1163; echo, lines 1330-1385 and 1536-1539); V\src\vs\workbench\contrib\tasks\common\tasks.ts (RevealKind, lines 130-163; RevealProblemKind, lines 165-196; defaults, lines 289-292); V\src\vs\workbench\contrib\tasks\common\jsonSchema_v2.ts (presentation schema); C bundle (schema keys echo, focus and revealProblems with NLS keys JsonSchema.tasks.presentation.echo and JsonSchema.tasks.presentation.revealProblems.onProblem; the defaults object with echo true, reveal 1, revealProblems 1, focus false; the Executing task string)

### PN-2.46 Task terminal name, icon and color
- Tag: both
- Evidence: source
- Boundary: no
- What: A task's terminal tab is named after the task (with the folder name in brackets in a multi-root workspace) and shows the icon and color the task sets in tasks.json (icon.id is a codicon id and icon.color is one of the terminal ANSI color ids); a task started as a dependency inherits any icon id or color it does not set from the task that asked for it, and each task terminal also gets a Rerun Task button on its tab row (see PN-2.37). V also lists a presentation option preserveTerminalName in the schema, but I found no code that reads it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (name, icon and color are present; the schema key preserveTerminalName is not in the bundle)
- Source: V\src\vs\workbench\contrib\tasks\browser\terminalTaskSystem.ts (_createTerminalName, lines 1195-1198; icon and color, lines 1231-1247, 1358-1361 and 1541-1542; dependency inheritance, lines 653-660; tab actions, lines 177 and 1407); V\src\vs\workbench\contrib\terminal\browser\terminalTabsList.ts (tab action buttons, lines 534-540); V\src\vs\workbench\contrib\tasks\common\tasks.ts (getQualifiedLabel, lines 701-708); V\src\vs\workbench\contrib\tasks\common\jsonSchema_v2.ts (icon schema, lines 115-145; preserveTerminalName, lines 211-215); C bundle (NLS key JsonSchema.tasks.icon; the code that reads configurationProperties.icon id and color)

### PN-2.47 Windows terminal backend options
- Tag: both
- Evidence: source
- Boundary: no
- What: Two Windows-only settings pick how the shell is hosted: one uses the conpty.dll shipped with the editor instead of the one inside Windows, and (Cursor only) one chooses ConPTY or the older Winpty; when ConPTY cannot start, the launch failure message tells you to try the DLL setting. In V 1.128.0 Winpty is gone, so only the DLL setting exists and it is on by default; in Cursor the DLL setting is off by default and tagged preview. The newer 1.144.0 tree adds terminal.integrated.windowsReparentConsoleWindow, which is not part of this feature in 1.128.0.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: terminal.integrated.windowsUseConptyDll; terminal.integrated.windowsEnableConpty
- Cursor: changed (windowsEnableConpty exists only in Cursor, default true; windowsUseConptyDll defaults to false and is tagged preview, V defaults to true)
- Source: V\src\vs\workbench\contrib\terminal\common\terminalConfiguration.ts (windowsUseConptyDll, lines 484-489); V\src\vs\platform\terminal\node\terminalProcess.ts (lines 159-168 and 250-257); C bundle (setting windowsUseConptyDll with tags preview and default false, NLS text "experimental conpty.dll (v1.22.250204002)"; setting windowsEnableConpty with NLS text "Winpty will be used if this is false"); N (windowsReparentConsoleWindow, terminalConfiguration.ts line 501, newer-than-1.128)

### PN-2.48 Return to the terminal after the editor's wait flag
- Tag: both
- Evidence: source
- Boundary: no
- What: Running the editor's command line tool with its wait flag from a terminal inside the editor (for example as the commit message editor for Git) keeps that command waiting until you close the opened file, and then moves keyboard focus back to the active terminal; it only applies to requests that come from the editor's own terminal (TERM_PROGRAM is vscode).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\electron-browser\terminalNativeContribution.ts (lines 29 and 48-75); C bundle (class with _onOpenFileRequest that tests termProgram equal to "vscode" and filesToWait, then focuses the active terminal)

### PN-2.49 Terminal tip at the first prompt
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the tip advertises the Cursor agent command line tool, and its text and action come from a remote experiment named terminal_tip.
- What: At the first prompt of an ordinary new terminal (not an extension terminal or a task or other feature terminal), a faint italic tip can appear after the cursor when the cursor is in the first 30 columns; it is only offered to a person whose machine has the Claude or Codex command line tool but not the Cursor one, the experiment parameters include a show count and an hours interval between shows, any key press removes it, and when the tip has an action then Shift+Tab as the very first key types that action into the prompt without running it.
- Commands: none found
- Menus: none found
- Keys: Shift+Tab (only as the first key press while the tip shows, when the tip has an action)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (contribution id terminal.cursorCliPrompt; experiment name terminal_tip with parameters enabled, message, action, show_count and show_every_hours; stored state terminalTipState; CLI detection status values claudeStatus, codexStatus and agentStatus; NLS module terminalContrib/cursorCliPrompt)

### PN-2.50 Install Cursor CLI command
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the command installs the Cursor agent command line tool, so its purpose is an agent surface even though the UI is a plain terminal.
- What: Run Install Cursor CLI to open a new terminal in the panel, focus it and run the installer line (curl https://cursor.com/install -fsS piped to bash), or get the message "Cursor CLI is already installed." when the app has already detected the tool; the palette flag f1 is set to the negation of a constant I could not resolve (it may mean Windows, since the line is a bash script, not verified).
- Commands: cursor.installCli
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (command id cursor.installCli; NLS keys cursor.installCli and cursor.cliAlreadyInstalled in module terminalContrib/cursorCliPrompt)

## Handed off

- Open in Integrated Terminal from Explorer folders and files, and from the Source Control repositories view (it starts a terminal in that folder): SB-2 and SB-4, where the menu entries and the `terminal.explorerKind` and `terminal.sourceControlRepositoriesKind` settings already live.
- Terminal menu rows in the menu bar (New Terminal, Split Terminal, Run Active File, Run Selected Text and the task rows Run Task, Run Build Task, Show Running Tasks, Restart Running Task, Terminate Task, Configure Default Build Task): the menu structure belongs to TB-1; the terminal behavior behind the first four is described here.
- Task pickers (the Run Task list, Run Build Task, Terminate Task and Restart Running Task pick lists, Show Running Tasks): QA-4.
- The Switch Terminal quick pick opened with the `term ` prefix and Switch Active Terminal: QA-4.
- Clear, Select All, Copy, Copy as HTML, Paste, scroll commands, select-to-command commands, find, links and font or cursor appearance: PN-3.
- Show Environment Contributions as a command (PN-2.13 only shows the hover link that runs it), Start and Stop Dictation (stock only), shell integration decorations, command guide, sticky scroll and the quick fix lightbulb settings (`terminal.integrated.shellIntegration.decorationsEnabled`, `quickFixEnabled`): PN-3.
- Terminal suggest popup: AC-5.
- Show, hide, maximize and move the whole panel, and the panel tab bar: PN-1.
- Terminal chat, the Cmd+K terminal generate bar, the hidden chat terminals row in the tabs list, the external agent buttons on the view toolbar (NLS key `terminal.externalAgentActions` in Cursor), agent hosted terminals and the agent profile settings: left out of scope, not listed.
- The warning "The terminal only supports monospace fonts..." with its Use 'monospace' button (shown when the terminal or editor font setting changes to a font that is not monospace; `terminalView.ts` lines 203-212) and the two developer test settings `terminal.integrated.developer.ptyHost.latency` and `terminal.integrated.developer.ptyHost.startupDelay` (`terminalConfiguration.ts` lines 681-694): PN-3.

## Unclear

- "Links to launch" in the request: I read it as the launch entry points and the links around launching a terminal, which are the Launch Profile drop-down (PN-2.4), the Open Help action on a failed launch (PN-2.32), the Learn more about shell integration and Open user settings links (PN-2.13 and PN-2.38), and Open in Integrated Terminal from Explorer and Source Control (handed off). I found no welcome view with command links in the Terminal view in V; its welcome state only exists when no terminal process support is registered (terminalView.ts lines 350-356) and I found no welcome text registered for it.
- Run Selected Text and Run Active File (PN-2.10) could also be claimed by PN-3 or by the editor groups, because they start in the editor. Pass 1 said the Terminal menu commands are described under PN-2, so I kept them here.
- `workbench.action.terminal.newInActiveWorkspace`, `splitInActiveWorkspace` and `newLocal` (PN-2.2 and PN-2.21): their names say what they do, but I did not trace who calls them or whether the palette shows them in a plain local window. `newLocal` is registered from the Electron-only remote file.
- The "JavaScript Debug Terminal" profile (PN-2.4) comes from the `ms-vscode.js-debug` extension in the Cursor install; stock VS Code downloads that extension at build time so it is not in V, and I did not check a stock release to say whether the profile is stock or Cursor only.
- Newer than 1.128: no new command or setting. Comparing the command and setting id enums of V and N, the only new setting ids are `terminal.integrated.fontRendering` and `terminal.integrated.windowsReparentConsoleWindow`, both PN-3. The terminal files for this group changed in N only in small ways (the rename box in the tabs list moves to a shared inline edit component, cancelling Change Icon on several tabs stops the loop, and a Start Dictation row is added to the terminal right-click menu), so no feature here carries the newer-than marker.
- Cursor is missing or changed against V for these ids and strings (each searched in the bundle, the key list or both): `workbench.action.terminal.newInNewWindow` with its menu row, the three setting ids `terminal.integrated.allowInUntrustedWorkspace`, `terminal.integrated.shellIntegration.timeout` and `terminal.integrated.tabs.allowAgentCliTitle` (and `shellIntegration.quickFixEnabled`, which belongs to PN-3), the New Terminal rows in the editor tab bar and empty-group menus, the compact window title menu, the top rows of the Launch Profile list, and a spelling difference in the folder picker text. This matches the pass 1 note that Cursor's terminal code is older than V in places.
- `terminal.integrated.tabs.allowAgentCliTitle` controls whether agent command line tools may set the tab title. It is a plain tab title setting, so I listed it in PN-2.11 even though the tools it names are agents.
- I read source and the Cursor bundle only. I did not run either app, so what shows on a fresh window (for example which toolbar buttons are visible by default, or the exact tab strip look) comes from code and settings defaults.
- Overflow check on 2026-10-25 added 10 features (PN-2.41 to PN-2.50).
