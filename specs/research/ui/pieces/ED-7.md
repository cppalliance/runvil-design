Group: ED-7 Notebooks, Interactive Window and REPL. Pass: 3. Date: 2026-10-25.

### ED-7.1
1. Choosing File > New File and picking the Jupyter Notebook entry, or running `ipynb.newUntitledIpynb` from the Command Palette, opens an untitled notebook that starts with one empty Python code cell.
2. Opening an `.ipynb` file from the Explorer or Quick Open shows it in the notebook editor as a list of cells instead of raw JSON.
3. When an `.ipynb` is showing as plain text, a CodeLens on it offers "Open in Notebook Editor", and clicking it (`ipynb.openIpynbInNotebookEditor`) switches the tab to the notebook editor.
4. When a notebook cannot be loaded, the editor area shows an error message with an Open As Text action (`workbench.notebook.action.openAsText`) that reopens the file in the text editor.
5. When a notebook is too large to display, the error says it is very large (with its size when known) and offers Open in Text Editor (`workbench.notebook.action.openInTextEditor`).
6. When no extension handles the notebook type, the error offers "Install extension for" or "Enable extension for" that type, depending on whether the extension is already installed.

### ED-7.2
1. Running Insert Code Cell Above or Insert Code Cell Below (`notebook.cell.insertCodeCellAbove`, `notebook.cell.insertCodeCellBelow`) adds a new empty code cell next to the focused cell.
2. Running Insert Markdown Cell Above or Insert Markdown Cell Below (`notebook.cell.insertMarkdownCellAbove`, `notebook.cell.insertMarkdownCellBelow`) adds a new empty markdown cell next to the focused cell.
3. With a cell selected but no text box focused, Ctrl+Enter inserts a code cell below and Ctrl+Shift+Enter inserts one above (Cmd instead of Ctrl on macOS).
4. Hovering the gap between two cells shows "+ Code" and "+ Markdown" buttons that insert a cell at that spot.
5. The top of the notebook shows the same two buttons above the first cell, and they insert a cell before it (`notebook.cell.insertCodeCellAtTop`, `notebook.cell.insertMarkdownCellAtTop`).
6. The Code and Markdown buttons on the notebook toolbar and the Insert Cell submenu in the cell menu also insert a cell.
7. `notebook.cell.insertCodeCellAboveAndFocusContainer` and `notebook.cell.insertCodeCellBelowAndFocusContainer` insert a code cell and leave focus on the cell itself rather than inside its text editor.
8. Setting `notebook.insertToolbarLocation` chooses where the insert buttons appear, with values `betweenCells`, `notebookToolbar`, `both` (default) and `hidden`.

### ED-7.3
1. Pressing Delete (Cmd+Backspace on macOS) with a cell focused and no text box active removes that cell from the notebook.
2. The trash icon on the cell toolbar deletes the cell it belongs to.
3. Each entry in the Interactive Window transcript has its own delete button that removes that entry.
4. Deleting a cell that is still running opens a question dialog "This cell is running, are you sure you want to delete it?" with a Delete button and a "Do not ask me again" checkbox.
5. Setting `notebook.confirmDeleteRunningCell` (default on) controls whether that dialog appears, and ticking "Do not ask me again" sets it to off.

### ED-7.4
1. Pressing Alt+Up or Alt+Down moves the focused cell up or down one place (`notebook.cell.moveUp`, `notebook.cell.moveDown`).
2. The cell menu lists Move Cell Up and Move Cell Down only when drag and drop is turned off.
3. Dragging a cell by its gutter handle and dropping it between other cells moves it to that place.
4. Holding Ctrl (Alt on macOS) while dropping a dragged cell copies it instead of moving it.
5. Setting `notebook.dragAndDropEnabled` (default on) turns dragging cells on or off.

### ED-7.5
1. With a cell focused and no text box active, Ctrl+C copies the whole cell and Ctrl+X cuts it out of the notebook (`notebook.cell.copy`, `notebook.cell.cut`).
2. Ctrl+V pastes the copied cell below the focused cell (`notebook.cell.paste`).
3. Ctrl+Shift+V pastes the copied cell above the focused cell (`notebook.cell.pasteAbove`).
4. Alt+Shift+Up and Alt+Shift+Down duplicate the focused cell above or below it (`notebook.cell.copyUp`, `notebook.cell.copyDown`).
5. The cell "..." menu lists Cut Cell, Copy Cell, Paste Cell and Copy Cell Down, and Paste Cell Above and Copy Cell Up are reached by key or Command Palette.
6. With the text cursor inside a cell's text editor, the same clipboard keys copy and paste text instead of whole cells.
7. On Windows, Ctrl+Insert, Shift+Delete and Shift+Insert also copy, cut and paste cells.

### ED-7.6
1. Pressing Ctrl+K Ctrl+Shift+\ inside an editable cell splits it at the text cursor into two cells (`notebook.cell.split`).
2. The split button on the cell toolbar splits the cell at the text cursor.
3. Pressing WinCtrl+Alt+J joins the focused cell with the cell after it (`notebook.cell.joinBelow`; WinCtrl is Ctrl on macOS and the Windows key elsewhere).
4. Pressing WinCtrl+Alt+Shift+J joins the focused cell with the cell before it (`notebook.cell.joinAbove`).
5. Join Selected Cells (`notebook.cell.joinSelected`) merges all selected cells into a single cell.
6. The cell "..." menu lists Join With Previous Cell, Join With Next Cell and Join Selected Cells so they can be used without the keys.

