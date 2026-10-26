Group: DL-2 Unsaved changes, close and quit prompts. Pass: 3. Date: 2026-10-25.

### DL-2.1
1. Closing an editor that has unsaved changes brings it to the front and asks "Do you want to save the changes you made to {file}?" with the detail "Your changes will be lost if you don't save them."
2. The box has three buttons: Save, Don't Save and Cancel.
3. Choosing Save writes the file and closes the editor, and an untitled file goes to Save As first.
4. Choosing Don't Save reverts the editor and closes it, even if the revert fails.
5. Choosing Cancel keeps the editor open.
6. The box is reached by pressing Ctrl+W (Cmd+W on macOS, and Ctrl+F4 on Windows) for `workbench.action.closeActiveEditor`, by File > Close Editor, by the Close button on the tab, and by Close in the tab context menu.
7. Commands that close several editors one by one, such as Close Others (`workbench.action.closeOtherEditors`), Close to the Right (`workbench.action.closeEditorsToTheRight`) and Close Editors in Group (`workbench.action.closeEditorsInGroup`, Ctrl+K W), ask once per dirty editor in turn.
8. An editor with its own close prompt, such as the merge editor, shows that prompt instead of this box.
9. Setting `window.dialogStyle` chooses whether the box is an OS message box or the in-page box, with values `native` and `custom`.

### DL-2.2
1. When more than one file is at stake, the box asks "Do you want to save the changes to the following {n} files?" and lists the file names.
2. The list shows up to ten names and then "...n additional files not shown".
3. The first button reads "Save All", and Don't Save and Cancel stay as they are.
4. The box is raised by Close All Editors and Close All Editor Groups (DL-2.3), and by quitting or closing with unsaved files that could not be backed up (DL-2.5).
5. Setting `window.dialogStyle` chooses whether the box is an OS message box or the in-page box, with values `native` and `custom`.

### DL-2.3
1. Close All Editors (`workbench.action.closeAllEditors`, Ctrl+K Ctrl+W) and Close All Editor Groups (`workbench.action.closeAllGroups`, Ctrl+K Ctrl+Shift+W, both also in the Command Palette under View) check every open editor for unsaved changes before closing.
2. Dirty editors that auto-save on focus change or window change are saved quietly, under `files.autoSave`, without a box.
3. The remaining ordinary dirty editors are revealed so you can see them, and then one box asks about all of them, using the DL-2.2 wording when there are several.
4. Editors that have their own close prompt are asked about after the ordinary ones.
5. Choosing Cancel at any step stops the whole close.
6. Close All Editors skips pinned tabs, and Close All Editor Groups does not.
7. A later stock build (newer-than-1.128) also skips editors that cannot be closed and removes only empty groups. [stock only]

### DL-2.4
1. With `files.autoSave` set to `onFocusChange` (or `onWindowChange` on Windows and Linux), closing a dirty file saves it and does not show the box.
2. If that save fails, the normal Save, Don't Save and Cancel box appears.
3. Untitled files are never auto-saved, so closing one still shows the box.
4. At quit or window close, files under an auto save mode are saved first, and the box asks only about what remains.

### DL-2.5
1. When the app quits, a window closes or reloads, or the folder changes, and unsaved files could not be backed up, a box with Save All, Don't Save and Cancel lists them with the DL-2.2 wording.
2. The box is reached by File > Exit or Cmd+Q (`workbench.action.quit`), by File > Close Window (`workbench.action.closeWindow`, Cmd+Shift+W, Alt+F4 or Ctrl+Shift+W), by File > Close Folder or Close Workspace (`workbench.action.closeFolder`, Ctrl+K F), and by `workbench.action.reloadWindow` (Ctrl+R in development builds only).
3. Choosing Save All saves each file, and an untitled file opens Save As.
4. If any save fails or is cancelled, the quit or close is stopped.
5. Choosing Don't Save reverts the files and the quit or close goes ahead.
6. Choosing Cancel stops the quit or close.
7. With `files.hotExit` set to `off`, the box shows whenever a window closes or the app quits with unsaved files.
8. With hot exit on, the box still shows when a window closes while the app keeps running and when the folder or workspace changes, and the value `onExitAndWindowClose` backs up folder windows instead of asking.

### DL-2.6
1. With hot exit on, quitting does not ask about unsaved files, because the app remembers them, untitled files included.
2. Setting `files.hotExit` chooses when unsaved files are backed up, with values `off`, `onExit` (the default) and `onExitAndWindowClose`.
3. With `onExit`, unsaved files are backed up when you quit, or close the last window on Windows and Linux.
4. With `onExitAndWindowClose`, unsaved files are also backed up for any window that has a folder open, even when it is not the last window.
5. With `off`, nothing is backed up and the Save All, Don't Save, Cancel box in DL-2.5 comes back.

