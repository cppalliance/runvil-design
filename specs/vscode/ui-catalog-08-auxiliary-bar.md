# 8. Auxiliary bar (AX)

The secondary side bar. Its stock occupant is Chat, which is out, so what remains is the container. AX-1 and AX-4 could merge. After the agent and chat parts are removed, Cursor adds almost nothing to this section.

5 groups, 27 features. Each feature lists its entry points and the pieces that finish it.

## AX-1 Show, hide and focus

(both) [boundary]. Open, close and focus the part, and pick its starting visibility. In Cursor the toggle opens or creates the agent chat.

4 features.

#### AX-1.1 Toggle the secondary side bar open or closed

- Tag: both. Evidence: source. Boundary: yes. Cursor: changed (id, label, key and both menu rows found; the action opens or creates the agent chat, is disabled in an auxiliary window, and no "Secondary Side Bar shown" or "hidden" alert string exists in the bundle)
- What: Press Ctrl+Alt+B, or use View > Appearance > Secondary Side Bar, to show or hide the part. Showing it reopens the container you used last (or the first container that has views), and hiding it also closes the active container and moves focus to the panel or the editor. Stock tells screen readers "Secondary Side Bar shown" or "hidden". Boundary: in Cursor, when the part opens with no container active, the toggle creates a new agent chat in it, and it sends `ai_pane.opened` and `ai_pane.closed` events.
- Commands: workbench.action.toggleAuxiliaryBar
- Menus: MenubarAppearanceMenu; LayoutControlMenuSubmenu
- Keys: Ctrl+Alt+B (Cmd+Alt+B on Mac); in Cursor the key and the command are off while a separate auxiliary window has focus
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (30-84); V\src\vs\workbench\browser\layout.ts (2253-2292); C bundle (class with `Toggle Secondary Side Bar Visibility`); C nls.messages.json (3305-3308)
- Pieces:
  1. Pressing `Ctrl+Alt+B` (`Cmd+Alt+B` on Mac) while the secondary side bar is hidden shows it and reopens the container used last, or the first container that has views.
  2. Pressing the same key while the part is shown hides it, closes its active container, and moves focus to the panel or the editor.
  3. View > Appearance > Secondary Side Bar and the row in the Layout Control drop-down run `workbench.action.toggleAuxiliaryBar`, and the Appearance row shows a check mark while the part is open.
  4. Toggling the part makes screen readers announce "Secondary Side Bar shown" or "Secondary Side Bar hidden". [stock only]
  5. When the part opens with no container active, the toggle creates a new agent chat in it, and the key and command are turned off while a separate auxiliary window has focus. [cursor only]

#### AX-1.2 Hide the part without toggling

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the container title menu row "Hide Secondary Side Bar" is present, NLS 3310; `workbench.action.closeAuxiliaryBar`, the X button and the `auxiliarybar-close` icon have 0 hits)
- What: Run Hide Secondary Side Bar from the Command Palette, click the X button in the part's title row, or right-click the container title and pick Hide Secondary Side Bar. All three close the part and never open it. The X button only shows when `workbench.activityBar.location` is `default`. Hiding while the part is maximized first leaves maximized mode.
- Commands: workbench.action.closeAuxiliaryBar; workbench.action.toggleAuxiliaryBar
- Menus: AuxiliaryBarTitle; ViewContainerTitleContext; CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (86-110, 181-191); V\src\vs\workbench\browser\layout.ts (2253-2256); C bundle (`ViewContainerTitleContext` item with the toggle id); C nls.messages.json (3310)
- Pieces:
  1. Running Hide Secondary Side Bar (`workbench.action.closeAuxiliaryBar`) from the Command Palette closes the part, and the command is disabled while the part is already hidden. [stock only]
  2. Clicking the X button in the part's title row closes the part, and the button shows only when `workbench.activityBar.location` is `default`. [stock only]
  3. Right-clicking the container title and picking Hide Secondary Side Bar closes the part and never opens it.
  4. Hiding the part while it is maximized first leaves maximized mode and then closes the part.

#### AX-1.3 Move keyboard focus into the part

