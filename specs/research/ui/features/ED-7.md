Group: ED-7 Notebooks, Interactive Window and REPL. Section: Editor (05-editor). Date: 2026-10-25. Pass: 2.
Features: 48.

### ED-7.1 Create or open a notebook
- Tag: both
- Evidence: source
- Boundary: no
- What: Create an untitled Jupyter notebook from File > New File or the Command Palette (it starts with one empty Python code cell), open an .ipynb in the notebook editor, or switch an .ipynb that opened as text with the "Open in Notebook Editor" CodeLens. When a notebook cannot load, the editor shows an error with actions Open As Text, Open in Text Editor (too large) or Install or Enable extension for the view type.
- Commands: ipynb.newUntitledIpynb; ipynb.openIpynbInNotebookEditor
- Menus: file/newFile; commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\ipynb\package.json; V\extensions\ipynb\src\ipynbMain.ts; V\src\vs\workbench\contrib\notebook\browser\notebookEditor.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C\extensions\ipynb\package.json; C bundle (workbench.notebook.action.openAsText, workbench.notebook.action.openInTextEditor, workbench.notebook.action.installOrEnableMissing, pane id workbench.editor.notebook)

### ED-7.2 Insert a cell
- Tag: both
- Evidence: source
- Boundary: no
- What: Add a code or markdown cell above or below the current cell, at the top of the notebook, from the "+ Code" and "+ Markdown" buttons that appear between cells and at the top, from the notebook toolbar, or from the Insert Cell submenu in the cell menu.
- Commands: notebook.cell.insertCodeCellAbove; notebook.cell.insertCodeCellBelow; notebook.cell.insertMarkdownCellAbove; notebook.cell.insertMarkdownCellBelow; notebook.cell.insertCodeCellAtTop; notebook.cell.insertMarkdownCellAtTop; notebook.cell.insertCodeCellAboveAndFocusContainer; notebook.cell.insertCodeCellBelowAndFocusContainer
- Menus: NotebookCellInsert; NotebookCellBetween; NotebookCellListTop; NotebookToolbar; NotebookCellTitle
- Keys: Ctrl+Enter insert code cell below and Ctrl+Shift+Enter insert code cell above, when notebookCellListFocused && !inputFocus (Cmd on macOS)
- Settings: notebook.insertToolbarLocation
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\insertCellActions.ts; V\src\vs\workbench\contrib\notebook\browser\controller\coreActions.ts; V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookTopCellToolbar.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.cell.insertCodeCellBelow, notebook.cell.insertMarkdownCellAtTop, NotebookCellBetween, NotebookCellListTop)

### ED-7.3 Delete a cell
- Tag: both
- Evidence: source
- Boundary: no
- What: Remove the focused cell with the Delete key or the trash icon on the cell toolbar; deleting a running cell asks for confirmation.
- Commands: notebook.cell.delete
- Menus: NotebookCellDelete; InteractiveCellDelete
- Keys: Delete (Cmd+Backspace on macOS), when notebookEditorFocused && !inputFocus && !notebookOutputInputFocused
- Settings: notebook.confirmDeleteRunningCell
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.cell.delete, NotebookCellDelete, notebook.confirmDeleteRunningCell)

### ED-7.4 Move or reorder cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Move a cell up or down with Alt+Up and Alt+Down (the cell menu shows Move Cell Up and Down only when drag and drop is off), or drag it by the gutter handle to a new place; holding Ctrl (Alt on macOS) while dropping copies the cell instead.
- Commands: notebook.cell.moveUp; notebook.cell.moveDown
- Menus: NotebookCellTitle
- Keys: Alt+Up; Alt+Down, when notebookEditorFocused && !inputFocus
- Settings: notebook.dragAndDropEnabled
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\cellCommands\cellCommands.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellDnd.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellDragRenderer.ts; C bundle (notebook.cell.moveUp, notebook.cell.moveDown, notebook.dragAndDropEnabled and its description in nls)

### ED-7.5 Cut, copy, paste and duplicate cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Cut, copy or paste whole cells (paste below or paste above) with the normal clipboard keys while a cell, not its text, has focus, or duplicate a cell upward or downward with Copy Cell Up and Copy Cell Down.
- Commands: notebook.cell.copy; notebook.cell.cut; notebook.cell.paste; notebook.cell.pasteAbove; notebook.cell.copyUp; notebook.cell.copyDown
- Menus: NotebookCellTitle
- Keys: Ctrl+C; Ctrl+X; Ctrl+V; Ctrl+Shift+V (paste above); Alt+Shift+Up; Alt+Shift+Down, when notebookEditorFocused && !inputFocus (Windows also takes Ctrl+Insert, Shift+Delete and Shift+Insert)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\clipboard\notebookClipboard.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\cellCommands\cellCommands.ts; C bundle (notebook.cell.copy, notebook.cell.pasteAbove, notebook.cell.copyDown)

### ED-7.6 Split and join cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Split a cell at the text cursor into two cells, or merge the focused cell with the previous cell, the next cell, or all selected cells.
- Commands: notebook.cell.split; notebook.cell.joinAbove; notebook.cell.joinBelow; notebook.cell.joinSelected
- Menus: NotebookCellTitle
- Keys: Ctrl+K Ctrl+Shift+\ (split, when editorTextFocus in an editable cell); WinCtrl+Alt+J (join with next); WinCtrl+Alt+Shift+J (join with previous), when notebookEditorFocused (WinCtrl is Ctrl on macOS, the Windows key elsewhere)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\cellCommands\cellCommands.ts; C bundle (notebook.cell.split, notebook.cell.joinAbove, notebook.cell.joinSelected, label Join Selected Cells)

### ED-7.7 Change a cell's type or language
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn a code cell into a markdown cell or back with the cell menu or the Y and M keys, pick the cell's language from the picker on the cell status bar or with Ctrl+K M, or accept the language the editor detected for the cell.
- Commands: notebook.cell.changeToCode; notebook.cell.changeToMarkdown; notebook.cell.changeLanguage; notebook.cell.detectLanguage
- Menus: NotebookCellTitle
- Keys: Y (to code); M (to markdown), when notebookEditorFocused && !inputFocus && !notebookOutputFocused; Ctrl+K M (change language), when notebookEditorFocused && editable
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\cellCommands\cellCommands.ts; V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\cellStatusBar\statusBarProviders.ts; C bundle (notebook.cell.changeLanguage, notebook.cell.changeToMarkdown, notebook.cell.detectLanguage)

### ED-7.8 Edit a markdown cell and render it
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Enter on a markdown cell to edit its source, press Escape or Ctrl+Enter to render it again (the cell toolbar shows an Edit or Stop Editing icon), or render every markdown cell at once. Cursor lacks Stop Editing All Cells.
- Commands: notebook.cell.edit; notebook.cell.quitEdit; notebook.quitEditAllCells; notebook.renderAllMarkdownCells
- Menus: NotebookCellTitle
- Keys: Enter (edit), when notebookCellListFocused && !inputFocus; Escape (quit edit), when notebookEditorFocused && inputFocus; WinCtrl+Enter (render markdown, Ctrl+Alt+Enter on Windows)
- Settings: none found
- Cursor: changed - notebook.quitEditAllCells and its label Stop Editing All Cells were not found; the other three ids are present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts; V\src\vs\workbench\contrib\notebook\browser\controller\executeActions.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\markupCell.ts; C bundle (notebook.cell.edit, notebook.cell.quitEdit, notebook.renderAllMarkdownCells)