### ED-7.7
1. The cell "..." menu entries Change Cell to Markdown and Change Cell to Code switch the cell's type (`notebook.cell.changeToMarkdown`, `notebook.cell.changeToCode`).
2. With a cell focused and no text box active, pressing Y turns it into a code cell and pressing M turns it into a markdown cell.
3. Clicking the language label on the cell status bar opens a Select Cell Language Mode list, and choosing a language sets the cell's language.
4. Pressing Ctrl+K M with a cell focused opens the same language list (`notebook.cell.changeLanguage`).
5. When the editor has detected a different language for a cell, an Accept Detected Language action switches the cell to it (`notebook.cell.detectLanguage`).

### ED-7.8
1. Pressing Enter on a focused rendered markdown cell switches it to editing its source (`notebook.cell.edit`).
2. The cell toolbar of a rendered markdown cell shows an Edit Cell icon, and while editing it shows a Stop Editing Cell icon.
3. Pressing Escape, or WinCtrl+Enter (Ctrl+Alt+Enter on Windows), while editing a markdown cell renders it again and returns focus to the cell (`notebook.cell.quitEdit`).
4. Running Render All Markdown Cells (`notebook.renderAllMarkdownCells`) switches every markdown cell that is being edited back to its rendered view.
5. The Stop Editing All Cells command (`notebook.quitEditAllCells`) renders every markdown cell that is being edited in one step, but it exists only in stock VS Code. [stock only]

### ED-7.9
1. Pressing Up on the first line of a cell editor or Down on its last line moves focus to the previous or next cell (`notebook.focusPreviousEditor`, `notebook.focusNextEditor`).
2. Pressing Ctrl+Home jumps to the first cell and Ctrl+End jumps to the last (`notebook.focusTop`, `notebook.focusBottom`).
3. Pressing PageUp or PageDown moves the cursor up or down by a page through the cells (`notebook.cell.cursorPageUp`, `notebook.cell.cursorPageDown`).
4. Pressing Shift+Up or Shift+Down extends the cell selection by one cell, and Shift+PageUp or Shift+PageDown extends it by a page (`list.expandSelectionUp`, `list.expandSelectionDown`, `notebook.cell.cursorPageUpSelect`, `notebook.cell.cursorPageDownSelect`).
5. Pressing Ctrl+L scrolls so the active cell sits in the middle of the view (`notebook.centerActiveCell`).
6. Pressing Ctrl+Down on Windows (Ctrl+Shift+Down elsewhere) moves focus into the focused cell's output, and Ctrl+Shift+Up moves it back out (`notebook.cell.focusInOutput`, `notebook.cell.focusOutOutput`).
7. Setting `notebook.navigation.allowNavigateToSurroundingCells` (default on) decides whether Up and Down at the first or last line move into the neighbouring cell.

### ED-7.10
1. With one or more cells selected and no text box active, pressing Ctrl+/ toggles line comments over all the lines of every selected cell (`notebook.commentSelectedCells`).
2. Pressing Ctrl+/ again on cells that are already commented removes the comment markers.
3. The key is active only in an editable notebook, so a read-only notebook is left unchanged.

### ED-7.11
1. Collapse Cell Input hides a code cell's code and leaves a one-line placeholder, and Expand Cell Input shows the code again (`notebook.cell.collapseCellInput`, `notebook.cell.expandCellInput`).
2. Collapse Cell Output hides a cell's output behind a placeholder reading "Outputs are collapsed", and Expand Cell Output shows it again (`notebook.cell.collapseCellOutput`, `notebook.cell.expandCellOutput`).
3. Double-clicking a collapsed input or output placeholder expands it, and the output placeholder's tooltip names the key.
4. Pressing Ctrl+K Ctrl+C on a focused cell toggles its input between collapsed and expanded.
5. Pressing Ctrl+K T on a focused cell toggles its output between collapsed and expanded (`notebook.cell.toggleOutputs`).
6. Collapse All Cell Inputs and Expand All Cell Inputs change the input of every cell at once (`notebook.cell.collapseAllCellInputs`, `notebook.cell.expandAllCellInputs`).
7. Collapse All Cell Outputs and Expand All Cell Outputs change the output of every cell at once (`notebook.cell.collapseAllCellOutputs`, `notebook.cell.expandAllCellOutputs`).

