Group: ED-6 Diffs, merges and change review. Pass: 3. Date: 2026-10-25.

### ED-6.1
1. Opening a diff, for example through `vscode.diff`, `vscode.changes` or `_workbench.diff`, shows the two versions of a file next to each other in the Text Diff Editor, with changed lines tinted, changed words tinted more strongly, and the right side editable.
2. Markers in the gutter and a colored overview ruler along the edge of the editor show where in the file the changes are.
3. A diff of files that are not text opens in the Binary Diff Editor, or in the Side by Side Editor when the two sides need different editors.
4. Setting `diffEditor.renderIndicators` chooses whether plus and minus marks show beside added and removed lines, with values true (default) and false.
5. Setting `diffEditor.codeLens` chooses whether CodeLens shows inside the diff editor, with values true and false (default).
6. Setting `diffEditor.wordWrap` chooses whether lines wrap in the diff editor, with values `off`, `on` and `inherit` (follow `editor.wordWrap`, the default).
7. Setting `diffEditor.diffAlgorithm` chooses how differences are worked out, with values `legacy`, `advanced` (default), `advanced-external` and `advanced-wasm`.
8. Settings `diffEditor.maxComputationTime` (milliseconds, default 5000, 0 for no timeout) and `diffEditor.maxFileSize` (megabytes, default 50, 0 for no limit) decide when the diff stops being computed for a slow or large file.
9. Settings `diffEditor.experimental.useTrueInlineView` (default off) draws word changes inline in the inline view, and `diffEditor.experimental.showEmptyDecorations` (default on) shows marks where characters were inserted or deleted.

### ED-6.2
1. Clicking Inline View in the editor title (`toggle.diff.renderSideBySide`) turns a two-column diff into one column with removed lines above the added ones, and clicking it again returns to two columns.
2. Setting `diffEditor.renderSideBySide` chooses the layout a diff opens in, with values true (side by side, default) and false (inline), and the title toggle changes this same setting.
3. When the diff editor is narrower than the breakpoint (900 pixels by default) and `diffEditor.useInlineViewWhenSpaceIsLimited` is on, the diff shows inline by itself without the person clicking anything.
4. A second toggle, Use Inline View When Space Is Limited (`diffEditor.toggleUseInlineViewWhenSpaceIsLimited`), appears in the editor title only once the editor has reached the narrow breakpoint, and turns that automatic drop to inline on or off.
5. Setting `diffEditor.useInlineViewWhenSpaceIsLimited` chooses whether a narrow diff editor drops to the inline view on its own, with values true (default) and false.
6. Both toggles can also be run from the Command Palette.
7. In Cursor the setting `diffEditor.renderSideBySideInlineBreakpoint` is not registered as a user setting and the 900 pixel breakpoint stays an internal value. [stock only]

### ED-6.3
1. Clicking the whitespace button in the diff editor title, labelled Show Leading/Trailing Whitespace Differences (`toggle.diff.ignoreTrimWhitespace`), flips whether lines that differ only by leading or trailing spaces count as changes.
2. The button appears pressed while whitespace differences are being shown, and unpressed while they are ignored.
3. The button sits in the navigation group of the editor title and shows only while a text compare editor is active.
4. Setting `diffEditor.ignoreTrimWhitespace` chooses whether leading and trailing whitespace changes are ignored in diffs, with values true (default) and false, and the title button changes this same setting.

### ED-6.4
1. Turning on the collapse behavior folds each long stretch of identical lines in a diff into a small bar that reads as hidden lines, and the changed regions stay open with a few lines of context.
2. Clicking a folded bar reveals more of that one stretch of hidden lines.
3. The map icon button in the editor title (`diffEditor.toggleCollapseUnchangedRegions`) turns the whole collapse behavior on or off.
4. The Command Palette entries Collapse All Unchanged Regions (`diffEditor.collapseAllUnchangedRegions`) and Show All Unchanged Regions (`diffEditor.showAllUnchangedRegions`), in category Diff Editor, fold every unchanged stretch or reveal them all at once.
5. Setting `diffEditor.hideUnchangedRegions.enabled` chooses whether unchanged stretches are folded, with values true and false (default).
6. Setting `diffEditor.hideUnchangedRegions.minimumLineCount` (default 3) chooses how many unchanged lines a stretch needs before it is folded.
7. Setting `diffEditor.hideUnchangedRegions.contextLineCount` (default 3) chooses how many unchanged lines stay visible around each change.
8. Setting `diffEditor.hideUnchangedRegions.revealLineCount` (default 20) chooses how many hidden lines one click on a folded bar reveals.

