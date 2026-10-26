Bucket: A. Date: 2026-10-25. Pass: 4.

### NEW-A-1 Move between a view's filter box and its list with Ctrl+Up and Ctrl+Down
- Group: WB-3
- Tag: both
- Evidence: source
- Boundary: no
- What: In views that pair a filter or search box with a list, Ctrl+Down (Cmd+Down on macOS) moves keyboard focus from the box into the list and Ctrl+Up moves it back.
- Commands: widgetNavigation.focusNext; widgetNavigation.focusPrevious
- Menus: none found
- Keys: Ctrl+Down (widgetNavigation.focusNext, when a navigable container has focus and either no list is focused or the list is scrolled to the bottom); Ctrl+Up (widgetNavigation.focusPrevious, same container condition and the list is scrolled to the top); both use Cmd on macOS
- Settings: none found (the code reads workbench.navigibleContainer.enableDebug, but no configuration registers it)
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\widgetNavigationCommands.ts:144-175; V\src\vs\workbench\contrib\markers\browser\markersView.ts:204; V\src\vs\workbench\contrib\comments\browser\commentsView.ts:211; V\src\vs\workbench\contrib\comments\browser\commentThreadWidget.ts:101; V\src\vs\workbench\contrib\testing\browser\testingExplorerView.ts:273; V\src\vs\workbench\contrib\debug\browser\repl.ts:629; V\src\vs\workbench\contrib\preferences\browser\settingsEditor2.ts:480; V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts:155; C bundle (both command ids with the same keys and when clause; registrations named commentThreadWidget, keybindingsEditor, settingsEditor2, repl, testingExplorerView, markersView, commentsView and extensionsView)
- Pieces:
  1. In the Problems view and the Comments view, Ctrl+Down in the filter box moves focus to the list, and Ctrl+Up from the list moves it back to the filter box.
  2. In the Testing view, Ctrl+Down moves focus into the test tree and Ctrl+Up from the tree moves it to the filter box.
  3. In the Settings editor, Ctrl+Down in the search box moves focus to the table of contents on the left, and Ctrl+Up moves it back to the search box.
  4. In the Keyboard Shortcuts editor, Ctrl+Down in the search box moves focus to the shortcuts table, and Ctrl+Up moves it back to the search box.
  5. In the Debug Console, Ctrl+Down moves focus from the filter box to the output tree and from the tree to the input, and Ctrl+Up from the input moves focus to the tree.
  6. In an open comment thread, Ctrl+Down opens the reply box and focuses it, and Ctrl+Up from the reply box moves focus to the comments above it.
  7. When a list in these views has focus, the keys only jump between widgets once the list is scrolled to the bottom (Ctrl+Down) or the top (Ctrl+Up); before that they keep scrolling the list.

### NEW-A-2 Remove keyboard focus from the focused element
- Group: WB-3
- Tag: both
- Evidence: source
- Boundary: no
- What: A command that drops keyboard focus from whatever element has it, so typing no longer goes to that element; it is not in the Command Palette and is meant to be bound to a key.
- Commands: workbench.action.blur
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\windowActions.ts:442-460; C bundle (class with id workbench.action.blur, one hit)
- Pieces:
  1. Running "Remove keyboard focus from focused element" makes the focused element lose focus.
  2. Nothing happens when no element has focus or the focused item is not an HTML element.
  3. The command has no default key and no Command Palette entry, so a person reaches it only by assigning a shortcut to it in the Keyboard Shortcuts editor, where it is listed by its title.

### NEW-A-3 Grow or shrink the editor area with Increase and Decrease Editor Width and Height
- Group: ED-1
- Tag: both
- Evidence: source
- Boundary: no
- What: Four Command Palette commands change the editor area's size by 60 pixels at a time, in width or in height, no matter which part has focus.
- Commands: workbench.action.increaseViewHeight; workbench.action.decreaseViewHeight; workbench.action.increaseViewWidth; workbench.action.decreaseViewWidth
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts:1138-1277; V\src\vs\workbench\browser\layout.ts:1799-1860; C bundle (increaseViewHeight and decreaseViewHeight classes calling resizePart with the editor part, 1 hit each, same 60 pixel step)
- Pieces:
  1. "Increase Editor Height" makes the editor area 60 pixels taller and "Decrease Editor Height" makes it 60 pixels shorter, and the neighboring parts give up or take the space.
  2. "Increase Editor Width" and "Decrease Editor Width" do the same for width.
  3. With one editor group, the whole editor area is resized.
  4. With several editor groups, the active group is resized first, and if it cannot move in that direction the whole editor area is resized instead.
  5. The step is 60 pixels scaled for the screen, and the commands do nothing while a separate editor window has focus.
  6. The four commands are only in the Command Palette and have no default keys, unlike Increase and Decrease Current View Size, which resize the focused part (SB-1.4).