### ED-7.12
1. A markdown cell that starts with a heading shows a folding arrow, and clicking it hides the cells under that heading until the next heading of the same or higher level.
2. A folded heading shows a hint reading "N cells hidden" ("1 cell hidden" when there is one).
3. Pressing Ctrl+Shift+[ folds the section and Ctrl+Shift+] unfolds it (Cmd+Alt+[ and Cmd+Alt+] on macOS), using the commands Fold Cell and Unfold Cell (`notebook.fold`, `notebook.unfold`).
4. The outline entry menu offers Fold Section and Expand Section (`notebook.section.foldSection`, `notebook.section.expandSection`).
5. A Run Cells In Section button on the markdown heading cell's toolbar, in the outline entry menu and in the sticky scroll menu runs every code cell under that heading (`notebook.section.runCells`).
6. A code cell's entry in the outline shows a Run Cell button that runs just that cell (`notebook.section.runSingleCell`).
7. Setting `notebook.showFoldingControls` chooses when the folding arrow shows, with values `always`, `never` (also narrows the gutter) and `mouseover` (default).

### ED-7.13
1. Clicking the play button in a code cell's gutter runs that cell (`notebook.cell.execute`).
2. With the cell list focused, WinCtrl+Enter (Ctrl+Alt+Enter on Windows) runs the focused cell and leaves focus on the cell itself (`notebook.cell.execute`, `notebook.cell.executeAndFocusContainer`).
3. Shift+Enter runs the focused cell and moves focus to the next cell, and on the last cell it adds a new cell below first (`notebook.cell.executeAndSelectBelow`).
4. Alt+Enter runs the focused cell and inserts a new cell below it (`notebook.cell.executeAndInsertBelow`).
5. While a cell is pending or running, its play button turns into a stop button that stops it (`notebook.cell.cancelExecution`, Stop Cell Execution).
6. Setting `notebook.scrolling.revealNextCellOnExecute` chooses how far the view scrolls to the next cell after Shift+Enter, with values `fullCell` (default), `firstLine` and `none`.
7. Setting `notebook.consolidatedRunButton` (default off) moves the extra run actions into a dropdown next to the run button.

### ED-7.14
1. Run All on the notebook toolbar runs every code cell in order (`notebook.execute`).
2. Execute Above Cells runs every cell above the focused one, and Execute Cell and Below runs the focused cell and all the cells after it (`notebook.cell.executeCellsAbove`, `notebook.cell.executeCellAndBelow`).
3. While anything is running, the Run All button is replaced by a Stop Execution button that stops the whole run (`notebook.cancelExecution`).
4. For kernels that support it, the replacement button reads Interrupt and interrupts the kernel instead (`notebook.interruptExecution`).
5. The Interactive Window toolbar shows an Interrupt button while code is running.
6. With `notebook.globalToolbar` off, Run All and the stop or interrupt button move to the editor title bar.
7. Setting `notebook.consolidatedRunButton` (default off) puts the run-above and run-below entries in a dropdown beside the cell's run button.

### ED-7.15
1. The Go To button on the notebook toolbar opens a menu with Go to Running Cell and Go to Most Recently Failed Cell.
2. Go to Running Cell scrolls to the cell that is running, or to the cell stopped at a debugger frame (`notebook.revealRunningCell`).
3. Go to Most Recently Failed Cell scrolls to the last cell whose run failed (`notebook.revealLastFailedCell`).
4. The Go To button shows a spinning icon while a cell runs and an error icon after a cell has failed.
5. With `notebook.globalToolbar` off, the Go To button moves to the editor title bar, and the Interactive Window toolbar shows it as well.

### ED-7.16
1. A thin bar under each code cell shows a status icon for success, failure, pending or executing.
2. After a run, the bar shows the execution count and how long the run took, with more details in a hover.
3. With `verbose` execution time detail, the bar also shows when the cell last ran.
4. The language label on the right of the bar shows the cell's language, and clicking it opens the language picker (`notebook.cell.changeLanguage`).
5. Items added by extensions appear in the bar next to the built-in ones.
6. Setting `notebook.showCellStatusBar` chooses when the bar shows, with values `hidden`, `visible` (default) and `visibleAfterExecute`.
7. Setting `notebook.cellExecutionTimeVerbosity` chooses how much run time detail shows, with values `default` and `verbose`.

### ED-7.17
1. After a code cell fails, a Quick Actions item appears in its status bar.
2. Clicking Quick Actions (`notebook.cell.openFailureActions`) opens a menu of code actions for the error.
3. Diagnostics for the failure are marked in the cell's code.
4. Setting `notebook.cellFailureDiagnostics` (default on) turns those diagnostics on or off.

### ED-7.18
1. With inline values on, running a code cell shows each variable's value as faded text after the variable in the cell's code.
2. Setting `notebook.inlineValues` chooses when inline values show, with values `on` (uses a text-matching fallback when no provider exists), `auto` (only when a provider exists) and `off` (default).
3. Inline values stay until the cell is edited or run again, and Clear All Outputs or Notebook: Clear Inline Values removes them at once (`notebook.clearAllInlineValues`).
4. A Variables button on the notebook toolbar, the editor title and the Interactive Window toolbar opens the Notebook Variables view when the kernel can list variables (`notebook.openVariablesView`).
5. Setting `notebook.variablesView` (default off) turns the Notebook Variables view on.