### ED-6.5
1. Turning on Show Moved Code Blocks (`diffEditor.toggleShowMovedCodeBlocks`) draws a link between a block of lines that was cut from one place and pasted in another.
2. Clicking a link between moved blocks switches the diff into a comparison of the two places, showing the block in its old and new position.
3. Pressing Escape while comparing moved code (`diffEditor.exitCompareMove`) leaves that comparison and returns to the normal diff.
4. The Show Moved Code Blocks button sits in the editor title in group 1_diff with a move icon.
5. Setting `diffEditor.experimental.showMoves` chooses whether detected code moves are shown, with values true and false (default).

### ED-6.6
1. The up and down arrows in the editor title, Previous Change and Next Change, move the cursor to the previous or next change in the diff, and they show only when the diff has changes.
2. Pressing Alt+F5 goes to the next change and Shift+Alt+F5 goes to the previous change while a text compare editor is visible (`workbench.action.compareEditor.nextChange`, `workbench.action.compareEditor.previousChange`).
3. Go to Next Change and Go to Previous Change are also in the Command Palette.
4. Open Accessible Diff Viewer, in group 2_diff of the editor title and in the Command Palette, shows the changes one at a time in a list that can be read with a screen reader.
5. Pressing F7 in a diff editor moves to the next difference and Shift+F7 to the previous one (`editor.action.accessibleDiffViewer.next` and `editor.action.accessibleDiffViewer.prev`, also reachable under the older ids `editor.action.diffReview.next` and `editor.action.diffReview.prev`).

### ED-6.7
1. In a diff whose right side is editable, a gutter button between the two sides appears beside a changed block, and clicking it puts that block back to the left side's text (`diffEditor.revert`, Revert Block).
2. Selecting lines on the right side makes the gutter button offer Revert Selection, which puts only the selected lines back to the left side's text.
3. The revert buttons do not appear when the right side is read-only.
4. Setting `diffEditor.renderMarginRevertIcon` chooses whether the diff shows revert arrows in its glyph margin, with values true (default) and false.
5. Setting `diffEditor.renderGutterMenu` chooses whether the diff draws a special gutter for revert and stage actions, with values true (default) and false.
6. Revert Block and Revert Selection are also in the Command Palette under category Diff Editor.

### ED-6.8
1. In a Git diff, right-clicking selected lines and choosing Stage Selected Ranges (`git.stageSelectedRanges`) stages only those lines.
2. Right-clicking selected lines on the right side of a diff of an index file and choosing Unstage Selected Ranges (`git.unstageSelectedRanges`) unstages only those lines.
3. Revert Selected Ranges (`git.revertSelectedRanges`) puts the selected lines in the working file back to their committed text.
4. A gutter button beside one changed block stages that block (`git.diff.stageHunk`), and a gutter button beside a selection stages the selection (`git.diff.stageSelection`).
5. The same stage and unstage entries are available from the editor title overflow menu in a Git diff.
6. Ctrl+K Ctrl+Alt+S stages the selected ranges and Ctrl+K Ctrl+R reverts them while the text of a file has focus, and Ctrl+K Ctrl+N unstages them on the right side of a diff of an index file.

### ED-6.9
1. Running Switch Side (`diffEditor.switchSide`) moves the cursor to the matching spot on the other side of the diff.
2. Focus commands jump keyboard focus to the other side, the left side or the right side (`workbench.action.compareEditor.focusOtherSide`, `workbench.action.compareEditor.focusPrimarySide`, `workbench.action.compareEditor.focusSecondarySide`).
3. The Swap Left and Right Side button in the editor title (`workbench.action.compareEditor.swapSides`) trades the two sides, and it shows only when the compare editor can swap.
4. Pressing Ctrl+K Ctrl+Shift+O in a diff editor opens the active side as its own editor (`workbench.action.compareEditor.openSide`), and Open Active Diff Side does the same from the Command Palette.
5. Swap Left and Right Editor Side and Switch Side are also in the Command Palette, with Switch Side under category Diff Editor.

### ED-6.10
1. Running Compare Active File with Saved (`workbench.files.action.compareWithSaved`) opens a diff of the unsaved edits in the active file against the version on disk.
2. Pressing Ctrl+K D runs Compare Active File with Saved.
3. Compare with Saved in the Open Editors list context menu appears in group 3_compare only for a dirty editor while auto save is off.
4. The command is in the Command Palette under category File.

### ED-6.11
1. Compare Active File With... (`workbench.files.action.compareFileWith`) shows a list of files to pick from, and the picked file opens in a diff against the active file.
2. Select for Compare (`selectForCompare`) marks one file, and Compare with Selected (`compareFiles`) on another file opens a diff of the two.
3. Selecting exactly two files and choosing Compare Selected (`compareSelected`) opens a diff of the two.
4. The compare entries appear in the Explorer context menu, in group 3_compare of the Open Editors context menu, and in group 1_compare of the editor tab context menu.
5. The compare commands are also in the Command Palette.

### ED-6.12
1. Running Compare Active File with Clipboard (`workbench.files.action.compareWithClipboard`) opens a diff between the active file and whatever text is on the clipboard.
2. Pressing Ctrl+K C runs Compare Active File with Clipboard.
3. The command is in the Command Palette under category File.

