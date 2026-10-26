Group: SS-2 Bar and item colors. Section: Status bar. Date: 2026-10-25. Pass: 2.
Features: 15.

### SS-2.1 Bar background, text and top line in a normal workspace
- Tag: both
- Evidence: source
- Boundary: no
- What: In a window with a folder or workspace open, the bar is painted with `statusBar.background` and `statusBar.foreground`, with a 1 px line along its top edge in `statusBar.border`, or in the high-contrast border color when the theme sets no border; the line is hidden while the bar has keyboard focus. The bar fades between background colors over 0.15 s when motion is on. Token defaults are `#007ACC` with white text and no border, which the shipped themes replace (stock Dark 2026 uses `#191A1B` with `#8C8C8C` text, Light 2026 uses `#FAFAFD` with `#606060`, Cursor Dark uses `#141414` with `#F0F0F099`).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (same tokens, same defaults and the same `updateStyles` code; the shipped theme values differ, see SS-2.13)
- Source: V\src\vs\workbench\common\theme.ts:284-319 (foreground, background, border tokens); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:656-682 (updateStyles, top line class); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (motion fade, `status-border-top` line); V\extensions\theme-defaults\themes\2026-dark.json:176-189 and 2026-light.json:181-194; V\src\vs\workbench\services\themes\common\workbenchThemeService.ts:42-45 (default themes are Dark 2026 and Light 2026); C bundle (tokens `statusBar.background` `#007ACC`, `statusBar.foreground`, `statusBar.border`, class set in `updateStyles`); C\extensions\theme-cursor\themes\cursor-dark-color-theme.json

### SS-2.2 Bar colors when no folder is open
- Tag: both
- Evidence: source
- Boundary: no
- What: A window with no folder or workspace open uses `statusBar.noFolderBackground`, `statusBar.noFolderForeground` and `statusBar.noFolderBorder` instead, and the bar switches over as soon as a folder is opened or closed. The token default background is purple `#68217A` (foreground and border follow the normal ones). Stock Dark 2026 sets no-folder equal to the normal bar, Dark Modern uses `#1F1F1F` against `#181818`, Light 2026 uses `#F0F0F3` against `#FAFAFD`, and the plain Dark and Light (Visual Studio) themes still show the purple.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (tokens and the same purple default are in the bundle; Cursor's own themes set no-folder equal to the normal bar, see SS-2.13)
- Source: V\src\vs\workbench\common\theme.ts:291, :300-305, :321; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:220, :663-675 (state switch on `WorkbenchState.EMPTY`); V\extensions\theme-defaults\themes\dark_modern.json:97, light_modern.json:113, 2026-dark.json:182-183, 2026-light.json:187-188, dark_vs.json and light_vs.json (no no-folder keys); C bundle (`statusBar.noFolderBackground` registration with `#68217A`, `onDidChangeWorkbenchState` handler)

### SS-2.3 Bar colors while debugging
- Tag: both
- Evidence: source
- Boundary: no
- What: While a debug session is running the whole bar changes to `statusBar.debuggingBackground`, `statusBar.debuggingForeground` and `statusBar.debuggingBorder`, and goes back when the session ends. It does not apply to sessions started without debugging, to sessions that ask to hide the debug status bar, or while the debugger is still starting. It overrides the no-folder colors too. Token default is orange `#CC6633`; stock Dark Modern uses `#0078D4`, Dark 2026 `#3994BC`, and Light Modern `#FD716C` with black text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the provider and the three tokens are in the bundle; Cursor's own themes tone it down, see SS-2.13)
- Source: V\src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts:20-34 (tokens), :44-107 (override with priority 10, state test); V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:714-724 (overrideStyle); V\extensions\theme-defaults\themes\dark_modern.json:93-94, light_modern.json:110-111, 2026-dark.json:180-181; C bundle (`wsg=Ft("statusBar.debuggingBackground"` with `#CC6633`, `overrideStyle({priority:10,...})`, `suppressDebugStatusbar` read in the debug session class)

### SS-2.4 Turn the debugging color on or off
- Tag: both
- Evidence: source
- Boundary: no
- What: The setting "Debug: Enable Status Bar Color" (description "Color of the status bar when the debugger is active", default on) decides whether SS-2.3 happens; switching it takes effect at once, even in a running session. It is a plain boolean in the Settings editor under Debug.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: debug.enableStatusBarColor
- Cursor: present (setting found with default true)
- Source: V\src\vs\workbench\contrib\debug\browser\debug.contribution.ts:689-693; V\src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts:75-91; V\src\vs\workbench\contrib\debug\common\debug.ts:826; C bundle (`"debug.enableStatusBarColor":{type:"boolean",description:k(6160,null),default:!0}`)

### SS-2.5 Prominent items stand out
- Tag: both
- Evidence: source
- Boundary: no
- What: An item marked prominent gets its own background and text from `statusBarItem.prominentBackground` and `statusBarItem.prominentForeground`, and its own hover pair `prominentHoverBackground` and `prominentHoverForeground` (the hover pair is skipped when the item sits in a compact pair). Token default is a black 50 percent layer with the bar's text color; stock Dark Modern uses `#6E768166` and Dark 2026 uses `#3994BC`. In stock the items that use it are Window Zoom, Restricted Mode, Screen Reader Optimized, Scrolling Locked, Tab Moves Focus, OVR, Column Selection, the notebook Select Kernel item and the Show Output quiet item.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the four tokens, the CSS and nine `kind:"prominent"` entries are in the bundle; Cursor's own themes use a faint tint, for example `#F0F0F011` in Cursor Dark)
- Source: V\src\vs\workbench\common\theme.ts:353-359; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (prominent rules and the compact exception); V\src\vs\workbench\browser\parts\statusbar\statusbarItem.ts:183-194 (kind class); V\src\vs\workbench\electron-browser\window.ts:1245; V\src\vs\workbench\contrib\workspace\browser\workspace.contribution.ts:686; V\src\vs\workbench\browser\parts\editor\editorStatus.ts:452, :471, :489; V\src\vs\workbench\contrib\accessibility\browser\accessibilityStatus.ts:90; V\src\vs\workbench\contrib\scrollLocking\browser\scrollLocking.ts:175; V\src\vs\workbench\contrib\notebook\browser\contrib\editorStatusBar\editorStatusBar.ts:173; V\src\vs\workbench\api\browser\mainThreadOutputService.ts:101; C bundle (`statusBarItem.prominentBackground` with `Pi.black.transparent(.5)`, `.prominent-kind` CSS)

