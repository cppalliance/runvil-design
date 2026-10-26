Group: SB-2 Explorer. Pass: 3. Date: 2026-10-25.

### SB-2.1
1. Pressing `Ctrl+Shift+E` (`workbench.view.explorer`) or picking Explorer in the View menu opens the side bar on the Explorer container and shows its Folders tree.
2. The Open Editors pane starts hidden, so a fresh Explorer shows only the Folders pane.
3. Running Focus on Files Explorer (`workbench.files.action.focusFilesExplorer`) from the Command Palette puts keyboard focus in the Folders tree so the arrow keys move between rows.
4. Running `workbench.explorer.fileView.focus` moves focus to the Folders pane directly.

### SB-2.2
1. Clicking the twisty arrow on a folder row opens or closes that folder.
2. Setting `workbench.tree.expandMode` chooses whether clicking a folder name opens it on one click (`singleClick`, the default) or on a double click (`doubleClick`).
3. With a folder row focused, Right opens it and Left closes it (`list.expand`, `list.collapse`).
4. Pressing Space on a focused folder row opens or closes it (`list.toggleExpand`).
5. Pressing `Ctrl+Left` closes every open folder in the tree at once (`list.collapseAll`).
6. The Collapse Folders in Explorer button in the Explorer header (also `workbench.files.action.collapseExplorerFolders` in the Command Palette) folds every folder, and in a multi-root workspace it folds the children while keeping each root open.
7. Opening a folder that holds only one child folder also opens that child.

### SB-2.3
1. Clicking a file in the tree opens it in the editor as a preview, shown with an italic tab title.
2. Double-clicking a file opens it as a kept-open editor, and the tab title is no longer italic.
3. Pressing Enter on a focused file opens it and moves keyboard focus to the editor (`explorer.openAndPassFocus`, `Cmd+Down` on macOS).
4. Pressing Space on a focused file opens it and keeps keyboard focus in the tree (`filesExplorer.openFilePreserveFocus`).
5. Setting `workbench.editor.enablePreview` chooses whether a file opened from the tree takes the preview tab or a normal tab, with values on and off.
6. Setting `workbench.list.openMode` chooses whether a file opens on a single click (`singleClick`, the default) or only on a double click (`doubleClick`).

### SB-2.4
1. Right-clicking a file and picking Open to the Side opens it in a new editor group beside the current one.
2. Pressing `Ctrl+Enter` (`Control+Enter` on macOS) while the Explorer has focus opens the selected file beside the current editor (`explorer.openToSide`).
3. With several files selected, Open to the Side opens each of them in the side group.
4. The right-click menu of an Open Editors row has the same Open to the Side entry.

### SB-2.5
1. Right-clicking a file and picking Open With... lets the person choose which editor opens it, such as the text editor or a custom viewer (`explorer.openWith`).
2. The Open With... entry shows only for files whose available editors context key is not empty.
3. The Open With... entry never shows on a folder.

### SB-2.6
1. Clicking the New File button in the Explorer header puts a name box in the tree, and typing a name and pressing Enter creates the file and opens it as a pinned tab.
2. Clicking the New Folder button in the Explorer header puts a name box in the tree, and pressing Enter creates the folder and selects it.
3. Right-clicking a folder and picking New File... or New Folder... shows the name box inside that folder, and with a file selected the box goes in that file's folder.
4. Double-clicking empty space in the tree starts a new file name box.
5. The name box shows an error as you type when the name is empty, starts with a slash, uses characters that are not valid, or is already used in that folder, and a warning for leading or trailing spaces.
6. Pressing Escape in the name box cancels without creating anything.
7. Typing a name that ends in a slash makes a folder instead of a file.
8. Running New File or New Folder from the Command Palette (`workbench.files.action.createFileFromExplorer`, `workbench.files.action.createFolderFromExplorer`) opens the Explorer and shows the same name box.
9. Creating inside a read-only folder fails with the error "Parent folder is readonly.".

### SB-2.7
1. Pressing F2 (Enter on macOS) on the selected row, or picking Rename... in the right-click menu, turns its name into an editable box in place with the file name selected but not its extension (`renameFile`).
2. The rename box shows the same name errors and warnings as the create box, and Enter confirms while Escape cancels.
3. Pressing F2 again inside the rename box moves the selection between the name, the whole text and the extension.
4. Rename... is not offered on workspace root folders or read-only items.