### ED-6.13
1. Opening the multi-file diff editor shows the changes of many files in one scrolling editor, each file as a collapsible diff under its own header.
2. Source Control's View Changes, a stash and a commit each open their changed files in the multi-file diff editor (`_workbench.openScmMultiDiffEditor`, and `_workbench.openMultiDiffEditor` for other callers).
3. Clicking a file header folds or unfolds that file's diff.
4. The editor title button Collapse All Diffs (`multiDiffEditor.collapseAll`) folds every file's diff down to its header, and it shows while at least one diff is open.
5. Once every diff is folded, the editor title button changes to Expand All Diffs (`multiDiffEditor.expandAll`), which opens them all again.
6. The collapse and expand buttons also show in the compact window editor title and are in the Command Palette.
7. In Cursor the setting `multiDiffEditor.experimental.enabled` is registered with default true, which stock 1.128.0 does not have. [cursor only]

### ED-6.14
1. A file header in the multi-file diff has an Open File button (`multiDiffEditor.goToFile`) that opens the real file.
2. Up and down arrows in the editor title go to the previous or next change across all the files in the multi-file diff (`multiDiffEditor.goToPreviousChange`, `multiDiffEditor.goToNextChange`). [stock only]
3. Pressing Alt+F5 goes to the next change and Shift+Alt+F5 to the previous change while the multi-file diff is the active editor. [stock only]

### ED-6.15
1. The header of a file that is a working tree or untracked change in a Git multi-file diff has a button that stages that file (`git.stageFile`).
2. The header of a file that is already staged has a button that unstages it (`git.unstageFile`).
3. These header buttons show only for diffs that come from Git, not for multi-file diffs from other sources.

### ED-6.16
1. A toggle in the editor title shows or hides a file list beside the multi-file diff (`multiDiffEditor.toggleFileList`). [stock only]
2. A command moves keyboard focus into the file list (`multiDiffEditor.focusFileList`). [stock only]
3. The file list can be shown as a flat list (`multiDiffEditor.viewFileListAsList`) or as a tree of folders (`multiDiffEditor.viewFileListAsTree`). [stock only]
4. The file list can sit on the left (`multiDiffEditor.positionFileListLeft`) or on the right (`multiDiffEditor.positionFileListRight`) of the diffs. [stock only]
5. Setting `multiDiffEditor.experimental.fileList` turns the file list feature on or off. [stock only]
6. The view and position choices are in a submenu of the editor title with groups 1_view and 2_position. [stock only]

### ED-6.17
1. A file with Git conflicts shows a Resolve in Merge Editor button at the top of the editor (`git.openMergeEditor`) that opens the three-way merge editor for that file.
2. The button shows only when the file is in the Git merge changes and has conflicts, and not inside a diff editor or a merge editor.
3. Setting `git.mergeEditor` chooses whether conflicted files open straight in the merge editor, with values true and false (default).
4. With `git.mergeEditor` off, a conflicted file opens as plain text with the Resolve in Merge Editor button above it, and with it on the file opens in the merge editor directly (`_open.mergeEditor`).

### ED-6.18
1. The merge editor in its default Mixed Layout (`merge.mixedLayout`) shows Input 1 and Input 2 next to each other above a Result pane.
2. Column Layout (`merge.columnLayout`) shows Input 1, Input 2 and Result as three columns.
3. Show Base (`merge.showBase`) adds a pane with the common ancestor, and Show Base Top and Show Base Center (`merge.showBaseTop`, `merge.showBaseCenter`) choose whether it sits on top or in the center.
4. Show Non-Conflicting Changes (`merge.showNonConflictingChanges`) marks the changes that do not clash.
5. Each input title has a Compare With Base button (`mergeEditor.compareInput1WithBase`, `mergeEditor.compareInput2WithBase`) that opens a diff of that input against the base.
6. Open File (`merge.openResult`) and Open Base File (`merge.openBaseEditor`) open the plain result file and the plain base file.
7. The layout and view choices sit in the editor title in groups navigation, 1_merge, 2_merge and 3_merge, and the input and base titles have their own toolbars.
8. Setting `mergeEditor.diffAlgorithm` chooses how the inputs are compared, with values `legacy`, `advanced` (default), `advanced-external` and `advanced-wasm`.
9. Setting `mergeEditor.showDeletionMarkers` chooses whether a deletion in the base or in one input is shown as a vertical bar, with values true (default) and false.

### ED-6.19
1. Ticking the Accept box beside a change in Input 1 or Input 2 copies that change into Result.
2. Ticking the Accept boxes of both inputs for the same conflict puts a combination of the two into Result.
3. Above a conflict in Result a row of actions appears, offering Accept, Accept Combination, Append, Ignore, Remove and Reset to base, each worded with the input title where it applies.
4. The label above a conflict in Result reads Manual Resolution when the conflict was edited by hand and No Changes Accepted when nothing was taken.
5. Toggle Current Conflict from Left and Toggle Current Conflict from Right (`merge.toggleActiveConflictInput1`, `merge.toggleActiveConflictInput2`) accept or unaccept the current conflict from that side by command.

