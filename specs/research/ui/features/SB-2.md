Group: SB-2 Explorer. Section: Side bar. Date: 2026-10-25. Pass: 2.
Features: 48.

### SB-2.1 Show and focus the Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Shift+E or pick Explorer in the View menu to show the Explorer container with its Folders tree (Open Editors starts hidden), or run Focus on Files Explorer to put the keyboard in the tree. In Cursor the Explorer is the first pinned tab of the Cursor tab row, which AB-6 owns.
- Commands: workbench.view.explorer; workbench.files.action.focusFilesExplorer; workbench.explorer.fileView.focus
- Menus: MenubarViewMenu; CommandPalette
- Keys: Ctrl+Shift+E (workbench.view.explorer)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts; V\src\vs\workbench\contrib\files\browser\fileActions.ts (FocusFilesExplorer); V\src\vs\workbench\services\views\browser\viewsService.ts; C bundle (container id workbench.view.explorer, FocusFilesExplorer id, focusCommand workbench.explorer.fileView.focus)

### SB-2.2 Expand and collapse folders
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a folder twisty or name, or press Right and Left, to open and close folders; Ctrl+Left collapses the whole tree, and the Collapse Folders in Explorer header button folds every folder (in a multi-root workspace it folds the children and keeps the roots open). Folders with a single child open on their own.
- Commands: workbench.files.action.collapseExplorerFolders; list.expand; list.collapse; list.collapseAll; list.toggleExpand
- Menus: ViewTitle (view == workbench.explorer.fileView); CommandPalette
- Keys: Right (list.expand); Left (list.collapse); Ctrl+Left (list.collapseAll); Space (list.toggleExpand)
- Settings: workbench.tree.expandMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (collapseAll, header button); V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts; C bundle (workbench.files.action.collapseExplorerFolders, list.collapseAll, workbench.tree.expandMode)

### SB-2.3 Open a file from the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Click a file to open it in the editor as a preview (italic tab), double-click to keep it open, press Enter to open it and move focus to the editor (Cmd+Down on macOS), or press Space to open it and keep focus in the tree.
- Commands: explorer.openAndPassFocus; filesExplorer.openFilePreserveFocus; list.select
- Menus: none found
- Keys: Enter (explorer.openAndPassFocus, when a non-folder is focused; macOS Cmd+Down); Space (filesExplorer.openFilePreserveFocus, when a non-folder is focused)
- Settings: workbench.editor.enablePreview; workbench.list.openMode; workbench.tree.expandMode
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (tree.onDidOpen); V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\platform\list\browser\listService.ts; C bundle (explorer.openAndPassFocus, filesExplorer.openFilePreserveFocus, workbench.list.openMode)

### SB-2.4 Open a file to the side
- Tag: both
- Evidence: source
- Boundary: no
- What: Open the selected file or files in a new editor group beside the current one from Open to the Side in the right-click menu or with Ctrl+Enter (Control+Enter on macOS). The same entry is in the Open Editors menu.
- Commands: explorer.openToSide
- Menus: ExplorerContext; OpenEditorsContext
- Keys: Ctrl+Enter (when the Explorer viewlet has focus; macOS Control+Enter)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (explorer.openToSide, label Open to the Side)

### SB-2.5 Open a file with another editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Open With... in the right-click menu on a file to choose which editor opens it (for example the text editor or a custom viewer). The entry shows for files that have editors listed in the available editors context key, and never for folders.
- Commands: explorer.openWith
- Menus: ExplorerContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\common\files.ts (ExplorerResourceAvailableEditorIdsContext); C bundle (explorer.openWith, label Open With...)

### SB-2.6 Create a file or folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Make a new file or folder with a name box typed straight into the tree. Use the New File and New Folder buttons in the Explorer header, New File... and New Folder... in the right-click menu on a folder, or double-click empty space for a new file. The box checks the name as you type (empty, leading slash, invalid characters, name already used).
- Commands: explorer.newFile; explorer.newFolder; workbench.files.action.createFileFromExplorer; workbench.files.action.createFolderFromExplorer
- Menus: ExplorerContext; ViewTitle (view == workbench.explorer.fileView); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (openExplorerAndCreate, validateFileName); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (header buttons, double-click on empty area); C bundle (explorer.newFile, explorer.newFolder, createFileFromExplorer, createFolderFromExplorer)

### SB-2.7 Rename a file or folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Press F2 (Enter on macOS) or pick Rename... in the right-click menu to edit the name in place; the box shows the same name checks as creating. Not offered on workspace root folders or read-only items.
- Commands: renameFile
- Menus: ExplorerContext
- Keys: F2 (renameFile, files Explorer focused, item not a root and writable; macOS Enter)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (renameHandler, validateFileName); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (renameFile)

### SB-2.8 Delete a file or folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Delete (Cmd+Backspace on macOS) or pick Delete in the right-click menu to send items to the Recycle Bin or Trash, or press Shift+Delete or hold Alt on the menu to delete permanently. With the OS trash turned off, Delete removes for good. The confirm box is owned by DL-4.
- Commands: moveFileToTrash; deleteFile
- Menus: ExplorerContext
- Keys: Delete (moveFileToTrash; macOS Cmd+Backspace or Delete); Shift+Delete (deleteFile; macOS Cmd+Alt+Backspace); Delete (deleteFile, when trash is unavailable; macOS Cmd+Backspace)
- Settings: files.enableTrash; explorer.confirmDelete
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (moveFileToTrashHandler, deleteFileHandler); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (moveFileToTrash, deleteFile, files.enableTrash, explorer.confirmDelete, label Delete Permanently)

