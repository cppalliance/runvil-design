Group: TB-4 Layout controls. Pass: 3. Date: 2026-10-25.

### TB-4.1
1. Setting `workbench.layoutControl.enabled` on (default) shows the layout control area in the title bar.
2. Setting `workbench.layoutControl.enabled` off hides the whole layout control area from the title bar.
3. Right-clicking the title bar or the title text area shows a Layout Controls row (`toggle.workbench.layoutControl.enabled`, group `2_config`, order 4) that is checked while the controls show and flips the setting when clicked.

### TB-4.2
1. Setting `workbench.layoutControl.type` to `menu` shows a single Configure Layout button that opens a drop-down of layout options.
2. Setting `workbench.layoutControl.type` to `toggles` shows separate toggle buttons for showing and hiding the side bars and the panel.
3. Setting `workbench.layoutControl.type` to `both` (default in stock and in Cursor) shows the layout button and the toggle buttons together.

### TB-4.3
1. Clicking the primary side bar button in the title bar (tooltip Toggle Primary Side Bar) hides the primary side bar when it is shown and shows it when it is hidden.
2. The button icon is a left panel icon when `workbench.sideBar.location` is `left` and a right panel icon when it is `right`, and it switches between an off and an on version as the side bar is hidden or shown.
3. In stock the button shows only while `workbench.layoutControl.type` is `toggles` or `both`, and never in a floating editor window. [stock only]
4. Ctrl+B (Cmd+B on macOS) toggles the primary side bar and the button state follows.
5. In stock, toggling the primary side bar tells screen readers "Primary Side Bar hidden" or "Primary Side Bar shown". [stock only]
6. In Cursor the button sits in the left toolbar when the Agents side bar is on the right (and is hidden there during onboarding), and in the right toolbar otherwise. [cursor only]
7. In Cursor, Ctrl+B and the toggle command are disabled while a floating editor window has focus, and toggling makes no screen reader announcement. [cursor only]

### TB-4.4
1. Clicking the panel button in the title bar (tooltip Toggle Panel) hides the bottom panel when it is shown and shows it when it is hidden.
2. The panel button icon switches between an off and an on version as the panel is hidden or shown.
3. In stock the panel button shows only while `workbench.layoutControl.type` is `toggles` or `both`, never in a floating editor window, and it sits between the two side bar buttons. [stock only]
4. Ctrl+J (Cmd+J on macOS) toggles the panel and the button state follows.
5. In Cursor the panel button is always in the right toolbar. [cursor only]
6. In Cursor, toggling the panel first leaves the maximized layout that hides the editor when that is active. [cursor only]
7. In Cursor, Ctrl+J and the panel toggle command are disabled while a floating editor window has focus. [cursor only]

### TB-4.5
1. A secondary side bar button in the title bar (tooltip Toggle Secondary Side Bar) hides the secondary side bar when it is shown and shows it when it is hidden. [stock only]
2. The secondary side bar button sits on the opposite side from the primary side bar button, with a left icon when `workbench.sideBar.location` is `right` and a right icon when it is `left`, and the icon switches between an off and an on version. [stock only]
3. The secondary side bar button shows only while `workbench.layoutControl.type` is `toggles` or `both`, and never in a floating editor window. [stock only]
4. Ctrl+Alt+B (Cmd+Alt+B on macOS) toggles the secondary side bar. [stock only]
5. Toggling the secondary side bar tells screen readers "Secondary Side Bar hidden" or "Secondary Side Bar shown". [stock only]
6. In Cursor no title bar button toggles the secondary side bar. [cursor only]

### TB-4.6
1. With `workbench.layoutControl.type` set to `menu`, a single Configure Layout button (layout icon) shows in the title bar and opens a drop-down list of layout options.
2. The drop-down lists Primary Side Bar, Secondary Side Bar and Panel as toggle rows, each checked while that part is shown, and clicking a row hides or shows the part.
3. Customize Layout... is the last row of the drop-down and opens the Customize Layout picker (TB-4.7).
4. The Configure Layout button is not shown in a floating editor window.
5. In Cursor the Secondary Side Bar row opens or creates the agent chat pane instead of only toggling a side bar. [cursor only]

