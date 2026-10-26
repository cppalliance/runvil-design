Group: DL-4 Delete, move and replace confirmations. Section: Dialogs. Date: 2026-10-25. Pass: 2.
Features: 15.

Short names used below: V = stock VS Code 1.128.0 source, N = VS Code main 1.144.0 (only used to spot newer things), C bundle = the Cursor 3.27.12 main bundle `workbench.desktop.main.js` (minified, so I cite message numbers from `out\nls.messages.json`, setting ids and command ids). V paths are written in full with the `V\` prefix. The Explorer boxes all go through the shared confirm and prompt boxes of DL-1, so the buttons and the "Do not ask me again" checkbox look the same as any other confirm.

### DL-4.1 Confirm moving a file to the Recycle Bin or Trash
- Tag: both
- Evidence: source
- Boundary: no
- What: Deleting in the Explorer asks "Are you sure you want to delete '{0}'?" with a "Move to Recycle Bin" (Windows) or "Move to Trash" button, a note on how to restore, and a "Do not ask me again" checkbox that turns `explorer.confirmDelete` off.
- Commands: moveFileToTrash
- Menus: ExplorerContext
- Keys: Delete (when the Explorer is focused and the item can go to trash); Cmd+Backspace or Delete (macOS, same condition)
- Settings: explorer.confirmDelete; files.enableTrash
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (deleteFiles function, moveFileToTrash, messages 7080, 7081, 7096 to 7100, 7109 to 7113, explorer.confirmDelete)

### DL-4.2 Confirm deleting permanently
- Tag: both
- Evidence: source
- Boundary: no
- What: Shift+Delete, the Alt version of the Delete menu item, or Delete when there is no trash asks "Are you sure you want to permanently delete '{0}'?" with a Delete button and "This action is irreversible!" for folders, and this box has no "Do not ask me again" checkbox.
- Commands: deleteFile
- Menus: ExplorerContext
- Keys: Shift+Delete; Cmd+Alt+Backspace (macOS); Delete (when the item cannot go to trash)
- Settings: explorer.confirmDelete; files.enableTrash
- Cursor: changed (in Cursor `explorer.confirmDelete` set to false skips only the Trash box and this box always asks, in stock 1.128.0 the setting skips both)
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (deleteFiles function condition `useTrash && explorer.confirmDelete === false`, messages 7082, 7093 to 7095, 7114 to 7118, setting message 7292)

### DL-4.3 Retry after a failed delete
- Tag: both
- Evidence: source
- Boundary: no
- What: When the Recycle Bin or Trash fails the box asks "Failed to delete using the Recycle Bin. Do you want to permanently delete instead?" with a Delete Permanently button, and when a permanent delete fails it shows the error with a Retry button.
- Commands: moveFileToTrash; deleteFile
- Menus: none found
- Keys: none found
- Settings: files.enableTrash
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; C bundle (deleteFiles catch branch, messages 7105 to 7108)

### DL-4.4 Warn when deleting files with unsaved changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Deleting a file or folder that has unsaved edits asks "You are deleting {0} with unsaved changes. Do you want to continue?" with "Your changes will be lost if you don't save them.", and it asks even when `explorer.confirmDelete` is off.
- Commands: moveFileToTrash; deleteFile
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; C bundle (messages 7083 to 7087)

### DL-4.5 Warn when deleting read-only files
- Tag: both
- Evidence: source
- Boundary: no
- What: Deleting a file or folder marked read-only asks "You are deleting a file {0} that is configured to be read-only. Do you want to continue?" with a Continue button, and it asks even when `explorer.confirmDelete` is off.
- Commands: moveFileToTrash; deleteFile
- Menus: none found
- Keys: none found
- Settings: files.readonlyInclude; files.readonlyExclude; files.readonlyFromPermissions
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (messages 7088 to 7092, files.readonlyInclude)

### DL-4.6 Confirm moving files by drag and drop
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging items to another folder inside the Explorer asks "Are you sure you want to move '{0}' into '{1}'?" (or lists several files) with a Move button and a "Do not ask me again" checkbox, and holding Ctrl (Alt on macOS) copies instead and skips the box.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.confirmDragAndDrop; explorer.enableDragAndDrop
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (handleExplorerDrop, messages 7366, 7368, 7369, 7370, explorer.confirmDragAndDrop)

### DL-4.7 Confirm reordering workspace root folders
- Tag: both
- Evidence: source
- Boundary: no
- What: Dragging a root folder in a multi-root workspace asks "Are you sure you want to change the order of root folder '{0}' in your workspace?" (or the multiple-folders wording) with a Move button and the same "Do not ask me again" checkbox.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.confirmDragAndDrop
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; C bundle (handleExplorerDrop, messages 7365 and 7367)

### DL-4.8 Replace prompt when a move lands on an existing name
- Tag: both
- Evidence: source
- Boundary: no
- What: When a drag move would overwrite items, the box asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" (or lists "The following {0} files and/or folders already exist") with a Replace button and "This action is irreversible!", and no setting turns it off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\workbench\contrib\files\browser\fileImportExport.ts; C bundle (doHandleExplorerDropOnMove conflict branch, messages 7119, 7120, 7214 to 7219)

### DL-4.9 Replace prompt when copying or pasting with incremental naming off
- Tag: both
- Evidence: source
- Boundary: no
- What: Pasting or Ctrl-dragging a copy asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" only when `explorer.incrementalNaming` is set to disabled, and in the default setting the copy is renamed "name copy" with no box.
- Commands: filesExplorer.paste; filesExplorer.copy; filesExplorer.cut
- Menus: ExplorerContext
- Keys: Ctrl+V or Cmd+V (Explorer focused); Ctrl+C or Cmd+C (Explorer focused); Ctrl+X or Cmd+X (Explorer focused)
- Settings: explorer.incrementalNaming
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (findValidPasteFileTarget, askForOverwrite, filesExplorer.paste, explorer.incrementalNaming, messages 7316 and 7119)

### DL-4.10 Replace prompt when dropping files from the OS
- Tag: both
- Evidence: source
- Boundary: no
- What: Dropping files from the OS file manager onto the Explorer asks the same "already exists in the destination folder" replace question once for each name that clashes, and there is no setting for it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: explorer.autoOpenDroppedFile
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileImportExport.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; C bundle (ExternalFileImport class, messages 7119, 7201, 7202, explorer.autoOpenDroppedFile)

### DL-4.11 Choose copy or add to workspace when dropping folders
- Tag: both
- Evidence: source
- Boundary: no
- What: Dropping folders from the OS onto a workspace root asks "Do you want to copy the folders or add the folders to the workspace?" with Add Folders to Workspace, Copy Folders and Cancel, and it only offers Copy when no workspace folder shares the dropped folder's file system.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileImportExport.ts; C bundle (ExternalFileImport.doImport, messages 7193 to 7200)

### DL-4.12 Confirm pasting files from outside
- Tag: both
- Evidence: source
- Boundary: no
- What: Pasting files copied in the OS file manager into the Explorer asks "Are you sure you want to paste '{0}'?" (or "the following {0} items") with a Paste button and a "Do not ask me again" checkbox that turns `explorer.confirmPasteNative` off.
- Commands: filesExplorer.paste
- Menus: ExplorerContext
- Keys: Ctrl+V or Cmd+V (Explorer focused)
- Settings: explorer.confirmPasteNative
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\views\explorerView.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (pasteFileHandler, messages 7140 to 7143, explorer.confirmPasteNative)

### DL-4.13 Confirm before undoing a file operation
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing undo with the Explorer focused asks "Would you like to undo '{0}'?" with Yes and No for steps that delete something (a copy or a new file), and `explorer.confirmUndo` set to verbose also asks for rename and move while light skips the box except for new files.
- Commands: undo; redo
- Menus: MenubarEditMenu
- Keys: Ctrl+Z (Explorer focused); Ctrl+Y or Ctrl+Shift+Z (Explorer focused, redo); Cmd+Z and Cmd+Shift+Z (macOS)
- Settings: explorer.confirmUndo; explorer.enableUndo
- Cursor: present
- Source: V\src\vs\platform\undoRedo\common\undoRedoService.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; V\src\vs\workbench\contrib\files\browser\fileActions.ts; V\src\vs\workbench\contrib\files\browser\views\explorerViewer.ts; V\src\vs\editor\browser\editorExtensions.ts; C bundle (explorer.confirmUndo, explorer.enableUndo, messages 2647 to 2649, 7294 to 7297)

### DL-4.14 Replace prompt when an untitled file is saved over a file that now exists
- Tag: both
- Evidence: source
- Boundary: no
- What: Saving an untitled tab that already has a file path to that path, after a file was created there in the meantime, asks "'{0}' already exists. Do you want to replace it?" with a Replace button, while the OS Save box asks its own replace question for normal Save As.
- Commands: workbench.action.files.save
- Menus: none found
- Keys: Ctrl+S
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\textfile\browser\textFileService.ts; V\src\vs\workbench\services\workingCopy\common\fileWorkingCopyManager.ts; C bundle (messages 13118, 13119, 13120, 13555 to 13557)

### DL-4.15 Warn before opening a very large file
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening a file over the size limit (1 GB local, 10 MB remote, 50 MB web by default) shows "The file is not displayed in the text editor because it is very large ({0})." inside the editor with Open Anyway and Configure Limit buttons, and this is a message in the editor pane and not a pop-up.
- Commands: workbench.action.openLargeFile; workbench.action.configureEditorLargeFileConfirmation
- Menus: none found
- Keys: none found
- Settings: workbench.editorLargeFileConfirmation
- Cursor: present
- Source: V\src\vs\workbench\common\editor.ts; V\src\vs\workbench\common\editor\resourceEditorInput.ts; V\src\vs\workbench\contrib\files\browser\editors\textFileEditor.ts; V\src\vs\workbench\browser\parts\editor\editorConfiguration.ts; V\src\vs\platform\files\common\files.ts; C bundle (workbench.action.openLargeFile, workbench.editorLargeFileConfirmation, messages 7009, 7010, 4407)

## Handed off

- The inline name box check when creating or renaming ("A file or folder X already exists at this location. Please choose a different name.") is an inline error and not a confirm: SB-2
- "Would you like to undo '{0}' across all files?" with Undo in {0} Files and Undo this File, raised by multi-file refactors in `V\src\vs\platform\undoRedo\common\undoRedoService.ts`: ED-4
- The large-file notification "{0}: tokenization, wrapping, folding, codelens, word highlighting and sticky scroll have been turned off for this large file..." with a "Forcefully Enable Features" button (`V\src\vs\workbench\contrib\codeEditor\browser\largeFileOptimizations.ts`, setting `editor.largeFileOptimizations`): ED-5
- The browser-only Upload command and its replace prompt (`explorer.upload`, shown only when `IsWebContext` is true): none, web only and not in the desktop app

## Unclear

- I did not run Cursor or VS Code. How each box looks, its button order on each OS and the exact wording shown for mixed file and folder selections are read from code and message numbers only.
- DL-4.15 is not a modal box. It is an editor placeholder with two buttons. I listed it because the request named large file warnings, but an owner like ED-10 or ED-1 may fit better. It is your call.
- Pass 1 pointed `explorer.confirmPasteNative` at `fileImportExport.ts` lines 470 to 490. That range is the drop-folders prompt (DL-4.11), which has no setting. The setting only guards pasting native files (`fileActions.ts` `pasteFileHandler`, DL-4.12).
- Pass 1 said no setting turns off the replace box. `explorer.incrementalNaming` does for paste and Ctrl-drag copies (only the value disabled asks, DL-4.9). The drag move and OS drop replace boxes (DL-4.8 and DL-4.10) have no setting.
- Pass 1 said the Do not ask me again checkbox is on the delete box. It is only on the Recycle Bin or Trash box (DL-4.1). The permanent delete box (DL-4.2) has none.
- Cut then paste onto a name that already exists does not show a Replace box. V passes no overwrite for a move unless `explorer.incrementalNaming` is disabled, so the file service conflict surfaces as an error notification ("The file(s) to paste have been deleted or moved since you copied them..."). Read from code, not run.
- There is no setting named `files.confirmDelete` or `files.confirmDragAndDrop` in V or Cursor. The ids are `explorer.confirmDelete` and `explorer.confirmDragAndDrop`.
- N (1.144.0) has the same delete, move, replace, paste and undo confirms and the same settings. I found nothing newer for this group. The Cursor setting text for `explorer.confirmDelete` ("when deleting a file via the trash", message 7292) is older wording than V's, which matches the behavior difference in DL-4.2.
- DL-4.13 lists the Edit menu entries for undo and redo because `undo` and `redo` are shared commands. The Explorer-only behavior is the `explorer` implementation registered at priority 110.
