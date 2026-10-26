Group: WB-8 Developer commands. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 25.

### WB-8.1 Developer: Inspect Context Keys
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

### WB-8.2 Developer: Log Storage Database Contents
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

### WB-8.3 Developer: Log Working Copies
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

### WB-8.4 Developer: Remove Large Storage Database Entries
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

### WB-8.5 Developer: Policy Diagnostics
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

### WB-8.6 Developer: Sync Account Policy
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

### WB-8.7 Developer: Toggle Screencast Mode
- Tag: both
- Evidence: source
- Boundary: no
- What: Run the command to show a colored circle where the mouse is pressed and a large overlay of the keys and commands being used, for demos and videos; run it again to stop.
- Commands: workbench.action.toggleScreencastMode
- Menus: CommandPalette; CommandPalette (category Developer)
- Keys: none found
- Settings: screencastMode.verticalOffset; screencastMode.fontSize; screencastMode.keyboardOptions; screencastMode.keyboardOverlayTimeout; screencastMode.mouseIndicatorColor; screencastMode.mouseIndicatorSize
- Cursor: present
- Source: V\src\vs\workbench\browser\actions\developerActions.ts:121-415, 1230-1315; C bundle (id workbench.action.toggleScreencastMode, the six screencastMode settings with NLS 3029-3039, and screencast-keyboard and screencast-mouse rules in workbench.desktop.main.css); V\src\vs\workbench\browser\actions\developerActions.ts (lines 120-395 and 1236-1316); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 157-158); C bundle (id workbench.action.toggleScreencastMode and the six settings)

### WB-8.8 Developer: Capture and Send Debugging Data
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

### WB-8.9 Developer: Capture Performance Profile and Show Performance Profiling Status
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

### WB-8.10 Developer: Toggle Watch for Crashes and Force Renderer OOM Crash
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

### WB-8.11 Developer: Dump Feature Gates
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

### WB-8.12 Developer: Log Data Directories
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

### WB-8.13 Developer: Reload With HMR and Restart in Temporary Profile
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

### WB-8.14 Toggle Developer Tools
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Toggle Developer Tools to open or close the Chromium developer tools of the window you are in, from the Help menu, the Command Palette or a key.
- Commands: workbench.action.toggleDevTools
- Menus: MenubarHelpMenu (group 5_tools, order 1); CommandPalette (Developer category)
- Keys: Ctrl+Shift+I (Cmd+Option+I on macOS); stock binds it only when the `isDevelopment` context is true, Cursor binds it with no `when` at weight 250
- Settings: none found
- Cursor: changed - the command, the Help row and the key are all present, and the key has no development-build condition
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:23-50; V\src\vs\workbench\electron-browser\desktop.contribution.ts:116; C bundle (id workbench.action.toggleDevTools, nls 11996, menu MenubarHelpMenu group 5_tools order 1, keybinding Ctrl|Cmd+Shift+I with mac Cmd+Alt+I)

### WB-8.15 Reload with extensions disabled
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Reload with Extensions Disabled from the Command Palette to reload the window with every extension turned off, to see whether a problem comes from an extension.
- Commands: workbench.action.reloadWindowWithExtensionsDisabled
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:74-87; V\src\vs\platform\native\electron-main\nativeHostMainService.ts:1089-1110 (reload passes the disable-extensions flag for that reload); C bundle (id workbench.action.reloadWindowWithExtensionsDisabled, nls 11998, category Developer)

### WB-8.16 Reveal User Data Folder
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Reveal User Data Folder from the Command Palette to show the folder where the app keeps its user data in the operating system's file manager.
- Commands: workbench.action.revealUserDataFolder
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:90-106; C bundle (id workbench.action.revealUserDataFolder, nls 11999, category Developer)