### ED-7.9 Move focus between cells and select several
- Tag: both
- Evidence: source
- Boundary: no
- What: Move between cells with the arrow keys at the first or last line, jump to the first or last cell, page up and down, center the active cell, move focus into and out of a cell's output, and select a range of cells with Shift plus arrows or page keys.
- Commands: notebook.focusNextEditor; notebook.focusPreviousEditor; notebook.focusTop; notebook.focusBottom; notebook.cell.cursorPageUp; notebook.cell.cursorPageDown; notebook.cell.cursorPageUpSelect; notebook.cell.cursorPageDownSelect; notebook.centerActiveCell; notebook.cell.focusInOutput; notebook.cell.focusOutOutput; list.expandSelectionDown; list.expandSelectionUp
- Menus: none found
- Keys: Up and Down at the cell edge (focusPreviousEditor, focusNextEditor); Ctrl+Home and Ctrl+End; PageUp and PageDown; Shift+PageUp and Shift+PageDown; Ctrl+L (center active cell); Ctrl+Down on Windows or Ctrl+Shift+Down elsewhere (focus in output); Ctrl+Shift+Up (focus out of output); Shift+Up and Shift+Down (list.expandSelection), mostly with notebookEditorFocused
- Settings: notebook.navigation.allowNavigateToSurroundingCells
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\navigation\arrow.ts; V\src\vs\workbench\browser\actions\listCommands.ts; C bundle (notebook.focusNextEditor, notebook.centerActiveCell, notebook.cell.cursorPageDownSelect, notebook.cell.focusInOutput, list.expandSelectionDown, notebook.navigation.allowNavigateToSurroundingCells)

### ED-7.10 Comment out the selected cells
- Tag: both
- Evidence: source
- Boundary: no
- What: With one or more cells selected (not editing text), press Ctrl+/ to toggle line comments over the whole content of every selected cell.
- Commands: notebook.commentSelectedCells
- Menus: none found
- Keys: Ctrl+/, when notebookEditorFocused && notebookEditable && !inputFocus
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts; C bundle (notebook.commentSelectedCells, label Comment Selected Cells)

### ED-7.11 Collapse and expand cell input and output
- Tag: both
- Evidence: source
- Boundary: no
- What: Hide a cell's code or its output behind a one-line placeholder ("Outputs are collapsed", double-click to expand), for one cell, all cells, or by toggling outputs.
- Commands: notebook.cell.collapseCellInput; notebook.cell.expandCellInput; notebook.cell.collapseCellOutput; notebook.cell.expandCellOutput; notebook.cell.toggleOutputs; notebook.cell.collapseAllCellInputs; notebook.cell.expandAllCellInputs; notebook.cell.collapseAllCellOutputs; notebook.cell.expandAllCellOutputs
- Menus: none found
- Keys: Ctrl+K Ctrl+C (collapse or expand input); Ctrl+K T (collapse or expand output), when notebookCellListFocused && !inputFocus
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\cellCommands\cellCommands.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\collapsedCellOutput.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\collapsedCellInput.ts; C bundle (notebook.cell.collapseCellInput, notebook.cell.toggleOutputs, notebook.cell.expandAllCellOutputs, label Collapse All Cell Outputs, label Outputs are collapsed)

### ED-7.12 Fold, unfold and run a markdown section
- Tag: both
- Evidence: source
- Boundary: no
- What: Fold the cells under a markdown heading with the folding arrow or the fold keys (a hint says "N cells hidden"), unfold them again, and run the cells of a section from the cell toolbar, the outline entry or the sticky scroll menu.
- Commands: notebook.fold; notebook.unfold; notebook.section.foldSection; notebook.section.expandSection; notebook.section.runSingleCell; notebook.section.runCells
- Menus: NotebookCellTitle; NotebookOutlineActionMenu; NotebookStickyScrollContext
- Keys: Ctrl+Shift+[ (fold; Cmd+Alt+[ on macOS); Ctrl+Shift+] (unfold; Cmd+Alt+] on macOS), when notebookEditorFocused && !inputFocus
- Settings: notebook.showFoldingControls
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\foldingController.ts; V\src\vs\workbench\contrib\notebook\browser\controller\sectionActions.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\foldedCellHint.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.fold, notebook.unfold, notebook.section.runCells, notebook.section.foldSection, notebook.showFoldingControls)

### ED-7.13 Run a cell and stop it
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the focused cell with the play button in its gutter or with Ctrl+Enter, run it and move to the next cell with Shift+Enter, run it and insert a new cell below with Alt+Enter, and stop a running or pending cell with the same button.
- Commands: notebook.cell.execute; notebook.cell.executeAndFocusContainer; notebook.cell.executeAndSelectBelow; notebook.cell.executeAndInsertBelow; notebook.cell.cancelExecution
- Menus: NotebookCellExecutePrimary; NotebookCellExecute
- Keys: WinCtrl+Enter (execute; Ctrl+Alt+Enter on Windows), when notebookCellListFocused; Shift+Enter (execute and select below), when notebookCellListFocused && !inlineChatFocused; Alt+Enter (execute and insert below), when notebookCellListFocused
- Settings: notebook.scrolling.revealNextCellOnExecute; notebook.consolidatedRunButton
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\executeActions.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\codeCellRunToolbar.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.cell.execute, notebook.cell.executeAndSelectBelow, notebook.cell.cancelExecution, notebook.cell.executeAndInsertBelow)

### ED-7.14 Run all cells or a range of cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Run every cell with Run All, run all cells above or from the current cell down, and stop the whole run, which swaps Run All for a Stop Execution button or, for kernels that support it, an Interrupt button.
- Commands: notebook.execute; notebook.cell.executeCellsAbove; notebook.cell.executeCellAndBelow; notebook.cancelExecution; notebook.interruptExecution
- Menus: EditorTitle; NotebookToolbar; NotebookCellExecute; NotebookCellTitle; InteractiveToolbar
- Keys: none found
- Settings: notebook.globalToolbar; notebook.consolidatedRunButton
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\executeActions.ts; C bundle (notebook.execute, notebook.cell.executeCellsAbove, notebook.cell.executeCellAndBelow, notebook.interruptExecution, notebook.cancelExecution, label Run All, label Interrupt)

### ED-7.15 Jump to the running or failed cell
- Tag: both
- Evidence: source
- Boundary: no
- What: Use the Go To button (spinning while a cell runs, an error icon after a failure) to scroll to the running cell, the cell stopped at a debugger frame, or the most recently failed cell.
- Commands: notebook.revealRunningCell; notebook.revealLastFailedCell
- Menus: EditorTitle; NotebookToolbar; NotebookCellExecuteGoTo; InteractiveToolbar
- Keys: none found
- Settings: notebook.globalToolbar
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\executeActions.ts; C bundle (notebook.revealRunningCell, notebook.revealLastFailedCell, NotebookCellExecuteGoTo, label Go to Running Cell)

