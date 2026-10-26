# 10. Notifications (NT)

6 groups, 84 features. Each feature lists its entry points and the pieces that finish it.

## NT-1 Toast popups

(both). Short-lived corner cards, up to three, that auto-hide by severity and pause on hover or focus. Cursor's timers are longer (15, 18, 20 seconds against 10, 12, 15).

15 features.

#### NT-1.1 Toast stack

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same limit of three cards and 450 px basis, but the card list is forced to 340 px, margin 6 px, no border, 6 px radius, glass shadow, z-index 1000
- What: New notifications appear as cards in a window corner, newest on top, at most three at once and at most 450 px wide, and a card that does not fit the window height stays hidden but remains in the center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragments MAX_NOTIFICATIONS=3, MAX_WIDTH=450, css .notification-toast and .notifications-list-container overrides)
- Pieces:
  1. A new notification appears as a card in a corner of the window, and the newest card sits on top of the older ones.
  2. At most three toast cards show at once, and each card is at most 450 px wide.
  3. A card that does not fit in the window height stays hidden but remains in the notification center.
  4. In Cursor the card list is set to 340 px with a 6 px margin, no border, a 6 px corner radius, a glass shadow, and a z-index of 1000. [cursor only]

#### NT-1.2 Slide and fade in

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A new toast slides up from below (from the right edge when the corner is top right) and fades in over 300 ms, and with reduced motion on it appears with no transition.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.reduceMotion
- Source: V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (css rules .notification-fade-in and .notification-fade-in-done, reduced motion transition 0ms rule)
- Pieces:
  1. A new toast slides up from below and fades in over 300 ms.
  2. When toasts are set to the top right, a new toast slides in from the right edge instead of from below. [stock only]
  3. With `workbench.reduceMotion` turned on, a new toast appears with no slide or fade.

#### NT-1.3 Burst guard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: If more than three notifications arrive within 800 ms, the extra ones do not get a toast and can only be found in the notification center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragment SPAM_PROTECTION={interval:800,limit:MAX_NOTIFICATIONS})
- Pieces:
  1. When more than three notifications arrive within 800 ms, the extra ones get no toast.
  2. A notification that got no toast because of the burst can still be found in the notification center.
  3. Notifications that arrive after the 800 ms burst has passed get toasts again.

#### NT-1.4 Toasts step aside for the center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While the notification center is open no new toasts appear, and opening the center removes the toasts that are showing.
- Commands: notifications.showList; notifications.toggleList
- Menus: none found
- Keys: Ctrl+K Ctrl+Shift+N (opens the center, which hides the toasts first)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (fragments isNotificationsCenterVisible check at the start of addToast, showList handler t.hide(),e.show())
- Pieces:
  1. While the notification center is open, a new notification does not get a toast.
  2. Opening the notification center with `notifications.showList` or Ctrl+K Ctrl+Shift+N first removes the toasts that are showing.
  3. Toasts removed because the center opened are still listed in the center.

#### NT-1.5 Auto-hide timers by severity

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - Info 15 s, Warning 18 s, Error 20 s, and a new Success at 15 s; the AI severity has no entry in the table
- What: Each toast hides itself after a time that depends on its severity - Info 10 s, Warning 12 s, Error 15 s in stock - and there is no setting to change the times.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragment PURGE_TIMEOUT={[Info]:15e3,[Warning]:18e3,[Error]:2e4,[Success]:15e3})
- Pieces:
  1. In stock, a toast hides itself after a time set by its severity: Info 10 s, Warning 12 s, Error 15 s, and no setting changes these times. [stock only]
  2. In Cursor, a toast hides itself after Info 15 s, Warning 18 s, or Error 20 s. [cursor only]
  3. In Cursor, a Success toast hides itself after 15 s. [cursor only]
  4. A toast that times out leaves the screen but the notification stays in the notification center.

#### NT-1.6 Hover and focus pause the timer

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A toast does not hide while the mouse is over it, while it has keyboard focus, or while the window is not focused, and the countdown starts again afterwards.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (fragment purgeNotification with MOUSE_OVER and MOUSE_OUT tracking, hostService.hasFocus check, list hasFocus check)
- Pieces:
  1. A toast does not hide while the mouse pointer is over it, and its countdown starts again when the pointer leaves.
  2. A toast does not hide while it has keyboard focus, and its countdown starts again when focus leaves it.
  3. A toast does not hide while the window is not focused, and its countdown starts again when the window regains focus.

#### NT-1.7 Sticky toasts stay until closed

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Some toasts never hide on their own - errors that have buttons, notifications you expanded (when they have no buttons), notifications with running progress, and any marked sticky by the caller.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\notifications.ts; V\src\vs\platform\notification\common\notification.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (fragment get sticky() with the same three rules)
- Pieces:
  1. An Error toast that has buttons stays on screen until it is closed.
  2. A notification that you expanded stays on screen until it is closed, as long as it has no buttons.
  3. A notification with running progress stays on screen while the progress runs.
  4. A notification that its caller marked sticky stays on screen until it is closed.

#### NT-1.8 Hide toasts with Escape

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Pressing Escape while toasts are showing dismisses all of them (they stay in the center), and if a toast had focus the focus goes to the next toast or back to the editor.
- Commands: notifications.hideToasts
- Menus: none found
- Keys: Escape (when notificationToastsVisible, low weight so editor widgets win first); Escape (when notificationToastsVisible && notificationFocus, high weight)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (command notifications.hideToasts, keybinding rules with weights 150 and 300)
- Pieces:
  1. Pressing Escape while toasts are showing hides all of them, and the notifications stay in the notification center.
  2. When a toast has keyboard focus, pressing Escape hides all toasts and puts focus back in the editor.
  3. When the focused toast is closed or times out and other toasts remain, focus moves to the next toast, or to the previous one when there is no next.
  4. When the editor or another widget handles Escape first, such as an open suggest widget, the toasts stay.

#### NT-1.9 Focus a toast from the Command Palette

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Focus Notification Toast command moves keyboard focus to the newest toast so you can reach its buttons and links without the mouse.
- Commands: notifications.focusToasts
- Menus: CommandPalette (when notificationToastsVisible)
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (command notifications.focusToasts, label Focus Notification Toast in nls module vs/workbench/browser/parts/notifications/notificationsCommands)
- Pieces:
  1. Running Focus Notification Toast (`notifications.focusToasts`) from the Command Palette moves keyboard focus to the newest toast.
  2. Focus Notification Toast is listed in the Command Palette only while a toast is visible.
  3. With a toast focused, Tab moves through its buttons and links so they can be used without the mouse.

#### NT-1.10 Move between toasts with the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a toast focused, Down and Up step to the next and previous toast and PageUp, Home, PageDown and End jump to the first and last one.
- Commands: notifications.focusNextToast; notifications.focusPreviousToast; notifications.focusFirstToast; notifications.focusLastToast
- Menus: none found
- Keys: Down; Up; PageUp or Home; PageDown or End (all when notificationFocus && notificationToastsVisible)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (the four command ids and the same four key rules)
- Pieces:
  1. With a toast focused, Down moves focus to the next toast (`notifications.focusNextToast`).
  2. With a toast focused, Up moves focus to the previous toast (`notifications.focusPreviousToast`).
  3. With a toast focused, PageUp or Home jumps to the first toast (`notifications.focusFirstToast`).
  4. With a toast focused, PageDown or End jumps to the last toast (`notifications.focusLastToast`).

#### NT-1.11 Silent notifications skip the toast

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A notification with silent priority never shows a toast and only appears in the notification center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\platform\notification\common\notification.ts; C bundle (fragment addToast returns early when priority is SILENT)
- Pieces:
  1. A notification with silent priority never shows a toast.
  2. A silent notification still appears in the notification center.

#### NT-1.12 Optional toasts keep off a focused input

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the checked quarter follows the side bar side instead of a position setting
- What: A toast with optional priority is skipped when you are typing in an input inside the corner where toasts would land, so it cannot cover what you are typing into.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.notifications.position (stock only, decides which window quarter is checked)
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (fragment isElementInNotificationQuarter using getSideBarPosition)
- Pieces:
  1. A toast with optional priority is skipped when you are typing in an input inside the corner where the toast would land.
  2. In stock, the corner that is checked for a focused input follows the window quarter chosen by `workbench.notifications.position`. [stock only]
  3. In Cursor, the corner that is checked for a focused input follows the side bar side. [cursor only]
  4. Toasts with other priorities still show over a focused input.

#### NT-1.13 Pick the toast corner with a setting

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A setting puts toasts in the bottom right (default), bottom left, or top right, where they sit below the title bar and slide in from the right, and the same setting also moves the center and the bell (the header dropdown that writes it is NT-3.11); it is tagged experimental with an automatic experiment mode.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.notifications.position
- Source: V\src\vs\workbench\browser\workbench.contribution.ts; V\src\vs\workbench\common\notifications.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; C bundle (searched workbench.notifications.position, setNotificationsPosition, configureNotificationsPosition, css bottom-left and top-right classes: no hits)
- Pieces:
  1. Setting `workbench.notifications.position` chooses the toast corner, with values bottom right (default), bottom left, and top right. [stock only]
  2. With the position set to top right, toasts sit below the title bar. [stock only]
  3. The same setting also moves the notification center and the bell to match the chosen corner. [stock only]
  4. The setting shows an experimental tag in the Settings editor and has an automatic experiment mode. [stock only]

#### NT-1.14 Toasts follow the side bar side

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor puts the toast stack in the bottom left when the side bar is on the left (the default) and in the bottom right when the side bar is on the right, and it moves when the side bar location changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Source: C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragments configureToastContainerPosition, getSideBarPosition()===1, onDidChangeConfiguration check for workbench.sideBar.location; css .notifications-toasts{bottom:25px;right:3px})
- Pieces:
  1. With the side bar on the left, which is the default, the toast stack sits in the bottom left of the window. [cursor only]
  2. With the side bar on the right, the toast stack sits in the bottom right of the window. [cursor only]
  3. Changing `workbench.sideBar.location` moves the toast stack to the matching corner without a restart. [cursor only]

