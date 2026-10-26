Section: Title bar (TB). Date: 2026-10-25. Pass: 1.

# Title bar in stock VS Code 1.128.0 (V)

This is the top strip of the window: the menu bar, the command center and window title, the window controls, the layout controls, the toolbar buttons, and the Accounts and Manage entries. The full list of menu items and command ids is in the entry-points file for this section. This file holds the proposed minor groups and what stock VS Code does for each.

## What I read in V

All paths are under `c:\Users\Vinnie\cursor\runvil-design\specs\temp\vscode-1.128.0\src\vs`.

- `workbench/browser/parts/titlebar/`: `titlebarPart.ts` (1050 lines), `menubarControl.ts`, `menubar.contribution.ts`, `commandCenterControl.ts`, `windowTitle.ts`, `titlebarActions.ts`, `media/` (styles).
- `workbench/electron-browser/parts/titlebar/titlebarPart.ts`: the desktop part (custom window controls, macOS height, always on top).
- `platform/menubar/`: `common/menubar.ts`, `electron-browser/menubar.ts`, `electron-main/menubar.ts`, `electron-main/menubarMainService.ts` (the native menu path, read by symbol search and skim only).
- `platform/window/common/window.ts`: helper functions and constants for title bar style, menu style, controls style.
- `workbench/contrib/update/browser/updateTitleBarEntry.ts` and `update.ts`: the update control in the title bar.
- `workbench/browser/parts/globalCompositeBar.ts`: Accounts and Manage view items, read by symbol search only.
- Settings: `workbench/browser/workbench.contribution.ts:771-900` and `workbench/electron-browser/desktop.contribution.ts:236-320`, `platform/update/common/update.config.contribution.ts:90`.
- Not present in 1.128.0, so nothing to read: `workbench/services/titlebar/` (the service interface is `workbench/services/title/browser/titleService.ts`, only seen as an import) and a separate `layoutControlsPart` (the layout controls are menu items drawn by `titlebarPart.ts`, there is no part of their own).
- Built-in extensions: none touch this section. I searched the six in-scope manifests for `menuBar/`, `file/share`, `window/title`, `titleBar` and `commandCenter` and found nothing.

## How the part is built

`BrowserTitlebarPart` (`titlebarPart.ts:226`) makes three columns: `.titlebar-left`, `.titlebar-center`, `.titlebar-right` (`createContentArea`, line 464).

- Left: the Windows and Linux app icon (`a.window-appicon`, line 474) and the custom menu bar (`div.menubar`, line 430).
- Center: `div.window-title`, which holds either plain title text or the command center (`createTitle`, line 591).
- Right: the center-adjacent toolbar (`MenuId.TitleBarAdjacentCenter`, the update indicator, line 495), the action toolbar (editor actions, layout controls, global actions, Accounts and Manage; line 514) and the window controls container (line 521).
- A context menu on the whole bar (`MenuId.TitleBarContext`) and, on macOS, a separate one on the title text (`MenuId.TitleBarTitleContext`, Cmd+click also opens it) (lines 553-583).
- Overflow: the toolbar keeps at most 12 items before it collapses into a `...` menu; Accounts, Manage and the editor navigation commands are exempt (`overflowBehavior`, line 674).
- Height: `DEFAULT_CUSTOM_TITLEBAR_HEIGHT = 35` when the command center is visible, otherwise 30 (`titlebarPart.ts:233`, `platform/window/common/window.ts:303`). On macOS the desktop part uses 28, or 32 on macOS Tahoe or newer.
- Auxiliary windows get their own copy (`AuxiliaryBrowserTitlebarPart`, line 1013) with no menu bar and, in compact mode, a smaller set of buttons.

## Proposed minor groups

Six groups. I stopped at six because the title bar is small and every feature I found fits one of them cleanly. The split follows how a person thinks about it: "the menus", "the search box in the middle", "the window frame", "the layout buttons", "the buttons on the right", "the account and gear menus". Tags: `both` means the item exists in V and in Cursor, checked by searching the Cursor bundle.

