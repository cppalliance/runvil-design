Group: NT-5 Progress notifications. Section: Notifications. Date: 2026-10-25. Pass: 2.
Features: 13.

### NT-5.1 Progress notification
- Tag: both
- Evidence: source
- Boundary: no
- What: A task started with the notification progress location appears as a notification with a thin progress bar under the message. It is an endless bar until the task reports a total, then a bar that fills by percent.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; V\src\vs\platform\progress\common\progress.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\common\notifications.ts; C bundle (modules vs/workbench/services/progress/browser/progressService and vs/workbench/browser/parts/notifications/notificationsViewer, methods withNotificationProgress and renderProgress)

### NT-5.2 Progress title and message line
- Tag: both
- Evidence: source
- Boundary: no
- What: The progress title and the latest step message join as "title: message", and the title stands alone until a message arrives. The notification text may hold [text](command:...) links, which the status bar entry turns into plain text, and the status entry tooltip adds a [source] prefix.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (updateWindowProgress, updateNotification); C bundle (module vs/workbench/services/progress/browser/progressService, nls 13067, 13068, 13069)

### NT-5.3 Cancel and extra buttons on a progress notification
- Tag: both
- Evidence: source
- Boundary: no
- What: A progress notification can show a Cancel button (the default label or a custom string) plus named buttons. Clicking one stops the task and reports which button was used. The buttons sit under the message like any other notification buttons. The ids here are action ids built in the service, not registered commands.
- Commands: progress.cancel; progress.button.<label>
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (createNotification); V\src\vs\platform\progress\common\progress.ts; C bundle (module vs/workbench/services/progress/browser/progressService, ids progress.cancel and progress.button.{label}, nls 13072)

### NT-5.4 Progress notification stays until finished
- Tag: both
- Evidence: source
- Boundary: no
- What: A running progress notification never auto-hides, has no close button, ignores middle-click and the Delete key, and is skipped by Clear All Notifications, so you cannot dismiss it by accident. It also stays at least 800 ms so a quick task does not flash.
- Commands: notification.clear; notifications.clearAll
- Menus: CommandPalette
- Keys: Delete (notification.clear, when a notification has focus; Cmd+Backspace on macOS)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\common\notifications.ts (sticky getter); V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsCommands and notificationsViewer, the hasProgress checks, ids notification.clear and notifications.clearAll)

### NT-5.5 Indeterminate and determinate bars
- Tag: both
- Evidence: source
- Boundary: no
- What: The shared 2 px progress bar has two looks. Indeterminate sweeps endlessly and switches to a stepped animation after 10 seconds. Determinate fills left to right by the work done and grows to 100 percent before it hides. It is reused by notifications, views, containers and editor groups.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\base\browser\ui\progressbar\progressbar.ts; V\src\vs\workbench\services\progress\browser\progressIndicator.ts; C bundle (the ProgressBar class with LONG_RUNNING_INFINITE_THRESHOLD=1e4, css classes monaco-progress-container, infinite, infinite-long-running, discrete, done)

### NT-5.6 Window progress in the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Work reported at the window location shows a spinner entry on the left end of the status bar (name Progress Message, entry id status.progress) with the progress text. It appears after 150 ms and stays at least 150 ms. The icon is the loading spinner or the syncing icon, and an optional command runs when you click it. The entry's own look is also listed under the status bar group.
- Commands: notifications.showList
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withWindowProgress, updateWindowProgress); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts; V\src\vs\workbench\services\statusbar\browser\statusbar.ts; C bundle (module vs/workbench/services/progress/browser/progressService, entry id status.progress, nls 13071, showProgress with loading and syncing)
### NT-5.7 Hidden progress notification falls back to the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: When a progress notification is silent or its toast gets dismissed, the work keeps running as a window progress entry in the status bar. Clicking that entry opens the notification center, where the bar is shown. The entry disappears when the notification shows again or the task ends.
- Commands: notifications.showList; notifications.hideToasts
- Menus: none found
- Keys: Escape (notifications.hideToasts, when notificationToastsVisible)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (createWindowProgress, onVisibilityChange); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (module vs/workbench/services/progress/browser/progressService, the createWindowProgress arrow with command notifications.showList)

### NT-5.8 Progress bar in a view or container
- Tag: both
- Evidence: source
- Boundary: no
- What: Work reported against Explorer, Source Control, Extensions, a view id or a container id shows the thin progress bar at the top of that view, or at the top of the container content. The bar runs only while the view or container is visible and resumes when you switch back to it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withViewProgress, withPaneCompositeProgress, withCompositeProgress); V\src\vs\workbench\services\progress\browser\progressIndicator.ts (ScopedProgressIndicator); V\src\vs\workbench\browser\parts\views\viewPane.ts; V\src\vs\workbench\browser\parts\compositePart.ts; C bundle (modules vs/workbench/browser/parts/views/viewPane and vs/workbench/browser/parts/compositePart, getProgressIndicator)

