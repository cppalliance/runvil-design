Group: QA-4 Switch and open lists. Section: Quick access. Date: 2026-10-25. Pass: 2.
Features: 19.

### QA-4.1 Open a view, panel, terminal or output channel by name
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Open View (or type `view ` in the box) and pick a row to open a Side Bar, Panel or Secondary Side Bar container, one of its views, a terminal, a debug console, or an Output channel. Rows sit under heading rows ("Side Bar", "Panel", "Secondary Side Bar", then one per container such as "Side Bar / Explorer", then "Terminal", "Debug Console", "Output"). Typing fuzzy-matches the view name or any part of its container name, and containers marked hide-if-empty are left out. With no match the list shows "No matching views".
- Commands: workbench.action.openView; workbench.action.quickOpenView
- Menus: MenubarViewMenu (View > Open View..., group 1_open, order 2); CommandPalette (Open View, f1; Quick Open View is f1 false so it stays out of the palette)
- Keys: Ctrl+Q (workbench.action.quickOpenView; Control+Q on Mac; no default key on Linux)
- Settings: none found
- Cursor: present (both ids, the `inViewsPicker` context key, the placeholder, the "&&Open View..." label, "No matching views" and the Ctrl+Q key rule with Linux set to none were found)
- Source: V\src\vs\workbench\contrib\quickaccess\browser\viewQuickAccess.ts (32-262); V\src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts (33-40, 73-80, 117-145); C bundle (workbench.action.openView, workbench.action.quickOpenView, inViewsPicker, provider `sdt`, message 8910 to 8912, nls "No matching views")

### QA-4.2 Pick an editor from the active group by most recent use
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Show Editors in Active Group By Most Recently Used" (or type `edt active `, or choose Show Opened Editors from the tab strip "..." menu) to list only the editors of the active group, most recently used first. Typing filters on name and description, picking a row opens that editor, and Ctrl+Tab and Ctrl+Shift+Tab open this same list while the key is held.
- Commands: workbench.action.showEditorsInActiveGroup; workbench.action.showEditorsInGroup; workbench.action.quickOpenPreviousRecentlyUsedEditorInGroup; workbench.action.quickOpenLeastRecentlyUsedEditorInGroup; workbench.action.quickOpenNavigateNextInEditorPicker; workbench.action.quickOpenNavigatePreviousInEditorPicker
- Menus: EditorTitle (Show Opened Editors, group 3_open, order 10); CommandPalette (View: Show Editors in Active Group By Most Recently Used)
- Keys: Ctrl+Tab and Ctrl+Shift+Tab (Control+Tab and Control+Shift+Tab on Mac; the two in-group commands need a non-empty active group; the cycle commands apply when inQuickOpen and inEditorsPicker)
- Settings: none found
- Cursor: changed (all ids, labels and keys found; the Ctrl+Tab in-group command has extra preconditions that test agents pane focus and composer focus)
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (35-190, 192-212); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (143-152, 312-331, 424); V\src\vs\workbench\browser\parts\editor\editorActions.ts (1710-1727, 1778-1861); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (1430-1446); C bundle (providers `rIe`, `tst`, `sIe`; key rule for quickOpenPreviousRecentlyUsedEditorInGroup with its precondition; ids showEditorsInActiveGroup, showEditorsInGroup; nls "Show Opened Editors", messages 3412 to 3416)

### QA-4.3 Pick from all open editors in order of appearance
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Show All Editors By Appearance" (or type `edt `) to list every open editor in all groups, left to right and group by group. A heading row names each group when more than one group is open, unsaved editors are marked, and editors in preview mode are drawn in italics.
- Commands: workbench.action.showAllEditors
- Menus: CommandPalette (File: Show All Editors By Appearance)
- Keys: Ctrl+K Ctrl+P (Cmd+Alt+Tab on Mac)
- Settings: none found
- Cursor: present (id and title found, help text "Show All Opened Editors By Appearance" found; the key chord was not pulled from the Cursor key table)
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (35-190, 219-245); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (154-160); V\src\vs\workbench\browser\parts\editor\editorActions.ts (1730-1755); C bundle (class with id workbench.action.showAllEditors, provider `tst`, messages 3414 and 3415)