### SB-2.9 Cut, copy and paste files
- Tag: both
- Evidence: source
- Boundary: no
- What: Cut, Copy and Paste files and folders with Ctrl+X, Ctrl+C and Ctrl+V or the right-click menu. Cut items look dimmed until pasted, and Esc cancels the cut. Pasting beside the original makes a copy with a new name, and files copied from the operating system paste in too.
- Commands: filesExplorer.cut; filesExplorer.copy; filesExplorer.paste; filesExplorer.cancelCut
- Menus: ExplorerContext
- Keys: Ctrl+X (filesExplorer.cut); Ctrl+C (filesExplorer.copy); Ctrl+V (filesExplorer.paste); Escape (filesExplorer.cancelCut, when something is cut)
- Settings: explorer.incrementalNaming; explorer.confirmPasteNative
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (cutFileHandler, copyFileHandler, pasteFileHandler, findValidPasteFileTarget); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (paste event, itemsCopied); C bundle (filesExplorer.cut, copy, paste, cancelCut, explorer.incrementalNaming, explorer.confirmPasteNative)

### SB-2.10 Undo and redo file operations
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Z in the Explorer to undo the last create, rename, move, copy or delete, and Ctrl+Y to redo. Some undo steps ask first, depending on the confirm level.
- Commands: undo; redo
- Menus: MenubarEditMenu
- Keys: Ctrl+Z (undo); Ctrl+Y or Ctrl+Shift+Z (redo; macOS Cmd+Shift+Z)
- Settings: explorer.enableUndo; explorer.confirmUndo
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\files.contribution.ts (UndoCommand and RedoCommand implementations named explorer); V\src\vs\editor\browser\editorExtensions.ts; C bundle (addImplementation(110,"explorer") for undo, explorer.enableUndo, explorer.confirmUndo)

### SB-2.11 Drag and drop to move or copy in the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag files and folders onto a folder to move them, or hold Ctrl (Alt on macOS) while dropping to copy. Hovering over a folder opens it, dropping on a file uses its parent folder, and in a multi-root workspace dragging a root folder reorders the roots. A confirm box asks before a move unless turned off (DL-4 owns the box).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.enableDragAndDrop; explorer.confirmDragAndDrop; explorer.incrementalNaming
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileDragAndDrop); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.enableDragAndDrop, explorer.confirmDragAndDrop, explorer.incrementalNaming)

### SB-2.12 Drop files in from the OS or drag them out
- Tag: both
- Evidence: source
- Boundary: no
- What: Drop files or folders from the operating system onto the tree to copy them into the folder under the drop (a dropped file opens by default), and drag tree items out to the editor area or outside the app. Dragging between two windows' Explorers also works.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.autoOpenDroppedFile
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileDragAndDrop.drop and onDragStart); V\src\vs\workbench\contrib\files\browser\fileImportExport.ts (ExternalFileImport); C bundle (explorer.autoOpenDroppedFile)

### SB-2.13 Download and upload files
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Download... on a remote file or folder to save a copy to the local machine, and Upload... on a writable folder to add local files (Upload is web only; Download on a local desktop folder does not show). Rarely visible in a local desktop session.
- Commands: explorer.download; explorer.upload
- Menus: ExplorerContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (DOWNLOAD_COMMAND_ID, UPLOAD_COMMAND_ID); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileImportExport.ts (FileDownload, BrowserFileUpload); C bundle (explorer.download, explorer.upload, labels Download... and Upload...)

### SB-2.14 Reveal the active file in the Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: The tree selects and scrolls to the file in the active editor by itself (auto reveal), or you run Reveal Active File in Explorer View, or Reveal in Explorer View on an editor tab menu, when you want it. Paths matching the auto reveal exclude list (node_modules and bower_components by default) are skipped.
- Commands: workbench.files.action.showActiveFileInExplorer; revealInExplorer
- Menus: EditorTitleContext; CommandPalette
- Keys: none found
- Settings: explorer.autoReveal; explorer.autoRevealExclude
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (ShowActiveFileInExplorer); V\src\vs\workbench\contrib\files\browser\fileCommands.ts (REVEAL_IN_EXPLORER_COMMAND_ID); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (selectActiveFile, selectResource); V\src\vs\workbench\contrib\files\browser\explorerService.ts; C bundle (showActiveFileInExplorer, revealInExplorer, explorer.autoReveal, explorer.autoRevealExclude)

### SB-2.15 Reveal a file in the OS file manager
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Reveal in File Explorer (Reveal in Finder on macOS, Open Containing Folder on Linux) on a file or folder, or press Ctrl+Alt+R (Shift+Alt+R on Windows), to show it in the operating system's file manager. Ctrl+K R does the same for the active editor. Desktop only.
- Commands: revealFileInOS; workbench.action.files.revealActiveFileInWindows
- Menus: ExplorerContext; OpenEditorsContext; EditorTitleContext; CommandPalette
- Keys: Ctrl+Alt+R (revealFileInOS, when no editor text has focus; Windows Shift+Alt+R); Ctrl+K R (workbench.action.files.revealActiveFileInWindows)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\electron-browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\electron-browser\fileCommands.ts; C bundle (revealFileInOS, workbench.action.files.revealActiveFileInWindows)

