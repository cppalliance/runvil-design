Group: SS-6 AI suggestions and AI stats. Section: Status bar. Date: 2026-10-25. Pass: 2.
Features: 16.

### SS-6.1 Inline suggestions row in the Language Status popup
- Tag: both
- Evidence: source
- Boundary: yes - the row says whether the suggestion provider has a suggestion right now, which is provider output. Only the row is listed.
- What: A row named "Inline Suggestions" (detail text "Inline suggestions") inside the Language Status popup, the bracket icon on the right of the bar that SS-3 owns. Its label follows the state of the editor: "Loading..." with a spinner, "Inline completion available", "Inline edit available", or "No inline suggestion available", the last three with a Copilot icon in front. The row has no command link. Like every language status row it has an "Add to Status Bar" pin, and once pinned it gets its own bar item that shows only an icon: a lightbulb, a lightbulb with a sparkle, or a circle with a slash (the busy spinner while loading). It is added only for editors that are not read-only. Stock hides it when AI features are turned off by the setting below. In Cursor the same labels exist, but the code is an editor contribution that works per editor and only for the focused one, and I found no AI-hidden check in it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: chat.disableAIFeatures
- Cursor: changed (same labels; class id `vs.editor.contrib.inlineCompletionLanguageStatusBarContribution`, focused editor only, no AI-hidden check found; `chat.disableAIFeatures` has 0 hits)
- Source: V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletionLanguageStatusBarContribution.ts; V\src\vs\workbench\contrib\inlineCompletions\browser\inlineCompletions.contribution.ts; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts; V\src\vs\editor\contrib\inlineCompletions\browser\model\inlineCompletionsModel.ts; V\src\vs\editor\contrib\inlineCompletions\browser\controller\inlineCompletionsController.ts; V\src\vs\workbench\services\chat\common\chatEntitlementService.ts; C bundle (NLS module `contrib/inlineCompletions/browser/inlineCompletionLanguageStatusBarContribution`, messages 7443 to 7448, `addStatus({...id:"inlineSuggestions"...})`)

### SS-6.2 AI usage gauge in the status bar
- Tag: vscode
- Evidence: source
- Boundary: yes - the gauge shows how much of the work came from AI.
- What: A right-side status bar item (priority 100, name "Inline Suggestions", aria label "Inline suggestions status bar") that has no text. It draws a thin bar, 50 by 6 pixels with rounded ends, that fills from the left to show the current AI share. It exists only when the setting `editor.aiStats.enabled` is on (default off, marked experimental) and AI features are not hidden. It has no click command. Hovering it opens the panel described in SS-6.3. It shares priority 100 with File Information, so their order comes from the id hash.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.aiStats.enabled; chat.disableAIFeatures
- Cursor: missing (0 hits for `editor.aiStats.enabled`, `aiStatsStatusBar` and the label "AI Usage Statistics")
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsFeature.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetry.contribution.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetryContribution.ts; V\src\vs\workbench\contrib\editTelemetry\browser\settingIds.ts

### SS-6.3 Read the AI usage chart in the gauge hover
- Tag: vscode
- Evidence: source
- Boundary: yes - the panel shows AI usage numbers.
- What: Hovering the gauge shows a panel headed "AI Usage Statistics" with two lines, "AI vs Typing Average: {n}%" and "Accepted inline suggestions today: {n}". Below a divider is a chart with a header that reads "AI Rate by Day" or "AI Rate by Session". Two small icon buttons at the right of that header, Days (calendar icon) and Sessions (flat list icon), switch the chart, and the active one is highlighted. An empty chart says "No data yet". The panel is built as an element tooltip, so it is rich content in the bar's hover rules (SS-1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: editor.aiStats.enabled
- Cursor: missing (0 hits for `AI Usage Statistics`, "AI vs Typing Average" and `aiStatsStatusBar`)
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsChart.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editStats\media.css

### SS-6.4 Open the AI stats setting from the gauge hover
- Tag: vscode
- Evidence: source
- Boundary: yes - it sits in the AI usage panel and only opens a setting.
- What: The header of the AI Usage Statistics panel has a gear icon button with the tooltip "Configure". Clicking it opens the Settings editor filtered to the `editor.aiStats.enabled` setting (the Settings editor itself is ED-8).
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: editor.aiStats.enabled
- Cursor: missing (the panel and `editor.aiStats.enabled` have 0 hits; the command id `workbench.action.openSettings` appears in the bundle for other callers)
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\editStats\aiStatsStatusBar.ts

