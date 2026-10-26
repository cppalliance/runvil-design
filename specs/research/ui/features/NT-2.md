Group: NT-2 Notification contents and buttons. Section: Notifications. Date: 2026-10-25. Pass: 2.
Features: 17.

### NT-2.1 Severity icon
- Tag: both
- Evidence: source
- Boundary: no
- What: Each notification starts with an icon for its severity (info, warning, error), colored by the theme's notification icon colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - Info uses the info-two glyph and Warning uses warning-two, Error is the same error glyph
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsToasts.css; V\src\vs\workbench\browser\parts\notifications\media\notificationsCenter.css; V\src\vs\workbench\common\theme.ts; C bundle (module vs/workbench/browser/parts/notifications/notificationsViewer, fragment toSeverityIcon, css .notification-list-item-icon.codicon.codicon-info-two and codicon-warning-two)

### NT-2.2 Success notifications
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor adds a Success severity that shows a check (pass) icon and a 15 s toast timer, and callers use it for confirmations such as the feedback thanks, applied worktree changes, and the bug report sent message.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (module vs/workbench/browser/parts/notifications/notificationsViewer, fragments case Severity.Success returns the pass icon, Severity enum Success=5, notify calls with severity Success and message Thanks for your feedback!)

### NT-2.3 AI severity
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's severity list also has an AI value (4); it would draw the default info icon, and no caller, icon, or style for it was found, so no one is known to see it today.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (Severity enum fragment e[e.AI=4]="AI" in the severity module, toSeverityIcon default branch, PURGE_TIMEOUT table without an AI entry)

### NT-2.4 Links in the message
- Tag: both
- Evidence: source
- Boundary: no
- What: A message can contain text links that open a web page or run a command, and they work with a click, a tap, or Space and Enter when a link has focus; command links show the hover text Click to execute command and the command id.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\base\common\linkedText.ts; C bundle (nls module vs/workbench/browser/parts/notifications/notificationsViewer, label Click to execute command '{0}')

### NT-2.5 Long messages
- Tag: both
- Evidence: source
- Boundary: no
- What: Messages are cut at 1000 characters with three dots, line breaks become spaces, a collapsed row shows one line with an ellipsis and a hover with the full text, and an expanded row wraps the whole message.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - same 1000 character cut, but text is 12 px on a 16 px line in a 28 px row (stock 22 px line, 42 px row)
- Source: V\src\vs\workbench\common\notifications.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsList.css; C bundle (fragments MAX_MESSAGE_LENGTH=1e3, ROW_HEIGHT=28, LINE_HEIGHT=16, css .notification-list-item-message)

### NT-2.6 Source line
- Tag: both
- Evidence: source
- Boundary: no
- What: An expanded notification that has a source shows a line reading Source and the name of the extension or area that raised it, with the name again on hover.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - source text is 11 px at 70 percent opacity with a minimum width of 10 characters
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\media\notificationsList.css; C bundle (label Source: {0} in module vs/workbench/browser/parts/notifications/notificationsViewer, css .notification-list-item-source)

### NT-2.7 Expand and collapse
- Tag: both
- Evidence: source
- Boundary: no
- What: A notification can be opened to show the full message, the source, and the buttons, or closed to one line, with the chevron button, Right and Left, Space or Enter, or a double click, except that a notification with buttons is always open and cannot be collapsed.
- Commands: notification.expand; notification.collapse; notification.toggle
- Menus: none found
- Keys: Right (when notificationFocus); Left (when notificationFocus); Space or Enter (when notificationFocus)
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\common\notifications.ts; C bundle (command ids, key rules, labels Expand Notification and Collapse Notification, fragment onMouseDblClick toggle)

### NT-2.8 Row icons show on hover or focus
- Tag: both
- Evidence: source
- Boundary: no
- What: The gear, chevron, and close icons stay hidden until you hover the row, focus the list, or expand the row, and each icon's hover text includes its key where one exists.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - close icon is dimmed to 50 percent opacity and the icon strip is 16 px high
- Source: V\src\vs\workbench\browser\parts\notifications\media\notificationsList.css; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; C bundle (css .notification-list-item-toolbar-container rules)

### NT-2.9 Buttons under the message
- Tag: both
- Evidence: source
- Boundary: no
- What: Choices passed with a notification show as buttons in the details row under the message, the first in primary style and the rest secondary, and clicking one runs it and closes the notification unless the choice asked to stay open.
- Commands: workbench.dialog.choice.<n> (action id pattern, not a registered command)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed - buttons are added in reverse order, so the first (primary) button ends up last in the row; text is 12 px with a 3 px radius; not seen on screen
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; V\src\vs\workbench\common\notifications.ts; C bundle (fragment renderButtons with a loop that counts down from the last choice and secondary:!(s===0), action id pattern workbench.dialog.choice., css .notification-list-item-buttons-container .monaco-button)