### NEW-A-4 Duplicate the folders as an untitled workspace in a new window
- Group: WB-10
- Tag: both
- Evidence: source
- Boundary: no
- What: Run "Workspaces: Duplicate As Workspace in New Window" and a new window opens on an untitled workspace that holds the same folders as the current window.
- Commands: workbench.action.duplicateWorkspaceInNewWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the command, label and multi-folder condition are present; the stock Agents window condition is not)
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts:292-325; C bundle (class with id workbench.action.duplicateWorkspaceInNewWindow, category Workspaces, precondition without an Agents window check)
- Pieces:
  1. Running the command opens a new window on an untitled multi-folder workspace that lists the same folders as the current window.
  2. The workspace settings of the current window are copied into the new workspace.
  3. The current window stays open and unchanged.
  4. When the current window is remote, the new window uses the same remote connection.
  5. The command is shown only when multi-folder workspaces are supported.
  6. In stock the command is also hidden in the Agents window; the Cursor bundle has no such condition. [stock only]
  7. It is reached only from the Command Palette under the Workspaces category, with no menu row and no key.

### NEW-A-5 Developer: Inspect Context Keys
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the command, click any element in the window, and the values of all context keys that apply at that spot are written to the developer tools console.
- Commands: workbench.action.inspectContextKeys
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:56-125; C bundle (class with id workbench.action.inspectContextKeys, 1 hit)
- Pieces:
  1. After the command runs, the mouse pointer becomes a crosshair everywhere and a translucent red box follows the element under the pointer.
  2. Clicking an element writes every context key value at that element to the console of the developer tools.
  3. The click is swallowed, so the clicked element does not react to it.
  4. After one click the crosshair and the red box go away.
  5. The command is in the Command Palette under Developer and has no menu row or key; the result is seen only by opening the developer tools console.

### NEW-A-6 Developer: Log Storage Database Contents
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the command and the contents of the storage database for the window are written to the developer tools console, then a box tells the person where to look.
- Commands: workbench.action.logStorage
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:420-441; C bundle (class with id workbench.action.logStorage, 1 hit)
- Pieces:
  1. Running the command writes the storage database contents to the developer tools console.
  2. An info box then says "The storage database contents have been logged to the developer tools." with the detail "Open developer tools from the menu and select the Console tab."
  3. The command is in the Command Palette under Developer and has no menu row or key.

### NEW-A-7 Developer: Log Working Copies
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the command and the list of open working copies and their backups is written to the Window log, and the Output view opens on that log.
- Commands: workbench.action.logWorkingCopies
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:442-483; C bundle (class with id workbench.action.logWorkingCopies, 1 hit)
- Pieces:
  1. The log gets a Working Copies list, one line per working copy with its address and type id, and a dirty one starts with a filled circle.
  2. The log also gets a Backups list in the same line format.
  3. A list with nothing in it shows as "<none>".
  4. The Output view opens on the Window log channel without taking keyboard focus.
  5. The command is in the Command Palette under Developer and has no menu row or key.

