Group: TB-5 Title bar buttons and update control. Section: Title bar. Date: 2026-10-25. Pass: 2.
Features: 11.

### TB-5.1 Update button in the title bar
- Tag: both
- Evidence: source
- Boundary: no
- What: A text button labelled Update shows next to the search box (stock) or in the left toolbar (Cursor) when an update is waiting. Stock shows it for available, downloaded and ready updates, hides it in editor Zen mode, in debug mode and while a chat request runs, and hides it when the window is too narrow to fit it. Cursor shows it only when the update service state is `ready`, the server experiment `ide_update_ux_exp` has the variant `non_sidebar`, and the window is not the Agents window.
- Commands: workbench.actions.updateIndicator; cursor.titlebar.installUpdate
- Menus: TitleBarAdjacentCenter (order 0, stock only; Cursor builds the button in code at left toolbar order 130)
- Keys: none found
- Settings: update.titleBar
- Cursor: changed (different command id, own button component IdeTitlebarUpdateButton, shown only when the update is ready and the experiment variant matches)
- Source: V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:57-75,150-217; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:495-512,918-936; C bundle (module IdeTitlebarUpdateButton strings 11480-11484; titlebarPart string 4034; computeToolbarState fields ideUpdateAvailable and isGlassWindow; toolbar list S6w entry BFr order 130; component c6w)

### TB-5.2 Click Update to download, install or restart
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the button acts on the update state. Stock runs download, install or restart depending on state, and when nothing is actionable it opens the tooltip instead. Cursor only restarts to install: the label turns to Installing with a spinner after a 300 ms arm timer, and it goes back to Update if the restart is skipped, the shutdown is vetoed or the command fails.
- Commands: update.downloadNow; update.install; update.restart; update.restartToUpdate
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (one action, restart-to-update, with an installing state and spinner)
- Source: V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:282-300; V\src\vs\workbench\contrib\update\browser\update.ts:84-160; C bundle (component c6w click handler runs `update.restartToUpdate` with source `titlebar`; attributes data-installing and aria-busy; CSS spinner classes `ide-titlebar-update-button-spinner`)

### TB-5.3 Update button shows check, download and install progress
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Stock changes the button label and fill as the update moves: Checking..., Downloading... with a percent fill, Installing... with a percent fill, Updating..., Restarting... and Cancelling.... Download, install and overwrite progress only show when the user started the update. The plain Update state is highlighted. Cursor shows none of these states, only Update and its own Installing spinner (TB-5.2).
- Commands: update.checking; update.downloading; update.updating; update.cancelling (the disabled state rows that mirror these labels in the Manage menu)
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing
- Source: V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:303-361; V\src\vs\workbench\contrib\update\browser\update.ts:84-150; C bundle (searched `updateIndicator`, `workbench.actions.updateIndicator` and the labels Checking... and Downloading... in the title bar part, no hits; its title bar button only reads the `ready` state)

### TB-5.4 Update tooltip card
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering the button opens a card. Stock shows a sticky card with a title for each state (Update Available, Downloading Update with speed and time left, Update is Ready to Install, Restart to Update, Up to Date, Updates Disabled, Update Error), the current and latest version with a Copy control, the release date, a Release Notes button and Download, Install or Restart buttons. It opens by itself on an update error and, at most once every 30 days, when updates are disabled by a bad config or by running as administrator. Cursor shows a small hover below the button with one line: "Restart Cursor to install the pending update", with the version when known, and a different line while installing.
- Commands: update.downloadNow; update.install; update.restart; update.showCurrentReleaseNotes
- Menus: none found
- Keys: none found
- Settings: update.mode
- Cursor: changed (plain hover text instead of the stock card; no updateTooltip strings in the bundle)
- Source: V\src\vs\workbench\contrib\update\browser\updateTooltip.ts:120-445; V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:36-37,160-217,255-275; C bundle (module IdeTitlebarUpdateButton strings 11480-11482; hover opened below the button with a pointer; searched nls keys for `updateTooltip`, no hits)