### SS-6.5 Developer: Edit Source Tracking item
- Tag: vscode
- Evidence: source
- Boundary: yes - the hover lists where edits came from, including AI edits.
- What: Developer item, off by default. When `telemetry.editStats.showStatusBar` is on, and `telemetry.editStats.enabled` is on (default on) and usage telemetry is allowed, a right-side item (id `editTelemetry`, priority 100, tooltip "Edit Source Tracking") reads "{n} chars inserted" with an edit icon. Its hover shows one JSON block per changed file (the last three), a "View Details" link, a color legend of edit sources, and a "Toggle decorations: Click here" link. The item click and "View Details" run `editTelemetry.showDebugDetails`, which I could not find registered anywhere (see Unclear). "Toggle decorations" runs `editTelemetry.toggleDebugDecorations`, which flips `telemetry.editStats.showDecorations`.
- Commands: editTelemetry.showDebugDetails; editTelemetry.toggleDebugDecorations
- Menus: none found
- Keys: none found
- Settings: telemetry.editStats.showStatusBar; telemetry.editStats.enabled; telemetry.editStats.showDecorations
- Cursor: missing (0 hits for `telemetry.editStats` and "Edit Source Tracking")
- Source: V\src\vs\workbench\contrib\editTelemetry\browser\telemetry\editSourceTrackingFeature.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetry.contribution.ts; V\src\vs\workbench\contrib\editTelemetry\browser\settings.ts; V\src\vs\workbench\contrib\editTelemetry\browser\editTelemetryContribution.ts

### SS-6.6 Cursor Tab status item (cross-reference only)
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cross-reference only. Cursor Tab shows in the bar as a right-side item named "Cursor Tab" with priority 1000, the highest on the bar, so it sits left-most of the right-side items. Its text is "Cursor Tab", "Cursor Tab (Snoozed)" or "Cursor Tab (Ignored)", and it is dimmed when Tab is off for the open file. Hovering opens a panel and clicking it while signed in keeps the panel open, while signed out the click runs Log In to Cursor. Nothing about the item is described again here. The controls are in AC-8 - AC-8.1 text states and dimmed look, AC-8.2 the hover panel, AC-8.3 turn Tab on or off, AC-8.4 turn Tab off for a language, AC-8.5 pick the Tab model, AC-8.6 snooze, AC-8.7 unsnooze and time left, AC-8.8 why Tab is not available, AC-8.9 log in. Hiding the item from the bar's right-click menu is SS-1. Seen running by the pass 1 status bar check as "Cursor Tab".
- Commands: statusBar.entry.showTooltip; editor.cpp.login
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`addEntry(this.statusBarElementProps(),"Cursor Tab",1,1e3)`; entry fields `name:"Cursor Tab"`, `cssClass`, `tooltip`; class `copilot-plus-plus-status-off`); V has no entry for it

### SS-6.7 Tab Stats item
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the numbers show Cursor Tab use.
- What: A right-side item named "AI Code Tracking Stats - Tab" (priority 100) with a tab icon. The text reads "Tab Stats: {accepted}/{suggested} ({percent}%)", for example "Tab Stats: 0/0 (0%)", and the hover reads "Tab AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". The aria label says the same in words. It has no click command. The tooltip names today, and there is a command to zero it (SS-6.10). The three AI Code Tracking items come from one class that adds them at startup in normal windows, not in local mode and not under the smoke test driver. I found no setting that turns the item off, so hiding it is the bar's right-click menu (SS-1). It shares priority 100 with File Information, so their order comes from the id hash.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`; `addEntry(this.tabEntry(),"aiCodeTracking.stats.tab",1,100)`; `tabEntry()` fields)