### TB-1 Menu bar

- Name in plain words: the application menu (File, Edit, Selection, View, Go, Run, Terminal, Help).
- One sentence: the row of drop-down menus at the left of the title bar, or the OS menu on macOS, that lists most commands and recent files.
- Tag: both. Evidence: source (V), bundle (C). Boundary: no.
- Examples: File menu with Open Recent; Run menu (menu id `MenubarDebugMenu`); menu bar visibility modes (`window.menuBarVisibility`); Help menu with update rows.
- Proof (V): `workbench/browser/parts/titlebar/menubar.contribution.ts:10-90` (top-level titles and order), `menubarControl.ts` (`CustomMenubarControl`, recents at 173-181, update rows 380-420, focus action 300), `workbench/contrib/debug/browser/debug.contribution.ts:261` (Run), `platform/menubar/electron-main/menubar.ts` (native menu), `platform/actions/common/actions.ts:117-145` (menu ids). Items: see the entry-points file.
- Behavior seen in source:
  - The custom menu bar is installed only when the OS menu is not used: not on macOS native, not in auxiliary windows, and not in `compact` mode (`titlebarPart.ts:481-489`, `hasNativeMenu` in `platform/window/common/window.ts:171`).
  - `window.menuBarVisibility` has five values: `classic` (default on desktop), `visible`, `toggle` (Alt shows it), `hidden`, `compact` (moves the menu into the side bar; default on web) (`workbench.contribution.ts:868-888`). Windows and Linux only.
  - Related settings: `window.enableMenuBarMnemonics`, `window.customMenuBarAltFocus` (Windows and Linux only, `workbench.contribution.ts:889-902`), `window.menuStyle` (`custom`, `native`, `inherit`; `desktop.contribution.ts:270`).
  - Commands: `workbench.action.toggleMenuBar` (View > Appearance > Menu Bar, and in the title bar context menu; Windows, Linux and web only, `layoutActions.ts:731`), `workbench.actions.menubar.focus` (Focus Application Menu), `workbench.action.focusTitleBar` (`titlebarPart.ts:121`).

### TB-2 Command center and window title

- Name in plain words: the search box and the window title.
- One sentence: the middle of the title bar, which shows the workspace name as a search-style box (command center) with back and forward arrows, or just the window title text when the command center is off.
- Tag: both. Evidence: source (V), bundle (C). Boundary: no.
- Examples: command center search box (opens Quick Open); back and forward arrows; window title template (`window.title`); title bar right-click menu with toggles.
- Proof (V): `commandCenterControl.ts` (control, `CommandCenterCenterViewItem`, label and tooltip logic), `windowTitle.ts` (template and variables), `editorActions.ts:1444,1479` (Back and Forward registered on `MenuId.CommandCenter`), `workbench/browser/actions/quickAccessActions.ts:161` (Quick Open on `MenuId.CommandCenterCenter`), `titlebarActions.ts:28-100,222-243` (context menu toggles), `workbench.contribution.ts:849-867` (settings).
- Behavior seen in source:
  - The box shows the workspace name, or the file name when tabs are hidden, or the whole formatted title when `window.title` is customised. With nothing to show it says "Search". Hover shows "Search {workspace} ({keybinding}) - {title}" (`commandCenterControl.ts:211-262`).
  - In compact mode (setting `chat.agentsControl.enabled` unset, true or `compact`) the search icon is hidden and the label is left-aligned (`commandCenterControl.ts:166-176`). That setting belongs to the chat feature, so this detail is a boundary note for a later pass.
  - It hides while a quick input is open at the top (`commandCenterControl.ts:62-79`).
  - `window.commandCenter` (default true), `workbench.navigationControl.enabled` (default true), `window.title`, `window.titleSeparator` (`" - "`, or an em dash on macOS), `window.customTitleBarVisibility`.
  - Title variables: `activeEditorShort`, `activeEditorMedium`, `activeEditorLong`, `activeEditorLanguageId`, `activeFolderShort/Medium/Long`, `folderName`, `folderPath`, `rootName`, `rootNameShort`, `rootPath`, `profileName`, `appName`, `remoteName`, `dirty`, `focusedView`, `activeRepositoryName`, `activeRepositoryBranchName`, `activeEditorState`, `separator` (`workbench.contribution.ts:822-846`). Extensions can add more through the command `registerWindowTitleVariable` (`titlebarPart.ts:138`).
  - Title bar right-click toggles: Command Center, Navigation Controls, Layout Controls, Notifications, Editor Actions, Integrated Browser (`browserTabManagementFeatures.ts:555`), Menu Bar, Hide Custom Title Bar and Hide Custom Title Bar In Full Screen.
  - Also on the command center menu: Share... (`workbench.action.share`, only with `workbench.experimental.share.enabled`) and, in debug mode with `debug.toolBarLocation` set to `commandCenter`, the debug toolbar (`debugToolBar.ts:438`).

