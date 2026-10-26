Group: TB-3 Window frame and controls. Section: Title bar (TB). Date: 2026-10-25. Pass: 2.
Features: 23.

### TB-3.1 Choose a native or custom title bar
- Tag: both
- Evidence: source
- Boundary: no
- What: `window.titleBarStyle` picks the OS title bar (native) or one drawn by the workbench (custom, the default) that carries the menu bar, command center, toolbar buttons and window controls. Changes need a full restart. On macOS, native tabs or turning native full screen off force the native style. In stock the style also decides whether menus and context menus are drawn by the app or the OS (through `window.menuStyle`, owned by TB-1).
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.titleBarStyle
- Cursor: changed - present with default custom; the setting text is older (it says the style also changes the menu look) and `window.menuStyle` is not in the bundle
- Source: V\src\vs\platform\window\common\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; C bundle (module `vs/workbench/electron-sandbox/desktop.contribution`, string 12061; settings block default `custom`)

### TB-3.2 Choose native, drawn or hidden window controls
- Tag: both
- Evidence: source
- Boundary: no
- What: Windows and Linux only, with the custom title bar. `native` (default) lets the OS draw minimize, maximize and close over the bar (the window controls overlay); the bar sends the OS its background color, text color and height so they match the theme and the zoom. `custom` has the workbench draw them (TB-3.3). `hidden` removes them. Changes need a full restart. The setting is not offered on macOS and is ignored with a native title bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.controlsStyle
- Cursor: present - setting registered with default `native`, hidden on macOS; the `updateWindowControls` call is in the bundle
- Source: V\src\vs\platform\window\common\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; C bundle (settings block, title bar part, `updateWindowControls` 6 hits)

### TB-3.3 Minimize, maximize or restore, and close with the drawn buttons
- Tag: both
- Evidence: source
- Boundary: no
- What: With `custom` controls, three 46 px buttons sit at the right end of the bar on Windows and Linux: minimize, maximize (which becomes restore while maximized), and close (red on hover). They are hidden in full screen, keep a fixed pixel size when the window is zoomed, and tint on hover to suit light or dark title bars.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.controlsStyle
- Cursor: present - classes `window-minimize`, `window-max-restore`, `window-close` and the red close hover rule are in the bundle and the CSS
- Source: V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)

### TB-3.4 Close the window by keyboard or File menu
- Tag: both
- Evidence: source
- Boundary: no
- What: Close Window closes the active window (main or floating). On macOS, Cmd+W also closes the window when no editor is open and there is one editor group. The confirm prompt belongs to DL-2.
- Commands: workbench.action.closeWindow
- Menus: MenubarFileMenu (group 6_close, order 4); CommandPalette
- Keys: Alt+F4 and Ctrl+Shift+W (Windows and Linux); Cmd+Shift+W (macOS); Cmd+W (macOS, when no editor is visible and there is a single group)
- Settings: none found
- Cursor: present - command id found (3 hits), menu title `miCloseWindow` in module `electron-sandbox/actions/windowActions`
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (module `vs/workbench/electron-sandbox/actions/windowActions`, string 12018)

### TB-3.5 macOS window buttons at the left
- Tag: both
- Evidence: source
- Boundary: no
- What: On macOS with the custom title bar, the traffic light buttons stay at the left (the OS draws them) and move to the right in right-to-left locales. The workbench adds no container for them, so dragging works right up to the buttons. Bar height is 28 px, 32 px on macOS Tahoe or newer, or 35 px with the command center.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - the left or right placement code with the right-to-left check is in the title bar part; the 28 and 32 px heights were not searched
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\platform\window\common\window.ts; C bundle (title bar part, `textInfo` direction check)

### TB-3.6 App icon at the left of the title bar
- Tag: both
- Evidence: source
- Boundary: no
- What: With a custom title bar on Windows and Linux, an app icon 35 px wide sits at the far left, before the menu bar. It is hidden in full screen. The only behavior in code is the double-click in TB-3.7.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - `window-appicon` element and CSS found, including the full-screen hide rule
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)