### SB-2.16 Copy a file path
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Copy Path or Copy Relative Path on a file or folder (several items copy one path per line) to put the path on the clipboard. Keys and Command Palette entries do the same for the active file, and two settings choose the slash style.
- Commands: copyFilePath; copyRelativeFilePath; workbench.action.files.copyPathOfActiveFile
- Menus: ExplorerContext; OpenEditorsContext; EditorTitleContext; CommandPalette
- Keys: Ctrl+Alt+C (copyFilePath, no editor focus; Windows Shift+Alt+C); Ctrl+Shift+Alt+C (copyRelativeFilePath, no editor focus; Windows Ctrl+K Ctrl+Shift+C); Ctrl+K P (workbench.action.files.copyPathOfActiveFile)
- Settings: explorer.copyPathSeparator; explorer.copyRelativePathSeparator
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (copyFilePath, copyRelativeFilePath, explorer.copyPathSeparator, explorer.copyRelativePathSeparator)

### SB-2.17 Compare two files from the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a file and pick Select for Compare, then right-click another file and pick Compare with Selected to open a diff, or select exactly two files and pick Compare Selected. Folders are not offered. The diff editor itself is ED-6.
- Commands: selectForCompare; compareFiles; compareSelected
- Menus: ExplorerContext; OpenEditorsContext; EditorTitleContext
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileConstants.ts; C bundle (selectForCompare, compareFiles, compareSelected, labels Select for Compare and Compare with Selected)

### SB-2.18 Open a folder in a terminal
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick Open in Integrated Terminal to start a terminal in the chosen folder (or a file's folder), or Open in External Terminal to start the system terminal there. A setting decides which of the two entries show. The terminal itself is PN-2.
- Commands: openInIntegratedTerminal; openInTerminal
- Menus: ExplorerContext
- Keys: none found
- Settings: terminal.explorerKind
- Cursor: present
- Source: V\src\vs\workbench\contrib\externalTerminal\browser\externalTerminal.contribution.ts; V\src\vs\workbench\contrib\externalTerminal\electron-browser\externalTerminal.contribution.ts (terminal.explorerKind); C bundle (openInIntegratedTerminal, openInTerminal, terminal.explorerKind)

### SB-2.19 Manage workspace root folders
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a root folder and pick Add Folder to Workspace... or Remove Folder from Workspace (shown when multi-root workspaces are supported or a workspace file is open). A multi-root workspace lists each root at the top level, and a setting expands a workspace that holds only one folder when the window starts. Open Folder Settings on a root is handed off.
- Commands: addRootFolder; removeRootFolder; workbench.action.removeRootFolder
- Menus: ExplorerContext
- Keys: none found
- Settings: explorer.expandSingleFolderWorkspaces
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\workbench\contrib\files\browser\fileCommands.ts (REMOVE_ROOT_FOLDER_COMMAND_ID); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (setTreeInput); C bundle (addRootFolder, removeRootFolder, labels Add Folder to Workspace and Remove Folder from Workspace, explorer.expandSingleFolderWorkspaces)

### SB-2.20 Refresh the Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Press the Refresh Explorer button in the Explorer header, or run the command, to reload the tree from disk. It is disabled while the tree find box is filtering.
- Commands: workbench.files.action.refreshFilesExplorer
- Menus: ViewTitle (view == workbench.explorer.fileView); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (refreshFilesExplorer action); C bundle (workbench.files.action.refreshFilesExplorer)

### SB-2.21 Compact folders
- Tag: both
- Evidence: source
- Boundary: no
- What: A folder that holds only one subfolder is shown on one row as a joined path (for example src/main/java). With such a row focused, Left, Right, Home and End move between the parts of the path. The setting is ignored when a screen reader is in use.
- Commands: previousCompressedFolder; nextCompressedFolder; firstCompressedFolder; lastCompressedFolder
- Menus: none found
- Keys: Left (previousCompressedFolder, compact row focused); Right (nextCompressedFolder); Home (firstCompressedFolder); End (lastCompressedFolder)
- Settings: explorer.compactFolders
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (isCompressionEnabled, compressed navigation); V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (CompressedNavigationController); V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (previousCompressedFolder, nextCompressedFolder, firstCompressedFolder, lastCompressedFolder, explorer.compactFolders)

### SB-2.22 File nesting
- Tag: both
- Evidence: source
- Boundary: no
- What: Related files group under one parent file, for example a .ts file with its .js and .d.ts files, and the parent gets a twisty. A setting turns nesting on (it is off by default), another decides whether nests start open, and a pattern list says which files nest under which.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.fileNesting.enabled; explorer.fileNesting.expand; explorer.fileNesting.patterns
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\common\explorerFileNestingTrie.ts; V\src\vs\workbench\contrib\files\common\explorerModel.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.fileNesting.enabled, explorer.fileNesting.expand, explorer.fileNesting.patterns with the same default patterns)

### SB-2.23 Sort order in the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose how files and folders sort: by name with folders first (default), mixed, files first, by file type, by last modified, or folders first with nesting parents ahead of other files. Also reverse the order and pick how upper and lower case names sort.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.sortOrder; explorer.sortOrderLexicographicOptions; explorer.sortOrderReverse
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileSorter); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.sortOrder, explorer.sortOrderLexicographicOptions, explorer.sortOrderReverse)

### SB-2.24 Hide files from the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Files and folders that match the files.exclude patterns (by default .git, .svn, .hg, .DS_Store and Thumbs.db) do not show in the tree, and a setting also hides what .gitignore lists. A hidden file you open in an editor shows up again, dimmed. Both are edited in the Settings editor (ED-8).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.exclude; explorer.excludeGitIgnore
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FilesFilter); V\src\vs\workbench\contrib\files\browser\views\explorerDecorationsProvider.ts (excluded items dimmed); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (files.exclude with the same default patterns, explorer.excludeGitIgnore)

