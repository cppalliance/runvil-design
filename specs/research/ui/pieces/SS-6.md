Group: SS-6 AI suggestions and AI stats. Pass: 3. Date: 2026-10-25.

### SS-6.1
1. The Language Status popup has a row named "Inline Suggestions" with the detail text "Inline suggestions", shown only for editors that are not read-only.
2. While the editor is waiting, the row label reads "Loading..." with a spinner.
3. When an inline completion is available the label reads "Inline completion available" with a Copilot icon in front, and when an inline edit is available it reads "Inline edit available" with the same icon.
4. When nothing is available the label reads "No inline suggestion available" with the Copilot icon in front.
5. The row has no command link.
6. Like every language status row it can be pinned with "Add to Status Bar", and once pinned its bar item shows only an icon: a lightbulb for a completion, a lightbulb with a sparkle for an edit, a circle with a slash for none, and the spinner while loading.
7. Stock hides the row when AI features are turned off with `chat.disableAIFeatures`. [stock only]
8. In Cursor the row has the same labels but follows only the focused editor, and nothing hides it when AI features are turned off, because `chat.disableAIFeatures` does not exist there. [cursor only]

### SS-6.2
1. With `editor.aiStats.enabled` on, an item with no text appears on the right side of the bar and draws a thin bar, 50 by 6 pixels with rounded ends, that fills from the left to show the current AI share. [stock only]
2. The item has the name "Inline Suggestions" and the screen reader label "Inline suggestions status bar". [stock only]
3. The item has no click action, and hovering it opens the AI Usage Statistics panel. [stock only]
4. Setting `editor.aiStats.enabled` chooses whether the item exists, with values on and off (default), and it is marked experimental. [stock only]
5. The item is hidden when AI features are turned off with `chat.disableAIFeatures`. [stock only]

### SS-6.3
1. Hovering the gauge shows a panel headed "AI Usage Statistics". [stock only]
2. Two lines under the header read "AI vs Typing Average: {n}%" and "Accepted inline suggestions today: {n}". [stock only]
3. Below a divider a bar chart shows one bar per day or per session, with 0%, 50% and 100% marks on its vertical axis. [stock only]
4. The chart header reads "AI Rate by Day" in the Days view, which is the view it opens in, and "AI Rate by Session" in the Sessions view. [stock only]
5. Two small icon buttons at the right of the chart header, Days with a calendar icon and Sessions with a flat list icon, switch the chart, and the active one is highlighted. [stock only]
6. In the Days view the bars have date labels along the bottom edge. [stock only]
7. When there is nothing to chart the chart area reads "No data yet". [stock only]

### SS-6.4
1. The header of the AI Usage Statistics panel has a gear icon button at its right with the tooltip "Configure". [stock only]
2. Clicking the gear opens the Settings editor filtered to `editor.aiStats.enabled` (`workbench.action.openSettings`). [stock only]

### SS-6.5
1. With `telemetry.editStats.showStatusBar` on, a right-side item with an edit icon and the text "{n} chars inserted" appears, with the tooltip "Edit Source Tracking". [stock only]
2. Hovering the item shows one JSON block for each of the last three changed files. [stock only]
3. The hover also holds a "View Details" link and a color legend of the edit sources. [stock only]
4. Clicking the item or the "View Details" link runs `editTelemetry.showDebugDetails`. [stock only]
5. The hover link "Toggle decorations: Click here" runs `editTelemetry.toggleDebugDecorations`, which flips `telemetry.editStats.showDecorations`. [stock only]
6. Setting `telemetry.editStats.showStatusBar` chooses whether the item is shown, with values on and off (default), and it is marked experimental. [stock only]
7. The item also needs `telemetry.editStats.enabled` on, which is its default, and usage telemetry to be allowed. [stock only]
8. Setting `telemetry.editStats.showDecorations` chooses whether colored backgrounds for each edit source are drawn in the editor, with values on and off (default). [stock only]

### SS-6.6
1. The right side of the bar has an item named "Cursor Tab" with the highest priority on the bar, so it sits left-most of the right-side items, and its text states and dimmed look are detailed in AC-8.1. [cursor only]
2. Hovering the item opens a panel, which is detailed in AC-8.2. [cursor only]
3. Clicking the item while signed in keeps the panel open (`statusBar.entry.showTooltip`), and while signed out it runs Log In to Cursor (`editor.cpp.login`), as detailed in AC-8.9. [cursor only]