### ED-7.16 Read the cell status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Read the thin bar under each code cell: a success, failure, pending or executing icon, the execution count and duration (last run time in verbose mode, details on hover), the cell language picker, and items added by extensions.
- Commands: notebook.cell.changeLanguage; notebook.cell.detectLanguage
- Menus: none found
- Keys: none found
- Settings: notebook.showCellStatusBar; notebook.cellExecutionTimeVerbosity
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\cellStatusBar\executionStatusBarItemController.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\cellStatusBar\statusBarProviders.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellStatusPart.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.showCellStatusBar, notebook.cellExecutionTimeVerbosity, label Select Cell Language Mode)

### ED-7.17 Show quick actions for a failed cell
- Tag: both
- Evidence: source
- Boundary: yes
- What: After a cell fails, a Quick Actions item appears in its status bar and opens a code action menu for the error, and diagnostics for the failure are shown in the cell. Boundary: the same menu also lists chat actions (Fix and Explain Cell Error), which are left out.
- Commands: notebook.cell.openFailureActions
- Menus: none found
- Keys: none found
- Settings: notebook.cellFailureDiagnostics
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\cellDiagnostics\cellDiagnosticsActions.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\cellDiagnostics\diagnosticCellStatusBarContrib.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.cell.openFailureActions, notebook.cellFailureDiagnostics, label Quick Actions)

### ED-7.18 See variable values
- Tag: both
- Evidence: source
- Boundary: no
- What: Show the value of variables inline in a code cell after it runs (off by default, with a command to clear them), and open the Notebook Variables view from a toolbar button when the kernel can list variables (the view is off by default).
- Commands: notebook.openVariablesView; notebook.clearAllInlineValues
- Menus: EditorTitle; NotebookToolbar; InteractiveToolbar
- Keys: none found
- Settings: notebook.inlineValues; notebook.variablesView
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookInlineVariables.ts; V\src\vs\workbench\contrib\notebook\browser\controller\variablesActions.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\notebookVariables\notebookVariables.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.openVariablesView, notebook.clearAllInlineValues, workbench.notebook.variables, notebook.inlineValues)

### ED-7.19 Pick a kernel
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose which kernel runs the notebook from the Select Kernel button in the toolbar or editor title (it reads Detecting Kernels while detection runs): a quick pick lists suggested and recent kernels, kernel sources, Select Another Kernel, and options to install suggested extensions or browse the marketplace for kernel extensions.
- Commands: _notebook.selectKernel; notebook.clearNotebookKernelsMRUCache
- Menus: EditorTitle; NotebookToolbar; InteractiveToolbar; NotebookKernelSource
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookKernelView.ts; V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookKernelQuickPickStrategy.ts; V\src\vs\workbench\contrib\notebook\browser\services\notebookKernelHistoryServiceImpl.ts; V\src\vs\workbench\contrib\notebook\browser\controller\coreActions.ts; C bundle (_notebook.selectKernel, notebook.clearNotebookKernelsMRUCache, NotebookKernelSource, labels Select Notebook Kernel and Detecting Kernels)

### ED-7.20 Read cell outputs
- Tag: both
- Evidence: source
- Boundary: no
- What: See each code cell's output below it, drawn by the built-in renderers for plain text, stdout and stderr streams, images, HTML and errors (a traceback with a Show Details and Hide Details link and clickable file paths), with a message when the cell has no output or no renderer fits.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: notebook.output.textLineLimit; notebook.output.lineHeight; notebook.output.fontSize; notebook.output.fontFamily; notebook.output.wordWrap; notebook.output.linkifyFilePaths; notebook.output.minimalErrorRendering
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellOutput.ts; V\src\vs\workbench\contrib\notebook\browser\view\renderers\backLayerWebView.ts; V\extensions\notebook-renderers\package.json; V\extensions\notebook-renderers\src\index.ts; V\extensions\notebook-renderers\src\textHelper.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C\extensions\notebook-renderers\package.json; C bundle (notebook.output.textLineLimit, notebook.output.minimalErrorRendering, label Cell has no output)

### ED-7.21 Use the output toolbar
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the "..." button beside an output to copy it, save an image output, show empty outputs, clear the cell outputs, open it in the Output Preview editor, or change its presentation (a mime type picker that also offers more renderers from the marketplace); Save Mimetype Display Order stores the choice in settings. Cursor lacks Save Image and Open in Output Preview.
- Commands: notebook.cellOutput.copy; notebook.cellOutput.saveImage; notebook.cellOutput.openInOutputPreview; notebook.cellOuput.showEmptyOutputs; notebook.output.pickMimetype; notebook.saveMimeTypeOrder
- Menus: NotebookOutputToolbar; webview/context
- Keys: none found
- Settings: notebook.consolidatedOutputButton; notebook.displayOrder; notebook.output.openInPreviewEditor.enabled
- Cursor: changed - notebook.cellOutput.saveImage, notebook.cellOutput.openInOutputPreview and the setting notebook.output.openInPreviewEditor.enabled were not found; copy, show empty outputs, Change Presentation and Save Mimetype Display Order are present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\cellOutputActions.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellOutput.ts; V\src\vs\workbench\contrib\notebook\browser\controller\layoutActions.ts; V\src\vs\workbench\contrib\notebook\browser\outputEditor\notebookOutputEditor.ts; V\extensions\ipynb\package.json; C bundle (notebook.cellOutput.copy, notebook.cellOuput.showEmptyOutputs, notebook.output.pickMimetype, notebook.saveMimeTypeOrder, label Change Presentation)

### ED-7.22 Handle long text output
- Tag: both
- Evidence: source
- Boundary: no
- What: When text output is longer than the line limit it is cut with a message "Output is truncated" that links to "scrollable element" (show it in a scroll box in place), "text editor" (open the raw output in a text editor) and the layout settings; Ctrl+K Y toggles the scroll box for the focused cell, and a cell with more outputs than the output limit shows a "show more" link that opens the raw output data in a text editor.
- Commands: notebook.cellOutput.openInTextEditor; workbench.action.openLargeOutput; cellOutput.enableScrolling; notebook.cell.toggleOutputScrolling
- Menus: webview/context
- Keys: Ctrl+K Y (toggle scroll cell output), when notebookCellListFocused && !inputFocus && notebookCellHasOutputs
- Settings: notebook.output.scrolling; notebook.output.textLineLimit; notebook.output.wordWrap
- Cursor: present
- Source: V\extensions\notebook-renderers\src\textHelper.ts; V\src\vs\workbench\contrib\notebook\browser\view\renderers\backLayerWebView.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellOutput.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\cellCommands\cellCommands.ts; V\src\vs\workbench\contrib\notebook\browser\controller\cellOutputActions.ts; V\extensions\ipynb\package.json; C bundle (notebook.cell.toggleOutputScrolling, cellOutput.enableScrolling, workbench.action.openLargeOutput, notebook.cellOutput.openInTextEditor, label Toggle Scroll Cell Output)

### ED-7.23 Clear outputs
- Tag: both
- Evidence: source
- Boundary: no
- What: Clear the output of one cell with Alt+Delete or the cell or output toolbar, or clear the output of every cell with Clear All Outputs on the notebook toolbar.
- Commands: notebook.cell.clearOutputs; notebook.clearAllCellsOutputs
- Menus: NotebookCellTitle; NotebookOutputToolbar; EditorTitle; NotebookToolbar
- Keys: Alt+Delete (clear cell outputs), when notebookEditorFocused && !inputFocus && notebookCellHasOutputs && editable
- Settings: notebook.consolidatedOutputButton
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\editActions.ts; C bundle (notebook.cell.clearOutputs, notebook.clearAllCellsOutputs, labels Clear Cell Outputs and Clear All Outputs)