### ED-7.19
1. The Select Kernel button on the notebook toolbar, the editor title and the Interactive Window toolbar shows the chosen kernel's name, and clicking it opens a kernel picker (`_notebook.selectKernel`).
2. While kernels are still being looked for, the button reads "Detecting Kernels".
3. The picker lists suggested and recently used kernels, and marks the current one "Currently Selected".
4. The picker also lists kernel sources and a "Select Another Kernel..." entry that opens a second list headed "Type to choose a kernel source".
5. The picker offers "Install/Enable suggested extensions" and "Browse marketplace for kernel extensions".
6. Choosing a kernel source opens a list headed "Select Kernel from" that source, and extension actions for sources can appear there.
7. Running Clear Notebook Kernels MRU cache from the Command Palette forgets the recently used kernels (`notebook.clearNotebookKernelsMRUCache`).

### ED-7.20
1. Each code cell's output appears directly below the cell.
2. Plain text and the stdout and stderr streams are drawn as lines of text.
3. Images and HTML outputs are drawn in place by the built-in renderers.
4. An error output shows its traceback with a Show Details link that turns into Hide Details, and file paths in it can be clicked.
5. A cell with no output shows "Cell has no output", and an output no renderer can draw shows a "No renderer could be found" message that lists its mime types.
6. Setting `notebook.output.textLineLimit` (default 30) sets how many lines of text output show before it is cut.
7. Settings `notebook.output.lineHeight`, `notebook.output.fontSize` and `notebook.output.fontFamily` set the look of output text, and 0 or blank uses the editor's own value.
8. Setting `notebook.output.wordWrap` (default off) chooses whether long output lines wrap.
9. Settings `notebook.output.linkifyFilePaths` (default on) and `notebook.output.minimalErrorRendering` (default off) turn file path links and a minimal error style on or off.

### ED-7.21
1. A "..." button beside an output opens a menu of output actions.
2. Copy Output in that menu copies the output to the clipboard (`notebook.cellOutput.copy`).
3. Show Empty Outputs reveals outputs that have no visible content (`notebook.cellOuput.showEmptyOutputs`).
4. Change Presentation opens a mime type picker that also offers more renderers from the marketplace (`notebook.output.pickMimetype`).
5. Save Mimetype Display Order asks for User Settings or Workspace Settings and writes the chosen mime type order to the `notebook.displayOrder` setting there (`notebook.saveMimeTypeOrder`).
6. Setting `notebook.consolidatedOutputButton` (default on) chooses whether the output actions are gathered under the "..." button.
7. Save Image in the output menu saves an image output to a file (`notebook.cellOutput.saveImage`). [stock only]
8. Open in Output Preview opens an output in its own Output Preview editor, and only after `notebook.output.openInPreviewEditor.enabled` is added to settings by hand (`notebook.cellOutput.openInOutputPreview`). [stock only]

### ED-7.22
1. Text output longer than the line limit is cut and followed by "Output is truncated. View as a scrollable element or open in a text editor. Adjust cell output settings..."
2. Clicking "scrollable element" shows that output in a scroll box in place (`cellOutput.enableScrolling`).
3. Clicking "text editor" opens the full raw output in a text editor (`notebook.cellOutput.openInTextEditor`).
4. Clicking "settings..." opens the notebook output layout settings.
5. Pressing Ctrl+K Y on a focused cell with output toggles that output between cut text and a scroll box (`notebook.cell.toggleOutputScrolling`).
6. A cell with more outputs than the output limit shows a "show more" link that opens the raw output data in a text editor (`workbench.action.openLargeOutput`).
7. Setting `notebook.output.scrolling` makes long outputs start out as scroll boxes, and it is off by default in stable builds.
8. With scrolling on, `notebook.output.textLineLimit` sets the height of the scroll box in lines and `notebook.output.wordWrap` chooses whether long lines wrap inside it.

### ED-7.23
1. Pressing Alt+Delete on a focused cell clears that cell's output (`notebook.cell.clearOutputs`).
2. The Clear Cell Outputs entry in the cell "..." menu and in the output "..." menu clears the output of that cell.
3. Clear All Outputs on the notebook toolbar clears the output of every cell (`notebook.clearAllCellsOutputs`).
4. With `notebook.globalToolbar` off, Clear All Outputs moves to the editor title bar.
5. The Alt+Delete key is active only when the cell has output and the notebook is editable.
6. Setting `notebook.consolidatedOutputButton` (default on) chooses whether the output actions, including clear, are gathered in the output toolbar.

### ED-7.24
1. Math written between single dollar signs in a markdown cell is drawn as a typeset equation inside the line of text.
2. Math written between double dollar signs in a markdown cell is drawn as a typeset equation block of its own.
3. Macros defined for the markdown preview are not applied in notebook cells, because the notebook uses no user macros.

### ED-7.25
1. Pasting an image from the clipboard into a markdown cell of an `.ipynb` file offers the choice "Insert Image as Attachment".
2. Dropping an image file onto a markdown cell of an `.ipynb` file offers the same "Insert Image as Attachment" choice.
3. Choosing it stores the image inside the cell, inserts an `attachment:` image reference into the cell text, and the rendered cell shows the image.
4. When a markdown cell refers to an attachment that is not in the cell, a warning marks the reference and the quick fix "Remove invalid image attachment reference" removes it (`ipynb.cleanInvalidImageAttachment`).
5. Setting `ipynb.pasteImagesAsAttachments.enabled` (default on) turns the paste and drop choice on or off.