### SS-2.6 Warning items turn amber
- Tag: both
- Evidence: source
- Boundary: no
- What: An item marked warning is drawn with `statusBarItem.warningBackground` and `statusBarItem.warningForeground`, and the hover pair `warningHoverBackground` and `warningHoverForeground`. Background default is the editor warning color darkened by 40 percent with white text. In stock the Problems visibility item, the Pty Host item and a language status entry with warning severity use it, and an extension item that asks for the warning background color is mapped to this kind.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (tokens, CSS and two `kind:"warning"` entries are in the bundle; I did not check which items they belong to)
- Source: V\src\vs\workbench\common\theme.ts:374-385; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (warning rules); V\src\vs\workbench\contrib\markers\browser\markers.contribution.ts:638; V\src\vs\workbench\contrib\terminal\browser\baseTerminalBackend.ts:70; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:414-418; V\src\vs\workbench\api\browser\statusBarExtensionPoint.ts:97-104 and V\src\vs\workbench\api\common\extHostStatusBar.ts:23-26; C bundle (`statusBarItem.warningBackground` registration, two `kind:"warning"` entries)

### SS-2.7 Error items turn red
- Tag: both
- Evidence: source
- Boundary: no
- What: An item marked error is drawn with `statusBarItem.errorBackground` and `statusBarItem.errorForeground`, and the hover pair `errorHoverBackground` and `errorHoverForeground`. Background default is the editor error color darkened by 40 percent with white text. In stock the Performance Trace recording item and a language status entry with error severity use it, and an extension item that asks for the error background color is mapped to this kind.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (tokens and CSS are in the bundle; the Performance Trace item that stock uses is missing from Cursor, see SS-5)
- Source: V\src\vs\workbench\common\theme.ts:361-372; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (error rules); V\src\vs\workbench\electron-browser\actions\developerActions.ts:181; V\src\vs\workbench\contrib\languageStatus\browser\languageStatus.ts:414-418; V\src\vs\workbench\api\browser\statusBarExtensionPoint.ts:97-104; C bundle (`statusBarItem.errorBackground` registration with `X2(Iwn,.4)`, `.error-kind` CSS)

### SS-2.8 Remote indicator color
- Tag: both
- Evidence: source
- Boundary: no
- What: The remote indicator at the far left is drawn with the remote kind colors, `statusBarItem.remoteBackground` and `statusBarItem.remoteForeground` plus the hover pair `remoteHoverBackground` and `remoteHoverForeground`; the defaults copy the activity bar badge colors, and Dark Modern sets `#0078D4` with white text. In stock the kind is applied only when the window is connected to a remote, so a plain local window shows an unstyled item. Cursor applies it all the time unless the machine is offline, so these tokens always show on the indicator.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: changed (the bundle sets `kind:this.networkState==="offline"?"offline":"remote"` with no check for a remote connection, and does not have the default-label test that stock uses)
- Source: V\src\vs\workbench\common\theme.ts:576-587; V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts:563-581 (line 568 picks the kind); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (remote rules); V\extensions\theme-defaults\themes\dark_modern.json:100-101, hc_black.json:12; C bundle (remote indicator entry `kind:...?"offline":"remote"`, token `statusBarItem.remoteBackground` equal to the activity bar badge token)