### TB-3.7 Double-click the app icon to close the window
- Tag: both
- Evidence: source
- Boundary: no
- What: With `window.doubleClickIconToClose` on, a double-click on the app icon closes the window and the icon can no longer be used to drag the window. Only works with the custom title bar.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.doubleClickIconToClose
- Cursor: present - setting found (3 hits, default false)
- Source: V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (settings block, title bar part)

### TB-3.8 Drag the title bar to move the window
- Tag: both
- Evidence: source
- Boundary: no
- What: A full-size drag region sits behind the bar, so dragging empty space moves the window, while buttons, the menu bar and the command center are no-drag. The region is hidden briefly while the Alt-toggled menu bar shows or has focus on Windows and Linux (an Electron issue, #52522). The workbench has no double-click handler on the bar itself, so double-click to maximize is left to the OS.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - `titlebar-drag-region` found in the bundle and the CSS
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)

### TB-3.9 Resize the window from the top edge of the custom frame
- Tag: both
- Evidence: source
- Boundary: no
- What: On Windows and Linux with drawn window controls, a 4 px no-drag strip at the top of the bar lets the pointer grab the window's top edge to resize it. It is hidden while the window is maximized and in full screen.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.controlsStyle
- Cursor: present - `div.resizer` created in the title bar part and `.resizer` rules in the CSS
- Source: V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS)

### TB-3.10 Hide or show the custom title bar
- Tag: vscode
- Evidence: source
- Boundary: no
- What: With the native title bar on, `window.customTitleBarVisibility` set to `never` removes the custom bar (menu bar, command center, toolbar buttons) and `auto` shows it. The setting flips back from `never` to `auto` when something that lives in the bar is turned on (command center, layout controls, activity bar at top or bottom, editor actions in the title bar). The right-click row shows only with the native style; the Appearance menu row is owned by TB-1's menu.
- Commands: showCustomTitleBar; hideCustomTitleBar; toggle.toggleCustomTitleBar; toggle.window.customTitleBarVisibility
- Menus: MenubarAppearanceMenu (group 2_workbench_layout, order 6); TitleBarContext; TitleBarTitleContext (group 3_toggle, order 0)
- Keys: none found
- Settings: window.customTitleBarVisibility
- Cursor: missing - no hits for the setting or for `showCustomTitleBar`, `hideCustomTitleBar` or `toggle.toggleCustomTitleBar`
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; V\src\vs\workbench\browser\layout.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (searched, none found)

### TB-3.11 Hide the custom title bar in full screen only
- Tag: vscode
- Evidence: source
- Boundary: no
- What: Setting `window.customTitleBarVisibility` to `windowed` hides the custom bar in full screen and keeps normal behavior otherwise. The command and the right-click row are offered only while in full screen. The visibility check returns early for this value, so the bar stays hidden in full screen even if the menu bar is toggled on.
- Commands: hideCustomTitleBarInFullScreen; toggle.window.customTitleBarVisibility.windowed
- Menus: TitleBarContext; TitleBarTitleContext (group 3_toggle, order 1, when `isFullscreen`)
- Keys: none found
- Settings: window.customTitleBarVisibility
- Cursor: missing - no hits for the setting or either id
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarActions.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; V\src\vs\workbench\browser\layout.ts; C bundle (searched, none found)

### TB-3.12 Restore the native title bar (Cursor)
- Tag: cursor
- Evidence: bundle
- Boundary: no
- What: A row on the title bar right-click menu sets `window.titleBarStyle` back to `native`. It shows only while the style is custom. By the two platform flags that guard it, it is registered on Linux desktop only: they match isLinux and isNative in the same bundle (compare the `included` rule of `window.menuBarVisibility` and the macOS right-to-left check).
- Commands: toggle.window.titleBarStyle
- Menus: TitleBarContext; TitleBarTitleContext (group 4_restore_native_title, order 0, when the `titleBarStyle` context key is `custom`)
- Keys: none found
- Settings: window.titleBarStyle
- Cursor: cursor only
- Source: C bundle (module `vs/workbench/browser/parts/titlebar/titlebarActions`, string 4021, registration guarded by the two platform flags)

