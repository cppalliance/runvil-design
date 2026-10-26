Group: AB-5 Placement, size and visibility. Section: Activity bar. Date: 2026-10-25. Pass: 2.
Features: 9.

### AB-5.1 Choose where the activity bar lives in Settings
- Tag: both
- Evidence: source
- Boundary: no
- What: Set `workbench.activityBar.location` to default (rail beside the Primary Side Bar), top, bottom or hidden, from the Settings editor or settings.json. Old values migrate: `workbench.activityBar.visible` set to false becomes hidden, and `side` becomes default.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: changed (the setting is kept with the same four values and the same descriptions, but the side bar and rail that read it load only in the vertical layout; the default horizontal tab row never reads it, and only small side effects still follow it there: the title bar empty check, a view header refresh, and the hover side of the compact menu button)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts 650-662 (setting), 974-995 (migrations); V\src\vs\workbench\services\layout\browser\layoutService.ts 45-47; V\src\vs\workbench\browser\layout.ts 3154-3160 (when the rail counts as hidden); C bundle: setting definition (NLS 4222-4226), migrations for `workbench.activityBar.visible` and `side`, class `ePt` (horizontal side bar, no read of the setting), class `JDt` (vertical side bar, reads it), `isActivityBarHidden` (returns true when orientation is horizontal)

### AB-5.2 Move the activity bar from a menu or the Command Palette
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Run Move Activity Bar to Side, Move Activity Bar to Top, Move Activity Bar to Bottom or Hide Activity Bar from the Command Palette, or pick Default, Top, Bottom or Hidden in the Activity Bar Position submenu. The submenu appears in View > Appearance, in the rail icon right-click menu, in the view container context menu and in the Secondary Side Bar title menu, and the current choice shows a check. Each palette command is offered only when the bar is not already there.
- Commands: workbench.action.activityBarLocation.default; workbench.action.activityBarLocation.top; workbench.action.activityBarLocation.bottom; workbench.action.activityBarLocation.hide
- Menus: ActivityBarPositionMenu; MenubarAppearanceMenu (submenu entry, group 3_workbench_layout_move, order 2); ViewContainerTitleContext (submenu entry, group 3_workbench_layout_move, order 1); CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: missing (searched the bundle and both NLS files for the four command ids, `ActivityBarPositionMenu`, `Activity Bar Position`, `Move Activity Bar` and `Hide Activity Bar`: 0 hits; the Cursor module `vs/workbench/browser/parts/activitybar/activitybarPart` keeps only the keys menu, hideMenu, reloadForActivityBar, previousSideBarView, nextSideBarView, focusActivityBar)
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 451-478 (rail menu submenu), 480-601 (four commands and both submenu entries); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts 244-258; V\src\vs\platform\actions\common\actions.ts 134 (menu id); C bundle: NLS module `vs/workbench/browser/parts/activitybar/activitybarPart` in `C\out\nls.keys.json`

