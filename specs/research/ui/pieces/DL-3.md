Group: DL-3 File open and save pickers. Pass: 3. Date: 2026-10-25.

### DL-3.1
1. Choosing File > Open File... shows the OS Open File box, and picking one or more files opens each as an editor tab.
2. Pressing Ctrl+O (not macOS native) or running `workbench.action.files.openFile` from the Command Palette shows the same box.
3. Setting `window.openFilesInNewWindow` chooses whether the picked files open in a new window or in an existing one, with values `on`, `off` and `default`.
4. Running `workbench.action.files.openFileInNewWindow` shows the same box and opens the picked files in a new window.
5. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Open File box.

### DL-3.2
1. Choosing File > Open Folder... shows the OS box for picking one folder, and the picked folder opens as the window's workspace.
2. Pressing Ctrl+K Ctrl+O (Windows and Linux only) or running `workbench.action.files.openFolder` from the Command Palette shows the same box.
3. Setting `window.openFoldersInNewWindow` chooses whether the picked folder opens in a new window or replaces the last active window, with values `on`, `off` and `default`.
4. Running `workbench.action.files.openFolderInNewWindow` shows the same box and opens the picked folder in a new window.
5. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS box for picking a folder.
6. On stock Windows the OS box button reads "Select folder". [stock only]

### DL-3.3
1. On macOS native, choosing File > Open... shows one OS box where either a file or a folder can be picked, and a file opens as an editor tab while a folder opens as the window's workspace.
2. Pressing Cmd+O (macOS native) or running `workbench.action.files.openFileFolder` from the Command Palette shows the same box.
3. Running `workbench.action.files.openFileFolderInNewWindow` shows the same box and opens the pick in a new window.
4. Settings `window.openFilesInNewWindow` (for a picked file) and `window.openFoldersInNewWindow` (for a picked folder) choose whether the pick opens in a new window, with values `on`, `off` and `default`.
5. In every OS Open box a macOS `.app` package is browsed like a folder and is not picked as one file.

### DL-3.4
1. Choosing File > Open Workspace from File... shows an OS box with a Workspace file type filter and an Open button, and picking a `.code-workspace` file opens that workspace in the window.
2. Running `workbench.action.openWorkspace` from the Command Palette shows the same box.
3. Running `workbench.action.openWorkspaceInNewWindow` shows the same box and opens the picked workspace in a new window.
4. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS box for picking a workspace file.

### DL-3.5
1. Choosing File > Add Folder to Workspace... shows an OS box with an Add button where one or more folders can be picked.
2. Every picked folder is added to the open workspace, and a window that had a single folder becomes a multi-root workspace.
3. The same box also opens from the Explorer context menu entry and from `workbench.action.addRootFolder` in the Command Palette.

### DL-3.6
1. Choosing File > Save As... shows a box titled "Save As" that starts at the active editor file's own folder and name, and saving writes the content to the chosen name and folder.
2. Pressing Ctrl+Shift+S or running `workbench.action.files.saveAs` from the Command Palette shows the same box.
3. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Save box.
4. In the in-app Save box a name typed without an extension gets the extension of the first file type filter added to it.

### DL-3.7
1. Pressing Ctrl+S or choosing File > Save on a new untitled tab shows the Save picker.
2. The suggested file name in the picker is built from the tab name and the extension of its language.
3. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Save box for the untitled tab.

### DL-3.8
1. The OS Save box has a file type list.
2. All Files is the first entry in the list.
3. The entry that matches the current file's extension comes right after All Files, named after its language, or after the extension in capitals when no language matches.
4. Every other known language follows in sorted order.
5. No Extension is the last entry in the list.

### DL-3.9
1. Choosing File > Save Workspace As... or running `workbench.action.saveWorkspaceAs` from the Command Palette shows a Save box titled "Save Workspace" that filters to the Workspace type.
2. Leaving the `.code-workspace` extension off the typed name makes the saved file end in `.code-workspace`.
3. Setting `files.simpleDialog.enable` turned on shows the in-app path list instead of the OS Save box.

### DL-3.10
1. Running `workbench.action.files.newFile` shows a Save box titled "Create File" whose button reads "Create File".
2. The box suggests the name `Untitled.txt` in the default folder, or the file name the command was given.
3. Confirming a name creates an empty file at that path and opens it as a pinned editor tab, and cancelling creates nothing.
4. Typing a file name in the New File list adds an entry "Create New File ({0})" at the top that opens this box with that name. [stock only]

### DL-3.11
1. Turning on `files.simpleDialog.enable` makes every open and save command show the app's own path list in place of the OS box.
2. Setting `files.simpleDialog.enable` is off by default, so the OS box shows until a person turns it on.
3. A file system that is not local files always shows the in-app path list, whatever the setting says.
4. The in-app list shows the same title and confirm button label that the OS box would show for that command.