### DL-2.7
1. After a hot exit, the next start reopens windows as set by `window.restoreWindows`, with values `preserve`, `all` (the default), `folders`, `one` and `none`.
2. Each backed-up unsaved file returns as a modified tab with its unsaved text back.
3. The restored tabs open in the background, so they do not take focus.
4. Windows that had no folder return as empty windows holding those files.
5. Backups of a folder window stay on disk until that folder is opened again, and the folder shows a dirty marker in Open Recent (DL-2.14).

### DL-2.8
1. When backing up, saving, reverting or discarding backups at shutdown takes a while, a pending box with a spinner appears.
2. The box message reads "Backing up editors with unsaved changes is taking a bit longer...", "Saving editors...", "Reverting editors..." or "Discarding backups...", depending on the step.
3. The backup box adds "Click 'Cancel' to stop waiting and to save or revert editors with unsaved changes."
4. Pressing Cancel, or Escape, stops the wait and falls back to the save prompt.
5. While untitled files are being saved, the progress moves to the status bar instead of a box, because a Save As box may need the screen.

### DL-2.9
1. If the backup cannot be written, an error box says "The following editors with unsaved changes could not be saved to the backup location."
2. If the save or revert you chose fails, the box says "...could not be saved or reverted." instead.
3. The detail lists the files and adds "Try saving or reverting the editors with unsaved changes first and then try again."
4. Choosing OK keeps the window open.
5. The second button reads "Close Anyway", "Quit Anyway" or "Reload Anyway", depending on what you were doing, and goes ahead and loses the changes.
6. The box is skipped in an extension development window.

### DL-2.10
1. When `window.confirmBeforeClose` asks, quitting shows "Are you sure you want to exit?" on Windows and Linux or "Are you sure you want to quit?" on macOS, with an Exit or Quit button and Cancel.
2. When `window.confirmBeforeClose` asks, closing a window shows "Are you sure you want to close the window?" with a Close Window button and Cancel.
3. A "Do not ask me again" checkbox on the box switches `window.confirmBeforeClose` to `never`.
4. Setting `window.confirmBeforeClose` chooses when the box shows, with values `always`, `keyboardOnly` (only when a keybinding started the action) and `never`.
5. The desktop default is `never`, and a browser tab defaults to `keyboardOnly`.
6. Closing the last window on Windows and Linux counts as quit.
7. The File menu has a Confirm Before Close toggle (`workbench.action.toggleConfirmBeforeClose`) in the browser build only. [stock only]
8. In Cursor the Quit Cursor? and Close this window? boxes (DL-2.17, DL-2.18) come first, and the stock box still shows after them when the setting asks for it. [cursor only]

### DL-2.11
1. On macOS, choosing Quit in the app menu (or pressing Cmd+Q) shows an OS question box "Are you sure you want to quit?" with Quit and Cancel when the setting asks for it.
2. Setting `window.confirmBeforeClose` decides whether the box shows, so `always` shows it, `keyboardOnly` shows it only when a key was used, and `never` skips it.
3. The box only acts when a window has focus, is minimized, or no window is open.

### DL-2.12
1. Closing a floating editor window with only clean editors just closes it.
2. Every editor with unsaved changes, or with its own close prompt, moves back into the main window, so no save box appears.
3. If an editor cannot be moved back, the floating window stays open.
4. When `window.confirmBeforeClose` asks, the "close the window" box (DL-2.10) shows first.
5. If something vetoes the close, an error box shows the reason and adds "Try saving or reverting the editors with unsaved changes first and then try again."

### DL-2.13
1. Closing a window that holds an untitled multi-root workspace, or switching to another workspace, asks "Do you want to save your workspace configuration as a file?" with the detail "Save your workspace if you plan to open it again."
2. Choosing Save opens a Save As picker for a workspace file, and cancelling the picker keeps the window open.
3. Choosing Don't Save deletes the untitled workspace and the close goes ahead.
4. Choosing Cancel stops the close.
5. An "Always discard untitled workspaces without asking" checkbox on the box turns the prompt off.
6. With `window.confirmSaveUntitledWorkspace` off, the untitled workspace is deleted silently and no box shows.
7. The box is not shown when closing the last window on Windows and Linux, because that counts as quit.

### DL-2.14
1. In the Open Recent list, a folder or workspace that still holds unsaved backups gets a dirty-dot button with the tooltip "Folder With Unsaved Files" or "Workspace With Unsaved Files".
2. Clicking the dirty-dot button asks "Do you want to open the folder to review the unsaved files?" (or workspace) with the detail "Folders with unsaved files cannot be removed until all unsaved files have been saved or reverted."
3. Confirming opens that folder or workspace in a window.
4. Entries with unsaved backups cannot be removed from the list until the files are saved or reverted.
5. Entries that have backups but are not in the history are added to the list too.
6. The list is reached by File > Open Recent > More..., by Open Recent in the Command Palette (`workbench.action.openRecent`, Ctrl+R), and by `workbench.action.quickOpenRecent`.