### SS-6.8 Agent Stats item
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the numbers show agent use.
- What: A right-side item named "AI Code Tracking Stats - Agent" (priority 99) with a speech bubble icon. The text reads "Agent Stats: {accepted}/{suggested} ({percent}%)" and the hover reads "Agent AI Stats (Today): {accepted}/{suggested} lines ({percent}%)". It sits just right of Tab Stats. No click command. Same creation rules and same lack of an off switch as SS-6.7. Seen running by the pass 1 status bar check.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`; `addEntry(this.composerEntry(),"aiCodeTracking.stats.composer",1,99)`; `composerEntry()` fields)

### SS-6.9 Recent commit AI score item
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the item reports how much of a commit came from Tab and the agent.
- What: A right-side item named "AI Code Tracking - Recent Commit" (priority 98) with a commit icon, the right-most of the three AI Code Tracking items. Until a commit is scored it reads "No commit scored", and the hover and aria label say "No commit has been scored yet". After that it reads the 7 character commit hash and "{n}% AI" (seen running as "b90411c 100.00% AI"). The hover is a block of lines: "Most Recent Commit Scored:", the hash, the commit message cut to 40 characters with "...", a "Repo:" line and a "Branch:" line when known, a rule line, "AI-Generated: {n}% ({lines} lines)", a "Tab:" line and a "Composer:" line each with added and deleted counts, and "Total Changes: {a} added, {d} deleted". The hover still uses the older word Composer where the bar says Agent. No click command. Same creation rules as SS-6.7.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`; `addEntry(this.recentCommitEntry(),"aiCodeTracking.stats.recentCommit",1,98)`; `recentCommitEntry()` fields)

### SS-6.10 Reset AI Code Tracking Stats
- Tag: cursor
- Evidence: bundle
- Boundary: yes - it clears the counters of Tab and agent use.
- What: Command Palette command "AI Code Tracking: Reset AI Code Tracking Stats" (category "AI Code Tracking"). It sets today's numbers for Tab and Agent back to zero in the current window, and the Tab Stats and Agent Stats items refresh to "0/0 (0%)" at once. In the code I read it does not touch the recent commit item. It shows in the palette for everyone, with no precondition and no default key.
- Commands: aiCodeTracking.resetDailyStats
- Menus: CommandPalette (f1 true)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingStatusBar`, messages 4831 and 4832; class with `id:"aiCodeTracking.resetDailyStats"`, `f1:!0`; `resetDailyStats()`)

### SS-6.11 Workspace name item
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A left-side item named "Workspace Name" (priority 50) that shows the name of what you have open as plain text, with the hover "Current workspace: {name}" and the aria label "Workspace: {name}". It has no click command. When the first folder has a source control repository, the text is the folder name if the repository root is that folder, otherwise the name of the repository root folder. With no repository it is the workspace label. The item is removed when there is no name, and it updates when the workspace is renamed, folders change, a repository is added, removed or updated, or label formats change. Seen running as "cursor".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/workspace/browser/workspace.view.contribution`, messages 11884 to 11886; class with `entryId="status.workspaceName"`, `addEntry(t,this.entryId,0,50)`); V has no hit for `status.workspaceName`; C bundle (module vs/workbench/contrib/workspace/browser/workspace.view.contribution, class that registers id status.workspaceName at priority 50, NLS keys status.workspaceName, status.workspaceNameAria, status.workspaceNameTooltip; searched V and N and found no such id)

### SS-6.12 High memory warning item
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A left-side item that shows a row of siren and warning emoji around the words "HIGH MEMORY USAGE". Its name is "Memory WARNING Indicator" or "Memory CRITICAL Indicator", and its hover and aria label read "{WARNING or CRITICAL} Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot." It has no click command. It appears only when the memory monitor config flag `show_status_entry` is on and the memory use is over the warning level, and it goes away when use drops. The built-in fallback for that flag is off (fallback base threshold 1536 MB, critical offset 512 MB), and the flag is part of the app's server-sent config, not a user setting, so a normal user may never see it. Not seen running.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution`; `addEntry(p,"workbench.memoryMonitor.indicator",0)`; `memory_monitor_config` with fallback values `show_status_entry:!1`); C bundle (module vs/workbench/contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution, id workbench.memoryMonitor.indicator, config flag show_status_entry, tooltip "Process Memory Usage (JS: {n}MB) exceeds threshold ({m}MB). Consider taking a heap snapshot."; searched V and N and found no such id)

