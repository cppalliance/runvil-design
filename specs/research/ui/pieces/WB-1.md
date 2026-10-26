Group: WB-1 Saving and file state. Pass: 3. Date: 2026-10-25.

### WB-1.1
1. If the file on disk is newer than the copy you edited, the notification reads "Failed to save '<name>': The content of the file is newer. Do you want to overwrite the file with your changes?" with the buttons Overwrite and Revert.
2. If the file is read-only, it reads "Failed to save '<name>': File is read-only. Select 'Overwrite' to attempt to make it writeable." with Overwrite, Save As... and Revert.
3. On a permission error, or a read-only file the app can overwrite with elevated rights, the first button is Retry as Admin... or Overwrite as Admin... on Windows and Retry as Sudo... or Overwrite as Sudo... elsewhere, with a message that names it.
4. For any other error it reads "Failed to save '<name>': <error>" with Retry, Save As... and Revert.
5. Save As... opens the Save As picker, and if that fails or is cancelled the same notification shows again.
6. The notification closes by itself as soon as the file is saved or reverted.

### WB-1.2
1. When a save command fails with an error that is not a cancel, an error notification reads "Failed to save '<file names>': <error message>".
2. The notification has a Retry button that runs the same save again.
3. It also has a Revert button, or Revert All when several files failed, that discards the unsaved changes; the button is left out when every failed editor is an untitled file so that new files are never discarded this way.
4. The notification id is built from the failed files, so a repeat failure for the same files replaces the earlier notification instead of stacking.

### WB-1.3
1. Running File: Save All Files from the Command Palette saves every unsaved file in every editor group at once.
2. Untitled files are skipped, so no Save As dialog opens and they stay unsaved.
3. It saves as an explicit save, so save-time steps such as format on save run as they do for Ctrl+S.
4. Save All in the File menu is a different command that also offers to save untitled files.

### WB-1.4
1. Pressing the chord or running File: Save without Formatting saves the active editor and skips every save-time step.
2. The skipped steps are format on save, code actions on save, trimming trailing whitespace, inserting a final newline and trimming final newlines.
3. The file is written even when it has no unsaved changes.

### WB-1.5
1. File > New Text File opens a new empty untitled text editor tab in the active group and keeps it open as a pinned tab.
2. Pressing Ctrl+N (Cmd+N on macOS) does the same, and the Command Palette lists it as File: New Untitled Text File.
3. Right-clicking the empty editor area or the tab strip offers a New Text File row, and the Open Editors view title has a New Untitled Text File button.
4. Setting `files.defaultLanguage` names the language id that a new untitled file starts with, and an empty value leaves it unset.
5. Setting `files.defaultLanguage` to `${activeEditorLanguage}` makes the new file start in the language of the text editor that was active before it opened, and the person can still change the language later by hand or by auto-detection.
6. The default language is not applied when a file is created with an associated path or with an explicit language.

### WB-1.6
1. File > Auto Save is a toggle row that shows a check mark whenever `files.autoSave` is anything other than `off`.
2. Choosing it while auto save is off sets `files.autoSave` to `afterDelay`, and choosing it while any auto save mode is on sets it to `off`.
3. File: Toggle Auto Save in the Command Palette does the same, with the description "Toggle the ability to save files automatically after typing".
4. Setting `files.autoSave` chooses `off` (the default on desktop), `afterDelay`, `onFocusChange` or `onWindowChange`.
5. With `afterDelay` an editor with unsaved changes is saved once `files.autoSaveDelay` milliseconds have passed, which is 1000 by default, and the setting only applies in that mode.
6. With `onFocusChange` the editor saves when it loses focus, and with `onWindowChange` it saves when the window loses focus.
7. Untitled editors and read-only editors are never auto saved.
8. Setting `files.autoSaveWorkspaceFilesOnly` limits auto save to files inside the open workspace, and it is off by default.
9. Setting `files.autoSaveWhenNoErrors` holds an auto save back while the file has errors in the Problems view and saves it once they are gone, and it is off by default.

## Unclear
none
