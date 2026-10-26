Group: ED-6 Diffs, merges and change review. Section: Editor. Date: 2026-10-25. Pass: 2.
Features: 58.

Short names used in Source: V is the stock 1.128.0 tree, N is the 1.144.0 clone, C bundle is `workbench.desktop.main.js` in the Cursor install, C\extensions is the Cursor `extensions` folder, and NLS is `out\nls.messages.json` next to the bundle. Nothing was run on screen, so every Cursor finding comes from strings, class names and manifests. Features 1 to 27 are stock (tag both, plus one newer-than-1.128 item). Features 28 to 40 are Cursor only. Cursor features that rest on class names alone say so in their What line.

### ED-6.1 View a diff in the diff editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Open a diff to see two versions of a file next to each other, with changed lines and words tinted, markers in the gutter and an overview ruler, and an editable modified side; files that are not text open in the Binary Diff Editor or the Side by Side Editor instead.
- Commands: vscode.diff; vscode.changes; _workbench.diff
- Menus: none found
- Keys: none found
- Settings: diffEditor.renderIndicators; diffEditor.renderMarginRevertIcon; diffEditor.renderGutterMenu; diffEditor.codeLens; diffEditor.wordWrap; diffEditor.diffAlgorithm; diffEditor.maxComputationTime; diffEditor.maxFileSize; diffEditor.experimental.useTrueInlineView; diffEditor.experimental.showEmptyDecorations; editor.wordWrapOverride1
- Cursor: present (vscode.diff, vscode.changes, _workbench.diff and every listed setting found in C bundle; panes Text Diff Editor, Binary Diff Editor and Side by Side Editor found in NLS)
- Source: V\src\vs\workbench\browser\parts\editor\textDiffEditor.ts; V\src\vs\workbench\browser\parts\editor\binaryDiffEditor.ts; V\src\vs\workbench\browser\parts\editor\sideBySideEditor.ts; V\src\vs\workbench\browser\parts\editor\editorCommands.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditorWidget.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts; C bundle (diff editor options and settings); NLS (pane labels)

### ED-6.2 Switch between side-by-side and inline view
- Tag: both
- Evidence: source
- Boundary: no
- What: Use the Inline View toggle in the editor title to show the diff as one column with removed lines above added ones, or turn it back to two columns; a second toggle makes the diff drop to inline on its own when the editor gets narrow.
- Commands: toggle.diff.renderSideBySide; diffEditor.toggleUseInlineViewWhenSpaceIsLimited
- Menus: EditorTitle (Inline View in group 1_diff order 10; Use Inline View When Space Is Limited in group 1_diff order 11, shown only once the narrow breakpoint is reached); CommandPalette
- Keys: none found
- Settings: diffEditor.renderSideBySide; diffEditor.useInlineViewWhenSpaceIsLimited; diffEditor.renderSideBySideInlineBreakpoint
- Cursor: changed (both commands, the two menu items and the first two settings found; the setting id diffEditor.renderSideBySideInlineBreakpoint is not in the bundle, only an internal option with default 900)
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (line 423); V\src\vs\workbench\browser\parts\editor\diffEditorCommands.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditor.contribution.ts; V\src\vs\editor\browser\widget\diffEditor\commands.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts; C bundle (toggle ids, option renderSideBySideInlineBreakpoint)

### ED-6.3 Ignore leading and trailing whitespace changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Click the whitespace button in the diff editor title (labelled Show Leading/Trailing Whitespace Differences) to stop or start counting lines that differ only by spaces at the ends as changes.
- Commands: toggle.diff.ignoreTrimWhitespace
- Menus: EditorTitle (navigation group, order 20, only in a text compare editor)
- Keys: none found
- Settings: diffEditor.ignoreTrimWhitespace
- Cursor: present (toggle.diff.ignoreTrimWhitespace, diffEditor.ignoreTrimWhitespace and the label found)
- Source: V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 682-694); V\src\vs\workbench\browser\parts\editor\diffEditorCommands.ts; C bundle; NLS (label)

### ED-6.4 Collapse and reveal unchanged regions
- Tag: both
- Evidence: source
- Boundary: no
- What: Fold the long stretches of identical lines in a diff into small "show hidden lines" bars, then reveal one stretch by clicking its bar, or reveal or fold all of them with a command; the title button toggles the whole behavior.
- Commands: diffEditor.toggleCollapseUnchangedRegions; diffEditor.collapseAllUnchangedRegions; diffEditor.showAllUnchangedRegions
- Menus: EditorTitle (navigation group, order 22, map icon); CommandPalette (Collapse All Unchanged Regions and Show All Unchanged Regions, category Diff Editor)
- Keys: none found
- Settings: diffEditor.hideUnchangedRegions.enabled; diffEditor.hideUnchangedRegions.minimumLineCount; diffEditor.hideUnchangedRegions.contextLineCount; diffEditor.hideUnchangedRegions.revealLineCount
- Cursor: present (all three commands and all four settings found in C bundle)
- Source: V\src\vs\editor\browser\widget\diffEditor\commands.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditor.contribution.ts; V\src\vs\editor\common\config\editorConfigurationSchema.ts; C bundle

### ED-6.5 Show moved code blocks
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn on Show Moved Code Blocks to have the diff draw a link between a block of lines that was cut from one place and pasted in another, click it to compare the two places, and press Escape to leave that comparison.
- Commands: diffEditor.toggleShowMovedCodeBlocks; diffEditor.exitCompareMove
- Menus: EditorTitle (Show Moved Code Blocks, group 1_diff order 10, move icon)
- Keys: Escape (diffEditor.exitCompareMove, only while comparing moved code)
- Settings: diffEditor.experimental.showMoves
- Cursor: present (both commands, the setting and the label found)
- Source: V\src\vs\editor\browser\widget\diffEditor\commands.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditor.contribution.ts; V\src\vs\editor\browser\widget\diffEditor\features\movedBlocksLinesFeature.ts; C bundle; NLS (label)

### ED-6.6 Go to the next or previous change in a diff
- Tag: both
- Evidence: source
- Boundary: no
- What: Step through the changes in a diff with the up and down arrows in the editor title or the keys, or open the Accessible Diff Viewer to read the changes one at a time in a list with F7.
- Commands: workbench.action.compareEditor.nextChange; workbench.action.compareEditor.previousChange; editor.action.accessibleDiffViewer.next; editor.action.accessibleDiffViewer.prev; editor.action.diffReview.next; editor.action.diffReview.prev; diffreview.close
- Menus: EditorTitle (Previous Change order 10 and Next Change order 11 when the diff has changes; Open Accessible Diff Viewer in group 2_diff); CommandPalette (Go to Next Change, Go to Previous Change, and the Accessible Diff Viewer pair)
- Keys: Alt+F5 (next change) and Shift+Alt+F5 (previous change) when a text compare editor is visible; F7 (next difference) and Shift+F7 (previous difference) in a diff editor
- Settings: none found
- Cursor: present (all six ids and the labels Previous Change, Next Change and Open Accessible Diff Viewer found)
- Source: V\src\vs\workbench\browser\parts\editor\diffEditorCommands.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 622-652); V\src\vs\editor\browser\widget\diffEditor\commands.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditor.contribution.ts; C bundle; NLS (labels)