### SS-6.13 Developer: see and switch the backend server
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Developer only. In a development build, or for a developer user, a right-side item (name "currentServer", priority 200, prominent text color) reads "Server: {name}" with the hover "Click to switch backend server". Clicking runs `cursor.selectBackend` ("Select Backend Server" in the palette), which opens a pick list with the placeholder "Select backend server to use" and the current row marked "Current". Choosing a row switches the server and may ask you to sign in again. In a production build for a non-developer the command only warns "Backend switching is not available in production builds." Nine more palette commands titled "Switch to {name} Backend" do the same switch without the list, each with a chord. The item ties with the Git blame item at priority 200, so their order comes from the id hash.
- Commands: cursor.selectBackend; debug.switchToprodBackend; debug.switchTostagingBackend; debug.switchTostagingLocalWebsiteBackend; debug.switchTolocalBackend; debug.switchTofullLocalBackend; debug.switchTofullLocalincludingcppBackend; debug.switchTolocalExceptEmbeddingsBackend; debug.switchTodevStagingEverythingBackend; debug.switchTolocalStagingBackend
- Menus: CommandPalette (f1 true, precondition isDevelopment or isDevUser)
- Keys: WinCtrl+Shift+Alt+1 (debug.switchToprodBackend); WinCtrl+Shift+Alt+2 (staging); WinCtrl+Shift+Alt+3 (stagingLocalWebsite); WinCtrl+Shift+Alt+4 (local); WinCtrl+Shift+Alt+5 (fullLocal); WinCtrl+Shift+Alt+6 (fullLocalincludingcpp); WinCtrl+Shift+Alt+7 (localExceptEmbeddings); WinCtrl+Shift+Alt+8 (devStagingEverything); WinCtrl+Shift+Alt+9 (localStaging); WinCtrl is the Windows key on Windows and Ctrl on macOS; no `when`, weight 200
- Settings: none found
- Cursor: cursor only
- Source: C bundle (`updateServerStatusBarOnlyRunsOnLocal`, `addEntry(t,"status.currentServer",1,200)`; class with `id:"cursor.selectBackend"`; factory `fQ_()` that builds `debug.switchTo${name}Backend`; key table with `Digit1` to `Digit9`; module name `cursorCredsService.js`)

### SS-6.14 Developer: report a bug or lag from the status bar
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Internal only. A right-side item "[Internal] Report" (priority 50) with a bug icon and the hover "Report a bug or lag with automatic session context". It shows only for internal users with the feature gate `internal_session_recording_status_bar`, and it acts on mouse down, not on click. It runs `workbench.action.sessionRecording.reportMenu` ("Report Bug or Lag"), which opens a pick list titled "[Internal] Report" with the placeholder "What do you want to report?" and two rows: "Report Bug" (detail "Flag this session and add context.") and "Report Lag" (detail "Send a one-click lag report."). Each row runs its own command: `workbench.action.sessionRecording.flagAndAddContext` ("Report Bug", opens a report dialog that is not described here) and `workbench.action.sessionRecording.reportLag` ("Report Lag"). All three commands are in the palette under the Developer category when the context key `isDevUser` is true.
- Commands: workbench.action.sessionRecording.reportMenu; workbench.action.sessionRecording.flagAndAddContext; workbench.action.sessionRecording.reportLag
- Menus: CommandPalette (f1 true, precondition isDevUser, category Developer)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS messages 11403 to 11417; class `workbench.contrib.sessionRecordingStatusBar` with `addEntry(e,"status.sessionRecording",1,50)` and `commandExecutionTrigger:"mousedown"`; `shouldShow()` gate)

### SS-6.15 Developer: score a commit for AI content
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the results report how much of a commit came from Tab and the agent.
- What: Developer commands that work only in a development build (context key `isDevelopment`). "AI Tracking: Score Commit SHA for AI Content" opens an input box (title "Score Commit for AI Content", placeholder "Enter commit hash (e.g., abc123...)"), asks which repository when there is more than one, asks "Report metrics to analytics?" with No and Yes, shows "Scoring commit ..." and then a notice "Commit scored! AI: {n}% ({a}/{b} lines). Check output channel for details.", and writes the details to the "aiCodeTracking" Output channel. "AI Tracking: Score Recent Commits" scores several recent commits (its screens were not read). "[Debug] Score Commit for AI Content" is not in the palette and instead appears as a row in the right-click menu of a Source Control history item. "[Debug] Score Commit with Details" is a hidden command that other code calls.
- Commands: developer.aiCodeTracking.scoreCommit; developer.aiCodeTracking.scoreMultipleCommits; developer.aiCodeTracking.scoreCommitInternal; developer.aiCodeTracking.scoreCommitWithDetails
- Menus: CommandPalette (f1 true, precondition isDevelopment); SCMHistoryItemContext (group 1_ai, order 1, when isDevelopment)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (NLS module `contrib/analytics/browser/aiCodeTrackingActions.contribution`, messages 4829 and 4830; classes with the four ids; context key `isDevelopment`); V has no hit