### TB-3.13 Toggle full screen
- Tag: both
- Evidence: source
- Boundary: no
- What: Switch the window in and out of full screen. In full screen the app icon, window controls and top resize strip are hidden, and with the classic menu bar mode the custom title bar hides until the menu bar is toggled (macOS hides the title bar in full screen). With `window.restoreFullscreen` on, a window that was in full screen when closed reopens in full screen.
- Commands: workbench.action.toggleFullScreen
- Menus: MenubarAppearanceMenu (group 1_toggle_view, order 1); CommandPalette
- Keys: F11; Ctrl+Cmd+F (macOS)
- Settings: window.restoreFullscreen; window.customTitleBarVisibility
- Cursor: present - command id found (5 hits), title and menu title strings 3256 and 3263, `window.restoreFullscreen` found; `window.customTitleBarVisibility` is missing in Cursor
- Source: V\src\vs\workbench\browser\actions\windowActions.ts; V\src\vs\workbench\services\layout\browser\layoutService.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (module `vs/workbench/browser/actions/windowActions`)

### TB-3.14 Use macOS native full screen
- Tag: both
- Evidence: source
- Boundary: no
- What: macOS only. `window.nativeFullScreen` (default on) uses the macOS full screen Space. Turning it off uses simple full screen without a new Space, and that forces the native title bar style. Native window tabs force native full screen on.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.nativeFullScreen
- Cursor: present - setting found (2 hits, default true, macOS only)
- Source: V\src\vs\platform\window\common\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; C bundle (settings block, string 12065)

### TB-3.15 macOS native window tabs
- Tag: both
- Evidence: source
- Boundary: no
- What: macOS only. With `window.nativeTabs` on, windows join a shared native tab bar and the native title bar style is forced. A restart is needed. The six tab commands show in the Command Palette only while the setting is on.
- Commands: workbench.action.newWindowTab; workbench.action.showPreviousWindowTab; workbench.action.showNextWindowTab; workbench.action.moveWindowTabToNewWindow; workbench.action.mergeAllWindowTabs; workbench.action.toggleWindowTabsBar
- Menus: CommandPalette (when `config.window.nativeTabs` is true)
- Keys: none found
- Settings: window.nativeTabs
- Cursor: present - all six ids and the setting found, titles in strings 12089-12094
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\platform\window\common\window.ts; C bundle (module `vs/workbench/electron-sandbox/desktop.contribution`)

### TB-3.16 Zoom the window in, out and reset
- Tag: both
- Evidence: source
- Boundary: no
- What: Zoom In, Zoom Out and Reset Zoom scale the whole window by one step each (a step is 20 percent). By default only the active window changes. With `window.zoomPerWindow` off the level applies to all windows and is saved in `window.zoomLevel`. Reset returns to the configured default level (or 0 when applying to all windows). The title bar counter-scales so it keeps its pixel size when it holds only non-zoomable content, and it never shrinks below the window controls when zoomed out. The status bar zoom entry is owned by SS (see Handed off).
- Commands: workbench.action.zoomIn; workbench.action.zoomOut; workbench.action.zoomReset
- Menus: MenubarAppearanceMenu (group 5_zoom, orders 1, 2 and 3); CommandPalette
- Keys: Zoom In - Ctrl+= (also Ctrl+Shift+= and Ctrl+Numpad+); Zoom Out - Ctrl+- (also Ctrl+Shift+- and Ctrl+Numpad-; on Linux Ctrl+- and Ctrl+Numpad-); Reset Zoom - Ctrl+Numpad0; Cmd replaces Ctrl on macOS
- Settings: window.zoomLevel; window.zoomPerWindow
- Cursor: present - all three ids, both settings and the `counter-zoom` class found; Cursor adds a hidden `workbench.action.setZoomLevel` (no menu, no palette entry) that its Window in Window feature calls
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (module `vs/workbench/electron-sandbox/actions/windowActions`, strings 12019-12034)