#### NT-1.15 Hide toasts for a set time

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A Cursor-only command hides every toast now and blocks new toasts for a number of milliseconds; it has no label and is not in the Command Palette, and only the onboarding code calls it (10 minutes after import from VS Code, 30 seconds on first landing).
- Commands: cursorai.action.hideNotificationsForMs
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/browser/parts/notifications/notificationsToasts, fragments hideForMs, shouldHideUntil, command id cursorai.action.hideNotificationsForMs, two executeCommand calls with 10*6e4 and 3e4)
- Pieces:
  1. Running `cursorai.action.hideNotificationsForMs` hides every toast that is showing right now. [cursor only]
  2. For the number of milliseconds passed to the command, new notifications get no toast. [cursor only]
  3. The command has no label and does not appear in the Command Palette. [cursor only]
  4. Onboarding runs it for 10 minutes after settings are imported from VS Code and for 30 seconds on first landing. [cursor only]


## NT-2 Notification contents and buttons

(both). The severity icon, message with links, source, buttons, gear menu, and close. Examples: primary and dropdown buttons with "Don't Show Again"; expand and collapse; per-source mute; Copy Text.

17 features.

#### NT-2.1 Severity icon

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - Info uses the info-two glyph and Warning uses warning-two, Error is the same error glyph
- What: Each notification starts with an icon for its severity (info, warning, error), colored by the theme's notification icon colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; V\src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css; V\src\vs\workbench\common\theme.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsViewer, fragment toSeverityIcon, css .notification-list-item-icon.codicon.codicon-info-two and codicon-warning-two)
- Pieces:
  1. Each notification row starts with an icon that matches its severity: info, warning, or error.
  2. The severity icon is colored by the theme's notification info, warning, and error icon colors.
  3. In Cursor, an Info notification shows the info-two glyph and a Warning notification shows the warning-two glyph, while Error keeps the same error glyph. [cursor only]

#### NT-2.2 Success notifications

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor adds a Success severity that shows a check (pass) icon and a 15 s toast timer, and callers use it for confirmations such as the feedback thanks, applied worktree changes, and the bug report sent message.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/browser/parts/notifications/notificationsViewer, fragments case Severity.Success returns the pass icon, Severity enum Success=5, notify calls with severity Success and message Thanks for your feedback!)
- Pieces:
  1. A notification with Success severity shows a check (pass) icon in its row. [cursor only]
  2. Sending feedback shows a Success notification reading Thanks for your feedback! [cursor only]
  3. Applying changes from a worktree shows a Success notification that confirms the changes were applied. [cursor only]
  4. Sending a bug report shows a Success notification saying the report was sent. [cursor only]

#### NT-2.3 AI severity

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor's severity list also has an AI value (4); it would draw the default info icon, and no caller, icon, or style for it was found, so no one is known to see it today.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (Severity enum fragment e[e.AI=4]="AI" in the severity module, toSeverityIcon default branch, PURGE_TIMEOUT table without an AI entry)
- Pieces:
  1. A notification raised with Cursor's AI severity draws the default info icon. [cursor only]
  2. An AI severity toast has no entry in Cursor's auto-hide table, so its hide time is not defined, and no feature that raises one was found. [cursor only]

#### NT-2.4 Links in the message

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A message can contain text links that open a web page or run a command, and they work with a click, a tap, or Space and Enter when a link has focus; command links show the hover text Click to execute command and the command id.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\base\common\linkedText.ts; C bundle (nls module vs/workbench/browser/parts/notifications/notificationsViewer, label Click to execute command '{0}')
- Pieces:
  1. A notification message can contain a text link, and clicking a link to a web page opens that page.
  2. Clicking a command link in a message runs that command.
  3. Hovering a command link shows the text Click to execute command and the command id.
  4. A link can be used with a tap, and with Space or Enter when the link has keyboard focus.

#### NT-2.5 Long messages

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - same 1000 character cut, but text is 12 px on a 16 px line in a 28 px row (stock 22 px line, 42 px row)
- What: Messages are cut at 1000 characters with three dots, line breaks become spaces, a collapsed row shows one line with an ellipsis and a hover with the full text, and an expanded row wraps the whole message.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\notifications.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsList.css; C bundle (fragments MAX_MESSAGE_LENGTH=1e3, ROW_HEIGHT=28, LINE_HEIGHT=16, css .notification-list-item-message)
- Pieces:
  1. A message longer than 1000 characters is cut at 1000 characters and ends with three dots.
  2. Line breaks in a message show as spaces.
  3. A collapsed row shows the message on one line and ends it with an ellipsis when it does not fit.
  4. Hovering a collapsed message that does not fit shows the full text.
  5. An expanded row wraps the whole message across as many lines as it needs.
  6. In stock, message text uses a 22 px line in a 42 px collapsed row. [stock only]
  7. In Cursor, message text is 12 px on a 16 px line in a 28 px collapsed row. [cursor only]

#### NT-2.6 Source line

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - source text is 11 px at 70 percent opacity with a minimum width of 10 characters
- What: An expanded notification that has a source shows a line reading Source and the name of the extension or area that raised it, with the name again on hover.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsList.css; C bundle (label Source: {0} in module vs/workbench/browser/parts/notifications/notificationsViewer, css .notification-list-item-source)
- Pieces:
  1. An expanded notification that has a source shows a line reading Source followed by the name of the extension or area that raised it.
  2. Hovering the source line shows the source name again.
  3. A collapsed notification, or one with no source, shows no source line.
  4. In Cursor, the source text is 11 px at 70 percent opacity with a minimum width of 10 characters. [cursor only]

#### NT-2.7 Expand and collapse

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A notification can be opened to show the full message, the source, and the buttons, or closed to one line, with the chevron button, Right and Left, Space or Enter, or a double click, except that a notification with buttons is always open and cannot be collapsed.
- Commands: notification.expand; notification.collapse; notification.toggle
- Menus: none found
- Keys: Right (when notificationFocus); Left (when notificationFocus); Space or Enter (when notificationFocus)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\common\notifications.ts; C bundle (command ids, key rules, labels Expand Notification and Collapse Notification, fragment onMouseDblClick toggle)
- Pieces:
  1. Clicking the chevron on a notification expands a collapsed one and collapses an expanded one, with hover text Expand Notification or Collapse Notification.
  2. With a notification focused, Right expands it (`notification.expand`) and Left collapses it (`notification.collapse`).
  3. With a notification focused, Space or Enter toggles it between open and closed (`notification.toggle`).
  4. Double clicking a notification toggles it between open and closed.
  5. An expanded notification shows the full message, the source, and the buttons, and a collapsed one shows a single line.
  6. A collapsed notification shows the chevron only when it has a source or its message does not fit on one line.
  7. A notification that has buttons is always open and cannot be collapsed.

#### NT-2.8 Row icons show on hover or focus

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - close icon is dimmed to 50 percent opacity and the icon strip is 16 px high
- What: The gear, chevron, and close icons stay hidden until you hover the row, focus the list, or expand the row, and each icon's hover text includes its key where one exists.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\media\notificationsList.css; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; C bundle (css .notification-list-item-toolbar-container rules)
- Pieces:
  1. The gear, chevron, and close icons are hidden until you hover the row or focus the list.
  2. An expanded row always shows its icons.
  3. Each icon's hover text includes its keyboard shortcut when it has one.
  4. In Cursor, the close icon is dimmed to 50 percent opacity and the icon strip is 16 px high. [cursor only]

#### NT-2.9 Buttons under the message

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - buttons are added in reverse order, so the first (primary) button ends up last in the row; text is 12 px with a 3 px radius; not seen on screen
- What: Choices passed with a notification show as buttons in the details row under the message, the first in primary style and the rest secondary, and clicking one runs it and closes the notification unless the choice asked to stay open.
- Commands: workbench.dialog.choice.<n> (action id pattern, not a registered command)
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; V\src\vs\workbench\common\notifications.ts; C bundle (fragment renderButtons with a loop that counts down from the last choice and secondary:!(s===0), action id pattern workbench.dialog.choice., css .notification-list-item-buttons-container .monaco-button)
- Pieces:
  1. Choices passed with a notification show as buttons in the details row under the message of an expanded notification.
  2. In stock, the first choice shows first in the row as a primary button and the rest follow as secondary buttons. [stock only]
  3. Clicking a button runs its choice and closes the notification.
  4. A choice that asked to stay open leaves the notification open after its button is clicked.
  5. A button label too long to fit shows its full text as a hover title.
  6. In Cursor, buttons are added in reverse order, so the primary button ends up last in the row, and this was not checked on screen. [cursor only]
  7. In Cursor, button text is 12 px with a 3 px corner radius. [cursor only]

#### NT-2.10 Button with a dropdown

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A primary choice can carry a menu of extra choices, in which case its button shows a dropdown arrow, and secondary choices cannot have one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\platform\notification\common\notification.ts; C bundle (fragment addButtonWithDropdown in renderButtons)
- Pieces:
  1. A primary choice that has a menu of extra choices shows its button with a dropdown arrow.
  2. Opening the dropdown lists the extra choices, and picking one runs it.
  3. A secondary choice cannot have a menu, so it never shows a dropdown arrow.

