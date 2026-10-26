Bucket: B. Date: 2026-10-25. Pass: 4.

### NEW-B-1 Toggle Developer Tools
- Group: WB-9
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
- Pieces:
  1. Choosing Help > Toggle Developer Tools opens the developer tools for the active window, and choosing it again closes them.
  2. The Command Palette entry "Developer: Toggle Developer Tools" does the same thing.
  3. With several windows open the command acts only on the window that has focus.
  4. Pressing `Ctrl+Shift+I` (`Cmd+Option+I` on macOS) toggles the tools in every Cursor build. [cursor only]
  5. In a release build of stock the key does nothing, because it is bound only in a development build, while the Help row and the palette entry still work. [stock only]

### NEW-B-2 Reload with extensions disabled
- Group: WB-9
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
- Pieces:
  1. Running "Developer: Reload with Extensions Disabled" reloads the window at once, with no question asked first, and the window comes back with all extensions off.
  2. The command exists only in the Command Palette under Developer, with no menu row and no key.
  3. The extensions are off for that reload only, so an ordinary Reload Window afterwards brings them back (the flag is passed for one reload; I did not run it).

### NEW-B-3 Reveal User Data Folder
- Group: WB-9
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
- Pieces:
  1. Running "Developer: Reveal User Data Folder" opens the file manager (Explorer on Windows, Finder on macOS) with the user data folder selected in its parent folder.
  2. Nothing shows inside the editor window itself, so there is no dialog, notification or editor tab.
  3. The command exists only in the Command Palette under Developer, with no menu row and no key.

### NEW-B-4 Show GPU info
- Group: WB-9
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
- Pieces:
  1. Running "Developer: Show GPU Info" opens a separate window, a child of the current window, that shows the Chromium `chrome://gpu` page.
  2. Running the command again while that window is open does not make a second one: it restores the window if it is minimized and brings it to the front.
  3. After you close the window, running the command opens a fresh one.
  4. The command exists only in the Command Palette under Developer, with no menu row and no key.

### NEW-B-5 Show content tracing
- Group: WB-9
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
- Pieces:
  1. Running "Developer: Show Content Tracing" opens a window with the Chromium `chrome://tracing` page, which stays hidden until the page has loaded and then shows. [stock only]
  2. Unlike the GPU info window, the tracing window is not a child of the editor window that opened it, so it has no parent to follow. [stock only]
  3. Running the command again while the window is open restores it if minimized and brings it to the front instead of opening another. [stock only]
  4. Cursor has no such command in its Command Palette. [stock only]

### NEW-B-6 Force refresh server config
- Group: WB-9
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
- Pieces:
  1. Running "Developer: Force Refresh Server Config" asks the server for the configuration again without waiting for the normal refresh. [cursor only]
  2. On success an info notification "Server config refreshed" appears, with the line "Request ID: <id>" under it. [cursor only]
  3. On failure an error notification "Failed to refresh server config" appears with the same Request ID line. [cursor only]
  4. The command exists only in the Command Palette, with no menu row and no key. [cursor only]

### NEW-B-7 Capture and upload a process snapshot
- Group: WB-9
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
- Pieces:
  1. Running "Developer: Capture and Upload Process Snapshot" starts the capture straight away, with no confirmation first, and writes the snapshot file to disk. [cursor only]
  2. While the file uploads, a progress message "Uploading snapshot to Cursor..." shows in the window. [cursor only]
  3. A notification then says "Snapshot uploaded successfully. Thank you!", or "Snapshot upload failed. The snapshot is still saved on disk.", or "Could not upload snapshot: unable to get upload URL." [cursor only]
  4. If the capture itself fails, an error notification reads "An error occurred while capturing debug snapshot: <message>". [cursor only]
  5. Running the command while another check or snapshot is in progress shows nothing and only writes a warning to the log. [cursor only]