### QA-4.4 Pick from all open editors by most recent use
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Show All Editors By Most Recently Used" (or type `edt mru `) to list every open editor from all groups as one list, newest first. Two more palette commands open the same list for quick stepping, one starting on the previous used editor and one on the least recently used.
- Commands: workbench.action.showAllEditorsByMostRecentlyUsed; workbench.action.quickOpenPreviousRecentlyUsedEditor; workbench.action.quickOpenLeastRecentlyUsedEditor
- Menus: CommandPalette (View: Show All Editors By Most Recently Used; View: Quick Open Previous Recently Used Editor; View: Quick Open Least Recently Used Editor)
- Keys: none found
- Settings: none found
- Cursor: present (all three ids and the help text "Show All Opened Editors By Most Recently Used" found)
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (251-275); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (162-168); V\src\vs\workbench\browser\parts\editor\editorActions.ts (1757-1822); C bundle (ids found, provider `sIe`, message 3416)

### QA-4.5 Close an editor from its row in the editor lists
- Tag: both
- Evidence: source
- Boundary: no
- What: Each row in the three editor lists has a "Close Editor" button on hover (an x). For an editor with unsaved changes the button is a dot and stays visible. Clicking it closes that editor and removes the row while the list stays open and focus stays in the box.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the "Close Editor" label and the three editor providers were found; the button wiring itself was not traced)
- Source: V\src\vs\workbench\browser\parts\editor\editorQuickAccess.ts (140-190); C bundle (nls "Close Editor", providers `rIe`, `tst`, `sIe`)

### QA-4.6 Switch Active Terminal from a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Terminal: Switch Active Terminal" (or type `term `) to list the panel terminals under a "panel" heading and terminal editors under an "editor" heading, each row with its icon, color, number and title (for example "1.2: bash") and its description. Picking a row shows and focuses that terminal, and Right arrow opens it without leaving the list. Two rows at the bottom, "Create New Terminal" and "Create New Terminal With Profile...", start a new one.
- Commands: workbench.action.quickOpenTerm; workbench.action.quickOpenNavigateNextInTerminalPicker; workbench.action.quickOpenNavigatePreviousInTerminalPicker; workbench.action.terminal.new; workbench.action.terminal.newWithProfile
- Menus: CommandPalette (Terminal: Switch Active Terminal)
- Keys: none found (the two navigate commands are registered with no default key)
- Settings: none found
- Cursor: present (id, `inTerminalPicker` context key, provider `jLt`, nls "Show All Opened Terminals", "Create New Terminal", "Create New Terminal With Profile..." and both navigate ids found)
- Source: V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminalQuickAccess.ts (25-143); V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminal.quickAccess.contribution.ts (18-40); C bundle (workbench.action.quickOpenTerm, provider `jLt`, messages 10862, 10863 and 10865)

### QA-4.7 Rename or kill a terminal from its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Each terminal row in the Switch Active Terminal list shows a pencil "Rename Terminal" button and a trash "Kill Terminal" button on hover. Rename asks for a new name and leaves the list open. Kill ends that terminal at once and removes the row.
- Commands: workbench.action.terminal.rename
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the "Rename Terminal" label (message 10867) and the provider were found; the kill button calls the terminal service directly, so it has no command id)
- Source: V\src\vs\workbench\contrib\terminalContrib\quickAccess\browser\terminalQuickAccess.ts (82-143); C bundle (nls "Rename Terminal", provider `jLt`)

