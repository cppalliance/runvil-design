Group: TB-3 Window frame and controls. Pass: 3. Date: 2026-10-25.

### TB-3.1
1. Setting `window.titleBarStyle` chooses the title bar, with `custom` (default) drawn by the workbench to carry the menu bar, command center, toolbar buttons and window controls, and `native` drawn by the OS.
2. Changing `window.titleBarStyle` only takes effect after a full restart of the app.
3. On macOS, turning on `window.nativeTabs` or turning off `window.nativeFullScreen` forces the native title bar no matter what the setting says.
4. In stock, setting `window.menuStyle` to `inherit` makes menus and context menus follow the title bar style, app-drawn with `custom` and OS-drawn with `native`. [stock only]
5. In Cursor the `window.titleBarStyle` description still says the style also changes the menu look, and there is no `window.menuStyle` setting. [cursor only]

### TB-3.2
1. On Windows and Linux with the custom title bar, `window.controlsStyle` set to `native` (default) has the OS draw minimize, maximize and close over the right end of the bar.
2. Setting `window.controlsStyle` to `custom` has the workbench draw the three window buttons itself (TB-3.3).
3. Setting `window.controlsStyle` to `hidden` removes the window buttons from the bar.
4. With `native` controls, the bar passes the OS its background color, text color and height, so the OS-drawn buttons match the theme and follow the window zoom.
5. Changing `window.controlsStyle` only takes effect after a full restart of the app.
6. `window.controlsStyle` is not offered on macOS and is ignored when the title bar style is `native`.

### TB-3.3
1. With `window.controlsStyle` set to `custom`, three buttons 46 px wide (minimize, maximize, close) sit at the right end of the title bar on Windows and Linux.
2. Clicking the minimize button minimizes the window.
3. Clicking the maximize button maximizes the window and the icon turns into a restore icon, and clicking restore returns the window to its earlier size.
4. Clicking the close button closes the window, and the close button turns red when the pointer is over it.
5. The minimize and maximize buttons tint on hover in a way that suits a light or a dark title bar.
6. The three buttons are hidden while the window is in full screen.
7. The buttons keep a fixed pixel size when the window is zoomed in or out.

### TB-3.4
1. Running Close Window (`workbench.action.closeWindow`) closes the active window, whether it is the main window or a floating editor window.
2. The File menu has a Close Window row (group `6_close`), and the Command Palette lists the same command.
3. Alt+F4 and Ctrl+Shift+W close the window on Windows and Linux, and Cmd+Shift+W closes it on macOS.
4. On macOS, Cmd+W also closes the window when no editor is visible and there is a single editor group.

### TB-3.5
1. On macOS with the custom title bar, the traffic light buttons stay at the left end of the bar, drawn by the OS.
2. In right-to-left locales on macOS, the traffic light buttons move to the right end of the bar.
3. The workbench adds no container around the traffic light buttons, so dragging the bar works right up to them.
4. On macOS the bar is 28 px high, or 32 px high on macOS Tahoe or newer.
5. On macOS the bar is 35 px high when the command center is shown.

### TB-3.6
1. With the custom title bar on Windows and Linux, an app icon 35 px wide sits at the far left of the bar, before the menu bar.
2. The app icon is hidden while the window is in full screen.
3. The app icon only shows on Windows and Linux with the custom title bar, and not on macOS or with the native bar.

### TB-3.7
1. With `window.doubleClickIconToClose` on, double-clicking the app icon closes the window.
2. With `window.doubleClickIconToClose` on, the app icon can no longer be used to drag the window.
3. `window.doubleClickIconToClose` is off by default, so double-clicking the icon does nothing and the icon area drags the window like the rest of the bar.

### TB-3.8
1. Dragging empty space on the title bar moves the window.
2. Buttons, the menu bar and the command center do not start a window drag, so pressing them clicks them instead.
3. On Windows and Linux the drag area is turned off for a moment while the Alt-toggled menu bar is showing or has focus.
4. The workbench has no double-click handler on the bar, so double-clicking empty space does whatever the OS does for a title bar double-click, such as maximize.

