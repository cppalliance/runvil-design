Group: DL-2 Unsaved changes, close and quit prompts. Section: Dialogs. Date: 2026-10-25. Pass: 2.
Features: 20.

### DL-2.1 Save changes box for one dirty editor
- Tag: both
- Evidence: source
- Boundary: no
- What: Closing an editor that has unsaved changes brings it to the front and asks "Do you want to save the changes you made to {file}?" with the detail "Your changes will be lost if you don't save them." and three buttons: Save, Don't Save, Cancel. Save writes the file and closes (an untitled file goes to Save As, see DL-3). Don't Save reverts and closes, even if the revert fails. Cancel keeps the editor open. Group commands that close several editors one by one, such as Close Others, ask once per dirty editor in turn. Editors with their own close prompt (the merge editor, for example) show that one instead.
- Commands: workbench.action.closeActiveEditor; workbench.action.closeEditorsInGroup; workbench.action.closeOtherEditors; workbench.action.closeEditorsToTheRight
- Menus: MenubarFileMenu (Close Editor, group 6_close, order 2); EditorTitleContext (Close, Close Others, Close to the Right, Close All); EditorTitle (Close button)
- Keys: Ctrl+W, or Cmd+W on macOS (closeActiveEditor; Windows also Ctrl+F4); Ctrl+K W (closeEditorsInGroup); Cmd+Alt+T on macOS (closeOtherEditors)
- Settings: window.dialogStyle
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (handleCloseConfirmation, lines 1677-1706; doHandleCloseConfirmation, lines 1708-1860); V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts (showSaveConfirm, lines 167-222); V\src\vs\workbench\browser\parts\editor\editorCommands.ts (close commands, lines 828-930); V\src\vs\workbench\browser\parts\editor\editor.contribution.ts (menus at lines 394-398 and 742-750); C bundle (doShowSaveConfirm, nls 12452 to 12457, close command ids)

### DL-2.2 Save changes box for several files
- Tag: both
- Evidence: source
- Boundary: no
- What: When more than one file is at stake the same box says "Do you want to save the changes to the following {n} files?" and lists up to ten names, then "...n additional files not shown", and the first button becomes "Save All". Don't Save and Cancel stay. It is raised by Close All Editors and Close All Editor Groups (DL-2.3) and by quitting or closing with no backup (DL-2.5).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.dialogStyle
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts (doShowSaveConfirm, lines 186-222); V\src\vs\platform\dialogs\common\dialogs.ts (getFileNamesMessage, MAX_CONFIRM_FILES, lines 606-622); C bundle (nls 12454, 12455, 1898, 1899)

### DL-2.3 Close all editors or groups with unsaved changes
- Tag: both
- Evidence: source
- Boundary: no
- What: Close All Editors and Close All Editor Groups first sort the open editors. Editors that auto-save on focus change or window change are saved quietly. The rest are revealed so you can see them, then one box (DL-2.2 when there are several) asks for all the ordinary ones, and editors with their own close prompt are asked after. Cancel at any step stops the whole close. Close All Editors skips pinned tabs, Close All Editor Groups does not. A later stock build also skips editors that cannot be closed and removes only empty groups (newer-than-1.128, not in Cursor).
- Commands: workbench.action.closeAllEditors; workbench.action.closeAllGroups
- Menus: CommandPalette (f1, category View)
- Keys: Ctrl+K Ctrl+W (closeAllEditors); Ctrl+K Ctrl+Shift+W (closeAllGroups)
- Settings: files.autoSave
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorActions.ts (AbstractCloseAllAction, lines 555-757; CloseAllEditorsAction, line 757; CloseAllEditorGroupsAction, line 781); N\src\vs\workbench\browser\parts\editor\editorActions.ts (CannotClose check); C bundle (ids workbench.action.closeAllEditors and workbench.action.closeAllGroups, revealEditorsToConfirm)