- Tag: both. Evidence: source. Boundary: no. Cursor: present (id and label "Focus into Secondary Side Bar" found; the Cursor action is off while an auxiliary window has focus and does nothing in the Agents window)
- What: Run Focus into Secondary Side Bar to show the part if it is hidden and put the cursor in the active container. There is no default key.
- Commands: workbench.action.focusAuxiliaryBar
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (112-139); C bundle (class with `workbench.action.focusAuxiliaryBar`); C nls.messages.json (3309)
- Pieces:
  1. Running Focus into Secondary Side Bar (`workbench.action.focusAuxiliaryBar`) from the Command Palette puts the keyboard cursor in the active container of the part.
  2. Running it while the part is hidden shows the part first and then moves focus into it.
  3. The command is listed in the Command Palette under the View category and has no default key.
  4. The command is turned off while a separate auxiliary window has focus. [cursor only]

#### AX-1.4 Choose whether a new workspace starts with the part open

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (the setting id has 0 hits; Cursor decides first-run visibility from its layout presets, which belong to the agent layout)
- What: In Settings, pick `hidden`, `visibleInWorkspace` (the default), or `visible` to decide if a workspace or window opened for the first time shows the part. The two `maximized` values belong to AX-3.4. The setting is skipped when the startup editor is the agent sessions page.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.secondarySideBar.defaultVisibility
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (597-610); V\src\vs\workbench\browser\layout.ts (2970-3000, 3045-3059); C bundle (0 hits for the id)
- Pieces:
  1. Setting `workbench.secondarySideBar.defaultVisibility` set to `visibleInWorkspace` (the default) shows the part the first time a folder or workspace opens, and leaves it hidden in an empty window. [stock only]
  2. Setting it to `visible` shows the part the first time any window opens, with or without a folder. [stock only]
  3. Setting it to `hidden` keeps the part hidden the first time a workspace or window opens. [stock only]
  4. The setting is ignored and the part starts hidden when `workbench.startupEditor` is the agent sessions page. [stock only]
  5. The setting only affects a workspace or window opened for the first time, and later opens use the saved layout. [stock only]


## AX-2 Where it sits and how wide

(both). It follows the primary side bar to the opposite side, flips with it, and resizes within width limits. Cursor uses a 300 minimum and a 540 preferred width.

4 features.

#### AX-2.1 The part sits opposite the primary side bar and flips with it

- Tag: both. Evidence: source. Boundary: no. Cursor: present (setting registered with the same text, "The secondary side bar will show on the opposite side of the workbench", NLS 4213; the three commands found with Primary Side Bar labels, NLS 3085-3087)
- What: The part has no side setting of its own. It always shows on the opposite side of the editor from the primary side bar, and when the side bar moves left or right the part moves the other way. There is no way to put both on the same side.
- Commands: workbench.action.toggleSidebarPosition; workbench.action.moveSideBarRight; workbench.action.moveSideBarLeft
- Menus: none found
- Keys: none found
- Settings: workbench.sideBar.location
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (566-572); V\src\vs\workbench\browser\layout.ts (636-660, 1965-2020); V\src\vs\workbench\browser\actions\layoutActions.ts (97-171); C bundle (`workbench.sideBar.location` registration, `toggleSidebarPosition` class)
- Pieces:
  1. With the primary side bar on the left, the secondary side bar shows on the right side of the editor.
  2. With the primary side bar on the right, the secondary side bar shows on the left side of the editor.
  3. Changing `workbench.sideBar.location` between `left` and `right` moves the secondary side bar to the opposite side at once.
  4. Running Toggle Primary Side Bar Position (`workbench.action.toggleSidebarPosition`) from the Command Palette, or the Move Primary Side Bar Right and Move Primary Side Bar Left commands (`workbench.action.moveSideBarRight`, `workbench.action.moveSideBarLeft`) from their menu rows, moves the primary side bar and so moves the secondary side bar the other way.
  5. No setting or command puts both side bars on the same side of the editor.

#### AX-2.2 Swap the two side bars from the part's own menus

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (neither label is in the NLS table or the bundle; the primary side bar rows "Move Primary Side Bar Right" and "Move Primary Side Bar Left" are present)
- What: Right-click the part's tab strip or its container title and pick Move Secondary Side Bar Left or Move Secondary Side Bar Right. The label flips with the current side. It runs the same command as the primary side bar swap, so both bars trade places.
- Commands: workbench.action.toggleSidebarPosition
- Menus: ViewContainerTitleContext; tab strip extra context actions (built in `auxiliaryBarPart.ts`, not a MenuId)
- Keys: none found
- Settings: workbench.sideBar.location
- Source: V\src\vs\workbench\browser\actions\layoutActions.ts (209-230); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (233-261); C bundle (container title context items for the primary side bar only)
- Pieces:
  1. Right-clicking the part's tab strip shows a row that reads Move Secondary Side Bar Left or Move Secondary Side Bar Right, and the label names the side the part would move to. [stock only]
  2. Right-clicking the container title shows the same Move Secondary Side Bar row, and the label flips with the current side. [stock only]
  3. Picking the row runs `workbench.action.toggleSidebarPosition`, so the primary and secondary side bars trade places. [stock only]