### NEW-B-8 Open Process Explorer
- Group: WB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Open Process Explorer from the Help menu or the Command Palette to see every process the app runs with its CPU and memory use, and to kill, copy or debug a process.
- Commands: workbench.action.openProcessExplorer; workbench.editor.processExplorer (the editor id, not a command); workbench.action.openGlassProcessExplorer (Cursor only, related)
- Menus: MenubarHelpMenu (group 5_tools, order 2); CommandPalette (Developer category)
- Keys: Alt+E kills the selected rows while the list has focus (a key handler on the list, not a keybinding rule)
- Settings: none found
- Cursor: changed - the command, the Help row and the Developer category are present; the stock editor id `workbench.editor.processExplorer` has 0 hits, because Cursor asks its process service to open a native window styled with the theme colors; Cursor adds Open Workspace Process Explorer
- Source: V\src\vs\workbench\contrib\processExplorer\browser\processExplorer.contribution.ts:38-171; V\src\vs\workbench\contrib\processExplorer\browser\processExplorerControl.ts (columns, context menu, Alt+E, 1 second refresh); C bundle (class with id workbench.action.openProcessExplorer, nls 7505 and 7494 for the Help row, processMainService.openProcessExplorer with theme styles, id workbench.action.openGlassProcessExplorer with nls 7506 and mode workspaceGroups)
- Pieces:
  1. Help > Open Process Explorer, or the palette entry "Developer: Open Process Explorer", opens a separate window with a tree of all processes of the app.
  2. Each row shows the columns Process Name, CPU (%), PID and Memory (MB), and the list refreshes about once a second without a button.
  3. Right-clicking a process offers Kill Process, Force Kill Process, Copy, Copy All, and Debug for Node style processes that can be attached to.
  4. Pressing `Alt+E` on the selected rows ends those processes with the same signal as Kill Process.
  5. Remote machines get their own heading in the list, and a machine that cannot be reached shows an error row.
  6. The stock window is an editor in an auxiliary window that stays on top of others, starts at 800 by 500 and remembers its size and place for next time. [stock only]
  7. Cursor draws the explorer in a native window styled with the current theme colors, not as an editor tab. [cursor only]
  8. Cursor adds "Developer: Open Workspace Process Explorer", which opens the same window in a mode that groups processes by workspace. [cursor only]

### NEW-B-9 Troubleshoot Issue
- Group: WB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Troubleshoot Issue from the Command Palette to be walked through finding out whether a problem comes from an extension, from your settings, or from the product, by reloading the window several times and asking each time if you still see it.
- Commands: workbench.action.troubleshootIssue.start; workbench.action.troubleshootIssue.stop
- Menus: CommandPalette (Help category)
- Keys: none found
- Settings: none found
- Cursor: present - both ids found
- Source: V\src\vs\workbench\contrib\issue\browser\issueTroubleshoot.ts:43-380; C bundle (ids workbench.action.troubleshootIssue.start and stop, category Help, precondition on the troubleshoot context key)
- Pieces:
  1. Running "Help: Troubleshoot Issue..." first shows a confirm box titled "Troubleshoot Issue" that says the window will reload repeatedly and that you must say each time whether you still see the issue, with a Troubleshoot Issue button.
  2. After you confirm, a temporary profile is created, the window reloads with all extensions off, and a sticky notification says "Issue troubleshooting is active and has temporarily disabled all installed extensions" with the buttons I Can't Reproduce, I Can Reproduce and Stop.
  3. Answering I Can't Reproduce at that stage means an extension causes it, so the window returns to your own profile and starts an extension bisect (NEW-B-10).
  4. Answering I Can Reproduce moves to a second stage in an empty temporary profile, with a notification that says your configurations were temporarily reset to defaults and the same three buttons.
  5. In the second stage, I Can't Reproduce reports that your configuration is the cause and tells you to share an exported profile, and I Can Reproduce reports that the issue is in the product; both then offer to open the issue reporter.
  6. On a stable build, before the reporter opens, a box asks you to try the Insiders build, with buttons to download it or to report the issue anyway.
  7. Stop, from the notification or from "Help: Stop Troubleshoot Issue" (listed only while a session is active), ends the session and switches back to the profile you started in.
  8. The session survives the reloads because its stage is stored with the profile, so the notification comes back after each reload.
  9. The start command is not offered in a remote window, in the web, or while a session is already active.