### ED-7.24 Render math in markdown cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Markdown cells draw LaTeX math written with dollar signs (inline and block) as typeset equations through the built-in KaTeX renderer that extends the Markdown renderer; the notebook uses no user macros.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\extensions\markdown-math\package.json; V\extensions\markdown-math\notebook\katex.ts; C\extensions\markdown-math\package.json; C\extensions\markdown-math\notebook-out\katex.js

### ED-7.25 Paste or drop an image into a markdown cell
- Tag: both
- Evidence: source
- Boundary: no
- What: Paste or drop an image into a markdown cell of an .ipynb file and choose "Insert Image as Attachment", which stores it in the cell and shows it through the attachment renderer; a warning with a quick fix "Remove invalid image attachment reference" appears when a referenced attachment is missing.
- Commands: ipynb.cleanInvalidImageAttachment
- Menus: none found
- Keys: none found
- Settings: ipynb.pasteImagesAsAttachments.enabled
- Cursor: present
- Source: V\extensions\ipynb\package.json; V\extensions\ipynb\src\notebookImagePaste.ts; V\extensions\ipynb\src\notebookAttachmentCleaner.ts; V\extensions\ipynb\notebook-src\cellAttachmentRenderer.ts; C\extensions\ipynb\package.json; C\extensions\ipynb\dist\ipynbMain.node.js

### ED-7.26 Use the notebook toolbar
- Tag: both
- Evidence: source
- Boundary: no
- What: Use the toolbar across the top of the notebook (on by default): Code and Markdown add buttons, Run All, Clear All Outputs, Go To, Variables, the layout menu and the kernel picker on the right, with labels set to always, never or dynamic, and a right-click menu; with it off, the run and kernel buttons move to the editor title.
- Commands: notebook.cell.insertCodeCellBelow; notebook.cell.insertMarkdownCellBelow; notebook.execute; notebook.clearAllCellsOutputs; _notebook.selectKernel; workbench.notebook.layout.configure
- Menus: NotebookToolbar; NotebookToolbarContext; EditorTitle; notebook/toolbar
- Keys: none found
- Settings: notebook.globalToolbar; notebook.globalToolbarShowLabel; notebook.insertToolbarLocation
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookEditorToolbar.ts; V\src\vs\workbench\contrib\notebook\browser\notebookEditorWidget.ts; V\src\vs\workbench\contrib\notebook\browser\controller\insertCellActions.ts; V\src\vs\workbench\contrib\notebook\browser\controller\executeActions.ts; V\src\vs\workbench\services\actions\common\menusExtensionPoint.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (NotebookToolbar, NotebookToolbarContext, notebook.globalToolbar, notebook.globalToolbarShowLabel)

### ED-7.27 Use the cell toolbar
- Tag: both
- Evidence: source
- Boundary: no
- What: Use the small toolbar on each cell for edit, split, delete and run-section buttons plus the "..." overflow menu (copy, insert, edit, view and share groups); it sits on the right by default, can move left or be hidden, and appears on hover or click.
- Commands: notebook.toggleCellToolbarPosition
- Menus: NotebookCellTitle; NotebookCellDelete; notebook/cell/title; EditorContext; EditorContextShare
- Keys: none found
- Settings: notebook.cellToolbarLocation; notebook.cellToolbarVisibility; notebook.consolidatedRunButton
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellToolbars.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellToolbarStickyScroll.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\layout\layoutActions.ts; V\src\vs\workbench\contrib\notebook\browser\controller\coreActions.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (NotebookCellTitle, notebook.toggleCellToolbarPosition, notebook.cellToolbarLocation, notebook.cellToolbarVisibility)

### ED-7.28 Customize the notebook layout
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the "Customize Notebook..." gear menu on the editor title or the Customize Notebook Layout entry on the toolbar to toggle line numbers (for all cells or per cell), breadcrumbs and the cell toolbar position, or jump to the notebook layout settings; a profile command (default, jupyter, colab) applies a bundle of layout settings. Cursor lacks the Select between Notebook Layouts entry that opens the profile walkthrough.
- Commands: workbench.notebook.layout.configure; workbench.notebook.layout.configure.editorTitle; workbench.notebook.layout.select; notebook.setProfile; notebook.toggleLineNumbers; notebook.toggleLineNumbersFromEditorTitle; notebook.cell.toggleLineNumbers; notebook.toggleCellToolbarPositionFromEditorTitle; breadcrumbs.toggleFromEditorTitle
- Menus: NotebookEditorLayoutConfigure; EditorTitle; NotebookToolbar; NotebookCellTitle
- Keys: none found
- Settings: notebook.lineNumbers; notebook.compactView; notebook.cellFocusIndicator; notebook.markup.fontSize; notebook.markdown.lineHeight; notebook.markup.fontFamily; notebook.editorOptionsCustomizations; notebook.experimental.openGettingStarted
- Cursor: changed - workbench.notebook.layout.select and the setting notebook.experimental.openGettingStarted were not found; the other ids are present
- Source: V\src\vs\workbench\contrib\notebook\browser\controller\layoutActions.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellEditorOptions.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\profile\notebookProfile.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\layout\layoutActions.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (workbench.notebook.layout.configure, workbench.notebook.layout.configure.editorTitle, notebook.setProfile, notebook.toggleLineNumbers, notebook.cell.toggleLineNumbers, label Customize Notebook...)

### ED-7.29 Show sticky scroll headings
- Tag: both
- Evidence: source
- Boundary: no
- What: Pin the markdown headings of the current section at the top of the notebook while you scroll (experimental, off by default), with flat or indented nesting, click to jump, and a right-click menu to run the section or turn the feature off.
- Commands: notebook.action.toggleNotebookStickyScroll
- Menus: NotebookStickyScrollContext; NotebookToolbarContext; CommandPalette
- Keys: none found
- Settings: notebook.stickyScroll.enabled; notebook.stickyScroll.mode
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookEditorStickyScroll.ts; V\src\vs\workbench\contrib\notebook\browser\controller\layoutActions.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.action.toggleNotebookStickyScroll, notebook.stickyScroll.enabled, NotebookStickyScrollContext, label Toggle Notebook Sticky Scroll)

### ED-7.30 Find and replace in the notebook
- Tag: both
- Evidence: source
- Boundary: no
- What: Open a find box over the notebook that searches all cells, with match case, whole word and regex, a Find Filters menu to limit the search to markdown source, rendered markdown, code cell source or code cell output, next and previous match, find in selection, and a toggle that reveals a replace row (not in the Interactive Window). Cursor lacks the two widget-focused Find Next and Find Previous commands.
- Commands: notebook.find; notebook.hideFind; notebook.findNext.fromWidget; notebook.findPrevious.fromWidget; actions.find; editor.action.startFindReplaceAction
- Menus: none found
- Keys: Ctrl+F (notebook.find), when notebookEditorFocused and the cell editor is not focused; Escape (hide find), when the find widget is visible; F3 and Shift+F3 (Cmd+G and Cmd+Shift+G on macOS), when the find widget is focused; Ctrl+H (replace, through editor.action.startFindReplaceAction)
- Settings: notebook.find.filters
- Cursor: changed - notebook.findNext.fromWidget and notebook.findPrevious.fromWidget were not found; notebook.find, notebook.hideFind, the Find Filters labels and the replace action id are present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\find\notebookFind.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\find\notebookFindReplaceWidget.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\find\notebookFindWidget.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\find\findFilters.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.find, notebook.hideFind, workbench.notebook.find, editor.action.startFindReplaceAction, label Find Filters)