### SS-6.7
1. A right-side item with a tab icon reads "Tab Stats: {accepted}/{suggested} ({percent}%)", for example "Tab Stats: 0/0 (0%)". [cursor only]
2. The hover reads "Tab AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". [cursor only]
3. The item is named "AI Code Tracking Stats - Tab" and its screen reader label says the same as the hover in words. [cursor only]
4. The item has no click action. [cursor only]
5. The item is added when a normal window starts, not in local mode, and no setting turns it off, so it can only be hidden from the right-click menu of the bar. [cursor only]

### SS-6.8
1. A right-side item with a speech bubble icon reads "Agent Stats: {accepted}/{suggested} ({percent}%)". [cursor only]
2. The hover reads "Agent AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". [cursor only]
3. The item is named "AI Code Tracking Stats - Agent" and sits just right of the Tab Stats item. [cursor only]
4. The item has no click action. [cursor only]
5. The item is added when a normal window starts and no setting turns it off, so it can only be hidden from the right-click menu of the bar. [cursor only]

### SS-6.9
1. A right-side item with a commit icon sits right-most of the three AI Code Tracking items and reads "No commit scored" until a commit has been scored, with the hover and screen reader label "No commit has been scored yet". [cursor only]
2. After a commit is scored the item reads the 7 character commit hash and "{n}% AI", for example "b90411c 100.00% AI". [cursor only]
3. The hover starts with "Most Recent Commit Scored:", the commit hash, and the commit message cut to 40 characters with "...". [cursor only]
4. The hover then shows a "Repo:" line and a "Branch:" line when they are known. [cursor only]
5. The hover ends with a rule line, "AI-Generated: {n}% ({lines} lines)", a "Tab:" line and a "Composer:" line each with added and deleted counts, and "Total Changes: {a} added, {d} deleted". [cursor only]
6. The item is named "AI Code Tracking - Recent Commit" and has no click action. [cursor only]
7. The item is added when a normal window starts and no setting turns it off. [cursor only]

### SS-6.10
1. The Command Palette lists "AI Code Tracking: Reset AI Code Tracking Stats" (`aiCodeTracking.resetDailyStats`) for everyone, with no precondition and no default key. [cursor only]
2. Running it sets today's Tab and Agent numbers back to zero in the current window. [cursor only]
3. The Tab Stats and Agent Stats items refresh to "0/0 (0%)" at once. [cursor only]
4. The recent commit item stays as it was. [cursor only]

### SS-6.11
1. A left-side item named "Workspace Name" shows the name of what you have open as plain text. [cursor only]
2. When the first folder has a source control repository, the text is the folder name if the repository root is that folder, and otherwise the name of the repository root folder. [cursor only]
3. With no repository the text is the workspace label. [cursor only]
4. The hover reads "Current workspace: {name}" and the screen reader label reads "Workspace: {name}". [cursor only]
5. The item has no click action. [cursor only]
6. The item is removed when there is no name. [cursor only]
7. The item updates when the workspace is renamed, when folders change, when a repository is added, removed or updated, and when label formats change. [cursor only]
8. A plain text item on the left side shows the repository folder name of the first workspace folder. [cursor only]
9. The item refreshes when workspace folders or repositories change. [cursor only]

### SS-6.12
1. A left-side item shows a row of siren and warning emoji around the words "HIGH MEMORY USAGE". [cursor only]
2. The item is named "Memory WARNING Indicator" or "Memory CRITICAL Indicator" to match the level. [cursor only]
3. The hover and the screen reader label read "{WARNING or CRITICAL} Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot." [cursor only]
4. The item has no click action. [cursor only]
5. The item appears only while the memory monitor flag `show_status_entry` is on and the memory use is over the warning level, and it goes away when use drops. [cursor only]
6. The flag comes from the app's server config and not from a user setting, and its built-in fallback is off, so a normal user may never see the item. [cursor only]
7. When the memory use of the window passes the warning threshold and a remote config flag allows it, a left item shows siren and warning emoji around the words "HIGH MEMORY USAGE". [cursor only]
8. The hover gives the memory used in MB, the threshold in MB, and a hint to take a heap snapshot. [cursor only]
9. The memory monitor flag `show_status_entry` comes from the app's server config and is not a user setting, so there is nothing in Settings to turn the item on or off. [cursor only]