### ED-6.20
1. The title of Input 1 has an Accept All Incoming Changes from Left button (`merge.acceptAllInput1`) and the title of Input 2 has an Accept All Current Changes from Right button (`merge.acceptAllInput2`), each copying all of that input's changes into Result.
2. The Result title has a Reset Result button (`mergeEditor.resetResultToBaseAndAutoMerge`) that returns Result to the base text plus automatic merging.
3. Compute Conflicts With Git (`git.runGitMerge`) and Compute Conflicts With Git (Diff3) (`git.runGitMergeDiff3`) in the Command Palette rebuild Result with Git's own merge.
4. Accept All Combination (`mergeEditor.acceptAllCombination`) takes the automatic combination for every conflict at once. [stock only]

### ED-6.21
1. The editor title arrows Go to Next Unhandled Conflict and Go to Previous Unhandled Conflict (`merge.goToNextUnhandledConflict`, `merge.goToPreviousUnhandledConflict`) move to the next or previous conflict that has not been dealt with.
2. Both conflict commands are in the Command Palette under category Merge Editor.
3. Pressing Ctrl+Shift+T in the merge editor flips focus between Input 1 and Input 2 (`mergeEditor.toggleBetweenInputs`). [stock only]

### ED-6.22
1. Pressing Ctrl+Enter or clicking Complete Merge (`mergeEditor.acceptMerge`) saves Result as the merged file and closes the merge.
2. If conflicts are still unhandled, a dialog asks "Do you want to complete the merge of" the file, says it contains unhandled conflicts, and offers a Complete with Conflicts button.
3. The Command Palette entry Reset Choice for 'Close with Conflicts' (`mergeEditor.resetCloseWithConflictsChoice`), in category Merge Editor, forgets the remembered Close with Conflicts choice.
4. When the Result editor belongs to a Git merge, a Complete Merge button (`git.acceptMerge`) shows at the top of the editor.

### ED-6.23
1. A file with conflict markers gets the Current and Incoming blocks tinted in different colors.
2. A row of CodeLens above each conflict reads Accept Current Change, Accept Incoming Change, Accept Both Changes and Compare Changes.
3. Compare Changes (`merge-conflict.compare`) opens a diff of the two sides, placed in the current group, beside it or below it.
4. Previous Conflict and Next Conflict arrows (`merge-conflict.previous`, `merge-conflict.next`) sit in the editor title only while the file has conflicts.
5. Command Palette commands accept the conflict at the cursor as Current, Incoming or Both, or the selection (`merge-conflict.accept.current`, `merge-conflict.accept.incoming`, `merge-conflict.accept.both`, `merge-conflict.accept.selection`).
6. Accept All Current, Accept All Incoming and Accept All Both (`merge-conflict.accept.all-current`, `merge-conflict.accept.all-incoming`, `merge-conflict.accept.all-both`) settle every conflict in the file, and the first two are also on the Merge Changes group in Source Control.
7. Setting `merge-conflict.codeLens.enabled` chooses whether the CodeLens row shows, and `merge-conflict.decorators.enabled` chooses whether the tinted blocks show, each true (default) or false.
8. Setting `merge-conflict.autoNavigateNextConflict.enabled` chooses whether the cursor jumps to the next conflict after one is resolved, true or false (default).
9. Setting `merge-conflict.diffViewPosition` chooses where Compare Changes opens, with values `Current`, `Beside` and `Below`.
10. In Cursor a fifth CodeLens, Resolve with AI, also shows in the row above each conflict, as a hand-off to chat that this catalog leaves out. [cursor only]

### ED-6.24
1. Colored bars in the gutter mark lines added, modified or deleted since the Git base.
2. Matching marks can also show in the overview ruler and in the minimap.
3. Setting `scm.diffDecorations` chooses where the marks show, with values `all` (default), `gutter`, `overview`, `minimap` and `none`.
4. Setting `scm.diffDecorationsGutterWidth` chooses how wide the gutter bars are in pixels, with values 1 to 5 and default 3.
5. Setting `scm.diffDecorationsGutterVisibility` chooses whether the gutter bars always show or only on hover, with values `always` (default) and `hover`.
6. Setting `scm.diffDecorationsGutterPattern` chooses whether added lines and modified lines draw their gutter bar with a pattern, with `added` false and `modified` true by default.
7. Setting `scm.diffDecorationsIgnoreTrimWhitespace` chooses whether the marks ignore leading and trailing whitespace, with values `true`, `false` (default) and `inherit` (follow `diffEditor.ignoreTrimWhitespace`).
8. Right-clicking the line numbers shows a Diff Decorations submenu with one toggle per change source (`workbench.scm.action.toggleQuickDiffVisibility.<provider id>`) to hide that source's marks. [stock only]