### NEW-B-10 Extension Bisect
- Group: WB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Start Extension Bisect to find an extension that causes a problem by turning half of your extensions off at a time and asking whether the problem is still there; it touches the extension system only as the thing it turns off.
- Commands: extension.bisect.start; extension.bisect.next; extension.bisect.stop
- Menus: CommandPalette (Help category); ViewContainerTitle (when viewContainer is workbench.view.extensions, group 2_enablement, order 4)
- Keys: none found
- Settings: none found
- Cursor: present - all three ids found (the Extensions view menu row was not checked)
- Source: V\src\vs\workbench\services\extensionManagement\browser\extensionBisect.ts:165-370; C bundle (ids extension.bisect.start, next and stop, and the call that opens the issue reporter with the found extension id)
- Pieces:
  1. "Help: Start Extension Bisect" (or the matching row in the Extensions view title menu) shows a confirm box "Extension Bisect" that says a binary search will reload the window about log2(n) plus 2 times, with a Start Extension Bisect button.
  2. After you confirm, the window reloads with half of your enabled extensions off, and a sticky urgent notification says "Extension Bisect is active and has disabled N extensions" with the buttons I can't reproduce, I can reproduce and Stop Bisect.
  3. Each answer narrows the search and reloads again; running "Help: Continue Extension Bisect" from the palette asks the same question in a box with a fourth button, Cancel Bisect.
  4. When an extension is found, a box says the bisect "has identified {id} as the extension causing the problem", with a checked box "Keep this extension disabled" and the buttons Report Issue & Continue and Continue.
  5. When nothing is found, a box says the bisect is done but no extension was identified and that it might be a problem with the product.
  6. "Help: Stop Extension Bisect" (listed only while a bisect is active) or the Stop Bisect button ends the search and reloads with your extensions as they were.
  7. The start command is hidden while a bisect is active, and the continue and stop commands are hidden when none is.

### NEW-B-11 Stock issue reporter
- Group: WB-6
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Choose Report Issue to describe a bug, a feature request or a performance problem in a form, or in a three step wizard with screenshots and video, that ends in a prefilled GitHub issue.
- Commands: workbench.action.openIssueReporter; vscode.openIssueReporter; workbench.action.reportPerformanceIssueUsingReporter; workbench.action.issueReporter.captureScreenshot; workbench.action.issueReporter.toggleRecording
- Menus: MenubarHelpMenu (group 3_feedback, order 3, Report Issue); CommandPalette (Help: Report Issue..., Help: Report Performance Issue...)
- Keys: Ctrl+Shift+S (capture a screenshot) and Ctrl+Shift+R (start or stop recording), Cmd instead of Ctrl on macOS, only while an issue reporter wizard tab is open
- Settings: issueReporter.wizard.enabled; issueReporter.wizard.fullWorkspaceScan; telemetry.feedback.enabled
- Cursor: missing - the two wizard ids and both wizard settings have 0 hits; the ids workbench.action.openIssueReporter, vscode.openIssueReporter and workbench.action.reportPerformanceIssueUsingReporter exist but open Cursor's own feedback pane (NEW-B-12)
- Source: V\src\vs\workbench\contrib\issue\common\issue.contribution.ts:60-130; V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts:34-150; V\src\vs\workbench\contrib\issue\electron-browser\issueService.ts:40-70; V\src\vs\workbench\contrib\issue\browser\issueFormService.ts:421-470; V\src\vs\workbench\contrib\issue\browser\issueReporterPage.ts; V\src\vs\workbench\contrib\issue\browser\issueReporterOverlay.ts; V\src\vs\workbench\contrib\issue\electron-browser\issueReporterKeybindings.ts; V\src\vs\workbench\contrib\issue\electron-browser\issueReporterEditorPane.ts; C bundle (searched all five ids and the two settings)
- Pieces:
  1. Help > Report Issue and the palette entry "Help: Report Issue..." open the reporter, and "Help: Report Performance Issue..." opens it with the Performance Issue type already chosen.
  2. By default the classic reporter opens in its own small window titled Issue Reporter with a "This is a" type list, a source list, an extension picker, a required title, a required description, and checkboxes for what to include (system information, running processes, workspace metadata, enabled extensions, A/B experiment info and extension data).
  3. The classic form tells you to try the problem again after "disabling all extensions and reloading the window", which is a link, and shows errors such as "A title is required." and "Please provide a longer description."
  4. Setting `issueReporter.wizard.enabled` (default off) swaps the form for a wizard in an editor tab with the steps Attachments, Describe and Review, shown as "Step n of 3".
  5. The Attachments step takes up to 5 screenshots or videos, using a draggable floating capture bar with a Screenshot button, a drop-down for the delay (No delay, 3, 5 or 10 seconds) and Hide Toolbar in Screenshots, and a Record video button; thumbnails can be clicked to edit or removed.
  6. While a wizard tab is open, `Ctrl+Shift+S` takes a screenshot and `Ctrl+Shift+R` starts or stops recording, even when a widget that swallows keys has focus.
  7. A recording stops by itself at the 100 MB upload limit with a message, and on macOS a missing Screen Recording permission shows a message with an Open System Settings button.
  8. The Describe step asks for a category (Bug, Feature Request or Performance Issue), a target (the product, an extension, the Extensions Marketplace), a title with a "Generate from description" button, and a description with guidance text for the category, and it lists similar issues found from the title; a missing field shows a message such as "Enter a title to continue."
  9. The Review step shows the title, description and attachments, with expandable system information, extension data, extension list, A/B experiments and performance data (running processes and workspace metadata), and a last button "Preview on GitHub" (or "Open External Issue Reporter" for an extension with its own page) that reads "Uploading..." while attachments upload.
  10. When `telemetry.feedback.enabled` is off the Report Issue command only shows the notification "Feedback is disabled." [stock only]

