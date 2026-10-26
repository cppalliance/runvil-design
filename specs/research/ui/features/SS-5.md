Group: SS-5 Alerts and background activity. Section: Status bar. Date: 2026-10-25. Pass: 2.
Features: 15.

### SS-5.1 Notifications bell with unread dot
- Tag: both
- Evidence: source
- Boundary: no
- What: A bell icon sits at the far right end of the bar (id status.notifications, name "Notifications", priority negative infinity); it turns into a bell with a dot when a notification arrived that you have not seen in the center, or when a notification with progress is running while the toasts and center are hidden, and opening the center clears the unread count; the tooltip reads "No Notifications", "No New Notifications", "1 New Notification", "{n} New Notifications", with "({n} in progress)" variants, and "Hide Notifications" while the center is open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (updateNotificationsCenterStatusItem, getTooltip, update); C bundle (module vs/workbench/browser/parts/notifications/notificationsStatus, id status.notifications on the right at negative infinity, icons bell and bell-dot, NLS keys noNotifications, oneNotification, notifications, noNotificationsWithProgress and the other progress variants)

### SS-5.2 Click the bell to show or hide the notification center
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the bell opens the notification center above it, with a beak on the item pointing at the center, and clicking again hides it; Show Notifications also runs from the Command Palette or the key chord, and Escape hides the center while it is open.
- Commands: notifications.showList; notifications.hideList
- Menus: commandPalette
- Keys: ctrl+k ctrl+shift+n (notifications.showList); escape when NotificationsCenterVisibleContext (notifications.hideList)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (command and showBeak); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts (SHOW_NOTIFICATIONS_CENTER, HIDE_NOTIFICATIONS_CENTER, keybindings, palette items); C bundle (commands notifications.showList with CtrlCmd+K then CtrlCmd+Shift+N, notifications.hideList on Escape, labels "Show Notifications" and "Hide Notifications")

### SS-5.4 Status message text
- Tag: both
- Evidence: source
- Boundary: no
- What: Workbench code can show a short text message in the bar (id status.message, name "Status Message", last item on the left), after an optional delay and for an optional time, replacing any earlier message; examples are "The extension host terminated unexpectedly. Restarting..." for 5 seconds and the message shown when an extension changes workspace folders for 10 seconds. It has no icon and no click action.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (doSetStatusMessage); V\src\vs\workbench\services\notification\common\notificationService.ts (status); V\src\vs\workbench\services\extensions\electron-browser\nativeExtensionService.ts; V\src\vs\workbench\api\browser\mainThreadWorkspace.ts; C bundle (id status.message, NLS key status.message "Status Message" in module vs/workbench/browser/parts/notifications/notificationsStatus)

### SS-5.5 Problem counts
- Tag: both
- Evidence: source
- Boundary: no
- What: A left item (id status.problems, name "Problems", priority 50) shows an error icon with its count and a warning icon with its count, plus an info icon and count only when there are infos; counts above 999 shorten to "1K" to "9K" and above 9999 to "10K+", and the hover reads "Errors: 2, Warnings: 1" or "No Problems".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (MarkersStatusBarContributions.getMarkersItem, getMarkersText, getMarkersTooltip, packNumber); C bundle (module vs/workbench/contrib/markers/browser/markers.contribution, id status.problems, NLS keys totalErrors, totalWarnings, totalInfos, noProblems, manyProblems)

### SS-5.6 Open or close the Problems view from the item
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the Problems item opens and focuses the Problems panel, or closes it when it is already visible.
- Commands: workbench.actions.view.toggleProblems
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (TOGGLE_MARKERS_VIEW_ACTION_ID action, item command); V\src\vs\workbench\contrib\markers\common\markers.ts; C bundle (command workbench.actions.view.toggleProblems on the status.problems item)

### SS-5.7 Problems visibility off warning item
- Tag: both
- Evidence: source
- Boundary: no
- What: When problems are turned off, a warning-colored whole-word icon item (id status.problemsVisibility, name "Problems Visibility", priority 49) appears right of the counts with the tooltip "Problems are turned off. Click to open settings." and a click opens Settings filtered to problems.visibility; it disappears when the setting is turned back on.
- Commands: workbench.action.openSettings
- Menus: none found
- Keys: none found
- Settings: problems.visibility
- Cursor: present
- Source: V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (getMarkersItemTurnedOff, addStatusBarEntry); V\src\vs\workbench\browser\workbench.contribution.ts (problems.visibility); C bundle (id status.problemsVisibility, icon whole-word, kind warning, NLS keys status.problemsVisibility and status.problemsVisibilityOff)