### ED-6.25
1. Clicking a gutter bar opens a small inline diff under that line showing the original text of the change.
2. Stage Change and Revert Change buttons in the peek title (`git.stageChange`, `git.revertChange`) stage or revert the peeked change.
3. Pressing Alt+F3 moves to the next change inside the peek and Shift+Alt+F3 to the previous one (`editor.action.dirtydiff.next`, `editor.action.dirtydiff.previous`).
4. Pressing Alt+F5 and Shift+Alt+F5 with text focus moves to the next or previous change in the file without needing a diff open (`workbench.action.editor.nextChange`, `workbench.action.editor.previousChange`).
5. Next Change and Previous Change in the Go menu, group 7_change_nav, do the same as those keys.
6. Pressing Escape closes the peek (`closeQuickDiff`).
7. Setting `scm.diffDecorationsGutterAction` chooses what clicking a gutter bar does, with values `diff` (show the peek, default) and `none` (do nothing).

### ED-6.26
1. The editor title of a changed file that is open normally shows an Open Changes button (`git.openChange`) that opens its diff.
2. In a diff, or in a file shown from a Git revision, the editor title shows an Open File button (`git.openFile`) that opens the working file.
3. The editor title overflow menu holds Stage Changes (`git.stage`) and Unstage Changes (`git.unstage`) for the open file.
4. A stash shown in an editor gets an Apply Stash button, which pops the stash instead when Alt is held (`git.stashApplyEditor`, `git.stashPopEditor`).
5. A stash shown in an editor also gets a Drop Stash button (`git.stashDropEditor`).

### ED-6.27
1. With Git Blame Editor Decoration on, the line the cursor is on shows faint text at its end with the last commit's subject, author and age.
2. Hovering that text opens a hover with the commit details and an Open Commit link (`git.viewCommit`).
3. Toggle Git Blame Editor Decoration (`git.blame.toggleEditorDecoration`) in the Command Palette turns the decoration on or off.
4. Setting `git.blame.editorDecoration.enabled` chooses whether the decoration shows, with values true and false (default).
5. Setting `git.blame.editorDecoration.template` chooses the text shown, with default `${subject}, ${authorName} (${authorDateAgo})`.
6. Setting `git.blame.ignoreWhitespace` chooses whether blame skips whitespace-only changes, with values true and false (default).
7. Setting `git.blame.editorDecoration.disableHover` chooses whether the hover is turned off, with values true and false (default). [stock only]
8. The application setting `git.blame.editorDecoration.forceDisabled` forcibly turns the decoration off and overrides workspace settings, with values true and false (default). [cursor only]

### ED-6.28
1. When the agent edits a file that is open in the text editor, the changes draw inside the editor itself, with deleted code shown in blocks between lines and added lines highlighted. [cursor only]
2. Toggle Inline Diffs (`workbench.action.toggleInlineDiffs`) in the Command Palette, category View, hides the inline diffs or shows them again. [cursor only]
3. When too many changes would draw at once, a notification says the inline diffs were suppressed and offers a Disable Protection button. [cursor only]
4. Setting `cursor.inlineDiff.enablePerformanceProtection` chooses whether that suppression is on, with values true and false. [cursor only]

### ED-6.29
1. Pressing Ctrl+Enter with text focus in a file that has inline diffs keeps every pending inline edit in that file at once. [cursor only]
2. Accept Edits (`editor.action.inlineDiffs.acceptAll`) in the Command Palette does the same as the key. [cursor only]
3. The key and the command work only while the active editor has inline diffs. [cursor only]

### ED-6.30
1. Pressing Ctrl+Shift+Backspace with text focus in a file that has inline diffs throws away every pending inline edit in that file (`editor.action.inlineDiffs.rejectAll`). [cursor only]
2. Undo Edits is the label of that action in the Command Palette. [cursor only]
3. Reject All Edits (`editor.action.inlineDiffs.rejectAllAcrossAllEditors`) in the Command Palette throws away the pending inline edits in every open editor, not only the active one. [cursor only]

### ED-6.31
1. A small accept and reject widget follows the change under the mouse or cursor in a file with inline diffs. [cursor only]
2. Accept Partial Edit (`editor.action.inlineDiffs.acceptPartialEdit`) keeps just that one change, with Ctrl+Shift+Y on Windows, Cmd+Y on macOS and Ctrl+Y on Linux. [cursor only]
3. Reject Partial Edit (`editor.action.inlineDiffs.rejectPartialEdit`) drops just that one change, with Ctrl+N or Cmd+N. [cursor only]
4. Both keys need text focus and inline diffs in the active editor, and both commands are also in the Command Palette. [cursor only]