### NEW-B-12 Cursor Send Feedback pane
- Group: WB-6
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Choose Help > Report Issue or Help > Give Feedback to open a pane where you pick Idea, Bug or Urgent, describe it, attach screenshots and send it to the Cursor team.
- Commands: aiFeedback.action.open; cursor.giveFeedback; workbench.action.openIssueReporter; vscode.openIssueReporter; workbench.action.reportPerformanceIssueUsingReporter
- Menus: MenubarHelpMenu (group 3_feedback; Report Issue order 3, Give Feedback... order 4); CommandPalette
- Keys: Report Issue has a two key chord whose second key is Ctrl+G (Cmd+G on macOS); the first key is the Cursor chord prefix constant `zd` (`Oh` on macOS) that ED-4 decoded as Ctrl+M (Cmd+R), which I did not decode again
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with id aiFeedback.action.open, label "Report Issue", nls 5368; class with id cursor.giveFeedback, label "Give Feedback", nls 5369; AIFeedbackOpenService; pane templates feedback-pane-title "Send Feedback", feedback-type-grid, feedback-screenshot-dropzone; command registrations that call the same service for the three stock ids)
- Pieces:
  1. Help > Report Issue, Help > Give Feedback..., their palette entries, and the three stock ids (`workbench.action.openIssueReporter`, `vscode.openIssueReporter`, `workbench.action.reportPerformanceIssueUsingReporter`, which ignore any prefill arguments) all open the same Send Feedback pane over the window. [cursor only]
  2. The pane has the title "Send Feedback", a close button, three type cards - Idea (Feature request), Bug (Something is off, chosen by default) and Urgent (Can't use Cursor) - and a text box that says "Describe your idea..." for Idea and "Describe the issue..." for the others. [cursor only]
  3. An "Attach screenshot" drop area, pasting an image from the clipboard, or choosing an image file adds a thumbnail with a remove button. [cursor only]
  4. The Send button, also `Ctrl+Enter` (`Cmd+Enter` on macOS), reads "Sending..." and is disabled while it works; sending with an empty text shows "Please describe your feedback", and a failure shows "Couldn't send. Try again." [cursor only]
  5. On success the pane switches to a check mark with "Thanks for your feedback!" and "Your feedback has been sent to the Cursor team.", a Done button closes it, and the saved draft is cleared. [cursor only]
  6. Cancel, the close button, `Esc` and a click on the dark backdrop all close the pane, and the chosen type, text and screenshots are kept so reopening the pane shows them again until they are sent. [cursor only]

### NEW-B-13 Bring a window to the front by command
- Group: WB-10
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run Switch to Main Window from a secondary editor window, or Focus Window from anywhere, to bring a window to the front and give it focus.
- Commands: workbench.action.switchToMainWindow; workbench.action.focusWindow
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing - 0 hits for both ids in the bundle
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts:393-432; V\src\vs\workbench\electron-browser\desktop.contribution.ts:44-45; C bundle (searched both ids, 0 hits)
- Pieces:
  1. "Switch to Main Window" is listed in the Command Palette only while the active window is an auxiliary window (a secondary editor window), and it brings the main window to the front. [stock only]
  2. "Focus Window" brings the current window to the foreground and focuses it even when the app is not the active one, by forcing focus from the operating system. [stock only]
  3. Focus Window has no menu row and no key, and its code comment says it is meant as the first step of a `runCommands` chain that must reveal the window before a command such as Quick Open. [stock only]

### NEW-B-14 Install or remove the shell command
- Group: WB-10
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS, run Shell Command: Install command in PATH to be able to start the app from a terminal, or the Uninstall command to remove it again.
- Commands: workbench.action.installCommandLine; workbench.action.uninstallCommandLine
- Menus: CommandPalette (Shell Command category)
- Keys: none found
- Settings: none found
- Cursor: changed - both ids present and registered on macOS only, as in stock; the Install title reads "Install '{0}' command" without "in PATH" and the success text reads "Shell command '{0}' successfully installed."
- Source: V\src\vs\workbench\electron-browser\actions\installActions.ts:19-80; V\src\vs\workbench\electron-browser\desktop.contribution.ts:65-69 (macOS only); C bundle (ids workbench.action.installCommandLine and uninstallCommandLine, category "Shell Command" nls 12007, titles nls 12008 to 12010, success text nls 12002 and 12006, registration behind the macOS check)
- Pieces:
  1. On macOS the palette lists "Shell Command: Install '<name>' command in PATH" (Cursor words it "Install '<name>' command"), and on Windows and Linux neither command exists.
  2. When the install works, an info box says "Shell command '<name>' successfully installed in PATH." (Cursor: "...successfully installed.")
  3. When it fails, an error box shows the error text, and cancelling the macOS administrator prompt closes quietly with no box (DL-1 notes the osascript prompt).
  4. "Shell Command: Uninstall '<name>' command from PATH" removes it and shows "Shell command '<name>' successfully uninstalled from PATH."

### NEW-B-15 Offer to open a workspace file found in the folder
- Group: WB-10
- Tag: both
- Evidence: source
- Boundary: no
- What: When you open a folder that has a .code-workspace file at its top level, a notification offers to open that workspace file instead.
- Commands: workspaces.dontPromptToOpen (the Don't Show Again id, not a command); workbench.action.openWorkspaceFromEditor (related, the Open Workspace button on a .code-workspace file, named in ED-5)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - the notification class, id and strings (nls 11961 to 11963) are in the bundle
- Source: V\src\vs\workbench\contrib\workspaces\browser\workspaces.contribution.ts:26-108; C bundle (class with doHandleWorkspaceFiles, neverShowAgain id workspaces.dontPromptToOpen with workspace scope)
- Pieces:
  1. After a single real (non virtual) folder opens, and its top level holds exactly one workspace file, an info notification says "This folder contains a workspace file '<name>'. Do you want to open it?" with a Learn more link and an Open Workspace button.
  2. When the folder holds several workspace files the text says it "contains multiple workspace files", and its Select Workspace button opens a pick list "Select a workspace to open" of the file names.
  3. Pressing Open Workspace, or picking a name from the list, opens that workspace file.
  4. The notification has a Don't Show Again choice that is remembered for this workspace only.
  5. In a workspace that has been opened before, the notification arrives silently, in the notification center without a pop-up; in a brand new workspace it pops up as usual.

### NEW-B-16 Slow close, quit or reload dialog with the pending steps
- Group: DL-2
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
- Pieces:
  1. If shutdown is still going after about 0.8 seconds, a progress message shows in the status bar: "Closing the window is taking a bit longer...", with variants for quitting the application, reloading the window and changing the workspace.
  2. If it is still going after about 1.2 seconds, a box with the same title opens and lists "The following operations are still running:" with one line per unfinished step.
  3. The box has one button, "Close Anyway", "Quit Anyway", "Reload Anyway" or "Change Anyway", that forces the shutdown; the box cannot be cancelled or dismissed and disappears when the shutdown finishes.
  4. The step names that can appear are Saving UI state, Saving text files, Backup working copies, Saving working copies, Saving local history, Stopping Extension Hosts, Disconnect Remote Agent (remote windows, last in order) and Terminating extension debug session (extension development windows).
  5. Cursor adds the steps "Cleaning up browser views" and "Flushing analytics events" to that list. [cursor only]
  6. If shutdown fails with an unexpected error, an error box says "An unexpected error prevented the window to close" (with quit, reload and change workspace variants) and shows "Error: <message>" as detail.

### NEW-B-17 Save failed notification for file working copies
- Group: WB-1
- Tag: both
- Evidence: source
- Boundary: no
- What: When a save fails for a file working copy, an error notification explains why and offers buttons such as Retry, Overwrite, Revert, Save As and a save as administrator; the plain text editor has its own separate save error handler.
- Commands: fileWorkingCopy.overwrite; fileWorkingCopy.revert; fileWorkingCopy.retry; fileWorkingCopy.unlock; fileWorkingCopy.saveElevated; fileWorkingCopy.saveAs (all six are button ids, not commands)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - all six ids and the strings (nls 13563 to 13579) are in the bundle
- Source: V\src\vs\workbench\services\workingCopy\common\storedFileWorkingCopy.ts:1143-1232; C bundle (doHandleSaveError with the six button ids, nls 13563 to 13579)
- Pieces:
  1. If the file on disk is newer than the copy you edited, the notification reads "Failed to save '<name>': The content of the file is newer. Do you want to overwrite the file with your changes?" with the buttons Overwrite and Revert.
  2. If the file is read-only, it reads "Failed to save '<name>': File is read-only. Select 'Overwrite' to attempt to make it writeable." with Overwrite, Save As... and Revert.
  3. On a permission error, or a read-only file the app can overwrite with elevated rights, the first button is Retry as Admin... or Overwrite as Admin... on Windows and Retry as Sudo... or Overwrite as Sudo... elsewhere, with a message that names it.
  4. For any other error it reads "Failed to save '<name>': <error>" with Retry, Save As... and Revert.
  5. Save As... opens the Save As picker, and if that fails or is cancelled the same notification shows again.
  6. The notification closes by itself as soon as the file is saved or reverted.

### NEW-B-18 Resize a table column from the Command Palette
- Group: WB-3
- Tag: both
- Evidence: source
- Boundary: no
- What: With a table focused, run List: Resize Column to pick a column and type the width it should have as a percentage, which is the keyboard way to resize columns.
- Commands: list.resizeColumn
- Menus: CommandPalette (List category)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\list\browser\listResizeColumnAction.ts:13-34; V\src\vs\workbench\contrib\list\browser\tableColumnResizeQuickPick.ts; V\src\vs\workbench\contrib\markers\browser\markersTable.ts, V\src\vs\workbench\contrib\preferences\browser\keybindingsEditor.ts, V\src\vs\workbench\contrib\userDataProfile\browser\userDataProfilesEditor.ts, V\src\vs\workbench\contrib\workspace\browser\workspaceTrustEditor.ts (tables that use the workbench table widget); C bundle (id list.resizeColumn, nls 7533 and 7534)
- Pieces:
  1. "List: Resize Column" is offered in the palette only while a list has focus, and it does nothing unless the focused list is a table.
  2. It opens a pick list with the placeholder "Select the column to resize, type to filter." that holds the table's column headings.
  3. After you pick a column an input box asks "Please enter a width in percentage for the '<column>' column." with the placeholder "i.e. 20, 60, 100...".
  4. Text that is not a whole number shows "Please enter an integer.", and a number outside 1 to 100 shows "Please enter a number greater than 0 and less than or equal to 100."
  5. The column takes the new width at once, and cancelling either box or entering nothing leaves the table as it was.
  6. It works on every table built with the workbench table widget, such as the Problems table, the Keyboard Shortcuts editor and the Profiles editor, not only one of them.

### NEW-B-19 Show or focus the hover of the focused element
- Group: WB-3
- Tag: both
- Evidence: source
- Boundary: no
- What: Press Ctrl+K Ctrl+I with focus on a button, tab, status bar item or tree row to show its hover and move keyboard focus into it, without using the mouse.
- Commands: workbench.action.showHover
- Menus: none found
- Keys: Ctrl+K Ctrl+I (stock: any focus, weight EditorCore; Cursor: only when the editor does not have text focus, weight 199)
- Settings: workbench.hover.delay; workbench.hover.reducedDelay
- Cursor: changed - the command and `workbench.hover.delay` are present with a `when` of not editorTextFocus; `workbench.hover.reducedDelay` has 0 hits
- Source: V\src\vs\platform\hover\browser\hoverService.ts:110-123, 506-560; V\src\vs\workbench\browser\workbench.contribution.ts:733-746; C bundle (registerCommandAndKeybindingRule for workbench.action.showHover with Ctrl+K Ctrl+I and the negated editorTextFocus condition; the setting workbench.hover.delay)
- Pieces:
  1. With focus on a part of the workbench that has a hover, pressing `Ctrl+K Ctrl+I` shows that hover and puts keyboard focus inside it.
  2. The command looks at the focused element and then at each parent until it finds one with a hover, and does nothing visible when none has one.
  3. While a hover is open, holding `Alt` locks it so it stays when the pointer leaves, releasing `Alt` closes it if the pointer is outside, and any other key that is not bound hides it and returns focus to the element.
  4. Setting `workbench.hover.delay` chooses how many milliseconds the pointer rests before a workbench hover shows (500, or 1500 on macOS).
  5. Setting `workbench.hover.reducedDelay` (default 500) is the shorter delay used where faster feedback helps. [stock only]
  6. In Cursor the workbench rule is switched off while a text editor has focus, so the key there belongs to the editor's own Show or Focus Hover (ED-4). [cursor only]

### NEW-B-20 Hide toolbar actions and reset all menus
- Group: WB-8
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click an action button in a toolbar to hide it, and use Reset Menu or View: Reset All Menus to bring hidden actions back.
- Commands: menu.resetHiddenStates
- Menus: CommandPalette (View category); the right-click menu of menu driven toolbars (rows Hide, a checklist of hideable actions and Reset Menu)
- Keys: none found
- Settings: none found
- Cursor: present - the command (nls 1783), the Hide row and the Reset Menu row (nls 1780 to 1782) are in the bundle
- Source: V\src\vs\platform\actions\common\menuResetAction.ts:12-27; V\src\vs\platform\actions\browser\toolbar.ts:120-290; V\src\vs\workbench\browser\parts\editor\editorTabsControl.ts:265-298; V\src\vs\workbench\browser\parts\views\viewPane.ts:478; V\src\vs\workbench\contrib\scm\browser\scmRepositoryRenderer.ts:97; V\src\vs\workbench\contrib\notebook\browser\viewParts\notebookEditorToolbar.ts:360; C bundle (id menu.resetHiddenStates, class with resetThisMenu)
- Pieces:
  1. Right-clicking an action button in a toolbar built from a menu (the editor title bar, view title bars, Source Control repository rows, the notebook toolbar) opens a menu with Configure Keybinding when the action can have a key, then Hide '<action>' and a checklist of every hideable action in that toolbar.
  2. Choosing Hide, or unchecking an entry in the checklist, removes the button from the toolbar and moves it into the "..." overflow menu, and checking it again puts it back.
  3. When anything in that toolbar is hidden, the menu also ends with Reset Menu, which shows everything again for that toolbar, and the same row is in the menu opened from the "..." button.
  4. An action that cannot be hidden shows a greyed "Hide" label in place of Hide '<action>'.
  5. "View: Reset All Menus" brings back every hidden action in every toolbar at once and writes "did RESET all menu hidden states" to the log.
  6. Toolbars set not to allow hiding, such as the title bar, the Command Center and the side bar and panel tab strips, show no hide rows.

### NEW-B-21 Configure runtime arguments
- Group: ED-8
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Preferences: Configure Runtime Arguments to open the argv.json file, where startup options such as the display language, hardware acceleration and log level are set.
- Commands: workbench.action.configureRuntimeArguments
- Menus: CommandPalette (Preferences category)
- Keys: none found
- Settings: none found
- Cursor: present - the command and its label (nls 11997) are in the bundle; the argv schema was not checked
- Source: V\src\vs\workbench\electron-browser\actions\developerActions.ts:52-72; V\src\vs\workbench\electron-browser\desktop.contribution.ts:404-488 (argv schema vscode://schemas/argv); C bundle (id workbench.action.configureRuntimeArguments, category Preferences)
- Pieces:
  1. Running "Preferences: Configure Runtime Arguments" opens the argv.json file in a pinned editor tab.
  2. The file has a schema, so completion and hover text describe the options, such as `locale`, `disable-hardware-acceleration`, `disable-lcd-text`, `force-color-profile`, `enable-crash-reporter`, `proxy-bypass-list`, `log-level`, `js-flags` and `remote-debugging-port`, plus Linux options such as `password-store`.
  3. Several descriptions say the app must be restarted before a changed option takes effect.

### NEW-B-22 Display language prompts
- Group: WB-5
- Tag: both
- Evidence: source
- Boundary: no
- What: Notifications offer to switch the display language after a language pack is installed or when the operating system language differs, and two palette commands change it by hand.
- Commands: langugage.update.donotask (the Don't Show Again id, spelled this way in the product, not a command); workbench.action.configureLocale; workbench.action.clearLocalePreference
- Menus: CommandPalette (Configure Display Language and Clear Display Language Preference)
- Keys: none found
- Settings: none found
- Cursor: present - the id, the strings (nls 7595 to 7604) and both related commands are in the bundle
- Source: V\src\vs\workbench\contrib\localization\electron-browser\localization.contribution.ts:20-217; V\src\vs\workbench\contrib\localization\common\localizationsActions.ts:16,99; C bundle (class with checkAndInstall and onDidInstallExtension, neverShowAgain id langugage.update.donotask)
- Pieces:
  1. After you install a language pack, a sticky notification asks "Would you like to change <product>'s display language to <language> and restart?" with a Change Language and Restart button.
  2. That notification has a Don't Show Again choice that is remembered for the whole app.
  3. At startup, when the operating system language differs from the display language and no pack for it is installed, a notification offers "Install and Restart" when one matching pack is found, or "Search Marketplace" when several are, with the text in the pack's language and the English text in brackets when a translation exists.
  4. Don't Show Again on that startup notification remembers the locale, so it is not offered again for it.
  5. Removing the pack of the active language switches the display language back to English.
  6. "Configure Display Language" picks an installed language, and "Clear Display Language Preference" resets it.

### NEW-B-23 Run, debug or cover the tests in the current file
- Group: SB-6
- Tag: both
- Evidence: source
- Boundary: no
- What: Run all tests of the file you are editing, debug them, or run them with coverage, from the Command Palette, the editor right-click menu or a key chord.
- Commands: testing.runCurrentFile; testing.debugCurrentFile; testing.coverageCurrentFile
- Menus: CommandPalette (Test category, when the matching run profile exists); EditorContext (group testing, when the active editor has tests and the profile exists)
- Keys: Ctrl+; then F (run); Ctrl+; then Ctrl+F (debug); Ctrl+; then Ctrl+Shift+F (coverage), all when the editor has text focus
- Settings: testing.saveBeforeTest
- Cursor: present - ids, labels (nls 11100 to 11102) and the chords are in the bundle
- Source: V\src\vs\workbench\contrib\testing\browser\testExplorerActions.ts:1239-1372; C bundle (classes with ids testing.runCurrentFile, testing.debugCurrentFile and testing.coverageCurrentFile and their key chords)
- Pieces:
  1. "Test: Run Tests in Current File" runs every test found in the active editor's file with the default run profile, "Test: Debug Tests in Current File" starts them as a debug session, and "Test: Run Tests with Coverage in Current File" runs them with coverage.
  2. Each entry shows in the palette, and in the editor's right-click menu under the testing group, only when a test extension has registered a matching run profile, and the editor menu also needs the file to contain tests.
  3. With text focus in the editor, `Ctrl+;` then `F` runs, `Ctrl+;` then `Ctrl+F` debugs, and `Ctrl+;` then `Ctrl+Shift+F` runs with coverage.
  4. When the file has no tests, the message "No tests found in this file" appears at the cursor in the editor.
  5. Setting `testing.saveBeforeTest` (default on) saves unsaved files before the run starts.

### NEW-B-24 Opening in Cursor Browser toast
- Group: ED-10
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the toast appears when a link is sent to the Cursor Browser, including automatic opens of dev server addresses, and the Browser is an agent-driven surface
- What: An info notification "Opening <url> in Cursor Browser" appears when a link opens in a Browser tab through the automatic opener, with a Don't Open button that switches the behavior off.
- Commands: urlOpenToast.dontOpen (the button id, not a command)
- Menus: none found
- Keys: none found
- Settings: autoOpenLocalhostUrls (a Cursor application storage key shown on the Cursor Settings Browser page as "Open Local Links in Cursor Browser", not a settings.json id); urlOpenToast.neverShowAgainv1 (the Don't Show Again key)
- Cursor: cursor only
- Source: C bundle (class urlOpenToastService with showUrlOpenToast, nls 12294 "Opening {0} in Cursor Browser" and 12295 "Don't Open"; callers in the open browser command and in OpenBrowserUrlsService)
- Pieces:
  1. When a URL opens in a Cursor Browser tab through the automatic opener, an info notification reads "Opening <url> in Cursor Browser". [cursor only]
  2. The notification's Don't Open button turns off "Open Local Links in Cursor Browser" (the stored switch `autoOpenLocalhostUrls`), so later links go to the system browser. [cursor only]
  3. Unless Don't Show Again was chosen, the toast shows only once per web address origin in a session, and Don't Show Again (application wide) stops it for good. [cursor only]