### SB-2.25 File decorations in the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Rows take on colors and letter badges from other features: Git status such as M and U, and error or warning marks from Problems. The Explorer adds its own: a bent arrow on symbolic links, a question mark on unknown file types, a red exclamation on a root folder that cannot be read, and dimmed names for hidden files that are open. Two settings turn colors and badges off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.decorations.colors; explorer.decorations.badges; git.decorations.enabled; problems.decorations.enabled
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerDecorationsProvider.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (fileDecorations); V\src\vs\workbench\contrib\markers\browser\markersFileDecorations.ts; V\src\vs\workbench\services\decorations\browser\decorationsService.ts; V\extensions\git\src\decorationProvider.ts; V\extensions\git\package.json; C bundle (explorer.decorations.colors, explorer.decorations.badges, problems.decorations.enabled, strings Symbolic Link, Unknown File Type, Unable to resolve workspace folder); C\extensions\git\package.json (git.decorations.enabled)

### SB-2.26 Find in the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+Alt+F or F3 with the tree focused to open the find box, type to highlight matches or to filter the tree down to them, switch between fuzzy and contiguous matching, and press Esc to close it. The search also reaches folders that were never opened, and it reads the search excludes.
- Commands: list.find; list.closeFind; list.toggleFindMode; list.toggleFindMatchType
- Menus: none found
- Keys: Ctrl+Alt+F or F3 (list.find, list that supports find is focused); Escape (list.closeFind, tree find box open)
- Settings: workbench.list.defaultFindMode; workbench.list.defaultFindMatchType; search.exclude; explorer.excludeGitIgnore
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (ExplorerFindProvider, searchInWorkspace); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (findProvider); V\src\vs\workbench\contrib\files\common\files.ts (ExplorerFindProviderActive); V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts; C bundle (list.find, list.closeFind, list.toggleFindMode, list.toggleFindMatchType, workbench.list.defaultFindMode, explorerFindProviderActive, ExplorerFindProvider)

### SB-2.27 Type to jump in the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Start typing a name with the tree focused to jump to the matching row. A setting picks whether typing starts the jump at any time (automatic) or only after you run the trigger command.
- Commands: list.triggerTypeNavigation
- Menus: none found
- Keys: none found
- Settings: workbench.list.typeNavigationMode
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\listCommands.ts; V\src\vs\platform\list\browser\listService.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (keyboardNavigationLabelProvider); C bundle (list.triggerTypeNavigation, workbench.list.typeNavigationMode with default automatic)

### SB-2.29 Tree indent and indent guides
- Tag: both
- Evidence: source
- Boundary: no
- What: Set how far each folder level is indented (4 to 40 pixels, default 8) and when the vertical indent guide lines show (never, on hover, or always; default on hover). The same two settings apply to every tree in the workbench.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.tree.indent; workbench.tree.renderIndentGuides
- Cursor: present
- Source: V\src\vs\platform\list\browser\listService.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FilesRenderer indent offset); C bundle (workbench.tree.indent, workbench.tree.renderIndentGuides with the same defaults)

### SB-2.30 Open Editors list
- Tag: both
- Evidence: source
- Boundary: no
- What: The Open Editors pane lists every open editor, with group headings when there is more than one editor group; each heading has Save All in Group and Close Group buttons. Rows show the file icon, an italic name for a preview editor, a pin or close button that turns into a dot for unsaved changes, and click switches while middle-click closes. The pane starts hidden (show it from the Explorer Views menu, SB-1) and Ctrl+K E focuses it.
- Commands: workbench.files.action.focusOpenEditorsView; workbench.files.action.saveAllInGroup; workbench.files.action.closeGroup; workbench.action.closeActiveEditor; workbench.action.unpinActiveEditor
- Menus: none found
- Keys: Ctrl+K E (workbench.files.action.focusOpenEditorsView; macOS Cmd+K E)
- Settings: explorer.openEditors.visible; explorer.openEditors.minVisible
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts; V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts (createOpenEditorsViewDescriptor, hideByDefault); V\src\vs\workbench\contrib\files\common\files.ts (OpenEditor); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (descriptor for workbench.explorer.openEditorsView with hideByDefault true, workbench.files.action.focusOpenEditorsView, explorer.openEditors.visible, explorer.openEditors.minVisible)

### SB-2.31 Open Editors header buttons
- Tag: both
- Evidence: source
- Boundary: no
- What: Buttons in the Open Editors title bar: Save All, Close All Editors, New Untitled Text File, and Toggle Vertical/Horizontal Editor Layout (the last shows only when there is more than one editor group).
- Commands: workbench.action.files.saveAll; openEditors.closeAll; openEditors.newUntitledFile; workbench.action.toggleEditorGroupLayout
- Menus: ViewTitle (view == workbench.explorer.openEditorsView); MenubarLayoutMenu
- Keys: Shift+Alt+0 (workbench.action.toggleEditorGroupLayout; macOS Cmd+Alt+0)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (registerAction2 blocks); C bundle (workbench.action.files.saveAll, openEditors.closeAll, openEditors.newUntitledFile, workbench.action.toggleEditorGroupLayout, label Toggle Vertical/Horizontal Editor Layout)