### QA-4.8 Run a task from the Run Task list
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Terminal > Run Task... (or the palette command) to open "Select the task to run". It shows a "recently used" section, a "configured" section, a "contributed" section with one folder row per task provider (for example npm), and a "Show All Tasks..." row. If no task exists, a "Configure a Task" row opens the setup. Picking a task runs it with its problem matcher. Picking a provider row opens a second step listing that provider's tasks. Each top level row also has a pin button that moves the task into a "pinned" section at the top (the pin widget itself is QA-6.24).
- Commands: workbench.action.tasks.runTask
- Menus: MenubarTerminalMenu (Run Task..., group 3_run, order 1, when TaskExecutionSupportedContext); CommandPalette (Tasks: Run Task)
- Keys: none found
- Settings: task.quickOpen.history; task.quickOpen.detail; task.quickOpen.skip; task.quickOpen.showAll
- Cursor: present (id found 6 times, all four task.quickOpen settings found, nls "Run Task", "Show All Tasks..." and "recently used" found)
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (585-620, 3110-3221); V\src\vs\workbench\contrib\tasks\browser\taskQuickPick.ts (14-260, 311-320); V\src\vs\platform\quickinput\browser\quickPickPin.ts; V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (163-173, 267-272, 515-536); C bundle (workbench.action.tasks.runTask, task.quickOpen.* settings, messages 9977 and 9983, pin function with message 2257)

### QA-4.9 Run a task by typing the task prefix
- Tag: both
- Evidence: source
- Boundary: no
- What: Type `task ` in the quick access box (placeholder "Type the name of a task to run.") to get the same top level task rows as Run Task, filtered by name as you type, with "No matching tasks" when nothing fits. Picking a task runs it, and picking a provider folder row opens the second step. The help list names it "Run Task", and the command center empty-box shortcut list has a Run Task row (order 60, just before More).
- Commands: none found (the help entry names no command id; the Run Task command in QA-4.8 opens a different picker)
- Menus: none found
- Keys: none found
- Settings: task.quickOpen.history; task.quickOpen.detail
- Cursor: present (provider `Wln`, context key `inTasksPicker`, placeholder 9941, help text 9942 and command center order 60 found)
- Source: V\src\vs\workbench\contrib\tasks\browser\tasksQuickAccess.ts (23-97); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (401-410); C bundle (provider `Wln`, messages 9941 and 9942)

### QA-4.10 Configure a task or forget a recent task from its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Task rows in the Run Task list and the `task ` list show a gear "Configure Task" button that opens or creates the task in tasks.json and closes the list. Rows in the "recently used" section also show an x "Remove Recently Used Task" button that drops the row from history and refreshes the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: task.quickOpen.history
- Cursor: present (nls "Configure Task" and "Remove Recently Used Task" found)
- Source: V\src\vs\workbench\contrib\tasks\browser\taskQuickPick.ts (42-48, 102-110, 190-197, 233-262); V\src\vs\workbench\contrib\tasks\browser\tasksQuickAccess.ts (62-80); C bundle (nls "Configure Task", "Remove Recently Used Task")

### QA-4.11 Run the build or test task, choosing one when there is no single default
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Build Task (Ctrl+Shift+B) or Run Test Task runs the default task of that group at once. When several tasks qualify (several defaults, or several glob matches for the open file) a list "Select the build task to run" or "Select the test task to run" appears, and when none qualifies a row offers to configure one.
- Commands: workbench.action.tasks.build; workbench.action.tasks.test
- Menus: MenubarTerminalMenu (Run Build Task..., group 3_run, order 2, when TaskExecutionSupportedContext); CommandPalette (Tasks: Run Build Task; Tasks: Run Test Task)
- Keys: Ctrl+Shift+B (workbench.action.tasks.build; Cmd+Shift+B on Mac; when TaskCommandsRegistered); Run Test Task has no default key
- Settings: none found
- Cursor: present (both ids, nls "Run Build Task", "Run Test Task", "Select the build task to run" and "Configure Build Task..." found; the Cursor key table was not pulled)
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (3260-3430); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (175-183, 311-327, 387-392); C bundle (workbench.action.tasks.build menu item and palette item, workbench.action.tasks.test)

