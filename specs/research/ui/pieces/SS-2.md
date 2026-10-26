Group: SS-2 Bar and item colors. Pass: 3. Date: 2026-10-25.

### SS-2.1
1. In a window with a folder or workspace open, the bar is painted with `statusBar.background` and its text uses `statusBar.foreground`.
2. A 1 px line along the top edge of the bar is drawn in `statusBar.border`, or in the high-contrast border color when the theme sets no border.
3. The bar fades between background colors over 0.15 s when motion is on.
4. When a theme sets nothing, the bar is `#007ACC` with white text and no top line.
5. Dark 2026 uses `#191A1B` with `#8C8C8C` text, and Light 2026 uses `#FAFAFD` with `#606060` text. [stock only]
6. Cursor Dark uses `#141414` with `#F0F0F099` text. [cursor only]

### SS-2.2
1. A window with no folder or workspace open paints the bar with `statusBar.noFolderBackground`, `statusBar.noFolderForeground`, and `statusBar.noFolderBorder`.
2. The bar switches between the normal colors and the no-folder colors as soon as a folder is opened or closed.
3. When a theme sets nothing, the no-folder background is purple `#68217A` and the foreground and border follow the normal bar.
4. Dark 2026 sets the no-folder colors equal to the normal bar, and Light 2026 uses `#F0F0F3` against `#FAFAFD`. [stock only]
5. Dark Modern uses `#1F1F1F` against `#181818`, and Dark (Visual Studio) and Light (Visual Studio) still show the purple.

### SS-2.3
1. While a debug session is running, the whole bar changes to `statusBar.debuggingBackground`, `statusBar.debuggingForeground`, and `statusBar.debuggingBorder`, and goes back when the session ends.
2. The debugging colors do not apply to sessions started without debugging, to sessions that ask to hide the debug status bar, or while the debugger is still starting.
3. The debugging colors override the no-folder colors.
4. When a theme sets nothing, the debugging background is orange `#CC6633`.
5. Dark Modern uses `#0078D4`, and Light Modern uses `#FD716C` with black text.
6. Dark 2026 uses `#3994BC`. [stock only]

### SS-2.4
1. Setting `debug.enableStatusBarColor` (Debug: Enable Status Bar Color, default on) decides whether the bar takes the debugging colors.
2. Switching the setting takes effect at once, even in a running debug session.
3. The setting shows as a checkbox under Debug in the Settings editor.

### SS-2.5
1. An item marked prominent gets its own background and text from `statusBarItem.prominentBackground` and `statusBarItem.prominentForeground`.
2. A prominent item uses `statusBarItem.prominentHoverBackground` and `statusBarItem.prominentHoverForeground` on hover, except when it sits in a compact pair.
3. When a theme sets nothing, the prominent background is a 50 percent black layer and the text is the bar's text color.
4. Dark Modern uses `#6E768166` for the prominent background.
5. Dark 2026 uses `#3994BC` for the prominent background. [stock only]
6. Cursor's own themes use a faint tint, for example `#F0F0F011` in Cursor Dark. [cursor only]
7. In stock, the items marked prominent are Window Zoom, Restricted Mode, Screen Reader Optimized, Scrolling Locked, Tab Moves Focus, OVR, Column Selection, the notebook Select Kernel item, and the Show Output quiet item.

### SS-2.6
1. An item marked warning is drawn with `statusBarItem.warningBackground` and `statusBarItem.warningForeground`.
2. A warning item uses `statusBarItem.warningHoverBackground` and `statusBarItem.warningHoverForeground` on hover.
3. When a theme sets nothing, the warning background is the editor warning color darkened by 40 percent, with white text.
4. In stock, the Problems visibility item, the Pty Host item, and a language status entry with warning severity are drawn as warning items.
5. An extension item that asks for the warning background color is drawn as a warning item.

### SS-2.7
1. An item marked error is drawn with `statusBarItem.errorBackground` and `statusBarItem.errorForeground`.
2. An error item uses `statusBarItem.errorHoverBackground` and `statusBarItem.errorHoverForeground` on hover.
3. When a theme sets nothing, the error background is the editor error color darkened by 40 percent, with white text.
4. A language status entry with error severity is drawn as an error item.
5. An extension item that asks for the error background color is drawn as an error item.
6. The Performance Trace recording item is drawn as an error item. [stock only]

