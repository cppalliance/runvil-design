Bucket: B. Date: 2026-10-25. Pass: 4.
MAP | developer.showCurrentFileAsReleaseNotes | ED-9.26 | command | Developer command that opens the current file as the release notes page; ED-9.26 names it as update.showCurrentReleaseNotesFromCurrentFile but the real id is this one (V update/common/update.ts); missing in Cursor
NEW | extension.bisect.next | NEW-B-10 | command | Continue Extension Bisect, run from the "I can reproduce" and "I can't reproduce" notification buttons or the palette; Help category; present in Cursor
NEW | extension.bisect.start | NEW-B-10 | command | Start Extension Bisect from the palette or the Extensions view title menu; opens a confirm box, then reloads
NEW | extension.bisect.stop | NEW-B-10 | command | Stop Extension Bisect from the Stop Bisect button or the palette; resets and reloads
MAP | iconSelectBox.focusDown | ED-8.14 | command | Down arrow in the icon grid of the Profiles editor icon picker
MAP | iconSelectBox.focusNext | ED-8.14 | command | Right arrow in the profile icon grid (only when the filter box is empty or not focused)
MAP | iconSelectBox.focusPrevious | ED-8.14 | command | Left arrow in the profile icon grid, same condition as focusNext
MAP | iconSelectBox.focusUp | ED-8.14 | command | Up arrow in the profile icon grid
MAP | iconSelectBox.selectFocused | ED-8.14 | command | Enter picks the focused icon in the Profiles editor icon grid
NEW | list.resizeColumn | NEW-B-18 | command | List: Resize Column in the palette works on any focused table (Problems table, Keyboard Shortcuts, Profiles editor and others), so it is not a Problems-only feature; present in Cursor
NEW | menu.resetHiddenStates | NEW-B-20 | command | View: Reset All Menus restores toolbar actions that were hidden with the right-click Hide row; present in Cursor
MAP | output.editor.contrib.filterController | PN-5.5 | command | Not a command: the editor contribution id of the Output filter that hides non-matching lines
NEW | vscode.openIssueReporter | NEW-B-12 | command | API alias that opens the issue reporter; in Cursor it opens the Send Feedback pane and ignores the legacy arguments; in stock it opens the reporter of NEW-B-11
EXCL | workbench.action.chat.upgradePlan | command | agent | Copilot plan upgrade command; the V line in the work file is a false hit, the real registration is in chat/browser/chatSetup; not in the Cursor bundle
NEW | workbench.action.configureRuntimeArguments | NEW-B-21 | command | Preferences: Configure Runtime Arguments opens argv.json in a pinned tab; present in Cursor
EXCL | workbench.action.download | command | web-only | Precondition IsWebContext and shown only on the web status bar remote menu; Cursor has the id but the desktop app never shows it
NEW | workbench.action.focusWindow | NEW-B-13 | command | Focus Window brings the current window to the front; missing in Cursor
NEW | workbench.action.installCommandLine | NEW-B-14 | command | Shell Command: Install command in PATH, macOS only in both; present in Cursor
NEW | workbench.action.issueReporter.captureScreenshot | NEW-B-11 | command | Ctrl+Shift+S while the issue reporter wizard tab is open captures a screenshot; stock only
NEW | workbench.action.issueReporter.toggleRecording | NEW-B-11 | command | Ctrl+Shift+R while the issue reporter wizard tab is open starts or stops a screen recording; stock only
MAP | workbench.action.localHistory.revealInOS | SB-7.14 | command | Reveal in File Explorer row on a Local History entry in the Timeline (file scheme only); not named by SB-7.14 but it opens the saved copy's location
EXCL | workbench.action.onboarding.resetShownState | command | developer-only | Developer: Reset Onboarding Shown State resets the stock spotlight tour engine, which has no tours registered in 1.128 (only tests register scenarios); missing in Cursor
NEW | workbench.action.openProcessExplorer | NEW-B-8 | command | Help menu row and Developer palette entry; opens the Process Explorer; present in Cursor as a separate native window
EXCL | workbench.action.previewColorTheme | command | extension-system | Called with an extension argument from the extension page to preview a theme; the live preview inside the Color Theme picker is a different path
NEW | workbench.action.reloadWindowWithExtensionsDisabled | NEW-B-2 | command | Developer: Reload with Extensions Disabled; present in Cursor
NEW | workbench.action.reportPerformanceIssueUsingReporter | NEW-B-12 | command | Stock Help: Report Performance Issue; in Cursor a hidden alias that opens the Send Feedback pane
NEW | workbench.action.revealUserDataFolder | NEW-B-3 | command | Developer: Reveal User Data Folder; present in Cursor
NEW | workbench.action.showContentTracing | NEW-B-5 | command | Developer: Show Content Tracing opens the Chromium tracing page in a window; missing in Cursor
NEW | workbench.action.showGPUInfo | NEW-B-4 | command | Developer: Show GPU Info opens the Chromium GPU page in a window; present in Cursor
NEW | workbench.action.showHover | NEW-B-19 | command | Ctrl+K Ctrl+I shows and focuses the hover of whatever UI element has focus (not the editor, which has editor.action.showHover); present in Cursor with an editor-not-focused condition
NEW | workbench.action.switchToMainWindow | NEW-B-13 | command | Switch to Main Window from an auxiliary window; missing in Cursor
NEW | workbench.action.toggleDevTools | NEW-B-1 | command | Help menu row, palette entry and Ctrl+Shift+I; present in Cursor with the key always on
NEW | workbench.action.troubleshootIssue.start | NEW-B-9 | command | Help: Troubleshoot Issue; present in Cursor
NEW | workbench.action.troubleshootIssue.stop | NEW-B-9 | command | Help: Stop Troubleshoot Issue, shown only while troubleshooting is active
NEW | workbench.action.uninstallCommandLine | NEW-B-14 | command | Shell Command: Uninstall command from PATH, macOS only
EXCL | workbench.actions.accounts.signIn | command | agent | Sign In for the Copilot default account (chat entitlement); not in the Cursor bundle
NEW | workbench.editor.processExplorer | NEW-B-8 | command | Not a command: the editor id of the stock Process Explorer tab that lives in an always-on-top auxiliary window; Cursor has no such editor
EXCL | workbench.extensions.action.openInstalledWebExtensionsResource | command | web-only | Registered only when isWeb, precondition IsWebContext
MAP | workbench.profiles.actions.help | ED-8.14 | command | Palette entry "Profiles" in the Help category that opens the Profiles documentation page; present in Cursor
EXCL | workbench.userDataSync.actions.downloadSyncActivity | command | other | Settings Sync is out of scope (catalog decision 9); Developer category and missing in Cursor
NEW | workspaces.dontPromptToOpen | NEW-B-15 | command | Not a command: the Don't Show Again id of the notification that offers to open a .code-workspace file found in the folder; present in Cursor
NEW | aiServerConfigService.forceRefresh | NEW-B-6 | command | Developer: Force Refresh Server Config in the palette; shows an info notification with a request id; Cursor only
EXCL | airlock.openLog | command | remote | Action id on the failure notice of Cursor's airlock connection (Dev Containers based remote connect)
EXCL | airlock.reloadWindow | command | remote | Second action id on the same airlock failure notice
MAP | authentication.command.anysphere.signOut | AB-4.2 | command | Palette "Sign Out of GitHub" removes the GitHub sessions without the confirm box; a second entry point for signing out of an account
NEW | cpuMonitor.triggerManualSnapshot | NEW-B-7 | command | Called by Developer: Capture and Upload Process Snapshot, which saves a heap snapshot and uploads it; Cursor only
EXCL | deeplink.command.create | command | agent | Creates a Cursor custom command file from a cursor:// link; custom commands are agent prompts
EXCL | deeplink.prReview.open | command | agent | Opens a GitHub pull request link in Review Changes pull request mode, which ED-6 left out as background agent and Bugbot flow
EXCL | deeplink.prompt.prefill | command | agent | Opens the Composer with a prefilled prompt from a cursor:// link
EXCL | deeplink.routeToWorkspaceName | command | internal | Helper that focuses the window of a named workspace and re-opens a deep link in it; no UI of its own
EXCL | deeplink.rule.create | command | agent | Creates a Cursor rule file from a cursor:// link; rules configure the agent
EXCL | dev.azure.com | command | other | Not a command: the Azure DevOps host name in remote URL parsing
EXCL | developer.analytics.clearCollectedEvents | command | developer-only | Not in the palette (f1 false) and needs the isDevelopment key; test API for analytics events
EXCL | developer.analytics.getCollectedEvents | command | developer-only | Same as above
EXCL | developer.analytics.getFilteredEvents | command | developer-only | Same as above
EXCL | developer.analytics.startEventCollection | command | developer-only | Same as above
EXCL | developer.analytics.stopEventCollection | command | developer-only | Same as above
MAP | diffreview.close | ED-6.6 | command | Close button of the Accessible Diff Viewer, which ED-6.6 covers
EXCL | eamodio.gitlens | command | other | Not a command: a GitLens extension id in the list of AI autocomplete extensions that Cursor skips or disables when importing from VS Code
EXCL | extensionService.versionMismatchCrash | command | extension-system | Notification id for an extension host crash with exit code 55 (version mismatch) that offers a restart
EXCL | exthost.action.toggleExtHostProfiler | command | extension-system | Developer: Start Network and IPC Profiler for the extension host
NEW | fileWorkingCopy.overwrite | NEW-B-17 | command | Overwrite button in the "content of the file is newer" save failure notification
NEW | fileWorkingCopy.retry | NEW-B-17 | command | Retry button in the generic save failure notification
NEW | fileWorkingCopy.revert | NEW-B-17 | command | Revert button in both save failure notifications
NEW | fileWorkingCopy.saveAs | NEW-B-17 | command | Save As button in the save failure notification
NEW | fileWorkingCopy.saveElevated | NEW-B-17 | command | Retry as Admin, Retry as Sudo, Overwrite as Admin or Overwrite as Sudo button
NEW | fileWorkingCopy.unlock | NEW-B-17 | command | Overwrite button when the file is read-only
MAP | inlineDiff.tooManyChangesWarning | ED-6.28 | command | Not a command: the Don't Show Again id of the "inline diffs suppressed" notification that ED-6.28 already describes with its Disable Protection button
NEW | join.browserViews | NEW-B-16 | command | Not a command: shutdown step label "Cleaning up browser views"; Cursor only
NEW | join.closeStorage | NEW-B-16 | command | Not a command: shutdown step label "Saving UI state"
NEW | join.disconnectRemote | NEW-B-16 | command | Not a command: shutdown step label "Disconnect Remote Agent"; only with a remote window
NEW | join.extensionDevelopment | NEW-B-16 | command | Not a command: shutdown step label "Terminating extension debug session"
NEW | join.fileWorkingCopyManager | NEW-B-16 | command | Not a command: shutdown step label "Saving working copies"
NEW | join.stopExtensionHosts | NEW-B-16 | command | Not a command: shutdown step label "Stopping Extension Hosts"
NEW | join.textFiles | NEW-B-16 | command | Not a command: shutdown step label "Saving text files"
NEW | join.windowAnalytics | NEW-B-16 | command | Not a command: shutdown step label "Flushing analytics events"; Cursor only
NEW | join.workingCopyBackups | NEW-B-16 | command | Not a command: shutdown step label "Backup working copies"
NEW | join.workingCopyHistory | NEW-B-16 | command | Not a command: shutdown step label "Saving local history"
NEW | langugage.update.donotask | NEW-B-22 | command | Not a command: Don't Show Again id (note the typo in the id) of the prompt to switch the display language after a language pack is installed
MAP | mainThreadWebview-markdown.preview | ED-1.23 | command | Not a command: the Markdown preview editor id that Cursor adds to its list of editors that lock a new group automatically
EXCL | notebook.developer.addCellOverlays | command | developer-only | Toggle Notebook Cell Overlays needs the isDevelopment key; ED-7 already lists the sibling notebook developer commands as not catalogued
EXCL | remote.reconnectGaveUp | command | remote | Notification id "Unable to reconnect to the remote workspace" with a Reconnect Now button
EXCL | remote.tunnelsView.autoForwardNeverShow | command | remote | Don't Show Again id of the forwarded port notification
EXCL | repositoryTracker.getStoredPath | command | internal | Helper command with no UI that returns a stored repository path
EXCL | repositoryTracker.isOrInCurrentWorkspace | command | internal | Helper command with no UI
EXCL | repositoryTracker.removeStoredPath | command | internal | Helper command with no UI
MAP | scm.input.cancelAction | SB-4.14 | command | Cancel button (stop icon) in the Source Control message box toolbar while its action runs, which in Cursor is Generate Commit Message
EXCL | shellExec.sandboxAppArmorSuggestion | command | agent | Don't Show Again id of a terminal sandbox warning for the agent shell tool on Linux
EXCL | shellExec.sandboxKernelTooOld | command | agent | Same sandbox warning for old Linux kernels
EXCL | solidjs.getTotalObservers | command | internal | Registered only in unbuilt or smoke test runs, returns a number for metrics
EXCL | supermaven.supermaven | command | other | Not a command: a Supermaven extension id in the AI autocomplete extension list used by the VS Code import
NEW | testing.debugCurrentFile | NEW-B-23 | command | Debug Tests in Current File from the palette, the editor right-click menu or Ctrl+; Ctrl+F (with Ctrl on the second key); present in Cursor
NEW | urlOpenToast.dontOpen | NEW-B-24 | command | Don't Open button on the "Opening {url} in Cursor Browser" toast; Cursor only
EXCL | vscjava.migrate-java-to-azure | command | other | Not a command: an extension id in the AI autocomplete extension list used by the VS Code import
EXCL | vscode.menubar.separator | command | internal | Marker id for separators in the data sent to the native macOS menu
MAP | workbench.input.interactive | ED-7.37 | command | Not a command: the editor input id of the Interactive Window that ED-7.37 covers
MAP | workbench.panel.testResults | PN-6.18 | command | Not a command: the id of the Test Results panel that PN-6.18 opens
MAP | workench.action.disableEmptyEditorHint | ED-9.24 | command | Not a command: the action id (typo in the bundle) of the "Disable Empty Editor Hint" right-click row that ED-9.24 describes