### NEW-A-8 Navigate to the view on the left, right, above or below
- Group: WB-3
- Tag: both
- Evidence: source
- Boundary: no
- What: Four commands move keyboard focus from the part that has it to the visible part on the chosen side, such as editor to side bar or panel to editor.
- Commands: workbench.action.navigateLeft; workbench.action.navigateRight; workbench.action.navigateUp; workbench.action.navigateDown
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\navigationActions.ts:23-240; C bundle (class holding the neighbor logic for workbench.parts.editor, panel, sidebar and auxiliarybar, and all four ids, 2 hits each). F6 and Shift+F6 (focusNextPart, focusPreviousPart) are a different behavior in the same file and are already named in SS-1.
- Pieces:
  1. "Navigate to the View on the Left", "Navigate to the View on the Right", "Navigate to the View Above" and "Navigate to the View Below" move keyboard focus to the part that lies in that direction from the focused part.
  2. From the editor area, the command first moves to the editor group in that direction, and only leaves the editor area when no group lies that way.
  3. Coming back into the editor area from the side bar or panel focuses the active group when no group lies in between, otherwise the first group when moving right and the last group for the other directions.
  4. Landing on the side bar, panel or secondary side bar focuses the view container that is open there, and a hidden part is never a target.
  5. The commands are in the Command Palette under View and have no default keys and no menu rows.

