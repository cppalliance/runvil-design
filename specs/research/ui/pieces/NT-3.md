Group: NT-3 Bell and notification center. Pass: 3. Date: 2026-10-25.

### NT-3.1
1. A bell button sits at the far right of the status bar (entry id `status.notifications`).
2. Clicking the bell opens the notification center (`notifications.showList`).
3. Clicking the bell again while the center is open closes it (`notifications.hideList`).
4. While the center is open, the bell shows a beak pointing at the center.
5. In Cursor, the bell is always right-aligned and rightmost in the status bar, whatever the toast side. [cursor only]

### NT-3.2
1. The bell changes from a plain bell to a bell with a dot when a notification arrives while the center is closed.
2. The bell shows the dot while a notification has running progress and neither toasts nor the center are showing.
3. Opening the center clears the unread dot from the bell.
4. Closing a notification that has not been seen lowers the unread count, and the dot goes away once no unread notification is left.

### NT-3.3
1. With one new notification, hovering the bell shows 1 New Notification.
2. With several new notifications, hovering the bell shows a count such as 3 New Notifications.
3. With notifications in the list but none new, hovering the bell shows No New Notifications.
4. With no notifications at all, hovering the bell shows No Notifications.
5. When notifications have running progress, the tooltip adds a count after the text, such as No New Notifications (1 in progress).
6. While the center is open, hovering the bell shows Hide Notifications.

### NT-3.4
1. With the notification position set to bottom left, the bell moves from the right side of the status bar to the far left of the left side. [stock only]
2. Changing `workbench.notifications.position` moves the bell at once without a restart. [stock only]

### NT-3.6
1. Running Show Notifications (`notifications.showList`) from the Command Palette opens the notification center with the first notification focused.
2. Pressing Ctrl+K Ctrl+Shift+N opens the notification center with the first notification focused.
3. Running the show command while the center is already open focuses the first notification again and does not reopen the center.

### NT-3.7
1. Clicking the Hide button in the center header closes the center.
2. Pressing Escape while the center is open closes it.
3. Running Hide Notifications (`notifications.hideList`) from the Command Palette closes the center, and the command is listed only while the center is open.
4. The center closes by itself when its last notification goes away.
5. When the center had keyboard focus as it closed, focus returns to the editor.

### NT-3.8
1. The center is a panel above the status bar with a header titled Notifications.
2. When there are no notifications, the header title reads No new notifications.
3. Under the header, a scrolling list shows every notification that is still open, and the panel is at most 450 by 400 px.
4. The list updates while the center is open as notifications are added, removed, or changed.
5. In stock, the header toolbar holds Clear All Notifications, Do Not Disturb, a position button, and Hide. [stock only]
6. In stock, the center follows `workbench.notifications.position`: bottom right, bottom left, or below the title bar at the top right, where its Hide icon points up. [stock only]
7. In Cursor, the header toolbar holds Clear All, Do Not Disturb, and Hide, with no position button. [cursor only]
8. In Cursor, the panel always sits at the bottom right even when toasts are on the left, and the list is set to 340 px with a dropdown-style background. [cursor only]

### NT-3.9
1. Clicking Clear All Notifications in the center header closes the center and every notification that has no running progress.
2. Running Clear All Notifications (`notifications.clearAll`) from the Command Palette does the same.
3. A notification with running progress stays after Clear All.
4. The Clear All button is greyed out when the list is empty or every notification has running progress.
5. Clearing all notifications plays a signal, controlled by `accessibility.signals.clear`.

### NT-3.10
1. Opening the center puts keyboard focus on the first notification.
2. Down and Up move focus to the next and previous notification in the list (`list.focusDown`, `list.focusUp`).
3. In a notification row, Tab reaches the buttons and links before the message, because the details row comes first in the page.
4. A screen reader announces the list as Notifications Center and the header toolbar as Notification Center Actions.

### NT-3.11
1. A position icon in the center header, with hover text Configure Notifications Position..., opens a drop-down. [stock only]
2. The drop-down lists Bottom Right, Bottom Left, and Top Right, in that order. [stock only]
3. The entry for the current position is shown checked. [stock only]
4. Picking an entry writes `workbench.notifications.position` and the notifications move to that position. [stock only]

### NT-3.12
1. With a notification focused, Open Accessible View (`editor.action.accessibleView`, Alt+F2, or Alt+Shift+F2 on Linux) shows the notification text and its source in the accessible view.
2. The accessible view text starts with Error:, Warning:, or Info: to match the notification's severity.
3. The accessible view lists the notification's buttons and secondary choices as actions, plus a Clear Notification action, and running one closes the notification.
4. The accessible view lets you step to the next or previous notification.
5. Setting `accessibility.verbosity.notification` adds a hint about opening the accessible view to each notification.

## Unclear

- NT-3.4 has only two pieces because it is one small behavior. It may be better merged into NT-3.5 or NT-1.13 if the owner wants fewer stock-only items.
- NT-3.8 pieces 5 and 7 split the toolbar by stock and Cursor, while the Do Not Disturb button itself is owned by NT-4.2 and the position button by NT-3.11.
- NT-3.12 piece 4 describes next and previous stepping from the source, and the exact keys for stepping were not traced. Opening the accessible view on a toast also opens the notification center as a side effect, and that is not given its own piece.
- NT-3.11 piece 2 uses the menu order values 1, 2, 3 in the stock source.
- Cursor facts here come from the Pass 2 bundle notes and were not seen on screen.
