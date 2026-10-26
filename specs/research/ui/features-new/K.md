Bucket: K. Date: 2026-10-25. Pass: 4.

### NEW-K-1 Open Recent picker
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: The person presses Ctrl+R or runs Open Recent from the Command Palette and gets a list of recent folders, workspaces and files to open, with a button on each row to remove it and, in stock only, a marker for folders and workspaces already open in a window.
- Commands: workbench.action.openRecent; workbench.action.quickOpenRecent
- Menus: MenubarRecentMenu (More..., group y_more, order 1); CommandPalette (Open Recent only, category File; Quick Open Recent has f1 false and is not listed)
- Keys: Ctrl+R on workbench.action.openRecent (the Control key on macOS too); none found for workbench.action.quickOpenRecent
- Settings: none found
- Cursor: changed - both command ids, the context key inRecentFilesPicker, the headings "folders & workspaces" and "files", the placeholder "Select to open (hold Ctrl-key to force new window or Alt-key for same window)", the remove button and the two unsaved-files tooltips are in the bundle; the four "Opened in a Window" and "Opened in Active Window" tooltips and the code that adds those buttons are not
- Source: V\src\vs\workbench\browser\actions\windowActions.ts (BaseOpenRecentAction lines 47-278, OpenRecentAction lines 280-309, QuickPickRecentAction lines 311-326); C bundle (class with id workbench.action.openRecent and key R, class with id workbench.action.quickOpenRecent and title "Quick Open Recent...", strings "folders & workspaces", "Remove from Recently Opened", "Folder With Unsaved Files"; no hit for "Folder Opened in a Window")
- Pieces:
  1. Pressing Ctrl+R, or running Open Recent... from the Command Palette or File > Open Recent > More..., opens a list with the placeholder "Select to open (hold Ctrl-key to force new window or Alt-key for same window)", which says Cmd-key and Option-key on macOS.
  2. The list has a "folders & workspaces" heading (just "folders" when none of the entries is a workspace) over the recent folders and workspaces, then a "files" heading over the recent files, in most recently used order.
  3. Each row shows the entry name with a file, folder or workspace icon and its parent path as a description, and typing in the box narrows the rows by name or path.
  4. Choosing a row opens it in the current window, holding Ctrl (Cmd on macOS) while choosing opens it in a new window, and holding Alt (Option on macOS) reuses the current window.
  5. When the first recent entry is the workspace you are already in, the highlight starts on the second row.
  6. Each row has a Remove from Recently Opened button, and clicking it takes the entry out of the list.
  7. A folder or workspace that has unsaved backups shows an always-visible dot button in place of the remove button, so it cannot be removed (see DL-2.14).
  8. A folder or workspace that is already open in a window shows an always-visible window icon with the tooltip "Folder Opened in a Window" or "Workspace Opened in a Window", or "Folder Opened in Active Window" or "Workspace Opened in Active Window" for the window you are in, and clicking the first kind removes the entry from the list. [stock only]
  9. Quick Open Recent (`workbench.action.quickOpenRecent`) shows the same list in quick-navigate mode with the input box hidden, moves through it with the keys bound to it, and is not shown in the Command Palette.