### NEW-A-9 Help > View License
- Group: WB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: The Help menu has a View License row that opens the product license page in the web browser, and the same command is in the Command Palette.
- Commands: workbench.action.openLicenseUrl
- Menus: MenubarHelpMenu (group 4_legal, order 1); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\helpActions.ts:239-278, 398-400; C bundle (class with id workbench.action.openLicenseUrl, title View License, mnemonic View &&License, menu MenubarHelpMenu group 4_legal order 1); C\product.json (licenseUrl is https://cursor.com/license.txt)
- Pieces:
  1. Choosing Help > View License opens the license page in the default web browser.
  2. Running "Help: View License" from the Command Palette does the same.
  3. When the window has a display language set, the page address gets a `lang=` query added with that language.
  4. The menu row sits in the legal group of the Help menu and has the mnemonic L.
  5. The command exists only when the product has a license address; Cursor sets it to https://cursor.com/license.txt.

### NEW-A-10 Open the workspace configuration file
- Group: WB-10
- Tag: both
- Evidence: source
- Boundary: no
- What: When a workspace file is open, "Workspaces: Open Workspace Configuration File" opens that file in an editor tab.
- Commands: workbench.action.openWorkspaceConfigFile
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (the command, label and workspace-only condition are present; the stock Agents window condition is not)
- Source: V\src\vs\workbench\browser\actions\workspaceActions.ts:188-210; C bundle (class with id workbench.action.openWorkspaceConfigFile, category Workspaces, precondition is only the workspace state)
- Pieces:
  1. Running the command opens the workspace file of the current multi-folder workspace as a pinned editor tab.
  2. The command is available only when the window is on a workspace file, so it does not show for a single folder or an empty window.
  3. In stock the command is also hidden in the Agents window; the Cursor bundle has no such condition. [stock only]
  4. It is reached only from the Command Palette under the Workspaces category, with no menu row and no key.

### NEW-A-11 Developer: Remove Large Storage Database Entries
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: A picker lists the biggest entries in the app's storage database, lets the person tick some, and removes them after a warning box.
- Commands: workbench.action.removeLargeStorageDatabaseEntries
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:480-590; C bundle (class with id workbench.action.removeLargeStorageDatabaseEntries and SIZE_THRESHOLD 1024*16)
- Pieces:
  1. The command opens a multi-select picker with the placeholder "Select large entries to remove from storage", one row per storage key with its size, biggest first.
  2. Each row's detail says "Scope: Global, Profile or Workspace, Target: Machine or User".
  3. In a built app only entries over 16 KB are listed, while a development build lists every key.
  4. For the default profile the Profile scope is skipped so no key shows twice.
  5. With nothing to remove, the picker's description says "There are no large storage entries to remove."
  6. The picker has a custom Remove button instead of OK and no select-all box.
  7. Pressing Remove opens a warning box "Do you want to remove the selected storage entries from the database?" that lists the keys and warns "This action is irreversible and may result in data loss!"
  8. After confirming, the entries are deleted and the storage database is compacted for each affected scope.

### NEW-A-12 Developer: Policy Diagnostics
- Group: WB-9
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run the command and a Markdown report on account, managed-settings and policy state opens in an untitled editor tab. Stock only.
- Commands: workbench.action.showPolicyDiagnostics
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:708-1182; C bundle (0 hits for workbench.action.showPolicyDiagnostics)
- Pieces:
  1. Running the command opens a pinned untitled Markdown editor headed "VS Code Policy Diagnostics" with a warning that the file may hold sensitive information. [stock only]
  2. The report has System Information with the time generated, product name and version, and commit. [stock only]
  3. Account Information shows a summary and a property table, with the session id and analytics tracking id hidden as ***. [stock only]
  4. Account Policy Gate shows its state and reason with a legend of the possible states. [stock only]
  5. Managed Settings lists the native MDM, GitHub server, and managed-settings.json sources, how each key resolves, and the effective values. [stock only]
  6. Policy-Controlled Settings is split into Applied Policy and Non-applied Policy. [stock only]
  7. Authentication Information lists providers and session details, and any lookup that fails is written into the report as an error line. [stock only]
  8. The command is in the Command Palette under Developer and has no menu row or key. [stock only]

### NEW-A-13 Developer: Sync Account Policy
- Group: WB-9
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run the command to force a refresh of the signed-in account's policy data, then a box says it worked or failed. Stock only.
- Commands: workbench.action.syncAccountPolicy
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:1185-1213; C bundle (0 hits for workbench.action.syncAccountPolicy)
- Pieces:
  1. Running the command forces a refresh of the default account and its policy data. [stock only]
  2. On success an info box says "Account policy has been synced." [stock only]
  3. On failure an error box says "Failed to sync account policy." with the error message as detail. [stock only]
  4. The command is in the Command Palette under Developer and has no menu row or key. [stock only]

### NEW-A-14 Developer: Toggle Screencast Mode
- Group: WB-9
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the command to show a colored circle where the mouse is pressed and a large overlay of the keys and commands being used, for demos and videos; run it again to stop.
- Commands: workbench.action.toggleScreencastMode
- Menus: CommandPalette
- Keys: none found
- Settings: screencastMode.verticalOffset; screencastMode.fontSize; screencastMode.keyboardOptions; screencastMode.keyboardOverlayTimeout; screencastMode.mouseIndicatorColor; screencastMode.mouseIndicatorSize
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:121-415, 1230-1315; C bundle (id workbench.action.toggleScreencastMode, the six screencastMode settings with NLS 3029-3039, and screencast-keyboard and screencast-mouse rules in workbench.desktop.main.css)
- Pieces:
  1. Running the command turns Screencast Mode on, and running it again turns it off and removes the overlays.
  2. While the mouse button is down a colored circle sits over the pointer and follows it, shrinking a little while the mouse is dragged.
  3. Each key press adds a key label to a large overlay near the bottom of the window, with the command name in front of it when the keys match a shortcut.
  4. The overlay clears when a modifier key, Backspace, Escape or an arrow key starts a new entry, when more than about 20 keys are shown, or after the timeout.
  5. It works across all open windows, including separate editor windows.
  6. Setting `screencastMode.verticalOffset` sets the overlay height from the bottom as a percentage (0 to 90, default 20) and `screencastMode.fontSize` sets its font size (20 to 100, default 56).
  7. Setting `screencastMode.keyboardOptions` turns on or off raw keys, keyboard shortcuts, command names, command group names, and single editor cursor moves, with only group names off by default.
  8. Setting `screencastMode.keyboardOverlayTimeout` sets how long the overlay stays (500 to 5000 ms, default 800).
  9. Settings `screencastMode.mouseIndicatorColor` (default #FF0000) and `screencastMode.mouseIndicatorSize` (20 to 100 px, default 20) change the circle, and all six settings apply live while it is on.

### NEW-A-15 Developer: Capture and Send Debugging Data
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: Run the command to pick what to collect, describe what went wrong, and have the app capture a debugging bundle and send it for support. Boundary yes: the picker can attach agent transcripts and chat or prompt data.
- Commands: workbench.action.developer.captureAndSendDebuggingData
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with id workbench.action.developer.captureAndSendDebuggingData, title Capture and Send Debugging Data, NLS 11418-11441 in out/nls.messages.json; the picker ids heapSnapshot, userDirectory, logs, featureFlagsAndExtensionSummary, processDiagnostics, agentTranscripts and agentHostHeapSnapshots)
- Pieces:
  1. Running the command opens a multi-select picker titled "Capture and Send Debugging Data" with the placeholder "Select debugging data to collect. Press Enter to continue.", and the picker stays open when focus moves away. [cursor only]
  2. The picker rows are Renderer heap snapshot, User database, Logs, Feature flags and extension summary, Process diagnostics, and Agent transcripts (last 7 days), each with a one-line detail. [cursor only]
  3. Logs, Feature flags and extension summary, and Process diagnostics start ticked, and the rest start unticked. [cursor only]
  4. The details for User database and Agent transcripts warn that chat and prompt data, which may include attached code, is uploaded even in privacy mode. [cursor only]
  5. An extra row, Agent host heap snapshots, shows only when the agent host saved heap snapshots in the last day. [cursor only]
  6. After Enter, an input box asks "What went wrong? (optional)", and closing it stops the capture. [cursor only]
  7. A progress notification reads "Capturing and sending debugging data..." or, when there is no upload address, "Capturing debugging data...". [cursor only]
  8. The end shows one message: "Debugging data was sent successfully.", or "Debugging data saved locally." with a Reveal in File Explorer button, or "Failed to capture or send debugging data." with the error; which of the first two shows was not traced. [cursor only]
  9. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

### NEW-A-16 Developer: Capture Performance Profile and Show Performance Profiling Status
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: One command captures a CPU and memory profile of the current window after a confirmation, and a second command shows how many profiles are running or scheduled and whether privacy mode forbids profiling.
- Commands: workbench.action.developer.captureProfile; workbench.action.developer.showProfilingStatus
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (classes with ids workbench.action.developer.captureProfile and workbench.action.developer.showProfilingStatus; NLS 5355-5365 in out/nls.messages.json; log prefix ContinuousProfilingContribution)
- Pieces:
  1. Running "Developer: Capture Performance Profile" first opens a confirm box "Capture Performance Profile" saying a CPU and allocations profile of the current window will be created and uploaded securely for analysis, with Continue and Cancel. [cursor only]
  2. After Continue, a progress notification reads "Capturing performance profile...". [cursor only]
  3. A successful start shows an info message "Performance profile started." with a Profile ID. [cursor only]
  4. A failed start shows "Failed to start performance profile." or "Error starting performance profile: {message}". [cursor only]
  5. "Developer: Show Performance Profiling Status" shows one info message: "Performance Profiling Status: Privacy mode: enabled (profiling forbidden) or disabled (profiling allowed), Active profiles: n, Scheduled profiles: n". [cursor only]
  6. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

### NEW-A-17 Developer: Toggle Watch for Crashes and Force Renderer OOM Crash
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: One command turns crash watching on or off, and a second command crashes the window on purpose so the watcher can be tested.
- Commands: workbench.action.developer.toggleWatchForCrashes; workbench.action.developer.forceRendererOOM
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (classes with ids workbench.action.developer.toggleWatchForCrashes and workbench.action.developer.forceRendererOOM; NLS 11442-11454 in out/nls.messages.json)
- Pieces:
  1. Turning crash watching on shows a confirm box "Toggle Watch for Crashes" warning that the editor may get slower and debugging data is sent automatically if a crash happens, with Continue and Cancel. [cursor only]
  2. After the toggle, an info message says "Watching for crashes." or "Stopped watching for crashes."; turning it off asks for no confirmation. [cursor only]
  3. "Developer: Force Renderer OOM Crash" opens a confirm box saying it will use memory until the renderer crashes, the window will freeze and then reload, and it is meant for testing the watcher, with Crash Now and Cancel. [cursor only]
  4. After Crash Now the window stops responding, crashes, and reloads. [cursor only]
  5. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

### NEW-A-18 Developer: Dump Feature Gates
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run the command to save the current feature gates to a JSON file in the home folder and get a notification that can show the file in the file manager.
- Commands: workbench.action.dumpFeatureGates
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with id workbench.action.dumpFeatureGates; NLS 3027, 3028 and 3053 in out/nls.messages.json)
- Pieces:
  1. Running the command writes a file named feature-gates plus a timestamp with a .json ending into the .cursor/feature-gates folder of the home folder, and creates the folder when it is missing. [cursor only]
  2. An info notification says "Feature gates dumped to" followed by the file path. [cursor only]
  3. The notification has a Reveal in File Explorer button that shows the file in the operating system's file manager. [cursor only]
  4. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