### SB-2.32 Open Editors right-click menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an editor row for Open to the Side, Reveal in File Explorer, Reopen Editor With..., Copy Path, Copy Relative Path, Save, Revert File, Save All (on a group heading), Compare with Saved, the three compare entries, and Close, Close Others, Close Saved, Close All. Entries show or hide by editor type, unsaved state, read-only state and auto save. Cursor adds nothing to this menu.
- Commands: explorer.openToSide; revealFileInOS; workbench.action.reopenWithEditor; copyFilePath; copyRelativeFilePath; workbench.action.files.save; workbench.action.files.revert; workbench.files.action.saveAllInGroup; workbench.files.action.compareWithSaved; selectForCompare; compareFiles; compareSelected; workbench.action.closeActiveEditor; workbench.action.closeOtherEditors; workbench.action.closeUnmodifiedEditors; workbench.action.closeEditorsInGroup
- Menus: OpenEditorsContext
- Keys: none found
- Settings: files.autoSave
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\electron-browser\fileActions.contribution.ts (Reveal entry); V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (onListContextMenu); C bundle (OpenEditorsContext items in groups 1_open, 2_save, 3_compare, 4_close, and the ids above)

### SB-2.33 Sort the Open Editors list
- Tag: both
- Evidence: source
- Boundary: no
- What: List editors in tab order (default), alphabetically by tab name inside each group, or alphabetically by full path. Dragging rows to reorder is switched off unless the order is tab order.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.openEditors.sortOrder
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (getElements, OpenEditorsDragAndDrop); V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (explorer.openEditors.sortOrder)

### SB-2.34 Unsaved count badge in Open Editors
- Tag: both
- Evidence: source
- Boundary: no
- What: A badge in the Open Editors title shows how many files have unsaved changes (for example 2 unsaved) and hides itself when there are none.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.autoSave
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (updateDirtyIndicator); C bundle (string {0} unsaved)

### SB-2.35 Reorder editors by dragging in Open Editors
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag rows to reorder editors or move them into another editor group (group headings are drop targets), drop files from the operating system to open them, and drag a row out to the editor area. Reordering works only when the list is in tab order.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.openEditors.sortOrder
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\openEditorsView.ts (OpenEditorsDragAndDrop); C bundle (the same drag over logic: sort order check for editorOrder, feedback index, native file drop check)

### SB-2.36 No Folder Opened welcome
- Tag: both
- Evidence: source
- Boundary: no
- What: With no folder or workspace open, the Explorer shows a No Folder Opened view with welcome text and buttons: Open Folder with Open Recent, or an add-a-folder link when a workspace file is open, or a remote variant. Five wordings are chosen by state, and when editors are open the text warns that opening a folder closes them and offers adding a folder instead. Dropping a folder or file onto the view opens it.
- Commands: workbench.action.files.openFolder; workbench.action.files.openFolderViaWorkspace; workbench.action.addRootFolder; workbench.action.openRecent
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts (registerViewWelcomeContent for EmptyView.ID); V\src\vs\workbench\contrib\files\browser\views\emptyView.ts (drop handler); V\src\vs\workbench\browser\actions\workspaceActions.ts; C bundle (view id workbench.explorer.emptyView, title No Folder Opened, strings You have not yet opened a folder and Opening a folder will close all currently open editors, and the four command ids)

### SB-2.37 Clone Repository text in the empty Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: When Git is on and no source control provider is registered, the empty Explorer adds the line "You can clone a repository locally" with a Clone Repository button, then a Learn more line about Git and source control, below the Open Folder text. The wording is the same in Cursor.
- Commands: git.clone
- Menus: none found
- Keys: none found
- Settings: git.enabled
- Cursor: present
- Source: V\extensions\git\package.json (viewsWelcome entries for view explorer); V\extensions\git\package.nls.json; C\extensions\git\package.json (same two entries); C\extensions\git\package.nls.json (same text); C bundle (git.clone)

### SB-2.38 Open a file in the app's browser
- Tag: both
- Evidence: source
- Boundary: yes
- What: Right-click a local file and pick Open in Browser (Cursor, for html, svg, pdf, text, Markdown, JSON, XML, YAML, CSV, log, CSS, script and image files) or Open in Integrated Browser (stock, for .html and .htm only) to open it in a browser tab inside the app. Boundary yes: it opens the Cursor Browser tab, which the catalog treats as agent-driven (ED-10).
- Commands: workbench.action.browser.openFile; filesExplorer.openFileInBrowser; workbench.action.openBrowserEditor
- Menus: ExplorerContext; EditorTitleContext; EditorTitle
- Keys: none found
- Settings: none found
- Cursor: changed (own id filesExplorer.openFileInBrowser, wider file list, Explorer menu only)
- Source: V\src\vs\workbench\contrib\browserView\electron-browser\features\browserTabManagementFeatures.ts (OpenFileInIntegratedBrowserAction); V\src\vs\platform\browserView\common\browserView.ts (OpenFile id); C bundle (filesExplorer.openFileInBrowser, NLS 7047 Open in Browser, the extension regex, the call to workbench.action.openBrowserEditor with trackingId context_menu_new_tab)

### SB-2.39 Cursor Blame for a file
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: Right-click a file and pick Cursor Blame to open that file's blame view (command title Open File Blame). The entry is not shown for folders or workspace roots, and it appears only when the cursor_blame feature gate and a team admin setting are both on. Boundary yes: the blame data comes from Cursor conversation tracking, not from Git alone.
- Commands: cursorBlame.openFileBlame
- Menus: ExplorerContext; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (cursorBlame.openFileBlame, NLS 5375 Open File Blame, NLS 5376 Cursor Blame, menu label NLS 5384, ExplorerContext group 12_cursor_blame order 1, gate cursor_blame with isCursorBlameEnabledForTeam, classifyConversations)