### ED-6.7 Revert a block or selection from the diff
- Tag: both
- Evidence: source
- Boundary: no
- What: When the right side is editable, a gutter button between the two sides reverts one changed block, or the selected lines, back to the left side's text.
- Commands: diffEditor.revert
- Menus: DiffEditorHunkToolbar (Revert Block); DiffEditorSelectionToolbar (Revert Selection); CommandPalette (category Diff Editor)
- Keys: none found
- Settings: diffEditor.renderGutterMenu; diffEditor.renderMarginRevertIcon
- Cursor: present (diffEditor.revert, both toolbar menu ids, both settings and the labels Revert Block and Revert Selection found)
- Source: V\src\vs\editor\browser\widget\diffEditor\commands.ts; V\src\vs\editor\browser\widget\diffEditor\diffEditor.contribution.ts; V\src\vs\editor\browser\widget\diffEditor\features\gutterFeature.ts; C bundle; NLS (labels)

### ED-6.8 Stage, unstage or revert the selected lines
- Tag: both
- Evidence: source
- Boundary: no
- What: In a Git diff, stage or unstage just the selected lines or one block from the right-click menu, the editor title overflow or the gutter buttons, or revert the selected lines in the working file.
- Commands: git.stageSelectedRanges; git.unstageSelectedRanges; git.revertSelectedRanges; git.diff.stageHunk; git.diff.stageSelection
- Menus: editor/context; editor/title; diffEditor/gutter/hunk; diffEditor/gutter/selection
- Keys: Ctrl+K Ctrl+Alt+S (stage selected ranges, text focus on a file); Ctrl+K Ctrl+N (unstage selected ranges, right side of a diff of an index file); Ctrl+K Ctrl+R (revert selected ranges, text focus on a file)
- Settings: none found
- Cursor: present (the five ids are in C\extensions\git\package.json with the same menus and keys; the toolbar menu ids found in C bundle)
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\src\vs\editor\browser\widget\diffEditor\features\gutterFeature.ts; C\extensions\git\package.json

### ED-6.9 Switch, swap or open a side of the diff
- Tag: both
- Evidence: source
- Boundary: no
- What: Move the cursor to the matching spot on the other side, jump focus to the left or right side, swap the two sides, or open the active side as its own editor.
- Commands: diffEditor.switchSide; workbench.action.compareEditor.focusOtherSide; workbench.action.compareEditor.focusPrimarySide; workbench.action.compareEditor.focusSecondarySide; workbench.action.compareEditor.swapSides; workbench.action.compareEditor.openSide
- Menus: EditorTitle (Swap Left and Right Side, order 15); CommandPalette (Swap Left and Right Editor Side, Open Active Diff Side, Switch Side in category Diff Editor)
- Keys: Ctrl+K Ctrl+Shift+O (open the active side, in a diff editor)
- Settings: none found
- Cursor: present (all six ids and the label Swap Left and Right Side found)
- Source: V\src\vs\workbench\browser\parts\editor\diffEditorCommands.ts; V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (lines 654-664); V\src\vs\editor\browser\widget\diffEditor\commands.ts; C bundle; NLS (labels)

### ED-6.10 Compare the active file with its saved version
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Compare Active File with Saved to open a diff of your unsaved edits against what is on disk, or use Compare with Saved on a dirty editor in the Open Editors list.
- Commands: workbench.files.action.compareWithSaved
- Menus: CommandPalette (category File); OpenEditorsContext (group 3_compare, dirty editors, auto save off)
- Keys: Ctrl+K D
- Settings: none found
- Cursor: present (command id and both labels found)
- Source: V\src\vs\workbench\contrib\files\browser\fileConstants.ts; V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle; NLS (labels)

### ED-6.11 Compare two files
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick a second file from a list with Compare Active File With..., or mark one file with Select for Compare and then use Compare with Selected on another, or select exactly two files and use Compare Selected; each opens a diff.
- Commands: workbench.files.action.compareFileWith; selectForCompare; compareFiles; compareSelected
- Menus: ExplorerContext; OpenEditorsContext (group 3_compare); EditorTitleContext (group 1_compare); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present (all four ids and the labels Select for Compare, Compare with Selected, Compare Selected and Compare Active File With... found)
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (line 499); V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileConstants.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle; NLS (labels)

### ED-6.12 Compare the active file with the clipboard
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Compare Active File with Clipboard to see a diff between the open file and whatever text is on the clipboard.
- Commands: workbench.files.action.compareWithClipboard
- Menus: CommandPalette (category File)
- Keys: Ctrl+K C
- Settings: none found
- Cursor: present (command id and label found)
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (line 823); C bundle; NLS (label)

### ED-6.13 Review several files in the multi-file diff editor
- Tag: both
- Evidence: source
- Boundary: no
- What: See the changes of many files in one scrolling editor, each file as a collapsible diff under its own header; Source Control's View Changes, a stash and a commit open it, and the editor title has a Collapse All Diffs or Expand All Diffs button.
- Commands: _workbench.openMultiDiffEditor; _workbench.openScmMultiDiffEditor; multiDiffEditor.collapseAll; multiDiffEditor.expandAll
- Menus: EditorTitle (collapse or expand, navigation group); CompactWindowEditorTitle; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (all four ids and the labels Collapse All Diffs and Expand All Diffs found; Cursor also registers the setting multiDiffEditor.experimental.enabled, default true, which stock 1.128.0 does not have)
- Source: V\src\vs\workbench\contrib\multiDiffEditor\browser\multiDiffEditor.contribution.ts; V\src\vs\workbench\contrib\multiDiffEditor\browser\actions.ts; V\src\vs\workbench\contrib\multiDiffEditor\browser\scmMultiDiffSourceResolver.ts; V\src\vs\workbench\browser\parts\editor\editorCommands.ts; V\extensions\git\src\commands.ts; C bundle (multi diff actions and the multiDiffEditor.experimental.enabled registration); NLS (labels)

### ED-6.14 Jump from the multi-file diff to a file or the next change
- Tag: both
- Evidence: source
- Boundary: no
- What: Use a file header's Open File button to jump to the real file, or step to the next or previous change across all the files in the diff.
- Commands: multiDiffEditor.goToFile; multiDiffEditor.goToNextChange; multiDiffEditor.goToPreviousChange
- Menus: MultiDiffEditorFileToolbar (Open File); EditorTitle (arrows, navigation group); CompactWindowEditorTitle; CommandPalette
- Keys: Alt+F5 (next change) and Shift+Alt+F5 (previous change) when the active editor is the multi-file diff
- Settings: none found
- Cursor: changed (multiDiffEditor.goToFile and the MultiDiffEditorFileToolbar menu id found; multiDiffEditor.goToNextChange and multiDiffEditor.goToPreviousChange not found in C bundle)
- Source: V\src\vs\workbench\contrib\multiDiffEditor\browser\actions.ts; V\src\vs\workbench\contrib\multiDiffEditor\browser\multiDiffEditor.contribution.ts; C bundle (goToFile, MultiDiffEditorFileToolbar; searched for the two next and previous ids and found none)

### ED-6.15 Stage or unstage a file from the multi-file diff
- Tag: both
- Evidence: source
- Boundary: no
- What: Each file header in a Git multi-file diff has a button that stages the file if it is a working tree or untracked change, or unstages it if it is staged.
- Commands: git.stageFile; git.unstageFile
- Menus: multiDiffEditor/resource/title
- Keys: none found
- Settings: none found
- Cursor: present (both commands and the same three menu items are in C\extensions\git\package.json)
- Source: V\extensions\git\package.json; C\extensions\git\package.json