### SS-6.13
1. In a development build, or for a developer user, a right-side item in a prominent text color reads "Server: {name}" with the hover "Click to switch backend server". [cursor only]
2. Clicking the item runs Select Backend Server (`cursor.selectBackend`), which opens a pick list with the placeholder "Select backend server to use". [cursor only]
3. The row of the server in use is marked "Current". [cursor only]
4. Choosing a row switches the server and may ask you to sign in again. [cursor only]
5. Select Backend Server is listed in the Command Palette only when `isDevelopment` or `isDevUser` is true. [cursor only]
6. In a production build for a non-developer the command only warns "Backend switching is not available in production builds." [cursor only]
7. Nine more Command Palette commands titled "Switch to {name} Backend" switch the server without showing the list. [cursor only]
8. The nine commands have the chords `WinCtrl+Shift+Alt+1` to `WinCtrl+Shift+Alt+9`, for prod, staging, stagingLocalWebsite, local, fullLocal, fullLocalincludingcpp, localExceptEmbeddings, devStagingEverything and localStaging in that order, where WinCtrl is the Windows key on Windows and Ctrl on macOS. [cursor only]

### SS-6.14
1. For internal users with the feature gate `internal_session_recording_status_bar`, a right-side item "[Internal] Report" with a bug icon appears, with the hover "Report a bug or lag with automatic session context". [cursor only]
2. The item acts when the mouse button goes down, not on a click. [cursor only]
3. Pressing the item runs Report Bug or Lag (`workbench.action.sessionRecording.reportMenu`), which opens a pick list titled "[Internal] Report" with the placeholder "What do you want to report?". [cursor only]
4. The list has two rows: "Report Bug" with the detail "Flag this session and add context." and "Report Lag" with the detail "Send a one-click lag report." [cursor only]
5. The "Report Bug" row runs `workbench.action.sessionRecording.flagAndAddContext`, which opens a report dialog. [cursor only]
6. The "Report Lag" row runs `workbench.action.sessionRecording.reportLag`. [cursor only]
7. All three commands are listed in the Command Palette under the Developer category when `isDevUser` is true. [cursor only]

### SS-6.15
1. In a development build, running "AI Tracking: Score Commit SHA for AI Content" (`developer.aiCodeTracking.scoreCommit`) opens an input box titled "Score Commit for AI Content" with the placeholder "Enter commit hash (e.g., abc123...)". [cursor only]
2. When more than one repository is open the command asks which repository to use. [cursor only]
3. The command then asks "Report metrics to analytics?" with the choices No and Yes. [cursor only]
4. The command shows "Scoring commit ..." and then the notice "Commit scored! AI: {n}% ({a}/{b} lines). Check output channel for details." and writes the details to the "aiCodeTracking" Output channel. [cursor only]
5. "AI Tracking: Score Recent Commits" (`developer.aiCodeTracking.scoreMultipleCommits`) scores several recent commits from the Command Palette. [cursor only]
6. "[Debug] Score Commit for AI Content" (`developer.aiCodeTracking.scoreCommitInternal`) is not in the Command Palette and shows as a row in the right-click menu of a Source Control history item, in group `1_ai`. [cursor only]
7. "[Debug] Score Commit with Details" (`developer.aiCodeTracking.scoreCommitWithDetails`) is a hidden command that other code calls. [cursor only]
8. All four commands and the menu row exist only in a development build (`isDevelopment`). [cursor only]

### SS-6.16
1. In a development build, running "AI Tracking: Reveal AI Code Tracking Paths" (`developer.aiCodeTracking.revealPaths`) writes the location of the tracking database file and the names of its key groups to the "aiCodeTracking" Output channel and shows that channel. [cursor only]
2. The command then posts the notice "AI Code Tracking paths have been written to the output channel." [cursor only]
3. If the Output channel is missing the command posts a warning instead. [cursor only]
4. The command is listed in the Command Palette only in a development build (`isDevelopment`). [cursor only]

## Unclear
- SS-6.6 is a cross-reference to AC-8, so its three pieces cover only the bar placement, the hover and the click, and the full count for the Cursor Tab item belongs to AC-8.
- SS-6.11 repeats SS-4.17 (same Workspace Name item) and SS-6.12 repeats SS-5.17 (same high memory item), so their pieces appear in two files each. One feature of each pair should be dropped.
- SS-6.1 piece 8 states the Cursor row from the pass 2 reading of the bundle, since the row was never seen on screen in Cursor.
- SS-6.5 piece 4 names `editTelemetry.showDebugDetails` as pass 2 did, but no registration for it was found, so what the click shows is not known.
- SS-6.13 to SS-6.16 are developer features that no catalog group clearly owns, so a later pass may move them.
- Every piece of the Cursor only features (SS-6.6 to SS-6.16) and of SS-6.2 to SS-6.5 (stock only) carries the matching tag, since each whole feature is one side only.