### ED-6.32
1. While a diff is still being written into the file, pressing Ctrl+Shift+Backspace runs Cancel Edits (`editor.action.inlineDiffs.cancelEdits`) and stops it. [cursor only]
2. Once nothing is being written any more, the same key runs Undo Edits instead, because Cancel Edits only takes priority while a diff is actively generating. [cursor only]
3. Cancel Edits is also in the Command Palette. [cursor only]

### ED-6.33
1. Pressing Alt+J moves to the next pending change in the file and Alt+K to the previous one (`editor.action.inlineDiffs.nextChange`, `editor.action.inlineDiffs.previousChange`). [cursor only]
2. Pressing Alt+L moves to the next file that has pending changes and Alt+H to the previous one (`editor.action.inlineDiffs.nextDiffFile`, `editor.action.inlineDiffs.previousDiffFile`). [cursor only]
3. Go to Next Change, Go to Previous Change, Go to Next Diff File and Go to Previous Diff File are also in the Command Palette. [cursor only]
4. The change keys need text focus and inline diffs in the active editor, and the file keys need inline diffs in some open editor. [cursor only]

### ED-6.34
1. Running View All Changes (`editor.action.inlineDiffs.viewAllChanges`) opens a tab named Multi File Edits that stacks the inline diffs of every file with pending agent edits. [cursor only]
2. View All Changes is in the Command Palette as an editor action. [cursor only]
3. The Multi File Edits tab is not restored when the window reloads. [cursor only]

### ED-6.35
1. A pinned Review Changes tab shows every changed file as a collapsible diff, with a file list. [cursor only]
2. In agent mode the tab lists all of the changes from the agent chat. [cursor only]
3. In git mode the tab shows the diff of the branch against the default branch, labelled Diff with Default Branch or Diff with the branch name. [cursor only]
4. The tab can also show unstaged Git changes. [cursor only]
5. The Review Branch button in the Source Control title (`workbench.scm.action.openReviewChangesPane`) opens the tab, and the command is hidden from the Command Palette. [cursor only]

### ED-6.36
1. The Review Changes header has a primary Keep All button that accepts all of the agent's pending edits in one step and saves the files. [cursor only]
2. When only one change is left the Keep All button reads Keep. [cursor only]
3. The header also has an Undo All button, and clicking it once turns it into Confirm with the hint "Click again to confirm undo", and clicking again throws away the pending edits. [cursor only]
4. When only one change is left the Undo All button reads Undo. [cursor only]

### ED-6.37
1. A primary button in the Review Changes header commits the changes, with the choices Commit, Commit & Push, Push and Create PR. [cursor only]
2. Choosing Create PR opens a Create Pull Request dialog. [cursor only]
3. The button is disabled with the message "Commit is unavailable until there are reviewable changes" while there is nothing to review. [cursor only]
4. The button is disabled with the message "Install Git to enable committing from Review Changes" when Git is not installed. [cursor only]

### ED-6.38
1. Pressing Ctrl+F while Review Changes is the active editor opens a find box that searches text across all its diffs (`actions.find`). [cursor only]
2. Pressing F3 goes to the next match and Shift+F3 to the previous one (Cmd+G and Cmd+Shift+G on macOS) (`editor.action.nextMatchFindAction`, `editor.action.previousMatchFindAction`). [cursor only]
3. The ellipsis menu has Unified View and Split View to choose how the diffs are laid out. [cursor only]
4. The ellipsis menu has Expand All and Collapse All to open or fold every file's diff. [cursor only]
5. The find keys are enabled only while Review Changes is the active editor. [cursor only]

### ED-6.39
1. Right-clicking in the editor of a file on disk and choosing Cursor Blame, then Toggle file blame (`cursorBlame.openFileBlame`), opens a pinned view of the file with one row per line showing revision hash, author, message and time. [cursor only]
2. Open File Blame is also in the Command Palette under category Cursor Blame. [cursor only]
3. Hovering a row of the file blame view shows Author, Message and When, with an Open in Cursor Blame link. [cursor only]
4. The file blame header has an Open original file button that goes back to the plain file. [cursor only]
5. The file blame header has a find box. [cursor only]
6. Setting `cursor.blame.hoverDelay` chooses how many milliseconds pass before a blame hover opens, with values 0 to 10000, and when unset it follows `editor.hover.delay` but never less than 500. [cursor only]
7. The Cursor Blame submenu, command and view appear only when the cursor_blame feature gate is on and the team has it enabled. [cursor only]