#### AX-2.3 Resize the part by dragging its edge

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (`get minimumWidth(){return 300}`, `Math.max(t,540)` for the preferred width, `get snap(){return!1}`; saved-size key `auxiliaryBar.size` with a default of 200 in its key table; an extra Cursor width key `cursor/layout/editor.auxiliaryBar.width`)
- What: Drag the edge toward the editor to change the width. Stock allows 170 pixels at the narrowest, drags shut if you pull below that, and a double-click on the edge sets a best-fit width that is never under 300. The size is saved and starts at 300 or a quarter of the window, whichever is smaller. Cursor raises the narrowest width to 300, the best-fit floor to 540, and turns snapping shut off.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (52-79); V\src\vs\workbench\browser\parts\paneCompositePart.ts (113-117); V\src\vs\base\browser\ui\grid\grid.ts (719); V\src\vs\workbench\browser\layout.ts (1823, 2827, 2970-2975); C bundle (aux bar part class, state key table)
- Pieces:
  1. Dragging the part's edge toward or away from the editor changes the part's width.
  2. The width is saved, and a fresh layout starts at 300 px or a quarter of the window width, whichever is smaller.
  3. Dragging the edge below 170 px closes the part by snapping it shut. [stock only]
  4. Double-clicking the edge sets a best-fit width for the open container that is never under 300 px. [stock only]
  5. The narrowest width is 300 px, a double-click on the edge sets a best-fit width that is never under 540 px, and dragging smaller never snaps the part shut. [cursor only]

#### AX-2.4 The part takes the Side Bar colors, with a border toward the editor

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (`updateStyles` sets the editor background color variable, falls back to a Cursor stroke color variable (`cursor-stroke-primary`) for the border, and ties the border to the editor and chat layout flags)
- What: Stock paints the part with the Side Bar background, foreground and border theme colors (`sideBar.background`, `sideBar.foreground`, `sideBar.border`, with the contrast border as a fallback) and draws a 1 pixel border only on the edge facing the editor, with the tab strip using the Panel title colors in the title row and the Activity Bar top colors in a top or bottom strip, while Cursor paints the background with the editor background, takes the border color from a Cursor stroke color, and draws the border only under certain layout conditions.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (181-199, 219-228); C bundle (aux bar part `updateStyles`)
- Pieces:
  1. The part is painted with `sideBar.background` and its text uses `sideBar.foreground`. [stock only]
  2. A 1 px border in `sideBar.border`, or in the contrast border color when the theme sets no border, shows only on the edge of the part that faces the editor. [stock only]
  3. In the title row the tab strip uses the Panel title colors, and in a strip at the top or bottom of the part it uses the Activity Bar top colors.
  4. The part is painted with the editor background, its border takes the Cursor stroke color `cursor-stroke-primary`, and the border shows only under certain layout conditions. [cursor only]


## AX-3 Maximize and restore

(vscode). Fill the window with the part and restore the layout later. Cursor has none of these commands.

6 features.

#### AX-3.1 Maximize the secondary side bar

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (searched the bundle and the NLS table for both ids, the labels, the `auxiliaryBarMaximized` context key and the `auxiliaryBar.wasLastMaximized` state key; 0 hits)
- What: Run Maximize Secondary Side Bar to make the part fill the window. The editor, the panel and the primary side bar are hidden, and the part shows first if it was hidden. The code remembers which parts were visible and the part's width so they can come back. It is disabled while the part is already maximized. Maximizing the panel and maximizing this part cannot be on together.
- Commands: workbench.action.maximizeAuxiliaryBar; workbench.action.toggleMaximizedAuxiliaryBar
- Menus: CommandPalette; AuxiliaryBarTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (216-239, 269-299); V\src\vs\workbench\browser\layout.ts (2132-2207, 2209-2213); V\src\vs\workbench\common\contextkeys.ts (162); C bundle (0 hits)
- Pieces:
  1. Running Maximize Secondary Side Bar (`workbench.action.maximizeAuxiliaryBar`) from the Command Palette makes the part fill the window and hides the editor, the panel, and the primary side bar. [stock only]
  2. Maximizing while the part is hidden shows the part first, and keyboard focus moves into it. [stock only]
  3. The command is disabled while the part is already maximized. [stock only]
  4. Running Toggle Maximized Secondary Side Bar (`workbench.action.toggleMaximizedAuxiliaryBar`) from the Command Palette maximizes the part, and running it again restores the layout. [stock only]
  5. The panel and this part are never maximized at the same time. [stock only]