### QA-4.12 Show a running task from a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Terminal > Show Running Tasks... (greyed when no task runs). With one running task, or several in one terminal group, its terminal is revealed at once. Otherwise the list "Select the task to show its output" appears, and "No task is running" shows if the list is empty.
- Commands: workbench.action.tasks.showTasks
- Menus: MenubarTerminalMenu (Show Running Tasks..., group 5_manage, order 1, precondition TASK_RUNNING_STATE); CommandPalette (Tasks: Show Running Tasks)
- Keys: none found
- Settings: none found
- Cursor: present (id found 6 times, the menu item in group 5_manage and nls "Show Running Tasks" found; the status bar items that also run it are in SS-5)
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (679-683, 3900-3935); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (186-195, 296-303); C bundle (workbench.action.tasks.showTasks, nls "Show Running Tasks")

### QA-4.13 Terminate or restart a running task from a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Terminal > Terminate Task... lists the running tasks under "Select a task to terminate" with an extra "All Running Tasks" row, and "No task is currently running" when empty. Terminal > Restart Running Task... lists them under "Select the task to restart" and shows "No task to restart" when empty. A single choice acts at once. Both are greyed when no task runs.
- Commands: workbench.action.tasks.terminate; workbench.action.tasks.restartTask
- Menus: MenubarTerminalMenu (Restart Running Task..., group 5_manage, order 2; Terminate Task..., group 5_manage, order 3; both with precondition TASK_RUNNING_STATE); CommandPalette (Tasks: Restart Running Task; Tasks: Terminate Task)
- Keys: none found
- Settings: none found
- Cursor: present (both ids found 4 times each; nls "Terminate Task...", "Restart Running Task...", "Select a task to terminate" and "Select the task to restart" found)
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (625-645, 3425-3475, 3490-3532); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (197-217, 279-289, 303-311); C bundle (ids and the four labels)

### QA-4.14 Pick a task to configure or to make the default build or test task
- Tag: both
- Evidence: source
- Boundary: no
- What: Terminal > Configure Tasks... opens a list "Select a task to configure" and opens the chosen task in tasks.json. Terminal > Configure Default Build Task... and the matching test command list tasks under "Select the task to be used as the default build task" (or test task) and mark the chosen one as the default.
- Commands: workbench.action.tasks.configureTaskRunner; workbench.action.tasks.configureDefaultBuildTask; workbench.action.tasks.configureDefaultTestTask
- Menus: MenubarTerminalMenu (Configure Tasks..., group 7_configure, order 1; Configure Default Build Task..., group 7_configure, order 2); CommandPalette (Tasks: Configure Task; Tasks: Configure Default Build Task; Tasks: Configure Default Test Task)
- Keys: none found
- Settings: none found
- Cursor: present (all three ids and nls "Configure Tasks...", "Select a task to configure" and "Select the task to be used as the default build task" found)
- Source: V\src\vs\workbench\contrib\tasks\browser\abstractTaskService.ts (661-676, 3700-3890); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (219-239, 249-256, 327-345); C bundle (workbench.action.tasks.configureTaskRunner, configureDefaultBuildTask, configureDefaultTestTask)

### QA-4.15 Start debugging from the launch configuration list
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Debug: Select and Start Debugging" (or type `debug `, or click the Debug status bar item) to list launch configurations under a "launch.json" heading, with the last used one first under "Most Recent". Entries from debug providers sit under "contributed" (recent dynamic configurations, and one "Provider..." row per provider). Picking a row selects that configuration and starts debugging, and a failed start shows an error notification. With no debugger available the list is empty. The same command is behind "Start Additional Session" in the Run and Debug view menu while debugging with a docked toolbar.
- Commands: workbench.action.debug.selectandstart
- Menus: CommandPalette (Debug: Select and Start Debugging); ViewContainerTitle (Start Additional Session, Run and Debug container, when debugging and the toolbar is docked or the launcher is hidden)
- Keys: none found (F5 starts the selected configuration without showing this list)
- Settings: debug.showInStatusBar
- Cursor: present (id found, `inLaunchConfigurationsPicker` context key, provider `w5a`, placeholder 5614, help text "Start Debugging" 5615 with command center order 50, nls "Most Recent" and "launch.json" and the `status.debug` item found)
- Source: V\src\vs\workbench\contrib\debug\browser\debugQuickAccess.ts (18-170); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (86-98, 145, 546-551); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (68, 125, 724-746); V\src\vs\workbench\contrib\debug\browser\debugStatus.ts (24-72); V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts (290-303); C bundle (workbench.action.debug.selectandstart, provider `w5a`, status.debug, debug.showInStatusBar)