### SS-6.16 Developer: reveal AI Code Tracking paths
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Developer command "AI Tracking: Reveal AI Code Tracking Paths" (development build only, context key `isDevelopment`). It writes the location of the tracking database file and the names of its key groups to the "aiCodeTracking" Output channel, shows that channel, and posts a notice "AI Code Tracking paths have been written to the output channel." If the channel is missing it posts a warning instead.
- Commands: developer.aiCodeTracking.revealPaths
- Menus: CommandPalette (f1 true, precondition isDevelopment)
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class with `id:"developer.aiCodeTracking.revealPaths"`, category "AI Tracking", module `contrib/analytics/browser/aiCodeTrackingActions.contribution`); V has no hit

## Handed off
- Pinning or unpinning the Inline Suggestions row ("Add to Status Bar" and "Remove from Status Bar") and the combined Language Status item it lives in: SS-3.
- Hiding or showing any item in this group from the bar's right-click menu, and where each one sits in the left and right order: SS-1.
- The warning notification with a "Capture Snapshot" button that the memory monitor raises (the memory warning item in SS-6.12 is its status bar side): NT-2.
- The Command Palette command "Developer: Capture and Upload Renderer Heap Snapshot" (`developer.captureAndUploadRendererHeapSnapshot`, palette for everyone, runs `memoryMonitor.triggerManualSnapshot`): no group owns Developer category commands, closest is QA-1 (low confidence).
- The "[Debug] Score Commit for AI Content" row in the right-click menu of a Source Control history item (`SCMHistoryItemContext`, group 1_ai, development build only): SB-4. The command is listed in SS-6.15, the menu row is for SB-4 to place.
- Colored backgrounds drawn in the text editor for each edit source, switched by `telemetry.editStats.showDecorations` from the Edit Source Tracking hover: ED-5 (low confidence, no ED group is a clean fit).
- The label class `inlineSuggestionStatusBarItemLabel`, which the Cursor bundle uses on the action bar items of the inline suggestions hover toolbar. It is an editor toolbar, not a window status bar item: AC-2.

## Unclear
- Whether the Inline Suggestions row (SS-6.1) shows in Cursor at runtime. The labels are in the bundle and the code runs per focused editor, but I did not see it on screen and did not trace what feeds the editor state when Cursor Tab is the suggestion source.
- `editTelemetry.showDebugDetails` is the command behind the Edit Source Tracking item click and its "View Details" link, but I found no registration for it in V source, V extensions, or N source (only the string in `editSourceTrackingFeature.ts`). In stock the click may do nothing, or the id may be registered by a Microsoft extension that is not in these trees.
- The Copilot Status entry (`chat.statusBarEntry`) and the Copilot extension's own bar items (for example `extensions\copilot\src\extension\completions-core\vscode-node\extension\src\statusBar.ts`) are chat or extension surfaces, so they are not listed. Cursor has 0 hits for `chat.statusBarEntry` and "Copilot Status".
- Whether Tab Stats, Agent Stats and the recent commit item show on a fresh Cursor profile. The code adds them at startup with no setting to turn them off, and the pass 1 screenshot is of a working window, so the default state is unverified.
- What turns on the high memory warning item. The flag `show_status_entry` falls back to off and comes from server config that I did not trace.
- Who counts as a developer user or an internal user, which decides whether SS-6.13 and SS-6.14 ever show for anyone outside Cursor staff. The code gates were read, not exercised.
- The Report Bug dialog that the "Report Bug" row opens (SS-6.14) was not traced. No group in the catalog clearly owns it, DL-1 is the nearest.
- SS-6.13 to SS-6.16 are developer commands and items. They sit next to the AI Code Tracking items in the bundle, so I kept them in this group, but the catalog has no group for Developer category commands, so a later pass may move them.
- Item order when priorities tie (Tab Stats and File Information at 100, the Server item and Git blame at 200) comes from the id hash rule in the bar code. I did not check the order on screen.
- Counts and labels are read from code and strings. Nothing here was run, apart from the three AI Code Tracking items and the workspace name that the pass 1 status bar check saw in the owner's window.
