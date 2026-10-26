Group: AB-1 Rail icons and view switching. Section: Activity bar. Date: 2026-10-25. Pass: 2.
Features: 12.

Reading note: the Cursor field records the stock vertical rail, which Cursor builds only when `workbench.activityBar.orientation` is `vertical`. Cursor's default is `horizontal`, which hides the stock rail and draws its own tab row (AB-6, owned elsewhere). Where the default row behaves differently, the What line says so.

### AB-1.1 Icon strip of view containers
- Tag: both
- Evidence: source
- Boundary: no
- What: A vertical strip of view container icons (Explorer, Search, Source Control, Run and Debug, Extensions, plus any extension container) sits beside the side bar, 48 px wide with 24 px icons, and each icon is a tab in a tablist named Active View Switcher; in Cursor's default horizontal row this is a different 26 px icon-only tab row at the top of the side bar (AB-6) with no tablist or tab roles found.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css; C bundle (rail part class `ore`, tablist label NLS 3321, `ariaRole:"tablist"`)

### AB-1.2 Active icon marker and rail colors
- Tag: both
- Evidence: source
- Boundary: no
- What: The open view's icon is drawn in the active color with a 2 px bar on the outer edge of the rail and the other icons are dimmed, with colors from the activityBar.* theme colors (background, border, foreground, inactiveForeground, activeBorder, activeFocusBorder, activeBackground, dropBorder), and high contrast themes hide the bar unless the icon has focus; Cursor's horizontal row instead fills the selected tab with a background and has no bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\common\theme.ts; C bundle (color ids activityBar.activeBorder, activityBar.activeFocusBorder, activityBar.activeBackground, activityBar.inactiveForeground, activityBar.dropBorder; CSS rule for `.activitybar.left` active-item-indicator)

### AB-1.3 Switch the side bar view by clicking an icon
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking an icon opens that container in the primary side bar and reveals the side bar if it was hidden, a right click never switches, and a second click within 300 ms is ignored so a double click does not toggle it back; in Cursor's horizontal row a click on a tab also opens that container.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts; C bundle (action class with `preventDoubleClickDelay=300`, right click guard `e.button===2`)

### AB-1.4 Click the open icon again to hide or focus the side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Clicking the icon of the view that is already open either hides the primary side bar (`toggle`, the default) or moves focus into it (`focus`); in Cursor's horizontal row the setting is ignored and a click on the open tab does nothing.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.iconClickBehavior
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts; V\src\vs\workbench\browser\workbench.contribution.ts; C bundle (setting definition with NLS 4228 to 4230, the same switch in the rail action class)

### AB-1.5 Hover text with name and shortcut
- Tag: both
- Evidence: source
- Boundary: no
- What: Hovering an icon shows its name followed by the shortcut of its open command in brackets, such as Explorer (Ctrl+Shift+E), on the side facing the side bar, and the same text is the icon's screen reader label; Cursor shows the hover at once on mouse over where stock waits for the hover delay, and its horizontal row set no tooltip in code (not run).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (hover appears at once, stock waits for the delay)
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; C bundle (composite bar item class using `showInstantHover` on mouse over, title format NLS 3322)

### AB-1.6 Additional Views overflow button
- Tag: both
- Evidence: source
- Boundary: no
- What: When the pinned icons do not fit the height of the rail, the extra ones go behind a "..." button at the bottom of the icon list whose menu is titled Additional Views and lists them as radio items with the open one checked; Cursor's horizontal row has a chevron drop-down instead, with pin toggles and drag reorder.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts (action id `additionalComposites.action`); C bundle (`additionalComposites.action`, label NLS 3323, `getOverflowingComposites`, radio checked representation)

### AB-1.7 The open icon always stays on the rail
- Tag: both
- Evidence: source
- Boundary: no
- What: The container that is open always has an icon on the rail, even if you hid its icon or it would not fit, and an unpinned icon goes away again once you switch to another view; Cursor's horizontal row shows the open unpinned container as one extra selected tab at the end of the row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts; V\src\vs\workbench\browser\parts\paneCompositeBar.ts; C bundle (`getOverflowingComposites` and the active item rule, identical to stock)

### AB-1.8 Icons that appear only when they have content
- Tag: both
- Evidence: source
- Boundary: no
- What: A container whose views are all inactive has no icon (Testing, Remote Explorer and References are examples), and its icon appears once one of its views turns active, such as the References icon after a references command fills the list.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (`shouldBeHidden`); V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; V\extensions\references-view\package.json (activitybar container `references-view`, view `references-view.tree` shown when `reference-list.isActive`); C bundle (`shouldBeHidden` with `hideIfEmpty`, Testing container registered with `hideIfEmpty:!0`)