### NT-2.10 Button with a dropdown
- Tag: both
- Evidence: source
- Boundary: no
- What: A primary choice can carry a menu of extra choices, in which case its button shows a dropdown arrow, and secondary choices cannot have one.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\platform\notification\common\notification.ts; C bundle (fragment addButtonWithDropdown in renderButtons)

### NT-2.11 Don't Show Again
- Tag: both
- Evidence: source
- Boundary: no
- What: A caller can ask for a Don't Show Again choice, which appears as the first button or as the last gear-menu entry, closes the notification, and remembers the answer for the workspace, the profile, or the whole application.
- Commands: workbench.notification.neverShowAgain (action id, not a registered command)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\notification\common\notificationService.ts; V\src\vs\platform\notification\common\notification.ts; C bundle (label Don't Show Again in module vs/workbench/services/notification/common/notificationService, action id workbench.notification.neverShowAgain)

### NT-2.12 Gear menu with extra actions
- Tag: both
- Evidence: source
- Boundary: no
- What: When a notification has secondary choices, a gear icon (More Actions...) opens a drop-down menu that lists them.
- Commands: workbench.action.configureNotification
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (command id, labels More Actions... and Notification Actions in modules notificationsActions and notificationsViewer)

### NT-2.13 Mute a source from the gear menu
- Tag: both
- Evidence: source
- Boundary: no
- What: On a notification that has both a named source and secondary actions, the gear menu starts with Turn Off Info and Warning Notifications from the source (or Turn On All Notifications from it when already muted), which silences that source's non-error notifications.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\services\notification\common\notificationService.ts; C bundle (labels Turn On All Notifications from '{0}' and Turn Off Info and Warning Notifications from '{0}' in module notificationsViewer, storage key notifications.perSourceDoNotDisturbMode)

### NT-2.14 Close a notification
- Tag: both
- Evidence: source
- Boundary: no
- What: The X button, or Delete (Cmd+Backspace on macOS) on a focused notification, closes it, and while progress is running the button is hidden and the key does nothing.
- Commands: notification.clear
- Menus: none found
- Keys: Delete (when notificationFocus); Cmd+Backspace on macOS (when notificationFocus)
- Settings: accessibility.signals.clear
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; V\src\vs\workbench\contrib\accessibility\browser\accessibilityConfiguration.ts; C bundle (command id notification.clear, key rule with Delete and mac Backspace, label Clear Notification)

### NT-2.15 Close with a middle click
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking a notification with the middle mouse button closes it (unless it has running progress) and does not paste into the editor behind it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsViewer.ts; C bundle (fragment AUXCLICK handler with button===1 and the hasProgress check)

### NT-2.16 Copy a notification's text
- Tag: both
- Evidence: source
- Boundary: no
- What: Right-clicking a notification shows one menu item, Copy Text, which copies the full original message to the clipboard.
- Commands: workbench.action.copyNotificationMessage
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsList.ts; V\src\vs\workbench\browser\parts\notifications\notificationsActions.ts; C bundle (command id, label Copy Text in module vs/workbench/browser/parts/notifications/notificationsActions)

### NT-2.17 Accept the first button from the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: The Accept Notification Primary Action command runs the first button of the focused notification (or the newest one) and closes it, and in stock the Ctrl+Shift+A key for it only works with a screen reader on.
- Commands: notification.acceptPrimaryAction
- Menus: CommandPalette
- Keys: Ctrl+Shift+A (Cmd+Shift+A on macOS), when screen reader mode is on and (notificationFocus or notificationToastsVisible)
- Settings: none found
- Cursor: changed - the key rule has no screen reader condition, so it is live whenever a notification has focus or a toast is visible
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; C bundle (command id, label Accept Notification Primary Action, key rule fragment with when:ge.or(Ime,OOe))

## Handed off

- Switching Do Not Disturb on or off, globally or by source, and the quick pick for choosing sources: NT-4. The gear-menu mute entry is listed here as NT-2.13 because the approved description names it, and NT-4 should reference it instead of listing it again.
- The progress bar drawn inside a notification and the Cancel button of a progress notification: NT-5.

## Unclear

- Action ids such as workbench.dialog.choice.<n> and workbench.notification.neverShowAgain are runtime action ids made in the notification service, not commands registered with the command registry. They are written in Commands only so a later tool can find them.
- The Cursor button order (NT-2.9) comes from the render loop and the css that has no row-reverse rule. It was not checked on screen.
- The Success icon has no color rule in the Cursor css that I could find (only Info, Warning, and Error icon colors). The AI severity has no caller. Cursor also plumbs toastTitle, toastDescription, toastCode, toastDurationMs, and closeNotificationOnToastDismiss on notifications, and no reader was found in the main bundle, so none is listed as a feature.
- The Cursor key rule for Ctrl+Shift+A (NT-2.17) differs from the stock source of the same 1.128.0 base. It may be an older copy of the file, and the reason is unknown.
- Newer than 1.128: in VS Code main the notification row height can be set (compact 34 px rows) and button state follows the action. Neither exists in 1.128.0, so neither is listed.