### ED-6.16 Show a file list beside the multi-file diff
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A file list beside the multi-file diff can be toggled and focused, shown as a flat list or a tree, and placed on the left or the right.
- Commands: multiDiffEditor.toggleFileList; multiDiffEditor.focusFileList; multiDiffEditor.viewFileListAsList; multiDiffEditor.viewFileListAsTree; multiDiffEditor.positionFileListLeft; multiDiffEditor.positionFileListRight
- Menus: EditorTitle (toggle button; a submenu with groups 1_view and 2_position, read from the unit test)
- Keys: none found
- Settings: multiDiffEditor.experimental.fileList
- Cursor: missing (searched C bundle for the six ids and the setting; the only multiDiffEditor ids found are collapseAll, expandAll, goToFile and experimental.enabled)
- Source: N\src\vs\workbench\contrib\multiDiffEditor\browser\actions.ts; N\src\vs\workbench\contrib\multiDiffEditor\browser\multiDiffEditorWithFileListWidget.ts; N\src\vs\workbench\contrib\multiDiffEditor\common\multiDiffEditor.ts; N\src\vs\workbench\contrib\multiDiffEditor\test\browser\actions.test.ts; C bundle (negative search)

### ED-6.17 Open the merge editor for a conflicted file
- Tag: both
- Evidence: source
- Boundary: no
- What: A file with Git conflicts shows a Resolve in Merge Editor button at the top of the editor that opens the three-way merge editor; with git.mergeEditor turned on (default off) conflicted files open there directly.
- Commands: git.openMergeEditor; _open.mergeEditor
- Menus: editor/content (group navigation, order -10)
- Keys: none found
- Settings: git.mergeEditor
- Cursor: present (git.openMergeEditor and git.mergeEditor are in C\extensions\git\package.json, _open.mergeEditor found in C bundle; the same editor/content menu also holds a chat button, git.resolveAllConflictsInChat, which is a chat hand-off and not listed)
- Source: V\extensions\git\package.json; V\extensions\git\src\commands.ts; V\src\vs\workbench\contrib\mergeEditor\browser\commands\commands.ts; V\src\vs\workbench\contrib\mergeEditor\browser\mergeEditor.contribution.ts; C\extensions\git\package.json; C bundle

### ED-6.18 Read a merge in three panes and change what it shows
- Tag: both
- Evidence: source
- Boundary: no
- What: The merge editor shows Input 1 and Input 2 above a Result pane (Mixed Layout) or as three columns (Column Layout); Show Base adds the common ancestor, Show Non-Conflicting Changes marks changes that do not clash, each input has a Compare With Base button, and Open File and Open Base File open the plain files.
- Commands: merge.mixedLayout; merge.columnLayout; merge.showBase; merge.showBaseTop; merge.showBaseCenter; merge.showNonConflictingChanges; mergeEditor.compareInput1WithBase; mergeEditor.compareInput2WithBase; merge.openResult; merge.openBaseEditor
- Menus: EditorTitle (groups navigation, 1_merge, 2_merge, 3_merge); MergeInput1Toolbar; MergeInput2Toolbar; MergeBaseToolbar
- Keys: none found
- Settings: mergeEditor.diffAlgorithm; mergeEditor.showDeletionMarkers
- Cursor: present (all ten ids and both settings found in C bundle; the labels Mixed Layout, Column Layout, Show Base, Open Base File, Compare Input 1 With Base and Text Merge Editor found in NLS)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\commands.ts; V\src\vs\workbench\contrib\mergeEditor\browser\mergeEditor.contribution.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\mergeEditor.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\baseCodeEditorView.ts; C bundle; NLS (labels)

### ED-6.19 Accept a change from either side of a conflict
- Tag: both
- Evidence: source
- Boundary: no
- What: Tick the Accept box beside a change in Input 1 or Input 2 to copy it into Result (tick both to combine them), use the actions drawn above a conflict in Result (Accept, Accept Combination, Append, Remove, Ignore, Reset to base), or toggle the current conflict from the left or right with a command.
- Commands: merge.toggleActiveConflictInput1; merge.toggleActiveConflictInput2
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (both commands found in C bundle; the label Accept Combination found in NLS; the Accept box and Ignore, Remove and Append actions were not individually searched)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\commands.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\conflictActions.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\inputCodeEditorView.ts; C bundle; NLS (Accept Combination)

### ED-6.20 Accept everything from one side, reset the result or recompute it
- Tag: both
- Evidence: source
- Boundary: no
- What: Use the buttons on an input's title to accept all of its changes into Result, use Reset in the Result title to go back to the base plus automatic merging, or run the palette commands that rebuild Result with Git's own merge.
- Commands: merge.acceptAllInput1; merge.acceptAllInput2; mergeEditor.acceptAllCombination; mergeEditor.resetResultToBaseAndAutoMerge; git.runGitMerge; git.runGitMergeDiff3
- Menus: MergeInput1Toolbar; MergeInput2Toolbar; MergeInputResultToolbar; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (all ids found except mergeEditor.acceptAllCombination, which is not in C bundle; the git ids are in C\extensions\git\package.json)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\commands.ts; V\extensions\git\package.json; V\extensions\git\src\commands.ts; C bundle; C\extensions\git\package.json

### ED-6.21 Move between conflicts and between the two inputs
- Tag: both
- Evidence: source
- Boundary: no
- What: Jump to the next or previous unhandled conflict from the editor title arrows, or flip focus between Input 1 and Input 2 with a key.
- Commands: merge.goToNextUnhandledConflict; merge.goToPreviousUnhandledConflict; mergeEditor.toggleBetweenInputs
- Menus: EditorTitle (navigation group); CommandPalette (category Merge Editor)
- Keys: Ctrl+Shift+T (toggle between inputs, in the merge editor)
- Settings: none found
- Cursor: changed (the two conflict commands and the label Go to Next Unhandled Conflict found; mergeEditor.toggleBetweenInputs not found in C bundle)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\commands.ts; C bundle; NLS (label)

### ED-6.22 Complete the merge
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Enter or use the Complete Merge button to save Result as the merged file; if conflicts are still unhandled a dialog offers Complete with Conflicts, and a palette command resets that remembered choice.
- Commands: mergeEditor.acceptMerge; git.acceptMerge; mergeEditor.resetCloseWithConflictsChoice
- Menus: editor/content (git.acceptMerge, when the Result editor belongs to a Git merge); CommandPalette (category Merge Editor)
- Keys: Ctrl+Enter (complete merge, in the merge editor)
- Settings: none found
- Cursor: present (all three ids found; the label Complete Merge found in NLS)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\commands.ts; V\extensions\git\package.json; C bundle; C\extensions\git\package.json; NLS (label)

### ED-6.23 Resolve conflict markers in the editor
- Tag: both
- Evidence: source
- Boundary: no
- What: A file with conflict markers gets tinted Current and Incoming blocks and a row of CodeLens above each conflict (Accept Current Change, Accept Incoming Change, Accept Both Changes, Compare Changes); previous and next conflict arrows sit in the editor title, and palette commands accept the conflict at the cursor or every conflict in the file as Current, Incoming or Both.
- Commands: merge-conflict.accept.current; merge-conflict.accept.incoming; merge-conflict.accept.both; merge-conflict.accept.selection; merge-conflict.accept.all-current; merge-conflict.accept.all-incoming; merge-conflict.accept.all-both; merge-conflict.next; merge-conflict.previous; merge-conflict.compare
- Menus: editor/title (previous and next, only while the file has conflicts); scm/resourceState/context (Accept All Current and Accept All Incoming on the Merge Changes group); CommandPalette
- Keys: none found
- Settings: merge-conflict.codeLens.enabled; merge-conflict.decorators.enabled; merge-conflict.autoNavigateNextConflict.enabled; merge-conflict.diffViewPosition
- Cursor: changed (all ten ids and all four settings are in C\extensions\merge-conflict\package.json, and the four CodeLens titles are in its dist file; Cursor adds an eleventh command merge-conflict.resolve-in-chat and a fifth CodeLens, Resolve with AI, both chat hand-offs that are not listed)
- Source: V\extensions\merge-conflict\package.json; V\extensions\merge-conflict\src\codelensProvider.ts; V\extensions\merge-conflict\src\commandHandler.ts; V\extensions\merge-conflict\src\mergeDecorator.ts; C\extensions\merge-conflict\package.json; C\extensions\merge-conflict\dist\mergeConflictMain.js

