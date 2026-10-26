Group: SB-5 Run and Debug. Pass: 3. Date: 2026-10-25.

### SB-5.1
1. Opening Run and Debug shows a side bar with the Variables, Watch, Call Stack and Breakpoints views stacked, plus Loaded Scripts when the debugger reports scripts.
2. Pressing `Ctrl+Shift+D` (`Cmd+Shift+D` on macOS) runs `workbench.view.debug` and shows the Run and Debug side bar.
3. Choosing Run from the View menu shows the Run and Debug side bar.
4. Clicking the Run and Debug icon in the activity bar shows the Run and Debug side bar.

### SB-5.2
1. When a breakpoint is hit during a debug session, the Run and Debug view opens on its own by default.
2. Setting `debug.openDebug` chooses when the view opens, with values `neverOpen`, `openOnSessionStart`, `openOnFirstSessionStart` and `openOnDebugBreak` (default).
3. A session started with Run Without Debugging never opens the view when it starts, whatever `debug.openDebug` says.
4. Setting `debug.openExplorerOnEnd` turned on brings the Explorer back when the last debug session ends and the side bar is visible, and it is off by default.

### SB-5.3
1. A play button in the Run and Debug view title starts the selected launch configuration, and its tooltip shows the keyboard shortcut.
2. Pressing `F5` starts debugging the selected configuration when a debugger is available and no session is running, `Ctrl+F5` runs it without the debugger, and the Start Debugging and Run Without Debugging rows in the Run menu do the same.
3. When no debugger extension is installed, `F5` and `Ctrl+F5` open the Run and Debug view instead of starting a session.
4. Before any launch configuration exists, and when no folder is open, the play button and the gear next to it are hidden.
5. Setting `debug.hideLauncherWhileDebugging` hides the play button while a session is active, and only matters when `debug.toolBarLocation` is not `docked`.
6. When the debug toolbar is docked or the play button is hidden, the "..." menu in the view title offers Start Additional Session.
7. Setting `debug.saveBeforeStart` chooses which editors are saved before a session starts, with values `allEditorsInActiveGroup` (default), `nonUntitledEditorsInActiveGroup` and `none`.
8. Setting `debug.onTaskErrors` chooses what happens when a `preLaunchTask` fails, with values `debugAnyway`, `showErrors`, `prompt` (default) and `abort`.
9. Setting `debug.hideSlowPreLaunchWarning` turned on hides the warning shown when a `preLaunchTask` has been running for a while, and it is off by default.
10. In Cursor the Run and Debug container has a taller second title row, 70 px high, that holds the play button and the configuration picker. [cursor only]

### SB-5.4
1. The dropdown beside the play button shows the name of the selected launch configuration, or "No Configurations" when there are none.
2. Opening the dropdown lists every launch configuration and compound from all launch scopes in launch.json presentation order, with hidden ones left out.
3. The list also shows up to three recently used dynamic configurations and one "Provider..." row for each dynamic configuration provider.
4. The last rows of the list are "Add Configuration..." (or "Add Config (folder)..." in a multi-folder workspace), one per folder.
5. Choosing a row only selects that configuration, and the play button is what starts it.
6. Pressing Right Arrow on the play button moves focus to the dropdown, and Left Arrow moves it back.
7. In Cursor the dropdown is an older plain select box with separator rows, with no group headings and no "Search configurations" filter box. [cursor only]

### SB-5.5
1. Clicking the gear opens the launch.json of the selected configuration's folder, scrolled to the selected entry.
2. With no configuration selected, the gear asks which workspace folder to create the launch.json in, or whether to add it to the workspace config file.
3. When the selected configuration lives in the workspace file or in the `launch` setting of user settings, the gear opens that location instead of a launch.json.
4. When no launch.json exists yet, the gear shows a "Select debugger" list with Suggested debuggers, "More X options..." and "Install an extension for X...", and picking a debugger builds a starter file.
5. The Run menu row Open Configurations and the Command Palette entry for `workbench.action.debug.configure` open the same launch.json.
6. While a session runs with a docked toolbar, the view title shows a second gear entry.

### SB-5.6
1. Choosing Add Configuration... inserts a new entry at the top of the `configurations` array in launch.json.
2. Right after the new entry is inserted, the snippet suggestions list opens so the person can pick a configuration template.
3. An Add Configuration... button shows inside the editor only when a text editor has `.vscode/launch.json` open.
4. The last row of the configuration dropdown, the Run menu row Add Configuration... and the Debug entry in the Command Palette all start the same flow.
5. In a multi-folder workspace the dropdown has one "Add Config (folder)..." row for each folder, and each adds to that folder's launch.json.

