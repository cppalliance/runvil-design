Group: PN-6 Debug Console and Test Results. Pass: 3. Date: 2026-10-25.

### PN-6.1
1. Pressing Ctrl+Shift+Y (`workbench.debug.action.toggleRepl`) opens the Debug Console tab in the panel when it is hidden and closes it when it is showing. [stock only]
2. In Cursor, pressing Ctrl+Shift+Alt+Y toggles the Debug Console tab, and Ctrl+Shift+Y is not the toggle key. [cursor only]
3. Choosing Debug Console in the View menu opens the Debug Console tab.
4. Choosing Debug Console in the "..." menu of the Run and Debug view opens the Debug Console tab.
5. Running the Command Palette entries for `workbench.debug.action.focusRepl` or `workbench.panel.repl.view.focus` brings up the Debug Console with keyboard focus inside it.
6. The Debug Console tab only appears in the panel tab bar while a debugger extension is active, and is absent otherwise.

### PN-6.2
1. Setting `debug.internalConsoleOptions` chooses when the Debug Console opens by itself, with values `neverOpen`, `openOnSessionStart` and `openOnFirstSessionStart`, default `openOnFirstSessionStart`.
2. With `neverOpen`, starting a debug session leaves the Debug Console closed.
3. With `openOnSessionStart`, the Debug Console opens every time a debug session starts.
4. With `openOnFirstSessionStart`, the Debug Console opens when the first session starts and not for later sessions.
5. Setting `debug.console.closeOnEnd`, off by default, makes the Debug Console close by itself when the debug session ends.
6. A launch configuration that sets its own `internalConsoleOptions` overrides the setting for that session.

### PN-6.3
1. Typing an expression in the input at the bottom of the console and pressing Enter (`repl.action.acceptInput`) runs it in the debugged program and adds the result to the console list.
2. A result that is an object shows a twisty that expands it into its child values.
3. The input grows taller to show several lines as the typed text gets longer.
4. With no active debug session the input is read only and shows the hint "Please start a debug session to evaluate expressions".

### PN-6.4
1. Pressing Up Arrow in an empty console input fills it with the previous expression, and pressing it again goes further back.
2. Pressing Down Arrow steps forward through the history toward the newest entry.
3. Pressing Alt+Up Arrow or Alt+Down Arrow moves through the history the same way as Up and Down.
4. While the suggest list is showing, Up and Down move through the suggestions and do not change the history.
5. The history holds up to 100 entries and drops the oldest first.
6. The history is saved per workspace, so earlier expressions come back after a reload.

### PN-6.5
1. Typing in the console input opens the editor suggest list with names from the debug adapter, after a dot or the adapter's trigger characters.
2. Earlier console input shows at the bottom of the suggest list.
3. Setting `debug.console.historySuggestions`, on by default, chooses whether earlier input is offered in the suggest list.
4. Setting `debug.console.acceptSuggestionOnEnter` chooses whether Enter accepts the highlighted suggestion, with values `off` and `on`, default `off`.
5. With `acceptSuggestionOnEnter` set to `off`, pressing Enter with the suggest list open runs the typed line instead of accepting a suggestion.

### PN-6.6
1. Output, warnings and errors from the debugged program appear as lines in the console list.
2. Each line is colored by its severity, so errors and warnings look different from plain output.
3. ANSI color codes in the program's text show as colors in the line.
4. Links inside a line, such as file paths and URLs, are clickable.

### PN-6.7
1. An output line that knows where it came from shows a small file name and line number label at its right.
2. Clicking the label opens that file at that line in the editor.
3. Hovering the label shows the full path of the file.
4. An output line that has no known origin shows no label.

### PN-6.8
1. Repeated identical output lines fold into one row with a number badge showing how many times the line occurred.
2. The badge count goes up as more identical lines arrive.
3. Setting `debug.console.collapseIdenticalLines`, on by default, chooses whether identical lines are folded, and when off each repeat shows as its own row.

### PN-6.9
1. Grouped output from the program, such as `console.group`, shows as a row with a twisty that expands and collapses the lines inside it.
2. Some groups open by themselves when they start, and others start collapsed.
3. Choosing Collapse All (`debug.collapseRepl`) in the console right-click menu folds every output group at once.

### PN-6.10
1. A filter box in the Debug Console header with the placeholder "Filter (e.g. text, !exclude, \escape)" hides output lines that do not match the typed text.
2. Separating terms with commas matches lines that contain any of the terms.
3. Starting a term with `!` excludes lines that match it.
4. Starting a term with a backslash matches that term literally.
5. A badge in the filter box says "Showing X of Y" while the filter hides some lines.
6. Typed input and evaluation results are never hidden by the filter.
7. The filter text and its history are kept per workspace.
8. Pressing Ctrl+F (`repl.action.filter`) in the console input moves focus to the filter box.