### SB-2.40 Open in Images Preview
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Right-click a folder, or an image or video file, and pick Open in Images Preview to browse the images of that folder in the Images Preview viewer. It is marked experimental in stock and on by default. Cursor has none of its ids, setting or label. The viewer itself is ED-10.
- Commands: workbench.action.openImagesInCarousel
- Menus: ExplorerContext
- Keys: none found
- Settings: imageCarousel.explorerContextMenu.enabled
- Cursor: missing
- Source: V\src\vs\workbench\contrib\imageCarousel\browser\imageCarousel.contribution.ts; V\src\vs\workbench\workbench.common.main.ts; C bundle (searched the command id, the setting id, the label Open in Images Preview and the module name imageCarousel, 0 hits each)

### SB-2.41 Select several files and folders at once
- Tag: both
- Evidence: source
- Boundary: no
- What: Hold Ctrl (Cmd on macOS) and click to add or remove one row, hold Shift and click to select a range, or press Shift+Up and Shift+Down to extend the selection from the keyboard. Ctrl+A selects the visible rows under the focused row's parent folder, Ctrl+Shift+Enter adds or removes the focused row, and Esc clears the selection. Delete, Cut, Copy, Compare Selected, Open to the Side and dragging then act on every selected row, while Rename and the New File entries need one row. A setting swaps the click modifier to Alt (Option on macOS), and its text says the Open to the Side mouse gestures adapt so they do not clash.
- Commands: list.expandSelectionDown; list.expandSelectionUp; list.selectAll; list.toggleSelection; list.clear
- Menus: none found
- Keys: Shift+Down (list.expandSelectionDown, list supports multi-select); Shift+Up (list.expandSelectionUp); Ctrl+A (list.selectAll, list supports multi-select); Ctrl+Shift+Enter (list.toggleSelection); Escape (list.clear, when something is selected or focused)
- Settings: workbench.list.multiSelectModifier
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (multipleSelectionSupport true, getContext respects multi selection); V\src\vs\workbench\browser\actions\listCommands.ts (list.expandSelectionDown, list.expandSelectionUp, list.selectAll, list.toggleSelection, list.clear); V\src\vs\platform\list\browser\listService.ts (multiSelectModifier setting and the Alt switch); C bundle (all five list command ids and workbench.list.multiSelectModifier with the same description text, NLS 2127)

### SB-2.42 Scrolling options for the tree
- Tag: both
- Evidence: source
- Boundary: no
- What: Five settings shape how the tree scrolls: sideways scrolling for long names (off by default, and the Explorer switches it off while a name box is open), smooth scrolling (off), clicking the scrollbar to scroll a page at a time (off; with it on, double-clicking blank space no longer makes a new file), the mouse wheel speed (1) and the Alt-held fast scroll speed (5). Ctrl+Up and Ctrl+Down nudge the view by 10 pixels without moving the focused row (the key only fires when the view is not already at the top or bottom). Scroll Left and Scroll Right have no default keys. The settings apply to every tree and list, and SB-1.23 also names the sideways scrolling setting.
- Commands: list.scrollUp; list.scrollDown; list.scrollLeft; list.scrollRight
- Menus: none found
- Keys: Ctrl+Up (list.scrollUp, list focused and not scrolled to the top); Ctrl+Down (list.scrollDown, list focused and not scrolled to the bottom)
- Settings: workbench.list.horizontalScrolling; workbench.list.smoothScrolling; workbench.list.scrollByPage; workbench.list.mouseWheelScrollSensitivity; workbench.list.fastScrollSensitivity
- Cursor: present
- Source: V\src\vs\platform\list\browser\listService.ts (the five setting definitions); V\src\vs\workbench\browser\actions\listCommands.ts (list.scrollUp, list.scrollDown, list.scrollLeft, list.scrollRight); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (setEditable turns horizontalScrolling off, onMouseDblClick checks scrollByPage); C bundle (the five setting ids, list.scrollUp with the same top-of-list condition, list.scrollDown, list.scrollLeft, list.scrollRight)

### SB-2.43 Hover a row for its path and status
- Tag: both
- Evidence: source
- Boundary: no
- What: Rest the pointer on a row in the tree to see a tooltip with the full path, followed by a dot and the status text from any decoration on it (for example a Git state or Symbolic Link). Press Ctrl+K Ctrl+I with a row focused to show the same tooltip from the keyboard. A setting sets the delay before it appears.
- Commands: list.showHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I (list.showHover, list focused)
- Settings: workbench.hover.delay
- Cursor: present
- Source: V\src\vs\workbench\browser\labels.ts (title built from the path label, decoration tooltip appended after a dot); V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FilesRenderer.renderStat passes the decorations and no title); V\src\vs\workbench\browser\actions\listCommands.ts (list.showHover); V\src\vs\workbench\browser\workbench.contribution.ts (workbench.hover.delay); C bundle (list.showHover, workbench.hover.delay). Nothing was run on screen, so the tooltip wording comes from source.

### SB-2.44 Folders header shows the workspace name
- Tag: both
- Evidence: source
- Boundary: no
- What: The header of the Folders pane reads the opened folder's name, or the workspace file's name followed by (Workspace), or Untitled (Workspace) for an unsaved workspace, and a remote suffix when the folder is remote. It updates when the workspace is renamed, and a screen reader hears it as Explorer Section followed by the name. Dragging something over the header while the pane is folded opens it after a short wait. When Folders is the only pane showing, the container title carries it instead (SB-1.20).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (name getter, renderHeader with setHeader and DelayedDragHandler); V\src\vs\workbench\services\label\common\labelService.ts (getWorkspaceLabel, Untitled (Workspace), {0} (Workspace)); C bundle (the same renderHeader code with NLS 7353 Explorer Section: {0} and the drag handler; NLS 2751 and 2752 for the two workspace labels)