### TB-4.7
1. Running Customize Layout... (`workbench.action.customizeLayout`) opens a picker titled Customize Layout with no filter box and a Close button, and it stays open when focus moves elsewhere.
2. Running Customize Layout... again while the picker is open closes it.
3. The picker has section headings Visibility, Primary Side Bar Position, Panel Alignment, Quick Input Position and Modes, and each row shows its on or off state and its key binding when it has one.
4. Choosing a row, or its button, runs that row's command, the list refreshes to the new state at once, and the picker stays open.
5. Customize Layout... is the last row of the Configure Layout drop-down and is also listed in the Command Palette.
6. In stock, with `workbench.layoutControl.type` set to `both`, a Customize Layout button (layout icon) shows in the title bar layout controls, in group `navigation` or in group `1_layout` when editor actions live in the title bar. [stock only]
7. In Cursor there is no standalone Customize Layout button in the title bar, only the drop-down row and the Command Palette entry. [cursor only]
8. In Cursor the Visibility section has no Activity Bar row, and its Secondary Side Bar row shows or hides the agent chat pane. [cursor only]

### TB-4.8
1. A Restore Defaults button (discard icon) is always visible at the top of the Customize Layout picker, next to the Close button.
2. Clicking Restore Defaults puts `workbench.activityBar.location`, `workbench.sideBar.location`, `workbench.statusBar.visible` and `workbench.panel.defaultLocation` back to their default values.
3. Clicking Restore Defaults also puts `window.menuBarVisibility` back to its default, except on macOS desktop where the menu bar is left alone.
4. Clicking Restore Defaults sets the panel alignment back to center and the quick input position back to top.

### TB-4.9
1. Right-clicking empty space in the left title bar toolbar opens a context menu with a single row. [cursor only]
2. The row reads Move Side Bar Right while the primary side bar is on the left, and Move Side Bar Left while it is on the right. [cursor only]
3. Clicking the row swaps the primary side bar to the other side (`workbench.sideBar.location`). [cursor only]
4. Moving the primary side bar this way also moves the Agents side bar to the opposite side. [cursor only]

### TB-4.10
1. In Cursor's no-title-bar layout, the side bar and panel buttons can show at the right end of the top-right editor tab strip instead of in the title bar. [cursor only]
2. The buttons go into the editor tab strip when the right edge has no visible side bar and the editor area or the panel is the right-most part. [cursor only]
3. The buttons stay in the title bar in every other case. [cursor only]
4. The buttons show in one place at a time, either in the title bar or in the editor tab strip. [cursor only]

### TB-4.11
1. Pressing Ctrl+Alt+Tab (Cmd+Alt+Tab on macOS), or running `workbench.action.openLayoutSwitcher` from the Command Palette (category View), opens a full-screen overlay with a row of tiles over a backdrop. [cursor only]
2. Each tile in the overlay shows an icon with a label, and the tile that matches the current layout starts selected. [cursor only]
3. Tab, Left and Right move the selection between tiles. [cursor only]
4. Pressing Enter or clicking a tile applies it and closes the overlay. [cursor only]
5. Pressing Escape closes the overlay without applying a tile. [cursor only]
6. Letting go of the modifier keys applies the selected tile and closes the overlay. [cursor only]

### TB-4.12
1. Picking the tile named Editor in the overlay applies the Editor layout (named Layout - Right elsewhere in the code). [cursor only]
2. The Editor layout hides the Agents side bar and the panel. [cursor only]
3. The Editor layout shows the chat pane, the editor area and the primary side bar. [cursor only]
4. The Editor layout puts the Agents side bar location on the right. [cursor only]

### TB-4.13
1. Other open Cursor windows, not counting floating editor windows or the current window, show first in the overlay as tiles. [cursor only]
2. Each window tile is named after the workspace folder of that window, with an Untitled name when it has none. [cursor only]
3. A separator divides the window tiles from the layout tiles. [cursor only]
4. Choosing a window tile brings that window to the front and focuses it. [cursor only]
5. The number of tiles shown is capped by the screen width. [cursor only]