#### NT-2.11 Don't Show Again

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A caller can ask for a Don't Show Again choice, which appears as the first button or as the last gear-menu entry, closes the notification, and remembers the answer for the workspace, the profile, or the whole application.
- Commands: workbench.notification.neverShowAgain (action id, not a registered command)
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\notification\common\notificationService.ts; V\src\vs\platform\notification\common\notification.ts; C bundle (label Don't Show Again in module vs/workbench/services/notification/common/notificationService, action id workbench.notification.neverShowAgain)
- Pieces:
  1. A notification that asks for Don't Show Again shows it as the first button.
  2. A notification that asks for Don't Show Again as a secondary choice shows it as the last entry in the gear menu.
  3. Choosing Don't Show Again closes the notification.
  4. The answer is remembered for the workspace, the profile, or the whole application, as the notification asked, and that notification never appears again.

#### NT-2.12 Gear menu with extra actions

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a notification has secondary choices, a gear icon (More Actions...) opens a drop-down menu that lists them.
- Commands: workbench.action.configureNotification
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (command id, labels More Actions... and Notification Actions in modules notificationsActions and notificationsViewer)
- Pieces:
  1. A notification that has secondary choices shows a gear icon with hover text More Actions...
  2. Clicking the gear opens a drop-down menu that lists the secondary choices.
  3. Picking an entry from the gear menu runs that choice.
  4. A notification with no secondary choices shows no gear icon.

#### NT-2.13 Mute a source from the gear menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: On a notification that has both a named source and secondary actions, the gear menu starts with Turn Off Info and Warning Notifications from the source (or Turn On All Notifications from it when already muted), which silences that source's non-error notifications.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (labels Turn On All Notifications from '{0}' and Turn Off Info and Warning Notifications from '{0}' in module notificationsViewer, storage key notifications.perSourceDoNotDisturbMode)
- Pieces:
  1. On a notification with a named source and secondary choices, the gear menu starts with Turn Off Info and Warning Notifications from that source.
  2. When that source is already muted, the first gear menu entry reads Turn On All Notifications from that source instead.
  3. Choosing the mute entry flips that source's mute state, so the next gear menu opened for the source shows the other wording.
  4. A separator sits between the mute entry and the notification's own secondary choices.

#### NT-2.14 Close a notification

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The X button, or Delete (Cmd+Backspace on macOS) on a focused notification, closes it, and while progress is running the button is hidden and the key does nothing.
- Commands: notification.clear
- Menus: none found
- Keys: Delete (when notificationFocus); Cmd+Backspace on macOS (when notificationFocus)
- Settings: accessibility.signals.clear
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (command id notification.clear, key rule with Delete and mac Backspace, label Clear Notification)
- Pieces:
  1. Clicking the X button on a notification closes it.
  2. Pressing Delete on a focused notification closes it (`notification.clear`), and Cmd+Backspace does the same on macOS.
  3. While a notification has running progress, the X button is hidden and Delete does nothing.
  4. Clearing a notification plays a signal, a sound and an announcement, controlled by `accessibility.signals.clear`.

#### NT-2.15 Close with a middle click

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Clicking a notification with the middle mouse button closes it (unless it has running progress) and does not paste into the editor behind it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; C bundle (fragment AUXCLICK handler with button===1 and the hasProgress check)
- Pieces:
  1. Clicking a notification with the middle mouse button closes it.
  2. A middle click on a notification does not paste into the editor behind it.
  3. A middle click does nothing while the notification has running progress.

#### NT-2.16 Copy a notification's text

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Right-clicking a notification shows one menu item, Copy Text, which copies the full original message to the clipboard.
- Commands: workbench.action.copyNotificationMessage
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (command id, label Copy Text in module vs/workbench/browser/parts/notifications/notificationsActions)
- Pieces:
  1. Right-clicking a notification shows a menu with one item, Copy Text.
  2. Choosing Copy Text copies the full original message to the clipboard (`workbench.action.copyNotificationMessage`).
  3. The right-click menu works on notifications in toasts and in the notification center.

#### NT-2.17 Accept the first button from the keyboard

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the key rule has no screen reader condition, so it is live whenever a notification has focus or a toast is visible
- What: The Accept Notification Primary Action command runs the first button of the focused notification (or the newest one) and closes it, and in stock the Ctrl+Shift+A key for it only works with a screen reader on.
- Commands: notification.acceptPrimaryAction
- Menus: CommandPalette
- Keys: Ctrl+Shift+A (Cmd+Shift+A on macOS), when screen reader mode is on and (notificationFocus or notificationToastsVisible)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (command id, label Accept Notification Primary Action, key rule fragment with when:ge.or(Ime,OOe))
- Pieces:
  1. Running Accept Notification Primary Action (`notification.acceptPrimaryAction`) from the Command Palette runs the first button of the focused notification, or the newest notification when none is focused, and closes it.
  2. Pressing Ctrl+Shift+A (Cmd+Shift+A on macOS) does the same as the command.
  3. In stock, the key works only when screen reader mode is on and a notification has focus or a toast is visible. [stock only]
  4. In Cursor, the key works whenever a notification has focus or a toast is visible, with no screen reader condition. [cursor only]
  5. When the notification has no buttons, the command does nothing.


## NT-3 Bell and notification center

(both). The status bar bell with an unread dot opens the center list and header toolbar. Examples: Clear All; Do Not Disturb; Hide; the Show, Hide and Clear All commands. Cursor lacks the position setting and the title bar bell.

11 features.

#### NT-3.1 Bell in the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the bell is always right-aligned and rightmost, where stock moves it to the left side for a bottom-left position and removes it for top right
- What: A bell button sits at the far right of the status bar (entry id status.notifications), a click opens the notification center, a second click closes it, and while the center is open the bell shows a beak pointing at it.
- Commands: notifications.showList; notifications.hideList
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsStatus, fragments addEntry(t,"status.notifications",1,Number.NEGATIVE_INFINITY) and showBeak:this.isNotificationsCenterVisible)
- Pieces:
  1. A bell button sits at the far right of the status bar (entry id `status.notifications`).
  2. Clicking the bell opens the notification center (`notifications.showList`).
  3. Clicking the bell again while the center is open closes it (`notifications.hideList`).
  4. While the center is open, the bell shows a beak pointing at the center.
  5. In Cursor, the bell is always right-aligned and rightmost in the status bar, whatever the toast side. [cursor only]

#### NT-3.2 Unread dot on the bell

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The bell changes from a plain bell to a bell with a dot when a notification arrived while the center was closed, or when progress is running and neither toasts nor the center are showing, and opening the center clears it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (fragments newNotificationsCount, the bell-dot text in updateNotificationsCenterStatusItem)
- Pieces:
  1. The bell changes from a plain bell to a bell with a dot when a notification arrives while the center is closed.
  2. The bell shows the dot while a notification has running progress and neither toasts nor the center are showing.
  3. Opening the center clears the unread dot from the bell.
  4. Closing a notification that has not been seen lowers the unread count, and the dot goes away once no unread notification is left.

#### NT-3.3 Bell tooltips

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Hovering the bell shows a count such as 1 New Notification, 3 New Notifications, No New Notifications (1 in progress), No Notifications, or Hide Notifications while the center is open.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (labels at nls indexes 3904 to 3911 in module vs/workbench/browser/parts/notifications/notificationsStatus, fragment getTooltip)
- Pieces:
  1. With one new notification, hovering the bell shows 1 New Notification.
  2. With several new notifications, hovering the bell shows a count such as 3 New Notifications.
  3. With notifications in the list but none new, hovering the bell shows No New Notifications.
  4. With no notifications at all, hovering the bell shows No Notifications.
  5. When notifications have running progress, the tooltip adds a count after the text, such as No New Notifications (1 in progress).
  6. While the center is open, hovering the bell shows Hide Notifications.

#### NT-3.4 Bell moves left for the bottom-left position

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: When notifications are set to the bottom left, the bell moves from the right side of the status bar to the far left of the left side.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.notifications.position
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (the status entry is added with fixed right alignment and no getDesiredAlignment branch; the position setting is not in the bundle)
- Pieces:
  1. With the notification position set to bottom left, the bell moves from the right side of the status bar to the far left of the left side. [stock only]
  2. Changing `workbench.notifications.position` moves the bell at once without a restart. [stock only]

#### NT-3.6 Open the notification center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Show Notifications command, the bell, or the Ctrl+K Ctrl+Shift+N chord opens the center with the first notification focused, and running it again while the center is open just refocuses the first one.
- Commands: notifications.showList
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+Shift+N
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; C bundle (command id, label Show Notifications in module vs/workbench/browser/parts/notifications/notificationsCommands, key rule fragment with the chord on KeyN)
- Pieces:
  1. Running Show Notifications (`notifications.showList`) from the Command Palette opens the notification center with the first notification focused.
  2. Pressing Ctrl+K Ctrl+Shift+N opens the notification center with the first notification focused.
  3. Running the show command while the center is already open focuses the first notification again and does not reopen the center.

#### NT-3.7 Close the notification center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The center closes with the Hide button in its header, Escape, the Hide Notifications command, or a second click on the bell, it closes itself when the last notification goes, and focus returns to the editor if the center had it.
- Commands: notifications.hideList
- Menus: CommandPalette (when notificationCenterVisible)
- Keys: Escape (when notificationCenterVisible)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (command id, label Hide Notifications, key rule with weight 250 and Escape)
- Pieces:
  1. Clicking the Hide button in the center header closes the center.
  2. Pressing Escape while the center is open closes it.
  3. Running Hide Notifications (`notifications.hideList`) from the Command Palette closes the center, and the command is listed only while the center is open.
  4. The center closes by itself when its last notification goes away.
  5. When the center had keyboard focus as it closed, focus returns to the editor.

#### NT-3.8 The center list