### TB-3.9
1. On Windows and Linux with drawn window controls, a 4 px strip at the very top of the title bar lets the pointer grab the top edge of the window to resize it.
2. The top resize strip is hidden while the window is maximized.
3. The top resize strip is hidden while the window is in full screen.
4. The top resize strip exists only when `window.controlsStyle` is `custom`.

### TB-3.10
1. With the native title bar on, setting `window.customTitleBarVisibility` to `never` removes the custom bar (menu bar, command center, toolbar buttons), and `auto` (default) shows it. [stock only]
2. Show Custom Title Bar (`showCustomTitleBar`) and Hide Custom Title Bar (`hideCustomTitleBar`) in the Command Palette set the value to `auto` or `never`, and each is available only while the bar is hidden or shown. [stock only]
3. With the native style, right-clicking the title bar shows a Hide Custom Title Bar row (`toggle.window.customTitleBarVisibility`, group `3_toggle`), and the row is not offered with the custom style. [stock only]
4. The View, Appearance menu has a Custom Title Bar row (`toggle.toggleCustomTitleBar`) that shows a check while the bar is visible and flips it between shown and hidden. [stock only]
5. The setting flips back from `never` to `auto` when something that lives in the bar is turned on, such as the command center, layout controls, the activity bar at top or bottom, or editor actions in the title bar. [stock only]

### TB-3.11
1. Setting `window.customTitleBarVisibility` to `windowed` hides the custom title bar while in full screen and keeps the normal behavior in a regular window. [stock only]
2. Hide Custom Title Bar In Full Screen (`hideCustomTitleBarInFullScreen`) in the Command Palette sets the value to `windowed`, and it is available only in full screen while the bar is shown. [stock only]
3. Right-clicking the title bar while in full screen shows a Hide Custom Title Bar In Full Screen row (`toggle.window.customTitleBarVisibility.windowed`, group `3_toggle`), and the row does not show outside full screen. [stock only]
4. With `windowed`, the custom bar stays hidden in full screen even if the menu bar is toggled on. [stock only]
5. With `windowed`, running the Appearance menu Custom Title Bar row in full screen sets `auto` and shows the bar, while running it in a regular window sets `never`. [stock only]

### TB-3.12
1. Right-clicking the title bar shows a row (`toggle.window.titleBarStyle`, group `4_restore_native_title`) that sets `window.titleBarStyle` back to `native`. [cursor only]
2. The restore row shows only while the title bar style is `custom`. [cursor only]
3. The restore row is registered only on Linux desktop, so it does not show on Windows or macOS. [cursor only]

### TB-3.13
1. Running Toggle Full Screen (`workbench.action.toggleFullScreen`) puts the window into full screen and running it again takes the window back out.
2. F11, Ctrl+Cmd+F on macOS, the View, Appearance menu Full Screen row (group `1_toggle_view`) and the Command Palette all run Toggle Full Screen.
3. In full screen the app icon, the window controls and the top resize strip are hidden.
4. In full screen with the classic menu bar mode, the custom title bar stays hidden until the menu bar is toggled, and on macOS the title bar is hidden in full screen.
5. With `window.restoreFullscreen` on, a window that was in full screen when it was closed reopens in full screen.
6. In stock, `window.customTitleBarVisibility` set to `windowed` hides the custom title bar in full screen only (TB-3.11), and Cursor does not have that setting. [stock only]

### TB-3.14
1. On macOS, `window.nativeFullScreen` on (default) makes full screen use the macOS full screen Space.
2. On macOS, turning `window.nativeFullScreen` off makes full screen simple full screen, with no new Space.
3. Turning `window.nativeFullScreen` off forces the native title bar style.
4. Turning on native window tabs (`window.nativeTabs`) forces native full screen on.

