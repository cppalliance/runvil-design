Group: NT-4 Do Not Disturb. Pass: 3. Date: 2026-10-25.

### NT-4.1
1. Running Toggle Do Not Disturb Mode (`notifications.toggleDoNotDisturbMode`) from the Command Palette turns Do Not Disturb on when it is off.
2. Running the same command again turns Do Not Disturb off.
3. The command flips one global switch that applies to every source at once, separate from the per-source choices.

### NT-4.2
1. A bell-slash button in the notification center header, with hover text Configure Do Not Disturb..., opens a menu (`workbench.action.configureDoNotDisturbMode`).
2. The first menu entry reads Enable Do Not Disturb Mode when the mode is off and Disable Do Not Disturb Mode when it is on, and choosing it flips the global mode.
3. Below a separator the menu lists up to 10 sources sorted by label, each as a checkable entry, where checked means that source may still show everything.
4. Choosing a source entry flips that source between showing everything and silencing info and warning notifications (`notifications.toggleDoNotDisturbModeBySource`).
5. When more than 10 sources are known, a separator and a More… entry follow the list, and More… opens the source picker.
6. When no source has shown a notification yet, the menu holds only the enable or disable entry.

### NT-4.3
1. Running Toggle Do Not Disturb Mode By Source... from the Command Palette opens a multi-select quick pick with the placeholder Select sources to enable all notifications from.
2. The picker lists each source by its label, sorted by label, and hovering a row shows the label with the source id in brackets.
3. Sources that currently show everything start ticked, and sources that are currently silent for info and warning start unticked.
4. Accepting the picker applies the ticks to every source at once and closes the picker, so ticked sources keep showing everything and unticked sources become silent for info and warning.
5. Closing the picker without accepting changes nothing.
6. Only sources that have already shown a notification in this install are listed.

### NT-4.4
1. After choosing Turn Off Info and Warning Notifications from a source in a notification's gear menu, later info and warning notifications from that source are silent while its errors still show.
2. After that choice, the source's entry in the center header dropdown loses its check mark and its row in the source picker is unticked.
3. Choosing Turn On All Notifications from the source brings back the check mark and the tick, and its notifications show again.
4. The mute entry changes only that one source and leaves the global mode and every other source as they were.

### NT-4.5
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

### NT-4.6
1. Switching the global mode on hides every toast that is showing.
2. Switching the global mode on closes the notification center if it is open.
3. Switching one source off removes that source's visible info and warning toasts.
4. Error toasts and urgent toasts from a muted source stay on screen.

### NT-4.7
1. With Do Not Disturb on, a notification with Error severity still shows as a toast.
2. With Do Not Disturb on, a notification with Urgent priority still shows as a toast.
3. With Do Not Disturb on, notifications with default or optional priority and a severity below Error become silent.
4. Stock prompts that set Urgent priority, such as window, integrity, extension host, and save participant prompts, still show with Do Not Disturb on.
5. In Cursor, a Success or AI severity notification is silenced like Info and Warning, because only Error is exempt. [cursor only]
6. In Cursor, the memory warning prompt is Urgent, so it still shows with Do Not Disturb on. [cursor only]

### NT-4.8
1. A notification silenced by Do Not Disturb gets no toast.
2. A silenced notification gets no screen reader announcement.
3. A silenced notification is still added to the notification center list, which opens with Ctrl+K Ctrl+Shift+N (`notifications.showList`).
4. A silenced notification still raises the unread dot on the bell, and Clear All Notifications (`notifications.clearAll`) in the center removes it like any other.

### NT-4.9
1. With Do Not Disturb on globally, a progress notification asked for in the notification location makes no toast.
2. With Do Not Disturb on for the progress source, the progress notification makes no toast either.
3. The silent progress shows only as a spinner entry on the left of the status bar until it ends.
4. Clicking the spinner entry opens the notification center with the progress notification in it (`notifications.showList`).
5. A progress notification with Urgent priority still shows as a toast with Do Not Disturb on.

### NT-4.10
1. Entering Zen Mode turns global Do Not Disturb on when `zenMode.silentNotifications` is true, which is the default.
2. Leaving Zen Mode turns Do Not Disturb off again when it was off before Zen Mode began.
3. When Do Not Disturb was already on before Zen Mode began, leaving Zen Mode leaves it on.
4. Changing `zenMode.silentNotifications` while in Zen Mode turns Do Not Disturb on or off at once.
5. With `zenMode.silentNotifications` false, entering Zen Mode leaves Do Not Disturb as it was.

### NT-4.11
1. The global Do Not Disturb switch is still on or off after a restart, as it was before.
2. Every per-source choice is still the same after a restart.
3. The choices are saved in application storage under `notifications.doNotDisturbMode` and `notifications.perSourceDoNotDisturbMode`, so they do not appear in Settings.
4. A source appears in the header dropdown and the picker only after its first notification, and stays listed afterward.
5. In Cursor, the service also has a reset-global-filter method and a do-not-persist option, but nothing in the main window calls them, so no change is visible. [cursor only]

### NT-4.12
1. With Do Not Disturb on, Cursor's update card still appears. [cursor only]
2. With Do Not Disturb on, Cursor's changelog card still appears. [cursor only]
3. Turning Do Not Disturb on does not hide an update or changelog card that is already on screen. [cursor only]

## Unclear

- NT-4.4 pieces cover what the gear menu mute entry changes. The entry and its wording are pieces of NT-2.13, so the two features do not repeat each other.
- NT-4.8 piece 4 adds Clear All Notifications from the feature's command list, and the Clear All behavior comes from NT-3.9. It was not tried on screen.
- NT-4.9 piece 4 comes from the progress service, where the status bar spinner entry runs `notifications.showList` when clicked.
- NT-4.5 piece 2 uses the same count rule as NT-3.2, and NT-4.5 piece 3 replaces the count tooltip of NT-3.3 while Do Not Disturb is on.
- Cursor pieces come from bundle strings and were not seen on screen.