- Tag: both. Evidence: source. Boundary: no. Cursor: changed - the panel always sits bottom right even when toasts are on the left, the list is forced to 340 px with a dropdown-style background, the header toolbar has Clear All, Do Not Disturb and Hide with no position button
- What: The center is a panel above the status bar with a header titled Notifications (or No new notifications when empty) and a scrolling list, at most 450 by 400 px, of every notification still open, which updates as notifications come and go.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css; C bundle (module vs/workbench/browser/parts/notifications/notificationsCenter, labels at nls 3877 to 3883, fragments MAX_DIMENSIONS=new(450,400), header toolbar build, css .notifications-center)
- Pieces:
  1. The center is a panel above the status bar with a header titled Notifications.
  2. When there are no notifications, the header title reads No new notifications.
  3. Under the header, a scrolling list shows every notification that is still open, and the panel is at most 450 by 400 px.
  4. The list updates while the center is open as notifications are added, removed, or changed.
  5. In stock, the header toolbar holds Clear All Notifications, Do Not Disturb, a position button, and Hide. [stock only]
  6. In stock, the center follows `workbench.notifications.position`: bottom right, bottom left, or below the title bar at the top right, where its Hide icon points up. [stock only]
  7. In Cursor, the header toolbar holds Clear All, Do Not Disturb, and Hide, with no position button. [cursor only]
  8. In Cursor, the panel always sits at the bottom right even when toasts are on the left, and the list is set to 340 px with a dropdown-style background. [cursor only]

#### NT-3.9 Clear all notifications

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The Clear All Notifications header button or command closes the center and every notification that has no running progress, and the button is greyed out when there is nothing to clear.
- Commands: notifications.clearAll
- Menus: CommandPalette
- Keys: none found
- Settings: accessibility.signals.clear
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (command id, label Clear All Notifications, fragments clearAll and updateTitle with the enabled check)
- Pieces:
  1. Clicking Clear All Notifications in the center header closes the center and every notification that has no running progress.
  2. Running Clear All Notifications (`notifications.clearAll`) from the Command Palette does the same.
  3. A notification with running progress stays after Clear All.
  4. The Clear All button is greyed out when the list is empty or every notification has running progress.
  5. Clearing all notifications plays a signal, controlled by `accessibility.signals.clear`.

#### NT-3.10 Keyboard focus and navigation in the center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Opening the center focuses the first notification, Up and Down move between them, and the buttons and links of a row come first in tab order because the details row sits before the message in the page.
- Commands: list.focusDown; list.focusUp
- Menus: none found
- Keys: Down (when a list has focus); Up (when a list has focus)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\actions\listCommands.ts; C bundle (ids list.focusDown and list.focusUp found, same row render order)
- Pieces:
  1. Opening the center puts keyboard focus on the first notification.
  2. Down and Up move focus to the next and previous notification in the list (`list.focusDown`, `list.focusUp`).
  3. In a notification row, Tab reaches the buttons and links before the message, because the details row comes first in the page.
  4. A screen reader announces the list as Notifications Center and the header toolbar as Notification Center Actions.

#### NT-3.11 Position dropdown in the center header

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: A position icon in the center header opens a drop-down with Bottom Right, Bottom Left, and Top Right (the current one checked), and picking one writes the notification position setting.
- Commands: workbench.action.configureNotificationsPosition; workbench.action.setNotificationsPosition.bottomRight; workbench.action.setNotificationsPosition.bottomLeft; workbench.action.setNotificationsPosition.topRight
- Menus: NotificationsCenterPositionMenu
- Keys: none found
- Settings: workbench.notifications.position
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\platform\actions\common\actions.ts; C bundle (searched setNotificationsPosition, configureNotificationsPosition, NotificationsCenterPositionMenu, Configure Notifications Position label: no hits)
- Pieces:
  1. A position icon in the center header, with hover text Configure Notifications Position..., opens a drop-down. [stock only]
  2. The drop-down lists Bottom Right, Bottom Left, and Top Right, in that order. [stock only]
  3. The entry for the current position is shown checked. [stock only]
  4. Picking an entry writes `workbench.notifications.position` and the notifications move to that position. [stock only]

#### NT-3.12 Read a notification in the accessible view

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: With a notification focused, Open Accessible View shows its text and source in the accessible view with its buttons as actions and lets you step to the next or previous notification, and a setting adds a hint about this to each notification.
- Commands: editor.action.accessibleView
- Menus: CommandPalette
- Keys: Alt+F2 (Alt+Shift+F2 on Linux)
- Settings: accessibility.verbosity.notification
- Source: V\src\vs\workbench\browser\parts\notifications\notificationAccessibleView.ts; V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibleViewActions.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; V\src\vs\workbench\browser\workbench.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationAccessibleView, label {0} Source: {1} at nls 3853, ids editor.action.accessibleView and accessibility.verbosity.notification)
- Pieces:
  1. With a notification focused, Open Accessible View (`editor.action.accessibleView`, Alt+F2, or Alt+Shift+F2 on Linux) shows the notification text and its source in the accessible view.
  2. The accessible view text starts with Error:, Warning:, or Info: to match the notification's severity.
  3. The accessible view lists the notification's buttons and secondary choices as actions, plus a Clear Notification action, and running one closes the notification.
  4. The accessible view lets you step to the next or previous notification.
  5. Setting `accessibility.verbosity.notification` adds a hint about opening the accessible view to each notification.


## NT-4 Do Not Disturb

(both). Makes non-error notifications silent, globally or per source, with a bell-slash icon. Includes Zen Mode silence.

12 features.

#### NT-4.1 Turn Do Not Disturb on or off

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Toggle Do Not Disturb Mode from the Command Palette to flip the global mode, which silences every info and warning notification (errors still show).
- Commands: notifications.toggleDoNotDisturbMode
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsCommands, id notifications.toggleDoNotDisturbMode, nls 3890)
- Pieces:
  1. Running Toggle Do Not Disturb Mode (`notifications.toggleDoNotDisturbMode`) from the Command Palette turns Do Not Disturb on when it is off.
  2. Running the same command again turns Do Not Disturb off.
  3. The command flips one global switch that applies to every source at once, separate from the per-source choices.

#### NT-4.2 Do Not Disturb dropdown in the center header

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The bell-slash button in the notification center header opens a menu with Enable Do Not Disturb Mode or Disable Do Not Disturb Mode, then a checked list of up to 10 sources sorted by label (checked means that source may still show everything), then More… when there are over 10. The dropdown is built in code, so it is not a MenuId. While the mode is on the center closes, and clicking the bell reopens it.
- Commands: workbench.action.configureDoNotDisturbMode; notifications.toggleDoNotDisturbMode; notifications.toggleDoNotDisturbModeBySource
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsCenter and notificationsActions, id workbench.action.configureDoNotDisturbMode, nls 3868, 3880, 3881, 3882, MAX_NOTIFICATION_SOURCES=10)
- Pieces:
  1. A bell-slash button in the notification center header, with hover text Configure Do Not Disturb..., opens a menu (`workbench.action.configureDoNotDisturbMode`).
  2. The first menu entry reads Enable Do Not Disturb Mode when the mode is off and Disable Do Not Disturb Mode when it is on, and choosing it flips the global mode.
  3. Below a separator the menu lists up to 10 sources sorted by label, each as a checkable entry, where checked means that source may still show everything.
  4. Choosing a source entry flips that source between showing everything and silencing info and warning notifications (`notifications.toggleDoNotDisturbModeBySource`).
  5. When more than 10 sources are known, a separator and a More… entry follow the list, and More… opens the source picker.
  6. When no source has shown a notification yet, the menu holds only the enable or disable entry.

#### NT-4.3 Choose sources in a picker

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Run Toggle Do Not Disturb Mode By Source... to get a multi-select quick pick with the placeholder "Select sources to enable all notifications from". Ticked sources keep showing everything, unticked sources become silent for info and warning, and accepting applies the choice. Only sources that have already shown a notification in this install are listed.
- Commands: notifications.toggleDoNotDisturbModeBySource
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsCommands, id notifications.toggleDoNotDisturbModeBySource, nls 3884, 3891)
- Pieces:
  1. Running Toggle Do Not Disturb Mode By Source... from the Command Palette opens a multi-select quick pick with the placeholder Select sources to enable all notifications from.
  2. The picker lists each source by its label, sorted by label, and hovering a row shows the label with the source id in brackets.
  3. Sources that currently show everything start ticked, and sources that are currently silent for info and warning start unticked.
  4. Accepting the picker applies the ticks to every source at once and closes the picker, so ticked sources keep showing everything and unticked sources become silent for info and warning.
  5. Closing the picker without accepting changes nothing.
  6. Only sources that have already shown a notification in this install are listed.

#### NT-4.4 Mute one source from a notification's gear menu

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The More Actions... gear menu on a notification that has a source offers Turn Off Info and Warning Notifications from '{source}', or Turn On All Notifications from '{source}' when that source is already muted. The choice flips that one source's filter, the same one the picker and the header dropdown change.
- Commands: workbench.action.configureNotification
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsViewer, id workbench.action.configureNotification, nls 3872, 3917, 3918)
- Pieces:
  1. After choosing Turn Off Info and Warning Notifications from a source in a notification's gear menu, later info and warning notifications from that source are silent while its errors still show.
  2. After that choice, the source's entry in the center header dropdown loses its check mark and its row in the source picker is unticked.
  3. Choosing Turn On All Notifications from the source brings back the check mark and the tick, and its notifications show again.
  4. The mute entry changes only that one source and leaves the global mode and every other source as they were.