### TB-5.5 Hide the Update button
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A setting turns the title bar update indicator off. Desktop only. Cursor has no such setting.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: update.titleBar
- Cursor: missing
- Source: V\src\vs\platform\update\common\update.config.contribution.ts:90-96; V\src\vs\workbench\contrib\update\browser\updateTitleBarEntry.ts:39,160-163; C bundle (searched `update.titleBar` in the bundle and the nls keys, no hits)

### TB-5.6 Cursor Settings gear button
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: A gear button in the right toolbar, always shown in the IDE window, that opens Cursor Settings. Hover text is Open Cursor Settings. Ctrl+Shift+J does the same, and Ctrl+, does too when the window is not the Agents window. Boundary reason: the button opens Cursor Settings, which configures the agent, models and account.
- Commands: workbench.action.openAgentLayoutQuickMenu; aiSettings.action.open
- Menus: none found
- Keys: Ctrl+Shift+J (Cmd+Shift+J on macOS); Ctrl+, (Cmd+, on macOS, when not isGlass)
- Settings: none found
- Cursor: cursor only
- Source: C bundle (class registered as workbench.action.openAgentLayoutQuickMenu with title string 3165 and icon settingsGear that runs `aiSettings.action.open`; right toolbar list y6w entry order 225 with isVisible true; titlebarPart strings 4038-4039; key objects with weight 401)

### TB-5.7 Upgrade button
- Tag: cursor
- Evidence: bundle
- Boundary: yes
- What: A pill button labelled Upgrade at the left end of the right toolbar. It shows only for accounts on the Free plan when no membership refresh is pending. Hover says "Upgrade for extended limits and full model access". Clicking starts the upgrade checkout for the next plan (a Stripe Pro checkout or a unified billing flow) and ignores further clicks until it finishes. Boundary reason: it depends on the signed-in account plan and starts a billing flow.
- Commands: cursor.titlebar.upgradeToPro
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (action id `cursor.titlebar.upgradeToPro`; component d6w with CSS classes `upgrade-pro-button` and `upgrade-pro-button-text`; computeToolbarState field shouldPromptUpgrade from membershipType equal to FREE; analytics event `titlebar.upgrade_pro_clicked`; right toolbar list y6w entry order 0)

### TB-5.8 Notification bell in the title bar
- Tag: vscode
- Evidence: source
- Boundary: no
- What: When notifications are set to appear at the top right (an experimental setting), a bell button at the far right of the action toolbar opens and closes the notification center. A setting and a right-click row, Notifications, hide it. Cursor has neither the button nor either setting. The id notifications.toggleList does exist in the Cursor bundle for the status bar bell, but there is no title bar row and no toggle row.
- Commands: notifications.toggleList; toggle.workbench.notifications.showInTitleBar
- Menus: TitleBar (group navigation, order 10000); TitleBarContext (group 2_config, order 5); TitleBarTitleContext (group 2_config, order 5); TitleBarContext (toggle row, order 5)
- Keys: none found
- Settings: workbench.notifications.showInTitleBar; workbench.notifications.position
- Cursor: missing
- Source: V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts:330-343; V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts:80-84; V\src\vs\workbench\browser\workbench.contribution.ts:630-648; N\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts:360 (same row, now registered through a disposable list); C bundle (searched ids and the settings); V\src\vs\workbench\browser\parts\notifications\notificationsCommands.ts; V\src\vs\workbench\browser\parts\notifications\notificationsStatus.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (searched toggle.workbench.notifications, Toggle Notifications, workbench.notifications.showInTitleBar: no hits; toggleList found)