### ED-6.24 See changed lines in the gutter
- Tag: both
- Evidence: source
- Boundary: no
- What: Colored bars in the gutter mark lines added, modified or deleted since the Git base, and matching marks can show in the overview ruler and minimap; right-click the line numbers for a Diff Decorations submenu to hide a source.
- Commands: workbench.scm.action.toggleQuickDiffVisibility.<provider id> (built at run time, one per source)
- Menus: EditorLineNumberContext (submenu Diff Decorations, which holds MenuId SCMQuickDiffDecorations)
- Keys: none found
- Settings: scm.diffDecorations; scm.diffDecorationsGutterWidth; scm.diffDecorationsGutterVisibility; scm.diffDecorationsGutterPattern; scm.diffDecorationsIgnoreTrimWhitespace
- Cursor: changed (all five settings and the quick diff model and provider names found in C bundle; the Diff Decorations label, MenuId SCMQuickDiffDecorations and group 9_quickDiffDecorations are not in the bundle or NLS, so the per-source toggle submenu looks absent)
- Source: V\src\vs\workbench\contrib\scm\browser\quickDiffDecorator.ts; V\src\vs\workbench\contrib\scm\browser\quickDiff.contribution.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; C bundle (scm.diffDecorations settings, QuickDiffModelService; negative search for SCMQuickDiffDecorations); NLS (settings text; negative search for Diff Decorations)

### ED-6.25 Peek a change, step through changes, stage or revert it
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a gutter bar to open a small inline diff under the line, with Stage Change and Revert Change buttons, and use the next and previous keys or the Go menu to move from change to change; Escape closes the peek.
- Commands: editor.action.dirtydiff.next; editor.action.dirtydiff.previous; workbench.action.editor.nextChange; workbench.action.editor.previousChange; closeQuickDiff; git.stageChange; git.revertChange; git.unstageChange
- Menus: SCMChangeContext; scm/change/title; MenubarGoMenu (Next Change and Previous Change, group 7_change_nav)
- Keys: Alt+F3 (next change in the peek) and Shift+Alt+F3 (previous); Alt+F5 (next change) and Shift+Alt+F5 (previous) with text focus; Escape (close the peek)
- Settings: scm.diffDecorationsGutterAction
- Cursor: present (the dirtydiff ids, closeQuickDiff, the two workbench.action.editor ids, SCMChangeContext and scm.diffDecorationsGutterAction found in C bundle; git.stageChange and git.revertChange in C\extensions\git\package.json, which has no git.unstageChange, a stock item that is hidden anyway)
- Source: V\src\vs\workbench\contrib\scm\browser\quickDiffWidget.ts; V\extensions\git\package.json (scm/change/title); C bundle; C\extensions\git\package.json

### ED-6.26 Use the Git buttons in the editor title
- Tag: both
- Evidence: source
- Boundary: no
- What: The editor title of a changed file shows Open Changes, in a diff it shows Open File, and the overflow holds Stage Changes and Unstage Changes; a stash shown in an editor adds Apply Stash and Drop Stash.
- Commands: git.openChange; git.openFile; git.stage; git.unstage; git.stashApplyEditor; git.stashPopEditor; git.stashDropEditor
- Menus: editor/title
- Keys: none found
- Settings: none found
- Cursor: present (the same editor/title items are in C\extensions\git\package.json; Cursor adds two more there, git.commitMessageAccept and git.commitMessageDiscard, which belong to the commit message editor)
- Source: V\extensions\git\package.json (editor/title); V\extensions\git\src\commands.ts; C\extensions\git\package.json

### ED-6.27 See Git blame on the current line
- Tag: both
- Evidence: source
- Boundary: no
- What: Turn on Git Blame Editor Decoration to show the last commit's subject, author and age as faint text at the end of the cursor line, with a hover that has an Open Commit link; the status bar item for the same data is listed in the status bar groups.
- Commands: git.blame.toggleEditorDecoration; git.viewCommit
- Menus: CommandPalette
- Keys: none found
- Settings: git.blame.editorDecoration.enabled; git.blame.editorDecoration.template; git.blame.editorDecoration.disableHover; git.blame.ignoreWhitespace
- Cursor: changed (command, enabled, template and ignoreWhitespace found in C\extensions\git\package.json; git.blame.editorDecoration.disableHover is missing and Cursor adds the application setting git.blame.editorDecoration.forceDisabled)
- Source: V\extensions\git\package.json; V\extensions\git\src\blame.ts; V\extensions\git\src\commands.ts; C\extensions\git\package.json; C\extensions\git\dist\main.js

### ED-6.28 See the agent's edits as inline diffs in the code editor
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the diffs are the agent's pending edits drawn inside the ordinary text editor)
- What: When the agent edits a file you have open, the changes show inside the text editor itself, with deleted code in blocks between lines, added lines highlighted and a small widget beside each change (from class names, not seen on screen); Toggle Inline Diffs hides or shows them, and when too many changes would draw, a notification says they were suppressed and offers Disable Protection.
- Commands: workbench.action.toggleInlineDiffs; inlineDiff.tooManyChangesWarning
- Menus: CommandPalette (Toggle Inline Diffs, category View)
- Keys: none found
- Settings: cursor.inlineDiff.enablePerformanceProtection
- Cursor: cursor only (toggle command, the setting, context keys inlineDiffs.activeEditorWithDiffs and inlineDiffs.hasInlineDiffsInAnyEditor, and modules inlineDiffController, inlineDiffPartialWidget and inlineDiffDeletedCodeMargin found in C bundle)
- Source: C bundle (module inlineDiffController, NLS 1381 and 1382; layoutActions toggle class, NLS 3163 and 3090; class inlineDiffViewZone; composer inlineDiffSuppressionNotification, NLS 5318 and 5319)