### ED-7.26
1. A toolbar across the top of the notebook shows the Code and Markdown add buttons on its left, followed by Run All, Clear All Outputs, Go To and Variables.
2. The right side of the toolbar holds the Customize Notebook Layout entry and the kernel picker.
3. Setting `notebook.globalToolbar` (default on) shows or hides the toolbar, and with it off the run and kernel buttons move to the editor title bar.
4. Setting `notebook.globalToolbarShowLabel` chooses whether toolbar buttons show text labels, with values `always` (default), `never` and `dynamic` (labels drop away when the toolbar is too narrow).
5. Right-clicking the toolbar opens a context menu of toolbar options, which includes the Sticky Scroll toggle.
6. Setting `notebook.insertToolbarLocation` decides whether the Code and Markdown buttons show on this toolbar, with `notebookToolbar` and `both` showing them and `betweenCells` and `hidden` not.

### ED-7.27
1. Each cell shows a small toolbar of icon buttons for editing a markdown cell, splitting, deleting and running a section (`NotebookCellTitle`).
2. A "..." button on the cell toolbar opens an overflow menu grouped into copy, insert, edit, view and share entries.
3. The cell toolbar stays visible at the top of a tall cell while you scroll through the cell.
4. Setting `notebook.cellToolbarLocation` places the toolbar per file type, with values `left`, `right` (default) and `hidden`.
5. Setting `notebook.cellToolbarVisibility` chooses whether the toolbar appears when the cell is hovered or clicked, with values `hover` and `click` (default).
6. Toggle Cell Toolbar Position switches the toolbar between the left and right side (`notebook.toggleCellToolbarPosition`).

### ED-7.28
1. The "Customize Notebook..." gear on the editor title opens a menu of layout options.
2. The Line Numbers entry in that menu toggles line numbers in every cell and shows a check while they are on (`notebook.toggleLineNumbersFromEditorTitle`, `notebook.toggleLineNumbers`).
3. The cell menu has a Toggle Line Numbers entry that turns line numbers on or off for just that cell (`notebook.cell.toggleLineNumbers`).
4. The gear menu also has a Breadcrumbs toggle and a Toggle Cell Toolbar Position entry (`breadcrumbs.toggleFromEditorTitle`, `notebook.toggleCellToolbarPositionFromEditorTitle`).
5. Customize Notebook Layout, in the gear menu and on the notebook toolbar, opens the Settings editor filtered to notebook layout settings (`workbench.notebook.layout.configure`, `workbench.notebook.layout.configure.editorTitle`).
6. Running Set Notebook Profile with `default`, `jupyter` or `colab` applies a bundle of layout settings at once (`notebook.setProfile`).
7. Settings `notebook.lineNumbers` (`off` default, `on`), `notebook.compactView` (default on) and `notebook.cellFocusIndicator` (`border`, `gutter` default) choose line numbers, compact margins and where the focus marker shows.
8. Settings `notebook.markup.fontSize`, `notebook.markdown.lineHeight` and `notebook.markup.fontFamily` set the font size, line height and font family of rendered markdown cells.
9. Setting `notebook.editorOptionsCustomizations` overrides chosen `editor.*` options inside notebook cell editors only.
10. Select between Notebook Layouts opens the notebook layouts walkthrough and is switched on by `notebook.experimental.openGettingStarted`, but Cursor has neither. [stock only]

### ED-7.29
1. With sticky scroll on, the markdown headings of the section you are in stay pinned at the top of the notebook while you scroll.
2. Setting `notebook.stickyScroll.mode` chooses how nested pinned headings stack, with values `flat` and `indented` (default).
3. Clicking a pinned heading jumps to that heading's cell.
4. Right-clicking a pinned heading opens a menu with Run Cells In Section and a Sticky Scroll toggle to turn the feature off.
5. Toggle Notebook Sticky Scroll in the Command Palette or the toolbar context menu turns the pinned headings on or off (`notebook.action.toggleNotebookStickyScroll`).
6. Setting `notebook.stickyScroll.enabled` (experimental, default off) turns sticky scroll on.

### ED-7.30
1. Pressing Ctrl+F with the notebook focused, and not inside a cell editor, opens a Find box over the notebook that searches all the cells (`notebook.find`).
2. The Find box has a Find input with match case, whole word and regular expression toggles.
3. The Previous Match and Next Match buttons step through the matches, which are highlighted in cell text and in outputs with the current match in a different color.
4. Pressing F3 or Shift+F3 (Cmd+G or Cmd+Shift+G on macOS) while the Find box has focus steps to the next or previous match, but Cursor lacks these two commands. [stock only]
5. A Find Filters menu has checks for Markdown Source, Rendered Markdown, Code Cell Source and Code Cell Output to limit what is searched.
6. Setting `notebook.find.filters` stores those four filters as `markupSource`, `markupPreview`, `codeSource` and `codeOutput`, all on by default.
7. A Find in Selection toggle limits the search to the selected cells.
8. A Toggle Replace button reveals a Replace input with Replace and Replace All buttons, and Ctrl+H opens it, but the Interactive Window has no replace row.
9. Pressing Escape or clicking Close hides the Find box (`notebook.hideFind`).