### QA-4.16 Edit, remove or add launch configurations from the list
- Tag: both
- Evidence: source
- Boundary: no
- What: In the Start Debugging list a gear "Configure Launch Configuration" button on each launch.json row opens that launch file and closes the list. Recent dynamic configurations show an x "Remove Launch Configuration" button that forgets them and closes the list. A "configure" section at the bottom has "Add Configuration..." rows (one per launch file, named "Add Config (name)..." in a multi-root workspace) that run the add configuration flow.
- Commands: debug.addConfiguration
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (id debug.addConfiguration, nls "Configure Launch Configuration", "Remove Launch Configuration" and "Add Configuration..." found)
- Source: V\src\vs\workbench\contrib\debug\browser\debugQuickAccess.ts (50-66, 105-117, 145-168); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (46); C bundle (debug.addConfiguration, messages 5827 and 5830)

### QA-4.17 Pick a debug console from a list
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Debug: Select Debug Console" (or type `debug consoles `) to list the debug sessions that have their own console. Picking one focuses that session and opens the Debug Console view, and Right arrow does it without leaving the list. A last row, "Start a New Debug Session", runs the Start Debugging list.
- Commands: workbench.action.debug.selectDebugConsole
- Menus: CommandPalette (Debug: Select Debug Console)
- Keys: none found
- Settings: none found
- Cursor: present (id, `inDebugConsolePicker` context key, provider `h5a`, messages 5616 and 5617, nls "Select Debug Console" and "Start a New Debug Session" found)
- Source: V\src\vs\workbench\contrib\debug\browser\debugConsoleQuickAccess.ts (15-65); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (100-107, 149); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (69, 126, 749-754); C bundle (workbench.action.debug.selectDebugConsole, provider `h5a`)

### QA-4.18 Select a debug session by name
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Debug: Select Debug Session" for a list with the placeholder "Search debug sessions by name". Sessions started from a parent show the parent's name as a description, the focused session is highlighted on open, and picking one focuses it and opens the Debug Console. A last row, "Start a New Debug Session", opens the Start Debugging list. This list is a plain quick pick, not a quick access prefix.
- Commands: workbench.action.debug.selectDebugSession
- Menus: CommandPalette (Debug: Select Debug Session)
- Keys: none found
- Settings: none found
- Cursor: present (id and nls "Select Debug Session" and "Search debug sessions by name" found)
- Source: V\src\vs\workbench\contrib\debug\browser\debugSessionPicker.ts (14-122); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (70, 757-761); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (150); C bundle (workbench.action.debug.selectDebugSession, message 5737 or 5901 for "Start a New Debug Session")

### QA-4.19 List extensions to report an issue on with the issue prefix
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Type `issue ` to list the product name and "Extension Marketplace" first, then every installed non-built-in extension and extension-contributed issue entry, sorted by name. Picking a row opens the Issue Reporter for it, an info button opens the extension page, and Right arrow picks without closing. The help list calls it "Open Issue Reporter". It exists only in the desktop build, only while `telemetry.feedback.enabled` and `extensions.experimental.issueQuickAccess` (default on) are both on, and the rows are drawn from the extension list (an out of scope system), which is why it is a judgment call to catalog it here.
- Commands: workbench.action.openIssueReporter
- Menus: IssueReporter (extension-contributed rows)
- Keys: none found
- Settings: extensions.experimental.issueQuickAccess; telemetry.feedback.enabled
- Cursor: missing (no provider, no `inReportIssuePicker` context key, no `extensions.experimental.issueQuickAccess` setting, and none of the labels "Open Issue Reporter", "Type the name of an extension to report on." or "Open Extension Page" in the bundle; the command id workbench.action.openIssueReporter itself exists in the bundle)
- Source: V\src\vs\workbench\contrib\issue\browser\issueQuickAccess.ts (20-150); V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts (78-121); V\src\vs\workbench\contrib\extensions\browser\extensions.contribution.ts (316-320); C bundle (searched ids and labels above, all absent except the command id)

