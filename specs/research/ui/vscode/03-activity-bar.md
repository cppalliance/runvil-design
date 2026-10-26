# Activity bar (AB) - VS Code 1.128.0 findings

Section: Activity bar (AB). Date: 2026-10-25. Pass: 1.

- Scope of this file: stock VS Code 1.128.0 only (root V). The Cursor delta is in the sibling file under `cursor\`.
- Level: minor groups only. Each group lists up to 4 example features, not all of them. No pieces yet.
- Paths below are relative to V unless they start with N (VS Code main 1.144.0, used only to spot newer things).

## What the section is in V

The activity bar is the strip of icons that switches the active view container in the primary side bar. Two icon stacks live in it: the view container icons (Explorer, Search, Source Control, Run and Debug, Extensions, Testing and so on) and, pinned at the bottom, the global items Accounts and Manage (the gear).

- Default look: a vertical rail, 48 px wide, 48 px tall items, 24 px icons, left of the side bar. Compact size is 36 px wide, 32 px items, 16 px icons.
- Top or bottom location: the rail is not drawn. `SidebarPart` draws the same container icons as a small icon tab strip inside the side bar title (top) or at the side bar bottom. That is still the activity bar. In this mode Accounts and Manage move to the title bar.
- Hidden location: nothing is drawn, and Accounts and Manage are not drawn anywhere (the title bar draws them only for top or bottom).
- Optional hamburger Menu button sits at the top of the rail when `window.menuBarVisibility` is `compact`.

Proof: `src\vs\workbench\browser\parts\activitybar\activitybarPart.ts` lines 44-60 (sizes), 138-169 (rail options), 171-181 (content area), 275-448 (ActivityBarCompositeBar, menu, global bar); `src\vs\workbench\browser\parts\sidebar\sidebarPart.ts` lines 133-160 (switching), 198-232 (top and bottom strip options), 235-280 (when the strip shows); `src\vs\workbench\browser\parts\titlebar\titlebarPart.ts` line 884 (title bar shows Accounts and Manage only when location is top or bottom).

## Proposed minor groups

- AB-1 Rail icons and view switching - tag `vscode` (cursor: see sibling file), evidence `source`, boundary no
  - One sentence: the icon strip itself, how a click or key press switches the side bar view, and how too many icons overflow.
  - Examples: click an icon to open its view, and a second click toggles or focuses the side bar (`workbench.activityBar.iconClickBehavior`); active item marker and hover text that includes the keybinding; "Additional Views" overflow menu; Previous and Next Primary Side Bar View plus Focus Activity Bar, with arrow key navigation inside the rail.
  - Proof: `src\vs\workbench\browser\parts\paneCompositeBar.ts` 765-833 (ViewContainerActivityAction, double click guard 300 ms, iconClickBehavior at 813); `src\vs\workbench\browser\parts\compositeBarActions.ts` 416-433 (title with keybinding), 449-512 (overflow action and menu), 760-791 (SwitchCompositeViewAction); `activitybarPart.ts` 375-408 (arrow key navigation), 603-640 (commands `workbench.action.previousSideBarView`, `workbench.action.nextSideBarView`, `workbench.action.focusActivityBar`); `src\vs\workbench\browser\parts\compositeBar.ts` 623-652 (what overflows).
- AB-2 Customizing the rail - tag `vscode`, evidence `source`, boundary no
  - One sentence: choosing which containers show, their order, and where each container lives, through the context menus and drag and drop.
  - Examples: Hide or Keep a container icon from its context menu; drag icons to reorder; drag a container or a single view onto the rail to move it here or create a new container; Move To (Primary Side Bar, Secondary Side Bar, Panel) and Reset Location.
  - Proof: `compositeBarActions.ts` 637-685 (icon context menu order), 722-742 (pin toggle), 560-596 (drag handlers, activate on drag over); `compositeBar.ts` 36-160 (CompositeDragAndDrop: composite and view drops), 398-416 (pin, unpin), 664-686 (rail background menu lists every container with a check); `paneCompositeBar.ts` 157-215 (Move To and Reset Location), 339-349 (new containers are pinned by default).
- AB-3 Badges on rail icons - tag `vscode`, evidence `source`, boundary no
  - One sentence: the small count, progress, or icon marker drawn on a rail icon and the per-icon switch to hide it.
  - Examples: number badge (Source Control changes, with 1000 or more shown as `1K+`); progress badge; icon badge; Hide Badge and Show Badge on the icon menu.
  - Proof: `compositeBarActions.ts` 308-372 (badge rendering), 374-398 (progress beats icon beats number), 416-433 (badge description added to hover text), 662-667 (Hide Badge and Show Badge labels), 744-757 (ToggleCompositeBadgeAction); `compositeBar.ts` 418-424 (badge enablement stored through the view descriptor service); badge colors `activityBarBadge.background` and `activityBarBadge.foreground` in `src\vs\workbench\common\theme.ts`.
- AB-4 Accounts and Manage entries - tag `vscode`, evidence `source`, boundary no
  - One sentence: the two global icons under the rail, their drop-down menus, and the setting to hide Accounts.
  - Examples: Accounts menu (signed-in accounts each with Manage Trusted Extensions, Manage Trusted MCP Servers, Sign Out; sign-in and access request items with a count badge; Backup and Sync Settings; Turn on Cloud Changes; Sign in to use GitHub Copilot); Manage menu (Command Palette, Profile, Settings, Keyboard Shortcuts, Extensions, Tasks, Snippets, Themes, update items); Hide Accounts and the Accounts toggle in the rail menu; profile text badge on the gear for a non-default profile.
  - Proof: `src\vs\workbench\browser\parts\globalCompositeBar.ts` 48-151 (GlobalCompositeBar, Accounts index 0, Manage always present, key `workbench.activity.showAccounts` at 260), 258-580 (Accounts menu build), 583-660 (Manage item and profile badge), 729-741 (title bar variant menu: Manage cannot be hidden); menu contributions `MenuId.AccountsContext` and `MenuId.GlobalActivity` listed under "Menu contributions" below.
- AB-5 Placement, size, and visibility - tag `vscode`, evidence `source`, boundary no
  - One sentence: where the activity bar lives (side, top, bottom, hidden), how big it is, and the commands and settings that change that.
  - Examples: `workbench.activityBar.location` (default, top, bottom, hidden); Activity Bar Position submenu with the four Move Activity Bar commands; `workbench.activityBar.compact` and the Activity Bar Size submenu (Default, Compact); `workbench.activityBar.autoHide` and Toggle Activity Bar Visibility.
  - Proof: `src\vs\workbench\browser\workbench.contribution.ts` 650-685 (settings); `activitybarPart.ts` 451-478 (menu inside the rail context menu), 480-581 (four position commands), 583-601 (View > Appearance submenu and the View Container title context entry); `sidebarPart.ts` 323-334 (Toggle Activity Bar Visibility, remembers the last visible position); `src\vs\workbench\browser\actions\layoutActions.ts` 62, 1366 (command id and Customize Layout toggle), 1594 (reset on Reset Layout).
- AB-6 Pinned container tab row (Cursor's replacement) - tag `cursor`, evidence `bundle`, boundary no
  - V has no direct counterpart. The nearest V thing is the top or bottom location in AB-5 (icon strip inside the side bar). V content for it is in the sibling Cursor file, because the group only exists in Cursor.

Confidence that this grouping is the right shape: high - each group matches one place a person would look (the strip, customizing it, badges, the two global icons, where it lives), and AB-6 is the only group the V side cannot fill.

## Settings (V)

- `workbench.activityBar.location`: string, `default` | `top` | `bottom` | `hidden`, default `default`. Descriptions: default = side of the Primary Side Bar and top of the Secondary Side Bar; top; bottom; hide. `workbench.contribution.ts` 650-662.
- `workbench.activityBar.autoHide`: boolean, default false. Hides the bar when only one view container shows. Applies when location is top or bottom. `workbench.contribution.ts` 663-668; logic in `sidebarPart.ts` 133-141 and 235-252.
- `workbench.activityBar.compact`: boolean, default false. Smaller icons and narrower rail. Applies only when location is `default`. `workbench.contribution.ts` 669-674; `activitybarPart.ts` 44-52, 95-110.
- `workbench.activityBar.iconClickBehavior`: `toggle` (hide the side bar if the clicked view is already visible) or `focus` (focus the side bar), default `toggle`. Ignored when location is not `default`. `workbench.contribution.ts` 675-685; `paneCompositeBar.ts` 810-830.
- `workbench.view.alwaysShowHeaderActions`: boolean, default false. This one is about view header actions inside the side bar, not the rail. Handed to SB (see left out). `workbench.contribution.ts` 686-690; `src\vs\workbench\browser\parts\views\viewPane.ts` 313, 702.
- Stored state (not settings): `workbench.activity.pinnedViewlets2`, `workbench.activity.placeholderViewlets`, `workbench.activity.viewletsWorkspaceState` (`activitybarPart.ts` 62-64); `workbench.activity.showAccounts` (`globalCompositeBar.ts` 260).
- Theme colors: `activityBar.background`, `.foreground`, `.inactiveForeground`, `.border`, `.activeBorder`, `.activeFocusBorder`, `.activeBackground`, `.dropBorder`, `activityBarBadge.background`, `activityBarBadge.foreground`; for the top or bottom strip `activityBarTop.background`, `.foreground`, `.inactiveForeground`, `.activeBorder`, `.activeBackground`, `.dropBorder`. `src\vs\workbench\common\theme.ts` (registerColor calls).
- Window menu: `window.menuBarVisibility` = `compact` adds a hamburger Menu button at the top of the rail. `activitybarPart.ts` 313-373, 326-337.

## Context menu order (V)

Right-click on a rail icon (`compositeBarActions.ts` 637-685), top to bottom:

- Configure Keybinding (only when the container has a keybinding id)
- Hide '{name}' or Keep '{name}' (Hide is disabled when it is the last pinned container)
- Hide Badge or Show Badge
- Separator, then Move To (Primary Side Bar, Secondary Side Bar, Panel; the current location is skipped) and Reset Location (only when the container or its single view is away from its default home) - `paneCompositeBar.ts` 157-199
- Separator, then the shared rail items from `ActivityBarCompositeBar.fillContextMenuActions` (`activitybarPart.ts` 326-346, 451-478): Menu (checked when compact) and Hide Menu, the Accounts toggle (checked when Accounts is shown), Activity Bar Position submenu, Activity Bar Size submenu (only when location is default), Move Primary Side Bar Right or Left, Hide Primary Side Bar.

Right-click on empty rail space shows the same lower part plus a checklist of every visible container (`compositeBar.ts` 664-686). In the top or bottom strip the menu also gets a Views submenu (`sidebarPart.ts` 209-217).

## Menu contributions that fill Accounts and Manage (V)

Manage gear (`MenuId.GlobalActivity`), by group:

- `1_command`: Command Palette... (`src\vs\workbench\contrib\quickaccess\browser\quickAccess.contribution.ts` 91-98)
- `2_configuration`: Profile ({name}) submenu (order 1, `contrib\userDataProfile\browser\userDataProfile.ts` 180-190), Settings (order 2, `contrib\preferences\browser\preferences.contribution.ts` 241-250), Extensions (order 3, `contrib\extensions\browser\extensions.contribution.ts` 676-683), Keyboard Shortcuts (order 4, `preferences.contribution.ts` 909-913), Configure Snippets (order 5, `contrib\snippets\browser\commands\configureSnippets.ts` 257-266), Tasks (order 6, `contrib\tasks\browser\task.contribution.ts` 363-373), Themes submenu (order 7, `contrib\themes\browser\themes.contribution.ts` 903-908)
- `3_configuration`: Backup and Sync Settings... and the other Settings Sync items (`contrib\userDataSync\browser\userDataSync.ts` 774-935)
- `7_update`: update items such as Show Update Release Notes (`contrib\update\browser\update.ts` 319, 333-340)

Accounts (`MenuId.AccountsContext`), by group:

- `1_settings`: Backup and Sync Settings..., Sign in to Sync Settings (1) (`userDataSync.ts` 784, 867-875)
- `2_editSessions`: Turn on Cloud Changes... (`contrib\editSessions\browser\editSessionsStorageService.ts` 475-492)
- `2_remoteTunnel`: remote tunnel turn on, manage (`contrib\remoteTunnel\electron-browser\remoteTunnel.contribution.ts` 503, 601, 620)
- `2_copilot`: Sign in to use GitHub Copilot... (`contrib\chat\browser\chatSetup\chatSetupContributions.ts` 369-380). Chat feature, boundary for AB-4, not cataloged further.
- `2_signInRequests` and `3_accessRequests`: "Sign in with {0} to use {1} (1)" and "Grant access to {0} for {1}... (1)" for extensions and MCP servers (`src\vs\workbench\services\authentication\browser\authenticationExtensionsService.ts` 401, 463; `authenticationMcpService.ts` 453, 513)
- no group, order 100: Manage Extension Account Preferences... (`contrib\authentication\browser\actions\manageAccountPreferencesForExtensionAction.ts` 26-31), Manage Language Model Access... (`contrib\chat\browser\actions\chatLanguageModelActions.ts` 33-39), Sign in to access Extensions Marketplace (`extensions.contribution.ts` 2118-2123)
- built into the item: one submenu per signed-in account with Manage Trusted Extensions, Manage Trusted MCP Servers, Sign Out; "Loading..." while accounts resolve; "{0} is currently unavailable"; Manage Dynamic Authentication Providers... (`globalCompositeBar.ts` 358-500)

## Built-in extension contributions (V)

- `extensions\references-view\package.json` lines 49-57: `contributes.viewsContainers.activitybar` adds one container `references-view` with icon `$(references)`, title from `%container.title%`. Its view `references-view.tree` shows only when `reference-list.isActive`, so the icon appears on demand. The icon is a rail entry (AB-1); the view contents belong to the references-view catalog. No other V built-in extension adds a rail container except `extensions\copilot`, which adds two (`copilot-chat` titled "Chat Debug" and `context-inspector` titled "Language Context Inspector", `extensions\copilot\package.json` 6564-6576); both are chat debug tools, out of scope. git, markdown-language-features, merge-conflict, simple-browser, and search-result add none.

## Left out (V)

- `workbench.view.alwaysShowHeaderActions`: view header actions inside the side bar. Belongs to SB. `viewPane.ts` 313, 702.
- `workbench.secondarySideBar.showLabels` and the Secondary Side Bar title icon switcher: belongs to AX (auxiliary bar). `workbench.contribution.ts` ~619-623, `src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts` 244.
- Title bar placement of Accounts and Manage (top or bottom location): drawn by the title bar, so TB catalogs where they sit. AB-4 owns what is in their menus. `titlebarPart.ts` 735-755, 884-887; `titlebarActions.ts` 277-292.
- Floating panels or Modern UI gutter around the rail (`workbench.experimental.modernUI`): experimental layout, not part of the plain activity bar. `activitybarPart.ts` 52-60, 77-79, 250-260; `workbench.contribution.ts` 814.
- Sessions window variants (`IsSessionsWindowContext`): the position commands hide there; the Agents window is out of scope. `activitybarPart.ts` 494-498.
- Explorer, Search, Source Control, Run and Debug, Extensions, Testing view contents: side bar and other sections. Only their icons are rail entries.

## Newer than 1.128 (seen only in N)

- `workbench.accounts.showAvatar` (boolean, default true): shows the signed-in account picture on the Accounts icon. `N\src\vs\workbench\browser\workbench.contribution.ts` 669-673; `N\src\vs\workbench\browser\parts\globalCompositeBar.ts` (avatar image code); constant in `N\src\vs\workbench\services\authentication\common\authentication.ts` 22. `newer-than-1.128`.
- A Codex account entry set in the Accounts menu (`codexAccountService`), agent related. `N\...\globalCompositeBar.ts` imports. `newer-than-1.128`, out of scope.
- Menu open and close animation (`menuMotion`) on the Accounts and Manage menus. `N\...\globalCompositeBar.ts`. `newer-than-1.128`.
- Compact rail items are 28 px tall in N (32 px in V), plus a Modern UI density setting `window.density.layout` and an inactive-window rail color. `N\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts` (COMPACT_ACTION_HEIGHT = 28, MODERN_ACTIVITY_BAR_INACTIVE_BACKGROUND); `N\src\vs\workbench\services\layout\browser\layoutService.ts` 55. `newer-than-1.128`.
- The rail action set (four position commands, two size commands), badges, pin, and drag and drop are otherwise the same in N: `compositeBar.ts` and `compositeBarActions.ts` show no diff against V, `paneCompositeBar.ts` differs by 6 lines.

## Could not verify

- Pixel sizes, hover delays, and animation timing: read from source constants only; nothing was run.
- Exact default theme color values for the activity bar colors: ids only, not the default hex values.
- The full list of `GlobalActivity` and `AccountsContext` items when remote, sync, or extension contributions are active. The list above is every registration found by searching `MenuId.GlobalActivity` and `MenuId.AccountsContext` under `src\vs`; extension-contributed items (`menus` key `accounts` or `statusBar`) were not enumerated.
- Order of icons by default (container `order` values) was not collected here; that belongs with each view container in SB.