### ED-6.40
1. Choosing View commit in the editor's Cursor Blame submenu (`cursorBlame.openCommit`) opens the commit of the line under the cursor in a pinned Cursor Blame tab. [cursor only]
2. When there is no Git repository, or the line has not been committed yet, a message reading "No git repository found" or "This line has not been committed yet" shows instead of opening the tab. [cursor only]
3. View Commit in the Command Palette (`cursorBlame.viewCommit`), category Cursor Blame, asks for a commit hash with the clipboard text filled in to start. [cursor only]
4. Open in Cursor Blame on a Source Control history item (`workbench.scm.action.graph.openInCursorBlame`, group 0_view) opens that commit in the tab. [cursor only]
5. The Open in Cursor Blame link in a file blame hover opens that commit in the tab. [cursor only]
6. The tab shows the commit's subject and message, with an All Changes tab and a Chats tab. [cursor only]
7. The tab has an Open in GitHub button. [cursor only]
8. A commit with very many files shows a banner reading "Large commit (N files). Diffs load when expanded." [cursor only]

### ED-6.41
1. When the two sides of a diff differ only by leading or trailing spaces and those are being ignored, the diff shows no changes and a floating button labelled Show Whitespace Differences appears in the editor.
2. Clicking that button turns `diffEditor.ignoreTrimWhitespace` off for that file, so the whitespace-only changes show up.
3. The button does not appear in a diff editor that is embedded inside another widget.

### ED-6.42
1. Dragging the divider between the left and right side of a side-by-side diff gives one side more room and the other less.
2. Double-clicking the divider puts it back in the middle.
3. Each side keeps at least 100 pixels of width, so the divider stops there.

### ED-6.43
1. In the inline view, hovering a block of removed lines shows a lightbulb-style button in the margin.
2. The button's menu offers Copy deleted lines, which copies all the removed lines, and Copy changed line, which copies one line.
3. The button's menu offers Revert this change when the file is editable.
4. Selecting text inside the removed lines and copying puts the original text on the clipboard.
5. The button is drawn only in the regular inline view, so it is not drawn when the diff is side by side (`diffEditor.renderSideBySide` true) or when `diffEditor.experimental.useTrueInlineView` is on.

### ED-6.44
1. Running Compare New Untitled Text Files (`workbench.files.action.compareNewUntitledTextFiles`) opens a pinned diff editor with two empty untitled text files.
2. Typing or pasting text into either side updates the differences between the two sides.
3. The command is in the Command Palette under category File.