### ED-7.31
1. The Outline view lists the notebook's markdown headings as a tree, and clicking an entry jumps to that cell.
2. The Outline view's Filter Entries menu has a Markdown Headers Only toggle that is on by default (`notebook.outline.toggleShowMarkdownHeadersOnly`).
3. The Code Cells toggle adds the notebook's code cells to the outline and is off by default (`notebook.outline.toggleCodeCells`).
4. The Code Cell Symbols toggle adds the symbols inside code cells, is on by default, and only matters while Code Cells is on (`notebook.outline.toggleCodeCellSymbols`).
5. Each outline entry has run and fold buttons next to it.
6. Settings `notebook.outline.showMarkdownHeadersOnly` (on), `notebook.outline.showCodeCells` (off) and `notebook.outline.showCodeCellSymbols` (on) store those three toggles.

### ED-7.32
1. Setting `notebook.multiCursor.enabled` (experimental, default off) turns multi-cursor editing across cells on.
2. Pressing Ctrl+D adds the next match of the current word, in this cell or a later one, as another cursor (`notebook.addFindMatchToSelection`).
3. Pressing Ctrl+Shift+L selects every match of the current word across all the cells (`notebook.selectAllFindMatches`).
4. Typing, Backspace and Delete then change the text at every cursor in every cell at once (`noteMultiCursor.deleteLeft`, `noteMultiCursor.deleteRight`).
5. Cut, copy and paste also act on every cursor at once.
6. Pressing Escape leaves multi-cursor mode (`noteMultiCursor.exit`, Exit Multi Cursor Mode).

### ED-7.33
1. Format Notebook formats the code in every cell, and Shift+Alt+F (Ctrl+Shift+I on Linux) runs it when no cell editor has text focus (`notebook.format`).
2. Format Cell formats only the focused cell, with Shift+Alt+F (Ctrl+Shift+I on Linux) inside a cell editor or from the right-click menu (`notebook.formatCell`).
3. Setting `notebook.formatOnSave.enabled` (default off) formats the whole notebook when it is saved.
4. Setting `notebook.formatOnCellExecution` (default off) formats a cell each time it is run.
5. Setting `notebook.defaultFormatter` picks the formatter extension used for notebooks, and it overrides the other formatter settings.
6. Setting `notebook.insertFinalNewline` (default off) adds a final new line to the end of code cells on save.
7. Setting `notebook.codeActionsOnSave` lists the code action kinds that run each time the notebook is saved.

### ED-7.34
1. Comparing two versions of an `.ipynb` file opens the Notebook Diff Editor, which shows the notebooks cell by cell, with the changed input, outputs and metadata of each cell in their own sections.
2. An Open File button on the editor title opens the notebook itself in the notebook editor (`notebook.diff.openFile`).
3. An Open Text Diff Editor button on the editor title switches to the plain text diff of the file (`notebook.diff.switchToText`).
4. Setting `notebook.diff.overviewRuler` (default off) draws an overview ruler beside the diff.
5. Setting `notebook.diff.enablePreview` (default on) allows the Notebook Diff Editor, which is used only when no screen reader is detected.
6. The opt-in second Notebook Diff Editor, switched on by `notebook.experimental.enableNewDiffEditor`, lists the changed cells like the multi-file diff, with a Go To Cell button on each one (`notebook.diffEditor.2.goToCell`).

### ED-7.35
1. The diff editor title menu has a Show Outputs Differences entry that shows or hides output differences, and it follows `notebook.diff.ignoreOutputs` (`notebook.diff.showOutputs`).
2. The diff editor title menu has a Show Metadata Differences entry that shows or hides metadata differences, and it follows `notebook.diff.ignoreMetadata` (`notebook.diff.showMetadata`).
3. A cell's input header can collapse the unchanged regions inside that cell, which follows `diffEditor.hideUnchangedRegions.enabled` (`notebook.diff.cell.toggleCollapseUnchangedRegions`).
4. Another entry shows or hides differences that are only whitespace (`notebook.toggle.diff.cell.ignoreTrimWhitespace`).
5. A cell's output header switches that output between rendered and text view (`notebook.diff.cell.switchOutputRenderingStyleToText`).
6. In the multi-file style diff, entries show or hide the cells that did not change (`notebook.diffEditor.showUnchangedCells`, `notebook.diffEditor.hideUnchangedCells`).
7. An experimental toggle switches the diff to an inline view, and it appears when `notebook.diff.experimental.toggleInline` is on (`notebook.diff.inline.toggle`).

### ED-7.36
1. Pressing Alt+F3 in the Notebook Diff Editor jumps to the next change and Shift+Alt+F3 jumps to the previous one (`notebook.diff.action.next`, `notebook.diff.action.previous`).
2. The editor title shows Show Next Change and Show Previous Change buttons that do the same.
3. A Revert Input button on a cell's input section header puts that cell's input back to the original (`notebook.diff.cell.revertInput`).
4. A Revert Outputs button on a cell's outputs section header puts the outputs back to the original (`notebook.diff.cell.revertOutputs`).
5. A Revert Metadata button on a cell's metadata section header puts the metadata back to the original (`notebook.diff.cell.revertMetadata`).
6. A revert button on the notebook-level metadata header puts the whole notebook's metadata back to the original (`notebook.diff.revertMetadata`).
7. In the multi-file style diff, each changed cell's toolbar has the same three revert buttons (`notebook.diffEditor.2.cell.revertInput`, `notebook.diffEditor.2.cell.revertOutputs`, `notebook.diffEditor.2.cell.revertMetadata`).