### ED-6.29 Accept all of the agent's edits in a file
- Tag: cursor
- Evidence: bundle
- Boundary: yes (acts on the agent's pending edits)
- What: Press Ctrl+Enter, or run Accept Edits, to keep every pending inline edit in the active file at once.
- Commands: editor.action.inlineDiffs.acceptAll
- Menus: CommandPalette (editor action Accept Edits)
- Keys: Ctrl+Enter (editor text focus, when the active editor has inline diffs)
- Settings: none found
- Cursor: cursor only (id, label Accept Edits and the key read from the action registration in C bundle)
- Source: C bundle (action with label Accept Edits, kbOpts Ctrl+Enter, precondition inlineDiffs.activeEditorWithDiffs, calls acceptSuggestion)

### ED-6.30 Undo the agent's edits in a file or in every editor
- Tag: cursor
- Evidence: bundle
- Boundary: yes (acts on the agent's pending edits)
- What: Press Ctrl+Shift+Backspace, or run Undo Edits, to throw away every pending inline edit in the active file; Reject All Edits does the same in every open editor.
- Commands: editor.action.inlineDiffs.rejectAll; editor.action.inlineDiffs.rejectAllAcrossAllEditors
- Menus: CommandPalette (editor actions Undo Edits and Reject All Edits)
- Keys: Ctrl+Shift+Backspace (Undo Edits, editor text focus, when the active editor has inline diffs); Reject All Edits has a key expression written with a key code where a modifier is expected, so its real binding is not resolved
- Settings: none found
- Cursor: cursor only (both ids, both labels and the Undo Edits key found in C bundle)
- Source: C bundle (actions with labels Undo Edits and Reject All Edits; rejectAllAcrossAllEditors loops over every code editor and calls rejectSuggestion)

### ED-6.31 Accept or reject one of the agent's changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (acts on one pending agent edit)
- What: A small accept and reject widget follows the change under the mouse or cursor; Accept Partial Edit keeps just that change and Reject Partial Edit drops just that change.
- Commands: editor.action.inlineDiffs.acceptPartialEdit; editor.action.inlineDiffs.rejectPartialEdit
- Menus: CommandPalette (editor actions Accept Partial Edit and Reject Partial Edit)
- Keys: Ctrl+Shift+Y on Windows, Cmd+Y on macOS and Ctrl+Y on Linux (accept partial); Ctrl+N or Cmd+N (reject partial); both need editor text focus and inline diffs in the active editor
- Settings: none found
- Cursor: cursor only (both ids, labels and keys found in C bundle; the widget class acceptRejectPartialEditWidget found)
- Source: C bundle (actions with labels Accept Partial Edit and Reject Partial Edit; module inlineDiffPartialWidget with div.acceptRejectPartialEditWidget; the Windows flag read from the platform module)

### ED-6.32 Cancel the agent's edit while it is still being written
- Tag: cursor
- Evidence: bundle
- Boundary: yes (acts on an edit the agent is still generating)
- What: While a diff is still streaming into the file, Ctrl+Shift+Backspace runs Cancel Edits and stops it; it shares the key with Undo Edits but has the higher priority only while generation is running.
- Commands: editor.action.inlineDiffs.cancelEdits
- Menus: CommandPalette (editor action Cancel Edits)
- Keys: Ctrl+Shift+Backspace (editor text focus, only while a diff is actively generating)
- Settings: none found
- Cursor: cursor only (id, label Cancel Edits and the key found in C bundle)
- Source: C bundle (action with label Cancel Edits, precondition hasActivelyGeneratingDiff, weight 600 against 500 for Undo Edits, calls cancelGeneration)

### ED-6.33 Step through the agent's changes and changed files
- Tag: cursor
- Evidence: bundle
- Boundary: yes (moves between the agent's pending edits)
- What: Jump to the next or previous pending change in the file with Alt+J and Alt+K, and to the next or previous file that has pending changes with Alt+L and Alt+H.
- Commands: editor.action.inlineDiffs.nextChange; editor.action.inlineDiffs.previousChange; editor.action.inlineDiffs.nextDiffFile; editor.action.inlineDiffs.previousDiffFile
- Menus: CommandPalette (editor actions Go to Next Change, Go to Previous Change, Go to Next Diff File, Go to Previous Diff File)
- Keys: Alt+J (next change); Alt+K (previous change); Alt+L (next diff file); Alt+H (previous diff file); all need editor text focus, and the file keys need inline diffs in some editor
- Settings: none found
- Cursor: cursor only (all four ids, labels and keys found in C bundle)
- Source: C bundle (four actions with preconditions inlineDiffs.activeEditorWithDiffs and inlineDiffs.hasInlineDiffsInAnyEditor)

### ED-6.34 Review all of the agent's edits in the Multi File Edits editor
- Tag: cursor
- Evidence: bundle
- Boundary: yes (it lists the agent's pending edits across files)
- What: Run View All Changes to open a tab named Multi File Edits that stacks the inline diffs of every file with pending agent edits; the tab is not restored when the window reloads.
- Commands: editor.action.inlineDiffs.viewAllChanges
- Menus: CommandPalette (editor action View All Changes)
- Keys: none found
- Settings: none found
- Cursor: cursor only (the action, the pane inlineMultiDiffEditor with label Multi File Edits, input workbench.input.inlineMultiDiffEditor and the resource scheme inline-multi-diff-editor found in C bundle)
- Source: C bundle (action with label View All Changes opening an editor with a resources list; function createInlineMultiDiffEditorInput; pane registration labelled Multi File Edits; serializer that refuses to save the tab)

### ED-6.35 Review the agent's or the branch's changes in Review Changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (opened from the agent chat and from Agent Review, and in agent mode it lists the agent's edits)
- What: A pinned Review Changes tab shows every changed file as a collapsible diff with a file list; it can show all changes from the agent chat, or the diff of the branch against the default branch (labelled Diff with Default Branch or Diff with the branch name), or unstaged Git changes, and the Source Control view's Review Branch button opens it.
- Commands: workbench.scm.action.openReviewChangesPane
- Menus: SCMTitle (Review Branch, group navigation, order -900; hidden from the Command Palette)
- Keys: none found
- Settings: none found
- Cursor: cursor only (command, labels Review Branch, Diff with Default Branch and Diff with {0}, pane workbench.editor.reviewchanges, context key activeEditorIsReviewChanges and the modules ReviewChangesFileList and ReviewChangesSummaryHeader found in C bundle)
- Source: C bundle (action workbench.scm.action.openReviewChangesPane, NLS 9381 to 9383; ReviewChangesEditorInput with resource schemes review-changes, git-changes and agent-review; getMultiDiffItemsForGit with mode unstaged or diffWithMain; NLS 4349)

### ED-6.36 Keep or undo the agent's changes in Review Changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (acts on the agent's pending edits)
- What: The Review Changes header has a primary Keep All button (Keep when one change is left) and an Undo All button (Undo, then Confirm), which accept or reject the agent's pending edits in one step.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (the labels Keep All, Undo All, Keep, Undo and Confirm and a keep-all handler that calls diffChangeSourceRegistry.accept and saves files found in the Review Changes pane code; whether the same buttons also sit in the chat panel was not separated)
- Source: C bundle (Review Changes pane code, log label MultiDiffTabContent, handler Keep all)

### ED-6.37 Commit, push or open a pull request from Review Changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the pane belongs to the agent flow, and the pull request flow generates a commit message and branch)
- What: A primary button in Review Changes offers Commit, Commit and Push, Push or Create PR, and it is disabled with the messages "Commit is unavailable until there are reviewable changes" or "Install Git to enable committing from Review Changes".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (labels Commit, Commit & Push, Push and Create PR under the DIFF_TAB action names, the module ReviewChangesCommitButton and both disabled messages found in C bundle)
- Source: C bundle (action label table DIFF_TAB_COMMIT, DIFF_TAB_COMMIT_AND_PUSH, DIFF_TAB_PUSH, DIFF_TAB_CREATE_PR; ReviewChangesCommitButton.readHeadRevision; Create Pull Request dialog text)

### ED-6.38 Search in Review Changes and change how it shows diffs
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the editor is the agent review surface)
- What: Press Ctrl+F while Review Changes is active to find text across its diffs (F3 and Shift+F3 step through matches), and use the ellipsis menu to choose Unified View or Split View and to Expand All or Collapse All.
- Commands: actions.find; editor.action.nextMatchFindAction; editor.action.previousMatchFindAction
- Menus: none found
- Keys: Ctrl+F (find); F3 (next match, Cmd+G on macOS); Shift+F3 (previous match, Cmd+Shift+G on macOS); all enabled by the context key activeEditorIsReviewChanges
- Settings: none found
- Cursor: cursor only (the three find ids with the extra activeEditorIsReviewChanges context, the ellipsis menu items Unified View, Split View, Expand All and Collapse All, and the module ReviewChangesFindWidget found in C bundle)
- Source: C bundle (find actions with kbExpr containing activeEditorIsReviewChanges; ReviewChangesEllipsisMenu; reviewChangesService.setDiffViewMode)

### ED-6.39 Open File Blame for a file
- Tag: cursor
- Evidence: bundle
- Boundary: yes (behind the cursor_blame feature gate, default off, and a team setting; the commit side of the same feature shows agent contributions)
- What: Right-click in the editor, pick Cursor Blame, then Toggle file blame, to open a pinned view of the file with one row per line showing revision hash, author, message and time; hovering a row shows Author, Message and When with an Open in Cursor Blame link, and the header has an Open original file button and a find box.
- Commands: cursorBlame.openFileBlame
- Menus: EditorContext (submenu Cursor Blame, id EditorContextCursorBlame, group 12_cursor_blame, for files on disk); EditorContextCursorBlame; CommandPalette (Open File Blame, category Cursor Blame)
- Keys: none found
- Settings: cursor.blame.hoverDelay
- Cursor: cursor only (command, the submenu id and labels Toggle file blame and Cursor Blame, the pane classes cursor-file-blame-* and the hover setting found in C bundle and NLS; the Explorer right-click entry for the same command is SB-2.39)
- Source: C bundle (cursorBlame.openFileBlame with NLS 5375 and 5376; gate cursor_blame; isCursorBlameEnabledForTeam; EditorContextCursorBlame registration; CSS classes cursor-file-blame-header, cursor-file-blame-row, cursor-file-blame-hover); NLS 5384 to 5389

### ED-6.40 View a commit in the Cursor Blame editor
- Tag: cursor
- Evidence: bundle
- Boundary: yes (behind the cursor_blame feature gate, default off, and a team setting; the view has tabs for the chats and agent contributions behind a commit)
- What: Open a commit in a pinned Cursor Blame tab from the editor's Cursor Blame submenu (View commit, for the line under the cursor), from the palette (View Commit, which asks for a commit hash and starts from the clipboard), from a Source Control history item (Open in Cursor Blame) or from the file blame hover link; the tab shows the subject and message, an All Changes tab and a Chats tab, an Open in GitHub button, and a banner for very large commits.
- Commands: cursorBlame.openCommit; cursorBlame.viewCommit; workbench.scm.action.graph.openInCursorBlame
- Menus: EditorContextCursorBlame (View commit); SCMHistoryItemContext (Open in Cursor Blame, group 0_view, order 2); CommandPalette (View Commit, category Cursor Blame)
- Keys: none found
- Settings: cursor.blame.hoverDelay
- Cursor: cursor only (three ids, labels View Commit, View commit and Open in Cursor Blame, messages "No git repository found" and "This line has not been committed yet", and the strings All Changes, Chats, Open in GitHub and "Large commit (N files). Diffs load when expanded." found in C bundle and NLS)
- Source: C bundle (cursorBlame.openCommit and cursorBlame.viewCommit with NLS 5377 to 5383; openInCursorBlame action checking the cursor_blame gate; CursorBlameEditorInput with pane workbench.editor.cursorblame; renderCursorBlamePane markup); NLS 5377 to 5389

### ED-6.41 Offer to show whitespace differences when only whitespace changed
- Tag: both
- Evidence: source
- Boundary: no
- What: When the two sides differ only by leading or trailing spaces and those are being ignored, a floating button labelled Show Whitespace Differences appears in the editor; clicking it turns the ignore setting off for that file so the whitespace-only changes show up.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: diffEditor.ignoreTrimWhitespace
- Cursor: present (label Show Whitespace Differences found in NLS and the setting found in C bundle)
- Source: V\src\vs\workbench\contrib\codeEditor\browser\diffEditorHelper.ts (lines 36-52); C bundle; NLS (label)

### ED-6.42 Resize the two sides of a diff
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag the divider between the left and right side of a side-by-side diff to give one side more room, and double-click it to put it back in the middle; each side keeps at least 100 pixels.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found (the options enableSplitViewResizing and splitViewDefaultRatio are API options, not user settings)
- Cursor: present (both option names found in C bundle)
- Source: V\src\vs\editor\browser\widget\diffEditor\components\diffEditorSash.ts (lines 12-81); V\src\vs\editor\browser\widget\diffEditor\diffEditorOptions.ts; C bundle

### ED-6.43 Copy or revert a change from the inline diff margin
- Tag: both
- Evidence: source
- Boundary: no
- What: In the inline view, hovering a block of removed lines shows a lightbulb-style button in the margin; its menu copies the removed or changed lines (all of them, or one line), and offers Revert this change when the file is editable; selecting text inside the removed lines and copying also copies the original text.
- Commands: diff.clipboard.copyDeletedContent; diff.clipboard.copyDeletedLineContent; diff.inline.revertChange (these are action ids inside the popup menu, not registered commands)
- Menus: none found (the menu is built in code)
- Keys: none found
- Settings: diffEditor.renderSideBySide; diffEditor.experimental.useTrueInlineView (the button is drawn only in the regular inline rendering)
- Cursor: present (diff.clipboard.copyDeletedContent and diff.inline.revertChange found in C bundle; the labels Copy deleted lines, Copy changed line and Revert this change found in NLS)
- Source: V\src\vs\editor\browser\widget\diffEditor\components\diffEditorViewZones\inlineDiffDeletedCodeMargin.ts; V\src\vs\editor\browser\widget\diffEditor\components\diffEditorViewZones\copySelection.ts; V\src\vs\editor\browser\widget\diffEditor\components\diffEditorViewZones\diffEditorViewZones.ts; C bundle; NLS (labels)

### ED-6.44 Compare two new untitled text files
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Compare New Untitled Text Files to open a pinned diff editor with two empty untitled files, so you can paste text into each side and compare.
- Commands: workbench.files.action.compareNewUntitledTextFiles
- Menus: CommandPalette (category File)
- Keys: none found
- Settings: none found
- Cursor: present (id and label found in C bundle and NLS)
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (lines 792-818); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (line 37); C bundle; NLS (label)

### ED-6.45 Resolve a save conflict by comparing with the file on disk
- Tag: both
- Evidence: source
- Boundary: no
- What: When a save fails because the file on disk is newer and you pick Compare in the failed-save notification, a pinned diff opens (file on disk against your version, titled "... (in file) <-> ... (in the product) - Resolve save conflict") with two buttons in the editor title - Use your changes and overwrite file contents (check) and Discard your changes and revert to file contents (discard) - and a notification with Learn More explains them.
- Commands: workbench.files.action.resolveConflict; workbench.files.action.acceptLocalChanges; workbench.files.action.revertLocalChanges; workbench.files.action.resolveConflictLearnMore
- Menus: EditorTitle (navigation group, orders -10 and -9, only while the save conflict diff is active)
- Keys: none found
- Settings: files.saveConflictResolution
- Cursor: present (the three command ids and the context key saveConflictResolutionContext found in C bundle; both button labels and the editor label found in NLS)
- Source: V\src\vs\workbench\contrib\files\browser\editors\textFileSaveErrorHandler.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (appendSaveConflictEditorTitleAction); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle; NLS (labels)

### ED-6.46 Reveal folded diff stretches from their symbol breadcrumbs
- Tag: both
- Evidence: source
- Boundary: no
- What: A folded "N hidden lines" bar also lists the functions, classes or other symbols that contain the hidden lines, each with its icon and separated by chevrons; clicking one reveals the hidden lines from where that symbol starts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: diffEditor.hideUnchangedRegions.enabled
- Cursor: present (setBreadcrumbsSourceFactory and getBreadcrumbItems found in C bundle)
- Source: V\src\vs\editor\browser\widget\diffEditor\features\hideUnchangedRegionsFeature.ts; V\src\vs\editor\contrib\diffEditorBreadcrumbs\browser\contribution.ts; V\src\vs\editor\editor.all.ts; C bundle

### ED-6.47 Switch the base of the change peek
- Tag: both
- Evidence: source
- Boundary: no
- What: When more than one source can show a diff for the line you peeked from the gutter, the peek title gets a dropdown labelled Switch Quick Diff Base to choose which source to compare against, and the title reads "<source> - 1 of 3 changes"; with a single source it reads "1 of 3 changes".
- Commands: none found (the dropdown is an action inside the widget)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (labels Switch Quick Diff Base and the "{0} - {1} of {2} changes" strings found in NLS; the class name QuickDiffPickerViewItem was not found in the minified C bundle)
- Source: V\src\vs\workbench\contrib\scm\browser\quickDiffWidget.ts (QuickDiffPickerViewItem, updateDropdown, shouldUseDropdown); NLS (labels)

### ED-6.48 Move focus between the change peek and the editor
- Tag: vscode
- Evidence: source
- Boundary: no
- What: While a change is peeked from the gutter, press Ctrl+K F2 to move focus from the editor into the peeked diff, and press it again to go back.
- Commands: toggleQuickDiffWidgetFocus
- Menus: none found
- Keys: Ctrl+K F2 (Cmd+K F2 on macOS), when a quick diff peek is visible
- Settings: none found
- Cursor: missing (searched C bundle for the id and for the widget's toggleFocus method; closeQuickDiff is registered there without this sibling, and the only toggleFocus found is the menu bar's)
- Source: V\src\vs\workbench\contrib\scm\browser\quickDiffWidget.ts (lines 956-973); C bundle (negative search)

### ED-6.49 See how many conflicts remain in the merge Result
- Tag: both
- Evidence: source
- Boundary: no
- What: The Result pane's header shows "N Conflicts Remaining" (or "1 Conflict Remaining") as a button; clicking it jumps to the next unhandled conflict, and when none are left its tooltip says all conflicts are handled and the merge can be completed.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the labels {0} Conflict Remaining, {0} Conflicts Remaining, Go to next conflict and All conflicts handled found in NLS)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\resultCodeEditorView.ts; NLS (labels)

### ED-6.50 Show the Accept checkboxes in the merge editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Setting mergeEditor.showCheckboxes to true in settings.json draws the gutter checkboxes beside each change in the merge editor panes; the setting is not registered, so it is not in the Settings editor, and it defaults to off, which means the Accept boxes described in ED-6.19 are not drawn until it is on and the actions above each change are the only per-change controls.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: mergeEditor.showCheckboxes
- Cursor: present (the key is read with default false in C bundle)
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\codeEditorView.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\inputCodeEditorView.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\resultCodeEditorView.ts; V\src\vs\workbench\contrib\mergeEditor\browser\view\editors\baseCodeEditorView.ts; C bundle

### ED-6.51 Hide original line numbers in inline diffs
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A setting hides the line numbers of the original side in inline diffs; when it is null the editor decides, hiding them in compact multi diff editors and showing them in other diff editors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: diffEditor.hideOriginalLineNumbers
- Cursor: missing (searched C bundle for the setting id, none found)
- Source: N\src\vs\editor\common\config\editorConfigurationSchema.ts; N\src\vs\editor\common\config\diffEditor.ts; N\src\vs\workbench\contrib\multiDiffEditor\browser\multiDiffEditorInput.ts; C bundle (negative search)

### ED-6.52 Open the agent's changes for the current chat in Review Changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (it opens the agent's edits for the selected chat, and the key only works while the chat input has focus)
- What: Run Open Agent Changes from the Command Palette, or press Ctrl+Shift+R while the chat input has focus, to open the Review Changes tab for the selected chat's edits; a second command, Close Agent Changes, closes that tab again but has no palette entry or key.
- Commands: cursor.openAgentChangesEditor; cursor.closeAgentChangesEditor
- Menus: CommandPalette (Open Agent Changes, f1, diff icon)
- Keys: Ctrl+Shift+R (Cmd+Shift+R on macOS), when the context key composerFocused is set
- Settings: none found
- Cursor: cursor only (both ids, the title Open Agent Changes, the key and the context key composerFocused read from the action registration in C bundle)
- Source: C bundle (actions cursor.openAgentChangesEditor, which calls openOrUpdateReviewChangesEditor for the selected chat and tracks review_changes.opened with entrypoint action, and cursor.closeAgentChangesEditor, which skips Review Changes tabs opened in git mode or from Bugbot)

### ED-6.53 Keep or undo one file's changes in Review Changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (acts on the agent's pending edits in one file)
- What: A file header in Review Changes can show a check button (Keep changes in this file) and an x button (Undo changes in this file), so one file's pending agent edits can be accepted or thrown away without touching the others; whether they show for a given file depends on flags the pane passes in, which were not traced.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (both hint texts and the showKeep and showUndo flags found in the Review Changes file item code in C bundle)
- Source: C bundle (module ReviewChangesFileItem, buttons with hints "Keep changes in this file" and "Undo changes in this file")

### ED-6.54 Show the whole file or only the diff in Review Changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the editor is the agent review surface)
- What: Each file header in Review Changes has an expander button whose hint reads Show entire file (to display the whole file around the changes) or Show diff only (to go back to just the changed stretches).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (both hint texts found in the Review Changes file item code in C bundle)
- Source: C bundle (module ReviewChangesFileItem, expander button with hints "Show entire file" and "Show diff only")

### ED-6.55 See a summary header above the changes
- Tag: cursor
- Evidence: bundle
- Boundary: yes (the header is shared by Review Changes and the Cursor Blame commit view, and its author avatar can be the agent)
- What: A header above the diffs shows a title, a Changes count (files and lines added and removed) and, when the data is there, an Author with avatar (the agent has its own avatar), Issues, Deployments links and a pull request status chip (Closed, Draft or Merged were seen), plus a summary text with a More or Less button; this comes from class names and markup strings, not from a screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (module names ReviewChangesSummaryHeader and ReviewChangesMarkdownDescription, the smart-review-panel header classes and the More and Less button found in C bundle)
- Source: C bundle (modules ReviewChangesSummaryHeader and ReviewChangesMarkdownDescription; classes smart-review-panel__header, __changes-section, __author-section, __deployments-section, __summary-more-button)

### ED-6.56 Group a commit's changed files by agent conversation
- Tag: cursor
- Evidence: bundle
- Boundary: yes (behind the cursor_blame feature gate and a team setting, and it groups by agent conversation)
- What: In a commit opened in the Cursor Blame editor, a Group By choice (under a Manage View button, inferred from strings) switches the changed-files list between a plain files list and groups named by agent conversation, with files no agent touched in a separate Non-agent files section.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (defaultGroupingMode files, the section titles Agent conversations and Non-agent files, and the Group By and Manage View labels found in C bundle)
- Source: C bundle (module CursorBlamePane with showGroupBySelector, groupedSectionTitle and secondarySectionTitle; the shared file list with grouping mode conversation)

### ED-6.57 See how much of a commit the agent wrote
- Tag: cursor
- Evidence: bundle
- Boundary: yes (behind the cursor_blame feature gate and a team setting, and it shows agent contribution)
- What: The commit header in Cursor Blame has a Contributions section with a segmented bar sized by percent; hovering a segment opens a tooltip and dims the other segments (from class names, not seen on screen).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only (classes smart-review-panel__agent-contribution-section, __agent-bar, __agent-segment and contribution-tooltip-container found in C bundle)
- Source: C bundle (module CursorBlamePane, contributions section with agent-segment width set from percent and a dimmed class)

### ED-6.58 See which added lines came from the agent or Tab
- Tag: cursor
- Evidence: bundle
- Boundary: yes (behind the cursor_blame feature gate and a team setting, and it marks agent and Tab authorship)
- What: In a commit opened in Cursor Blame, added lines get a margin mark and a line tint, in one of six colors per agent conversation or a separate mark for lines written by Tab, and removed lines get matching marks; hovering a margin mark (after the hover delay) opens a hover that Escape closes, with content that was not read (from class names and code, not seen on screen).
- Commands: none found
- Menus: none found
- Keys: Escape (closes the margin hover)
- Settings: cursor.blame.hoverDelay
- Cursor: cursor only (classes cursor-blame-glyph-agent, cursor-blame-glyph-tab, cursor-blame-line-agent, cursor-blame-line-tab, the deleted variants, cursor-blame-glyph-agent-conv-N and cursor-blame-gutter-hover found in C bundle)
- Source: C bundle (module CursorBlamePane, decoration builder over rangeAnnotations with conversationId, glyph class chosen by a hash of the conversation id modulo six, showInstantHover with class cursor-blame-gutter-hover)

## Handed off

- Compare, restore and rename of Local History entries (workbench.action.localHistory.*): SB-7, already listed there as SB-7.13 to SB-7.15, and they open the diff editor described in ED-6.1.
- Compare two Git history entries and Timeline select for compare (git.timeline.selectForCompare, git.timeline.compareWithSelected): SB-7, already listed as SB-7.18.
- Git Graph compare commands (git.graph.compareRef, git.graph.compareWithRemote, git.graph.compareWithMergeBase) and Compare with Workspace (git.compareWithWorkspace): SB-4, already covered by SB-4.4 and the graph features.
- Explorer right-click entry Cursor Blame for a file: SB-2, already listed as SB-2.39.
- Agent Review view and Start Agent Review (workbench.scm.action.triggerAgentReview): SB-4, already listed as SB-4.39.
- Review Branch button in the Source Control title (workbench.scm.action.openReviewChangesPane): SB-4.38 lists the button; ED-6.35 lists the editor it opens.
- Git blame status bar item (git.blame.statusBarItem.enabled, git.blame.statusBarItem.template, git.blame.toggleStatusBarItem): SS-4.
- The "Inline diffs have been suppressed" notification with its Disable Protection button: NT-2 (it is a notification; ED-6.28 lists the setting it switches).
- Notebook diff editor (panes Notebook Diff Editor and Notebook Multi Diff Editor, notebook.diff.* commands): ED-7.
- Accessibility help pages for the diff editor and the merge editor (DiffEditorAccessibilityHelp, MergeEditorAccessibilityHelp) and the settings accessibility.verbosity.diffEditor, accessibility.verbosity.diffEditorActive and accessibility.verbosity.mergeEditor: WB (accessibility).
- Sounds for diff lines (accessibility.signals.diffLineInserted, diffLineDeleted, diffLineModified, played in the Accessible Diff Viewer): WB (accessibility signals).
- Merge editor developer commands (merge.dev.copyContentsJson, merge.dev.saveContentsToFolder, merge.dev.loadContentsFromFolder, merge.dev.openContentsJson, merge.dev.openSelectionInTemporaryMergeEditor, labelled Copy Merge Editor State as JSON, Save Merge Editor State to Folder, Load Merge Editor State from Folder, Open Merge Editor State from JSON and Open Selection In Temporary Merge Editor): WB (developer-category commands).
- The notification "The diff algorithm was stopped early (after {0} ms.)" with its Remove Limit button, and the failed-save notification whose Compare button opens the diff in ED-6.45: NT-2.
- The prompt "Disable Inline Diffs: Enable auto-keep to have changes be updated automatically" with its Enable auto-keep button (it flips the stored noInlineDiffs value that Toggle Inline Diffs in ED-6.28 also flips): NT-2.
- timeline.compareSelected (newer-than-1.128, found only in N): SB-7.
- Review Changes pull request mode (the editor can list a GitHub pull request's files and comments through the GitHub API) and the pull request button with View PR, Mark ready for review, Squash merge, Rebase merge and Merge commit: out of scope, they sit in the background agent and Bugbot flow, so they are not listed.

## Unclear

- Cap: the 40 limit left out Compare New Untitled Text Files (workbench.files.action.compareNewUntitledTextFiles, palette only, no key) and the diff editor's Reopen as Text title button for custom editors (ReOpenInTextEditorAction).
- ED-6.11 repeats the compare actions that SB-2.17 lists from the Explorer side. I kept it here because the brief for this group asked for Compare Selected; the entry points are the same, so one of the two can be dropped.
- Chat hand-offs inside this group were left out as asked: merge-conflict.resolve-in-chat, git.resolveAllConflictsInChat, and the CodeLens Resolve with AI. The inline edit prompt bar ids editor.action.inlineDiffs.focusEditor, submitFix and cancelPromptBar belong to the Cmd+K prompt bar (inline chat) and are also left out.
- editor.action.inlineDiffs.cancelEdits uses the context key hasActivelyGeneratingDiff and may also cover inline edit (Cmd+K) diffs, not only agent edits. I could not tell from the strings.
- The key for editor.action.inlineDiffs.rejectAllAcrossAllEditors is written as Ctrl plus a key code plus Backspace in the bundle, which looks like a typo for Ctrl+Shift+Backspace. Whether it ever fires was not determined.
- Review Changes: the Keep All and Undo All buttons were found in the pane code and in nearby generic button code; I could not separate the pane's own buttons from a shared pending-changes bar. The tab can be opened from the composer's Review Changes button and from Bugbot, but those entry points are chat surfaces and have no command id I could find.
- `cursor.tinderdiffeditor`: a URI scheme named next to cursor.reviewchanges in the bundle. No pane, label or command with that name was found, so nothing was listed.
- Quick diff: the per-source visibility toggles (Diff Decorations submenu on the line-number menu) look missing in Cursor, but I only searched for the label and MenuId strings. Whether Cursor has another way to hide a source was not checked.
- The Cursor Blame commit view and File Blame look and behavior come from markup strings and CSS class names only. The feature gate cursor_blame has default false, so the tabs may not appear for most users.
- ED-6.2: stock 1.128.0 lists diffEditor.renderSideBySideInlineBreakpoint as a user setting, but the bundle has only the internal option, so Cursor may hide or drop that setting. Not confirmed on screen.
- Overflow check on 2026-10-25 added 18 features (ED-6.41 to ED-6.58).