### DL-2.15
1. If shutting down throws an unexpected error, an error box says "An unexpected error prevented the window to close" with the detail "Error: {message}".
2. The wording follows what you were doing, as "...the application to quit", "...the window to reload" or "...to change the workspace".

### DL-2.16
1. When Save As lands on a path that the read-only settings cover, a warning box says "'{file}' is marked as read-only. Do you want to save anyway?" with the detail "Paths can be configured as read-only via settings."
2. Choosing Save Anyway saves the file, and choosing Cancel does not.
3. Setting `files.readonlyInclude` lists paths or glob patterns to mark as read-only.
4. Setting `files.readonlyExclude` lists paths or glob patterns that are kept out of `files.readonlyInclude`.
5. Setting `files.readonlyFromPermissions` marks files as read-only when their file permissions say so.

### DL-2.17
1. The Quit command (File > Exit on Windows and Linux, Cmd+Q on macOS and Linux, `workbench.action.quit`) opens a box titled "Quit Cursor?" with "Are you sure you want to quit?", a Quit button and a Cancel button. [cursor only]
2. A "Don't ask again" checkbox sits in the button row. [cursor only]
3. Pressing Enter chooses Quit and pressing Escape chooses Cancel. [cursor only]
4. The box asks at the default setting, where stock never asks. [cursor only]
5. Ticking "Don't ask again" and choosing Quit means the box never asks again. [cursor only]

### DL-2.18
1. The Close Window command (`workbench.action.closeWindow`) closes the window at once when no folder or workspace is open. [cursor only]
2. When a folder or workspace is open, the command first opens a box titled "Close this window?" with "Are you sure you want to close this window?", a Close button and a Cancel button. [cursor only]
3. A "Don't ask again" checkbox on the box stops it from asking again. [cursor only]
4. Pressing Enter chooses Close and pressing Escape chooses Cancel. [cursor only]
5. The box asks at the default setting, where stock never asks. [cursor only]
6. The box is reached by File > Close Window, Cmd+Shift+W on macOS, Alt+F4 or Ctrl+Shift+W on Windows and Linux, and the Command Palette. [cursor only]
7. On macOS Cmd+W (`workbench.action.closeWindowOrEditorOnMac`) closes the active editor when there is one, which may ask DL-2.1, and otherwise runs Close Window. [cursor only]

### DL-2.19
1. If shutdown is still going after about 0.8 seconds, a progress message shows in the status bar: "Closing the window is taking a bit longer...", with variants for quitting the application, reloading the window and changing the workspace.
2. If it is still going after about 1.2 seconds, a box with the same title opens and lists "The following operations are still running:" with one line per unfinished step.
3. The box has one button, "Close Anyway", "Quit Anyway", "Reload Anyway" or "Change Anyway", that forces the shutdown; the box cannot be cancelled or dismissed and disappears when the shutdown finishes.
4. The step names that can appear are Saving UI state, Saving text files, Backup working copies, Saving working copies, Saving local history, Stopping Extension Hosts, Disconnect Remote Agent (remote windows, last in order) and Terminating extension debug session (extension development windows).
5. Cursor adds the steps "Cleaning up browser views" and "Flushing analytics events" to that list. [cursor only]
6. If shutdown fails with an unexpected error, an error box says "An unexpected error prevented the window to close" (with quit, reload and change workspace variants) and shows "Error: <message>" as detail.

### DL-2.20
1. Setting `debug.confirmOnExit` chooses whether the window asks before closing with live debug sessions, with values `never` (the default) and `always`.
2. With `always` and one debug session running, closing the window shows a warning box "There is an active debug session, are you sure you want to stop it?" with a Stop Debugging button and a Cancel button.
3. With several sessions running the box reads "There are active debug sessions, are you sure you want to stop them?"
4. Only top-level sessions are counted, so sessions started by another session do not add to the number.
5. Cancel keeps the window open and the sessions running, and Stop Debugging lets the window close.
6. The check does not look at why the window is shutting down, so quitting and reloading ask the same question.
7. With `never`, or with no session running, no box shows.

## Unclear
- DL-2.10 piece 8: whether Cursor shows its own box and then the stock box in a row, with `window.confirmBeforeClose` at `always`, was read from the quit command and not run.
- DL-2.11: on macOS it is not clear whether Cmd+Q reaches the app-menu box in the main process or the page keybinding, so I cannot say whether the Quit Cursor? box (DL-2.17) shows there. I wrote no Cursor piece for it.
- DL-2.18 piece 6: Alt+F4 and the Windows and Linux Ctrl+Shift+W reach the box only through the Close Window command, and it is not confirmed that the window's own close button does.
- DL-2.16 pieces 3 to 5 describe the three read-only settings from the stock setting text, and Cursor was not run to see them applied.
- Nothing was run on screen, so all Cursor-only pieces (DL-2.17 and DL-2.18) rest on bundle strings.