### DL-2.4 Auto save replaces the prompt on close
- Tag: both
- Evidence: source
- Boundary: no
- What: With auto save set to focus change (or window change on Windows and Linux), closing a dirty file saves it instead of asking, because the box itself would take focus and trigger the save anyway. If that save fails, the normal Save, Don't Save, Cancel box appears. Untitled files are never auto-saved, so they still ask. At quit or window close, any file under an auto save mode is saved first and only the remainder is asked about.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.autoSave
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorGroupView.ts (autoSave branch, lines 1738-1770 and 1812-1830); V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (onBeforeShutdownWithModified, lines 88-113); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (files.autoSave, lines 261-275); C bundle (getAutoSaveMode, setting files.autoSave)

### DL-2.5 Save prompt at quit, window close or workspace change
- Tag: both
- Evidence: source
- Boundary: no
- What: When the app quits, a window closes or reloads, or the folder changes, and unsaved files could not be backed up, a Save All, Don't Save, Cancel box lists them (DL-2.2 wording). Save All saves each (an untitled file opens Save As); if any save fails or is cancelled the quit or close is stopped. Don't Save reverts them. Cancel stops the quit or close. It shows whenever hot exit is off. With hot exit on it still shows when a window closes while the app keeps running (macOS, or another window stays open on Windows and Linux) and when the folder or workspace changes, except that the "on exit and window close" value backs up folder windows instead.
- Commands: workbench.action.quit; workbench.action.closeWindow; workbench.action.closeFolder; workbench.action.reloadWindow
- Menus: MenubarFileMenu (Exit, group z_Exit, order 1, not macOS); MenubarFileMenu (Close Window, group 6_close, order 4); MenubarFileMenu (Close Folder or Close Workspace, group 6_close, order 3)
- Keys: Cmd+Q (macOS and Linux, quit); Cmd+Shift+W on macOS, Alt+F4 on Windows and Linux, or Ctrl+Shift+W (closeWindow); Ctrl+K F (closeFolder); Ctrl+R in development builds only (reloadWindow)
- Settings: files.hotExit
- Cursor: present
- Source: V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (onBeforeShutdown, handleModifiedBeforeShutdown, shouldBackupBeforeShutdown, confirmBeforeShutdown, lines 60-330); V\src\vs\workbench\electron-browser\desktop.contribution.ts (quit and Exit row, lines 72-92 and 129-139); V\src\vs\workbench\electron-browser\actions\windowActions.ts (CloseWindowAction, lines 36-67); V\src\vs\workbench\browser\actions\workspaceActions.ts (CloseWorkspaceAction, lines 161-185); C bundle (backup tracker class with isHotExitEnabled, nls 13589 to 13600)

### DL-2.6 Hot exit backs up unsaved files instead of asking
- Tag: both
- Evidence: source
- Boundary: no
- What: Hot exit remembers unsaved files, including untitled ones, so quitting does not ask. The default (on exit) backs up when you quit or close the last window on Windows and Linux. The second value also backs up any window that has a folder open, even when it is not the last window. Off turns it off and brings back the box in DL-2.5. Windows without a folder come back at the next start (DL-2.7).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.hotExit
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\files.contribution.ts (hotExitConfiguration, lines 123-146; files.hotExit, line 321); V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (shouldBackupBeforeShutdown, lines 185-230); V\src\vs\platform\files\common\files.ts (HotExitConfiguration); C bundle (setting files.hotExit with nls 7222 to 7228)

### DL-2.7 Reopen restores unsaved files
- Tag: both
- Evidence: source
- Boundary: no
- What: After a hot exit, the next start reopens windows as set by the restore setting (default all of them). Each backed-up unsaved file returns as a modified tab, opened in the background so it does not take focus, with its unsaved text back. Windows that had no folder return as empty windows holding those files. Backups of a folder window stay on disk until that folder is opened again, and the folder shows a dirty marker in Open Recent (DL-2.14).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.restoreWindows; files.hotExit
- Cursor: present
- Source: V\src\vs\workbench\services\workingCopy\common\workingCopyBackupTracker.ts (restoreBackups, lines 335-420); V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (onBeforeShutdownWithoutModified, lines 377-420); V\src\vs\platform\windows\electron-main\windowsMainService.ts (restore setting, lines 1009-1078); V\src\vs\workbench\electron-browser\desktop.contribution.ts (window.restoreWindows, lines 183-196); C bundle (unrestoredBackups, restoreBackups, setting window.restoreWindows)