### TB-5.9 Browser button in the title bar
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A globe button that opens a new integrated browser tab, or the list of open browser tabs when one is already open. A setting chooses always, never, or only while a browser editor is open (default). A right-click row, Integrated Browser, toggles it. Cursor has no title bar browser button, and its own Browser opens through other commands (see Handed off).
- Commands: workbench.action.browser.openOrList; workbench.action.browser.open; workbench.action.browser.quickOpen; toggle.workbench.browser.showInTitleBar
- Menus: TitleBar (group navigation, order 10); TitleBarContext (group 2_config, order 8); TitleBarTitleContext (group 2_config, order 8)
- Keys: Ctrl+Alt+/ (Cmd+Alt+/ on macOS)
- Settings: workbench.browser.showInTitleBar
- Cursor: missing
- Source: V\src\vs\workbench\contrib\browserView\electron-browser\features\browserTabManagementFeatures.ts:477-526,553-557,905-922; V\src\vs\platform\browserView\common\browserView.ts:12-19; C bundle (searched `workbench.action.browser`, `workbench.browser.` and `showInTitleBar`, no hits)

### TB-5.10 Title bar button overflow menu
- Tag: both
- Evidence: source
- Boundary: no
- What: When the action toolbar holds more buttons than its limit, the extra ones collapse into a "..." menu. Stock allows 12 and Cursor allows 10. Accounts, Manage and the editor navigation commands are never collapsed. Cursor sets the "..." position to the end of the toolbar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (limit 10 instead of 12)
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:663-680; C bundle (method createActionToolBar with overflowBehavior maxItems 10 and moreActionPlacement end)

### TB-5.11 Hover a title bar button for its name and shortcut
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering a button in the action toolbar shows its name and, when it has one, its key binding. The toolbar also carries an accessible label, Title actions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts:663-680; C bundle (createActionToolBar passes getKeyBinding and hoverDelegate; string 4026; hover on the left toolbar buttons was not checked)

## Handed off

- Go Back and Go Forward buttons that Cursor draws in its left toolbar when the command center is hidden (`workbench.action.navigateBack`, `workbench.action.navigateForward`, strings 4040-4041): TB-2
- Accounts and Manage icons and the update rows in the Manage menu (`update.check`, `update.downloadNow`, `update.install`, `update.restart`, Show Update Release Notes): AB-4. The Help menu update rows: TB-1.
- The stock post-update card (`update.showPostInstallInfo`, command `_update.showUpdateInfo`), a floating card after a restart that shows what is new: NT-6, the closest group to Cursor's what's-new cards. Cursor's bundle has no postUpdateWidget strings.
- The notification center that the bell opens, and its Do Not Disturb controls: NT-3 and NT-4. NT-3's description also names the title bar bell, so TB-5.8 and NT-3 may overlap. Dedupe when merging.
- Editor actions shown in the title bar (`workbench.editor.editorActionsLocation`): ED-1
- The Browser editor that the browser button opens (Cursor: `workbench.action.openBrowserEditor`): ED-10
- The Cursor Settings editor that the gear opens: ED-8

## Unclear

- Which users see the Cursor Update button: it needs the server experiment `ide_update_ux_exp` to give the variant `non_sidebar`. The variants are `control`, `sidebar` and `non_sidebar`. The `sidebar` variant shows the same update button in the Agents sidebar (module AgentSidebarUpdateButton, an agent surface, so out of scope). What `control` shows, if anything, was not found.
- Private Inference Mode lock button on MenuId.TitleBar in Cursor (`privateInference.turnOn`, `privateInference.turnOff`) was left out per the catalog decision on inference.
- Newer than 1.128: stock main moves the update indicator from TitleBarAdjacentCenter to a new menu TitleBarUpdate, lets Show Hover move keyboard focus into the update tooltip, and adds a role and an aria label to the button. These are changes to TB-5.1 and TB-5.4, not new features. Cursor has none of them.
- Whether the left toolbar buttons in Cursor show key bindings on hover. Only the right toolbar options were read.
- Which upgrade target the Upgrade button picks (it calls a helper with the plan info and the next upgrade). Plan names and the checkout screens were not decoded.
- Whether the title bar browser button (TB-5.9) should count as boundary because Browser is listed as an agent-driven in-scope feature. It is marked no because the stock button is not agent related and Cursor has no such button.
- Skipped as out of scope, as instructed: New Agent, Chat History, More Actions, Open Agents Window, Toggle Agents.
