Group: WB-6 Help and diagnostics. Pass: 3. Date: 2026-10-25.

### WB-6.1
1. Choosing Help > View License opens the license page in the default web browser.
2. Running "Help: View License" from the Command Palette does the same.
3. When the window has a display language set, the page address gets a `lang=` query added with that language.
4. The menu row sits in the legal group of the Help menu and has the mnemonic L.
5. The command exists only when the product has a license address; Cursor sets it to https://cursor.com/license.txt.

### WB-6.2
1. Help > Open Process Explorer, or the palette entry "Developer: Open Process Explorer", opens a separate window with a tree of all processes of the app.
2. Each row shows the columns Process Name, CPU (%), PID and Memory (MB), and the list refreshes about once a second without a button.
3. Right-clicking a process offers Kill Process, Force Kill Process, Copy, Copy All, and Debug for Node style processes that can be attached to.
4. Pressing `Alt+E` on the selected rows ends those processes with the same signal as Kill Process.
5. Remote machines get their own heading in the list, and a machine that cannot be reached shows an error row.
6. The stock window is an editor in an auxiliary window that stays on top of others, starts at 800 by 500 and remembers its size and place for next time. [stock only]
7. Cursor draws the explorer in a native window styled with the current theme colors, not as an editor tab. [cursor only]
8. Cursor adds "Developer: Open Workspace Process Explorer", which opens the same window in a mode that groups processes by workspace. [cursor only]

### WB-6.3
1. Running "Help: Troubleshoot Issue..." first shows a confirm box titled "Troubleshoot Issue" that says the window will reload repeatedly and that you must say each time whether you still see the issue, with a Troubleshoot Issue button.
2. After you confirm, a temporary profile is created, the window reloads with all extensions off, and a sticky notification says "Issue troubleshooting is active and has temporarily disabled all installed extensions" with the buttons I Can't Reproduce, I Can Reproduce and Stop.
3. Answering I Can't Reproduce at that stage means an extension causes it, so the window returns to your own profile and starts an extension bisect (WB-6.4).
4. Answering I Can Reproduce moves to a second stage in an empty temporary profile, with a notification that says your configurations were temporarily reset to defaults and the same three buttons.
5. In the second stage, I Can't Reproduce reports that your configuration is the cause and tells you to share an exported profile, and I Can Reproduce reports that the issue is in the product; both then offer to open the issue reporter.
6. On a stable build, before the reporter opens, a box asks you to try the Insiders build, with buttons to download it or to report the issue anyway.
7. Stop, from the notification or from "Help: Stop Troubleshoot Issue" (listed only while a session is active), ends the session and switches back to the profile you started in.
8. The session survives the reloads because its stage is stored with the profile, so the notification comes back after each reload.
9. The start command is not offered in a remote window, in the web, or while a session is already active.

### WB-6.4
1. "Help: Start Extension Bisect" (or the matching row in the Extensions view title menu) shows a confirm box "Extension Bisect" that says a binary search will reload the window about log2(n) plus 2 times, with a Start Extension Bisect button.
2. After you confirm, the window reloads with half of your enabled extensions off, and a sticky urgent notification says "Extension Bisect is active and has disabled N extensions" with the buttons I can't reproduce, I can reproduce and Stop Bisect.
3. Each answer narrows the search and reloads again; running "Help: Continue Extension Bisect" from the palette asks the same question in a box with a fourth button, Cancel Bisect.
4. When an extension is found, a box says the bisect "has identified {id} as the extension causing the problem", with a checked box "Keep this extension disabled" and the buttons Report Issue & Continue and Continue.
5. When nothing is found, a box says the bisect is done but no extension was identified and that it might be a problem with the product.
6. "Help: Stop Extension Bisect" (listed only while a bisect is active) or the Stop Bisect button ends the search and reloads with your extensions as they were.
7. The start command is hidden while a bisect is active, and the continue and stop commands are hidden when none is.

### WB-6.5
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

### WB-6.6
1. Help > Report Issue, Help > Give Feedback..., their palette entries, and the three stock ids (`workbench.action.openIssueReporter`, `vscode.openIssueReporter`, `workbench.action.reportPerformanceIssueUsingReporter`, which ignore any prefill arguments) all open the same Send Feedback pane over the window. [cursor only]
2. The pane has the title "Send Feedback", a close button, three type cards - Idea (Feature request), Bug (Something is off, chosen by default) and Urgent (Can't use Cursor) - and a text box that says "Describe your idea..." for Idea and "Describe the issue..." for the others. [cursor only]
3. An "Attach screenshot" drop area, pasting an image from the clipboard, or choosing an image file adds a thumbnail with a remove button. [cursor only]
4. The Send button, also `Ctrl+Enter` (`Cmd+Enter` on macOS), reads "Sending..." and is disabled while it works; sending with an empty text shows "Please describe your feedback", and a failure shows "Couldn't send. Try again." [cursor only]
5. On success the pane switches to a check mark with "Thanks for your feedback!" and "Your feedback has been sent to the Cursor team.", a Done button closes it, and the saved draft is cleared. [cursor only]
6. Cancel, the close button, `Esc` and a click on the dark backdrop all close the pane, and the chosen type, text and screenshots are kept so reopening the pane shows them again until they are sent. [cursor only]

### WB-6.7
1. Right-clicking a process row whose command line has an inspect, inspect-brk or inspect-port option with a port other than 0, or that runs `node`, adds a Debug row under Copy and Copy All.
2. Choosing Debug starts a Node attach debug session named `process <pid>`.
3. When the command line gives an inspect port, the session attaches to that port, and with no port it attaches by process id, which signals the process to open its inspector.
4. An inspect-port option with a port number in the command line overrides the port.
5. Rows for other processes show no Debug row, and the row is never offered in the web version.
6. The session starts in the main window like any other debug session, so the floating debug toolbar and the Run and Debug view react to it.
7. Cursor opens the Process Explorer as its own native window, where stock opens it as an editor in an always-on-top auxiliary window, and its Debug row sends the same command. [cursor only]

## Unclear
none