### ED-7.37
1. Typing code in the input box under the Interactive Window transcript and pressing Ctrl+Enter, or clicking the run button beside the box, runs the code (`interactive.execute`).
2. Setting `interactiveWindow.executeWithShiftEnter` (default off) chooses the other run key, where off makes Enter run the code and on makes Shift+Enter run it and Enter add a new line.
3. An empty input box shows the hint "Press <key> to execute." naming the run key, and `interactiveWindow.showExecutionHint` (default on) turns the hint off.
4. Pressing Up on the first line of the input box or Down on its last line, with the suggest list hidden, brings back earlier or later inputs from history (`interactive.history.previous`, `interactive.history.next`).
5. The clear command empties the input box (`interactive.input.clear`).
6. Pressing Ctrl+Up moves focus from the input box to the transcript history, and Focus Input Editor moves it back (`interactive.history.focus`, `interactive.input.focus`).

### ED-7.38
1. The transcript above the input box shows each piece of code that was run as a cell, in the order it ran, with its output below.
2. Pressing Ctrl+Home scrolls the transcript to the top and Ctrl+End scrolls it to the bottom (Cmd+Up and Cmd+Down on macOS) (`interactive.scrollToTop`, `interactive.scrollToBottom`).
3. Setting `interactiveWindow.alwaysScrollOnNewCell` (default on) scrolls to the output of the last run, and with it off the window scrolls only if the last cell was already in view.
4. Setting `interactiveWindow.collapseCellInputCode` chooses whether code cells start collapsed, with values `always`, `never` and `fromEditor` (default).
5. The window toolbar shows the kernel picker, an Interrupt button while code runs, the Go To button and the Variables button (`InteractiveToolbar`).
6. Each transcript cell has its own title buttons, a delete button and a run button (`InteractiveCellTitle`, `InteractiveCellDelete`, `InteractiveCellExecute`).
7. Setting `interactiveWindow.promptToSaveOnClose` (default off) asks to save the window when it is closed, and it affects only newly opened windows.

### ED-7.39
1. The REPL editor shows what has been run as read-only cells above an input box.
2. Pressing Ctrl+Enter or clicking the run button runs the input, and Enter runs it too unless `interactiveWindow.executeWithShiftEnter` is on, in which case Shift+Enter runs it (`repl.execute`).
3. An empty input box shows the hint "Press <key> to execute.", and `interactiveWindow.showExecutionHint` (default on) turns it off.
4. Pressing Alt+End twice moves focus to the most recent execution (`repl.focusLastItemExecuted`, Focus Most Recent REPL Execution).
5. Pressing Ctrl+Down from a history cell, or Alt+Home twice, moves focus back to the input box (`repl.input.focus`, Focus Input Editor).
6. Setting `interactiveWindow.alwaysScrollOnNewCell` (default on) keeps the view scrolled to the latest result.
7. The REPL editor shows the kernel buttons of the Interactive Window toolbar (`InteractiveToolbar`).

### ED-7.40
1. With a cell focused and no text box or output focused, pressing F opens the notebook find box (`notebook.find`). [cursor only]
2. Pressing J moves focus to the next cell and K moves it to the previous cell (`list.focusDown`, `list.focusUp`). [cursor only]
3. Pressing A inserts a code cell above and B inserts one below, leaving focus on the new cell (`notebook.cell.insertCodeCellAboveAndFocusContainer`, `notebook.cell.insertCodeCellBelowAndFocusContainer`). [cursor only]
4. Pressing D twice deletes the focused cell (`notebook.cell.delete`). [cursor only]
5. Pressing Z undoes the last change and S saves the notebook (`undo`, `workbench.action.files.save`). [cursor only]
6. Pressing C copies, X cuts and V pastes the focused cell (`notebook.cell.copy`, `notebook.cell.cut`, `notebook.cell.paste`). [cursor only]
7. Pressing Shift+L toggles line numbers for all cells and L toggles them for the focused cell (`notebook.toggleLineNumbers`, `notebook.cell.toggleLineNumbers`). [cursor only]
8. Pressing O collapses or expands the focused cell's output (`notebook.cell.toggleOutputs`). [cursor only]
9. Pressing Ctrl+Shift+- while typing in a cell splits the cell at the text cursor (`notebook.cell.split`). [cursor only]
10. Pressing Ctrl+Enter runs the focused cell and keeps focus on it, and Shift+Enter on a code cell runs it and selects the next one (`notebook.cell.executeAndFocusContainer`, `notebook.cell.executeAndSelectBelow`). [cursor only]