### ED-7.31 Browse the notebook outline
- Tag: both
- Evidence: source
- Boundary: no
- What: The Outline view lists the notebook's markdown headings (and optionally code cells and their symbols), jumps to a cell on click, has a Filter Entries menu with Markdown Headers Only, Code Cells and Code Cell Symbols toggles, and shows run and fold buttons on each entry.
- Commands: notebook.outline.toggleShowMarkdownHeadersOnly; notebook.outline.toggleCodeCells; notebook.outline.toggleCodeCellSymbols
- Menus: ViewTitle; NotebookOutlineFilter; NotebookOutlineActionMenu
- Keys: none found
- Settings: notebook.outline.showMarkdownHeadersOnly; notebook.outline.showCodeCells; notebook.outline.showCodeCellSymbols
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\outline\notebookOutline.ts; V\src\vs\workbench\contrib\notebook\browser\controller\sectionActions.ts; C bundle (notebook.outline.toggleCodeCells, notebook.outline.toggleShowMarkdownHeadersOnly, NotebookOutlineFilter, NotebookOutlineActionMenu, label Markdown Headers Only)

### ED-7.32 Edit several cells with multi-cursor
- Tag: both
- Evidence: source
- Boundary: no
- What: Experimental and off by default: select the next or all matches of the current word across cells (Ctrl+D, Ctrl+Shift+L) and type, delete, cut, copy or paste in every cell at once, then press Escape to leave multi-cursor mode.
- Commands: notebook.addFindMatchToSelection; notebook.selectAllFindMatches; noteMultiCursor.exit; noteMultiCursor.deleteLeft; noteMultiCursor.deleteRight
- Menus: none found
- Keys: Ctrl+D (add next match); Ctrl+Shift+L (select all matches); Escape (exit); Backspace and Delete, when notebook.multiCursor.enabled is true and a cell editor is focused
- Settings: notebook.multiCursor.enabled
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\multicursor\notebookMulticursor.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.addFindMatchToSelection, notebook.selectAllFindMatches, noteMultiCursor.exit, notebook.multiCursor.enabled, label Exit Multi Cursor Mode)

### ED-7.33 Format a notebook or cell
- Tag: both
- Evidence: source
- Boundary: no
- What: Format every cell with Format Notebook, format only the focused cell with Format Cell, or let notebooks format on save, on cell run, and apply code actions on save, using the notebook default formatter.
- Commands: notebook.format; notebook.formatCell
- Menus: EditorContext
- Keys: Shift+Alt+F (Ctrl+Shift+I on Linux), when the cell editor text is not focused (Format Notebook) and when it is focused (Format Cell)
- Settings: notebook.formatOnSave.enabled; notebook.formatOnCellExecution; notebook.defaultFormatter; notebook.insertFinalNewline; notebook.codeActionsOnSave
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\format\formatting.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\saveParticipants\saveParticipants.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (notebook.format, notebook.formatCell, notebook.formatOnSave.enabled, notebook.codeActionsOnSave, labels Format Notebook and Format Cell)

### ED-7.34 Compare notebook versions in the notebook diff editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Any compare of two .ipynb versions opens the Notebook Diff Editor, which shows the notebooks cell by cell, with the changed input, outputs and metadata of each cell in their own sections, an optional overview ruler, an Open File button and a switch to the plain text diff (it is used only while the enhanced diff setting is on and no screen reader is detected). A second, opt-in Notebook Diff Editor lists changed cells like the multi-file diff with a Go To Cell button per item.
- Commands: notebook.diff.openFile; notebook.diff.switchToText; notebook.diffEditor.2.goToCell
- Menus: EditorTitle; MultiDiffEditorFileToolbar
- Keys: none found
- Settings: notebook.diff.enablePreview; notebook.diff.overviewRuler; notebook.experimental.enableNewDiffEditor
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\diff\notebookDiffEditor.ts; V\src\vs\workbench\contrib\notebook\browser\diff\notebookMultiDiffEditor.ts; V\src\vs\workbench\contrib\notebook\browser\diff\notebookDiffOverviewRuler.ts; V\src\vs\workbench\contrib\notebook\browser\diff\notebookDiffActions.ts; V\src\vs\workbench\contrib\notebook\browser\services\notebookServiceImpl.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (workbench.editor.notebookTextDiffEditor, workbench.editor.notebookMultiTextDiffEditor, notebook.diff.openFile, notebook.diff.switchToText, notebook.diffEditor.2.goToCell, notebook.diff.enablePreview)

### ED-7.35 Choose what the notebook diff shows
- Tag: both
- Evidence: source
- Boundary: no
- What: Use the diff editor title menu and section headers to show or hide output differences and metadata differences, collapse unchanged regions in each cell, show whitespace-only differences, switch an output between rendered and text, show or hide unchanged cells in the multi-file style diff, and (experimental) toggle an inline view.
- Commands: notebook.diff.showOutputs; notebook.diff.showMetadata; notebook.diff.cell.toggleCollapseUnchangedRegions; notebook.toggle.diff.cell.ignoreTrimWhitespace; notebook.diff.cell.switchOutputRenderingStyleToText; notebook.diffEditor.showUnchangedCells; notebook.diffEditor.hideUnchangedCells; notebook.diff.inline.toggle
- Menus: EditorTitle; NotebookDiffCellInputTitle; NotebookDiffCellOutputsTitle
- Keys: none found
- Settings: notebook.diff.ignoreOutputs; notebook.diff.ignoreMetadata; notebook.diff.experimental.toggleInline; diffEditor.hideUnchangedRegions.enabled
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\diff\notebookDiffActions.ts; V\src\vs\workbench\contrib\notebook\browser\diff\unchangedEditorRegions.ts; V\src\vs\workbench\contrib\notebook\browser\diff\diffComponents.ts; C bundle (notebook.diff.showOutputs, notebook.diff.showMetadata, notebook.diff.cell.toggleCollapseUnchangedRegions, notebook.toggle.diff.cell.ignoreTrimWhitespace, notebook.diff.inline.toggle, notebook.diffEditor.hideUnchangedCells)

### ED-7.36 Step through and revert notebook changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump to the next or previous change in the notebook diff, and undo a change by reverting a cell's input, its outputs or its metadata, or the notebook-level metadata, from the buttons on each section header.
- Commands: notebook.diff.action.next; notebook.diff.action.previous; notebook.diff.cell.revertInput; notebook.diff.cell.revertOutputs; notebook.diff.cell.revertMetadata; notebook.diff.revertMetadata; notebook.diffEditor.2.cell.revertInput; notebook.diffEditor.2.cell.revertOutputs; notebook.diffEditor.2.cell.revertMetadata
- Menus: EditorTitle; NotebookDiffCellInputTitle; NotebookDiffCellOutputsTitle; NotebookDiffCellMetadataTitle; NotebookDiffDocumentMetadata; MultiDiffEditorFileToolbar
- Keys: Alt+F3 (next change); Shift+Alt+F3 (previous change), when the active editor is the Notebook Diff Editor
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\diff\notebookDiffActions.ts; C bundle (notebook.diff.action.next, notebook.diff.action.previous, notebook.diff.cell.revertInput, notebook.diff.revertMetadata, notebook.diffEditor.2.cell.revertOutputs, labels Revert Input and Revert Outputs)