### DL-3.12
1. The simple dialog shows the current folder's entries in a quick pick list, with a ".." row that goes up one folder.
2. Folders are listed before files, and each file shows its file icon.
3. Files that do not match the file type filter are left out of the list.
4. Pressing Enter on a folder row steps into that folder and the list refreshes to show its entries.

### DL-3.13
1. The text box in the simple dialog holds a full path that starts at the picker's folder.
2. Typing a slash after a folder name jumps the list to that folder.
3. Typing `~` jumps the list to the home folder, and typing a backslash jumps it to the drive root.
4. Typing a path that does not exist followed by a slash shows "The path does not exist. Use ~ to go to your home directory." under the box.
5. The empty text box shows the placeholder "Folder path". [stock only]

### DL-3.14
1. While a person types a name, the first matching entry is completed inline in the text box, with the added letters selected.
2. Pressing Up or Down fills the text box with the highlighted entry.
3. When a screen reader is on, the inline completion is skipped.

### DL-3.15
1. Pressing the eye button in the simple dialog hides entries whose names start with a dot, and pressing it again shows them.
2. While dot files are shown the button is an open eye with the tooltip "Hide dot files", and while they are hidden it is a closed eye with the tooltip "Show dot files".
3. Dot files are shown by default, and the choice is remembered per workspace under the key `remoteFileDialog.showDotFiles`.

### DL-3.16
1. Accepting text that cannot be turned into a path shows "Please enter a valid path." under the box.
2. Accepting a folder in a file-only Open box shows "Please select a file.", and accepting a file in a folder-only Open box shows "Please select a folder."
3. Accepting a path that does not exist in an Open box shows "Please enter a path that exists."
4. Accepting an existing folder's name in a Save box shows "The folder already exists. Please use a new file name.", and an unusable file name shows "Please enter a valid file name."
5. Accepting a read-only folder in a Save box shows "This folder cannot be used as a save destination. Please choose another folder".
6. On Windows, accepting the bare root path shows "Please start the path with a drive letter."

### DL-3.17
1. Accepting an existing file name in the simple Save As shows a small prompt that asks "{0} already exists. Are you sure you want to overwrite it?", with an OK button and a Cancel button.
2. Pressing OK overwrites the file, and pressing Cancel closes the prompt and returns to the path list with nothing written.
3. Pressing Escape or typing in the prompt's box closes the prompt and counts as Cancel.
4. With the OS Save box the replace question is asked by the OS and not by this prompt.

### DL-3.18
1. Accepting a Save path in a folder that does not exist shows a prompt asking "The folder {0} does not exist. Would you like to create it?" with the missing folder's name.
2. Pressing OK in that prompt goes on with the save, and pressing Cancel returns to the path list.
3. Accepting a path in the folder-only Open box whose last folder is missing, but whose parent exists, asks "The folder {0} does not exist. Would you like to create it?", and OK creates the folder and picks it. [stock only]
4. If creating the folder fails, "Could not create folder: {0}" shows under the box. [stock only]

### DL-3.19
1. In a window connected to a remote machine, the simple dialog shows a "Show Local" button.
2. Pressing "Show Local" closes the list and opens the matching picker for the local machine's files.
3. Hovering the "Show Local" button shows the local command's label with its key, such as "Open Local File... (Ctrl+O)".
4. Running `workbench.action.files.openLocalFile`, `workbench.action.files.openLocalFolder`, `workbench.action.files.openLocalFileFolder` or `workbench.action.files.saveLocalFile` opens the picker for the local machine's files.
5. While the remote picker is showing, Ctrl+O, Ctrl+K Ctrl+O (not macOS), Cmd+O (macOS) and Ctrl+Shift+S or Cmd+Shift+S open the matching local picker.

### DL-3.20
1. A picker starts in the folder of the last active file.
2. The Open Folder picker starts in the folder that contains the last workspace folder.
3. The Open Workspace picker starts in the folder of the current workspace file.
4. With no such folder, a picker starts in the folder set in `files.dialog.defaultPath`, and then in the user home folder.
5. Setting `files.dialog.defaultPath` takes an absolute folder path, and the Settings editor shows "Default path for file dialogs must be an absolute path (e.g. C:\myFolder or /myFolder)." for a path that is not.

## Unclear

- DL-3.2: `workbench.action.files.openFolderViaWorkspace` has no piece because its `when` needs `openFolderWorkspaceSupport` to be false, and I found no way to see it on desktop.
- DL-3.3: the file-or-folder box is macOS native only, so pieces 1 and 2 cannot be checked on Windows or Linux.
- DL-3.19: Show Local appears when the picker's file system is not local and local files are among the available file systems, which I read from code and did not run against a real remote window.
- Cursor differences for DL-3.2, DL-3.13 and DL-3.18 are written as `[stock only]` pieces, since Cursor shows the OS default label, no "Folder path" placeholder, and no folder-create offer in the folder-only Open box.