### TB-3 Window frame and controls

- Name in plain words: the window frame.
- One sentence: whether the window has an OS title bar or a drawn one, the minimize, maximize and close buttons, the app icon, the drag area and how the bar behaves when zoomed or full screen.
- Tag: both. Evidence: source (V), bundle (C). Boundary: no.
- Examples: custom or native title bar (`window.titleBarStyle`); minimize, maximize and close buttons (`window.controlsStyle`); app icon double-click to close (`window.doubleClickIconToClose`); show or hide the custom title bar (`window.customTitleBarVisibility`).
- Proof (V): `titlebarPart.ts:464-590` (icon, drag region, controls container, macOS traffic light side, RTL check), `workbench/electron-browser/parts/titlebar/titlebarPart.ts:33-230` (custom minimize, maximize and close icons at 181-205, resizer at 208, always-on-top context key at 94, double-click icon at 120-175), `platform/window/common/window.ts:148-303` (style helpers), `desktop.contribution.ts:236-320` (settings), `titlebarActions.ts:84-224` (hide and show actions).
- Behavior seen in source:
  - `window.titleBarStyle`: `native` or `custom`, default `custom`. `window.controlsStyle` (not macOS): `native`, `custom`, `hidden`, default `native`. `window.customTitleBarVisibility`: `auto`, `windowed`, `never`. `window.dialogStyle`, `window.nativeTabs` (macOS), `window.clickThroughInactive`, `window.border` are neighbours in the same block.
  - Commands: `showCustomTitleBar`, `hideCustomTitleBar`, `hideCustomTitleBarInFullScreen` (all in the Command Palette, `titlebarActions.ts:178-224`), and View > Appearance > Custom Title Bar (`toggle.toggleCustomTitleBar`, line 131).
  - Web: the window controls overlay (WCO) area is respected on both sides (`titlebarPart.ts:547-552`).
  - The bar counter-scales against zoom (`preventZoom`, the `zoom-factor` CSS variable) so it stays a fixed pixel size when it has no zoomable content (`titlebarPart.ts:890-910,945-952`).
  - Auxiliary windows have a compact mode and an always-on-top toggle on the layout menu (`auxiliaryEditorPart.ts:75-96`, `windowActions.ts:526-556`).

### TB-4 Layout controls