#### AX-3.2 Restore the layout after maximizing

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (id, label and the `auxiliaryBarMaximized` key have 0 hits)
- What: While the part is maximized, run Restore Secondary Side Bar or press Ctrl+W (Ctrl+F4 or Ctrl+W on Windows) to bring back the editor, panel and primary side bar that were visible before, at their earlier sizes, and put focus back in the part.
- Commands: workbench.action.restoreAuxiliaryBar
- Menus: CommandPalette
- Keys: Ctrl+W (Cmd+W on Mac); Ctrl+F4 and Ctrl+W on Windows; only while the part is maximized
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (241-267); V\src\vs\workbench\browser\layout.ts (2182-2205); C bundle (0 hits)
- Pieces:
  1. Running Restore Secondary Side Bar (`workbench.action.restoreAuxiliaryBar`) while the part is maximized brings back the editor, the panel, and the primary side bar that were visible before maximizing. [stock only]
  2. The restored parts and the secondary side bar return at their earlier sizes, and keyboard focus moves back into the part. [stock only]
  3. Pressing `Ctrl+W` (`Cmd+W` on Mac), or `Ctrl+F4` on Windows, restores the layout while the part is maximized. [stock only]
  4. The command is disabled and the keys are not bound while the part is not maximized. [stock only]

#### AX-3.3 Maximize or restore with the button in the title row

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (command id, icon id and menu row have 0 hits; Cursor's own title buttons are agent buttons, which are left out)
- What: The first button in the part's title row, with the `auxiliarybar-maximize` screen-full icon, maximizes the part. While maximized its tooltip changes to "Restore Secondary Side Bar" and the same click restores.
- Commands: workbench.action.toggleMaximizedAuxiliaryBar
- Menus: AuxiliaryBarTitle
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (22, 269-299); C bundle (0 hits)
- Pieces:
  1. The first button in the part's title row shows a screen-full icon with the tooltip Maximize Secondary Side Bar, and clicking it maximizes the part. [stock only]
  2. While the part is maximized, the same button's tooltip reads Restore Secondary Side Bar, and clicking it restores the layout. [stock only]
  3. The button sits before the Hide X button in the title row. [stock only]

#### AX-3.4 Start a new workspace with the part maximized

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (the setting id has 0 hits)
- What: In Settings, set the starting visibility to `maximized` (always) or `maximizedInWorkspace` (only when a folder or workspace is open). The first time a workspace opens, the part is shown and maximized, and the Welcome page is skipped at startup while it is maximized. The other three values of the setting are AX-1.4.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.secondarySideBar.defaultVisibility
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (597-610); V\src\vs\workbench\browser\layout.ts (3045-3059, 3086-3092); V\src\vs\workbench\contrib\welcomeGettingStarted\browser\startupPage.ts (118-121); C bundle (0 hits)
- Pieces:
  1. Setting `workbench.secondarySideBar.defaultVisibility` to `maximized` shows the part maximized, with the editor, panel, and primary side bar hidden, the first time any workspace or window opens. [stock only]
  2. Setting it to `maximizedInWorkspace` does the same only when a folder or workspace is open, and an empty window follows the normal visibility rules. [stock only]
  3. The Welcome page is skipped at startup while the part is maximized. [stock only]

#### AX-3.5 Keep the part maximized whenever no editor is open

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (the setting id has 0 hits)
- What: An experimental setting forces the part to maximize at startup and again each time the last editor closes. Turning it on while no editor is open maximizes at once, and turning it off while maximized restores. With it on, the first-run width is half the window or 300, whichever is larger.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.secondarySideBar.forceMaximized
- Source: V\src\vs\workbench\browser\workbench.contribution.ts (611-617); V\src\vs\workbench\browser\layout.ts (365-389, 447-455, 2970-2975); C bundle (0 hits)
- Pieces:
  1. Setting `workbench.secondarySideBar.forceMaximized` (experimental, off by default) to true makes the part start maximized. [stock only]
  2. With the setting on, the part maximizes again each time the last open editor closes. [stock only]
  3. Turning the setting on while no editor is open maximizes the part at once, and turning it off while the part is maximized restores the layout. [stock only]
  4. With the setting on, a first-run part is half the window wide or 300 px, whichever is larger. [stock only]

#### AX-3.6 Maximized mode ends when you bring back another part

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (no maximize state exists to end)
- What: While the part is maximized, showing the editor, the primary side bar or the panel, or opening an editor yourself, restores the saved layout first, an editor opened automatically instead of by your own action does not end maximized mode, and hiding the part also restores first.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\layout.ts (344-363, 1872-1875, 1911-1914, 2050-2057, 2253-2256); C bundle (state key table has `PANEL_WAS_LAST_MAXIMIZED` and no aux bar maximize keys)
- Pieces:
  1. While the part is maximized, showing the editor, the primary side bar, or the panel restores the saved layout first. [stock only]
  2. While the part is maximized, opening an editor yourself restores the saved layout first. [stock only]
  3. An editor opened automatically, not by your own action, does not end maximized mode. [stock only]
  4. Hiding the part while it is maximized restores the saved layout first and then hides the part. [stock only]


## AX-4 Tab strip and title buttons

(both). The row of tabs, its right-click menu, and previous and next view. Cursor's strip scrolls and has fewer menu entries.

10 features.

#### AX-4.1 Switch containers with the tab strip

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for the label "Active View Switcher", NLS 3321, and the `tablist` role; found. The Cursor strip is shown or hidden by the Agents sidebar state, not by a setting, and its tab chips are styled for chat tabs)
- What: The title row shows one tab per view container in the part. Click a tab to open that container, or move between tabs with the arrow keys and press Enter or Space. The active tab has an underline in the Panel title colors. Hovering a tab shows its name and key. The strip is a tab list that screen readers announce as "Active View Switcher".
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (286-320); V\src\vs\workbench\browser\parts\compositeBarActions.ts (418); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (201-231, 263-284); C bundle (composite bar `create`, aux bar `shouldShowCompositeBar`)
- Pieces:
  1. The part's title row shows one tab for each view container in the part, and the strip is a tab list that screen readers announce as "Active View Switcher".
  2. Clicking a tab opens that container in the part.
  3. The arrow keys move focus between tabs, and Enter or Space opens the focused tab.
  4. The active tab has an underline drawn in the Panel title colors.
  5. Hovering a tab shows its name and its key.
  6. In Cursor the tab strip is shown or hidden by the Agents sidebar state instead of a setting. [cursor only]

