Group: SS-5 Alerts and background activity. Pass: 3. Date: 2026-10-25.

### SS-5.1
1. A bell icon sits at the far right end of the status bar.
2. The bell becomes a bell with a dot when a notification arrived that you have not seen in the center.
3. The bell also gets the dot when a notification with progress is running while the toasts and the center are hidden.
4. Opening the notification center clears the unread state and the dot.
5. The tooltip reads "No Notifications", "No New Notifications", "1 New Notification" or "{n} New Notifications" to match what is in the center.
6. While notifications with progress are running the tooltip adds an "({n} in progress)" variant of those texts.
7. While the notification center is open the tooltip reads "Hide Notifications".

### SS-5.2
1. Clicking the bell opens the notification center above it, and clicking the bell again hides it.
2. While the center is open the bell item shows a beak pointing at the center.
3. Show Notifications (`notifications.showList`) opens the center from the Command Palette or from the chord `Ctrl+K Ctrl+Shift+N`.
4. Pressing `Escape` while the center is open hides it (`notifications.hideList`), and Hide Notifications is also in the Command Palette.

### SS-5.4
1. A short text message can appear as the last item on the left side of the bar, with no icon.
2. A new message replaces any earlier message.
3. A message can wait for a delay before it shows and can disappear after a set time, for example "The extension host terminated unexpectedly. Restarting..." for 5 seconds.
4. The message shown when an extension changes the workspace folders stays for 10 seconds.
5. The message item has no click action.

### SS-5.5
1. The Problems item on the left shows an error icon with the error count and a warning icon with the warning count.
2. An info icon with its count appears only when there are infos.
3. Counts above 999 shorten to "1K" through "9K", and counts above 9999 show as "10K+".
4. The hover reads "Errors: 2, Warnings: 1" with the current counts, or "No Problems" when there are none.

### SS-5.6
1. Clicking the Problems item runs Toggle Problems (`workbench.actions.view.toggleProblems`) and opens and focuses the Problems panel.
2. Clicking the Problems item while the Problems panel is already visible closes the panel.

### SS-5.7
1. When problems are turned off, a warning-colored item with a whole-word icon appears right of the Problems counts.
2. The tooltip reads "Problems are turned off. Click to open settings."
3. Clicking the item opens Settings filtered to `problems.visibility` (`workbench.action.openSettings`).
4. The item disappears when the setting is turned back on.
5. Setting `problems.visibility` chooses whether problems are visible throughout the workbench, with values on (default) and off.

### SS-5.8
1. While the cursor is on a problem and `problems.showCurrentInStatus` is on, the left side shows the first line of that problem with an icon.
2. The icon matches the severity of the problem: error, warning or info.
3. Moving the cursor off the problem removes the item.
4. The item has no click action.
5. Setting `problems.showCurrentInStatus` chooses whether this item can appear, with values on and off (default).

### SS-5.9
1. A left item with a debug icon and the name of the selected launch configuration appears, with the launch file name added when there is more than one launch file.
2. The tooltip reads "Select and Start Debug Configuration".
3. By default the item shows only after the first debug session has started.
4. Setting `debug.showInStatusBar` chooses when the item shows, with values `never`, `always` and `onFirstSessionStart` (default).
5. With `debug.showInStatusBar` set to `always` the item shows before any debug session has started, and with `never` it does not show at all.

### SS-5.10
1. Clicking the Debug item opens a picker of the launch configurations and the dynamic debug providers.
2. Choosing a row in the picker selects that configuration and starts it.
3. Running Select and Start Debugging (`workbench.action.debug.selectandstart`) from the Command Palette opens the same picker.

### SS-5.11
1. While tasks are running, a tools icon with the number of running tasks appears on the left, right of the Problems item.
2. The tooltip reads "Show Running Tasks".
3. Clicking the item opens the Show Running Tasks picker (`workbench.action.tasks.showTasks`).
4. The item disappears when no task is running.
5. The Terminal menu row "Show Running Tasks..." in the Manage group opens the same picker.

### SS-5.12
1. Work that reports window progress puts a spinning item at the far end of the left side, with the text "title: message".
2. The tooltip repeats the text and adds the source in brackets.
3. The item appears only after 150 ms of progress and stays on screen for at least 150 ms, so very short work does not flash.
4. Clicking the item runs the command that the work supplied.
5. While a build task runs the item reads "Building..." and clicking it opens the Show Running Tasks picker (`workbench.action.tasks.showTasks`).

### SS-5.13
1. On the desktop a prominent zoom-out icon appears on the right side when this window is zoomed out from the configured level, and a zoom-in icon when it is zoomed in.
2. When the window zoom matches the configured level the item is hidden.
3. Hovering the item opens a mini toolbar with Zoom Out, the zoom level number, Zoom In, Reset and a Settings gear.
4. The zoom level number has the tooltip "Zoom Level: 2 (144%)" with the current level and percentage.
5. The Zoom Out, Zoom In and Reset buttons run `workbench.action.zoomOut`, `workbench.action.zoomIn` and `workbench.action.zoomReset` and the item updates at once.
6. Pressing `Ctrl+=` (also `Ctrl+Shift+=` and `Ctrl+Numpad+`) zooms in, `Ctrl+-` (also `Ctrl+Shift+-` and `Ctrl+Numpad-`) zooms out, and `Ctrl+Numpad0` resets the zoom.
7. The Settings gear opens Settings on `window.zoom` (`workbench.action.openSettings`).
8. Setting `window.zoomPerWindow` chooses whether Zoom In and Zoom Out change all windows or only the active one, with values on (default) and off, and each window has its own item.
9. Setting `window.zoomLevel` chooses the default zoom level for all windows, a number with default 0 where each step is 20 percent larger or smaller.

### SS-5.14
1. When an extension asks to show an output channel while the Output panel is hidden and the panel is marked in `workbench.view.showQuietly`, a prominent output icon appears on the right next to the bell instead.
2. The tooltip reads "Show {channel} Output Channel".
3. Clicking the item opens that channel (`workbench.action.output.show.{channelId}`).
4. Setting `workbench.view.showQuietly` is an object with one key, `workbench.panel.output`, which turns the quiet indicator on or off for the Output panel.
5. The setting starts empty, so no item is shown by default.

### SS-5.15
1. When the terminal pty host stops answering, an amber warning item with a disconnect icon and the text "Pty Host" appears on the left.
2. The tooltip says that terminals may stop working.
3. Clicking the item restarts the pty host, and Restart Pty Host (`workbench.action.terminal.restartPtyHost`) does the same from the Command Palette.
4. The item goes away when the pty host answers again.

### SS-5.16
1. Running Developer: Record Terminal Session (`workbench.action.terminal.recordSession`) from the Command Palette opens a terminal.
2. While it records, a spinning item reading "Recording terminal session..." shows on the left.
3. When the terminal has been idle for 5 seconds the recording ends and the item goes away.
4. At the end the recording is copied to the clipboard as JSON.
5. The recording item has no click action.

## Unclear
- SS-5.17 and SS-6.12 describe the same high memory warning item, so its pieces appear in both files. One of the two features should be dropped.
- The text of the status message for extension workspace folder changes (SS-5.4 piece 4) is not given in the feature block, so the piece names only the 10 second duration.
- In SS-5.17 I tagged every piece `[cursor only]` because the whole feature is Cursor only.