### WB-8.17 Show GPU info
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Show GPU Info from the Command Palette to open Chromium's GPU status page in its own window.
- Commands: workbench.action.showGPUInfo
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:109-123; V\src\vs\platform\native\electron-main\nativeHostMainService.ts:1214-1234 (child window on chrome://gpu); C bundle (id workbench.action.showGPUInfo, nls 12000, category Developer)

### WB-8.18 Show content tracing
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run Show Content Tracing from the Command Palette to open Chromium's tracing recorder page in its own window.
- Commands: workbench.action.showContentTracing
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Cursor: missing - 0 hits for the id in the bundle (Cursor keeps `workbench.action.stopTracing` and has its own Start Electron Trace instead)
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:126-141; V\src\vs\platform\native\electron-main\nativeHostMainService.ts:1236-1268 (window on chrome://tracing); C bundle (searched id, 0 hits)

### WB-8.19 Force refresh server config
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run Force Refresh Server Config from the Command Palette to make Cursor fetch its server-side configuration again right now, and see a notification with the result.
- Commands: aiServerConfigService.forceRefresh
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action class with id aiServerConfigService.forceRefresh, title "Force Refresh Server Config", category Developer, f1 true; run calls forceRefreshServerConfig with a request id and rethrow)

### WB-8.20 Capture and upload a process snapshot
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run Capture and Upload Process Snapshot from the Command Palette to save a debugging snapshot of the window's memory use to disk and send it to Cursor.
- Commands: developer.captureAndUploadProcessSnapshot; cpuMonitor.triggerManualSnapshot
- Menus: CommandPalette (Developer category)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with id developer.captureAndUploadProcessSnapshot, nls 5367, which runs the command cpuMonitor.triggerManualSnapshot registered by the memory monitor contribution; methods _captureAndUploadSnapshot and _uploadSavedSnapshot)

### WB-8.21 Toggle sign-in debugging
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Run Developer: Toggle Auth Debugging from the Command Palette to switch sign-in debug logging on or off and open the Cursor Auth Debug output channel.
- Commands: cursorAuth.toggleAuthDebug
- Menus: commandPalette (no category, no condition)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action cursorAuth.toggleAuthDebug, title "Developer: Toggle Auth Debugging", f1 true with no precondition; auth service toggleAuthDebug, getAuthDebugOutputChannelId and channel id cursorAuthDebug; first log line "Auth debugging enabled: AuthId ..., http2Disabled=..."). The sibling palette command cursorAuth.triggerTokenRefresh has no visible result and is excluded.

### WB-8.22 Save, load and open the merge editor state (developer)
- Tag: both
- Evidence: source
- Boundary: no
- What: Developer commands in the Command Palette under "Merge Editor (Dev)" copy the state of the open merge editor as JSON, save it to a folder, or reopen a saved state in a new merge editor.
- Commands: merge.dev.copyContentsJson; merge.dev.saveContentsToFolder; merge.dev.loadContentsFromFolder; merge.dev.openContentsJson; merge.dev.openSelectionInTemporaryMergeEditor
- Menus: CommandPalette (category Merge Editor (Dev); copy and save only while a merge editor is active)
- Keys: none found
- Settings: none found
- Cursor: present - all five ids, the category and the labels (including "Select folder to save to") are in the bundle and in nls. ED-6 handed these commands to the developer group, so they are collected here.
- Source: V\src\vs\workbench\contrib\mergeEditor\browser\commands\devCommands.ts (lines 24-193); V\src\vs\workbench\contrib\mergeEditor\electron-browser\devCommands.ts (lines 20-187); V\src\vs\workbench\contrib\mergeEditor\browser\mergeEditor.contribution.ts (lines 97-99); C bundle (merge.dev.* ids); C out\nls.messages.json (labels and notices)

### WB-8.23 Developer: Apply Update from File
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

### WB-8.24 Developer: Inspect Key Mappings
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Developer: Inspect Key Mappings, or its JSON form, and a new untitled tab opens with a dump of the keyboard layout and key mappings, used to find out why a key does not do what it should.
- Commands: workbench.action.inspectKeyMappings; workbench.action.inspectKeyMappingsJSON
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Cursor: present - both class ids and titles are in the bundle, with the Developer category and the same untitled editor call
- Source: V\src\vs\workbench\contrib\codeEditor\browser\inspectKeybindings.ts (lines 14-54); V\src\vs\workbench\services\keybinding\browser\keybindingService.ts (_dumpDebugInfo, lines 395-401; _dumpDebugInfoJSON, lines 403-409); C bundle (classes with ids workbench.action.inspectKeyMappings and workbench.action.inspectKeyMappingsJSON, NLS 5029 and 5030)

### WB-8.25 Developer: Open URL
- Tag: both
- Evidence: source
- Boundary: no
- What: The person runs Developer: Open URL, types an address in the input box, and the app hands it to its URL service as if the link had been opened from outside.
- Commands: workbench.action.url.openUrl
- Menus: CommandPalette (category Developer)
- Keys: none found
- Settings: none found
- Cursor: changed - the id, the title, the Developer category and the "URL to open" prompt are in the bundle, but Cursor's box starts empty and keeps no last input
- Source: V\src\vs\workbench\contrib\url\browser\url.contribution.ts (OpenUrlAction, lines 27-57); C bundle (class with id workbench.action.url.openUrl, title "Open URL", category Developer, prompt message 11526, no storage read)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