### SB-2.8
1. Pressing Delete (`Cmd+Backspace` on macOS) on the selected rows, or picking Delete in the right-click menu, sends them to the Recycle Bin or Trash and removes them from the tree (`moveFileToTrash`).
2. Pressing `Shift+Delete` (`Cmd+Alt+Backspace` on macOS) deletes the rows permanently (`deleteFile`).
3. Holding Alt while the right-click menu is open changes Delete into Delete Permanently.
4. Setting `files.enableTrash` chooses whether Delete uses the OS trash, and when it is off the Delete key removes items for good.
5. Setting `explorer.confirmDelete` chooses whether a confirmation box appears before deleting, with values on and off and on as the default (the box is DL-4).

### SB-2.9
1. Pressing `Ctrl+X` or picking Cut in the right-click menu marks the selected rows as cut, and they look dimmed until they are pasted.
2. Pressing Escape while rows are cut cancels the cut and the dimming goes away (`filesExplorer.cancelCut`).
3. Pressing `Ctrl+C` or picking Copy in the right-click menu copies the selected rows.
4. Pressing `Ctrl+V` or picking Paste with a folder selected puts copies of the copied items in that folder.
5. Pasting cut items into a folder moves them there.
6. Pasting copied items beside the originals makes a copy with a new name.
7. Setting `explorer.incrementalNaming` chooses the new name of a pasted copy, with values `simple` (adds the word copy and maybe a number), `smart` (adds or raises a number) and `disabled` (asks to overwrite instead).
8. Files copied from the operating system file manager paste into the tree with `Ctrl+V`.
9. Setting `explorer.confirmPasteNative` chooses whether pasting files from the operating system asks first, with values on and off and on as the default (the box is DL-4).

### SB-2.10
1. Pressing `Ctrl+Z` with the Explorer focused undoes the last create, rename, move, copy or delete, and the tree shows the item back as before.
2. Pressing `Ctrl+Y` or `Ctrl+Shift+Z` (`Cmd+Shift+Z` on macOS) with the Explorer focused redoes the undone file operation.
3. The Undo and Redo entries in the Edit menu do the same while the Explorer has focus.
4. Setting `explorer.enableUndo` turns Explorer undo on or off, with on as the default.
5. Setting `explorer.confirmUndo` chooses when an undo asks first, with values `verbose` (before every undo), `default` (before destructive undos) and `light` (no prompt when the Explorer is focused).

### SB-2.11
1. Dragging files or folders onto a folder in the tree moves them into that folder.
2. Holding `Ctrl` (`Alt` on macOS) while dropping copies the dragged items instead of moving them.
3. Holding a dragged item over a closed folder opens that folder after a short wait.
4. Dropping onto a file row puts the dragged items in that file's parent folder.
5. In a multi-root workspace, dragging a root folder onto another root reorders the roots.
6. A confirm box asks before a move unless `explorer.confirmDragAndDrop` is off, with on as the default (the box is DL-4).
7. Setting `explorer.enableDragAndDrop` turns moving by drag and drop inside the Explorer on or off, with on as the default.
8. Setting `explorer.incrementalNaming` chooses the name of a dropped copy when the target folder already holds that name, with values `simple`, `smart` and `disabled`.

### SB-2.12
1. Dropping files or folders from the operating system onto the tree copies them into the folder under the drop.
2. A file dropped in from the operating system opens in the editor by default, and setting `explorer.autoOpenDroppedFile` turns that on or off.
3. Dragging rows from the tree into the editor area opens those files in the editor.
4. Dragging rows from the tree to a place outside the app hands the files to the operating system.
5. Dragging rows from one window's Explorer onto another window's Explorer copies them there.

### SB-2.13
1. Right-clicking a remote file or folder and picking Download... saves a copy of it to the local machine (`explorer.download`).
2. The Download... entry does not show on a folder or file in a local desktop session.
3. Right-clicking a writable folder and picking Upload... lets the person pick local files and adds them to that folder (`explorer.upload`).
4. The Upload... entry shows only in the web version and not on read-only folders.

