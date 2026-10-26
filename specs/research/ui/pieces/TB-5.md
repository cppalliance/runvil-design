Group: TB-5 Title bar buttons and update control. Pass: 3. Date: 2026-10-25.

### TB-5.1
1. When an update is waiting, a text button labelled Update shows in the title bar, next to the search box in stock and in the left toolbar in Cursor.
2. In stock the Update button shows for the available, downloaded and ready update states, and the plain Update label is highlighted. [stock only]
3. In stock the Update button is hidden in editor Zen mode, in debug mode and while a chat request is running. [stock only]
4. In stock the Update button is hidden when the window is too narrow to fit it. [stock only]
5. In Cursor the Update button shows only when the update state is `ready`, the server experiment `ide_update_ux_exp` has the variant `non_sidebar`, and the window is not the Agents window. [cursor only]

### TB-5.2
1. In stock, clicking Update when an update is available starts the download (`update.downloadNow`), and clicking it when the update is downloaded starts the install (`update.install`). [stock only]
2. In stock, clicking Update when the update is ready restarts the app to apply it (`update.restart`). [stock only]
3. In stock, clicking Update when no action is possible opens the update card instead. [stock only]
4. In Cursor, clicking Update restarts Cursor to install the update (`update.restartToUpdate`), as its only action. [cursor only]
5. In Cursor, a moment after the click (a 300 ms timer) the label turns to Installing with a spinner and the button is marked busy for screen readers. [cursor only]
6. In Cursor, the label goes back to Update when the restart is skipped, the shutdown is vetoed or the command fails. [cursor only]

### TB-5.3
1. While the app checks for updates, the button reads Checking... with an indefinite progress look. [stock only]
2. While an update downloads, the button reads Downloading... and fills left to right by the percent downloaded. [stock only]
3. While an update installs, the button reads Installing... and fills by the percent installed. [stock only]
4. The button reads Updating... while an update is being overwritten, Restarting... while the app restarts to update, and Cancelling... while an update is being cancelled, each with an indefinite progress look. [stock only]
5. Download, install and overwrite progress show on the button only when the person started the update. [stock only]
6. The plain Update label for the available, downloaded and ready states is highlighted. [stock only]

### TB-5.4
1. Hovering the Update button opens a card with a pointer that stays open until it is dismissed. [stock only]
2. The card title follows the update state: Update Available, Downloading Update, Update is Ready to Install, Restart to Update, Up to Date, Updates Disabled or Update Error. [stock only]
3. While an update downloads, the card shows the download speed and the time left. [stock only]
4. The card shows Current Version and Latest Version lines with a Copy control for the version, and a Released date line. [stock only]
5. The card has a Release Notes button (`update.showCurrentReleaseNotes`) and, depending on the state, a Download (`update.downloadNow`), Install (`update.install`) or Restart (`update.restart`) button. [stock only]
6. The card opens by itself on an update error, and at most once every 30 days when updates are disabled by an invalid config or by running as administrator. [stock only]
7. Setting `update.mode` changes the message on the Up to Date card (automatic updates disabled, checked but not installed, applied on restart, or enabled), and the Updates Disabled card tells the person to change `update.mode` when updates were turned off by hand. [stock only]
8. In Cursor, hovering the Update button shows a small hover below it with the line "Restart Cursor to install the pending update", with the version when it is known, and a different line while installing. [cursor only]

### TB-5.5
1. Setting `update.titleBar` on (default) shows the update indicator in the title bar when an update is waiting, and off hides it. [stock only]
2. With `update.titleBar` off, the update card also never opens by itself on an update error. [stock only]
3. `update.titleBar` is offered on desktop only, and Cursor has no such setting. [stock only]

### TB-5.6
1. A gear button in the right toolbar of the title bar, always shown in the IDE window, opens Cursor Settings when clicked (`workbench.action.openAgentLayoutQuickMenu` runs `aiSettings.action.open`). [cursor only]
2. Hovering the gear button shows the text Open Cursor Settings. [cursor only]
3. Ctrl+Shift+J (Cmd+Shift+J on macOS) opens Cursor Settings. [cursor only]
4. Ctrl+, (Cmd+, on macOS) also opens Cursor Settings, but only when the window is not the Agents window. [cursor only]