#### AX-4.2 Show tabs as text or icons

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the setting is registered, NLS 4220, and `canShowLabels` is always true; the `auxiliarybar.toggleShowLabels` id has 0 hits)
- What: Tabs show as text labels by default. Turn the setting off, or right-click the strip and pick Show Icons, to show icons only. Pick Show Labels to switch back. The choice is ignored when the strip sits at the top or bottom of the part, which always uses icons. In Cursor the setting still works, but the menu row is gone and labels are always allowed.
- Commands: workbench.action.auxiliarybar.toggleShowLabels
- Menus: tab strip extra context actions (built in `auxiliaryBarPart.ts`, not a MenuId)
- Keys: none found
- Settings: workbench.secondarySideBar.showLabels
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (163-170, 247-252); V\src\vs\workbench\browser\workbench.contribution.ts (618-623); C bundle (aux bar `resolveConfiguration`)
- Pieces:
  1. Setting `workbench.secondarySideBar.showLabels` to true (the default) shows the tabs as text labels.
  2. Setting it to false shows the tabs as icons only.
  3. Right-clicking the strip and picking Show Icons switches the tabs to icons, and the row then reads Show Labels to switch back. [stock only]
  4. The setting is ignored when the strip sits at the top or bottom of the part, where the tabs always show icons.
  5. The setting still switches between labels and icons, the Show Icons and Show Labels row is not in the right-click menu, and labels are always allowed. [cursor only]

#### AX-4.3 Put the strip at the top or bottom of the part, or hide it