### SS-2.8
1. The remote indicator at the far left is drawn with `statusBarItem.remoteBackground` and `statusBarItem.remoteForeground`.
2. The remote indicator uses `statusBarItem.remoteHoverBackground` and `statusBarItem.remoteHoverForeground` on hover.
3. The default remote colors copy the activity bar badge colors, and Dark Modern sets `#0078D4` with white text.
4. The remote colors apply only while the window is connected to a remote, so a plain local window shows an unstyled indicator. [stock only]
5. The remote colors always show on the indicator unless the machine is offline. [cursor only]

### SS-2.9
1. When the system reports that the network is offline, the remote indicator switches to the offline colors.
2. The offline background `statusBarItem.offlineBackground` defaults to dark red `#6c1717`, and `statusBarItem.offlineForeground` defaults to the remote foreground.
3. The offline indicator uses `statusBarItem.offlineHoverBackground` and `statusBarItem.offlineHoverForeground` on hover.
4. The indicator returns to the remote or normal look when the network comes back.

### SS-2.10
1. While the pointer is over an item that has a command, its background becomes `statusBarItem.hoverBackground` and its text becomes `statusBarItem.hoverForeground`.
2. Items of the prominent, warning, error, remote, and offline kinds use their own hover colors instead.
3. The two halves of a compact pair light up together, and the half under the pointer uses `statusBarItem.compactHoverBackground`.
4. High-contrast themes add a dashed outline in the contrast active border color and skip the compact pair effect.
5. When a theme sets nothing, the hover background is white 12 percent on dark themes and black 12 percent on light themes, and the hover text follows the bar text.

### SS-2.11
1. While the mouse button is held down on an item that has a command, the item's background changes to `statusBarItem.activeBackground`.
2. When a theme sets nothing, the pressed background is white 18 percent, and black 18 percent in high contrast light.
3. High-contrast themes draw a solid outline in the contrast active border color instead of the fill.

### SS-2.12
1. When the bar itself has keyboard focus, it draws an outline in `statusBar.focusBorder` and hides its top line.
2. When an item has keyboard focus, it gets a 1 px solid outline in the theme's contrast active border color if there is one, and otherwise in `statusBarItem.focusBorder`.
3. The item outline sits 2 px inside the item when the bar has a border, and 1 px inside when it does not.
4. On macOS, when the window is not full screen, a focused bar gets rounded bottom corners, 10 px and 16 px on Tahoe, so the outline fits.
5. When a theme sets nothing, both focus colors are the bar's text color, and Dark Modern sets `#0078D4`.

### SS-2.13
1. Cursor Dark sets the no-folder bar background to `#141414` and Cursor Light to `#F3F3F3`, equal to their normal bars. [cursor only]
2. The two High Contrast themes set the no-folder background to `#0A0A0A` and `#FFFFFF`, Midnight sets it to `#191c22`, and the two Colorblind themes match their parents, all equal to their normal bars. [cursor only]
3. Cursor Dark, Cursor Light, the two Colorblind themes, and Light High Contrast set the remote indicator to the bar color, so no purple or blue block appears. [cursor only]
4. While debugging, Cursor Dark and its Colorblind twin show a faint tint `#F0F0F01C`. [cursor only]
5. While debugging, Dark High Contrast and Midnight shift the bar to the gray-blue `#434C5E`, and the three light themes show no change. [cursor only]
6. Dark High Contrast and Midnight do not set the remote colors, so the remote indicator falls back to the activity bar badge colors. [cursor only]
7. Setting `workbench.colorTheme` chooses which of the seven Cursor themes colors the bar. [cursor only]

### SS-2.14
1. Newer than 1.128: the token `statusBar.inactiveBackground` repaints the bar while the window has lost focus, in a window with a folder or workspace open and no debug color active. [stock only]
2. Newer than 1.128: the token has no default value, so nothing changes until a theme sets it. [stock only]

### SS-2.15
1. Adding any of the 35 `statusBar.*` and `statusBarItem.*` keys, 32 core keys plus the three debugging keys, to `workbench.colorCustomizations` in settings.json recolors the bar.
2. Switching the theme with Color Theme (`workbench.action.selectTheme`, setting `workbench.colorTheme`) recolors the bar.
3. There is no settings page for status bar colors.

## Unclear

- SS-2.2, SS-2.3, and SS-2.5 give some theme values without a Cursor tag because Dark Modern and Light Modern ship in both products, but I did not check Cursor's copy of those theme files, so the values are the stock ones.
- SS-2.6 and SS-2.7 do not say which Cursor items use the warning and error kinds. The feature blocks confirmed only the tokens and the styles.
- SS-2.13 values come from Cursor's theme files, not from a running window, and a user's `workbench.colorCustomizations` can override them.