### TB-3.17 Title bar colors and dimming when the window loses focus
- Tag: both
- Evidence: source
- Boundary: no
- What: The bar takes its colors from the `titleBar.*` theme colors (active and inactive background and foreground, and border). When the window loses focus the bar switches to the inactive colors and drops to 60 percent opacity, and the command center and window control icons follow. Light backgrounds add a `light` class that changes the control hover tint.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - `titleBar.activeBackground` and its siblings are in the built-in themes and the `part.titlebar.inactive` rules are in the CSS
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part, workbench CSS, theme data)

### TB-3.18 Window border color (Windows)
- Tag: vscode
- Evidence: source
- Boundary: no
- What: On Windows, `window.border` sets the window's border color: `default` uses the theme colors `window.activeBorder` and `window.inactiveBorder` and falls back to the system color, `system` follows Windows only, `off` removes it, or a color value sets it. It is ignored with the native title bar and is refreshed when full screen changes.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.border
- Cursor: missing - no hits for `window.border` or `updateWindowAccentColor`; only the theme color id `window.activeBorder` appears (1 hit)
- Source: V\src\vs\workbench\electron-browser\window.ts; V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\browser\layout.ts; C bundle (searched, only the color id found)

### TB-3.19 Click through an inactive window (macOS)
- Tag: both
- Evidence: source
- Boundary: no
- What: macOS only. With `window.clickThroughInactive` on (default), the first click on an inactive window both activates it and presses whatever is under the pointer. Off means the first click only activates. Changing it prompts for a restart.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: window.clickThroughInactive
- Cursor: present - setting found (2 hits, default true, macOS only)
- Source: V\src\vs\workbench\electron-browser\desktop.contribution.ts; V\src\vs\workbench\contrib\relauncher\browser\relauncher.contribution.ts; V\src\vs\platform\windows\electron-main\windows.ts; C bundle (settings block, string 12066)

### TB-3.20 Pin a floating editor window on top
- Tag: vscode
- Evidence: source
- Boundary: no
- What: In a floating editor window (auxiliary window), a pin button on the layout controls turns Always on Top on, and a pinned icon turns it off. The palette command toggles it. The main window cannot be pinned.
- Commands: workbench.action.toggleWindowAlwaysOnTop; workbench.action.enableWindowAlwaysOnTop; workbench.action.disableWindowAlwaysOnTop
- Menus: LayoutControlMenu (group navigation, order 1, auxiliary windows only); CommandPalette
- Keys: none found
- Settings: none found
- Cursor: missing - none of the three ids is in the bundle
- Source: V\src\vs\workbench\electron-browser\actions\windowActions.ts; V\src\vs\workbench\electron-browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\services\auxiliaryWindow\electron-browser\auxiliaryWindowService.ts; C bundle (searched, none found)

### TB-3.21 Compact mode for floating editor windows
- Tag: vscode
- Evidence: source
- Boundary: no
- What: A floating editor window can switch to compact mode, which trims its title bar to a smaller set of buttons: no command center, no global actions, no Accounts or Manage, and a separate editor title menu. A button on the layout controls and a palette command turn it on and off, and the `isCompactTitleBar` context key follows.
- Commands: workbench.action.toggleCompactAuxiliaryWindow; workbench.action.enableCompactAuxiliaryWindow; workbench.action.disableCompactAuxiliaryWindow
- Menus: LayoutControlMenu (group navigation, order 0, auxiliary windows only); CommandPalette; LayoutControlMenu
- Keys: none found
- Settings: none found
- Cursor: missing - none of the three ids and no `isCompactTitleBar` key is in the bundle
- Source: V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts; V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\common\contextkeys.ts; C bundle (searched, none found); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts (lines 50 to 105 for the actions, lines 174 to 185 for what compact changes); C bundle (none of the three ids found); V\src\vs\workbench\browser\parts\editor\auxiliaryEditorPart.ts:50-105; C bundle (searched the three ids and the labels Turn On Compact Mode and Turn Off Compact Mode, no hits)