### DL-2.8 Waiting box when a shutdown step is slow
- Tag: both
- Evidence: source
- Boundary: no
- What: If backing up, saving, reverting or discarding backups at shutdown takes a while, a pending box with a spinner appears and says "Backing up editors with unsaved changes is taking a bit longer...", "Saving editors...", "Reverting editors..." or "Discarding backups...". The backup box adds "Click 'Cancel' to stop waiting and to save or revert editors with unsaved changes." Cancel stops the wait and falls back to the save prompt. When untitled files are being saved the progress moves to the status bar instead, because a Save As box may need the screen. The box look is DL-1.16 and the progress service side is NT-5.10.
- Commands: none found
- Menus: none found
- Keys: Escape (cancel)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (withProgressAndCancellation callers, lines 256-262, 331-359, 361-375, 425-460); V\src\vs\workbench\services\progress\browser\progressService.ts (withDialogProgress); C bundle (nls 13596 to 13600)

### DL-2.9 Backup or save failed box
- Tag: both
- Evidence: source
- Boundary: no
- What: If the backup cannot be written, or the save or revert you chose fails, an error box says "The following editors with unsaved changes could not be saved to the backup location." (or "...could not be saved or reverted."). The detail lists the files and adds "Try saving or reverting the editors with unsaved changes first and then try again." OK keeps the window open. The second button is "Close Anyway", "Quit Anyway" or "Reload Anyway", depending on what you were doing, and goes ahead and loses the changes. It is skipped in an extension development window.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.hotExit
- Cursor: present
- Source: V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupTracker.ts (handleModifiedBeforeShutdown, showErrorDialog, toForceShutdownLabel, lines 142-182 and 232-262); C bundle (nls 13589 to 13595, 4086 to 4088)

### DL-2.10 Confirm before quit or close
- Tag: both
- Evidence: source
- Boundary: no
- What: One setting asks "Are you sure you want to exit?" (Windows and Linux), "Are you sure you want to quit?" (macOS) or "Are you sure you want to close the window?" with an Exit, Quit or Close Window button, Cancel, and a "Do not ask me again" checkbox that switches the setting to never. Values: always, only when a key started the action, or never. The desktop default is never, a browser tab defaults to key-only. Closing the last window on Windows and Linux counts as quit. A File menu toggle called Confirm Before Close exists only in the browser build. In Cursor the extra boxes in DL-2.17 and DL-2.18 come first, and the stock box still runs after them when the setting asks for it. I did not run the case where both appear.
- Commands: workbench.action.quit; workbench.action.toggleConfirmBeforeClose
- Menus: MenubarFileMenu (Confirm Before Close, group z_ConfirmClose, order 1, web only)
- Keys: Cmd+Q (macOS and Linux, quit)
- Settings: window.confirmBeforeClose
- Cursor: changed - Cursor's own quit and close boxes run first; the checkbox is not shown in the Agents window
- Source: V\src\vs\workbench\browser\window.ts (confirmOnShutdown, lines 183-210); V\src\vs\workbench\electron-browser\window.ts (onBeforeShutdown, lines 468-505); V\src\vs\workbench\electron-browser\desktop.contribution.ts (quit, lines 72-92); V\src\vs\workbench\browser\workbench.contribution.ts (window.confirmBeforeClose, lines 933-952); V\src\vs\workbench\browser\actions\windowActions.ts (toggle and menu row, lines 495-511); C bundle (confirmOnShutdown class with nls 4067 to 4073, setting default expression, veto.confirmBeforeClose)