### NEW-A-19 Developer: Log Data Directories
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run the command to log every folder the app uses for data and to show the list in a notification with a copy button.
- Commands: workbench.action.logDataDirectories
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with id workbench.action.logDataDirectories, title Log Data Directories, NLS 11968)
- Pieces:
  1. Running the command writes a banner and one line for each of 21 folders and files, such as User Data Directory, Logs Home, Extensions Path, Cache Home and Argv Resource, to the log. [cursor only]
  2. An info notification titled "VS Code Data Directories:" lists the same paths and ends with "To reset VS Code data, delete the User Data Directory and restart VS Code." [cursor only]
  3. The notification has a Copy to Clipboard button, and after it runs the message "Data directories copied to clipboard" or "Failed to copy to clipboard" shows. [cursor only]
  4. The command is in the Command Palette under Developer and has no menu row or key. [cursor only]

### NEW-A-20 Developer: Reload With HMR and Restart in Temporary Profile
- Group: WB-9
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Two Developer commands restart the window in a special way: one reloads with hot module replacement turned on, the other relaunches the app on a throwaway user data folder.
- Commands: workbench.action.reloadWindowWithHMR; workbench.action.restartInTemporaryProfile
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (classes with ids workbench.action.reloadWindowWithHMR, calling reload with hmr true, and workbench.action.restartInTemporaryProfile, calling relaunch with a user data folder argument; NLS 11969 and 11973)
- Pieces:
  1. Running "Developer: Reload With HMR" reloads the window with hot module replacement turned on, with no confirmation. [cursor only]
  2. Running "Developer: Restart in Temporary Profile" relaunches the whole app with no confirmation. [cursor only]
  3. The relaunched app uses a new empty user data folder named vscode-temp-profile plus a timestamp in the temp folder, so it starts with default settings and no saved window state. [cursor only]
  4. The normal user data folder is left untouched by the restart. [cursor only]
  5. Both commands are in the Command Palette under Developer and have no menu rows or keys. [cursor only]

