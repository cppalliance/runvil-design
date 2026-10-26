Group: NT-5 Progress notifications. Pass: 3. Date: 2026-10-25.

### NT-5.1
1. A task started with the notification progress location shows a notification with its message and a thin progress bar under the text.
2. Until the task reports a total, the bar is an endless sweeping bar with no fill.
3. Once the task reports a total, the bar changes to a fill that grows left to right by the percent of work done.
4. When the task reports a new step message, the same notification updates its text in place instead of opening another notification.
5. When the task ends, the notification closes by itself.

### NT-5.2
1. A progress notification that has only a title shows just the title, and one with no title shows just the latest step message.
2. Once a step message arrives, the notification text becomes "title: message".
3. A later step message replaces the earlier message while the title prefix stays.
4. Markdown links such as `[text](command:...)` in the progress text show as clickable links inside the notification.
5. The status bar entry for the same progress shows those links as plain text.
6. The status bar entry tooltip reads "[source] title: message" when the progress names a source, and plain "title: message" when it does not.

### NT-5.3
1. A progress notification started as cancellable shows a Cancel button under the message.
2. When the task supplies its own cancel text, the button shows that text instead of "Cancel".
3. A progress notification can show extra named buttons, one per name, placed before the Cancel button.
4. Clicking Cancel stops the task and reports a plain cancel (`progress.cancel`).
5. Clicking a named button stops the task and reports which button was used (`progress.button.<label>`).
6. A progress notification that is not cancellable and has no named buttons shows no button row.

### NT-5.4
1. A running progress notification never auto-hides, so its toast stays on screen until the task ends.
2. A running progress notification has no close button.
3. Middle-clicking a running progress notification, or pressing `Delete` (`Cmd+Backspace` on macOS) with it focused (`notification.clear`), does not remove it.
4. Running "Clear All Notifications" (`notifications.clearAll`) from the Command Palette removes the other notifications and leaves running progress notifications in place.
5. A task that ends in under 800 ms still keeps its notification on screen until 800 ms have passed, so it does not flash.

### NT-5.5
1. An indeterminate bar shows a short segment sweeping across a 2 px high line, over and over.
2. After an indeterminate bar has run for 10 seconds, the sweep changes to a stepped animation.
3. A determinate bar fills left to right by the share of work done.
4. When a determinate bar reaches 100 percent it grows to full width and then hides, and an indeterminate bar fades out and hides when its work ends.
5. The bar uses the theme color `progressBar.background` and looks the same in notifications, views, containers and editor groups.

### NT-5.6
1. Work reported at the window location with a command shows a spinner entry named "Progress Message" (entry id `status.progress`) in the status bar.
2. The entry sits on the left side of the status bar, after all the other left-side entries.
3. The entry text is "title: message", the title alone, or the message alone, and with neither the entry is not shown.
4. The entry icon is the loading spinner by default, or the syncing icon when the task asks for it.
5. Clicking the entry runs the command the task supplied, for example `notifications.showList`.
6. When several window progress tasks run at once, the entry shows the newest one and falls back to the next one when it ends.

### NT-5.7
1. A progress notification marked silent does not pop up as a toast, and its work shows only as a spinner entry in the status bar.
2. When notifications are filtered to errors only (Do Not Disturb), a progress notification is silent in the same way.
3. Dismissing the toast of a running progress notification, for example with `Escape` (`notifications.hideToasts`), leaves the work visible as the status bar entry.
4. Clicking that status bar entry opens the notification center (`notifications.showList`), where the notification shows its progress bar.
5. The status bar entry disappears when the notification shows again or when the task ends.
6. Window progress without a command is shown as a silent notification, so it first appears only as a status bar entry and can be brought forward by clicking it.

### NT-5.8
1. Work reported for the Explorer, Source Control or Extensions location shows the thin progress bar at the top of that container's content.
2. Work reported against a view id shows the bar at the top of that view only.
3. Work reported against a container id shows the bar at the top of the container content.
4. The bar runs only while the view or container is visible, and switching away hides it and switching back shows it again if the work is still going.
5. The bar goes away when the work ends.

### NT-5.9
1. Work on a side bar container shows a small spinning progress badge on that container's icon in the activity bar.
2. Work on a view inside a side bar container puts the same badge on that view's container icon.
3. While the progress badge runs, it shows instead of any icon badge or number badge on that container.
4. The progress badge disappears when the work ends.

### NT-5.10
1. Work reported at the dialog location opens a modal dialog with a spinning loading icon and the message.
2. An optional detail line shows under the message.
3. Unless the dialog is sticky, it has a "Dismiss" button, or "Cancel" (or the task's own cancel text) when the task is cancellable, placed after any named buttons.
4. Clicking that button closes the dialog and, if the task is still running, tells the task which button was used.
5. As the task reports messages, the dialog message updates in place.
6. A sticky dialog has no button and cannot be closed with the close action or the default action until the task ends.
7. The dialog closes by itself when the task ends.

### NT-5.11
1. A notification progress started with a delay option stays hidden for that long and shows only if the task is still running, with no 800 ms minimum afterward.
2. Window progress waits 150 ms before it shows and then stays at least 150 ms, so a task shorter than 150 ms never shows an entry.
3. The activity bar badge waits 300 ms, or the task's own delay, before it shows and then stays at least 300 ms.
4. View and container bars wait for the task's own delay before they show, with no fixed default.
5. A dialog progress waits for an optional delay before it first appears, and later message updates do not wait again.

### NT-5.12
1. After a progress bar has been showing for 3 seconds, a sound plays and repeats on a loop until the bar hides.
2. With the announcement turned on, a screen reader announces "Progress" on a loop in the same way until the bar hides.
3. A bar that hides before 3 seconds makes no sound and no announcement.
4. Setting `accessibility.signals.progress` has a `sound` value that is `auto` (only with a screen reader attached, the default), `on` or `off`.
5. Setting `accessibility.signals.progress` has an `announcement` value that is `auto` (only in screen reader optimized mode) or `off`, with `off` as the default.

### NT-5.13
1. When the shell environment is still loading after 1.6 seconds, a window progress item reading "Resolving shell environment..." shows in the status bar with a Learn More button that opens the documentation.
2. If resolving times out or fails, an error notification shows the reason, for example "Unable to resolve your shell environment in a reasonable time. Please review your shell configuration and restart."
3. The notification's Restart button relaunches the application.
4. The notification's Configure button opens the user Settings filtered to `application.shellEnvironmentResolutionTimeout`.
5. The notification's Learn More button opens the documentation page.
6. `application.shellEnvironmentResolutionTimeout` sets the number of seconds to wait, from 1 to 120, default 10, and applies to the whole application.
7. On Windows and when the app was started from a terminal nothing is resolved, so neither the progress item nor the error appears, and the setting is left out of the Settings editor on Windows.

## Unclear
- Delay and minimum display times for window progress (NT-5.6) and the activity bar badge (NT-5.9) are written once, as pieces of NT-5.11, so the two feature blocks that also mention them have no timing piece of their own.
- The feature block for NT-5.6 says the entry is on the left end of the status bar, while the source places it on the left side with the lowest priority, which is after all the other left-side entries. The piece follows the source.
- The delayed notification path (NT-5.11 piece 1) and the 800 ms minimum (NT-5.4 piece 5) come from the same source branch, so a tracker should treat them as two sides of one rule.
- Nothing was run on screen. All pieces come from source and bundle strings, so real timings and animation were not seen.