## Handed off

- Open Recent picker (`workbench.action.openRecent`, Ctrl+R, the "More..." row of File > Open Recent, and the Ctrl+R stepping): TB-1. TB-1.14 lists the submenu. TB-1's own hand-off line points the picker back to QA-4, so one of the two needs to take it; the parent brief for this pass says it is owned elsewhere.
- Switch Window picker (`workbench.action.switchWindow`, `workbench.action.quickSwitchWindow`): TB-1 (the command is in its macOS Window menu feature). It is a plain quick pick built in windowActions.ts.
- Quick Open Previous Editor from History (`workbench.action.openPreviousEditorFromHistory`) and the Go to File list that opens with the second row active (`workbench.action.quickOpenPreviousEditor`): QA-2 (QA-2.18) and ED-1.30.
- Ctrl+Tab stepping, the held-key behavior and release-to-open for the editor, view and terminal lists: QA-5 (QA-5.11 and QA-5.13) and ED-1.30. QA-4.1 and QA-4.2 name the keys only as entry points.
- Go > Switch Editor submenu (Next Editor, Next Used Editor, in group variants), which switches without a list: TB-1.21 and ED-1.29.
- Rerun All Running Tasks (`workbench.action.tasks.rerunAllRunningTasks`, palette only, no list; not in the Cursor bundle): PN-2 task features (PN-2.37).
- Next and previous debug console commands (`workbench.action.debug.nextConsole`, `workbench.action.debug.prevConsole`) and the Debug Console view's own console switcher: PN-6.
- The Start Debugging play button, launch configuration drop-down and the Debug status bar item itself: SB-5.3, SB-5.4 and SS-5.

## Unclear

- Overlap with ED-1: ED-1.30 and ED-1.31 already list the same editor switcher commands and the `edt` prefixes, and QA-2 and PN-2 hand the lists to QA-4. QA-4.2 to QA-4.5 describe the lists themselves, so ED-1.31 could point at them instead of repeating the commands. Same for PN-2's hand-off of the Run Task, Run Build Task, Terminate Task and Restart Running Task pick lists, which are QA-4.8, QA-4.11 and QA-4.13.
- "Other prefix providers visible in the help list" turned up only one in-scope extra, `issue ` (QA-4.19). Not listed because they are out of scope: `ext ` and `ext install ` (extension system, both present in the Cursor bundle), `mcpr ` (MCP, absent from Cursor), `agent ` (agent sessions, absent from Cursor), and Cursor's own `chat:` and `agents mru ` providers (agent pickers, bundle messages 4786 and 4793).
- Whether the quick access Open View list in Cursor lists the same containers as stock was not traced. The Cursor tab row that replaces the activity bar (AB-6) may change which containers exist; only the provider and its strings were checked.
- The `task ` prefix has no command id of its own. It is reachable by typing the prefix, from the help list, and from the command center shortcut row. The palette command Run Task opens a different two-level picker.
- Cursor Ctrl+Tab precondition: the in-group command is gated by agents pane focus and composer focus. What the gate evaluates to when the agent pane has focus was not traced, so it is not clear whether Ctrl+Tab does nothing there or falls through to another binding.
- Default keys for Cursor were not pulled from the Cursor key table except where stated (Ctrl+Q rule, Ctrl+Tab rule, Ctrl+Shift+B menu item). The other keys are from V.
- Behavior on screen was not watched. Every statement comes from reading source or bundle strings. Nothing newer than 1.128.0 was found for this group: the picker files for views, editors, terminals, tasks and debug are line for line equal in the 1.144.0 clone (N).