### DL-2.11 macOS app menu Quit confirm
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS the Quit row of the app menu is handled by the main process, not the page. When the setting asks for it (always, or key-only and a key was used) it shows an OS question box "Are you sure you want to quit?" with Quit and Cancel. It only acts when a window has focus, is minimized, or no window is open. Cursor keeps this code and I found no call to its own quit box in it, so this row probably skips the DL-2.17 box. Read from code only, not run on a Mac.
- Commands: workbench.action.quit
- Menus: macOS application menu (built in the main process)
- Keys: Cmd+Q
- Settings: window.confirmBeforeClose
- Cursor: present
- Source: V\src\vs\platform\menubar\electron-main\menubar.ts (quit row, lines 424-440; confirmBeforeQuit, lines 464-481); C out\main.js (confirmBeforeQuit with nls 2182 to 2184)

### DL-2.12 Closing a floating window keeps unsaved editors
- Tag: both
- Evidence: source
- Boundary: no
- What: When you close a floating editor window, clean editors just close and every editor with unsaved changes (or its own close prompt) is moved back into the main window, so no save box appears. If an editor cannot be moved, the window stays open. When the confirm-before-close setting asks, the same "close the window" box (DL-2.10) comes first. If something vetoes the close, an error box shows the reason with "Try saving or reverting the editors with unsaved changes first and then try again."
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.confirmBeforeClose
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts (onBeforeUnload, doClose, mergeGroupsToMainPart, lines 265-280 and 425-480); V\src\vs\workbench\services\auxiliaryWindow\browser\auxiliaryWindowService.ts (lines 160-200); V\src\vs\workbench\services\auxiliaryWindow\electron-browser\auxiliaryWindowService.ts (lines 100-125); C bundle (excludeConfirming, mergeGroupsToMainPart, handleVetoBeforeClose, nls 12321 and 12316)

### DL-2.13 Untitled workspace save prompt
- Tag: both
- Evidence: source
- Boundary: no
- What: Closing a window that holds an untitled multi-root workspace, or switching to another workspace, asks "Do you want to save your workspace configuration as a file?" with the detail "Save your workspace if you plan to open it again." Save opens a Save As picker for a workspace file (cancel the picker and the window stays open). Don't Save deletes the untitled workspace. Cancel stops the close. A checkbox "Always discard untitled workspaces without asking" turns the prompt off. It is not asked when closing the last window on Windows and Linux, because that is a quit. With the setting off the untitled workspace is deleted silently.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.confirmSaveUntitledWorkspace
- Cursor: present
- Source: V\src\vs\workbench\services\workspaces\electron-browser\workspaceEditingService.ts (saveUntitledBeforeShutdown, lines 74-160); V\src\vs\workbench\electron-browser\desktop.contribution.ts (window.confirmSaveUntitledWorkspace, lines 167-171); C bundle (nls 13614 to 13618, setting window.confirmSaveUntitledWorkspace, 3 hits)

### DL-2.14 Folder or workspace with unsaved files in Open Recent
- Tag: both
- Evidence: source
- Boundary: no
- What: In the Open Recent list, a folder or workspace that still holds unsaved backups gets a dirty-dot button with the tooltip "Folder With Unsaved Files" or "Workspace With Unsaved Files". Clicking it asks "Do you want to open the folder to review the unsaved files?" (or workspace) with the detail "Folders with unsaved files cannot be removed until all unsaved files have been saved or reverted." Confirm opens that window. These entries cannot be removed from the list while dirty. Entries that have backups but are not in the history are added to the list too.
- Commands: workbench.action.openRecent; workbench.action.quickOpenRecent
- Menus: MenubarRecentMenu (More..., group y_more, order 1); CommandPalette (f1, category File)
- Keys: Ctrl+R (workbench.action.openRecent; the Control key on macOS too)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (BaseOpenRecentAction, lines 50-260; OpenRecentAction, lines 280-302); C bundle (nls 3240 to 3252, class dirty-workspace, command id workbench.action.openRecent)