### SB-2.45 The tree remembers which folders were open
- Tag: both
- Evidence: source
- Boundary: no
- What: Close the window and open the same workspace again and the Explorer shows the same folders unfolded, with the same row selected and the same scroll position (auto reveal of the active file, SB-2.14, can then move it). In a multi-root workspace with no saved state, up to five roots open on their own, and a root you add opens when it appears. There is no setting and no button for it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (TREE_VIEW_STATE_STORAGE_KEY workbench.explorer.treeViewState, storeTreeViewState, setTreeInput view state and root expansion); C bundle (TREE_VIEW_STATE_STORAGE_KEY with the same key, storeTreeViewState, getViewState)

### SB-2.46 The tree follows changes on disk
- Tag: both
- Evidence: source
- Boundary: no
- What: Files that are created, deleted or renamed outside the app show up in the tree on their own, after a short delay, and the tree also refreshes when the window gets focus again. It holds off while a name box is open. Every workspace folder is watched except the paths the exclude setting names, and an include setting adds extra paths such as symbolic-link folders. Changing either setting restarts watching. If the system cannot watch, or the watcher dies, a warning notification appears (NT-2 is the closest owner for the notification itself).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.watcherExclude; files.watcherInclude
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\explorerService.ts (onDidFilesChange with the reaction delay, refresh on window focus, no refresh while editable); V\src\vs\workbench\contrib\files\browser\workspaceWatcher.ts (watchWorkspace, setting listeners, the two warning prompts); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (the two settings and the default exclude list); C bundle (files.watcherExclude, files.watcherInclude, NLS 7384 and 7386 for the two warnings)

### SB-2.47 Paste text to make a new file
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: With a row focused in the tree and plain text on the clipboard, Ctrl+V makes a new file named Untitled (Untitled 1, Untitled 2 and so on when the name is taken) holding that text, in the focused folder, the folder of the focused file, or the first root when nothing is focused. The file opens as a pinned tab without taking focus, and its name goes straight into the rename box so you can type the real name. It does nothing when the clipboard is blank or holds files (those paste as in SB-2.9), when a name box is already open, or when the focused row is read-only, and a read-only folder gives the error "Parent folder is readonly." Stock VS Code has no such command.
- Commands: explorer.createFileFromPaste
- Menus: none found
- Keys: Ctrl+V (browser paste event while the tree has focus, no command key rule)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (command registration explorer.createFileFromPaste and its handler writing the Untitled file, the paste listener on the Explorer view that calls it when clipboardData has text and no files, NLS 7135 Parent folder is readonly.); V\src\vs\workbench\contrib\files\browser\views\explorerView.ts (stock paste listener only forwards files to filesExplorer.paste; 0 hits for createFileFromPaste in V and in N)

### SB-2.48 Drag an editor tab into the tree to copy its file
- Tag: both
- Evidence: source
- Boundary: no
- What: Drag an editor tab onto a folder in the tree to copy that file into the folder (the tree treats the drop as coming from outside, so it copies instead of moving). A name clash asks before it replaces (DL-4 owns the box). The drop is refused when the folder is read-only or when drag and drop is turned off. Rows dragged from the Open Editors list are refused, because the tree turns away drags that start in another list. The reverse direction, tree rows to the editor area, is SB-2.12. Read from the accepted drag types in source, not run.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.enableDragAndDrop
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts (FileDragAndDrop.handleDragOver accepts Files, code files and ResourceURLs data, drop calls ExternalFileImport); V\src\vs\workbench\contrib\files\browser\fileImportExport.ts (ExternalFileImport.doImport uses extractEditorsAndFilesDropData, importResources); V\src\vs\platform\dnd\browser\dnd.ts (extractEditorsDropData reads editor and resource drag data); V\src\vs\workbench\browser\dnd.ts (fillEditorsDragData sets ResourceURLs and editors data); V\src\vs\base\browser\ui\list\listView.ts (drag data from another list becomes ExternalElementsDragAndDropData, drag data from outside any list becomes NativeDragAndDropData); C bundle (handleDragOver with the same accepted types FILES, code files and RESOURCES)

### SB-2.49 Run or debug tests under a file or folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a file or folder and pick Run Tests, Debug Tests or Run Tests with Coverage to run every test the test providers found under it; results show in the Test Explorer (SB-6). Each entry appears only while some test provider offers that kind of run, so a plain install with no test extension shows none of them. When nothing is found under the item, an info notification says "No tests found in the selected file or folder". SB-6 handed these three entries to this group, and the first pass here had handed them to SB-6, so they are listed here.
- Commands: testing.run.uri; testing.debug.uri; testing.coverage.uri
- Menus: ExplorerContext (group 6.5_testing; each entry needs the matching run capability context key)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts (ExecuteTestsUnderUriAction, RunTestsUnderUri, DebugTestsUnderUri, CoverageTestsUnderUri, labels at lines 85 to 87); V\src\vs\workbench\contrib\testing\common\constants.ts (the three ids); C bundle (testing.run.uri, testing.debug.uri, testing.coverage.uri, ExplorerContext group 6.5_testing with the capability key, NLS 11059 and 11066 to 11068)

## Handed off