#### NT-4.5 Bell-slash icon in the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: While Do Not Disturb is on, the status bar bell turns into a bell-slash (a bell-slash-dot when something new arrived or progress is running), with the accessible name Do Not Disturb and the tooltip Do Not Disturb Mode is Enabled. Clicking it still shows or hides the center.
- Commands: notifications.showList; notifications.hideList; notifications.toggleDoNotDisturbMode; notifications.toggleDoNotDisturbModeBySource
- Menus: commandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsStatus, status entry status.notifications, bell-slash and bell-slash-dot, nls 3902, 3903); V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts (NotificationsFilter.ERROR branch); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (icons bell-slash and bell-slash-dot, NLS keys status.doNotDisturb and status.doNotDisturbTooltip, commands notifications.toggleDoNotDisturbMode and notifications.toggleDoNotDisturbModeBySource)
- Pieces:
  1. While Do Not Disturb is on, the status bar bell shows as a bell-slash.
  2. While Do Not Disturb is on, the bell-slash gets a dot when a notification arrived while the center was closed or when progress is running.
  3. The bell-slash has the accessible name Do Not Disturb and the hover text Do Not Disturb Mode is Enabled.
  4. Clicking the bell-slash opens the notification center when it is closed and closes it when it is open (`notifications.showList`, `notifications.hideList`).
  5. Turning Do Not Disturb off puts the plain bell back.
  6. With Do Not Disturb on, the bell icon becomes a bell with a slash.
  7. The slashed bell gets a dot when something is unread or in progress.
  8. The tooltip reads "Do Not Disturb Mode is Enabled" and the screen reader name is "Do Not Disturb".
  9. Clicking the bell still opens the notification center in this state.
  10. Switching Do Not Disturb with Toggle Do Not Disturb Mode (`notifications.toggleDoNotDisturbMode`) or the per source toggle from the Command Palette changes the bell at once.

#### NT-4.6 Turning Do Not Disturb on clears what is on screen

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Switching the global mode on hides every visible toast and closes the notification center. Switching one source off removes that source's visible info and warning toasts, while its errors and urgent items stay.
- Commands: notifications.toggleDoNotDisturbMode
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsToasts and notificationsCenter, the onDidChangeFilter handlers)
- Pieces:
  1. Switching the global mode on hides every toast that is showing.
  2. Switching the global mode on closes the notification center if it is open.
  3. Switching one source off removes that source's visible info and warning toasts.
  4. Error toasts and urgent toasts from a muted source stay on screen.

#### NT-4.7 What still pops up: errors and urgent notifications

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (Cursor adds Success and AI severities; the filter only exempts Error, so Success is silenced like Info and Warning; the memory warning prompt is Urgent so it still shows)
- What: With Do Not Disturb on, any notification with Error severity and any with Urgent priority still shows as a toast. Only Default and Optional priority items below error severity become silent. Stock callers that set Urgent include the window, integrity, extension host and save participant prompts.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\common\notifications.ts; V\src\vs\platform\notification\common\notification.ts; V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; C bundle (module vs/workbench/common/notifications, the create method with the global and per-source check, module memoryMonitor.contribution)
- Pieces:
  1. With Do Not Disturb on, a notification with Error severity still shows as a toast.
  2. With Do Not Disturb on, a notification with Urgent priority still shows as a toast.
  3. With Do Not Disturb on, notifications with default or optional priority and a severity below Error become silent.
  4. Stock prompts that set Urgent priority, such as window, integrity, extension host, and save participant prompts, still show with Do Not Disturb on.
  5. In Cursor, a Success or AI severity notification is silenced like Info and Warning, because only Error is exempt. [cursor only]
  6. In Cursor, the memory warning prompt is Urgent, so it still shows with Do Not Disturb on. [cursor only]

#### NT-4.8 Silenced notifications still land in the center

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A notification silenced by Do Not Disturb gets no toast and no screen reader announcement, but it is still added to the notification center list and still raises the unread dot on the bell.
- Commands: notifications.showList; notifications.clearAll
- Menus: CommandPalette
- Keys: Ctrl+K Ctrl+Shift+N (notifications.showList)
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsToasts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsAlerts.ts; V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsAlerts, notificationsToasts and notificationsStatus, the Silent priority checks)
- Pieces:
  1. A notification silenced by Do Not Disturb gets no toast.
  2. A silenced notification gets no screen reader announcement.
  3. A silenced notification is still added to the notification center list, which opens with Ctrl+K Ctrl+Shift+N (`notifications.showList`).
  4. A silenced notification still raises the unread dot on the bell, and Clear All Notifications (`notifications.clearAll`) in the center removes it like any other.

#### NT-4.9 Progress stays quiet under Do Not Disturb

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When Do Not Disturb is on globally or for the progress source, a notification-location progress turns silent. It makes no toast and shows only as the left status bar spinner entry until it ends, unless its priority is Urgent.
- Commands: notifications.showList
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; C bundle (module vs/workbench/services/progress/browser/progressService, the case 15 branch that sets the Silent priority)
- Pieces:
  1. With Do Not Disturb on globally, a progress notification asked for in the notification location makes no toast.
  2. With Do Not Disturb on for the progress source, the progress notification makes no toast either.
  3. The silent progress shows only as a spinner entry on the left of the status bar until it ends.
  4. Clicking the spinner entry opens the notification center with the progress notification in it (`notifications.showList`).
  5. A progress notification with Urgent priority still shows as a toast with Do Not Disturb on.

#### NT-4.10 Zen Mode turns Do Not Disturb on

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Entering Zen Mode switches global Do Not Disturb on when zenMode.silentNotifications is true, and leaving Zen Mode switches it off again, but only if the mode was off when Zen Mode began. Changing the setting while in Zen Mode applies at once. Zen Mode itself is owned by the layout groups.
- Commands: workbench.action.toggleZenMode; workbench.action.exitZenMode
- Menus: MenubarAppearanceMenu
- Keys: Ctrl+K Z (workbench.action.toggleZenMode); Escape Escape (workbench.action.exitZenMode, when InEditorZenModeContext)
- Settings: zenMode.silentNotifications
- Source: V\src\vs\workbench\browser\layout.ts; V\src\vs\workbench\browser\workbench.zenMode.contribution.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; C bundle (module vs/workbench/browser/workbench.contribution, setting zenMode.silentNotifications default true, nls 4319, layout code with handleNotificationsDoNotDisturbMode)
- Pieces:
  1. Entering Zen Mode turns global Do Not Disturb on when `zenMode.silentNotifications` is true, which is the default.
  2. Leaving Zen Mode turns Do Not Disturb off again when it was off before Zen Mode began.
  3. When Do Not Disturb was already on before Zen Mode began, leaving Zen Mode leaves it on.
  4. Changing `zenMode.silentNotifications` while in Zen Mode turns Do Not Disturb on or off at once.
  5. With `zenMode.silentNotifications` false, entering Zen Mode leaves Do Not Disturb as it was.

