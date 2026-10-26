Group: DL-3 File open and save pickers. Section: Dialogs. Date: 2026-10-25. Pass: 2.
Features: 20.

Short names used below: V = stock VS Code 1.128.0 source, N = VS Code main 1.144.0 (only used to spot newer things), C bundle = the Cursor 3.27.12 main bundle `workbench.desktop.main.js` (minified, so I cite message numbers from `out\nls.messages.json` and command ids), C main = Cursor `out\main.js`. V paths are written in full with the `V\` prefix.

### DL-3.1 Open File
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick one or more files in the OS Open File box (or in the in-app list when it is turned on) and open them as editor tabs.
- Commands: workbench.action.files.openFile; workbench.action.files.openFileInNewWindow
- Menus: MenubarFileMenu; CommandPalette
- Keys: Ctrl+O (not macOS native)
- Settings: window.openFilesInNewWindow; files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; C bundle (command id workbench.action.files.openFile, messages 1902 and 3268); C main (pickFile, doPick)

### DL-3.2 Open Folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick a folder and open it as the window's workspace, and on stock Windows the OS box button reads "Select folder" while Cursor leaves the OS default label.
- Commands: workbench.action.files.openFolder; workbench.action.files.openFolderInNewWindow; workbench.action.files.openFolderViaWorkspace
- Menus: MenubarFileMenu; CommandPalette
- Keys: Ctrl+K Ctrl+O (Windows and Linux only)
- Settings: window.openFoldersInNewWindow; files.simpleDialog.enable
- Cursor: changed (Windows "Select folder" button label is missing)
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (command id workbench.action.files.openFolder, message 1901); C main (pickFolder with no buttonLabel)

### DL-3.3 Open file or folder in one box (macOS)
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS native the Open box lets you choose a file or a folder in the same dialog (macOS `.app` packages are browsed like folders in every OS Open box).
- Commands: workbench.action.files.openFileFolder; workbench.action.files.openFileFolderInNewWindow
- Menus: MenubarFileMenu; CommandPalette
- Keys: Cmd+O (macOS native)
- Settings: window.openFilesInNewWindow; window.openFoldersInNewWindow
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; C bundle (command id workbench.action.files.openFileFolder, message 3271); C main (doPick with treatPackageAsDirectory)

### DL-3.4 Open Workspace from File
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick a `.code-workspace` file in a box that filters to the Workspace type and has an Open button, then open it in the window.
- Commands: workbench.action.openWorkspace; workbench.action.openWorkspaceInNewWindow
- Menus: MenubarFileMenu; CommandPalette
- Keys: none found
- Settings: files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\platform\dialogs\electron-main\dialogMainService.ts; V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; C bundle (command id workbench.action.openWorkspace, messages 1903, 1904, 3272, 12461)

### DL-3.5 Add Folder to Workspace
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick one or more folders (the button reads Add) to add to the open workspace, which turns a single folder into a multi-root workspace.
- Commands: workbench.action.addRootFolder; addRootFolder
- Menus: MenubarFileMenu; ExplorerContext; CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\browser\actions\workspaceCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (command id workbench.action.addRootFolder, picker with messages 3289 and 3290)

### DL-3.6 Save As
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick a new name and folder for the active editor's file, starting from the file's own path, and save a copy there.
- Commands: workbench.action.files.saveAs
- Menus: MenubarFileMenu; CommandPalette
- Keys: Ctrl+Shift+S
- Settings: files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\contrib\files\browser\fileConstants.ts; V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; V\src\vs\workbench\services\textfile\browser\textFileService.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (command id workbench.action.files.saveAs, messages 7060, 12463, 12464)

### DL-3.7 Save an untitled file
- Tag: both
- Evidence: source
- Boundary: no
- What: Saving a new untitled tab opens the same Save picker with a suggested file name built from the tab name and its language.
- Commands: workbench.action.files.save
- Menus: MenubarFileMenu
- Keys: Ctrl+S
- Settings: files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\services\textfile\browser\textFileService.ts; V\src\vs\workbench\services\workingCopy\common\fileWorkingCopyManager.ts; V\src\vs\workbench\contrib\files\browser\fileActions.contribution.ts; C bundle (command id workbench.action.files.save)

### DL-3.8 Choose a file type when saving
- Tag: both
- Evidence: source
- Boundary: no
- What: The OS Save box has a file type list ordered All Files, the type that matches the current extension, every known language, then No Extension.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; C bundle (messages 12465 All Files and 12466 No Extension)

### DL-3.9 Save Workspace As
- Tag: both
- Evidence: source
- Boundary: no
- What: Pick where to save the workspace file in a box titled Save Workspace that filters to the Workspace type, and `.code-workspace` is added if you leave it off.
- Commands: workbench.action.saveWorkspaceAs
- Menus: MenubarFileMenu; CommandPalette
- Keys: none found
- Settings: files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts; V\src\vs\workbench\services\workspaces\browser\abstractWorkspaceEditingService.ts; C bundle (command id workbench.action.saveWorkspaceAs, messages 3274, 13609)

### DL-3.10 Create File picker
- Tag: both
- Evidence: source
- Boundary: no
- What: The New File command with a file name opens a Save picker whose button reads Create File, then creates the empty file and opens it.
- Commands: workbench.action.files.newFile
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (command and Create File button are present, but the typed-name entry "Create New File" in the New File list is missing)
- Source: V\src\vs\workbench\contrib\files\browser\fileCommands.ts; V\src\vs\workbench\contrib\welcomeViews\common\newFile.contribution.ts; C bundle (command id workbench.action.files.newFile, message 7180, welcome.showNewFileEntries)

### DL-3.11 Switch to the in-app simple file dialog
- Tag: both
- Evidence: source
- Boundary: no
- What: Turning on `files.simpleDialog.enable` replaces every OS open and save box with the app's own path list, and file systems that are not local files always use it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts; V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (files.simpleDialog.enable, message 7268, shouldUseSimplified)

### DL-3.12 Browse folders in the simple dialog
- Tag: both
- Evidence: source
- Boundary: no
- What: The simple dialog lists the current folder in a quick pick with a ".." row, folders first, files with icons, files that miss the type filter hidden, and Enter on a folder steps into it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (SimpleFileDialog class with createBackItem and createItems)

### DL-3.13 Type a path in the simple dialog
- Tag: both
- Evidence: source
- Boundary: no
- What: The text box holds a full path that starts at the picker's folder, and typing a slash after a folder name, "~" for the home folder or a backslash for the drive root jumps the list there, with a "Folder path" placeholder in stock.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the "Folder path" placeholder text is missing)
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (tryUpdateItems, canTildaEscapeHatch, message 12475, no filePickBox.placeholder assignment)

### DL-3.14 Autocomplete in the simple dialog
- Tag: both
- Evidence: source
- Boundary: no
- What: While you type a name the first matching entry is completed inline with the added letters selected, and Up and Down fill the box with the highlighted entry, but the inline fill is skipped when a screen reader is on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (setAutoComplete with isScreenReaderOptimized check)

### DL-3.15 Show or hide dot files
- Tag: both
- Evidence: source
- Boundary: no
- What: An eye button in the simple dialog toggles files whose names start with a dot, its tooltip reads "Hide dot files" or "Show dot files", and the choice is remembered per workspace under the key `remoteFileDialog.showDotFiles`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (remoteFileDialog.showDotFiles, messages 12473 and 12474)

### DL-3.16 Path messages in the simple dialog
- Tag: both
- Evidence: source
- Boundary: no
- What: Accepting a bad path shows a message under the box, such as "Please enter a valid path.", "Please select a file.", "Please select a folder.", "The folder already exists. Please use a new file name.", "Please enter a valid file name." or "Please start the path with a drive letter."
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (messages 12475 to 12487)

### DL-3.17 Overwrite prompt in the simple Save As
- Tag: both
- Evidence: source
- Boundary: no
- What: Saving over a name that exists asks "{0} already exists. Are you sure you want to overwrite it?" as a small prompt with an OK button and a Cancel button, in place of the OS box's own replace question.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.simpleDialog.enable
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (yesNoPrompt, messages 12476 Cancel and 12479)

### DL-3.18 Offer to create a missing folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Saving into a folder that does not exist asks "The folder {0} does not exist. Would you like to create it?", and in stock the folder-only Open box asks the same when just the last folder is missing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.simpleDialog.enable
- Cursor: changed (the save case is present, the folder-only Open case is missing)
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; C bundle (message 12481 used only in the save branch, open branch ends at message 12484)

### DL-3.19 Show Local and Open Local commands in a remote window
- Tag: both
- Evidence: source
- Boundary: no
- What: In a window connected to a remote machine the simple dialog gets a "Show Local" button that closes the list and opens the picker for the local machine's files, backed by the Open Local File, Open Local Folder, Open Local and Save Local File commands.
- Commands: workbench.action.files.openLocalFile; workbench.action.files.openLocalFolder; workbench.action.files.openLocalFileFolder; workbench.action.files.saveLocalFile
- Menus: none found
- Keys: Ctrl+O (not macOS, when remoteFileDialogVisible); Ctrl+K Ctrl+O (not macOS, when remoteFileDialogVisible); Cmd+O (macOS, when remoteFileDialogVisible); Ctrl+Shift+S or Cmd+Shift+S (when remoteFileDialogVisible)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts; V\src\vs\workbench\contrib\remote\electron-browser\remote.contribution.ts; C bundle (the four command ids, remoteFileDialogVisible, messages 12467 to 12472)

### DL-3.20 Where a picker starts
- Tag: both
- Evidence: source
- Boundary: no
- What: Pickers start in the last active file's folder (Open Folder starts in the folder that contains the last workspace folder, Open Workspace starts in the current workspace file's folder), then fall back to the folder set in `files.dialog.defaultPath` or the user home.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: files.dialog.defaultPath
- Cursor: present
- Source: V\src\vs\workbench\services\dialogs\browser\abstractFileDialogService.ts; V\src\vs\workbench\contrib\files\browser\files.contribution.ts; C bundle (files.dialog.defaultPath, preferredHome, messages 7266 and 7267)

## Handed off

- Download... in the Explorer, which opens a Save picker titled "Choose Where to Download" for non-local files (`explorer.download`, `V\src\vs\workbench\contrib\files\browser\fileImportExport.ts`): SB-2
- Remove Folder from Workspace... (`workbench.action.removeRootFolder`, a quick pick of folders and not a file picker): SB-2
- The Open Folder and Add Folder buttons in the Explorer empty state (`V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts`), which call the commands above: SB-2
- The Open File, Open Folder and Open buttons on the Welcome page and the empty editor watermark, which call the commands above: ED-9
- Cursor's "Choose a folder for your new project" picker with the button "Create Project Here" (`workbench.action.files.newProject` constant, `createProjectAndReload` in the C bundle), part of the agent-first onboarding: ED-9
- The folder pickers in the Workspace Trust editor ("Select Folder To Trust", "Add Folder"): DL-5
- "Workspace does not exist" confirm with an Open Workspace button, shown only after connecting to a remote (`V\src\vs\workbench\contrib\remote\common\remote.contribution.ts`), which then opens the pickers here: DL-1
- "'{0}' is marked as read-only. Do you want to save anyway?" with Save Anyway, raised during Save As (`V\src\vs\workbench\services\textfile\browser\textFileService.ts`): DL-2
- Pickers that belong to other features (Install from VSIX, profile import and export, chat import and export, Update from file, merge editor developer commands): left out with those features, no group here

## Unclear

- I did not run Cursor or VS Code. The look of each box, the exact order of the OS file type list and where each key works on each OS are read from code and message numbers only.
- `workbench.action.files.openFolderViaWorkspace` is registered in Cursor (2 hits) but its `when` needs `openFolderWorkspaceSupport` to be false, and that key defaults to true, so I did not find a way to see it on desktop.
- The four `...InNewWindow` commands have no menu, palette entry or key in V or Cursor. They are reached through code (Welcome page, other commands) or a custom keybinding, so I folded them into the Open features instead of listing them.
- Newer than 1.128 (seen only in N, so they are changes to features above and not features of their own): `showOpenDialog` in N now starts in the default folder path when it only allows folders (N `src\vs\workbench\services\dialogs\electron-browser\fileDialogService.ts`), and the simple dialog's create-folder offer in N can reach any writable ancestor and not just the parent (`canCreateFolder` in N `src\vs\workbench\services\dialogs\browser\simpleFileDialog.ts`). Cursor has neither (searched, zero hits for `canCreateFolder`).
- Pass 1 listed the simple dialog's create-folder offer only under newer-than-1.128. V 1.128.0 already has a one-level version for the folder-only Open box, and Cursor's open branch has no create offer at all, so DL-3.18 is tagged `changed` in Cursor. Only the any-ancestor rule is newer.
- Whether `files.simpleDialog.enable` also changes the macOS "Open..." box is only shown by code (the same `shouldUseSimplified` rule). Not run.
- The simple dialog's Save As uses the first filter extension to add a missing extension (for example `.code-workspace`). I left it inside DL-3.9 and DL-3.6 and did not make it a feature.