### NT-5.9 Progress badge on the activity bar icon
- Tag: both
- Evidence: source
- Boundary: no
- What: Work on a side bar view or container also puts a small spinning progress badge on that container's icon after 300 ms, for at least 300 ms. A progress badge wins over icon and number badges while it runs. The badge look is also listed under the activity bar group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (showOnActivityBar); V\src\vs\workbench\services\progress\browser\media\progressService.css; V\src\vs\workbench\browser\parts\compositeBarActions.ts (getVisibleBadges); V\src\vs\workbench\services\activity\common\activity.ts (ProgressBadge); C bundle (module vs/workbench/services/progress/browser/progressService, showOnActivityBar, class progress-badge)

### NT-5.10 Progress in a dialog
- Tag: both
- Evidence: source
- Boundary: no
- What: Work reported at the dialog location opens a modal pending dialog with the message and an optional detail line. It has a Cancel or Dismiss button, the message updates as progress is reported, and sticky mode removes the button and blocks closing. An optional delay holds the dialog back. The dialog look is also listed under the dialogs group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withDialogProgress); V\src\vs\base\browser\ui\dialog\dialog.ts (pending type); C bundle (module vs/workbench/services/progress/browser/progressService, withDialogProgress with type pending, nls 13073, 13074)

### NT-5.11 Delayed and minimum display times
- Tag: both
- Evidence: source
- Boundary: no
- What: Progress avoids flicker. A notification with a delay option waits that long, otherwise it stays at least 800 ms. Window progress waits 150 ms and stays 150 ms. The activity bar badge waits 300 ms (or the caller's delay) and stays 300 ms. View and container bars take the caller's delay, and the dialog takes an optional one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; V\src\vs\base\browser\ui\progressbar\progressbar.ts (show with delay); C bundle (module vs/workbench/services/progress/browser/progressService, delay:150, the 800 ms timeout, the 300 ms badge delay)

### NT-5.12 Progress sound and announcement
- Tag: both
- Evidence: source
- Boundary: no
- What: When a progress bar has been showing for 3 seconds, a sound and or a screen reader announcement (Progress) repeats until the bar hides. The setting turns each on or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.signals.progress
- Cursor: present
- Source: V\src\vs\base\browser\ui\progressbar\progressbar.ts (PROGRESS_SIGNAL_DEFAULT_DELAY 3000); V\src\vs\base\browser\ui\progressbar\progressAccessibilitySignal.ts; V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (setting accessibility.signals.progress, nls 1754, 4663 to 4665, ProgressBar PROGRESS_SIGNAL_DEFAULT_DELAY=3e3)

### NT-5.13 Slow shell environment progress and failure notice
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS and Linux, when the app was not started from a terminal, a status bar item reads "Resolving shell environment..." while your shell variables load, and an error notification with Restart, Configure and Learn More appears if that fails or runs out of time.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: application.shellEnvironmentResolutionTimeout
- Cursor: present (the error handler vscode:showResolveShellEnvError with three buttons, the progress with a 1600 ms delay and a Learn More button, and the setting with the same 1 to 120 range and a flag that leaves it out on Windows are in the bundle)
- Source: V\src\vs\workbench\electron-browser\window.ts (error prompt and the progress call); V\src\vs\workbench\electron-browser\desktop.contribution.ts (the setting); V\src\vs\platform\shell\node\shellEnv.ts (when it runs, timeout, messages); V\src\vs\code\electron-main\app.ts (sends the error to the window); C bundle (handler vscode:showResolveShellEnvError, call to shellEnv with delay 1600, setting application.shellEnvironmentResolutionTimeout)

## Handed off
- The editor group progress bar (IEditorProgressService, shown only when the group has an editor): ED-1
- The progress bar under the quick input box (QA-6); the outline pane bar (SB-7); notebook cell and find-in-notebook bars (ED-7)
- The bell dot and the "(N in progress)" tooltip text for running progress notifications: NT-3
- The update progress badges on the Manage gear (Checking for updates, Downloading, Updating): AB-4, with the number badge for a ready update (see NT-6 handoffs)
- Host-side look of the three non-notification shapes: SS-5 (status entry spinner), AB-3 (progress badge), DL-1 (pending dialog). I listed NT-5.6, NT-5.9 and NT-5.10 here because the prompt for this group asked for them, so those owners should only cross-reference these ids

## Unclear
- Cursor's progress service accepts two extra string location prefixes (THIS_IS_A_VIEW_AND_NOT_A_VIEW_CONTAINER12xpQ3: routes to view progress, THIS_IS_A_VIEW_AND_NOT_A_VIEW_CONTAINERSLKDJFHS routes to window progress). It is plumbing with no visible UI and I found no caller that shows it.
- Cursor's dialog progress builds its dialog options with fewer arguments than stock (no host service), so the pending dialog may differ in small ways. Not checked on screen.
- Whether Cursor's default horizontal pinned tab row draws the progress badge was not verified. The AB-6 note says that row has no badges.
- Cursor itself uses notification progress for "Capturing snapshot. The window may briefly freeze." and "Uploading snapshot to Cursor...", both not cancellable (see NT-6.16).
- Newer than 1.128 (seen only in N, not listed as features): hasActiveProgress replaces hasProgress so a finished progress no longer blocks close or Clear All, progress titles and messages accept structured NotificationText, and a cancel telemetry hook.
- Everything here was read from code and strings. Nothing was run on screen, so real timings and animation are unseen.