#### NT-4.11 Do Not Disturb choices persist

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (same storage keys; Cursor's service adds a resetGlobalFilter method and a doNotPersist option, with no caller in the main bundle)
- What: The global switch and every per-source choice are saved in application storage (keys notifications.doNotDisturbMode and notifications.perSourceDoNotDisturbMode, not settings) and survive restarts. A source only appears in the lists after its first notification.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (module vs/workbench/services/notification/common/notificationService, keys GLOBAL_FILTER_SETTINGS_KEY and PER_SOURCE_FILTER_SETTINGS_KEY)
- Pieces:
  1. The global Do Not Disturb switch is still on or off after a restart, as it was before.
  2. Every per-source choice is still the same after a restart.
  3. The choices are saved in application storage under `notifications.doNotDisturbMode` and `notifications.perSourceDoNotDisturbMode`, so they do not appear in Settings.
  4. A source appears in the header dropdown and the picker only after its first notification, and stays listed afterward.
  5. In Cursor, the service also has a reset-global-filter method and a do-not-persist option, but nothing in the main window calls them, so no change is visible. [cursor only]

#### NT-4.12 Cursor floating cards ignore Do Not Disturb

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Cursor's update and changelog cards are drawn by a separate component that never goes through the notification service and never reads the Do Not Disturb filter, so turning the mode on does not hide them. See NT-6 for the cards.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (the Zyk update card and Qyk changelog card components, which use updateNotificationService, commandService and openerService but no notification service)
- Pieces:
  1. With Do Not Disturb on, Cursor's update card still appears. [cursor only]
  2. With Do Not Disturb on, Cursor's changelog card still appears. [cursor only]
  3. Turning Do Not Disturb on does not hide an update or changelog card that is already on screen. [cursor only]


## NT-5 Progress notifications

(both). The progress service shows as a notification bar with Cancel and as a status bar spinner. Pieces also live in SS, AB, and DL.

13 features.

#### NT-5.1 Progress notification

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A task started with the notification progress location appears as a notification with a thin progress bar under the message. It is an endless bar until the task reports a total, then a bar that fills by percent.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; V\src\vs\platform\progress\common\progress.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\common\notifications.ts; C bundle (modules vs/workbench/services/progress/browser/progressService and vs/workbench/browser/parts/notifications/notificationsViewer, methods withNotificationProgress and renderProgress)
- Pieces:
  1. A task started with the notification progress location shows a notification with its message and a thin progress bar under the text.
  2. Until the task reports a total, the bar is an endless sweeping bar with no fill.
  3. Once the task reports a total, the bar changes to a fill that grows left to right by the percent of work done.
  4. When the task reports a new step message, the same notification updates its text in place instead of opening another notification.
  5. When the task ends, the notification closes by itself.

#### NT-5.2 Progress title and message line

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The progress title and the latest step message join as "title: message", and the title stands alone until a message arrives. The notification text may hold [text](command:...) links, which the status bar entry turns into plain text, and the status entry tooltip adds a [source] prefix.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (updateWindowProgress, updateNotification); C bundle (module vs/workbench/services/progress/browser/progressService, nls 13067, 13068, 13069)
- Pieces:
  1. A progress notification that has only a title shows just the title, and one with no title shows just the latest step message.
  2. Once a step message arrives, the notification text becomes "title: message".
  3. A later step message replaces the earlier message while the title prefix stays.
  4. Markdown links such as `[text](command:...)` in the progress text show as clickable links inside the notification.
  5. The status bar entry for the same progress shows those links as plain text.
  6. The status bar entry tooltip reads "[source] title: message" when the progress names a source, and plain "title: message" when it does not.

#### NT-5.3 Cancel and extra buttons on a progress notification

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A progress notification can show a Cancel button (the default label or a custom string) plus named buttons. Clicking one stops the task and reports which button was used. The buttons sit under the message like any other notification buttons. The ids here are action ids built in the service, not registered commands.
- Commands: progress.cancel; progress.button.<label>
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (createNotification); V\src\vs\platform\progress\common\progress.ts; C bundle (module vs/workbench/services/progress/browser/progressService, ids progress.cancel and progress.button.{label}, nls 13072)
- Pieces:
  1. A progress notification started as cancellable shows a Cancel button under the message.
  2. When the task supplies its own cancel text, the button shows that text instead of "Cancel".
  3. A progress notification can show extra named buttons, one per name, placed before the Cancel button.
  4. Clicking Cancel stops the task and reports a plain cancel (`progress.cancel`).
  5. Clicking a named button stops the task and reports which button was used (`progress.button.<label>`).
  6. A progress notification that is not cancellable and has no named buttons shows no button row.

#### NT-5.4 Progress notification stays until finished

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: A running progress notification never auto-hides, has no close button, ignores middle-click and the Delete key, and is skipped by Clear All Notifications, so you cannot dismiss it by accident. It also stays at least 800 ms so a quick task does not flash.
- Commands: notification.clear; notifications.clearAll
- Menus: CommandPalette
- Keys: Delete (notification.clear, when a notification has focus; Cmd+Backspace on macOS)
- Settings: none found
- Source: V\src\vs\workbench\common\notifications.ts (sticky getter); V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCenter.ts; V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (modules vs/workbench/browser/parts/notifications/notificationsCommands and notificationsViewer, the hasProgress checks, ids notification.clear and notifications.clearAll)
- Pieces:
  1. A running progress notification never auto-hides, so its toast stays on screen until the task ends.
  2. A running progress notification has no close button.
  3. Middle-clicking a running progress notification, or pressing `Delete` (`Cmd+Backspace` on macOS) with it focused (`notification.clear`), does not remove it.
  4. Running "Clear All Notifications" (`notifications.clearAll`) from the Command Palette removes the other notifications and leaves running progress notifications in place.
  5. A task that ends in under 800 ms still keeps its notification on screen until 800 ms have passed, so it does not flash.

#### NT-5.5 Indeterminate and determinate bars

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: The shared 2 px progress bar has two looks. Indeterminate sweeps endlessly and switches to a stepped animation after 10 seconds. Determinate fills left to right by the work done and grows to 100 percent before it hides. It is reused by notifications, views, containers and editor groups.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\base\browser\ui\progressbar\progressbar.ts; V\src\vs\workbench\services\progress\browser\progressIndicator.ts; C bundle (the ProgressBar class with LONG_RUNNING_INFINITE_THRESHOLD=1e4, css classes monaco-progress-container, infinite, infinite-long-running, discrete, done)
- Pieces:
  1. An indeterminate bar shows a short segment sweeping across a 2 px high line, over and over.
  2. After an indeterminate bar has run for 10 seconds, the sweep changes to a stepped animation.
  3. A determinate bar fills left to right by the share of work done.
  4. When a determinate bar reaches 100 percent it grows to full width and then hides, and an indeterminate bar fades out and hides when its work ends.
  5. The bar uses the theme color `progressBar.background` and looks the same in notifications, views, containers and editor groups.

#### NT-5.6 Window progress in the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Work reported at the window location shows a spinner entry on the left end of the status bar (name Progress Message, entry id status.progress) with the progress text. It appears after 150 ms and stays at least 150 ms. The icon is the loading spinner or the syncing icon, and an optional command runs when you click it. The entry's own look is also listed under the status bar group.
- Commands: notifications.showList
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withWindowProgress, updateWindowProgress); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts; V\src\vs\workbench\services\statusbar\browser\statusbar.ts; C bundle (module vs/workbench/services/progress/browser/progressService, entry id status.progress, nls 13071, showProgress with loading and syncing)
- Pieces:
  1. Work reported at the window location with a command shows a spinner entry named "Progress Message" (entry id `status.progress`) in the status bar.
  2. The entry sits on the left side of the status bar, after all the other left-side entries.
  3. The entry text is "title: message", the title alone, or the message alone, and with neither the entry is not shown.
  4. The entry icon is the loading spinner by default, or the syncing icon when the task asks for it.
  5. Clicking the entry runs the command the task supplied, for example `notifications.showList`.
  6. When several window progress tasks run at once, the entry shows the newest one and falls back to the next one when it ends.

#### NT-5.7 Hidden progress notification falls back to the status bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a progress notification is silent or its toast gets dismissed, the work keeps running as a window progress entry in the status bar. Clicking that entry opens the notification center, where the bar is shown. The entry disappears when the notification shows again or the task ends.
- Commands: notifications.showList; notifications.hideToasts
- Menus: none found
- Keys: Escape (notifications.hideToasts, when notificationToastsVisible)
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (createWindowProgress, onVisibilityChange); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (module vs/workbench/services/progress/browser/progressService, the createWindowProgress arrow with command notifications.showList)
- Pieces:
  1. A progress notification marked silent does not pop up as a toast, and its work shows only as a spinner entry in the status bar.
  2. When notifications are filtered to errors only (Do Not Disturb), a progress notification is silent in the same way.
  3. Dismissing the toast of a running progress notification, for example with `Escape` (`notifications.hideToasts`), leaves the work visible as the status bar entry.
  4. Clicking that status bar entry opens the notification center (`notifications.showList`), where the notification shows its progress bar.
  5. The status bar entry disappears when the notification shows again or when the task ends.
  6. Window progress without a command is shown as a silent notification, so it first appears only as a status bar entry and can be brought forward by clicking it.

#### NT-5.8 Progress bar in a view or container

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Work reported against Explorer, Source Control, Extensions, a view id or a container id shows the thin progress bar at the top of that view, or at the top of the container content. The bar runs only while the view or container is visible and resumes when you switch back to it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withViewProgress, withPaneCompositeProgress, withCompositeProgress); V\src\vs\workbench\services\progress\browser\progressIndicator.ts (ScopedProgressIndicator); V\src\vs\workbench\browser\parts\views\viewPane.ts; V\src\vs\workbench\browser\parts\compositePart.ts; C bundle (modules vs/workbench/browser/parts/views/viewPane and vs/workbench/browser/parts/compositePart, getProgressIndicator)
- Pieces:
  1. Work reported for the Explorer, Source Control or Extensions location shows the thin progress bar at the top of that container's content.
  2. Work reported against a view id shows the bar at the top of that view only.
  3. Work reported against a container id shows the bar at the top of the container content.
  4. The bar runs only while the view or container is visible, and switching away hides it and switching back shows it again if the work is still going.
  5. The bar goes away when the work ends.

#### NT-5.9 Progress badge on the activity bar icon

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Work on a side bar view or container also puts a small spinning progress badge on that container's icon after 300 ms, for at least 300 ms. A progress badge wins over icon and number badges while it runs. The badge look is also listed under the activity bar group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (showOnActivityBar); V\src\vs\workbench\services\progress\browser\media\progressService.css; V\src\vs\workbench\browser\parts\compositeBarActions.ts (getVisibleBadges); V\src\vs\workbench\services\activity\common\activity.ts (ProgressBadge); C bundle (module vs/workbench/services/progress/browser/progressService, showOnActivityBar, class progress-badge)
- Pieces:
  1. Work on a side bar container shows a small spinning progress badge on that container's icon in the activity bar.
  2. Work on a view inside a side bar container puts the same badge on that view's container icon.
  3. While the progress badge runs, it shows instead of any icon badge or number badge on that container.
  4. The progress badge disappears when the work ends.

