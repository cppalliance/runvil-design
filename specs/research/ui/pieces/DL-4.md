Group: DL-4 Delete, move and replace confirmations. Pass: 3. Date: 2026-10-25.

### DL-4.1
1. Pressing Delete on an Explorer item (Cmd+Backspace or Delete on macOS) asks "Are you sure you want to delete '{0}'?" with a "Move to Recycle Bin" button on Windows and a "Move to Trash" button on other systems.
2. Choosing Delete in the Explorer context menu shows the same box.
3. The box adds "You can restore this file from the Recycle Bin." (or "from the Trash"), and a folder or several selected items change the wording to name the folder and its contents or to list the files.
4. Ticking the "Do not ask me again" checkbox and confirming sets `explorer.confirmDelete` to false.
5. Setting `explorer.confirmDelete` set to false deletes without this box, and it is on by default.
6. Setting `files.enableTrash` chooses whether Delete moves items to the OS trash (on by default) or deletes them for good (off), and off shows the permanent delete box instead of this one.

### DL-4.2
1. Pressing Shift+Delete on an Explorer item (Cmd+Alt+Backspace on macOS) asks "Are you sure you want to permanently delete '{0}'?" with a Delete button.
2. Holding Alt over the Delete entry in the Explorer context menu turns it into "Delete Permanently", which shows the same box.
3. When an item cannot be moved to the trash, pressing Delete shows this box and the context menu entry reads "Delete Permanently".
4. For a folder the box adds "This action is irreversible!", and for a file it adds a note that the file can be restored with the Undo command.
5. This box has no "Do not ask me again" checkbox.
6. Setting `explorer.confirmDelete` set to false also skips this box. [stock only]
7. Setting `explorer.confirmDelete` set to false skips only the Trash box, and this box still asks every time. [cursor only]

### DL-4.3
1. When moving to the Recycle Bin or Trash fails, a warning box asks "Failed to delete using the Recycle Bin. Do you want to permanently delete instead?" (reading "Trash" on macOS and Linux) with a "Delete Permanently" button.
2. Pressing "Delete Permanently" deletes the items for good with no further question, and cancelling leaves them in place.
3. When a permanent delete fails, a warning box shows the error message with a "Retry" button.
4. Pressing "Retry" tries the delete again with no further question.
5. With `files.enableTrash` off every delete is permanent, so only the "Retry" box can show.

### DL-4.4
1. Deleting a file with unsaved changes shows a warning "You are deleting {0} with unsaved changes. Do you want to continue?" with the detail "Your changes will be lost if you don't save them."
2. For a folder the warning reads "You are deleting a folder {0} with unsaved changes in {1} files. Do you want to continue?" (or "in 1 file"), and for several items it reads "You are deleting files with unsaved changes. Do you want to continue?"
3. The confirm button reads the same as in the regular delete box, "Move to Recycle Bin" or "Move to Trash", or "Delete" for a permanent delete.
4. The box still shows when `explorer.confirmDelete` is false.
5. Confirming this box skips the regular delete box, so the person is not asked twice.

### DL-4.5
1. Deleting a file marked read-only shows a warning "You are deleting a file {0} that is configured to be read-only. Do you want to continue?" with the detail "The read-only protection will be overridden if you continue." and a "Continue" button.
2. For a folder the warning says "You are deleting a folder {0} that is configured to be read-only...", and for several items it says "You are deleting files that are configured to be read-only..."
3. The box still shows when `explorer.confirmDelete` is false.
4. Settings `files.readonlyInclude` and `files.readonlyExclude` take paths or glob patterns that mark files as read-only and take some of them back out.
5. Setting `files.readonlyFromPermissions` (off by default) also marks files read-only when their file permissions say so.

### DL-4.6
1. Dragging an item onto another folder in the Explorer asks "Are you sure you want to move '{0}' into '{1}'?" with a "Move" button.
2. Dragging several items asks "Are you sure you want to move the following {0} files into '{1}'?" and lists their names.
3. Holding Ctrl (Alt on macOS) while dropping copies the items instead and skips this box.
4. Ticking "Do not ask me again" and confirming sets `explorer.confirmDragAndDrop` to false.
5. Setting `explorer.confirmDragAndDrop` set to false moves dropped items with no box, and it is on by default.
6. Setting `explorer.enableDragAndDrop` chooses whether items can be dragged to move them inside the Explorer at all, and it is on by default.

### DL-4.7
1. Dragging a root folder to a new place in a multi-root workspace asks "Are you sure you want to change the order of root folder '{0}' in your workspace?" with a "Move" button.
2. Dragging several root folders at once asks "Are you sure you want to change the order of multiple root folders in your workspace?"
3. Ticking "Do not ask me again" and confirming sets `explorer.confirmDragAndDrop` to false.
4. With `explorer.confirmDragAndDrop` false the root folders change order with no box.

### DL-4.8
1. When a drag move would land on a name that already exists, a box asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a "Replace" button.
2. The box adds "This action is irreversible!"
3. When several names clash, the box asks "The following {0} files and/or folders already exist in the destination folder. Do you want to replace them?" and lists the names.
4. Pressing "Replace" moves the items over the existing ones, and cancelling leaves the existing items in place.
5. The box has no "Do not ask me again" checkbox and no setting hides it.