### NEW-K-2 Auto save
- Group: WB-1
- Tag: both
- Evidence: source
- Boundary: no
- What: The person switches auto save on or off from the File menu or the Command Palette, or picks a mode in settings, so files with unsaved changes save themselves after a delay or when focus moves away.
- Commands: workbench.action.toggleAutoSave
- Menus: MenubarFileMenu (Auto Save, group 5_autosave, order 1, checked while files.autoSave is not off); CommandPalette (File: Toggle Auto Save)
- Keys: none found
- Settings: files.autoSave; files.autoSaveDelay; files.autoSaveWorkspaceFilesOnly; files.autoSaveWhenNoErrors
- Cursor: present - the command id with its description, the File menu row in group 5_autosave (checked while `files.autoSave` is not off) and all four settings were found in the bundle; `files.autoSave` has the same default (off on desktop, after a delay on web) and is in Cursor's list of commonly used settings
- Source: V\src\vs\workbench\contrib\files\browser\fileActions.ts (lines 538-556); V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts (lines 717-724); V\src\vs\workbench\contrib\files\browser\files.contribution.ts (lines 261-293); V\src\vs\workbench\services\filesConfiguration\common\filesConfigurationService.ts (lines 464-475); V\src\vs\workbench\browser\parts\editor\editorAutoSave.ts; C bundle (id workbench.action.toggleAutoSave, menu group 5_autosave with the toggled condition on config.files.autoSave, the four settings, product.json commonlyUsedSettings)
- Pieces:
  1. File > Auto Save is a toggle row that shows a check mark whenever `files.autoSave` is anything other than `off`.
  2. Choosing it while auto save is off sets `files.autoSave` to `afterDelay`, and choosing it while any auto save mode is on sets it to `off`.
  3. File: Toggle Auto Save in the Command Palette does the same, with the description "Toggle the ability to save files automatically after typing".
  4. Setting `files.autoSave` chooses `off` (the default on desktop), `afterDelay`, `onFocusChange` or `onWindowChange`.
  5. With `afterDelay` an editor with unsaved changes is saved once `files.autoSaveDelay` milliseconds have passed, which is 1000 by default, and the setting only applies in that mode.
  6. With `onFocusChange` the editor saves when it loses focus, and with `onWindowChange` it saves when the window loses focus.
  7. Untitled editors and read-only editors are never auto saved.
  8. Setting `files.autoSaveWorkspaceFilesOnly` limits auto save to files inside the open workspace, and it is off by default.
  9. Setting `files.autoSaveWhenNoErrors` holds an auto save back while the file has errors in the Problems view and saves it once they are gone, and it is off by default.

### NEW-K-3 Developer: Apply Update from File
- Group: WB-8
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows the person runs Developer: Apply Update from File... from the Command Palette, picks a setup file, and the app updates from that file instead of downloading one.
- Commands: _update.applyupdate
- Menus: CommandPalette (category Developer, Windows only)
- Keys: none found
- Settings: update.enableWindowsBackgroundUpdates
- Cursor: changed - the command id, its Windows-only registration, the Developer category, the Setup .exe filter and the idle-state condition are in the bundle; Cursor words the entry "Apply Update...", the dialog title "Apply Update" and the button "&&Update", where stock says "Apply Update from File...", "Apply Update from File" and the same button; the main-process updater behind it was not read
- Source: V\src\vs\workbench\contrib\update\browser\update.contribution.ts (lines 215-246); V\src\vs\platform\update\electron-main\updateService.win32.ts (lines 653-671); V\src\vs\platform\update\common\update.config.contribution.ts (lines 66-73); C bundle (class with id _update.applyupdate, category Developer, precondition idle, strings "Apply Update...", "Apply Update", "&&Update")
- Pieces:
  1. On Windows the Command Palette lists Developer: Apply Update from File..., and the command is not registered on macOS or Linux.
  2. Running it opens a file picker titled "Apply Update from File" that shows only setup `.exe` files and has an Update button.
  3. Closing the picker without choosing a file does nothing.
  4. Choosing a setup file makes the updater treat that file as an update that has already been downloaded, with no network download.
  5. When `update.enableWindowsBackgroundUpdates` is on, which is the default, and the app is a per-user install, the update starts installing at once.
  6. Otherwise the update becomes ready, and the person finishes it from the update control or the Restart to Update row (see TB-5).
  7. The command is available only while the updater is idle, so it cannot be run while an update is downloading, downloaded or ready.
  8. In Cursor the entry reads Developer: Apply Update... and the picker is titled Apply Update.