### NEW-A-21 Install or remove the cursor and code shell commands
- Group: WB-10
- Tag: both
- Evidence: source
- Boundary: no
- What: Four Shell Command entries put the cursor or code command on the path or take it off, and Cursor also shows a one-time notification offering to do it.
- Commands: workbench.action.installCommandLine; workbench.action.uninstallCommandLine; workbench.action.replaceCommandLine; workbench.action.restoreCommandLine
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: changed (stock has only install and uninstall of its own command, macOS only; Cursor keeps those, reworded, and adds the two code commands and the prompt)
- Source: V\src\vs\workbench\electron-browser\actions\installActions.ts:15-80; V\src\vs\workbench\electron-browser\desktop.contribution.ts:65-69; C bundle (four classes in category Shell Command, NLS 12002-12010; registration Li&&(Xe(y4w),Xe(w4w)),Xe(S4w),Xe(k4w) where Li is the macOS flag; prompt code reading cmdLineHookState with buttons Install 'code', Install 'cursor' and Never show again)
- Pieces:
  1. "Shell Command: Install 'cursor' command" puts the cursor command on the path and shows "Shell command 'cursor' successfully installed." or the error text; it exists on macOS only, as in stock where the label is "Install '{0}' command in PATH".
  2. "Shell Command: Uninstall 'cursor' command from PATH" removes it and shows "Shell command 'cursor' successfully uninstalled from PATH."; cancelling shows nothing.
  3. "Shell Command: Install 'code' command" makes the code command open Cursor and shows "Successfully installed Cursor in the 'code' command." [cursor only]
  4. "Shell Command: Uninstall 'code' command from PATH" unlinks Cursor from the code command and shows "Successfully unlinked Cursor from the 'code' command." [cursor only]
  5. The two code commands are registered on every platform, while the two cursor commands are registered on macOS only. [cursor only]
  6. A sticky notification asks "Want to launch Cursor from the command line? You can add the `code` or `cursor` commands now, or later with the Command Palette key and typing 'install code command'." with buttons Install 'code', Install 'cursor' and Never show again. [cursor only]
  7. The notification appears about 5 minutes after start, on Windows and macOS only, at most twice, and not again within 5 days of showing; pressing any of its three buttons stops it for good. [cursor only]
  8. All four entries are in the Command Palette under Shell Command and have no menu rows or keys.