### DL-2.15 Shutdown error box
- Tag: both
- Evidence: source
- Boundary: no
- What: If shutting down throws an unexpected error, an error box says "An unexpected error prevented the window to close" (or "...the application to quit", "...the window to reload", "...to change the workspace") with the detail "Error: {message}".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\window.ts (onBeforeShutdownError and toShutdownLabel, lines 505-580); C bundle (nls 12122, 12124 to 12127)

### DL-2.16 Save a read-only file anyway
- Tag: both
- Evidence: source
- Boundary: no
- What: When Save As lands on a path that the read-only settings cover, a warning box says "'{file}' is marked as read-only. Do you want to save anyway?" with the detail "Paths can be configured as read-only via settings." and Save Anyway or Cancel. It is raised from the Save As flow (DL-3.6), and the same text exists in the file working copy manager.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.readonlyInclude; files.readonlyExclude; files.readonlyFromPermissions
- Cursor: present
- Source: V\src\vs\workbench\services\textfile\browser\textFileService.ts (confirmMakeWriteable, lines 394-404 and 639-648); V\src\vs\workbench\services\workingCopy\common\fileWorkingCopyManager.ts (line 514); C bundle (nls 13121 and 13558, the three settings)

### DL-2.17 Quit Cursor? box
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor adds its own box to the Quit command. Unless the app reports running agents or shell tasks (then another flow asks), it opens "Quit Cursor?" with "Are you sure you want to quit?", Quit and Cancel, and a "Don't ask again" checkbox in the button row. It asks at the default setting, where stock never asks. Tick the box and choose Quit and it never asks again. The stock confirm (DL-2.10) runs after it when that setting asks. The macOS menu row may skip it (DL-2.11). The running-agent prompts are agent-only and left out.
- Commands: workbench.action.quit
- Menus: MenubarFileMenu (Exit, group z_Exit, order 1, Windows and Linux)
- Keys: Cmd+Q (macOS and Linux); Enter (Quit); Escape (Cancel)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (quit command rule with helper HOw and function pvg, dialogKey quitConfirmation, nls 12011, 12013, 12015, 12017)

### DL-2.18 Close this window? box
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's Close Window command closes at once when no folder or workspace is open. When one is open it first opens "Close this window?" with "Are you sure you want to close this window?", Close and Cancel, and a "Don't ask again" checkbox. It asks at the default setting, where stock never asks. On macOS Cursor adds a Cmd+W command that closes the active editor when there is one (which may ask DL-2.1) and otherwise runs Close Window. I did not confirm whether the window's own close button or the OS-level Alt+F4 reach this box, since the code only calls it from the command.
- Commands: workbench.action.closeWindow; workbench.action.closeWindowOrEditorOnMac
- Menus: MenubarFileMenu (Close Window, group 6_close, order 4); CommandPalette (f1)
- Keys: Cmd+Shift+W (macOS); Alt+F4 (Windows and Linux); Ctrl+Shift+W (Windows and Linux); Cmd+W (macOS, closeWindowOrEditorOnMac); Enter (Close); Escape (Cancel)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with id workbench.action.closeWindow, function pvg, dialogKey closeWindowConfirmation, command workbench.action.closeWindowOrEditorOnMac, nls 12012, 12014, 12016, 12018, 12030)