### SS-5.8 Current problem text
- Tag: both
- Evidence: source
- Boundary: no
- What: With problems.showCurrentInStatus on (off by default) the left side shows the severity icon and first line of the problem under the cursor (id statusbar.currentProblem, name "Current Problem"); it disappears when the cursor leaves the problem and has no click action.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: problems.showCurrentInStatus
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\editor\editorStatus.ts (ShowCurrentMarkerInStatusbarContribution); V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts (setting, default false); C bundle (id statusbar.currentProblem, setting problems.showCurrentInStatus default false, severity icons error, warning, info)

### SS-5.9 Debug item shows the selected launch configuration
- Tag: both
- Evidence: source
- Boundary: no
- What: A left item with a debug icon and the name of the selected launch configuration (with the launch file name added when there is more than one) appears with the tooltip "Select and Start Debug Configuration" (id status.debug, name "Debug", priority 30); by default it shows only after the first debug session starts, and a setting makes it always or never show.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.showInStatusBar
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugStatus.ts (DebugStatusContribution); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (debug.showInStatusBar, default onFirstSessionStart); C bundle (id status.debug at priority 30, setting debug.showInStatusBar with never, always, onFirstSessionStart)

### SS-5.10 Click the debug item to pick and start a configuration
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the Debug item runs Select and Start Debugging, a picker of the launch configurations and dynamic debug providers; choosing one selects it and starts it.
- Commands: workbench.action.debug.selectandstart
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\debug\browser\debugStatus.ts (command); V\src\vs\workbench\contrib\debug\browser\debugCommands.ts (SELECT_AND_START_ID); V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts (registerDebugCommandPaletteItem); C bundle (command workbench.action.debug.selectandstart on the status.debug item)

### SS-5.11 Running Tasks count
- Tag: both
- Evidence: source
- Boundary: no
- What: While tasks are running a tools icon with the count appears right of the Problems item (id status.runningTasks, name "Running Tasks", tooltip "Show Running Tasks"); clicking it opens the Show Running Tasks picker, and the item disappears when no task is running.
- Commands: workbench.action.tasks.showTasks
- Menus: MenubarTerminalMenu (group Manage, "Show Running Tasks...")
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (TaskStatusBarContributions._updateRunningTasksStatus, menu items); C bundle (id status.runningTasks placed next to status.problems at priority 50, command workbench.action.tasks.showTasks)

### SS-5.12 Progress message with a spinner
- Tag: both
- Evidence: source
- Boundary: no
- What: Work that reports window progress, such as a running build task ("Building...") or a language feature, puts a spinning item at the far end of the left side (id status.progress, name "Progress Message") with "title: message" text and a tooltip that adds the source in brackets; it appears after 150 ms, stays at least 150 ms, and a click runs the command the work gave (the build item opens Show Running Tasks).
- Commands: workbench.action.tasks.showTasks
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withWindowProgress, updateWindowProgress); V\src\vs\workbench\contrib\tasks\browser\task.contribution.ts (Building... progress); C bundle (id status.progress at negative max value, window progress location 10, "Building..." message with command workbench.action.tasks.showTasks)

### SS-5.13 Window zoom indicator and mini toolbar
- Tag: both
- Evidence: source
- Boundary: no
- What: On the right side a prominent zoom-out or zoom-in icon appears whenever this window's zoom differs from the configured level (id status.windowZoom, name "Window Zoom", priority 102); its hover shows a mini toolbar with Zoom Out, the zoom level number (tooltip "Zoom Level: 2 (144%)"), Zoom In, Reset and a Settings gear that opens window.zoom, and every window has its own item. Desktop only.
- Commands: workbench.action.zoomIn; workbench.action.zoomOut; workbench.action.zoomReset; workbench.action.openSettings
- Menus: none found
- Keys: ctrl+= (also ctrl+shift+= and ctrl+numpad+) for zoom in; ctrl+- (also ctrl+shift+- and ctrl+numpad-) for zoom out; ctrl+numpad0 for reset
- Settings: window.zoomLevel; window.zoomPerWindow
- Cursor: present
- Source: V\src\vs\workbench\electron-browser\window.ts (ZoomStatusEntry, updateWindowZoomStatusEntry); V\src\vs\workbench\electron-browser\actions\windowActions.ts (zoom commands and keys); V\src\vs\workbench\electron-browser\desktop.contribution.ts (settings); C bundle (module vs/workbench/electron-sandbox/window, id status.windowZoom kind prominent priority 102, icons zoom-in and zoom-out, NLS key zoomNumber, commands and settings)