### ED-7.37 Run code from the Interactive Window input box
- Tag: both
- Evidence: source
- Boundary: no
- What: Type code in the input box under the Interactive Window transcript and run it with Enter, Shift+Enter or Ctrl+Enter (the hint line "Press ... to execute" names the key), recall earlier input with Up and Down, clear the box, and move focus between the box and the history. The window itself is opened by extension code with _interactive.open, which has no Command Palette entry in stock.
- Commands: interactive.execute; interactive.history.previous; interactive.history.next; interactive.input.clear; interactive.input.focus; interactive.history.focus; workbench.input.interactive
- Menus: InteractiveInputExecute; CommandPalette
- Keys: Ctrl+Enter (execute); Shift+Enter (execute, when interactiveWindow.executeWithShiftEnter is true); Enter (execute, when it is false); Up and Down (history, at the input's first or last line, suggest list hidden); Ctrl+Up (focus history), all when the active editor is the Interactive Window
- Settings: interactiveWindow.executeWithShiftEnter; interactiveWindow.showExecutionHint
- Cursor: present
- Source: V\src\vs\workbench\contrib\interactive\browser\interactive.contribution.ts; V\src\vs\workbench\contrib\interactive\browser\interactiveEditor.ts; V\src\vs\workbench\contrib\interactive\browser\replInputHintContentWidget.ts; V\src\vs\workbench\contrib\interactive\browser\interactiveHistoryService.ts; C bundle (interactive.execute, interactive.history.previous, interactive.input.clear, interactive.history.focus, _interactive.open, workbench.editor.interactive)

### ED-7.38 Read and steer the Interactive Window transcript
- Tag: both
- Evidence: source
- Boundary: no
- What: Read the transcript of executed cells with their outputs above the input box, scroll to the top or bottom, have it scroll to each new result, start code cells collapsed, and use the window toolbar (kernel picker, interrupt, go to running cell, variables) and the per-cell title, delete and run buttons.
- Commands: interactive.scrollToTop; interactive.scrollToBottom; _notebook.selectKernel; notebook.interruptExecution; notebook.revealRunningCell; notebook.openVariablesView
- Menus: InteractiveToolbar; InteractiveCellTitle; InteractiveCellDelete; InteractiveCellExecute
- Keys: Ctrl+Home (scroll to top); Ctrl+End (scroll to bottom), when the active editor is the Interactive Window (Cmd+Up and Cmd+Down on macOS)
- Settings: interactiveWindow.alwaysScrollOnNewCell; interactiveWindow.collapseCellInputCode; interactiveWindow.promptToSaveOnClose
- Cursor: present
- Source: V\src\vs\workbench\contrib\interactive\browser\interactive.contribution.ts; V\src\vs\workbench\contrib\interactive\browser\interactiveEditor.ts; V\src\vs\workbench\contrib\interactive\browser\interactiveEditorInput.ts; V\src\vs\workbench\contrib\notebook\browser\controller\executeActions.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C bundle (interactive.scrollToTop, interactive.scrollToBottom, InteractiveToolbar, InteractiveCellTitle, interactiveWindow.alwaysScrollOnNewCell)

### ED-7.39 Use the REPL editor
- Tag: both
- Evidence: source
- Boundary: no
- What: A REPL editor shows run history as read-only cells above an input box that runs on Enter, Shift+Enter or Ctrl+Enter, with a hint line, and commands to jump to the most recent execution and back to the input box. It is created only by extension code, with no Command Palette or menu entry in stock.
- Commands: repl.execute; repl.focusLastItemExecuted; repl.input.focus
- Menus: ReplInputExecute; InteractiveToolbar; CommandPalette
- Keys: Ctrl+Enter (execute); Shift+Enter (execute, when interactiveWindow.executeWithShiftEnter is true); Enter (execute, when it is false), when the active editor is the REPL editor and the cell list is not focused; Alt+End Alt+End (focus most recent execution); Ctrl+Down when a history cell is focused, or Alt+Home Alt+Home (focus input editor)
- Settings: interactiveWindow.executeWithShiftEnter; interactiveWindow.showExecutionHint; interactiveWindow.alwaysScrollOnNewCell; accessibility.replEditor.autoFocusReplExecution; accessibility.replEditor.readLastExecutionOutput
- Cursor: present
- Source: V\src\vs\workbench\contrib\replNotebook\browser\repl.contribution.ts; V\src\vs\workbench\contrib\replNotebook\browser\replEditor.ts; V\src\vs\workbench\contrib\replNotebook\browser\replEditorInput.ts; V\src\vs\workbench\contrib\interactive\browser\replInputHintContentWidget.ts; C bundle (repl.execute, repl.focusLastItemExecuted, repl.input.focus, ReplInputExecute, workbench.editor.repl, workbench.editorinputs.replEditorInput)

### ED-7.40 Use Jupyter single-key cell shortcuts
- Tag: cursor
- Evidence: source
- Boundary: no
- What: Cursor ships a built-in extension, Jupyter Keymap (ms-toolsai, kept in the folder vscode-jupyter-keymap), with 17 keybindings that give Jupyter-style command-mode keys on a focused cell: F find, J and K move focus down and up, A and B insert a cell above and below, D D delete, Z undo, S save, C X V copy cut paste cells, Shift+L and L line numbers, O toggle outputs, plus Ctrl+Shift+- split and Ctrl+Enter and Shift+Enter run. Stock VS Code has no copy of this extension.
- Commands: notebook.find; list.focusUp; list.focusDown; notebook.cell.insertCodeCellAboveAndFocusContainer; notebook.cell.insertCodeCellBelowAndFocusContainer; notebook.cell.delete; undo; workbench.action.files.save; notebook.cell.copy; notebook.cell.cut; notebook.cell.paste; notebook.toggleLineNumbers; notebook.cell.toggleLineNumbers; notebook.cell.toggleOutputs; notebook.cell.split; notebook.cell.executeAndFocusContainer; notebook.cell.executeAndSelectBelow
- Menus: none found
- Keys: F; K; J; A; B; D D; Z; S; C; X; V; Shift+L; O; L, all when notebookEditorFocused && !inputFocus && !notebookOutputFocused; Ctrl+Shift+-, when editorTextFocus && inputFocus && notebookEditorFocused && !notebookOutputFocused; Ctrl+Enter, when editorTextFocus && inputFocus && notebookEditorFocused or notebookCellListFocused; Shift+Enter, same plus notebookCellType == 'code'
- Settings: none found
- Cursor: cursor only
- Source: C\extensions\vscode-jupyter-keymap\package.json; C bundle (notebookKeymapService, keyword notebook-keymap, label Disable other keymaps)

### ED-7.41 Undo and redo notebook changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Undo and Redo while focus is inside the notebook step through the notebook's own edit history, which covers changes to the list of cells as well as typing, and afterwards the cell that changed is revealed (a markdown cell that changed goes back into edit mode). A setting picks whether each cell keeps its own undo history for typing (on, the default) or all cells share one history, so Ctrl+Z walks back through edits across cells in the order they were made; the layout profiles set it (jupyter on, default and colab off).
- Commands: undo; redo
- Menus: MenubarEditMenu; CommandPalette
- Keys: Ctrl+Z (undo); Ctrl+Y and Ctrl+Shift+Z (redo; Cmd+Shift+Z on macOS) - these are the standard editor keys, answered by the notebook while focus is inside it
- Settings: notebook.undoRedoPerCell
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\undoRedo\notebookUndoRedo.ts; V\src\vs\workbench\contrib\notebook\browser\viewModel\notebookViewModelImpl.ts; V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\profile\notebookProfile.ts; V\src\vs\editor\browser\editorExtensions.ts; C bundle (workbench.contrib.notebookUndoRedo, notebook.undoRedoPerCell, label "Whether to use separate undo/redo stack for each cell.")

### ED-7.42 See run progress on a cell and on the notebook
- Tag: both
- Evidence: source
- Boundary: no
- What: While a code cell runs, a thin animated progress line moves along the top of that cell (along the one-line placeholder when the cell's input is collapsed), and it hides while the cell is paused at a debugger frame. When a whole-notebook run is going, or a running cell has been scrolled out of view, the standard editor progress bar also shows across the top of the notebook so you can tell something is still running.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the contribution id workbench.notebook.executionEditorProgress is in the bundle; the per-cell bar has no id or label of its own, so only the editor-level part was matched by name)
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\execute\executionEditorProgress.ts; V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellProgressBar.ts; V\src\vs\workbench\contrib\notebook\browser\view\renderers\cellRenderer.ts; V\src\vs\workbench\contrib\notebook\browser\notebookEditorWidget.ts (showProgress, hideProgress); C bundle (workbench.notebook.executionEditorProgress)

### ED-7.43 See problems and find matches on the notebook scroll strip
- Tag: both
- Evidence: source
- Boundary: no
- What: A narrow strip (10 pixels wide, three lanes) at the right edge of the notebook's scroll area draws a mark for each hit at the place where it sits in the whole notebook, not just in one cell: errors and warnings from the Problems list in the right lane, and find matches in the middle lane (the current match in a second color, with matches inside outputs marked too). The notebook debug decorations for breakpoints and the paused cell draw on it as well (ED-11). It has no settings of its own; the text editor's version is ED-5.29.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (notebook-overview-ruler-container is in the bundle and the CSS, and the contribution id workbench.notebook.markerDecoration is in the bundle)
- Source: V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookOverviewRuler.ts; V\src\vs\workbench\contrib\notebook\browser\notebookEditorWidget.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\marker\markerProvider.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\find\findMatchDecorationModel.ts; V\src\vs\workbench\contrib\notebook\browser\contrib\debug\notebookDebugDecorations.ts; V\src\vs\workbench\contrib\notebook\browser\media\notebook.css; C bundle (notebook-overview-ruler-container, workbench.notebook.markerDecoration)

### ED-7.44 Step through problems across cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Inside a cell editor, Go to Next Problem and Go to Previous Problem treat the whole notebook as one file, so Alt+F8 moves from the last problem in one cell to the first problem in the next cell that has one, and Shift+Alt+F8 goes back the same way, instead of wrapping inside the cell. The inline problem widget and F8 across files are ED-4.27.
- Commands: editor.action.marker.next; editor.action.marker.prev
- Menus: MenubarGoMenu (Next Problem and Previous Problem, group 6_problem_nav)
- Keys: Alt+F8 (next); Shift+Alt+F8 (previous), when editor focus is in a cell
- Settings: none found
- Cursor: present (the contribution id workbench.contrib.markerListProvider and both command ids are in the bundle)
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\marker\markerProvider.ts (MarkerListProvider); V\src\vs\editor\contrib\gotoError\browser\markerNavigationService.ts; V\src\vs\editor\contrib\gotoError\browser\gotoError.ts; C bundle (workbench.contrib.markerListProvider, editor.action.marker.next, editor.action.marker.prev)

### ED-7.45 Highlight matching text across cells
- Tag: both
- Evidence: source
- Boundary: no
- What: Select a piece of text on one line in a cell and every copy of that exact text (case sensitive, no whole-word rule, single-line selections only) is shaded in all the cells of the notebook; put the cursor on a name and the other uses of the same symbol are also marked in the other cells that are currently drawn. In a plain file both stay inside that file (ED-5.33 and ED-5.34).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.selectionHighlight; editor.occurrencesHighlight
- Cursor: present (the contribution id notebook.selectionHighlighter and both settings are in the bundle)
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\multicursor\notebookSelectionHighlight.ts; V\src\vs\editor\contrib\wordHighlighter\browser\wordHighlighter.ts (notebook case in the model list); C bundle (notebook.selectionHighlighter, editor.selectionHighlight, editor.occurrencesHighlight)

### ED-7.46 Read a rendered markdown cell
- Tag: both
- Evidence: source
- Boundary: no
- What: A markdown cell that is not being edited is drawn by the built-in markdown-it renderer: headings get anchors and a link that starts with # scrolls the notebook to that heading, web addresses become links, fenced code blocks are syntax colored by their language tag, raw HTML is allowed (but cleaned of unsafe tags and scripts while the workspace is not trusted), and an empty cell shows the line "Empty markdown cell, double-click or press enter to edit." The same renderer also draws outputs whose type is markdown, LaTeX, JSON or a source-code type such as text/x-python (75 types in all). Font size, line height and font family are set by notebook settings.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: notebook.markup.fontSize; notebook.markdown.lineHeight; notebook.markup.fontFamily
- Cursor: present (the built-in extension declares the renderer vscode.markdown-it-renderer with the same 75 types and ships notebook-out\index.js; the bundle has notebook-cell-markup-empty-content and vscode-code-block, and the empty-cell label is in nls.messages.json)
- Source: V\extensions\markdown-language-features\package.json; V\extensions\markdown-language-features\notebook\index.ts; V\src\vs\workbench\contrib\notebook\browser\view\renderers\webviewPreloads.ts (anchor scrolling, code block highlighting); V\src\vs\workbench\contrib\notebook\browser\view\renderers\backLayerWebView.ts (empty cell text); V\src\vs\workbench\contrib\notebook\browser\notebook.contribution.ts; C\extensions\markdown-language-features\package.json; C\extensions\markdown-language-features\notebook-out\index.js; C bundle (notebook-cell-markup-empty-content, vscode-code-block, data-vscode-code-block-lang)

### ED-7.47 Keep or undo the agent's changes to a notebook from the breadcrumb row
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: When the agent has edited an open notebook, a small group appears in the notebook's breadcrumb row (or beside the tab label when breadcrumbs sit inline in the tab bar): green and red counts of lines added and removed, an "Undo Notebook" button that rejects the changes, and a primary "Keep Notebook" button that accepts them, with its key shown on it. Boundary: the changes come from the agent, and changes from worktree agent sessions are left out. It needs the breadcrumbs to be visible.
- Commands: editor.action.inlineDiffs.acceptAll (only used to look up the key shown on Keep Notebook; the buttons call the diff change registry directly)
- Menus: none found
- Keys: Ctrl+Enter, shown on Keep Notebook (the key of editor.action.inlineDiffs.acceptAll as listed in ED-6.29)
- Settings: breadcrumbs.enabled
- Cursor: cursor only
- Source: C bundle (notebook contribution id notebook.breadcrumbReview, attribute data-notebook-breadcrumb-review-controls, button labels "Undo Notebook" and "Keep Notebook", class breadcrumbs-action-btn, command id editor.action.inlineDiffs.acceptAll, change source speculative-full-file)

### ED-7.48 Select all text in a cell output
- Tag: both
- Evidence: source
- Boundary: no
- What: With keyboard focus inside a notebook cell's output, Ctrl+A selects all the text of that output instead of all the cells.
- Commands: notebook.cell.output.selectAll
- Menus: none found
- Keys: Ctrl+A (Cmd+A on macOS), when the notebook editor is focused and a cell output is focused
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\notebook\browser\contrib\clipboard\notebookClipboard.ts (lines 593-625); C bundle (id notebook.cell.output.selectAll with CtrlCmd+A and the same two context keys)

## Handed off

- Notebook Variables view (id workbench.notebook.variables, registered in the Run and Debug container) - SB-5. The toolbar button that opens it is ED-7.18.
- The Outline view container, its title bar and sort and follow options - SB-7. The notebook-specific entries and filters are ED-7.31.
- Notebook breadcrumbs (cells in the breadcrumb path, setting notebook.breadcrumbs.showCodeCells, toggle breadcrumbs.toggleFromEditorTitle) - ED-5.
- Notebook status bar items (Notebook Kernel Info, Select Kernel, "Cell N of M", Notebook Indentation with notebook.selectIndentation) - SS-3.
- Cell breakpoints, the paused-cell highlight and the debug stack in notebooks (V\src\vs\workbench\contrib\notebook\browser\contrib\debug) - ED-11.
- Cell comment threads (V\src\vs\workbench\contrib\notebook\browser\view\cellParts\cellComments.ts) - ED-11.
- Go to Symbol in a notebook (setting notebook.gotoSymbols.showAllSymbols) - QA-3.
- The "notebooks" walkthrough (category notebooks, step notebookProfile), opened by workbench.notebook.layout.select and notebook.experimental.openGettingStarted, both missing in Cursor - ED-9.
- How a notebook diff gets opened (Compare With, a Git change, Timeline) and the merge editor input registered for notebook types - ED-6 and SB-4.
- Markdown preview math (markdown.math.enabled, markdown.math.macros, the preview styles from markdown-math) - ED-10.
- Notebook accessibility help and accessible view (notebookAccessibilityHelp.ts, notebookAccessibleView.ts, replEditorAccessibleView.ts, settings accessibility.replEditor.*) - unowned in the catalog (the catalog lists the accessible view as unowned).
- The text editor twin of ED-7.47: "Undo File" and "Keep File" buttons in the breadcrumb row (C bundle, attribute data-breadcrumb-review-controls, same class breadcrumbs-action-btn) - ED-6. No feature in ED-6 named those labels when this was checked.
- Tab and tooltip titles for a notebook cell or a cell output opened as its own editor, written "{0} • Cell {1}" and "{0} • Cell {1} • Output {2}" (V\src\vs\workbench\browser\labels.ts; the same strings are in Cursor's nls.messages.json) - ED-1.

## Unclear

- notebook.quitEditAllCells (Stop Editing All Cells) is in V editActions.ts, but Cursor has neither its id nor its label (checked the bundle, nls.keys.json and nls.messages.json). The pass 1 list of 13 missing notebook ids does not include it.
- The Interactive Window and the REPL editor can be opened only by extension code (_interactive.open has f1 false; the repl editor resolver uses a blank pattern, so no file name opens it). I listed what the person sees and does once they are open. The open step itself is not a feature here.
- notebook.cellOutput.openInOutputPreview needs the setting notebook.output.openInPreviewEditor.enabled, whose registration is commented out in V notebook.contribution.ts, so the menu entry shows only if the key is added to settings.json by hand. N removed the setting. The Notebook Output Editor pane (workbench.editor.notebookOutputEditor) is not in the Cursor bundle.
- notebook.experimental.enableNewDiffEditor is read in two V files but registered in none, so the second Notebook Diff Editor is opt-in by hand. notebook.diff.experimental.toggleInline defaults on only when product quality is not stable, so its default in Cursor is unverified.
- replNotebook.input.execute is a constant in V notebookCommon.ts with no registration found. Cursor has no string for it. N removed it.
- The keybinding list.find.replInputFocus in repl.contribution.ts targets the Debug Console view (REPL_VIEW_ID), not the REPL editor, so it belongs to PN-6 and is not listed.
- ED-7.40 evidence is `source` because I read the extension manifest in the Cursor install folder directly (the brief only offers source or bundle). The manifest has no main entry. Whether the extension is enabled by default in Cursor was not verified, and I did not run Cursor. Cursor also carries the notebook keymap service that asks "Disable other keymaps (...) to avoid conflicts between keybindings?" when a notebook keymap extension is installed or enabled. That prompt comes from extension management, so I left it out.
- ED-7.28: notebook.setProfile has no screen of its own in V. It runs only with an argument, from the walkthrough step. Cursor has the command but not the entry that opens the walkthrough.
- Left out as agent surfaces: notebook cell chat (notebook.cell.chat.start, startAtTop, notebook.inlineChat.acceptChangesAndRun), Add Cell Output to Chat (notebook.cellOutput.addToChat), Fix and Explain Cell Error (notebook.cell.chat.fixError, explainError), notebook.chat.selectAndInsertKernelVariable, vscode.notebook.variable, the empty cell chat hint, and the notebook inline diff decorators used by chat editing (menu ChatEditingDeletedNotebookCell).
- Developer-only commands not listed: Toggle Notebook Layout Troubleshoot, Inspect Notebook Layout, Clear Notebook Editor Type Cache, Toggle Notebook View Zones, Toggle Notebook Clipboard Troubleshooting, Reset Notebook Webview, Reset notebook getting started.
- Nothing newer than 1.128 was found. The notebook, interactive and replNotebook folders in N have the same command and setting ids as V (N dropped a few old or disabled ids) and one new file, cellEditorOverflowWidgets.ts, which is not a feature.
- Cursor labels live in nls.messages.json (the bundle holds message indexes), so for labels I searched that file and for ids I searched the bundle. I read the keys from V source and did not run either app, so key differences in Cursor are not ruled out.
- ED-7.47 (Cursor only) was read from minified code in the bundle, not from source. The conditions I report (breadcrumbs visible, agent changes with source speculative-full-file, worktree sessions excluded) come from that code and I did not run Cursor to see the buttons. Its text editor twin is handed off to ED-6, which may want to merge the two into one feature.
- ED-7.41, ED-7.43, ED-7.44 and ED-7.45 are notebook versions of behavior whose editor-level feature is owned elsewhere (undo and redo, ED-5.29, ED-4.27, ED-5.33 and ED-5.34). I kept them because the notebook code that widens each one to all cells lives in the notebook folders and the cross-cell result is not described by those features.
- Two small items stay inside existing features and were not given ids: the setting workbench.editor.languageDetectionHints (key notebookEditors) turns the "Accept Detected Language" lightbulb in ED-7.7 on or off, and the cell status bar shows a warning icon for an unknown cell language that links to an extension search (extension search is out of scope).
- notebook.webHost.confirm (the prompt "Do you trust the authors of this notebook?") runs only in a serverless web build (isWeb without a remote authority), so it is not a desktop feature and is not listed. It is not in the Cursor bundle.
- Overflow check on 2026-10-25 added 7 features (ED-7.41 to ED-7.47).