### ED-6.45
1. When a save fails because the file on disk is newer, choosing Compare in the failed-save notification opens a pinned diff of the file on disk against your version, titled with "(in file)", "(in the product)" and "Resolve save conflict" (`workbench.files.action.resolveConflict`).
2. The editor title button Use your changes and overwrite file contents, with a check icon, saves your version over the file on disk (`workbench.files.action.acceptLocalChanges`).
3. The editor title button Discard your changes and revert to file contents, with a discard icon, drops your version and reloads the file from disk (`workbench.files.action.revertLocalChanges`).
4. A notification with a Learn More button explains what the two buttons do (`workbench.files.action.resolveConflictLearnMore`).
5. The two buttons show in the editor title only while the save conflict diff is the active editor.
6. Setting `files.saveConflictResolution` chooses what happens on a save conflict, with values `askUser` (default, refuse to save and ask) and `overwriteFileOnDisk` (overwrite the file on disk with the editor's changes).

### ED-6.46
1. A folded bar of hidden lines also lists the functions, classes or other symbols that contain the hidden lines, each with its icon and separated by chevrons.
2. Clicking one of those symbols reveals the hidden lines from where that symbol starts.
3. The symbol list shows only while unchanged regions are being folded (`diffEditor.hideUnchangedRegions.enabled`).

### ED-6.47
1. When more than one source can show a diff for the line peeked from the gutter, the peek title gets a dropdown labelled Switch Quick Diff Base.
2. Choosing a source in the dropdown compares the change against that source.
3. With several sources the peek title reads "<source> - 1 of 3 changes", and with a single source it reads "1 of 3 changes" and has no dropdown.

### ED-6.48
1. While a change is peeked from the gutter, pressing Ctrl+K F2 (Cmd+K F2 on macOS) moves focus from the editor into the peeked diff, and pressing it again moves focus back (`toggleQuickDiffWidgetFocus`). [stock only]
2. The key works only while a quick diff peek is visible. [stock only]

### ED-6.49
1. The header of the Result pane shows a button reading "N Conflicts Remaining", or "1 Conflict Remaining" when one is left.
2. Clicking the button jumps to the next unhandled conflict, and its tooltip reads "Go to next conflict".
3. When no conflicts are left its tooltip reads "All conflicts handled, the merge can be completed now."

### ED-6.50
1. Setting `mergeEditor.showCheckboxes` set to true in settings.json draws the Accept checkboxes beside each change in the merge editor panes.
2. With the setting off, which is the default, the checkboxes are not drawn and the actions above each change are the only per-change controls.
3. The setting is not registered, so it does not appear in the Settings editor and has to be typed into settings.json.

### ED-6.51
1. Setting `diffEditor.hideOriginalLineNumbers` set to true hides the line numbers of the original side in inline diffs. [stock only]
2. Set to false, the original side's line numbers show in inline diffs. [stock only]
3. When the setting is null, compact multi diff editors hide the original line numbers and other diff editors show them. [stock only]

### ED-6.52
1. Open Agent Changes (`cursor.openAgentChangesEditor`) in the Command Palette opens the Review Changes tab for the selected chat's edits. [cursor only]
2. Pressing Ctrl+Shift+R (Cmd+Shift+R on macOS) while the chat input has focus opens the same tab. [cursor only]
3. Close Agent Changes (`cursor.closeAgentChangesEditor`) closes that tab again, and it has no Command Palette entry or key. [cursor only]
4. Close Agent Changes leaves alone Review Changes tabs that were opened in git mode or from Bugbot. [cursor only]

### ED-6.53
1. A file header in Review Changes can show a check button with the hint "Keep changes in this file" that accepts only that file's pending agent edits. [cursor only]
2. A file header in Review Changes can show an x button with the hint "Undo changes in this file" that throws away only that file's pending agent edits. [cursor only]
3. Whether the check and x buttons show on a given file header depends on options the pane passes in, so some headers show neither. [cursor only]

### ED-6.54
1. A file header in Review Changes has an expander button with the hint "Show entire file" that displays the whole file around the changes. [cursor only]
2. After the whole file is shown, the same button's hint reads "Show diff only" and clicking it goes back to just the changed stretches. [cursor only]

### ED-6.55
1. A header above the diffs shows a title and a Changes count with the number of files and the lines added and removed. [cursor only]
2. When the data is there, the header shows an Author with an avatar, and the agent has its own avatar. [cursor only]
3. When the data is there, the header shows Issues and Deployments links. [cursor only]
4. When there is a pull request, the header shows a status chip such as Closed, Draft or Merged. [cursor only]
5. A summary text under the header has a More button that expands it and a Less button that folds it again. [cursor only]
6. The same header is used by Review Changes and by the Cursor Blame commit view. [cursor only]

### ED-6.56
1. In a commit opened in the Cursor Blame editor, a Group By choice under a Manage View button switches the changed-files list between a plain files list and groups by agent conversation. [cursor only]
2. The changed-files list shows as a plain files list by default. [cursor only]
3. With grouping on, files are grouped under a section titled Agent conversations, one group per conversation. [cursor only]
4. Files that no agent touched are listed in a separate Non-agent files section. [cursor only]

### ED-6.57
1. The commit header in Cursor Blame has a Contributions section with a segmented bar whose segments are sized by percent of the commit. [cursor only]
2. Hovering a segment of the bar opens a tooltip and dims the other segments. [cursor only]

### ED-6.58
1. In a commit opened in Cursor Blame, added lines get a margin mark and a line tint in one of six colors, one color per agent conversation. [cursor only]
2. Added lines that were written by Tab get their own margin mark and line tint. [cursor only]
3. Removed lines get matching margin marks and tints. [cursor only]
4. Hovering a margin mark opens a hover after the delay set by `cursor.blame.hoverDelay`, and pressing Escape closes it. [cursor only]

## Unclear

- ED-6.19 and ED-6.50 overlap on the Accept boxes: the boxes are described in ED-6.19, and ED-6.50 says they are not drawn unless `mergeEditor.showCheckboxes` is on, so the tick pieces in ED-6.19 are only reachable with that setting on.
- ED-6.23 piece 10 is a chat hand-off (the Resolve with AI CodeLens and `merge-conflict.resolve-in-chat`); it is kept only as the one visible Cursor difference that makes the feature `changed`, and its chat behavior is out of scope.
- ED-6.24 piece 8 and ED-6.2 piece 7 are marked stock only from searches of the Cursor bundle for labels and ids, not from the screen, so Cursor may still have another way to reach them.
- ED-6.30 does not list a piece for the key of Reject All Edits, because its key expression is written with a key code where a modifier is expected and could not be resolved.
- ED-6.35 pieces 2 to 4 name the agent, git and unstaged modes, but how a person switches between them inside the tab was not found in the bundle strings.
- ED-6.37 and ED-6.40 have unresolved detail: whether Commit, Commit & Push, Push and Create PR are one split button or separate entries, and where the "No git repository found" message appears, were not determined.
- ED-6.53 piece 3 is vague on purpose, because the flags that decide whether the check and x buttons show were not traced.
- ED-6.28 piece 4 gives no default for `cursor.inlineDiff.enablePerformanceProtection`, because the default could not be read cleanly from the bundle.
- ED-6.39 and ED-6.40 both list `cursor.blame.hoverDelay`; the setting piece sits in ED-6.39 and the margin hover use of it sits in ED-6.58, so ED-6.40 has no separate setting piece.
- ED-6.1 lists `diffEditor.renderMarginRevertIcon` and `diffEditor.renderGutterMenu`; their pieces sit in ED-6.7 because that is where their behavior is seen.
- None of the features needed more than 10 pieces.
