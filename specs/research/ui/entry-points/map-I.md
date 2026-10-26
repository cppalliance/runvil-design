Bucket: I. Date: 2026-10-25. Pass: 4.
MAP | Focus Comments view | PN-7.3 | label | title of commentsFocusViewFromFilter, Ctrl+Down from the filter box back to the list
MAP | Clear filter text | PN-7.3 | label | title of commentsClearFilterText, Escape in the Comments filter box
MAP | Focus comments filter | PN-7.3 | label | title of commentsFocusFilter, Ctrl+F in the Comments view
EXCL | Expression Value | label | agent | row in the chat context picker for debug values (debugChatIntegration.ts); the full label is "Expression Value..."
EXCL | All variables in | label | agent | row in the chat context picker, full label "All variables in {0}" with the scope name
EXCL | Type an expression to evaluate | label | agent | disabled hint row in the chat context picker, full label ends with "..."
EXCL | Attach to Current Code Renderer | label | developer-only | no palette entry, throws "Refusing to attach to renderer outside of development context" unless the window is an extension development or test window; not in Cursor
MAP | Debug: Add Conditional Breakpoint | ED-11.5 | label | editor action editor.debug.action.conditionalBreakpoint; V label ends with "..." and the menu title is Conditional Breakpoint...
MAP | Debug: Add Triggered Breakpoint | ED-11.7 | label | editor action editor.debug.action.triggerByBreakpoint; V label ends with "..."
MAP | Close Exception Widget | ED-11.16 | label | editor.debug.action.closeExceptionWidget, Escape while the exception widget shows
MAP | Debug Console: Focus Filter | PN-6.10 | label | repl.action.filter, Ctrl+F in the console input
MAP | Debug Console: Focus Find | PN-6.11 | label | repl.action.find, Ctrl+Alt+F and the search icon in the console title
MAP | Open Interactive Window | ED-7.37 | label | _interactive.open, f1 false, called by extension code; ED-7.37 already says so
MAP | Execute Code | ED-7.37 | label | title of interactive.execute, Ctrl+Enter in the Interactive Window input box
MAP | Clear the interactive window input editor contents | ED-7.37 | label | interactive.input.clear, f1 false
MAP | Previous value in history | ED-7.37 | label | interactive.history.previous, Up at the top line of the input box
MAP | Next value in history | ED-7.37 | label | interactive.history.next, Down at the bottom line of the input box
MAP | Compare Input 2 With Base | ED-6.18 | label | mergeEditor.compareInput2WithBase, the toolbar button on Input 2
MAP | Toggle Between Merge Editor Inputs | ED-6.21 | label | mergeEditor.toggleBetweenInputs, Ctrl+Shift+T; not in Cursor, as ED-6.21 already notes
NEW | Select folder to save to | NEW-I-4 | label | open-folder dialog title of the Merge Editor (Dev) commands that save and load the merge editor state; reused by the load command
MAP | Accept (result is dirty) | ED-6.19 | label | tooltip of the Accept checkbox when Result no longer matches the input
MAP | Undo accept | ED-6.19 | label | tooltip of the Accept checkbox after the first input was accepted
MAP | Undo accept (currently second) | ED-6.19 | label | tooltip of the Accept checkbox when the change was accepted as the second input
MAP | Split Cell | ED-7.6 | label | notebook.cell.split
MAP | Show Cell Failure Actions | ED-7.17 | label | notebook.cell.openFailureActions, Ctrl+. on a failed cell
EXCL | Fix Cell Error | label | agent | notebook.cell.chat.fixError sends the error to chat; not in Cursor
MAP | Hide Find in Notebook | ED-7.30 | label | notebook.hideFind, Escape
MAP | Find in Notebook | ED-7.30 | label | notebook.find, Ctrl+F
MAP | Format Cells | ED-7.33 | label | undo stack name of the edit made by format on cell execution and on save, not a command title
MAP | Delete Left | ED-7.32 | label | noteMultiCursor.deleteLeft, Backspace in notebook multi-cursor mode
MAP | Delete Right | ED-7.32 | label | noteMultiCursor.deleteRight, Delete in notebook multi-cursor mode
EXCL | Keypresses that should be handled by the focused element in the cell output | label | internal | title of notebook.cell.nullAction, a no-op that binds the arrow keys so the list does not steal them from an input inside an output
MAP | Focus Next Cell Editor | ED-7.9 | label | notebook.focusNextEditor, Down at the last line of a cell
MAP | Focus Previous Cell Editor | ED-7.9 | label | notebook.focusPreviousEditor, Up at the first line of a cell
MAP | Focus First Cell | ED-7.9 | label | notebook.focusTop, Ctrl+Home
MAP | Focus Last Cell | ED-7.9 | label | notebook.focusBottom, Ctrl+End
MAP | Focus In Active Cell Output | ED-7.9 | label | notebook.cell.focusInOutput
MAP | Focus Out Active Cell Output | ED-7.9 | label | notebook.cell.focusOutOutput
MAP | Cell Cursor Page Up | ED-7.9 | label | notebook.cell.cursorPageUp
MAP | Cell Cursor Page Up Select | ED-7.9 | label | notebook.cell.cursorPageUpSelect, Shift+PageUp
MAP | Cell Cursor Page Down | ED-7.9 | label | notebook.cell.cursorPageDown
MAP | Cell Cursor Page Down Select | ED-7.9 | label | notebook.cell.cursorPageDownSelect, Shift+PageDown
MAP | Clear All Inline Values | ED-7.18 | label | notebook.clearAllInlineValues
EXCL | Execute Notebook Variable Provider | label | internal | _executeNotebookVariableProvider, f1 false, API command that returns variables to extension code
MAP | Set Profile | ED-7.28 | label | notebook.setProfile, only runs with an argument from the layout walkthrough
MAP | Copy Cell Output | ED-7.21 | label | notebook.cellOutput.copy in the output toolbar
MAP | Open Cell Output in Text Editor | ED-7.22 | label | notebook.cellOutput.openInTextEditor, f1 false, the "text editor" link of truncated output
EXCL | Accept and Run | label | agent | notebook.inlineChat.acceptChangesAndRun, button of the notebook inline chat; not in Cursor
EXCL | Accept the changes and run the cell | label | agent | tooltip of the same notebook inline chat button
MAP | Delete Cell | ED-7.3 | label | notebook.cell.delete
MAP | Change Cell Language | ED-7.7 | label | notebook.cell.changeLanguage, Ctrl+K M
MAP | Accept Detected Language for Cell | ED-7.7 | label | notebook.cell.detectLanguage, Alt+Shift+D
MAP | Execute Cell and Focus Container | ED-7.13 | label | notebook.cell.executeAndFocusContainer
MAP | Execute Notebook Cell and Select Below | ED-7.13 | label | notebook.cell.executeAndSelectBelow, Shift+Enter
MAP | Execute Notebook Cell and Insert Below | ED-7.13 | label | notebook.cell.executeAndInsertBelow, Alt+Enter
MAP | Insert Code Cell Above and Focus Container | ED-7.2 | label | notebook.cell.insertCodeCellAboveAndFocusContainer
MAP | Insert Code Cell Below and Focus Container | ED-7.2 | label | notebook.cell.insertCodeCellBelowAndFocusContainer
MAP | Add Code Cell At Top | ED-7.2 | label | notebook.cell.insertCodeCellAtTop, f1 false, the "+ Code" button above the first cell
MAP | Add Markdown Cell At Top | ED-7.2 | label | notebook.cell.insertMarkdownCellAtTop, f1 false, the "+ Markdown" button above the first cell
MAP | Add Code Cell | ED-7.2 | label | tooltip of the "+ Code" buttons between cells, at the top and on the toolbar
MAP | Add Code | ED-7.2 | label | short title of the same button when the toolbar is minimal
MAP | Add Markdown Cell | ED-7.2 | label | tooltip of the "+ Markdown" buttons
MAP | Toggle Notebook Line Numbers | ED-7.28 | label | notebook.toggleLineNumbersFromEditorTitle, in the Customize Notebook menu
MAP | Show Unchanged Cells | ED-7.35 | label | notebook.diffEditor.showUnchangedCells and the link on the collapsed-cells bar of the multi-file notebook diff
MAP | Toggle Collapse Unchanged Regions | ED-7.35 | label | notebook.diff.cell.toggleCollapseUnchangedRegions, toggled by diffEditor.hideUnchangedRegions.enabled
MAP | Revert Notebook Metadata | ED-7.36 | label | notebook.diff.revertMetadata, f1 false, button on the notebook metadata header
EXCL | Switch Outputs Rendering | label | not-ui | only inside commented-out code in notebookDiffActions.ts and not in Cursor; the live command is Switch Output Rendering (ED-7.35)
MAP | Switch Output Rendering | ED-7.35 | label | notebook.diff.cell.switchOutputRenderingStyleToText, f1 false, button on the outputs header
MAP | Toggle Inline View | ED-7.35 | label | notebook.diff.inline.toggle, experimental, in the notebook diff title menu
MAP | Show Cell Line Numbers | ED-7.28 | label | notebook.cell.toggleLineNumbers, checked item in the cell menu View group
MAP | Notebook webview content | ED-7.20 | label | accessible title of the web view that draws cell outputs, not a command
EXCL | Explain Changes | label | agent | workbench.scm.action.graph.summarizeHistoryItem sends a commit to chat; not in Cursor
MAP | History Item Reference Picker | SB-4.34 | label | workbench.scm.action.graph.pickHistoryItemRefs, the branch icon in the Graph title
MAP | View & Sort | SB-4.5 | label | submenu of the Source Control title that holds list or tree view and the sort keys
EXCL | Get Search Results | label | internal | search.action.getSearchResults, f1 false, returns the results as text to callers; SB-3 already lists it as having no UI
MAP | Toggle Query Details | SB-3.10 | label | workbench.action.search.toggleQueryDetails, Ctrl+Shift+J, shown to people as Toggle Search Details
MAP | Open Match To Side | SB-3.21 | label | search.action.openResultToSide, Ctrl+Enter on a match
MAP | Focus Search Editor Files to Include | ED-3.32 | label | search.action.focusFilesToInclude
MAP | Focus Search Editor Files to Exclude | ED-3.32 | label | search.action.focusFilesToExclude
MAP | Toggle Match Case | ED-3.31 | label | toggleSearchEditorCaseSensitive, the Search Editor copy of the Alt+C key
MAP | Toggle Match Whole Word | ED-3.31 | label | toggleSearchEditorWholeWord
MAP | Toggle Use Regular Expression | ED-3.31 | label | toggleSearchEditorRegex
NEW | Continue without scanning the task output | NEW-I-1 | label | first row of the problem matcher picker shown before a task runs; runs once and writes nothing
NEW | Never scan the task output for this task | NEW-I-1 | label | row that writes an empty problem matcher list into that task
NEW | Never scan the task output for tasks | NEW-I-1 | label | the V label is "Never scan the task output for {0} tasks" with the task type; row that adds the type to task.problemMatchers.neverPrompt
NEW | Learn more about scanning the task output | NEW-I-1 | label | row that opens the problem matcher documentation page and does not run the task
NEW | No instance is currently running | NEW-I-2 | label | single row of the "Select an instance to terminate" picker when a running task is started again; stock only, Cursor has no instancePolicy
MAP | is already marked as the default test task | QA-4.14 | label | V label "{0} is already marked as the default test task." is the preselected row of the Configure Default Test Task picker
EXCL | Run Rebuild Task | label | not-ui | registration is commented out in task.contribution.ts, so the command does not exist; not in Cursor
EXCL | Run Clean Task | label | not-ui | registration is commented out in task.contribution.ts, so the command does not exist; not in Cursor
MAP | Hide Inline Coverage | ED-13.5 | label | toggled-state title of Toggle Inline Coverage (testing.toggleInlineCoverage)
EXCL | Get Selected Profiles | label | internal | testing.getSelectedProfiles, no palette entry, API command for extension code; SB-6 already lists it as having no UI
EXCL | Get Explorer Selection | label | internal | _testing.getExplorerSelection, no palette entry, API command for extension code; SB-6 already lists it as having no UI
NEW | Run Test at Cursor with Coverage | NEW-I-3 | label | testing.coverageAtCursor; ED-11 handed the three at-cursor commands to SB-6 and SB-6 has no feature for them