### AB-5.3 See the activity bar as a strip at the top or bottom of the side bar
- Tag: both
- Evidence: source
- Boundary: no
- What: With location top or bottom the 48 px rail is not drawn. The same container icons (16 px, 30 px overflow slot) show as a horizontal strip in the Primary Side Bar title (top) or along its bottom edge (bottom), using the `activityBarTop.*` colors, and a right-click on the strip adds a Views submenu. In stock, Accounts and Manage also move to the title bar (owned by AB-4) and the custom title bar is never treated as empty.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: changed (the strip is drawn only by the old stock side bar class that loads when orientation is `vertical`; the default horizontal layout never draws it; Accounts and Manage do not move to the title bar because the title bar's activity actions are switched off)
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 197-232 (strip options), 234-273 (when it shows and where); V\src\vs\workbench\browser\parts\paneCompositePart.ts 793-805 (Views submenu); V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts 884-887; V\src\vs\workbench\services\layout\browser\layoutService.ts 590-612 (title bar never empty); C bundle: class `JDt` methods `shouldShowCompositeBar` and `getCompositeBarPosition`, title bar `get activityActionsEnabled(){return!1}`, `getDummyActivityBarView` (zero-size stand-in in horizontal mode)

### AB-5.4 Hide the activity bar
- Tag: both
- Evidence: source
- Boundary: no
- What: Choose Hidden and no rail or strip is drawn for the Primary or Secondary Side Bar, and the Accounts and Manage icons go with it. Reach it from the setting, the Hide Activity Bar command or the Hidden entry in the Activity Bar Position submenu. The layout state key `activityBar.hidden` follows the setting, and in stock a change to it writes `hidden` back to the setting.
- Commands: workbench.action.activityBarLocation.hide
- Menus: ActivityBarPositionMenu; CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location; workbench.activityBar.visible (legacy, migrated to location)
- Cursor: changed (hiding by the setting works in the vertical layout; the Hide Activity Bar command and the Hidden menu entry are missing; in the default horizontal layout there is no rail, so nothing is left to hide)
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 558-582; V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 255-262; V\src\vs\workbench\browser\layout.ts 1361-1362, 2296-2297, 2851, 2940-2941, 3154-3160; C bundle: `setActivityBarHidden`, `isActivityBarHidden`, `updateLegacySettingsFromState` (writes `hidden` back to the setting)

### AB-5.5 Toggle Activity Bar Visibility
- Tag: both
- Evidence: source
- Boundary: no
- What: Run Toggle Activity Bar Visibility to hide the bar, and run it again to bring it back where it was last visible (side, top or bottom, remembered in profile storage). The Customize Layout picker has an Activity Bar eye toggle that runs the same command. The command has no Command Palette entry, no menu item and no default key in stock.
- Commands: workbench.action.toggleActivityBarVisibility
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.location
- Cursor: changed (the command id and title are kept and work as stock in the vertical layout; the Customize Layout picker has no Activity Bar item, its list is Menu Bar, Primary Side Bar, Secondary Side Bar, Panel and Status Bar; in the default horizontal layout the command clears `workbench.activityBar.location` and shows an info dialog "Please quit and reopen for the activity bar to be displayed vertically.")
- Source: V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 275-290 (remember last visible position), 321-335 (command); V\src\vs\workbench\browser\actions\layoutActions.ts 62 (id), 1360-1370 (picker items); C bundle: command `workbench.action.toggleActivityBarVisibility` (NLS 3985, module `vs/workbench/browser/parts/sidebar/sidebarPart` key toggleActivityBar), dialog text (NLS 3301, key reloadForActivityBar), picker list built next to `config.workbench.sideBar.location` (no activity bar entry)

### AB-5.6 Switch the activity bar between default and compact size
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Turn on `workbench.activityBar.compact`, or pick Compact in the Activity Bar Size submenu of the rail right-click menu, for a narrower rail (36 px wide, 32 px items, 16 px icons) instead of 48 px, 48 px and 24 px. The submenu shows only when location is default, and the rail is rebuilt at once. The two size entries are built in code and are not registered commands.
- Commands: workbench.action.activityBar.size.default; workbench.action.activityBar.size.compact
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.compact; workbench.activityBar.location
- Cursor: missing (searched for the setting id, `workbench.action.activityBar.size`, `Activity Bar Size`, `activityBarCompact` and a compact class: 0 hits; the Cursor rail is fixed at 48 px wide, 48 px items, 24 px icons, item size 52)
- Source: V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts 44-52 (sizes), 95-120 (live switch), 456-466 (submenu); V\src\vs\workbench\browser\workbench.contribution.ts 669-674; V\src\vs\workbench\test\browser\parts\activitybar\activitybarPart.test.ts; C bundle: class `ore` (vertical rail, fixed sizes)

### AB-5.7 Auto-hide the activity bar when only one container shows
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With `workbench.activityBar.autoHide` on and location top or bottom, the strip disappears when the side bar has a single visible view container, and comes back when a second one appears. It applies to both the Primary and Secondary Side Bars.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.autoHide; workbench.activityBar.location
- Cursor: missing (searched for `activityBar.autoHide`, `activityBarAutoHide` and the visible-containers tracker class: 0 hits; Cursor's `shouldShowCompositeBar` only tests for top or bottom)
- Source: V\src\vs\workbench\browser\workbench.contribution.ts 663-668; V\src\vs\workbench\browser\parts\sidebar\sidebarPart.ts 131-143, 234-253; V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts 130-161, 262-283; C bundle: class `JDt` method `shouldShowCompositeBar`

### AB-5.8 Hide the activity bar when Zen Mode turns on
- Tag: both
- Evidence: source
- Boundary: no
- What: With `zenMode.hideActivityBar` on (the default), entering Zen Mode hides the activity bar, changing the setting while in Zen Mode shows or hides it at once, and leaving Zen Mode brings it back unless it was already hidden.
- Commands: workbench.action.toggleZenMode
- Menus: none found
- Keys: none found
- Settings: zenMode.hideActivityBar
- Cursor: changed (setting kept with default true and the same description; its change handler is an older copy that does not keep the strip hidden for top or bottom; with the default horizontal layout there is no rail, so it has no visible effect)
- Source: V\src\vs\workbench\browser\workbench.zenMode.contribution.ts 56-59, 102; V\src\vs\workbench\browser\layout.ts 1480-1481, 1509-1512, 1564-1565; V\src\vs\workbench\services\layout\browser\layoutService.ts 37; C bundle: `zenMode.hideActivityBar` definition (NLS 4316) and the Zen Mode configuration listener that calls `setActivityBarHidden`

### AB-5.9 See smaller badges in the compact layouts
- Tag: both
- Evidence: source
- Boundary: no
- What: Badges shrink where the bar is compact. On a compact rail (`workbench.activityBar.compact` on) the number pill is 13 px tall and the profile text on the gear is 8 px, instead of 16 px and 9 px. On the top or bottom strip the badge uses the `compact` class, and a number of three or more characters adds `compact-content` with an 8 px font. Progress badges on the strip use a 13 px mask.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.compact
- Cursor: changed (the strip badge styles are kept and used by the vertical layout; the compact rail styles are missing along with the setting, 0 hits for `activitybar.compact` in the Cursor CSS; the default horizontal row draws no badges)
- Source: V\src\vs\workbench\browser\parts\compositeBarActions.ts 325-350; V\src\vs\workbench\browser\parts\activitybar\media\activityaction.css 158-205; V\src\vs\workbench\browser\parts\media\paneCompositePart.css 235-262; C bundle: `this.options.compact&&s.length>=3&&i.push("compact-content")`, CSS `badge.compact` rules in `workbench.desktop.main.css`, class `JDt` option `compact:!0`

## Handed off
- Activity bar position on the Secondary Side Bar (the Activity Bar Position submenu in its title menu, its top strip, `workbench.secondarySideBar.showLabels`) goes to AX-4.
- `workbench.activityBar.iconClickBehavior`, and Focus Activity Bar bringing a hidden bar back at its remembered position, go to AB-1.
- Accounts and Manage moving into the title bar when location is top or bottom goes to AB-4.
- The title bar counting as non-empty for top or bottom, and `window.customTitleBarVisibility` flipping from never to auto when the bar moves there (V layout.ts 420-430), go to TB-3.
- The Customize Layout picker itself (its Activity Bar eye toggle is an entry point of AB-5.5, and its Reset button also resets `workbench.activityBar.location`) goes to TB-4.
- The Views submenu on the top or bottom strip right-click menu (view show and hide toggles) goes to SB-1.
- Move Primary Side Bar Right and Left and Hide Primary Side Bar in the rail menu (`workbench.action.toggleSidebarPosition`, `workbench.action.toggleSidebarVisibility`), and `workbench.view.alwaysShowHeaderActions`, go to SB-1.
- Menu and Hide Menu in the rail menu (the compact `window.menuBarVisibility` hamburger button) go to TB-1.
- Activity bar theme colors (`activityBar.*`, `activityBarTop.*`, badge colors) have no group of their own here and go to AB-1.

## Unclear
- `entry-points\01-title-bar.md` says the four `workbench.action.activityBarLocation.*` commands are "still there" in the Cursor bundle. My searches of the bundle and both NLS files found 0 hits for those ids, the four titles, `ActivityBarPositionMenu` and `Activity Bar Position`. I treat them as missing. Pass 1's Cursor file agrees with that.
- Everything on the Cursor side is read from strings and code. Nothing was run. In particular, how a vertical Cursor layout behaves with location top or bottom (strip drawn, no Accounts or Manage anywhere) is derived from the code.
- The Cursor bundle has code that skips `workbench.activityBar.location` when its value is `top` while importing VS Code settings. I did not follow it; it looks like onboarding and is not a feature of this group.
- Zen Mode has no catalog group of its own. AB-5.8 covers only the activity bar part of it.
- `workbench.experimental.modernUI` (floating panels and a rail gutter) is not a feature here. The stock pass 1 file lists it as left out. Cursor has 0 hits for it. In VS Code main (1.144) the compact rail items are 28 px instead of 32 px; this is a size change only, not a new feature.
- AB-3 handed the compact badge look to this group and described `compact-content` as a compact rail style. In the source, `compact` and `compact-content` belong to the top or bottom strip (it passes `compact: true`), and the compact rail uses separate `.activitybar.compact` rules. AB-5.9 covers both.
- Pixel sizes come from source constants and CSS, not from a screenshot.