- Name in plain words: the layout buttons.
- One sentence: small buttons at the right of the title bar that hide or show the primary side bar, the secondary side bar and the panel, or a Configure Layout drop-down with the same choices.
- Tag: both. Evidence: source (V), bundle (C). Boundary: no for V. In Cursor the layout presets reach into agent panes, see the Cursor file.
- Examples: side bar and panel toggle buttons; Configure Layout drop-down; Customize Layout...; layout control type (`workbench.layoutControl.type`).
- Proof (V): `workbench/browser/actions/layoutActions.ts:172-182` (Configure Layout submenu), `349-395` (primary side bar toggles), `1420-1445` (Customize Layout...), `workbench/browser/parts/auxiliarybar/auxiliaryBarActions.ts:130-175`, `workbench/browser/parts/panel/panelActions.ts:300-340`, `titlebarPart.ts:773-785` (reads `MenuId.LayoutControlMenu`).
- Behavior seen in source:
  - `workbench.layoutControl.enabled` (default true) and `workbench.layoutControl.type` (`menu`, `toggles`, `both`; default `both`) (`workbench.contribution.ts:777-798`). Toggles show when the type is `toggles` or `both`. The Configure Layout drop-down shows when the type is `menu` (`layoutActions.ts:174-182`). Customize Layout... (a quick pick with toggles for each part, `layoutActions.ts:1420-1470`) shows as its own button when the type is `both`: in group `navigation` normally, or in group `1_layout` when editor actions live in the title bar (`layoutActions.ts:1432-1445`). It is also the last row of the Configure Layout drop-down.
  - The primary side bar toggle changes its icon with the side bar position (left or right) and its toggled state.
  - Not in the layout menu but part of the same area: Turn On or Off Compact Mode and Always on Top for auxiliary windows.
  - `workbench.experimental.modernUI` exists in V (`layoutService.ts:53`, default false): when on, side bars and panel float as cards and the title bar colours change. N refines it (see newer-than-1.128).

### TB-5 Title bar buttons and update control

- Name in plain words: the buttons next to the search box and at the far right.
- One sentence: single-purpose buttons the title bar can show, including the update indicator, the notification bell and the integrated browser button.
- Tag: both. Evidence: source (V), bundle (C). Boundary: yes in Cursor (the Cursor Settings and Upgrade buttons depend on the account and on agent features), no in V.
- Examples: update indicator (V: `workbench.actions.updateIndicator`); notification bell (`notifications.toggleList`); integrated browser button (`workbench.action.browser.openOrList`); editor actions placed in the title bar (`workbench.editor.editorActionsLocation: titleBar`, belongs to the editor section).
- Proof (V): `workbench/contrib/update/browser/updateTitleBarEntry.ts` (370 lines), `workbench/browser/parts/notifications/notificationsCommands.ts:331`, `workbench/contrib/browserView/electron-browser/features/browserTabManagementFeatures.ts:477-548,908`, `titlebarPart.ts:495-512,692-752` (toolbar assembly), `update.config.contribution.ts:90`.
- Behavior seen in source:
  - Update indicator: command `workbench.actions.updateIndicator`, title "Update", on `MenuId.TitleBarAdjacentCenter` (`updateTitleBarEntry.ts:57-75`). It is hidden in editor Zen mode, in debug mode and while a chat request is running. It follows the update state: Update (available, downloaded or ready), Checking..., Downloading..., Installing..., Updating..., Restarting..., Cancelling.... Clicking it runs `update.downloadNow`, `update.install` or `update.restart` depending on state (`updateTitleBarEntry.ts:283-300`). Setting `update.titleBar` (default true, desktop only) turns it off.
  - The bar hides this toolbar when it would push the window controls off screen (`titlebarPart.ts:918-936`, issue 303222).
  - Notification bell: `notifications.toggleList` on `MenuId.TitleBar`, group `navigation`, order 10000, shown when `workbench.notifications.position` is `top-right` and `workbench.notifications.showInTitleBar` is on.
  - Browser button: `workbench.action.browser.openOrList`, icon `globe`, key Ctrl+Alt+/ (Cmd+Alt+/), group `navigation` order 10 on `MenuId.TitleBar`. `workbench.browser.showInTitleBar` takes `true` (always), `false` (never) or `whenOpen` (default, only while a browser editor is open) (`browserTabManagementFeatures.ts:489-501,908-919`). With no browser open the command opens one, with one open it opens the quick pick of browser tabs (`browserTabManagementFeatures.ts:508-520`). Also in View (`4_auxbar`).
  - Layout of the right side, in order: leading global actions, editor actions (only if they are set to live in the title bar), layout controls, global actions, Accounts, Manage.

### TB-6 Accounts and Manage entries