#### NT-5.10 Progress in a dialog

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Work reported at the dialog location opens a modal pending dialog with the message and an optional detail line. It has a Cancel or Dismiss button, the message updates as progress is reported, and sticky mode removes the button and blocks closing. An optional delay holds the dialog back. The dialog look is also listed under the dialogs group.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts (withDialogProgress); V\src\vs\base\browser\ui\dialog\dialog.ts (pending type); C bundle (module vs/workbench/services/progress/browser/progressService, withDialogProgress with type pending, nls 13073, 13074)
- Pieces:
  1. Work reported at the dialog location opens a modal dialog with a spinning loading icon and the message.
  2. An optional detail line shows under the message.
  3. Unless the dialog is sticky, it has a "Dismiss" button, or "Cancel" (or the task's own cancel text) when the task is cancellable, placed after any named buttons.
  4. Clicking that button closes the dialog and, if the task is still running, tells the task which button was used.
  5. As the task reports messages, the dialog message updates in place.
  6. A sticky dialog has no button and cannot be closed with the close action or the default action until the task ends.
  7. The dialog closes by itself when the task ends.

#### NT-5.11 Delayed and minimum display times

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: Progress avoids flicker. A notification with a delay option waits that long, otherwise it stays at least 800 ms. Window progress waits 150 ms and stays 150 ms. The activity bar badge waits 300 ms (or the caller's delay) and stays 300 ms. View and container bars take the caller's delay, and the dialog takes an optional one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\services\progress\browser\progressService.ts; V\src\vs\base\browser\ui\progressbar\progressbar.ts (show with delay); C bundle (module vs/workbench/services/progress/browser/progressService, delay:150, the 800 ms timeout, the 300 ms badge delay)
- Pieces:
  1. A notification progress started with a delay option stays hidden for that long and shows only if the task is still running, with no 800 ms minimum afterward.
  2. Window progress waits 150 ms before it shows and then stays at least 150 ms, so a task shorter than 150 ms never shows an entry.
  3. The activity bar badge waits 300 ms, or the task's own delay, before it shows and then stays at least 300 ms.
  4. View and container bars wait for the task's own delay before they show, with no fixed default.
  5. A dialog progress waits for an optional delay before it first appears, and later message updates do not wait again.

#### NT-5.12 Progress sound and announcement

- Tag: both. Evidence: source. Boundary: no. Cursor: present
- What: When a progress bar has been showing for 3 seconds, a sound and or a screen reader announcement (Progress) repeats until the bar hides. The setting turns each on or off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: accessibility.signals.progress
- Source: V\src\vs\base\browser\ui\progressbar\progressbar.ts (PROGRESS_SIGNAL_DEFAULT_DELAY 3000); V\src\vs\base\browser\ui\progressbar\progressAccessibilitySignal.ts; V\src\vs\platform\accessibilitySignal\browser\accessibilitySignalService.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (setting accessibility.signals.progress, nls 1754, 4663 to 4665, ProgressBar PROGRESS_SIGNAL_DEFAULT_DELAY=3e3)
- Pieces:
  1. After a progress bar has been showing for 3 seconds, a sound plays and repeats on a loop until the bar hides.
  2. With the announcement turned on, a screen reader announces "Progress" on a loop in the same way until the bar hides.
  3. A bar that hides before 3 seconds makes no sound and no announcement.
  4. Setting `accessibility.signals.progress` has a `sound` value that is `auto` (only with a screen reader attached, the default), `on` or `off`.
  5. Setting `accessibility.signals.progress` has an `announcement` value that is `auto` (only in screen reader optimized mode) or `off`, with `off` as the default.

#### NT-5.13 Slow shell environment progress and failure notice

- Tag: both. Evidence: source. Boundary: no. Cursor: present (the error handler vscode:showResolveShellEnvError with three buttons, the progress with a 1600 ms delay and a Learn More button, and the setting with the same 1 to 120 range and a flag that leaves it out on Windows are in the bundle)
- What: On macOS and Linux, when the app was not started from a terminal, a status bar item reads "Resolving shell environment..." while your shell variables load, and an error notification with Restart, Configure and Learn More appears if that fails or runs out of time.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: application.shellEnvironmentResolutionTimeout
- Source: V\src\vs\workbench\electron-browser\window.ts (error prompt and the progress call); V\src\vs\workbench\electron-browser\desktop.contribution.ts (the setting); V\src\vs\platform\shell\node\shellEnv.ts (when it runs, timeout, messages); V\src\vs\code\electron-main\app.ts (sends the error to the window); C bundle (handler vscode:showResolveShellEnvError, call to shellEnv with delay 1600, setting application.shellEnvironmentResolutionTimeout)
- Pieces:
  1. When the shell environment is still loading after 1.6 seconds, a window progress item reading "Resolving shell environment..." shows in the status bar with a Learn More button that opens the documentation.
  2. If resolving times out or fails, an error notification shows the reason, for example "Unable to resolve your shell environment in a reasonable time. Please review your shell configuration and restart."
  3. The notification's Restart button relaunches the application.
  4. The notification's Configure button opens the user Settings filtered to `application.shellEnvironmentResolutionTimeout`.
  5. The notification's Learn More button opens the documentation page.
  6. `application.shellEnvironmentResolutionTimeout` sets the number of seconds to wait, from 1 to 120, default 10, and applies to the whole application.
  7. On Windows and when the app was started from a terminal nothing is resolved, so neither the progress item nor the error appears, and the setting is left out of the Settings editor on Windows.


## NT-6 Cursor update and what's-new cards

(cursor). Floating cards for new versions with Install Now, Later and Changelog, snoozed up to 24 hours. Examples: the "Update to v" card; the "New update available" card; the "New in" changelog card; classic update prompts. Placement here is a judgment call (medium confidence).

16 features.

#### NT-6.1 "Update to v" card

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A floating card with the eyebrow "Update to v{major.minor}", a one-line blurb, a Changelog link button, a pulsing Install Now button and a close X, shown when the target version has a built-in blurb or the force flag is set.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Zyk mounted by k6m from the side bar part, classes update-notification-container, update-notification-eyebrow, update-notification-title, update-notification-footer; blurb table lut; css update-notification-*)
- Pieces:
  1. A floating card shows the small heading "Update to v{major.minor}", using the version of the ready update. [cursor only]
  2. Under the heading the card shows a one-line blurb about that version.
  3. The card footer has a Changelog link button and a pulsing Install Now button.
  4. The card has a close X.
  5. The card shows only when the target version has a built-in blurb or the force flag is set, and otherwise the compact "New update available" card shows instead.

#### NT-6.2 "New update available" card

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The default compact card when the version has no blurb: a gift icon, the text "New update available", a Later button and an Install Now button.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Zyk fallback branch, classes minor-version-notification-container, minor-version-notification-text, minor-version-notification-buttons; css minor-version-notification-*)
- Pieces:
  1. The compact card shows a gift icon with the text "New update available". [cursor only]
  2. The compact card has a Later button and an Install Now button.
  3. The compact card shows instead of the "Update to v" card when the version has no built-in blurb.

#### NT-6.3 "New in {version}" changelog card

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: After a major or minor version change a small card says "New in {major.minor}" with a one-line blurb, a View changelog button and a close X, shown only when the version has a blurb.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (component Qyk, class changelog-notification, setShouldShowReleaseNotes in the startup class jln that compares releaseNotes/lastVersion with the current version, blurb table lut)
- Pieces:
  1. After a major or minor version change, a small card titled "New in {major.minor}" appears with a one-line blurb. [cursor only]
  2. The card has a View changelog button and a close X.
  3. The card shows only when the new version has a built-in blurb, and otherwise nothing appears.
  4. A version change that is not a major or minor change shows no card.

#### NT-6.4 Install Now button

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Install Now on either update card runs Attempt Update with the source notification and hides the card. Attempt Update checks, downloads, installs or restarts depending on the update state, and says "attemptUpdate: update already in progress" when it is busy.
- Commands: cursor.doupdate; update.checkForUpdate; update.downloadUpdate; update.installUpdate; update.restartToUpdate
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (executeCommand cursor.doupdate with notification, command cursor.doupdate titled Attempt Update, ids update.restartToUpdate and update.showCurrentReleaseNotes, module vs/workbench/contrib/update/browser/update.contribution)
- Pieces:
  1. Clicking Install Now on either update card runs Attempt Update (`cursor.doupdate`) with the source set to notification, and hides the card. [cursor only]
  2. Attempt Update checks for updates (`update.checkForUpdate`), downloads (`update.downloadUpdate`), installs (`update.installUpdate`) or restarts to update (`update.restartToUpdate`), whichever fits the current update state.
  3. When an update step is already running, Attempt Update says "attemptUpdate: update already in progress".
  4. Attempt Update is listed in the Command Palette and runs the same flow without a card.

#### NT-6.5 Snooze with Later or the close X

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Later on the compact card and the X on the update card hide the card and snooze it for the configured number of hours (clamped to 1 to 240), and the X on the changelog card hides that card for good.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (handler y in component Zyk, setSnooze and snoozedUntilTime, experiment config update_prompt_config key snooze_duration_hours, storage keys updatePromptSnoozedUntilTime and lastUpdateHiddenVersion)
- Pieces:
  1. Clicking Later on the compact card hides the card and snoozes it, so it does not come back until the snooze ends. [cursor only]
  2. Clicking the close X on the update card hides the card and snoozes it in the same way.
  3. The snooze length in hours comes from the experiment value `snooze_duration_hours`, is limited to between 1 and 240 hours, and falls back to 24 when the value is missing.
  4. Clicking the close X on the changelog card hides that card for good.

#### NT-6.6 Changelog link buttons

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The Changelog button on the update card and the View changelog button on the changelog card open https://changelog.cursor.com in the browser (nightly builds add ?nightly=true), and only the second also hides its card.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (components Zyk and Qyk, url changelog.cursor.com, the nightly check on productService.version)
- Pieces:
  1. Clicking Changelog on the update card opens https://changelog.cursor.com in the browser and leaves the card showing. [cursor only]
  2. Clicking View changelog on the changelog card opens the same page and hides the card.
  3. On nightly builds both buttons open the page with `?nightly=true` added.

#### NT-6.7 When a card shows, and the rate limits

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A check runs every 20 seconds and shows a card only when an update is ready to install, the window is a main window, and the prompt limits allow it (fallback config: at least 48 hours apart, 3 per version, 1 per day), while update.mode silentlyApplyOnQuit skips the card and installs on quit.
- Commands: cursor.checkonupdate
- Menus: none found
- Keys: none found
- Settings: update.mode; update.enableWindowsBackgroundUpdates; update.channel
- Source: C bundle (command cursor.checkonupdate with f1 false, class UpdateCheckCronContribution with task update-check-task at 20000 ms, class zgr updateNotificationService with shouldShowPrompt and recordPromptShown, fallbackValues of update_prompt_config, setting update.mode, telemetry cursor.update.promptShown)
- Pieces:
  1. Every 20 seconds Cursor checks the update state in the background and shows an update card only when an update is ready to install. [cursor only]
  2. Setting `update.enableWindowsBackgroundUpdates` turns Windows background download and install on or off, and with it on and a system-scope install the card shows once the update is downloaded instead of ready.
  3. Only a main window shows an update card.
  4. Cards are rate limited, and the fallback config allows at least 48 hours between cards, 3 cards per version and 1 card per day.
  5. The first time a new version runs, the snooze and prompt timing from the older version are cleared.
  6. When the update is no longer ready, a card that was showing goes away.
  7. Setting `update.mode` chooses how updates are fetched, with values `none`, `manual`, `start`, `default` and `silentlyApplyOnQuit`.
  8. With `update.mode` set to `silentlyApplyOnQuit`, no card shows and the update installs without a prompt.
  9. The hidden command `cursor.checkonupdate` runs one such check on demand and is not listed in the Command Palette.

