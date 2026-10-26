Bucket: A. Date: 2026-10-25. Pass: 4.
MAP | list.collapseAllToFocus | SB-1.23 | command | Tree only, no default key, collapses the focused node and selects it; same list and tree keyboard behavior as SB-1.23
MAP | list.focusAnyDown | SB-1.23 | command | Alt+Down moves focus down without changing the selection; macOS also Ctrl+Alt+N
MAP | list.focusAnyFirst | SB-1.23 | command | Alt+Home jumps focus to the first row without selecting it
MAP | list.focusAnyLast | SB-1.23 | command | Alt+End jumps focus to the last row without selecting it
MAP | list.focusAnyUp | SB-1.23 | command | Alt+Up moves focus up without changing the selection; macOS also Ctrl+Alt+P
MAP | list.focusFirst | SB-1.23 | command | Home key; SB-1.23 already names Home but lists only focusDown and focusUp
MAP | list.focusLast | SB-1.23 | command | End key; same note as list.focusFirst
MAP | list.focusPageDown | SB-1.23 | command | Page Down in a focused list or tree
MAP | list.focusPageUp | SB-1.23 | command | Page Up in a focused list or tree
MAP | list.focusParent | SB-1.23 | command | Tree only, no default key, moves focus to the parent row; no other caller found in V
MAP | list.selectAndPreserveFocus | SB-1.23 | command | No default key, selects the focused row and keeps keyboard focus in the list; no other caller found in V
EXCL | vscode.moveViews | command | internal | Extension API command with object arguments, has a title but no palette entry, menu or key; the picker version is SB-1.14
EXCL | vscode.newWindow | command | internal | Extension API command that forwards to the New Window code with reuseWindow and remoteAuthority options, no UI of its own
EXCL | vscode.removeFromRecentlyOpened | command | internal | Extension API command that takes a path, no UI of its own; the clear list entry is TB-1.16
NEW | widgetNavigation.focusNext | NEW-A-1 | command | Ctrl+Down hops from a filter or search box to its list in Problems, Comments, Testing, Debug Console and the Settings and Keyboard Shortcuts editors
NEW | widgetNavigation.focusPrevious | NEW-A-1 | command | Ctrl+Up hops back from the list to the filter or search box
EXCL | workbench.action.agentSessions.openFile | command | agent | Declared in V under src/vs/sessions (Agents window changes view); no hit in the Cursor main bundle; the work file path is wrong
EXCL | workbench.action.agentToggleSidebarVisibility | command | agent | Declared in V under src/vs/sessions layoutActions (Agents window); no hit in the Cursor main bundle; the work file path is wrong
NEW | workbench.action.blur | NEW-A-2 | command | Remove keyboard focus from focused element; no key, no palette entry, can be bound in Keyboard Shortcuts
NEW | workbench.action.decreaseViewHeight | NEW-A-3 | command | Decrease Editor Height, shrinks the editor area by 60 px; SB-1.4 already holds the width pair and the Current View Size pair
NEW | workbench.action.duplicateWorkspaceInNewWindow | NEW-A-4 | command | Workspaces: Duplicate As Workspace in New Window; DL-3 has Save Workspace As but not this
NEW | workbench.action.increaseViewHeight | NEW-A-3 | command | Increase Editor Height, grows the editor area by 60 px
NEW | workbench.action.inspectContextKeys | NEW-A-5 | command | Developer: Inspect Context Keys; crosshair pick, then values logged to the developer console
NEW | workbench.action.logStorage | NEW-A-6 | command | Developer: Log Storage Database Contents; logs to developer tools and shows an info box
NEW | workbench.action.logWorkingCopies | NEW-A-7 | command | Developer: Log Working Copies; writes to the Window log and opens it in Output
NEW | workbench.action.navigateDown | NEW-A-8 | command | Navigate to the View Below; moves keyboard focus to the part under the focused one
NEW | workbench.action.navigateRight | NEW-A-8 | command | Navigate to the View on the Right
NEW | workbench.action.navigateUp | NEW-A-8 | command | Navigate to the View Above
NEW | workbench.action.openLicenseUrl | NEW-A-9 | command | Help > View License; Cursor keeps this row (licenseUrl is set in its product.json); TB-1.30 covers only the stock rows Cursor removed
MAP | workbench.action.openNewsletterSignupUrl | TB-1.30 | command | Palette-only stock Help-category link, shown only when product.newsletterSignupUrl is set; Cursor has no hit and no URL, so it is a stock-only sibling of the TB-1.30 rows
NEW | workbench.action.openWorkspaceConfigFile | NEW-A-10 | command | Workspaces: Open Workspace Configuration File; only when a workspace file is open
NEW | workbench.action.removeLargeStorageDatabaseEntries | NEW-A-11 | command | Developer: Remove Large Storage Database Entries; picker, then a warning box
EXCL | workbench.action.resetUserData | command | web-only | Registered in web.main.ts for the browser build only; no hit in the Cursor bundle
NEW | workbench.action.showPolicyDiagnostics | NEW-A-12 | command | Developer: Policy Diagnostics; stock only, no hit in the Cursor bundle
EXCL | workbench.action.snapshotTrackedDisposables | command | developer-only | Registered only when product.commit is empty (development builds), in stock and in Cursor
EXCL | workbench.action.startTrackDisposables | command | developer-only | Registered only when product.commit is empty (development builds), in stock and in Cursor
EXCL | workbench.action.stopTrackDisposables | command | developer-only | Registered only when product.commit is empty (development builds); opens a leak report only for developers
NEW | workbench.action.syncAccountPolicy | NEW-A-13 | command | Developer: Sync Account Policy; stock only, no hit in the Cursor bundle
NEW | workbench.action.toggleScreencastMode | NEW-A-14 | command | Developer: Toggle Screencast Mode; the six screencastMode settings belong to the same feature
EXCL | workbench.experimental.requestHidDevice | command | web-only | Extension API command in browser window.ts for web only; no hit in the Cursor bundle
EXCL | workbench.experimental.requestSerialPort | command | web-only | Extension API command in browser window.ts for web only; no hit in the Cursor bundle
EXCL | workbench.experimental.requestUsbDevice | command | web-only | Extension API command in browser window.ts for web only; no hit in the Cursor bundle
EXCL | workbench.action.automationsEditor.save | command | agent | Save Automation, Ctrl+S inside the Cursor Automations editor, which the catalog left out as an agent surface (decision 7)
EXCL | workbench.action.clearStatsigCache | command | developer-only | Developer command enabled only when the isDevelopment key is true and registered only in builds without a commit
MAP | workbench.action.configurePreferredDropAction | ED-2.29 | command | Not Cursor only: the same id is in V src/vs/workbench/contrib/dropOrPasteInto/browser/commands.ts; the drop widget action that opens the drop preferences setting in settings JSON
MAP | workbench.action.configurePreferredPasteAction | ED-2.29 | command | Same file as the drop one; the paste widget action that opens the paste preferences setting in settings JSON
EXCL | workbench.action.devAutoLoginFakeForTesting | command | developer-only | Dev Auto Login with fake tokens; enabled only when isDevelopment is true
EXCL | workbench.action.devBatchSetApplicationStorageForTesting | command | internal | Test hook, no palette entry, throws unless the smoke test driver flag is on
EXCL | workbench.action.devDelayNextSmartModeClassifier | command | agent | Delays the Auto-review (agent tool approval) check; also needs a local backend
EXCL | workbench.action.devForceNextSmartModeClassifierBlock | command | agent | Forces the next Auto-review check to block; also needs a local backend
EXCL | workbench.action.devGetApplicationStorageForTesting | command | internal | Test hook, no palette entry, smoke test driver only
EXCL | workbench.action.devGetAutorunSettingForTesting | command | internal | Test hook, no palette entry, smoke test driver only
EXCL | workbench.action.devGetConfiguredHookStepsForTesting | command | internal | Plain registered command used by tests, no title or menu
EXCL | workbench.action.devGetLoadedPluginsSummaryForTesting | command | internal | Test hook, no palette entry, smoke test driver only
EXCL | workbench.action.devGetModelConfigForSurfaceForTesting | command | internal | Test hook, no palette entry, smoke test driver only
EXCL | workbench.action.devRefreshTeamAdminSettings | command | internal | Test hook, no palette entry, smoke test driver only
EXCL | workbench.action.devSetAnyrunCluster | command | agent | Picks the cluster for background agents; enabled only when isDevelopment is true
EXCL | workbench.action.devSetApplicationStorageForTesting | command | internal | Test hook, no palette entry, smoke test driver only
EXCL | workbench.action.devWaitForSlashMenuSourcesForTesting | command | internal | Test hook, no palette entry, smoke test driver only
NEW | workbench.action.developer.captureAndSendDebuggingData | NEW-A-15 | command | Developer: Capture and Send Debugging Data; boundary yes because the picker can attach agent transcripts and chat data
NEW | workbench.action.developer.captureProfile | NEW-A-16 | command | Developer: Capture Performance Profile; confirm box, progress, result message
NEW | workbench.action.developer.forceRendererOOM | NEW-A-17 | command | Developer: Force Renderer OOM Crash; confirm box, then the window crashes on purpose
NEW | workbench.action.developer.showProfilingStatus | NEW-A-16 | command | Developer: Show Performance Profiling Status; one info message with three counts
NEW | workbench.action.developer.toggleWatchForCrashes | NEW-A-17 | command | Developer: Toggle Watch for Crashes; confirm box on turn on, info message on each toggle
NEW | workbench.action.dumpFeatureGates | NEW-A-18 | command | Developer: Dump Feature Gates; writes a JSON file and offers Reveal in File Explorer
EXCL | workbench.action.exportNotepads | command | agent | Developer: Export Notepads to Files; shown only when legacy chat Notepads exist in storage
EXCL | workbench.action.extensionHostProfiler.stopAndSave | command | extension-system | Plain command of the extension host profile service; stock code for it is under contrib/extensions
EXCL | workbench.action.hangRendererThread | command | developer-only | Freezes the window for 30 s; enabled only when isDevelopment is true
EXCL | workbench.action.injectCursorSocketFailure | command | developer-only | Enabled only when isDevelopment or isDevUser is true; kills the agent socket connections for testing
NEW | workbench.action.logDataDirectories | NEW-A-19 | command | Developer: Log Data Directories; logs 21 paths and shows a box with Copy to Clipboard
MAP | workbench.action.newCursorRule | ED-8.65 | command | File: New Cursor Rule; second entry point for creating a Rules item; opens the create-rule chat flow, so boundary yes like ED-8.65
MAP | workbench.action.newProjectCursorCommand | ED-8.65 | command | File: New Project Cursor Command; asks for a name and creates a command file in the first workspace folder's .cursor/commands
MAP | workbench.action.newUserCursorCommand | ED-8.65 | command | File: New User Cursor Command; same flow but the file goes in the home .cursor/commands folder
NEW | workbench.action.reloadWindowWithHMR | NEW-A-20 | command | Developer: Reload With HMR; reloads the window with hot module replacement turned on
NEW | workbench.action.replaceCommandLine | NEW-A-21 | command | Shell Command: Install 'code' command; points the code command at Cursor
NEW | workbench.action.restartInTemporaryProfile | NEW-A-20 | command | Developer: Restart in Temporary Profile; relaunches the app on a throwaway user data folder, grouped with Reload With HMR
NEW | workbench.action.restoreCommandLine | NEW-A-21 | command | Shell Command: Uninstall 'code' command from PATH; unlinks Cursor from the code command
EXCL | workbench.action.runCursorWorktreeCleanupDev | command | developer-only | Run Cursor Worktree Cleanup (Dev); enabled only when isDevelopment is true
EXCL | workbench.action.simulatePermanentRemoteDisconnect | command | remote | Connection: Simulate Permanent Disconnect; enabled only for development or dev users
EXCL | workbench.action.simulateRemoteConnectionDrop | command | remote | Connection: Simulate Connection Drop; enabled only for development or dev users
EXCL | workbench.action.terminal.addTooltip | command | internal | No palette entry; helper for a terminal hint card whose two switches are false in the bundle, so nothing shows
EXCL | workbench.action.terminal.debugCheckExecutionServiceVersion | command | agent | Debug: Check Terminal Execution Service Version; shows version v2 or v3 of the service the agent uses to run terminal commands
EXCL | workbench.parts.activitybar | command | not-ui | Not a command; this is the part id string from the Parts enum used as an argument; AB-1 to AB-6 describe the part
EXCL | workbench.parts.auxiliarybar | command | not-ui | Not a command; part id string; AX-1 and AX-2 describe the part
EXCL | workbench.parts.editor | command | not-ui | Not a command; part id string; ED-1 describes the editor area
EXCL | workbench.parts.panel | command | not-ui | Not a command; part id string; PN-1 describes the panel
EXCL | workbench.parts.statusbar | command | not-ui | Not a command; part id string; SS-1 describes the status bar
EXCL | workbench.parts.titlebar | command | not-ui | Not a command; part id string; TB-1 to TB-5 describe the title bar