### TB-5.7
1. A pill button labelled Upgrade shows at the left end of the right toolbar. [cursor only]
2. The Upgrade button shows only for accounts on the Free plan when no membership refresh is pending. [cursor only]
3. Hovering the Upgrade button shows "Upgrade for extended limits and full model access". [cursor only]
4. Clicking the Upgrade button starts the checkout for the next plan, either a Stripe Pro checkout or a unified billing flow (`cursor.titlebar.upgradeToPro`). [cursor only]
5. Further clicks on the Upgrade button are ignored until the checkout flow finishes. [cursor only]

### TB-5.8
1. With `workbench.notifications.position` set to `top-right` (an experimental setting with values `bottom-right`, `bottom-left` and `top-right`), a bell button shows at the far right of the title bar action toolbar (group `navigation`, order 10000). [stock only]
2. Clicking the bell opens the notification center and clicking it again closes it (`notifications.toggleList`). [stock only]
3. Setting `workbench.notifications.showInTitleBar` (default on) hides the bell when it is off, and it applies only while notifications are at the top right. [stock only]
4. Right-clicking the title bar shows a Notifications row (`toggle.workbench.notifications.showInTitleBar`, group `2_config`, order 5) that flips the bell on and off, and the row shows only while notifications are at the top right. [stock only]
5. With the notification position set to top right, the bell is removed from the status bar. [stock only]
6. With the position at top right, a bell button with hover text Toggle Notifications appears at the right end of the title bar. [stock only]
7. Setting `workbench.notifications.showInTitleBar` shows or hides the title bar bell. [stock only]
8. Right-clicking the title bar shows a Notifications toggle row that shows or hides the title bar bell (`toggle.workbench.notifications.showInTitleBar`). [stock only]

### TB-5.9
1. A globe button in the title bar (group `navigation`, order 10) opens a new integrated browser tab when none is open, and opens the list of open browser tabs when one is already open (`workbench.action.browser.openOrList`, which calls `workbench.action.browser.open` or `workbench.action.browser.quickOpen`). [stock only]
2. Setting `workbench.browser.showInTitleBar` chooses when the globe button shows, with `true` always, `false` never, and `whenOpen` (default) only while a browser editor is open. [stock only]
3. Right-clicking the title bar shows an Integrated Browser row (`toggle.workbench.browser.showInTitleBar`, group `2_config`, order 8) that is checked while the setting is `true` and flips the setting when clicked. [stock only]
4. Ctrl+Alt+/ (Cmd+Alt+/ on macOS) opens a new browser tab or the list of open browser tabs, the same as the globe button. [stock only]

### TB-5.10
1. When the title bar action toolbar holds more buttons than its limit, the extra buttons collapse into a "..." menu.
2. In stock the toolbar shows up to 12 buttons before it collapses the rest. [stock only]
3. In Cursor the toolbar shows up to 10 buttons before it collapses the rest. [cursor only]
4. In Cursor the "..." menu sits at the end of the toolbar. [cursor only]
5. Accounts, Manage and the editor navigation commands are never collapsed into the "..." menu.

### TB-5.11
1. Hovering a button in the title bar action toolbar shows its name.
2. The hover also shows the key binding of the button when it has one.
3. The title bar action toolbar has the accessible label Title actions.

## Unclear
- TB-5.1 piece 5 depends on a server experiment, so who sees the Cursor Update button was not decoded. What the `control` variant shows, if anything, was not found.
- TB-5.2 pieces 1 and 2 use the stock command ids. Update rows with the same ids in the Manage menu belong to AB-4.
- TB-5.3 does not list the Manage menu rows (`update.checking`, `update.downloading`, `update.updating`, `update.cancelling`), which belong to AB-4.
- TB-5.4 and the Help menu or Manage menu release notes row may overlap with TB-1 and AB-4. Dedupe when merging.
- TB-5.8 may overlap with NT-3, whose description also names the title bar bell. Dedupe when merging.
- TB-5.9 is not marked boundary, as the feature block says, because Cursor has no title bar browser button.
- TB-5.11 has no Cursor piece for the left toolbar buttons, because whether they show key bindings on hover was not checked.
- TB-5.7 piece 4 does not name the plan targets or checkout screens, because the feature block says they were not decoded.