### ED-7.41
1. Pressing Ctrl+Z while focus is inside the notebook undoes the last notebook change, including adding, deleting or moving cells (`undo`).
2. Pressing Ctrl+Y or Ctrl+Shift+Z (Cmd+Shift+Z on macOS) while focus is inside the notebook redoes the change (`redo`).
3. After an undo or redo the cell that changed is revealed, and a markdown cell that changed goes back into edit mode.
4. The Edit menu entries Undo and Redo and the Command Palette entries do the same while the notebook has focus.
5. Setting `notebook.undoRedoPerCell` chooses whether each cell keeps its own typing history (on, default) or all cells share one history so Ctrl+Z walks back through edits across cells in the order they were made (off).
6. Applying the `jupyter` layout profile turns per-cell history on, while the `default` and `colab` profiles turn it off.

### ED-7.42
1. While a code cell runs, a thin animated progress line moves along the top of that cell.
2. When the running cell's input is collapsed, the progress line moves along its one-line placeholder instead.
3. The progress line hides while the cell is paused at a debugger frame.
4. While a whole-notebook run is going, the standard editor progress bar shows across the top of the notebook.
5. When a running cell has been scrolled out of view, the same editor progress bar shows so you can tell something is still running.

### ED-7.43
1. A narrow strip, 10 pixels wide with three lanes, runs down the right edge of the notebook's scroll area.
2. Errors and warnings from the Problems list appear as marks in the right lane, placed by where they sit in the whole notebook rather than in one cell.
3. Find matches appear as marks in the middle lane, with the current match in a second color and matches inside outputs marked too.
4. The strip has no settings of its own.

### ED-7.44
1. Pressing Alt+F8 inside a cell editor moves to the next problem, and from the last problem in one cell it goes to the first problem in the next cell that has one (`editor.action.marker.next`).
2. Pressing Shift+Alt+F8 inside a cell editor goes back the same way across cells instead of wrapping inside the cell (`editor.action.marker.prev`).
3. The Go menu entries Next Problem and Previous Problem do the same while a cell editor has focus.

### ED-7.45
1. Selecting text on one line in a cell shades every identical copy of that text in all the cells of the notebook, matching case and ignoring whole word rules, and multi-line selections are not shaded.
2. Putting the cursor on a name marks the other uses of the same symbol in the other cells that are currently drawn.
3. Setting `editor.selectionHighlight` turns the shading of selected text on or off.
4. Setting `editor.occurrencesHighlight` turns the symbol marks off with `off`, and in a notebook the other cells are marked with either `singleFile` or `multiFile`.

### ED-7.46
1. A markdown cell that is not being edited is drawn as rendered markdown by the built-in markdown-it renderer.
2. Headings get anchors, and a link that starts with # scrolls the notebook to that heading.
3. Web addresses in the text become clickable links.
4. Fenced code blocks are syntax colored by their language tag.
5. Raw HTML is allowed, but unsafe tags and scripts are removed while the workspace is not trusted.
6. An empty markdown cell shows the line "Empty markdown cell, double-click or press enter to edit."
7. The same renderer draws outputs of type markdown, LaTeX, JSON and source-code types such as `text/x-python`.
8. Settings `notebook.markup.fontSize` (0 uses 120 percent of the editor font size), `notebook.markdown.lineHeight` (0 uses normal) and `notebook.markup.fontFamily` (blank uses the workbench font) set the look of rendered markdown.

### ED-7.47
1. When the agent has edited an open notebook, a small group of controls appears in the notebook's breadcrumb row, or beside the tab label when breadcrumbs sit inline in the tab bar. [cursor only]
2. The group shows a green count of lines added and a red count of lines removed. [cursor only]
3. An "Undo Notebook" button in the group rejects the agent's changes to the notebook. [cursor only]
4. A primary "Keep Notebook" button in the group accepts the changes and shows its key, Ctrl+Enter, on the button. [cursor only]
5. The group appears only while breadcrumbs are visible, so turning `breadcrumbs.enabled` off hides it. [cursor only]
6. Changes made by worktree agent sessions do not show the group. [cursor only]

### ED-7.48
1. With focus in a cell's output (reached with the focus-in-output key from ED-7.9), `Ctrl+A` selects the whole text of that output.
2. When the focus is on a text box that lives inside the output, such as an input a renderer draws, `Ctrl+A` selects that box's contents instead.
3. When focus is in the notebook but not inside the output page, the command does nothing.

## Unclear

- ED-7.8: the feature's What line says Ctrl+Enter renders a markdown cell again, but its Keys line and V source give WinCtrl+Enter (Ctrl+Alt+Enter on Windows), and plain Ctrl+Enter is the insert code cell below key from ED-7.2, so the piece follows the source.
- ED-7.17: the failure menu also lists chat actions (Fix and Explain Cell Error), which are left out as boundary, so no piece covers them.
- ED-7.37 and ED-7.39: the Interactive Window and the REPL editor are opened only by extension code, so no piece covers opening them, as the feature says.
- ED-7.40: whether the Jupyter Keymap extension is on by default in Cursor was not verified, so these pieces assume it is active.
- ED-7.47: pieces come from the minified Cursor bundle only and were not seen running, and its text editor twin is handed off to ED-6.
- ED-7.19 piece 6 and ED-7.26 piece 5 describe extension-fed and context menus only at the level the feature gives, since their entries depend on installed extensions.