### NEW-K-4 Switch Window picker
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Switch Window... to see every open window in a list, picks one to bring it to the front, or closes a window from its row.
- Commands: workbench.action.switchWindow; workbench.action.quickSwitchWindow
- Menus: CommandPalette (Switch Window... only; Quick Switch Window has f1 false and is not listed); the macOS Window menu row Switch Window... (see TB-1.25)
- Keys: Ctrl+W on macOS (the Control key) on workbench.action.switchWindow; none found on Windows and Linux; none found for workbench.action.quickSwitchWindow
- Settings: none found
- Cursor: changed - both command ids, the context key inWindowsPicker, the placeholder "Select a window to switch to", the "window group" heading, the "Current Window" description, the "{0}, window with unsaved changes" label and a Close Window button were found in the bundle; the separate Close Active Window button for the window you are in was not, and Cursor's current-window row has no button
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts (BaseSwitchWindow lines 233-356, SwitchWindowAction lines 358-376, QuickSwitchWindowAction lines 378-391); C bundle (classes with ids workbench.action.switchWindow and workbench.action.quickSwitchWindow, closeWindowAction and closeDirtyWindowAction only, strings 12029 "Select a window to switch to" and "window group")
- Pieces:
  1. Running Switch Window... from the Command Palette opens a list with the placeholder "Select a window to switch to" and one row per open window, each with the window title and a file or folder icon.
  2. Choosing a row brings that window to the front and gives it focus.
  3. The row for the window you are in carries the description "Current Window", and the highlight starts on the next window below it.
  4. The rows of other windows have a Close Window button, and clicking it closes that window and takes its row out of the list.
  5. A window with unsaved changes shows an always-visible dot button, and a screen reader reads its row as "{title}, window with unsaved changes".
  6. When any window has floating windows, the rows are grouped, with a "window group" heading over each main window followed by its floating windows.
  7. The row for the window you are in has an always-visible Close Active Window button. [stock only]
  8. Quick Switch Window (`workbench.action.quickSwitchWindow`) shows the same list in quick-navigate mode with the input box hidden, and it is not shown in the Command Palette.
  9. On macOS Ctrl+W opens the list, and the OS Window menu has a Switch Window... row.

### NEW-K-5 Completions and checks in keybindings.json
- Group: ED-8
- Tag: both
- Evidence: source
- Boundary: no
- What: While editing the user `keybindings.json`, the person gets completions for command ids, hover descriptions and error marks for bad entries, and in stock can add `"systemWide": true` to make a key work even when the app is not focused.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the schema (id vscode://schemas/keybindings, title "Keybindings configuration"), its field descriptions, the multiple-commands error text and the configuration-editing extension that ties it to `keybindings.json` were found; the `systemWide` property, its two warning notifications and the code that registers the shortcuts have no hit in the bundle
- Source: V\src\vs\workbench\services\keybinding\browser\keybindingService.ts (class KeybindingsJsonSchema, lines 836-1016); V\extensions\configuration-editing\package.json (jsonValidation entries for keybindings.json); V\src\vs\workbench\contrib\keybindings\electron-browser\systemWideKeybindings.contribution.ts; V\src\vs\workbench\contrib\keybindings\electron-browser\systemWideKeybindings.specification.md; V\src\vs\platform\globalKeybindings\electron-main\globalKeybindingsMainService.ts; C bundle (schema id and nls strings, C extensions\configuration-editing\package.json; 0 hits for systemWide)
- Pieces:
  1. The built-in configuration-editing extension attaches the keybindings schema to the user `keybindings.json` and to each profile's copy, so those files get completion, hover text and error marks.
  2. Starting a new entry in the array offers a snippet with `key`, `command` and `when` as tab stops.
  3. The `command` value completes every known command id except those that start with an underscore, shows the command's description or menu title beside each, and also accepts the same id with a leading minus sign to remove its default key.
  4. The `key`, `when` and `args` properties show the descriptions "Key or key sequence (separated by space)", "Condition when the key is active." and "Arguments to pass to the command to execute."
  5. When a command declares the shape of its single argument, `args` is checked against that shape and becomes required if the argument is not optional.
  6. A `command` written as a list is marked with "Incorrect type. Expected "string". The field 'command' does not support running multiple commands. Use command 'runCommands' to pass it multiple commands to run."
  7. Comments and trailing commas in the file raise no errors.
  8. A `systemWide` property, true or false with false as the default, is offered with a description that calls it an OS-wide shortcut that works when the app is not focused, on desktop only, for single key combinations only, with `when` ignored. [stock only]
  9. An entry in the user file with `"systemWide": true` runs its command even when the app window is not focused, provided the key is a single combination, and keys from the defaults or from extensions never do. [stock only]
  10. A warning notification says the `when` clause is ignored for system-wide keybindings, and another lists key combinations that could not be registered because the OS or another app already holds them. [stock only]