- Tag: vscode. Evidence: source. Boundary: no. Cursor: changed (the setting is registered and its text still names the Secondary Side Bar, NLS 4222-4226, but `resolveConfiguration` returns the fixed position `default`; the four command ids, `ActivityBarPositionMenu` and the label "Activity Bar Position" have 0 hits)
- What: Set the Activity Bar location to `top`, `bottom` or `hidden` and the part's tab strip follows. With `default` the tabs sit in the title row. With `top` or `bottom` the tabs move to their own strip and the title row shows the container name. With `hidden` the tabs are gone. The right-click menu has an Activity Bar Position submenu with the same four choices. Cursor keeps the setting but this part pins the position to `default`.
- Commands: workbench.action.activityBarLocation.default; workbench.action.activityBarLocation.top; workbench.action.activityBarLocation.bottom; workbench.action.activityBarLocation.hide
- Menus: ActivityBarPositionMenu; ViewContainerTitleContext; CommandPalette
- Keys: none found
- Settings: workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (163-179, 286-294); V\src\vs\workbench\browser\parts\activitybar\activitybarPart.ts (483-601); V\src\vs\workbench\browser\workbench.contribution.ts (650-662); C bundle (aux bar `resolveConfiguration`)
- Pieces:
  1. Setting `workbench.activityBar.location` to `default` puts the tabs in the part's title row.
  2. Setting it to `top` moves the tabs into their own strip at the top of the part, and the title row shows the container name instead.
  3. Setting it to `bottom` moves the tabs into their own strip at the bottom of the part, and the title row shows the container name instead.
  4. Setting it to `hidden` removes the tab strip from the part.
  5. Right-clicking the strip shows an Activity Bar Position submenu with Default, Top, Bottom, and Hidden, with a check mark on the current choice. [stock only]
  6. The Command Palette lists Move Activity Bar to Side, Move Activity Bar to Top, Move Activity Bar to Bottom, and Hide Activity Bar (`workbench.action.activityBarLocation.default`, `.top`, `.bottom`, `.hide`), each only while it is not the current position. [stock only]
  7. The setting is kept but this part pins the position to `default`, so the tabs never leave the title row or disappear. [cursor only]

#### AX-4.4 Hide the strip when only one container is left

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (`workbench.activityBar.autoHide` has 0 hits and the visible-containers tracker is not in the Cursor part)
- What: With the strip at the top or bottom, turn on Auto Hide and the strip disappears when the part holds a single visible container, and returns when a second one appears. It has no effect with the strip in the title row.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: workbench.activityBar.autoHide; workbench.activityBar.location
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (133-161, 268-281); V\src\vs\workbench\browser\workbench.contribution.ts (663-668); C bundle (0 hits)
- Pieces:
  1. With the strip at the top or bottom and `workbench.activityBar.autoHide` on, the strip disappears when the part holds a single visible container. [stock only]
  2. The strip returns when a second container becomes visible in the part. [stock only]
  3. Auto Hide has no effect while the strip sits in the title row. [stock only]

#### AX-4.5 Right-click menu on the tab strip

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the strip menu builder and the labels "Hide '{0}'" and "Keep '{0}'", NLS 3325-3326, are present; the Cursor part passes an empty `fillExtraContextMenuActions:t=>{}`)
- What: Right-click the strip or the title row to get a list of the part's containers with check marks to show or hide each tab (the last tab cannot be hidden). Stock adds a Views submenu (strip in the title row), Activity Bar Position, Move Secondary Side Bar Left or Right, Show Icons or Show Labels, and Hide Secondary Side Bar. Right-clicking a single tab adds the per-tab entries (Hide or Keep, Hide Badge, Configure Keybinding, Move To, Reset Location). Cursor keeps the check-mark list and the per-tab entries but drops all the stock extras.
- Commands: workbench.action.toggleSidebarPosition; workbench.action.toggleAuxiliaryBar
- Menus: tab strip extra context actions (built in `auxiliaryBarPart.ts`, not a MenuId); ActivityBarPositionMenu; ViewsSubMenu
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (654-686); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (233-261); V\src\vs\workbench\browser\parts\compositeBarActions.ts (640-666); V\src\vs\workbench\browser\parts\paneCompositePart.ts (758-805); C bundle (composite bar `getContextMenuActions`, aux bar options)
- Pieces:
  1. Right-clicking the strip or the title row lists the part's containers with check marks, and picking a row shows or hides that tab.
  2. The last visible tab cannot be hidden from the list.
  3. With the strip in the title row, stock adds a Views submenu to the list. [stock only]
  4. Stock also adds Activity Bar Position, Move Secondary Side Bar Left or Right, Show Icons or Show Labels, and Hide Secondary Side Bar below a separator. [stock only]
  5. Right-clicking a single tab adds the per-tab entries Hide or Keep, Hide Badge, Configure Keybinding, Move To, and Reset Location.