### TB-3.22 Window controls overlay for installed web apps (web)
- Tag: both
- Evidence: source
- Boundary: no
- What: Web only. When the page runs as an installed app with the window controls overlay, the title bar reserves space for the browser's controls on the right, and on the left as well on macOS, using the overlay's size values, and the bar height follows the overlay height. Desktop Cursor never takes this branch.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: present - the `wco-enabled` class and the web branch of the container code are in the bundle, but Cursor ships as desktop only
- Source: V\src\vs\workbench\browser\parts\titlebar\titlebarPart.ts; V\src\vs\workbench\browser\parts\titlebar\media\titlebarpart.css; C bundle (title bar part)

### TB-3.23 No title bar layout (Cursor)
- Tag: cursor
- Evidence: bundle
- Boundary: yes - the layout lives in Cursor's agent-centered unified layout and the freed title row hosts agent tabs and moves layout buttons into the editor tab strip
- What: On macOS and Linux the page body gets the class `no-titlebar-layout`. The title bar becomes a transparent 34 px strip fixed at the top that passes clicks through except on its toolbars, the window title is hidden, the command center is forced off, and side bar, panel and auxiliary bar get about 35 px of top padding. The mode is driven by a stored preference `cursor/noTitlebarLayout.visibility` whose default is `hide`. It never applies on Windows, is off when the feature gate `disable_no_title_bar` is on, and is off when editor tabs are set to `none`. The context key `cursor.noTitlebarLayout.enabled` reports it.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Cursor: cursor only
- Source: C bundle (workbench CSS rules for `no-titlebar-layout`; layout module with the storage key, the gate and the platform check; title bar part `isCommandCenterVisible`)

## Handed off
- Menu style (`window.menuStyle`, stock only), the Menu Bar toggle (`workbench.action.toggleMenuBar`), Focus Title Bar (`workbench.action.focusTitleBar`), and the screen reader prompt that suggests the custom menu style (`menubar/accessibleMenubarNotified`, string `menubar.customTitlebarAccessibilityNotification`, present in Cursor with older text): TB-1.
- Dialog style (`window.dialogStyle`, native or custom dialogs): DL-1.
- Confirm before closing a window or quitting (`window.confirmBeforeClose` and the close prompts): DL-2.
- The status bar Window Zoom entry (`status.windowZoom`, zoom out, level, zoom in, Reset and Settings buttons, right side, priority 102, shown with a zoom-in or zoom-out icon only while the window zoom differs from the configured level; present in Cursor, strings 12138-12144): SS-1 is the closest group, but none fits exactly.
- The layout controls menu itself that hosts the pin and compact buttons for floating windows (`LayoutControlMenu`): TB-4 owns the menu; TB-3.20 and TB-3.21 own the two features.

## Unclear
- This group is large (23 features) because it covers the frame, three platforms, zoom and full screen. A later pass may split out macOS items (TB-3.5, TB-3.14, TB-3.15, TB-3.19) or the floating window items (TB-3.20, TB-3.21).
- TB-3.20 and TB-3.21 may also be claimed by TB-4, since their buttons sit on the layout controls menu of floating windows.
- Window lifecycle settings in the same settings block have no owner and are not title bar UI: `window.newWindowDimensions`, `window.closeWhenEmpty`, `window.restoreWindows`, `window.openFilesInNewWindow`, `window.openFoldersInNewWindow`, `window.openWithoutArgumentsInNewWindow`. Palette commands `workbench.action.closeOtherWindows` (not in the Cursor bundle), Switch Window and Quick Switch Window have no owner either.
- Cursor has CSS classes `window-in-window` and `wiw-show-title-bar` that hide the app icon and window controls, and it calls the hidden `workbench.action.setZoomLevel` with a forced zoom level from Window in Window code. I did not find what opens such a window or whether a user can reach it.
- I did not find a command, setting or menu row that flips the Cursor no title bar preference (TB-3.23). The default of `hide` and the gate come from the code, not from a visible control.
- Cursor was not run. Everything on the Cursor side is from strings and minified code in the bundle. The macOS and Linux behaviors were not seen on those systems.