### SB-5.7
1. A `compounds` entry in launch.json shows up in the configuration dropdown like any other configuration.
2. Selecting a compound and pressing play starts all the sessions it lists together.
3. A compound lists its sessions by configuration name, or by name and folder pair in a multi-folder workspace.
4. The `stopAll` option on a compound makes manually stopping one session stop all of them.
5. The `preLaunchTask` option on a compound runs a task before any of its sessions start.
6. The `presentation` option on a compound sets its group, order and hidden state in the dropdown.
7. Starting a compound shows an error when a named configuration cannot be found or the name is ambiguous.
8. Setting `launch` in user settings or the workspace file can hold compounds, and they appear in the dropdown too.

### SB-5.8
1. Until a configuration is selected or a session has run, the Run and Debug container shows a "Run" welcome view instead of the Variables, Watch, Call Stack and Loaded Scripts views.
2. The Breakpoints view stays beside the welcome view only when some breakpoints exist.
3. The "Open a file" link in the welcome view, which says "which can be debugged or run", opens the file picker.
4. The "Run and Debug" button in the welcome view starts debugging.
5. The "create a launch.json file" link opens the configure flow, and it reads "open a folder" and opens the folder picker when no folder is open.
6. When every debug extension is disabled, the welcome view shows the plain-text notice "All debug extensions are disabled."

### SB-5.9
1. With `debug.toolBarLocation` set to `docked`, the Run and Debug title replaces the play button with the debug toolbar while a session is active, and the play button returns when no session is active.
2. Setting `debug.toolBarLocation` chooses where the debug toolbar sits, with values `floating` (default), `docked`, `commandCenter` (experimental) and `hidden`.
3. The docked toolbar shows Continue (or Pause while the program runs), Step Over, Step Into, Step Out, Restart and Stop buttons in that order.
4. The Stop button has a dropdown with Disconnect and Disconnect and Suspend.
5. For debuggers that support them, Step Back and Reverse buttons also appear in the docked toolbar.
6. Pressing `F5` continues when stopped, `F6` pauses when running, `F10` steps over, `F11` steps into, `Shift+F11` steps out, `Shift+F5` stops (or disconnects an attach session) and `Ctrl+Shift+F5` restarts, while debugging.

### SB-5.10
1. When more than one debug session runs, a "Debug Session" dropdown appears in the docked toolbar in the view title, and it is absent with a single session.
2. Picking a session from the dropdown changes which session the Variables, Watch, Call Stack and Breakpoints views show.
3. Child sessions are listed indented under their parent in the dropdown.
4. Setting `debug.showSubSessionsInToolBar` chooses whether sub-sessions appear in the toolbar, with values on and off (default off), and with it off, Stop on a sub-session also stops its parent.

### SB-5.11
1. The Variables view shows the scopes of the focused stack frame, such as Locals and Globals, as an expandable tree of variables.
2. When the program stops, the first scope that is cheap to compute opens by itself.
3. Each stack frame remembers which variable rows were expanded when you come back to it.
4. A lazy value, such as a getter, shows an eye icon with the tooltip "Click to expand", and clicking it resolves the value.
5. Setting `debug.autoExpandLazyVariables` chooses whether lazy values resolve by themselves, with values `auto` (default, only in screen reader optimized mode), `on` and `off`.
6. Setting `debug.showVariableTypes` turned on shows each variable's type next to its value, and it is off by default.
7. A Collapse All button in the Variables title collapses every open node.
8. The Command Palette command `workbench.debug.action.focusVariablesView` moves keyboard focus into the Variables view.

### SB-5.12
1. Double-clicking a variable's value turns it into an input where a new value can be typed in place.
2. Pressing `F2` (`Enter` on macOS) on a focused variable row opens the same input.
3. Choosing Set Value in a variable's right-click menu opens the same input.
4. Set Value is missing for a row when the debugger cannot set variables or the value is read-only or lazy.

### SB-5.13
1. Choosing Copy Value in a variable or watch row's right-click menu copies the value to the clipboard.
2. Choosing Copy as Expression in a variable's right-click menu copies a path that can be pasted into code as an expression.
3. Choosing Copy Expression in a watch row's right-click menu copies the watch expression text.
4. Choosing Copy All in the watch right-click menu copies every watch as lines of "name: value". [stock only]
5. Pressing `Ctrl+C` (`Cmd+C` on macOS) on a focused Variables or Watch row copies it when no input box is open.
6. Pressing `Ctrl+Alt+C` (`Cmd+Alt+C` on macOS) on a focused watch row copies the watch expression.

### SB-5.14
1. Choosing Break on Value Change in a variable's right-click menu adds a data breakpoint that stops the program when that value is written.
2. Choosing Break on Value Read or Break on Value Access adds a data breakpoint that stops when the value is read or accessed.
3. Each Break on Value row appears only when the debugger offers that kind of data breakpoint for that value.
4. The same Break on Value rows appear in a watch row's right-click menu.
5. A data breakpoint added this way shows up in the Breakpoints view.