### TB-4.14
1. Layouts saved in storage show as extra tiles in the overlay after the Agent and Editor tiles. [cursor only]
2. Each saved layout tile has a small preview icon drawn from the pane visibility saved with it. [cursor only]
3. Saved layout tiles are skipped when no folder or workspace is open. [cursor only]
4. A saved layout whose id or name clashes with a default tile is dropped and does not show. [cursor only]
5. Choosing a saved layout tile applies its stored visibility of the agent, chat, editor, panel and side bar parts and its part widths. [cursor only]

### TB-4.15
1. A preset named Layout - Zen shows only the editor area, with the Agents side bar, chat pane, panel and primary side bar all hidden. [cursor only]
2. No tile or button in the IDE window applies the Zen preset, so a person cannot pick it from the overlay. [cursor only]

### TB-4.16
1. A preset named Browser shows the editor area and the chat pane, with the Agents side bar, panel and primary side bar hidden. [cursor only]
2. No tile or button in the IDE window applies the Browser preset, so a person cannot pick it from the overlay. [cursor only]
3. The Browser preset only sets which panes show and does not itself open a browser. [cursor only]

### TB-4.17
1. When a workspace has no saved side bar or panel visibility, Cursor opens it in the layout last picked in the overlay, or in the Editor layout when none was picked. [cursor only]
2. The first-open layout always starts with the Agents side bar hidden and the chat pane shown. [cursor only]
3. The editor area starts hidden in the first-open layout only when the auto-hide-editor-when-empty flag is on. [cursor only]
4. A workspace that already has saved side bar or panel visibility keeps it and does not get the first-open layout. [cursor only]

### TB-4.19
1. In a floating editor window the layout controls hold a pin button labelled Turn On Always on Top that keeps that window above other windows. [stock only]
2. While the window is on top, the same spot shows a pinned icon labelled Turn Off Always on Top that releases it. [stock only]
3. The pin button is in group `navigation` at order 1 of the layout controls and shows only in floating editor windows, not in the main window. [stock only]
4. The Command Palette lists Toggle Window Always on Top (`workbench.action.toggleWindowAlwaysOnTop`), which works only while a floating editor window has focus, along with Turn On Always on Top (`workbench.action.enableWindowAlwaysOnTop`) and Turn Off Always on Top (`workbench.action.disableWindowAlwaysOnTop`). [stock only]

### TB-4.20
1. newer-than-1.128: with the experimental Modern UI on (`workbench.experimental.modernUI`), Customize Layout gets a Layout Density section with Default and Compact rows. [stock only]
2. newer-than-1.128: choosing Default or Compact sets `window.density.layout` through `workbench.action.setLayoutDensity.default` or `workbench.action.setLayoutDensity.compact`, and the chosen row shows a check. [stock only]
3. newer-than-1.128: Restore Defaults in Customize Layout also resets the layout density. [stock only]

## Unclear
- TB-4.18 and TB-4.19 are the same buttons as TB-3.21 and TB-3.20. Dedupe when merging.
- TB-4.15 and TB-4.16 describe presets that no tile in the IDE window applies, so a person cannot see them. The parent should decide whether to keep them.
- TB-4.12 piece 4 (Agents side bar goes to the right) is medium confidence, as the feature block says the preset side field is probably the Agents side bar side.
- TB-4.14 has no piece for creating, renaming or deleting a saved layout, because no such control was found in the IDE window.
- TB-4.8 lists the settings that Restore Defaults resets from stock source, and the feature block says the exact Cursor list was not decoded.
- TB-4.20 piece 1 says the Layout Density rows are inside Customize Layout, which follows the feature block, and the source read here only shows the density menu and its two commands.
- TB-4.10 piece 4 is an inference from the positioning mode and CSS class names, and the feature block says when the no-title-bar layout is active was not decoded.