### PN-6.11
1. Clicking the search icon in the Debug Console header (`repl.action.find`) opens the find box for the console output.
2. Choosing the find entry in the console right-click menu opens the find box.
3. Pressing Ctrl+Alt+F while the console or its input is focused opens the find box.
4. Typing in the find box searches the console output list.

### PN-6.12
1. Setting `debug.console.wordWrap`, on by default, makes long output lines wrap onto several rows.
2. With `debug.console.wordWrap` off, long lines stay on one row and the list scrolls sideways.
3. Changing `debug.console.wordWrap` rebuilds the console list right away with the new wrapping.

### PN-6.13
1. Setting `debug.console.fontSize` sets the font size in pixels for the console list and input, default 14 (12 on macOS).
2. Setting `debug.console.fontFamily` sets the font family for the console list and input, default `default`.
3. Setting `debug.console.lineHeight` sets the line height in pixels for the console list and input, default 0.
4. A `debug.console.lineHeight` of 0 makes the line height follow from the font size.

### PN-6.14
1. Clicking the Clear Console icon in the Debug Console header removes all output from the current console.
2. Choosing Clear Console in the console right-click menu removes all output from the current console.
3. Running Clear Console (`workbench.debug.panel.action.clearReplAction`) from the Command Palette removes all output from the current console.
4. On macOS, pressing Cmd+K while the console view is focused clears the console.
5. Clearing the console of an ended session also removes that session from the console list.

### PN-6.15
1. Choosing Copy in the console right-click menu (`debug.replCopy`) copies the selected text, or the value of the clicked result when nothing is selected.
2. Choosing Copy All in the console right-click menu (`workbench.debug.action.copyAll`) copies all visible lines.
3. Choosing Paste in the console right-click menu (`debug.replPaste`) adds the clipboard text to the input, and only works while a debug session is active.
4. The command "Debug: Console Copy All" (`repl.action.copyAll`) copies all console lines while the input has focus.