### SB-5.15
1. A small inline icon that points to memory appears on a variable or watch row only while debugging and only when the debugger can show that value's bytes.
2. Clicking the icon opens the value's bytes in a hex view in the editor area, and it reuses an already open view of the same bytes.
3. The first click asks to install the Hex Editor extension, with the message "Inspecting binary data requires this extension", before the bytes open.

### SB-5.16
1. An experimental "Notebook Variables" view joins the Run and Debug container when a notebook is open whose selected kernel can list variables.
2. Setting `notebook.variablesView` turns the view on or off, and it is off by default.
3. The view also appears when the REPL editor is the active editor, whatever `notebook.variablesView` says.
4. Right-clicking a variable row in the view offers Copy Value, which copies that variable's value.

### SB-5.17
1. Clicking the + (Add Expression) button in the Watch title opens an input in the list where an expression can be typed to add it.
2. Double-clicking empty space in the Watch view opens the same input.
3. Choosing Add Expression in a watch row's right-click menu opens the same input.
4. Choosing Add to Watch in a variable's right-click menu adds that variable's expression without typing, and only shows when the variable has an expression.
5. Every watch expression is evaluated again each time the program stops.
6. The Command Palette command `workbench.debug.action.focusWatchView` moves keyboard focus into the Watch view.

### SB-5.18
1. Pressing `F2` (`Enter` on macOS) on a watch row, or choosing Edit Expression in its right-click menu, turns the expression into an input for renaming.
2. Choosing Set Value in a watch row's right-click menu lets the person type a new value for that watch's result.
3. Clicking the X on a watch row, or pressing `Delete` (`Cmd+Backspace` on macOS) on a focused row with no input open, removes that watch.
4. Remove All Expressions in the Watch title or right-click menu removes every watch.
5. Dragging a watch row up or down changes the order of the watch list.
6. A Collapse All button in the Watch title collapses every open node.

### SB-5.19
1. The Call Stack view lists each debug session as a top-level row that shows the session's state.
2. Each thread under a session shows Running, or "Paused on" and the reason such as a breakpoint or an exception.
3. A paused thread lists its stack frames, each with the function name, file and line.
4. Clicking a stack frame opens its source file at that line and points the Variables and Watch views at that frame.
5. Library frames fold into a single "Show N More" row, and clicking it reveals them.
6. A long stack ends in a "Load More Stack Frames" row that fetches more frames when clicked.
7. A Collapse All button in the Call Stack title collapses every open node.
8. While stopped, the Command Palette offers commands that move the selected frame to the top, bottom, up or down of the call stack, and `workbench.debug.action.focusCallStackView` moves keyboard focus into the view.

### SB-5.20
1. Hovering a thread row, or a session row that has only one thread, shows inline buttons for Pause or Continue, Step Over, Step Into, Step Out, Restart and Stop or Disconnect.
2. A running thread offers Pause and a stopped thread offers Continue, and the three step buttons are disabled unless the thread is stopped.
3. Right-clicking a session row offers Restart, Disconnect, Stop and, when the debugger supports it, Disconnect and Suspend.
4. Right-clicking a thread row offers Terminate Thread when the debugger supports it.

### SB-5.21
1. Choosing Restart Frame in a stack frame's right-click menu re-runs that function from its start.
2. Hovering a stack frame shows a small button that does the same thing as Restart Frame.
3. Restart Frame and the hover button are available only when the debugger supports restarting that frame.

### SB-5.22
1. Choosing Copy Call Stack in a stack frame's right-click menu puts every frame of that frame's thread on the clipboard as text, one frame per line.
2. Copy Call Stack appears only on stack frame rows, not on session or thread rows.

### SB-5.23
1. The Breakpoints view lists exception filters first, then function breakpoints, data breakpoints, instruction breakpoints and line breakpoints.
2. A line breakpoint row shows its file name, its folder and a badge with the line number.
3. Hovering a breakpoint row shows its condition, hit count or log message, or the reason the debugger left it unverified.
4. The breakpoint that just stopped the program is highlighted in the list.
5. When the debugger left some breakpoints unverified, a warning appears in the view header.
6. Clicking a breakpoint row opens its code, and pressing `Ctrl+Enter` (`Cmd+Enter` on macOS) or `Alt+Enter` on a focused row opens it beside the current editor.
7. The Command Palette command `workbench.debug.action.focusBreakpointsView` moves keyboard focus into the Breakpoints view.

### SB-5.24
1. Ticking or clearing the checkbox on a breakpoint row turns that breakpoint on or off without removing it.
2. Pressing `Space` on a focused breakpoint row, with no input box focused, toggles that breakpoint.
3. Enable All Breakpoints and Disable All Breakpoints, in the Breakpoints right-click menu and the Run menu, switch every breakpoint on or off at once.
4. The Toggle Activate Breakpoints button in the Breakpoints title makes the debugger skip every breakpoint without deleting any, and pressing it again turns them back on.
5. Reapply All Breakpoints in the Breakpoints right-click menu sends every breakpoint again to a running session.

