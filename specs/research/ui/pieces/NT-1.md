Group: NT-1 Toast popups. Pass: 3. Date: 2026-10-25.

### NT-1.1
1. A new notification appears as a card in a corner of the window, and the newest card sits on top of the older ones.
2. At most three toast cards show at once, and each card is at most 450 px wide.
3. A card that does not fit in the window height stays hidden but remains in the notification center.
4. In Cursor the card list is set to 340 px with a 6 px margin, no border, a 6 px corner radius, a glass shadow, and a z-index of 1000. [cursor only]

### NT-1.2
1. A new toast slides up from below and fades in over 300 ms.
2. When toasts are set to the top right, a new toast slides in from the right edge instead of from below. [stock only]
3. With `workbench.reduceMotion` turned on, a new toast appears with no slide or fade.

### NT-1.3
1. When more than three notifications arrive within 800 ms, the extra ones get no toast.
2. A notification that got no toast because of the burst can still be found in the notification center.
3. Notifications that arrive after the 800 ms burst has passed get toasts again.

### NT-1.4
1. While the notification center is open, a new notification does not get a toast.
2. Opening the notification center with `notifications.showList` or Ctrl+K Ctrl+Shift+N first removes the toasts that are showing.
3. Toasts removed because the center opened are still listed in the center.

### NT-1.5
1. In stock, a toast hides itself after a time set by its severity: Info 10 s, Warning 12 s, Error 15 s, and no setting changes these times. [stock only]
2. In Cursor, a toast hides itself after Info 15 s, Warning 18 s, or Error 20 s. [cursor only]
3. In Cursor, a Success toast hides itself after 15 s. [cursor only]
4. A toast that times out leaves the screen but the notification stays in the notification center.

### NT-1.6
1. A toast does not hide while the mouse pointer is over it, and its countdown starts again when the pointer leaves.
2. A toast does not hide while it has keyboard focus, and its countdown starts again when focus leaves it.
3. A toast does not hide while the window is not focused, and its countdown starts again when the window regains focus.

### NT-1.7
1. An Error toast that has buttons stays on screen until it is closed.
2. A notification that you expanded stays on screen until it is closed, as long as it has no buttons.
3. A notification with running progress stays on screen while the progress runs.
4. A notification that its caller marked sticky stays on screen until it is closed.

### NT-1.8
1. Pressing Escape while toasts are showing hides all of them, and the notifications stay in the notification center.
2. When a toast has keyboard focus, pressing Escape hides all toasts and puts focus back in the editor.
3. When the focused toast is closed or times out and other toasts remain, focus moves to the next toast, or to the previous one when there is no next.
4. When the editor or another widget handles Escape first, such as an open suggest widget, the toasts stay.

### NT-1.9
1. Running Focus Notification Toast (`notifications.focusToasts`) from the Command Palette moves keyboard focus to the newest toast.
2. Focus Notification Toast is listed in the Command Palette only while a toast is visible.
3. With a toast focused, Tab moves through its buttons and links so they can be used without the mouse.

### NT-1.10
1. With a toast focused, Down moves focus to the next toast (`notifications.focusNextToast`).
2. With a toast focused, Up moves focus to the previous toast (`notifications.focusPreviousToast`).
3. With a toast focused, PageUp or Home jumps to the first toast (`notifications.focusFirstToast`).
4. With a toast focused, PageDown or End jumps to the last toast (`notifications.focusLastToast`).

### NT-1.11
1. A notification with silent priority never shows a toast.
2. A silent notification still appears in the notification center.

### NT-1.12
1. A toast with optional priority is skipped when you are typing in an input inside the corner where the toast would land.
2. In stock, the corner that is checked for a focused input follows the window quarter chosen by `workbench.notifications.position`. [stock only]
3. In Cursor, the corner that is checked for a focused input follows the side bar side. [cursor only]
4. Toasts with other priorities still show over a focused input.

### NT-1.13
1. Setting `workbench.notifications.position` chooses the toast corner, with values bottom right (default), bottom left, and top right. [stock only]
2. With the position set to top right, toasts sit below the title bar. [stock only]
3. The same setting also moves the notification center and the bell to match the chosen corner. [stock only]
4. The setting shows an experimental tag in the Settings editor and has an automatic experiment mode. [stock only]

### NT-1.14
1. With the side bar on the left, which is the default, the toast stack sits in the bottom left of the window. [cursor only]
2. With the side bar on the right, the toast stack sits in the bottom right of the window. [cursor only]
3. Changing `workbench.sideBar.location` moves the toast stack to the matching corner without a restart. [cursor only]

### NT-1.15
1. Running `cursorai.action.hideNotificationsForMs` hides every toast that is showing right now. [cursor only]
2. For the number of milliseconds passed to the command, new notifications get no toast. [cursor only]
3. The command has no label and does not appear in the Command Palette. [cursor only]
4. Onboarding runs it for 10 minutes after settings are imported from VS Code and for 30 seconds on first landing. [cursor only]

## Unclear

- NT-1.8 piece 4 relies on the low weight of the Escape rule, so a widget that handles Escape wins. It was not tried on screen with a real suggest widget.
- NT-1.9 piece 3 and NT-1.10 depend on the toast having focus. How focus lands on a button versus the list row was not traced.
- The Cursor toast duration fields toastDurationMs and closeNotificationOnToastDismiss are not given pieces because no reader was found in the main bundle.