### TB-3.15
1. On macOS, setting `window.nativeTabs` on makes windows join a shared native tab bar.
2. Turning on `window.nativeTabs` forces the native title bar style and needs a full restart.
3. New Window Tab (`workbench.action.newWindowTab`) opens a new window as a tab in the native tab bar.
4. Show Previous Window Tab (`workbench.action.showPreviousWindowTab`) and Show Next Window Tab (`workbench.action.showNextWindowTab`) move between the window tabs.
5. Move Window Tab to New Window (`workbench.action.moveWindowTabToNewWindow`) pulls the current tab out into its own window, and Merge All Windows (`workbench.action.mergeAllWindowTabs`) joins all open windows into one tab bar.
6. Toggle Window Tabs Bar (`workbench.action.toggleWindowTabsBar`) shows or hides the native tab bar.
7. The six window tab commands show in the Command Palette only while `window.nativeTabs` is on, and running one with the setting off does nothing.

### TB-3.16
1. Zoom In (`workbench.action.zoomIn`) scales the whole window up by one step of 20 percent.
2. Zoom Out (`workbench.action.zoomOut`) scales the whole window down by one step of 20 percent.
3. Reset Zoom (`workbench.action.zoomReset`) returns the window to the configured default zoom level, or to 0 when the level applies to all windows.
4. The View, Appearance menu has Zoom In, Zoom Out and Reset Zoom rows (group `5_zoom`), and the Command Palette lists all three commands.
5. Zoom In is Ctrl+= (also Ctrl+Shift+= and Ctrl+Numpad+), Zoom Out is Ctrl+- (also Ctrl+Shift+- and Ctrl+Numpad-, but only Ctrl+- and Ctrl+Numpad- on Linux), Reset Zoom is Ctrl+Numpad0, and Cmd replaces Ctrl on macOS.
6. By default only the active window changes zoom, and with `window.zoomPerWindow` off the zoom level applies to all windows.
7. Setting `window.zoomLevel` holds the default zoom level (0 by default, 20 percent per step), is updated with the new level when zoom applies to all windows, and is the level Reset Zoom goes back to when zoom is per window.
8. When zoomed, the title bar counter-scales so it keeps its pixel size when it holds only non-zoomable content, and it never shrinks below the window controls when zoomed out.

### TB-3.17
1. The title bar takes its background, text and border colors from the `titleBar.*` theme colors, using the active set while the window has focus.
2. When the window loses focus, the bar switches to the inactive background and text colors (`titleBar.inactiveBackground` and `titleBar.inactiveForeground`).
3. When the window loses focus, the bar also drops to 60 percent opacity.
4. The command center and window control icons dim along with the bar when the window loses focus.
5. A light title bar background gives the window controls a different hover tint than a dark one.

### TB-3.18
1. On Windows, `window.border` set to `default` (default value) uses the theme colors `window.activeBorder` and `window.inactiveBorder` for the window border, and falls back to the Windows system color. [stock only]
2. On Windows, `window.border` set to `system` follows the Windows settings only, and `off` removes the border. [stock only]
3. On Windows, `window.border` set to a color value in hex, RGB, RGBA, HSL or HSLA format draws the border in that color. [stock only]
4. `window.border` is ignored when the title bar style is `native`. [stock only]
5. The window border is refreshed when the window enters or leaves full screen. [stock only]

### TB-3.19
1. On macOS, `window.clickThroughInactive` on (default) makes the first click on an inactive window both activate it and press whatever is under the pointer.
2. On macOS, `window.clickThroughInactive` off makes the first click on an inactive window only activate it.
3. Changing `window.clickThroughInactive` prompts for a restart.

### TB-3.20
1. In a floating editor window, a pin button on the layout controls turns Always on Top on for that window. [stock only]
2. While Always on Top is on, a pinned icon in the same spot of the layout controls turns it off. [stock only]
3. Toggle Window Always on Top (`workbench.action.toggleWindowAlwaysOnTop`) in the Command Palette flips the state while a floating editor window has focus, and Turn On Always on Top (`workbench.action.enableWindowAlwaysOnTop`) and Turn Off Always on Top (`workbench.action.disableWindowAlwaysOnTop`) set it explicitly. [stock only]
4. The main window cannot be pinned, so the pin button and the pin commands apply only to floating editor windows. [stock only]

