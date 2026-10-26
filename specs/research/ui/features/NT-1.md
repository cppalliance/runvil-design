Group: NT-1 Toast popups. Section: Notifications. Date: 2026-10-25. Pass: 2.
Features: 15.

### NT-1.1 Toast stack
- Tag: both
- Evidence: source
- Boundary: no
- What: New notifications appear as cards in a window corner, newest on top, at most three at once and at most 450 px wide, and a card that does not fit the window height stays hidden but remains in the center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - same limit of three cards and 450 px basis, but the card list is forced to 340 px, margin 6 px, no border, 6 px radius, glass shadow, z-index 1000
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragments MAX_NOTIFICATIONS=3, MAX_WIDTH=450, css .notification-toast and .notifications-list-container overrides)

### NT-1.2 Slide and fade in
- Tag: both
- Evidence: source
- Boundary: no
- What: A new toast slides up from below (from the right edge when the corner is top right) and fades in over 300 ms, and with reduced motion on it appears with no transition.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.reduceMotion
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (css rules .notification-fade-in and .notification-fade-in-done, reduced motion transition 0ms rule)

### NT-1.3 Burst guard
- Tag: both
- Evidence: source
- Boundary: no
- What: If more than three notifications arrive within 800 ms, the extra ones do not get a toast and can only be found in the notification center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragment SPAM_PROTECTION={interval:800,limit:MAX_NOTIFICATIONS})

### NT-1.4 Toasts step aside for the center
- Tag: both
- Evidence: source
- Boundary: no
- What: While the notification center is open no new toasts appear, and opening the center removes the toasts that are showing.
- Commands: notifications.showList; notifications.toggleList
- Menus: none found
- Keys: Ctrl+K Ctrl+Shift+N (opens the center, which hides the toasts first)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (fragments isNotificationsCenterVisible check at the start of addToast, showList handler t.hide(),e.show())

### NT-1.5 Auto-hide timers by severity
- Tag: both
- Evidence: source
- Boundary: no
- What: Each toast hides itself after a time that depends on its severity - Info 10 s, Warning 12 s, Error 15 s in stock - and there is no setting to change the times.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - Info 15 s, Warning 18 s, Error 20 s, and a new Success at 15 s; the AI severity has no entry in the table
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragment PURGE_TIMEOUT={[Info]:15e3,[Warning]:18e3,[Error]:2e4,[Success]:15e3})

### NT-1.6 Hover and focus pause the timer
- Tag: both
- Evidence: source
- Boundary: no
- What: A toast does not hide while the mouse is over it, while it has keyboard focus, or while the window is not focused, and the countdown starts again afterwards.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (fragment purgeNotification with MOUSE_OVER and MOUSE_OUT tracking, hostService.hasFocus check, list hasFocus check)

### NT-1.7 Sticky toasts stay until closed
- Tag: both
- Evidence: source
- Boundary: no
- What: Some toasts never hide on their own - errors that have buttons, notifications you expanded (when they have no buttons), notifications with running progress, and any marked sticky by the caller.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\common\notifications.ts; V\src\vs\platform\notification\common\notification.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (fragment get sticky() with the same three rules)

### NT-1.8 Hide toasts with Escape
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing Escape while toasts are showing dismisses all of them (they stay in the center), and if a toast had focus the focus goes to the next toast or back to the editor.
- Commands: notifications.hideToasts
- Menus: none found
- Keys: Escape (when notificationToastsVisible, low weight so editor widgets win first); Escape (when notificationToastsVisible && notificationFocus, high weight)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (command notifications.hideToasts, keybinding rules with weights 150 and 300)

### NT-1.9 Focus a toast from the Command Palette
- Tag: both
- Evidence: source
- Boundary: no
- What: The Focus Notification Toast command moves keyboard focus to the newest toast so you can reach its buttons and links without the mouse.
- Commands: notifications.focusToasts
- Menus: CommandPalette (when notificationToastsVisible)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (command notifications.focusToasts, label Focus Notification Toast in nls module vs/workbench/browser/parts/notifications/notificationsCommands)

### NT-1.10 Move between toasts with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: With a toast focused, Down and Up step to the next and previous toast and PageUp, Home, PageDown and End jump to the first and last one.
- Commands: notifications.focusNextToast; notifications.focusPreviousToast; notifications.focusFirstToast; notifications.focusLastToast
- Menus: none found
- Keys: Down; Up; PageUp or Home; PageDown or End (all when notificationFocus && notificationToastsVisible)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (the four command ids and the same four key rules)

### NT-1.11 Silent notifications skip the toast
- Tag: both
- Evidence: source
- Boundary: no
- What: A notification with silent priority never shows a toast and only appears in the notification center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\platform\notification\common\notification.ts; C bundle (fragment addToast returns early when priority is SILENT)

### NT-1.12 Optional toasts keep off a focused input
- Tag: both
- Evidence: source
- Boundary: no
- What: A toast with optional priority is skipped when you are typing in an input inside the corner where toasts would land, so it cannot cover what you are typing into.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.notifications.position (stock only, decides which window quarter is checked)
- Cursor: changed - the checked quarter follows the side bar side instead of a position setting
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (fragment isElementInNotificationQuarter using getSideBarPosition)

### NT-1.13 Pick the toast corner with a setting
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A setting puts toasts in the bottom right (default), bottom left, or top right, where they sit below the title bar and slide in from the right, and the same setting also moves the center and the bell (the header dropdown that writes it is NT-3.11); it is tagged experimental with an automatic experiment mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.notifications.position
- Cursor: missing
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\common\notifications.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; C bundle (searched workbench.notifications.position, setNotificationsPosition, configureNotificationsPosition, css bottom-left and top-right classes: no hits)

### NT-1.14 Toasts follow the side bar side
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor puts the toast stack in the bottom left when the side bar is on the left (the default) and in the bottom right when the side bar is on the right, and it moves when the side bar location changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragments configureToastContainerPosition, getSideBarPosition()===1, onDidChangeConfiguration check for workbench.sideBar.location; css .notifications-toasts{bottom:25px;right:3px})

### NT-1.15 Hide toasts for a set time
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A Cursor-only command hides every toast now and blocks new toasts for a number of milliseconds; it has no label and is not in the Command Palette, and only the onboarding code calls it (10 minutes after import from VS Code, 30 seconds on first landing).
- Commands: cursorai.action.hideNotificationsForMs
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragments hideForMs, shouldHideUntil, command id cursorai.action.hideNotificationsForMs, two executeCommand calls with 10*6e4 and 3e4)

## Handed off

- Toasts that disappear when Do Not Disturb turns on (all of them, or only those from a muted source), and errors or urgent-priority notifications that ignore Do Not Disturb: NT-4.
- The progress bar and Cancel button inside a toast, and the rule that a running-progress toast stays: NT-5 (the stay-open rule is listed here only as part of NT-1.7).
- The floating Update to v, New update available and New in cards: NT-6 (they do not use the toast service at all).

## Unclear

- Cursor passes toastDurationMs and closeNotificationOnToastDismiss on some notifications (feedback, bug report, and sync success toasts), but the toast timer in the main bundle reads only the severity table, so those fields look unused here. A reader may exist in the Agents window bundle, which was not read.
- The AI severity (value 4) has no entry in the Cursor timer table, so its toast would get an undefined delay. Not seen on screen, and no caller was found.
- Cursor-specific toast contents (the memory warning with a Capture Snapshot button, the sign-in policy error) are not covered by any NT group. They are content, not UI features, so none is listed.
- All Cursor facts come from strings and minified code in the bundle. Nothing was run on screen, and pixel sizes come from code and css constants.