### SB-2.14
1. With `explorer.autoReveal` on, switching to another editor selects and scrolls to that file in the tree, and opens the folders above it if they were closed.
2. Setting `explorer.autoReveal` chooses what happens when a file opens, with values `true` (reveal and select, the default), `false` (do nothing) and `focusNoScroll` (focus the row without scrolling to it).
3. Running Reveal Active File in Explorer View from the Command Palette (`workbench.files.action.showActiveFileInExplorer`) selects and scrolls to the active file on demand.
4. Picking Reveal in Explorer View from an editor tab's right-click menu (`revealInExplorer`) selects that tab's file in the tree.
5. Files matching `explorer.autoRevealExclude` are skipped by auto reveal, with `**/node_modules` and `**/bower_components` as the default entries.

### SB-2.15
1. Right-clicking a file or folder and picking Reveal in File Explorer shows it in the operating system's file manager (`revealFileInOS`), and the label reads Reveal in Finder on macOS and Open Containing Folder on Linux.
2. The Open Editors menu and the editor tab menu have the same Reveal in File Explorer entry.
3. Pressing `Ctrl+Alt+R` (`Shift+Alt+R` on Windows) while no editor text has focus reveals the selected item in the file manager.
4. Pressing `Ctrl+K R` reveals the active editor's file in the file manager (`workbench.action.files.revealActiveFileInWindows`).
5. These entries and keys exist only in the desktop app and not in a browser session.

