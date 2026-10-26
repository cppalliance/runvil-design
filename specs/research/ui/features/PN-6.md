Group: PN-6 Debug Console and Test Results. Section: Panel. Date: 2026-10-25. Pass: 2.
Features: 32.

### PN-6.1 Show or hide the Debug Console
- Tag: both
- Evidence: source
- Boundary: no
- What: Open, toggle or focus the Debug Console tab from a key, the View menu, the Command Palette or the Run and Debug view menu. The tab only exists while a debugger extension is active.
- Commands: workbench.debug.action.toggleRepl; workbench.debug.action.focusRepl; workbench.panel.repl.view.focus; debug.toggleReplIgnoreFocus; workbench.panel.repl; workbench.panel.repl.view
- Menus: MenubarViewMenu; Views (the "..." menu of the Run and Debug view, entry "Debug Console"); CommandPalette
- Keys: Ctrl+Shift+Y (workbench.debug.action.toggleRepl)
- Settings: none found
- Cursor: changed (toggle key is Ctrl+Shift+Alt+Y, stock is Ctrl+Shift+Y)
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\debug\browser\debugCommands.ts; V\src\vs\workbench\contrib\debug\browser\debugViewlet.ts; V\src\vs\workbench\services\views\browser\viewsService.ts; C bundle (workbench.debug.action.toggleRepl with keybindings CtrlCmd|Shift|Alt|KeyY)

### PN-6.2 Open and close the console automatically with a session
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether the Debug Console opens when a debug session starts (never, every session, or first session only) and whether it closes when the session ends. A launch configuration can override the open choice.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.internalConsoleOptions; debug.console.closeOnEnd
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\debug\browser\debugService.ts; V\src\vs\workbench\contrib\debug\common\debug.ts; C bundle (debug.internalConsoleOptions, debug.console.closeOnEnd)

### PN-6.3 Evaluate an expression in the console
- Tag: both
- Evidence: source
- Boundary: no
- What: Type an expression in the input at the bottom of the console and press Enter to run it in the debugged program. The result lands in the console list and objects expand with a twisty. The input grows to several lines, and is read only with the hint "Please start a debug session to evaluate expressions" when no session is active.
- Commands: repl.action.acceptInput
- Menus: none found
- Keys: Enter (when text input focus and in the debug console)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\replViewer.ts; C bundle (repl.action.acceptInput, repl-input-wrapper, replEvaluationResult)

### PN-6.4 Step through earlier input
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Up or Down in an empty console input to bring back earlier expressions. History holds up to 100 entries and is saved per workspace.
- Commands: history.showPrevious; history.showNext
- Menus: none found
- Keys: Up Arrow; Down Arrow; Alt+Up Arrow; Alt+Down Arrow (when the history input has focus and history navigation is enabled)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\platform\history\browser\contextScopedHistoryWidget.ts; C bundle (history.showPrevious, history.showNext, debug.repl.history)