### DL-2.19 Slow close, quit or reload dialog with the pending steps
- Tag: both
- Evidence: source
- Boundary: no
- What: When closing, quitting, reloading or switching workspace takes a while, a progress message and then a box that lists the steps still running, with a button to force it, appear.
- Commands: join.closeStorage; join.textFiles; join.workingCopyBackups; join.fileWorkingCopyManager; join.workingCopyHistory; join.stopExtensionHosts; join.disconnectRemote; join.extensionDevelopment; join.browserViews; join.windowAnalytics (all ten are step ids shown in the list, not commands)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - the dialog branch with nls 12123, the force labels and the step labels are in the bundle, and Cursor adds two steps
- Source: V\src\vs\workbench\electron-browser\window.ts:505-590 (progressOnBeforeShutdown, onWillShutdown, labels); V\src\vs\workbench\electron-browser\desktop.main.ts:170; V\src\vs\workbench\services\textfile\electron-browser\nativeTextFileService.ts:64; V\src\vs\workbench\services\workingCopy\electron-browser\workingCopyBackupService.ts:36; V\src\vs\workbench\services\workingCopy\common\workingCopyHistoryService.ts:873; V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopyManager.ts:216; V\src\vs\workbench\services\extensions\common\abstractExtensionService.ts:214-232; V\src\vs\workbench\services\extensions\electron-browser\localProcessExtensionHost.ts:652; C bundle (steps join.browserViews "Cleaning up browser views", join.windowAnalytics "Flushing analytics events", nls 12095, 12123, 12128 to 12131, 13580, 13586)

### DL-2.20 Confirm before closing the window with a debug session running
- Tag: both
- Evidence: source
- Boundary: no
- What: With `debug.confirmOnExit` set to always, closing, quitting or reloading the window while a debug session is live shows a warning box that asks whether to stop it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.confirmOnExit
- Cursor: present - the setting with values never and always and the default never, and the shutdown check with the veto reason veto.debug, are in the bundle; the box strings come through NLS indexes
- Source: V\src\vs\workbench\contrib\debug\common\debugLifecycle.ts (lines 14-58); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (lines 663-672); C bundle (setting `debug.confirmOnExit`; class with onBeforeShutdown veto "veto.debug")

## Handed off
- The merge editor's own save prompt ("Do you want to save the changes you made to {n} files?" and its revert wording, in mergeEditorInputModel) - ED-6
- Terminal close and exit confirms (`terminal.integrated.confirmOnExit`, `confirmOnKill`) - PN-2
- Debug confirm on exit with live sessions (`debug.confirmOnExit`) - SB-5
- The status bar progress line "Closing the window is taking a bit longer..." that shows during a slow shutdown - NT-5 (NT-5.6)
- The Save As and Save Workspace As pickers opened by the Save buttons - DL-3
- The "'{0}' already exists. Do you want to replace it?" box when an untitled file is saved over a new file - DL-4 (DL-4.14)
- The "Failed to save" notification with its Retry, Save As, Overwrite and Compare actions (the text file save error handler) - NT-2

## Unclear
- Cursor's Save Error box: when a save fails and the save options carry showPrettyDialogOnError, Cursor shows a pretty dialog titled "Save Error" instead of the notification above. That flag name also appears in the Agents window bundle, which I did not read, and I found no setter in the editor bundle, so I left the box out as probably agent-only. If a person can reach it from the editor window, it would be a DL-2 or NT-2 feature.
- I could not tell from code whether the window's own close button and OS-level Alt+F4 raise the Cursor "Close this window?" box. The code calls it only from workbench.action.closeWindow, and from the macOS Cmd+W command.
- With window.confirmBeforeClose at always, Cursor may show its box and then the stock box in a row. Read from the quit command, not run.
- Which path Cmd+Q takes on macOS (the app-menu accelerator in the main process, or the page keybinding) is not clear, so I cannot say whether it reaches Cursor's Quit box.
- The when clause of Cursor's Cmd+W command (workbench.action.closeWindowOrEditorOnMac) uses context keys I did not decode. Stock's own macOS Cmd+W rule closes the window when no editor is visible and there is one group.
- The stock command Close Other Windows (workbench.action.closeOtherWindows, Command Palette only) has no id hit and no label hit in Cursor. It closes the other windows without asking, so it is outside this group, but no other group lists it (TB-3 does not), so I am noting it here.
- Cursor's quit and close flows also ask about running agents and background tasks. Those boxes are agent-only and not listed.
- Nothing was run on screen. Cursor findings are strings and minified code in the bundle.