### AB-1.9 Open a view with its own shortcut or Show command
- Tag: both
- Evidence: source
- Boundary: no
- What: Pressing a container's shortcut, or running its Show command from the Command Palette or the View menu, opens that view in the side bar, and pressing it again while the side bar already has focus sends focus back to the editor.
- Commands: workbench.view.explorer; workbench.view.search; workbench.view.scm; workbench.view.debug; workbench.view.extensions; workbench.view.extension.test
- Menus: MenubarViewMenu (group 3_sidebar); CommandPalette (f1)
- Keys: Ctrl+Shift+E (workbench.view.explorer); Ctrl+Shift+G (workbench.view.scm); Ctrl+Shift+D (workbench.view.debug); Ctrl+Shift+X (workbench.view.extensions); workbench.view.search is bound to Ctrl+Shift+F only behind a never-matching condition, the real Ctrl+Shift+F belongs to Find in Files; workbench.view.extension.test has no key
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\services\views\browser\viewsService.ts; V\src\vs\workbench\contrib\files\browser\explorerViewlet.ts; V\src\vs\workbench\contrib\search\browser\search.contribution.ts; V\src\vs\workbench\contrib\scm\browser\scm.contribution.ts; V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts; V\src\vs\workbench\contrib\extensions\browser\extensions.contribution.ts; V\src\vs\workbench\contrib\testing\browser\testing.contribution.ts; C bundle (the same container descriptors with the same keys, Show and Toggle labels NLS 13435 and 13436)

### AB-1.10 Step to the previous or next side bar view
- Tag: both
- Evidence: source
- Boundary: no
- What: Previous Primary Side Bar View and Next Primary Side Bar View move the open view one step back or forward through the icons currently on the rail (pinned icons plus the open one) and wrap around at the ends; in Cursor's horizontal row both commands run but cycle through every container, pinned or not.
- Commands: workbench.action.previousSideBarView; workbench.action.nextSideBarView
- Menus: CommandPalette (f1)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\compositeBarActions.ts (`SwitchCompositeViewAction`); C bundle (both command ids, titles NLS 3302 and 3303, `getVisiblePaneCompositeIds` returns every container in the horizontal part)

### AB-1.11 Focus the rail
- Tag: both
- Evidence: source
- Boundary: no
- What: Focus Activity Bar puts keyboard focus on the rail, first bringing the bar back if its location was set to hidden; in Cursor the command is enabled only when orientation is `vertical`, so it is disabled in the default horizontal row, and it shows a "quit and reopen" info dialog if the rail does not exist.
- Commands: workbench.action.focusActivityBar
- Menus: CommandPalette (f1)
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts; V\src\vs\workbench\browser\layout.ts; C bundle (command id with precondition `config.workbench.activityBar.orientation == vertical`, label NLS 3304, dialog text NLS 3301)

### AB-1.12 Move between rail icons with the keyboard
- Tag: both
- Evidence: source
- Boundary: no
- What: With focus on the rail, Up and Down (or Left and Right) move between icons without looping, Home and End jump to the first and last icon, and Enter or Space opens the focused icon; Down past the last icon moves into the Accounts and Manage icons, and Up before the first moves to the compact Menu button when it is shown; Cursor's horizontal row has no key handling in its tab code.
- Commands: none found
- Menus: none found
- Keys: Up and Down arrows (rail focused); Left and Right arrows (rail focused); Home; End; Enter; Space - hard-coded in the action bar, not rebindable
- Settings: none found
- Cursor: present
- Source: V\src\vs\base\browser\ui\actionbar\actionbar.ts; V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts; V\src\vs\workbench\browser\parts\compositeBar.ts; C bundle (`registerKeyboardNavigationListeners` with the same arrow handling, `triggerKeys:{keyDown:!0}`)

## Handed off

- Accounts and Manage icons under the rail and their menus: AB-4.
- The hamburger Menu button at the top of the rail and the Menu and Hide Menu entries in the rail menu: TB-1 (`window.menuBarVisibility`).
- Activity Bar Position and Activity Bar Size entries in the rail menu, the location setting, compact, and auto hide: AB-5.
- Move Primary Side Bar Right or Left and Hide Primary Side Bar entries in the rail menu: SB-1.
- Dropping a dragged rail icon onto the view area of another side bar container to merge their views: SB-1.
- Dragging a rail icon to the Panel tab bar, the Secondary Side Bar, or the editor edge to open those parts: PN-1 and AX-5.
- Run and Debug's two-row title with its own toolbar (Cursor `wantsActionToolbar`): SB-5.
- The pinned tab row, its chevron list, its tab menu, and the orientation setting: AB-6.

## Unclear

- The `workbench.view.*` Show commands could also be claimed by SB-2 to SB-6 (one per view). They are kept here because they are the keyboard way to switch rail views.
- Cursor's rail icon menu adds chat-only entries (Rename Chat, Open as Editor, Close Chat, Close Other Chats) for chat containers and filters out `workbench.panel.aichat.*` items. That is agent and chat surface, so it is not cataloged.
- Cursor's rail item class also has a delete button, a status indicator and an inline rename input for chat containers, and a "More Chats" label on its overflow action for the chat bar. Same reason, not cataloged.
- Whether the horizontal row shows a tooltip on hover: its tab template sets no `title` and uses no hover service, but nothing was run.
- Cursor and stock hover timing: stock calls `setupDelayedHover` with the hover delay setting, Cursor calls `showInstantHover` on mouse over (delay 1200 ms only for chat containers). Real feel was not checked on screen.
- Stock 1.144.0 shows no change to the rail click, overflow, or switch commands. `paneCompositeBar.ts` only adds an optional `transformContextMenuActionsForComposite` option, which Cursor lacks.