### PN-6.5 Get completions while typing in the console
- Tag: both
- Evidence: source
- Boundary: no
- What: The input shows the editor suggest list with names from the debug adapter (after a dot or the adapter's trigger characters), plus earlier input at the bottom of the list. Enter only accepts a suggestion when the setting is on, because Enter also runs the line.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.historySuggestions; debug.console.acceptSuggestionOnEnter
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (supportsCompletionsRequest, completionTriggerCharacters, debug.console.historySuggestions, debug.console.acceptSuggestionOnEnter)

### PN-6.6 Read program output with severity colors and links
- Tag: both
- Evidence: source
- Boundary: no
- What: Output, warnings and errors from the debugged program appear as lines in the console list, colored by severity, with ANSI colors and clickable links in the text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\replViewer.ts; V\src\vs\workbench\contrib\debug\common\replModel.ts; C bundle (outputReplElement, wasANSI)

### PN-6.7 Open the source location of an output line
- Tag: both
- Evidence: source
- Boundary: no
- What: Output lines that know where they came from show a small file name and line label at the right. Click it to open that file at that line, hover it for the full path.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\replViewer.ts (SourceWidget); C bundle (value-and-source, sourceData)

### PN-6.8 Collapse identical lines with a count badge
- Tag: both
- Evidence: source
- Boundary: no
- What: Repeated identical output lines fold into one row with a number badge showing how many times it occurred.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.collapseIdenticalLines
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\replViewer.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (debug.console.collapseIdenticalLines, count-badge-wrapper)

### PN-6.9 Expand and collapse output groups
- Tag: both
- Evidence: source
- Boundary: no
- What: Grouped output from the program (console.group style) shows as rows you expand or collapse, and some groups open by themselves. Collapse All folds every group at once.
- Commands: debug.collapseRepl
- Menus: DebugConsoleContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\common\replModel.ts; C bundle (debug.collapseRepl, startGroup)

### PN-6.10 Filter the console output
- Tag: both
- Evidence: source
- Boundary: no
- What: A filter box in the Debug Console header hides output lines that do not match. Separate terms with commas, start a term with ! to exclude it, or with a backslash to match it literally. A badge says "Showing X of Y", and the filter text and its history are kept per workspace. Typed input and results are never hidden.
- Commands: repl.action.filter
- Menus: none found
- Keys: Ctrl+F (when text input focus in the console input)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\replFilter.ts; C bundle (repl.action.filter, debug.repl.filterValue, debug.repl.filterHistory)

### PN-6.11 Find text in the console
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the list's find box to search the console output, from the search icon in the header, the right-click menu or a key.
- Commands: repl.action.find; list.find.replInputFocus
- Menus: ViewTitle; DebugConsoleContext
- Keys: Ctrl+Alt+F (when in the debug console or the console view is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; C bundle (repl.action.find)

### PN-6.12 Wrap long lines
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn wrapping of long output lines on or off. Changing it rebuilds the list, and when it is off the list scrolls sideways.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.wordWrap
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (debug.console.wordWrap)

### PN-6.13 Set the console font and line height
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose the font size, font family and line height used in the console list and input. A line height of 0 works it out from the font size.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.fontSize; debug.console.fontFamily; debug.console.lineHeight
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts (ReplOptions); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (debug.console.fontSize, debug.console.fontFamily, debug.console.lineHeight)

### PN-6.14 Clear the console
- Tag: both
- Evidence: source
- Boundary: no
- What: Remove all output from the current console, from the header icon, the right-click menu or the Command Palette. Clearing an ended session also drops it from the console list.
- Commands: workbench.debug.panel.action.clearReplAction
- Menus: ViewTitle; DebugConsoleContext; CommandPalette
- Keys: Cmd+K on macOS only (when the console view is focused)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; C bundle (workbench.debug.panel.action.clearReplAction, Clear Console)

### PN-6.15 Copy console text and paste into the input
- Tag: both
- Evidence: source
- Boundary: no
- What: The console right-click menu has Copy (the selection, or the value of the clicked result), Copy All (all visible lines) and Paste (adds clipboard text to the input, only while a session is active). Copy All also exists as an editor command in the input.
- Commands: debug.replCopy; workbench.debug.action.copyAll; repl.action.copyAll; debug.replPaste
- Menus: DebugConsoleContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts; C bundle (debug.replCopy, workbench.debug.action.copyAll, repl.action.copyAll, debug.replPaste)

### PN-6.16 Switch the console between debug sessions
- Tag: both
- Evidence: source
- Boundary: no
- What: When more than one session has its own console, a drop-down in the header picks which session's console you see. Keys and commands move to the next or previous console, and a Command Palette entry opens a list of all consoles.
- Commands: workbench.action.debug.selectRepl; workbench.action.debug.nextConsole; workbench.action.debug.prevConsole; workbench.action.debug.selectDebugConsole
- Menus: ViewTitle; CommandPalette
- Keys: Ctrl+PageDown (next); Ctrl+PageUp (previous); Shift+Cmd+] and Shift+Cmd+[ on macOS (when in the debug console)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\repl.ts (SelectReplActionViewItem); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; C bundle (workbench.action.debug.selectRepl, workbench.action.debug.nextConsole, workbench.action.debug.prevConsole, workbench.action.debug.selectDebugConsole, multiSessionRepl)

### PN-6.17 Limit how many console lines are kept
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Cap the number of output entries the console keeps (10000 by default). The oldest entries are dropped first.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.console.maximumLines
- Cursor: missing
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\debug\common\replModel.ts; C bundle (zero hits for debug.console.maximumLines and maximumLines)

### PN-6.18 Open the Test Results view
- Tag: both
- Evidence: source
- Boundary: no
- What: The Test Results tab shows up once any test run has produced results. Open it with Show Output (also a button in the Test Explorer header) or the Focus command, and it jumps to the latest run.
- Commands: testing.showMostRecentOutput; workbench.panel.testResults.view.focus; workbench.panel.testResults
- Menus: ViewTitle (Test Explorer header, navigation group); CommandPalette
- Keys: Ctrl+; then Ctrl+O (testing.showMostRecentOutput)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts; V\src\vs\workbench\contrib\testing\browser\testingOutputPeek.ts (TestResultsView); V\src\vs\workbench\services\views\browser\viewsService.ts; C bundle (testing.showMostRecentOutput, workbench.panel.testResults.view, testing.hasAnyResults)

### PN-6.19 Open Test Results automatically
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose whether starting tests opens the Test Results view, the Test Explorer, or nothing, or whether it opens only on the first failure. Default opens Test Results when tests start.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: testing.automaticallyOpenTestResults; testing.openTesting
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testingProgressUiService.ts; V\src\vs\workbench\contrib\testing\common\configuration.ts; C bundle (testing.automaticallyOpenTestResults with openOnTestStart default, testing.openTesting migration)

### PN-6.20 See the latest run as a tree
- Tag: both
- Evidence: source
- Boundary: no
- What: A tree pane beside the details pane lists the newest run: each task with its tests under it, a status icon on every row, the parent test names shown dimmed after the label, tests sorted by state, and failure messages as child rows ("+ N more lines"). Chains with a single child fold into one row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; C bundle (test-output-peek-tree, testing-stdtree, Test Output Peek)

### PN-6.21 Browse older runs
- Tag: both
- Evidence: source
- Boundary: no
- What: Earlier runs sit in a collapsed "N older results" row under the newest run. Expand it to open any past run, with its own tasks and tests. Stock keeps up to 128 results.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts (OlderResultsElement); V\src\vs\workbench\contrib\testing\common\testResultStorage.ts; C bundle (older- element id in the results tree)

### PN-6.22 Read a failed test message
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick a message row (or a test with a message) and the details pane shows the failure text, as colored plain text or rendered markdown, under a header with the test name.
- Commands: none found
- Menus: TestMessageContent
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsOutput.ts (PlainTextMessagePeek, MarkdownTestMessagePeek); V\src\vs\workbench\contrib\testing\browser\testMessageColorizer.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsSubject.ts; C bundle (messageUri, markdown.showPreview)

### PN-6.23 Compare expected and actual values in a diff
- Tag: both
- Evidence: source
- Boundary: no
- What: When a failure carries an expected and an actual value, the details pane shows them in a side by side diff labeled "Expected result" and "Actual result", with line numbers when either value spans several lines.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsOutput.ts (DiffContentProvider); C bundle (expectedUri, actualUri, isDiffable)

### PN-6.24 Read test or run output in a terminal pane
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Show Result Output on a run or a test and the details pane streams its output in a read-only terminal view, with colors and links that resolve from the test's folder. Empty cases say "The test case did not report any output." or "The test run did not record any output." and old runs say output is only kept for new runs.
- Commands: none found
- Menus: TestPeekElement
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsOutput.ts (TerminalMessagePeek); V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts (the Show Result Output row action, built in code); C bundle (testing.outputPeek.showResultOutput, createDetachedTerminal, selectMarkedRange)

### PN-6.25 Read the stack trace under a failure
- Tag: both
- Evidence: source
- Boundary: no
- What: When a message has stack frames, the details pane lists them under the message with a file and line for each. Click a frame to open that spot, and fold or unfold frames. The Collapse Stack Frames button exists only on the editor peek toolbar, not on the panel view.
- Commands: testing.collapsePeekStack; callStackWidget.goToFile
- Menus: TestPeekTitle; DebugCallStackToolbar
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; V\src\vs\workbench\contrib\testing\browser\testingOutputPeek.ts; V\src\vs\workbench\contrib\debug\browser\callStackWidget.ts; C bundle (testing.collapsePeekStack, callStackWidget, test-output-call-stack-container)

### PN-6.26 Cancel, rerun or debug a whole run
- Tag: both
- Evidence: source
- Boundary: no
- What: A run row has hover buttons: Cancel Test Run while it is running, otherwise Rerun Last Run and Debug Last Run. The same actions are chords in the Command Palette, plus a Rerun Last Run with Coverage entry.
- Commands: testing.cancelRun; testing.reRunLastRun; testing.debugLastRun; testing.coverageLastRun
- Menus: TestPeekElement; CommandPalette; ViewTitle (Test Explorer header, cancel only)
- Keys: Ctrl+; then Ctrl+X (cancel); Ctrl+; then L (rerun last); Ctrl+; then Ctrl+L (debug last); Ctrl+; then Ctrl+Shift+L (coverage last)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts (TreeActionsProvider, row actions built in code); V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts; C bundle (testing.cancelRun, testing.reRunLastRun, testing.debugLastRun, testing.coverageLastRun, testing.outputPeek.rerun)

### PN-6.27 Rerun or debug only the failed tests of a run
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When a run has failed tests, its row also offers Rerun Failed Tests and Debug Failed Tests. The Command Palette has the matching "from Last Run" commands. The Test Explorer's own Rerun Failed Tests command (testing.reRunFailTests) is present in both products and belongs to SB-6.
- Commands: testing.reRunFailedFromLastRun; testing.debugFailedFromLastRun
- Menus: TestPeekElement; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; C bundle (zero hits for testing.reRunFailedFromLastRun, testing.debugFailedFromLastRun and the failed-test row actions)

### PN-6.28 Rerun or debug one test from its row or the details header
- Tag: both
- Evidence: source
- Boundary: no
- What: Test rows show Run Test and Debug Test buttons, and the details pane header has Rerun Test and Debug Test for the test being shown (or the whole run when a run is shown).
- Commands: testing.callStack.run; testing.callStack.debug; vscode.runTestsById
- Menus: TestCallStack; TestPeekElement
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; C bundle (testing.callStack.run, testing.callStack.debug, testing.outputPeek.runTest, testing.outputPeek.debugTest, vscode.runTestsById)

### PN-6.29 Jump from a result to the test, the error line or the Test Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Row buttons open the test's source (Go to Test) or the failing line (Go to Error), and the right-click menu has Reveal in Test Explorer. Selecting a test row also selects it in the Test Explorer.
- Commands: vscode.revealTest; _revealTestInExplorer
- Menus: TestPeekElement; TestMessageContext
- Keys: none found
- Settings: none found
- Cursor: changed (Go to Test appears on message rows only, stock also shows it on test rows)
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsTree.ts; C bundle (testing.outputPeek.goToTest, testing.outputPeek.goToError, testing.outputPeek.revealInExplorer, vscode.revealTest, _revealTestInExplorer)

### PN-6.30 Clear all results
- Tag: both
- Evidence: source
- Boundary: no
- What: The header icon, or the Command Palette entry, throws away every stored run. The Test Results tab then hides itself because there is nothing left to show.
- Commands: testing.clearTestResults
- Menus: ViewTitle (Test Results header, navigation group); TestPeekTitle; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ClearTestResultsAction); V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; C bundle (testing.clearTestResults on the view title when view is workbench.panel.testResults.view)

### PN-6.31 Swap the tree and details sides
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A header button, or a setting, moves the run tree between the right and left of the details pane.
- Commands: testing.toggleResultsViewLayout
- Menus: ViewTitle (Test Results header, navigation group)
- Keys: none found
- Settings: testing.resultsView.layout
- Cursor: missing
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ToggleResultsViewLayoutAction); V\src\vs\workbench\contrib\testing\common\configuration.ts; V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts; C bundle (zero hits for testing.toggleResultsViewLayout, testing.resultsView.layout, treeLeft)

