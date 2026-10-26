Group: NT-3 Bell and notification center. Section: Notifications. Date: 2026-10-25. Pass: 2.
Features: 11.

### NT-3.1 Bell in the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: A bell button sits at the far right of the status bar (entry id status.notifications), a click opens the notification center, a second click closes it, and while the center is open the bell shows a beak pointing at it.
- Commands: notifications.showList; notifications.hideList
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the bell is always right-aligned and rightmost, where stock moves it to the left side for a bottom-left position and removes it for top right
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsStatus, fragments addEntry(t,"status.notifications",1,Number.NEGATIVE_INFINITY) and showBeak:this.isNotificationsCenterVisible)

### NT-3.2 Unread dot on the bell
- Tag: both
- Evidence: source
- Boundary: no
- What: The bell changes from a plain bell to a bell with a dot when a notification arrived while the center was closed, or when progress is running and neither toasts nor the center are showing, and opening the center clears it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (fragments newNotificationsCount, the bell-dot text in updateNotificationsCenterStatusItem)

### NT-3.3 Bell tooltips
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering the bell shows a count such as 1 New Notification, 3 New Notifications, No New Notifications (1 in progress), No Notifications, or Hide Notifications while the center is open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (labels at nls indexes 3904 to 3911 in module vs/workbench/browser/parts/notifications/notificationsStatus, fragment getTooltip)

### NT-3.4 Bell moves left for the bottom-left position
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When notifications are set to the bottom left, the bell moves from the right side of the status bar to the far left of the left side.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.notifications.position
- Cursor: missing
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (the status entry is added with fixed right alignment and no getDesiredAlignment branch; the position setting is not in the bundle)

### NT-3.6 Open the notification center
- Tag: both
- Evidence: source
- Boundary: no
- What: The Show Notifications command, the bell, or the Ctrl+K Ctrl+Shift+N chord opens the center with the first notification focused, and running it again while the center is open just refocuses the first one.
- Commands: notifications.showList
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+Shift+N
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; C bundle (command id, label Show Notifications in module vs/workbench/browser/parts/notifications/notificationsCommands, key rule fragment with the chord on KeyN)

### NT-3.7 Close the notification center
- Tag: both
- Evidence: source
- Boundary: no
- What: The center closes with the Hide button in its header, Escape, the Hide Notifications command, or a second click on the bell, it closes itself when the last notification goes, and focus returns to the editor if the center had it.
- Commands: notifications.hideList
- Menus: CommandPalette (when notificationCenterVisible)
- Keys: Escape (when notificationCenterVisible)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (command id, label Hide Notifications, key rule with weight 250 and Escape)

### NT-3.8 The center list
- Tag: both
- Evidence: source
- Boundary: no
- What: The center is a panel above the status bar with a header titled Notifications (or No new notifications when empty) and a scrolling list, at most 450 by 400 px, of every notification still open, which updates as notifications come and go.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - the panel always sits bottom right even when toasts are on the left, the list is forced to 340 px with a dropdown-style background, the header toolbar has Clear All, Do Not Disturb and Hide with no position button
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css; C bundle (module vs/workbench/browser/parts/notifications/notificationsCenter, labels at nls 3877 to 3883, fragments MAX_DIMENSIONS=new(450,400), header toolbar build, css .notifications-center)

### NT-3.9 Clear all notifications
- Tag: both
- Evidence: source
- Boundary: no
- What: The Clear All Notifications header button or command closes the center and every notification that has no running progress, and the button is greyed out when there is nothing to clear.
- Commands: notifications.clearAll
- Menus: CommandPalette
- Keys: none found
- Settings: accessibility.signals.clear
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (command id, label Clear All Notifications, fragments clearAll and updateTitle with the enabled check)

### NT-3.10 Keyboard focus and navigation in the center
- Tag: both
- Evidence: source
- Boundary: no
- What: Opening the center focuses the first notification, Up and Down move between them, and the buttons and links of a row come first in tab order because the details row sits before the message in the page.
- Commands: list.focusDown; list.focusUp
- Menus: none found
- Keys: Down (when a list has focus); Up (when a list has focus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\actions\listCommands.ts; C bundle (ids list.focusDown and list.focusUp found, same row render order)

### NT-3.11 Position dropdown in the center header
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A position icon in the center header opens a drop-down with Bottom Right, Bottom Left, and Top Right (the current one checked), and picking one writes the notification position setting.
- Commands: workbench.action.configureNotificationsPosition; workbench.action.setNotificationsPosition.bottomRight; workbench.action.setNotificationsPosition.bottomLeft; workbench.action.setNotificationsPosition.topRight
- Menus: NotificationsCenterPositionMenu
- Keys: none found
- Settings: workbench.notifications.position
- Cursor: missing
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\platform\actions\common\actions.ts; C bundle (searched setNotificationsPosition, configureNotificationsPosition, NotificationsCenterPositionMenu, Configure Notifications Position label: no hits)

### NT-3.12 Read a notification in the accessible view
- Tag: both
- Evidence: source
- Boundary: no
- What: With a notification focused, Open Accessible View shows its text and source in the accessible view with its buttons as actions and lets you step to the next or previous notification, and a setting adds a hint about this to each notification.
- Commands: editor.action.accessibleView
- Menus: CommandPalette
- Keys: Alt+F2 (Alt+Shift+F2 on Linux)
- Settings: accessibility.verbosity.notification
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationAccessibleView.ts; V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; V\src\vs\workbench\browser\workbench.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationAccessibleView, label {0} Source: {1} at nls 3853, ids editor.action.accessibleView and accessibility.verbosity.notification)

## Handed off

- The Do Not Disturb dropdown in the center header, the bell-slash icon on the bell, and the center closing when Do Not Disturb turns on: NT-4.
- The Status Message entry that the notification service writes into the status bar (status.message, labeled Status Message): SS-5, since it is a status bar item and the catalog does not give it to any NT group.
- The title bar right-click row Notifications, which only shows or hides the title bar bell: TB-2 owns the title bar menu. The bell it controls is NT-3.5.
- The Progress Message status entry that opens the center on click: NT-5 (window progress) with the status bar rendering in SS-5.

## Unclear

- SS-5 also names the notifications bell, and TB-5 names stock's title bar bell. I kept both here because the approved NT-3 description lists the bell and says Cursor lacks the title bar bell. If the parent prefers one owner, the status bar item is the part to move.
- Screen reader announcements when a notification arrives (Error:, Warning:, Info: prefixes, in notificationsAlerts.ts) have no visible UI, so no feature is listed. The same labels exist in Cursor (nls 3874 to 3876).
- Newer than 1.128: VS Code main updates the accessible view live and adds link actions to it. Neither is in 1.128.0, so NT-3.12 describes only the older behavior.
- The stock position setting is tagged experimental with an automatic experiment mode, so whether stock users saw the position features (NT-3.4, NT-3.5, NT-3.11) by default was not checked.
- Cursor facts come from strings and minified code in the bundle and were not seen on screen.
