Group: NT-4 Do Not Disturb. Section: Notifications. Date: 2026-10-25. Pass: 2.
Features: 12.

### NT-4.1 Turn Do Not Disturb on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Toggle Do Not Disturb Mode from the Command Palette to flip the global mode, which silences every info and warning notification (errors still show).
- Commands: notifications.toggleDoNotDisturbMode
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsCommands, id notifications.toggleDoNotDisturbMode, nls 3890)

### NT-4.2 Do Not Disturb dropdown in the center header
- Tag: both
- Evidence: source
- Boundary: no
- What: The bell-slash button in the notification center header opens a menu with Enable Do Not Disturb Mode or Disable Do Not Disturb Mode, then a checked list of up to 10 sources sorted by label (checked means that source may still show everything), then More… when there are over 10. The dropdown is built in code, so it is not a MenuId. While the mode is on the center closes, and clicking the bell reopens it.
- Commands: workbench.action.configureDoNotDisturbMode; notifications.toggleDoNotDisturbMode; notifications.toggleDoNotDisturbModeBySource
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsCenter and notificationsActions, id workbench.action.configureDoNotDisturbMode, nls 3868, 3880, 3881, 3882, MAX_NOTIFICATION_SOURCES=10)

### NT-4.3 Choose sources in a picker
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Toggle Do Not Disturb Mode By Source... to get a multi-select quick pick with the placeholder "Select sources to enable all notifications from". Ticked sources keep showing everything, unticked sources become silent for info and warning, and accepting applies the choice. Only sources that have already shown a notification in this install are listed.
- Commands: notifications.toggleDoNotDisturbModeBySource
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsCommands, id notifications.toggleDoNotDisturbModeBySource, nls 3884, 3891)

### NT-4.4 Mute one source from a notification's gear menu
- Tag: both
- Evidence: source
- Boundary: no
- What: The More Actions... gear menu on a notification that has a source offers Turn Off Info and Warning Notifications from '{source}', or Turn On All Notifications from '{source}' when that source is already muted. The choice flips that one source's filter, the same one the picker and the header dropdown change.
- Commands: workbench.action.configureNotification
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsViewer, id workbench.action.configureNotification, nls 3872, 3917, 3918)

### NT-4.5 Bell-slash icon in the status bar
- Tag: both
- Evidence: source
- Boundary: no
- What: While Do Not Disturb is on, the status bar bell turns into a bell-slash (a bell-slash-dot when something new arrived or progress is running), with the accessible name Do Not Disturb and the tooltip Do Not Disturb Mode is Enabled. Clicking it still shows or hides the center.
- Commands: notifications.showList; notifications.hideList; notifications.toggleDoNotDisturbMode; notifications.toggleDoNotDisturbModeBySource
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsStatus, status entry status.notifications, bell-slash and bell-slash-dot, nls 3902, 3903); V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (NotificationsFilter.ERROR branch); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (icons bell-slash and bell-slash-dot, NLS keys status.doNotDisturb and status.doNotDisturbTooltip, commands notifications.toggleDoNotDisturbMode and notifications.toggleDoNotDisturbModeBySource)

### NT-4.6 Turning Do Not Disturb on clears what is on screen
- Tag: both
- Evidence: source
- Boundary: no
- What: Switching the global mode on hides every visible toast and closes the notification center. Switching one source off removes that source's visible info and warning toasts, while its errors and urgent items stay.
- Commands: notifications.toggleDoNotDisturbMode
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsToasts and notificationsCenter, the onDidChangeFilter handlers)
### NT-4.7 What still pops up: errors and urgent notifications
- Tag: both
- Evidence: source
- Boundary: no
- What: With Do Not Disturb on, any notification with Error severity and any with Urgent priority still shows as a toast. Only Default and Optional priority items below error severity become silent. Stock callers that set Urgent include the window, integrity, extension host and save participant prompts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (Cursor adds Success and AI severities; the filter only exempts Error, so Success is silenced like Info and Warning; the memory warning prompt is Urgent so it still shows)
- Source: V\src\vs\workbench\common\notifications.ts; V\src\vs\platform\notification\common\notification.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (module vs/workbench/common/notifications, the create method with the global and per-source check, module memoryMonitor.contribution)