### SS-2.9 Offline indicator color
- Tag: both
- Evidence: source
- Boundary: no
- What: When the system reports that the network is offline, the remote indicator changes to the offline kind, with `statusBarItem.offlineBackground` (default dark red `#6c1717`), `statusBarItem.offlineForeground` (the remote foreground), and the hover pair `offlineHoverBackground` and `offlineHoverForeground`. It returns to the remote kind or the normal look when the network comes back.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (tokens, CSS and the `"offline"` choice in the remote indicator entry are in the bundle)
- Source: V\src\vs\workbench\common\theme.ts:589-600; V\src\vs\workbench\contrib\remote\browser\remoteIndicator.ts:102, :303-304, :433-437, :568; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (offline rules); C bundle (`statusBarItem.offlineBackground` with `#6c1717`, the remote indicator entry)

### SS-2.10 Item hover colors
- Tag: both
- Evidence: source
- Boundary: no
- What: While the pointer is over an item that has a command, its background becomes `statusBarItem.hoverBackground` and its text `statusBarItem.hoverForeground`; items of the colored kinds use their own hover pair instead (prominent, warning, error, remote, offline). The two halves of a compact pair light up together, and the half under the pointer uses `statusBarItem.compactHoverBackground`. High-contrast themes add a dashed outline in the contrast active border color and skip the compact effect. Defaults are white 12 percent on dark themes and black 12 percent on light themes, and the hover text follows the bar text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (the hover, hover text and compact hover tokens, the per-kind hover tokens and the hover CSS are in the bundle; the compact hover colors are set by `updateCompactEntries`, whose spacing differs, see SS-1.5)
- Source: V\src\vs\workbench\common\theme.ts:337-351; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:511-593 (compact hover colors, skipped when `isHighContrast`); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (the `a:hover:not(.disabled)` rules and the per-kind hover rules); C bundle (`statusBarItem.hoverBackground`, `statusBarItem.hoverForeground`, `statusBarItem.compactHoverBackground` found; hover rules in the bundle CSS)

### SS-2.11 Item pressed color
- Tag: both
- Evidence: source
- Boundary: no
- What: While the mouse button is held down on an item that has a command, its background changes to `statusBarItem.activeBackground` (default white 18 percent, black 18 percent in high contrast light), and high-contrast themes draw a solid outline in the contrast active border color instead of the fill.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present
- Source: V\src\vs\workbench\common\theme.ts:323-328; V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (`a:active:not(.disabled)` rules); C bundle (token `statusBarItem.activeBackground`, matching rules in the bundle CSS)

### SS-2.12 Keyboard focus outline
- Tag: both
- Evidence: source
- Boundary: no
- What: When the bar itself has keyboard focus it draws an outline in `statusBar.focusBorder`, and the top line is hidden to make room. When an item has focus by keyboard, it gets a 1 px solid outline in the theme's contrast active border color if it has one, otherwise in `statusBarItem.focusBorder`, set 2 px inside the item when the bar has a border and 1 px otherwise. On macOS, when the window is not full screen, the focused bar gets rounded bottom corners (10 px, 16 px on Tahoe) so the outline fits. In normal themes both focus tokens default to the bar's text color; Dark Modern sets `#0078D4`.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present (both tokens, the `:focus` and `:focus-visible` stylesheet written by `updateStyles`, and the macOS corner rules are in the bundle)
- Source: V\src\vs\workbench\common\theme.ts:314-319, :330-335; V\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:684-708 (dynamic stylesheet); V\src\vs\workbench\browser\parts\statusbar\media\statusbarpart.css (macOS corner rules, top line hidden on focus); V\extensions\theme-defaults\themes\dark_modern.json:95, :98; C bundle (`statusBar.focusBorder`, `statusBarItem.focusBorder`, `updateStyles` stylesheet text); C\out\vs\workbench\workbench.desktop.main.css (mac rules)