### PN-6.16
1. When more than one debug session has its own console, a drop-down in the Debug Console header (`workbench.action.debug.selectRepl`) lists those consoles and picking one shows that session's console.
2. With only one console, the header shows no session drop-down.
3. Pressing Ctrl+PageDown (`workbench.action.debug.nextConsole`) shows the next session's console.
4. Pressing Ctrl+PageUp (`workbench.action.debug.prevConsole`) shows the previous session's console.
5. On macOS, pressing Shift+Cmd+] and Shift+Cmd+[ in the debug console shows the next and previous console.
6. Running Select Debug Console (`workbench.action.debug.selectDebugConsole`) from the Command Palette opens a list of all consoles, and choosing one shows it.

### PN-6.17
1. Setting `debug.console.maximumLines`, default 10000, caps how many output entries the console keeps. [stock only]
2. Once the console holds more entries than the cap, the oldest entries are dropped first as new output arrives. [stock only]
3. Lowering the value below the number of entries already shown trims the list down to the new cap when the next output arrives. [stock only]

### PN-6.18
1. The Test Results tab appears in the panel tab bar only once a test run has produced results.
2. Running Show Output (`testing.showMostRecentOutput`) from the Command Palette opens the Test Results view on the latest run.
3. Clicking the Show Output button in the Test Explorer header opens the Test Results view on the latest run.
4. Pressing Ctrl+; and then Ctrl+O opens the Test Results view on the latest run.
5. Running the focus command `workbench.panel.testResults.view.focus` brings up the Test Results view with keyboard focus in it.

### PN-6.19
1. Setting `testing.automaticallyOpenTestResults` chooses what opens when tests start, with values `neverOpen`, `openOnTestStart`, `openOnTestFailure` and `openExplorerOnTestStart`, default `openOnTestStart`.
2. With `openOnTestStart`, starting tests opens the Test Results view.
3. With `openOnTestFailure`, the Test Results view opens on any test failure and not when tests start.
4. With `openExplorerOnTestStart`, starting tests opens the Test Explorer instead of Test Results.
5. With `neverOpen`, starting tests opens neither testing view.
6. A value in the older setting `testing.openTesting` is carried over into `testing.automaticallyOpenTestResults`.

### PN-6.20
1. A tree pane beside the details pane lists the newest run, with each task as a row and its tests under it.
2. Every row has a status icon showing the state of that task, test or message.
3. The names of a test's parent tests show dimmed after the test's own label.
4. Tests are sorted by their state.
5. A test's failure messages show as child rows, and a long message is cut off with a "+ N more lines" label.
6. A chain of rows that each have a single child folds into one row.

### PN-6.21
1. Earlier runs sit in a collapsed "N older results" row under the newest run, reading "1 older result" when there is one.
2. Expanding the older results row lists the past runs.
3. Opening a past run shows its own tasks and tests in the tree.
4. The list keeps up to 128 results and drops the oldest beyond that.

### PN-6.22
1. Picking a message row, or a test that has a message, shows the failure text in the details pane under a header with the test name.
2. A message sent as plain text shows as colored plain text.
3. A message sent as markdown shows as rendered markdown.

### PN-6.23
1. When a failure carries an expected and an actual value, the details pane shows them in a side by side diff.
2. The two sides are labeled "Expected result" and "Actual result".
3. Line numbers show in the diff when either value spans several lines.

### PN-6.24
1. Choosing Show Result Output on a run or a test row streams its output into a read-only terminal view in the details pane.
2. The streamed output shows its colors, and links in it resolve from the test's folder.
3. A test with no output shows "The test case did not report any output."
4. A run with no output shows "The test run did not record any output."
5. An old run shows "Test output is only available for new test runs."

### PN-6.25
1. When a failure message has stack frames, the details pane lists them under the message with a file and line for each frame.
2. Clicking a stack frame opens that file at that line in the editor.
3. Stack frames can be folded and unfolded in the list.
4. The Collapse Stack Frames button (`testing.collapsePeekStack`) shows on the toolbar of the editor peek and folds every frame, and the Test Results panel header has no such button.

### PN-6.26
1. While a run is still going, hovering its row in the results tree shows a Cancel Test Run button.
2. When a run is not running, hovering its row shows Rerun Last Run and Debug Last Run buttons.
3. Clicking Rerun Last Run runs the same tests again, and clicking Debug Last Run runs them again under the debugger.
4. The Test Explorer header also shows Cancel Test Run while a run is going, with no rerun or debug buttons.
5. The Command Palette lists Cancel Test Run (`testing.cancelRun`), Rerun Last Run (`testing.reRunLastRun`), Debug Last Run (`testing.debugLastRun`) and Rerun Last Run with Coverage (`testing.coverageLastRun`).
6. Pressing Ctrl+; then Ctrl+X cancels, Ctrl+; then L reruns the last run, Ctrl+; then Ctrl+L debugs the last run, and Ctrl+; then Ctrl+Shift+L reruns it with coverage.

### PN-6.27
1. When a run has failed tests, its row also shows Rerun Failed Tests and Debug Failed Tests buttons on hover. [stock only]
2. A run with no failed tests shows neither of those buttons. [stock only]
3. Clicking Rerun Failed Tests runs only the failed tests of that run again, and Debug Failed Tests runs them under the debugger. [stock only]
4. The Command Palette lists Rerun Failed Tests from Last Run (`testing.reRunFailedFromLastRun`) and Debug Failed Tests from Last Run (`testing.debugFailedFromLastRun`). [stock only]

### PN-6.28
1. A test row shows a Run Test button, when the test can be run, that runs just that test.
2. A test row shows a Debug Test button, when the test can be debugged, that runs just that test under the debugger.
3. The details pane header shows a Rerun Test button (`testing.callStack.run`) that runs the test being shown again.
4. The details pane header shows a Debug Test button (`testing.callStack.debug`) that runs the test being shown again under the debugger.
5. When a whole run is shown in the details pane instead of a test, the header Rerun and Debug buttons act on the whole run.

### PN-6.29
1. The Go to Test button on a message row opens the test's source file at the test.
2. In stock VS Code, the Go to Test button also shows on test rows. [stock only]
3. In Cursor, the Go to Test button shows on message rows only. [cursor only]
4. The Go to Error button on a message row that has a location opens the failing line in the editor without moving keyboard focus.
5. Right-clicking a test or message row offers Reveal in Test Explorer, which shows that test in the Test Explorer.
6. Selecting a test row in the results tree also selects that test in the Test Explorer.

### PN-6.30
1. Clicking the Clear All Results icon in the Test Results header throws away every stored run.
2. Running Clear All Results (`testing.clearTestResults`) from the Command Palette throws away every stored run.
3. After results are cleared, the Test Results tab hides itself because there is nothing left to show.
4. Clear All Results is also a button on the title bar of the editor peek.

### PN-6.31
1. A Toggle Tree Position button in the Test Results header (`testing.toggleResultsViewLayout`) moves the run tree to the other side of the details pane. [stock only]
2. Setting `testing.resultsView.layout` chooses where the run tree sits, with values `treeRight` and `treeLeft`, default `treeRight`. [stock only]
3. Changing the layout, from the button or the setting, rearranges an open Test Results view right away. [stock only]

### PN-6.32
1. When a test extension offers follow-up actions for a failed message, the first one shows as a link under the message.
2. When there is more than one follow-up, the rest sit behind a "+N More..." link.
3. Clicking "+N More..." opens a pick list of the remaining follow-up actions, and choosing one runs it.
4. A failed message with no follow-up actions shows no links, because nothing built in supplies them.

## Unclear
- PN-6.33 shares its whole widget with the panel Test Results view, and the Collapse Stack Frames button and Clear All Results button on the peek title bar are counted in PN-6.25 piece 4 and PN-6.30 piece 4 and not repeated there.
- PN-6.6 stays at the level of the feature block for severity colors and link types, since the exact colors come from the theme and were not read.