#### AX-4.6 Reorder tabs by dragging

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for `horizontallyBefore`, `getTargetIndex` and `canDrop` in the drag handler, all found; the drop marker color is set in the Cursor strip options. Whether it works in the running app is unverified)
- What: Drag a tab sideways and drop it before or after another tab to change the order. A colored marker shows where it will land. Dropping a container from another bar between tabs, or dropping a dragged view onto a tab, places the new tab at that spot and opens it. The part-wide drop target is AX-5.1.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (36-138, 319-320); V\src\vs\workbench\browser\parts\paneCompositeBar.ts (114-118); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (227); C bundle (composite drag and drop class)
- Pieces:
  1. Dragging a tab sideways and dropping it before or after another tab changes the order of the tabs.
  2. A colored marker shows where the dragged tab will land.
  3. Dropping a container from another bar between two tabs places its tab at that spot and opens it.
  4. Dropping a dragged view onto a tab places a new tab at that spot and opens it.

#### AX-4.7 Reach hidden tabs from the Additional Views button

- Tag: vscode. Evidence: source. Boundary: no. Cursor: changed (the label "Additional Views", NLS 3323, and the overflow code are present for the other bars, but this part passes `scrollable:!0`, so its width limit is unlimited and it never overflows; see AX-4.8)
- What: When the tabs do not all fit, the strip shows an Additional Views button at the end. Click it for a list of the tabs that are cut off, and pick one to open it. The tab you have open is kept in the strip when it can fit.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\compositeBar.ts (530-652); V\src\vs\workbench\browser\parts\compositeBarActions.ts (449-513); V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (217-218); C bundle (composite bar size computation)
- Pieces:
  1. When the tabs do not all fit, the strip shows an Additional Views button at its end. [stock only]
  2. Clicking the Additional Views button lists the tabs that are cut off, and picking one opens it. [stock only]
  3. The tab that is open stays in the strip whenever it can fit. [stock only]

#### AX-4.8 Scroll the tab strip sideways

- Tag: cursor. Evidence: bundle. Boundary: no. Cursor: cursor only (searched for `scrollable:!0`, `composite-bar-scrollable` and `autoScrollActiveItemIntoView`, all found; the CSS hides the bar until the strip is hovered)
- What: Cursor's strip for this part does not wrap or cut off tabs, it scrolls sideways with a thin 3 pixel bar that fades in on hover, it scrolls the active tab into view when you switch to it, and its tab chips are about 60 to 120 pixels wide with a fade on long text.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: C bundle (aux bar `getCompositeBarOptions`, composite bar `create` and `autoScrollActiveItemIntoView`); C workbench.desktop.main.css (`.auxiliarybar .composite-bar-scrollable`)
- Pieces:
  1. The strip never wraps or cuts off tabs, and scrolls sideways when the tabs are wider than the part. [cursor only]
  2. A thin 3 px scroll bar is hidden until the strip is hovered and then fades in. [cursor only]
  3. Switching to a tab scrolls that tab into view. [cursor only]
  4. Tab chips are about 60 to 120 px wide, and long text fades out at the end of the chip. [cursor only]

#### AX-4.9 Step to the previous or next view in the part

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and the labels, NLS 3311 and 3312, found)
- What: Run Previous Secondary Side Bar View or Next Secondary Side Bar View from the Command Palette to open the neighboring container in the strip. It wraps around at the ends. There is no default key.
- Commands: workbench.action.previousAuxiliaryBarView; workbench.action.nextAuxiliaryBarView
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarActions.ts (194-214); V\src\vs\workbench\browser\parts\compositeBarActions.ts (760-791); C bundle (two classes extending the switch action)
- Pieces:
  1. Running Next Secondary Side Bar View (`workbench.action.nextAuxiliaryBarView`) from the Command Palette opens the next container in the strip.
  2. Running Previous Secondary Side Bar View (`workbench.action.previousAuxiliaryBarView`) from the Command Palette opens the previous container in the strip.
  3. Both commands wrap around from the last container to the first and from the first to the last.
  4. Neither command has a default key.

#### AX-4.10 Title row buttons fold into an overflow menu when the part is narrow

- Tag: vscode. Evidence: source. Boundary: no. Cursor: missing (`getToolbarResponsiveBehavior` and `overflowFrom` have 0 hits in the bundle)
- What: newer-than-1.128. When the part is narrow, the buttons on the right of its title row collapse into a "..." menu, starting from the left-most button. The title label is also capped at 40 percent of the width.
- Commands: none found
- Menus: AuxiliaryBarTitle
- Keys: none found
- Settings: none found
- Source: N\src\vs\workbench\browser\parts\auxiliarybar\auxiliaryBarPart.ts (164-174); N\src\vs\workbench\browser\parts\auxiliarybar\media\auxiliaryBarPart.css (30-43); C bundle (0 hits)
- Pieces:
  1. Newer than 1.128: when the part is narrow, the buttons on the right of its title row fold into a "..." menu. [stock only]
  2. Newer than 1.128: the buttons fold starting from the left-most one. [stock only]
  3. Newer than 1.128: the title label is capped at 40 percent of the part's width. [stock only]


