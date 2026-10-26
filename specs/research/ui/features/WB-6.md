Group: WB-6 Help and diagnostics. Section: Workbench-wide. Date: 2026-10-25. Pass: 4.
Features: 7.

### WB-6.1 Help > View License
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

### WB-6.2 Open Process Explorer
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

### WB-6.3 Troubleshoot Issue
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

### WB-6.4 Extension Bisect
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

### WB-6.5 Stock issue reporter
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Choose Report Issue to describe a bug, a feature request or a performance problem in a form, or in a three step wizard with screenshots and video, that ends in a prefilled GitHub issue.
- Commands: workbench.action.openIssueReporter; vscode.openIssueReporter; workbench.action.reportPerformanceIssueUsingReporter; workbench.action.issueReporter.captureScreenshot; workbench.action.issueReporter.toggleRecording
- Menus: MenubarHelpMenu (group 3_feedback, order 3, Report Issue); CommandPalette (Help: Report Issue..., Help: Report Performance Issue...); MenubarHelpMenu (row Report Issue); CommandPalette
- Keys: Ctrl+Shift+S (capture a screenshot) and Ctrl+Shift+R (start or stop recording), Cmd instead of Ctrl on macOS, only while an issue reporter wizard tab is open; Ctrl+Shift+S (workbench.action.issueReporter.captureScreenshot) and Ctrl+Shift+R (workbench.action.issueReporter.toggleRecording), Cmd on macOS, both only while an Issue Reporter tab is open
- Settings: issueReporter.wizard.enabled; issueReporter.wizard.fullWorkspaceScan; telemetry.feedback.enabled
- Cursor: missing - the two wizard ids and both wizard settings have 0 hits; the ids workbench.action.openIssueReporter, vscode.openIssueReporter and workbench.action.reportPerformanceIssueUsingReporter exist but open Cursor's own feedback pane (WB-6.6)
- Source: V\src\vs\workbench\contrib\issue\common\issue.contribution.ts:60-130; V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts:34-150; V\src\vs\workbench\contrib\issue\electron-browser\issueService.ts:40-70; V\src\vs\workbench\contrib\issue\browser\issueFormService.ts:421-470; V\src\vs\workbench\contrib\issue\browser\issueReporterPage.ts; V\src\vs\workbench\contrib\issue\browser\issueReporterOverlay.ts; V\src\vs\workbench\contrib\issue\electron-browser\issueReporterKeybindings.ts; V\src\vs\workbench\contrib\issue\electron-browser\issueReporterEditorPane.ts; C bundle (searched all five ids and the two settings); V\src\vs\workbench\contrib\issue\electron-browser\issue.contribution.ts (lines 47-76); V\src\vs\workbench\contrib\issue\electron-browser\nativeIssueFormService.ts (lines 59-85); V\src\vs\workbench\contrib\issue\electron-browser\issueReporterKeybindings.ts (lines 21-112); V\src\vs\workbench\contrib\issue\browser\issueReporterOverlay.ts (step titles, field labels and the attachment limit of 5); V\src\vs\workbench\contrib\issue\common\issue.contribution.ts (lines 70-130); V\src\vs\workbench\contrib\preferences\browser\settingsLayout.ts (lines 376-380, the Issue Reporter section with hide set to not web); C bundle (0 hits for IssueReporterEditor, IssueReporterOverlay and issueReporter.wizard)

### WB-6.6 Cursor Send Feedback pane
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

### WB-6.7 Attach the debugger to a process from the Process Explorer
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-click a Node-style process in the Process Explorer and choose Debug to start a debug session attached to that process.
- Commands: debug.startFromConfig; workbench.action.openProcessExplorer
- Menus: Process Explorer process row right-click menu (built in code, rows Kill Process, Force Kill Process, Copy, Copy All, Debug)
- Keys: none found
- Settings: none found
- Cursor: changed (the Process Explorer is a separate native window of older code, and its Debug row sends the same command with the same attach rules)
- Source: V\src\vs\workbench\contrib\processExplorer\browser\processExplorerControl.ts (onTreeContextMenu, isDebuggable, attachTo, DEBUG_FLAGS_PATTERN, DEBUG_PORT_PATTERN); V\src\vs\workbench\contrib\processExplorer\browser\processExplorer.contribution.ts (opens the explorer as an editor in an auxiliary window); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (debug.startFromConfig); C\out\vs\code\electron-sandbox\processExplorer\processExplorerMain.js (attachTo sends debug.startFromConfig); C bundle (debug.startFromConfig handler calling startDebugging, workbench.action.openProcessExplorer calling openProcessExplorer on the native host)

## Handed off
- none

## Unclear
- Created in pass 4 from the coverage check.
