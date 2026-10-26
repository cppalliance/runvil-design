Group: NT-2 Notification contents and buttons. Pass: 3. Date: 2026-10-25.

### NT-2.1
1. Each notification row starts with an icon that matches its severity: info, warning, or error.
2. The severity icon is colored by the theme's notification info, warning, and error icon colors.
3. In Cursor, an Info notification shows the info-two glyph and a Warning notification shows the warning-two glyph, while Error keeps the same error glyph. [cursor only]

### NT-2.2
1. A notification with Success severity shows a check (pass) icon in its row. [cursor only]
2. Sending feedback shows a Success notification reading Thanks for your feedback! [cursor only]
3. Applying changes from a worktree shows a Success notification that confirms the changes were applied. [cursor only]
4. Sending a bug report shows a Success notification saying the report was sent. [cursor only]

### NT-2.3
1. A notification raised with Cursor's AI severity draws the default info icon. [cursor only]
2. An AI severity toast has no entry in Cursor's auto-hide table, so its hide time is not defined, and no feature that raises one was found. [cursor only]

### NT-2.4
1. A notification message can contain a text link, and clicking a link to a web page opens that page.
2. Clicking a command link in a message runs that command.
3. Hovering a command link shows the text Click to execute command and the command id.
4. A link can be used with a tap, and with Space or Enter when the link has keyboard focus.

### NT-2.5
1. A message longer than 1000 characters is cut at 1000 characters and ends with three dots.
2. Line breaks in a message show as spaces.
3. A collapsed row shows the message on one line and ends it with an ellipsis when it does not fit.
4. Hovering a collapsed message that does not fit shows the full text.
5. An expanded row wraps the whole message across as many lines as it needs.
6. In stock, message text uses a 22 px line in a 42 px collapsed row. [stock only]
7. In Cursor, message text is 12 px on a 16 px line in a 28 px collapsed row. [cursor only]

### NT-2.6
1. An expanded notification that has a source shows a line reading Source followed by the name of the extension or area that raised it.
2. Hovering the source line shows the source name again.
3. A collapsed notification, or one with no source, shows no source line.
4. In Cursor, the source text is 11 px at 70 percent opacity with a minimum width of 10 characters. [cursor only]

### NT-2.7
1. Clicking the chevron on a notification expands a collapsed one and collapses an expanded one, with hover text Expand Notification or Collapse Notification.
2. With a notification focused, Right expands it (`notification.expand`) and Left collapses it (`notification.collapse`).
3. With a notification focused, Space or Enter toggles it between open and closed (`notification.toggle`).
4. Double clicking a notification toggles it between open and closed.
5. An expanded notification shows the full message, the source, and the buttons, and a collapsed one shows a single line.
6. A collapsed notification shows the chevron only when it has a source or its message does not fit on one line.
7. A notification that has buttons is always open and cannot be collapsed.

### NT-2.8
1. The gear, chevron, and close icons are hidden until you hover the row or focus the list.
2. An expanded row always shows its icons.
3. Each icon's hover text includes its keyboard shortcut when it has one.
4. In Cursor, the close icon is dimmed to 50 percent opacity and the icon strip is 16 px high. [cursor only]

### NT-2.9
1. Choices passed with a notification show as buttons in the details row under the message of an expanded notification.
2. In stock, the first choice shows first in the row as a primary button and the rest follow as secondary buttons. [stock only]
3. Clicking a button runs its choice and closes the notification.
4. A choice that asked to stay open leaves the notification open after its button is clicked.
5. A button label too long to fit shows its full text as a hover title.
6. In Cursor, buttons are added in reverse order, so the primary button ends up last in the row, and this was not checked on screen. [cursor only]
7. In Cursor, button text is 12 px with a 3 px corner radius. [cursor only]

### NT-2.10
1. A primary choice that has a menu of extra choices shows its button with a dropdown arrow.
2. Opening the dropdown lists the extra choices, and picking one runs it.
3. A secondary choice cannot have a menu, so it never shows a dropdown arrow.

### NT-2.11
1. A notification that asks for Don't Show Again shows it as the first button.
2. A notification that asks for Don't Show Again as a secondary choice shows it as the last entry in the gear menu.
3. Choosing Don't Show Again closes the notification.
4. The answer is remembered for the workspace, the profile, or the whole application, as the notification asked, and that notification never appears again.

### NT-2.12
1. A notification that has secondary choices shows a gear icon with hover text More Actions...
2. Clicking the gear opens a drop-down menu that lists the secondary choices.
3. Picking an entry from the gear menu runs that choice.
4. A notification with no secondary choices shows no gear icon.

### NT-2.13
1. On a notification with a named source and secondary choices, the gear menu starts with Turn Off Info and Warning Notifications from that source.
2. When that source is already muted, the first gear menu entry reads Turn On All Notifications from that source instead.
3. Choosing the mute entry flips that source's mute state, so the next gear menu opened for the source shows the other wording.
4. A separator sits between the mute entry and the notification's own secondary choices.

### NT-2.14
1. Clicking the X button on a notification closes it.
2. Pressing Delete on a focused notification closes it (`notification.clear`), and Cmd+Backspace does the same on macOS.
3. While a notification has running progress, the X button is hidden and Delete does nothing.
4. Clearing a notification plays a signal, a sound and an announcement, controlled by `accessibility.signals.clear`.

### NT-2.15
1. Clicking a notification with the middle mouse button closes it.
2. A middle click on a notification does not paste into the editor behind it.
3. A middle click does nothing while the notification has running progress.

### NT-2.16
1. Right-clicking a notification shows a menu with one item, Copy Text.
2. Choosing Copy Text copies the full original message to the clipboard (`workbench.action.copyNotificationMessage`).
3. The right-click menu works on notifications in toasts and in the notification center.

### NT-2.17
1. Running Accept Notification Primary Action (`notification.acceptPrimaryAction`) from the Command Palette runs the first button of the focused notification, or the newest notification when none is focused, and closes it.
2. Pressing Ctrl+Shift+A (Cmd+Shift+A on macOS) does the same as the command.
3. In stock, the key works only when screen reader mode is on and a notification has focus or a toast is visible. [stock only]
4. In Cursor, the key works whenever a notification has focus or a toast is visible, with no screen reader condition. [cursor only]
5. When the notification has no buttons, the command does nothing.

## Unclear

- NT-2.3 has only two pieces and the second is not something a person can see today. The feature may be a candidate to drop from the catalog.
- NT-2.2 pieces 3 and 4 use plain descriptions of the confirmation, because only the feedback message wording was found in the bundle.
- NT-2.9 piece 6 and NT-2.17 piece 4 were read from code only and were not checked on screen.
- NT-2.12 piece 4 is taken from the render code, where the gear is added only when secondary choices exist.
- NT-2.13 owns the gear menu entry and its wording. What the mute changes in notifications, the header dropdown, and the picker is given to NT-4.4, so the two features do not repeat each other.