### SB-2.16
1. Right-clicking a file or folder and picking Copy Path puts its full path on the clipboard (`copyFilePath`).
2. Picking Copy Relative Path puts the path relative to the workspace folder on the clipboard (`copyRelativeFilePath`).
3. With several rows selected, each copy entry puts one path per line on the clipboard.
4. The Open Editors menu and the editor tab menu have the same Copy Path and Copy Relative Path entries.
5. Pressing `Ctrl+Alt+C` (`Shift+Alt+C` on Windows) copies the path and `Ctrl+Shift+Alt+C` (`Ctrl+K Ctrl+Shift+C` on Windows) copies the relative path while no editor text has focus, and `Ctrl+K P` copies the path of the active file.
6. Copy Path of Active File (`workbench.action.files.copyPathOfActiveFile`) and Copy Relative Path of Active File in the Command Palette copy the path of the active file.
7. Settings `explorer.copyPathSeparator` and `explorer.copyRelativePathSeparator` choose the slash style of the copied path, with values `/`, `\` and `auto` (the operating system's own, the default).

### SB-2.17
1. Right-clicking a file and picking Select for Compare marks it as the left side of a comparison.
2. Right-clicking another file and picking Compare with Selected opens a diff of the two files, and that entry shows only after a file has been marked.
3. Selecting exactly two files and picking Compare Selected opens a diff of the two.
4. The compare entries are not offered on folders.
5. The Open Editors menu and the editor tab menu have the same compare entries.

### SB-2.18
1. Right-clicking a folder and picking Open in Integrated Terminal opens a terminal whose working folder is that folder (`openInIntegratedTerminal`).
2. Picking Open in Integrated Terminal on a file opens a terminal in that file's folder.
3. Picking Open in External Terminal starts the system terminal in the chosen folder (`openInTerminal`).
4. Setting `terminal.explorerKind` chooses which entries show for local files, with values `integrated` (the default), `external` and `both`.
5. In a remote window the integrated terminal opens even when the external entry is chosen.

### SB-2.19
1. Right-clicking a root folder and picking Add Folder to Workspace... opens a folder picker and adds the chosen folder as another root (`addRootFolder`).
2. Right-clicking a root folder and picking Remove Folder from Workspace takes that root out of the tree (`removeRootFolder`).
3. The add and remove entries show only when multi-root workspaces are supported or a workspace file is open.
4. A multi-root workspace lists each root folder as its own top level row in the Folders tree.
5. Setting `explorer.expandSingleFolderWorkspaces` chooses whether a workspace holding only one folder starts expanded when the window opens, with on as the default.

### SB-2.20
1. Clicking the Refresh Explorer button in the Explorer header reloads the tree from disk.
2. Running Refresh Explorer from the Command Palette (`workbench.files.action.refreshFilesExplorer`) does the same.
3. The refresh is disabled while the tree find box is filtering.

### SB-2.21
1. A folder that holds only one subfolder shows on a single row as a joined path such as `src/main/java`.
2. With a joined row focused, Left and Right move focus to the previous or next part of the path (`previousCompressedFolder`, `nextCompressedFolder`).
3. With a joined row focused, Home and End jump to the first or last part of the path (`firstCompressedFolder`, `lastCompressedFolder`).
4. Setting `explorer.compactFolders` turns joining on or off, with on as the default, and with it off every folder gets its own row.
5. The setting is ignored when a screen reader is in use, so folders show on separate rows.

### SB-2.22
1. With file nesting on, related files group under one parent file, for example `package.json` with `package-lock.json` and `yarn.lock` beneath it.
2. A parent file row gets a twisty, and its nested files show indented under it when it is open.
3. Clicking a nesting parent's name opens the file, and only its twisty opens or closes the nest.
4. Setting `explorer.fileNesting.enabled` turns nesting on or off, with off as the default.
5. Setting `explorer.fileNesting.expand` chooses whether nests start open, with on as the default.
6. Setting `explorer.fileNesting.patterns` lists which files nest under which parent, as parent patterns with comma separated child patterns.

### SB-2.23
1. By default the tree sorts items by name with folders listed before files.
2. Setting `explorer.sortOrder` chooses the order, with values `default`, `mixed`, `filesFirst`, `type`, `modified` and `foldersNestsFiles`.
3. With `mixed` files and folders sort together by name, and with `filesFirst` files are listed before folders.
4. With `type` items group by extension and then by name with folders first, and with `modified` they sort by last modified date, newest first, with folders first.
5. With `foldersNestsFiles` folders come first and files that have nested children come before other files.
6. Setting `explorer.sortOrderLexicographicOptions` chooses how upper and lower case names sort, with values `default` (mixed together), `upper` (uppercase first), `lower` (lowercase first) and `unicode` (Unicode order).
7. Setting `explorer.sortOrderReverse` reverses the sort order, with off as the default.

### SB-2.24
1. Files and folders that match the `files.exclude` patterns do not show in the tree, and the default patterns hide `.git`, `.svn`, `.hg`, `.DS_Store` and `Thumbs.db`.
2. Setting `explorer.excludeGitIgnore` also hides everything that `.gitignore` lists, with off as the default.
3. A hidden file that is open in an editor shows in the tree again with a dimmed name.

### SB-2.25
1. Rows take on a color and a letter badge from Git status, such as `M` for modified and `U` for untracked.
2. Rows take on error and warning colors and marks from Problems.
3. A symbolic link shows a bent arrow badge with the status text Symbolic Link.
4. A file of an unknown type shows a question mark badge.
5. A root folder that cannot be read shows a red exclamation mark.
6. A file hidden by the exclude patterns but open in an editor shows with a dimmed name.
7. Settings `explorer.decorations.colors` and `explorer.decorations.badges` turn decoration colors and badges off, with both on as the default.
8. Settings `git.decorations.enabled` and `problems.decorations.enabled` turn the Git marks and the Problems marks on or off.

### SB-2.26
1. Pressing `Ctrl+Alt+F` or F3 with the tree focused opens the find box over the tree (`list.find`).
2. In highlight mode, typing in the find box highlights the matching rows and the up and down keys visit only those matches.
3. In filter mode, typing in the find box hides every row that does not match, and `list.toggleFindMode` switches between highlight and filter.
4. Setting `workbench.list.defaultFindMode` chooses the starting mode, with values `highlight` (the default) and `filter`.
5. Setting `workbench.list.defaultFindMatchType` chooses fuzzy or contiguous matching, with `fuzzy` as the default, and `list.toggleFindMatchType` switches between them.
6. The search also finds items inside folders that were never opened, and it skips what `search.exclude` and `explorer.excludeGitIgnore` hide.
7. A folder that holds matches shows a count badge with the hover text "Directory contains N matches".
8. When there are too many matches the find box warns that the result set only contains a subset of all matches.
9. Pressing Escape closes the find box (`list.closeFind`).

### SB-2.27
1. Typing a name with the tree focused jumps focus to the row whose name matches what was typed.
2. Setting `workbench.list.typeNavigationMode` chooses when typing jumps, with values `automatic` (the default) and `trigger`.
3. With `trigger`, typing jumps only after the `list.triggerTypeNavigation` command has been run.

### SB-2.29
1. Setting `workbench.tree.indent` sets how many pixels each folder level is indented, from 4 to 40, with 8 as the default.
2. Setting `workbench.tree.renderIndentGuides` chooses when vertical indent guide lines show, with values `none`, `onHover` and `always`.
3. With the default `onHover`, the guide lines show while the pointer is over the tree.
4. Both settings apply to every tree in the workbench and not only the Explorer.

### SB-2.30
1. The Open Editors pane lists every open editor as one row.
2. When there is more than one editor group, each group gets a heading row with the editors listed beneath it.
3. Each group heading has a Save All in Group button and a Close Group button.
4. Each row shows the file icon and name, and the name is italic for a preview editor.
5. Each row has a close button, a pinned editor shows an unpin button in its place, and a row with unsaved changes shows a dot in place of the button.
6. Clicking a row switches to that editor, and middle-clicking a row closes it.
7. The pane starts hidden and is shown from the Explorer Views menu (SB-1), and `Ctrl+K E` (`Cmd+K E` on macOS) focuses it.
8. Setting `explorer.openEditors.visible` sets the most editors shown before a scroll bar appears, with 9 as the default.
9. Setting `explorer.openEditors.minVisible` sets how many editor slots the pane keeps free, with 0 as the default, which lets the pane resize to the number of editors.

### SB-2.31
1. Clicking the Save All button in the Open Editors title bar saves every editor with unsaved changes (`workbench.action.files.saveAll`).
2. Clicking the Close All Editors button closes every open editor (`openEditors.closeAll`).
3. Clicking the New Untitled Text File button opens a new untitled text file (`openEditors.newUntitledFile`).
4. The Toggle Vertical/Horizontal Editor Layout button shows only when there is more than one editor group and switches the groups between side by side and stacked.
5. Pressing `Shift+Alt+0` (`Cmd+Alt+0` on macOS) or picking the entry in the Layout menu does the same as the layout button (`workbench.action.toggleEditorGroupLayout`).

### SB-2.32
1. Right-clicking an editor row shows Open to the Side and Reveal in File Explorer, and Open to the Side is offered only for file and untitled editors.
2. Reopen Editor With... shows for editors that have other editors to choose from, and not on a group heading.
3. Copy Path and Copy Relative Path show for editors whose resource is a file on disk.
4. Save and Revert File are greyed out unless the editor has unsaved changes, and Revert File is hidden for read-only editors.
5. Compare with Saved shows for file editors with unsaved changes and is hidden while `files.autoSave` saves after a short delay.
6. Select for Compare and Compare with Selected show for file and untitled editors, and Compare Selected shows when two rows are selected.
7. Close, Close Others, Close Saved and Close All end the menu, and Close and Close Others are not on a group heading.
8. Save All shows only when the right-click lands on a group heading, and it is greyed out when nothing is unsaved.

### SB-2.33
1. Setting `explorer.openEditors.sortOrder` chooses how the Open Editors list is ordered, with values `editorOrder`, `alphabetical` and `fullPath`.
2. With `editorOrder`, the default, rows follow the order of the editor tabs.
3. With `alphabetical`, rows are sorted by tab name inside each editor group.
4. With `fullPath`, rows are sorted by full path inside each editor group.
5. Dragging rows to reorder them is turned off unless the order is `editorOrder`.

### SB-2.34
1. A badge in the Open Editors title shows how many files have unsaved changes, with text such as "2 unsaved".
2. The badge hides itself when no file has unsaved changes.
3. With `files.autoSave` set to save after a short delay, files that will be saved on their own are not counted in the badge.

### SB-2.35
1. Dragging an editor row up or down inside a group reorders the editors when the list is in tab order.
2. Dragging an editor row onto another group, or onto its heading, moves that editor into that group.
3. Dropping files from the operating system onto the list opens them in editors.
4. Dragging a row out to the editor area opens that editor there.
5. Dragging a row to reorder is refused when `explorer.openEditors.sortOrder` is not `editorOrder`.

### SB-2.36
1. With no folder or workspace open, the Explorer shows a No Folder Opened view with the text "You have not yet opened a folder." and an Open Folder button.
2. Clicking the Open Folder button opens the folder picker (`workbench.action.files.openFolder`).
3. When editors are open, the text adds "Opening a folder will close all currently open editors. To keep them open, add a folder instead." and the add a folder link runs `workbench.action.addRootFolder`.
4. When a workspace file with no folders is open, the text reads "You have not yet added a folder to the workspace." with a button that adds a folder.
5. In a remote window with no folder open, the text reads "Connected to remote." with an Open Folder button.
6. When a workspace file is open and entering workspaces is not supported, the text offers Open Folder (`workbench.action.files.openFolderViaWorkspace`) and Open Recent (`workbench.action.openRecent`) buttons.
7. Dropping a folder or file onto the view opens it.

### SB-2.37
1. Below the Open Folder text, the empty Explorer shows the line "You can clone a repository locally" with a Clone Repository button.
2. Clicking Clone Repository starts the clone flow (`git.clone`).
3. A Learn more line about Git and source control follows the clone button.
4. The clone text shows only when `git.enabled` is on and no source control provider is registered.

### SB-2.38
1. Right-clicking a local file and picking Open in Browser opens it in a Browser tab inside the app, and the entry is offered for html, svg, pdf, text, Markdown, JSON, XML, YAML, CSV, log, CSS, script and image files (`filesExplorer.openFileInBrowser`). [cursor only]
2. The Open in Browser entry is in the Explorer right-click menu only, with no entry in the editor tab menu or the editor title bar. [cursor only]
3. Right-clicking a local `.html` or `.htm` file and picking Open in Integrated Browser opens it in a browser tab inside the app (`workbench.action.browser.openFile`). [stock only]
4. The Open in Integrated Browser entry also shows in the editor tab right-click menu and as a globe button in the editor title bar for local `.html` and `.htm` files. [stock only]

### SB-2.39
1. Right-clicking a file and picking Cursor Blame opens that file's blame view (`cursorBlame.openFileBlame`). [cursor only]
2. The Command Palette lists Open File Blame (`cursorBlame.openFileBlame`). [cursor only]
3. The Cursor Blame entry is not shown on folders or workspace roots. [cursor only]
4. The Cursor Blame entry shows only when the `cursor_blame` feature gate and a team admin setting are both on. [cursor only]

### SB-2.40
1. Right-clicking a folder, or an image or video file, and picking Open in Images Preview opens the images of that folder in the Images Preview viewer (`workbench.action.openImagesInCarousel`). [stock only]
2. Setting `imageCarousel.explorerContextMenu.enabled` turns the Open in Images Preview entry on or off, with on as the default, and is tagged experimental. [stock only]
3. The Explorer right-click menu in Cursor has no Open in Images Preview entry and no setting for it. [stock only]

### SB-2.41
1. Holding `Ctrl` (`Cmd` on macOS) and clicking a row adds it to or removes it from the selection.
2. Holding Shift and clicking a row selects the whole range from the last selected row.
3. Pressing `Shift+Down` or `Shift+Up` extends the selection by one row (`list.expandSelectionDown`, `list.expandSelectionUp`).
4. Pressing `Ctrl+A` selects the visible rows under the focused row's parent folder (`list.selectAll`).
5. Pressing `Ctrl+Shift+Enter` adds the focused row to the selection or removes it (`list.toggleSelection`).
6. Pressing Escape clears the selection (`list.clear`).
7. Delete, Cut, Copy, Compare Selected, Open to the Side and dragging act on every selected row.
8. Rename and the New File entries need exactly one row and are not offered when several are selected.
9. Setting `workbench.list.multiSelectModifier` chooses the click modifier for adding rows, with values `ctrlCmd` (the default) and `alt`, and with `alt` the Open to the Side mouse gestures change so they do not clash.

### SB-2.42
1. Setting `workbench.list.horizontalScrolling` lets long names scroll sideways, with off as the default, and the Explorer turns it off while a name box is open.
2. Setting `workbench.list.smoothScrolling` makes the tree scroll smoothly, with off as the default.
3. Setting `workbench.list.scrollByPage` makes a click in the scroll bar scroll a page at a time, with off as the default, and with it on a double click on blank space no longer starts a new file.
4. Setting `workbench.list.mouseWheelScrollSensitivity` sets the mouse wheel speed multiplier, with 1 as the default.
5. Setting `workbench.list.fastScrollSensitivity` sets the speed multiplier while Alt is held, with 5 as the default.
6. Pressing `Ctrl+Up` or `Ctrl+Down` nudges the view by 10 pixels without moving the focused row, and only when the view is not already at the top or bottom (`list.scrollUp`, `list.scrollDown`).
7. Scroll Left and Scroll Right move the view sideways and have no default keys (`list.scrollLeft`, `list.scrollRight`).
8. The five scrolling settings apply to every tree and list in the workbench and not only the Explorer.

### SB-2.43
1. Resting the pointer on a row shows a tooltip with the full path of that file or folder.
2. The tooltip adds a dot and then the status text of any decoration on the row, such as a Git state or Symbolic Link.
3. Pressing `Ctrl+K Ctrl+I` with a row focused shows the same tooltip from the keyboard (`list.showHover`).
4. Setting `workbench.hover.delay` sets the delay in milliseconds before the tooltip appears, with 500 as the default (1500 on macOS).

### SB-2.44
1. The Folders pane header reads the name of the opened folder.
2. For an open workspace file, the header reads the workspace file's name followed by (Workspace).
3. For an unsaved workspace, the header reads Untitled (Workspace).
4. For a remote folder, the header adds a remote suffix to the name.
5. The header updates when the workspace is renamed.
6. A screen reader announces the header as Explorer Section followed by the name.
7. Dragging something over the header while the pane is folded opens the pane after a short wait.
8. When Folders is the only pane showing, the container title carries the name instead (SB-1.20).

### SB-2.45
1. Closing the window and opening the same workspace again shows the same folders unfolded as before.
2. The reopened tree has the same row selected and the same scroll position.
3. In a multi-root workspace with no saved tree state, up to five roots open on their own.
4. A root folder added to the workspace opens when it appears.

### SB-2.46
1. Files created, deleted or renamed outside the app show up in the tree on their own after a short delay.
2. The tree also refreshes when the window gets focus again.
3. The tree holds off refreshing while a name box is open.
4. Every workspace folder is watched except the paths named in `files.watcherExclude`.
5. Setting `files.watcherInclude` adds extra paths to watch, such as symbolic link folders.
6. Changing `files.watcherExclude` or `files.watcherInclude` restarts watching without reloading the window.
7. When the system cannot watch, a warning notification says "Unable to watch for file changes" and has an Instructions button (NT-2 owns the notification).
8. When the watcher dies, a warning notification says "File changes watcher stopped unexpectedly" and has a Reload button (NT-2 owns the notification).

### SB-2.47
1. With a row focused and plain text on the clipboard, pressing `Ctrl+V` makes a new file named Untitled holding that text. [cursor only]
2. The new file goes in the focused folder, in the folder of the focused file, or in the first root when no row is focused. [cursor only]
3. When the name is taken the new file is named Untitled 1, Untitled 2 and so on. [cursor only]
4. The new file opens as a pinned tab without taking focus, and its name goes straight into the rename box so the person can type the real name. [cursor only]
5. Nothing happens when the clipboard is blank or holds files (files paste as in SB-2.9), or when a name box is already open. [cursor only]
6. Nothing happens when the focused row is read-only, and a read-only folder gives the error "Parent folder is readonly.". [cursor only]

### SB-2.48
1. Dragging an editor tab onto a folder in the tree copies that file into the folder, and does not move it.
2. When the folder already holds a file of that name, a box asks before replacing it (DL-4 owns the box).
3. The drop is refused when the target folder is read-only.
4. The drop is refused when `explorer.enableDragAndDrop` is off.
5. Rows dragged from the Open Editors list are refused by the tree.

### SB-2.49
1. Right-clicking a file or folder and picking Run Tests runs every test found under it, with results in the Test Explorer (SB-6) (`testing.run.uri`).
2. Picking Debug Tests runs the tests under the item in debug mode (`testing.debug.uri`).
3. Picking Run Tests with Coverage runs the tests under the item and collects coverage (`testing.coverage.uri`).
4. Each of the three entries shows only while some test provider offers that kind of run, so an install with no test extension shows none of them.
5. When no tests are found under the item, an info notification says "No tests found in the selected file or folder".

## Unclear

- SB-2.38 mixes Cursor and stock behavior, so the Cursor pieces are tagged `[cursor only]` and the stock pieces `[stock only]`, and the exact Cursor file type list was taken from the feature block and not re-read from the bundle.
- SB-2.5: whether Open With... shows for every file or only for files with more than one editor was not settled in pass 2, so piece 2 states only the context key test.
- SB-2.43: the 1500 ms macOS default of `workbench.hover.delay` was read from `V\src\vs\workbench\browser\workbench.contribution.ts` and Cursor's own default was not compared.
- SB-2.41 and SB-2.42 describe workbench-wide list behavior seen through the Explorer, and pass 4 may want to merge or move them as noted in the pass 2 file.