#### NT-6.8 Card placement and look

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: The update and compact cards float 8 px from the window edge on the side bar side (left when the side bar is on the left, else right), 30 px above the bottom with the status bar visible or 16 px without, in a glass style 342 px wide, and the changelog card is 340 px wide.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Source: C bundle css (update-notification-outer-container with position fixed and z-index 2000, update-notification-outer-container-left, update-notification-outer-container-right, update-notification-container, minor-version-notification-container), components Zyk and Qyk
- Pieces:
  1. The update card and the compact card float 8 px in from the window edge, above other UI. [cursor only]
  2. The cards sit 30 px above the bottom edge when the status bar is visible and 16 px above it when the status bar is hidden.
  3. The update card and the compact card are 342 px wide with a glass look.
  4. The changelog card is 340 px wide.
  5. Setting `workbench.sideBar.location` chooses which edge the cards sit on, `left` or `right`, following the side bar.

#### NT-6.9 Classic "update available" prompts

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When an update is available for manual download Cursor shows a notification "There is an available update." with Download Update, Later and Release Notes, or on Linux package installs "A newer {0} package ({1}) is available via {2}." with Update, Later and Release Notes. Stock 1.128 no longer has them.
- Commands: update.downloadUpdate; update.showCurrentReleaseNotes
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/contrib/update/browser/update, methods onUpdateAvailable and onPackageUpdateAvailable, nls 11503 to 11510); V\src\vs\workbench\contrib\update\browser\update.ts (no such prompts in 1.128.0)
- Pieces:
  1. When an update is available for manual download, a notification "There is an available update." appears. [cursor only]
  2. That notification has the buttons Download Update (`update.downloadUpdate`), Later and Release Notes.
  3. On Linux package installs the notification says "A newer {0} package ({1}) is available via {2}." with the buttons Update, Later and Release Notes.
  4. Clicking Release Notes (`update.showCurrentReleaseNotes`) opens the release notes web page in the browser.
  5. Clicking Later dismisses the notification without starting a download.

#### NT-6.10 Update did not install prompt

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: After a restart that did not finish an update, a warning says "{0} couldn't finish installing the update to version {1}." with a Try Again button, or an info note says it is downloading the update again when a retry is already running.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/contrib/update/browser/update, method maybeShowLastInstallFailedNotification, nls 11492, 11493, 11494)
- Pieces:
  1. After a restart that did not finish an update, a warning says "{0} couldn't finish installing the update to version {1}." with a Try Again button. [cursor only]
  2. Clicking Try Again starts the update again.
  3. When a retry is already running, an info notice says the update is downloading again, and the warning is not shown.

#### NT-6.11 Updates disabled as Administrator notice

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: On Windows, running the user-scope install as Administrator shows "Updates are disabled because you are running the user-scope installation of {0} as Administrator." with Learn More (opens the downloads page) and a Don't Show Again choice.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/contrib/update/browser/update, disabled state with reason 5, neverShowAgain id no-updates-running-as-admin, nls 11495, 11496); V\src\vs\workbench\contrib\update\browser\updateTooltip.ts (stock shows this state only in the title bar tooltip)
- Pieces:
  1. On Windows, running the user-scope install as Administrator shows "Updates are disabled because you are running the user-scope installation of {0} as Administrator." [cursor only]
  2. The notice has a Learn More button that opens the downloads page in the browser.
  3. The notice has a Don't Show Again choice that stops the notice from coming back.

#### NT-6.12 Update error notice

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: When the updater ends in an error, Cursor shows an error notification from the source "Update Service" with the error text, except for request timeouts and lost network connections, and a Mac Squirrel error is replaced by a note to quit, move Cursor to Applications and relaunch.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/contrib/update/browser/update, method onError, nls 11501)
- Pieces:
  1. When the updater ends in an error, an error notification from the source "Update Service" shows the error text. [cursor only]
  2. Request timeouts and lost network connections show no error notification.
  3. A Squirrel error on macOS is replaced by a note to quit Cursor, move it to Applications and open it again.

#### NT-6.13 Post-update widget (stock)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: After a major or minor VS Code update a sticky card titled "New in {version}" appears near the top right of the window with a banner, a close button, an optional badge, a feature list or markdown, and buttons (by default a single Release Notes button), Escape closes it, and Show Update Info from the palette previews it.
- Commands: _update.showUpdateInfo; update.showUpdateInfo; update.showCurrentReleaseNotes
- Menus: CommandPalette
- Keys: none found
- Settings: update.showPostInstallInfo
- Source: V\src\vs\workbench\contrib\update\browser\postUpdateWidget.ts; V\src\vs\workbench\contrib\update\browser\update.contribution.ts; V\src\vs\platform\update\common\update.config.contribution.ts; V\src\vs\workbench\contrib\update\common\updateInfoParser.ts; V\src\vs\workbench\contrib\update\common\updateUtils.ts; V\src\vs\workbench\contrib\update\browser\media\postUpdateWidget.css; C bundle (searched _update.showUpdateInfo, update.showUpdateInfo, postUpdateWidget, update.showPostInstallInfo: no hits)
- Pieces:
  1. After a major or minor update, a sticky card titled "New in {version}" appears near the top right of the window. [stock only]
  2. The card shows a banner at the top, a gradient by default or an image from the update info, and a close button.
  3. The card can show a small badge above the title.
  4. The card body shows a feature list, or markdown text when there is no feature list, and links in the text open when clicked.
  5. The card has the buttons from the update info, and by default a single Release Notes button that runs `update.showCurrentReleaseNotes`.
  6. Pressing `Escape` closes the card, and keyboard focus stays inside it while it is open.
  7. Running "Show Update Info" (`update.showUpdateInfo`) from the Command Palette previews the card.
  8. Setting `update.showPostInstallInfo`, off by default, shows this card after an update instead of opening the release notes editor.
  9. When the update info cannot be fetched, no card appears.

#### NT-6.14 Release notes fallback prompt (stock)

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing
- What: After a major or minor update, if the release notes editor cannot open, VS Code shows an info prompt "Welcome to {0} v{1}! Would you like to read the Release Notes?" with a Release Notes button that opens the notes web page.
- Commands: update.showCurrentReleaseNotes
- Menus: none found
- Keys: none found
- Settings: update.showReleaseNotes; update.showPostInstallInfo
- Source: V\src\vs\workbench\contrib\update\browser\update.ts (class ProductContribution); V\src\vs\platform\update\common\update.config.contribution.ts; C bundle (label "Would you like to read the Release Notes" and setting update.showReleaseNotes not found in the label table or the bundle)
- Pieces:
  1. After a major or minor update, VS Code opens the release notes editor, and if it cannot, an info prompt asks "Welcome to {0} v{1}! Would you like to read the Release Notes?" [stock only]
  2. The prompt has a Release Notes button that opens the release notes web page in the browser. [stock only]
  3. Setting `update.showReleaseNotes`, on by default, turns the release notes after an update on or off, and off shows neither the editor nor the prompt. [stock only]
  4. With `update.showPostInstallInfo` on, neither the release notes editor nor this prompt shows after an update, because the post-update card takes over. [stock only]

#### NT-6.15 Memory warning prompt

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: A sticky warning notification says "Cursor is using a lot of memory ({N} MB). We can capture a snapshot for debugging, and attempt to free up some memory in the process." with a Capture Snapshot button, shown once per session when the JS heap passes 80 percent of the V8 limit (or 50 percent for live heap).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution, method _checkMemoryUsage with prompt priority Urgent and sticky, telemetry renderer.memoryMonitor.userToast, experiment config memory_monitor_config with fallback show_user_toast true, user_toast_heap_percent 80, user_toast_live_heap_percent 50, check interval 10 seconds)
- Pieces:
  1. A sticky warning notification says "Cursor is using a lot of memory ({N} MB). We can capture a snapshot for debugging, and attempt to free up some memory in the process." with a Capture Snapshot button. [cursor only]
  2. The warning shows when the JS heap passes 80 percent of the V8 limit, or 50 percent for the live heap.
  3. The warning shows at most once per session.
  4. The warning is urgent, so it shows even when notifications are filtered to errors only.
  5. Memory is checked every 10 seconds, and the warning only shows while the experiment flag `show_user_toast` is true, which it is in the fallback config.
  6. Clicking Capture Snapshot starts the snapshot capture described in NT-6.16.

#### NT-6.16 Capture Snapshot flow

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only
- What: Capture Snapshot runs a non-cancellable progress notification "Capturing snapshot. The window may briefly freeze.", then a "Snapshot saved." confirm with Send to Cursor or No thanks (or OK in privacy mode), and ends with an upload progress notification and result, or a prompt "Heap snapshot saved to disk." with Reveal in File Explorer, and the Developer palette command runs the same capture and upload directly.
- Commands: developer.captureAndUploadRendererHeapSnapshot; memoryMonitor.triggerManualSnapshot
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: C bundle (module vs/workbench/contrib/memoryMonitor/electron-sandbox/memoryMonitor.contribution, methods _captureUserSnapshot, _captureSnapshotToDisk, _uploadSavedSnapshot, _showRevealNotification, command developer.captureAndUploadRendererHeapSnapshot, nls 7770)
- Pieces:
  1. Capture Snapshot shows a progress notification "Capturing snapshot. The window may briefly freeze." with no Cancel button. [cursor only]
  2. When the capture ends, a confirm box "Snapshot saved." offers Send to Cursor and No thanks.
  3. In privacy mode the confirm box has only an OK button.
  4. Clicking Send to Cursor shows a progress notification "Uploading snapshot to Cursor..." that cannot be cancelled, followed by a notification with the upload result.
  5. When the snapshot stays on disk instead of being uploaded, a prompt says "Heap snapshot saved to disk." with a Reveal in File Explorer button that opens the file's folder.
  6. The palette command "Developer: Capture and Upload Renderer Heap Snapshot" (`developer.captureAndUploadRendererHeapSnapshot`) runs the same capture and upload directly.