## AX-5 Moving views and containers in

(both). Drag, menu or command routes into the part, plus the extension contribution point. Cursor lacks the extension block's stock behavior but adds its own.

3 features.

#### AX-5.1 Drop a view or container onto the part

- Tag: both. Evidence: source. Boundary: no. Cursor: changed (the label "Drag a view here to display.", NLS 3926, is present, but `createEmptyPaneMessage` hides the message and returns before the drop target is registered when the part is the auxiliary bar)
- What: Drag a view header, a container tab, or the container title label over the part. It shows a drop color (`editorGroup.dropBackground`) when the drop is allowed. An empty part says "Drag a view here to display." Dropping a container moves all its views in, and dropping a view creates a new tab for it and opens it. Cursor does not register this part-wide drop target for this part and hides the empty message. Dropping onto a tab in its strip is AX-4.6, and the drag side of this is SB-1.12 and SB-1.13.
- Commands: none found
- Menus: none found
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositePart.ts (252-330, 368-380); V\src\vs\workbench\browser\parts\compositeBar.ts (36-138); C bundle (`createEmptyPaneMessage`); C nls.messages.json (3926)
- Pieces:
  1. Dragging a view header, a container tab, or a container title label over the part shows a drop color (`editorGroup.dropBackground`) over the part when the drop is allowed. [stock only]
  2. An empty part shows the message "Drag a view here to display." [stock only]
  3. Dropping a whole container on the part moves all of its views into the part. [stock only]
  4. Dropping a single view on the part creates a new tab for it and opens it. [stock only]

#### AX-5.2 Send a container into the part with Move To

- Tag: both. Evidence: source. Boundary: no. Cursor: present (searched for the labels "Move To", "Reset Location" and "Secondary Side Bar", NLS 3920-3925, and the ids `moveViewContainerTo`, 15 hits; all found)
- What: Right-click the tab of a container in the activity bar, the panel or the strip, choose Move To, then Secondary Side Bar. The container goes to the end of this part's tabs and opens. The Move To list leaves out the place the container is already in. Reset Location sends it back to its default home. Cursor still lists Secondary Side Bar in Move To, but what happens when you pick it was not run.
- Commands: none found
- Menus: tab right-click menu (built in `paneCompositeBar.ts`, not a MenuId); ViewContainerTitleContext
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\paneCompositeBar.ts (157-216); V\src\vs\workbench\services\views\browser\viewDescriptorService.ts (354-360, 898-922); C bundle (Move To builder); C nls.messages.json (3920-3925)
- Pieces:
  1. Right-clicking a container tab in the activity bar, the panel, or this part's strip shows a Move To submenu with the Primary Side Bar, Secondary Side Bar, and Panel rows, leaving out the place the container is in now.
  2. Picking Secondary Side Bar in Move To moves the container to the end of this part's tabs and opens it.
  3. Reset Location on the tab menu sends a moved container back to its default home and opens it, and the row shows only while the container is away from its default home.
  4. Right-clicking a container title also shows Reset Location while the container is away from its default home.

#### AX-5.3 Move all panel views to the secondary side bar

- Tag: both. Evidence: source. Boundary: no. Cursor: present (both ids and the label "Move Panel Views To Secondary Side Bar", NLS 3966-3967, found; the reverse move is PN-1.19)
- What: Run Move Panel Views To Secondary Side Bar to move every container in the panel into this part in one step. The part is shown, and the container that was open in the panel opens here if the part had none. A hidden twin command, `workbench.action.movePanelToSidePanel`, does the same.
- Commands: workbench.action.movePanelToSecondarySideBar; workbench.action.movePanelToSidePanel
- Menus: CommandPalette
- Keys: none found
- Settings: none found
- Source: V\src\vs\workbench\browser\parts\panel\panelActions.ts (344-397); C bundle (shared move class with the two ids); C nls.messages.json (3966-3967)
- Pieces:
  1. Running Move Panel Views To Secondary Side Bar (`workbench.action.movePanelToSecondarySideBar`) from the Command Palette moves every container in the panel into the part in one step.
  2. The part is shown if it was hidden, and the container that was open in the panel opens here if the part held no containers before the move.
  3. The hidden command `workbench.action.movePanelToSidePanel` does the same thing and does not appear in the Command Palette.