### NT-4.8 Silenced notifications still land in the center
- Tag: both
- Evidence: source
- Boundary: no
- What: A notification silenced by Do Not Disturb gets no toast and no screen reader announcement, but it is still added to the notification center list and still raises the unread dot on the bell.
- Commands: notifications.showList; notifications.clearAll
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+Shift+N (notifications.showList)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsAlerts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsAlerts, notificationsToasts and notificationsStatus, the Silent priority checks)

### NT-4.9 Progress stays quiet under Do Not Disturb
- Tag: both
- Evidence: source
- Boundary: no
- What: When Do Not Disturb is on globally or for the progress source, a notification-location progress turns silent. It makes no toast and shows only as the left status bar spinner entry until it ends, unless its priority is Urgent.
- Commands: notifications.showList
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; C bundle (module vs/workbench/services/progress/browser/progressService, the case 15 branch that sets the Silent priority)

### NT-4.10 Zen Mode turns Do Not Disturb on
- Tag: both
- Evidence: source
- Boundary: no
- What: Entering Zen Mode switches global Do Not Disturb on when zenMode.silentNotifications is true, and leaving Zen Mode switches it off again, but only if the mode was off when Zen Mode began. Changing the setting while in Zen Mode applies at once. Zen Mode itself is owned by the layout groups.
- Commands: workbench.action.toggleZenMode; workbench.action.exitZenMode
- Menus: MenubarAppearanceMenu
- Keys: Ctrl+K Z (workbench.action.toggleZenMode); Escape Escape (workbench.action.exitZenMode, when InEditorZenModeContext)
- Settings: zenMode.silentNotifications
- Cursor: present
- Source: V\src\vs\workbench\browser\layout.ts; V\src\vs\workbench\browser\workbench.zenMode.contribution.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; C bundle (module vs/workbench/browser/workbench.contribution, setting zenMode.silentNotifications default true, nls 4319, layout code with handleNotificationsDoNotDisturbMode)

### NT-4.11 Do Not Disturb choices persist
- Tag: both
- Evidence: source
- Boundary: no
- What: The global switch and every per-source choice are saved in application storage (keys notifications.doNotDisturbMode and notifications.perSourceDoNotDisturbMode, not settings) and survive restarts. A source only appears in the lists after its first notification.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (same storage keys; Cursor's service adds a resetGlobalFilter method and a doNotPersist option, with no caller in the main bundle)
- Source: V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (module vs/workbench/services/notification/common/notificationService, keys GLOBAL_FILTER_SETTINGS_KEY and PER_SOURCE_FILTER_SETTINGS_KEY)

### NT-4.12 Cursor floating cards ignore Do Not Disturb
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's update and changelog cards are drawn by a separate component that never goes through the notification service and never reads the Do Not Disturb filter, so turning the mode on does not hide them. See NT-6 for the cards.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (the Zyk update card and Qyk changelog card components, which use updateNotificationService, commandService and openerService but no notification service)

## Handed off
- cursorai.action.hideNotificationsForMs (hide toasts now and block new ones for a number of milliseconds, a separate suppression from Do Not Disturb, used by onboarding): NT-1
- The unread and in-progress dot and tooltip counts on the bell (for example "No New Notifications (1 in progress)"): NT-3
- The Toggle Zen Mode command, its other settings (hide status bar, center layout, hide tabs) and Zen exit: TB-4 (layout controls), since no group names Zen Mode itself

## Unclear
- NT-4.4 overlaps with the NT-2 catalog text, which also names per-source mute. I kept it here because it flips the same Do Not Disturb source filter. The NT-2 owner should point to NT-4.4 and not list it again.
- Cursor's resetGlobalFilter is called only from the Agents window bundle (the file list showed a match in workbench.glass.main.js, which I did not read). In the main bundle it is unused.
- Which sources appear in the header dropdown and the picker depends on which extensions and features have raised a notification with a source. I did not run Cursor, so I could not list real sources.
- Everything here was read from code and strings. Nothing was run on screen, so toast and center timing under Do Not Disturb is unseen.