- Name in plain words: the person icon and the gear icon.
- One sentence: two menus, Accounts (sign-in state and permission requests) and Manage (command palette, settings, themes, updates), shown in the title bar when the activity bar is moved to the top or bottom.
- Tag: vscode for the title bar placement. The menus themselves exist in Cursor too (see the Cursor file). Evidence: source (V). Boundary: no.
- Examples: Accounts menu (Sign in to Sync Settings, Remote Tunnel, extension access requests); Manage menu (Command Palette, Settings, Extensions, Keyboard Shortcuts, Themes); Check for Updates and Restart to Update rows; account visibility preference.
- Proof (V): `titlebarPart.ts:741-752,886-888` (shown only when `workbench.activityBar.location` is `top` or `bottom` and the part is not auxiliary), `titlebarActions.ts:277-291` (the two tile actions `ACCOUNTS_ACTIVITY_TILE_ACTION` and `GLOBAL_ACTIVITY_TITLE_ACTION`), `workbench/browser/parts/globalCompositeBar.ts:295,609` (the two menus `AccountsContext` and `GlobalActivity`), `workbench/common/activity.ts` (ids). Item lists: entry-points file, 18 `AccountsContext` and 16 plus 7 update `GlobalActivity` registrations.
- Behavior seen in source:
  - In the default layout (activity bar at the side) the same two entries live at the bottom of the activity bar, which belongs to the Activity bar section.
  - `workbench.activity.showAccounts` (storage key `ACCOUNTS_VISIBILITY_PREFERENCE_KEY`) lets the user hide Accounts. Manage is always there.
  - Hover appears below the icon (`HoverPosition.BELOW`).

## Settings that shape the title bar (V)

| Setting | Default | Where |
|-|-|-|
| `window.titleBarStyle` | `custom` | `desktop.contribution.ts:243` |
| `window.controlsStyle` | `native` (not macOS) | `desktop.contribution.ts:250` |
| `window.customTitleBarVisibility` | `auto` | `desktop.contribution.ts:258` |
| `window.menuStyle` | `inherit` (macOS stable: `native`) | `desktop.contribution.ts:270` |
| `window.doubleClickIconToClose` | `false` | `desktop.contribution.ts:237` |
| `window.menuBarVisibility` | `classic` (web: `compact`) | `workbench.contribution.ts:868` |
| `window.enableMenuBarMnemonics`, `window.customMenuBarAltFocus` | `true` | `workbench.contribution.ts:889,895` |
| `window.title`, `window.titleSeparator` | template in `windowTitle.ts:41-53` | `workbench.contribution.ts:849-860` |
| `window.commandCenter` | `true` | `workbench.contribution.ts:860` |
| `workbench.navigationControl.enabled` | `true` | `workbench.contribution.ts:771` |
| `workbench.layoutControl.enabled`, `workbench.layoutControl.type` | `true`, `both` | `workbench.contribution.ts:777,787` |
| `workbench.notifications.showInTitleBar`, `workbench.notifications.position` | see source | `workbench/common/notifications.ts:800-801` |
| `workbench.browser.showInTitleBar` | `whenOpen` (also `true`, `false`) | `browserTabManagementFeatures.ts:908` |
| `update.titleBar` | `true` (desktop only) | `update.config.contribution.ts:90` |
| `workbench.activityBar.location` | `default` | decides whether Accounts and Manage sit in the title bar |
| `workbench.editor.editorActionsLocation` | `default` | `titleBar` puts editor actions in the title bar |
| `workbench.experimental.modernUI` | `false` | `layoutService.ts:53` |

Context keys: `TitleBarVisibleContext`, `TitleBarStyleContext`, `IsCompactTitleBarContext`, `IsMainWindowFullscreenContext` (`workbench/common/contextkeys.ts`, used in `titlebarActions.ts`).


## Newer than 1.128 (seen only in N, `C:\Users\Vinnie\src\vscode`, 1.144.0)

All marked `newer-than-1.128`. Nothing here is in Cursor. I compared the same four files by line (`titlebarPart.ts`, `titlebarActions.ts`, `menubarControl.ts`, `commandCenterControl.ts`) and counted `MenuId.Menubar*` references (equal in V and N, so no menu bar items were added or removed between the two).