- Find in Folder... and Find in Workspace... in the Explorer menu (filesExplorer.findInFolder, filesExplorer.findInWorkspace, Shift+Alt+F on a folder): SB-3.
- Open Timeline on a file (files.openTimeline): SB-7.
- Run Tests and Debug Tests on a folder or file (testing entries in ExplorerContext, group 6.5_testing): SB-6.
- Open Folder Settings on a root folder (_workbench.action.openFolderSettings): ED-8.
- Open Preview on Markdown files (markdown.showPreview in explorer/context): ED-10.
- Find All File References on Markdown and TypeScript files (markdown.findAllFileReferences, typescript.findAllFileReferences in explorer/context): SB-7.
- Compare Active File With..., Compare Active File with Clipboard (Ctrl+K C), Compare Active File with Saved (Ctrl+K D) and Compare New Untitled Text Files, all palette commands: ED-6.
- The unsaved file count badge on the Explorer icon (DirtyFilesIndicator, workbench.contrib.dirtyFilesIndicator): AB-3.
- The Share submenu in the Explorer and Open Editors menus (ExplorerContextShare, OpenEditorsContextShare, workbench.action.share, setting workbench.experimental.share.enabled, off by default): TB-1, since Share is also a menu bar entry.
- The Cursor Blame submenu in the editor right-click menu (Toggle file blame, View commit; cursorBlame.openCommit, cursorBlame.viewCommit): ED-6 is the closest group, but the catalog names no group for Cursor Blame, so please confirm.
- File menu entries Save, Save As, Revert File and Close Editor that also work while the Explorer has focus: TB-1.
- The confirm boxes for delete, drag and drop move, native paste, replace and undo: DL-4.
- Open Active Editor in New Empty Workspace (workbench.action.files.showOpenedFileInNewWindow, File category, palette only): ED-1, which owns new windows. Found in the files folder during the overflow check and not named by any group.
- The two file watcher warnings (Unable to watch for file changes, File changes watcher stopped unexpectedly, from workspaceWatcher.ts): NT-2 is the closest group, since no group owns individual notifications. SB-2.46 mentions them only as the visible result.

## Unclear

- Skipped as out of scope on your instruction: the four chat items in the file menu, Add File to Cursor Chat, Add Directory to Cursor Chat, Add File to New Cursor Chat and Add Directory to New Cursor Chat (composer.addfilestocomposer and composer.addfilestonnewcomposer, menu group 3_composer).
- Left out as extension system or outside the built-in list: the Install Extension VSIX entry shown for .vsix files, and the npm extension's Run NPM Script entry and NPM Scripts pane (npm.runScriptFromFolder, hidden by default).
- Read-only protection (files.readonlyInclude, files.readonlyExclude, files.readonlyFromPermissions; context keys explorerResourceReadonly and explorerResourceParentReadonly) turns off rename, cut, paste, drop and new file in the tree, and Cursor has the settings. Not listed as its own feature because it is a rule on other features, not a thing a person does.
- The file icon theme (workbench.iconTheme, picker workbench.action.selectIconTheme) sets the tree icons and can hide the arrows. No catalog group owns the picker.
- Whether Open With... (SB-2.5) shows for every file or only for files with more than one editor was not settled from source: the menu test is only that the available editors context key is not empty.
- An opened folder with no children: no empty-state message was found in the Explorer source, only an empty tree where a double-click on blank space creates a file (SB-2.6). Nothing was run to confirm how it looks.
- Newer than 1.128 and not in Cursor: in N the unsaved badge in Open Editors is a button that opens the first unsaved editor (hover text Open Unsaved Editor, 0 hits in the Cursor bundle), and the Explorer title keeps the casing of a user-chosen workspace name. Neither is listed as a feature.
- Cursor kept the stock defaults for the 26 explorer.* settings (the nesting pattern list was compared only at its first entries), files.exclude, and the workbench.tree sticky scroll, indent, guide and expand settings, checked by reading the setting definitions in the bundle. Nothing was run on screen, so row looks, hover text and menu order come from source and strings only.
- The Cursor Explorer menu was read from the bundle as appendMenuItem calls on the ExplorerContext menu. No explorer/context entries were found in the Cursor built-in extension manifests other than the markdown, npm and typescript ones that stock also has.
- Overflow check, hand-off clash: the first pass handed the Explorer Run Tests, Debug Tests and Run Tests with Coverage entries to SB-6, and the SB-6 file handed them back to SB-2 and does not list them, so SB-2.49 lists them. The Handed off line above for them is out of date.
- Overflow check, shared list behavior: SB-2.41 (multi-select) and SB-2.42 (tree scrolling options) describe workbench-wide list behavior seen through the Explorer. The catalog's unowned list names multi-select for the future workbench-wide section, and SB-1.23 already names two of the settings, so pass 4 may want to merge or move them. Cursor has all the ids and settings, found by searching the bundle.
- Overflow check, not run: SB-2.48 (tab drag into the tree) is read from the accepted drag types in source and the matching bundle code, with no run. Stock has no setting that turns off only that drop, only explorer.enableDragAndDrop for all drops.
- Overflow check, left out on purpose: the busy bar on the Explorer while it loads (ProgressLocation.Explorer with an 800 ms delay), the F2 key inside the rename box that cycles the selection between the name, the whole text and the extension (part of SB-2.7), the Directory contains N matches badge on folders during tree find (part of SB-2.26), the Open Editors row text and decorations (part of SB-2.30), and the Add Folder or Copy Folder box when a folder from the OS is dropped on a root (already in DL-4).
- Overflow check on 2026-10-25 added 9 features (SB-2.41 to SB-2.49).