### SS-5.14 Show Output quietly indicator
- Tag: both
- Evidence: source
- Boundary: no
- What: When the Output panel is marked in workbench.view.showQuietly and an extension asks to show an output channel while the panel is hidden, a prominent output icon appears on the right next to the bell (id status.view.showQuietly, name "Show Output", tooltip "Show {channel} Output Channel") and a click opens that channel; nothing is shown by default because the setting starts empty.
- Commands: workbench.action.output.show.{channelId}
- Menus: none found
- Keys: none found
- Settings: workbench.view.showQuietly
- Cursor: present
- Source: V\src\vs\workbench\api\browser\mainThreadOutputService.ts (_showChannelQuietly); V\src\vs\workbench\browser\workbench.contribution.ts (workbench.view.showQuietly); C bundle (id status.view.showQuietly placed left of status.notifications, setting workbench.view.showQuietly with workbench.panel.output)

### SS-5.15 Pty Host unresponsive warning
- Tag: both
- Evidence: source
- Boundary: no
- What: If the terminal's pty host process stops answering, an amber warning item with a disconnect icon and "Pty Host" appears on the left (id ptyHostStatus, name "Pty Host Status"); the tooltip says terminals may stop working and a click restarts the pty host, and the item goes away when the host answers again.
- Commands: workbench.action.terminal.restartPtyHost
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminal\browser\baseTerminalBackend.ts (onPtyHostUnresponsive entry); V\src\vs\workbench\contrib\terminalContrib\developer\browser\terminal.developer.contribution.ts (Restart Pty Host); C bundle (id ptyHostStatus, kind warning, command workbench.action.terminal.restartPtyHost, title "Restart Pty Host")

### SS-5.16 Terminal session recording item
- Tag: both
- Evidence: source
- Boundary: no
- What: Developer: Record Terminal Session opens a terminal and shows a spinning "Recording terminal session..." item on the left (id recordSession) until the terminal has been idle for 5 seconds, then copies the recording as JSON to the clipboard; the item has no click action.
- Commands: workbench.action.terminal.recordSession
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\contrib\terminalContrib\developer\browser\terminal.developer.contribution.ts (RecordSession); V\src\vs\workbench\contrib\terminalContrib\developer\common\terminal.developer.ts; C bundle (command workbench.action.terminal.recordSession, title "Record Terminal Session", entry id recordSession)

## Handed off

- The notification center list, toasts and Clear All -> NT-3, NT-1 and NT-2; the Capture Snapshot warning notification that comes with the memory item -> NT-1 and NT-2.
- Turning Do Not Disturb on, off or per source -> NT-4.
- Where the bell sits (bottom right, bottom left or top right through workbench.notifications.position, which Cursor lacks) and the title bar bell -> NT-3 and TB-5.
- Progress shown as a notification with Cancel -> NT-5.
- The Problems view, and turning problem markers off everywhere -> PN-4 and PN-4.19.
- The launch configuration picker and Run Task pickers that the Debug and Running Tasks items open -> QA-4; the Terminal menu row for Show Running Tasks -> TB-1; the debugging bar color -> SS-2.3 and SS-2.4.
- The Output panel and its channels -> PN-5.
- The zoom commands in the View menu -> TB-1; click to open the zoom tooltip -> SS-1.11; the prominent kind color -> SS-2.5.
- The terminal pty host restart and the Developer terminal commands as terminal features -> PN-2.
- Metered Connection and Performance Trace items (pass 1 listed them here) -> SS-3.37 and SS-3.36.

## Unclear

- No update available item exists on the status bar in V or in Cursor. Updates show in the title bar (TB-5) and as update cards (NT-6). I checked every addEntry call in the Cursor bundle and in V src.
- Screen recording: there is no screen recording item. The nearest items are the terminal session recording item (SS-5.16) and Cursor's internal "[Internal] Report" item (id status.sessionRecording, "Report a bug or lag with automatic session context", runs on mouse down), which only shows for internal users with a feature gate and is not a user feature. Cursor's internal "Server: {name}" item (id status.currentServer, command cursor.selectBackend) only shows in developer builds. Neither is listed.
- Extension Profiler item (id status.profiler, "Profiling Extension Host") and the Extension Status message from window.setStatusBarMessage (id status.extensionMessage) are left out: both belong to the extension host and extension API, which are out of scope. Debug auto attach (status.debug.autoAttach) and the js-debug profiling item come from debug support extensions and are also left out.
- The high memory item (SS-5.17) depends on a config flag show_status_entry whose source I did not trace, so it may not show for ordinary users. It was not seen running.
- The bell, Problems counts, workspace name and the zoom icon were seen running in Cursor in pass 1; everything else in Cursor comes from the bundle and NLS strings, not from running it.
- Newer than 1.128, details of existing items: in N (1.144.0) the bell decides "in progress" with notification.hasActiveProgress where V and Cursor use notification.hasProgress. I did not trace what the difference means on screen.
- In V the build progress item (SS-5.12) and the running tasks item (SS-5.11) share the same command; I listed the command on both because each item runs it on click.