- New menu id `MenuId.TitleBarUpdate` (3 references in N, `platform/actions/common/actions.ts:185`). The title bar gets a second optional toolbar, `div.update-toolbar-container`, placed before the right-aligned actions (`workbench/browser/parts/titlebar/titlebarPart.ts` in N). V has only `TitleBarAdjacentCenter`.
- N hides both optional toolbars (center-adjacent and update) with one animation-frame scheduler (`titleBarToolBarOverflowScheduler`, `updateTitleBarToolBarOverflow`) when the bar overflows. V does it for the center-adjacent toolbar only.
- Open and close animation on the drop-down menus of the title bar action toolbar (`dropdownMenuClassName: WORKBENCH_MENU_MOTION_CLASS` and `dropdownMenuCloseAnimation: workbenchMenuCloseAnimation`, from `workbench/browser/actions/menuMotion.ts`, in N `titlebarPart.ts`).
- The custom menu bar adds a `menubar-focused` class on the title bar element.
- Recent entries in File > Open Recent are cut in the middle after 120 characters (`MAX_MENU_RECENT_LABEL_LENGTH = 120`, `menubarControl.ts` in N).
- Modern UI title bar colours: `MODERN_UI_SHELL_BACKGROUND`, `MODERN_UI_INACTIVE_SHELL_BACKGROUND`, a `modern-ui-shell-background` CSS variable, and no title bar border when `workbench.experimental.modernUI` is on. V has the setting but not these colours.
- Window active state is tracked per window (`activeWindowId`) so the inactive style is correct with several windows.
- `titlebarActions.ts` in N adds an `IsSessionsWindowContext.negate()` guard to the Custom Title Bar toggle in the Appearance menu.
- `commandCenterControl.ts` in N reads `ChatAIDisabledSettingId` instead of the string `chat.disableAIFeatures`.

## Left out of this section

- Anything about chat or agents in the title bar: the `Agents` control on `MenuId.CommandCenter` (`AgentsTitleBarControlMenu`, `agentSessionsExperiments.contribution.ts:254`), the `Chat` submenu on `MenuId.TitleBar` (`agentSessionsExperiments.contribution.ts:268`), `Open in Agents` (`workbench.action.openWorkspaceInAgentsWindow`, `agentSessionsActions.ts:56`), the chat sign-in button on `TitleBarAdjacentCenter` (`ChatSetupSignInTitleBarAction`), the Accounts rows for Copilot sign-in and language model access, the `chat.agentsControl.enabled` setting, and the View menu entry Chat.
- The whole Agents window code, `src/vs/sessions` (`sessions/browser/parts/menubar.contribution.ts` repeats the menu bar items for that window).
- Editor actions shown in the title bar (`workbench.editor.editorActionsLocation`): they are `MenuId.EditorTitle` items and belong to the Editor section. The title bar part only hosts them.
- The Extensions view entries in View and Preferences, and the Extension Monitor row (Cursor), because the extension system is out of scope.
- Notification center contents (the bell only opens it): Notifications section.
- Remote tunnel, Cloud Changes and Settings Sync rows inside the Accounts menu are listed in the entry-points file for completeness but I did not describe them as features.

## Could not verify

- Default keybindings of the menu items. I did not extract them.
- Native macOS menu behavior (`platform/menubar/electron-main/menubar.ts`): read by symbol search and skim, not run.
- How `MenubarShare` is filled for share providers: the core registers only Export Profile and `share.contribution.ts` lists `MenubarShare` as a target menu for share actions behind `workbench.experimental.share.enabled`. I did not trace the extension path beyond the `github` extension row.
- The exact `when` of a few rows is cut at 90 characters in the entry-points tables. The full expression is at the file and line shown.
- Pixel sizes and styles (`media/titlebarpart.css`) were not read. Later passes should read them.
- Hover texts and tooltips for the layout toggle buttons were not captured (they come from the `title` of each menu command).
