Group: SB-5 Run and Debug. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 31.

### SB-5.1 Open the Run and Debug view
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+D, pick Run from the View menu, or click the Run and Debug icon to open the side bar that stacks Variables, Watch, Call Stack, Breakpoints and Loaded Scripts (a Run welcome view stands in before the first run, see SB-5.8).
- Commands: workbench.view.debug
- Menus: MenubarViewMenu (item generated from the container's openCommandActionDescriptor, mnemonic title "&&Run")
- Keys: Ctrl+Shift+D (Cmd+Shift+D on macOS)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:453-475; V\src\vs\workbench\services\views\browser\viewsService.ts:457; C bundle (container id workbench.view.debug with the same Ctrl+Shift+D chord, order 3)

### SB-5.2 Let the view open itself when debugging starts or hits a breakpoint
- Tag: both
- Evidence: source
- Boundary: no
- What: The Run and Debug view opens on its own - by default when a breakpoint is hit, or on every session start, or only the first one, or never - and the Explorer can be brought back when the session ends.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.openDebug; debug.openExplorerOnEnd
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:514-518, 563-567; V\src\vs\workbench\contrib\debug\browser\debugService.ts:643-650, 791; V\src\vs\workbench\contrib\debug\browser\debugSession.ts:1392; C bundle (debug.openDebug enum with openOnDebugBreak as default, debug.openExplorerOnEnd)

### SB-5.3 Start Debugging button in the view title
- Tag: both
- Evidence: source
- Boundary: no
- What: A play button (tooltip shows the key) in the view title starts the selected launch configuration (F5 starts it, Ctrl+F5 runs it without the debugger); the button and the gear next to it are hidden in the simple layout (before a configuration exists) and when no folder is open, the button can be hidden while debugging, and "Start Additional Session" appears in the "..." menu once the toolbar is docked or the button is hidden.
- Commands: workbench.action.debug.start; workbench.action.debug.run; workbench.action.debug.selectandstart; debug.openView
- Menus: ViewContainerTitle (group navigation, order 10, for the Run and Debug container); ViewContainerTitle (Start Additional Session, order 10, no group); MenubarDebugMenu (group 1_debug, Start Debugging and Run Without Debugging)
- Keys: F5 (when debuggersAvailable and debugState is inactive); Ctrl+F5 (Ctrl+F5 on macOS, using the Control key); F5 and Ctrl+F5 when no debugger extension exists (opens this view)
- Settings: debug.hideLauncherWhileDebugging; debug.saveBeforeStart; debug.onTaskErrors; debug.hideSlowPreLaunchWarning
- Cursor: changed (the Run and Debug container asks for a second title row, wantsActionToolbar, 70 px tall, to hold the button and picker; the button code and ids are otherwise the same)
- Source: V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:171-192, 290-303; V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:82-122; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:764-789, 1161-1176; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:272-282, 651-700; C bundle (start-debug-action-item render code, workbench.action.debug.start and run ids, wantsActionToolbar on the container registration and sidebar heightOfTitle 70)

### SB-5.4 Pick a launch configuration from the dropdown
- Tag: both
- Evidence: source
- Boundary: no
- What: The dropdown beside the play button shows the selected configuration and lists every configuration and compound from all launch scopes (grouped and ordered by launch.json presentation group and order, hidden ones left out), up to three recent dynamic configurations, one "Provider..." row per dynamic provider, and an "Add Configuration..." row per folder, and choosing a row only selects it (the button starts it).
- Commands: workbench.action.debug.start
- Menus: ViewContainerTitle (the dropdown is rendered by the Start Debugging action view item)
- Keys: Right Arrow (button to dropdown); Left Arrow (dropdown to button)
- Settings: none found
- Cursor: changed (an older native-style select box with separator rows, no group categories and no "Search configurations" filter box; stock 1.128.0 uses an action widget dropdown with a filter)
- Source: V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:36-353; V\src\vs\workbench\contrib\debug\common\debugSchemas.ts:163-187; V\src\vs\workbench\contrib\debug\browser\debugConfigurationManager.ts:314-326; C bundle (StartDebugActionViewItem code with this.selectBox.setOptions and separator options; NLS has "Debug Launch Configurations" and "No Configurations" but not "Search configurations")

### SB-5.5 Open or create launch.json with the gear button
- Tag: both
- Evidence: source
- Boundary: no
- What: The gear beside the picker opens the launch.json of the selected configuration's folder (asking which folder or the workspace file when none is selected, or opening the workspace file or the `launch` entry in user settings when the selected configuration lives there), scrolling to the chosen entry; if no file exists, a "Select debugger" list (Suggested, "More X options...", "Install an extension for X...") builds a starter file.
- Commands: workbench.action.debug.configure
- Menus: ViewContainerTitle (group navigation, order 20; a second entry at order 20 shows while debugging with a docked toolbar); MenubarDebugMenu (group 2_configuration, order 1, "Open Configurations"); CommandPalette (f1); DebugCreateConfiguration
- Keys: none found
- Settings: launch
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:194-262; V\src\vs\workbench\contrib\debug\browser\debugConfigurationManager.ts:630-800; V\src\vs\workbench\contrib\debug\browser\debugAdapterManager.ts:382-456; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:611-619; C bundle (workbench.action.debug.configure, NLS "Select debugger", "Suggested", "More {0} options...", "Install an extension for {0}...", "Select a workspace folder to create a launch.json file in or add it to the workspace config file", "user settings")

### SB-5.6 Add a configuration to launch.json
- Tag: both
- Evidence: source
- Boundary: no
- What: "Add Configuration..." (from the launch.json editor button, the picker's last row, the Run menu or the palette) inserts a new entry at the top of the `configurations` array and opens the snippet suggestions list so the person can pick a template.
- Commands: debug.addConfiguration
- Menus: EditorContent (only for .vscode/launch.json in a text editor); MenubarDebugMenu (group 2_configuration, order 2); CommandPalette (f1, category Debug)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:985-1015; V\src\vs\workbench\contrib\debug\browser\debugEditorContribution.ts:678-740; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:313-322; V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:325-333; C bundle (debug.addConfiguration id, regex for .vscode launch.json, NLS "Add Configuration..." and "Add Config ({0})...")

### SB-5.7 Run several configurations at once with a compound
- Tag: both
- Evidence: source
- Boundary: no
- What: A `compounds` entry in launch.json (name, list of configuration names or name-and-folder pairs, stopAll, preLaunchTask, presentation) shows up in the picker like a normal configuration and starts all its sessions together, with an error if a named configuration is missing or ambiguous.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: launch
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\common\debugSchemas.ts:189-263; V\src\vs\workbench\contrib\debug\browser\debugService.ts:376-435; V\src\vs\workbench\contrib\debug\browser\debugConfigurationManager.ts:372-380, 531-560; C bundle (NLS "List of compounds. Each compound references multiple configurations which will get launched together.", "Controls whether manually terminating one session will stop all of the compound sessions.", "Task to run before any of the compound configurations start.", "Could not find launch configuration")

### SB-5.8 Run welcome view before the first run
- Tag: both
- Evidence: source
- Boundary: no
- What: Until a configuration is selected or a session has run, the container shows a "Run" welcome view instead of Variables, Watch, Call Stack and Loaded Scripts (Breakpoints stays only if some exist), with an "Open a file which can be debugged or run" link, a "Run and Debug" button, a "create a launch.json file" link (or "open a folder" when none is open) and, when every debug extension is off, a plain-text notice.
- Commands: workbench.action.debug.start; workbench.action.debug.configure; workbench.action.files.openFile; workbench.action.files.openFolder
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\welcomeView.ts:31-167; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:470-476; V\src\vs\workbench\contrib\debug\browser\debugService.ts:311-313; C bundle (view id workbench.debug.welcome with label "Run", NLS "[Open a file](command:{0}) which can be debugged or run.", "To customize Run and Debug [create a launch.json file]({0}).", "All debug extensions are disabled.")

### SB-5.9 Debug toolbar docked into the view title
- Tag: both
- Evidence: source
- Boundary: no
- What: With debug.toolBarLocation set to docked, the Run and Debug title swaps the play button for the debug toolbar while a session is active - Continue or Pause, Step Over, Step Into, Step Out, Restart, Stop with a dropdown (Disconnect, Disconnect and Suspend) and, for adapters that support them, Step Back and Reverse; the default location is a floating bar (ED-11).
- Commands: workbench.action.debug.continue; workbench.action.debug.pause; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop; workbench.action.debug.disconnect; workbench.action.debug.disconnectAndSuspend; workbench.action.debug.stepBack; workbench.action.debug.reverseContinue
- Menus: DebugToolBar (group navigation, orders 10 to 100, mirrored into ViewContainerTitle while docked); DebugToolBarStop (items of the stop dropdown)
- Keys: F5 (Continue, when stopped); F6 (Pause, when running); F10 (Step Over, when stopped); F11 (Step Into, when a session is active; Alt+F11 on Windows web); Shift+F11 (Step Out, when stopped); Shift+F5 (Stop, or Disconnect for attach sessions, in debug mode); Ctrl+Shift+F5 (Restart, in debug mode)
- Settings: debug.toolBarLocation
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugToolBar.ts:390-491; V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:100-117; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:471-692; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:535-547; C bundle (menu ids DebugToolBar and DebugToolBarStop, debug.toolBarLocation enum floating, docked, commandCenter, hidden, the same command ids and F-key rules)

### SB-5.10 Switch the focused debug session
- Tag: both
- Evidence: source
- Boundary: no
- What: When more than one session runs, a "Debug Session" dropdown in the docked toolbar lists them (child sessions indented, or hidden unless sub-sessions are enabled) and picking one changes which session the Variables, Watch, Call Stack and Breakpoints views show.
- Commands: workbench.action.debug.focusProcess
- Menus: DebugToolBar (group navigation, order 100, only with several sessions and not in the command center); ViewContainerTitle (mirror while docked)
- Keys: none found
- Settings: debug.showSubSessionsInToolBar
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugActionViewItems.ts:355-430; V\src\vs\workbench\contrib\debug\browser\debugToolBar.ts:456; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:709-721; V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts:100-107; C bundle (workbench.action.debug.focusProcess id, NLS "Debug Session" and "Focus Session", setting debug.showSubSessionsInToolBar)

### SB-5.11 Browse variables by scope
- Tag: both
- Evidence: source
- Boundary: no
- What: The Variables view shows the scopes (Locals, Globals and so on) of the focused stack frame as a tree, opens the first cheap scope by itself, remembers expansion per frame, resolves lazy values on a click of the eye icon ("Click to expand"), can show each value's type, and has a Collapse All button.
- Commands: workbench.debug.action.focusVariablesView; variables.collapse
- Menus: ViewTitle (Collapse All, view Variables); DebugScopesContext
- Keys: none found
- Settings: debug.showVariableTypes; debug.autoExpandLazyVariables
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\variablesView.ts:75-215, 833-851; V\src\vs\workbench\contrib\debug\browser\baseDebugView.ts:154-176; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:470, 491-495, 677-686; C bundle (view id workbench.debug.variablesView, focus command with 3 hits, variables.collapse, NLS "Debug Variables" and "Click to expand")

### SB-5.12 Edit a variable's value
- Tag: both
- Evidence: source
- Boundary: no
- What: Double-click a value, press F2 (Enter on macOS), or choose Set Value to type a new value in place, when the debugger supports setting variables and the value is not read-only or lazy.
- Commands: debug.setVariable
- Menus: DebugVariablesContext (group 3_modification, order 10)
- Keys: F2 (Enter on macOS, when the Variables view is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\variablesView.ts:211-238, 495-503, 670-681; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:866-886; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:177-181; C bundle (debug.setVariable, NLS "Set Value" and "Type new variable value")

### SB-5.13 Copy a value, an expression or the watch list
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a variable or watch row to copy its value or a path that can be pasted as an expression, copy a watch expression's text, or copy every watch as "name: value" lines, and use Ctrl+C on a focused row.
- Commands: workbench.debug.viewlet.action.copyValue; debug.copyEvaluatePath; debug.copyWatchExpression; workbench.debug.viewlet.action.copyAllWatchExpressions
- Menus: DebugVariablesContext (group 5_cutcopypaste, orders 10 and 20); DebugWatchContext (group 3_modification, orders 20 to 50, Copy All at 45)
- Keys: Ctrl+C (Cmd+C on macOS, Variables or Watch focused and no input open); Ctrl+Alt+C (Cmd+Alt+C on macOS, copy watch expression, Watch focused)
- Settings: none found
- Cursor: changed (Copy All is missing, no workbench.debug.viewlet.action.copyAllWatchExpressions id; the other three commands are present)
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:176-224; V\src\vs\workbench\contrib\debug\browser\variablesView.ts:683-732, 810-823; V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:660-720; C bundle (copyValue, copyEvaluatePath and copyWatchExpression ids, NLS "Copy as Expression" and "Copy Expression", no copyAllWatchExpressions id)

### SB-5.14 Break when a variable's value changes or is read
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a variable or watch row and choose Break on Value Change, Read or Access (only the ones the debugger offers) to add a data breakpoint that stops when that value is written or read.
- Commands: debug.breakWhenValueChanges; debug.breakWhenValueIsAccessed; debug.breakWhenValueIsRead
- Menus: DebugVariablesContext (group z_commands, orders 200 to 220); DebugWatchContext (group z_commands, orders 200 to 220)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:917-946; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:187-211; V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:410-440; C bundle (the three ids with 1 hit each, NLS "Break on Value Change", "Break on Value Access", "Break on Value Read")

### SB-5.15 View a variable's binary data
- Tag: both
- Evidence: source
- Boundary: no
- What: A small inline icon on a variable or watch row that points to memory opens its bytes in a hex view, and the first use asks to install the Hex Editor extension (the install itself is the extension system, left out).
- Commands: workbench.debug.viewlet.action.viewMemory
- Menus: DebugVariablesContext (group inline, order 15); DebugWatchContext (group inline, order 10)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\variablesView.ts:734-808; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:183, 205; C bundle (workbench.debug.viewlet.action.viewMemory, ms-vscode.hexeditor, NLS "View Binary Data")

### SB-5.16 Notebook Variables view
- Tag: both
- Evidence: source
- Boundary: no
- What: An experimental "Notebook Variables" view joins the Run and Debug container when the setting is on (or the REPL editor is active) and the notebook's kernel can list variables, and a right-click on a row offers Copy Value.
- Commands: notebook.openVariablesView; workbench.debug.viewlet.action.copyWorkspaceVariableValue
- Menus: NotebookVariablesContext
- Keys: none found
- Settings: notebook.variablesView
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariables.ts:30-100; V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariablesView.ts:35-135; V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariableCommands.ts:17-45; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts:1276-1280; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:213; C bundle (view id workbench.notebook.variables, setting notebook.variablesView, notebook.openVariablesView, copyWorkspaceVariableValue, NLS "Notebook Variables")

### SB-5.17 Add a watch expression
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the + (Add Expression) button in the Watch title, double-click empty space in the view, or right-click a variable and choose Add to Watch, then type an expression in place; every watch is evaluated again at each stop.
- Commands: workbench.debug.viewlet.action.addWatchExpression; debug.addToWatchExpressions; workbench.debug.action.focusWatchView; workbench.debug.watchExpressionsView
- Menus: ViewTitle (group navigation, view Watch); DebugWatchContext (group 3_modification, order 10); DebugVariablesContext (group z_commands, order 100)
- Keys: none found
- Settings: accessibility.debugWatchVariableAnnouncements
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:200-216, 611-634; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:185, 201, 471; C bundle (workbench.debug.viewlet.action.addWatchExpression, debug.addToWatchExpressions, focus command with 3 hits, NLS "Add Expression", "Add to Watch", "Expression to watch")

### SB-5.18 Edit, remove or reorder watch expressions
- Tag: both
- Evidence: source
- Boundary: no
- What: In Watch, press F2 (Enter on macOS) or choose Edit Expression to rename a watch, choose Set Value to change a result, remove one with its X or the Delete key, remove all from the title, drag rows to reorder them, and collapse all.
- Commands: debug.renameWatchExpression; debug.setWatchExpression; debug.removeWatchExpression; workbench.debug.viewlet.action.removeAllWatchExpressions; watch.collapse
- Menus: DebugWatchContext (Edit Expression order 20, Set Value order 30, Remove Expression inline order 20, Remove All Expressions group z_commands order 20); ViewTitle (Remove All Expressions order 20, Collapse All order 30, group navigation, view Watch)
- Keys: F2 (Enter on macOS, when the Watch view is focused); Delete (Cmd+Backspace on macOS, Watch focused and no input open)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\watchExpressionsView.ts:200-216, 520-584, 586-657; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:833-913; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:200-211; C bundle (rename, set and remove watch ids with 1 hit each, watch.collapse, DebugModel moveWatchExpression for drag reorder, NLS "Edit Expression", "Remove Expression", "Remove All Expressions")

### SB-5.19 Browse sessions, threads and stack frames
- Tag: both
- Evidence: source
- Boundary: no
- What: The Call Stack view lists each debug session with its state, its threads (Running or "Paused on <reason>") and the frames of a paused thread with file and line; clicking a frame opens its source and points Variables and Watch at it, library frames fold into "Show N More" rows, long stacks end in "Load More Stack Frames", and a Collapse All button sits in the title.
- Commands: workbench.debug.action.focusCallStackView; callStack.collapse; workbench.action.debug.callStackTop; workbench.action.debug.callStackBottom; workbench.action.debug.callStackUp; workbench.action.debug.callStackDown
- Menus: ViewTitle (Collapse All, group navigation, order 10, view Call Stack); CommandPalette (the four Navigate commands, shown when stopped)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\callStackView.ts:150-300, 830-940, 1015-1045, 1106-1124; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:406-436; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:147-152, 471; C bundle (view id workbench.debug.callStackView, callStack.collapse, the four callStack navigate ids, NLS "Paused on {0}", "Show {0} More: {1}", "Show {0} More Stack Frames", "Load More Stack Frames", "Debug Call Stack")

### SB-5.20 Control a session or thread from its row
- Tag: both
- Evidence: source
- Boundary: no
- What: Hover a session row, or a thread row (or a session that has one thread), to get inline Pause or Continue, Step Over, Step Into, Step Out, Restart and Stop or Disconnect buttons, and right-click for Disconnect and Suspend and Terminate Thread.
- Commands: workbench.action.debug.pause; workbench.action.debug.continue; workbench.action.debug.stepOver; workbench.action.debug.stepInto; workbench.action.debug.stepOut; workbench.action.debug.restart; workbench.action.debug.stop; workbench.action.debug.disconnect; workbench.action.debug.disconnectAndSuspend; workbench.action.debug.terminateThread
- Menus: DebugCallStackContext (group inline, orders 10 to 60; menu groups 3_modification and termination)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\callStackView.ts:1128-1146; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:164-182; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:360-366, 648-667; C bundle (menu id DebugCallStackContext with 20 hits, the ten command ids, NLS "Disconnect and Suspend", "Terminate Thread")

### SB-5.21 Restart a stack frame
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a stack frame, or use the small hover button on it, and choose Restart Frame to re-run the function from its start, when the debugger supports it for that frame.
- Commands: workbench.action.debug.restartFrame
- Menus: DebugCallStackContext (order 10, shown for stack frames when restart is supported); the hover button is built in the row renderer, not a menu item
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:181; V\src\vs\workbench\contrib\debug\browser\callStackView.ts:770-790; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:668-683; C bundle (workbench.action.debug.restartFrame, debug.callStack.restartFrame action, NLS "Restart Frame")

### SB-5.22 Copy the call stack
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a stack frame and choose Copy Call Stack to put the whole stack of that thread on the clipboard as text.
- Commands: debug.copyStackTrace
- Menus: DebugCallStackContext (group 3_modification, order 20, for stack frames)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:182; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:327-340; C bundle (debug.copyStackTrace, NLS "Copy Call Stack")

### SB-5.23 See and open breakpoints
- Tag: both
- Evidence: source
- Boundary: no
- What: The Breakpoints view lists exception filters first, then function, data, instruction and line breakpoints (file name, folder and a line badge), hover text gives condition, hit count, log message or why one is unverified, the breakpoint that just stopped the program is highlighted, a header warning appears when the debugger left some unverified, and a click opens the code (Ctrl+Enter opens it to the side).
- Commands: workbench.debug.action.focusBreakpointsView; debug.openBreakpointToSide
- Menus: none found
- Keys: Ctrl+Enter (Cmd+Enter on macOS) and Alt+Enter (when the Breakpoints view is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:178-300, 380-500, 506-560, 895-950, 1800-1950; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:1060-1078; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:472; C bundle (view id workbench.debug.breakPointsView, debug.openBreakpointToSide, NLS "Unverified Breakpoint", "Hit after breakpoint", "Instruction breakpoint at address", "Function breakpoints are not supported by this debug type")

### SB-5.24 Enable, disable or skip breakpoints
- Tag: both
- Evidence: source
- Boundary: no
- What: Tick a breakpoint's checkbox or press Space to turn it on or off, use Enable All and Disable All, or press the title button Toggle Activate Breakpoints to skip every breakpoint without deleting any; Reapply All Breakpoints re-sends them to a running session.
- Commands: workbench.debug.viewlet.action.enableAllBreakpoints; workbench.debug.viewlet.action.disableAllBreakpoints; workbench.debug.viewlet.action.toggleBreakpointsActivatedAction; workbench.debug.viewlet.action.reapplyBreakpointsAction; debug.toggleBreakpoint
- Menus: ViewTitle (Toggle Activate Breakpoints, group navigation, order 20, view Breakpoints); DebugBreakpointsContext (group z_commands: Enable All order 10, Disable All order 20, Reapply All order 30); MenubarDebugMenu (group 5_breakpoints, Enable All order 1, Disable All order 2)
- Keys: Space (when the Breakpoints view is focused and no input has focus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:280-292, 2149-2170, 2243-2322; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:790-807; C bundle (the four viewlet action ids with 1 hit each, debug.toggleBreakpoint, NLS "Toggle Activate Breakpoints", "Reapply All Breakpoints", "Enable All Breakpoints", "Disable All Breakpoints")

### SB-5.25 Remove breakpoints
- Tag: both
- Evidence: source
- Boundary: no
- What: Remove a breakpoint with the X on its row, the Delete key, a middle-click or Remove Breakpoint in its menu, or clear every kind at once with Remove All Breakpoints.
- Commands: workbench.debug.viewlet.action.removeBreakpoint; debug.removeBreakpoint; workbench.debug.viewlet.action.removeAllBreakpoints
- Menus: DebugBreakpointsContext (Remove Breakpoint in group 3_modification and inline; Remove All Breakpoints in group 3_modification, order 20); ViewTitle (Remove All Breakpoints, group navigation, order 30, view Breakpoints); MenubarDebugMenu (group 5_breakpoints, order 3)
- Keys: Delete (Cmd+Backspace on macOS, when the Breakpoints view is focused and no condition box has focus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:242-255, 2171-2241; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:947-970; C bundle (workbench.debug.viewlet.action.removeBreakpoint and removeAllBreakpoints, debug.removeBreakpoint, NLS "Remove Breakpoint", "Remove All Breakpoints")

### SB-5.26 Edit a breakpoint's condition, hit count or mode
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a breakpoint, or press the pencil on its row, to edit its condition, hit count or mode in an inline box (a line breakpoint opens the editor's breakpoint box instead, see ED-11); function breakpoints get Edit Function Condition and Edit Hit Count, data breakpoints get Edit Hit Count, and debuggers that define modes add a mode pick.
- Commands: debug.editBreakpoint; debug.editFunctionBreakpoint; debug.editFunctionBreakpointHitCount; debug.editBreakpointMode
- Menus: DebugBreakpointsContext (group navigation, orders 10 to 20; Edit Condition also inline)
- Keys: none found
- Settings: none found
- Cursor: changed (the mode command is titled "Manual Mode..." where stock says "Edit Mode..."; the other three commands and their labels are present)
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:2349-2486; C bundle (debug.editBreakpointMode registration with title NLS entry "Manual Mode...", "Select Breakpoint Mode", "Edit Condition...", "Edit Hit Count...", "Edit Function Condition...")

### SB-5.27 Add a function breakpoint
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the function-breakpoint button in the Breakpoints title (or use New Breakpoint then Function Breakpoint in the Run menu) and type a function name in the inline box that says "Function to break on"; double-click a function row to rename it.
- Commands: workbench.debug.viewlet.action.addFunctionBreakpointAction
- Menus: ViewTitle (group navigation, order 10, view Breakpoints); MenubarNewBreakpointMenu (group 1_breakpoints, order 3); CommandPalette (f1)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:270-280, 1480-1500, 1953-1983; C bundle (workbench.debug.viewlet.action.addFunctionBreakpointAction, NLS "Add Function Breakpoint", "Function to break on")

### SB-5.28 Add a data breakpoint at a memory address
- Tag: both
- Evidence: source
- Boundary: no
- What: When the debugger can break on bytes, a title button (or New Breakpoint then Data Breakpoint in the Run menu) asks for a range such as 0x1234 + 0xff and then an access type (read, write or access), and Edit Address on the row changes the range later.
- Commands: workbench.debug.viewlet.action.addDataBreakpointOnAddress; workbench.debug.viewlet.action.editDataBreakpointOnAddress
- Menus: ViewTitle (group navigation, order 11, view Breakpoints); MenubarNewBreakpointMenu (group 1_breakpoints, order 4); DebugBreakpointsContext (Edit Address, group navigation, order 15)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:1984-2147; C bundle (both ids with 1 hit, NLS "Add Data Breakpoint at Address", "Edit Address...", "Enter a memory range in which to break", "Select the access type to monitor")

### SB-5.29 Exception breakpoints
- Tag: both
- Evidence: source
- Boundary: no
- What: Exception filters offered by the debugger (for example all or uncaught exceptions) sit at the top of the Breakpoints list with checkboxes, some take a condition typed in place, and a palette command toggles several at once.
- Commands: debug.toggleExceptionBreakpoints; debug.editBreakpoint
- Menus: CommandPalette (f1, category Debug); DebugBreakpointsContext (Edit Condition on exception rows)
- Keys: none found
- Settings: none found
- Cursor: changed (exception rows, the condition box and its labels are present; the Toggle Exception Breakpoints command and its label are missing)
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:508-520, 990-1060, 1660-1730; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:1080-1160; C bundle (NLS "Unverified Exception Breakpoint", "Expression condition: {0}", "Type exception breakpoint condition"; no debug.toggleExceptionBreakpoints id or "Toggle Exception Breakpoints" label)

### SB-5.30 Show breakpoints as a list or as a tree by file
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A title button flips the Breakpoints view between a flat list (file name, folder, line badge) and a tree grouped by file, where each file row has a tri-state checkbox and a "Remove Breakpoints in File" button and each line row shows that line's code.
- Commands: workbench.debug.viewlet.action.toggleBreakpointsPresentation
- Menus: ViewTitle (group navigation, order 10, view Breakpoints)
- Keys: none found
- Settings: debug.breakpointsView.presentation
- Cursor: missing
- Source: V\src\vs\workbench\contrib\debug\browser\breakpointsView.ts:135-139, 500-560, 740-830, 895-975, 2325-2347; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:641-646; C bundle (searched for the command id, the setting key and the label "Toggle Breakpoints View Presentation" and the setting text, none found)

### SB-5.31 Browse and open loaded scripts
- Tag: both
- Evidence: source
- Boundary: no
- What: The Loaded Scripts view (collapsed by default and shown only for debuggers that report scripts) lists every script the program loaded, grouped by session and folder, opens one read-only on click, narrows by name as you type, and has a Collapse All button; a palette command shows the same list as a quick pick.
- Commands: loadedScripts.collapse; workbench.action.debug.showLoadedScripts
- Menus: ViewTitle (Collapse All, group navigation, order 30, view Loaded Scripts); CommandPalette (Open Loaded Script, when in debug mode and the debugger supports it)
- Keys: none found
- Settings: debug.closeReadonlyTabsOnEnd
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\loadedScriptsView.ts:396-410, 506-520, 601-625, 763-805; V\src\vs\workbench\contrib\debug\common\loadedScriptsPicker.ts:1-108; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts:694-699; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:147, 476; C bundle (view id workbench.debug.loadedScriptsView, loadedScripts.collapse, workbench.action.debug.showLoadedScripts, NLS "Loaded Scripts", "Open Loaded Script...", "Debug Loaded Scripts")

## Handed off
- Debug Console (the REPL view, its title buttons, Select Debug Console, Select Debug Session and the "Debug Console" toggle in this view's "..." menu, id debug.toggleReplIgnoreFocus): PN-6. Cursor binds the console toggle to Ctrl+Shift+Alt+Y where stock uses Ctrl+Shift+Y.
- Gutter breakpoints and the breakpoint box (conditional, logpoint, triggered, inline breakpoint, next and previous breakpoint, Toggle Breakpoint): ED-11.
- Inline values, debug hover and its menu (DebugHoverContext), the exception widget, Run to Cursor, Jump to Cursor, Evaluate in Debug Console, Add to Watch from an editor selection, Show Debug Hover: ED-11.
- Floating debug toolbar (the default debug.toolBarLocation) and the editor title "Run or Debug..." split button: ED-11.
- Debug toolbar in the command center (debug.toolBarLocation commandCenter, experimental): TB-2.
- Disassembly view, its context menu, the "Open Disassembly View" row in the Call Stack menu, and instruction breakpoints set there: ED-11.
- Run menu rows (Start Debugging, Run Without Debugging, Stop, Restart, Open Configurations, Add Configuration, step rows, New Breakpoint submenu, enable, disable and remove all, Install Additional Debuggers): TB-1.
- Start Debugging quick pick (the "debug " prefix with Most Recent, contributed and configure rows, plus Select and Start Debugging), the Open Loaded Script pick and the Select Debug Session pick: QA-4.
- Debug rows in the Command Palette: QA-1.
- Debug status bar item and the colored status bar while debugging (debug.showInStatusBar, debug.enableStatusBarColor): SS-5 for the item and SS-2 for the color.
- Notebook toolbar "Variables" button (notebook.openVariablesView in NotebookToolbar, InteractiveToolbar and EditorTitle): ED-7.
- Confirm on window close with live sessions (debug.confirmOnExit), the pre-launch task error prompt (debug.onTaskErrors set to prompt) and the "Find extension" box shown when no debugger matches: DL-2 for the first, DL-1 for the other two.
- Debug buttons in the Test Explorer: SB-6. The call stack widget in Test Results (menu DebugCallStackToolbar): PN-6.

## Unclear
- Pass 1 for this group names `DebugCallStackToolbar` as the menu of the Call Stack row buttons. In source that menu belongs to the call stack widget used by the Test Results peek (callStackWidget.ts, testResultsViewContent.ts). The Call Stack view row buttons are in `DebugCallStackContext` with group inline, so SB-5.20 lists that menu.
- Cursor registers two commands that stock lacks, `workbench.action.debug.recordStepByStep` and `workbench.action.debug.stopRecording` (they walk a thread step by step and record each call stack). No menu item, key, button or setting was found for them in the bundle, so they look agent-driven and are not listed as features.
- Cursor also has `cursor.configureDebugTimeout` ("Configure: Debug Timeout Prevention", palette only, picks Local Only, Always or Never, setting `cursor.debug.timeoutPrevention`). I could not tell from the bundle whether it affects the Run and Debug UI, so it is not listed.
- "Visualize Variable..." and "Remove Visualizer" rows in Variables and Watch only appear when an extension supplies a visualizer, and no built-in provider exists in V, so they are not listed. The "Add to Chat" rows on variables, scopes and watches are chat integration and left out.
- The JavaScript debug views were left out as instructed. The built-in extensions `debug-auto-launch` and `debug-server-ready` exist in V and C but are not in the scope list and were not examined.
- Run and Debug accessibility help (runAndDebugAccessibilityHelp.ts) and the Touch Bar debug buttons (macOS only) have no catalog group and were not listed.
- N (1.144) differs in one visible way: Step Out is hidden unless the thread has more than one stack frame (context key CONTEXT_THREAD_HAS_MULTIPLE_STACK_FRAMES) in the toolbar, the Call Stack row, the Run menu and the palette. It is a behavior change and not a new feature, so no feature carries `newer-than-1.128`.
- The Cursor picker (SB-5.4) and other Cursor findings come from reading bundle fragments and the NLS message list, never from a running window. NLS entries are indexed, so a missing label means the string is absent from the message list.
- The count of 31 is above the usual guide because this group covers the start controls, the launch configuration UI and five stock views plus the notebook view.