### DL-4.9
1. With `explorer.incrementalNaming` set to `disabled`, pasting an Explorer item whose name already exists in the target folder asks "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a "Replace" button.
2. With the same setting, dropping a Ctrl-drag copy (Alt on macOS) onto an existing name asks the same question.
3. Setting `explorer.incrementalNaming` chooses how a pasted duplicate is named, with values `simple` (adds the word "copy", the default and no box), `smart` (adds a number or raises the one in the name) and `disabled` (keeps the name and asks to replace).
4. The Explorer context menu entries Copy, Cut and Paste, and the keys Ctrl+C, Ctrl+X and Ctrl+V (Cmd on macOS) with the Explorer focused, start the copy, cut and paste.

### DL-4.10
1. Dropping files from the OS file manager onto the Explorer asks, for each name that clashes, "A file or folder with the name '{0}' already exists in the destination folder. Do you want to replace it?" with a "Replace" button.
2. The box adds "This action is irreversible!" and has no setting that turns it off.
3. Setting `explorer.autoOpenDroppedFile` chooses whether a file dropped into the Explorer opens in the editor, and it is on by default.

### DL-4.11
1. Dropping folders from the OS onto a workspace root folder asks "Do you want to copy the folders or add the folders to the workspace?" (for one folder "Do you want to copy '{0}' or add '{0}' as a folder to the workspace?") with "Add Folders to Workspace", "Copy Folders" and Cancel buttons.
2. Pressing "Add Folders to Workspace" adds the dropped folders as workspace folders.
3. Pressing "Copy Folders" copies the dropped folders into the Explorer.
4. When no workspace folder shares the dropped folder's file system, the box asks only "Are you sure to want to copy folders?" (for one folder "Are you sure to want to copy '{0}'?") with "Copy Folders" and Cancel, and no Add button.
5. Dropping folders onto an Explorer item that is not a workspace root copies them in with no box.

### DL-4.12
1. Pasting files copied in the OS file manager into the Explorer asks "Are you sure you want to paste '{0}'?" with a "Paste" button.
2. Pasting several items asks "Are you sure you want to paste the following {0} items?" and lists their names.
3. Ticking "Do not ask me again" and confirming sets `explorer.confirmPasteNative` to false.
4. Setting `explorer.confirmPasteNative` set to false pastes with no box, and it is on by default.
5. Pressing Ctrl+V or Cmd+V with the Explorer focused, or choosing Paste in the Explorer context menu, starts the paste.

### DL-4.13
1. With the Explorer focused, pressing Ctrl+Z (Cmd+Z on macOS) or choosing Edit > Undo asks "Would you like to undo '{0}'?" with Yes and No buttons when the last file step deleted something, such as a paste, a copy or a new file.
2. Pressing Yes undoes the step and pressing No leaves it in place.
3. Setting `explorer.confirmUndo` chooses which undo steps ask, with values `verbose` (also rename and move), `default` (steps that delete something) and `light` (no box except when undoing a new file).
4. With the Explorer focused, Ctrl+Y or Ctrl+Shift+Z (Cmd+Shift+Z on macOS) redoes the undone file step.
5. Setting `explorer.enableUndo` chooses whether the Explorer supports undoing file and folder steps at all, and it is on by default.

### DL-4.14
1. Saving an untitled tab that has its own file path with Ctrl+S, after a file was created at that path in the meantime, shows a warning "'{0}' already exists. Do you want to replace it?" with a "Replace" button.
2. The box adds "A file or folder with the name '{0}' already exists in the folder '{1}'. Replacing it will overwrite its current contents."
3. Pressing "Replace" saves over the existing file, and cancelling saves nothing.
4. A normal Save As that picks an existing name is answered by the OS Save box's own replace question and does not show this box.

### DL-4.15
1. Opening a file over the size limit shows, inside the editor pane, a warning "The file is not displayed in the text editor because it is very large ({0})." with the file size in place of {0}.
2. Pressing "Open Anyway" opens the file in the editor with no size limit.
3. Pressing "Configure Limit" opens User Settings filtered to `workbench.editorLargeFileConfirmation`.
4. Setting `workbench.editorLargeFileConfirmation` sets the size in MB where the message shows, with a minimum of 1 and defaults of 1 GB for local files, 10 MB for remote files and 50 MB on the web.
5. When the size is not known the message reads "The file is not displayed in the text editor because it is very large." with no size.

## Unclear

- DL-4.15 is an editor pane message and not a modal box. The feature file already says another group may fit it better.
- DL-4.9: a cut and paste onto an existing name has no Replace piece, because the feature file says V shows no Replace box for it unless `explorer.incrementalNaming` is `disabled`, and I did not run it to see what shows.
- DL-4.13: the feature file says `light` skips the box "except for new files". I took that from the code, where a new file always asks before undo, and the setting text says `light` does not prompt, so piece 3 follows the code.