### SS-2.13 Cursor themes match the bar in no-folder windows and on the remote indicator
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: Cursor's seven themes set the no-folder bar background equal to the normal bar (Cursor Dark `#141414`, Cursor Light `#F3F3F3`, the High Contrast pair `#0A0A0A` and `#FFFFFF`, Midnight `#191c22`, and the two Colorblind themes the same as their parents) and set the remote indicator to the bar color too in Cursor Dark, Cursor Light, the two Colorblind themes and Light High Contrast, so no purple or blue block appears. The debugging look is a faint tint in Cursor Dark and its Colorblind twin (`#F0F0F01C`), a gray-blue shift (`#434C5E`) in Dark High Contrast and Midnight, and no change at all in the three light themes. Dark High Contrast and Midnight do not set the remote colors and fall back to the activity bar badge colors.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.colorTheme
- Cursor: cursor only
- Source: C\extensions\theme-cursor\themes\cursor-dark-color-theme.json; cursor-dark-colorblind-color-theme.json; cursor-dark-hc-color-theme.json; cursor-dark-midnight-color-theme.json; cursor-light-color-theme.json; cursor-light-colorblind-color-theme.json; cursor-light-hc-color-theme.json (keys `statusBar.*` and `statusBarItem.*`); C\extensions\theme-cursor\package.json (theme list); C bundle (default themes `Cursor Dark` and `Cursor Light`, `workbench.colorTheme` found)

### SS-2.14 Dimmed bar when the window is inactive
- Tag: vscode
- Evidence: source
- Boundary: no
- What: newer-than-1.128. A new token, `statusBar.inactiveBackground`, repaints the bar while the window has lost focus, in a window with a folder or workspace open and no debug color active. Its default is unset, so nothing changes until a theme sets it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: missing (0 hits for `statusBar.inactiveBackground` in the bundle)
- Source: N\src\vs\workbench\common\theme.ts:310-315; N\src\vs\workbench\browser\parts\statusbar\statusbarPart.ts:232-235, :732-734; N\src\vs\workbench\contrib\modernUI\README.md:209, :229; C bundle (searched, no match)

### SS-2.15 Recolor the bar yourself
- Tag: both
- Evidence: source
- Boundary: no
- What: There is no settings page for status bar colors. A person changes them by adding any of the 35 `statusBar.*` and `statusBarItem.*` keys (32 core tokens plus the three debugging ones) to `workbench.colorCustomizations` in settings.json, or by switching the theme.
- Commands: workbench.action.selectTheme
- Menus: none found
- Keys: none found
- Settings: workbench.colorCustomizations; workbench.colorTheme
- Cursor: present
- Source: V\src\vs\workbench\common\theme.ts:284-385, :576-600 (core tokens); V\src\vs\workbench\contrib\debug\browser\statusbarColorProvider.ts:20-34; C bundle (`workbench.colorCustomizations`, `workbench.colorTheme` and `workbench.action.selectTheme` found, all 35 token names found)

## Handed off

- The command center background while debugging (`commandCenter.debuggingBackground`, applied when `debug.toolBarLocation` is `commandCenter`): TB-2. It comes from the same provider as SS-2.3 and is not repeated here.
- The Color Theme picker and the theme list as a whole: left to whichever group owns Preferences > Themes. Only the status bar colors that a theme sets are covered here.
- Coloring that extension items can request (`color` and `backgroundColor` on `createStatusBarItem`, limited to the two error and warning background tokens): extension system, out of scope. The visible result is SS-2.6 and SS-2.7.
- Which item gets which kind (the Problems and Restricted Mode items, the language status severity, the remote indicator and so on): SS-3 to SS-5, the groups that own those items. This file covers only the colors.
- Colors of the notebook cell status bar, the suggest widget footer and hover footers: sections 05 and 06, not the window status bar.

## Unclear

- The first pass said stock "default" dark and light bars use `#68217A` with no folder open. That value is only the token default in `theme.ts`. Stock 1.128 ships Dark 2026 and Light 2026 as its default themes, and Dark 2026 sets no-folder equal to the normal bar, so purple shows only in themes that do not set it, such as Dark and Light (Visual Studio).
- The values for the seven Cursor themes come from the theme files, not from a running window. A theme file can be overridden by the user's `workbench.colorCustomizations`, which I did not look at. The first pass also read only Cursor Dark and Cursor Light.
- Which Cursor items use the two `kind:"warning"` and the many `kind:"error"` hits in the bundle (20 for error, most of them are probably not status bar entries). I confirmed only the tokens, the CSS and the remote indicator's use of the kinds.
- Cursor does not ship the stock Dark 2026 and Light 2026 themes (its `theme-defaults` folder has Dark Modern, Dark+, Dark (Visual Studio), the two High Contrast themes, Light Modern, Light+ and Light (Visual Studio)), so those stock default values are for the stock side only.
- The remote indicator note in SS-2.8 is read from one entry in the bundle; the in-between code that decides the text was not read, so a case where Cursor hides the indicator is not ruled out.
- Nothing was run on screen. The first pass saw a dark gray bar with no colored remote badge in the running window, which fits SS-2.13.