### TB-3.21
1. Turning on compact mode in a floating editor window trims its title bar, with no command center, no global actions and no Accounts or Manage buttons. [stock only]
2. In compact mode the title bar shows editor actions from a separate compact window editor title menu in place of the usual editor title menu. [stock only]
3. Compact mode also hides the status bar of the floating window. [stock only]
4. A button on the layout controls of a floating editor window (group `navigation`, order 0) shows Turn On Compact Mode while the window is not compact and Turn Off Compact Mode while it is. [stock only]
5. The Command Palette lists Toggle Window Compact Mode (`workbench.action.toggleCompactAuxiliaryWindow`) and the two explicit commands `workbench.action.enableCompactAuxiliaryWindow` and `workbench.action.disableCompactAuxiliaryWindow`. [stock only]
6. A floating window leaves compact mode on its own when an editor group is added to it, and the `isCompactTitleBar` context key follows the state. [stock only]
7. Toggle Window Compact Mode in the Command Palette turns compact mode on or off while an editor window is focused (`workbench.action.toggleCompactAuxiliaryWindow`). [stock only]
8. A layout control button in the editor window's title bar turns compact mode on, and while compact mode is on it turns into a button that turns it off (`workbench.action.enableCompactAuxiliaryWindow`, `workbench.action.disableCompactAuxiliaryWindow`). [stock only]
9. Compact mode slims the title bar and the editor part and hides the status bar of that window. [stock only]
10. In a floating editor window the layout controls hold a button labelled Turn On Compact Mode that switches the window to compact mode, a smaller title bar with fewer buttons. [stock only]

### TB-3.22
1. When the page runs as an installed web app with the window controls overlay, the title bar reserves space at its right end for the browser's own window controls.
2. On macOS the title bar also reserves space at its left end for the overlay controls.
3. The title bar height follows the height of the window controls overlay.
4. Desktop Cursor never shows this behavior, because it ships as a desktop app only.

### TB-3.23
1. On macOS and Linux with the no title bar layout on, the title bar becomes a transparent 34 px strip fixed at the top of the window. [cursor only]
2. The transparent strip passes clicks through to what is under it, except on its toolbars. [cursor only]
3. In this layout the window title is hidden and the command center is forced off. [cursor only]
4. In this layout the side bar, the panel and the auxiliary bar get about 35 px of top padding so they clear the strip. [cursor only]
5. The layout is driven by a stored preference `cursor/noTitlebarLayout.visibility` whose default is `hide`, and the context key `cursor.noTitlebarLayout.enabled` reports whether the layout is on. [cursor only]
6. The layout never applies on Windows. [cursor only]
7. The layout is off while the feature gate `disable_no_title_bar` is on, and off when editor tabs are set to `none`. [cursor only]

## Unclear
- TB-3.2 piece 2 (`custom` controls) and TB-3.3 (the drawn buttons) describe the same value of `window.controlsStyle`, once as a setting choice and once as the visible result.
- TB-3.10 piece 4 (View, Appearance menu row `toggle.toggleCustomTitleBar`) may overlap with a TB-1 piece, since the feature says TB-1 owns that menu row. Dedupe when merging.
- TB-3.13 piece 6 repeats the setting that TB-3.11 owns. It is kept only so the setting listed on TB-3.13 is covered.
- TB-3.16 does not have a piece for the hidden Cursor command `workbench.action.setZoomLevel` (no menu, no palette entry), because no person can reach it.
- TB-3.20 and TB-3.21 overlap with TB-4.19 and TB-4.18 (same buttons on the layout controls of a floating window). Dedupe when merging.
- TB-3.23 has no visible control that turns the layout on or off, so pieces 1 to 4 describe what happens "with the layout on" without a way for a person to switch it on.