### SB-5.25
1. Clicking the X on a breakpoint row removes that breakpoint.
2. Pressing `Delete` (`Cmd+Backspace` on macOS) on a focused breakpoint row removes it, unless a condition box has focus.
3. Middle-clicking a breakpoint row removes it.
4. Choosing Remove Breakpoint in a breakpoint's right-click menu removes it.
5. Remove All Breakpoints, from the Breakpoints title, its right-click menu or the Run menu, clears every kind of breakpoint at once.

### SB-5.26
1. Choosing Edit Condition... in a breakpoint's right-click menu opens an inline box on the row to type the condition, hit count or mode.
2. A pencil button on a breakpoint row opens the same inline Edit Condition box.
3. Editing a line breakpoint opens the editor's breakpoint box instead of an inline box in the list.
4. Function breakpoints offer Edit Function Condition... and Edit Hit Count... in their right-click menu.
5. Data breakpoints offer Edit Hit Count... in their right-click menu.
6. When the debugger defines breakpoint modes, the right-click menu adds a mode row that opens a "Select Breakpoint Mode" pick.
7. In Cursor the breakpoint mode row is titled "Manual Mode..." where stock VS Code says "Edit Mode...". [cursor only]

### SB-5.27
1. Clicking the function breakpoint button in the Breakpoints title opens an inline box with the placeholder "Function to break on", and typing a name adds the function breakpoint.
2. Choosing New Breakpoint then Function Breakpoint in the Run menu, or Add Function Breakpoint in the Command Palette, opens the same inline box.
3. Double-clicking a function breakpoint row turns its name into an input for renaming.

### SB-5.28
1. When the debugger can break on bytes, a button in the Breakpoints title, or New Breakpoint then Data Breakpoint in the Run menu, asks for a memory range to watch, written like `0x1234 + 0xff`.
2. After the range is entered, a pick asks which access type to watch, with read, write or access.
3. The new data breakpoint appears in the Breakpoints list as a data breakpoint row.
4. Choosing Edit Address... in the data breakpoint's right-click menu lets the person change the range later.

### SB-5.29
1. Exception filters offered by the debugger, such as all exceptions or uncaught exceptions, sit at the top of the Breakpoints list, each with a checkbox.
2. An exception filter that supports a condition shows Edit Condition... in its right-click menu and opens an inline box for typing it, with the placeholder "Break when expression evaluates to true" unless the debugger gives its own hint.
3. A condition set on an exception filter shows in its row beside the name, and hovering it reads "Expression condition:" followed by the condition.
4. A Toggle Exception Breakpoints command in the Command Palette turns several exception filters on or off in one pick. [stock only]

### SB-5.30
1. A button in the Breakpoints title flips the view between a flat list and a tree grouped by file. [stock only]
2. In list mode each row shows the file name, the folder and a line badge. [stock only]
3. In tree mode each file row has a checkbox that is checked, unchecked or partly checked depending on its breakpoints. [stock only]
4. In tree mode each file row shows a "Remove Breakpoints in File" button on hover. [stock only]
5. In tree mode each line row shows the code on that line. [stock only]
6. Setting `debug.breakpointsView.presentation` chooses how breakpoints are shown, with values `list` (default) and `tree`. [stock only]

### SB-5.31
1. The Loaded Scripts view lists every script the program has loaded, grouped by session and then by folder.
2. The view is collapsed by default and shows only for debuggers that report loaded scripts.
3. Clicking a script opens it read-only in the editor.
4. Typing while the Loaded Scripts tree is focused narrows the list to scripts whose names match.
5. A Collapse All button in the Loaded Scripts title collapses every open node.
6. The Open Loaded Script... command in the Command Palette, available while debugging when the debugger supports it, shows the same scripts as a quick pick.

## Unclear
- SB-5.4: the Cursor piece describes a plain select box with separator rows, and it comes only from bundle strings, never from a running window.
- SB-5.9 and SB-5.10: the `commandCenter` toolbar location is only named in the setting values here, and its look is owned by TB-2.
- SB-5.23: the feature block lists `workbench.debug.action.focusBreakpointsView` with no menu, key or title entry, so the focus piece is Command Palette only as far as the block shows.
- SB-5.3: this feature has 10 pieces after merging key and menu entry points, and it may be worth splitting launch settings (`debug.saveBeforeStart`, `debug.onTaskErrors`, `debug.hideSlowPreLaunchWarning`) into their own feature.
- SB-5.30: Cursor is missing, so every piece ends `[stock only]`, including the setting.