### PN-6.32 Run a follow-up action offered under a failure
- Tag: both
- Evidence: source
- Boundary: no
- What: A test extension can offer follow-up links under a failed message. The first shows as a link, any others sit behind "+N More..." which opens a pick list. The links come from the extension, nothing built in supplies them.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testResultsView\testResultsViewContent.ts (FollowupActionWidget); C bundle (provideTestFollowups, testing-followup-action)

## Handed off
- Debug Console list picker for `debug consoles ` (the quick access prefix opened by workbench.action.debug.selectDebugConsole): QA-4
- The editor suggest list widget that shows console completions (the console only feeds it items): AC-1
- Test gutter run and debug icons, inline failure text (testing.toggleInlineTestOutput), Peek Related Test and Peek Related Code, and coverage highlights in the editor: ED-11 (no ED group names tests, so this is the closest fit)
- The "View Test Coverage" and "Close Test Coverage" rows inside the results tree, and the coverage view they open: SB-6
- Test Explorer run commands that do not use the results view (Run All Tests, Rerun Failed Tests testing.reRunFailTests, Debug Failed Tests, Run at Cursor): SB-6

## Unclear
- PN-6 has 33 features because it covers two separate views. If the catalog splits it, features 1 to 17 are the Debug Console and 18 to 33 are Test Results.
- The in-editor test peek (feature 33) lives in the Editor but shares its whole widget with Test Results. The brief for this group named it, so it is listed here. ED-11 could own it instead.
- The accessible view and accessibility help for the Debug Console (replAccessibleView.ts, replAccessibilityHelp.ts, replAccessibilityAnnouncer.ts) were not listed. The catalog says nobody owns the accessible view. Cursor has the replAccessibilityAnnouncer string and the accessibility.verbosity.debug setting.
- Row buttons in the results tree (Show Result Output, Cancel, Rerun, Debug, Go to Test) are built in code with action ids that start with testing.outputPeek. They are not registered commands, so the Commands lists give the real commands they call.
- The Test Results tab has no open command of its own (the container sets doNotRegisterOpenCommand). Only Show Output and the generic Focus command reach it.
- The Cursor bundle is older than 1.128.0 in the testing and debug code: it lacks the results layout toggle and setting, the from-last-run failed-test commands and the debug.console.maximumLines setting. I searched zero-hit strings only, not the NLS table, so a renamed label is not ruled out.
- Compared V and N source for the Debug Console and Test Results files: no new commands, settings or views after 1.128.0. Only inner code changed.
- No Cursor-only feature or menu entry was found in either view. The Debug Console and Test Results menus match stock in the bundle.
